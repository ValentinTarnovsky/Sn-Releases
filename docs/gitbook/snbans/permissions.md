# Permissions

Every SnBans node defaults to `op` except the two player-facing ones, `snbans.helpop` and `snbans.report`, which default to **true**. There is no `snbans.use` node: each command root carries its own leaf node instead. The tree is 27 nodes, declared once for both platforms: the 25 SnBans checks itself behave identically on a backend and on the proxy, and the two SnLib owns are Paper only.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snbans.admin` | op | Full administrative access of SnBans (parent of every other node) |
| `snbans.admin.reload` | op | Allows `/snbans reload` |
| `snbans.admin.debug` | op | Allows `/snbans debug` (Paper only) |
| `snbans.admin.update` | op | Receive update notifications of SnBans (Paper only) |
| `snbans.admin.match` | op | Allows `/snbans match` |
| `snbans.admin.rollback` | op | Allows `/snbans rollback` |
| `snbans.admin.import` | op | Allows `/snbans import` |
| `snbans.admin.wipe` | op | Allows `/snbans wipe` (console only at runtime) |
| `snbans.notify` | op | Receive staff notifications of SnBans punishments, alt scans and refused attempts |
| `snbans.requests.receive` | op | Receive the `/helpop` and `/report` notices of the whole network |
| `snbans.helpop` | **true** | Allows `/helpop` |
| `snbans.report` | **true** | Allows `/report` |
| `snbans.silent` | op | Allows the `-s` / `-p` visibility flag on punishments and reverts |
| `snbans.noreason` | op | Allows issuing a punishment without typing a reason |
| `snbans.ban` | op | Allows `/ban` |
| `snbans.ipban` | op | Allows `/ipban` |
| `snbans.unban` | op | Allows `/unban` |
| `snbans.mute` | op | Allows `/mute` |
| `snbans.ipmute` | op | Allows `/ipmute` |
| `snbans.unmute` | op | Allows `/unmute` |
| `snbans.blacklist` | op | Allows `/blacklist` |
| `snbans.unblacklist` | op | Allows `/unblacklist` (console only at runtime) |
| `snbans.kick` | op | Allows `/kick` |
| `snbans.ipkick` | op | Allows `/ipkick` |
| `snbans.alts` | op | Allows `/alts` |
| `snbans.history` | op | Allows `/history` |
| `snbans.staffhistory` | op | Allows `/staffhistory` |

## What granting `snbans.admin` does

`snbans.admin` is declared with an exhaustive children map: the other 26 nodes are all listed under it. Granting `snbans.admin` alone therefore grants the seven `snbans.admin.*` nodes, `snbans.notify`, `snbans.requests.receive`, `snbans.silent`, `snbans.noreason`, and all fifteen flat command roots. A full-staff rank needs that one node and nothing else.

On Paper, Bukkit expands the children map inside its own permission registry, so a group holding only the parent passes `snbans.ban` without the leaf being granted. A proxy has no descriptor and no registry, so SnBans reproduces the same semantics in its own check: the node's own value is read first, then the two default-true nodes, and only after that does an undefined leaf fall through to the parent. Reproducing the **defaults** as well as the children map is what makes `/helpop` work for an ordinary player on a proxy install, where nothing has granted them anything.

{% hint style="info" %}
An explicit value always wins over the parent, on both platforms. Grant `snbans.admin` and then set `snbans.blacklist` to `false` in LuckPerms, and the revoke is honored. The console passes every node on either platform.
{% endhint %}

## Nodes SnLib owns

Two nodes of the tree belong to SnLib rather than to SnBans:

| Permission | Owner | Why |
|-----------|-------|-----|
| `snbans.admin.debug` | SnLib | Guards the `debug` subcommand SnLib injects into `/snbans`. |
| `snbans.admin.update` | SnLib | Guards the UpdateChecker chat notice SnLib sends. |

SnLib declares and tests both itself, so no line of SnBans code ever names either one. `plugin.yml` still declares them, because the descriptor has to describe the whole node tree an owner grants. Both are Paper only: the proxy shell has no `debug` subcommand (its debug channel is the logger's own level) and runs no update checker.

`snbans.admin.reload` is not in that category. SnLib injects `reload` on Paper and checks the node itself, and the proxy writes its own `reload` subcommand that checks the same node. The node works on both platforms.

## Visibility, reason and notification nodes

`snbans.silent` gates the `-s` / `-p` flag on every issue and every revert. Without it, the per-type `silent-by-default` setting in `config.yml` always decides who hears a punishment. Staff who type a flag they may not use are refused with the branded no-permission line, never silently downgraded.

`snbans.noreason` gates issuing a punishment with no reason at all. Without it, `/ban Notch` answers the usage line and nothing is stored. Holders may run the command bare, and the row records the `messages.format.no-reason` text ("No reason" by default), which is then shown as `{reason}` by every announcement, history line, Discord embed and disconnect screen. It is a node of its own rather than part of `snbans.ban`, because the reason is what a history line and an appeal are read from - so the strict behaviour stays the default and you opt individual ranks out of it. The console holds it implicitly.

`snbans.notify` holders receive the notices for silent punishments, the output of the automatic join alt scan (filtered by `alts.notify-states`) and the attempt notices of `attempt-notices` - a banned account that tried to join, a muted one that tried to talk. The console always receives all three, on a backend and on the proxy alike. One node for the three, because they are one job: watching what SnBans is doing while nobody typed a command.

`snbans.requests.receive` holders receive the `/helpop` and `/report` notices of the whole network. It is a node of its own for exactly the reason the three above share one: a staff request is somebody typing a command precisely so that staff would read it, which is the opposite job - and answering helpops is plausibly a rank that has no business seeing silent punishment notices.

`snbans.helpop` and `snbans.report` are the only two nodes here that default to **true**, because they are the only two commands a player is meant to run. Revoke them to restrict the commands; `staff-requests` in `config.yml` is what switches the feature off, and it answers the player rather than silently doing nothing.

## How the nodes are enforced

`plugin.yml` deliberately declares no `permission:` key on any command. Bukkit tests such a key before the executor runs and answers with the raw descriptor string, which would replace the branded no-permission message. The command dispatchers check the nodes themselves on both platforms, so removing a node from the descriptor does not disable the check.

{% hint style="warning" %}
Two nodes grant less than they look like they do. `snbans.unblacklist` only makes `/unblacklist` grantable and tab-visible, and `snbans.admin.wipe` does the same for `/snbans wipe`: both commands are console only at runtime, and each flow refuses a non-console sender before it reads its own arguments. Granting either node to a rank does not put the command in that rank's hands.
{% endhint %}

{% hint style="info" %}
The staff hierarchy is not a permission node. It reads the LuckPerms primary-group weight and is configured under `hierarchy` in `config.yml`. Without LuckPerms the check stays off, whatever `hierarchy.enabled` says, and every rank can punish every rank.
{% endhint %}
