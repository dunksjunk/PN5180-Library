# Changelog

All notable changes to **PN5180 ESP32** are recorded here, newest first.
Versions follow [Semantic Versioning](https://semver.org/): a major bump means
breaking changes, a minor bump adds features, a patch bump fixes bugs.

PN5180 ESP32 is a fork of [tueddy/PN5180-Library](https://github.com/tueddy/PN5180-Library),
the very useful PN5180 library by Andreas Trappmann and Dirk Carstensen (tueddy) with
contributions from many others. Versions 1.0 to 2.3.7 below are the upstream history,
kept for reference. From 3.0.0 on, changes are made in this fork.

**Version 3.0.0 – unreleased**

Major rework of the fork. Behavior may change and there may be breaking changes;
they are listed here as they land.

- Forked from tueddy/PN5180-Library 2.3.7 and renamed to **PN5180 ESP32** (`PN5180-ESP32` on PlatformIO)
- Optimizing for the general ESP32 platform first; other microcontrollers may be optimized later
- Arduino 1.5 library layout: sources moved to `src/`, reference documents moved from `doc/` to `extras/`
- Added a PlatformIO manifest (`library.json`)
- Renamed the internal `Debug.h`/`Debug.cpp` to `PN5180Debug.h`/`PN5180Debug.cpp` so they can't clash with other libraries
- `keywords.txt`: added the `PN5180ISO14443` class and the missing public methods, removed stale entries
- Added GitHub Actions CI: arduino-lint plus compiling every example for Arduino Uno and ESP32
- Moved release notes from `README.md` into this file
- Added [`docs/tested-hardware.md`](docs/tested-hardware.md), a list of boards tested on real hardware
- **Behavior change:** register, EEPROM, RF on/off and `loadRFConfig()` calls now return `false` when the SPI transaction times out (previously they always returned `true`). `readData(len)` returns `NULL` in that case, and `getIRQStatus()` returns `0` instead of an uninitialized value
- **Behavior change:** `prepareLPCD()` only writes EEPROM bytes that differ from the wanted value and returns `false` on any EEPROM failure. Previously it rewrote all five bytes on every call (wearing the EEPROM when called on each deep-sleep wake-up) and always returned `true`
- Fix: `readMultipleBlock()` rejected every start block except 0
- Fix: `getInventoryMultiple()` could write past the caller's `uid` buffer when more than `maxTags` tags answered, could record a collided response as a UID, and used a variable-length array. It now stops at `maxTags`, bounds the number of anti-collision rounds, and returns the error if a poll fails
- Fix: when an inventory poll failed partway (SPI timeout), `getInventoryMultiple()` returned without switching the RF field off and reloading the ISO 15693 config, leaving `TX_CONFIG` set to send EOF only, so the next ISO 15693 command sent no data. Every exit now goes through that reset, and a failed `sendData`, `RX_STATUS` read or reset is reported as `ISO15693_EC_UNKNOWN_ERROR`
- Fix (AVR): the IRQ bit masks in `PN5180.h` were plain `int` shifts, so on 16-bit AVR `GENERAL_ERROR_IRQ_STAT` (`1<<17`) and `LPCD_IRQ_STAT` (`1<<19`) overflowed. As a result `prepareLPCD()` never enabled the LPCD interrupt and `PN5180ISO14443` never enabled the general-error interrupt. All masks are now `1UL<<n`. The examples' `showIRQStatus()` had the same bug, which printed wrong flags for bits 15–19
- **Behavior change / Fix:** `setRF_on()` waited for the TX_RFON IRQ even when the field was already on. That IRQ only fires on a switch-on, so the call always timed out after 500 ms and returned `false`. Every `readCardSerial()`/`isCardPresent()` hit this, because `activateTypeA()` calls `setRF_on()` each time, so each one took about 630 ms instead of about 130 ms (measured on ESP32). `setRF_on()`/`setRF_off()` now check `RF_STATUS` and return `true` right away if the field is already in the requested state, and clear a stale IRQ flag before switching. `activateTypeA()` now returns `-1` if `setRF_on()` fails (it used to ignore the result)
- Fix: for 7-byte UIDs, `activateTypeA()` overwrote the ATQA in `buffer[0..1]` with UID bytes. The ATQA is now kept there, as documented. If sending SELECT (cascade level 1) fails, it now returns `-2` instead of reporting a 4-byte UID that was never selected, and it no longer prints "Read 5 bytes failed!" to `Serial` outside debug mode
- **Behavior change / Fix:** `readCardSerial()` (and so `isCardPresent()`) sent REQA with the field still on. A card left ACTIVE by the previous call ignores REQA, so back-to-back calls alternated between success and `-1` (e.g. `isCardPresent()` followed by `readCardSerial()`), and a card halted with `mifareHalt()` was never found again. It now switches the field off for 10 ms first, which resets every card to IDLE. This drops any MIFARE Classic authentication made before the call. Docs now state that "no card" returns `-1`, not `0`
- Examples: `PN5180-ISO14443` treated `readCardSerial() == -1` as success and printed an uninitialized 8-byte UID. `PN5180-LowPowerCardDetection` stored the result in a `uint8_t`, so `-1` became 255 and it printed 255 bytes from a 10-byte buffer. `PN5180-ISO14443`, `PN5180-ReadUID` and `PN5180-LowPowerCardDetection` now call `readCardSerial()` once (no separate `isCardPresent()`) and only use the UID when the result is `> 0`
- Fix: `sendCommand()` was declared, documented and listed in `keywords.txt`, but never implemented, so any sketch calling it failed to link (`undefined reference to PN5180::sendCommand`). It now passes the command to the library's internal SPI command path
- Fix: `getInventoryMultiple()` always failed with `ISO15693_EC_UNKNOWN_ERROR` as soon as a tag answered. `inventoryPoll()` read one byte more than the PN5180 had received (`readData(len+1)`), which left the PN5180 BUSY, so the next slot's command timed out. Before 3.0 that failure was silently ignored. It now reads exactly the received length (verified on ESP32 with a single ISO 15693 tag)
- Debug tracing: turning on `#define DEBUG 1` in one `.cpp` file (as `docs/debugging.md` describes) failed to link with `undefined reference to formatHex`, because the helper was only compiled when `PN5180Debug.cpp` itself had `DEBUG` defined. It is now always compiled; release builds are unchanged (the linker drops it)
- Fix: ISO 15693 reads didn't check how many bytes the tag actually returned. A tag that answers a `readMultipleBlock()` running past its last block with only the blocks it has (seen on hardware) made the call return OK, with the missing blocks filled from stale read-buffer contents. It now returns `ISO15693_EC_BLOCK_NOT_AVAILABLE`. `readSingleBlock()`, `getInventory()`, `getSystemInfo()` and `getRandomNumber()` now return `ISO15693_EC_UNKNOWN_ERROR` on a short answer, and every ISO 15693 command now reports a failed `sendData()`/`RX_STATUS` read instead of ignoring it
- `switchToLPCD()` now returns `false` if clearing the IRQ flags or enabling the LPCD IRQ fails (it used to ignore both and report only the mode switch)
- Examples: `PN5180-LowPowerCardDetection` wrote the five LPCD EEPROM bytes on every boot (EEPROM wear) and used 511 bytes of stack for it, which is a lot on an Uno. It now calls `prepareLPCD()`. **Behavior change:** the example's LPCD threshold is now the library default `0x03` instead of `0x04`
- Fix (examples): the optional `WRITE_ENABLED` branch in `PN5180-Library` and `PN5180-MultipleBlock` did not compile (`malloc` without a cast) and leaked its buffer on every loop. It now uses a stack buffer.
- Fix: `mifareBlockWrite16()` returned whatever byte was in the receive buffer (typically `0xFF`) when the card didn't answer. It now waits for the answer and returns `0` if none came. It no longer sends the 16 data bytes after the card NAKs the block number. **Behavior change:** the ACK/NAK value is the 4-bit frame only (`0x0A` = ACK)
- Fix: `mifareHalt()` always returned `true`; it now returns `false` if the HLTA command could not be sent
- Fix: `sendData()` ignored failures while switching the PN5180 to Transceive, and `sendData()`/`writeEEprom()` copied the data into a variable-length array on the stack. The data now goes to the SPI bus directly. `sendData()` rejects a negative `len`
- Fix: debug tracing of reads longer than 255 bytes looped forever (8-bit loop counter)
- Fix: `mifareBlockWrite16()` ignored SPI failures. It now stops at the first failure, always re-enables RX CRC, and returns `0` on failure
- Fix: the library did not compile on AVR (and other non-ESP32 boards) since 2.3.2, because `begin()` called the ESP32-only `SPI.begin(sck, miso, mosi, ss)`. Custom SPI pins are now ESP32-only and ignored elsewhere
- Fix: the `PN5180-LPCD-ESP32-deep-sleep` example used an undefined `WAKEUP_PIN_MASK` and did not compile
- Documentation moved into `docs/`: installation, wiring, examples, a per-class API reference, debugging and troubleshooting. The README keeps the overview and quick start

## Upstream history (tueddy/PN5180-Library)

**Version 2.3.7 – 01.09.2025**
- ISO14443: Explicitly allow unknown manufacturer ID 0xFF, thanks to tom!

**Version 2.3.6 – 03.06.2025**
- Allow changing the SPI frequency #16, thanks to @saidmoya12!

**Version 2.3.5 – 15.05.2025**
- Less blocking delays when using other tasks #15, thanks to @joe91!

**Version 2.3.4 – 31.10.2024**
- Better debug #13, thanks to @mjmeans!
- Fix warning "will be initialized after [-Wreorder]"
- Suppress [-Wunused-variable]

**Version 2.3.3 – 27.10.2024**
- Bugfix: start with default SPI

**Version 2.3.2 – 27.10.2024**
- Allow custom SPI pins #12, thanks to @mjmeans!
- Create .gitattributes #10, thanks to @mjmeans
- Replace errno, which is often a macro, thanks to @egnor

**Version 2.3.1 – 01.10.2024**
- Create a release with new version numbering

**Version 2.3 – 29.05.2024**
- cppcheck: make some params const
- transceiveCommand: restore state of SS in case of an error

**Version 2.2 – 13.01.2024**
- Add MIFARE Classic authentication, thanks to @golyalpha!
- Unify and simplify command creation
- Smaller memory footprint when reading UID only (saves ~500 bytes)
- Refactor SPI transaction into transceiveCommand()

**Version 2.1 – 01.04.2023**
- readMultipleBlock(), getInventoryMultiple() and getInventoryPoll() implemented, thanks to @laplacier!
- Multiple ISO-15693 tags can be read at once, thanks to @laplacier!
- Fixed some bugs and warnings with C++17

**Version 2.0 – 17.10.2022**
- Allow instantiation with a custom SPI class, thanks to @mwick83!

**Version 1.9 – 05.10.2021**
- Avoid endless loop in reset()

**Version 1.8 – 05.04.2021**
- Revert previous changes; SPI class was copied and caused problems
- Speed up reading (shorter delays) in reset/transceive commands
- Better initialization for ISO-14443 cards, see [AN12650](https://www.nxp.com.cn/docs/en/application-note/AN12650.pdf)

**Version 1.7 – 27.03.2021**
- Allow setup with another SPIClass than the default SPI, thanks to @tyllmoritz!

**Version 1.6 – 31.01.2021**
- Fix compiler warnings for PlatformIO
- Add LPCD (low power card detection) example for ESP32 (with deep sleep to save battery power)

**Version 1.5 – 07.12.2020**
- ISO-14443 protocol, basic support for MIFARE cards
- Low power card detection
- Handle transceiveCommand timeout

**Version 1.4 – 13.11.2019**
- ICODE SLIX2 specific commands, see the [SL2S2602 datasheet](https://www.nxp.com/docs/en/data-sheet/SL2S2602.pdf)
- Example usage, currently commented out

**Version 1.3 – 21.05.2019**
- Initialize reset pin with HIGH
- Make readBuffer static
- Typo fixes
- Data type corrections for length parameters

**Version 1.2 – 28.01.2019**
- Clear option bit in PN5180ISO15693::readSingleBlock and ::writeSingleBlock

**Version 1.1 – 26.10.2018**
- Cleanup, bug fixing, refactoring
- Automatic Arduino vs. ESP32 platform detection via compiler switches
- Merged open pull requests
- Documentation work

**Version 1.0.x – 21.09.2018**
- Initial versions
