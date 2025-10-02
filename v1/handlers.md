# handlers.py - Message Handling Logic

## Description
Message handling logic for chat modes with optimized Direct MCP integration.

## Code

```python
"""Message handling logic for chat modes - Optimized with Direct MCP"""
import chainlit as cl
from loguru import logger
from modules.llm import get_llm
from agents import run_deep_research


async def get_all_tools():
    """Get all tools from session (direct MCP integration)"""
    return cl.user_session.get("tools", [])


async def handle_mode_toggle(current_mode: str) -> str:
    """Toggle between normal and deep research mode"""
    if current_mode == "normal":
        cl.user_session.set("chat_mode", "deep_research")
        await cl.Message(content="🔍 Deep Research mode activated! Ask your research question.").send()
        return "deep_research"
    else:
        cl.user_session.set("chat_mode", "normal")
        await cl.Message(content="💬 Normal chat mode activated.").send()
        return "normal"


async def handle_deep_research(query: str):
    """
    Handle deep research queries with MCP tool integration.
    Tools are now LangChain-compatible and ready to use directly.
    """
    # Get all available tools (already LangChain format!)
    tools = await get_all_tools()
    
    async with cl.Step(name="Deep Research", type="process") as step:
        step.input = query
        
        # Tools are already LangChain-compatible, pass directly
        result = await run_deep_research(
            query=query,
            langchain_tools=tools  # Pass LangChain tools directly
        )
        
        step.output = result
        await cl.Message(content=result).send()


async def handle_normal_chat(query: str):
    """
    Handle normal chat with optional MCP tool usage.
    Tools are already in LangChain format - no conversion needed!
    """
    llm = get_llm()
    tools = await get_all_tools()
    
    if not tools:
        # No tools available, simple LLM call
        response = await llm.ainvoke([{"role": "user", "content": query}])
        await cl.Message(content=response.content).send()
        return
    
    # Tools are already LangChain-compatible! No conversion needed!
    llm_with_tools = llm.bind_tools(tools)
    messages = [{"role": "user", "content": query}]
    
    # Get LLM response
    response = await llm_with_tools.ainvoke(messages)
    
    # Check if LLM wants to use tools
    if hasattr(response, 'tool_calls') and response.tool_calls:
        await _execute_tool_calls(response, tools, messages, llm)
    else:
        # No tool calls, send direct response
        await cl.Message(content=response.content).send()


async def _execute_tool_calls(response, langchain_tools: list, messages: list, llm):
    """Execute tool calls and get final response"""
    tool_results = []
    
    for tool_call in response.tool_calls:
        tool_name = tool_call["name"]
        tool_args = tool_call["args"]
        
        try:
            tool_func = next((t for t in langchain_tools if t.name == tool_name), None)
            if tool_func:
                async with cl.Step(name=f"Tool: {tool_name}", type="tool") as step:
                    step.input = tool_args
                    result = await tool_func.ainvoke(tool_args)
                    step.output = result
                    tool_results.append(f"Tool {tool_name}: {result}")
            else:
                tool_results.append(f"Tool {tool_name} not found")
        except Exception as e:
            logger.error(f"Error calling tool {tool_name}: {e}")
            tool_results.append(f"Error calling tool {tool_name}: {str(e)}")
    
    # Send tool results
    tool_message = "\n".join(tool_results)
    await cl.Message(content=f"Tool results:\n{tool_message}").send()
    
    # Get final response with tool results
    final_messages = messages + [
        {"role": "assistant", "content": response.content or ""},
        {"role": "user", "content": f"Based on these tool results, please provide a final answer:\n{tool_message}"}
    ]
    final_response = await llm.ainvoke(final_messages)
    await cl.Message(content=final_response.content).send()
```

## Key Functions

### 1. `get_all_tools()`
- Retrieves tools from user session
- Returns LangChain-compatible tools
- No conversion needed

### 2. `handle_mode_toggle(current_mode)`
- Switches between normal and deep research modes
- Updates session state
- Sends confirmation message

### 3. `handle_deep_research(query)`
- Processes research queries
- Uses DeepAgents for comprehensive research
- Displays results in Chainlit Step

### 4. `handle_normal_chat(query)`
- Handles regular chat messages
- Binds tools to LLM
- Executes tool calls if needed

### 5. `_execute_tool_calls(response, tools, messages, llm)`
- Executes LLM-requested tool calls
- Displays each tool execution in UI
- Sends results back to LLM for final response

## Flow

```
User Message
    ↓
handle_normal_chat()
    ↓
LLM with tools
    ↓
Tool calls? ──No──→ Direct response
    ↓ Yes
_execute_tool_calls()
    ↓
Execute each tool
    ↓
Send results to LLM
    ↓
Final response
```

## Features
- ✅ Direct tool access from session
- ✅ No tool conversion overhead
- ✅ Visual tool execution steps
- ✅ Error handling for tool failures
- ✅ Fallback for missing tools
