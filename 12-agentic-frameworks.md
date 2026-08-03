# Agentic AI Frameworks — Building Blocks for AI Applications
![1785759834282](image/12-agentic-frameworks/1785759834282.png)
https://blog.jetbrains.com/pycharm/2026/06/top-agentic-frameworks-for-building-applications-2026/
## The Problem: Building Everything from Scratch

You want to build an AI agent that:
- Takes a user goal
- Retrieves relevant documents (RAG)
- Calls tools
- Maintains conversation state
- Handles errors
- Traces every step

Building this from scratch every time:

```python
# You end up writing:
- LLM API wrapper with retries
- Tool execution engine
- State management
- Conversation memory
- RAG pipeline (chunking, embedding, retrieval)
- Prompt templates
- Output parsing
- Error handling
- Observability/tracing
- Evaluation framework
```

That's months of work. For every project.

Frameworks solve this.

---

## What is an Agentic AI Framework?

> **A framework provides pre-built components for LLM applications: model calls, tools, memory, RAG, state management, agents, and orchestration — so you assemble instead of build from scratch.**

Think of it like web frameworks:

```
Without framework: Write HTTP parsing, routing, sessions, auth... manually
With Flask/Django: Import, configure, build your app logic

Without AI framework: Write LLM calls, tools, RAG, memory... manually  
With LangChain/etc: Import, configure, build your AI logic
```

---

## The Major Frameworks

```
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  LangChain         → High-level, broad integrations         │
│  LangGraph         → Stateful graphs, complex workflows     │
│  CrewAI            → Multi-agent teams with roles           │
│  AutoGen           → Multi-agent conversation/events        │
│  Strands Agents    → AWS-friendly, model-driven SDK         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

Let's understand each one.

---

## 1. LangChain

### Philosophy: "Universal Swiss Army Knife for LLM Apps"

LangChain provides everything:

```
- LLM wrappers (OpenAI, Anthropic, Bedrock, local models)
- Prompt templates
- Output parsers
- Document loaders (PDF, web, databases, APIs)
- Text splitters (chunking)
- Embedding models
- Vector stores
- Retrievers
- Tools and tool calling
- Agents
- Memory
- Callbacks and tracing (LangSmith)
```

### When to Use LangChain

```
✅ Quick prototyping
✅ Need many integrations out of the box
✅ Standard RAG or tool-calling patterns
✅ Team is new to LLM development
✅ Want one library that connects to everything
```

### When NOT to Use LangChain

```
❌ You need precise control over every step
❌ Complex stateful workflows (use LangGraph instead)
❌ Minimal dependencies desired
❌ You only need one or two LLM calls (just use the SDK directly)
```

### Example: Simple RAG with LangChain

```python
from langchain_community.document_loaders import WebBaseLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain.chains import RetrievalQA

# Load and chunk documents
docs = WebBaseLoader("https://docs.example.com").load()
chunks = RecursiveCharacterTextSplitter(chunk_size=1000).split_documents(docs)

# Create vector store
vectorstore = Chroma.from_documents(chunks, OpenAIEmbeddings())

# Create QA chain
qa = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(),
    retriever=vectorstore.as_retriever()
)

# Ask questions
answer = qa.invoke("What is the deployment procedure?")
```

---

## 2. LangGraph

### Philosophy: "State Machines for AI Workflows"

LangGraph is built on top of LangChain but focuses on:

```
- Explicit state management
- Graph-based workflows (nodes + edges)
- Conditional routing
- Loops with exit conditions
- Checkpointing (pause/resume)
- Human-in-the-loop (interrupt and continue)
- Durable execution
```

### The Mental Model

```
Traditional code:   Linear → if → else → return
LangGraph:         Graph of nodes connected by edges

┌────────┐     ┌──────────┐     ┌────────────┐
│  Start  │────→│  Research │────→│  Analyze    │
└────────┘     └──────────┘     └──────┬─────┘
                     ↑                   │
                     │         ┌─────────┴─────────┐
                     │         ▼                    ▼
                     │  ┌────────────┐    ┌──────────────┐
                     └──│ Need More? │    │ Write Report  │
                        └────────────┘    └──────────────┘
```

### When to Use LangGraph

```
✅ Multi-step workflows with branching
✅ Need to pause and resume (human approval)
✅ Complex agent loops with exit conditions
✅ Need checkpointing and replay
✅ Long-running tasks
```

### Example: Investigation Workflow

```python
from langgraph.graph import StateGraph, END

class InvestigationState(TypedDict):
    query: str
    evidence: list
    diagnosis: str
    approved: bool

