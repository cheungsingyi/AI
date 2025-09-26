# test_chatbot.py - Chatbot Tests

```python
"""Test script for the chatbot"""

import asyncio
from modules.llm import get_llm
from modules.mcp import get_all_mcp_tools, call_mcp_tool

async def test_chat():
    """Test a simple chat interaction"""
    llm = get_llm()
    
    # Simulate user message
    user_message = "Hello! Can you tell me about yourself?"
    print(f"User: {user_message}")
    
    # Get LLM response
    response = await llm.ainvoke([{"role": "user", "content": user_message}])
    print(f"AI: {response.content}")
    
    # Simulate follow-up question
    follow_up = "What can you help me with?"
    print(f"User: {follow_up}")
    
    # Get LLM response with conversation history
    response = await llm.ainvoke([
        {"role": "user", "content": user_message},
        {"role": "assistant", "content": response.content},
        {"role": "user", "content": follow_up}
    ])
    print(f"AI: {response.content}")

async def test_tool_usage():
    """Test using tools in a chat"""
    # Get available tools
    tools = await get_all_mcp_tools()
    print(f"Available tools: {[t['name'] for t in tools]}")
    
    # Test weather tool
    city = "Paris"
    print(f"User: What's the weather in {city}?")
    
    try:
        result = await call_mcp_tool("get_weather", {"city": city.lower()})
        print(f"Tool result: {result}")
        print(f"AI: Based on the weather data, it's {result} in {city}.")
    except Exception as e:
        print(f"Error using weather tool: {e}")
    
    # Test calculator tool
    expression = "15 * 7 + 22"
    print(f"User: Calculate {expression}")
    
    try:
        result = await call_mcp_tool("calculate", {"expression": expression})
        print(f"Tool result: {result}")
        print(f"AI: The result of {expression} is {result}.")
    except Exception as e:
        print(f"Error using calculator tool: {e}")

async def main():
    """Run all tests"""
    print("=== Testing Basic Chat ===")
    await test_chat()
    
    print("\n=== Testing Tool Usage ===")
    await test_tool_usage()

if __name__ == "__main__":
    asyncio.run(main())
```
