# Third-party software and data

DreamHub is built on free software. Thanks to everyone behind it. The packages carry the following; each project's own licence text applies to it.

## In the program (Go)

| Module | Version | Licence |
| --- | --- | --- |
| github.com/dustin/go-humanize | v1.0.1 | MIT |
| github.com/google/uuid | v1.6.0 | BSD-style |
| github.com/remyoudompheng/bigfft | v0.0.0-20230129092748-24d4a6f8daec | BSD-style |
| github.com/skip2/go-qrcode | v0.0.0-20200617195104-da1b6568686e | MIT |
| github.com/ulikunitz/xz | v0.5.17 | BSD-style |
| go.yaml.in/yaml/v3 | v3.0.5 | Apache-2.0 |
| golang.org/x/crypto | v0.58.0 | BSD-style |
| golang.org/x/sys | v0.49.0 | BSD-style |
| modernc.org/libc | v1.77.1 | BSD-style |
| modernc.org/mathutil | v1.7.1 | BSD-style |
| modernc.org/memory | v1.12.1 | BSD-style |
| modernc.org/sqlite | v1.60.1 | BSD-style |

## In the web interface (bundled; their own dependencies come with them)

| Package | Version | Licence |
| --- | --- | --- |
| @fontsource/source-code-pro | 5.3.1 | OFL-1.1 |
| @fontsource/source-sans-3 | 5.3.1 | OFL-1.1 |
| hls.js | 1.6.19 | Apache-2.0 |
| pinia | 4.1.0 | MIT |
| qrcode | 1.5.4 | MIT |
| vue | 3.5.44 | MIT |
| vue-i18n | 11.4.14 | MIT |
| vue-router | 5.4.0 | MIT |

The fonts *Source Sans 3* and *Source Code Pro* (Adobe) are used under the SIL Open Font License 1.1; they are bundled, DreamHub never loads fonts from the internet.

## Data and services you can switch on

DreamHub talks to these only when you enter your own key or switch them on, and shows their data with credit:

- **TMDB** (themoviedb.org): *This product uses the TMDB API but is not endorsed or certified by TMDB.* Covers, descriptions and ratings.
- **fanart.tv**: logos for titles.
- **TVmaze**, **OMDb**: episode and rating data.
- **tv-logo/tv-logos** (GitHub): the public collection of channel logos the picon search uses.
- **Rytec** and other XMLTV guide sources you add yourself.

Dreambox, DreamOS, VU+, Plex, Emby, Sonoff and the other names belong to their owners; DreamHub is an independent project and not affiliated with or endorsed by them.
