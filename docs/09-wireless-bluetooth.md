# Wi-Fi, Bluetooth and Apple Continuity

## Physical topology — this is an added Broadcom card, not a simple swap

The reference laptop retains its original Intel Wireless-AC 8265. A Broadcom BCM943602CS / BCM43602 was added through the WLAN_LTE slot and adapter path; its BCM20703A1 Bluetooth USB connection is routed through the WLAN_LTE USB wiring. Windows validation confirmed the Broadcom card, adapter, antennas and Bluetooth USB path.

The final macOS profile disables the unused Intel RP08 WLAN device only under Darwin through SSDT-DRP08. Windows retains the physical Intel device.

## Two wireless phases — keep them distinct

### Phase A: original Intel AC8265

The historical Sonoma 14.8.9 path first used AirportItlwm_Sonoma144 2.4.0. It could not reliably join the user's hidden 5 GHz/2.4 GHz networks, so production moved to itlwm 2.3.0 + HeliPort 1.5.0 on demand + a Keychain-backed hidden-SSID helper. This solved the hidden-network requirement without storing the Wi-Fi password in plaintext.

That was a workable Sonoma bridge, but it was not the final Ventura reference profile and did not expose the expected AWDL path for full Apple Continuity.

### Phase B: final Broadcom card

The final reference machine uses a Broadcom BCM943602CS card (BCM43602 Wi-Fi / BCM20703A1 Bluetooth family). Ventura uses the native Apple Wi-Fi and Bluetooth stack.

Acceptance recorded:

- Wi-Fi connects and remains available after wake.
- Bluetooth controller enumerates and is On.
- AirDrop file transfer between Apple devices succeeded on 2026-10-25.
- Handoff, Universal Clipboard, Sidecar, Universal Control and every other Continuity feature were not all individually accepted; do not infer them from AirDrop alone.
- No Intel Bluetooth kext stack or HeliPort is required in this final configuration.

The OpenCore config includes SSDT-DRP08 to disable the unused RP08 Intel WLAN device in this Broadcom configuration.

## Do not load both stacks

Do not simultaneously load:
- Intel itlwm/AirportItlwm + HeliPort; and
- a Broadcom-native stack with a Darwin-only Intel WLAN disable.

Choose the path matching the physical card. If you retain the Intel 8265, use the current OpenIntelWireless instructions for your macOS version and independently test sleep/wake. Do not assume the Broadcom results apply.

## Bluetooth sleep caveat

The September deep audit found seven historical bluetoothd crash reports with more than one crash signature, plus a separate HIDProfile prepareForSleep timeout. These were tracked separately from the 0x27 storage/DMA wake panic and the framebuffer mosaic; no single shared root cause was established.

The Sonoma S4 investigation also recorded Bluetooth maintenance-wake watchdog/HID slow-ack symptoms in some tests. The final Battery S3→S4 profile later passed with Bluetooth enabled, but this does not prove every Bluetooth peripheral will behave identically.

Do not add BlueToolFixup, BrcmPatchRAM or Intel Bluetooth kexts simply because an old guide lists them. Add a patch only when the exact controller and symptom require it.
