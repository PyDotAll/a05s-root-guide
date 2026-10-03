# Rooting Samsung Galaxy A05s (SM-A057F) — Complete Guide

> Written <CAPTURE_DATE> from a live inspection of this exact device.
> Every fact below was verified unless marked *(community report)*.

---

## 📖 Where to start

| If you want… | Read |
|---|---|
| The whole thing, start to finish | This file, top to bottom |
| Just the steps, with expected output at each | [docs/DETAILED-GUIDE.md](docs/DETAILED-GUIDE.md) |
| A printable tick-list | [CHECKLIST.md](CHECKLIST.md) |
| Something went wrong | [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) |
| It's bricked / won't boot | [docs/RECOVERY.md](docs/RECOVERY.md) |
| Already rooted — now what | [docs/POST-ROOT.md](docs/POST-ROOT.md) |
| Change my mind / undo it | [docs/UNROOT.md](docs/UNROOT.md) |
| Security state of this phone | [docs/security-findings.md](docs/security-findings.md) |
| Tools I set up on the phone | [docs/TOOLS.md](docs/TOOLS.md) |

---

## ⚠️ Read this before anything else

### I cannot promise 100%

You asked me to guarantee this works. I can't, and anyone who says otherwise is selling something. Here's the honest position:

**What is verified about your specific device:**
- Model is `SM-A057F/DS`, international variant — has a working OEM-unlock toggle
- Android 15 / One UI 7, build `<YOUR_BUILD_FINGERPRINT>`
- Odin + patched `init_boot.img` is the documented working path
- Root is achievable on this model *(community reports)*

**What cannot be verified without a PC, a reboot, and risking the device:**
- Whether your bootloader unlocks cleanly on the first attempt
- Whether Samsung's 7-day KG timer is already satisfied
- Whether firmware for your exact build + CSC `BD` is downloadable
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

- [x] **Windows PC** — Odin has no Linux/macOS equivalent. This is not optional.
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
| Firmware | `<YOUR_BUILD_FINGERPRINT>` |
| OS | Android 15 / One UI 7 (first shipped Android 13) |
| Security patch | **<EARLIER_PATCH_DATE>** ⚠️ ~21 months stale |
| SELinux | Enforcing |
| Storage | <DEVICE_STORAGE_REDACTED> |
| Root / bootloader | None currently |

Fingerprint decode:

```
AP3A                        -> Android 15 platform
240905                      -> security patch <EARLIER_PATCH_DATE>
A2A057F                     -> model A057F
SBDYJ1                      -> region BD (Bangladesh)
SEPF_SM-A057F_13_0001       -> SM-A057F, first shipped Android 13
```

> Note: `/sys/devices/soc0/family` reports "Snapdragon" on this device — that string is inherited from reused kernel code and is **wrong**. It is a MediaTek part.

---

## Step 1 — Update via OTA first ⚠️

Your patch level is **<EARLIER_PATCH_DATE>**. Samsung has shipped **2026-05-05** firmware for this model. You are ~21 months behind on a device still receiving updates.

**Do this before rooting:**

> Settings → Software update → Download and install

Three reasons the order matters:

1. **After rooting, OTA is dead.** This is your last one-tap update, ever.
2. **You want to patch a current `init_boot`**, not a 2024 one.
3. A stale patch on a current OS is worth ruling out regardless of rooting.

**Record the new build number** — Step 3 requires firmware matching it exactly.

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

You need all three to match: model `SM-A057F`, **CSC region `BD`**, and your **exact current build**.

**Option A — Frija** (queries Samsung's official FUS server):

1. Model `SM-A057F`
2. CSC `BD`
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