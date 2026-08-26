---
title: "Mixture of Experts: How Trillion-Parameter AI Models Run Efficiently"
date: 2026-08-26
permalink: /posts/2026/08/mixture-of-experts-moe/
tags:

- Artificial Intelligence
- Large Language Models
- Mixture of Experts
- MoE
- DeepSeek
- Kimi
- Transformers
- Machine Learning
---

If you look at the architecture of many recent open-weight language models, including **DeepSeek, Kimi, GLM, and Qwen**, you'll notice that many of them share an important architectural idea:

**Mixture of Experts (MoE).**

The idea sounds almost contradictory at first. Some MoE models contain hundreds of billions—or even trillions—of parameters, yet only a relatively small fraction of those parameters are used to process any individual token.

That's the fundamental idea behind Mixture of Experts:

> **Increase total model capacity without increasing active computation proportionally.**

Instead of running every token through one enormous neural network, an MoE model contains many smaller specialized networks called **experts**. A router dynamically decides which experts should process each token.

Let's break down how this works, why it makes modern LLMs dramatically more scalable, and what problems appear when you actually try to train and serve these models.

---

# First: How a Normal Transformer Works

Before understanding Mixture of Experts, we need to look at a normal Transformer block.

At a high level, it contains two major components:

1. **Self-Attention**
2. **Feed-Forward Network (FFN)**

Self-attention handles communication between tokens.

Consider:

> "The animal was tired, so it stopped."

When processing the token `it`, attention can look at `animal` to understand what `it` refers to.

The Feed-Forward Network works differently.

After attention has created a contextual representation for each token, the FFN processes each token independently.

A simplified FFN looks like this:

```text
Token Representation
        │
        ▼
  Up Projection
        │
        ▼
    Activation
        │
        ▼
 Down Projection
        │
        ▼
Updated Representation
```

The up-projection expands the token representation into a much larger intermediate dimension.

If the Transformer hidden dimension is:

```text
d_model
```

the FFN dimension might traditionally be around:

```text
d_ff ≈ 4 × d_model
```

That large intermediate layer gives the network enormous capacity to learn useful features.

---

# Why the Feed-Forward Network Is So Important

You can think of neurons inside the FFN as responding to different learned patterns.

For example, some neurons might strongly activate for:

* programming concepts
* emotional language
* mathematical patterns
* grammatical structures
* factual associations

If the token `tired` enters the network, neurons associated with fatigue or emotional state might activate strongly, while neurons associated with programming syntax might barely activate.

This creates an interesting observation.

**Different tokens activate very different parts of the network.**

A programming token doesn't necessarily need the same features as a medical token.

Yet in a dense Transformer, every token still passes through the entire FFN.

That becomes extremely expensive as models grow.

---

# The Scaling Problem

Suppose we make the FFN ten times larger.

The model gains much more capacity.

Unfortunately, every token must now interact with roughly ten times as many parameters.

So:

```text
10× parameters ≈ 10× FFN computation
```

At some point, scaling becomes prohibitively expensive.

But there's an interesting opportunity.

If different tokens naturally require different features, why should every token activate the entire network?

Imagine a university.

If you have a question about quantum mechanics, you go to the **physics department**.

If you have a question about medieval Europe, you go to the **history department**.

You don't ask every department in the university to answer every question.

Mixture of Experts applies essentially the same idea to neural networks.

---

# Enter Mixture of Experts

Instead of one enormous Feed-Forward Network:

```text
              Giant FFN
Token ─────────────────────────► Output
```

we divide it into multiple smaller networks:

```text
                 ┌─ Expert 1
                 ├─ Expert 2
Token ─► Router ─┼─ Expert 3
                 ├─ Expert 4
                 └─ Expert N
```

These networks are called **experts**.

For every token, only a subset of the experts is activated.

Importantly, the experts aren't usually manually assigned labels such as:

```text
Expert 1 → Programming
Expert 2 → Mathematics
Expert 3 → Medicine
```

Specialization emerges automatically during training.

The model learns which experts are useful for which representations.

---

# The Router: Who Chooses the Experts?

The component responsible for assigning tokens to experts is called the **router**.

The router receives the token's hidden representation:

```text
x
```

and multiplies it by a learned routing matrix:

```text
logits = xW_router
```

This produces one score for every expert.

For example:

```text
Expert 1 → 0.03
Expert 2 → 0.80
Expert 3 → 0.12
Expert 4 → 0.05
```

The model clearly believes Expert 2 is the most relevant.

A normalization function such as softmax can transform those values into routing probabilities.

The selected experts process the token independently, and their outputs are weighted according to the router's scores.

Conceptually:

```text
Output =
    w₁ Expert₁(x)
  + w₂ Expert₂(x)
  + ...
  + wₙ Expertₙ(x)
```

But there's still a problem.

