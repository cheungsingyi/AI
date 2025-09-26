# mcp.json - MCP Configuration

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
