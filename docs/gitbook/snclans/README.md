# SnClans

SnClans is a full-featured clans and guilds system for Paper servers. Players create clans, manage members through a role ladder, fight together, rally to a banner, and climb custom-point leaderboards.

## Features

- Create and manage clans with names, optional tags, and a Leader / Co-Leader / Officer / Member role ladder.
- Closed by default so members join by invite, or set a clan open for instant joins; a per-clan permission matrix lets the leader decide which roles may invite, kick, promote, and more.
- Clan homes with a teleport warmup, plus rally banners with a configurable countdown hologram, rendered by SnLib's built-in holograms or by DecentHolograms.
- An ally system with its own master switch, friendly-fire control for clanmates and allies, and dedicated clan and ally chat channels.
- Custom point types with leaderboards, awarded on kills or granted by staff.
- Look up any clan with `/clan info`, by clan name or by an online player's nick, and notify online members when a clanmate connects or disconnects.
- Every interactive feature shows as a GUI or in chat, and can be gated by WorldGuard regions.

## Optional integrations

SnClans runs on **SnLib** alone. Three optional soft dependencies unlock extra behavior when present:

- **PlaceholderAPI**: exposes the `%snclans_*%` placeholders.
- **WorldGuard**: enables region gating for homes, banners, and PvP.
- **DecentHolograms**: an alternative backend for the rally banner hologram. When it is absent, the hologram falls back to SnLib's built-in renderer.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
