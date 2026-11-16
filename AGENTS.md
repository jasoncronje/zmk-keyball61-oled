# Agent guide

ZMK config for the Keyball61 with the original OLED screens: bought PCB, split, two nice!nano v2, trackball on the right half. The nice!view setup lives in a separate repo, `jasoncronje/zmk-keyball61`; changes made here do nothing on that keyboard. `README.md` covers features and hardware. This file holds only what the code doesn't tell you.

## Which build to flash

`build.yaml` builds one shield, `keyball61`.

- Left: `keyball61_left nice_oled-nice_nano_v2-zmk.uf2`
- Right: `keyball61_right nice_oled-nice_nano_v2-zmk.uf2`, or `keyball61_right-keylog-nice_nano_v2-zmk.uf2` if the user runs the key logger (`tools/keylog`), or logging stops.

## Where things live

- Keymap: `config/keyball61.keymap`. Shield settings: `config/keyball61.conf` and `config/boards/shields/keyball61/*.conf`.
- Trackball, auto-mouse and scroll setup is in `config/boards/shields/keyball61/keyball61_right.overlay`. It uses layers by number (MOUSE 3, SCROLL 4, HSCROLL 5). Update it if the keymap's layer order changes.
- A key's MOUSE-layer binding only runs if its position is in `excluded-positions` in that overlay. Otherwise auto-mouse sees the press first, drops the layer, and the key resolves on DEFAULT.
- If a change also suits the nice!view keyboard, say so; it has to be made again in `jasoncronje/zmk-keyball61`.

## Build, push, flash

- There is no local build. Push, and GitHub Actions builds both halves in 2 to 7 minutes: `gh run list --commit <sha>`, `gh run watch <id> --exit-status`, `gh run download <id> -D <dir>`. Artifact names contain spaces, so quote them.
- A push that touches `config/` makes the keymap-drawer bot push a `[Draw]` commit about 30 s later. Run `git pull --rebase` before the next push.
- Flash by following `.agents/skills/keyball-firmware-flash/SKILL.md`. Never use Finder or `cp`.
- The right half is the central and holds the keymap. Keymap, behavior, trackball and host Bluetooth changes need the right half only. Flash the left only for what it runs itself: its screen, Touch ID power, the split link.
- Never flash `settings_reset` unless asked.
- A passed build or a checked transfer does not prove the change works. Ask the user to test on the keyboard, and wait for the answer before calling it done.

## Working on it

- Screen changes: render them with `python3 modules/status-screen/scripts/preview.py <dir>` and show the PNGs (Markdown images, absolute paths) before building.
- Tap, hold and double-tap timings: read the key log first (`python3 tools/keylog/analyze.py --days 7`). Leave values that were tried and rejected in a comment next to the setting.
- Firmware misbehaves and the cause isn't in the code: get logs from the keyboard, don't guess from the Mac. Build the affected half with the `zmk-usb-logging` snippet and ZMK Studio off, read `/dev/cu.usbmodem*` at 115200, then flash the normal build back.
- Bluetooth timing on macOS: check what the host applied with `/usr/bin/log show --last 10m --predicate 'process == "bluetoothd" AND eventMessage CONTAINS[c] "parameters"'`, not by feel. Plain `log` is a zsh builtin and prints nothing.
- Soldering or probing: name the pad by the label printed on the PCB and where it sits. The PCB prints AVR names, so `pm1` or `P0.06` alone means nothing to the user.
- Tests: `python3 -m unittest discover -s tests`.
