# Permissions

Admin nodes default to op; the basic access node defaults to true.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snrtp.use` | true | Open the menu and run random teleports. |
| `snrtp.admin` | op | Full administrative access. |
| `snrtp.admin.reload` | op | Allows `/rtp reload`. |
| `snrtp.admin.debug` | op | Allows `/rtp debug`. |
| `snrtp.admin.update` | op | Receive update notifications. |

{% hint style="info" %}
Per-world gating is optional. Set `permission` under a world in `config.yml` to require a node of your choice, then grant that node in your permissions plugin.
{% endhint %}
