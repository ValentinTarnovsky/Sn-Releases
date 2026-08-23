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

### I wrote `{group-color}` in `menus.boost-line` and it printed literally. Why?

Because before 1.8.1 that fragment was formatted without it. The boost lines are built one by
one and then handed to the pet cell as the single `{boosts}` value, and a placeholder value is
never scanned again for placeholders, so the cell's own `{group-color}` could not reach inside
them. 1.8.1 resolves the colour while the line itself is being built. The line you already
edited starts working on the next boot: only the COMMENT above the key changed in the shipped
file, your value is kept.

The same release binds `{group-color}` on the boosts menu's three stat cells and four roll
buttons. It still does not exist on `menus.grade-row`, `menus.group-separator` or the
`menus.trait-effect-*` lines, which describe a ladder rung or a trait rather than a pet.

### My box has `[rgb]` in its display-name and chat shows the tag instead of the gradient.

Fixed in 1.8.1. `[rgb]` is a prefix tag: SnLib reads it at the START of a finished line, and a
box or pet name spliced into a message as `{box}` or `{pet}` sits in the middle of one, so the
tag was left as text (`The [rgb]Basic Pet Box failed to open...`) while the ITEM, whose name IS
the whole line, rendered fine. Both `boxes.yml` and `pets/<id>.yml` now have their display-name
tags applied when the file is read. Nothing to change on your side - those files are seed-only
and are not touched.

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

### My pets changed shape after updating to 1.6.0. What happened?

1.6.0 added `formation.shape`, which picks the curve the arc is cut from. `CIRCLE` is the old
behaviour: every pet sits `formation.radius` blocks out. `OVAL` keeps `radius` at the owner's
SIDES and uses the new `formation.back-radius` straight behind them, so the arc hugs the owner's
back and opens out at the flanks.

`config.yml` is managed, which means SnLib inserts the two new keys into your existing file but
never overwrites a value you already had. So your server became an OVAL with your own old
`radius` at the sides and `1.4` blocks behind.

In practice that is a very small change, because a 180-degree arc puts its two ends exactly on
the owner's sides, where an oval and a circle coincide. With the old `arc-degrees: 180`, one pet
and two pets do not move at all, and from three pets up nothing moves more than 0.2 blocks. Set
`formation.shape: CIRCLE` and run `/pets reload` to pin the old look exactly.

Two behaviours are worth knowing before you keep the oval. The pets take an even share of the
*angle*, which is an even spacing on the ground only on a circle - on an oval the ones near the
ends bunch up (about 23% at four pets, 28% at six; one, two and three pets are unaffected). And
`formation.facing: OUTWARD` / `CENTER` point along the circle's radius rather than the oval's true
normal, off by up to 10 degrees; that costs nothing under the default `OWNER_YAW`, which ignores
the angle entirely.

An unknown value is not fatal: the plugin logs one warning per load and falls back to `CIRCLE`.
An **absent** value falls back to `CIRCLE` silently - that is deliberate, and it is what keeps a
config written before 1.6.0 on its old circle. So deleting the key does not restore `OVAL`, it
gives you `CIRCLE`.

### I updated to 1.6.0 but my pets are still the old size and height. Why?

Because that is deliberate. 1.6.0 also changed the SHIPPED defaults - `formation.radius` 1.6 to
2.0, `height-offset` 0.35 to 1.9, `arc-degrees` 180 to 190, `facing-offset` 0 to 180,
`animation.head-size` 0.7 to 1.3 and the bounce to 0.06 / 0.1 - but a managed file only ever
gains missing KEYS, never new VALUES. A new install gets the new look; an existing one keeps
every number you set. Copy the values above into your `config.yml` by hand if you want it.

One to watch: `models.height-offset` is SUMMED with `formation.height-offset`. A BetterModel pet
that should stand on the ground wants roughly the negative of whatever `formation.height-offset`
is on **your** server - about `-1.9` if you adopt the new `1.9`, but still about `-0.35` if you
kept your old value. Copying `-1.9` onto a server that never adopted `1.9` sinks the model about
1.55 blocks into the floor.

### Can I announce only SOME fusions?

Yes, since 1.6.0. `config.yml`'s `fusion.broadcast` is now only the DEFAULT. Any pet can override
it in its own `pets/<id>.yml`:

```yaml
fusion:
  into: gale_sprite
  chance: 35.0
  cost: 25000
  broadcast: true
```

