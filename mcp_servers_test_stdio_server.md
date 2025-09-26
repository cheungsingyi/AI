# mcp_servers/test_stdio_server.py - Stdio MCP Server

```python
#!/usr/bin/env python3
"""Test MCP Server - Stdio Type"""

from fastmcp import FastMCP

# Create server
server = FastMCP("test-stdio-server")

@server.tool()
def get_weather(city: str) -> str:
    """Get weather information for a city"""
    weather_data = {
        "new york": "Sunny, 22°C",
        "london": "Cloudy, 15°C",
        "tokyo": "Rainy, 18°C",
        "paris": "Clear, 20°C"
    }
    return weather_data.get(city.lower(), f"Weather data not available for {city}")

@server.tool()
def calculate(expression: str) -> str:
    """Calculate a mathematical expression"""
    try:
        # Basic security - only allow safe operations
        allowed_chars = set("0123456789+-*/(). ")
        if not all(c in allowed_chars for c in expression):
            return "Error: Only basic arithmetic operations allowed"

        result = eval(expression, {"__builtins__": {}})
        return str(result)
    except Exception as e:
        return f"Error: {str(e)}"

@server.tool()
def search_database(query: str) -> str:
    """Search a mock database"""
    mock_data = {
        "python": "Python is a programming language",
        "ai": "AI stands for Artificial Intelligence",
        "mcp": "MCP stands for Model Context Protocol",
        "chainlit": "Chainlit is a framework for building chatbots"
    }

    results = [f"{k}: {v}" for k, v in mock_data.items() if query.lower() in k.lower() or query.lower() in v.lower()]
    return "\n".join(results) if results else f"No results found for '{query}'"

if __name__ == "__main__":
    server.run()
```
