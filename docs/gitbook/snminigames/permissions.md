# Permissions

Player permissions default to `true`; every admin node defaults to `op`. `snminigames.admin` grants the whole admin tree.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snminigames.use` | true | Basic usage: join, leave and list minigames |
| `snminigames.admin` | op | Full administrative access (parent of every node below) |
| `snminigames.admin.start` | op | Force-start a round |
| `snminigames.admin.stop` | op | Force-stop a running round |
| `snminigames.admin.forcejoin` | op | Force a player into a round |
| `snminigames.admin.setup` | op | Map setup subcommands and the setup wand |
| `snminigames.admin.reload` | op | `/minigames reload` |
| `snminigames.admin.debug` | op | `/minigames debug` |
| `snminigames.admin.update` | op | Receive update notifications |
