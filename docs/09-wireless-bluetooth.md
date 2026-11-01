# Wi-Fi, Bluetooth and Apple Continuity

## Two wireless phases — keep them distinct

### Phase A: original Intel AC8265

The machine originally used Intel Wireless-AC 8265. The historical Sonoma path used itlwm/AirportItlwm with HeliPort, including a Keychain-based hidden-SSID auto-join helper.

This was a workable alternative but not the final Ventura reference profile. It had its own reconnect, hidden-network and sleep/wake behavior to validate.

### Phase B: final Broadcom card

The final reference machine uses a Broadcom BCM943602CS card (BCM43602 Wi-Fi / BCM20703A1 Bluetooth family). Ventura uses the native Apple Wi-Fi and Bluetooth stack.

Acceptance recorded:

- Wi-Fi connects and remains available after wake.
- Bluetooth controller enumerates and is On.
- AirDrop file transfer between Apple devices succeeded.
- No Intel Bluetooth kext stack or HeliPort is required in this final configuration.

The OpenCore config includes SSDT-DRP08 to disable the unused RP08 Intel WLAN device in this Broadcom configuration.

## Do not load both stacks

Do not simultaneously load:
- Intel itlwm/AirportItlwm + HeliPort; and
- a Broadcom-native stack with a Darwin-only Intel WLAN disable.

Choose the path matching the physical card. If you retain the Intel 8265, use the current OpenIntelWireless instructions for your macOS version and independently test sleep/wake. Do not assume the Broadcom results apply.

## Bluetooth sleep caveat

The Sonoma S4 investigation recorded Bluetooth maintenance-wake watchdog/HID slow-ack symptoms in some tests. The final Battery S3→S4 profile later passed with Bluetooth enabled, but this does not prove every Bluetooth peripheral will behave identically.

Do not add BlueToolFixup, BrcmPatchRAM or Intel Bluetooth kexts simply because an old guide lists them. Add a patch only when the exact controller and symptom require it.
