# Examples

All examples are in [`examples/`](../examples). In the Arduino IDE they are under **File → Examples → PN5180 ESP32**. Each sketch selects its pins with `#define`s at the top. Check them against your [wiring](wiring.md).

| Example | Protocol | Boards | What it shows |
|---------|----------|--------|---------------|
| [PN5180-ReadUID](../examples/PN5180-ReadUID) | 14443 + 15693 | all | Detect either card type on one reader and print the UID |
| [PN5180-ISO14443](../examples/PN5180-ISO14443) | 14443 | all | MIFARE / NTAG card detection and UID reading |
| [PN5180-Library](../examples/PN5180-Library) | 15693 | all | Inventory, system info, reading and writing blocks |
| [PN5180-MultipleBlock](../examples/PN5180-MultipleBlock) | 15693 | all | Read several blocks in one command |
| [PN5180-ReadMultipleUID](../examples/PN5180-ReadMultipleUID) | 15693 | all | Read the UIDs of several tags in the field at once (anti-collision) |
| [PN5180-LowPowerCardDetection](../examples/PN5180-LowPowerCardDetection) | 14443 + 15693 | all; deep sleep on ESP32 | Low Power Card Detection (LPCD) |
| [PN5180-LPCD-ESP32-deep-sleep](../examples/PN5180-LPCD-ESP32-deep-sleep) | none (base class) | ESP32 only | LPCD that wakes the ESP32 from deep sleep through the IRQ pin |

## Using both protocols on one reader

The PN5180 is configured for one protocol at a time. Create one object per protocol on the same pins and switch with `reset()` + `setupRF()`:

```cpp
PN5180ISO14443 nfc14443(PN5180_NSS, PN5180_BUSY, PN5180_RST);
PN5180ISO15693 nfc15693(PN5180_NSS, PN5180_BUSY, PN5180_RST);

void loop() {
  nfc14443.reset();
  nfc14443.setupRF();
  // ... look for ISO 14443 cards ...

  nfc15693.reset();
  nfc15693.setupRF();
  // ... look for ISO 15693 tags ...
}
```

`PN5180-ReadUID` is a complete version of this pattern.