def gather_evidence(state):
    # Call tools to gather information
    evidence = search_logs(state["query"])
    return {"evidence": evidence}

def analyze(state):
    # LLM analyzes the evidence
    diagnosis = llm.analyze(state["evidence"])
    return {"diagnosis": diagnosis}

def needs_approval(state):
    if "delete" in state["diagnosis"] or "restart" in state["diagnosis"]:
        return "human_review"
    return "deliver"

# Build the graph
graph = StateGraph(InvestigationState)
graph.add_node("gather", gather_evidence)
graph.add_node("analyze", analyze)
graph.add_node("human_review", wait_for_human)
graph.add_node("deliver", deliver_result)

graph.add_edge("gather", "analyze")
graph.add_conditional_edges("analyze", needs_approval)
graph.add_edge("human_review", "deliver")

workflow = graph.compile(checkpointer=memory)
```

---

## 3. CrewAI

### Philosophy: "Team of Specialized Agents with Roles"

CrewAI models multi-agent collaboration like a team:

```
Agent 1: Researcher (finds information)
Agent 2: Analyst (interprets data)
Agent 3: Writer (produces reports)
Agent 4: Reviewer (checks quality)
```

Each agent has:
- A role and backstory
- Specific tools
- A goal
- Collaboration rules

### When to Use CrewAI

```
✅ Task naturally decomposes into specialist roles
✅ You want agents to collaborate on complex outputs
✅ Business process automation
✅ Content creation pipelines
```

### Example: Incident Response Crew

```python
from crewai import Agent, Task, Crew

researcher = Agent(
    role="Incident Researcher",
    goal="Gather all relevant evidence about the incident",
    tools=[log_search, metrics_query, deployment_history],
    backstory="Senior SRE with 10 years of incident investigation"
)

analyst = Agent(
    role="Root Cause Analyst",
    goal="Determine the root cause from gathered evidence",
    tools=[correlation_analyzer],
    backstory="Expert in distributed systems failure analysis"
)

writer = Agent(
    role="Report Writer",
    goal="Write a clear, actionable incident report",
    tools=[],
    backstory="Technical writer specializing in incident documentation"
)

# Define tasks
research_task = Task(
    description="Investigate the API latency spike from last night",
    agent=researcher
)

analysis_task = Task(
    description="Analyze the evidence and determine root cause",
    agent=analyst,
    context=[research_task]  # Gets output from research
)

report_task = Task(
    description="Write the incident report",
    agent=writer,
    context=[research_task, analysis_task]
)

# Run the crew
crew = Crew(agents=[researcher, analyst, writer], tasks=[...])
result = crew.kickoff()
```

---

## 4. AutoGen

### Philosophy: "Conversational Multi-Agent Systems"

AutoGen focuses on agents that communicate through messages:

```
Agent A sends message → Agent B processes → Replies → Agent C reviews → ...
```

Key concepts:
- Agents communicate via conversation
- Can be event-driven and asynchronous
- Supports distributed architectures
- Human agents can join the conversation

### When to Use AutoGen

```
✅ Need agents to debate/discuss/negotiate
✅ Distributed agent systems
✅ Research into agent collaboration patterns
✅ Complex approval chains
✅ Event-driven architectures
```

### Example: Code Review System

```python
from autogen import AssistantAgent, UserProxyAgent

coder = AssistantAgent(
    name="Coder",
    system_message="You write Python code. Respond with code blocks."
)

reviewer = AssistantAgent(
    name="Reviewer", 
    system_message="You review code for bugs, security, and style."
)

human = UserProxyAgent(
    name="Human",
    human_input_mode="TERMINATE"  # Human approves final result
)

# Agents converse until human approves
human.initiate_chat(
    coder,
    message="Write a secure login function with rate limiting"
)
```

---

## 5. Strands Agents

### Philosophy: "Simple, Model-Driven Agent SDK"

Strands is AWS-friendly and focuses on:

```
- Model-driven tool selection (model decides)
- Python/TypeScript SDK
- Built-in MCP support
- OpenTelemetry for observability
- Works with any model provider
- Tools are just Python functions
```

### When to Use Strands

```
✅ AWS environment
✅ Want model-provider flexibility
✅ Need MCP integration out of the box
✅ Prefer minimal abstraction
✅ Want built-in telemetry
```

### Example: Simple Agent

```python
from strands import Agent
from strands.tools import tool

@tool
def get_service_health(service_name: str) -> dict:
    """Check the health status of a service."""
    return check_health(service_name)

@tool  
def get_recent_logs(service_name: str, minutes: int = 30) -> str:
    """Get recent logs for a service."""
    return fetch_logs(service_name, minutes)

