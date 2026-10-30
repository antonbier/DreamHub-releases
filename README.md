# DreamHub

A web app and companion for your Dreambox: recordings, timers, the TV guide, internet channels (IPTV), series, an assistant — in the browser on your phone, tablet or computer, in your home network.

**Current release: 0.12.1** — packages are attached to the [latest release](../../releases/latest). The manual is [MANUAL.md](MANUAL.md) (English) and [MANUAL.de.md](MANUAL.de.md) (Deutsch).

## What you need

- A **Dreambox One or Two** (DreamOS) — developed and tested on a Dreambox One. The DM900 UHD / DM920 UHD (`armhf`) run too — a user reports it working on a DM920 UHD; there was no in-depth test, but the basic functions should work.
- A phone, tablet or computer in the same network.
- Other receivers (**VU+ Solo2 with VTi** checked, other OpenWebif boxes expected to work) can be added as a second, *Basic* box — the manual has the table of what works where.

## Install

1. Download `dreamhub_0.12.1_arm64.deb` from the [latest release](../../releases/latest) (`armhf` for a DM900 / DM920 UHD).
2. Copy it to the box and install it:
   ```sh
   scp -O dreamhub_0.12.1_arm64.deb root@<box IP>:/tmp/
   ssh root@<box IP> 'dpkg -i /tmp/dreamhub_0.12.1_arm64.deb'
   ```
   If `dpkg` complains about a missing package (DreamHub needs `ffmpeg`), run `apt-get update && apt-get -f install` — it fetches what is missing and finishes the installation.
3. Open `http://<box IP>:8765` in a browser and follow the first-steps assistant.

Installing never restarts enigma2: the TV launcher plugin loads at the next restart (`systemctl restart enigma2`) when it suits you; DreamHub itself works in the browser at once.

## Update and go back

- **Update:** install the newer `.deb` the same way; only the DreamHub service restarts.
- **Go back:** keep the previous `.deb`; installing it again puts the old version back. Before an update, copy `/usr/bin/dreamhub` and `/data/dreamhub/dreamhub.db` somewhere safe if you want a way back that includes your data.
- **Remove:** `dpkg -r dreamhub` keeps your data, `dpkg -P dreamhub` removes it.

## Good to know

- DreamHub **never changes recordings, timers, channel lists or the box without showing you exactly what will change and asking you to confirm.**
- It runs alongside the box's own web interface; it does not replace or modify it.
- **Docker apps** (Apps → Docker) are a **beta**: simple containers work; fixed ports, mounting folders and some compose files are still limited.
- Everything stays in your network. DreamHub talks to outside services (TV guide data, cover art, the optional AI assistant) only when you switch those on.

## Feedback

Questions and problems: open an [issue](../../issues) here. Say which box and which version (Settings → Box admin → Device shows it). Tried it on another box? Tell us what worked — it helps everyone.

## License

[Creative Commons Attribution-NonCommercial-ShareAlike 3.0](LICENSE).
