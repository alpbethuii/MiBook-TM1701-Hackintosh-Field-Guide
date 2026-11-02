# Changelog

## 2026-11-01 — Initial public engineering notebook

- Created a dedicated public repository for TM1701/TM1707 Kaby Lake Refresh.
- Reconstructed the 2026-09-21 to 2026-11-01 project timeline from maintained project records, hardware acceptance reports, OpenCore/ACPI audits and S4 case studies.
- Added the tested Ventura 13.7.8 baseline, BIOS/EC notes, OpenCore/ACPI/kext inventory, hardware validation matrix and rejected experiment log.
- Added the full S3→S4 case study, including early 0x27 failures, LID/GPE/EC wake experiments, Sonoma production acceptance and Ventura LPM0 A/B.
- No personal EFI, SMBIOS identity, BIOS/EC image, raw disk image or private log is included.

## 2026-11-02 — second-pass evidence and reproducibility update

- Corrected the external DP 4K60 status: historical pass retained as a case study, not a final production guarantee.
- Added detailed DP/CDCLK and EC fan-policy case studies.
- Added a complete sanitized build checklist tying BIOS, ACPI, kexts, DeviceProperties, OpenCore quirks, wireless and acceptance together.
- Added a symptom-to-evidence troubleshooting index for same-model users.
- Corrected nested VoodooI2C plugin versions from the archived Ventura EFI metadata.
- Expanded source provenance, wireless hardware topology, BIOS Write Lock chronology and current Apple OS security snapshot.
- Rechecked local links, sensitive-pattern boundaries and repository publication exclusions.
