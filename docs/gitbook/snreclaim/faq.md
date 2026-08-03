# FAQ

### A player says they got nothing from `/reclaim`.

Check, in this order:

1. Are they actually in the group? `/lp user <name> info`. The rank's `group:` (or its key,
   if `group:` is unset) must match a group they are in.
2. Have they already claimed it? Claims are permanent. `/reclaim reset <player>` gives them
   another go - but it wipes **all** their claims, not one rank.
3. Is redemption open? `/reclaim status`.
4. Does the rank actually have commands? A rank with no usable commands is skipped at load
   and says so in the console.

### Nobody gets the join notification.

Almost always `join-notify.delay-ticks` being too short for your permissions plugin. Raise
it. It exists because group data is not ready the instant a player joins.

### The plugin refuses to start.

Three causes, all of which print a clear console line:

- **No Vault permission provider.** Vault is installed but no permissions plugin registered
  a group service. Install LuckPerms.
- **No SnLib.** `SnLib.jar` must be in `plugins/`.
- **License.** The key goes in the shared `plugins/.Sn-License/license.yml`.

### I updated from 1.x and the season lock came off by itself.

Known and documented - see [Migrating from 1.x](migration.md). Your old `settings:` block
is still on disk but is no longer read, so `redeem.enabled` started at its default of
`true`. The plugin prints a warning block naming every stale key at boot.

### Can I give one rank's reclaim back without wiping the rest?

No. `/reclaim reset` is all-or-nothing. If you need per-rank surgery, delete that row from
`snreclaim_claims` directly.

### Can a reward run as the player instead of the console?

Yes: `run-as: player` on the rank, or `redeem.run-as: player` for all of them. An
unrecognised value falls back to console and warns in the console.

### The reward should announce something to everyone.

That is what `messages.redeem-broadcast` does, once per rank redeemed. A player redeeming
three ranks produces three broadcasts - that is intentional.

### Does `/reclaim` work from the console?

No, redeeming is per-player. The console gets the help instead. The admin subcommands
(`reset`, `enable`, `disable`, `status`, `reload`) all work from the console.

### Where are the claims stored?

`plugins/SnReclaim/data.db`, table `snreclaim_claims(uuid, rank_key, claimed_at)`. Switch
`database.type` to `mysql` to share them across servers.

### Can two ranks point at the same group?

Yes. They are separate reclaims and each is claimed once. What you **cannot** have is two
rank keys that differ only in case - they would collide in the database, so the second is
skipped with a warning.
