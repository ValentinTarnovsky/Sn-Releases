# SnDrop

SnDrop blocks item drops for every player by default. A player runs `/drop` and gets a personal,
time-boxed window during which their drops go through; when it expires the block returns on its
own. It is the plugin you install when dropping loot on the floor is how items leave your
economy.

## Features

- **Blocked by default, opened on demand.** Nobody can drop anything until they ask for a window,
  and the window belongs to that one player. There is no server-wide toggle to forget about.
- **Expiry costs nothing.** A window is a single deadline compared against the clock at the moment
  of each drop attempt. The plugin owns no repeating task, no countdown and no sweeper, so a busy
  server pays one map lookup per drop attempt and nothing per tick.
- **The crafting-grid pocket is closed.** Vanilla preserves the 2x2 crafting grid across a death
  and never includes it in the death drops, which made it a smuggling pocket: park the valuables,
  die on purpose, pick everything up on respawn. SnDrop forces those slots into the death drops.
  The crafting result slot is cleared but never dropped, so nothing is duplicated.
- **A warning that does not spam.** The "you cannot drop items right now" line is rate-limited per
  player, at a rate you choose. Holding the drop key gives a steady reminder, not a wall of text
  and not silence.
- **Restart-clean.** Windows live in memory and die with the server, so a restart never leaves
  somebody with drops silently unlocked.
- **Traceable.** `/drop debug` turns on runtime output that says why a drop was blocked or
  allowed, what each `/drop` did to the window, and what the death sweep took.

## What it deliberately does not do

Reading this section will save you a support ticket.

- **There is no bypass permission.** Operators and creative-mode players are blocked exactly like
  a fresh player. If you need staff exempt, that is a change request, not a setting.
- **Only the drop action is guarded.** Dropping via the drop key or `Q` fires the event SnDrop
  cancels. Moving items into other containers is not this plugin's job.
- **A window does not survive a disconnect.** Log out mid-window and you come back blocked. This
  is deliberate: it stops a player banking an open window for later.
- **Nothing is written to disk.** SnDrop stores no player data at all.

## Requirements

- Java 21+
- Paper 1.21 or newer
- `SnLib.jar` in `plugins/`