If every expert processes every token, we haven't saved any computation.

That's where **sparse routing** comes in.

---

# Top-1 Routing

One of the simplest approaches is **Top-1 routing**.

The router selects only the highest-scoring expert.

```text
Expert 1 → 0.03
Expert 2 → 0.80  ← SELECTED
Expert 3 → 0.12
Expert 4 → 0.05
```

Only Expert 2 executes.

This approach is extremely computationally efficient.

However, there's a limitation.

Real-world concepts aren't always cleanly separable.

A token could simultaneously involve:

```text
Programming + Mathematics
```

or:

```text
Medicine + Biology + Statistics
```

Forcing that representation through a single expert can limit the model's expressive capacity.

---

# Top-K Routing

Modern MoE architectures therefore commonly use **Top-K routing**.

Instead of selecting one expert, the router selects several.

For example:

```text
Expert 1 → 0.10
Expert 2 → 0.45  ✓
Expert 3 → 0.35  ✓
Expert 4 → 0.10
```

With:

```text
K = 2
```

Experts 2 and 3 process the token.

Their outputs are then combined according to their routing weights.

This costs slightly more than Top-1 routing but remains dramatically cheaper than activating every expert.

And this gives us the defining property of sparse MoE models:

```text
Total Parameters >> Active Parameters
```

A model can therefore have enormous overall capacity while activating only a fraction of it for each token.

---

# The First Big Problem: Expert Collapse

Unfortunately, training the router isn't straightforward.

At initialization, routing decisions are mostly random.

Suppose Expert 1 randomly receives slightly more tokens than the others.

Because Expert 1 receives more tokens, it also receives more gradient updates.

Therefore:

```text
More tokens
   ↓
More gradient updates
   ↓
Better expert
   ↓
Router prefers that expert
   ↓
Even more tokens
```

This creates a positive feedback loop.

Eventually:

```text
Expert 1 → overloaded
Expert 2 → mostly idle
Expert 3 → mostly idle
Expert 4 → mostly idle
```

This phenomenon is known as **expert collapse**.

At that point, we've built an expensive multi-expert architecture but effectively use only a tiny part of it.

---

# Auxiliary Load-Balancing Loss

One traditional solution is to add another objective to training.

Normally an LLM primarily minimizes something like cross-entropy:

```text
L_language
```

For MoE, we introduce another term:

```text
L_balance
```

which encourages tokens to be distributed more evenly across experts.

The total objective becomes conceptually:

```text
L_total = L_language + αL_balance
```

where `α` determines how strongly we care about balancing.

If one expert receives almost all the traffic, the balancing loss becomes large.

If traffic is distributed more evenly, the penalty becomes smaller.

This can prevent expert collapse.

But it introduces another problem.

---

# Prediction Quality vs. Load Balancing

The language-modeling objective says:

> Route tokens wherever prediction becomes best.

The balancing objective says:

> Make sure every expert receives a reasonable amount of traffic.

Those goals don't always agree.

Their gradients can push the router in different directions.

That means aggressive balancing can potentially hurt routing quality.

This leads to an interesting development used by models such as **DeepSeek-V3**.

---

# Auxiliary-Loss-Free Load Balancing

Instead of modifying the training objective, DeepSeek's approach applies a dynamic bias directly to expert routing scores.

Suppose an expert receives too much traffic.

Its routing bias is slightly reduced:

```text
Overloaded expert
        ↓
Decrease routing bias
        ↓
Less likely to be selected
```

If another expert receives too little traffic:

```text
Underused expert
        ↓
Increase routing bias
        ↓
More likely to be selected
```

Crucially, this bias isn't trained through ordinary backpropagation.

It is adjusted based on the observed expert load.

The model can therefore encourage balanced routing without directly introducing a competing optimization objective.

---

# Extreme Sparsity Changes the Problem

As models become larger, MoE routing becomes even more extreme.

The supplied material describes Kimi's architecture as scaling to hundreds of experts while activating only a small number for each token.

At that level of sparsity, each individual expert receives only a tiny percentage of tokens.

This makes simple fixed-step balancing slow.

If the adjustment is too small:

```text
Expert stays underused for thousands of steps
```

If it's too large:

```text
Underused → overloaded → underused → overloaded
```

Routing becomes unstable.

---

# Quantile-Based Biasing

An alternative described for Kimi is to calculate the required routing bias directly from the current score distribution.

Instead of repeatedly saying:

```text
+ small adjustment
+ small adjustment
+ small adjustment
...
```

the system estimates the appropriate threshold from the batch distribution itself.

An overloaded expert tends to have higher routing scores.

Its quantile threshold therefore becomes higher, resulting in a negative correction.

An underutilized expert receives the opposite treatment.

Conceptually:

```text
Overused expert
→ higher score quantile
→ negative bias
→ fewer tokens

Underused expert
→ lower score quantile
→ positive bias
→ more tokens
```

