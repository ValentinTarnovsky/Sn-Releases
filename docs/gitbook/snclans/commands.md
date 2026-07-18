# Commands

The root command is `/clan`. It ships with the alias `/c`, which you can change under `command.aliases` in `config.yml`. That alias list is authoritative and is re-read on `/clan reload`. Run `/clan` alone to open the main menu, or `/clan help` to list every subcommand. Most member actions are gated by your clan's permission matrix, which the leader edits with `/clan permissions`.

## Player commands

| Command | Description |
|---------|-------------|
| `/clan` or `/clan menu` | Open the clan menu |
| `/clan create <name> [tag]` | Create a clan |
| `/clan disband [confirm]` | Disband your clan (leader only) |
| `/clan invite <player>` | Invite a player to your clan |
| `/clan accept <clan>` | Accept a clan invite |
| `/clan deny <clan>` | Decline a clan invite |
| `/clan join <clan>` | Join an open clan |
| `/clan leave` | Leave your clan |
| `/clan kick <player>` | Kick a member |
| `/clan ban <player>` | Ban a player from your clan |
| `/clan unban <player>` | Unban a player |
| `/clan promote <player>` | Promote a member one rank |
| `/clan demote <player>` | Demote a member one rank |
| `/clan transfer <player> [confirm]` | Transfer clan ownership (leader only) |
| `/clan rename <name>` | Rename your clan |
| `/clan open` | Open the clan to instant joins |
| `/clan close` | Make the clan invite-only |
| `/clan description [text]` | Set or clear the clan description |
| `/clan permissions [role] [action] [allow\|deny]` | View or edit the clan permission matrix |
| `/clan ally <clan>` | Request or accept an alliance |
| `/clan unally <clan>` | End an alliance |
| `/clan home` | Teleport to the clan home |
| `/clan sethome` | Set the clan home |
| `/clan banner` | Place or rally to the clan banner |
| `/clan pvp` | Toggle clan friendly fire |
| `/clan chat [message]` | Toggle clan chat, or send one clan message |
| `/clan allychat [message]` | Toggle ally chat, or send one ally message |
| `/clan info [clan]` | Show clan details |
| `/clan list` | List all clans |
| `/clan top <point>` | Show the leaderboard for a point type |
| `/clan stats` | Show your clan kills, deaths, and KDR |
| `/clan members` | Show your clan members |
| `/clan help` | Show command help |

{% hint style="info" %}
Clans are invite-only by default. `/clan open` lets anyone join with `/clan join`, and `/clan close` reverts to invite-only. Run `/clan permissions` with no arguments to open the matrix editor, or `/clan permissions <role> <action> allow|deny` to set one entry from chat. The leader is always allowed and only the leader can edit the matrix.
{% endhint %}

## Admin commands

Every `/clan admin` action needs the `snclans.admin` parent node plus its own leaf node below.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/clan givepoint <point> <amount> [clan]` | `snclans.admin.givepoint` | Grant or remove custom points |
| `/clan admin rename <clan> <name> [-s]` | `snclans.admin.rename` | Rename any clan |
| `/clan admin disband <clan> [-s]` | `snclans.admin.disband` | Force-disband any clan |
| `/clan admin transfer <clan> <player> [-s]` | `snclans.admin.transfer` | Force-transfer clan leadership |
| `/clan admin join <clan> <player> [-s]` | `snclans.admin.join` | Force a player into a clan |
| `/clan admin kick <clan> <player> [-s]` | `snclans.admin.kick` | Kick a member from any clan |
| `/clan admin promote <clan> <player> [-s]` | `snclans.admin.promote` | Promote a member in any clan |
| `/clan admin demote <clan> <player> [-s]` | `snclans.admin.demote` | Demote a member in any clan |
| `/clan admin ban <clan> <player> [-s]` | `snclans.admin.ban` | Ban a player from any clan |
| `/clan admin unban <clan> <player>` | `snclans.admin.unban` | Unban a player from any clan |
| `/clan admin description <clan> [text]` | `snclans.admin.description` | Set or clear any clan's description |
| `/clan admin open <clan> [-s]` | `snclans.admin.open` | Open any clan to instant joins |
| `/clan admin close <clan> [-s]` | `snclans.admin.close` | Make any clan invite-only |
| `/clan reload` | `snclans.admin.reload` | Reload configuration and language files |
| `/clan debug` | `snclans.admin.debug` | Toggle runtime debug output |

{% hint style="info" %}
The optional `-s` flag runs an admin action silently, skipping the target clan's notifications and any broadcast. It works on every `/clan admin` action except `unban`, and requires the `snclans.admin.silent` permission.
{% endhint %}

{% hint style="danger" %}
`/clan admin disband` permanently deletes a clan and its data. This cannot be undone.
{% endhint %}

## Tab completion

Every argument suggests real values as you type:

- Clan names for `join`, `ally`, `unally`, `info`, `givepoint`, and the `/clan admin` subcommands.
- Your own clan's members for `kick`, `ban`, `promote`, `demote`, and `transfer`.
- Your clan's ban list for `unban`.
- Online player names for `invite` and the `/clan admin` player argument.
- Roles, actions, and the `allow`/`deny` literals for `/clan permissions`.
- The `confirm` and `-s` literals where those apply.
- An angle-bracket hint such as `<name>` or `<message>` for free-form arguments.

{% hint style="info" %}
Suggestions are a convenience only. Offline members stay typable, any typed value is still accepted, and every command's behavior and error messages are unchanged.
{% endhint %}
