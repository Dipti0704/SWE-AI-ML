
# 04_Modern_Text_Image_AI_Model_Architectures

---

# Modern Text & Image AI Model Architectures


---

# 1. Topics Covered

According to the **topic overview slide (Page 2)**  the lecture explains the following concepts:

1. Transformers
2. Encoder & Decoder
3. Decoder-only architecture
4. Data flow inside the model
5. Token & positional embedding
6. Attention mechanisms
7. Decoding strategies
8. PyTorch implementation
9. Pretraining & fine-tuning
10. Chat fine-tuning
11. Vision Transformers
12. Multimodality

These topics together explain **how modern AI models like GPT, Llama, Claude, and Gemini actually work internally.**

---

# 2. The Journey to Modern LLMs

The slide titled **"The Journey to Modern LLMs" (Page 3)**  introduces the evolution of language models.

Modern LLMs originate from the **Transformer architecture introduced in 2017**.

Paper:

```
Attention Is All You Need
```

Published by:

```
Google Brain
```

---

## What Problem Were Transformers Solving?

Before transformers, NLP models were:

* RNNs
* LSTMs
* GRUs

Problems with them:

1. Hard to parallelize
2. Forget long-range context
3. Training was slow

Transformers solved this using:

```
Self-Attention
```

---

# 3. Encoder and Decoder

Transformers contain two main parts.

```
Encoder
Decoder
```

---

# Encoder

Encoder's job:

```
Understand the input
```

Example:

```
Sentence → Meaning representation
```

Example model:

```
BERT
```

---

### How Encoder Works

Encoder uses **Bidirectional Attention**.

This means:

A word can look at:

```
words before it
words after it
```

Example sentence:

```
The bank near the river flooded.
```

The word **bank** can look at:

```
river
flooded
```

So the model understands **bank means river bank, not financial bank.**

---

# Decoder

Decoder’s job:

```
Generate text
```

Example model:

```
GPT
```

Decoder works using:

```
Causal Attention
```

Meaning:

A token can only look at **previous tokens**, not future tokens.

Example:

```
I love machine learning
```

While generating **learning**, the model can only see:

```
I love machine
```

It cannot see the future.

---

# Seq2Seq Models

Encoder + Decoder together form **Sequence-to-Sequence models**.

Example task:

```
English → French translation
```

Input:

```
I love AI
```

Output:

```
J'aime l'IA
```

Encoder reads English.

Decoder generates French.

Example models:

```
T5
BART
```

---

# 4. Three Architecture Paths

The slide **Three Architecture Paths (Page 4)**  shows that modern transformer models evolved into three types.

---

# 1. Encoder-only Models

Example:

```
BERT
```

Purpose:

```
Language understanding
```

Used for:

* classification
* sentiment analysis
* search ranking
* question answering

Training method:

```
Masked Language Modeling
```

Example:

```
The cat sat on the [MASK]
```

Model predicts:

```
mat
```

---

# 2. Encoder–Decoder Models

Example:

```
T5
BART
```

Purpose:

```
Text transformation
```

Examples:

* translation
* summarization
* paraphrasing

---

# 3. Decoder-only Models

Examples:

```
GPT
Llama
Claude
```

Purpose:

```
Text generation
```

Training objective:

```
Next-token prediction
```

This is the foundation of **modern LLMs**.

---

# 5. Decoder-only Breakthrough

The slide **Decoder Only Breakthrough (Page 5)**  highlights GPT-2.

GPT-2 was released in:

```
2019
```

---

### Important GPT-2 Characteristics

1️⃣ **48 decoder blocks (XL model)**

Meaning:

The network contains **48 transformer layers stacked on top of each other**.

Each layer learns deeper language patterns.

---

2️⃣ **Web-scale training corpus**

The model was trained on massive internet text.

Examples:

* books
* articles
* websites
* forums

This allows the model to learn:

```
knowledge
facts
grammar
reasoning patterns
```

