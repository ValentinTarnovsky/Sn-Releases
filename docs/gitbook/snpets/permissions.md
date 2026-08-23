# Permissions

The three player nodes default to `true`, so everyone can use the plugin out of the box.
Every admin node defaults to `op`. The two numeric capacity nodes have no default and must be
granted per rank.

## Player

| Permission | Default | Description |
|-----------|---------|-------------|
| `snpets.use` | true | Basic usage of SnPets |
| `snpets.toggle` | true | Allows `/pets toggle` |
| `snpets.hide` | true | Allows `/pets hide` |

## Administration

`snpets.admin` is the parent of every node below and grants all of them at once.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snpets.admin` | op | Full administrative access of SnPets |
| `snpets.admin.reload` | op | Allows `/pets reload` |
| `snpets.admin.debug` | op | Allows `/pets debug` |
| `snpets.admin.update` | op | Receives update notifications of SnPets |
| `snpets.admin.info` | op | Allows `/pets admin info` |
| `snpets.admin.list` | op | Allows `/pets admin list` |
| `snpets.admin.pet` | op | Allows `/pets admin pet` |
| `snpets.admin.open` | op | Allows `/pets admin open` |
| `snpets.admin.setlevel` | op | Allows `/pets admin setlevel` |
| `snpets.admin.setexp` | op | Allows `/pets admin setexp` |
| `snpets.admin.settype` | op | Allows `/pets admin settype` |
| `snpets.admin.settrait` | op | Allows `/pets admin settrait` |
| `snpets.admin.setboost` | op | Allows `/pets admin setboost` |
| `snpets.admin.setowner` | op | Allows `/pets admin setowner` |
| `snpets.admin.equip` | op | Allows `/pets admin equip` |
| `snpets.admin.unequip` | op | Allows `/pets admin unequip` |
| `snpets.admin.removepet` | op | Allows `/pets admin removepet` |
| `snpets.admin.clear` | op | Allows `/pets admin clear` |
| `snpets.admin.reset` | op | Allows `/pets admin reset` |
| `snpets.admin.give` | op | Allows `/pets admin give` |
| `snpets.admin.givebox` | op | Allows `/pets admin givebox` |
| `snpets.admin.giveallbox` | op | Allows `/pets admin giveallbox` |
| `snpets.admin.slots` | op | Allows `/pets admin slots` |
| `snpets.admin.storage` | op | Allows `/pets admin storage` |
| `snpets.admin.currency` | op | Allows `/pets admin currency` |

## Rank capacities

| Permission | Default | Description |
|-----------|---------|-------------|
| `snpets.storage.<n>` | not set | Grants a pet storage capacity of `<n>` |
| `snpets.slots.<n>` | not set | Grants `<n>` pet equip slots |

These two nodes are numeric, so they cannot be declared in `plugin.yml`. Grant them per rank
in your permissions plugin, for example `snpets.storage.250` and `snpets.slots.4` on a
LuckPerms group. The plugin reads the highest `<n>` a player holds, so stacking several nodes
is safe: the largest one wins rather than summing.

{% hint style="info" %}
The resolved capacity is frozen into the database when the player joins. That means an admin
query against an offline player still reports the right total, and a rank change applies on
the player's next join.
{% endhint %}
