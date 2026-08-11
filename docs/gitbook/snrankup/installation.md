# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper, both 1.20.x and 1.21.x |
| Required | `SnLib.jar` (SnLib 1.25.0 or newer) |
| Optional | Vault plus an economy plugin, PlaceholderAPI |
| Database | SQLite (default, no setup) or MySQL |
| License | Yes - SnRankUp is part of the licensed bundle |

The jar is compiled for Java 21. A 1.20.x server still running Java 17 refuses to load it, so move
the server to Java 21 first.

## Steps

1. Download `SnRankUp-<version>.jar` from the releases page (tags prefixed `snrankup-`).
2. Put it in `plugins/`, together with `SnLib.jar`.
3. Start the server. SnRankUp creates `plugins/.Sn-License/license.yml` and then disables itself,
   because that file still holds a placeholder.
4. Paste your bundle key on the first non-comment line of `plugins/.Sn-License/license.yml`.
5. Restart the server. SnRankUp validates the key at startup.
6. Edit `plugins/SnRankUp/rankup.yml` to build your ladder, then run `/rankup reload`.

{% hint style="info" %}
One key unlocks the whole bundle. `plugins/.Sn-License/license.yml` is shared by every Sn bundle
plugin on the server, so if you already run one of them the file exists and SnRankUp reads the key
that is already there. Nothing to paste twice.
{% endhint %}

{% hint style="info" %}
The key is checked once, at startup. SnRankUp refuses to enable without a valid one, so the server
log is the place to look if the plugin is missing from `/plugins`.
{% endhint %}

{% hint style="warning" %}
Updating **SnLib** always needs a full restart, never a `/reload`.
{% endhint %}

## First boot

```
plugins/
  .Sn-License/
    license.yml          your bundle key - shared by every Sn bundle plugin on this server
  SnRankUp/
    config.yml           language, command aliases, database, menu mode, leaderboard, currencies
    rankup.yml           the ladder: one entry per rank, with its price, rewards and menu item
    guis/                the layout of both menus, one file each
      rankup-menu.yml    the single-button menu, used when menu.mode is single
      rankup-list.yml    the paginated ladder, used when menu.mode is paginated
    lang/
      messages_en.yml    every line the plugin sends to players and admins
    data.db              SQLite database, when database.type is sqlite
```

Nothing has to be created by hand. Every file above is written on the first boot that passes the
licence check, and the shipped `rankup.yml` is a working four rank ladder you can rename, extend or
replace.

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| Vault | No |
| PlaceholderAPI | No |

Vault and PlaceholderAPI are only needed by the currency types that name them. A ladder priced in
`hours` alone needs neither.

## Updating

Replace the jar and restart. New keys are merged into your `config.yml`, `rankup.yml`,
`lang/messages_en.yml` and `guis/*.yml` on boot, and your values, comments and additions are
preserved. Example entries you deleted stay deleted.

{% hint style="info" %}
There is no `config-version` key and nothing to migrate by hand. Set `update-configs: false` to
freeze every managed file, and SnLib then only warns about missing keys.
{% endhint %}
