# SnDTC

Destroy The Core events for Paper servers. Turn any block into a named "core" with a health
pool, let your players grind it down, pay the top damagers with your own console commands, and
bring it back on a schedule.

## Features

- **Any block is a core.** Look at a block, run `/sndtc create <name>`, and it has health, a
  schedule and a scoreboard. Each core keeps its own materials, health pool, schedule, display
  range and rewards.
- **Hardness is the difficulty.** One completed block break is one point of damage, so the
  material you choose and the tool the player holds are the entire difficulty model. A diamond
  block core and an obsidian core are genuinely different events.
- **The block is protected.** Twenty event handlers keep a core's block where the registry
  believes it is: explosions, pistons, fluids, sponges, dispensers, buckets, fire, decay, and
  every way a block can grow, spread or form into a core's coordinate.
- **Two schedule grammars.** Simple intervals (`30s`, `4h`, `1d12h`) that run on a fixed
  wall-clock grid, or 5-field cron (`0 0 * * 1`) when an event has to happen at a set time.
- **Rewards you author.** Console commands per finishing position, plus a default list for
  everyone else, with a cap on how many are paid in one destruction. Any core can override the
  global lists with its own.
- **Live display.** A boss bar and an action bar for everyone in range, and a floating hologram
  above the block, all range-based and rendered per viewer.
- **Region-friendly.** Cores inside protected regions work with no setup and no dependency on
  WorldGuard.

## Optional integrations

- **PlaceholderAPI**: unlocks the `%sndtc_*%` placeholders. Without it, the placeholders simply
  do not resolve; everything else works normally.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
