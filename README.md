# MtrScope

**Continuous per-hop network probing (mtr) for Android.**

MtrScope is a lightweight, native Android implementation of the classic
[`mtr`](https://en.wikipedia.org/wiki/MTR_(software)) (My Traceroute) tool.
It continuously probes every hop between your device and a target host,
re-live-updating loss % and latency so you can watch a network path degrade or
recover in real time — a familiar workflow if you are used to `mtr` on Linux.

![Platform](https://img.shields.io/badge/platform-Android-0891B2)
![Version](https://img.shields.io/badge/version-1.8.31-green)
![API](https://img.shields.io/badge/API-26%2B-blue)

## Install

**Google Play** — <https://play.google.com/store/apps/details?id=com.paperclip.mtr>

**Or download the APK directly** — <https://github.com/saycc1982/mtr-scope/releases/latest>

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
  while tracing. IPv4 (`in-addr.arpa`) and IPv6 (`ip6.arpa`) are both handled.
- **WHOIS** — tap a hop (or a domain/IP in the header) to look up WHOIS for
  IPs and eTLD domains. Replies are decoded UTF-8 first, so non-ASCII registry
  data such as the Chinese registrant / registrar returned for `.cn` and
  `.com.cn` domains reads correctly.
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
    result shown separately. Before any IPv6 target is probed, the app fetches
    **its own public IPv6 from an IPv6-only service** — a host that publishes a
    AAAA record and *no* A record, so a reply proves IPv6 really works. If none
    comes back it reports **"Your network does not support IPv6"** and offers a
    one-tap *switch to IPv4* rather than pretending every port is closed. Results
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

| Empty start screen | History dialog |
| --- | --- |
| ![Empty start screen](screenshots/dark-empty.png) | ![History dialog](screenshots/history-dialog.png) |

---

## Getting started

**Google Play** (auto-updates, no sideload prompt):
<https://play.google.com/store/apps/details?id=com.paperclip.mtr>

This repository is also the **direct APK distribution** channel for MtrScope —
it carries no source code, and the single signed APK sits on the latest
[release](https://github.com/saycc1982/mtr-scope/releases/latest):

1. Download `mtrscope-v<version>.apk`.
2. Open it on the device and allow the install when prompted (it is a sideload).
3. The app needs no account, no ads and only the two network permissions it uses
   (`INTERNET` + `ACCESS_NETWORK_STATE`). It runs on Android **8.0** (API 26) or
   newer.

The same APK is mirrored on the author's own download server, so a release stays
obtainable even if a CDN ever drops it — the exact URL for the current build is
listed in that release's notes.

The Play listing and the GitHub APK carry the same build, so picking either one
gets you the identical app; the Play route simply keeps itself updated.

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

## Internals / How it works

The following describes the design at a concept level. It is documentation, not
buildable code — this public repository contains no source.

| Piece | What it does |
| --- | --- |
| `MtrEngine` | Orchestrates the continuous trace using the system `ping` binary with increasing TTLs. |
| `GeoLookup` | Resolves each hop to country / ASN via `ip-api.com` (public IPs only), with a disk cache and country-centroid fallback for the map. |
| `Whois` / `Psl` | WHOIS lookups including eTLD extraction (`Public Suffix List`). Port 43 replies are decoded UTF-8 first and fall back to Latin-1 only when strict UTF-8 yields a replacement character (measured: CNNIC answers in UTF-8, Verisign / APNIC / IANA are pure ASCII). |
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
  follows the chosen family. Whether IPv6 is usable at all is decided **only**
  by fetching the device's own public IPv6 from an IPv6-only service (AAAA
  record, no A record). Nothing else on the system is trusted as proof:
  `NET_CAPABILITY_INET6` (15 = IPv4, 16 = IPv6 — hidden constants, hence the
  magic numbers) is set for the pseudo/dummy IPv6 prefixes that IPv4-only
  cellular links hand out, so a v4-only phone can legitimately be labelled
  "dual-stack", and an echo service that also has an A record is reachable over
  v4 and answers with your **IPv4** address. The capability flag is therefore
  shown in *Network Info* as a claim only ("IPv4 only — OS claims IPv6, but no
  IPv6 address is reachable"), never as proof. Each probe is still classified
  **open / refused / timed out**, and a *refused* result is deliberately not
  taken as proof of a dead stack, because a network that can fabricate an IPv6
  prefix can fabricate the reset as well.

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

- **Why does WHOIS show accented garbage like `ä¸­å­½çµä¿¡`?** That was our
  bug, since fixed: WHOIS replies are decoded as UTF-8 first (with Latin-1 only as
  a fallback), and registries such as CNNIC (`whois.cnnic.cn`) really do return
  UTF-8 Chinese — `chinatelecom.com.cn` yields
  `Registrant: 中國電信集團公司`. If a reply ever looks wrong again, the
  registry in question is the exception, and the raw bytes can be checked with
  `printf 'domain\r\n' | nc <whois-server> 43`.
- **Why are some hops internal IPs?** Those are your ISP's route-local
  addresses; they have no public geolocation and are skipped on the map.
- **Why do some hops show `*`?** No reply for that TTL on this round (a router
  that silences the packet or plain loss) — the row still counts as sent.
- **What does `[HK AS9429]` mean?** Country code + the autonomous system (ISP)
  that owns that IP.
- **Why does the map route cross the Pacific?** It draws the great-circle
  shortest path, not a flat straight line.
- **Why does a TCP port scan of an IPv6 address say "Your network does not
  support IPv6"?** The app tried to fetch this device's own public IPv6 from an
  IPv6-only service and got nothing, so no IPv6 packet can leave the device and
  an IPv6 scan could only ever time out. Use an IPv6-capable network or scan
  IPv4. A **closed** port shows as *refused* (the host was reached) while an
  **unreachable** one shows as *timed out* — those are not the same.
- **Network Info says "Dual-stack (v4+v6)" but IPv6 does not work — why?**
  Carrier IPv4-only links can hand out a *pseudo* IPv6 prefix (for NAT64), and
  Android reports its IPv6 capability for it, so the label can be true to the OS
  while no IPv6 traffic flows. Where the app can measure the difference it says
  so: *"IPv4 only — OS claims IPv6, but no IPv6 address is reachable"*. Never
  treat that capability label as proof; only actually fetching your own IPv6
  address is.
- **Why is the public IPv6 row sometimes missing?** It is only shown when an
  IPv6-only service actually answers with an IPv6 address; when that probe fails
  the stale value is cleared, so a v4-only network never shows a v6 address it
  cannot use.

---

## About

- **Author:** Edward Poon
- **Email:** [edward@hkt.cc](mailto:edward@hkt.cc)
- **Company:** FOREWIN TELECOM GROUP LIMITED
- **Location:** HONG KONG, CHINA
- <https://www.hkt.cc> · <https://www.say.cc>
- **Google Play:** <https://play.google.com/store/apps/details?id=com.paperclip.mtr>

---

## License

The published binaries are distributed under the terms stated in the release
notes of the version you download. This repository is a distribution channel: it
holds the installable APK, the changelog and the documentation only — **no
source code is published here**, and the app is closed-source.
