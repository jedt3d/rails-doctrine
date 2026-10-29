---
status: accepted
---
# Authorization by scoping through the current user, not a policy layer

Projects authorize by loading records through the current user's associations (`Current.user.posts.find(...)`), by plain predicate methods on models, and by `before_action` guards, with no policy objects or policy gem. Devise and Pundit are out: authentication comes from the Rails 8 generator (User, Session, `Current.session`, the `Authentication` concern, `allow_unauthenticated_access`, rate limiting) and is extended rather than replaced. We chose this because it is the pattern the Rails generator and the Rails core team's own applications use, and because a policy layer is a second place to look for rules that models already hold.

## Sources

- Rails Security guide, authorization section: "query the user's access rights, too: `@project = @current_user.projects.find(params[:id])`" (Tier 1).
- rails/rails issue #50446 (DHH, Dec 2023) stating the generator's intent, and PR #52328 (merged 2024-07-16) that shipped it (Tier 2). Quotes in `docs/research/mvc-active-record-decisions.md`, section 4.

## Consequences

- A record a user may not touch is simply not found, so most authorization is implicit in the association chain and tested through request specs.
- Introducing a policy layer later is a doctrine change recorded in a new ADR, with the project that demonstrated the need.
