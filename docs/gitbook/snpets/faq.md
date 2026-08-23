# FAQ

### How do I update SnPets?

Download the newer `snpets-v*` release and replace the jar. `config.yml`, the language files
and the menu layouts auto-merge on restart, so new keys appear while your values and comments
stay. Your `pets/` folder and your `traits.yml`, `boost-grades.yml` and `boxes.yml` files are
never overwritten.

Updating **to 1.4.0** also moves your `boxes/` folder into a single `boxes.yml` automatically -
see the question about `boxes-migrated/` below.

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

### Can I run an admin command without telling anyone?

Yes. Add `-s` to skip the message the target player would get, `-sf` to skip your own confirmation,
or both, in any order:

```
/pets admin give Bob ember_fox 3 5 -s -sf
/pets admin givebox Bob basic 10 -s
/pets admin clear Bob all -sf
```

`-sf` only hides confirmations of things that WORKED. If the command is refused - a full storage,
a pet that does not exist, an unknown box - or if a query fails, you are told regardless. Silence
under `-sf` therefore means "it worked", never "something went wrong and you missed it".

Put the flags at the END of the line. Anything typed after the first flag is ignored, so
`/pets admin give Bob ember_fox -s 5` gives one pet rather than five.

### My players never used to be told when I gave them a pet. Did that change?

Yes, in 1.5.0. `/pets admin give`, `/pets admin givebox` and `/pets admin giveallbox` now send the
receiver a line of their own - `messages.pet-received` and `messages.box-received`, both new keys
in `lang/messages_<code>.yml`. They are merged into your existing lang file automatically on the
first boot after the update, so you can restyle or blank them like any other message, and you can
suppress them per command with `-s`. `/pets admin currency` already worked this way and is
unchanged. Offline players are never messaged.

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

### My boxes/ folder was renamed to boxes-migrated/. Where did my boxes go?

Into `boxes.yml`, which is where every box lives from 1.4.0 on: one top-level key per box,
named after the file it came from. The migration runs once, on the first boot after updating,
and it never deletes anything - `boxes-migrated/` is your original folder, kept exactly as it
was, including the per-box comments that a YAML reader cannot carry onto a key.

Every box id is preserved, so the box items already in players' inventories and shulkers keep
opening. The one exception is a file name containing a dot (`my.box.yml`), which cannot be a
yml key: it becomes `my_box` and the console says so loudly, because that box's item id changes
and the copies already handed out stop opening.

Once you are happy with `boxes.yml` you can delete `boxes-migrated/` yourself. Nothing reads it.

### My migrated boxes never fail to open. Where is the success chance?

`boxes.yml` is seed-only, so an update never adds keys to it - which is exactly what protects
the boxes you wrote. That means a migrated box has no `chance` block, and a box without one
always opens: identical to how it behaved before 1.4.0. Add the band by hand to any box you
want to turn into a gamble:

```yaml
rare:
  chance:
    min: 60
    max: 90
```

Add `{chance}` to that box's lore too if you want the odds shown on the item.

### A player opened a box and got nothing. Are the pets lost?

If the box **failed its success roll**, that is working as designed: the box is consumed, the
player is told which percentage it was rolling against, and nothing is granted. Only boxes with
a `chance` band below 100 can do this, and the percentage is written on the item itself.

Otherwise, no, nothing is lost. If the database write fails, the boxes are handed straight back
and the player is told, so nothing is consumed. If storage filled up between the click and the
grant, the open lands partially on purpose and reports how many pets did not fit. Free storage
and open the rest.

### How do I give a rank more pet slots or storage?

Grant `snpets.slots.<n>` or `snpets.storage.<n>` in your permissions plugin. The highest value
a player holds wins, so stacking nodes across ranks is safe. The value is read on join, so a
rank change applies the next time the player logs in.

### Can I add my own pets?

Yes. Copy a file in `pets/` and rename it: the file name is the pet id. The folder is yours
after the first boot, so nothing you add or delete there is ever undone by an update. Boxes
work the same way, except they are keys inside `boxes.yml` rather than separate files: copy a
whole top-level block and rename the key.
