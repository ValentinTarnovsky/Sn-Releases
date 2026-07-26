# Permissions

Every node defaults to `op`, except `snantixray.bypass`, which defaults to false and must be granted
explicitly.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snantixray.admin` | OP | Full administrative access. Gates the whole command tree |
| `snantixray.admin.reload` | OP | Allows `/antixray reload` |
| `snantixray.admin.debug` | OP | Allows `/antixray debug` |
| `snantixray.admin.update` | OP | Receive update notifications in chat |
| `snantixray.admin.bypass` | OP | Allows `/antixray bypass` |
| `snantixray.admin.alerts` | OP | Allows `/antixray alerts` |
| `snantixray.admin.check` | OP | Allows `/antixray check` |
| `snantixray.admin.reset` | OP | Allows `/antixray reset` |
| `snantixray.admin.stats` | OP | Allows `/antixray stats` |
| `snantixray.alerts` | OP | Eligible to receive detection alerts. Grants no command access |
| `snantixray.bypass` | false | See the true, unfiltered world. Grants no command access |

## Granting staff access

Granting `snantixray.admin` gives every `snantixray.admin.*` child plus `snantixray.alerts`. That is
the one node most servers need.

{% hint style="warning" %}
`snantixray.bypass` is deliberately **not** included in that bundle. Bundling it would make every
admin see through the plugin automatically, and your staff need to be able to see exactly what a
normal player sees. Grant it per person, and only while it is needed.
{% endhint %}

## The two passive nodes

`snantixray.alerts` and `snantixray.bypass` gate no command. They change what happens to you rather
than what you can run.

- **`snantixray.alerts`** decides whether you are eligible to receive detection alerts.
  `/antixray alerts` then toggles your personal preference on top of it. Both must be on before an
  alert reaches you, so the permission is the gate and the command is your mute switch.
- **`snantixray.bypass`** makes all three protection layers skip you. The persisted flag set by
  `/antixray bypass` does the same thing and is stored per player, so either one is enough.

{% hint style="info" %}
`/antixray check` reports the stored bypass flag, not the permission. A player who bypasses through
`snantixray.bypass` alone still reads as off there.
{% endhint %}
