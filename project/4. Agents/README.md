# Phase 4: Multi-Agent Systems 🤖

*Learn agent orchestration patterns with LangGraph or AutoGen*

## What You're Building

A **standalone multi-agent system** to explore:
- **Agent Orchestration**: How multiple AI agents work together
- **Framework Patterns**: Choose your agent framework (LangGraph or AutoGen)
- **Tool Usage**: How agents decide when and how to use different tools
- **Research Workflows**: Multi-step reasoning and information synthesis

**📚 Implementation Details:** 
- **Track A (LangGraph)**: See [`LANGGRAPH_ARCHITECTURE.md`](LANGGRAPH_ARCHITECTURE.md) for detailed workflow implementation
- **Track B (AutoGen)**: See [`AUTOGEN_ARCHITECTURE.md`](AUTOGEN_ARCHITECTURE.md) for detailed conversation-based implementation

## Your Mission: 2-Week Agent Exploration

Learn multi-agent patterns by building a research system that demonstrates key agent concepts. This is a **separate learning project** - your Phase 1-3 system remains complete and valuable as-is.

### **Week 1: Core Agent Framework**

#### **Day 1-3: Framework Setup & Basic Agent Interaction**

**Your Mission:** Get your chosen framework working with basic agent interactions.

### **Track A: LangGraph Implementation**

**What to Build:**
- **Router Node** - Intent classification using conditional edges
- **Simple Agent Node** - Basic response generation  
- **State Schema** - Define AgentState with messages and routing decisions
- **Basic Graph** - Connect router → agent → end

**Key Learning:**
```python
def router_node(state: AgentState):
    last_message = state["messages"][-1]
    if "research" in last_message.content.lower():
        return {"routing_decision": "research"}
    return {"routing_decision": "simple"}

workflow.add_conditional_edges("router", 
    lambda state: state["routing_decision"],
    {"simple": "simple_agent", "research": "research_planner"}
)
```

### **Track B: AutoGen Implementation**

**What to Build:**
- **User Proxy Agent** - Handles user interaction and task initiation
- **Research Assistant** - Primary research agent with web search tools
- **Quality Critic** - Reviews and improves research outputs
- **Group Chat** - Coordinate multi-agent conversations

**Key Learning:**
```python
user_proxy = UserProxyAgent("user_proxy", 
    human_input_mode="NEVER",
    code_execution_config={"use_docker": False}
)

researcher = AssistantAgent("researcher",
    system_message="You research topics thoroughly using available tools"
)

critic = AssistantAgent("critic", 
    system_message="You review research and suggest improvements"
)
```

**🎮 Side Quest:** Experiment with different routing prompts, add confidence scoring for routing decisions.

**⚠️ Gotcha:** Router needs to be fast (< 1 second) and accurate. Test with diverse query types.

#### **Day 4-7: Multi-Agent Research Workflow**

**Your Mission:** Build the complete research workflow with multiple agents.

### **Track A: LangGraph Multi-Agent Chain**

**What to Build:**
- **Research Planner Node** - Breaks down complex topics
- **Research Gatherer Node** - Collects information using tools
- **Report Builder Node** - Synthesizes comprehensive reports
- **Tool Node** - Shared tool execution (Perplexity, knowledge base)
- **Conditional Logic** - Smart routing based on research completeness

**Key Learning:**
```python
def research_gatherer_node(state: AgentState):
    # Agent decides which tools to use
    if needs_web_research(state):
        return call_perplexity_tool(state)
    elif needs_knowledge_base(state):
        return call_kb_search_tool(state)
    else:
        return {"messages": [AIMessage("Research complete")]}

workflow.add_conditional_edges("research_gatherer",
    lambda state: "continue" if has_tool_calls(state) else "build_report"
)
```

### **Track B: AutoGen Research Team**

**What to Build:**
- **Topic Planner Agent** - Breaks down research topics
- **Knowledge Researcher Agent** - Searches knowledge bases  
- **Web Researcher Agent** - Handles web search and synthesis
- **Report Writer Agent** - Creates comprehensive markdown reports
- **Research Coordinator** - Orchestrates the research team

