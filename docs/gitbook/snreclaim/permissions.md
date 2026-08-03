# Permissions

| Node | Default | Grants |
|---|---|---|
| `snreclaim.use` | true | Run `/reclaim` and redeem your own reclaims |
| `snreclaim.admin` | op | Parent node - grants every admin node below |
| `snreclaim.admin.reset` | op | `/reclaim reset <player>` |
| `snreclaim.admin.manage` | op | `/reclaim enable` and `/reclaim disable` |
| `snreclaim.admin.status` | op | `/reclaim status` (read-only) |
| `snreclaim.admin.reload` | op | `/reclaim reload` |
| `snreclaim.admin.debug` | op | `/reclaim debug` |
| `snreclaim.admin.update` | op | Receive update notices in chat |

Granting `snreclaim.admin` unlocks all of them at once - the children map is exhaustive.

## Which ranks a player qualifies for is NOT a permission

This is the part that surprises people. SnReclaim does not define a node per rank. A player
qualifies for a rank because they are **in the Vault permission group** that rank names, and
that is decided entirely by your permissions plugin.

```yaml
ranks:
  vip:
    group: "vip"     # <- this is a LuckPerms GROUP, not a permission node
```

So a player is offered the `vip` reclaim when LuckPerms says they are in the `vip` group.
Adding `snreclaim.something` to them changes nothing.

{% hint style="info" %}
Group names are matched case-insensitively, so a player in `VIP` still matches a rank
configured as `vip`.
{% endhint %}

## Useful grants

**Let helpers check the season lock without being able to change it:**

```
/lp group helper permission set snreclaim.admin.status true
```

**A moderator who can reset players but not close redemption:**

```
/lp group mod permission set snreclaim.admin.reset true
```

**Staff who should see update notices:**

```
/lp group admin permission set snreclaim.admin.update true
```

{% hint style="warning" %}
`snreclaim.use` defaults to true. If you negate it for a restricted group, that group
cannot run `/reclaim` **at all** - including the admin subcommands, because the root
command is gated on it. Grant `snreclaim.use` alongside any admin node.
{% endhint %}
