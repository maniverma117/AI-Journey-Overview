# RAG — Retrieval-Augmented Generation (The Knowledge Base)

## The Problem: LLMs Don't Know YOUR Stuff

An LLM was trained on public internet data.

It knows about Python, Kubernetes, and history.

It does NOT know:

```
❌ Your company's internal runbooks
❌ Your customer documentation
❌ Your codebase
❌ Your policies and procedures
❌ Anything created after its training cutoff
❌ Private or proprietary information
```

If you ask:

```
"What's our procedure for handling a database failover?"
```

The model will either:
- Hallucinate a made-up procedure
- Give a generic answer from the internet
- Say "I don't have access to that information"

---

## The Solution: RAG

> **RAG (Retrieval-Augmented Generation) = Search your documents first, then give the relevant results to the LLM as context, so it can answer based on YOUR actual information.**

It's like giving someone a book before asking them a question:

```
Without RAG:
    "What's our failover procedure?"
    → LLM guesses (hallucination risk)

With RAG:
    Step 1: Search your docs for "failover procedure"
    Step 2: Find the actual runbook page
    Step 3: Give that page to the LLM as context
    Step 4: LLM answers based on the REAL document
    → Accurate answer with citations!
```

---

## How RAG Works (The Big Picture)

RAG has two phases:

### Phase 1: Ingestion (Offline - Prepare Your Knowledge)

```
Your Documents
(PDFs, Wikis, Code, Runbooks)
         │
         ▼
    Split into chunks
    (paragraphs, sections)
         │
         ▼
    Convert to vectors
    (embeddings)
         │
         ▼
    Store in vector database
    (searchable index)
```

### Phase 2: Query (Online - Answer Questions)

```
User asks a question
         │
         ▼
    Convert question to vector
         │
         ▼
    Search vector database
    (find similar chunks)
         │
         ▼
    Retrieve top matches
         │
         ▼
    Add matches to LLM context
         │
         ▼
    LLM generates answer
    (grounded in YOUR documents)
         │
         ▼
    Return answer + citations
```

---

## Step-by-Step: Ingestion Pipeline

### Step 1: Collect Documents

```
Sources:
  - Confluence wiki pages
  - PDF runbooks
  - Markdown documentation
  - Slack threads (archived)
  - Jira ticket resolutions
  - Code README files
```

### Step 2: Parse and Clean

Convert everything to clean text:

```
PDF → Extract text (handle tables, images, headers)
HTML → Strip tags, preserve structure
Markdown → Keep formatting, remove noise
Code → Include comments and docstrings
```

### Step 3: Chunk (The Critical Decision)

You can't feed a 200-page document into an LLM.

You split it into **chunks** — smaller, meaningful pieces.

```
BAD chunking (fixed size, no meaning):
    Chunk 1: "...database. You should always back up be"
    Chunk 2: "fore making changes. Step 1: Connect to th"
    Chunk 3: "e primary instance using..."
    (Words cut mid-sentence!)

GOOD chunking (semantic, meaningful):
    Chunk 1: "Database Backup Procedure
              You should always back up before making changes."
    
    Chunk 2: "Step 1: Connect to the primary instance
              Use the admin credentials from Secrets Manager.
              Run: pg_dump -h primary.db.internal..."
    
    Chunk 3: "Step 2: Verify the backup
              Check the file size is within expected range..."
```

Chunking strategies:

```
By paragraph:     Split at double newlines
By section:       Split at headings (H1, H2, H3)
By sentences:     Fixed number of sentences
By tokens:        Fixed token count with overlap
By semantic:      Use an LLM to identify natural boundaries
```

### Step 4: Add Metadata

Each chunk gets tagged:

```json
{
  "text": "Step 1: Connect to the primary instance...",
  "metadata": {
    "source": "runbook-database-failover.md",
    "section": "Backup Procedure",
    "version": "2.3",
    "last_updated": "2024-06-15",
    "team": "platform",
    "access_level": "internal",
    "customer_scope": "all"
  }
}
```

Metadata enables filtering later!

### Step 5: Create Embeddings

Convert each chunk into a vector (list of numbers):

```
"Connect to the primary database instance"
    ↓
[0.23, -0.87, 0.45, 0.12, -0.33, ... ] (768-1536 dimensions)
```

Similar meaning = similar vectors:

```
"Connect to the main DB server" → [0.21, -0.84, 0.43, ...]  (CLOSE!)
"Order pizza for the team"      → [-0.55, 0.33, 0.91, ...]  (FAR!)
```