**Key Learning:**
```python
# Agents work together through conversation
groupchat = GroupChat(
    agents=[planner, knowledge_researcher, web_researcher, writer],
    messages=[],
    max_round=10
)

manager = GroupChatManager(groupchat=groupchat, llm_config=config)

user_proxy.initiate_chat(
    manager,
    message="Research the evolution of transformer architectures"
)
```

**Research Tools:**
- `research_topic_breakdown` - Creates structured research plan
- `search_knowledge_base` - Returns [KB-X] sources from your Phase 1 system
- `perplexity_search` - Returns [WEB-X] sources with AI-powered deep research
- `create_report_outline` - Structures comprehensive reports

**🎮 Side Quest:** Add research quality scoring, implement iterative refinement loops.

**⚠️ Gotcha:** Multi-agent workflows can get complex fast. Start simple and add complexity incrementally.

### **Week 2: Integration & Production**

#### **Day 8-10: Agent Patterns & Tools**
**Your Mission:** Learn core agent patterns and tool usage.

**What to Build:**
- **Agent Communication** - How agents pass information between each other
- **Tool Selection Logic** - How agents decide which tools to use when
- **Error Handling** - How agents recover from failures and continue workflows
- **State Persistence** - How to maintain context across complex agent interactions

**Key Learning Goals:**
- Understand when to use multiple agents vs single agent
- Learn your chosen framework's coordination patterns
- Practice agent prompt engineering and role design
- Build robust tool usage and error handling patterns

**🎯 Recommended Approach**: Pick ONE framework and go deep rather than trying both. Each teaches valuable but different agent patterns.

## 🔧 **Choose Your Agent Framework**

**Pick ONE framework and follow its implementation track:**

### **Track A: LangGraph** (Explicit Workflows)
**Best for**: Learning state management, complex routing, visual workflows

**Key Concepts**: Nodes, edges, state schemas, conditional routing
```python
# LangGraph approach: Explicit workflow graph
workflow = StateGraph(AgentState)
workflow.add_node("planner", research_planner_node)  
workflow.add_node("gatherer", research_gatherer_node)
workflow.add_conditional_edges("gatherer", should_continue)
```

### **Track B: AutoGen** (Conversational Agents)  
**Best for**: Learning agent conversations, role-based coordination, natural interactions

**Key Concepts**: Agent roles, group chats, conversation patterns
```python
# AutoGen approach: Agents converse to solve problems
researcher = AssistantAgent("Researcher", llm_config=config)
writer = AssistantAgent("Writer", llm_config=config) 
critic = AssistantAgent("Critic", llm_config=config)

user_proxy.initiate_chat(researcher, message="Research RAG systems")
```

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

**Choose Your Tech Stack:**

### **Track A: LangGraph Stack**
- **LangChain** - Agent definitions, tool integration, embeddings
- **LangGraph** - Multi-agent workflow orchestration and state management
- **Langfuse** - Complete observability and tracing

### **Track B: AutoGen Stack**  
- **AutoGen** - Multi-agent conversation framework
- **OpenAI/Anthropic** - LLM providers (no LangChain required)
- **Custom Tools** - Perplexity API, knowledge base search
- **Optional Langfuse** - Observability (requires custom integration)

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

### **Track A: LangGraph Setup**
```bash
# Core LLM APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key  # Optional

# LangChain/LangGraph
LANGCHAIN_API_KEY=your_langsmith_api_key  # For tracing
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT="multi-agent-research"

# Research APIs
PERPLEXITY_API_KEY=your_perplexity_key
# Add your knowledge base API keys here

# Observability  
LANGFUSE_SECRET_KEY=your_langfuse_secret  # Alternative to LangSmith
LANGFUSE_PUBLIC_KEY=your_langfuse_public
```

### **Track B: AutoGen Setup**
```bash
# Core LLM APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key  # Optional

# Research APIs
PERPLEXITY_API_KEY=your_perplexity_key
# Add your knowledge base API keys here

# AutoGen doesn't require LangChain
# Custom observability setup if desired
LANGFUSE_SECRET_KEY=your_langfuse_secret  # Optional
LANGFUSE_PUBLIC_KEY=your_langfuse_public
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

**Learning Note:** This phase focuses on **agent concepts and patterns**. Your Phase 1-3 system is complete and valuable as-is! You can optionally connect agent learnings to your existing system, but the main goal is understanding how multi-agent systems work. 🚀
