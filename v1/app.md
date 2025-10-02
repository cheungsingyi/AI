# app.py - Main Chainlit Application

## Description
Main Chainlit application for AI Chatbot with MCP integration using Hybrid Mode (Direct MCP + UI support).

## Code

```python
"""Main Chainlit application for AI Chatbot with MCP integration - Hybrid Mode"""
from loguru import logger
import chainlit as cl
from modules.handlers import handle_mode_toggle, handle_deep_research, handle_normal_chat
from langchain_mcp_adapters.client import MultiServerMCPClient
from mcp import ClientSession
import json

@cl.on_chat_start
async def on_chat_start():
    """Initialize chat session with direct MCP client"""
    try:
        logger.info("=" * 60)
        logger.info("🚀 Chat session starting with DIRECT MCP integration...")
        
        cl.user_session.set("chat_mode", "normal")
        
        # Load MCP configuration
        with open("mcp.json") as f:
            mcp_config = json.load(f)
        
        logger.info(f"📋 Found {len(mcp_config.get('mcpServers', {}))} MCP servers in config")
        
        # Convert config to MultiServerMCPClient format
        servers = {}
        for name, config in mcp_config.get("mcpServers", {}).items():
            servers[name] = {
                "command": config["command"],
                "args": config["args"],
                "env": config.get("env", {}),
                "transport": "stdio"
            }
            logger.info(f"   📡 Configuring server: {name}")
        
        # Create MCP client and load tools directly
        logger.info("🔌 Initializing MultiServerMCPClient...")
        client = MultiServerMCPClient(servers)
        
        # Load all tools (no context manager needed!)
        logger.info("🛠️ Loading tools from MCP servers...")
        tools = await client.get_tools()
        tool_count = len(tools)
        
        logger.info(f"✅ Loaded {tool_count} tools: {[t.name for t in tools]}")
        
        # Store in session
        cl.user_session.set("mcp_client", client)
        cl.user_session.set("tools", tools)
        
        status = f"✅ {tool_count} MCP tools loaded" if tool_count > 0 else "⚠️ No MCP tools available"
        
        # Create detailed tool list
        tool_details = "\n".join([f"  • **{t.name}**: {t.description}" for t in tools])
        
        welcome = f"""Welcome to the AI Chatbot! {status}

🤖 **Normal Chat**: Ask questions with LLM + MCP tools
🔍 **Deep Research**: Type '/research' for web-powered research

## 🛠️ Available MCP Tools ({tool_count} tools)

{tool_details}

💡 **Using Direct MCP Integration** - Tools are loaded and ready to use!

**Note:** Tools won't appear in the MCP panel because we're using direct integration for better performance and reliability."""
        
        await cl.Message(content=welcome).send()
        logger.info(f"✨ Chat session started with {tool_count} MCP tools")
        logger.info("=" * 60)
    except Exception as e:
        logger.error(f"❌ Error starting chat: {e}")
        logger.exception(e)
        await cl.Message(content=f"Sorry, there was an error starting the chat: {str(e)}").send()


@cl.on_mcp_connect
async def on_mcp_connect(connection, session: ClientSession):
    """Handle MCP servers added via Chainlit UI"""
    logger.info(f"🔌 Chainlit MCP connection: {connection.name}")
    
    try:
        # List available tools from this connection
        result = await session.list_tools()
        
        # Convert to our format
        ui_tools = []
        for tool in result.tools:
            ui_tools.append({
                "name": tool.name,
                "description": tool.description,
                "input_schema": tool.inputSchema,
            })
        
        logger.info(f"   ✅ Loaded {len(ui_tools)} tools from UI: {[t['name'] for t in ui_tools]}")
        
        # Store UI-added tools separately
        ui_mcp_tools = cl.user_session.get("ui_mcp_tools", {})
        ui_mcp_tools[connection.name] = ui_tools
        cl.user_session.set("ui_mcp_tools", ui_mcp_tools)
        
        # Notify user
        await cl.Message(
            content=f"✅ MCP server '{connection.name}' connected with {len(ui_tools)} tools:\n" + 
                    "\n".join([f"  • **{t['name']}**: {t['description']}" for t in ui_tools])
        ).send()
        
    except Exception as e:
        logger.error(f"Error loading tools from {connection.name}: {e}")
        await cl.Message(content=f"❌ Error loading tools from '{connection.name}': {str(e)}").send()


@cl.on_mcp_disconnect
async def on_mcp_disconnect(name: str, session: ClientSession):
    """Handle MCP server disconnection"""
    logger.info(f"🔌 MCP disconnected: {name}")
    
    # Remove from UI tools
    ui_mcp_tools = cl.user_session.get("ui_mcp_tools", {})
    if name in ui_mcp_tools:
        del ui_mcp_tools[name]
        cl.user_session.set("ui_mcp_tools", ui_mcp_tools)
        await cl.Message(content=f"MCP server '{name}' disconnected").send()


@cl.on_chat_end
async def on_chat_end():
    """Cleanup MCP client when chat ends"""
    try:
        client = cl.user_session.get("mcp_client")
        if client:
            logger.info("🔌 Closing MCP client...")
            # MultiServerMCPClient handles cleanup automatically
            # No explicit close needed in 0.1.0+
            logger.info("✅ MCP client session ended")
    except Exception as e:
        logger.error(f"Error closing MCP client: {e}")

@cl.on_message
async def on_message(message: cl.Message):
    """Handle incoming messages"""
    try:
        chat_mode = cl.user_session.get("chat_mode", "normal")
        
        # Handle mode toggle command
        if message.content.startswith("/research"):
            await handle_mode_toggle(chat_mode)
            return
        
        # Route to appropriate handler
        if chat_mode == "deep_research":
            await handle_deep_research(message.content)
        else:
            await handle_normal_chat(message.content)
        
        logger.info(f"Processed: {message.content[:50]}...")
    except Exception as e:
        logger.error(f"Error processing message: {e}")
        await cl.Message(content=f"Sorry, I encountered an error: {str(e)}").send()
```

## Key Features

### 1. Direct MCP Integration
- Loads MCP servers from `mcp.json` configuration file
- Uses `MultiServerMCPClient` for direct tool loading
- Tools available immediately on app start

### 2. Hybrid MCP Support
- **Direct Integration**: Auto-loads tools from config
- **UI Integration**: Supports user-added MCP servers via Chainlit UI
- Both tool sets work together seamlessly

### 3. Chat Modes
- **Normal Chat**: LLM with MCP tool support
- **Deep Research**: Advanced research with DeepAgents

### 4. Lifecycle Hooks
- `@cl.on_chat_start`: Initialize session and load tools
- `@cl.on_mcp_connect`: Handle UI-added MCP servers
- `@cl.on_mcp_disconnect`: Cleanup disconnected servers
- `@cl.on_chat_end`: Session cleanup
- `@cl.on_message`: Message routing

## Dependencies
- `chainlit`: UI framework
- `langchain_mcp_adapters`: MCP to LangChain tool conversion
- `loguru`: Logging
- `mcp`: MCP protocol support
