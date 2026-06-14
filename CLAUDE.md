# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a ZMK firmware configuration repository for the Kinesis Advantage 360 Pro split keyboard. It does not contain application source code — it contains keyboard firmware configuration files (keymaps, device trees, Kconfig) that are compiled via a custom ZMK fork using Docker.

## Build Commands

All builds require Docker or Podman. The Makefile auto-detects which is available (Podman takes priority).

```bash
make          # Build firmware for both halves
make left     # Build left side only
make clean    # Remove docker container and firmware output
make clean_firmware   # Remove firmware UF2 files only
make clean_image      # Remove the docker image only
```

Firmware outputs to `firmware/` as `*.uf2` files. The `config/version.dtsi` file is auto-generated during build (contains timestamp + git hash) and reset afterward.

### GitHub Actions

Two CI jobs run in parallel on push/PR:
- **build** — Standard firmware (artifacts: `firmware-no-clique`)
- **build-clique** — ZMK Studio-enabled firmware (artifacts: `firmware-clique`)

## Architecture

### ZMK Dependency

The firmware uses a **custom ZMK fork** (`github.com/ReFil/zmk`, branch `adv360-z3.5-2`) rather than upstream ZMK. This fork adds RGB underglow and backlight support specific to the Advantage 360 Pro. Dependency is declared in `config/west.yml`.

### Configuration Files

