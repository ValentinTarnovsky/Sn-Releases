# Installation

1. Download the latest `sndrop-v*` release from the
   [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases/releases?q=sndrop-) page.
2. Put `SnDrop-<version>.jar` into your server's `plugins/` folder.
3. Make sure `SnLib.jar` is in `plugins/` as well. SnDrop hard-depends on it and will not load
   without it.
4. Start the server.

On first boot SnDrop generates:

```
plugins/SnDrop/
  config.yml
  lang/messages_en.yml
```

Both files are managed by SnLib: when a future version adds a key, it is merged into your file on
boot and your own values and comments are kept. There is no version key to bump and nothing is
ever regenerated behind your back.

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper 1.21 or newer |
| Dependency | `SnLib.jar` (`com.sn:snlib` 1.24.1+) |

SnDrop calls an inventory API whose type changed in 1.21, so 1.21 is a real floor rather than a
cautious one. It will not load on 1.20.

## Licensing

SnDrop is part of the Sn bundle. Paste your bundle key into:

```
plugins/.Sn-License/license.yml
```

That file is shared by every bundle plugin on the server, so one key covers all of them. The file
is created automatically on first boot with a placeholder line to replace.

The license is checked once, at startup. If it validates, the plugin runs normally for that
session; if it does not, the plugin disables itself and logs a single line. There are no runtime
checks and nothing degrades mid-session.
