# Capsule Corp AI Journal

A standalone, fully offline AI audio journal and voice companion device for kids — built around a Raspberry Pi 5, housed in a custom 3D-printed enclosure themed after the Capsule Corporation from Dragon Ball.

> **Fan project disclaimer:** This is a personal, non-commercial fan build. It is not affiliated with, endorsed by, or connected to Toei Animation, Shueisha, Funimation, or any Dragon Ball IP rights holders.

---

## What It Does

The device listens to spoken thoughts, reflects them back gently, and helps consolidate ideas — designed specifically for a child who is a strong verbal processor. It runs entirely offline with no subscription, no network connection, and no data leaving the device.

- Press the button → speak your thoughts → the device listens, thinks, and responds
- Five animated face states on the LCD give clear, friendly feedback at every stage
- Warm, conversational AI responses powered by a local LLM (no cloud)
- Battery-powered and portable

---

## Hardware

| Component | Part | Source |
|-----------|------|--------|
| Compute | Raspberry Pi 5 8GB + Official Active Cooler | PiShop |
| UPS / Battery | Geekworm X1201 HAT + 2× 18650 cells (Samsung 35E or Panasonic NCR18650B, ≤65.3mm) | Geekworm / Illumn |
| Display | Waveshare 1.83" IPS LCD, 240×280, NV3030B driver, GH1.25 8-pin SPI | Waveshare |
| Microphone | ReSpeaker 2-Mics Pi HAT V2.0 (TLV320AIC3104, Pi 5 compatible) | Seeed Studio |
| Amplifier | Adafruit MAX98357A I2S Mono Amp breakout | Adafruit |
| Speaker | Adafruit Stereo Enclosed Speaker Set 3W 4Ω (one speaker used) | Adafruit |
| Button | Adafruit Rugged Metal Pushbutton with White LED Ring, 16mm | Adafruit |
| Pi Breakout PCB | Custom EasyEDA design, Rev 2.0 (fabricated) | EasyEDA / JLCPCB |
| ReSpeaker Breakout PCB | Custom EasyEDA design, Rev 1.0 (fabricated) | EasyEDA / JLCPCB |

### GPIO Pin Map

| GPIO | Function | Notes |
|------|----------|-------|
| GPIO 2 | I2C SDA | ReSpeaker codec |
| GPIO 3 | I2C SCL | ReSpeaker codec |
| GPIO 8 | LCD SPI CS | |
| GPIO 10 | LCD SPI MOSI | Shared with ReSpeaker APA102 — APA102 not initialized |
| GPIO 11 | LCD SPI SCLK | Shared with ReSpeaker APA102 — APA102 not initialized |
| GPIO 18 | I2S BCLK | Shared between ReSpeaker and MAX98357A |
| GPIO 19 | I2S LRCLK | Shared between ReSpeaker and MAX98357A |
| GPIO 20 | I2S PCM_DIN | Mic data in (ReSpeaker) |
| GPIO 21 | I2S DIN | Speaker data out (MAX98357A) |
| GPIO 22 | Button NO | Short press = journal, long press ≥3s = shutdown |
| GPIO 24 | MAX98357A SD_MODE | |
| GPIO 25 | LCD DC | |
| GPIO 26 | LCD Backlight | Remapped from GPIO 18 |
| GPIO 27 | LCD RST | Remapped from GPIO 17 |
| Button LED+ | 3.3V direct | Built-in resistor — always-on white, no GPIO needed |
| Button LED− | GND | |

---

## Software Stack

| Role | Software |
|------|----------|
| OS | Raspberry Pi OS Lite 64-bit |
| LLM inference | llama.cpp + Llama 3.2 3B Instruct Q4_K_M GGUF (bartowski / Hugging Face) |
| Speech-to-text | whisper.cpp + Whisper small model |
| Wake word | openWakeWord |
| Text-to-speech | Piper TTS (aarch64) — en_US-lessac-medium or en_US-ryan-medium |
| Display driver | Waveshare NV3030B Python driver |
| Audio driver | HinTak seeed-voicecard fork (Pi 5 compatible) |
| Python libs | SpeechRecognition, pyaudio, sounddevice, numpy, Pillow |

### Pipeline States

The device moves through five states, each with a distinct LCD face animation and color:

| State | Color | Face | Description |
|-------|-------|------|-------------|
| Idle | Blue | Slow blink, breathing ring | Waiting for wake word |
| Listening | Green | Calm/pensive, breathing ring | Recording speech |
| Thinking | Purple | Upward gaze, orbiting particles | LLM inference |
| Speaking | Amber | Squint eyes, talking mouth, energy rays | TTS playback |
| Happy | Yellow | Spinning star eyes, sparkles | Positive interaction complete |
| Shutdown | Red | — | Shutdown in progress (LCD only) |

