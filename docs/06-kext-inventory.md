# Kext inventory and version rationale

The list below is reconstructed from the final Ventura config and the archived bundle version metadata. It describes the reference profile, not a universal list for every TM1701.

## Enabled in the final Ventura reference config

| Kext / plugin | Version | Role |
|---|---:|---|
| Lilu | 1.7.2 | Patch framework dependency |
| VirtualSMC | 1.3.7 | SMC emulation |
| SMCBatteryManager | 1.3.7 | Battery reporting |
| SMCProcessor | 1.3.7 | CPU sensor/reporting plugin |
| SMCLightSensor | 1.3.7 | Ambient-light compatibility |
| WhateverGreen | 1.7.0 | Intel graphics support |
| AppleALC | 1.9.7 | ALC298 audio layout 30 |
| HibernationFixup | 1.5.4 | Part of the validated S4 resume path |
| NullEthernet | 1.0.8 | Supports the RMNE/en0 identity topology |
| VoodooI2C | 2.8 | I²C touchpad controller |
| VoodooI2CServices | 1.1 | VoodooI2C plugin |
| VoodooGPIO | 1.0 | GPIO support |
| VoodooInput | 1.1.7 | HID input layer |
| VoodooI2CHID | 1.0 | HID touchpad interface |
| VoodooPS2Controller | 2.3.8 | PS/2 keyboard/controller path |
| VoodooPS2Keyboard | 2.3.8 | Internal keyboard |
| USBMap | 1.0 | Precise 11-port USB map |

## Wireless stack

The final Ventura profile does **not** load Intel AirportItlwm/itlwm or HeliPort. The reference machine uses a Broadcom BCM943602CS card and the native Apple Wi-Fi/Bluetooth stack.

The archived EFI contains an old Sonoma-specific AirportItlwm bundle, but it is not an enabled entry in the final Ventura config. Presence on disk is not proof of a loaded kext.

## Components intentionally not enabled

| Component | Decision | Why |
|---|---|---|
| NVMeFix 1.1.3 | Disabled | Early controlled A/B showed improved boot stability with it disabled; no current APST issue justified re-enabling |
| RestrictEvents | Disabled | Single-variable removal passed the tested boot/device checks |
| BlueToolFixup / BrcmPatchRAM | Not used | Native Broadcom stack worked; no evidence justified extra Bluetooth patches |
| IntelBluetoothFirmware / Injector | Not used | Final Broadcom card uses native Apple Bluetooth |
| ALCPlugFix | Not installed | Physical headphone replug and headset microphone tests passed |
| RealtekCardReader / Friend | Not installed | SD card was not a priority; actual media read/write was not accepted |
| NVMeFix | Not enabled | Do not add it merely because a generic guide recommends APST |

## The VoodooI2C version lesson

VoodooI2C 2.9.1 was tested and rolled back. With 2.9.1 the TPD0 path did not establish the complete HID/Multitouch child chain, even with VoodooInput 1.1.7. Returning to VoodooI2C 2.8 restored the expected Magic Trackpad 2 event chain.

**Do not upgrade a kext only because a newer version exists.** Verify the exact plugin set, dependency order, OS compatibility and physical behavior.

## Update policy

When updating any kext:

1. Record the old bundle version and hash.
2. Read upstream release notes and OS support.
3. Update only one kext or one dependency group.
4. Rebuild the OpenCore snapshot/Vault.
5. Run matching ocvalidate.
6. Boot and test the exact feature.
7. Roll back if the full device chain regresses.
