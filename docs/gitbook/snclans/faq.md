# FAQ

### How do I update SnClans?
Download the newer `snclans-v*` release and replace the jar. Configs auto-merge on restart, so your edits stay.

### Does it support Folia?
No, SnClans is not Folia-compatible. It targets Paper single-server setups.

### Does it require PlaceholderAPI or WorldGuard?
No. Both are optional. PlaceholderAPI adds placeholders, and WorldGuard adds region gating for homes, banners, and PvP.

### Does the banner hologram need DecentHolograms?
No. The rally hologram uses SnLib's built-in renderer by default. To use DecentHolograms instead, install it and set `banner.hologram.provider` to `decentholograms`. When that plugin is missing, SnClans falls back to the built-in renderer and logs a console warning.

### Where is clan data stored?
In SQLite by default. Switch to MySQL under the `database` section of `config.yml`.

### Can I show clan features in chat instead of GUIs?
Yes. Set each entry under the `presentation` section of `config.yml` to `chat` instead of `gui`.

### How do I change the "Open" / "Closed" or "Allowed" / "Denied" wording?
In `lang/messages_en.yml`, under the `lists` block: `status-open`, `status-closed`, `status-allowed`, `status-denied`, and the rest of the `status-*` keys. Each one is shared by the chat commands, the menus, and the placeholders, so editing it once changes every place that word appears. The permission names next to them come from the `actions` block in the same file. See [State words and action names](configuration.md#state-words-and-action-names).
