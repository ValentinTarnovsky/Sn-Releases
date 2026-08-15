# Permissions

Every node defaults to `op`. `snrooms.admin` carries all of them as children, so a group granted
the parent gets the whole surface.

| Permission | Default | Description |
|---|---|---|
| `snrooms.admin` | op | Full administrative access; also gates the `/rooms` root itself |
| `snrooms.admin.wand` | op | Allows `/rooms wand` |
| `snrooms.admin.create` | op | Allows `/rooms create` |
| `snrooms.admin.redefine` | op | Allows `/rooms redefine` |
| `snrooms.admin.delete` | op | Allows `/rooms delete` |
| `snrooms.admin.list` | op | Allows `/rooms list` |
| `snrooms.admin.info` | op | Allows `/rooms info` |
| `snrooms.admin.tp` | op | Allows `/rooms tp` |
| `snrooms.admin.setexit` | op | Allows `/rooms setexit` |
| `snrooms.admin.set` | op | Allows `/rooms set` |
| `snrooms.admin.open` | op | Allows `/rooms open` |
| `snrooms.admin.close` | op | Allows `/rooms close` |
| `snrooms.admin.reset` | op | Allows `/rooms reset` |
| `snrooms.admin.reload` | op | Allows `/rooms reload` |
| `snrooms.admin.debug` | op | Allows `/rooms debug` |
| `snrooms.admin.update` | op | Receive the update notice on join |
| `snrooms.admin.bypass` | op | Enter a room that is not open without being sent back outside, and build inside it |

`snrooms.admin.reload`, `.debug` and `.update` are the three nodes SnLib derives from the plugin
name for the subcommands it injects into every command root.

## The `/rooms` root is gated by the parent

The root itself requires `snrooms.admin`. That means the per-action nodes narrow what a
staff member can do **once they already have the parent** - they do not, on their own, let you
hand out a single action to someone who otherwise has no access to `/rooms`.

## `snrooms.admin.bypass` is not a command gate

It changes how the world treats you, and it is the one node worth understanding:

- you walk into a room that is starting, fighting or closed **without being evicted**;
- you are **never counted into a composition**, so an admin standing in a 1v1 to watch does not
  read as a third player and does not stop the round;
- you are **exempt from the block protection**, so you can repair a room without opening it.

Give it to staff who need to observe or fix rooms. Do not give it to players: a bypass holder
inside a fighting room is invisible to the round.

## Players need nothing

There is no player-facing permission. Taking part in a round is walking into a region, and the
plugin does not check anything to let you do that.
