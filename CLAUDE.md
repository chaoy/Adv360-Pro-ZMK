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
| `config/adv360.keymap` | Main keymap (6 layers: Base, Keypad, Function, Modifier, Flykey, Num) |
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
| 5 | `num` | — | `&lt_b 5 BACKSPACE` (hold left Backspace thumb key) |
| 6 | `extra1` | Red | Reserved for ZMK Studio / Clique |
| 7 | `extra2` | Purple | Reserved for ZMK Studio / Clique |
| 8 | `extra3` | Cyan | Reserved for ZMK Studio / Clique |
| 9 | `extra4` | Yellow | Reserved for ZMK Studio / Clique |

**Why flykey/num come before the reserved layers:** The non-Studio firmware uses `DT_INST_FOREACH_CHILD_STATUS_OKAY_SEP` to enumerate layers, which **skips** nodes with `status = "reserved"`. If flykey were at DTS index 8 (after 4 reserved layers), only 6 layers would be compiled in non-Studio builds and `&lt_b 8` would silently fail. By placing flykey at DTS index 4 and num at 5, both Studio and non-Studio builds use the same layer indices. The reserved layers are appended last — Studio includes them via `DT_INST_FOREACH_CHILD_SEP` (all nodes); non-Studio skips them with no effect since no binding references indices 6–9.

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
Row 4 (NM,.): Home  PgUp   PgDn   End              — page navigation
```

### Left-hand editing layout

```
Row 2 (QWERT): LA(↑)  LG(⌫)  LA(⌫)  LA(⌦)  LG(⌦)  — move-line, word/line delete
Row 3 (ASDFG): LA(↓)  LC(U)   ⌫      ⌦     LC(K)   — kill-line, backspace, delete
Row 4 (ZXCVB):  Esc  LG(↑)  LG(↓)   Ret    Tab     — escape, doc nav, confirm
```

### Num layer layout

Activated by holding the left Backspace thumb key (`&lt_b 5 BACKSPACE`). Numpad on the right hand, arithmetic operators on the outer column:

```
Right row 2 (UIOP+): 7   8   9   *
Right row 3 (HJKL=): 4   5   6   =
Right row 4 (NM,./): 1   2   3   /
Right inner thumb:   0
Left outer column:   +   (Y position)   -   (H position)
```

### Dual-role Keys (`hm` and `hm_b` behaviors)

Two hold-tap behaviors are used for dual-role keys, both with `tapping-term-ms = 200` and `quick_tap_ms = 175`:

- **`hm`** (`tap-preferred`) — for home-row mods and top thumb cluster. Tap registers if released before timeout or if another key is pressed and released within the window. Best for fast typing on frequently-used keys.
- **`hm_b`** (`balanced`) — for outer edge keys (pinky ctrl columns, shift columns). Hold triggers when another key is pressed **and released** while the key is still held, even before timeout. Better for keys that are naturally held while typing another key.

**Home row (uses `hm` — tap-preferred):**

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

**Outer pinky column (uses `hm_b` — balanced):**

| Physical key | Tap | Hold |
|---|---|---|
| Left of A (outer pinky) | `` ` `` / `~` | Left Control |
| Right of ; (outer pinky) | `'` / `"` | Right Control |

**Outer shift column (uses `hm_b` — balanced):**

| Physical key | Tap | Hold |
|---|---|---|
| Left of Z | `\` / `\|` | Left Shift |
| Right of / | `/` / `?` | Right Shift |

**Top thumb cluster (uses `hm` — tap-preferred):**

| Physical key | Tap | Hold |
|---|---|---|
| Left thumb outer (pos 35) | `[` / `{` | Left Alt |
| Left thumb inner (pos 36) | `]` / `}` | Left Command |
| Right thumb inner (pos 37) | `-` / `_` | Right Command |
| Right thumb outer (pos 38) | `=` / `+` | Right Alt |

To tune timing: adjust `tapping-term-ms` (increase for slower typists) or `quick_tap_ms` in the behavior definitions in `adv360.keymap`. The `hm` and `hm_b` behaviors can be tuned independently.

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

If homerow mods cause accidental modifier triggers, adjust `tapping-term-ms` (increase for slower typists) or `quick_tap_ms` in the `hm` behavior definition in `adv360.keymap`. Do **not** change `flavor` from `tap-preferred` without testing — other flavors (`hold-preferred`, `balanced`) interact differently with fast typing.

### `lt_b` (layer-tap) on thumb keys

Three thumb keys use `&lt_b` (custom balanced layer-tap, `tapping-term-ms = 200`, `quick_tap_ms = 175`):
- Both **outer Space** keys (left pos 65, right pos 70): `&lt_b 4 SPACE` — tap = Space, hold = Flykey layer
- **Left middle** (pos 66): `&lt_b 5 BACKSPACE` — tap = Backspace, hold = Num layer

The `balanced` flavor ensures the layer activates when another key is pressed and released while the thumb key is held, even before the timeout. The tap keycode must match the key's primary role. Do not change the tap keycode without also updating physical label expectations.

### `keymap.json` and `info.json` sync

`config/keymap.json` is the GUI-editor representation of the keymap. After manually editing `adv360.keymap`, update `keymap.json` to match so the Nick Coutsos keymap editor remains usable. `info.json` describes physical key positions and rarely needs changing unless hardware layout changes.

## Key Constraints

- **No test suite** — validation is done by building and flashing to hardware.
- The `config/version.dtsi` file is regenerated on every build; don't manually edit it.
- `firmware/*.uf2` files are gitignored.
- Flashing requires putting each half into bootloader mode separately (hold reset while connecting USB).
