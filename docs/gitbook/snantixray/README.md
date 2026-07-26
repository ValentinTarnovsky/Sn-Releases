# SnAntiXray

SnAntiXray hides ore from X-Ray cheaters at the packet level, per player. Your world is never
modified. Only what each player is sent changes, so two players standing in the same cave can be
shown different terrain while the server keeps one true world.

It also plants decoy ore veins that exist for nobody. A legitimate player cannot see or target one,
so any interaction with one is a strong cheat signal that feeds a detection score.

## Features

- **Anti X-Ray**: configured ore blocks are sent as plain stone, deepslate, netherrack or end
  stone, chosen per dimension, until that specific player legitimately exposes them.
- **Anti ESP**: chests, barrels, furnaces, spawners, beds and other block entities are hidden the
  same way. Their block-entity payload is withheld, so container contents never leave the server.
- **Fake ore veins**: decoy veins injected per world into the client view only. They are
  deterministic, so a decoy stays exactly where it was across chunk reloads, relogs and restarts.
- **Cheat detection**: breaking or damaging a decoy raises a per-player score. A periodic sweep
  applies hourly decay and alerts your staff at two thresholds.
- **Alerts only**: SnAntiXray never punishes anyone automatically. It tells your staff and stops
  there, so a false positive costs a look rather than a ban.
- **Bypass**: a persisted per-player flag, or the `snantixray.bypass` permission, makes all three
  layers skip entirely. The holder sees the true world.
- **Folia support**: runs on the region-threaded Paper fork with no extra configuration.

## How the reveal works

A hidden block becomes real the moment a player legitimately exposes it, and it stays real. The
plugin watches every change that can expose terrain: breaking, explosions, pistons, buckets and
fluid flow. When a block stops blocking vision, everything behind it that this plugin was hiding is
sent for real to every player who can see that chunk.

That decision is remembered for the rest of the session, so a chunk resent an hour later still
carries the block. A chest you place yourself never vanishes on you.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
