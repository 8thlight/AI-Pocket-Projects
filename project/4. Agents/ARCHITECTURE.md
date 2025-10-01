# System Architecture

## Overview

This is a **multi-agent research system** with intelligent routing. It uses LangGraph to orchestrate different workflows based on user intent.

## High-Level Flow

```
Request
   ↓
┌──────────────┐
│   Router     │
└──────┬───────┘
       │
   ┌───┴────┐
   ↓        ↓
SIMPLE   RESEARCH
   │        │
   │        ↓
   │    ┌─────────────────────┐
   │    │ 1. Research Planner │ ← Dedicated agent
   │    │    - Breaks down    │
   │    │      topic          │
   │    │    - Creates plan   │
   │    └──────┬──────────────┘
   │           ↓
   │    ┌─────────────┐
   │    │   Tools     │
   │    │   (planning)│
   │    └──────┬──────┘
   │           ↓
   │    ┌─────────────────────┐
   │    │ 2. Research Gatherer│ ← Dedicated agent
   │    │    - Searches KB    │
   │    │    - Searches web   │
   │    │    - Collects info  │
   │    └──────┬──────────────┘
   │           │
   │      ┌────┴────┐
   │      ↓         ↓
   │   More?     Ready?
   │      │         │
   │      ↓         │
   │   ┌──────┐    │
   │   │Tools │    │
   │   │(KB,  │    │
   │   │ web) │    │
   │   └───┬──┘    │
   │       │       │
   │       └───┐   │
   │           ↓   ↓
   │    ┌─────────────────────┐
   │    │ 3. Report Builder   │ ← Dedicated agent
   │    │    - Creates outline│
   │    │    - Writes report  │
   │    │    - Cites sources  │
   │    └──────┬──────────────┘
   │           ↓
   │         END
   │
   ↓
┌─────────────────┐
│  Simple RAG     │
│  (auto retrieve)│
└────────┬────────┘
         ↓
┌─────────────────┐
│  Simple Agent   │
│  (answer)       │
└────────┬────────┘
         ↓
       END
```

## Components

### 1. Router Node

**Purpose**: Intelligent request classification

**Input**: User message

**Output**: Routing decision ("simple" or "research")

**LLM**: GPT-3.5-turbo (fast, cheap)

**Logic**:
- Analyzes user's language for intent
- Detects keywords: "comprehensive", "research", "report", "deep dive" → RESEARCH
- Detects patterns: "What is...", "Define...", "Who..." → SIMPLE
- Routes to appropriate workflow

---

## Simple Path

### 2. Simple RAG Node

**Purpose**: Automatic knowledge base retrieval

**Process**:
1. Extract user query from message
2. Query Pinecone vector store (k=3 documents)
3. Retrieve document chunks with metadata (title, source, score)
4. Format context with citations
5. Inject context as system message

**Output**: State with RAG context added to messages

**No tool choice** - retrieval happens automatically

### 3. Simple Agent Node

**Purpose**: Generate concise answer with citations

**System Prompt**: "Answer using provided context, cite sources"

**Tools Available**: 
- Web search (optional, if needs current info)

**Process**:
1. Receive messages with RAG context already injected
2. Generate answer citing sources by number
3. Optionally call web search if needs latest info
4. Format response with source list

**Output**: Answer with source citations

---

## Research Path

### 4. Research Planner Agent

**Purpose**: Strategic research planning

**System Prompt**: "Break down research topics into manageable subtopics"

**Tools Available**:
- `research_topic_breakdown` - Creates structured research plan

**Process**:
1. Analyze research topic
2. Break down into subtopics
3. Identify key questions to answer
4. Create research strategy

**Output**: Research plan with subtopics and questions

**Transitions to**: Tools node → Research Gatherer

---

### 5. Research Gatherer Agent

**Purpose**: Comprehensive information collection

**System Prompt**: "Gather information from KB and web, track all sources"

**Tools Available**:
- `search_knowledge_base` - Returns [KB-X] sources
- `perplexity_search` - Returns [WEB-X] sources with AI-powered research

**Process**:
1. Use KB search for foundational concepts
2. Use Perplexity search for latest information and deep research
3. Collect sources for each subtopic in plan
4. Track all sources with citations
5. Signal completion when has sufficient sources (5+ KB, 5+ web)

