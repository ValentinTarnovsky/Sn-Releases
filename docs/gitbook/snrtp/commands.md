# Commands

SnRTP registers one root command, `/rtp`, with the default alias `/wild`.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/rtp` | `snrtp.use` | Open the random-teleport menu. |
| `/rtp <world>` | `snrtp.use` | Teleport directly to a configured world. |
| `/rtp help` | `snrtp.use` | Show the command help. |
| `/rtp reload` | `snrtp.admin.reload` | Reload the configuration. |
| `/rtp debug` | `snrtp.admin.debug` | Toggle runtime debug output. |

{% hint style="info" %}
`/rtp <world>` completes every configured world key. A world defines its own permission in `config.yml` only when you set one.
{% endhint %}
