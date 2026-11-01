# Accepted OpenCore / ACPI decisions with local evidence

This is the companion to the [rejected experiment log](rejected-experiments.md). It records settings that differ from a generic sample because the reference TM1701 was actually tested.

## Accepted cleanup

| Setting / component | Change | Evidence and reason |
|---|---|---|
| `ACPI.Quirks.NormalizeHeaders` | True → False | Three real reboots passed; Wi-Fi, audio and UHD 620 remained functional |
| `SSDT-MCHC` | Disabled/removed | Synthetic node was not needed by the live device tree |
| `SSDT-PMCR` | Disabled/removed | No live dependency; native PCI device driver already bound |
| `SSDT-MEM2` | Disabled/removed | No live dependency or OEM references |
| `SSDT-HPET` | Disabled/removed | OEM HPET restored; three reboots and a Deep Idle wake passed |
| `Kernel.Quirks.ThirdPartyDrives` | True → False | Both NVMe devices retained TRIM support; three reboots passed |
| `PlatformInfo.Generic.AdviseFeatures` | True → False | Three reboots passed; Windows boot topology remained correct |
| `FixupAppleEfiImages` | True → False | SecureBootModel=Default log showed the fixup was ignored; boot/Deep Idle and device checks passed |
| `RestrictEvents` | Disabled | Single-variable removal passed the tested boot, wake and device checks |
| `NVMeFix` | Disabled | Early stability A/B improved after disabling; no reproduced APST issue justified re-enabling |

## Quirks deliberately retained

| Setting | Final value | Why |
|---|---|---|
| `ReleaseUsbOwnership` | True | False caused a reproducible Apple-logo/progress boot hang |
| `SetupVirtualMap` | True | False caused an earlier boot failure before the progress bar |
| `ProtectUefiServices` | True | OpenCore debug logs showed runtime-service recovery evidence |
| `DevirtualiseMmio` | True | Live logs showed a significant MMIO runtime mapping being released |
| `RebuildAppleMemoryMap` | True | Retained in the tested memory-map path |
| `SyncRuntimePermissions` | True | Retained with the accepted memory-map configuration |
| `PowerTimeoutKernelPanic` | True | Kept for power-transition panic diagnostics |
| `DisableRtcChecksum` | True | No unattended A/B; disabling may reset CMOS/BIOS values on this platform |
| `AppleXcpmCfgLock` | False | Firmware CFG Lock was physically unlocked |
| `DisableIoMapper` | False | Final accepted configuration uses custom no-RMRR DMAR with AppleVTD active |

## ForceBooterSignature nuance

This setting has a history, not a one-line universal rule:

- An earlier isolated True→False test passed normal boot and Deep Idle.
- During later S4 engineering, the accepted hibernation-resume configuration retained `ForceBooterSignature=True` with `ResetHwSig=True` and HibernationFixup 1.5.4.
- Therefore the early False result must not be copied over the later S4 production profile.

## AppleVTD / DMAR nuance

The project briefly tested AppleVTD both enabled and disabled while isolating S4 0x27 failures. The final accepted profile retained AppleVTD with a custom no-RMRR DMAR and `DisableIoMapper=False`. The investigation did not establish that AppleVTD alone caused the original panic; the wake path involved DMA/IOMMU and PCI power-state restoration.

## Why this matters

A generic guide describes a starting point. The final quirk values must reflect the exact firmware and device tree. On this TM1701, two seemingly harmless changes — disabling `ReleaseUsbOwnership` or `SetupVirtualMap` — prevented boot. Do not “normalize” a working config without a new hypothesis and rollback path.
