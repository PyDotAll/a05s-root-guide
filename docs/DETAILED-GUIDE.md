# Detailed Guide — SM-A057F

Every step, with the **expected result** at each checkpoint so you know when it's worked.
Times are approximate for a novice on a mid-range PC.

---

## Table of contents

- [1. Preflight](#1-preflight)
- [2. OTA update](#2-ota-update)
- [3. Unlock verification](#3-unlock-verification)
- [4. Backup](#4-backup)
- [5. Firmware acquisition](#5-firmware-acquisition)
- [6. Patch with Magisk](#6-patch-with-magisk)
- [7. Flash with Odin](#7-flash-with-odin)
- [8. Manual boot](#8-manual-boot)
- [9. Verification](#9-verification)
- [Command reference](#command-reference)

---

## 1. Preflight

### 1.1 PC setup

Install, in this order:

1. **Samsung USB drivers** — without these, Odin's `ID:COM` stays grey
2. **Odin v3.13.x or newer** (run as Administrator)
3. **Android platform-tools** (`adb`) — on the PC, or on the phone via Termux
4. **Frija** — for firmware retrieval from Samsung's official server

**Free disk:** ~25 GB. The `AP` file alone is ~7 GB.

### 1.2 On the phone

- Charge above 60%
- Developer options ON
- **OEM unlocking** ON
- **USB debugging** ON

### 1.3 Cable

Test it. A charge-only cable gives you *nothing* — no Download Mode, no Odin, no adb.
Cheapest possible failure: try a second cable before assuming a software problem.

---

## 2. OTA update

> Settings → Software update → Download and install

**Checkpoint:** Settings → About phone → Software information. Note the new **Build number**.
Write it down. Step 5 requires matching firmware.

Why first, not last:

- OTA is permanently disabled once rooted
- You want to patch a *current* `init_boot`, not a 2024 one
- ~21 months of missing patches is worth closing regardless

---

## 3. Unlock verification

> Settings → Developer options → **OEM unlocking**

| State | Meaning |
|---|---|
| Toggles ON | ✅ International variant. Proceed. |
| OFF | Turn ON, continue |
| Greyed + "Connect to internet…" | Enable mobile data, retry |
| Absent | ❌ Carrier-locked. Not rootable. Stop. |

### 3.1 The 7-day KG lock

Samsung blocks new unlocks for 7 days. You'll see `KG status: Checking` in Download Mode.

Check the timer: Settings → About phone. **You can prepare everything else while waiting** —
firmware download and Magisk patching don't require the unlock.

---

## 4. Backup

Your shared storage currently holds:

```
Personal file inventory removed
```

- [ ] Google Photos / Drive sync **complete** — verify, don't assume
- [ ] 2FA seeds exported (Knox breaks some authenticators)
- [ ] Irreplaceable files copied off-device
- [ ] Banking app credentials recoverable (PIN, not just biometrics)

### 4.1 What each flashing route costs you

| Route | Data | Time |
|---|---|---|
| AP only, CSC empty | ✅ **Preserved** | 5 min |
| AP + `HOME_CSC` | ✅ **Preserved** | 5 min |
| AP + `CSC` | ❌ **Full wipe** | 10 min setup after |

AP-only is enough to root. Start there — it costs nothing and preserves everything.

---

## 5. Firmware acquisition

### 5.1 Retrieve

**Frija** → Manual tab:

| Field | Value |
|---|---|
| Model | `SM-A057F` |
| CSC | `BD` |
| IMEI | dial `*#06#` on the phone |

Check Update → Download. Get the build matching Step 2's build number.

**Alternative:** <https://samfw.com/firmware/SM-A057F>

### 5.2 Extract

Unzip. You get 4–5 `.tar.md5` files. They're plain tars with a trailing MD5.

You only need `AP_*.tar.md5`.

> ⚠️ **CSC mismatch** can be refused by the bootloader or alter carrier config. Never guess.

---

## 6. Patch with Magisk

### 6.1 The known bug — read this

**Magisk 27.0+ cannot patch a full Samsung `AP` tar.** Output gets an AVB footer; Odin hangs at `File Analysis` or throws `Unexpected end of archive`.

[topjohnwu/Magisk#8735](https://github.com/topjohnwu/Magisk/issues/8735)

**Patch the individual boot image instead.**

### 6.2 Which image

The A05s is **System-as-Root** → patch **`init_boot.img`**, not `boot.img`.

If your firmware has no `init_boot.img.lz4`, fall back to `boot.img.lz4`.

### 6.3 Extract from the AP

**On the PC (Windows):**

1. Right-click `AP_*.tar.md5` → **7-Zip** → Open
2. Navigate to `init_boot.img.lz4`
3. Drag it out

**On the phone (Termux, already set up):**

```bash
ap=/path/to/AP_A057FXXXXXXX.tar.md5
mkdir -p ~/work && cd ~/work
7z e "$ap" init_boot.img.lz4 vbmeta.img.lz4 -y
lz4 -d -f init_boot.img.lz4 init_boot.img
ls -la init_boot.img          # should be ~ tens of MB
```

**Checkpoint:** `init_boot.img` exists and is tens of MB. If it's a few KB, you grabbed the wrong file.

### 6.4 Patch

1. Copy `init_boot.img` to the phone
2. Magisk → **Install** → **Select and Patch a File**
3. Choose `init_boot.img`
4. Wait — it writes `magisk_patched-<timestamp>.img` alongside the source

**Checkpoint:** `magisk_patched-*.img` exists, tens of MB, not 0 bytes.

### 6.5 Repack for Odin

```bash
lz4 -B6 --content-size init_boot.img init_boot.img.lz4
xxd -l 8 -p init_boot.img.lz4
```

Must output starting `04224d186c`:

```
04224d186c50400d...   ✅ content-size present
04224d1864500840...   ❌ missing content-size — Odin will say "LZ4 is invalid"
```

Then:

```bash
tar -cf magisk_patched.tar init_boot.img.lz4
```

**Checkpoint:** `magisk_patched.tar` exists.

*(All tools verified present on your device — see [TOOLS.md](TOOLS.md).)*

---

## 7. Flash with Odin

### 7.1 Download Mode

1. Power off completely
2. Hold **Vol Up + Vol Down**
3. Connect USB
4. Release; a warning screen appears
5. Press **Vol Up** to continue → green Download Mode screen

### 7.2 Odin

1. Open Odin **as Administrator**
2. `ID:COM` → **blue** = connected. Grey = driver problem.
3. Load slots:

| Slot | Load |
|---|---|
| **AP** | `magisk_patched.tar` |
| **BL** | *empty* |
| **CP** | *empty* |
| **CSC** | *empty* — or `HOME_CSC` to be explicit about preserving data |

### 7.3 Options tab

| Setting | State |
|---|---|
| Auto Reboot | ☐ **UNCHECK — critical** |
| F. Reset Time | ☑ checked |
| Re-Partition | ☐ never |
| Nand Erase | ☐ never |

### 7.4 Flash

Click **Start**. Expect:

1. `super.img` / partition writes — **log looks frozen for several minutes. This is normal.**
2. Green **`PASS!`**
3. Device does **not** reboot (Auto Reboot is off)

**Do not unplug during the write.** Interrupting can brick.

---

## 8. Manual boot

⚠️ **A normal power press boots the STOCK image. Magisk will not load.** Vol Up is what forces the patched image.

1. **Vol Down + Power** → screen off
2. Immediately **Vol Up + Power** → hold through to the Samsung logo
3. First boot: several minutes
4. Open Magisk → **`Installed`**

If Magisk shows `Not Installed`, you booted the wrong image — repeat from step 1.

---

## 9. Verification

```bash
adb shell su
# expect: root shell, e.g. uid=0(root)

adb shell magisk -V
# expect: a version number

adb shell getprop ro.build.version.release
# expect: 15
```

Magisk app must read **Installed**. All three agree = success.

---

## Command reference

```bash
# --- verify LZ4 magic (must be 6c) ---
xxd -l 8 -p init_boot.img.lz4

# --- extract from AP ---
7z e AP_x.tar.md5 init_boot.img.lz4 -y

# --- decompress / recompress ---
lz4 -d -f init_boot.img.lz4 init_boot.img
lz4 -B6 --content-size init_boot.img init_boot.img.lz4

# --- pack for Odin ---
tar -cf magisk_patched.tar init_boot.img.lz4

# --- device-side ---
adb devices
adb reboot download
adb push init_boot.img /sdcard/Download/
adb pull /sdcard/Download/magisk_patched-*.img .
adb shell su
adb shell magisk -V
```

---

## Time estimate

| Step | Time |
|---|---|
| OTA update | 20–40 min |
| Backup | 1–3 hours |
| Firmware download (7 GB on mobile data — use WiFi) | 30–60 min |
| Extract + patch | 20–30 min |
| Flash | 10 min |
| First boot | 10 min |

**Firmware over mobile data will consume most of a data allowance.** Use WiFi. Your cellular
data is on `rmnet_data0` — a 7 GB download is substantial.

---

Next: [TROUBLESHOOTING.md](TROUBLESHOOTING.md) · [RECOVERY.md](RECOVERY.md)