# Migrating from 1.x

SnKills 2.0.0 is a rebuild on SnLib. Everything the plugin did, it still does - but the config,
the language file and the permission nodes are all new. **This is not a drop-in update.**

{% hint style="warning" %}
Back up `plugins/SnKills/` before you start.
{% endhint %}

## Do this first: permissions

This is the change that breaks silently. A staff rank granted the old nodes keeps working
right up until someone runs `/snkills reload` and gets a no-permission line.

| 1.x | 2.0.0 |
|---|---|
| `snkills.use` | gone - there are no player-facing commands any more |
| `snkills.reload` | `snkills.admin.reload` |
| `snkills.*` | gone - grant `snkills.admin` |
| - | `snkills.admin.debug` (new) |
| - | `snkills.admin.update` (new) |

Update your LuckPerms groups **before** you restart.

## What happens to your config on first boot

Your `config.yml` is renamed to `old-config.yml` and a fresh one is generated. An existing
backup is never overwritten - a second migration writes `old-config-1.yml` and so on. The
console names every key that moved.

The rename is necessary, not cosmetic: 1.x shipped `debug` as a plain value and 2.0.0 ships it
as a section, and merging the new schema into the old layout produces a file that is not valid
YAML. If that happened, the plugin would come up with an empty config and - because the vanilla
death message is suppressed regardless - every death on your server would go silent with
nothing to explain it. Moving the file aside is what prevents that.

## Key-by-key

| 1.x | 2.0.0 |
|---|---|
| `death-messages.KILLED` | `death-messages.PLAYER_KILL` |
| `item-format` | `weapon-display.format` |
| `empty-hand-text` | `weapon-display.empty-hand` |
| `prefix` | `prefix` in `lang/messages_en.yml` |
| `messages.reload` | `snlib.reload-done` in `lang/messages_en.yml` |
| `messages.no-permission` | `snlib.no-permission` in `lang/messages_en.yml` |
| `messages.help-header` / `-reload` / `-help` | `snlib.help.header` / `.entry` / `.footer` |
| `config-version` | retired - new keys merge in automatically now |
| `debug: false` | `debug.enabled: false` (plus `level` and `categories`) |
| - | `weapon-display.amount-format` (was hardcoded) |

## About `KILLED`

If you edited `death-messages.KILLED` in 1.x and wondered why it never fired: `KILLED` is not
a Bukkit damage cause. It was in the shipped default config, the loader dropped it silently,
and a clean install printed no death message at all while still suppressing the vanilla one.

`PLAYER_KILL` is the key that does what `KILLED` looked like it did: it fires whenever the
victim has a player killer, whatever the damage cause was. Move your PvP text there.

## Other visible differences

- `/snkills help` is SnLib's paginated, permission-filtered help. Different layout, same job.
- Update notices go to `snkills.admin.update` holders only.
- `command.aliases` in `config.yml` adds aliases live on reload. `skills` also ships in
  `plugin.yml`, so removing it from the config does not unregister it.
- MiniMessage colour tags now render. Interactive tags (`<click>`, `<hover>`, `<font>`,
  `<newline>`) render as literal text on purpose - a placeholder can resolve to text a player
  chose, and this message goes to everyone.
- A fresh install now prints a fallback line for every death instead of nothing.

## Requirements that are new

- `SnLib.jar` in `plugins/`. The plugin will not load without it, and updating SnLib always
  needs a restart, never a reload.
- A bundle licence key in `plugins/.Sn-License/license.yml`. See
  [Installation](installation.md).
