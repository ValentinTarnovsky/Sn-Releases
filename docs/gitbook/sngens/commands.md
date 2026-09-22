# Commands

Everything lives under one root command, `/sngens`, plus a standalone `/sell`.

The root accepts these aliases, so pick whichever your players already type:

```
/sngens   /sngen   /ngens   /ngen   /gens   /gen   /generator   /generators
```

This page uses `/gens` throughout. Several subcommands have aliases of their own, listed in the
tables below, so `/gens pickup` and `/pickupgens` do the same thing.

`/gens help` prints the list of subcommands the sender can actually use, so a player never sees
an admin command. Both the description and the usage line of each entry can be rewritten in
`lang/messages_<code>.yml` under `help-entries.<subcommand>`.

## Player commands

| Command | Aliases | Permission | What it does |
|---------|---------|------------|--------------|
| `/gens help` | | none | List the commands you can use |
| `/gens shop` | `genshop`, `gensshop` | `sngens.shop` | Open the generator shop |
| `/gens upgrade` | `upgradegens` | `sngens.upgradegens` | Open the bulk upgrade menu |
| `/gens pickup [player]` | `pickupgens`, `pickupgen`, `pickgens`, `pickgen` | `sngens.pickup` | Pick up all your placed generators into your vault |
| `/gens recover [all]` | | `sngens.recover` | Claim generators stored in your vault |
| `/gens repair [player]` | `repairgens`, `repairgen`, `genrepair`, `gensrepair` | `sngens.repairgens` | Repair all of your corrupted generators |
| `/gens top` | | `sngens.top` | Open the leaderboard |
| `/gens hidestats` | | `sngens.hidestats` | Toggle hiding your own stats placeholders |
| `/sell` | | `sngens.sell` | Sell every generator drop in your inventory |

A few of these deserve a note.

**`/gens pickup [player]`** with no argument picks up your own generators. Passing a player name
requires `sngens.pickup.others` and picks up theirs. Generators are never destroyed by this: they
land in the target's vault and the owner claims them with `/gens recover`.

**`/gens recover`** claims what fits in the inventory and tells the player how many are still
stored. `/gens recover all` claims everything, and anything that does not fit is dropped on the
ground, so it asks for confirmation first: run it twice within the confirmation window. The
same command also hands back collectors and infinite hoppers stored by an island kick, leave,
ban or disband, plus collectors stored by an admin pickup. They are reported on their own line,
and every collector comes back at the level it was bought to.

**`/gens repair`** repairs your own corrupted generators. Passing a player name requires
`sngens.admin`.

{% hint style="info" %}
`/sell` is registered over any existing `/sell` on the server, on purpose, so the generator
sell pipeline is always the one that answers. If another plugin owns `/sell` on your server,
give one of them a different name.
{% endhint %}

## Admin commands

### Generators

| Command | Permission | What it does |
|---------|------------|--------------|
| `/gens give <player> <id> [amount]` | `sngens.admin` | Give generator items by id |
| `/gens removegenerators <player>` | `sngens.admin` | Remove every generator that player has placed |
| `/gens startcorruption` | `sngens.admin` | Run a corruption wave immediately |
| `/gens pause` | `sngens.admin` | Toggle the generator tick globally. No drops are produced while paused |
| `/gens reload` | `sngens.admin` | Reload every config file and re-read the settings |

```
/gens give Notch diamond_generator 16
```

`/gens pause` lives in memory only. A restart always comes back unpaused.

### Inspection

| Command | Permission | What it does |
|---------|------------|--------------|
| `/gens inspect <player\|uuid>` | `sngens.admin.inspect` | Open a read-only menu of that player's generators |

```
/gens inspect Notch
/gens inspect 069a79f4-44e9-4726-a5be-fca90e38aaf5
```

Pass the UUID for a player who has not joined in a long time. The server only remembers the
names of recent players.

The menu looks like `/gens upgrade`, one item per tier with the amount, but nothing in it changes
anything. Only the arrows and the two toggles react to clicks.

The view toggle switches between the player's own generators and those of their whole island.
It only appears when the player has an island. The sort toggle switches between most owned
first and upgrade chain order. The summary item shows the island, the totals, the corrupted
count and the player's slots.

The data is read from the database when you run the command, so offline players are included,
even when their island is not loaded. Run the command again to refresh the menu. For an offline
player, the slot maximum leaves out `sngens.max.<n>` permission bonuses.

### Player economy

