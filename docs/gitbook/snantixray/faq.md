# FAQ

### How do I update SnAntiXray?

Download the newer `snantixray-v*` release and replace the jar. Configs auto-merge on restart, so
your values and comments are preserved.

### Does it support Folia?

Yes, SnAntiXray is Folia-compatible. Three thread keys are ignored there because Folia's region
scheduler replaces those pools: `threads.fake-ore-generator`, `threads.ghost-block-ticker` and
`threads.async-block-checks`. The startup log names them so you know why tuning them changed
nothing. `threads.outgoing-packet-handler` still applies.

### Does this modify my world?

No. SnAntiXray never writes a block, never saves anything to a region file and never changes a
chunk on disk. It rewrites the chunk packet on its way to one player. Uninstalling it returns every
player to the true world immediately.

### Will players see ore disappear or reappear?

No. A hidden block becomes real the moment a player legitimately exposes it, and that decision is
remembered for the rest of their session. Chunks resent later still carry the real block.

### Why does my server refuse to start after installing this?

Check that PacketEvents is installed. It is a hard dependency on purpose, because the alternative is
a server that starts happily with every protection layer silently switched off.

### Detection never flags anyone. Is it broken?

Check `worlds.yml` first. Decoy veins are the only signal the detection layer scores, so detection
is inert when no world carries a usable rule. Two defaults switch things off silently:

- a world without `enabled: true` receives no veins
- a rule without a `frequency` above zero generates nothing

The startup log says so when it happens. Read it after editing the file.

### Can it ban or kick cheaters automatically?

No, and that is deliberate. SnAntiXray raises alerts and stops there. Your staff decide what to do,
so a false positive costs someone a look instead of a wrongful ban.

### A staff member cannot see the alerts. Why?

Two things gate an alert. The staff member needs the `snantixray.alerts` permission, and their
personal preference must be on. `/antixray alerts` toggles the preference. If they turn it on
without the permission, the plugin tells them so rather than pretending it worked.

### Should I put decoy diamonds at any height I like?

No. Mirror vanilla ore distribution. Diamonds at Y=-50 read as natural terrain, but diamonds at Y=80
tell even an honest player who stumbles on one that the placement is wrong. That teaches cheaters
which veins are bait and blows the cover of the whole layer.

### Does it cost TPS?

The chunk rewrite runs on the network thread that carries the packet, not on the server tick. Blocks
that become exposed are sent under a per-player per-tick budget, so a large explosion spreads its
updates over several ticks instead of spiking one. Use `/antixray stats` to see what the engine is
actually doing on your server.

### How do I let a staff member see the real world?

Run `/antixray bypass <player>`, or grant them `snantixray.bypass`. Either one makes all three layers
skip them. Note that `snantixray.admin` deliberately does not include it, so admins see what players
see until you say otherwise.
