# Commands

The root command is `/antixray`, with the aliases `/axr` and `/snax`. You can change the aliases at
`command.aliases` in `config.yml`, and they apply on the next `/antixray reload` without a restart.

Every subcommand is staff-only. SnAntiXray has no player-facing commands, so the root itself is
gated behind `snantixray.admin` and a normal player never sees a help page.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/antixray help` | `snantixray.admin` | Show the help menu |
| `/antixray reload` | `snantixray.admin.reload` | Re-read `config.yml` and `worlds.yml` |
| `/antixray debug` | `snantixray.admin.debug` | Toggle runtime debug output |
| `/antixray bypass <player>` | `snantixray.admin.bypass` | Toggle a player's persisted bypass flag |
| `/antixray alerts` | `snantixray.admin.alerts` | Toggle whether you receive detection alerts |
| `/antixray check <player>` | `snantixray.admin.check` | Show a player's detection score and stored state |
| `/antixray reset <player>` | `snantixray.admin.reset` | Clear a player's detection score |
| `/antixray stats` | `snantixray.admin.stats` | Show the engine counters |

## Offline players work

`bypass`, `check` and `reset` all accept a player who is not online. They read and write the stored
row directly, so you can act on a cheater who already logged off.

## Reading /antixray stats

`stats` prints seven numbers. Some are lifetime totals since the last restart, and some are live
gauges that move both ways. The label on each line tells you which.

| Line | Meaning |
|------|---------|
| Chunks rewritten | Chunk packets this plugin changed on the way out |
| Blocks replaced | Individual blocks swapped for a filler across those packets |
| Queued block updates | Reveals waiting to be sent right now |
| Queue overflows | Times a player's reveal queue was full, each answered by resending the chunk |
| Revealed positions | Positions the reveal layer is holding, and across how many players |
| Decoy chunks cached | Cached decoy chunk sets against total cache slots |
| Decoy chunks generated | Decoy chunk sets derived since startup, cache misses included |

{% hint style="info" %}
Watch **Revealed positions**. That set is deliberately never pruned, because forgetting a position
would re-hide ore a player has already dug out and can plainly see. It grows with mining activity
and is released when the player disconnects.
{% endhint %}

{% hint style="info" %}
A climbing **Queue overflows** counter means players are exposing blocks faster than the per-tick
budget delivers them. Raise `ghost-blocks.per-player-per-tick`, or raise
`ghost-blocks.max-queued-per-player`.
{% endhint %}