**Output**: Gathered research with tracked sources

**Loop Condition**: 
- If has tool calls → back to Tools node (gather more)
- If signals "RESEARCH COMPLETE" → Report Builder

---

### 6. Report Builder Agent

**Purpose**: Synthesize findings into comprehensive report

**System Prompt**: "Write detailed MARKDOWN report with citations"

**Tools Available**:
- `create_report_outline` - Structures report sections

**Process**:
1. Create report outline with sections
2. Write comprehensive markdown report
3. Cite all sources inline using (KB-X) or (WEB-X) format
4. Include ## References section at end with:
   - Knowledge Base Sources: [KB-X] Title (file)
   - Web Sources: [WEB-X] Title (URL)

**Output**: Complete markdown research report

**Transitions to**: END

---

## Tools

### Tool Node (Shared)

**Purpose**: Execute tool calls from agents

**Available Tools**:
- `research_topic_breakdown` - Planning
- `search_knowledge_base` - KB retrieval
- `perplexity_search` - Deep web research with AI analysis
- `create_report_outline` - Structuring

**Process**:
1. Receive tool call from agent
2. Execute tool function
3. Return results to agent

**Transitions**: Back to calling agent node

---

## State Management

### AgentState Schema

```python
class AgentState(TypedDict):
    messages: Annotated[Sequence[BaseMessage], add_messages]
    sources: List[dict]
    routing_decision: str  # "simple" or "research"
```

**State Flow**:
- Router sets `routing_decision`
- Simple RAG adds `sources` from retrieval
- Messages accumulate through workflow
- Final state contains complete conversation history

---

## Decision Points

### 1. Router Decision

```python
def _determine_mode(state):
    decision = state.get("routing_decision", "simple")
    return decision  # "simple" or "research"
```

Routes to:
- `"simple"` → Simple RAG Node
- `"research"` → Research Planner Agent

### 2. Simple Agent Continuation

```python
def _should_continue(state):
    if last_message.tool_calls:
        return "continue"  # → Tools (web search)
    return "end"  # → END
```

### 3. Research Gatherer Readiness

```python
def _check_research_ready(state):
    if "RESEARCH COMPLETE" in last_message.content:
        return "build_report"  # → Report Builder
    if last_message.tool_calls:
        return "gather_more"  # → Tools
    return "build_report"  # Default: ready
```

---

## LangGraph Implementation

```python
workflow = StateGraph(AgentState)

# Nodes
workflow.add_node("router", _route_request)
workflow.add_node("simple_rag", _simple_rag)
workflow.add_node("simple_agent", _simple_agent)
workflow.add_node("research_planner", _research_planner)
workflow.add_node("research_gatherer", _research_gatherer)
workflow.add_node("report_builder", _report_builder)
workflow.add_node("tools", ToolNode(tools))

# Entry
workflow.set_entry_point("router")

# Edges
workflow.add_conditional_edges("router", _determine_mode, {
    "simple": "simple_rag",
    "research": "research_planner"
})

# Simple path
workflow.add_edge("simple_rag", "simple_agent")
workflow.add_conditional_edges("simple_agent", _should_continue, {
    "continue": "tools",
    "end": END
})

# Research path
workflow.add_edge("research_planner", "tools")
workflow.add_edge("tools", "research_gatherer")
workflow.add_conditional_edges("research_gatherer", _check_research_ready, {
    "gather_more": "tools",
    "build_report": "report_builder"
})
workflow.add_edge("report_builder", END)

graph = workflow.compile()
```

---

## Example Traces

### Simple Query: "What is RAG?"

```
1. router → Decision: SIMPLE
2. simple_rag → Retrieved 3 docs from Pinecone
3. simple_agent → Generated answer with citations
4. END
```

**Time**: ~3 seconds  
**Cost**: ~$0.02  
**Sources**: 3 KB sources

---

### Research Query: "Write a comprehensive report on RAG systems"

