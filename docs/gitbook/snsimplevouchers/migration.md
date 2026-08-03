# Migrating from 1.x

v2.0.0 is a full rebuild on SnLib. It is a MAJOR release: the config layout, the language layout and the new SnLib requirement all break compatibility.

**Back up `plugins/SnSimpleVouchers/` before you start.** No migrator ships.

## This is the last update that ever resets your config

v1.x carried a `config-version` key. Every time it was bumped - six times across the plugin's life - the affected file was renamed to `old-<file>.yml` and regenerated, destroying every prefix, translation, GUI slot and colour you had set. That was the design, and it was the single largest source of pain in this plugin's history.

v2.0.0 deletes that machinery. From now on new keys merge into your files while your values **and your comments** are preserved. This upgrade is a one-time break that ends a recurring one.

## What carries over untouched

{% hint style="success" %}
**Your vouchers are safe.** The `vouchers/` folder and its file schema are unchanged, and **every voucher item already sitting in a player's inventory or enderchest still works**. The tag that identifies a voucher item was deliberately kept identical.
{% endhint %}

Also unchanged: `basehead-<base64>` heads, `&#RRGGBB` colours, `trim:`, `color:`, decimal `amount`, `mode:`, per-reward `condition:`, the `{voucher} {category} {amount} {count} {player}` placeholders, and `/voucher give` / `/voucher open` as top-level commands with the order-independent `[amount] [-s]` grammar.

## Steps

1. Stop the server.
2. Copy `plugins/SnSimpleVouchers/` somewhere safe.
3. Install `SnLib.jar` in `plugins/`.
4. Replace the plugin jar.
5. Delete `config.yml` and `messages.yml` from `plugins/SnSimpleVouchers/`. Leave `vouchers/` alone.
6. Start the server. New `config.yml`, `lang/messages_en.yml` and `guis/*.yml` are generated.
7. Re-apply your settings and translations by hand, from the backup.
8. Grant `snsimplevouchers.admin` to your staff ranks.

## What changed

| Change | What you must do |
|---|---|
| **`depend: [SnLib]`** | Install `SnLib.jar`. Without it the plugin does not load at all |
| **`/voucher` now requires `snsimplevouchers.admin`** | Grant it to anyone who should reach the command, including its help |
| **`messages.yml` -> `lang/messages_en.yml`** | Re-apply translations. Key paths changed |
| **`config-version` retired** | Nothing, and never again |
| **`config.yml` regenerated** | Re-apply settings by hand |
| **GUI moves to `guis/*.yml`** | The `gui:` block leaves `config.yml`. Menu edits are redone in a richer schema: layout, regions, actions, pagination |
| **Aliases move to `config.yml`** | `command.aliases` is now authoritative over `plugin.yml` |
| **Help is now SnLib's** | Permission-filtered, paginated, rendered under the alias actually typed |
| **Tab completion is permission-filtered** | v1.4.3 showed `reload`/`open`/`give` to every player. It no longer does |
| **Permission nodes expand** | `snsimplevouchers.admin` still grants everything; `.give`, `.open`, `.reload`, `.debug`, `.update` are its children |
| **Update notices** | Grant `snsimplevouchers.admin.update` to staff who should see them |
| **Unknown-material fallback** | `PAPER` becomes `STONE`. Cosmetic, and only for a misconfigured voucher |

The shipped `claim.bulk-category` default was Spanish in an otherwise English file and is now English. If you were relying on the shipped text, re-set it.

## Behaviour changes that can affect a working config

These come out of the pre-release audit. Each one was silently wrong before.

{% hint style="warning" %}
**A quoted `enabled: "false"` now actually disables the voucher.** Quoted values were ignored, so the voucher stayed fully live and claimable. If you parked a voucher by quoting the value, it was never parked - and it is now. Check your files for quoted booleans before you update.
{% endhint %}

- The same fix applies to `auto-claim` and `glow`, where a quoted `"true"` was silently read as false. An `auto-claim: "true"` voucher was being handed out as a physical item instead of dispatching its rewards.
- **`item.material` values that cannot exist as an item are refused at load**, naming the voucher. `AIR`, `WATER` and similar previously passed every check and were handed out as nothing at all while the plugin reported success.
- **`snsimplevouchers.admin.give` now also gates the give button in `/voucher open`.** Previously the menu handed out vouchers with no permission test, so a rank with `.open` and `.give` negated could still mint every voucher on the server.
- **A `custom-model-data` written as a quoted string** used to build the item with model data `0`, which is a different item. It now warns and is ignored.
- **An `amount:` too large to write out is refused at load** and the voucher degrades to consuming one item per claim.

## After migrating

Test these before you consider it done:

1. **Right-click a voucher into the air**, not at a block. This is the path that broke repeatedly in 1.x.
2. Right-click one at a block.
3. Claim a stack of an `amount:` voucher and confirm it is a single command execution.
4. Open `/voucher open`, click a category, click a voucher.
5. Check the console for warnings naming any voucher that failed to load.
