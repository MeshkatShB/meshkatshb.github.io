---
title: "[OpenAI] Structured Outputs: Unlocking Reliable AI Applications"
date: 2024-12-27
permalink: /posts/2024/12/structured-outputs/
tags:
  - OpenAI
  - DevDay
  - structured outputs
  - LLMs
  - API
---

OpenAI introduced **structured outputs** in August, a transformative feature in their API that ensures AI-generated outputs adhere to developer-specified JSON schemas. This innovation addresses long-standing challenges in working with LLMs, particularly reliability issues in text-to-JSON transformations. For full video, check out this link: [YouTube](https://www.youtube.com/watch?v=kE4BkATIl9c&ab_channel=OpenAI)

### Why Structured Outputs?

Since GPT-3's launch in 2020, LLMs have enabled applications across diverse fields, from generating marketing materials to customer service tools. By 2023, GPT-4 introduced advanced reasoning and action capabilities, but inconsistent outputs often hindered seamless integrations. Developers faced unreliable JSON formats, prompting the need for extensive prompt engineering or custom parsers.

**Structured outputs** eliminate these inefficiencies by allowing developers to enforce strict adherence to JSON schemas, improving reliability in applications such as API calls, UI generation, and workflow automation.

### How It Works

Structured outputs operate in two modes:

1. **Function Calling:** Enables models to generate JSON parameters for tool calls, integrating with application functionalities.
2. **Response Formats:** Ensures model responses conform to specific JSON schemas for direct user interactions.

Developers can activate structured outputs by setting the "strict" parameter to true, which guarantees responses strictly follow the predefined schema. This eliminates errors like invalid JSON or mismatched data types.

### Engineering Innovation

Structured outputs rely on constrained decoding, using token masking to limit the model's vocabulary at each step, ensuring valid outputs. By precomputing token masks and leveraging context-free grammars, OpenAI supports recursive and deeply nested JSON schemas, enhancing expressiveness and reliability.

### Research Advancements

Models were fine-tuned with diverse and complex schemas to improve their understanding of both syntax and semantics. As a result, model accuracy in adhering to schemas rose from 26% (GPT-4) to over 93% with structured outputs. Combining these advancements with constrained decoding achieves 100% reliability.

### Real-World Impact

Applications like Shopify have utilized structured outputs to reduce hallucinations and enhance reliability. Developers can now extract data, generate UI, and create robust multi-step workflows with minimal errors, unlocking the full potential of LLM-driven applications.

### OpenAI's Mission

Structured outputs exemplify OpenAI’s commitment to simplifying AI for developers, enabling them to focus on innovation while ensuring safe and reliable AI applications.
