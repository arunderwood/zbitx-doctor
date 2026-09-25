# zBitx v1 main board: Pi header, panel connector, power

This page covers the main board parts that matter for the Pi to front panel link.
It uses the zBitx v1 schematic, KiCad sheet set "sBitx", Rev v3.3, dated 2024-04-17.

## Sources

| Name in this page | File | Permalink |
|---|---|---|
| Schematic | `schematics_zbitx.pdf`, 4 pages | [afarhan/zbitxv2 @ 2726c57](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/schematics_zbitx.pdf) |
| Same schematic, stock v1 repo | byte-identical copy | [afarhan/zbitx @ c671ac8](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/schematics_zbitx.pdf) |
| Stock v1 Pi software | `afarhan/zbitx` | [afarhan/zbitx @ c671ac8](https://github.com/afarhan/zbitx/tree/c671ac81e244242cb5718cd5b2b63f1d136695e0) |

Page numbers below are PDF page numbers.
The title block "Id" numbers in the PDF do not match the PDF page order.

| PDF page | KiCad sheet | Contents |
|---|---|---|
| 1 | `/` (root) | Barrel jack J1, KEY jack J3, MIC jack J5, EAR jack J6 |
| 2 | `/digital/` | Pi header, 5 V regulator, WM8731 codec, connector J4 |
| 3 | `/pa/` | PA, PIN-diode low-pass filters, SWR bridge |
| 4 | `/main/` | Si5351 synthesizer, DS3231 RTC, RX/TX mixers, LM386 audio amp |

`zbitxv2_schematic.pdf` in the same repo is the zBitx v2 board.
This page does not use it.

## Pi header pin use

The schematic labels the Pi GPIO pins with **wiringPi** numbers, not BCM numbers.
For example, physical pin 7 shows "GPIO7". That is wiringPi 7, which is BCM 4.
The stock software also uses wiringPi numbers ([`sbitx.c` L36-L43](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L36-L43), [`sbitx_gtk.c` L65-L75](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L65-L75)).
Tools like `pinctrl` and `raspi-gpio` use BCM numbers.

Source: schematic page 2, Pi header block, and nets on pages 2 to 4.

| Phys pin | Schematic label (wiringPi) | BCM | Net | Function | Code name |
|---|---|---|---|---|---|
| 2, 4 | 5VDC | - | +5DV | Pi 5 V input from LM7805 U2 | - |
| 3 | SDA | 2 | SDA | Hardware I2C1 data to WM8731 codec (via RD14). 2.2 kΩ pull-up RD5. | kernel `i2c_arm` |
| 5 | SCL | 3 | SDC | Hardware I2C1 clock to WM8731 codec (via RD13). 2.2 kΩ pull-up RD6. | kernel `i2c_arm` |
| 7 | GPIO7 | 4 | PTT | PTT / dot key input. 1 kΩ pull-up R89, 3.3 V zener DD29. | `PTT (7)` |
| 10 | GPIO16 | 15 | RX_LINE | RX enable, drives RX switching on page 3 | `RX_LINE 16` |
| 12 | GPIO1 | 18 | I2S | Codec bit clock via RD9 | ALSA |
| 16 | GPIO4 | 23 | TX_LINE | TX enable, drives PA switching on pages 3 and 4 | `TX_LINE 4` |
| 18 | GPIO5 | 24 | R-A | LPF select A (page 3, Q26) | `LPF_A 5` |
| 22 | GPIO6 | 25 | R-B | LPF select B (page 3, Q27) | `LPF_B 6` |
| 24 | GPIO10 | 8 | R-C | LPF select C (page 3, Q25) | `LPF_C 10` |
| 26 | GPIO-11 | 7 | R-D | LPF select D (page 3, Q24) | `LPF_D 11` |
| 29 | GPIO21 | 5 | R-DASH | Dash key input. 1 kΩ pull-up R90, 3.3 V zener D2. | `DASH (21)` |
| 31 | GPIO22 | 6 | SCL_BB | Bit-banged I2C clock | `SCL 22` |
| 33 | GPIO23 | 13 | SDA_BB | Bit-banged I2C data | `SDA 23` |
| 35 | GPIO24 | 19 | I2S | Codec frame clock via RD10–RD12 | ALSA |
| 38 | GPIO-28 | 20 | I2S | Codec data via RD10–RD12 | ALSA |
| 40 | GPIO-29 | 21 | I2S | Codec data via RD10–RD12 | ALSA |
| 6, 9, 14, 20, 25, 30, 34, 39 | GND | - | GND | Ground | - |

Pins 1, 8, 11, 13, 15, 17, 19, 21, 23, 27, 28, 32, 36 and 37 show a no-connect mark.
Pins 11, 13 and 15 are the sBitx encoder pins. On the zBitx the encoder is on the front panel, not the Pi header.

**Unverified:** The exact I2S pin-to-codec-pin mapping through RD9–RD12. Confirm by zooming schematic page 2 at WM8731 pins 3 to 7.

### The bit-banged I2C bus (SCL_BB / SDA_BB)

The front panel link uses this bus. It is software I2C, not the Pi hardware I2C controller.

- Pins: BCM 6 (SCL_BB, phys 31) and BCM 13 (SDA_BB, phys 33). Source: schematic page 2.
- Pull-ups: R1 and R2, 3.3 kΩ, to the Si5351 3.3 V rail. Source: schematic page 4, next to U1.
- The stock code sets these pins as SDA=wiringPi 23 and SCL=wiringPi 22 ([`si5351v2.c` L8-L9](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/si5351v2.c#L8-L9), [L261](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/si5351v2.c#L261)).
- The stock code also enables the Pi internal pull-ups on wiringPi 22 and 23 at start ([`sbitx_gtk.c` L61-L63](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L61-L63)).

Devices on the bus:

| Address (7-bit) | Device | Where | Evidence |
|---|---|---|---|
| 0x60 | Si5351A synthesizer U1 | Main board, page 4 | [`si5351v2.c` L52](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/si5351v2.c#L52) |
| 0x68 | DS3231 RTC U3 | Main board, page 4 | [`ntputil.c` L21](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/ntputil.c#L21) |
| 0x0A | Front panel RP2040 | Off board, through J4 pins 4 and 5 | [`sbitx_gtk.c` L218](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L218) |
| 0x08 | Probed only when `hw=` is absent | Not fitted on zBitx | [`sbitx.c` L1401-L1408](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L1401-L1408) |

The Si5351 and the panel share one bus.
So audible band noise proves that the Pi can write to this bus at start-up.
The Si5351 must be programmed before the receiver produces noise.

**Unverified:** A stuck bus after start-up would not silence the receiver, because the Si5351 keeps its last setting. Confirm by tuning while the panel is stalled (web UI or telnet) and listening for a change.

## Front panel connector J4

J4 is a 10-pin header, `Conn_01x10`. Source: schematic page 2, left edge.

| J4 pin | Net | Direction (inferred) | Notes |
|---|---|---|---|
| 1 | GND | - | |
| 2 | INT_MIC | Panel to board | Built-in microphone. Also on MIC jack J5 (page 1). |
| 3 | SPKR | Board to panel | LM386 U5 output via C1 47 µF (page 4) |
| 4 | SCL_BB | Both | Bit-banged I2C clock, Pi BCM 6 |
| 5 | SDA_BB | Both | Bit-banged I2C data, Pi BCM 13 |
| 6 | +9VA | Board to panel | Panel power, unregulated supply rail |
| 7 | GND | - | |
| 8 | +9VA ÷ 2 | Board to panel | Supply sense. R14 1 kΩ and R8 1 kΩ divider, CD1 0.1 µF filter. |
| 9 | VREF | Board to panel | SWR bridge reflected voltage (page 3, D28) |
| 10 | VFWD | Board to panel | SWR bridge forward voltage (page 3, D27) |

**Unverified:** J4 is the front panel cable connector. The schematic does not name it. The signal set implies it. Confirm with a continuity test or with the front panel schematic.

### What J4 tells us about the symptom

- The panel gets its power from +9VA on pin 6. It does not depend on the Pi for power.
- The panel measures supply voltage itself, from pin 8. The Pi does not send it.
- The stock Pi software receives the voltage from the panel as a `vbatt` command ([`sbitx_gtk.c` L5000-L5001](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L5000-L5001)).
- So a live voltage readout proves only that the panel main loop runs. It does not prove that the I2C link works.
- The panel also reads the SWR bridge (pins 9 and 10) directly. **Unverified:** the panel sends power and SWR to the Pi. The stock code accepts `power` and `vswr` commands ([`sbitx_gtk.c` L4995-L4999](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4995-L4999)).

## Power

Source: schematic pages 1, 2 and 4.

| Rail | Source | Feeds |
|---|---|---|
| +9VA | Barrel jack J1 (page 1) | PA (page 3), LM386 (page 4), J4 pin 6 (front panel), LM7805 input |
| +5DV | LM7805 U2 from +9VA (page 2) | Pi header pins 2 and 4. AMS1117 U7 and UD4 inputs. |
| 3.3 V (synth) | AMS1117-3.3 U7 from +5DV (page 4) | Si5351 U1, DS3231 U3, SCL_BB/SDA_BB pull-ups |
| 3.3 V (codec) | AMS1117-3.3 UD4 from +5DV (page 2) | WM8731 U4 |
| RTC_BATT | J7 (page 4) | DS3231 VBAT backup |

The Pi takes 5 V through the header, not through its USB port.
The stock install notes say this causes false low-voltage warnings ([`install.txt` L113-L119](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/install.txt#L113-L119)).

The stock software blocks transmit when the panel reports more than 900 in `#batt` and `hw=4` ([`sbitx_gtk.c` L3313-L3316](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L3313-L3316)).
It prints "Reduce the power supply voltage to transmit" in that case.

## Open questions

1. Is J4 the front panel cable? What is the connector on the panel side? Confirm with the panel schematic or a continuity test.
2. What scaling does the panel apply to J4 pin 8? The divider gives about 4 V at 8 V supply, above the RP2040 ADC range. The panel must divide again. Confirm from the panel schematic.
3. Does the zBitx v2 board (`zbitxv2_schematic.pdf`) change J4? The v2 software moves the panel link to WiFi, so v2 may drop SCL_BB/SDA_BB from J4.
4. Which WM8731 pins do RD9–RD12 connect to? This page does not need it for the panel fault.
