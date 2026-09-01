# FAQ

### How do I update SnLootBoxes?

Download the newer `snlootboxes-v*` release and replace the jar. Configs auto-merge on restart.

### Does it support Folia?

No, SnLootBoxes is not Folia-compatible.

### Why do my old keys not stack with new ones?

They do, once the plugin has seen them. Every key a player holds is re-rendered from the current lootbox definition when the player joins and right before any `/lootbox give`, `/lootbox giveall` or editor key grant. Keys from before a reward edit, or from a time when `key-items.stackable` was `false`, then merge with new keys. Set `key-items.stackable: false` to keep every grant on its own stack instead.

### Why can I not fast-open a stack of keys?

`fast-open.max-stack-amount` (default `1`) is the largest stack that sneak + right-click may open. A bigger stack is denied and the keys are kept. Split the stack, open it without sneaking, or raise the limit (`0` removes it).
