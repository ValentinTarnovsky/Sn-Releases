# Permissions

`snrankup.use` defaults to true, so every player can open the menu and rank up out of the box.
Everything else defaults to op.

## Nodes

| Permission | Default | Description |
|-----------|---------|-------------|
| `snrankup.use` | true | Open the rank menu and rank up |
| `snrankup.admin` | op | Parent of every admin permission below |
| `snrankup.admin.force` | op | Allows `/rankup force` |
| `snrankup.admin.set` | op | Allows `/rankup set` |
| `snrankup.admin.reset` | op | Allows `/rankup reset` |
| `snrankup.admin.bypass` | op | Allows `/rankup bypass` |
| `snrankup.admin.reload` | op | Allows `/rankup reload` |
| `snrankup.admin.debug` | op | Allows `/rankup debug` |
| `snrankup.admin.update` | op | Receive the update notice in chat |

`snrankup.admin` lists every child above, so a group granted the parent gets every admin action.

{% hint style="info" %}
There is no per rank permission node. Who may reach a rank is decided by its requirements in
`rankup.yml`, not by a permission.
{% endhint %}

```
/lp group moderator permission set snrankup.admin.force
/lp group moderator permission set snrankup.admin.set
/lp group builder permission set snrankup.use false
```

Revoking `snrankup.use` removes the command entirely for that group, menu included.
