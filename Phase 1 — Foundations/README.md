# Phase 1 — Foundations

The first three projects. By the end of this phase you have built three working AI red team harnesses, learned three different judge architectures, and have three GitHub repos to point at.

## Theme

Each project introduces a **new attack dimension** AND a **new judge architecture**. By the end, you've covered three core attack dimensions of LLM red teaming — persona-hijack rule violation, encoding-channel indirect injection, and few-shot / context information extraction — using three core evaluation patterns: binary, payload-goal, and graded with canary detection.

| Attack dimension | Judge architecture | Project |
|---|---|---|
| Persona-hijack (does an alternate persona overwrite the rule?) | Binary LLM-as-judge | `pers0na` |
| Indirect injection via invisible / encoded channels (does hidden content take over?) | Payload-goal LLM-as-judge | `p0is0n` |
| Few-shot / context leakage (how much sensitive context, including canaries, leaks?) | Graded judge with regex + paraphrase detection | `m3m0ry` |

After Phase 1 you understand the conceptual map of LLM red teaming. The remaining phases scale, polish, and specialize.

## Build order

Build in this exact order. Each project assumes patterns from the previous one and the guides explicitly skip setup steps that were covered earlier.

### G1.1. [`pers0na`](pers0na-project-guide.md) — Persona-Hijack Direct Prompt Injection Harness

**~4 hours.** The foundation project.

Build a CLI tool that defines three target chatbots — each with a fictional persona and a rule the persona must enforce — fires a battery of persona-hijack attacks at them (DAN-style, authority-cloaked roleplay, persona-extraction probes), and uses a second LLM call as an automated judge to score whether each attack got the target to violate its rule.

**What you learn:** Python harness structure, Google AI Studio / Gemini API basics, environment-variable hygiene for API keys, `.gitignore` discipline, your first end-to-end Git + GitHub workflow, the LLM-as-judge pattern, and the cross-cutting `runs/<run_id>/trials.jsonl` streaming output schema that every later guide reuses. Maps to **OWASP LLM01 — Prompt Injection (Direct)** and **MITRE ATLAS AML.T0051.000**.

This is the only guide that assumes zero prior LLM-API or Git/GitHub experience. Everything from here on builds on it.

### G1.2. [`p0is0n`](p0is0n-project-guide.md) — Encoding-Channel Indirect Prompt Injection Harness

**6–10 hours across multiple sessions.** Builds directly on `pers0na`.

Where `pers0na` made the user the attacker via direct chat, `p0is0n` makes *content the assistant retrieves* the attacker — and hides the attack in channels a human reviewer can't see at a glance: HTML comments, zero-width Unicode, Unicode tag-chars (the U+E0000 "Tags" block), CSS `display:none`, image alt-text, and base64 blocks the model is told to decode. Tests whether a knowledge-base assistant ingesting "internal wiki" pages follows hidden instructions invisible to the human reading the same page.

**What you learn:** Trust-boundary analysis (user message vs. retrieved data are different tiers of trust), the encoding-channel taxonomy, payload-goal evaluation (does the model do *the attacker's specific hidden instruction*?), and using `beautifulsoup4` to *render* documents the human-friendly way so you can see what the LLM sees and a reviewer doesn't. Maps to **OWASP LLM01 — Prompt Injection (Indirect)**, **MITRE ATLAS AML.T0051.001**, and **NIST AI RMF MEASURE**.

### G1.3. [`m3m0ry`](m3m0ry-project-guide.md) — Few-Shot / Context Leakage Harness

**6–10 hours across multiple sessions.** Builds on `pers0na` and `p0is0n`.

Test whether a customer-support assistant can be coaxed into revealing the few-shot examples and "current ticket history" attached to its context — and *measure how much actually leaked*. Each target's hidden context is seeded with canary tokens (random unique strings) and synthetic PII shapes; a two-tier graded judge scores each trial as **verbatim** (regex detected an exact tracer), **paraphrase** (LLM-as-judge graded a semantic leak), or **clean**.

**What you learn:** Graded evaluation. Binary "did the attack succeed?" isn't enough when partial leaks matter — a verbatim copy of an identifier is qualitatively worse than a vague paraphrase. You'll build a two-tier judge (deterministic regex + LLM-as-judge) and a four-state outcome (verbatim / paraphrase / clean / error) — the same evaluation shape used in data-exfiltration testing, membership-inference research, and copyright/training-data extraction work. Maps to **OWASP LLM02 — Sensitive Information Disclosure**, **MITRE ATLAS AML.T0057 — LLM Data Leakage**, and **NIST AI RMF MEASURE/MANAGE**.

## What you have at the end of this phase

- Three working Python harnesses, each in its own GitHub repo with a clean README.
- Three demonstrated judge architectures.
- Coverage of three core attack dimensions of LLM red teaming.
- Fluency with the Gemini API, Python project structure, environment-variable hygiene, and Git/GitHub workflow.
- A standardized `runs/<run_id>/trials.jsonl` + `summary.json` schema that the Phase 2 polish projects (`f4mily`, `dashb0rd`) consume directly without retrofitting.
