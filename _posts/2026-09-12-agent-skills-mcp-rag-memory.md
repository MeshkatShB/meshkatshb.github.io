---
title: "Agent Skills vs. MCP vs. RAG vs. Memory: How AI Agents Get the Knowledge They Need"
date: 2026-09-12
permalink: /posts/2026/09/agent-skills-mcp-rag-memory/
tags:
  - AI Agents
  - Agentic AI
  - Agent Skills
  - MCP
  - Model Context Protocol
  - RAG
  - Retrieval Augmented Generation
  - AI Memory
  - Context Engineering
  - AI Engineering
---

Agent Skills vs. MCP vs. RAG vs. Memory

An AI agent cannot rely only on what its underlying model learned during training. Real-world tasks often require information specific to your organization, systems, users, or previous incidents.

Four useful ways to give an agent that additional knowledge and capability are:

- **Agent Skills**
- **MCP**
- **RAG**
- **Memory**

They may sound similar, but they solve different problems. A useful way to understand them is through one concrete example.

---

# The Problem: A 500 Internal Server Error

Imagine a checkout page suddenly starts returning:

```text
500 Internal Server Error
```

We want an AI agent to investigate and resolve it.

The first instinct might be to gather everything—runbooks, dashboards, customer history, logs, and documentation—and put it all into the model's context window.

But simply adding more context does not necessarily make an agent better. It can get distracted by irrelevant information, follow dead ends, or behave too generically.

The better question is:

> **What kind of information or capability does the agent need at this particular moment?**

That is where Skills, MCP, RAG, and Memory become useful.

---

# 1. Agent Skills: How Should the Agent Perform the Task?

An **agent skill** is a reusable set of instructions for performing a particular kind of task.

A skill may contain procedures, instructions, decision criteria, escalation rules, and sometimes supporting code.

For our checkout failure, we could create a **Checkout Triage Skill**:

```text
1. Check the current error rate.
2. Inspect recent deployments.
3. Check the health of checkout dependencies.
4. Compare the failure against known incident patterns.
5. Escalate if the defined conditions are met.
```

Now the agent knows how your organization expects this type of problem to be investigated.

A good skill can also carry judgment—for example, when the agent should stop investigating autonomously and escalate to a human.

So a skill answers:

> **How should I perform this type of task?**

## Progressive Disclosure

You do not necessarily want every skill loaded into every prompt.

An agent may have skills for incident triage, database migrations, PR review, deployment, security investigations, and customer support. The relevant skill can be pulled in only when the task calls for it.

This keeps the working context focused.

But a skill has an important limitation: it can tell the agent to **check the error rate**, but it does not necessarily give the agent access to the monitoring system.

That is where MCP enters the picture.

---

# 2. MCP: How Does the Agent Reach External Systems?

MCP stands for **Model Context Protocol**.

A skill might tell the agent:

```text
Check the checkout error rate.
```

MCP can provide a standardized way for the agent to interact with the external system containing that information.

Conceptually:

```text
Agent
  │
  ▼
MCP
  │
  ├──► Logging System
  ├──► Metrics Platform
  ├──► Database
  └──► Other Services
```

The application containing the agent acts as the MCP host, while capabilities for external systems can be exposed through MCP servers.

The model itself does not need to know the proprietary details of how to query every backend.

## MCP in the 500 Error Example

Our triage skill says:

```text
Step 1: Check error rate.
Step 2: Check recent deployments.
```

With MCP, the agent can actually reach the relevant systems and retrieve those values.

```text
Skill:
"Check the error rate."

        ↓

Agent selects capability

        ↓

MCP call

        ↓

Metrics system

        ↓

Checkout error rate: 18.7%
```

The same approach can provide access to logs or other operational systems.

So MCP answers:

> **How can the agent interact with an external system?**

---

# Skills vs. MCP

The distinction is simple:

### Skill

```text
Check the error rate first.
```

### MCP

```text
Here is the capability that lets you retrieve the error rate.
```

The skill provides **procedural knowledge**.

MCP provides **access to external capabilities and systems**.

They complement each other.

---

# 3. RAG: Retrieve Knowledge That Has Been Written Down

At this point, the agent can follow the incident procedure and retrieve logs and metrics.

But it may still not understand the checkout architecture. It may need to know which services depend on each other, what the expected behavior is, or what the system documentation says.

This is where **RAG**, or **Retrieval-Augmented Generation**, becomes useful.

Instead of loading an entire knowledge base into the context window, RAG retrieves relevant information when it is needed.

Suppose your organization has:

- Manuals
- Architecture documentation
- Dependency maps
- Troubleshooting guides

When the agent asks:

> What services does the checkout API depend on?

the retrieval system searches the available documents and returns matching pieces.

```text
Agent Question
      │
      ▼
Retrieval
      │
      ▼
Knowledge Base
      │
      ▼
Relevant Chunks
      │
      ▼
Agent Context
```

## Semantic Search

RAG commonly uses semantic search to retrieve chunks related to the meaning of the query.

For example:

```text
Query:
"What does checkout depend on?"
```

could retrieve:

```text
The checkout service communicates with
PaymentGateway, InventoryService and OrderDB.
```

The useful information is then placed into the model's working context.

The objective is not maximum context.

