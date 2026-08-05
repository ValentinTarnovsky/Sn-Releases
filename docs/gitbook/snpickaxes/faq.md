# FAQ

### How do I update SnPickaxes?
Download the newer `snpickaxes-v*` release and replace the jar. Configs auto-merge on restart, so your values and comments are preserved.

### Does it support Folia?
No, SnPickaxes is not Folia-compatible. It runs on the main thread by design, because the drops of a cube must materialise in the same tick as the swing.

### Do the pickaxes break protected blocks?
No. Every block in a cube is checked individually against the same protection your server already uses. WorldGuard, Lands, GriefPrevention and any plugin that cancels `BlockBreakEvent` are all respected. A protected block is skipped and the rest of the cube still breaks.

### Do I need WorldGuard?
Only for the per-pickaxe region restrictions, which limit where each ability works. Without WorldGuard the pickaxes work everywhere, and per-block protection from any other plugin still applies.

### Why is my cube smaller than the size I configured?
The `size` value is clamped to `max-size`, and an even number is rounded up to the next odd one. Both corrections are logged as a warning on boot. Raise `max-size` if you want bigger cubes.

### Can the Duplicator copy a full shulker box?
No. A block that carries contents is never duplicated, including a shulker box with items and a bee-filled hive. Empty containers duplicate normally.

### The auto-pickup on my server swallows the duplicated drops.
Set `give-to-inventory: true` so the copies go straight to the miner's inventory. If your core suppresses drops without cancelling the break, also keep `duplicate-when-drops-suppressed: true`.

### Can I add a third pickaxe?
No. Exactly two ids exist, `area` and `duplicator`. Adding another section to `config.yml` or `items.yml` does nothing.

### The lore on my pickaxes does not match the config anymore.
Placeholders resolve once, when the item is created. Changing `size` or `extra-drops` changes the behaviour immediately, but existing items keep their original text. Re-give them with `/snpickaxes give`.

### Should I change `block-break-event-priority`?
Only if a server core cancels the break to auto-collect. At any value below `MONITOR` the handler can run before the protection plugins, so the Duplicator hands out copies for breaks those plugins then refuse. Use a lower value only on a server with no block protection.
