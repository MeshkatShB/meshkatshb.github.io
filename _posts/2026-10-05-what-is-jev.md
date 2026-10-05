---
date: 2026-10-05
permalink: /posts/2026/10/what-is-jev/
tags:
- Jev
- Artificial Intelligence
- System 1 AI
- AI Engineering
- LLM
- Reasoning Models
- Reinforcement Learning
- AI Guardrails
title: "Jev - System 1 AI: Fast, Calibrated Decisions Beyond Generative LLMs"
---

Most AI models we interact with today are **generative**. Give them a
prompt and they produce text token by token. Reasoning models go further
by spending additional computation on difficult problems before
producing an answer.

But not every AI problem requires generation or deep reasoning.

Many decisions inside software are much simpler:

-   Is this a refund request?
-   Which team should handle this ticket?
-   How urgent is this message?
-   Is this input potentially malicious?

These are closer to quick judgment calls than open-ended generation.

That is the idea behind **System 1 AI**: models designed for fast
decisions rather than long-form generation. A recent example is **Jev**,
a model from TypeSafe that does not generate text as its primary output.
Instead, it selects among defined possibilities and returns
probabilities for those decisions.

The broader idea is more interesting than any individual model: **not
every step in an AI system needs an LLM generating tokens.**

------------------------------------------------------------------------

# System 1 vs. System 2 Thinking

The terminology comes from Daniel Kahneman's *Thinking, Fast and Slow*.

Kahneman describes two broad modes of human thinking.

**System 1** is fast and automatic. If someone asks `2 × 2`, you
probably answer `4` immediately.

**System 2** is slower and more deliberate. If someone asks `17 × 24`,
most people need to work through the calculation.

This gives us a useful analogy:

``` text
System 1                    System 2
Fast                        Slow
Automatic                   Deliberate
Frequent                    Reasoning-heavy
Decision-oriented           Generative / analytical
```

Modern reasoning models resemble the System 2 side of this analogy. But
much of software consists of much faster decisions.

------------------------------------------------------------------------

# A Simple Example: Customer Support Routing

Imagine a customer sends:

``` text
I was charged twice this month.
Please fix it.
```

A support system may need to answer three questions:

``` text
Is this a refund request?
Yes / No

Which team should receive it?
Billing / Technical / Sales

How urgent is it?
Low / Medium / High / Critical
```

A human can usually make these judgments almost immediately.

We could send the message to an LLM and request structured JSON:

``` json
{
  "refund_request": true,
  "team": "billing",
  "urgency": "high"
}
```

That works, but it leaves another important question:

> **How confident should the software be in each decision?**

------------------------------------------------------------------------

# The Problem With Asking an LLM for Confidence

You can ask a generative model to return:

``` json
{
  "refund_request": true,
  "confidence": 0.97
}
```

But that confidence value is itself generated text. It does not
necessarily represent a properly calibrated probability that the
classification is correct.

A model can sound extremely confident while being wrong.

That matters when AI output controls software behavior.

There is a meaningful difference between:

``` text
"This is probably a refund request."
```

and a genuinely calibrated:

``` text
P(refund request) = 0.92
```

If the probability is calibrated, software can use it directly when
deciding whether to automate an action or involve a human.

------------------------------------------------------------------------

# Decision Models Instead of Text Generators

A System 1-style model such as Jev approaches the problem differently.

Instead of asking it to generate prose, we provide **state** and
**questions**.

The state contains information relevant to the decision:

``` text
Customer email
Recent customer charges
Account information
Relevant transaction context
```

Then we ask questions about that state.

For a binary decision:

``` text
Is this a refund request?

P(yes) = 0.90
```

For a choice:

``` text
Which team should handle this?

Billing:   0.85
Technical: 0.10
Sales:     0.05
```

For urgency, the output can represent a score or probabilities across an
ordered scale.

The key property is that the model returns **decision probabilities**,
not an open-ended textual answer.

------------------------------------------------------------------------

# Why This Can Be Faster

A generative model produces an answer token by token. Even a short JSON
response requires generation and then parsing.

A decision-oriented model can instead follow a simpler abstraction:

``` text
Input
  ↓
Decision Model
  ↓
Probabilities
```

For workloads containing huge numbers of small classifications, routing
decisions, or judgments, avoiding unnecessary text generation can reduce
latency and cost.

------------------------------------------------------------------------

# Calibration: The Important Part

