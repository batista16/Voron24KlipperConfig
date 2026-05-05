# Known-good baseline — 2026-04-30

Snapshot of the printer software stack at a known-working state. If a future change breaks something, this is what to roll back to.

This file lives inside `printer_data/config/` so it's versioned with the rest of your config and survives an SD card swap.

## Hardware

- Raspberry Pi 4 Model B Rev 1.5 (4GB), Debian 12 bookworm, kernel 6.6.51, aarch64
- Hostname: `v2`
- Printer: Voron 2.4 350mm corexy, quad-Z, Stealthchanger toolchanger (T0 + T1)
- MCU: single STM32F446 (`/dev/serial/by-id/usb-Klipper_stm32f446xx_33002D000750534E4E313020-if00`)

## Stack versions (all installed via KIAUH at `~/`)

| Component | Branch | Commit | Date |
| --- | --- | --- | --- |
| klipper | master | `5666b88c6` | 2025-06-08 — "ar100: Convert to or1k-elf toolchain" |
| moonraker | master | `6d65a91` | 2025-06-11 — "analysis: include route prefix in dump url" |
| mainsail | (web release) | v2.13.2 | — |
| mainsail-config | master | `ff3869a` | 2024-11-24 — "feat: add _CLIENT_LINEAR_MOVE macro (#39)" |
| klipper-toolchanger-easy | main | `ab34c48` | 2025-05-21 — "Merge PR #2 from VIN-y/main" |
| klipper_auto_speed | main | `6331531` | 2025-02-25 — "Merge PR #37 from FileNotFound7/main" |
| klipper-led_effect | master | `56d1d48` | 2024-10-01 — "Fix paths for manual installation" |
| crowsnest | master | `f9323cb` | 2025-05-27 — "docs(changelog): update changelog" |
| sonar | main | `5eaf906` | 2025-03-02 — "fix: remove old description in make help" |
| mobileraker_companion | main | `5abf2f1` | 2025-04-27 — "Update Configuration_Updates.md" |
| moonraker-timelapse | main | `c7fff11` | 2023-12-16 — "Feature: Add user configurable delayed_gcode interval (#…)" |
| katapult | master | `bc1ecea` | 2025-03-31 — "flashtool: handle situation where software version is no…" |
| kiauh | master | `b99e661` | 2025-03-19 — "feat(ci): add automated release workflow…" |

To roll any of these back to this baseline:
```
git -C ~/<repo> fetch origin && git -C ~/<repo> checkout <commit>
```

**Update caveat:** ktc-easy is sensitive to Klipper API changes. Don't bump Klipper independently of ktc-easy — verify ktc-easy's compatibility note for the target Klipper commit first.

## Manually-placed files (invisible to `update_manager` / `git`)

These files live inside other repos' working trees but aren't tracked by those repos. They survive `git pull` of the host repo but won't be updated automatically.

In `~/klipper/klippy/extras/` (untracked relative to klipper repo):
- `auto_speed.py`, `bed_thermal_adjust.py`, `led_effect.py`, `rounded_path.py`, `save_babies.py`, `tool.py`, `tool_probe.py`, `tool_probe_endstop.py`, `toolchanger.py`, `tools_calibrate.py` — symlinks to third-party repos
- `gcode_shell_command.py` — **dropped-in file, not a symlink, not version-tracked anywhere on this Pi**
- `autospeed/` — **dropped-in directory, not a symlink, not version-tracked anywhere on this Pi**

In `~/moonraker/moonraker/components/` (untracked relative to moonraker repo):
- `timelapse.py` — dropped-in from moonraker-timelapse install

In `~/klipper-toolchanger-easy/klipper/extras/` (untracked relative to ktc-easy repo):
- `save_babies.py` — manually placed; ktc-easy's git tree doesn't expect it here

If ktc-easy / klipper / moonraker get re-cloned, these dropped-in files will be lost. Worth either re-vendoring them properly or just keeping a tarball backup.

**Override pattern for ktc-easy stealthchanger configs**: most files under `printer_data/config/stealthchanger/toolchanger/` are symlinks to `~/klipper-toolchanger-easy/stealthchanger/`. Don't edit them in place (it dirties the ktc-easy git tree and conflicts with future updates). Instead, redefine the section in `printer_data/config/stealthchanger/toolchanger-config.cfg` — that file is loaded last and Klipper merges same-named sections, with later definitions overriding earlier keys. Existing examples: `[toolchanger] params_pickup_path` (custom YY-wiggle) and `[tool_probe_endstop] crash_gcode` (PAUSE instead of M84, added 2026-05-05).

## `printer_data/config` repo state

- Remote: `github.com/batista16/Voron24KlipperConfig` (public)
- HEAD at baseline: `5923e475d` ("Autocommit from 2026-04-07 15:51:20")
- Recovered from git object corruption on 2026-04-30 (power-failure damage). Pre-recovery backup at `~/printer_data/config.bak-2026-04-30`.

## Service inventory

Running: `klipper`, `moonraker`, `crowsnest`, `mobileraker`, `nginx` (mainsail).
Not running: `sonar` (in crash loop, exit code 2 — wifi keepalive, not print-critical).

## Update this file when

- You knowingly upgrade any pinned component.
- You add or remove a third-party Klipper/Moonraker extra.
- You change the GitHub remote.
- You replace the SD card or migrate to a new Pi.
