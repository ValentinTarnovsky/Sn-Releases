# Migrating from 1.x

SnReclaim 2.0.0 was rebuilt from scratch on SnLib. Everything the plugin did, it still
does - but the plumbing under it was replaced, and four things change on your server.

**Back up `plugins/SnReclaim/` before updating.**

## 1. SnLib is required

The plugin will not load without `SnLib.jar` in `plugins/`. Updating SnLib itself always
needs a full restart, never a `/reload`.

## 2. A license key is required

SnReclaim joined the licensed bundle. The key goes in the **shared**
`plugins/.Sn-License/license.yml`, not in the plugin's own folder - so a server already
running another bundle plugin needs nothing new. A server running only SnReclaim will not
start without one.

## 3. Permission nodes were renamed

This one breaks **silently**: LuckPerms keeps happily granting a node nothing checks any
more, so staff simply lose access with no error anywhere.

| 1.x | 2.0.0 |
|---|---|
| `reclaim.use` | `snreclaim.use` |
| `reclaim.admin` | `snreclaim.admin` |
| `reclaim.admin.reload` | `snreclaim.admin.reload` |
| `reclaim.admin.manage` | `snreclaim.admin.manage` - now only `enable`/`disable` |
| `reclaim.admin.reset` | `snreclaim.admin.reset` |
| - | `snreclaim.admin.status` **(new)** - `/reclaim status`, grantable on its own |
| - | `snreclaim.admin.debug` **(new)** |
| - | `snreclaim.admin.update` **(new)** - who sees update notices |

Granting `snreclaim.admin` covers all of them.

## 4. config.yml changed shape

{% hint style="danger" %}
**If you had reclaims globally disabled, they come back ENABLED on the first 2.0.0 boot.**
{% endhint %}

SnLib **merges** managed files rather than replacing them, so your 1.x `config.yml` survives
intact on disk while the 2.0.0 keys are added beside it with their shipped defaults. Nothing
errors and nothing looks wrong - your old values are simply no longer read.

| 1.x key | 2.0.0 key |
|---|---|
| `settings.reclaims-enabled` | `redeem.enabled` |
| `settings.default-run-as` | `redeem.run-as` |
| `settings.notify-on-join` | `join-notify.enabled` |
| `settings.join-notify-delay-ticks` | `join-notify.delay-ticks` |
| `messages.*` | moved to `lang/messages_en.yml` |
| `prefix` | the `prefix` line of `lang/messages_en.yml` |
| `config-version` | retired - SnLib merges new keys automatically |

The plugin prints a warning block at boot naming every stale key it still finds, and says
outright when the season lock has been lifted. **Re-apply your values under the new keys and
delete the old block.**

Your `ranks:` entries also need re-authoring in the new file. Keep the **same rank keys** -
see the warning below.

## 5. Messages are English now

The Spanish strings 1.x shipped are not carried over. Re-apply your wording in
`lang/messages_en.yml`, or create `lang/messages_es.yml` and set `lang: es` in `config.yml`.

Your hand-rolled help lines are gone too: `/reclaim help` is now generated, paginated and
filtered by permission.

## What does NOT change

- **Every command still works.** `/reclaim`, `/rc`, `reload`, `reset <player>`, `enable`,
  `disable`, `status`. The admin subcommands were deliberately left where they were so your
  macros, command blocks, signs and staff documentation keep working.
- **Your claims carry over.** Same file (`data.db`), same table, same columns. Nobody gets
  their reclaims back by accident.

{% hint style="danger" %}
When you re-author `ranks:`, keep the **exact same keys** you used in 1.x. The key is the
claim's identity in the database - a rank renamed from `vip` to `VIP` reads as a rank nobody
has ever claimed, and every player who already took it gets the reward again.
{% endhint %}

## Checklist

1. Back up `plugins/SnReclaim/`.
2. Install `SnLib.jar`; put the licence key in `plugins/.Sn-License/license.yml`.
3. Drop in the new jar and start the server.
4. Read the warning block in the console - it names every stale key.
5. Re-apply `settings.*` under `redeem:` / `join-notify:`, keeping the same values.
6. Re-author `ranks:` with the **same keys**.
7. Re-apply your message wording in `lang/messages_en.yml`.
8. Update your permission groups to the `snreclaim.*` nodes.
9. Delete the leftover 1.x block from `config.yml`, then `/reclaim reload`.
10. Confirm with `/reclaim status` that the lock is where you want it.
