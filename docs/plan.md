# Plan: doctrine, hooks, skills, evals, MCP

Revised 2026-10-29 after the grill in `docs/grill/2026-10-29-doctrine-plan.md`.
Per-phase specs with checklists live in `docs/specs/`; those supersede the
phase sections below where they differ. Key revisions: Rails pinned at 8.1.4;
rules delivered as CLAUDE.md under 200 lines plus path-scoped `.claude/rules/`;
positive phrasing with a disclosed declined list; plugin distribution;
`claude plugin eval` as the eval engine with post-hoc mechanical checks;
GitHub API for rails/rails is unavailable from the cloud session, so mining
uses a clone plus web pages.

Goal: make Claude's defaults under silence match Rails intent, for Ruby 3.4
and a pinned Rails 8.x, omakase stack plus rspec-rails, and be able to prove
it rather than feel it.

Two kinds of evaluation run through everything:
- Validity: is a rule true to Rails's design intent? (provenance + checks)
- Efficacy: does the rule plus tooling change Claude's output? (behavioral evals)

---------------------------------------------------------------------------
## Phase 0. Reference lab (prerequisite for everything else)

Deliverables
- `lab/` : two apps generated with the pinned Rails and Ruby 3.4.
  - `lab/minitest_app` : `rails new` defaults.
  - `lab/rspec_app`    : `rails new --skip-test` + rspec-rails + rspec:install.
  - In both: `rails g scaffold Post title body:text`, `rails g authentication`,
    `rails g model Comment`, one Turbo Stream broadcast, one Solid Queue job.
  - Commit the generator output untouched. This is Tier 0 evidence: what the
    core team ships today, not what the internet remembers.
- `lab/SOURCES.md` : local paths to installed gems (`bundle show rails` etc.)
  so skills can quote source comments from the exact version.

Acceptance: `bin/ci` green in both apps; generator diff recorded.

---------------------------------------------------------------------------
## Phase 1. Doctrine (CLAUDE.md + rule records)

Shape
- `CLAUDE.md` : dense. Versions, stack allowlist, project type, then rule
  statements only, one line each, with rule ids. Target under 150 lines
  because it is loaded on every turn.
- `doctrine/rules/<area>.md` : one record per rule. Loaded by skills, not
  always-on.

Rule record format
```
id: R-MODEL-07
statement: Normalize input in the model with `normalizes`, not in controllers or callbacks.
rails_reason: Normalization is a property of the attribute, so it must apply on assignment, on query, and in every entry point (console, jobs, forms). Rails 7.1 added `normalizes` for exactly this.
source_tier: 1
sources:
  - activerecord/lib/active_record/normalization.rb (doc comment)
  - guides: active_record_validations / normalizes section
  - PR rails/rails#43945 (description)
versions: >= 7.1
kind: rails_intent | our_preference
exceptions: cross-attribute cleanup that needs a transaction -> before_validation
verify: grep -q "def normalizes" $(bundle show activerecord)/lib/active_record/normalization.rb
evals: [E-MODEL-03]
```

Source tiers (what counts as "Rails intent")
- Tier 0: Rails source and generator templates in the pinned version.
- Tier 1: Rails Guides, API docs, CHANGELOGs, release notes, Rails Doctrine.
- Tier 2: core team PR descriptions, review comments on merged and rejected
  PRs, rubyonrails.org posts, Rails World / RailsConf keynotes.
- Tier 3: core authors outside the org: dev.37signals.com, world.hey.com/dhh,
  Hotwire handbooks, Campfire/Writebook source.
- Tier 4: no source. Must be labeled `our_preference`, never `rails_intent`.

Rule areas (target 10 to 15 rules each, fewer is fine)
1. Resources and routing (new resource over custom action; nested shallow;
   `resource` singular; no `member do` by reflex).
2. Controllers (thin; before_action for loading; params.expect in 8.0;
   respond with turbo_stream / html / json in that order of preference).
3. Models (concerns as abilities; callbacks for internal consistency only;
   validations; normalizes; enum positional syntax; delegated_type over STI
   for heterogeneous collections; Current attributes; generates_token_for;
   encrypts; has_secure_password).
