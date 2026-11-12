# CrossPaper agent guide

@AGENTS.md

The import above is CrossPoint's agent guide, unchanged. Its technical guidance applies here: HAL and SDK
contracts, memory and allocation rules, coding standards, testing, translations, generated files, and its skills
under `.agents/skills/`. Where it conflicts with the CrossPaper rules below, **the CrossPaper rules win in this
repository**. When preparing work for CrossPoint itself, follow `AGENTS.md` as written.

## What CrossPaper is

- Firmware for the M5Stack **PaperDIY** (K159), built on CrossPoint Reader and the FreeInk SDK, designed for its
  real input: two buttons (A and B) and no touch.
- Repository `nandanito/crosspaper`. Pre-release: nothing is published yet.
- Independent community firmware, not affiliated with or supported by M5Stack. Use that wording
  ([ADR 0004](docs/decisions/0004-identity-rules-and-layout.md)).
- Not a place to develop general CrossPoint features. Something useful to every CrossPoint board is offered
  upstream; we carry it only until it lands.
- Maintainer: Nandan.

## Plan and decisions

Read [docs/plan.md](docs/plan.md), and the decision records your task touches, before you start. Do not re-plan
and do not re-open a decision in passing: propose a new record that supersedes it
([docs/decisions/](docs/decisions/README.md)).

| # | Decision |
|---|---|
| 0001 | New repo with CrossPoint's full history (not a GitHub fork); `main`, never rebased, merge commits only; `upstream` fetch-only, its tags as `upstream/*` |
| 0002 | Base: CrossPoint `develop` `3a5bfac3` (2026-11-11), FreeInk SDK pin `6a208c0` |
| 0003 | SDK submodule from `nandanito/freeink-sdk`: CrossPoint's pin plus the PaperDIY board commits; each pinned commit tagged `crosspaper/<sdk-short-sha>` and published before any firmware commit that pins it is pushed |
| 0004 | Name CrossPaper; this file replaces upstream's `CLAUDE.md -> AGENTS.md` symlink; our code in new files; every edited upstream file in the seams table |
| 0005 | Own SemVer from `0.1.0`; release env, version macros, About, OTA URL and asset names are one lead step (L1) |
| 0006 | Upstream `partitions.csv` (two 6.25 MiB app slots) |

Upstream merges, the SDK sync and the seams: [docs/upstream-sync.md](docs/upstream-sync.md).

## Repository layout

- **Ours:** `src/paper/`, `lib/paper-*`, `scripts/paper/`, `test/paper/`, `.github/README.md`, `docs/plan.md`,
  `docs/decisions/`, `docs/upstream-sync.md`, and the files the plan adds (`docs/two-button-ux.md`,
  `docs/install.md`, `docs/hardware-test-checklist.md`, `VERSIONING.md`, `RELEASING.md`, `CHANGELOG.md`,
  `THIRD-PARTY-NOTICES`).
- **Upstream:** everything else, including `README.md`, `SCOPE.md`, `ROADMAP.md`, `AGENTS.md`, `USER_GUIDE.md` and
  the rest of `.github/`. Never delete an upstream file.
- **Seams:** upstream files we edit, listed in [docs/upstream-sync.md](docs/upstream-sync.md). Prefer a new file
  in our folders over an edit. When an edit is unavoidable, keep it small and add or update its row in the seams
  table in the same PR. This `CLAUDE.md` is itself a seam.
- `freeink-sdk/` is a submodule. The gitlink is the truth: run `git submodule update --init --recursive`, compare
  `git submodule status` with the SDK commit your task names, and never use `git submodule update --remote`.

## Where these rules override AGENTS.md

