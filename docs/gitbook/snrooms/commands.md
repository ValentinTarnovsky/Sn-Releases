# Commands

The root is `/rooms`, and it also answers to `/room` and `/snrooms`. The alias list lives in
`config.yml` under `command.aliases` and is re-read on every `/rooms reload`.

Every subcommand is admin-facing. **Players never type a command to play** - they walk into a
room.

| Command | Permission | Description |
|---|---|---|
| `/rooms wand` | `snrooms.admin.wand` | Get the room selection wand |
| `/rooms create <id>` | `snrooms.admin.create` | Create a room from your current selection |
| `/rooms redefine <id>` | `snrooms.admin.redefine` | Replace the region of a room with your current selection |
| `/rooms delete <id>` | `snrooms.admin.delete` | Delete a room |
| `/rooms list` | `snrooms.admin.list` | List every configured room with its live state |
| `/rooms info <id>` | `snrooms.admin.info` | Show every setting of one room |
| `/rooms tp <id>` | `snrooms.admin.tp` | Teleport to the centre of a room |
| `/rooms setexit <id>` | `snrooms.admin.setexit` | Set the room's exit to where you stand |
| `/rooms set <id> <key> <value>` | `snrooms.admin.set` | Change one setting of a room |
| `/rooms open <id>` | `snrooms.admin.open` | Open a room and re-read who is inside |
| `/rooms close <id>` | `snrooms.admin.close` | Close a room and seal its shell |
| `/rooms reset <id>` | `snrooms.admin.reset` | Cancel whatever a room is doing and leave it open |
| `/rooms help [page]` | `snrooms.admin` | Paginated help, filtered by what you may use |
| `/rooms reload` | `snrooms.admin.reload` | Reload the configuration and the language file |
| `/rooms debug` | `snrooms.admin.debug` | Toggle the runtime debug output |

`help`, `reload` and `debug` are provided by SnLib on every command root, not written by this
plugin.

## Creating a room

`/rooms wand` hands out the selection wand. Left click marks corner 1, right click marks corner
2, and the box's edges are drawn in particles while you work. `/rooms create <id>` turns that
box into a room.

Ids are lower case, and may use letters, digits, `_` and `-`, up to 32 characters. An id typed
in capitals is accepted and stored lower case.

`/rooms create` does no validation of what is already inside the region - that is deliberate.
The shell only ever takes over blocks it can put back, so building a room around an existing
structure is a supported thing to do.

## Moving or resizing a room

`/rooms redefine <id>` points an existing room at a new region: mark the new box with the wand,
then run it. The room keeps its id, its exit and every setting it had - only the region changes,
and the new one may be anywhere, including another world.

The room is **reset onto the new region**, which is what makes this safe rather than clever. A
room that was sealed, counting down or fighting has a shell standing in the world and players
counted inside it, all of it describing the region it is about to stop having, so the shell comes
down and the round is cancelled before the swap. Afterwards the room is reopened and reads who is
already standing in the new area, so a room drawn around waiting players starts as soon as they
form a valid composition.

Two things worth knowing:

- a round in progress is **cancelled**, not paused. Redefine an idle room unless you mean to end
  what is happening in it;
- if the new region ends up **containing the room's exit**, the plugin says so. Leaving it there
  would teleport every future round's survivors back inside the room, so set a new one with
  `/rooms setexit <id>` from outside the region.

## Editing a room

`/rooms set <id> <key> <value>` changes one setting and saves it immediately.

| Key | Value | Meaning |
|---|---|---|
| `teams` | whole number, 1 to 1000 | How many sides fight each other |
| `team-size` | whole number, 1 to 1000 | Players per side; above 1 the sides come from SnClans |
| `material` | a block name | Block the shell is built with |
| `walls` | true / false | Seal the four vertical faces |
| `ceiling` | true / false | Seal the top face |
| `floor` | true / false | Seal the bottom face |
| `start-countdown` | a duration (`5s`, `100t`) | Wait between a full composition and the seal |
| `close-after-round` | a duration (`10s`) | How long the room stays sealed after a round ends |
| `effects` | `TYPE:amplifier,TYPE:amplifier` or `none` | Effects applied to every participant |
| `interior-build` | true / false | Whether participants may build inside during a round |
| `enabled` | true / false | Whether the room is in service at all |

`teams x team-size` is the exact number of players that starts a round. A 1v1 is `teams: 2`,
`team-size: 1`. A 3v3 is `teams: 2`, `team-size: 3`, and needs SnClans to decide who is on which
side.

### Shell materials

`material` accepts any placeable block **except** two families that a shell cannot use:

- **blocks with gravity** (sand, gravel, anvils) fall out of the wall the moment they are
  placed, so the room is not sealed and the fallen blocks end up somewhere the plugin never
  recorded;
- **fluids** flow out of it, with the same result over a wider area.

Both are refused with a message naming the reason.

## Managing a live room

- `/rooms open <id>` cancels whatever the room is doing, opens it and re-reads who is standing
  inside, so a round can form again without everyone stepping out and back in.
- `/rooms close <id>` takes the room out of play and seals it, until you open it again. This is
  a live lockdown: the shell comes down on shutdown like any other, so a restart brings the room
  back open. To take a room out of service permanently, use `/rooms set <id> enabled false`.
- `/rooms reset <id>` is the panic button: it cancels everything, takes the shell down and
  leaves the room open.

`/rooms delete <id>` resets the room before removing it, so deleting a sealed room never leaves
its blocks behind.
