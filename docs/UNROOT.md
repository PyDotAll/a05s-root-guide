# Unrooting / Restoring — SM-A057F

Two levels: remove root, or restore stock.

---

## Level 1 — Remove root, keep data

Magisk → **Uninstall** → **Complete uninstall (recommended)**

Removes Magisk and restores the stock boot image. **Data preserved.**

Then flash full stock firmware if you want certainty:

| Slot | File |
|---|---|
| BL | `BL_*.tar.md5` |
| AP | `AP_*.tar.md5` (stock, unpatched) |
| CP | `CP_*.tar.md5` |
| CSC | `HOME_CSC_*.tar.md5` — preserves data. Use `CSC` to wipe. |

Auto Reboot **on**. Normal boot.

---

## Level 2 — Full stock restore

The reliable path. Wipes data unless you use `HOME_CSC`.

1. Download stock firmware for `SM-A057F` / `<YOUR_FIRMWARE_BUILD>` / your carrier CSC
2. Extract `BL`, `AP`, `CP`, `CSC`
3. Download Mode (**Vol Up + Vol Down** + USB → **Vol Up**)
4. Odin as Administrator, load all four slots
5. Options: **Auto Reboot ✓**, **F. Reset Time ✓**, Re-Partition ✗, Nand Erase ✗
6. Start → `PASS!`

This is also the fix for a bootloop — see [RECOVERY.md](RECOVERY.md).

---

## ⚠️ What does NOT come back

### Knox — permanently, irreversibly

**No method resets the Knox counter.** Not Magisk uninstall. Not stock reflash. Not
factory reset. Not a service centre. It is a one-way eMMC fuse trip.

If you rooted, Knox is tripped forever. Consequences:

- Samsung Pay / Google Wallet — permanently non-functional
- Secure Folder — permanently broken
- Most banking apps — permanently broken
- Some corporate MDM/VPN — permanently broken

**Verify your Knox status before rooting** if you still can:

> Settings → About phone → **Knox warranty void: 0x0** = not tripped.
> Any other value = already tripped, root will not make it worse.

If it reads `0x0` and you rely on banking, **seriously reconsider**. This is the one
irreversible cost.

### OTA updates stay broken

Even after full stock restore, Samsung servers may refuse updates to a device whose
bootloader was unlocked. Manual Odin flashing may be your only path forever.

---

## If you only need root for one task

Consider: **don't root at all.**

| Goal | Rootless alternative |
|---|---|
| See installed apps | Settings → Apps, or ADB from the PC |
| Live socket table | `adb shell` + `ss`, or a PC-side Wireshark on WiFi |
| Uninstall bloatware | ADB `pm uninstall --user 0` (works without root) |
| Analyse an APK | Static analysis — no device changes needed |
| Per-app traffic stats | Android's own data usage settings |

ADB from a PC gets you most of what root was for here, with **zero** irreversible cost.

---

## If you need root once and it's gone

1. OTA is disabled — check Settings → Software update
2. If no update offers, reflash stock firmware via Odin manually
3. Re-patch `init_boot.img` and re-flash to root again
4. You can repeat this indefinitely — nothing is consumed

---

Next: back to [README](../README.md)