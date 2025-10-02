# llm.py - LLM Configuration

## Description
LLM configuration and initialization using OpenRouter.

## Code

```python
"""LLM configuration and initialization"""
import os
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()


def get_llm():
    """
    Get configured LLM instance using OpenRouter.
    
    Returns:
        ChatOpenAI: Configured LLM instance
    """
    return ChatOpenAI(
        api_key=os.getenv("OPENROUTER_API_KEY"),
        model=os.getenv("OPENROUTER_MODEL", "x-ai/grok-4-fast:free"),
        base_url="https://openrouter.ai/api/v1"
    )
```

## Configuration

### Environment Variables
- `OPENROUTER_API_KEY`: Your OpenRouter API key
- `OPENROUTER_MODEL`: Model to use (default: `x-ai/grok-4-fast:free`)

### OpenRouter Integration
- Uses OpenRouter as LLM provider
- Compatible with LangChain's ChatOpenAI interface
- Supports tool calling via `bind_tools()`

## Usage

```python
from modules.llm import get_llm

# Get LLM instance
llm = get_llm()

# Simple call
response = await llm.ainvoke([{"role": "user", "content": "Hello"}])

# With tools
llm_with_tools = llm.bind_tools(tools)
response = await llm_with_tools.ainvoke(messages)
```

## Features
- ✅ OpenRouter integration
- ✅ Environment-based configuration
- ✅ Tool calling support
- ✅ Async operations
- ✅ Default model fallback
