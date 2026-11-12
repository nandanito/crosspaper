# 0006. Partition table

- Status: Accepted
- Date: 2026-11-12

## Context

The PaperDIY has 16 MB of flash (ESP32-S3-WROOM-1-N16R8). CrossPoint's `partitions.csv` gives each of the two app
slots 0x640000 (6,553,600 B, 6.25 MiB), and keeps a 0x360000 `spiffs` partition and a 0x10000 `coredump`
partition (`partitions.csv:1-7`).

dcarnicer's `pio run -e paperdiy` at CrossPoint `5d731041` used 5,347,703 of 6,553,600 B, 81.6 % of a slot
(`paperdiy-reader/docs/paperdiy/06-porting.md:108`). CrossMono saw the same pressure on the PaperMono (82 % at its
base) and moved to two 0x7F0000 slots without `spiffs` (CrossMono ADR 0005).

OTA writes only an app slot and `otadata`, never the partition table: a device keeps the table it was last flashed
with over USB. The stock PaperDIY firmware has a different table (its `nvs` is 0x6000), so the first CrossPaper
install is a full image at 0x0 in any case.

## Options considered

1. **Keep upstream's table** (chosen for now). No seam, no partition file of our own, and the image fits with
   about 18 % to spare.
2. **CrossMono's table now** (2 × 0x7F0000, no `spiffs`). About 1.7 MiB more per slot, but a file and a host test
   to carry before any feature needs the room.

## Decision

- CrossPaper uses upstream's `partitions.csv` unchanged.
- Revisit, the way CrossMono ADR 0005 did, only if the image needs it. A change is a new record.

## Consequences

- Room is tight from the start: about 1.2 MB per slot at dcarnicer's size. W3's release tooling should report the
  size against the slot, as CrossMono's size gate does.
- Changing the table after a release means every installed device needs one USB install, because OTA cannot
  change it. If the size trend points that way, decide before `0.1.0`.

## Sources

- `partitions.csv:1-7` at `3a5bfac3`
- `paperdiy-reader/docs/paperdiy/06-porting.md:108`, `paperdiy-reader/docs/paperdiy/11-upstream-note.md:19`
- CrossMono `docs/decisions/0005-partitions.md`
- Stock firmware backup and its table: the maintainer's notes of 2026-11-08 ([docs/plan.md](../plan.md) §2)
