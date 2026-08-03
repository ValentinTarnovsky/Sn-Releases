# FAQ

### The plugin will not load at all

SnTags requires **SnLib**. Without `SnLib.jar` in `plugins/`, the server refuses to load it. Check the startup log for a missing-dependency line.

If it loads but disables itself, check the license: the key goes in `plugins/.Sn-License/license.yml` (note the leading dot - the folder is hidden by default on Linux and in most SFTP clients), and the plugin refuses to enable without a valid one.

### I upgraded from 1.x and everyone lost their tags

You almost certainly kept the old `config.yml`. See [Installation](installation.md#upgrading-from-1-x): 2.0.0 does not read the 1.x layout, and leaving the old file in place produces a config the server cannot parse, after which the plugin runs on defaults - which means SQLite on a fresh file, even if you run MySQL.

Stop the server, restore your backup of `plugins/SnTags/`, delete `config.yml`, `messages.yml` and `guis.yml`, re-enter your database settings and your `table-prefix`, and start again. The ownership rows were never touched.

### Everyone is wearing nothing after the upgrade

Expected. 1.x stored the equipped tag in a table 2.0.0 does not read, so every player starts with none equipped and re-picks from `/tags`. The tags they own are intact.

### I edited tags.yml and my changes disappeared

If you edited it while the server was running and then ran `/tagadmin create` or `/tagadmin delete`, those commands rewrote the file from the copy loaded at startup.

Run `/tags reload` after editing by hand and before using those commands. Editing while the server is stopped is always safe. This is a known issue, planned for 2.0.1.

### A tag renders in the menu but the placeholder is empty

The two disagree when the stored selection no longer resolves - the tag was deleted from `tags.yml`, or the player's ownership was revoked. The selector filters those out and the placeholder returns the no-tag value. Have the player re-pick from `/tags`, or re-grant the tag.

If it happens in a hologram or a console-context parse, see [Placeholders](placeholders.md#both-placeholders-need-a-viewer) - the placeholder is not resolving at all there.

### My tag shows `<VIP>` as literal text, or the command rejected it

Angle brackets are MiniMessage syntax. Any `<name>` that is not a colour or a decoration is rejected on input, including a plain label like `<VIP>`. Write `[VIP]` instead.

This is deliberate: tag text is shown to other players, and an interactive element like `<click:run_command>` inside a tag would run as whoever clicks it.

### /tagadmin removecustom does not suggest the id I just created

Tab completion is capped at 100 suggestions and the index is ordered oldest-first, so past 100 live personal tags the newest ids stop being offered. The command still works when you type the id in full.

`/tagadmin custom` prints the id when it creates the tag - keep it.

### Can I reorder the tags in the menu?

Yes. The order of the keys in `tags.yml` is the order they appear. Move a block up or down; the list is never sorted alphabetically.

### Can I remove a button from the selector?

Delete its letter from the `layout:` mask in `guis/tag-selector.yml`. Deleting the item block instead is undone on the next start, because that file is merged on update.

### Changing table-prefix did nothing

It is read once at startup, so a reload will not pick it up. It is also only safe to set **before the first start** - changing it on a live server points the plugin at a different set of tables, which looks exactly like every player losing every tag.

### Does it work on 1.20 and 1.21?

Yes, both, on Paper. Java 21 or newer.
