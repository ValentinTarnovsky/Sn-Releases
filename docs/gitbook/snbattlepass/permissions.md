# Permissions

Players get `snbattlepass.use` by default. Every admin node defaults to op. The two pass nodes default to false, so nobody holds a pass by accident.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snbattlepass.use` | true | Open the menus, view progress, claim rewards |
| `snbattlepass.admin` | op | Parent of every admin node below |
| `snbattlepass.admin.reload` | op | Allows `/battlepass reload` |
| `snbattlepass.admin.debug` | op | Allows `/battlepass debug` |
| `snbattlepass.admin.update` | op | Receive update notifications |
| `snbattlepass.admin.manage` | op | Allows `setpass`, `removepass`, `settier`, `addtier`, `addxp` and `setxp` |
| `snbattlepass.admin.reset` | op | Allows `reset` and `resetall` |
| `snbattlepass.admin.bypass` | op | Allows `bypass` |
| `snbattlepass.admin.challenges` | op | Allows `givechallenge` and `reroll` |
| `snbattlepass.pass.gold` | false | Grants the Gold pass while the permission is held |
| `snbattlepass.pass.diamond` | false | Grants the Diamond pass while the permission is held |

## Passes granted by permission

`snbattlepass.pass.gold` and `snbattlepass.pass.diamond` grant a pass dynamically, for exactly as long as the player holds the node. Nothing is written to the database, so removing the permission removes the benefit immediately and cleanly.

That makes these nodes the right way to attach a pass to a rank or a subscription: when the rank goes, the pass goes with it, and the player keeps any reward they already claimed.

{% hint style="info" %}
A player who buys a pass in game has it stored permanently instead. Permission and purchase stack safely: the player is treated as holding whichever pass is higher.
{% endhint %}
