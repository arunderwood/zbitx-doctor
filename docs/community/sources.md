# zBitx community and vendor sources

This page lists vendor and community knowledge about the zBitx HF transceiver.
It supports the diagnosis of one fault on zBitx v1 hardware.
The panel shows "waiting", paints part of the ready screen, then stalls.
Touch, encoder and waterfall stop. The battery voltage readout still updates.

Research date: 2026-09-25.

Conventions:

- **Official** means HF Signals, its product pages, its manual, or a post by Ashhar Farhan (VU2ESE).
- **Custom** means the drexjj project, maintained by JJ (W9JES).
- **Forum** means a post by another member of the BITX20 group. Treat it as an anecdote.
- **Unverified:** marks a claim that no primary source confirms.
- Forum posts are cited by message number. Posters appear by callsign or public handle only.

Access notes:

- groups.io refuses automated fetches (HTTP 402 and a bot check). We read threads in a logged-in browser.
- The drexjj Discord server needs a login. We did not review it.
- The zBitx Facebook group needs a login. We did not review it.

## Official resources

| Resource | What it holds |
|---|---|
| [HF Signals zBitx (v1) page](https://www.hfsignals.com/index.php/zbitx/) | Product page. It says the v1 no longer ships. It links the manual, source and stock panel `.uf2`. |
| [Stock v1 panel firmware (raw)](https://github.com/afarhan/zbitxfrontpanel/raw/refs/heads/main/zbitx_front_panel_sw.ino.uf2) | The link the v1 page uses. It returns a binary of 901,632 bytes (checked 2026-09-25). |
| [HF Signals zBitx v2 page](https://www.hfsignals.com/index.php/zbitx-v2/) | v2 product page. Its `.uf2` link is a GitHub `blob` page, not the raw file (see [Known issues](#k12-a-bad-uf2-download-leaves-the-old-firmware-in-place)). |
| [zBitx User Manual (PDF)](https://www.sbitx.net/wp-content/uploads/2025/02/zBitx-User-Manual-.pdf) | v1 manual, created 2025-03-30. It covers power-up, the Pi update and the Pico update. |
| [sbitx.net: Updating the zBitx](https://www.sbitx.net/index.php/updating-the-zbitx/) | Official Pi update (`cd sbitx`, `./update`) and Pico update steps. |
| [sbitx.net: Powering up the zBitx](https://www.sbitx.net/index.php/setting-up-the-zbitx/) | Normal boot sequence and power supply advice. |
| [sbitx.net: Creating a backup microSD card](https://www.sbitx.net/index.php/creating-a-backup-microsd-card-image/) | Links the stock v1 SD image on Google Drive. It tells you to save `hw_settings.ini` and `sbitx.db` first. |
| [sbitx.net: Upgrading zBitx v1 to v2 software](https://www.sbitx.net/index.php/2026/04/21/upgrading-your-zbitx-v1-to-zbitx-v2-software/) | 2026-04-21 procedure. It links the v2 image and v2 panel firmware. It has no revert steps. |
| [afarhan/zbitx](https://github.com/afarhan/zbitx) | Stock v1 Pi software, schematics (`schematics_zbitx.pdf`) and a copy of the panel `.uf2`. |
| [afarhan/zbitxfrontpanel](https://github.com/afarhan/zbitxfrontpanel) | Stock v1 panel source, panel schematic (`zbitx_front_panel_sch.pdf`) and `.uf2`. |
| [afarhan/zbitxv2](https://github.com/afarhan/zbitxv2) and [afarhan/zbitxv2_front_panel](https://github.com/afarhan/zbitxv2_front_panel) | v2 software. The Pi and panel talk over WiFi. |
| [YouTube: zBitx – Updating the Pico firmware](https://www.youtube.com/watch?v=Ii7MHGMr_dE) | Video of the Pico update. VU2ESE points users to it in [#123663](https://groups.io/g/BITX20/message/123663). |
| [BITX20 group on groups.io](https://groups.io/g/BITX20) | The only support channel HF Signals offers. A member quotes the vendor terms in [#123722](https://groups.io/g/BITX20/message/123722). |

Official boot sequence ([manual](https://www.sbitx.net/wp-content/uploads/2025/02/zBitx-User-Manual-.pdf), [sbitx.net](https://www.sbitx.net/index.php/setting-up-the-zbitx/)):

1. The panel lights first. It shows its firmware version and a "waiting for zBitx to start" message.
2. The Pi boots. The panel then shows the Pi software version.
3. The waterfall appears and band noise plays from the speaker.

Official Pico update ([manual](https://www.sbitx.net/wp-content/uploads/2025/02/zBitx-User-Manual-.pdf) p. 24):

1. Remove power.
2. Hold the main knob button down.
3. Connect the USB cable to the port marked CAT, not USB.
4. Release the knob. The screen goes all grey. The Pico mounts as a drive.
5. Copy the `.uf2` file to the drive. Restart the radio.

## Software lineage

### Stock v1 software (I2C link)

The Pi is the I2C master and the Pico is the slave ([VU2ESE, #116200](https://groups.io/g/BITX20/message/116200)).
The panel link shares one bit-banged I2C bus with the Si5351 and the RTC ([VU2ESE, #123092](https://groups.io/g/BITX20/message/123092)).
The WM8731 codec uses a separate bus ([VU2ESE, #123092](https://groups.io/g/BITX20/message/123092)).
Constant panel polling on that bus causes CW latency and receiver "hash" ([VU2ESE, #123631](https://groups.io/g/BITX20/message/123631)).

| Date | Component | Version | Source |
|---|---|---|---|
| 2025-03 | Pi app on early image | 3.021 | [#120928](https://groups.io/g/BITX20/message/120928), [#120994](https://groups.io/g/BITX20/message/120994) |
| 2025-03-26 | Panel | 1.04 | [commit b2da492](https://github.com/afarhan/zbitx/commit/b2da492) |
| 2025-03-30 | Panel | 1.05 | [commit 593f38f](https://github.com/afarhan/zbitxfrontpanel/commit/593f38f) |
| 2025-04-02 | Panel | 1.06 | [commit e29b3ba](https://github.com/afarhan/zbitxfrontpanel/commit/e29b3ba) |
| 2025-04-12 | Pi app | 3.052 | [commit d94fbc8](https://github.com/afarhan/zbitx/commit/d94fbc8) |
| 2025-07-06 | Pi app "release candidate" plus new panel `.uf2` | Pi stays 3.052; panel 1.07d | [#119408](https://groups.io/g/BITX20/message/119408), [#119418](https://groups.io/g/BITX20/message/119418), [#119472](https://groups.io/g/BITX20/message/119472) |
| 2025-07-09 | Panel `.uf2` in `zbitxfrontpanel` rebuilt (log time fix) | Reported as 1.07d | [commit dc4a5a5](https://github.com/afarhan/zbitxfrontpanel/commit/dc4a5a5) |

The current stock v1 pair is Pi 3.052 with panel 1.07d ([K1VP, #124161](https://groups.io/g/BITX20/message/124161), [#124177](https://groups.io/g/BITX20/message/124177)).

Two different "stock" panel files exist:

| Copy | Last change | Size | Git blob SHA |
|---|---|---|---|
| [afarhan/zbitxfrontpanel](https://github.com/afarhan/zbitxfrontpanel/blob/main/zbitx_front_panel_sw.ino.uf2) (HF Signals links this) | 2025-07-09 | 901,632 | `87df8a74` |
| [afarhan/zbitx](https://github.com/afarhan/zbitx/blob/main/zbitx_front_panel_sw.ino.uf2) | 2025-07-06 | 901,632 | `af854183` |

**Unverified:** both files show "1.07d" on screen. The data comes from the GitHub commit API on 2026-09-25.

The stock `./update` script sets the git remote to `afarhan/zbitx`, pulls, and rebuilds ([AC9TU, #119668](https://groups.io/g/BITX20/message/119668)).
A healthy stock start log prints "zBitx front panel detected" and repeated "reading data from i2c:" lines ([#118034](https://groups.io/g/BITX20/message/118034)).

### Stock v2 software (WiFi link)

- VU2ESE released v2 software in April 2026 ([afarhan/zbitxv2](https://github.com/afarhan/zbitxv2), [#123650](https://groups.io/g/BITX20/message/123650)).
- The Pi runs a WiFi access point with SSID `zbitx`. The Pico W joins it instead of using I2C ([README](https://github.com/afarhan/zbitxv2)).
- The README warns v1 owners to try it only if they know what they are doing.
- The v2 panel shows "Waiting for the zbitx wifi..." while it waits ([AC9KQ, #123697](https://groups.io/g/BITX20/message/123697)).
- The HF Signals v2 file reports v2.03. A build from the repo reports v4.00 (2026-04-27) ([N8ME, #124692](https://groups.io/g/BITX20/message/124692)).

A v2 panel image is not relevant to a stock v1 revert. It matters only if one was ever flashed.

### Custom software by drexjj (W9JES)

| Item | Detail | Source |
|---|---|---|
| Pi app repo | [drexjj/zbitx](https://github.com/drexjj/zbitx). Install clones into `~/sbitx` and runs `./update`. | [README](https://github.com/drexjj/zbitx) |
| SD image | v5.11s, released 2026-09-20. Debian Bookworm, 32-bit. For v1 and v2. | [Release v5.11s](https://github.com/drexjj/zbitx/releases/tag/v5.11s) |
| Panel repo | [drexjj/zbitxfrontpanel](https://github.com/drexjj/zbitxfrontpanel). Prebuilt file in [`build/`](https://github.com/drexjj/zbitxfrontpanel/tree/main/build). | [README](https://github.com/drexjj/zbitxfrontpanel) |
| Panel versions | v1.10 (2026-05-19, battery/SWR/power telemetry), v1.11–v1.12 (Aug 2026), v5.04z (2026-08-27), v5.05z–v5.11f (Sept 2026), v5.12, v5.13f (2026-09-23). | [commit history](https://github.com/drexjj/zbitxfrontpanel/commits/main) |
| Current panel file | `zbitxfrontpanel.v513f.uf2`, 376,320 bytes. | [Release v5.13f](https://github.com/drexjj/zbitxfrontpanel/releases/tag/v5.13f) |
| Pairing rule | v5.11s needs panel v5.11f or later. A mismatch gives missing menus, blank fields or dead controls. | [Release v5.11s](https://github.com/drexjj/zbitx/releases/tag/v5.11s) |
| Link type | Still I2C. It uses an OS-level I2C driver, not the stock user-space bit-bang. | [AC9TU, #125810](https://groups.io/g/BITX20/message/125810) |
| Related upstream proposal | Kernel `i2c-gpio` instead of user-space bit-banging. | [afarhan/zbitx PR #77](https://github.com/afarhan/zbitx/pull/77) |
| Panel without Pi update | Does not work. The panel sends commands the stock Pi app does not implement. | [AC9TU, #125824](https://groups.io/g/BITX20/message/125824) |
| Revert procedure | None published. The wiki and release notes cover install and upgrade only. | [drexjj wiki](https://github.com/drexjj/zbitx/wiki) |

The drexjj wiki says Menu → SETUP shows "Software version ---" when the panel gets no I2C reply ([Troubleshooting](https://github.com/drexjj/zbitx/wiki/Troubleshooting)).
This menu exists only in drexjj panel firmware.

Other forks exist: the K5KHK fork ([#123110](https://groups.io/g/BITX20/message/123110)), and `zbitxd` daemons by dg0jde and WP3DN ([#125648](https://groups.io/g/BITX20/message/125648)).

## Hardware revisions

| Feature | v1 | v2 |
|---|---|---|
| Release | Announced Dec 2024, shipped ~Mar 2025 ([#114800](https://groups.io/g/BITX20/message/114800), [#116200](https://groups.io/g/BITX20/message/116200)) | April 2026, USD 269 ([#123631](https://groups.io/g/BITX20/message/123631)) |
| Power switch | None ([KC1PHL, #125595](https://groups.io/g/BITX20/message/125595)) | Yes, with polarity protection ([#123631](https://groups.io/g/BITX20/message/123631)) |
| Main 5 V supply | Linear 7805 on the rear plate ([#123795](https://groups.io/g/BITX20/message/123795)) | Switching regulator ([#123631](https://groups.io/g/BITX20/message/123631)) |
| Panel 5 V supply | 78L05 (100 mA) through diode D1 to Pico VSYS ([#123751](https://groups.io/g/BITX20/message/123751), [#123752](https://groups.io/g/BITX20/message/123752)) | **Unverified:** not documented |
| Panel MCU | Pico W per VU2ESE ([#123784](https://groups.io/g/BITX20/message/123784)). Some early units have a plain Pico ([#123788](https://groups.io/g/BITX20/message/123788), [zbitxv2 #1](https://github.com/afarhan/zbitxv2/issues/1)). | Pico W |
| Stock panel link | I2C | WiFi |
| PA | IRF510 ([#123888](https://groups.io/g/BITX20/message/123888)) | AFT05MS004N LDMOS |
| Spur | −24 kHz spur fails FCC purity. The N3SB shield and "tombstone" mod fixes it ([#121612](https://groups.io/g/BITX20/message/121612), [#124537](https://groups.io/g/BITX20/message/124537)). | Compliant per AC9TU tests ([#124537](https://groups.io/g/BITX20/message/124537)) |
| Supply range | 6–9 V, 9 V at 3 A or more for the bench ([manual](https://www.sbitx.net/wp-content/uploads/2025/02/zBitx-User-Manual-.pdf)) | Same nominal range |

Software pairings reported to work:

| Hardware | Pi software | Panel firmware | Notes |
|---|---|---|---|
| v1 | Stock 3.051 / 3.052 | Stock 1.07d | Factory state ([#124161](https://groups.io/g/BITX20/message/124161)) |
| v1 | Stock v2 | v2.03 or v4.00 | Needs Pico W. Needs a stronger panel regulator or a 9 V supply ([#123736](https://groups.io/g/BITX20/message/123736), [#123922](https://groups.io/g/BITX20/message/123922)) |
| v1 or v2 | drexjj v5.11s | drexjj v5.11f or later | No hardware mod needed for the panel ([#125810](https://groups.io/g/BITX20/message/125810)) |
| v2 | Stock v2 | v2.03 or v4.00 | Factory state |

## Known issues matching our symptom

Match scale: **High** fits most of our symptom. **Medium** fits part of it. **Low** fits the family only.

### K1. Touch dead after reverting to the v1 SD card and old panel firmware

- **Date:** 2026-04-21 to 2026-04-22.
- **Links:** [OK1MAE #123672](https://groups.io/g/BITX20/message/123672), [#123675](https://groups.io/g/BITX20/message/123675); [KI7PBR #123680](https://groups.io/g/BITX20/message/123680), [#123687](https://groups.io/g/BITX20/message/123687), [#123698](https://groups.io/g/BITX20/message/123698); [VU2ESE #123674](https://groups.io/g/BITX20/message/123674).
- **Symptom:** Owners flash v2, then return to the v1 card and old `.uf2`. The screen draws normally. Touch does not respond.
- **Resolution:** A touch recalibration restores touch (official advice from VU2ESE).
- **Match:** High for touch. It does not explain the dead encoder or the missing waterfall.

### K2. Corrupt touch calibration on stock v1

- **Date:** 2025-07-07.
- **Link:** [KJ5KSH #119474](https://groups.io/g/BITX20/message/119474), [VU2ESE #119484](https://groups.io/g/BITX20/message/119484), [#119485](https://groups.io/g/BITX20/message/119485).
- **Symptom:** After a Pico update, touch goes sporadic, then dead. The screen displays correctly. The web UI works.
- **Resolution:** VU2ESE says calibration corruption causes this. Recalibration fixes it.
- **Match:** Medium.

### K3. Calibration data survives a UF2 flash

- **Date:** 2026-05-24 to 2026-05-27.
- **Links:** [N8ME #124549](https://groups.io/g/BITX20/message/124549), [#124550](https://groups.io/g/BITX20/message/124550), [#124560](https://groups.io/g/BITX20/message/124560), [#124575](https://groups.io/g/BITX20/message/124575).
- **Symptom:** On v2 hardware, calibration hangs at the third corner. `flash_nuke.uf2` then v2.03 still fails.
- **Resolution:** `flash_nuke.uf2` then stock 1.07d calibrates all four corners. The buttons that need no Pi reply work. A later WP3DN build with a WiFi start-up delay fixes v2 ([#124637](https://groups.io/g/BITX20/message/124637), [#124639](https://groups.io/g/BITX20/message/124639)).
- **Match:** Medium. It shows one firmware can inherit stored state from another.

### K4. "Waiting for zBitx to start" after a revert, caused by a corrupt SD card

- **Date:** 2026-04-25 to 2026-04-28.
- **Links:** [WB9FUW #123802](https://groups.io/g/BITX20/message/123802), [#123855](https://groups.io/g/BITX20/message/123855), [#123857](https://groups.io/g/BITX20/message/123857), [#123862](https://groups.io/g/BITX20/message/123862).
- **Symptom:** Original v1 card plus 1.07d. The panel stays at the waiting message. The Pi green LED stays solid.
- **Resolution:** A fresh card with the v1 image fixes it.
- **Match:** Low to medium. Our Pi boots and plays band noise.

### K5. Partial revert: panel works but still says "waiting"

- **Date:** 2026-06-25 to 2026-06-26.
- **Links:** [PA1CJT #125088](https://groups.io/g/BITX20/message/125088), [#125095](https://groups.io/g/BITX20/message/125095), [#125109](https://groups.io/g/BITX20/message/125109).
- **Symptom:** v2 card with the 1.07d `.uf2`. The screen works but reports it is waiting for the zBitx.
- **Resolution:** The original v1 card plus 1.07d restores full operation and RF.
- **Match:** Medium. It shows a Pi/panel software mismatch can look like a stuck handshake.

### K6. Panel link loss with I2C errors on stock v1

- **Date:** 2025-08-18 to 2025-08-29.
- **Links:** [#120392](https://groups.io/g/BITX20/message/120392), [KK7BVY #120395](https://groups.io/g/BITX20/message/120395), [AC9TU #120412](https://groups.io/g/BITX20/message/120412), [#120762](https://groups.io/g/BITX20/message/120762), [AC9TU #120767](https://groups.io/g/BITX20/message/120767).
- **Symptom:** The start log shows "I2CBB connection lost". The panel stops working. The panel lights from CAT USB power but not from 9 V.
- **Resolution:** None posted. Helpers suspect the 5 V regulator, the Pico, or the codec.
- **Match:** Medium. It is the closest stock-v1 I2C failure report.

### K7. "Repeating I2C" loop after hardware work

- **Date:** 2026-05-08 to 2026-08-16.
- **Link:** [thread 119222378](https://groups.io/g/BITX20/topic/119222378) ([#124082](https://groups.io/g/BITX20/message/124082), [#124522](https://groups.io/g/BITX20/message/124522), [#124524](https://groups.io/g/BITX20/message/124524), [#125568](https://groups.io/g/BITX20/message/125568)).
- **Symptom:** After a Pico swap and regulator rework, the app loops on I2C errors, with or without the panel.
- **Resolution:** A new Si5351 fixes it. AC9TU suggests you remove the panel to find a panel-side short.
- **Match:** Low. Our band noise shows the Si5351 works. The isolation method still applies.

### K8. Front end not talking to back end, "waiting for Pi"

- **Date:** 2026-06-06.
- **Link:** [K9AJE #124904](https://groups.io/g/BITX20/message/124904).
- **Symptom:** After a reimage, the panel says it waits for the Pi. HDMI and keyboard control work.
- **Resolution:** None posted.
- **Match:** Medium on symptom. The radio runs v2 software over WiFi, so the cause likely differs.

### K9. White screen or dead controls from low panel supply

- **Date:** 2026-04-21 to 2026-04-30.
- **Links:** [W2NDG #123723](https://groups.io/g/BITX20/message/123723), [#123724](https://groups.io/g/BITX20/message/123724), [#123731](https://groups.io/g/BITX20/message/123731); [W9JES #123745](https://groups.io/g/BITX20/message/123745); [PA3CNO #123751](https://groups.io/g/BITX20/message/123751), [#123922](https://groups.io/g/BITX20/message/123922); [KI7PBR #123736](https://groups.io/g/BITX20/message/123736).
- **Symptom:** With v2 WiFi firmware on v1, the panel goes white or ignores input below ~8.5 V. W9JES measures a 1–2 V drop at D1 to Pico pin 39.
- **Resolution:** Use a 9 V supply, or feed the Pico from USB. Or replace the 78L05 with a DI78M05 (500 mA) and D1 with a Schottky (SS14).
- **Match:** Low for stock v1 firmware. It is relevant if the panel rail is marginal or damaged.
- A secondary blog repeats these fixes ([vu3dxr.in](https://vu3dxr.in/zbitx-v2-migration-community-fixes-for-v1-hardware-owners/)). **Unverified:** it cites no threads.

### K10. Touch and knob dead on v2 after reflashing

- **Date:** 2026-05-26 and 2026-06-16.
- **Links:** [thread 119501264](https://groups.io/g/BITX20/topic/119501264) ([#124615](https://groups.io/g/BITX20/message/124615)); [afarhan/zbitxv2 #8](https://github.com/afarhan/zbitxv2/issues/8).
- **Symptom:** The screen draws. Touch and knob do nothing.
- **Resolution:** The WP3DN delayed-WiFi build fixes the forum case ([#124656](https://groups.io/g/BITX20/message/124656)). The GitHub issue is open.
- **Match:** Low. It is v2 WiFi firmware.

### K11. No waterfall from codec start failure

- **Date:** 2026-08-21 to 2026-08-23.
- **Link:** [thread 120864355](https://groups.io/g/BITX20/topic/120864355) ([KC1PHL #125592](https://groups.io/g/BITX20/message/125592), [AC9TU #125596](https://groups.io/g/BITX20/message/125596), [#125600](https://groups.io/g/BITX20/message/125600)).
- **Symptom:** Saturated waterfall and no audio. The log shows ALSA device errors.
- **Resolution:** The radio recovers after a long power-off. AC9TU links this to WM8731 start-up.
- **Match:** Low. Our audio works.

### K12. A bad UF2 download leaves the old firmware in place

- **Date:** 2025-03-30 to 2026-04-22.
- **Links:** [afarhan/zbitxfrontpanel #2](https://github.com/afarhan/zbitxfrontpanel/issues/2); [ON4JBS #119419](https://groups.io/g/BITX20/message/119419), [#119420](https://groups.io/g/BITX20/message/119420); [#123712](https://groups.io/g/BITX20/message/123712); [DH1AKY #123792](https://groups.io/g/BITX20/message/123792).
- **Symptom:** Browser downloads give a 0-byte file or files of different sizes. After the copy, the screen stays white or shows no new version.
- **Resolution:** Get the file from the HF Signals link, or use GitHub "download raw". No official checksums exist.
- **Match:** Medium. See [Diagnostic tips](#diagnostic-tips) for why this matters here.

Supporting facts:

- The v2 page links a GitHub `blob` URL. That URL returns `text/html`, not firmware (checked 2026-09-25).
- A forum reply quotes the `blob` URL for the v1 file too ([#124148](https://groups.io/g/BITX20/message/124148)).
- The Pico drive shows only `INDEX.HTM` and `INFO_UF2.TXT`, even after a good copy. This is normal ([SP5DAA #120714](https://groups.io/g/BITX20/message/120714), [#120715](https://groups.io/g/BITX20/message/120715)).
- **Unverified:** the RP2040 boot ROM ignores a non-UF2 file and keeps the old firmware. This is standard RP2040 behaviour, not a zBitx source.

### K13. Frozen front screen, then Pi reboot loop

- **Date:** 2025-08-25 to 2025-08-28.
- **Link:** [WA4THR #120668](https://groups.io/g/BITX20/message/120668), [#120711](https://groups.io/g/BITX20/message/120711), [#120744](https://groups.io/g/BITX20/message/120744).
- **Symptom:** The panel freezes while audio continues. Later the Pi reboots in a loop.
- **Resolution:** WA4THR resoldered a jumper across a burned trace and changed batteries.
- **Match:** Low. It shows a power fault can freeze the panel first.

## Revert-to-stock procedures as documented by others

No official revert procedure exists for v1. The steps below combine official update steps with forum reports.

1. Save `hw_settings.ini`, `user_settings.ini` and `sbitx.db` from `/home/pi/sbitx/data` ([sbitx.net](https://www.sbitx.net/index.php/creating-a-backup-microsd-card-image/), [VU2ESE #123650](https://groups.io/g/BITX20/message/123650)).
2. Write the stock v1 image to a new, good-quality card. The image is on [Google Drive](https://drive.google.com/file/d/1jpcgWAclM6i2csNBxPCs3Q1CK_7ma5UC/view?usp=sharing), linked from [sbitx.net](https://www.sbitx.net/index.php/creating-a-backup-microsd-card-image/).
   - AA6AT reports SHA-256 `a42a50eb56a7aad312bf7b4fcbe4ba2e584d0b963b92f2844f4a34f2e7ee5c68` ([#120992](https://groups.io/g/BITX20/message/120992)). **Unverified:** no official checksum exists.
   - The image boots Pi app 3.051 and works with panel 1.07d ([#120992](https://groups.io/g/BITX20/message/120992), [#124161](https://groups.io/g/BITX20/message/124161)).
   - Do not use the older KB9HV image (3.021). It fails with 1.07d ([#120994](https://groups.io/g/BITX20/message/120994)).
   - Use a new card, not a reflash of a suspect one ([AC9TU #125594](https://groups.io/g/BITX20/message/125594), [WB9FUW #123862](https://groups.io/g/BITX20/message/123862)).
3. Optional: run `./update` in `~/sbitx` to reach 3.052 ([K1VP #124177](https://groups.io/g/BITX20/message/124177)).
4. Flash the stock panel file with the official Pico procedure ([manual](https://www.sbitx.net/wp-content/uploads/2025/02/zBitx-User-Manual-.pdf)). Use the [raw link](https://github.com/afarhan/zbitxfrontpanel/raw/refs/heads/main/zbitx_front_panel_sw.ino.uf2).
5. Optional: erase the Pico flash first with Raspberry Pi's [`flash_nuke.uf2`](https://datasheets.raspberrypi.com/soft/flash_nuke.uf2), then flash stock ([N8ME #124560](https://groups.io/g/BITX20/message/124560)). This is a forum practice, not an HF Signals instruction.
6. Recalibrate touch ([VU2ESE #123674](https://groups.io/g/BITX20/message/123674), [#119484](https://groups.io/g/BITX20/message/119484)):
   1. Hold the stylus on the screen while you apply power.
   2. Lift the stylus when the screen flashes. Red targets appear in the four corners.
   3. Tap each target in the order shown: top-left, bottom-left, top-right, bottom-right ([N8ME #124549](https://groups.io/g/BITX20/message/124549)).
   4. Tap fast, and tap each corner once ([EA3HLF #125446](https://groups.io/g/BITX20/message/125446)).
   5. If a corner does not respond, loosen the front-plate screws and try again ([KX4TT #119475](https://groups.io/g/BITX20/message/119475), [#123741](https://groups.io/g/BITX20/message/123741)).
   6. If it still fails, power the panel from the CAT USB port only during calibration ([W9JES #123734](https://groups.io/g/BITX20/message/123734)).
7. Copy back the three saved files and restart ([VU2ESE #123650](https://groups.io/g/BITX20/message/123650)).

## Diagnostic tips

Tips from the vendor and the community:

- **Read the start log.** Connect HDMI and a keyboard. Run `./start.sh` from `/home/pi`. A healthy stock log shows "zBitx front panel detected", then "reading data from i2c:" lines ([#118034](https://groups.io/g/BITX20/message/118034), [#118035](https://groups.io/g/BITX20/message/118035)).
- **Isolate the panel.** Unplug the panel board. The stock Pi app still runs and the web UI works ([WP3DN #124085](https://groups.io/g/BITX20/message/124085), [AC9TU #124522](https://groups.io/g/BITX20/message/124522), [#124524](https://groups.io/g/BITX20/message/124524)).
- **Power the panel alone.** Feed the panel from the CAT USB port with the radio unpowered. This tests the Pico and display without the Pi ([KK7BVY #120395](https://groups.io/g/BITX20/message/120395)).
- **Check the rails.** Measure the main 5 V regulator output ([AC9TU #120412](https://groups.io/g/BITX20/message/120412), [AC9XH #120422](https://groups.io/g/BITX20/message/120422)). Measure the panel supply at D1 cathode to Pico pin 39 ([W9JES #123745](https://groups.io/g/BITX20/message/123745)).
- **Check audio devices.** Run `aplay -l` ([WP3DN #125598](https://groups.io/g/BITX20/message/125598)), or play a WAV file ([AC9TU #120767](https://groups.io/g/BITX20/message/120767)).
- **Let it rest.** Leave power off for five minutes or more if the codec fails to start ([AC9TU #125600](https://groups.io/g/BITX20/message/125600)).
- **Confirm the panel version on the boot screen.** It must read the stock version, not a drexjj `5.x` version. A failed copy leaves the old firmware running ([K12](#k12-a-bad-uf2-download-leaves-the-old-firmware-in-place)).
- **Check the `.uf2` file.** The stock file is 901,632 bytes. A drexjj v5.13f file is 376,320 bytes. An HTML page is neither.
- **Use a solid supply.** Use 9 V at 3 A or more on the bench ([manual](https://www.sbitx.net/wp-content/uploads/2025/02/zBitx-User-Manual-.pdf)).
- **Shut down cleanly.** Wait for the Pi green LED to go out before you remove power ([drexjj wiki](https://github.com/drexjj/zbitx/wiki/Troubleshooting)).

## Open questions

1. Where does the panel's battery voltage come from? The stock log shows the Pi reads `VBATT` from the panel ([#118034](https://groups.io/g/BITX20/message/118034)). **Unverified:** the Pico measures battery voltage itself. If so, a live readout proves only that the Pico main loop runs. It does not prove the I2C link works. The source code settles this.
2. Where does each panel firmware store touch calibration and settings in Pico flash? A UF2 copy does not erase other sectors. Stock 1.07d may read data that drexjj firmware wrote ([K3](#k3-calibration-data-survives-a-uf2-flash)). The source code settles this.
3. Which panel firmware runs now? The boot screen version string settles this. The two stock copies differ in content ([Software lineage](#stock-v1-software-i2c-link)).
4. Was the "stock SD" freshly imaged from the Google Drive file? A reused card may keep drexjj boot settings, such as an I2C overlay in `/boot/config.txt`. **Unverified:** drexjj adds such an overlay. The image contents settle this.
5. Is the panel MCU a Pico or a Pico W? It matters only if v2 firmware was ever flashed ([#123788](https://groups.io/g/BITX20/message/123788)).
6. Does the drexjj Discord hold revert reports? It needs a login. We did not review it.
