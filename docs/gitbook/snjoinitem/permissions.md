# Permissions

| Node | Default | Grants |
|---|---|---|
| `snjoinitem.admin` | op | Parent node - grants every admin node below |
| `snjoinitem.admin.give` | op | `/joinitem give [player]` |
| `snjoinitem.admin.toggle` | op | `/joinitem disable <player>` **and** `/joinitem enable <player>` |
| `snjoinitem.admin.reload` | op | `/joinitem reload` |
| `snjoinitem.admin.update` | op | Receive the update notice on join. Not a command. |
| `snjoinitem.bypass` | false | The player does not **receive** the join item |

Granting `snjoinitem.admin` unlocks the four admin nodes at once: the children list is
exhaustive, there is nothing else under it.

`/joinitem` and `/joinitem help` carry no node at all, so anyone can run them. The help
list and tab completion are filtered per sender, so a player without any admin node sees
the header and nothing under it.

There is no `snjoinitem.use`. Receiving the item is not gated on a permission a player has
to hold; it is gated on `snjoinitem.bypass` being **absent**.

## `snjoinitem.bypass` means "does not receive", not "not locked"

This node is read at the single eligibility check that every **automatic** delivery
consults: the join give, the respawn re-give, and the restore sweep. A player who has it is
skipped by all three, so on a normal server they simply never see the item.

It does nothing else. In particular it does not exempt anyone from the lock:

- The lock is enforced by SnLib across its extraction vectors (click, drag, manual equip,
  hand swap, drop, death drops, hopper transfer) and reads no permissions at all.
- Creative-mode clicks are covered by that same click denial, because Bukkit routes them
  through the ordinary inventory-click event.
- What SnJoinItem adds on top - blocking the item being handed to an item frame or an armour
  stand, and the client resync that has to follow a cancelled creative-mode edit - is
  deliberately not gated on `snjoinitem.bypass` either. Honouring it in one place and not the
  other would make the item movable in creative but pinned in survival for the same player.
- The click actions run for anyone holding the item, bypass included.

So the lock is uniform for everyone actually holding a copy, staff included.

A bypass holder can end up holding one in two ways: an admin ran `/joinitem give` on them
(that command ignores **every** eligibility filter, bypass among them), or they were granted
bypass after they already had the item, since nothing in the plugin takes a copy away on its
own.

**To take the item off someone, use `/joinitem disable <player>`.** That is the only path
in the plugin that removes a held copy, and it removes unconditionally - even from a player
who was already opted out - precisely so a locked copy can never become permanent.

{% hint style="warning" %}
A bypass holder who was handed the item by `/joinitem give` keeps it, but the restore sweep
will not maintain it: they fail the eligibility check, so an external `/clear` takes it away
for good and nothing re-places it.
{% endhint %}

## `snjoinitem.admin.update` is a notice, not a command

There is **no `/joinitem update` subcommand**. SnLib injects `reload` and `help` into every
command root, but not `update`, so nothing on the `/joinitem` tree reads this node.

All it decides is who is told in chat that a newer SnJoinItem release exists on the
`Sn-Releases` feed: on join, and on the spot for anyone already online when a new release is
detected. Granting it adds no ability and revoking it removes none: it only turns that one
message on or off for that group.

Silence the notice for a group that does not maintain the server:

```
/lp group mod permission set snjoinitem.admin.update false
```

## A wildcard `*` grant stops staff receiving the item

`snjoinitem.bypass` is registered in `plugin.yml`, so a wildcard grant includes it. Any
group holding `*` therefore holds bypass, and every member of that group is skipped by
delivery on join, on respawn and by the restore sweep. The usual report is "the item works
for normal players but nobody on staff gets one", and this is almost always the cause.

The node defaults to `false` on purpose so that op status alone never grants it, but a
wildcard outranks that default. Negate it explicitly on the affected group:

```
/lp group admin permission set snjoinitem.bypass false
```

{% hint style="info" %}
To hand yourself the item for a quick test without touching permissions, run
`/joinitem give`. It ignores bypass, the opt-out list, `delivery.worlds` and
`delivery.give-on-join`.
{% endhint %}

## Useful grants

**A moderator who can opt players in and out but cannot reload the plugin:**

```
/lp group mod permission set snjoinitem.admin.toggle true
```

**A helper who can only hand the item out again after a clear:**

```
/lp group helper permission set snjoinitem.admin.give true
```

**A lobby-only rank that should never carry the item, without opting each player out:**

```
/lp group builder permission set snjoinitem.bypass true
```
