# mcp_servers/test_http_server.py - HTTP MCP Server

```python
#!/usr/bin/env python3
"""Test MCP Server - HTTP Type"""

from fastapi import FastAPI, HTTPException
from fastmcp import FastMCP
import uvicorn

app = FastAPI()
server = FastMCP("test-http-server")

@server.tool()
def translate_text(text: str, target_lang: str) -> str:
    """Translate text to the target language"""
    translations = {
        "hello": {
            "spanish": "hola",
            "french": "bonjour",
            "german": "hallo",
            "italian": "ciao",
            "japanese": "こんにちは",
            "chinese": "你好"
        },
        "goodbye": {
            "spanish": "adiós",
            "french": "au revoir",
            "german": "auf wiedersehen",
            "italian": "arrivederci",
            "japanese": "さようなら",
            "chinese": "再见"
        },
        "thank you": {
            "spanish": "gracias",
            "french": "merci",
            "german": "danke",
            "italian": "grazie",
            "japanese": "ありがとう",
            "chinese": "谢谢"
        }
    }
    
    text_lower = text.lower()
    target_lang_lower = target_lang.lower()
    
    if text_lower in translations and target_lang_lower in translations[text_lower]:
        return translations[text_lower][target_lang_lower]
    else:
        return f"Translation not available for '{text}' to {target_lang}"

@server.tool()
def get_news(category: str) -> str:
    """Get latest news by category"""
    news = {
        "technology": [
            "New AI model breaks performance records",
            "Tech company launches revolutionary smartphone",
            "Quantum computing milestone achieved"
        ],
        "business": [
            "Stock markets reach all-time high",
            "Major merger announced between tech giants",
            "New economic policy introduced"
        ],
        "science": [
            "Scientists discover new species in deep ocean",
            "Breakthrough in renewable energy storage",
            "Mars mission reveals surprising findings"
        ],
        "health": [
            "New medical treatment shows promising results",
            "Study reveals benefits of Mediterranean diet",
            "Health authorities issue new guidelines"
        ]
    }
    
    category_lower = category.lower()
    if category_lower in news:
        return "\n".join(news[category_lower])
    else:
        return f"No news available for category: {category}"

@server.tool()
def stream_analysis(query: str) -> str:
    """Analyze streaming data for patterns"""
    analyses = {
        "user engagement": "User engagement shows 23% increase in active sessions with peak times between 7-9 PM",
        "performance": "System performance metrics indicate 99.8% uptime with average response time of 120ms",
        "errors": "Error analysis shows 404 errors decreased by 15% while API timeout errors increased by 3%",
        "traffic": "Traffic patterns show 65% mobile, 30% desktop, and 5% tablet usage",
        "conversion": "Conversion rates improved by 2.7% after recent UI updates"
    }
    
    query_lower = query.lower()
    results = []
    
    for key, value in analyses.items():
        if query_lower in key or query_lower in value.lower():
            results.append(f"{key}: {value}")
    
    if results:
        return "\n".join(results)
    else:
        return f"No analysis available for query: {query}"

# Register FastMCP with FastAPI
server.register_fastapi(app)

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8002)
```
