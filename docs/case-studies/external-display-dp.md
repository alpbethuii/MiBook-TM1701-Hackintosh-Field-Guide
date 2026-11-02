# Case study: external DP 4K60 — a historical success that did not become the production baseline

> **Final decision:** HDMI 4K30 is physically accepted. DP 4K60 was recorded in a historical session. A later reproducible 4K60 candidate used DynamicCDCLK, but that candidate was not accepted as a stable daily-use profile. The final Ventura candidate keeps **framebuffer-flags=0x00C3070B** with DynamicCDCLK disabled.

## Executive summary

The MiBook's UHD 620 can expose a working 4K60 path under the right Kaby Lake display-clock configuration. That does **not** mean the path is stable across boot, sleep/wake, monitor link training and long-running WindowServer composition.

The project deliberately chose the lower-risk final profile:

- framebuffer-flags = **0x00C3070B** (DynamicCDCLK bit off);
- enable-max-pixel-clock-override = **1** retained;
- external Skyworth XG27U1 physically accepted at 3840×2160@30 on the final Ventura installation;
- no production promise of DP 4K60.

## Timeline

### 2026-09-29 — one historical 4K60 pass

A maintained historical record documents a normal S3/Deep Idle wake followed by a true 3840×2160@60 external mode:

- pixel clock approximately 533.28 MHz;
- 2× HiDPI desktop;
- PIPE A/B/C underrun counters at zero.

This is evidence that a 4K60 session was possible at least once. The surviving record does not establish a single configuration change as the cause, and it does not prove long-term stability.

### 2026-10-14 — the problem became reproducible

The same XG27U1 later showed a black/no-signal condition:

- HDMI/adapter route and a separately tested full-feature USB-C route both failed to provide a stable physical image in that session;
- macOS fell back to 3840×2160@30 (297 MHz);
- the external display's Pipe B repeatedly reported underruns, while internal Pipe A remained clean;
- lowering the external mode to 2560×1440@60 did not eliminate the underrun pattern.

The project therefore did not attribute the failure to one adapter, cable or simple GPU-load ceiling.

### 2026-10-14 — the actual clock-limit distinction

The machine had enable-max-pixel-clock-override=1, which raised the reported IOFB timing range to 675 MHz. However, the Kaby Lake display core still reported a 337 MHz CD clock limit and rejected the 533/594 MHz 4K60 timings.

These are different checks:

- **IOFB timing range override:** allows the framebuffer timing validator to consider a higher pixel clock.
- **Actual CDCLK:** the display core's clock limit; the override alone does not raise it.

The kernel explicitly rejected the 533.28 MHz mode because the platform CD clock was 337 MHz.

### 2026-10-15 — DynamicCDCLK opened the 4K60 path

A one-bit A/B changed the IGPU framebuffer flags:

| Candidate | Value |
|---|---:|
| Production baseline | 0x00C3070B |
| DynamicCDCLK candidate | 0x00E3070B |
| XOR difference | 0x00200000 |

With the DynamicCDCLK bit enabled:

- the kernel reported Using 675Mhz CdClk;
- macOS accepted 3840×2160@60 at 594 MHz;
- PIPE A/B/C underruns were zero in the short samples;
- a CoreGraphics mode switch to 4K60 completed and restored correctly.

The physical display then recovered after the user power-cycled the Skyworth monitor. The successful physical route in this DynamicCDCLK trial was MiBook USB-C → Xiaomi ZJQ02TM adapter → miniDP-to-DP → XG27U1 DP input. A later correction explicitly ruled out the Thunderbolt 5 direct USB-C cable as the path used for this particular physical success.

### 2026-10-15 — why the candidate was not accepted

A dual-display period with DynamicCDCLK enabled was followed by a WindowServer watchdog timeout and a kernel panic involving the Intel graphics/IOAccelerator path. The evidence strongly implicated a GPU/WindowServer hang, but did not prove that the clock bit alone was the only cause.

A later single-display, lid-closed OpenGL stress test ran for approximately 30 seconds at high GPU utilization with zero PIPE underruns and no new panic. This was a useful short pass, **not** a 20-minute stability pass or a complete daily-use acceptance.

The external DP sink also entered a no-signal/link-training state after a later restart. A monitor power-cycle restored lane training and physical picture. This is another reason not to treat system_profiler reporting “Online” or a 4K60 mode as proof of a stable physical link.

### 2026-10-17 — S4 was a separate acceptance dimension

With the experimental DynamicCDCLK profile still active, the machine completed a battery S3→S4/Standby cycle and resumed the original session. The user saw a brief black/white mosaic before the login screen and accepted it as a known visual imperfection for that test.

This did not establish that DynamicCDCLK was the safest graphics configuration. S4 success, physical display recovery and long-duration GPU stability are separate acceptance dimensions.

### 2026-10-25 — final Ventura production decision

The Ventura production candidate returned to framebuffer-flags=0x00C3070B. The external XG27U1 was physically accepted at 4K30. The DynamicCDCLK bit was not carried into the final baseline.

## Final technical interpretation

1. enable-max-pixel-clock-override=1 is not equivalent to enabling DynamicCDCLK.
2. The 0x00200000 flag can raise the actual KBL CD clock to 675 MHz on this machine and permit a 4K60 timing.
3. A mode being accepted by CoreGraphics, a display being reported Online, or a zero underrun counter is not enough to prove the physical monitor has a stable image.
4. A short GPU stress pass does not prove long-duration WindowServer stability.
5. A graphics change must also pass ordinary sleep, S4 resume, monitor retraining and repeated cold boot before it can be promoted.

## Final decision

- Keep 0x00C3070B in the final production profile.
- Keep the existing max-pixel-clock override; it serves a separate purpose.
- Use 4K30 as the accepted external display mode on the final reference setup.
- Do not re-enable DynamicCDCLK merely to recover 4K60 without a new hypothesis, complete rollback pair (config + Vault), physical monitor observation and a separately approved stability test.

The rejected candidate and its panic/display evidence remain private. This public chapter preserves the reasoning, not the personal EFI or crash dumps.
