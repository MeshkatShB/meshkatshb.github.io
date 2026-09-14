---
title: "Graph Engineering for AI Agents: Harnesses, Loops, Graphs, and Swarms"
date: 2026-09-04
permalink: /posts/2026/09/graph-engineering-ai-agents/
tags:
  - AI Agents
  - Agentic AI
  - Graph Engineering
  - Multi-Agent Systems
  - Agentic Harness
  - Loop Engineering
  - Agent Swarms
  - Google ADK
  - AI Engineering
---

Graph Engineering for AI Agents: Harnesses, Loops, Graphs, and Swarms

If you've been following recent trends in agentic AI, you may have noticed a new term showing up more often:

**Graph Engineering.**

At first, the name can be confusing. Is it about knowledge graphs? Graph databases? GraphRAG?

Not exactly.

In the context of AI agents, **graph engineering is about designing the behavior and control flow of an agentic system as a graph**.

That graph can contain:

- Agent nodes
- Function nodes
- Deterministic logic
- Routers
- Parallel branches
- Join points
- Human approval steps
- Other workflows

The graph defines **what happens, in what order, and what should happen next**.

---

## Harness vs. Loop vs. Graph

A lot of terminology in agentic AI overlaps, so it helps to separate three ideas:

- **Harness**
- **Loop**
- **Graph**

### Agentic Harness

The **harness** is everything around the model.

It may include:

- Tools
- Memory
- Guardrails
- Context management
- Execution environments
- Permissions
- Orchestration logic

The model is only one part of the complete agentic system.

### Agent Loop

The **loop** is the repeated cycle an agent runs inside that harness.

A simple loop might look like this:

```text
Reason
  ↓
Choose an action
  ↓
Use a tool
  ↓
Observe the result
  ↓
Reason again
```

The loop continues until the agent reaches its goal or decides it cannot continue.

### Graph

The **graph** describes the larger workflow.

Instead of one agent repeatedly looping, a graph can contain multiple steps, branches, agents, functions, and decision points.

For example:

```text
Start
  ↓
Collect Data
  ↓
Analyze
  ↓
Route
 ┌─────────────┐
 ↓             ↓
Fix Issue   Human Review
```

Each node performs some piece of work, while edges determine where execution goes next.

---

## Shared State Between Nodes

When a system traverses a graph, information often needs to move from one node to another.

This can be represented as **shared state**.

For example, in a multi-agent workflow built with a framework such as Google's Agent Development Kit, multiple agents or workflow nodes may share information such as:

```text
PR details
Test results
Code analysis
Security findings
Current status
```

That shared state allows downstream nodes to work with what earlier parts of the graph discovered.

---

# A Real Example: Automating Pull Request Review

A good example of graph engineering is an automated **pull request review workflow**.

PR review is a useful example because the steps are usually known ahead of time.

You can define the workflow explicitly.

One possible structure contains three major patterns:

1. **Fan-out**
2. **Join**
3. **Router**

---

## 1. Fan-Out: Run Multiple Checks in Parallel

Suppose a pull request needs several independent checks.

You might want to analyze:

- Code quality
- Security
- Tests
- Documentation
- Style

Instead of running them one after another, you can run them in parallel:

```text
             ┌─ Security Review
             ├─ Test Analysis
Pull Request ├─ Style Review
             ├─ Documentation Review
             └─ Code Quality Review
```

This is called **fan-out**.

The workflow starts several branches at once.

Parallel processing can significantly reduce the time required for the overall workflow.

---

## 2. Join: Combine the Results

Once the parallel branches finish, the workflow needs to bring their results back together.

That is the role of the **join node**.

```text
Security ──────┐
Tests ─────────┤
Style ─────────┼──► Join
Docs ──────────┤
Code Quality ──┘
```

The join waits for the required branches and then synthesizes their outputs.

The resulting state might contain something like:

```text
Security: PASS
Tests: FAIL
Style: PASS
Documentation: PASS
Code Quality: WARNING
```

Now the workflow has enough information to decide what should happen next.

---

## 3. Router: Decide Where the Workflow Goes

The next step is the **router**.

A router evaluates some condition and decides which branch of the workflow should execute.

For the PR example:

```text
                ┌── FAIL ──► Fixer Agent
Review Results ─┤
                └── PASS ──► Human Approval
```

If a check fails, the workflow can send the task to a specialized fixer agent.

If everything passes, the workflow can send the pull request to a human approval step.

This is very similar to control flow in traditional software engineering.

The difference is that some nodes may now contain AI agents rather than only deterministic functions.

---

# Graph Engineering Is Mostly About Behavior

One of the most important distinctions is between **graph engineering** and **knowledge graphs**.

The word *graph* appears in both, but they describe different things.

## Knowledge Graphs

Knowledge graphs focus primarily on **data and relationships**.

For example:

```text
Person ──works_at──► Company
Company ──located_in──► City
Person ──knows──► Person
```

The graph represents knowledge.

## Graph Engineering

Graph engineering focuses primarily on **behavior and execution flow**.

For example:

```text
Input
  ↓
Research Agent
  ↓
Validation Function
  ↓
Router
 ┌──────────┐
 ↓          ↓
Retry     Complete
```

The graph represents what the system does.

A useful distinction is:

