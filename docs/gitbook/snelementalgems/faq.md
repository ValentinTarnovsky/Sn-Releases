# FAQ

### How do I update SnElementalGems?
Download the newer `snelementalgems-v*` release and replace the jar. Configs auto-merge on restart, so your edits stay.

### I deleted a drop type / shop category and it came back. Why?
It should not, as of 1.3.0. The drop catalogues in `droprates.yml`, `categories` in `shops.yml` and each `levels` table in `upgrades.yml` are treated as your data: entries you delete stay deleted. If a deleted entry reappears, you most likely deleted the whole section rather than an entry inside it - the section key itself is plugin schema, so removing it restores the shipped defaults. To keep a section with no entries, leave it as an empty map (`categories: {}`). See [Configuration](configuration.md).

### Why does a new drop type from the changelog not appear in my droprates.yml?
Because that section is yours, the updater never writes into it again, so new shipped entries only reach a fresh file. Copy the entry from the release notes or the default file and add it by hand.

### Does it support Folia?
No. SnElementalGems does not declare Folia support and is not explicitly tested on Folia. It targets Paper single-server setups.

### Does it require PlaceholderAPI or Vault?
No. Both are optional. PlaceholderAPI adds the `%snelementalgems_*%` placeholders, and Vault registers gems as the server economy when you enable the hook.

### Where is gem data stored?
In SQLite by default. Switch to MySQL under the `database` section of `config.yml`. Player balances never live in a YAML file.

### Do I need a stacker plugin for automated drops?
No. RoseStacker and WildStacker are optional. When present, automated drops honor a mob's real stack size. When absent, each death counts as one.
