# Phase 5 — Capstone

The single project of Phase 5. Phases 1-3 built harnesses that probed AI systems for failure modes. Phase 4 flipped the operator-AI surface, putting the LLM in the analyst chair (`s1ftr`) and then the action-taking-operator chair (`expl0rer`). Phase 5 closes the series with the *retrospective analysis* of everything that came before: a small reproduction-scale paper that reads the trial JSONL produced by the prior projects and writes up cross-model attack-transferability findings.

## Theme

The capstone teaches *measurement-as-deliverable*. The first ten projects in the series produced data — `runs/<run_id>/trials.jsonl` for each — by running attacks against models, analysts against fragments, operators against binaries. `tr4nsf3r` is what you do with that data when the building stops: load it, compute statistics that respect the small-sample reality, draw figures that don't hide the uncertainty, and write a measured paper-style writeup that says only what the data supports.

The framework citations don't pivot again. NIST AI RMF MEASURE 2.5 and 2.9 are the same subcategories `s1ftr` and `expl0rer` cited; this project is the *documented-limitations* and *output-interpreted-in-context* layer of those measurements, written as a paper rather than as JSONL.

| Project | What's new beyond Phase 4 | Status |
|---|---|---|
| `tr4nsf3r` | The deliverable is a *paper*, not a harness. Reads JSONL across all three task_types from Phases 1-4 (`adversarial_probe`, `offensive_analysis`, `offensive_exploitation`) via a normalized loader. Computes per-(unit, model) bypass-rate Wilson 95% CIs (statsmodels) plus a per-attack-family transferability index with a bootstrap 95% CI. Generates a heatmap and a forest plot. Writes the paper itself in `paper/paper.md`. No LLM API calls at runtime — pandas + statsmodels + matplotlib. The headline is the writeup. | Written |

## Build order

Phase 5 has only one project. Build it after `expl0rer` (Phase 4 G4.2). Bring whatever real run output you've accumulated from prior projects, or use the synthetic corpus the guide ships with (57 trials, hand-authored, with planted differential transfer patterns).

### G5. [`tr4nsf3r`](tr4nsf3r-project-guide.md) — Cross-Model Attack Transferability (Capstone Paper)

**6-10 hours across two sessions.** Builds directly on the trial JSONL that `f4mily`, `tap0ut`, `c4lhij4ck`, `s1ftr`, and `expl0rer` produced.

Build a small analysis pipeline that reads JSONL trial records across runs, dispatches on `task_type`, computes Wilson 95% CIs on per-(attack-family, target-model) cell bypass rates, and bootstraps a 95% CI on a per-attack-family "transferability index" (the ratio of the worst per-model point estimate to the best). Generate a heatmap and a forest plot. Write a 600-900-word paper-style writeup with abstract, introduction, methods, results, discussion, and references.

The synthetic corpus that ships with the guide is hand-authored (57 trials across three task_types with planted differential transfer patterns) so the analysis pipeline produces a clean readable headline finding regardless of which prior projects the student finished. A real-data swap-in is a single CLI flag — point `--data-root` at any number of `runs/` directories from prior projects and the same pipeline runs against them.

**What you learn:** the discipline of *retrospective* measurement work — reading what your harnesses produced rather than building more of them; small-sample statistical hygiene (Wilson CI, bootstrap, why those choices are right at n=5); cross-task-type schema dispatch via the `task_type` discriminator the prior phases established; per-cell figure design that doesn't hide uncertainty (heatmap cells annotated with both point estimate AND 95% Wilson CI bounds; forest plot with horizontal CI bars); honest writeup discipline ("consistent with" beats "demonstrates" at small n; documenting limitations is the strongest claim available); paper-shape compression (600-900 words plus two figures, not 5000 words). Maps to **NIST AI RMF MEASURE 2.5 — Validity and Reliability** and **MEASURE 2.9 — Explained, Validated, Documented; Output Interpreted in Context** (the *documented limitations* and *output interpreted in context* layers), references **OWASP LLM Top 10 LLM01** (the headline analysis contributes empirical data), and cites the academic literature on transferability (Goodfellow et al. 2014, Wei et al. 2023, Zou et al. 2023).

## What you have at the end of this phase

- A reproducible measurement-and-writeup pipeline (`pandas` + `statsmodels` + `matplotlib`) that runs against any `runs/*.jsonl` produced by the prior projects, or against the synthetic corpus.
- A `paper/results.json` with every per-cell number the paper cites by reference.
- Two PNG figures (`paper/figures/heatmap.png`, `paper/figures/forest.png`) at 200 DPI.
- A `paper/paper.md` writeup of cross-model attack-transferability findings, structured as title / authors / abstract / introduction / methods / results / discussion / references.
- The completed `gr4dient` portfolio. Eleven projects across five phases, from "I have an API key" to "I wrote a small reproduction-scale paper about my own measurements." The natural follow-on is to scale n, bring real attack-family diversity, and submit the writeup to a workshop / blog post / internal write-up.

The series is done.