The flag belongs to the PARENT - the pet being consumed, the one whose file declares `into` - so
you announce a fusion by writing the key on the pet players fuse AWAY, not on the one they get.
Leave the key out and that pet follows the global setting. `pets/` is seed-only, so no pet file
you already have receives the key on update: everything keeps following `fusion.broadcast` until
you write it yourself.

On a **fresh** install the two shipped pet files already set it - `stone_golem.yml` has
`broadcast: true` and `ember_fox.yml` has `broadcast: false` - and seed-only means they stay that
way. So turning the global on will not make Ember Fox announce; edit or delete the key in its file
for that. A value that is present but not a boolean (`broadcast: 1`, where `yes` and `on` would
have worked) reads as `false` and logs a warning naming the file.

Fuse All never announces, whatever any pet file says. It rolls per pair and would otherwise post
one line for every winning pair.

### How do players trade pets with each other?

Since 1.7.0, by turning the pet into an item. In the main menu, **shift + right click** a pet in
the storage grid: it leaves the storage and becomes a player head in the player's inventory,
wearing that pet's own texture and carrying its whole state - pet type, level, experience, trait
and the three boost grades. Anyone can then **right click** that head to put the pet into *their*
storage, with everything it had. Hand it over, drop it, put it in a chest, or sell it on a shop
plugin: the head is an ordinary item.

Nothing is destroyed by a refusal. A redeem into a full storage, into a profile that is still
loading, in creative mode while `allow-creative` is off, or with the feature switched off hands
the item straight back with its state intact. Taking a pet out is refused - and the pet is left
exactly where it was - when it is EQUIPPED (unequip it first), when its `pets/<id>.yml` is gone,
or when the player has no free inventory slot. A pet is never dropped on the floor to make the
click work.

The head is not placeable, so clicking a block with it can never place it and destroy the pet on
it, and `boxes.blocked-blocks` applies to the redeem click too - clicking a chest opens the chest.

Switch the whole feature off with `pet-items.enabled: false`, or delete
`templates.pet-entry.shift-right-click-actions` from `guis/main.yml` to stop new pets leaving
storage while the heads already in circulation stay redeemable.

### I updated to 1.7.0 but the pet cells do not mention shift + right click. Is it working?

It is. `guis/main.yml` is managed, so your install received the new
`templates.pet-entry.shift-right-click-actions` key and the feature works immediately. What it
did NOT receive are the two lore lines that advertise it, because those are part of a lore LIST
you already have and the merge never rewrites your own list values. Add them by hand:

```yaml
      - ""
      - "&e&lSHIFT + RIGHT CLICK"
      - "&6Take this pet out as an item you can trade"
```

or delete the whole `lore:` list under `templates.pet-entry` and let the next boot write the
shipped one back.

### How do I put a name above every pet?

Since 1.8.0 that is built in. `config.yml` has a `holograms` band controlling how the text LOOKS -
`height-offset`, `line-spacing`, `scale`, `background` (ARGB hex, empty for none), `shadow`,
`see-through` and `line-width` - plus `default-lines`, the lines used by any pet that does not name
its own. What each pet SAYS is in its `pets/<id>.yml`:

```yaml
hologram:
  enabled: true
  lines:
    - "{group-color}{pet}"
    - "&7Lv. &f{level}&7/&f{level-cap}"
    - "&8{owner}"
  # height-offset: 0.9
```

You can use every placeholder a pet cell of any menu uses - `{pet}`, `{level}`, `{level-cap}`,
`{exp}`, `{exp-next}`, `{percent}`, `{group}`, `{group-color}`, `{trait}`, `{buff}`,
`{buff-value}` - plus `{owner}`. PlaceholderAPI tokens work too and resolve against the pet's
OWNER, not against whoever is reading, because one label is shown to everyone who can see the pet.

Note that `pets/` is seeded once and never merged again, so your existing pet files will NOT
receive a `hologram:` block. They use `holograms.default-lines` instead until you write one. Only
an explicit `enabled: false` silences a pet.

### I updated to 1.8.0 and every pet suddenly has text over it. How do I turn that off?

`config.yml` is managed, so your server received the whole `holograms` band on the first boot after
the update, with `enabled: true` and a two-line default. Set `holograms.enabled: false` and run
`/pets reload` to go back to bare pets, or replace `holograms.default-lines` with `[]` to keep the
feature available for the pets that declare their own lines while every other pet stays silent.

