# FAQ

### The plugin disabled itself with `[Sn] License: FAIL`

On a **first** run this is expected: SnRooms creates `plugins/.Sn-License/license.yml` and stops.
Put your key in that file and restart.

If it happens **after** a key is in place, the server could not reach the licence backend. The
check runs once at startup and needs outbound HTTPS; the plugin refuses to enable rather than
run half-licensed.

The file is shared by every Sn bundle plugin, so one key covers all of them.

### My room never starts

`/rooms info <id>` shows what it is waiting for, and the players inside see the reason on their
action bar. The usual causes:

- **Not enough players.** A round needs exactly `teams x team-size` people inside.
- **Too many.** More than that number is also a refusal, not a start.
- **The sides do not balance.** Six players in a 2x3 room can still be four against two.
- **A room of bigger sides with no clan data.** `team-size` above 1 needs SnClans; without it
  the room stays open and logs one warning naming the room.
- **The room is off.** `enabled: false` in `/rooms info`.
- **The region is too big to seal.** Over `shell.max-blocks` the room refuses and stays open,
  with a warning naming the count it needed.

### A player logged out inside a room and now the room is stuck

It is not. A player who disconnects is taken out of the round straight away, and one who logs
back in inside a room is counted the moment they arrive.

### Can players break out of a sealed room?

Not by hand: the shell and, unless `interior-build` is on, the whole interior are protected while
a room is not open. Explosions, pistons, fluids, fire, endermen and falling blocks cannot take a
shell block either.

The one thing to be aware of is the **floor**, which is off by default. A region drawn above the
ground has an open bottom, and a player can walk out through it. Turn `floor` on for a room that
needs to be genuinely closed.

### What happens if the server crashes mid-round?

Every sealed shell is recorded in `shell-state.yml` as it goes up, and the blocks are put back on
the next boot. You should come back to open rooms, not to a world full of glass.

The one gap is a crash **during** the seal itself, in the couple of seconds a very large shell
takes to go up: blocks placed in that window may not be recorded yet. It does not apply to
ordinary room sizes, which seal in a single pass.

### A room's world is not loaded at startup

Its entry in `shell-state.yml` is deliberately kept rather than dropped, and the blocks go back
once the world is available. This is the normal case on a server whose worlds are loaded by a
manager after plugins. Do not clear the file by hand to "fix" it - that is what would make those
blocks permanent.

### Do I need to worry about `shell-state.yml` growing?

No. It only ever holds entries for rooms that are sealed **right now**, and an entry is removed
as soon as its shell comes down. In normal operation the file is empty.

### Will the shell destroy what I built inside the region?

No. It only takes over blocks it can put back exactly, and it records every one it changes.
Anything else on a shell position is left untouched - which means a wall you built yourself
becomes part of the room's wall rather than being replaced by glass.

### Can I use sand or water as the shell material?

No, and it is refused with a message saying why. Blocks with gravity fall out of the wall and
fluids flow out of it; in both cases the room is not sealed and the escaped blocks end up
somewhere the plugin never recorded, so they cannot be cleaned up.

### An admin gets teleported out of a room they teleported into

`/rooms tp` and `snrooms.admin.bypass` are separate permissions. Without bypass, an admin who
teleports into a room that is not open is treated as an intruder and put back outside. Grant
bypass to staff who need to be inside a live room.

### Two rooms overlapping

Supported but not recommended. Where regions overlap, the first room in file order wins the
lookup, which means a room that is open can leave its sealed neighbour's shell unprotected
inside the overlapping volume. Keep regions apart.

### A round is not ending

`round.max-duration` (10 minutes by default) ends it as a draw and reopens the room. If you have
set it to `0s` and also turned `leaving-counts-as-defeat` off, a player who walks out and stays
out can keep a round running - that combination is what the safety net exists for. `/rooms reset`
clears it immediately.

### I marked the wrong region, or I need to move a room

`/rooms redefine <id>` with a fresh wand selection. The room keeps its id, its exit and every
setting; it is reset onto the new region and reopened, so anything it was doing is cancelled
first. Deleting and recreating the room would work too, but it would hand the room back the
`default-room` template and lose everything `/rooms set` had changed.

### Changing `default-room` did nothing to my rooms

That section is the template `/rooms create` copies from. Rooms that already exist keep their
own values in `rooms.yml`; change those with `/rooms set`.
