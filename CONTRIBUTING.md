# Contributing

We welcome evidence from other TM1701/TM1707 owners. The goal is to make the guide more useful without turning one machine's configuration into a universal recipe.

## A useful report includes

- Exact model (TM1701 or TM1707) and CPU.
- BIOS and EC version (do not upload firmware dumps).
- Panel model and EDID vendor/product (remove EDID serial if present).
- Audio codec and touchpad bus.
- Wi-Fi/BT card model.
- macOS version/build and OpenCore version.
- Relevant config diff with SMBIOS identity values removed.
- Exact reproduction steps and expected/actual result.
- Logs with serial numbers, UUIDs, MAC addresses, private IPs, hostnames and user paths redacted.

## Evidence standard

Please label results as PASS, FAIL, DEGRADED or UNVERIFIED. Include the exact test conditions. For sleep/S4 reports, include the power source before sleep, `pmset -g custom`, the event sequence (`Entering Sleep`, `DarkWake`, `Maintenance Sleep`, `Wake from Standby` or `Wake from Deep Idle`), same-boot-session evidence and panic status.

## What not to submit

- Full EFI folders or Vaults containing SMBIOS identity.
- BIOS/SPI/EC binary images.
- Apple serials, MLB, ROM, UUID or real MAC addresses.
- Passwords, API keys, Wi-Fi PSKs or private network information.
- Private logs containing unrelated personal data.

## Pull request structure

Prefer one topic per PR:
- BIOS / firmware observation;
- ACPI table or namespace difference;
- kext compatibility;
- display/connector behavior;
- sleep/wake test;
- installation correction.

Do not silently overwrite a failed experiment. Add the result to the decision log and explain whether it supersedes an earlier decision.
