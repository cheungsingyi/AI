# Chainlit Chatbot with DeepAgents and MCP Integration

This project implements an advanced chatbot using Chainlit, with DeepAgents for deep research capabilities and MCP (Model Context Protocol) integration for tool usage.

## Features

- **Normal Chat Mode**: Standard LLM interaction with MCP tool support
- **Deep Research Mode**: Advanced research capabilities using LangChain DeepAgents and Tavily search
- **MCP Integration**: Automatic loading of MCP tools from configuration file
- **Tool Usage**: Weather, calculator, search, and more through MCP servers

## Components

- `app.py`: Main application entry point
- `agents.py`: DeepAgents implementation for deep research
- `modules/`: Core functionality modules
  - `llm.py`: LLM configuration and setup
  - `mcp.py`: MCP tool handling and management
  - `agents.py`: Legacy simple agent implementation
- `mcp_servers/`: MCP server implementations
  - `test_stdio_server.py`: Standard I/O based MCP server
  - `test_http_server.py`: HTTP-based MCP server

## Setup

1. Install dependencies:
   ```
   pip install -r requirements.txt
   ```

2. Configure MCP servers in `mcp.json`:
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

3. Run the application:
   ```
   chainlit run app.py
   ```

## Usage

- **Normal Chat**: Just type your questions
- **Deep Research**: Type `/research` to toggle research mode
- **MCP Tools**: Available automatically from configuration
