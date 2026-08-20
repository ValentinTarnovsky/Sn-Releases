# Permissions

Every permission node in SnPacketBosses defaults to `op`. A fresh install therefore gives normal players nothing, and operators get the full admin surface. Grant the nodes explicitly through your permissions plugin to reach non-operator staff ranks.

| Permission | Default | Description |
| --- | --- | --- |
| `snpacketbosses.admin` | `op` | Full administrative access of SnPacketBosses. |
| `snpacketbosses.admin.reload` | `op` | Allows `/packetbosses reload`. |
| `snpacketbosses.admin.debug` | `op` | Allows `/packetbosses debug`. |
| `snpacketbosses.admin.update` | `op` | Receives update notifications for SnPacketBosses. |
| `snpacketbosses.admin.give` | `op` | Allows `/packetbosses give`. |
| `snpacketbosses.admin.kill` | `op` | Allows `/packetbosses kill`. |
| `snpacketbosses.admin.list` | `op` | Allows `/packetbosses list`. |
| `snpacketbosses.admin.view` | `op` | Allows `/packetbosses view`. |
| `snpacketbosses.admin.lock-bypass` | `op` | Bypasses the command, teleport and flight locks during a boss fight. |

## The parent node and its children

`snpacketbosses.admin` declares an exhaustive children map. Granting it grants all eight child nodes at once, the lock bypass included. Give a staff rank the parent when you want the whole admin surface, or pick individual children when you want a narrower role.

The parent node also gates the root command itself. No subcommand of `/packetbosses` is player facing, so a player without the parent never opens an empty help page.

{% hint style="info" %}
The `reload` and `debug` subcommands are injected by SnLib into the command tree, so their nodes exist even though no plugin code declares those commands. The `update` node is the chat notice of the update checker, not a command.
{% endhint %}

## What the lock bypass actually does

While a fight is active, the plugin can lock the owner down. It can block commands outside the allowlist, block every teleport, and block flight in the configured worlds. All of that is configured under `locks` in `config.yml`.

A player holding `snpacketbosses.admin.lock-bypass` is exempt from all three locks. Use it for staff who need to move, teleport or fly while a fight of their own is running. Spectating admins are never locked, because the locks only apply to the owner of a fight.

{% hint style="warning" %}
Granting `snpacketbosses.admin` to a staff rank silently grants the lock bypass too. If you want a moderator who can hand out eggs but still fights under the normal rules, grant `snpacketbosses.admin.give` alone instead of the parent.
{% endhint %}

{% hint style="success" %}
The `/packetbosses` root always stays runnable for a locked player, whatever the allowlist says. That is the admin recovery path out of a stuck fight, and it cannot be closed by editing the config.
{% endhint %}

## Players need nothing

There is no player-facing permission node. Summoning a boss requires only a boss egg in hand, and eggs are tradeable between players. Access control for regular players is the egg itself, not a permission.
