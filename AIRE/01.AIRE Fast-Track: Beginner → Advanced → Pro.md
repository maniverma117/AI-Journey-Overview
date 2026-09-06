# AIRE Fast-Track: Beginner → Advanced → Pro

We'll structure everything into **6 levels**.

```text
LEVEL 0   AI Infrastructure Foundations
                 ↓
LEVEL 1   GPU + LLM Inference
                 ↓
LEVEL 2   Production vLLM
                 ↓
LEVEL 3   Kubernetes/EKS AI Platform
                 ↓
LEVEL 4   Distributed AI / llm-d / Autoscaling
                 ↓
LEVEL 5   AI Reliability Engineering
                 ↓
LEVEL 6   AI Solution Architect / Platform Architect
```

I would make every level end with a **real project and architecture interview**.

---

# LEVEL 0 — AI Infrastructure Foundations

### Goal

Connect what you already understand about GenAI with how models actually run.

You don't need to become a data scientist.

You need to understand this pipeline:

```text
User
 ↓
Prompt
 ↓
Tokenizer
 ↓
Tokens
 ↓
Transformer
 ↓
Attention
 ↓
KV Cache
 ↓
Logits
 ↓
Sampling
 ↓
Generated token
```

## Tutorial 0.1 — Tokens

Learn:

```text
"How are you?"
       ↓
Tokenizer
       ↓
[How] [are] [you] [?]
       ↓
Token IDs
```

Understand:

* tokenizer
* vocabulary
* token IDs
* input tokens
* output tokens
* context window
* maximum sequence length

Why AIRE cares:

```text
more tokens
   ↓
more compute
   ↓
more KV cache
   ↓
more GPU memory
   ↓
higher latency
   ↓
higher cost
```

---

# Tutorial 0.2 — Transformer basics

Understand only the operational pieces:

```text
Embedding
   ↓
Attention
   ↓
MLP
   ↓
Attention
   ↓
MLP
   ↓
Logits
```

You need to understand:

* parameters
* layers
* hidden size
* attention heads
* embeddings
* model weights

Not the advanced mathematics initially.

---

# Tutorial 0.3 — Prefill vs Decode

Critical concept.

### Prefill

```text
5000 input tokens
       ↓
Transformer
       ↓
KV Cache
       ↓
first output token
```

Metric:

```text
TTFT
Time To First Token
```

### Decode

```text
token 1
 ↓
token 2
 ↓
token 3
 ↓
token 4
```

Metric:

```text
TPOT
Time Per Output Token
```

This will later explain half of your performance problems.

---

# LEVEL 1 — GPU Engineer for AI

Now we'll make your existing infrastructure knowledge GPU-aware.

## Tutorial 1.1 — GPU architecture

Understand:

```text
GPU
├── Compute
│    ├── CUDA Cores
│    └── Tensor Cores
│
├── Memory
│    └── HBM / VRAM
│
├── Interconnect
│    ├── PCIe
│    └── NVLink
│
└── Software
     ├── NVIDIA Driver
     ├── CUDA
     └── NCCL
```

You should confidently explain the difference between:

```text
GPU compute
vs
GPU memory
vs
memory bandwidth
vs
inter-GPU bandwidth
```

---

# Tutorial 1.2 — Precision

Learn:

```text
FP32
 ↓
FP16
 ↓
BF16
 ↓
FP8
 ↓
INT8
 ↓
INT4
```

And why:

```text
70B model
×
2 bytes BF16
≈
140 GB weights
```

Meaning an 80 GB GPU cannot even hold the weights alone.

Then architecture decisions appear:

```text
Quantization
or
Tensor Parallelism
or
larger GPU
```

---

# Tutorial 1.3 — CUDA stack

Understand:

```text
Application
     ↓
PyTorch / vLLM
     ↓
CUDA Runtime
     ↓
CUDA Driver
     ↓
NVIDIA Driver
     ↓
GPU
```

And operational failures like:

