# DreamHub

A web app and companion for your Dreambox: recordings, timers, the TV guide, internet channels (IPTV), series, an assistant — in the browser on your phone, tablet or computer, in your home network.

**Current release: 0.13.1** — packages are attached to the [latest release](../../releases/latest). The manual is [MANUAL.md](MANUAL.md) (English) and [MANUAL.de.md](MANUAL.de.md) (Deutsch).

## What you need

- A **Dreambox One or Two** (DreamOS) — developed and tested on a Dreambox One. The DM900 UHD / DM920 UHD (`armhf`) run too — a user reports it working on a DM920 UHD; there was no in-depth test, but the basic functions should work.
- A phone, tablet or computer in the same network.
- Other receivers (**VU+ Solo2 with VTi** checked, other OpenWebif boxes expected to work) can be added as a second, *Basic* box — the manual has the table of what works where.

## Install

**The easy way — the package feed** (on the box's shell; WebAdmin's terminal does):

```sh
echo "deb [trusted=yes] https://antonbier.github.io/DreamHub-releases/apt/arm64 ./" > /etc/apt/sources.list.d/dreamhub.list
apt-get update && apt-get install dreamhub
```

(DM900 / DM920 UHD: `armhf` instead of `arm64` in the address.) `apt` fetches what is missing, e.g. `ffmpeg`. (The package adds this feed by itself the first time it is installed, so a box that got DreamHub by hand updates from it too.) Then open `http://<box IP>:8765` and follow the first-steps assistant. The feed is not signed (`[trusted=yes]`, like the other Dreambox plugin feeds), so add only this address.

**Or by hand:** download `dreamhub_0.13.1_arm64.deb` (`armhf` for a DM900 / DM920 UHD) from the [latest release](../../releases/latest), copy it to the box and run `dpkg -i`; if `dpkg` complains about a missing package (DreamHub needs `ffmpeg`), `apt-get update && apt-get -f install` finishes it.

Installing never restarts enigma2: the TV launcher plugin loads at the next restart (`systemctl restart enigma2`) when it suits you; DreamHub itself works in the browser at once.

## On a Raspberry Pi or a server

DreamHub does not have to run on the box: install **`dreamhub-server`** (`arm64` / `armhf` for a Raspberry Pi, `amd64` for a normal PC with Debian or Ubuntu) with `sudo apt install ./dreamhub-server_*.deb`, then `sudo dreamhub-setup <box address>`. It controls the Dreambox over the network. What needs DreamHub on the box itself (renaming and moving recordings, picon folders, services, Docker apps) is not available this way. Details in the [manual](MANUAL.md).

- **Raspberry Pi, mini PC, server:** DreamHub can also run off the box — see [SERVER.md](SERVER.md) (preview, untested on real hardware).

## Update and go back

- **Update:** in DreamHub itself — **Settings → System → About DreamHub** shows a newer version and what is new in it, and installs it after a preview (DreamHub restarts for about half a minute; the page reloads by itself). With the feed also `apt-get update && apt-get upgrade`, or **Apps → Packages**. By hand: install the newer `.deb` the same way. Only the DreamHub service restarts, never enigma2.
- **Go back:** keep the previous `.deb`; installing it again puts the old version back. Before an update, copy `/usr/bin/dreamhub` and `/data/dreamhub/dreamhub.db` somewhere safe if you want a way back that includes your data.
- **Remove:** `dpkg -r dreamhub` keeps your data, `dpkg -P dreamhub` removes it.

## Good to know

- DreamHub **never changes recordings, timers, channel lists or the box without showing you exactly what will change and asking you to confirm.**
- It runs alongside the box's own web interface; it does not replace or modify it.
- **Docker apps** (Apps → Docker) are a **beta**: simple containers work; fixed ports, mounting folders and some compose files are still limited.
- Everything stays in your network. DreamHub talks to outside services (TV guide data, cover art, the optional AI assistant) only when you switch those on.

## About

DreamHub is an independent hobby project — **not made by or affiliated with Dream Multimedia** or any of the makers whose names appear here. It was written with the help of an AI assistant (Claude) and checked by hand on real boxes. The source code is **closed for now**; bug reports and ideas are very welcome. The software and data it builds on are listed in [THIRD_PARTY.md](THIRD_PARTY.md) (TMDB, fanart.tv and others are used only if you switch them on, with your own key).

## Feedback

Questions and problems: open an [issue](../../issues) here. Say which box and which version (Settings → Box admin → Device shows it). Tried it on another box? Tell us what worked — it helps everyone.

## License

[Creative Commons Attribution-NonCommercial-ShareAlike 3.0](LICENSE).
