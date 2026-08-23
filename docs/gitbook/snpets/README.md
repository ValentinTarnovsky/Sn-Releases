# SnPets

SnPets gives every player a squad of floating pets that follow them on an arc formation, cut
from a circle or from an oval that hugs their back. The formation rotates with the owner's
camera, so the pets always stay in view.
Pets are drawn with packets instead of entities, so they never collide, never despawn and
never wander off. The whole server shares one animation tick, so pet count adds no scheduler
tasks and unequipped pets cost nothing.

A pet is more than a cosmetic. It levels up from a configurable experience source, carries a
rollable trait and graded stat boosts, and grants its owner damage, resistance and speed
buffs that scale with its level. Players collect pets from weighted pet boxes, fuse
duplicates into stronger ones, and manage everything from seven configurable menus.

## Features

- Packet arc formation, circle or oval, with configurable radius, back radius, arc width,
  height and bob.
- Levels and experience per pet type, with per-type level caps.
- Damage, resistance and speed buffs, each with a global cap and a per-world gate.
- Rollable traits and graded stat boosts, bought with trait tickets and dice.
- Fusion: combine duplicates into a stronger pet that inherits levels, traits and boosts, with
  a server-wide announcement you can switch on per pet.
- Pet boxes with weighted tables, a per-box success chance stamped on the item, bulk opening,
  cooldowns and a full refund on a failed save.
- Pet items: shift + right click a stored pet, or press Q over it, to take it out as a physical
  head carrying its whole state, and let anyone right click it back into their own storage. That
  is how players trade pets.
- Holograms: configurable text above every pet, per pet type, drawn as packet entities that
  RIDE the pet, so the text and the pet can never move at different times.
- Seven menus you can re-skin without touching code, with a per-player switch that decides
  whether that player's roll animation plays or the result appears instantly, and a per-group
  colour every pet template can draw with.
- Equipped pets are kept in slots 1..n with no gap, so the free slots are always the last ones.

## Optional integrations

- **PlaceholderAPI**: registers the `%snpets_...%` expansion for scoreboards, tab and holograms.
  Without it the plugin runs normally and simply skips the expansion.
- **BetterModel**: lets a pet type render as an animated model with separate idle and moving
  animations. Without it every pet renders as a player head, which is the built-in default.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
