# Phase 3: Skills

Status: proposal (needs-triage). Blocked by: Phase 4 baseline run (ADR-0003) and Phase 1 first batch.

## Problem Statement

Rules say what; skills carry the decision trees, references and generated examples that let Claude apply them in unfamiliar cases, loaded only when a task matches.

## Solution

Twelve skills in the plugin, each a SKILL.md under 100 lines with decision rules pointing at rule ids, references excerpted from Tier 0 to 2 sources for the pinned version, and examples lifted from the lab.

## User Stories

1. As Claude asked to "add an endpoint", I want the resources skill to load and steer me to a new resource, so that custom actions stop appearing.
2. As Claude asked for a live-updating list, I want the Hotwire skill's Frame versus Stream versus morph decision tree, so that I do not reach for fetch and JSON.
3. As Claude in an RSpec project, I want only the RSpec testing skill to load, so that Minitest advice never appears there.
4. As the maintainer, I want skill triggering measured, so that a skill that never fires is fixed by rewording its description.

## Implementation Decisions

- Skills: rails-resources, rails-models, rails-queries, hotwire, rails-testing-rspec, rails-testing-minitest, rails-auth, rails-infra, rails-migrations, rails-version-delta, ruby-34-idioms, rails-doctrine-review.
- Descriptions carry the trigger branches; `paths` frontmatter narrows file-triggered skills (ruby-34-idioms on `**/*.rb`, testing skills on `spec/**` or `test/**`).
- Reference material lives in sibling files linked from SKILL.md; examples are copied from `lab/` by a script so they regenerate on a Rails bump.
- rails-doctrine-review is also what the Stop hook calls.
- Written following the writing-for-agents reference: positive phrasing, one meaning in one place, pointers tuned before material is inlined.

## Testing Decisions

- Triggering: twenty matching prompts and twenty distractors per skill through `claude plugin eval` with `tool_used` graders on the Skill tool.
- Content: every decision rule in a SKILL.md maps to a rule id; checked by `bin/doctrine lint`.
- Effect: the Phase 4 eval set under the doctrine-plus-skills condition.

## Out of Scope

- Skills for gems outside the allowlist.
- Agent definitions (subagents); revisit if a skill needs an isolated context.

## Checklist

- [ ] Twelve SKILL.md files under 100 lines each with trigger-bearing descriptions.
- [ ] References excerpted from pinned sources, each excerpt with its URL or gem path.
- [ ] Examples regenerated from `lab/` by script.
- [ ] Triggering evals pass at or above 0.9 for matches and distractors.
- [ ] Every decision rule maps to a rule id.
- [ ] Phase 4 rerun shows improvement over the doctrine-only condition per rule area.
