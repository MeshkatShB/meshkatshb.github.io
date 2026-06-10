---
title: "Document Review and Fact-Checking Agent"
excerpt: "An agentic review system that extracts claims, retrieves evidence, and ranks contradictions, missing evidence, and logic changes."
collection: portfolio
priority: 2
---

I designed an internal document-review and fact-checking system for comparing claims against source evidence across mixed document formats.

The system ingests PDF, Word, PowerPoint, HTML, TXT, Markdown, and CSV files; extracts atomic claims; embeds them with Qwen3 embeddings; stores them in Milvus; and performs bidirectional claim-evidence retrieval.

## What It Detects

- Contradictions between claims and source evidence.
- Missing evidence and unsupported claims.
- Logic changes between related statements.
- Self-contradictions inside a document.
- Pair contradictions across documents.
- Conditional conflicts that need context-sensitive validation.

## Approach

The workflow combines rule-based checks, NLI-style validation, LLM context validation, vector retrieval, and structured ranking. This makes it more useful than a simple "ask the PDF" workflow: the system can surface review targets, explain why they matter, and prioritize likely issues.

## Technologies

Python, LangChain, local LLMs, Qwen3 embeddings, Milvus, document parsing, information retrieval, NLI, and RAG.