```text
Driver too old
CUDA mismatch
GPU unavailable
CUDA OOM
NCCL failure
```

---

# Tutorial 1.4 — AWS GPU families

Become comfortable choosing between classes of GPU.

Think:

```text
Prototype / cheaper inference
          ↓
      G-family

Large-scale inference/training
          ↓
      P-family
```

Don't memorize every EC2 SKU.

Learn how to choose based on:

```text
VRAM
GPU count
memory bandwidth
NVLink/NVSwitch
CPU
RAM
network
price
availability
```

---

# LAB 1

Build:

```text
EC2 GPU
 ↓
NVIDIA Driver
 ↓
Docker
 ↓
NVIDIA Container Toolkit
 ↓
nvidia-smi
 ↓
CUDA container
```

You should deliberately break:

```text
driver
CUDA
container runtime
GPU visibility
```

and fix each problem.

That troubleshooting experience is valuable.

---

# LEVEL 2 — vLLM Engineer

This is where we go deep.

## Tutorial 2.1 — First model server

Architecture:

```text
Client
   ↓
OpenAI-compatible API
   ↓
vLLM
   ↓
Model
   ↓
GPU
```

Example conceptually:

```bash
vllm serve <model>
```

Then test:

```text
POST /v1/chat/completions
```

---

# Tutorial 2.2 — KV Cache

Understand this extremely well.

```text
Model weights
████████████████████

KV Cache
████████████

CUDA
████

Workspace
███
```

GPU memory roughly becomes:

```text
weights
+
KV cache
+
activations/workspace
+
CUDA graphs
+
runtime overhead
```

And:

```text
longer context
       ↓
larger KV cache
       ↓
less concurrency
```

---

# Tutorial 2.3 — PagedAttention

Traditional allocation:

```text
Request A
██████████░░░░░░

Request B
████████████████

Request C
████░░░░░░░░░░░
```

Wasted space.

PagedAttention manages cache more like virtual-memory pages:

```text
KV blocks

[A][A][B][C][A][B][C][B]
```

This is fundamental to vLLM.

---

# Tutorial 2.4 — Continuous batching

Without it:

```text
Batch
[A][B][C]

wait until all finish

Next batch
[D][E][F]
```

With continuous batching:

```text
Time ──────────────────────>

A █████████
B █████
C ███████████
          D █████
      E █████████
```

When one request finishes another can enter.

That increases GPU utilization.

---

# Tutorial 2.5 — Important vLLM controls

We will experiment with:

```text
max_model_len
max_num_seqs
gpu_memory_utilization
tensor_parallel_size
dtype
quantization
prefix caching
chunked prefill
```

You won't just learn what they mean.

We'll benchmark each one.

---

# Tutorial 2.6 — Throughput engineering

Metrics:

```text
TTFT
TPOT
E2E latency
requests/sec
input tokens/sec
output tokens/sec
total tokens/sec
KV cache utilization
queue depth
GPU utilization
GPU memory utilization
```

You need to recognize:

```text
Concurrency
     ↓
Throughput increases
     ↓
GPU saturates
     ↓
Queue appears
     ↓
TTFT increases rapidly
```

The sweet spot is where:

```text
maximum useful throughput
while
SLO still passes
```

---

# LAB 2 — Benchmark

Run concurrency tests:

```text
1
2
4
8
16
32
64
128
```

For each collect:

| Concurrency | TTFT | TPOT | Req/s | Tok/s | GPU | KV |
| ----------: | ---: | ---: | ----: | ----: | --: | -: |
|           1 |      |      |       |       |     |    |
|           4 |      |      |       |       |     |    |
|           8 |      |      |       |       |     |    |
|          16 |      |      |       |       |     |    |
|          32 |      |      |       |       |     |    |

At this point, you start thinking like an **AI performance engineer**.

---

# LEVEL 3 — EKS AI Platform Engineer

Now we'll use your strongest existing skills.

Architecture:

