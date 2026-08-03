# Machine Learning — Teaching Machines to Learn from Data

## The Problem Before Machine Learning

Imagine you're a programmer in the 1980s.

Your boss says:

```
Build a system that detects spam emails.
```

You write rules.

```
IF subject contains "FREE MONEY"
    → SPAM

IF sender is unknown
    → SPAM

IF body contains "Click here to win"
    → SPAM
```

Works for a week.

Then spammers change their wording.

```
"Fr33 M0ney"
"F.R.E.E G.I.F.T"
"Congratulations! You've been selected"
```

Your rules break.

You write more rules.

100 rules.

500 rules.

1000 rules.

Still breaks.

---

## The Fundamental Limitation

Rule-based systems have one fatal flaw:

> **A human must anticipate every possible scenario.**

The world is too complex.

Languages change.

Patterns shift.

Edge cases are infinite.

---

## The Revolutionary Idea

What if instead of writing rules...

We showed the computer **examples**?

```
Email 1: "Free money now!" → SPAM
Email 2: "Meeting at 3pm"  → NOT SPAM
Email 3: "Win a prize!!!"  → SPAM
Email 4: "Project update"  → NOT SPAM
Email 5: "Claim your reward" → SPAM
...
```

10,000 examples.

Then we say:

> **"Computer, figure out the pattern yourself."**

This is Machine Learning.

---

## What is Machine Learning?

Machine Learning is:

> **The computer discovers patterns from data, instead of being programmed with explicit rules.**

```
Traditional Programming:
    Rules + Data → Answer

Machine Learning:
    Data + Answers → Rules (learned automatically)
```

Read that again.

In traditional programming, humans write the rules.

In machine learning, the **computer discovers** the rules.

---

## How Does It Actually Work?

Let's use a real example.

Suppose we want to predict house prices.

### Step 1: Collect Data

```
House 1: 1000 sq ft, 2 bedrooms → $200,000
House 2: 1500 sq ft, 3 bedrooms → $300,000
House 3: 2000 sq ft, 4 bedrooms → $400,000
House 4: 800 sq ft, 1 bedroom  → $150,000
House 5: 2500 sq ft, 5 bedrooms → $500,000
```

### Step 2: Choose a Model

The simplest model is a line.

```
Price = W1 × Size + W2 × Bedrooms + Bias
```

W1, W2, and Bias are **unknown numbers**.

Initially random.

```
W1 = 0.5  (random)
W2 = 3.0  (random)
Bias = 10  (random)
```

### Step 3: Make a Prediction

Input: 1000 sq ft, 2 bedrooms

```
Price = 0.5 × 1000 + 3.0 × 2 + 10
     = 500 + 6 + 10
     = 516
```

Predicted: $516

Actual: $200,000

**Terrible.**

### Step 4: Measure the Error

```
Error = Predicted - Actual
     = 516 - 200,000
     = -199,484
```

Very wrong.

### Step 5: Adjust the Weights

The algorithm slightly changes W1, W2, and Bias.

```
W1: 0.5 → 50
W2: 3.0 → 5000
Bias: 10 → 1000
```

Try again.

```
Price = 50 × 1000 + 5000 × 2 + 1000
     = 50,000 + 10,000 + 1000
     = 61,000
```

Still wrong. But **less wrong**.

### Step 6: Repeat Thousands of Times

After thousands of adjustments:

```
W1 = 180
W2 = 10,000
Bias = 5,000
```

Now:

```
Price = 180 × 1000 + 10,000 × 2 + 5,000
     = 180,000 + 20,000 + 5,000
     = 205,000
```

Close to $200,000!

---

## What Just Happened?

The computer discovered:

```
Each square foot adds about $180 to price.
Each bedroom adds about $10,000 to price.
Base price is about $5,000.
```

**Nobody told it this.**

It figured it out from data.

Those final numbers (180, 10000, 5000) are called **learned parameters** or **weights**.

---

## The Training Process

```
Start with random weights
        │
        ▼
Make a prediction
        │
        ▼
Compare with actual answer
        │
        ▼
Calculate error
        │
        ▼
Adjust weights slightly
        │
        ▼
Repeat millions of times
        │
        ▼
Weights become good
        │
        ▼
Model is "trained"
```

This process is called **training**.

