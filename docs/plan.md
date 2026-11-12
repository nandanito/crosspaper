# CrossPaper plan (v2, 2026-11-12)

> Copied unchanged on 2026-11-12 from the lead session's plan, as approved for P0. It is not edited to track
> progress; a decision changes through a new record in [decisions/](decisions/README.md). `D`, `M` and `U` (§2) are
> local checkouts on the maintainer's machine. The memory names in §4 (`model-workflow`, `codex-and-builds`) are
> the lead session's own notes.

Firmware for the M5Stack PaperDIY (K159), built on CrossPoint Reader and the FreeInk SDK, designed for its real
input: two buttons, no touch. Maintainer: Nandan (`nandanito`). Modelled on CrossMono
(`~/Development/eink/papermono/crossmono`, `nandanito/crossmono`).

History: v1 drafted 2026-11-12. Codex targeted plan review round 1 (`crosspaper-plan-r1`): 2 P1, 11 P2, 1 P3, all
applied in v2. Round 2 (`crosspaper-plan-r2`): no P1, verdict "Ready for P0"; its 1 P2 and 2 factual P3 applied.
Converged; no further plan rounds needed. Reviews: `~/Development/eink/paperdiy/drafts/codex-plan-review-r1.md`,
`-r2.md`.

## 1. Goal and non-goals

Goal for 0.1.0: every screen a reader needs (Home, Library, file browser, reader and its menu, Settings, Wi-Fi
setup, file transfer, About) is usable with A/B alone, on a PaperDIY, with CrossPaper's own identity, releases and
updates, and an agreed minimum battery runtime.

Not in 0.1.0: PaperS3 or other boards; hardware mods (extra buttons on the FPC GPIOs); new themes or network
connectors (CrossPoint `SCOPE.md` freeze).

## 2. Facts this plan rests on (checked 2026-11-08 to 2026-11-12)

