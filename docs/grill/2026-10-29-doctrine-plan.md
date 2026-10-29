# Grill: the doctrine plan

Date: 2026-10-29. Skill: `/grill-with-docs` (grilling + domain-modeling).
Deviation: at the owner's request the agent answered its own questions with researched sources, as a test of its Rails and Ruby understanding. Every ➡️ is the agent's recommendation. Items the owner must decide are marked **OWNER DECISION**. Facts carry a source; where a fact came from a research file in `docs/research/`, the file is named.

Design tree root: **a distribution that, installed into a Rails 8.1 application, makes Claude Code's defaults match Rails intent, with evidence.**

---

## Round 1: scope and pins

❓ **Q1** - **Destination**: What does "done" look like for v1? Options: (a) a doctrine file and hooks in one Rails app; (b) a reusable distribution installable into any Rails 8.1 app, with an eval report showing improvement over baseline; (c) also an MCP server.

➡️ (b). (c) is gated by evals (ADR-0003). (a) is not reusable and cannot be measured against a second project.

**Answer**: (b). Done means: `bin/doctrine install` into a fresh `rails new` app yields CLAUDE.md, rules files, hooks and skills; `bin/doctrine verify` passes every rule's check against the pinned Rails; the eval report shows doctrine plus skills plus hooks beating baseline on every rule area with no worse over-correction on negative tasks.

---

❓ **Q2** - **Rails pin**: Pin Rails 8.1.4?

➡️ Yes.

**Answer**: 8.1.4 is the latest stable release (announced 2026-09-24, with 7.2.4 as the last 7.2 release). Rails 8.2 has not shipped; its release notes exist only as a draft on `main` (notable: `Rails.app`, `Rails.app.creds`, `:header_only` forgery protection default). Rails 8.1 requires Ruby ≥ 3.2.0 (`rails.gemspec`, `8-1-stable`). Pin `~> 8.1.4` in Gemfiles and record 8.1 as the pinned minor in `docs/decisions.md`. The version delta skill covers 7.0 → 8.1.
Sources: rubyonrails.org release list; `raw.githubusercontent.com/rails/rails/8-1-stable/rails.gemspec`; msp-greg mirror of `8_2_release_notes` (draft).

---

❓ **Q3** - **Ruby pin**: Pin Ruby 3.4 at the latest patch (3.4.12 today), accepting 3.4.10 where that is what the environment can build?

➡️ Yes. `.ruby-version` = `3.4.12`; the lab setup falls back to the newest 3.4.x the installer offers and records which.

**Answer**: ruby-lang.org lists 3.4.12 as the current 3.4 patch and 4.0.9 as current stable; this cloud container's rbenv offers 3.4.10 and 4.0.6, and Docker is unavailable. Ruby 4.0 stays out (ADR-0005). The language features the doctrine names from 3.4: `it`, Prism default parser, chilled-string warnings.
Sources: ruby-lang.org/en/downloads; `rbenv install --list` in this container.

---

❓ **Q4** - **Database for the lab**: SQLite, as `rails new` defaults?

➡️ Yes. Rules are written database-agnostic, with a "PostgreSQL note" field where behaviour differs (uuid defaults, `insert_all` returning, check constraints syntax).

**Answer**: SQLite is the Rails 8 default and the core team's stated production target for single-server apps. Using it keeps the lab identical to generator evidence. Projects that choose PostgreSQL are still omakase (`rails new -d postgresql`), so the allowlist includes `pg` as a generator-produced gem.
Sources: Rails 8.0 release notes (SQLite production readiness, Solid adapters); `app_base.rb` `database_gemfile_entry`.

---

❓ **Q5** - **Lab composition**: Two apps, one Minitest (pure `rails new`) and one RSpec (`rails new --skip-test` plus rspec-rails), with scaffold, authentication, one Turbo Stream broadcast and one Solid Queue job generated and committed untouched?

➡️ Yes.

**Answer**: The two testing conventions produce different generated files (test/ vs spec/, fixtures paths, system test base class), and both testing skills need generator evidence for their examples. Generator facts already confirmed for 8.1: the scaffold controller is HTML-only (no `respond_to`, no JSON format), uses `params.expect`, renders failures with `:unprocessable_content`, and redirects after update/destroy with `:see_other`; jbuilder is present in the Gemfile but commented out unless `--api`.
Sources: `scaffold_controller/templates/controller.rb.tt` and `app_base.rb#jbuilder_gemfile_entry` on `8-1-stable`.

