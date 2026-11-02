# OpenCore 1.0.7 — reference baseline and important quirks

This is a sanitized description of the final Ventura reference configuration. It is not a replacement for the official OpenCore Configuration.pdf and is not a complete config.plist.

## Core identity and boot behavior

| Setting | Reference value | Rationale |
|---|---|---|
| OpenCore | 1.0.7 | Matching config schema and ocvalidate |
| SMBIOS model | MacBookPro15,4 | Reference platform identity only |
| SMBIOS serial / MLB / ROM / UUID | **Not published** | Must be generated uniquely per machine |
| `Misc → Boot → ShowPicker` | `False` | Normal boot bypasses picker |
| `Misc → Boot → Timeout` | `0` | No automatic picker delay |
| `PollAppleHotKeys` | `True` | Allows boot hotkeys to call the picker |
| `HideAuxiliary` | `True` | Keeps auxiliary entries hidden by default |
| `LauncherOption` | `Full` | Maintains the selected OpenCore boot chain |
| `HibernateMode` | `Auto` | Retains OpenCore hibernation detection |
| Vault | Basic Vault | Hash consistency checked after config updates |
| SecureBootModel | Default | Kept; no OCLP/root-patch route in the final Ventura profile |
| DMG loading | Signed | Only signed installer images are accepted by the configured policy |
| SIP | Enabled | `csr-active-config=00000000` |

The user physically confirmed a reboot directly entered Ventura without a picker/countdown. `PollAppleHotKeys=True` remains enabled so the picker can still be summoned deliberately. `LauncherOption=Full` is paired with `RequestBootVarRouting=True` for the Windows/OpenCore boot chain. No routine NVRAM reset was required.

## UEFI drivers and boot sound

Enabled UEFI drivers in the archived Ventura candidate:

- `OpenRuntime.efi`
- `HfsPlus.efi`
- `OpenCanopy.efi`
- `AudioDxe.efi`
- `ResetNvramEntry.efi`

`UEFI/Audio/AudioSupport=True` and `PlayChime=Auto` are configured. This is the OpenCore preboot audio path; it is separate from AppleALC/macOS audio acceptance.

## Important boot arguments

The final Ventura reference config retained:

```text
ps2kbdonly=1 keepsyms=1 debug=0x100 -hbfxdbg hbfx-patch-pci=XHC,IMEI,IGPU,PXSX hbfx-ahbm=129
```

These are not generic defaults. In particular, the HibernationFixup arguments were retained because they were part of the experimentally validated S4 path. Do not remove or add PCI device names by copying this line to a different ACPI namespace.

## Quirks with decisive local evidence

| Setting | Reference decision | Evidence |
|---|---|---|
| `UEFI → Quirks → ReleaseUsbOwnership` | **True** | False caused a real Apple-logo/progress boot hang |
| `Booter → Quirks → SetupVirtualMap` | **True** | False caused an earlier Apple-logo boot failure |
| `Booter → Quirks → ForceBooterSignature` | True in final S4 profile | Part of the final S4-resume configuration |
| `Booter → Quirks → FixupAppleEfiImages` | False | Single-variable A/B passed with SecureBootModel=Default |
| `Booter → Quirks → ProtectUefiServices` | True | Live debug logs showed runtime-service recovery evidence |
| `Booter → Quirks → DevirtualiseMmio` | True | Live logs showed a material MMIO runtime mapping being released; the retained whitelist includes the verified 0xFFA00000 range |
| `Booter → Quirks → RebuildAppleMemoryMap` | True | Retained after real runtime validation |
| `Booter → Quirks → SyncRuntimePermissions` | True | Retained with the validated memory-map path |
| `Kernel → Quirks → AppleXcpmCfgLock` | False | Firmware CFG Lock was physically unlocked |
| `Kernel → Quirks → DisableIoMapper` | False | Final profile uses a custom no-RMRR DMAR with AppleVTD active |
| `Kernel → Quirks → ThirdPartyDrives` | False | Both NVMe devices reported TRIM support |
| `Kernel → Quirks → XhciPortLimit` | False | A precise USBMap is used |

## Graphics and audio properties

- UHD 620 framebuffer reference flags: `0x00C3070B`.
- `enable-max-pixel-clock-override=1` was retained for the tested connector/mode set.
- HDEF uses AppleALC layout 30.
- Do not copy the full DeviceProperties block to a different panel or connector map.

## Additional non-default quirks in the archived Ventura candidate

These values are part of the exact candidate config and should be treated as a recorded baseline, not a checklist to toggle casually:

- `ACPI/Quirks/ResetHwSig=True`
- `ACPI/Quirks/NormalizeHeaders=False`
- `Booter/Quirks/AvoidRuntimeDefrag=True`
- `Booter/Quirks/ResizeAppleGpuBars=-1` and `UEFI/Quirks/ResizeGpuBars=-1`
- `Kernel/Quirks/CustomSMBIOSGuid=True`
- `Kernel/Quirks/DisableLinkeditJettison=True`
- `Kernel/Quirks/DisableRtcChecksum=True`
- `Kernel/Quirks/PanicNoKextDump=True`
- `Kernel/Quirks/PowerTimeoutKernelPanic=True`
- `Kernel/Quirks/SetApfsTrimTimeout=-1`
- `UEFI/Quirks/EnableVectorAcceleration=True`
- `UEFI/Quirks/RequestBootVarRouting=True`

`DisableRtcChecksum=True` is retained because RTC checksum handling can otherwise reset firmware settings on this platform. Do not change it merely to make the plist resemble another laptop's sample. `PowerTimeoutKernelPanic=True` is retained for power-transition diagnostics; its presence does not mean every possible wake failure is hidden or suppressed.

The candidate's final ACPI section also deletes the OEM DMAR (`TableSignature=DMAR`) before adding the custom no-RMRR table. `ResetHwSig` is an **ACPI quirk**, not a `Misc/Boot` or `Booter/Quirk` entry.

## AppleVTD / DMAR

The final profile retains:
- OEM DMAR deletion followed by the custom no-RMRR DMAR table;
- `DisableIoMapper=False`;
- AppleVTD active at runtime.

The project did not simply disable VT-d to hide every DMA/IOMapper log. The S4 investigations showed that early panic stacks involved DMA/IOMMU restore paths, but the final stable S4 profile used AppleVTD with the custom DMAR after controlled comparisons. The exact root cause was not reduced to a single switch.

## Dual boot and NVRAM

- Keep the Windows EFI separate and untouched.
- OpenCore-first boot is the reference design; select Windows from OpenCore when required.
- Do not regenerate SMBIOS identity just for cleanliness after Apple services are working.
- Do not reset NVRAM as a routine installation step. Reset only when a concrete boot-state problem and recovery plan justify it.

## Validation requirement

Every change to OpenCore must use the matching version's `ocvalidate`, verify the Vault, inspect enabled-file references and preserve a complete rollback copy. A successful plist parse alone is not sufficient. The 2026-11-01 closeout archive contains the exact candidate config and Vault; the guarded install reported matching write-after-hash checks and the user accepted the direct boot. A later read-only EFI mount attempt did not mount, so the project does **not** claim an independent post-install live plist readback.
