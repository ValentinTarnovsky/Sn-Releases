# FAQ

### How do I update SnGens?

Download the newer `sngens-v*` release, replace the jar, and restart. `config.yml`,
`wands.yml`, `storages.yml`, the menu layouts and the language files pick up any new keys
automatically, keeping your values and comments.

`generators.yml`, `events.yml`, `armors.yml` and `offhands.yml` are never rewritten, since they
hold content you own. If a release note mentions a new key inside one of those, add it by hand.

### Does it support Folia?

Yes. The plugin declares Folia support and schedules its per region work through a Folia aware
scheduler. The same jar runs on Paper and on Folia with no configuration change.

### The plugin refuses to enable. What do I check first?

In order:

1. Vault, PlaceholderAPI and DecentHolograms must all be installed. They are hard
   dependencies, so a missing one stops the plugin from loading at all.
2. `plugins/SnGens/license.yml` must contain your real license key, not the placeholder.
3. The server must reach the license backend at startup.

The console line tells you which of the three failed.

### Do generators produce while the owner is offline?

Not by default. `online-only: true` in `config.yml` restricts ticking to generators whose owner
is connected. Set it to `false` for an idle economy where farms keep producing overnight.

A single generator can override the global setting with its own `online-only` key in
`generators.yml`, which is how you make one premium tier keep running while the rest do not.

### Do generators produce in unloaded chunks?

No. The tick loop walks loaded chunks only, so a farm in an unloaded chunk is idle until
something loads it. That is deliberate: it is what keeps a server with hundreds of thousands of
generators playable.

The leaderboard is not affected. It is computed from the database, so generators in unloaded
chunks still count toward `/gens top`.

### Where do I set the price of a generator in the shop?

Nowhere directly. The shop derives every price from the upgrade chain: the first generator
costs `base-cost` from `gui/shop_gui.yml`, and each following tier costs the previous tier's
price plus the previous tier's `upgrade.upgrade-cost`.

So to make a tier more expensive in the shop, raise the `upgrade-cost` of the tier before it.
The full worked table is in [Configuration](configuration.md).

### How do I add a new generator?

Add a top-level key to `generators.yml` and run `/gens reload`. To slot it into the progression,
point the previous last tier's `upgrade.next-generator` at your new id. There is a full worked
example in [Configuration](configuration.md).

### I deleted a generator from generators.yml, but players still have it placed. What happens?

Nothing is destroyed. The plugin first tries to re-bind those placed blocks to a generator whose
item material matches. If none matches, the blocks stay inert, and any pickup that touches them
is skipped with a console warning naming the missing id. The database rows are left alone.

Put the id back and run `/gens reload` to bring them fully back to life.

### How do I give a rank more generator slots or a better sell multiplier?

Two permission families do this, and both are additive across every node the player holds:

```
sngens.max.25       # +25 generator slots
sngens.multi.0_5    # +0.5 sell multiplier
```

The underscore is a decimal point, since a dot separates permission nodes. See
[Permissions](permissions.md).

For a one off grant to a single player, use `/gens addmax <player> <amount>` and
`/gens addmultiplier <player> <amount>`. Those are stored per player and count while offline
too, unlike permission nodes.

### I gave a player an armor set with a sell bonus and nothing changed. Why?

Equipment sell bonuses are multiplicative, not additive. They add a percentage of the multiplier
the player already has, so a player whose multiplier comes from nothing gains nothing.

An example. A player holds `sngens.multi.8` and swings a 2x sellwand, so their additive base is
10. A set granting `sell-multiplier: 10` adds ten percent of that base, for a total of 11. The
same set on a player with no other multiplier adds ten percent of nothing.

Give players a small permission multiplier, and the gear starts to matter.

### My server lags when a lot of generators tick at once.

Work through these in order:

1. Keep `item-stacking.enabled: true`. It is what turns thousands of individual drops into a
   handful of stacked item entities per chunk.
2. Raise `generator-tick-interval-seconds`. Twenty seconds is the default; thirty halves the
   work and players barely notice if you raise drop values to match.
3. Hand out collectors. A collector captures every drop in its chunk before the item entity is
   ever created, which is the single largest saving available.
4. Set `item-stacking.unbounded-stack-size: true` on very large farms, so one entity can carry
   more than 64 items.
5. Run `/gens debugspawn 20` on the affected island and read the JSON dump in
   `plugins/SnGens/debug-dumps/`. It measures instead of guessing.

`/gens pause` stops all generation instantly if you need breathing room while you look.

### What is the difference between a collector and an infinite hopper?

A collector owns a chunk. It takes every generator drop in that chunk before the drop becomes an
item entity, stores unlimited types, has a sell all button, sells its contents when broken, and
keeps a sale log.

An infinite hopper owns a spot. It scans a small box above itself and absorbs whatever lands
there, including drops that arrived on a water stream. It locks in a fixed number of item types,
sells only when a sellwand is swung at it, and has no sale log.

The collector is the tool for absorbing a whole farm. The hopper is the tool for specialising on
a few valuable drops, and it keeps the visible sucked up animation players like.

### My hopper stopped absorbing a new item type.

