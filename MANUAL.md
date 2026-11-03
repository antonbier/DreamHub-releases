# DreamHub user guide

DreamHub is a web interface for **Dreamboxes running DreamOS** — developed and tested on a Dreambox One; see *What you need* for the others. It runs on the box itself, next to the box's own WebControl, and you use it from a browser — on your phone, tablet or computer. This guide covers installing it, the first steps, and what each part does.

DreamHub changes nothing on its own: every change to recordings, timers or settings on the box is shown to you first ("Preview · nothing changed yet") and only happens when you confirm it. Every applied change is listed under **Box admin → Overview → Change log**.

---

## 1. Install
<!-- help: install -->

### What you need
<!-- help: needs -->

- A Dreambox running DreamOS:
  - **Dreambox One or Two** (ARM64) — package `dreamhub_<version>_arm64.deb`. Developed and tested on a Dreambox One with DreamOS 2.6 (Gemini/AIO image).
  - **DM900 UHD or DM920 UHD** (ARMv7) — package `dreamhub_<version>_armhf.deb`. Built the same way. A user reports it running on a DM920 UHD; there was no in-depth test, but the basic functions should work. Feedback is welcome.
  - The MIPS boxes (DM7080, DM820, DM520, DM525) aren't supported yet, older Dreamboxes without DreamOS not at all.
