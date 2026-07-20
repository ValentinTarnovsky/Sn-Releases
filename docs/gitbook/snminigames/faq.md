# FAQ

### How do I update SnMiniGames?
Download the newer `snminigames-v*` release and replace the jar. Update SnLib first if the release notes ask for a newer version. Configs auto-merge on restart; `games/parkour.yml` is never touched.

### Why are the map setup commands missing from `/mg help`?
They are hidden from the root help on purpose. Every minigame ships its own set of setup commands, so listing them all would make `/mg help` unreadable once several games are installed. Each game exposes one entry instead, `/mg admin <game> help`, which lists that game's commands on its own page. The commands themselves are unchanged: they still run, and they still tab-complete under `/mg admin <game> `.

### Where do I configure the leave item?
Its material and name live in `config.yml` under `leave-item`, because one item is registered for every minigame and cannot differ between them. Whether a game hands it out and which hotbar slot it uses stay per-game, under `leave-item` in that game's file.

### How do I build a parkour map?
Create it with `/mg admin parkour create <name>`, then get the wand with `/mg admin wand` (running it again removes the wand). `/mg admin parkour help` lists every setup command if you forget one. Click a block and run `/mg admin parkour setstart <map>`, `addwaypoint <map>` or `setwin <map>`. Stand where you want the waiting lobby, end point or hold spot and run `setwaiting <map>`, `setend <map>` or `sethold <map>`.

### How do I start a round manually?
Two admin commands cover it. `/mg admin start <game>` opens a fresh queue right away without waiting for auto-start; add an optional map (`/mg admin start <game> <map>`) to force a specific map instead of the rotation pick, and the rotation cursor stays where it was. Once players are queued, `/mg admin forcestart <game>` skips the remaining countdown and begins the round immediately with whoever is in the queue - the minimum-players requirement is bypassed on purpose, so an empty queue is refused instead.

### Why does the [JOIN] chat button do nothing for some players?
Bedrock players (connecting through Geyser) cannot click chat buttons - that is a Bedrock limitation, not a bug. The announce also shows the plain command (`/minigames join <game>`), which works for everyone. If Java players cannot click either, check the startup log: the plugin warns when a language file edit or translation lost the `<click:...>` tag of a message.

### Can players move during the 3-2-1 countdown?
No. Movement is frozen at the start point until the GO title: position changes are reverted, walking is zeroed and anyone who still drifts is teleported back each second (this also holds for Bedrock players). Looking around stays free.

### What happens to a player's items if the server crashes mid-game?
Nothing is lost. The full player state is written to disk before the game touches it, and it restores on the next start or when the player reconnects.

### Can more than one player win?
Yes. Set `winners` above 1 on a map: each finisher waits frozen at the `finish-hold` spot until enough players finish or the time limit ends the round.

### Does it support Folia?
No, SnMiniGames targets Paper 1.20.x and 1.21.x.
