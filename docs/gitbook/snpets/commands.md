# Commands

Everything runs through the root command `/pets`, aliased `/pet`. The alias list lives in
`command.aliases` in `config.yml` and is authoritative at runtime, so you can rename or extend
it without touching the jar. Bare `/pets` opens the storage menu for a player and prints the
help for the console.

Arguments in `<angle brackets>` are required. Arguments in `[square brackets]` are optional.

## Player commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/pets` | `snpets.use` | Opens the pet menu |
| `/pets toggle` | `snpets.toggle` | Shows or hides your own pets |
| `/pets hide` | `snpets.hide` | Shows or hides other players' pets |
| `/pets help` | `snpets.use` | Shows the generated help |
| `/pets reload` | `snpets.admin.reload` | Reloads the configuration |
| `/pets debug` | `snpets.admin.debug` | Toggles the debug output |

## Admin commands

Every admin action sits under `/pets admin <action>`. The `<instance>` argument is the pet
instance id shown by `/pets admin list`, not the pet type id.

### Inspection

| Command | Permission | Description |
|---------|-----------|-------------|
| `/pets admin info <player>` | `snpets.admin.info` | Shows a summary of a player's pets |
| `/pets admin list <player> [page]` | `snpets.admin.list` | Lists every pet a player owns |
| `/pets admin pet <player> <instance>` | `snpets.admin.pet` | Shows the full card of one pet |
| `/pets admin open <player>` | `snpets.admin.open` | Opens a player's own pet menu on their screen |

### Editing one pet

| Command | Permission | Description |
|---------|-----------|-------------|
| `/pets admin setlevel <player> <instance> <level>` | `snpets.admin.setlevel` | Sets the level of one pet |
| `/pets admin setexp <player> <instance> <exp>` | `snpets.admin.setexp` | Sets the experience of one pet |
| `/pets admin settype <player> <instance> <pet>` | `snpets.admin.settype` | Turns one pet into another pet |
| `/pets admin settrait <player> <instance> <trait>` | `snpets.admin.settrait` | Sets or clears the trait of one pet |
| `/pets admin setboost <player> <instance> <stat> <grade>` | `snpets.admin.setboost` | Sets or clears one boost grade of one pet |
| `/pets admin setowner <player> <instance> <target>` | `snpets.admin.setowner` | Moves one pet to another player |
| `/pets admin equip <player> <instance>` | `snpets.admin.equip` | Equips one pet of a player |
| `/pets admin unequip <player> <instance>` | `snpets.admin.unequip` | Sends one pet of a player back to storage |
| `/pets admin removepet <player> <instance>` | `snpets.admin.removepet` | Deletes one pet of a player |

Pass `none` as the `<trait>` or `<grade>` to clear it instead of setting it. The three boost
stats are `experience`, `level` and `buff`.

### Bulk actions

| Command | Permission | Description |
|---------|-----------|-------------|
| `/pets admin clear <player> <what> [group]` | `snpets.admin.clear` | Empties a player's storage, slots, one group or everything |
| `/pets admin reset <player>` | `snpets.admin.reset` | Puts a player back to their first day |

The `<what>` argument of `clear` is `storage` (every pet that is not equipped), `equipped`
(sends every equipped pet back to storage), `group` (one whole group, named by the optional
`[group]` argument) or `all` (the entire collection).

### Granting

| Command | Permission | Description |
|---------|-----------|-------------|
| `/pets admin give <player> <pet> [amount] [level]` | `snpets.admin.give` | Gives pets to a player |
| `/pets admin givebox <player> <box> [amount] [chance]` | `snpets.admin.givebox` | Gives pet boxes to an online player. Never refused: a box is an item, and a full pet storage only blocks OPENING it |
| `/pets admin giveallbox <box> [amount] [chance]` | `snpets.admin.giveallbox` | Gives pet boxes to every online player, all at one success chance |
| `/pets admin givescroll <player> <scroll> [amount] [levels]` | `snpets.admin.givescroll` | Gives scrolls to an online player. Never refused: a scroll is an item |
| `/pets admin slots <mode> <player> <amount>` | `snpets.admin.slots` | Adds or sets the equip slots a player bought; they add on top of `max(base, permission)`, with the total capped at `slots.max-count` |
| `/pets admin storage <mode> <player> <amount>` | `snpets.admin.storage` | Adds or sets the storage a player bought; it adds on top of `max(base, permission)`, with the total capped at `storage.max-capacity` |
| `/pets admin currency <currency> <mode> <player> <amount>` | `snpets.admin.currency` | Changes one of a player's three balances |

The `<mode>` argument is `give` or `set` for slots and storage, and `give`, `take` or `set`
for a currency. Capacity cannot be taken: lowering a purchase is a `set`. The three currencies
are `trait-ticket`, `dice-normal` and `dice-special`.

The three scrolls are `level`, `ownership` and `rarity`. `[levels]` is what ONE level scroll is
worth: it is stamped onto the stack when it is handed out, so scrolls of different sizes circulate
side by side and lowering `scrolls.level.default-levels` later never devalues one already given.
Omit it and the configured default is used; the other two scrolls ignore it. A scroll whose
`enabled` is `false` is refused even when the token is spelled correctly. See
[Configuration](configuration.md#scrolls).

## Silent flags

Every `/pets admin` command accepts two optional flags, in any order and combinable:

| Flag | Effect |
|---|---|
| `-s` | Do not message the target player |
| `-sf` | Do not message yourself |

```
/pets admin give Bob ember_fox 3 5 -s -sf
/pets admin currency dice-normal give Bob 10 -s
/pets admin clear Bob all -sf
```

{% hint style="info" %}
`-sf` silences **success confirmations only**. An error, a refusal (storage full, no such pet, an
unknown box) or a failed query always reaches the admin who ran the command - an admin command that
fails in silence is the worst possible outcome.
{% endhint %}

`-s` silences the message the RECEIVER gets. Five commands send one: `give`
(`messages.pet-received`), `givebox` and `giveallbox` (`messages.box-received`), `givescroll`
(`messages.scroll-received` / `messages.scroll-received-level`) and `currency`
(`messages.currency-received`). The other seventeen have never messaged their target, so `-s` is
accepted there and simply has nothing to silence. An offline receiver is never messaged either way.

{% hint style="warning" %}
Put the flags **last**. Everything after the first flag is treated as part of the flag tail, so
`/pets admin give Bob ember_fox -s 5` grants one pet, not five - the `5` is read as trailing noise.
Only the six commands with an optional argument (`list`, `clear`, `give`, `givebox`,
`giveallbox`, `givescroll`) are sensitive to the order; the rest have nothing to confuse.
{% endhint %}

### Tab completion

Since 1.9.0 both flags are declared as the last two optional parameters of every `/pets admin`
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

The flags do not apply to `/pets reload`, `/pets help` or `/pets debug`, which SnLib owns at root
level, nor to `/pets toggle` and `/pets hide`, which act on you alone and have no second party to
silence.

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/pets admin removepet` deletes one pet,
`/pets admin clear` empties a whole category, and `/pets admin reset` wipes a player's pets,
capacities and currencies back to a first-join state.
{% endhint %}