---

3️⃣ **Zero-shot task transfer**

Meaning:

The model can perform tasks **without task-specific training**.

Example:

```
Translate English to French:
Hello → Bonjour
```

Even though the model was never trained specifically for translation.

---

# 6. Why Decoder-Only Models Won

The slide **Why Decoder Only Won (Page 6)**  explains four reasons.

---

# 1. Training Alignment

Decoder models learn:

```
P(x_t | x_1 ... x_t-1)
```

Meaning:

Probability of next word given previous words.

Example:

```
I love machine → ?
```

Model predicts:

```
learning
```

This is the **same objective used during inference**.

So training and usage are aligned.

---

# 2. Simplicity

Decoder-only models are simpler.

They only use:

```
Decoder blocks
```

No encoder.

No cross attention.

This simplifies scaling.

---

# 3. Emergent Abilities

When models become large, new abilities appear.

Example:

GPT-3 has:

```
175 Billion parameters
```

Emergent abilities:

* reasoning
* math
* translation
* coding

Even though they were not explicitly programmed.

---

# 4. Unified Interface

Everything becomes:

```
Prompt → Completion
```

Example:

```
Prompt:
Explain gravity.

Completion:
Gravity is a force that attracts objects with mass.
```

One model can solve many tasks.

---

# 7. Input and Output of GPT-2

The slide **IP & OP GPT-2 (Page 7)**  shows how GPT generates text.

Example:

Input:

```
recite the first law of robotics
```

Output generated step-by-step:

```
A robot may not injure a human being...
```

The model predicts **one token at a time**.

---

# 8. Core Principle of Decoder-Only Models

The slide **Decoder Only Core Principle (Page 8)**  shows the fundamental formula:

```
P(x_t | x_1 ... x_t-1)
```

Meaning:

Probability of next token given previous tokens.

---

### Architecture

Decoder-only models:

```
Stack N transformer decoder blocks
```

Example:

```
GPT-3 → 96 layers
```

---

### Direction

Processing happens:

```
Left → Right
```

---

### Constraint

Strict rule:

```
Causal Masking
```

A token **cannot see future tokens**.

---

# 9. Data Flow Inside the Model

The slide **Data Flow Inside the Model (Page 9)**  shows the entire pipeline.

Flow:

```
Text
↓
Tokens
↓
Embeddings
↓
Positional Encoding
↓
Transformer Decoder Blocks
↓
Logits
↓
Softmax
↓
Next Token
```

---

# 10. Tokenization

The slide **Tokenization (Page 10)**  explains how text becomes numbers.

---

## What is Tokenization?

Tokenization converts:

```
Text → integers
```

Example:

```
I love AI
```

Tokens:

```
[I, love, AI]
```

Token IDs:

```
[12, 473, 89]
```

---

# Byte Pair Encoding (BPE)

Most LLMs use:

```
BPE tokenizer
```

It splits words into **subwords**.

Example:

```
unbelievable
```

Could become:

```
un + believable
```

or

```
un + believ + able
```

---

# Vocabulary Size

GPT-2:

```
50,257 tokens
```

Llama 3:

```
128K tokens
```

Larger vocabulary allows:

* more languages
* better code support
* fewer token splits

---

# 11. Embedding Lookup

The slide **Embedding Lookup (Page 11)**  explains converting tokens into vectors.

---

## Why Embeddings?

Neural networks cannot process text.

They only understand:

```
numbers
```

So tokens become **vectors**.

---

### GPT-2 Embedding Matrix

Size:

```
50257 × 768
```

Meaning:

```
50,257 tokens
768 features per token
```

Example embedding:

```
"AI" → [0.13, -0.88, 0.47, ...]
```

---

# 12. Positional Encoding

The slide **Positional Encoding (Page 12)**  explains how transformers understand order.

Self-attention alone **does not know token order**.

Example:

```
dog bites man
man bites dog
```

