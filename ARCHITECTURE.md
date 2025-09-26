# Technical Architecture

This document provides a deeper technical overview of the chatbot architecture.

## Core Components

### 1. Chainlit Integration
The application is built on the Chainlit framework, which provides:
- Chat UI and session management
- MCP protocol support
- Tool visualization and step tracking

Key handlers:
- `@cl.on_chat_start` - Initializes the chat session and loads MCP tools
- `@cl.on_message` - Processes incoming messages and routes to appropriate handlers
- `@cl.on_mcp_connect` - Handles MCP server connections

### 2. LLM Integration
The application uses OpenRouter (fallback to OpenAI) for LLM capabilities:
- `get_llm()` - Factory function that returns configured LLM
- Tool binding via `llm.bind_tools(langchain_tools)`
- Conversation history management

### 3. MCP Architecture
The Model Context Protocol integration enables tool usage:
- **Tool Discovery**: Automatic loading from `mcp.json`
- **Tool Registration**: Dynamic registration with Chainlit UI
- **Tool Execution**: Parameter validation and error handling
- **Mock Implementation**: Fallback for when servers are unavailable

### 4. DeepAgents System
The deep research capability uses LangChain's DeepAgents:
- **Main Agent**: Coordinates research process and generates final report
- **Analysis Sub-Agent**: Specializes in synthesizing complex information
- **Verification Sub-Agent**: Focuses on fact-checking and source validation
- **Tavily Integration**: Provides web search capabilities

## Data Flow

1. User sends message to Chainlit UI
2. Message is routed based on mode:
   - Normal → LLM with tools
   - Research → DeepAgents
3. For normal mode:
   - LLM determines if tools are needed
   - Tools are executed via MCP
   - Results are returned to LLM for final response
4. For research mode:
   - Query is sent to DeepAgents
   - Main agent coordinates research
   - Sub-agents perform specialized tasks
   - Tavily provides web search results
   - Structured report is generated

## Extension Points

1. **New MCP Servers**: Add new entries to `mcp.json`
2. **New Tools**: Implement in MCP servers
3. **New Sub-Agents**: Add to the DeepAgents configuration
4. **Custom LLMs**: Modify `get_llm()` function
