## 🔍 What is Attention?

In a Transformer, **attention** helps a word (or token) focus on *other important words* when trying to understand the meaning of a sentence.

For example:

> *"The cat sat on the mat."*

When processing “sat”, the model might look at “cat” to understand who is sitting. That’s attention.

---

## 💡 What is Sparse Attention?

**Sparse Attention** means:

> *“Don’t look at **every** word—just look at a **few** important ones.”*

Regular attention (called **full attention**) looks at **all** the tokens in a sequence. If the sentence is long, that becomes very expensive (slow and memory-heavy).

So, sparse attention is like:

> “Let’s save time and memory by only attending to nearby words or a few important global words.”

---

## 🍕 Analogy: Ordering Pizza

Imagine you’re throwing a party and want to ask your **friends** what pizza to order.

* **Full Attention**: You ask **everyone** at the party, even people you don’t know well. That takes time.
* **Sparse Attention**: You only ask the people **near you** or the ones who **always have good taste** (like your foodie friend). That’s faster and usually good enough.

---

## ✅ Types of Sparse Attention (Examples)

1. **Local Attention**:

   * Only look at nearby tokens.
   * Example: In the sentence `"The cat sat on the mat"`, when looking at `"sat"`, only check `"cat"` and `"on"`.

2. **Global Tokens**:

   * Some special tokens (like headings or keywords) can attend to *everything*, and everything can attend to them.
   * Example: In a document, the word `"Title:"` might be a global token. All other tokens can look at it, no matter where it is.

3. **Strided Attention**:

   * Look at every 2nd or 3rd token.
   * Example: Look at token 0, 2, 4, 6, etc.

---

## 🧠 Why Use Sparse Attention?

Because attention across **long sequences** is **slow** and uses **a lot of memory**.

| Attention Type | Speed  | Memory Use | Accuracy                  |
| -------------- | ------ | ---------- | ------------------------- |
| Full           | ❌ Slow | ❌ High     | ✅ High                    |
| Sparse         | ✅ Fast | ✅ Low      | ✅ Good (if designed well) |

That's why models like **Longformer**, **BigBird**, and your **VerticalSlashAttention** use sparse attention to scale better.

---

## 👀 Vertical Slash Attention (your example)

Your code does something like this:

1. Each token looks at a **window** of nearby tokens (local attention).
2. Also lets every token look at some **global tokens**.

So it’s like saying:

> “I’ll mostly look around me, but I’ll also glance at the important headers.”

---

## 🧸 Sentence:

> `"The quick brown fox jumps over the lazy dog"`

Let’s say we convert each word into a **token**. So we have:

```
[0] The  
[1] quick  
[2] brown  
[3] fox  
[4] jumps  
[5] over  
[6] the  
[7] lazy  
[8] dog
```

Let’s imagine we’re applying **sparse attention with a local window size of 2**:

* Each token can only “see” **itself** and **2 tokens before and after** it.
* This is **local attention**.

---

### 🧠 Full Attention (Just for comparison)

In full attention, every token attends to **all tokens**:

|     | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
| --- | - | - | - | - | - | - | - | - | - |
| 0   | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| 1   | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| 2   | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| ... |   |   |   |   |   |   |   |   |   |
| 8   | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |

Too much for long sequences! ❌

---

### ✅ Sparse Attention (Window = 2)

Let’s define a simple rule:

* A token at position `i` can attend to: `i-2, i-1, i, i+1, i+2` (within bounds).

Example:
For token `fox [3]`, it can attend to `[1] quick`, `[2] brown`, `[3] fox`, `[4] jumps`, `[5] over`.

Now let’s build the matrix:

|   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
| - | - | - | - | - | - | - | - | - | - |
| 0 | ✔ | ✔ | ✔ |   |   |   |   |   |   |
| 1 | ✔ | ✔ | ✔ | ✔ |   |   |   |   |   |
| 2 | ✔ | ✔ | ✔ | ✔ | ✔ |   |   |   |   |
| 3 |   | ✔ | ✔ | ✔ | ✔ | ✔ |   |   |   |
| 4 |   |   | ✔ | ✔ | ✔ | ✔ | ✔ |   |   |
| 5 |   |   |   | ✔ | ✔ | ✔ | ✔ | ✔ |   |
| 6 |   |   |   |   | ✔ | ✔ | ✔ | ✔ | ✔ |
| 7 |   |   |   |   |   | ✔ | ✔ | ✔ | ✔ |
| 8 |   |   |   |   |   |   | ✔ | ✔ | ✔ |

✅ Much sparser, faster for long texts!

---

### 🚀 Add Global Attention (Optional)

Let’s say the word `"The"` at position `0` is a **global token**.

* Every token can look at token 0.
* Token 0 can look at all tokens.

Update:

