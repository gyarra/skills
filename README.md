# Skills

Gabe's personal Claude Code skills, settings, and Docker harness for autonomous coding sessions.

## Workflow

1. **Write requirements.** Use the [`s-requirements-writing`](.claude/skills/s-requirements-writing/SKILL.md) skill to draft a GitHub issue with clear acceptance criteria.

2. **Spin up Docker.** Start a containerized Claude Code session with the dangerous flag (skips permission prompts so the agent can run autonomously). See [`docs/claude-docker.md`](docs/claude-docker.md) for the run modes and the exact commands. The dangerous flag isn't an issue because it's running in a Docker container.

3. **Prompt the session.** Paste the [Session Start](docs/claude-docker.md#session-start) block from `claude-docker.md` as the initial prompt, replacing the issue URL with the new one. This tells the agent to track environment + skill issues and points it at the issue to ship.

4. **Let it ship.** The agent follows [`s-ship-feature`](.claude/skills/s-ship-feature/SKILL.md), which orchestrates the full lifecycle by invoking sub-skills in order:
   - [`s-plan-feature`](.claude/skills/s-plan-feature/SKILL.md) — read the issue, clarify ambiguities, post an implementation plan
   - [`s-implement-plan`](.claude/skills/s-implement-plan/SKILL.md) — write the code and tests
   - [`s-frontend-visual-review`](.claude/skills/s-frontend-visual-review/SKILL.md) — visual check (if frontend changed)
   - [`s-pr-pre-push-review`](.claude/skills/s-pr-pre-push-review/SKILL.md) — self-review against the [code review checklist](.claude/skills/s-code-review-checklist/SKILL.md), then push and open the PR

The Ship Feature has gatechecks to prevent skipping steps.

5. **Automated review.** GitHub Copilot and Codex automatically review the PR and any follow-up commits, posting their comments directly on the PR.

6. **Respond to feedback.** The agent uses [`s-pr-respond-to-feedback`](.claude/skills/s-pr-respond-to-feedback/SKILL.md) to read every comment, post numbered agree/disagree replies on the PR, and ask me which items to fix. After I confirm, it implements the fixes, runs the checks, pushes, and waits for the next round (capped at 3 rounds by default).

## Layout

- `.claude/skills/` — the skill definitions invoked above
- `.claude/memory/` — long-lived feedback/reference notes that persist across sessions
- `.claude/settings.json` — shared Claude Code settings (permissions, hooks)
- `.github/copilot-instructions.md` — repo conventions Copilot reviewers read
- `.github/copilot-review-instructions.md` — review-specific guidance for Copilot
- `docker/` — `init.sh` and `mcp.json` consumed by the container at startup
- `docs/claude-docker.md` — Docker harness reference (build, run modes, session prompts)
