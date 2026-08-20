# FAQ

### How do I update SnDTC?
Download the newer `sndtc-v*` release and replace the jar. Configs auto-merge on restart, and
`dtcs.yml` is never touched by an update - your cores are safe.

### Does it support Folia?
No, SnDTC is not Folia-compatible.

### My core does nothing when I hit it.
Check its **active** material. A core is destroyed by *breaking* its block, so the active block
must be something a player can break. `BEDROCK`, `AIR`, `WATER` and `LAVA` can all be placed but
never broken. `/sndtc setblock <core> active <a breakable block>` fixes it. SnDTC refuses those
materials now and says so, but a core created before 2.0.0 may still carry one.

If the material is fine, check `sndtc.break` - a player without it is told once and then rejected
silently.

### My core is inside a WorldGuard region and cannot be broken.
It should work with no setup. SnDTC un-cancels the left click on an active core for players
holding `sndtc.break`, which is the whole region compatibility - there is no WorldGuard
dependency and nothing to configure. If it is still immune, another plugin is cancelling the
break itself at a later priority.

### Why did my core come back at full health after a reload?
That is deliberate, and the console explains it when it happens. The damage board lives only in
memory: a reload or a restart cannot carry it. Leaving the core damaged with an empty board would
hand the entire top reward to whoever landed the next hit, so it is refilled instead.

To set the health of a core that is alive, use `/sndtc sethealth` - it keeps the damage board.

### I edited `current-health` in `dtcs.yml` and nothing happened.
For an **active** core, that edit is overwritten by the refill described above on the very reload
meant to apply it. It works for a core that is stopped or destroyed. Use `/sndtc sethealth` for a
live one.

### A core disappeared from `/sndtc list` but `/sndtc create` says it already exists.
Its section failed to load. The console names it and says why - a missing coordinate, an unknown
material, or a `max-health` that is not a usable number. Fix that line in `dtcs.yml` and reload.

Note that while it is in that state **its block is not protected**: it can be mined, exploded or
pushed like any other block, and it keeps its drops.

### My `5h` core fires at a different time every day.
That is how the interval grid works, and it is not a bug. Intervals run on a fixed grid measured
from a fixed anchor, so they land on midnight only when they divide evenly into 24 hours - `30s`,
`5m`, `1h`, `2h`, `3h`, `4h`, `6h`, `8h`, `12h`, `1d`, `2d`. `5h` still fires exactly every five
hours but walks across the day.

If the event has to happen at a set wall-clock time, use the cron form: `0 8 * * *` for 08:00
every day.

### Players are farming my core.
Work out the arithmetic. How often a payout fires is `max-health` and `cron` together, not the
`rewards` block. A 100-health core on a `5m` schedule can be soloed in about 25 seconds and comes
back 288 times a day. Raise `max-health`, lengthen `cron`, or lower the position rewards.

`limits.hit-cooldown-ms` caps how fast one player can land counted hits, but it does not change
how often the core dies overall.

### Someone won a position and got nothing.
`rewards.top-positions` is higher than the number of position lists you configured. A position
with no list is skipped - its winner gets nothing, not the default reward. SnDTC names this in
the console. Either add a `rewards.positions.<n>` list or lower `top-positions`.

Also check the console for an offline winner: `give`-style commands do nothing for a player who
has logged out. Use commands that work offline, or a mail/claim plugin, for reward lines.

### The plugin did not start and the console mentions a tag or a colour.
A colour or gradient tag with the wrong arguments in an owner-edited template - `<gradient:#8354f2>`
with a single colour, for instance. The console names which value to fix. As of 2.0.0 a bad tag
costs that one line rather than the plugin.

### Can a core's hologram show placeholders from other plugins?
Yes, and they resolve against the server. SnDTC's own `%sndtc_*%` tokens do not work there - one
hologram is shown to everyone, so there is no viewer to resolve against. Use the local `{core}`
`{health}` `{max_health}` `{percent}` `{status}` `{time}` tokens instead.

### Can I name a core with a space in it?
`/sndtc create` refuses it, because the other subcommands take the core name as a single token
and could not then address it - you would end up with a core only a text editor could manage. A
core already named that way in `dtcs.yml` still loads and still plays.

### Does deleting a core remove its block?
No. Deleting un-registers a block; it does not clear the terrain you built around it. The block
stays exactly where it is.