Tokens are same but meaning different.

So we add **positional information**.

---

# Absolute Positional Embedding (GPT-2)

GPT-2 stores position embeddings.

Matrix:

```
1024 × 768
```

Meaning:

```
max context = 1024 tokens
```

Each position has a vector.

---

# RoPE Positional Encoding (Llama)

Modern models use:

```
RoPE (Rotary Position Embedding)
```

Instead of adding position vectors, it **rotates vectors in embedding space**.

Benefits:

* relative position awareness
* longer context windows
* better scaling

---

# 13. RoPE Mechanism

The slide **RoPE Mechanism (Page 13)**  explains this improvement.

Key idea:

```
Encode distance directly into attention calculation
```

Advantages:

* captures relative token distance
* supports longer context
* removes absolute embedding matrix

Example:

Tokens farther away receive **naturally lower attention**.

---

# 14. Transformer Block Structure

The slide **Transformer Block Structure (Page 14)**  shows the internal architecture.

Each block contains:

```
LayerNorm
Self Attention
Residual
LayerNorm
Feed Forward
Residual
```

---

### Layer Normalization

Purpose:

```
stabilize values
```

Keeps activations balanced.

---

### Residual Connections

Formula:

```
x = x + Attention(LN(x))
```

Purpose:

* prevent information loss
* improve gradient flow

---

# 15. Self Attention

The slide **Self Attention (Page 15)**  explains the attention concept.

Each token asks:

```
Which other tokens matter to me?
```

Three vectors are created.

```
Query (Q)
Key (K)
Value (V)
```

---

### Query

Represents:

```
What am I looking for?
```

---

### Key

Represents:

```
What information do I contain?
```

---

### Value

Represents:

```
What information will I give if selected?
```

---

# 16. Self Attention Formula

The slide **Self Attention Formula (Page 16)**  shows:

```
Attention(Q,K,V) = softmax(QK^T / √d_k) V
```

Steps:

1. Compute similarity between tokens
2. Scale by √dimension
3. Apply softmax
4. Multiply with value vectors

---

# 17. GPT-2 Attention Specs

The slide **GPT-2 Attention Specs (Page 17)** .

Model dimension:

```
768
```

Number of heads:

```
12
```

Head dimension:

```
768 / 12 = 64
```

Each head learns **different relationships**.

Examples:

* grammar
* semantics
* references

---

# 18. Causal Self Attention

The slide **Causal Self Attention (Page 18)**  explains masking.

Rule:

```
If j > i → -∞
```

Meaning:

Future tokens cannot be seen.

Softmax of -∞ becomes:

```
0
```

So attention weight = 0.

---

# 19. Multi Head Attention

The slide **Multi Head Attention (Page 19)** .

Instead of one attention computation, we compute **many in parallel**.

Steps:

```
Split embeddings
Compute attention per head
Concatenate outputs
Apply projection
```

---

### Why Multiple Heads?

Each head learns different patterns:

Examples:

* grammar
* semantic similarity
* long-range dependencies

---

# 20. Head Dimensions

The slide **Head Dimensions (Page 20)** .

Input tensor shape:

```
[Batch, Sequence, 768]
```

Split into heads:

```
[Batch, Heads, Sequence, 64]
```

Then merged back to:

```
768
```

---

# 21. Grouped Query Attention (GQA)

The slide **Grouped Query Attention (Page 21)**  explains a modern optimization.

Used in:

```
Llama 2
Llama 3
```

Instead of separate K and V for every head:

```
multiple query heads share K and V
```

Example:

```
32 Q heads
8 KV heads
```

Benefits:

* 75% memory reduction
* faster inference

---

# 22. Feed Forward Network

The slide **Feed Forward Network (Page 22)** .

This layer processes tokens individually.

Steps:

### Expand

```
768 → 3072
```

---

### Activation

Apply nonlinear function.

Common functions:

```
GELU
SwiGLU
```

---

### Project Back

