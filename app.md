# app.py - Main Application

```python
import logging
from loguru import logger
import chainlit as cl
from modules.llm import get_llm
from agents import run_deep_research
from modules.mcp import get_all_mcp_tools, call_mcp_tool
from langchain_core.tools import tool
import asyncio
import subprocess
import time
import os

async def initialize_default_mcp_tools():
    """Initialize default MCP tools by loading from mcp.json configuration file"""
    try:
        logger.info("Initializing default MCP tools from mcp.json...")
        
        # Load MCP configuration from mcp.json
        mcp_config_path = os.path.join(os.path.dirname(__file__), "mcp.json")
        
        if not os.path.exists(mcp_config_path):
            logger.warning("mcp.json not found, skipping MCP tool initialization")
            return
        
        import json
        with open(mcp_config_path, "r") as f:
            mcp_config = json.load(f)
        
        mcp_servers = mcp_config.get("mcpServers", {})
        if not mcp_servers:
            logger.info("No MCP servers configured in mcp.json")
            return
        
        # Try to import langchain MCP adapters
        try:
            from langchain_mcp_adapters.client import MultiServerMCPClient
            logger.info(f"Loading {len(mcp_servers)} MCP servers from configuration...")
            
            # Create MCP client with the configured servers
            mcp_client = MultiServerMCPClient(mcp_servers)
            
            # Get tools from all configured servers
            mcp_tools = await mcp_client.get_tools()
            
            # Store tools in session for use by the LLM
            mcp_tools_dict = {}
            for tool in mcp_tools:
                server_name = getattr(tool, "server_name", "default")
                if server_name not in mcp_tools_dict:
                    mcp_tools_dict[server_name] = []
                mcp_tools_dict[server_name].append({
                    "name": tool.name,
                    "description": tool.description,
                    "input_schema": tool.inputSchema
                })
            
            cl.user_session.set("mcp_tools", mcp_tools_dict)
            logger.info(f"Successfully loaded MCP tools from {len(mcp_servers)} servers")
            
        except ImportError:
            logger.warning("langchain-mcp-adapters not available, falling back to mock tools")
            # Fallback to mock tools if langchain-mcp-adapters is not installed
            fallback_tools = []
            for server_name, server_config in mcp_servers.items():
                if "stdio-weather" in server_name:
                    fallback_tools.extend([
                        {
                            "name": "get_weather",
                            "description": "Get weather information for a city",
                            "input_schema": {
                                "type": "object",
                                "properties": {
                                    "city": {
                                        "type": "string",
                                        "description": "The city to get weather for"
                                    }
                                },
                                "required": ["city"]
                            }
                        }
                    ])
                elif "http-calculator" in server_name:
                    fallback_tools.extend([
                        {
                            "name": "calculate",
                            "description": "Calculate a mathematical expression",
                            "input_schema": {
                                "type": "object",
                                "properties": {
                                    "expression": {
                                        "type": "string",
                                        "description": "Mathematical expression to calculate"
                                    }
                                },
                                "required": ["expression"]
                            }
                        }
                    ])
            
            cl.user_session.set("mcp_tools", {"default": fallback_tools})
            logger.info(f"Loaded {len(fallback_tools)} fallback MCP tools")
            
    except Exception as e:
        logger.error(f"Error initializing MCP tools: {e}")
        # Don't fail the entire chat initialization

@cl.on_chat_start
async def on_chat_start():
    """Initialize the chat session with default MCP tools"""
    try:
        cl.user_session.set("chat_mode", "normal")  # normal or deep_research
        
        # Initialize MCP tools by default
        await initialize_default_mcp_tools()
        
        # Get available tools count
        mcp_tools = await get_all_mcp_tools()
        tool_count = len(mcp_tools)
        
        welcome_message = "Welcome to the AI Chatbot!"
        
        if tool_count > 0:
            welcome_message += f" ✅ {tool_count} MCP tools loaded automatically."
        else:
            welcome_message += " ⚠️ No MCP tools available (servers may not be running)."
        
        welcome_message += """
        
🤖 Normal Chat: Ask questions normally with LLM + MCP tools
🔍 Deep Research: Type '/research' for advanced research with web search
🛠️ MCP Tools: Available for calculations, weather, search, etc."""
        
        await cl.Message(content=welcome_message).send()
        logger.info(f"Chat session started successfully with {tool_count} MCP tools")
        
    except Exception as e:
        logger.error(f"Error starting chat: {e}")
        await cl.Message(content="Sorry, there was an error starting the chat.").send()

@cl.on_message
async def on_message(message: cl.Message):
    """Handle incoming messages"""
    logger.info(f"on_message called with: {message.content}")
    try:
        chat_mode = cl.user_session.get("chat_mode", "normal")

        if message.content.startswith("/research"):
            logger.info("Processing research command")
            # Toggle research mode
            if chat_mode == "normal":
                cl.user_session.set("chat_mode", "deep_research")
                await cl.Message(content="🔍 Deep Research mode activated! Ask your research question.").send()
            else:
                cl.user_session.set("chat_mode", "normal")
                await cl.Message(content="💬 Normal chat mode activated.").send()
            return

        if chat_mode == "deep_research":
            # Use deep research agent
            async with cl.Step(name="Deep Research", type="process") as step:
                step.input = message.content
                result = await run_deep_research(message.content)
                step.output = result
                await cl.Message(content=result).send()
        else:
            # Normal chat with LLM and MCP tools
            llm = get_llm()
            
            # Get available MCP tools and convert to LangChain format
            mcp_tools = await get_all_mcp_tools()
            langchain_tools = []
            
            for tool_info in mcp_tools:
                # Create a LangChain tool wrapper for each MCP tool
                # Use the tool's input schema to determine parameters
                input_schema = tool_info.get("input_schema", {})
                properties = input_schema.get("properties", {})
                
                # Get required parameters
                required_params = input_schema.get("required", [])
                
                if required_params:
                    # Create tool with proper parameters
                    param_name = required_params[0]  # Take first required param
                    param_info = properties.get(param_name, {})
                    param_desc = param_info.get("description", f"{param_name} parameter")
                    
                    @tool
                    async def mcp_tool_wrapper(**kwargs):
                        """{tool_info['description']}"""
                        try:
                            tool_name = tool_info["name"]
                            # Pass the kwargs directly as they should match the MCP tool parameters
                            result = await call_mcp_tool(tool_name, kwargs)
                            return str(result)
                        except Exception as e:
                            return f"Error calling MCP tool {tool_info['name']}: {str(e)}"
                    
                    # Set the function name and description
                    mcp_tool_wrapper.name = tool_info["name"]
                    mcp_tool_wrapper.description = tool_info["description"]
                    
                    # Add parameter info for LangChain
                    mcp_tool_wrapper.args_schema = None  # Let LangChain infer from function signature
                    
                else:
                    # Fallback for tools without proper schema
                    @tool
                    async def mcp_tool_wrapper(input_data: str):
                        """{tool_info['description']}"""
                        try:
                            tool_name = tool_info["name"]
                            input_dict = {"input": input_data}
                            result = await call_mcp_tool(tool_name, input_dict)
                            return str(result)
                        except Exception as e:
                            return f"Error calling MCP tool {tool_info['name']}: {str(e)}"
                    
                    mcp_tool_wrapper.name = tool_info["name"]
                    mcp_tool_wrapper.description = tool_info["description"]
                
                langchain_tools.append(mcp_tool_wrapper)
            
            if langchain_tools:
                # Bind tools to LLM
                llm_with_tools = llm.bind_tools(langchain_tools)
                
                # Create the initial message
                messages = [{"role": "user", "content": message.content}]
                
                # Get response from LLM
                response = await llm_with_tools.ainvoke(messages)
                
                # Check if there are tool calls
                if hasattr(response, 'tool_calls') and response.tool_calls:
                    # Execute tool calls
                    tool_results = []
                    for tool_call in response.tool_calls:
                        tool_name = tool_call["name"]
                        tool_args = tool_call["args"]
                        
                        try:
                            # Find the corresponding tool
                            tool_func = next((t for t in langchain_tools if t.name == tool_name), None)
                            if tool_func:
                                # Call the tool
                                with cl.Step(name=f"Tool: {tool_name}", type="tool") as step:
                                    step.input = tool_args
                                    result = await tool_func.ainvoke(tool_args)
                                    step.output = result
                                    tool_results.append(f"Tool {tool_name}: {result}")
                            else:
                                tool_results.append(f"Tool {tool_name} not found")
                        except Exception as e:
                            tool_results.append(f"Error calling tool {tool_name}: {str(e)}")
                    
                    # Send tool results and get final response
                    tool_message = "\n".join(tool_results)
                    await cl.Message(content=f"Tool results:\n{tool_message}").send()
                    
                    # Get final response with tool results
                    final_messages = messages + [
                        {"role": "assistant", "content": response.content},
                        {"role": "user", "content": f"Based on these tool results, please provide a final answer:\n{tool_message}"}
                    ]
                    final_response = await llm.ainvoke(final_messages)
                    await cl.Message(content=final_response.content).send()
                else:
                    # No tool calls, just send the response
                    await cl.Message(content=response.content).send()
            else:
                # No MCP tools available, simple LLM call
                response = await llm.ainvoke([{"role": "user", "content": message.content}])
                await cl.Message(content=response.content).send()

        logger.info(f"Processed message: {message.content[:50]}...")
    except Exception as e:
        logger.error(f"Error processing message: {e}")
        await cl.Message(content=f"Sorry, I encountered an error: {str(e)}").send()

# Import MCP handlers
from modules.mcp import on_mcp_connect, on_mcp_disconnect
```
