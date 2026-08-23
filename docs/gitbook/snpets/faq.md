# FAQ

### How do I update SnPets?

Download the newer `snpets-v*` release and replace the jar. `config.yml`, the language files
and the menu layouts auto-merge on restart, so new keys appear while your values and comments
stay. Your `pets/`, `boxes/`, `traits.yml` and `boost-grades.yml` files are never overwritten.

### Can players skip the trait and boost roulette animation?

Yes, since 1.3.0: clicking the spinning cell reveals the result at once. It can never change
what came out, because the roll is decided and saved before the first frame is drawn. The
`click-actions` key that arms it merges into `guis/boosts.yml` and `guis/traits.yml` on the
first boot after the update; the `&eClick to skip the animation` hint is a new line in an
existing `lore` list, so add that one by hand if you want it shown. Delete the `click-actions`
block on the `spinning` template to force the animation to always run to the end.

### A player unequipped the pet in slot 2 and the others moved. Is that a bug?

No, that is 1.3.0's behaviour. Equipped pets are kept in slots `1..n` with no gap, so the free
slots are always the last ones. The order of the pets is unchanged and the formation around the
player looks exactly the same; only the slot numbers shift. An unequip performed while the
player is OFFLINE leaves the gap until they next log in, where it is closed automatically.

### How do I colour a pet's menu lines by its group?

Give the group a `color:` in `config.yml` and use `{group-color}` in the menu templates. See
[Configuration](configuration.md) for the full recipe. Existing configs do not receive the new
key: until you add one, `{group-color}` reuses the colour codes that group's `display` already
starts with.

### After updating to 1.2.1 my empty bulk delete buttons are still grey glass. Why?

Because the merge adds missing keys but never overwrites a value your file already carries, and
that material is a value. Open `guis/bulk_delete.yml`, set `templates.group-empty.material` to
`"{icon}"`, and restart. Deleting the file and restarting reseeds the whole thing instead. New
installs already ship the fixed value.

### A player picked a pet in Boosts, closed the menu and it was still picked. Is that fixed?

Yes, since 1.2.1. Closing every SnPets menu forgets the pick, while moving between menus keeps
it. It is driven by the `close-actions` key each `guis/*.yml` now carries; the key merges in on
the first boot after the update.

### Does it support Folia?

No, SnPets is not Folia-compatible. Run it on Paper 1.20.4 or newer. Both the 1.20 and 1.21
lines are supported.

### Do pets lag the server with many players online?

No. Pets are packets, not entities, so they never enter the entity table, never tick physics
and never get saved to the region files. The whole server shares one animation tick rather than
one task per pet or per owner, so adding players adds work linearly, not a scheduler task each.
An unequipped pet costs nothing at all.

### Do I need BetterModel?

No. Without it every pet renders as a player head, which is the built-in default. Install it
only if you want a pet type to render as an animated model with separate idle and moving
animations.

### A player says their pets vanished. What happened?

Check `/pets toggle` first: it hides the player's own pets and is the usual answer. If other
players cannot see them either, check `/pets hide` on the viewer's side, then the `worlds`
section in `config.yml`, which can disable rendering in a named world.

### Why does a player receive no buff in one world?

Buffs have a per-world gate in `config.yml`. In a gated world the placeholder reports zero
because the player genuinely receives nothing there. That is configuration, not a bug.

### Can two players see each other's pets?

Yes, unless the viewer ran `/pets hide` or the world gates rendering off. Each viewer's own
preference is respected, so one player hiding pets never affects anyone else's view.

### A player opened a box and got nothing. Are the pets lost?

No. If the database write fails, the boxes are handed straight back and the player is told, so
nothing is consumed. If storage filled up between the click and the grant, the open lands
partially on purpose and reports how many pets did not fit. Free storage and open the rest.

### How do I give a rank more pet slots or storage?

Grant `snpets.slots.<n>` or `snpets.storage.<n>` in your permissions plugin. The highest value
a player holds wins, so stacking nodes across ranks is safe. The value is read on join, so a
rank change applies the next time the player logs in.

### Can I add my own pets?

Yes. Copy a file in `pets/` and rename it: the file name is the pet id. The folder is yours
after the first boot, so nothing you add or delete there is ever undone by an update. The same
is true for `boxes/`.
