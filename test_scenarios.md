# test_scenarios.py - Test Scenarios

```python
"""Test scenarios for the chatbot"""

import asyncio
from modules.llm import get_llm
from modules.mcp import get_all_mcp_tools, call_mcp_tool
from agents import run_deep_research

async def test_llm():
    """Test the LLM"""
    llm = get_llm()
    response = await llm.ainvoke([{"role": "user", "content": "Say hello!"}])
    print(f"LLM response: {response.content}")

async def test_mcp_tools():
    """Test MCP tools"""
    # Get all available tools
    tools = await get_all_mcp_tools()
    print(f"Found {len(tools)} MCP tools:")
    for tool in tools:
        print(f"- {tool['name']}: {tool['description']}")
    
    # Test weather tool
    weather = await call_mcp_tool("get_weather", {"city": "london"})
    print(f"Weather in London: {weather}")
    
    # Test calculator tool
    calculation = await call_mcp_tool("calculate", {"expression": "2 + 2 * 3"})
    print(f"2 + 2 * 3 = {calculation}")
    
    # Test search tool
    search = await call_mcp_tool("search_database", {"query": "python"})
    print(f"Search for 'python': {search}")

async def test_deep_research():
    """Test deep research agent"""
    query = "What is artificial intelligence?"
    print(f"Running deep research on: {query}")
    result = await run_deep_research(query)
    print(f"Deep research result: {result[:500]}...")  # Show first 500 chars

async def run_all_tests():
    """Run all test scenarios"""
    print("=== Testing LLM ===")
    await test_llm()
    
    print("\n=== Testing MCP Tools ===")
    await test_mcp_tools()
    
    print("\n=== Testing Deep Research ===")
    await test_deep_research()

if __name__ == "__main__":
    asyncio.run(run_all_tests())
```
