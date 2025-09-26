# DeepAgents Research System

This document explains the DeepAgents research system implemented in the chatbot.

## Overview

The DeepAgents research system provides advanced research capabilities by:
1. Using a multi-agent architecture with specialized sub-agents
2. Integrating with Tavily for web search capabilities
3. Following a structured research methodology
4. Generating comprehensive research reports

## Components

### 1. Main Research Agent

```python
agent = create_deep_agent(
    tools=[internet_search],
    instructions=research_instructions,
    model=llm,
    subagents=[analysis_sub_agent, verification_sub_agent]
)
```

The main agent:
- Coordinates the overall research process
- Delegates specialized tasks to sub-agents
- Synthesizes findings into a final report
- Uses Tavily search for information gathering

### 2. Analysis Sub-Agent

```python
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
```

This sub-agent specializes in:
- Breaking down complex topics
- Identifying patterns and relationships
- Synthesizing information from multiple sources

### 3. Verification Sub-Agent

```python
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
```

This sub-agent specializes in:
- Fact-checking and verification
- Assessing source credibility
- Identifying inconsistencies or controversies

### 4. Tavily Search Integration

```python
def internet_search(
    query: str,
    max_results: int = 5,
    topic: Literal["general", "news", "finance"] = "general",
    include_raw_content: bool = False,
):
    """Run a web search using Tavily API"""
    return tavily_client.search(
        query,
        max_results=max_results,
        include_raw_content=include_raw_content,
        topic=topic,
    )
```

The Tavily integration provides:
- Web search capabilities
- Filtering by topic (general, news, finance)
- Control over result count and content depth

## Research Process

1. **Query Analysis**: The main agent analyzes the research query
2. **Research Planning**: The agent plans the research approach
3. **Information Gathering**: The agent uses Tavily to gather information
4. **Analysis**: The analysis sub-agent processes the information
5. **Verification**: The verification sub-agent checks facts and sources
6. **Report Generation**: The main agent creates a structured report

## Report Structure

The research reports follow a consistent structure:
1. **Executive Summary**: Brief overview of findings
2. **Key Findings**: Main points and insights
3. **Detailed Analysis**: Supporting evidence and reasoning
4. **Sources**: List of sources used
5. **Limitations**: Any gaps or caveats
6. **Recommendations**: Further exploration suggestions

## Usage

To use the deep research mode:
1. Type `/research` to toggle research mode
2. Ask a research question
3. Wait for the comprehensive report

Example:
```
/research
What are the latest developments in quantum computing?
```

## Technical Implementation

The DeepAgents system is built on:
- LangChain's DeepAgents framework
- LangGraph for agent orchestration
- Context isolation between agents
- Tavily API for web search

The system ensures each agent maintains its specialized focus while contributing to the overall research goal.
