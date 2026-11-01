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
| [ELEGOO ESP-32 Dev Board (ESP-WROOM-32)](#elegoo-esp-32-dev-board-esp-wroom-32) | ESP32-D0WD-V3 rev 3.1 / ESP-WROOM-32 | 4 MB / none | CP2102 | `esp32:esp32:esp32` | esp32 3.3.12 | 3.0.0-dev (9c04e37) | Partial |

## Board details

### ELEGOO ESP-32 Dev Board (ESP-WROOM-32)

- **Chip:** ESP32-D0WD-V3, revision v3.1, dual core 240 MHz, 40 MHz crystal (read with esptool)
- **Flash:** 4 MB, 3.3 V. No PSRAM.
- **USB:** Silicon Labs CP2102. On Windows this needs the
  [CP210x VCP driver](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers),
  otherwise Device Manager shows "Code 28" and no COM port is created.
  `arduino-cli board list` reports the board as "Unknown", so pass `--fqbn esp32:esp32:esp32`
  (Arduino IDE: "ESP32 Dev Module").
- **PN5180 module:** product version 4.0, firmware 4.0, EEPROM version 153.0
- **Wiring:** default ESP32 pins from [Wiring](wiring.md) (SCK 18, MISO 19, MOSI 23, NSS 16, BUSY 5,
  RST 17, IRQ 15), powered from USB. The PN5180's IRQ pin is configured active high (`IRQ_PIN_CONFIG` = 0x01).
  The tag sits on a plastic spacer on the antenna.
- **Examples run** (2026-11-01, library at 9c04e37), all reading the tag on every loop:
  `PN5180-ISO14443`, `PN5180-ReadUID` (NTAG215, free heap steady at 331 KB), `PN5180-Library`,
  `PN5180-MultipleBlock`, `PN5180-ReadMultipleUID` (ISO 15693 tag), and `PN5180-LowPowerCardDetection`
  (arms and stays quiet with nothing changing). Not run: `PN5180-LPCD-ESP32-deep-sleep`, which uses
  different pins, and the examples' `WRITE_ENABLED` branches
- **Tags tested:**
  - NTAG215 (ISO 14443A, 7-byte UID `04 BB 6A 04 CC 2A 81`, SAK 00, ATQA 00 44): activation, UID,
    `mifareBlockRead()` of pages 0–134 (200/200 reads OK), GET_VERSION, `mifareHalt()` + WUPA,
    `mifareBlockWrite16()` on page 100 (written and restored)
  - ISO 15693 tag, UID `E0 1D A1 E8 3F 9F 9E D9`, 32 blocks of 4 bytes: `getInventory()` (20/20, 33 ms each),
    `getInventoryMultiple()` (single tag), `getSystemInfo()`, `readSingleBlock()`, `readMultipleBlock()`
    including a non-zero start, `writeSingleBlock()` on block 20 (written and restored), wrong UID fails cleanly.
    Not tested: multiple tags at once (anti-collision), ICODE SLIX2 privacy/password commands
  - LPCD (`prepareLPCD()` + `switchToLPCD(300)`, threshold 0x03): fired when a hand reached for the card
    and when the ISO 15693 tag was placed on the empty reader. 0 false triggers over about 2 minutes at
    thresholds 0x03–0x20 with nothing changing. In one early run it fired every 0.3–3 s with no visible
    cause; that didn't reproduce
- **Bugs found and fixed on this rig:** see the 3.0.0 entries in [CHANGELOG](../CHANGELOG.md):
  - `setRF_on()`/`setRF_off()` 500 ms timeout when the field was already in that state (be2976d)
  - ATQA overwritten in `activateTypeA()` (50fd439)
  - `readCardSerial()` alternating success/`-1` and never finding a halted card (08e3f41)
  - the examples' handling of the `readCardSerial()` result (5d45b0e)
  - `sendCommand()` declared but never implemented (5748c04)
  - per-file `DEBUG` tracing failing to link (7eeb26a)
  - `getInventoryMultiple()` failing whenever a tag answered, because of an over-read that left the PN5180 BUSY (c96777d)
  - ISO 15693 reads not checking the response length (0465be3)
  - `switchToLPCD()` ignoring its setup steps, and the LPCD example rewriting EEPROM on every boot (9c04e37)

  A `readCardSerial()` call with the card present went from about 630 ms to about 160 ms
- **Open observation:** before the `readCardSerial()` fixes, about 1 in 10 `mifareBlockRead()` calls failed
  with an RX data-integrity error (`RX_STATUS` 0x003D0000). Afterwards, 200 out of 200 reads succeeded
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