```text
                       ALB
                        │
                       EKS
                        │
              ┌─────────┴─────────┐
              │                   │
          CPU Nodes            GPU Nodes
                                  │
                         ┌────────┼────────┐
                         │        │        │
                       vLLM     vLLM     vLLM
                         │        │        │
                        GPU      GPU      GPU
```

---

# Tutorial 3.1 — NVIDIA on Kubernetes

Learn:

```text
GPU Node
   ↓
NVIDIA Driver
   ↓
Container Runtime
   ↓
NVIDIA Device Plugin
   ↓
kubelet
   ↓
nvidia.com/gpu
```

Then:

```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```

---

# Tutorial 3.2 — Scheduling

Become expert in:

```text
nodeSelector
nodeAffinity
podAffinity
antiAffinity
taints
tolerations
topologySpreadConstraints
PriorityClass
PDB
```

Especially:

```text
GPU Type
GPU memory
Availability Zone
Spot/on-demand
```

---

# Tutorial 3.3 — Karpenter

Architecture:

```text
vLLM Pod
   ↓
Pending
   ↓
Needs GPU
   ↓
Karpenter
   ↓
EC2 Fleet
   ↓
GPU instance
   ↓
Node joins
   ↓
vLLM scheduled
```

You'll learn:

* NodePool
* EC2NodeClass
* instance requirements
* capacity type
* consolidation
* interruption handling
* disruption budget
* Spot fallback

---

# Tutorial 3.4 — Model loading

This becomes an important production problem.

Bad:

```text
Pod starts
 ↓
download 150 GB
 ↓
wait
 ↓
serve
```

Instead consider:

```text
Hugging Face/S3
      ↓
model cache
      ↓
local NVMe/EBS
      ↓
vLLM
```

We'll benchmark cold start.

---

# LEVEL 4 — Production Scaling

This is where you stop being "DevOps running AI" and become AIRE.

## Tutorial 4.1 — Don't autoscale solely on GPU %

Bad assumption:

```text
GPU > 80%
    ↓
scale
```

An inference GPU may be close to saturated while still meeting its SLO.

Better signals:

```text
queue depth
waiting requests
TTFT
request latency
KV pressure
request rate
```

---

# Tutorial 4.2 — KEDA/HPA

Architecture:

```text
vLLM metrics
     ↓
Prometheus
     ↓
Prometheus Adapter / KEDA
     ↓
HPA
     ↓
More vLLM pods
```

Then:

```text
Pods cannot schedule
      ↓
Karpenter
      ↓
more GPUs
```

Two layers:

```text
Request pressure
      ↓
Pod scaling
      ↓
Node scaling
```

---

# Tutorial 4.3 — Scale-to-zero

Important for cost but dangerous for latency.

```text
No traffic
 ↓
0 GPU
 ↓

request arrives
 ↓
Pod creation
 ↓
GPU provisioning
 ↓
model loading
 ↓
ready
```

Cold start may become huge.

We'll learn:

```text
minimum replicas
warm pools
model cache
predictive scaling
scheduled capacity
```

Modal-like behavior starts appearing here.

---

# LEVEL 5 — llm-d / Distributed Inference

Now llm-d becomes relevant.

Current llm-d documentation describes it as a Kubernetes-native distributed inference framework coordinating fleets of vLLM instances, providing cluster-level capabilities vLLM itself doesn't aim to provide. ([vLLM][3])

Instead of:

```text
Load Balancer
      ↓
round robin
      ↓
vLLM
```

think:

```text
                    Gateway
                       │
                Intelligent Router
                       │
          ┌────────────┼────────────┐
          │            │            │
       vLLM-1       vLLM-2       vLLM-3
          │            │            │
       KV state     KV state     KV state
```

Routing becomes model/inference-aware.

---

# Tutorial 5.1 — Data parallelism

```text
             Router
         ┌─────┼─────┐
         ↓     ↓     ↓
       GPU1  GPU2  GPU3
       model model model
```

Good when each GPU can hold the model.

---

