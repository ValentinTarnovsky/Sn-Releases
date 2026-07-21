# FAQ

### How do I update SnElementalGems?
Download the newer `snelementalgems-v*` release and replace the jar. Configs auto-merge on restart, so your edits stay.

### Does it support Folia?
No. SnElementalGems does not declare Folia support and is not explicitly tested on Folia. It targets Paper single-server setups.

### Does it require PlaceholderAPI or Vault?
No. Both are optional. PlaceholderAPI adds the `%snelementalgems_*%` placeholders, and Vault registers gems as the server economy when you enable the hook.

### Where is gem data stored?
In SQLite by default. Switch to MySQL under the `database` section of `config.yml`. Player balances never live in a YAML file.

### Do I need a stacker plugin for automated drops?
No. RoseStacker and WildStacker are optional. When present, automated drops honor a mob's real stack size. When absent, each death counts as one.
