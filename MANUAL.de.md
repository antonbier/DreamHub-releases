# DreamHub Benutzerhandbuch

DreamHub ist eine Web-Oberfläche für **Dreamboxen mit DreamOS** – entwickelt und getestet auf einer Dreambox One; siehe *Was du benötigst* für andere Modelle. Es läuft direkt auf der Box, neben dem eigenen WebControl der Box, und wird über einen Browser genutzt – auf deinem Smartphone, Tablet oder Computer. Dieses Handbuch deckt die Installation, die ersten Schritte und die Funktionen der einzelnen Bereiche ab.

DreamHub ändert von sich aus nichts: Jede Änderung an Aufnahmen, Timern oder Einstellungen auf der Box wird dir zuerst angezeigt („Vorschau · noch nichts geändert“) und wird erst ausgeführt, wenn du sie bestätigst. Jede angewendete Änderung wird unter **Gerät → Übersicht → Änderungsprotokoll** aufgelistet.

---

## 1. Installation
<!-- help: install -->

### Was du benötigst
<!-- help: needs -->

* Eine Dreambox mit DreamOS:
* **Dreambox One oder Two** (ARM64) – Paket `dreamhub_<version>_arm64.deb`. Entwickelt und getestet auf einer Dreambox One mit DreamOS 2.6 (Gemini/AIO Image).
* **DM900 UHD oder DM920 UHD** (ARMv7) – Paket `dreamhub_<version>_armhf.deb`. Auf dieselbe Weise gebaut. Ein Nutzer berichtet, dass es auf einer DM920 UHD läuft; einen ausführlichen Test gab es nicht, die Grundfunktionen sollten aber gehen. Feedback ist willkommen.
* MIPS-Boxen (DM7080, DM820, DM520, DM525) werden derzeit noch nicht unterstützt, ältere Dreamboxen ohne DreamOS überhaupt nicht.


