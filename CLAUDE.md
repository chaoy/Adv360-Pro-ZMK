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

## Key Constraints

- **No test suite** — validation is done by building and flashing to hardware.
- The `config/version.dtsi` file is regenerated on every build; don't manually edit it.
- `firmware/*.uf2` files are gitignored.
- Flashing requires putting each half into bootloader mode separately (hold reset while connecting USB).
