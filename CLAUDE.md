# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a bilingual documentation-only repository. It contains a step-by-step tutorial for flashing Android 11 (X98Q firmware) onto the **Panacom TV-1016** TV box (SoC: Amlogic S905W2, 2 GB RAM, 16 GB eMMC) and configuring the original IR remote control to work with the new firmware.

There is no code, build system, or test suite — only two Markdown files:

- `README.md` — English version
- `README.es.md` — Spanish version

## Structure and conventions

Both files must remain in sync. Any change to one (new step, corrected value, updated command, added note) must be mirrored in the other. The Spanish version uses informal second-person address (*vos* conjugation, Argentine Spanish).

## Key technical constants

These values are specific to this hardware/remote and must stay consistent across both files:

| Item | Value |
|------|-------|
| SoC | Amlogic S905W2 |
| RAM / eMMC | 2 GB / 16 GB |
| Replacement firmware | X98Q Android 11 |
| IR custom_code | `0x7f80` |
| Termux SSH port | 8022 |
| Mouse button keycode (Linux) | 133 (INFO) |
| Mouse button keycode (Android) | 165 (INFO) |

## Tutorial flow

The guide follows this sequence, and edits should preserve the logical dependency order:

1. Flash X98Q firmware via Amlogic USB Burning Tool (Windows required)
2. First boot, WiFi setup, FOTA updates *(must happen before Magisk)*
3. Decode IR codes from `dmesg | grep meson-ir`
4. Install Magisk via CLI boot-patch method *(app GUI patching fails on this hardware)*
5. Create Magisk module (`/data/adb/modules/panacom_remote`) + boot script (`/data/adb/service.d/panacom_remote.sh`) using `/vendor/bin/remotecfg`
6. Optional: MATVT accessibility app for software mouse mode

## Critical warnings preserved in the docs

- FOTA updates must be applied **before** installing Magisk (OTA overwrites the boot partition).
- The Magisk CLI patch method (`boot_patch.sh`) is required; the GUI "Process error" is a known issue on this device.
- The X98Q firmware is compatible because it shares the exact SoC+memory config; other S905W2 firmwares (X98 Mini, etc.) have incompatible WiFi drivers.
- The `custom_code` (`0x7f80`) and scancodes are unit-specific — readers with different remotes must re-run step 4.
