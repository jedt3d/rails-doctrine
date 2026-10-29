---
status: accepted
---
# Evals are built before skills, and the MCP server is gated by eval results

The behavioral eval harness and a baseline run come before any skill is written, so every skill is measured against something, and an MCP server is built only if the evals show failures from stale Rails API or verification avoided because of boot time. The obvious order (write the helpful material first, measure later) was rejected because it leaves no way to tell style change from intent change, and because an MCP server is the most expensive artifact with the least evidence that it is the bottleneck.

## Consequences

- Phase 4 (evals) runs before Phase 3 (skills) despite the numbering, which is kept for continuity with the plan.
- A rule with no eval exercising it is reported as untested in every run.
