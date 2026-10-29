# Execution plan: how each phase is attacked, which model does what, and how review stays honest

Date: 2026-10-29. Companion to `docs/plan.md` and `docs/specs/`. Prices are Anthropic API list prices as cached by the claude-api reference on 2026-10-06, per million tokens, input / output: Fable 5.1 $10 / $50, Opus 5.5 $4 / $20, Sonnet 5.5 $2 / $10, Haiku 5.5 $0.10 / $0.50.

## 1. The bias problem, stated plainly

The same agent wrote the plan, the specs, the checklists and the grill answers, and would by default also write the rules, the eval tasks and the judge prompts. Every one of those steps can quietly accept the previous one. Three failure modes matter:

- **Self-acceptance**: the reviewer shares the author's assumptions, so a wrong rule reads as obvious.
- **Grader fit**: eval tasks and graders are written to match what the author already writes, so the doctrine "improves" what it was already going to do.
- **Judge self-preference**: a model judge prefers output that resembles its own, and defers to labels such as "baseline".

Promises do not fix these. Structure does. The protocol below makes each control a file, a tag or a separate run that the owner can see.

## 2. Review protocol (applies to every phase)

### Roles, never combined in one context

| Role | What it sees | What it may not see | Default model |
| --- | --- | --- | --- |
| Author | Research files, lab, the task | Nothing withheld | Fable 5.1 for rules and specs; Sonnet 5.5 for scripts |
| Verifier | Sources only, plus the artifact's stated goal | The author's draft, reasoning, or chat history | Opus 5.5 (a different model from the author) |
| Adversary | The artifact and the sources | The author's reasoning | Opus 5.5, prompted to find what is wrong or missing, graded by findings the owner upholds |
| Judge | Anonymized diffs or end states, the rule text, the sources | Condition names, run order, who produced what | Opus 5.5 when Sonnet 5.5 is under test; a second judge on a sample for agreement |
| Owner | Findings, disagreements, random samples | Not asked to read everything | Human |

Independence here is partial: all model roles are Claude models and share training. That is why mechanical checks rank first, why generator evidence is the top source tier, and why the owner's labelled sample is the calibration anchor. A judge from another model family is an option for close calls and is noted as such, not required.

### Controls that are files, not intentions

1. **Pre-registration.** The eval task set, mechanical checks, judged items, pass criteria and the held-out split are committed and tagged `evals-v1` before any skill is written. A change after the tag is `evals-v2` and reruns the baseline.
2. **Held-out split.** Thirty percent of tasks, drawn at random stratified by rule area, are run only at the final confirmation. Skills are written against the remaining seventy percent. Train and held-out means must agree within noise at baseline, or the split is redrawn.
3. **Oracle and null runs.** Before the first full pass: an oracle diff built from generator evidence must pass every check, and two nulls (an empty diff, and a deliberately drifted diff that adds Devise, a service object and factories) must fail. A grader that passes a null is fixed before anything is measured.
4. **Known-negative judge tests.** The judge is fed an empty output, "I don't know", and a confident answer to the wrong task, and must fail all three.
5. **Blind, atomic judging.** Diffs are anonymized, order is randomized, no condition is named "baseline" or "reference" in the judge's input, and each judged property is a separate call with structured output and a required quote.
6. **Owner calibration.** The owner labels twenty to thirty judged items independently once; judge agreement is reported, and below roughly ninety percent on clear-cut items the judge prompt is reworked before its scores steer anything.
7. **Failure spot check.** Ten graded failures are read per run; more than one grader error in ten means the grader is fixed before the pass counts.
8. **Mechanism check.** With rules files removed, the score must drop; transcripts must show the rules and skills loading. If the score barely moves either way, the eval is not measuring the lever.
9. **Noise floor on the record.** With 30 tasks and 5 runs the overall pass-rate interval is about ±8 points; per rule area (about 3 tasks × 5 runs) about ±26 points; per rule, directional only. Decisions are made at overall and area level; per-rule numbers point at what to read, not at what to ship.
10. **Spec and rule review by adversary, not by author.** Each spec, rule batch and skill gets an adversary pass whose output is a list of findings with a source quote per finding. The owner reads findings, not drafts.

## 3. Model matrix

The principle: spend on judgment, save on mechanics, and test against the model that will actually be used day to day.

