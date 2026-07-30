# Commands

The root command is `/robots`, with `/robot` as its default alias. The alias list is configurable
under `command.aliases` in `config.yml`, and that key wins over the built-in default at runtime.

## Player commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/robots` | `sneconomyrobots.use` | Open the robots menu: active slots, storage and the claim bag |
| `/robots claim` | `sneconomyrobots.claim` | Claim every pending economy in your bag |
| `/robots help` | `sneconomyrobots.use` | Show the help menu |
| `/robots reload` | `sneconomyrobots.admin.reload` | Reload the configuration |
| `/robots debug` | `sneconomyrobots.admin.debug` | Toggle debug logging |

## Admin commands

Every admin action lives under `/robots admin` and carries its own permission, so you can grant a
single action to a rank without granting the rest.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/robots admin give <player> <robot> <tier> [amount]` | `sneconomyrobots.admin.give` | Give robot items of one type and tier |
| `/robots admin givebox <player> <tier> [amount]` | `sneconomyrobots.admin.givebox` | Give robot boxes of one box tier |
| `/robots admin giveslot <player> <amount>` | `sneconomyrobots.admin.giveslot` | Grant extra active robot slots |
| `/robots admin takeslot <player> <amount>` | `sneconomyrobots.admin.takeslot` | Remove active robot slots |
| `/robots admin setslots <player> <amount>` | `sneconomyrobots.admin.setslots` | Set how many active slots a player owns |
| `/robots admin clearstorage <player>` | `sneconomyrobots.admin.clearstorage` | Delete every robot in a player's storage |
| `/robots admin setupgrade <player> <slot> <track> <level>` | `sneconomyrobots.admin.setupgrade` | Set an upgrade level on an equipped robot |
| `/robots admin resetlimit <player> <slot>` | `sneconomyrobots.admin.resetlimit` | Clear the production counter of an equipped robot |
| `/robots admin bag <player>` | `sneconomyrobots.admin.bag` | List what a player has pending |
| `/robots admin setbag <player> <economy> <amount>` | `sneconomyrobots.admin.setbag` | Overwrite one pending balance |
| `/robots admin clearbag <player>` | `sneconomyrobots.admin.clearbag` | Drop everything pending without paying it |
| `/robots admin info <player>` | `sneconomyrobots.admin.info` | Show a player's slots, robots and capacity |
| `/robots admin list` | `sneconomyrobots.admin.list` | List every robot type loaded from `robots/` |

## Online and offline targets

`setslots`, `clearstorage`, `setbag` and `clearbag` accept an offline player. The rest need the
target online, either because they hand over an item or because they read the player's loaded slots.
`giveslot` and `takeslot` are relative, so they are online only on purpose. Use `setslots` for an
offline player.

{% hint style="danger" %}
These commands are destructive and cannot be undone: `clearstorage` deletes every robot in a
player's storage, and `clearbag` drops their whole pending balance without paying it. Neither asks
for confirmation.
{% endhint %}
