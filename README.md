# PN5180 ESP32

Arduino library for the **NXP PN5180** NFC frontend module, optimized for ESP32. It also builds on Arduino AVR boards (Uno, Nano, Mega); other microcontrollers may be optimized later.

This is a fork of [tueddy/PN5180-Library](https://github.com/tueddy/PN5180-Library), the very useful PN5180 library by Andreas Trappmann, Dirk Carstensen and contributors. Version 3.0.0 is a major rework and may change behavior or break existing code; see [CHANGELOG.md](CHANGELOG.md).

## Features

- **ISO 15693** (NFC-V, e.g. NXP ICODE SLIX / SLIX2): inventory of one or many tags, block read/write, system info, SLIX2 privacy mode
- **ISO 14443 Type A** (e.g. MIFARE Classic, MIFARE Ultralight, NTAG): card detection, 4- and 7-byte UIDs, MIFARE Classic authentication, block read/write
- **Low Power Card Detection (LPCD)**, including waking an ESP32 from deep sleep
- Low-level access to PN5180 registers, EEPROM and the RF field
- Any `SPIClass`, custom SPI pins on ESP32, adjustable SPI clock
- Every hardware wait is time-limited and yields (`delay(1)`), so other FreeRTOS tasks keep running

## Quick start

1. **Install** the library: in PlatformIO, add `https://github.com/dunksjunk/PN5180-Library.git` to `lib_deps`; for the Arduino IDE, see [Installation](docs/installation.md).
2. **Wire** the module. On ESP32: SCK 18, MISO 19, MOSI 23, NSS 16, BUSY 5, RST 17, plus 3.3 V, 5 V and GND. On 5 V boards you need level shifters; see [Wiring](docs/wiring.md).
3. **Run** one of the sketches below, or the [examples](docs/examples.md).

Read the UID of an ISO 14443 card (MIFARE, NTAG, …):

```cpp
#include <PN5180.h>
#include <PN5180ISO14443.h>

#define PN5180_NSS  16
#define PN5180_BUSY 5
#define PN5180_RST  17

PN5180ISO14443 nfc(PN5180_NSS, PN5180_BUSY, PN5180_RST);

void setup() {
  Serial.begin(115200);
  nfc.begin();
  nfc.reset();
  nfc.setupRF();
}

void loop() {
  uint8_t uid[10];
  int8_t uidLength = nfc.readCardSerial(uid);
  if (uidLength > 0) {
    Serial.print("UID:");
    for (int i = 0; i < uidLength; i++) {
      Serial.print(uid[i] < 0x10 ? " 0" : " ");
      Serial.print(uid[i], HEX);
    }
    Serial.println();
  }
  delay(500);
}
```

Read the UID of an ISO 15693 tag (ICODE):

```cpp
#include <PN5180.h>
#include <PN5180ISO15693.h>

#define PN5180_NSS  16
#define PN5180_BUSY 5
#define PN5180_RST  17

PN5180ISO15693 nfc(PN5180_NSS, PN5180_BUSY, PN5180_RST);

void setup() {
  Serial.begin(115200);
  nfc.begin();
  nfc.reset();
  nfc.setupRF();
}

void loop() {
  uint8_t uid[8];
  ISO15693ErrorCode rc = nfc.getInventory(uid);
  if (rc == ISO15693_EC_OK) {
    Serial.print("UID:");
    for (int i = 7; i >= 0; i--) {   // UID is stored LSB first
      Serial.print(uid[i] < 0x10 ? " 0" : " ");
      Serial.print(uid[i], HEX);
    }
    Serial.println();
  } else {
    Serial.println(nfc.strerror(rc));
  }
  delay(500);
}
```

Nothing happens? Check that the module answers by reading its product version. See [Debugging](docs/debugging.md#checking-the-module-responds) and [Troubleshooting](docs/troubleshooting.md).

## Documentation

Full documentation is in [`docs/`](docs/README.md):

- [Installation](docs/installation.md) · [Wiring](docs/wiring.md) · [Examples](docs/examples.md)
- [API reference](docs/api/README.md): [`PN5180`](docs/api/PN5180.md), [`PN5180ISO14443`](docs/api/PN5180ISO14443.md), [`PN5180ISO15693`](docs/api/PN5180ISO15693.md)
- [Debugging](docs/debugging.md) · [Troubleshooting](docs/troubleshooting.md)
- Datasheets in [`extras/`](extras)

## License

GNU Lesser General Public License v2.1 – see [LICENSE](LICENSE).

Copyright (c) 2018 Andreas Trappmann, (c) 2019 Dirk Carstensen, and contributors.

## Changelog

See [CHANGELOG.md](CHANGELOG.md).
