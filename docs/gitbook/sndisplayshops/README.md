# SnDisplayShops

SnDisplayShops is a player shop plugin for Paper. A player places a block, sets a price on it, and
it trades on its own with a rotating hologram over it. There is no command to learn: the shop is
the block, and right-clicking it is the whole interface.

## Features

- **A shop is a placed block.** Players get a tagged item, place it, and that is a shop. Identity
  lives in the item's tag rather than in its material, so you can change the material later and
  every shop already in the world keeps working.
- **A rotating hologram over every shop.** A spinning item above a text block that shows the owner,
  the item, the price and the stock. DecentHolograms drives the text where it is installed; where
  it is not, the plugin uses a native display entity and looks the same.
- **Two menus.** Buyers see a compact offer with a one-click button and a trade-all button. Owners
  see a management screen for the item, the price, the currency, the direction, pausing, the stock
  grid and pickup.
- **Buy or sell.** A shop faces either way. In SELL mode it sells its stock to players; in BUY mode
  it buys from them and pays out of its owner's balance.
- **Storage without slots.** Stock is held per item variant as a 64-bit amount rather than as
  inventory slots, so a shop can hold far more of something than a chest could represent, and an
  item's enchantments and custom name are kept as their own variant.
- **As many currencies as you configure.** Each one is independently backed by an economy plugin's
  commands or by EdTools, so coins, gems and tokens can run side by side and each shop picks one.
- **Island aware.** With SuperiorSkyblock installed, shops on an island can be removed with it when
  the island disbands or when their owner loses membership. Both are switches, and both are off
  limits to guesswork: see the warnings in `config.yml`.
- **Everything is configurable.** Every line the plugin says, every number it uses and both menu
  layouts live in YAML, and your edits survive an update.

## Optional integrations

- **PlaceholderAPI**: resolves `%sndisplayshops_*%` anywhere on the server, lets currency names and
  hologram lines render per player, and is what a command-backed currency reads a balance through.
- **DecentHolograms**: takes over the floating text. Without it the plugin uses its own display
  entity and nothing is lost.
- **EdTools**: back a currency with the EdTools API instead of with commands.
- **SuperiorSkyblock**: the two island toggles above. Detected by its API class, so both the fork
  and upstream work.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
