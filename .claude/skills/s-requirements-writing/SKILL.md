---
name: s-requirements-writing
description: Use when starting new work, picking up an existing task, or when requirements need to be captured or updated in GitHub Issues
---

# GitHub Issues for Work Tracking

## Overview

Track all non-trivial work in GitHub Issues. Issues serve as the source of truth for what's being built and why — write them before starting, update them during implementation, and keep them current as requirements evolve.

## When to Use

- Starting new work that spans more than a single quick fix
- User mentions a task, feature, or bug that doesn't have an issue yet
- Picking up work that may already have an issue
- Requirements changed during implementation and the issue is stale

## Before Work Starts

Before starting any new work, ask the user if a GitHub issue should be created for the task.

### Writing Requirements

Create the issue with two sections in the body:

```markdown
## Functional

What is being built and why. Describe the user-facing behavior, acceptance criteria, and edge cases.

## Technical

How it should be built. Reference specific files, modules, APIs, data models, or architectural decisions.
```

### Breaking Down Tasks

For tasks with multiple phases, add subtask checkboxes. Each subtask should map to one PR. Keep subtasks specific — include what is being built and where (backend/frontend), not just a vague label.

**Backend first.** When a feature spans backend and frontend, the backend subtask should come first. The Django backend and Next.js frontend are deployed independently, so backend changes (models, migrations, Celery tasks) should land before the frontend consumes them.

```markdown
## Subtasks

- [ ] Backend: Add new scraper for CinePlex theaters with HTML snapshot tests
- [ ] Frontend: Build theater detail page showing showtimes grouped by movie
```

**If a task is very large** (more than 3-4 subtasks, or subtasks that are themselves complex), break it into separate linked issues instead. Each issue gets its own functional/technical sections and its own PR. Link them with "Part of #N" in the body.

**Planning complex tasks.** For tasks that need a detailed implementation plan beyond what fits in the issue's Technical section, **REQUIRED SUB-SKILL:** Use superpowers:writing-plans to create a step-by-step plan after the issue is written.

### Creating the Issue

```bash
gh issue create --title "Title" --body "$(cat <<'EOF'
## Functional

...

## Technical

...

## Subtasks

- [ ] ...
EOF
)"
```

## During Work

### Loading Context

When the user asks to start work on something, ask if there's an existing GitHub issue for it. If so, read the issue first:

```bash
gh issue view <NUMBER>
```

### Updating Issues

As work progresses, update the issue with:
- Status updates (what's done, what's in progress, what's blocked)
- Decisions made or requirements that changed during implementation
- Useful notes for future reference (gotchas, dependencies, things tried and abandoned)

```bash
gh issue comment <NUMBER> --body "Status update text"
```

### Completing Subtasks

Check off subtask checkboxes as they're completed by editing the issue body.

### Keeping Requirements Current

If requirements evolve significantly during implementation, revise the functional/technical sections in the issue body to keep them accurate — the issue should always reflect current truth, not just the original plan.

```bash
gh issue edit <NUMBER> --body "$(cat <<'EOF'
Updated body with revised requirements
EOF
)"
```