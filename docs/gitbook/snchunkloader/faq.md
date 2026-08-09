# FAQ

### How do I update SnChunkLoader?
Download the newer `snchunkloader-v*` release and replace the jar. Configs auto-merge on restart.

### Does it support Folia?
No, SnChunkLoader is not Folia-compatible.

### Do loaded chunks spawn mobs?
No. SnChunkLoader uses Paper plugin chunk tickets, which keep chunks loaded and ticking without a player nearby. Redstone, crops, hoppers and generators keep running, but mob spawning still needs a player in range.

### What happens when a loader runs out of time?
It turns itself off and stays where it is. Nothing is deleted, and the loader keeps its place, its size and its owner. Right-click it with another loader item of the same size to feed it more time, then turn it back on.

### Does the timer run while a loader is off?
No. The lifetime only counts down while the loader is on and actually holding chunks. Pausing a loader from its menu is a real pause.

### How do players add time to a placed loader?
They right-click it while holding another loader item of the same size. The time on the item is added to the loader and the item is consumed. Sizes that do not match are refused, and a refused attempt consumes nothing.

### Is there a ceiling on stored time?
Yes, `chunk-loader.time.max-stack-seconds`. Stacking past it clamps to it, and the player is told the excess was lost. Set it to 0 for no ceiling. The ceiling does not bind an infinite item: feeding a permanent loader item to a placed loader of the same size makes that loader permanent whatever the ceiling says. A loader that is already permanent refuses further time and consumes nothing.

### Who gets the loader when it is broken?
The player who broke it. The gate is island membership, not ownership, so any member of the island can break a loader standing on it and keep the item with its remaining time. Set `chunk-loader.island.require-own-island` to `true` to keep that gate on. With it off, anyone at all can do it.

### What happens to a loader when its owner leaves the island?
With `return-on-membership-loss` enabled, the loader is removed and returned to its owner with its remaining time. This covers kicks, quits, bans and disbands. These returns go to the owner, unlike a normal break, which goes to the breaker.

### What happens if the receiving player's inventory is full, or they are offline?
The loader is queued in the database and handed over on their next join. A loader is never dropped on the ground.

### Can a loader be destroyed by an explosion or pushed by a piston?
No. A placed loader can only be removed by a player breaking it. Explosions skip it, fire and block decay cannot take it, and endermen and falling blocks cannot change it. A piston that would push or pull a loader does not move at all, so a redstone contraption that runs into one silently stops working.

### What happens to loaders that are over the limit after I lower it?
Nothing. The limits are checked when a loader is placed and never again, so lowering `chunk-loader.limits.per-player` or `chunk-loader.limits.per-island` only blocks new placements. Loaders already standing keep running until a player breaks them.

### Do loaders survive a server restart?
Yes. The loaders themselves are stored in the database, and their chunk tickets are re-applied on boot. The re-apply is spread across ticks, so a server with many loaders does not freeze while it starts.

### Can I change the loader block after loaders exist?
Avoid it. A placed loader is recognised by its block material. Changing `chunk-loader.material` makes every existing loader unrecognisable, so the reconciliation sweep unplaces them and owes each item back to its owner, re-minted in the new material. Loader items already handed out are the real cost: they carry the old material, placing one is refused, and nothing refunds them. Every loader in an inventory, a chest or an ender chest at that moment becomes unplaceable for good.

### Which database does it use?
SQLite by default, with no configuration needed. Set `database.type` to `mysql` and fill in the connection block to use MySQL instead.

### Can I use my own hologram plugin?
You can pick between SnLib's built-in displays and DecentHolograms with `hologram.provider`. If you select `decentholograms` and that plugin is not installed, SnChunkLoader falls back to `snlib` on its own. A display keeps the backend and the height it was created with, so changing `hologram.provider` or `hologram.offset-y` and running `/chunkloader reload` only affects displays created after it. Restart the server to move every existing display onto the new setting.

### Does it add PlaceholderAPI placeholders?
No. SnChunkLoader provides no placeholders. It does resolve PAPI tokens that you write into its own language, menu and hologram files.
