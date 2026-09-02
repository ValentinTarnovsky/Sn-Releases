# FAQ

### The sneak-redeem does nothing.

Check three things, in this order:

1. `keys.allow-redeem` is `true` in `config.yml`.
2. The crate accepts `VIRTUAL`. A crate that only accepts `PHYSICAL` has no balance for the keys to
   become, so the redeem is refused by design.
3. The crate still exists. A key whose crate was deleted is left alone on purpose - deleting or
   converting it would destroy something a player paid for, over a crate you may well recreate under
   the same id.

The gesture is **sneak + right-click while not looking at a crate block**, and clicking into empty
air is the intended form. If it works when you happen to be aiming at a block and not when you aim
at the sky, that is a different bug and worth reporting.

### A player opened a crate and got nothing.

The reward is decided before the first animation frame renders and delivered exactly once at settle,
so "nothing" is almost always one of these:

- The reward had `give-item: false`. It is command-only: the item shown is the icon, the player gets
  whatever the commands grant.
- The player had **deactivated** that reward with the preview filter. It can still be rolled and
  still burns its limits; it is simply never handed to them.
- The reward's commands failed. A failing command is logged in the console and the rest still run.
- Their inventory was full at the moment of delivery.
- The crate has a [fail chance](configuration.md#fail) and it came up. The player was told so
  (`messages.open.fail`), the spin landed on the "nothing" item, and `opening.log` says `failed`
  where the reward's name would go.

`opening.log` records one line per open with the reward's display name, which answers the question
directly.

### Every reward in a crate is on limit. What happens?

The player is told `Every reward in this crate is on cooldown or limit - nothing to win right now.`
and **keeps their key**. The winner is picked before the key is consumed, so a crate with nothing
winnable refuses the open rather than charging for it.

### I deleted a crate and its blocks are unbreakable.

They should break. Deleting a crate unbinds every block bound to it, and a block still bound to a
crate that no longer exists releases itself the next time somebody tries to break it - including
after a restart.

If a block really will not break, check that it is not bound to a **different** crate that still
exists.

Note that explosions behave differently on purpose: a blast next to an orphaned crate block leaves
it standing. A blast is nobody asking to remove a crate block, so it cannot clear a binding as a
side effect. Swing at the block instead.

### Can a crate have no key at all?

Yes, three ways:

- `accepted-key-types: [PERMISSION]` and a node. Nothing is consumed when it opens - a node cannot
  be spent - so limits are what stop a rank opening it forever.
- `accepted-key-types: [VIRTUAL]` with balances topped up by your vote listener or webstore through
  `/crates key give`.
- Remove the `key-item` block entirely from a crate that accepts `VIRTUAL` or `PERMISSION`.

`PERMISSION` is written in the crate file (or in `defaults.accepted-key-types`), not chosen from the
editor. The **Accepted Keys** button cycles `PHYSICAL`, `VIRTUAL` and both, and clicking it on a
rank-gated crate restarts that cycle at `PHYSICAL` - so once a crate is on `PERMISSION`, edit
everything else about it freely and leave that one button alone.

### How do I clear everyone's keys for a season reset?

`/crates key wipe` - it deletes the virtual key balances of every player, offline players included,
and `/crates key wipe <crate>` narrows it to one crate. It cannot be undone, so the first
invocation only warns and counts; run it again with the confirmation word to go through with it.
See [the key commands](commands.md#crates-key-wipe-crate-confirm).

It needs `sncrates.admin.wipekeys` on top of `sncrates.admin.keys`, so it is not something a
moderator can reach by default.

Two things it does **not** do. It does not touch **physical** keys - those are items in inventories
and ender chests, and no balance command reaches them; if your crates use a key item, the wipe is
not what resets them. And it does not reset open counters, win history or reward limits: those are
separate records, and `%sncrates_total_opens%` keeps counting from where it was.

### How do I make a crate players open from a menu, with no block anywhere?

Give it `VIRTUAL` in `accepted-key-types` and hand out balances with `/crates key give`. It appears
in `/crates` for anyone with a balance, and they open it from there. Never bind a block.

### Why is my crate missing from `/crates`?

Because it accepts **only** `PHYSICAL`. That crate has no virtual balance, so a menu showing it
would show a permanent `0` and clicking would do nothing. It is skipped before a slot is consumed,
so the layout has no hole either.

Add `VIRTUAL` to `accepted-key-types` if you want it listed.

### The withdraw button is not there.

It only appears on a crate that accepts **both** `VIRTUAL` and `PHYSICAL`. On a virtual-only crate
the withdrawn key would be unusable, and on a physical-only crate there is no balance to withdraw
from. Check `withdraw.enabled` too.

### `%sncrates_keys_mycrate%` prints literally.

The raw text means the id does not exist. Check the spelling against the **top-level key of the
crate file**, not the display name.

If it prints `0` instead, the id is right and either the balance really is zero or the player's data
is still loading (a second after join). See [Placeholders](placeholders.md).

### `%player_name%` in a reward's name prints literally.

Item text inside `crates/*.yml` resolves placeholders **once, when crates load**, against no
particular player. Only `%server_name%`-style placeholders work there.

Menu lore under `guis/` is different: that resolves per viewer, every time the item is built. Put
per-player text in the menu layout.

### My win command runs but the target plugin says "invalid number".

Almost always a placeholder that did not resolve. SnCrates substitutes `{player}`, `{amount}` and
`{crate}` first and then hands the whole line to PlaceholderAPI, so nested placeholders like
`%math_5*{edtools_leveling_level_level}/100%` work - but only with PlaceholderAPI installed and the
relevant expansion downloaded.

Also check you did not put colour codes in the command. Commands are never colour-translated, and a
`&a` in one corrupts the text.

### A player has 4,000 keys and mass-opening froze the server.

On 2.3.0 and later it should not: a mass open is spread over ticks, `access.mass-open-per-tick`
crates per tick (shipped `10`), so 4,000 keys take about twenty seconds of ordinary ticks rather
than one enormous one. If the server still stutters, lower `mass-open-per-tick`; a reward whose
commands are expensive is the usual reason.

Before 2.3.0, `access.mass-open-max` at `-1` or `0` opened the whole balance in one tick. `64` is
still the shipped cap, because a summary of 4,000 opens is not something a player wants to read
either.

### How do I make some opens fail?

Set `fail.chance` in `config.yml` to a percentage, or give one crate its own with
`settings.fail-chance` in its file or the **Fail Chance** button on its panel in the editor. A
failed open spends the key, runs the animation and lands on `fail.item` with `effects.fail-sound`,
tells the player, counts as an open and writes `failed` in `opening.log` - and delivers nothing,
burns no limit and fires no `CrateRewardEvent`. The preview's info icon says how often the crate
fails, and the mass-open summary counts the fails. See [Fail](configuration.md#fail).

A crate whose every reward is on limit still refuses the open with the key unspent, whatever its
fail chance: the fail is only rolled once something is winnable.

### Can players lose a reward by closing the animation?

No. The winner is computed before the first frame, and the delivery happens once, at a single
choke-point, guarded so it cannot run twice. Closing the menu, disconnecting, being kicked, a
`/crates reload` and a server stop mid-spin all deliver the reward exactly once and consume exactly
one key.

### An admin can take the reward item out of the editor's capture window.

They cannot. That window has no writable cell: it copies the item from your **main hand** and every
click in it is cancelled, like every other menu in the plugin. Hold the item, click the capture cell,
and you keep what you were holding.

If you remember dragging an item into a slot and closing the window, that was the old gesture. It is
gone.

### I captured a plain key over an enchanted one and it came back enchanted after a restart.

Known issue. Capturing a new key item does not clear the appearance keys the previous key had
written in the crate file - enchantments, colour, trim, potion effects, unbreakable, skull owner and
about a dozen more - so they re-apply on the next load.

Workaround: open `crates/<id>.yml` and delete the leftover keys under `key-item` by hand.

### I edited a crate in game and my comments are gone.

The editor rewrites a crate's whole entry when it saves, so comments **inside a crate you edit in
game** do not survive. Your values do, and so does the inherit/override/off shape of every settings
and effects key.

Comments in `config.yml`, `guis/*.yml` and the language file are never touched.

### I added a key to `config.yml` and it disappeared.

It should not have - SnLib keeps extra keys. Check `update-configs` is still `true` and look for a
pre-merge backup in the plugin folder. If you deleted an entry from a section marked
`# sn:extensible`, that deletion is honoured and the entry stays gone by design.

### `/llaves` does not exist after I added it.

`command.user-open-aliases` entries register their own top-level command while the server **starts**.
A `/crates reload` cannot register a command with Bukkit, so this one setting needs a restart.

`command.aliases` is the one that reloads.

### Nobody on staff can open crates.

Opening a crate needs no permission at all, so this is never a permissions problem on the open path.
Check whether the crate accepts `PERMISSION` and staff are missing `sncrates.open.<crateId>` - that
node is not declared in `plugin.yml` (it is one node per crate), so a wildcard `*` grant does **not**
include it.

### Can I exempt staff from the key protections?

No, and this is deliberate. There is no bypass node. A `sncrates.key.bypass` existed for two releases
and destroyed the keys of exactly the people who held it: staff handle keys the most, and the
exemption let them place a candle key as a block, frame it, or eat an edible one.

`keys.restrict-usage` is a server-wide switch, not a permission.

### The completion particle does not appear on 1.20.

`HAPPY_VILLAGER` is the 1.20.5+ spelling. On 1.20.1 to 1.20.4 the constant is `VILLAGER_HAPPY`. Put
that in `effects.complete-particle` instead; the console logs one warning per load telling you the
name did not resolve.

### The console says `complete-particle-count` is invalid, and prints the same number twice.

```
Invalid value in config.yml -> 'effects.complete-particle-count': received '40', using default '40'
```

Nothing was wrong with your value, and it was fixed in **2.1.0**. The crate editor's panel read that
key as text when it holds a number, once per control per render, so opening a crate panel printed the
line every time. Update, and it stops. On any version, the count itself was always being used.

### My server ran an older SnCrates and everything is empty.

Expected. SnCrates 2.0.0 is a clean break: it reads no file, no key and no database row written by
an older version. Key balances, statistics, history, limits, filters and every block binding start
empty, and previously placed crate blocks are breakable again.

There is no importer and no migration path. Re-bind your crate blocks and re-issue balances.

{% hint style="warning" %}
This applies to any pre-release 2.0.0 build too, not only to 1.x. A staging server that ran an early
2.0.0 also starts from empty.
{% endhint %}

### Is there a `/crates update`?

No. `sncrates.admin.update` is a notify-only node: it decides who sees the update notice on join, not
who can run something.

### The plugin will not start.

Three causes, in order of likelihood: `SnLib.jar` is missing or too old, the license key is missing
or invalid, or the server is on Java 17. Each prints a distinct line in the console - see
[Installation](installation.md).

### How do I stop updates touching my config files?

Set `update-configs: false`. SnLib then only warns about keys it would have inserted. Crate files are
never merged either way.
