# AGENTS.md

This repository is a ZMK configuration and local shield definition for a PandaKB Sofle split keyboard.

It builds two keyboard firmware artifacts plus a `settings_reset` artifact:

```yaml
include:
  - board: nice_nano_v2
    shield: Sofle_L nice_oled
    snippet: studio-rpc-usb-uart
    artifact-name: Sofle_L_oled
  - board: nice_nano_v2
    shield: Sofle_R nice_oled
    artifact-name: Sofle_R_oled
  - board: nice_nano_v2
    shield: settings_reset
```

The repo is not the full ZMK source tree. It is the user config plus a local `Sofle` shield that ZMK discovers through `zephyr/module.yml`.

## What This Repo Contains

```text
.
├── build.yaml
├── config/
│   ├── west.yml
│   ├── Sofle.keymap
│   ├── Sofle.conf
│   └── Sofle.json
├── boards/
│   └── shields/
│       └── Sofle/
│           ├── Kconfig.shield
│           ├── Kconfig.defconfig
│           ├── Sofle.dtsi
│           ├── Sofle-layouts.dtsi
│           ├── Sofle_L.overlay
│           ├── Sofle_R.overlay
│           ├── Sofle_L.conf
│           ├── Sofle_R.conf
│           ├── Sofle.keymap
│           └── Sofle.zmk.yml
└── zephyr/
    └── module.yml
```

## Repository-Specific Model

This is a split wireless Sofle keyboard with:

* `nice_nano_v2` controllers.
* Local shield siblings `Sofle_L` and `Sofle_R`.
* `nice_oled` stacked in both keyboard build targets.
* A `settings_reset` build target kept from upstream for clearing ZMK settings.
* ZMK Studio enabled for the left half via `snippet: studio-rpc-usb-uart`.
* OLED display support through the local shield and `zmk-nice-oled` module.
* Per-key/underglow WS2812 RGB configured in `Sofle.dtsi`.
* EC11 encoders on both halves.
* Battery reporting via `zmk,battery-nrf-vddh` in each half overlay.
* A physical layout definition for Studio in `Sofle-layouts.dtsi` and `config/Sofle.json`.

The left half is central by default:

```dts
config ZMK_SPLIT_ROLE_CENTRAL
    default y
```

## Most Common Change Locations

Use the smallest file change that matches the request:

```text
Change user key behavior        -> config/Sofle.keymap
Change layers/combos/macros     -> config/Sofle.keymap
Change encoder actions          -> config/Sofle.keymap sensor-bindings
Change firmware feature flags   -> config/Sofle.conf
Change build targets/shields    -> build.yaml
Change ZMK/module dependencies  -> config/west.yml
Change matrix pins              -> boards/shields/Sofle/Sofle_L.overlay or Sofle_R.overlay
Change shared hardware          -> boards/shields/Sofle/Sofle.dtsi
Change physical layout/Studio   -> boards/shields/Sofle/Sofle-layouts.dtsi and possibly config/Sofle.json
Change shield metadata          -> boards/shields/Sofle/Sofle.zmk.yml
```

For ordinary keymap edits, do not touch `build.yaml`, `config/west.yml`, `zephyr/module.yml`, or hardware files.

## Keymap Notes

There are two `Sofle.keymap` files, and they have different roles:

* `config/Sofle.keymap` is the active user keymap with this repo's personal layout changes. When the user asks to change the keymap, change this file first unless they explicitly say they want the shield default changed.
* `boards/shields/Sofle/Sofle.keymap` is the default keymap that belongs to the local keyboard shield. Treat it as a baseline/example for the hardware definition, not as the primary user layout.

The active user keymap applies to both halves.

Current layers:

```dts
#define BASE 0
#define LOWER 1
#define RAISE 2
#define ADJUST 3
```

`ADJUST` is activated by pressing `LOWER` and `RAISE` together using `zmk,conditional-layers`.

Important bindings already present:

* Bluetooth controls are on `RAISE` and `ADJUST`; keep profile select/clear reachable.
* External power toggle is on `LOWER`/`ADJUST`.
* RGB controls are on `ADJUST`; base layer has `&rgb_ug RGB_TOG`.
* Encoders use `sensor-bindings`; current user keymap uses volume on the left encoder and mouse wheel style scroll on the right encoder.
* `config/Sofle.keymap` includes `<dt-bindings/zmk/pointing.h>` and defines `inc_dec_msc` for encoder mouse scroll.

Preserve the matrix-shaped formatting in `bindings = < ... >;`. Each layer should keep the same binding count as the transform in `boards/shields/Sofle/Sofle.dtsi`.

