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

## Historical conversation archive and older compatibility material

The private MiBook macOS14 archive indexed on 2026-10-31 contains 9 main conversation threads and 8 subtask threads (two main entries were empty), alongside project workfiles and recovery evidence. The public guide uses sanitized engineering conclusions and maintained project records; it does not publish raw conversations, EFI archives or machine identity data.

A user-supplied historical OpenCore reference contains High Sierra 10.13.6 and Catalina installation/tutorial material. That is **generic historical compatibility reference**, not evidence that this particular laptop's currently documented project installed or ran those OS versions. The reproducible project record in this guide begins with the 2026-09-21 reactivation. Do not infer an earlier exact first-install date or version without a primary record. The same user-supplied OpenCore guide is from the OpenCore 0.5.6 era; it was treated only as a checklist of topics. Any setting carried into the current profile was rechecked against OpenCore 1.0.7 documentation, upstream KBL material and this machine's live evidence.

## Important source corrections

- A previous 2026-07-26 retirement decision was reversed on 2026-09-21. The current project is the subsequent reactivated engineering effort.
- The 2026-10-21 Sonoma recommendation was a point-in-time recommendation based on the then-required ChatGPT desktop app and newer security updates. The user made a later explicit decision on 2026-10-24 to return to Ventura 13.7.8, accepting browser-based workflows in exchange for the native Broadcom path and lower maintenance complexity. The later decision supersedes the earlier recommendation.
- An apparent 2026-10-16 overnight S4 was not a valid S4 test because AC was disconnected after the machine had already entered sleep. It was classified as “Wake from Deep Idle”, not “Wake from Standby”.
- A successful OpenCore config validation is not a physical acceptance. The validation matrix separates static config, runtime, and user-visible tests.
- The 2026-10-25 Ventura S4 LPM0 test proves LPM0 was sufficient for one matched 600/600-second cycle. It does not prove every threshold combination or long-term cycle is infallible.
- The final production policy was selected after the S4 tests: Battery high 12h / low 3h / LPM0; AC standby disabled. Do not replace this with the earlier Sonoma 3h/LPM1 policy without new evidence.

## Coverage limitations

The searchable archive preserves a detailed record from the 2026-09-21 project reactivation through the 2026-11-01 closeout. The original 2026-09-22 ChatGPT discussion was recovered as a summary rather than a complete verbatim transcript. Earlier installation work before the current archived project restart may not be fully represented.

A separate targeted history search for actual pre-2026-09-21 installations returned no matching conversation evidence. The only older item recovered was the generic upstream/user-supplied README described above; it is not an installation log. Therefore the earliest actual macOS version and install date on this physical machine remain unknown.

This guide therefore claims **best-effort reconstruction of the available MiBook project record**, not a complete forensic export of every historical installation or every conversation.

## Adjacent MiBook projects intentionally excluded

- Windows Dolby Atmos / DAX3 reverse engineering is a separate Windows audio project; it does not describe macOS ACPI, kext or OpenCore installation.
- The native Fn HUD helper was a separate AppKit utility and was retired on 2026-11-01; it is not part of the Hackintosh driver stack.
- R1 voice-assistant/audio routing and other homelab work are separate projects.

These records may share the physical laptop, but they must not be mixed into the Hackintosh installation baseline.

## Reproducibility boundary

The original private EFI, exact SMBIOS identity, BIOS/SPI/EC images, full panic dumps and raw recovery archives remain private. The public guide publishes the engineering conclusions and sanitized settings needed to understand and reproduce the work without exposing another person's machine identity or firmware image.
