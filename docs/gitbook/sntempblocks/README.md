# SnTempBlocks

SnTempBlocks makes player-placed blocks expire inside WorldGuard zones. Bucket-placed water and
lava are tracked together with every block their flow reached, so a zone never keeps something
nobody remembers placing. Blocks outside a zone are never touched.

## Features

- **Two engines, chosen per zone.** `PER_BLOCK` gives every block its own lifetime, set per
  material with a default for everything else. `INTERVAL` runs one timer for the whole zone and
  wipes everything tracked at once, with warnings before it fires.
- **Liquids are handled properly.** A bucket creates a body, the plugin follows it as it spreads,
  and the whole body disappears in the same instant. Spread that could not be cleaned up later is
  cancelled instead of allowed.
- **Waterlogged blocks keep their block.** When water only flags a stair or a fence, expiry
  clears that flag. It never deletes the block a player placed.
- **Restart-safe.** Lifetimes run on real time, not ticks, and the index is mirrored to a
  database. Blocks whose lifetime elapsed while the server was down are removed at startup, and
  an interval cycle neither restarts nor skips.
- **Built to stay cheap.** The whole plugin owns two repeating tasks, every removal batch is
  capped per tick, and a block placed in a world with no zone costs one hash lookup.
- **Never force-loads a chunk.** A removal waiting on an unloaded chunk is executed when the
  server loads that chunk for its own reasons.

## Optional integrations

- **PlaceholderAPI**: unlocks the `sntempblocks` expansion, so a scoreboard or hologram can show
  the countdown and the tracked count of a zone. Without it the plugin runs normally and simply
  registers nothing.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
