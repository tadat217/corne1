# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

ZMK firmware configuration for a Corne (CRKBD) split keyboard running on Nice!Nano v2 microcontrollers. The build is fully managed by ZMK's CI pipeline — there is no local build toolchain to install.

## Building Firmware

**Trigger a build:** push or open a PR — GitHub Actions runs automatically via `.github/workflows/build.yml`, which delegates to ZMK's official reusable workflow at `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`.

**Build matrix** is defined in `build.yaml`. It builds two shields: `corne_left` and `corne_right`, both with ZMK Studio support.

**Download firmware:** go to the GitHub Actions run → download the artifact zip → flash each `.uf2` to the corresponding half by putting it in bootloader mode (double-tap reset).

## Repository Structure

All user-editable files live in `config/`:

| File | Purpose |
|------|---------|
| `config/corne.keymap` | All key bindings and layer definitions |
| `config/corne.conf` | Feature flags (BLE power, RGB, display, power management) |
| `config/corne.overlay` | Device tree: OLED (I2C), RGB (SPI3), external power |
| `config/corne.dtsi` | Hardware matrix transform and GPIO scan config |
| `config/west.yml` | ZMK version pin (currently `v0.3`) |

`zephyr/module.yml` tells west to treat this repo as a ZMK module. The `boards/` and `zephyr/` dirs are structural stubs required by the module system — not edited directly.

## Keymap Architecture

### Layer Index

| # | Name | Access |
|---|------|--------|
| 0 | Base | default |
| 1 | Navigation | `&lt 1 SPACE` (left thumb hold) |
| 2 | Number | `&lt 2 BACKSPACE` (right thumb hold) |
| 3 | Symbols | `&lt 3 ENTER` (left thumb hold) |
| 4 | Functions | `&lt 4 ESC` (right thumb hold) |
| 5 | Mouse | `&lt 5 X` (left pinky row hold) |
| 6 | Media | `&lt 6 PERIOD` (right ring row hold) |
| 7 | Bluetooth | `&mo 7` (left thumb) |
| 8 | Gaming | `&to 8` (right thumb), back with `&to 0` |

### Key Position Map (0-indexed, left→right, top→bottom)

```
 0   1   2   3   4   5  |  6   7   8   9  10  11
12  13  14  15  16  17  | 18  19  20  21  22  23
24  25  26  27  28  29  | 30  31  32  33  34  35
               36  37  38  | 39  40  41
```

### Home Row Mods (base layer, row 1)

| Position | Key | Modifier |
|----------|-----|----------|
| 13 | A | Left Ctrl |
| 14 | S | Left Alt |
| 15 | D | Left Cmd |
| 16 | F | Left Shift |
| 19 | J | Right Shift |
| 20 | K | Right Cmd |
| 21 | L | Right Alt |
| 22 | ; | Right Ctrl |

Tapping term is set globally: `&mt { tapping-term-ms = <300>; };`

## Hardware Notes

- **RGB:** WS2812 underglow via SPI3, 27 LEDs per half (54 total), GRB color order, max brightness capped at 30 in `.conf`
- **Display:** SSD1306 OLED 128×32 via I2C at address `0x3c`
- **BLE TX power:** +8 dBm
- **Power timeouts:** 60s → idle, 20min → deep sleep

## ZMK-Specific Patterns

- Combos go in a `combos { }` node inside the root `/ { }` block, **before** the `keymap { }` node.
- Custom behaviors (mod-morph, tap-dance, etc.) go in a `behaviors { }` node at the root level.
- When adding combos on HRM keys (mod-taps), prefer plain `&kp` keys for combo positions to avoid hold-tap timing conflicts.
- ZMK Studio is enabled (`CONFIG_ZMK_STUDIO=y`) with locking disabled — live keymap editing works over USB without reflashing.
