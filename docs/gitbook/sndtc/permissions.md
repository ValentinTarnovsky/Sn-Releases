# Permissions

| Permission | Default | Description |
|------------|---------|-------------|
| `sndtc.use` | true | Access the `/sndtc` command tree |
| `sndtc.break` | true | Damage an active core |
| `sndtc.admin` | OP | Every admin action - a parent of all the nodes below |

## Admin nodes

`sndtc.admin` grants all of these. Give them individually to hand out part of the surface.

| Permission | Command |
|------------|---------|
| `sndtc.admin.create` | `/sndtc create` |
| `sndtc.admin.delete` | `/sndtc delete` |
| `sndtc.admin.start` | `/sndtc start` |
| `sndtc.admin.stop` | `/sndtc stop` |
| `sndtc.admin.setblock` | `/sndtc setblock` |
| `sndtc.admin.sethealth` | `/sndtc sethealth` |
| `sndtc.admin.setcron` | `/sndtc setcron` |
| `sndtc.admin.setrange` | `/sndtc setrange` |
| `sndtc.admin.sethologram` | `/sndtc sethologram` |
| `sndtc.admin.list` | `/sndtc list` |
| `sndtc.admin.info` | `/sndtc info` |
| `sndtc.admin.reload` | `/sndtc reload` |
| `sndtc.admin.debug` | `/sndtc debug` |
| `sndtc.admin.update` | Update notifications |

## Making cores staff-only

Revoke `sndtc.break`. A player without it is told once, on their first click, that they may not
break the core, and the click is cancelled so no breaking animation starts. Repeated attempts are
rate-limited to one message every three seconds, so holding the mouse button down does not flood
their chat.

The message they see is `messages.no-break-permission` in `lang/messages_en.yml`.

{% hint style="info" %}
The break itself is always cancelled at a core's coordinates, whatever the player's permissions.
`sndtc.break` decides whether the hit **counts as damage**, never whether the block can be mined
out of the world.
{% endhint %}

## Cores inside protected regions

This is the one place the permissions interact with another plugin. A protection plugin cancels
the left click inside its regions, and a cancelled left click means the client never starts
breaking - so without special handling, a core inside a spawn region would simply be immune.

SnDTC un-cancels that click for an active core when the player holds `sndtc.break`. That single
step is the entire region compatibility: there is no WorldGuard dependency and nothing to
configure. Cores are meant to live in protected areas, so this is the normal deployment.

A consequence worth knowing: because SnDTC deliberately overrides another plugin's veto on a core
block, a jail, freeze or spectator plugin cannot stop a specific player from damaging a core.
`sndtc.break` is the gate for that.
