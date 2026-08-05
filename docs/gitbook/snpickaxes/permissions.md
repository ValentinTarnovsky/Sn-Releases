# Permissions

Every node defaults to OP. `snpickaxes.admin` is the parent, and its children list is exhaustive: a group granted the parent gets everything.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snpickaxes.admin` | OP | Full administrative access, and the gate of the whole command |
| `snpickaxes.admin.give` | OP | Allows `/snpickaxes give` |
| `snpickaxes.admin.reload` | OP | Allows `/snpickaxes reload` |
| `snpickaxes.admin.debug` | OP | Allows `/snpickaxes debug` |
| `snpickaxes.admin.update` | OP | Receives the update notice on join |

{% hint style="info" %}
`snpickaxes.admin` also gates the root command itself. A sender without it does not see `/snpickaxes` in tab completion, and is refused by the server before the plugin is reached.
{% endhint %}

The pickaxes need no permission at all. Anyone holding one can use it, so you control access by controlling who receives the item.
