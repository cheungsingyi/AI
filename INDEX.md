# Chainlit Chatbot with DeepAgents and MCP Integration

This project implements an advanced chatbot using Chainlit, with DeepAgents for deep research capabilities and MCP (Model Context Protocol) integration for tool usage.

## Project Structure

### Core Application
- [app.py](app.md) - Main application entry point and Chainlit handlers
- [agents.py](agents.md) - DeepAgents implementation for deep research with Tavily search

### Modules
- [modules/llm.py](modules_llm.md) - LLM configuration and setup
- [modules/mcp.py](modules_mcp.md) - MCP tool handling and management
- [modules/agents.py](modules_agents.md) - Legacy simple agent implementation

### MCP Servers
- [mcp_servers/test_stdio_server.py](mcp_servers_test_stdio_server.md) - Standard I/O based MCP server
- [mcp_servers/test_http_server.py](mcp_servers_test_http_server.md) - HTTP-based MCP server

### Testing
- [test_scenarios.py](test_scenarios.md) - Test scenarios for the chatbot
- [test_chatbot.py](test_chatbot.md) - Chatbot testing script

### Configuration
- [mcp.json](mcp_json.md) - MCP server configuration

## Features

### 1. Multi-Mode Operation
- **Normal Chat Mode**: Standard LLM interaction with MCP tool support
- **Deep Research Mode**: Advanced research capabilities using LangChain DeepAgents

### 2. MCP Integration
- Automatic loading of MCP tools from configuration file
- Tool usage for weather, calculator, search, and more
- Support for both stdio and HTTP-based MCP servers

### 3. DeepAgents Architecture
- Multi-agent system with specialized sub-agents
- Web search integration with Tavily API
- Structured research reports with executive summaries

## Setup and Usage

### Installation
```bash
pip install -r requirements.txt
```

### Running the Application
```bash
chainlit run app.py
```

### Using the Chatbot
- **Normal Chat**: Just type your questions
- **Deep Research**: Type `/research` to toggle research mode
- **MCP Tools**: Available automatically from configuration

## Implementation Details

The application uses a modular architecture with:

1. **Chainlit Framework**: For UI and chat management
2. **LangChain**: For LLM interactions and tool binding
3. **DeepAgents**: For advanced research capabilities
4. **MCP Protocol**: For tool integration and extensibility

The MCP tools are loaded automatically from the `mcp.json` configuration file and registered with the UI panel, making them immediately available for use without manual setup.