```
3072 → 768
```

---

### Purpose

Acts like:

```
knowledge memory
```

---

# 23. Output Projection

The slide **Output Projection (Page 23)** .

Steps:

1. Take hidden state of last token
2. Map to vocabulary size
3. Apply softmax

Example:

```
Vocabulary = 50,000
```

Output:

```
50,000 probabilities
```

---

# 24. Decoding Process

The slide **Decoding Process (Page 24)** .

Pipeline:

```
Logits → Decoding Strategy → Text
```

Logits are raw scores.

Decoding converts them into words.

---

# 25. Greedy Search

The slide **Greedy Search (Page 25)**  explains the simplest decoding method.

Algorithm:

```
Pick token with highest probability
```

Example:

```
I have a dream
```

Possible next tokens:

```
of 15.7%
being 9.2%
a 30.5%
```

Greedy chooses:

```
a
```

---

# 26. Limitation of Greedy Search

The slide **Limitation of Greedy Search (Page 26)** .

Problem:

Greedy chooses **local best choice**.

But that may not produce **best sentence overall**.

Example:

```
I like eating pizza with...
```

Greedy might choose:

```
ketchup
```

But better continuation might be:

```
extra cheese
```

But greedy cannot reconsider earlier decisions.

---

# End Summary

Modern LLM pipeline:

```
Text
→ Tokenization
→ Embedding
→ Positional Encoding
→ Transformer Blocks
→ Attention
→ FFN
→ Logits
→ Decoding
→ Generated Text
```

---
---
---
---

# Deep Understanding of Transformers & GPT

*(Advanced Concept Notes for AI Engineering)*

---

# 1. Intuition of Attention (Very Important)

Attention is the **core mechanism behind transformers**.

Instead of processing words sequentially like RNNs, transformers allow **every word to look at other words simultaneously**.

This allows the model to understand **context**.

---

# Example of Attention

Sentence:

```
I deposited my money in the bank.
```

Word: `bank`

Possible meanings:

1. Financial bank
2. River bank

The model must determine which meaning is correct.

Attention allows the word **bank** to look at:

```
deposited
money
```

These words indicate **financial bank**.

---

# Another Example

Sentence:

```
He sat on the river bank.
```

Here **bank** attends to:

```
river
sat
```

Now the meaning becomes **river bank**.

---

# Key Idea of Attention

Each word asks:

```
Which other words are important for me?
```

This importance is represented as **attention weights**.

Example attention weights:

| Token | Attention weight |
| ----- | ---------------- |
| river | 0.52             |
| sat   | 0.30             |
| he    | 0.10             |
| on    | 0.08             |

The higher the weight, the more influence that token has.

---

# 2. Query, Key, Value (Core of Attention)

Every token generates three vectors:

```
Query (Q)
Key (K)
Value (V)
```

These are learned representations.

---

# Query

Query represents:

```
What am I looking for?
```

Example:

The token **bank** might search for **financial context**.

---

# Key

Key represents:

```
What information do I contain?
```

Each token advertises what type of information it has.

Example:

```
money → financial context
river → nature context
```

---

# Value

Value represents:

```
The information the token provides if selected
```

This is the actual content passed to the next layer.

---

# How Attention Works

Steps:

1. Compute similarity between Q and K
2. Convert similarity to probabilities
3. Weight the value vectors

Mathematical formula:

```
Attention(Q,K,V) = softmax(QKᵀ / √d_k) V
```

---

# Why Divide by √d_k ?

When vectors have large dimensions:

```
dot products become very large
```

This causes **softmax saturation**.

Scaling stabilizes training.

---

# 3. Visual Transformer Architecture

A transformer model is composed of many **stacked transformer blocks**.

Structure:

```
Input tokens
↓
Embedding Layer
↓
Positional Encoding
↓
Transformer Block 1
↓
Transformer Block 2
↓
Transformer Block 3
↓
...
↓
Transformer Block N
↓
Output Layer
↓
Predicted Token
```

