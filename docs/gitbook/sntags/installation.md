# Installation

1. Download the latest `sntags-v*` release from [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sntags-).
2. Place the `.jar` file into your server's `plugins/` folder.
3. Start the server once. SnTags creates a shared license file at `plugins/.Sn-License/license.yml`.
4. Paste your Sn license key into that file, replacing the placeholder line.
5. Restart the server. SnTags validates the key at startup and then enables.

{% hint style="warning" %}
Requires **SnLib** installed (`depend: [SnLib]`). The server refuses to load SnTags without `SnLib.jar` present in `plugins/`.
{% endhint %}

{% hint style="info" %}
The key in `plugins/.Sn-License/license.yml` is shared by every bundled Sn plugin on the server, so you paste it once. Note the leading dot in the folder name: it is hidden by default on Linux and in most SFTP clients.
{% endhint %}

## Dependencies

| Plugin | Required |
|--------|----------|
| SnLib | Yes |
| PlaceholderAPI | No (optional, enables placeholders) |

**Requirements**: Java 21+, Paper 1.20.x or 1.21.x.

## Upgrading from 1.x

{% hint style="danger" %}
Back up `plugins/SnTags/` **and your database** first. Two of the changes below cannot be undone.
{% endhint %}

2.0.0 is a rewrite, not a drop-in update. Before starting the server, delete `config.yml`, `messages.yml` and `guis.yml` from `plugins/SnTags/`.

This is not optional. SnTags 2.0.0 does not read the 1.x config layout, and leaving the old `config.yml` in place produces a file the server cannot parse. When that happens the plugin starts on built-in defaults, which means **a MySQL server silently falls back to a new, empty SQLite file and serves every player no tags**, with nothing in the log that names the cause.

Keep `tags.yml` - it is read as-is.

### What carries over

| | |
|---|---|
| Tag ownership (who owns which tag) | **Kept.** Same table, same schema. |
| Personal tags from `/tagadmin custom` | **Kept**, including their numeric ids. |
| `tags.yml` | **Kept** and read as-is. |
| Whichever tag each player had **equipped** | **Reset.** 1.x stored this in a table 2.0.0 does not read. Everyone keeps their tags and re-picks one from `/tags`. |
| Mixed-case tag identifiers | **Merged, irreversibly.** 1.x stored `VIP` and `vip` as two tags; on the first start they become one and the duplicate ownership row is deleted. Players keep the tag through the surviving row. |
| `config.yml`, `messages.yml`, `guis.yml` | **Not read, nothing converted.** Re-enter your database credentials, your `table-prefix` and any restyling by hand. |

### Behaviour changes to check

- **Placeholders now need a viewer.** See [Placeholders](placeholders.md) - this one silently changes the result of any condition you wrote in a hologram.
- **Display text is validated on input.** `/tagadmin create` and `/tagadmin custom` refuse interactive MiniMessage. See [Commands](commands.md).
- **MySQL pool tuning is one knob**, `database.pool-size`, replacing the four `pool.*` keys of 1.x.
