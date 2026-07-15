# Commands

The root command is `/clan`. It ships with the alias `/c`, which you can change under `command.aliases` in `config.yml`. That alias list is authoritative and is re-read on `/clan reload`. Run `/clan` alone to open the main menu, or `/clan help` to list every subcommand. Many actions require a minimum clan role, which you set in `config.yml`.

## Player commands

| Command | Description |
|---------|-------------|
| `/clan` or `/clan menu` | Open the clan menu |
| `/clan create <name> [tag]` | Create a clan |
| `/clan disband [confirm]` | Disband your clan (leader only) |
| `/clan invite <player>` | Invite a player to your clan |
| `/clan accept <name>` | Accept an invite or join request |
| `/clan deny <name>` | Decline an invite or join request |
| `/clan join <clan>` | Join or request to join a clan |
| `/clan leave` | Leave your clan |
| `/clan kick <player>` | Kick a member |
| `/clan ban <player>` | Ban a player from your clan |
| `/clan unban <player>` | Unban a player |
| `/clan promote <player>` | Promote a member one rank |
| `/clan demote <player>` | Demote a member one rank |
| `/clan transfer <player> [confirm]` | Transfer clan ownership (leader only) |
| `/clan rename <name>` | Rename your clan |
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
| `/clan roster` | Show your clan members |
| `/clan settings <policy\|motd> [value]` | Set the join policy or message of the day |
| `/clan help` | Show command help |

## Admin commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/clan givepoint <point> <amount> [clan]` | `snclans.admin.givepoint` | Grant or remove custom points |
| `/clan forcedisband <clan> [-s]` | `snclans.admin.forcedisband` | Force-disband any clan |
| `/clan forcetransfer <clan> <player> [-s]` | `snclans.admin.forcetransfer` | Force-transfer clan leadership |
| `/clan reload` | `snclans.admin.reload` | Reload configuration and language files |
| `/clan debug` | `snclans.admin.debug` | Toggle runtime debug output |

{% hint style="info" %}
The optional `-s` flag on `forcedisband` and `forcetransfer` runs the action silently. It requires the `snclans.admin.silent` permission.
{% endhint %}

{% hint style="danger" %}
`forcedisband` permanently deletes a clan and its data. This cannot be undone.
{% endhint %}

## Tab completion

Every argument suggests real values as you type:

- Clan names for `join`, `ally`, `unally`, `info`, `givepoint`, `forcedisband`, and `forcetransfer`.
- Your own clan's members for `kick`, `ban`, `promote`, `demote`, and `transfer`.
- Your clan's ban list for `unban`.
- Online player names for `invite` and the `forcetransfer` target.
- The `confirm` and `-s` literals where those apply.
- An angle-bracket hint such as `<name>` or `<message>` for free-form arguments.

{% hint style="info" %}
Suggestions are a convenience only. Offline members stay typable, any typed value is still accepted, and every command's behavior and error messages are unchanged.
{% endhint %}