4. Querying (preload vs includes vs eager_load; strict_loading; find_each;
   insert_all / upsert_all; `with` CTE; load_async; no raw SQL strings
   without a reason; scopes over class methods when chainable).
5. Views and Hotwire (partials with strict locals; helpers over presenters;
   Turbo Frame for islands, Stream for multi-target updates, morph for
   broadcasts; Stimulus controllers small and HTML-driven; importmap pins;
   no fetch-JSON-render-client by default).
6. Forms (form_with model; nested attributes; error rendering convention).
7. Jobs, mail, storage, text, cable (Active Job on Solid Queue; mailers with
   previews; Active Storage variants; Action Text; broadcasts).
8. Auth (generator as base; authorization by scoping through Current.user;
   predicates; before_action guards; no policy layer without doctrine change).
9. Configuration (credentials; config_for; config.x; no dotenv; no ENV reads
   outside config/).
10. Migrations and DB (null: false by default; FK constraints; check
    constraints; never edit a migrated file; data migrations as jobs or
    runner scripts; SQLite acceptable per Rails 8 guidance).
11. Testing, RSpec variant and Minitest variant (see DECISIONS.md).
12. Ruby 3.4 style (blocks over loops; `it`; pattern matching for shape
    dispatch; Data.define for values; keyword args; endless methods for
    one-liners; implicit return; Comparable/Enumerable; frozen literals).
13. Things we do not do, each with Rails's stated reason (service layer,
    repositories, SPA by default, factories, controller specs, Devise,
    Pundit, dotenv, Sidekiq, Redis for cache, webpack).

Process per rule
1. Extract: pull sources (PR bodies via `gh`, CHANGELOG lines, guide
   sections, source comments). Weight core team authorship.
2. Draft the record.
3. Verify (see Evaluation A below).
4. Independent re-derivation: a fresh agent, given only the sources, writes
   its own statement. Diff. Disagreements are investigated, not averaged.
5. Owner review: you read the record and either accept, relabel as
   `our_preference`, or reject. This is also the learning path you asked for.

Acceptance: every rule has a source tier, a version range, a kind, and at
least one eval id. No `rails_intent` rule at Tier 4.

---------------------------------------------------------------------------
## Phase 2. Hooks (deterministic enforcement)

Location: `.claude/hooks/*.rb`, wired in `.claude/settings.json`. Ruby, so
the project's own toolchain runs them. Each hook has its own tests.

| Hook | Event / matcher | Behavior |
|---|---|---|
| versions | SessionStart | Inject `ruby -v`, `rails -v`, table list from schema, resource list from routes, test framework in use. |
| gem-allowlist | PreToolUse: Edit/Write on Gemfile; Bash matching `bundle add`, `gem install` | Block unless gem is in `doctrine/allowlist.txt`. Message names the rule and says "adding a gem is a doctrine change". |
| forbidden-paths | PreToolUse: Write/Edit on `app/services/**`, `spec/factories/**`, `.env*`, `app/policies/**` | Block with rule id and the Rails alternative. |
| frozen-migrations | PreToolUse: Edit on `db/migrate/*` already present in `schema.rb` version | Block. New migration instead. |
| omakase-lint | PostToolUse: Edit/Write on `*.rb`, `*.erb` | Run `rubocop -A --config rubocop-rails-omakase` on the file; return remaining offenses. |
| related-test | PostToolUse: Edit/Write under `app/**` | Map to `spec/**/*_spec.rb` or `test/**/*_test.rb`; run it; return failures. |
| doctrine-review | Stop | Run the `rails-doctrine-review` skill checklist over the diff; report violations with rule ids. Advisory, not blocking. |
| ci-gate | Stop (opt-in per project) | `bin/ci`. Expensive; enable for small apps. |

Hook tests: feed recorded tool-call JSON for allow and block cases; assert
exit code and message. Include false-positive cases (editing Gemfile.lock via
`bundle install`, adding an allowlisted gem).

Metric to watch: block rate per session over time. Falling means the
doctrine is teaching; flat means it is only enforcing.

---------------------------------------------------------------------------
## Phase 3. Skills

Each skill: `SKILL.md` under 100 lines (when it applies, decision rules with
rule ids, anti-patterns with Rails's reason), `references/` (excerpts from
Tier 0 to 2 sources for the pinned version), `examples/` (code from the lab
apps, not hand-written).

