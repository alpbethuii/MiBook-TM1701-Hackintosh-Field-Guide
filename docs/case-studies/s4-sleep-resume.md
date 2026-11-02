# Case study: making S3 → S4 work on the TM1701

> This is the central engineering record of the project. It deliberately preserves failed tests and corrections. The result is a practical compromise, not a claim that this laptop behaves exactly like a MacBook.

## Executive summary

The TM1701 can enter deep S4/Standby after a battery sleep delay and resume the original macOS session through OpenCore. The difficult part was not simply entering hibernation: it was reliably restoring PCI/DMA state, keeping the physical panel usable, preserving the same kernel session, and understanding that the embedded controller does not wake the laptop from deep S4 on lid-open alone.

The final policy is:

| Power source | Policy |
|---|---|
| Battery, high charge | S3 first; enter S4 after 12 hours |
| Battery, low charge | S3 first; enter S4 after 3 hours |
| Battery low-power mode | Off (`lowpowermode=0`) |
| Battery hibernation | `hibernatemode=3`, `standby=1` |
| AC | Ordinary sleep; `standby=0` |

When deep S4 has been entered, opening the lid alone does not wake the machine. A short power-button press triggers firmware/OpenCore and resumes the original macOS session. A brief upper-black/lower-mosaic flash has been observed on some resumes; it was accepted as a known imperfection when it clears immediately and the session resumes without panic.

The `highstandbythreshold=50` setting selects the high-delay path above the 50% battery threshold and the low-delay path at/below it. The long 12-hour high-charge delay is intentional: frequent S4 entry saves little extra energy compared with preserving a fast S3 resume for normal daily work. The user explicitly preferred continuity of open projects over marginal battery savings.

## What counts as a real S4 pass?

A valid S4 pass requires all of the following:

1. Battery power was confirmed **before** the sleep request.
2. The log shows a real clamshell/ordinary sleep entry.
3. After the configured delay, the log shows RTC/Maintenance DarkWake followed by Maintenance Sleep.
4. The later event is explicitly `Wake from Standby`, not merely `Wake from Deep Idle`.
5. The original kernel boot session remains the same; it was not a cold reboot.
6. No new kernel panic or Sleep Wake Failure is generated.
7. The physical panel reaches a usable login/desktop, and keyboard/input/network are available.
8. The test script restores every temporary pmset value and the restored values are independently read back.

A `Wake from Deep Idle` is not S4. A power button that causes a full cold boot is not a successful hibernate-image resume. A normal desktop screenshot is not proof that the physical LCD panel was clean.

## Timeline and reasoning

### 2026-09-23 — first failures: mode 25 and 0x27

The early forced-S4 attempts produced repeated `0x00000027` wake-timeout panics. Stack evidence included power-state restoration, IONVMe/IODMACommand/dmaMap and IOMapper-related waits. Some failures also left the physical panel black, cyan or split-screen mosaic.

Key correction: a normal Deep Idle/RTC wake passing did not prove S4 worked. USB mapping was also not the root cause: a later 0x27 panic occurred with the new precise USBMap under an explicitly forced S4 test.

### 2026-09-24 — isolate hibernation, PCI and display variables

The investigation compared:
- HibernationFixup on/off;
- AppleVTD / DisableIoMapper states;
- ForceBooterSignature and ResetHwSig;
- HibernationFixup PCI patch scope;
- reserved-memory handling;
- framebuffer recovery behavior.

Some combinations completed a true S4 resume, but later tests on the then-current EFI still reproduced 0x27 or physical display corruption. The project therefore did not declare success from one clean wake.

A critical outcome was to stop treating the issue as “just a graphics problem” or “just AppleVTD”. DMA/IOMMU, PCI power-state restore, hibernation metadata and display handoff could interact. The logs did not justify a single-cause claim.

### 2026-09-28 to 2026-09-29 — automatic standby and the PXSX correction

The project moved from manually forcing mode 25 toward the normal macOS model: hibernatemode 3 with standby enabled after a delay.

A key HibernationFixup boot-argument change expanded the PCI patch list from:

```text
XHC,IMEI,IGPU
```

to:

```text
XHC,IMEI,IGPU,PXSX
```

With `hbfx-ahbm=129` and the final list, controlled automatic S3→S4 cycles completed. Three short cycles restored the original session, but all showed a transient physical display artifact; one cycle also recorded a Bluetooth maintenance-wake crash and a HID sleep timeout. Those cycles were useful evidence, not yet full production acceptance.

### 2026-09-29 — the lid-open wake hypothesis was tested and rejected

