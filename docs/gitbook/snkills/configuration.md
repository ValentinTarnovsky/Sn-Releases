# Configuration

Two files: `config.yml` holds the death messages and the weapon presentation,
`lang/messages_en.yml` holds the prefix and the command messages. New keys merge into your
files on boot, keeping your values **and** your comments.

## Death messages

```yaml
death-messages:
  # Wins whenever the victim has a player killer, whatever the cause was.
  PLAYER_KILL: '&c&l☠ &7&l» &c%victim% &7was slain by &c%attacker% &7using %item%'

  FALL: ''
  LAVA: '&6%victim% &7tried to swim in lava.'
  # ... one key per DamageCause, all empty by default

  FALLBACK: '&7&l» &f%victim% &7died.'
```

Lookup order, first non-empty wins:

1. `PLAYER_KILL`, if a player did the killing
2. the entry for the damage cause itself
3. `FALLBACK`

All three empty means a silent death - and the vanilla message is suppressed regardless, so
that death produces nothing at all. SnKills says so in the console once, at boot, rather than
leaving you to discover it.

A death whose cause the server never recorded (some plugin-dealt kills) is looked up under
`CUSTOM`.

## Placeholders

| Placeholder | Meaning |
|---|---|
| `%victim%` | The dying player |
| `%attacker%` | The killer, empty when no player did it |
| `%item%` | The killer's weapon, with its tooltip. Empty when no player did it |
| `%prefix%` | The `prefix:` line from `lang/messages_en.yml` |
| any PAPI | Resolved against the **victim** |

{% hint style="info" %}
PlaceholderAPI resolves against the victim, so `%luckperms_prefix%` written next to
`%attacker%` shows the *victim's* rank. There is no killer-scoped placeholder.
{% endhint %}

## Colours and tags

`&a` codes, `&#RRGGBB` hex, and the MiniMessage colour tags (`<gradient>`, `<rainbow>`,
`<bold>`) all render.

Everything else - `<click>`, `<hover>`, `<insertion>`, `<font>`, `<newline>` - renders as
literal text on purpose. A placeholder can resolve to text a player chose, and this message
goes to every online player.

Two shape notes:

- The item renders as its own component, so its name shows in its own colour. Put `%item%`
  last in a template, or restate the colour after it.
- `[center]`, `[rgb]` and `[small]` apply to the text up to the first `%item%` only.

## Weapon display

```yaml
weapon-display:
  # Wrapper around the item name. Only the first %item% wraps.
  format: '%item%'
  # Shown when the killer's hand is empty. Never carries a hover, and is NOT
  # put through the format above.
  empty-hand: 'Fists'
  # Prepended when the killer held a stack. Empty hides the count.
  amount-format: '&f%amount%x '
```

These three are styled with colour codes only - **no PlaceholderAPI**, unlike the death
templates.

## Base keys

```yaml
lang: en
update-configs: true
debug:
  enabled: false
  level: DEBUG
  categories: []
command:
  aliases: [skills]
```

`update-configs: false` freezes the language file. It does not freeze `config.yml`: SnLib
always merges the main config so a new schema key can never be missing.

## Damage causes across Minecraft versions

Every `DamageCause` the running server knows is present in `config.yml` after boot. A cause
added by a Minecraft update is inserted in place, matching your file's indentation, without
touching anything else. A key that matches no cause on your server is named once in the
console - which is how a typo like `KILLED` gets caught instead of silently doing nothing.
