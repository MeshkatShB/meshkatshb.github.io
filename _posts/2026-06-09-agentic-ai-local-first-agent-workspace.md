---
title: "Agentic-AI: Building A Local-First AI Agent Workspace"
date: 2026-06-09
permalink: /posts/2026/06/agentic-ai-local-first-agent-workspace/
tags:
  - AI Agents
  - Agentic AI
  - Local First AI
  - Ollama
  - RAG
  - MCP
  - Browser Automation
  - FastAPI
  - React
  - AI Tools
---

Agentic-AI is a local-first AI agent workspace that combines chat, document retrieval, tool execution, MCP integrations, browser automation, reminders, Telegram access, Exchange/EWS actions, and a Chrome extension into one practical assistant platform. It is built for people who want an AI agent that can work with local files, uploaded documents, web pages, APIs, scheduled tasks, and external tools while keeping control, permissions, and model choice in the user's hands.

# Building A Local-First AI Agent Workspace With Tools, Documents, MCP, Browser Automation, And Telegram

AI assistants are becoming more capable every month, but many of the most useful workflows still need more than a chat box. They need private documents, local context, real tools, scheduling, browser actions, external integrations, and a way to keep the user in control.

That is the idea behind Agentic-AI: a local-first AI agent workspace designed for people who want an assistant that can reason, search, use tools, and connect to real workflows while keeping the core experience close to the machine.

Agentic-AI brings together a FastAPI backend, a React interface, local model support through Ollama, vector search over uploaded documents, custom tool creation, MCP server integrations, browser automation, cron-based reminders, Telegram access, Exchange/EWS actions, and a Chrome extension for asking questions about the page you are viewing.

It is not just another chat UI. It is a practical agent platform for local productivity and experimentation.

## Why I Built It

Most AI tools are powerful, but they often live far away from the user's actual working environment. You ask a question in one place, copy information from another, manually search files elsewhere, then switch again to schedule a reminder, inspect a webpage, or call an API.

Agentic-AI is built around a different workflow:

1. Keep conversations and app data local by default.
2. Let the user choose the model provider.
3. Make tools explicit and permissioned.
4. Support real document retrieval instead of pasting the same files again and again.
5. Allow integrations through MCP and custom tools.
6. Turn reminders, Telegram access, browser tasks, and webpage Q&A into first-class capabilities.

The result is a workspace where the assistant can actually participate in the tasks around it.

## Local-First By Default

The default setup uses Ollama, SQLite, and Chroma. That means the app can run on your own machine with local models, local conversation history, local uploaded documents, and local vector storage.

For teams or advanced users, it also supports external LLM providers:

- OpenAI
- DeepSeek
- Mistral
- Google Gemini
- Ollama

Each user can configure their own provider, model, temperature, token limits, embedding model, tool permissions, and integration settings.

This gives the app a useful balance: local-first when privacy and control matter most, but flexible enough to use hosted models when the workflow needs them.

## A Chat Interface That Shows Its Work

The chat experience supports streaming responses, conversation history, file attachments, image inputs, page context from the Chrome extension, and detailed agent steps.

When the assistant uses a tool, the backend stores those actions as structured agent steps. This makes it easier to inspect what happened: which tool was called, what input it received, what came back, and how the assistant used the result.

That transparency matters. It turns the agent from a black box into something you can debug, trust, and improve.

## Document Uploads And RAG

Agentic-AI includes a document workflow for retrieval-augmented generation.

Users can upload:

- PDF
- DOCX
- TXT
- Markdown
- HTML

The backend stores the files per user, splits them into chunks, indexes them in the vector store, and makes them searchable through the `search_local_files` tool.

Once a document is indexed, the assistant can answer questions from it without the user repeatedly uploading or pasting the same context. This is useful for personal notes, technical documents, project references, reports, policy documents, and any knowledge base that should stay close to the user.

## Built-In Tools

The app includes a growing tool registry. Tools are permissioned, and users can decide what the agent is allowed to use.

Current tool families include:

- Local document search
- File reading
- Document parsing
- Web search
- Webpage scraping
- Generic HTTP requests
- Weather API lookup
- System information
- Code analysis
- Image metadata analysis
- Network utilities
- Hash calculation
- Database reads
- Scheduled reminders
- Exchange/EWS email, calendar, and tasks

