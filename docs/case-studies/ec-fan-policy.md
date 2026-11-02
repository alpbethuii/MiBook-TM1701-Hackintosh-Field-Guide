# Case study: EC fan policy — E07 BIOS, EC502 control loop and the 62°C decision

> This is optional hardware/firmware engineering. It is not required to install macOS, and the custom EC image is deliberately not published.

## Executive summary

The reference machine kept the main BIOS at **XMAKB5R0P0E07 (E07)** but moved the embedded-controller fan-control region to an official-EC502-derived implementation. The final user-accepted temperature policy is:

| Fan stage | Turn on (°C) | Turn off (°C) |
|---:|---:|---:|
| 1 | 62 | 58 |
| 2 | 66 | 63 |
| 3 | 72 | 67 |
| 4 | 78 | 73 |
| 5 | 84 | 79 |

The final decision was to keep this 62°C profile, not pursue 65°C thresholds or 1800–2000 RPM targets. The evidence did not justify increasing thermal thresholds or reducing the fan's minimum target speed.

## Hardware and firmware architecture

- Main BIOS: E07.
- EC base: official EC502-derived code.
- The custom work changed the temperature threshold tables and their checksum/alignment, not the closed-loop RPM target tables, PWM seed tables or the separate higher-temperature control path.
- The EC maintains its own temperature sampling and state machine. The primary fan-control temperature uses a moving window of approximately 20 samples; it is not the same measurement as the instantaneous CPU-die temperature reported by macOS.
- The EC reads tachometer feedback for both fans and adjusts PWM in a closed loop to stay within target RPM bands.
- The PWM seed values used during a stage transition are **not** the final steady-state PWM duty. The controller can adjust PWM after the initial transition.
- A separate higher-temperature path can request higher fan stages. The exact physical sensor identity and every protective condition of that path were not fully established, so this project does not claim that the main temperature curve alone represents every thermal safeguard.

These details explain why a number that looks plausible in a binary table is not automatically a safe physical fan speed.

## Chronology

### 2026-09-24 — E07 threshold experiments

The project first tested E07-based fan threshold modifications. The +8/+12-style threshold edits did not produce the desired low-load behavior after a proper EC reset. Those experiments were not carried forward as the production solution.

### 2026-09-25 — official EC502 base

The main BIOS remained E07 while the 128 KiB EC region was replaced with the official EC502-derived region. The write was read back and verified; a strong EC reset was required before the controller behavior changed.

After reset, the machine demonstrated a true 0-RPM idle state while retaining automatic fan response under load. This established that the physical fans could stop at idle and restart automatically; it did not yet establish the final quiet threshold policy.

### 2026-09-25 — custom quiet threshold candidate

The user wanted fewer interruptions and less frequent short fan cycles, not a permanently slow fan or blocked airflow. A custom threshold candidate was created to delay low-load activation and widen hysteresis while preserving a conservative top stage.

The project explicitly rejected blanket +5/+12/+20 changes across all stages because that would raise the high-load ceiling without a measured need.

### 2026-09-30 — 62°C profile accepted

A structured review compared temperature thresholds, fan acoustics, EC control-loop behavior and safety stop conditions. The final 62°C profile was selected:

- On: [62, 66, 72, 78, 84] °C
- Off: [58, 63, 67, 73, 79] °C

Representative controlled observations included a roughly 12 W package-load run lasting about 181.5 seconds, with CPU-die readings around 59–68°C. The first roughly 125 seconds were close to the acoustic background; a concentrated fan event occurred later, followed by cooling. The 75°C experimental stop line was not reached.

A separate roughly 10 W run had different ambient and software conditions. The two runs are not a valid direct comparison of room temperature or cooling capacity.

The result supported the user's preferred pattern: remain quiet under light work, then perform a more concentrated cooling event when the threshold is reached.

## Why not 65°C?

The review did not establish that a 65°C first-stage threshold would provide a meaningful user benefit without increasing thermal risk. CPU-die readings and the EC's smoothed internal sensor are different variables, so comparing the instantaneous CPU reading directly to the EC threshold would be misleading.

The final policy therefore retains 62°C. The project does not recommend raising it simply because a forum post or another laptop uses a higher number.

## Why not 1800–2000 RPM?

The EC code can represent numeric values in that range, but four different questions must be answered:

1. Can the firmware represent the requested target?
2. Can both physical fans reliably start from zero at that target?
3. Can they maintain that speed without stalling or oscillating?
4. Is the resulting airflow sufficient under sustained heat load?

The project did not complete those four physical validations. macOS also has no accepted, non-invasive live RPM readout for this EC. Therefore 1800–2000 RPM was not adopted.

## BIOS Write Lock note

The reference machine has an owner-specific BIOS Write Lock decision: after the initial firmware preparation had restored write protection, the user explicitly chose to leave the setting disabled during later EC work. This is **not** a Hackintosh requirement and must not be copied to another machine. Keep firmware write protection enabled by default unless a necessary, explicitly authorized firmware operation requires otherwise.

## Stop and rollback policy

The EC project froze the 62°C profile and did not authorize further automatic firmware writes. Any future EC change would require:

- a fresh readback and backup of the active EC region;
- verified rollback image and hash;
- an offline candidate audit;
- an exact write range and strong-reset plan;
- a separate explicit authorization for that specific EC write;
- temperature, fan-start, stall, oscillation, sleep/wake and panic stop criteria.

The experimental stop line used in controlled low-load tests was CPU-die ≥75°C. This is a test stop condition, not a published hardware maximum.

## Final decision

Keep **E07 main BIOS + the accepted EC502-derived 62°C profile**. Do not publish or flash the private EC image from this document. Do not use fan tuning as a prerequisite for a fresh Hackintosh installation.
