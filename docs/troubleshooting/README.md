# Troubleshooting

Runbooks are in progress. Until they exist, start from the open case and the reference pages.

## Cases

| Case | Status | Summary |
|---|---|---|
| [2026-09 panel stall](cases/2026-09-panel-stall.md) | Touch fixed | Touch and encoder dead after a firmware change. Touch recalibration fixed both. Waterfall unconfirmed. |

## Quick facts for any panel fault

- The panel measures battery voltage itself. A live readout proves only that the panel loop runs. See [front-panel-firmware.md](../firmware/front-panel-firmware.md#what-keeps-the-battery-readout-alive).
- The encoder acts only on a field that touch has selected. Dead touch makes the encoder look dead. See [front-panel-firmware.md](../firmware/front-panel-firmware.md#encoder-and-push-switch).
- Touch calibration survives a `.uf2` flash. Hold a finger on the screen at power-on to recalibrate. See [front-panel-firmware.md](../firmware/front-panel-firmware.md#when-calibration-runs).
- The Pi and the panel share one bit-banged I2C bus with the Si5351 and the RTC. See [protocol.md](../link/protocol.md).
- Stock v1 Pi software is `afarhan/zbitx`. The `zbitxv2` line uses Wi-Fi for the panel link. See [pi-software.md](../firmware/pi-software.md).
