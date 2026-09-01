# Configuration

Two files: `config.yml` (ranks and behaviour) and `lang/messages_en.yml` (every message).

## Ranks

The heart of the plugin. Each entry's key is the rank's identity - **it is also what gets
stored in the database**, so renaming a rank makes it a new one and everyone who already
claimed it can claim again.

```yaml
# sn:extensible
ranks:
  vip:
    display-name: "&aVIP"        # shown wherever {rango} appears; defaults to the key; PAPI works
    group: "vip"                 # the Vault group matched; defaults to the key
    run-as: console              # console or player; defaults to redeem.run-as
    message:                     # private message, sent only to the redeeming player
      - "&aYou claimed your {rango} &areclaim."
      - " "                      # a single space is a blank spacer line
      - "&7Enjoy your rewards!"
    commands:                    # the actual reward
      - "crates give {player} vip 1"
      - "give {player} diamond 5"
```

- `{player}` is replaced with the player's name in commands and messages.
- `{rango}` is the rank's `display-name`. The display name may itself be a PlaceholderAPI
  token - `display-name: '%snrankperks_prefix%'` - and it resolves against the **redeeming
  player**, in the public broadcast and in the private message alike.
- Messages support `&` colours, `&#RRGGBB` HEX and PlaceholderAPI. **No prefix is added** -
  the layout is entirely yours.
- A leading `/` on a command is optional.
- The section is marked `# sn:extensible`: ranks you delete stay deleted.

{% hint style="danger" %}
**Never rename a rank key to "fix" a typo on a live server.** The key is the claim's
identity in the database. Renaming `vip` to `Vip` makes it a rank nobody has claimed, and
every player who already took it gets the reward a second time.
{% endhint %}

A rank that can grant nothing - no usable commands **and** no message - is skipped at load
with a warning, rather than burning a player's one-time claim for nothing.

## Redemption

```yaml
redeem:
  enabled: true       # the season lock; /reclaim enable|disable writes here
  run-as: console     # default execution context for reward commands
```

## Join notification

```yaml
join-notify:
  enabled: true
  delay-ticks: 40     # 20 ticks = 1 second
```

{% hint style="info" %}
**The delay is not cosmetic.** Your permissions plugin has not finished attaching a
player's groups when they join, so a notification sent instantly finds no ranks and says
nothing. 40 ticks is a safe default; raise it on a busy server, do not set it to 0.
{% endhint %}

## Messages

Everything players and admins read lives in `lang/messages_en.yml`. Restyle any line - your
edits survive updates.

To run a different language, drop `lang/messages_es.yml` next to it and set `lang: es` in
`config.yml`.

The top-level `status:` block holds the short words spliced into other messages
(`ENABLED` / `DISABLED`), so recolouring them once changes every line that renders them.

{% hint style="info" %}
Do not write `{prefix}` in a message value. SnLib prepends the `prefix` line automatically.
{% endhint %}

## Database

```yaml
database:
  type: sqlite
  file: data.db
```

`file: data.db` holds every claim ever made. Changing it starts from an empty table, so
everyone can claim again.
