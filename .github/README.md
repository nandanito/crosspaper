# CrossPaper

Open-source reading firmware for the M5Stack **PaperDIY** e-paper kit, made for its two buttons. It is built on
[CrossPoint Reader](https://github.com/crosspoint-reader/crosspoint-reader) and the
[FreeInk SDK](https://github.com/Free-Ink/freeink-sdk).

> **Status: pre-release.** CrossPaper has just started. There is no release and nothing to install yet. This
> repository holds CrossPoint's code and history, and the plan.

## What it is

CrossPoint Reader is community-built e-reader firmware: EPUB, TXT and XTC books, custom fonts, highlights,
dictionaries, KOReader sync, Wi-Fi file transfer and much more.

The PaperDIY has a 4.7" 960×540 panel, two buttons (A and B), and no touch screen. CrossPoint's screens expect four
buttons or touch: on a PaperDIY some keys of the on-screen keyboard cannot be reached, so Wi-Fi cannot be set up
on the device, and the button hints show four actions. CrossPaper makes every screen a reader needs work with A and
B alone.

The goal for 0.1.0: Home, Library, the file browser, the reader and its menu, Settings, Wi-Fi setup, file transfer
and About, all usable with two buttons, with CrossPaper's own releases and updates. The plan is in
[docs/plan.md](../docs/plan.md), the decisions in [docs/decisions/](../docs/decisions/README.md).

## Credits

- [CrossPoint Reader](https://github.com/crosspoint-reader/crosspoint-reader), the firmware CrossPaper is built
  on.
- The [FreeInk SDK](https://github.com/Free-Ink/freeink-sdk), its board and driver layer.
- [dcarnicer's paperdiy-reader](https://github.com/dcarnicer/paperdiy-reader), the first port of CrossPoint to the
  PaperDIY. CrossPaper's board support starts from it.

## Licence

The code is MIT, like CrossPoint ([LICENSE](../LICENSE)). The firmware image as a whole is GPLv3, because it
includes wolfSSL.

CrossPaper is independent community firmware. It is not affiliated with or supported by M5Stack. PaperDIY is a
product of M5Stack.
