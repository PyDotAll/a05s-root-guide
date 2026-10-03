# Windows 11 Toolchain — Open-Source & Free Only

Every tool below is **free and open-source**. Where no open-source option is adequate, that is
stated plainly rather than papered over.

Last updated <CAPTURE_DATE>.

---

## ⚠️ The one unavoidable problem

**Classic Odin3 is proprietary leaked software.** It is not open-source, Samsung never released
it, and it circulates through unofficial download sites — many of which serve bundled adware.

Your requirement is open-source and free. Odin3 fails both the *open-source* test and the
*trustworthy-provenance* test. So this page documents the open-source route, and is honest
where it's less mature.

| Function | Open-source option | Maturity |
|---|---|---|
| Download firmware | **samloader-rs / Bifrost / samloader** | ✅ Mature |
| Flash firmware | **samloader-rs** or **odin4** | ⚠️ Less battle-tested than Odin3 |
| USB driver | **Zadig** (libusb/WinUSB) | ✅ Mature |

---

## 1. Firmware download

All query Samsung's official FUS server — the same source Frija uses — but with published
source code.

### ⭐ samloader-rs — recommended

| | |
|---|---|
| Repo | `github.com/topjohnwu/samloader-rs` |
| License | **Apache 2.0** |
| Author | **topjohnwu** — the Magisk maintainer |
| Platform | Windows, Linux, macOS |

Best choice: download **and** flash in one tool. Decrypts on-the-fly, parallel downloads to
beat server throttling, streams raw LZ4 straight to the device.

Flashing is a derivative of **Heimdall** (MIT), copyright headers preserved.

```bash
# download + decrypt
samloader download --model SM-A057F --region <CSC> --firmware <YOUR_FIRMWARE_BUILD> --imei <SERIAL>

# list available firmware
samloader list --model SM-A057F --region <CSC>

# flash
samloader flash --file BL_xxx.tar.md5
```

