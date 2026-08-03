# Commands

## Player

| Command | Description | Permission |
|---------|-------------|------------|
| `/tags` | Open the tag selector menu | `sntags.use` |
| `/tags help` | Show the help menu | `sntags.use` |

`/tag` is a default alias of `/tags`. The alias list lives in `config.yml` under `command.aliases`.

## Admin

| Command | Description | Permission |
|---------|-------------|------------|
| `/tagadmin create <id> <display...>` | Create a global tag in `tags.yml` | `sntags.admin.create` |
| `/tagadmin delete <id>` | Delete a global tag | `sntags.admin.delete` |
| `/tagadmin give <player> <id>` | Grant a tag to a player | `sntags.admin.give` |
| `/tagadmin remove <player> <id>` | Revoke a tag from a player | `sntags.admin.remove` |
| `/tagadmin custom <player> <display...>` | Create a personal tag for one player | `sntags.admin.custom` |
| `/tagadmin removecustom <id>` | Delete a personal tag by its numeric id | `sntags.admin.removecustom` |
| `/tagadmin list` | List every global tag | `sntags.admin.list` |
| `/tagadmin reload` | Reload the configuration | `sntags.admin.reload` |
| `/tagadmin debug` | Toggle debug output | `sntags.admin.debug` |
| `/tagadmin help` | Show the admin help | `sntags.admin` |

`reload`, `help` and `debug` are provided by SnLib on both roots, so `/tags reload` and `/tagadmin reload` are the same action behind the same permission.

`/tagadmin give` and `/tagadmin remove` work on offline players.

## Tag identifiers

Lowercase `a-z`, digits, `-` and `_`, up to 64 characters. Identifiers are lowercased on read, so `VIP` and `vip` are the same tag.

The 64-character limit is the width of the database column. An identifier longer than that would be truncated by MySQL into a grant that vanishes when the player relogs and cannot be revoked, so it is rejected at the command instead.

## Display text

Up to 256 characters. Accepted:

- legacy `&` colour codes
- `&#RRGGBB` hex
- cosmetic MiniMessage: colours, decorations, `<gradient>`, `<rainbow>`

Rejected: **any** `<name>` that is not a colour or a decoration. That includes `<click>`, `<hover>` and `<insertion>`, but also a plain label like `<VIP>` - write `[VIP]` instead. Tag text is shown to other players and exported to other plugins, so interactive elements are refused at the door: a `<click:run_command>` in a tag would run as whoever clicks it.

{% hint style="warning" %}
The check does not cover PlaceholderAPI tokens. A display may contain `%token%`, and it resolves against whoever receives the text - so a tag display is an expression evaluated in someone else's context. This is deliberate and has worked since 1.x, but it means `sntags.admin.custom` is a trusted node. Do not hand it to junior staff.
{% endhint %}

Validation applies to `/tagadmin create` and `/tagadmin custom`. Entries you write directly into `tags.yml` are not re-checked - you already have file access, so there is nothing to protect there.

## Personal tags

`/tagadmin custom` prints the new tag's numeric id. **That id is the only handle the tag ever has**: it is what `/tagadmin removecustom` takes. Keep it if you may need to delete the tag later. `/tagadmin list` shows global tags only.