That is the lock-in working as designed. The first `hopper.max-types` types to arrive claim the
slots permanently, and anything else falls through. Raise `max-types` in `storages.yml` for new
hoppers, and remember to add matching entries to `type-slots` in `gui/hopper_main.yml` so the
extra types are actually drawn.

With `void-excess: true` the rejected items are deleted instead of dropping on the floor.

### A player was kicked from their island and lost all their generators.

They are not lost. With `island-pickup: true` the generators are moved to that player's vault,
and the player claims them with `/gens recover`. They are told on their next join.

The same happens when a player leaves, is banned, or the island is disbanded. Set
`island-pickup: false` if you want removal to be permanent instead.

### `/gens recover` says some generators are still stored.

The command only gives back what fits in the inventory, and reports the remainder. The player
frees space and runs it again. `/gens recover all` claims everything and drops the overflow on
the ground, so it asks for confirmation first: run it twice.

### Can I stop players placing generators in certain worlds?

Yes. `blacklisted-worlds` in `config.yml` denies placement, and existing generators in those
worlds stop ticking.

To restrict placement to islands, set `island-place-only: true`. That one needs
SuperiorSkyblock2, and does nothing without it.

### Another plugin owns `/sell` on my server.

SnGens registers `/sell` over any existing mapping, on purpose, so its own sell pipeline always
answers. If you need the other plugin's `/sell`, rename one of the two commands. There is no
setting that disables the SnGens one.

### Why is `/gens top` empty or out of date?

The board is a snapshot recomputed every `gens-top.refresh-interval-seconds`, five minutes out of
the box. Run `/gens forcetop` to recompute it immediately.

If it is empty on a server with generators, check `gens-top.mode`. `ISLAND` needs
SuperiorSkyblock2, and it skips players who have no island unless `include-islandless: true`. On
a server without a skyblock plugin, use `mode: PLAYER`.

### My top players are so far ahead that nobody competes any more.

Set a cutoff. `/gens toplimit diamond_generator` makes every tier after diamond stop counting,
so the maxed players stop growing their score and the rest can catch up. `/gens toplimit none`
clears it. The value is saved into `config.yml` for you.

### Players see `???` instead of my scoreboard numbers.

That is `/gens hidestats`, and it is on by default for the placeholders that support it. The
`_hide` variants return the `hide-stats-placeholder` text while a player has hiding enabled.

Use `%sngens_currentplaced%` rather than `%sngens_currentplaced_hide%` where the value should
always be visible, for instance in the player's own menu.

### How do I hand out an unlimited wand?

Pass `-1` as the number of uses.

```
/gens sellwand Notch 2 -1
/gens upgradewand Notch free -1
```

The item then shows the `unlimited-placeholder` text from `wands.yml` instead of a number.

### A player already has a sellwand. What happens if I give them another?

Charges are folded into the wand they already hold, and both sides are told how many were added
and what the new total is. Sellwands merge when the multiplier matches, build wands when the
distance matches, and upgrade wands when the type and radius match. A wand with a different
multiplier stays a separate item.

### What is `/gens handicap` for?

It scales one player's payout, silently. The percentage is applied to the final money amount
after every other multiplier, accepts anything from `-100` to `1000`, and never shows up in what
the player sees: the multiplier in their sell message is unchanged.

Use it to slow down a single runaway account without announcing a nerf, or to compensate one
player after an incident. `/gens handicap list` shows everyone who currently has one.

### Can I turn corruption off?

Yes, `corruption.enabled: false` in `config.yml`. Generators already broken can still be
repaired.

To keep the system but protect specific generators, add their ids to
`corruption.blacklisted-generators`, or set `corrupted.enabled: false` on the generator itself in
`generators.yml`.

### Which changes need a restart, and which need only a reload?

`/gens reload` covers configuration, generators, wands, events, storages, gear, menus and
messages. Two things need a restart: the database connection block, since the pool is built at
startup, and anything already handed out as an item, since a wand or generator in an inventory
carries its own data.

### I added a key to config.yml and it vanished, or my file is missing new keys.

Keys are only ever added, never removed, so a vanishing key means a YAML error somewhere above
it. Check the console on boot.

If your file is missing keys a new version added, check `update-configs`. When it is `false` the
plugin only logs how many sections are missing instead of inserting them.

### My translated language file is missing keys after an update.

It is not, or at least not for long. Only the English file ships inside the jar, so the plugin
merges any missing key into your translated file from the English one and logs how many it
added. Translate the new entries when convenient. The plugin never leaves a message key
unresolved: anything missing falls back to English.

### Can two servers share one database?

No. SnGens caches generators and players in memory and writes them back periodically, so two
servers pointing at the same database would overwrite each other's data. Give each server its own
database.

### Can I move from SQLite to MySQL and keep everything?

Not through the plugin. Switching `mysql.enabled` to `true` starts against an empty schema,
which the plugin creates on first connect. Moving existing data means copying the eight
`sngens_*` tables across yourself, with the server stopped.

### Does the plugin expose a developer API?

Yes, fourteen events, a read-only query service and a sell multiplier extension point, all in
the plugin jar. See [Developer API](api.md). Server owners can switch every API event off with
`api-events.enabled: false`.
