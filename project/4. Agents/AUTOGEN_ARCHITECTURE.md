# AutoGen Implementation Architecture

## Overview

This document covers the **Track B: AutoGen implementation** of the multi-agent research system. If you're following **Track A: LangGraph**, see [`LANGGRAPH_ARCHITECTURE.md`](LANGGRAPH_ARCHITECTURE.md) instead.

AutoGen uses **conversation-based coordination** where agents communicate through natural language to accomplish tasks. No explicit workflow graphs or state management needed.

## High-Level Flow

```
User Request
     ↓
┌─────────────────┐
│   User Proxy    │ ← Initiates conversation
└─────────┬───────┘
          ↓
┌─────────────────┐
│  Router Agent   │ ← Classifies request complexity
└─────────┬───────┘
          │
     ┌────┴────┐
     ↓         ↓
  SIMPLE    RESEARCH
     │         │
     │         ↓
     │  ┌─────────────────┐
     │  │  Research Team  │ ← Full multi-agent workflow
     │  │  Group Chat     │
     │  └─────────┬───────┘
     │            │
     │            ↓
     │  ┌─────────────────────────────┐
     │  │  Agent Conversation         │
     │  │                             │
     │  │  Planner: "Breaking down..." │
     │  │  Researcher: "Found sources" │
     │  │  Writer: "Creating report..." │  
     │  │  Critic: "Needs improvement" │
     │  │  Writer: "Updated report"    │
     │  └─────────┬───────────────────┘
     │            │
     ↓            ↓
┌─────────────────────────────┐
│     Final Response          │
│  (Simple or Comprehensive)  │
└─────────────────────────────┘
```

## Core Components

### 1. Router Agent

**Purpose**: Intelligent request classification and workflow routing

**Key Features**:
- Analyzes user queries for complexity and intent
- Routes simple questions to direct response
- Routes research requests to multi-agent team
- Provides reasoning for routing decisions

```python
router = AssistantAgent(
    "router",
    system_message="""You are an intelligent request router.
    
    Analyze user requests and determine the appropriate response approach:
    
    SIMPLE_RESPONSE for:
    - Direct factual questions ("What is RAG?")
    - Definition requests ("Define transformer architecture") 
    - Quick explanations ("How does attention work?")
    
    RESEARCH_TEAM for:
    - Comprehensive analysis requests ("Write a report on...")
    - Multi-faceted research ("Analyze the evolution of...")
    - Comparative studies ("Compare different approaches to...")
    - Current trend analysis ("What are the latest developments in...")
    
    Respond with exactly: "ROUTE: SIMPLE_RESPONSE" or "ROUTE: RESEARCH_TEAM"
    Then provide a brief explanation of your routing decision.""",
    llm_config=llm_config
)
```

### 2. Simple Response Agent

**Purpose**: Handle straightforward queries with direct, cited answers

```python
simple_agent = AssistantAgent(
    "simple_responder", 
    system_message="""You provide direct, accurate answers to straightforward questions.
    
    Use available tools to search knowledge bases and provide well-cited responses.
    Keep answers focused and concise (200-500 words).
    Always include proper source citations.
    
    If the question requires comprehensive research, suggest: 
    "This topic would benefit from comprehensive research. Would you like a detailed report?"
    """,
    llm_config=llm_config
)
```

### 3. User Proxy Agent

**Purpose**: Interface between human user and agent team

**Key Features**:
- Initiates conversations with research requests
- Can execute code/tools if configured
- Manages human-in-the-loop interactions
- Terminates conversations when complete

```python
user_proxy = UserProxyAgent(
    "user_proxy",
    human_input_mode="NEVER",  # Fully autonomous
    max_consecutive_auto_reply=10,
    code_execution_config={"use_docker": False}
)
```

### 2. Research Assistant Agents

**Purpose**: Specialized agents for different research tasks

#### **Topic Planner Agent**
```python
planner = AssistantAgent(
    "planner",
    system_message="""You are a research topic planner. 
    Break down complex research topics into manageable subtopics.
    Identify key questions that need to be answered.
    Create a structured research plan.""",
    llm_config=llm_config
)
```