The point is not just to give the model more power. The point is to give the user a controlled action layer.

## Custom Tools Without Editing The Codebase

Agentic-AI also supports custom tools created at runtime.

A user can define:

- Tool name
- Display name
- Description
- Permission level
- JSON parameter schema
- Python implementation

There is also an AI-assisted tool generator that can draft tool code and schemas from a natural language description.

Custom tools are stored in the database and registered into the tool registry when the user runs the agent. This makes the app extensible without requiring every workflow to become a permanent backend feature.

## MCP Server Integrations

Model Context Protocol support is another major capability.

Users can add MCP servers through the UI using:

- HTTP transport
- stdio transport

After testing a server connection, its tools can be discovered and added to the agent's available tool list. This means Agentic-AI can grow with the broader MCP ecosystem and connect to external tool providers without custom integration work for every service.

For anyone building agent workflows, MCP support is one of the most exciting parts of the project. It turns the app into a flexible host for many specialized capabilities.

## Reminders, Cron Jobs, And Notifications

Agentic-AI can schedule reminders directly from chat.

For example:

> Remind me tomorrow at 9am to review the deployment notes.

The assistant is instructed to call the scheduling tool rather than simply saying it will remember. The backend stores the job, runs a background scheduler, creates in-app notifications, records run history, and supports recurring schedules through cron expressions.

This makes the assistant useful beyond a single response. It can create future actions.

## Telegram Access

The app includes optional Telegram bot support.

Users can pair their account through a pairing code, then chat with the same agent from Telegram. Telegram can have its own tool selection and MCP server preferences, which is useful if you want a narrower, safer toolset on mobile.

This turns the assistant into something reachable outside the browser while still routing through the same local backend and user permissions.

## Browser Automation With Local Browsers

Agentic-AI also includes a browser automation page powered by `browser-use`.

The backend starts a local browser and uses the user's configured LLM provider to carry out browser tasks. It explicitly avoids cloud browser mode, keeping automation local.

This can be used for tasks like:

- Opening a webpage and summarizing it.
- Navigating a local app.
- Checking whether a page loads.
- Performing simple browser workflows.

It is a powerful capability, and the local-first design makes it especially useful for experimentation.

## Chrome Extension For Page-Aware Questions

The Chrome extension reads the current webpage and sends page context to the chat endpoint. Then the assistant can answer questions about what you are viewing.

Instead of copying page text manually, you can ask:

> What is this article saying about the main tradeoff?

or:

> Summarize this documentation page and tell me which steps apply to my setup.

The backend treats the provided page context as the source of truth, so the assistant can answer directly from the page instead of unnecessarily searching the web.

## Docker And Operations

The project includes Docker support for:

- Backend
- Frontend
- Optional Ollama
- Optional Qdrant
- Optional SearXNG

It also includes health endpoints, metrics, logs, migration checks, and operational docs for deployment and troubleshooting.

## Who It Is For

Agentic-AI is useful for:

- Developers building local AI workflows.
- Builders experimenting with agent tools.
- Teams exploring private document assistants.
- People who want local-first productivity automation.
- Anyone who wants a practical bridge between chat, tools, documents, browser tasks, and reminders.

It is intentionally modular. You can use it as a local assistant, a document Q&A workspace, an MCP host, a tool sandbox, a browser automation interface, or a Telegram-accessible agent.

## What Makes It Different

The most important thing about Agentic-AI is not one single feature. It is the combination:

- Local-first foundation.
- Multi-provider model support.
- Per-user permissions.
- Document retrieval.
- Built-in tools.
- Runtime custom tools.
- MCP integration.
- Scheduling.
- Telegram access.
- Browser automation.
- Chrome page context.

Together, these make the app feel less like a demo and more like a real agent workspace.

## Final Thoughts

Agentic-AI is my attempt to build an AI assistant that lives closer to real work: documents, tools, browsers, messages, schedules, and integrations.

The goal is simple: give users a private, extensible, transparent agent platform that they can run, inspect, adapt, and grow.

If you are interested in local-first AI agents, MCP, tool-using assistants, or practical personal automation, Agentic-AI is built for exactly that space.
