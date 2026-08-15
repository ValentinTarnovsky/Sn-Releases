# SnBattlePass

Permanent battle pass for Paper 1.20.x and 1.21.x. Players climb tiers by farming blocks with EdTools and by completing rotating challenges. Every tier can pay a free reward and a premium one, and the premium track opens with a pass they buy in game, receive from your store, or hold through a permission.

The pass is permanent. There is no season to reset, and nothing expires underneath a player.

## Features

- Free and premium reward tracks, with 54 tiers out of the box and a curve set by two numbers.
- Passive XP from EdTools block farming, with a per-tool override map so a late game drill pays more than a starter pickaxe.
- Five rotating challenge slots per player, rolled from a weighted pool you curate.
- Rewards are console commands, so any plugin's items, currency, keys or ranks can be given.
- The premium track unlocks retroactively: buying a pass opens every premium reward on tiers the player already passed.
- Any currency works. The plugin reads balances through a placeholder and charges through a console command, so Vault, tokens, gems and points all work unchanged.
- Three menus, each an editable YAML layout, each with an alternate version selected per viewer.

## Optional integrations

- **SnCrates**: unlocks the crate key challenge type. Without it, set `enabled: false` on those pool entries so they cannot roll.
- **SnGens**: unlocks the generator upgrade and repair challenge types. Without it, disable those pool entries.
- **SnEnvoys**: unlocks the envoy claim challenge type. Without it, disable those pool entries.
- **RivalPets**: a pet's buff boosts farming XP while it is active. Without it, farming XP is unboosted and nothing else changes.
- **PlaceholderAPI**: exposes progress, tier, XP, pass, playtime and per slot challenge state to other plugins, and lets menu text use placeholders. Without it, those lines render blank and the rest of the plugin works normally.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
