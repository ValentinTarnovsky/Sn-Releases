# FAQ

### How do I update SnRTP?
Download the newer `snrtp-v*` release and replace the jar. Configs auto-merge on restart.

### Does it support Folia?
No, SnRTP is not Folia-compatible. It uses the GUI menu, which runs on Paper only.

### How do I add another world?
Add a block under `worlds:` in `config.yml`, then add a matching button in `guis/menu.yml`. Use the same key in both files.

### How do I remove a world?
Delete its block under `worlds:` in `config.yml` and its button under `items:` in `guis/menu.yml`. Both stay deleted from SnRTP 1.3.0 on.

### I deleted a world and it came back after a restart.
That was fixed in 1.3.0. Older versions treated the shipped `overworld` / `nether` / `end` entries as mandatory, so the updater re-inserted whatever you removed on every boot. Update SnRTP (and SnLib to 1.15.0), delete the world and its menu button once more, and it will stick.

### Can I run SnRTP 1.4.0 on an older SnLib?
No. It needs SnLib 1.20.0 or later and disables itself with a console notice on anything older.

### The console says `[Sn] License: FAIL` and SnRTP does not enable.
From 1.4.0 on SnRTP needs a valid Sn bundle key in `plugins/.Sn-License/license.yml` (one shared
file for every bundle plugin on the server). Check, in order: the placeholder line was actually
replaced with your key, the subscription is still active, the server has outbound HTTPS at boot,
and the jar is the unmodified one from the release - the license is bound to that build's checksum.

### Do I need one key per plugin?
No. The bundle key is per server, not per plugin: paste it once into
`plugins/.Sn-License/license.yml` and every Sn bundle plugin on that server reads the same file.

### Does SnRTP phone home while the server runs?
No. The license is checked once, at startup. After that there are no further calls - a network
outage or an expiring key mid-session will not stop or degrade a running server; the next restart
is when it is checked again.

### A random teleport landed near a player claim. Why?
Region avoidance needs WorldGuard installed. ProtectionStones claims are WorldGuard regions, so install WorldGuard to skip them.

### Can a teleport drop a player into lava or the void?
No. The search only accepts solid footing with two blocks of clearance, and it rejects the unsafe blocks listed in `config.yml`.
