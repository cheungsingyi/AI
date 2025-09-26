# modules/mcp.py - MCP Handling

```python
import chainlit as cl
from mcp import ClientSession
from typing import Dict, List, Any

@cl.on_mcp_connect
async def on_mcp_connect(connection, session: ClientSession):
    """Called when an MCP connection is established"""
    print(f"MCP connection established: {connection.name}")

    # List available tools
    try:
        result = await session.list_tools()
        tools = [{
            "name": t.name,
            "description": t.description,
            "input_schema": t.inputSchema,
        } for t in result.tools]

        # Store tools for later use
        mcp_tools = cl.user_session.get("mcp_tools", {})
        mcp_tools[connection.name] = tools
        cl.user_session.set("mcp_tools", mcp_tools)

        await cl.Message(content=f"MCP server '{connection.name}' connected with {len(tools)} tools available.").send()
    except Exception as e:
        await cl.Message(content=f"Error connecting to MCP server '{connection.name}': {str(e)}").send()

@cl.on_mcp_disconnect
async def on_mcp_disconnect(name: str, session: ClientSession):
    """Called when an MCP connection is terminated"""
    print(f"MCP connection terminated: {name}")

    # Remove tools from session
    mcp_tools = cl.user_session.get("mcp_tools", {})
    if name in mcp_tools:
        del mcp_tools[name]
        cl.user_session.set("mcp_tools", mcp_tools)

    await cl.Message(content=f"MCP server '{name}' disconnected.").send()

async def get_all_mcp_tools() -> List[Dict[str, Any]]:
    """Get all available MCP tools from connected servers, with defaults if none connected"""
    mcp_tools = cl.user_session.get("mcp_tools", {})
    all_tools = []
    for connection_tools in mcp_tools.values():
        all_tools.extend(connection_tools)
    
    # If no MCP tools are connected, provide default mock tools
    if not all_tools:
        default_tools = [
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
            },
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
            },
            {
                "name": "search_database",
                "description": "Search a mock database",
                "input_schema": {
                    "type": "object",
                    "properties": {
                        "query": {
                            "type": "string",
                            "description": "Search query for the database"
                        }
                    },
                    "required": ["query"]
                }
            }
        ]
        all_tools.extend(default_tools)
    
    return all_tools

async def call_mcp_tool(tool_name: str, tool_input: Dict[str, Any]) -> Any:
    """Call an MCP tool by name, with mock implementations for default tools"""
    # Find which MCP connection has this tool
    mcp_tools = cl.user_session.get("mcp_tools", {})
    for connection_name, tools in mcp_tools.items():
        if any(t["name"] == tool_name for t in tools):
            mcp_session, _ = cl.context.session.mcp_sessions.get(connection_name)
            if mcp_session:
                result = await mcp_session.call_tool(tool_name, tool_input)
                return result
    
    # If no real MCP connection found, try mock implementations for default tools
    if tool_name == "get_weather" and "city" in tool_input:
        city = tool_input["city"].lower()
        weather_data = {
            "new york": "Sunny, 22°C",
            "london": "Cloudy, 15°C", 
            "tokyo": "Rainy, 18°C",
            "paris": "Clear, 20°C"
        }
        return weather_data.get(city, f"Weather data not available for {city}")
    
    elif tool_name == "calculate" and "expression" in tool_input:
        expression = tool_input["expression"]
        try:
            # Basic security - only allow safe operations
            allowed_chars = set("0123456789+-*/(). ")
            if not all(c in allowed_chars for c in expression):
                return "Error: Only basic arithmetic operations allowed"
            result = eval(expression, {"__builtins__": {}})
            return str(result)
        except Exception as e:
            return f"Error: {str(e)}"
    
    elif tool_name == "search_database" and "query" in tool_input:
        query = tool_input["query"].lower()
        mock_data = {
            "python": "Python is a programming language",
            "ai": "AI stands for Artificial Intelligence",
            "mcp": "MCP stands for Model Context Protocol",
            "chainlit": "Chainlit is a framework for building chatbots"
        }
        results = [f"{k}: {v}" for k, v in mock_data.items() if query in k.lower() or query in v.lower()]
        return "\n".join(results) if results else f"No results found for '{query}'"
    
    raise ValueError(f"Tool {tool_name} not found in any connected MCP server")
```
