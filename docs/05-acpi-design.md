# ACPI / SSDT design

The final Ventura reference profile is machine-specific. Each table below has a purpose and an evidence boundary. Do not copy the AML files or patches to another laptop without comparing the live DSDT and IORegistry.

## Enabled ACPI tables

| Table | Purpose in this machine | Why retained |
|---|---|---|
| `SSDT-ALS0.aml` | Synthetic ALS compatibility device | The table exposes a fixed 300-lux illuminance value; it is not a physical ambient-light sensor and cannot provide genuine automatic brightness data |
| `SSDT-DDGPU.aml` | Calls the OEM dGPU power-off method | The `_OFF` call is Darwin-gated; MX150 is excluded from the macOS display path while Windows behavior is preserved. Direct hardware power draw was not independently measured, so do not claim a quantified dGPU power saving |
| `SSDT-EC-USBX.aml` | Synthetic EC and USBX power properties | Compatibility layer only; it does not replace or modify the physical EC firmware |
| `SSDT-GPRW.aml` | Corrects wake behavior around GPRW | Paired with GPRW→XPRW ACPI patch |
| `SSDT-LGPA.aml` | MiBook brightness/function-key event logic | Paired with LGPA→XGPA patch |
| `SSDT-PLUG.aml` | XCPM CPU plugin-type injection | Live PR00 plugin-type was verified |
| `SSDT-PNLF.aml` | Apple backlight panel device | Live panel path bound to Apple backlight driver |
| `SSDT-PS2K.aml` | Internal keyboard integration | Works with VoodooPS2 and MiBook key event handling |
| `SSDT-RMNE.aml` | Built-in Ethernet identity anchor | Preserves en0 identity for the iServices/network topology |
| `SSDT-TPD0.aml` | Darwin-specific I²C/GPIO touchpad path | Required for the tested ETD2303 touchpad chain |
| `SSDT-DRP08.aml` | Disables the RP08 Intel WLAN device | Darwin-only; added for the Ventura Broadcom-native wireless configuration while the Intel card remains physically present |
| `DMAR.aml` | Custom no-RMRR DMAR table | Allows AppleVTD to remain enabled in the tested profile |

## ACPI patches

- `GPRW → XPRW`: paired with SSDT-GPRW.
- `LGPA → XGPA`: paired with SSDT-LGPA.
- The older global `_UPC → XUPC` rename was retired after moving to a precise USBMap.

## Items removed after live audit

| Item | Decision | Reason |
|---|---|---|
| `SSDT-MCHC` | Removed | Synthetic node not registered/matched; no live dependency |
| `SSDT-PMCR` | Removed | Not matched; native PCI device driver already bound |
| `SSDT-MEM2` | Removed | Legacy synthetic memory node with no OEM references |
| `SSDT-HPET` | Removed | Restored OEM HPET after isolated reboot/Deep Idle regression passed |
| `SSDT-USB-WLAN_LTEBT` | Removed | Replaced by a precise USBMap |
| Global `_UPC → XUPC` | Removed | Broad namespace patch affected many OEM methods; no longer needed |

The removals were not based on aesthetic cleanup alone: each was tested independently, followed by reboot and feature checks.

## Why no full DSDT replacement

The project used the live OEM DSDT as the reference namespace and added narrow SSDTs. It deliberately avoided replacing the complete DSDT. This limits the patch surface and makes firmware updates easier to audit.

## AppleVTD and DMAR construction

The OEM E07 DMAR was 168 bytes and contained two DRHD remapping structures plus two RMRR reserved-memory regions. The custom table preserved the two DRHD structures byte-for-byte, removed both RMRR structures, and recalculated table length/checksum. The resulting table is 104 bytes and has a valid ACPI checksum. OpenCore deletes the OEM DMAR before adding the custom table.

A log mentioning IOMapper is not enough to conclude that AppleVTD must be disabled. Early S4 failures involved IODMACommand / dmaMap / IOMapper paths, but the investigation included both AppleVTD-off and AppleVTD-on candidates. The final accepted profile retained AppleVTD with a custom no-RMRR DMAR and `DisableIoMapper=False`.

## Important warning

The S4 LID/GPE experiments described in the S4 case study are rejected. Do not re-add a LID0 depth patch, register GPE16 as a wake source, or add a virtual LIDW device based on this historical work.
