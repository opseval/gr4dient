# Phase 3 — Adaptive and Agentic

The three projects of Phase 3. By the end of this phase the LLM has moved to the *attack side* — the harness no longer fires a static probe set but instead runs an attacker LLM with a strategy, an agentic target with tools the attacker can hijack, and a guardrail-classifier probe that looks more like adversarial-ML research than a chat benchmark.

## Theme

Phase 1 and Phase 2 produced exhaustive harnesses: every payload, every target, count the hits, report the rate. That mental model finds the easy class of model failure — the prompts that land cold. The class it misses is the one real attackers actually use: an attacker that probes, watches the refusal style, retreats, reframes, and adapts. Most published jailbreak research since 2023 — PAIR, Tree-of-Attacks, Crescendo — sits in this class, and so does the agentic-side analogue: tool-call hijacking and guardrail-classifier probing. Phase 3 is where you build harnesses for those.

| Project | What's new beyond Phase 2 | Status |
|---|---|---|
| `tap0ut` | First adaptive harness in the series — PAIR-style attacker LLM, per-turn graded judge (1–10 score plus `attack_succeeded`), multi-turn conversation loop with early stopping. The headline artifact is a transcript, not a percentage. | Written |
| `c4lhij4ck` | First agentic-target harness — the target is a five-tool customer-support agent, and the attacker iterates a *malicious support-ticket body* across turns, exercising indirect prompt injection through the tool-result data channel. The judge inspects emitted `tool_calls[]` structurally and semantically. Maps to OWASP Agentic AI T2 (Tool Misuse) and MITRE ATLAS AML.T0051.001 + AML.T0053. | Written |
| `gu4rdpr0be` | Guardrail-classifier probing — adversarial inputs aimed at a Llama-Guard / Azure-Content-Safety-style classifier, not at the assistant. Local Ollama for the guardrail model so the harness runs offline; pairs the cross-vendor measurement discipline of `f4mily` with the adaptive-search discipline of `tap0ut`. Two-mode attacker (FN-search / FP-search) with bidirectional `attack_succeeded`. | Written |

## Build order

Build in this exact order. `tap0ut` introduces the multi-turn attacker pattern that `c4lhij4ck` reuses against an agentic target, and the per-turn graded judge that `gu4rdpr0be` reuses against a guardrail target. The shared infrastructure goes in first.

### G3.1. [`tap0ut`](tap0ut-project-guide.md) — Adaptive Multi-Turn Jailbreak

**8–12 hours across two or three sessions.** Builds directly on `f4mily`.

Build a PAIR-style adaptive multi-turn red-team harness. Three vendor targets fronted by a synthetic SupportBot system prompt with five explicit policies (refund cap, roadmap confidentiality, competitor neutrality, persona integrity, system-prompt confidentiality). One attacker LLM (Llama 3.3 70B) composes the next user message on each turn, reading the conversation history and the previous turn's judge score to adapt. One per-turn judge LLM (Qwen 2.5 72B, deliberately outside the target *and* attacker families) scores every turn on a 1–10 scale. The loop runs until the target taps out (`attack_succeeded=true`, early stop) or `MAX_TURNS=5` is reached.

**What you learn:** the three-role attacker / target / judge pattern that every modern jailbreak paper uses; PAIR's canonical attacker-output schema (`improvement` + `prompt`); per-turn graded judging with a 1–10 score driving adaptation and a boolean driving early-stopping; the qualitative-not-statistical mental shift (the artifact is the per-turn `improvement` reasoning trace, not a per-target percentage); attacker-refusal detection and clean error separation across five distinct error types. Maps to **OWASP LLM01 — Prompt Injection (multi-turn variant)**, **MITRE ATLAS AML.T0054 — LLM Jailbreak** + **AML.T0051 — LLM Prompt Injection**, and **NIST AI RMF MEASURE 2.7**.

### G3.2. [`c4lhij4ck`](c4lhij4ck-project-guide.md) — Tool-Call Hijacking via Indirect Prompt Injection

**8–12 hours across two or three sessions.** Builds directly on `tap0ut`.

Build a tool-call hijacking harness against an agentic target. A synthetic SupportBot+ has access to five tools — `read_support_ticket`, `lookup_order`, `issue_refund`, `send_email`, `escalate_to_human` — with per-tool policies declared in the system prompt (refund cap of $50, send_email only to the verified customer address, escalate_to_human reason codes from a closed list). The user's chat message is benign and constant across every trial; the attack lives entirely inside the support ticket the agent reads. A PAIR-style attacker LLM iterates the *malicious ticket body* across up to five turns, adapting based on the previous turn's emitted tool calls and judge score. A structural-plus-semantic judge inspects each AgentRun.

