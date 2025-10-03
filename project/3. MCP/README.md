# Phase 3: Web Intelligence (MCP) 🌐

*Adding real-time web search and scraping to your AI platform*

## What You're Building

A Model Context Protocol (MCP) server that extends your AI system with:
- Real-time web search capabilities with source URL tracking
- Intelligent web page scraping and content extraction
- **Mandatory source citations** for all web-sourced information
- Integration with your existing RAG + Voice pipeline
- Content cleaning and markdown conversion for consistent formatting
- Rate limiting and caching for efficient web operations
- Tool selection intelligence - when to search web vs. local documents

**📚 Working Example:** [WebCat](https://github.com/Kode-Rex/webcat) - Full MCP implementation for web search and scraping

**📋 Official MCP Documentation:** [Model Context Protocol](https://modelcontextprotocol.io/) - Complete specification and standards

## Your Mission: 2-Week Web Intelligence Sprint

Transform your voice assistant into a web-connected intelligence system that can answer questions about current events, recent developments, and real-time information.

### **Week 1: Core MCP Foundation**

#### **Day 1-2: MCP Server Setup**
**Your Mission:** Build the foundational MCP server with proper tool registration.

**What to Build:**
- **MCP server** using FastMCP (Python) or custom Node.js implementation
- Tool registration for `web_search` and `scrape_url` functions
- Basic HTTP client setup with proper headers and user agents
- Error handling and timeout management for web requests
- Integration testing with your existing RAG system

**🎮 Side Quest:** Add tool descriptions and parameter validation, experiment with different MCP client integrations.

**⚠️ Gotcha:** MCP servers need proper tool schemas - the AI needs to understand when and how to use each tool. Test tool calling extensively.

#### **Day 3-4: Web Search Implementation**
**Your Mission:** Implement intelligent web search with multiple search engines.

**What to Build:**
- **Serp API** as primary search engine (Google results without API limits)
- **DuckDuckGo** as fallback search engine (free, privacy-focused)
- Search query optimization and refinement
- Result ranking and filtering for relevance
- **Source URL preservation** - every search result must include origin URL
- Search result caching to avoid duplicate queries
- Automatic fallback when primary search fails

**🎮 Side Quest:** Try semantic search result re-ranking, implement search query expansion.

**⚠️ Gotcha:** Serp API has generous limits but costs per search. DuckDuckGo is free but has basic rate limiting. Implement caching aggressively and smart fallback logic.

#### **Day 5-7: Web Scraping & Content Extraction**
**Your Mission:** Build robust web scraping that extracts clean, useful content.

**What to Build:**
- **Web scraping engine** using Trafilatura, Playwright, or Scrapy
- Content extraction focusing on main article text
- HTML to Markdown conversion for consistent formatting
- Image and media handling (descriptions, alt text)
- JavaScript-rendered content support for modern websites
- **Metadata preservation** - title, author, publish date, source URL

**🎮 Side Quest:** Add PDF extraction, implement content summarization for long articles.

**⚠️ Gotcha:** Modern websites are complex - many require JavaScript rendering. Start with static scraping, add dynamic rendering as needed.

### **Week 2: Integration & Intelligence**

#### **Day 8-10: RAG + Web Integration**
**Your Mission:** Intelligently combine local documents with web search results.

**What to Build:**
- **Hybrid search strategy** - paring local docs with web search tooling
- **Source attribution mixing** - clearly distinguish local vs. web sources
- Content freshness detection - prefer recent web content for time-sensitive queries

**🎮 Side Quest:** Implement source credibility scoring, add fact-checking across multiple sources.

**⚠️ Gotcha:** Users expect the AI to know when information might be outdated. Always indicate source dates and freshness.

#### **Day 11-12: Voice Integration & Citations**
**Your Mission:** Make web search results work seamlessly in voice conversations.

**What to Build:**
- **Spoken citations** for web sources - "According to a recent article from TechCrunch..."
- URL shortening or QR code generation for easy access
- Voice-friendly summaries of web content
- Follow-up question handling - "tell me more about that article"
- **Real-time information** integration - stock prices, weather, news

**🎮 Side Quest:** Add voice commands like "search the web for X" or "what's the latest on Y".

**⚠️ Gotcha:** URLs are terrible in voice interfaces. Focus on source credibility and provide alternative access methods.

#### **Day 13-14: Performance & Production**
**Your Mission:** Optimize for speed, reliability, and cost management.

**What to Build:**
- **Intelligent caching** - cache search results and scraped content
- Rate limiting and request queuing for API management
- Content freshness policies - when to refresh cached content
- Error recovery and fallback strategies
- Performance monitoring and cost tracking
- **Production deployment** with proper security and scaling

**🎮 Side Quest:** Add content archiving, implement distributed caching with Redis.

**⚠️ Gotcha:** Web scraping can be legally complex. Respect robots.txt, implement proper delays, and consider terms of service.

## Tech Stack Recommendations

**Search Engines:**
- **SerpAPI** (Google search results without Google API limits)
- **DuckDuckGo API** (free fallback, privacy-focused)
- Alternative: Tavily Search API (AI-optimized results)
- Alternative: Bing Web Search API (Microsoft's search engine)

**Web Scraping:**
- **Trafilatura** (best-in-class content extraction, handles boilerplate removal)
- **Playwright** (handles JavaScript, multiple browsers)
- Scrapy (full-featured scraping framework)
- Selenium (fallback for complex sites)

**Content Processing:**
- **html2text** or **markdownify** for HTML to Markdown conversion
- **newspaper3k** for article extraction
- **readability-lxml** for content cleaning
- **PyPDF2** or **pdfplumber** for PDF processing

**MCP Framework:**
- **Official MCP SDK** (Python or TypeScript) - See [MCP Documentation](https://modelcontextprotocol.io/docs)
- **FastMCP** (Python) - Dedicated MCP server framework
- **Node.js MCP implementation** - Manual MCP specification implementation
- WebSocket support for real-time communication
- JSON-RPC for MCP protocol compliance

**Caching & Storage:**
- **Redis** for search result and content caching
- SQLite for metadata and URL tracking

## Environment Variables

```bash
# Search APIs
SERP_API_KEY=your_serp_api_key
# DuckDuckGo is free - no API key needed

# Optional alternatives
TAVILY_API_KEY=your_tavily_key
BING_SEARCH_API_KEY=your_bing_key

# Web Scraping
USER_AGENT="YourBot/1.0 (contact@yoursite.com)"
SCRAPING_DELAY=1  # seconds between requests
MAX_CONTENT_LENGTH=50000  # characters

# MCP Server
MCP_SERVER_PORT=3001
MCP_SERVER_HOST=localhost

# Caching
REDIS_URL=redis://localhost:6379
CACHE_TTL=3600  # 1 hour default

# From previous phases
OPENAI_API_KEY=your_openai_key
LANGFUSE_SECRET_KEY=your_langfuse_secret
LANGFUSE_PUBLIC_KEY=your_langfuse_public
```

## MCP Tool Definitions

Your MCP server should expose these tools:

**web_search(query: str, num_results: int = 5)**
- Searches the web for current information
- Returns results with titles, snippets, and source URLs
- Automatically filters for relevance and recency

**scrape_url(url: str, extract_main_content: bool = True)**
- Scrapes content from a specific URL
- Extracts main article text and metadata
- Returns clean markdown with preserved source attribution

**search_and_scrape(query: str, max_sources: int = 3)**
- Combined search and scrape operation
- Searches web, then scrapes top results
- Returns comprehensive information with full source citations

## Success Criteria

By the end of 2 weeks, you should have:
- ✅ Working MCP server with web search and scraping tools
- ✅ Seamless integration with your RAG + Voice system
- ✅ **Perfect source attribution** for all web-sourced information
- ✅ Intelligent decision-making between local docs and web search
- ✅ Voice-friendly presentation of web search results
- ✅ Production-ready caching and rate limiting
- ✅ Cost-effective API usage with proper monitoring

## Advanced Challenges

**🚀 Multi-Source Verification:** Cross-reference information across multiple web sources
**🚀 Real-Time Monitoring:** Track breaking news and trending topics
**🚀 Content Summarization:** Automatically summarize long articles for voice consumption
**🚀 Fact Checking:** Compare web sources against known reliable information
**🚀 Visual Content:** Extract and describe images, charts, and infographics

## Common Gotchas

**Rate Limiting Hell:** Every search API has different limits. Monitor usage religiously and implement exponential backoff.

**JavaScript Everywhere:** Modern websites are SPAs. Static scraping often returns empty pages - you need browser automation.

**Content Quality:** Web content is messy. Invest heavily in content cleaning and extraction - garbage in, garbage out.

**Legal Considerations:** Respect robots.txt, implement delays, and be aware of terms of service. Some sites prohibit scraping.

**Citation Complexity:** Web sources need more context than local docs - include publication date, author, and credibility indicators.

**Cache Invalidation:** Web content changes constantly. Balance freshness with performance - news articles need frequent updates, reference materials don't.

**Cost Explosion:** Search APIs and browser automation are expensive. Cache aggressively and monitor costs closely.

**Voice URL Problem:** "According to h-t-t-p-s colon slash slash w-w-w dot..." sounds terrible. Focus on source credibility instead.

Remember: Web intelligence is about augmenting your AI's knowledge with current, accurate information while maintaining perfect source attribution. Quality and reliability matter more than speed.

---

**Congratulations!** You've built a complete AI system with knowledge retrieval, voice interaction, and web intelligence. Your AI can now access both curated knowledge and real-time information while maintaining perfect source attribution. 
