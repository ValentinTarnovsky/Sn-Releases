# FAQ

### How do I update SnMiniGames?
Download the newer `snminigames-v*` release and replace the jar. Update SnLib first if the release notes ask for a newer version. Configs auto-merge on restart; the per-game files (`games/parkour.yml`, `games/tntrun.yml`, `games/tnttag.yml`, `games/spleef.yml`) are never touched.

### Why are the map setup commands missing from `/mg help`?
They are hidden from the root help on purpose. Every minigame ships its own set of setup commands, so listing them all would make `/mg help` unreadable once several games are installed. Each game exposes one entry instead, `/mg admin <game> help`, which lists that game's commands on its own page. The commands themselves are unchanged: they still run, and they still tab-complete under `/mg admin <game> `.

### Where do I configure the leave item?
Its material and name live in `config.yml` under `leave-item`, because one item is registered for every minigame and cannot differ between them. Whether a game hands it out and which hotbar slot it uses stay per-game, under `leave-item` in that game's file.

### How do I build a parkour map?
Create it with `/mg admin parkour create <name>`, then get the wand with `/mg admin wand` (running it again removes the wand). `/mg admin parkour help` lists every setup command if you forget one. Click a block and run `/mg admin parkour setstart <map>`, `addwaypoint <map>` or `setwin <map>`. Stand where you want the waiting lobby, end point or hold spot and run `setwaiting <map>`, `setend <map>` or `sethold <map>`.

### How do I build a TNT Run arena?
Create it with `/mg admin tntrun create <name>`. Get the cuboid wand with `/mg admin wand region`, left-click one corner of the arena and right-click the opposite one (the edges are drawn with particles while you select), then run `/mg admin tntrun setregion <map>` (the particles stop once the arena is saved). Stand on the top layer and run `setstart <map>`, stand in the mini-lobby and run `setwaiting <map>`, and set the elimination height with `setelimy <map> <y>`. `/mg admin tntrun help` lists everything else (`setdelay`, `setdepth`, `setwinners`, `settimelimit`, `setend`).

### How do I build a TNT Tag arena?
Create it with `/mg admin tnttag create <name>`. Get the cuboid wand with `/mg admin wand region`, left-click one corner and right-click the opposite one, then run `/mg admin tnttag setregion <map>`. Stand inside the arena and run `setstart <map>` - the start spawn **must be inside the region**, because it is also where an escapee gets sent back to, and a spawn outside it would loop forever (the plugin refuses to open such a round and logs why). Stand in the mini-lobby and run `setwaiting <map>`. `/mg admin tnttag help` lists the rest (`settaggers`, `setroundtime`, `setdecrement`, `setminroundtime`, `setgrace`, `setbreak`, `setwinners`, `settimelimit`, `setend`).

### How do I build a Spleef arena?
Create it with `/mg admin spleef create <name>`. Get the cuboid wand with `/mg admin wand region`, left-click one corner and right-click the opposite one, then run `/mg admin spleef setregion <map>`. Select the snow floor **and the headroom above it**: the region is what may be broken, and it is also the area the melt picks from. Stand on the floor and run `setstart <map>` - the start spawn must be in the region's world and over the arena's footprint, or the plugin refuses to open the round and logs why rather than dropping everyone into the void. Stand in the mini-lobby and run `setwaiting <map>`, and set the elimination height with `setelimy <map> <y>`. `/mg admin spleef help` lists the rest (`setsnowballs`, `setmeltstart`, `setmeltrate`, `setwinners`, `settimelimit`, `setend`).

### The Spleef block breaks, comes back and breaks again. Is my server lagging?
No, and it is fixed in **v1.9.1** - update the jar. Nothing was ever wrong with the world; the client was told twice. Your client draws the break the instant you click and then waits for the server to confirm it, and because the plugin cancelled that click the server answered "the block is still there" a moment before the plugin's own removal arrived. The interact is now left alone for a legitimate dig and the empty block is confirmed to the digger immediately, so the break is drawn once. Snowball and melt breaks never had this - only the block you click yourself is predicted by your client.

### My Spleef shovel does not break anything. Why?
Almost always the floor list. `removable-materials` in `games/spleef.yml` is both what may be broken and what the shovel is allowed to break, so if your floor is not snow you have to put its material there and run `/mg reload`. Clicking a block outside the arena region never does anything either, by design. If the log carries a line about the server refusing the can-destroy restriction, report your server version: without it the client sends no dig packet at all, because players in a round are in adventure mode.

### Can I use a different floor material, or more than one?
Yes. Put every material in `removable-materials` and run `/mg reload`; the shovel is restricted to exactly that list at the start of each round. What you cannot do is leave it empty to mean "anything" - an empty restriction is a shovel that breaks nothing, so an empty or unrecognised list falls back to `SNOW_BLOCK` with a warning.

