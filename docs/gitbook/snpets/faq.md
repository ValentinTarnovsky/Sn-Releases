# FAQ

### How do I update SnPets?

Download the newer `snpets-v*` release and replace the jar. `config.yml`, the language files
and the menu layouts auto-merge on restart, so new keys appear while your values and comments
stay. Your `pets/`, `boxes/`, `traits.yml` and `boost-grades.yml` files are never overwritten.

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
