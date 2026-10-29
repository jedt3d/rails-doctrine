# Adversary review: planning documents on `feature/doctrine-planning`

Date: 2026-10-29. Role: Adversary (ADR-0007). Scope: README.md, GLOSSARY.md, docs/decisions.md, docs/adr/0001 to 0007, docs/grill/2026-10-29-doctrine-plan.md, docs/specs/phase-0 to phase-6, docs/execution-plan.md, docs/plan.md (where the specs do not supersede it), docs/agents/*.md, checked against docs/research/*.md and primary sources fetched on 2026-10-29.

Method: every disputed claim is quoted, then contradicted by a quote from a primary source (Rails `8-1-stable` on raw.githubusercontent.com, guides sources on `8-1-stable`, rubyonrails.org, ruby-lang.org, rubygems.org API, rspec-rails `v8.0.4`, jbuilder `main`, thor `main`, code.claude.com/docs/en/*.md) or by the repository's own text. Local checks ran in this cloud container. GitHub HTML pages (github.com/rails/rails/pull/...) returned HTTP 403, so PR descriptions were not re-checked. The tracker for this repository was read with `gh api`.

Severity: **blocks-build** = building the phase as written produces wrong evidence, a broken lab, an unmeasurable condition or a deadlock. **should-fix** = wrong or weak but the phase can start. **nit** = wording or hygiene.

## Summary

| id | severity | document | claim in one line |
| --- | --- | --- | --- |
| A-01 | blocks-build | decisions.md, grill Q5/Q11 | jbuilder is active by default, not commented out, so a default 8.1 scaffold emits `respond_to` and JSON |
| A-02 | blocks-build | ADR-0006, grill Q9/Q16, phase-4 | `claude plugin eval` never loads CLAUDE.md or `.claude/rules`, so the doctrine condition cannot be measured with it |
| A-03 | blocks-build | phase-0 | Generating authentication after the scaffold makes the generated scaffold tests fail; "bin/ci green" and "untouched" cannot both hold |
| A-04 | blocks-build | phase-0, grill Q15, decisions.md | `--skip-test` removes the test step from `config/ci.rb` and drops capybara/selenium; the RSpec app's `bin/ci` runs no tests |
| A-05 | should-fix | decisions.md, grill Q11, phase-2 | The allowlist omits generator-produced gems (tailwindcss-rails, dartsass-rails, redis) and misstates what `-c` adds |
| A-06 | should-fix | decisions.md section 3 | `assert_difference` / `assert_changes` are not available in rspec-rails example groups by default |
| A-07 | should-fix | plan.md rule record example | The example `verify` command targets a file that does not exist on 8-1-stable |
| A-08 | should-fix | phase-4, grill Q16 | Several engine details are wrong or missing (`--baseline` flag, judge default, trace window, turn/time caps, Bash sandbox prerequisites) |
| A-09 | should-fix | decisions.md, grill Q3, phase-0 | `.ruby-version` 3.4.12 cannot run on the 3.4.10 the container has, and `rails new` writes its own |
| A-10 | should-fix | phase-2, GLOSSARY | Hooks are described as deterministic enforcement; the docs call `if` filtering best-effort and a Stop hook cannot "call" a skill |
| A-11 | nit | grill Q14 | "Controller specs need rails-controller-testing" is not what rspec-rails says |
| A-12 | nit | grill Q4 | The SQLite claim is attributed to the 8.0 release notes, which do not make it |
| A-13 | nit | execution-plan header | Price line omits the Haiku 5.5 long-prompt tier and the cache-read prices the budget relies on |
| A-14 | blocks-build | grill Q18 item 4, plan.md | The pre-registered negative task marks as wrong exactly what the Rails Callbacks guide shows |
| A-15 | should-fix | grill Q13, phase-2 | The frozen-migrations criterion (schema version) is not the guide's criterion (committed or propagated) |
| A-16 | should-fix | grill Q12 | "A hook enforces only Rails intent" is false for four of the six forbidden paths and for the gem allowlist |
| A-17 | should-fix | decisions.md section 5, grill Q18 item 2 | "Client-side components only inside a Turbo Frame" has no Rails source and misdescribes Frames |
| A-18 | should-fix | ADR-0004 | "the pattern the Rails generator ... uses" overreaches; the generator ships no authorization |
| A-19 | nit | phase-0, ADR-0002 | rspec-rails generator output in the lab will be treated as Tier 0 Rails evidence |
| A-20 | blocks-build | tracker, phase-1/3/4 | The native blocking edges plus the checklists form two dependency cycles |
| A-21 | should-fix | plan.md, phase-4, execution-plan, decisions.md | Runs per case (3 vs 5) and task counts (15-30, 25-30, 30-35, 36) disagree |
| A-22 | should-fix | grill Q19 vs execution-plan | Per-rule 0.8 pass bar contradicts "per rule, directional only" |
| A-23 | should-fix | grill Q18, GLOSSARY | Negative tasks 4 and 5 do not fit the glossary definition of a negative task |
| A-24 | should-fix | plan.md task template, phase-4 | Mechanical checks penalize the Rails generator path, break in the Minitest app and fit one route name |
| A-25 | nit | several | Document hygiene: ADR count, read order, stale status, glossary violations, stale names |
| A-26 | should-fix | phase-0 checklist | Items that cannot be met or observed as worded |
| A-27 | should-fix | phase-1 checklist | Items that test plumbing or are open-ended |
| A-28 | should-fix | phase-2 checklist | "Verified in a live session" and "exact matchers" are not observable |
| A-29 | should-fix | phase-3 checklist | "Improvement per rule area" is unfalsifiable at the stated noise |
| A-30 | should-fix | phase-4 checklist | Several controls have no threshold or rely on self-report |
| A-31 | nit | phase-5, phase-6 checklists | "Implemented and documented", "latency budgets" have no numbers |
| A-32 | blocks-build | execution-plan 2.9, grill Q19, phase-4 | Noise-floor arithmetic ignores clustering and comparison; "every area above baseline" is near-unattainable even for a working doctrine |
| A-33 | should-fix | execution-plan 2.2, phase-4 | A 30 percent split stratified by 12 areas is about 9 tasks: fewer than one per area |
| A-34 | should-fix | execution-plan 2 and 3 | Fable 5.1 writes the rules and judges the Opus runs; the protocol's own self-preference risk |
| A-35 | should-fix | execution-plan 2.3, 2.4, 2.6, 2.7 | Grader validation controls test only easy cases and never measure false passes |
| A-36 | should-fix | execution-plan 2.1 | Pre-registration by git tag is neither tamper-evident nor bounded |
| A-37 | should-fix | execution-plan 2.5, phase-4 | Blinding fails when the judge sees transcripts that show skills and hooks firing |
| A-38 | should-fix | execution-plan 2, phase-4 | Tasks authored from rule statements measure doctrine compliance, not Rails intent |
| A-39 | should-fix | execution-plan 3 | Cost estimate omits triggering evals, duplicated baselines and judge calls |
| A-40 | should-fix | execution-plan 2, ADR-0007 | Adversary independence is weaker than stated and its recall is never measured |
| A-41 | should-fix | grill Q16 fallback, docs/agents/skills.md | The fallback runner would load the machine's user config and installed plugins into the baseline |
| A-42 | should-fix | ADR-0006, grill Q1 | `bin/doctrine install`, `doctrine update` and the plugin manifest have no ticket or checklist item |
| A-43 | should-fix | ADR-0001, phase-2 | No control on JavaScript dependencies (importmap pins, package.json) |
| A-44 | should-fix | phase-4 | Eval workspace provisioning (lab copy, gems, sandbox, network) is unplanned |
| A-45 | should-fix | ADR-0002, phase-1, phase-6 | Rejected PRs and the 7.0 to 8.1 delta cannot be mined the way the specs describe |
| A-46 | should-fix | execution-plan 4 (#2) | Lab determinism check ignores floating gem versions |
| A-47 | should-fix | phase-1 paths triggers | Query and form rules do not load where N+1s and form code are written |
| A-48 | should-fix | phase-3 | Triggering evals cannot fire `paths`-scoped skills in an empty workspace; no cross-variant negative |
| A-49 | nit | phase-3 references | Guides excerpts are CC BY-SA 4.0; the repository is MIT |

Counts: blocks-build 7, should-fix 35, nit 7.

---

## 1. Factual claims

### A-01 (blocks-build): jbuilder is active by default, so the default scaffold is not HTML-only

Documents and text:
- decisions.md: "jbuilder (commented out unless --api)".
- grill Q5: "Generator facts already confirmed for 8.1: the scaffold controller is HTML-only (no `respond_to`, no JSON format) ... jbuilder is present in the Gemfile but commented out unless `--api`."
- grill Q11: "jbuilder (commented unless `--api`)".
- decisions.md, revisions: "Rails 8.1 scaffold controllers are HTML-only with `params.expect`, `:unprocessable_content` and `:see_other`; rules cite this as Tier 0."

Evidence:
- `railties/lib/rails/generators/app_base.rb` on 8-1-stable: `GemfileEntry.new "jbuilder", nil, "Build JSON APIs with ease [https://github.com/rails/jbuilder]", {}, options[:api]`, where the fifth argument is `commented_out` (`def initialize(name, version, comment, options = {}, commented_out = false)`). jbuilder is therefore uncommented in a default app and commented out only with `--api`. The inverse of the docs.
- jbuilder `lib/jbuilder/railtie.rb`: `generators do |app| ... require 'generators/rails/scaffold_controller_generator' end`, and that file reopens `Rails::Generators::ScaffoldControllerGenerator` with `source_paths << File.expand_path('../templates', __FILE__)` and `hook_for :jbuilder, type: :boolean, default: true`.
- thor `lib/thor/actions.rb`: `paths += source_paths; paths << source_root if source_root`, and `find_in_source_paths` returns the first match. jbuilder's `controller.rb` is searched before railties' `controller.rb.tt`.
- jbuilder `lib/generators/rails/templates/controller.rb`: `respond_to do |format| ... format.json { render :show, status: :created, location: ... }`.

So in an app made by `rails new` with defaults (the lab), `rails g scaffold Post` produces a controller with `respond_to` and `format.json` plus `.json.jbuilder` views. The railties HTML-only template is used only with `--skip-jbuilder`. Rules that cite "HTML-only scaffold" as Tier 0, and the judged item "Turbo Stream or redirect used, not JSON + client code", would be contradicted by the lab itself.

Proposed change: correct the three statements. Decide explicitly whether the lab uses defaults (JSON-capable scaffold, which is what Claude will see in real apps) or `--skip-jbuilder`, record the choice as owner preference, and cite the railties template only together with that option. Add a Phase 0 checklist item: "`grep -c respond_to lab/*/app/controllers/posts_controller.rb` recorded in lab/SOURCES.md".

### A-02 (blocks-build): `claude plugin eval` does not load CLAUDE.md or `.claude/rules`

Documents and text:
- ADR-0006: "Skills and hooks are packaged as a Claude Code plugin ... so ... `claude plugin eval`, which only targets plugins, can measure them. CLAUDE.md, `.claude/rules/*.md` and the gem allowlist are per-project files written ... by `bin/doctrine install`."
- phase-4: "Conditions beyond baseline and full plugin are produced by plugin variants (rules only; rules plus skills; rules plus skills plus hooks)."
- grill Q16: the only named risk is "whether kept workspaces are complete enough to run `bin/rails routes` and the suite".

Evidence (code.claude.com/docs/en/plugin-evals.md, "How runs are isolated"): "Nothing personal or project-level loads. ... no `.claude/` directory, `CLAUDE.md`, or `.mcp.json` loads from above the workspace or inside it, even one a `scaffold_script` wrote ... Ship any skills, agents, hooks, or MCP servers a case depends on in the plugin under test, since a `scaffold_script` can supply only files and git state." And plugins-reference.md: "A `CLAUDE.md` at the plugin root isn't loaded as context ... To include instructions that load into Claude's context, put them in a skill."

The doctrine's primary delivery mechanism (grill Q6: CLAUDE.md plus path-scoped rules) cannot exist inside a plugin-eval run. A "rules only" plugin variant would have to inject the rules some other way (a skill or a SessionStart hook), which measures a different mechanism from the one shipped: always-on or on-demand text instead of path-scoped loading.

Proposed change: make the `claude -p` runner the primary engine for all conditions that include rules files (it loads project CLAUDE.md and `.claude/rules` from the working directory, per headless.md), and keep `claude plugin eval` only for skill-triggering checks. Amend ADR-0006's second reason ("so `claude plugin eval` ... can measure them") and the Q16 answer. Promote this from "fog" to the first Phase 4 ticket.

### A-03 (blocks-build): authentication after scaffold breaks the generated tests

Document and text: phase-0: "Generators run in this order ...: `rails new`, `scaffold Post title body:text`, `authentication`, ..."; "Generated files are never edited in the lab"; checklist "`bin/ci` green in both apps".

Evidence:
- `railties/lib/rails/generators/rails/authentication/authentication_generator.rb` (8-1-stable): `inject_into_class "app/controllers/application_controller.rb", "ApplicationController", "  include Authentication\n"`; the generated concern has `before_action :require_authentication` (docs/research/mvc-active-record-decisions.md section 4.3).
- `railties/lib/rails/generators/test_unit/scaffold/templates/functional_test.rb.tt`: `get <%= index_helper(type: :url) %>` then `assert_response :success`, with no sign-in. The test_unit authentication generator only adds `test/test_helpers/session_test_helper.rb`; it does not edit existing tests.

After the authentication step every PostsController test receives a redirect to the session page, so `bin/rails test` fails unless generated tests are edited. The same holds for rspec-rails request specs.

Proposed change: either generate authentication first and the scaffold second (scaffold tests still need `sign_in_as`, so check), or allow one recorded, minimal edit commit ("lab: sign in scaffold tests") outside the "untouched" rule, and say so in the spec. Rewrite the checklist item to name the expected failing tests or the edit commit.

### A-04 (blocks-build): the RSpec app's `bin/ci` runs no tests and has no system-test gems

Documents and text:
- grill Q15: "In RSpec projects `config/ci.rb` swaps the test step for `bundle exec rspec`."
- decisions.md: "`bin/ci` (Rails 8.1) runs `bundle exec rspec` in place of `rails test`."
- phase-0 checklist: "`rails new rspec_app --skip-test` ... committed untouched." and "`bin/ci` green in both apps."
- decisions.md: "System specs only for flows that need a browser."

Evidence:
- `config/ci.rb.tt` on 8-1-stable: `<% unless options[:skip_test] -%>  step "Tests: Rails", "bin/rails test"` ... `<% end -%>`. With `--skip-test` there is no test step at all, and nothing swaps one in.
- rspec-rails v8.0.4 `install_generator.rb` copies `.rspec`, `spec/spec_helper.rb` and `spec/rails_helper.rb` only; it does not touch `config/ci.rb`.
- `app_base.rb`: `def depends_on_system_test? !(options[:skip_system_test] || options[:skip_test] || options[:api]) end`, which gates the `capybara` and `selenium-webdriver` group in `Gemfile.tt`.

So "`bin/ci` green" in the RSpec app is vacuous (it runs RuboCop, audits and seeds, no specs), the swap the docs describe is a hand edit that violates "untouched", and system specs need two gems added by hand.

Proposed change: state that the RSpec app needs a recorded edit adding `step "Tests: RSpec", "bundle exec rspec"` to `config/ci.rb` and adding capybara and selenium-webdriver, both as owner-preference commits. Change the checklist to "`bin/ci` output in rspec_app contains a line for `bundle exec rspec` with N examples, 0 failures".

### A-05 (should-fix): the allowlist is not what the template produces

Documents and text: decisions.md: "jsbundling-rails / cssbundling-rails when -j / -c are used. The allowlist file is generated from the template, not hand-written." grill Q11: "plus jsbundling-rails/cssbundling-rails when `-j`/`-c` options are used".

Evidence (`app_base.rb`, 8-1-stable):
- `CSS_OPTIONS = %w( tailwind bootstrap bulma postcss sass )`, and `css_gemfile_entry`: `if !using_js_runtime? && options[:css] == "tailwind"` then `GemfileEntry.floats "tailwindcss-rails"`; `elsif ... options[:css] == "sass"` then `GemfileEntry.floats "dartsass-rails"`; else `cssbundling-rails`.
- `cable_gemfile_entry`: `if !options[:skip_action_cable] && options[:skip_solid]` then `GemfileEntry.new("redis", ">= 4.0.1", ...)`.

`rails new -c tailwind` (the most common `-c`) yields tailwindcss-rails, not cssbundling-rails. tailwindcss-rails, dartsass-rails and redis are generator-produced and absent from every list in the docs. The hand-written lists prove the allowlist was not yet derived from the template, contrary to the claim.

Proposed change: generate the allowlist by running `rails new --pretend` (or parsing `app_base.rb`) across the option matrix and record the option per gem, as Q11 promises. Add tailwindcss-rails, dartsass-rails and redis (or exclude `--skip-solid` explicitly as owner preference).

### A-06 (should-fix): Rails assertions are not available in RSpec by default

Document and text: decisions.md: "Rails assertions (assert_difference, assert_changes) are available and fine."

Evidence: rspec-rails v8.0.4 `lib/rspec/rails/adapters.rb`: `Assertions = Minitest::Assertions`, and `MinitestAssertionAdapter.assertion_method_names` selects methods of `::RSpec::Rails::Assertions` only. `rails_example_group.rb` includes `ActiveSupport::CurrentAttributes::TestHelper` and `ActiveSupport::ExecutionContext::TestHelper`, not `ActiveSupport::Testing::Assertions`, where `assert_difference` and `assert_changes` are defined. Request example groups add `ActionDispatch::IntegrationTest::Behavior`, whose includes (8-1-stable `integration.rb`) are `Integration::Runner`, `ActionController::TemplateAssertions` and `TestHelpers::PageDumpHelper`. I inferred this from source, not from a run; one `rspec` run in the lab settles it.

Proposed change: either add `config.include ActiveSupport::Testing::Assertions` to rails_helper (recorded as owner preference, which conflicts with "Keep rails_helper.rb close to generated") or use RSpec's `change` matcher and drop the sentence. Verify in Phase 0.

### A-07 (should-fix): the example rule's `verify` command fails on the pinned version

Document and text: plan.md rule record: `verify: grep -q "def normalizes" $(bundle show activerecord)/lib/active_record/normalization.rb`, `rails_reason: ... it must apply on assignment, on query, and in every entry point`.

Evidence: `https://raw.githubusercontent.com/rails/rails/8-1-stable/activerecord/lib/active_record/normalization.rb` returns 404; `activemodel/lib/active_model/attributes/normalization.rb` returns 200 and its doc comment reads "The normalization is applied when the attribute is assigned or validated." The 8.1 activemodel CHANGELOG: "Backport `ActiveRecord::Normalization` to `ActiveModel::Attributes::Normalization`". docs/research/mvc-active-record-decisions.md section 7 item 8 already recorded the move. The "on query" wording comes from the 8-0-stable comment.

The model example that every rule author will copy fails `bin/doctrine verify` on day one, and its reason quotes the wrong version's doc.

Proposed change: fix the example to the 8.1 path and wording, and add a Phase 1 test: the example record in the docs passes `bin/doctrine verify` against the lab.

### A-08 (should-fix): engine details in the Phase 4 spec

Document and text: phase-4: "Engine: `claude plugin eval` with `--baseline`, `--runs 3`, `--judge-model`, `--keep-temp`, `--json`."

Evidence (plugin-evals.md):
- There is no `--baseline` option; the arms are set with "`--ablation <mode>` ... `none` runs one arm; `with-without` adds the no-plugin baseline".
- "By default, the judge for `llm` and `baseline` graders is the model Claude Code uses for background tasks." The execution plan's Opus or Fable judge needs `--judge-model` every time.
- `trace`: "an `llm` judge sees the first 12 and the last 12" messages; `files`: "The list of paths Claude created ... Not their contents, and not files that ... Claude only modified."
- `max_turns` default `10`, `timeout_seconds` default `300`. A Rails feature with migrations, specs and a suite run will hit both.
- "If you grant Bash or PowerShell on a machine with no sandbox backend, Claude Code refuses each run ... on Linux, install `bubblewrap` and `socat` first." In this container `command -v bwrap` and `command -v socat` both print nothing.
- "network access is limited to domains you grant", so `bundle add` fails in every arm for reasons unrelated to the doctrine.

Proposed change: correct the flag, set `max_turns` and `timeout_seconds` per case, name the judge model explicitly, add a sandbox prerequisite ticket, and decide the network policy (see A-44).

### A-09 (should-fix): `.ruby-version` cannot be 3.4.12 here

Documents and text: decisions.md: "Ruby 3.4.12 in `.ruby-version` (3.4.10 is what the cloud container's rbenv offers today ...)"; grill Q3: "`.ruby-version` = `3.4.12`; the lab setup falls back".

Evidence: `rbenv install --list` here prints `3.4.10` as the 3.4 entry; ruby-lang.org/en/downloads lists "Ruby 3.4.12" (confirmed). A `.ruby-version` naming an uninstalled version makes rbenv refuse to run `ruby` and `bundle` in that directory. `rails new` also writes its own `.ruby-version` from the running Ruby, so the lab apps would carry 3.4.10 while the repo says 3.4.12, and changing it edits generated output.

Proposed change: `.ruby-version` holds the installed patch, recorded in lab/SOURCES.md; the doctrine's pin is "3.4.x" (patch is an environment fact, not a rule). Alternatively update ruby-build first and make that the first Phase 0 ticket.

### A-10 (should-fix): hooks are not deterministic enforcement, and a Stop hook cannot call a skill

Documents and text: GLOSSARY "Hook: A script ... enforcing a rule deterministically." decisions.md: "A PreToolUse hook rejects Gemfile edits and `bundle add` outside this allowlist." phase-2: "`doctrine-review` (Stop; advisory checklist over the diff with rule ids)"; phase-3: "rails-doctrine-review is also what the Stop hook calls."

Evidence (hooks.md): "Because the `if` filter is best-effort, use the permission system rather than a hook to enforce a hard allow or deny." For Stop, the available outputs are "`decision: "block"`, `reason`" and `additionalContext` "for non-error feedback that continues the conversation"; hook types are `command`, `http`, `mcp_tool`, `prompt`, `agent`. None invokes a skill. A Gemfile can be changed with `Bash` (`sed -i`, `cat >>`) which no `Edit|Write` matcher sees.

Proposed change: drop "deterministically" from the glossary; add permission `deny` rules alongside hooks; specify the Stop mechanism (for example `decision: "block"` with a reason that names the review skill, guarded by `stop_hook_active`), and state that it continues the session, which changes what the eval measures under condition 4.

### A-11 (nit): controller specs and rails-controller-testing

Document and text: grill Q14: "Controller specs need `rails-controller-testing`, which is outside the allowlist, so request specs are the only option".

Evidence: rspec-rails README: "Request specs provide a high-level alternative to controller specs. In fact, as of RSpec 3.5, both the Rails and RSpec teams discourage directly testing controllers in favor of functional tests like request specs." The extra gem is needed only for `assigns`/`assert_template`; controller specs themselves run without it.

Proposed change: cite the README sentence as the reason; drop the causal claim.

### A-12 (nit): SQLite source

Document and text: grill Q4: "SQLite is the Rails 8 default and the core team's stated production target for single-server apps." Source given: "Rails 8.0 release notes".

Evidence: the 8.0 release notes mention SQLite only for Solid Queue ("It also works with SQLite.") and removals. The claim's actual source is the announcement (rubyonrails.org/2024/11/7/rails-8-no-paas-required): "a bunch of work has gone into making the SQLite adapter and Ruby driver suitable for real production use in Rails 8", and Solid Queue "For more modest needs, it also works with SQLite". "Suitable" is not "target".

Proposed change: cite the announcement and use its words.

### A-13 (nit): price line

Document and text: execution-plan header: "Haiku 5.5 $0.10 / $0.50"; section 3: "once cache reads are counted".

Evidence: the claude-api reference (cached 2026-10-06): Haiku 5.5 "Prices above are for prompts up to 100K tokens ($0.50 / $2.50 beyond)"; cache reads are listed per model (Opus 5.5 and Sonnet 5.5 $0.20, Fable 5.1 $0.25). The four headline prices themselves match.

Proposed change: add the Haiku tier and the cache-read prices that the per-run estimate depends on.

---

## 2. Rails intent

### A-14 (blocks-build): negative task 4 contradicts the Callbacks guide

Document and text: grill Q18 item 4: "Sending a welcome email: an `after_create_commit` callback that enqueues a mailer is wrong; the controller or the model method that performs the signup enqueues it. Tests whether 'callbacks for internal consistency only' is applied." plan.md area 3: "callbacks for internal consistency only".

Evidence:
- Rails Guides, Active Record Callbacks (8-1-stable `guides/source/active_record_callbacks.md`, Tier 1), the canonical create-callback example: `after_create :send_welcome_email` ... `UserMailer.welcome_email(self).deliver_later`.
- docs/research/mvc-active-record-decisions.md section 3.4: Manrubia's `Filing` example "uses `after_create_commit :process_later, unless: :completed?` to enqueue a job" (Tier 3, 37signals practice).
- No source in the repository states "callbacks for internal consistency only". The research file quotes the guide's cautions ("Refrain from using methods like update, save ... within your callback methods"), which concern side effects on the record, not enqueueing.

Pre-registering this task freezes a Tier 4 preference as a Rails-intent grader and penalizes the guide's own example.

Proposed change: remove task 4 from the negatives, or relabel the rule `owner_preference` and the task an ordinary (non-negative) owner-preference task. If kept, the rule needs a Tier 0 to 2 source before `evals-v1` is tagged.

### A-15 (should-fix): frozen-migrations criterion

Documents and text: grill Q13: "Block edits to a migration whose version is at or below the version in `db/schema.rb` ... Rails guides state that a migration that has run should not be edited." phase-2 false-positive case: "editing a new, unmigrated migration is allowed."

Evidence (`active_record_migrations.md`, 8-1-stable, "Changing Existing Migrations"): "If you have already run the migration ... You must rollback the migration (for example with `bin/rails db:rollback`), edit your migration, and then run `bin/rails db:migrate`"; "In general, editing existing migrations that have been already committed to source control is not a good idea"; "However, editing a freshly generated migration that has not yet been committed to source control ... is common." The same guide: Rails records each run in the `schema_migrations` table, so "at or below the schema version" is not "has run" either (an older-timestamped migration from another branch can be pending).

The guide's line is "committed or propagated", and it explicitly sanctions rollback-edit-migrate for an uncommitted migration that has run. The hook would block that.

Proposed change: block edits only to migrations tracked by git at HEAD (`git ls-files --error-unmatch`), and add the guide's sanctioned case (migrated, uncommitted) as an allow test.

### A-16 (should-fix): forbidden paths and the allowlist enforce more than Rails intent

Document and text: grill Q12: "Each path is a layer name Rails declined or replaced: services and repositories ..., policies (ADR-0004), factories (fixtures), dotenv (credentials). ... a hook enforces only Rails intent."

Evidence:
- GLOSSARY "Declined: A feature or pattern the Rails core team explicitly chose not to adopt, recorded with its stated reason." No core-team rejection is cited for services, repositories, factories or dotenv. The only source on services is Manrubia, dev.37signals.com (research section 3.4), which plan.md classes as Tier 3 ("Tier 3: core authors outside the org: dev.37signals.com"), and ADR-0002 says Tier 3 "can support a rule but not establish it".
- Rails' own generated `.gitignore` (8-1-stable `gitignore.tt`): "# Ignore all environment files. /.env*", and `.kamal/secrets` comments "All secrets should be pulled from either password manager, ENV, or a file". The 8.2 draft notes on main: "Add `Rails.app.creds` for combined access to credentials stored in either ENV or the encrypted credentials file". Rails treats ENV as a first-class secrets source.
- The gem allowlist enforces ADR-0001, which is owner preference ("The owner prefers RSpec's readability"); the Doctrine says "Substitutions are still possible, but not required" (rubyonrails.org/doctrine). Practice check (plan.md Evaluation A step 6): basecamp/once-campfire and basecamp/writebook Gemfiles use `resque`, `redis` and `mocha`/`faker`; basecamp/fizzy uses `mocha`, `webmock`, `vcr`.

Proposed change: either drop "a hook enforces only Rails intent" or limit hooks to it. Label each forbidden path and the allowlist with its `kind` (most are `owner_preference`), and make block messages say "owner preference" where that is the truth.

### A-17 (should-fix): "client-side components only inside a Turbo Frame"

Documents and text: decisions.md section 5: "Client-side components only inside a Turbo Frame where the interaction model demands client state." grill Q18 item 2: "a JavaScript component mounted inside a Turbo Frame is right".

Evidence: Turbo handbook: "Turbo Frames allow predefined parts of a page to be updated on request. Any links and forms inside a frame are captured". Mounting a component is Stimulus's job. The research's only source on heavy clients is "roll out the heavy machinery for the part of your application that can really benefit from it ... Our calendars tend to use client-side rendering. Our text editor is Trix" (Stimulus handbook origin), which says nothing about Frames. The Rails JS guide (8-1-stable) shows `bin/importmap pin react react-dom` as a supported path. For rich text the omakase answer is Action Text.

Proposed change: restate as "a client-side component is mounted by a Stimulus controller on the part of the page that needs client state", cite the Stimulus origin quote, and rewrite negative task 2's expected answer accordingly (and decide whether Action Text is the expected first move).

### A-18 (should-fix): ADR-0004 overreaches on the generator

Document and text: ADR-0004: "We chose this because it is the pattern the Rails generator and the Rails core team's own applications use".

Evidence: the 8-1-stable authentication generator creates sessions, users, passwords and a mailer; it has no authorization code (`authentication_generator.rb`). PR #52328 as quoted in research section 4.2: "This is not intended to be an all-singing, all-dancing answer to every possible authentication concern." The core team's applications do use the pattern (see section 7), but they are Tier 3 under plan.md.

Proposed change: replace "the Rails generator" with the Security guide quote (Tier 1, which ADR-0004 already cites) and cite Fizzy/Campfire as Tier 3 practice.

### A-19 (nit): rspec-rails generator output is not Rails intent

Documents and text: phase-0: "the RSpec app adds `rspec-rails` and runs `rspec:install` before scaffolding so generated specs appear"; ADR-0002 Tier 0: "Rails source and generators".

Evidence: rspec-rails v8.0.4 `scaffold_generator.rb`: `class_option :view_specs, ... default: true`, `:helper_specs, ... default: true`, `:routing_specs, ... default: true`. The lab will contain view, helper and routing specs, and phase-3 says examples are "lifted from generated code".

Proposed change: classify rspec-rails generator output as third-party, not Tier 0, and decide whether the lab passes `--no-view-specs --no-helper-specs --no-routing-specs`.

---

## 3. Internal consistency

### A-20 (blocks-build): two dependency cycles

Documents and text:
- Tracker (via `gh api .../dependencies/blocked_by`): #5 Skills blocked by #6; #6 Evals blocked by #2, #3, #4; #4 Hooks blocked by #3.
- phase-4 checklist: "Acceptance met: every rule area above baseline under the full condition" and "Mechanism check: ... transcripts show rules and skills loading". The full condition includes skills.
- phase-1 checklist: "Second batch completed after the first eval run."
- docs/agents/issue-tracker.md: "A ticket is unblocked when every blocker is closed."

Cycle 1: #6 cannot close until skills exist; #5 cannot start until #6 closes. Cycle 2: #3 cannot close until an eval run exists; #6 (and #4) cannot start until #3 closes. The specs' prose ("Blocked by: Phase 1 (first batch ...)", "Runs its baseline before Phase 3 starts") describes partial dependencies the native edges cannot express.

Proposed change: split #3 into "first batch" and "second batch" issues and #6 into "harness and baseline" and "final confirmation" issues, and wire the edges to those.

### A-21 (should-fix): runs and task counts disagree

Documents and text:
- plan.md: "each task run 5 times per condition"; execution-plan 2.9: "With 30 tasks and 5 runs"; section 3: "Thirty tasks × five runs × four conditions".
- phase-4: "three runs per case" and "`--runs 3`".
- decisions.md: "Build the eval set (15 to 30 prompts)". plan.md: "Task set (25 to 30)" including "At least 5 negative tasks". phase-4: "twenty-five to thirty silent prompts, two to three per rule area, plus the five negative tasks" (30 to 35).
- phase-1 lists 12 rule areas; "two to three per rule area" gives 24 to 36. execution-plan 2.9: "per rule area (about 3 tasks × 5 runs)". With 30 tasks including 5 negatives, it is about 2 tasks per area.

Proposed change: one number each in PREREGISTRATION.md (tasks, negatives, areas, runs), and recompute the noise floor from those numbers (see A-32).

### A-22 (should-fix): per-rule pass bar

Documents and text: grill Q19: "Per-rule pass rate at or above 0.8 across runs under doctrine plus skills plus hooks" (answered "Yes"). execution-plan 2.9: "per rule, directional only. Decisions are made at overall and area level; per-rule numbers point at what to read, not at what to ship."

Evidence: a rule exercised by one task with 5 runs has at most 5 to 10 trials; Wilson 95 percent interval for 8/10 is 0.49 to 0.94.

Proposed change: delete the per-rule 0.8 criterion from Q19 or mark it superseded.

### A-23 (should-fix): negative tasks 4 and 5 are not negative tasks

Documents and text: GLOSSARY "Negative task: An eval task where Rails intent is the opposite of the doctrine's usual answer, used to detect over-correction." grill Q18 item 4 tests whether a doctrine rule "is applied"; item 5 tests asking a question.

Evidence: item 4's expected answer is the doctrine's usual answer, so it measures compliance, not over-correction. Item 5 has no doctrine rule it is the exception to. Mixing them into "over-correction rate on negative tasks" makes that metric partly a compliance metric.

Proposed change: keep three true negatives (1 to 3) and report 4 and 5 separately, or add a glossary term for "clarification task".

### A-24 (should-fix): the task template's mechanical checks fight the ADRs

Document and text: plan.md task file: `no_new_gems: git diff --quiet HEAD -- Gemfile`; `tests_added: git diff --name-only HEAD | grep -qE "^spec/(requests|models)/"`; `suite: bin/rails test || bundle exec rspec`; `new_resource_not_action: ... grep -qE "posts/:post_id/publication"`. phase-4 repeats "Gemfile unchanged".

Evidence:
- 8-1-stable `authentication_generator.rb` `enable_bcrypt`: `uncomment_lines "Gemfile", /gem "bcrypt"/` then `bundle_command("install --quiet")`. Running the generator that ADR-0004 mandates fails `no_new_gems`.
- `tests_added` can never pass in the Minitest lab app (tests live in `test/`).
- `a || b` passes if either passes; in the Minitest app rspec never runs, and in the RSpec app `bin/rails test` may succeed with no tests (unverified; check in Phase 0), so neither suite is guaranteed to run.
- The route regex accepts only one noun ("publication"). execution-plan section 1 names this failure mode: "Grader fit: eval tasks and graders are written to match what the author already writes".

Proposed change: compare the Gemfile's gem set against the allowlist rather than the file bytes; make checks testing-variant aware; run the variant's suite explicitly; check route shape (a new controller, no member route) rather than a noun.

### A-25 (nit): document hygiene

- execution-plan 5.2: "reviews the grill answers, the five ADRs and the seven specs". There are seven ADRs.
- README "Read in this order" omits docs/execution-plan.md, which ADR-0007 names as the protocol.
- decisions.md: "(Q9, owner to confirm)" while ADR-0006 says "Accepted by the owner on 2026-10-29".
- plan.md uses `our_preference` throughout; GLOSSARY: "Owner preference ... _Avoid_: house style, our taste"; specs use `owner_preference`. plan.md's Evaluation A is not superseded by any spec, so the old term is live.
- plan.md: "see DECISIONS.md" (file is docs/decisions.md); "`doctrine/rules/<area>.md`" vs grill Q7 "`doctrine/rules/<area>.yml`"; "Target under 150 lines" vs 200; "6 hooks + tests" vs phase-2 "all eight hooks"; hooks in `.claude/settings.json` vs ADR-0006 plugin `hooks/hooks.json`.
- phase-4 uses "cases" and "graders" (plugin-eval terms); GLOSSARY lists "test case" and "grader" under _Avoid_.

Proposed change: fix in one hygiene commit; add a banner to plan.md listing which sections are superseded.

---

## 4. Checklist quality

docs/agents/skills.md sets the bar: "Every spec checklist item is something a reviewer can observe, not 'understood' or 'considered'."

### A-26 (should-fix): Phase 0

| Item (quoted) | Problem | Replacement |
| --- | --- | --- |
| "Scaffold, authentication, Comment model, broadcast and job generated in both apps, one commit per generator." | No generator produces a broadcast; it is hand-written code in a generated file, contradicting "Generated files are never edited". | "One commit per generator command, each commit message is the exact command; the broadcast is a separate commit labelled `hand-written` with its diff in lab/diffs/." |
| "`bin/ci` green in both apps." | Cannot hold untouched (A-03) and is vacuous in rspec_app (A-04). | "`bin/ci` log committed for each app; minitest_app shows `bin/rails test` with N runs, 0 failures; rspec_app shows `bundle exec rspec` with N examples, 0 failures." |
| Implementation: "`bin/setup-lab` ... fails loudly if any step would alter a previous commit." | Every generator after `rails new` alters files from earlier commits (routes.rb, Gemfile, ApplicationController). Unmeetable as worded. | "`bin/setup-lab` fails if any file changes that the generator's own output log (`create`/`insert`/`route`/`gemfile` lines) does not name." |
| "RubyGems and GitHub reachable ... record the result" | Fine, but says nothing about the sandbox the evals need. | Add: "`bwrap` and `socat` installed, or the eval engine choice recorded." |

### A-27 (should-fix): Phase 1

| Item | Problem | Replacement |
| --- | --- | --- |
| "Every rails_intent rule cites a Tier 0 to 2 source" | Lint can check presence, not that the source says it (plumbing). | "Each source entry carries a verbatim quote; `bin/doctrine verify-quotes` finds every quote as a substring of the cited lab gem file or the fetched page snapshot under doctrine/sources/." |
| "First batch ... drafted, verified, re-derived independently, owner-reviewed." | "Re-derived independently" is not observable. | "For each rule, doctrine/rederivation/<id>.md exists with the verifier session id, its statement, and a diff against the authored statement; the owner's verdict is recorded in the record." |
| "`docs/research/` holds ... plus one per remaining area as it is mined." | Open-ended; never complete. | "One research file per rule area listed in this spec (12 files)." |
| "`bin/doctrine verify` runs verify commands against `lab/SOURCES.md` paths." | Tests the runner, not the rules; `verify` fields typically grep for a method name (plan.md example). | "Every `verify` command fails when run against a lab built from the previous minor (8.0), or the rule records why it is version-independent." |
| "Owner review time recorded per batch." | Plumbing, harmless. | Keep, but move to a metrics file, not acceptance. |

### A-28 (should-fix): Phase 2

| Item | Problem | Replacement |
| --- | --- | --- |
| "`hooks/hooks.json` wires all eight hooks with exact matchers." | "Exact" is undefined; Bash and Write bypasses are not covered (A-10). | "A test loads hooks.json and asserts each hook's matcher and `if` value against a table in the spec; red-team bypass list is empty or each entry has a test." |
| "Hooks verified in a live session against both lab apps." | Not observable. | "A committed `claude -p --output-format stream-json --include-hook-events` transcript per lab app showing each hook's event firing once with the expected decision." |
| "Block messages name a rule id and the Rails alternative." | "Rails alternative" is false for owner-preference blocks (A-16). | "Block messages name a rule id and its `kind`; tests assert both." |

### A-29 (should-fix): Phase 3

| Item | Problem | Replacement |
| --- | --- | --- |
| "Phase 4 rerun shows improvement over the doctrine-only condition per rule area." | "Improvement" is undefined and per-area noise is about plus or minus 36 points (A-32). | "Pre-registered paired test on train tasks: overall doctrine-plus-skills minus doctrine-only difference, with its interval reported; per-area deltas reported, not gated." |
| "Triggering evals pass at or above 0.9 for matches and distractors." | 20 prompts per side: 18/20 has interval 0.70 to 0.97; `paths` skills cannot trigger in an empty workspace (A-48). | "On a scaffolded lab workspace, at least 18 of 20 matching prompts invoke the skill and at most 2 of 20 distractors do; testing skills additionally: 0 of 10 RSpec-project prompts load the Minitest skill." |
| "Every decision rule maps to a rule id." | Plumbing; does not check that the skill text agrees with the record. | "Lint checks that each rule id referenced in SKILL.md exists and that the SKILL.md sentence does not contradict the record's statement (verifier pass recorded)." |

### A-30 (should-fix): Phase 4

| Item | Problem | Replacement |
| --- | --- | --- |
| "train and held-out baseline means agree within noise." | "Within noise" undefined; with about 9 held-out tasks the test has almost no power (A-33). | "Difference of baseline means lies inside the pre-registered interval computed in PREREGISTRATION.md." |
| "Owner calibration set of 20 to 30 judged items labelled once; agreement reported." | No threshold here (the 90 percent bar is only in execution-plan) and no chance correction. | "Cohen's kappa between owner and judge at least 0.7 on all 30 items, not only clear-cut ones; below that the judge prompt is reworked and a fresh set labelled." |
| "Task authoring session received only rule statements and sources, recorded in the task files' provenance field." | Self-reported. | "The authoring session id and the exact input file list are committed; the input list contains no file from doctrine/ other than rule statements." |
| "Mechanism check: score drops with rules removed; transcripts show rules and skills loading." | No magnitude; rules loading is invisible in -p transcripts unless logged. | "Doctrine minus baseline overall difference exceeds its pre-registered interval; an `InstructionsLoaded` hook log shows `path_glob_match` loads for each rules file in at least one run." |
| "Judge fails the three known negatives" | Trivial inputs (A-35). | Add: "Judge fails at least 9 of 10 plausible-but-wrong Rails diffs written by someone other than the task author." |
| "Owner spot check of ten percent of judged items recorded." | Fine, but only failures are read elsewhere (execution-plan 2.7). | "Spot check samples passes and failures in equal numbers; false-pass and false-fail counts recorded." |

### A-31 (nit): Phases 5 and 6

| Item | Problem | Replacement |
| --- | --- | --- |
| phase-5: "tools implemented with contract tests and latency budgets." | No numbers. | "p95 latency per tool under N ms recorded on the lab app." |
| phase-6: "`bin/doctrine bump` implemented and documented." | "Documented" not observable. | "`bin/doctrine bump 8.1.4` (no-op) and `bump <next>` both run in CI; README section exists." |
| phase-6: "Owner time per bump recorded." | Plumbing. | Keep as metric, not acceptance. |

---

## 5. The review protocol

### A-32 (blocks-build): noise floor and the acceptance rule

Documents and text: execution-plan 2.9: "With 30 tasks and 5 runs the overall pass-rate interval is about ±8 points; per rule area (about 3 tasks × 5 runs) about ±26 points". grill Q19 and phase-4: "every rule area above baseline".

Evidence (arithmetic, 95 percent, p = 0.5 worst case, computed with Python):
- The ±8 and ±26 figures are 1.96 × sqrt(0.25/n) with n = 150 and 15, i.e. they treat every run as an independent trial. Runs of the same task are correlated. With intra-task correlation 0.5 (design effect 3) the overall half-width is ±13.9 and per-area ±43.8; if runs were fully correlated the effective n is the task count: ±17.9 overall, ±56.6 per area.
- Decisions compare two conditions. The difference of two independent proportions has half-width √2 larger: ±11.3 overall, ±35.8 per area (n = 15), ±46.2 per area with the 3 runs phase-4 actually specifies.
- "Every area above baseline" across 12 areas, even under the plan's own independence assumption: if the doctrine truly adds 10 points in every area, P(one area observed above baseline) = 0.71 and P(all 12) = 0.016; at a true +20 points, 0.17; at +30 points, 0.54. With 3 runs: 0.007, 0.07, 0.28. A working doctrine fails the gate most of the time, and the gate rewards luck.
- "over-correction on negative tasks no worse than baseline": 5 tasks × 5 runs gives a difference half-width of ±27.7 points, so "no worse" cannot be distinguished from a large regression.
- "Reps are the cheap knob: paired designs make them go further than more tasks" (section 3). Reps only shrink within-task noise; the task-by-condition interaction shrinks only with more tasks, so when tasks vary, more tasks beat more reps.

Proposed change: pre-register a paired analysis (per-task condition differences, bootstrap over tasks), a minimum effect size, and a gate on the overall difference plus a non-inferiority margin for negatives; report per-area deltas without gating, or require "no area significantly below baseline" instead of "every area above".

### A-33 (should-fix): the held-out split

Document and text: execution-plan 2.2: "Thirty percent of tasks, drawn at random stratified by rule area, are run only at the final confirmation. ... Train and held-out means must agree within noise at baseline, or the split is redrawn."

Evidence: 30 percent of 30 is 9 tasks; with 12 areas and 2 to 3 tasks per area, stratification gives 0 or 1 held-out task per area, so area-level acceptance on the held-out set is impossible. "Redrawn" until means agree is a forking path: the author can redraw until a convenient split appears.

Proposed change: hold out whole tasks at one per area (12 tasks, 40 percent) or accept that held-out confirms only the overall number; draw once with a committed seed and never redraw.

### A-34 (should-fix): model roles reintroduce self-preference

Documents and text: execution-plan section 2 Author: "Fable 5.1 for rules and specs"; section 3 Judge: "Fable 5.1 for Opus runs on judged items only"; section 1: "Judge self-preference: a model judge prefers output that resembles its own". Verifier, adversary and the Sonnet-run judge are all "Opus 5.5".

Evidence: the judge's prompt contains "the rule text and sources" (GLOSSARY "Judge"), and the rule text is Fable's prose; for Opus runs, Fable grades conformance to its own writing. Verifier, adversary and judge share one model, so their errors are correlated. The adversary's prompt and the source set it receives are written and curated by the author session (execution-plan 5.2: "given the sources and the branch"; the sources are the author's research files).

Proposed change: never let the rule-author model judge; use a non-Claude judge for the final confirmation (already listed as an option); give the adversary a source list the author did not curate (for example the Rails guides index and CHANGELOGs, not docs/research/).

### A-35 (should-fix): grader validation controls test only easy cases

Documents and text: execution-plan 2.3: "an oracle diff built from generator evidence must pass every check, and two nulls (an empty diff, and a deliberately drifted diff that adds Devise, a service object and factories) must fail." 2.4: "The judge is fed an empty output, 'I don't know', and a confident answer to the wrong task". 2.6: "below roughly ninety percent on clear-cut items". 2.7: "Ten graded failures are read per run".

Evidence: no generator produces a publish/unpublish feature, so the oracle is hand-written by the same author who wrote the checks; passing it proves the checks accept the author's answer, not that they accept other correct answers. The nulls and judge negatives are caught by a Gemfile diff alone. Reading only failures never measures false passes. Agreement on "clear-cut" items, without chance correction, overstates calibration (27/30 has interval 0.74 to 0.97).

Proposed change: require a second correct solution written independently (alternative oracle) to pass; add subtle-drift nulls (a member action instead of a resource, fetch-and-JSON, a callback chain); add near-miss judge negatives; sample passes as well as failures; report kappa on all items.

### A-36 (should-fix): pre-registration is not tamper-evident

Document and text: execution-plan 2.1: "committed and tagged `evals-v1` before any skill is written. A change after the tag is `evals-v2` and reruns the baseline."

Evidence: git tags can be moved (`git tag -f`) and force-pushed by the same agent that benefits; there is no limit on versions, and a rerun baseline after a grader change still lets graders be tuned toward the doctrine's output once doctrine runs are seen.

Proposed change: post the tag's commit SHA in a comment on the parent issue at tagging time (GitHub timestamps it), require every version and its reason to be listed in the final report, and forbid grader changes after any non-baseline condition has run except bug fixes approved by the owner.

### A-37 (should-fix): blinding fails on transcripts

Documents and text: execution-plan 2.5: "Diffs are anonymized, order is randomized, no condition is named 'baseline' or 'reference' in the judge's input". phase-4: "Judged items use `llm` graders".

Evidence: plugin-evals.md: an `llm` grader with `focus: trace` sees the session, and the with-arm session contains `Skill` tool calls, hook block messages and rule-file reads, which name the condition. The built-in `llm` grader also runs per run inside the engine, so the project's anonymization and order randomization cannot be applied to it.

Proposed change: judge only post-hoc workspace diffs (`git diff` against the lab commit) in a separate script that strips paths under `.claude/` and shuffles; do not use built-in `llm` graders for judged items.

### A-38 (should-fix): the eval measures compliance with the doctrine, not Rails intent

Documents and text: phase-4 checklist: "Task authoring session received only rule statements and sources". Q1: the distribution "makes Claude Code's defaults match Rails intent, with evidence."

Evidence: every task, check and judged item is derived from the rule statements, so a wrong rule produces a task that rewards the wrong behaviour (A-14 is an instance). The held-out split protects against overfitting skills, not against wrong rules. execution-plan section 1 names "Grader fit"; the protocol has no control for it at the rule level.

Proposed change: state in the README and report that efficacy evals measure doctrine compliance and that Rails-intent validity rests on Evaluation A alone; add an external-validity slice: tasks written from a Rails guide or generator diff by a session that has never seen the doctrine.

### A-39 (should-fix): cost estimate gaps

Document and text: execution-plan section 3: "Thirty tasks × five runs × four conditions is six hundred runs, so iteration on Sonnet 5.5 lands in the low thousands of dollars for a full matrix, and the final Opus confirmation on the held-out set at three runs is a few hundred."

Evidence: 600 × $1 to $3 is $600 to $1,800 per matrix, so "low thousands" assumes about two matrices; say so. Omitted: (a) phase-3 triggering evals: 12 skills × 40 prompts × 3 runs = 1,440 agent runs, more than a full matrix; (b) plugin-evals.md: "a suite makes roughly cases × runs agent runs with the plugin and the same number again for the no-plugin baseline", so three plugin variants each run their own baseline unless `--ablation none`; (c) "three short judge calls per `llm` or `baseline` grader per run"; (d) the Sonnet half of "Opus 5.5, plus Sonnet 5.5" final confirmation. The held-out Opus figure (9 tasks × 3 runs × 4 conditions × $2 to $6 = $216 to $648) holds.

Proposed change: a cost table per phase with these lines, and `--max-cost-usd` / `--max-budget-usd` set from the owner's ceiling.

### A-40 (should-fix): adversary independence and recall

Documents and text: execution-plan 2 Adversary: "graded by findings the owner upholds"; ADR-0007: "only structural separation, owner calibration and generator evidence provide independent signal".

Evidence: scoring by upheld findings rewards volume and never measures misses. This review's own prompt was written by the author session and enumerates what to check; an adversary that only answers the author's questions is not structurally independent of the author.

Proposed change: seed each artifact with 3 to 5 known defects before the adversary pass (recorded privately by the owner) and report the adversary's recall; let the adversary prompt be a fixed template, not authored per artifact by the author.

### A-41 (should-fix): baseline contamination in the fallback runner

Documents and text: grill Q16 fallback: "a runner over `claude -p --output-format json` with per-condition settings files". docs/agents/skills.md: "Install once per machine: `claude plugin install mattpocock-skills@claude-plugins-official`."

Evidence: headless.md: "Without it [`--bare`], `claude -p` discovers and loads them as an interactive session would, including anything configured in the working directory or `~/.claude`." The machine that runs evals has the author's user plugins and any `~/.claude/CLAUDE.md`, which would load into the baseline.

Proposed change: run every condition with a dedicated empty `HOME` (or `CLAUDE_CONFIG_DIR`) and `--setting-sources project,local`; record `claude --version` and the loaded plugin list per run.

---

## 6. Missing work

### A-42 (should-fix): install, update and plugin packaging have no ticket

Documents and text: grill Q1: "Done means: `bin/doctrine install` into a fresh `rails new` app yields CLAUDE.md, rules files, hooks and skills". ADR-0006: "written and re-rendered by `bin/doctrine install`".

Evidence: `grep -ri "doctrine install\|doctrine update\|plugin.json\|plugin validate"` over docs/specs returns nothing. No checklist item builds the installer, the update path, the plugin manifest, `claude plugin validate`, a marketplace entry, or the RSpec-or-Minitest variant selection at install time.

Proposed change: add these to Phase 1 (installer, renderer target) and Phase 2 (manifest, validate) with an end-to-end checklist item: "`bin/doctrine install` on a fresh `rails new` app, then `claude plugin validate`, then one eval task passes".

### A-43 (should-fix): JavaScript dependencies are uncontrolled

Documents and text: ADR-0001: "uses the gems a fresh `rails new` produces ... and nothing else". phase-2 allowlist covers Gemfile and `bundle add`.

Evidence: the Rails JS guide (8-1-stable) documents `bin/importmap pin react react-dom`; `-j esbuild` apps use package.json. Drift toward npm packages passes every hook and the `no_new_gems` check.

Proposed change: a pin and package allowlist (initially empty beyond what `rails new` pins) and a check on `config/importmap.rb` and package.json diffs.

### A-44 (should-fix): eval workspace provisioning

Document and text: phase-4: "First ticket verifies that kept workspaces support `bin/rails routes` and the suite".

Evidence (plugin-evals.md): "Each run starts in an empty workspace"; scaffold scripts run "only when you pass `--scaffold`, with a minimal environment and a 120-second limit"; under Bash grant "your home directory and Claude Code configuration are unreadable, and network access is limited to domains you grant"; Linux needs bubblewrap and socat (absent here). Nothing plans where the lab's installed gems live so a sandboxed `bin/rails` can read them, or whether rubygems.org is reachable (which decides whether "gem additions" measure the doctrine or the sandbox).

Proposed change: a provisioning ticket: copy the lab app per run, vendor gems inside the workspace or a readable path, install bubblewrap/socat, and fix one network policy for all conditions.

### A-45 (should-fix): mining rejected PRs and the version delta

Documents and text: grill Q20: "CHANGELOG entries name their PR numbers, which drives the fetch list." ADR-0002: "Rejected core-team PRs are first-class evidence and are collected in the declined list." phase-6: "The version delta is regenerated from the per-framework CHANGELOGs of the clone".

Evidence: rejected PRs never reach a CHANGELOG or git history, so a CHANGELOG-driven fetch list cannot find them; github.com PR pages returned 403 from this session and research section 7 notes fetches return only 125-character fragments. `activerecord/CHANGELOG.md` on 8-1-stable ends "Please check [8-0-stable] ... for previous changes", so a 7.0 to 8.1 delta needs the 7-0 through 8-1 stable branches, not one clone of 8-1-stable.

Proposed change: specify the declined-list source (owner-supplied PR list, or search pages if reachable) and its fallback; the delta script reads each `X-Y-stable` CHANGELOG.

### A-46 (should-fix): lab determinism ignores floating gems

Document and text: execution-plan section 4 (#2): "a second regeneration must match the first except timestamps and keys."

Evidence: `app_base.rb` uses `GemfileEntry.floats` for propshaft, importmap-rails, turbo-rails, stimulus-rails and others (no version constraint), so a later `rails new` resolves newer gems and different generator output.

Proposed change: commit the lockfile from the first generation and regenerate with `BUNDLE_FROZEN` from it, or accept and diff gem versions explicitly.

### A-47 (should-fix): path triggers miss where the code is written

Document and text: phase-1: "queries (`app/models/**`, `app/jobs/**`)", "forms (`app/views/**`)".

Evidence: memory.md: "A path-scoped rule loads when Claude uses the Read, Write, or Edit tool on a matching file." N+1 queries are triggered by iterating associations in views and loaded in controllers (research: Query Interface guide "Retrieving a list of records N ... can sometimes trigger N extra queries"); form objects (`ActiveModel::API`) live in `app/models` and strong params in controllers.

Proposed change: add `app/views/**` and `app/controllers/**` to queries; add `app/models/**` and `app/controllers/**` to forms; test loading with an `InstructionsLoaded` log.

### A-48 (should-fix): triggering evals cannot exercise `paths` skills

Document and text: phase-3: "`paths` frontmatter narrows file-triggered skills (ruby-34-idioms on `**/*.rb`, testing skills on `spec/**` or `test/**`)"; "twenty matching prompts and twenty distractors per skill through `claude plugin eval`".

Evidence: skills.md: "When set, Claude loads the skill automatically only when working with files matching the patterns." plugin-evals.md: "Each run starts in an empty workspace". User story 3 ("only the RSpec testing skill to load") has no checklist item.

Proposed change: run triggering cases on a scaffolded lab copy; add the cross-variant negative to the checklist (see A-29).

### A-49 (nit): licensing of excerpts

Documents and text: phase-3: "references excerpted from Tier 0 to 2 sources". LICENSE: "MIT License".

Evidence: guides.rubyonrails.org footer: "This work is licensed under a Creative Commons Attribution-ShareAlike 4.0 International License".

Proposed change: keep guide excerpts in a separately licensed references directory with attribution, or link instead of copying.

---

## 7. What I could not fault

Checked against primary sources and found sound:
- Rails 8.1.4 is the latest release, published 2026-09-24 (rubygems.org API; rubyonrails.org post). 7.2.4 is "the last release of the 7.2.x series" (rubyonrails.org/2026/9/24/Rails-Version-7-2-4-has-been-released).
- `rails.gemspec` on 8-1-stable: `s.required_ruby_version = ">= 3.2.0"`; `RAILS_VERSION` is `8.1.4`.
- Rails 8.2 draft notes on main contain `Rails.app`, `Rails.app.creds` and `:header_only` "(default for new 8.2 apps)".
- ruby-lang.org: Ruby 4.0.9 and 3.4.12 are the stable releases; rbenv here offers 3.4.10 and 4.0.6. Ruby 3.4 NEWS: "`it` is added", "The default parser is now Prism", chilled-string deprecation warning. Ruby 4.0 NEWS: ZJIT, `Ruby::Box`, "`Set` is now a core class".
- The railties scaffold template (8-1-stable) uses `params.expect`, `UNPROCESSABLE_CONTENT` and `status: :see_other`, with no `respond_to` (true of the railties file; see A-01 for the default app).
- `config/ci.rb.tt` steps: setup, RuboCop, bundler-audit, importmap audit, Brakeman, `bin/rails test`, seeds, optional `gh signoff` (A-04 concerns `--skip-test` only).
- rspec-rails 8.0.4 released 2026-03-10/11, requires actionpack/railties `>= 7.2`; Rails 8.2 fixes are under "Development" in the Changelog. The generated `rails_helper.rb` sets `fixture_paths`, `use_transactional_fixtures = true` and leaves `infer_spec_type_from_file_location!` commented; `config.global_fixtures` is honoured by `fixture_support.rb`; `have_enqueued_job`, `have_enqueued_mail` and `have_broadcasted_to` exist.
- Claude Code docs: CLAUDE.md "target under 200 lines"; `.claude/rules/` with `paths` loading on Read/Write/Edit; imports "don't reduce its context cost". Hooks: exit 2 blocks with stderr as reason, `permissionDecision: "deny"`, SessionStart plain stdout added as context, PostToolUse `additionalContext`, `Edit|Write` exact-name matchers, `${CLAUDE_PROJECT_DIR}` and `${CLAUDE_PLUGIN_ROOT}`. plugin eval: three runs by default, no-plugin arm with delta, `--judge-model`, two-of-three `llm` votes, the six grader types, "There are no custom-code graders", `--keep-temp`, JSON and HTML reports. The `claude -p --output-format json` fallback with `--settings`, `--setting-sources` and `--plugin-dir` is viable.
- Model prices in the execution-plan header match the claude-api reference cached 2026-10-06; the per-run Opus-to-Sonnet cost ratio (2x) matches the price ratio; 30 × 5 × 4 = 600.
- Security guide: "query the user's access rights, too" with `@project = @current_user.projects.find(params[:id])` (ADR-0004's Tier 1 source).
- ADR-0004's practice claim holds at Tier 3: basecamp/fizzy `cards_controller.rb` uses `Current.user.boards.find`, `Current.user.accessible_cards.find_by!` and `head :forbidden unless Current.user.can_administer_card?(@card)`; once-campfire's `Authorization` concern is `head :forbidden unless Current.user.can_administer?`.
- Rails Doctrine quotes used in the research ("Substitutions are still possible, but not required", "proved to be the practical way", "SQL for the occasional complicated query", "rare rather than common fit") are verbatim on rubyonrails.org/doctrine.
- `active_record/CHANGELOG.md` scoping, the migrations guide text and the Callbacks guide text quoted above are from 8-1-stable sources.
- Docker: the client is installed but no daemon socket exists, so "Docker is unavailable" is accurate in effect.
- Tracker: issues #1 to #8 exist, all labelled `needs-triage` and `phase`, #2 to #8 are sub-issues of #1, matching README.

Not verified (no access or out of scope): the description of rails/rails PR #43945 and other PR pages (github.com returned 403); the `writing-for-agents` reference cited in grill Q8; `gh` 2.94 `--add-sub-issue` (installed `gh` is 2.89.0); whether `bin/rails test` exits 0 in a `--skip-test` app (A-24).
