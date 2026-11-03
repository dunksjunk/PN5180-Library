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
| [ELEGOO ESP-32 Dev Board (ESP-WROOM-32)](#elegoo-esp-32-dev-board-esp-wroom-32) | ESP32-D0WD-V3 rev 3.1 / ESP-WROOM-32 | 4 MB / none | CP2102 | `esp32:esp32:esp32` | esp32 3.3.12 | 3.0.0-dev (76e0f45) | Partial |

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
  different pins, and the examples' `WRITE_ENABLED` branches (these compile since d0cdb01, but haven't been run)
- **Tags tested:**
  - NTAG215 (ISO 14443A, 7-byte UID `04 BB 6A 04 CC 2A 81`, SAK 00, ATQA 00 44): activation, UID,
    `mifareBlockRead()` of pages 0–134 (200/200 reads OK), GET_VERSION, `mifareHalt()` + WUPA,
    `mifareBlockWrite16()` on page 100 (written and restored). Rerun 2026-11-02 with 4ef0528/4bc4c60:
    `mifareHalt()` returns `true`, the halted card ignores REQA and answers WUPA, a write to the halted
    card returns `0` (no answer), page 100 written (ACK `0x0A`) and restored, a write to page 200 (doesn't
    exist) is not reported as written, 200/200 page reads in 4.0 s
  - ISO 15693 tag, UID `E0 1D A1 E8 3F 9F 9E D9`, 32 blocks of 4 bytes: `getInventory()` (20/20, 33 ms each),
    `getInventoryMultiple()` (single tag), `getSystemInfo()`, `readSingleBlock()`, `readMultipleBlock()`
    including a non-zero start, `writeSingleBlock()` on block 20 (written and restored), wrong UID fails cleanly.
    Not tested: ICODE SLIX2 privacy/password commands
  - Two more ISO 15693 cards (2026-11-02), UIDs `E0 1D A1 2A 46 95 16 C7` and `E0 1D A1 15 04 7F 39 11`:
    `getInventory()` and `getInventoryMultiple()` on each card alone, 10/10
  - A MIFARE-type ISO 14443A card with a 4-byte UID (`56 57 2F 07`): UID read only
  - **ISO 15693 regression with the byte-wise send path** (2026-11-02, d25df7f, card `E0 1D A1 15 04 7F 39 11`,
    32 blocks of 4 bytes): inventory (20/20, 33 ms each), system info, single and multi-block reads,
    a read past the last block (`BLOCK_NOT_AVAILABLE`), `getInventoryMultiple()`, wrong UID. No writes
  - **EEPROM** (2026-11-02, d25df7f): a 2-byte `writeEEprom()` to `LPCD_FIELD_ON_TIME`/`LPCD_THRESHOLD`
    (0x36–0x37), read back, restored to the original `F0 03` and read back again; the caller's buffer
    was unchanged
  - **Argument checks** (2026-11-02, 3d4d772): 22 checks of the datasheet limits and `NULL` buffers
    (`readEEprom()` at address 254, negative and zero lengths, `writeEEprom()`, `sendData()` `validBits`,
    `switchToLPCD()` outside 1–2690 leaving `IRQ_ENABLE` untouched, `mifareAuthenticate()`,
    `sendCommand()` leaving its buffer unchanged), followed by a successful card read
  - **`reset()`** (2026-11-02): after the RST pulse the field is off and `IRQ_ENABLE` is back at its
    power-on value (`0x04`), so the chip really resets. It takes about 9 ms
  - **Several ISO 15693 tags at once** (2026-11-02, d0cdb01): `getInventoryMultiple()` found both tags of a
    pair 10/10, with no duplicates, for two different pairs. Both UIDs then answered addressed reads, and
    `maxTags` = 1 returned exactly one tag. The three cards answer in slots 9, 7 and 1, so no two of them
    collide: this covers the 16-slot round, **not** collision resolution. Test with an anti-collision fix
    that is not yet committed: the same pair 10/10; a masked inventory with masks of 0–8 UID nibbles
    answered in the expected slot, and non-matching masks got no answer (3 runs). That test found and
    fixed a missed answer in slot 0 after a long masked request. Not tested yet: two tags answering in
    the same slot, more than 2 tags at once
  - **Anti-collision with many tags** (2026-11-03, with the 2026-11-02 fix): about 20 NXP ICODE label tags
    (`E0 04 01 08 66 …`, cut from a roll) in a loose pile, plus older cards. `getInventoryMultiple()`
    returned up to 8 tags in one call, 21 distinct UIDs over 70 calls, no duplicates and no invalid UIDs.
    Tags that answer in the same slot were returned together in one call many times (slot 12: `..33BC`,
    `..213C`, `..0BDC`, `..47CC`; slot 3: `..D493` + `..E803`; slot 5: `..6A65` + `..E805`; slot 9:
    `..C0E9` + `..57A9`), so collisions are resolved through masked rounds. A call took 340 ms with no collisions and up to about 1.4 s with
    them (`maxTags` = 32). Not tested yet: tags whose UIDs share the last two nibbles, which needs a
    second mask level. Tags on a roll, or stacked tightly, don't answer at all. Laid side by side, only
    the tag over the antenna centre answers. In this pile the set of answering tags changed from second to
    second, so addressed reads right after the inventory reached only 4 of 17 UIDs. Two tags
    (`..D495`, `..C0EA`) answered `readSingleBlock(0)` with tag error `0x0F`; not investigated
  - **Antenna limits with full-size cards:** two ISO 15693 cards stacked flat on this module's antenna
    blocked each other completely (no answers at all). With card stock between them the top card was out
    of range; with one sheet of printer paper both read reliably. With three cards, or with one particular
    pair, at least one card stopped answering whatever the spacing or fanning. The PN5180 `AGC_VALUE`
    register stayed at 496 throughout, so it doesn't show this loading in reader mode
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

  Found by code review and verified on this rig (2026-11-02): `sendData()`/`writeEEprom()` stack copies
  (d25df7f), `mifareBlockWrite16()` reporting a stale byte when the card stayed silent (4ef0528),
  `getSystemInfo()` length check (d91588f), `sendCommand()` overwriting its buffer (4bc4c60), argument and
  `NULL` checks (3d4d772)

  A `readCardSerial()` call with the card present went from about 630 ms to about 160 ms
