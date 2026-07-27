# SnAntiXray

SnAntiXray hides ore from X-Ray cheaters at the packet level, per player. Your world is never
modified. Only what each player is sent changes, so two players standing in the same cave can be
shown different terrain while the server keeps one true world.

It also plants decoy ore veins that exist for nobody. A legitimate player cannot see or target one,
so any interaction with one is a strong cheat signal that feeds a detection score.

## Features

- **Anti X-Ray**: configured ore blocks are sent as plain stone, deepslate, netherrack or end
  stone, chosen per dimension, for as long as they stay buried. Ore that is already open to a cave
  is sent as it really is.
- **Anti ESP**: chests, barrels, furnaces, spawners, beds and other block entities are hidden the
  same way. Their block-entity payload is withheld, so container contents never leave the server.
- **Fake ore veins**: decoy veins injected per world into the client view only, and only into
  positions sealed off on all six sides. They are deterministic, so a decoy stays exactly where it
  was across chunk reloads, relogs and restarts.
- **Cheat detection**: breaking or damaging a decoy raises a per-player score. A periodic sweep
  applies hourly decay and alerts your staff at two thresholds.
- **Alerts only**: SnAntiXray never punishes anyone automatically. It tells your staff and stops
  there, so a false positive costs a look rather than a ban.
- **Bypass**: a persisted per-player flag, or the `snantixray.bypass` permission, makes all three
  layers skip entirely. The holder sees the true world.
- **Folia support**: runs on the region-threaded Paper fork with no extra configuration.

## What gets hidden

A block is replaced only when **all six of its neighbours block vision** - when it is genuinely
buried. A block that is already open to air, water or lava is sent exactly as it is: an ore on a
cave wall, a spawner in a dungeon room, a chest standing in a basement. A player standing there
already sees those, so taking them away would break the game rather than protect it.

- **An honest player** sees a completely normal world. Nothing pops in, nothing turns to stone,
  containers can be seen and opened, dungeons and mineshafts look like dungeons and mineshafts.
- **An X-Ray client** still sees nothing buried, which is essentially every ore worth cheating for.
  What it can see is the ore already exposed in caves - exactly what any player finds by walking
  into that cave. This is the same rule Paper's own obfuscation and Orebfuscator apply, and the
  same trade they make.
- **An ESP or radar client** still cannot find a container walled into terrain, or read its
  contents. A container standing in an open room is in plain sight for anyone who walks in, so it
  is not hidden either.
- **A decoy vein** is only ever planted in a sealed position, so no honest player is shown one. Dig
  such a position open and the plugin immediately sends you the real block, so the decoy is gone
  before you could ever mine it. Only a client that sees through terrain gets to look at one.

## How the reveal works

A hidden block becomes real the moment a player exposes it, and it stays real. The plugin watches
every change that can expose terrain: breaking, explosions, pistons, buckets and fluid flow. When a
block stops blocking vision, everything behind it that this plugin was hiding is sent for real to
every player who can see that chunk.

That decision is remembered for the rest of the session, so a chunk resent an hour later still
carries the block. A chest you place yourself never vanishes on you.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