#### **Knowledge Researcher Agent**
```python
knowledge_researcher = AssistantAgent(
    "knowledge_researcher",
    system_message="""You are a knowledge base researcher.
    Search internal knowledge bases for foundational information.
    Provide well-cited information with source attribution.
    Focus on established facts and core concepts.""",
    llm_config=llm_config
)
```

#### **Web Researcher Agent**
```python
web_researcher = AssistantAgent(
    "web_researcher", 
    system_message="""You are a web research specialist.
    Search the web for latest information and recent developments.
    Use Perplexity API for AI-powered research synthesis.
    Always include source URLs and publication dates.""",
    llm_config=llm_config
)
```

#### **Report Writer Agent**
```python
writer = AssistantAgent(
    "writer",
    system_message="""You are a research report writer.
    Synthesize information from multiple sources into comprehensive reports.
    Write in clear markdown format with proper citations.
    Include executive summary and detailed analysis.""",
    llm_config=llm_config
)
```

#### **Quality Critic Agent**
```python
critic = AssistantAgent(
    "critic",
    system_message="""You are a research quality critic.
    Review research reports for accuracy, completeness, and clarity.
    Suggest specific improvements and identify gaps.
    Ensure proper source attribution throughout.""",
    llm_config=llm_config
)
```

### 3. Group Chat Manager

**Purpose**: Orchestrates multi-agent conversations

**Key Features**:
- Decides which agent speaks next
- Manages conversation flow and turn-taking
- Enforces maximum rounds to prevent infinite loops
- Handles agent selection based on context

```python
groupchat = GroupChat(
    agents=[user_proxy, planner, knowledge_researcher, web_researcher, writer, critic],
    messages=[],
    max_round=15,
    speaker_selection_method="auto"  # or "manual", "random"
)

manager = GroupChatManager(
    groupchat=groupchat,
    llm_config=llm_config,
    system_message="You are a research coordination manager."
)
```

## Routing Logic Implementation

### Routing Conversation Pattern
```python
def initiate_routed_conversation(user_request: str):
    # Step 1: Router classifies the request
    routing_chat = GroupChat(
        agents=[user_proxy, router],
        messages=[],
        max_round=2
    )
    
    routing_manager = GroupChatManager(groupchat=routing_chat)
    
    # Get routing decision
    routing_result = user_proxy.initiate_chat(
        routing_manager,
        message=f"Please route this request: {user_request}"
    )
    
    # Step 2: Execute appropriate workflow
    if "SIMPLE_RESPONSE" in routing_result.summary:
        return handle_simple_request(user_request)
    elif "RESEARCH_TEAM" in routing_result.summary:
        return handle_research_request(user_request)

def handle_simple_request(request: str):
    # Direct conversation with simple response agent
    simple_chat = GroupChat(
        agents=[user_proxy, simple_agent],
        messages=[],
        max_round=4
    )
    
    simple_manager = GroupChatManager(groupchat=simple_chat)
    return user_proxy.initiate_chat(simple_manager, message=request)

def handle_research_request(request: str):
    # Full research team conversation  
    research_chat = GroupChat(
        agents=[user_proxy, planner, knowledge_researcher, 
                web_researcher, writer, critic],
        messages=[],
        max_round=15
    )
    
    research_manager = GroupChatManager(groupchat=research_chat)
    return user_proxy.initiate_chat(research_manager, message=request)
```

## Agent Conversation Patterns

### 1. Simple Response Pattern
```
User → Router → "ROUTE: SIMPLE_RESPONSE" → Simple Agent → User
```

### 2. Research Team Pattern  
```
User → Router → "ROUTE: RESEARCH_TEAM" → Planner → Knowledge Researcher → Web Researcher → Writer → Critic → User
```

### 3. Iterative Refinement Pattern (Research)
```
User → Router → Research Team → Writer → Critic → Writer → Critic → Writer → User
(Writer and Critic iterate until quality threshold met)
```

### 4. Collaborative Research Pattern
```
User → Router → Research Team → Planner → [Knowledge Researcher + Web Researcher in parallel] → Writer → User
```

