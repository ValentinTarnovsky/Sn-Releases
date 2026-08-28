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
| `/companions admin setboost <player> <instance> <stat> <grade>` | `sncompanions.admin.setboost` | Sets or clears one boost grade of one companion |
| `/companions admin setowner <player> <instance> <target>` | `sncompanions.admin.setowner` | Moves one companion to another player |
| `/companions admin equip <player> <instance>` | `sncompanions.admin.equip` | Equips one companion of a player |
| `/companions admin unequip <player> <instance>` | `sncompanions.admin.unequip` | Sends one companion of a player back to storage |
| `/companions admin removecompanion <player> <instance>` | `sncompanions.admin.removecompanion` | Deletes one companion of a player |

Pass `none` as the `<grade>` to clear it instead of setting it. The three boost
stats are `experience`, `level` and `buff`.

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
| `/companions admin givebox <player> <box> [amount] [chance]` | `sncompanions.admin.givebox` | Gives companion boxes to an online player. Never refused: a box is an item, and a full companion storage only blocks OPENING it |
| `/companions admin giveallbox <box> [amount] [chance]` | `sncompanions.admin.giveallbox` | Gives companion boxes to every online player, all at one success chance |
| `/companions admin slots <mode> <player> <amount>` | `sncompanions.admin.slots` | Adds or sets the equip slots a player bought; they add on top of `max(base, permission)`, with the total capped at `slots.max-count` |
| `/companions admin storage <mode> <player> <amount>` | `sncompanions.admin.storage` | Adds or sets the storage a player bought; it adds on top of `max(base, permission)`, with the total capped at `storage.max-capacity` |
| `/companions admin currency <currency> <mode> <player> <amount>` | `sncompanions.admin.currency` | Changes one of a player's three balances |

The `<mode>` argument is `give` or `set` for slots and storage, and `give`, `take` or `set`
for a currency. Capacity cannot be taken: lowering a purchase is a `set`. The three currencies
are `trait-ticket`, `dice-normal` and `dice-special`.

## Silent flags

Every `/companions admin` command accepts two optional flags, in any order and combinable:

| Flag | Effect |
|---|---|
| `-s` | Do not message the target player |
| `-sf` | Do not message yourself |

```
/companions admin give Bob ember_fox 3 5 -s -sf
/companions admin currency dice-normal give Bob 10 -s
/companions admin clear Bob all -sf
```

{% hint style="info" %}
`-sf` silences **success confirmations only**. An error, a refusal (storage full, no such companion, an
unknown box) or a failed query always reaches the admin who ran the command - an admin command that
fails in silence is the worst possible outcome.
{% endhint %}

`-s` silences the message the RECEIVER gets. Four commands send one: `give`
(`messages.companion-received`), `givebox` and `giveallbox` (`messages.box-received`) and `currency`
(`messages.currency-received`). The other seventeen have never messaged their target, so `-s` is
accepted there and simply has nothing to silence. An offline receiver is never messaged either way.

{% hint style="warning" %}
Put the flags **last**. Everything after the first flag is treated as part of the flag tail, so
`/companions admin give Bob ember_fox -s 5` grants one companion, not five - the `5` is read as trailing noise.
Only the five commands with an optional argument (`list`, `clear`, `give`, `givebox`,
`giveallbox`) are sensitive to the order; the rest have nothing to confuse.
{% endhint %}

### Tab completion

Since 1.9.0 both flags are declared as the last two optional parameters of every `/companions admin`
command, so they complete on tab and show up in the usage line as `[-s] [-sf]`. Type a command's
own arguments, press tab, and you get `-s` and `-sf`; press tab again after typing one and you
still get the other.

Before 1.9.0 they were accepted but not declared, and completion stops at a command's last declared
argument - so on the sixteen commands whose arguments are all required they were suggested nowhere
and had to be typed from memory.

Inside the position of a real optional argument (the `[page]` of `list`, the `[amount]` of `give`)
the flags still appear only once you type a leading `-`, so they never crowd the list of values you
were actually reaching for. A trailing token that is neither an argument nor a flag is still
accepted in silence, exactly as before.

The flags do not apply to `/companions reload`, `/companions help` or `/companions debug`, which SnLib owns at root
level, nor to `/companions toggle` and `/companions hide`, which act on you alone and have no second party to
silence.

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/companions admin removecompanion` deletes one companion,
`/companions admin clear` empties a whole category, and `/companions admin reset` wipes a player's companions,
capacities and currencies back to a first-join state.
{% endhint %}
