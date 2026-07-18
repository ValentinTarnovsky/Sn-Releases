# Permissions

Basic play permissions default to `true`, so any player can use clans out of the box. Staff and admin nodes default to `op`.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snclans.use` | true | Basic usage: create, join, and manage a clan |
| `snclans.home` | true | Use `/clan home` |
| `snclans.sethome` | true | Use `/clan sethome` |
| `snclans.banner` | true | Use `/clan banner` |
| `snclans.ally` | true | Use alliance commands and ally chat |
| `snclans.chat.color` | true | Use color codes in clan and ally chat |
| `snclans.notify` | op | Receive staff notifications of clan events |
| `snclans.admin` | op | Full administrative access (parent of every admin node) |
| `snclans.admin.reload` | op | Use `/clan reload` |
| `snclans.admin.debug` | op | Use `/clan debug` |
| `snclans.admin.givepoint` | op | Use `/clan givepoint` |
| `snclans.admin.silent` | op | Use the `-s` silent flag on admin actions |
| `snclans.admin.rename` | op | Use `/clan admin rename` |
| `snclans.admin.disband` | op | Use `/clan admin disband` |
| `snclans.admin.transfer` | op | Use `/clan admin transfer` |
| `snclans.admin.join` | op | Use `/clan admin join` |
| `snclans.admin.kick` | op | Use `/clan admin kick` |
| `snclans.admin.promote` | op | Use `/clan admin promote` |
| `snclans.admin.demote` | op | Use `/clan admin demote` |
| `snclans.admin.ban` | op | Use `/clan admin ban` |
| `snclans.admin.unban` | op | Use `/clan admin unban` |
| `snclans.admin.description` | op | Use `/clan admin description` |
| `snclans.admin.open` | op | Use `/clan admin open` |
| `snclans.admin.close` | op | Use `/clan admin close` |

{% hint style="info" %}
These nodes only cover the server-side `/clan admin` tools. In-clan member actions such as invite, kick, and promote are not permissions: they are governed by each clan's own permission matrix, which the leader edits with `/clan permissions`.
{% endhint %}