---

## Round 2: doctrine shape

❓ **Q6** - **Where rules live in a consumer app**: CLAUDE.md under 200 lines for cross-cutting rules, plus path-scoped `.claude/rules/*.md` files per rule area, instead of one dense CLAUDE.md?

➡️ Yes.

**Answer**: Claude Code documents a target of under 200 lines per CLAUDE.md and supports `.claude/rules/` Markdown files with a `paths` frontmatter that loads a rule file only when Read, Write or Edit touches a matching file. That is a better fit than always-loaded text: model rules load when editing `app/models/**`, routing rules on `config/routes.rb` and `app/controllers/**`, Hotwire rules on `app/views/**` and `app/javascript/**`, migration rules on `db/**`, testing rules on `spec/**` or `test/**`. CLAUDE.md keeps the versions, the allowlist pointer, the testing choice and the handful of rules that apply before any file is opened (resource-first design, omakase, positive phrasing of what to build). Imports (`@path`) do not reduce context cost, so they are not a substitute.
Sources: code.claude.com/docs/en/memory.md (size target, `.claude/rules/`, `paths`, imports).

---

❓ **Q7** - **Rule record format**: YAML as the single source of truth (`doctrine/rules/<area>.yml`), from which a generator renders the `.claude/rules/<area>.md` files, the human pages and the verification run?

➡️ Yes.

**Answer**: The record needs machine-readable fields (`verify`, `versions`, `tier`, `evals`) and a human-readable reason. One YAML list per area keeps related rules together; rendering guarantees the rules Claude reads and the rules the verifier checks are the same text. Fields: `id`, `statement`, `rails_reason`, `source_tier`, `sources[]`, `versions`, `kind` (rails_intent | owner_preference | contested), `exceptions`, `postgres_note`, `verify`, `evals[]`.

---

❓ **Q8** - **Rule phrasing**: Positive statements of the behaviour to produce, with the declined list kept as disclosed reference behind a pointer, instead of a "things we don't do" section in CLAUDE.md?

➡️ Yes.

**Answer**: The writing-for-agents reference argues that prohibitions drag the forbidden behaviour into context; the doctrine's whole purpose is to change defaults under silence, so always-loaded negatives would work against it. Each former "don't" becomes a positive rule ("Authenticate with the Rails generator's Session model") with the declined alternative and Rails's reason recorded in `doctrine/declined.md`, reached by one pointer: "Before proposing a gem or a new layer, read declined.md."

---

❓ **Q9** - **Distribution**: Ship skills and hooks as a Claude Code plugin (manifest plus `hooks/hooks.json`), and ship CLAUDE.md, `.claude/rules/` and the allowlist through `bin/doctrine install`?

➡️ Yes. **OWNER DECISION: accepted 2026-10-29** (ADR-0006).

**Answer**: Three facts push toward a plugin: `claude plugin eval` only targets a plugin, so the eval harness needs one anyway; plugins update in place across projects; hooks and skills are first-class plugin contents. Rules files and CLAUDE.md are per-project files, so an install script writes them and a `doctrine update` re-renders them from the pinned rule records. Alternative rejected: copying `.claude/` by hand into each app (no updates, no evals).
Sources: code.claude.com/docs/en/plugin-evals.md; plugins/manifest-reference.md; plugins/loading.md.

---

## Round 3: enforcement

❓ **Q10** - **Hook implementation**: Ruby scripts using only the standard library (`json`), exit code 2 with the reason on stderr for blocks, stdout for SessionStart context, `additionalContext` for PostToolUse feedback, tested with Minitest in this repo?

➡️ Yes.

**Answer**: Claude Code hooks receive JSON on stdin (`hook_event_name`, `tool_name`, `tool_input`, `cwd`), block with exit 2 (stderr becomes the reason Claude sees) or with a `permissionDecision: deny` JSON, inject SessionStart context through plain stdout, and can attach `hookSpecificOutput.additionalContext` to a PostToolUse result. Scripts live in `.claude/hooks/` or a plugin's `hooks/` and are referenced through `${CLAUDE_PROJECT_DIR}` or `${CLAUDE_PLUGIN_ROOT}`. Matchers `Edit|Write` and `Bash` are exact-name lists. Ruby is the project's language and needs no gems for this; Minitest because this repo is tooling (ADR-0001).
Sources: code.claude.com/docs/en/hooks.md.

