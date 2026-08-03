# SnSimpleVouchers

Click-to-claim voucher items, one YAML file per voucher.

Every voucher is its own file under `vouchers/`. Drop files in, group them in folders, and each one becomes a physical item a player right-clicks to run its reward commands. A voucher can hand out the same thing every time, pick one reward at random, pick one by weight, or give a different reward to different players based on a PlaceholderAPI condition.

## What it does

- **One file per voucher.** The filename is the voucher id; a subfolder is a category.
- **Right-click to claim**, in the air or at a block, main hand.
- **Bulk claim**: a voucher carrying an `amount:` consolidates a whole stack into **one** command execution. 64 money vouchers are a single `eco give`, not 64. No shift required.
- **Category aggregation**: one click can also consume sibling vouchers from the same folder that share a reward list, summing their amounts into a single payout.
- **Three reward modes**: `LIST` runs every eligible reward, `WEIGHTED` picks one by weight, `RANDOM` picks one evenly.
- **Per-reward conditions**: a `condition:` expression per reward entry, so one voucher hands a better key to a higher-level player.
- **Auto-claim vouchers** that skip the item entirely and dispatch straight to the player.
- **A full item builder**: HEX colours, lore, glow, custom model data, armour trims, leather and potion tinting, and custom-texture player heads.
- **A paginated admin GUI** for browsing and handing out vouchers.

## Typical use

Crate keys, money notes, rank vouchers, seasonal reward drops. Anything you would otherwise hand out with a command, turned into an item a player can hold, trade and redeem when they want.

## Requirements

Java 21+, Paper 1.20.x or 1.21.x, **SnLib**, and a valid licence key. PlaceholderAPI is optional and only needed if you use `condition:`.

{% hint style="warning" %}
Upgrading from SnSimpleVouchers 1.x? Read [Migrating from 1.x](migration.md) first. v2.0.0 is a rebuild: the config and language layouts both change, `/voucher` now needs a permission, and SnLib is required. Your voucher files and every voucher item already in a player's inventory carry over untouched.
{% endhint %}
