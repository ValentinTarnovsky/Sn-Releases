# FAQ

### How do I update SnMiniGames?
Download the newer `snminigames-v*` release and replace the jar. Configs auto-merge on restart; `games/parkour.yml` is never touched.

### How do I build a parkour map?
Get the wand with `/mg admin wand`, click a block, then run `/mg admin parkour <map> setstart`, `addwaypoint` or `setwin`. Stand where you want the waiting lobby, end point or hold spot and run `setwaiting`, `setend` or `sethold`.

### What happens to a player's items if the server crashes mid-game?
Nothing is lost. The full player state is written to disk before the game touches it, and it restores on the next start or when the player reconnects.

### Can more than one player win?
Yes. Set `winners` above 1 on a map: each finisher waits frozen at the `finish-hold` spot until enough players finish or the time limit ends the round.

### Does it support Folia?
No, SnMiniGames targets Paper 1.20.x and 1.21.x.
