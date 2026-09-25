# zBitx Pi software: boot, state, install, restore, diagnostics

This page covers the Raspberry Pi Zero 2 W side of a zBitx v1.
It compares the stock software with the drexjj software.
The Pi to panel wire protocol is out of scope here. See [docs/link/](../link/).

## Sources

| Short name | Repo | Commit |
|---|---|---|
| Stock v1 | [afarhan/zbitx](https://github.com/afarhan/zbitx/tree/c671ac81e244242cb5718cd5b2b63f1d136695e0) | `c671ac81e244242cb5718cd5b2b63f1d136695e0` (2025-07-05) |
| Stock v2 | [afarhan/zbitxv2](https://github.com/afarhan/zbitxv2/tree/2726c574c531da99b862a0adf68c1ddba671e1e9) | `2726c574c531da99b862a0adf68c1ddba671e1e9` (2026-04-27) |
| drexjj | [drexjj/zbitx](https://github.com/drexjj/zbitx/tree/7d6e51140503eb939b01cf1f2fb8855dd16f30e3) | `7d6e51140503eb939b01cf1f2fb8855dd16f30e3` (2026-09-20) |
| drexjj image | [Release v5.11s](https://github.com/drexjj/zbitx/releases/tag/v5.11s) | Release notes, not in the repo |

## Which software is "stock" for a v1 radio

The stock software for zBitx **v1** is `afarhan/zbitx`. It is not `afarhan/zbitxv2`.

- The stock v1 `update` script points `~/sbitx` at `afarhan/zbitx` ([`update` L8-L11](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/update#L8-L11)).
- The v2 README says v1 talks to the panel over I2C. It says v2 talks to the panel over WiFi ([`README.md` L1-L10](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/README.md#L1-L10)).
- In v2, the I2C panel poll is inside a comment block ([`sbitx_gtk.c` L4074-L4164](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/sbitx_gtk.c#L4074-L4164)). v2 never talks to a v1 panel over I2C.
- The v2 `update` script repoints `~/sbitx` at `afarhan/zbitxv2` ([`update` L8-L11](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/update#L8-L11)). Running it on a v1 radio converts it to v2 software.
- The stock v1 version string is `sbitx v3.052` ([`sdr_ui.h` L70](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sdr_ui.h#L70)).
- The last stock v1 commits (2025-07-05) change the Pi code and the panel UF2 together ([51a7695](https://github.com/afarhan/zbitx/commit/51a76954f4942354b9047f814fc9dda614c00853), [c671ac8](https://github.com/afarhan/zbitx/commit/c671ac81e244242cb5718cd5b2b63f1d136695e0)). A factory card from early 2025 can be older than the repo head.

The drexjj repo is a fork of `afarhan/zbitx`. Its history contains the stock commits up to [b9cabb4](https://github.com/drexjj/zbitx/commit/b9cabb446b78e02116a4310abdf1b52aa0d41f47) (2025-03-31).

## Boot chain

### Stock v1

No repo file defines the autostart.
The repo has no systemd unit, no `rc.local`, and no LXDE autostart file.

What the repo does show:

- The launcher script kills audio users, then runs `./sbitx` from `~/sbitx` ([`start.sh` L1-L3](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/start.sh#L1-L3)). It uses a relative `cd sbitx`, so it must start in `/home/pi`.
- The desktop entry runs `/home/pi/sbitx/sbitx` in a terminal ([`sBitx.desktop` L6-L8](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sBitx.desktop#L6-L8)).
- The program is a GTK app. It calls `gtk_init` and needs an X display ([`sbitx_gtk.c` L4420](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4420)).
- User `pi`, home `/home/pi`, working directory `/home/pi/sbitx`. Paths are hard-coded to `/home/pi/sbitx` and `$HOME/sbitx` (see [Persistent state](#persistent-state-on-the-sd-card)).
- The stock OS is Raspbian Buster. The v2 AP script targets "Raspberry Pi Zero 2W (Raspbian Buster)" ([`setup-ap.sh` L2](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/setup-ap.sh#L2)). The stock install notes edit `/boot/config.txt`, the Buster path ([`install.txt` L93-L111](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/install.txt#L93-L111)).

**Unverified:** The stock image autostarts `start.sh` or `sbitx` from the LXDE desktop session as user `pi`. Confirm on the card:

```bash
ls ~/.config/autostart/ /etc/xdg/autostart/ 2>/dev/null
cat ~/.config/lxsession/LXDE-pi/autostart /etc/xdg/lxsession/LXDE-pi/autostart 2>/dev/null
systemctl list-unit-files | grep -iE 'bitx|zbitx'
crontab -l; sudo cat /etc/rc.local
```

### drexjj

- `make` builds a binary named `zbitx`, not `sbitx` ([`Makefile` L73-L74](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/Makefile#L73-L74)).
- The default build is a headless daemon (`-DJJ_HEADLESS_DAEMON=1`). It does not open a GTK window ([`Makefile` L73](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/Makefile#L73), [`src/sbitx_gtk.c` L8572-L8576](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L8572-L8576)).
- `make` runs `sudo setcap cap_sys_nice,cap_sys_time` on the binary ([`Makefile` L84](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/Makefile#L84)).
- The launcher runs `./zbitx` from `/home/pi/sbitx` ([`start.sh` L2-L5](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/start.sh#L2-L5)). The desktop entry calls that script ([`zBitx.desktop` L5-L6](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/zBitx.desktop#L5-L6)).
- The code expects "headless/service launches" where `HOME` can be unset ([`src/sbitx_gtk.c` L1710-L1722](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L1710-L1722)).
- The single-instance guard is disabled ([`src/sbitx_gtk.c` L8566](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L8566)). Two copies can run at once.
- The image is Debian Bookworm, 32-bit ([release v5.11s](https://github.com/drexjj/zbitx/releases/tag/v5.11s)).

**Unverified:** The drexjj image starts `zbitx` from a systemd service. The repo has no unit file. Confirm with `systemctl list-unit-files | grep -iE 'bitx|zbitx'` on a drexjj card.

## config.txt requirements

Buster uses `/boot/config.txt`. Bookworm uses `/boot/firmware/config.txt`.

### Stock v1

The repo ships a reference `config.txt`. These lines matter ([`config.txt` L46-L47, L57, L67-L75](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/config.txt#L46-L75)):

```ini
dtparam=i2c_arm=on
dtparam=i2s=on
#dtparam=spi=on
#dtparam=audio=on

# sbitx related additions
gpio=4,5,9,10,11,17,22,27=ip,pu
gpio=24,23=op,pu
dtoverlay=audioinjector-wm8731-audio
dtoverlay=i2s-mmap
avoid_warnings=1
```

What each line does on a zBitx v1. Pin names come from [main-board.md](../hardware/main-board.md).

| Line | Effect |
|---|---|
| `dtparam=i2c_arm=on` | Enables hardware I2C1 (BCM 2/3). The WM8731 codec uses it. The panel does **not** use it. |
| `dtparam=i2s=on`, `dtoverlay=i2s-mmap` | Enables I2S audio to the WM8731. |
| `dtoverlay=audioinjector-wm8731-audio` | Loads the WM8731 driver. The code expects it as ALSA card 0 (`hw:0`) ([`sbitx.c` L985-L986](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L985-L986)). |
| `#dtparam=audio=on` | Keeps the Pi built-in audio off, so the codec stays card 0. |
| `gpio=4,5,...=ip,pu` | BCM 4 (PTT) and BCM 5 (dash) become inputs with pull-ups at boot. |
| `gpio=24,23=op,pu` | BCM 23 (TX_LINE) and BCM 24 (R-A) become outputs at boot. |

The panel link pins (BCM 6 and BCM 13) have no `config.txt` line.
No overlay or kernel driver may claim them. The software drives them with wiringPi.

The software also needs the ALSA loopback module ([`install.txt` L78-L89](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/install.txt#L78-L89)):

```bash
sudo modprobe snd-aloop enable=1,1,1 index=1,2,3
```

The code opens `plughw:1,0` and `plughw:2,1` for loopback ([`sbitx_sound.c` L1053](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_sound.c#L1053), [L1080](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_sound.c#L1080)).

### drexjj

The drexjj code drives the panel bus through the **kernel**, not wiringPi.
It opens `/dev/i2c-3` and calls `abort()` if the device is missing ([`src/i2c.c` L22-L36](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/i2c.c#L22-L36)).
The Makefile builds `src/i2c.c` in place of the stock `i2cbb.c` ([`Makefile` L22](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/Makefile#L22)).

No file in the drexjj repo creates `/dev/i2c-3`.
The drexjj image must add a device-tree overlay for it.

**Unverified:** The drexjj image has an `i2c-gpio` overlay on bus 3 that claims BCM 6 and BCM 13. Confirm with:

```bash
grep -nE 'i2c|gpio|spi|dtoverlay|dtparam|arm_freq|over_voltage|force_turbo|core_freq' \
  /boot/firmware/config.txt /boot/config.txt 2>/dev/null
ls -l /dev/i2c-*
```

Why this matters for the fault:

- With that overlay, a kernel driver owns the same two pins that stock code bit-bangs.
- The kernel `i2c-gpio` overlay defaults to BCM 23 (SDA) and BCM 24 (SCL) when no pins are given. On a zBitx those are TX_LINE and R-A.
- An `arm_freq`, `force_turbo` or governor change alters the stock bit-bang clock. The stock delay is a CPU spin loop of 400 counts, not a timed wait ([`i2cbb.c` L37](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L37), [L92-L96](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L92-L96)).
- `dtparam=spi=on` claims BCM 7 to 11. BCM 7 and BCM 8 drive LPF lines R-D and R-C.

All four points apply only if stock software runs on a card that once held the drexjj image.

## Runtime structure and logs

### Stock v1

Start order in `main()` ([`sbitx_gtk.c` L5296-L5309](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L5296-L5309)):

1. `setup()` reads `hw_settings.ini`, plans FFTs, programs the Si5351, and starts the audio thread.
2. `rtc_read()` reads the DS3231 on the bit-bang bus ([`ntputil.c` L80-L91](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/ntputil.c#L80-L91)).
3. `zbitx_init()` writes the version string to the panel at 0x0A. Success sets `zbitx_available` ([`sbitx_gtk.c` L4190-L4200](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4190-L4200)).
4. `zbitx_poll(1)` sends every field value once.
5. The main thread raises itself to `SCHED_FIFO` at maximum priority, then enters `gtk_main()`.

Who drives the panel link:

- The **GTK main thread** drives it. There is no separate I2C thread.
- `hw_init()` registers `ui_tick` as a 1 ms GTK timeout ([`sbitx_gtk.c` L3838](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L3838)).
- `ui_tick` calls `zbitx_poll(0)` every 50, 100 or 200 ticks, by mode ([`sbitx_gtk.c` L4319-L4320](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4319-L4320)).
- The panel probe runs **once**. If it fails, stock code never retries. drexjj added a retry for this reason ([`src/sbitx_gtk.c` L7068-L7088](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L7068-L7088)).
- The bit-bang code waits in an unbounded loop while SCL reads low (clock stretching) ([`i2cbb.c` L101-L104](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L101-L104)).

**Unverified:** If SCL_BB stays low, the stock GTK main thread spins forever at real-time priority. The audio thread keeps running, so band noise continues. This matches the symptom. Confirm with the SCL level check and `top -H` in [Diagnostics](#pi-side-diagnostics).

Logging: the stock code writes to stdout with `printf`. It has no log file and no syslog output.
If it starts from a desktop entry with `Terminal=true`, the output goes to that terminal window on the Pi screen.

Messages that matter for the panel link:

| Message | Meaning | Source |
|---|---|---|
| `hw version: 4` | `hw=4` read from `hw_settings.ini`. Healthy. | [`sbitx.c` L1409](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L1409) |
| `hw version: 0` or `1` | `hw=` line missing. The code probed 0x08 and guessed an sBitx model. | [`sbitx.c` L1401-L1409](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L1401-L1409) |
| `RTC not detected` | DS3231 read failed on the bit-bang bus | [`ntputil.c` L90](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/ntputil.c#L90) |
| `zBitx front panel detected` | Panel ACKed the first write | [`sbitx_gtk.c` L4198](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4198) |
| `Retrying I2C n` / `Sucess on n` | A field write to the panel failed, then retried | [`sbitx_gtk.c` L4110-L4122](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L4110-L4122) |
| `I2CBB connection lost:<where>` | SDA read low when it should be high | [`i2cbb.c` L76-L79](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L76-L79) |
| `Repeating I2C #n` | Si5351 write failed. This loop retries forever. | [`si5351v2.c` L60-L64](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/si5351v2.c#L60-L64) |
| `Generating Wisdom File...` | No FFTW wisdom. First start is slow. | [`sbitx.c` L174](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L174) |

### drexjj

- The GTK/UI tick still drives the panel, on its own period ([`src/sbitx_gtk.c` L7052-L7095](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L7052-L7095)).
- It logs a read failure to stderr: `zbitx_poll: i2cbb_read_rll(0xa) FAILED (-1)` ([`src/sbitx_gtk.c` L6757](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L6757)).
- It logs `Failed to open I2C 3` and aborts when `/dev/i2c-3` is missing ([`src/i2c.c` L33-L34](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/i2c.c#L33-L34)).
- On reconnect it prints `zBitx front panel (re)connected -- syncing all fields` ([`src/sbitx_gtk.c` L7082](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L7082)).

## Persistent state on the SD card

All paths are under `/home/pi/sbitx/` unless noted.

| File | Stock reads | Stock writes | drexjj touches | Survives a stock reinstall into `~/sbitx`? |
|---|---|---|---|---|
| `data/hw_settings.ini` | Yes, at start ([`sbitx.c` L1037-L1047](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L1037-L1047)) | After band power calibration ([`sbitx.c` L1117-L1138](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L1117-L1138)) | Reads and rewrites ([`src/sbitx.c` L2469-L2493](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx.c#L2469-L2493)) | Yes. Git ignores it in both repos. |
| `data/user_settings.ini` | Yes, at start ([`sbitx_gtk.c` L5251-L5261](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L5251-L5261)) | Every 30 s when changed ([`sbitx_gtk.c` L1428-L1470](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L1428-L1470)) | Rewrites every 5 s when changed ([`src/sbitx_gtk.c` L1707](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx_gtk.c#L1707)) | Yes. Git ignores it. |
| `data/sbitx.db` | Logbook, SQLite ([`logbook.c` L270](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/logbook.c#L270)) | Yes | Same schema. `create_db.sql` is identical. | Yes |
| `data/result_rows.txt` | Logbook query output | Yes ([`logbook.c` L86](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/logbook.c#L86)) | Yes | Yes |
| `data/sbitx_wisdom.wis`, `data/sbitx_wisdom_f.wis` | FFTW plans, hard-coded `/home/pi` path ([`sbitx.c` L66](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L66), [`fft_filter.c` L21](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/fft_filter.c#L21)) | Yes | Uses CPU-specific names ([`src/sbitx.c` L255-L285](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx.c#L255-L285)) | Yes. Not tracked in git under `data/`. Stock regenerates them when missing. |
| `web/*.mc` | Macros ([`macros.c` L42-L44](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/macros.c#L42-L44)) | No | drexjj ignores them in git | Yes |
| `web/grids.txt`, `main_ui.ini`, `sbitx.conf`, `sbitx.rc` | Relative paths in the working directory ([`hist_disp.c` L36](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/hist_disp.c#L36), [`sbitx_gtk.c` L2438](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx_gtk.c#L2438)) | Yes | Yes | Yes |
| `zbitx` (drexjj binary) | No | No | Built by drexjj `make` | **Yes.** Stock `.gitignore` does not list it. |
| `/etc/hostname`, `/etc/hosts` | OS | Stock install sets hostname `sbitx` ([`hostname` L1](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/hostname#L1), [`install.txt` L138-L142](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/install.txt#L138-L142)) | Not changed by the repo | OS level. A software swap does not touch it. |
| `/boot/config.txt` or `/boot/firmware/config.txt` | Kernel | Manual edit only | Not changed by the repo. The drexjj image ships its own. | OS level. A software swap does not touch it. |
| `/etc/ssh/ssh_host_*` | OS | No | drexjj `sshkeygen` deletes and regenerates them ([`src/sshkeygen.c` L10-L15](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sshkeygen.c#L10-L15)) | OS level |
| NetworkManager WiFi profiles | OS | No | drexjj panel WiFi control calls `nmcli` ([`src/wifi_panel.c` L42-L43](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/wifi_panel.c#L42-L43)) | OS level |
| `/etc/hostapd/`, `/etc/dnsmasq.d/uap0.conf`, `uap0.service` | Only v2 | Written by v2 `setup-ap.sh` ([`setup-ap.sh` L74-L143](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/setup-ap.sh#L74-L143)) | No | OS level. Only present if someone ran the v2 AP script. |

Git ignore rules: stock ignores `data/hw_settings.ini` and `data/user_settings.ini` ([`.gitignore` L1-L2](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/.gitignore#L1-L2)). drexjj also ignores `data/sbitx.db` ([`.gitignore` L4-L6](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/.gitignore#L4-L6)).
The drexjj `update` runs `git stash --include-untracked`. That does not stash ignored files, so these settings stay in place.

### The `hw=4` line

The v1 factory file starts with `bfo_freq=40035000` and `hw=4` ([`data/hw_settings_zbitxv1.ini` L1-L2](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/data/hw_settings_zbitxv1.ini#L1-L2)).

- Stock and drexjj `save_hw_settings()` both write `bfo_freq` and the band table only. Neither writes `hw=`.
- So any band power calibration deletes the `hw=4` line.
- drexjj then defaults to `hw=4` ([`src/sbitx.c` L2743-L2744](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sbitx.c#L2743-L2744)). The loss is invisible under drexjj.
- Stock instead probes address 0x08 and picks sBitx DE or V2 ([`sbitx.c` L1401-L1409](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L1401-L1409)).
- With a non-4 value, stock drives the LPF pins on each tune and uses a different T/R switch routine ([`sbitx.c` L351-L352](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L351-L352), [L1351-L1361](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sbitx.c#L1351-L1361)).

**Unverified:** A missing `hw=4` breaks the panel link. The code paths above touch LPF and T/R pins, not the bit-bang bus. Treat it as a separate defect to fix. Check with `grep -n '^hw' ~/sbitx/data/hw_settings.ini`.

### Same-card state that could explain the stall

These apply only if the "stock" card once held the drexjj image or drexjj software.
Each one is **Unverified**. The check column confirms or rules it out.

| Rank | Leftover state | Why it could stall the panel | Check |
|---|---|---|---|
| 1 | A drexjj `zbitx` process still starts at boot next to stock `sbitx` | Two programs drive the same two bus pins. drexjj uses the kernel, stock uses wiringPi. Their transfers collide. | `pgrep -a -f 'sbitx\|zbitx'` shows more than one process |
| 2 | drexjj `config.txt` overlay for `/dev/i2c-3` | A kernel driver owns BCM 6 and BCM 13 under stock code | `ls /dev/i2c-*`, `grep i2c-gpio` in `config.txt` |
| 3 | drexjj CPU clock or governor settings | The stock bit-bang speed follows CPU speed | `grep -E 'arm_freq\|force_turbo' config.txt`, `vcgencmd measure_clock arm` |
| 4 | `hw_settings.ini` without `hw=4` | Wrong board model. Touches LPF and T/R pins, not the bus. | `grep '^hw' ~/sbitx/data/hw_settings.ini` |

The symptom is the same under drexjj and stock software.
Ranks 1 to 3 need the drexjj OS on the card.
If the "stock" card is a clean factory card, none of these apply. The cause is then hardware or panel-side.

## Install and update procedures

### Stock v1 update (in place)

The factory card has the repo at `/home/pi/sbitx`. The update script does this ([`update` L8-L11](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/update#L8-L11)):

1. `cd /home/pi/sbitx`
2. Set `origin` to `https://github.com/afarhan/zbitx.git`.
3. `git pull`
4. `./build sbitx`. This compiles all sources with gcc and links wiringPi, ALSA, FFTW, ncurses, SQLite and GTK 3 ([`build` L36-L43](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/build#L36-L43)).

`update_zbitx` is the same, but it checks out the `farhandev` branch ([`update_zbitx` L8-L12](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/update_zbitx#L8-L12)).
The repo also commits a prebuilt `sbitx` binary.

A fresh OS install follows `install.txt`: wiringPi, FFTW3, ALSA and GTK dev packages, `snd-aloop`, the `config.txt` lines, iptables port 80 to 8080, hostname `sbitx`, and a fresh logbook ([`install.txt` L1-L152](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/install.txt#L1-L152)).

### drexjj install and update

The drexjj software targets its own Bookworm 32-bit image ([`README.md` L5](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/README.md#L5), [L18-L20](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/README.md#L18-L20)).
There are two ways to install it.

**A. Flash the image** ([release v5.11s](https://github.com/drexjj/zbitx/releases/tag/v5.11s)):

1. Back up `data/sbitx.db`, `data/hw_settings.ini`, `data/user_settings.ini` and `web/*.mc`.
2. Flash `zbitx511s.img.xz` with Etcher or Raspberry Pi Imager. The release notes say this erases the card.
3. Boot, then copy the backups into `~/sbitx/data` and `~/sbitx/web`.
4. Flash the drexjj panel firmware v5.11f or later.

**B. Git install on an existing drexjj image** ([`README.md` L35](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/README.md#L35), [L41](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/README.md#L41), [L49](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/README.md#L49)):

1. `mv ~/sbitx ~/sbitx_orig`
2. `git clone https://github.com/drexjj/zbitx sbitx && cd sbitx && ./update`
3. Copy `sbitx_orig/web/*.mc` and `sbitx_orig/data/*` into the new tree.

The drexjj `update` script does this ([`update` L8-L19](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/update#L8-L19)):

1. `cd $HOME/sbitx`
2. `git stash --include-untracked`
3. Set `origin` to `https://github.com/drexjj/zbitx.git`.
4. `git stash`, `git pull`, `git checkout main`
5. Rebuild `src/ft8_lib`, then `make clean` and `make`.

No drexjj script edits `config.txt`, systemd units, or `/etc`.
The only system-level scripts are `ntpsec.sh` (reinstalls NTPsec) and `sshkeygen` (new SSH host keys).

Method B on a stock Buster card fails at run time. The binary aborts because `/dev/i2c-3` does not exist.
So a working drexjj install almost always means the drexjj image was flashed.

**Unverified:** The owner flashed the drexjj image. If so, the card that held it has a Bookworm OS and a drexjj `config.txt`. Ask the owner which card they flashed.

## Restore to stock

No repo in scope names an official stock SD image or a download URL. See [Open questions](#open-questions).

### Option 1: separate stock card (preferred)

1. Use the original factory card, never flashed with the drexjj image. Or use a fresh image from HF Signals.
2. Confirm it is untouched. Check the OS and remote:

   ```bash
   cat /etc/os-release | head -3       # expect Raspbian Buster
   git -C ~/sbitx remote -v            # expect afarhan/zbitx
   ls ~/sbitx/zbitx 2>/dev/null        # expect: no such file
   grep -n '^hw' ~/sbitx/data/hw_settings.ini   # expect hw=4
   ```

3. Flash the stock panel UF2 from the same repo commit ([`zbitx_front_panel_sw.ino.uf2`](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/zbitx_front_panel_sw.ino.uf2)).

### Option 2: stock software on the same card (weaker)

Use this only if no stock card exists. It leaves the drexjj OS, `config.txt` and services in place.

1. Stop every radio process: `pkill -x zbitx; pkill -x sbitx`.
2. Disable any drexjj autostart or service found in [Boot chain](#boot-chain).
3. `mv ~/sbitx ~/sbitx_drexjj`
4. `git clone https://github.com/afarhan/zbitx.git ~/sbitx`
5. Copy back `hw_settings.ini`, `user_settings.ini` and `sbitx.db` from `~/sbitx_drexjj/data/`.
6. Add `hw=4` as line 2 of `hw_settings.ini` if it is missing.
7. `cd ~/sbitx && ./build sbitx`
8. Remove any `/dev/i2c-3` overlay from `config.txt`, then reboot.

**Unverified:** Stock software builds and runs on the drexjj Bookworm image. Stock targets Buster. The wiringPi version and the loopback module on the drexjj image are unknown.

Do not run the stock `update` inside a drexjj tree. The drexjj tree moved the sources to `src/`, so `git pull` from `afarhan/zbitx` merges two diverged histories.

## Pi-side diagnostics

### Get a shell

| Method | Details | Source |
|---|---|---|
| Hostname | `sbitx` on the stock image, so `sbitx.local` on the LAN | [`hostname` L1](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/hostname#L1) |
| SSH | `ssh pi@sbitx.local`. **Unverified:** SSH is on by default on the stock image. | - |
| Default login, drexjj image | User `pi`. The password is in the release notes. | [release v5.11s](https://github.com/drexjj/zbitx/releases/tag/v5.11s) |
| Default login, stock image | Not in any repo in scope | - |
| Host key warning | Expected after a drexjj flash, because the image regenerates host keys. Clear with `ssh-keygen -R sbitx.local`. | [`src/sshkeygen.c` L10-L15](https://github.com/drexjj/zbitx/blob/7d6e51140503eb939b01cf1f2fb8855dd16f30e3/src/sshkeygen.c#L10-L15) |
| Web UI | Port 8080. Port 80 only if the iptables step ran. | [`webserver.c` L15](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/webserver.c#L15), [`install.txt` L130-L136](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/install.txt#L130-L136) |
| Telnet command port | 8081. Accepts the same text commands as the panel. | [`remote.c` L35](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/remote.c#L35) |
| Hamlib rigctl | Port 4532 | [`hamlib.c` L205](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/hamlib.c#L205) |
| v2 AP mode | SSID `zbitx`, gateway 192.168.4.1. Only after `setup-ap.sh`. The default passphrase is in that script. | [`setup-ap.sh` L5](https://github.com/afarhan/zbitxv2/blob/2726c574c531da99b862a0adf68c1ddba671e1e9/setup-ap.sh#L5) |
| Local console | Mini-HDMI plus USB keyboard on the Pi. The stock program runs in a terminal window there. | [`sBitx.desktop` L8](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/sBitx.desktop#L8) |

If the web UI tunes the radio while the panel is frozen, the Pi main loop is alive.
If the web UI also freezes, the GTK main thread is stuck.

### Identify what is installed (read-only)

```bash
cat /etc/os-release | head -3
uname -a
git -C ~/sbitx remote -v; git -C ~/sbitx log -1 --oneline
ls -l ~/sbitx/sbitx ~/sbitx/zbitx 2>/dev/null
pgrep -a -f 'sbitx|zbitx'            # expect exactly one radio process
head -3 ~/sbitx/data/hw_settings.ini
```

### Check the panel bus pins (read-only)

Run these while the panel is stalled. They read pin state and do not drive anything.

```bash
pinctrl get 6,13        # Bookworm
raspi-gpio get 6 13     # Buster
```

Healthy idle bus: both pins read **high** (`hi` / `level=1`).
A pin that stays **low** means something holds the line: the panel, a short, or a Pi-side driver.
**Unverified:** Expected pin function is input with pull-up between transfers. The stock bit-bang sets input to release a line ([`i2cbb.c` L55-L63](https://github.com/afarhan/zbitx/blob/c671ac81e244242cb5718cd5b2b63f1d136695e0/i2cbb.c#L55-L63)).

### Check the radio process

```bash
top -H -p "$(pgrep -f -o 'sbitx|zbitx')"   # one thread pinned near 100% = spin in the bit-bang loop
dmesg | grep -iE 'i2c|gpio|wm8731|snd|under-voltage' | tail -40
lsmod | grep -E 'snd_aloop|i2c_gpio|i2c_bcm'
aplay -l                                  # expect the WM8731 as card 0, loopback as cards 1-3
ls -l /dev/i2c-*                          # stock: i2c-1 only; drexjj image: also i2c-3
journalctl -b --no-pager | grep -iE 'bitx|zbitx' | tail -40
```

Do not run `i2cdetect` on a bus while the radio runs. A probe collides with the radio's own traffic.

### Run the software in the foreground

Stock (GTK, needs the desktop session):

```bash
pkill -x sbitx; pkill -x zbitx
cd ~/sbitx && DISPLAY=:0 ./sbitx 2>&1 | tee /tmp/sbitx.log
```

drexjj (headless):

```bash
pkill -x zbitx; pkill -x sbitx
cd ~/sbitx && ./zbitx 2>&1 | tee /tmp/zbitx.log
```

**Unverified:** `DISPLAY=:0` works from SSH on the stock image without extra X authority setup.

Healthy stock output contains, in order:

1. `hw version: 4`
2. No `RTC not detected`, unless the RTC battery or chip is absent.
3. `zBitx front panel detected`
4. No repeating `Retrying I2C` or `I2CBB connection lost` lines.

## Open questions

1. Is the "stock" card the original factory card, or the card that held the drexjj image? The answer decides between a card-state cause and a hardware cause.
2. Where does HF Signals publish the official zBitx v1 SD image, and which version does it carry? No repo in scope links it.
3. How does the stock image autostart the radio: LXDE autostart, desktop entry, or systemd? Read it off a factory card.
4. What exact overlay creates `/dev/i2c-3` on the drexjj image, and on which pins? Read `config.txt` on a drexjj card.
5. Does the drexjj image run `zbitx` as a systemd service? If yes, does it still start after stock software goes into `~/sbitx`?
6. Does the drexjj image change CPU clock or governor settings that alter the stock bit-bang timing?
7. Does SCL_BB (BCM 6) read low while the panel is stalled? This splits "bus held" from "panel firmware stuck".
8. Is SSH enabled on the stock factory image, and what are its default credentials?
