# FAQ

### How do I update SnEnvoys?
Download the newer `snenvoys-v*` release and replace the jar. Configs auto-merge on restart.

### Does it support Folia?
No, SnEnvoys is not Folia-compatible.

### Why is the falling Supply Drop block not a chest?
Minecraft draws a falling block with its plain block model, and a chest renders via a
block-entity, so a falling `CHEST` would be invisible on the client. `fall-visual-material`
controls the falling visual on its own (default `BARREL`); `material` still decides the block
that actually lands. Set `fall-visual-material` to `auto` to reuse `material` whenever that
block can be drawn while falling.

### Can I use a custom head instead of a block for envoys?
Yes. Set `envoy.material` to `basehead-<base64>`, pasting the Base64 value from a
minecraft-heads.com listing's "Other / Value" field after the `basehead-` prefix.

### Why can't players in Creative mode claim rewards?
`claims.deny-creative` (default `true`) blocks it, since the reward commands would still run
regardless of game mode otherwise. Set it to `false` to allow Creative claims.
