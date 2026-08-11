# SnKits

SnKits is a kits plugin for Paper, built to be run from the menus and edited in game. Every kit
lives in its own file, every menu is a YAML file you can add to, and nothing about a kit needs a
text editor. `/kit edit <kit>` opens the whole thing.

## Features

- **Edit kits in game.** A 6 row grid holds the kit contents at the exact position the preview
  shows them. Drag items in from your own inventory and close the grid to save.
- **Kit items keep everything.** Items are stored with Bukkit's own serialization, so
  enchantments, custom names, lore, custom model data, leather colors, skull textures and shulker
  contents all survive a round trip.
- **Command items.** An item that is never given, but runs commands when the kit is claimed, as
  the console or as the player. It can have a display item in the preview, or none at all.
- **Menus are yours.** Any file you drop in `guis/` becomes a menu you can open by name. Each kit
  declares four icons, one per state, and the plugin paints the one that matches the viewer.
- **Multi-claim.** A player can own several copies of the same kit, one extra claim each. The
  amount lives in a permission, so a shop sells a copy with one LuckPerms command.
- **Auto-claim on join.** Hand kits out on first join or on every join, through the same claim
  path as everything else.
- **A claim never destroys anything.** It does not overwrite worn armor or a held item, and with
  `drop-items-when-full: false` it refuses rather than dropping the overflow on the ground.

## Optional integrations

- **PlaceholderAPI**: resolves `%snkits_*%` placeholders anywhere on the server, and lets kit item
  names and lore render per player. Without it the plugin is fully functional, and its own
  `{cooldown}`, `{uses_left}` and `{uses_max}` tokens still work inside SnKits menus and messages.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
