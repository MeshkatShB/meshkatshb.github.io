---
title: "Next Concept Modeling: Generating Text from Coarse to Fine"
date: 2026-10-22
permalink: /posts/2026/10/next-concept-modeling-ncm/
tags:
  - AI Engineering
  - Language Models
  - Next Concept Modeling
  - Diffusion Models
  - Text Generation
  - Research
---

Most language models generate text **autoregressively**: one token after another. Each new token depends on the tokens already generated. This is effective, but it imposes a sequential dependency even on hardware capable of substantial parallel computation.

Researchers at Stanford's Hazy Research group have explored another approach: **Next Concept Modeling (NCM)**. Instead of writing a response from left to right, NCM predicts a hierarchy of discrete representations, beginning with a coarse sketch of the full sequence and progressively adding detail.

The interesting result is not that NCM has replaced conventional LLMs. It is that **text generation need not always proceed one token at a time**.

## The bottleneck in autoregressive generation

An autoregressive model follows a process like this:

```text
Prompt → Token 1 → Token 2 → Token 3 → ... → Token N
```

The next step must wait for the previous one. During generation, the system repeatedly uses model parameters and context; the precise performance bottleneck depends on hardware, model design, and serving configuration. Still, the sequential structure creates a fundamental limit on how much of one response can be generated simultaneously.

## Why not use diffusion?

Diffusion language models provide a different strategy. They begin with masked or incomplete output positions and refine them over multiple passes.

```text
Masked sequence
      ↓
Refinement pass 1
      ↓
Refinement pass 2
      ↓
Refinement pass 3
      ↓
Completed text
```

Multiple positions can be processed together, but there is a trade-off: too few refinement steps may reduce quality, while repeated passes can be computationally expensive because they revisit much of the sequence.

NCM takes a different route. Rather than repeatedly refining the same representation, it generates **successively more detailed levels**.

## What does “concept” mean in NCM?

The name can be misleading. A *concept* here is not necessarily a human-readable idea, sentence, or outline bullet.

In NCM, concepts are **discrete codes in a learned multiscale representation**. Earlier levels capture coarser information; later levels supply details missing from those earlier representations.

Think of sketching an image: first the overall shapes, then smaller forms, and finally fine details. NCM uses a comparable coarse-to-fine organization for text, without implying that its intermediate codes can be interpreted directly as natural-language plans.

## How the architecture works

The system has two main components:

1. **A multiscale tokenizer** that encodes text into a hierarchy of discrete codes and learns to reconstruct the text.
2. **A transformer generator** that predicts the hierarchy, one level at a time.

The tokenizer is based on a multiscale residual vector-quantized variational autoencoder (VQ-VAE). The first level is short and coarse. Subsequent levels have more positions and add residual information. The final level reaches the full sequence length.

```text
Original text
     ↓
Multiscale tokenizer
     ↓
Level 1:  [ coarse codes ]
Level 2:  [ more detailed codes       ]
Level 3:  [ still finer codes                 ]
Level 4:  [ full-length representation                ]
     ↓
Decoder
     ↓
Reconstructed text
```

Training proceeds in two stages. First, the tokenizer learns to encode and reconstruct text accurately. Then the tokenizer is frozen and the transformer learns to predict the code levels in sequence.

**Within each level, the generator predicts the codes in parallel**, conditioned on the levels already generated. Once the hierarchy is complete, the tokenizer's decoder converts it back into text.

This distinction matters: NCM does not draft a complete paragraph in its first pass. It generates a representation that eventually decodes into the paragraph.

## NCM versus autoregressive and diffusion generation

| Property | Autoregressive | Diffusion language model | NCM |
|---|---|---|---|
| Generation unit | Next token | Masked or incomplete positions | Discrete codes at a hierarchy level |
| Order | Left to right | Iterative refinement | Coarse to fine |
| Parallelism within a step | Limited for one sequence | Multiple positions | Codes within a level |
| Main trade-off | Sequential decoding | Repeated refinement cost | Hierarchy design and reconstruction quality |

Both diffusion and NCM use multiple model evaluations, but those evaluations do different work. Diffusion commonly revisits output positions; NCM predicts a level and advances to the next.

An evaluation count alone does not establish real-world speed. Different passes can have different computational costs, which is why **wall-clock measurements** are important.

## What did the experiments show?

The reported findings are promising, but their scope needs to be stated precisely.

