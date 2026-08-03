# Commands

Root command: `/reclaim`, alias `/rc`.

| Command | Description | Permission |
|---|---|---|
| `/reclaim` | Redeem every reclaim you have available | `snreclaim.use` |
| `/reclaim reset <player>` | Wipe a player's claims so they can claim again | `snreclaim.admin.reset` |
| `/reclaim enable` | Open redemption for everyone | `snreclaim.admin.manage` |
| `/reclaim disable` | Close redemption for everyone (season lock) | `snreclaim.admin.manage` |
| `/reclaim status` | Show whether redemption is open | `snreclaim.admin.status` |
| `/reclaim reload` | Reload config and language files | `snreclaim.admin.reload` |
| `/reclaim help` | Paginated help, filtered by what you can run | `snreclaim.use` |
| `/reclaim debug` | Toggle runtime debug output | `snreclaim.admin.debug` |

Help and tab completion only ever show a player the subcommands their permissions allow,
so a regular player sees `/reclaim` and nothing else.

## `/reclaim`

Redeems **every** qualifying unclaimed rank in one go. For each one, in order: the claim is
recorded, the rank's reward commands run, the player gets that rank's private message, and
the server gets a broadcast.

- Nothing available: the player is told so, and nothing runs.
- Redemption globally disabled: refused with a message.
- Already redeeming (a reward command that loops back into `/reclaim`): silently ignored,
  so nothing is granted twice.

## `/reclaim reset <player>`

Removes every claim that player owns, online or offline, so they can redeem from scratch.
The name completes from online players and also accepts anyone the server has seen before.

{% hint style="warning" %}
This wipes **all** of that player's claims, not one rank. There is no per-rank reset.
{% endhint %}

## `/reclaim enable | disable | status`

The season lock. `disable` closes redemption for everyone and also stops the join
notification advertising reclaims nobody can take. The state is written to `config.yml`, so
it survives a restart.

`status` is deliberately a **separate permission** from `enable`/`disable`, so you can let
staff check whether reclaims are open without giving them the power to close them.
