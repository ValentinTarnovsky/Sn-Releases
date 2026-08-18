# SnEnvoys

Timed envoy drop events for Paper servers. A block (a chest, or a custom player-head skin)
appears at an admin-defined location on a fixed interval. The first player to click it wins a
weighted random reward, and the block is restored to whatever it was before.

## Features

- **Timed envoy events.** Every `interval-minutes`, up to `envoys-per-event` locations from an
  admin-curated pool spawn a claimable block, with pre-event warnings, a during-event boss bar
  and periodic reminders.
- **An in-game location editor.** `/envoy editor` hands the admin diamond blocks: place one to
  register a spawn location, break one to remove it. Nothing is ever hand-edited in a file.
- **Weighted rewards.** Each reward carries a relative chance, one or more console commands, and
  an optional broadcast. Add, rename or delete rewards freely in `config.yml`.
- **Optional Supply Drop.** A second, independent event: a block falls from the sky at a random
  point inside a ring, marked by a locator beam, and is protected from griefing until claimed.
- **PlaceholderAPI countdowns** for both events, and a public developer API for other plugins.

## Optional integrations

- **PlaceholderAPI**: unlocks the `%snenvoys_*%` placeholders. Without it, the placeholders
  simply do not resolve; everything else works normally.
- **WorldGuard**: Supply Drop landing spots avoid regions when WorldGuard is installed. Without
  it, region avoidance is skipped entirely and drops can land anywhere in the ring.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
