---
name: personal-skills-location
description: Personal skills in ~/.agents/skills/ are NOT loaded by Claude Code — they need to be installed as plugins to appear in sessions
type: reference
---

Skills in `~/.agents/skills/` (architecture-patterns, django-patterns, find-skills, python-background-jobs, python-design-patterns, python-project-structure, refactoring-expert, supabase-postgres-best-practices) are not automatically available in Claude Code sessions. They need to be installed as plugins via the CLI to appear in autocomplete and be invocable. Project skills in `.claude/skills/` and plugin skills in `enabledPlugins` are the two recognized sources.