---

❓ **Q11** - **Gem allowlist**: Derive it from the 8.1 `rails new` Gemfile template rather than hand-write it, add `rspec-rails`, and keep commented-out entries (jbuilder, bcrypt, rack-cors) allowed since `rails new` can produce them?

➡️ Yes.

**Answer**: The 8.1 template yields: rails, propshaft, sqlite3 or pg or mysql2/trilogy (by `-d`), puma, importmap-rails, turbo-rails, stimulus-rails, jbuilder (commented unless `--api`), bcrypt (commented), tzinfo-data, solid_cache, solid_queue, solid_cable, bootsnap, kamal, thruster, image_processing, rack-cors (commented, API), debug, bundler-audit, brakeman, rubocop-rails-omakase, web-console, capybara, selenium-webdriver, plus jsbundling-rails/cssbundling-rails when `-j`/`-c` options are used (Rails-org gems, still omakase). Add rspec-rails per ADR-0001. The allowlist file records the generating option next to each gem so a reviewer sees why it is there.
Sources: `Gemfile.tt` and `app_base.rb` on `8-1-stable`.

---

❓ **Q12** - **Forbidden paths hook**: Block writes to `app/services/**`, `app/policies/**`, `app/repositories/**`, `spec/factories/**`, `test/factories/**` and `.env*`, with the message naming the rule and the Rails home for that code?

➡️ Yes, with this exact list and nothing broader.

**Answer**: Each path is a layer name Rails declined or replaced: services and repositories (rich models, concerns, plain Ruby objects under `app/models`), policies (ADR-0004), factories (fixtures), dotenv (credentials). The block is on the directory, not on plain Ruby objects, which remain welcome under `app/models`. Presenters and form objects are left alone: Rails has no stated position beyond helpers and `ActiveModel::API`, so they are owner preference, not Rails intent, and a hook enforces only Rails intent.

---

❓ **Q13** - **Frozen migrations hook**: Block edits to a migration whose version is at or below the version in `db/schema.rb`?

➡️ Yes.

**Answer**: Rails guides state that a migration that has run should not be edited; a new migration is the mechanism. The check is mechanical: compare the file's timestamp prefix with `ActiveRecord::Schema[...].define(version:)`.
Source: Rails Guides, Active Record Migrations, "Changing Existing Migrations".

---

## Round 4: testing conventions

❓ **Q14** - **RSpec project defaults**: Keep rspec-rails's generated `rails_helper.rb` (fixtures at `spec/fixtures`, transactional fixtures on), add `config.global_fixtures = :all` as owner preference, request specs by default, system specs only for browser flows?

➡️ Yes.

**Answer**: The generated `rails_helper.rb` already sets `config.fixture_paths = [Rails.root.join('spec/fixtures')]` and `config.use_transactional_fixtures = true`; `infer_spec_type_from_file_location!` is commented out, so generated specs carry explicit `type:` metadata. rspec-rails 8.0.4 (2026-03) supports Rails ≥ 7.2 and carries Rails 8.2 fixes only in unreleased changes, which is the lag ADR-0001 accepts. Controller specs need `rails-controller-testing`, which is outside the allowlist, so request specs are the only option, which matches the Rails recommendation.
Sources: rspec-rails `rails_helper.rb` generator template; rspec-rails Changelog.

---

❓ **Q15** - **Minitest project defaults**: Generated `test_helper.rb` as is (parallelize, fixtures :all), declarative `test "..." do` blocks, integration tests over controller tests, `bin/ci` as the gate?

➡️ Yes.

**Answer**: Rails 8.1 ships `bin/ci` driven by `config/ci.rb` with setup, RuboCop, bundler-audit, importmap audit, Brakeman and `bin/rails test` steps, and an optional GitHub signoff. In RSpec projects `config/ci.rb` swaps the test step for `bundle exec rspec`.
Sources: Rails 8.1 release notes; `config/ci.rb.tt` on `8-1-stable`.

---

## Round 5: evaluation

❓ **Q16** - **Eval engine**: Use `claude plugin eval` for runs, ablation and judged items, keep its temp workspaces, and run mechanical checks over those workspaces with a small Ruby script, instead of writing a session runner from scratch?

➡️ Yes, with a verification ticket first.

