# Security and privacy policy

This repository is public. Treat every submitted file as public forever.

Do not submit:
- SMBIOS serial numbers, MLB, ROM, SystemUUID or generated identity fields;
- MAC addresses, private IPs, hostnames or local account names;
- credentials, passwords, tokens, private keys or Wi-Fi PSKs;
- raw EFI folders, OpenCore Vaults or complete config.plist files from a personal machine;
- BIOS/SPI/EC firmware dumps, raw disk images or private recovery archives;
- unredacted panic logs or system reports containing personal identifiers.

If a report needs to show a config value, provide a minimal sanitized snippet with identity fields removed.

If a secret or personal identifier is accidentally committed, opening a follow-up commit that deletes it is not sufficient: the value may remain in Git history. Contact the repository maintainer and rotate/reissue the exposed credential or identity where applicable.
