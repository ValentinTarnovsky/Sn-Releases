# Permissions

Every administrative node defaults to OP. `customcrafting.use` defaults to true, so any player can see the help page. Using a workstation needs no permission at all: if you registered the block, players can craft on it.

| Permission | Default | Description |
|-----------|---------|-------------|
| `customcrafting.*` | OP | Everything below |
| `customcrafting.use` | true | Basic usage |
| `customcrafting.admin` | OP | All admin commands |
| `customcrafting.admin.set` | OP | Register a workstation |
| `customcrafting.admin.remove` | OP | Remove a workstation |
| `customcrafting.admin.bypass` | OP | Skip craft time |
| `customcrafting.admin.give` | OP | Give plugin items |
| `customcrafting.admin.reload` | OP | Reload configuration |
| `customcrafting.admin.debug` | OP | Toggle debug output |
| `customcrafting.admin.update` | OP | Receive update notifications |

## The three SnLib-derived nodes

`reload`, `debug` and the update notice are built by SnLib, which derives their permission from the plugin's own name rather than from the command name. What it actually tests is `sncustomcrafting.admin.reload`, `sncustomcrafting.admin.debug` and `sncustomcrafting.admin.update`.

The three nodes in the table are declared as parents of those, so granting the documented ones, or the `customcrafting.*` wildcard, is enough on LuckPerms and on any permission plugin that applies Bukkit child permissions.

{% hint style="info" %}
If your permission plugin does not apply Bukkit child permissions, grant `sncustomcrafting.admin.reload`, `sncustomcrafting.admin.debug` and `sncustomcrafting.admin.update` directly. There is no `sncustomcrafting.*` wildcard to grant.
{% endhint %}
