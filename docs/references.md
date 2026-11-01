# References and upstream sources

This project is a machine-specific engineering record. The upstream documents below are the authoritative starting points for new users; this repository records what was actually tested on one TM1701.

## Xiaomi Mi Notebook Pro

- [daliansky/XiaoMi-Pro-Hackintosh](https://github.com/daliansky/XiaoMi-Pro-Hackintosh) — upstream project, model families, release packs, BIOS/HiDPI notes and changelog.
- [Upstream TM1701/TM1707 KBL configuration](https://github.com/daliansky/XiaoMi-Pro-Hackintosh/tree/main/OC) — inspect the current KBL branch and release notes; do not blindly use the moving main branch as a production EFI.
- [Upstream DVMT / 0xE2 helper](https://github.com/daliansky/XiaoMi-Pro-Hackintosh/tree/main/BIOS/TM1701/DVMT_and_0xE2_fix) — historical model-specific firmware preparation; use only after reading its current instructions and assessing brick risk.

## OpenCore / Dortania

- [OpenCore Install Guide — Kaby Lake laptop](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/kaby-lake.html)
- [OpenCore Configuration reference](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf)
- [OpenCore releases](https://github.com/acidanthera/OpenCorePkg/releases)
- [Getting Started with ACPI](https://dortania.github.io/Getting-Started-With-ACPI/)
- [USB mapping guide](https://dortania.github.io/OpenCore-Post-Install/usb/)
- [iServices guide](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html)

## Kext projects

- [Lilu](https://github.com/acidanthera/Lilu)
- [VirtualSMC](https://github.com/acidanthera/VirtualSMC)
- [WhateverGreen](https://github.com/acidanthera/WhateverGreen)
- [AppleALC](https://github.com/acidanthera/AppleALC)
- [HibernationFixup](https://github.com/acidanthera/HibernationFixup)
- [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C)
- [VoodooPS2](https://github.com/acidanthera/VoodooPS2)
- [OpenIntelWireless / itlwm](https://github.com/OpenIntelWireless/itlwm)
- [USBMap](https://github.com/corpnewt/USBMap)

## Apple platform and security references

- [Apple — About the security content of macOS Ventura 13.7.8](https://support.apple.com/en-us/124929) — lists Ventura 13.7.8 as released on 2025-08-20.
- [Apple security releases](https://support.apple.com/en-us/100100) — check the current supported release before choosing an OS for a new installation.

## Attribution

The upstream Xiaomi-Pro project and each kext project retain their own licenses and copyrights. This repository does not redistribute their EFI binaries or firmware images. Documentation here is original commentary based on a specific machine's test records; see the repository license.
