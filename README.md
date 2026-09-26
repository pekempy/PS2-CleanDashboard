# PS2 Home Screen

A custom modern PS2 home-screen / dashboard UI built with [ps2ui/OPHTML](https://github.com/coffeedevsolutions/OPHTML)
— HTML+CSS compiled into an ultra-fast PS2 homebrew `.uib` UI blob. Six screens:
Home (dashboard), PS2 Games (4-col poster grid), PSX Games (4-col poster grid), Tools, Memory Card, and Game Detail view.

Includes an **always-on FTP status card** in the sidebar and a complete **[Step-by-Step PSBBN Migration Guide](PSBBN_MIGRATION_GUIDE.md)**.

Flat, no gradients, no drop shadows — the design language ps2ui's CSS
subset supports natively, and the right look for a CRT-safe UI anyway.

## Layout

```
ps2ui-homescreen/
  ps2ui.json          project file: screens, shared CSS, strict flags
  ui/
    dashboard.css      shared design tokens + every component style
    home.html           dashboard: stats, "Continue Playing"
    ps2.html             PS2 library: filters + list
    psx.html              PSX library: filters + list
    tools.html              utility launcher list
    memcard.html              save list, single memory-card slot
    detail.html                shared game detail / launch screen
    assets/            baked placeholder cover art (PIL, flat + border)
  fonts/               project-local font + generated metrics
  build/               `ps2ui build` output (git-ignored; see below)
```

## Design

- Palette and component classes live in `ui/dashboard.css`: role-based
  custom properties (`--bg-page`, `--accent`, `--tag-ps2`, …) under a
  `:root` default and an `@theme light` override, so the two preview
  themes recolour with zero geometry change.
- Sidebar nav (`Home / PS2 Games / PSX Games / Tools / Memory Card`) is
  repeated verbatim on every screen — ps2ui has no include/template
  system, so this is the toolchain's own authoring convention (see
  `examples/opl-env` in the OPHTML repo).
- **Cover art aspect ratios are platform-accurate**: PS2 titles use a
  DVD-case portrait ratio (~0.714 w:h — `.row-art-ps2`, `.resume-art`,
  `.detail-art`); PSX titles use a square jewel-case ratio
  (`.row-art-psx`). Library rows stream real art at runtime
  (`data-tex-slot`); the three home-screen "Continue Playing" cards use
  small baked placeholder PNGs in `ui/assets/`.
- The detail screen shows a poster (`.detail-art`) plus a separate key-art
  background banner (`.detail-bg`) above the info panel. ps2ui has no
  `position` / z-index support (confirmed unsupported — the linter warns
  and drops it), so this is a stacked/sectioned composition rather than
  true image-behind-text overlay; that's the toolchain-native way to get
  a "poster + background" hero treatment without fighting the compiler.
- Tools and Memory Card use CSS-only two-letter icon chips
  (`.tool-icon`/`.save-icon`) rather than art, since those are small,
  fixed, non-photographic lists.

## Prerequisites

- Python 3 + the `ps2ui` package (editable install from the OPHTML
  clone): `pip install -e <OPHTML>/packages/baker`. This provides
  `ps2ui`, `ps2ui-bake`, `ps2ui-check`, `ps2ui-fontgen` on `PATH`.
- Node 20+ for `packages/layout`'s zero-dependency layout compiler
  (invoked internally by `ps2ui build`/`ps2ui serve`).
- A DejaVu Sans + DejaVu Sans Bold TTF pair (already vendored into
  `fonts/`, with generated metrics in `fonts/fonts.json`).

```sh
source <path-to-OPHTML>/.venv/bin/activate   # or your own venv with ps2ui installed
```

## Build

```sh
ps2ui build ps2ui.json
```

Produces, under `build/`:

- `ui.uib` — the baked blob (6 screens, shared texture/font/slot tables)
- `preview.png` — a render of the initial screen
- `states.png` — a montage of every focusable's focus state on the
  initial screen

The project file sets `"strict": true` and `"minFontSize": 11`, so the
build fails on any CRT-linter warning (overscan, contrast, 1px
interlace flicker, sub-floor text, etc.) rather than shipping one.

## Validate the blob

```sh
ps2ui-check build/ui.uib
```

Runs the full structural/runtime-invariant suite: focus-graph
reachability and D-pad-edge closure per screen, scissor push/pop
balance, VRAM budget, slot/font/texture table consistency, and more.
Should report `PASS: 107 checks, 0 error(s), 0 warning(s)`.

## Preview

```sh
ps2ui serve ps2ui.json --no-watch
```

Opens an interactive browser preview (arrow keys drive the same D-pad
focus graph the console uses; the Screen/Theme dropdowns switch screens
and themes without rebuilding). Drop `--no-watch` during active editing
to rebuild on save. `--console` fills the theme with mock game data if
you want richer content while iterating (note: streamed `data-tex-slot`
art is intentionally blank in any preview — it only gets pixels from the
real console app calling `ps2ui_tex_set` at runtime, same as on
hardware).

## Iterating on one screen

For fast iteration, the layout compiler can check a single screen
without baking:

```sh
node <OPHTML>/packages/layout/bin/ps2ui-layout.js ui/ps2.html ui/dashboard.css \
  --min-font-size 11 -o /tmp/ps2.json
```

Zero warnings here doesn't guarantee a clean full build — slot names
must be unique across the *whole project* (not just per screen), which
only `ps2ui build` checks — so always finish with a full build before
calling a change done.

## Getting it onto a PS2

This project ships the UI blob only; wiring it to a running app and
getting that app onto real hardware are separate steps, both covered in
OPHTML's own docs:

1. **Wire the runtime.** `ps2ui vendor-runtime --starter src/` (from an
   install) generates `ps2ui.c`/`ps2ui.h` plus a starter `main.c` and
   Makefile. Fill `data-tex-slot`s with real cover art via
   `ps2ui_tex_set`, `data-slot`s with real scanned titles via
   `ps2ui_slot_set`, and wire the sidebar `nav-*` buttons' confirm
   action to `ps2ui_screen_set` (screen transitions are runtime logic —
   nothing in the `.uib` itself switches screens). See
   `<OPHTML>/docs/tutorial-uc3.md`.
2. **Cross-compile.** Needs the [ps2dev toolchain](https://github.com/ps2dev/ps2dev)
   (MIPS target); CI in the upstream repo uses the
   `ghcr.io/ps2dev/ps2dev` container. See `<OPHTML>/docs/deploying.md`
   for the exact `docker run` invocation.
3. **Get the ELF onto the console**: a memory card (plain or a
   multi-channel device like a PSxMemCard GEN2), USB under
   FreeMcBoot (fastest iteration loop — required anyway if streamed
   textures are read from `mass:/ps2ui/`), or listed as an app under
   Open PS2 Loader. Full options and the exact channel-switch ordering
   that trips people up are in `<OPHTML>/docs/deploying.md`.
4. **Autoboot** is a FreeMcBoot setting once the ELF launches and
   responds to the D-pad manually — configure a bypass button before
   relying on it.

## Design tokens reference

All colors/spacing live in `ui/dashboard.css`'s `:root` block. To retheme
project-wide, edit the custom properties there (and their `@theme
light` counterparts) rather than touching component rules — every class
in the file consumes them by `var()`.
