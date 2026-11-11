# TWRP for Infinix NOTE 50 (X6858)

Unofficial TWRP 16.0 recovery for the Infinix NOTE 50.

> [!WARNING]
> **Requires Android 16 firmware and a compatible 6.12 kernel.**
> Android 15 and older firmware are not supported.

## Device information

| Item | Details |
|---|---|
| Device | Infinix NOTE 50 |
| Codename | `Infinix-X6858` |
| SoC | MediaTek Helio G100 Ultimate (MT6789) |
| Haptics | MTK regulator-vibrator |
| Display | AMOLED, 1080x2436, 144Hz, 480 dpi |
| Partition layout | Virtual A/B with dynamic partitions |
| Recovery image | `vendor_boot.img` |
| Development branch | `twrp-16.0-tos16` |
| Firmware | Android 16, `BP2A.250605.031.A3` |
| Kernel | 6.12.38 |

## Port status

This tree is ported from the X6886 TWRP tree and uses the stock X6858
payload for kernel modules and firmware. The build and the features below
are hardware-confirmed on an X6858.

Hardware-confirmed features:

- Recovery boot
- Touchscreen
- Vibration
- USB OTG, including hotplug and multi-partition drives
- FBE decryption
- Battery and charging status
- ADB and ADB sideload
- Fastbootd and A/B slot switching
- Normal Android boot alongside TWRP

## Reporting issues

Please include your firmware and kernel versions, the recovery build you
used, and steps to reproduce the problem.

Use **Advanced → Export logs** to save recovery logs. Review them for
personal information before sharing.

## Development

This tree uses the [TWRP-Test manifest](https://github.com/TWRP-Test/platform_manifest_twrp_aosp)
(`twrp-16.0` branch), not the official TeamWin minimal manifest.

Shared device support is maintained in the MT6789 common tree
(`device/transsion/mt6789-common`, branch `twrp-16.0-tos16`). Build
instructions and porting notes are in [AGENTS.md](AGENTS.md).

## Credits

- [Team Win Recovery Project](https://twrp.me)
- [OrangeFox Recovery Project](https://orangefox.download)
- MediaTek and Transsion for stock firmware components
- X6886 TWRP tree by [rama982 / ramabondanp](https://github.com/ramabondanp)
