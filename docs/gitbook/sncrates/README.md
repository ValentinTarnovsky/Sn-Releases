# SnCrates

Physical and virtual crates with animated openings, weighted rewards, keys and an in-game editor.

You describe a crate once - its rewards, their weights, their limits and what winning one runs -
and players open it by right-clicking a block in the world, by clicking it in a menu, or with a
command. Four opening animations are included. Everything a crate does is editable in game, without
touching a file.

## What it does

- **Three ways to hold a key, per crate.** `PHYSICAL` is a real item in the inventory, `VIRTUAL` is
  a balance stored in the database, `PERMISSION` is a node. A crate accepts any non-empty subset,
  so the same crate can take a key item from players and a permission from a rank.
- **Physical keys cannot be destroyed by handling them.** Placing one as a block, putting it in an
  item frame, putting it on an armour stand and eating it are refused for everyone, operators
  included, and there is no bypass. Storing keys in chests, trading them and dropping them stay
  allowed.
- **Crate blocks.** Bind any block in the world to a crate from the editor. Right-click opens one,
  sneak + right-click mass-opens, left-click shows the preview. A bound block cannot be mined and
  survives explosions.
- **Four animations, plus none.** A CS:GO-style scrolling roulette, a multi-row wheel, a grid of
  cards flipped one by one, and a quick flourish. `NONE` opens instantly with no menu. Every timing,
  sound and particle is configurable, globally and per crate.
- **The reward is decided before the first frame renders.** Closing the menu, disconnecting mid-spin
  or the plugin disabling mid-spin each deliver the reward exactly once, and consume exactly one key.
- **Weighted rewards with real limits.** Per-player and global caps, either as lifetime caps or over
  a rolling window you set in seconds. A reward can hand over its item, run console commands, both,
  or be display-only.
- **A read-only preview** with the computed chance of every reward, and an optional per-player
  filter: a player can turn a reward off for themselves so it is never handed to them, without
  changing anybody's odds.
- **Withdraw**, on crates that accept both key kinds: turn a virtual balance back into the key item
  from inside the preview.
- **The whole plugin is editable in game.** `/crates editor` creates and deletes crates, edits every
  setting, adds rewards from the item in your hand, sets weights, limits and win commands, binds
  blocks and duplicates anything. Chat prompts handle the values that need typing.
- **An append-only open log**, one readable line per open, for when you need to answer "what did
  they actually win".
- **PlaceholderAPI output**: total opens, per-crate key balance, per-crate opens and the computed
  chance of any reward.
- **A small developer API**: two Bukkit events, both notifications, that keep their real class names
  through obfuscation.

## Typical use

A server that sells crate keys. Configure a crate per tier, bind a block for each in spawn, and
hand keys out with `/crates givekey` or a webstore command. Players right-click the block, watch the
roulette, and the win is announced. Rare rewards get `broadcast: true` and a `per-player-limit: 1`
so nobody wins the same rank twice.

The same shape covers vote crates (a `VIRTUAL` balance topped up by your vote listener, opened from
`/crates` with no block anywhere), rank-gated crates (`PERMISSION`, no key item at all) and event
crates you delete afterwards.

## Requirements

Java 21+, Paper 1.20.x or 1.21.x, and **SnLib** in `plugins/`. PlaceholderAPI is optional and only
needed for the `sncrates` placeholders and for placeholders inside reward win commands.

{% hint style="info" %}
SnCrates is part of the licensed bundle. Your key goes in the shared
`plugins/.Sn-License/license.yml` - see [Installation](installation.md).
{% endhint %}
