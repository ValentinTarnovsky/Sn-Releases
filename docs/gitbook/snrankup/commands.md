# Commands

Everything lives under `/rankup`. It answers to `ru` and `rank` as well, and that alias list is a
config value, so you can change it and reload.

A bare `/rankup` opens the menu, which is the path players are meant to use. The rest of the tree
is administrative: it corrects a player's position on the ladder, and none of it charges anything.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/rankup` | `snrankup.use` | Opens the rank menu |
| `/rankup force <player>` | `snrankup.admin.force` | Advances a player one rank and fires that rank's rewards |
| `/rankup set <player> <rank>` | `snrankup.admin.set` | Writes a rank onto a player, with no charge and no rewards |
| `/rankup reset <player>` | `snrankup.admin.reset` | Puts a player back on the starting rank |
| `/rankup bypass [player]` | `snrankup.admin.bypass` | Toggles ignoring rank requirements, for you or for someone else |
| `/rankup reload` | `snrankup.admin.reload` | Re-reads every managed file and rebuilds the ladder |
| `/rankup help` | `snrankup.use` | Lists the subcommands you may run |
| `/rankup debug` | `snrankup.admin.debug` | Toggles the debug output described in `config.yml` |

Every argument tab-completes. `<player>` completes to online players, and `<rank>` completes to the
rank keys your `rankup.yml` declares.

## What each admin command actually does

`force` is the only one that behaves like a real rankup: it moves the player up one step and fires
the rewards of the rank they reach. It never charges the price, so it is the way to hand somebody a
rank they earned outside the plugin.

`set` writes a position. Nothing is charged and nothing fires, which makes it the tool for fixing a
mistake rather than for granting a promotion.

`reset` writes the starting rank, the one with the lowest order. Same rules as `set`: no charge, no
rewards.

`bypass` is a toggle held in memory. While it is on, that player's rankups skip the requirement
check and pay nothing. It is cleared when the plugin shuts down, so it never outlives a restart.

{% hint style="info" %}
`force` and `set` need the player to be online, because both complete against the online player
list.
{% endhint %}

{% hint style="warning" %}
`/rankup reset` cannot be undone. The player's stored rank is replaced by the starting rank, and
the plugin keeps no history of what it was.
{% endhint %}
