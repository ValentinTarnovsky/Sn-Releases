# Permissions

| Node | Default | Grants |
|---|---|---|
| `snkills.admin` | op | Everything below. Also what makes `/snkills` visible at all |
| `snkills.admin.reload` | op | `/snkills reload` |
| `snkills.admin.debug` | op | `/snkills debug` |
| `snkills.admin.update` | op | Receives the update notice in chat when a new version is out |

`snkills.admin` is declared as the parent of the other three, so granting it in LuckPerms
grants the whole set.

There is no player-side node. SnKills has nothing a player can run - the death messages
themselves are broadcast to everyone regardless of permissions.

{% hint style="danger" %}
**Upgrading from 1.x:** the nodes changed. `snkills.use` and the `snkills.*` wildcard are
gone, and `snkills.reload` is now `snkills.admin.reload`. A staff rank granted the old nodes
silently loses `/snkills reload` - update your groups **before** you restart. See
[Migrating from 1.x](migration.md).
{% endhint %}
