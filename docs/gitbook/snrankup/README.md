# SnRankUp

SnRankUp is a rank ladder for Paper. Every rank lives in one file, a rank is priced in whatever
currency your server already runs, and the menu players open is a YAML file you lay out yourself.

## Features

- **One file holds the ladder.** Each rank is an entry in `rankup.yml` with an order and a display
  prefix. Orders do not have to be contiguous, so 10, 20, 30 leaves room to insert a rank later
  without renumbering anything.
- **Prices in any currency.** `hours` of playtime is always available and needs no setup. Beyond
  that, a rank can cost Vault money, or any balance a plugin exposes as a PlaceholderAPI
  placeholder.
- **Charges are reversible.** Requirements are charged in the order you wrote them, and if a later
  charge is refused, everything already taken is handed back.
- **Two menus, one switch.** `menu.mode` picks between a single dynamic button for the next rank
  and the whole ladder paginated, one slot per rank in four states.
- **Menus are yours.** Both menus are ordinary `guis/*.yml` files: a title, rows, a character grid
  for the layout, and items you can move, restyle or delete.
- **Rewards use the full action set.** Broadcasts, messages, console commands, titles, action bars,
  sounds and a chance guard, one line each.
- **A live leaderboard.** The top of the ladder is rebuilt off the server thread and published as
  placeholders that work with no requesting player, so holograms and Discord bridges read it.

## Optional integrations

- **Vault**: unlocks the `vault` currency type, so a rank can cost money from any economy plugin.
  Without it, that currency is skipped at load with a warning and requirements naming it are
  dropped, which makes the rank cheaper rather than unreachable.
- **PlaceholderAPI**: resolves `%snrankup_*%` anywhere on the server, and unlocks the `placeholder`
  currency type. Without it the plugin still works, and its own `{req_*}` and `{missing_*}` tokens
  still render inside SnRankUp menus.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
