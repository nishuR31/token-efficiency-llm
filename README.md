

# token-efficiency-knowledge

[![Topic](https://img.shields.io/badge/topic-token%20efficiency-black)]()
[![Focus](https://img.shields.io/badge/focus-prompt%20optimization-blue)]()
[![Status](https://img.shields.io/badge/status-informational-orange)]()
[![Scope](https://img.shields.io/badge/scope-language%20and%20prompting-purple)]()
[![Use](https://img.shields.io/badge/use-llms%20and%20agents-green)]()

## Overview

This document explains how token usage works in large language models and how to reduce it effectively. It focuses on prompt efficiency, language choice, compression strategies, and realistic trade-offs.

Token optimization is not about minimizing characters blindly. It is about **reducing total cost per correct result**, which includes input tokens, output tokens, retries, and corrections.

---

## Why token efficiency matters

* Token-based billing directly affects cost
* Context windows are limited
* Latency increases with token volume
* Larger prompts reduce space for memory and reasoning

Poor prompt design typically wastes more tokens than language choice ever will.

---

## Core idea

Token efficiency must be evaluated holistically:

* Fewer tokens ≠ better result
* Shorter prompts ≠ clearer instructions
* Language choice ≠ guaranteed savings

The only metric that matters:

> **Cost per successful, correct output**

---

## Language choice and token efficiency

Using languages like Chinese can **sometimes reduce token usage**, but only under specific conditions.

### Why Chinese can be more token-efficient

Chinese is a logographic language. Individual characters often encode more meaning than English subword tokens.

Example conceptually:

* English: multiple tokens per word
* Chinese: fewer tokens per semantic unit

This can lead to:

* shorter prompts for simple instructions
* denser representation of repeated commands

---

### When it actually helps

Language switching can be useful when:

* prompts are short and repetitive
* instructions are simple and structured
* token limits are tight
* the model handles the language well
* the workflow is large-scale (cost sensitivity matters)

In these cases, you might see measurable reductions in input tokens.

---

### Where it breaks

This is where most people mess up.

* Models are often more optimized for English instructions
* Subtle meaning can get lost in translation
* Output formatting may become less consistent
* Mixed-language prompts can confuse behavior
* Debugging becomes harder for teams
* Token savings in input may be offset by longer outputs or retries

So yes, Chinese can reduce tokens. But it can also quietly degrade system reliability.

---

### Practical rule

Use alternative languages only if:

* you have benchmarked token savings
* output quality remains stable
* your team can maintain those prompts
* total cost (not just input tokens) improves

Otherwise, you are trading clarity for marginal gains.

---

## Better token-saving methods than language switching

Most meaningful optimization comes from prompt design.

### 1. Remove filler language

Bad:
“Please carefully analyze the following and provide a detailed response.”

Better:
“Analyze and return structured output.”

---

### 2. Use compact, explicit instructions

Structure:

* task
* constraints
* format

Example:
“Summarize in 5 bullets. Max 10 words each. No intro.”

---

### 3. Force structured output

Prefer:

* JSON
* key-value
* bullet lists

Avoid long-form prose unless necessary.

---

### 4. Control output length

Output tokens are often the biggest cost.

Examples:

* “Return only JSON”
* “Max 3 lines”
* “No explanation”

---

### 5. Avoid repeated context

Major waste source:

* resending same instructions
* repeating system prompts
* duplicating examples

Send only what changes.

---

### 6. Use controlled shorthand

Example:
Instead of repeating long instructions:
“Concise, technical, structured.”

Short, reusable patterns reduce tokens safely if consistent.

---

### 7. Limit examples

Few-shot prompting is expensive. Use only when necessary.

---

### 8. Separate instruction from data

Clean structure improves efficiency and accuracy:

* instruction
* constraints
* input
* output format

---

### 9. Compress behavioral rules

Avoid long descriptions of tone or style. Use minimal directives.

---

### 10. Avoid obvious instructions

Do not waste tokens on:

* “Be clear”
* “Use proper grammar”

Specify only what is critical.

---

## Tricks and their real value

### Multilingual prompting

* Sometimes useful
* Requires testing
* Not universally reliable

### Abbreviations

* Efficient if standardized
* Risky if ambiguous

### Symbol-based prompts

* Works in controlled systems
* Not robust for general usage

### Extreme compression

* Often reduces clarity
* Can increase retries

---

## Efficiency is not just input tokens

True efficiency includes:

* input tokens
* output tokens
* retry frequency
* correction overhead
* formatting accuracy
* task success rate

A slightly longer prompt that works correctly once is cheaper than a shorter one that fails twice.

---

## Decision model

Before applying any optimization:

* Does it reduce tokens?
* Does it preserve correctness?
* Does it reduce retries?
* Is it maintainable?
* Does it work consistently across models?

If only token count improves, the optimization is weak.

---

## Bottom line

* Language choice (like Chinese) can help in some cases
* It is a **secondary optimization**, not a primary one
* Prompt design and output control deliver far larger gains
* Measure everything before adopting any trick

---

## Writing Prompts (Critical)

Do not blindly translate. Maintain **instructional equivalence**.

Guidelines:

* Keep constraints explicit and mirrored
* Prefer **schema-first outputs** (JSON) over prose
* Avoid idioms and culturally loaded phrasing
* Keep system prompts minimal and stable across variants
* Separate **instruction** from **data**

Example (paired):

```txt
# en
Extract fields. Return JSON with keys: name, email. No extra text.

# zh
提取字段。返回JSON，键为：name, email。不要输出多余内容。
```

Bad practice:

* Mixing languages within a single prompt
* Changing output format between variants
* Adding politeness or filler that shifts token counts without adding control

---

## Interpreting Results

What typically holds:

* Chinese variants often reduce **input tokens** for concise instructions
* Output tokens are task-dependent and may not shrink proportionally

Where it breaks:

* Models tuned primarily on English may show:

  * higher variance in outputs
  * schema violations
  * subtle instruction drift

Decision rule:

* Prefer the variant with **lowest cost per successful task**, not lowest tokens

---

## Trade-offs

* Token efficiency: potentially higher (zh), model-dependent
* Instruction fidelity: generally more stable (en)
* Debuggability: higher (en)
* Maintainability: higher (en)
* Latency: correlated with total tokens, but network/model variance dominates

---

## Cost Model

Cost is computed as:

```
total_cost = (input_tokens * price_in) + (output_tokens * price_out)
```

Prices are versioned in `utils/cost.ts`. Update as vendors change rates.

---

## Reproducibility

* Pin model versions where possible
* Record timestamps and model IDs per run
* Run multiple trials; report mean and variance
* Keep prompts versioned; avoid silent edits

---

## Limitations

* Tokenizers differ across models; results are not portable
* Chinese prompts require manual curation for parity
* Quality scoring for generative tasks is approximate
* Pricing and model behavior change over time

---

## Recommended Strategy

Do not adopt language switching as a default optimization.

Prioritize:

* prompt compression (remove redundancy)
* structured outputs (JSON)
* context reuse and caching
* shorter system prompts with stricter constraints

Use cross-language variants only if:

* you have measured consistent gains
* task success rate is unaffected
* your team can maintain dual-language prompts

---

## License

MIT

---

## Contributing

* Add paired prompts under `prompts/en` and `prompts/zh`
* Include a validation strategy for each new task
* Provide before/after metrics in the PR description

---