Example:

```
GPT-3 → 96 transformer layers
GPT-4 → even more
```

Each layer refines the understanding of the sentence.

---

# 4. Inside a Transformer Block

Each transformer block has **two major components**:

```
1. Self Attention
2. Feed Forward Network
```

Full structure:

```
LayerNorm
↓
Self Attention
↓
Residual Add
↓
LayerNorm
↓
Feed Forward Network
↓
Residual Add
```

---

# Why Residual Connections?

Residual connection formula:

```
x = x + f(x)
```

Purpose:

```
Preserve original information
Improve gradient flow
Prevent vanishing gradients
```

Without residuals, deep models cannot train effectively.

---

# 5. Full Data Flow of GPT

Now we trace **one sentence through the model**.

Example sentence:

```
I love machine learning
```

---

# Step 1: Tokenization

Text is converted into tokens.

```
I love machine learning
```

Tokens:

```
[I, love, machine, learning]
```

Token IDs:

```
[12, 473, 992, 884]
```

---

# Step 2: Embedding Layer

Each token ID maps to a vector.

Example embedding dimension:

```
768
```

Example vector:

```
love → [0.23, -0.14, 0.88, ...]
```

Embedding matrix size (GPT-2):

```
50257 × 768
```

---

# Step 3: Positional Encoding

Attention does not know token order.

So we add positional information.

Example:

| Token    | Position |
| -------- | -------- |
| I        | 0        |
| love     | 1        |
| machine  | 2        |
| learning | 3        |

Position vector is added to token embedding.

```
hidden_state = token_embedding + position_embedding
```

---

# Step 4: Transformer Layers

The hidden states pass through many transformer blocks.

Each block performs:

```
Self Attention
+
Feed Forward Network
```

This gradually builds **semantic understanding**.

---

# Step 5: Logits Layer

Final hidden state is projected into vocabulary space.

Example:

```
hidden_dim → vocab_size
```

Example:

```
768 → 50257
```

The model outputs **logits**.

Logits represent raw scores for each possible word.

Example:

| Token    | Score |
| -------- | ----- |
| learning | 9.8   |
| science  | 8.1   |
| model    | 5.4   |

---

# Step 6: Softmax

Softmax converts logits into probabilities.

Example:

| Token    | Probability |
| -------- | ----------- |
| learning | 0.64        |
| science  | 0.21        |
| model    | 0.05        |

---

# Step 7: Decoding

The model selects the next token.

Example strategies:

```
Greedy search
Beam search
Top-k sampling
Top-p sampling
Temperature sampling
```

---

# Greedy Search Example

Sentence:

```
I love machine
```

Possible next tokens:

| Token        | Probability |
| ------------ | ----------- |
| learning     | 0.62        |
| intelligence | 0.21        |
| vision       | 0.05        |

Greedy selects:

```
learning
```

---

# 6. Multi-Head Attention

Instead of one attention mechanism, transformers use **multiple attention heads**.

Example GPT-2:

```
12 attention heads
```

Each head focuses on different relationships.

Examples:

Head 1:

```
grammar relationships
```

Head 2:

```
subject-verb agreement
```

Head 3:

```
long-distance context
```

---

# Attention Dimensions Example

GPT-2:

```
embedding size = 768
heads = 12
```

Per head dimension:

```
768 / 12 = 64
```

So each head processes:

```
64-dimensional vectors
```

---

# 7. Feed Forward Network (FFN)

After attention, each token passes through a neural network.

Structure:

```
Linear
↓
Activation
↓
Linear
```

Example dimensions:

```
768 → 3072 → 768
```

Activation functions used:

```
GELU
ReLU
SwiGLU
```

---

# Why FFN Exists

Attention mixes **information between tokens**.

FFN processes **information inside each token**.

Think of it as:

```
attention = communication
FFN = reasoning
```

---

