---
title: "Remote Embedding: Load Your Embedding Once For GPU Optimization"
date: 2026-04-23
permalink: /posts/2026/04/remote-embedding/
tags:
  - LLM
  - Embeddings
  - GPU Optimization
  - Remote Embedding
---

Remote Embedding: run your embeddings on one server to load it once in a GPU for better optimization. Install it via [PyPI](https://pypi.org/project/remote-embedding/) with below command and check out the GitHub Repository at [Remote-Embedding](https://github.com/MeshkatShB/remote-embedding):

```bash
pip install remote-embedding
```

New version of remote embedding is out now:

- remote-embedding v0.3.0: Better Long-Running GPU Stability

`remote-embedding` packages two simple pieces together:

- a FastAPI embedding server
- a LangChain-compatible remote client

The point is to let multiple applications share one loaded embedding model instead of each process loading its own copy into VRAM.

Version `v0.3.0` is focused on one problem: **`long-running GPU behavior`**.

## What was happening

In real usage, embedding servers do not just need to work for one request. They need to survive hours or days of indexing, retrieval, retries, and mixed client traffic without GPU memory drifting upward.

In earlier versions, this package could gradually hold onto more GPU memory than intended in a few situations:

- concurrent requests could race while loading models
- multiple model configurations could stay resident indefinitely
- large embedding requests could push memory higher than necessary
- operators did not have enough visibility into what the server was holding

That does not always mean a true leak. Sometimes it is allocator high-water behavior. Sometimes it is multiple cached model instances. Either way, it becomes an operational problem.

## What changed in v0.3.0

### 1. Serialized model loading

Model loading is now protected by the same async lock used for GPU inference.

That matters because two requests arriving at the same time should not both decide they need to load the same model. In practice, that race can cause duplicate initialization and unnecessary VRAM growth.

### 2. Bounded model cache with eviction

The server now uses an LRU-style cache for loaded embedding models instead of keeping every unique configuration forever.

New settings:

- `MAX_LOADED_MODELS`
- `--max-loaded-models`

The default is `1`, which matches the main design goal of the project: share one loaded model across many applications.

When older models are evicted, the server attempts to release them cleanly and clears CUDA cache where appropriate.

### 3. Request-size guardrails

The server now limits how many input strings can be embedded in one request.

New settings:

- `MAX_INPUTS_PER_REQUEST`
- `--max-inputs-per-request`

The default is `128`.

This is not about being restrictive for its own sake. It is there to prevent a single oversized request from destabilizing a long-running server.

### 4. Stable default embedding batch size

The server now applies a default encoder batch size:

- `EMBEDDING_BATCH_SIZE`
- `--embedding-batch-size`

The default is `32`.

This keeps embedding behavior more predictable and reduces the chance that throughput tuning quietly turns into VRAM pressure.

### 5. Better observability

The `/health` endpoint now reports cache and request-limit state, including:

- loaded model count
- maximum loaded models
- maximum inputs per request
- embedding batch size

That makes it much easier to answer operational questions such as:

- Is the server holding more than one model?
- Are requests hitting configured limits?
- Did someone start the server with different runtime settings than expected?

## Why this release matters

The original goal of `remote-embedding` was always practical: reduce duplicated VRAM usage by centralizing embedding inference.

`v0.3.0` moves that idea closer to production reality.

It is one thing to share a model across apps. It is another thing to keep that shared service stable under real traffic. This release improves the second part.

If you are using `remote-embedding` for:

- RAG pipelines
- background indexing jobs
- multiple local tools sharing one embedding server
- constrained GPU setups

then `v0.3.0` should be a safer default than earlier versions.

## Upgrade notes

The package version is now:

```toml
version = "0.3.0"
```

Typical server usage:

```bash
remote-embedding-server \
  --model-name Qwen/Qwen3-Embedding-0.6B \
  --device cuda \
  --max-loaded-models 1 \
  --max-inputs-per-request 128 \
  --embedding-batch-size 32
```

Or through environment variables:

```bash
export EMBEDDING_MODEL_NAME=Qwen/Qwen3-Embedding-0.6B
export DEVICE=cuda
export MAX_LOADED_MODELS=1
export MAX_INPUTS_PER_REQUEST=128
export EMBEDDING_BATCH_SIZE=32
```

## Final note

GPU memory behavior is one of those issues that looks simple until a service runs long enough.

`v0.3.0` does not pretend CUDA memory is trivial. It just narrows the failure modes, adds sane defaults, and makes the server easier to reason about when it is under load.
