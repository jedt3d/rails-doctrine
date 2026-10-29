# Phase 5: MCP server (conditional)

Status: proposal (needs-triage). Blocked by: Phase 4 report. Build only if the gate opens (ADR-0003).

## Problem Statement

If Claude hallucinates or misremembers Rails API, or avoids verifying because `bin/rails` boot time is high, structured tools that wrap Rails's own commands could help. Whether that is the bottleneck is an empirical question the evals answer.

## Solution

If gated in: an MCP server exposing a persistent sandboxed console, source lookup inside the installed gems, changelog search with version attribution, routes and schema reflection, and a single-file test runner. Kept only if a rerun of the eval set improves a rule area.

## User Stories

1. As Claude, I want to check a scope's SQL without paying boot time, so that I verify instead of guessing.
2. As Claude, I want the source and doc comment for a Rails method from the installed version, so that I learn the core team's intent from their own comments.
3. As the maintainer, I want the server kept only on eval evidence, so that it does not become maintenance without benefit.

## Implementation Decisions

- Gate: Phase 4 shows failures attributable to stale or invented API, or transcripts show verification avoided because of boot time.
- Tools: rails_eval, rails_source, rails_changelog, rails_routes, rails_schema, rails_test. All wrap `bin/rails` or read installed gem files.
- The Ruby MCP SDK is the one dependency outside Rails, recorded as an exception in the allowlist for the server's own Gemfile, not the application's.

## Testing Decisions

- Contract tests per tool; latency budget per call; eval rerun as condition five.

## Out of Scope

- Any tool that writes to the application or the database outside the sandboxed console.

## Checklist

- [ ] Gate decision recorded with the eval evidence that opened or closed it.
- [ ] If open: tools implemented with contract tests and latency budgets.
- [ ] Eval rerun under condition five; keep or remove decision recorded.
