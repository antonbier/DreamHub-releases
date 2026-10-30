# DreamHub on a Raspberry Pi or a server — preview

> **Preview — not yet tested on real hardware.** These packages are built and the pieces were tried one by one (the program against a real Dreambox, the setup command), but nobody has installed them on a Pi or a PC with systemd yet. If you do, please tell us how it went.

DreamHub does not have to run on the Dreambox. **`dreamhub-server`** runs on a machine in your network — a Raspberry Pi, a mini PC, a home server — and controls the Dreambox over its web interface.

## Which file

From the [preview release](../../releases/tag/v0.12.1-server-preview):

| Your machine | File |
| --- | --- |
| Raspberry Pi with a 64-bit system (Pi 3, 4, 5) | `dreamhub-server_<version>_arm64.deb` |
| Raspberry Pi with the 32-bit Raspberry Pi OS | `dreamhub-server_<version>_armhf.deb` |
| A normal PC or mini PC (Debian, Ubuntu) | `dreamhub-server_<version>_amd64.deb` |

## Install

```sh
sudo apt install ./dreamhub-server_*.deb      # brings ffmpeg along
sudo dreamhub-setup 192.168.1.50              # your Dreambox's address
```

Then open `http://<this machine>:8765` in a browser. DreamHub only **reads** at first; to let it change things on the box (every change still shows a preview first): `sudo dreamhub-setup 192.168.1.50 --writes`.

## What is missing this way

Everything that needs DreamHub on the box itself: renaming, moving and deleting recordings, the box's picon folders, its services and Docker apps. The self-test shows those lines as warnings (the "services" line may even read *failed* on a server — that is a known flaw of the preview). If you want them, install the normal `dreamhub` package on the box as well and pair the two. The server package replaces the box package: do not install both on one machine.

## More

The [manual](MANUAL.md) ([Deutsch](MANUAL.de.md)) has a section *On a Raspberry Pi or a server*. Remove with `sudo apt remove dreamhub-server` (your data stays) or `sudo apt purge dreamhub-server` (data goes too).

A Docker image for Unraid and other NAS systems is planned.