After true S4 entry/resume was demonstrated, the remaining MacBook-like behavior request was: close lid, enter deep S4, open lid and wake without touching the power button.

The project tested the firmware/ACPI wake semantics separately:

| Candidate | Test result | Decision |
|---|---|---|
| Change `LID0._PRW` wake depth from S3 to S4 | Did not enable lid-open wake; a later wake also hit 0x27/display corruption | Rejected |
| Replace LID0 wake GPE with GPE16 | Normal clamshell state broke; macOS did not recognize the lid as closed | Rejected |
| Add a separate virtual LIDW device using GPE16 | Also broke clamshell semantics | Rejected |
| Restore stock E07 EC instead of EC502-derived EC | Normal clamshell remained, but lid-open still did not wake deep S4 | Rejected |

The BIOS `Panel Open Wake Up` setting was already enabled. The separate USB charging S4/S5 option was not the lid-wake control.

Final conclusion: the EC/firmware path does not provide a reliable lid-open wake event from deep S4 on this reference TM1701. Further LID0/GPE16/stock-EC experiments were stopped.

### 2026-09-29 → 2026-09-30 — real overnight production acceptance

The important acceptance was not a short artificial cycle:

- Battery sleep started at approximately 20:49.
- Around 23:49 (three hours), RTC/Maintenance DarkWake occurred and immediately entered Maintenance Sleep / deep standby.
- At 07:05 the next morning, a short power-button action led to `Wake from Standby`.
- The same macOS session resumed.
- No new panic or Sleep Wake Failure was recorded.
- Battery moved from approximately 79% to 77% over the overnight closed-lid period.

This established a real Battery 3-hour S3→S4 path. AC remained S3-only.

### 2026-10-16 — an apparent overnight S4 that was not S4

The user unplugged AC **after** macOS had already entered sleep. The sleep request therefore used the AC profile, where `standby=0`. The next morning the log showed `Wake from Deep Idle`, not `Wake from Standby`.

This was correctly reclassified as **not an S4 test**. It demonstrates why power source must be confirmed before the sleep request, not inferred from the cable state many hours later.

### 2026-10-17 — another true three-hour battery S4

The log showed:

- 00:44:55: entering sleep on battery;
- 03:44:58: RTC/Maintenance DarkWake after the three-hour delay;
- Maintenance Sleep;
- 07:23:08: `Wake from Standby`, WakeTime about 0.703 s;
- same kernel boot session, no new kernel panic.

The user observed a brief black/white mosaic flash before the login screen, then normal operation. The user accepted this as a known visual imperfection for that test.

### 2026-10-25 — replaying Sonoma's successful method on Ventura

The OS was migrated to Ventura 13.7.8. The team first compared the exact Sonoma test conditions rather than blindly reusing a single delay.

**Attempt 1: only high-delay was shortened.**

- Battery `standbydelayhigh=600`;
- `standbydelaylow=10800` remained unchanged;
- `lowpowermode=0`.

After roughly 13m39s, the event was `Wake from Deep Idle`, not `Wake from Standby`. The test did not prove S4 entry.

**Attempt 2: exact Sonoma 10-minute profile.**

- Battery high=600 and low=600;
- `lowpowermode=1`;
- standby=1, hibernatemode=3;
- AC unchanged.

At 608 seconds the system logged RTC/Maintenance DarkWake, immediately entered Maintenance Sleep, then later logged `Wake from Standby`. The same original session resumed through OpenCore/power-button. No new panic was found. A brief upper-black/lower-mosaic flash remained.

**Attempt 3: matched LPM0 A/B.**

The same high=600/low=600 test was repeated with only `lowpowermode=0`. It also entered Maintenance DarkWake → Maintenance Sleep and resumed from Standby. The original session remained alive; no new panic was found. The display artifact flashed twice in this cycle.

This proves that low-power mode 1 was **not necessary** for one successful Ventura S4 cycle under the matched 600/600 test. It does not prove that every delay combination or every long-duration condition behaves identically.

All temporary test values were restored to the original 86400/10800/LPM0 values after each test.

## Exact acceptance snapshots

The maintained project logs preserve these representative cycles:

