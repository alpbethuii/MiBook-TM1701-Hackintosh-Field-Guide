# Engineering timeline — from reactivation to the Ventura baseline

This is a condensed index of the major decisions. Detailed S4 evidence lives in the [S4 case study](case-studies/s4-sleep-resume.md).

## 2026-07-26 → 2026-09-21 — retirement reversed; project reactivated

A previous decision to retire the Hackintosh on 2026-07-26 was explicitly withdrawn on 2026-09-21. The work resumed with the goal of approaching a reliable, Mac-like daily experience rather than merely reaching the desktop.

## 2026-09-22 — hardware and upstream audit

- Confirmed the reference platform is TM1701 / Kaby Lake Refresh, not the 10th-gen CML family.
- Audited the upstream Xiaomi project, BIOS helper and hardware compatibility notes.
- Completed the one-time DVMT 64 MB / CFG Lock preparation.
- Identified the Broadcom BCM943602CS path as a possible route to native Apple Wi-Fi/BT and Continuity.

## 2026-09-23 — live EFI and ACPI cleanup

- Audited live ACPI namespace, kext loading and OpenCore configuration.
- Disabled synthetic ACPI nodes after independent removal tests.
- Disabled NVMeFix after an early stability A/B.
- Kept required platform-specific quirks after real boot failures showed that generic defaults were wrong for this firmware.
- Replaced the legacy USB strategy with a precise 11-port USBMap.
- Separated ordinary Deep Idle success from true S4 acceptance.

## 2026-09-23 to 09-24 — first S4 and display investigation

- Forced S4/mode-25 tests produced 0x27 wake-timeout panics and physical display corruption.
- Tested framebuffer, AppleVTD/DMAR, HibernationFixup, PCI patch scope and memory-map variables.
- Some combinations passed individual cycles but failed later; no single switch was declared the universal root cause.
- Restored a known-good baseline after failed experiments.

## 2026-09-24 — EFI and display baseline

- Cleaned disabled/orphaned EFI components.
- Validated OpenCore 1.0.7 configuration and Vault.
- Accepted real BOE EDID plus software HiDPI 1424×802 and macOS-generated Color LCD profile.
- Rejected unnecessary EDID spoofing.

## 2026-09-25 — Wi-Fi and EC fan work

- Solved hidden-SSID auto-join for the Intel Wi-Fi route using Keychain-backed credentials rather than plaintext passwords.
- Selected an EC502-derived fan policy after comparing original E07 behavior.
- Kept fan policy separate from macOS kext/ACPI installation requirements.

## 2026-09-28 to 09-30 — automatic S3→S4

- Moved from manual S4 forcing toward delayed automatic standby.
- Added PXSX to the HibernationFixup PCI patch scope.
- Rejected LID0/GPE16/virtual-LIDW and stock-E07-EC experiments for lid-open S4 wake.
- Passed a real overnight Battery S3→S4 test; deep S4 still required a short power-button press.
- Accepted AC ordinary sleep and Battery delayed S4 as distinct policies.

## 2026-10-14 to 10-17 — external display and long sleep

- Investigated USB-C/DisplayPort hotplug and DynamicCDCLK candidates.
- Accepted both USB-C data ports at 5 Gbps.
- Recorded another true Battery S4 cycle with a brief display mosaic.
- Corrected an invalid overnight S4 assumption where AC had been unplugged only after the sleep request.

## 2026-10-21 to 10-24 — OS decision and migration preparation

- Compared Ventura, Sonoma, Sequoia and Tahoe.
- Re-evaluated Broadcom native Wi-Fi against Intel/HeliPort and OCLP root patching.
- Final user choice returned to Ventura 13.7.8.
- Prepared a separate installer and Broadcom-only Ventura EFI while keeping Windows isolated.

## 2026-10-25 — Ventura installation and acceptance

- Installed Ventura 13.7.8.
- Re-established internal 1424×802 HiDPI and ColorSync.
- Accepted audio, headset microphone, touchpad, keyboard and F-key behavior.
- HDMI 4K30 restored external picture. DP 4K60 remained a historical/experimental path and was not accepted in the final production profile; see the dedicated case study.
- Replayed Sonoma S4 conditions on Ventura and completed a matched LPM0 A/B.
- Chose Battery high-charge 12h / low-charge 3h / LPM0, with AC S3-only.

## 2026-11-01 — public project and maintenance phase

- Closed active construction.
- Preserved the original EFI backup and rescue USB.
- Archived the exact final configuration and power receipts privately.
- Began publishing this sanitized engineering guide.
- Future work is observation-driven; no routine experimental S4 loops or EFI changes are planned.
