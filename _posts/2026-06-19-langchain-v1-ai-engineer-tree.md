---
title: "LangChain v1 AI Engineer Tree: A Practical Map For Building Agents"
date: 2026-06-19
permalink: /posts/2026/06/langchain-v1-ai-engineer-tree/
tags:
- LangChain
- AI Agents
- Agentic AI
- LLM Engineering
- RAG
- Middleware
- Structured Output
- LangGraph
- LangSmith
- Python
---

LangChain v1 gives AI engineers a cleaner way to think about agent development. Instead of treating models, tools, prompts, memory, RAG, structured output, middleware, and observability as separate pieces, LangChain v1 puts the agent at the center. The main entry point is `create_agent`, and almost everything else connects to it: the model, tools, system prompt, middleware, response format, checkpointer, runtime context, state, invocation style, RAG layer, multi-agent patterns, and debugging stack.

# Understanding LangChain v1 As An AI Engineer Tree

LangChain has changed a lot from its earlier versions. In older LangChain workflows, it was common to think in terms of chains, prompt templates, tool wrappers, memory objects, retrievers, and executors as separate building blocks.

In LangChain v1, the mental model is more agent-centered.

The most important idea is simple:

```python
from langchain.agents import create_agent
```

This is the main entry point for creating an agent.

A typical agent looks like this:

```python
agent = create_agent(
    model=...,
    tools=[...],
    system_prompt=...,
    middleware=[...],
    response_format=...,
    checkpointer=...,
    context_schema=...,
    state_schema=...
)
```

So, if you want to understand LangChain v1 as an AI engineer, you should not only memorize functions. You should understand the tree around `create_agent`.

The agent is the root. Everything else is connected to it.

## Why This Tree Matters

When building real AI applications, the hard part is usually not just calling an LLM.

The hard part is answering questions like:

1. Which model should handle this task?
2. What tools should the agent be allowed to call?
3. How should the system prompt change based on the user or context?
4. How do I return structured JSON instead of plain text?
5. How do I persist conversation state?
6. How do I add RAG?
7. How do I debug tool calls and intermediate steps?
8. How do I add retries, guardrails, human approval, or PII filtering?
9. How do I build a multi-agent workflow?
10. How do I observe what happened during execution?

LangChain v1 tries to organize these concerns around the agent.

That is why I like thinking about it as an AI engineer tree.

## 1. Agent Entry Point

The root of the tree is:

```python
from langchain.agents import create_agent
```

You use `create_agent` to create an agent that can call a model, use tools, receive messages, maintain state, apply middleware, and return a result.

A minimal example looks like this:

```python
from langchain.agents import create_agent

agent = create_agent(
    model="openai:gpt-5",
    tools=[],
    system_prompt="You are a helpful assistant."
)

result = agent.invoke({
    "messages": [
        {"role": "user", "content": "Hello!"}
    ]
})
```

This is the simplest possible shape.

But in a real application, the same root can grow into a much more complete system:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[search_docs, get_weather, query_database],
    system_prompt="You are a helpful assistant with access to tools.",
    middleware=[...],
    response_format=...,
    checkpointer=...,
    context_schema=...,
    state_schema=...
)
```

The important point is that `create_agent` is where the major pieces come together.

## 2. Model

The model is the reasoning engine of the agent.

In LangChain v1, you can define the model in different ways.

The simplest way is to use a string model identifier:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[]
)
```

This is clean and useful for quick setups.

You can also pass a chat model object:

```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-5")

agent = create_agent(
    model=model,
    tools=[]
)
```

Depending on your provider, you may use different integrations:

```python
from langchain_openai import ChatOpenAI
from langchain_ollama import ChatOllama
from langchain_anthropic import ChatAnthropic
from langchain_google_genai import ChatGoogleGenerativeAI
```

This means the model layer is not locked to one provider. You can use OpenAI, Anthropic, Google Gemini, Ollama, or other supported providers.

A more advanced idea is dynamic model selection.

For example:

* Use a cheaper model for simple questions.
* Use a stronger model for complex reasoning.
* Use a local model for private tasks.
* Use a hosted model for tasks that need stronger performance.

In LangChain v1, this kind of routing is usually handled through middleware, especially with `wrap_model_call`.

So the model branch looks like this:

```text
Model
├── Simple string model id
├── Chat model object
└── Dynamic model selection through middleware
```

## 3. Tools

Tools are how the agent interacts with the outside world.

Without tools, the agent can only respond using the model. With tools, the agent can search documents, call APIs, query databases, inspect files, retrieve information, or perform actions.

The simplest way to define a tool is with the `@tool` decorator:

