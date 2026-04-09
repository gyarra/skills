---
name: feedback-docker-fail-fast
description: Docker init.sh should fail fast on missing seed files rather than adding fallback defaults
type: feedback
---

When init.sh can't find seed config files (~/.claude, ~/.claude.json), let it fail immediately rather than creating empty defaults or adding existence checks.

**Why:** Gabriel prefers hard failures that surface configuration problems immediately, rather than silent fallbacks that hide issues.

**How to apply:** Don't add defensive existence checks to Docker scripts when the missing resource means the container is misconfigured. Let set -euo pipefail do its job.
