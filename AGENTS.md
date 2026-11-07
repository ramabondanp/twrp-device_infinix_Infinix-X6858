# AGENTS.md — Infinix X6858 / TWRP 16.0

## Device

Infinix NOTE 50, codename `Infinix-X6858`, MediaTek MT6789
(Helio G100 Ultimate), Android 16 / kernel 6.12.38.

This tree uses the shared `device/transsion/mt6789-common` tree for boot
layout, recovery fstab, init, TEE and crypto configuration.

## Constraints

1. Android 16 firmware with a 6.12 kernel only. Android 15 and older are
   unsupported.
2. Keep TWRP/build-system changes as patches under `patches/`, applied by
   `vendorsetup.sh`. Do not leave unrecorded source edits as the
   implementation of a fix.
3. Do not claim hardware validation without an actual X6858 test. Build
   success alone is not hardware validation.
4. Preserve unrelated work. Do not reset or discard unrelated trees.

## Build

```bash
repo init --depth=1 -u https://github.com/TWRP-Test/platform_manifest_twrp_aosp.git -b twrp-16.0
repo sync
# device trees are local checkouts under device/infinix/
source build/envsetup.sh
lunch twrp_X6858
m vendorbootimage
```

Output: `out/target/product/Infinix-X6858/vendor_boot.img`.

Use `~/twrp-16.0/tmp/` for scratch files. Do not write build scratch to
`/tmp`.

## Porting architecture

- `prebuilt/dtb.img` is the stock X6858 vendor_boot DTB
  (`c6a71e8dd261193acdf8d13838a6e906640dbe2c93cfc953360fb1c2747b7b85`).
- `recovery/root/lib/modules/` is the stock X6858 platform-ramdisk module
  set, plus the touch modules from `odm_dlkm`:
  - `adaptive-ts.ko` — patched by `tools/patch_adaptive_ts_ko.py` so the
    driver probes the panel in recovery. The patched hash is
    `4896112fc234f000d426bb558b550216bf4b332ed305dd6378d369a60f6be719`.
    Do not replace it with the unpatched stock module.
  - `focaltech_ft3683g.ko` — FocalTech touch controller.
  - `gt9916_common.ko` — Goodix GT9916 touch controller.
  - `modules.load.recovery` loads all three after their dependencies.
- `recovery/root/vendor/firmware/` and `recovery/root/odm/firmware/` are
  copied from the X6858 dump, including FocalTech, Goodix, haptic, LED
  and wireless-charging firmware.
- The common tree builds an empty `emptyrecovery` RECOVERY fragment and
  keeps the TWRP payload in the platform ramdisk.
### Haptics

X6858 uses the MTK `regulator-vibrator` (`regulator-vibrator.ko`),
exposing `/sys/class/leds/vibrator/{brightness,duration,activate}`.
The X6858 DTBO has no `haptic_hv` node; it only has
`/soc/regulator-vibrator` with `compatible = "regulator-vibrator"`.
The dump's generic vendor `vibrator-mtk-default.rc` mentions
`aw8697_haptic` / `vibrator_single`, but X6858 ships neither
`richtap_haptic_hv.ko` nor RichTap firmware; treat those references as
vestigial. The TWRP haptics patch falls through to the generic `vibrator`
LED pair and brightness pulse.

## Kernel module updates

The current payload uses only two stock module sources:

1. `vendor_boot.img` platform ramdisk — `lib/modules/`:
   `modules.load.recovery`, `modules.load`, `modules.dep`,
   `modules.alias`, `modules.softdep`, and all 242 modules. It is
   self-contained: every stock `modules.load.recovery` entry and every
   `modules.dep` dependency exists in this directory.
2. `odm_dlkm` — `lib/modules/`:
   `adaptive-ts.ko`, `focaltech_ft3683g.ko`, `gt9916_common.ko`.
   These are normally loaded by `odm_dlkm/lib/modules/modules.load`, not
   by the stock recovery list.

`vendor_dlkm` and `system_dlkm` are not used directly. Only pull a module
from them if a future stock `modules.load.recovery` references a module
that is missing from the new platform ramdisk.

### New firmware update

1. Extract the new matching `vendor_boot.img` platform ramdisk and
   `odm_dlkm` from the firmware dump.
2. Replace `recovery/root/lib/modules` with the new platform ramdisk
   `lib/modules` (keep the metadata files).
3. Copy the new `odm_dlkm` touch modules into the same directory:
   `adaptive-ts.ko`, `focaltech_ft3683g.ko`, `gt9916_common.ko`.
4. Re-apply the adaptive touch patch:

   ```bash
   python3 tools/patch_adaptive_ts_ko.py \
     recovery/root/lib/modules/adaptive-ts.ko
   ```

   `vendorsetup.sh` re-checks it automatically.
5. Re-append the three touch modules to `modules.load.recovery` and add
   the matching `modules.dep` / `modules.alias` entries. Do not rely on
   the stock recovery list including them.
6. Refresh touch firmware from the new `odm/firmware` and
   `vendor/firmware` if changed.
7. Verify:
   - every module in `modules.load.recovery` exists in the directory;
   - every dependency in `modules.dep` exists;
   - `modinfo -F vermagic` matches the stock kernel for all modules;
   - `prebuilt/dtb.img` still matches the new `vendor_boot` DTB; update it
     from the new image if the hash changed;
   - rebuild and inspect the ramdisk with `unpack_bootimg`.
8. Never mix modules from different firmware/kernel builds.

## Known validation gaps

The following still require X6858 hardware testing:

- Touch panel bring-up with the patched `adaptive-ts.ko` / `gt9916_common.ko`
- Haptics/vibration
- Charging and battery reporting
- FBE decryption
- USB OTG hotplug
- Fastbootd and slot switching
