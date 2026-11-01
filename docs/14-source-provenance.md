# Source provenance and historical coverage

This repository was reconstructed from maintained engineering records, test reports and handoffs. It is not a verbatim export of private ChatGPT conversations.

## Main source families

| Topic | Historical source records used for reconstruction |
|---|---|
| Initial model identification and upstream audit | Upstream Xiaomi-Pro README/Docs, 2026-09-22 source audit |
| BIOS / DVMT / CFG Lock | 2026-09-22 BIOS audit and the project's BIOS preparation handoff |
| ACPI / OpenCore cleanup | ACPI-AUDIT-PHASE1 (2026-09-23), CONFIG-AUDIT (2026-09-23), EFI-GOLD (2026-09-24), decisions log |
| USB mapping | 2026-09-23 USBMap audit and later final-state record |
| AppleVTD / DMAR | AppleVTD DMAR preflight and retest records (2026-09-23) |
| Kext versions and enablement | Final Ventura candidate config and the archived pre-cutover EFI bundle metadata |
| Display / HiDPI / ColorSync | 2026-09-24 display acceptance, 2026-10-14 DP/CDCLK A/B, 2026-10-25 Ventura display acceptance |
| Audio / microphone / touchpad / keyboard | 2026-10-25 Ventura hardware acceptance and earlier live IORegistry audits |
| Wi-Fi / Bluetooth | 2026-09-22 wireless audit, 2026-09-25 hidden-SSID A/B, 2026-09-29 Bluetooth audit, 2026-10-21 Broadcom/OCLP review |
| EC / fan control | 2026-09-30 EC fan council evidence, hysteresis analysis and rollback rules |
| OS selection | 2026-10-21 macOS version re-decision and 2026-10-24 Ventura migration decision |
| S4 | 2026-09-23/24 S4 and framebuffer handoffs, 2026-09-28/30 automatic S4 evidence, 2026-10-17 S4 acceptance, 2026-10-25 Sonoma replay and Ventura LPM0 A/B |
| Final Ventura state | 2026-10-25 hardware acceptance, 2026-11-01 production closeout and live power-policy receipt |

## Important source corrections

- A previous 2026-07-26 retirement decision was reversed on 2026-09-21. The current project is the subsequent reactivated engineering effort.
- The 2026-10-21 Sonoma recommendation was a point-in-time recommendation based on the then-required ChatGPT desktop app and newer security updates. The user made a later explicit decision on 2026-10-24 to return to Ventura 13.7.8, accepting browser-based workflows in exchange for the native Broadcom path and lower maintenance complexity. The later decision supersedes the earlier recommendation.
- An apparent 2026-10-16 overnight S4 was not a valid S4 test because AC was disconnected after the machine had already entered sleep. It was classified as “Wake from Deep Idle”, not “Wake from Standby”.
- A successful OpenCore config validation is not a physical acceptance. The validation matrix separates static config, runtime, and user-visible tests.
- The 2026-10-25 Ventura S4 LPM0 test proves LPM0 was sufficient for one matched 600/600-second cycle. It does not prove every threshold combination or long-term cycle is infallible.
- The final production policy was selected after the S4 tests: Battery high 12h / low 3h / LPM0; AC standby disabled. Do not replace this with the earlier Sonoma 3h/LPM1 policy without new evidence.

## Coverage limitations

The searchable archive preserves a detailed record from the 2026-09-21 project reactivation through the 2026-11-01 closeout. The original 2026-09-22 ChatGPT discussion was recovered as a summary rather than a complete verbatim transcript. Earlier installation work before the current archived project restart may not be fully represented.

This guide therefore claims **best-effort reconstruction of the available MiBook project record**, not a complete forensic export of every historical installation or every conversation.

## Reproducibility boundary

The original private EFI, exact SMBIOS identity, BIOS/SPI/EC images, full panic dumps and raw recovery archives remain private. The public guide publishes the engineering conclusions and sanitized settings needed to understand and reproduce the work without exposing another person's machine identity or firmware image.
