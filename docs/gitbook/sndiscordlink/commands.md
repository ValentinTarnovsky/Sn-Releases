# Commands

The root command is `/discord`, with the aliases `/dl` and `/discordlink`. The aliases are configured under `command.aliases` in `config.yml` and re-read on `/discord reload`. Three shortcuts are registered on their own: `/link`, `/unlink` and `/booster`.

## Player commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/discord link` | `sndiscordlink.use` | Get a 6-digit code to redeem in Discord with `/link <code>` |
| `/link` | `sndiscordlink.use` | Shortcut of `/discord link` |
| `/discord verify <code>` | `sndiscordlink.use` | Redeem a code issued from Discord with `/link` |
| `/discord unlink` | `sndiscordlink.use` | Unlink your Discord account |
| `/unlink` | `sndiscordlink.use` | Shortcut of `/discord unlink` |
| `/discord status` | `sndiscordlink.use` | Show your link, link date and boost status |
| `/discord claim <claim>` | `sndiscordlink.claim` | Claim a configured reward gated by a boost or a Discord role |
| `/booster` | `sndiscordlink.claim` | Shortcut of `/discord claim booster` |
| `/discord help` | none | Show the help menu |

## Admin commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/discord info <player>` | `sndiscordlink.admin.info` | Show a player's link, Discord roles and rank groups |
| `/discord forceunlink <player>` | `sndiscordlink.admin.forceunlink` | Remove a player's link |
| `/discord forcelink <player> <discordId>` | `sndiscordlink.admin.forcelink` | Link a player to a Discord id |
| `/discord sync <player>` | `sndiscordlink.admin.sync` | Re-snapshot a player's permanent ranks for the role sync |
| `/discord stats` | `sndiscordlink.admin.stats` | Link count, boosters and which server hosts the bot |
| `/discord reload` | `sndiscordlink.admin.reload` | Reload the configuration and the language file |
| `/discord debug` | `sndiscordlink.admin.debug` | Toggle the live debug output |

{% hint style="info" %}
`/discord stats` reports the reconciler of the server you run it on. Run it on the bot host for the full picture.
{% endhint %}

## Discord commands

The bot registers these slash commands in your guild. Their names are configurable under `discord.commands` in `config.yml`.

| Command | Description |
|---------|-------------|
| `/link [code]` | Redeem a code from `/discord link`, or issue a code for `/discord verify` when run without one |
| `/unlink` | Unlink the Minecraft account of the caller |
| `/dlink info <player>` | Staff: show a player's link |
| `/dlink unlink <player>` | Staff: remove a player's link |
| `/dlink accounts <user>` | Staff: list the accounts linked to a Discord user |
| `/dlink stats` | Staff: link count and boosters |

{% hint style="warning" %}
The `/dlink` group needs the **Manage Server** permission in Discord. It is not gated by a Minecraft permission node.
{% endhint %}
