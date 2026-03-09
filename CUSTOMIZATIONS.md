# Custom Keymap Guide

How to use the modified Kinesis Advantage 360 Pro firmware.

## What's Different

| Customization | How to use |
|---|---|
| **Flykey layer** | Hold either Space key — home-row navigation and editing |
| **Edit layer** | Hold left Backspace or Enter — numpad + mods + edit shortcuts |
| **NoFly layer** | Press Space while in Flykey — disables accidental layer-hops |
| **Home-row mods** | Hold any home-row key ≥200 ms to send a modifier |
| **Caps Lock → Esc** | Caps Lock sends Escape |
| **Q+W → Esc** | Simultaneous Q+W (within 50 ms) sends Escape |

---

## How Layers Switch

```mermaid
flowchart LR
    B([Base\nlayer 0])
    F([Flykey\nlayer 4])
    E([Edit\nlayer 5])
    N([NoFly\nlayer 6])

    B -->|"Hold Space (either)"| F
    F -->|"Release Space"| B
    F -->|"Press Space while held"| N
    N -->|"Press Space or any layer key"| B

    B -->|"Hold left Backspace"| E
    B -->|"Hold Enter"| E
    E -->|"Release Backspace / Enter"| B
    E -->|"Press Backspace"| B
    E -->|"Press Space (either)"| N
```

**Momentary layers** (Flykey, Edit) are active only while you hold the key — release to return.
**NoFly** is different: it *latches*. Hold Space → tap Space → release Space. You stay in NoFly until you press Space or a layer key again.

---

## Base Layer

Diagrams use `key/MOD` for dual-role keys: tap the key for the letter, hold for the modifier.
Modifier abbreviations: `⌃` Control · `⌥` Option/Alt · `⌘` Command · `⇧` Shift

### Main Keys

```
 ←────────────── Left half ──────────────→   ←─────────────── Right half ──────────────→

  =    1    2    3    4    5   [Kp]          [Mo]   6    7    8    9    0    -
 Tab   Q    W    E    R    T                         Y    U    I    O    P    \
`/⌃  A/⌃  S/⌥  D/⌘  F/⇧   G                         H   J/⇧  K/⌘  L/⌥  ;/⌃  '/⌃
\/⇧   Z    X    C    V    B   Hom           PgU      N    M    ,    .    /   //⇧
```

`[Kp]` toggles the Keypad layer.  `[Mo]` holds the Mod layer (Bluetooth, RGB, etc.).
`Hom` / `PgU` are dedicated keys on row 4 center.

### Thumb Cluster

```
 Left thumb                        Right thumb
 ┌──────┬──────┐                   ┌──────┬──────┐
 │ [/⌥  │ ]/⌘  │  ← top row        │ -/⌘  │ =/⌥  │
 └──────┴──────┘                   └──────┴──────┘

 ┌────┬────┬─────┬───┬───┐         ┌─────┬────┬─────┐   ┌───┬───┬───┬───┬────┐
 │ Fn │ `  │ Esc │ ← │ → │         │PgDn │Ent │ End │   │ ↑ │ ↓ │ [ │ ] │ Fn │
 └────┴────┴─────┴───┴───┘         └─────┴────┴─────┘   └───┴───┴───┴───┴────┘
           ┌───────────┬──────────┐         ┌───────────┐
           │   Space   │ Backspace│         │   Space   │
           │ (Flykey)  │  (Edit)  │         │ (Flykey)  │
           └───────────┴──────────┘         └───────────┘
```

Thumb Space keys: **tap** = Space, **hold** = Flykey layer.
Left Backspace: **tap** = Backspace, **hold** = Edit layer.
Enter: **tap** = Enter, **hold** = Edit layer.
`Fn` corner keys: **hold** = Fn layer.

---

## Flykey Layer — Hold Space

Hold either Space key to activate. The left hand handles text editing; the right hand handles cursor movement. Everything else passes through to the Base layer (home-row mods still work).

### Full Keyboard

```
 ←────────────── Left half ──────────────→   ←─────────────── Right half ──────────────→

  ·    ·    ·    ·    ·    ·    ·             ·      ·    ·    ·    ·    ·    ·
  ·   mv↑  ⌘⌫   ⌥⌫   ⌥⌦  ⌘⌦   ·             ·     ⌘←   ⌥←   ↑    ⌥→  ⌘→   ·
  ·   mv↓  ^U   ⌫    ⌦   ^K   ·              ·      ·   ^A   ←    ↓    →   ^E   ·
  ·   Esc  ⌘↑   ⌘↓   ↵    ⇥   ·    ·         Hm  PgU  PgD  End   ·    ·
```

`·` = same as Base.  Thumb Space → **NoFly layer** (instead of normal Space).

### Left Hand — Editing

| Key | What it does |
|-----|-------------|
| Q | Move current line up (⌥↑ — works in VS Code / most editors) |
| W | Delete to start of line (⌘⌫) |
| E | Delete word to the left (⌥⌫) |
| R | Delete word to the right (⌥⌦) |
| T | Delete to end of line (⌘⌦) |
| A | Move current line down (⌥↓) |
| S | Kill to start of line, ^U (Emacs / Unix terminals) |
| D | Backspace |
| F | Forward delete |
| G | Kill to end of line, ^K (Emacs / Unix terminals) |
| Z | Escape |
| X | Jump to top of document (⌘↑) |
| C | Jump to bottom of document (⌘↓) |
| V | Return |
| B | Tab |
| **Space** | **Switch to NoFly layer** |

### Right Hand — Navigation

