# gr4dient

A progressive series of AI security project guides — written to take a Computer Science / Cybersecurity student from "I have an API key" to a portfolio of working AI-red-team and AI-leveraged-offensive-security projects.

Each guide is a self-contained, step-by-step build manual: ~4–12 hours of focused work, every command spelled out, every design decision explained, every output mapped back to a recognized industry framework (OWASP Top 10 for LLMs, OWASP Agentic AI Top 10, MITRE ATLAS, NIST AI RMF).

The series name is a nod to gradient ascent — you climb the gradient one project at a time, each one slightly steeper than the last.

---

## Who this is for

The guides assume:

- **Beginner-to-intermediate Python** — comfortable with functions, dictionaries, and reading a stack trace; not yet fluent with async, packaging, or large refactors.
- **Strong security fundamentals** — Security+ / PenTest+ level. CTF experience helps but isn't required.
- **No prior LLM API experience needed.** The first guide walks through getting an API key, setting up a project, and writing the first call from scratch.
- **macOS** — commands and tooling target macOS (Homebrew, zsh, Terminal). Linux users can mostly follow along; Windows users will need to translate.

If you are further along than that, the guides will still be useful as reference material for the patterns (provider abstraction, LLM-as-judge, graded evaluation, agentic tool-use red teaming) — but you can skim the setup sections.

---

## What this series aims to cover

These projects are sequenced to give you hands-on exposure to two complementary halves of AI security:

1. **Adversarial testing of AI** — building harnesses that probe AI systems for prompt injection, sensitive-information disclosure, agentic-tool hijacking, and similar failure modes. This is the focus of Phases 1–3.
2. **Use of AI in offensive testing** — applying AI as a planner, analyst, and assistant inside traditional security workflows. This is the focus of Phase 4.

The early phases focus on attacking AI; the later phases pivot to using AI as the attacker's tool. By the end, you have a portfolio that touches both halves. What you do with it from there is up to you.

---

## The roadmap at a glance

| Phase | Theme | Projects | Status |
|---|---|---|---|
| **1 — Foundations** | Three core attack dimensions × three judge architectures | `pers0na`, `p0is0n`, `m3m0ry` | All three written |
| **2 — Scale and Polish** | Turn the harnesses into something that looks like a product | `f4mily`, `dashb0rd` | `f4mily` written; `dashb0rd` forthcoming |
| **3 — Adaptive and Agentic** | Difficulty jump — LLMs move to the attack side | `tap0ut`, `c4lhij4ck`, `gu4rdpr0be` | Forthcoming |
| **4 — The Offensive Pivot** | Flip from "red team AI" to "use AI to red team" | `s1ftr`, `expl0rer` | Forthcoming |
| **5 — Capstone** | A research deliverable that ties everything together | `tr4nsf3r` (attack-transferability paper) | Forthcoming |

Each phase folder has its own README explaining the projects in that phase and the order to build them in.

---

## Repository layout

The repo grows phase by phase as guides land. Today:

```
.
├── README.md                              ← you are here
├── LICENSE                                ← CC BY-SA 4.0
├── CLAUDE.md                              ← AI-assisted-editing conventions
├── Phase 1 — Foundations/
│   ├── README.md
│   ├── pers0na-project-guide.md           ← G1.1 (first guide)
│   ├── p0is0n-project-guide.md            ← G1.2 (second guide)
│   └── m3m0ry-project-guide.md            ← G1.3 (third guide; Phase 1 complete)
└── Phase 2 — Scale and Polish/
    ├── README.md
    └── f4mily-project-guide.md            ← G2.1 (cross-family benchmark; Phase 2 in progress)
```

Phase 3 through 5 directories will appear as their first guides are written, in the order shown in the roadmap above.

---

## How to use these guides

**Pick a guide. Open it in your editor. Follow it from top to bottom.** That's it.

The format is consistent across every guide:

1. **Header block** — estimated time, difficulty, language, API used, goal in one sentence.
2. **What you're building** — the concept and the data flow.
3. **Why this matters** — the framework mappings (OWASP, ATLAS, NIST) and the named industry tools you're echoing (PyRIT, garak, promptfoo).
4. **Setup checklist** — every prerequisite, with troubleshooting links. Done before the build session so you don't burn project time on environment problems.
5. **Step-by-step build** — six phases, numbered, with explicit git checkpoints. Run the code at each checkpoint; if it doesn't work, stop and fix before moving on.
6. **Stretch goals** — optional follow-ups when you finish early.
7. **README template for the GitHub repo** — every project ships with its own clean repo. The guide gives you the README to drop in.

### Recommended order

Build in order. Each project assumes patterns learned in the previous one. Skipping ahead works in theory but you will end up backfilling the missing concepts.

If time forces cuts, the working priority is:

