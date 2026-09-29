# git_sync Skill

Automatically commits and pushes updated persona, skills, and memories to the private remote repository.

## When to Use

When you need to persist Hermes configuration, skill definitions, and session memories across sessions by syncing to the configured remote repository.

## Prerequisites

- Hermes running from `/opt/data`
- Git remote `origin` already configured to `https://github.com/gilbertoesp/hermes-docker.git`
- Write access to the remote repository
- `GITHUB_TOKEN` available in `/opt/data/.env` (or set as environment variable)

## How to Run

```bash
hermes git_sync
```

Or invoke directly:

```bash
cd /opt/data && hermes agent run git_sync
```

## Quick Reference

| Step | Command |
|---|---|
| Add files | `git add config.yaml SOUL.md skills/ memories/ cron/ profiles/` |
| Check changes | `git status --porcelain` |
| Commit | `git commit -m "auto(memory): sync skills, persona and long-term memory [$(date -u +'%Y-%m-%d %H:%M UTC')]"` |
| Push | `git push origin main` |

## Pitfalls

- If `git status --porcelain` returns no output, no commit or push will occur — this is intentional to avoid empty commits.
- Push will fail if the remote has newer commits; run `git pull` first to resolve any divergences.
- Ensure `GITHUB_TOKEN` has `repo` scope; token auth failures will surface as `remote: Permission denied`.
- The log file `/opt/data/logs/git_sync.log` must be writable; create `/opt/data/logs/` if it does not exist.

## Verification

After running `git_sync`, verify:

```bash
git status --porcelain   # should be clean (no staged changes pending)
git log --oneline -1     # should show the auto-commit with timestamp
git push origin main     # should succeed or report merge issues
```

## Procedure

1. `cd /opt/data`
2. Run `git add config.yaml SOUL.md skills/ memories/ cron/ profiles/`
3. If `git status --porcelain` has output, commit with timestamped message
4. Push to `origin main`
5. Append stdout/stderr to `/opt/data/logs/git_sync.log`

## Pitfalls

- No-op when no changes detected — check `git status --porcelain` manually if sync seems stuck.
- Push rejection indicates remote has ahead commits; resolve with `git pull --rebase` then retry.
- Token scope issues: `GITHUB_TOKEN` must include `repo` permission for `git push origin main`.

## Verification

After running the skill, confirm:

- `/opt/data/logs/git_sync.log` contains the sync session output
- `git log --oneline -1` shows the auto-commit with UTC timestamp
- `git status --porcelain` is clean (no staged/pending changes)
- Remote `origin` URL still points to the correct repository

## Author

Hermes Agent