| Skill | Triggers on | Core content |
|---|---|---|
| rails-resources | routes, controller, "add an action", "endpoint" | New resource over custom action; shallow nesting; params.expect; response ordering. |
| rails-models | model, concern, callback, validation, enum, token | Concern design; callback boundary; normalizes; delegated_type; Current. |
| rails-queries | query, slow, N+1, bulk, report | Loading strategies; strict_loading; bulk writes; CTE; async; explain. |
| hotwire | page update, live, modal, inline edit, form without reload | Frame vs Stream vs morph decision tree; Stimulus patterns; importmap. |
| rails-testing-rspec | spec, test (in RSpec projects) | Fixtures; request specs; matchers; nesting cap; what not to mock. |
| rails-testing-minitest | test (in Minitest projects) | Declarative tests; fixtures; parallelize; integration over controller. |
| rails-auth | login, session, password, permission, admin | Generator anatomy; scoping authorization; predicates; guards. |
| rails-infra | job, mail, upload, cache, websocket, deploy | Solid trio; Active Job; Storage; Text; credentials; Kamal basics. |
| rails-migrations | migration, column, index, schema | Constraints; irreversible changes; data migrations; SQLite notes. |
| rails-version-delta | upgrade, deprecated, "does Rails have" | Removed and renamed APIs 7.0 -> pinned, generated from CHANGELOGs. |
| ruby-34-idioms | always loaded alongside any Ruby edit (via rule in CLAUDE.md) | Idioms list with before/after pairs. |
| rails-doctrine-review | end of task, "review" | Checklist over diff, outputs rule ids violated; used by Stop hook. |

Skill tests
- Triggering: skill-creator description evals (does the right skill load for
  20 sample prompts, and not load for 20 distractors).
- Content: every decision rule in a SKILL.md maps to a rule id with a source.

---------------------------------------------------------------------------
## Phase 4. Evaluation harness (built before any MCP)

Layout
```
evals/
  tasks/E-RES-01.yaml ...      # one task per file
  run.rb                       # copies lab app, runs claude -p under a condition, grades
  judge/PROMPT.md              # rubric prompt for LLM judge, with rule text + sources
  results/<date>/<condition>.json
```

Task file
```
id: E-RES-01
prompt: "Users need to be able to publish and unpublish posts."   # silent on stack
category: resources
rules: [R-RES-01, R-MODEL-03, R-TEST-02]
mechanical:
  - name: no_new_gems
    cmd: git diff --quiet HEAD -- Gemfile
  - name: new_resource_not_action
    cmd: bin/rails routes | grep -qE "posts/:post_id/publication" && ! bin/rails routes | grep -q "posts/:id/publish"
  - name: tests_added
    cmd: git diff --name-only HEAD | grep -qE "^spec/(requests|models)/"
  - name: lint
    cmd: rubocop --config .rubocop.yml $(git diff --name-only HEAD | grep '\.rb$')
  - name: suite
    cmd: bin/rails test || bundle exec rspec
judged:
  - "Publication state lives in the model, not the controller"
  - "Turbo Stream or redirect used, not JSON + client code"
negative: false
```

Task set (25 to 30)
- Each rule area gets 2 to 3 tasks. Prompts are deliberately silent on
  stack choices, because silence is where drift happens.
- At least 5 negative tasks where Rails intent is the opposite of the
  reflex, to catch over-correction: a case where a PORO is right, a case
  where a JS component inside a frame is right, a case where a raw SQL
  query is right, a case where a callback is wrong, a case where asking a
  question is the right move.

Conditions, each task run 5 times per condition (outputs vary)
1. baseline: no doctrine, no skills, no hooks
2. doctrine only
3. doctrine + skills
4. doctrine + skills + hooks
5. (later) + MCP

Graders, in order of trust
1. Mechanical checks (shell commands, exit codes).
2. LLM judge: a separate model run, given the rule text and sources, asked
   per judged item for pass/fail with a quote from the diff as evidence.
   Never the session that wrote the code.
3. Owner spot check: 10 percent sample per run, focused on judged items.

Reports
- Pass rate per rule id (not per task), per condition.
- Gem-addition count, hook-block count, question-asked count.
- Over-correction rate on negative tasks.
- Rules with no eval exercising them (untested rules list).

