# FAQ

### How do I update SnRankPerks?
Download the newer `snrankperks-v*` release and replace the jar. Configs auto-merge on
restart.

### Does it support Folia?
Yes, SnRankPerks is Folia-compatible through SnLib's scheduler.

### Do I need LuckPerms?
No. Set `access.mode` to `PERMISSION` in `config.yml` to gate access with a plain permission
node instead. LuckPerms stays useful as a fallback prefix source when it is installed, even in
PERMISSION mode.

### Can I use MySQL instead of SQLite?
Yes, set `database.type` to `mysql` in `config.yml` and fill in the connection details below
it. SQLite is the default and needs no extra configuration.

### Why does a player keep their glow, prefix or chat color after they lose access?
They don't: losing access (a LuckPerms group change, or a permission revoke followed by
`/rankperks reload`) removes the live glow, prefix and chat color immediately. The database
row is left untouched, so the same selections come back automatically if access is restored.
