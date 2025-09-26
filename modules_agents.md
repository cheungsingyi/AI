# modules/agents.py - Simple Agents (Legacy)

```python
from modules.llm import get_llm
import chainlit as cl

async def run_deep_research(query: str) -> str:
    """Run deep research on a query using enhanced prompting"""
    try:
        llm = get_llm()
        
        # Create a research-focused prompt
        research_prompt = f"""You are an expert research assistant. Please provide a comprehensive, well-structured response to the following query:

Query: {query}

Please structure your response with:
1. Key findings and insights
2. Supporting evidence or reasoning
3. Any limitations or caveats
4. Suggestions for further exploration

Be thorough but concise. Use clear headings and bullet points where appropriate."""

        response = await llm.ainvoke([{"role": "user", "content": research_prompt}])
        return response.content
    except Exception as e:
        return f"Error in deep research: {str(e)}. Query was: {query}"
```
