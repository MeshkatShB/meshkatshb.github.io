---
title: "Remote Embedding"
excerpt: "A PyPI package I built to run embedding models as a shared FastAPI service, reducing duplicated GPU memory usage across RAG and LLM applications."
collection: portfolio
priority: 0
---

**About:**  
`remote-embedding` is an open-source Python package I created and maintain for teams and developers who want to run embedding models once on a dedicated server and reuse them from multiple applications.

The package is available on [PyPI](https://pypi.org/project/remote-embedding/) and the source code is available on GitHub at [MeshkatShB/remote-embedding](https://github.com/MeshkatShB/remote-embedding).

```bash
pip install remote-embedding
```

---

## Why I Built It

Many RAG and semantic search systems load the same embedding model separately in each application, worker, or notebook. That pattern wastes GPU memory, slows down startup time, and becomes harder to manage as more tools depend on embeddings.

I built `remote-embedding` to make embedding inference reusable infrastructure:

- load the embedding model once on a GPU server
- expose embeddings through a FastAPI service
- call the server from Python applications
- use a LangChain-compatible remote client
- reduce repeated model loading across indexing, retrieval, and agent workflows

---

## Core Features

- **FastAPI embedding server:** Hosts a selected embedding model behind an HTTP API.
- **LangChain-compatible client:** Lets LangChain applications use the remote service as an embedding backend.
- **GPU-focused design:** Helps avoid duplicated VRAM usage when several applications need the same model.
- **Operational guardrails:** Supports bounded model caching, request-size limits, configurable embedding batch size, and health-check visibility.
- **Simple installation:** Published as a Python package through PyPI.

---

## Example Use Case

A common workflow is to run one embedding server for a model such as `Qwen/Qwen3-Embedding-0.6B`, then let multiple RAG pipelines, background indexing jobs, and local tools send embedding requests to that shared service.

```bash
remote-embedding-server \
  --model-name Qwen/Qwen3-Embedding-0.6B \
  --device cuda \
  --max-loaded-models 1 \
  --max-inputs-per-request 128 \
  --embedding-batch-size 32
```

---

## Technologies Used

- Python
- FastAPI
- LangChain
- Sentence Transformers / embedding models
- CUDA-oriented GPU deployment

---

## Key Takeaways

- I designed and implemented `remote-embedding` as a practical infrastructure package for embedding-heavy AI systems.
- The package targets real RAG and semantic search workloads where repeated embedding model loading becomes expensive.
- The project reflects my work in LLM infrastructure, retrieval systems, GPU optimization, and production-minded AI tooling.

**Explore the package:** [PyPI](https://pypi.org/project/remote-embedding/)  
**Explore the source code:** [GitHub](https://github.com/MeshkatShB/remote-embedding)
