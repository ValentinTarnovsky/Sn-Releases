# FAQ

### How do I update SnCompanions?

Download the newer `sncompanions-v*` release and replace the jar. `config.yml`, the language files
and the menu layouts auto-merge on restart, so new keys appear while your values and comments
stay. Your `companions/` folder and your `eggs.yml` file are
never overwritten.

### A player unequipped the companion in slot 2 and the others moved. Is that a bug?

No, that is 1.3.0's behaviour. Equipped companions are kept in slots `1..n` with no gap, so the free
slots are always the last ones. The order of the companions is unchanged and the formation around the
player looks exactly the same; only the slot numbers shift. An unequip performed while the
player is OFFLINE leaves the gap until they next log in, where it is closed automatically.

### How do I colour a companion's menu lines by its group?

Give the group a `color:` in `config.yml` and use `{group-color}` in the menu templates. See
[Configuration](configuration.md) for the full recipe. Existing configs do not receive the new
key: until you add one, `{group-color}` reuses the colour codes that group's `display` already
starts with.

### My egg has `[rgb]` in its display-name and chat shows the tag instead of the gradient.

Fixed in 1.8.1. `[rgb]` is a prefix tag: SnLib reads it at the START of a finished line, and an
egg or companion name spliced into a message as `{egg}` or `{companion}` sits in the middle of one, so the
tag was left as text (`The [rgb]Basic Egg gave you...`) while a line whose name IS
the whole line rendered fine. Both `eggs.yml` and `companions/<id>.yml` now have their display-name
tags applied when the file is read. Nothing to change on your side - those files are seed-only
and are not touched.

### After updating to 1.2.1 my empty bulk delete buttons are still grey glass. Why?

Because the merge adds missing keys but never overwrites a value your file already carries, and
that material is a value. Open `guis/bulk_delete.yml`, set `templates.group-empty.material` to
`"{icon}"`, and restart. Deleting the file and restarting reseeds the whole thing instead. New
installs already ship the fixed value.

### Can I run an admin command without telling anyone?

Yes. Add `-s` to skip the message the target player would get, `-sf` to skip your own confirmation,
or both, in any order:

```
/companions admin give Bob ember_fox 3 5 -s -sf
/companions admin openegg Bob basic_egg 10 -s
/companions admin clear Bob all -sf
```

`-sf` only hides confirmations of things that WORKED. If the command is refused - a full storage,
a companion that does not exist, an unknown egg - or if a query fails, you are told regardless. Silence
under `-sf` therefore means "it worked", never "something went wrong and you missed it".

Put the flags at the END of the line. Anything typed after the first flag is ignored, so
`/companions admin give Bob ember_fox -s 5` gives one companion rather than five.

### My players never used to be told when I gave them a companion. Did that change?

Yes, in 1.5.0. `/companions admin give` sends the receiver a line of their own,
`messages.companion-received`, a key merged into your existing lang file automatically on the first
boot after the update, so you can restyle or blank it like any other message and suppress it with
`-s`. Offline players are never messaged. `/companions admin openegg` is different on purpose: what
the player reads is the egg's own reward line, announced by the egg engine exactly as it would be
for a bought egg, so `-s` does not touch it.

### My companions changed shape after updating to 1.6.0. What happened?

1.6.0 added `formation.shape`, which picks the curve the arc is cut from. `CIRCLE` is the old
behaviour: every companion sits `formation.radius` blocks out. `OVAL` keeps `radius` at the owner's
SIDES and uses the new `formation.back-radius` straight behind them, so the arc hugs the owner's
back and opens out at the flanks.

`config.yml` is managed, which means SnLib inserts the two new keys into your existing file but
never overwrites a value you already had. So your server became an OVAL with your own old
`radius` at the sides and `1.4` blocks behind.

In practice that is a very small change, because a 180-degree arc puts its two ends exactly on
the owner's sides, where an oval and a circle coincide. With the old `arc-degrees: 180`, one companion
and two companions do not move at all, and from three companions up nothing moves more than 0.2 blocks. Set
`formation.shape: CIRCLE` and run `/companions reload` to pin the old look exactly.

Two behaviours are worth knowing before you keep the oval. The companions take an even share of the
*angle*, which is an even spacing on the ground only on a circle - on an oval the ones near the
ends bunch up (about 23% at four companions, 28% at six; one, two and three companions are unaffected). And
`formation.facing: OUTWARD` / `CENTER` point along the circle's radius rather than the oval's true
normal, off by up to 10 degrees; that costs nothing under the default `OWNER_YAW`, which ignores
the angle entirely.

