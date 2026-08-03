# Permissions

| Node | Description | Default |
|------|-------------|---------|
| `sntags.use` | Open `/tags` and wear a tag | `true` |
| `sntags.admin` | Full administrative access; required for every `/tagadmin` subcommand | `op` |
| `sntags.admin.create` | `/tagadmin create` | `op` |
| `sntags.admin.delete` | `/tagadmin delete` | `op` |
| `sntags.admin.give` | `/tagadmin give` | `op` |
| `sntags.admin.remove` | `/tagadmin remove` | `op` |
| `sntags.admin.custom` | `/tagadmin custom` | `op` |
| `sntags.admin.removecustom` | `/tagadmin removecustom` | `op` |
| `sntags.admin.list` | `/tagadmin list` | `op` |
| `sntags.admin.viewall` | Show every global tag in the selector, owned or not | `op` |
| `sntags.admin.reload` | `/tags reload` and `/tagadmin reload` | `op` |
| `sntags.admin.debug` | `/tags debug` and `/tagadmin debug` | `op` |
| `sntags.admin.update` | Receive update notifications in chat | `op` |
| `sntags.*` | Everything above | `op` |

## How the admin nodes combine

`sntags.admin` gates the `/tagadmin` root itself and is checked **before** any subcommand. A staff member with only `sntags.admin.create` cannot run `/tagadmin create` at all - the root refuses them first.

So the per-leaf nodes are there to take permissions **away** from someone who already has `sntags.admin`, not to hand out one subcommand on its own. Granting `sntags.admin` grants all of its children with it; negate the ones you do not want.

{% hint style="warning" %}
`sntags.admin` therefore includes `sntags.admin.custom`, and a custom tag display can contain PlaceholderAPI tokens that resolve against whoever reads the text. Treat `sntags.admin` as a trusted role.
{% endhint %}

## sntags.admin.viewall

Staff with this node see the whole catalogue in `/tags`, including tags they were never granted, and can equip any of them. Useful for previewing a tag before handing it out.

Two things worth knowing:

- The stored selection outlives the permission. Someone who equips a tag through `viewall` and later loses the node stops rendering it on their next load, reload or relog, but the row stays on disk until they pick something else.
- It is a child of `sntags.admin`, so every staff member with `/tagadmin` has it by default.
