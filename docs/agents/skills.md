# Matt Pocock skills: which ones this repo uses, and why

Install once per machine: `claude plugin install mattpocock-skills@claude-plugins-official`. The plugin ships every promoted skill; this file says which ones this repo's workflow relies on, so nobody has to remember the whole catalogue.

| Skill | Used for | Why it fits this repo |
| --- | --- | --- |
| `/grill-with-docs` | Every planning session: a plan, a phase spec, a rule area | Grilling resolves decisions in rounds; domain-modeling writes `GLOSSARY.md` and ADRs as they crystallise. Rules written without a glossary drift into synonyms. |
| `research` (model-invoked) | Source mining for rules | Its contract is primary sources only with a citation per claim, which is exactly the provenance gate in ADR-0002. Output lands in `docs/research/`. |
| `/to-spec` | Turning a grilled phase into a spec issue | Produces the Problem, Solution, Decisions, Testing Decisions, Out of Scope shape every phase issue uses. |
| `/to-tickets` | Splitting a phase spec into tracer-bullet tickets with native blocking edges | Each ticket cuts through rule record, verification and eval, not one layer at a time. |
| `writing-for-agents` (model-invoked) | Writing `CLAUDE.md`, `.claude/rules/`, every `SKILL.md` | Its rules on context pointers, positive phrasing and progressive disclosure decide how a rule is worded. See "Conventions it imposes" below. |
| `tdd` (model-invoked) | Implementing hooks, the eval runner, the rule verifier | These are small Ruby programs with crisp seams (JSON in, exit code out). |
| `code-review` (model-invoked) | Before every PR in this repo | Reviews the diff against the originating spec. |
| `/retro` | After an implementation session | Feeds the maintenance loop with environment improvements. |

Not used, and why:

- `/wayfinder`: the phases are already charted; a map would duplicate the plan. Revisit if a phase turns out to be larger than one session can hold.
- `/triage`: a solo repo has no inbound issue stream. Its label vocabulary is kept because `to-spec` and `to-tickets` apply it, and because `ready-for-agent` doubles as the owner's implementation permission.
- `/implement-spec`: parallel subagents across tickets are premature until the first phase has been implemented once by hand.

## Conventions it imposes on the doctrine

- **Positive rules.** A rule names the behaviour to produce, not the one to avoid; the declined list is disclosed reference reached by a pointer, never always-loaded text.
- **One meaning, one place.** A rule's reason lives in its rule record; `CLAUDE.md`, rules files and skills point at it rather than restating it.
- **Pointers do the triggering.** A skill's `description` and a rules file's `paths` carry the trigger branches; wording there is tuned before material is inlined.
- **Completion criteria are checkable.** Every spec checklist item is something a reviewer can observe, not "understood" or "considered".

## Deviation recorded

The owner asked that the first grill of the plan be answered by the agent itself, with researched sources, as a test of its understanding of Rails and Ruby. The grilling skill expects a human to answer. The transcript in `docs/grill/` marks every answer as the agent's recommendation and marks the decisions only the owner can make as `OWNER DECISION`, so the deviation is visible and reversible.
