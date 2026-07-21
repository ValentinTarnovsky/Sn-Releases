# SnElementalGems

SnElementalGems is a gem-economy plugin for Paper servers. Players earn a custom gem currency from combat, mining, farming, and fishing, then spend it in a shop and on account-wide upgrades.

## Features

- Earn gems from killing mobs, breaking blocks, fishing, and automated grinder deaths, each with its own configurable rate.
- Spend gems in a category shop GUI and on five account-wide upgrades: Fortune, Luck, Discount, Charity, and Execution.
- Withdraw gems into physical, redeemable items, and pay gems to other online players.
- Rank the richest players with a cached leaderboard, exposed through `/gems top` and placeholders.
- Store balances in SQLite or MySQL, cached in memory for instant reads.
- Manage balances with admin give, add, remove, and set commands, including a `*` target for everyone online.

## Optional integrations

- **PlaceholderAPI**: exposes the `%snelementalgems_*%` placeholders. When absent, those tokens stay raw.
- **Vault**: registers gems as the server economy when `vault.hook` is on. When absent, gems stay an internal currency.
- **HeadDatabase**: lets the gem item use a custom head via `SKULL:` or `HDB:` in its material. When absent, the item uses its configured material.
- **RoseStacker**: sizes automated drops to a stacked mob's real stack size. When absent, each death counts as one.
- **WildStacker**: an alternative stacker for automated drop sizing. When absent, each death counts as one.
- **WildTools**: grants gems from harvester-hoe block breaks. When absent, only normal block breaks drop gems.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
