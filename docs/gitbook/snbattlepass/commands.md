# Commands

The root command is `/battlepass`, with the aliases `bp` and `pass`. Aliases are configurable through `command.aliases` in `config.yml`, and they are re-read on reload. Every argument tab-completes.

## Player commands

| Command | Description |
|---------|-------------|
| `/battlepass` | Opens the pass menu: tiers, progress, and the claim buttons |
| `/battlepass challenges` | Opens the challenge menu, where slots are rolled and tracked |

Both need `snbattlepass.use`, which is granted to everyone by default.

## Admin commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/battlepass setpass <player> <pass>` | `snbattlepass.admin.manage` | Gives a player the Gold or Diamond pass |
| `/battlepass removepass <player>` | `snbattlepass.admin.manage` | Puts a player back on the free pass |
| `/battlepass settier <player> <level>` | `snbattlepass.admin.manage` | Sets a player's tier |
| `/battlepass addtier <player> <amount>` | `snbattlepass.admin.manage` | Adds tiers to a player |
| `/battlepass addxp <player> <amount>` | `snbattlepass.admin.manage` | Adds battle pass XP |
| `/battlepass setxp <player> <amount>` | `snbattlepass.admin.manage` | Sets battle pass XP |
| `/battlepass reset <player>` | `snbattlepass.admin.reset` | Wipes one player's pass, tier, XP and claims |
| `/battlepass resetall confirm` | `snbattlepass.admin.reset` | Wipes every player's battle pass data |
| `/battlepass bypass [player]` | `snbattlepass.admin.bypass` | Toggles ignoring challenge cooldowns |
| `/battlepass givechallenge <player> <slot> <type>` | `snbattlepass.admin.challenges` | Places a challenge of a given type into a slot |
| `/battlepass reroll <player> <slot>` | `snbattlepass.admin.challenges` | Rerolls one challenge slot |
| `/battlepass reload` | `snbattlepass.admin.reload` | Re-reads the configuration files |
| `/battlepass help` | - | Lists the commands you can run |
| `/battlepass debug` | `snbattlepass.admin.debug` | Toggles debug output |

`setpass`, `removepass`, `settier`, `addtier`, `addxp`, `setxp` and `reset` also work on offline players. They read and write the player's real stored row, so an edit is never applied to a blank record.

{% hint style="danger" %}
These commands are destructive and cannot be undone: `/battlepass reset <player>` wipes one player's progress, and `/battlepass resetall confirm` wipes every player's progress on the server.
{% endhint %}
