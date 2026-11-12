# CrossPaper decision records

Each project-level decision is recorded once, here, as a short decision record (ADR). A record is not rewritten
when plans change: a later record replaces it, and the old one gets the status `Superseded by NNNN`. The format
and the rules are CrossMono's.

Line references (`path:line`) point at the bootstrap base unless a record or an addendum says otherwise:

- CrossPoint Reader `3a5bfac3` (`develop`, 2026-11-11), the first commit on our `main`
- FreeInk SDK `6a208c0`, the submodule commit pinned by that base

Paths under `freeink-sdk/` are inside the submodule. `paperdiy-reader/` is dcarnicer's port,
https://github.com/dcarnicer/paperdiy-reader at `ccb631d2` (base CrossPoint `5d731041`, FreeInk SDK `fb7e926`).
The plan these records come from is [docs/plan.md](../plan.md).

## Changing a record

- **A decision changes:** write a new record. The old record gets the status `Superseded by NNNN`, or names the
  part that is replaced, and the index row says the same.
- **A fact is new, or a detail is settled, and the decision stands:** add a dated section at the end of the
  record, `## Addendum YYYY-MM-DD: <topic>`.
- **The record states something wrong:** add `## Erratum YYYY-MM-DD: <topic>` at the end. Say what was wrong, and
  what is true now.
- Leave the earlier text as it was written, so the record still shows what was known when it was decided. An
  addendum or erratum that cites lines at a later base says which base (or commit) it uses.

## Index

| # | Decision | Status |
|---|---|---|
| [0001](0001-repo-and-upstream.md) | Repository and upstream | Accepted |
| [0002](0002-base.md) | Base: CrossPoint `develop` at bootstrap | Accepted |
| [0003](0003-sdk.md) | FreeInk SDK: our fork, CrossPoint's pin plus the PaperDIY board commits | Accepted |
| [0004](0004-identity-rules-and-layout.md) | Identity, agent rules and repository layout | Accepted |
| [0005](0005-versioning-releases-and-updates.md) | Versioning, releases and updates | Accepted |
| [0006](0006-partitions.md) | Partition table | Accepted |

Numbers reserved by the plan ([docs/plan.md](../plan.md) §3), written when their phase decides them: 0007
two-button interaction model, 0008 power and refresh baseline, 0009 quality, 0010 install.

## Template

Copy this into `NNNN-short-title.md` with the next free number, and add a row to the index.

```markdown
# NNNN. Title

- Status: Proposed | Accepted | Superseded by NNNN
- Date: YYYY-MM-DD

## Context

The problem, and the facts that limit the answer. Cite files as `path:line` and give URLs.

## Options considered

Each option with its main cost and benefit.

## Decision

What we do. Short enough to check against the code later.

## Consequences

What gets easier, what gets harder, and the follow-up work this creates.

## Sources

The files, links and measurements this record relies on.
```
