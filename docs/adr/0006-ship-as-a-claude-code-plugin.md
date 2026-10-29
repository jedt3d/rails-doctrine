---
status: accepted
---
# Skills and hooks ship as a Claude Code plugin; rules files are installed per project

Skills and hooks are packaged as a Claude Code plugin (manifest plus `hooks/hooks.json`) so they update in place across projects and so `claude plugin eval`, which only targets plugins, can measure them. CLAUDE.md, `.claude/rules/*.md` and the gem allowlist are per-project files written and re-rendered by `bin/doctrine install` from the pinned rule records. The alternative, copying `.claude/` by hand into each application, was rejected because it has no update path and no eval path. Accepted by the owner on 2026-10-29 (grill Q9).
