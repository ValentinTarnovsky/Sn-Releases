# FAQ

### How do I update SnKillEffects?

Download the newer `snkilleffects-v*` release and replace the jar. Configs auto-merge on restart, so your edits and comments survive and any newly shipped effect appears in `effects.yml` on its own.

### Does it support Folia?

No, SnKillEffects is not Folia-compatible. It targets Paper 1.20.x and 1.21.x.

### Can an effect kill or hurt somebody?

No. No effect deals damage, sets anything on fire, knocks a player back or changes a block. The lightning is the visual-only bolt, the explosions are particles and sound, the falling anvil is a display entity that places nothing, and the plugin cancels any damage its own fireworks would deal.

### How do I sell effects on my store?

Have your store run the give command from console. It works on offline players, so a purchase lands whether or not the buyer is connected:

```
killeffects give %player% rocket
killeffects give %player% nuke 30d
```

A timed grant expires on its own. Granting an effect the player already has extends it rather than replacing it.

### A player deleted an effect from effects.yml and it came back. Why?

Every effect id is bound to code, so a deleted block is restored on the next boot to let plugin updates ship new effects. To turn an effect off, set its `enabled: false` instead. A disabled effect leaves the menu, cannot be selected or granted, and never plays, even for someone who already owned it.

### Why does a player still hear an effect after using `/killeffects toggle`?

The toggle hides other players' particles and display entities, but the server broadcasts sounds and the lightning flash itself, so those cannot be filtered per player. Everything else is hidden.

### Can players buy effects in game?

Not in this version. Effects are delivered by command, which is what a web store runs. The locked entries in the menu run a configurable action list you can point at your store: see `locked-click-actions` in `config.yml`.

### Where do I put my license key?

In `plugins/.Sn-License/license.yml`, which the plugin creates on its first start. It is one shared file for every Sn bundle plugin on the server, so a single key unlocks the whole pack.
