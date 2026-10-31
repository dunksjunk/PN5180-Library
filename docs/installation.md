# Installation

PN5180 ESP32 is not in the Arduino Library Manager or the PlatformIO registry yet. Install it from GitHub using one of the methods below.

Headers to include in your sketch:

| Header | Use it for |
|--------|------------|
| `PN5180.h` | Always; the base class and register/EEPROM constants |
| `PN5180ISO14443.h` | ISO 14443 Type A cards (MIFARE, NTAG) |
| `PN5180ISO15693.h` | ISO 15693 tags (ICODE SLIX / SLIX2) |

The library uses the Arduino `SPI` library, which comes with every board core. It has no other dependencies.

## Arduino IDE

1. On GitHub, choose **Code → Download ZIP**.
2. In the Arduino IDE, choose **Sketch → Include Library → Add .ZIP Library…** and select the downloaded file.
3. The examples appear under **File → Examples → PN5180 ESP32**.

Or clone the repository straight into your sketchbook's `libraries` folder:

```sh
cd ~/Documents/Arduino/libraries     # Windows: %USERPROFILE%\Documents\Arduino\libraries
git clone https://github.com/dunksjunk/PN5180-Library.git
```

## arduino-cli

Installing from a Git URL has to be enabled once:

```sh
arduino-cli config set library.enable_unsafe_install true
arduino-cli lib install --git-url https://github.com/dunksjunk/PN5180-Library.git
```

## PlatformIO

Add the repository to `lib_deps` in `platformio.ini`:

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps =
    https://github.com/dunksjunk/PN5180-Library.git
```

To pin a specific version, add `#<tag>` or `#<commit>` to the URL.

## Supported boards

| Platform | Status |
|----------|--------|
| ESP32 (Arduino core 3.x) | Primary target; optimized and tested first |
| Arduino AVR (Uno, Nano, Mega) | Builds; needs level shifters (see [Wiring](wiring.md)) |
| Other Arduino-compatible boards | Should work if the core provides `SPI`; not tested |
