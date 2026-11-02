# Troubleshooting index — symptoms, evidence and the accepted response

This page is a navigation aid. It does not replace the detailed case studies and does not authorize firmware or EFI changes by itself.

| Symptom | First distinction / evidence | Accepted response | Do not do |
|---|---|---|---|
| Apple logo/progress bar hangs before macOS | Compare exact OpenCore version/config and whether USB ownership is released | Keep UEFI ReleaseUsbOwnership=True; it was required on the E07 reference machine | Do not set ReleaseUsbOwnership=False because another laptop's sample does |
| Earlier boot failure before progress bar | Check SetupVirtualMap | Keep SetupVirtualMap=True; False failed on this machine | Do not normalize every quirk to upstream defaults |
| Touchpad missing or only partially enumerates | Check TPD0 ACPI path and the full I²C/GPIO/HID IORegistry chain | Use the tested VoodooI2C 2.8 dependency set | Do not upgrade to VoodooI2C 2.9.1 on this reference hardware without a new regression test |
| Headphone output does not return after unplug/replug | Confirm ALC298, AppleALC layout 30 and physical jack state | Test the actual jack and input/output endpoints first; ALCPlugFix is not in the accepted baseline because the reference replug test passed | Do not add ALCPlugFix pre-emptively |
| Hidden SSID does not auto-join on Sonoma | Separate AirportItlwm from itlwm behavior and confirm Keychain-backed helper | Historical Sonoma path: itlwm 2.3.0 + HeliPort 1.5.0 on demand + Keychain helper | Do not store the Wi-Fi PSK in plaintext in EFI or treat HeliPort as native Apple Wi-Fi |
| Broadcom Wi-Fi and Intel Wi-Fi interfere | Identify the physical card and loaded kexts | Ventura reference: Broadcom native Apple stack; disable Intel RP08 only under Darwin via SSDT-DRP08 | Do not load Intel AirportItlwm/itlwm and the Broadcom-native path simultaneously |
| Wi-Fi works but AirDrop does not | Check card model, AWDL and actual transfer | The reference Broadcom setup passed an actual AirDrop file transfer; test other Continuity features separately | Do not infer Handoff/Universal Clipboard/Sidecar from card enumeration alone |
| S4 wake produces 0x27 | Confirm a real S4 event: Maintenance DarkWake → Maintenance Sleep → Wake from Standby, same boot session | Preserve the validated HibernationFixup/PXSX/DMAR evidence and use the S4 case study; distinguish storage/DMA restore, display and Bluetooth faults | Do not count Wake from Deep Idle as S4; do not disable AppleVTD as a blanket fix |
| S4 resumes but screen briefly mosaics | Compare physical panel and same-session evidence; 51 ms VBlank timeout also appeared in clean cycles | Record as DEGRADED if it clears and no panic occurs; final user accepted the brief flash | Do not keep cycling gfxrst/FastLink/ProvideConsoleGop experiments |
| Opening lid does not wake deep S4 | Check the already-enabled Panel Open Wake setting and the actual EC/GPE path | Short power-button press is the accepted deep-S4 resume method | Do not re-add LID0 S4 depth, GPE16 or virtual LIDW patches; they broke clamshell detection |
| External HDMI 4K60 is black | Confirm that the cable is physically on HDMI, not USB-C/DP | Use 3840×2160@30 on the reference HDMI 1.4 path | Do not blindly patch framebuffer connectors |
| DP 4K60 is missing or link-trains unreliably | Distinguish IOFB timing range from actual CDCLK and verify the physical monitor image | Keep the final 0x00C3070B profile; see the DP case study for the historical DynamicCDCLK candidate | Do not re-enable DynamicCDCLK as a routine fix |
| NVMe CSTS=0xffffffff panic | Confirm that exact panic signature and SSD/controller context | Only then investigate the separate APST/forceRenderStandby hypothesis | Do not treat it as the same failure as the historical S4 0x27 IODMA/IOMapper panic; do not add NVMeFix by habit |
| SD card reader enumerates but no card mounts | Distinguish USB device enumeration from actual media read/write | Status remains UNVERIFIED; the user deprioritized SD media support | Do not install beta RealtekCardReader kexts without a requested test |
| Windows boot entry disappears or disk numbers change | Re-identify by physical SSD model and partition UUID | Keep Windows EFI separate; use OpenCore's validated boot chain | Do not hard-code disk0/disk1 or modify the Windows ESP as a first response |
| BIOS settings unexpectedly reset | Check RTC/CMOS behavior and the firmware baseline | Keep the recorded DisableRtcChecksum=True quirk and verify DVMT/CFG Lock readback | Do not flip RTC checksum quirks or rerun DVMT/0xE2 tools blindly |

## Deep-dive chapters

- [OpenCore quirks and boot decisions](04-opencore-baseline.md)
- [ACPI/SSDT/DMAR design](05-acpi-design.md)
- [Kext inventory and version decisions](06-kext-inventory.md)
- [Display and DP 4K60 case study](case-studies/external-display-dp.md)
- [S4 sleep/resume case study](case-studies/s4-sleep-resume.md)
- [EC fan policy case study](case-studies/ec-fan-policy.md)
- [Rejected experiments](decision-log/rejected-experiments.md)

## Evidence rule

A symptom label is not a root cause. Before changing anything, capture the exact OS build, OpenCore version, loaded kexts, active ACPI tables, relevant log window and the physical test conditions. Change one variable, retain a rollback, and verify the actual feature.
