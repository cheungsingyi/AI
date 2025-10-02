# mcp.py - MCP Server Connection and Tool Management

## Description
MCP server connection and tool management optimized with langchain-mcp-adapters.

## Code

```python
"""MCP server connection and tool management - Optimized with langchain-mcp-adapters"""
import chainlit as cl
from mcp import ClientSession
from typing import List, Any
from loguru import logger
from langchain_mcp_adapters.tools import load_mcp_tools
from langchain_core.tools import BaseTool

# Log that MCP module is loaded
logger.info("📦 MCP module loaded - hooks registered")
logger.info(f"   Chainlit version: {cl.__version__ if hasattr(cl, '__version__') else 'unknown'}")


@cl.on_mcp_connect
async def on_mcp_connect(connection, session: ClientSession):
    """
    Called when an MCP connection is established.
    Now uses langchain-mcp-adapters for efficient tool loading.
    """
    logger.info(f"🔌 MCP connection established: {connection.name}")
    logger.info(f"   Connection type: {type(connection)}")
    logger.info(f"   Session type: {type(session)}")
    
    try:
        # Use official adapter - returns LangChain tools directly!
        logger.info(f"   Loading tools from {connection.name}...")
        tools = await load_mcp_tools(session)
        logger.info(f"   ✅ Loaded {len(tools)} tools: {[t.name for t in tools]}")
        
        # Store LangChain-ready tools (no conversion needed!)
        mcp_tools = cl.user_session.get("mcp_tools", {})
        mcp_tools[connection.name] = tools
        cl.user_session.set("mcp_tools", mcp_tools)
        
        logger.info(f"   📦 Stored tools in session. Total servers: {len(mcp_tools)}")
        
        await cl.Message(content=f"✅ MCP server '{connection.name}' connected with {len(tools)} tools").send()
        logger.info(f"✨ Successfully loaded {len(tools)} tools from {connection.name}")
    except Exception as e:
        logger.error(f"❌ Error connecting to MCP server '{connection.name}': {e}")
        logger.exception(e)
        await cl.Message(content=f"❌ Error connecting to MCP server '{connection.name}': {str(e)}").send()


@cl.on_mcp_disconnect
async def on_mcp_disconnect(name: str, session: ClientSession):
    """Called when an MCP connection is terminated"""
    logger.info(f"MCP connection terminated: {name}")
    
    # Remove tools from session
    mcp_tools = cl.user_session.get("mcp_tools", {})
    if name in mcp_tools:
        del mcp_tools[name]
        cl.user_session.set("mcp_tools", mcp_tools)
    
    await cl.Message(content=f"MCP server '{name}' disconnected").send()

async def get_all_mcp_tools() -> List[BaseTool]:
    """
    Get all available MCP tools from connected servers.
    Returns LangChain-compatible tools ready to use.
    """
    mcp_tools = cl.user_session.get("mcp_tools", {})
    all_tools = []
    for connection_tools in mcp_tools.values():
        all_tools.extend(connection_tools)
    return all_tools


async def get_tool_by_name(tool_name: str) -> BaseTool:
    """
    Get a specific tool by name.
    
    Args:
        tool_name: Name of the tool to retrieve
        
    Returns:
        LangChain tool if found
        
    Raises:
        ValueError: If tool not found
    """
    all_tools = await get_all_mcp_tools()
    for tool in all_tools:
        if tool.name == tool_name:
            return tool
    raise ValueError(f"Tool {tool_name} not found in any connected MCP server")


def get_tool_names() -> List[str]:
    """
    Get names of all available tools (synchronous).
    
    Returns:
        List of tool names
    """
    mcp_tools = cl.user_session.get("mcp_tools", {})
    all_names = []
    for connection_tools in mcp_tools.values():
        all_names.extend([tool.name for tool in connection_tools])
    return all_names
```

## Key Functions

### 1. `@cl.on_mcp_connect`
- Triggered when MCP server connects via UI
- Uses `load_mcp_tools()` for efficient conversion
- Stores LangChain-ready tools in session

### 2. `@cl.on_mcp_disconnect`
- Handles server disconnection
- Removes tools from session
- Notifies user

### 3. `get_all_mcp_tools()`
- Retrieves all tools from all connected servers
- Returns LangChain BaseTool instances
- Async function

### 4. `get_tool_by_name(tool_name)`
- Finds specific tool by name
- Raises ValueError if not found
- Async function

### 5. `get_tool_names()`
- Returns list of all tool names
- Synchronous helper function

## Integration

### LangChain MCP Adapters
- Uses `langchain-mcp-adapters` package
- Automatic tool conversion
- No manual schema mapping needed

### Tool Storage
```python
# Session structure
{
    "mcp_tools": {
        "server-name-1": [BaseTool, BaseTool, ...],
        "server-name-2": [BaseTool, BaseTool, ...]
    }
}
```

## Features
- ✅ Automatic LangChain conversion
- ✅ Multi-server support
- ✅ Session-based storage
- ✅ Error handling and logging
- ✅ UI notifications
- ✅ Tool lookup utilities
