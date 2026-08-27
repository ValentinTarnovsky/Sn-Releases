# Permissions

Everything is namespaced under `sngens.`. Nodes marked `true` are granted to everyone by
default, nodes marked `op` only to operators, and nodes marked `false` to nobody until you
grant them.

## Player

| Permission | Default | Description |
|------------|---------|-------------|
| `sngens.sell` | `true` | Use `/sell` |
| `sngens.shop` | `true` | Open `/gens shop` |
| `sngens.upgradegens` | `true` | Open `/gens upgrade` |
| `sngens.pickup` | `true` | Pick up your own generators |
| `sngens.recover` | `true` | Claim generators from your vault |
| `sngens.top` | `true` | Open `/gens top` |
| `sngens.hidestats` | `true` | Toggle `/gens hidestats` |
| `sngens.repairgens` | `op` | Use `/gens repair` on your own generators |

{% hint style="warning" %}
`sngens.repairgens` defaults to `op`, which is easy to miss. It only gates the bulk
`/gens repair` command. Repairing a single generator by clicking the block never needs it, so a
default player can still fix their generators one by one. Grant the node if you want the bulk
command available to everyone.
{% endhint %}

## Administration

| Permission | Default | Description |
|------------|---------|-------------|
| `sngens.admin` | `op` | The administrative subcommands: give, multipliers, slots, events, corruption, reload, pause, toplimit, wand grants, stresstest |
| `sngens.pickup.others` | `op` | Pick up another player's generators |
| `sngens.break.others` | `op` | Break a generator you do not own |
| `sngens.top.force` | `op` | Force a leaderboard refresh with `/gens forcetop` |
| `sngens.wand` | `op` | Use the admin region wand |
| `sngens.collector.admin` | `op` | Give and pick up collectors, break other players' collectors |
| `sngens.hopper.admin` | `op` | Give and pick up infinite hoppers, break other players' hoppers |
| `sngens.upgrade.admin` | `op` | Grant upgrade uses with `/gens addupgrades` |
| `sngens.upgrade.bypass` | `op` | Skip the island owner check and the upgrade usage limits |
| `sngens.command.armor.give` | `op` | `/gens armor give` |
| `sngens.command.armor.giveset` | `op` | `/gens armor giveset` |
| `sngens.command.offhand.give` | `op` | `/gens offhand give` |
| `sngens.command.debugspawn` | `op` | `/gens debugspawn` |
| `sngens.admin.handicap` | `op` | `/gens handicap` |
| `sngens.admin.wipeuser` | `op` | `/gens wipeuser`, the destructive purge |

`sngens.admin` also acts as a general override in a few places: it bypasses the island only
restriction on wands, the generator slot check when using a build wand, and the owner check when
selling a display shop with a sellwand.

## Numeric nodes

Two nodes are parents of a family. They are granted with a number attached, and the plugin reads
that number off the node itself. Both default to `false`, so nothing is granted until you say so.

### Sell multiplier: `sngens.multi.<value>`

Adds `<value>` to the player's sell multiplier. Write a decimal point as an underscore, since a
dot separates permission nodes.

| Node | Adds |
|------|------|
| `sngens.multi.1` | +1 |
| `sngens.multi.0_5` | +0.5 |
| `sngens.multi.2_25` | +2.25 |

Every matching node the player holds is summed, so a rank ladder stacks naturally:

```
default   -> nothing
vip       -> sngens.multi.0_5      (0.5)
vip+      -> sngens.multi.1        (1.0, if inherited on top of vip: 1.5)
```

A malformed node is ignored and logged, so a typo costs a warning in the console rather than a
broken sale.

### Generator slots: `sngens.max.<amount>`

Adds `<amount>` to the player's generator slot allowance, on top of
`player-defaults.max-generators` and on top of any bonus given with `/gens addmax`.

| Node | Adds |
|------|------|
| `sngens.max.10` | +10 slots |
| `sngens.max.50` | +50 slots |

Nodes are summed the same way. Whole numbers only.

{% hint style="info" %}
Permission slots are only counted while the player is online, since they come from the
permission plugin rather than from the database. Bonus slots granted with `/gens addmax` are
stored per player and always count. If `player-generator-limit.enabled` is `true`, the total is
still clamped to `player-generator-limit.limit`.
{% endhint %}

## Recipe: a VIP rank

```
# LuckPerms
/lp group vip permission set sngens.multi.0_5 true
/lp group vip permission set sngens.max.15 true
/lp group vip permission set sngens.repairgens true
```

That gives VIPs a +0.5 sell multiplier, 15 extra generator slots and the bulk repair command.
