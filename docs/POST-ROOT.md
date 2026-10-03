# After Rooting — SM-A057F

The flash worked. Now deal with the consequences.

---

## 1. Confirm it's actually rooted

```bash
adb shell su            # expect uid=0(root)
adb shell magisk -V     # expect a version
```

Magisk app must show **Installed**. If it doesn't, see
[TROUBLESHOOTING](TROUBLESHOOTING.md#boots-but-magisk-shows-not-installed).

---

## 2. Update immediately — while you still can via Odin

**OTA is gone.** Root disabled it. From now on the only update path is manual:

1. Download new stock firmware (SamFW / Frija)
2. Flash full stock via Odin (`BL`, `AP`, `CP`, `CSC`)
3. Re-patch and re-root

Put this in your calendar — check monthly. Your current patch is already **<EARLIER_PATCH_DATE>**;
don't let it drift further.

---

## 3. Play Integrity / banking apps

Rooting trips **Knox** and breaks **Play Integrity**. Expect:

- Google Wallet / Samsung Pay — **will not work**
- Most banking apps — refuse to launch or show an error
- Netflix HD, DRM content — downgraded or blocked
- Some corporate VPN / MDM apps — permanently incompatible

### Options

| Approach | Notes |
|---|---|
| **Do nothing** | Honest. Most reliable. |
| **Play Integrity Fix (Zygisk)** | `Kdrag0n/PlayIntegrityFix` on GitHub. Device must pass **hardware** integrity — must be a **GSM/A057F**, **not** a US carrier variant. |
| **LSPosed + Shamiko / Hideroot** | Hides root from apps. Play Integrity is *not* fixed by hiding alone. |

⚠️ On a **non-GSM US carrier variant** Play Integrity **cannot** be spoofed — the hardware
attestation is hardware-locked. Yours is GSM, so the Zygisk route is viable.

**Keep receipts.** Only install modules from the official repos. Fake "root hiding" modules
are a common malware vector — and they'd defeat the purpose of rooting safely.

---

## 4. Now do the forensics you originally wanted

This was the actual goal. Root unlocks everything that was blocked before.

### Live network state

```bash
cat /proc/net/tcp        # IPv4 sockets (hex — see decoder below)
cat /proc/net/tcp6       # IPv6 sockets
cat /proc/net/udp
cat /proc/net/arp        # neighbour table
cat /proc/net/route      # routing table
cat /proc/net/dev        # per-interface byte counters
```

`/proc/net/tcp` is hex-encoded. Decode it:

```bash
cat /proc/net/tcp | awk 'NR>1 {
  split($2,l,":"); split($3,r,":");
  printf "%-22s <- %-22s %s\n",
    l[1] ":" strtonum("0x" l[2]),
    r[1] ":" strtonum("0x" r[2]),
    $4 }'
```

State `01` = ESTABLISHED, `0A` = LISTEN.

### Enumerate apps

```bash
pm list packages -3                    # third-party only
pm list packages -s                    # system
pm list packages -f -3                 # with APK paths
dumpsys package <pkg> | grep -A5 permission
```

### Per-process sockets

```bash
lsof -nP -i
lsof -nP -i -a -p <pid>
```

### Look for common compromise indicators

```bash
# apps with accessibility service enabled (major malware vector)
dumpsys accessibility_config | grep -i enabled

# device admin apps
pm list packages -s | grep -iE 'device.?admin|locker'

# recently installed
ls -lt /data/app/*/ | head -30

# listening services
cat /proc/net/tcp | awk '$4=="0A"'
```

### Boot persistence

```bash
ls -la /data/adb/service.d/ /data/adb/post-fs-data.d/
ls /system/etc/init/
```

---

## 5. Hardening

Rooted ≠ automatically unsafe, but the bar is higher:

- Keep Zygisk modules to a minimum
- Re-check for root-hiding modules you didn't install
- Keep Termux's own packages updated (`pkg upgrade`)
- Review Magisk's app list for anything unfamiliar
- **Magisk app → Settings → hide the app** if you don't want it discoverable

---

## 6. Recommended first modules

| Module | Purpose |
|---|---|
| `PlayIntegrityFix` (Kdrag0n) | Restore Play Integrity on GSM variants |
| `LSPosed` | Xposed framework — app customisation, ad-blocking |
| `Shamiko` | Hide root from apps (pairs with LSPosed) |
| `Debloater` | Remove bloatware |

Install only from official repos. Verify checksums where published.

---

## 7. Get the guide off the phone

You're rooted and the phone may get wiped again. Keep this guide safe:

```bash
git clone https://github.com/PyDotAll/a05s-root-guide.git
```

It's private — you'll need a token or SSH key configured.

---

Next: [UNROOT.md](UNROOT.md) · back to [README](../README.md)