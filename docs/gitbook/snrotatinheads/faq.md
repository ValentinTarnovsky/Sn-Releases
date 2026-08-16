# FAQ

### The head shows no text label.

Labels need DecentHolograms. Without it, heads spin, bounce and click normally, and every
`/rh hologram add` answers with a notice that the label is not shown. Install DecentHolograms, or
check `hologram.enabled` in `config.yml`, then run `/rh reload`. If DecentHolograms was enabled after
this plugin, the labels are re-anchored automatically.

### The head does not react to left click, right click works.

Left click arrives as an attack on the hitbox. A plugin that cancels attacks before they reach the
entity (some hub and anti-PvP plugins) can swallow it. Right click uses a different event and is
unaffected. Bind the action to `right` or `any` for such setups.

### The head is not clickable at the top of its bounce.

It should be: the hitbox is stretched by the full bounce height. If you changed `bounce-height` by
hand in `heads.yml`, run `/rh reload` so the hitbox is rebuilt.

### I edited heads.yml and nothing changed.

Run `/rh reload`. It re-reads the file, removes every head and label, purges leftovers and respawns
everything. Ids are stored lower case; a mixed-case id in the file is normalised on load.

### heads.yml says it could not be read and nothing saves.

The file has a YAML mistake, an entry that is not a map, an id containing a dot, or two ids that
differ only in case. The plugin refuses to save until you fix or remove the file and run `/rh reload`,
so nothing is overwritten. Every command answers with the same notice until then.

### Heads spin but look static when I walk around them.

They should keep spinning at a constant rate from every angle. If they seem to face you, another
plugin is changing the display billboard; SnRotatinHeads uses a fixed billboard on purpose.

### Which texture formats does /rh texture accept?

A base64 `textures` value (the long `eyJ...` string), a `texture:` or `base64:` prefixed value, or a
skin URL (`http://textures.minecraft.net/texture/...`). Anything else is rejected with a message.

### Are click actions the same syntax as in other Sn plugins?

Yes. Every line is `[tag] argument`, the SnLib action syntax used by the whole bundle. A plain command
without a tag is rejected on `action add`. See the commands page for the most useful tags.

### How do I update SnRotatinHeads?
Download the newer `snrotatinheads-v*` release and replace the jar. Configs auto-merge on restart;
`heads.yml` is never touched by an update.

### Does it support Folia?
No, SnRotatinHeads is not Folia-compatible: the shared animation task and the entity work run on the
main server thread.
