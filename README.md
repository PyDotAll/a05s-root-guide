# Rooting Samsung Galaxy A05s (SM-A057F) — Complete Guide

> Written <CAPTURE_DATE> from a live inspection of this exact device.
> Every fact below was verified unless marked *(community report)*.

---

## 📖 Where to start

| If you want… | Read |
|---|---|
| **Your exact device specs first** | **[docs/DEVICE-SPECIFICATIONS.md](docs/DEVICE-SPECIFICATIONS.md)** |
| The whole thing, start to finish | This file, top to bottom |
| Just the steps, with expected output at each | [docs/DETAILED-GUIDE.md](docs/DETAILED-GUIDE.md) |
| A printable tick-list | [CHECKLIST.md](CHECKLIST.md) |
| Something went wrong | [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) |
| It's bricked / won't boot | [docs/RECOVERY.md](docs/RECOVERY.md) |
| Already rooted — now what | [docs/POST-ROOT.md](docs/POST-ROOT.md) |
| Change my mind / undo it | [docs/UNROOT.md](docs/UNROOT.md) |
| Security state of this phone | [docs/security-findings.md](docs/security-findings.md) |
| **Windows 11 tools (all open-source/free)** | **[docs/WINDOWS-11-TOOLS.md](docs/WINDOWS-11-TOOLS.md)** |
| Tools I set up on the phone | [docs/TOOLS.md](docs/TOOLS.md) |

---

## ⚠️ Read this before anything else

### I cannot promise 100%

You asked me to guarantee this works. I can't, and anyone who says otherwise is selling something. Here's the honest position:

**What is verified about your specific device:**
- Model is `SM-A057F/DS`, international variant — has a working OEM-unlock toggle
- Android 15 / One UI 7, firmware `<YOUR_FIRMWARE_BUILD>`, patch **<YOUR_PATCH_DATE>**
- Odin + patched `init_boot.img` is the documented working path
- Root is achievable on this model *(community reports)*

**What cannot be verified without a PC, a reboot, and risking the device:**
- Whether your bootloader unlocks cleanly on the first attempt
- Whether Samsung's 7-day KG timer is already satisfied
- Whether firmware for `<YOUR_FIRMWARE_BUILD>` + your carrier CSC is downloadable
- Whether AVB will accept the patched image on your firmware revision

Realistically this is a **high-confidence** procedure with normal failure modes, not a certainty. The failure modes are all recoverable — see [docs/RECOVERY.md](docs/RECOVERY.md) — but the process can cost you an afternoon and a factory reset.

### Irreversible consequences

| Consequence | Detail |
|---|---|
| **Knox trips — permanently** | Never resets, even by reflashing stock. Samsung Pay, Secure Folder, and most banking/DRM apps stop working forever. |
| **OTA updates disabled** | You will only be able to update by manually reflashin via Odin. |
| **Warranty** | Samsung will deny service for an unlocked bootloader. |
| **Data** | 52 GB of your data is at risk. Flash `CSC` to wipe; `HOME_CSC` or AP-only preserves. |

### Hard requirements

- [x] **Windows PC** — or WSL2, or a Linux/macOS machine
      (Odin3 itself is proprietary leaked software; see [docs/WINDOWS-11-TOOLS.md](docs/WINDOWS-11-TOOLS.md) for the
      fully open-source toolchain)
- [x] **Data-capable USB cable** — not a charge-only cable
- [ ] ~25 GB free on the PC (the `AP` firmware is ~7 GB, plus extracted images)
- [ ] Phone charged above 60%

### 🚫 Never install these

**KingRoot**, **KingoRoot**, **iRoot**, and any "root without PC" app are documented malware / PUP vectors. Many ship a real root backdoor and keep it permanently — which would directly defeat the purpose of rooting safely. Only use **Magisk** from `github.com/topjohnwu/Magisk`.

---

## Device facts (verified on your phone)

| Property | Value |
|---|---|
| Model | **Samsung Galaxy A05s SM-A057F/DS** |
| Variant | International — the unlockable one |
| Chipset | MediaTek Helio G80 (MT6765), octa-core, aarch64 |
| **Kernel** | **`<YOUR_KERNEL_VERSION>`** |
| **Firmware** | **`<YOUR_FIRMWARE_BUILD>`** |
| OS | Android 15 / One UI 7 (first shipped Android 13) |
| **One UI** | **7.0** |
| **Baseband** | `<YOUR_FIRMWARE_BUILD>` |
| **Security patch** | **<YOUR_PATCH_DATE>** (SMR Nov-2025 R1, ASKS v9.4 20260111) |
| **Google Play System** | **<YOUR_PLAY_SYSTEM_DATE>** ← only ~2 months old |
| Service provider | `<YOUR_CSC>` — **carrier-branded** |
| SELinux | Enforcing |
| Storage | <DEVICE_STORAGE_REDACTED> |
| Root / bootloader | None currently |

