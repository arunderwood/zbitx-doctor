# zBitx v1 front panel hardware

This page describes the front panel board of the zBitx v1. It covers the MCU, the display, the touch controller, the encoder, the power supply, and the analog inputs.

The primary source is the schematic `zbitx_front_panel_sch.pdf`. The stock and drexjj repos carry the same file (byte-identical). Links below point to the stock copy: [zbitx_front_panel_sch.pdf, page 1][sch]. The schematic has one page.

For the software that runs on this board, see [front-panel-firmware.md](../firmware/front-panel-firmware.md).

## Summary

| Item | Part | Source |
|---|---|---|
| MCU module | Raspberry Pi Pico footprint (U1, RP2040) | [Schematic p1][sch] |
| Display | ILI9488 SPI TFT, 480 × 320, landscape | [TFT_setup.h L4][s-tft-4], [zbitx.h L2-L3][s-zh-2] |
| Touch | Resistive panel on the display module, read over the shared SPI bus | [TFT_setup.h L16-L18][s-tft-16] |
| Encoder | SW1 "KNOB", quadrature encoder with push switch | [Schematic p1][sch] |
| Regulator | U2 L78L05 (SOT-89), 9 V in, 5 V out | [Schematic p1][sch] |
| Link to main board | J3, 10-pin connector: I2C, 9 V, analog sense, audio | [Schematic p1][sch] |

