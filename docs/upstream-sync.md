# Syncing with CrossPoint and the FreeInk SDK

CrossPaper's `main` shares history with CrossPoint's `develop` ([ADR 0001](decisions/0001-repo-and-upstream.md)).
We merge upstream at each CrossPoint release. Our FreeInk SDK pin is CrossPoint's pin plus the PaperDIY board
commits, in our fork ([ADR 0003](decisions/0003-sdk.md)). This file is the procedure for both, and the list of
upstream files we edit (the seams).

## One-time setup in a new clone

```sh
git clone https://github.com/nandanito/crosspaper.git && cd crosspaper
git remote add upstream https://github.com/crosspoint-reader/crosspoint-reader.git
git remote set-url --push upstream DISABLED        # never push to CrossPoint by accident
git config remote.upstream.tagOpt --no-tags        # keep upstream tags out of our tag namespace...
git config --add remote.upstream.fetch '+refs/tags/*:refs/tags/upstream/*'   # ...and fetch them as upstream/<tag>
git fetch upstream
git submodule update --init --recursive
git submodule status                               # must show the pin named in the last sync below
gh repo set-default nandanito/crosspaper           # otherwise gh may resolve to CrossPoint
```

The repository itself was created the same way, with `git init -b main` and `git checkout -B main
upstream/develop` instead of the clone, so CrossPoint's tags never entered our namespace.

Push our own tags one by one (`git push origin 0.1.0`), never with `--tags`. Pass `--repo nandanito/crosspaper` on
every `gh` write.

## When to sync

- At every CrossPoint release (a new tag, final or `rc`). Between releases only for a fix we need: merge
  `upstream/develop` at a known commit, the same way.
- Last sync: the bootstrap, CrossPoint `develop` `3a5bfac3` (2026-11-11), FreeInk SDK `6a208c0`
  ([ADR 0002](decisions/0002-base.md)). The PaperDIY board commits are not in the pin yet; W1 adds them.

## Procedure

```sh
git fetch upstream
git switch main && git pull --ff-only
OLD_PIN=$(git rev-parse HEAD:freeink-sdk)        # our SDK pin before the merge
TARGET=refs/tags/upstream/<version>              # or the exact upstream/develop commit, for a between-release sync
git switch -c sync/crosspoint-<version>
git merge --no-ff --no-commit --log "$TARGET"    # never commit before the checks and the handoff below
```

Then:

1. Resolve conflicts with the rules below. Then set the SDK pin with the next section, before building.
2. Check every seam in the table (`grep` for the CrossPaper change in each file; it must still be there and still
   make sense).
3. Read the upstream changes to `.github/workflows/*`, `partitions.csv` and `platformio.ini` even if they merged
   cleanly.
4. If the upstream pin moved, read the SDK changes between the old and the new upstream pin
   (`git -C freeink-sdk log --oneline <old upstream pin>..<new upstream pin>`), above all `InputManager`,
   `BoardConfig` (with the shared `M5Pm1.h`), `PowerManager` and every file the board series touches.
5. Build the PaperDIY environments and run the host tests (`pio run -t unit-tests`). One PlatformIO build at a
   time across all worktrees.
6. Run `firmware-handoff` (`AGENTS.md`); only then commit the merge (`git commit`, which keeps the `--log`
   message).
7. Push the branch and open a PR titled `chore: sync CrossPoint <version>`, each with Nandan's approval. Note in
   the PR the new base commit, the new SDK pin and any seam that needed rework.
8. Merge with a **merge commit**. Never squash or rebase a sync.

### Conflict rules

- **Upstream wins in files it owns.** Outside the seams table we take upstream's version as is.
- **In a seam, take upstream's version first, then re-apply our change on top.** Keep our change as small as
  before; if upstream now offers a hook or a flag that makes our change unnecessary, drop ours and use theirs.
- **Never resolve a conflict by deleting an upstream file.**
- Files we own outright (below) do not conflict; if upstream adds a file with the same path, rename ours.
- **SDK pin conflict:** resolved by the SDK sync below, never by picking one side of the gitlink.

## SDK sync

Our pin = CrossPoint's pin + the PaperDIY board commits, on branch `crosspaper/main` of
`nandanito/freeink-sdk`, each pinned commit tagged `crosspaper/<sdk-short-sha>` ([ADR 0003](decisions/0003-sdk.md)).
The gitlink is the truth: never `git submodule update --remote`.

In the submodule, `origin` follows the `.gitmodules` URL: our fork, once W1 has changed it and `git submodule sync`
has run. Add FreeInk as a remote once:

```sh
git -C freeink-sdk remote add freeink https://github.com/Free-Ink/freeink-sdk.git
```

At every sync, fetch both, and compare CrossPoint's pin at the merge target with the upstream commit our board
series sits on (`OLD_UP`, the "Upstream pin" column of the last row in the table below):

```sh
git -C freeink-sdk fetch freeink
git -C freeink-sdk fetch origin --tags
OLD_UP=$(git -C freeink-sdk rev-parse --verify '<upstream pin, last row of the table>^{commit}')
git -C freeink-sdk merge-base --is-ancestor "$OLD_UP" "$OLD_PIN" || echo "STOP: OLD_PIN is not on OLD_UP"
NEW_UP=$(git rev-parse "${TARGET}:freeink-sdk")  # CrossPoint's pin at the merge target; keep the braces (zsh)
```

