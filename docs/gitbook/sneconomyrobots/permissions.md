# Permissions

Player nodes default to `true`, admin nodes to `op`, and the storage capacity ranks to `false`.

| Permission | Default | Description |
|-----------|---------|-------------|
| `sneconomyrobots.use` | true | Basic usage of SnEconomyRobots. Gates the whole `/robots` tree and the main menu |
| `sneconomyrobots.claim` | true | Allows claiming the robot income bag |
| `sneconomyrobots.merge` | true | Allows using the robot merge menu |
| `sneconomyrobots.upgrade` | true | Allows opening the robot upgrade menu, which is where upgrades are bought |
| `sneconomyrobots.admin` | op | Full administrative access. Parent of every node below |
| `sneconomyrobots.admin.reload` | op | Allows `/robots reload` |
| `sneconomyrobots.admin.debug` | op | Allows `/robots debug` |
| `sneconomyrobots.admin.update` | op | Receive update notifications of SnEconomyRobots |
| `sneconomyrobots.admin.give` | op | Allows `/robots admin give` |
| `sneconomyrobots.admin.givebox` | op | Allows `/robots admin givebox` |
| `sneconomyrobots.admin.giveslot` | op | Allows `/robots admin giveslot` |
| `sneconomyrobots.admin.takeslot` | op | Allows `/robots admin takeslot` |
| `sneconomyrobots.admin.setslots` | op | Allows `/robots admin setslots` |
| `sneconomyrobots.admin.clearstorage` | op | Allows `/robots admin clearstorage` |
| `sneconomyrobots.admin.setupgrade` | op | Allows `/robots admin setupgrade` |
| `sneconomyrobots.admin.resetlimit` | op | Allows `/robots admin resetlimit` |
| `sneconomyrobots.admin.bag` | op | Allows `/robots admin bag` |
| `sneconomyrobots.admin.setbag` | op | Allows `/robots admin setbag` |
| `sneconomyrobots.admin.clearbag` | op | Allows `/robots admin clearbag` |
| `sneconomyrobots.admin.info` | op | Allows `/robots admin info` |
| `sneconomyrobots.admin.list` | op | Allows `/robots admin list` |
| `sneconomyrobots.storage.vip` | false | Grants the vip storage capacity |
| `sneconomyrobots.storage.vip-plus` | false | Grants the vip-plus storage capacity |

## Storage capacity ranks

The storage rank nodes are config driven. Every key under `storage.ranks` in `config.yml` mints a
node named `storage.permission-prefix` plus that key, so adding a rank there adds its permission.
The two nodes above are the shipped examples.

Capacity is resolved live from the player's permissions, so a rank change applies without a relog. A
player holding no rank node gets `storage.default`.

{% hint style="info" %}
These nodes default to `false` on purpose. A capacity rank is granted to a rank, never held by
everyone.
{% endhint %}