The button LED is always-on white — all state indication is handled by the LCD face.

---

## Enclosure

- **Theme:** Capsule Corporation (Dragon Ball)
- **Form factor:** Vertical pill/capsule silhouette — 257mm tall × 115.7mm diameter
- **Material:** PolyLite PETG — matte white body, teal and orange accents
- **CAD:** Designed in OnShape
- **Print orientation:** Open-top-and-bottom cylinder printed vertically; top and bottom caps snap-fit into place
- **Front:** 1.83" LCD flush mounted upper, circular speaker grille lower, two mic ports flanking display
- **Top cap:** 16mm button hole for Adafruit rugged pushbutton
- **Right side:** Horizontal thermal exhaust vent slots aligned to active cooler fan
- **Lower side:** USB-C port (X1201 charge/power)
- **Rear:** microSD access slot
- **Internals:** M2.5 heat-set inserts for Pi mounting; snap-fit and friction-fit elsewhere

---

## Repository Structure

```
capsule-ai-journal/
├── README.md
├── LICENSE.md                    # Software (MIT) + license index
├── LICENSE-hardware.md           # PCB designs (CERN-OHL-S v2)
├── LICENSE-docs.md               # Enclosure + docs (CC BY 4.0)
│
├── hardware/
│   ├── pi-breakout/              # Pi GPIO breakout board Rev 2.0
│   │   ├── schematic.pdf
│   │   └── gerbers/
│   └── respeaker-breakout/       # ReSpeaker breakout board Rev 1.0
│       ├── schematic.pdf
│       └── gerbers/
│
├── enclosure/
│   ├── renders/                  # Reference renders and drawings
│   ├── stl/                      # Print-ready STL exports
│   └── step/                     # STEP exports for editing
│
├── src/
│   ├── main.py                   # Main pipeline entry point
│   ├── state_machine.py          # State management
│   ├── audio/
│   │   ├── capture.py            # Mic input / whisper.cpp interface
│   │   └── playback.py           # Piper TTS / speaker output
│   ├── inference/
│   │   └── llm.py                # llama.cpp interface
│   ├── display/
│   │   ├── face.py               # Pillow face animation engine
│   │   └── states/               # Per-state animation logic
│   └── wakeword/
│       └── detector.py           # openWakeWord interface
│
├── config/
│   ├── alsa/
│   │   └── asound.conf           # Shared I2S bus ALSA config
│   └── systemd/
│       └── capsule.service       # Systemd service unit
│
├── scripts/
│   ├── setup.sh                  # Full setup script
│   ├── install_audio_driver.sh   # HinTak seeed-voicecard install
│   ├── install_llama.sh          # llama.cpp build + model download
│   ├── install_whisper.sh        # whisper.cpp build + model download
│   └── install_piper.sh          # Piper TTS install
│
└── docs/
    ├── setup-guide.md            # Step-by-step setup instructions
    ├── gpio-map.md               # Full GPIO reference
    ├── bom.md                    # Bill of materials with sourcing links
    └── assembly.md               # Physical assembly guide
```

---

## Setup

Full instructions: [docs/setup-guide.md](docs/setup-guide.md)

Quick order of operations:

1. Flash Raspberry Pi OS Lite 64-bit, enable SSH
2. Install HinTak seeed-voicecard audio driver
3. Verify ReSpeaker mic input and MAX98357A speaker output
4. Build and test llama.cpp with Llama 3.2 3B Q4_K_M
5. Build and test whisper.cpp with Whisper small
6. Install and test Piper TTS
7. Install and test openWakeWord
8. Install Waveshare NV3030B display driver, deploy face animations
9. Wire and test full pipeline with state machine
10. Final enclosure assembly

---

## Status

| Area | Status |
|------|--------|
| Hardware | ✅ All components on hand |
| Enclosure | 🟡 Prototype printed, remaining pieces in progress |
| Breakout PCBs | 🟡 Fabricated, in transit |
| Software | 🔴 Not yet started |
| Face animations | 🟡 Complete as React prototype, pending Pillow translation |

---

## License

This project uses different licenses for different artifact types. See [LICENSE.md](LICENSE.md) for the full breakdown.

- **Software** (`src/`, `scripts/`, `config/`): MIT License
- **Hardware designs** (`hardware/`): CERN Open Hardware Licence v2 — Strongly Reciprocal
- **Enclosure & docs** (`enclosure/`, `docs/`): Creative Commons Attribution 4.0

---

*Built with ❤️ for a kid who has a lot of thoughts and just needs somewhere to put them.*
