---
status: accepted
---
# Omakase stack, with rspec-rails as the only exception

Every project built with this doctrine uses the gems a fresh `rails new` produces for the pinned version, and nothing else, so Rails upgrades break less and Claude has one convention set to learn. The owner prefers RSpec's readability, so rspec-rails is the single approved third-party gem for applications; engines and shared code stay on Minitest. The ecosystem that usually follows RSpec (FactoryBot, shoulda-matchers, database_cleaner, parallel_tests) stays out, because each one replaces a Rails facility (fixtures, Rails assertions, transactional tests, built-in parallelization).

## Consequences

- rspec-rails lags Rails releases: as of 2026-10 its latest release (8.0.4, 2026-03) carries Rails 8.2 fixes only in unreleased changes. Projects on RSpec accept upgrade lag at Rails minors.
- RSpec projects lose built-in parallel tests. If parallelism becomes necessary, the project is a Minitest candidate.
- Adding any other gem is a doctrine change, enforced by the gem allowlist hook.
