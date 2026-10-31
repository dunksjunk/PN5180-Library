# API reference

| Class | Header | Use it for |
|-------|--------|------------|
| [`PN5180`](PN5180.md) | `PN5180.h` | Base class: SPI setup, reset, registers, EEPROM, raw RF data, RF field, LPCD, MIFARE authentication |
| [`PN5180ISO14443`](PN5180ISO14443.md) | `PN5180ISO14443.h` | ISO 14443 Type A cards: MIFARE Classic / Ultralight, NTAG |
| [`PN5180ISO15693`](PN5180ISO15693.md) | `PN5180ISO15693.h` | ISO 15693 tags: ICODE SLIX / SLIX2 |

`PN5180ISO14443` and `PN5180ISO15693` inherit from `PN5180`, so every base-class method is also available on them.

## Typical call sequence

```cpp
nfc.begin();      // set up pins and start SPI
nfc.reset();      // hardware reset of the PN5180
nfc.setupRF();    // load the protocol's RF configuration and switch the field on
// ... protocol commands ...
```

## Conventions

- Most low-level methods return `bool`: `true` means the SPI transaction finished in time. It does **not** mean a card answered.
- `PN5180ISO15693` methods return an [`ISO15693ErrorCode`](PN5180ISO15693.md#error-codes).
- UIDs are byte arrays. ISO 14443 UIDs are 4 or 7 bytes, most significant byte first. ISO 15693 UIDs are 8 bytes, stored **least** significant byte first.
- Buffers are always supplied by the caller, except the pointer returned by [`readData(len)`](PN5180.md#raw-rf-data).
