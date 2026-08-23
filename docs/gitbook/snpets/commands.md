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
| `/pets admin list <player>` | `snpets.admin.list` | Lists every pet a player owns |
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
| `/pets admin slots <mode> <player> <amount>` | `snpets.admin.slots` | Adds or sets the equip slots a player bought |
| `/pets admin storage <mode> <player> <amount>` | `snpets.admin.storage` | Adds or sets the storage a player bought |
| `/pets admin currency <currency> <mode> <player> <amount>` | `snpets.admin.currency` | Changes one of a player's three balances |

The `<mode>` argument is `give` or `set` for slots and storage, and `give`, `take` or `set`
for a currency. Capacity cannot be taken: lowering a purchase is a `set`. The three currencies
are `trait-ticket`, `dice-normal` and `dice-special`.

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/pets admin removepet` deletes one pet,
`/pets admin clear` empties a whole category, and `/pets admin reset` wipes a player's pets,
capacities and currencies back to a first-join state.
{% endhint %}
