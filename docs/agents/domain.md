# Domain Docs

Single-context repo: one `GLOSSARY.md` at the root and ADRs under `docs/adr/`.

## Before exploring, read these

- `GLOSSARY.md` at the repo root.
- `docs/adr/`: read the ADRs that touch the area you are about to work in.

## Use the glossary's vocabulary

When your output names a domain concept (an issue title, a rule, a spec, a test name), use the term as defined in `GLOSSARY.md` and avoid the synonyms it lists under _Avoid_. A missing term is a signal: either you are inventing language the project does not use, or there is a gap for `/domain-modeling`.

## Flag ADR conflicts

If your output contradicts an ADR, say so explicitly rather than silently overriding:

> _Contradicts ADR-0004 (authorization by scoping), but worth reopening because…_

## Rails vocabulary

Rails's own terms (resource, concern, callback, scope, fixture, Turbo Frame, Turbo Stream) are used in their Rails Guides sense and are not redefined in the glossary.