## Tool Integration

### Function Calling in AutoGen

```python
@user_proxy.register_for_execution()
@planner.register_for_llm(description="Search knowledge base for information")
def search_knowledge_base(query: str) -> str:
    """Search internal knowledge base"""
    # Your Phase 1 RAG system integration
    results = rag_system.search(query)
    return format_kb_results(results)

@user_proxy.register_for_execution()  
@web_researcher.register_for_llm(description="Search web using Perplexity")
def perplexity_search(query: str) -> str:
    """AI-powered web research"""
    # Your Phase 3 Perplexity integration
    results = perplexity_api.search(query)
    return format_web_results(results)
```

### Tool Usage Patterns

**Knowledge Base First Pattern**:
1. Knowledge Researcher searches internal sources
2. Web Researcher fills gaps with external sources
3. Writer synthesizes both types of information

**Comprehensive Research Pattern**:
1. Planner breaks down topic into subtopics
2. Each researcher takes different subtopics
3. Writer combines all research into unified report

## State Management (AutoGen Style)

Unlike LangGraph's explicit state, AutoGen manages state through:

### 1. Conversation History
- All agent messages preserved in group chat
- Agents reference previous messages for context
- Natural conversation flow maintains state

### 2. Agent Memory
```python
# Agents can maintain internal memory
class ResearcherWithMemory(AssistantAgent):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.research_cache = {}
        self.sources_found = []
    
    def search_and_remember(self, query):
        if query in self.research_cache:
            return self.research_cache[query]
        # Perform search and cache result
```

### 3. Shared Resources
```python
# Shared research state across agents
research_state = {
    "sources": [],
    "current_topic": "",
    "research_plan": {},
    "completed_sections": []
}

# Agents can access and update shared state
```

## Decision Points

### 1. Agent Selection
AutoGen's GroupChatManager automatically selects next speaker based on:
- Context relevance
- Agent capabilities  
- Conversation flow
- Custom selection logic

### 2. Conversation Termination
```python
def is_termination_msg(content):
    return "RESEARCH_COMPLETE" in content or "TERMINATE" in content

# Each agent can signal termination
user_proxy = UserProxyAgent(
    "user_proxy",
    is_termination_msg=is_termination_msg
)
```

### 3. Quality Gates
```python
# Critic agent enforces quality standards
critic_system_message = """
Review the research report. If it meets these criteria, say "APPROVED":
- At least 5 reliable sources cited
- Comprehensive coverage of topic
- Clear executive summary
- Proper markdown formatting

If not, provide specific improvement suggestions.
"""
```

## Example Implementation

### Research Workflow
```python
def run_research_project(topic: str):
    # Initialize conversation
    initial_message = f"""
    Research Project: {topic}
    
    Please coordinate to produce a comprehensive research report including:
    1. Executive summary
    2. Key findings from multiple sources  
    3. Analysis and insights
    4. Proper citations
    5. Markdown formatting
    """
    
    # Start group conversation
    user_proxy.initiate_chat(
        manager,
        message=initial_message,
        max_turns=20
    )
    
    # Extract final report from conversation
    final_report = extract_report_from_conversation()
    return final_report
```

### Conversation Flow Examples

#### Simple Query Flow:
```
User Proxy: "What is a transformer architecture?"

Router: "ROUTE: SIMPLE_RESPONSE - This is a direct factual question about a core AI concept"

Simple Agent: "A transformer architecture is a neural network model introduced in 2017..."
[Searches knowledge base, provides 300-word answer with citations]

User Proxy: "COMPLETE"
```

#### Research Query Flow:
```
User Proxy: "Research the evolution of transformer architectures"

Router: "ROUTE: RESEARCH_TEAM - This requires comprehensive analysis of developments over time"

Planner: "I'll break this into subtopics:
1. Original transformer (2017)  
2. BERT and encoder improvements
3. GPT and decoder scaling
4. Recent innovations (2023-2024)
5. Future directions"

Knowledge Researcher: "I'll search our knowledge base for foundational information about transformers..."
[Searches knowledge base, finds 5 sources]

Web Researcher: "I'll search for recent developments and innovations..."
[Uses Perplexity API, finds 5 recent sources]

Writer: "Based on the research, I'll create a comprehensive report..."
[Writes initial 2000-word report with citations]

Critic: "The report is good but needs more detail on recent innovations. Also missing comparison table..."

Writer: "I'll add the requested improvements..."
[Updates report with critic's suggestions]

Critic: "APPROVED - Report now meets quality standards"

User Proxy: "RESEARCH_COMPLETE"
```

