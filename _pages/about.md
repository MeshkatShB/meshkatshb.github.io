---
permalink: /
title: "About"
author_profile: true
redirect_from:
  - /about/
  - /about.html
---

I am **Meshkat Shariat Bagheri**, a **Senior AI Engineer** based in Tehran, building enterprise search and workflow automation with **local LLMs**, **agentic AI**, and **retrieval-augmented generation (RAG)**. I connect language models with organizational documents, databases, and business tools to help people find reliable answers and complete everyday work.

I work on the **Almas Team at Gam Electronics**, where I joined as an AI Engineer in June 2025 and became a Senior AI Engineer in October 2026. I partner with Sales, Support, HR, and Production leads and the CEO to translate business requirements into AI applications and lead their architecture and deployment. My work spans internal tools serving 1–15 users each and AI features in customer-facing products with up to 10,000 users.

My approach puts access control, source evidence, evaluation, and human approval alongside model quality. Earlier software engineering roles at Gam Electronics and Najva gave me a foundation in .NET applications, Python/Django services, and database optimization.

## Focus Areas

- **Agentic AI and automation:** tool calling, multi-agent systems, LangGraph workflows, evaluation, auditable actions, and approval checkpoints.
- **Enterprise search:** RAG, GraphRAG, dense/BM25 hybrid retrieval, embeddings, OCR, and permission-aware answers.
- **Language and speech:** Persian writing assistance, summarization, speech recognition, information extraction, and document review.
- **Engineering stack:** Python, SQL, TypeScript, FastAPI, React, LangChain, LangGraph, vLLM, Ollama, Llama.cpp, PyTorch, Transformers, Langfuse, Milvus, Neo4j, OracleDB, SQL Server, PostgreSQL, Redis, Docker, and Linux.

## Selected Work

- **Enterprise letter search:** Architected a Milvus/Neo4j RAG pipeline that returns cited answers in **under 15 seconds**, with **approximately 99% accuracy on 100 evaluation questions**. Oracle permissions are enforced before retrieval and answer generation. OCR, incremental indexing, hybrid retrieval, and Persian ranking made scanned correspondence searchable in an archive of **250k+ records**.
- **CRM and business insights:** Built a Dynamics 365 copilot that reduced reporting time from approximately **5 minutes to 1 minute (80%)**. Delivered sales and proposal assistants using read-only SQL Server queries and traceable CRM evidence, plus a bonus assistant with deterministic payout calculations and natural-language explanations.
- **Persian writing and voice applications:** Developed local LLM APIs for correspondence drafts, revisions, and summaries. Built voice-based support with grounded answers and consent-based CRM case creation, alongside meeting transcription workflows that produce minutes and proposed Jira actions.
- **Workflow automation:** Converted monitoring emails and logs into CRM incidents and draft correspondence with LangGraph approval checkpoints. Built a Jira agent for worklog analysis, capacity forecasts, and Epic progress reports, with authenticated previews and approval for write actions.
- **Smart inbox and document review:** Designed an inbox that summarizes authorized letters, highlights deadlines and decisions, and recommends recipients using a LinUCB ranker trained on feedback. Built claim-level retrieval and contradiction checks across **seven document formats**, with source evidence for review.

## Education and Research

I am a **Ph.D. student in Information Technology Engineering at Shahid Beheshti University** (since September 2024). My proposed research explores **agentic AI for capability-centric business architecture**. I earned my **M.Sc. in Information Technology Engineering** (2024) and **B.Sc. in Computer Engineering** (2020) from the **University of Tehran**.

My master's research investigated long-text summarization using community detection and Graph Attention Networks (**CDLTS**). I also co-authored **[DCOR: Anomaly Detection in Attributed Networks via Dual Contrastive Learning Reconstruction]({{ '/publication/DCOR' | relative_url }})**, published in Springer's *Complex Networks & Their Applications XIII*.

From 2019 to 2025, I supported six courses as a teaching assistant at the University of Tehran, covering network analysis, systems theory, research methods, security, operating systems, and networking. I ranked **4th nationally** in the 2024 Ph.D. entrance exam for Information Technology.

## Open Source

- **[`remote-embedding`](https://github.com/MeshkatShB/remote-embedding):** I created this [PyPI package](https://pypi.org/project/remote-embedding/) to share one embedding model across applications through a FastAPI service and LangChain client, reducing GPU memory use from approximately **9 GB to 3 GB (67%)**. It supports CPU/GPU batching, request timeouts, dimension validation, and connection cleanup.
- **[Google LangExtract](https://github.com/google/langextract):** Added Llama.cpp support for local information extraction on CPUs and improved extraction visualization for Persian, Chinese, and Japanese.
- **[browser-use WebUI](https://github.com/browser-use/web-ui):** Improved default LLM configurations, settings, configuration warnings, runtime stability, and CLI documentation.
- **Learning resources:** I maintain [LangChain Zero to Mastery](https://github.com/MeshkatShB/langchain-zero-to-mastery) and [LLM Zero to Mastery](https://github.com/MeshkatShB/llm-zero-to-mastery), two tutorial repositories with practical implementations for LangChain and LLM development.
