
# token-efficiency-llm

[![Node.js](https://img.shields.io/badge/node-%3E%3D18-green)](https://nodejs.org)
[![TypeScript](https://img.shields.io/badge/language-typescript-blue)](https://www.typescriptlang.org/)
[![License](https://img.shields.io/badge/license-MIT-black)](LICENSE)
[![Status](https://img.shields.io/badge/status-experimental-orange)]()
[![LLM](https://img.shields.io/badge/llm-OpenAI%20%7C%20Anthropic-purple)]()

---

## Overview

This repository evaluates whether **language choice (e.g., Chinese vs English)** materially reduces **token usage** in LLM interactions, and whether any savings persist **without degrading task performance**.

It provides:

* deterministic token counting
* cross-language prompt variants
* cost estimation
* task-level benchmarking (quality + latency)

Target models include offerings from OpenAI and Anthropic.

---

## Problem Statement

LLMs are constrained by:

* token-based pricing
* context window limits
* tokenizer-specific segmentation

Hypothesis:

* logographic languages (e.g., Chinese) may encode semantics with fewer tokens than English

Counter-risk:

* instruction fidelity and output quality may degrade
* model-specific tokenizers behave differently

---

## What This Repo Does

* Compares **token counts** for semantically equivalent prompts across languages
* Benchmarks **task outcomes** (summarization, extraction, transformation)
* Estimates **cost deltas** under real pricing
* Surfaces **failure modes** (instruction drift, hallucination, formatting errors)

---

## Non-Goals

* Not a translation library
* Not a production prompt framework
* Not claiming language switching as a primary optimization strategy

---

## Methodology

1. **Prompt Parity**

   * Create equivalent prompts in `en` and `zh`
   * Enforce structural symmetry (same constraints, output schema)

2. **Token Measurement**

   * Use model-aligned tokenizers (e.g., `tiktoken` for OpenAI)
   * Count input, output, and total tokens

3. **Task Evaluation**

   * Deterministic checks where possible (schema validation, regex)
   * Heuristic scoring for free-form tasks (ROUGE-like, length bounds)

4. **Cost Modeling**

   * Map tokens → USD using current pricing tables
   * Report per-call and aggregate costs

5. **Repeatability**

   * Fixed seeds where supported
   * Multiple runs to reduce variance

---

## Repository Structure

```bash
.
├── prompts/
│   ├── en/                 # English prompts
│   └── zh/                 # Chinese prompts (manual, not auto-translated)
├── tasks/
│   ├── summarize.ts
│   ├── extract.ts
│   └── transform.ts
├── scripts/
│   ├── compareTokens.ts    # raw token counts
│   ├── runBench.ts         # executes tasks across models/languages
│   └── report.ts           # aggregates metrics
├── utils/
│   ├── tokenizer.ts        # model-specific tokenization
│   ├── client.ts           # OpenAI/Anthropic wrappers
│   ├── cost.ts             # pricing calculations
│   └── eval.ts             # validators and scorers
├── results/
│   ├── runs.json
│   └── summary.csv
├── .env.example
└── README.md
```

---

## Installation

```bash
git clone https://github.com/your-username/token-efficiency-llm.git
cd token-efficiency-llm
npm install
```

---

## Configuration

Create `.env`:

```bash
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
DEFAULT_MODEL_OPENAI=gpt-4.1-mini
DEFAULT_MODEL_ANTHROPIC=claude-3-5-sonnet
```

---

## Usage

### 1) Token comparison (no API calls)

```bash
npm run compare
```

Outputs per-prompt token counts:

```json
{
  "prompt": "summarize/basic",
  "en": { "input_tokens": 42 },
  "zh": { "input_tokens": 28 },
  "delta_pct": -33.3
}
```

---

### 2) Benchmark tasks (API calls)

```bash
npm run bench
```

Produces `results/runs.json` with:

* input/output tokens
* latency
* cost
* validation pass/fail
* normalized score (if applicable)

---

### 3) Aggregate report

```bash
npm run report
```

Outputs `results/summary.csv`:

| task | model | lang | in_tok | out_tok | total | cost_usd | latency_ms | pass_rate |
| ---- | ----- | ---- | ------ | ------- | ----- | -------- | ---------- | --------- |

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
