# Phase 2 — Scale and Polish

The two projects of Phase 2. By the end of this phase you have turned the single-target Phase 1 harnesses into something that looks like a product: a multi-vendor benchmark with statistically honest numbers, and a visual report a security architect can read without opening a JSON file.

## Theme

Phase 1 built three single-target harnesses — three attack dimensions, three judge architectures, all against a single Gemini target. Phase 2 keeps the *patterns* and changes the *posture*: same probe set across many models, real cross-vendor measurement, real statistical reporting, real visualization. The OpenRouter unified API replaces single-vendor SDKs as the primary path so the family of targets can span Google, Anthropic, OpenAI, Meta, Mistral, DeepSeek, Qwen, and others through one client.

| Project | What's new beyond Phase 1 | Status |
|---|---|---|
| `f4mily` | Multi-vendor benchmark across six models from six vendors; OpenRouter as primary path; Wilson 95% confidence intervals on per-target leak rates; first project to actually populate the `target_models` dimension `m3m0ry`'s schema was always designed for. | Written |
| `dashb0rd` | Reads `runs/<run_id>/summary.json` + `trials.jsonl` from `f4mily` and renders a visual report — by-target bar chart with Wilson CI whiskers, family×target heatmap, per-trial drill-through. Streamlit-based; the first project in the series that produces a *visual* artifact rather than a JSON one. | Written |

## Build order

Build in this exact order. `dashb0rd` consumes `f4mily`'s output schema directly and assumes you already have one or more `runs/<run_id>/` directories to point it at.

### G2.1. [`f4mily`](f4mily-project-guide.md) — Cross-Family System-Prompt-Extraction Benchmark

**8–10 hours across two or three sessions.** Builds directly on `m3m0ry`.

Fire a curated set of twelve system-prompt-extraction probes — across five technique families (direct elicitation, authority framing, completion baiting, encoding-channel extraction, persona pivot) — at six frontier chat models from six different vendors. Reuse `m3m0ry`'s two-tier graded judge (regex + LLM-as-judge) and four-state outcome (verbatim / paraphrase / clean / error). Aggregate per-target and per-family leakage rates with hand-rolled **Wilson 95% confidence intervals** so the report is honest about small-sample uncertainty. The judge is a model from a *seventh* vendor (Llama 3.3 70B), deliberately outside the target family to avoid self-evaluation bias.

**What you learn:** the provider-abstraction pattern via OpenRouter (one SDK, one base URL, six vendors); cross-vendor refusal-style variance and what it means for measurement; small-sample statistical hygiene via the Wilson score interval (hand-rolled — no `scipy`/`statsmodels` dependency); reading a benchmark report like a security architect (point estimate vs. interval, "safer than what?", what `clean` actually means). Maps to **OWASP LLM07 — System Prompt Leakage**, **MITRE ATLAS AML.T0057 — LLM Data Leakage**, and **NIST AI RMF MEASURE**.

### G2.2. [`dashb0rd`](dashb0rd-project-guide.md) — Benchmark Visualization

**6–8 hours across two sessions.** Builds on `f4mily`'s output schema.

Read `runs/<run_id>/summary.json` + `trials.jsonl` from a `f4mily` run and render a visual report — by-target bar chart with Wilson 95% CI whiskers, family×target leak-count heatmap with annotated cells, and a per-trial drill-through with multiselect filters and expandable cards showing each probe payload, target response, and judge reasoning. Streamlit-based, single-file app. The fixture-fallback path lets the dashboard render against hand-written `fixtures/summary.json` + `fixtures/trials.jsonl` so students can build the dashboard before they have a real `f4mily` run on disk.

**What you learn:** the Streamlit pattern (declarative reactive Python UI in a single file); matplotlib's `errorbar` for asymmetric Wilson CIs and `imshow` for annotated heatmaps; the cache-key discipline of `@st.cache_data` keyed on `(path, mtime)` so re-runs of the upstream harness invalidate cache automatically; the discipline of building visualizations that *do not lie* — overlapping CIs that get drawn as obvious overlaps, single-hue sequential color (no diverging red/green that implies "good model" vs "bad model"), explicit `—` for missing cells instead of silent zero, and `n=N` sample-size labels on every chart so readers don't have to dig through a methods section to understand what the bar means. Maps to **NIST AI RMF MEASURE/MANAGE** (visualization is part of measurement reporting; a benchmark that nobody can read is a benchmark that doesn't get measured against). Appendix A extends the dashboard with a multi-run comparison tab — direct setup for the Phase 5 capstone (`tr4nsf3r`, attack-transferability across runs).

## What you have at the end of this phase

- A multi-vendor benchmark you can point at any new model OpenRouter adds with a one-line config change.
- A visual report someone unfamiliar with the harness can read in 60 seconds.
- An honest statistical posture: every leak-rate gets an interval, every interval gap gets read carefully, no "model X is 14% worse than model Y" claims unsupported by the sample size.
- A `runs/<run_id>/` schema that the Phase 5 capstone (`tr4nsf3r`, the attack-transferability paper) consumes directly without retrofitting.
