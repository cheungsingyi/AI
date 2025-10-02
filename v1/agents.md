# agents.py - Deep Research Agent

## Description
Deep research agent with MCP tool integration, optimized for LangChain tools using DeepAgents framework.

## Code

```python
"""Deep research agent with MCP tool integration - Optimized for LangChain tools"""
import os
from modules.llm import get_llm
from dotenv import load_dotenv
from langchain_core.tools import BaseTool
from typing import List, Optional, Dict

load_dotenv()


def select_tools_by_name(all_tools: List[BaseTool], tool_names: Optional[List[str]]) -> List[BaseTool]:
    """
    Select specific tools by name from the full list.
    
    Args:
        all_tools: List of all available LangChain tools
        tool_names: List of tool names to select, or None for all tools
    
    Returns:
        List of selected LangChain tools
    """
    if not tool_names:  # If None or empty, return all
        return all_tools
    
    selected = []
    for tool in all_tools:
        if tool.name in tool_names:
            selected.append(tool)
    return selected


async def run_deep_research(
    query: str,
    langchain_tools: Optional[List[BaseTool]] = None,
    main_agent_tools: Optional[List[str]] = None,
    subagent_tools: Optional[Dict[str, List[str]]] = None
) -> str:
    """
    Run deep research with LangChain tools and optional selective tool distribution.
    
    Args:
        query: Research query
        langchain_tools: List of LangChain BaseTool instances (from MCP or other sources)
        main_agent_tools: Tool names for main agent (None = all tools)
        subagent_tools: Dict mapping subagent name -> tool names (None = all tools)
    
    Behavior:
        - No langchain_tools: Returns error message
        - langchain_tools provided, no selection: ALL tools to ALL agents
        - Selective lists: Only specified tools to each agent
    
    Returns:
        Research report as string
    """
    try:
        # Use async version for async tools (MCP tools are async!)
        from deepagents import async_create_deep_agent
        from loguru import logger
        
        if not langchain_tools:
            return "Error: No tools available. Please ensure MCP servers are running and connected."
        
        # Define research instructions for the agent
        research_instructions = """You are an expert researcher. Your job is to conduct thorough research on the given query and then write a polished, comprehensive report.

You have access to various tools including web search, calculations, weather data, and more. Use these tools strategically to gather comprehensive information.

## Research Process:
1. **Analyze the query** - Understand what specific information is needed
2. **Plan your research** - Identify key aspects to investigate
3. **Gather information** - Use available tools to find relevant data
4. **Synthesize findings** - Combine information from multiple sources
5. **Write comprehensive report** - Structure your response with clear sections

## Response Structure:
Please structure your response with:
1. **Executive Summary** - Brief overview of findings
2. **Key Findings** - Main points and insights
3. **Detailed Analysis** - Supporting evidence and reasoning
4. **Sources** - List of sources/tools used
5. **Limitations** - Any gaps or caveats
6. **Recommendations** - Further exploration suggestions

Be thorough but concise. Use clear headings and bullet points where appropriate."""

        # Get LLM instance
        llm = get_llm()
        
        # Prepare main agent tools (already LangChain tools!)
        selected_for_main = select_tools_by_name(langchain_tools, main_agent_tools)
        logger.info(f"Selected {len(selected_for_main)} tools for main agent: {[t.name for t in selected_for_main]}")
        
        # Ensure unique tools by name (avoid duplicates that cause hashability issues)
        seen_names = set()
        main_tools = []
        for tool in selected_for_main:
            if tool.name not in seen_names:
                main_tools.append(tool)
                seen_names.add(tool.name)
        
        logger.info(f"Using {len(main_tools)} unique LangChain tools for main agent")
        logger.info(f"Tool names: {[t.name for t in main_tools]}")
        logger.info(f"Tool types: {[type(t).__name__ for t in main_tools]}")
        
        # Prepare sub-agent configurations
        analysis_sub_agent = {
            "name": "analysis-agent",
            "description": "Used to analyze and synthesize complex information",
            "prompt": """You are a specialized analysis agent. Your role is to:
            - Break down complex topics into understandable components
            - Identify key patterns and relationships
            - Provide critical analysis of information
            - Synthesize findings from multiple sources
            Focus on clarity, accuracy, and comprehensive understanding."""
        }
        
        verification_sub_agent = {
            "name": "verification-agent",
            "description": "Used to verify facts and cross-reference information",
            "prompt": """You are a fact-checking and verification specialist. Your role is to:
            - Verify claims against multiple sources
            - Identify credible vs unreliable information
            - Cross-reference facts across different sources
            - Flag any inconsistencies or controversies
            Ensure all information presented is accurate and well-substantiated."""
        }
        
        # Assign tools to sub-agents (ensure uniqueness)
        def make_unique_tools(tools_list):
            """Ensure tools are unique by name"""
            seen = set()
            unique = []
            for t in tools_list:
                if t.name not in seen:
                    unique.append(t)
                    seen.add(t.name)
            return unique
        
        # For analysis agent
        analysis_tool_names = None
        if subagent_tools and "analysis-agent" in subagent_tools:
            analysis_tool_names = subagent_tools["analysis-agent"]
        
        selected = select_tools_by_name(langchain_tools, analysis_tool_names)
        analysis_sub_agent["tools"] = make_unique_tools(selected)
        
        # For verification agent
        verification_tool_names = None
        if subagent_tools and "verification-agent" in subagent_tools:
            verification_tool_names = subagent_tools["verification-agent"]
        
        selected = select_tools_by_name(langchain_tools, verification_tool_names)
        verification_sub_agent["tools"] = make_unique_tools(selected)
        
        # Create agent with distributed tools
        logger.info(f"Creating deep agent with {len(main_tools)} tools")
        logger.info(f"Sub-agents: analysis ({len(analysis_sub_agent.get('tools', []))} tools), verification ({len(verification_sub_agent.get('tools', []))} tools)")
        
        # Create async agent for async tools (MCP tools are async!)
        agent = async_create_deep_agent(
            tools=main_tools,
            instructions=research_instructions,
            model=llm,
            subagents=[analysis_sub_agent, verification_sub_agent]
            # Note: builtin_tools parameter removed in newer versions
        )
        
        logger.info("Async agent created successfully, invoking with query...")
        logger.info(f"Query: {query}")

        # Invoke the agent with the research query (use ainvoke for async)
        try:
            result = await agent.ainvoke({
                "messages": [{"role": "user", "content": query}]
            })
            logger.info(f"Agent invocation complete, result type: {type(result)}")
        except Exception as e:
            logger.error(f"Error during agent invocation: {e}")
            logger.exception(e)
            raise

        # Extract the final response from the agent result
        if "messages" in result and result["messages"]:
            # Find the last AI message (the final response after tool usage)
            for message in reversed(result["messages"]):
                if hasattr(message, 'type') and message.type == 'ai':
                    if hasattr(message, 'content') and message.content and not message.content.startswith('{'):
                        return message.content
                elif hasattr(message, 'content') and message.content and len(message.content) > 100:
                    # Fallback: any substantial message that looks like a response
                    return message.content

            # If no suitable message found, return the last message
            final_message = result["messages"][-1]
            if hasattr(final_message, 'content'):
                return final_message.content
            else:
                return str(final_message)
        else:
            return "Research completed but no response generated."

    except Exception as e:
        return f"Error in deep research: {str(e)}. Query was: {query}"
```

