# AWS Bedrock AgentCore — Production Infrastructure for AI Agents

## The Problem: From Demo to Production

You've built an AI agent.

It works on your laptop:

```python
agent = Agent(model="claude", tools=[search, query_db])
agent("Investigate the incident")
# Works! Great demo.
```

But production needs:

```
❌ Who is calling this agent? (Authentication)
❌ What are they allowed to access? (Authorization)
❌ How does the agent call external services securely? (Credentials)
❌ Where does the agent run? (Hosting)
❌ How does it remember previous conversations? (Memory)
❌ What happens when it fails? (Error handling)
❌ How do we know what it did? (Observability)
❌ How do we control costs? (Budget)
❌ How do we update without downtime? (Deployment)
❌ How do we scale? (Infrastructure)
```

Your notebook demo handles NONE of this.

---

## What is AWS Bedrock AgentCore?

> **AgentCore provides production infrastructure for AI agents: hosting, identity, tool connectivity, memory, and observability — so you focus on agent logic, not platform engineering.**

Think of it like this:

```
Your agent code = The application
AgentCore = The production platform (like ECS/Lambda but for AI agents)

Just like:
  Your web app code = The application  
  AWS ECS/Lambda = The production platform

You write the logic.
AgentCore handles the infrastructure.
```

---

## AgentCore Building Blocks

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS Bedrock AgentCore                      │
│                                                              │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  │
│  │    RUNTIME    │  │   IDENTITY    │  │   GATEWAY     │  │
│  │               │  │               │  │               │  │
│  │ Host agent    │  │ Auth agents   │  │ Connect to    │  │
│  │ code,         │  │ and users,    │  │ tools and     │  │
│  │ serverless    │  │ manage creds  │  │ resources     │  │
│  │ execution     │  │               │  │ securely      │  │
│  └───────────────┘  └───────────────┘  └───────────────┘  │
│                                                              │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  │
│  │    MEMORY     │  │  OBSERVABILITY│  │  BUILT-IN     │  │
│  │               │  │               │  │  TOOLS        │  │
│  │ Managed       │  │ Metrics,      │  │               │  │
│  │ memory for    │  │ traces,       │  │ Code interp,  │  │
│  │ agents        │  │ logs          │  │ browser, etc  │  │
│  └───────────────┘  └───────────────┘  └───────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 1. Runtime — Where Your Agent Lives

AgentCore Runtime hosts your agent code in a serverless environment.

### What It Does

```
- Runs your agent code (Python, containers)
- Scales automatically
- Manages compute resources
- Handles invocations
- Supports multiple frameworks (Strands, LangGraph, CrewAI, custom)
```

### Two Deployment Modes

```
Direct Deployment:
  - Push your code directly
  - AWS manages the runtime environment
  - You manage: agent code and dependencies
  - AWS manages: language runtime, patching

Container Deployment:
  - Build a Docker image
  - Push to ECR, deploy to AgentCore
  - You manage: everything in the container
  - More control, more responsibility
```

### Key Concept: Framework Agnostic

AgentCore doesn't force a framework:

```
Your agent can be:
  - Strands Agent
  - LangGraph workflow
  - CrewAI crew
  - Plain Python with direct API calls
  - Any framework that runs in Python/Container
```

---

## 2. Identity — Who's Calling and What Can They Do?

Two different identity problems:

### Inbound Identity (Who is calling the agent?)

```
User/System → "I want to invoke this agent"
             
AgentCore: "Who are you? Prove it. What are you allowed to do?"
```

### Outbound Identity (What credentials does the agent use?)

```
Agent → "I need to call Jira API"

AgentCore: "Here are scoped, short-lived credentials for Jira.
           You can only read tickets for Customer X."
```

### Why This Matters

```
WRONG (common in demos):
  Agent has admin access to everything.
  User's prompt determines what data to access.
  
  Problem: User says "Show me ALL customer data"
  → Agent complies because it HAS access to all data!

RIGHT (production):
  Agent's credentials are scoped to the caller's permissions.
  Even if the model tries to access other data, the credentials won't allow it.
  
  Problem: User says "Show me ALL customer data"
  → API returns only data the user is authorized to see.
```

