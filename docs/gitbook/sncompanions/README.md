# SnCompanions

SnCompanions gives every player a squad of floating companions that follow them on an arc formation, cut
from a circle or from an oval that hugs their back. The formation rotates with the owner's
camera, so the companions always stay in view.
Companions are drawn with packets instead of entities, so they never collide, never despawn and
never wander off. The whole server shares one animation tick, so companion count adds no scheduler
tasks and unequipped companions cost nothing.

A companion is more than a cosmetic. It levels up from a configurable experience source, carries a
rollable trait and graded stat boosts, and grants its owner damage, resistance and speed
buffs that scale with its level. Players collect companions from weighted companion boxes, fuse
duplicates into stronger ones, and manage everything from seven configurable menus.

## Features

- Packet arc formation, circle or oval, with configurable radius, back radius, arc width,
  height and bob.
- Levels and experience per companion type, with per-type level caps and a choice of six experience
  sources: vanilla XP, blocks, EdTools omnitool breaks, mobs, damage dealt or playtime.
- Damage, resistance and speed buffs, each with a global cap and a per-world gate.
- Rollable traits and graded stat boosts, bought with trait tickets and dice.
- Fusion: combine duplicates into a stronger companion that inherits levels, traits and boosts, with
  a server-wide announcement you can switch on per companion.
- Companion boxes with weighted tables, a per-box success chance stamped on the item, bulk opening,
  cooldowns and a full refund on a failed save.
- Companion items: shift + right click a stored companion, or press Q over it, to take it out as a physical
  head carrying its whole state, and let anyone right click it back into their own storage. That
  is how players trade companions. Since 1.17.0 the head also remembers who took it out, and only that
  player can redeem it.
- Scrolls (1.18.0): three consumable items an admin hands out with `/companions admin givescroll`, each
  with its own look and its own list of companions it accepts. The OWNERSHIP scroll drops onto a companion item
  in your own inventory and makes that head yours, which is the way out of the 1.17.0 owner lock.
  Since 1.19.0 the LEVEL and RARITY scrolls are applied by holding one and clicking a companion in the
  main menu - an equipped one or a stored one: the level scroll raises the companion by the number
  stamped on the stack, the rarity scroll turns it into whatever its `upgrades-to` names, and both
  are consumed one per use.
- Holograms: configurable text above every companion, per companion type, drawn as packet entities that
  RIDE the companion, so the text and the companion can never move at different times.
- Seven menus you can re-skin without touching code, with a per-player switch that decides
  whether that player's roll animation plays or the result appears instantly, and a per-group
  colour every companion template can draw with.
- Equipped companions are kept in slots 1..n with no gap, so the free slots are always the last ones.

## Optional integrations

- **PlaceholderAPI**: registers the `%sncompanions_...%` expansion for scoreboards, tab and holograms.
  Without it the plugin runs normally and simply skips the expansion.
- **BetterModel**: lets a companion type render as an animated model with separate idle and moving
  animations. Without it every companion renders as a player head, which is the built-in default.
- **EdTools**: lets an equipped companion grant EdTools currency boosters and the global enchant
  multiplier, summed across every equipped companion and widened by that companion's boost grade and trait,
  with an optional per-companion `max:` ceiling on each entry. A TRAIT can grant them too, as flat
  percentage points that any equipped companion carrying it pays whether or not the companion declares
  boosters of its own. It also unlocks the `EDTOOLS_BLOCK_BREAK`
  experience source, so a companion can level from the blocks
  an EdTools omnitool consumes - blocks a vanilla `BlockBreakEvent` never sees. Without it nothing
  is granted, that source earns nothing, no EdTools class is ever loaded and the plugin runs
  normally.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