### My model pets have the text inside them. Why?

Because the offset is measured from the pet, and "the pet" is not the same object in both cases. A
head pet's label rides the head itself, which already sits `formation.height-offset` above the
owner's feet. A model pet's label rides the invisible carrier its bones ride, which also carries
`models.height-offset` - so on a server that pushed models down to stand on the ground
(`models.height-offset: -1.9`), a label at `0.9` lands 0.9 blocks above the model's FEET.

Give those pets a bigger `hologram.height-offset` of their own. The shipped `stone_golem.yml` does
exactly that, with `1.2`.

### Why is there no DecentHolograms option? I already run it.

Because a DecentHolograms hologram cannot ride the pet, and riding the pet is the whole point. It
is a server-side hologram with its own tick and its own teleports, per line and per viewer: a
second stream of position updates on a second clock, which is exactly the drift that makes a
follower hologram look wrong when the server hitches. SnLib's own `HologramUtil` cannot be used
either - it has no move call at all and does not expose the entity id, so following a pet would
mean deleting and respawning a real entity several times a second.

What SnPets sends instead is a client-side text entity MOUNTED on the pet. The client positions it
from the pet on every one of its own ticks, and the plugin never sends a single position packet for
a label. Nothing has to be installed and nothing can fall behind.

Two deliberate limits come with that: the text does not bob along with a head pet's bounce (the
bounce is a rendering transformation, which passengers do not inherit), and it always turns to face
the reader rather than staying fixed.

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

### A player has more pets stored than their capacity allows. How?

They opened boxes faster than the pets could be written. Before 1.8.2 the capacity check ran the
instant you clicked, while the pets themselves were saved a few ticks later, so a second click
that arrived in between still saw the old count and was let in again. Shift-clicking six or seven
stacks in a row could leave a storage of 54 holding 107 pets.

Update to 1.8.2. Each open now holds the places it was granted for as long as its pets are in
flight, so a simultaneous open sees them as already taken: the excess is trimmed on the click
that does not fit, the boxes it would have opened are returned, and `storage-full` and
`box-partial` quote the corrected numbers. Nothing to configure.

Storages that are already over capacity stay as they are - the update stops the overfill, it does
not delete anyone's pets. Nothing new enters until the player is back under their limit, which is
the same rule that has always applied after unequipping a pet into a full storage.

### How do I give a rank more pet slots or storage?

Grant `snpets.slots.<n>` or `snpets.storage.<n>` in your permissions plugin. The highest value
a player holds wins, so stacking nodes across ranks is safe. The value is read on join, so a
rank change applies the next time the player logs in.

### I gave someone 100 slots and they only got 7. Why?

`slots.max-count` in `config.yml`, which ships at `7` from 1.9.0 on. It is the ceiling on the
purchased slots an admin command may leave, and a command past it is clamped rather than
cancelled - so the grant went through, it just stopped at 7, and the admin was told so before the
usual confirmation. `7` is the number of pet cells the shipped `guis/main.yml` layout can draw;
anything past it would be bought and never usable.

Raise the key if you widened the menu layout, or set it to `0` to remove the ceiling entirely.
`storage.max-capacity` is the same knob for storage and ships at `0`, so storage grants are
unlimited out of the box.

Two things it does not do: it does not cap `snpets.slots.<n>` permission grants, so a rank can
still grant more than a command can; and it never lowers a row by itself. Lowering the key on a
live server leaves players above it alone until the next `slots give`/`set` on them.

### Can I still type `-s` and `-sf` the way I always did?

Yes, nothing about them changed in 1.9.0 - they just tab-complete now, as the last two optional
parameters of every `/pets admin` command. Trailing junk is still accepted in silence, and the
flags are still trailing and order-independent.

Your `lang/messages_en.yml` will grow an `args` entry for each of them under every admin command
on the first boot after the update. That block is only the visible labels of each argument in the
usage line; editing or deleting an entry never changes how a command is typed.

### Can I add my own pets?

Yes. Copy a file in `pets/` and rename it: the file name is the pet id. The folder is yours
after the first boot, so nothing you add or delete there is ever undone by an update. Boxes
work the same way, except they are keys inside `boxes.yml` rather than separate files: copy a
whole top-level block and rename the key.
