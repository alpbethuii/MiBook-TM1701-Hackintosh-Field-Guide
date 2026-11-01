# Installation guide — reproduce the tested TM1701 baseline

This chapter is a controlled installation outline, not a one-click installer. The exact reference baseline was macOS Ventura 13.7.8 / OpenCore 1.0.7 on a TM1701 i7-8550U. Re-check current Apple security support before choosing Ventura for a new daily-use machine.

## 0. Confirm the target

Do not start until all of the following are known:

- It is TM1701/TM1707 8th-gen KBL-R, not TM1905/TM1963 CML.
- CPU is supported and UHD 620 is present.
- Panel, audio codec, touchpad and Wi-Fi card have been identified.
- There is a complete backup of all personal data and both internal drives.
- A separate rescue USB exists and has been boot-tested.
- Windows EFI/BCD is not being used as a scratch area.

## 1. Firmware preparation

Read [BIOS and EC notes](03-bios-and-ec.md). The reference machine had a one-time DVMT 64 MB / CFG Lock adjustment. Do not repeat firmware tools just because a guide says so. Verify the current firmware state first.

Do not flash a BIOS or EC image from another machine. Do not copy the custom EC fan image described in the engineering notes.

## 2. Start from the correct upstream KBL release

Use the upstream [daliansky/XiaoMi-Pro-Hackintosh](https://github.com/daliansky/XiaoMi-Pro-Hackintosh) project and select the **KBL / 8th-gen release** that matches the macOS target.

- Do not use TM1905/TM1963 CML EFI.
- Do not treat the upstream `main` branch as a production release.
- Read the release notes and inspect the config before use.
- Use the [Dortania Kaby Lake laptop guide](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/kaby-lake.html) as the architecture reference, not as a substitute for model-specific validation.

## 3. Build a machine-specific OpenCore config

The public guide does not ship a ready-to-flash EFI. Build a config for the exact machine:

1. Use the matching OpenCore release and its own Sample.plist / ocvalidate.
2. Add only the ACPI tables and kexts documented in this guide that match your hardware.
3. Generate a unique SMBIOS identity for your own machine. Never copy another person's serial, MLB, ROM or UUID.
4. Keep the correct model identifier family (the reference used MacBookPro15,4); do not copy the private identity fields from anyone else's plist.
5. Validate the config with the matching `ocvalidate`.
6. Rebuild Basic Vault hashes after every config or EFI content change.
7. Check that every enabled ACPI/kext/driver file exists and that no disabled/orphaned experimental files are accidentally loaded.

## 4. Prepare a USB installer

Use a separate USB device. Confirm the physical device by model and capacity before erasing it. Create the macOS installer using Apple's supported `createinstallmedia` workflow, then place the machine-specific OpenCore EFI on the USB EFI partition.

Do not erase or repartition either internal NVMe as part of USB preparation.

## 5. First boot and installation

1. Boot the USB through UEFI.
2. Confirm the OpenCore picker identifies the installer and the rescue path.
3. Install macOS to the intended macOS target only.
4. Do not modify the Windows disk or Windows EFI partition.
5. Keep the USB EFI available until the internal installation has been boot-tested independently.
6. After first boot, verify that the installed system is running from the intended internal APFS volume.

## 6. Post-install acceptance

Complete the [validation matrix](11-validation-matrix.md) before treating the system as daily-use ready:

- normal cold boot and reboot;
- UHD 620 acceleration and internal display;
- brightness and HiDPI;
- audio output, internal mic, headset mic and jack replug;
- touchpad gestures and keyboard/function keys;
- Wi-Fi reconnect and Bluetooth;
- AirDrop transfer if supported by the installed card;
- USB mapping and both USB-C ports;
- ordinary sleep/wake;
- battery S4 only after a separate safety-reviewed test.

## 7. Do not copy the reference machine's power settings blindly

The final 12h/3h policy is the result of months of A/B testing on one firmware/EC combination. It is not a universal first-boot default. Start with ordinary sleep, collect a clean baseline, then evaluate S4 separately.

## 8. Recovery plan

Before any internal EFI write:

- identify the macOS SSD by exact physical model and EFI partition UUID;
- make a byte-level or complete file-level backup;
- preserve the rescue USB;
- verify the backup hash;
- change one logical group;
- write, sync, unmount and verify;
- do not reboot until the post-write audit passes.

Disk identifiers such as `disk0` and `disk1` are transient and may swap after reboot. Never hard-code them in an installation script.
