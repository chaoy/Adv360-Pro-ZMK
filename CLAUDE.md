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

The keymap defines 6 layers. Layer indices in ZMK bindings are zero-based:

| Index | Name | Display | Activated by |
|-------|------|---------|--------------|
| 0 | `default_layer` | Base | Always on |
| 1 | `keypad` | Kp | `&tog 1` (toggle on number row) |
| 2 | `fn` | Fn | `&mo 2` (hold left/right thumb corners) |
| 3 | `mod` | Mod | `&mo 3` (hold right upper thumb key) |
| 4 | `flykey` | — | `&lt 4 SPACE` (hold either Space thumb key) |
| 5 | `num` | — | `&lt 5 BACKSPACE` (hold left Backspace thumb key) |

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

Activated by holding the left Backspace thumb key (`&lt 5 BACKSPACE`). Numpad on the right hand, arithmetic operators on the outer column:

```
Right row 2 (UIOP+): 7   8   9   *
Right row 3 (HJKL=): 4   5   6   =
Right row 4 (NM,./): 1   2   3   /
Right inner thumb:   0
Left outer column:   +   (Y position)   -   (H position)
```

### Homerow Mods (`hm` behavior)

Base layer home-row keys use `hm` (a `hold-tap` with `flavor = "tap-preferred"`):

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

Parameters: `tapping-term-ms = 200`, `quick_tap_ms = 175`. The `tap-preferred` flavor means the key registers as a tap if released before `tapping-term-ms` or if another key is pressed and released within that window.

## Guidelines for Modifying the Keymap

### Layer index discipline

Never renumber layers without updating **all** `&lt`, `&mo`, `&tog`, and `&to` references throughout the keymap. The six layers are tightly coupled by index. When adding a new layer, append it after layer 5 and add the activation binding explicitly.

### Flykey layer changes

The flykey layout is intentionally symmetric around macOS shortcut conventions:
- Keep deletion shortcuts on the **left hand** (near the editing hand).
- Keep cursor movement on the **right hand** (mirror of a standard arrow cluster).
- Prefer `LA()` / `LG()` macOS variants over raw `LC()` equivalents where both exist, to preserve feel in GUI apps.
- The `LC(A)` / `LC(E)` / `LC(U)` / `LC(K)` bindings are intentional Emacs/Unix additions that also work in macOS terminal and native text inputs — do not replace them with GUI equivalents.

### Homerow mods tuning

If homerow mods cause accidental modifier triggers, adjust `tapping-term-ms` (increase for slower typists) or `quick_tap_ms` in the `hm` behavior definition in `adv360.keymap`. Do **not** change `flavor` from `tap-preferred` without testing — other flavors (`hold-preferred`, `balanced`) interact differently with fast typing.

### `lt` (layer-tap) on thumb keys

The Space and Backspace thumb keys use `&lt` (layer-tap). The tap keycode must match the key's primary role (Space sends Space, Backspace sends Backspace). Do not change the tap keycode unless also updating the physical label expectations.

### `keymap.json` and `info.json` sync

`config/keymap.json` is the GUI-editor representation of the keymap. After manually editing `adv360.keymap`, update `keymap.json` to match so the Nick Coutsos keymap editor remains usable. `info.json` describes physical key positions and rarely needs changing unless hardware layout changes.

## Key Constraints

- **No test suite** — validation is done by building and flashing to hardware.
- The `config/version.dtsi` file is regenerated on every build; don't manually edit it.
- `firmware/*.uf2` files are gitignored.
- Flashing requires putting each half into bootloader mode separately (hold reset while connecting USB).
