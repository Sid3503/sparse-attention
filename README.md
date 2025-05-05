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

## Summary (TL;DR)

| Concept          | Meaning                                                                |
| ---------------- | ---------------------------------------------------------------------- |
| Attention        | Helps tokens focus on others to understand meaning.                    |
| Full Attention   | Looks at **all** tokens (slow for long sequences).                     |
| Sparse Attention | Looks at a **few** nearby or important tokens (faster).                |
| Your Code        | Uses a mix of **local** + **global** attention for better performance. |

---
