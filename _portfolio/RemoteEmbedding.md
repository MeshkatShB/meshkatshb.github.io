---
title: "remote-embedding"
excerpt: "A PyPI package that exposes embedding models through a shared FastAPI server and LangChain-compatible RemoteEmbeddings client."
collection: portfolio
priority: 0
---

[`remote-embedding`](https://github.com/MeshkatShB/remote-embedding) is an open-source Python package I created and maintain for RAG, semantic search, and agentic AI applications that need embeddings without repeatedly loading the same model in every process.

The package is available on [PyPI](https://pypi.org/project/remote-embedding/).

```bash
pip install remote-embedding
```

## Why I Built It

Many local RAG systems load the same Hugging Face embedding model separately in every app, worker, notebook, or indexing job. That wastes VRAM, slows startup, and makes production deployments harder to reason about.

`remote-embedding` turns embeddings into reusable local infrastructure:

- run one FastAPI embedding server
- keep one server-side model instance loaded
- call it from multiple Python applications
- use a LangChain-compatible `RemoteEmbeddings` client
- reduce duplicated GPU memory usage across RAG and semantic-search workflows

## Features

- Shared FastAPI embedding service for server-side model inference.
- LangChain-compatible client for drop-in RAG usage.
- Configurable model selection, cache paths, CPU/GPU device choice, batching, request limits, model kwargs, and encode kwargs.
- Client-side timeout handling, expected-dimension validation, per-request model overrides, HTTP connection cleanup, and context-manager support.
- CLI and environment-variable configuration for local and server deployments.

## Example

```bash
remote-embedding-server \
  --model-name Qwen/Qwen3-Embedding-0.6B \
  --device cuda \
  --max-inputs-per-request 128 \
  --embedding-batch-size 32
```

## Technologies

Python, FastAPI, LangChain, Hugging Face embedding models, CUDA-oriented deployment, and PyPI packaging.

**Source:** [GitHub](https://github.com/MeshkatShB/remote-embedding)  
**Package:** [PyPI](https://pypi.org/project/remote-embedding/)