- A **VU+ or another OpenWebif receiver** works as a second box without installing anything on it — see *More than one box* below and the table in the [README](README.md#what-you-need).
- The box on your home network. DreamHub uses port **8765**; WebControl stays on port 80.
- About 15 MB on the root partition for the program; its database lives on `/data`.

### Today: install the package by hand
<!-- help: install-manual -->

The easiest way is DreamHub's **package feed** (see *From the feed* below): add it once, and installing and updating work like any other plugin. If you prefer, install the `.deb` package directly, as described here.

1. **Get the package** for your box from the latest release: [github.com/antonbier/DreamHub-releases/releases](https://github.com/antonbier/DreamHub-releases/releases) — `…_arm64.deb` for a Dreambox One/Two, `…_armhf.deb` for a DM900/DM920.
2. **Copy it to the box**, for example to `/tmp/` — over the box's network share (Samba), with an SCP program such as WinSCP or Cyberduck (user `root`), or from a USB stick.
3. **Install it.** Open a shell on the box. No SSH client is needed: the box's own **WebAdmin** has a terminal in the browser at `http://<box>/webadmin/terminal/` (log in as `root`). Then run:

   ```sh
   dpkg -i /tmp/dreamhub_*.deb
   ```

   If it complains about missing packages, run `apt-get -f install` and it fetches them.

   This installs the DreamHub service and starts it, and adds the **DreamHub** plugin to the TV's plugin list. The TV's interface (enigma2) is **never restarted** by the installation: the plugin appears at the next restart of enigma2 (the message at the end of the installation says so), which you do when it suits you — DreamHub itself works in the browser right away.
4. **Check it:** open DreamHub (next section), then **Box admin → Diagnostics → Run self-test**. It asks the box everything DreamHub needs and should come back all green.

### From the feed
<!-- help: install-feed -->

DreamHub has its own package feed for the **box package** (`dreamhub`; the server package is installed from its file, see below), so installing and updating work without copying files. **From version 0.13.0 the package adds it by itself** on its first install (`/etc/apt/sources.list.d/dreamhub-feed.list`, said in the install output); a feed you added yourself is left alone, and one you remove or switch off is not added again. To install DreamHub from the feed in the first place, add it once — on the box's shell (WebAdmin's terminal does), for a Dreambox One/Two:

```sh
echo "deb [trusted=yes] https://antonbier.github.io/DreamHub-releases/apt/arm64 ./" > /etc/apt/sources.list.d/dreamhub.list
apt-get update && apt-get install dreamhub
```

For a DM900/DM920 use `armhf` instead of `arm64` in the address. Instead of the shell you can also upload a file called `dreamhub-feed.list` with that one line in WebAdmin's feed management (**"Upload new \*-feed.list"**) and then install `dreamhub` like any other plugin. The feed is not signed (`[trusted=yes]`, like the other Dreambox plugin feeds): you trust the address, so only add it from this guide. `apt` fetches missing packages (such as `ffmpeg`) by itself.

Once the feed is added, DreamHub finds a new version of itself there (see *Updating*).

### On a Raspberry Pi or a server
<!-- help: install-server -->

DreamHub does not have to run on the box. On a Raspberry Pi (Pi 3, 4 or 5 with Raspberry Pi OS), a mini PC or a home server running Debian or Ubuntu you install **`dreamhub-server`** instead, and it controls the Dreambox over the network.

1. Download `dreamhub-server_<version>_<arch>.deb` — `arm64` (Pi with a 64-bit system), `armhf` (32-bit Pi OS) or `amd64` (a normal PC) — and install it: `sudo apt install ./dreamhub-server_*.deb` (it brings `ffmpeg` along).
2. Tell it which box to talk to: `sudo dreamhub-setup 192.168.1.50` (your Dreambox's address). It looks at the box first and writes nothing if the address does not answer like a Dreambox.
3. Open `http://<this machine>:8765` in a browser.

DreamHub only **reads** at first. To let it change things on the box (timers, channel lists — each change still shows a preview first) run `sudo dreamhub-setup 192.168.1.50 --writes`.

**What is missing this way:** everything that needs DreamHub on the box itself — renaming, moving and deleting recordings, the box's picon folders, its services and the Docker apps. The self-test shows those lines as warnings. If you want them, install the normal `dreamhub` package on the box too and pair the two (see *More than one box*). The server package replaces the box package: do not install both on the same machine.

### Updating
<!-- help: updating -->

**From DreamHub itself:** **Settings → System → About DreamHub** shows whether a newer version is out, with **What's new?** (the list of changes, in English) and **Install update…**. It looks twice a day and puts a new version into the notifications once; **Check now** asks at once. The update is a preview like every change; after you confirm, DreamHub downloads the new package, installs it and restarts — the page waits for it and reloads by itself (about half a minute; TV and recordings carry on). It comes from the DreamHub feed when the box has it, otherwise from the release on GitHub, checked against its published checksum before anything is installed. (The checksum catches a damaged download; like the feed, which is not signed, it rests on trusting the `DreamHub-releases` repository — the same trust as installing the `.deb` by hand.) Afterwards the card says it was updated, and if the TV plugin changed, that it loads at the next enigma2 restart. If an install fails, the card says so with the end of its output, and the previous version keeps running. The box needs internet for this, and device writes switched on (on a server: `dreamhub-setup … --writes`).

**By hand:** install the newer `.deb` the same way (`dpkg -i …`), or `apt-get update && apt-get install dreamhub` with the feed. Your settings, keys and dashboard are kept: they live in `/data/dreamhub`, which an update never touches.

### Uninstalling
<!-- help: uninstalling -->

```sh
dpkg -r dreamhub      # removes the program, keeps your settings in /data/dreamhub
dpkg -P dreamhub      # also deletes /data/dreamhub — settings, keys, change log
```

---

## 2. First steps
<!-- help: first-steps -->

### Open DreamHub
<!-- help: open -->

Any of these works:

- **From the TV:** open the plugin list (or the blue Extensions menu), choose **DreamHub**, and scan the QR code with your phone.
- **By name:** `http://<box name>.local:8765` — the box announces itself on the network (mDNS). Most phones and computers find it this way.
- **By address:** `http://<box IP>:8765`, e.g. `http://192.168.1.50:8765`.

Tip: put it on your phone's home screen — it then opens like an app. See [section 4](#4-dreamhub-as-an-app).

### The setup assistant
<!-- help: setup-assistant -->

The first visit starts a short assistant: language, connection to the box, a password for DreamHub, which hubs to show, your channel list, and optionally an AI key. Everything in it can be changed later under **Settings**, and **Settings → System → Run setup wizard** starts it again. Settings are split into tabs — General, Recordings, Services, Notifications, Hubs, Access, System — and any setting can be found with **Ctrl+K**.

### Profiles: one per person
<!-- help: profiles -->

Under **Settings → Profiles** an admin adds a profile for each person in the household. Each device asks once "Who's watching?" and remembers the answer; a profile can have a PIN (admins need one as soon as there's more than one profile). **Viewers** watch live TV, recordings and Plex, use the guide and the remote, and plan recordings — they don't rename or delete recordings, change other people's timers, or see the settings beyond their own. Per viewer, an admin can allow "plan recordings" and "rename and delete recordings". **Design the start page** sets the start page every viewer sees until they change their own.

### More than one box
<!-- help: boxes -->

With one box the Boxes tab stays hidden; Settings → General ends with an **Another box** card that opens it. Settings → **Boxes** (admins) adds another Dreambox by its address and, if its web interface asks for one, its login; **Test** tells you whether it answers and whether the login is right. With two or more boxes a **box selector** appears in the top bar, and the start page shows a slim strip with what the other boxes are doing — tap one to switch. Everything below (TV guide, timers, recordings, remote) is then about the selected box, and the address says which (`/b/living-room/guide`), so bookmarks work. DreamHub remembers your last box on each device. Live TV and recordings of another box are streamed through DreamHub, so the box's own login never reaches your browser. Notifications (the bell, and push on your phone) say which box they are about; the bell has an **Only this box** switch for this device.

A box added this way is controlled through its web interface: live TV, guide, timers, channel lists, the remote and watching its recordings. Managing recordings in place, logos, apps and the system tools need DreamHub on that box itself. A box that doesn't answer is greyed in the selector and the status line says so — DreamHub never waits for it. In a profile's settings you can limit which boxes it may use (nothing ticked means all). Removing a box makes DreamHub forget it and what it saved for it; the box itself is not touched.

**Receivers other than a Dreambox (VU+ and other OpenWebif boxes).** Anything that offers the OpenWebif web interface can be added as a Basic box in the same way — checked so far on a **VU+ Solo2 with the VTi 15 image**. Live TV, the TV guide, timers (with what each tuner is doing), channel lists, the remote, screenshots and watching its recordings work — a recording can also be played on that box's own TV, from the start; DreamHub itself runs on a Dreambox or a server and reaches the receiver over the network (these boxes have too little room and no systemd for DreamHub itself). Not available there: AutoTimer and SerienRecorder unless the box has them, and everything that needs DreamHub on the box. Set-top boxes on Wi-Fi are slow with large channel lists — wire them if you can. Not yet checked: a box with its web login switched on, and other makes and images; tell us what you find.

If a box runs DreamHub itself, you can **pair** the two: on that box open Settings → Boxes → *Pair this box with another DreamHub* and show the code (it also appears on that box's TV); in this DreamHub open the box under Settings → Boxes → Edit and type the code under *DreamHub on this box*. The code works once and for ten minutes. DreamHub then knows the other one, shows its version and tells you if one of the two needs an update. To end a connection, choose **Disconnect** and delete the token named "Hub: …" on the box under Settings → Access. A paired box can also play Plex films; the box fetches them from this DreamHub's address, which is found automatically (Advanced lets you set it).

A box that is switched off can be **switched on from here**. Under Settings → Boxes → Edit → *Switch on when it is off* choose **Wake-on-LAN** (type the box's MAC address; Wake-on-LAN must be on in the box's network settings and DreamHub on the same network) or **an extension's action** (for example the smart plug the box hangs on). A box that does not answer then shows a **Switch on** button — in the status line when it is the selected box, in the dashboard's strip for the others. Wake-on-LAN goes out at once; an extension's action is previewed first with the extension's own words, because a socket's action usually *toggles* — check that it says "on". Afterwards DreamHub looks at the box every few seconds for up to three minutes and the box turns normal as soon as it answers. Any profile that may use the box and has the remote can do this.

**Watching a recording on another box.** With two or more boxes, an open recording offers **Play on <box>** for every other box that is on and runs DreamHub (the box DreamHub is installed on, or one that is paired). The recording starts in that box's movie player, with seeking; DreamHub reads it from the box it is on, which may be a Basic box. The box you play it on must be able to reach DreamHub's address (the same address as for Plex — see *StreamBase* under Advanced in the box's settings). Needs the remote's permission.

### API and tokens
<!-- help: tokens -->

For apps and scripts (a DreamDroid-style app, Home Assistant, your own tools): **Settings → Access → API tokens** creates a token — shown once, revocable on its own, acting as the profile that created it (so it sees that profile's boxes and permissions). Send it with every request:

```
curl -H "Authorization: Bearer <token>" "http://dreambox.local:8765/api/v1/boxes"
curl -H "Authorization: Bearer <token>" "http://dreambox.local:8765/api/v1/timers?box=bedroom"
```

- **`box=<id>`** says which box a request is about; `GET /api/v1/boxes` lists the IDs you may use, with what each box can do (`capabilities`). Without it the first box you may use is meant, so a script written for one box keeps working when a second one is added.
- **Changes take two steps.** A call ending in `/plan` changes nothing and returns a preview with an `id`; `POST /api/v1/plans/<id>/apply` makes the change on the box the plan was made for. A plan is valid for ten minutes and once. Requests that change something also need the header `X-DreamHub-Request: 1`.
- **The full description** of every call is `/api/v1/openapi.json` (linked under the token list); load it into a tool such as Swagger UI or a code generator. Request and response shapes are filled in for the most useful calls first (status, boxes, timers, recordings, zap); the rest is listed with what it does.

### Assistants (MCP)
<!-- help: mcp -->

An AI assistant you already use (Claude Code, Claude Desktop, any client that speaks the Model Context Protocol) can look things up on your box and **suggest** changes. It can never make a change itself: what it suggests lands in **AI Center → Inbox**, where you see exactly what would happen and confirm it — or not.

**Switching it on.** It is off by default. **Settings → Access → Assistants** switches the endpoint `/mcp` on. Then open **AI Center → Connect**. The tab compares the built-in assistant (asks in DreamHub, paid with your API key) with an external one (a program on your computer, paid through its own subscription), then walks you through three steps: pick your program (Claude Code, Cursor, VS Code or another), name the connection and copy the command or config shown (it already contains your box address and the new key; the key is shown once and never again), and press **Test connection** after you ran it — it reports whether the connection has asked yet. Under *Connections and what they may do*, each connection has its toggles in three groups (look things up, make proposals, sensitive data); a change is saved at once. Claude Desktop and claude.ai in the browser cannot be connected this way (they need a different sign-in, which DreamHub deliberately does not offer). While *allowed tools* is filled in under Settings → AI, the built-in assistant's toggles are set there instead.

**What a token can do.**

- Read what its list allows: recordings, timers and conflicts, the programme guide, series, disk space, hub status. Sensitive reads (watching history, settings) are off unless you tick them.
- Suggest a timer, a trash, an ad mark. Each suggestion is a proposal with the assistant's reason; you open it, read the preview, and apply or discard it.
- Check an extension manifest, list and describe your extensions, and suggest a new or changed extension (a proposal like the others; you read the hosts it will contact and the requests its buttons send, confirm, and then enter any secrets yourself). An assistant never sees or sets a secret. Letting it *try* an extension's sources once (`extension_dry_run`, which sends read-only requests from the box to the devices the manifest names) is off unless you tick it.
- It acts with the rights of the profile that created the token, never more.
- It cannot use the rest of DreamHub (the web API, settings, the terminal, scripts, other tokens, profiles). A token stolen from your assistant's configuration therefore allows: reading what its list allows and creating proposals you can discard — nothing else. If one leaks, **revoke** it in Connect; it stops working at once.

**Limits that protect the box.** Each token has a call budget (30 a minute, 2000 a day, two at a time); four assistant requests are served at once across all tokens; answers are cut to a page with a "more" cursor. After ten rejected tokens an address is paused for five minutes. **Connect** shows what each token has used and when; the call log keeps who called what and how it went — never the questions or the answers — for 30 days (at most 50,000 entries).

**Text from your box is data.** Programme descriptions and recording names can contain sentences aimed at an assistant ("ignore your instructions and…"). DreamHub marks all such text so the assistant treats it as data, and the worst it can lead to is a proposal that you read before anything happens. Still, read proposals before applying them, like any change.

**Reaching it from outside the house.** DreamHub is built for your home network. If you want an assistant elsewhere (for example a Claude session on another server), use something you already trust to carry the connection — a VPN such as Tailscale, or a reverse proxy with HTTPS — and keep the token private. Plain HTTP over the internet exposes the token. If you use a reverse proxy, all clients appear to come from the proxy, so the rejected-token pause applies to them together.

**What was and was not checked.** The security review (milestone M5) checked token handling, who may call what, the separation from the rest of the API, markers on box text, load on a small box, error text and the call log, all against the mock box. It did not check a real client end to end on a real box, or anything outside the home network.

### Recommended: protect your network access
<!-- help: protect -->

- **Set a DreamHub password** (Settings → Access → Authentication). Without one, anyone on your home network can use DreamHub.
- **Lock down the box's own web interface too.** By default many boxes let every device on the home network use WebControl (port 80) without a password, even when one is set — DreamHub can't change that for you. In the box's own web interface settings, set **`localauth`** to *on* (`config.plugins.Webinterface.localauth=true`). If you do, enter the box's login under **Settings → Services → WebControl** so DreamHub can still reach it.

### Optional keys
<!-- help: keys -->

DreamHub works without any of these; each one adds something. All are entered under **Settings**, are stored only on your box, and are never shown again after saving.

| Key | Free? | What it adds |
| --- | --- | --- |
| **TMDB** (themoviedb.org → Settings → API) | yes | Posters, backdrops, ratings, descriptions everywhere; "Save covers"; name suggestions; the guide's film/rating filters |
| **OMDb** (omdbapi.com/apikey.aspx) | yes, 1,000 lookups a day | Rotten Tomatoes, IMDb rating and Metascore in a title's details |
| **fanart.tv** (fanart.tv → profile → API) | yes | The title's logo above the details: a recording's details, a programme's in the TV guide and Recommendations, a series' page. Needs TMDB too (it finds the title), and fanart.tv doesn't have a logo for every title |
| **AI provider** (OpenAI, Claude or an OpenAI-compatible server; Settings → AI) | paid per use, capped by a daily token budget you set | AI recommendations, AI name suggestions and the built-in assistant |
| **ntfy** topic | yes | Notifications on your phone — see *Notifications* below |

If the **DreamMediaInfo** plugin is installed on the box, DreamHub uses what it already knows first and only asks TMDB for the rest. Cover pictures that another plugin already saved next to a recording are always preferred.

---

## 3. What's where
<!-- help: whats-where -->

### Home
<!-- help: home -->

A dashboard of tiles: **Now on the box** (what the box shows, with channel up/down, mute and the remote), upcoming timers, conflicts, disk space, series episodes coming up, hub status. **Edit** rearranges tiles (arrows on each tile move it up or down; with a mouse you can also drag); **Custom tile** builds your own view ("films from TMDB with a rating above 7 on tonight", say).

### Recordings
<!-- help: recordings -->

Every recording folder on the box, folder by folder.

- **Click a title** for its details: picture, ratings, description, and **Play on the TV**.
- **Rename** a recording: all of its files are renamed together; suggestions come from the recording itself, TMDB or AI. You can also change the title the box's own list shows.
- **Select several** (or **Select all**) and:
  - **Rename by scheme** — names follow your scheme from *Settings → Recordings* (e.g. `{title} - S{season:02}E{episode:02} - {episodeTitle}`). With a TMDB key set, a film's missing year and an episode's title are filled in from TMDB by themselves. Tick **Sort into folders** to also move them into folders by your folder scheme (e.g. `{title}/Season {season:02}`); missing folders are created.
  - **Save covers** — writes the poster and backdrop next to recordings that have none, so the TV's own recording list and other plugins show them too (DreamHub shows posters without saving). Existing covers are never replaced.
  - **Kodi files** — writes the `.nfo` files and artwork Kodi reads (`tvshow.nfo`, poster, backdrop, logo and season posters in the series folder, an `.nfo` per episode, a film's `.nfo` with its pictures), with the TMDB, TheTVDB and IMDb ids. Best after renaming by the scheme with folders; after a batch rename it is offered for the renamed recordings. Logos need a fanart.tv key. Existing files are never replaced.
  - **Move to trash** — nothing is deleted right away; the **Trash** tab restores or deletes for good.

- **Cut by hand** — in a recording's details: name the parts to cut out (`m:ss`, from … to …), look at each boundary in the browser, preview, confirm. No ad detection needed; nothing is re-encoded (a cut can be off by up to a second) and the original goes to the trash, so you can restore it.
- **Stopped early** — a recording far shorter than the programme it was announced as (a restart, a full disk, a timer ended by hand) says so in the list. One you cut yourself isn't flagged.
- **Ads** — in a recording's details, **Detect ads** finds the ad breaks by the channel's logo, on the box, in the background (it waits while the box records). The breaks show on a timeline; tap one to watch from there in the browser. **Cut out the ads …** ticks the breaks it is sure about — the uncertain ones are yours to decide — shows what will happen and cuts them out without re-encoding. The original keeps all its files and goes to the trash, so it can be restored. For several recordings at once: select them, then **Detect ads**. The list shows "14 min ads" next to a recording that was checked.

Files are only changed when DreamHub runs on the box itself, and never while a recording is still being written or played.

### Plex library
<!-- help: plex -->

The films and series of your **Plex** server. Connect it under **Settings → Services → Plex → Sign in with Plex**: open plex.tv/link, enter the code shown, then pick your server and the address the box should use (one in your home network is fastest, or type your own, e.g. a reverse proxy). The server's other addresses are kept as **fallbacks**: if the one in use stops answering, DreamHub switches to the next one that answers as the same server. You can edit them under "The server's other addresses". Choose which libraries to show in the same place.

- Details and series pages show the title's logo from Plex when the server has one.
- **Tiles or list** — switch at the top right; filter by text or by watched state, sort by title, recently added, year or rating.
- **Watched** is Plex's own: marking a film or episode changes it in Plex right away (with **Undo**); a whole season or series shows every episode first.
- **Play on the box** plays the file straight from the Plex server with the box's own player, from the beginning — no Plex app on the box — in the box's movie player, so you can seek and pause with the remote. Whether a file plays depends on its formats, shown in the details.
- When the Plex server is off, the library still shows what DreamHub last saw ("Plex offline · as of …"); playing and marking work again once it answers.
- On **Home**, **Edit** can switch on the tile **Continue watching (Plex)**.

### Series
<!-- help: series -->

Your series from **SerienRecorder**. A series' page shows planned episodes, further broadcasts in the guide — each with its episode number and whether you're **missing** it — your recordings by season, and (with TVmaze, on by default) the full episode list with what's recorded, planned, missing or still to come. "Missing" is only shown when your recordings carry episode numbers like `S01E02`. Daily soaps that SerienRecorder numbers straight through (`S00E8622`) are matched by that number; only episodes from your first recording on count as missing. A broadcast shown on several regional channels at once (RTL's, for example) is listed once. Administrators can open SerienRecorder's own web interface for markers, seasons and settings — in a new tab, or on the page itself while DreamHub is opened over plain `http://` (a browser does not show the box's http page inside the secure address).

### TV guide
<!-- help: tv-guide -->

Four views, switched at the top; your choice is remembered:

- **Grid** — the classic timeline, four hours at a time.
- **Now & next** — what's on every channel right now with a progress bar, and what comes next; **Switch** changes the TV's channel. The default on phones.
- **Tonight** — what starts between about 20:00 and 21:30 as picture cards, up to a week ahead.
- **Channel** — one channel's whole day as a list.

**Films only** and **Rated 7+** filter every view (needs TMDB). Any programme opens its details with **Record** and, while it's on, **Switch now**.

### Live TV
<!-- help: live-tv -->

TV channels right in the browser — on the phone, tablet or PC, no app needed. Pick a channel from the list; **Watch what the TV shows** next to the channel list plays what the box is showing. The box sends its signal unchanged and your device decodes it, so the box itself does no extra work. **Space** pauses, **M** mutes, **F** or a double click switches to full screen; with several audio tracks (another language, Dolby) you choose one under the picture.

The box has one tuner: another channel only works while the TV and any recording are on the same frequency. Otherwise the picture says so — **Watch what the TV shows** always works. In the **TV guide**, a programme that's on now has **Watch in the browser** too, and **Home** can show a **Live TV** tile with your channels (switch it on under **Edit**). In **Recordings**, **Watch in the browser** plays a recording, with a bar to jump anywhere in it.

Works in current Chrome, Edge, Firefox and Safari. HD channels use your device's own video decoder — but browsers allow that only on a secure page (HTTPS). Opened over plain `http://<box-IP>:8765`, HD is decoded in software, which most computers cannot keep up with: the picture stutters while the sound keeps playing, and the player says so. Standard-definition channels are not affected.

**Open in player.** While a live channel plays, **Open in player** downloads a small playlist (`.m3u`) that points straight at the box's own stream (port 8001); opened with VLC, IINA or any player that reads playlists, it plays the channel without the browser — smooth in HD. It works where your device reaches the box directly (at home, or over a VPN); DreamHub is not in the path, so nothing is checked or logged there, and if the box's web interface asks for a login the player asks for it too. The button is hidden when the box's address isn't known to this page — for a second box, only administrators see it.

The **Stream details** under the picture show what your browser can do (secure page, WebCodecs, H.264 decoders); **Copy report** puts it on the clipboard as plain text to paste into a forum post.

### Radio
<!-- help: radio -->

Your radio stations, with what's on now and next where the station sends programme information. **Play on the box** switches the box to that station — you listen on the TV or whatever is connected to it; the station playing is shown on top. **In the browser** plays a station right on this device instead (like Live TV, it needs the tuner to be free or on the same frequency). Under the station that plays, the line the station sends about what it plays (artist and title, or the programme) shows when it sends one — it can take up to half a minute to appear. Any programme opens its details with **Record**; the radio recordings coming up are listed at the bottom. Stations can also be found through the search at the top.

### Timers
<!-- help: timers -->

All timers on the box, by day, with likely tuner conflicts marked. **Disable** keeps a timer in the list without recording; **Enable** turns it back on (with a conflict check first). **Delete** removes it — deleting a timer that is recording right now stops that recording, and the preview says so. **Clean up finished** removes every finished one-off timer in one preview (switched-off and repeating timers stay). **AutoTimer** records everything that matches its rules automatically; the page lists them, and **Show what would be recorded** previews what they would add. Administrators also get **Open the AutoTimer editor**, the plugin's own page on the box (it changes every AutoTimer without asking DreamHub's permissions, so other profiles don't see it). Each row has one action in sight (switch off/on, or delete when it can't be switched); the rest is under **⋯**.

### Recommendations
<!-- help: recommendations -->

AI picks from a bouquet's programme for the next hours, optionally fitted to your taste (what you record and follow — **What gets sent** shows the exact list before anything is sent). The picks stay until you generate new ones, also on other devices. Click a pick for its details and **Create timer**.

Every pick says **why** it was chosen. The **filter** above the picks narrows them to the evening (from 18:00) or to those without a timer yet. **Not for me** on a pick (or a series below) is remembered for you: that title — all episodes of that series — is not proposed again, and the line under the picks (or **Your taste**) offers **Undo**. **Series to continue** lists the next broadcast of the series you follow (SerienRecorder, AutoTimer) or have recordings of, straight from the programme — it needs no AI and works without a provider. **Your taste** shows what DreamHub knows from your box and what you marked as not for you; nothing is sent from there. The programme has no genre information, so feedback is per title, not per genre.

### AI Center: Assistant
<!-- help: ai-assistant -->

**AI Center → Assistant** (administrators only) lets you put a question to DreamHub's own assistant in your own words — "what will I miss this week because of overlapping timers?", "what is recorded of this series?", "what could I delete?". It looks things up on your box with the same read-only tools an outside assistant gets, and may *propose* a change; it never applies one. The tab says up front what the assistant can do (look things up, propose timers, trashes and rules) and what it cannot (it does not build extensions; for that it points you to Apps → Extensions or an outside assistant), and offers a few starter questions that only fill the box. Type the question, press **Ask** (or Ctrl+Enter), and wait up to a minute.

**Switching it on.** It needs an AI provider: an OpenAI key, a Claude key, or the address of an OpenAI-compatible server, entered under **Settings** (AI). Entering your own key is the opt-in. Until then the tab says no provider is set up. What is sent to the provider: your question, the earlier turns of the conversation and the results of the lookups the assistant made (titles, channels, times) — never settings, passwords or tokens.

**What you see.** The answer, a folded list of *what the assistant looked up*, and a folded list of *what the answer is based on* (the recordings, timers and programmes it read), so a wrong answer is noticeable — check a title or a time there. If it proposed something it says "I proposed …" with a card in the conversation (and a list *Waiting for you* beside it); a change you can review right there, a rule opens in **AI Center → Inbox**, where you read the preview and confirm or discard it, as with any suggestion. Nothing has changed before that.

**Your conversation stays in your browser tab.** The box stores none of it; closing the tab or **Clear conversation** forgets it. What the browser keeps is sent along with your next question, so the assistant can follow it up; treat the assistant's earlier answers as unverified — it checks again with a lookup.

**Limits that protect the box and your bill.** At most six lookups, 30,000 tokens and 60 seconds per question; one question at a time per profile and two across the box. Tokens count against a **daily budget** (shown in the tab; default 200,000, shared with Recommendations) — when it is used up the tab says so and stops. The assistant only has the lookups allowed for it (by default every read and suggest tool that is not sensitive); it never has the tools for managing extensions, and it can only propose rules (a person confirms them in the Inbox); it cannot use the rest of DreamHub (settings, terminal, other profiles).

**Text from your box is data.** Programme descriptions and recording names can contain sentences aimed at an assistant. DreamHub marks such text and tells the assistant to treat it as data; the worst it can lead to is a proposal that you read before anything happens.

**Local models are untested.** The provider setting accepts any OpenAI-compatible address (Ollama, LM Studio, vLLM, OpenRouter …), so a model on your own network can answer without a cloud account. That path has **not been tested by the project owner**: small local models often handle tool calls and the "this text is data" rule poorly. The limits above apply regardless, but treat the answers with extra care.

**What was and was not checked.** The security review (A5) covered what the assistant can reach, text from the box and from earlier turns, the "never applies" rule, load and cost limits, error text and the web door, all against the mock box and scripted models. It did **not** include a real provider or model, a real box, or streaming.

### AI Center: Inbox
<!-- help: ai-inbox -->

**AI Center → Inbox** collects everything that waits for your confirmation: suggestions from the built-in assistant, from an external assistant or from an automation rule. A filter by origin (with counts) and an origin line on each card show who suggested it; **Review** shows the current preview in place, and only **Apply** (with your PIN where one is set) changes anything. The number of waiting suggestions is on the menu entry and on the tab. Suggestions expire after a while; the card says so. A suggested rule only comes into being when you confirm it here, with its sentence and dry run.

### Channel list
<!-- help: channel-list -->

Your bouquets, editable on the phone (admins). With more than one box, **Copy from another box** (under the bouquets) makes this box's bouquets look like another box's: bouquets of the same name are brought in line, missing ones are created, the rest stays, and channels this box doesn't know are listed and left out. Move channels (buttons, or drag on a computer), put one on a given number, rename, remove, add from all the channels the box knows, add headings; create, rename and delete bouquets. **All channels** searches everything the box knows (new ones, ones in no bouquet, HD, by provider), **Logos** lists the channels without a logo and what else deserves a look (channels gone after a scan, twice in a bouquet, without programme data), shows the installed logo sets and lets you pick the one DreamHub uses — and, for a single channel, a different logo: from any installed set, your own picture, or one found in the public TV-logos collection (also from the check's "Without a logo" list). That choice only changes DreamHub's screens, never the TV, and **Back to automatic** undoes it. The list always reads the box's current channel list; it does not search for new channels — run a channel scan on the box for that. Every change is collected and shown as one preview ("ZDF HD moves from 14 to 2"); applying makes a backup of the whole channel list first and restores it by itself if the result isn't what was previewed. Older backups can be restored. Needs the BouquetEditor plugin on the box.

### IPTV
<!-- help: iptv -->

Internet channels in one place: **Media → IPTV**. (**Live TV** is the box's own channels; IPTV is what comes from the internet or from other devices on your network.) It is **one page, without tabs**: a column with **Mine** (last watched, favourites and your folders) and **Sources** on the left (a fold-out above the list on a phone), the channels on the right. Adding and changing sources, and making bouquets, is for those who manage the channel list; everyone with IPTV can browse, and playing needs the remote permission.

**Browsing.** The chips **All · TV · Radio** narrow what is shown. A tap on a channel **plays it on the TV** at once (it does not need to be in a bouquet; an internet stream has no tuner, so there is no **Record**). Everything else is under **⋯**: **Details…**, **Put in folder…**, **Make a bouquet…**, **Hide this channel**, **Hide the group**. The switch above the list has **Tiles** (the default; as many across as fit the screen), **List** and **Programme**; the choice is remembered on this device. Radio tiles show the station's logo and, where the stream sends it, the song playing. **Programme** is the time grid of a folder, or of a source or group of up to about 200 channels (a larger one asks you to put the channels you watch in a folder). A group of a source (Films, News…) is a chip above the list. The same channel from several sources is shown once in **All**, in folders and in a search, with "from 3 sources"; inside a source it is listed as that source has it. The **A–Z strip** at the right edge jumps to a letter (slide a finger along it to scrub). **Select** turns taps into ticks, with a bar to put the ticked channels in a folder, make a bouquet of them or let the assistant look at them. The search field (press **/**) looks in every source. Keys: arrow keys move, **Enter** plays, **i** opens the details, **Esc** closes a sheet or menu.

**Favourites and folders.** The star on a tile or row marks a favourite — yours alone, kept per profile. Favourites live in **folders** (a channel is in one folder); file one with **⋯ → Put in folder…** or by dragging it onto a folder in the column; folders can be renamed, moved and deleted (the channels stay as favourites). **Change order** in a folder's header puts its channels in order — drag, or use the arrow buttons, or **A–Z** or **By channel number** — and **Done** ends it. A favourite whose channel has left its source stays, greyed, "No longer available" — with the same name from another source offered when there is one. Limits: 100 folders, 5,000 favourites and 5,000 hidden items per profile. **Hidden** at the foot of the column is the way back for hidden channels and groups.

**The channel sheet.** **Details…** (or **i**) opens a channel's sheet with **Play on the TV**, the star and the folder, and two parts. **Programme**: today with a strip for the week; **Guide: … · change** shows which guide the channel uses and lets an administrator pick another by hand (remembered per box); the guide is loaded a day back and a week ahead and reloaded every few hours ("updated … ago", **Load again**). **Properties**: the name and group; the **Picon** of the channel itself (**Choose a picon…**: an **online search first**, then the installed sets, or a file of your own; it is DreamHub's own choice, shown on the tiles and in a bouquet made from the channel, and **Automatic** takes it back); the guide (a satellite channel's twin, or the source's own); the **streams** of the channel, one per source, each with **Test** and **Use this one** (which changes what a bouquet entry plays, after a preview); the bouquets the channel is in, with **Rename…** and **Picon…** for the entry; and **Hide this channel**. The same sheet opens from the **Find channels** list of the channel list, on the internet entries of your bouquets (filter **IPTV · Radio · TV**).

**Watch here.** A channel whose stream is an http address has **Watch here** in its sheet and in its ⋯ menu (not for rtsp, udp or `.ts` addresses). It plays in the browser when the stream needs no conversion: **HLS**, a video file (MP4, WebM) or an audio stream. DreamHub first opens the stream for a moment to see what it serves, then passes it on to the player — the address and any login in it stay on the box, and nothing is converted. When the browser cannot play it, the panel says why — an MPEG-TS broadcast stream, a protected stream, a list of channels instead of a stream, no answer — and offers **Play on the TV**. Live TV is never converted in software. Sound and picture also depend on your browser's formats; a stream it does not know is reported as not playable here. The picture has no captions to offer for a live stream.

**Sources.** A source is where a list of channels comes from. **+ Add source** asks what you have: **A list** (an M3U/M3U8 address, a file from this device, or a file in a public GitHub repository; optionally **also add it to Netcast**, which writes Netcast's file on the box after a preview), **An account** (host, user name and password of an Xtream-style account, live channels only), **A device on my network** (Tvheadend, HDHomeRun — the address is filled from its host name — or a SAT>IP server: paste the list address it announces) or **Something from Netcast** (a source Netcast knows, switched on here). The list is read once before anything is kept; a wrong login or a non-list says so and nothing is stored. A **file** is kept by DreamHub as a source of its own: it has no address and is not read again; a newer file replaces it in Manage sources. A login stays on DreamHub: it is never shown again, never sent to your browser and not part of the backup; the source only says "with login". Each source in the column has a state dot, its channel count and a **⋯** menu: **Refresh**, **Programme…** and **Settings…**; **Netcast** is one row that unfolds into its sources. A source that could not be read is said above the list, with **Retry** and **Settings**. **Manage sources** is where sources are set up: the place in the column (↑ ↓), whether it is shown there (off hides it everywhere and deletes nothing; bouquets already made keep working), which profiles see it, the address or login of a source you added, **Remove source…** (asks first, and lists what goes), and — for a source in Netcast's file — its address and removal, each as a preview of the change to that file. **Sources & programme** (the button above the list) shows whether the sources work, how much of a source has a programme, the guide files (with **Refresh** and **+ Add guide file**: the suggested list, an address or a public GitHub repository, optionally also in Netcast's file) and, if you have any, the lists remembered by address before bouquets stayed linked.

**Into a bouquet.** **Make a bouquet…** works on a channel (⋯), on the ticked channels (the Select bar), on a folder, and on a whole source or one of its groups. One panel shows which channels go in, lets you choose a new or an existing bouquet, tests the streams (untick the ones that do not answer), gives a channel the guide of one of the box's own channels, and shows exactly what will change before it waits for your confirmation, as everywhere. A bouquet made from a folder, source or group **stays linked**: the folder's (or list's) header says "in line" or "2 new, 1 removed" with **Bring in line…** (a preview again; it can also put the bouquet in the folder's order) and **Forget the link**, which leaves the bouquet as it is. Favourites, folders and hiding are only DreamHub's own notes for your profile and change nothing on the box.

**Let the assistant look.** In the Select bar, **Let the assistant look…** looks at the ticked channels (up to 60 at a time) and shows, for each, whether it has a **programme ✓ or ✗** (found in a guide file you loaded, on the box, or through a remembered twin), which **satellite channel's guide** it could show (a guide twin; it is offered only when the names are the same, and you can pick another candidate or leave it), and a **picon** from the public TV-logo collection for a channel without one. The best suggestion is already chosen; set a list to *Leave as it is* to skip it. Nothing is saved by looking: **Preview** turns your choices into one plan, and only **Save** after reading it keeps them. Twins and picons are DreamHub's own notes: no bouquet and nothing on the box changes; the next **Make a bouquet…** uses the twins. An outside assistant (MCP) or the built-in one can *propose* the same plan (**Guide twins and picons for internet channels**); it then waits in AI Center → Proposals for you to review and apply.

**Check programme.** Next to the channel count, **Check programme** looks at the channels the list shows (up to 200 at a time) and puts a small **✓ or ✗** on each tile and line, with a tally (“3 of 5 channels have a programme · 2 without”) and *Only those without a programme* to bring the gaps together; it saves nothing. A channel counts as ✓ when a guide file you loaded, a remembered guide twin, the box or — for an internet account (Xtream) — the account itself has a programme for it. An account is asked one channel at a time, 40 channels per check; the others show **?** — check again. The same count is used by *Let the assistant look*.

**Adult channels** are left out by default: a profile without the permission *See adult channels in IPTV* never receives them (not in the lists, the search, the guide or to play); administrators have it, and the page still keeps them out of sight until **Show adult groups** is ticked. Groups are judged by name, so some will slip through and some innocent ones are caught.

**Search** (the field at the top, **Ctrl+K**) also finds channels of your sources by name; a result opens the page at that source. Older links and bookmarks (`/netcast`, `/iptv?tab=guide`, `/channels?tab=iptv`) still work: they open the page, and the sheet or dialog that took the tab's place.

**What was and was not checked.** Built and tested against the mock box, a made-up catalogue of about 1,900 channels and fake servers. **Not** checked on a real box or with a real provider: an Xtream account (the calls follow the published API notes), the Tvheadend and HDHomeRun paths (from public documentation summaries), whether the box's player plays an Xtream `.ts` address, the songs shown on radio tiles for real stations, Watch here with a stream of a provider other than the Netcast ones (checked on a Dreambox One with HLS channels of the Netcast lists), a source from a file, and the memory and load time of a week of guide on a Dreambox One.

### Remote
<!-- help: remote -->

A remote control for the box in the browser, with a live picture of the TV screen, and **Message to the TV** to show a short text on screen. Opens from the remote icon at the top on every page.

### Box admin
<!-- help: box-admin -->

**Overview** shows the box, its disks, and the **Change log** of every change made through DreamHub. **Diagnostics** runs the box self-test. **Backup and restore** of DreamHub's own settings and dashboard are under **Settings → System** (admins running Docker apps can also include each app's own configuration in that backup).

**Services & logs** (admins only) shows the box's **services** — enigma2, DreamHub, Samba and the others — with their state; **Restart** (or **Start**, when one is down) shows first what it interrupts, such as a recording running right now. Below that, logs — DreamHub's own and the box's — live if you like; **Logs** next to a service shows just its lines, and crash logs.

**Advanced** (admins only) is for people who know their way around a shell: a **script library** (upload a script, run it with a confirm) and a **terminal** — off by default, needs DreamHub's own password protection turned on and re-entered to switch on, with a standing warning while it's on.

### Apps
<!-- help: apps -->

The page has three tabs in a fixed order — **System software**, **Extensions**, **Apps** — each opening with one sentence saying what it is. DreamHub remembers the tab you looked at last (per browser).

**System software** is the box's own software (DreamOS): TV plugins, skins and tools. Search at the top, then the filters **Installed**, **Available** and **Updates**, and category chips (Extensions, Skins, System plugins, Drivers, Channel logos, Kernel modules, System). When updates are waiting, a bar says "3 updates available" with **Update all** — one preview lists every package apt would touch, and one confirmation applies them. Each row has one action (Install, Update or Remove) and a **⋯** menu with the details (version, where the description comes from). Every row says in a sentence what the package is, in your language; where DreamHub has no text of its own it shows the package's own description, and for a package with neither, an optional **Ask the AI** entry appears in **⋯** only if an AI key is set up — nothing needs it. A package that a DreamHub feature uses says "Needed by Recordings" (and so on), and removing one asks first. DreamHub's own list of descriptions is still short; it grows as the box's package feed is read. The gear opens **Package sources (feeds)** — switch one on or off, or add one, each with a preview; adding warns that these feeds are unsigned, so only add one you trust.

**Extensions** are small add-ons for DreamHub itself — for example one that switches a smart plug. An extension is a package: install it under System software (its name starts with `dreamhub-ext-`). Each shows as a card with its own status and actions (never its own web page); an extension that needs something typed in, such as a device key, has a **Settings** button.

**Declarative extensions: what they may do.** An extension that is only a manifest (a list of devices, values to read and buttons) runs no program; DreamHub reads it and nothing else. It can only talk to the addresses it lists, and only to devices on your home network unless you ticked *may use the internet*; it can never reach the box itself (the TV software, DreamHub's own pages, the terminal). Every change, every button press and every secret goes through the usual preview and confirmation, and a changed manifest that adds an address, a button or internet access is highlighted before you confirm. Passwords and tokens you type in are kept on the box in its database (readable only by DreamHub, like the AI key); they are never shown again, never leave in a backup or export, and are sent only to the extension's own devices. A *probe* ("Try it") reads each value once with a plain GET request: a device that switches when it is merely asked (some plugs do) will switch, so probe only addresses you know. A *webhook* address contains its secret; treat it like a password, do not post it in a chat or a log, and replace it (rotate) if it leaked. If your box sits behind a reverse proxy, all callers look the same to DreamHub, so a flood of wrong guesses makes the real sender wait for a minute. A value read from a media or timer feature is shown to everyone who sees the card only if they could read it themselves.

**Docker (beta)** — optional: extra apps in containers, not needed for normal operation. *Beta:* simple containers work; fixed ports, mounting folders from your disks and some compose files are still limited, and a refused compose file says why. **Apps** are programs that run in their own Docker container on the box, separate from the TV software. If Docker isn't set up yet, a note says so and points to **Docker settings** (the gear at the top right of the tab), where **Set up Docker** shows what it will do before it does it. A short hand-picked list (AdGuard Home, Pi-hole, Nextcloud, Vaultwarden, Syncthing, Home Assistant, qBittorrent) shows whether each works on your box's processor, its memory and port, and a note for this box; a search for anything else on Docker Hub is folded away (a result can be checked for compatibility or added as your own container). **Install** opens a small form: the **web port** (changeable only where the app's image allows it — otherwise it says so), the **disk** to store the app on (only real disks, never the box's own flash memory) and any field the app requires; **Show advanced options** adds every other field plus free rows for **environment variables**, **ports** and **folders** and a memory limit. A secret field (a password) is hidden while you type and never shown again. Then it shows exactly what will be created — images, ports, the folders on the box, the variables — checked against free memory, disk space and ports, before anything runs. The tab has two views: **My containers** (what is on the box) and **App catalog** (what you can add). In **My containers** each row has the app's picture, name, a status dot, its port, an **Auto-start** switch (start it whenever the box starts; saved at once) and one main button — **Open** while it runs, otherwise **Start**. Behind **⋯**: **Stop**, **Logs**, a **Console** (an interactive shell inside the app — same off-by-default, password-re-entry, standing-warning treatment as the terminal), **Edit** (the same form as at install, filled in; it shows what will change and makes the app again — its data stays), **Back up**, **Update** and **Remove** (its settings are kept). An "Update available" badge and a notification appear when a newer version exists — DreamHub never updates one on its own. Some apps are more than one container — Nextcloud brings its own database, handled together with it and listed under it with its own logs. Apps share the box's network, so each port is the box's port; on the reference box port 53 is already taken by the box's own DNS resolver, so AdGuard Home and Pi-hole can't be installed there. In **Settings → System → Backup** each app folder is listed with its size: a database is in by default, bulk files (Nextcloud's, downloads) only when ticked.

**Add container** (above the list, admins only) is for anything that isn't in the list: paste a **compose file** (`docker-compose.yml`), type an **image name**, or paste an **Unraid template** (the XML; DreamHub bundles no templates and downloads nothing — it only reads the ports, folders and variables). You get the same form as for the list, built from the file: every `${NAME:-default}` in it becomes a field. What is understood: services with image, ports, volumes, environment, command, a memory limit, restart and depends_on; anything else (build, networks, secrets, …) is refused with a reason. Apps use the box's own network, so a port can't be mapped to another one — the app listens on its own port. A folder written `.` or `./name` is kept in the app's storage; any other folder must be below `/media`. **You are responsible for what you run**: DreamHub refuses a container any folder of the box's system (`/`, `/etc`, `/usr`, `/var`, `/boot`, `/proc`, `/sys`, `/dev`), DreamHub's own data, the Docker socket and other apps' data — also through links — and asks for an extra confirmation before it gives a container privileged mode, extra capabilities, devices or the box's process view; it cannot judge the software itself. Containers you added are edited and updated through **⋯** like the others, except ones added before editing existed (remove and add those again).

**Updates.** When something can be updated, a box at the top of the page says how many packages and containers, with **Update all**. The preview lists everything in the order it runs: the containers one after the other, then the packages. Containers go first because an update that does not run goes back to the previous version by itself, so nothing else changes if one fails; the packages follow only when every container is fine. If a step fails, the rest is not started, and the result says what was done, what failed and what was not started. DreamHub itself is not part of it (a note says so); update it under **Settings → System → About DreamHub** (see *Updating*). While a recording runs, or one starts within 15 minutes, nothing is updated: the box says "A recording is running … Updates are possible again at 21:15." **Automatic updates** are off by default: one switch for containers in Docker settings, one for packages in Package settings (only packages that do not restart enigma2); "Only update between" limits them to a time of day (pressing Update yourself is only held back by a recording). Their results appear in the notifications. In a container's **⋯** menu, **Update** updates that one and **Roll back** (shown while the previous version is kept) goes back; Docker settings list the kept versions and can remove them. In a package's menu, **Downgrade to …** appears only if the feed still has an older version. If an app keeps stopping, a notification links to its logs.

**Backup and bringing apps back.** *Settings → System → Backup & restore* can include your apps: tick the apps (and which of their folders) and download the backup, or choose **Save on the box** to keep it on the recording disk, which suits big app data. The apps part also holds the passwords DreamHub made for the apps, so keep the file safe. On a new box, restore the file; the apps then appear under *My containers* as **Apps you can bring back** with *Install again*. You see the usual preview and confirm it. An app you removed earlier is offered the same way.

**Docker settings** (the gear) show whether Docker is running and **Set up Docker** when it isn't. Once it runs they also hold: where **new** apps keep their data (the disks with their free space; apps already installed stay where they are), whether new apps start with the box, the **address for the Open links** (empty = the address you opened DreamHub with; set it to the box's own address when you reach DreamHub through a reverse proxy — apps open on their own port over plain http), the **memory limit per app** (automatic, or your own number for containers that don't state one), how often DreamHub **checks for newer app versions** and whether it tells you in the notifications (nothing is ever updated by itself), and **Clean up unused images** — old images stay on the disk when an app is removed or updated; it shows how much space that is, then lists them and asks before it deletes. Each setting shows **Reset** only when it differs from the default. At the end: **Uninstall Docker**. Uninstalling is a preview like every change: it stops and removes the app containers DreamHub created, switches off Docker and DreamHub's Docker helper, stops loading the overlay kernel module at boot and deletes the Docker program files. It asks one question — **Also delete the app data?** — and by default keeps it, so the apps come back after setting Docker up again. Recordings and the TV software are never touched.


---

### Automations: what they may do
<!-- help: automations -->

**Automation** in the menu (administrators only, not in the simple view) holds *rules*: "when a recording finishes, tell me", "every night at 02:30, …". A rule is a small, readable object — a trigger, optional conditions, up to five actions and limits — not a program. You start from one of the template cards (disk almost full, timer conflict, a followed episode without a timer, a service or app that fails, a daily disk check, …), from an empty rule, or an assistant suggests it. A new rule is built in three steps: **When** (the trigger and optional conditions), **Then** (what DreamHub does, a name and the limits) and **Review**, where the rule is shown as a plain sentence next to a dry run over the last days. The sentence always follows the stored rule. Nothing is saved before the review. Once you have confirmed it, it runs **without any AI**.

**You confirm the rule, once.** Saving shows the rule in plain words, what it can touch, its limits and a dry run against recent events. Nothing is saved before that. Changing a rule opens its form; saving shows the same plain-words summary and dry run again. A rule an assistant suggested only appears as a proposal in **AI Center → Inbox**; it does not exist until you open it and confirm it there, and it is created as yours.

**Three stages.**

- **Tell me** — when it triggers, you get a message ("this would run now"). Nothing else happens. New rules suggested by an assistant start here or at *Suggest*.
- **Suggest** — when it triggers, DreamHub puts a proposal in AI Center → Inbox, like an assistant's. You read the preview and apply it, or not. The default for new rules.

- **Run by itself** — when it triggers and the box is free, the rule applies its changes without asking. This is the only stage that does, and it is the most guarded: it is off until you also switch on **Settings → Access → Let automations run by themselves** (off by default; "Pause all automations" next to it stops every rule at once), and each rule needs its own confirmation. **No rule changes your recordings, timers, channel list or settings without you applying it — unless you confirmed that one rule for this stage.**

**Letting a rule run by itself.** Open a saved rule and choose *Run by itself*. You see the whole *envelope* on one screen, and nothing is changed until you tick that you read it, enter your PIN if your profile has one, and press the button:

- *What it may do on its own* — the exact plannable changes it may use (today only **create a timer**, and actions of your own extensions that are marked safe to automate and do not need the system permission) and what kind of change each may make. Deleting, trashing, cutting, channel lists, settings and packages can never be automated.
- *How much* — at most 24 runs a day (or the lower limit you set), 4 an hour, at least 5 minutes between two runs, at most 10 changes per run; per extension, over all rules, 6 actions an hour and 24 a day.
- *What stops it* — the first failed run pauses it; reaching a limit pauses it; while the box is recording or cannot be reached it leaves a proposal instead; a plan that does more than you confirmed, or that carries a warning, becomes a proposal instead; and **any change** to the rule, to a capability it uses or to an extension action it calls cancels the confirmation: the rule goes back to *Suggest* and the bell tells you.

Only you can do this — an assistant can never raise a rule to this stage, an app's token cannot either, and the profile's PIN is always asked when it has one. Every run is in the change log under the rule's number. You can pause the rule at any moment, also while it is running (it stops before its next action). Running by itself uses no AI.

**What a rule can and cannot do.**

- It can only use what DreamHub lists as readable values, events and plannable changes — the same list an assistant sees. It cannot run commands, change settings, install packages, touch the channel list, call an assistant, or create or change other rules.
- It acts with the rights of the profile that owns it, checked at every run. If that profile loses the right, the rule stops.
- Every rule has limits: runs per day and changes per run. A rule that fails three times in a row (a rule that runs by itself: once), plans more than it declared, or no longer fits the current DreamHub pauses itself and tells you in the bell. Over its daily limit it only waits for the next day.
- A rule's own actions never start a rule of the same chain, and each event starts a rule at most once.
- Text that comes from the box (programme or recording names) can fill a slot such as a message or a timer name, but never decides *what* the rule does. It is shortened and cleaned of control characters.
- Time triggers use your time zone; if DreamHub was off at the time, that run is skipped, not made up later.

**Pause, lower, delete.** Each rule has a pause switch, can always be put back to *Tell me*, and can be deleted. Every run is listed on the rule's page with its result.

**What was and was not checked.** The security review (milestone R7) covered the rule format and its limits, who may see, create, confirm and change a rule, what an assistant can learn from a dry run, the loop guard, the limits and how box text is shown. It was done against the mock box and DreamHub's tests. The *Run by itself* stage had its own review (R4): the confirmation, the hash that cancels it on any change, the caps, the loop guard, the switches and who may set the stage. It was done against the mock box and DreamHub's tests too; **no rule was run on a real box**, a loop that closes through a real device was only simulated, and "something is playing" is not detected (only recording).

## 4. DreamHub as an app
<!-- help: app -->

DreamHub can sit on your phone's home screen or your computer's desktop and open in its own window, without the browser's address bar. **Settings → General → DreamHub as an app** tells you what works in the browser you're using, with an **Install** button where the browser offers one.

- **iPhone / iPad:** in Safari, tap **Share → Add to Home Screen**. This works over plain `http://`, no further setup.
- **Android, Windows, Mac, Linux (Chrome, Edge):** browsers only offer to install over **HTTPS**. Set up the box's **secure connection** (next paragraph); until then, use the browser menu's **Add to home screen**: that gives you an icon, just without the app window.

**The box's own secure connection (HTTPS).** DreamHub also listens on port **8766** with a certificate from the box's own certificate authority — the plain `http://<box-IP>:8765` keeps working. No public authority issues a certificate for a home address, so each device is told **once** to trust the box's certificate: **Settings → General → Secure connection** has the download, the certificate's fingerprint to compare, and the steps for iPhone/iPad, Android, Mac, Windows and Firefox (which keeps its own list). Afterwards open the secure address (`https://<box-IP>:8766`; for the browser it is a place of its own, so you pick your profile once more there and move your bookmark or installed app over) — HD live TV is then decoded by your device's hardware, installing the app and notifications work too. The setup assistant mentions it as an optional step. The certificate can vouch only for addresses at home (such as `192.168.…`) and names ending in `.local`, `.lan`, `.home.arpa` or `.fritz.box` — it is worthless for any website on the internet, so trusting it costs nothing there. If you open the box under a further name, list it in `DREAMHUB_TLS_NAMES` (comma-separated, in `/etc/default/dreamhub`); `DREAMHUB_TLS_ADDR=off` switches the port off, another value (`:9443`) moves it. The certificate is made when the box's clock is right and renewed on its own, also when the box's address changes. To start over (devices must then trust again), stop DreamHub, delete the folder `tls` in its data directory and start it again.

**Putting DreamHub behind HTTPS with a name of your own** (optional, instead of the secure connection above) needs a second device that's always on and a name with a real certificate — for example:

- **Tailscale:** on any always-on machine in your network, `tailscale serve --bg http://192.168.1.50:8765` gives `https://<machine>.<tailnet>.ts.net`, reachable from your devices at home and away.
- **A reverse proxy** you already run (Caddy, Nginx Proxy Manager, Traefik on a NAS), with a domain that points to it, forwarding to `http://<box IP>:8765`.

Either way, enter that host name under **Settings → Access → Network → Additional allowed host names**, or DreamHub refuses requests addressed to it (protection against DNS rebinding).

Over HTTPS the browser also decodes HD channels in Live TV with the device's own hardware (it refuses that on plain `http://`), so HD plays more smoothly on phones. Confirmed with Nginx Proxy Manager. When the machine running the proxy is off, the HTTPS address is gone too — the box's own `http://<box IP>:8765` keeps working.

**Keep it in your home network.** For access from away, use Tailscale (or another VPN) rather than opening the proxy to the internet. If DreamHub is reachable from outside at all, switch on its login (**Settings → Access**) first: without it, anyone who reaches the address can control the box.

When the box is off or out of reach, the app opens anyway and says so, and connects by itself once the box answers again. Nothing from the box is kept on the phone: timers, recordings and the guide are always live.

---

## 5. Notifications on your phone
<!-- help: notifications -->

*In the browser:* the bell at the top has **Show notifications on this device**. The browser then shows a message when DreamHub is open in a background tab or the installed app — it needs HTTPS (or `localhost`) and your permission. With DreamHub closed, use ntfy below.

DreamHub can send a short message to your phone when something happens, through the free **ntfy** app (iPhone and Android; no account needed):

1. Install **ntfy** on your phone.
2. In DreamHub, **Settings → Notifications**: enter a **topic** — a name that's hard to guess; the page suggests one — and save.
3. In the ntfy app, subscribe to the same topic (server `ntfy.sh` unless you run your own), then press **Send a test** in DreamHub.

Choose which events send a message: a recording finished (on), a recording started (off), **a missing episode of one of your series airs in the next 24 hours** (on), a possible timer conflict (on), the disk is almost full (on), **a service on the box goes down or comes back** (on — Samba, the softcam, enigma2 itself; DreamHub checks every minute and only reports a service that stays down). The bell at the top of DreamHub shows all of these too, with or without push.

Privacy: a message's text (a recording's or series' title, a channel name) passes through the ntfy server. On the public `ntfy.sh` anyone who knows your topic could read it — that's why it should be hard to guess. For full control, run your own ntfy server and enter its address.

---

## 6. When something doesn't work
<!-- help: troubleshooting -->

- **Not sure what works on your box?** **Box admin → Diagnostics → Run self-test** asks the box everything DreamHub needs, without changing anything, and marks each answer: fine, worth a look, failed, or not applicable. **Download the box's answers** gives a zip that helps when reporting a problem.
- **DreamHub doesn't open:** check that the service runs — in the WebAdmin terminal, `systemctl status dreamhub`. Its log: `journalctl -u dreamhub -n 100`.
- **DreamHub doesn't start and the log says "address already in use":** another program on the box uses DreamHub's port (**8765**). Pick a free one in the TV plugin (**DreamHub → Settings → Backend port**; DreamHub restarts after you confirm), or in the terminal: `echo 'DREAMHUB_LISTEN_ADDR=:8800' > /etc/default/dreamhub && systemctl restart dreamhub` — then open `http://<box>:8800`.
- **A hub shows "unavailable" or "degraded":** the sidebar dot and the bell say why and how to fix it (a missing key, a plugin that isn't installed, the box not answering). **Box admin → Diagnostics** lists all of them.
- **"Could not reach WebControl":** the box's interface is restarting or busy (after an update, enigma2 takes a moment), or it now asks for a login — enter it under **Settings → Services → WebControl**.
- **File actions are missing in Recordings:** they need DreamHub to run on the box itself; the page says so when they're not available.
- **No pictures:** add a TMDB key and make sure *Load pictures, ratings and overviews* is on (Settings → Services → TMDB).
- **More detail for a bug report:** **Box admin → Services & logs → Logs**, set the level to *debug* for a while (it switches back by itself).

Problems and ideas: the repository's issue tracker on GitHub.
