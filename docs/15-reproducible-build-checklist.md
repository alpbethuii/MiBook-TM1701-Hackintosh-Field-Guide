# Reproducible build checklist — assemble a TM1701-class EFI from evidence

This is the short-form assembly route for an experienced Hackintosh builder. It is **not** a ready-to-flash EFI, and it does not replace the OpenCore Configuration.pdf or the upstream Xiaomi KBL documentation.

The reference result is TM1701 / i7-8550U / UHD 620 / 8 GB, macOS Ventura 13.7.8, OpenCore 1.0.7. Hardware substitutions require a new audit.

## 0. Confirm the exact machine family

- Supported reference family: **TM1701 / TM1707, 8th-gen Kaby Lake Refresh**.
- Do not use the TM1905 / TM1963 10th-gen Comet Lake EFI, ACPI or sleep conclusions.
- Record the panel EDID, audio codec, touchpad bus, wireless cards, NVMe models and BIOS/EC version before building.

## 1. Match the reference hardware where possible

| Component | Reference | Substitution rule |
|---|---|---|
| CPU / iGPU | i7-8550U / UHD 620 | Must remain KBL-R/UHD620 for this profile |
| dGPU | NVIDIA MX150 | Disabled only in macOS; do not expect Optimus support |
| Panel | BOE NV156FHM-N61, 1920×1080 | Re-read EDID; do not copy BOE override blindly |
| Audio | Realtek ALC298 | Confirm codec before using layout 30 |
| Touchpad | ETD2303 I²C/GPIO | Confirm exact bus and ACPI path |
| Wi-Fi | Added Broadcom BCM943602CS / BCM43602 | The original Intel AC8265 remains physically installed; Broadcom is an added WLAN_LTE-slot path |
| Bluetooth | Broadcom BCM20703A1 | USB data is routed through the WLAN_LTE hardware path |
| Storage | Samsung PM961-class NVMe for macOS; separate Windows NVMe | Identify by physical model and partition UUID, not disk number |

If the Broadcom card is absent, do not load the Broadcom-specific final profile and assume Intel will behave identically. The historical Sonoma Intel path used itlwm 2.3.0 + HeliPort 1.5.0 and a Keychain-backed hidden-SSID helper; it did not provide the same native Apple AWDL/Continuity path.

## 2. Firmware baseline

Reference BIOS: **XMAKB5R0P0E07 (E07)**.

The verified one-time firmware preparation on the reference machine was:

- DVMT pre-allocation = 64 MB;
- CFG Lock / MSR 0xE2 = unlocked;
- the BIOS Panel Open Wake Up option was already enabled.

Do not rerun DVMT/0xE2 tools on a machine that already has the target state. Do not flash another unit's BIOS or EC image. Keep BIOS Write Lock enabled by default; the reference owner's later 0x00 choice is not a general recommendation.

The optional custom EC502-derived 62°C fan profile is documented separately. It is not required for macOS installation.

## 3. Select and prepare the macOS installer

Ventura 13.7.8 is the **tested reference OS**, not a current security recommendation. Check Apple's current security releases and application requirements before choosing an OS for a new internet-connected daily-use installation.

On a supported Mac, after obtaining the full Apple installer application, create the USB installer:

```bash
sudo "/Applications/Install macOS Ventura.app/Contents/Resources/createinstallmedia" --volume "/Volumes/InstallUSB"
```

Confirm the USB by physical model and capacity before running the command; it erases the selected USB volume. Never target either internal NVMe.

## 4. Build the OpenCore tree

- Use **OpenCore 1.0.7 RELEASE** and its matching Sample.plist, documentation and ocvalidate.
- Start from the Xiaomi **KBL / 8th-gen** source, not the CML pack.
- Do not copy a complete EFI from another person.
- Generate a unique SMBIOS identity for the target machine. The reference model identifier is MacBookPro15,4; never copy its serial, MLB, ROM or UUID.
- Keep a separate rescue USB.
- Keep the Windows ESP/BCD untouched.

## 5. Add the ACPI tables

The final Ventura candidate has these 12 ACPI entries:

1. SSDT-ALS0
2. SSDT-DDGPU
3. SSDT-EC-USBX
4. SSDT-GPRW
5. SSDT-LGPA
6. SSDT-PLUG
7. SSDT-PNLF
8. SSDT-PS2K
9. SSDT-RMNE
10. SSDT-TPD0
11. SSDT-DRP08
12. DMAR (custom no-RMRR)

Also apply the two paired method renames:

- GPRW → XPRW, paired with SSDT-GPRW;
- LGPA → XGPA, paired with SSDT-LGPA.

Delete the OEM DMAR before adding the custom table. Do not replace the entire DSDT. The full purpose and evidence for each table are in [ACPI design](05-acpi-design.md).

## 6. Add the kexts

The 17 enabled Kernel/Add entries in the final Ventura candidate are:

1. Lilu 1.7.2
2. HibernationFixup 1.5.4
3. VirtualSMC 1.3.7
4. SMCLightSensor 1.3.7
5. SMCBatteryManager 1.3.7
6. SMCProcessor 1.3.7
7. WhateverGreen 1.7.0
8. AppleALC 1.9.7
9. NullEthernet 1.0.8
10. VoodooI2CServices 1.0 (nested)
11. VoodooGPIO 1.1 (nested)
12. VoodooI2C 2.8
13. VoodooI2CHID 1.0
14. VoodooPS2Controller 2.3.8
15. VoodooPS2Keyboard 2.3.8 (nested)
16. VoodooInput 1.1.7 (nested)
17. USBMap 1.0

