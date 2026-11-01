# USB mapping, USB-C, storage and Windows coexistence

## USB map

The reference profile uses a precise **11-port USBMap.kext** and keeps `XhciPortLimit=False`.

It replaced an older broad strategy that combined:
- a global `_UPC → XUPC` rename; and
- `SSDT-USB-WLAN_LTEBT`.

The precise map was validated with actual devices and ordinary sleep/wake. Do not enable the old map and USBMap simultaneously.

## USB-C acceptance

Both USB-C ports passed USB 3.0 5 Gbps data tests.

| Port | Link | Representative measured result |
|---|---|---|
| Upper USB-C | 5 Gbps | ~451 MB/s write, ~364 MB/s read |
| Lower USB-C | 5 Gbps | ~454 MB/s write, ~362 MB/s read |

The lower port was tested in both plug orientations with a known-good full-function cable. An earlier 480 Mbps observation was traced to the cable/device negotiation context, not proven to be a USBMap defect.

The project does not claim Thunderbolt functionality merely because a connector is USB-C. The accepted data path is USB 3.0 5 Gbps plus separately tested display output.

## NVMe and APFS

- The reference macOS drive is a Samsung PM961-class NVMe.
- The Windows system is on a separate NVMe.
- Both drives reported SMART Verified and TRIM support during the audit.
- NVMeFix was disabled after an early controlled A/B; do not add it by habit.
- `forceRenderStandby=0` is not a general NVMe fix. Revisit it only if the specific `CSTS=0xffffffff` panic is reproduced.

## Windows dual boot

Keep Windows EFI/BCD separate. The reference project deliberately did not write OpenCore or rescue files to the Windows ESP.

Important:
- identify disks by physical model and partition UUID;
- never assume disk0/disk1 stays the same;
- keep a separate OpenCore rescue USB;
- use OpenCore to select Windows in the validated dual-boot path;
- do not run BCDBoot or Windows boot repair as part of a macOS installation unless the user explicitly authorizes the Windows-side change.

## NTFS auto-mount and idle power

An earlier Sonoma phase found that automatic NTFS mounting could keep UVFSService active and materially increase idle CPU/power. The reference system used a no-auto-mount policy for the Windows NTFS volume and mounted it only when needed. This is an OS-level choice, not an EFI fix.
