---
status: accepted
---
# Author, verifier, adversary and judge are separate sessions with different inputs

The agent that writes a rule, spec or skill never verifies, reviews or grades it. A verifier re-derives from sources alone, an adversary hunts for errors with a source quote per finding, and a judge grades anonymized end states with mechanical checks ranked above it. The eval set is pre-registered and tagged before skills exist, with a held-out split touched once. We chose this over "the author double-checks" because all our model roles share training and the author's context, so only structural separation, owner calibration and generator evidence provide independent signal. The protocol is in `docs/execution-plan.md`, section 2.

## Consequences

- More sessions per artifact and a higher token bill per rule; the trade is that a wrong rule is caught before it shapes a hundred projects.
- The owner's time is spent on findings and disagreements, not on reading drafts.
