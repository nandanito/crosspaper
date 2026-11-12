# 0005. Versioning, releases and updates

- Status: Accepted
- Date: 2026-11-12

## Context

CrossPoint's version is the `CROSSPOINT_VERSION` build flag, set per environment from `[crosspoint] version`
(`platformio.ini:6-7`, for example `platformio.ini:331`). Its updater:

- asks `https://api.github.com/repos/crosspoint-reader/crosspoint-reader/releases/latest`
  (`src/network/OtaUpdater.cpp:22`), which never returns a GitHub pre-release;
- looks for the asset `crosspoint-<tag>-<board>.bin` in a 48-byte buffer (`src/network/OtaUpdater.cpp:41-59`);
- compares versions with `sscanf("%d.%d.%d")` as major, minor, patch, and treats the release as newer when the
  numbers are equal and the running version contains `-rc` (`src/network/OtaUpdater.cpp:94-134`).

dcarnicer's port makes the release URL a build flag, `CROSSPOINT_OTA_RELEASES_URL`, still CrossPoint's by default
(`paperdiy-reader/src/network/OtaUpdater.cpp:21-26`), and keeps CrossPoint's asset prefix (`:62`). CrossMono
replaced all of this for its board in one place, with its own `VERSIONING.md` and `RELEASING.md` (CrossMono ADR
0004 and ADR 0009).

## Options considered

1. **Continue CrossPoint's numbers with a suffix** (`2.0.0-paper.1`). Shows the base, but suggests a maturity we
   do not have, and our releases will not follow upstream's.
2. **Own SemVer from `0.1.0`** (chosen), as CrossMono. A new, pre-1.0 product; the CrossPoint base is recorded
   next to the version.
3. **Calendar versions.** Say nothing about compatibility and do not fit the updater's three-number comparison.

For who changes it: several workers each touching a part (version flags, OTA, About, USB strings) would share
`platformio.ini` and `src/network/OtaUpdater.cpp`; one integration step owned by the lead avoids that.

## Decision

- SemVer from `0.1.0`, independent of CrossPoint's numbers, following CrossMono's scheme: tags without `v`
  (`0.1.0`), release candidates `0.1.0-rc.1` published as GitHub pre-releases, development builds
  `0.1.0-dev+<short sha>`. `CHANGELOG.md` follows Keep a Changelog. W3 writes `VERSIONING.md` and `RELEASING.md`
  from CrossMono's.
- The release environment, the version macros, the About identity, the OTA release URL and the OTA asset names
  (and the USB product strings) are one integration step, **L1**, owned by the lead after W1 and W3
  ([docs/plan.md](../plan.md) §5). W3 is done only when L1 builds.
- About shows the CrossPaper version, the CrossPoint base commit and the SDK commit.
- Until L1, a build still points at CrossPoint's releases and names itself CrossPoint.

## Consequences

- No `v` prefix, ever: `sscanf` cannot parse it and the comparison reads uninitialised numbers.
- L1 edits `platformio.ini` and `src/network/OtaUpdater.cpp`; both become seams then, with rows in
  [docs/upstream-sync.md](../upstream-sync.md).
- The OTA URL and asset-name hunks of dcarnicer's `firmware.patch` are L1's; W1 defers them. The board tag that
  the asset suffix uses is board support and stays with W1.
- The first install on a PaperDIY, from stock firmware or from dcarnicer's port, is by USB anyway
  ([ADR 0006](0006-partitions.md); install is the planned ADR 0010).

## Sources

- `platformio.ini:6-7,331`, `src/network/OtaUpdater.cpp:22,41-59,94-134` at `3a5bfac3`
- `paperdiy-reader/src/network/OtaUpdater.cpp:21-26,62`; `paperdiy-reader/platformio.ini:651`
- CrossMono `docs/decisions/0004-versioning.md`, `docs/decisions/0009-install.md`, `VERSIONING.md`,
  `RELEASING.md`
- SemVer 2.0.0: https://semver.org; Keep a Changelog 1.1.0: https://keepachangelog.com/en/1.1.0/
