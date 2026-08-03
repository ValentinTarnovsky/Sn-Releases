# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper 1.20.x or 1.21.x |
| Required | `SnLib.jar` (SnLib 1.20.0 or newer - API level 12) |
| Optional | PlaceholderAPI |
| License | Yes - SnJoinItem is part of the licensed bundle |

The jar is compiled for Java 21. A 1.20.x server still running Java 17 will refuse to load it, so
move the server to Java 21 first.

## Steps

1. Download `SnJoinItem-<version>.jar` from the releases page (tags prefixed `snjoinitem-`).
2. Put it in `plugins/`, together with `SnLib.jar`.
3. Start the server. SnJoinItem writes its config files, creates
   `plugins/.Sn-License/license.yml` and then disables itself, because that file still holds a
   placeholder.
4. Paste your bundle key on the first non-comment line of `plugins/.Sn-License/license.yml`.
5. Restart the server.
6. Edit `plugins/SnJoinItem/items.yml` (the item) and `plugins/SnJoinItem/config.yml` (when and
   where it is given), then run `/joinitem reload`.

{% hint style="info" %}
Updating **SnLib** always needs a full restart, never a `/reload`.
{% endhint %}

## First boot

```
plugins/
  .Sn-License/
    license.yml          your bundle key - shared by every Sn bundle plugin on this server
  SnJoinItem/
    config.yml           delivery (when the item is given) and placement (which slot)
    items.yml            the item itself: material, name, lore, the lock, the click actions
    lang/
      messages_en.yml    every line the plugin sends to players and admins
```

Those three files under `plugins/SnJoinItem/` are written even on a boot where the license check
fails, because SnLib mounts them before the gate runs. There is no `messages.yml`, and no
`config-version` key anywhere: SnLib merges new keys into your files on every boot and keeps your
values and comments.

`data.yml` is **not** created on first boot. It appears in `plugins/SnJoinItem/` the first time you
run `/joinitem disable <player>`, and holds the opt-out list by UUID. It is the one file SnJoinItem
never seeds, merges or regenerates, so an update can never hand the item back to players you had
opted out.

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

## Checking it worked

- The console shows no `Unknown dependency SnLib` and no `[Sn] License: FAIL`.
- With PlaceholderAPI installed you also get
  `[SnJoinItem] Registered PlaceholderAPI expansion 'snjoinitem'.`
- Join the server. About one second later (`delivery.give-delay-ticks: 20`) a compass named
  `Menu (click)` appears in slot `4`, the middle slot of the hotbar. The delay is deliberate: other
  plugins repopulate the inventory right after a join, and an item handed out instantly gets wiped.
- Try to drag, drop or shift-click it. It stays in its slot.
- `/joinitem give` hands the item to yourself immediately and ignores every filter, which makes it
  the quickest way to confirm the item itself is valid.

{% hint style="warning" %}
If your permissions plugin grants a `*` wildcard to admins, they hold `snjoinitem.bypass` and will
never receive the item on join. That is expected, not a broken install - `/joinitem give` still
works on them. Test with a normal account.
{% endhint %}

`/joinitem reload` re-reads `config.yml`, `items.yml`, `data.yml` and the language file, and rebuilds
the copy every online player is already holding. It delivers to nobody, so an empty slot right after
a reload is not a failure: the restore sweep or the next join fills it.

## Failure: SnLib is missing

`plugin.yml` declares `depend: [SnLib]`, so Paper rejects the jar before any of its code runs:

```
Could not load 'plugins/SnJoinItem-2.0.0.jar' in folder 'plugins'
org.bukkit.plugin.UnknownDependencyException: Unknown dependency SnLib
```

There is no `plugins/SnJoinItem/` folder, `/joinitem` answers `Unknown command`, and nothing else in
this page applies. Put `SnLib.jar` in `plugins/` and restart fully.

A second shape of the same problem is SnLib being present but too old. The plugin then loads, refuses
to start, and says exactly what it needs:

```
[SnJoinItem] Requires SnLib API level 12 (installed: 11). Update SnLib.jar (restart required):
https://github.com/ValentinTarnovsky/SnLib/releases
```

Update `SnLib.jar` and restart. Always ship the two jars together: the version check compares what
SnJoinItem was built against with what is installed, and a `/reload` cannot swap SnLib.

## Failure: no valid license key

The license is checked once, at startup. On any failure the console gets one line and no diagnostics
at all - that silence is intentional:

```
[SnJoinItem] [Sn] License: FAIL
```

The plugin then disables itself: no item, no commands, no listeners. The config files stay on disk
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
There are no runtime checks after startup: once SnJoinItem is up it stays up for the whole session,
and a network problem an hour later changes nothing. But any disable/enable cycle re-runs the gate,
so a `/reload confirm` or a PlugMan re-enable while the backend is unreachable will disable the
plugin. Restart the server instead.
{% endhint %}
