# Commands

SnRTP registers one root command, `/rtp`, with the default alias `/wild`.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/rtp` | `snrtp.use` | Open the random-teleport menu. |
| `/rtp reload` | `snrtp.admin.reload` | Reload the configuration. |
| `/rtp debug` | `snrtp.admin.debug` | Toggle runtime debug output. |

{% hint style="info" %}
The menu is the only way in. Since **1.2.0** there is no `/rtp <world>` shortcut and no `/rtp help`: players get exactly one command, `/rtp`, which opens the GUI, and the destination is picked from a menu button. A world still enforces its own `config.yml` permission when you set one.
{% endhint %}
