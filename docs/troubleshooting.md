# Troubleshooting

## Product version reads 255 (0xFF) / nothing is detected

The PN5180 is not answering on SPI.

- On 5 V boards, check the level shifters on SCK, MOSI, NSS and RST (see [Wiring](wiring.md)).
- Most modules need **both** 5 V and 3.3 V connected.
- Check that BUSY and RST are connected to the pins passed to the constructor.
- Check that `begin()` is called before any other library call.

## Commands time out / the sketch hangs for half a second

Each command waits for the BUSY line, for at most `commandTimeout` milliseconds (500 by default). Repeated timeouts usually mean BUSY is not connected or is stuck high. A `reset()` usually recovers the chip.

## Unreliable reads with long wires

Lower the SPI clock (default 7 MHz) after `begin()`:

```cpp
nfc.setSPISettingsFrecuency(2000000);   // 2 MHz
```

## Switching between ISO 14443 and ISO 15693

The chip is configured for one protocol at a time. Call `reset()` and the matching `setupRF()` before using the other protocol (see [Examples](examples.md#using-both-protocols-on-one-reader)).

## Data read with `readData()` changes unexpectedly

`uint8_t* readData(int len)` returns a pointer into a buffer owned by the library. The next `readData()` call overwrites it. Copy the bytes you need first, or use the `readData(len, buffer)` overload with your own buffer.

## Still stuck?

Turn on [trace output](debugging.md) for the relevant file and open an issue with the log at <https://github.com/dunksjunk/PN5180-Library/issues>.
