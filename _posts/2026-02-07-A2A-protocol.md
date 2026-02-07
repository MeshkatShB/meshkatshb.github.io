---
title: "Collaborating AI Agents with A2A"
date: 2026-02-07
permalink: /posts/2026/02/A2A-protocol/
tags:
  - AI
  - A2A
  - Agent
  - Protocol
---

# Overview of Google’s A2A Protocol for AI Agent Collaboration

Today I’m sharing an in-depth overview of the **A2A protocol** created by Google — a standard designed to enable AI agents to communicate and collaborate with each other seamlessly, even if they’re built with entirely different systems.

---

## What Is A2A and Why It’s Useful

AI agents are becoming ubiquitous, but the challenge is that all these agents might be built on different frameworks and technologies. So how do we get them to cooperate on complex tasks?

Imagine planning a trip. Ideally, you’d have:

- A flight agent
- A hotel agent
- An activities agent

…but building each one is a ton of work. You might want to reuse someone else’s agent, but often it’s a black box — you don’t know how it’s implemented or how to work with it.

That’s exactly where **Agent-to-Agent (A2A)** protocol comes in.

A2A is an **open standard** that provides a common language for agent communication and collaboration — regardless of how the agent is implemented. Think of it like how LangChain made it effortless to switch between models; A2A allows multiple agents to communicate consistently.

An easy metaphor is **Lego blocks**:  
Each A2A agent broadcasts standardized info about itself and supports the same public methods, so any other agent can call it to help complete tasks. This opens up powerful orchestration possibilities.

---

## How A2A Structures Agent Communication

A2A connects three main roles:

- The **End User**
- A **Client Agent**
- A **Remote (Server) Agent**

A **Client Agent** generates requests and handles interaction with the end user. A **Remote Agent** receives those requests and attempts to respond or act as needed.

Importantly, any agent could act as both client and remote depending on context. It’s not a strict division — it’s dynamic.

---

## Adoption and Ecosystem Support

A protocol is only as useful as its adoption, and A2A has already seen strong interest in the software industry. While there’s a larger list in the official docs, many major partners have publicly committed to supporting the standard.

---

## Core Capabilities of A2A

Some of the main things A2A supports:

- **Dynamic discovery** of other agents
- **Standardized task collaboration**
- **Multimodal content sharing**
- **Support for long-running tasks**
- **Enterprise-grade security**

Importantly, A2A keeps each agent _opaque_ — meaning implementation details do not need to be exposed to follow the protocol.

---

## How the Protocol Works — A Walkthrough

To make this real, let’s walk through an example where **Agent A** needs **Agent B** to perform a task.

### How Agent A Discovers Agent B

Agent B publishes an **Agent Card** — a standard JSON file hosted at a known URI on its domain. This file acts like a digital business card and tells Agent A:

- Agent B’s name
- What it can do (skills/capabilities)
- Its HTTP endpoint for A2A communication
- Supported features (like streaming)
- Authentication information

Essentially, it functions like a combination of robots.txt and a microservices registry.

### How Agents Actually Talk

Communication is done over **HTTPS** using **JSON-RPC 2.0**. Each message represents one turn of a conversation and contains:

- A **role** — user or agent
- One or more **parts** — the actual content (text, files, structured JSON, etc.)

When Agent B receives a request, it processes it via something called an **Agent Executor** — a class that links the generic A2A protocol (handled by the SDK) to the agent’s specific logic.  
This executor lets the agent behave like a Lego block that can be easily connected to other agents.

---

## Handling Long-Running Tasks

Not all tasks complete quickly. Some require time, and the protocol handles that gracefully.

When an agent starts a long process:

1. The task gets a unique ID and status lifecycle
2. Initially it may respond with a “working” status
3. Agent A can periodically ask for status updates using the task ID
4. Later, Agent B will return a completed result inside the same task object

This polling approach works, but it’s not the most efficient.

### Streaming Updates (SSE)

If Agent B supports **streaming** (using Server-Sent Events), Agent A can keep an HTTP connection open and receive live updates:

- Initial task object
- Task status events
- Task artifact updates (e.g., breaking large outputs into chunks)

This enables **live progress feedback**, like a document appearing piece by piece as it’s generated — a much better user experience.

You can find a detailed tutorial and sample agents at the A2A tutorial link.

---

## Where A2A Fits in the Agent Stack

Is A2A all you need to productionize your agent?

Not exactly — A2A is **one part of the full agent stack**. Google recommends a layered stack, but you’re free to choose different components based on your needs.

You might also notice another protocol called **MCP** being used in agent ecosystems.

---

## How A2A and MCP Relate

A2A and MCP are **complementary**, not competitors:

- **MCP** standardizes how an agent connects to its own tools, APIs, and resources — essentially how it performs function calls.
- **A2A**, on the other hand, standardizes how _different independent agents_ talk to each other — sharing tasks and workflows.

In sophisticated agent systems, both will be used:

- Agents use **MCP** to interact with tools they control
- Agents use **A2A** to collaborate with other agents

---

## Get Started with A2A

The best way to learn is by trying it yourself:

- Check out the **full A2A specification** and documentation
- Use the official **Python SDK** (`pip install a2a-sdk`)
- Browse sample code in the A2A samples repo
- Explore the full set of official repos in the **Google A2A GitHub organization**

---

## Final Thoughts

A2A unlocks powerful workflows in a multi-agent world. Let me know what your plan is with agent collaboration using A2A!

---