It is **relevant context**.

So RAG answers:

> **What relevant documented knowledge should the agent know right now?**

---

# 4. Memory: What Has the Agent Learned From Experience?

Memory can initially look similar to RAG because both can retrieve useful information into context.

The key distinction is **where the knowledge comes from**.

With RAG, the knowledge comes from documents deliberately stored as a knowledge source.

With memory, the knowledge comes from **previous interactions or experiences that the agent has retained**.

Suppose the documentation says:

```text
Checkout failures are commonly caused by
PaymentGateway timeout errors.
```

That is documented knowledge and fits naturally with RAG.

But suppose three weeks ago the agent investigated the same error and discovered:

```text
A stale connection pool in OrderService
caused the PaymentGateway timeout indirectly.
```

That was learned through a previous incident.

That is the kind of information that can become **memory**.

## Memory Creates Experience

When a similar failure happens again, the agent can retrieve the previous experience:

```text
Previous Incident:

Symptoms:
- Checkout 500 errors
- Payment timeout
- Elevated latency

Root Cause:
- Stale OrderService connection pool

Resolution:
- Restart connection pool
- Verify database connections
```

Instead of solving the same problem entirely from scratch, the agent can benefit from what happened before.

Memory can also write back what was learned after a new incident is resolved.

So memory answers:

> **What have I learned from previous experience?**

---

# The Four Concepts Together

Return to our original checkout incident.

The relevant **skill** tells the agent how to investigate:

```text
1. Check error rate.
2. Inspect deployments.
3. Inspect dependencies.
4. Escalate under defined conditions.
```

**MCP** gives it access to external systems:

```text
Metrics
Logs
Deployment System
Other Services
```

**RAG** retrieves documented knowledge:

```text
Architecture docs
Dependency maps
Manuals
```

And **memory** provides previous experience:

```text
Similar incidents
Previous fixes
Observed failure patterns
Lessons learned
```

Together:

```text
                     ┌──────────────┐
                     │    Skill     │
                     │ How to act   │
                     └──────┬───────┘
                            │
                            ▼
┌──────────────┐      ┌───────────┐      ┌──────────────┐
│     RAG      │─────►│   Agent   │◄─────│    Memory    │
│ Documents    │      └─────┬─────┘      │ Experience   │
└──────────────┘            │            └──────────────┘
                            ▼
                     ┌──────────────┐
                     │     MCP      │
                     │ World access │
                     └──────┬───────┘
                            │
                            ▼
                     External Systems
```

Each component solves a different part of the problem.

---

# Skills vs. MCP vs. RAG vs. Memory

| Method | Main Purpose | Example |
|---|---|---|
| **Skills** | Give the agent a procedure to follow | How to triage an incident |
| **MCP** | Connect the agent to external systems and capabilities | Read logs or metrics |
| **RAG** | Retrieve relevant documented knowledge | Architecture documentation |
| **Memory** | Retrieve knowledge learned from previous experience | How a similar incident was fixed |

Another useful way to remember the distinction:

### Skills

> **How should I do this?**

### MCP

> **How can I access or interact with that system?**

### RAG

> **What documented information do I need?**

### Memory

> **What have I learned from previous experience?**

---

# They Are Not Competitors

A common mistake is to ask:

> Should I use MCP or RAG?

or:

> Should I use skills or memory?

These approaches solve different problems and can work together.

A single incident agent could use all four:

```text
Skill
  ↓
"Check recent deployment."

MCP
  ↓
Query deployment platform.

RAG
  ↓
Retrieve documentation about deployment architecture.

Memory
  ↓
Recall that a similar deployment caused this error previously.
```

The combination can be much more useful than treating them as competing alternatives.

---

# Why Context Engineering Matters

There is a broader lesson behind all four approaches.

Agent design is increasingly about deciding:

> **What should the model know at this exact moment?**

Simply stuffing everything into the context window is often not the best strategy.

Instead, we can selectively provide:

```text
Instructions → Skills

Capabilities → MCP

Documented Knowledge → RAG

Experience → Memory
```

This creates a more structured approach to context.

---

# A Practical Decision Rule

When designing an agent, ask where the missing knowledge or capability comes from.

If someone has defined **how a repeatable task should be performed**, consider a:

```text
Skill
```

If the agent needs to **interact with an external system**, consider:

```text
MCP
```

If the information exists in **documents or another curated knowledge source**, consider:

```text
RAG
```

If the information comes from **what the agent learned or observed previously**, consider:

```text
Memory
```

A simple rule of thumb is:

```text
Procedure   → Skill
Access      → MCP
Documents   → RAG
Experience  → Memory
```

---

# Final Thoughts

Giving an AI agent more information is not simply a matter of making its context window larger.

Different kinds of knowledge and capability should be provided in different ways.

**Agent Skills** provide repeatable procedures and judgment.

**MCP** connects the agent to external systems so it can retrieve information or act.

**RAG** retrieves relevant knowledge that has already been documented.

**Memory** allows the agent to benefit from what it learned through previous experiences.

A useful mental model is:

> **Skills tell the agent how to work. MCP lets it reach the world. RAG gives it documented knowledge. Memory gives it experience.**

And in real agentic systems, the most capable architecture may not choose between them at all.

It may use **all four together**.
