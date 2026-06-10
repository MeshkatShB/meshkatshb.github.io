---
permalink: /
title: "About"
author_profile: true
redirect_from:
  - /about/
  - /about.html
---

I am **Meshkat Shariat Bagheri**, an **AI Engineer** focused on enterprise-grade **agentic AI**, **local LLM systems**, **RAG applications**, and **NLP**. My current work is practical and local-first: building agents that connect private models, company documents, databases, messaging tools, CRM systems, and internal workflows without sending sensitive data out of the organization.

My recent engineering work centers on local LLM agents with tool calling, retrieval pipelines, hybrid vector search, OCR over scanned enterprise documents, CRM case intelligence, document-review agents, natural-language database search, and Persian ASR. I like systems where the model is only one part of the machine: permissions, retrieval quality, observability, indexing jobs, evaluation hooks, and user feedback loops matter just as much.

I am currently an **AI Engineer at Gam Electronics** on the Almas Team, where I build production-ready local AI systems integrated with OracleDB, Microsoft CRM/Dynamics 365, RocketChat, Zulip, Exchange/EWS, Milvus, and internal document platforms.

I also created and maintain [`remote-embedding`](https://github.com/MeshkatShB/remote-embedding), a Python package on [PyPI](https://pypi.org/project/remote-embedding/) that exposes embedding models through a shared FastAPI service and a LangChain-compatible client. The goal is simple: load an embedding model once, then let multiple RAG and semantic-search applications reuse it without duplicating GPU memory.

## Focus Areas

- **Agentic AI:** tool-calling agents, multi-step workflows, memory, streamed responses, tool-call traces, evaluation, and local-first deployment.
- **RAG and retrieval:** dense + BM25 hybrid search, semantic search, exact-match boosts, score thresholds, chunk cleanup, source previews, and permission-aware retrieval.
- **Enterprise AI systems:** CRM retrieval and analytics, document search, invoice generation, meeting-minutes generation, email RAG, quality scoring, and operations automation.
- **NLP and research:** graph neural networks, long-text summarization, information retrieval, anomaly detection in attributed networks, and multilingual AI.
- **Engineering stack:** Python, LangChain, LangGraph, Ollama, Llama.cpp, PyTorch, Transformers, FastAPI, Streamlit, Gradio, Milvus, FAISS, Qdrant, OracleDB, PostgreSQL, Redis, Docker, and Linux.

## Selected Work

- Built a fully local enterprise RAG agent platform with reasoning models, company documents, tool catalogs, conversation memory, streamed responses, and traceable tool calls.
- Developed an access-controlled letter-search agent with Exchange login, Oracle-backed access mapping, per-letter permission checks, Milvus retrieval, reconstructed full-text answers, previews, and feedback analytics.
- Engineered OCR pipelines using DeepSeek-OCR and other OCR/VLM models for 250k+ OracleDB document records with multiple scanned pages per record.
- Improved Milvus retrieval quality with dense + BM25 hybrid search, multi-concept ranking, Persian exact-match boosts, adaptive thresholds, duplicate cleanup, and nightly indexing jobs.
- Designed document-review and fact-checking agents that extract claims, retrieve evidence, and identify contradictions, missing support, logic changes, and conditional conflicts.
- Built natural-language-to-OracleDB search, CRM resolved-case retrieval, CRM analytics, email RAG, meeting-minutes generation, invoice generation, page-aware browser workflows, and Persian ASR applications.

## Education and Research

I am a **Ph.D. student in Information Technology Engineering at Shahid Beheshti University**, working around Generative AI in Business Architecture. I earned my **M.Sc. in Information Technology Engineering** and **B.Sc. in Computer Engineering** from the **University of Tehran**.

My master's thesis, **"Text Summarization with Graph Neural Networks,"** introduced a Community Detection Long-Text Summarization model combining community detection with graph attention networks. My publication work includes **DCOR: Anomaly Detection in Attributed Networks via Dual Contrastive Learning Reconstruction**, published in *Complex Networks & Their Applications XIII*.

## Open Source

Alongside `remote-embedding`, I contribute to local-first and AI tooling projects, including Google LangExtract and browser-use WebUI. My learning repositories, including LangChain Zero to Mastery and LLM Zero to Mastery, collect hands-on notes and practical implementations from my work with LLMs, RAG, NLP, and software engineering.