- **Must-ship:** `f4mily` (G2.1), `tap0ut` (G3.1), `c4lhij4ck` (G3.2), `s1ftr` (G4.1), `tr4nsf3r` (G5).
- **Cut first:** `gu4rdpr0be` (G3.3), then `dashb0rd` (G2.2).
- **Don't skip `s1ftr` (G4.1).** Without at least one "AI-leveraged offensive security" project, the portfolio only covers half of what the series aims at — adversarial testing of AI, with no demonstration of using AI inside an offensive workflow.

### Time budget

Single-sitting guides target **~4 hours** of hands-on work. Larger projects (`f4mily`, `tap0ut`, `c4lhij4ck`, `s1ftr`, `expl0rer`) are budgeted at **6–12 hours across two or three sessions**. The capstone is a **weekend plus writing time**.

### Tooling and budget

- **Free-tier first.** The default API is **Google AI Studio (Gemini)** — free with no credit card.
- **Optional paid path.** Some later projects benefit from **OpenRouter** with a one-time **$20 credit purchase**, which gets you access to 200+ models through one API. Most runs stay on free-tier or near-free models even when using OpenRouter.
- **Two heavier dependencies show up later.** `gu4rdpr0be` (Phase 3) installs **Ollama** to run a small local guardrail model. `expl0rer` (Phase 4) uses **Docker Desktop** or **Colima** to run a Linux container for binary-exploitation tooling. Both are free, both install via Homebrew. Until you reach those guides, you don't need either.
- **No paid infrastructure.** Everything runs locally in Python. No cloud, no databases.

---

## Security and safety hygiene

These guides teach defensive habits from day one, because building offensive tooling without good habits is how juniors leak credentials and embarrass themselves on GitHub.

- **API keys live in environment variables**, never in code, never in `.env` files committed to git.
- **Every project ships with a `.gitignore`** that excludes secrets, transcripts, and run artifacts.
- **Red team transcripts are sensitive.** They contain attack payloads, system prompts, and sometimes leaked information. Treat them like incident artifacts: review before you share, redact before you publish.
- **All targets are vendored or local synthetic.** Nothing in this series instructs you to attack a real third-party product. The harnesses are for *measuring* attacks against fictitious chatbots and against vulnerable test artifacts you build yourself.
- **Responsible-disclosure framing.** This is offensive tooling built for defensive purposes. The framing of the work — in commit messages, READMEs, and how you talk about it — matters; treating it as research rather than a brag keeps the line clear.

---

## Frameworks the guides map to

Every guide explicitly cites the recognized framework(s) the project addresses, so anyone scanning the repo can match it to their own threat model:

- [**OWASP Top 10 for LLM Applications**](https://genai.owasp.org/llm-top-10/) — LLM01 (Prompt Injection), LLM02 (Sensitive Information Disclosure), and others as later projects come online.
- [**OWASP Agentic AI Top 10**](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/) — central to the Phase 3 agentic projects.
- [**MITRE ATLAS**](https://atlas.mitre.org/) — the adversarial-ML technique catalog. Each project guide cites the specific ATLAS technique IDs it exercises.
- [**MITRE ATT&CK**](https://attack.mitre.org/) — the traditional adversary-tactics catalog. Phase 4's offensive-pivot projects cite ATT&CK rather than ATLAS.
- [**NIST AI RMF**](https://www.nist.gov/itl/ai-risk-management-framework) — particularly the MEASURE function for the comparative-evaluation projects.

---

## Industry tools these projects echo

The guides aren't claiming to compete with mature AI security tooling. They're claiming to *teach the patterns those tools use* in a small, readable form. The tools to know about:

- [**Microsoft PyRIT**](https://github.com/microsoft/PyRIT) — Python Risk Identification Toolkit for generative AI. The reference implementation for orchestrated red teaming.
- [**NVIDIA garak**](https://github.com/NVIDIA/garak) — LLM vulnerability scanner.
- [**promptfoo**](https://www.promptfoo.dev/) — eval and red team harness with strong CI/CD ergonomics.
- [**Llama Guard / OpenAI moderation / Azure Content Safety**](https://genai.owasp.org/llm-top-10/) — the guardrail / classifier layer that Phase 3's `gu4rdpr0be` project tests against.

---

## Contributing / forking

This repo is published under [CC BY-SA 4.0](LICENSE). You are welcome to fork it, lift the guides, or remix them — share-alike means your derivatives stay open under the same terms. The only ask: keep the framework mappings honest. Don't claim a project covers MITRE ATLAS / OWASP LLM Top 10 / OWASP Agentic AI Top 10 / NIST AI RMF techniques it doesn't actually exercise.

The student-repo READMEs the guides instruct you to create are also CC BY-SA 4.0, by convention. If you publish a project from one of these guides, you can keep the share-alike license or relicense to anything compatible with CC BY-SA — your call.

---

## License

[CC BY-SA 4.0](LICENSE).
