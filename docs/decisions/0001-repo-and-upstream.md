# 0001. Repository and upstream

- Status: Accepted
- Date: 2026-11-12

## Context

CrossPaper is firmware for the M5Stack PaperDIY built on CrossPoint Reader. CrossPoint is MIT licensed and its
default branch is `develop`. CrossMono, the same maintainer's firmware for the PaperMono, already follows CrossPoint
this way (CrossMono ADR 0001), and its procedures are reused here.

CrossPoint issue #4005 (PaperDIY support) is open and its maintainer is sceptical, so CrossPaper cannot rely on
upstream taking the board. dcarnicer's port (`paperdiy-reader/`) runs CrossPoint on the PaperDIY, but its UI still
needs four buttons or touch ([docs/plan.md](../plan.md) §2). We want to follow upstream closely, keep our diff
small, and be public from the first day.

## Options considered

1. **GitHub fork of `crosspoint-reader/crosspoint-reader`.** One click, but the repository stays in CrossPoint's
   fork network: GitHub search hides forks by default, and the pull-request UI defaults the base repository to the
   parent, so a PR meant for us can be opened against CrossPoint by mistake.
2. **New repository with CrossPoint's full history imported** (chosen). A normal, searchable repository with its
   own default branch and issues. Full history keeps `git blame`, and upstream updates are ordinary merges.
   "Forked from" is lost, so the README credits CrossPoint by hand.
3. **New repository from a snapshot, or a fork of dcarnicer's port.** A snapshot loses history and makes every
   sync a hand-made patch. dcarnicer's port vendors the SDK as a plain directory (`git ls-tree` shows a tree, not a
   gitlink) and is based 30 commits behind `develop`; we take its board work as patches instead ([ADR
   0003](0003-sdk.md)).

## Decision

- New public repository `nandanito/crosspaper` with CrossPoint's full history (option 2). Not a GitHub fork.
- Default branch `main`, which starts at the base in [ADR 0002](0002-base.md). `main` is never rebased; PRs into
  it are merged with a merge commit only.
- Remote `upstream` = CrossPoint, fetch only: push URL `DISABLED`, `tagOpt --no-tags`, and upstream tags fetched as
  `refs/tags/upstream/*`, so they never mix with our release tags. The setup is in
  [docs/upstream-sync.md](../upstream-sync.md); the repository was not made with a plain `git clone`, which would
  have put CrossPoint's tags in our namespace.
- `gh repo set-default nandanito/crosspaper` in every clone, and `--repo nandanito/crosspaper` on every `gh` write:
  in CrossMono, with `origin` and `upstream`, `gh pr create` resolved to CrossPoint.
- We merge upstream at each CrossPoint release: branch `sync/crosspoint-<version>` → PR → merge commit
  ([docs/upstream-sync.md](../upstream-sync.md)).
- Upstream's workflows arrive unchanged. Wave 1 (W3) adapts them.

### Bootstrap record (2026-11-12)

| Item | Value |
|---|---|
| Upstream base | `3a5bfac375c293cb38f7e6e28f7736e7d451ef57` on `develop`, see [ADR 0002](0002-base.md) |
| Repository | https://github.com/nandanito/crosspaper, public, created 2026-11-12, description "CrossPoint Reader firmware for the M5Stack PaperDIY" |
| First push | `main` at the base only; no tags. `git ls-remote origin` showed `refs/heads/main` alone |
| Remotes | `origin` = `nandanito/crosspaper`; `upstream` = CrossPoint, fetch only, tags as `refs/tags/upstream/*` (41 tags at bootstrap) |
| Workflow runs on the first push | None (`gh run list` empty), as expected from the `develop`-only push trigger |

## Consequences

- `main` shares history with CrossPoint `develop`, so it must never be rebased or squashed.
- Upstream files come along unchanged, including `README.md`, `SCOPE.md`, `AGENTS.md` and `.github/workflows/*`.
  The front page is ours from the bootstrap PR on: GitHub shows `.github/README.md` before the root `README.md`.
- The inherited `ci.yml` pushes only on `develop` and runs on every pull request (`.github/workflows/ci.yml:3-6`).
  So the first push of `main` ran nothing, and the bootstrap PR runs CrossPoint's CI: clang-format, cppcheck, and
  builds for eight CrossPoint boards (`ci.yml:81-103`), none of them the PaperDIY. `pr-firmware-links.yml` then
  comments on the PR with links to those builds, and `pr-formatting-check.yml` checks the PR title. W3 adapts the
  workflows to the PaperDIY.
- `CLAUDE.md` was a symlink to upstream's `AGENTS.md`. It is now a real file ([ADR
  0004](0004-identity-rules-and-layout.md)), the first entry in the seams table.

## Sources

- https://github.com/crosspoint-reader/crosspoint-reader (licence, default branch); CrossPoint issue #4005
- CrossMono `docs/decisions/0001-repo-and-upstream.md` and `docs/upstream-sync.md` (one-time setup)
- `.github/workflows/ci.yml`, `pr-firmware-links.yml`, `pr-formatting-check.yml` at `3a5bfac3`
- `git ls-tree ccb631d2 freeink-sdk` in `paperdiy-reader/` (a tree, the vendored SDK)
- Bootstrap session 2026-11-12: `git ls-remote origin`, `gh run list --repo nandanito/crosspaper`
