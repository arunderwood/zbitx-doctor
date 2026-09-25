# zBitx front panel firmware

This page describes the firmware that runs on the RP2040 front panel. It covers the three firmware lines, the build and flashing. It also covers stored data, startup and input handling.

For the board, pins and power, see [front-panel.md](../hardware/front-panel.md). This page covers the panel side of the Pi link only. The link docs cover the bus and the wire protocol.

## Key facts for diagnosis

1. The panel has no link state machine. It never gates touch or the encoder on the Pi. See [input handling](#input-handling).
2. The encoder and its push switch do nothing until a touch selects a field. A dead touch screen makes the encoder look dead too.
3. The battery readout comes from the panel's own ADC. It proves the main loop runs. It proves nothing about the Pi link.
4. Touch calibration lives in the last 4 KB sector of flash. Flashing a UF2 does not erase it. Bad calibration data makes every touch read as "no touch".
5. The case photo shows the drexjj screen layout, not the stock layout. See [which firmware is running](#which-firmware-is-running).

## Firmware variants

| | Stock (v1) | drexjj fork | v2 line |
|---|---|---|---|
| Repo | [afarhan/zbitxfrontpanel][s-repo] | [drexjj/zbitxfrontpanel][d-repo] | [afarhan/zbitxv2_front_panel][v-repo] |
| Commit cited | `dc4a5a5dcf96…` | `ec075bf36083…` | `63caacfba0c6…` |
| Version string | v1.07d ([sketch L375][s-ino-375]) | 5.13f ([zbitx.h L7][d-zh-7]) | v4.00 ([sketch L444][v-ino-444]) |
| Link to the Pi | I2C, panel is target 0x0A | I2C, panel is target 0x0A | Wi-Fi TCP to 192.168.4.1:8081 |
| Prebuilt file | `zbitx_front_panel_sw.ino.uf2` | `build/zbitxfrontpanel.v513f.uf2` | `zbitx_front_panel_v2.ino.uf2` |
| arduino-pico version (from binary) | 4.6.0 | 6.0.0 | 5.5.1 |
| Board target (from binary) | Pico W | Pico (not W) | Pico W |
| Flash range written | 0x10000000–0x1006E100 | 0x10000000–0x1002DF00 | 0x10000000–0x1007DD00 |
| Touch calibration store | EEPROM bytes 0–11 | EEPROM bytes 0–11 | EEPROM `struct saved`, bytes 16–35 |
| Display driver | TFT_eSPI | TFT_eSPI | Own driver `tft_ili9488.cpp` |

How the "from binary" rows were found: each UF2 was parsed block by block for its target addresses. The embedded strings give the arduino-pico build path and the USB product name. The stock and v2 binaries contain the CYW43 Wi-Fi firmware and the string `RaspberryPi.PicoW`. The drexjj binary contains neither. All three UF2 files use the RP2040 family ID `0xE48BFF56`.

### Stock v1 firmware

- This is the firmware that pairs with the I2C era of the stock Pi software.
- The binary targets the Pico W. It still runs on a plain Pico. arduino-pico probes the board at boot and skips Wi-Fi setup on a plain Pico ([cyw43_wrappers.cpp L116-L151][ap-cyw]).

### drexjj fork

- It forks the stock firmware at `dc4a5a5`. It adds 58 commits (2026-04-21 to 2026-09-23).
- It keeps the I2C link, the pins, the encoder code, and the touch calibration storage unchanged.
- Some menu items send new commands (`MACROLIST`, `VERSION`, `SHUTDOWN`, `WIFI …`, `USB toggle`). These need the drexjj Pi software ([fields.ino L360-L472][d-f-menu]). **Unverified:** How the stock Pi software handles these commands. The link docs cover this.

### v2 line (Wi-Fi)

- It branches from the stock history after `dc4a5a5`. Its first new commit is 2026-03-31.
- It has no I2C code. It joins the Wi-Fi network `zbitx` and opens TCP port 8081 on 192.168.4.1 ([sketch L42-L43][v-ino-42], [L493][v-ino-493], [L537][v-ino-537]).
- It needs a Pico W, because it uses the on-board Wi-Fi.
- It runs on two cores. Core 1 runs the UI, touch, encoder and display ([L424-L458][v-ino-424]). Core 0 runs Wi-Fi and TCP ([L508-L569][v-ino-508]).
- It pairs with the current stock Pi software. At commit `2726c57` the Pi runs a TCP server on port 8081 ([remote.c L208][p-remote]) and an access point `zbitx` ([setup-ap.sh L5][p-ap]). The Pi's I2C panel poll, `zbitx_poll()`, sits inside a comment block ([sbitx_gtk.c L4074-L4164][p-poll]).
- **Conclusion:** "v2" names a software line, not a new panel board. It suits v1 hardware only with a Pico W module and the Wi-Fi Pi software.
- **Unverified:** Which stock SD image the owner used. An image built after 2026-04-07 has no active I2C panel code. The link docs confirm this.

## Which firmware is running

The case photo ([case record](../troubleshooting/cases/2026-09-panel-stall.md)) shows a drexjj layout. It does not match the stock layout.

| Screen detail | Stock v1.07d | drexjj 5.13f | Photo |
|---|---|---|---|
| Top row order | MENU, MODE, DRIVE, IF, **RIT**, FREQ, **AUDIO** ([L9-L15][s-fl-9]) | MENU, MODE, DRIVE, IF, **AUDIO**, FREQ ([L15-L24][d-fl-15]) | MENU, MODE, DRIVE, IF, AUDIO, FREQ |
| MODE box width | 48 px, same as MENU | 64 px, wider than MENU | Wider than MENU |
| Row 2, right end | **SET** ([L21][s-fl-21]) | **AGC** ([L29][d-fl-29]) | AGC |
| Boot banner | `zBitx firmware v1.07d` then `Waiting for the zBitx to start...` ([L375][s-ino-375]) | `Waiting for the zBitx to start...` only ([L397][d-ino-397]) | Not recorded |

The AUDIO position first appears in drexjj commit `847259e` (v1.12, 2026-08-25). So the photo shows drexjj v1.12 or later.

The owner confirms the photo shows the drexjj phase. To check that a stock flash took, power on and read the console banner. Stock v1.07d prints its version line. drexjj prints no version.

## Build

### Toolchain

- Core: earlephilhower arduino-pico ([drexjj README L19-L27][d-readme]).
- Board: 2 MB flash. The drexjj README uses `--fqbn rp2040:rp2040:rpipico` ([README L42-L49][d-readme]). The drexjj `build.sh` uses `rpipicow` ([build.sh L3-L6][d-build]). The published 5.13f binary is a plain-Pico build.
- Display library: TFT_eSPI ([drexjj README L32][d-readme]). The README also installs Adafruit GFX. No source file includes it.
- Other libraries: `Wire` and `EEPROM` from the core.
- **Unverified:** The TFT_eSPI version for each binary. The binaries do not name it. V2.5.43 is the current release. This page cites that release.

### Display setup file

- The pin setup lives in `TFT_setup.h` in the sketch folder ([stock TFT_setup.h][s-tft]). The three repos carry the same file.
- The build must also define `USER_SETUP_LOADED` and `ILI9488_DRIVER`. The drexjj repo ships these in `platform_local.txt` ([L1-L2][d-plat]). You rename it to `platform.local.txt` and copy it into the arduino-pico core folder ([Read_First.txt][d-readfirst]).
- The stock repo describes this file in `Read_Me.ino` but does not include it.
- TFT_eSPI includes `<tft_setup.h>` when the compiler finds it ([TFT_eSPI.h L59-L66][tft-h-inc]). The repo file is named `TFT_setup.h`. On a case-sensitive file system the names do not match.
- The drexjj README gives a fallback. Copy `TFT_setup.h` over the library `User_Setup.h` ([README L57-L67][d-readme]).
- **Unverified:** How the pins reach TFT_eSPI in each published binary. A build log with `-v` shows which setup file the compiler used.

### Build command

```console
arduino-cli compile --fqbn rp2040:rp2040:rpipico --output-dir ./build .
```

## Flash

1. Put the RP2040 into BOOTSEL mode. Use one of these methods:
   - Hold the encoder knob pressed while you power on the panel. The firmware sees GP1 low and calls `reset_usb_boot()` ([stock L377-L378][s-ino-377], [drexjj L399-L400][d-ino-399]). This needs working firmware and a working switch.
   - Hold the Pico BOOTSEL button while you connect USB. This works with any firmware.
2. Connect the Pico USB port to a computer. A drive named `RPI-RP2` appears.
3. Copy the UF2 file to the drive. The Pico writes it and reboots.

**Unverified:** Whether the Pico USB port is reachable without opening the case.

## Persistent storage on the panel

### What the v1 and drexjj firmware store

- The firmware uses the arduino-pico `EEPROM` library. `EEPROM.begin(512)` runs in `screen_init()` ([screen_gx.cpp L56][s-gx-56]).
- Only touch calibration is stored. It uses bytes 0–11 ([L44-L51][s-gx-44], [L70-L75][s-gx-70]). TFT_eSPI uses the first 10 bytes as five 16-bit values: x0, x1, y0, y1, flags ([Touch.cpp L320-L326][tft-cal-out]).
- The drexjj fork does not change this code.
- Nothing else persists. The firmware does not use LittleFS. Frequency, mode and settings live on the Pi.

### Where the EEPROM lives in flash

- arduino-pico emulates EEPROM in one 4 KB flash sector ([EEPROM.cpp L37-L45][ap-eeprom]).
- For every 2 MB Pico and Pico W flash option, the sector starts at 0x101FF000. That is the last 4 KB of flash ([boards.txt L91][ap-b-pico], [L314][ap-b-picow] for 6.0.0; [L90, L305][ap-b-460] for 4.6.0).
- `EEPROM.commit()` erases the whole sector and writes the buffer back ([EEPROM.cpp L110-L126][ap-eeprom-commit]).

### Does flashing a UF2 erase it?

No. A UF2 writes only the flash blocks it contains. The three prebuilt files end at or below 0x1007DD00. None reaches 0x101FF000. So calibration data survives every reflash, including a return to stock.

`flash_nuke.uf2` from Raspberry Pi erases the whole flash, including this sector. After a nuke, the sector reads 0xFF. See the next section for what that does to touch.

### When calibration runs

- At boot, `screen_init()` reads the touch once, before it loads the stored calibration ([L64][s-gx-64]).
- If you hold a finger on the screen, the firmware waits for release. Then it runs `tft.calibrateTouch()` and writes the result ([L66-L75][s-gx-64]).
- The calibration screen shows arrows in each corner. You touch each corner in turn. The routine waits forever for each touch ([Touch.cpp L270][tft-cal-wait]).
- If you do not touch the screen at boot, the firmware loads bytes 0–11 and calls `setTouch()` ([L78, L82][s-gx-64]).
- The boot check uses the TFT_eSPI default calibration, because `setTouch()` has not run yet ([Touch.h L38-L39][tft-h-def]). So the check works even when the stored data is bad.

### How bad calibration data kills touch

- `getTouch()` converts the raw reading with the calibration values ([Touch.cpp L201-L223][tft-conv]).
- If the result is off-screen, `getTouch()` returns "no touch" ([L188][tft-gettouch]).
- Example: an erased sector gives x0 = x1 = y0 = y1 = 0xFFFF and all flag bits set. Every raw X then converts to about 929–960. That is past the 480-pixel width. So every touch reads as "no touch".
- Other bad values, such as a calibration done with a stuck or noisy panel, can do the same thing. They can also map touches to the wrong place.
- **Unverified:** What bytes 0–11 hold on the affected panel. A test firmware that prints them over USB serial confirms it.

### v2 storage is not compatible

- The v2 firmware stores `struct saved` at offset 0. It holds a magic word `0x00C0FFEE`, an ID, calibration and Wi-Fi keys ([zbitx.h L157-L162][v-zh-saved]).
- With 8-byte alignment, the calibration sits at bytes 16–35. **Unverified:** The compiler layout. A `sizeof` and `offsetof` print confirms it.
- If v2 finds no magic word, it zeroes the block and does not save ([storage.cpp L23-L27][v-storage]). Zero calibration then maps touches to wrong places, because the v2 driver does not reject off-screen points ([tft_ili9488.cpp L743-L754][v-tft]).
- If v2 calibrates, it writes its block. A later v1 or drexjj firmware then reads the magic word and ID as calibration.
- Result: a switch between the v2 line and the v1 line breaks touch until you recalibrate.

## Startup sequence (v1 and drexjj)

The two firmwares follow the same steps. Line links are to the stock sketch unless marked.

1. `setup()` starts USB serial at 115200 baud ([L346][s-ino-346]). It does not wait for a host.
2. `screen_init()` starts EEPROM, starts the display, and runs the calibration check ([L350][s-ino-346], [screen_gx.cpp L53-L83][s-gx-53]).
3. `field_init()` loads the field table and clears the waterfall buffer ([fields.ino L21-L41][s-f-init]).
4. `field_set("MODE","CW")` sets the CW layout ([L354][s-ino-346]). That layout shows the console box ([fields.ino L122-L123][s-f-panel]).
5. `Wire1` starts as an I2C target at address 0x0A on GP6 and GP7. It registers `on_receive` and `on_request` ([L357-L362][s-ino-357]).
6. The encoder pins get pull-ups and pin-change interrupts ([L366-L373][s-ino-366]).
7. The banner goes into the console ([L375][s-ino-375]).
8. If the knob is pressed, the panel reboots into BOOTSEL ([L377-L378][s-ino-377]).
9. `loop()` runs `ui_slice()`, then `measure_voltages()`, then waits 1 ms ([L397-L407][s-ino-loop]).

### The "waiting" screen

- The waiting screen is the default field values plus the banner in the console box.
- No flag records "Pi connected". No timer detects a lost link. No heartbeat exists.

### What moves the panel to the ready screen

- The Pi writes text blocks of the form `{LABEL value}` over I2C.
- `on_receive()` runs in interrupt context and copies bytes into a 4000-entry queue ([L238-L244][s-ino-rx]).
- `ui_slice()` drains the whole queue at the start of each pass ([L287-L288][s-ino-slice]).
- `command_tokenize()` splits blocks and calls `field_set()` ([L140-L180][s-ino-tok]).
- A `MODE` value calls `field_set_panel()`, which swaps the button layout ([fields.ino L203-L204][s-f-set]).
- So the "ready screen" is the Pi's field values drawn over the defaults. If the Pi stops writing, the panel shows the last values forever.
- The Pi reads the panel with I2C read requests. `on_request()` returns one pending field change. With no change, it returns `vbatt`, `power` and `vswr` ([L203-L235][s-ino-req]).

### What the waterfall needs

- The Pi must send `{WF <characters>}` blocks. Each block calls `waterfall_update()` ([fields.ino L181-L200][s-f-set]).
- `waterfall_draw()` copies the buffer to the screen unless the WF value is `OFF` ([waterfall.cpp L93-L97][s-wf-draw]). WF starts as `ON` ([fields_list.h L123][s-fl-wf]).
- Only `waterfall_update()` draws the passband strip and the yellow spectrum trace into the buffer ([drexjj waterfall.cpp L112-L155][d-wf]).
- The draw copies 144 rows. The WF field is 176 rows high. The bottom 32 rows keep old pixels.

### Reading the case photo

- The photo shows the grey passband strip, the white, green and red lines, and a flat yellow trace. So at least one WF block arrived and parsed.
- The flat trace sits at the bottom. So the spectrum values were near zero. Near-zero values also give an almost black heat map.
- So the photo cannot tell "WF stopped" from "WF keeps arriving with near-zero data".
- The lower grey block in the waterfall area sits at screen rows 240–272. These are the 32 rows the draw never repaints. The block is stale pixels, not a fault.
- The S-meter shows two lit bars. The Pi sends the S-meter value (`SMETER`). If these bars move with band noise, the Pi still sends data to the panel.

## Input handling

### Touch

- Each `ui_slice()` pass reads the touch after it draws the fields ([L318-L323][s-ino-slice]). No `return` sits between the draw and the touch read.
- A new touch on a visible field calls `field_select()` ([L326-L334][s-ino-slice]). This changes the selection or value on the panel, redraws it, and flags it for the Pi.
- A held touch does nothing more, except auto-repeat on keyboard keys ([L335-L340][s-ino-slice]).
- The drexjj fork reads the touch four times per pass. It rejects the touch if any read fails or the reads spread more than 8 px ([drexjj screen_gx.cpp L174-L206][d-gx-read]). It also grows each field's hit box by 4 px ([drexjj fields.ino L315-L336][d-f-at]).

### Encoder and push switch

- Rotation interrupts update `wheel_move` ([L42-L66][s-ino-enc]). The main loop turns more than three steps (one step on FREQ) into UP or DOWN ([L303-L316][s-ino-slice]).
- A switch press calls `field_input(ENTER)` ([L295-L301][s-ino-slice]).
- `field_input()` returns at once when no field is selected ([fields.ino L628-L631][s-f-input]).
- `f_selected` starts as NULL ([fields.ino L9][s-f-sel]). Only `field_select()` sets it ([L302][s-f-sel302]), and only a touch calls `field_select()` from the main loop.
- **Result:** After boot, the knob and its switch do nothing until a touch selects a field. This is normal on a healthy panel.

### Link state and input

- No input path reads link state. Touch and the encoder change the display without the Pi.
- Changes reach the Pi only when the Pi next reads the panel.
- A value from the Pi does not overwrite a field for 1 s after a local change ([L154][s-ino-tok]).

## What keeps the battery readout alive

1. `loop()` calls `measure_voltages()` every pass. It reads ADC2 every 50 ms into `vbatt` ([L248-L278][s-ino-meas]).
2. `ui_slice()` calls `field_draw_all()` every pass ([L318][s-ino-slice]).
3. `field_draw_all()` calls `smeter_draw()` at its end ([fields.ino L776-L777][s-f-drawall]).
4. `smeter_draw()` redraws on every 25th call. It prints `vbatt` in the top right ([fields.ino L481-L492][s-f-smeter]).

None of these steps needs the Pi. The I2C interrupts do not block them.

The drexjj fork skips `smeter_draw()` while a menu dialog is open ([drexjj fields.ino L1074-L1077][d-f-drawall]). So a live battery readout on drexjj also proves the panel is not inside a dialog.

**Conclusion:** A live battery readout proves that `ui_slice()` reaches the touch read on every pass. So the touch fault is in the touch data, not in a stalled loop. The data fails in one of these ways:

- `getTouch()` returns "no touch": pressure below threshold, or calibration maps the point off-screen.
- The point maps to a spot with no visible field.
- A stuck touch holds `mouse_down` true, so no new selection starts.

## Differences between drexjj and stock

`git diff` from stock `dc4a5a5` to drexjj `ec075bf`: 17 files, 962 lines added, 248 removed. This table lists the changes that touch the areas of this fault.

| Area | drexjj change | Source | Effect on this fault |
|---|---|---|---|
| Touch read | Four agreeing samples, 8 px limit | [screen_gx.cpp L174-L206][d-gx-read] | Stricter. A noisy panel can fail more often. |
| Touch calibration and EEPROM | No change | [screen_gx.cpp L44-L83][d-gx-cal] | None |
| Hit test | Hit box grows by 4 px | [fields.ino L315-L336][d-f-at] | More forgiving |
| Focus posting | TEXT, FREQ and NUMBER fields no longer post on focus | [fields.ino L571-L590][d-f-post] | Changes what the Pi receives |
| Menus | New dialogs send new commands to the Pi | [fields.ino L360-L472][d-f-menu] | Needs drexjj Pi software |
| Dialog loop | Drains the I2C queue while a dialog is open | [fields.ino L139-L150][d-f-dialog] | None at the ready screen |
| Battery overlay | Hidden while a dialog is open | [fields.ino L1074-L1077][d-f-drawall] | See above |
| Waterfall | Fixed 240-sample buffer; scales to 480 px in voice modes | [waterfall.cpp L112-L179][d-wf] | Display only |
| Telemetry | `volatile` values; SWR shows 10.0 with no RF | [sketch L286-L292][d-ino-swr] | Sends `vswr 100` at idle |
| I2C setup, address, encoder, BOOTSEL | No change | [sketch L377-L400][d-ino-377] | None |

## Self-test features in the firmware

- **USB serial.** The firmware opens USB CDC serial at 115200 baud. It prints only a few error lines that start with `#`, such as `#Calibrating the screen` ([screen_gx.cpp L65][s-gx-64]). It reads no commands.
- **Simulated waterfall.** `simulate_waterfall()` fills the waterfall from ADC0 noise. The call in `loop()` is commented out ([L381-L402][s-ino-sim]). A test build can enable it to exercise the display with no Pi.
- **Link counters.** `dcount` counts bytes received ([L237-L240][s-ino-rx]). `req_count` and `total` count replies ([L198-L199][s-ino-wire]). The queue counts overflows ([queue.cpp L34][s-q]). Nothing prints them.
- **Knob-at-boot BOOTSEL.** This tests the push switch with no other tools.
- **Touch-at-boot calibration.** This tests the touch controller, and it rewrites the stored calibration.
- The v2 firmware prints more over USB serial, for example the stored block and Wi-Fi state ([storage.cpp L80-L88][v-storage]).

A hardware-test firmware can add these at low cost:

- Raw touch X, Y and pressure.
- Encoder counts and switch state.
- Raw ADC counts and I2C byte counters.
- A hex dump of EEPROM bytes 0–11.

## Tests this page supports

Run these in order. Each one takes under 5 minutes.

1. **Firmware identity.** Power on and read the console banner. Stock v1.07d shows its version. Also compare the layout with [the table above](#which-firmware-is-running).
2. **Pi-to-panel data.** Watch the S-meter bars for 30 seconds. Moving bars mean the Pi still sends data.
3. **Touch controller.** Hold a finger firmly on the screen during power-on. Corner arrows mean the touch controller reads. Touch all four corners to save new calibration. Then test touch.
4. **Encoder switch.** Hold the knob pressed during power-on. A `RPI-RP2` drive on a USB host means the switch and GP1 work.
5. **Encoder rotation.** After step 3 works, touch DRIVE and turn the knob. The value should change.

If step 3 shows no arrows, suspect the touch hardware. Check the controller, the module flex, and the T_CS, T_DO and SCK lines.

## Hypotheses for the case

| Hypothesis | Explains | Does not explain | Test |
|---|---|---|---|
| Bad touch calibration in flash | Touch dead, encoder dead, survives reflash | No waterfall | Test 3 |
| Touch hardware fault | Touch dead, encoder dead, survives reflash | No waterfall | Test 3 shows no arrows |
| Pi stops sending after the first data | No waterfall | Dead touch (touch is local) | Test 2 |
| Panel still runs drexjj | Photo layout | Dead touch on its own | Test 1 |

Ruled out by the code:

- A hung main loop. The battery readout keeps changing.
- A panel stuck in a dialog. The drexjj fork hides the battery readout in dialogs. The stock fork would show the dialog.
- Input gated on the link. No such code exists.
- A UF2 reflash that resets calibration. The UF2 files do not reach the EEPROM sector.

## Open questions

- Which firmware runs on the affected panel now? What does the boot banner say?
- What do EEPROM bytes 0–11 hold on the affected panel?
- Did the owner ever flash the v2 Wi-Fi firmware, or touch the screen during a boot?
- Which stock SD image did the owner use? Does it still run the I2C panel poll?
- Which TFT_eSPI version built each published binary?
- Does the drexjj four-sample touch filter reject touches on panels with weak pressure readings?
- How does the stock Pi software handle the drexjj-only commands?

[s-repo]: https://github.com/afarhan/zbitxfrontpanel/tree/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37
[d-repo]: https://github.com/drexjj/zbitxfrontpanel/tree/ec075bf36083e0f04a84617f647dee8491429408
[v-repo]: https://github.com/afarhan/zbitxv2_front_panel/tree/63caacfba0c60e0801e21b8a83e35e38ba6af034

[s-ino-346]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L345-L356
[s-ino-357]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L357-L362
[s-ino-366]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L366-L373
[s-ino-375]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L375
[s-ino-377]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L377-L378
[s-ino-loop]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L397-L407
[s-ino-rx]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L237-L244
[s-ino-req]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L203-L235
[s-ino-wire]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L184-L200
[s-ino-tok]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L140-L180
[s-ino-slice]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L282-L342
[s-ino-enc]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L38-L66
[s-ino-meas]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L248-L278
[s-ino-sim]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L381-L402
[s-gx-44]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/screen_gx.cpp#L44-L51
[s-gx-53]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/screen_gx.cpp#L53-L83
[s-gx-56]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/screen_gx.cpp#L56
[s-gx-64]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/screen_gx.cpp#L62-L82
[s-gx-70]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/screen_gx.cpp#L70-L75
[s-f-init]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L21-L41
[s-f-panel]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L114-L138
[s-f-set]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L141-L208
[s-f-sel]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L9
[s-f-sel302]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L300-L306
[s-f-input]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L628-L631
[s-f-smeter]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L481-L492
[s-f-drawall]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L762-L778
[s-fl-9]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields_list.h#L9-L15
[s-fl-21]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields_list.h#L21
[s-fl-wf]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields_list.h#L123
[s-wf-draw]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/waterfall.cpp#L93-L97
[s-q]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/queue.cpp#L31-L36
[s-tft]: https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/TFT_setup.h

[d-zh-7]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/zbitx.h#L7
[d-ino-397]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/zbitxfrontpanel.ino#L397
[d-ino-399]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/zbitxfrontpanel.ino#L399-L400
[d-ino-377]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/zbitxfrontpanel.ino#L377-L400
[d-ino-swr]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/zbitxfrontpanel.ino#L286-L292
[d-gx-read]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/screen_gx.cpp#L174-L206
[d-gx-cal]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/screen_gx.cpp#L44-L83
[d-f-at]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields.ino#L315-L336
[d-f-post]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields.ino#L571-L590
[d-f-menu]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields.ino#L360-L472
[d-f-dialog]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields.ino#L138-L151
[d-f-drawall]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields.ino#L1057-L1078
[d-fl-15]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields_list.h#L15-L24
[d-fl-29]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields_list.h#L29
[d-wf]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/waterfall.cpp#L112-L179
[d-readme]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/README.md
[d-build]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/build.sh#L3-L6
[d-plat]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/platform_local.txt#L1-L2
[d-readfirst]: https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/Read_First.txt

[v-ino-42]: https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx_front_panel_v2.ino#L42-L43
[v-ino-424]: https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx_front_panel_v2.ino#L424-L458
[v-ino-444]: https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx_front_panel_v2.ino#L444
[v-ino-493]: https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx_front_panel_v2.ino#L493
[v-ino-508]: https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx_front_panel_v2.ino#L508-L569
[v-ino-537]: https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx_front_panel_v2.ino#L537
[v-zh-saved]: https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx.h#L151-L162
[v-storage]: https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/storage.cpp
[v-tft]: https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/tft_ili9488.cpp#L729-L754

[p-remote]: https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/remote.c#L208
[p-ap]: https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/setup-ap.sh#L5
[p-poll]: https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/sbitx_gtk.c#L4074-L4164

[tft-h-inc]: https://github.com/Bodmer/TFT_eSPI/blob/6ae4c97c5a07bd64fce8993e9cfa95d9a81e7c81/TFT_eSPI.h#L59-L66
[tft-h-def]: https://github.com/Bodmer/TFT_eSPI/blob/6ae4c97c5a07bd64fce8993e9cfa95d9a81e7c81/Extensions/Touch.h#L38-L39
[tft-gettouch]: https://github.com/Bodmer/TFT_eSPI/blob/6ae4c97c5a07bd64fce8993e9cfa95d9a81e7c81/Extensions/Touch.cpp#L169-L194
[tft-conv]: https://github.com/Bodmer/TFT_eSPI/blob/6ae4c97c5a07bd64fce8993e9cfa95d9a81e7c81/Extensions/Touch.cpp#L201-L223
[tft-cal-wait]: https://github.com/Bodmer/TFT_eSPI/blob/6ae4c97c5a07bd64fce8993e9cfa95d9a81e7c81/Extensions/Touch.cpp#L262-L276
[tft-cal-out]: https://github.com/Bodmer/TFT_eSPI/blob/6ae4c97c5a07bd64fce8993e9cfa95d9a81e7c81/Extensions/Touch.cpp#L320-L326

[ap-cyw]: https://github.com/earlephilhower/arduino-pico/blob/6fc3165d3a6024d221ed8897cf2b6f52c613fb10/cores/rp2040/cyw43_wrappers.cpp#L116-L151
[ap-eeprom]: https://github.com/earlephilhower/arduino-pico/blob/0bcd162ad6c9173544cc37c6a1b63de19239e359/libraries/EEPROM/src/EEPROM.cpp#L37-L45
[ap-eeprom-commit]: https://github.com/earlephilhower/arduino-pico/blob/0bcd162ad6c9173544cc37c6a1b63de19239e359/libraries/EEPROM/src/EEPROM.cpp#L110-L126
[ap-b-pico]: https://github.com/earlephilhower/arduino-pico/blob/0bcd162ad6c9173544cc37c6a1b63de19239e359/boards.txt#L91
[ap-b-picow]: https://github.com/earlephilhower/arduino-pico/blob/0bcd162ad6c9173544cc37c6a1b63de19239e359/boards.txt#L314
[ap-b-460]: https://github.com/earlephilhower/arduino-pico/blob/6fc3165d3a6024d221ed8897cf2b6f52c613fb10/boards.txt#L90
