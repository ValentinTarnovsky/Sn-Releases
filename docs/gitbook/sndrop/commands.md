# Commands

SnDrop has one command. The bare form is the one players use.

| Command | What it does | Permission |
|---|---|---|
| `/drop` | Opens your own drop window | `sndrop.use` |
| `/drop help` | Lists the commands you can use | `sndrop.use` |
| `/drop reload` | Re-reads `config.yml` and the language file | `sndrop.admin.reload` |
| `/drop debug` | Toggles runtime debug output | `sndrop.admin.debug` |

## `/drop`

Opens a window of `drop.duration-seconds` for the player who ran it. There is no target argument:
you cannot open a window for somebody else.

What you are told depends on the state:

- **No window running** - the window opens and you are told for how long.
- **A window is already running**, with `drop.refresh-window-if-active: false` (the default) -
  the command is refused and you are told how many seconds are left. The running window is not
  touched.
- **A window is already running**, with `drop.refresh-window-if-active: true` - the timer restarts
  from now, for the full configured duration. It does not add to the time you had left, so
  repeating the command gives a rolling window rather than an accumulating one.

Console cannot run it: a drop window belongs to a player.

## `/drop reload`

Re-reads both files. Windows that are already running are not cancelled and keep their original
end time, so a changed `drop.duration-seconds` applies only to windows opened after the reload.

Command aliases are re-read too, so adding or removing one in `config.yml` takes effect without a
restart.

## `/drop debug`

Turns runtime debug output on or off and remembers the choice in `config.yml`. With it on, the
console shows:

- why each drop attempt was blocked or allowed, and whether the warning was sent or rate-limited
- what each `/drop` did to the window, and the remaining seconds when it was refused
- how many crafting-grid and cursor stacks the sweep took on a death, whose they were, and whether
  they went to the death drops or back into a kept inventory
- the same for the sweep that returns those slots when a player disconnects

This is the answer to "the plugin ate my stack" and "why can't I drop this".

## Aliases

None ship by default. Add them under `command.aliases` in `config.yml`; that list is authoritative
and is re-read on `/drop reload`.
