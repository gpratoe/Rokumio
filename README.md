<div align="center">

# Rokumio

### Stremio's addon ecosystem, on your Roku.

<p>
  <img src="https://img.shields.io/badge/platform-Roku%20%2B%20Android-662D91?style=flat-square" alt="Roku and Android" />
  <img src="https://img.shields.io/badge/runtime-SceneGraph%20%C2%B7%20Kotlin-8B5CF6?style=flat-square" alt="SceneGraph and Kotlin" />
  <a href="https://github.com/gpratoe/rokumio-client/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-22C55E?style=flat-square" alt="MIT License" /></a>
</p>

#### Rokumio bridges [Stremio](https://www.stremio.com/) and Roku. A native 10-foot Roku client for Stremio addons, plus an Android companion that runs a Stremio-compatible streaming server and pushes your setup to the TV.

</div>

---

## What Rokumio is

Rokumio is a free, open-source project that lets a Roku browse and play media
through **Stremio addons**. It has two halves that work together:

| Repository | Role | Stack |
|---|---|---|
| **[rokumio-client](https://github.com/gpratoe/rokumio-client)** | The Roku app. Browse catalogs, search, view details, pick streams, and play — remote-friendly, built for the living room. | BrightScript / SceneGraph (BrighterScript) |
| **[rokumio-service](https://github.com/gpratoe/rokumio-service)** | The Android companion. Runs a Stremio-compatible streaming server on your phone and pushes your server address + addons to the Roku over the LAN. | Kotlin (Android), bundled Node.js + ffmpeg |

Rokumio is a **client and a launcher** — it does not host, index, or distribute
media. Content and streams come from the addons you install and from a streaming
server you point it at. See [What Rokumio is not](#what-rokumio-is-not).

## Architecture at a glance

There are two independent flows: the **media/playback path**, and a **setup/control
path** used only by the Android companion.

### Media / playback path

```
                  catalogs · metadata · stream URLs (HTTP)
      ┌───────────────┐ ◀──────────────────────────────────────────┐
      │   ADDONS      │                                            │
      │  Cinemeta     │                                            │
      │  OpenSubtitles|                                            │
      │  + your own   │                                            │
      └───────────────┘                                            │
                                                                   │
      ╔═══════════════════════════════╗                            │
      ║   Roku TV — rokumio-client    ║ ◀──────────────────────────┘
      ║   Home · Search · Discover    ║
      ║   Details · Streams · Player  ║
      ╚════════╤══════════════════════╝
               │ ▲
   torrent     │ │   HLS (H.264 / AAC)
   requested   │ │
   (infoHash,  │ │
    fileIdx)   ▼ │
      ╔═════════════════════════════════════════╗
      ║   Streaming server (you configure)      ║
      ║   rokumio-service on a phone, or        ║
      ║   stremio-service on a desktop          ║
      ║   runs server.js + Node + ffmpeg        ║
      ╚════════════════════╤════════════════════╝
                           ▼
                    peer swarm / trackers
```

**In words:** the Roku client asks the addons for catalogs, metadata and stream
URLs (plain HTTP). When a stream is a torrent, the client does **not** download it
itself — it **requests that torrent from the streaming server** by `infoHash` and
`fileIdx`, and the server runs the torrent engine and transcodes on the fly,
**streaming HLS (H.264/AAC) back to the client** to play. The client sends torrents
to the server and receives HLS from it. (Streams that are already a direct, 
Roku-playable URL may be played by the client without the server.)

### Setup / control path (Android companion)

```
   ┌──────────────────────────────┐   ECP / DIAL over your LAN   ┌─────────────────────┐
   │  Android — rokumio-service   │ ───────────────────────────▶ │   Roku client       │
   │  runs the streaming server,  │  pushes installed addon URLs │   applies addons +  │
   │  manages addons, finds Rokus │  + the server's LAN address  │   server address    │
   └──────────────────────────────┘                              └─────────────────────┘
```

**In words:** the Android app is an optional convenience. It hosts the streaming
server on your phone (the same server the Roku talks to in the playback path above)
and pushes your configuration to the Roku over ECP (Roku's External Control
Protocol), so no desktop machine is needed.

### Part 1 — The Roku client

A BrighterScript/SceneGraph app packaged with `bsc` into a sideloadable
`dist/rokumio-client.zip`. Highlights:

- **Home** — Continue Watching plus the catalog rows you configure.
- **Search** — text search across movies and series via your addons.
- **Discover** — browse by chart, type and genre, with infinite scroll.
- **Details & Streams** — full metadata and a picker for available streams.
- **Addons manager** — install/uninstall Stremio addons from a manifest URL.
- **Settings** — streaming-server address, UI language, and (optional) Stremio login.
- **Stremio account (optional)** — link-code login to sync your addon collection
  and library with your Stremio account; guest mode keeps everything local.

Key stores live under `source/stores/` (`Transport`, `Addons`, `Catalog`,
`Episodes`, `Playback`, `Subtitles`, `Library`, `Auth`, `Settings`, `DeepLink`),
and screens under `components/`.

Build and test:

```bash
npm install     # build/test tooling (BrighterScript + brs interpreter)
npm run build   # compile + package -> dist/rokumio-client.zip
npm test        # brs unit tests + mock addon server integration tests
```

See the [client README](https://github.com/gpratoe/rokumio-client) for sideloading
instructions.

### Part 2 — The Android service

`rokumio-service` turns an Android phone into the streaming server behind Rokumio.
It bundles the runtime (a standalone Node.js 26 executable and a cross-compiled
ffmpeg/ffprobe) but **not** `server.js` — that is proprietary Stremio code and a
user-supplied input you import at runtime. The app stands the server up, keeps it
alive as a foreground service with a partial wake lock, and exposes it to the Roku
on the local network.

It also discovers Rokus on the LAN (SSDP, `ST: roku:ecp`, port 8060) and pushes your
configuration — installed addon manifest URLs and the running server's address —
to the Roku channel over **ECP/DIAL** using a small strict-JSON payload:

```json
{ "schema": 1, "addons": ["https://..."], "settings": { "serverAddress": "http://ip:11470" } }
```

Build (Gradle 8 + JDK 17):

```bash
scripts/fetch-node-android.sh     # stage the Node executable
scripts/build-ffmpeg-android.sh   # cross-compile ffmpeg/ffprobe for Android
gradle :app:assembleDebug         # -> app/build/outputs/apk/debug/app-debug.apk
```

> The app is **arm64-only** and targets Android 8+. There is no automated test
> suite yet; it is exercised on-device.

See the [service README](https://github.com/gpratoe/rokumio-service) and
`rokumio-service/ARCHITECTURE.md` for details.

## Getting a streaming server

Rokumio does not download, cache, or transcode anything itself. It pulls playable
streams through a streaming server you configure. Two ways to get one:

1. **rokumio-service** — the Android companion, which stands one up on your phone
   (you supply the `server.js` file).
2. **[stremio-service](https://github.com/Stremio/stremio-service)** — run Stremio's
   standalone server on a desktop machine and point Rokumio at it.

## What ships by default

The client ships with Stremio's official built-ins only:

- **Cinemeta** (`com.linvo.cinemeta`) — titles, artwork and descriptions.
- **OpenSubtitles v3** (`org.stremio.opensubtitlesv3`) — subtitles.

Every other addon (content, streams, subtitles) is installed by **you** from a
manifest URL you provide. Rokumio does not recommend, rank, bundle, link to, or
distribute third-party addons.

## What Rokumio is not

- It does **not** host, index, cache, or distribute media content.
- It does **not** run a content server or streaming service of its own, and it has
  **no torrent client** — it never processes magnet links, `.torrent` files, or
  peer-to-peer swarms itself.
- It does **not** provide, operate, or control the streaming server or the addons
  you connect; those are supplied and run by you.
- It is an **independent project**, not affiliated with, endorsed by, or associated
  with Roku, Inc., Stremio, or the Node.js Foundation. All trademarks belong to
  their respective owners.

## Privacy and terms

- **[Privacy Policy](PRIVACY.md)** — what the client and service do and don't do
  with your data (short version: no analytics or telemetry; guest mode is fully
  local; Stremio login is optional and only then does anything sync to Stremio).
- **[Terms of Use](TERMS.md)** — the rules and disclaimers for using Rokumio.

## License

MIT. See each submodule's `LICENSE` file
([client](https://github.com/gpratoe/rokumio-client/blob/main/LICENSE),
[service](https://github.com/gpratoe/rokumio-service/blob/main/LICENSE)).
Bundled runtimes (Node.js, FFmpeg) and the user-supplied `server.js` keep their own
licenses and terms; they are not covered by Rokumio's MIT license.

## Disclaimer

Rokumio is provided **"as is", without warranty of any kind**. You are responsible
for making sure the sources you connect and the content you access are lawful in
your jurisdiction. Don't use this software to infringe copyright or to circumvent
access controls.
