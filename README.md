# PandaKB Sofle RGB MX ZMK Config

My ZMK configuration for the [PandaKB Sofle RGB MX](https://github.com/PandaKBLab/zmk-for-PandaKB/tree/PandaKB_Sofle) keyboard.

## Differences From Upstream PandaKB_Sofle

This comparison is based on `https://github.com/PandaKBLab/zmk-for-PandaKB/tree/PandaKB_Sofle`.

Main differences:

* The repository is focused on the dongle-less SSD1306 OLED version: `build.yaml` keeps `Sofle_L_oled`, `Sofle_R_oled`, and `settings_reset`.
* RGB turns off on idle and USB disconnect so the keyboard does not keep glowing after the host shuts down.
* Configuration files were cleaned up by removing outdated and unused values.
* The right encoder now works as a mouse wheel.
* The left encoder controls volume.
* The base layer is adapted to my personal layout.
