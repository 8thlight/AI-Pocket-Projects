# Phase 4: Multi-Agent Research System 🤖

*Orchestrating specialized AI agents for intelligent research workflows*

## What You're Building

A **multi-agent research system** that intelligently routes user requests between:
- **Simple RAG**: Fast answers with automatic knowledge retrieval 
- **Deep Research**: Comprehensive reports with multi-source synthesis
- **Intelligent Routing**: Automatic intent classification
- **Perfect Citations**: Source attribution across all workflows

**📚 System Architecture:** See [`ARCHITECTURE.md`](ARCHITECTURE.md) for complete technical details

## Your Mission: 2-Week Agent Sprint

Transform your RAG system into an intelligent multi-agent platform that can handle everything from quick questions to comprehensive research reports.

### **Week 1: Core Agent Framework**

#### **Day 1-3: Router & Simple Agent Path**
**Your Mission:** Build the intelligent routing system and fast-response pipeline.

**What to Build:**
- **Router Agent** - Intent classification (GPT-4.1 for speed)
- **Simple RAG Node** - Automatic knowledge base retrieval (from Phase 1)
- **Simple Agent** - Concise answers with citations
- **LangGraph workflow** connecting router → simple path → end

**Routing Logic:**
- Detect research keywords: "comprehensive", "report", "deep dive", "research" → RESEARCH
- Detect simple patterns: "What is...", "Define...", "Who..." → SIMPLE
- Default to SIMPLE for fast responses

**🎮 Side Quest:** Experiment with different routing prompts, add confidence scoring for routing decisions.

**⚠️ Gotcha:** Router needs to be fast (< 1 second) and accurate. Test with diverse query types.

#### **Day 4-7: Research Agent Path**
**Your Mission:** Build the multi-agent research workflow for comprehensive reports.

**What to Build:**
- **Research Planner Agent** - Strategic topic breakdown
- **Research Gatherer Agent** - Multi-source information collection  
- **Report Builder Agent** - Comprehensive markdown report synthesis
- **Tool Node** - Shared tool execution for all agents
- **LangGraph workflow** connecting research path with proper state management

**Research Tools:**
- `research_topic_breakdown` - Creates structured research plan
- `search_knowledge_base` - Returns [KB-X] sources from your Phase 1 system
- `perplexity_search` - Returns [WEB-X] sources with AI-powered deep research
- `create_report_outline` - Structures comprehensive reports

**🎮 Side Quest:** Add research quality scoring, implement iterative refinement loops.

**⚠️ Gotcha:** Multi-agent workflows can get complex fast. Start simple and add complexity incrementally.

### **Week 2: Integration & Production**

#### **Day 8-10: Phase Integration**
**Your Mission:** Connect with your existing RAG, Voice, and MCP systems.

**What to Build:**
- **RAG Integration** - Use your Phase 1 Pinecone/vector database for `search_knowledge_base`
- **Perplexity Integration** - Connect Perplexity API for AI-powered deep research
- **Voice Integration** - Adapt agents for voice interaction from Phase 2
- **Unified State Management** - Consistent state across all phases

**Integration Points:**
- Simple path uses existing RAG retrieval automatically
- Research path uses RAG tools + Perplexity deep research
- Voice interface can trigger both simple and research workflows
- All workflows maintain perfect citation compliance

**🎮 Side Quest:** Add conversation memory for multi-turn research, implement research session persistence.

**⚠️ Gotcha:** Integration bugs are subtle. Test each integration point thoroughly before moving on.

#### **Day 11-12: Advanced Agent Features**
**Your Mission:** Add sophisticated agent behaviors and decision-making.

**What to Build:**
- **Dynamic Tool Selection** - Agents choose between local and web sources intelligently
- **Research Quality Assessment** - Agents evaluate their own research completeness
- **Iterative Refinement** - Agents can loop back to gather more information
- **Source Quality Scoring** - Prefer authoritative sources and recent information
- **Research Session Management** - Handle long-running research workflows

