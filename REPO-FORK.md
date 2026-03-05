# Fork Management Guide

This document explains how we maintain our personal fork of [steipete/gogcli](https://github.com/steipete/gogcli) — keeping it in sync with upstream while layering our own feature branches on top.

## Repository Layout

```
origin   → https://github.com/peteradams2026/gogcli  (our fork)
upstream → https://github.com/steipete/gogcli        (official)
```

**Key branches:**

| Branch | Tracks | Purpose |
|--------|--------|---------|
| `main` | `upstream/main` | Mirror of official upstream; never add custom commits here |
| `feat/*` | branches off `main` | Feature branches submitted as PRs to upstream |
| `pa-prod` | rebased on `main` | Our production branch: `main` + all our patches. Used to build our local `gog` binary |

## Daily Binary

Our installed `gog` binary (`~/.local/bin/gog`) is compiled from `pa-prod`, not from `origin/main`. This ensures we always have our patches active.

To rebuild after changes:
```bash
cd ~/ws/gogcli
git checkout pa-prod
make
cp bin/gog ~/.local/bin/gog
```

## Sync Workflow: Keeping Up With Upstream

When the official repo releases new commits:

```bash
cd ~/ws/gogcli

# 1. Fetch upstream changes
git fetch upstream

# 2. Fast-forward main to upstream
git checkout main
git merge --ff-only upstream/main
git push origin main   # keep fork in sync too

# 3. Rebase pa-prod on the new main
git checkout pa-prod
git rebase main
# Resolve any conflicts, then:
git push origin pa-prod --force-with-lease

# 4. Rebase any in-flight feature branches too
git checkout feat/extra-scopes
git rebase main
git push origin feat/extra-scopes --force-with-lease
```

## Adding a New Feature Branch (for PR to upstream)

```bash
cd ~/ws/gogcli
git checkout main
git pull upstream main  # make sure main is current
git checkout -b feat/<short-name>

# ... implement feature, commit, test ...

# Push and open PR against steipete/gogcli:main
git push origin feat/<short-name>
gh pr create \
  --repo steipete/gogcli \
  --head peteradams2026:feat/<short-name> \
  --base main \
  --title "feat(...): ..." \
  --body "Closes #<issue>"
```

## Promoting a Feature to pa-prod

Once a feature branch is ready (either before or after PR merge), cherry-pick it onto `pa-prod`:

```bash
cd ~/ws/gogcli
git checkout pa-prod

# Option A: cherry-pick individual commits
git cherry-pick <commit-sha>

# Option B: merge the whole branch (creates a merge commit)
git merge --no-ff feat/<short-name> -m "merge feat/<short-name> into pa-prod"

git push origin pa-prod
```

After the upstream **merges** our PR:
```bash
# Once upstream merges, the commit lands on main.
# Sync main (see above), then rebase pa-prod — the cherry-picked commit
# will be detected as already-applied and dropped automatically.
git fetch upstream
git checkout main && git merge --ff-only upstream/main
git checkout pa-prod && git rebase main
```

## Current Feature Branches

| Branch | Issue | PR | Status |
|--------|-------|----|--------|
| `feat/extra-scopes` | [#420](https://github.com/steipete/gogcli/issues/420) | [#421](https://github.com/steipete/gogcli/pull/421) | ⏳ PR open |

*(Update this table as branches are added/merged)*

## Files That Live Only on pa-prod (Never PRed Upstream)

These are personal meta-docs we keep on `pa-prod` but intentionally exclude from upstream PRs:

- `REPO-FORK.md` (this file)
- `CLAUDE.md` (agent instructions for this fork)

They are listed in `.git/info/exclude` (local gitignore) or staged only on `pa-prod`.

## Resolving Conflicts During Rebase

If `git rebase main` hits conflicts on `pa-prod`:

1. Fix the conflict in the file
2. `git add <file>`
3. `git rebase --continue`
4. If hopeless: `git rebase --abort` and ask for help

Common conflict: `CHANGELOG.md` — upstream keeps adding entries; just manually merge both sections.