| Evaluation | Reported result | Important qualification |
|---|---|---|
| XSum summarization | Comparable ROUGE-L to Aurora LM, with up to **31×** faster generation | Comparison is against a diffusion baseline in the reported setup |
| LM1B language modeling | Better generative perplexity than the strongest compared diffusion baseline, **21×** faster | Research benchmark, not a general chatbot comparison |
| OpenWebText | Competitive with the strongest compared diffusion baseline, **10×** faster | Depends on the evaluated configuration |
| Sudoku | Outperformed the tested autoregressive and diffusion baselines across evaluated difficulties | Structured benchmark; does not prove general reasoning superiority |
| WMT14 German–English translation | Quality can improve with minimum Bayes risk (MBR) decoding | Generating and comparing multiple candidates adds compute |

For the XSum speed comparison, the source describes generation of **1,024 sequences**, with a **batch size of 16**, on a single **NVIDIA H100 SXM GPU**.

The headline **31×** figure therefore does **not** mean NCM is 31 times faster than ChatGPT, faster on every task, or universally superior to standard autoregressive models.

### Why Sudoku is interesting

Sudoku imposes global constraints: one choice affects what is possible elsewhere in the puzzle. A coarse-to-fine representation offers a different way to construct a globally consistent solution rather than committing to final positions strictly from left to right.

The reported results are noteworthy, but Sudoku is a controlled test. It should not be treated as proof of broad improvements in coding, reasoning, or open-ended assistant tasks.

### Translation and inference-time selection

The paper also examines German-to-English translation. With **minimum Bayes risk decoding**, a system generates multiple candidate translations and selects a candidate using agreement or expected utility across candidates.

This can improve output quality, but the extra candidates and comparisons consume additional compute. Any quality comparison should therefore account for the **total inference budget**, not just a single generation pass.

## What remains unresolved?

Several issues separate a promising research technique from a general-purpose production model.

**Scaling.** The reported scaling experiments cover generators up to **725 million parameters**. That does not establish how the approach behaves at the scale of the largest deployed models.

**Flexible output lengths.** The evaluated systems use defined hierarchies and task-specific configurations. This is different from a general assistant that freely alternates between one-sentence answers and long reports.

**Reconstruction fidelity.** If the learned representation loses information, the decoder may fail to recover exact names, numbers, or other critical details.

**Benchmark dependence.** Relative speed changes with hardware, batch size, sequence length, implementation, and the comparison baseline. A summarization result cannot simply be transferred to code generation or interactive chat.

**Independent validation.** Reproduction on realistic workloads is essential before treating any reported speedup as an engineering guarantee.

## Can developers run NCM?

The source notes that research checkpoints have been published on Hugging Face for tasks including **XSum, WMT14 German-to-English translation, Sudoku, TinyStories, and LM1B**, while OpenWebText was listed as pending at the time described.

That means it would be inaccurate to say that no NCM models are downloadable. However, task-specific research checkpoints are not equivalent to a drop-in, general-purpose chatbot. Running them requires the corresponding project code and configuration.

## Is NCM still a language model?

**Yes.** NCM still uses a transformer to model and generate text. Its novelty lies in the **representation being predicted** and the **order of generation**, not in eliminating language models or transformers.

It challenges the assumption that text must always be produced sequentially as final tokens. It does not establish that autoregressive LLMs are obsolete.

## What should AI engineers take away?

Three lessons stand out:

1. **Read speed claims in context.** The 31× result refers to NCM versus a particular diffusion baseline on XSum, not a universal comparison with commercial assistants.
2. **Representation design changes the computational problem.** Generating a hierarchy of codes can create different opportunities for parallelism and different quality–compute trade-offs.
3. **Evaluate on your own workload.** Compare quality and end-to-end latency on the same hardware and batch sizes, especially for exact details, long outputs, and tasks that matter to your users.

## Final thoughts

Next Concept Modeling is an interesting research direction because it rethinks the **unit and sequence of text generation**. Rather than predicting each final token in order, it builds a multiscale representation and decodes the finished hierarchy into text.

Its reported benchmark gains suggest that coarse-to-fine generation deserves further study. But the evidence is still task- and setup-specific, and important questions remain around scale, flexible generation, reconstruction, and production performance.

The most useful conclusion is not that NCM ends autoregressive language models. It is that **the design space for generating language is larger than token-by-token decoding**.