This is particularly useful when dealing with extremely sparse routing.

---

# MoE Is Also a Distributed Systems Problem

Once models reach hundreds of billions or trillions of parameters, all experts cannot live on one GPU.

Instead:

```text
GPU 1 → Experts 1–16
GPU 2 → Experts 17–32
GPU 3 → Experts 33–48
GPU 4 → Experts 49–64
...
```

Now routing isn't merely a neural-network decision.

It becomes a **network communication problem**.

If a token is assigned to Expert 27, its representation must be transferred to the GPU hosting Expert 27.

This means poor load balancing can overload not only experts but also:

* GPUs
* memory bandwidth
* interconnects
* communication infrastructure

Efficient MoE inference therefore depends heavily on systems engineering.

---

# Router Stability and Z-Loss

Another problem appears during training.

Router logits can grow increasingly large.

With softmax:

```text
softmax(x_i) = exp(x_i) / Σ exp(x_j)
```

large logits produce enormous exponentials.

In reduced-precision formats, that can create numerical instability.

A standard numerical trick subtracts the largest logit before applying softmax:

```text
x'_i = x_i - max(x)
```

Because softmax is translation-invariant, this doesn't change the probabilities.

However, it only fixes the immediate numerical calculation.

It doesn't necessarily stop the underlying logits from growing.

That's where **Router Z-Loss** comes in.

The idea is to penalize excessive router-logit growth so the routing network remains numerically stable during training.

---

# The GPU Problem: Expert Capacity

Even if routing works perfectly mathematically, we still need to execute the experts efficiently.

GPUs love large, regular matrix operations.

MoE routing creates irregular workloads.

Imagine:

```text
Expert 1 → 35 tokens
Expert 2 → 25 tokens
Expert 3 → 20 tokens
```

If GPU kernels require every expert batch to have the same shape, we might allocate:

```text
35 × Expert 1
35 × Expert 2
35 × Expert 3
```

But Experts 2 and 3 don't need that much space.

The unused rows become padding.

That wastes:

```text
GPU memory + GPU compute
```

Alternatively, we could allocate only 25 slots per expert.

Then Expert 1 overflows and some tokens must be dropped.

Neither solution is ideal.

---

# MegaBlocks: Removing the Padding

**MegaBlocks** attacks this systems problem using non-padded block-sparse matrix multiplication.

Instead of forcing every expert into the same matrix shape, expert workloads can correspond more closely to their actual token counts.

For example:

```text
Expert 1 → 35 tokens
Expert 2 → 25 tokens
Expert 3 → 20 tokens
```

The workloads are divided into smaller GPU-friendly blocks.

For a block size of 16:

```text
Expert 1 → 16 + 16 + 3
Expert 2 → 16 + 9
Expert 3 → 16 + 4
```

The GPU processes these blocks efficiently rather than allocating a huge padded matrix for every expert.

The result is much less wasted memory and computation.

---

# Expert-Choice Routing

There's another interesting way to solve load balancing.

So far we've assumed:

> Tokens choose experts.

But what if we reverse it?

> **Experts choose tokens.**

This is known as **Expert-Choice Routing**.

Instead of each token selecting its Top-K experts, every expert selects the tokens it considers most relevant.

If we have:

```text
8 tokens
4 experts
capacity = 2
```

each expert selects exactly two tokens.

Load balancing is therefore built directly into the routing mechanism.

```text
Expert 1 → 2 tokens
Expert 2 → 2 tokens
Expert 3 → 2 tokens
Expert 4 → 2 tokens
```

No expert becomes overloaded.

Interestingly, important tokens can still be selected by multiple experts while simpler tokens may receive less computation.

---

# Soft Mixture of Experts

Traditional routing uses hard decisions:

```text
Token → Expert 3
```

or:

```text
Token → Experts 3 and 7
```

**SoftMoE** removes that discrete boundary.

Instead, experts receive weighted combinations of tokens.

Conceptually:

```text
Expert 1 Input =
    0.6 × Token A
  + 0.2 × Token B
  + 0.1 × Token C
  + ...
```

The expert processes that combined representation, and its output is redistributed back according to the corresponding weights.

This makes routing differentiable and avoids hard token dropping.

---

# Heterogeneous Experts

There's also no fundamental requirement that every expert must have the same size.

In **Heterogeneous MoE**, some experts can be large and powerful while others are smaller and cheaper.

The router could conceptually learn:

```text
Simple token
    ↓
Small expert

Complex token
    ↓
Large expert
```

This introduces another dimension of conditional computation.

Instead of only deciding **which knowledge** a token needs, the model can potentially determine **how much computation** that token deserves.

---

# Shared Experts

Modern architectures also recognize that not everything should be specialized.

Some language patterns are useful for almost every token.