#### Edge Case - Escalation Flow:
```
User Proxy: "What is attention mechanism?"

Router: "ROUTE: SIMPLE_RESPONSE - Direct question about core concept"

Simple Agent: "Attention mechanism allows models to focus on relevant parts of input... 
However, this topic has significant depth. Would you like a comprehensive research report covering attention variants, mathematical foundations, and recent innovations?"

User Proxy: "Yes, please provide comprehensive research"

[System automatically escalates to research team]

Router: "ROUTE: RESEARCH_TEAM - User requested comprehensive analysis"

[Full research workflow continues...]
```

## Observability

### Conversation Logging
```python
# AutoGen automatically logs all conversations
# Access via groupchat.messages

for message in groupchat.messages:
    print(f"{message['name']}: {message['content']}")
```

### Custom Tracing (Optional Langfuse Integration)
```python
from langfuse.decorators import langfuse_context

@langfuse_context.observe()
def research_with_tracing(topic):
    # Your research workflow with Langfuse tracing
    pass
```

## Advantages of AutoGen Architecture

✅ **Natural Communication**: Agents communicate like humans  
✅ **Flexible Roles**: Easy to add/modify agent specializations  
✅ **Emergent Behavior**: Agents develop unexpected collaboration patterns  
✅ **Simple Setup**: Less infrastructure than explicit workflow systems  
✅ **Transparent Process**: All reasoning visible in conversation log  

## Key Differences from LangGraph

| Aspect | AutoGen | LangGraph |
|--------|---------|-----------|
| **Coordination** | Natural conversation | Explicit workflow graph |
| **State** | Conversation history | Explicit state schema |  
| **Flow Control** | Agent selection | Conditional edges |
| **Debugging** | Read conversation log | Trace workflow execution |
| **Flexibility** | High (emergent behavior) | Medium (predefined paths) |
| **Predictability** | Medium (conversation-dependent) | High (explicit control) |

## Best Practices

### Agent Design
- **Clear roles**: Each agent should have distinct responsibilities
- **Focused system messages**: Specific instructions for each agent type
- **Quality gates**: Include critic/reviewer agents for important outputs

### Conversation Management  
- **Termination conditions**: Clear signals when task is complete
- **Round limits**: Prevent infinite conversations
- **Context management**: Keep conversations focused on current task

### Tool Integration
- **Register tools properly**: Use `@register_for_execution` and `@register_for_llm`
- **Error handling**: Tools should handle failures gracefully  
- **Response formatting**: Structure tool outputs for easy agent consumption

### Performance Optimization
- **Parallel research**: Multiple agents can work on different subtopics
- **Caching**: Avoid duplicate API calls through intelligent caching
- **Smart agent selection**: Guide conversation flow through manager prompts

## Future Enhancements

Potential AutoGen improvements:
- [ ] Add conversation branching for exploring multiple research directions
- [ ] Implement agent expertise scoring for better speaker selection  
- [ ] Add real-time collaboration with human researchers
- [ ] Create agent personality profiles for more diverse perspectives
- [ ] Implement hierarchical agent teams (team leaders + specialists)

---

## Summary

AutoGen provides a **conversation-centric approach** to multi-agent coordination:

✅ **Natural Agent Interactions** - Agents communicate through dialogue  
✅ **Flexible Team Composition** - Easy to add/remove specialized agents  
✅ **Emergent Intelligence** - Agents develop unexpected collaboration patterns  
✅ **Transparent Process** - All reasoning visible in conversation history  
✅ **Simple Architecture** - Less infrastructure than workflow-based systems  

Perfect for learning how agents can coordinate through natural language! 🤖💬
