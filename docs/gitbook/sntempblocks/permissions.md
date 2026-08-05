# Permissions

Every node defaults to `op`. Granting `sntempblocks.admin` gives the whole administrative
surface, so a staff group needs that one node only. The individual nodes below exist so you can
hand out a subset instead.

| Permission | Default | Description |
|-----------|---------|-------------|
| `sntempblocks.admin` | op | Full administrative access. Parent of every node below |
| `sntempblocks.admin.list` | op | Allows `/tempblocks list` |
| `sntempblocks.admin.info` | op | Allows `/tempblocks info` |
| `sntempblocks.admin.here` | op | Allows `/tempblocks here` |
| `sntempblocks.admin.wipe` | op | Allows `/tempblocks wipe` |
| `sntempblocks.admin.purge` | op | Allows `/tempblocks purge` |
| `sntempblocks.admin.bypass` | op | Allows `/tempblocks bypass` |
| `sntempblocks.admin.reload` | op | Allows `/tempblocks reload` |
| `sntempblocks.admin.debug` | op | Allows `/tempblocks debug` |
| `sntempblocks.admin.update` | op | Receive update notifications in chat |

## The bypass node

`sntempblocks.admin.bypass` is what lets your builders work inside an arena without their work
expiring under them. It applies when a block is placed and when a bucket is emptied, so liquids
are covered too.

**The node gates the switch, it does not flip it.** Holding it means a staff member is allowed to
run `/tempblocks bypass`; until they do, their blocks are tracked exactly like a player's. Grant
it to your build team and let them turn it on when they need it.

{% hint style="info" %}
The bypass lasts for one session and is dropped on logout, so nobody can walk away leaving an
arena permanently untracked. Revoking the node ends an active bypass at the next block placed,
without waiting for that logout.
{% endhint %}

{% hint style="warning" %}
**Upgrading from 1.0.0:** the old `sntempblocks.bypass` node no longer exists. It bypassed on its
own, with no way to turn it off, which meant every operator was silently unable to place a
temporary block and testing a zone as staff looked broken. Grant `sntempblocks.admin.bypass` to
anyone who had the old node.
{% endhint %}