| Date / phase | Sleep entry | S4 transition | Resume | Notes |
|---|---|---|---|---|
| 2026-09-29 short Sonoma test | 15:40:29, Battery clamshell; high=600 / low=600 / LPM1 | 15:50:31, RTC/Maintenance DarkWake → Maintenance Sleep | 15:59:06, Wake from Standby | Same boot session; WakeTime about 0.683 s |
| 2026-09-29 second short Sonoma test | 16:26:58 | 16:37:01, Maintenance transition | 16:43:17, Wake from Standby | WakeTime about 0.710 s |
| 2026-09-29/30 production overnight | ~20:49, Battery clamshell | ~23:49, after 3 h | 07:05:24, Wake from Standby | rd=111 ms, WakeTime=0.741 s; battery 79%→77%; no new panic |
| 2026-10-17 production-like test | 00:44:55, Battery clamshell | 03:44:58, after 3 h | 07:23:08, Wake from Standby | WakeTime about 0.703 s; brief mosaic accepted |
| 2026-10-25 Ventura replay, LPM1 | 20:10:47, Battery clamshell | 20:20:55, +608 s | 20:44:41, Wake from Standby | rd=140 ms, WakeTime=3.254 s; same boot session; brief mosaic |
| 2026-10-25 Ventura replay, LPM0 | 21:31:22, Battery clamshell | 21:41:30, +608 s | 21:49:08, Wake from Standby | rd=125 ms, WakeTime=3.264 s; same boot session; mosaic flashed twice |

The Sonoma production policy before the Ventura migration used a three-hour Battery S3→S4 delay (high and low both 10800 seconds), `lowpowermode=1`, and AC `standby=0`. The final Ventura policy deliberately differs: high-charge 12 hours, low-charge 3 hours, `lowpowermode=0`, AC `standby=0`. The difference reflects a later user decision about work-session continuity and normal performance, not an assumption that the two OS versions have identical power management.

## Bluetooth wake faults were tracked separately

The 2026-09-29 deep audit found seven historical bluetoothd crash reports across several different signatures: some were `dispatch_block_cancel` faults, others were CoreBluetooth queue application faults, and one was a main-thread segmentation fault. A separate HIDProfile `prepareForSleep` transition timed out during maintenance sleep. These reports were not proven to share one cause and were not evidence that Bluetooth caused the 0x27 IONVMe/IODMA/IOMapper panic or the framebuffer mosaic.

A later Sonoma battery S4 test with Bluetooth enabled passed the actual Standby transition and same-session resume, although a transient bluetoothd resource spike was separately observed. The final Ventura profile uses the Broadcom native stack and passed an actual AirDrop file transfer; long-term reconnect behavior for every third-party Bluetooth peripheral was not individually accepted. Do not merge Bluetooth daemon faults, S4 entry/resume and display recovery into one root-cause claim.

## Why the final policy is 12h / 3h / LPM0

The final decision was based on the actual use case, not on copying Apple's exact thresholds:

- S3 resume is fast and preserves the current work session with minimal friction.
- Deep S4 on this laptop requires a power-button press and an OpenCore resume path.
- The S4 path has a known transient display artifact and a historical 0x27 failure history.
- Low-power mode 1 is not necessary for a successful matched Ventura S4 test and can constrain normal performance.
- The user values preserving the desktop/work session more than saving the last 1–2% battery during a normal overnight sleep.

Therefore:
- high charge waits 12 hours before S4;
- low charge waits 3 hours;
- lowpowermode stays 0;
- AC stays ordinary sleep with standby disabled.

This is a user-specific, evidence-backed compromise for the reference machine, not a universal recommendation for every TM1701.

## What not to repeat

- Do not count `Wake from Deep Idle` as S4.
- Do not test while the machine is actually on AC and later infer it was on battery.
- Do not shorten only one delay and assume the other one is irrelevant.
- Do not reintroduce the rejected LID0/GPE16/LIDW patches.
- Do not re-run gfxrst / ProvideConsoleGop / FastLink experiments without new physical evidence.
- Do not run repeated artificial short S4 cycles just to “prove it again”.
- Do not claim a clean S4 experience if the physical display is still visibly corrupted or a new panic occurs.

## Final production profile

```text
Battery (2026-11-01 live receipt):
  hibernatemode       3
  standby             1
  standbydelayhigh    43200
  standbydelaylow     10800
  highstandbythreshold 50
  lowpowermode        0
  powernap            0
  proximitywake       0
  sleep               1

AC (2026-11-01 live receipt):
  hibernatemode       3
  standby             0
  standbydelayhigh    86400
  standbydelaylow     10800
  lowpowermode        0
  powernap            1
  proximitywake       1
  sleep               0
```

The full pmset receipt also includes display sleep, network, TCP keepalive, disk sleep and other platform values; this excerpt lists the power-management fields most relevant to S3→S4. Do not copy the profile without checking the target OS and firmware.