```python
from langchain.tools import tool

@tool
def search_docs(query: str) -> str:
    """Search internal documents."""
    return "Search results..."
```

Then you pass it to the agent:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[search_docs],
    system_prompt="Use the search_docs tool when the user asks about internal documents."
)
```

Tools can be simple functions, but in real systems they often need access to runtime data.

For that, LangChain provides `ToolRuntime`:

```python
from langchain.tools import ToolRuntime
```

This is useful when a tool needs access to context, state, configuration, permissions, or user/session information.

Common tool categories include:

```text
Tools
├── Tool decorator
├── Tool function
├── ToolRuntime
├── Built-in/community tools
│   ├── Tavily search
│   ├── Retriever tools
│   ├── SQL tools
│   ├── File tools
│   └── Custom API tools
└── Tool error handling
```

In production, tool design matters a lot.

A bad tool can make an agent unreliable. A good tool gives the agent a clear, safe, and useful action space.

## 4. Prompt And Instructions

The prompt tells the agent how to behave.

The simplest version is a static system prompt:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[search_docs],
    system_prompt="You are a helpful assistant. Be concise and accurate."
)
```

This is where you define the role, behavior, constraints, and tone of the agent.

For example:

```python
system_prompt = """
You are a technical documentation assistant.
Use the search_docs tool when answering questions about internal docs.
If you do not know the answer, say that you do not know.
Do not invent documentation details.
"""
```

Then:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[search_docs],
    system_prompt=system_prompt
)
```

The user input is passed as messages:

```python
agent.invoke({
    "messages": [
        {"role": "user", "content": "How do I configure the retriever?"}
    ]
})
```

This message format is important because agents are conversation-based.

But prompts do not always have to be static.

In many real systems, the prompt should change based on:

* The user.
* The tenant.
* The current task.
* The user permissions.
* The current state.
* The selected tools.
* The conversation summary.

This is where dynamic prompt middleware becomes useful.

The prompt branch looks like this:

```text
Prompt / instructions
├── Static system_prompt
├── Dynamic prompt middleware
└── Messages input
```

## 5. Middleware

Middleware is one of the most important parts of LangChain v1.

Middleware lets you customize what happens before, during, and after the agent runs.

You can think of middleware as the control layer around the agent.

It can help with:

* Summarizing long conversations.
* Trimming messages.
* Injecting context.
* Dynamically routing models.
* Adding guardrails.
* Filtering PII.
* Requiring human approval.
* Retrying failed model calls.
* Retrying failed tool calls.
* Logging and observability.

LangChain provides built-in middleware such as:

```text
Built-in middleware
├── SummarizationMiddleware
├── HumanInTheLoopMiddleware
├── PIIMiddleware
├── ToolRetryMiddleware
├── ModelRetryMiddleware
├── LLMToolSelectorMiddleware
├── TodoListMiddleware
└── LLMToolEmulatorMiddleware
```

You can also create custom middleware using hooks such as:

```text
Custom middleware
├── before_agent
├── before_model
├── after_model
├── after_agent
├── wrap_model_call
└── wrap_tool_call
```

For example, if you want to change the model based on the input, you can use model-call middleware.

If you want to block a tool unless the user has permission, you can use tool-call middleware.

If you want to inject profile data before the model sees the prompt, you can use a before-model hook.

This makes middleware one of the main production engineering layers in LangChain v1.

A simple way to think about it:

```text
Prompt defines behavior.
Tools define action.
Middleware defines control.
```

## 6. Structured Output

Sometimes you do not want a normal text answer.

You want a predictable object.

For example, instead of returning this:

```text
The user wants to book a flight from Tehran to Istanbul next week.
```

You may want this:

```json
{
  "origin": "Tehran",
  "destination": "Istanbul",
  "date": "next week"
}
```

In LangChain v1, structured output is configured with `response_format`.

A common pattern is to define a Pydantic model:

```python
from pydantic import BaseModel

class TravelRequest(BaseModel):
    origin: str
    destination: str
    date: str
```

Then pass it to the agent:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[],
    response_format=TravelRequest
)
```

LangChain can handle structured output through different strategies.

One is provider-native structured output:

```text
ProviderStrategy
└── Provider-native JSON/schema output
```

Another is tool-based structured output:

```text
ToolStrategy
└── Artificial tool-calling strategy for structured output
```

This is very important for application development.

Plain text is fine for chat. Structured output is better for APIs, automation, backend workflows, validation, and downstream systems.

## 7. Memory And Persistence

Memory allows the agent to continue a conversation instead of treating every request as isolated.

In LangChain v1, short-term memory is usually about conversation or thread state.

