# Permissions

Every administrative node defaults to `op`; `lootboxes.use` defaults to `true`.

| Permission | Default | Description |
|-----------|---------|-------------|
| `lootboxes.use` | `true` | Allows opening lootboxes with a valid key item. |
| `lootboxes.admin` | `op` | Full administrative access; a grant bundle of every node below. No command requires it on its own. |
| `lootboxes.admin.list` | `op` | Allows `/lootbox list`. |
| `lootboxes.admin.give` | `op` | Allows `/lootbox give`. |
| `lootboxes.admin.giveall` | `op` | Allows `/lootbox giveall`. |
| `lootboxes.admin.create` | `op` | Allows `/lootbox create`. |
| `lootboxes.admin.delete` | `op` | Allows `/lootbox delete`. |
| `lootboxes.admin.editor` | `op` | Allows `/lootbox editor`. |
| `snlootboxes.admin.reload` | `op` | Allows `/lootbox reload`. |
| `snlootboxes.admin.debug` | `op` | Allows `/lootbox debug`. |
| `snlootboxes.admin.update` | `op` | Receive update notifications of SnLootBoxes. |

The plugin's own action nodes use the `lootboxes.admin.*` prefix. The reload, debug and update nodes are injected by SnLib under the `snlootboxes.admin.*` prefix. `lootboxes.admin` grants all of them. Each action node works on its own: a helper holding only `lootboxes.admin.give` can run `/lootbox give`, and the generated `/lootbox` help lists only the subcommands the sender may run.
