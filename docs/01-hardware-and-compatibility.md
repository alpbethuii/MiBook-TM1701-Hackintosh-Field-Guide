# Hardware baseline and compatibility boundaries

## Tested reference machine

| Component | Reference configuration | Notes |
|---|---|---|
| Model | Xiaomi Mi Notebook Pro 15.6-inch TM1701 | Kaby Lake Refresh platform |
| CPU | Intel Core i7-8550U | 4 cores / 8 threads |
| Memory | 8 GB | Do not assume every unit has the same capacity |
| iGPU | Intel UHD Graphics 620 | Primary and supported macOS graphics path |
| dGPU | NVIDIA GeForce MX150 | Not supported as the macOS display GPU; disabled in the macOS ACPI path |
| Internal panel | BOE NV156FHM-N61, 1920×1080 | Preserve real EDID |
| Audio codec | Realtek ALC298 | AppleALC layout 30 validated |
| Touchpad | ETD2303 I²C/GPIO | VoodooI2C 2.8 path |
| Keyboard | Internal PS/2 keyboard path | VoodooPS2 + model-specific ACPI key handling |
| Original Wi-Fi | Intel Wireless-AC 8265 | Historical itlwm/HeliPort route |
| Final Wi-Fi/BT | Broadcom BCM943602CS / BCM43602 + BCM20703A1 | Native Apple stack on the final Ventura build |
| Storage | Samsung PM961 NVMe for macOS; separate Windows NVMe | Disk numbering can change across boots |

## Model families — do not mix

The upstream Xiaomi-Pro project distinguishes:

- **TM1701 / TM1707**: 8th-generation Intel i5-8250U / i7-8550U, Kaby Lake Refresh (KBL-R), UHD 620.
- **TM1905 / TM1963**: 10th-generation i5-10210U / i7-10510U, Comet Lake (CML), UHD 620.

The shared product name does not mean shared ACPI, EC, BIOS, trackpad, audio or sleep behavior. The CML deep-sleep notes and patches must not be copied to TM1701 without independent validation.

## Variants that require a fresh audit

Before using this guide, record:

- Exact model printed on the chassis / DMI model.
- CPU generation and iGPU PCI ID.
- Panel EDID vendor/product and panel model.
- Audio codec.
- Touchpad vendor and bus (I²C/GPIO vs PS/2).
- Wi-Fi and Bluetooth card model.
- NVMe controller models and physical disk layout.
- BIOS version and EC version.

Do not assume a TM1701 with a different panel or wireless card is identical to the reference machine.

## Compatibility verdict

| Feature | Reference unit | Other same-model units |
|---|---|---|
| Ventura 13.7.8 boot | PASS | Must validate BIOS/EFI and storage |
| UHD 620 acceleration | PASS | Expected on KBL-R; verify framebuffer |
| MX150 | Disabled | Do not expect macOS Optimus support |
| ALC298 layout 30 | PASS | Confirm codec/revision first |
| ETD2303 touchpad | PASS | Confirm exact bus and ACPI namespace |
| BCM943602CS native Wi-Fi/BT | PASS | Requires same supported card and antenna wiring |
| Intel AC8265 | Historical route only | itlwm/HeliPort may work, but is not the final Ventura profile |
| HiDPI 1424×802 | PASS on BOE panel | Do not copy display override to another panel without reading its EDID |
| Automatic battery S4 | PASS with limitations | Firmware/EC/power policy can vary; reproduce the tests |

## Hardware substitutions

A different Wi-Fi card, SSD controller, panel EDID, touchpad or EC firmware is not a cosmetic substitution. It changes the evidence boundary. Record the change and re-test the affected subsystem instead of assuming the whole profile remains valid.