### Step 6: Store in Vector Database

```
Vector Database stores:
    Chunk 1: vector + text + metadata
    Chunk 2: vector + text + metadata
    Chunk 3: vector + text + metadata
    ...
    Chunk 50,000: vector + text + metadata
```

Options:
- Pinecone (managed)
- Weaviate (managed/self-hosted)
- pgvector (PostgreSQL extension)
- OpenSearch (with vector support)
- ChromaDB (lightweight)
- FAISS (in-memory, by Meta)

---

## Step-by-Step: Query Pipeline

### Step 1: User Asks a Question

```
"How do I handle a database failover?"
```

### Step 2: Convert Question to Vector

```
"How do I handle a database failover?"
    ↓ (same embedding model used during ingestion)
[0.18, -0.82, 0.41, 0.09, -0.37, ... ]
```

### Step 3: Search Vector Database

Find chunks whose vectors are most similar:

```
Query vector: [0.18, -0.82, 0.41, ...]

Results (sorted by similarity):
  Chunk 42: "Database Failover Procedure..."    similarity: 0.94
  Chunk 43: "Step 1: Verify primary is down..." similarity: 0.91
  Chunk 44: "Step 2: Promote standby..."        similarity: 0.89
  Chunk 17: "Database backup procedures..."     similarity: 0.82
  Chunk 99: "Network failover guide..."         similarity: 0.71
```

### Step 4: Retrieve Top Results

Take the top 3-5 most relevant chunks.

### Step 5: Assemble Context

```
System: You are a helpful assistant. Answer based ONLY on the 
        provided documents. If the answer isn't in the documents, 
        say "I don't have information about that."
        Always cite your sources.

Context Documents:
---
[Source: runbook-database-failover.md, Section: Overview]
Database Failover Procedure: When the primary database becomes 
unavailable, follow these steps to promote the standby...

[Source: runbook-database-failover.md, Section: Step 1]
Step 1: Verify primary is down. Check connectivity using...

[Source: runbook-database-failover.md, Section: Step 2]  
Step 2: Promote standby to primary. Run the following command...
---

User Question: How do I handle a database failover?
```

### Step 6: LLM Generates Answer

```
Based on the database failover runbook, here's the procedure:

1. **Verify primary is down** — Check connectivity using [method from doc]
2. **Promote standby** — Run [command from doc]
3. **Update DNS** — Point applications to new primary

Source: runbook-database-failover.md (v2.3, updated June 2024)
```

Grounded in real documents. With citations.

---

## Why Vector Search Works

Remember embeddings from the LLM chapter?

Words with similar meaning have similar vectors.

Documents work the same way:

```
Question: "How do I handle database failover?"
Vector: [0.18, -0.82, 0.41, ...]

Chunk: "Database Failover Procedure..."
Vector: [0.19, -0.80, 0.39, ...]

Similarity: 0.94 (very close!)
```

The magic: You don't need EXACT keyword matches!

```
Question: "What do I do when the DB goes down?"
    ↓
Still finds: "Database Failover Procedure"

Because the MEANING is similar, even though the WORDS are different.
```

This is called **semantic search** (search by meaning, not keywords).

---

## Hybrid Search: Best of Both Worlds

Vector search alone has weaknesses:

```
Query: "Error code ERR-4521"
Vector search might miss this (it's not about "meaning")
Keyword search finds it instantly!
```

Hybrid search combines both:

```
Hybrid Search = Vector Search (semantic) + Keyword Search (exact match)

1. Vector search: Find semantically similar chunks
2. Keyword search: Find exact term matches
3. Combine and re-rank results
```

---

## RAG vs Fine-Tuning vs Prompt Stuffing

```
┌──────────────────────────────────────────────────────────────┐
│ PROMPT STUFFING                                               │
│ Put all info directly in the prompt.                          │
│                                                               │
│ Works when: Small amount of context (fits in context window)  │
│ Fails when: Too much data, or data changes frequently         │
├──────────────────────────────────────────────────────────────┤
│ RAG                                                           │
│ Search relevant docs, include as context.                     │
│                                                               │
│ Works when: Large knowledge base, changing data, need citations│
│ Fails when: Docs are poor quality, chunking is wrong          │
├──────────────────────────────────────────────────────────────┤
│ FINE-TUNING                                                   │
│ Retrain the model on your data.                              │
│                                                               │
│ Works when: Need consistent style/behavior change             │
│ Fails when: Data changes often, need citations, need privacy  │
└──────────────────────────────────────────────────────────────┘
```

