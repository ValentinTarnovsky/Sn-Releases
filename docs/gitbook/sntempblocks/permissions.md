# Permissions

Every node defaults to `op`. Granting `sntempblocks.admin` gives the whole administrative
surface, including `sntempblocks.bypass`, so a staff group needs that one node only. The
individual nodes below exist so you can hand out a subset instead.

| Permission | Default | Description |
|-----------|---------|-------------|
| `sntempblocks.admin` | op | Full administrative access. Parent of every node below |
| `sntempblocks.admin.list` | op | Allows `/tempblocks list` |
| `sntempblocks.admin.info` | op | Allows `/tempblocks info` |
| `sntempblocks.admin.here` | op | Allows `/tempblocks here` |
| `sntempblocks.admin.wipe` | op | Allows `/tempblocks wipe` |
| `sntempblocks.admin.purge` | op | Allows `/tempblocks purge` |
| `sntempblocks.admin.reload` | op | Allows `/tempblocks reload` |
| `sntempblocks.admin.debug` | op | Allows `/tempblocks debug` |
| `sntempblocks.admin.update` | op | Receive update notifications in chat |
| `sntempblocks.bypass` | op | Blocks this player places are never tracked |

## The bypass node

`sntempblocks.bypass` is what lets your builders work inside an arena without their work expiring
under them. It is checked when a block is placed and when a bucket is emptied, so it applies to
liquids too.

{% hint style="info" %}
Bypass is evaluated at placement time, not at expiry time. A block placed before you granted the
node is already tracked and still expires. A block placed while the player holds the node is
never tracked at all, so it stays forever.
{% endhint %}