# Tutorial 5.2 — Tensor parallelism

One model:

```text
Model
  │
 ┌┼┐
 ↓↓↓
GPU GPU GPU GPU
```

Layers/tensors are split across GPUs.

Now NCCL and interconnect matter.

---

# Tutorial 5.3 — Pipeline parallelism

```text
GPU1
Layers 1–20
     ↓
GPU2
Layers 21–40
     ↓
GPU3
Layers 41–60
```

Different tradeoff.

---

# Tutorial 5.4 — Prefill/Decode disaggregation

Important advanced architecture.

```text
Request
   ↓
Router
   │
   ├──────────────┐
   ↓              │
Prefill GPU       │
   ↓              │
KV Transfer       │
   ↓              │
Decode GPU ◄──────┘
```

Why?

Prefill and decode have different computational characteristics.

This allows us to optimize them independently.

---

# LEVEL 6 — AI Reliability Engineer

Now your existing SRE skills get upgraded.

## Golden AI signals

Traditional:

```text
Latency
Traffic
Errors
Saturation
```

AI adds:

```text
TTFT
TPOT
tokens/sec
input tokens
output tokens
queue depth
KV utilization
batch size
GPU compute
GPU memory
model errors
```

---

# Observability architecture

```text
                     vLLM
                       │
                  /metrics
                       │
                    Prometheus
                       │
           ┌───────────┼───────────┐
           │           │           │
         vLLM        DCGM        K8s
        metrics      metrics     metrics
           │           │           │
           └───────────┼───────────┘
                       ↓
                    Grafana
```

Logs:

```text
vLLM
 ↓
Fluent Bit / Alloy
 ↓
Loki
```

Tracing:

```text
Application
 ↓
OpenTelemetry
 ↓
Tempo
```

Eventually:

```text
User request
 ↓
Agent
 ↓
RAG
 ↓
Vector DB
 ↓
LLM gateway
 ↓
vLLM
 ↓
GPU
```

becomes one trace.

---

# SLO Engineering

Example:

```text
Availability:
99.95%

P95 TTFT:
< 800ms

P99 TTFT:
< 1.5 sec

TPOT:
< 50 ms/token

5xx:
< 0.1%
```

Then:

```text
SLI
 ↓
SLO
 ↓
Error Budget
 ↓
Alert
 ↓
Incident
 ↓
Postmortem
```

---

# LEVEL 7 — AI CI/CD and ModelOps

A model deployment is different from an application deployment.

```text
Git
 ↓
CI
 ↓
Unit tests
 ↓
Security scanning
 ↓
Image build
 ↓
ECR
 ↓
Model validation
 ↓
Inference benchmark
 ↓
Evaluation
 ↓
Deploy
 ↓
Canary
 ↓
Production
```

We'll validate:

```text
quality
TTFT
TPOT
throughput
VRAM
errors
cost
```

---

# Canary deployment

```text
                    Gateway
                       │
               ┌───────┴───────┐
               │               │
              95%             5%
               ↓               ↓
           Model V1        Model V2
```

Compare:

```text
quality
latency
cost
errors
```

Then:

```text
5%
 ↓
10%
 ↓
25%
 ↓
50%
 ↓
100%
```

or rollback.

---

# LEVEL 8 — Security

This area will grow substantially because agents are becoming more capable.

Astra itself demonstrates why: OpenAI says it reached its **Critical cybersecurity capability threshold** and required stronger isolation, monitoring and safeguards. ([OpenAI][1])

We should therefore learn:

```text
IAM
IRSA / Pod Identity
Secrets Manager
KMS
network segmentation
private endpoints
mTLS
WAF
API authentication
rate limiting
prompt injection controls
tool authorization
MCP authorization
agent sandboxing
audit logs
supply-chain security
```

And particularly:

```text
Agent
  ↓
Tool permission?
  ↓
IAM authorization
  ↓
Policy
  ↓
Action
```

Never:

```text
LLM decides
 ↓
root credentials
 ↓
production
```

