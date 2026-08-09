# Permissions

Administrative nodes default to `op`. The single player-facing node defaults to `true`, so everyone can use loaders unless you revoke it.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snchunkloader.admin` | op | Full administrative access. Also gates the root command itself. |
| `snchunkloader.admin.reload` | op | Allows `/chunkloader reload`. |
| `snchunkloader.admin.debug` | op | Allows `/chunkloader debug`. |
| `snchunkloader.admin.update` | op | Receives update notifications in chat. |
| `snchunkloader.admin.give` | op | Allows `/chunkloader give`. |
| `snchunkloader.admin.list` | op | Allows `/chunkloader list`. |
| `snchunkloader.admin.chunks` | op | Allows `/chunkloader chunks`. |
| `snchunkloader.use` | true | Basic usage: placing a loader, breaking one, and right-clicking one to add time or open its menu. |

{% hint style="info" %}
`snchunkloader.admin` is a parent node and its child list is exhaustive. A rank granted the parent gets every administrative action, so you do not need to grant the children one by one.
{% endhint %}

{% hint style="warning" %}
There is no `snchunkloader.*` wildcard. Grant `snchunkloader.admin` for staff and leave `snchunkloader.use` at its default for players.
{% endhint %}
