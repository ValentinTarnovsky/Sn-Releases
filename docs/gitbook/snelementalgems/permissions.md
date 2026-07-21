# Permissions

Basic play permissions default to `true`, so any player can earn and spend gems out of the box. Admin nodes default to `op`.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snelementalgems.use` | true | Basic usage: balance, shop, upgrades, and top |
| `snelementalgems.pay` | true | Use `/gems pay` |
| `snelementalgems.withdraw` | true | Use `/gems withdraw` |
| `snelementalgems.balance.others` | op | View another player's gem balance |
| `snelementalgems.gemdrop.entity` | true | Receive gems from killing entities |
| `snelementalgems.gemdrop.block` | true | Receive gems from breaking blocks |
| `snelementalgems.gemdrop.fish` | true | Receive gems from fishing |
| `snelementalgems.admin` | op | Full administrative access (parent of every admin node) |
| `snelementalgems.admin.reload` | op | Use `/gems reload` |
| `snelementalgems.admin.debug` | op | Use `/gems debug` |
| `snelementalgems.admin.update` | op | Receive update notices in chat |
| `snelementalgems.admin.give` | op | Use `/gems admin give` |
| `snelementalgems.admin.add` | op | Use `/gems admin add` |
| `snelementalgems.admin.remove` | op | Use `/gems admin remove` |
| `snelementalgems.admin.set` | op | Use `/gems admin set` |

{% hint style="info" %}
Individual shop rewards can require any permission node you choose, set through `required-permission` in `shops.yml`. These are your own nodes, not fixed plugin permissions.
{% endhint %}
