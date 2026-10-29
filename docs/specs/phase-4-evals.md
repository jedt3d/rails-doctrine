# Phase 4: Evaluation harness

Status: proposal (needs-triage). Blocked by: Phase 0, Phase 1 first batch, Phase 2. Runs its baseline before Phase 3 starts.

## Problem Statement

Without measurement, a doctrine can only make Claude more dogmatic without anyone noticing. The harness must show, per rule, whether the doctrine, skills and hooks change what Claude writes under silent prompts, and whether they cause over-correction.

## Solution

`claude plugin eval` cases under `evals/`, three runs per case, a no-plugin baseline arm, a separate judge model for judged items, kept workspaces over which a Ruby script runs mechanical checks, and a per-rule report.

## User Stories

1. As the owner, I want a pass rate per rule id per condition, so that I can see which rules work and which are hopes.
2. As the maintainer, I want the baseline run before skills exist, so that skills are measured against something.
3. As the owner, I want negative tasks reported separately, so that over-correction is visible.
4. As a rule author, I want the list of rules no task exercises, so that untested rules are obvious.

## Implementation Decisions

- Engine: `claude plugin eval` with `--baseline`, `--runs 3`, `--judge-model`, `--keep-temp`, `--json`. Conditions beyond baseline and full plugin are produced by plugin variants (rules only; rules plus skills; rules plus skills plus hooks).
- Mechanical checks run over kept workspaces: Gemfile unchanged, routes shape, specs or tests present in the right directory, RuboCop clean, suite green, no forbidden paths created.
- Judged items use `llm` graders with the rule text and sources in the criteria and a required quote from the diff.
- Task set: twenty-five to thirty silent prompts, two to three per rule area, plus the five negative tasks from the grill (Q18).
- Report: pass rate per rule id per condition, gem additions, hook blocks, questions asked, over-correction rate, untested rules.
- First ticket verifies that kept workspaces support `bin/rails routes` and the suite; fallback is a runner over `claude -p --output-format json` with per-condition settings.

## Testing Decisions

- The checker script has Minitest tests against fixture workspaces (one passing, one failing per check).
- A dry run with one task proves the pipeline end to end before the task set is written.

## Out of Scope

- Measuring speed or cost; only compliance and over-correction.
- Human-only grading (the owner spot-checks ten percent of judged items; that is review, not the metric).

## Checklist

- [ ] Verification ticket: kept workspaces are complete enough for mechanical checks, or the fallback runner is chosen and recorded.
- [ ] Case layout and grader files for one dry-run task committed and passing end to end.
- [ ] Twenty-five to thirty tasks written, each naming rule ids and carrying mechanical checks and judged items.
- [ ] Five negative tasks included and tagged.
- [ ] Baseline run completed and archived before Phase 3 starts.
- [ ] Report generator produces per-rule, per-condition pass rates plus the untested-rules list.
- [ ] Owner spot check of ten percent of judged items recorded.
- [ ] Acceptance met: every rule area above baseline under the full condition; over-correction no worse than baseline.
