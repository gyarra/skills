# Running Claude Code in Docker

Claude Code can be run inside a Docker container to give it an isolated, reproducible environment with all required tools pre-installed: Node.js, Python, `uv`, `gh`, Chromium (for browser-based frontend review), and Claude Code itself.

## Files

- `Dockerfile-claude` — the container image definition
- `docker/init.sh` — init script baked into the image and invoked by the `docker run` commands
- `docker/mcp.json` — MCP configuration baked into the image (tools and server settings)

## Build the Image

Run this once (and again whenever `Dockerfile-claude` changes):

```bash
docker build -f Dockerfile-claude -t claude-sandbox .
```

## Run Modes

### Isolated mode

No workspace mount at all. Claude must clone the repo inside the container (using the mounted `gh` credentials), do its work, and push to origin. When the container exits, nothing is left on the host.

```bash
docker run -it --rm \
  -v ~/.claude:/tmp/.claude.seed:ro \
  -v ~/.claude.json:/tmp/.claude.json.seed:ro \
  -v ~/.config/gh:/home/claude/.config/gh:ro \
  --env-file backend/.env \
  --env-file frontend/.env.local \
  claude-sandbox \
  sh -c '/home/claude/init.sh claude --dangerously-skip-permissions'
```

Use this for fully autonomous runs (e.g. "implement this issue and open a PR") where complete isolation is preferred.


### Shared workspace

Mounts your current directory directly into the container. Changes Claude makes are immediately visible on the host — same files, same git history.

```bash
docker run -it --rm \
  -v $(pwd):/workspace \
  -v ~/.claude:/tmp/.claude.seed:ro \
  -v ~/.claude.json:/tmp/.claude.json.seed:ro \
  -v ~/.config/gh:/home/claude/.config/gh:ro \
  --env-file backend/.env \
  --env-file frontend/.env.local \
  claude-sandbox \
  sh -c '/home/claude/init.sh claude --dangerously-skip-permissions'
```

Use this for collaborative/interactive work where you want to see changes as they happen.

### Separate git worktree

Creates an isolated branch via `git worktree` and mounts that instead. Your main working tree is untouched — Claude works on a separate branch in a separate directory.

```bash
# Create a worktree branching from main (one-time per task), one level up
# If you have multiple instances of claude in docker, create a separate workspace directory for each.
git worktree add ../cine-docker-workspace-1 -b feature/my-feature main

# Run Docker pointing at the worktree.
# $(pwd)/../cine-docker-workspace-1 is the path to the worktree created above.
# If you have multiple worktrees, change this path to point at the one you want.
# git worktree add creates the directory — you don't need to create it first.
docker run -it --rm \
  -v $(pwd)/../cine-docker-workspace-1:/workspace \
  -v ~/.claude:/tmp/.claude.seed:ro \
  -v ~/.claude.json:/tmp/.claude.json.seed:ro \
  -v ~/.config/gh:/home/claude/.config/gh:ro \
  --env-file backend/.env \
  --env-file frontend/.env.local \
  claude-sandbox \
  sh -c '/home/claude/init.sh claude --dangerously-skip-permissions'

# Clean up when done
git worktree remove ../cine-docker-workspace-1
```

Use this for autonomous tasks where you don't want Claude touching your current working tree. The worktree shares the full git history and all local branches.


## How Startup Works

There is no entrypoint script. Docker runs the command you pass directly (e.g. `claude --dangerously-skip-permissions`).

### Chromium (on-demand)

Chromium is installed in the image but **not started at container startup**. This avoids background processes competing for stdin, which caused Claude's interactive prompts to freeze.

When Claude needs browser-based testing (e.g. the `s-frontend-visual-review` skill), it launches Chromium itself:

```bash
chromium --headless=new --no-sandbox --disable-dev-shm-usage \
  --remote-debugging-port=9222 --remote-debugging-address=0.0.0.0 \
  </dev/null 2>/dev/null &
```

The `chrome-devtools-mcp` server connects to `localhost:9222` via the `CDP_URL` env var set in the Dockerfile.

## Git Authentication

`init.sh` configures git to use the mounted `gh` credentials for HTTPS authentication via a Git credential helper that delegates to `gh auth`, so `git clone`, `git pull`, and `git push` all work without additional setup.

If you need to manually fix git auth inside the container, prefer configuring Git to use the GitHub CLI as a credential helper instead of embedding tokens in remote URLs:
```bash
gh auth setup-git
```

