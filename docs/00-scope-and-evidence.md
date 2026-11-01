# Scope, chronology and evidence rules

## Scope

This public guide reconstructs the engineering work recorded for one Xiaomi Mi Notebook Pro TM1701 beginning with the project restart on 2026-09-21 and ending with the Ventura 13.7.8 production baseline on 2026-11-01.

It does not claim to reconstruct every historical macOS installation ever performed on every TM1701. The public source record available for this edition begins with the September 2026 project restart, the 2026-09-22 upstream KBL source audit and the 2026-09-23 live ACPI/OpenCore audit.

## Timeline at a glance

| Date | Milestone | Result |
|---|---|---|
| 2026-09-21 | Hackintosh project reactivated | Reopened an existing TM1701 installation and began evidence-based cleanup |
| 2026-09-22 | Upstream KBL source and BIOS documentation audit | Confirmed Kaby Lake Refresh branch; one-time DVMT/CFG Lock work completed |
| 2026-09-23 | Live ACPI, OpenCore and kext audits | Removed inert/legacy entries; isolated NVMeFix, USB and boot quirk failures |
| 2026-09-23–24 | S4 / framebuffer / IOMMU experiments | Several real 0x27 wake failures; no single-variable root cause claimed |
| 2026-09-24 | EFI GOLD and display baseline | Clean OpenCore 1.0.7 baseline; HiDPI and ColorSync accepted |
| 2026-09-25 | Wi-Fi and EC fan policy | Hidden-SSID handling improved; EC502-derived fan strategy selected |
| 2026-09-28–30 | Automatic S3→S4 work | Battery 3-hour S4 path reached overnight acceptance; lid-open S4 wake remained unsupported |
| 2026-10-14–17 | Display-link and long sleep observations | USB-C/DP and USB-C data paths accepted; S4 wake/display limitations documented |
| 2026-10-21–24 | OS decision and Ventura migration preparation | Ventura 13.7.8 selected; Broadcom-native Wi-Fi path chosen |
| 2026-10-25 | Ventura installation and hardware acceptance | Ventura 13.7.8 booted; HiDPI, audio, keyboard and sleep replay validated |
| 2026-10-25 | S4 LPM0 A/B and production power policy | LPM0 shown sufficient for one controlled S4 cycle; final 12h/3h policy selected |
| 2026-11-01 | Closeout | Temporary build material archived/cleaned; project moved to observation-only |

## How to read the evidence

A configuration being present is not proof that it works. A kext being on disk is not proof that it is loaded. A log line saying “Wake” is not proof of S4. A screenshot is not proof that the physical panel displayed correctly.

The project therefore separates:

1. **Configuration evidence** — plist parsing, `ocvalidate`, Vault hashes, file inventories.
2. **Runtime evidence** — IORegistry, loaded kexts, device enumeration, power assertions, `pmset`.
3. **Physical acceptance** — actual keyboard/trackpad/monitor/headphone/AirDrop/sleep-wake tests.
4. **Regression evidence** — same boot session, no new panic, restored settings, successful rollback.

### Result vocabulary

| Label | Meaning |
|---|---|
| PASS | The exact feature passed the stated test |
| PASS (configuration) | Static configuration validation only |
| DEGRADED | Works, but a user-visible imperfection remains |
| FAIL | Reproducible or decisive failure |
| REJECTED | Candidate is not part of the production baseline |
| UNVERIFIED | Evidence is insufficient; do not infer success |

## Privacy and publication boundary

This repository intentionally omits:

- SMBIOS `SystemSerialNumber`, `MLB`, `SystemUUID`, `ROM` and all generated identity values.
- Real MAC addresses, private IPs, hostnames, local user names, credentials and local filesystem paths.
- Full EFI directories, OpenCore Vaults, BIOS/SPI dumps, EC firmware images, raw disk images and private recovery archives.
- Raw logs containing machine identifiers or unrelated personal data.

Only sanitized settings, version numbers, functional descriptions, selected error signatures and engineering conclusions are published.

## Engineering method

The work followed a simple discipline:

1. Establish a known-good baseline and back it up.
2. Identify the physical machine and the exact disk by model/partition identity, not by a transient disk number.
3. Change one variable or one logically inseparable group.
4. Validate the configuration before reboot.
5. Test the actual user-visible feature.
6. Check kernel continuity, panic reports and logs.
7. Restore the baseline if the candidate fails.
8. Record the decision so later sessions do not repeat the same experiment.

This is why the S4 chapter is long: it preserves the distinction between an apparent wake, a real S4 transition, a successful kernel resume, and a fully acceptable physical display experience.