---

## 3. Gateway — Secure Tool Connectivity

Gateway provides a governed connection point between your agent and external tools.

```
Agent → "I want to call get_ticket(PROJ-123)"
           │
           ▼
┌─────────────────────────┐
│        GATEWAY           │
│                          │
│  1. Validate tool name   │
│  2. Validate parameters  │
│  3. Check authorization  │
│  4. Apply rate limits    │
│  5. Get credentials      │
│  6. Call downstream API  │
│  7. Log the call         │
│  8. Return result        │
└─────────────────────────┘
           │
           ▼
    Jira API (downstream)
```

### Why Not Call APIs Directly?

```
Direct:
  Agent code → Jira API
  
  Problems:
  - Credentials hardcoded or broadly scoped
  - No centralized audit
  - No rate limiting
  - Each tool manages its own auth

Gateway:
  Agent code → Gateway → Jira API
  
  Benefits:
  - Centralized credential management
  - Unified audit log
  - Rate limiting and policies
  - Consistent auth pattern
```

---

## 4. Memory — What the Agent Remembers

LLMs are stateless. They forget everything between calls.

AgentCore Memory provides managed persistence:

```
Types of memory:

Session Memory:
  "What happened in THIS conversation"
  - Last 10 messages
  - Current investigation findings
  - Temporary state

Long-Term Memory:
  "What I learned from PREVIOUS conversations"
  - User preferences
  - Past decisions
  - Learned facts about the environment
```

### Critical Decision: What Should Be Remembered?

```
REMEMBER:
  ✅ User prefers concise answers
  ✅ Last investigation found the issue was DNS
  ✅ Customer X uses PostgreSQL 14

DO NOT REMEMBER:
  ❌ Full customer data (security risk)
  ❌ Credentials or secrets
  ❌ Unverified guesses from the model
  ❌ Data that changes frequently (becomes stale)
```

---

## 5. Observability — What Happened and Why?

Production agents need full tracing:

```
Request arrives
    │ trace_id: abc-123
    │
    ├── Authentication: user=john, customer=acme
    │
    ├── Agent Step 1: Thinking (200ms, 500 tokens)
    │     └── Decision: "Need to check metrics"
    │
    ├── Agent Step 2: Tool call (800ms)
    │     ├── Tool: get_metrics("api-gateway")
    │     ├── Gateway: authorized, rate_limit_ok
    │     └── Result: {latency: "180ms", error_rate: "5%"}
    │
    ├── Agent Step 3: Thinking (300ms, 800 tokens)
    │     └── Decision: "Need deployment history"
    │
    ├── Agent Step 4: Tool call (400ms)
    │     ├── Tool: get_deployments("last_24h")
    │     └── Result: {deployment: "v2.4.1 at 02:30"}
    │
    ├── Agent Step 5: Final answer (500ms, 1200 tokens)
    │     └── Generated diagnosis with citations
    │
    └── Total: 2.2s, 2500 tokens, 2 tool calls, cost: $0.003
```

Without this, you can't debug, audit, or optimize.

---

## 6. MCP on AgentCore

AgentCore can host MCP servers:

```
Your AI App (anywhere)
      │
      │ MCP Protocol (Streamable HTTP)
      ▼
AgentCore Runtime
(Hosting your MCP Server)
      │
      ▼
Gateway → Downstream APIs
```

This means:
- Build an MCP server
- Deploy it on AgentCore
- Any MCP-compatible AI app can connect to it
- AgentCore handles hosting, scaling, auth, observability

---

## Complete Architecture Example

### Scenario: Customer Operations Investigation Agent

An engineer asks: "Why is Customer Acme's API returning errors?"

