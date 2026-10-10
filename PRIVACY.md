# Rokumio — Privacy Policy

**Last updated: October 10, 2026**

This Privacy Policy explains how the Rokumio project ("Rokumio", "we", "us")
handles information in connection with the **rokumio-client** (the Roku app) and
**rokumio-service** (the Android companion app), together the "Apps".

Rokumio is an open-source project. This policy is written to describe what the code
actually does. Where an app stores something locally or sends something over the
network, it is described below.

**Short version:** Rokumio has **no analytics, telemetry, advertising, or crash
reporting**, and no Rokumio account or server. Guest use is fully local. If you
choose to log in with a Stremio account, some data is exchanged directly with
Stremio to sync your library and addons. The Android companion talks only to your
Roku and the server running on your own phone.

---

## 1. Who is responsible

Rokumio is an independent, community-run open-source project. We do not operate any
backend servers that collect personal data. There is no controller-side database of
users. Questions can be raised through the project's GitHub repositories
([client](https://github.com/gpratoe/rokumio-client),
[service](https://github.com/gpratoe/rokumio-service)).

## 2. Summary of what Rokumio does not do

- **No analytics or telemetry.** Neither app contains analytics, tracking, or
  telemetry SDKs. Neither app sends usage statistics anywhere.
- **No advertising.** There are no ad SDKs and no ad identifiers.
- **No crash reporting.** No crash/error reporting service is integrated.
- **No Rokumio accounts.** You do not create an account with Rokumio, and Rokumio
  does not receive or store your identity.
- **No device identifiers transmitted.** Device information used by the apps is
  read locally only and is not sent to Rokumio.
- **No data sold or shared** for marketing or any commercial purpose.
- **Not directed at children.** Rokumio is not directed at children and does not
  knowingly collect personal information from anyone, including children.

## 3. The Roku client (rokumio-client)

### 3.1 Data stored on your Roku

The client stores its data **locally on the Roku device**, in Roku's registry
(per-device, private to the channel). No local data is uploaded to Rokumio.

| Stored item | Contents | When |
|---|---|---|
| `auth` → `session` | Your Stremio session token (`authKey`) and basic profile (`_id`, `email`, `avatar`) | Only if you log in with a Stremio account |
| `settings` | Streaming-server address and UI language | When you set them |
| `addons` | Your installed addon records, their order, and any removed built-ins | When you install/remove addons |
| `library` | Your saved items, watched state and Continue Watching list | As you browse and watch |

- In **guest mode** (no login), everything above except `auth` stays entirely on
  the device and nothing is synced.
- If you **log in with a Stremio account**, the client stores your session token
  locally and syncs addon/library data with Stremio (see §5).

### 3.2 Network connections the client makes

The client connects directly to:

- **The addons you install**, at the manifest URLs you supply, to fetch catalogs,
  metadata, streams and subtitles. This includes the two built-in Stremio services
  — **Cinemeta** (`v3-cinemeta.strem.io`) and **OpenSubtitles v3**
  (`opensubtitles-v3.strem.io`) — and any addons you add yourself.
- **The streaming server you configure** (for example, a `rokumio-service` instance
  on your phone or a `stremio-service` instance on a desktop).
- **Stremio servers**, only if you choose to log in:
  `link.stremio.com` (link-code sign-in) and `api.strem.io` (profile, addon and
  library sync, logout).
- **A Twemoji CDN** (`cdn.jsdelivr.net`) for emoji glyphs used in titles.
- **Artwork and images** hosted at whatever URLs the addons return.

The client also runs a small **local setup web page** (on a LAN port, default
`8744`) so you can import an addon URL from a phone's browser, and it receives
commands from the Roku OS over ECP (port `8060`). These are local-network features;
they do not send data to Rokumio.

### 3.3 Data sent to Stremio (only if you log in)

Logging in is **optional**. If you link a Stremio account, the client talks
directly to Stremio to:

- authenticate via a link code and read your basic profile (`_id`, `email`,
  `avatar`);
- retrieve and update your **addon collection**;
- retrieve and update **library items** (saved titles, watch state and position).

That exchange is between you (your device) and Stremio, and is governed by
Stremio's own terms and privacy policy. Rokumio does not receive a copy. If you do
not log in, none of this happens.

## 4. The Android companion (rokumio-service)

### 4.1 Permissions and why they are needed

| Permission | Purpose |
|---|---|
| `INTERNET` | HTTP to your Roku (port `8060`) and localhost probes of the server |
| `CHANGE_WIFI_MULTICAST_STATE` | SSDP multicast discovery of Rokus on your LAN |
| `ACCESS_NETWORK_STATE` | Network-state checks around discovery/probing |
| `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_DATA_SYNC` | Run the streaming server reliably in the background |
| `WAKE_LOCK` | Keep the CPU serving with the screen off |
| `POST_NOTIFICATIONS` | Show the "server running" notification with a Stop action |

The app requests **no** storage, location, camera, microphone, contacts, or
Bluetooth permissions.

### 4.2 Data stored on your phone

In the app's private storage, the app keeps:

- **Preferences** (`SharedPreferences` file `rokumio`): the list of addon manifest
  URLs you've added, a dev-only channel-id override, and whether the notification
  prompt was shown. **The last connected Roku is kept in memory only and is never
  persisted** — a fresh launch starts disconnected.
- **The `server.js` file you import**, plus `preload.js`, a seeded
  `server-settings.json`, the streaming cache directory, and a `node.log`.

### 4.3 Network behavior

The companion app itself makes **no connections to the internet**. Its only outbound
requests are to your **local network**: your Roku (ECP/DIAL) and the streaming
server running in loopback on the phone. Your addon list and server address are
delivered **only to your Roku** over the LAN. The imported `server.js` is never
uploaded anywhere.

The app contains donation links that, when tapped, open your browser; the app does
not fetch them itself and any resulting visit is governed by the destination site.

### 4.4 The user-supplied `server.js`

The app can only run a Stremio-compatible streaming server, and the `server.js`
that powers it is **proprietary Stremio code that you supply at runtime**. It is
not bundled, provided, or distributed by Rokumio. That server runs as a **separate
process** and may make its own internet connections (for example, to addons, peers,
or torrent trackers) depending on how it is configured and what you do with it.
That behavior is inside the server software you chose to run and is **not
performed or controlled by the Rokumio app**. You are responsible for the software
you run and what it connects to.

## 5. Third parties

Rokumio does not share data with third parties. However, by using addons, a
streaming server, or Stremio login, your device communicates **directly** with
services that are independent of Rokumio. Those services have their own privacy
practices, which we do not control, including:

- **Stremio** (`strem.io`) — when you log in or use its addons.
- **Any addon provider** whose manifest URL you install.
- **Any streaming server** you configure.
- **Content hosts / images** referenced by addon metadata.

Rokumio is not affiliated with, endorsed by, or associated with Roku, Inc., Stremio,
or the Node.js Foundation.

## 6. Security

All personal data the apps handle is stored locally on your own devices. Sensitive
values such as a Stremio session token are kept in the app's private storage
(Roku registry or the Android app's private files). Local-network traffic uses
cleartext HTTP, which is standard for Roku ECP and is scoped to your LAN. No method
of storage or transmission is perfectly secure, and you use the Apps at your own
risk.

## 7. Your choices and control

- **Don't log in** to keep everything local (guest mode).
- **Log out** in Settings to clear the stored Stremio session from the device.
- **Remove addons** at any time.
- **Uninstall the Roku channel or Android app** to remove the associated local
  data.
- **Delete your Stremio account data** through Stremio, since library/addon sync
  lives in your Stremio account, not with Rokumio.

## 8. Changes to this policy

This policy may be updated as the project evolves. Material changes will be made in
this file in the project repository, with an updated "Last updated" date. Continued
use after a change means you accept the revised policy.

## 9. Contact

For privacy questions, open an issue in the project repositories:
[rokumio-client](https://github.com/gpratoe/rokumio-client/issues) or
[rokumio-service](https://github.com/gpratoe/rokumio-service/issues).

---

*This document describes the behavior of the Rokumio project's software. It is
provided for transparency and is not legal advice.*
