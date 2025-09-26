# MCP Integration Guide

This document explains how the Model Context Protocol (MCP) is integrated into the chatbot.

## What is MCP?

The Model Context Protocol (MCP) is a standard that allows Large Language Models (LLMs) to interact with external tools and services. It provides a consistent interface for tools to expose their functionality to LLMs.

## MCP Components in This Project

### 1. MCP Configuration (`mcp.json`)

```json
{
  "mcpServers": {
    "stdio-weather": {
      "command": "python",
      "args": ["mcp_servers/test_stdio_server.py"],
      "env": {}
    },
    "http-calculator": {
      "url": "http://localhost:8002",
      "headers": {},
      "timeout": 30
    }
  }
}
```

This configuration defines:
- **stdio-weather**: A stdio-based MCP server for weather information
- **http-calculator**: An HTTP-based MCP server for calculations

### 2. MCP Servers

#### Stdio Server (`mcp_servers/test_stdio_server.py`)
- Uses FastMCP for stdio communication
- Provides tools: `get_weather`, `calculate`, `search_database`
- Runs as a subprocess

#### HTTP Server (`mcp_servers/test_http_server.py`)
- Uses FastAPI + FastMCP for HTTP communication
- Provides tools: `translate_text`, `get_news`, `stream_analysis`
- Runs as a web server on port 8002

### 3. MCP Handlers (`modules/mcp.py`)

- `on_mcp_connect`: Handles new MCP server connections
- `on_mcp_disconnect`: Handles MCP server disconnections
- `get_all_mcp_tools`: Retrieves all available MCP tools
- `call_mcp_tool`: Executes MCP tools with parameters

### 4. Automatic MCP Loading (`app.py`)

```python
async def initialize_default_mcp_tools():
    """Initialize default MCP tools by loading from mcp.json configuration file"""
    # Load MCP configuration from mcp.json
    mcp_config_path = os.path.join(os.path.dirname(__file__), "mcp.json")
    
    if os.path.exists(mcp_config_path):
        with open(mcp_config_path, "r") as f:
            mcp_config = json.load(f)
        
        mcp_servers = mcp_config.get("mcpServers", {})
        # Initialize MCP clients and load tools
```

## How MCP Tools Are Used

1. **Tool Discovery**:
   - MCP tools are loaded from `mcp.json` on chat start
   - Tools are registered with Chainlit UI panel

2. **Tool Conversion**:
   - MCP tools are converted to LangChain tools
   - Tool schemas are preserved for parameter validation

3. **Tool Execution**:
   - LLM decides when to use tools based on user queries
   - Tool calls are executed via MCP servers
   - Results are returned to LLM for final response

4. **Fallback Mechanism**:
   - If MCP servers are unavailable, mock implementations are used
   - This ensures the chatbot remains functional even without servers

## Adding New MCP Tools

1. **Create a new MCP server**:
   ```python
   from fastmcp import FastMCP
   
   server = FastMCP("my-server")
   
   @server.tool()
   def my_tool(param1: str, param2: int) -> str:
       """Tool description"""
       # Implementation
       return result
   ```

2. **Add to `mcp.json`**:
   ```json
   "my-server": {
     "command": "python",
     "args": ["path/to/my_server.py"],
     "env": {}
   }
   ```

3. **Restart the chatbot** - Tools will be automatically loaded
