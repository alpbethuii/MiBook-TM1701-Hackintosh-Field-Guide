# BIOS, DVMT, CFG Lock and EC notes

## Reference firmware

- Main BIOS recorded on the reference TM1701: **XMAKB5R0P0E07 (E07)**.
- The project also tested E07 and EC502-derived embedded-controller behavior separately.
- BIOS and EC firmware are not interchangeable across laptop variants.

## One-time DVMT / CFG Lock preparation

On 2026-09-22 the reference machine completed the Xiaomi KBL `DVMT_and_0xE2_fix` preparation:

| Setting | Recorded result | Why it mattered |
|---|---|---|
| iGPU DVMT pre-allocation | 64 MB | Supports the tested UHD 620 framebuffer / HiDPI path |
| CFG Lock / MSR 0xE2 | Unlocked | Allows native XCPM configuration without relying on a software lock workaround |

The machine returned to Windows successfully after the firmware operation, and subsequent macOS display behavior supported the 64 MB setting.

**This was a one-time operation. Do not run the tool again on a machine that already has the target state.** Firmware modification can brick a laptop or reset unrelated firmware settings. Use the exact model-specific source, a full backup, stable power and a recovery plan.

The project intentionally does not publish the reference machine's SPI image or a raw firmware patch.

## Panel-open wake option

The firmware variable corresponding to `Panel Open Wake Up` was read from the machine-specific SPI snapshot and was already enabled. Changing it did not solve S4 lid-open wake. Do not assume the option controls deep-S4 wake depth; the separate USB charging S4/S5 option was shown by IFR to be unrelated.

## BIOS Write Lock

The project history contains an owner-specific decision to leave a firmware write-protection setting at `0x00` during custom EC work. This is **not a generic Hackintosh requirement** and is not part of the public installation recipe. For a new installation, preserve firmware write protection unless a documented, necessary firmware operation explicitly requires otherwise, then restore it.

## EC / fan control — optional hardware engineering, not an install prerequisite

The reference machine's original E07 EC fan behavior was considered too noisy. A later controlled comparison showed that an EC502-derived strategy could provide true 0-RPM idle while preserving automatic cooling under load.

The accepted custom threshold profile recorded on 2026-09-30:

- Fan-on thresholds: `[62, 66, 72, 78, 84] °C`
- Fan-off thresholds: `[58, 63, 67, 73, 79] °C`

This is an owner-specific EC policy, not a drop-in recommendation. The EC's internal state machine, averaging, tachometer feedback and PWM closed loop mean the table values must not be interpreted as simple fan percentages. A PWM seed is not necessarily the steady-state PWM duty.

The project rejected:
- further E07 threshold-table edits after a proper reset did not change the behavior;
- blanket `+5/+12/+20` changes across every threshold;
- lowering the first fan stage directly to 1800–2000 RPM;
- raising the top thermal threshold to 65 °C without a measured need;
- installing a custom RPM-reporting SSDT/kext merely to expose a sensor value.

**Do not flash an EC image based on this document.** No firmware binary is published. The macOS installation can be reproduced without this optional fan customization.

## Firmware changes that should not be copied blindly

- Do not disable `DisableRtcChecksum` experiments casually: on this platform a bad RTC/CMOS interaction may reset BIOS settings, including DVMT/CFG Lock.
- Do not change EC regions based only on a guessed offset. The recorded EC region boundaries were verified against the actual image; the end boundary is not the start address.
- Do not assume an E07 EC and an EC502 EC have equivalent non-fan behavior.