---

# LEVEL 9 — Cost Engineering

This will be one of the strongest Solution Architect skills.

Your optimization objective isn't:

> GPU utilization = 100%.

It is:

> **Lowest cost per successful token/request while meeting the SLO.**

Measure:

```text
$/request

$/1M input tokens

$/1M output tokens

$/1M total tokens
```

Then compare:

```text
H100
H200
A100
L40S
other accelerators
```

along with:

```text
FP16
BF16
FP8
INT8
INT4
```

---

# LEVEL 10 — Solution Architect

Now we'll start doing architecture questions instead of tutorials.

Customer says:

> We have 50 concurrent users.

You ask:

```text
Which model?

Average input tokens?

P95 input tokens?

Average output?

P95 output?

Required context?

Streaming?

Expected concurrency?

Requests/sec?

TTFT target?

TPOT target?

Availability target?

Data sensitivity?

Region?

Growth?

Budget?

Open model or managed API?
```

Then architecture follows.

This is an extremely important mindset:

```text
Requirements
     ↓
Workload characterization
     ↓
Benchmark
     ↓
Capacity model
     ↓
Architecture
     ↓
Cost model
     ↓
SLO
```

**Not:**

```text
Customer wants AI
 ↓
Let's buy H100
```

---

# Your final architecture capability

By the end, I want you able to design something like:

```text
                       Route53
                          │
                        WAF
                          │
                         ALB
                          │
                    AI Gateway
                          │
                  Inference Router
                          │
           ┌──────────────┼──────────────┐
           │              │              │
       Model Pool A   Model Pool B   Model Pool C
           │              │              │
         vLLM           vLLM           vLLM
           │              │              │
         GPU            GPU            GPU
           └──────────────┼──────────────┘
                          │
                         EKS
                          │
          ┌───────────────┼────────────────┐
          │               │                │
      Karpenter          KEDA          llm-d
          │
          ↓
      EC2 GPU Fleet

             Observability
                   │
      ┌────────────┼────────────┐
      ↓            ↓            ↓
 Prometheus       Loki         Tempo
      │            │            │
      └────────────┼────────────┘
                   ↓
                Grafana
```

and defend every decision.

---

# What about Modal?

We should absolutely study it.

But we'll treat Modal as **one implementation of the platform abstraction**.

```text
Developer experience

Modal
──────────────────────────
What gets abstracted?
──────────────────────────
Container lifecycle
GPU allocation
Scheduling
Autoscaling
Model loading
Caching
Networking
Endpoints
Observability
```

Then we'll build the equivalent ourselves:

```text
Modal abstraction
       ↓
Understand internals
       ↓
AWS implementation
       ↓
EKS
       ↓
Karpenter
       ↓
vLLM / llm-d
```

That gives you much deeper career durability.

---

# But Astra changes one thing in our roadmap

Previously I might have emphasized:

```text
Self-host every model
```

I wouldn't recommend that anymore.

A modern AI Solution Architect needs **three serving strategies**:

```text
                        AI Workload
                             │
              ┌──────────────┼──────────────┐
              │              │              │
           Managed         Serverless      Self-hosted
             API              GPU             GPU
              │              │                │
          OpenAI etc.      Modal etc.      EKS/vLLM
```

You must know **when to choose each one**.

For example:

### Managed frontier model

```text
Application
 ↓
GPT-6 Astra
```

Use when frontier capability is the requirement.

### Serverless GPU

```text
Application
 ↓
Modal
 ↓
Open model
```

Excellent when you want minimal platform operations.

### Self-hosted

```text
Application
 ↓
EKS
 ↓
vLLM
 ↓
GPU
```

Makes sense when economics, model control, data requirements, custom inference, scale, or infrastructure ownership justify it.

This hybrid understanding is what will make you a good architect.

---

# Your accelerated curriculum

Because you're already DevOps, I wouldn't spend months on introductory Kubernetes/AWS.

I'd organize our work approximately as:

