# Phase 1: Doctrine

Status: proposal (needs-triage). Blocked by: Phase 0.

## Problem Statement

Claude's Rails defaults under a silent prompt regress to community-popular choices. Correcting that needs rules with reasons, verified against the pinned Rails, in a form Claude reads at the right moment and a script can check.

## Solution

Rule records in YAML, one file per area, rendered into a consumer CLAUDE.md under 200 lines, path-scoped `.claude/rules/*.md` files, human pages and a verification run. Every rule carries a source tier, a version range, a kind (rails_intent, owner_preference, contested), exceptions and a verify command. A declined list and a research corpus back the rules.

## User Stories

1. As Claude editing a model, I want the model rules loaded only then, so that always-on context stays small.
2. As the owner, I want every rule that claims Rails intent to cite a Tier 0 to 2 source, so that I can check it and learn from it.
3. As the maintainer, I want `bin/doctrine verify` to fail when a rule's API no longer exists in the pinned Rails, so that version drift is mechanical.
4. As a reviewer, I want contested rules marked, so that disagreements inside the core team are visible rather than averaged.
5. As a future reader, I want the declined list to say why Rails declined each thing, so that nobody reintroduces it by accident.

## Implementation Decisions

- Record fields: id, statement, rails_reason, source_tier, sources, versions, kind, exceptions, postgres_note, verify, evals.
- Rule areas and their `paths` triggers: resources-and-controllers (`config/routes.rb`, `app/controllers/**`), models (`app/models/**`), queries (`app/models/**`, `app/jobs/**`), views-and-hotwire (`app/views/**`, `app/javascript/**`, `app/helpers/**`), forms (`app/views/**`), infra (`app/jobs/**`, `app/mailers/**`, `config/**`), auth (`app/controllers/**`, `app/models/**`), configuration (`config/**`), migrations (`db/**`), testing-rspec (`spec/**`), testing-minitest (`test/**`), ruby-style (`**/*.rb`). Cross-cutting rules (resource-first, omakase, testing choice, pinned versions) live in CLAUDE.md.
- Rules are phrased positively; the declined list is disclosed reference reached by one pointer in CLAUDE.md.
- Process per rule: extract sources, draft, verify, independent re-derivation by a fresh agent from sources only, owner review.
- Source mining uses a bounded clone of rails/rails, the guides, rubyonrails.org posts and PR pages fetched over HTTPS; the GitHub API is not available for rails/rails from the cloud session.
- First batch: resources-and-controllers, models, queries, views-and-hotwire, declined list. Second batch: the rest.

## Testing Decisions

- `bin/doctrine verify` runs every `verify` command against the lab's installed gems and reports pass, fail and unverifiable per rule.
- `bin/doctrine lint` rejects a record missing any required field, a rails_intent rule at Tier 3 or 4, a rule without an eval id, or a statement phrased as a prohibition.
- Both scripts have Minitest tests with fixture records.

## Out of Scope

- Skill bodies (Phase 3), hooks (Phase 2), eval tasks (Phase 4).
- Rules for Rails features outside 7.0 to 8.1.

## Checklist

- [ ] Rule record schema written and enforced by `bin/doctrine lint`.
- [ ] Renderer produces CLAUDE.md (under 200 lines), `.claude/rules/*.md` with `paths`, and `docs/doctrine/*.md`.
- [ ] `bin/doctrine verify` runs verify commands against `lab/SOURCES.md` paths.
- [ ] `doctrine/declined.md` lists each declined feature with the core team's stated reason and source.
- [ ] `docs/research/` holds the cited research files for JavaScript decisions and MVC and Active Record decisions, plus one per remaining area as it is mined.
- [ ] First batch (four areas plus declined) drafted, verified, re-derived independently, owner-reviewed.
- [ ] Every rails_intent rule cites a Tier 0 to 2 source; Tier 4 rules are labeled owner_preference.
- [ ] Every rule has at least one eval id (placeholder ids allowed until Phase 4 creates the tasks).
- [ ] Second batch completed after the first eval run.
- [ ] Owner review time recorded per batch.