```
┌───────────────────────────────────────────────────────────────┐
│                                                                │
│  Engineer                                                      │
│     │                                                          │
│     │ "Investigate Acme API errors"                           │
│     ▼                                                          │
│  Internal Portal (authenticates engineer)                      │
│     │                                                          │
│     │ Authenticated request + customer scope                  │
│     ▼                                                          │
│  ┌──────────────────────────────────────┐                     │
│  │  AgentCore RUNTIME                    │                     │
│  │                                       │                     │
│  │  Agent Code (Strands/LangGraph/etc)  │                     │
│  │     │                                 │                     │
│  │     ├── THINK → "Check service health"│                     │
│  │     │                                 │                     │
│  │     ├── ACT → request tool call      │                     │
│  │     │              │                  │                     │
│  │     │              ▼                  │                     │
│  │     │     ┌────────────────┐         │                     │
│  │     │     │  GATEWAY       │         │                     │
│  │     │     │  - validate    │         │                     │
│  │     │     │  - authorize   │         │                     │
│  │     │     │  - credential  │         │                     │
│  │     │     │  - execute     │         │                     │
│  │     │     │  - audit       │         │                     │
│  │     │     └───────┬────────┘         │                     │
│  │     │             │                   │                     │
│  │     │             ▼                   │                     │
│  │     │     Metrics API (Acme-scoped)  │                     │
│  │     │                                 │                     │
│  │     ├── OBSERVE result               │                     │
│  │     ├── THINK → "Check deployments"  │                     │
│  │     ├── ACT → another tool call      │                     │
│  │     ├── OBSERVE result               │                     │
│  │     └── FINAL ANSWER + citations     │                     │
│  │                                       │                     │
│  │  ┌────────┐  ┌──────────────┐       │                     │
│  │  │ MEMORY │  │ OBSERVABILITY │       │                     │
│  │  │ (save  │  │ (trace every │       │                     │
│  │  │ findings)│ │  step)       │       │                     │
│  │  └────────┘  └──────────────┘       │                     │
│  └──────────────────────────────────────┘                     │
│                                                                │
│     Result: "Acme API errors caused by deployment v2.4.1      │
│     at 2:30 AM. Recommend rollback."                          │
│                                                                │
│     │                                                          │
│     ▼                                                          │
│  REQUIRES APPROVAL (rollback is destructive)                   │
│     │                                                          │
│     ▼                                                          │
│  Senior Engineer approves                                      │
│     │                                                          │
│     ▼                                                          │
│  Deterministic Workflow (Step Functions) executes rollback     │
│                                                                │
└───────────────────────────────────────────────────────────────┘
```

---

## Runtime vs Harness (AWS Terminology)

AWS distinguishes between two approaches:

### AgentCore Runtime (Bring Your Own Agent)

```
You write: Complete agent code (any framework)
AgentCore provides: Hosting, scaling, identity, gateway, memory

Best for:
  - Custom agent logic
  - Specific framework requirements
  - Complex workflows
  - Full control over agent behavior
```

### Bedrock Agents (Managed/Harness Approach)

```
You configure: Prompts, tools, knowledge bases via console/API
AWS provides: Agent loop, model integration, orchestration

Best for:
  - Standard agent patterns
  - Quick deployment
  - Less code to maintain
  - AWS-managed orchestration
```

```
Simple analogy:
  Runtime = "Give me a server, I'll run my code" (like EC2/ECS)
  Harness = "I'll describe what I want, you run it" (like Lambda)
```

---

## Security Boundaries

```
1. USER BOUNDARY
   Engineer is authenticated before agent is invoked.
   Agent cannot be invoked anonymously.

2. CUSTOMER BOUNDARY  
   Every tool call is filtered to the customer scope.
   Agent code cannot access data outside the authorized customer.
   
3. MODEL BOUNDARY
   User input and retrieved text are DATA, not instructions.
   The model cannot override authorization by generating text.
   
4. ACTION BOUNDARY
   Read operations: Agent can do freely (within scope)
   Write operations: Require human approval + separate workflow
   
5. TELEMETRY BOUNDARY
   Logs capture what happened without storing secrets or PII.
```

