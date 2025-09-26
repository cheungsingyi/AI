# Chainlit Chatbot Documentation

Welcome to the comprehensive documentation for the Chainlit Chatbot with DeepAgents, MCP integration, and RAG capabilities.

## Table of Contents

### Core Documentation
- [Project Overview](INDEX.md)
- [Technical Architecture](ARCHITECTURE.md)
- [MCP Integration Guide](MCP_GUIDE.md)
- [DeepAgents Research System](DEEPAGENTS.md)
- [RAG Implementation](rag.md)
- [RAG Integration Guide](RAG_INTEGRATION.md)

### Code Documentation
- [app.py](app.md) - Main application
- [agents.py](agents.md) - DeepAgents implementation
- [modules/llm.py](modules_llm.md) - LLM configuration
- [modules/mcp.py](modules_mcp.md) - MCP handling
- [modules/agents.py](modules_agents.md) - Legacy agents
- [mcp_servers/test_stdio_server.py](mcp_servers_test_stdio_server.md) - Stdio MCP server
- [mcp_servers/test_http_server.py](mcp_servers_test_http_server.md) - HTTP MCP server
- [test_scenarios.py](test_scenarios.md) - Test scenarios
- [test_chatbot.py](test_chatbot.md) - Chatbot tests
- [mcp.json](mcp_json.md) - MCP configuration

## Quick Start

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Configure MCP servers in `mcp.json`

3. Run the application:
   ```bash
   chainlit run app.py
   ```

4. Use the chatbot:
   - **Normal Chat**: Just type your questions
   - **Deep Research**: Type `/research` to toggle research mode
   - **RAG Mode**: Follow the [RAG Integration Guide](RAG_INTEGRATION.md)

## Key Features

- **Multi-Mode Operation**: Normal chat, deep research, and RAG capabilities
- **MCP Integration**: Tool usage through Model Context Protocol
- **DeepAgents**: Advanced research with web search and sub-agents
- **RAG System**: Document-based Q&A with source attribution

## Contributing

To extend the chatbot:
1. Add new MCP tools in `mcp_servers/`
2. Configure in `mcp.json`
3. Add new sub-agents in `agents.py`
4. Add documents to the RAG system

## License

This project is available under the MIT License.
