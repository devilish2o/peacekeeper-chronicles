# 🐧 PEACEKEEPER
### The Chromebook that refused to remain a Chromebook

> **ChromeOS + Debian 13 + KDE Plasma 6 / Wayland**  
> *A field journal of small victories, questionable decisions, and a surprisingly capable Celeron.*

![Status](https://img.shields.io/badge/status-daily%20driver-238636?style=flat-square) ![Debian](https://img.shields.io/badge/Debian-13%20Trixie-A81D33?style=flat-square&logo=debian) ![Desktop](https://img.shields.io/badge/KDE-Plasma%206-1D99F3?style=flat-square&logo=kde) ![Display](https://img.shields.io/badge/display-Wayland-555?style=flat-square) ![Updates](https://img.shields.io/badge/document-living-blue?style=flat-square)

**Last updated:** 15 October 2026 · **Maintainer:** ashvin · **Machine:** PEACEKEEPER

> [!NOTE]
> This is a **record of the configuration that worked on this particular machine**, not a universal Chromebook installer. Items marked **Verified** were reported working; **Check** means the result needs validation after future updates; **Historical** means it was part of the journey but is no longer the current setup. A command in a code block is documentation, **not** a command to execute blindly.

---

## Contents

1. [The machine and the mission](#1-the-machine-and-the-mission)
2. [From ChromeOS to Debian dual boot](#2-from-chromeos-to-debian-dual-boot)
3. [The great input-device rebellion](#3-the-great-input-device-rebellion)
4. [Audio: the silent treatment](#4-audio-the-silent-treatment)
5. [From XFCE to KDE Plasma Wayland](#5-from-xfce-to-kde-plasma-wayland)
6. [Keyboard layouts, shortcuts and screenshots](#6-keyboard-layouts-shortcuts-and-screenshots)
7. [Firefox, Spotify, networking and everyday software](#7-firefox-spotify-networking-and-everyday-software)
8. [Making a small Chromebook feel fast](#8-making-a-small-chromebook-feel-fast)
9. [Thunderbird: the graphite makeover](#9-thunderbird-the-graphite-makeover)
10. [Birdtray: the Wayland-versus-X11 diplomatic incident](#10-birdtray-the-wayland-versus-x11-diplomatic-incident)
11. [Battery indicator and session polish](#11-battery-indicator-and-session-polish)
12. [Known compromises and upgrade risks](#12-known-compromises-and-upgrade-risks)
13. [Diagnostics and quick recovery](#13-diagnostics-and-quick-recovery)
14. [Changelog and how to add the next fix](#14-changelog-and-how-to-add-the-next-fix)

---

## 1. The machine and the mission

| Component | PEACEKEEPER |
|---|---|
| Manufacturer / model | ASUS Chromebook CX1405CKA / CB1405CKA |
| Chromebook board | **AWASUKI** |
| CPU | Intel Celeron **N4500** (Jasper Lake) |
| Memory | **4 GB RAM** (as observed in earlier sessions; recheck if hardware variant differs) |
| Storage | **128 GB eMMC** |
| Original OS | ChromeOS 154.16805.0 (at initial setup) |
| Added OS | Debian **13 Trixie**, 64-bit |
| Current desktop | KDE Plasma **6.3.6**, **Wayland** |
| Initial desktop | XFCE, retained as fallback |
| Hostname | **PEACEKEEPER** (previously GALACTIC-PEACEKEEPER) |
| Locale | `en_CA.UTF-8` |
| Browser policy | **Firefox only** on Debian; no Chromium/Chrome installation |

**The mission:** preserve ChromeOS, install a real Debian desktop alongside it, get every piece of Chromebook hardware working, and avoid turning a lightweight machine into a benchmark for how slowly software can start.

**Outcome:** dual boot works; touchpad, keyboard, audio, KDE Plasma Wayland, screenshot shortcuts, Bluetooth, Spotify, Firefox, Thunderbird, SSHFS, power indicator and Birdtray were all reported working.

> **Philosophy:** A Celeron N4500 doesn't need more software. It needs *the right software*, preferably without 400 MB of “helpful” dependencies.

---

## 2. From ChromeOS to Debian dual boot

**Status: Verified booting both systems.**

The Chromebook was placed into **Developer Mode** and configured with **MrChromebox RW_LEGACY / edk2** firmware. This preserves the stock ChromeOS installation while making an alternate boot path available. Debian 13 was installed on a partition of the internal eMMC (approximately **64 GB allocated to Debian**), with GRUB reached through the alternate firmware boot route.

### What mattered

- **ChromeOS was preserved**, rather than replacing the stock firmware or wiping the whole drive.
- The board identifier **AWASUKI** mattered when looking up Chromebook-specific support.
- Debian booted successfully through edk2/GRUB.
- The installation initially used **XFCE**, before migrating to Plasma.

> [!WARNING]
> **Do not treat this paragraph as a fresh-install recipe.** Exact partition boundaries, firmware key sequences, and the actual boot menu entries were not retained in the available notes. Re-derive them from the live device before attempting a reinstall. Firmware and ChromeOS updates can affect dual-boot behaviour. Keep recovery media and a backup before changing firmware or partitions.

**Historical snag:** Debian booted, but the built-in keyboard and touchpad stopped responding once the kernel took over. The keyboard still worked in the boot menu. `i8042.reset` did **not** solve the problem. At least the computer had the courtesy to boot before refusing to speak to us.

---

## 3. The great input-device rebellion

### 3.1 Touchpad pressure quirk — **Verified**

**Symptom:** internal touchpad not usable correctly under Debian; eventually worked after login.

**Hardware identification:** `PNP0C50:00 093A:200F Touchpad`.

**Persistent fix:** `/etc/libinput/local-overrides.quirks`

```ini
[ASUS Awasuki Touchpad Pressure]
MatchName=PNP0C50:00 093A:200F Touchpad
MatchUdevType=touchpad
AttrPressureRange=35:25
AttrPalmPressureThreshold=250
```

This was the confirmed working local libinput override. The quirk survives ordinary package updates because it is a local `/etc` configuration file. After a major libinput upgrade, test tapping, dragging, palm rejection and behaviour at the login screen.

**Check:**

```bash
cat /etc/libinput/local-overrides.quirks
libinput --version
```

### 3.2 Keyboard — **Verified**

The keyboard worked in the boot environment but initially failed in the Debian session. After troubleshooting, it was working; the initial kernel-level recovery sequence is not fully documented, so **do not confuse the later layout fix with a hardware-driver fix**.

Under the original XFCE desktop, the stable layout was the Chromebook keyboard model with Canadian English:

```bash
setxkbmap -model chromebook -layout ca -variant eng
setxkbmap -query
```

Persistent XFCE values (historical; Plasma now manages its own layouts):

```text
/Default/XkbDisable = false
/Default/XkbModel   = chromebook
/Default/XkbLayout  = ca
/Default/XkbVariant = eng
```

**Why `ca(eng)`?** Other tested layouts, including `ca(fr)` and `fr-legacy`, produced awkward punctuation mappings (notably `/`). `ca(eng)` restored the expected dedicated slash key.

---

## 4. Audio: the silent treatment

**Status: Verified working, including speaker tray, volume OSD, media keys, mute, volume up and down.**

**Device:** Intel Jasper Lake HD Audio `8086:4dc8`, using the **SOF / rt5650** audio path.

The stock Debian installation did not immediately produce working Chromebook audio. The successful intervention was the community **chromebook-linux-audio** script from **WeirdTreeThing**, followed by testing in the real Plasma session.

**Result:** audio played correctly and the media keys worked.

> [!CAUTION]
> The exact script revision, invocation and files it changed were not preserved. Do **not** rerun a random copy of an online audio-fix script on a working machine. Before kernel or SOF firmware changes, capture current package versions and test sound after reboot.

Useful non-destructive diagnostics:

```bash
lspci -nn | grep -i audio
wpctl status
systemctl --user status wireplumber --no-pager
```

The sound of success was, for once, actual sound.

---

## 5. From XFCE to KDE Plasma Wayland

**Status: Verified.**

The first desktop was XFCE, a sensible lightweight starting point. We then moved to **KDE Plasma 6.3.6 running on Wayland**, keeping XFCE as a fallback.

**What worked:** Plasma session, window management, panel/system tray, media-key OSD, touchpad, audio, Spectacle and desktop effects. **Wobbly Windows** and **Magic Lamp** were enabled for fun. The traditional **Desktop Cube** is not included by default in this Plasma 6.3 setup.

The practical lesson: *modern Plasma can feel responsive on modest hardware when you avoid installing everything that happens to have a checkbox.* This is an observed experience, not a measured benchmark.

### Desktop screenshot gallery

The screenshots below are selected captures recovered from our earlier conversation. Captions are intentionally conservative where a screenshot's exact diagnostic context is uncertain.

![PEACEKEEPER desktop and application appearance](assets/desktop-20261015.png)

*Recovered screenshot from the desktop-customization sessions, 15 October 2026.*

---

## 6. Keyboard layouts, shortcuts and screenshots

### 6.1 Plasma keyboard layouts — **Verified**

Two layouts were configured in KDE: **Canadian English** and **US International with dead keys**. KDE handles the active layouts in its own settings; don't apply the old XFCE `setxkbmap` startup configuration blindly to a Wayland session.

### 6.2 Spectacle selection → clipboard — **Verified**

**Goal:** select a rectangle and put it directly on the clipboard, without manually saving a PNG.

Working command:

```bash
spectacle -r -c -b -n
```

**Shortcut:** `Meta+Shift+S`.

There was a shortcut conflict; removing the conflicting custom assignment and then configuring the desired action made the shortcut work. Spectacle version was **6.3.5**.

**Test:** press `Meta+Shift+S`, select a region, then paste into an application.

### 6.3 Function/media keys — **Verified in Plasma**

The Chromebook media keys (volume and mute) functioned after the desktop/audio fixes. The keyboard layout was tuned for the actual Canadian-labelled Chromebook keyboard, rather than assuming a generic US PC keyboard.

---

## 7. Firefox, Spotify, networking and everyday software

### 7.1 Firefox-only browser policy — **Verified preference**

Firefox ESR is the chosen Debian browser. Chromium/Chrome were deliberately avoided to reduce duplicated browser installations and keep storage/RAM pressure down. WhatsApp Web was used through Firefox. Firefox ESR **153.4.0** was observed in a later session.

### 7.2 Spotify signing key — **Verified installation**

A Spotify APT repository key problem was fixed; the key fingerprint recorded during troubleshooting was:

```text
E109 6BCB FF6D 4187 96DE 7851 5384 CE82 BA52 C83A
```

Spotify was subsequently installed and running. For a future reinstall, follow the then-current official repository/keyring instructions; don't resurrect deprecated `apt-key` recipes merely because they're nostalgic.

### 7.3 SSHFS to DietPi — **Verified**

The remote DietPi filesystem is mounted at:

```text
/mnt/dietpi
```

A **systemd automount** was configured and a permissions issue was resolved. This allows PEACEKEEPER to browse remote storage without permanently mounting it during every login.

**Check without changing configuration:**

```bash
findmnt /mnt/dietpi
systemctl list-units --type=automount --all
```

The exact mount unit and remote target were not retained in the available notes. Read the current unit before reproducing it on another host.

### 7.4 Terminal locale and drawing characters — **Verified**

The locale was set to `en_CA.UTF-8`; `xterm-256color` and box-drawing characters worked. Handy sanity check:

```bash
locale
locale charmap
printf 'Unicode: ───── │ ┌──┐ ✓ é è à €\n'
```

---

## 8. Making a small Chromebook feel fast

**The rule:** *every megabyte is innocent until proven guilty; every unnecessary daemon is a suspect.*

### 8.1 Package cleanup — **Verified removals**

Removed as unnecessary for this installation:

- LibreOffice and GIMP, including an unneeded Swedish GIMP help package (`gimp-help-sv`).
- Unrelated firmware packages, including Netronome, AMD graphics, NVIDIA graphics and Marvell Prestera, after checking the Chromebook's hardware.
- IBus packages and orphaned dependencies when they were no longer needed.
- Extra Noto font bundles: `fonts-noto-extra`, `fonts-noto-cjk-extra`, `fonts-noto-ui-extra`.

Kept useful font coverage, including `fonts-noto-core`, `fonts-noto-cjk` and `fonts-noto-color-emoji`. The font cleanup alone saved roughly **600 MB**; disk use was observed around **22%** after cleanup.

> [!WARNING]
> This is a historical removal list, **not** a bulk purge command. Some packages may be dependencies on a future Debian version. Always preview removal transactions and verify actual hardware first.

### 8.2 Preferred APT workflow

```bash
sudo apt update
sudo apt -s full-upgrade       # inspect before major changes
sudo apt -s autoremove         # inspect proposed removals
```

For targeted installs, use `--no-install-recommends` when appropriate. Avoid `autoremove` without reading the transaction, particularly around kernels, audio and firmware.

### 8.3 IBus warning cleanup — **Verified package removal; session environment needed attention**

IBus warnings appeared in the KDE environment. IBus and its GTK/Hangul integration packages were removed because they weren't needed for the chosen keyboard layouts. Environment variables referencing IBus persisted in one session after removal; verify a fresh login rather than assuming a package purge clears inherited session variables.

```bash
printenv | grep -i ibus
```

Do not indiscriminately delete input-method settings if you later need complex-language input.

### 8.4 Performance observations — **Not benchmarked**

- Plasma Wayland felt responsive on the N4500.
- Unnecessary applications, font sets and background integrations were removed.
- No controlled ChromeOS-versus-Debian battery, memory or benchmark results have been collected.
- Real-world checks still worth doing: suspend/resume, idle drain, battery runtime, Wi-Fi/Bluetooth after wake and browser memory usage.

---

## 9. Thunderbird: the graphite makeover

**Status: Verified working.**

The original dark Thunderbird theme was too close to black. We created a more restrained **graphite / charcoal** palette with muted blue selections, then styled both the Thunderbird chrome and the message-reading surface.

![Thunderbird graphite appearance](assets/thunderbird-graphite.png)

*Recovered screenshot from the Thunderbird customization session.*

### 9.1 The profile and CSS locations

Thunderbird profile observed on PEACEKEEPER:

```text
/home/ashvin/.thunderbird/9a5vajr4.default-default/
```

The **correct** paths are:

```text
~/.thunderbird/9a5vajr4.default-default/chrome/userChrome.css
~/.thunderbird/9a5vajr4.default-default/chrome/userContent.css
```

**Important failed attempt:** placing `userChrome.css` at the *profile root* did nothing. It must be in the `chrome/` subdirectory. After moving it, the styling took effect.

In Thunderbird's Config Editor, enable:

```text
toolkit.legacyUserProfileCustomizations.stylesheets = true
```

### 9.2 Graphite palette

| Purpose | Colour |
|---|---|
| Toolbar / base graphite | `#252b34` |
| Folder pane | `#2b323d` |
| Thread list | `#292f39` |
| Message header | `#303743` |
| Selected row | `#355476` |
| Main text | `#e6eaf0` |
| Accent / links | `#58b4f5` |

The original full CSS files should be backed up from the live Thunderbird profile; this journal documents their placement and palette, not an invented replacement for every selector.

**Upgrade caveat:** Thunderbird may change internal selectors. The CSS files usually survive updates, but some visual rules can stop applying.

### 9.3 Remote content and privacy

Thunderbird's remote-content warning was discussed. Leaving remote content blocked is the privacy-preserving default; selectively allowing trusted senders is preferable to turning on every remote image globally.

---

## 10. Birdtray: the Wayland-versus-X11 diplomatic incident

**Status: Verified after logout/login.** Thunderbird starts hidden, Birdtray appears in the tray, and clicking the tray icon restores the window.

Birdtray package observed: **`1.9.0+ds-2+b2`**.

### 10.1 The original problem

Birdtray crashed with a **segmentation fault** when started using the default Qt Wayland platform. Starting it with Qt's XCB backend worked:

```bash
QT_QPA_PLATFORM=xcb birdtray
```

But native-Wayland Thunderbird windows could not be hidden reliably by this Birdtray setup. The working combination was **Birdtray on XWayland + Thunderbird on X11/XWayland**.

### 10.2 The working Thunderbird wrapper — **Verified**

Create:

```text
/home/ashvin/.local/bin/thunderbird-x11
```

Contents:

```sh
#!/bin/sh
export MOZ_ENABLE_WAYLAND=0
export GDK_BACKEND=x11
exec /usr/bin/thunderbird "$@"
```

Make executable:

```bash
chmod +x ~/.local/bin/thunderbird-x11
```

In **Birdtray → Settings → Advanced**, set **Thunderbird command line** to the executable path **only**:

```text
/home/ashvin/.local/bin/thunderbird-x11
```

**Failed attempt worth remembering:** entering `env MOZ_ENABLE_WAYLAND=0 GDK_BACKEND=x11 thunderbird` in that Birdtray field produced `execvp: No such file or directory`. Birdtray treated the whole string as an executable path, not a shell command. A wrapper script fixed it.

Window name pattern: `Mozilla Thunderbird`.

### 10.3 Birdtray KDE autostart — **Verified**

File:

```text
~/.config/autostart/birdtray.desktop
```

Contents:

```ini
[Desktop Entry]
Type=Application
Name=Birdtray
Comment=Thunderbird system tray integration
Exec=env QT_QPA_PLATFORM=xcb birdtray
Icon=birdtray
Terminal=false
X-KDE-autostart-after=panel
```

### 10.4 Birdtray settings — **Verified**

Under **Settings → Hiding**:

- **Start Thunderbird when Birdtray starts:** enabled.
- **And hide it:** enabled for background startup.
- **Hide/show Thunderbird window when clicking on tray icon:** enabled.
- **Hide Thunderbird window when it is minimized:** enabled.
- **Terminate Thunderbird when quitting Birdtray:** left **unchecked**.

A prompt about monitoring email folders appeared at startup. Choose **Don't ask again** if Birdtray is only needed for minimizing/restoring Thunderbird. Folder monitoring is optional.

**Behavioural detail:** minimize Thunderbird to hide it in the tray. The **X** close button may still quit Thunderbird; this configuration does not redefine Close as Minimize.

### 10.5 Don't force all Mozilla apps onto X11

If an earlier experiment created a global override, remove it **only if present and no longer needed**:

```bash
rm -f ~/.config/environment.d/thunderbird.conf
```

The wrapper scopes X11 settings to Thunderbird and avoids changing Firefox's display backend.

> **Lesson:** Wayland and X11 can coexist peacefully, provided you don't ask Birdtray to mediate international relations without a wrapper script.

---

## 11. Battery indicator and session polish

**Status: Verified after login.**

The battery indicator was initially missing from the minimal Plasma installation. Installing PowerDevil fixed the missing functionality:

```bash
sudo apt install --no-install-recommends powerdevil
```

After a later logout/login, the indicator was missing **visually**, but the fix was simple:

**Right-click system tray → Configure System Tray → Entries → Power and Battery → Always shown.**

The icon reappeared immediately. No reinstall required.

If the service itself fails in the future:

```bash
dpkg -l powerdevil upower | grep '^ii'
systemctl --user status plasma-powerdevil.service --no-pager
upower -e
```

Restart only if the service exists and is actually unhealthy:

```bash
systemctl --user restart plasma-powerdevil.service
```

**Lesson:** before rebuilding the power-management stack, check whether KDE has simply hidden the icon. Sometimes the dragon is just behind the curtain.

---

## 12. Known compromises and upgrade risks

| Area | Likely update resilience | What to verify |
|---|---|---|
| KDE preferences and shortcuts | High | Shortcuts, tray entries, effects |
| libinput touchpad quirk in `/etc` | High | Touchpad after new libinput/kernel |
| Chromebook audio fix | Medium | Speakers, mic, volume keys after kernel/SOF changes |
| Thunderbird custom CSS | Medium | Theme after Thunderbird major updates |
| Birdtray XWayland workaround | Medium | Autostart, hide/restore after Qt/KDE/Thunderbird changes |
| SSHFS systemd automount | High | Mount availability and permissions |
| RW_LEGACY / ChromeOS dual boot | Special care | Boot paths after firmware/ChromeOS updates |
| Battery life and suspend | Unmeasured | Test against ChromeOS on same hardware |

**Debian gains:** native Linux packages, host-level administration, KDE customization, systemd, SSHFS, development and server tools. **ChromeOS retains advantages:** integrated hardware tuning, verified-boot/security model (subject to Developer Mode), Android app support where available, and generally simpler automatic maintenance. Keeping both systems is the right compromise for this machine.

### Upgrade discipline

1. Check what APT intends to change before large upgrades.
2. Preserve a working older kernel where practical.
3. Reboot and test input, audio, suspend, tray and Birdtray.
4. Save configuration backups before major upgrades.
5. **Do not overwrite known-good configuration with untested snippets from the internet — including snippets in this journal.**

**Backup roadmap (not yet implemented):** VPS first, then PEACEKEEPER and DietPi. Avoid documenting a backup system as operational until one exists and a restore has been tested.

---

## 13. Diagnostics and quick recovery

These commands inspect the system without changing it:

```bash
# Identity and desktop
hostnamectl
cat /etc/os-release
uname -a
printf 'Session: %s\n' "$XDG_SESSION_TYPE"

# Storage and memory
lsblk -f
df -h /
free -h

# Input and locale
cat /etc/libinput/local-overrides.quirks
locale

# Audio and battery
wpctl status
upower -e
systemctl --user status plasma-powerdevil.service --no-pager

# Birdtray and Thunderbird
pgrep -a -f 'birdtray|thunderbird'
cat ~/.config/autostart/birdtray.desktop
cat ~/.local/bin/thunderbird-x11

# Network mount
findmnt /mnt/dietpi

# Package changes
apt history 2>/dev/null || true
zgrep -hE '^(Start-Date|Commandline|Install|Remove|Purge):' /var/log/apt/history.log* 2>/dev/null | tail -80
```

**If something breaks:** record the exact symptom, version and output *before* trying fixes. One change at a time. The goal is to repair PEACEKEEPER, not create a new archaeological layer.

---

## 14. Changelog and how to add the next fix

### 2026-10-15 — Thunderbird, Birdtray and power indicator

- Graphite Thunderbird styling verified with CSS in the profile's `chrome/` directory.
- Birdtray Qt Wayland crash bypassed using `QT_QPA_PLATFORM=xcb`.
- Thunderbird X11 wrapper added; Birdtray command-line parsing pitfall resolved.
- KDE autostart configured; after logging out and back in, Thunderbird started hidden as intended.
- Power-and-Battery tray entry restored using **Always shown**.
- This living journal was started.

### 2026-10-13–14 — Plasma polish

- KDE Plasma Wayland operational, media keys/audio and touchpad working.
- Spectacle rectangle-to-clipboard shortcut fixed.
- Firefox, Spotify and SSHFS operational.
- Unneeded fonts, packages and IBus integrations cleaned up.
- PowerDevil added to minimal Plasma setup.

### 2026-10-11–13 — The migration

- ChromeOS preserved; Debian 13 installed through RW_LEGACY/edk2 and GRUB.
- Keyboard/touchpad troubles investigated; libinput touchpad quirk and keyboard layout settled.
- Chromebook audio repaired with the WeirdTreeThing script.
- XFCE gave way to Plasma Wayland.

### Template for the next repair

Copy this block at the **top of the changelog** whenever we fix something:

```markdown
### YYYY-MM-DD — Short, memorable title

**Status:** Verified / Check / Historical  
**Symptoms:** What broke, and how to reproduce it.  
**Cause:** Confirmed root cause (or "not established").  
**Fix:** Exact commands, file paths and UI settings.  
**Validation:** What was tested, including reboot/login if relevant.  
**Rollback:** How to undo safely.  
**Screenshot:** `assets/descriptive-name.png` (if useful).  
**One-liner:** A little humour at the expense of the bug, not the user.
```

---

## Appendix: screenshots and source hygiene

This document references a small set of recovered conversation screenshots. They live in `assets/`, so the Markdown remains **one editable journal** while the images remain ordinary GitHub-friendly files. More screenshots can be added later with descriptive names and captions.

- `assets/desktop-20261015.png` — recovered desktop/customization capture.
- `assets/thunderbird-graphite.png` — recovered Thunderbird customization capture.

**Privacy check before publishing publicly:** screenshots can contain email addresses, subject lines, personal files, browser tabs and hostnames. Review and redact images before pushing to a public repository. Likewise, never commit passwords, tokens, private SSH keys, `.kdbx` files or full Thunderbird mail profiles.

**Evidence policy:** this journal combines user-confirmed outcomes and retained troubleshooting notes. Where exact historical commands or screenshots were unavailable, that is stated instead of inventing them. When we next have terminal access to PEACEKEEPER, we can enrich it with `dpkg` versions, actual systemd unit files, boot layout and current configuration hashes.

---

<p align="center"><strong>PEACEKEEPER</strong> · <em>Too old for ChromeOS boredom. Too stubborn to retire.</em> 🐧</p>
