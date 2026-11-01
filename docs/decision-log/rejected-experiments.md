# Rejected experiments and decisions not to repeat

This file is intentionally explicit. The purpose is to prevent future contributors from repeating expensive or risky tests without new evidence.

## OpenCore / boot

| Candidate | Result | Final decision |
|---|---|---|
| `ReleaseUsbOwnership=False` | Apple logo/progress boot hang | Keep True |
| `SetupVirtualMap=False` | Earlier Apple logo boot failure | Keep True |
| `FixupAppleEfiImages=True` | No benefit in the current SecureBootModel=Default path; False passed | Keep False |
| `ForceBooterSignature=False` | Passed a prior isolated boot/Deep Idle A/B, but later S4 production profile retained True | Do not generalize an old test over the later S4 baseline |
| `DisableRtcChecksum=False` | Risk of CMOS/BIOS reset; not accepted | Keep existing safe value; no unattended A/B |

## Kext / ACPI cleanup

| Candidate | Result | Final decision |
|---|---|---|
| NVMeFix 1.1.3 | Early boot instability; three boots passed after disabling | Disabled |
| RestrictEvents | Single-variable removal passed tested boot/device checks | Disabled |
| VoodooI2C 2.9.1 | Broken/incomplete TPD0 HID/Multitouch chain | Keep 2.8 |
| SSDT-MCHC / SSDT-PMCR | Synthetic/unmatched nodes; independent removal tests passed | Removed |
| SSDT-MEM2 | Legacy synthetic node; independent removal tests passed | Removed |
| SSDT-HPET | OEM HPET restoration passed reboot/Deep Idle regression | Removed |
| Global `_UPC→XUPC` + SSDT-USB-WLAN_LTEBT | Broad legacy USB strategy | Replaced by precise 11-port USBMap |
| BlueToolFixup / BrcmPatchRAM | No evidence justified extra stack for native Broadcom path | Not used |
| ALCPlugFix | Physical headset jack replug worked | Not installed |
| RealtekCardReader | SD card not a project priority; actual media read/write was not accepted | Not installed |

## S4 / wake

| Candidate | Result | Final decision |
|---|---|---|
| Direct `hibernatemode=25` tests | Repeated 0x27 wake-timeout and display corruption in early baseline | Not a general production test method |
| LID0 `_PRW` depth S3→S4 | Did not make lid-open wake deep S4 | Rejected |
| GPE16 as LID0 wake source | Broke normal clamshell detection | Rejected |
| Separate virtual LIDW/GPE16 device | Also broke normal clamshell semantics | Rejected |
| Stock E07 EC to gain S4 lid wake | Did not wake from lid-open; fan behavior regressed | Rejected |
| Treating `Wake from Deep Idle` as S4 | Incorrect classification | Never do this |
| Battery-only 600/600/LPM0 test | One matched Ventura S4 cycle passed | LPM1 not required for that tested cycle |
| Repeated short S4 loops | Diminishing information after a full pass | Do not repeat without a new hypothesis |

## Graphics / display

| Candidate | Result | Final decision |
|---|---|---|
| framebuffer flags 0x00C3070A (FastLink bit cleared) | Physical mosaic flashes reproduced/worsened | Restore 0x00C3070B |
| ProvideConsoleGop=True | Did not improve physical S4 mosaic | Rejected |
| gfxrst=1/3/4 | No durable improvement; some variants produced boot/display artifacts | Rejected |
| global force-online | No useful discriminator | Removed |
| custom complete-modeset framebuffer list | No durable benefit | Removed |
| DynamicCDCLK candidate | No production benefit established | Not promoted |

## General lesson

A configuration can look more elegant and still be worse. Keep a candidate only when it has:
- a clear hypothesis;
- one controlled change;
- a known rollback;
- a physical test;
- a result that distinguishes it from the baseline.

Do not “clean up” a working quirk just because an upstream sample or generic guide uses a different value.
