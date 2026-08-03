# Transformers — The Architecture That Changed Everything

## The Problem with Previous Approaches

Before Transformers, we used RNNs (Recurrent Neural Networks) for language.

RNNs read text one word at a time.

```
"The" → process → "cat" → process → "sat" → process → "on" → process ...
```

Two fatal problems:

### Problem 1: Forgetting

By the time the RNN reaches word 50, it has mostly forgotten word 1.

```
"The cat that my neighbor who lives across the street adopted 
last summer from the shelter on Main Street sat on the mat 
because it was tired."
```

When the model reaches "it" — does it still remember "cat"?

Often, no.

### Problem 2: Slow Training

Words are processed **sequentially**.

```
Word 1 → then Word 2 → then Word 3 → ...
```

Can't parallelize.

Training takes forever.

---

## The Breakthrough: Attention Is All You Need (2017)

A team at Google published a paper that changed AI.

Their key insight:

> **Why read one word at a time when every word can look at every other word simultaneously?**

---

## The Core Idea: Self-Attention

Imagine a classroom.

In the old system (RNN):

```
Student 1 whispers to Student 2
Student 2 whispers to Student 3
Student 3 whispers to Student 4
...
```

By Student 20, the message is garbled.

In the new system (Transformer):

```
Every student can directly talk to every other student.
At the same time.
```

Student 20 can directly ask Student 1: "What did you say?"

No whisper chain. No information loss.

---

## How Self-Attention Works

Let's take a simple sentence:

```
"The cat sat because it was tired"
```

The question is: What does **"it"** refer to?

### Step 1: Every Word Creates Three Vectors

Each word generates:

```
Query (Q): "What am I looking for?"
Key (K):   "What do I contain?"
Value (V): "What information can I provide?"
```

Think of it like a library:

```
Query = Your search question
Key = Book title (does this match your question?)
Value = Book content (the actual information)
```

### Step 2: Every Word Asks Every Other Word

The word "it" creates a Query:

```
"it" Query: "Who am I referring to? I need a noun."
```

Every other word has a Key:

```
"The" Key: "I'm an article"
"cat" Key: "I'm an animal noun"
"sat" Key: "I'm a verb"
"because" Key: "I'm a conjunction"
"was" Key: "I'm a helper verb"
"tired" Key: "I'm an adjective"
```

### Step 3: Calculate Attention Scores

"it" compares its Query against every Key:

```
"it" → "The":     Score = 0.02 (low match)
"it" → "cat":     Score = 0.85 (high match!)
"it" → "sat":     Score = 0.05 (low match)
"it" → "because": Score = 0.01 (low match)
"it" → "was":     Score = 0.03 (low match)
"it" → "tired":   Score = 0.04 (low match)
```

"cat" has the highest score!

### Step 4: Weighted Sum of Values

Now "it" takes the Values from every word, weighted by attention:

```
Output for "it" = 0.02 × Value("The")
                + 0.85 × Value("cat")    ← mostly this!
                + 0.05 × Value("sat")
                + 0.01 × Value("because")
                + 0.03 × Value("was")
                + 0.04 × Value("tired")
```

Result: The output vector for "it" is now **rich with information about "cat"**.

The model has learned that "it" refers to "cat" — not by a rule, but by learned attention patterns.

---

## The Math (Simplified)

```
Attention(Q, K, V) = softmax(Q × K^T / √d) × V
```

Let's break it down:

```
Q × K^T       → Compare every query with every key
                 (how relevant is each word to each other word?)

/ √d          → Scale down (prevent numbers from getting too large)

softmax(...)  → Convert scores to probabilities (sum to 1)

× V           → Weighted combination of values
```

That's the entire self-attention mechanism.

---

## Why "Self" Attention?

Because the sentence attends to **itself**.

Every word in the input looks at every other word in **the same** input.

```
Word 1 → looks at → Word 1, Word 2, Word 3, ... Word N
Word 2 → looks at → Word 1, Word 2, Word 3, ... Word N
Word 3 → looks at → Word 1, Word 2, Word 3, ... Word N
...
```

N words × N comparisons = N² attention calculations.

This is why Transformers are powerful but expensive for long sequences.

---

## Multi-Head Attention

One attention head might learn:

```
Head 1: "Who is the subject?"
Head 2: "What is the action?"
Head 3: "Where is it happening?"
Head 4: "When did it happen?"
```

Multiple heads look at different types of relationships **simultaneously**.

```
Multi-Head Attention = Concat(Head1, Head2, ..., Head8) × W

Each head has its own Q, K, V weight matrices.
```

GPT-3 uses 96 attention heads.

Each head learns to focus on different linguistic patterns.

---

## The Complete Transformer Block

A single Transformer block:

```
Input Vectors
      │
      ▼
┌─────────────────────┐
│  Multi-Head          │
│  Self-Attention      │
└─────────────────────┘
      │
      + (Residual Connection: add original input back)
      │
      ▼
┌─────────────────────┐
│  Layer Normalization │
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  Feed-Forward        │
│  Neural Network      │
│  (2 dense layers)    │
└─────────────────────┘
      │
      + (Residual Connection)
      │
      ▼
┌─────────────────────┐
│  Layer Normalization │
└─────────────────────┘
      │
      ▼
Output Vectors
```

