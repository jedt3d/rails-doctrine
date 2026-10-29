# Phase 6: Maintenance loop

Status: proposal (needs-triage). Blocked by: Phase 3.

## Problem Statement

Rails changes every few months and the doctrine's truth is version-bound. Without a loop, rules rot silently and the lab stops matching what `rails new` produces.

## Solution

On every Rails minor or patch bump: regenerate the lab, diff generator output, run every verify command, regenerate the version delta, rerun evals, and surface only what changed to the owner.

## User Stories

1. As the owner, I want to review only rules that a bump flagged, so that maintenance costs minutes, not hours.
2. As the maintainer, I want generator diffs per bump, so that new core-team defaults become rule candidates.

## Implementation Decisions

- `bin/doctrine bump <version>` runs the sequence and opens one issue listing flagged rules, generator diffs and eval deltas.
- The version delta is regenerated from the per-framework CHANGELOGs of the clone; curation happens in the review issue.

## Testing Decisions

- A rehearsal bump from 8.1.4 to the next 8.1 patch or to 8.2 when it ships proves the loop.

## Out of Scope

- Automatic rule rewriting; the loop flags, the owner decides.

## Checklist

- [ ] `bin/doctrine bump` implemented and documented.
- [ ] Rehearsal bump executed and its issue reviewed.
- [ ] Owner time per bump recorded.
