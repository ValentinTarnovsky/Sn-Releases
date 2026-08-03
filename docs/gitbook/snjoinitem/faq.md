# FAQ

### Players join and the item is not there.

Nine times out of ten this is `delivery.give-delay-ticks`. Lobby, login/auth and
inventory-restore plugins repopulate the inventory a moment after a player joins, and they
overwrite anything given before they run. Raise the delay to `40` or `80` and try again.

`0` does not mean "inside the join event" - the item is never given there. It means the next
tick, which is as early as it is ever handed out, and that is exactly the timing those
plugins win.

If the delay is not it, check in this order:

1. `delivery.give-on-join`. It is the master switch for **all** automatic delivery, not just
   joining, so `false` also stops the respawn re-give and the restore sweep.
2. `delivery.worlds`. If the list is not empty, only players joining in a listed world are
   given the item (names are case-insensitive).
3. `snjoinitem.bypass`. A player who has it never receives the item. Watch for a LuckPerms
   wildcard `*` handing it to all your staff.
4. The opt-out list. `/joinitem disable` is permanent until you undo it - check
   `data.yml`, or `%snjoinitem_disabled%`.
5. A full inventory. Placement is skipped on purpose rather than destroying something, and
   the restore sweep retries once a slot frees. With `delivery.restore-interval-ticks: 0`
   there is no retry.
6. The console. A line reading `Join item disabled: ...` means `items.yml` is the problem,
   not delivery.

### A player has two copies of the item, and cannot get rid of them.

They should not be able to. Every give - join, respawn, `/joinitem give` and the restore
sweep - first removes every copy outside the target slot, and the sweep re-places whenever a
stray exists anywhere, not only when the slot is empty.

To clean one player up right now, run `/joinitem give <player>`: it sweeps the strays and
re-places the single copy in one go.

Then find out why it happened, because copies of a locked item that nobody can drop pile up
until the inventory is unusable:

- **`delivery.restore-interval-ticks: 0`.** Nothing sweeps between joins, so a stray survives
  the whole session.
- **`locked: false` in `items.yml` while the sweep is running.** That is a working item
  duplicator: the player stores a copy, the sweep mints another. The plugin warns about this
  combination on boot.
- **A material that can leave the inventory by a route the lock does not cover.** Two classes
  to avoid: things the player spends (food, potions, buckets, ender pearls, snowballs, eggs,
  fireworks, discs, books) and things an entity takes out of your hand (wheat and seeds for
  breeding, bone for taming, name tags, dye, leads, saddles). Item frames and armour stands
  are blocked by the plugin; the rest are not. The default `COMPASS` is inert on purpose.

### How do I take the item off one player?

`/joinitem disable <player>`. It is the only thing in the plugin that can, because the item is
locked and the player cannot remove it themselves. The player has to be online.

It removes every copy they hold, including the one on their cursor, and it does that even if
they were already disabled - so it is also the recovery path for a player left holding a copy
they should not have.

To give it back: `/joinitem enable <player>`.

### How do I stop one player receiving it permanently?

The same `/joinitem disable <player>`. The opt-out is stored per UUID in
`plugins/SnJoinItem/data.yml` and survives restarts and updates: that file is never seeded or
merged from the jar, so an update cannot hand the item back to everyone you excluded.

It is read at the single point every automatic give consults - join, respawn and the restore
sweep. `/joinitem give` deliberately ignores it, so an admin can still hand the item to an
opted-out player on purpose.

### Can staff be exempt?

Give them `snjoinitem.bypass` (default `false`). Read what it does carefully:

| | `snjoinitem.bypass` | `/joinitem disable <player>` |
|---|---|---|
| Stops them receiving the item | Yes | Yes |
| Removes a copy they already hold | **No** | Yes |
| Exempts from the lock | **No** | Not applicable |
| Applies to | anyone with the node | one player, by UUID |

Two consequences worth knowing:

- **Nobody is exempt from the lock.** It is enforced with no per-player permission awareness,
  so a staff member who is holding the item cannot move or drop it either. Take it off them
  with `/joinitem disable`.
- **Granting bypass to someone who already holds the item changes nothing for them.** They
  keep it, because the sweep now skips them entirely and there is no path that removes it.
  Run `/joinitem disable <player>` after granting the node.

{% hint style="warning" %}
`snjoinitem.bypass` defaults to `false` on purpose. If your staff group has a wildcard `*`,
they hold it whether you meant it or not, and they will quietly stop getting the join item.
{% endhint %}

### Clicking the item does nothing.

Work down this list:

1. **Only four clicks reach the actions**: left, right, shift-left and shift-right. Middle
   click, the drop key, number keys, off-hand swap and double-click never fire, on either
   surface, and say nothing in the console.
2. **`cooldown` is in ticks**, not seconds or milliseconds. `20` is one second. A value copied
   from another plugin's millisecond field throttles the item into silence. `0` disables it.
   One cooldown covers both the world click and the inventory click.
3. **Only one click list declared.** A world click on the undeclared side does nothing at all.
   See "Can I have different left-click and right-click actions?" further down this page.
