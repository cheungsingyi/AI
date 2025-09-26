# modules/llm.py - LLM Configuration

```python
import os
from langchain_openai import ChatOpenAI
from langchain_core.language_models import BaseChatModel

def get_llm() -> BaseChatModel:
    """Get the LLM instance to use for the chatbot"""
    # Use OpenRouter API
    openrouter_api_key = os.environ.get("OPENROUTER_API_KEY", "")
    
    # Default to OpenRouter if key is available
    if openrouter_api_key:
        return ChatOpenAI(
            openai_api_key=openrouter_api_key,
            base_url="https://openrouter.ai/api/v1",
            model="anthropic/claude-3-opus-20240229",
            temperature=0.7,
            max_tokens=4000,
        )
    else:
        # Fallback to OpenAI if OpenRouter key is not available
        return ChatOpenAI(
            model="gpt-4-turbo",
            temperature=0.7,
            max_tokens=4000,
        )
```