- **Upstream did not move its pin** (`NEW_UP` = `OLD_UP`): keep ours. The merge leaves our gitlink at `OLD_PIN`;
  check with `git rev-parse :freeink-sdk`, then `git submodule update --init --recursive`.
- **Upstream moved its pin:** replay the board series onto the new pin.
  1. Start from the pin `main` ships, never from a local branch that may be stale.
     `git -C freeink-sdk rev-parse origin/crosspaper/main` must print `OLD_PIN`. If it does not, stop: the fork
     and `main` disagree, and Nandan decides which board commits count. Then
     `git -C freeink-sdk switch -C crosspaper/main "$OLD_PIN"` and
     `git -C freeink-sdk rebase --onto "$NEW_UP" "$OLD_UP" crosspaper/main`.
  2. Drop board commits that upstream has absorbed. `git rebase` drops a commit that becomes empty. A commit that
     upstream took in part may conflict, or may apply cleanly with only the rest left, so compare the series
     before and after (`git -C freeink-sdk range-diff "$OLD_UP..$OLD_PIN" "$NEW_UP..crosspaper/main"`) and say in
     the PR what was dropped or reduced.
  3. Update the SDK's own submodules, `git -C freeink-sdk submodule update --init --recursive` (not the top-level
     update, which would restore the old pin). Then build the PaperDIY environment and run the SDK host tests
     from inside the SDK (`freeink-sdk/docs/testing.md`), at least
     `(cd freeink-sdk && sh libs/hardware/InputManager/test/host/run.sh)` and the suites for the files the series
     touches.
  4. **Publish and tag before pinning**, with approval for each push:
     `git -C freeink-sdk push --force-with-lease=crosspaper/main:"$OLD_PIN" origin crosspaper/main` (refused if
     the fork's branch moved after `OLD_PIN`), then `git -C freeink-sdk tag crosspaper/<sdk-short-sha>` and
     `git -C freeink-sdk push origin crosspaper/<sdk-short-sha>`. Check that the tag is in the fork:
     `git ls-remote https://github.com/nandanito/freeink-sdk.git refs/tags/crosspaper/<sdk-short-sha>`.
  5. Move the gitlink on the sync branch, which also resolves a gitlink conflict from the merge:
     `git -C freeink-sdk checkout --detach crosspaper/main` and `git add freeink-sdk`. Only after this, and the
     rest of the procedure, commit and push the firmware branch.
- Tags are never moved or deleted; `crosspaper/main` may be rebased. Every pin we ever pushed stays reachable
  through its tag.
- Record the result in the table below and in the sync PR: old and new upstream pin, our new pin and tag, the
  commits replayed and the commits dropped.

To change the submodule URL: edit `.gitmodules`, then `git submodule sync --recursive` and
`git submodule update --init --recursive`.

| Our pin | Tag | Upstream pin | Board commits | Set in |
|---|---|---|---|---|
| `6a208c0` | none (FreeInk's commit) | `6a208c0` | none | bootstrap, `3a5bfac3` |

## Seams: upstream files we edit

Line numbers are at the bootstrap base `3a5bfac3`. Update this table in the same PR that adds, moves or removes a
seam.

| File | What we change | Status |
|---|---|---|
| `CLAUDE.md` | Was a symlink to `AGENTS.md`; now a real file that imports `@AGENTS.md` and adds the CrossPaper rules ([ADR 0004](decisions/0004-identity-rules-and-layout.md)) | edited |
| `.gitmodules` | `freeink-sdk` URL → `https://github.com/nandanito/freeink-sdk.git` ([ADR 0003](decisions/0003-sdk.md)) | planned (W1) |
| `freeink-sdk` (gitlink) | Our pin: CrossPoint's pin plus the PaperDIY board commits (SDK sync above) | planned (W1) |

Not seams:

- `partitions.csv` stays upstream's ([ADR 0006](decisions/0006-partitions.md)).
- Upstream-visible names stay CrossPoint's at bootstrap: the hotspot `CrossPoint-Reader`, `crosspoint.local`, and
  the `/.crosspoint/` folder on the SD card. L1 decides on the network names
  ([ADR 0004](decisions/0004-identity-rules-and-layout.md)).
- `README.md` stays CrossPoint's. GitHub shows our `.github/README.md` first.

Upstream workflows arrive unchanged at bootstrap. W3 adapts them; each one it edits gets a row here.

## Files we own outright

These are new files; upstream never touches them.

- `src/paper/`, `lib/paper-*`, `scripts/paper/`, `test/paper/`
- `.github/README.md` (upstream has no file there; if it adds one, the merge conflicts on it and the sync decides)
- `docs/plan.md`, `docs/decisions/`, `docs/upstream-sync.md`, and as they are added `docs/two-button-ux.md`,
  `docs/install.md`, `docs/hardware-test-checklist.md`
- As they are added: `VERSIONING.md`, `RELEASING.md`, `CHANGELOG.md`, `THIRD-PARTY-NOTICES`