Acceptance to move on: doctrine+skills+hooks beats baseline on every rule
area, and over-correction on negative tasks is not worse than baseline.

---------------------------------------------------------------------------
## Phase 5. MCP server (conditional)

Build only if Phase 4 shows either (a) failures from hallucinated or stale
Rails API, or (b) Claude avoiding verification because `bin/rails` boot time
is high.

Tools, all wrapping Rails's own commands
- rails_eval: persistent sandboxed console (one booted process per project);
  returns value, exceptions, generated SQL.
- rails_source: constant or method -> source_location -> file excerpt with
  doc comment from the installed gem.
- rails_changelog: keyword search across framework CHANGELOGs and release
  notes, with version attribution.
- rails_routes, rails_schema: structured reflection.
- rails_test: run one file or example, return structured failures.

Evaluation: rerun Phase 4 as condition 5; keep the server only if a rule
area improves. Contract tests per tool; latency budget per call.

---------------------------------------------------------------------------
## Phase 6. Maintenance loop

On every Rails minor or patch bump:
1. Regenerate lab apps; diff generator output; update examples.
2. Run every rule's `verify:` command; flag failures.
3. Regenerate rails-version-delta from CHANGELOGs.
4. Rerun evals; compare with last results.
5. Owner reviews flagged rules only.

---------------------------------------------------------------------------
## Evaluation A: is a rule true to Rails intent?

1. Provenance gate. No rule is `rails_intent` without a Tier 0 to 2 source.
   Tier 3 may support, not establish. Tier 4 is `our_preference`.
2. Executable verification against Tier 0 where possible (`verify:` field).
   Catches version drift mechanically.
3. Counter-evidence search. For each rule, look for a core-team statement
   that contradicts it. Record both sides and the context in the rule.
   A rule with unresolved contradiction is downgraded to `contested`.
4. Negative space. Maintain `doctrine/declined.md`: features the core team
   rejected (PRs closed by core with reasons). A rule that matches a
   declined feature's reasoning is strong; a rule that proposes a declined
   feature is wrong.
5. Independent re-derivation. A fresh agent, with sources only and no draft,
   writes the rule. Agreement is weak evidence of truth; disagreement is
   strong evidence that something needs checking.
6. Practice check. Compare the rule against generator output and against
   Campfire/Writebook. Intent that the authors do not practice is suspect.
7. Falsifiability. A rule must say what would violate it. "Prefer X" with
   no violation condition is rewritten or removed.
8. Owner review of every `rails_intent` rule, with its sources open.

Known limits: much early intent (MVC, Active Record) is in books and
interviews, not repositories; those rules will cite Tier 1 and Tier 3 and
say so. And the drafting model shares the biases it is trying to correct,
which is why steps 2, 4 and 5 exist.

## Evaluation B: do the rules and tools work?

Phase 4, in short: silent prompts, mechanical graders first, separate judge,
negative tasks for over-correction, pass rate per rule, ablation by
condition, repeated runs. A rule with no eval is a hope, not a rule.

## Evaluation C: is the tooling itself correct?

Hook unit tests with allow/block/false-positive cases. Skill triggering
evals. MCP contract tests. Hook block-rate trend as a learning signal.

---------------------------------------------------------------------------
## Sequence and owner time

| Step | Output | Owner time |
|---|---|---|
| 0 Lab | two apps, generator diffs | 0.5 h: confirm Rails minor |
| 1 Doctrine, areas 1 to 5 and 13 first | ~60 rule records, CLAUDE.md | 2 h review per batch of 30 |
| 2 Hooks | 6 hooks + tests | 0.5 h: approve allowlist |
| 4 Evals (before 3, so skills are measured) | harness + 25 tasks + baseline run | 1 h: review 10 tasks and negatives |
| 3 Skills | 12 skills | 1 h: read rails-resources and hotwire |
| 4 again | conditions 2 to 4 | 0.5 h: read report |
| 1 Doctrine, remaining areas | | 2 h |
| 5 MCP | only if gated in | |
| 6 Loop | on each bump | 0.5 h |

Note the order: the eval baseline runs before skills exist, so the skills
are measured against something.