# 8. Mini GPT Implementation (PyTorch)

Here is a simplified attention example.

```python
import torch
import torch.nn.functional as F

# Random hidden states
x = torch.randn(3, 4)

# Projection matrices
Wq = torch.randn(4, 4)
Wk = torch.randn(4, 4)
Wv = torch.randn(4, 4)

# Compute Q, K, V
Q = x @ Wq
K = x @ Wk
V = x @ Wv

# Attention scores
scores = Q @ K.T

# Scale
scores = scores / torch.sqrt(torch.tensor(4.0))

# Softmax
weights = F.softmax(scores, dim=-1)

# Weighted sum
output = weights @ V

print(output)
```

---

# Code Explanation

```
x
```

Hidden state matrix.

```
Wq, Wk, Wv
```

Weight matrices to create Q, K, V.

```
Q = x @ Wq
```

Query vector generation.

```
scores = Q @ K.T
```

Similarity between tokens.

```
softmax
```

Convert scores into probabilities.

```
weights @ V
```

Apply attention to values.

---

# Final Big Picture

Modern LLM pipeline:

```
Text
↓
Tokenization
↓
Embedding
↓
Positional Encoding
↓
Transformer Blocks
   ↓
   Self Attention
   ↓
   Feed Forward
↓
Logits
↓
Softmax
↓
Decoding
↓
Generated Text
```

---

# What You Should Remember for Interviews

The most important ideas:

```
1. Attention replaces recurrence
2. QKV mechanism computes relationships
3. Transformers stack many layers
4. Decoder-only models predict next tokens
5. Multi-head attention captures multiple patterns
6. FFN adds non-linear reasoning
```

---

---
---
---


# Mathematical Derivation of Self-Attention
*(Advanced Notes — AI Engineering)*

---

# 1. Why Self-Attention Exists

Traditional sequence models (RNN, LSTM) process text **sequentially**.

Example sentence:

"I love machine learning"

RNN flow:

word₁ → word₂ → word₃ → word₄

Problems:

1. Hard to parallelize
2. Long-range dependencies are difficult
3. Training becomes slow

Transformers solve this using:

```

Self-Attention

```

Self-attention allows **every token to interact with every other token simultaneously**.

---

# 2. Representing Words as Vectors

Before attention works, words must be converted into vectors.

Example sentence:

```

I love AI

```

Tokenized:

```

[I, love, AI]

```

Embedding dimension example:

```

d_model = 4

```

Token embeddings:

| Token | Vector |
|------|------|
| I | [0.2, 0.1, 0.3, 0.4] |
| love | [0.7, 0.2, 0.5, 0.9] |
| AI | [0.6, 0.8, 0.1, 0.3] |

Stack them into matrix:

```

X = [3 × 4]

```
```

X =
[0.2 0.1 0.3 0.4]
[0.7 0.2 0.5 0.9]
[0.6 0.8 0.1 0.3]

```

Shape:

```

(sequence_length × embedding_dimension)

```

---

# 3. Creating Query, Key, Value

The transformer learns three matrices:

```

W_Q
W_K
W_V

```

These are weight matrices.

Dimensions:

```

W_Q = d_model × d_k
W_K = d_model × d_k
W_V = d_model × d_v

```

Example:

```

d_model = 4
d_k = 4
d_v = 4

```

---

# Compute Q, K, V

Matrix multiplication:

```

Q = XW_Q
K = XW_K
V = XW_V

```

Shapes:

```

Q → [3 × 4]
K → [3 × 4]
V → [3 × 4]

```

---

# 4. Computing Attention Scores

Next step: measure similarity between tokens.

Formula:

```

scores = QKᵀ

```

If:

```

Q = [3 × 4]
Kᵀ = [4 × 3]

```

Then:

```

scores = [3 × 3]

```

Example matrix:

```

scores =
[1.2 0.5 0.8]
[0.4 1.6 0.7]
[0.9 0.3 1.4]

```

