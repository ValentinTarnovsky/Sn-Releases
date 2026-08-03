# Commands

Root command: `/joinitem`, alias `/jit`.

| Command | Description | Permission |
|---|---|---|
| `/joinitem` | Shows the help page | none |
| `/joinitem give [player]` | Give the join item, ignoring every filter | `snjoinitem.admin.give` |
| `/joinitem disable <player>` | Stop giving the item to a player and take back every copy they hold | `snjoinitem.admin.toggle` |
| `/joinitem enable <player>` | Clear that opt-out and hand the item straight back | `snjoinitem.admin.toggle` |
| `/joinitem reload` | Reload the configuration, the item and the language file | `snjoinitem.admin.reload` |
| `/joinitem help [page]` | Paginated help, filtered by what you can run | none |

The root command itself carries no permission, so a bare `/joinitem` renders help for anyone.
Help and tab completion only list the subcommands the sender is actually allowed to run, so a
regular player sees `/joinitem help` and nothing else.

Extra aliases are read from `command.aliases` in `config.yml` (shipped as `[jit]`) and are
re-read on `/joinitem reload`, so an alias you add works without a restart. `jit` itself is also
declared in the plugin's `plugin.yml`, where Bukkit owns it, so emptying that list does not
unregister `/jit`.

{% hint style="info" %}
There is no `/joinitem update`. `snjoinitem.admin.update` is a notify-only node: it decides who
sees the update notice on join, not who can run something.
{% endhint %}

Every `<player>` argument is an **online, exact** player name. An offline or misspelled name is
rejected with `Player not found`, so there is no way to opt an offline player in or out from the
command.

## `/joinitem give [player]`

Places the join item in the configured slot, ignoring **every** eligibility rule the plugin has:

- the opt-out list (`/joinitem disable`),
- the `snjoinitem.bypass` permission,
- the `delivery.worlds` filter,
- `delivery.give-on-join`, even when it is `false`.

With no argument it gives the item to yourself. From the console the player argument is required,
and omitting it answers `Please specify a player from the console.`

Placement follows the normal rules: any stray copy elsewhere in the inventory is swept first,
whoever occupies `placement.slot` is moved to a free slot rather than overwritten, and a full
inventory is skipped. When the item could not be placed you get a failure line instead of a
success line, which also covers a misconfigured `items.yml` (the reason is printed once in the
console).

{% hint style="warning" %}
`give` does not clear an opt-out. A player you gave the item to while they were disabled keeps
that copy, and because the item is locked they cannot get rid of it. Use `/joinitem disable` to
take it back.
{% endhint %}

## `/joinitem disable <player>`

Two things happen, in this order:

1. The player is added to the opt-out list, stored per UUID in `data.yml`, so they stop receiving
   the item on join, on respawn and from the restore sweep. It survives restarts.
2. Every copy they are currently holding is removed, the item on their cursor included.

The removal is unconditional. Running the command on someone who was **already** disabled still
strips the copies they hold and answers `... already had the join item disabled. Removed any copy
they were still holding.` That is the point: `/joinitem give` deliberately ignores the opt-out, so
a disabled player can be holding the item, and this is the only path that can take it off them.

{% hint style="info" %}
This is the only way to remove the item from a player at all. The item is locked, so they cannot
drop, move or store it themselves, and the restore sweep puts back anything an external `/clear`
takes away.
{% endhint %}

The **target** is told `Your join item was removed by an administrator.`, but only when a copy was
actually taken (a bypassed player holding nothing is not messaged about an item that never
existed), and never when an admin ran the command on themselves.

## `/joinitem enable <player>`

Clears the opt-out and immediately gives the item back, so the effect is visible without waiting
for the next join.

- If the player was **not** opted out, you are told so and nothing else happens. `enable` never
  delivers the item to a player who was already enabled; use `/joinitem give` for that.
- If their inventory is full, the opt-out is still cleared and you are told the item will be given
  automatically once a slot frees. That is a partial success, not a failure: the restore sweep
  finishes the job.
- If `items.yml` is broken, nothing can be delivered and you get the give failure line pointing at
  the console instead.

The **target** is told `You have received the join item again.` only when the item actually landed
in its slot, and never when an admin ran the command on themselves.

## `/joinitem reload`

Re-reads `config.yml`, `items.yml`, `lang/messages_en.yml` and `data.yml` from disk, restarts the
restore sweep with the new interval, and re-reads the command aliases.

It also rebuilds the item in the hands of everyone already online, so an appearance or action edit
in `items.yml` applies to copies players are already carrying, including copies held by bypassed
or opted-out players.

{% hint style="warning" %}
`reload` delivers the item to **nobody**. A player who currently has no copy gets one from join,
respawn or the restore sweep, not from a reload, so a reload never overrides
`delivery.restore-interval-ticks: 0`.
{% endhint %}

If the item is still unusable after the reload, the reason is repeated in the console, and a
repaired `items.yml` is confirmed there too.

## `/joinitem help [page]`

Same output as a bare `/joinitem`. It needs no permission and lists only the subcommands the
sender can use, one page at a time.
