# Troubleshooting — SM-A057F

Symptom-first. Find your symptom, apply the fix.

---

## During patching

### Magisk patching fails or produces a tiny/corrupt output

**Cause:** patching the whole `AP` container with Magisk 27+.
**Fix:** patch `init_boot.img` individually. See [DETAILED-GUIDE §6](DETAILED-GUIDE.md#6-patch-with-magisk).

### `init_boot.img` is only a few KB

**Cause:** you extracted the LZ4 wrapper, or grabbed the wrong member.
**Fix:** `lz4 -d -f init_boot.img.lz4 init_boot.img`. Expect tens of MB.

### Magisk says no `init_boot.img` in the AP

**Cause:** older firmware predates System-as-Root.
**Fix:** patch `boot.img` instead.

### Magisk hangs at "Extracting Boot.img"

**Cause:** low storage or a multi-GB file it can't stream.
**Fix:** free up at least 10 GB in `Download/`, retry.

---

## During LZ4 repack

### Odin: `FAIL! LZ4 is invalid`

**Cause:** LZ4 compressed without the content-size flag.
**Fix:**

```bash
lz4 -B6 --content-size init_boot.img init_boot.img.lz4
xxd -l 8 -p init_boot.img.lz4    # must start 04224d186c
```

`64` = missing flag ❌ · `6c` = correct ✅

### The tar opens but is empty

**Fix:** `tar -tf magisk_patched.tar` — should list `init_boot.img.lz4`.

---

## During flashing

### Odin `ID:COM` stays grey

**Cause:** missing/wrong USB driver, or a charge-only cable.
**Fix:** install Samsung USB drivers; try a different cable and a direct USB port (not a hub).

### Odin stuck at `File Analysis`

**Cause:** corrupt patched tar — almost always the Magisk 27+ AP bug.
**Fix:** repatch `init_boot.img` individually.

### `Unexpected end of archive`

Same cause and fix.

### `FAIL! Written header command : INVALIDATE`

**Fix:** the file wasn't a genuine Odin `.tar`. Rebuild with `tar -cf`.

### Odin disconnects mid-flash

**Cause:** cable, port, or power. Interrupting can brick.
**Fix:** use a short good cable, direct port, no hub, battery above 60%. Never interrupt.

### `KG status: Checking`

**Cause:** Samsung's 7-day unlock wait.
**Fix:** wait out the timer. All prep can be done meanwhile.

### `NAND WRITE FAIL!!`

**Cause:** usually cable/power, occasionally genuinely dead flash.
**Fix:** reseat cable, different port, retry. If it persists repeatedly, the storage may be
failing — back up immediately and expect a warranty claim.

---

## After flashing

### Device boots but Magisk shows `Not Installed`

**Cause:** you booted the stock image instead of the patched one.
**Fix:** power off fully, then **Vol Up + Power** to boot. A normal power press loads stock.

### Boots straight to Download Mode / `ODIN MODE (AVB Fail)` / `CUSTOM VBMETA`

**Cause:** AVB rejecting the modified image.
**Fix:** flash **stock `vbmeta.img` and `vbmeta_samsung.img` from the identical firmware build.**

```bash
7z e AP_x.tar.md5 vbmeta.img.lz4 vbmeta_samsung.img.lz4 -y
lz4 -d -f vbmeta.img.lz4 vbmeta.img
lz4 -d -f vbmeta_samsung.img.lz4 vbmeta_samsung.img
tar -cf stock_vbmeta.tar vbmeta.img.lz4 vbmeta_samsung.img.lz4
```

Flash `stock_vbmeta.tar` via Odin. ⚠️ **Must be the same firmware build** — vbmeta is
cryptographically bound to the firmware.

> Note: simply setting the AVB flags to `03` (disable verification + verity) frequently
> does **not** work on newer MTK builds. Stock vbmeta from the same build is the reliable fix.

### Bootloop at Samsung logo

**Cause:** usually the patched image or a CSC/vbmeta mismatch.
**Fix:** see [RECOVERY.md](RECOVERY.md). Usually recoverable by reflashing stock firmware.

### First boot takes >20 minutes

Usually fine on a slow device with a 52 GB data volume. Let it finish. Force-rebooting
during first-boot setup can strand it — if it exceeds ~40 min, see [RECOVERY.md](RECOVERY.md).

### `adb devices` shows nothing

**Fix:** on the phone, Developer options → **USB debugging (Security settings)** may also
need enabling; toggle USB config to **File transfer**; accept the RSA prompt.

### `adb: device unauthorized`

**Fix:** on the phone, Revoke USB debugging authorizations → reconnect → accept.

---

## Post-root issues

### Banking / DRM apps refuse to open

**Expected.** Knox tripped and/or Play Integrity failing. See [POST-ROOT.md](POST-ROOT.md).

### `adb shell su` hangs

**Fix:** Magisk needs interactive approval. Run `adb shell su` and grant on-screen.

### System app crashes after rooting

**Cause:** Zygisk module conflict.
**Fix:** Magisk → Modules → disable Zygisk, reboot.

---

## Sanity checklist before you flash

Run through this — it prevents most failures:

- [ ] Firmware **model** is `SM-A057F`
- [ ] Firmware **CSC** is `BD`
- [ ] Firmware **build** matches your current build number
- [ ] Patched **`init_boot.img`**, tens of MB, not a few KB
- [ ] LZ4 magic starts **`04224d186c`**
- [ ] Odin **Auto Reboot** is **unchecked**
- [ ] **Re-Partition** and **Nand Erase** are **unticked**
- [ ] Battery above 60%, good cable, direct USB port
- [ ] You know how to hold **Vol Up + Power**
- [ ] Stock firmware is on the PC in case you need to restore

---

Still stuck? [RECOVERY.md](RECOVERY.md)