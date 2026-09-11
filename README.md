# MtrScope

**Continuous per-hop network probing (mtr) for Android.**

MtrScope is a lightweight, native Android implementation of the classic
[`mtr`](https://en.wikipedia.org/wiki/MTR_(software)) (My Traceroute) tool.
It continuously probes every hop between your device and a target host,
re-live-updating loss % and latency so you can watch a network path degrade or
recover in real time — a familiar workflow if you are used to `mtr` on Linux.

![Platform](https://img.shields.io/badge/platform-Android-0891B2)
![Version](https://img.shields.io/badge/version-1.8.27-green)
![API](https://img.shields.io/badge/API-26%2B-blue)

---

## Features

- **MTR / PING trace** — continuous live per-hop probing with real-time loss %,
  sent count, last / average latency, and per-hop sparklines & loss bars.
- **Multi-tab parallel trace** — trace several hosts at once (one tab each); a
  STOP stops all running tabs. Batch mode (several hosts separated by commas /
  newlines) opens a tab for each in a single tap.
- **IP geolocation + ASN** — every hop tagged with its country code and
  autonomous system (e.g. `[HK AS9429 HKT]`), toggled on/off via the ASN
  box. Reserved/private ranges are skipped (no public ASN).
- **rDNS (reverse DNS)** — resolves the hostname of each hop; toggleable live
  while tracing.
- **WHOIS** — tap a hop (or a domain/IP in the header) to look up WHOIS for
  IPs and eTLD domains.
- **Health bar + trend chart** — target-hop summary (`Loss / Avg / Jitter`),
  tap for the time-series trend chart.
- **Visual route map** — plots each *public* hop on an OpenStreetMap / Leaflet
  map and draws the **great-circle** route with red direction arrows, skipping
  private/internal hops and starting from your own public IP.
- **Your public IP (v4 + v6)** — shown in the header via a live external probe,
  cached and re-checked automatically when the route or network changes.
- **`MORE` toolbox**
  - **Net Info** — interface, IP stack (Dual-stack / v4 / v6), MTU, gateway,
    DNS servers.
  - **DNS Lookup** — raw UDP A / AAAA / CNAME / NS / MX / TXT / PTR / WHOIS / ALL,
    with history; tap a returned host/IP to start an MTR or PING on it.
  - **TCP Port Scan** — single / list / range / mixed port specs
    (`443` · `80,443,8080` · `1-1024` · `80,443,1-1024`), an **IP family selector**
    (IPv4 / IPv6 / IPv4 + IPv6, default IPv4, remembered) with each family's
    result shown separately. Before any IPv6 target is probed, the app asks an
    **IPv6-only echo service for your own public IPv6 address** — the only
    reliable proof that IPv6 really works on that network — and when nothing
    comes back it reports **"Your network does not support IPv6"** with a
    one-tap *switch to IPv4* instead of pretending every port is closed. Results
    also distinguish *open* / *refused* (host reached, port closed) / *timed out*
    (unreachable).
- **History** — search / compare / favourite / clear past traces, auto-saved
  after the first complete round (not only on STOP).
- **Export & copy** — copy the table as text / CSV / JSON, share, or export the
  table as a PNG (QR code of the result included); a `☰` diagnostic log.
- **Dark / light theme** — 3-state (system / light / dark), switchable even
  while tracing; the active host is resumed automatically.
- **Home widget** — tap the widget to enter a host, then `▶ MTR` / `▶ PING`
  runs it straight away (each widget instance remembers its own host).
- **`☰` hamburger menu** — Log, About and FAQ tucked away to keep the main UI
  clean.

> Note on probing: raw ICMP is blocked on Android, so MtrScope shells out to
> the system `ping` binary with increasing TTLs (`-t`) to reveal each hop —
> the same technique `mtr` uses. Real devices return real mid-route routers.

---

## Screenshots

*(Add screenshots of the hop list, route map, and settings here.)*

---

## Getting Started

### Prerequisites

- Android **API 26+** (Android 8.0)
- Android Studio / Gradle with JDK 17
- An Android device with internet access (geolocation + map tiles are fetched
  live)

### Build

```bash
# Local release APK
./gradlew assembleRelease
# Google Play AAB bundle
./gradlew bundleRelease
```

The release APK is produced at:

```
app/build/outputs/apk/release/app-release.apk
```

> The release build is signed. The keystore is intentionally **not** committed
> to this repository — see [Building a signed release](#building-a-signed-release).

### Install

```bash
adb install -r app/build/outputs/apk/release/app-release.apk
```

---

## How to use

1. Type a hostname or IP in the input (e.g. `8.8.8.8`, `yam.com`).
2. Tap **MTR** for a hop-by-hop trace, or **PING** to just watch the target's
   latency/loss over time.
3. While tracing, watch the live table. Toggle **rDNS** / **ASN** to show or
   hide reverse names and geo tags.
4. Tap **🗺 map** to open the route map, **🕘 history** for past traces,
   **🖒 top** to pin to the top, and **📋 copy** to copy the current table.
5. Tap a hop to WHOIS it; long-press to copy that hop.

---

## Building a signed release

This repo does **not** include the signing keystore (committing a release key
or its password would be a security risk). To build a release you must supply
your own `mtrscope.keystore` in the project root and set the matching
credentials in `app/build.gradle`:

```groovy
signingConfigs {
    release {
        storeFile rootProject.file('mtrscope.keystore')
        storePassword 'YOUR_STORE_PASSWORD'
        keyAlias 'YOUR_KEY_ALIAS'
        keyPassword 'YOUR_KEY_PASSWORD'
    }
}
```

Consider reading the password from an environment variable or a
`gradle.properties` that is git-ignored rather than hard-coding it.

---

## Internals / How it works

| Piece | What it does |
| --- | --- |
| `MtrEngine` | Orchestrates the continuous trace using the system `ping` binary with increasing TTLs. |
| `GeoLookup` | Resolves each hop to country / ASN via `ip-api.com` (public IPs only), with a disk cache and country-centroid fallback for the map. |
| `Whois` / `Psl` | WHOIS lookups including eTLD extraction (`Public Suffix List`). |
| `DnsQuery` | Minimal raw UDP DNS resolver (A / AAAA / CNAME / MX / NS / TXT / PTR), random per-query id with reply matching and full bounds guarding. |
| `PortScan` | Concurrent TCP connect scanner with a family selector (IPv4 / IPv6 / both) that reports each family separately. |
| `NetInfo` | Active-interface report: transport, IP stack (via network capabilities), MTU, gateway, DNS servers. |
| `map.html` | A local Leaflet 1.9.4 page that draws the great-circle route, unwraps the antimeridian, and renders red direction arrows. |
| `HistoryStore` | Persists past traces for search / compare / favourite / clear. |

### Network stack handling

- Probes run **sequentially** (one TTL at a time) — concurrent probes inflate
  latency on real devices.
- The engine always passes `-t <ttl>` for MTR mode, including `ttl=1`; without
  it the first probe reaches the target directly and hides every real hop.
- Every start bumps an internal **generation** counter, so leftover replies
  from a previous target can never land in a fresh table.
- TCP Port Scan picks the family by explicit selection (not auto-fallback):
  a literal IPv4 / IPv6 address is always probed as typed, and a hostname
  follows the chosen family. Whether IPv6 is usable at all is decided by
  **asking an IPv6-only echo service for the device's own public IPv6 address**
  — a reply can only have arrived over IPv6, so it is proof, whereas
  `NET_CAPABILITY_INET6` (15 = IPv4, 16 = IPv6 — hidden constants, hence the
  magic numbers) is only a hint that some firmware omits and that may sit on a
  non-default route. A cached public IPv6 or a published IPv6 route is accepted
  for free; otherwise the probe runs before any IPv6 dial, and each probe is
  still classified **open / refused / timed out** so a closed port and an
  unreachable host are never conflated.

### Route map detail

- Hops with **private/internal** IPs (`10.x`, `192.168.x`, …) are skipped — they
  have no public geolocation.
- The route **origin** is your device's own public IP (shown as
  `IP [CC ASnnn Name]`), fetched best-effort with a short timeout so the map
  always opens quickly.
- The path is a **great circle** (shortest arc on the globe), so e.g.
  Hong Kong → US renders across the **Pacific**, not a flat line across Asia.

---

## FAQ

A longer version lives in-app under `☰ → FAQ`. Highlights:

- **Why are some hops internal IPs?** Those are your ISP's route-local
  addresses; they have no public geolocation and are skipped on the map.
- **Why do some hops show `*`?** No reply for that TTL on this round (a router
  that silences the packet or plain loss) — the row still counts as sent.
- **What does `[HK AS9429]` mean?** Country code + the autonomous system (ISP)
  that owns that IP.
- **Why does the map route cross the Pacific?** It draws the great-circle
  shortest path, not a flat straight line.
- **Why does a TCP port scan of an IPv6 address say "Your network does not
  support IPv6"?** The app asked an IPv6-only echo service for this device's own
  public IPv6 address and got nothing, which means no IPv6 packet can leave the
  device — so an IPv6 scan could only ever time out. Use an IPv6-capable network
  or scan IPv4. Note a **closed** port shows as *refused* (host was reached)
  while an **unreachable** one shows as *timed out*; those are not the same.
- **Why not just trust the OS "IPv6" capability flag?** Some firmware never sets
  `NET_CAPABILITY_INET6`, and on Android the default network can be IPv4-only
  even while IPv6 works, so the flag gives false answers in both directions.
  Reaching an IPv6-only service and reading back your own IPv6 address is the
  only end-to-end proof.

---

## About

- **Author:** Edward Poon
- **Email:** [edward@hkt.cc](mailto:edward@hkt.cc)
- **Company:** FOREWIN TELECOM GROUP LIMITED
- **Location:** HONG KONG, CHINA
- <https://www.hkt.cc> · <https://www.say.cc>

---

## License

*Add your license here (e.g. MIT / Apache-2.0) before publishing the repo.*