### Does a Spleef snowball hurt anyone?
No. Nobody in a Spleef round can lose a single heart, from a snowball or from a punch. A snowball only pushes you - the hurt animation and the sound are feedback, not damage - and punching does nothing at all, not even knockback. The push is applied by the plugin rather than by the server, so it feels the same with PvP on and PvP off.

### Why is my Spleef floor disappearing on its own?
That is the anti-camping melt. After `melt-start` seconds (default 120) the arena starts losing `melt-rate` random floor blocks every second, so waiting on a safe island stops working. Set `melt-start: 0` on the map to switch it off completely; the arena is then never scanned at round start either.

### Does TNT Tag damage blocks or players?
Neither. The "explosion" is a particle plus a sound - the plugin never calls the vanilla explosion, so not a single block is touched and nobody takes damage from it. Hitting someone to pass the tag costs no health either: since v1.8.1 the hit is kept but its damage is zeroed, so it still knocks the other player back and plays the hurt animation while no participant ever loses a heart (a second guard re-zeroes the damage after every other plugin has had its say). An exploded player is simply removed from the round with their pre-game state restored.

### Hitting someone does not pass the TNT, or the hit feels like nothing. Is PvP required?
No, not since v1.8.0. It used to be, in effect: with PvP disabled the server refuses a player-on-player hit before any plugin can see it, so the tag never moved. The tag is now also read from the attacker's swing, with the target resolved by a ray trace from their eyes, so it works exactly the same on a world with PvP off. If hits still feel unreliable on a laggy server, raise `tag-reach` in `games/tnttag.yml` a little (default `3.5` blocks, maximum `6`); too high starts counting near-misses. A swing through a wall never counts - blocks stop the trace.

And since v1.8.1 the hit also *lands*: knockback, hurt animation and hurt sound, with the damage zeroed so no health is lost. With PvP on that is the server's own knockback. With PvP off the server applies none, so the plugin reproduces it - tune it with `hit-knockback` (`0.4` is vanilla's strength, `0` disables it) and `sounds.hit`. One caveat: if PvP is blocked by a **region plugin** rather than by the world flag, the server still refuses the hit while the plugin's fallback stays off, so the hit produces no knockback; disable PvP at the world level for the arena world instead.

### Why does the player holding the TNT run faster?
That is `tag-speed` in `games/tnttag.yml`, `1` (Speed I) by default: the chaser is the one who has to catch somebody. Set it to `0` to remove the boost, or up to `5` for a much faster hunter. It is applied when the tag arrives and removed the instant it is passed on, and it can never leak out of a round - the pre-game state restore clears every effect.

### Why is there a pause after the explosion before the next round?
That is the observation break, `round-break` on the map (3 seconds by default). It gives the survivors a moment to see who blew up instead of being tagged again the same tick, and the counter shows "next round in ..." while it runs. Nobody holds the TNT during the break, so it can never eliminate anyone. Set it with `/mg admin tnttag setbreak <map> <seconds>`, or `0` to chain the rounds back to back like earlier versions did.

### Where do players go between rounds?
Back to the map's `start-spawn`, at the moment the next round OPENS. The observation break runs first, where the previous round ended, and then everyone is teleported to the spawn (velocity and fall distance cleared) and the new TNT is handed out - so a chase never carries over and nobody keeps the position they earned in the previous round. Maps with `round-break: 0` are teleported too, they just have no pause in between.

### Nobody explodes in my TNT Tag round, or the counter never appears. Why?
The round most likely never started. TNT Tag refuses to open a round whose map has no region, no start spawn, a start spawn in a different world from the region, or a start spawn outside the region - each case is logged as a warning when the round would have begun. Check `/mg admin tnttag list` to confirm the region is set. If the counter is invisible but the round is running, check `counter-display` in `games/tnttag.yml`: `BOSSBAR` draws a bar at the top of the screen, `ACTIONBAR` draws a line above the hotbar.

### What happens in TNT Tag if the tagged player just logs off?
The tag moves to a random survivor, so a round can never be left with nobody holding it. To keep that fair, if the counter had less than `reassign-grace` seconds left (5 by default) it is raised back to that value, so whoever inherits the TNT gets a real chance to pass it on. Set `reassign-grace: 0` to disable the floor and let the clock run down untouched.

### Can I have more than one player tagged at once in TNT Tag?
Yes, set `taggers` above 1 on the map - every tagged player explodes when the counter hits zero. The number is clamped at runtime to `survivors - winners`, so a round can never blow up its own last survivors and end with no winner; with 3 players left, `winners: 1` and `taggers: 5`, only 2 are tagged.