```
1. router → Decision: RESEARCH
2. research_planner → Tool call: research_topic_breakdown
3. tools → Executed breakdown, returned plan
4. research_gatherer → Tool call: search_knowledge_base("RAG fundamentals")
5. tools → Returned 5 KB sources [KB-1 to KB-5]
6. research_gatherer → Tool call: perplexity_search("latest RAG research 2025")
7. tools → Returned 5 web sources [WEB-1 to WEB-5] with AI analysis
8. research_gatherer → Signal: "RESEARCH COMPLETE"
9. report_builder → Tool call: create_report_outline
10. tools → Returned structured outline
11. report_builder → Generated 2500-word markdown report
12. END
```

**Time**: ~30 seconds  
**Cost**: ~$0.15  
**Sources**: 5 KB + 5 web = 10 total sources

---

## Key Architectural Decisions

### Why Separate Agents?

1. **Separation of Concerns**: Each agent has focused responsibility
2. **Tool Isolation**: Agents only access relevant tools
3. **Explicit Phases**: Clear boundaries between planning, gathering, building
4. **Debuggability**: Easy to see which phase failed
5. **Observability**: Langfuse shows clear agent transitions

### Why Router First?

1. **Cost Optimization**: Simple queries don't waste tokens on research prompts
2. **Performance**: Quick answers are actually quick
3. **Appropriate Resources**: Each path uses right tools/approach
4. **User Experience**: Natural language routing (no flags needed)

### Why Auto RAG in Simple Path?

1. **Speed**: No decision-making overhead
2. **Traditional**: Follows standard RAG pattern
3. **Predictable**: Always retrieves before answering
4. **Efficient**: One Pinecone query, one LLM call

### Why Tool-Based KB in Research Path?

1. **Flexible**: Agent decides when/how to search
2. **Multiple Searches**: Can query KB multiple times for different subtopics
3. **Agentic**: Agent autonomously orchestrates research
4. **Comprehensive**: Enables deeper, multi-faceted research

---

## Technology Stack

- **LangChain**: LLM integration, embeddings, tool definitions
- **LangGraph**: Multi-agent workflow orchestration
- **Langfuse**: Observability and tracing
- **OpenAI GPT-4**: Research agents (quality)
- **OpenAI GPT-3.5**: Router (speed/cost)
- **Pinecone**: Vector database for RAG
- **Perplexity**: AI-powered deep research API
- **FastAPI**: REST API with streaming

---

## Observability

Every request is **100% traced** in Langfuse:

- Router decision visible
- Each agent node traced separately
- Tool calls with inputs/outputs
- Token usage per node
- Latency per node
- Complete conversation history

View at: https://cloud.langfuse.com/

**Optional LangSmith**: For teams preferring LangChain's native observability, LangSmith provides equivalent tracing capabilities.

---

## Citation System

### Simple Mode
- Sources: `Source 1 (Title)`, `Source 2 (Title)`
- Format: Inline citations with source list at end

### Research Mode
- KB Sources: `(KB-1)`, `(KB-2)` inline
- Web Sources: `(WEB-1)`, `(WEB-2)` inline
- References section:
  ```markdown
  ## References
  
  ### Knowledge Base Sources
  [KB-1] Title (source.md)
  [KB-2] Title (source.md)
  
  ### Web Sources
  [WEB-1] Title (https://...)
  [WEB-2] Title (https://...)
  ```

---

## Future Enhancements

Potential improvements:
- [ ] Add "MEDIUM" research mode for moderate depth
- [ ] Implement conversation memory across sessions
- [ ] Add code generation agent for technical examples
- [ ] Export reports to PDF/Word
- [ ] Multi-modal research (images, diagrams)
- [ ] Collaborative research (multiple gatherers)
- [ ] Incremental refinement (user feedback loop)
- [ ] Domain-specific research templates

---

## Summary

This architecture provides:

✅ **Intelligent Routing** - Automatic intent classification  
✅ **Dual Workflows** - Simple RAG vs. Deep Research  
✅ **Multi-Agent** - Specialized agents per phase  
✅ **Tool Orchestration** - Dynamic tool selection  
✅ **Source Tracking** - Complete citation system  
✅ **Full Observability** - Every step traced  
✅ **Cost Optimized** - Appropriate resources per path  
✅ **Scalable** - Easy to add new agents/tools  

A production-ready agentic research system! 🚀