|   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |          |
| - | - | - | - | - | - | - | - | - | - | -------- |
| 0 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ← Global |
| 1 | ✔ | ✔ | ✔ | ✔ |   |   |   |   |   |          |
| 2 | ✔ | ✔ | ✔ | ✔ | ✔ |   |   |   |   |          |
| 3 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |   |   |   |          |
| 4 | ✔ |   | ✔ | ✔ | ✔ | ✔ | ✔ |   |   |          |
| 5 | ✔ |   |   | ✔ | ✔ | ✔ | ✔ | ✔ |   |          |
| 6 | ✔ |   |   |   | ✔ | ✔ | ✔ | ✔ | ✔ |          |
| 7 | ✔ |   |   |   |   | ✔ | ✔ | ✔ | ✔ |          |
| 8 | ✔ |   |   |   |   |   | ✔ | ✔ | ✔ |          |

Global token adds **long-distance awareness**!

---

## Techniques for Sparse Attention Implementation

### 🧩 1. **Unit of Sparsification** (Where to apply attention)

This is about the **pattern** or **shape** of attention — who can "talk" to who.

#### A. Local Window

📦 **Idea**: Each token can only see its neighbors.

👶 **Example**: Sentence = `A B C D E`

If we use a window of size 3, then:

```
C → [B C D]  (C can see B, C, D)
E → [D E]    (E can only see D and itself)
```

Like how you talk to your friends sitting next to you in class.

---

#### B. Global Tokens

📦 **Idea**: Some special tokens (like \[CLS]) can see all others, and all others can see them.

👶 **Example**:

```
Tokens: [CLS] A B C D
[CLS] → [A B C D]
A → [CLS A]
```

Useful when you need a summary token (like a team leader who listens to everyone).

---

#### C. Diagonals / Strided

📦 **Idea**: Each token looks at past tokens with a fixed gap.

👶 **Example**:

```
T0 T1 T2 T3 T4 T5

T5 → [T3, T4, T5]   (looks 2 steps back)
```

Like checking every 2nd page in a book.

---

#### D. Block Attention

📦 **Idea**: Group tokens into chunks (like 4 words at a time) and attend between groups.

👶 **Example**:

Tokens = \[A B C D] \[E F G H] (two blocks)

```
Block 1 → Block 1 and Block 2
```

Instead of person-to-person, it's **group-to-group** talk.

---

### 🧩 2. **Importance Estimation** (Which tokens are important?)

This decides **which tokens to keep** based on rules.

---

#### A. Fixed Importance

📦 **Idea**: Always use the same rule.

👶 **Example**: Always keep the first 2 tokens.

```
Keep: T0, T1 (no matter the sentence)
```

Simple but not smart — like always picking the first 2 students in line.

---

#### B. Dynamic Importance

📦 **Idea**: Use token properties like attention score or embedding size to decide.

👶 **Example**:

Tokens = T0, T1, T2, T3
Embedding norms: \[1.2, 2.0, 0.5, 2.5]

Keep tokens with top 2 norms → T1 and T3

```
Keep: T1 (2.0), T3 (2.5)
```

Like picking top scorers in a test dynamically.

---

### 🧩 3. **Budget Allocation** (How many tokens to keep)

This is about **how many** tokens each part of the model gets to use.

---

#### A. Uniform Budget

📦 **Idea**: Each head or layer gets same number of important tokens.

👶 **Example**: Every attention head gets to keep 4 tokens.

```
Head 1 → 4 tokens  
Head 2 → 4 tokens
```

Fair, but not always efficient.

---

#### B. Adaptive Budget

📦 **Idea**: More important layers or heads get more budget.

👶 **Example**:

```
Layer 1 → 8 tokens  
Layer 6 → 3 tokens
```

Like giving senior employees more resources.

---

### 4. **KV Cache Management** (Which keys/values to keep during decoding)

This is for **text generation** where you can’t keep all history due to memory limits.

---

#### A. Sliding Window

📦 **Idea**: Only keep the last N tokens.

👶 **Example**: N = 3, and sentence so far = A B C D E

```
Keep: C D E (drop A, B)
```

Like only remembering the last few lines of a conversation.

---

#### B. Attention Score Based

📦 **Idea**: Keep tokens that were most useful in the past (high attention).

👶 **Example**:

Tokens: A B C D
Attention scores: \[0.1, 0.9, 0.3, 0.8]

Keep top-2: B and D

```
Drop: A and C
```

Like keeping people you talked to the most at a party.

---

#### C. Combine Strategies

📦 **Idea**: Keep recent + most useful tokens.

👶 **Example**:

Keep: last 2 tokens + any token with high score

```
Result: D E + B (if B had high attention)
```

Best of both worlds.

---

## Summary (TL;DR)

| Concept          | Meaning                                                                |
| ---------------- | ---------------------------------------------------------------------- |
| Attention        | Helps tokens focus on others to understand meaning.                    |
| Full Attention   | Looks at **all** tokens (slow for long sequences).                     |
| Sparse Attention | Looks at a **few** nearby or important tokens (faster).                |

---