---

## When to Use AgentCore vs Build Your Own

```
USE AGENTCORE WHEN:
  ✅ Running agents in AWS
  ✅ Need managed identity for agent workloads
  ✅ Want centralized tool gateway with policies
  ✅ Need managed memory/state
  ✅ Want integrated observability
  ✅ Multiple agents sharing infrastructure

BUILD YOUR OWN WHEN:
  ✅ Multi-cloud requirement
  ✅ On-premises deployment needed
  ✅ Very custom requirements not supported
  ✅ Already have equivalent infrastructure
  ✅ Cost optimization at extreme scale
```

---

## Production Readiness Checklist

Before deploying an agent to production on AgentCore:

```
Identity:
  □ Inbound auth configured (who can invoke?)
  □ Outbound credentials scoped (what can agent access?)
  □ Customer boundary enforced at every tool call

Tools:
  □ Each tool has typed schema and clear description
  □ Input validation on every tool call
  □ Timeouts and rate limits configured
  □ Write tools require approval workflow

Memory:
  □ Decided what to persist vs what's ephemeral
  □ Retention policy defined
  □ Deletion process for customer data removal
  □ No secrets stored in memory

Observability:
  □ End-to-end traces configured
  □ Token usage and cost tracked
  □ Error rates monitored with alerts
  □ Audit log covers all tool calls and decisions

Safety:
  □ Max steps/tokens/time configured
  □ Human approval for destructive actions
  □ Graceful failure handling (escalation path)
  □ Prompt injection defenses in place

Deployment:
  □ Staging environment tested
  □ Evaluation test suite passing
  □ Rollback procedure documented
  □ Cost budget alerts configured
```

---

## Summary

```
AWS Bedrock AgentCore = Production platform for AI agents

Components:
  Runtime:       Host and run agent code (any framework)
  Identity:      Auth for users AND for agent-to-tool connections
  Gateway:       Secure, governed tool connectivity
  Memory:        Managed state and recall for agents
  Observability: Full traces, metrics, and audit logs
  Built-in Tools: Code execution, browser, etc.

Key principle:
  AgentCore provides INFRASTRUCTURE.
  YOU provide LOGIC and BUSINESS CONTROLS.
  
  It doesn't make your agent smart or safe.
  It makes your agent DEPLOYABLE and OPERABLE.
```

---

## Key Takeaways

1. AgentCore bridges the gap between demo and production for AI agents
2. It's framework-agnostic — bring Strands, LangGraph, CrewAI, or custom code
3. Identity is two-sided: who calls the agent AND what the agent can access
4. Gateway centralizes tool access with auth, rate limits, and audit
5. Memory needs governance — decide what's stored, for how long, who can access
6. Observability is non-negotiable — you can't debug what you can't trace
7. AgentCore provides infrastructure, not safety — you still own security design

---

## Congratulations! You've Completed the Journey

```
01. Machine Learning      ✅ — Machines learn from data
02. Deep Learning         ✅ — Machines discover features themselves
03. Transformers          ✅ — Attention mechanism for language
04. LLM Architecture      ✅ — How GPT actually works inside
05. Prompt Engineering    ✅ — How to talk to AI effectively
06. Context Engineering   ✅ — What information to give the AI
07. Harness Engineering   ✅ — The system that wraps the AI
08. Tool Calling          ✅ — Giving AI the ability to act
09. AI Agents             ✅ — Autonomous problem-solving loops
10. MCP                   ✅ — Universal standard for AI-tool connections
11. RAG                   ✅ — Giving AI your private knowledge
12. Agentic Frameworks    ✅ — Building blocks for AI applications
13. GUI Platforms         ✅ — Visual AI app building
14. AWS Bedrock AgentCore ✅ — Production infrastructure for agents
```

You now understand the **complete journey of AI** — from basic pattern recognition to production-grade autonomous agents.

The future is Agentic, Autonomous, and Augmented.

Now go build something amazing.
