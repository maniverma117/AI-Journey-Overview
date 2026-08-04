# AI Agents — Autonomous Problem-Solving Loops
![1785759030619](image/07-ai-agents/1785759030619.png)
https://billtcheng2013.medium.com/autonomous-agent-part-1-c3931090c9a4
## The Problem with Simple Tool Calling

Tool calling is great for one-shot questions:

```
"What's the weather?" → call tool → answer
```

But what about complex tasks?

```
"Investigate why our API latency increased 3x last night,
find the root cause, and propose a fix."
```

This requires:

1. Check metrics dashboard
2. Based on metrics, look at specific service logs
3. Based on logs, check recent deployments
4. Based on deployments, look at the code diff
5. Correlate all findings
6. Propose a solution

No human told the model to follow these exact steps.

The model must **figure out the plan** and **execute it step by step**.

This is an AI Agent.

---

## What is an AI Agent?

> **An AI Agent is a system where the LLM operates in a loop — observing, thinking, acting, and repeating — until it achieves a goal or decides it cannot.**

The key difference from simple tool calling:

```
Tool Calling:
    Human asks → Model calls ONE tool → Done.

Agent:
    Human sets GOAL → Model plans → Calls tool → Observes result →
    Thinks about what to do next → Calls another tool → Observes →
    ... LOOPS until goal is achieved or fails.
```

---

## The Agent Loop

Every agent follows this pattern:

```
┌─────────────────────────────────────┐
│           USER GOAL                  │
│ "Find why latency increased"        │
└──────────────┬──────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│           THINK                       │
│ "I should check the metrics first"   │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│           ACT                         │
│ Call: get_metrics("api-gateway",      │
│                   "latency", "24h")   │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│           OBSERVE                     │
│ Result: Latency spiked at 2:30 AM    │
│ from 50ms to 180ms                   │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│           THINK                       │
│ "Spike at 2:30 AM. Let me check     │
│  what was deployed around that time"  │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│           ACT                         │
│ Call: get_deployments("2:00-3:00 AM") │
└──────────────┬───────────────────────┘
               │
               ▼
         ... continues looping ...
               │
               ▼
┌──────────────────────────────────────┐
│           FINAL ANSWER                │
│ "Root cause: Deployment at 2:28 AM   │
│  introduced an unindexed DB query.   │
│  Fix: Add index on users.email"      │
└──────────────────────────────────────┘
```

---

## The ReAct Pattern (Reason + Act)

Most agents use the ReAct framework:

```
Thought: I need to understand what caused the latency spike.
         Let me check the metrics first.
Action:  get_metrics(service="api-gateway", metric="latency", range="24h")
Observation: Latency jumped from 50ms to 180ms at 2:30 AM.

Thought: Something happened at 2:30 AM. Was there a deployment?
Action:  get_deployments(time_range="2:00-3:00 AM")
Observation: Deployment #4521 at 2:28 AM by user john@company.com

Thought: Deployment #4521 is suspicious. Let me see what changed.
Action:  get_deployment_diff(deployment_id="4521")
Observation: Added query "SELECT * FROM users WHERE email LIKE '%@gmail%'"

Thought: That query uses LIKE with a leading wildcard on a large table. 
         This would cause a full table scan. This is the root cause.
Final Answer: The latency increase was caused by deployment #4521...
```

The model alternates between **reasoning** and **acting**.

---

## Agent vs Chatbot vs Tool-Calling App

```
┌────────────────────────────────────────────────────────────────┐
│ CHATBOT                                                         │
│ User asks → Model answers from knowledge → Done                 │
│ No tools. No actions. Just conversation.                        │
├────────────────────────────────────────────────────────────────┤
│ TOOL-CALLING APP                                                │
│ User asks → Model calls a tool → Gets result → Answers         │
│ One round of tool use. Human controls the flow.                 │
├────────────────────────────────────────────────────────────────┤
│ AGENT                                                           │
│ User sets goal → Model plans → Loops (think/act/observe)       │
│ Multiple rounds. MODEL controls the flow.                       │
│ Decides what to do, when to stop, how to recover from errors.  │
└────────────────────────────────────────────────────────────────┘
```

The critical difference: **who controls the loop?**

- Chatbot: No loop
- Tool-calling: Human controls (one tool call per turn)
- Agent: MODEL controls (decides next action autonomously)

---

