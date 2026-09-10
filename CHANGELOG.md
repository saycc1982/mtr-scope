# Changelog

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