Do not add NVMeFix, RestrictEvents, BlueToolFixup, BrcmPatchRAM, Intel Bluetooth kexts or ALCPlugFix without a reproducible symptom and a controlled test. VoodooI2C 2.9.1 is a known regression on this reference touchpad.

## 7. Apply the reference DeviceProperties

The following values were extracted from the archived Ventura production candidate. They are a starting point for the matching panel/connector layout, not universal properties for all TM1701 revisions.

**IGPU — PciRoot(0x0)/Pci(0x2,0x0)**

| Property | Data |
|---|---|
| AAPL,ig-platform-id | 05001c59 |
| device-id | 16590000 |
| enable-max-pixel-clock-override | 01000000 |
| framebuffer-con1-enable | 01000000 |
| framebuffer-con1-pipe | 0a000000 |
| framebuffer-con1-type | 00080000 |
| framebuffer-con2-enable | 01000000 |
| framebuffer-con2-type | 00040000 |
| framebuffer-flags | 0b07c300 (little-endian representation of 0x00C3070B) |

**HDEF — PciRoot(0x0)/Pci(0x1f,0x3)**

- layout-id = 30;
- hda-gfx = onboard-1;
- hda-idle-support = 1.

Do not turn on DynamicCDCLK just to make DP 4K60 appear; see the external display case study.

## 8. Apply the validated OpenCore settings

Important reference values:

- Misc/Boot: ShowPicker=False, Timeout=0, PollAppleHotKeys=True, HideAuxiliary=True, HibernateMode=Auto, LauncherOption=Full.
- Misc/Security: SecureBootModel=Default, Vault=Basic, DmgLoading=Signed.
- ACPI quirks: ResetHwSig=True, NormalizeHeaders=False.
- Booter quirks: ReleaseUsbOwnership=True, SetupVirtualMap=True, ForceBooterSignature=True, ProtectUefiServices=True, DevirtualiseMmio=True, RebuildAppleMemoryMap=True, SyncRuntimePermissions=True, AvoidRuntimeDefrag=True.
- Kernel quirks: AppleXcpmCfgLock=False, DisableIoMapper=False, DisableRtcChecksum=True, ThirdPartyDrives=False, XhciPortLimit=False, PowerTimeoutKernelPanic=True.
- UEFI quirks: EnableVectorAcceleration=True, RequestBootVarRouting=True.
- SIP: enabled (csr-active-config=00000000).

The final boot arguments are:

```text
ps2kbdonly=1 keepsyms=1 debug=0x100 -hbfxdbg hbfx-patch-pci=XHC,IMEI,IGPU,PXSX hbfx-ahbm=129
```

Use a visible OpenCore picker during installation and early recovery testing. Hide the picker only after the internal boot path and rescue USB have been accepted.

## 9. USB and wireless

- Use the precise 11-port USBMap; keep XhciPortLimit=False.
- Do not re-enable the old global _UPC→XUPC rename or SSDT-USB-WLAN_LTEBT.
- For the Broadcom configuration, keep the Darwin-only SSDT-DRP08 that disables the unused Intel RP08 WLAN device.
- The Broadcom Wi-Fi and Bluetooth stack is native in the final Ventura profile; do not load the Intel Wi-Fi/BT stack at the same time.

## 10. Post-install system configuration

- On the reference Ventura installation, the BOE-specific 1424×802 HiDPI override was restored separately after the initial OS migration and physically accepted on 2026-10-25. For another machine, install only after confirming the exact BOE panel EDID; it is a system-level display override, not an EFI feature.
- Let macOS generate and bind its own Color LCD ICC profile; do not inject a fake Apple EDID or a generic ICC profile.
- ALC298 layout 30 is the accepted audio path. ALCPlugFix is not needed when headphone replug and headset microphone already pass.
- Verify ETD2303 gestures and the full keyboard/F-key path physically.
- Test AirDrop with a real file transfer; do not infer all Continuity features from the presence of AWDL or a Broadcom card.

## 11. Sleep and S4

First establish ordinary S3 sleep/wake. Do not begin with forced hibernatemode=25.

The reference machine's final Battery S3→S4 policy is high-charge 12 hours / low-charge 3 hours, with lowpowermode=0. AC standby is disabled. This is the result of extensive machine-specific testing, including historical 0x27 wake failures, PXSX/HibernationFixup changes and a matched Ventura LPM0 A/B.

Do not copy the timing values to another unit until ordinary sleep, battery detection, power-source selection and rollback have been validated. Do not count Wake from Deep Idle as a successful S4 resume.

## 12. Final acceptance

Complete the [validation matrix](11-validation-matrix.md). At minimum:

- cold boot, reboot, rescue USB and Windows boot;
- UHD 620 acceleration, internal panel, brightness and HiDPI;
- ALC298 speakers, internal mic, headset mic and jack replug;
- ETD2303 gestures, keyboard and function keys;
- Wi-Fi reconnect, Bluetooth and AirDrop;
- all physical USB ports and both USB-C orientations;
- HDMI 4K30 and the exact monitor/cable route;
- ordinary S3 sleep/wake;
- battery S4 only after separate approval and a safe rollback plan.

A configuration that passes ocvalidate is not automatically a physically accepted laptop.
