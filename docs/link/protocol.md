# Pi to front panel link (zBitx v1)

This page describes how the Raspberry Pi Zero 2 W talks to the RP2040 front panel on zBitx v1.
It supports the diagnosis in [the panel stall case](../troubleshooting/cases/2026-09-panel-stall.md).

Research date: 2026-09-25.

## Sources and versions

| Short name | Repository and commit | Role |
|---|---|---|
| Stock Pi | [afarhan/zbitx @ c671ac8](https://github.com/afarhan/zbitx/tree/c671ac81e244242cb5718cd5b2b63f1d136695e0) | Stock v1 Pi software, `sbitx v3.052`. Last code change is [51a7695](https://github.com/afarhan/zbitx/commit/51a76954f4942354b9047f814fc9dda614c00853). |
| Stock Pi (copy) | [afarhan/zbitxv2 @ d94fbc8](https://github.com/afarhan/zbitxv2/tree/d94fbc8393d5d8321cc6e5c488de7b52a0fded1a) | The same v1 code (3.052) before the v2 rewrite. `i2cbb.c` is identical to the stock Pi. |
| Stock panel | [afarhan/zbitxfrontpanel @ dc4a5a5](https://github.com/afarhan/zbitxfrontpanel/tree/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37) | Stock v1 panel firmware, banner `zBitx firmware v1.07d`. |
| v2 Pi | [afarhan/zbitxv2 @ 2726c57](https://github.com/afarhan/zbitxv2/tree/2726c574c531da99b862a0adf68c1ddba671e1e9) | v2 software. It uses WiFi, not I2C. |
| v2 panel | [afarhan/zbitxv2_front_panel @ 63caacf](https://github.com/afarhan/zbitxv2_front_panel/tree/63caacfba0c60e0801e21b8a83e35e38ba6af034) | v2 panel firmware, banner `zBitx firmware v4.00`. It uses WiFi. |
| drexjj Pi | [drexjj/zbitx @ 7d6e511](https://github.com/drexjj/zbitx/tree/7d6e51140503eb939b01cf1f2fb8855dd16f30e3) | Custom Pi software, `zbitx v5.11s`. |
| drexjj panel | [drexjj/zbitxfrontpanel @ ec075bf](https://github.com/drexjj/zbitxfrontpanel/tree/ec075bf36083e0f04a84617f647dee8491429408) | Custom panel firmware, version `5.13f`. |

### Which panel firmware is "stock v1"

- The stock v1 panel firmware is `afarhan/zbitxfrontpanel`. It uses I2C.
- `afarhan/zbitxv2_front_panel` is **not** v1 firmware. It joins the WiFi network `zbitx` and opens TCP port 8081 ([source](https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx_front_panel_v2.ino#L43), [source](https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx_front_panel_v2.ino#L493)).
- The v2 README says v1 used "I2C digital lines" and v2 uses WiFi ([README](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/README.md#L6)).
- In the v2 Pi software, the I2C `zbitx_poll()` is inside a comment block ([source](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/sbitx_gtk.c#L4074-L4075)).
- The root of `afarhan/zbitxv2` holds `zbitx_front_panel_v2.ino.uf2`. That file is the WiFi firmware. Do not flash it on a v1 system that runs I2C software.

Each firmware shows a different boot banner. The banner identifies the running firmware.

| Firmware | Boot banner text | Source |
|---|---|---|
| Stock v1 panel | `zBitx firmware v1.07d` then `Waiting for the zBitx to start...` | [zbitx_front_panel_sw.ino#L375](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L375) |
| drexjj panel | `Waiting for the zBitx to start...` only | [zbitxfrontpanel.ino#L397](https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/zbitxfrontpanel.ino#L397) |
| v2 panel | `zBitx firmware v4.00 ...` then `Waiting for the zbitx wifi...` | [zbitx_front_panel_v2.ino#L444](https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/zbitx_front_panel_v2.ino#L444) |

We extracted these strings from the three `.uf2` files in the repositories. The strings match the source.

### The case photo shows drexjj panel firmware

The screen layout in the [case photo](../troubleshooting/cases/img/2026-09-panel-stall.webp) is the drexjj layout.

- The photo shows AUDIO at x=240, directly left of the frequency. It shows AGC at the right end of row 2. It shows no SET button and no RIT button.
- drexjj puts AUDIO at x=240 and AGC at (432, 48). It moves SET and RIT into the MENU dialog ([fields_list.h#L14-L32](https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields_list.h#L14-L32)).
- Stock puts RIT at x=192, AUDIO at x=432 and SET at (432, 48) ([fields_list.h#L9-L21](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields_list.h#L9-L21)).
- No stock commit ever used the drexjj layout. drexjj introduced it in [847259e](https://github.com/drexjj/zbitxfrontpanel/commit/847259e6e509a9f096667e8d3c7461007544dc17) (v1.12, 2026-08-25).

The owner confirms the photo shows the drexjj phase. So it says nothing about whether the stock flash took. The stock boot banner settles that.

## Physical layer

The link is a single I2C bus. The Pi is the only master. The panel is a slave at 7-bit address `0x0A`.

### Pin table

| Signal | Pi Zero 2 W | Main board net | Main board J4 | Panel J3 | RP2040 (Pico) |
|---|---|---|---|---|---|
| SCL | BCM GPIO6, header pin 31, wiringPi 22 | `SCL_BB` | pin 4 | pin 4 `I2C_SCL` | GPIO7 (Pico pin 10), `Wire1` SCL |
| SDA | BCM GPIO13, header pin 33, wiringPi 23 | `SDA_BB` | pin 5 | pin 5 `I2C_SDA` | GPIO6 (Pico pin 9), `Wire1` SDA |
| Supply | – | `+9VA` | pin 6 | pin 6 `9V` | L78L05 to VSYS |
| Supply sense | – | `+9VA` through 1K/1K divider (R14, R8) | pin 8 | pin 8 `VBATT` | GPIO28 / ADC2 |
| Forward power | – | `VFWD` | pin 10 | pin 10 | GPIO27 / ADC1 |
| Reflected power | – | `VREF` | pin 9 | pin 9 | GPIO26 / ADC0 |
| Ground | – | GND | pins 1, 7 | pins 1, 7 | GND |

Evidence:

- Pi pins: the stock code defines `SDA 23` and `SCL 22` in wiringPi numbers ([si5351v2.c#L8-L9](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/si5351v2.c#L8-L9)). wiringPi 22 is BCM GPIO6. wiringPi 23 is BCM GPIO13.
- The schematic labels header pins 31 and 33 as `GPIO22` and `GPIO23`. These are wiringPi numbers, not BCM numbers. Nets `SCL_BB` and `SDA_BB` connect to them ([schematics_zbitx.pdf](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/schematics_zbitx.pdf), page 2, header RPI1 and connector J4).
- J4 pin 8 carries half of `+9VA` through R14 and R8 (same PDF, page 2, below J4).
- Panel pins: the firmware calls `Wire1.setSDA(6)`, `Wire1.setSCL(7)` and `Wire1.begin(0x0a)` ([zbitx_front_panel_sw.ino#L357-L362](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L357-L362)).
- The panel schematic shows `I2C_SDA` on GPIO6, `I2C_SCL` on GPIO7, and `VBATT` on GPIO28_ADC2 ([zbitx_front_panel_sch.pdf](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sch.pdf), page 1, U1 and J3).

### Pull-ups and other devices

- R1 and R2 (3.3K) pull `SDA_BB` and `SCL_BB` up to the 3.3 V rail (schematics_zbitx.pdf, page 4, next to U1 Si5351).
- The panel schematic shows no I2C pull-ups. The panel depends on the main board pull-ups.
- The bus is shared. The Si5351 clock generator (0x60) and the DS3231 RTC (0x68) sit on the same wires ([si5351v2.c#L52](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/si5351v2.c#L52), [ntputil.c#L21](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/ntputil.c#L21), schematic page 4).
- The stock Pi also reads address 0x08 for an SWR bridge that zBitx v1 does not have ([sbitx.c#L744](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L744)). drexjj removed this read ([sbitx_gtk.c#L6168-L6171](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L6168-L6171)).
- RD5 and RD6 (2.2K) on header pins 3 and 5 belong to a different bus (the Pi hardware I2C). Do not confuse them with `SDA_BB` and `SCL_BB`.

A stuck `SCL_BB` or `SDA_BB` line also stops frequency changes, because the Si5351 is on the same bus.

### Master implementation

| Pi software | I2C driver | Clock-stretch timeout |
|---|---|---|
| Stock v1 | User-space bit-bang with wiringPi (`i2cbb.c`) | **None** |
| drexjj (v5.05z and later) | Linux kernel, `/dev/i2c-3`, `I2C_RDWR` ioctl (`src/i2c.c`) | Kernel adapter timeout |

Stock bit-bang details:

- Each half bit is a busy loop of 400 iterations ([i2cbb.c#L37](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L37)). **Unverified:** the real SCL frequency. A logic analyzer measures it.
- The master waits for SCL high in four loops with no timeout ([i2cbb.c#L103](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L103), [#L125](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L125), [#L153](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L153), [#L176](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L176)). The source comments say "You should add timeout to this loop". If any device holds SCL low, the calling thread spins forever.
- On a lost arbitration the code prints `I2CBB connection lost:` and continues ([i2cbb.c#L76-L80](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L76-L80)).
- The block write length is a `uint8_t` ([i2cbb.c#L284-L285](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L284-L285)). A message longer than 255 bytes wraps silently.

drexjj kernel driver details:

- The Pi opens `/dev/i2c-3`. If the device node is missing, the program calls `abort()` ([src/i2c.c#L26-L37](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/i2c.c#L26-L37)).
- The change came in commit [3eaf018](https://github.com/drexjj/zbitx/commit/3eaf0183438d90e25fe59a4fcff66bdd130896a1) ("Converted to kernel level I2C").
- The repository contains no `config.txt` overlay that creates `/dev/i2c-3`. **Unverified:** the drexjj SD image adds an `i2c-gpio` overlay on GPIO13 and GPIO6. `grep -i i2c /boot/firmware/config.txt` on that image confirms it.

### Slave implementation

- The panel uses the arduino-pico `Wire1` slave on the RP2040 hardware I2C1 block.
- `Wire1.setClock(400000L)` has no effect on a slave. **Unverified:** this follows from the I2C standard. The master sets the clock.
- `on_receive()` runs in interrupt context. It copies bytes into a ring queue ([zbitx_front_panel_sw.ino#L238-L244](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L238-L244)).
- The queue holds 4000 entries ([zbitx.h#L116](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx.h#L116)). When it is full, new bytes are dropped ([queue.cpp#L31-L36](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/queue.cpp#L31-L36)).
- `on_request()` also runs in interrupt context. It answers one message per read ([zbitx_front_panel_sw.ino#L203-L235](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L203-L235)).
- **Unverified:** how arduino-pico handles a read that is longer than the data in the TX FIFO. The RP2040 slave stretches SCL while its TX FIFO is empty. Check `libraries/Wire/src/Wire.cpp` in the arduino-pico core version that built each `.uf2`.

## Protocol

### Pi to panel (I2C write)

Each update is one I2C block write to address `0x0A`.

- The "command" byte is `{`. The data is `LABEL VALUE}`. So the panel sees `{LABEL VALUE}`.
- The label ends at the first space. The value runs to the `}`.
- The panel ignores bytes outside `{` and `}`. A new `{` discards any unfinished message ([zbitx_front_panel_sw.ino#L140-L180](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L140-L180)).
- The panel ignores a label that it does not know ([fields.ino#L153-L157](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L153-L157)).
- For 1 second after a local user change, the panel ignores Pi updates to that field ([zbitx_front_panel_sw.ino#L154](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L154)).

Example messages (bytes after the address):

```text
{MODE FT8}
{FREQ 7074000}
{DRIVE 50}
{TX_PITCH 990}
{9 sbitx v3.052}          console line 9: the Pi version at start-up
{9 \nzBitx on http://192.0.2.10\n}
{IN_TX 0}
{WF <about 251 characters, each 32..127>}
{QSO 3|FT8|21074|...}     logbook row
```

The Pi builds field messages with `sprintf("%s %s}", label, value)` ([sbitx_gtk.c#L4111](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4111)).

Console text goes through a queue and uses the style number as the label ([sbitx_gtk.c#L4005-L4021](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4005-L4021)). Labels 5, 9 and 10 go to the console. Labels 6 and 7 go to the FT8 list ([fields.ino#L145-L148](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L145-L148)).

### Waterfall data

- The Pi sends the spectrum as field `WF`. The value is text: one character per bin ([sbitx_gtk.c#L4026-L4070](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4026-L4070)).
- Each character is `level + 32`. A space means "at or below the floor". `0x7F` means full scale.
- In receive, the loop makes about 251 bins. In transmit, it makes 250 bins from the modulation display.
- So the write is `{` plus about 255 bytes. That is at the 255-byte limit of the `uint8_t` length. **Unverified:** the exact count. Floating-point rounding decides it. A bus capture confirms it.
- The panel rescales the text to the waterfall width. It subtracts 32 from each character ([fields.ino#L181-L200](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L181-L200)).
- Each `WF` message draws one spectrum trace. It also scrolls the waterfall by one row ([waterfall.cpp#L101-L133](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/waterfall.cpp#L101-L133)).
- The same call paints the bandwidth strip (grey), the centre line (white) and the pitch line (green).
- In FT8, it also paints the TX pitch line (red).
- A tap on the waterfall toggles it. The panel then sends `WF OFF` or `WF ON` ([fields.ino#L241-L247](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L241-L247)). With `OFF`, the panel stops drawing the waterfall ([waterfall.cpp#L93-L97](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/waterfall.cpp#L93-L97)).
- The stock Pi then stops sending `WF` ([sbitx_gtk.c#L4145](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4145), [#L4175-L4178](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4175-L4178)). Both flags live in RAM only. A power cycle sets both back to `ON`.

### Panel to Pi (I2C read)

The Pi reads one message per poll with `i2cbb_read_rll()` ([i2cbb.c#L368-L386](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L368-L386)).

- The panel sends a length byte, then that many text bytes. It sends no terminator ([zbitx_front_panel_sw.ino#L184-L200](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L184-L200)).
- The panel picks the message in this order ([zbitx_front_panel_sw.ino#L203-L235](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L203-L235)):
  1. A raw command in `message_buffer`, for example `FT8 <message>` from the FT8 list ([ft8.cpp#L31](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/ft8.cpp#L31)).
  2. The first button with a pending change, as `LABEL VALUE`.
  3. The first other field with a pending change, as `LABEL VALUE`.
  4. If nothing is pending: `vbatt N\npower N\nvswr N\n`.
- The Pi puts the text on a command queue. The GTK timer drains the queue and runs each line as a command ([sbitx_gtk.c#L2755-L2761](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L2755-L2761), [#L4244-L4254](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4244-L4254)).

Example replies:

```text
0x0C "FREQ 7075000"
0x08 "MODE USB"
0x1B "vbatt 812\npower 0\nvswr 100\n"
```

Touch and encoder events do not travel as raw events. The panel applies them to a local field. It marks the field for sending. The Pi collects it on a later read.

### Poll cadence

- The stock Pi calls `zbitx_poll(0)` from the GTK timer `ui_tick()` ([sbitx_gtk.c#L4314-L4320](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4314-L4320)).
- The timer is set to 1 ms ([sbitx_gtk.c#L3838](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L3838)). The poll runs every 200 ticks in FT8 and every 50 ticks in CW ([sbitx_gtk.c#L4302-L4313](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4300-L4313)). **Unverified:** the real period. GTK timers slip under load.
- One poll does this, in order ([sbitx_gtk.c#L4096-L4187](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4096-L4187)):
  1. Write each field that changed since the last poll. Wait 10 ms after each one.
  2. Write any queued console lines.
  3. Write one `WF` line, unless the panel turned the waterfall off.
  4. Write `IN_TX 0` or `IN_TX 1`.
  5. Write logbook rows, if the panel asked for them.
  6. Read one reply from the panel.
- The poll runs on the GTK main thread. The input path and the waterfall both depend on that thread.

## Start-up handshake

The panel has no "Pi present" flag. It shows the banner until text arrives for console field `9`. Any field update then repaints the screen.

```mermaid
sequenceDiagram
    participant P as Panel (RP2040)
    participant M as Pi main thread
    Note over P: setup(): banner "Waiting for the zBitx to start..."
    Note over P: loop(): draw fields, read touch, read ADC (battery)
    Note over M: main(): load settings, start sound, web, hamlib
    M->>P: {9 sbitx v3.052}  (probe; ACK means panel present)
    M->>P: sbitx v3.052  (no braces; panel discards it)
    M->>P: {9 \nzBitx on http://<ip>\n}
    loop every field in active_layout
        M->>P: {LABEL VALUE}  then 10 ms delay
    end
    M->>P: {WF <spectrum>}
    M->>P: {IN_TX 0}
    P-->>M: first read: length + "vbatt ..." or a pending field
    Note over M: set SCHED_FIFO max priority, enter gtk_main()
    loop GTK timer, every 50-200 ticks
        M->>P: changed fields, console, {WF ...}, {IN_TX n}
        P-->>M: one reply
    end
```

Numbered sequence, stock Pi ([sbitx_gtk.c#L5299-L5309](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L5299-L5309)):

1. `zbitx_init()` writes `{9 sbitx v3.052}`. An ACK sets `zbitx_available = 1` and prints `zBitx front panel detected` ([sbitx_gtk.c#L4190-L4221](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4190-L4221)).
2. A NACK leaves `zbitx_available = 0`. The stock Pi never tries again. The panel stays on the banner.
3. `zbitx_poll(1)` sends every field, then one `WF` line and `IN_TX`, then does the **first read**.
4. The main thread sets itself to `SCHED_FIFO` at maximum priority ([sbitx_gtk.c#L5306-L5307](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L5306-L5307)).
5. `gtk_main()` starts. From here, only the GTK timer calls `zbitx_poll(0)`.

The start-up dump is the only full send. The periodic poll sends only changed fields. So a screen full of Pi values proves step 3 ran. It does not prove that step 5 runs.

### Values that show the dump arrived

The panel boots with its own defaults. A value that differs from the default came from the Pi.

| Field | Panel default (stock) | Value in the case photo |
|---|---|---|
| MODE | USB | FT8 |
| DRIVE | 100 | 50 |
| IF | 40 | 51 |
| AUDIO | 95 | 60 |
| FREQ | 14074000 | 7074000 |
| SPAN | 25K | 6K |
| BW | 2200 | 4000 |
| AGC | MED | FAST |
| TX_PITCH (shown as PITCH) | 2200 | 990 |

Defaults: [fields_list.h](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields_list.h#L9-L103). drexjj uses the same defaults for these fields ([fields_list.h](https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields_list.h#L14-L29)). The F1 to F5 labels (CQ, Call, Reply) are panel defaults. They prove nothing.

### What the photo shows in the waterfall area

The top grey block is the output of **one** `waterfall_update()` call.

- The grey strip starts at the white centre line. It runs to the right edge. That matches BW 4000 at SPAN 6K.
- The red line sits about 40 px right of centre. At SPAN 6K that is about 1000 Hz. It matches TX_PITCH 990.
- So at least one `WF` message arrived **after** SPAN, BW and TX_PITCH. That order matches step 3.
- The yellow trace lies flat on the bottom row. So every bin in that `WF` line was near zero (spaces).
- **Unverified:** a flat trace fits a spectrum sent before the DSP had real data. It also fits a stream of floor-level lines. The check in the failure table separates these.
- The lower grey block sits below the 144-row waterfall bitmap. **Unverified:** what draws it.

## Battery voltage is measured on the panel

This fact decides a lot. The battery readout does **not** prove that the link works.

- The panel reads ADC2 (GPIO28) every 50 ms into a global `vbatt` ([zbitx_front_panel_sw.ino#L248-L278](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L248-L278)).
- `smeter_draw()` prints that global as `+8.0v` ([fields.ino#L481-L492](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L481-L492)).
- `field_draw_all()` calls `smeter_draw()` on every loop pass ([fields.ino#L776-L777](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L776-L777)).
- The panel sends `vbatt` to the Pi, not the other way ([zbitx_front_panel_sw.ino#L233](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L233)). The Pi stores it in `#batt` ([sbitx_gtk.c#L5000-L5001](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L5000-L5001)).
- The Pi can echo `VBATT` back. The panel stores it in a hidden field. The display does not use that field ([fields_list.h#L135](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields_list.h#L135)).

A changing battery readout proves one thing only: the panel main loop runs. The S-meter bars also come from the loop, but their value comes from the Pi.

## Input path depends on both ends

- Touch goes through `screen_read()` and `field_select()`. Both are local ([zbitx_front_panel_sw.ino#L320-L341](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L320-L341)).
- A tap on a selection field (for example MODE) changes its value **on the panel at once**. It does not wait for the Pi ([fields.ino#L309-L338](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L309-L338)).
- The encoder acts only on the selected field. With no field selected, rotation and push do nothing ([fields.ino#L628-L631](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L628-L631)). drexjj has the same rule.
- So "encoder ignored" follows from "touch ignored". It is not a separate fault until a field is selected.
- Touch calibration lives in emulated EEPROM, bytes 0 to 11 ([screen_gx.cpp#L44-L83](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/screen_gx.cpp#L44-L83)). drexjj uses the same bytes.
- If the screen is touched at power-on, the firmware runs the calibration and writes new values ([screen_gx.cpp#L64-L76](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/screen_gx.cpp#L64-L76)).
- TFT_eSPI `getTouch()` returns "no touch" when the calibrated point is off the screen. So a bad calibration looks like a dead touchscreen.

## Failure modes

| # | Failure | Observable symptom | How to confirm |
|---|---|---|---|
| F1 | Pi main thread stops after the start-up dump. Example: it spins in an `i2cbb` clock-stretch loop because SCL stays low. | All Pi values paint. One `WF` line paints. Nothing changes after that. Pi desktop GUI freezes or never opens. Frequency changes from the web UI do not reach the panel. Band audio continues on its own thread. | On the Pi: `top -H -p $(pgrep -f sbitx)`. One thread at about 100 % CPU confirms a spin. `pinctrl get 6,13` (or `raspi-gpio get 6,13`) shows level 0 on GPIO6 if SCL is stuck. |
| F2 | Pi process exits after the start-up dump. | Same panel picture as F1. No `sbitx` or `zbitx` process. The start script terminal shows `Press enter to continue...`. | `pgrep -a 'sbitx|zbitx'`. Read the terminal output for a crash message. |
| F3 | Two bus masters. A second `sbitx`/`zbitx` process, or a kernel `i2c-gpio` driver, uses GPIO6 and GPIO13. | Random NACKs. `I2CBB connection lost:` and `Retrying I2C` lines on stdout. Partial updates. | `pgrep -a 'sbitx|zbitx'`. `ls /dev/i2c*`. `grep -i i2c /boot/firmware/config.txt /boot/config.txt`. |
| F4 | Touch calibration in panel flash is bad. It survives a `.uf2` flash. | Taps do nothing, even local ones. The encoder does nothing, because no field is selected. Pi data still flows. | Tap MODE. If it does not cycle on the panel, touch is dead locally. Power on with a finger on the screen to recalibrate. |
| F5 | Waterfall turned off by a tap on the waterfall. | Waterfall stops. Other fields update. Touch works. | Tap the waterfall area once. The state resets at power-on. |
| F6 | SDA held low by a stuck slave. | Stock master reads every ACK as good. Writes "succeed" but carry nothing. `I2CBB connection lost:i2c_start_cond` repeats on stdout. Frequency changes stop. | `pinctrl get 13` shows level 0 with no traffic. A logic analyzer shows SDA low between frames. |
| F7 | `WF` or other message longer than 255 bytes. | The length wraps. That message is lost or cut. Other fields still update. | Capture the bus. Count the bytes in the `{WF` frame. |
| F8 | Field value longer than 127 characters. | `field_set()` copies it into a 128-byte buffer with no check ([fields.ino#L201-L206](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/fields.ino#L201-L206)). Memory next to the field is overwritten. Behaviour is undefined. A long value can crash the panel. | Look for long values in `~/sbitx/data/user_settings.ini`. Capture the bus during the start-up dump. |
| F9 | Wrong firmware pair (see below). | Depends on the pair. | Read the boot banner. On drexjj, read Menu, SETUP. |
| F10 | v2 WiFi pair (v2 Pi + v2 panel) stalls on a busy WiFi channel. | Values paint, then the stream stalls. The v2 notes describe this ([WIFI_CHANNEL_FIX.txt](https://github.com/afarhan/zbitxv2_front_panel/blob/63caacfba0c60e0801e21b8a83e35e38ba6af034/WIFI_CHANNEL_FIX.txt)). | Banner reads `Waiting for the zbitx wifi...`. `iw dev wlan0 info` and `iw dev uap0 info` show different channels. |

### Fit to the case symptom

The case shows: values paint, one waterfall line, then no waterfall, no touch, no encoder. Battery updates.

- F1 and F2 explain the waterfall and the lack of radio response with one cause. They do not explain a tap that changes nothing **locally**.
- F4 explains touch and encoder with one cause. It does not explain the frozen waterfall.
- F1 or F2 plus F4 explains everything. So does F1 or F2 alone, if "touch ignored" means "the radio does not react".
- The first test is therefore: does a tap on MODE change MODE on the panel?

## Mismatched pairs

The message format is the same in all four I2C builds. Unknown labels are ignored on both ends. So a mixed pair mostly degrades, it does not wedge.

| Pi | Panel | Expected result |
|---|---|---|
| Stock v1 | Stock v1 | Designed pair. |
| drexjj | drexjj | Designed pair. drexjj requires panel 5.11f or later for current software ([release_notes.md](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/release_notes.md#L10-L17)). |
| Stock v1 | drexjj | Works at protocol level. The Pi ignores drexjj-only commands such as `WIFI ...`. drexjj layout shows. |
| drexjj | Stock v1 | Works at protocol level. The panel ignores `PIVERSION`, `MACROLIST` and WiFi labels. **Unverified:** the 255-byte read (see below) may drop panel events. |
| Any I2C Pi | v2 panel | No link. The panel waits for WiFi forever. |
| v2 Pi | Any I2C panel | No link. The v2 Pi never polls I2C. The panel stays on the banner. |

## drexjj compared with stock

### Pi side

| Topic | Stock v1 | drexjj |
|---|---|---|
| Driver | Bit-bang, no timeouts ([i2cbb.c](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c)) | Kernel `/dev/i2c-3` ([src/i2c.c](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/i2c.c)) |
| Panel read | Reads the length byte, then that many bytes | Always reads 255 bytes, then uses the length byte ([src/i2c.c#L138-L165](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/i2c.c#L138-L165)) |
| Panel detection | Once at start-up | Retries about once a second, then sends a full dump ([sbitx_gtk.c#L7068-L7087](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L7069-L7087)) |
| Poll period | 50 to 200 GTK ticks by mode | 100 ticks, 500 in CW ([sbitx_gtk.c#L7062-L7066](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L7062-L7066)) |
| CW transmit | Sends fields and `WF` | Skips fields and `WF` ([sbitx_gtk.c#L6594](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L6594)) |
| Extra start-up messages | – | `PIVERSION ...`, `MACROLIST ...` ([sbitx_gtk.c#L6802](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L6802), [#L6819](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L6819)) |
| Extra periodic messages | – | `SMETER n`, WiFi panel status |
| Reply parsing | Whole reply to one command | Parses `vbatt`/`power`/`vswr`, splits other replies on newlines ([sbitx_gtk.c#L6670-L6750](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L6670-L6750)) |
| Read failure log | None | `zbitx_poll: i2cbb_read_rll(0xa) FAILED (-1)` every 2 s ([sbitx_gtk.c#L6757](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L6757)) |
| Detection failure log | None | `zbitx_init: front panel NOT detected ...` ([sbitx_gtk.c#L6832](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L6832)) |
| GTK thread priority | `SCHED_FIFO` max | `SCHED_OTHER` ([sbitx_gtk.c#L8711-L8721](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L8711-L8721)) |
| Binary name | `sbitx` | `zbitx` ([start.sh](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/start.sh)) |

**Unverified:** the 255-byte read. The panel prepares one message per read request. A read longer than that message may trigger `on_request()` again. Each extra call clears one pending field. The Pi then drops it. A bus capture of a drexjj read against a stock panel confirms this.

### Panel side

The I2C code is the same in both firmwares: pins, address, tokenizer, queue size, reply format ([stock](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L184-L244), [drexjj](https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/zbitxfrontpanel.ino#L189-L256)). The differences:

| Topic | Stock v1.07d | drexjj 5.13f |
|---|---|---|
| `message_buffer` | 100 bytes | 200 bytes ([zbitxfrontpanel.ino#L36-L41](https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/zbitxfrontpanel.ino#L36-L41)) |
| Telemetry globals | Plain `int` | `volatile int`, VSWR divide-by-zero guard |
| Touch read | One `getTouch()` | Four samples within 8 px ([screen_gx.cpp#L174-L205](https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/screen_gx.cpp#L174-L205)) |
| Layout | See [Which panel firmware](#the-case-photo-shows-drexjj-panel-firmware) | AUDIO, AGC moved. SET, RIT in MENU. |
| Field value copy | No length check | No length check ([fields.ino#L293-L297](https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/fields.ino#L293-L297)) |
| Board target | **Unverified** | `rp2040:rp2040:rpipicow` ([build.sh](https://github.com/drexjj/zbitxfrontpanel/blob/ec075bf36083e0f04a84617f647dee8491429408/build.sh)) |

## What survives a reflash

### Panel flash

- A `.uf2` copy writes only the program blocks. The stock `.uf2` covers 0x10000000 to about 0x1006E100. The drexjj `.uf2` covers 0x10000000 to about 0x1002DF00 (we parsed both files).
- The emulated EEPROM holds the touch calibration. **Unverified:** arduino-pico places it in the last flash sector, outside both images. Then it survives every `.uf2` flash. Confirm it with a `flash_nuke.uf2` test: calibration must run again after it.
- No other link setting lives in panel flash in either firmware. We found no other `EEPROM` or LittleFS use in the source.
- A failed flash leaves the old firmware in place. The boot banner shows which firmware runs.

### Pi SD card

In the [open case](../troubleshooting/cases/2026-09-panel-stall.md), the factory card never had drexjj on it, so none of this applies there. If the owner re-imaged the card, nothing survives. If the owner only swapped the software folder, these can survive:

- `~/sbitx/data/user_settings.ini`, `hw_settings.ini` and `sbitx.db`. The drexjj guide tells users to copy these between installs ([README.md](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/README.md)).
- A kernel `i2c-gpio` overlay in `config.txt` from the drexjj image. **Unverified:** the image has one. It would give the kernel a claim on GPIO6 and GPIO13.
- A second autostart entry or binary (`zbitx` next to `sbitx`).

A setting file alone cannot stop the stock poll. **Unverified:** a value longer than 127 characters in `user_settings.ini` could corrupt panel memory (F8).

## How to observe the link

### Pi logs

Run the software from a terminal, so stdout is visible: `cd ~/sbitx && ./sbitx` (stock) or `./zbitx` (drexjj).

| Message | Meaning | Source |
|---|---|---|
| `zBitx front panel detected` | The probe got an ACK. | [sbitx_gtk.c#L4198](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4198) |
| `Retrying I2C n` / `Sucess on n` | A field write got a NACK. | [sbitx_gtk.c#L4115-L4121](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4115-L4121) |
| `i2cbb: address failed a, cmd 7b, length...` | The panel did not ACK its address. | [i2cbb.c#L322-L325](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L322-L325) |
| `I2CBB connection lost:<where>` | SDA was low when the master released it. | [i2cbb.c#L76-L80](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L76-L80) |
| `i2c busy` | Two threads used the bit-bang bus at once. | [i2cbb.c#L287-L292](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L287-L292) |
| `FT8 processing from zbitx` | A reply from the panel reached the Pi. | [sbitx_gtk.c#L4173](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4173) |

No message at all after start-up means one of two things. The link is healthy, or the main thread is stuck. `top -H` separates the two.

### Pi process and pin state

1. `pgrep -a 'sbitx|zbitx'` lists every radio process. Exactly one must run.
2. `top -H -p <pid>` shows per-thread CPU. A thread pinned near 100 % means a spin.
3. `pinctrl get 6,13` (Bookworm) or `raspi-gpio get 6,13` (older) shows the line levels. Both must read 1 when the bus is idle.
4. `ls /dev/i2c*` and `grep -i i2c /boot/firmware/config.txt /boot/config.txt` show any kernel claim on the bus.
5. **Unverified:** `gdb -p <pid>` then `thread apply all bt` names the function where the main thread waits.

### i2cdetect

- Stock software drives the bus from user space. No `/dev/i2c-N` node covers GPIO6 and GPIO13. So `i2cdetect` cannot see this bus on a stock image.
- On a drexjj image, `sudo i2cdetect -y 3` lists 0x0A, 0x60 and 0x68 on a healthy bus. **Unverified:** that bus 3 maps to GPIO13 and GPIO6.
- Stop the radio software first. Two masters on one bus cause errors.

### Logic analyzer

- Clip to J4 pin 4 (SCL), J4 pin 5 (SDA) and J4 pin 1 (GND) on the main board. The Pi header pins 31 and 33 carry the same nets.
- Use an I2C decoder with the address shown as 7-bit.
- A healthy start-up capture shows writes to 0x0A that begin with `7B` (`{`). The text ends with `7D` (`}`).
- Then one `{WF ` frame of about 256 bytes, then `{IN_TX 0}`, then a read from 0x0A.
- The read returns a length byte and that many ASCII bytes. The master NACKs the last byte.
- After start-up, the same pattern repeats every poll. In FT8, expect a `{WF` frame several times a second.
- A stuck bus shows SCL or SDA low with no clock edges.

### Panel USB serial

- The panel USB port is the port marked CAT. Holding the knob at power-on puts the Pico in upload mode on that port ([zbitxv2 README.md#L44-L45](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/README.md#L44-L45), [zbitx_front_panel_sw.ino#L377-L378](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L377-L378)).
- The firmware opens USB serial at 115200 baud ([zbitx_front_panel_sw.ino#L346](https://github.com/afarhan/zbitxfrontpanel/blob/dc4a5a5dcf9673c0fd16e99d86a2e60413db8c37/zbitx_front_panel_sw.ino#L346)).
- It prints very little: `#Calibrating the screen`, `#... field not found`, `#Wire sending[...] is too long`. It does not trace I2C traffic.
- **Unverified:** that the CAT port shows the Pico serial port in normal operation. A drexjj "USB mode" can change the port role.

## Open questions

1. Which banner shows at power-on now: `zBitx firmware v1.07d` or only `Waiting for the zBitx to start...`?
2. Does a tap on MODE change MODE on the panel itself?
3. Does a frequency change from the web interface or the Pi desktop reach the panel?
4. Is any Pi thread at 100 % CPU after the panel stalls? Are GPIO6 and GPIO13 high at idle?
5. What does arduino-pico `Wire` do on a read longer than the prepared reply? Check the core versions that built each `.uf2`.
6. Where does arduino-pico put the emulated EEPROM sector for the `rpipicow` target? Does a `.uf2` flash ever erase it?
7. Does the drexjj SD image add an `i2c-gpio` overlay or an RTC overlay on GPIO6 and GPIO13?
8. What is the real SCL frequency of the stock bit-bang master?
