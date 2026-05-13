# lora-ew-system

Anti-jamming communication system built on ESP32 using LoRa SX1276. The system uses Frequency Hopping Spread Spectrum (FHSS) to maintain a reliable link under active RF jamming, with AES-128 encryption on all transmitted payloads.

Built as a senior design project. The goal was to implement EW-resilient communications on real embedded hardware, not just simulate it.

## Hardware

- 2x ESP32 dev boards
- 2x LoRa SX1276 modules (915 MHz)
- Flipper Zero for jamming simulation

## How it works

Both nodes share a synchronized pseudo-random hop sequence. The transmitter sends on the current channel and waits for an ACK. If no ACK comes back within the timeout window, the system treats it as a jamming event and both nodes jump to the next frequency in the sequence. Target switching time is under 200ms.

RSSI is monitored continuously. If signal strength drops below a set threshold and packet loss climbs past the tolerance window, the system triggers an early hop instead of waiting for the scheduled one.

Every payload is encrypted with AES-128 before transmission. Both nodes use a pre-shared symmetric key. This means even if someone captures a packet on the current frequency, they cannot read it.

## Project structure

```
lora-ew-system/
├── src/
│   ├── main.cpp        # system init and main loop
│   ├── fhss.cpp        # hopping logic and jamming detection
│   ├── fhss.h
│   ├── crypto.cpp      # AES-128 encrypt/decrypt
│   ├── crypto.h
│   ├── radio.cpp       # SX1276 driver interface
│   └── radio.h
├── test/
│   └── test_fhss.cpp   # hop sequence unit tests
├── platformio.ini
└── README.md
```

## Build

Uses PlatformIO.

```bash
git clone https://github.com/adam9798/lora-ew-system
cd lora-ew-system
pio run
pio run --target upload
```

## Testing

Jamming was simulated using a Flipper Zero transmitting interference on the target frequency band. Tested three scenarios: spot jamming on a single channel, sweep jamming across the full hopping band, and high-power noise injection. The system detected signal degradation and completed frequency hops within the 200ms target in all cases.

## Current limitations

- Hop sequence is pre-shared. No dynamic key exchange yet.
- AES key is hardcoded for development purposes.
- All testing was done indoors. Not yet validated in field conditions.