For pulling changes, `gh repo sync` is a reliable alternative to `git pull` since it uses gh credentials directly.

## Python Version

The backend requires Python 3.13 or newer in its config, but `uv` may install a newer version (e.g. 3.14) if 3.13 is not available in the container. This generally works fine — `uv` manages the Python installation independently.

## Volumes Explained

| Volume | Purpose |
|--------|---------|
| `$(pwd):/workspace` | Mounts the project directory (shared/worktree modes) |
| `~/.claude:/tmp/.claude.seed:ro` | Claude settings seed — copied into the container at startup so each container gets its own writable copy |
| `~/.claude.json:/tmp/.claude.json.seed:ro` | Claude config seed — same copy-on-start approach |
| `~/.config/gh:/home/claude/.config/gh:ro` | GitHub CLI credentials — lets `gh` commands run without re-authenticating |

> **Note:** Both `~/.claude` and `~/.claude.json` must be writable (Claude writes to them during startup). All host config is mounted read-only to `/tmp/` seed paths and copied at startup. This way multiple containers each get their own writable copies without conflicting with each other or the host. The Docker-specific MCP config (`docker/mcp.json`) is baked into the image and merged into `~/.claude.json` at startup by `init.sh`.

## Environment Variables

Both `--env-file` flags are read by Docker from the **host** filesystem at container startup — they don't need to be inside a mounted volume. This means they work in all three run modes.

```
--env-file backend/.env         # Django settings, DB URL, Supabase keys, etc.
--env-file frontend/.env.local  # Next.js public vars, API URLs, etc.
```

Docker's `--env-file` parser:
- Ignores lines starting with `#` and blank lines
- Does **not** support `export KEY=VALUE` syntax
- Does **not** handle shell quoting around values (`KEY="value"` — quotes become part of the value)

## What's Installed in the Image

| Tool | Purpose |
|------|---------|
| Node.js 20 | Runs Claude Code and frontend tooling |
| `@anthropic-ai/claude-code` | Claude Code CLI |
| `uv` | Python package manager for backend |
| `python3` | Required for Django backend commands |
| `gh` | GitHub CLI for PR creation, issue management |
| `git` | Version control |
| `jq` | JSON processing (used by hooks) |
| `chromium` | Headless browser for `chrome-devtools-mcp` |
| `file`, `xxd` | File inspection utilities |
| `curl` | HTTP requests, used by hooks and scripts |


## Test Database Setup

See "Test Database Setup" in the root `CLAUDE.md` for the setup command and troubleshooting. The script is `backend/scripts/setup_test_db.sh`. Configuration is in `config/settings_test.py` (env vars: `TEST_DB_NAME`, `TEST_DB_USER`, `TEST_DB_PASSWORD`, `TEST_DB_HOST`, `TEST_DB_PORT`).

## Initial instructions for Claude in Docker

Copy and paste the following into the Claude prompt after the container starts. Replace the skill path and issue URL with the actual task.

---
### Session Start

You are running in a Docker container. It may not be configured perfectly. Follow these rules for the entire session:

**Track environment issues.** If anything is unclear, missing, or broken in the container environment (missing tools, wrong versions, permission errors, missing env vars, unexpected behavior), note it. If the issue blocks your work, stop and tell me immediately. If it doesn't block you but was confusing or required a workaround, keep a running list.

**Track skill issues.** `.claude/skills/s-ship-feature/SKILL.md` is a template skill that orchestrates other skills in sequence (requirements, planning, implementation, pre-push review, PR creation, and review rounds). As you follow each skill, note any confusing instructions, missing context, contradictions between skills, or steps that didn't work well. At session end, report specific suggestions for improving the skills — what was unclear, what should be added, and what should be removed or reworded.

Clone the repo:
```
gh repo clone gyarra/cine_medallo_2
```

After the repo is checked out, use `.claude/skills/s-ship-feature/SKILL.md` to implement https://github.com/gyarra/cine_medallo_2/issues/178


---
### Session End


Use the end-session skill (`/s-end-session`) to end the session.

Also report:
   - Every environment issue you encountered, even minor ones
   - What you did to work around each issue
   - Specific suggestions for updating `docs/claude-docker.md`, `Dockerfile-claude`, or `docker/init.sh` to prevent the issue in future sessions
