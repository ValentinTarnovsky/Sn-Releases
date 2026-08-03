# Permissions

| Node | Default | Grants |
|---|---|---|
| `snsimplevouchers.admin` | op | Full administrative access. Parent of every node below |
| `snsimplevouchers.admin.give` | op | `/voucher give`, **and** the give button inside `/voucher open` |
| `snsimplevouchers.admin.open` | op | `/voucher open` |
| `snsimplevouchers.admin.reload` | op | `/voucher reload`, and receives the post-reload voucher count |
| `snsimplevouchers.admin.debug` | op | `/voucher debug` |
| `snsimplevouchers.admin.update` | op | Receives update notifications |

## There is no player node

Claiming a voucher needs no permission at all. A player who holds a voucher item can right-click it. Every command is staff-facing, which is why the root itself is gated rather than each subcommand individually.

## The root node is required, not just inherited

{% hint style="warning" %}
`/voucher` is gated on `snsimplevouchers.admin`, so a child node **on its own is not enough**. A rank granted only `snsimplevouchers.admin.open` is refused.
{% endhint %}

To give a helper rank partial access, grant the parent and negate what you want withheld:

```
/lp group helper permission set snsimplevouchers.admin true
/lp group helper permission set snsimplevouchers.admin.give false
/lp group helper permission set snsimplevouchers.admin.reload false
```

That rank can browse `/voucher open` but cannot hand anything out, from the command **or** from the menu.

## Upgrading from 1.x

`snsimplevouchers.admin` still grants everything, because every new node is declared as its child. A group that had the old single node keeps working with no changes.

What did change: in v1.4.3 the command itself was ungated, so ordinary players could run `/voucher` and saw `reload`, `open` and `give` in tab completion. They no longer can. If any non-staff rank relied on reaching the command, grant it `snsimplevouchers.admin` explicitly.