- **Open observation:** before the `readCardSerial()` fixes, about 1 in 10 `mifareBlockRead()` calls failed
  with an RX data-integrity error (`RX_STATUS` 0x003D0000). Afterwards, 200 out of 200 reads succeeded
- **Notes:** board self-test without the PN5180 (2026-11-01) passed: flash, heap, timing, both
  cores, and pull-up/pull-down on GPIO 18, 19, 23, 16, 17. GPIO5 (default BUSY) reads high even
  with the internal pull-down, because the board has an external pull-up on this boot strapping pin.
  That's fine for BUSY, which the PN5180 drives actively. With the PN5180 attached, the same self-test
  reports GPIO5 as failing: the idle PN5180 drives BUSY low, which is expected.
- **Wiring fault (2026-11-02):** a MOSI jumper that was working loose made a tag answer one or two slots
  early and made `getInventoryMultiple()` miss it in 3 of 10 runs. The wire then came off completely;
  every call then failed after its timeout without hanging. After reseating it, the same card answered in
  its correct slot every time. Check the jumpers before suspecting the library when ISO 15693 results
  are erratic.
- **Open observation (2026-11-02):** for about an hour the NTAG215 stopped answering REQA (the trace shows
  the correct `09 07 26` frame going out, and no ATQA), with both the committed library and the changes
  under test, while ISO 15693 cards kept reading. `reset()` didn't help, although it does reset the chip
  (see above). Unplugging the ESP32's USB for 10 s fixed it; no wires were touched. Cause unknown. If
  ISO 14443 suddenly stops answering, power-cycle the board before debugging code

## Adding a board

Add a row to the table and a section under "Board details" with:

- board name as sold, and the chip and revision as read by `esptool chip-id` (or the MCU for non-ESP32 boards)
- flash and PSRAM size, and the USB bridge
- FQBN, core version (`arduino-cli core list`) and library version or commit
- PN5180 module used, wiring (or "default pins"), and the power supply if it isn't plain USB
- which examples were run and which tags were read or written (ISO 15693 / ISO 14443 type)
- anything that failed or needed a workaround
