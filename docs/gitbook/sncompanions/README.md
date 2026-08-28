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
  sources: vanilla XP, blocks, EdTools omnitool breaks, mobs, damage dealt or playtime.
- Damage, resistance and speed buffs, each with a global cap and a per-world gate.
- Fusion: combine duplicates into a stronger companion that inherits levels, with
  a server-wide announcement you can switch on per companion.
- Companion eggs with weighted tables, price tiers charged in Vault or in one of your EdTools
  currencies, a hatch animation and per-egg cooldowns. An egg is a purchase, not an item: the table
  rolls the moment it is opened and the companions land straight in the owner's storage. A missing
  economy refuses the purchase instead of giving the egg away, and a purchase whose companions
  cannot be saved is refunded in full.
- An in-game eggs shop, `/companions eggs [egg]` or the egg button of the storage menu: every
  companion the egg can give with its live odds, and one button per price tier. One menu file
  serves every egg you declare.
- Companion items: shift + right click a stored companion, or press Q over it, to take it out as a physical
  head carrying its whole state, and let anyone right click it back into their own storage. That
  is how players trade companions. Since 1.17.0 the head also remembers who took it out, and only that
  player can redeem it.
- Holograms: configurable text above every companion, per companion type, drawn as packet entities that
  RIDE the companion, so the text and the companion can never move at different times.
- Four menus you can re-skin without touching code, with a per-group
  colour every companion template can draw with.
- Equipped companions are kept in slots 1..n with no gap, so the free slots are always the last ones.

## Optional integrations

- **PlaceholderAPI**: registers the `%sncompanions_...%` expansion for scoreboards, tab and holograms.
  Without it the plugin runs normally and simply skips the expansion.
- **BetterModel**: lets a companion type render as an animated model with separate idle and moving
  animations. Without it every companion renders as a player head, which is the built-in default.
- **EdTools**: lets an equipped companion grant EdTools currency boosters and the global enchant
  multiplier, summed across every equipped companion,
  with an optional per-companion `max:` ceiling on each entry. It also unlocks the `EDTOOLS_BLOCK_BREAK`
  experience source, so a companion can level from the blocks
  an EdTools omnitool consumes - blocks a vanilla `BlockBreakEvent` never sees. Without it nothing
  is granted, that source earns nothing, no EdTools class is ever loaded and the plugin runs
  normally.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
