# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper 1.20.x or 1.21.x (Display entities need 1.19.4 or newer) |
| Required | `SnLib.jar` (SnLib 1.28.0 or newer - API level 19) |
| Optional | DecentHolograms (labels), PlaceholderAPI (placeholders) |
| License | Yes - SnRotatinHeads is part of the licensed bundle |

## Steps

1. Download `SnRotatinHeads-<version>.jar` from the releases page (tags prefixed `snrotatinheads-`).
2. Put it in `plugins/`, together with `SnLib.jar`.
3. Start the server. SnRotatinHeads writes its config files, creates
   `plugins/.Sn-License/license.yml` and then disables itself, because that file still holds a
   placeholder.
4. Paste your bundle key on the first non-comment line of `plugins/.Sn-License/license.yml`.
5. Restart the server.
6. Stand where you want the first head and run `/rh create <id>`.

{% hint style="info" %}
Updating **SnLib** always needs a full restart, never a `/reload`.
{% endhint %}

## First boot

```
plugins/
  .Sn-License/
    license.yml          your bundle key - shared by every Sn bundle plugin on this server
  SnRotatinHeads/
    config.yml           animation interval, defaults for new heads, label settings
    lang/
      messages_en.yml    every line the plugin sends to admins
    heads.yml            appears after your first /rh create - the head data
```

There is no `config-version` key anywhere: SnLib merges new keys into `config.yml` and the language
file on every boot and keeps your values and comments. `heads.yml` is data: the plugin writes it
after every change and never seeds, merges or regenerates it. You can edit it by hand and run
`/rh reload`.

### The license file

The file is created with a placeholder:

```yaml
# Paste your Sn license key on the line below (replace the placeholder).
PASTE-YOUR-LICENSE-KEY-HERE
```

Replace that line with your key. One key unlocks every bundle plugin on the server, so if you
already run another Sn bundle plugin here, this step is already done.

{% hint style="info" %}
`.Sn-License` starts with a dot, so some panel file managers hide it until you enable "show hidden
files".
{% endhint %}

## Checking it worked

- The console shows no `Unknown dependency SnLib` and no `[Sn] License: FAIL`, and prints
  `Loaded 0 rotating head(s).` on the first tick.
- `/rh` prints the generated help.
- `/rh create test` places a plain spinning head at your position. `/rh texture test <base64>` skins
  it, `/rh hologram add test &aHello` labels it (with DecentHolograms installed).

## Failure: SnLib is missing

`plugin.yml` declares `depend: [SnLib]`, so Paper rejects the jar before any of its code runs:

```
Could not load 'plugins/SnRotatinHeads-<version>.jar' in folder 'plugins'
org.bukkit.plugin.UnknownDependencyException: Unknown dependency SnLib
```

Put `SnLib.jar` in `plugins/` and restart fully. If SnLib is present but too old, the plugin loads,
refuses to start and names the API level it needs; update `SnLib.jar` and restart.

## Failure: no valid license key

The license is checked once, at startup. On any failure the console gets one line and no diagnostics:

```
[SnRotatinHeads] [Sn] License: FAIL
```

The plugin then disables itself. Check, in order: the placeholder is still in `license.yml`; the key
is not on the first non-comment line; the server cannot reach the license backend during boot (allow
outbound HTTPS); the jar was repackaged or edited; the key is wrong or lapsed.

{% hint style="warning" %}
There are no runtime checks after startup, but any disable/enable cycle re-runs the gate, so a
`/reload confirm` while the backend is unreachable disables the plugin. Restart the server instead.
{% endhint %}
