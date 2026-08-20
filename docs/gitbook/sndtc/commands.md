# Commands

The root command is `/sndtc`, with `/dtc` as the shipped alias. Aliases are configurable via
`command.aliases` in `config.yml` and are re-read on `/sndtc reload`.

Every core argument is matched case-insensitively, so `MyCore` and `mycore` are the same core.

| Command | Description |
|---------|-------------|
| `/sndtc` | Show the help for everything you have permission to use |
| `/sndtc create <name>` | Turn the block you are looking at into a core |
| `/sndtc delete <core>` | Un-register a core. The block stays in the world |
| `/sndtc start <core>` | Bring a core back to full health now |
| `/sndtc stop <core>` | Take a core down by hand |
| `/sndtc setblock <core> <active\|inactive> <material>` | Change one of a core's two materials |
| `/sndtc sethealth <core> <amount>` | Resize the health pool and refill it |
| `/sndtc setcron <core> <schedule>` | Change the regeneration schedule |
| `/sndtc setrange <core> <range>` | How far the boss bar and action bar reach |
| `/sndtc sethologram <core>` | Toggle this core's floating text |
| `/sndtc list` | List every core with its state and health |
| `/sndtc info <core>` | Everything one core is configured with |
| `/sndtc reload` | Reload every configuration file |

## Notes on individual commands

### `/sndtc create <name>`

You must be looking at a block within 5 blocks. Names may hold letters, digits, underscores and
hyphens - **not spaces**, because the other subcommands could not then address the core, and not
dots, which YAML reads as a path separator.

The command refuses a name that already exists **on disk**, not just one in the registry. A core
whose section failed to load is missing from `/sndtc list` while its section is still in
`dtcs.yml`, and re-creating it would overwrite the very core the console told you to go and fix.

### `/sndtc start <core>`

Starting a core wipes its damage board. That is deliberate - it is how a fresh event begins - but
it means a redundant `/sndtc start` in the middle of a fight erases everyone's standing, and the
destruction that follows pays the wrong players. A core that is already running is reported
rather than started again.

### `/sndtc stop <core>`

Stopping is a pause, not a reset: health and the damage board are left exactly as they stand, and
the regeneration is still scheduled, so no core is ever left permanently dark. If players were
mid-fight, the command says how many - their totals live only in memory and nobody is paid for
them.

### `/sndtc sethealth <core> <amount>`

Resizes the pool and refills the core to the new maximum, **keeping** the damage board, so a
resize mid-event does not erase who has been fighting.

### `/sndtc setblock <core> active <material>`

The active material must be a block players can **break**, because breaking it is what damages
the core. `BEDROCK`, `AIR`, `WATER` and `LAVA` can all be placed but never broken, so a core
wearing one of them could never be destroyed and would never pay. The command refuses them for
the active slot and says so.

The inactive slot has no such rule - it is not meant to be hittable, and `AIR` is a supported
choice there.

### `/sndtc delete <core>`

The block is left standing in the world on purpose: deleting a core un-registers a block, it does
not clear the terrain you built around it.

## When a change could not be saved

Every command that changes something reports whether the change reached `dtcs.yml`. If it could
not - the file does not parse, or the core's name cannot be used as a storage key - you are told
in the same breath as the success line, because the change is live right now and will be lost on
restart. The console says which of the two it was.
