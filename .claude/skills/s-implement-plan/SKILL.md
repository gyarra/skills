---
name: s-implement-plan
description: Use when you have an approved implementation plan — guides through coding, testing, and verification
---

# Implement Plan — Code Through Verification

Executes an approved implementation plan. Requires that planning is already done (via `s-plan-feature` or equivalent).

**At the very start, output this state tracker and update it (checking boxes) as each step completes:**

```
IMPLEMENT PLAN STATE
[ ] Step 1: Write Code
[ ] Step 2: Completion Verification
```

## Step 1: Write Code

1.1 Confirm the implementation plan exists — read it from the GitHub issue. If no plan exists, STOP and invoke `s-plan-feature` first.
1.2 Follow the implementation plan's task sequence. For plans with independent tasks, use `superpowers:executing-plans`.
1.3 After each task, run the relevant checks:
   ```bash
   # Backend
   cd backend && source .venv/bin/activate && ruff check . && pyright
   ```
   ```bash
   # Frontend
   cd frontend && npm run lint && npx tsc --noEmit
   ```
1.4 Write tests alongside implementation — use `superpowers:test-driven-development` when appropriate.
1.5 If blocked or uncertain, ask the user before proceeding.
1.6 Update the GitHub issue as work progresses (status, decisions, completed subtasks).
1.7 Output: `STEP 1 COMPLETE — <N> tasks implemented, all intermediate checks passing`

**Do not output the completion marker until 1.1–1.6 are done.**

Then output the gate check:

```
STEP 1 GATE CHECK
[ ] Implementation plan was read from the GitHub issue
[ ] All tasks in the plan are implemented
[ ] Checks were run after each task
[ ] Tests written for new functionality
[ ] GitHub issue updated with progress
Proceeding to Step 2: YES/NO
```

**Do not proceed until all boxes are checked and the answer is YES.**

## Step 2: Completion Verification

**REQUIRED SUB-SKILL:** Invoke `s-pr-pre-push-review` to run the full review and verification checklist.

Additionally verify:

2.1 All implementation plan tasks are done.
2.2 Migrations generated if needed (`python manage.py makemigrations --check --dry-run`).
2.3 GitHub issue updated with final state.
2.4 Diagrams created/updated (if applicable).
2.5 Output: `STEP 2 COMPLETE — all checks pass, ready to push`

**Do not output the completion marker until 2.1–2.4 are done.**

```
STEP 2 GATE CHECK
[ ] s-pr-pre-push-review completed with zero failures
[ ] All implementation plan tasks marked done
[ ] Migrations checked
[ ] GitHub issue reflects final state
Implementation complete: YES/NO
```
