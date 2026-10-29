# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues on `jedt3d/rails-doctrine`. Use the `gh` CLI for all operations.

## Conventions

- **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line bodies.
- **Read an issue**: `gh issue view <number> --json number,title,body,labels,comments`.
- **List issues**: `gh api repos/jedt3d/rails-doctrine/issues?state=open` (the GraphQL-backed `gh issue list` is not available from Claude Code cloud sessions; REST works).
- **Sub-issues**: `gh api --method POST repos/jedt3d/rails-doctrine/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>` where the id is the child's numeric database id (`gh api repos/jedt3d/rails-doctrine/issues/<n> --jq .id`). `gh` 2.94+ also accepts `gh issue edit <parent> --add-sub-issue <child>`.
- **Blocking**: native issue dependencies. `gh api --method POST repos/jedt3d/rails-doctrine/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`. Fallback: a `Blocked by: #<n>` line at the top of the child body. A ticket is unblocked when every blocker is closed.
- **Count open blockers**: `gh api repos/jedt3d/rails-doctrine/issues/<n> --jq .issue_dependencies_summary.blocked_by`.
- **Comment**: `gh issue comment <number> --body "..."`.
- **Labels**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`.
- **Close**: comment the explanation, then `gh issue close <number>`.

## Permission to implement

Implementation of any phase waits for the owner. A phase issue labeled `needs-triage` is a proposal under review. The owner grants permission by relabeling it `ready-for-agent`. An agent may start implementation only on a `ready-for-agent` issue whose blockers are all closed.

## Pull requests as a triage surface

**PRs as a request surface: no.**

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Read it as in **Read an issue** above.
