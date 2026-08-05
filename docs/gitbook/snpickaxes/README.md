# SnPickaxes

SnPickaxes adds two configurable custom pickaxes to your Paper server. One breaks a cube of blocks around the block you mine. The other multiplies the drops of everything it breaks. Both respect region protection on a per-block basis, so a protected block inside a cube is skipped instead of blocking the whole swing.

## Features

- Two independent pickaxes: an area miner and a drop duplicator, each with its own master switch.
- Area size configurable to any odd value, bounded by a `max-size` safety cap.
- Duplication amount configurable, with copies sent to the inventory or dropped on the ground.
- Optional omnitool mode: the pickaxe morphs in hand into the axe, shovel, hoe or pickaxe that matches the block.
- Optional per-pickaxe WorldGuard region restrictions, in whitelist or blacklist mode, with an independent list each.
- Real vanilla breaks, so drops, experience, durability and statistics behave normally, and Fortune and Silk Touch apply.
- Full appearance control per pickaxe in `items.yml`: material, name, lore, model data, glow, enchantments and more.

## Optional integrations

- **WorldGuard**: unlocks the per-pickaxe region restrictions, which limit where each special ability works. Without WorldGuard the restriction block is ignored and both pickaxes work everywhere.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
