# SnJoinItem

One configurable item, given on join and pinned to a slot the player cannot empty.

You describe the item once in `items.yml` - material, name, lore and what a click should run.
Every player who joins gets it, a short configurable delay after they land, in one fixed
inventory slot. From that point the item stays put: it cannot be dropped, moved, stored or
lost on death, and if something outside the plugin wipes it, a repeating sweep puts it back.

## What it does

- **One item, fully described in `items.yml`**: material, display name, lore, glow,
  unbreakable, tooltip flags and optional custom model data. Colour codes, HEX and
  PlaceholderAPI all resolve **per player**, every time the item is built.
- **Pinned to a fixed slot.** The slot is a full-inventory index, so it can be the hotbar,
  the main inventory, an armour slot or the off-hand.
- **It never eats a player's item.** Whatever occupied the slot is relocated to a free one.
  If there is no free slot, placement is skipped entirely and retried later.
- **Locked.** Clicking, dragging, equipping, hand-swapping, dropping, death drops and hopper
  transfer are all denied, in creative mode as well as survival. SnJoinItem additionally
  blocks handing the item to an item frame or an armour stand.
- **Exactly one copy, ever.** Stray copies anywhere else in the inventory (or on the cursor)
  are swept before the item is placed, and the sweep also runs on its own - locked duplicates
  would otherwise be un-removable by the player who ends up holding them.
- **Restored after an external clear.** The lock stops players, not plugins: `/clear`,
  Essentials `/ci` or a lobby plugin emptying the inventory all slip past it. A repeating
  check re-gives the item, on a period you set, or you can switch the check off.
- **Given again on respawn**, so death never costs the player the item.
- **Clicks run actions you configure.** Separate left-click and right-click lists, optional
  shifted variants, requirements and deny-actions. Actions run as the player or from the
  console, with a per-player cooldown measured in ticks.
- **Both click surfaces work**: clicking the item while playing, and clicking it inside an
  open inventory screen.
- **Restricted to the worlds you name**, or every world if you name none.
- **Per-player opt-out.** `/joinitem disable <player>` stops giving the item and takes back
  the copy they are holding - the only way to remove a locked item from someone.
  `/joinitem enable` reverses it. The list is stored by UUID and survives restarts.
- **`snjoinitem.bypass`** exempts a player from **receiving** the item, for staff who want
  their hotbar to themselves.
- **`/joinitem give [player]`** hands the item out by force, ignoring every filter.
- **Reload without a restart.** `/joinitem reload` re-reads your files and rebuilds the copy
  every online player is already holding, so an edit is visible immediately.
- **PlaceholderAPI output**: `%snjoinitem_enabled%` and `%snjoinitem_disabled%` report
  whether a player has opted out.

## Typical use

A lobby or hub. The shipped defaults are already this: a compass named **Menu** in the middle
of the hotbar, where both clicks run `/menu` as the player. Nobody can drop it, shove it into
a chest, hand it to an item frame or lose it in the void, and when your lobby plugin clears
inventories the compass is back a couple of seconds later. Staff who want a clean hotbar get
`snjoinitem.bypass`; the one player who really does not want it gets `/joinitem disable`.

The same shape covers a warp selector, a cosmetics book, a shop item or a vote reminder -
anything that has to be in a known slot, in every player's hand, and impossible to lose.

## Requirements

Java 21+, Paper 1.20.x or 1.21.x, and **SnLib** in `plugins/`. PlaceholderAPI is optional and
only needed for placeholders in the item, the messages or the click actions.

{% hint style="info" %}
SnJoinItem is part of the licensed bundle. Your key goes in the shared
`plugins/.Sn-License/license.yml` - see [Installation](installation.md).
{% endhint %}
