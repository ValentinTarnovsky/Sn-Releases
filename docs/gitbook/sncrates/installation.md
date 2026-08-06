# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper 1.20.x or 1.21.x |
| Required | `SnLib.jar` (SnLib 1.24.0 or newer - API level 16) |
| Optional | PlaceholderAPI |
| Database | SQLite (default, no setup) or MySQL |
| License | Yes - SnCrates is part of the licensed bundle |

The jar is compiled for Java 21. A 1.20.x server still running Java 17 will refuse to load it, so
move the server to Java 21 first.

## Steps

1. Download `SnCrates-<version>.jar` from the releases page (tags prefixed `sncrates-`).
2. Put it in `plugins/`, together with `SnLib.jar`.
3. Start the server. SnCrates writes its config files, creates
   `plugins/.Sn-License/license.yml` and then disables itself, because that file still holds a
   placeholder.
4. Paste your bundle key on the first non-comment line of `plugins/.Sn-License/license.yml`.
5. Restart the server.
6. Edit `plugins/SnCrates/config.yml` and the crate files under `plugins/SnCrates/crates/`, then
   run `/crates reload`.

{% hint style="info" %}
Updating **SnLib** always needs a full restart, never a `/reload`.
{% endhint %}

## First boot

```
plugins/
  .Sn-License/
    license.yml          your bundle key - shared by every Sn bundle plugin on this server
  SnCrates/
    config.yml           defaults, keys, animations, effects, limits, database
    crates/
      example.yml        a working crate with two rewards - rename it, edit it or delete it
    guis/                the layout of every menu, one file each
      preview.yml
      preview-compact.yml
      key-balance.yml
      animation-csgo.yml
      animation-wheel.yml
      animation-reveal.yml
      animation-quick.yml
      editor-main.yml
      editor-crate.yml
      editor-rewards.yml
      editor-reward.yml
      reward-deposit.yml
    lang/
      messages_en.yml    every line the plugin sends to players and admins
    opening.log          appears after the first crate is opened
    data.db              SQLite database, when database.type is sqlite
```

Those files are written even on a boot where the license check fails, because SnLib mounts them
before the gate runs. There is no `messages.yml`, and no `config-version` key anywhere: SnLib merges
new keys into your files on every boot and keeps your values and comments.

### Crate files are seeded once and never merged

`crates/example.yml` is written on the first boot only. **Delete it and it does not come back.** No
crate file is ever merged, re-seeded or rewritten by an update, so an omitted key falls back to the
default documented beside it in the file and stays omitted.

That is deliberate: crate files are content, not configuration. `config.yml`, `guis/*.yml` and the
language file are the merged ones.

### The license file

The file is created with a placeholder:

```yaml
# Paste your Sn license key on the line below (replace the placeholder).
PASTE-YOUR-LICENSE-KEY-HERE
```

Replace that line with your key. The first line that is neither blank nor a comment is read as the
key; `license-id: YOUR-KEY` is also accepted, and surrounding quotes are stripped. One key unlocks
every bundle plugin on the server, so if you already run another Sn bundle plugin here, this step is
already done.

{% hint style="info" %}
`.Sn-License` starts with a dot, so some panel file managers hide it until you enable "show hidden
files".
{% endhint %}

## The database

`database.type: sqlite` is the default and needs nothing: a `data.db` file appears in the plugin
folder. Switch to MySQL by setting the type and filling in host, port, database, username and
password.

Seven tables are created, every one prefixed `sncrate_`. They hold virtual key balances, limit
counters, open statistics, the win history, per-player reward filters and the locations of bound
crate blocks. **Item data is never stored in the database** - rewards and key items live in the
crate YAML files, so a database wipe costs you balances and statistics, never your crates.

{% hint style="warning" %}
Do not point two different plugins at the same table names. `CREATE TABLE IF NOT EXISTS` never
compares columns, so a table another plugin already created under a name SnCrates expects would be
adopted as-is and every query against it would fail for the life of the install. The `sncrate_`
prefix exists for exactly that reason.
{% endhint %}

## Checking it worked

- The console shows no `Unknown dependency SnLib` and no `[Sn] License: FAIL`.
- With PlaceholderAPI installed you also get
  `[SnCrates] Registered PlaceholderAPI expansion 'sncrates'.`
- `/crates` as a player opens the key balance menu. With no keys it is empty, which is correct.
- `/crates givekey <you> example` hands you a key item named **Example Key**.
- `/crates preview example` shows the two shipped rewards with their chances.
- `/crates open example` spends the key and runs the CS:GO roulette.

`/crates reload` re-reads `config.yml`, the language file, every `guis/*.yml` and every crate file,
and re-sources the command aliases. It does not need a restart.

{% hint style="warning" %}
`command.user-open-aliases` is the one setting a reload cannot apply. Each entry registers its own
top-level command while the server starts, so adding or removing one needs a full restart.
{% endhint %}

## Failure: SnLib is missing

`plugin.yml` declares `depend: [SnLib]`, so Paper rejects the jar before any of its code runs:

```
Could not load 'plugins/SnCrates-2.0.0.jar' in folder 'plugins'
org.bukkit.plugin.UnknownDependencyException: Unknown dependency SnLib
```

There is no `plugins/SnCrates/` folder, `/crates` answers `Unknown command`, and nothing else in
this page applies. Put `SnLib.jar` in `plugins/` and restart fully.

A second shape of the same problem is SnLib being present but too old. The plugin then loads,
refuses to start, and says exactly what it needs:

```
[SnCrates] Requires SnLib API level 16 (installed: 13). Update SnLib.jar (restart required):
https://github.com/ValentinTarnovsky/SnLib/releases
```

Update `SnLib.jar` and restart. Always ship the two jars together: the version check compares what
SnCrates was built against with what is installed, and a `/reload` cannot swap SnLib.

## Failure: no valid license key

The license is checked once, at startup. On any failure the console gets one line and no diagnostics
at all - that silence is intentional:

```
[SnCrates] [Sn] License: FAIL
```

The plugin then disables itself: no commands, no listeners, no crates. The config files stay on disk
untouched, so nothing is lost while you sort the key out.

Work through the causes in this order:

| Cause | What to do |
|---|---|
| The placeholder is still in `license.yml` | Paste your key. While the line contains `PASTE-YOUR-LICENSE`, no request is even sent. |
| The key sits below another content line, or on a commented line | Only the first non-blank, non-comment line is read. Put the key there. |
| The server cannot reach the license backend while booting | Allow outbound HTTPS. The check is 3 attempts (immediately, then after 1s and 3s), each timing out after 10 seconds. |
| The jar was repackaged, re-zipped or edited | Use the jar exactly as downloaded. The license is bound to the released build's SHA-256. |
| The key is wrong or the subscription lapsed | Nothing on the server side will fix this - check the key you were issued. |

{% hint style="warning" %}
There are no runtime checks after startup: once SnCrates is up it stays up for the whole session,
and a network problem an hour later changes nothing. But any disable/enable cycle re-runs the gate,
so a `/reload confirm` or a PlugMan re-enable while the backend is unreachable will disable the
plugin. Restart the server instead.
{% endhint %}

## Failure: the database is unreachable

With `database.type: mysql` pointed at a host that does not answer, the connection attempt fails
after `database.connect-timeout-seconds` (10 by default) and the plugin logs it. Key balances,
limits and block bindings are unavailable, so openings that depend on them are refused rather than
run uncounted.

Raise `connect-timeout-seconds` only if your database is genuinely slow to accept connections. It is
also how long a server **stop** can wait on an unreachable host while the plugin flushes.
