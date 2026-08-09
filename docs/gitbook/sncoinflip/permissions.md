# Permissions

| Permission | Default | Description |
|------------|---------|-------------|
| `sncoinflip.use` | true | Basic access to the coinflip commands and menus. |
| `sncoinflip.admin` | OP | Full administrative access. Implies the three nodes below. |
| `sncoinflip.admin.reload` | OP | Allows `/coinflip reload`. |
| `sncoinflip.admin.debug` | OP | Allows `/coinflip debug`. |
| `sncoinflip.admin.update` | OP | Receives update notifications on join. |

{% hint style="warning" %}
`sncoinflip.use` is a single gate over creating, joining, cancelling and viewing stats. There is no separate node for creating a coinflip, so revoking `sncoinflip.use` from a player who already has one listed also takes away their ability to cancel it. Cancel it for them, or let it resolve, before revoking.
{% endhint %}

{% hint style="info" %}
Gambling is often something a server wants to disable for a specific group without touching anything else. Today that means revoking `sncoinflip.use`, which removes the whole plugin for that group.
{% endhint %}
