# Rooting Samsung Galaxy A05s (SM-A057F) — End-to-End Guide

> Survives a factory reset. Everything needed is here.

---

## 0. Device facts (verified)

| Property | Value |
|---|---|
| Model | **Samsung Galaxy A05s, SM-A057F/DS** |
| Variant | **International** (has OEM-unlock toggle) — the rootable one |
| Chipset | MediaTek Helio G80 (MT6765), octa-core, aarch64 |
| Build fingerprint | `<YOUR_BUILD_FINGERPRINT>` |
| OS | Android 15 / One UI 7 (originally shipped Android 13) |
| Security patch | **<EARLIER_PATCH_DATE>** ⚠️ see [Step 1](#step-1--update-via-ota-first) |
| SELinux | **Enforcing** |
| Bootloader | Unlockable (international variant) |
| TWRP / fastboot | **Neither works.** Odin is the only route. |

Fingerprint decode:

```
AP3A      -> Android 15 platform
240905    -> security patch <EARLIER_PATCH_DATE>
A2A057F   -> model A057F
SBDYJ1    -> region BD (Bangladesh)
SEPF_SM-A057F_13_0001 -> SM-A057F, first shipped Android 13
```

---

## ⚠️ Read before starting

- **A Windows PC and USB cable are mandatory.** No PC = no Odin = no root. There is no Termux-only path on this device.
- **Knox trips permanently.** Irreversible, even by reflashing stock firmware. Samsung Pay, Secure Folder, and most banking apps stop working forever.
- **OTA updates die after rooting.** You will only be able to update by reflashing firmware manually via Odin.
- **Backup everything first.** See [Step 3](#step-3--back-up-everything).

### 🚫 Never use these

**KingRoot**, **KingoRoot**, and any "root without PC" app are documented malware / PUP vectors. Many are trojans that grant real root and keep it. Avoid them.

Only use **Magisk** from the official source:
`https://github.com/topjohnwu/Magisk/releases` (verify the release page exists and is the official repo).

---

## Step 0 — Prerequisites

- [ ] Windows PC (Win10/11)
- [ ] USB cable (data-capable, not charge-only)
- [ ] Samsung USB drivers installed on the PC
- [ ] Odin **3.13.x or newer** on the PC
- [ ] `Frija` (queries Samsung's official FUS server) **or** access to samfw.com
- [ ] `adb` (Android platform-tools) — on PC or on the phone via Termux
- [ ] ~20 GB free on the PC (the AP firmware is ~7 GB)

---

## Step 1 — Update via OTA FIRST

Your security patch is **<EARLIER_PATCH_DATE>**. Samsung has shipped **2026-05-05** firmware for this model — you are roughly 21 months behind on a device that still receives updates.

**Update before rooting.** Settings → Software update → Download and install.

Why this ordering matters:
1. Once rooted, **OTA is disabled** — this is your last easy update.
2. You want to patch a *current* boot image, not a 2024 one.
3. A stale patch on a current OS is worth ruling out on its own.

**Write down the new build number** — Step 4 requires firmware matching it exactly.

---

## Step 2 — The go/no-go gate

Settings → About phone → tap **Build number** 7 times → Developer options.

| State | Action |
|---|---|
| **OEM unlocking** toggle present and turns ON | ✅ Go |
| Present but OFF | Turn it ON now |
| Greyed out, "Connect to internet…" | Reconnect mobile data, retry |
| Missing entirely | ❌ Carrier-locked variant — **cannot be rooted**, stop |

Also enable **USB debugging** — Step 5 uses it.

> Seeing the OEM unlocking toggle proves Developer options is already enabled (that screen only exists once it is).

### Expected: 7-day lock period

Samsung enforces a **7-day wait** on new unlocks. You will see `KG status: Checking` in Download Mode and flashing will be refused until the timer expires.

You can do **all** the prep (firmware download, Magisk patching) while it counts down. Check Settings → About phone for the timer.

---

## Step 3 — Back up everything

`/data` is 109 GB with ~52 GB used. Verified contents of shared storage at time of writing:

```
Personal storage inventory removed
```

- Confirm Google Photos / Google Drive sync is **complete**
- Copy anything irreplaceable off the device
- Export 2FA seeds from your authenticator app (Knox-sensitive apps break)
- Note: flashing `CSC` wipes the device. `HOME_CSC` preserves data. The AP-only route (Step 5) preserves data.

---

## Step 4 — Obtain matching stock firmware

You need firmware matching **all three**: model `SM-A057F`, your **CSC region** (`BD`), and your **exact current build** (from Step 1).

**Option A — Frija (official Samsung FUS server):**
1. Model: `SM-A057F`
2. CSC: `BD`
3. IMEI: dial `*#06#`
4. Check Update → Download

**Option B — SamFW:** `https://samfw.com/firmware/SM-A057F`

After unzipping you have 4–5 files:

| File | Purpose | Size |
|---|---|---|
| `BL_*.tar.md5` | Bootloader | small |
| `AP_*.tar.md5` | **System / boot / init_boot / vbmeta** | **~7 GB** |
| `CP_*.tar.md5` | Modem | medium |
| `CSC_*.tar.md5` | Carrier config — **wipes data** | small |
| `HOME_CSC_*.tar.md5` | Carrier config — **preserves data** | small |

> ⚠️ **Region mismatch can be refused by the bootloader.** Never guess the CSC.

---

## Step 5 — Patch with Magisk

### ⚠️ Known bug you must avoid

**Magisk 27.0 and newer produce a CORRUPT patched AP tar.** Odin hangs at "File Analysis", or errors `Unexpected end of archive`. The patched tar gets an AVB footer that puts Odin in a dead loop.

Reference: [topjohnwu/Magisk#8735](https://github.com/topjohnwu/Magisk/issues/8735)

**Therefore: do NOT patch the whole `AP` file.** Patch the individual boot images instead.

### Procedure

1. Open `AP_*.tar.md5` with 7-Zip (it is a plain tar; ignore the trailing MD5).
2. Extract these three:
   ```
   boot.img.lz4
   init_boot.img.lz4
   vbmeta.img.lz4
   ```
   > The A05s is System-as-Root — **`init_boot.img` is the correct target**, not `boot.img`.

3. Decompress with LZ4:
   ```bash
   lz4 -d init_boot.img.lz4 init_boot.img
   ```

4. Copy `init_boot.img` to the phone (via adb or directly).

5. On the phone, Magisk → **Install → Select and Patch a File** → pick `init_boot.img`.
   Output: `magisk_patched-<timestamp>.img` in the same folder.

6. Pull it back to the PC and repack for Odin.

### 🔑 The LZ4 gotcha

If you recompress, LZ4 **must** be built with `--content-size`:

```bash
lz4 -B6 --content-size init_boot.img init_boot.img.lz4
```

Verify the magic bytes — they **must start `04 22 4d 18 6c`**:

```bash
xxd -l 8 init_boot.img.lz4
# 04224d18 6c ...   ← the 6c is the --content-size flag
```

Plain `lz4` omits it and Odin fails with `FAIL! LZ4 is invalid`.

Then tar it:
```bash
tar -cf magisk_patched.tar init_boot.img.lz4
```

---

## Step 6 — Flash with Odin

1. Power off the phone.
2. **Download Mode:** hold **Volume Up + Volume Down**, connect USB, then press **Volume Up** to continue past the warning.
3. Open **Odin as Administrator**. `ID:COM` box turns **blue** (grey = install Samsung USB drivers).
4. Load files:

   | Odin slot | File | Notes |
   |---|---|---|
   | **AP** | your `magisk_patched.tar` | the patched image |
   | **BL** | *empty* | unless unlocking bootloader separately |
   | **CP** | *empty* | |
   | **CSC** | *empty* **or** `HOME_CSC` | empty + AP-only = **no wipe** |

5. **Options tab:** uncheck **Auto Reboot**. Leave `F. Reset Time` ticked.
   **Never tick `Re-Partition` or `Nand Erase`.**
6. **Start** → wait for green **`PASS!`**. Do not unplug. The AP write looks frozen for minutes; that is normal.

---

## Step 7 — Boot manually into the patched image

⚠️ **Do not let Odin auto-reboot — you set Auto Reboot OFF for this reason.**

1. Press **Volume Down + Power** until the screen turns off.
2. Immediately hold **Volume Up + Power** until the Samsung logo.
   (Vol Up forces boot into the patched image; a normal power press boots the stock image and Magisk will not load.)
3. First boot after a flash takes several minutes.
4. Open Magisk → it should show **Installed** and offer a reboot to finalise.
5. Verify: `adb shell su` → should grant a root shell.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Odin stuck at "File Analysis" | Magisk 27+ corrupted AP patch | Patch `init_boot.img` individually (Step 5) |
| `Unexpected end of archive` | same as above | same as above |
| `FAIL! LZ4 is invalid` | `.lz4` missing content-size header | Recompress with `lz4 -B6 --content-size`; magic must start `04 22 4d 18 6c` |
| Boots to **ODIN MODE (AVB Fail) / CUSTOM VBMETA** | AVB rejecting modified image | Flash **stock `vbmeta.img` + `vbmeta_samsung.img` from the exact same firmware** |
| AVB still failing after patching vbmeta | Setting flags to `03` is insufficient on newer MTK | Use the **stock** vbmeta from the identical build, not a hand-patched one |
| `KG status: Checking` | Samsung 7-day unlock wait | Wait out the timer |
| No OEM-unlock toggle | Carrier-locked variant | Not rootable |
| Odin `ID:COM` stays grey | Missing USB driver | Install Samsung USB drivers |
| Magisk patched but no root | Booted stock image instead of patched | Reboot holding **Vol Up + Power** |
| Device bootloops | Wrong image / wrong firmware build | Reflash stock firmware via Odin; never improvise |

---

## Unroot / restore

- Magisk → **Uninstall** (restores stock boot image)
- Or reflash the original stock firmware via Odin
- **Knox stays tripped. It never resets.**

---

## After rooting — verification

```bash
adb shell su                      # expect a root shell
adb shell magisk -V              # expect a version number
adb shell getprop ro.build.version.release
```

Root enables the forensics that were previously impossible on this device:

```bash
# live socket table — was SELinux-blocked without root
cat /proc/net/tcp
cat /proc/net/tcp6

# installed packages
pm list packages -3

# per-process file + socket handles
lsof -nP -i

# traffic counters
cat /proc/net/dev

# routing + neighbour tables
ip route; ip neigh
```

---

## Sources

- [Magisk official releases](https://github.com/topjohnwu/Magisk/releases)
- [Magisk issue #8735 — patching Samsung AP corruption](https://github.com/topjohnwu/Magisk/issues/8735)
- [SamFW firmware archive](https://samfw.com/firmware/SM-A057F)
- [SamMobile firmware archive](https://www.sammobile.com/samsung/galaxy-a05s/)
- [XDA — A05s root thread](https://xdaforums.com/t/request-root-method-for-galaxy-a05s-sm-a057f-ds-one-ui-7-android-15.4755956/)
- [XDA — Magisk on newer Samsung firmware](https://xdaforums.com/t/magisk-samsung-a05s.4763159/)
- [sanjarbiy/samsung-a05s-pixelos-gsi](https://github.com/sanjarbiy/samsung-a05s-pixelos-gsi) — documents that TWRP and fastboot do not work; Odin-only