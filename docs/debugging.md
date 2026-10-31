# Debugging

The library can print a detailed, indented trace of every command and SPI frame to `Serial`. It is off by default and compiles to nothing, so it costs no flash or RAM when off.

## Turning on trace output

Tracing is switched on per source file by editing the library itself. Uncomment this line near the top of the file(s) you want to trace:

```cpp
//#define DEBUG 1
```

| File | Traces |
|------|--------|
| `src/PN5180.cpp` | SPI frames, register/EEPROM access, RF field, LPCD |
| `src/PN5180ISO14443.cpp` | ISO 14443 activation, UID reading, MIFARE commands |
| `src/PN5180ISO15693.cpp` | ISO 15693 commands and responses |

Notes:

- A `#define DEBUG` in your sketch does **not** work. The library's `.cpp` files are compiled separately and never see it.
- Call `Serial.begin()` in `setup()` before the first library call.
- Tracing slows every command down noticeably and produces a lot of output. Turn it on only for the file you are investigating.
- Remember to comment the line out again. Installing or updating the library overwrites your edit.

## Checking the module responds

Read the product version from the PN5180's EEPROM after `reset()`:

```cpp
uint8_t productVersion[2];
nfc.readEEprom(PRODUCT_VERSION, productVersion, sizeof(productVersion));
Serial.print(F("Product version: "));
Serial.print(productVersion[1]);
Serial.print('.');
Serial.println(productVersion[0]);
```

A value of `0xFF` (255) means the PN5180 is not answering on SPI. See [Troubleshooting](troubleshooting.md).
