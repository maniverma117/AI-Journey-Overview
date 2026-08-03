# Large Language Models — Complete Architecture Deep Dive

## What is an LLM?

A Large Language Model is:

```
Transformer Architecture
+ Billions of Parameters
+ Trained on Trillions of Words
= A system that predicts the next token
```

That's it.

GPT, Claude, Gemini, Llama — all are variations of this formula.

---

## The Complete Architecture

Let's trace what happens from the moment you type "Hello" to the moment the model responds.

```
┌─────────────────────────────────────────────┐
│              YOUR PROMPT                      │
│         "Explain Kubernetes"                 │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│            1. TOKENIZER                      │
│  "Explain" → 1053                           │
│  " Kuber" → 48192                           │
│  "netes" → 923                              │
│                                              │
│  Output: [1053, 48192, 923]                 │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│         2. EMBEDDING LAYER                   │
│  1053 → [0.21, -0.34, 0.87, ... ] (4096d)  │
│  48192 → [-0.18, 0.93, 0.12, ... ] (4096d) │
│  923 → [0.44, -0.62, 0.33, ... ] (4096d)   │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│      3. POSITIONAL ENCODING                  │
│  Add position information to each vector     │
│  Token 1 gets Position 1 pattern            │
│  Token 2 gets Position 2 pattern            │
│  Token 3 gets Position 3 pattern            │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│    4. TRANSFORMER BLOCKS (x80 or more)       │
│  ┌─────────────────────────────┐            │
│  │  Multi-Head Self-Attention   │            │
│  │  + Residual + LayerNorm      │            │
│  ├─────────────────────────────┤            │
│  │  Feed-Forward Network        │            │
│  │  + Residual + LayerNorm      │            │
│  └─────────────────────────────┘            │
│         (repeated 80+ times)                 │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│         5. OUTPUT LAYER                      │
│  Final vector → Score for every token       │
│  in vocabulary (100,000+ scores)            │
│                                              │
│  "is": 8.2                                  │
│  "are": 3.1                                 │
│  "means": 2.8                               │
│  "the": 1.1                                 │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│         6. SAMPLING                          │
│  Convert scores to probabilities             │
│  Pick one token (based on temperature)       │
│                                              │
│  Selected: "is" (highest probability)        │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│         7. APPEND AND REPEAT                 │
│  Input becomes: "Explain Kubernetes is"      │
│  Go back to Step 1                          │
│  Generate next token: "an"                  │
│  Repeat until done                          │
└─────────────────────────────────────────────┘
```

---

## Part 1: Tokenization (Deep Dive)

### Why Not Use Whole Words?

English has millions of possible words:

```
Kubernetes
KubernetesCluster
KubernetesOperator
```

Storing every possible word = impossible.

### BPE (Byte Pair Encoding)

GPT uses around 100,000 subword tokens.

Think of LEGO bricks:

```
"unhappiness" → ["un", "happiness"]
"Kubernetes"  → ["Kuber", "netes"]
"ChatGPT"     → ["Chat", "G", "PT"]
```

With 100,000 pieces, you can build ANY word.

### Important Details

```
" Hello" is NOT the same as "Hello"
```

The space is part of the token!

```
"I love AI" → ["I", " love", " AI"] = 3 tokens
"Supercalifragilistic" → 5-8 tokens (broken into pieces)
```

### Why Tokenization Matters

1. Context Window = max tokens (not max words)
2. API Cost = per token
3. Speed = per token generated
4. Shared subwords help the model understand word families

---

## Part 2: Embeddings (Deep Dive)

### The Embedding Table

The model has a giant lookup table:

```
Token 0:     [0.12, -0.44, 0.81, 0.09, ... ] (4096 numbers)
Token 1:     [-0.55, 0.72, -0.31, 0.28, ... ]
Token 2:     [0.91, -0.12, 0.04, -0.63, ... ]
...
Token 99999: [0.33, 0.18, -0.90, 0.44, ... ]
```

Parameters in embedding layer alone:

```
100,000 tokens x 4,096 dimensions = 409,600,000 parameters
```

Over 400 million numbers just for embeddings!

### Why 4096 Dimensions?

One number cannot capture meaning.

```
"King" = 7     ← tells you nothing
```

4096 numbers can capture rich relationships:

```
"King"  = [0.82, -0.31, 0.94, 0.12, -0.67, ... ]
"Queen" = [0.79, -0.28, 0.91, 0.45, -0.64, ... ]  ← similar!
"Car"   = [-0.44, 0.88, -0.21, 0.33, 0.71, ... ]  ← different!
```

The famous example:

```
King - Man + Woman ≈ Queen
```

This works because vectors encode semantic relationships.

### Embeddings Are LEARNED

Nobody writes these vectors by hand.

They start random. Training adjusts them.

After seeing "dog" and "cat" in similar contexts trillions of times, their vectors become similar.

---

## Part 3: Positional Encoding

Transformers have no built-in sense of order.

Without position encoding:

```
"Dog bites man" = "Man bites dog"  (same tokens!)
```

### RoPE (Rotary Position Embeddings)

Modern LLMs use RoPE. The idea:

```
Rotate each vector based on its position.

Position 1: rotate by angle A
Position 2: rotate by angle 2A
Position 3: rotate by angle 3A
```

Why rotation? The relative distance between positions is preserved regardless of absolute position.

---

## Part 4: Inside the Transformer Block

### Self-Attention Step by Step

For each token:

```
1. Create Q, K, V vectors:
   Q = Input x W_Q  (What am I looking for?)
   K = Input x W_K  (What do I contain?)
   V = Input x W_V  (What info can I provide?)

2. Calculate scores:
   Scores = Q x K^T / sqrt(d_k)

3. Apply causal mask:
   Token 3 can ONLY attend to tokens 1, 2, 3
   Future tokens get -infinity score
   
4. Softmax → probabilities

5. Output = Attention_Probs x V
```

### The Causal Mask

```
Can attend to: Y    Cannot: N

         Token1  Token2  Token3  Token4
Token1:    Y       N       N       N
Token2:    Y       Y       N       N
Token3:    Y       Y       Y       N
Token4:    Y       Y       Y       Y
```

This is why GPT generates left-to-right. Each token only sees the past.

### Feed-Forward Network

```
FFN(x) = GELU(x * W1 + b1) * W2 + b2
```

Typically:

```
4096 → 16384 → 4096 (expand then contract)
```

Attention = "Who should I listen to?"
Feed-Forward = "Now let me think about what I heard."

---

## Part 5: The Output Layer

After all transformer blocks process the sequence:

```
Final hidden state for last position (4096 dimensions)
         │
         ▼
Linear layer (4096 → 100,000)
         │
         ▼
Score for every possible next token
         │
         ▼
Softmax → Probabilities
```

Example:

```
Input: "The capital of France is"

Scores:
  "Paris": 15.2  → probability 0.92
  "Lyon": 8.1   → probability 0.03
  "a": 7.8      → probability 0.02
  ...
```

---

## Part 6: Sampling Strategies

### Temperature

Controls randomness.

```
Temperature = 0.0: Always pick top token (deterministic)
Temperature = 0.7: Mostly pick likely tokens (balanced)
Temperature = 1.0: Sample according to probabilities
Temperature = 2.0: Very random (creative but noisy)
```

Mathematically:

```
adjusted_scores = scores / temperature
```

Low temp = sharpen differences = more predictable
High temp = flatten differences = more creative

### Top-K

Only consider the K most likely tokens.

```
Top-K = 5: Pick from only the top 5 candidates
```

### Top-P (Nucleus Sampling)

Consider tokens until cumulative probability reaches P.

```
Top-P = 0.9:
  "Paris" (0.92) → already past 0.9 → only Paris considered!
```

---

## Part 7: How LLMs Are Trained

### Phase 1: Pre-training (Next Token Prediction)

The task is deceptively simple:

```
Input: "The cat sat on the"
Target: "mat"

Input: "def hello_world():"
Target: "\n"
```

Do this on trillions of tokens from books, code, Wikipedia, websites...

The model learns:
- Grammar and language
- Facts and knowledge
- Reasoning patterns
- Code syntax
- Mathematics
- Multiple languages

ALL from predicting the next token.

### Phase 2: Supervised Fine-Tuning (SFT)

Pre-trained model just continues text. It doesn't follow instructions.

Fine-tuning teaches conversation:

```
Human: What is Python?
Assistant: Python is a high-level programming language...

Human: Write a sort function.
Assistant: Here's a Python sort function:
```

Thousands of high-quality instruction/response pairs.

Now the model follows instructions instead of just completing text.

### Phase 3: RLHF (Reinforcement Learning from Human Feedback)

Model generates multiple responses. Humans rank them.

