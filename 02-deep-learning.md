# Deep Learning — Machines That Discover Their Own Features
![1785757192205](image/02-deep-learning/1785757192205.png)

## The Problem with Machine Learning

Machine Learning was revolutionary.

But it had one critical weakness.

---

Suppose you want to recognize cats in photos.

An image is just numbers.

```
Pixel 1: 132
Pixel 2: 140
Pixel 3: 98
...
Pixel 10000: 255
```

For ML to work, a human must define **features**:

```
Feature 1: Edge count
Feature 2: Average color
Feature 3: Symmetry score
Feature 4: Texture roughness
```

This is called **feature engineering**.

The problem?

A human decides what's important **before** the model learns.

What if the human picks the wrong features?

What if important features are things no human would think of?

---

## The Deep Learning Revolution

What if the model could discover features **by itself**?

Not just learn weights.

But learn **what to look at** in the first place.

This is Deep Learning.

---

## From One Calculation to Many Layers

Remember Machine Learning?

```
Input → Weights → Prediction
```

One step.

Deep Learning adds **layers**.

```
Input → Layer 1 → Layer 2 → Layer 3 → Prediction
```

Each layer discovers something different.

---

## The Factory Analogy

Imagine building a car.

You don't do everything in one room.

```
Room 1: Raw metal → Cut parts
Room 2: Cut parts → Welded frame  
Room 3: Welded frame → Painted body
Room 4: Painted body → Assembled car
```

Each room transforms material into something more useful.

A neural network works the same way.

```
Layer 1: Pixels → Edges
Layer 2: Edges → Shapes
Layer 3: Shapes → Parts (eyes, ears)
Layer 4: Parts → Object (cat!)
```

Nobody programs "find edges" or "find eyes."

The network **discovers** these intermediate representations during training.

---

## What is a Neuron?

Forget biology.

A neuron is a tiny calculator.

```
Inputs: X1, X2, X3, X4
Weights: W1, W2, W3, W4

Calculation:
    Sum = X1×W1 + X2×W2 + X3×W3 + X4×W4 + Bias

    Output = Activation(Sum)
```

That's all.

A weighted sum followed by an activation function.

---

## Example: One Neuron

Inputs:

```
X1 = 5 (Petal Length)
X2 = 2 (Petal Width)
X3 = 6 (Sepal Length)
X4 = 3 (Sepal Width)
```

Weights (learned):

```
W1 = 8
W2 = 3
W3 = 1
W4 = 0.5
```

Calculation:

```
Sum = 5×8 + 2×3 + 6×1 + 3×0.5
    = 40 + 6 + 6 + 1.5
    = 53.5
```

