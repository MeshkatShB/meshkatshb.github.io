---
title: "Understanding Prompt Caching: Speeding Up LLMs and Reducing Costs"
date: 2026-02-08
permalink: /posts/2026/02/understanding-prompt-caching/
tags:
  - LLM
  - Prompt Caching
  - Optimization
  - AI
---

Prompt caching can significantly improve the **speed and cost-effectiveness of large language models (LLMs)**; but what exactly _is_ prompt caching?

In this post, I’ll break down:

- What prompt caching _is_ and _what it’s not_
- How it works under the hood
- What types of prompt content can be cached
- How cache matching works
- When it’s most beneficial
- Practical constraints you need to know

By the end, you’ll understand how prompt caching can push LLM performance and efficiency to the next level.

---

## What Prompt Caching _Is Not_

Before we define prompt caching, it helps to clarify what it _isn’t_.

One common misunderstanding is thinking prompt caching is the same as output caching.

For example:

1. A database query returns results that are cached
2. A future request with the _same query_ can retrieve that cached result
3. The system avoids re-running the query

That’s output caching; storing the **result** of a call to reuse later.

While output caching _can_ be applied to LLM responses, it’s not what prompt caching refers to.

---

## What Prompt Caching _Is_

Prompt caching focuses _only_ on the **input prompt**, or more precisely, part of that input, and caches how the model _interprets_ it.

Here’s the key idea:  
When you send a prompt into an LLM, the model doesn’t begin generating output immediately. Instead, it performs an expensive internal computation called **key/value (KV) pair generation**.

- The model computes KV pairs for every token
- These pairs represent the model’s _internal understanding_ of the text
- This phase often takes more compute than generating even the first output token

Prompt caching saves **these computed KV pairs** so that the model doesn’t have to recompute them again for similar input.

---

## How Prompt Caching Works Under the Hood

When an LLM receives a prompt:

1. It analyzes each token in every transformer layer
2. It computes internal KV pairs for those tokens
3. This analysis happens _before_ output generation

With prompt caching:

- KV pairs from a previous prompt are stored
- When a new prompt partially matches a cached one, the model skips recomputing matching segments
- The model only processes the _new portion_ of the prompt

This means developers can structure prompts so that large static content is cached once — and reused across multiple queries.

---

## What Can Be Cached?

Prompt caching typically applies to static or semi-static parts of prompts:

### Common Examples

✅ **System prompts**: instructions that define agent behavior  
✅ **Large documents**: manuals, research papers, contracts  
✅ **Few-shot examples**: demonstration examples for output formatting  
✅ **Static context blocks**: any repeated context that remains unchanged

In contrast, dynamic portions (like a user’s question) usually come _after_ these static elements and are not cached.

---

## How the Cache Decides What to Reuse

Prompt caching systems use **prefix matching**.

- The system matches prompts _from the beginning_, token by token
- As long as new input matches the cached prompt, those KV pairs are reused
- Once there’s a difference, caching stops and standard processing resumes

This means **prompt structure matters**; static parts should come first.