## Key Components

### 1. `select_tools_by_name(all_tools, tool_names)`
- Filters tools by name
- Returns all tools if `tool_names` is None
- Enables selective tool distribution

### 2. `run_deep_research(query, langchain_tools, ...)`
- Main research function
- Uses DeepAgents framework
- Supports tool distribution to sub-agents

### 3. Agent Architecture
- **Main Agent**: Coordinates research
- **Analysis Sub-Agent**: Analyzes and synthesizes information
- **Verification Sub-Agent**: Fact-checks and verifies

### 4. Tool Distribution
- Main agent gets selected tools
- Sub-agents can have different tool sets
- Automatic deduplication

## DeepAgents Integration

### Agent Creation
```python
agent = async_create_deep_agent(
    tools=main_tools,              # LangChain tools
    instructions=research_instructions,
    model=llm,                     # LangChain LLM
    subagents=[...]                # Sub-agent configs
)
```

### Execution
```python
result = await agent.ainvoke({
    "messages": [{"role": "user", "content": query}]
})
```

## Research Process

1. **Query Analysis** - Understand information needs
2. **Planning** - Identify research aspects
3. **Tool Usage** - Gather data using available tools
4. **Synthesis** - Combine findings
5. **Report Generation** - Structured comprehensive report

## Output Structure

1. **Executive Summary**
2. **Key Findings**
3. **Detailed Analysis**
4. **Sources**
5. **Limitations**
6. **Recommendations**

## Features
- ✅ Async tool support (MCP compatible)
- ✅ Multi-agent architecture
- ✅ Selective tool distribution
- ✅ Automatic tool deduplication
- ✅ Comprehensive error handling
- ✅ Structured research output
- ✅ DeepAgents 0.0.10+ compatible
