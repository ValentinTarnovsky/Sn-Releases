# Permissions

| Node | Default | Grants |
|---|---|---|
| `sncrates.use` | **true** | `/crates` and every user-open alias: your own key balance menu |
| `sncrates.admin` | op | Parent node - grants every admin node below |
| `sncrates.admin.reload` | op | `/crates reload` |
| `sncrates.admin.debug` | op | `/crates debug` |
| `sncrates.admin.update` | op | Receive the update notice on join. Not a command. |
| `sncrates.admin.editor` | op | `/crates editor` and every mutation inside it |
| `sncrates.admin.keys` | op | `/crates key give`, `giveall`, `take` and `set` |
| `sncrates.admin.wipekeys` | op | `/crates key wipe` - **required on top of** `sncrates.admin.keys` |
| `sncrates.admin.givekey` | op | `/crates givekey` |
| `sncrates.admin.open` | op | `/crates open`, for yourself or another player |
| `sncrates.admin.preview` | op | `/crates preview` |
| `sncrates.admin.balance` | op | `/crates balance <player>` |
| `sncrates.open.<crateId>` | false | Opens that one crate, on crates that accept the `PERMISSION` key type |

Granting `sncrates.admin` unlocks the ten admin nodes at once: the children list is exhaustive,
there is nothing else under it.

## `sncrates.admin.wipekeys` stacks, it does not replace

`/crates key wipe` sits inside the `key` group, and SnCrates checks **every** node on the path from
the root down to the leaf. So running a wipe needs `sncrates.admin.keys` *and*
`sncrates.admin.wipekeys`; the second narrows the first rather than standing in for it.

That is what lets you delegate the ordinary key commands without delegating the one that cannot be
undone. A moderator holding only `sncrates.admin.keys` can give, take and set balances all day and
gets `You do not have permission to use this command.` from `wipe`.

## Opening a crate needs no permission

Right-clicking a bound crate block, clicking a crate in `/crates` and pressing Q to mass-open all
require nothing beyond `sncrates.use` (and, for the menu, nothing more than that). Opening a crate
has never required a node, and adding one would silently take crates away from every player on
every server that updated.

What gates an open is the **key**: the player has to hold a physical key, have a virtual balance, or
hold the crate's permission node - whichever key types that crate accepts.

## `sncrates.open.<crateId>` is per crate and is not declared

The `PERMISSION` key type turns a permission node into a key. A crate that lists it checks:

- the crate's own `open-permission` value, when it declares one, or
- `sncrates.open.<crateId>` when it does not.

Neither can be declared in `plugin.yml` - the space is one node per crate and is only known once
your crates are loaded - so they will not show up in a permissions plugin's auto-complete. Grant
them by hand:

```
/lp group vip permission set sncrates.open.vipcrate true
```

Set `open-permission` on the crate (from the editor's **Open Permission** button, or in the crate
file) to check a node you already use instead:

```yaml
vipcrate:
  accepted-key-types: [PERMISSION]
  open-permission: "myserver.rank.vip"
```

{% hint style="info" %}
`open-permission` is checked **only** by a crate that accepts `PERMISSION`. On a crate that accepts
only `PHYSICAL` or `VIRTUAL` the value is stored and ignored - it is not a second gate on top of
the key.
{% endhint %}

A `PERMISSION` crate consumes nothing when it opens: the node is the key, and a node cannot be
spent. Limits are what stop a rank opening the same crate forever, not the key.

## Key protections read no permission at all

A physical key cannot be placed as a block, put in an item frame, put on an armour stand or eaten.
Those four are enforced for **everyone**, operators and wildcard holders included, and there is no
node that exempts anybody.

That is not an oversight. A `sncrates.key.bypass` node existed for two releases and did exactly what
a bypass does: the people most likely to be handling keys are staff, so the exemption destroyed the
keys of precisely the players who held it - a candle-material key placed as a block, a key put in an
item frame, an edible-material key eaten. It was removed and will not come back.

The same applies to the extra restrictions `keys.restrict-usage` adds (crafting and processing
stations, and a plain right-click being inert). They are a config switch that applies to the whole
server, not a permission.

## `sncrates.admin.update` is a notice, not a command

There is **no `/crates update` subcommand**. SnLib injects `reload`, `help` and `debug` into the
command root, but not `update`, so nothing on the `/crates` tree reads this node.

All it decides is who is told in chat that a newer SnCrates release exists on the `Sn-Releases`
feed: on join, and on the spot for anyone already online when a new release is detected.

Silence the notice for a group that does not maintain the server:

```
/lp group mod permission set sncrates.admin.update false
```

## The editor node is re-checked, not just at the door

`sncrates.admin.editor` opens the editor, and it is checked again at every mutation the editor can
perform - including the ones that resolve after the menu is gone:

- a chat prompt whose answer arrives seconds later,
- an armed block bind or unbind that resolves when the admin clicks a block in the world.

An admin whose node is revoked mid-session is refused on all of them, and a refusal also disarms a
pending block gesture. Revoking the node is enough; you do not have to get them out of the menu.

## Useful grants

**A moderator who can hand out keys but cannot edit crates or wipe the economy:**

```
/lp group mod permission set sncrates.admin.keys true
/lp group mod permission set sncrates.admin.givekey true
```

`sncrates.admin.wipekeys` is deliberately absent - that is the point of it being a separate node.

**An owner who wants the season-reset command, on top of the moderator grant above:**

```
/lp group owner permission set sncrates.admin.keys true
/lp group owner permission set sncrates.admin.wipekeys true
```

**A helper who can look at what a player is holding, and nothing else:**

```
/lp group helper permission set sncrates.admin.balance true
```

**A builder who can bind crate blocks but should not be able to delete a crate** - not expressible.
The editor is one node: `sncrates.admin.editor` grants block binding and crate deletion together.
Grant it only to people you would trust with `/crates editor` as a whole.

**A rank that opens one crate without ever holding a key:**

```yaml
# crates/vipcrate.yml
vipcrate:
  accepted-key-types: [PERMISSION]
```

```
/lp group vip permission set sncrates.open.vipcrate true
```

{% hint style="warning" %}
A wildcard `*` grant includes every node declared in `plugin.yml`, so any group holding it holds the
whole admin surface. It does **not** include `sncrates.open.<crateId>`, which is not declared
anywhere - so a wildcard admin still cannot open a `PERMISSION` crate unless you grant that node
explicitly.
{% endhint %}