* Ein **VU+ oder ein anderer OpenWebif-Receiver** funktioniert als zweite Box, ohne dass darauf etwas installiert wird – siehe *Mehr als eine Box* weiter unten und die Tabelle in der [README](README.md#what-you-need).
* Die Box in deinem Heimnetzwerk. DreamHub nutzt Port **8765**; WebControl bleibt auf Port 80.
* Ca. 15 MB Speicherplatz auf der Root-Partition für das Programm; seine Datenbank liegt auf `/data`.

### Aktuell: Manuelle Installation des Pakets
<!-- help: install-manual -->

Am einfachsten geht es über den **Paket-Feed** von DreamHub (siehe unten *Über den Feed*): einmal eintragen, danach funktionieren Installation und Updates wie bei jedem anderen Plugin. Wenn du magst, installierst du das `.deb`-Paket direkt, wie hier beschrieben.

1. **Lade das Paket** für deine Box aus dem neuesten Release herunter: [github.com/antonbier/DreamHub-releases/releases](https://github.com/antonbier/DreamHub-releases/releases) – `…_arm64.deb` für eine Dreambox One/Two, `…_armhf.deb` für eine DM900/DM920.
2. **Kopiere es auf die Box**, zum Beispiel nach `/tmp/` – über die Netzwerkfreigabe der Box (Samba), mit einem SCP-Programm wie WinSCP oder Cyberduck (Benutzer `root`) oder von einem USB-Stick.
3. **Installiere es.** Öffne ein Terminal (Shell) auf der Box. Es ist kein SSH-Client erforderlich: Das eigene **WebAdmin** der Box bietet ein Terminal im Browser unter `http://<box>/webadmin/terminal/` (Anmeldung als `root`). Führe dann aus:
```sh
dpkg -i /tmp/dreamhub_*.deb

```


Falls Pakete fehlen, hole sie mit `apt-get -f install` nach.

Dadurch wird der DreamHub-Dienst installiert und gestartet sowie das **DreamHub**-Plugin zur Plugin-Liste des TVs hinzugefügt. Die Benutzeroberfläche des TVs (enigma2) wird bei der Installation **nie neu gestartet**: Das Plugin erscheint beim nächsten Neustart von enigma2 (die Meldung am Ende der Installation weist darauf hin), den du machst, wenn es dir passt – DreamHub selbst läuft sofort im Browser.
4. **Überprüfe es:** Öffne DreamHub (nächster Abschnitt), gehe dann zu **Gerät → Diagnose → Selbsttest ausführen**. Es fragt die Box nach allem ab, was DreamHub benötigt, und sollte durchgehend grün anzeigen.

### Über den Feed
<!-- help: install-feed -->

DreamHub hat einen eigenen Paket-Feed für das **Box-Paket** (`dreamhub`; das Server-Paket installierst du aus seiner Datei, siehe unten), damit Installation und Updates ohne Kopieren von Dateien gehen. **Ab Version 0.13.0 trägt das Paket ihn selbst ein**, bei der ersten Installation (`/etc/apt/sources.list.d/dreamhub-feed.list`, steht in der Ausgabe der Installation); einen selbst eingetragenen Feed lässt es in Ruhe, und einen, den du entfernst oder ausschaltest, trägt es nicht wieder ein. Um DreamHub gleich aus dem Feed zu installieren, trage ihn einmal ein, in der Shell der Box (das Terminal im WebAdmin genügt), für eine Dreambox One/Two:

```sh
echo "deb [trusted=yes] https://antonbier.github.io/DreamHub-releases/apt/arm64 ./" > /etc/apt/sources.list.d/dreamhub.list
apt-get update && apt-get install dreamhub
```

Für eine DM900/DM920 nimmst du `armhf` statt `arm64` in der Adresse. Statt der Shell kannst du auch im WebAdmin unter der Feed-Verwaltung (**„Neues \*-feed.list hochladen“**) eine Datei `dreamhub-feed.list` mit dieser einen Zeile hochladen und `dreamhub` dann wie jedes andere Plugin installieren. Der Feed ist nicht signiert (`[trusted=yes]`, wie die anderen Dreambox-Plugin-Feeds): Du vertraust der Adresse, trage sie also nur aus diesem Handbuch ein. `apt` holt fehlende Pakete (etwa `ffmpeg`) selbst.

Sobald der Feed eingetragen ist, findet DreamHub dort eine neue Version von sich selbst (siehe *Aktualisieren*).

### Auf einem Raspberry Pi oder Server
<!-- help: install-server -->

DreamHub muss nicht auf der Box laufen. Auf einem Raspberry Pi (Pi 3, 4 oder 5 mit Raspberry Pi OS), einem Mini-PC oder einem Heimserver mit Debian oder Ubuntu installierst du stattdessen **`dreamhub-server`**, und es steuert die Dreambox über das Netzwerk.

1. Lade `dreamhub-server_<Version>_<Architektur>.deb` herunter – `arm64` (Pi mit 64-Bit-System), `armhf` (32-Bit-Pi-OS) oder `amd64` (normaler PC) – und installiere es: `sudo apt install ./dreamhub-server_*.deb` (es bringt `ffmpeg` mit).
2. Sage ihm, mit welcher Box es sprechen soll: `sudo dreamhub-setup 192.168.1.50` (die Adresse deiner Dreambox). Es sieht sich die Box zuerst an und schreibt nichts, wenn die Adresse nicht wie eine Dreambox antwortet.
3. Öffne `http://<dieser Rechner>:8765` im Browser.

DreamHub **liest** zunächst nur. Damit es Dinge auf der Box ändern darf (Timer, Senderlisten – jede Änderung zeigt weiter zuerst eine Vorschau), führe `sudo dreamhub-setup 192.168.1.50 --writes` aus.

**Was auf diesem Weg fehlt:** alles, wofür DreamHub auf der Box selbst laufen muss – Aufnahmen umbenennen, verschieben und löschen, die Picon-Ordner der Box, ihre Dienste und die Docker-Apps. Der Selbsttest zeigt diese Zeilen als Warnungen. Wenn du sie brauchst, installiere zusätzlich das normale Paket `dreamhub` auf der Box und koppele beide (siehe *Mehr als eine Box*). Das Server-Paket ersetzt das Box-Paket: Installiere nicht beide auf demselben Rechner.

### Aktualisieren
<!-- help: updating -->

**Aus DreamHub heraus:** **Einstellungen → System → Über DreamHub** zeigt, ob es eine neuere Version gibt, mit **Was ist neu?** (die Änderungsliste, auf Englisch) und **Update installieren …**. DreamHub schaut zweimal am Tag nach und meldet eine neue Version einmal in den Benachrichtigungen; **Jetzt prüfen** fragt sofort. Das Update ist eine Vorschau wie jede Änderung; nach dem Bestätigen lädt DreamHub das neue Paket, installiert es und startet neu – die Seite wartet darauf und lädt von selbst neu (etwa eine halbe Minute; Fernsehen und Aufnahmen laufen weiter). Es kommt aus dem DreamHub-Feed, wenn die Box ihn hat, sonst vom Release auf GitHub und wird vor dem Installieren mit seiner veröffentlichten Prüfsumme verglichen. (Die Prüfsumme erkennt einen beschädigten Download; wie der Feed, der nicht signiert ist, beruht sie auf dem Vertrauen in das Repository `DreamHub-releases` – dasselbe Vertrauen wie beim Installieren des `.deb` von Hand.) Danach sagt die Karte, dass aktualisiert wurde, und wenn sich das TV-Plugin geändert hat, dass es beim nächsten enigma2-Neustart lädt. Schlägt eine Installation fehl, sagt die Karte das mit dem Ende der Ausgabe, und die bisherige Version läuft weiter. Die Box braucht dafür Internet und eingeschaltete Änderungen an der Box (auf einem Server: `dreamhub-setup … --writes`).

**Von Hand:** Installiere das neuere `.deb` auf dieselbe Weise (`dpkg -i …`) oder mit dem Feed `apt-get update && apt-get install dreamhub`. Deine Einstellungen, Schlüssel und dein Dashboard bleiben erhalten: Sie liegen in `/data/dreamhub`, was von einem Update niemals angerührt wird.

### Deinstallieren
<!-- help: uninstalling -->

```sh
dpkg -r dreamhub      # entfernt das Programm, behält deine Einstellungen in /data/dreamhub
dpkg -P dreamhub      # löscht auch /data/dreamhub — Einstellungen, Schlüssel, Änderungsprotokoll

```

---

## 2. Erste Schritte
<!-- help: first-steps -->

### DreamHub öffnen
<!-- help: open -->

Jede dieser Optionen funktioniert:

* **Vom Fernseher aus:** Öffne die Plugin-Liste (oder das blaue Erweiterungen-Menü), wähle **DreamHub** und scanne den QR-Code mit deinem Smartphone.
* **Über den Namen:** `http://<box-name>.local:8765` – die Box meldet sich selbst im Netzwerk an (mDNS). Die meisten Smartphones und Computer finden sie auf diese Weise.
* **Über die IP-Adresse:** `http://<box-IP>:8765`, z. B. `[http://192.168.1.50:8765](http://192.168.1.50:8765)`.

Tipp: Füge es zum Startbildschirm deines Smartphones hinzu – es öffnet sich dann wie eine App. Siehe [Abschnitt 4](https://www.google.com/search?q=%234-dreamhub-als-app).

### Der Einrichtungsassistent
<!-- help: setup-assistant -->

Beim ersten Aufruf startet ein kurzer Assistent: Sprache, Verbindung zur Box, ein Passwort für DreamHub, welche Hubs angezeigt werden sollen, deine Senderliste und optional ein KI-Schlüssel. Alles darin kann später unter **Einstellungen** geändert werden, und **Einstellungen → System → Einrichtungsassistent ausführen** startet ihn erneut. Die Einstellungen sind in Tabs unterteilt – Allgemein, Aufnahmen, Dienste, Benachrichtigungen, Hubs, Zugriff, System – und jede Einstellung kann mit **Strg+K** gefunden werden.

### Profile: Eines pro Person
<!-- help: profiles -->

Unter **Einstellungen → Profile** legt ein Admin ein Profil für jede Person im Haushalt an. Jedes Gerät fragt einmal „Wer schaut gerade?“ und merkt sich die Antwort; ein Profil kann eine PIN haben (Admins benötigen eine, sobald es mehr als ein Profil gibt). **Zuschauer** sehen Live-TV, Aufnahmen und Plex, nutzen den TV-Guide und die Fernbedienung und planen Aufnahmen – sie benennen oder löschen keine Aufnahmen, ändern keine Timer anderer Personen und sehen keine Einstellungen über ihre eigenen hinaus. Pro Zuschauer kann ein Admin „Aufnahmen planen“ sowie „Aufnahmen umbenennen und löschen“ erlauben. **Startseite gestalten** legt die Startseite fest, die jeder Zuschauer sieht, bis er seine eigene ändert.

### Mehr als eine Box
<!-- help: boxes -->

Unter Einstellungen → **Boxen** (Admins) fügst du eine weitere Dreambox mit ihrer Adresse und – falls ihre Weboberfläche eines verlangt – ihrem Login hinzu; **Testen** zeigt, ob sie antwortet und der Login stimmt. Ab zwei Boxen erscheint oben in der Leiste eine **Box-Auswahl**, und die Startseite zeigt einen schmalen Streifen mit dem, was die anderen Boxen gerade tun – antippen wechselt. Alles darunter (TV-Guide, Timer, Aufnahmen, Fernbedienung) bezieht sich dann auf die gewählte Box, und die Adresse sagt welche (`/b/wohnzimmer/guide`), Lesezeichen funktionieren also. DreamHub merkt sich die letzte Box pro Gerät.

Eine so hinzugefügte Box wird über ihre Weboberfläche gesteuert: Live-TV, Guide, Timer, Senderlisten, Fernbedienung und das Ansehen ihrer Aufnahmen. Aufnahmen direkt verwalten, Logos, Apps und die Systemwerkzeuge brauchen DreamHub auf der Box selbst. Eine Box, die nicht antwortet, ist in der Auswahl ausgegraut und die Statuszeile sagt es – DreamHub wartet nie auf sie. In den Einstellungen eines Profils lässt sich begrenzen, welche Boxen es nutzen darf (nichts angehakt heißt alle). Eine Box zu entfernen lässt DreamHub sie und alles Gespeicherte vergessen; die Box selbst bleibt unberührt.

**Andere Receiver als eine Dreambox (VU+ und weitere OpenWebif-Boxen).** Alles mit dem OpenWebif-Webinterface lässt sich genauso als Basic-Box hinzufügen – bisher geprüft an einer **VU+ Solo2 mit dem Image VTi 15**. Live-TV, TV-Guide, Timer (mit dem, was jeder Tuner gerade tut), Senderlisten, Fernbedienung, Screenshots und das Ansehen ihrer Aufnahmen funktionieren – eine Aufnahme lässt sich auch auf dem Fernseher dieser Box abspielen, vom Anfang an; DreamHub selbst läuft auf einer Dreambox oder einem Server und erreicht den Receiver übers Netz (für DreamHub selbst haben diese Boxen zu wenig Platz und kein systemd). Dort nicht verfügbar: AutoTimer und SerienRecorder, sofern die Box sie nicht hat, und alles, was DreamHub auf der Box selbst braucht. Receiver im WLAN sind bei großen Senderlisten langsam – wenn möglich per Kabel anbinden. Noch nicht geprüft: eine Box mit eingeschaltetem Web-Login sowie andere Hersteller und Images; sag uns, was du findest.

Eine ausgeschaltete Box lässt sich **von hier einschalten**. Unter Einstellungen → Boxen → Bearbeiten → *Einschalten, wenn die Box aus ist* wählst du **Wake-on-LAN** (MAC-Adresse der Box eintragen; Wake-on-LAN muss in den Netzwerkeinstellungen der Box an sein, und DreamHub im selben Netz) oder **die Aktion einer Erweiterung** (zum Beispiel die Steckdose, an der die Box hängt). Eine Box, die nicht antwortet, zeigt dann einen Knopf **Einschalten** – in der Statuszeile, wenn es die gewählte Box ist, im Streifen der Startseite bei den anderen. Wake-on-LAN geht sofort raus; die Aktion einer Erweiterung wird vorher mit deren eigenem Wortlaut gezeigt, weil die Aktion einer Steckdose meist *umschaltet* – prüfe, dass „ein“ dasteht. Danach schaut DreamHub bis zu drei Minuten lang alle paar Sekunden nach der Box, und sie ist wieder normal, sobald sie antwortet. Jedes Profil, das die Box nutzen darf und die Fernbedienung hat, kann das.

**Eine Aufnahme auf einer anderen Box ansehen.** Mit zwei oder mehr Boxen bietet eine geöffnete Aufnahme **Auf <Box> abspielen** für jede andere Box, die an ist und DreamHub hat (die Box mit DreamHub oder eine gekoppelte). Die Aufnahme startet im Movie-Player dieser Box, mit Vor- und Zurückspulen; DreamHub liest sie von der Box, auf der sie liegt – das darf auch eine Basic-Box sein. Die Box, auf der sie läuft, muss die Adresse von DreamHub erreichen können (dieselbe wie bei Plex – *StreamBase* unter Erweitert in den Einstellungen der Box). Braucht die Berechtigung für die Fernbedienung.

### API und Tokens
<!-- help: tokens -->

Für Apps und Skripte (eine DreamDroid-artige App, Home Assistant, eigene Werkzeuge): **Einstellungen → Zugriff → API-Tokens** erstellt ein Token – es wird nur einmal angezeigt, ist einzeln widerrufbar und handelt als das Profil, das es erstellt hat (es sieht also dessen Boxen und Berechtigungen). Sende es bei jeder Anfrage mit:

```
curl -H "Authorization: Bearer <token>" "http://dreambox.local:8765/api/v1/boxes"
curl -H "Authorization: Bearer <token>" "http://dreambox.local:8765/api/v1/timers?box=bedroom"
```

- **`box=<id>`** sagt, um welche Box es geht; `GET /api/v1/boxes` listet die erlaubten IDs. Ohne Angabe gilt die erste Box, die du nutzen darfst – ein Skript für eine Box funktioniert also weiter, wenn eine zweite dazukommt.
- **Änderungen brauchen zwei Schritte.** Ein Aufruf, der auf `/plan` endet, ändert nichts und liefert eine Vorschau mit einer `id`; `POST /api/v1/plans/<id>/apply` führt die Änderung auf der Box aus, für die der Plan erstellt wurde. Ein Plan gilt zehn Minuten und nur einmal. Anfragen, die etwas ändern, brauchen außerdem den Header `X-DreamHub-Request: 1`.
- **Die vollständige Beschreibung** aller Aufrufe ist `/api/v1/openapi.json` (unter der Token-Liste verlinkt).

### Assistenten (MCP)
<!-- help: mcp -->

Ein KI-Assistent, den du schon nutzt (Claude Code, Cursor, jeder Client, der das Model Context Protocol spricht), kann Dinge auf deiner Box nachschlagen und Änderungen **vorschlagen**. Er kann nie selbst etwas ändern: Was er vorschlägt, landet in **KI-Zentrale → Eingang**, wo du genau siehst, was passieren würde, und es bestätigst – oder nicht.

**Einschalten.** Standardmäßig aus. **Einstellungen → Zugriff → Assistenten** schaltet den Endpunkt `/mcp` ein. Dann öffne **KI-Zentrale → Verbinden**. Der Tab vergleicht den eingebauten Assistenten (fragt in DreamHub, bezahlt mit deinem API-Schlüssel) mit einem externen (ein Programm auf deinem Rechner, bezahlt über dessen eigenes Abo) und führt dich in drei Schritten durch: Programm wählen, Verbindung benennen und den angezeigten Befehl bzw. die Konfiguration kopieren (sie enthält bereits die Adresse deiner Box und den neuen Schlüssel; der Schlüssel wird nur einmal angezeigt), dann nach dem Ausführen **Verbindung testen** – es meldet, ob die Verbindung schon etwas abgefragt hat. Unter *Verbindungen und was sie dürfen* hat jede Verbindung Schalter in drei Gruppen (nachschlagen, Vorschläge machen, sensible Daten); jede Änderung wird sofort gespeichert. Claude Desktop und claude.ai im Browser lassen sich so nicht verbinden (sie brauchen eine andere Anmeldung, die DreamHub bewusst nicht anbietet). Solange unter Einstellungen → KI *Erlaubte Werkzeuge* ausgefüllt ist, werden die Schalter des eingebauten Assistenten dort gesetzt.

**Was ein Token kann.**

- Lesen, was seine Liste erlaubt: Aufnahmen, Timer und Konflikte, Programmführer, Serien, Speicherplatz, Hub-Status. Sensible Lesezugriffe (Wiedergabeverlauf, Einstellungen) sind aus, bis du sie ankreuzt.
- Einen Timer, einen Papierkorb-Eintrag oder eine Werbemarke vorschlagen. Jeder Vorschlag ist ein Vorschlag mit Begründung des Assistenten; du öffnest ihn, liest die Vorschau und wendest ihn an oder verwirfst ihn.
- Eine Erweiterungs-Beschreibung prüfen, deine Erweiterungen auflisten und beschreiben und eine neue oder geänderte Erweiterung vorschlagen (ein Vorschlag wie die anderen; du siehst, welche Hosts sie kontaktiert und was ihre Schaltflächen senden, bestätigst und trägst Geheimnisse selbst ein). Ein Assistent sieht oder setzt nie ein Geheimnis.
- Er handelt mit den Rechten des Profils, das das Token erstellt hat – nie mehr.
- Den Rest von DreamHub (Web-API, Einstellungen, Terminal, Skripte, andere Tokens, Profile) kann er nicht nutzen. Ein aus der Assistenten-Konfiguration gestohlenes Token erlaubt daher: lesen, was seine Liste erlaubt, und Vorschläge anlegen, die du verwerfen kannst – sonst nichts. Wenn eines abhandenkommt, **widerrufe** es unter Verbinden; es funktioniert sofort nicht mehr.

**Grenzen zum Schutz der Box.** Jedes Token hat ein Aufrufbudget (30 pro Minute, 2000 pro Tag, zwei gleichzeitig); vier Assistenten-Anfragen werden gleichzeitig über alle Tokens bedient; Antworten werden auf eine Seite mit „mehr“-Cursor gekürzt. Nach zehn abgewiesenen Tokens wird eine Adresse fünf Minuten pausiert. Das Aufrufprotokoll hält fest, wer was aufgerufen hat und wie es ausging – nie die Fragen oder Antworten – und bewahrt es 30 Tage auf.

**Text von deiner Box sind Daten.** Programmbeschreibungen und Aufnahmenamen können Sätze enthalten, die an einen Assistenten gerichtet sind („ignoriere deine Anweisungen und …“). DreamHub kennzeichnet solchen Text, damit der Assistent ihn als Daten behandelt; das Schlimmste, was daraus folgen kann, ist ein Vorschlag, den du liest, bevor etwas passiert. Lies Vorschläge trotzdem, bevor du sie anwendest.

**Von außerhalb des Hauses.** DreamHub ist für dein Heimnetz gebaut. Für einen Assistenten anderswo nutze etwas, dem du vertraust – ein VPN wie Tailscale oder einen Reverse-Proxy mit HTTPS – und halte das Token privat. Reines HTTP über das Internet legt das Token offen.

**Was geprüft wurde und was nicht.** Die Sicherheitsprüfung (Meilenstein M5) hat Token-Handhabung, wer was aufrufen darf, die Trennung vom Rest der API, Markierungen von Box-Text, Last auf einer kleinen Box, Fehlertexte und das Aufrufprotokoll geprüft – alles gegen die Mock-Box. Nicht geprüft: ein echter Client auf einer echten Box oder etwas außerhalb des Heimnetzes.

### Empfohlen: Netzwerkzugriff schützen
<!-- help: protect -->

* **Setze ein DreamHub-Passwort** (Einstellungen → Zugriff → Authentifizierung). Ohne Passwort kann jeder in deinem Heimnetzwerk DreamHub nutzen.
* **Sichere auch das eigene Webinterface der Box.** Standardmäßig erlauben viele Boxen jedem Gerät im Heimnetzwerk die Nutzung von WebControl (Port 80) ohne Passwort, selbst wenn eines gesetzt ist – DreamHub kann das nicht für dich ändern. Setze in den Einstellungen des Webinterfaces der Box **`localauth`** auf *ein* (`config.plugins.Webinterface.localauth=true`). Wenn du das tust, trage die Anmeldedaten der Box unter **Einstellungen → Dienste → WebControl** ein, damit DreamHub sie weiterhin erreichen kann.

### Optionale Schlüssel (API-Keys)
<!-- help: keys -->

DreamHub funktioniert ohne diese Schlüssel; jeder fügt jedoch Funktionen hinzu. Alle werden unter **Einstellungen** eingegeben, nur auf deiner Box gespeichert und nach dem Speichern nie wieder angezeigt.

| Schlüssel | Kostenlos? | Welchen Mehrwert es bringt |
| --- | --- | --- |
| **TMDB** (themoviedb.org → Einstellungen → API) | ja | Poster, Hintergründe (Backdrops), Bewertungen, Beschreibungen überall; „Cover speichern“; Namensvorschläge; Film-/Bewertungsfilter im TV-Guide |
| **OMDb** ([omdbapi.com/apikey.aspx](https://www.google.com/search?q=https%3A%2F%2Fomdbapi.com%2Fapikey.aspx)) | ja, 1.000 Abfragen am Tag | Rotten Tomatoes, IMDb-Bewertung und Metascore in den Details eines Titels |
| **fanart.tv** (fanart.tv → Profil → API) | ja | Das Logo des Titels über den Details: Details einer Aufnahme, Sendungsdetails im TV-Guide und in Empfehlungen, Serienseite. Benötigt auch TMDB (zum Finden des Titels); fanart.tv hat nicht für jeden Titel ein Logo |
| **KI-Anbieter** (OpenAI, Claude oder ein OpenAI-kompatibler Server; Einstellungen → KI) | kostenpflichtig pro Nutzung, begrenzt durch ein von dir festgelegtes Tages-Token-Budget | KI-Empfehlungen, KI-Namensvorschläge und der eingebaute Assistent |
| **ntfy-Thema** (Topic) | ja | Benachrichtigungen auf deinem Smartphone – siehe *Benachrichtigungen* unten |

Wenn das **DreamMediaInfo**-Plugin auf der Box installiert ist, nutzt DreamHub zuerst das bereits Bekannte und fragt TMDB nur nach dem Rest. Cover-Bilder, die ein anderes Plugin bereits neben einer Aufnahme gespeichert hat, werden immer bevorzugt.

---

## 3. Was ist wo
<!-- help: whats-where -->

### Übersicht (Home)
<!-- help: home -->

Ein Dashboard aus Kacheln: **Jetzt auf der Box** (was die Box anzeigt, mit Sender rauf/runter, Stummschaltung und Fernbedienung), anstehende Timer, Konflikte, Speicherplatz, kommende Serienfolgen, Hub-Status. **Bearbeiten** ordnet Kacheln neu an (Pfeile auf jeder Kachel bewegen sie nach oben oder unten; mit der Maus kannst du auch ziehen); **Eigene Kachel** erstellt deine eigene Ansicht (z. B. „Filme von TMDB mit einer Bewertung über 7 heute Abend“).

### Aufnahmen
<!-- help: recordings -->

Jeder Aufnahmeordner auf der Box, Ordner für Ordner.

* **Klicke auf einen Titel** für seine Details: Bild, Bewertungen, Beschreibung und **Auf dem Fernseher abspielen**.
* **Eine Aufnahme umbenennen**: Alle dazugehörigen Dateien werden zusammen umbenannt; Vorschläge kommen von der Aufnahme selbst, TMDB oder KI. Du kannst auch den Titel ändern, den die eigene Liste der Box anzeigt.
* **Mehrere auswählen** (oder **Alle auswählen**) und:
* **Nach Schema umbenennen** – Namen folgen deinem Schema aus *Einstellungen → Aufnahmen* (z. B. `{title} - S{season:02}E{episode:02} - {episodeTitle}`). Wenn ein TMDB-Schlüssel gesetzt ist, werden das fehlende Erscheinungsjahr eines Films und der Episodentitel automatisch von TMDB ergänzt. Aktiviere **In Ordner einsortieren**, um sie zusätzlich nach deinem Ordnerschema in Ordner zu verschieben (z. B. `{title}/Staffel {season:02}`); fehlende Ordner werden erstellt.
* **Cover speichern** – schreibt Poster und Hintergrundbild neben Aufnahmen, die noch keine haben, sodass die eigene Aufnahmeliste des TVs und andere Plugins diese ebenfalls anzeigen (DreamHub zeigt Poster an, ohne sie zu speichern). Bestehende Cover werden niemals überschrieben.
* **In den Papierkorb verschieben** – nichts wird sofort gelöscht; der Tab **Papierkorb** stellt Dateien wieder her oder löscht sie endgültig.


* **Von Hand schneiden** – in den Details einer Aufnahme: nenne die Teile, die herausgeschnitten werden sollen (`m:ss`, von … bis …), sieh dir jede Schnittstelle im Browser an, Vorschau, bestätigen. Dafür ist keine Werbeerkennung nötig; es wird nichts neu codiert (ein Schnitt kann bis zu eine Sekunde danebenliegen), und das Original kommt in den Papierkorb und lässt sich wiederherstellen.
* **Vorzeitig beendet** – eine Aufnahme, die deutlich kürzer ist als die angekündigte Sendung (Neustart, volle Platte, von Hand beendeter Timer), sagt das in der Liste. Eine selbst geschnittene Aufnahme wird nicht markiert.
* **Werbung** – in den Details einer Aufnahme findet **Werbung erkennen** die Werbeunterbrechungen anhand des Senderlogos direkt auf der Box im Hintergrund (es wartet, während die Box aufnimmt). Die Unterbrechungen werden auf einer Zeitleiste angezeigt; tippe auf eine, um von dort im Browser zu schauen. **Werbung schneiden …** hakt die Unterbrechungen ab, bei denen es sich sicher ist – bei unsicheren entscheidest du – zeigt an, was passieren wird, und schneidet sie ohne Neucodierung (Re-Encoding) heraus. Das Original behält alle seine Dateien und wandert in den Papierkorb, sodass es wiederhergestellt werden kann. Für mehrere Aufnahmen gleichzeitig: Wähle sie aus und klicke auf **Werbung erkennen**. Die Liste zeigt „14 Min. Werbung“ neben einer geprüften Aufnahme an.

Dateien werden nur geändert, wenn DreamHub direkt auf der Box läuft, und niemals, während eine Aufnahme noch geschrieben oder abgespielt wird.

### Plex-Mediathek
<!-- help: plex -->

Die Filme und Serien deines **Plex**-Servers. Verbinde ihn unter **Einstellungen → Dienste → Plex → Bei Plex anmelden**: Öffne plex.tv/link, gib den angezeigten Code ein und wähle dann deinen Server sowie die Adresse aus, die die Box verwenden soll (eine im Heimnetzwerk ist am schnellsten, oder gib eine eigene ein, z. B. einen Reverse Proxy). Die anderen Adressen des Servers werden als **Ausweichoptionen (Fallbacks)** aufbewahrt: Reagiert die verwendete Adresse nicht mehr, wechselt DreamHub zur nächsten, die als derselbe Server antwortet. Du kannst sie unter „Weitere Adressen des Servers“ bearbeiten. Wähle an derselben Stelle aus, welche Mediatheken angezeigt werden sollen.

* Detail- und Serienseiten zeigen das Logo des Titels aus Plex, wenn der Server eines hat.
* **Kacheln oder Liste** – oben rechts umschalten; nach Text oder Gesehen-Status filtern, nach Titel, kürzlich hinzugefügt, Jahr oder Bewertung sortieren.
* **Gesehen** stammt direkt von Plex: Das Markieren eines Films oder einer Episode ändert dies sofort in Plex (mit **Rückgängig**); eine ganze Staffel oder Serie zeigt zuerst jede Episode an.
* **Auf der Box abspielen** spielt die Datei direkt vom Plex-Server mit dem eigenen Player der Box von Anfang an ab – keine Plex-App auf der Box erforderlich – im Movie-Player der Box, sodass du mit der Fernbedienung spulen und pausieren kannst. Ob eine Datei abgespielt werden kann, hängt von ihren Formaten ab, die in den Details angezeigt werden.
* Wenn der Plex-Server ausgeschaltet ist, zeigt die Mediathek weiterhin an, was DreamHub zuletzt gesehen hat („Plex offline · Stand …“); Abspielen und Markieren funktionieren wieder, sobald er antwortet.
* Auf der **Startseite (Home)** kann über **Bearbeiten** die Kachel **Weitersehen (Plex)** aktiviert werden.

### Serien
<!-- help: series -->

Deine Serien aus dem **SerienRecorder**. Die Seite einer Serie zeigt geplante Episoden, weitere Ausstrahlungen im TV-Guide – jeweils mit Episodennummer und der Info, ob sie dir **fehlt** – deine Aufnahmen nach Staffeln und (mit TVmaze, standardmäßig aktiv) die vollständige Episodenliste mit dem, was aufgenommen, geplant, fehlend oder noch ausstehend ist. „Fehlend“ wird nur angezeigt, wenn deine Aufnahmen Episodennummern wie `S01E02` tragen. Tägliche Soaps, die der SerienRecorder fortlaufend nummeriert (`S00E8622`), werden anhand dieser Nummer zugeordnet; erst ab deiner ersten Aufnahme zählen vorherige Folgen als fehlend. Eine Ausstrahlung, die auf mehreren Regionalsendern gleichzeitig läuft (z. B. RTL-Fenster), wird nur einmal aufgeführt.

### TV-Guide (EPG)
<!-- help: tv-guide -->

Vier Ansichten, oben umschaltbar; deine Auswahl wird gespeichert:

* **Raster** – die klassische Zeitleiste, jeweils vier Stunden auf einmal.
* **Jetzt & Danach** – was auf jedem Sender gerade läuft (mit Fortschrittsbalken) und was als Nächstes kommt; **Umschalten** wechselt den Sender am Fernseher. Standard auf Smartphones.
* **Heute Abend** – was zwischen ca. 20:00 und 21:30 Uhr startet, als Bildkarten, bis zu eine Woche im Voraus.
* **Sender** – der gesamte Tag eines Senders als Liste.

**Nur Filme** und **Bewertung 7+** filtern jede Ansicht (benötigt TMDB). Jede Sendung öffnet ihre Details mit **Aufnehmen** und, während sie läuft, **Jetzt umschalten**.

### Live-TV
<!-- help: live-tv -->

TV-Sender direkt im Browser – auf dem Smartphone, Tablet oder PC, keine App erforderlich. Wähle einen Sender aus der Liste; **Schauen, was der TV zeigt** neben der Senderliste spielt das ab, was die Box gerade anzeigt. Die Box sendet ihr Signal unverändert und dein Gerät dekodiert es, sodass die Box selbst keine zusätzliche Arbeit hat. **Leertaste** pausiert, **M** schaltet stumm, **F** oder ein Doppelklick wechselt in den Vollbildmodus; bei mehreren Tonspuren (andere Sprache, Dolby) wählst du eine unter dem Bild aus.

Die Box hat einen Tuner: Ein anderer Sender funktioniert nur, solange der Fernseher und etwaige Aufnahmen auf derselben Frequenz liegen. Andernfalls weist das Bild darauf hin – **Schauen, was der TV zeigt** funktioniert immer. Im **TV-Guide** hat eine aktuell laufende Sendung auch die Option **Im Browser schauen**, und die **Startseite (Home)** kann eine **Live-TV**-Kachel mit deinen Sendern anzeigen (unter **Bearbeiten** aktivieren). Unter **Aufnahmen** spielt **Im Browser schauen** eine Aufnahme ab, inklusive einer Leiste zum Springen an beliebige Stellen.

Funktioniert in aktuellen Versionen von Chrome, Edge, Firefox und Safari. HD-Sender nutzen den eigenen Videodekoder deines Geräts; ältere oder sehr kleine Geräte können bei HD ruckeln.

### Radio
<!-- help: radio -->

Deine Radiosender mit Infos zu dem, was jetzt und danach läuft (sofern der Sender Programminformationen sendet). **Auf der Box abspielen** schaltet die Box auf diesen Sender um – du hörst über den Fernseher oder das angeschlossene Soundsystem; der aktuell laufende Sender wird oben angezeigt. **Im Browser** spielt einen Sender stattdessen direkt auf diesem Gerät ab (wie bei Live-TV muss der Tuner frei oder auf derselben Frequenz sein). Unter dem laufenden Sender steht die Zeile, die der Sender zu seinem Programm schickt (Interpret und Titel oder die Sendung), sofern er eine sendet – sie kann bis zu eine halbe Minute brauchen. Jede Sendung öffnet ihre Details mit **Aufnehmen**; die anstehenden Radio-Aufnahmen sind unten aufgelistet. Sender können auch über die Suche oben gefunden werden.

### Timer
<!-- help: timers -->

Alle Timer auf der Box, nach Tagen geordnet, mit Markierung wahrscheinlicher Tuner-Konflikte. **Deaktivieren** behält einen Timer in der Liste, ohne aufzunehmen; **Aktivieren** schaltet ihn wieder ein (mit vorheriger Konfliktprüfung). **Löschen** entfernt ihn – das Löschen eines Timers, der gerade aufnimmt, stoppt diese Aufnahme, worauf in der Vorschau hingewiesen wird. **Abgeschlossene aufräumen** entfernt alle abgeschlossenen Einmal-Timer in einer einzigen Vorschau (deaktivierte und wiederkehrende Timer bleiben erhalten). **AutoTimer** nimmt automatisch alles auf, was seinen Regeln entspricht; die Seite listet sie auf und **Zeigen, was aufgenommen würde** gibt eine Vorschau auf die dadurch hinzugefügten Aufnahmen. Jede Zeile hat eine sichtbare Aktion (ein-/ausschalten oder löschen, wenn er nicht geschaltet werden kann); der Rest befindet sich unter **⋯**.

### Empfehlungen
<!-- help: recommendations -->

Die KI wählt aus dem Programm eines Bouquets für die nächsten Stunden Empfehlungen aus, optional angepasst an deinen Geschmack (was du aufnimmst und verfolgst – **Was gesendet wird** zeigt die genaue Liste, bevor irgendetwas gesendet wird). Die Empfehlungen bleiben bestehen, bis du neue generierst, auch auf anderen Geräten. Klicke auf eine Empfehlung für ihre Details und **Timer erstellen**.

Jede Empfehlung sagt, **warum** sie gewählt wurde. Der **Filter** über den Empfehlungen grenzt sie auf den Abend (ab 18:00) oder auf solche ohne Timer ein. **Nicht für mich** bei einer Empfehlung (oder einer Serie darunter) wird für dich gemerkt: Dieser Titel – alle Folgen der Serie – wird nicht mehr vorgeschlagen; die Zeile unter den Empfehlungen (oder **Dein Geschmack**) bietet **Rückgängig**. **Serien zum Weiterschauen** listet die nächste Sendung der Serien, die du verfolgst (SerienRecorder, AutoTimer) oder aufgenommen hast, direkt aus dem Programm – ohne KI, es funktioniert auch ohne Anbieter. **Dein Geschmack** zeigt, was DreamHub von deiner Box weiß und was du als „nicht für dich“ markiert hast; von dort wird nichts gesendet. Das Programm enthält keine Genre-Angaben, das Feedback gilt deshalb pro Titel, nicht pro Genre.

### KI-Zentrale: Assistent
<!-- help: ai-assistant -->

**KI-Zentrale → Assistent** (nur Admins) lässt dich dem eingebauten Assistenten von DreamHub Fragen in deinen eigenen Worten stellen – „was verpasse ich diese Woche wegen überlappender Timer?“, „was ist von dieser Serie aufgenommen?“, „was könnte ich löschen?“. Er schlägt auf deiner Box mit denselben Nur-Lese-Werkzeugen nach wie ein externer Assistent und darf eine Änderung *vorschlagen*; angewendet wird nie etwas von ihm. Der Tab sagt vorab, was der Assistent kann (nachschlagen, Timer, Papierkorb-Einträge und Regeln vorschlagen) und was nicht (er baut keine Erweiterungen; dafür verweist er auf Apps → Erweiterungen oder einen externen Assistenten), und bietet ein paar Beispielfragen, die nur das Eingabefeld füllen. Frage eintippen, **Fragen** drücken (oder Strg+Enter) und bis zu einer Minute warten.

**Einschalten.** Er braucht einen KI-Anbieter: einen OpenAI-Schlüssel, einen Claude-Schlüssel oder die Adresse eines OpenAI-kompatiblen Servers, eingetragen unter **Einstellungen → KI**. Den eigenen Schlüssel einzutragen ist das Einverständnis. Bis dahin sagt der Tab, dass kein Anbieter eingerichtet ist. An den Anbieter gesendet werden: deine Frage, die früheren Gesprächsrunden und die Ergebnisse der Nachschlagen (Titel, Sender, Zeiten) – nie Einstellungen, Passwörter oder Tokens.

**Was du siehst.** Die Antwort, eine eingeklappte Liste *was der Assistent nachgeschlagen hat* und eine *worauf die Antwort beruht*, damit eine falsche Antwort auffällt. Schlägt er etwas vor, erscheint eine Karte im Gespräch (und daneben die Liste *Wartet auf dich*): Eine Änderung kannst du dort gleich prüfen, eine Regel öffnet sich in **KI-Zentrale → Eingang**, wo du die Vorschau liest und bestätigst oder verwirfst. Vorher ist nichts geändert.

**Dein Gespräch bleibt im Browser-Tab.** Die Box speichert nichts davon; Tab schließen oder **Gespräch löschen** vergisst es.

**Grenzen für Box und Rechnung.** Höchstens sechs Nachschlagen, 30.000 Token und 60 Sekunden pro Frage; eine Frage gleichzeitig pro Profil und zwei auf der Box. Tokens zählen gegen ein **Tagesbudget** (im Tab angezeigt; Standard 200.000, geteilt mit Empfehlungen) – ist es verbraucht, sagt der Tab das und stoppt. Der Assistent hat nur die für ihn erlaubten Werkzeuge (standardmäßig alle nicht sensiblen Lese- und Vorschlags-Werkzeuge); er hat nie die Werkzeuge zum Verwalten von Erweiterungen, Regeln kann er nur vorschlagen (ein Mensch bestätigt sie im Eingang), und den Rest von DreamHub (Einstellungen, Terminal, andere Profile) kann er nicht nutzen.

**Lokale Modelle sind ungetestet.** Die Anbieter-Einstellung nimmt jede OpenAI-kompatible Adresse (Ollama, LM Studio, vLLM …). Dieser Weg wurde vom Projekt **nicht getestet**: Kleine lokale Modelle gehen oft schlecht mit Werkzeugaufrufen und der Regel „dieser Text sind Daten“ um. Prüfe Antworten besonders sorgfältig.

**Was geprüft wurde und was nicht.** Die Sicherheitsprüfung (A5) umfasste, was der Assistent erreichen kann, Text von der Box, die Regel „wendet nie an“, Last- und Kostengrenzen, Fehlertexte und den Web-Zugang – gegen die Mock-Box und geskriptete Modelle. Nicht geprüft: ein echter Anbieter oder echtes Modell, eine echte Box.

### KI-Zentrale: Eingang
<!-- help: ai-inbox -->

**KI-Zentrale → Eingang** sammelt alles, was auf deine Bestätigung wartet – Vorschläge des eingebauten Assistenten, eines externen Assistenten oder einer Automatisierungsregel. Ein Filter nach Herkunft (mit Zahlen) und eine Herkunftszeile zeigen, von wem ein Vorschlag stammt; **Prüfen** zeigt die aktuelle Vorschau an Ort und Stelle, erst **Anwenden** (bei Bedarf mit PIN) ändert etwas. Die Zahl der wartenden Vorschläge steht am Menüpunkt und am Tab. Vorschläge laufen nach einiger Zeit ab; die Karte sagt es. Eine vorgeschlagene Regel entsteht erst, wenn du sie hier bestätigst – mit Satz und Probelauf.

### Senderliste (Bouquets)
<!-- help: channel-list -->

Deine Bouquets, auf dem Smartphone bearbeitbar (Admins). Bei mehr als einer Box gleicht **Von einer anderen Box kopieren** (unter den Bouquets) die Bouquets dieser Box an die einer anderen an: gleichnamige Bouquets werden angeglichen, fehlende angelegt, alles andere bleibt; Sender, die diese Box nicht kennt, werden aufgelistet und ausgelassen. Verschiebe Sender (Schaltflächen oder Ziehen am Computer), setze einen Sender auf eine bestimmte Nummer, benenne um, entferne, füge aus allen der Box bekannten Sendern hinzu, füge Überschriften ein; erstelle, benenne um und lösche Bouquets. **Alle Sender** durchsucht alles, was die Box kennt (neue Sender, Sender in keinem Bouquet, HD, nach Anbieter). **Logos** listet Sender ohne Logo auf sowie Dinge, die einen Blick wert sind (Sender, die nach einem Suchlauf weg sind, doppelt in einem Bouquet, ohne Programmdaten), zeigt die installierten Logo-Pakete und lässt dich das von DreamHub genutzte Paket auswählen – und für einen einzelnen Sender ein abweichendes Logo: aus jedem installierten Paket, ein eigenes Bild oder eines aus der öffentlichen TV-Logos-Sammlung (auch aus der Liste „Ohne Logo“ der Prüfung). Diese Auswahl ändert nur die DreamHub-Bildschirme, niemals den Fernseher, und **Zurück zu Automatisch** macht es rückgängig. Die Liste liest immer die aktuelle Senderliste der Box; sie sucht nicht nach neuen Sendern – führe dafür einen Sendersuchlauf auf der Box durch. Jede Änderung wird gesammelt und als eine Vorschau angezeigt („ZDF HD wechselt von 14 auf 2“); beim Anwenden wird zuerst ein Backup der gesamten Senderliste erstellt, das automatisch wiederhergestellt wird, falls das Ergebnis nicht der Vorschau entspricht. Ältere Backups können wiederhergestellt werden. Benötigt das BouquetEditor-Plugin auf der Box.

### IPTV
<!-- help: iptv -->

Internet-Sender an einem Ort: **Medien → IPTV**. (**Live-TV** sind die eigenen Sender der Box; IPTV kommt aus dem Internet oder von anderen Geräten im Netz.) Es ist **eine Seite ohne Reiter**: links eine Spalte mit **Meine** (zuletzt gesehen, Favoriten und deine Ordner) und **Quellen** (am Handy aufklappbar über der Liste), rechts die Sender. Quellen hinzufügen und ändern sowie Bouquets machen ist für alle, die die Senderliste verwalten; stöbern kann jeder mit IPTV, zum Abspielen braucht es die Fernbedienungs-Berechtigung.

**Stöbern.** Die Chips **Alle · TV · Radio** grenzen ein, was gezeigt wird. Ein Tipp auf einen Sender **spielt ihn sofort auf dem TV** (er muss in keinem Bouquet sein; ein Internet-Stream hat keinen Tuner, deshalb gibt es kein **Aufnehmen**). Alles andere steht unter **⋯**: **Details…**, **In Ordner legen…**, **Bouquet erstellen…**, **Diesen Sender ausblenden**, **Die Gruppe ausblenden**. Der Umschalter über der Liste hat **Kacheln** (Standard; so viele nebeneinander, wie auf den Bildschirm passen), **Liste** und **Programm**; die Wahl wird auf diesem Gerät gemerkt. Radio-Kacheln zeigen das Logo des Senders und, wo der Stream es sendet, den laufenden Titel. **Programm** ist das Zeitraster eines Ordners oder einer Quelle bzw. Gruppe mit bis zu etwa 200 Sendern (bei einer größeren wirst du gebeten, die Sender, die du schaust, in einen Ordner zu legen). Eine Gruppe einer Quelle (Filme, Nachrichten…) ist ein Chip über der Liste. Derselbe Sender aus mehreren Quellen erscheint in **Alle**, in Ordnern und in der Suche nur einmal, mit „aus 3 Quellen“; innerhalb einer Quelle steht er so, wie sie ihn hat. Die **A–Z-Leiste** am rechten Rand springt zu einem Buchstaben (mit dem Finger entlangwischen zum Scrubben). **Auswählen** macht aus Tipps Haken, mit einer Leiste, um die markierten Sender in einen Ordner zu legen, ein Bouquet daraus zu machen oder den Assistenten darauf schauen zu lassen. Das Suchfeld (Taste **/**) sucht in allen Quellen. Tasten: Pfeiltasten bewegen, **Enter** spielt, **i** öffnet die Details, **Esc** schließt ein Blatt oder Menü.

**Favoriten und Ordner.** Der Stern an Kachel oder Zeile markiert einen Favoriten — nur deiner, pro Profil. Favoriten liegen in **Ordnern** (ein Sender in einem Ordner); einsortieren mit **⋯ → In Ordner legen…** oder per Ziehen auf einen Ordner in der Spalte; Ordner lassen sich umbenennen, verschieben und löschen (die Sender bleiben Favoriten). **Reihenfolge ändern** in der Kopfzeile eines Ordners ordnet seine Sender — ziehen, mit den Pfeiltasten, **A–Z** oder **Nach Senderplatz** — und **Fertig** beendet es. Ein Favorit, dessen Sender die Quelle verlassen hat, bleibt ausgegraut „Nicht mehr verfügbar“ — mit dem gleichnamigen Sender einer anderen Quelle als Ersatz, falls vorhanden. Grenzen: 100 Ordner, 5.000 Favoriten und 5.000 ausgeblendete Einträge pro Profil. **Ausgeblendet** am Fuß der Spalte führt zu ausgeblendeten Sendern und Gruppen zurück.

**Das Senderblatt.** **Details…** (oder **i**) öffnet das Blatt eines Senders mit **Auf dem TV abspielen**, Stern und Ordner und zwei Teilen. **Programm**: das heutige Programm mit Wochenleiste; **Guide: … · ändern** zeigt, welchen Guide der Sender nutzt, und lässt einen Administrator von Hand einen anderen wählen (pro Box gemerkt); der Guide wird einen Tag zurück und eine Woche voraus geladen und alle paar Stunden neu („aktualisiert vor …“, **Neu laden**). **Eigenschaften**: Name und Gruppe; das **Picon** des Senders selbst (**Picon wählen…**: zuerst eine **Online-Suche**, dann die installierten Sätze oder eine eigene Datei; es ist DreamHubs eigene Wahl, erscheint auf den Kacheln und in einem aus dem Sender gemachten Bouquet, **Automatisch** nimmt es zurück); der Guide (der Zwilling eines Satelliten-Senders oder der eigene der Quelle); die **Streams** des Senders, einer pro Quelle, jeder mit **Testen** und **Diesen verwenden** (das ändert, was ein Bouquet-Eintrag abspielt, nach einer Vorschau); die Bouquets, in denen der Sender ist, mit **Umbenennen…** und **Logo…** für den Eintrag; und **Diesen Sender ausblenden**. Dasselbe Blatt öffnet sich aus der Liste **Sender finden** der Senderliste, bei den Internet-Einträgen deiner Bouquets (Filter **IPTV · Radio · TV**).

**Hier ansehen.** Ein Sender, dessen Stream eine http-Adresse ist, hat im Blatt und im ⋯-Menü **Hier ansehen** (nicht bei rtsp-, udp- oder `.ts`-Adressen). Er läuft im Browser, wenn der Stream keine Umwandlung braucht: **HLS**, eine Videodatei (MP4, WebM) oder ein Audiostream. DreamHub öffnet den Stream zuerst kurz, um zu sehen, was er liefert, und reicht ihn dann an den Player weiter — die Adresse und eine Anmeldung darin bleiben auf der Box, und nichts wird umgewandelt. Kann der Browser ihn nicht abspielen, sagt das Feld warum — ein MPEG-TS-Sendestream, ein geschützter Stream, eine Senderliste statt eines Streams, keine Antwort — und bietet **Auf dem TV abspielen** an. Live-TV wird nie per Software umgewandelt. Bild und Ton hängen auch von den Formaten deines Browsers ab; einen Stream, den er nicht kennt, meldet das Feld als hier nicht abspielbar. Untertitel kann ein Live-Stream nicht mitbringen.

**Quellen.** Eine Quelle ist der Ort, aus dem eine Senderliste kommt. **+ Quelle hinzufügen** fragt, was du hast: **Eine Liste** (eine M3U/M3U8-Adresse, eine Datei von diesem Gerät oder eine Datei in einem öffentlichen GitHub-Repository; auf Wunsch **auch zu Netcast hinzufügen**, das schreibt nach einer Vorschau die Datei von Netcast auf der Box), **Ein Konto** (Host, Benutzername und Passwort eines Xtream-Kontos, nur Live-Sender), **Ein Gerät in meinem Netz** (Tvheadend, HDHomeRun — die Adresse wird aus dem Rechnernamen gefüllt — oder ein SAT>IP-Server: die angekündigte Listenadresse einfügen) oder **Etwas aus Netcast** (eine Quelle, die Netcast kennt, hier eingeschaltet). Die Liste wird einmal gelesen, bevor etwas gespeichert wird; ein falsches Login oder eine Nicht-Liste wird gemeldet und nichts gespeichert. Eine **Datei** behält DreamHub als eigene Quelle: Sie hat keine Adresse und wird nicht neu gelesen; eine neuere Datei ersetzt sie unter „Quellen verwalten“. Ein Login bleibt auf DreamHub: Es wird nie wieder angezeigt, nie an den Browser geschickt und steht nicht in der Sicherung; die Quelle sagt nur „mit Login“. Jede Quelle in der Spalte hat einen Statuspunkt, ihre Senderzahl und ein **⋯**-Menü: **Aktualisieren**, **Programm…** und **Einstellungen…**; **Netcast** ist eine Zeile, die sich in ihre Quellen auffaltet. Eine Quelle, die nicht gelesen werden konnte, wird über der Liste gemeldet, mit **Erneut versuchen** und **Einstellungen**. **Quellen verwalten** ist der Ort, an dem Quellen eingerichtet werden: die Stelle in der Spalte (↑ ↓), ob sie dort gezeigt wird (aus blendet sie überall aus und löscht nichts; schon gemachte Bouquets funktionieren weiter), welche Profile sie sehen, Adresse oder Login einer selbst hinzugefügten Quelle, **Quelle entfernen…** (fragt vorher und listet, was verschwindet) und — bei einer Quelle in Netcasts Datei — ihre Adresse und Entfernung, jeweils als Vorschau der Änderung an dieser Datei. **Quellen & Programm** (die Schaltfläche über der Liste) zeigt, ob die Quellen funktionieren, wie viel einer Quelle ein Programm hat, die Guide-Dateien (mit **Aktualisieren** und **+ Guide-Datei hinzufügen**: die vorgeschlagene Liste, eine Adresse oder ein öffentliches GitHub-Repository, auf Wunsch auch in Netcasts Datei) und, falls vorhanden, die nach Adresse gemerkten Listen aus der Zeit, bevor Bouquets verknüpft blieben.

**In ein Bouquet.** **Bouquet erstellen…** gibt es an einem Sender (⋯), an den markierten Sendern (Auswahl-Leiste), an einem Ordner und an einer ganzen Quelle oder einer ihrer Gruppen. Ein Fenster zeigt, welche Sender hineinkommen, lässt ein neues oder ein vorhandenes Bouquet wählen, testet die Streams (Haken weg bei denen, die nicht antworten), gibt einem Sender den Guide eines eigenen Senders der Box und zeigt genau, was sich ändert, bevor es auf deine Bestätigung wartet, wie überall. Ein Bouquet aus einem Ordner, einer Quelle oder einer Gruppe **bleibt verknüpft**: die Kopfzeile sagt „abgeglichen“ oder „2 neu, 1 entfernt“ mit **Angleichen…** (wieder mit Vorschau; es kann das Bouquet auch in die Reihenfolge des Ordners bringen) und **Verknüpfung vergessen**, das das Bouquet lässt, wie es ist. Favoriten, Ordner und Ausblenden sind nur DreamHubs eigene Notizen für dein Profil und ändern nichts an der Box.

**Den Assistenten schauen lassen.** In der Auswahl-Leiste schaut **Den Assistenten schauen lassen…** sich die markierten Sender an (bis zu 60 auf einmal) und zeigt zu jedem, ob er ein **Programm ✓ oder ✗** hat (gefunden in einer geladenen Guide-Datei, auf der Box oder über einen gemerkten Zwilling), den Guide **welches Satellitensenders** er zeigen könnte (ein Guide-Zwilling; er wird nur angeboten, wenn die Namen gleich sind, du kannst einen anderen Kandidaten wählen oder es lassen) und für einen Sender ohne **Picon** eins aus der öffentlichen TV-Logo-Sammlung. Der beste Vorschlag ist schon gewählt; stelle eine Liste auf *So lassen*, um ihn zu überspringen. Das Anschauen speichert nichts: **Vorschau** macht aus deiner Auswahl einen Plan, und erst **Speichern** nach dem Lesen behält sie. Zwillinge und Picons sind DreamHubs eigene Notizen: Kein Bouquet und nichts auf der Box ändert sich; das nächste **Bouquet erstellen…** nutzt die Zwillinge. Ein externer Assistent (MCP) oder der eingebaute kann denselben Plan *vorschlagen* (**Guide-Zwillinge und Picons für Internet-Sender**); er wartet dann in KI-Zentrale → Vorschläge darauf, dass du ihn prüfst und anwendest.

**Programm prüfen.** Neben der Kanalzahl schaut **Programm prüfen** auf die Kanäle, die die Liste zeigt (bis zu 200 auf einmal), und setzt ein kleines **✓ oder ✗** auf jede Kachel und Zeile, dazu eine Zählung („3 von 5 Kanälen haben ein Programm · 2 ohne“) und *Nur die ohne Programm*, um die Lücken zusammenzuholen; gespeichert wird nichts. Ein Kanal zählt als ✓, wenn eine geladene Programmdatei, ein gemerkter Programm-Zwilling, die Box oder – bei einem Internet-Konto (Xtream) – das Konto selbst ein Programm dafür hat. Ein Konto wird Kanal für Kanal gefragt, 40 Kanäle pro Prüfung; die übrigen zeigen **?** – prüf noch einmal. Dieselbe Zählung nutzt *Den Assistenten schauen lassen*.

**Jugendgefährdende Sender** bleiben standardmäßig draußen: Ein Profil ohne die Berechtigung *See adult channels in IPTV* (in der Oberfläche in deiner Sprache) bekommt sie nie (nicht in Listen, Suche, Guide oder zum Abspielen); Administratoren haben sie, die Seite hält sie aber verborgen, bis **Erwachsenen-Gruppen anzeigen** angehakt ist. Gruppen werden am Namen erkannt, deshalb rutscht manches durch und manches Harmlose wird erfasst.

**Suche** (Feld oben, **Strg+K**) findet auch Sender deiner Quellen nach Namen; ein Treffer öffnet die Seite bei dieser Quelle. Ältere Links und Lesezeichen (`/netcast`, `/iptv?tab=guide`, `/channels?tab=iptv`) funktionieren weiter: Sie öffnen die Seite und das Blatt oder Fenster, das den Platz des Reiters übernommen hat.

**Was geprüft wurde und was nicht.** Gebaut und getestet gegen die Mock-Box, einen erfundenen Katalog mit rund 1.900 Sendern und Fake-Server. **Nicht** geprüft auf einer echten Box oder mit einem echten Anbieter: ein Xtream-Konto (die Aufrufe folgen den veröffentlichten API-Hinweisen), die Pfade für Tvheadend und HDHomeRun (aus öffentlichen Dokumentationszusammenfassungen), ob der Player der Box eine Xtream-`.ts`-Adresse abspielt, die Titel auf Radio-Kacheln bei echten Sendern, Hier ansehen mit dem Stream eines anderen Anbieters als der Netcast-Listen (auf einer Dreambox One mit HLS-Sendern der Netcast-Listen geprüft), eine Quelle aus einer Datei sowie Speicher und Ladezeit einer Woche Guide auf einer Dreambox One.

### Fernbedienung
<!-- help: remote -->

Eine Fernbedienung für die Box im Browser mit Live-Bild des TV-Bildschirms und **Nachricht an den TV**, um einen kurzen Text auf dem Bildschirm anzuzeigen. Lässt sich über das Fernbedienungssymbol oben auf jeder Seite öffnen.

### Gerät
<!-- help: box-admin -->

**Übersicht** zeigt die Box, ihre Festplatten/Speicher und das **Änderungsprotokoll** jeder über DreamHub vorgenommenen Änderung. **Diagnose** führt den Selbsttest der Box aus. **Sicherung und Wiederherstellung** der eigenen Einstellungen und des Dashboards von DreamHub befinden sich unter **Einstellungen → System** (Admins, die Docker-Apps ausführen, können auch die Konfiguration jeder App in dieses Backup einbeziehen).

**Dienste & Protokolle** (nur Admins) zeigt die **Dienste** der Box – enigma2, DreamHub, Samba und andere – mit ihrem Status; **Neustart** (oder **Start**, wenn einer gestoppt ist) zeigt zuerst an, was dadurch unterbrochen wird, z. B. eine aktuell laufende Aufnahme. Darunter befinden sich Protokolle (Logs) – DreamHub-eigene und die der Box – auf Wunsch live; **Protokolle** neben einem Dienst zeigt nur dessen Zeilen sowie Absturzprotokolle (Crashlogs).

**Erweitert** (nur Admins) ist für Personen, die sich mit der Shell auskennen: eine **Skript-Bibliothek** (Skript hochladen, mit Bestätigung ausführen) und ein **Terminal** – standardmäßig deaktiviert, erfordert das Aktivieren und erneute Eingeben des eigenen Passwortschutzes von DreamHub zum Einschalten, mit einer dauerhaften Warnung, solange es aktiv ist.

### Apps
<!-- help: apps -->

Die Seite hat drei Tabs in fester Reihenfolge – **Pakete**, **Erweiterungen**, **Docker** –, die jeweils mit einem Satz öffnen, der erklärt, was es ist. DreamHub merkt sich den zuletzt angesehenen Tab (pro Browser).

**Pakete** ist die eigene Software der Box (DreamOS): TV-Plugins, Skins und Tools. Suche oben, dann die Filter **Installiert**, **Verfügbar** und **Updates** sowie Kategorie-Chips (Erweiterungen, Skins, System-Plugins, Treiber, Senderlogos, Kernel-Module, System). Wenn Updates ausstehen, zeigt eine Leiste „3 Updates verfügbar“ mit **Alle aktualisieren** – eine Vorschau listet jedes Paket auf, das `apt` anfassen würde, und eine Bestätigung wendet sie an. Jede Zeile hat eine Aktion (Installieren, Aktualisieren oder Entfernen) und ein **⋯**-Menü mit den Details (Version, Herkunft der Beschreibung). Jede Zeile erklärt in einem Satz in deiner Sprache, was das Paket ist; wo DreamHub keinen eigenen Text hat, zeigt es die eigene Beschreibung des Pakets an, und für ein Paket ohne beides erscheint ein optionaler Eintrag **KI fragen** in **⋯** nur, wenn ein KI-Schlüssel eingerichtet ist – zwingend erforderlich ist er für nichts. Ein Paket, das von einer DreamHub-Funktion verwendet wird, zeigt „Benötigt von Aufnahmen“ (und so weiter), und das Entfernen erfordert eine vorherige Bestätigung. DreamHubs eigene Beschreibungsliste ist noch kurz; sie wächst, während der Paket-Feed der Box gelesen wird. Das Zahnrad öffnet **Paketquellen (Feeds)** – schalte sie ein/aus oder füge eine hinzu, jeweils mit Vorschau; beim Hinzufügen wird gewarnt, dass diese Feeds unsigniert sind, füge also nur vertrauenswürdige hinzu.

**Erweiterungen** sind kleine Add-ons für DreamHub selbst – zum Beispiel eines, das eine smarte Steckdose schaltet. Jede wird als Karte mit eigenem Status und Aktionen angezeigt (niemals als eigene Webseite); eine Erweiterung, die Eingaben erfordert (z. B. einen Geräteschlüssel), verfügt über eine Schaltfläche **Einstellungen**.

**Deklarative Erweiterungen: was sie dürfen.** Eine Erweiterung, die nur ein Manifest ist (eine Liste von Geräten, Werten zum Lesen und Schaltflächen), führt kein Programm aus; DreamHub liest sie und sonst nichts. Sie darf nur mit den Adressen sprechen, die sie auflistet, und nur mit Geräten in deinem Heimnetz, außer du hast *darf das Internet nutzen* angekreuzt; die Box selbst (TV-Software, DreamHubs eigene Seiten, das Terminal) erreicht sie nie. Jede Änderung, jeder Tastendruck und jedes Geheimnis läuft über die übliche Vorschau und Bestätigung; ein geändertes Manifest, das eine Adresse, eine Schaltfläche oder Internetzugriff hinzufügt, wird vor der Bestätigung hervorgehoben. Eingetragene Passwörter und Tokens liegen in der Datenbank der Box (nur für DreamHub lesbar), werden nie wieder angezeigt, verlassen die Box nie in einer Sicherung und gehen nur an die Geräte der Erweiterung. Ein *Probelauf* („Ausprobieren“) liest jeden Wert einmal mit einem einfachen GET: Ein Gerät, das schon beim bloßen Abfragen schaltet (manche Steckdosen), schaltet – probiere also nur Adressen, die du kennst. Eine *Webhook*-Adresse enthält ihr Geheimnis; behandle sie wie ein Passwort und ersetze sie (rotieren), wenn sie durchgesickert ist. Auch ein Assistent kann eine Erweiterung nur als Vorschlag anlegen, den du hier prüfst und bestätigst.

**Docker (Beta)** – optional: zusätzliche Apps in Containern, für den normalen Betrieb nicht erforderlich. *Beta:* einfache Container funktionieren; feste Ports, Ordner von deinen Platten und manche Compose-Dateien sind noch eingeschränkt, und eine abgelehnte Compose-Datei nennt den Grund. **Apps** sind Programme, die in einem eigenen Docker-Container auf der Box laufen, getrennt von der TV-Software. Wenn Docker noch nicht eingerichtet ist, weist ein Hinweis darauf hin und verweist auf die **Docker-Einstellungen** (das Zahnrad oben rechts im Tab), wo **Docker einrichten** anzeigt, was getan wird, bevor es ausgeführt wird. Eine kurze, handverlesene Liste (AdGuard Home, Pi-hole, Nextcloud, Vaultwarden, Syncthing, Home Assistant, qBittorrent) zeigt, ob die jeweilige App auf dem Prozessor, dem Arbeitsspeicher und Port deiner Box läuft, sowie eine Anmerkung für diese Box; eine Suche nach allem anderen auf Docker Hub ist ausgeblendet (ein Ergebnis kann auf Kompatibilität geprüft oder als eigener Container hinzugefügt werden). **Installieren** öffnet ein kleines Formular: den **Web-Port** (nur änderbar, wo das Image der App es zulässt – andernfalls wird darauf hingewiesen), die **Festplatte/Speicher** zum Speichern der App (nur echte Datenträger, niemals der eigene Flash-Speicher der Box) und jedes von der App benötigte Feld; **Erweiterte Optionen anzeigen** fügt jedes weitere Feld hinzu sowie freie Zeilen für **Umgebungsvariablen**, **Ports** und **Ordner** sowie ein Arbeitsspeicherlimit. Ein geheimes Feld (ein Passwort) wird während der Eingabe verborgen und nie wieder angezeigt. Anschließend wird genau angezeigt, was erstellt wird – Images, Ports, die Ordner auf der Box, die Variablen –, geprüft gegen freien Arbeitsspeicher, Speicherplatz und Ports, bevor irgendetwas läuft. Der Tab bietet zwei Ansichten: **Meine Container** (was auf der Box ist) und **App-Katalog** (was du hinzufügen kannst). In **Meine Container** hat jede Zeile das Bild der App, den Namen, einen Statuspunkt, ihren Port, einen **Autostart**-Schalter (startet sie immer, wenn die Box startet; wird sofort gespeichert) und eine Hauptschaltfläche – **Öffnen**, während sie läuft, ansonsten **Starten**. Hinter **⋯**: **Stopp**, **Protokolle**, eine **Konsole** (eine interaktive Shell innerhalb der App – dieselbe Handhabung wie beim Terminal: standardmäßig aus, Passwort-Neueingabe, dauerhafte Warnung), **Bearbeiten** (dasselbe Formular wie bei der Installation, vorausgefüllt; es zeigt an, was sich ändert und erstellt die App neu – ihre Daten bleiben erhalten), **Sichern**, **Aktualisieren** und **Entfernen** (ihre Einstellungen bleiben erhalten). Ein Badge „Update verfügbar“ und eine Benachrichtigung erscheinen, wenn eine neuere Version existiert – DreamHub aktualisiert eine App niemals von selbst. Einige Apps bestehen aus mehr als einem Container – Nextcloud bringt eine eigene Datenbank mit, die zusammen mit ihr verwaltet und darunter mit eigenen Protokollen aufgelistet wird. Apps teilen sich das Netzwerk der Box, daher ist jeder Port der Port der Box; auf der Referenzbox ist Port 53 bereits durch den eigenen DNS-Resolver der Box belegt, sodass AdGuard Home und Pi-hole dort nicht installiert werden können. Unter **Einstellungen → System → Sicherung** ist jeder App-Ordner mit seiner Größe aufgelistet: eine Datenbank ist standardmäßig enthalten, große Dateien (die von Nextcloud, Downloads) nur, wenn sie angehakt sind.

**Container hinzufügen** (über der Liste, nur Admins) ist für alles, was nicht in der Liste steht: Füge eine **Compose-Datei** (`docker-compose.yml`) ein, gib einen **Image-Namen** ein oder füge ein **Unraid-Template** ein (die XML; DreamHub bündelt keine Templates und lädt nichts herunter – es liest nur Ports, Ordner und Variablen). Du erhältst dasselbe Formular wie für die Liste, aufgebaut aus der Datei: Jedes `${NAME:-default}` darin wird zu einem Feld. Was verstanden wird: Dienste mit Image, Ports, Volumes, Environment, Command, einem Arbeitsspeicherlimit, Restart und `depends_on`; alles andere (`build`, `networks`, `secrets`, …) wird mit Begründung abgelehnt. Apps nutzen das eigene Netzwerk der Box, daher kann ein Port nicht auf einen anderen gemappt werden – die App lauscht auf ihrem eigenen Port. Ein Ordner, der als `.` oder `./name` geschrieben ist, wird im Speicher der App aufbewahrt; jeder andere Ordner muss unterhalb von `/media` liegen. **Du bist selbst verantwortlich für das, was du ausführen lässt**: DreamHub verweigert einem Container jeglichen Systemordner der Box (`/`, `/etc`, `/usr`, `/var`, `/boot`, `/proc`, `/sys`, `/dev`), die eigenen Daten von DreamHub, den Docker-Socket und die Daten anderer Apps – auch über Links – und fordert eine zusätzliche Bestätigung an, bevor es einem Container den Privileged-Modus, zusätzliche Capabilities, Geräte oder die Prozessansicht der Box gewährt; es kann die Software selbst nicht beurteilen. Von dir hinzugefügte Container werden über **⋯** wie die anderen bearbeitet und aktualisiert, außer solche, die hinzugefügt wurden, bevor die Bearbeitung existierte (diese entfernen und erneut hinzufügen).

**Updates.** Wenn etwas aktualisiert werden kann, zeigt eine Box oben auf der Seite an, wie viele Pakete und Container, mit **Alle aktualisieren**. Die Vorschau listet alles in der Reihenfolge auf, in der es ausgeführt wird: die Container nacheinander, dann die Pakete. Container kommen zuerst, da ein Update, das nicht läuft, von selbst zur vorherigen Version zurückkehrt, sodass sich bei einem Fehlschlag nichts anderes ändert; die Pakete folgen nur, wenn jeder Container in Ordnung ist. Wenn ein Schritt fehlschlägt, wird der Rest nicht gestartet, und das Ergebnis gibt an, was getan wurde, was fehlgeschlagen ist und was nicht gestartet wurde. DreamHub selbst ist kein Teil davon (ein Hinweis weist darauf hin); aktualisiere es unter **Einstellungen → System → Über DreamHub** (siehe *Aktualisieren*). Während eine Aufnahme läuft oder eine innerhalb von 15 Minuten startet, wird nichts aktualisiert: Die Box meldet „Eine Aufnahme läuft … Updates sind ab 21:15 Uhr wieder möglich.“ **Automatische Updates** sind standardmäßig ausgeschaltet: ein Schalter für Container in den Docker-Einstellungen, einer für Pakete in den Paket-Einstellungen (nur Pakete, die enigma2 nicht neustarten); „Nur aktualisieren zwischen“ beschränkt sie auf eine Tageszeit (das manuelle Drücken von Aktualisieren wird nur durch eine Aufnahme zurückgehalten). Ihre Ergebnisse erscheinen in den Benachrichtigungen. Im **⋯**-Menü eines Containers aktualisiert **Aktualisieren** diesen Container, und **Zurückrollen** (angezeigt, solange die vorherige Version aufbewahrt wird) kehrt zurück; die Docker-Einstellungen listen die aufbewahrten Versionen auf und können sie entfernen. Im Menü eines Pakets erscheint **Downgrade auf …** nur, wenn der Feed noch eine ältere Version bereitstellt. Wenn eine App immer wieder stoppt, verlinkt eine Benachrichtigung zu ihren Protokollen.

**Sicherung und Wiederherstellen von Apps.** *Einstellungen → System → Sicherung & Wiederherstellung* kann deine Apps einbeziehen: Hake die Apps (und welche ihrer Ordner) an und lade die Sicherung herunter, oder wähle **Auf der Box speichern**, um sie auf der Aufnahmefestplatte aufzubewahren, was sich für große App-Daten eignet. Der App-Teil enthält auch die Passwörter, die DreamHub für die Apps erstellt hat, bewahre die Datei also sicher auf. Stelle die Datei auf einer neuen Box wieder her; die Apps erscheinen dann unter *Meine Container* als **Apps, die du wiederherstellen kannst** mit *Erneut installieren*. Du siehst die übliche Vorschau und bestätigst sie. Eine zuvor entfernte App wird auf dieselbe Weise angeboten.

**Docker-Einstellungen** (das Zahnrad) zeigen an, ob Docker läuft, und **Docker einrichten**, wenn es nicht läuft. Sobald es läuft, enthalten sie außerdem: wo **neue** Apps ihre Daten aufbewahren (die Datenträger mit ihrem freien Speicherplatz; bereits installierte Apps bleiben dort, wo sie sind), ob neue Apps mit der Box starten, das **Arbeitsspeicherlimit pro App** (automatisch oder deine eigene Zahl für Container, die keines angeben), wie oft DreamHub **nach neueren App-Versionen sucht** und ob es dich in den Benachrichtigungen informiert (nichts wird jemals von selbst aktualisiert), sowie **Ungenutzte Images aufräumen** – alte Images bleiben auf dem Datenträger, wenn eine App entfernt oder aktualisiert wird; es zeigt an, wie viel Speicherplatz das ist, listet sie dann auf und fragt nach, bevor gelöscht wird. Jede Einstellung zeigt **Zurücksetzen** nur an, wenn sie vom Standard abweicht. Am Ende: **Docker deinstallieren**. Die Deinstallation ist eine Vorschau wie jede Änderung: Sie stoppt und entfernt die von DreamHub erstellten App-Container, schaltet Docker und den Docker-Helper von DreamHub aus, stoppt das Laden des Overlay-Kernel-Moduls beim Booten und löscht die Docker-Programmdateien. Es stellt eine Frage – **Auch App-Daten löschen?** – und behält diese standardmäßig bei, sodass die Apps nach dem erneuten Einrichten von Docker wiederkommen. Aufnahmen und die TV-Software werden niemals angerührt.

### Automatisierungen: was sie dürfen
<!-- help: automations -->

**Automatisierung** im Menü (nur Admins, nicht in der einfachen Ansicht) enthält *Regeln*: „wenn eine Aufnahme fertig ist, sag mir Bescheid“, „jede Nacht um 02:30 …“. Eine Regel ist ein kleines, lesbares Objekt – ein Auslöser, optionale Bedingungen, bis zu fünf Aktionen und Grenzen –, kein Programm. Du startest mit einer der Vorlagen-Karten (Platte fast voll, Timer-Konflikt, eine verfolgte Folge ohne Timer, ein Dienst oder eine App fällt aus, tägliche Plattenprüfung …), mit einer leeren Regel, oder ein Assistent schlägt sie vor. Eine neue Regel entsteht in drei Schritten: **Wann** (Auslöser und Bedingungen), **Dann** (was DreamHub tut, ein Name und die Grenzen) und **Prüfen**, wo die Regel als einfacher Satz neben einem Probelauf über die letzten Tage steht. Der Satz folgt immer der gespeicherten Regel. Vor der Prüfung wird nichts gespeichert. Nach deiner Bestätigung läuft sie **ganz ohne KI**.

**Du bestätigst die Regel einmal.** Das Speichern zeigt die Regel in einfachen Worten, was sie berühren kann, ihre Grenzen und einen Probelauf gegen jüngste Ereignisse. Eine Regel, die ein Assistent vorgeschlagen hat, erscheint nur als Vorschlag in **KI-Zentrale → Eingang**; sie existiert erst, wenn du sie dort öffnest und bestätigst, und wird als deine angelegt.

**Drei Stufen.**

- **Sag mir Bescheid** – beim Auslösen bekommst du eine Meldung („das würde jetzt laufen“). Sonst passiert nichts.
- **Vorschlagen** – beim Auslösen legt DreamHub einen Vorschlag in den Eingang, wie ein Assistent. Du liest die Vorschau und wendest sie an – oder nicht. Der Standard für neue Regeln.
- **Selbstständig ausführen** – beim Auslösen und wenn die Box frei ist, wendet die Regel ihre Änderungen ohne Nachfrage an. Nur diese Stufe tut das, und sie ist am stärksten abgesichert: Sie ist aus, bis du auch **Einstellungen → Zugriff → Automatisierungen selbstständig laufen lassen** einschaltest (standardmäßig aus; „Alle Automatisierungen pausieren“ daneben stoppt jede Regel sofort), und jede Regel braucht ihre eigene Bestätigung. **Keine Regel ändert Aufnahmen, Timer, Senderliste oder Einstellungen ohne dein Anwenden – außer du hast genau diese Regel für diese Stufe bestätigt.**

**Eine Regel selbstständig laufen lassen.** Öffne eine gespeicherte Regel und wähle *Selbstständig ausführen*. Du siehst den ganzen *Rahmen* auf einem Bildschirm; geändert wird erst, wenn du abhakst, dass du ihn gelesen hast, deine PIN eingibst (falls dein Profil eine hat) und den Knopf drückst: *was sie allein tun darf* (heute nur **Timer anlegen** und als automatisierbar markierte Aktionen eigener Erweiterungen; Löschen, Papierkorb, Schneiden, Senderlisten, Einstellungen und Pakete können nie automatisiert werden), *wie viel* (höchstens 24 Läufe am Tag oder dein niedrigeres Limit, 4 pro Stunde, mindestens 5 Minuten Abstand, höchstens 10 Änderungen pro Lauf) und *was sie stoppt* (der erste Fehler pausiert sie; das Erreichen eines Limits pausiert sie; während die Box aufnimmt oder nicht erreichbar ist, bleibt ein Vorschlag; **jede Änderung** an der Regel oder an einer genutzten Funktion hebt die Bestätigung auf – die Regel fällt auf *Vorschlagen* zurück und die Glocke meldet es). Nur du kannst das – ein Assistent oder das Token einer App nie. Jeder Lauf steht im Änderungsprotokoll unter der Nummer der Regel. Selbstständiges Ausführen nutzt keine KI.

**Was eine Regel kann und nicht kann.**

- Sie kann nur nutzen, was DreamHub als lesbare Werte, Ereignisse und planbare Änderungen listet – dieselbe Liste, die ein Assistent sieht. Sie kann keine Befehle ausführen, Einstellungen ändern, Pakete installieren, die Senderliste anfassen, einen Assistenten aufrufen oder andere Regeln anlegen oder ändern.
- Sie handelt mit den Rechten des Profils, dem sie gehört, bei jedem Lauf geprüft. Verliert das Profil das Recht, stoppt die Regel.
- Jede Regel hat Grenzen: Läufe pro Tag und Änderungen pro Lauf. Eine Regel, die dreimal hintereinander scheitert (selbstständig: einmal), mehr plant als angegeben oder nicht mehr zu DreamHub passt, pausiert sich und sagt es in der Glocke.
- Text von der Box (Programm- oder Aufnahmenamen) kann einen Platz wie eine Meldung füllen, entscheidet aber nie, *was* die Regel tut.
- Zeit-Auslöser nutzen deine Zeitzone; war DreamHub zur Zeit aus, entfällt der Lauf.

**Pausieren, zurückstufen, löschen.** Jede Regel hat einen Pausenschalter, lässt sich jederzeit auf *Sag mir Bescheid* zurücksetzen und löschen. Jeder Lauf steht mit Ergebnis auf der Seite der Regel.

**Was geprüft wurde und was nicht.** Die Sicherheitsprüfung (R7, und R4 für die Stufe *Selbstständig ausführen*) lief gegen die Mock-Box und DreamHubs Tests. **Keine Regel wurde auf einer echten Box ausgeführt.**

---

## 4. DreamHub als App
<!-- help: app -->

DreamHub kann auf dem Startbildschirm deines Smartphones oder dem Desktop deines Computers abgelegt werden und sich in einem eigenen Fenster öffnen, ohne die Adressleiste des Browsers. **Einstellungen → Allgemein → DreamHub als App** zeigt dir, was in dem von dir verwendeten Browser funktioniert, inklusive einer **Installieren**-Schaltfläche, sofern der Browser eine anbietet.

* **iPhone / iPad:** Tippe in Safari auf **Teilen → Zum Home-Bildschirm**. Dies funktioniert über einfaches `http://`, ohne weitere Einrichtung.
* **Android, Windows, Mac, Linux (Chrome, Edge):** Browser bieten die Installation nur über **HTTPS** an, und DreamHub auf der Box ist einfaches `http://` (für eine LAN-Adresse kann kein Zertifikat existieren, dem ein Smartphone vertrauen würde). Solange es nicht hinter HTTPS liegt, nutze im Browser-Menü **Zum Startbildschirm hinzufügen**: Das gibt dir ein Icon, nur ohne das eigenständige App-Fenster.

**DreamHub hinter HTTPS bereitstellen** (optional) erfordert ein zweites Gerät, das dauerhaft eingeschaltet ist, sowie einen Namen mit einem echten Zertifikat – zum Beispiel:

* **Tailscale:** Auf jedem dauerhaft laufenden Gerät in deinem Netzwerk liefert `tailscale serve --bg [http://192.168.1.50:8765](http://192.168.1.50:8765)` eine Adresse wie `https://<gerätename>.<tailnet>.ts.net`, erreichbar von deinen Geräten zu Hause und unterwegs.
* **Ein Reverse Proxy**, den du bereits betreibst (Caddy, Nginx Proxy Manager, Traefik auf einem NAS), mit einer Domain, die darauf verweist und an `http://<box-IP>:8765` weiterleitet.

Trage diesen Hostnamen in jedem Fall unter **Einstellungen → Zugriff → Netzwerk → Zusätzlich erlaubte Hostnamen** ein, da DreamHub an ihn gerichtete Anfragen andernfalls verweigert (Schutz vor DNS-Rebinding).

Über HTTPS dekodiert der Browser HD-Sender im Live-TV zudem mit der eigenen Hardware des Geräts (über einfaches `http://` verweigert er dies), sodass HD auf Smartphones flüssiger läuft. Bestätigt mit Nginx Proxy Manager. Wenn das Gerät, auf dem der Proxy läuft, ausgeschaltet ist, ist auch die HTTPS-Adresse nicht erreichbar – die eigene Adresse der Box (`http://<box-IP>:8765`) funktioniert weiterhin.

**Behalte es in deinem Heimnetzwerk.** Nutze für den Zugriff von unterwegs Tailscale (oder ein anderes VPN), anstatt den Proxy für das Internet zu öffnen. Wenn DreamHub von außen erreichbar ist, schalte zuerst die Anmeldung ein (**Einstellungen → Zugriff**): Ohne diese kann jeder, der die Adresse erreicht, die Box steuern.

Wenn die Box ausgeschaltet oder nicht erreichbar ist, öffnet sich die App trotzdem, weist darauf hin und verbindet sich von selbst, sobald die Box wieder antwortet. Nichts von der Box wird auf dem Smartphone gespeichert: Timer, Aufnahmen und der TV-Guide sind immer live.

---

## 5. Benachrichtigungen auf deinem Smartphone
<!-- help: notifications -->

*Im Browser:* Die Glocke oben bietet **Benachrichtigungen auf diesem Gerät anzeigen**. Der Browser zeigt dann eine Meldung an, wenn DreamHub in einem Hintergrund-Tab oder als installierte App geöffnet ist – dies erfordert HTTPS (oder `localhost`) und deine Erlaubnis. Bei geschlossenem DreamHub nutze ntfy unten.

DreamHub kann eine kurze Nachricht an dein Smartphone senden, wenn etwas passiert, über die kostenlose App **ntfy** (iPhone und Android; kein Konto erforderlich):

1. Installiere **ntfy** auf deinem Smartphone.
2. In DreamHub unter **Einstellungen → Benachrichtigungen**: Gib ein **Thema (Topic)** ein – einen schwer zu erratenden Namen; die Seite schlägt einen vor – und speichere.
3. Abonniere in der ntfy-App dasselbe Thema (Server `ntfy.sh`, sofern du keinen eigenen betreibst) und drücke dann in DreamHub auf **Test senden**.

Wähle aus, welche Ereignisse eine Nachricht senden: Eine Aufnahme ist beendet (ein), eine Aufnahme hat gestartet (aus), **eine fehlende Episode einer deiner Serien läuft in den nächsten 24 Stunden** (ein), ein möglicher Timer-Konflikt (ein), der Datenträger ist fast voll (ein), **ein Dienst auf der Box fällt aus oder kehrt zurück** (ein – Samba, die Softcam, enigma2 selbst; DreamHub prüft jede Minute und meldet nur Dienste, die dauerhaft ausfallen). Die Glocke oben in DreamHub zeigt all diese Ereignisse ebenfalls an, mit oder ohne Push.

Datenschutz: Der Text einer Nachricht (Titel einer Aufnahme oder Serie, Sendername) läuft über den ntfy-Server. Auf dem öffentlichen `ntfy.sh` könnte jeder, der dein Thema kennt, diesen mitlesen – deshalb sollte er schwer zu erraten sein. Für die vollständige Kontrolle betreibe deinen eigenen ntfy-Server und gib dessen Adresse ein.

---

## 6. Wenn etwas nicht funktioniert
<!-- help: troubleshooting -->

* **Nicht sicher, was auf deiner Box funktioniert?** **Gerät → Diagnose → Selbsttest ausführen** fragt die Box nach allem ab, was DreamHub benötigt, ohne etwas zu ändern, und markiert jede Antwort: in Ordnung, einen Blick wert, fehlgeschlagen oder nicht zutreffend. **Antworten der Box herunterladen** liefert eine ZIP-Datei, die beim Melden eines Problems hilft.
* **DreamHub startet nicht und im Log steht „address already in use“:** Ein anderes Programm auf der Box nutzt den Port von DreamHub (**8765**). Wähle im TV-Plugin einen freien (**DreamHub → Einstellungen → Backend-Port**; nach deiner Bestätigung startet DreamHub neu) oder im Terminal: `echo 'DREAMHUB_LISTEN_ADDR=:8800' > /etc/default/dreamhub && systemctl restart dreamhub` – danach `http://<Box>:8800` öffnen.
* **DreamHub öffnet sich nicht:** Überprüfe, ob der Dienst läuft – im WebAdmin-Terminal mit `systemctl status dreamhub`. Sein Protokoll: `journalctl -u dreamhub -n 100`.
* **Ein Hub zeigt „Nicht verfügbar“ oder „Eingeschränkt“:** Der Punkt in der Seitenleiste und die Glocke erklären den Grund und die Lösung (fehlender Schlüssel, nicht installiertes Plugin, keine Antwort der Box). **Gerät → Diagnose** listet alle auf.
* **„WebControl konnte nicht erreicht werden“:** Die Benutzeroberfläche der Box startet neu oder ist beschäftigt (nach einem Update benötigt enigma2 einen Moment) oder verlangt jetzt eine Anmeldung – gib diese unter **Einstellungen → Dienste → WebControl** ein.
* **Datei-Aktionen fehlen unter Aufnahmen:** Sie erfordern, dass DreamHub direkt auf der Box läuft; die Seite weist darauf hin, wenn sie nicht verfügbar sind.
* **Keine Bilder:** Füge einen TMDB-Schlüssel hinzu und stelle sicher, dass *Bilder, Bewertungen und Übersichten laden* aktiviert ist (Einstellungen → Dienste → TMDB).
* **Mehr Details für einen Fehlerbericht:** **Gerät → Dienste & Protokolle → Protokolle**, stelle das Level für eine Weile auf *Debug* (es schaltet sich von selbst wieder zurück).

Probleme und Ideen: Der Issue-Tracker des Repositorys auf GitHub.