Fingerprint decode:

```
<YOUR_BUILD_FINGERPRINT>
     ^^^^^^ AOSP AP branch date — NOT your patch level (see below)
AP3A      -> Android 15 platform
A2A057F   -> model A057F
SBDYJ1    -> firmware revision, matches Settings
SEPF_SM-A057F_13_0001 -> first shipped Android 13
```

> **⚠️ The fingerprint's `240905` is *not* your patch date.** Samsung ships the Google/AP
> patch and the Samsung Monthly Release on independent cadences, so they legitimately disagree.
> Your real patch level is **<YOUR_PATCH_DATE>**, from Settings — about **11 months** old.
> I initially misread this as <EARLIER_PATCH_DATE> / 21 months stale. That was wrong; see
> [DEVICE-SPECIFICATIONS.md](docs/DEVICE-SPECIFICATIONS.md#️-correction--i-had-the-patch-date-wrong).

> Note: `/sys/devices/soc0/family` reports "Snapdragon" on this device — that string is
> inherited from reused kernel code and is **wrong**. It is a MediaTek part.

### What the kernel tells us

`5.15.167-android13-8` means:

- **System-as-Root** → **`init_boot.img` is the correct patch target**, not `boot.img` ✓
- **5.15 LTS** → squarely within Magisk's supported range; no exotic kernel to work around
- **No GKI/generic-kernel complications** — Samsung ships its own kernel builds
- Kernel is the **Android 13 branch while the OS is Android 15** — normal Samsung behaviour,
  *not* a sign of a tampered build

---

## Step 1 — Update via OTA (worthwhile, not urgent) ⚠️

Your patch level is **<YOUR_PATCH_DATE>** — about 11 months behind. Real exposure, but modest, and
Samsung's monthly channel is still reaching this device.

**Do this before rooting:**

> Settings → Software update → Download and install

Two reasons:

1. **After rooting, OTA is disabled.** This is your last one-tap update, ever.
2. **You want to patch a current `init_boot`**, not a stale one.

⚠️ **Record the new build number AND patch date** — Step 4 requires firmware matching both.

---

## Step 2 — Confirm you can unlock

> Settings → About phone → **Build number** ×7 → Developer options

| What you see | Verdict |
|---|---|
| **OEM unlocking** toggle present, turns ON | ✅ proceed |
| Present but OFF | Turn it ON |
| Greyed, "Connect to internet…" | Reconnect mobile data, retry |
| **Missing entirely** | ❌ carrier-locked — cannot be rooted. Stop. |

Also enable **USB debugging** — Step 5 uses adb.

### Expect a 7-day lock

Samsung enforces a **7-day wait** on new unlocks. In Download Mode you'll see `KG status: Checking` and flashing is refused until it expires. Check Settings → About phone for the timer.

**You can do all preparation while it counts down** — firmware download and Magisk patching don't require the unlock.

> Seeing the OEM toggle confirms Developer options is already on — that screen only exists once enabled.

---

## Step 3 — Back up

Verified contents of your shared storage:

```
Personal file inventory removed
```

- Confirm Google Photos / Drive sync is **complete**, not in progress
- Copy irreplaceables off the device
- **Export 2FA seeds** — Knox-sensitive authenticator apps break after rooting
- Save any banking app credentials you can't re-verify

---

## Step 4 — Get matching firmware

You need model, **CSC**, and build to all match.

| Field | Value |
|---|---|
| Model | **`SM-A057F`** |
| Firmware / PDA | **`<YOUR_FIRMWARE_BUILD>`** |
| Patch level | **<YOUR_PATCH_DATE>** |
| Service provider | `<YOUR_CSC>` |

⚠️ **Your build is carrier-branded** (`SAOMC`). Flashing a different CSC can alter VoLTE,
carrier settings, network bands, and mobile-data provisioning. **Confirm your CSC in
Settings → About phone → Software information** — do not take my inference from the
fingerprint region field on trust.

**Option A — Frija** (queries Samsung's official FUS server):

1. Model `SM-A057F`
2. CSC — read it from Settings
3. IMEI — dial `*#06#`
4. Check Update → Download

**Option B — SamFW:** <https://samfw.com/firmware/SM-A057F>

You get four or five files:

| File | Contents | Size | Needed? |
|---|---|---|---|
| `BL_*.tar.md5` | Bootloader | small | Only if unlocking bootloader separately |
| `AP_*.tar.md5` | **system, init_boot, boot, vbmeta** | **~7 GB** | **Yes — this is the one you patch** |
| `CP_*.tar.md5` | Modem/radio | medium | No |
| `CSC_*.tar.md5` | Carrier config | small | Wipes device |
| `HOME_CSC_*.tar.md5` | Carrier config | small | Preserves data |

> ⚠️ **Wrong CSC can be refused by the bootloader, or change carrier config.** Never guess.

---

## Step 5 — Patch with Magisk

### ⚠️ The bug that wastes everyone's afternoon

**Magisk 27.0+ cannot patch a full Samsung `AP` tar.** The output gets an AVB footer and Odin either hangs at `File Analysis` or fails with `Unexpected end of archive`.

Reference: [topjohnwu/Magisk#8735](https://github.com/topjohnwu/Magisk/issues/8735)

**So patch the individual boot image, not the AP container.**

The A05s is **System-as-Root** — the target is **`init_boot.img`**, not `boot.img`.

Full procedure with commands: [docs/DETAILED-GUIDE.md §5](docs/DETAILED-GUIDE.md#5-patch-with-magisk)

Two paths:

| Path | How | Trade-off |
|---|---|---|
| **A (recommended)** | Extract `init_boot.img` from AP → patch with current Magisk → repack for Odin | Works with any Magisk version. More steps. |
| **B (legacy)** | Use Magisk **26.4** to patch the whole AP directly | Fewer steps, but requires deliberately downgrading Magisk. |

### 🔑 The LZ4 content-size gotcha

If you recompress the image, LZ4 **must** carry the content-size flag, or Odin fails with `FAIL! LZ4 is invalid`.

```bash
lz4 -B6 --content-size init_boot.img init_boot.img.lz4
```

Verify — the magic **must start `04 22 4d 18 6c`**:

```bash
xxd -l 8 -p init_boot.img.lz4
# 04224d186c...   <- 6c = content-size present  ✅
# 04224d1864...   <- 64 = content-size MISSING  ❌
```

*(Verified empirically on this device with `lz4` 1.10.0 — see [docs/TOOLS.md](docs/TOOLS.md).)*

---

## Step 6 — Flash with Odin

1. Power off
2. **Download Mode:** hold **Vol Up + Vol Down**, connect USB, press **Vol Up** to continue
3. Odin as **Administrator**; `ID:COM` turns **blue** (grey = install Samsung USB drivers)
4. Load slots per the table in [docs/DETAILED-GUIDE.md §6](docs/DETAILED-GUIDE.md#6-flash-with-odin)
5. **Uncheck Auto Reboot** — critical, see Step 7
6. Start → wait for green **`PASS!`**. The AP write looks frozen for minutes. **Do not unplug.**

---

## Step 7 — Boot into the patched image manually

⚠️ **A normal power-button press boots the STOCK image and Magisk will not load.** You must force it.

1. **Vol Down + Power** until the screen goes dark
2. Immediately **Vol Up + Power** until the Samsung logo (Vol Up = boot patched image)
3. First boot after a flash takes several minutes
4. Open Magisk → should show **Installed**
5. Verify: `adb shell su`

---

## Step 8 — Verify

```bash
adb shell su                  # expect a root shell
adb shell magisk -V          # expect a version number
adb shell getprop ro.build.version.release   # 15
```

Also open the **Magisk app** — it must read `Installed`, not `Not Installed`.

---

## What rooting unlocks

The forensics that were impossible on this device before:

```bash
cat /proc/net/tcp          # live socket table  (was SELinux-blocked)
cat /proc/net/dev          # per-interface traffic counters
pm list packages -3        # installed third-party apps
lsof -nP -i                # per-process sockets
ip route; ip neigh         # routing + neighbour tables
```

---

## Sources

- [Magisk releases](https://github.com/topjohnwu/Magisk/releases) — official only
- [Magisk#8735 — AP patch corruption](https://github.com/topjohnwu/Magisk/issues/8735)
- [SamFW](https://samfw.com/firmware/SM-A057F) · [SamMobile](https://www.sammobile.com/samsung/galaxy-a05s/)
- [XDA — A05s root thread](https://xdaforums.com/t/request-root-method-for-galaxy-a05s-sm-a057f-ds-one-ui-7-android-15.4755956/)
- [XDA — Magisk on newer Samsung firmware](https://xdaforums.com/t/magisk-samsung-a05s.4763159/)
- [sanjarbiy/samsung-a05s-pixelos-gsi](https://github.com/sanjarbiy/samsung-a05s-pixelos-gsi) — confirms **no TWRP, no fastboot**; Odin-only