agent = Agent(
    model="anthropic.claude-sonnet",
    tools=[get_service_health, get_recent_logs],
    system_prompt="You are a DevOps assistant. Investigate issues methodically."
)

response = agent("Why is the payment service returning 500 errors?")
```

---

## Framework Comparison Table

| Feature | LangChain | LangGraph | CrewAI | AutoGen | Strands |
|---------|-----------|-----------|--------|---------|---------|
| Primary Use | General LLM apps | Stateful workflows | Multi-agent teams | Agent conversations | Model-driven agents |
| Complexity | Medium | High | Medium | High | Low |
| State Mgmt | Basic | Advanced (graphs) | Per-task | Per-conversation | Session-based |
| Multi-Agent | Basic | Yes | Yes (core focus) | Yes (core focus) | Yes |
| Human-in-Loop | Limited | Built-in | Limited | Built-in | Via tools |
| MCP Support | Via integrations | Via LangChain | Limited | Limited | Built-in |
| Best For | Quick prototypes, RAG | Complex workflows | Role-based teams | Distributed agents | AWS + simple agents |
| Learning Curve | Low-Medium | Medium-High | Low | High | Low |

---

## Decision Framework: Which to Choose?

```
Is your task a simple RAG or tool-calling app?
    │
    ├── YES → LangChain (or just direct SDK calls)
    │
    └── NO → Does it need complex state, pause/resume, branching?
              │
              ├── YES → LangGraph
              │
              └── NO → Does it need multiple specialist agents?
                        │
                        ├── YES → Do they collaborate via conversation?
                        │          │
                        │          ├── YES → AutoGen
                        │          └── NO → CrewAI
                        │
                        └── NO → Simple agent with tools?
                                  │
                                  ├── AWS environment → Strands
                                  └── Otherwise → LangChain Agent or direct code
```

---

## When NOT to Use a Framework

Sometimes the answer is: **Don't use a framework.**

```
Use direct SDK calls when:
  - You have 1-2 LLM calls
  - Simple input → LLM → output
  - You want minimal dependencies
  - You need maximum control
  - Performance is critical

Use a workflow engine (Step Functions, Airflow) when:
  - Steps are fixed and known
  - You need retry/timeout guarantees
  - Compliance requires deterministic execution
  - Human approvals are part of a business process
```

---

## Production Considerations for All Frameworks

Regardless of which framework you choose:

### 1. Observability

```
Every framework needs:
  - Request tracing (end-to-end)
  - Token usage tracking
  - Latency per step
  - Error rates
  - Cost attribution
```

### 2. Testing

```
Test types:
  - Unit: Individual tool functions
  - Integration: Tool + LLM interaction
  - Evaluation: Answer quality on test sets
  - Regression: Known-good outputs still work after changes
```

### 3. Security

```
  - Never let the model choose which tools exist (define them)
  - Validate all tool inputs
  - Scope permissions per user/customer
  - Log every tool call and result
  - Require approval for write operations
```

### 4. Cost Control

```
  - Set max tokens per request
  - Set max steps for agents
  - Track cost per user/feature
  - Use cheaper models for simple tasks
  - Cache frequent queries
```

---

## Summary

```
Frameworks provide building blocks for AI applications:

LangChain  → Broad integrations, quick start, RAG, tools
LangGraph  → Stateful graphs, complex workflows, pause/resume
CrewAI     → Role-based multi-agent teams
AutoGen    → Conversational multi-agent systems
Strands    → Simple model-driven agents, AWS-friendly, MCP support

Choose based on:
  1. Workflow complexity (simple → complex)
  2. State requirements (stateless → durable state)
  3. Number of agents (single → multi-agent)
  4. Deployment environment (cloud provider, team skills)
  5. Control needs (high abstraction → low-level control)

Remember: No framework replaces good architecture.
Auth, security, testing, and observability are YOUR responsibility.
```

---

## Key Takeaways

1. Frameworks save months of boilerplate — use them for complex AI apps
2. LangChain for breadth, LangGraph for depth, CrewAI for teams, Strands for simplicity
3. Sometimes NO framework is best — direct SDK calls for simple cases
4. Frameworks don't provide security, auth, or correctness — you still own those
5. Choose based on your workflow type, not framework popularity
6. All frameworks are evolving rapidly — evaluate current capabilities before committing
7. Observability and testing matter more than framework choice

---

## Next → [13-gui-platforms.md](./13-gui-platforms.md)

> Not everyone can code. What if business analysts, product managers, or operations teams want to build AI workflows WITHOUT writing Python? GUI platforms let anyone create AI applications through visual interfaces — drag, drop, connect, deploy.
