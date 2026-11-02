# Graphics, internal panel, HiDPI and external displays

## UHD 620 baseline

- Intel UHD Graphics 620 is the supported macOS display GPU.
- NVIDIA MX150 is not used as the macOS display GPU; the ACPI path disables it.
- The accepted production framebuffer flags are `0x00C3070B` (DynamicCDCLK bit `0x200000` is off).
- `enable-max-pixel-clock-override=1` was retained for the tested connector/mode set.

## Internal panel

The reference panel is BOE NV156FHM-N61 with a real BOE EDID. The final display setup does not replace it with a fake Apple EDID.

Accepted display state on the final Ventura reference build:

- Logical HiDPI mode: 1424×802 @ 60 Hz.
- Backing resolution: 2848×1604 (2×).
- Display name: Color LCD.
- ColorSync: macOS-generated Color LCD ICC profile was verified as the active display color space.
- Text clarity and the graphical resolution picker were physically accepted by the user.

Chronology matters: immediately after the Ventura OS migration, the custom override directory was absent and the panel was temporarily at 1920×1080 1×. On 2026-10-25 the BOE-specific override was installed as a separate system-level change; after reboot the user physically accepted 1424×802 and ColorSync verified the generated Color LCD ICC as the active profile. This final acceptance supersedes the earlier pre-override inventory.

The custom override is a **system-level display configuration**, not an EFI setting. It must be regenerated for another panel's EDID; do not copy the BOE ProductID override to a different display.

## External display limits

The final Ventura production profile has **physical HDMI 4K30 acceptance**. Do not claim production-stable DP 4K60:

- Laptop HDMI: 3840×2160 @ 60 Hz was recognized by macOS but produced no visible image. 3840×2160 @ 30 Hz restored the picture. This is consistent with the TM1701 HDMI 1.4 path limit.
- USB-C / DisplayPort: one 4K60 session was historically recorded on 2026-09-29. Later, when trying to reproduce and stabilize 4K60, the DynamicCDCLK bit raised the KBL CD clock to 675 MHz and enabled a 4K60 mode, but this candidate was followed by a WindowServer watchdog/GPU hang in a dual-display test and DP link-training instability. The final profile reverted to `0x00C3070B`. See the [external DP case study](case-studies/external-display-dp.md).

If an external monitor is black:
1. identify the actual cable and physical port;
2. test a conservative mode such as 4K30 on HDMI;
3. confirm EDID and hotplug state;
4. only then inspect framebuffer connector properties.

Do not blindly copy connector patches or remove the pixel-clock override.

## Rejected graphics experiments

The following did not become part of the final profile:

- `gfxrst=1/3/4`: produced no durable improvement; some variants caused boot/display artifacts.
- `ProvideConsoleGop=True`: no improvement in the physical S4 mosaic.
- Clearing `FBAvoidFastLinkTraining` (flags `0x00C3070A`): reproduced/worsened mosaic flashes; restored `0x00C3070B`.
- Global force-online and custom complete-modeset framebuffers: removed.
- DynamicCDCLK: not promoted to production; no benefit justified its risk.

A 51 ms VBlank timeout log appeared in both visually clean and artifact cycles. It was therefore not a discriminating root-cause marker by itself. The 4K60 experiment is not a reason to turn DynamicCDCLK back on in the production profile.

## DVMT and HiDPI

The 64 MB DVMT change is described in [BIOS and EC](03-bios-and-ec.md). The higher 1520×855 mode was not pursued; 1424×802 was the accepted sleep-safe mode on this panel.
