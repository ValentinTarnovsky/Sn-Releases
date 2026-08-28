# Permissions

The three player nodes default to `true`, so everyone can use the plugin out of the box.
Every admin node defaults to `op`. The two numeric capacity nodes have no default and must be
granted per rank.

## Player

| Permission | Default | Description |
|-----------|---------|-------------|
| `sncompanions.use` | true | Basic usage of SnCompanions |
| `sncompanions.toggle` | true | Allows `/companions toggle` |
| `sncompanions.hide` | true | Allows `/companions hide` |

## Administration

`sncompanions.admin` is the parent of every node below and grants all of them at once.

| Permission | Default | Description |
|-----------|---------|-------------|
| `sncompanions.admin` | op | Full administrative access of SnCompanions |
| `sncompanions.admin.reload` | op | Allows `/companions reload` |
| `sncompanions.admin.debug` | op | Allows `/companions debug` |
| `sncompanions.admin.update` | op | Receives update notifications of SnCompanions |
| `sncompanions.admin.info` | op | Allows `/companions admin info` |
| `sncompanions.admin.list` | op | Allows `/companions admin list` |
| `sncompanions.admin.companion` | op | Allows `/companions admin companion` |
| `sncompanions.admin.open` | op | Allows `/companions admin open` |
| `sncompanions.admin.setlevel` | op | Allows `/companions admin setlevel` |
| `sncompanions.admin.setexp` | op | Allows `/companions admin setexp` |
| `sncompanions.admin.settype` | op | Allows `/companions admin settype` |
| `sncompanions.admin.setboost` | op | Allows `/companions admin setboost` |
| `sncompanions.admin.setowner` | op | Allows `/companions admin setowner` |
| `sncompanions.admin.equip` | op | Allows `/companions admin equip` |
| `sncompanions.admin.unequip` | op | Allows `/companions admin unequip` |
| `sncompanions.admin.removecompanion` | op | Allows `/companions admin removecompanion` |
| `sncompanions.admin.clear` | op | Allows `/companions admin clear` |
| `sncompanions.admin.reset` | op | Allows `/companions admin reset` |
| `sncompanions.admin.give` | op | Allows `/companions admin give` |
| `sncompanions.admin.givebox` | op | Allows `/companions admin givebox` |
| `sncompanions.admin.giveallbox` | op | Allows `/companions admin giveallbox` |
| `sncompanions.admin.slots` | op | Allows `/companions admin slots` |
| `sncompanions.admin.storage` | op | Allows `/companions admin storage` |
| `sncompanions.admin.currency` | op | Allows `/companions admin currency` |

## Rank capacities

| Permission | Default | Description |
|-----------|---------|-------------|
| `sncompanions.storage.<n>` | not set | Grants a companion storage capacity of `<n>` |
| `sncompanions.slots.<n>` | not set | Grants `<n>` companion equip slots |

These two nodes are numeric, so they cannot be declared in `plugin.yml`. Grant them per rank
in your permissions plugin, for example `sncompanions.storage.250` and `sncompanions.slots.4` on a
LuckPerms group. The plugin reads the highest `<n>` a player holds, so stacking several nodes
is safe: the largest one wins rather than summing. Since 1.22.0 that value is the player's
FLOOR - it replaces the config base when higher, and slots or storage sold with
`/companions admin slots|storage give` add on top of it.

{% hint style="info" %}
The resolved capacity is frozen into the database when the player joins. That means an admin
query against an offline player still reports the right total, and a rank change applies on
the player's next join.
{% endhint %}