| Command | Aliases | Permission | What it does |
|---------|---------|------------|--------------|
| `/gens addmultiplier <player> <amount> [-s]` | `addmulti` | `sngens.admin` | Add to a player's stored sell multiplier |
| `/gens removemultiplier <player> <amount>` | `removemulti` | `sngens.admin` | Subtract from it |
| `/gens setmultiplier <player> <amount>` | `setmulti` | `sngens.admin` | Set it to an absolute value |
| `/gens addmax <player> <amount> [-s]` | | `sngens.admin` | Add bonus generator slots |
| `/gens removemax <player> <amount>` | | `sngens.admin` | Remove bonus slots |
| `/gens resetmax <player>` | | `sngens.admin` | Reset the bonus slot pool to zero |
| `/gens handicap <player> <percent>` | | `sngens.admin.handicap` | Silently scale a player's payout |
| `/gens handicap list` | | `sngens.admin.handicap` | List every player with a handicap |
| `/gens handicap clear <player>` | | `sngens.admin.handicap` | Remove that player's handicap |

The `-s` flag on `addmultiplier` and `addmax` is silent mode: the change is applied but the
target is not told about it.

```
/gens addmultiplier Notch 0.5      # Notch is told
/gens addmultiplier Notch 0.5 -s   # Notch is not told
```

**Handicap** is a payout correction, not a multiplier. The percentage is applied to the final
money amount after everything else, accepts `-100` to `1000`, and is deliberately invisible: the
`{multiplier}` the player sees in their sell message does not change. Use it to slow down one
account without announcing it.

```
/gens handicap Notch -40    # Notch earns 40% less per sale
/gens handicap Notch 25     # Notch earns 25% more per sale
/gens handicap clear Notch  # back to normal
```

### Wands

| Command | Aliases | Permission | What it does |
|---------|---------|------------|--------------|
| `/gens sellwand <player> <multiplier> <uses>` | `sellwands` | `sngens.admin` | Give a sellwand |
| `/gens buildwand <player> <distance> <uses>` | `buildwands` | `sngens.admin` | Give a build wand |
| `/gens upgradewand <player> <free\|radius> <uses> [radius]` | `upgradewands` | `sngens.admin` | Give an upgrade wand |
| `/gens wand <give\|pick\|fill\|clear>` | | `sngens.wand` | The admin region wand |

`uses` is the number of charges. Pass `-1` for an unlimited wand, which renders as the
`unlimited-placeholder` text from `wands.yml`.

```
/gens sellwand Notch 2 500          # 2x multiplier, 500 charges
/gens sellwand Notch 1.5 -1         # 1.5x, unlimited
/gens buildwand Notch 10 25         # clones a line of 10 generators, 25 charges
/gens upgradewand Notch free 50     # 50 free single upgrades
/gens upgradewand Notch radius 20 5 # 20 charges, 5x5 area per click
```

Giving a wand to a player who already holds a matching one folds the charges into the existing
item instead of handing out a second wand. Sellwands match by multiplier, build wands by
distance, upgrade wands by type and radius.

The admin region wand works like a WorldEdit selection: `give` hands you the wand, left click
sets the first corner and right click the second, `pick` opens a menu to choose which generator
to fill with, `fill` fills the cuboid, and `clear` drops the selection.

### Equipment

| Command | Permission | What it does |
|---------|------------|--------------|
| `/gens offhand give <player> <id>` | `sngens.command.offhand.give` | Give an off-hand item from `offhands.yml` |
| `/gens armor give <player> <setId> <piece>` | `sngens.command.armor.give` | Give one armor piece |
| `/gens armor giveset <player> <setId>` | `sngens.command.armor.giveset` | Give all four pieces of a set |

```
/gens offhand give Notch swift
/gens armor give Notch greed_set helmet
/gens armor giveset Notch prosper_set
```

### Server events

| Command | Permission | What it does |
|---------|------------|--------------|
| `/gens startevent <id\|random>` | `sngens.admin` | Start an event now |
| `/gens stopevent` | `sngens.admin` | End the active event |

```
/gens startevent gold_rush
/gens startevent random
```

An event marked `only-by-command: true` in `events.yml` never appears in the automatic rotation
and can only be started this way.

### Leaderboard

| Command | Permission | What it does |
|---------|------------|--------------|
| `/gens forcetop` | `sngens.top.force` | Recompute the leaderboard right now |
| `/gens toplimit <id\|none>` | `sngens.admin` | Set the counting cutoff, or clear it |