Returning probabilities is only useful if those probabilities mean
something.

Suppose a model repeatedly makes predictions with confidence `0.8`.

A well-calibrated model should be correct roughly 80% of the time on
predictions of that kind.

Conceptually:

``` text
Predicted probability ≈ Observed correctness
```

A useful intuition is:

``` text
0.2 confidence → correct ~20% of the time
0.5 confidence → correct ~50% of the time
0.8 confidence → correct ~80% of the time
0.9 confidence → correct ~90% of the time
```

If predictions labeled 90% confidence are only correct 60% of the time,
the model is poorly calibrated.

Calibration turns confidence into something software can reason about.

------------------------------------------------------------------------

# Why Generative Models Can Sound More Certain Than They Are

A simplified modern LLM training pipeline often contains:

``` text
Pre-training
     ↓
Post-training
```

During pre-training, the model learns from huge amounts of text by
predicting tokens.

Post-training then shapes it into something more useful for interaction.

## RLHF

With **Reinforcement Learning from Human Feedback**, people compare
model responses and provide preference signals.

This can make models more useful, but people often prefer responses that
sound clear and confident.

That means fluent confidence and calibrated confidence are not
necessarily the same thing.

## RLVR

**Reinforcement Learning with Verifiable Rewards** uses outcomes that
can be checked automatically.

For example:

``` text
Math answer → Correct / Incorrect
Code → Unit tests pass / fail
```

This has helped reasoning models become much stronger at math and
coding.

But the reward generally focuses on whether the final answer is correct,
not whether the model accurately represents its uncertainty.

------------------------------------------------------------------------

# Training for Calibrated Decisions

The source describes Jev as using **RLCD --- Reinforcement Learning for
Calibrated Decisions**.

Public details about the architecture are limited, so the important idea
is the objective rather than assumptions about the internal
implementation.

The model returns probabilities and is rewarded when those probabilities
accurately reflect outcomes.

The goal is therefore not only:

``` text
Choose the correct answer.
```

It is closer to:

``` text
Choose the answer
AND
estimate its probability correctly.
```

That difference becomes important when AI is embedded inside
deterministic software workflows.

------------------------------------------------------------------------

# Turning Probabilities Into Software Logic

Suppose the model returns:

``` text
P(refund_request) = 0.96
```

Software can apply explicit thresholds:

``` python
if probability >= 0.90:
    send_to_refund_queue()
elif probability <= 0.10:
    continue_normal_processing()
else:
    request_human_review()
```

The AI does not need to control the entire workflow.

It provides a probabilistic judgment, and deterministic software decides
what to do with it.

``` text
Unstructured Input
       ↓
AI Decision Model
       ↓
Calibrated Probability
       ↓
Deterministic Policy
       ↓
Action
```

------------------------------------------------------------------------

# Human-in-the-Loop Based on Uncertainty

Calibrated probabilities also enable smarter escalation.

Instead of sending everything to a human or trusting the AI with
everything:

``` text
High-confidence positive → Automate

Uncertain                 → Human review

High-confidence negative → Normal negative path
```

The exact thresholds should depend on the cost of mistakes.

A low-risk classification might tolerate a lower automation threshold. A
decision involving money, security, or legal consequences may require
much higher confidence.

This creates a direct relationship:

``` text
Model uncertainty
      ↓
Business risk
      ↓
Automation policy
```

That is much easier to operationalize than a model simply saying, "I'm
very confident."

------------------------------------------------------------------------

# System 1 Models as AI Guardrails

Fast decision models can also sit around other AI systems.

Before a message reaches a chatbot:

``` text
User Input
    ↓
Guardrail Model
    ↓
Main LLM
```

The guardrail could evaluate questions such as:

``` text
Is this a jailbreak attempt?
Does this contain prohibited content?
Does it contain sensitive information?
Should it require additional review?
```

The same pattern can be applied to outputs:

``` text
LLM Response
     ↓
Guardrail Model
     ↓
Allow / Review / Block
```

A fast classifier with calibrated probabilities can be attractive for
high-volume decision layers like these.

------------------------------------------------------------------------

# System 1 Models Do Not Replace LLMs

The conclusion is not that fast decision models replace large language
models.

They solve different problems.

A decision model is useful for:

``` text
Classification
Routing
Scoring
Filtering
Guardrails
Triage
Risk estimation
```

