# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A ZMK firmware configuration for a wireless split keyboard: the **Charybdis 4×6** (BastardKB) with a **Prospector dongle** and PMW3610 optical trackball sensor. This is not a traditional software project — there is no local build toolchain. Firmware is built via GitHub Actions using the ZMK GitHub workflow.

## Building Firmware

Firmware builds are triggered automatically on push/pull request via [.github/workflows/build.yml](.github/workflows/build.yml), which delegates to `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`.

There is no local build command. To build firmware:
1. Push changes to the repository (or trigger `workflow_dispatch` manually in GitHub Actions).
2. Download the resulting `.uf2` artifacts from the Actions run.
3. Flash each part (left half, right half, dongle) separately by putting the controller into bootloader mode and copying the `.uf2`.

## Repository Structure

### `config/`
- **`charybdis.keymap`** — the active keymap (6 layers: BASE, NAVNUM, SYMBOL, MEDIA, MOUSE, SCROLL). This is the primary file to edit for keymap changes.
- **`charybdis.conf`** — runtime Kconfig options (BLE power, experimental connection flags).
- **`west.yml`** — west manifest pinning ZMK (`v0.3`), PMW3610 driver (`badjeff/zmk-pmw3610-driver@zmk-0.3`), and the Prospector module (`carrefinho/prospector-zmk-module@main`).

### `boards/shields/charybdis/`
Shield definition files — hardware description, not typically edited for keymap work:
- **`charybdis.dtsi`** — base hardware: matrix transform (12 columns × 5 rows), kscan stub, and `charybdis_6col_layout` physical layout.
- **`charybdis_left.overlay`** — left half: kscan GPIO matrix wiring for Pro Micro pins.
- **`charybdis_right.overlay`** — right half: same kscan wiring + includes `charybdis_3610.dtsi` (PMW3610 trackball via SPI0) and `split_input_common.dtsi` (input split relay).
- **`charybdis_dongle.overlay`** — Prospector dongle: mock kscan (no physical switches), virtual trackball input relay, and ST7789 display over SPI3 (XIAO pins).
- **`charybdis_3610.dtsi`** — PMW3610 trackball sensor definition: SPI0, CPI=800, GPIO interrupt.
- **`split_input_common.dtsi`** — trackball input processing: snipe mode (NAVNUM layer, 1/3 speed), scroll mode (SCROLL layer, Y-inverted, 1/3 speed), auto-mouse layer (BASE/MOUSE, 400ms timeout, 500ms prior-idle).
- **`charybdis.zmk.yml`** — shield metadata; siblings: `charybdis_dongle`, `charybdis_left`, `charybdis_right`.
- **`charybdis.keymap`** — default/fallback keymap included in the shield (minimal, 4 layers). The active user keymap in `config/` takes precedence.

### Key Modules (from `config/west.yml`)
- **`zmk`** — ZMK firmware at `v0.3`
- **`zmk-pmw3610-driver`** (badjeff) — driver for the PMW3610 optical sensor (`pixart,pmw3610-alt` compatible string)
- **`prospector-zmk-module`** (carrefinho) — Prospector dongle support

## Keymap Layers (config/charybdis.keymap)

| Index | Name   | Purpose |
|-------|--------|---------|
| 0     | BASE   | Dvorak layout with home-row mod-taps (`hml`/`hmr`) |
| 1     | NAVNUM | Navigation (arrows, home/end/pg) + numpad, F-keys |
| 2     | SYMBOL | Symbols, brackets, operators |
| 3     | MEDIA  | Media controls |
| 4     | MOUSE  | Mouse button layer (auto-activated by trackball) |
| 5     | SCROLL | Trackball scroll mode |

Key behaviors defined in the keymap:
- `hml`/`hmr` — home-row mod-taps (balanced, 2000ms tapping term, 250ms prior-idle, one-sided trigger positions)
- `hml_s`/`hmr_s` — same but `require-prior-idle-ms = 0` for Shift
- `smart_shift` — tap for sticky shift, hold for caps_word
- `u_ltm` — hold-tap: layer momentary / mouse button

## Editing the Keymap

The [Keymap Editor](https://nickcoutsos.github.io/keymap-editor/) can be used as a GUI for `config/charybdis.keymap`. Connect it to this GitHub repo for a visual editing experience. Keymap Drawer was used to generate [docs/keymap.svg](docs/keymap.svg).

When editing `.keymap` files, key positions in combos and `hold-trigger-key-positions` refer to the indices in the `charybdis_6col_layout` position map defined in `charybdis.dtsi`. Left-hand key indices are `KEYS_L = 0–4, 10–14, 20–24`; right-hand are `KEYS_R = 5–9, 15–19, 25–29`; thumbs are `THUMBS = 30–35`.
