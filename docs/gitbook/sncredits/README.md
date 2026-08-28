# SnCredits

SnCredits is a credits economy for a Velocity network. The proxy owns every balance, every shop
and every transaction. Your backends only draw the menus and answer placeholders.

One jar covers both halves. You install the same file on the proxy and on every Paper backend.

## Features

- **Network-wide balances.** One database behind the proxy, shared by every server on it. Credits
  are whole numbers, so no rounding drifts between servers.
- **A shop per backend.** Every registered server gets its own shop file, with its own catalogue,
  its own prices and its own Discord webhook.
- **Purchase prerequisites.** An item can require that the buyer already owns another one, counted
  on the backend they are standing on.
- **Coinflip.** An animated menu, bet bounds you set, and a broadcast when a bet is created or won.
- **Leaderboard.** A paginated menu backed by a refreshing cache, with a permission that keeps
  staff off the list.
- **Purchase tooling.** Read a player's history, re-run the commands of past purchases, and
  summarise what a backend sold.
- **Bypass mode.** Try a shop item without paying for it, logging it or announcing it.

The currency name, its symbol and the root command all live in `config.yml`. Rename the currency
and the shop, the coinflip and the leaderboard follow it.

## Optional integrations

- **PlaceholderAPI**: on each backend, registers the `%sncredits_...%` expansion for scoreboards,
  tab and menus. Without it the placeholders stay unparsed and nothing else changes.
- **LuckPerms**: on the proxy, hides players who hold the leaderboard exempt permission. Without
  it the top list is never filtered.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
