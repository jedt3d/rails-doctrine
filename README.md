# rails-doctrine

How to set up Claude Code to be a Rails expert with the right taste: a distribution of rules, skills, hooks and evaluations that makes Claude's default choices in a Rails 8.1 application match the Rails core team's design intent, and measures it.

Status: planning. Implementation of each phase waits for the owner's permission (an issue labeled `ready-for-agent`).

## Read in this order

1. `GLOSSARY.md`: the vocabulary.
2. `docs/decisions.md`: what has been decided and why.
3. `docs/adr/`: the decisions that are hard to reverse.
4. `docs/grill/2026-10-29-doctrine-plan.md`: the plan stress-tested question by question, with sources.
5. `docs/plan.md` and `docs/specs/phase-*.md`: the phases, each with a checklist.
6. `docs/research/`: cited primary-source research behind the rules.
7. `docs/agents/`: how agents work in this repo (tracker, labels, domain docs, which Matt Pocock skills we use).

## Scope

Rails 8.1.x and the gems `rails new` produces, plus rspec-rails. Ruby 3.4.x. Nothing else until the doctrine says otherwise.

## Tracker

Parent: [jedt3d/rails-doctrine#1](https://github.com/jedt3d/rails-doctrine/issues/1). Phases: #2 lab, #3 doctrine, #4 hooks, #5 skills, #6 evals, #7 MCP (conditional), #8 maintenance. Blocking edges are native issue dependencies; the owner permits implementation by relabeling a phase `ready-for-agent`.