Rather than repeatedly learning that information independently inside many experts, architectures such as DeepSeek can include **shared experts**.

```text
                 ┌──────── Shared Expert ────────┐
Token ───────────┤                               ├─► Output
                 └─ Router → Specialized Experts┘
```

The shared expert processes every token.

Meanwhile, routed experts specialize in more specific patterns.

This creates a useful separation between:

```text
Common knowledge
        +
Specialized knowledge
```

---

# Fine-Grained Expert Segmentation

Another powerful idea is to replace a small number of large experts with many smaller experts.

Imagine we have:

```text
16 large experts
Top-2 routing
```

There are only a limited number of possible expert combinations.

Instead, we could divide the same total expert capacity into:

```text
64 smaller experts
Top-8 routing
```

while keeping active computation roughly similar.

The number of possible combinations grows enormously.

This gives the router much finer control over which mixture of capabilities should process each token.

The source describes this combination of **shared experts and fine-grained expert segmentation** as a major part of modern DeepSeek-style MoE architecture.

---

# Putting MoE Inside the Transformer

Now we can assemble everything.

A normal Transformer layer roughly looks like:

```text
Tokens
   │
   ▼
Embeddings
   │
   ▼
Self-Attention
   │
   ▼
Residual + Normalization
   │
   ▼
Dense FFN
   │
   ▼
Residual + Normalization
```

An MoE Transformer replaces the dense FFN with:

```text
Tokens
   │
   ▼
Embeddings
   │
   ▼
Self-Attention
   │
   ▼
Residual + Normalization
   │
   ▼
Router
   │
   ├──► Expert 1
   ├──► Expert 2
   ├──► Expert 3
   ├──► ...
   └──► Expert N
          │
          ▼
Weighted Combination
          │
          ▼
Residual + Normalization
```

The attention mechanism can remain dense.

The enormous FFN capacity becomes sparse.

---

# Why Total Parameters Can Be Misleading

This also explains why parameter count alone doesn't tell you the computational cost of an MoE model.

Consider a simplified example.

Suppose an FFN contains:

```text
48B parameters
```

In a dense model:

```text
Active per token = 48B
```

Now divide it into eight experts:

```text
8 × 6B = 48B
```

and use Top-2 routing.

Each token activates:

```text
2 × 6B = 12B
```

So:

```text
Total capacity = 48B
Active capacity = 12B
```

The model has access to a huge parameter space without evaluating the entire parameter space for every token.

This is the central computational advantage of Mixture of Experts.

---

# Dense Models vs. Mixture of Experts

The difference can be summarized like this:

| Dense Model                           | Mixture of Experts                           |
| ------------------------------------- | -------------------------------------------- |
| Every token uses the same parameters  | Tokens use different experts                 |
| All FFN parameters are active         | Only selected experts are active             |
| Compute scales with model size        | Compute scales mainly with active parameters |
| Simpler execution                     | Complex routing and communication            |
| Easier GPU utilization                | Requires specialized kernels                 |
| No expert balancing problem           | Must manage expert load                      |
| Straightforward distributed inference | Expert parallelism becomes important         |

MoE isn't free efficiency.

It trades straightforward computation for architectural and systems complexity.

---

# The Bigger Idea: Conditional Computation

Mixture of Experts is ultimately an example of a broader concept:

**conditional computation**.

Traditional neural networks essentially say:

```text
Every input
→ Every layer
→ Every parameter
```

MoE instead says:

```text
Input
→ Determine what's needed
→ Activate relevant computation
```

That's a much more scalable idea.

Different tokens require different capabilities.

Different problems require different amounts of computation.

And instead of forcing every token through the entire model, we dynamically allocate computation where it's useful.

---

# Final Thoughts

Mixture of Experts is one of the key architectural ideas enabling extremely large language models.

The fundamental idea is surprisingly simple:

> **Don't use the entire model for every token.**

Build many experts.

Let a router decide which ones are useful.

Activate only a small subset.

But implementing that simple idea introduces a surprisingly deep collection of problems:

* expert collapse
* load balancing
* routing stability
* token overflow
* GPU padding
* expert parallelism
* inter-GPU communication
* routing quality
* numerical stability

Modern architectures address these problems with techniques such as:

* Top-K routing
* auxiliary load-balancing losses
* auxiliary-loss-free balancing
* quantile-based routing bias
* Router Z-Loss
* MegaBlocks
* Expert-Choice Routing
* SoftMoE
* heterogeneous experts
* shared experts
* fine-grained expert segmentation

The result is one of the most important ideas in modern LLM scaling:

```text
Massive total capacity
        +
Sparse activation
        =
Much lower compute per token
```

That's how models can move toward **hundreds of billions or even trillions of parameters** without requiring every single parameter to run for every token.

And that's why Mixture of Experts has become such an important part of the architecture behind the new generation of large-scale AI models.
