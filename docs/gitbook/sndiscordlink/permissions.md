# Permissions

Every player gets the basic nodes by default. The admin nodes default to op, and `sndiscordlink.admin` is a parent that grants all of them at once.

| Permission | Default | Description |
|-----------|---------|-------------|
| `sndiscordlink.use` | true | Basic usage: link, verify, unlink and status |
| `sndiscordlink.claim` | true | `/discord claim <claim>` and the `/booster` shortcut |
| `sndiscordlink.admin` | op | Full administrative access, grants every node below |
| `sndiscordlink.admin.info` | op | Allows `/discord info <player>` |
| `sndiscordlink.admin.forceunlink` | op | Allows `/discord forceunlink <player>` |
| `sndiscordlink.admin.forcelink` | op | Allows `/discord forcelink <player> <discordId>` |
| `sndiscordlink.admin.sync` | op | Allows `/discord sync <player>` |
| `sndiscordlink.admin.stats` | op | Allows `/discord stats` |
| `sndiscordlink.admin.reload` | op | Allows `/discord reload` |
| `sndiscordlink.admin.debug` | op | Allows `/discord debug` |
| `sndiscordlink.admin.update` | op | Receives the update notification in chat |

{% hint style="info" %}
Grant `sndiscordlink.admin` to a staff group in LuckPerms and every child node comes with it. The children list is exhaustive, so no admin action is left behind.
{% endhint %}
