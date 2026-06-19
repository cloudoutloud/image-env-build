---
name: hadolint-fix
description: This skill should be used when the user asks to "scan Dockerfiles with hadolint", "lint images for best practices", "fix Dockerfile issues", or wants to audit the images/ Dockerfiles against Docker best practices and open a PR with fixes. Runs hadolint, applies best-practice fixes, and creates a branch + PR.
version: 0.1.0
---

# Hadolint Scan & Fix

## Overview

This skill scans the repository's Dockerfiles with [hadolint](https://github.com/hadolint/hadolint),
reports findings, applies best-practice fixes, and opens a pull request with the changes.

Dockerfiles live under `images/<folder>/Dockerfile`. Each image folder also has a `Makefile`
defining `image_name` and `tag` targets — do not modify Makefiles, only Dockerfiles.

## Prerequisites

- `hadolint` must be installed (`which hadolint`). If missing, tell the user to install it
  (`brew install hadolint`) and stop.
- `gh` CLI must be authenticated for PR creation (`gh auth status`). The skill uses the local
  `gh` credentials already on the machine to authenticate — it does not provision or supply its own.

## Workflow

### 1. Determine scope

- If the user names a specific image (e.g. "scan github-runner"), scan only
  `images/<that-folder>/Dockerfile`.
- Otherwise, scan every Dockerfile: `find images -name Dockerfile`.

### 2. Run the scan

Run hadolint on each target Dockerfile. Capture output per file:

```
hadolint images/<folder>/Dockerfile
```

Exit code is non-zero when findings exist. Findings have severities: `error`, `warning`,
`info`, `style`. Each line is `file:line RULE severity: message`.

### 3. Report findings

Present a per-file table of findings (line, rule code, severity, message) before changing
anything. If there are **no findings across all targets**, report a clean result and STOP —
do not create a branch or PR.

### 4. Decide what to fix

Apply fixes only for findings that are safe and clearly correct. Common ones in this repo:

- **DL3027** — replace `apt` with `apt-get`.
- **DL3059** — consolidate consecutive `RUN` instructions into one (joined with `&&`),
  and clean up downloaded archives in the same layer (e.g. `rm -rf *.tar.gz`).
- **DL3008** — pin `apt-get install` package versions where practical.
- **DL3015** — add `--no-install-recommends` to `apt-get install`.
- **DL3009** — `rm -rf /var/lib/apt/lists/*` after installs.
- **Missing non-root USER** (hadolint won't flag this, but check anyway): if the Dockerfile
  has no `USER` and the base image runs as root, that's worth raising. Verify the base
  image's effective user with `docker image inspect ... --format '{{.Config.User}}'`
  before claiming it runs as root. **Do not add `USER` without confirming the base
  image actually runs as root** — many images already drop privileges.

Do **not** auto-fix:
- **DL3029** (`--platform` on `FROM`) — usually intentional for amd64-only runner images.
  Mention it but leave it, unless the user asks. Optionally suppress with
  `# hadolint ignore=DL3029`.

If unsure whether a fix is safe, list it as a recommendation rather than applying it.

### 5. Create branch, commit, and PR

Only if fixes were applied:

1. Create a branch off `main`: `git checkout -b hadolint-fix-<image-or-all>-<short-date>`.
2. Apply the Dockerfile edits.
3. Re-run hadolint to confirm the targeted findings are resolved. Include before/after
   counts in the PR body.
4. Commit. End the commit message with:
   ```
   Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>
   ```
5. Push and open a PR with `gh pr create`. The PR body should:
   - Summarize which Dockerfiles changed and which rules were addressed.
   - Note any findings intentionally left (e.g. DL3029) and why.
   - End with:
     ```
     🤖 Generated with [Claude Code](https://claude.com/claude-code)
     ```

### 6. Report

Give the user the PR URL and a one-line summary of what was fixed and what was left.

## Notes

- Never modify a Makefile, workflow, or non-Dockerfile file as part of this skill.
- Keep each fix minimal and reviewable — prefer the smallest change that resolves the rule.
- If `gh` is not authenticated, apply the fixes on a branch and tell the user to open the PR
  themselves rather than failing silently.