| Work | Model | Why |
| --- | --- | --- |
| Grill, rule drafting, declined list, skill prose | Fable 5.1 | Low volume, highest judgment; the cost is in reading sources, not output |
| Independent re-derivation of rules; adversary reviews | Opus 5.5 | Strong and different from the author |
| Hooks, rule verifier, renderer, eval checker (TDD) | Sonnet 5.5 | Well-specified Ruby scripts with crisp seams |
| Lab generation, excerpt extraction, example regeneration | Sonnet 5.5, Haiku 5.5 for pure extraction | Command running and copying |
| Eval subject during iteration | Sonnet 5.5 | Cheapest realistic subject; a doctrine should help the mid-tier model most, and that is where drift is largest |
| Eval subject at final confirmation | Opus 5.5, plus Sonnet 5.5 | Confirms the effect holds on the model the owner will use daily |
| Judge | Opus 5.5 for Sonnet runs; Fable 5.1 for Opus runs on judged items only | Judge at least one tier above subject where possible; mechanical checks carry most of the grade |

Budget shape for Phase 4, order of magnitude: a Rails eval run is a short Claude Code session, roughly one to three dollars on Sonnet 5.5 and two to six on Opus 5.5 once cache reads are counted. Thirty tasks × five runs × four conditions is six hundred runs, so iteration on Sonnet 5.5 lands in the low thousands of dollars for a full matrix, and the final Opus confirmation on the held-out set at three runs is a few hundred. Reps are the cheap knob: paired designs make them go further than more tasks. A per-phase dollar ceiling is an owner decision and is the first item in the pre-build gate.

## 4. Attack plan per issue

Each phase is split into tracer-bullet tickets with to-tickets when it is permitted. The first ticket of every phase is the one that can prove the phase impossible, so failure is cheap.

**#2 Lab.** Tickets: environment reachability and Ruby install; Minitest app chain; RSpec app chain; setup script, diffs and sources file. Review: determinism, a second regeneration must match the first except timestamps and keys. No judgment involved, so no adversary pass.

**#3 Doctrine.** Tooling first: record schema and lint, renderer, verifier, each test-driven. Then areas in this order: resources and controllers, models, queries, views and Hotwire, declined list. Per rule: author drafts from research; verifier re-derives from sources only; the two statements are diffed; agreements go to the owner as a batch, disagreements as individual items with both texts. Adversary pass per area. Done when lint, verify and the owner's review all pass and every rule has an eval id.

**#4 Hooks.** One ticket per hook, test-driven from recorded tool-call JSON. Then a red-team ticket: an adversary agent is told to add a gem, create a service object and edit a shipped migration by any means, and every bypass it finds becomes a test. Done when the red team reports no bypass within its budget.

**#6 Evals.** Tickets: engine verification (kept workspaces usable, or the fallback runner); dry run of one task end to end; oracle and null runs; task authoring by a session that receives only rule statements and sources, never the rule author's reasoning; held-out split and `evals-v1` tag; checker script; report generator; baseline run; owner calibration set. This phase starts before #5.

**#5 Skills.** One ticket per skill, written with the writing-for-agents reference, examples lifted by script from the lab. Triggering evals per skill. Effect measured on the train split only during writing; the held-out split is touched once, at the end.

**#7 MCP.** One ticket: the gate decision, written from the Phase 4 report with the transcripts that show stale API or verification avoidance, or their absence.

**#8 Maintenance.** One rehearsal bump when the next Rails patch ships.

## 5. The pre-build gate (what happens before any `rails new`)

1. **Owner decisions**: a dollar ceiling for Phase 4 iteration and for the final confirmation; whether a non-Claude judge is wanted for close calls; confirmation of the model matrix.
2. **Adversary pass on the planning itself**: a fresh Opus 5.5 session, given the sources and the branch but none of this session's reasoning, reviews the grill answers, the five ADRs and the seven specs and posts findings on the parent issue. The owner upholds or dismisses each finding. This is the first use of the protocol, applied to the documents that define it.
3. **Pre-registration skeleton**: `evals/PREREGISTRATION.md` with pass criteria, the noise-floor arithmetic, the split rule, the oracle and null definitions and the judge protocol, committed now so that nothing in it can be tuned after results exist.
4. **Permission**: the owner relabels #2 `ready-for-agent`.
