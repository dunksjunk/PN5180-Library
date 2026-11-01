# Tested hardware

Boards the library has been run on with a real PN5180 module and tags. CI only compiles the
examples (for `arduino:avr:uno` and `esp32:esp32:esp32`). A board is listed here only after
it has been checked on hardware. A board that just compiles does not count as tested.

Status values:

- **Verified**: the listed examples and tags worked on this board with the listed library version
- **Partial**: some features worked; see the notes
- **Pending**: the board is identified and wired up, but nothing has been verified yet
- **Not working**: tried and failed; see the notes

## Boards

| Board | MCU / module | Flash / PSRAM | USB bridge | FQBN | Core | Library | Status |
|---|---|---|---|---|---|---|---|
| [ELEGOO ESP-32 Dev Board (ESP-WROOM-32)](#elegoo-esp-32-dev-board-esp-wroom-32) | ESP32-D0WD-V3 rev 3.1 / ESP-WROOM-32 | 4 MB / none | CP2102 | `esp32:esp32:esp32` | esp32 3.3.12 | 3.0.0-dev | Pending |

## Board details

### ELEGOO ESP-32 Dev Board (ESP-WROOM-32)

- **Chip:** ESP32-D0WD-V3, revision v3.1, dual core 240 MHz, 40 MHz crystal (read with esptool)
- **Flash:** 4 MB, 3.3 V. No PSRAM.
- **USB:** Silicon Labs CP2102. On Windows this needs the
  [CP210x VCP driver](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers),
  otherwise Device Manager shows "Code 28" and no COM port is created.
  `arduino-cli board list` reports the board as "Unknown", so pass `--fqbn esp32:esp32:esp32`
  (Arduino IDE: "ESP32 Dev Module").
- **PN5180 module:** not yet wired
- **Wiring:** default ESP32 pins from [Wiring](wiring.md) (to be confirmed when the rig is built)
- **Examples run:** none yet
- **Tags tested:** none yet
- **Notes:** board self-test without the PN5180 (2026-11-01) passed: flash, heap, timing, both
  cores, and pull-up/pull-down on GPIO 18, 19, 23, 16, 17. GPIO5 (default BUSY) reads high even
  with the internal pull-down, because the board has an external pull-up on this boot strapping pin.
  That's fine for BUSY, which the PN5180 drives actively.

## Adding a board

Add a row to the table and a section under "Board details" with:

- board name as sold, and the chip and revision as read by `esptool chip-id` (or the MCU for non-ESP32 boards)
- flash and PSRAM size, and the USB bridge
- FQBN, core version (`arduino-cli core list`) and library version or commit
- PN5180 module used, wiring (or "default pins"), and the power supply if it isn't plain USB
- which examples were run and which tags were read or written (ISO 15693 / ISO 14443 type)
- anything that failed or needed a workaround
