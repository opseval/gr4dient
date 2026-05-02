# Phase 2 — Scale and Polish

The first two projects of Phase 2. By the end of this phase you have turned the single-target Phase 1 harnesses into something that looks like a product: a multi-vendor benchmark with statistically honest numbers, and a visual report a security architect can read without opening a JSON file.

## Theme

Phase 1 built three single-target harnesses — three attack dimensions, three judge architectures, all against a single Gemini target. Phase 2 keeps the *patterns* and changes the *posture*: same probe set across many models, real cross-vendor measurement, real statistical reporting, real visualization. The OpenRouter unified API replaces single-vendor SDKs as the primary path so the family of targets can span Google, Anthropic, OpenAI, Meta, Mistral, DeepSeek, Qwen, and others through one client.

| Project | What's new beyond Phase 1 | Status |
|---|---|---|
| `f4mily` | Multi-vendor benchmark across six models from six vendors; OpenRouter as primary path; Wilson 95% confidence intervals on per-target leak rates; first project to actually populate the `target_models` dimension `m3m0ry`'s schema was always designed for. | Written |
| `dashb0rd` | Reads `runs/<run_id>/summary.json` from `f4mily` and renders a visual report — by-target bar chart with CI whiskers, by-family heatmap. Streamlit-based; the first project that produces a *visual* artifact rather than a JSON one. | Forthcoming |

## Build order

Build in this exact order. `dashb0rd` consumes `f4mily`'s output schema directly and assumes you already have one or more `runs/<run_id>/` directories to point it at.

### G2.1. [`f4mily`](f4mily-project-guide.md) — Cross-Family System-Prompt-Extraction Benchmark

**8–10 hours across two or three sessions.** Builds directly on `m3m0ry`.

Fire a curated set of twelve system-prompt-extraction probes — across five technique families (direct elicitation, authority framing, completion baiting, encoding-channel extraction, persona pivot) — at six frontier chat models from six different vendors. Reuse `m3m0ry`'s two-tier graded judge (regex + LLM-as-judge) and four-state outcome (verbatim / paraphrase / clean / error). Aggregate per-target and per-family leakage rates with hand-rolled **Wilson 95% confidence intervals** so the report is honest about small-sample uncertainty. The judge is a model from a *seventh* vendor (Llama 3.3 70B), deliberately outside the target family to avoid self-evaluation bias.

**What you learn:** the provider-abstraction pattern via OpenRouter (one SDK, one base URL, six vendors); cross-vendor refusal-style variance and what it means for measurement; small-sample statistical hygiene via the Wilson score interval (hand-rolled — no `scipy`/`statsmodels` dependency); reading a benchmark report like a security architect (point estimate vs. interval, "safer than what?", what `clean` actually means). Maps to **OWASP LLM07 — System Prompt Leakage**, **MITRE ATLAS AML.T0057 — LLM Data Leakage**, and **NIST AI RMF MEASURE**.

### G2.2. [`dashb0rd`](dashb0rd-project-guide.md) — Benchmark Visualization (forthcoming)

**6–8 hours.** Builds on `f4mily`'s output schema.

Read one or more `runs/<run_id>/summary.json` files and render a visual report — by-target bar chart with CI whiskers, by-family heatmap, drill-through into trial detail. Streamlit-based. Turns the JSON into an artifact a non-engineer can read without writing a query.

**What you learn:** the Streamlit pattern (declarative reactive Python UI in a single file); CI-whisker plotting with `matplotlib`; the discipline of building visualizations that *do not lie* — overlapping CIs that get drawn as obvious gaps, color choices that don't imply false precision. Maps to **NIST AI RMF MEASURE/MANAGE**: a benchmark that nobody can read is a benchmark that doesn't get measured against.

## What you have at the end of this phase

- A multi-vendor benchmark you can point at any new model OpenRouter adds with a one-line config change.
- A visual report someone unfamiliar with the harness can read in 60 seconds.
- An honest statistical posture: every leak-rate gets an interval, every interval gap gets read carefully, no "model X is 14% worse than model Y" claims unsupported by the sample size.
- A `runs/<run_id>/` schema that the Phase 5 capstone (`tr4nsf3r`, the attack-transferability paper) consumes directly without retrofitting.