An unknown value is not fatal: the plugin logs one warning per load and falls back to `CIRCLE`.
An **absent** value falls back to `CIRCLE` silently - that is deliberate, and it is what keeps a
config written before 1.6.0 on its old circle. So deleting the key does not restore `OVAL`, it
gives you `CIRCLE`.

### I updated to 1.6.0 but my companions are still the old size and height. Why?

Because that is deliberate. 1.6.0 also changed the SHIPPED defaults - `formation.radius` 1.6 to
2.0, `height-offset` 0.35 to 1.9, `arc-degrees` 180 to 190, `facing-offset` 0 to 180,
`animation.head-size` 0.7 to 1.3 and the bounce to 0.06 / 0.1 - but a managed file only ever
gains missing KEYS, never new VALUES. A new install gets the new look; an existing one keeps
every number you set. Copy the values above into your `config.yml` by hand if you want it.

One to watch: `models.height-offset` is SUMMED with `formation.height-offset`. A BetterModel companion
that should stand on the ground wants roughly the negative of whatever `formation.height-offset`
is on **your** server - about `-1.9` if you adopt the new `1.9`, but still about `-0.35` if you
kept your old value. Copying `-1.9` onto a server that never adopted `1.9` sinks the model about
1.55 blocks into the floor.

### Can I announce only SOME fusions?

Yes, since 1.6.0. `config.yml`'s `fusion.broadcast` is now only the DEFAULT. Any companion can override
it in its own `companions/<id>.yml`:

```yaml
fusion:
  into: gale_sprite
  chance: 35.0
  cost: 25000
  broadcast: true
```

The flag belongs to the PARENT - the companion being consumed, the one whose file declares `into` - so
you announce a fusion by writing the key on the companion players fuse AWAY, not on the one they get.
Leave the key out and that companion follows the global setting. `companions/` is seed-only, so no companion file
you already have receives the key on update: everything keeps following `fusion.broadcast` until
you write it yourself.

On a **fresh** install the two shipped companion files already set it - `stone_golem.yml` has
`broadcast: true` and `ember_fox.yml` has `broadcast: false` - and seed-only means they stay that
way. So turning the global on will not make Ember Fox announce; edit or delete the key in its file
for that. A value that is present but not a boolean (`broadcast: 1`, where `yes` and `on` would
have worked) reads as `false` and logs a warning naming the file.

Fuse All never announces, whatever any companion file says. It rolls per pair and would otherwise post
one line for every winning pair.

### How do players trade companions with each other?

Since 1.7.0, by turning the companion into an item. In the main menu, **shift + right click** a companion in
the storage grid - or, since 1.11.0, press **Q** over it, the drop key, Ctrl+Q included: it
leaves the storage and becomes a player head in the player's inventory,
wearing that companion's own texture and carrying its whole state - companion type, level and
experience. A **right click** with that head puts the companion into the clicker's
storage, with everything it had. Hand it over, drop it, put it in a chest, or sell it on a shop
plugin: the head is an ordinary item.

