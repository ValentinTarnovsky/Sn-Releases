# FAQ

### How do I update SnClans?
Download the newer `snclans-v*` release and replace the jar. Configs auto-merge on restart, so your edits stay.

### Does it support Folia?
No, SnClans is not Folia-compatible. It targets Paper single-server setups.

### Does it require PlaceholderAPI or WorldGuard?
No. Both are optional. PlaceholderAPI adds placeholders, and WorldGuard adds region gating for homes, banners, and PvP.

### Where is clan data stored?
In SQLite by default. Switch to MySQL under the `database` section of `config.yml`.

### Can I show clan features in chat instead of GUIs?
Yes. Set each entry under the `presentation` section of `config.yml` to `chat` instead of `gui`.
