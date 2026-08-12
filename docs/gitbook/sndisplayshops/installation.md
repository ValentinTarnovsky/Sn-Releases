# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper, both 1.20.x and 1.21.x |
| Required | `SnLib.jar` (SnLib 1.24.1 or newer) |
| Optional | PlaceholderAPI, DecentHolograms, EdTools, SuperiorSkyblock |
| Database | SQLite (default, no setup) or MySQL |
| License | Yes - SnDisplayShops is part of the licensed bundle |

The jar is compiled for Java 21. A 1.20.x server still running Java 17 refuses to load it, so move
the server to Java 21 first.

## Steps

1. Download `SnDisplayShops-<version>.jar` from the releases page (tags prefixed `sndisplayshops-`).
2. Put it in `plugins/`, together with `SnLib.jar`.
3. Start the server. SnDisplayShops writes its files, creates `plugins/.Sn-License/license.yml` and
   then disables itself, because that file still holds a placeholder.
4. Paste your bundle key on the first non-comment line of `plugins/.Sn-License/license.yml`.
5. Restart the server. SnDisplayShops validates the key at startup.
6. Give yourself a shop item with `/dshop give <player>` and place it.

{% hint style="info" %}
One key unlocks the whole bundle. `plugins/.Sn-License/license.yml` is shared by every Sn bundle
plugin on the server, so if you already run one of them the file exists and SnDisplayShops reads the
key that is already there. Nothing to paste twice.
{% endhint %}

{% hint style="info" %}
The key is checked once, at startup. The plugin refuses to enable without a valid one, so the server
log is the place to look if it is missing from `/plugins`.
{% endhint %}

{% hint style="warning" %}
Updating **SnLib** always needs a full restart, never a `/reload`.
{% endhint %}

## First boot

```
plugins/
  .Sn-License/
    license.yml          your bundle key - shared by every Sn bundle plugin on this server
  SnDisplayShops/
    config.yml           shop item, hologram, limits, currencies, database, integrations
    owner-menu.yml       the layout of the shop owner's management menu
    guis/
      buyer.yml          the layout of the menu everyone else sees
    lang/
      messages_en.yml    every line the plugin sends, plus the hologram text
    database.db          SQLite database, when database.type is sqlite
```

Nothing has to be created by hand. Every file above is written on first boot.

{% hint style="info" %}
`owner-menu.yml` sits at the root rather than under `guis/` on purpose. The owner menu accepts items
dragged into it, which a normal Sn menu cannot do, so it is not one of them.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PlaceholderAPI | No, but a command-backed currency needs it to read balances |
| DecentHolograms | No |
| EdTools | No |
| SuperiorSkyblock | No |

## Updating

Replace the jar and restart. New keys are merged into `config.yml`, `lang/messages_en.yml`,
`guis/buyer.yml` and `owner-menu.yml` on boot, and your values, comments and additions are
preserved. Example entries you deleted stay deleted.

{% hint style="info" %}
`currencies:` in `config.yml` is marked extensible: the entries there are your economy, not the
plugin's examples, so currencies you remove are never put back.
{% endhint %}
