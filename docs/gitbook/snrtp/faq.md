# FAQ

### How do I update SnRTP?
Download the newer `snrtp-v*` release and replace the jar. Configs auto-merge on restart.

### Does it support Folia?
No, SnRTP is not Folia-compatible. It uses the GUI menu, which runs on Paper only.

### How do I add another world?
Add a block under `worlds:` in `config.yml`, then add a matching button in `guis/menu.yml`. Use the same key in both files.

### A random teleport landed near a player claim. Why?
Region avoidance needs WorldGuard installed. ProtectionStones claims are WorldGuard regions, so install WorldGuard to skip them.

### Can a teleport drop a player into lava or the void?
No. The search only accepts solid footing with two blocks of clearance, and it rejects the unsafe blocks listed in `config.yml`.
