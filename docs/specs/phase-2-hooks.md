# Phase 2: Hooks

Status: proposal (needs-triage). Blocked by: Phase 1 (first batch, for the allowlist and rule ids).

## Problem Statement

Stated intentions do not survive a long session. The parts of the doctrine that can be checked mechanically should be enforced by the harness, with the reason attached, so Claude cannot drift past them.

## Solution

Ruby standard-library scripts wired as Claude Code hooks, shipped in the plugin, each with Minitest tests that feed recorded tool-call JSON.

## User Stories

1. As the owner, I want `bundle add devise` refused with the rule and the Rails alternative in the message, so that the doctrine is enforced without my presence.
2. As Claude, I want the versions, test framework, schema tables and resource list injected at session start, so that I never guess them.
3. As Claude, I want RuboCop omakase results after each Ruby or ERB edit, so that style is corrected in the same turn.
4. As Claude, I want the related spec or test to run after an edit under `app/`, so that regressions surface immediately.
5. As the maintainer, I want the hook block rate per session recorded, so that I can see whether the doctrine is teaching or only enforcing.

## Implementation Decisions

- Hooks and events: `versions` (SessionStart, stdout), `gem-allowlist` (PreToolUse on Edit|Write of Gemfile and on Bash matching bundle add or gem install; exit 2 with reason), `forbidden-paths` (PreToolUse on Write|Edit under `app/services/**`, `app/policies/**`, `app/repositories/**`, `spec/factories/**`, `test/factories/**`, `.env*`; exit 2), `frozen-migrations` (PreToolUse on Edit of a migration at or below the schema version; exit 2), `omakase-lint` (PostToolUse on Edit|Write of `*.rb`, `*.erb`; `additionalContext` with remaining offenses after `rubocop -A`), `related-test` (PostToolUse under `app/**`; `additionalContext` with failures), `doctrine-review` (Stop; advisory checklist over the diff with rule ids), `ci-gate` (Stop, opt-in; `bin/ci`).
- The allowlist is generated from the 8.1 Gemfile template plus rspec-rails and records the generating option per gem.
- Scripts read JSON from stdin, use only `json` from the standard library, and are referenced through `${CLAUDE_PLUGIN_ROOT}` in the plugin's `hooks/hooks.json`.
- Each block message names the rule id and the Rails home for that code.
- Block events are appended to `.claude/doctrine-hooks.log` (gitignored) for the block-rate metric.

## Testing Decisions

- Each hook has allow, block and false-positive cases: `bundle install` touching Gemfile.lock is allowed; adding an allowlisted gem is allowed; editing a new, unmigrated migration is allowed.
- Tests run the script as a subprocess with fixture JSON and assert exit code and stderr or stdout.

## Out of Scope

- Hooks that judge design (that is the review skill and the evals).
- Any hook that edits files other than RuboCop autocorrect on the edited file.

## Checklist

- [ ] `hooks/hooks.json` wires all eight hooks with exact matchers.
- [ ] Allowlist generated from the Gemfile template, reviewed by the owner.
- [ ] Each hook has Minitest tests for allow, block and at least one false-positive case.
- [ ] Block messages name a rule id and the Rails alternative.
- [ ] SessionStart injection stays under 60 lines for the lab apps.
- [ ] Block-rate log written and a one-line summary script exists.
- [ ] Hooks verified in a live session against both lab apps.