> **Knowledge graphs describe relationships in data. Graph engineering describes relationships between actions.**

---

# Graph Engineering vs. Loop Engineering

Another related idea is **loop engineering**.

A loop typically contains one recurring execution cycle:

```text
Think
  ↓
Act
  ↓
Observe
  ↓
Repeat
```

The agent keeps running until it reaches a goal.

Graph engineering is broader.

A graph might contain:

- Several loops
- Multiple agents
- Deterministic functions
- Parallel branches
- Human approvals
- Routers
- External services

A loop can therefore be **one component inside a larger graph**.

---

## When Should You Use a Loop?

A loop can work well when the task is relatively simple and open-ended.

For example:

> Create a one-paragraph summary of this document.

The agent can inspect the input, reason, produce an answer, refine it if necessary, and stop.

The workflow does not necessarily need many explicitly defined stages.

---

## When Should You Use a Graph?

Graphs become more useful when the workflow is more complex and you already understand the major steps.

For example:

> Produce a 50-page report with research, charts, validation, formatting, and human approval.

That might require:

```text
Research
   ↓
Data Collection
   ↓
Analysis
   ↓
Chart Generation
   ↓
Drafting
   ↓
Quality Review
   ↓
Human Approval
```

Different steps may require different agents and different tools.

Graph engineering gives you more structure and control over the process.

---

# Graph Engineering vs. Agent Swarms

Graph workflows and agent swarms are both approaches to orchestrating multiple agents, but they are quite different.

## Graph Workflow

In a graph workflow, the engineer defines the structure.

You determine:

- Which nodes exist
- What each node does
- What data each node receives
- What conditions control transitions
- What should happen next

The behavior is relatively predictable.

This makes graph workflows attractive when the problem can be clearly defined.

Advantages include:

- Predictability
- Debuggability
- Control
- Easier testing
- Explicit state transitions

A node also does not necessarily need to understand the entire history of the workflow.

It may only need the state relevant to its specific task.

---

## Agent Swarm

An agent swarm is more flexible.

Instead of defining a strict workflow, you create several agents with roles or personalities and allow them to coordinate dynamically.

For example:

```text
Research Agent
Security Agent
Engineering Agent
Critic Agent
Product Agent
```

Then you give the group a problem.

The agents decide how to collaborate.

This can be useful when the problem is ambiguous and difficult to express as a predefined workflow.

---

# Predictability vs. Flexibility

The trade-off can be summarized like this:

| Graph Workflow | Agent Swarm |
|---|---|
| Engineer defines the workflow | Agents determine collaboration dynamically |
| More predictable | More flexible |
| Easier to debug | Harder to predict |
| Good for known processes | Good for ambiguous problems |
| Explicit control flow | Emergent control flow |
| Easier governance | Potentially more autonomous |

If you already know the workflow, a graph is often a natural choice.

If you do not know the workflow and want the agents to figure out how to solve the problem, a swarm may be more appropriate.

---

# Putting Everything Together

These concepts can coexist inside one system.

A complete agentic architecture might look like this:

```text
                         Agentic Harness
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Tools     Memory     Guardrails     Execution           │
│                                                          │
│                ┌──── Graph Workflow ────┐                │
│                │                        │                │
│                ▼                        ▼                │
│           Agent Node              Function Node          │
│                │                        │                │
│                │                        ▼                │
│                │                     Router              │
│                │                   ┌────┴────┐           │
│                ▼                   ▼         ▼           │
│           Agent Loop          Agent Node  Human Review   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

The terms describe different layers.

- **Harness** = the complete system around the model.
- **Loop** = the repeated reasoning/action cycle of an agent.
- **Graph** = the structured workflow connecting multiple nodes.
- **Swarm** = a more flexible collection of agents coordinating dynamically.

---

# Graph Engineering Is Control Flow for the Agentic Era

Once you remove the new terminology, graph engineering begins to look familiar.

Software engineers have used ideas such as:

- Branching
- Parallel execution
- State machines
- Routers
- Pipelines
- DAGs
- Queues
- Conditional logic

for decades.

Graph engineering applies many of those ideas to systems where some nodes are powered by AI agents.

Instead of writing:

```python
if tests_failed:
    fix_code()
else:
    request_approval()
```

you may now build:

```text
Test Agent
    ↓
Router
 ┌───────┴────────┐
 ↓                ↓
Fixer Agent    Human Approval
```

The concepts are familiar.

The components inside the workflow are changing.

---

# Final Thoughts

Graph engineering is not about inventing an entirely new form of computer science.

It is about applying structured workflow design to increasingly capable AI agents.

The core idea is simple:

> **Represent an agentic workflow as nodes and edges, then explicitly define how information and execution move through the system.**

This gives developers more control over complex multi-step AI systems.

A useful way to remember the terminology is:

```text
Harness = everything around the model

Loop = how an agent repeatedly reasons and acts

Graph = how multiple steps and agents are connected

Swarm = how multiple agents collaborate with less predefined structure
```

If the workflow is already known, a graph can give you strong predictability, observability, and control.

If the task is highly ambiguous, more dynamic patterns such as agent swarms may provide greater flexibility.

And in practice, many sophisticated agent systems will probably combine all of these ideas.

Maybe we really are reinventing classical control flow for the agentic age.

But that's not necessarily a bad thing.