## Agent Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     AGENT SYSTEM                          │
│                                                          │
│  ┌─────────────┐                                        │
│  │    GOAL     │ "Investigate latency issue"            │
│  └──────┬──────┘                                        │
│         │                                                │
│         ▼                                                │
│  ┌─────────────┐                                        │
│  │    MEMORY   │ Conversation history + findings so far │
│  └──────┬──────┘                                        │
│         │                                                │
│         ▼                                                │
│  ┌─────────────┐     ┌──────────────┐                  │
│  │     LLM     │────→│   DECISION   │                  │
│  │  (Brain)    │     │ Call tool? Done? Ask user?       │
│  └─────────────┘     └──────┬───────┘                  │
│                              │                           │
│              ┌───────────────┼───────────────┐          │
│              ▼               ▼               ▼          │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐   │
│  │   Tool A     │ │   Tool B     │ │   Tool C     │   │
│  │ get_metrics  │ │ get_logs     │ │ get_deploys  │   │
│  └──────────────┘ └──────────────┘ └──────────────┘   │
│                                                          │
│  ┌─────────────────────────────────────────────┐        │
│  │ GUARDRAILS                                   │        │
│  │ - Max 10 tool calls                         │        │
│  │ - 60 second timeout                         │        │
│  │ - No write operations without approval      │        │
│  │ - Must stay within customer scope           │        │
│  └─────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────┘
```

---

## Agent Memory Types

### Short-Term Memory (Within One Task)

```
What I've done so far:
1. Checked metrics → latency spike at 2:30 AM
2. Checked deployments → found deployment #4521
3. Checked diff → found problematic query

