# SnRTP

SnRTP is a lightweight, fully asynchronous, per-world random teleport for Paper.

## Features

- One GUI menu with a block-icon per world (overworld, nether and end by default).
- Configure any world: ring radius, center, warmup, cooldown and optional permission.
- Fully async safe-location search that never freezes the server.
- Never lands players inside protected regions (WorldGuard and ProtectionStones claims).
- Dimension-aware safe scan: highest block on the surface, a downward pocket scan in the Nether.
- Toggleable particle and sound effects with portal, spiral and beam styles.

## Optional integrations

- **WorldGuard**: random teleports skip any protected region. Without it, region avoidance stays inactive and teleports still work.
- **ProtectionStones**: player claims are avoided automatically, since every claim is a WorldGuard region. No direct hook is needed.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