**Unverified:** The module is a plain Pico or a Pico W. The schematic symbol says "Raspberry Pi Pico". The Pico W has the same pinout. The stock v1.07d binary targets the Pico W and also runs on a plain Pico (see [firmware page](../firmware/front-panel-firmware.md#firmware-variants)). A photo of the module, or the USB product string in BOOTSEL mode, confirms it.

## MCU pin map

Pin numbers are Pico header pins. "Code" is where the firmware sets the pin. The stock and drexjj firmware use the same pins.

| GPIO | Pico pin | Function | Code | Schematic net | Connector |
|---|---|---|---|---|---|
| GP0 | 1 | Not connected | — | — | — |
| GP1 | 2 | Encoder push switch (active low) | `ENC_S 1` [zbitx.h L31][s-zh-31] | SW1 S2 | — |
| GP2 | 4 | Encoder phase | `ENC_A 2` [zbitx.h L32][s-zh-31] | SW1 **B** | — |
| GP3 | 5 | Encoder phase | `ENC_B 3` [zbitx.h L33][s-zh-31] | SW1 **A** | — |
| GP4 | 6 | Display D/C | `TFT_DC 4` [TFT_setup.h L13][s-tft-9] | DC | J1-5 |
| GP5 | 7 | Display reset | `TFT_RST 5` [TFT_setup.h L14][s-tft-9] | RESET | J1-4 |
| GP6 | 9 | I2C1 SDA (panel is target) | `Wire1.setSDA(6)` [sketch L357][s-ino-357] | I2C_SDA | J3-5 |
| GP7 | 10 | I2C1 SCL | `Wire1.setSCL(7)` [sketch L358][s-ino-357] | I2C_SCL | J3-4 |
| GP8 | 11 | Not connected | — | — | — |
| GP9 | 12 | Touch chip select | `TOUCH_CS 9` [TFT_setup.h L16][s-tft-16] | T_CS | J1-11 |
| GP10 | 14 | SPI1 SCK | `TFT_SCLK 10` [TFT_setup.h L11][s-tft-9] | SCK, T_CLK | J1-7, J1-10 |
| GP11 | 15 | SPI1 MOSI | `TFT_MOSI 11` [TFT_setup.h L12][s-tft-9] | MOSI | J1-6, J1-12 |
| GP12 | 16 | SPI1 MISO, touch data out only | `TFT_MISO 12` [TFT_setup.h L10][s-tft-9] | T_DO | J1-13 |
| GP13 | 17 | Display chip select | `TFT_CS 13` [TFT_setup.h L15][s-tft-9] | CS | J1-3 |
| GP14-GP22 | 19-29 | Not connected | — | — | — |
| GP26 / ADC0 | 31 | Reflected power sense | `VREF A0` [zbitx.h L35][s-zh-31] | VREF | J3-9 |
| GP27 / ADC1 | 32 | Forward power sense | `VFWD A1` [zbitx.h L34][s-zh-31] | VFWD | J3-10 |
| GP28 / ADC2 | 34 | Battery voltage sense | `analogRead(A2)` [sketch L260][s-ino-258] | VBATT | J3-8 |
| RUN | 30 | Not connected | — | — | — |
| SWCLK, SWDIO | 41, 43 | Not connected on this board | — | — | — |

The encoder, D/C, reset, touch and SPI pins match between the code and the schematic.

### Code and schematic disagreements

- **Encoder phase names.** The code calls GP2 `ENC_A`. The schematic connects GP2 to encoder pin **B**. This only reverses the sense of rotation in the names. The firmware tuning direction is what the owner sees, so this is not a fault.
- **Forward and reflected power.** `zbitx.h` defines `VFWD` as A1 and `VREF` as A0, which matches the schematic. But `measure_voltages()` does not use these names. It reads A0 into the forward value and A1 into the reflected value ([sketch L258-L259][s-ino-258]). So the code swaps forward and reflected relative to the schematic. **Unverified:** The main board may also swap VFWD and VREF on J3. A TX test into a known load confirms which is which.

## Display

- Controller: ILI9488, set by `ILI9488_DRIVER` ([TFT_setup.h L4][s-tft-4]).
- Resolution: 480 × 320. The firmware uses rotation 3, which is landscape ([screen_gx.cpp L59][s-gx-53]).
- Bus: RP2040 SPI1 (`TFT_SPI_PORT 1`). Write clock 66 MHz, read clock 20 MHz, touch clock 2.5 MHz ([TFT_setup.h L9, L31-L35][s-tft-9]).
- The display SDO pin (J1-9) is not connected. The firmware never reads the display. The MISO line goes only to the touch controller ([TFT_setup.h L10][s-tft-9]).
- Backlight: J1-8 connects to 3V3. The backlight is always on. The firmware cannot dim it or turn it off.

### J1 display connector

| J1 pin | Net | Module function |
|---|---|---|
| 1 | 3V3 | VCC |
| 2 | GNDPWR | GND |
| 3 | CS (GP13) | Display CS |
| 4 | RESET (GP5) | Display reset |
| 5 | DC (GP4) | Data / command |
| 6 | MOSI (GP11) | Display SDI |
| 7 | SCK (GP10) | Display SCK |
| 8 | 3V3 | Backlight (LED) |
| 9 | Not connected | Display SDO |
| 10 | T_CLK (GP10) | Touch clock |
| 11 | T_CS (GP9) | Touch CS |
| 12 | MOSI (GP11) | Touch DIN |
| 13 | T_DO (GP12) | Touch DOUT |
| 14 | Not connected | Touch IRQ |

**Unverified:** The "module function" names for pins 8, 9 and 14 come from the common 14-pin 3.5-inch ILI9488 module pinout. The schematic shows only the net and the pin number. The silkscreen on the display module confirms the names.

## Touch controller

- The touch panel is resistive. The display module carries the touch controller.
- The firmware reads touch through TFT_eSPI (`tft.getTouch()`), [screen_gx.cpp L174-L176][s-gx-174]. TFT_eSPI supports only XPT2046-type resistive controllers.
- **Unverified:** The chip is an XPT2046 or a clone (for example HR2046). The part marking on the display module confirms it.
- The touch controller shares SPI1 with the display. It has its own chip select on GP9.
- T_IRQ (J1-14) is not connected. The firmware polls the touch controller once per main-loop pass.
- A touch counts only when the pressure reading is above a threshold. The default threshold is 600 raw units ([TFT_eSPI Touch.h L18][tft-h-18], [Touch.cpp L125-L162][tft-cpp-125]).
- The firmware converts raw readings to screen coordinates with calibration data that it stores in flash. Bad calibration data makes every touch fall off-screen, and the library then reports "no touch". See [persistent storage](../firmware/front-panel-firmware.md#persistent-storage-on-the-panel).

## Encoder and push switch

- SW1 is a quadrature encoder with a push switch.
- Encoder common (C) connects to GND. Phases A and B connect to GP3 and GP2.
- The push switch connects between GP1 (S2) and GND (S1). A press pulls GP1 low.
- The board has no external pull-up resistors and no debounce capacitors. The firmware enables the RP2040 internal pull-ups on GP1, GP2 and GP3 ([sketch L366-L368][s-ino-366]).
- The firmware reads rotation with pin-change interrupts on GP2 and GP3 ([sketch L372-L373][s-ino-366]). It polls the push switch in the main loop ([sketch L295-L301][s-ino-295]).
- A press held at power-on sends the RP2040 into its USB bootloader (BOOTSEL) ([sketch L377-L378][s-ino-377]).

## Power

```
J3-6 (9 V) ──┬── U2 L78L05 IN
             C2 1 µF
U2 OUT (5 V) ──┬── D1 (M7) ── Pico VSYS (pin 39)
               C1 1 µF
Pico 3V3 (pin 36) ── display VCC (J1-1) and backlight (J1-8)
```

- The main board supplies 9 V on J3-6. U2, an L78L05 in SOT-89, makes 5 V.
- D1 (M7, a 1 A general-purpose rectifier) feeds VSYS. VSYS sees about 5 V minus one diode drop.
- The Pico's on-board regulator makes 3V3. The 3V3 rail powers the display and its backlight.
- Pico VBUS (pin 40) and 3V3_EN (pin 37) are not connected on this board.
- **Unverified:** When a USB cable connects to the Pico, USB power reaches VSYS through the Pico's own VBUS diode. D1 then blocks current back into U2. This follows the standard Pico design. A meter check on VSYS with only USB connected confirms it.

### Ground symbols

The schematic uses three ground symbols: GND, GNDPWR and Earth. The mic and speaker returns use Earth (J3-1). The regulator and display use GNDPWR.

**Unverified:** The schematic does not show a tie between these ground nets. The PCB layout, or a continuity test between J3-1, J3-7 and a Pico GND pin, confirms the tie.

## J3 main-board connector

| J3 pin | Net | Direction (panel view) | Use |
|---|---|---|---|
| 1 | Earth | — | Mic and speaker return |
| 2 | MIC | Out | Mic element (J "MIC" pin 1) |
| 3 | SPEAKER | In | Speaker (J "SPEAKER" pin 1) |
| 4 | I2C_SCL | In/out | To GP7 |
| 5 | I2C_SDA | In/out | To GP6 |
| 6 | 9V | In | Panel supply |
| 7 | GND | — | Ground |
| 8 | VBATT | In | Battery sense to GP28 / ADC2 |
| 9 | VREF | In | Reflected power sense to GP26 / ADC0 |
| 10 | VFWD | In | Forward power sense to GP27 / ADC1 |

- The mic and speaker pass through the panel. The RP2040 does not touch audio.
- The panel has no I2C pull-up resistors. **Unverified:** The pull-ups sit on the main board or the Pi. See the link docs for the bus.

## Battery voltage sense

- VBATT enters on J3-8 and goes straight to GP28 / ADC2. The panel has no divider, no filter capacitor and no protection on this line.
- The main loop reads ADC2 every 50 ms ([sketch L255-L277][s-ino-258]).
- The firmware computes `vbatt = (500 × ADC) / 278` ([sketch L260][s-ino-258]). The unit is 10 mV. The display shows `vbatt / 100` volts with one decimal ([fields.ino L490-L492][s-f-481]).
- With the arduino-pico default 10-bit ADC and a 3.3 V reference, 1 V of battery gives about 55.6 counts. That is about 0.18 V at the ADC pin per battery volt.
- **Unverified:** The main board carries a divider of about 5.6 : 1. At 8.0 V the ADC pin sees about 1.44 V. The main-board schematic, or a meter on J3-8, confirms the ratio.
- The battery readout needs only the ADC, the main loop and the display. It does not need the Pi or the I2C link. See [what keeps the battery readout alive](../firmware/front-panel-firmware.md#what-keeps-the-battery-readout-alive).

## Hardware checks this page supports

1. Encoder switch: hold the knob pressed at power-on. A good switch sends the Pico into BOOTSEL. A USB host then sees a drive named `RPI-RP2`.
2. Touch path: hold a finger on the screen at power-on. A working touch controller starts the calibration screen (corner arrows).
3. Battery sense: compare the on-screen voltage with a meter on the battery. A large error points at the main-board divider or J3-8.
4. I2C lines: measure GP6 and GP7 at idle. Both lines idle high when the pull-ups work.

## Open questions

- Is the module on shipped v1 panels a Pico or a Pico W?
- Is the touch controller an XPT2046 or a clone? Does the clone change pressure readings?
- Where are the I2C pull-ups, and what value are they?
- What is the VBATT divider ratio on the main board?
- Does the main board swap VFWD and VREF on J3, or does the panel code swap them?
- Are GND, GNDPWR and Earth tied on the panel PCB?
- Is the Pico USB connector reachable without opening the case?

[sch]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sch.pdf
[s-tft-4]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/TFT_setup.h#L4
[s-tft-9]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/TFT_setup.h#L9-L35
[s-tft-16]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/TFT_setup.h#L16-L18
[s-zh-2]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx.h#L2-L3
[s-zh-31]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx.h#L31-L35
[s-ino-258]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L248-L278
[s-ino-295]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L295-L301
[s-ino-357]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L357-L362
[s-ino-366]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L366-L373
[s-ino-377]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L377-L378
[s-gx-53]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/screen_gx.cpp#L53-L83
[s-gx-174]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/screen_gx.cpp#L174-L176
[s-f-481]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L481-L492
[tft-h-18]: https://github.com/Bodmer/TFT_eSPI/blob/6ae4c97c5a07bd64fce8993e9cfa95d9a81e7c81/Extensions/Touch.h#L18
[tft-cpp-125]: https://github.com/Bodmer/TFT_eSPI/blob/6ae4c97c5a07bd64fce8993e9cfa95d9a81e7c81/Extensions/Touch.cpp#L125-L162
