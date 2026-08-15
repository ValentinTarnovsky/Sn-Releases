# SnRooms

PvP rooms built directly in the world. An admin marks a cuboid with the selection wand; players
walk in while the room is open, and once the configured team composition is inside, the room
seals its faces with real blocks, applies its effects and runs the round. When one side is left
standing the shell comes down and the region is restored exactly as it was.

There is no queue, no arena schematic and no command for players. A room is a place.

## Features

- **Regions, not commands.** Players join a round by walking into it. When the number inside
  matches `teams x team-size` and they group into that many balanced sides, the room counts
  down and seals.
- **An exact shell.** The room only ever takes over blocks it can put back, and it records every
  single one, so unsealing restores the region precisely - including anything you built inside
  it. Blocks it cannot restore faithfully are left alone.
- **Crash-safe.** Every sealed shell is written to `shell-state.yml`, so a server that dies
  mid-round comes back up with its rooms open instead of full of glass.
- **Never a freeze.** A large shell is placed AND removed a few hundred blocks per tick, and a
  region too large to seal is refused rather than attempted.
- **Anti-glitch.** Anyone who reaches the inside of a room that is not open is returned to the
  last position they held outside it - the spot they walked from, not a guess.
- **Clan teams.** With SnClans installed, rooms of more than one player per side build their
  sides from clans, and mutually allied clans can be made to fight as one.
- **Staff can watch.** `snrooms.admin.bypass` holders walk into a fighting room without being
  evicted and without being counted into the composition, so an admin watching a 1v1 does not
  block it.
- **English and Spanish** message files, both merged on update without losing your edits.

## How a round runs

1. The room sits **open**. Players walk in and out freely.
2. The composition completes: `teams x team-size` players inside, in that many balanced sides.
3. The room **counts down** (`start-countdown`, 5s by default), shown on the action bar, a boss
   bar, a title, or not at all.
4. The shell **seals** and the round is on. Configured potion effects are applied.
5. Players are eliminated by dying, disconnecting, or - if you leave it on - walking out.
6. One side is left standing. The outcome is announced and the room stays **closed** for
   `close-after-round`, so the winner is not jumped by whoever was waiting outside.
7. The shell comes down, survivors are sent to the room's exit if it has one, and the room is
   open again.

## Requirements

- Java 21+, Paper 1.20.4+
- SnLib.jar 1.27.0 or newer
- A licence key in `plugins/.Sn-License/license.yml`
- Optional: SnClans, for clan-based teams

## Pages

- [Installation](installation.md)
- [Commands](commands.md)
- [Permissions](permissions.md)
- [Configuration](configuration.md)
- [FAQ](faq.md)
