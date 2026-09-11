# Changelog

## v1.8.28
- IPv6 availability is now decided **only** by fetching the device's own public IPv6 from an AAAA-only (no A record) echo service — both previous shortcuts were removed and both were proven wrong on a real carrier network:
  - `NET_CAPABILITY_INET6` is set for the **pseudo/dummy IPv6 prefixes** that IPv4-only cellular links hand out for NAT64, so "Dual-stack (v4+v6)" in Network Info can be true to Android while no IPv6 traffic flows — this is why the previous build still reported v6 as available and sent no warning
  - an echo service that *also* has an A record is reached over IPv4 and answers with your **IPv4** address (`api64.ipify.org` was measured doing exactly this) — it has been removed from the probe list
  - a cached "public IPv6" is no longer trusted either, and a failed probe now clears that field so the header never shows a v6 address the network cannot use
- An IPv6 port scan that cannot be delivered now reports the failure instead of a misleading "all scanned ports closed (connection refused)": the probe result is classified *open / refused / timed out*, and a *refused* answer is deliberately **not** read as "v6 works", since a network able to fabricate a prefix can fabricate the reset too
- Network Info now shows the OS claim separately from the measurement ("IPv4 only — OS claims IPv6, but no IPv6 address is reachable — pseudo/dummy prefix") and never labels the link Dual-stack from an unmeasured verdict
- Added `tools/check-v6-endpoints.sh` which verifies each probe host really is AAAA-only, has no A record, and is live (run it before touching `MY_IP_SOURCES_V6`; this repo's build host has no IPv6, so liveness must be confirmed on an IPv6-capable network)

## v1.8.27
- TCP Port Scan now determines IPv6 availability the only reliable way: it asks an **IPv6-only echo service for the device's own public IPv6 address** — a reply can only arrive over IPv6, so silence proves IPv6 is unusable. The OS `NET_CAPABILITY_INET6` flag is kept only as a free fast-path and is no longer trusted alone (firmware can omit it, and Android's default network may be IPv4-only even when IPv6 works — which is why an IPv6 scan on an IPv4-only network previously showed no warning)
- The check runs off the UI thread and is cached per network (invalidated when the network changes); a fresh cached public IPv6 or a published IPv6 route short-circuits it
- When IPv6 is unusable, no probes are sent at all: the dialog reports "Your network does not support IPv6" and turns its own button into *switch to IPv4*
- README (TCP Port Scan, Network stack handling, FAQ) and AGENTS.md document the probe-based rule

## v1.8.26
- TCP Port Scan: IPv6 detection now has a second, independent net. If every IPv6 probe **timed out** (nothing refused, nothing answered) and no network route advertises IPv6, the result reports **"No IPv6 connectivity"** instead of a misleading "no open ports" — this catches firmware that does not publish `NET_CAPABILITY_INET6` (or publishes the IPv6 route outside the default network), which previously let an IPv6 scan quietly look like "all ports closed"
- TCP Port Scan: each probe is now classified **open / refused / timed out** and the per-family block says which ("N/M open", "N/M refused", "no route — all timed out"), so a closed port and an unreachable host are never confused
- README: TCP Port Scan + Network-stack-handling + FAQ sections document the two safety nets and the open/refused/timed-out distinction

## v1.8.25
- Documentation brought back in line with the app: README feature list rewritten (multi-tab + batch, health bar + trend, public v4/v6 IP, MORE = Net Info / DNS Lookup / TCP Port Scan with the IP-family selector, history search/compare/favourite, PNG share, 3-state theme, widget) — the removed **Batch Summary / Geo Summary / Trace Diff / Schedule Trace / Gateway Test / unreachable alarm / background scheduler** entries are gone for good
- In-app About no longer advertises "trace diff" or a "background scheduler"
- Version badge now tracks the real release (1.8.25)
- publish-github.sh now mirrors **README.md** to the public repo together with CHANGELOG.md (docs only)

## v1.8.24
- TCP Port Scan IPv4+IPv6 now shows BOTH families (each scanned independently) instead of short-circuiting after the first IPv6 hit: result has one block per family with its IP, open count, and port list; a hostname with only v4 or only v6 records naturally shows a single block
- No-IPv6 detection hardened: `hasIpv6Route()` scans `cm.allNetworks` (any route with NET_CAPABILITY_INET6) — checking only the active/default route wrongly reports "no IPv6" on the common Android split where default is IPv4-only and IPv6 lives on a separate route
- Warn text: "This network has no IPv6 route…"

