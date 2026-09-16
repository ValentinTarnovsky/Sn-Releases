# FAQ

### How do I update SnLootBoxes?

Download the newer `snlootboxes-v*` release and replace the jar. Configs auto-merge on restart.

### Does it support Folia?

No, SnLootBoxes is not Folia-compatible.

### Why do my old keys not stack with new ones?

They do, once the plugin has seen them. Every key a player holds is re-rendered from the current lootbox definition when the player joins and right before any `/lootbox give`, `/lootbox giveall` or editor key grant. Keys from before a reward edit, or from a time when `key-items.stackable` was `false`, then merge with new keys. Set `key-items.stackable: false` to keep every grant on its own stack instead.

### Why does the item a lootbox gives not stack with the real one?

Because the reward is a snapshot. When you add an item reward, the plugin stores the exact bytes of the item you clicked, which is the only way a lootbox can hand out an item that belongs to another plugin. That snapshot never changes afterwards, so if the plugin that mints the item later changes how it renders the name or the lore, your stored copy keeps looking the same while its underlying components no longer match a freshly obtained one, and the game will not merge them.

Run `/lootbox audit` to see exactly which reward items are affected, then hold one freshly obtained copy and run `/lootbox sync`: every reward of that item is refreshed across every lootbox at once, keeping each reward's weight, amount, delivery method and enabled state.

If the item is unique by construction, such as a player head with a randomized profile id, no two copies stack anywhere in the game and no refresh can fix it. Hand it out as a `COMMAND` reward so its owning plugin mints it on the win.

### Why can I not fast-open a stack of keys?

`fast-open.max-stack-amount` (default `1`) is the largest stack that sneak + right-click may open. A bigger stack is denied and the keys are kept. Split the stack, open it without sneaking, or raise the limit (`0` removes it).
