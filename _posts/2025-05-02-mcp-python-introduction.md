---
title: "Building a Custom MCP Server with Python"
date: 2025-05-02
permalink: /posts/2025/05/mcp-python-introduction/
tags:
  - MCP
  - Claude
  - AI Tools
  - Python
  - Anthropic
---

# Introduction

In this post, we’ll explore what the **Model Context Protocol (MCP)** is and how to build a **custom MCP server using Python**. If you've been hearing about MCP and wondering what it does or why it's important, this guide is for you.

---

## What is MCP?

**MCP** (Model Context Protocol) is an open standard developed by Anthropic that allows you to connect **external tools, resources, and prompt templates** to AI applications.

It’s often described as the **USB-C port for AI**, providing a standardized way to plug tools and context into models like Claude.

### Why MCP Matters

- ✅ Port custom integrations (e.g., Slack, Google Drive) into AI apps.
- 🔁 Reuse toolsets across environments (VS Code, Cursor, WindSurf).
- 🌍 Tap into the open ecosystem of MCP-compatible servers and tools.

---

## How MCP Works

MCP uses a **client-server architecture**:

- The **client** (e.g. Claude Desktop) sends requests.
- The **server** (your custom code) responds with tools, resources, or prompts.

Think of it like a **coffee shop**:

- You (client) ask for a Pumpkin Spice Latte.
- The barista (server) prepares and serves it.

---

## MCP Client

- Embedded inside AI applications.
- Handles discovery of tools and resources.
- Manages execution (LLMs can’t do it themselves).
- Already implemented in apps like Claude Desktop.

As a user or developer, you typically don’t need to build the client — **only the server**.

---

## MCP Server

The server is an **independent module** that:

- Listens for requests.
- Provides:
  - **Prompts** (e.g., resume writing templates)
  - **Resources** (static files, databases)
  - **Tools** (functions, APIs, scripts)

### Transport Methods

- `stdio` for local development.
- `HTTP` + `Server Sent Events (SSE)` for cloud deployments.

---

## Example: MCP Server with Python

Using Anthropic’s [Python SDK](https://github.com/anthropics/devtools), you can spin up a server that connects tools to Claude Desktop.

### Steps:

1. **Install `uv`** – a Rust-based fast Python environment manager.
2. **Use the SDK**:
   - Define prompts via `@mcp.prompt`
   - Load resources via `@mcp.resource`
   - Execute tools via `@mcp.tool`

Example tools:

- Writing to Gmail.
- Accessing CSV contact lists.
- Reusing markdown templates.

---

## Connecting to Claude Desktop

1. Install Claude Desktop.
2. Go to **Settings > Developer > Edit Config**.
3. Add the MCP server JSON config.
4. Restart Claude.

You’ll now see all the tools, resources, and prompts live inside Claude — ready to use!

---

## Summary

MCP makes AI apps:

- More customizable 🚀
- More interoperable 🧩
- More powerful 💡

You define your own workflows and carry them between environments or apps — all with just a bit of Python and MCP.

---

> Want to get started? Clone the [example repo](https://github.com/anthropics/devtools) and spin up your first MCP server today.
