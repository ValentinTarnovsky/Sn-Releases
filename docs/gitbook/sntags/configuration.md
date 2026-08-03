# Configuration

SnTags ships four files in `plugins/SnTags/`.

| File | What it holds |
|------|---------------|
| `config.yml` | Database, table prefix, menu cooldown, the no-tag placeholder |
| `lang/messages_en.yml` | Every message the plugin sends |
| `tags.yml` | The global tag catalogue |
| `guis/tag-selector.yml` | The selector menu: layout, items, pagination, sounds |

New keys are merged in on update and your values are kept, so you never re-apply your settings after upgrading.

## config.yml

```yaml
lang: en
update-configs: true

debug:
  enabled: false
  level: DEBUG
  categories: []

command:
  aliases: [tag]

database:
  type: sqlite
  file: data.db
  host: localhost
  port: 3306
  database: sntags
  username: root
  password: ""
  pool-size: 10
  ssl: false

table-prefix: "sntags_"

menu:
  click-cooldown-ms: 1500

placeholders:
  no-tag: ""
```

| Key | Meaning |
|-----|---------|
| `command.aliases` | Aliases of `/tags`. Adding one works; note that `tag` is also declared in the plugin's own descriptor, so it cannot be removed from here. |
| `database.type` | `sqlite` or `mysql`. |
| `database.file` | SQLite filename, ignored on MySQL. Keep it as `data.db` unless you are starting from nothing - that is where an existing SnTags database lives. |
| `database.pool-size` | MySQL connection pool size. |
| `table-prefix` | Prefix of the plugin's tables. |
| `menu.click-cooldown-ms` | Throttle between selector clicks. `0` disables it. |
| `placeholders.no-tag` | What `%sntags_tag%` returns for a player with no tag. Empty by default. |

{% hint style="warning" %}
Change `table-prefix` **before the first start only**. Changing it later points the plugin at a different set of tables, which reads as every player having lost every tag. It is captured at startup, so editing it and reloading appears to do nothing until the next restart.
{% endhint %}

## tags.yml

The global catalogue, and the only source of truth for global tags.

```yaml
tags:
  vip:
    display: "&#55efc4[VIP]"
  mvp:
    display: "&#ffeaa7[MVP]"
```

**The order of the keys is the order tags appear in the menu.** Move a block up or down to reorder the selector; it is never sorted alphabetically.

Identifiers are lowercased on read, so `VIP` and `vip` are the same tag. Entries you add, edit or delete are yours and stay as you leave them - the file is seeded once and never merged again.

{% hint style="warning" %}
If you edit `tags.yml` by hand **while the server is running**, run `/tags reload` before using `/tagadmin create` or `/tagadmin delete`. Those commands rewrite the file from the copy the plugin loaded at startup, so unreloaded hand edits are lost. Editing the file while the server is stopped needs no reload, and `/tags reload` on its own is always safe.

This is a known issue and is planned for 2.0.1.
{% endhint %}

## guis/tag-selector.yml

The whole selector menu. Slots come from a `layout:` mask rather than numbers, so you draw the menu instead of computing indices:

```yaml
layout:
  - "fffffffff"
  - "fttttttt f"
```

Each letter maps to an item defined further down, and the region letter marks the cells the tags page through. Materials, display names, lore, glow, sounds and the page arrows are all yours to edit.

Two constraints worth knowing:

- Every row of `layout:` must be exactly 9 characters.
- On the page arrows, keep `[previous-page]` / `[next-page]` **before** `[custom] tags-rebind` in the action list. The actions run in order, and the rebind has to slice the entries for the page you just moved to.

To remove a button, delete its letter from `layout:`. Deleting the item block instead is undone on the next start, because the file is merged on update.
