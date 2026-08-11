# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper 1.20.4 or newer, both 1.20.x and 1.21.x |
| Required | `SnLib.jar` (SnLib 1.24.1 or newer) |
| Optional | PlaceholderAPI |
| Database | SQLite (default, no setup) or MySQL |
| License | Yes, SnKits needs its own Sn license key |

The jar is compiled for Java 21. A 1.20.x server still running Java 17 refuses to load it, so move
the server to Java 21 first.

## Steps

1. Download `SnKits-<version>.jar` from the releases page (tags prefixed `snkits-`).
2. Put it in `plugins/`, together with `SnLib.jar`.
3. Start the server. SnKits writes its files, creates `plugins/SnKits/license.yml` and then
   disables itself, because that file still holds a placeholder.
4. Paste your license id into `license-id` in `plugins/SnKits/license.yml`.
5. Restart the server. SnKits validates the key at startup.
6. Edit `plugins/SnKits/config.yml` and the menus under `plugins/SnKits/guis/`, then run
   `/kit reload`.

{% hint style="info" %}
The key is checked once, at startup. SnKits refuses to enable without a valid one, so the server
log is the place to look if the plugin is missing from `/plugins`.
{% endhint %}

{% hint style="warning" %}
Updating **SnLib** always needs a full restart, never a `/reload`.
{% endhint %}

## First boot

```
plugins/
  SnKits/
    config.yml           menus, auto-claim, multi-claim, sounds, database, editor strings
    license.yml          your SnKits license id
    kits/
      default.yml        a working kit, ready to rename, edit or delete
    guis/                the layout of every menu, one file each
      kits-main.yml      the display menu a bare /kit opens
      kits-more.yml      a second page, linked both ways
      kit-preview.yml    the read-only preview of a kit's contents
      kit-edit.yml       the admin edit menu
      command-item.yml   the command item editor
    lang/
      messages_en.yml    every line the plugin sends to players and admins
    data.db              SQLite database, when database.type is sqlite
```

Nothing has to be created by hand. Every file above is written on first boot.

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PlaceholderAPI | No |

## Updating

Replace the jar and restart. New keys are merged into your `config.yml`, `lang/messages_en.yml`
and `guis/*.yml` on boot, and your values, comments and additions are preserved. Example entries
you deleted stay deleted.

{% hint style="info" %}
Kit files under `kits/` are data, not managed config. The plugin rewrites each one whole every
time you save a kit, so nothing is ever merged into them.
{% endhint %}
