# SnCustomCrafting

SnCustomCrafting turns ordinary blocks into timed crafting workstations. Register a block in the world, and players right-click it to pick a recipe, spend the ingredients and wait out a timer before claiming the rewards.

## Features

- **Workstations anywhere.** Point at any block and register it with one command. Each one is bound to a recipe set, so a forge and an alchemy table can offer completely different menus.
- **Timed crafts that survive restarts.** Every craft rolls its own duration between the recipe's minimum and maximum, counts down only while the player is online, and picks up where it left off after a reboot.
- **Ingredients that cannot be faked.** Recipes consume items this plugin creates, signed so a renamed vanilla lookalike never counts. An anvil cannot forge one, and neither can a shop.
- **Reward commands, not just items.** A finished craft runs any console commands you configure, so a recipe can pay out items, money, permissions or a crate key.
- **Protected blocks.** Registered workstations cannot be broken, blown up, burnt or pushed by a piston.
- **Per-set menus in YAML.** The menu layout, the border and the title live in a file per recipe set. Adding a recipe never means editing the menu.

## Optional integrations

- **PlaceholderAPI**: unlocks the status, remaining-time and progress placeholders, so a scoreboard or tab list can show what each player is crafting. Without it the plugin works normally and simply registers no placeholders.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