`toplimit` takes a generator id. Every tier that comes after it in the upgrade chain stops
counting toward `/gens top`, which keeps the board competitive once the top players are all
maxed. The value is written back into `config.yml` and the board refreshes immediately.

```
/gens toplimit diamond_generator   # netherite no longer counts
/gens toplimit none                # every tier counts again
```

### Upgrade menu limits

| Command | Aliases | Permission | What it does |
|---------|---------|------------|--------------|
| `/gens addupgrades <player> <amount>` | `addupgrade` | `sngens.upgrade.admin` | Grant bonus upgrade uses to that player's island |

The bonus pool is separate from the daily pool and survives the daily reset. Pass `-1` to grant
unlimited uses.

### Collectors and hoppers

| Command | Aliases | Permission | What it does |
|---------|---------|------------|--------------|
| `/gens collector give <player> [amount] [level]` | `collectors` | `sngens.collector.admin` | Give collector blocks, optionally already upgraded |
| `/gens collector pickup <player>` | `collectors` | `sngens.collector.admin` | Pick up every collector that player placed |
| `/gens collector adjust <percent>` | `collectors` | `sngens.collector.admin` | Scale what the collector you look at stores |
| `/gens hopper give <player> [amount]` | `hoppers` | `sngens.hopper.admin` | Give infinite hopper blocks |
| `/gens hopper pickup <player>` | `hoppers` | `sngens.hopper.admin` | Pick up every hopper that player placed |
| `/gens hopper adjust <percent>` | `hoppers` | `sngens.hopper.admin` | Scale what the hopper you look at stores |

```
/gens collector give Notch 2 3     # two level 3 collectors (5x5 chunks each)
/gens collector adjust 10          # +10% of every stored type
/gens hopper adjust -25%           # -25% of every stored type
```

`[level]` defaults to 1 and is capped at 32. The level is kept on the item, so the collector
places back at that level. See [Collector levels](configuration.md#collector-levels).

While the owner is online, `pickup` runs the normal break flow for each block. The stored
contents are handled exactly as if the owner had broken it themselves. With the owner offline,
the collectors go to the owner's vault at their levels, claimed with `/gens recover`. Their
contents drop at each block, whoever runs the command, up to `collector.break.drop-cap` stacks
per collector. The rest is voided and logged. A hopper pickup also works on offline
owners. It drops the contents and hands the hopper item to you, or drops it at the block.

**`adjust`** is player only. Look at the block within `admin-adjust.target-range` blocks,
eight by default, and pass a percent. It accepts `10`, `+10`, `-10`, `12.5` and `10%`, down to
`-100` and with at most six decimals. Every stored item type is scaled and rounded to the
nearest whole item. A type that rounds to 0 is removed, which frees a hopper type slot.

{% hint style="warning" %}
An adjust ignores capacity, so a capacity limited collector can end above its limit. It then
stops absorbing until it is sold down. If the total would pass the storable maximum, nothing
changes.
{% endhint %}

### Diagnostics

| Command | Permission | What it does |
|---------|------------|--------------|
| `/gens debugspawn <iterations> [chunkRadius]` | `sngens.command.debugspawn` | Record drop spawn telemetry into a JSON dump |
| `/gens stresstest [confirm]` | `sngens.admin` | Fill the current chunk with generators to measure performance |

`debugspawn` samples the given number of generator ticks around you and writes
`plugins/SnGens/debug-dumps/spawn-*.json`. Use it when drops feel slow and you want numbers
rather than impressions.

### Destructive

| Command | Permission | What it does |
|---------|------------|--------------|
| `/gens wipeuser <player\|uuid> confirm [--include-island]` | `sngens.admin.wipeuser` | Erase a player's entire SnGens state |

```
/gens wipeuser Notch confirm
/gens wipeuser Notch confirm --include-island
```

{% hint style="danger" %}
`/gens wipeuser` deletes that player's generators, vault contents, multiplier, bonus slots and
storage blocks. Nothing is refunded, nothing is dropped, and there is no undo. The literal word
`confirm` is required, and `--include-island` widens the wipe to every member of their island.

`/gens removegenerators` and `/gens stresstest` are also worth care: the first deletes placed
generators without a refund, the second fills the chunk you are standing in.
{% endhint %}

## Tab completion

Every subcommand completes its own arguments, and only for senders who hold the permission. Player
names, generator ids, event ids, armor set ids, off-hand ids and the `-s` flag all complete as you
type. `adjust` suggests a few sample percents.