| AGENTS.md | In CrossPaper |
|---|---|
| Integration work targets `develop` | Work targets `main`. Branch from `main`; `develop` exists only upstream |
| Branch types `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf` (`.agents/rules/git-workflow.md:22`) | Also `sync/crosspoint-<version>` for upstream merges ([docs/upstream-sync.md](docs/upstream-sync.md)) |
| Never open or close a PR | Open PRs in `nandanito/crosspaper` when Nandan asks. Never open, close or comment on PRs or issues in other projects |
| The human writes PR descriptions | In our repo a short, plain, factual PR body is fine. For other projects (CrossPoint, FreeInk SDK, M5Stack, dcarnicer's port), draft plain text for Nandan to post |
| Local commits need explicit approval | An approved plan, or an explicit request to commit, covers commits on its branch |
| Hardware testing before a PR is opened | Hardware testing is Nandan's: for firmware PRs, list what to test on the device; the PR is merged only after that |

`firmware-handoff` (with its four review subagents) still applies to firmware and build changes. The Codex review
on the PR comes on top of it.

## Workflow

- Branch → PR → Codex review → merge with a **merge commit**. Never squash, never rebase `main` (it shares
  history with CrossPoint).
- PR titles are semantic (`feat:`, `fix:`, `docs:`, `chore:` …). No assistant co-author or generated-by trailers
  in commits or PR bodies (`.agents/rules/git-workflow.md:16`).
- Commits are SSH-signed through 1Password. If signing fails, ask Nandan to unlock 1Password. Never bypass
  signing (`--no-gpg-sign`, `-c commit.gpgsign=false`).
- Ask first, every time, before: forking, creating a repository, pushing (here or to the SDK fork), opening a PR,
  tagging, publishing a release, or posting anywhere. Approval for one of these does not carry over to the next.
- `gh` can resolve to CrossPoint in a clone with `upstream`. Run `gh repo set-default nandanito/crosspaper` in
  every clone, and pass `--repo nandanito/crosspaper` on every `gh` write.
- Push tags one by one (`git push origin <tag>`), never `git push --tags`: the clone holds CrossPoint's tags as
  `upstream/*`.
- `upstream` is fetch-only. Never push to CrossPoint.

## Build and test

There is no PaperDIY environment at the bootstrap base; wave 1 (W1) adds `paperdiy`.

```sh
~/.local/bin/pio run -e paperdiy       # development build (after W1)
~/.local/bin/pio run -t unit-tests     # host tests (CMake + GoogleTest)
bin/clang-format-fix                   # format; needs clang-format 21
```

Run **one PlatformIO build at a time across all worktrees**, with the global `~/.platformio`:
`scripts/patch_pioarduino_cache.py` edits the installed platform and framework packages, so the core is shared
mutable state. Each worktree keeps its own `.pio`. Do not give a checkout its own PlatformIO core.

## Verify before relying

Line numbers are at the bootstrap base `3a5bfac3` and, under `freeink-sdk/`, at `6a208c0`.

- **Input is two buttons, no touch.** A = GPIO4 and B = GPIO3 in dcarnicer's profile, through
  `InputStyle::DigitalTwoButton`: a short press is Up (A) or Down (B), sent on release; a hold of
  `TWO_BUTTON_HOLD_MS` (650 ms) is Back (A) or Confirm (B); both held for the same time is Power. Left and Right
  never occur (`freeink-sdk/libs/hardware/InputManager/src/InputManager.cpp:561-608`,
  `freeink-sdk/libs/hardware/InputManager/include/InputManager.h:534`).
- **The SDK has no PaperDIY profile at `6a208c0`.** W1 ports it from dcarnicer's `sdk.patch` ([ADR
  0003](docs/decisions/0003-sdk.md)).
- **The panel is 960×540 and slow.** dcarnicer measured `fast` refreshes at 569-620 ms and `text` at
  1538-1542 ms (`docs/paperdiy/06-porting.en.md:229,309` in https://github.com/dcarnicer/paperdiy-reader); every
  list move or keyboard step costs at least about 0.6 s.
- **The firmware image is GPLv3 as a whole** because it links wolfSSL; the code is MIT (`LICENSE:23-26`).
