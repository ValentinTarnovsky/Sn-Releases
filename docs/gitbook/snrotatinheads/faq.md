# FAQ

### The head shows no text label.

Labels need DecentHolograms. Without it, heads spin, bounce and click normally, and every
`/rh hologram add` answers with a notice that the label is not shown. Install DecentHolograms, or
check `hologram.enabled` in `config.yml`, then run `/rh reload`. If DecentHolograms was enabled after
this plugin, the labels are re-anchored automatically.

### The label sits too high, too low, or its lines are too far apart.

Both are per head: `/rh hologram offset <id> <blocks>` moves the whole label up or down (a
negative value sits it below the anchor) and `/rh hologram spacing <id> <blocks>` sets the distance
between the lines (`0.25` is the DecentHolograms default). `hologram.y-offset` and
`hologram.line-height` in `config.yml` only decide what a NEW head starts with.

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

### Can a head show a custom model or an ItemsAdder / Nexo item instead of a skull?

Yes. Hold the item and run `/rh model <id> hand`: the head takes its material and its model selector
(`item_model` on 1.21.2 or newer, otherwise `custom-model-data`). You can also type it:
`/rh model <id> paper:1001` or `/rh model <id> paper:mypack:crown`. `/rh model <id> player_head`
goes back to the textured skull. If the model renders small, offset or tilted, change its display
context with `/rh transform <id> fixed` (or `head`); `ground` is the default and is what skulls use.

### Can a head be an animated Blockbench model (bones, idle animation)?

Yes, through a model engine. Install ModelEngine (R4) or BetterModel (2.2.0 or newer; 3.x needs
Java 25 on the server), load the `.bbmodel` in the engine, then `/rh model <id> meg:<model>` or
`/rh model <id> bm:<model>`. The head spins, bounces, keeps its label and clicks, and plays the
`idle` animation on loop by default; change it with `/rh animation set <id> <name>`. `size` scales
the model.

### Can the model play an extra animation every so often (a wave, a dance)?

Yes: `/rh animation add <id> <seconds> <name>` plays `<name>` once every `<seconds>` on top of the
looping animation, and the model returns to the loop by itself when it ends. Add as many timed
animations as you want (`/rh animation list`, `remove <index>`, `clear` manage them); the minimum
period is 1 second and the first play happens one full period after the head spawns.

### My engine-model head is invisible but the label and clicks work.

The engine has no such model right now: it is not installed, still loading its models, reloading,
or the model was renamed. Check the engine's console output and `/rh info <id>` for the model name.
The head retries on its own every second and appears as soon as the model is there.

### I set a texture and nothing changed.

The head shows a model that is not a player head, so there is no skin to render; the command says
so. Run `/rh model <id> player_head` and the texture shows again.

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
