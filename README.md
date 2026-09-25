# zbitx-doctor

A knowledge base and diagnostic kit for the [zBitx](https://www.hfsignals.com/) HF transceiver.

The zBitx has two computers. A Raspberry Pi Zero 2 W runs the radio. An RP2040 front panel runs the touchscreen, the encoder, and the display. When the radio fails, this repo helps you find which part failed and why.

This project is not affiliated with HF Signals.

> **A note on how this was made.** This repo is heavily AI generated. It aims to be both human and agent consumable, but may exhibit common AI smells due to how it came together. My zBitx front panel quit on me, and I had Claude dig through the upstream source, the schematics, and a lot of forum threads while I poked at the hardware.

## Contents

| Path | Contents |
|---|---|
| [docs/hardware/](docs/hardware/) | Front panel and main board reference: pins, parts, power |
| [docs/link/](docs/link/) | The Pi to front panel link: bus, protocol, failure modes |
| [docs/firmware/](docs/firmware/) | Stock and drexjj software: build, flash, persistent state, restore to stock |
| [docs/troubleshooting/](docs/troubleshooting/) | Symptom index, runbooks, and case records |
| [docs/community/](docs/community/) | Official resources, forum reports, and known issues |
| [refs/](refs/) | Upstream source repos as git submodules, pinned to the commits the docs cite |

## Get the source references

```bash
git clone --recurse-submodules https://github.com/arunderwood/zbitx-doctor.git
```

## Upstream repos

| Submodule | Role |
|---|---|
| `refs/afarhan_zbitx` | Stock Pi software for v1. The panel link is I2C. |
| `refs/afarhan_zbitxfrontpanel` | Stock front panel firmware for v1 |
| `refs/afarhan_zbitxv2` | Stock Pi software, v2 line. The panel link is Wi-Fi. The I2C poll is commented out. |
| `refs/afarhan_zbitxv2_front_panel` | Front panel firmware for the v2 line. Needs a Pico W. |
| `refs/drexjj_zbitx` | drexjj Pi software |
| `refs/drexjj_zbitxfrontpanel` | drexjj front panel firmware, a fork of the stock firmware |

## Conventions

- Every non-obvious claim links to its source. Code links are GitHub permalinks at a full commit SHA.
- **Unverified:** marks a claim that nobody has confirmed yet. The text says what would confirm it.
- The docs use short sentences in active voice.

## License

- Code (firmware, scripts, tools) uses [GPL-3.0](LICENSE). This matches the license the stock zBitx Pi software states in its source.
- Documentation (`docs/` and all Markdown files) uses [CC BY-SA 4.0](LICENSE-docs).
- The submodules under `refs/` keep their own terms. The front panel firmware repos have no license file. This repo links to them and does not copy their code.
