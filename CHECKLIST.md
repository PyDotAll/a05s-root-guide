# Offline Checklist

**Keep this off the wipe.** Top to bottom. Don't skip the ⛔ gates.

---

## Tools — all open-source & free

Full audit: [docs/WINDOWS-11-TOOLS.md](docs/WINDOWS-11-TOOLS.md)

| Tool | License | For |
|---|---|---|
| samloader-rs | Apache 2.0 | firmware download + flash |
| odin4 | Open source | Thor-protocol flash |
| Zadig | GPL-2.0 | WinUSB driver |
| Magisk | Apache 2.0 | root |
| platform-tools | Apache 2.0 | adb |
| 7-Zip | LGPL | open `.tar.md5` |
| lz4 | BSD-2 | LZ4 recompress |
| Git | GPL-2.0 | build from source |

⚠️ **Classic Odin3 is proprietary leaked software** — not open-source, from mirrors of unknown
trustworthiness. Open-source flashing tools are less battle-tested; accept that trade-off.

---

## ⛔ Gate — check before starting anything

- [ ] **Knox warranty void reads `0x0`** (Settings → About phone)
      If not `0x0`, it's already tripped and root costs you nothing extra.
      If it IS `0x0` and you use banking → **seriously reconsider**, Knox never resets.
- [ ] **OEM unlocking** toggle present and turns ON
      Absent = carrier-locked = **not rootable**. Stop.
- [ ] Windows PC available
- [ ] Data-capable USB cable (test it!)
- [ ] ~XXX GB free on PC
- [ ] Second USB cable as backup
- [ ] Serial + IMEI written down **off the phone** (password manager / paper)
      *Needed for firmware lookup. Not stored in this repo — see DEVICE-SPECIFICATIONS.md.*

---

## 1. Preparation

- [ ] Samsung USB drivers installed
- [ ] Odin 3.13.x+ installed
- [ ] Android platform-tools (`adb`) available
- [ ] Frija available
- [ ] Phone charged >60%

## 2. Update first

- [ ] Settings → Software update → downloaded and installed
- [ ] New **build number** recorded: `________________`
- [ ] New **patch level** recorded: `________________`

> Do this BEFORE rooting. OTA dies once you root.

## 3. Unlock

- [ ] Developer options ON (Build number ×7)
- [ ] OEM unlocking ON
- [ ] USB debugging ON
- [ ] Noted 7-day KG timer if present: `________`

## 4. Back up

- [ ] Google Photos / Drive sync **verified complete**
- [ ] 2FA seeds exported
- [ ] Irreplaceable files copied off device
- [ ] Banking credentials recoverable

At risk: **XXX GB** total · Pictures XXX · DCIM XXX · Download XXX · XXX.XX GB shared storage

## 5. Firmware

- [ ] Model = `SM-A057F`
- [ ] Firmware = `<YOUR_FIRMWARE_BUILD>` (or newer)
- [ ] Patch = `<YOUR_PATCH_DATE>` (or newer)
- [ ] **CSC confirmed in Settings** — carrier-branded build `<YOUR_CSC>`
- [ ] Build matches step 2
- [ ] Downloaded (~XXX GB — use WiFi, not cellular)
- [ ] Extracted: `BL`, `AP`, `CP`, `CSC`, `HOME_CSC`
- [ ] **Kept on PC permanently** for recovery

## 6. Extract boot image

- [ ] `7z e AP_x.tar.md5 init_boot.img.lz4 -y`
- [ ] `lz4 -d -f init_boot.img.lz4 init_boot.img`
- [ ] Size is tens of MB, not a few KB ⛔

## 7. Patch

- [ ] Magisk installed (`Download/Magisk-v30.7.apk`)
- [ ] Install → **Select and Patch a File** → `init_boot.img`
- [ ] `magisk_patched-*.img` created, tens of MB ⛔
- [ ] Repacked: `lz4 -B6 --content-size init_boot.img init_boot.img.lz4`
- [ ] `xxd -l 8 -p` → starts `04224d186c` ⛔
- [ ] `tar -cf magisk_patched.tar init_boot.img.lz4`
- [ ] `tar -tf magisk_patched.tar` lists the file

## 8. Flash

- [ ] Phone powered off
- [ ] Download Mode: **Vol Up + Vol Down** + USB → **Vol Up**
- [ ] Odin as Administrator, `ID:COM` **blue**
- [ ] AP = `magisk_patched.tar`
- [ ] BL / CP / CSC **empty**
- [ ] **Auto Reboot UNCHECKED** ⛔
- [ ] F. Reset Time ✓ · Re-Partition ☐ · Nand Erase ☐ ⛔
- [ ] Start → **`PASS!`**
- [ ] Cable not touched during write

## 9. Boot patched image

- [ ] **Vol Down + Power** → screen off
- [ ] **Vol Up + Power** → hold to Samsung logo ⛔
  (A plain power press boots STOCK — Magisk won't load)
- [ ] First boot completed (several minutes)
- [ ] Magisk shows **Installed**

## 10. Verify

- [ ] `adb shell su` → uid=0
- [ ] `adb shell magisk -V` → version
- [ ] `getprop ro.build.version.release` → 15
- [ ] Magisk app → Installed

## 11. After

- [ ] Set a monthly reminder — **OTA is disabled**, update via Odin only
- [ ] Read [docs/POST-ROOT.md](docs/POST-ROOT.md) re banking apps
- [ ] Only install modules from official repos

---

## Emergency card

| Situation | Do |
|---|---|
| Magisk shows `Not Installed` | Reboot holding **Vol Up + Power** |
| Stuck `File Analysis` | Patch `init_boot.img` alone, not the AP |
| `FAIL! LZ4 is invalid` | Used plain `lz4` — needs `--content-size` |
| `ODIN MODE (AVB Fail)` | Flash stock `vbmeta` from the **same build** |
| Odin shows no device | Different cable → different port → different PC |
| Bootloop | Flash full stock firmware via Odin |
| Completely dead | Service centre / authorised reflash |

Full detail: [docs/RECOVERY.md](docs/RECOVERY.md)

---

## Quick reference

```bash
# LZ4 magic check — MUST start 04224d186c
xxd -l 8 -p init_boot.img.lz4

# extract from AP
7z e AP_x.tar.md5 init_boot.img.lz4 -y

# decompress / recompress
lz4 -d -f init_boot.img.lz4 init_boot.img
lz4 -B6 --content-size init_boot.img init_boot.img.lz4

# pack for Odin
tar -cf magisk_patched.tar init_boot.img.lz4

# verify root
adb shell su
adb shell magisk -V
```