| File | Purpose |
|------|---------|
| `config/adv360.keymap` | Main keymap (7 layers: Base, Keypad, Function, Modifier, Flykey, Util, NoFly) |
| `config/macros.dtsi` | Custom ZMK behaviors and macro definitions |
| `config/boards/arm/adv360/adv360.dtsi` | Hardware device tree (matrix, LEDs, battery, SPI) |
| `config/boards/arm/adv360/adv360_left_defconfig` | Left half build config (NRF52840, BT, USB, RGB) |
| `config/boards/arm/adv360/Kconfig.defconfig` | Default Kconfig values for both halves |
| `config/west.yml` | West manifest — declares ZMK fork dependency |
| `config/keymap.json` | JSON keymap for GUI editors (e.g., Nick Coutsos's keymap editor) |
| `config/info.json` | Physical key layout metadata (positions, rotations) used by editors |

### Split Keyboard Structure

- `adv360_left.keymap` and `adv360_right.keymap` simply `#include` the main `adv360.keymap`
- `adv360_left.dts` and `adv360_right.dts` are separate device trees for each half
- The build produces two separate UF2 files — one per half

### Keymap Format

Keymaps use ZMK's device tree syntax (`.keymap` files are parsed as DTS). Layers are defined as `bindings` arrays. Key behaviors are referenced via `&behavior_name`.

### Versioning

A version macro in `macros.dtsi` (auto-populated via `config/version.dtsi`) displays firmware version as `YYYYMMDD-XXXX-YYYYYY` (date, short hash, branch) when triggered by `Mod+V`.

## Layer Map

The keymap defines 10 layers. Layer indices in ZMK bindings are zero-based:

| Index | Name | Display | Activated by |
|-------|------|---------|--------------|
| 0 | `default_layer` | Base | Always on |
| 1 | `keypad` | Kp | `&tog 1` (toggle on number row) |
| 2 | `fn` | Fn | `&mo 2` (hold left/right thumb corners) |
| 3 | `mod` | Mod | `&mo 3` (hold right upper thumb key) |
| 4 | `flykey` | — | `&lt_b 4 SPACE` (hold left/right Space thumb key) |
| 5 | `num` | Util | `&lt_b 5 BACKSPACE` (hold left Backspace) or `&lt 5 ENTER` (hold Enter) |
| 6 | `plain` | NoFly | `&to 6` from flykey left Space tap (hold right Space, tap left Space); latching layer |
| 7 | `extra1` | Red | Reserved for ZMK Studio / Clique |
| 8 | `extra2` | Purple | Reserved for ZMK Studio / Clique |
| 9 | `extra3` | Cyan | Reserved for ZMK Studio / Clique |
| 10 | `extra4` | Yellow | Reserved for ZMK Studio / Clique |
| 7 | `extra2` | Purple | Reserved for ZMK Studio / Clique |
| 8 | `extra3` | Cyan | Reserved for ZMK Studio / Clique |
| 9 | `extra4` | Yellow | Reserved for ZMK Studio / Clique |

**Why flykey/num/plain come before the reserved layers:** The non-Studio firmware uses `DT_INST_FOREACH_CHILD_STATUS_OKAY_SEP` to enumerate layers, which **skips** nodes with `status = "reserved"`. Custom layers placed before the reserved block get stable indices in both Studio and non-Studio builds. The reserved layers are appended last — Studio includes them via `DT_INST_FOREACH_CHILD_SEP`; non-Studio skips them with no effect since no binding references indices 7–10.

## Flykey Layer Concept

Flykey is a home-row navigation + text-editing overlay. **Holding Space** activates it so both hands stay on the home position for all cursor movement and editing — no arrow key cluster required.

### Design principles

- **Right hand = navigation**: Arrow keys under J/K/L (left/down/right), Up under I. Word and line jumps on the row above (UIOP).
- **Left hand = editing**: Backspace and Delete under D/F (home position). Kill-line shortcuts (Ctrl+U, Ctrl+K) flank them. Move-line and word-delete shortcuts above. Esc/Enter/Tab on the bottom row.
- **macOS-first** shortcuts: `LG(←)` / `LG(→)` for line home/end; `LA(←)` / `LA(→)` for word jump; `LG(⌫)` / `LG(⌦)` for line delete; `LA(⌫)` / `LA(⌦)` for word delete.
- **Emacs/Unix overlap**: `LC(A)` / `LC(E)` for BOL/EOL (works in terminal and macOS native inputs); `LC(U)` / `LC(K)` for kill-line.

### Right-hand navigation layout

```
Row 2 (UIOP): LG(←)  LA(←)   ↑    LA(→)  LG(→)   — line/word jumps
Row 3 (HJKL): LC(A)    ←      ↓      →    LC(E)   — arrow keys + BOL/EOL
Row 4 (NM,./): Home  PgUp   PgDn   End   Globe     — page navigation; / = &kp GLOBE (macOS Globe / next-input-source, consumer 0x029D)
```

### Left-hand editing layout

```
Row 2 (QWERT): LA(↑)  LG(⌫)  LA(⌫)  LA(⌦)  LG(⌦)  — move-line, word/line delete
Row 3 (ASDFG): LA(↓)  LC(U)   ⌫      ⌦     LC(K)   — kill-line, backspace, delete
Row 4 (ZXCVB):  Esc  LG(↑)  LG(↓)   Ret    Tab     — escape, doc nav, confirm
```

### Util layer layout

Activated by holding left Backspace (`&lt_b 5 BACKSPACE`) or Enter (`&lt 5 ENTER`).

**Right hand — numpad:**
```
Right row 2 (YUIOP): +   7   8   9   *
Right row 3 (HJKL=): 0   4   5   6   =
Right row 4 (NM,./): -   1   2   3   /
```

**Left hand — pure modifiers (home row, no dual-role):**
```
A=⌃  S=⌥  D=⌘  F=⇧  G=Space
```

**Left hand — edit shortcuts (bottom row):**
```
Z=Undo(⌘Z)  X=Cut(⌘X)  C=Copy(⌘C)  V=Paste(⌘V)  B=Redo(⌘⇧Z)
```

**Thumb — layer navigation (exit NoFly):**
- Left Backspace (`&to 0`) and Right Enter (`&to 0`) → Base
- Exit gesture is symmetric: hold one of the two thumb keys (left Backspace or right Enter), tap the other → Util's `&to 0` fires → Base

### Dual-role Keys (`hm` and `hm_b` behaviors)

Two hold-tap behaviors are used for dual-role keys:

- **`hm`** (`balanced`, `tapping-term-ms = 280`, `quick_tap_ms = 175`, `require-prior-idle-ms = 150`) — "timeless HRM" pattern for home-row mods only. Hold triggers when another key is pressed **and released** while held. The `require-prior-idle-ms` guard prevents accidental mod triggers during fast typing by treating the key as a pure tap if any key was pressed within 150ms before it.
- **`hm_hp`** (`hold-preferred`, `tapping-term-ms = 200`, `quick_tap_ms = 175`) — for outer edge keys (pinky ctrl, shift columns) and top thumb cluster. Hold triggers as soon as ANY other key is pressed, even before release. Best for dedicated modifier positions where hold behavior should dominate.

**Home row (uses `hm` — balanced + idle guard):**

| Physical key | Tap | Hold |
|---|---|---|
| A | A | Left Control |
| S | S | Left Alt |
| D | D | Left Command |
| F | F | Left Shift |
| J | J | Right Shift |
| K | K | Right Command |
| L | L | Right Alt |
| ; | ; | Right Control |

**Outer pinky column (uses `hm_hp` — hold-preferred):**

| Physical key | Tap | Hold |
|---|---|---|
| Left of A (outer pinky) | `` ` `` / `~` | Left Control |

**Right of ; (outer pinky):** Plain `&kp SQT` — no dual-role.

**Outer shift column (uses `hm_hp` — hold-preferred):**

| Physical key | Tap | Hold |
|---|---|---|
| Left of Z | `\` / `\|` | Left Shift |
| Right of / | `/` / `?` | Right Shift |

**Top thumb cluster (uses `hm_hp` — hold-preferred):**

| Physical key | Tap | Hold |
|---|---|---|
| Left thumb outer (pos 35) | `[` / `{` | Left Alt |
| Left thumb inner (pos 36) | `]` / `}` | Left Command |
| Right thumb inner (pos 37) | `-` / `_` | Right Command |
| Right thumb outer (pos 38) | `=` / `+` | Right Alt |

To tune home-row mods: adjust `require-prior-idle-ms` (increase to reduce accidental mods, decrease for faster activation after typing), `tapping-term-ms`, or `quick_tap_ms` in the `hm` behavior definition in `adv360.keymap`. For outer/thumb keys: tune the `hm_hp` behavior independently.

## Guidelines for Modifying the Keymap

### Clique (ZMK Studio) compatibility

The Clique firmware (`build-clique` CI job) enables ZMK Studio via the `studio-rpc-usb-uart` snippet and `-DCONFIG_ZMK_STUDIO=y`. Three things must be preserved for Clique to work:

1. **`#include <dt-bindings/zmk/pointing.h>`** — must stay in `adv360.keymap`. `CONFIG_ZMK_POINTING=y` is set in `adv360_left_defconfig`; removing the include creates a mismatch that can break Studio initialization.
2. **`stp STP_BAT`** in the mod layer — Kinesis-fork-specific Studio Transport Protocol battery behavior. Required for Studio to complete its device handshake.
3. **`extra1`–`extra4` reserved layers** — Studio enumerates all layers at boot including `status = "reserved"` ones. Removing them causes Studio initialization to stall, which blocks HID in this firmware fork.

Removing any of these three things will likely produce a Clique firmware where the keyboard sends no key events to the computer even though the non-Clique firmware works fine.

### Layer index discipline

Never renumber layers without updating **all** `&lt`, `&mo`, `&tog`, and `&to` references throughout the keymap. The ten layers are tightly coupled by index. When adding a new layer, append it after the reserved layers (i.e., after index 9). Do not insert layers between indices 0–5, as that would shift flykey/num indices and break thumb key activation. Do not insert layers between flykey/num and the reserved block (indices 4–5 vs 6–9), as that would shift the reserved layer indices within Studio.

### Flykey layer changes

The flykey layout is intentionally symmetric around macOS shortcut conventions:
- Keep deletion shortcuts on the **left hand** (near the editing hand).
- Keep cursor movement on the **right hand** (mirror of a standard arrow cluster).
- Prefer `LA()` / `LG()` macOS variants over raw `LC()` equivalents where both exist, to preserve feel in GUI apps.
- The `LC(A)` / `LC(E)` / `LC(U)` / `LC(K)` bindings are intentional Emacs/Unix additions that also work in macOS terminal and native text inputs — do not replace them with GUI equivalents.

### Homerow mods tuning

The `hm` behavior uses the "timeless HRM" pattern (`balanced` + `require-prior-idle-ms = <150>`). If accidental mods occur during fast typing, increase `require-prior-idle-ms` (try 175 or 200). If mods feel sluggish after a typing pause, decrease it (try 125 or 100). The `hm_hp` behavior uses `hold-preferred` for dedicated modifier keys — if it triggers hold too eagerly, consider switching it to `balanced`.

### `lt_b` (layer-tap) on thumb keys

Three thumb keys use `&lt_b` (custom hold-preferred layer-tap, `tapping-term-ms = 200`, `quick_tap_ms = 175`):
- Both **outer Space** keys (left pos 65, right pos 70): `&lt_b 4 SPACE` — tap = Space, hold = Flykey layer
- **Left middle** (pos 66): `&lt_b 5 BACKSPACE` — tap = Backspace, hold = Num layer

The `hold-preferred` flavor activates the layer as soon as another key is pressed while the thumb key is held, even before the timeout expires. No `quick_tap_ms` is set — unlike the homerow mod behaviors, `quick_tap_ms` on a Space key would block flykey activation when holding Space immediately after typing a word ending in Space. The tap keycode must match the key's primary role. Do not change the tap keycode without also updating physical label expectations.

### `keymap.json` and `info.json` sync

`config/keymap.json` is the GUI-editor representation of the keymap. After manually editing `adv360.keymap`, update `keymap.json` to match so the Nick Coutsos keymap editor remains usable. `info.json` describes physical key positions and rarely needs changing unless hardware layout changes.

## Key Constraints

- **No test suite** — validation is done by building and flashing to hardware.
- The `config/version.dtsi` file is regenerated on every build; don't manually edit it.
- `firmware/*.uf2` files are gitignored.
- Flashing requires putting each half into bootloader mode separately (hold reset while connecting USB).

---

## Session State (last updated 2026-03-24)

### Current Layer Names

| Index | DTS node | Display name | Activated by |
|-------|----------|--------------|--------------|
| 4 | `flykey` | — | `&lt_b 4 SPACE` |
| 5 | `num` | **Util** | `&lt_b 5 BACKSPACE` or `&lt 5 ENTER` |
| 6 | `plain` | **NoFly** | `&to 6` from flykey **left Space** (hold right Space, tap left Space) |

### Key Findings and Decisions

| Finding | Decision |
|---------|----------|
| Root cause of flykey bug: `DT_INST_FOREACH_CHILD_STATUS_OKAY` skips `status = "reserved"` nodes | Workaround: moved flykey → index 4, util → index 5 (before reserved layers). **Committed and working.** |
| Flykey modifier combos (`LA(LEFT)` etc.) sent wrong HID sequence on macOS — standalone modifier tap visible | Replaced all 16 flykey modifier bindings with `&macro_press/&macro_tap/&macro_release` macros. **Committed and working.** |
| `lt` built-in has no `quick_tap_ms` → holding Space fires flykey | Added custom `lt_b` (hold-preferred, no `quick_tap_ms`). Intentionally no quick-tap to avoid swallowing the flykey hold trigger after a space-ending word. |
| Util layer expanded beyond just numpad | Left hand: pure home-row mods (A/S/D/F=⌃/⌥/⌘/⇧, G=Space) + bottom-row shortcuts (Z-B=Undo/Cut/Copy/Paste/Redo). Right hand: numpad (+/7-9/*/0/4-6/=/−/1-3/). **Committed.** |
| ZMK upstream patch (`zmk-fix-keymap-layer-reordering.patch`) also fixes the bug properly for Studio builds | Patch authored; can't push — proxy only authorized for `chaoy/Adv360-Pro-ZMK`, not `chaoy/zmk`. |
| NoFly accidental activation during thumb switching in flykey | Moved `&to 6` from Space keys to Backspace/Enter thumb keys in flykey layer. Space keys now `&trans` (pass-through to `&lt_b 4 SPACE`). **Superseded — see below.** |
| Home-row mods hard to trigger as modifiers | `hm` switched to balanced + `require-prior-idle-ms=150` ("timeless HRM"). Outer/thumb keys: `hm_b` renamed to `hm_hp` (hold-preferred). Right quote key (`'`) now plain `&kp SQT`. **Committed.** |
| NoFly *still* triggered accidentally on Backspace/Enter (frequent during editing) | NoFly entry restricted to **left Space tap only** (flykey pos 65 = `&to 6`). The only way to reach it is hold right Space + tap left Space — you cannot hold and tap the same thumb. Backspace/Enter in flykey reverted to `&trans`. **Committed.** |
| NoFly exit worked only one direction (hold right Enter + tap left Backspace) | Added `&to 0` to right Enter (Util pos 69) to mirror left Backspace. Exit now works either way: hold either thumb key, tap the other. **Committed.** |
| Bottom-row left keys (backtick, Esc) underused | Remapped Base pos 61/62 from `GRAVE`/`ESC` to `ESC`/`CAPS`. **Committed.** |
| Want macOS Globe keycode for apps that bind to Globe (e.g. Typeless) | `refil/zmk@adv360-z3.5-2` defines `GLOBE` = `C_AC_NEXT_KEYBOARD_LAYOUT_SELECT` (consumer 0x029D). Bound `&kp GLOBE` on flykey pos 58 (`/`). **Caveat:** sends consumer "next input source"; does NOT set the Fn modifier flag / keycode 0x3F, so apps that detect Globe via the Fn flag may not recognize it. Needs hardware test. **Committed.** |

### Blocked / Pending Items

- **ESC placement** — Q+W combo (positions 15+16) implemented with 50ms timeout. Other options were deferred.
- **ZMK fork patch** (`chaoy/zmk`, branch `adv360-z3.5-2`) — blocked; no proxy access to that repo. User must manually apply `zmk-fix-keymap-layer-reordering.patch` (steps in `/root/.claude/plans/parsed-growing-lampson.md`). After that, update `config/west.yml` to point at the fork.
- **Space-repeat evaluation** — user needs to test on hardware whether the current `lt_b` (hold-preferred, no `quick_tap_ms`) meets needs or whether adding `quick_tap_ms = <175>` is acceptable despite the edge-case conflict with flykey-after-space.

### Side Investigations

- `west.yml` still points to `ReFil/zmk` (not `chaoy/zmk`) — no build impact until the ZMK patch is needed for Studio.
- Q+W combo for ESC defined (positions 15+16, 50ms timeout).
- `keymap.json` has not been updated to reflect the new layer ordering, `lt_b` bindings, or util layer changes — GUI editor will be out of sync until synced.