The key piece is the checkpointer:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[],
    checkpointer=...
)
```

Common checkpointing options include:

```text
Memory / persistence
├── Short-term memory
│   └── Conversation/thread state
├── Checkpointer
│   ├── InMemorySaver
│   ├── SQLite/Postgres checkpointers
│   └── LangGraph persistence backends
└── Long-term memory
    ├── Vector store
    ├── User profile store
    ├── Semantic memory
    └── External DB
```

Short-term memory means the agent remembers the current thread.

Long-term memory is different. It usually lives in an external store, such as a vector database, user profile database, semantic memory system, or application database.

A simple distinction:

```text
Short-term memory = what happened in this conversation.
Long-term memory = what the system should remember across conversations.
```

For example:

* Short-term memory remembers what the user asked five turns ago.
* Long-term memory remembers user preferences, project context, saved facts, or persistent knowledge.

In production, you usually need both.

## 8. Runtime And Context

Runtime context is information available during an agent run.

This can include:

```text
Runtime / context
├── user_id
├── permissions
├── tenant_id
├── session_id
├── organization_id
├── configuration
└── feature flags
```

LangChain v1 supports typed runtime context through `context_schema`.

For example:

```python
from dataclasses import dataclass

@dataclass
class UserContext:
    user_id: str
    permissions: list[str]
    tenant_id: str
```

Then:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[],
    context_schema=UserContext
)
```

Runtime context is useful inside middleware and tools.

For example, a tool may need to know:

* Which user is calling it.
* Which tenant the user belongs to.
* What data the user is allowed to access.
* Which model configuration should be used.
* Whether the user has permission to perform an action.

This is especially important in enterprise AI systems.

Two users may ask the same question:

```text
Show me the latest financial report.
```

But one user may have permission to access it, and another user may not.

Runtime context helps the agent behave differently depending on who is asking.

## 9. State

State is the data the agent carries during execution.

By default, LangChain agents use `AgentState`, which includes messages.

```text
Default AgentState
└── messages
```

For many agents, that is enough.

But sometimes you need custom state fields, such as:

```text
Custom state
├── current_task
├── retrieved_documents
├── approval_status
├── intermediate_results
├── conversation_summary
└── selected_tools
```

You can define custom state by extending `AgentState`:

```python
from langchain.agents import AgentState

class CustomState(AgentState):
    current_task: str
```

Then:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[],
    state_schema=CustomState
)
```

However, in many cases, it is better to define custom state through middleware when possible. This keeps the main agent setup cleaner and makes state-related behavior more modular.

A useful rule:

```text
Use context for external data passed into the run.
Use state for data the agent carries during execution.
Use memory for data that should persist across turns or threads.
```

## 10. Invocation

After creating the agent, you need to call it.

The simplest way is `invoke`:

```python
result = agent.invoke({
    "messages": [
        {"role": "user", "content": "Summarize this document."}
    ]
})
```

This returns the final answer after the agent finishes.

But many real applications need streaming.

You can stream values:

```python
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "Write a report."}]},
    stream_mode="values"
):
    print(chunk)
```

You can also stream messages or tokens:

```python
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "Write a report."}]},
    stream_mode="messages"
):
    print(chunk)
```

When using memory or checkpointing, configuration becomes important.

For example:

```python
config = {
    "configurable": {
        "thread_id": "user-123-thread-1"
    }
}

