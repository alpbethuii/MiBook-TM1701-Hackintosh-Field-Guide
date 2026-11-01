# Audio, microphone, touchpad and keyboard

## Audio — Realtek ALC298

Reference stack:

- AppleALC 1.9.7
- Lilu 1.7.2
- ALC298 layout 30

Physical acceptance on the reference Ventura build:

- Internal speakers: PASS.
- 3.5 mm headphone plug/unplug: PASS.
- Headset microphone: PASS.
- Built-in microphone: PASS after increasing input level from 50 to 80 and re-testing.

The config had more than one audio layout-related property visible in different parts of the IORegistry. The AppleALC-selected `alc-layout-id=30` was the authoritative value used for the working path. Do not switch to a generic layout number based only on a text property elsewhere in IORegistry.

ALCPlugFix was not installed because the exact headphone replug failure it targets was not reproducible.

## Touchpad — ETD2303

The working chain uses:

- SSDT-TPD0;
- VoodooI2C 2.8;
- VoodooI2CServices / VoodooGPIO / VoodooInput;
- VoodooI2CHID.

The IORegistry chain reached AppleMultitouchTrackpadHIDEventDriver and the expected multitouch device path. The project physically accepted ordinary gestures.

VoodooI2C 2.9.1 was a tested regression and was rolled back. See [kext inventory](06-kext-inventory.md).

## Internal keyboard and function keys

The internal keyboard uses VoodooPS2Controller 2.3.8 plus the model-specific SSDT-PS2K/LGPA logic. The physical F-key mapping was checked on the reference machine; F9/F10 behavior passed the acceptance performed for the main installation.

The key mapping is not merely a cosmetic macOS setting: the ACPI event path and PS/2 keyboard driver must agree. Do not add a generic remapper if the firmware/ACPI path already emits the intended events.

## What remains out of scope

The later experimental Fn HUD was a separate user-interface helper, not part of the Hackintosh driver stack. It was retired and is not needed for the final keyboard configuration.