What I still need to do:
4. Verify this is actually the cause
5. Propose a fix
```

This is just the conversation context.

### Long-Term Memory (Across Tasks)

```
From previous sessions:
- This service had a similar issue 3 months ago (deployment #3102)
- The team prefers PostgreSQL indexes over query rewrites
- John is the database admin for this service
```

Stored externally (database, vector store) and retrieved when relevant.

### Working Memory (Scratchpad)

```
Current findings:
- Spike started at 2:30 AM (confirmed by 3 data sources)
- Deployment #4521 is the only change in the window
- The query uses LIKE with leading wildcard
- Table has 50M rows, no index on email column
- Expected impact: full table scan = ~150ms added latency
- Actual impact: 130ms increase. MATCHES!

Confidence: HIGH
```

---

## When to Use Agents (And When NOT to)

### Use an Agent When:

```
✅ The task requires multiple dynamic steps
✅ You don't know the exact steps in advance
✅ The model needs to make decisions based on intermediate results
✅ Investigation/research tasks
✅ The path depends on what's discovered along the way
```

### Do NOT Use an Agent When:

```
❌ Steps are known and fixed → Use a deterministic workflow
❌ The task is simple → Use basic tool calling
❌ Failures are catastrophic → Use human-controlled pipelines
❌ You need 100% predictability → Use traditional code
❌ Cost is critical → Agents use many LLM calls (expensive)
```

### The Decision Framework

```
Is the sequence of steps known in advance?
    │
    ├── YES → Use a workflow engine (Step Functions, Airflow)
    │         Cheaper, faster, predictable, testable
    │
    └── NO → Does it require reasoning about results?
              │
              ├── YES → Agent might be appropriate
              │         (with guardrails!)
              │
              └── NO → Use simple tool calling
```

---

## Agent Failure Modes

### 1. Infinite Loops

```
Agent: "Let me check the logs"
Agent: "Hmm, not enough info. Let me check the logs again"
Agent: "Still not clear. Let me check the logs one more time"
... forever ...
```

Fix: Maximum step count.

### 2. Scope Creep

```
Goal: "Check why API is slow"
Agent: "Let me also optimize the database"
Agent: "While I'm at it, let me refactor the code"
Agent: "Actually, let me redesign the entire architecture"
```

Fix: Clear boundaries in the system prompt.

### 3. Hallucinated Actions

```
Agent: "I'll call deploy_fix()"  ← This tool doesn't exist!
```

Fix: Strict tool validation in the harness.

### 4. Wrong Conclusions

```
Agent: "Latency increased. I see a deployment. MUST be the deployment."
(Correlation is not causation)
```

Fix: Require the agent to verify findings from multiple sources.

### 5. Cost Explosion

```
Agent makes 50 LLM calls × 100K tokens each = 5M tokens = $$$$
```

Fix: Token budgets and step limits.

---

## Safety Controls for Agents

```python
class SafeAgent:
    MAX_STEPS = 10
    MAX_TOKENS = 500_000
    TIMEOUT_SECONDS = 120
    
    def run(self, goal, user):
        steps = 0
        total_tokens = 0
        
        while steps < self.MAX_STEPS:
            # Think and decide next action
            decision = self.llm.decide(goal, self.memory)
            
            if decision.type == "final_answer":
                return decision.answer
            
            if decision.type == "tool_call":
                # SAFETY CHECKS
                if decision.tool not in user.allowed_tools:
                    self.memory.add("Tool not permitted. Try another approach.")
                    continue
                
                if decision.tool in WRITE_TOOLS:
                    approval = self.request_approval(decision, user)
                    if not approval:
                        self.memory.add("Action denied. Cannot proceed.")
                        continue
                
                # Execute
                result = self.execute_tool(decision.tool, decision.args)
                self.memory.add(f"Result: {result}")
            
            steps += 1
            total_tokens += decision.tokens_used
            
            if total_tokens > self.MAX_TOKENS:
                return "Budget exceeded. Partial findings: ..."
        
        return "Max steps reached. Partial findings: ..."
```

---

## Single Agent vs Multi-Agent

### Single Agent

One model, one loop, multiple tools.

```
Agent → thinks → uses tool → thinks → uses tool → answer
```

Good for: Most tasks.

### Multi-Agent

Multiple specialized models collaborating.

```
Researcher Agent → finds information
Analyzer Agent → interprets findings
Writer Agent → produces final report
Reviewer Agent → checks for errors
```

Good for: Very complex tasks where specialization helps.

More on this in [12-agentic-frameworks.md](./12-agentic-frameworks.md).

---

## Real-World Example: Incident Investigation Agent

```
GOAL: "Customer reports API errors on order service"

Step 1 - THINK: "Check service health first"
Step 1 - ACT: get_service_health("order-service")
Step 1 - OBSERVE: {status: "degraded", error_rate: "23%", p99_latency: "4.2s"}

Step 2 - THINK: "23% error rate is high. Let me see what errors."
Step 2 - ACT: get_error_logs("order-service", severity="error", last="1h")
Step 2 - OBSERVE: {top_error: "ConnectionTimeout to payment-service", count: 1847}

Step 3 - THINK: "Order service can't reach payment service. Is payment healthy?"
Step 3 - ACT: get_service_health("payment-service")
Step 3 - OBSERVE: {status: "unhealthy", pods_ready: "0/3", event: "ImagePullBackOff"}

Step 4 - THINK: "Payment pods can't pull their image. Container registry issue?"
Step 4 - ACT: get_deployment_events("payment-service")
Step 4 - OBSERVE: {event: "Failed to pull image payment-service:v2.4.1, unauthorized"}

Step 5 - THINK: "Image pull is unauthorized. Registry credentials may have expired."
Step 5 - ACT: get_secret_expiry("ecr-credentials")
Step 5 - OBSERVE: {expired: true, expired_at: "2024-01-15T03:00:00Z"}

FINAL ANSWER:
"Root cause: ECR registry credentials expired at 3:00 AM today.
Payment-service pods cannot pull image v2.4.1.
This causes ConnectionTimeout errors in order-service (23% error rate).

Recommended fix:
1. Rotate ECR credentials (requires ops-team approval)
2. Payment pods will restart and pull image successfully
3. Order service errors will resolve

Urgency: HIGH - customer-facing impact"
```

The agent discovered a multi-layer issue through autonomous investigation.

---

## Summary

```
AI Agent = LLM + Loop + Tools + Memory + Goal

The loop:
  1. THINK (reason about what to do next)
  2. ACT (call a tool)
  3. OBSERVE (see the result)
  4. REPEAT (until goal is achieved)

Key differences from tool calling:
  - Agent controls the loop (autonomous)
  - Multiple steps and decisions
  - Can recover from errors and try alternatives
  - Must have safety guardrails

Guardrails:
  - Max steps, max tokens, max time
  - Permission checks on every tool call
  - Human approval for write operations
  - Clear scope boundaries
```

---

## Key Takeaways

1. An agent is an LLM that LOOPS: think → act → observe → repeat
2. The model controls the flow — it decides what to do next
3. Agents are powerful for investigation and multi-step reasoning
4. Without guardrails, agents can loop forever, hallucinate, or cause damage
5. Use agents ONLY when steps are truly dynamic — use workflows for fixed steps
6. Safety: max steps, token budgets, permission checks, human approval for actions
7. Most failures are scope creep, loops, or wrong conclusions — plan for them

---

## Next → [08-mcp.md](./08-mcp.md)

> Every agent needs tools. But what if you have 50 different AI apps that all need access to the same tools? Do you rebuild integrations for each one? MCP (Model Context Protocol) is a standard that lets any AI app connect to any tool through one universal protocol.
