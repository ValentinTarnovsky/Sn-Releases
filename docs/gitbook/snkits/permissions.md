# Permissions

`snkits.use` defaults to true, so every player can open the menus and claim kits out of the box.
Everything else defaults to op.

## Nodes

| Permission | Default | Description |
|-----------|---------|-------------|
| `snkits.use` | true | Open the kit menus and claim kits |
| `snkits.admin` | op | Parent of every admin permission below |
| `snkits.admin.gui` | op | Allows `/kit gui` |
| `snkits.admin.create` | op | Allows `/kit create` |
| `snkits.admin.delete` | op | Allows `/kit delete` |
| `snkits.admin.edit` | op | Allows `/kit edit` |
| `snkits.admin.toggle` | op | Allows `/kit enable` and `/kit disable` |
| `snkits.admin.give` | op | Allows `/kit give` |
| `snkits.admin.reset` | op | Allows `/kit reset` |
| `snkits.admin.import` | op | Allows `/kit import` |
| `snkits.admin.reload` | op | Allows `/kit reload` |
| `snkits.admin.update` | op | Receive the update notice in chat |

`snkits.admin` lists every child above, so a group granted the parent gets every admin action.

## Per kit nodes

These two take data in their last segment, so they are not declared in `plugin.yml`. Grant them
like any other node.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snkits.kit.<id>` | op | Entitles the player to one kit |
| `snkits.uses.<id>.<n>` | op | How many copies of that kit the player owns |
| `snkits.uses.*.<n>` | op | The same amount applied to every kit |

{% hint style="info" %}
`snkits.kit.<id>` is only checked when that kit sets `requires-permission: true`. A kit that does
not set the flag is claimable by anyone with `snkits.use`, whatever nodes exist.
{% endhint %}

`snkits.uses.<id>.<n>` only does anything while `multi-claim.enabled` is true, and the prefix
follows `multi-claim.permission-prefix`. The highest matching amount wins, so selling a third copy
never requires unsetting the second.

```
/lp user Snopeyy permission set snkits.kit.ninja
/lp user Snopeyy permission set snkits.uses.ninja.2
/lp group vip permission set snkits.uses.*.2
```
