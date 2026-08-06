# Permissions

Most nodes default to `op`. Two default to `true`, so every player has them unless you revoke them: `snchat.use` and `snchat.showcase`.

## Player

| Permission | Default | Description |
|-----------|---------|-------------|
| `snchat.use` | true | Allows `/snchat`. It is the root permission, checked before any subcommand |
| `snchat.showcase` | true | Allows the `[item]`, `[inv]` and `[ec]` chat tokens |
| `snchat.color` | op | Allows `&` codes and `&#RRGGBB` hex in your own messages. The cosmetic MiniMessage tags (`<red>`, `<gradient>`, `<rainbow>`) render too |
| `snchat.prefixtags` | op | Allows the line tags `[center]`, `[rgb]`, `[small]` and `[noprefix]` at the start of a message. Needs `snchat.color` |
| `snchat.notify` | op | Eligibility for chat control violation alerts. Alerts are **opt-in** and start off - turn your own on with `/snchat alerts`, once |

{% hint style="info" %}
Both style nodes take effect only in a message that also carries a `&` somewhere. A message with no `&` is published exactly as typed, so `<red>hi` and `[center]hi` reach chat as those literal characters. That guard stops SnChat overwriting a line another plugin already coloured. Type `&r<red>hi` to get the tag rendered.
{% endhint %}

`snchat.prefixtags` is separate from `snchat.color` on purpose. The four line tags affect the whole line rather than a span of it: `[center]` injects padding and `[small]` rewrites the letters. Selling coloured chat to a donor rank therefore does not also sell line centring.

## Administration

| Permission | Default | Description |
|-----------|---------|-------------|
| `snchat.admin` | op | Full administrative access. Parents every node below, plus `snchat.use` and `snchat.notify`. It does **not** grant the bypasses |
| `snchat.admin.reload` | op | Allows `/snchat reload` |
| `snchat.admin.update` | op | Receive update notifications |
| `snchat.admin.debug` | op | Allows `/snchat debug` |
| `snchat.admin.announce` | op | Allows `/snchat announce` |
| `snchat.admin.clearchat` | op | Allows `/snchat clearchat` and `/clearchat` |
| `snchat.admin.mutechat` | op | Allows `/snchat mutechat` and `/mutechat` |
| `snchat.admin.blockcommands` | op | Allows `/snchat blockcommands` |

{% hint style="warning" %}
`snchat.admin` deliberately does not grant `snchat.bypass.*`. An admin is still moderated unless you grant those separately.
{% endhint %}

## Bypasses

| Permission | Default | Description |
|-----------|---------|-------------|
| `snchat.bypass` | op | Parent of the six nodes below, and of those only |
| `snchat.bypass.cooldown` | op | The cooldown clocks |
| `snchat.bypass.caps` | op | The caps check |
| `snchat.bypass.flood` | op | The character repetition check |
| `snchat.bypass.syntax` | op | The `plugin:command` block |
| `snchat.bypass.mutechat` | op | The global mute |
| `snchat.bypass.blockcommands` | op | The command whitelist |

{% hint style="info" %}
The item blacklist is an anti-abuse rule, not a restriction you can be exempted from. Sharing a blacklisted item cancels the message for everyone, `snchat.bypass` holders included.
{% endhint %}