Then apply activation function (we'll explain why next).

---

## Why Activation Functions?

Without activation, neurons just do linear math.

```
Y = W1×X1 + W2×X2
```

This is a straight line.

But the real world isn't linear!

Cat vs Dog boundaries aren't straight lines.

Activation functions add **curves**.

### ReLU (Most Common)

```
If input > 0: output = input
If input ≤ 0: output = 0
```

Example:

```
53.5 → 53.5 (positive, stays)
-2.3 → 0    (negative, becomes zero)
```

Simple. But it lets the network learn complex, non-linear patterns.

### Sigmoid

```
Squashes any number to between 0 and 1.

-100 → 0.0000
0    → 0.5
100  → 0.9999
```

Used when you need a probability output.

### GELU (Used in Transformers/GPT)

Similar to ReLU but smoother. Used in modern language models.

---

## What is a Layer?

One neuron sees one pattern.

But we need to see many patterns simultaneously.

A **layer** is many neurons working in parallel.

```
Layer 1 (4 neurons):

    Neuron 1: detects horizontal edges
    Neuron 2: detects vertical edges
    Neuron 3: detects diagonal edges
    Neuron 4: detects color changes
```

Each neuron has **different weights**.

So each learns to detect a **different pattern**.

Nobody assigns these roles. Training makes them specialize.

---

## Stacking Layers: Where "Deep" Comes From

```
Image (pixels)
      │
      ▼
Layer 1 (64 neurons)
    Finds: edges, gradients, simple textures
      │
      ▼
Layer 2 (128 neurons)
    Finds: corners, curves, circles
      │
      ▼
Layer 3 (256 neurons)
    Finds: eyes, ears, wheels, windows
      │
      ▼
Layer 4 (128 neurons)
    Finds: faces, car fronts, buildings
      │
      ▼
Output Layer
    Predicts: Cat / Dog / Car
```

**"Deep"** learning means many layers.

Each layer builds on the previous one.

Simple patterns → Complex patterns → Concepts → Final answer.

---

## How Many Parameters?

Each connection between neurons has a weight.

```
Layer 1: 100 neurons × 784 inputs = 78,400 weights
Layer 2: 200 neurons × 100 inputs = 20,000 weights
Layer 3: 50 neurons × 200 inputs  = 10,000 weights
Output: 10 neurons × 50 inputs    = 500 weights

Total = 108,900 parameters
```

This is a **small** network.

Modern networks:

```
Image Recognition (ResNet):    25 million parameters
GPT-2:                         1.5 billion parameters
GPT-3:                         175 billion parameters
Llama 3 (large):               405 billion parameters
```

More parameters = more capacity to learn complex patterns.

---

## Training a Deep Network

Same idea as ML, but scaled up massively.

### Forward Pass

```
Input image
    → Layer 1 calculations
    → Layer 2 calculations
    → Layer 3 calculations
    → Prediction: "Dog" (70% confidence)

Actual answer: "Cat"

WRONG!
```

### Calculate Loss

```
Loss = How wrong was the prediction?

Expected: Cat (100%)
Got: Dog (70%), Cat (20%), Bird (10%)

Loss = High (bad prediction)
```

### Backpropagation (The Key Innovation)

The network asks:

> "Which of my millions of weights contributed most to this mistake?"

It traces the error backward through every layer.

```
Output Layer: These weights were most wrong
    ↓
Layer 3: These weights amplified the error
    ↓
Layer 2: These weights started the mistake
    ↓
Layer 1: These weights missed the feature
```

Then it adjusts **every weight** slightly.

This is called **backpropagation** — the algorithm that makes deep learning possible.

### Repeat

```
Show image 1 → Forward → Error → Backward → Adjust
Show image 2 → Forward → Error → Backward → Adjust
Show image 3 → Forward → Error → Backward → Adjust
...
Show image 1,000,000 → Almost perfect!
```

---

## Why GPUs?

Each neuron does simple math (multiply and add).

But there are **millions** of them.

A CPU does calculations one at a time.

A GPU does **thousands simultaneously**.

```
CPU:  1 + 1, then 2 + 2, then 3 + 3 ...
GPU:  1+1, 2+2, 3+3, 4+4, 5+5 ... ALL AT ONCE
```

This is why NVIDIA became a trillion-dollar company.

Deep Learning needs parallel math.

GPUs provide it.

---

## What Deep Learning Can Do

| Task | Input | Output |
|------|-------|--------|
| Image Classification | Photo | "Cat" |
| Object Detection | Photo | Boxes around objects |
| Speech Recognition | Audio | Text |
| Translation | English text | French text |
| Anomaly Detection | Server logs | "Unusual pattern!" |

---

## The Mental Model

```
Traditional ML:
    Human picks features → Model learns weights

Deep Learning:
    Model discovers features AND learns weights
    (All automatically from data)
```

This is why Deep Learning dominated after 2012.

It removed the human bottleneck of feature engineering.

---

## The Limitation: Sequential Understanding

Deep Learning excels at:

✅ Images (every pixel is processed together)

✅ Fixed-size inputs

But what about **language**?

```
"The cat sat on the mat because it was tired"
```

What does "it" refer to?

- The cat?
- The mat?

To answer this, you need to understand **relationships between words far apart**.

Early deep learning processed sequences word by word:

```
The → Cat → Sat → On → The → Mat → Because → It → ...
```

By the time it reaches "it", it might have forgotten "cat" was mentioned earlier.

This is called the **vanishing gradient problem** in RNNs (Recurrent Neural Networks).

Long sequences = lost context.

---

## What Came Next

In 2017, a paper called **"Attention Is All You Need"** introduced a completely new architecture.

Instead of reading words one at a time...

Every word looks at **every other word simultaneously**.

```
"It" can directly ask: "Who am I referring to?"
"Cat" answers: "Me! I'm the one who is tired."
```

This is the **Transformer**.

And it changed everything.

---

## Summary

```
Machine Learning:  Human picks features, model learns weights
Deep Learning:     Model discovers features AND weights automatically

Key ideas:
- Neurons = tiny calculators (weighted sum + activation)
- Layers = many neurons in parallel
- Deep = many layers stacked
- Training = forward pass + error + backpropagation + weight update
- GPUs = parallel math that makes it feasible

Limitation: Struggled with long sequences and relationships between distant words
```

---

## Key Takeaways

1. Deep Learning removes manual feature engineering — the model discovers features itself
2. Stacking layers lets the model build simple patterns → complex concepts
3. Backpropagation is the algorithm that traces errors back and adjusts millions of weights
4. More parameters = more capacity (but needs more data and compute)
5. GPUs made deep learning practical by enabling massive parallel computation
6. The limitation was understanding long-range relationships in sequences (text, speech)

---

## Next → [03-transformers.md](./03-transformers.md)

> Transformers solve the sequence problem. Instead of reading word by word, every word can attend to every other word simultaneously. This is the architecture behind GPT, Claude, and Gemini.