Since **1.17.0** the head also remembers WHO took it out, and only that player can redeem it -
see [Can somebody else redeem my companion item?](#can-somebody-else-redeem-my-companion-item) below.

Nothing is destroyed by a refusal. A redeem into a full storage, into a profile that is still
loading, in creative mode while `allow-creative` is off, or with the feature switched off hands
the item straight back with its state intact. Taking a companion out is refused - and the companion is left
exactly where it was - when it is EQUIPPED (unequip it first), when its `companions/<id>.yml` is gone,
or when the player has no free inventory slot. A companion is never dropped on the floor to make the
click work.

The head is not placeable, so clicking a block with it can never place it and destroy the companion on
it, and `companion-items.blocked-blocks` makes the redeem click step aside - clicking a chest opens the chest.

Both triggers run the same action with the same refusals. `companion-items.enabled: false` switches the
whole feature off, both triggers at once, and leaves the heads already in circulation redeemable.

{% hint style="warning" %}
**Correction to what this page used to say.** Up to 1.10.0 this answer told you that deleting
`templates.companion-entry.shift-right-click-actions` from `guis/main.yml` stops new companions leaving
storage. It does not, and never did: `guis/main.yml` is managed and carries no extensible marker,
so the next boot merges the deleted key straight back. The same applies to `drop-click-actions`.
See [Taking a companion out: the two triggers](configuration.md#taking-a-companion-out-the-two-triggers) for
the ways that do hold.
{% endhint %}

### Can somebody else redeem my companion item?

Not since **1.17.0**, if the head was taken out on 1.17.0 or later. Extracting a companion now stamps
the head with your UUID and your name; a player who right clicks a head that is not theirs is
refused with `messages.companion-item-not-yours`, which names the owner, and the head goes back into
their inventory untouched. Nothing is destroyed and nothing is consumed.

The UUID is what the lock matches on, never the name. Changing your nick does not cost you your
companions, and a player who takes your old nick does not gain them.

{% hint style="info" %}
**Every companion head extracted before 1.17.0 stays redeemable by anyone.** Those items carry no owner
tag at all, and nothing on the server records who took them out, so there is no owner to restore -
they keep the exact behaviour they were traded under. Only companions taken out from 1.17.0 onwards are
locked.
{% endhint %}

There is no config switch for this: an item with no owner tag is free and an item with one is
locked, which is the whole rule. Redeeming your own head works exactly as before, and the companion it
creates belongs to whoever redeemed it - so a player who redeems a head and later takes the same
companion out again is stamped as its owner in their turn.

{% hint style="warning" %}
**This deliberately narrowed item trading.** Up to 1.16.0 handing
over the head handed over the companion, and that stopped being true for a head taken out on 1.17.0 or
later: the receiver holds an item they cannot redeem. The only way to move a locked companion to
another owner is `/companions admin setowner <player> <instance> <target>`, which is an admin action
by design - a lock a player could lift themselves would not be a lock. Heads extracted before
1.17.0 were never locked at all.
{% endhint %}

### How does somebody claim a companion item that is not theirs?

They do not - a player cannot lift the lock themselves. An admin moves the companion with
`/companions admin setowner <player> <instance> <target>`, which transfers the companion to another
player outright. There is no player-facing path, on purpose: a lock the holder of the item could
open would not be a lock.

A head that carries no owner tag at all - anything extracted before 1.17.0 - was never locked, and
stays redeemable by whoever holds it.

### I updated but the companion cells do not mention shift + right click or Q. Is it working?

It is. `guis/main.yml` is managed, so your install received the
`templates.companion-entry.shift-right-click-actions` key (1.7.0) and `drop-click-actions` (1.11.0) and
both triggers work immediately. What it did NOT receive are the lore lines that advertise them,
because those are part of a lore LIST you already have and the merge never rewrites your own list
values. Add them by hand:

```yaml
      - ""
      - "&e&lSHIFT + RIGHT CLICK"
      - "&e&lQ (DROP)"
      - "&6Take this companion out as an item you can trade"
```

or delete the whole `lore:` list under `templates.companion-entry` and let the next boot write the
shipped one back. Upgrading from 1.10.0 or earlier, `&e&lQ (DROP)` is the single line you are
missing.

### How do I put a name above every companion?

Since 1.8.0 that is built in. `config.yml` has a `holograms` band controlling how the text LOOKS -
`height-offset`, `line-spacing`, `scale`, `background` (ARGB hex, empty for none), `shadow`,
`see-through` and `line-width` - plus `default-lines`, the lines used by any companion that does not name
its own. What each companion SAYS is in its `companions/<id>.yml`:

```yaml
hologram:
  enabled: true
  lines:
    - "{group-color}{companion}"
    - "&7Lv. &f{level}&7/&f{level-cap}"
    - "&8{owner}"
  # height-offset: 0.9
```

You can use every placeholder a companion cell of any menu uses - `{companion}`, `{level}`, `{level-cap}`,
`{exp}`, `{exp-next}`, `{percent}`, `{group}`, `{group-color}`, `{buff}`,
`{buff-value}` - plus `{owner}`. PlaceholderAPI tokens work too and resolve against the companion's
OWNER, not against whoever is reading, because one label is shown to everyone who can see the companion.

Note that `companions/` is seeded once and never merged again, so your existing companion files will NOT
receive a `hologram:` block. They use `holograms.default-lines` instead until you write one. Only
an explicit `enabled: false` silences a companion.

### I updated to 1.8.0 and every companion suddenly has text over it. How do I turn that off?

`config.yml` is managed, so your server received the whole `holograms` band on the first boot after
the update, with `enabled: true` and a two-line default. Set `holograms.enabled: false` and run
`/companions reload` to go back to bare companions, or replace `holograms.default-lines` with `[]` to keep the
feature available for the companions that declare their own lines while every other companion stays silent.

### My model companions have the text inside them. Why?

Because the offset is measured from the companion, and "the companion" is not the same object in both cases. A
head companion's label rides the head itself, which already sits `formation.height-offset` above the
owner's feet. A model companion's label rides the invisible carrier its bones ride, which also carries
`models.height-offset` - so on a server that pushed models down to stand on the ground
(`models.height-offset: -1.9`), a label at `0.9` lands 0.9 blocks above the model's FEET.

Give those companions a bigger `hologram.height-offset` of their own. The shipped `stone_golem.yml` does
exactly that, with `1.2`.

### Why is there no DecentHolograms option? I already run it.

Because a DecentHolograms hologram cannot ride the companion, and riding the companion is the whole point. It
is a server-side hologram with its own tick and its own teleports, per line and per viewer: a
second stream of position updates on a second clock, which is exactly the drift that makes a
follower hologram look wrong when the server hitches. SnLib's own `HologramUtil` cannot be used
either - it has no move call at all and does not expose the entity id, so following a companion would
mean deleting and respawning a real entity several times a second.

What SnCompanions sends instead is a client-side text entity MOUNTED on the companion. The client positions it
from the companion on every one of its own ticks, and the plugin never sends a single position packet for
a label. Nothing has to be installed and nothing can fall behind.

One deliberate limit comes with that: the text does not bob along with a head companion's bounce (the
bounce is a rendering transformation, which passengers do not inherit). The DecentHolograms LOOK is
not a limit though - see the next question.

### The text above my companions leans towards me and it looks bad. Can it stay straight?

Yes, and from 1.10.0 it does by default. Labels are drawn with the `vertical` billboard: they turn
only around the vertical axis, so the lines stay upright however far above or below them you stand,
which is the DecentHolograms look. Before 1.10.0 they used `center`, which turns on both axes and
tilts the whole stack towards the camera.

`config.yml` is managed, so upgrading is enough - `holograms.billboard: vertical` arrives on the
next boot. If you preferred the tilt, set that key to `center` and everything else stays as it is.
`horizontal` and `fixed` are also accepted; neither reads well on a name plate. The setting is
global, with no per-companion override, so a server has one label aesthetic rather than a mix.

### Does it support Folia?

No, SnCompanions is not Folia-compatible. Run it on Paper 1.20.4 or newer. Both the 1.20 and 1.21
lines are supported.

### Do companions lag the server with many players online?

No. Companions are packets, not entities, so they never enter the entity table, never tick physics
and never get saved to the region files. The whole server shares one animation tick rather than
one task per companion or per owner, so adding players adds work linearly, not a scheduler task each.
An unequipped companion costs nothing at all.

### Do I need BetterModel?

No. Without it every companion renders as a player head, which is the built-in default. Install it
only if you want a companion type to render as an animated model with separate idle and moving
animations.

### A player says their companions vanished. What happened?

Check `/companions toggle` first: it hides the player's own companions and is the usual answer. If other
players cannot see them either, check `/companions hide` on the viewer's side, then the `worlds`
section in `config.yml`, which can disable rendering in a named world.

### Why does a player receive no buff in one world?

Buffs have a per-world gate in `config.yml`. In a gated world the placeholder reports zero
because the player genuinely receives nothing there. That is configuration, not a bug.

### Can two players see each other's companions?

Yes, unless the viewer ran `/companions hide` or the world gates rendering off. Each viewer's own
preference is respected, so one player hiding companions never affects anyone else's view.

### A player has more companions stored than their capacity allows. How?

They opened faster than the companions could be written. Before 1.8.2 the capacity check ran the
instant you clicked, while the companions themselves were saved a few ticks later, so a second click
that arrived in between still saw the old count and was let in again. Six or seven opens in a row
could leave a storage of 54 holding 107 companions.

Update to 1.8.2. Each open now holds the places it was granted for as long as its companions are in
flight, so a simultaneous open sees them as already taken: the one that does not fit is refused
outright and costs nothing. Nothing to configure.

Storages that are already over capacity stay as they are - the update stops the overfill, it does
not delete anyone's companions. Nothing new enters until the player is back under their limit, which is
the same rule that has always applied after unequipping a companion into a full storage.

### The refusal said `(57/54)` on a storage of 54. Is the limit being bypassed?

No, and on 1.12.1 the message no longer says that. The limit was always enforced - what you were
reading was a counting artefact in the message itself. While an open's companions are being written, the
places they will take are held for them, and for a fraction of a second a companion can be visible as
both "held" and "already stored". The decision was correct throughout (there really was no room),
but the number quoted added those places twice, so a stack spammed fast enough produced `(57/54)`,
`(62/54)`, `(67/54)`.

Update to 1.12.1. Held places may now raise the quoted number up to the capacity and no further,
so the refusals read `(54/54)`. Nothing to configure, and nothing about what is accepted or refused
changed - only the number shown.

A number above the capacity is still shown when it is real: a storage left over its limit by a
historic overfill, an admin grant or a capacity you lowered will correctly say `(107/54)`, because
that is the player's actual state and not a companion counted twice.

### How do I give a rank more companion slots or storage?

Grant `sncompanions.slots.<n>` or `sncompanions.storage.<n>` in your permissions plugin. The highest value
a player holds wins, so stacking nodes across ranks is safe. The value is read on join, so a
rank change applies the next time the player logs in. Since 1.22.0 the permission is the FLOOR:
it replaces the config base when higher, and everything sold with `/companions admin slots|storage
give` adds on top of it.

### I gave a player 1 slot and their total did not go up. Why?

You are on 1.21.0 or earlier. Until then the effective count was the *highest* of the config
base, the rank permission and the purchased value, so with `slots.base-count: 1` a first
purchased slot vanished into the base the player already had. Since 1.22.0 purchases ADD on top
of `max(base, permission)`: give 1 slot on a stock install and the total goes from 1 to 2.
Nothing has to be migrated - existing purchases simply start counting the moment 1.22.0 boots.

### I gave someone 100 slots and they only got 7. Why?

`slots.max-count` in `config.yml`, which ships at `7` from 1.9.0 on. It is the ceiling on the
TOTAL slots an admin command may leave (on the purchased slots before 1.22.0), and a command past
it is clamped rather than cancelled - so the grant went through, it just stopped at a total of 7,
and the admin was told so before the usual confirmation. `7` is the number of companion cells the
shipped `guis/main.yml` layout can draw; anything past it would be bought and never usable.

Raise the key if you widened the menu layout, or set it to `0` to remove the ceiling entirely.
`storage.max-capacity` is the same knob for storage and ships at `0`, so storage grants are
unlimited by default.

Two things it does not do: it does not cap `sncompanions.slots.<n>` permission grants, so a rank can
still grant more than a command can; and it never lowers a row by itself. Lowering the key on a
live server leaves players above it alone until the next `slots give`/`set` on them.

### Can I still type `-s` and `-sf` the way I always did?

Yes, nothing about them changed in 1.9.0 - they just tab-complete now, as the last two optional
parameters of every `/companions admin` command. Trailing junk is still accepted in silence, and the
flags are still trailing and order-independent.

Your `lang/messages_en.yml` will grow an `args` entry for each of them under every admin command
on the first boot after the update. That block is only the visible labels of each argument in the
usage line; editing or deleting an entry never changes how a command is typed.

### Can I add my own companions?

Yes. Copy a file in `companions/` and rename it: the file name is the companion id. The folder is yours
after the first boot, so nothing you add or delete there is ever undone by an update. Eggs
work the same way, except they are keys inside `eggs.yml` rather than separate files: copy a
whole top-level block and rename the key.

### Do I need EdTools?

No. It is optional, exactly like BetterModel. Without it no companion grants a booster, no companion can use the
`EDTOOLS_BLOCK_BREAK` experience source, no EdTools class is ever loaded and everything else works
unchanged. Install it only if you want an equipped companion to boost your server's currencies or its
global enchant multiplier, or to level from the blocks your omnitools break.

### I gave a companion `edtools-boosts` and it grants nothing. Why?

Almost certainly the one-decimal rule. SnCompanions never hands EdTools more than one decimal of the
fraction it wants, so the granted boost moves in **steps of 10%**: a summed total below 5% rounds to
zero and the booster is removed rather than written at nothing. A companion with `initial: 2.0` at level 1
is worth 2%, which rounds away.

| Summed percent across every equipped companion | What EdTools receives |
|---|---|
| under 5% | nothing |
| 5% to 14% | +10% |
| 15% to 24% | +20% |
| 25% to 34% | +30% |

Write your companions in tens if you want what you wrote. Two other things to check: the console line at
boot must say `EdTools detected`, and an unknown currency id logs one warning naming it.

Since **1.15.0**, check the entry's `max:` too if it has one. The ceiling is applied to that companion
BEFORE the totals are summed and before this rounding, so a companion capped at `max: 4` contributes 4
and, on its own, still rounds away to nothing. That ordering is also what makes the ceiling safe to
use: three companions capped at 4 sum to 12 and grant +10%, which capping after the rounding would have
lost. A `max:` of `0`, a negative one, or no `max:` at all all mean no ceiling.

### My companion's menu line says "Buff Damage: 0.0%" even though the companion boosts currencies

That was the pre-1.20.0 behavior. `{buff}` and `{buff-value}` used to show only the vanilla
`buff:` block - a companion that declares none defaults to a damage buff worth zero at every level,
which is the "Damage 0.0%" you saw, no matter the companion's level. Since **1.20.0** a companion whose
vanilla buff grants nothing resolves both placeholders from its `edtools-boosts` block instead:
`{buff}` shows the boosted currencies (named per your `messages.edtools-currency-<id>` lang
entries, the raw id otherwise) and `{buff-value}` the live value at the companion's current level, cap
and widening included. A companion that declares a real vanilla `buff:` is untouched. If you still see
the zero line on 1.20.0, the companion's `edtools-boosts` block grants nothing at its level - see the
one-decimal question above.

### I already run SnCompanions. Why do my companions have no `edtools-boosts` block?

Because `companions/` is seeded once and never merged again, which is the same reason your companion files did
not grow a `hologram:` block in 1.8.0. The commented example ships in `companions/ember_fox.yml` for a
fresh install only; on a server you already run you paste the block into the companion files yourself.
The `edtools` band of `config.yml` does arrive on its own, because that file is managed.

### How do I turn the companion boosters off without uninstalling EdTools?

Set `edtools.enabled: false` and run `/companions reload`. The boosters already granted are removed
immediately; you do not need to restart. Turning it back on and reloading writes them again.

### Can a companion boost one specific enchant?

No, only the **global** enchant multiplier. That is the whole of what EdTools exposes to other
plugins, so it is a limit of the integration rather than a decision SnCompanions made. Use one of
`enchants`, `enchant`, `global-enchants` or `encantamientos` as the key.

### My companions get no experience from EdTools mining. Why?

Because `BLOCK_BREAK` does not see it. EdTools consumes the blocks its omnitools break without ever
firing a vanilla `BlockBreakEvent`, so on a farming server those two count completely different
things. Set the companion's `experience.source` to **`EDTOOLS_BLOCK_BREAK`** instead, and leave
`experience.sources.edtools-block-break` on in `config.yml`. `companions/<id>.yml` is seed only, so you
edit the companion files you already have by hand; the config key arrives on its own.

### Can a companion level from only one of my EdTools tools?

Yes. Add an optional `experience.tools` list to that companion's file, holding the EdTools tool ids as
EdTools itself names them:

```yaml
experience:
  source: EDTOOLS_BLOCK_BREAK
  ratio: 0.5
  tools:
    - crop-tool
```

The ids are matched case-insensitively and are never validated against EdTools, so one that names no
tool simply never matches. Leave the list out (or empty) and every omnitool counts. The
`experience.materials` whitelist still applies on top, exactly as it does for `BLOCK_BREAK`.

### Does the EdTools experience source lag the server?

No. It is the busiest event a farming server produces - thousands a second with bulk enchants - and
EdTools fires it off the main thread, so the handler does one cancel check and one counter
increment and nothing else. A single shared task pays the totals out coalesced per
`(player, block, tool)` every 5 ticks. There is no task per block, no one-tick timer, and no task
per player. On a server without EdTools the listener is never registered and the task never runs.

### I turned EdTools off mid-session. Do I have to restart?

No. Disabling or enabling EdTools unregisters or registers the break listener on its own, and stops
or starts the drain with it. The same is true if you install EdTools after SnCompanions has already
started.
