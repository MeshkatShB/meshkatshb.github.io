---
title: "Building Smarter AI with Deep Agents and Virtual File Systems"
date: 2026-02-07
permalink: /posts/2026/02/deep-agents-virtual-file-systems/
tags:
  - AI
  - Deep Agents
  - LangChain
  - Virtual File System
---

AI agents are powerful, but when it comes to **complex problems and long-running tasks**, basic agents often fall short. That’s where **Deep Agents** come in.

Deep Agents is a standalone library built on top of LangChain’s `create_agent` — but with a **battery-included approach** that gives agents advanced capabilities such as planning, task decomposition, context management, and deep sub-agent delegation.

In this post, I’ll walk through:

- The four core capabilities that make Deep Agents more reliable
- A practical example of giving an agent access to a **virtual file system**
- How Deep Agents use multi-origin backends to access structured data
- A complete example generating a tailored sales proposal

Let’s dive in.

---

## What Deep Agents Adds to the Standard Agent

When building agents for real-world use cases, we found there are **four core capabilities** that significantly improve reliability, especially for more complex or long-running problems.

### 1. **Planning & Task Decomposition**

Instead of single-shot actions, Deep Agents can **break tasks into smaller steps** — planning before execution. This is enabled through a **to-do middleware** layer that orchestrates subtasks.

### 2. **Context Management with a Virtual File System**

Agents often need to reference large amounts of data, but context windows are limited. Deep Agents solve this using a **file system abstraction** that lets agents browse and load context on demand.

### 3. **Sub-Agent Spawning**

Rather than tackling all problems within a single agent, Deep Agents can **spawn sub-agents** designed for specific tasks. This helps avoid context overflow and isolates reasoning into manageable chunks.

### 4. **Long-Running Memory**

Deep Agents can remember what happened across previous conversations — not just within one run — giving them long-running memory across sessions.

Together, these make agents far more capable regardless of the underlying model.

---

## Example: Virtual File System for Tailored Sales Proposals

Now let’s look at an example where we build a Deep Agent that generates **tailored sales proposals** with access to multiple data sources through a **virtual file system**.

In this system, the virtual file system is composed of three backends:

1. An **SQL database backend** — giving the agent access to user information and prior sales conversations
2. An **S3-compatible storage backend** — providing company policies, pricing, and detailed documents
3. A **local file system backend** — allowing the agent to write or print the final report

### Setting Up Backends

First, we define a **backend factory** that returns a composite backend mapping directories to each storage layer:

- The `workspace` directory maps to a local file system
- `users` maps to the SQL backend
- `s3_bucket` maps to the S3 storage backend

This lets the agent access files without knowing which backend they come from — the library handles those details.

The SQL backend can map database rows to virtual file paths, so listing files under `users/` will return database records as files. Likewise, the S3 backend maps files from buckets directly.

---

## Building and Running the Agent

Next, we set up various Deep Agent components:

- A **system prompt** that instructs the agent where to find specific information
- A **model** to drive reasoning
- A **checkpointer** to store conversation history
- The `create_deep_agent()` function, which acts like `create_agent()` but includes Deep Agents features like virtual file systems

With everything defined, we send the agent a prompt to **generate a personalized sales proposal** for a user named _Sarah Chen_. We also specify where the final report should be stored.

When running the example:

- A workspace directory is created
- The agent processes its available data
- After a few seconds, it writes the final report into the workspace
- The contents of that report are logged back to the terminal

Examining the generated proposal reveals a tailored email to Sarah — formatted using data from the SQL database, S3, and contextual information drawn from multiple sources.

---

## What This Enables

By integrating virtual file systems and intelligent orchestration into agents:

- Agents can access **large structured datasets** without manual context injection
- Multiple backends become transparent to the agent
- Sales proposals, reports, and other complex artifacts can be generated reliably

Virtual file systems are only one of Deep Agents’ core capabilities. Future posts will explore:

- **Sub-agent orchestration** for deep reasoning without context overflow
- Running arbitrary code safely using **sandboxes**
- Advanced planning and decomposition patterns

---

## Conclusion

Deep Agents represent an evolution in agent design — empowering AI with structured access to persistent knowledge and enabling more reliable, long-running, and complex workflows.

With planning, context management, sub-agent spawning, and memory, agents can do more than ever before — and remain manageable and predictable.

Thanks for reading, and stay tuned for the next deep dive into agent engineering!
