# FAQ

### How do I update SnMiniGames?
Download the newer `snminigames-v*` release and replace the jar. Update SnLib first if the release notes ask for a newer version. Configs auto-merge on restart; the per-game files (`games/parkour.yml`, `games/tntrun.yml`) are never touched.

### Why are the map setup commands missing from `/mg help`?
They are hidden from the root help on purpose. Every minigame ships its own set of setup commands, so listing them all would make `/mg help` unreadable once several games are installed. Each game exposes one entry instead, `/mg admin <game> help`, which lists that game's commands on its own page. The commands themselves are unchanged: they still run, and they still tab-complete under `/mg admin <game> `.

### Where do I configure the leave item?
Its material and name live in `config.yml` under `leave-item`, because one item is registered for every minigame and cannot differ between them. Whether a game hands it out and which hotbar slot it uses stay per-game, under `leave-item` in that game's file.

### How do I build a parkour map?
Create it with `/mg admin parkour create <name>`, then get the wand with `/mg admin wand` (running it again removes the wand). `/mg admin parkour help` lists every setup command if you forget one. Click a block and run `/mg admin parkour setstart <map>`, `addwaypoint <map>` or `setwin <map>`. Stand where you want the waiting lobby, end point or hold spot and run `setwaiting <map>`, `setend <map>` or `sethold <map>`.

### How do I build a TNT Run arena?
Create it with `/mg admin tntrun create <name>`. Get the cuboid wand with `/mg admin wand region`, left-click one corner of the arena and right-click the opposite one (the edges are drawn with particles while you select), then run `/mg admin tntrun setregion <map>`. Stand on the top layer and run `setstart <map>`, stand in the mini-lobby and run `setwaiting <map>`, and set the elimination height with `setelimy <map> <y>`. `/mg admin tntrun help` lists everything else (`setdelay`, `setdepth`, `setwinners`, `settimelimit`, `setend`).

### Does TNT Run destroy my world?
No. Every block a round removes is recorded and put back when the round ends - and also when you run `/mg reload` or shut the server down with a round still live. Blocks are removed with a direct block set rather than a `BlockBreakEvent`, so anti-grief and logging plugins never see them. Only blocks inside the configured region can ever vanish, and the region itself is capped by `region-wand.max-volume` in `config.yml` (250,000 blocks by default); `setregion` refuses a bigger selection.

### Nothing disappears when I run around my TNT Run arena. Why?
Three usual causes. The arena `region` is not set or does not actually contain the floor you are standing on - check `/mg admin tntrun list`. The map has a `removable-materials` whitelist that excludes your floor blocks (leave the list empty to allow any block). Or the round never started: the plugin refuses to open a round whose map has no region, no start spawn, a start spawn in a different world from the region, or a start spawn below `elimination-y` - each case is logged as a warning at startup of the round.

### How do I start a round manually?
Two admin commands cover it. `/mg admin start <game>` opens a fresh queue right away without waiting for auto-start; add an optional map (`/mg admin start <game> <map>`) to force a specific map instead of the rotation pick, and the rotation cursor stays where it was. Once players are queued, `/mg admin forcestart <game>` skips the remaining countdown and begins the round immediately with whoever is in the queue - the minimum-players requirement is bypassed on purpose, so an empty queue is refused instead.

### Why does the [JOIN] chat button do nothing for some players?
Bedrock players (connecting through Geyser) cannot click chat buttons - that is a Bedrock limitation, not a bug. The announce also shows the plain command (`/minigames join <game>`), which works for everyone. If Java players cannot click either, check the startup log: the plugin warns when a language file edit or translation lost the `<click:...>` tag of a message.

### Can players move during the 3-2-1 countdown?
No. Movement is frozen at the start point until the GO title: position changes are reverted, walking is zeroed and anyone who still drifts is teleported back each second (this also holds for Bedrock players). Looking around stays free.

### What happens to a player's items if the server crashes mid-game?
Nothing is lost. The full player state is written to disk before the game touches it, and it restores on the next start or when the player reconnects.

### Can more than one player win?
Yes. In Parkour, set `winners` above 1 on a map: each finisher waits frozen at the `finish-hold` spot until enough players finish or the time limit ends the round. In TNT Run, `winners` is the number of players still standing that ends the round; all of them are announced as winners. Note that TNT Run has no score to rank survivors by, so with `winners` above 1 their relative positions are stable but not earned - keep the reward difference between those positions small, or leave `winners` at 1.

### Does it support Folia?
No, SnMiniGames targets Paper 1.20.x and 1.21.x.