| Key | What it does |
|-----|-------------|
| Y | Line start — ⌘← in GUI apps |
| U | Word left (⌥←) |
| I | Up |
| O | Word right (⌥→) |
| P | Line end — ⌘→ in GUI apps |
| H | Beginning of line — ^A (Emacs; also works in macOS text fields and terminal) |
| J | Left |
| K | Down |
| L | Right |
| ; | End of line — ^E (Emacs; also works in macOS text fields and terminal) |
| N | Home |
| M | Page up |
| , | Page down |
| . | End |

> **Line start/end:** Use `H`/`;` (^A/^E) in terminal and Emacs; use `Y`/`P` (⌘←/⌘→) in GUI apps like browsers and editors.

---

## Edit Layer — Hold Left Backspace or Enter

Hold the left Backspace or Enter thumb key. The right hand becomes a numpad; the left hand provides pure modifier keys and common edit shortcuts.

### Full Keyboard

```
 ←────────────── Left half ──────────────→   ←─────────────── Right half ──────────────→

  ·    ·    ·    ·    ·    ·    ·             ·      ·    +    7    8    9    *    ·
  ·    ·    ·    ·    ·    ·                          ·    0    4    5    6    =    ·
  ·    ⌃    ⌥    ⌘    ⇧   Spc  ·              ·       ·   -    1    2    3    /    ·
  ·   Undo Cut  Copy Pst  Redo  ·    ·
```

`·` = same as Base.  Thumb cluster:

```
           ┌───────────┬──────────┐         ┌───────────┐
           │  NoFly    │  →Base   │         │  NoFly    │
           │  (Space)  │  (Bsp)   │         │  (Space)  │
           └───────────┴──────────┘         └───────────┘
```

### Left Hand

**Home row** — pure modifiers (no letter output, no tap delay):

| Key | Output |
|-----|--------|
| A | ⌃ Control |
| S | ⌥ Option |
| D | ⌘ Command |
| F | ⇧ Shift |
| G | Space |

**Bottom row** — edit shortcuts (⌘-chord macros):

| Key | Action |
|-----|--------|
| Z | Undo (⌘Z) |
| X | Cut (⌘X) |
| C | Copy (⌘C) |
| V | Paste (⌘V) |
| B | Redo (⌘⇧Z) |

**Thumb:**
- Left Space → NoFly layer
- Backspace → back to Base (`&to 0`)
- Right Space → NoFly layer

### Right Hand — Numpad

```
 ←── Right: numpad ────────────────→

  ·    +    7    8    9    *    ·
  ·    0    4    5    6    =    ·
  ·    -    1    2    3    /    ·
```

- `+` is under the Y key (left column of right hand, row 2)
- `0` is under H (left column of right hand, row 3)
- `-` is under N (left column of right hand, row 4)

---

## NoFly Layer — Tap Space while in Flykey

**Problem it solves:** When typing fast, holding Space briefly while reaching for the next key can accidentally trigger the Flykey layer, sending navigation commands instead of characters.

**How to enter:** While holding Space (Flykey active), tap Space again, then release.

### Full Keyboard

Most keys behave as Base. The changes are in the outer columns and thumb cluster:

```
 ←────────────── Left half ──────────────→   ←─────────────── Right half ──────────────→

  ·    ·    ·    ·    ·    ·    ·             ·      ·    ·    ·    ·    ·    ·
  ·    ·    ·    ·    ·    ·                          ·    ·    ·    ·    ·    ·
  ⌃   A/⌃  S/⌥  D/⌘  F/⇧   ·   ⌥    ⌘      ⌥   ⌘   ·   J/⇧  K/⌘  L/⌥  ;/⌃   '
  ⇧    ·    ·    ·    ·    ·                          ·    ·    ·    ·    ·    ⇧
```

**What changes:**
- Outer-column keys that had dual tap/hold roles now send their modifier directly (no tap symbol).
- Space thumb keys send plain Space — **no Flykey activation**.
- Home-row mods (A/S/D/F/J/K/L/;) are **unchanged** — still work as normal.

**How to leave:** Press either Space or any layer key (Fn, Kp, etc.).

---

## Quick Reference

### Flykey shortcuts by category

**Cursor movement**

| Flykey key | Action | Shortcut |
|---|---|---|
| J | Left | ← |
| K | Down | ↓ |
| I | Up | ↑ |
| L | Right | → |
| U | Word left | ⌥← |
| O | Word right | ⌥→ |
| Y | Line start (GUI) | ⌘← |
| P | Line end (GUI) | ⌘→ |
| H | Line start (terminal) | ^A |
| ; | Line end (terminal) | ^E |
| N | Home | Home |
| . | End | End |
| M | Page up | PgUp |
| , | Page down | PgDn |
| X | Document top | ⌘↑ |
| C | Document bottom | ⌘↓ |

**Deletion**

| Flykey key | Action | Shortcut |
|---|---|---|
| D | Backspace | ⌫ |
| F | Delete | ⌦ |
| E | Delete word left | ⌥⌫ |
| R | Delete word right | ⌥⌦ |
| W | Delete to line start | ⌘⌫ |
| T | Delete to line end | ⌘⌦ |
| S | Kill to line start | ^U |
| G | Kill to line end | ^K |

**Other**

| Flykey key | Action |
|---|---|
| Z | Escape |
| V | Return |
| B | Tab |
| Q | Move line up |
| A | Move line down |
| Space | Enter NoFly layer |

### Edit layer shortcuts

**Left hand modifiers** (pure — no letter output)

| Key | Modifier |
|---|---|
| A | ⌃ Control |
| S | ⌥ Option |
| D | ⌘ Command |
| F | ⇧ Shift |
| G | Space |

**Left hand edit shortcuts**

| Key | Action |
|---|---|
| Z | Undo ⌘Z |
| X | Cut ⌘X |
| C | Copy ⌘C |
| V | Paste ⌘V |
| B | Redo ⌘⇧Z |
