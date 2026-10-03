# Offline Checklist

Print this / keep it off the wipe. Work top to bottom.

## Before
- [ ] Windows PC + data-capable USB cable
- [ ] Samsung USB drivers installed
- [ ] Odin 3.13.x+
- [ ] Frija available
- [ ] ~20 GB free on PC
- [ ] Phone charged 60%+

## Gate
- [ ] Developer options ON (Build number x7)
- [ ] OEM unlocking toggle present -> ON  (absent = STOP)
- [ ] USB debugging ON
- [ ] Note 7-day KG unlock timer if shown

## Data
- [ ] Photos/Drive sync verified complete
- [ ] 2FA seeds exported
- [ ] Irreplaceables copied off device

## Update
- [ ] OTA update installed FIRST
- [ ] New build number recorded: ____________

## Firmware
- [ ] Model SM-A057F / CSC BD / build matches
- [ ] BL, AP, CP, CSC, HOME_CSC extracted

## Patch
- [ ] boot.img, init_boot.img, vbmeta.img.lz4 extracted
- [ ] init_boot.img.lz4 decompressed
- [ ] Magisk -> Install -> Select and Patch a File
- [ ] magisk_patched.img produced
- [ ] Repacked with `lz4 -B6 --content-size`
- [ ] Magic verified: starts `04 22 4d 18 6c`

## Flash
- [ ] Download Mode (Vol Up + Vol Down + USB, Vol Up to continue)
- [ ] Odin ID:COM blue
- [ ] AP = patched tar; BL/CP empty; CSC empty or HOME_CSC
- [ ] Auto Reboot UNCHECKED
- [ ] Start -> PASS!

## Boot
- [ ] Vol Down + Power (off)
- [ ] Vol Up + Power (boot patched image)
- [ ] Magisk shows Installed
- [ ] `adb shell su` works

## Gotchas
- Stuck "File Analysis"      -> patch init_boot.img alone, not AP
- "Unexpected end of archive" -> same
- "FAIL! LZ4 is invalid"      -> used plain lz4, needs --content-size
- "ODIN MODE (AVB Fail)"      -> flash stock vbmeta from SAME firmware
- `KG status: Checking`        -> 7-day Samsung wait
- No root after flash          -> reboot holding Vol Up + Power