> **Don't paste your serial into a shared terminal, screenshot, or this repo.**
> See [DEVICE-SPECIFICATIONS.md](DEVICE-SPECIFICATIONS.md#serial-number--deliberately-not-stored-here).

### Bifrost — GUI option

| | |
|---|---|
| Repo | `github.com/zacharee/Bifrost` |
| License | **MIT** |

Graphical, cross-platform, works on Android too. Download-only.

### samloader — the original

| | |
|---|---|
| Repo | `github.com/nlscc/samloader` |
| License | **GPL** |

Python, widely used, well documented.

### Also available

| Tool | License | Notes |
|---|---|---|
| `Samsung-Loki/Syndical` | MPL-2.0 | C#, faster than samloader |
| `jesec/samfirm.js` | GPL-3.0 | npm-based, streaming |

**Avoid:** Frija — free but not open-source. `samfw.com` — a site, not source you can audit.

---

## 2. Flashing

This is where the open-source requirement bites. Two options:

### ⭐ samloader-rs (flash subcommand)

Apache 2.0, actively developed, Windows native. **Best-supported open-source choice.**

Uses the **Heimdall** protocol path. On some Samsung devices this works; on others Samsung's
throttling has closed it.

### odin4 — implements Samsung's Thor protocol

| | |
|---|---|
| Repo | `github.com/Llucs/odin4` |
| License | Open source — check repo for current terms |
| Language | C++ |
| Interfaces | CLI + Qt6 GUI |

Why it matters: this implements **Thor**, the same protocol Odin uses, rather than Heimdall's.
Thor is faster and has broader device support.

**Status — read this before relying on it:**

- Originally Linux-only; cross-platform support (including a `windows-2022` build) landed via PR #138, June 2026
- The author states: *"The project compiles and the core protocol logic is implemented, but I still need real device testing before the first official stable release"*
- So: **promising, not proven.** Small project (27 stars). Auditable, but young.

```bash
git clone https://github.com/Llucs/odin4.git
cd odin4
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build

./build/odin4 -l                       # list Download Mode devices
./build/odin4 --check-only -a AP_x.tar.md5    # dry run — validate, don't flash
./build/odin4 -b BL_x.tar.md5 -a magisk_patched.tar
```

Windows build needs `cmake`, `ninja`, `libusb`, `Crypto++` (vcpkg or Chocolatey).

### odin4 (MhAhmadAli) — reconstructed

| | |
|---|---|
| Repo | `github.com/MhAhmadAli/odin4` |
| License | **MIT** |

A reconstruction of Samsung's leaked **Odin4** (their internal Linux build), from Ghidra
decompilation, referencing Heimdall. Multi-device support, PIT handling, MD5/SHA256
verification.

> ⚠️ Its README states **"educational purposes only."** Read the terms yourself.

### Heimdall

| | |
|---|---|
| Repo | `github.com/Benjamin-Dobell/Heimdall` |
| License | **MIT** |

Long-standing, but **development has slowed substantially**. Fine for bootloader operations;
less reliable as a full flashing replacement.

### Bottom line on flashing

There is **no mature, widely-proven, fully open-source Odin replacement for Windows.**

Honest framing:

- If open-source is a **hard requirement** → use `samloader-rs` flash, or `odin4`. Accept more
  troubleshooting.
- If reliability is paramount → Odin3 is what the community actually uses, but it is
  proprietary, leaked, and only as trustworthy as whatever site you downloaded it from.

Given your original concern was a possible compromise, **a tool from a random mirror is itself
a risk.** Prefer building from source.

---

## 3. USB driver

Odin needs Samsung's proprietary driver. Open-source tools use **libusb/WinUSB** instead.

### Zadig ⭐

| | |
|---|---|
| Repo | `github.com/dalesteiner/zadig` (official builds at `zadig.akeo.ie`) |
| License | **GPL-2.0** |

Install the **WinUSB** driver for the Samsung Download Mode device instead of Samsung's
proprietary driver. Works with samloader, odin4, and Heimdall.

> ⚠️ If you also want Samsung Smart Switch working, keep Samsung's driver installed and switch
> between them in Device Manager as needed.

---

## 4. Supporting tools

| Tool | License | Purpose | Get it |
|---|---|---|---|
| **Magisk** | Apache 2.0 | Root | `github.com/topjohnwu/Magisk/releases` |
| **Android platform-tools** (`adb`) | Apache 2.0 | ADB | `developer.android.com/tools/releases/platform-tools` |
| **7-Zip** | LGPL | Open `.tar.md5` | `7-zip.org` |
| **lz4** | BSD-2 | LZ4 decompress/recompress | `lz4.org` · Windows: `github.com/lz4/lz4/releases` |
| **Python** | PSF | scripting | `python.org` |
| **Git** | GPL-2.0 | cloning sources | `git-scm.com` |
| **Notepad++** | GPL-3.0 | editing text | `notepad-plus-plus.org` |

### Windows convenience: WSL2

Since the phone already has a full Debian PRoot with `lz4`, `7z`, `tar`, and `python3`
preinstalled, **WSL2** gives you the same tooling on Windows 11 natively:

```bash
wsl --install
```

Then extract and repatch firmware inside WSL using the same Linux commands in
[DETAILED-GUIDE §6](DETAILED-GUIDE.md#6-patch-with-magisk).

---

## 5. Complete download list

| # | Tool | License | Where |
|---|---|---|---|
| 1 | **samloader-rs** | Apache 2.0 | `github.com/topjohnwu/samloader-rs` |
| 2 | **odin4** | Open source | `github.com/Llucs/odin4` |
| 3 | **Zadig** | GPL-2.0 | `zadig.akeo.ie` |
| 4 | **Magisk** | Apache 2.0 | `github.com/topjohnwu/Magisk/releases` |
| 5 | **platform-tools** | Apache 2.0 | `developer.android.com/tools/releases/platform-tools` |
| 6 | **7-Zip** | LGPL | `7-zip.org` |
| 7 | **lz4** | BSD-2 | `github.com/lz4/lz4/releases` |
| 8 | **Git** | GPL-2.0 | `git-scm.com` |
| 9 | **Python** | PSF | `python.org` |

All free. All open-source. Nothing here needs a licence, purchase, or account.

---

## 6. Verify what you download

Standard practice, and it matters given your concern:

```powershell
# check a downloaded archive against its published hash
Get-FileHash .\samloader-rs.exe -Algorithm SHA256
```

Compare against the checksum published in the project's GitHub release. Where a project
publishes signed releases, prefer those.

```bash
# on Linux/WSL
sha256sum file
sha256sum -c checksums.txt
```

---

## 7. Post-root modules — all open-source

| Module | License | Purpose |
|---|---|---|
| **PlayIntegrityFix** (`Kdrag0n`) | Open source | Restore Play Integrity on GSM variants |
| **LSPosed** | GPL | Xposed framework |
| **Shamiko** | Open source | Hide root from apps |
| **Debloater** | Open source | Remove bloatware |

Install **only** from official repositories. Fake "root hiding" modules are a real malware
vector — and would defeat the purpose of rooting safely.

---

Next: back to [README](../README.md) · [DETAILED-GUIDE.md](DETAILED-GUIDE.md)