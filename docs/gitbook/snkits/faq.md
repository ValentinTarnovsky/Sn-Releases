# FAQ

### How do I update SnKits?

Download the newer `snkits-v*` release and replace the jar. Your `config.yml`, language file and
menus auto-merge on restart, and your kits are untouched.

### Does it support Folia?

No, SnKits is not Folia compatible. It runs on Paper 1.20.4 and newer, on both 1.20.x and 1.21.x.

### Why does the plugin not appear in `/plugins`?

Two usual causes. Either `SnLib.jar` is missing from `plugins/`, because SnKits declares
`depend: [SnLib]` and will not enable without it, or the bundle key in
`plugins/.Sn-License/license.yml` is still the placeholder. Both are reported in the server log at
startup.

### How do I add a kit?

`/kit create <id>` builds one from your current inventory. Then give it a free letter in the layout
of `guis/kits-main.yml` and copy the four `kit-default-*` template entries, renamed to your kit id.
See [Configuration](configuration.md).

### Why is my new kit missing from the menu?

A kit exists as a file, and a menu draws it only when the menu has entries for it. Add the four
`kit-<id>-*` templates and a layout letter, then run `/kit reload`.

### Why is a kit showing but the icon looks wrong?

Each kit declares four icons, one per state. You are seeing the state the viewer is actually in:
available, on cooldown, one-time used, or disabled. Check which of the four entries you edited.

### How do I sell a kit?

Grant `snkits.kit.<id>` and set `requires-permission: true` on the kit. To sell extra copies of the
same kit, enable `multi-claim` and grant `snkits.uses.<id>.<n>`. The highest amount wins, so a
shop never has to unset the previous node.

### Why does nothing happen when I grant `snkits.uses.<id>.<n>`?

`multi-claim.enabled` is false by default. While it is false, no permission of that shape is read
at all, and every player gets exactly one claim per cooldown.

### Does a claim overwrite the armor a player is wearing?

Never. `auto-armor` and `auto-offhand` only fill slots that are empty. If the helmet slot is taken,
the kit's helmet goes to the inventory instead.

### What happens when a player claims with a full inventory?

It depends on the kit. With `drop-items-when-full: true` the overflow drops on the ground. With
`false` the claim is refused instead, and the refusal costs nothing: no cooldown spent, no one-time
claim spent.

### How do I make an item run a command instead of being given?

Open `/kit edit <kit>`, then the items button, then click slot 53. Right-click the new item in the
grid to add commands and choose whether each runs as the console or as the player. A command item
with no display item is invisible in the preview and still runs.

### How do I set a kit's off-hand item?

Shift-right-click the item in the items editor. A kit has at most one, and shift-right-clicking it
again unsets it. Shields and totems still auto-equip by material.

### Does closing the editor with Escape lose my changes?

No. Closing the grid is what saves it, Escape included. The only thing that discards an edit is
deleting the kit while its editor is open.

### How do I reset a season?

Set `auto-claim.mode` to `first-join`, then run `/kit reset all confirm` when the season starts.
Every player's usage is cleared, so the kit is handed out again on each player's next join.
`/kit reset <player> *` re-arms a single player.

### I use PlayerKits2. Can I bring my kits over?

`/kit import` reads `plugins/PlayerKits2/kits/*.yml` and converts items, cooldowns, the one-time
and permission flags, and claim actions that run commands. Existing kits are left alone unless you
pass `/kit import overwrite`. A file that fails to convert is reported and skipped, and anything
PlayerKits2 supports that SnKits does not is named in the output rather than silently dropped.

Imported kits default to `requires-permission: false` and `auto-armor: false`. Open `/kit edit`
afterwards if you want them on.

### Do I need PlaceholderAPI?

No. Everything works without it. With it installed, `%snkits_*%` placeholders resolve anywhere on
the server and kit item names and lore render per player. See [Placeholders](placeholders.md).

### Why does my hex color show up as text?

Use `&#RRGGBB`. A bare `#RRGGBB` is not recognized as a color and renders literally.

### Can I have more than one page of kits?

Yes. There is no pagination built in: a second page is another file in `guis/` with buttons linking
both ways. That is what `guis/kits-more.yml` is. Chain as many as you like.

### Where is player data stored?

In the database configured under `database` in `config.yml`. SQLite by default, with no setup, or
MySQL. It holds cooldown timestamps and spent one-time claims, and nothing else.