The algorithm that adjusts weights is called **gradient descent**.

---

## Key Vocabulary

### Features

The inputs you give the model.

```
Size = 1000 sq ft      ← Feature
Bedrooms = 2           ← Feature
```

### Labels

The correct answers in training data.

```
Price = $200,000       ← Label
```

### Training Data

Examples with both features and labels.

### Model

The mathematical formula with learnable weights.

### Parameters / Weights

The numbers the model learns during training.

### Prediction / Inference

Using the trained model on new data.

---

## Types of Machine Learning

### Supervised Learning

You give the model **examples with answers**.

```
Input → Correct Answer
Input → Correct Answer
Input → Correct Answer
```

The model learns the mapping.

Examples:
- Spam detection (email → spam/not spam)
- Price prediction (features → price)
- Image classification (image → cat/dog)

### Unsupervised Learning

You give the model **data without answers**.

```
Input
Input
Input
```

The model finds patterns on its own.

Examples:
- Customer grouping (find similar customers)
- Anomaly detection (find unusual patterns)

### Reinforcement Learning

The model learns by **trial and error**.

```
Action → Reward/Punishment
Action → Reward/Punishment
```

Examples:
- Game playing (AlphaGo)
- Robot control

---

## A Real Example: Iris Flower Classification

This is the most famous ML dataset.

We have 150 flowers. Each has 4 measurements.

```
Sepal Length = 5.1
Sepal Width = 3.5
Petal Length = 1.4
Petal Width = 0.2

Species = Setosa
```

Three species exist:

```
Setosa
Versicolor
Virginica
```

The model learns:

```
IF Petal Length is small AND Petal Width is small
    → Probably Setosa

IF Petal Length is medium
    → Probably Versicolor

IF Petal Length is large
    → Probably Virginica
```

But it discovers this **from numbers**, not from rules.

After training:

```
New flower:
    Petal Length = 5.0
    Petal Width = 1.8

Model predicts: Versicolor ✓
```

---

## What Does a Trained Model Contain?

This surprises most people.

A trained model is just **a file full of numbers**.

```
W1 = 8.14
W2 = -2.76
W3 = 4.81
W4 = 1.92
Bias = 0.33
```

That's it.

No English.

No rules.

No intelligence.

Just numbers that, when combined with inputs using math, produce accurate predictions.

---

## Training vs Inference

### Training (Learning Phase)

```
Lots of data
+ Random model
+ Time (hours/days)
= Trained model (good weights)
```

Expensive. Slow. Done once.

### Inference (Using Phase)

```
New input
+ Trained model
+ Math
= Prediction
```

Fast. Cheap. Done millions of times.

---

## When Machine Learning Works

✅ You have lots of examples

✅ The pattern exists in the data

✅ The pattern is too complex for manual rules

✅ The pattern doesn't change too quickly

## When Machine Learning Fails

❌ Too little data

❌ Data is biased or wrong

❌ The pattern changes faster than you retrain

❌ You need explainable decisions (sometimes)

❌ The problem is simple enough for IF/THEN rules

---

## The Limitation That Led to Deep Learning

Machine Learning works great for **simple patterns**.

```
4 features → Prediction
```

But what about images?

An image is 100×100 pixels = 10,000 numbers.

For ML to work on images, a **human** must decide what features matter:

```
Feature 1: Average brightness
Feature 2: Number of edges
Feature 3: Color distribution
Feature 4: Texture pattern
```

This is called **feature engineering**.

It requires domain expertise.

It's slow.

It's limited.

What if the computer could discover **features by itself**?

That's Deep Learning.

---

## Summary

```
Before ML:     Humans write rules explicitly
After ML:      Computers learn rules from data

What's stored: Numbers (weights/parameters)
How it learns: Adjust weights until predictions match reality
Key idea:      Data + Algorithm = Pattern Discovery
Limitation:    Humans still pick the features
```

---

## Key Takeaways

1. ML learns patterns from examples instead of following programmed rules
2. A trained model is just a file of numbers (weights)
3. Training = adjusting weights millions of times until accurate
4. Inference = using those weights to predict on new data
5. The big limitation: humans must define what features to look at

---

## Next → [02-deep-learning.md](./02-deep-learning.md)

> Deep Learning removes the need for manual feature engineering. The model discovers features **by itself** using layers of neurons.