| Phase             | Main focus                          | Outcome                   |
| ----------------- | ----------------------------------- | ------------------------- |
| **Beginner**      | Transformer + tokens + GPU          | Understand inference      |
| **Beginner+**     | EC2 + CUDA + Docker                 | Operate GPU hosts         |
| **Intermediate**  | vLLM                                | Serve/benchmark models    |
| **Intermediate+** | EKS GPU                             | Build GPU clusters        |
| **Advanced**      | Karpenter/KEDA                      | Production autoscaling    |
| **Advanced**      | Observability/SLO                   | Operate reliably          |
| **Advanced+**     | llm-d/distributed inference         | Scale inference           |
| **Pro**           | Security/CI-CD/FinOps               | Production platform       |
| **Architect**     | Requirement → architecture → sizing | Design customer solutions |

But we're going to overlap them heavily so you learn faster.

---

# And our labs will become progressively harder

We won't just read documentation.

### LAB 01

```text
EC2
+
GPU
+
Docker
+
vLLM
```

### LAB 02

Benchmark one model.

### LAB 03

Understand VRAM mathematically.

### LAB 04

Break vLLM and troubleshoot it.

### LAB 05

Prometheus + DCGM + Grafana.

### LAB 06

Move it onto EKS.

### LAB 07

GPU scheduling.

### LAB 08

Karpenter GPU provisioning.

### LAB 09

KEDA inference scaling.

### LAB 10

Model caching/cold-start optimization.

### LAB 11

Multi-GPU tensor parallelism.

### LAB 12

llm-d.

### LAB 13

Prefill/decode separation.

### LAB 14

Canary model rollout.

### LAB 15

AI SLO/error budgets.

### LAB 16

Chaos testing:

```text
kill vLLM
kill GPU node
kill AZ
break model download
exhaust KV cache
generate traffic spike
```

### LAB 17

FinOps.

### LAB 18

HA architecture.

### LAB 19

Customer capacity sizing.

### LAB 20 — Final

You receive:

> **Design production inference for a 70B model supporting 10,000 users with P95 TTFT <1s, 99.99% availability, CI/CD, automatic scaling, observability and DR.**

You produce:

```text
HLD
LLD
GPU sizing
capacity calculations
architecture
EKS design
vLLM configuration
autoscaling
CI/CD
monitoring
SLO
DR
security
cost model
runbook
```

At that point we're training for **AI Solution Architect / AIRE**, not simply learning another tool.

---

## And yes, I think you're making the right career adjustment

Astra and future models will automate **more application-level work**.

That doesn't eliminate infrastructure engineering. It changes what infrastructure engineers operate:

```text
2015
VMs
 ↓

2020
Kubernetes
 ↓

2024
Cloud-native platforms
 ↓

2026+
AI inference
GPU platforms
agent infrastructure
AI gateways
model routing
AI security
AI observability
AI SRE
```

Some low-level operational work will absolutely become automated. Therefore our goal shouldn't be:

> **person who writes Kubernetes YAML for GPUs.**

It should be:

> **person who understands AI workloads deeply enough to design the reliability, performance, security, capacity and economics of production AI systems.**

That is considerably harder to commoditize.

**We'll start next with `AIRE Level 0 — Lesson 1: From prompt → token → transformer → GPU → KV cache → response`, but we'll teach it from a DevOps/Solution Architect perspective rather than an ML-research perspective.**

[1]: https://openai.com/index/safety-overview-gpt-6-astra/?utm_source=chatgpt.com "Safety overview: GPT-6 Astra | OpenAI"
[2]: https://vllm-project.github.io/2026/07/16/keeping-vllm-production-quality.html?utm_source=chatgpt.com "Keeping vLLM Production Quality: A Look Inside CI, Benchmarking, and the Release Process | vLLM Blog"
[3]: https://docs.vllm.ai/en/latest/deployment/integrations/llm-d/?utm_source=chatgpt.com "llm-d - vLLM"
