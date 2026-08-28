# Commands

The root command is your currency name, so the shipped configuration gives you `/credits`. The
`currency.command-aliases` list adds more names, `cr` and `creds` by default.

Running the root command with no arguments shows your own balance. `/credits help` lists what you
are allowed to run. SnCredits registers the command on the proxy, so it answers from any backend of
the network.

## Player commands

| Command | Description |
|---------|-------------|
| `/credits` | Show your own balance |
| `/credits help` | List the commands you may run |
| `/credits balance [player]` | Show your balance, or another player's. Alias: `bal` |
| `/credits pay <player> <amount>` | Send credits to another player |
| `/credits shop` | Open the shop of the backend you are standing on |
| `/credits coinflip [amount\|cancel]` | Open the bet list, create a bet, or cancel your own. Alias: `cf` |
| `/credits top` | Open the leaderboard |

`coinflip` with no argument opens the list of open bets. With an amount it creates one, and
`cancel` withdraws every bet you have waiting.

## Admin commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/credits give <player> <amount>` | `sncredits.admin` | Add credits to a player |
| `/credits set <player> <amount>` | `sncredits.admin` | Set a player's balance |
| `/credits remove <player> <amount>` | `sncredits.admin` | Take credits from a player |
| `/credits reset <player>` | `sncredits.admin` | Reset a player's balance |
| `/credits reload` | `sncredits.admin` | Re-read every configuration file and push it to the backends |
| `/credits history <player> [page]` | `sncredits.admin` | List a player's purchases, ten per page |
| `/credits resend <backend> <fromId>` | `sncredits.admin` | Re-run the commands of that backend's purchases, from that id on |
| `/credits summary <backend> [timeframe]` | `sncredits.admin` | Purchase totals and top items for one backend |
| `/credits bypass [on\|off]` | `sncredits.admin.bypass` | Toggle test-purchase mode for yourself |
| `/credits balance <player>` | `sncredits.balance.others` | Read another player's balance |

`summary` accepts `today`, `week`, `month`, `year`, a `YYYY-MM` month, or `fromId:<id>`. It uses
`month` when you leave it out.

`resend` queues the commands of any buyer who is offline, and runs them when that player next
connects to the backend.

{% hint style="info" %}
Amounts are whole numbers. A decimal amount is refused outright, and nothing is rounded for you.
{% endhint %}

{% hint style="warning" %}
`pay`, `give`, `set`, `remove` and `reset` need their target online, since they write to a live
session. `balance` and `history` accept any name that has joined before, online or not.
{% endhint %}

## From a backend console

A backend console cannot reach the proxy on its own, so it forwards your command through a player
connected to that backend. The usage line names `give`, `set`, `remove`, `reset` and `reload`.

With nobody online on that backend there is no route, and the console answers:

```
[SnCredits] No players online. Cannot forward command to proxy.
```

Run the command from the proxy console instead when that happens. There it executes directly, with
no player involved.

{% hint style="danger" %}
`set` and `reset` are destructive and cannot be undone. Both overwrite a balance outright, and
SnCredits keeps no copy of what was there before.
{% endhint %}
