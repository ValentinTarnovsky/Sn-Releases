# Permissions

`sngifts.use` defaults to `true`, so every player can open the menu out of the box. Everything else
defaults to `op`.

| Permission | Default | Description |
|-----------|---------|-------------|
| `sngifts.use` | true | Basic usage of SnGifts. Gates the whole `/gifts` tree |
| `sngifts.admin` | op | Full administrative access. Grants every child below |
| `sngifts.admin.reload` | op | Allows `/gifts reload` |
| `sngifts.admin.debug` | op | Allows `/gifts debug` |
| `sngifts.admin.update` | op | Receive update notifications of SnGifts |
| `sngifts.admin.reset` | op | Allows `/gifts reset` |
| `sngifts.admin.resetall` | op | Allows `/gifts resetall` |
| `sngifts.admin.resetgifts` | op | Allows `/gifts resetgifts` |
| `sngifts.admin.bypass` | op | Allows `/gifts bypass` |
| `sngifts.claim.bypass-ip` | op | Skips the per IP claim limit when claiming a gift |

## Granting staff access

`sngifts.admin` is a parent node with an exhaustive children list, so one grant covers every admin
action including the per IP bypass:

```
/lp group mod permission set sngifts.admin true
/lp group mod permission set sngifts.use true
```

The second line matters. `sngifts.use` is not a child of `sngifts.admin`, and it sits on the root
of the command tree, so a group without it cannot run the admin subcommands either. Operators never
notice, because op satisfies both.

{% hint style="info" %}
`sngifts.claim.bypass-ip` is an entitlement, not a command gate. A player who has it is never
counted against `claim.ip-limit-per-gift`, and their claims never consume a slot for the address
they share.
{% endhint %}
