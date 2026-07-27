# Permissions

Player nodes default to `true` so everyone can use the catalogue out of the box. Admin nodes default to `op`. Granting `snkilleffects.admin` gives every admin node, because the parent lists them all as children.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snkilleffects.use` | true | Basic usage of SnKillEffects |
| `snkilleffects.preview` | true | Allows `/killeffects preview` |
| `snkilleffects.toggle` | true | Allows `/killeffects toggle` |
| `snkilleffects.admin` | op | Full administrative access of SnKillEffects |
| `snkilleffects.admin.reload` | op | Allows `/killeffects reload` |
| `snkilleffects.admin.debug` | op | Allows `/killeffects debug` |
| `snkilleffects.admin.update` | op | Receive update notifications of SnKillEffects |
| `snkilleffects.admin.give` | op | Allows `/killeffects give` |
| `snkilleffects.admin.take` | op | Allows `/killeffects take` |
| `snkilleffects.admin.giveall` | op | Allows `/killeffects giveall` |
| `snkilleffects.admin.unlockall` | op | Allows `/killeffects unlockall` |
| `snkilleffects.admin.set` | op | Allows `/killeffects admin set` |
| `snkilleffects.admin.clear` | op | Allows `/killeffects admin clear` |
| `snkilleffects.admin.list` | op | Allows `/killeffects admin list` |
| `snkilleffects.admin.silent` | op | Allows the `-s` silent flag on admin actions |

{% hint style="info" %}
`snkilleffects.preview` and `snkilleffects.toggle` are separate nodes so you can revoke either one without taking the catalogue away. Revoking `snkilleffects.preview` also disables the right-click preview inside the menu, and revoking `snkilleffects.toggle` disables the menu's toggle tile: the menu never works around a permission the command respects.
{% endhint %}