`D` = `~/Development/eink/paperdiy/paperdiy-reader` (dcarnicer's fork, base CrossPoint `5d731041`, SDK vendored),
`M` = `~/Development/eink/papermono/crossmono`, `U` = `~/Development/eink/papermono/crosspoint-reader` (user's
CrossPoint clone).

Hardware and input
- ESP32-S3-WROOM-1-N16R8 (16 MB flash, 8 MB PSRAM); ED047TC1 4.7" 960×540, 16 grays, 8-bit parallel bus, pins
  identical to the PaperS3; A = GPIO4, B = GPIO3; power button behind the M5PM1 (I²C SDA41/SCL42); 140 mAh cell;
  microSD over SPI; no touch, frontlight, RTC, IMU or audio (`D/README.en.md:59-71`).
- FreeInk `InputStyle::DigitalTwoButton`: short A/B = Up/Down, emitted on release; a hold of
  `TWO_BUTTON_HOLD_MS = 650` ms = Back (A) / Confirm (B); A+B = Power. Never Left/Right
  (`D/freeink-sdk/libs/hardware/InputManager/src/InputManager.cpp:564-628`, `.../include/InputManager.h:530`).
  The PaperMono uses the same style but has touch.
- Unreachable today without Left/Right or four buttons: the keyboard's column moves
  (`D/src/activities/util/KeyboardEntryActivity.cpp:637-680`, so no Wi-Fi password on the device); tab changes by
  continuous navigation (`D/src/activities/UiTabListActivity.cpp:66`); the four-role front-button remap offered on
  non-touch boards (`D/src/activities/settings/SettingsActivity.cpp:117`,
  `D/src/activities/settings/ButtonRemapActivity.cpp:13,95-110`).
- Four-action hint bar on every non-touch board. Base and Lyra hard-code positions for 480/528-wide panels
  (`D/src/components/themes/BaseTheme.cpp:216-249`, `LyraTheme.cpp:112-123`); RoundedRaff computes from
  `pageWidth` (`RoundedRaffTheme.cpp:210`) but still shows four actions. `UITheme::getMetrics()` zeroes
  `buttonHintsHeight` only for touch (`D/src/components/UITheme.cpp:75-87`).
- Keyboard-free paths that exist (not yet tried on the device): open hotspot `CrossPoint-Reader`
  (`D/src/activities/network/CrossPointWebServerActivity.cpp:26-27`) plus web Settings → Wi-Fi (`/api/wifi`,
  `D/src/network/CrossPointWebServer.cpp:360-362`); fonts copied to `/fonts/` or `/.fonts/` on the SD
  (`D/docs/sd-card-fonts.md`) over USB drive mode (`FREEINK_CAP_USB_MSC=1`).
- Refresh, measured by dcarnicer: `fast` 569-620 ms, `text` 1538-1542 ms; the fast waveform is still a trial
  (`D/docs/paperdiy/06-porting.en.md:229,309`). Every keyboard step or list move costs at least ~0.6 s.
- Power: sleep first asks the PM1 to shut down; A/B wake is only the fallback (`D/lib/hal/HalPowerManager.cpp:162`).
  Measured: A+B over USB powered off via the PM1 and A did not wake; only power turns it on. Sleep current and
  battery life unmeasured (`D/docs/paperdiy/06-porting.en.md:320-328`).

dcarnicer's port
- Two patches, MIT: `D/patches/sdk.patch` (FreeInk `fb7e926`, 18 files, +566/−14: profile `M5PAPER_DIY`,
  `BoardPaperDiy`, `M5Pm1.h`, LGFX driver, BatteryMonitor, InputManager, PowerManager, SDCardManager) and
  `D/patches/firmware.patch` (CrossPoint `5d731041`, 31 files, +322/−40). The firmware patch is more than a board
  adapter: it also migrates a persisted refresh setting (`firmware.patch:366`) and changes list, menu and popup
  refresh behaviour (`firmware.patch:414-628`); its OTA code looks for `crosspoint-…` assets
  (`D/src/network/OtaUpdater.cpp:62`).
- Flashed on the user's device 2026-11-08: boot, A/B, SD, EPUB OK (session observation).
- Its `Makefile` gives each checkout its own PlatformIO core (`CORE_DIR ?= $(CURDIR)/.platformio-core`, ~5 GB).

Upstream and tooling
- CrossPoint `develop` head `3a5bfac3` (2026-11-11), 30 commits past `5d731041`. `2.0.0` (`42d58daa`) pins SDK
  `5e94fe7`, on which GitHub release downloads (OTA, fonts) fail; `develop` pins `6a208c0` with the fix
  (`M/docs/decisions/0015-sdk-pin-ahead-of-a-release.md`).
- Upstream `CLAUDE.md` is a symlink to `AGENTS.md` (`U/CLAUDE.md -> AGENTS.md`). `AGENTS.md:17`: integration work
  targets `develop`. `AGENTS.md` (Human ownership): no fully autonomous end-to-end agents; reviewers read-only;
  commits need approval; push only on explicit instruction; never open or close a PR; the human writes PR
  descriptions and does hardware testing. `.agents/rules/git-workflow.md:16`: no assistant co-author or
  generated-by trailers.
- CrossMono overrides those rules in its own `CLAUDE.md` (`M/CLAUDE.md:64-90`): work targets `main`; PRs only in
  our repo when Nandan asks; short factual PR bodies in our repo; an approved plan covers commits on its branch;
  ask first, every time, before forking, creating a repository, pushing, tagging, releasing or posting; push tags
  one by one; commits SSH-signed through 1Password, never bypassed.
- `scripts/patch_pioarduino_cache.py:19,51-92` edits the installed PlatformIO platform and framework packages,
  outside the project. A shared `~/.platformio` is therefore shared mutable state.
- FreeInk SDK (`Free-Ink/freeink-sdk`, default `main`) has no PaperDIY code and merges board-specific PRs often.
  User fork `nandanito/freeink-sdk` exists (public).
- GitHub (2026-11-12): `nandanito/crosspaper` and the org `crosspaper` are free; `nandanito/crosspoint-reader` is
  the user's fork for upstream PRs; CrossPoint issue #4005 (PaperDIY support) is open, maintainer sceptical.
- Stock firmware backup: `~/Development/eink/paperdiy/paperdiy-original.bin`, SHA-256
  `00d880ccbb9bdab7402e5935a0f5da68e04ace34a1de5a95eef16039c896bef6`. Device port seen as `/dev/cu.usbmodem2101`.
  Download mode: hold power about 10 s until the red LED blinks. First install: full image at `0x0`.

## 3. Decisions to record as ADRs in CrossPaper (`docs/decisions/`)

Use CrossMono's ADR format and rules (`M/docs/decisions/README.md`).

- **0001 Repository and upstream.** `nandanito/crosspaper`, public, new repository with CrossPoint's full history
  (not a GitHub fork), default branch `main`, never rebased, merge commits only. Remote `upstream` = CrossPoint,
  fetch only (push `DISABLED`), upstream tags as `refs/tags/upstream/*` (`M/docs/upstream-sync.md` one-time
  setup). `gh repo set-default nandanito/crosspaper` in every clone, and `--repo nandanito/crosspaper` on `gh`
  writes. Upstream workflows arrive unchanged. Their CI push trigger is `develop` only and they run on pull
  requests (`.github/workflows/ci.yml:4-6`; `M/docs/decisions/0001-repo-and-upstream.md:61`), so the first push of
  `main` runs nothing and the bootstrap PR runs CI. Adapt them in W3.
- **0002 Base.** CrossPoint `develop` at the head fetched on bootstrap day (not `2.0.0`, because of the SDK
  download bug). Record SHA, date and SDK pin.
- **0003 SDK.** `freeink-sdk` submodule URL changes to `https://github.com/nandanito/freeink-sdk.git`. The pinned
  SDK commit = CrossPoint's pinned SDK commit + the PaperDIY board-layer commits ported from dcarnicer's
  `sdk.patch` (credited). Rules:
  - Publish the SDK commit to the fork, and tag it `crosspaper/<sdk-short-sha>`, before any firmware commit that
    pins it is pushed. Every pinned commit stays reachable through its tag forever; tags are never moved or
    deleted. Working branch `crosspaper/main` may be rebased; tags may not.
  - Changing the URL: edit `.gitmodules`, `git submodule sync --recursive`, `git submodule update --init
    --recursive`. Never use `update --remote`; the gitlink is the truth. Source links (About, docs, licence asset)
    name the fork.
  - SDK sync (a CrossPaper-specific section in `docs/upstream-sync.md`): at a CrossPoint sync, if upstream moved
    its SDK pin, replay the remaining board commits onto the new upstream pin, build and host-test, publish and
    tag, then move the gitlink; if upstream did not move it, keep ours. Commits that upstream has absorbed are
    dropped from the replay.
  - Aim: upstream the PaperDIY profile to `Free-Ink/freeink-sdk`, then return to upstream pins. Coordinate with
    dcarnicer and prateek (#4005) first; Nandan posts.
- **0004 Identity, rules and layout.** Name CrossPaper; "independent community firmware, not affiliated with or
  supported by M5Stack". Replace the `CLAUDE.md -> AGENTS.md` symlink with a real file (remove the link first;
  never write through it) that imports `@AGENTS.md` and carries a CrossPaper override table modelled on
  `M/CLAUDE.md:64-90`, including "work targets `main`". Our code in new files (`src/paper/`, `lib/paper-*`,
  `scripts/paper/`, `test/paper/`); every edited upstream file goes in the seams table of `docs/upstream-sync.md`,
  in the same PR that edits it. Upstream-visible names (hotspot `CrossPoint-Reader`, `crosspoint.local`,
  `.crosspoint/`) stay at bootstrap. CrossMono now uses its own hotspot and mDNS names (`M/src/mono/Brand.h:12`);
  whether CrossPaper does the same is decided in L1.
- **0005 Versioning, releases and updates.** Own semver from `0.1.0`, independent of CrossPoint (CrossMono
  `VERSIONING.md`, `RELEASING.md`). Release env, version macros, About identity, OTA URL and OTA asset names are one
  integration step owned by the lead (L1 in §5). About shows the CrossPoint base and SDK commit.
- **0006 Partitions.** Upstream `partitions.csv` (two 6.25 MiB app slots; dcarnicer's build used 81.6 %). Revisit
  like CrossMono 0005 only if needed.
- **0007 Two-button interaction model.** From the W2 spec, after Nandan approves it.
- **0008 Power and refresh baseline.** Shutdown versus ESP deep sleep, wake sources, saved state, recovery; the
  waveform and repaint policy for navigation. Decided at G1 from measurements.
- **0009 Quality.** CrossMono's CI adapted; `firmware-handoff` with the four review agents
  (`~/.claude/agents/review-*.md`) plus one native Codex pass per PR, run last; a hardware test checklist; Nandan
  flashes and tests.
- **0010 Install.** First install: backup, then full image at `0x0` with esptool. Web flasher at
  `crosspaper.github.io` and M5Burner (`D/scripts/make_m5burner_package.py` as reference) later.

## 4. Agent workflow

Lead: a Claude Code session in `~/Development/eink/crosspaper`, which is the repository root. It is the main agent
in `AGENTS.md` terms: it writes task specs, integrates, runs builds, runs `firmware-handoff`, and stops at every
gate.

Model routing follows Nandan's split (CrossMono memory `model-workflow`): Fable plans and designs, Opus 5.5
executes, Sonnet 5.5 reviews, looks things up and does mechanical work, and Codex gives a second opinion. At the
start of each session and at each phase change, the lead compares its own model with this table and asks Nandan to
switch with `/model` if they differ.

| Work | Agent | Model / effort |
|---|---|---|
| W2 interaction spec; ADR 0007/0008 drafts; judgment on hard review findings | Claude | Fable 5.1 (`claude-fable-5-1`), high |
| Lead; W1 board bring-up; L1 integration; input core; keyboard A | Claude | Opus 5.5 (`claude-opus-5-5`), xhigh |
| W3 identity and release tooling (copy and adapt from CrossMono); strings, docs, formatting | Claude | Sonnet 5.5 (`claude-sonnet-5-5`), high |
| The four `firmware-handoff` reviews; lookups | Claude subagents | Sonnet 5.5 (`review-*` agents) |
| Keyboard B (competing design); review of W1 and of the W2 spec | Codex | `gpt-6.1-sol` (user config) |

Orca
- One Orca Run per wave. `orca orchestration worker-start --worktree new-child --setup run --agent claude|codex
  --model <id> --effort <level> --json`; never pass a prompt before setup succeeds (`wait-for-setup`). Verify the
  effective model and effort from the receipt.
- Every task spec names Target, Change, Constraints, Ownership (an explicit file allowlist), Observable
  acceptance, and the base revision. It tells the worker to read `CLAUDE.md`, `docs/plan.md` and the ADRs it
  touches first.
- Worktree setup runs `git submodule update --init --recursive` and checks `git submodule status` against the
  expected SDK SHA; a mismatch stops the task.
- Worktree sessions have other paths and do not see this memory. The plan lives in the repo (`docs/plan.md`).

Limits
- At most 2-3 implementation workers at once. Workers own disjoint files; shared files have one named owner per
  wave.
- One PlatformIO build at a time across all worktrees: the global `~/.platformio` is shared mutable state (§2).
  Each worktree has its own `.pio` (2-3 GB). Do not copy dcarnicer's isolated-core `Makefile` default.
- Codex: one round at a time machine-wide; native pass once per PR, run last. Invocations, hook rules and the
  wait loop are in the CrossPaper memory `codex-and-builds`. A Codex worker launched through Orca is not seen by
  the Bash hook; the lead applies the same policy by hand.
- Reviewer packets include CrossPaper's `CLAUDE.md` overrides (the `review-*` agent files mention CrossMono).

Approvals (CrossMono rules, `M/CLAUDE.md:84-86`)
- Nandan's bootstrap request authorises: cloning CrossPoint into the folder, creating `nandanito/crosspaper`
  (public) and pushing `main` as imported (CrossPoint history only, no CrossPaper content).
- Everything after that needs its own approval, each time: every push, every PR, tag, release, fork, and any post.
  An approved plan covers commits on that plan's branch; commits are SSH-signed through 1Password, never bypassed.
- No assistant trailers in commits or PR bodies (`.agents/rules/git-workflow.md:16`; same choice as CrossMono).
- Human-owned publication: CrossPoint Discussions, PRs to CrossPoint or FreeInk, the #4005 announcement. The
  lead drafts plain text into `~/Development/eink/paperdiy/drafts/`; Nandan posts.

## 5. Phases and gates

**P0 Bootstrap (lead alone, Opus 5.5).**
1. Check that `~/Development/eink/crosspaper` is empty. `git init -b main`; add `upstream` with the one-time setup
   from `M/docs/upstream-sync.md` (push `DISABLED`, `tagOpt --no-tags`, tags fetched as `refs/tags/upstream/*`;
   not a plain `git clone`, which would put CrossPoint's tags in our namespace); `git fetch upstream`;
   `git checkout -B main upstream/develop`; `git submodule update --init --recursive`; record base SHA, date and
   SDK pin.
2. `gh repo create nandanito/crosspaper --public` (description: "CrossPoint Reader firmware for the M5Stack
   PaperDIY"); add `origin`; `gh repo set-default nandanito/crosspaper`; `git push -u origin main` (branch only,
   no tags).
3. On branch `docs/bootstrap`, prepare without committing: `CLAUDE.md` (symlink replaced, ADR 0004),
   `docs/plan.md` (this plan), ADRs 0001-0006 and the index, `.github/README.md` front page, and
   `docs/upstream-sync.md` with the one-time setup, the SDK sync section and a seams table whose first row is the
   `CLAUDE.md` replacement (`M/docs/upstream-sync.md:99-104` records the same seam). Show Nandan the files.
- **Gate G0:** Nandan approves the files → commit; Nandan approves the push and PR → push, open the PR, run the
  native Codex pass; Nandan approves the merge → merge commit.

**P1 Wave 1 (parallel), after G0.**
- **W1 Board bring-up** (Opus 5.5, xhigh). Ownership: the SDK fork branch `crosspaper/main`; in CrossPaper,
  `.gitmodules`, the `freeink-sdk` gitlink, `platformio.ini`, `lib/hal/**`, `src/main.cpp`, and the other files
  the kept hunks touch. Steps: port `sdk.patch` onto CrossPoint's pinned SDK commit; classify every
  `firmware.patch` hunk as board support, display behaviour or unrelated, keep the board-support hunks, and record
  each omitted or deferred hunk in a table in the PR body. Acceptance: `pio run -e paperdiy` passes; SDK host tests
  pass; the SDK commit is published and tagged before the firmware branch is pushed (both pushes need approval);
  credit in commits and `NOTICE`.
- **W2 Interaction spec** (Fable 5.1, high). Ownership: `docs/two-button-ux.md` only. Contents: inventory (file
  and line) of everything that needs Left/Right, four buttons, touch, held-direction repeat, or nested long
  actions, including tab lists, button remap and persisted input settings; an input-core contract (events, the
  650 ms hold, release-based shorts, release suppression after a hold, cancellation, A+B chord); per-screen
  interaction table; two keyboard designs (A and B) costed against the measured 0.6 s and 1.5 s refreshes; hint
  design; settings that must be hidden on two-button boards; an always-reachable escape path; power-button and
  sleep semantics, aligned with CrossMono's keys where possible (CrossMono: holding both keys goes Home); what can
  go upstream.
- **W3 Identity and release tooling** (Sonnet 5.5, high). Allowlist: `VERSIONING.md`, `RELEASING.md`,
  `CHANGELOG.md`, `docs/install.md`, `docs/hardware-test-checklist.md`, `scripts/paper/**`, `test/paper/**`,
  `.github/workflows/**`, `THIRD-PARTY-NOTICES`. Not allowed: `docs/plan.md`, `docs/decisions/**`,
  `docs/two-button-ux.md`, `docs/upstream-sync.md`, `CLAUDE.md`, `platformio.ini`, `src/**`, `lib/**`.
- **Codex** reviews W1's diff and the W2 spec (targeted rounds per the policy).
- **L1 Integration** (lead, after W1 and W3): release env, version macros, About identity, OTA URL and asset names,
  USB product strings. W3 is done only when L1 builds.
- **Gate G1:** Nandan flashes the bring-up build and runs the hardware checklist: boot, A/B, SD, EPUB, the
  hotspot Wi-Fi path, sleep and wake, a runtime measurement from full charge against a minimum runtime Nandan
  sets, and a refresh baseline for navigation. Nandan approves the spec; ADRs 0007 and 0008 are written.

**P2 Input core, then keyboards.**
- **P2a** (Opus 5.5): implement the input-core contract from the spec; handoff; device test; merge.
- **P2b**, both branched from the P2a merge commit: keyboard A (Opus 5.5) and keyboard B (Codex), each in its own
  worktree, owning only the keyboard files. The Sonnet reviewers review both. Nandan compares them on the device
  and picks one; the other is closed with a note. Gate G2.

**P3 Screens (2-3 workers).** Before dispatch, assign each shared file (`UiListActivity`, `OptionPopup`, theme
metrics, translation sources) to one owner. Work: lists and value pickers, tabs, dialogs and popups, hint bar,
960×540 layout fixes, reader menu, settings visibility, strings. Opus for behaviour, Sonnet for strings and docs.
Gate per merge.

**P4 Release 0.1.0.** Hardware checklist, release workflow, install doc. Nandan posts any announcement.

**Upstream track (alongside, low priority).** The lead drafts: a CrossPoint Discussion on two-button keyboard and
hints, later PRs from `nandanito/crosspoint-reader`, and the FreeInk board-profile PR after coordinating with
dcarnicer and prateek. Nandan posts.

## 6. Risks and open questions

- SDK fork upkeep until the profile is upstream (replay at syncs that move the pin).
- dcarnicer may upstream their own work or object to ours; coordinate first.
- 30 commits of CrossPoint drift since `5d731041` may conflict with `firmware.patch`.
- Power viability: 140 mAh and PM1 shutdown behaviour may not give a useful runtime; G1 decides, power fixes
  allowed.
- Refresh latency makes multi-step input slow; the spec must design for it.
- Power button: app-handled click (`PAPERDIY_PM1_APP_POWER_BUTTON=1`) versus the factory reset; the spec decides.
- BLE page-turner pairing unvalidated.
- Testing throughput is the bottleneck.
- Licence: the firmware image is GPLv3 because of wolfSSL; follow CrossMono's `THIRD-PARTY-NOTICES`.