```
Prompt: "Explain gravity"

Response A: "Gravity is the force that attracts objects..."
Response B: "Yo gravity is like when stuff falls lol"
Response C: "The gravitational force described by Newton..."

Human ranking: A > C > B
```

The model learns to prefer responses humans rate highly.

This is what makes ChatGPT feel helpful, safe, and well-behaved.

---

## Part 8: Model Sizes and What They Mean

### Parameter Counts

```
GPT-2:           1.5 Billion parameters
Llama 3 8B:      8 Billion parameters
GPT-3:           175 Billion parameters  
Llama 3 70B:     70 Billion parameters
Llama 3 405B:    405 Billion parameters
```

### What's Inside Those Parameters?

```
Embedding Matrix:       Vocab size x Dimension
Attention Weights:      Q, K, V matrices per layer per head
Feed-Forward Weights:   Two large matrices per layer
Output Layer:           Dimension x Vocab size
Layer Norm:             Small per layer
```

For a 70B model:

```
~80 transformer layers
~64 attention heads per layer
~8192 hidden dimension
~28672 feed-forward dimension
```

### Storage Size

```
1 parameter = 2 bytes (FP16) or 4 bytes (FP32)

70B parameters x 2 bytes = 140 GB (just weights!)
```

This is why running large models requires multiple GPUs.

---

## Part 9: How Generation Actually Works

When you ask ChatGPT "What is AI?", here's the token-by-token generation:

```
Step 1: Input = "What is AI?"
        → Process through all layers
        → Predict next token: "AI"
        → Wait, append: "AI"

Step 2: Input = "What is AI? AI"
        → Process through all layers  
        → Predict: "stands"

Step 3: Input = "What is AI? AI stands"
        → Predict: "for"

Step 4: Input = "What is AI? AI stands for"
        → Predict: "Artificial"

... and so on, one token at a time
```

Each token requires a FULL forward pass through all 80+ layers.

This is why:
- Generation is slow (one token at a time)
- Longer contexts are slower (more to process each step)
- Streaming shows tokens as they're generated

### KV Cache (Speed Optimization)

Without cache: Each new token recomputes attention for ALL previous tokens.

With KV cache: Store the Key and Value vectors from previous tokens. Only compute for the new token.

```
Without cache: Token 100 → compute attention with all 100 tokens from scratch
With cache:    Token 100 → use cached K,V for tokens 1-99, only compute new one
```

This is why you see "prefill time" (first token) vs "decode time" (subsequent tokens).

---

## Part 10: What the Model Does NOT Do

Common misconceptions:

```
❌ "The model searches the internet"         → No, it uses trained weights
❌ "The model remembers our conversation"     → No, context is re-sent each time
❌ "The model understands meaning"            → It predicts likely next tokens
❌ "The model has opinions"                   → It generates probable text
❌ "The model runs code"                      → It generates code as text
❌ "The model learns from your prompts"       → Weights don't change during inference
```

An LLM is a **statistical next-token predictor** that happens to be so good at prediction that it appears to reason, understand, and converse.

---

## Summary

```
LLM Architecture:
  Tokenizer → Embeddings → Positional Encoding →
  Transformer Blocks (x80+) → Output Layer → Sampling → Token
  
Training:
  Phase 1: Predict next token on trillions of words (pre-training)
  Phase 2: Learn to follow instructions (SFT)
  Phase 3: Align with human preferences (RLHF)

Generation:
  One token at a time, left to right
  Each token = full forward pass through all layers
  KV cache makes it faster

Key numbers:
  Vocabulary: ~100,000 tokens
  Embedding: 4096-12288 dimensions
  Layers: 32-128 transformer blocks
  Parameters: 7B to 400B+
```

---

## Key Takeaways

1. An LLM is a giant next-token predictor built on the Transformer architecture
2. It generates text one token at a time, always left-to-right
3. The model is trained in phases: pre-training → fine-tuning → RLHF
4. All "knowledge" lives in billions of weight parameters (just numbers)
5. The model is stateless — it doesn't remember between conversations
6. Context window is the maximum input+output tokens per request
7. Temperature/Top-P/Top-K control randomness, not correctness

---

## Next → [05-prompt-engineering.md](./05-prompt-engineering.md)

> You have a powerful LLM. But garbage in = garbage out. Prompt Engineering is the art of crafting inputs that make the model produce exactly what you want. The same model can be useless or brilliant depending on how you talk to it.