**What you learn:** the trust-boundary mental model at the heart of agentic-AI security (the data the agent reads is a separate trust boundary from the user channel); how to plumb function-calling into a multi-step agent loop using the OpenAI SDK's `tools=`/`tool_calls`/`role:'tool'` contract (and Gemini's `types.FunctionDeclaration` / `types.Part.from_function_response` equivalents in Appendix A); structural-plus-semantic judging of an AgentRun (a sequence of emitted tool calls plus a final text reply) against a per-objective success signal; the schema-extension discipline that lets one analysis pipeline read both `tap0ut`'s and `c4lhij4ck`'s `runs/<run_id>/trials.jsonl` outputs. Maps to **OWASP LLM01 — Prompt Injection (Indirect Variant)**, **OWASP Agentic AI Threats and Mitigations — T2 Tool Misuse + T6 Intent Breaking & Goal Manipulation**, **MITRE ATLAS AML.T0051.001** (LLM Prompt Injection: Indirect) **+ AML.T0053** (AI Agent Tool Invocation) **+ AML.T0086** (Exfiltration via AI Agent Tool Invocation), and **NIST AI RMF MEASURE 2.7**.

### G3.3. [`gu4rdpr0be`](gu4rdpr0be-project-guide.md) — Guardrail-Classifier Probing

**8–12 hours across two or three sessions.** Builds directly on `tap0ut` and on `f4mily`'s cross-classifier discipline.

Build an adaptive guardrail-classifier probing harness. Two sizes of Meta's Llama Guard 3 (8B and 1B) hosted locally via Ollama serve as the classifiers under test. A PAIR-style adaptive attacker LLM (Llama 3.3 70B) rewrites a hand-labeled seed payload across up to five turns, in one of two modes: **`fn_search`** (rewrite a harmful seed until the classifier marks it safe) or **`fp_search`** (rewrite a benign seed until the classifier marks it unsafe). A ground-truth-preservation judge LLM (Qwen 2.5 72B) confirms each rewrite still encodes the seed's original intent — the line between a real classifier failure and an attacker who walked the payload off the test distribution. The seed bank is 12 hand-labeled payloads (6 harmful in HarmBench shape across S1/S2/S6/S9/S10/S11; 6 benign-but-tricky in topic areas documented to trip safety classifiers).

**What you learn:** classifiers as a *measurable security control* (not just a black-box guard); the bidirectional `attack_succeeded` discipline (FN-search and FP-search in one harness, with an `attacker_mode` field on every trial) and why a single Attacker class with a `mode` parameter is cleaner than two near-duplicate classes; ground-truth-preservation judging with two worked examples (preserves vs flips intent) calibrating the boolean against concrete cases; intra-family size scaling vs cross-vendor framing — what 1B-vs-8B can and cannot answer; cross-classifier correlated-failure analysis (when the primary failed, did the secondary also fail on the same mutation, or independently?); local-model integration via the official `ollama` Python package alongside the OpenRouter cloud SDK. Maps to **OWASP LLM01 — Prompt Injection** (defense-evaluation lens), **MITRE ATLAS AML.T0043 — Craft Adversarial Data** (primary mapping) **+ AML.T0054 — LLM Jailbreak** (secondary, lens shifted to the safety filter), and **NIST AI RMF MEASURE 2.7**.

## What you have at the end of this phase

- An adaptive multi-turn attack harness — the kind of thing the published PAIR / TAP / Crescendo papers describe, in a small, readable, attacker-target-judge form.
- An agentic-target harness — your first project that exercises the LLM's *action-taking* surface, not just its text-generation surface.
- A guardrail-probing harness — your first project that treats a *classifier* as the system under test, with a measurement pattern that combines Phase 2's cross-vendor discipline and `tap0ut`'s adaptive discipline.
- A `runs/<run_id>/` schema that's been **extended** for nested per-turn data without breaking what `dashb0rd` already consumes. `tap0ut` added the `turns[]` array inside each trial (the per-turn `attacker_improvement` / `attacker_prompt` / `target_response` / `judge` records). `c4lhij4ck` extends each turn record further with an `agent_run.tool_calls[]` array — every emitted tool call's name, arguments, and the stub result returned to the agent — plus an `agent_run.final_text` field. `gu4rdpr0be` adds `primary_verdict` and `secondary_verdict` (the latter nullable, for single-classifier mode), an `attacker_mode` field on the trial root (`fn_search` / `fp_search`), and a baseline turn (`turn=0`, `is_baseline=true`) that records the unmodified seed's verdicts before any attacker rewrite. Extending the schema rather than forking it is the discipline that lets the Phase 5 capstone (`tr4nsf3r`) compare across all of Phases 1, 2, and 3 with one analysis.
