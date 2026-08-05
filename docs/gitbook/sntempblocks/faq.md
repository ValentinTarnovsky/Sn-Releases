# FAQ

### How do I update SnTempBlocks?

Download the newer `sntempblocks-v*` release and replace the jar. Configs auto-merge on restart,
and `zones.yml` is never touched.

### Does it support Folia?

No, SnTempBlocks is not Folia-compatible. Its tracking index is single-threaded main-thread state
and its region queries go through WorldGuard.

### Do blocks keep expiring while the server is off?

Yes. Lifetimes run on real time, not on server ticks. A block whose lifetime elapsed during
downtime is removed at startup, and an interval cycle neither restarts nor skips.

### Does a reload wipe what is already tracked?

No. A reload re-reads `config.yml` and `zones.yml` and re-applies the settings. The tracked
blocks are runtime state, so they survive. Clearing them would make every block already placed in
every zone permanent.

### What happens to blocks whose zone I delete from zones.yml?

That is `tracking.on-zone-removed`. `EXPIRE_NOW` removes them on the following sweeps, which
leaves no orphans. `KEEP` stops tracking them and leaves them in the world permanently. The
policy applies whether you edited the file live and reloaded, or with the server stopped.

### Why is my zone disabled in the console?

Its WorldGuard region or its world does not exist. Check the spelling of `world` and `region`
against WorldGuard. A zone in that state is disabled on its own, and every other zone keeps
working.

### Why did nothing get tracked in my zone?

Three usual reasons. The zone's lifetime for that material is `0`, which means "do not track this
material". The player has `/tempblocks bypass` switched on. Or the block was not placed by a
player: SnTempBlocks tracks `BlockPlaceEvent` and bucket placement only, so world generation,
mobs, dispensers and other plugins are out of scope.

Since 1.1.0 the bypass is a switch a staff member turns on, never something a permission does on
its own, so an operator testing a zone sees their blocks tracked normally.

### Does it track water and lava?

Yes, when a player empties a bucket inside a zone. The plugin follows the body as it spreads and
removes the source together with every block the flow reached, in one pass. Spread it could not
clean up later is cancelled rather than allowed: flow leaving the zone, and flow past
`limits.max-flow-blocks`.

### Water expired but my fence is still there. Is that a bug?

No, that is deliberate. When water only waterlogs a block, such as a stair, slab or fence, expiry
clears the waterlogged flag and leaves the block alone. That block belongs to whoever placed it,
and deleting it would be griefing.

### Will a big wipe lag my server?

No. Removals are capped by `limits.max-removals-per-tick` and spread across ticks, and that
applies to scheduled wipes, `/tempblocks wipe` and deleted zones alike. Raise the cap if you want
wipes to finish faster.

### What happens to blocks in an unloaded chunk?

They wait. SnTempBlocks never force-loads a chunk to delete a block, because that would drag
chunk generation and entity ticking into an expiry tick for a block no player is near. The
removal runs the moment the server loads that chunk. `/tempblocks info <zone>` shows how many are
waiting.

### Can I use something other than WorldGuard for zones?

No. Zones are WorldGuard regions, which is why WorldGuard is a hard dependency. Use
`region: __global__` if you want a whole world rather than a region.

### Does it drop items when a block expires?

No. An expired block is removed silently, and nothing is dropped or restored.
