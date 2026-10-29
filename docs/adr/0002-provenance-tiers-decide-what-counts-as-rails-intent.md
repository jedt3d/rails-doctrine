---
status: accepted
---
# Provenance tiers decide what counts as Rails intent

A rule may claim Rails intent only when it cites a Tier 0 to Tier 2 source: Rails source and generators (0), guides, API docs, changelogs and the Rails Doctrine (1), or core-team PR descriptions, review comments, rubyonrails.org posts and keynotes (2). Core authors writing outside the org (3) can support a rule but not establish it. A rule with no source (4) is labeled owner preference. We chose this over "Claude's best judgment" because the drafting model shares the biases the doctrine exists to correct, and a citation requirement is the cheapest control that survives a long session.

## Consequences

- Rule writing is slower; early-era decisions (MVC, Active Record) cite books and interviews and say so.
- Rules can be downgraded mechanically when a verify command fails on a new Rails version.
- Rejected core-team PRs are first-class evidence and are collected in the declined list.
