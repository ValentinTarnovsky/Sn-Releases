# Installation

SnGens is a licensed Sn plugin. Install it, boot once to generate the license file, paste your
key, then restart.

1. Download the latest `sngens-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sngens-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Install the three required plugins listed below, if you do not run them already.
4. Start the server once. The plugin creates `plugins/SnGens/license.yml`.
5. Open that file and replace the placeholder with your Sn license key:

```yaml
# Sn License - paste the license_id you received from snopeyy here.
license-id: "PASTE-YOUR-LICENSE-ID-HERE"
```

6. Restart the server. The plugin validates the key at startup.

{% hint style="danger" %}
Without a valid key the plugin refuses to enable and disables itself during startup. The check
runs once, at boot: there are no periodic re-checks and no degraded mode while running.
{% endhint %}

## Dependencies

| Plugin | Required | What it does |
|--------|----------|--------------|
| Vault | Yes | Every payment and payout. Buying, upgrading, repairing and selling all go through your Vault economy plugin. |
| PlaceholderAPI | Yes | Registers the `%sngens_...%` expansion for scoreboards, tab, holograms and menus. |
| DecentHolograms | Yes | Draws the floating text over broken generators, collectors and hoppers. |
| SuperiorSkyblock2 | No | Island mode: island leaderboards, island only placement, shared upgrades and vault returns on kick or disband. |
| SnDisplayShops | No | Lets a sellwand sell the generator drops stored inside a display shop. |

{% hint style="warning" %}
The three required plugins are declared as hard dependencies. If any of them is missing the
server will not load SnGens at all.
{% endhint %}

## Server requirements

| Requirement | Value |
|-------------|-------|
| Server software | Paper 1.21.x or a fork of it |
| Folia | Supported |
| Java | 21 or newer |
| Database | SQLite out of the box, MySQL or MariaDB optional |

SnGens is built against the Paper API and schedules its per region work through a Folia aware
scheduler, so the same jar runs on both Paper and Folia. Nothing needs to change in the config
when you move between them.

## What the first boot creates

On a clean install the plugin writes these files into `plugins/SnGens/`:

| File | Purpose |
|------|---------|
| `config.yml` | Every global tunable: language, database, tick rate, limits, corruption, leaderboard, upgrade menu |
| `generators.yml` | Every generator: item, drops, sell values, upgrade chain, corruption cost |
| `wands.yml` | The sellwand, build wand, upgrade wands and the admin region wand |
| `events.yml` | The timed server events and their rotation |
| `storages.yml` | The collector and the infinite hopper |
| `armors.yml` | Four piece armor sets and their full set bonuses |
| `offhands.yml` | Off-hand items and their bonuses |
| `lang/messages_en.yml` | Every player facing message |
| `gui/*.yml` | Eight menu layouts: shop, buy amount, upgrade, top, three collector menus, hopper |
| `license.yml` | Your license key |
| `SnGens.db` | The SQLite database, unless you switch to MySQL |

The plugin also creates `debug-dumps/` the first time you run `/gens debugspawn`.

## Which files are merged on update, and which are yours

This matters when you update the jar, so it is worth reading once.

**Merged files.** `config.yml`, `wands.yml`, `storages.yml`, `gui/*.yml` and the language files
are managed. On every boot and on every `/gens reload` the plugin inserts keys that the new
version added and that your file does not have yet. Your values, your lists and your comments
are never touched.

**Seeded files.** `generators.yml`, `events.yml`, `armors.yml` and `offhands.yml` are written
once, on a fresh install, and never written to again. They are yours. A generator you delete
stays deleted, and a generator you add is picked up on the next reload. New example content
that a later version ships does not appear in these four files: that is deliberate, since
re-inserting an example generator into a live economy would be worse than missing it.

{% hint style="info" %}
Set `update-configs: false` at the top of `config.yml` to freeze the merged files as well. The
plugin then only logs a warning listing how many sections are missing, and keeps running on
safe defaults for them.
{% endhint %}

## Switching to MySQL

SQLite needs no setup and is the default. For a MySQL or MariaDB server, edit `config.yml`:

```yaml
mysql:
  enabled: true
  host: 127.0.0.1
  port: 3306
  database: db_sngens
  user: sngens
  password: 'your-password'
  useSSL: false
```

Restart the server after changing these values, since the connection pool is built at startup.
The eight `sngens_*` tables and their indexes are created automatically on first connect, on
both database types.

{% hint style="warning" %}
One server per database. SnGens caches generators and players in memory and writes them back
periodically, so two servers pointing at the same database would overwrite each other.
{% endhint %}

## Changing the language

`lang: en` in `config.yml` selects `lang/messages_en.yml`. To ship your own language, copy the
English file to `lang/messages_es.yml`, translate the values, and set `lang: es`.

Only the English file ships inside the jar, so your translated file cannot be updated from the
jar directly. Instead the plugin merges any missing key into it from the English file, and logs
how many were added, so a translation never ends up with a hole in it after an update.

## First steps after install

{% hint style="success" %}
Run `/gens give <player> wheat_generator 3` to hand out generators, then `/gens shop` to see
the shop the plugin built from `generators.yml`. Both work out of the box with the example
configuration.
{% endhint %}
