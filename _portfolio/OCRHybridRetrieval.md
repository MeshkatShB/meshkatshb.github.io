---
title: "OCR and Hybrid Retrieval Pipeline"
excerpt: "OCR, indexing, and hybrid retrieval workflows for scanned enterprise documents and Persian-language search."
collection: portfolio
priority: 5
---

I developed OCR and indexing workflows for scanned enterprise documents stored in OracleDB, then connected the extracted text to hybrid retrieval systems for agentic search.

## Highlights

- Processed 250k+ OracleDB document records, each with multiple scanned pages.
- Used DeepSeek-OCR and other OCR/VLM models for text extraction.
- Indexed extracted content into Milvus for retrieval and agent workflows.
- Combined dense vector search with BM25 for better recall and precision.
- Added multi-concept AND ranking, adaptive thresholds, Persian exact-match boosts, duplicate-chunk cleanup, indexed-letter discovery, and nightly incremental indexing jobs.
- Built lookback windows for delayed Oracle rows and operational maintenance workflows for OCR/no-OCR indexing.

## Technologies

Python, OracleDB, Milvus, BM25, embeddings, DeepSeek-OCR, OCR/VLM models, Persian text retrieval, indexing jobs, and local LLM agents.
