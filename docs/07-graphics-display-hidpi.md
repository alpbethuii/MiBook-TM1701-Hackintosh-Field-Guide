# Graphics, internal panel, HiDPI and external displays

## UHD 620 baseline

- Intel UHD Graphics 620 is the supported macOS display GPU.
- NVIDIA MX150 is not used as the macOS display GPU; the ACPI path disables it.
- The accepted framebuffer flags are `0x00C3070B`.
- `enable-max-pixel-clock-override=1` was retained for the tested connector/mode set.

## Internal panel

The reference panel is BOE NV156FHM-N61 with a real BOE EDID. The final display setup does not replace it with a fake Apple EDID.

Accepted display state:

- Logical HiDPI mode: 1424×802 @ 60 Hz.
- Backing resolution: 2848×1604 (2×).
- Display name: Color LCD.
- ColorSync: macOS-generated Color LCD ICC profile was verified as the active display color space.
- Text clarity and the graphical resolution picker were physically accepted.

The custom override is a **system-level display configuration**, not an EFI setting. It must be regenerated for another panel's EDID; do not copy the BOE ProductID override to a different display.

## External display limits

The same monitor was observed on two distinct physical paths:

- Laptop HDMI: 3840×2160 @ 60 Hz was recognized by macOS but produced no visible image. 3840×2160 @ 30 Hz restored the picture. This is consistent with the TM1701 HDMI 1.4 path limit.
- USB-C / DisplayPort: a separate 4K60 path passed physical acceptance. Do not infer that HDMI and USB-C/DP share the same bandwidth or connector path.

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

A 51 ms VBlank timeout log appeared in both visually clean and artifact cycles. It was therefore not a discriminating root-cause marker by itself.

## DVMT and HiDPI

The 64 MB DVMT change is described in [BIOS and EC](03-bios-and-ec.md). The higher 1520×855 mode was not pursued; 1424×802 was the accepted sleep-safe mode on this panel.
