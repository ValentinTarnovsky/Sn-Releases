# SnCompanions

SnCompanions gives every player a squad of companions that follow them in a straight line behind
them, at their feet - or, one key away, on an arc cut from a circle or from an oval that hugs their
back. The formation rotates with the owner's camera, so the companions always stay in view.
Companions are drawn with packets instead of entities, so they never collide, never despawn and
never wander off. The whole server shares one animation tick, so companion count adds no scheduler
tasks and unequipped companions cost nothing.

A companion is more than a cosmetic. It levels up from a configurable experience source and
grants its owner damage, resistance and speed
buffs that scale with its level. Players collect companions from weighted companion eggs, fuse
duplicates into stronger ones, and manage everything from configurable menus.

## Features

- Packet formation - a straight line behind the owner, or an arc cut from a circle or an oval -
  with configurable spacing, distance, radius, back radius, arc width, height and bob.
- Levels and experience per companion type, with per-type level caps and a choice of six experience
  sources: vanilla XP, blocks, EdTools omnitool breaks, mobs, damage dealt or playtime. Since
  1.10.0 a companion is created at level 0, so its first level is earned like every level after it.
- Damage, resistance and speed buffs, each with a global cap and a per-world gate.
- Fusion: combine duplicates into a stronger companion that inherits levels, with
  a server-wide announcement you can switch on per companion.
- Companion eggs with weighted tables, price tiers charged in Vault, in one of your EdTools
  currencies, or in several at once, a Dragon Egg hatch show and per-egg cooldowns. An egg is a purchase, not an item: the table
  rolls the moment it is opened and the companions land straight in the owner's storage. A missing
  economy refuses the purchase instead of giving the egg away, and a purchase whose companions
  cannot be saved is refunded in full.
- An in-game eggs shop, `/companions eggs [egg]` or the egg button of the storage menu: every
  companion the egg can give with its live odds, and one button per price tier. One menu file
  serves every egg you declare.
- A hatch show on every open: a Dragon Egg appears in front of the buyer, shakes faster and
  faster, breaks and reveals the companion they won. Drawn with packets like the companions
  themselves, so nothing is spawned and nothing can be walked into - and any player who would
  rather not watch it switches it off for themselves from the eggs menu.
- Companion items, **off by default since 1.11.0**: switched on, shift + right click a stored
  companion, or press Q over it, to take it out as a physical head carrying its whole state, and
  right click it back into a storage. The head remembers who took it out, and only that player can
  redeem it, so moving a companion to another owner is an admin action. Left off, a companion never
  leaves its owner's storage as an object.
- Holograms: configurable text above every companion, per companion type, drawn as packet entities that
  RIDE the companion, so the text and the companion can never move at different times.
- A Bedrock fallback, **on by default since 1.12.0**: a player on Bedrock is sent a fake baby zombie
  wearing that companion's own head plus leather armour, moved by the same arithmetic as the real
  companion and carrying the same name plate, because the Display entity every companion body is
  drawn with has no Bedrock definition at all and Geyser drops it without a word. The head is what
  those players see whichever backend the companion uses, so a companion that renders as a
  BetterModel model reaches them as its head rather than as the animated model - give every companion
  a head texture if your server has Bedrock players. Java players receive byte for byte what they
  received before and see no change whatsoever. Floodgate is optional, and is what tells the plugin
  who is on Bedrock.
- Four menus you can re-skin without touching code, with a per-group
  colour every companion template can draw with.
- Equipped companions are kept in slots 1..n with no gap, so the free slots are always the last ones.
- A developer API in the same jar: a read-only query service plus eight Bukkit events, five of them
  cancellable.

## Optional integrations

- **PlaceholderAPI**: registers the `%sncompanions_...%` expansion for scoreboards, tab and holograms.
  Without it the plugin runs normally and simply skips the expansion.
- **BetterModel**: lets a companion type render as an animated model with separate idle and moving
  animations. Without it every companion renders as a player head, which is the built-in default.
- **Floodgate**: tells the plugin which players are on Bedrock, so those players can be sent the
  baby-zombie substitute instead of a Display entity their client cannot be shown. It is asked once
  per player, on their join, and never from the animation tick. Without Floodgate every player
  counts as a Java client and the whole `bedrock` band of `config.yml` does nothing - on a Geyser
  server that means Bedrock players keep seeing nothing where their companions are. Installing or
  removing it on a running server needs no restart: the plugin re-asks every online player and
  rebuilds every formation one tick later.
- **EdTools**: lets an equipped companion grant EdTools currency boosters and the global enchant
  multiplier, summed across every equipped companion,
  with an optional per-companion `max:` ceiling on each entry. It also unlocks the `EDTOOLS_BLOCK_BREAK`
  experience source, so a companion can level from the blocks
  an EdTools omnitool consumes - blocks a vanilla `BlockBreakEvent` never sees. Without it nothing
  is granted, that source earns nothing, no EdTools class is ever loaded and the plugin runs
  normally.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
