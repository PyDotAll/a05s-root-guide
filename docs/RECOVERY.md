# Recovery — SM-A057F

Everything here assumes you have **stock firmware for your exact build** on a PC with Odin.
**Download it before you start rooting.** Without it, recovery is guesswork.

---

## Recovery paths, best to worst

| Severity | Symptom | Path |
|---|---|---|
| 🟢 Soft | Boots fine, Magisk not installed | Reflash patched image ([TROUBLESHOOTING](TROUBLESHOOTING.md)) |
| 🟡 Medium | `ODIN MODE (AVB Fail)` | Flash stock vbmeta from same build |
| 🟠 Hard | Download Mode only, won't boot | Reflash stock firmware |
| 🔴 Bootloop | Samsung logo loop | Stock firmware via Odin |
| ⚫ Worst | Odin can't detect device | DFU mode, then Odin |

---

> **Interface note.** Recovery paths below describe the **Odin GUI**. With the open-source
> route, substitute `odin4 -b BL_x.tar.md5 -a AP_x.tar.md5 -c CP_x.tar.md5 -s CSC_x.tar.md5`
> — see [WINDOWS-11-TOOLS.md](WINDOWS-11-TOOLS.md). Always run `--check-only` first.
>
> **Whatever tool you use, keep stock firmware on the PC permanently.** Every recovery path
> depends on it.

---

## 🟢 Soft — boots, no root

Nothing is broken. Reflash the patched image:

1. Verify LZ4 magic (`04224d186c`)
2. Repack the tar
3. Odin: AP = patched, everything else empty, Auto Reboot **off**
4. `PASS!` → **Vol Down + Power**, then **Vol Up + Power**

---

## 🟡 `ODIN MODE (AVB Fail)`

Build and flash stock vbmeta from the **identical** firmware build:

```bash
7z e AP_x.tar.md5 vbmeta.img.lz4 vbmeta_samsung.img.lz4 -y
lz4 -d -f vbmeta.img.lz4 vbmeta.img
lz4 -d -f vbmeta_samsung.img.lz4 vbmeta_samsung.img
tar -cf stock_vbmeta.tar vbmeta.img.lz4 vbmeta_samsung.img.lz4
```

Odin: load `stock_vbmeta.tar` in **AP**, Auto Reboot **off** → `PASS!` → manual boot.

> Setting AVB flags to `03` often fails on newer MTK. Stock vbmeta from the same build is
> what actually works.

---

## 🟠 Hard — stuck in Download Mode

1. Download Mode: **Vol Up + Vol Down**, connect USB, **Vol Up**
2. Odin should show a **greyed** device
3. Flash **full stock**: `BL`, `AP`, `CP`, `CSC` (use `CSC` — accepts the wipe)
4. Auto Reboot **on**
5. `PASS!` → normal boot

The wipe is the point here: you're restoring a known-good state.

---

## 🔴 Bootloop at the logo

1. Force off: hold **Power + Vol Down** 10+ seconds
2. Download Mode as above
3. Flash **full stock firmware** — `BL`, `AP`, `CP`, `CSC`
4. Let it boot fully

If it still bootloops with **completely stock** firmware, the problem is not the root attempt —
suspect the firmware itself, or hardware.

---

## ⚫ Odin / odin4 doesn't detect the device

### DFU mode

1. Download Mode first (**Vol Up + Vol Down** + USB, **Vol Up**)
2. Screen black, no download UI
3. Press and hold **Vol Down + Vol Up**, then tap **Back** 3–4×, release keys

Odin will show:

```
Odin Protocol
Download Phone: OFF
```

That's DFU. Odin can flash from here.

### Still nothing

- Different USB **cable** (data-capable — the #1 cause)
- Different USB **port** (directly on the PC, no hub)
- Different **PC**
- Reinstall USB drivers — **Zadig** (GPL-2.0) for WinUSB, or Samsung's for Odin
- On another PC: install the **Android Bootloader Interface** driver
- Confirm the tool sees it: `odin4 -l` (Download Mode) or `adb devices`

---

## ⚠️ When root is truly unrecoverable

A **hard brick** (no download mode, no Odin, nothing) cannot be fixed by software. It needs
a service centre or authorised reflash of the eMMC/UB bootloader.

Practical warning: **flashing firmware that doesn't match your model/region is how phones
get hard-bricked.** This is why every doc here names `SM-A057F` / `<YOUR_FIRMWARE_BUILD>` and your
carrier's CSC explicitly — your build is carrier-branded (`<YOUR_CSC>`), so a
CSC mismatch can break VoLTE and mobile-data provisioning, not just region settings.

---

## What rooting does NOT break

Reassurance, since recovery anxiety is what stops people finishing:

- **A failed Magisk patch never bricks anything.** It's a file on your PC.
- **A failed Odin flash is recoverable.** Download Mode is very hard to kill.
- **Knox is the only truly irreversible thing** — and it's not a functional brick. The phone
  works; only Knox-dependent features are gone forever.
- Samsung stock firmware remains flashable indefinitely. **Keep it on your PC permanently.**

---

## Prevention checklist

- [ ] Stock firmware for your exact build **downloaded and extracted on the PC**
- [ ] Model verified `SM-A057F`
- [ ] Firmware `<YOUR_FIRMWARE_BUILD>` / patch `<YOUR_PATCH_DATE>`
- [ ] CSC confirmed from Settings (carrier-branded build)
- [ ] LZ4 magic verified `04224d186c`
- [ ] Second USB cable available
- [ ] Odin + Samsung drivers working **before** flashing anything

---

Next: [POST-ROOT.md](POST-ROOT.md) · [UNROOT.md](UNROOT.md)