## v1.8.23
- TCP Port Scan: added an **IP FAMILY** dropdown (IPv4 / IPv6 / IPv4+IPv6, default IPv4, remembered). Forces which stack a hostname is dialed over, so you can deliberately test a host's IPv4 or IPv6 path
- Rule: a literal IPv4 address is always probed over IPv4 and a literal IPv6 over IPv6, regardless of the selection; a hostname follows the selection (Both = IPv6 first, then IPv4 fallback)
- If the network route has no IPv6 (no NET_CAPABILITY_INET6) and an IPv6 probe is requested (IPv6 literal, or hostname with IPv6/Both), the scan now **stops and warns** "No IPv6 connectivity" with a one-tap "switch to IPv4" option, instead of silently returning no-open-port

## v1.8.22
- TCP Port Scan hostname resolution fixed: the scan now tries every resolved address (IPv6-first) and uses the **first family that actually responds**, so a hostname whose AAAA records are unreachable (no/broken IPv6 route on the device) no longer appears as "cannot resolve / no open ports" — it falls back to the reachable stack (this is why plain-IP scans worked but hostnames did not)
- DNS pre-check now distinguishes a real "Resolve failed" (no records) from a resolvable-but-no-open-port target, so the message is accurate
- Host input is normalized consistently (scheme/path/port and IPv6 brackets stripped) so a pasted URL or `[::1]:443` scans the right host

## v1.8.21
- DNS resolver hardened: random per-query transaction id + reply id check (defends off-path spoofing), full bounds-guarding so a malformed/truncated packet can never crash a lookup, and label-length clamp
- rDNS cache eviction is now a single atomic (size-check + evict + put) step — previously the separate size/keys/remove calls could corrupt the map under concurrent probes
- TCP Port Scan dialog: explicit "HOST / IP" and "PORTS" field labels, keyboard auto-raises and pre-filled text is selected on open, removed the duplicated hint setter
- App is English-only by design (no localization) — documented in AGENTS.md

## v1.8.20
- TCP Port Scan: the host/IP field now opens with the keyboard raised and the field focused, so it is obviously editable even when pre-filled
- TCP Port Scan: the scan title and result now print the resolved IP and family — `[IPv6]` or `[IPv4]` — and a hostname is resolved IPv6-first (Happy Eyeballs), so you can tell which stack was used
- Engine: the `ping` probe process is now explicitly destroyed on the no-reply path (previously leaked a process/fd on every dropped probe)

## v1.8.19
- TCP Port Scan: the host/IP is now an editable field (defaults to the current input; a URL is auto-stripped to its hostname) so you can scan any address directly from the dialog
- TCP Port Scan: remembers the last used port list (port_spec) — the next scan opens with your previous list; the factory default is only used the very first time

## v1.8.18
- Public IP: fixed Wi-Fi→cellular not refreshing promptly. On Android the Wi-Fi-off broadcast is fired before the cellular route becomes the default, so the first check still saw the old (Wi-Fi) network. Every CONNECTIVITY_ACTION now (a) busts and re-resolves immediately, (b) always busts regardless of route comparison, (c) schedules a one-shot retry a moment later so the new IP appears even when the route settles late — works both while the app is open and on resume

## v1.8.17
- Public IP: replaced the NetworkCallback-based detection (often not fired by the OS on a Wi-Fi↔cellular route switch) with a CONNECTIVITY_ACTION broadcast receiver, which is guaranteed to fire whenever the route changes. Switching Wi-Fi off to use the mobile data card now re-resolves the public IP instantly
- Public IP: added a resume-time safety net — on returning to the app the active route is re-checked once, so an IP change that happened while backgrounded is also caught

## v1.8.16
- Public IP: fixed the WiFi→cellular / network-change detection not firing — the connectivity callback runs on a background worker (no Looper), so the refresh was silently dropped. Callbacks are now marshalled onto the main looper, so switching networks (e.g. turning off Wi-Fi and using the mobile data card) now busts the cache and refreshes the public IP in real time