**Answer**: `claude plugin eval` runs each case three times by default in a fresh non-interactive session with only the plugin loaded, supports a no-plugin baseline arm with a reported delta, a `--judge-model`, `llm` graders decided by two of three votes, `regex`, `tool_used`, `tool_order`, `file_exists` and `baseline` graders, JSON and HTML reports, and `--keep-temp`. It has no shell graders, so mechanical checks (Gemfile unchanged, routes shape, specs present, RuboCop clean, suite green) run afterwards over the kept workspaces. Risk to verify in the first eval ticket: whether kept workspaces are complete enough to run `bin/rails routes` and the suite; the fallback is a runner over `claude -p --output-format json` with per-condition settings files.
Sources: code.claude.com/docs/en/plugin-evals.md; headless.md; cli-reference.md.

---

❓ **Q17** - **Judge**: A different model from the one under test, with the rule text and sources in its prompt and a quoted line from the diff required as evidence, and mechanical checks ranked above it?

➡️ Yes.

**Answer**: Judge and subject share training biases, which is why mechanical checks come first and the judge only grades items that need reading. The `--judge-model` flag makes separation a one-line choice.

---

❓ **Q18** - **Negative tasks**: Include these five from the start?

➡️ Yes:
1. A report that needs a window function: raw SQL through `find_by_sql` or `select` is right; wrapping it in Arel or Ruby is over-correction.
2. A rich text editor with collaborative cursors inside one panel: a JavaScript component mounted inside a Turbo Frame is right; Stimulus-only is over-correction.
3. A multi-step signup that spans three models and is not persisted until the end: a plain Ruby object with `ActiveModel::API` under `app/models` is right; forcing it into an Active Record model or a callback chain is over-correction.
4. Sending a welcome email: an `after_create_commit` callback that enqueues a mailer is wrong; the controller or the model method that performs the signup enqueues it. Tests whether "callbacks for internal consistency only" is applied.
5. An ambiguous request ("add tags") where the right first move is a question about whether tags are shared across users, not code.

---

❓ **Q19** - **Pass criteria**: Per-rule pass rate at or above 0.8 across runs under doctrine plus skills plus hooks, every rule area above baseline, and over-correction on negative tasks no worse than baseline?

➡️ Yes.

---

## Round 6: source mining

❓ **Q20** - **Mining mechanics**: Given that this session's GitHub API is bound to the configured repository, mine rails/rails through a bounded clone (CHANGELOGs, source doc comments, `git log` on `8-1-stable`), guides and rubyonrails.org pages, and public PR pages fetched as web pages, and attempt to attach rails/rails with credentials once in Phase 1 to see whether API reads become available?

➡️ Yes.

**Answer**: `gh api search/...` and `gh api repos/rails/rails/...` both return 403 in this session; anonymous git reads of public repositories work. PR descriptions are not in git history (Rails uses merge commits), so PR pages are fetched over HTTPS. CHANGELOG entries name their PR numbers, which drives the fetch list.

---

❓ **Q21** - **Core-team weighting**: Use the roster on rubyonrails.org/community as the Tier 2 author list, with DHH as direction and the rest as engineering trade-off voices, and treat 37signals developers as Tier 3?

➡️ Yes. The roster snapshot lives in `docs/research/mvc-active-record-decisions.md`.

---

## Round 7: environment and fog

❓ **Q22** - **Phase 0 environment**: Run the lab in this cloud container with rbenv 3.4.x, verifying first that RubyGems is reachable through the proxy, and keep a `bin/setup-lab` script so the lab can be regenerated anywhere?

➡️ Yes. First ticket of Phase 0 is that reachability check.

---

❓ **Q23** - **MCP**: Nothing to decide now?

➡️ Correct. ADR-0003 gates it on eval evidence.

---

## Not yet specified (fog)

- The content of each rule area: depends on the research files and the lab's generator evidence.
- Eval task bodies beyond the five negatives: written per rule area once rule ids exist.
- How the version delta is generated from CHANGELOGs (script or curated).
- Whether `claude plugin eval` kept workspaces support post-hoc mechanical checks (Q16 risk).

## Out of scope

- Devise, Pundit, FactoryBot, shoulda-matchers, database_cleaner, Sidekiq, Redis caching, Webpacker, ViewComponent (ADR-0001 and Q12).
- Ruby 4.0 features (ADR-0005).
- An MCP server before eval evidence (ADR-0003).
