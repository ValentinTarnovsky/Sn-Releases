# Commands

Everything runs through the root command `/companions`, aliased `/companion`. The alias list lives in
`command.aliases` in `config.yml` and is authoritative at runtime, so you can rename or extend
it without touching the jar. Bare `/companions` opens the storage menu for a player and prints the
help for the console.

Arguments in `<angle brackets>` are required. Arguments in `[square brackets]` are optional.

## Player commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/companions` | `sncompanions.use` | Opens the companion menu |
| `/companions eggs [egg]` | `sncompanions.eggs` | Opens the companion eggs shop. With no argument it shows `eggs.default`; the optional `[egg]` tab-completes over the ids `eggs.yml` declares |
| `/companions toggle` | `sncompanions.toggle` | Shows or hides your own companions |
| `/companions hide` | `sncompanions.hide` | Shows or hides other players' companions |
| `/companions help` | `sncompanions.use` | Shows the generated help |
| `/companions reload` | `sncompanions.admin.reload` | Reloads the configuration |
| `/companions debug` | `sncompanions.admin.debug` | Toggles the debug output |

## Admin commands

Every admin action sits under `/companions admin <action>`. The `<instance>` argument is the companion
instance id shown by `/companions admin list`, not the companion type id.

### Inspection

| Command | Permission | Description |
|---------|-----------|-------------|
| `/companions admin info <player>` | `sncompanions.admin.info` | Shows a summary of a player's companions |
| `/companions admin list <player> [page]` | `sncompanions.admin.list` | Lists every companion a player owns |
| `/companions admin companion <player> <instance>` | `sncompanions.admin.companion` | Shows the full card of one companion |
| `/companions admin open <player>` | `sncompanions.admin.open` | Opens a player's own companion menu on their screen |

### Editing one companion

| Command | Permission | Description |
|---------|-----------|-------------|
| `/companions admin setlevel <player> <instance> <level>` | `sncompanions.admin.setlevel` | Sets the level of one companion |
| `/companions admin setexp <player> <instance> <exp>` | `sncompanions.admin.setexp` | Sets the experience of one companion |
| `/companions admin settype <player> <instance> <companion>` | `sncompanions.admin.settype` | Turns one companion into another companion |
| `/companions admin setowner <player> <instance> <target>` | `sncompanions.admin.setowner` | Moves one companion to another player |
| `/companions admin equip <player> <instance>` | `sncompanions.admin.equip` | Equips one companion of a player |
| `/companions admin unequip <player> <instance>` | `sncompanions.admin.unequip` | Sends one companion of a player back to storage |
| `/companions admin removecompanion <player> <instance>` | `sncompanions.admin.removecompanion` | Deletes one companion of a player |


### Bulk actions

| Command | Permission | Description |
|---------|-----------|-------------|
| `/companions admin clear <player> <what> [group]` | `sncompanions.admin.clear` | Empties a player's storage, slots, one group or everything |
| `/companions admin reset <player>` | `sncompanions.admin.reset` | Puts a player back to their first day |

The `<what>` argument of `clear` is `storage` (every companion that is not equipped), `equipped`
(sends every equipped companion back to storage), `group` (one whole group, named by the optional
`[group]` argument) or `all` (the entire collection).

### Granting

| Command | Permission | Description |
|---------|-----------|-------------|
| `/companions admin give <player> <companion> [amount] [level]` | `sncompanions.admin.give` | Gives companions to a player |
| `/companions admin openegg <player> <egg> [amount]` | `sncompanions.admin.openegg` | Opens companion eggs for an online player without charging them; the companions go to their storage. This is the command a shop, a crate or a vote reward runs |
| `/companions admin slots <mode> <player> <amount>` | `sncompanions.admin.slots` | Adds or sets the equip slots a player bought; they add on top of `max(base, permission)`, with the total capped at `slots.max-count` |
| `/companions admin storage <mode> <player> <amount>` | `sncompanions.admin.storage` | Adds or sets the storage a player bought; it adds on top of `max(base, permission)`, with the total capped at `storage.max-capacity` |

The `<mode>` argument is `give` or `set`. Capacity cannot be taken: lowering a purchase is a
`set`.

## Silent flags

Every `/companions admin` command accepts two optional flags, in any order and combinable:

| Flag | Effect |
|---|---|
| `-s` | Do not message the target player |
| `-sf` | Do not message yourself |

```
/companions admin give Bob ember_fox 3 5 -s -sf
/companions admin openegg Bob basic_egg 10 -s
/companions admin clear Bob all -sf
```

{% hint style="info" %}
`-sf` silences **success confirmations only**. An error, a refusal (storage full, no such companion, an
unknown egg) or a failed query always reaches the admin who ran the command - an admin command that
fails in silence is the worst possible outcome.
{% endhint %}

`-s` silences the message the RECEIVER gets. One command sends one: `give`
(`messages.companion-received`). `openegg` only looks like a second: what the player reads is the
egg's own reward line, announced by the egg engine exactly as it would be for a bought egg, so `-s`
does not touch it - and neither does it touch the other lines an open can produce, such as a
storage that filled or companions that could not be saved, which come from the same engine.
The other sixteen have never messaged their target, so `-s` is
accepted there and simply has nothing to silence. An offline receiver is never messaged either way.

{% hint style="warning" %}
Put the flags **last**. Everything after the first flag is treated as part of the flag tail, so
`/companions admin give Bob ember_fox -s 5` grants one companion, not five - the `5` is read as trailing noise.
Only the four commands with an optional argument (`list`, `clear`, `give`, `openegg`) are
sensitive to the order; the rest have nothing to confuse.
{% endhint %}

### Tab completion

Both flags are declared as the last two optional parameters of every `/companions admin`
command, so they complete on tab and show up in the usage line as `[-s] [-sf]`. Type a command's
own arguments, press tab, and you get `-s` and `-sf`; press tab again after typing one and you
still get the other. Declaring the two slots is what reaches past a command's last real argument:
completion stops there, so on the thirteen commands whose arguments are all required an undeclared
flag would be suggested nowhere and have to be typed from memory.

Inside the position of a real optional argument (the `[page]` of `list`, the `[amount]` of `give`)
the flags appear only once you type a leading `-`, so they never crowd the list of values you
were actually reaching for. A trailing token that is neither an argument nor a flag is
accepted in silence.

The flags do not apply to `/companions reload`, `/companions help` or `/companions debug`, which SnLib owns at root
level, nor to `/companions toggle` and `/companions hide`, which act on you alone and have no second party to
silence.

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/companions admin removecompanion` deletes one companion,
`/companions admin clear` empties a whole category, and `/companions admin reset` wipes a player's companions
and capacities back to a first-join state.
{% endhint %}