## v1.8.15
- Public IP: now re-detected automatically the instant the network route changes (Wi-Fi→Wi-Fi, Wi-Fi↔cellular, reconnect) — no longer stuck on the old network's cached address (cache busted only on a real route change, so no endpoint spam)
- Public IP endpoints: pruned all dead / unreachable sources (v4v6.ident.me, 6.ip.sb, 6.myip.cc, 6.tned.me and others had no DNS record) and kept only confirmed-live AAAA endpoints (v6.ipinfo.io, v6.ident.me, api64.ipify.org); v6 responses are validated to contain a real IPv6 so an IPv4-only hop never echoes the v4 address into the v6 row

## v1.8.14
- Geo/ASN: IPv6 addresses are now split into public vs private/reserved (loopback ::1, link-local fe80::/10, ULA fc00::/7, multicast ff00::/8, site-local, v4-mapped, ORCHIDv2) — public IPv6 hops get geolocated, private ones are skipped just like private IPv4 so the free ip-api.com quota is not wasted

## v1.8.13
- MORE menu: removed Batch Summary, Geo Summary, Trace Diff, Schedule Trace and Gateway Test (all code dropped — rarely useful, cluttered the menu)
- Geo: removed the dead ipapi.se fallback; lookups go straight to ip-api.com
- Manifest: dropped the now-unused WAKE_LOCK / POST_NOTIFICATIONS permissions

## v1.8.12
- Fixed: ASN/Country codes never show up after stopping the trace — geo callback was dropped on a false 'not running' guard; now the result always updates its row, tab-scoped

## v1.8.11
- MTR result header: dropped the `(1.0s x 30)` interval/hops suffix to save screen space
- Geo/ASN fallback to ipapi.se when ip-api.com returns an empty result for an IP (fixes blank ASN tags and map route failures on some public hops)

## v1.8.10
- Shared PNG: ad text and QR auto-flip ink for the current theme (white on dark, black on light)
- Shared PNG: hops with no responding IP show '*' instead of an empty cell

## v1.8.9
- Shared PNG: unresponsive hops now show '*' instead of an empty cell
- Shared PNG: ad text and QR color auto-flip with the current theme (dark mode -> white, light mode -> black)

## v1.8.8
- Shared PNG: the long IPv6 Target line soft-wraps onto the next line inside the exported image too, so it no longer runs past the right edge

## v1.8.7
- Header: long IPv6 target lines now wrap onto the next line instead of being cut off
- Shared PNG: ad text moved to the footer right corner (left of the QR code)

## v1.8.6
- MORE menu: "Gateway Test" — one tap fills the current gateway IP and starts the MTR run
- Shared PNG: footer now includes a QR code pointing to https://www.say.cc

## v1.8.5
- Fixed: History was empty unless STOP was pressed — every run is now recorded at the end of its first completed cycle
- Removed: packet-loss vibrate (feature and setting fully removed; VIBRATE permission dropped)

## v1.8.4
- Widget: tap the widget to type a host/IP in a quick dialog, then ▶ MTR / ▶ PING to run instantly
- Widget: own ▶ MTR / ▶ PING buttons trace the saved host without opening the app

## v1.8.3
- Tab pill shows a red ✕ when its target goes unreachable (clears on recovery)
- Alarm toasts name the host; simultaneous multi-tab losses coalesce into one alert
- Added FAQ entry: which tab went down

## v1.8.2
- FAQ: documented the configurable home widget

## v1.8.1
- Fixed theme toggle (Light/Dark modes now actually applied on tap)
- Home-screen widget: configurable host, mode (MTR/PING), freely resizable; long-press to configure
- About screen: expanded feature list; semantic version shown (1.8.x)

## v1.8
- Theme button: 4 distinct icons (follow-system / light / dark / auto), toast on each toggle, long-press to jump to Auto
- FAQ: interact-with-results and long-press-hop entries

## v1.7 and earlier
- Long-press a hop IP to re-fetch its ASN/Geo
- Input hints for URL auto-strip and comma-separated batch mode
- Spinner arrow moved to the left so it never overlaps the value text
- PNG export: public IPv4/IPv6 header rows, table grid/striping, footer, column titles
- Dark-mode dialog text contrast fix
