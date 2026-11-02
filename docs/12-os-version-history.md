# macOS version history and why Ventura became the final reference

The OS choice changed more than once. The important lesson is to preserve the reasoning at the time rather than rewriting history as if the final choice had always been obvious.

## Current Apple security snapshot (checked 2026-11-02)

This section is a current-security warning, not a rewrite of the historical OS decision:

- Ventura 13.7.8 is still the reference build, but Apple's security page lists it as released **2025-08-20**; no later Ventura security release appears in the current list.
- Sonoma 14.8.9 is listed as released **2026-08-06**.
- Sequoia 15.8.2 and Tahoe 26.7.2 are listed as released **2026-10-26**.
- macOS Golden Gate 27.2 is also listed, but Apple's compatibility list limits it to Apple-silicon Macs; it is not a viable OS target for this Intel TM1701.

Therefore Ventura 13.7.8 remains the **historically tested reference**, not the safest current OS recommendation for a new internet-connected install. A newer Intel-compatible OS would require its own KBL EFI, wireless, audio, display and sleep acceptance. Source: [Apple Security Releases](https://support.apple.com/en-us/100100).

## Decision timeline

| Date | Candidate / decision | Reasoning |
|---|---|---|
| 2026-09-21 | Project reactivated; initial target Ventura 13.6.x | Stability, lower resource demand, and the physically installed Broadcom BCM943602CS offered a clean native Apple wireless path |
| 2026-09-22 | Sonoma 14.8.9 chosen for a period | The user wanted the then-current ChatGPT/Codex macOS desktop apps, which required macOS 14 or later |
| 2026-09-22 to 09-25 | Sonoma Intel Wi-Fi route refined | Intel AC8265 AirportItlwm could not reliably join the hidden networks; itlwm + HeliPort + Keychain helper solved the hidden-SSID requirement |
| 2026-09-24 onward | Sonoma continuity ceiling identified | The Intel Wi-Fi route did not expose the expected AWDL interfaces; full AirDrop/Handoff/Universal Clipboard could not be assumed |
| 2026-10-21 | Full 13/14/15/26 re-evaluation | Sonoma retained better app compatibility and current security updates; Ventura had the cleaner Broadcom-native architecture; Sequoia/Tahoe offered no proven advantage on this hardware |
| 2026-10-24 | Final user decision changed back to Ventura 13.7.8 | The user accepted browser-based ChatGPT/Codex workflows and prioritized native Broadcom/iPad continuity, stability and the older machine's resource budget |
| 2026-10-25 | Ventura 13.7.8 installed and accepted | The final reference OS became Ventura 13.7.8 (22H730) |

## Why not simply stay on Sonoma?

Sonoma 14.8.9 had important advantages:
- It had been the production system for the extensive September sleep, display, USB, Wi-Fi and fan work.
- It met the desktop ChatGPT/Codex application requirement at that time.
- It had a newer security baseline than Ventura 13.7.8 in late 2026.

But the tested Intel Wi-Fi/HeliPort path did not expose the expected AWDL continuity path. The alternative Sonoma Broadcom route required OCLP root patching, with SIP/security-policy and update-maintenance costs. OCLP does not officially support Hackintosh root patching.

The September 22 Broadcom-only EFI was an early candidate staged for external testing, not a completed full Continuity acceptance. The later 2026-10-25 Ventura installation supplied the stronger final evidence: native Broadcom Wi-Fi/Bluetooth were active and the user completed a real AirDrop file transfer. That does not imply every Continuity feature was tested.

## Why Ventura was selected in the end

The final decision prioritized:
- the native Broadcom BCM943602CS Wi-Fi/BT path;
- AirDrop and Apple-device continuity, with actual AirDrop file transfer later accepted;
- avoiding OCLP root patches and extra legacy Wi-Fi stacks;
- the i7-8550U / 8 GB resource envelope;
- stability and a smaller maintenance surface.

The user accepted browser workflows for ChatGPT/Codex rather than requiring the desktop apps on the MiBook itself.

## Why not Sequoia 15 or Tahoe 26?

They were reviewed, not ignored:
- the upstream Xiaomi notes say the Intel AirportItlwm path is not supported on Sequoia and later; the alternate itlwm/HeliPort path is not a native Apple wireless solution;
- community Broadcom/OCLP support exists but is not a guarantee of complete continuity or stable sleep on this exact laptop;
- Tahoe introduces further SMBIOS, audio, boot and unsupported-model compatibility questions;
- a newer macOS version does not make the MX150 usable or improve the UHD 620 hardware capability.

They were not selected because there was no demonstrated gain that outweighed the new variables.

## Security caveat: a frozen reference is not a current recommendation

Apple's Ventura 13.7.8 security-content page lists its release date as 2025-08-20. By late 2026, this is an old security baseline. This repository documents the user's chosen and tested configuration; it does not recommend Ventura 13.7.8 as the safest OS for every new installation.

Before installing today:
1. Check Apple's current security release list.
2. Check whether the desired applications still support the OS.
3. Evaluate a newer supported OS with the same evidence discipline.
4. Do not assume that a bootable EFI implies a complete hardware experience.

## References

- [Apple security content for Ventura 13.7.8](https://support.apple.com/en-us/124929)
- [Apple security releases](https://support.apple.com/en-us/100100)
- [Xiaomi upstream project](https://github.com/daliansky/XiaoMi-Pro-Hackintosh)
- [OCLP project and support boundary](https://github.com/dortania/OpenCore-Legacy-Patcher)
