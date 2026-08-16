# Permissions

Every node defaults to `op`. Grant `snrotatinheads.admin` to a staff rank and it inherits every node
below; grant single nodes for finer control.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snrotatinheads.admin` | op | Full administrative access (parent of every node below) |
| `snrotatinheads.admin.reload` | op | `/rh reload` |
| `snrotatinheads.admin.debug` | op | `/rh debug` |
| `snrotatinheads.admin.update` | op | Receive update notifications on join |
| `snrotatinheads.admin.create` | op | `/rh create` |
| `snrotatinheads.admin.remove` | op | `/rh remove` |
| `snrotatinheads.admin.list` | op | `/rh list` |
| `snrotatinheads.admin.info` | op | `/rh info` |
| `snrotatinheads.admin.movehere` | op | `/rh movehere` |
| `snrotatinheads.admin.tp` | op | `/rh tp` |
| `snrotatinheads.admin.texture` | op | `/rh texture` |
| `snrotatinheads.admin.size` | op | `/rh size` |
| `snrotatinheads.admin.rotationspeed` | op | `/rh rotationspeed` |
| `snrotatinheads.admin.bouncespeed` | op | `/rh bouncespeed` |
| `snrotatinheads.admin.bounceheight` | op | `/rh bounceheight` |
| `snrotatinheads.admin.viewrange` | op | `/rh viewrange` |
| `snrotatinheads.admin.hologram` | op | The whole `/rh hologram` group |
| `snrotatinheads.admin.action` | op | The whole `/rh action` group |

Clicking a head needs no permission: any player who can reach the hitbox fires its actions.