An LLM or reasoning model is useful when the system needs:

``` text
Generation
Explanation
Conversation
Complex reasoning
Planning
Synthesis
Open-ended responses
```

The strongest architecture may combine both.

------------------------------------------------------------------------

# Combining System 1 and System 2 AI

Return to the customer support example.

A workflow could look like:

``` text
Customer Email
      ↓
System 1 Model
      ↓
Classify:
- Refund?
- Team?
- Urgency?
      ↓
Routing Logic
      ↓
LLM / Reasoning Model
      ↓
Generate Customer Response
      ↓
System 1 Model
      ↓
Classify Follow-up
```

The fast model handles repeated judgment calls.

The larger generative model is invoked when the task requires language
generation or deeper reasoning.

This is the core architectural insight:

> **Use expensive reasoning where reasoning is needed, and fast decision
> models where a quick judgment is enough.**

------------------------------------------------------------------------

# Toward More Heterogeneous AI Systems

Many applications today send nearly every AI-shaped problem to one large
generative model:

``` text
Everything
    ↓
Large Generative Model
```

Future AI architectures may become more heterogeneous:

``` text
Incoming Task
      ↓
Task Router
 ┌────┼─────────────┐
 ↓    ↓             ↓
Fast  LLM       Deterministic
Model Reasoning     Code
 ↓    ↓             ↓
 └────┴─────────────┘
      ↓
   Workflow
```

Different computational tools can handle different kinds of problems.

Traditional software already works this way: we do not use a database
for every problem or a GPU for every computation.

There is little reason to expect one class of AI model to be optimal for
every cognitive task.

------------------------------------------------------------------------

# Where Fast Decision Models Could Matter

If AI judgment becomes sufficiently cheap and fast, it can appear in
places where repeatedly invoking a large generative model would be
impractical:

``` text
Every support ticket
Every incoming message
Every transaction
Every database row
Every log entry
Every agent action
Every generated response
```

That creates interesting possibilities for:

-   Continuous classification
-   Intelligent routing
-   Anomaly triage
-   AI guardrails
-   Agent supervision
-   Risk scoring
-   Large-scale data processing

The key is making each decision inexpensive enough to perform at very
high volume.

------------------------------------------------------------------------

# Limitations Still Matter

Decision-oriented AI is not magic.

The source notes that Jev currently takes text input, is not designed
for tasks such as mathematical reasoning or counting, and can still be
affected by malicious or misleading instructions embedded in the text it
reads.

Probabilities also do not make a model infallible.

Calibration means:

> When the model says it is 90% confident, predictions of that kind
> should be correct approximately 90% of the time.

It does **not** mean that a particular 90% prediction is guaranteed to
be correct.

Good system design still requires:

-   Thresholds
-   Validation
-   Human escalation
-   Monitoring
-   Security controls
-   Evaluation on the actual application domain

------------------------------------------------------------------------

# The Jevons Paradox Connection

The name **Jev** references economist William Stanley Jevons.

Jevons observed that improvements in the efficiency of steam engines did
not necessarily reduce coal consumption. Greater efficiency made steam
power useful in more places, potentially increasing total consumption.

This became known as **Jevons paradox**.

There is an interesting analogy for AI.

If an AI judgment becomes dramatically cheaper and faster, organizations
may not simply spend less on AI.

They may use AI in many more places.

Instead of:

``` text
100 expensive AI decisions
```

we might eventually see:

``` text
10,000,000 cheap AI decisions
```

Efficiency can expand the set of economically viable use cases.

------------------------------------------------------------------------

# Final Thoughts

The rise of reasoning models has pushed AI toward deeper and more
deliberate computation.

But there is another important direction:

> **Make simple AI decisions extremely fast, cheap, and measurable.**

Not every problem needs a model to generate paragraphs of text or spend
seconds reasoning through a chain of steps.

Sometimes software simply needs to know:

``` text
Yes or no?
Which category?
How urgent?
How risky?
How confident are we?
```

For those problems, calibrated decision models represent an interesting
alternative.

The broader architecture may eventually look less like one giant model
doing everything and more like a collection of specialized cognitive
components:

``` text
System 1 Models
    +
Reasoning Models
    +
Generative LLMs
    +
Deterministic Software
    +
Human Oversight
```

The important engineering question is no longer simply:

> **Which AI model should we use?**

It is:

> **Which kind of intelligence should handle each decision in the
> system?**
