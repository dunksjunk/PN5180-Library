# PN5180 ESP32 documentation

Start with the [Quick start](../README.md#quick-start) in the main README, then use these pages for details.

## Getting started

- [Installation](installation.md): Arduino IDE, arduino-cli and PlatformIO
- [Wiring](wiring.md): pinout, 3.3 V logic levels, default pins used by the examples
- [Examples](examples.md): what each example sketch shows

## Reference

- [API reference](api/README.md)
  - [`PN5180`](api/PN5180.md): base class (SPI, registers, EEPROM, RF field, LPCD, MIFARE authentication)
  - [`PN5180ISO14443`](api/PN5180ISO14443.md): ISO 14443 Type A (MIFARE, NTAG)
  - [`PN5180ISO15693`](api/PN5180ISO15693.md): ISO 15693 (ICODE SLIX / SLIX2)

## Help

- [Debugging](debugging.md): turning on the library's trace output
- [Troubleshooting](troubleshooting.md): common problems and fixes

## Datasheets

The [`extras`](../extras) folder holds the vendor documents the library is built from:

- [PN5180 datasheet](../extras/PN5180A0XX_C3_C4.pdf)
- [PN5180 module schematic](../extras/PN5180%20R1.1-170710_SCH.PDF)
- [ICODE SLIX2 datasheet](../extras/SL2S2602.pdf)
- [MIFARE Ultralight datasheet](../extras/MF0ICU1.pdf)