### Why Residual Connections?

```
Output = Layer(Input) + Input
```

This lets gradients flow directly through the network.

Without it, very deep networks (80+ layers) can't train — gradients vanish.

### Why Layer Normalization?

Keeps numbers in a stable range so training doesn't explode or collapse.

### Why Feed-Forward Network?

Attention finds relationships between words.

The feed-forward network **processes** each word's information independently.

Think of it as "thinking time" for each position.

---

## Stacking Transformer Blocks

GPT doesn't use one block.

It stacks many.

```
Input Embeddings
      │
      ▼
Transformer Block 1
      │
      ▼
Transformer Block 2
      │
      ▼
Transformer Block 3
      │
      ...
      ▼
Transformer Block 96  (GPT-3)
      │
      ▼
Output
```

Each block refines understanding.

Block 1: Basic word relationships

Block 10: Grammar patterns

Block 40: Semantic meaning

Block 80: Complex reasoning

---

## Positional Encoding: How Does It Know Word Order?

Attention has no built-in sense of position.

"Dog bites man" and "Man bites dog" would look the same!

Solution: Add position information to each word's embedding.

```
Embedding("cat") + Position(3) = Final Input for word "cat" at position 3
```

This tells the model: "cat" is the 3rd word.

Different positions get different mathematical patterns, allowing the model to learn that word order matters.

---

## Encoder vs Decoder

The original Transformer paper had two parts:

### Encoder (Understands Input)

- Reads the full input
- Every word attends to every other word (bidirectional)
- Used in: BERT, sentence understanding

### Decoder (Generates Output)

- Generates one token at a time
- Each word can only attend to **previous** words (causal/masked)
- Used in: GPT, Claude, text generation

```
Encoder:  "I love AI" → [understands full sentence]

Decoder:  "I" → "love" → "AI" → "because" → "it" → ...
          (generates one word at a time, left to right)
```

GPT models use **decoder-only** architecture.

They can only look backward, never forward.

---

## Why Transformers Won

| Feature | RNN | Transformer |
|---------|-----|-------------|
| Reads input | One word at a time | All words at once |
| Training speed | Slow (sequential) | Fast (parallel) |
| Long-range memory | Forgets distant words | Direct attention to any word |
| Scalability | Limited | Scales with compute |

Transformers can be trained in parallel on GPUs.

RNNs cannot.

This made it possible to train on **trillions** of words.

Which led to Large Language Models.

---

## The Cost of Attention

Nothing is free.

```
Sequence Length = N
Attention Cost = N × N = N²
```

```
N = 100 words    → 10,000 calculations
N = 1,000 words  → 1,000,000 calculations
N = 100,000 words → 10,000,000,000 calculations
```

This is why context windows have limits.

It's also why researchers work on efficient attention variants.

---

## Putting It All Together

When you type into ChatGPT:

```
"Explain Kubernetes"
```

Here's what happens inside the Transformer:

```
1. Tokenize: "Explain" → 1053, "Kubernetes" → [48192, 923]

2. Embed: Each token ID → Vector (e.g., 4096 numbers)

3. Add Position: Each vector gets position information

4. Pass through 80+ Transformer Blocks:
   - Self-Attention: tokens relate to each other
   - Feed-Forward: process information
   - Repeat many times

5. Output Layer: Predict next token
   - "Kubernetes" → "is" (highest probability)
   
6. Append "is" and repeat from step 3
   - "Kubernetes is" → "an"
   
7. Continue until done:
   - "Kubernetes is an open-source container orchestration..."
```

---

## Summary

```
Before Transformers: Read sequences word by word (slow, forgetful)
After Transformers:  Every word attends to every other word (parallel, no forgetting)

Key Components:
- Self-Attention: Words find relationships to other words (Q, K, V)
- Multi-Head: Multiple attention patterns simultaneously
- Feed-Forward: Process information at each position
- Residual Connections: Help deep networks train
- Positional Encoding: Preserve word order
- Stacked Blocks: Deeper understanding with each layer

The breakthrough: Parallelizable + long-range attention + scalable
```

---

## Key Takeaways

1. Transformers let every word attend to every other word simultaneously
2. Self-Attention uses Query, Key, Value to find relationships
3. Multi-Head Attention captures different types of relationships in parallel
4. Transformer Blocks stack to build deeper understanding
5. The architecture is parallelizable — enabling training on massive data
6. Cost scales as N² with sequence length (context window tradeoff)
7. GPT uses decoder-only (generates left to right, one token at a time)

---

## Next → [04-llm-architecture.md](./04-llm-architecture.md)

> What happens when you take the Transformer architecture and train it on trillions of words? You get a Large Language Model. Let's see the complete architecture of GPT, how it's trained, and how it generates text.
