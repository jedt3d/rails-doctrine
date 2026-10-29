# Rails doctrine: decisions so far

Format for every rule: Rails does X because Y. We do X. Exceptions: Z.
This file is the seed of the project CLAUDE.md. Keep reasons; they are what
let Claude generalize to cases not listed here.

## 1. Versions

- Ruby 3.4.x only. Not Ruby 4.0 yet.
  Why: small, stable language surface. Ruby 3.4 gives the `it` block
  parameter, Prism as default parser, and chilled-string warnings that
  prepare code for frozen literals. Ruby 4.0 features (ZJIT, Ruby::Box,
  core Set) are out of scope until revisited.
- Rails 8.1.4 (latest stable as of 2026-10-29; Rails 8.2 is unreleased, draft
  notes on main only). Rails 8.1 requires Ruby >= 3.2. Pin `~> 8.1.4`.
- Ruby 3.4.12 in `.ruby-version` (3.4.10 is what the cloud container's rbenv
  offers today; record the installed patch in lab/SOURCES.md).

## 2. Stack: omakase, no third-party gems by default

- Why: fewer compatibility breaks on Rails upgrades, one set of
  conventions to learn, and it matches what `rails new` generates.
- Allowed because `rails new` 8.1 generates them (source: Gemfile.tt and
  app_base.rb on 8-1-stable): rails, propshaft, sqlite3 | pg | mysql2 | trilogy,
  puma, importmap-rails, turbo-rails, stimulus-rails, jbuilder (commented out
  unless --api), bcrypt (commented out), tzinfo-data, solid_cache, solid_queue,
  solid_cable, bootsnap, kamal, thruster, image_processing, rack-cors (API only),
  debug, bundler-audit, brakeman, rubocop-rails-omakase, web-console, capybara,
  selenium-webdriver, and jsbundling-rails / cssbundling-rails when -j / -c are
  used. The allowlist file is generated from the template, not hand-written.
- A PreToolUse hook rejects Gemfile edits and `bundle add` outside this
  allowlist. Adding a gem is a doctrine change, not a coding decision.

## 3. Testing: RSpec is the one approved exception

Decision: rspec-rails is allowed (ADR-0001). Nothing else from the RSpec
ecosystem is. rspec-rails 8.0.4 (2026-03) supports Rails >= 7.2; its Rails 8.2
fixes are unreleased as of 2026-10, which is the upgrade lag we accept.

Default per project type:
- Applications: RSpec (owner preference; cost is bounded, see guide).
- Engines, gems, anything that may be extracted or upstreamed: Minitest,
  because Rails itself and `rails plugin new` use it, and contributors
  expect it.
- Never both in one codebase. Decide at `rails new` time
  (`rails new app --skip-test`, then rspec-rails and `rails g rspec:install`).

Decision questions at project start (any "yes" tips toward Minitest):
1. Must this upgrade to a new Rails major on release week?
2. Is it an engine, gem, or code that may be contributed back to Rails?
3. Does it need parallel tests with zero extra gems?
4. Will humans unfamiliar with RSpec maintain it?
Otherwise RSpec.

Rules when using RSpec (these keep the Rails intent intact):
- Fixtures, not factories. The generated rails_helper already sets
  `config.fixture_paths = [Rails.root.join('spec/fixtures')]` and transactional
  fixtures; we add `config.global_fixtures = :all` (owner preference). No FactoryBot.
- Request specs, not controller specs. System specs only for flows that
  need a browser. No rails-controller-testing.
- Real objects over mocks. No mocking Active Record models. No
  `allow_any_instance_of`.
- Rails helpers through rspec-rails matchers: have_enqueued_job,
  have_enqueued_mail, have_broadcasted_to; Rails assertions
  (assert_difference, assert_changes) are available and fine.
- No shoulda-matchers, no database_cleaner (transactional fixtures already
  handle it), no parallel_tests. If parallelism becomes necessary, that is
  a signal to reconsider Minitest for that project.
- Keep rails_helper.rb close to generated. Spec dirs mirror app dirs.
- `describe` the behavior, `context` the state, `it` the outcome. Max two
  levels of context; deeper nesting means the model needs a smaller method.
- `bin/ci` (Rails 8.1) runs `bundle exec rspec` in place of `rails test`.

Rules when using Minitest:
- `test "does X" do` declarative style, fixtures, `parallelize` on,
  integration tests over controller tests, system tests sparingly.
- No minitest/spec DSL; the Rails-blessed readable form is `test "..."`.

## 4. Authentication and authorization

- Devise and Pundit are out for now.
- Authentication: `bin/rails generate authentication` (Rails 8). It gives
  User with has_secure_password, a Session model, Current.session,
  the Authentication concern, allow_unauthenticated_access, password
  reset mailer, and rate limiting. Extend it; do not replace it.
- Authorization, the Rails way: scope queries through the current user
  (`Current.user.posts.find(params[:id])`) so most authorization is
  implicit, plus plain predicate methods on models (`user.admin?`) and
  before_action guards for the rest. A policy layer is added only when a
  project shows real need, and that is a doctrine change.

## 5. Why the framework is shaped this way (summary for Claude)

- MVC in Rails means: model owns the domain, controller is a thin HTTP
  adapter, view is a template. The directory conventions are what make
  convention over configuration possible.
- Active Record over Data Mapper because most web apps map objects to
  tables one-to-one, Ruby reflects schema at load time so no mapping is
  needed, and one class per concept is easier to read. Cost: persistence
  coupled to domain, callbacks easy to abuse, N+1 by default. Rails answers
  with sharp knives (ActiveModel POROs, delegated_type, strict_loading),
  not with a mapper layer.
- Own JavaScript (Turbo, Stimulus, importmap) because one full-stack team
  shipping one app is the goal, and a JSON API plus client framework
  rebuilds routing, templates and validation in a second language.
- Default to native views and Hotwire. Client-side components only inside
  a Turbo Frame where the interaction model demands client state. API plus
  separate frontend only when non-browser clients or a separate frontend
  team need the API anyway.

## Revisions after the 2026-10-29 grill

- Rules are delivered as CLAUDE.md (under 200 lines) plus path-scoped
  `.claude/rules/*.md` files, not one dense CLAUDE.md (grill Q6).
- Rules are phrased positively; the declined list is disclosed reference (Q8).
- Skills and hooks ship as a Claude Code plugin; CLAUDE.md and rules files are
  written by `bin/doctrine install` (Q9, owner to confirm).
- Evals run on `claude plugin eval` with post-hoc mechanical checks (Q16).
- Rails 8.1 scaffold controllers are HTML-only with `params.expect`,
  `:unprocessable_content` and `:see_other`; rules cite this as Tier 0.

## Open items

- Pin Rails minor and record generator output for scaffold and
  authentication as skill references.
- Write the Gemfile allowlist hook and rubocop PostToolUse hook.
- Build the eval set (15 to 30 prompts) before any MCP work.