**Advanced Behaviors:**
- Research Gatherer signals "RESEARCH COMPLETE" when sufficient sources collected
- Report Builder can request additional information if gaps found
- Agents track source diversity (don't over-rely on single sources)
- Intelligent caching to avoid duplicate searches

**🎮 Side Quest:** Add collaborative research (multiple gatherer agents), implement research templates for common topics.

**⚠️ Gotcha:** Agent loops can become infinite. Always implement clear termination conditions.

#### **Day 13-14: Observability & Production Polish**
**Your Mission:** Make the system production-ready with comprehensive monitoring.

**What to Build:**
- **Langfuse Integration** - Complete tracing of all agent interactions (consistent with Phases 1-3)
- **Performance Monitoring** - Track latency, costs, and success rates per agent
- **Error Handling** - Graceful degradation and retry logic
- **Cost Optimization** - Smart model selection (GPT-4 for research, GPT-3.5 for routing)
- **API Rate Management** - Proper queuing and backoff strategies

**Production Features:**
- **Streaming Responses** - Real-time output for long research workflows
- **Progress Indicators** - Show users which research phase is active
- **Cancellation Support** - Allow users to stop long-running research
- **Session Persistence** - Save and resume research workflows
- **Multi-tenant Support** - Separate user research sessions

**🎮 Side Quest:** Add research analytics dashboard, implement A/B testing for different agent prompts.

**⚠️ Gotcha:** Agent workflows generate lots of LLM calls. Monitor costs and implement budget controls.

## Tech Stack

**Core Framework:**
- **LangChain** - Agent definitions, tool integration, embeddings
- **LangGraph** - Multi-agent workflow orchestration and state management
- **Langfuse** - Complete observability and tracing (100% coverage)

**LLM Selection:**
- **OpenAI GPT-4** - Research agents (quality for complex reasoning)
- **OpenAI GPT-3.5-turbo** - Router agent (speed and cost efficiency)
- Alternative: **Claude 3.5 Sonnet** for research agents (excellent reasoning)

**Integration:**
- **Phase 1 RAG** - Pinecone vector database, OpenAI embeddings
- **Phase 2 Voice** - Streaming responses, conversation state
- **Phase 3 MCP** - Perplexity API integration for deep web research
- **FastAPI** - REST API with streaming support

**State Management:**
- **AgentState Schema** - Messages, sources, routing decisions
- **Redis** - Session persistence and caching (if using voice)
- **Async Processing** - Handle long-running research workflows

## Environment Variables

```bash
# Core LLM APIs
OPENAI_API_KEY=your_openai_key

# Observability (Consistent with Phases 1-3)
LANGFUSE_SECRET_KEY=your_langfuse_secret
LANGFUSE_PUBLIC_KEY=your_langfuse_public
LANGFUSE_HOST=https://cloud.langfuse.com

# Optional: LangSmith (Alternative to Langfuse)
# LANGCHAIN_API_KEY=your_langsmith_api_key
# LANGCHAIN_TRACING_V2=true
# LANGCHAIN_PROJECT="multi-agent-research"

# From Previous Phases
PINECONE_API_KEY=your_pinecone_key  # Phase 1 RAG
PERPLEXITY_API_KEY=your_perplexity_key  # Phase 3 Deep Research

# Optional
ANTHROPIC_API_KEY=your_anthropic_key  # If using Claude
REDIS_URL=redis://localhost:6379     # Session persistence
```

## Success Criteria

By the end of 2 weeks, you should have:
- ✅ **Intelligent Routing** - Automatic classification between simple/research modes
- ✅ **Multi-Agent Workflow** - Planner → Gatherer → Builder working seamlessly
- ✅ **Perfect Citations** - All sources tracked with [KB-X] and [WEB-X] format
- ✅ **Phase Integration** - RAG, Voice, and MCP systems working together
- ✅ **Complete Observability** - Every agent decision traced in Langfuse
- ✅ **Production Ready** - Error handling, rate limiting, cost controls
- ✅ **Quality Research** - Comprehensive reports with 5+ sources minimum

## Example Usage

**Simple Query:**
```
User: "What is retrieval-augmented generation?"
→ Router: SIMPLE
→ Simple RAG: Retrieves 3 KB sources
→ Simple Agent: Answers with citations [KB-1], [KB-2], [KB-3]
Time: ~3 seconds, Cost: ~$0.02
```

**Research Query:**
```
User: "Write a comprehensive report on the evolution of transformer architectures"
→ Router: RESEARCH  
→ Research Planner: Creates structured plan with 5 subtopics
→ Research Gatherer: Searches KB (5 sources) + Perplexity (5 sources)
→ Report Builder: Generates 2500-word markdown report
→ Output: Complete report with [KB-1] to [KB-5] + [WEB-1] to [WEB-5]
Time: ~30 seconds, Cost: ~$0.15
```

## Advanced Challenges

**🚀 Multi-Modal Research:** Add support for image analysis and document understanding
**🚀 Collaborative Agents:** Multiple research gatherers working on different subtopics in parallel
**🚀 Research Templates:** Pre-built workflows for common research types (market analysis, technical deep-dives)
**🚀 Incremental Refinement:** User feedback loops to improve research quality
**🚀 Export Formats:** Generate PDF, Word, or presentation formats from research
**🚀 Perplexity Pro Features:** Leverage advanced reasoning models and real-time search capabilities

## Common Gotchas

**Agent Loops:** Research agents can get stuck in "need more information" loops. Implement clear termination conditions.

**State Explosion:** LangGraph state can grow large with multiple agents. Use state reducers and cleanup functions.

**Cost Control:** Research workflows can be expensive with multiple LLM calls. Monitor token usage and implement budgets.

**Citation Integrity:** With multiple sources, citation tracking gets complex. Validate all citations reference real sources.

**Tool Conflicts:** Different agents using same tools can cause conflicts. Design tools to be stateless and idempotent.

**Integration Testing:** Multi-phase integration is complex. Test each connection point thoroughly.

**Observability Overhead:** Complete tracing generates lots of data. Balance visibility with performance.

Remember: Multi-agent systems are powerful but complex. Start with simple workflows and add sophistication incrementally. Focus on reliability and clear agent responsibilities over fancy features.


---

**Integration Note:** This phase brings together everything from Phases 1-3 into a cohesive multi-agent system. You'll have a complete AI platform capable of handling any query from simple questions to comprehensive research reports! 🚀
