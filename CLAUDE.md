# rails-doctrine

A distribution that makes Claude Code's Rails defaults match the Rails core team's intent, and measures it. Read `GLOSSARY.md` and use its terms. Decisions live in `docs/adr/`; the plan in `docs/plan.md`; what has been decided in `docs/decisions.md`.

Implementation of any phase waits for the owner's permission: a phase issue is implementable only when labeled `ready-for-agent` and its blockers are closed (see `docs/agents/issue-tracker.md`).

Prose in this repo uses no em-dashes.

## Agent skills

### Issue tracker

GitHub issues on `jedt3d/rails-doctrine`, REST API via `gh api`. See `docs/agents/issue-tracker.md`.

### Triage labels

Canonical names, unchanged; `ready-for-agent` also means "owner permitted implementation". See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `GLOSSARY.md` at the root, ADRs in `docs/adr/`. See `docs/agents/domain.md`.

### Which skills we use

See `docs/agents/skills.md`.