agent.invoke(
    {"messages": [{"role": "user", "content": "Continue our discussion."}]},
    config=config
)
```

The `thread_id` tells the checkpointer which conversation thread should be continued.

The invocation branch looks like this:

```text
Invocation
├── Final answer with invoke
├── Streaming values
├── Streaming messages/tokens
└── Config with thread_id
```

## 11. RAG Layer

RAG stands for Retrieval-Augmented Generation.

This is how the agent can answer questions using external documents instead of relying only on the model’s internal knowledge.

A typical RAG workflow looks like this:

```text
RAG layer
├── Load documents
├── Split documents
├── Embed documents
├── Store in vector DB
├── Create retriever
├── Wrap retriever as tool
└── Pass retriever tool into create_agent(tools=[...])
```

The key idea is that the retriever becomes a tool.

For example:

```python
agent = create_agent(
    model="openai:gpt-5",
    tools=[retriever_tool],
    system_prompt="Use the retriever tool when answering questions from the knowledge base."
)
```

This creates an agentic RAG pattern.

In a simple RAG chain, retrieval may happen automatically every time.

In an agentic RAG system, the agent can decide:

```text
Do I need retrieval?
What should I search for?
Should I search again?
Do I have enough evidence?
Can I answer now?
```

This is more flexible, but it also requires stronger evaluation, better tool design, and better observability.

## 12. Multi-Agent Layer

Sometimes one agent is enough.

Sometimes you need multiple agents with different responsibilities.

For example:

```text
Multi-agent layer
├── Research agent
├── Coding agent
├── Data analysis agent
├── Writing agent
└── Supervisor agent
```

There are two common patterns.

The first pattern is handoff.

In this design, one agent transfers control to another agent. For example, a supervisor agent may decide that a request should be handled by a coding specialist.

The second pattern is agent-as-tool.

In this design, one agent is exposed as a tool to another agent.

Conceptually:

```text
Main agent
├── search tool
├── database tool
├── coding agent tool
└── research agent tool
```

This is useful when you want specialists with different prompts, tools, permissions, or responsibilities.

However, multi-agent systems can become complex quickly.

Before using multiple agents, it is worth asking:

```text
Can one agent with good tools solve this?
Do the agents really need different responsibilities?
Do they need different tools or permissions?
Is handoff necessary?
Would agent-as-tool be simpler?
```

Multi-agent architecture is powerful, but it should be used when it actually reduces complexity, not when it only makes the system look more advanced.

## 13. Observability And Debugging

A production agent needs observability.

It is not enough to only see the final answer. You need to inspect what happened inside the run.

Important things to observe include:

```text
Observability / debugging
├── LangSmith tracing
├── Stream intermediate steps
├── Inspect tool calls
├── Inspect messages
├── Inspect state
├── Inspect runtime context
├── Track errors
├── Track retries
├── Track latency
├── Track token usage
└── Track cost
```

This helps answer questions like:

```text
Why did the agent call this tool?
Why did it not call a tool?
What prompt was sent to the model?
What did the tool return?
Where did the wrong answer come from?
Which step failed?
How much did this run cost?
```

LangSmith is commonly used for tracing, debugging, evaluation, and monitoring LangChain applications.

You can also add custom logging middleware to record application-specific events.

For serious AI engineering, observability is not optional. It is part of the system design.

## The Full LangChain v1 AI Engineer Tree

The full tree can be summarized like this:

```text
LangChain v1 AI Engineer Tree
│
├── Agent entry point
│   └── create_agent
│
├── Model
│   ├── Simple string model id
│   ├── Chat model object
│   └── Dynamic model selection
│
├── Tools
│   ├── @tool decorator
│   ├── Tool functions
│   ├── ToolRuntime
│   ├── Built-in/community tools
│   └── Tool error handling
│
├── Prompt / instructions
│   ├── system_prompt
│   ├── Dynamic prompt middleware
│   └── Messages input
│
├── Middleware
│   ├── Built-in middleware
│   ├── Custom middleware
│   └── Control logic
│
├── Structured output
│   ├── response_format
│   ├── Pydantic models
│   ├── ProviderStrategy
│   └── ToolStrategy
│
├── Memory / persistence
│   ├── Short-term memory
│   ├── Checkpointer
│   └── Long-term memory
│
├── Runtime / context
│   ├── context_schema
│   ├── runtime.context
│   └── User/session configuration
│
├── State
│   ├── Default AgentState
│   ├── Custom state
│   └── state_schema
│
├── Invocation
│   ├── invoke
│   ├── stream values
│   ├── stream messages
│   └── config
│
├── RAG layer
│   ├── Documents
│   ├── Embeddings
│   ├── Vector DB
│   ├── Retriever
│   └── Retriever tool
│
├── Multi-agent layer
│   ├── Subagents
│   ├── Handoffs
│   └── Agent-as-tool
│
└── Observability / debugging
    ├── LangSmith tracing
    ├── Intermediate steps
    ├── Tool calls
    ├── Messages/state
    └── Custom logging
```

## Final Thoughts

The most useful way to understand LangChain v1 is not as a collection of random APIs.

It is better to see it as an agent engineering framework.

At the center, you have:

```python
create_agent()
```

Around it, you attach:

```text
Model
Tools
Prompt
Middleware
Structured output
Memory
Runtime context
State
Invocation
RAG
Multi-agent patterns
Observability
```

Each part has a specific role.

1. The model thinks.

2. The tools act.

3. The prompt guides.

4. The middleware controls.

5. The response format structures.

6. The checkpointer remembers.

7. The context personalizes.

8. The state tracks.

9. The retriever grounds.

10. The subagents specialize.

11. The observability layer explains what happened.

That is the practical mental model. If you understand this tree, you can look at almost any LangChain v1 agent and know where each part belongs. With this overview, you can now build any agent you want, knowing how to handle different scenarios.
