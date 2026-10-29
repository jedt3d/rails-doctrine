# Rails Doctrine

A distribution of rules, skills, hooks and evaluations that makes Claude Code's default choices in a Rails application match the Rails core team's design intent, and proves it.

## Language

### The doctrine

**Doctrine**:
The complete set of rules, with their reasons and sources, that Claude follows in a Rails project.
_Avoid_: style guide, guidelines, conventions doc

**Rule**:
One statement of what to do in a situation, carrying the Rails reason, a source tier, a version range and a verification check.
_Avoid_: guideline, lint, cop

**Rails intent**:
A design decision the Rails core team made deliberately, evidenced by a Tier 0 to Tier 2 source.
_Avoid_: best practice, community convention

**Owner preference**:
A rule chosen by the repository owner where Rails is silent or where the owner deviates knowingly.
_Avoid_: house style, our taste

**Tier**:
The rank of a source's authority for establishing Rails intent, from Tier 0 (Rails source and generators) to Tier 4 (no source).
_Avoid_: level, grade

**Declined**:
A feature or pattern the Rails core team explicitly chose not to adopt, recorded with its stated reason.
_Avoid_: anti-pattern, banned, forbidden

**Omakase**:
The stack and gems a fresh `rails new` produces for the pinned version, accepted without debate.
_Avoid_: default stack, vanilla

**Pinned version**:
The exact Rails minor and Ruby minor a project commits to, against which every rule is verified.
_Avoid_: target version, supported version

**Version delta**:
The list of APIs added, renamed, deprecated or removed between Rails 7.0 and the pinned version.
_Avoid_: upgrade notes, changelog summary

### Behaviour and measurement

**Drift**:
Claude's regression toward community-popular choices when a prompt is silent on stack or architecture.
_Avoid_: hallucination, bias, bad defaults

**Silent prompt**:
An eval prompt that states a user need without naming any library, layer or pattern, so drift can show.
_Avoid_: neutral prompt, vague prompt

**Eval task**:
One silent prompt with its rule ids, mechanical checks and judged items.
_Avoid_: test case, scenario, benchmark

**Negative task**:
An eval task where Rails intent is the opposite of the doctrine's usual answer, used to detect over-correction.
_Avoid_: counter-example, trick question

**Condition**:
A configuration under which the eval set runs: baseline, doctrine, doctrine plus skills, plus hooks, plus MCP.
_Avoid_: variant, arm, treatment

**Mechanical check**:
A shell command whose exit code grades one aspect of an eval task output.
_Avoid_: assertion, lint check

**Judge**:
A separate model run that grades judged items with the rule text and sources in its prompt and a quote as evidence.
_Avoid_: grader, reviewer, LLM-as-judge

**Over-correction**:
Following a rule where Rails intent calls for its exception.
_Avoid_: dogma, rigidity

### Artifacts

**Lab**:
The reference Rails applications generated from the pinned version with generator output committed untouched.
_Avoid_: sample app, demo, fixture app

**Generator evidence**:
Code produced by `rails new` or `rails generate` for the pinned version, the strongest Tier 0 source.
_Avoid_: scaffold, boilerplate

**Hook**:
A script Claude Code runs on a tool event that blocks, injects context or reports, enforcing a rule deterministically.
_Avoid_: guard, linter, plugin

**Allowlist**:
The gems a project may depend on without a doctrine change.
_Avoid_: whitelist, approved gems

**Skill**:
An on-demand instruction file Claude loads when a task matches its description.
_Avoid_: prompt, playbook

**Rule record**:
The machine-readable form of a rule from which the human page and the verification run are generated.
_Avoid_: rule file, YAML entry
