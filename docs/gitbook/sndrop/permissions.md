# Permissions

| Node | Default | Grants |
|---|---|---|
| `sndrop.use` | everyone | Access to `/drop` and its subcommands |
| `sndrop.admin` | op | Everything below (parent node) |
| `sndrop.admin.reload` | op | `/drop reload` |
| `sndrop.admin.debug` | op | `/drop debug` |
| `sndrop.admin.update` | op | Receives the chat notice when a new version is released |

Granting `sndrop.admin` in LuckPerms grants all three child nodes with it.

## Two things worth knowing

**`sndrop.use` gates the whole command, including the admin subcommands.** It is the permission on
the command root, and every subcommand inherits it. If you explicitly revoke `sndrop.use` from a
staff rank, that rank cannot run `/drop reload` either, even holding `sndrop.admin.reload`. Since
`sndrop.use` defaults to everyone, this only happens if you deliberately negate it.

**There is no bypass node.** The drop restriction applies to every player without exception,
including operators and players in creative mode. This is intentional: the plugin exists to make
dropping impossible unless a window is open, and an exempt rank would be a hole in that. If you
need one, it is a change request.