### The region wand particles will not go away. How do I clear the selection?
Run `/mg admin wand region` again. It takes the wand back and closes the selection, so the edges stop rendering - and it closes the selection even when you dropped or stored the wand, so there is always a way out without relogging. `setregion` also closes the selection by itself once the arena is saved, since it has consumed it. If you would rather have selections expire on their own, set `region-wand.timeout-ticks` in `config.yml` to a number of ticks (`0`, the default, means never).

### Can a player survive by sneaking onto the edge of a block, or by standing perfectly still?
No, and neither costs the arena any extra blocks. A player is 0.6 blocks wide, so the block their body rests on is not always the one under their feet: sneaking at the rim of a hole lets the hitbox hang over the gap while a sliver still sits on the neighbouring block. Each check therefore clears the one column carrying most of the body - normally the column under the player's centre, which is why running leaves a one-block-wide trail even diagonally, and the neighbour holding them up when that centre column is already gone. On top of the movement check, a sweep re-checks every player in the round a few times a second, so a player who stops moving loses the ground under them just the same. A corner is no safer than the middle of a block, and standing still is not a strategy.

### Does TNT Run destroy my world?
No. Every block a round removes is recorded and put back when the round ends - and also when you run `/mg reload` or shut the server down with a round still live. Blocks are removed with a direct block set rather than a `BlockBreakEvent`, so anti-grief and logging plugins never see them. Only blocks inside the configured region can ever vanish, and the region itself is capped by `region-wand.max-volume` in `config.yml` (250,000 blocks by default); `setregion` refuses a bigger selection.

### Nothing disappears when I run around my TNT Run arena. Why?
Three usual causes. The arena `region` is not set or does not actually contain the floor you are standing on - check `/mg admin tntrun list`. The map has a `removable-materials` whitelist that excludes your floor blocks (leave the list empty to allow any block). Or the round never started: the plugin refuses to open a round whose map has no region, no start spawn, a start spawn in a different world from the region, or a start spawn below `elimination-y` - each case is logged as a warning at startup of the round.

### My floor is two layers and only one of them breaks. Why?
That was a bug, fixed in v1.8.4 - update the plugin. Until then the probe decided where to cut the column by looking one block below your feet, which is right only when you are standing on a full block: on anything shallower - a slab, soul sand, a dirt path, farmland, a carpet, mud, honey - your feet are *inside* that block, so the layer beneath it was taken and the surface was left floating. The leftover then broke on its own the next time somebody stepped on it, which is why it looked like "sometimes only the bottom, sometimes only the top". Since v1.8.4 the cut always starts at the block you are walking on and `remove-depth` counts downward from there. Two other things still stop the second layer legitimately: `remove-depth` left at `1` (`/mg admin tntrun setdepth <map> 2`), and a lower layer that is outside the arena `region` or outside the map's `removable-materials` whitelist.

### How do I start a round manually?
Two admin commands cover it. `/mg admin start <game>` opens a fresh queue right away without waiting for auto-start; add an optional map (`/mg admin start <game> <map>`) to force a specific map instead of the rotation pick, and the rotation cursor stays where it was. Once players are queued, `/mg admin forcestart <game>` skips the remaining countdown and begins the round immediately with whoever is in the queue - the minimum-players requirement is bypassed on purpose, so an empty queue is refused instead.

### Why does the [JOIN] chat button do nothing for some players?
Bedrock players (connecting through Geyser) cannot click chat buttons - that is a Bedrock limitation, not a bug. The announce also shows the plain command (`/minigames join <game>`), which works for everyone. If Java players cannot click either, check the startup log: the plugin warns when a language file edit or translation lost the `<click:...>` tag of a message.

### Can players move during the 3-2-1 countdown?
No. Movement is frozen at the start point until the GO title: every move that changes position is refused, and anyone who still drifts is teleported back each second (this also holds for Bedrock players). Looking around stays free. Since v1.8.0 the freeze uses no potion effect at all, so there is no icon and no screen tint during the countdown.

### What happens to a player's items if the server crashes mid-game?
Nothing is lost. The full player state is written to disk before the game touches it, and it restores on the next start or when the player reconnects.

### Can more than one player win?
Yes. In Parkour, set `winners` above 1 on a map: each finisher waits frozen at the `finish-hold` spot until enough players finish or the time limit ends the round. In TNT Run and TNT Tag, `winners` is the number of players still standing that ends the round; all of them are announced as winners. Note that neither of those two has a score to rank survivors by, so with `winners` above 1 their relative positions are stable but not earned - keep the reward difference between those positions small, or leave `winners` at 1.

### Does it support Folia?
No, SnMiniGames targets Paper 1.20.x and 1.21.x.
