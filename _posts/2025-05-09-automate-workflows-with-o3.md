---
title: "Automating workflows with o3"
date: 2025-05-09
permalink: /posts/2025/05/automate-workflows-with-o3/
tags:
  - GPT
  - o3
  - Reasoning
  - Tool
---

# Automating Multi-Step Workflows with o3

## Introduction

In the latest version of o3, OpenAI has taken reasoning and tool usage to a new level. The most exciting capability introduced is its ability to **combine multi-step reasoning with agentic tool use**—enabling it to complete complex tasks more autonomously than ever before.

---

## Running a Month-End Variance Report

Let’s walk through a demo of o3 running a **month-end variance report**.

### Step 1: Uploading the Prompt and Data

We begin by asking o3 to generate a variance report based on some dummy department data. Each uploaded spreadsheet contains:

- Budgeted vs. actual spend
- Department names
- Cost centers
- Forecast figures

### Step 2: Task Decomposition

Normally, this workflow would involve:

1. Harmonizing and analyzing the data
2. Flagging variances over a threshold (e.g., 7%)
3. Visualizing the data
4. Searching the web for benchmarks
5. Creating an executive summary for the CFO

o3 is now capable of automatically completing **each of these discrete steps**—calling the appropriate tool or performing the right operation at each phase.

---

## Execution and Tool Use

Once the task is sent off, o3 begins its **chain-of-thought reasoning**, visible in the live output:

- **Data Analysis**: o3 parses the uploaded CSV files, writes Python code, and flags ~20 out of 25 lines that exceed 7% variance.
- **Web Search**: It searches the internet for benchmarking data from reliable sources (e.g., KPMG).
- **Visualization**: It generates interactive graphs visualizing the flagged variances.
- **Insight Generation**: o3 provides analysis summaries and citable insights.

---

## Output Generation

In about a minute, o3 produces:

- Flagged variances (highlighted)
- Visual breakdowns and key takeaways
- Benchmarks from verified sources
- Executive summary
- Slack-ready post for the CFO

---

## Conclusion

This example highlights the shift from **reactive** AI to **agentic** AI. o3 didn’t just wait for instructions, it proactively:

- Broke the task into sub-goals
- Chose the best tools for each sub-goal
- Coordinated them autonomously
- Delivered a high-quality report in a fraction of the usual time

This is not just generative AI; **this is agentic execution at scale**.
