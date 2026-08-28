# Permissions

SnCredits checks every permission on the proxy, and the backend half checks none. Grant these nodes
in your proxy permission plugin, LuckPerms on Velocity for example. Granting them on a backend has
no effect.

The plugin ships no permissions block, so none of these nodes carries a declared default. A node
you have not granted is simply not held.

| Permission | Default | Description |
|-----------|---------|-------------|
| `sncredits.admin` | Not set | Every admin subcommand: give, set, remove, reset, reload, history, resend and summary |
| `sncredits.admin.bypass` | Not set | Toggle test-purchase mode with `/credits bypass` |
| `sncredits.balance.others` | Not set | Read another player's balance with `/credits balance <player>` |
| `sncredits.leaderboard.exempt` | Not set | Keep the holder off the leaderboard |

Everything else is open to every player. Checking your balance, paying somebody, the shop, the
coinflip and the leaderboard need no node at all.

{% hint style="info" %}
`sncredits.admin` does not imply `sncredits.balance.others`. Grant both to an admin who should read
other players' balances.
{% endhint %}

{% hint style="warning" %}
The exempt node is resolved through LuckPerms on the proxy. Without LuckPerms the leaderboard is
never filtered and every player appears on it. A player holding the wildcard `*` node is hidden
too. Rename the node with `leaderboard.exempt-permission` in `config.yml`.
{% endhint %}
