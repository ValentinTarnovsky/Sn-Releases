# SnGens

SnGens is a generator tycoon plugin. A generator is a block a player places on their plot or
island. Every tick it drops items, and those items carry a sell value, so the block is a
small money printer the player owns, upgrades and defends.

Around that single idea the plugin ships a whole economy loop: a shop to buy the first tier, an
upgrade chain that walks a generator from wheat to netherite, a corruption system that breaks
generators until the owner repairs them, sellwands, storage blocks that swallow the drops before
they ever become entities, a leaderboard, timed server events and gear that boosts your
production. Everything is configured in YAML, and nothing is hardcoded.

## The loop in one minute

1. A new player joins and receives their starting generators, set in `config.yml`.
2. They place them. Each generator drops items on a global tick, by default every 20 seconds.
3. They run `/sell` or swing a sellwand at a chest to turn those drops into money.
4. They spend the money in `/gens shop`, or upgrade a placed generator with a shift + right
   click, or upgrade in bulk from `/gens upgrade`.
5. A better tier drops more valuable items, so step 3 pays more. Repeat.

## Features

- **Generator tiers and upgrades.** Every generator declares the id it upgrades into and the
  price of that step. That chain also drives shop prices, so one number changes both.
- **Corruption.** On a timer a share of placed generators break, hover a hologram over
  themselves and stop producing until the owner pays a repair fee.
- **Generator shop.** A paginated menu built automatically from your generator list, with an
  optional buy-quantity submenu that buys as many as the player can afford.
- **Selling.** `/sell` empties an inventory, right click sells from the hand, and sellwands sell
  the contents of a container in one swing. All three share one payout pipeline.
- **Sell multipliers.** Permissions, player balance data, active events, the wand itself and
  equipment all feed one multiplier, with an optional server-wide cap.
- **Wands.** A sellwand, a build wand that clones a generator down a line, an upgrade wand that
  upgrades for free or in an area, and an admin region wand that fills a cuboid.
- **Collectors and infinite hoppers.** Two storage blocks that capture generator drops. The
  collector eats every drop in its chunk before the item entity exists. The hopper scans its
  own column and locks in a fixed number of item types.
- **Equipment.** Off-hand items and four piece armor sets that grant generator speed, tier
  offset, sell multiplier, drop multiplier and lucky double drops.
- **Server events.** Timed global modifiers such as double drops, faster ticks, a tier boost, a
  sell multiplier or fully randomised drops, on a rotation or launched by command.
- **Leaderboard.** `/gens top` ranks players or entire islands by the total value of everything
  they have placed, with broadcasts when the podium changes.
- **Generator vault.** Generators removed by a pickup, an island kick or a disband are stored
  instead of destroyed, and the owner claims them back with `/gens recover`.
- **Performance first.** Drops are merged per chunk into single stacked item entities, the tick
  loop is chunk bucketed, database work is async, and the plugin runs on Folia.

## Optional integrations

- **SuperiorSkyblock2**: unlocks island mode. The leaderboard can rank islands instead of
  players, placement and wand use can be restricted to islands you belong to, island members can
  upgrade each other's generators, and generators are returned to the vault when a member is
  kicked, banned, leaves or the island is disbanded. Without it every island option is a no-op
  and the plugin ranks individual players.
- **SnDisplayShops**: a sellwand swung at a display shop sells the generator drops stored in
  that shop, instead of falling through to the generic container path. Only the shop owner, or
  a player with `sngens.admin`, may do it. Without it the click is treated as a normal block.

{% hint style="info" %}
Vault, PlaceholderAPI and DecentHolograms are required, not optional. See
[Installation](installation.md).
{% endhint %}

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
