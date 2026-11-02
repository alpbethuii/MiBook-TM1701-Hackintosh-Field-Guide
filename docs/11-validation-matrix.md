# Validation matrix

Use this matrix after building a machine-specific EFI. The status column is the reference TM1701 result, not a guarantee for other units.

| Subsystem | Reference status | Evidence / remaining boundary |
|---|---|---|
| Normal boot | PASS | Ventura 13.7.8 booted from internal EFI |
| OpenCore picker hidden | PASS | Physical reboot directly entered macOS |
| Windows dual boot | PASS with boundary | Windows remains on separate NVMe; do not write its ESP |
| UHD 620 / Metal 3 | PASS | Runtime acceleration and display |
| MX150 | Disabled | Not a supported macOS display path |
| Internal BOE panel | PASS | Real EDID retained |
| HiDPI 1424×802 | PASS | 2848×1604 backing; ColorSync Color LCD profile verified |
| HDMI 4K30 | PASS | Physical picture restored and accepted at 4K30 |
| USB-C / DP 4K60 | HISTORICAL SESSION / NOT PRODUCTION-ACCEPTED | DynamicCDCLK enabled a 4K60 session, but later WindowServer watchdog and DP link instability caused rejection from the final profile; final flags are `0x00C3070B` |
| ALC298 speakers | PASS | User physical test |
| Headset plug/unplug | PASS | User physical test |
| Headset microphone | PASS | User physical test |
| Built-in microphone | PASS after gain adjustment | Input gain increased from 50 to 80 |
| ETD2303 touchpad | PASS | I²C/GPIO event chain and gestures |
| Internal keyboard / F keys | PASS | Physical key mapping test |
| Broadcom Wi-Fi | PASS | Native Apple stack |
| Broadcom Bluetooth | PASS | Controller enumerated and enabled |
| AirDrop | PASS | Actual file transfer |
| USB-C data | PASS | Both ports 5 Gbps |
| USB mapping | PASS | Precise 11-port USBMap |
| Battery S3→S4 | PASS with limitations | 12h/3h policy; power-button resume |
| S4 lid-open wake | FAIL / hardware boundary | Lid alone does not wake deep S4 |
| S4 display transition | DEGRADED | Brief mosaic flash can occur |
| SD reader media read/write | UNVERIFIED | Reader enumerated; no accepted SD media test |
| Long-term thermal stability | Observation only | EC fan policy has controlled tests, not every ambient/load case |
| All third-party peripherals | UNVERIFIED | Must be tested per device |

## Suggested acceptance sequence

1. Cold boot and ordinary restart.
2. Internal panel, brightness, HiDPI and ColorSync.
3. Speakers, internal microphone, headset microphone, jack replug.
4. Touchpad gestures, internal keyboard and function keys.
5. Wi-Fi reconnect, Bluetooth pairing and AirDrop.
6. Every USB port and USB-C orientation with known-good cables.
7. Ordinary S3 sleep/wake.
8. Only after a complete backup and a separate safety review, controlled battery S4.
9. Verify same boot session and no new panic.
10. Restore all temporary power settings and independently read them back.
