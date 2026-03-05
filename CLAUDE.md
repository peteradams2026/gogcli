# Agent Instructions for gogcli Fork

This file is for AI coding agents (Claude Code, Codex, etc.) working in this repository.
It extends the upstream `AGENTS.md` with fork-specific context.

> **Read `REPO-FORK.md` first** — it explains our branch structure and sync workflow.

## Who We Are

This is a personal fork of [steipete/gogcli](https://github.com/steipete/gogcli) maintained by
[@peteradams2026](https://github.com/peteradams2026) for [Zainan Victor Zhou](https://github.com/xinbenlv).

## Branch Rules (Critical)

| Branch | Rule |
|--------|------|
| `main` | **Never commit here.** Only `git merge --ff-only upstream/main`. |
| `feat/*` | Feature work intended for upstream PR. Keep it clean and minimal. |
| `pa-prod` | Our integration branch. OK to add meta-docs, personal patches, cherry-picks. |

**When asked to implement a feature for PR:** work on `feat/<name>`, branched off `main`.  
**When asked to update pa-prod or maintain our fork:** work on `pa-prod`.

## Our Active Features (Patches on pa-prod)

- `feat/extra-scopes` — adds `--extra-scopes` flag to `gog auth add` ([issue #420](https://github.com/steipete/gogcli/issues/420))

## Building

```bash
make                  # builds bin/gog
cp bin/gog ~/.local/bin/gog   # install to our PATH (takes precedence over homebrew)
```

## Files Private to This Fork

`REPO-FORK.md` and `CLAUDE.md` (this file) are **not** submitted to upstream.
Do not include them in any PRs to `steipete/gogcli`.

## PR Checklist (before pushing a feature branch)

- [ ] Branched off latest `main` (not `pa-prod`)
- [ ] `make ci` passes (fmt + lint + test)
- [ ] Smoke-tested with `--dry-run`
- [ ] CHANGELOG.md entry added under `## Unreleased`
- [ ] No private files (`REPO-FORK.md`, `CLAUDE.md`) included in the commit diff

## Upstreaming Checklist

After upstream merges our PR:
1. `git fetch upstream && git checkout main && git merge --ff-only upstream/main`
2. `git checkout pa-prod && git rebase main`
3. Remove the feature from "Active Features" table above
4. Rebuild: `make && cp bin/gog ~/.local/bin/gog`
