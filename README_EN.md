# MiBook TM1701/TM1707 Hackintosh Field Guide

An evidence-driven engineering notebook for the Xiaomi Mi Notebook Pro 15.6-inch **TM1701/TM1707 (8th-gen Kaby Lake Refresh)**.

This repository documents the path from a booting Hackintosh to a carefully validated daily-use setup: BIOS/firmware, OpenCore, ACPI/SSDT, kexts, graphics, audio, Wi-Fi/Bluetooth, USB mapping, sleep, hibernation and failed experiments.

The reference machine is an i7-8550U / UHD 620 / 8 GB TM1701 running a frozen, tested **macOS Ventura 13.7.8** baseline with OpenCore 1.0.7. The final wireless setup uses a Broadcom BCM943602CS card with native Apple Wi-Fi/Bluetooth support.

## Start here

- [Hardware and model compatibility](docs/01-hardware-and-compatibility.md)
- [Installation walkthrough](docs/02-installation.md)
- [BIOS and EC notes](docs/03-bios-and-ec.md)
- [OpenCore baseline](docs/04-opencore-baseline.md)
- [ACPI design](docs/05-acpi-design.md)
- [Kext inventory](docs/06-kext-inventory.md)
- [S4 sleep/hibernation case study](docs/case-studies/s4-sleep-resume.md)
- [Rejected experiments](docs/decision-log/rejected-experiments.md)

## What this repository is — and is not

It is a public engineering record with reproducible reasoning, version notes, test outcomes and rollback lessons.

It is **not** a ready-to-flash EFI pack. No machine-specific SMBIOS serial, MLB, ROM, UUID, MAC address, raw EFI, BIOS image, EC firmware image or private backup is published. Generate unique SMBIOS identity data for your own machine and build your EFI from the upstream project and current OpenCore documentation.

TM1905/TM1963 are 10th-gen Comet Lake systems and are **not covered** by the TM1701 ACPI/BIOS/power conclusions in this guide.

## Important caveat

Ventura 13.7.8 is the version validated on this particular machine, not a claim that it is the best or safest OS choice for a new installation in 2026. Re-check Apple's security support and application compatibility before deployment.

See [scope and evidence rules](docs/00-scope-and-evidence.md), [validation matrix](docs/11-validation-matrix.md), and [references](docs/references.md).
