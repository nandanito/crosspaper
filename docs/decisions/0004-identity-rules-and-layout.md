# 0004. Identity, agent rules and repository layout

- Status: Accepted
- Date: 2026-11-12

## Context

The project needs a name, wording that makes clear it is not an M5Stack product, agent rules that fit a downstream
repository, and a layout that keeps our diff against CrossPoint small enough to merge at every release.

- On 2026-11-12 `nandanito/crosspaper` and the GitHub organisation `crosspaper` were free. The repository now
  exists ([ADR 0001](0001-repo-and-upstream.md)).
- Upstream's `CLAUDE.md` is a symlink to `AGENTS.md`. `AGENTS.md` is written for CrossPoint: integration targets
  `develop` (`AGENTS.md:17`); agents never open or close a PR, the human writes PR descriptions, commits need
  approval, and hardware testing comes before a PR (`AGENTS.md:49-71`); no assistant trailers
  (`.agents/rules/git-workflow.md:16`). Writing to the symlink would change `AGENTS.md` itself, an upstream file
  that then conflicts at every sync.
- CrossMono solved the same problems: a real `CLAUDE.md` that imports `@AGENTS.md` and overrides it in a table
  (CrossMono `CLAUDE.md:64-90`), code in its own folders (`src/mono/`, `lib/mono-*`, `scripts/mono/`,
  `test/mono/`), and a seams table for every upstream file it edits (CrossMono `docs/upstream-sync.md`).
- Upstream-visible names: the hotspot `CrossPoint-Reader` and mDNS `crosspoint`
  (`src/activities/network/CrossPointWebServerActivity.cpp:26,28`), Calibre Connect's `crosspoint`
  (`src/activities/network/CalibreConnectActivity.cpp:17`), the DHCP name `CrossPoint-Reader-<MAC>`
  (`src/activities/network/WifiSelectionActivity.cpp:478-485`) and the SD card folder `.crosspoint/`. CrossMono
  moved to its own hotspot and mDNS names in 0.2.0 (CrossMono `src/mono/Brand.h:12-14`).

## Options considered

For the name:

1. **CrossPaper** (chosen). Says "CrossPoint on the PaperDIY" the way CrossMono does for the PaperMono, and was
   free on GitHub.
2. **Keep CrossPoint's name.** Suggests that CrossPoint supports the build, which it does not (#4005).
3. **"PaperDIY Reader".** Close to dcarnicer's `paperdiy-reader`, so the two would be confused.

For the agent rules: edit `AGENTS.md` (conflicts at every sync, and changes CrossPoint's own rules), or a real
`CLAUDE.md` that imports it unchanged and overrides what differs (chosen, as CrossMono).

For the layout: edit upstream files where the change belongs (small at first, painful at every sync), or put our
code in new files and keep each unavoidable edit small and listed (chosen, as CrossMono).

## Decision

- Name **CrossPaper**. Standard wording, used in the README, the web interface and announcements:

  > CrossPaper is independent community firmware. It is not affiliated with or supported by M5Stack. PaperDIY is a
  > product of M5Stack.

- `CLAUDE.md` becomes a real file. The symlink is removed first, never written through. The file imports
  `@AGENTS.md` and carries a CrossPaper override table modelled on CrossMono's, including "work targets `main`".
- Our code lives in new files: `src/paper/`, `lib/paper-*`, `scripts/paper/`, `test/paper/`, and our own docs.
- Every upstream file we edit gets a row in the seams table of [docs/upstream-sync.md](../upstream-sync.md), in
  the same PR that edits it.
- Upstream-visible names (hotspot `CrossPoint-Reader`, `crosspoint.local`, `.crosspoint/`) stay as they are at
  bootstrap. Whether CrossPaper uses its own hotspot and mDNS names, as CrossMono does, is decided in L1, the
  identity integration step ([ADR 0005](0005-versioning-releases-and-updates.md)).

## Consequences

- `CLAUDE.md` is the first seam. At a sync, upstream's change to the symlink (if any) is read and the import kept.
- `AGENTS.md` and its skills stay CrossPoint's and merge cleanly; the overrides live in one place.
- Orca workers and reviewers read `CLAUDE.md` and [docs/plan.md](../plan.md) first. The user-level `review-*`
  agents mention CrossMono, so their packets carry CrossPaper's override table.
- Keeping `.crosspoint/` leaves the SD card layout CrossPoint's (for example `src/WifiCredentialStore.h:44`), so
  CrossPaper adds no storage seam.

## Sources

- `AGENTS.md:17,49-71` and `.agents/rules/git-workflow.md:16` at `3a5bfac3`; `git ls-tree 3a5bfac3 CLAUDE.md`
  (mode `120000`, a symlink)
- CrossMono `CLAUDE.md:64-90`, `docs/decisions/0002-identity-and-scope.md`, `docs/upstream-sync.md`
- GitHub name check, 2026-11-12 ([docs/plan.md](../plan.md) §2)
