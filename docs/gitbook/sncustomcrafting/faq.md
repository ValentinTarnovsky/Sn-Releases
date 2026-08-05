# FAQ

### How do I update SnCustomCrafting?

Download the newer `sncustomcrafting-v*` release and replace the jar. Configs auto-merge on restart, so your values and comments are preserved.

### Does it support Folia?

No, SnCustomCrafting is not Folia-compatible. It targets Paper 1.20.x and 1.21.x.

### Why does a recipe not appear in the menu?

Check the console on boot. A recipe is skipped when it has no `display` block, when its `slot` falls outside the menu, or when one of its requirements names an item id that `items.yml` does not declare. Each case prints a line saying which recipe and why.

Two recipes configured on the same slot are also a cause: the later one in the file wins and the earlier one is invisible.

### Why does a vanilla item not count as an ingredient?

Because it is not one. Recipes consume items **this plugin creates**, and those carry a signature only the plugin can write. A vanilla stack with the same material and the same name never matches. Hand out the real thing with `/customcrafting give <player> <item> [amount]`.

### Can a player run several crafts at once?

Yes, one per workstation. A player with a craft running on a workstation who right-clicks it again is told how long is left rather than being charged a second time, but they can start another craft on a different workstation.

### Does the timer run while the player is offline?

No. Craft time only counts down while the player is online, and crafts survive a restart, so a craft picks up exactly where it left off.

### A workstation block stopped responding. What happened?

Most likely it was registered twice. One block holds one workstation, so registering a second id on an occupied block unregisters the first. Run `/customcrafting set <id>` again on that block to re-point it.

### Can I stop workstation blocks from being protected?

Yes. Set `settings.protect-workstation-blocks: false` in `config.yml`. Protection covers breaking, explosions, fire and pistons; other ways a block can vanish, such as an enderman or a world edit, are not intercepted.

### How do I move the crafts to MySQL?

Set `database.type: mysql` in `config.yml`, fill in the connection block, and restart. Crafts are then shared by every server pointed at the same database. The SQLite default needs no setup.

### The plugin refuses to enable. What do I check?

Two things, and the console names which one. Either SnLib is missing or older than 1.24.0, or the license key in `plugins/.Sn-License/license.yml` is missing or invalid. The key is checked once at startup, and the plugin needs outbound internet access at that moment.