Interpretation:

Each row represents **how much one token attends to others**.

---

# 5. Scaling the Scores

Large vector dimensions cause dot products to grow large.

Solution:

```

scores / √d_k

```

Full formula:

```

scaled_scores = QKᵀ / √d_k

```

Example:

```

d_k = 4
√4 = 2

```

So:

```

scaled_scores = scores / 2

```

---

# Why Scaling is Necessary

Without scaling:

• Softmax saturates  
• Gradients become unstable  
• Training becomes difficult  

Scaling keeps numbers in a **reasonable range**.

---

# 6. Applying Softmax

Softmax converts scores into probabilities.

Formula:

```

softmax(x_i) = e^x_i / Σ e^x_j

```

Example:

Before softmax:

```

[2.1, 1.4, 0.9]

```

After softmax:

```

[0.52, 0.32, 0.16]

```

These are **attention weights**.

Each row sums to:

```

1

```

---

# 7. Weighted Sum of Values

Now we apply weights to value vectors.

Formula:

```

Attention = softmax(QKᵀ / √d_k) V

```

Matrix multiplication:

```

[3 × 3] × [3 × 4] → [3 × 4]

```

Result:

New contextual embeddings.

Example:

```

output =
[0.51 0.23 0.44 0.67]
[0.62 0.41 0.32 0.55]
[0.48 0.29 0.37 0.71]

```

Each token now contains **information from other tokens**.

---

# 8. Full Attention Formula

Final equation:

```

Attention(Q,K,V) = softmax(QKᵀ / √d_k) V

```

Interpretation:

| Component | Meaning |
|------|------|
Q | What the token is looking for |
K | What each token contains |
V | Information carried by tokens |

---

# 9. Multi-Head Attention

Instead of computing attention once, we compute it **multiple times**.

Example:

```

h = 8 heads

```

Each head learns **different relationships**.

Examples:

Head 1 → grammar  
Head 2 → semantic meaning  
Head 3 → subject-verb relations  

---

# Multi-Head Formula

```

head_i = Attention(QW_Qi, KW_Ki, VW_Vi)

```

Concatenate:

```

MultiHead(Q,K,V) =
Concat(head₁,...,headₕ) W_O

```

Where:

```

W_O = output projection matrix

```

---

# 10. Causal Masking

Decoder-only models use **causal attention**.

Rule:

```

Token i cannot attend to token j if j > i

```

Matrix example:

```

[0   -∞  -∞]
[0    0  -∞]
[0    0   0]

```

After softmax:

```

future tokens receive probability 0

```

This ensures **next-token prediction works correctly**.

---

# 11. Complexity of Self-Attention

Self-attention compares **every token with every token**.

If sequence length is:

```

n

```

Complexity:

```

O(n²)

```

Example:

| Tokens | Attention comparisons |
|------|------|
100 | 10,000 |
1000 | 1,000,000 |

This is why long contexts are expensive.

---

# 12. Final Intuition

Self-attention can be understood as:

```

Every word asking:
Which other words help me understand my meaning?

```

Example:

Sentence:

```

The animal didn't cross the street because it was tired.

```

Word:

```

it

```

Attention helps the model link **it → animal**, not **street**.

---

# 13. Complete Transformer Flow

```

Text
↓
Tokenization
↓
Embedding
↓
Positional Encoding
↓
Self-Attention
↓
Feed Forward Network
↓
Stacked Transformer Layers
↓
Logits
↓
Softmax
↓
Next Token

```

---

# Key Takeaways

1. Self-attention replaces recurrence.
2. Q, K, V determine token relationships.
3. Softmax converts similarity into probabilities.
4. Multi-head attention captures multiple patterns.
5. Causal masking enables next-token generation.

---

# Most Important Formula

```

Attention(Q,K,V) = softmax(QKᵀ / √d_k) V

```

Understanding this formula means you understand **how modern LLMs think about context**.
```

---