4. **`[click-air]` and `[click-block]` guards.** They are world-position guards, so a line
   carrying one is skipped silently on an inventory click.
5. **`interact-requirements` failing.** When they fail, `deny-actions` run instead - and if you
   never wrote a `deny-actions` list, nothing runs.
6. **The command itself.** `[player] ...` runs as the player, so the player needs permission
   for it. Use `[console] ...` for anything they cannot run themselves.
7. **The console.** If the item is disabled because `items.yml` is broken, no click path works.

### My `%player%` prints literally.

`%player%` is bound only for a click made **inside an open inventory**. A click made out in the
world runs through SnLib, which builds its own context and does not bind it, so it dispatches
as the literal text.

If the action has to work from both surfaces, use `%player_name%` and install PlaceholderAPI.
That is also what the item's `display-name` and `lore` need for any placeholder beyond `&`
colours and `&#RRGGBB` hex.

### I changed the material in `items.yml` and everyone online still has the old one.

Run `/joinitem reload`.

The item is identified by an invisible tag, not by its material or name, so a copy minted
before your edit still counts as the join item: the restore sweep sees a valid item in the
right slot and never replaces it. `/joinitem reload` is the only thing that rebuilds the copies
already in players' hands, and it does that for everyone online - bypass holders and opted-out
holders included - while delivering the item to nobody.

This is not cosmetic. A stale copy no longer matches the material the plugin cached at boot, so
its inventory-click actions go dead and it drops out of the item-frame guard, while the lock
keeps working and hides the fact.

### I set a block as the material and the item vanished entirely.

`material` must be a real **item**. `AIR` and block-only materials - `POTATOES` the crop rather
than `POTATO` the item, `WATER`, `FIRE`, any `*_WALL_SIGN` - look valid but have no item form,
so the plugin refuses to use them and switches the item off completely, restore sweep included.
The console names the offending material.

Fix `items.yml` and run `/joinitem reload`. The reload answers either way: it repeats the reason
if the item is still broken, or logs `Join item re-enabled (MATERIAL)` when it is fixed. A boot
warning is printed once, so the reload is where you get your confirmation.

Do not put a PlaceholderAPI token in `material` either. Placeholders resolve per player, while
the plugin caches one material at boot for its hot paths.

### Does it work alongside auth and lobby plugins?

Yes, and `delivery.give-delay-ticks` exists precisely for them - its default of `20` already
assumes something else is touching the inventory. If a login plugin freezes or restores the
inventory, raise the delay past the point where it finishes.

`delivery.restore-interval-ticks` covers the rest: anything that clears an inventory later
(Essentials `/ci`, `/clear`, a kit plugin) is undone on the next sweep, because the lock stops
players, not plugins.

One thing it does not do: `delivery.worlds` gates **giving** only. A player who receives the
item in your lobby and then walks into a survival world keeps it.

### Can I have different left-click and right-click actions?

Yes. `right-click-actions` and `left-click-actions` are separate lists, and you can add
`shift-right-click-actions` and `shift-left-click-actions`, which replace the base list when
the click is shifted. Both surfaces honour all four.

{% hint style="danger" %}
The two sides never cross. Declare only `right-click-actions` and a left click does nothing at
all, in the world and inside an inventory alike, with nothing logged. Declare **both** base lists
if both clicks have to act.
{% endhint %}

### The slot I picked already has something in it.

The occupant is moved to the first free slot, never overwritten. If there is no free slot the
plugin does nothing at all that tick and the restore sweep tries again once one frees, so a
player with a full inventory joins without the item rather than losing an item to it.

`placement.slot` is a full-inventory index: `0-8` hotbar, `9-35` main inventory, `36-39` armour,
`40` off-hand. Anything outside `0-40` falls back to `4` with a console warning.

### How do I stop updates touching my config files?

Set `update-configs: false` in `config.yml`. SnLib then only warns in the console about keys it
would have added instead of inserting them, and your files stay byte-for-byte as you left them.
Any key that is missing falls back to the default documented in the file.

You do not need it to protect player data: `data.yml` is mounted plain and is never seeded,
merged or regenerated, whatever this setting says.

### The plugin will not start.

Both causes print a clear console line:

- **No SnLib.** `SnLib.jar` must be in `plugins/`. SnJoinItem declares `depend: [SnLib]` and
  will not load without it.
- **License.** The key goes in the shared `plugins/.Sn-License/license.yml`, one file for every
  bundle plugin on that server. Without a valid key the plugin disables itself at startup.

See [Installation](installation.md) for the full sequence.

### Is there a `/joinitem update`?

No. `reload` and `help` are the only subcommands added on top of `give`, `disable` and `enable`.
`snjoinitem.admin.update` is notify-only: it decides who sees the update notice on join, and it
is not a command. The full list is on the [Commands](commands.md) page.

### What exactly do the placeholders report?

`%snjoinitem_enabled%` and `%snjoinitem_disabled%` return the literal strings `true` and
`false`, resolved by UUID so they answer for offline players too.

They reflect the **opt-out flag only**. A player who is bypassed, standing in a world outside
`delivery.worlds`, or on a server with `give-on-join: false` still reads `enabled = true`,
because none of those is an opt-out.