If a change must keep the default shield keymap aligned with the user keymap, say so explicitly and update both files intentionally.

## Config Notes

`config/Sofle.conf` currently configures:

* Battery report interval.
* Kscan debounce.
* BLE TX power.
* Keyboard name `PandaKB_Sofle`.
* Windows battery-report workaround via `CONFIG_BT_GATT_ENFORCE_SUBSCRIPTION=n`.
* OLED display and custom status screen support.
* Idle/deep sleep.
* ZMK Studio.
* Pointing support.
* RGB underglow/WS2812 behavior.
* EC11 encoder support.

Be conservative with power-hungry changes because this is a wireless nice!nano build. Avoid enabling verbose logging, high scan rates, bright RGB defaults, or disabling sleep unless the user explicitly asks.

Do not invent Kconfig symbols. Check existing ZMK docs or nearby working config before adding options.

## Local Shield Notes

`boards/shields/Sofle/` is not optional decoration; it is how this custom `Sofle` shield exists.

Important files:

* `Kconfig.shield` declares `SHIELD_SOFLE_L` and `SHIELD_SOFLE_R`.
* `Kconfig.defconfig` enables split behavior, makes `Sofle_L` central, and supplies OLED/LVGL defaults.
* `Sofle.dtsi` defines the shared matrix transform, rows, encoders, OLED, RGB SPI strip, and physical layout selection.
* `Sofle_L.overlay` and `Sofle_R.overlay` define side-specific column GPIO order, enable the side's encoder, and set battery reporting.
* `Sofle-layouts.dtsi` defines the physical key positions used by ZMK Studio.
* `Sofle.zmk.yml` declares metadata, features, required controller, and siblings.

Do not change pin assignments, row/column order, encoder pins, OLED pins, RGB chain length, or transform mappings unless the task is explicitly about hardware or a known hardware bug.

If key positions are changed for Studio, keep `Sofle-layouts.dtsi`, `Sofle.dtsi`, and `config/Sofle.json` consistent.

## Dependency Notes

`config/west.yml` uses:

* ZMK from `zmkfirmware/zmk`.
* `zmk-nice-oled` from `mctechnology17/zmk-nice-oled`.
* Default revision `v0.3`, with `zmk-nice-oled` pinned to `main`.
* `self.path: config`.

Do not casually change ZMK revisions or external modules. If dependency versions are changed, call out the build impact clearly.

`zephyr/module.yml` sets:

```yaml
build:
  settings:
    board_root: .
```

Do not remove it. Without this module metadata ZMK may not discover `boards/shields/Sofle`.

## Build And Validation

Preferred build commands, from the workspace containing the ZMK checkout:

```bash
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD="Sofle_L nice_oled" -DZMK_CONFIG="$PWD/config"
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD="Sofle_R nice_oled" -DZMK_CONFIG="$PWD/config"
```

For the Studio-enabled left artifact, include the snippet if the local build environment supports it:

```bash
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD="Sofle_L nice_oled" -DSNIPPET=studio-rpc-usb-uart -DZMK_CONFIG="$PWD/config"
```

If local ZMK dependencies are not available, at least validate by inspection:

* `build.yaml` still contains `Sofle_L nice_oled`, `Sofle_R nice_oled`, and `settings_reset`.
* Keymap layers have the correct binding count and balanced `< ... >`.
* Devicetree braces and semicolons are balanced.
* `.conf` syntax is valid `CONFIG_NAME=value`.
* No generated firmware artifacts were added.

## Documentation Sync

Before committing repo-shaping changes, update documentation so future agents and humans do not work from stale assumptions.

Update `README.md` and this `AGENTS.md` when changes affect:

* build targets or artifact names in `build.yaml`;
* which keymap is primary versus default shield keymap;
* supported hardware variants, such as dongle-less, dongle, OLED, nice!view, RGB, encoders, or pointing;
* ZMK or module dependencies in `config/west.yml`;
* local shield structure or hardware responsibilities under `boards/shields/Sofle/`;
* build, validation, flashing, or reset instructions.

For small key binding edits inside `config/Sofle.keymap`, documentation usually does not need to change unless the README explicitly describes that behavior.

## Generated Files

Do not commit generated build output:

```text
build/
.zmk-build/
*.uf2
*.hex
*.elf
*.map
*.bin
*.log
```

Firmware artifacts should normally be produced by GitHub Actions or a local `west build`, then flashed separately.

## Response Expectations

When making changes in this repo:

1. State which files changed.
2. Explain why those files are the right place for the change.
3. Mention whether both halves need rebuilding/flashing.
4. Include exact validation or build commands when useful.
5. Call out uncertainty instead of guessing ZMK option names.