RAG is usually the right choice when you need:
- Up-to-date information
- Source citations
- Privacy (data stays in your system)
- Low cost (no retraining)

---

## Common RAG Failures

### 1. Bad Chunking

```
Problem: Chunks cut in the middle of important context
Result: Retrieved chunk is incomplete, model gives partial answer

Fix: Chunk by semantic boundaries (sections, paragraphs)
     Add overlap between chunks (50-100 tokens)
```

### 2. Irrelevant Retrieval

```
Problem: Top results aren't actually relevant
Result: Model answers based on wrong context

Fix: Better embeddings, metadata filtering, re-ranking
```

### 3. Missing Context

```
Problem: Answer spans multiple chunks but only one is retrieved
Result: Partial or incorrect answer

Fix: Retrieve more chunks, use parent-document retrieval
```

### 4. Stale Data

```
Problem: Document was updated but index wasn't refreshed
Result: Model gives outdated information

Fix: Automated re-indexing pipeline, version tracking
```

### 5. Hallucination Despite Context

```
Problem: Model ignores provided context and makes up an answer
Result: Confident but wrong response

Fix: Strong system prompt ("answer ONLY from documents"),
     lower temperature, output validation
```

### 6. Permission Leaks

```
Problem: User A retrieves documents they shouldn't see
Result: Data breach

Fix: Filter retrieval by user permissions BEFORE returning results
```

---

## Production RAG Architecture

```
┌─────────────────────────────────────────────────────────┐
│                 INGESTION PIPELINE                        │
│                                                          │
│  Documents → Parse → Clean → Chunk → Embed → Store      │
│                                                          │
│  Triggers: New doc, doc updated, scheduled refresh       │
│  Metadata: source, version, team, access level           │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                   QUERY PIPELINE                          │
│                                                          │
│  Question → Embed → Search → Filter by permissions       │
│  → Re-rank → Select top K → Assemble context            │
│  → LLM generates → Validate → Return with citations     │
│                                                          │
│  Guards: Permission check, relevance threshold,          │
│          max chunks, output validation                   │
└─────────────────────────────────────────────────────────┘
```

---

## Evaluation: How Do You Know RAG Is Working?

### Retrieval Quality

```
Precision: Of retrieved docs, how many were relevant?
Recall: Of all relevant docs, how many were retrieved?

Good RAG: High precision AND high recall
```

### Answer Quality

```
Faithfulness: Does the answer match the retrieved documents?
             (Not hallucinating beyond what was provided)

Relevance: Does the answer address the user's question?

Completeness: Does the answer cover all important points?
```

### Test with known questions:

```
Q: "What's the escalation procedure for P1 incidents?"
Expected: Should retrieve incident-procedures.md, section 4
          Should mention: 15-min SLA, notify VP, bridge call

Actual: [check if correct documents were retrieved]
        [check if answer matches expected content]
```

---

## Summary

```
RAG = Search your documents + Give results to LLM as context

Two phases:
  INGESTION: Documents → Chunks → Vectors → Database
  QUERY: Question → Vector → Search → Context → LLM → Answer

Why it works:
  - Semantic search finds relevant content by meaning
  - LLM synthesizes a natural language answer
  - Citations make answers verifiable

Key decisions:
  - How to chunk (semantic boundaries, overlap)
  - How to search (vector, keyword, hybrid)
  - How many results to include
  - How to handle permissions
  - How to keep the index fresh
```

---

## Key Takeaways

1. RAG gives LLMs access to private/current knowledge without retraining
2. Ingestion pipeline: Documents → Chunks → Embeddings → Vector Store
3. Query pipeline: Question → Embed → Search → Context → LLM → Answer
4. Chunking strategy is the most impactful design decision
5. Hybrid search (vector + keyword) outperforms either alone
6. Permission-aware retrieval prevents data leaks
7. Always include citations — let users verify answers
8. Evaluate both retrieval quality AND answer quality separately

---

## Next → [12-agentic-frameworks.md](./12-agentic-frameworks.md)

> You now understand ML, Deep Learning, Transformers, LLMs, Prompts, Context, Harnesses, Tools, Agents, MCP, and RAG. But building all of this from scratch for every project is exhausting. Agentic AI Frameworks (LangChain, LangGraph, CrewAI, AutoGen, Strands) provide building blocks so you don't reinvent the wheel.
