# Commands

The root command is `/killeffects`, with the aliases `/ke` and `/killeffect`. You control the alias list from `command.aliases` in `config.yml`, and it is re-read on every `/killeffects reload`.

Running `/killeffects` with no arguments opens the catalogue for a player, and prints the help for the console.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/killeffects` | `snkilleffects.use` | Open the effect catalogue |
| `/killeffects select <effect>` | `snkilleffects.use` | Activate one of your effects. Tab completion suggests only what you own, plus `random` |
| `/killeffects off` | `snkilleffects.use` | Deactivate your current effect |
| `/killeffects list` | `snkilleffects.use` | List the effects you own and how long each one has left |
| `/killeffects preview <effect>` | `snkilleffects.preview` | Play an effect at your own location. Grants nothing |
| `/killeffects toggle` | `snkilleffects.toggle` | Hide or show other players' kill effects |
| `/killeffects give <player> <effect> [duration] [-s]` | `snkilleffects.admin.give` | Grant an effect, permanently or for a time. Works on offline players |
| `/killeffects take <player> <effect> [-s]` | `snkilleffects.admin.take` | Remove one effect from a player |
| `/killeffects giveall <effect> [duration] [-s]` | `snkilleffects.admin.giveall` | Grant an effect to every online player |
| `/killeffects admin set <player> <effect> [-s]` | `snkilleffects.admin.set` | Force a player's active effect, granting it first if needed |
| `/killeffects admin clear <player> [-s]` | `snkilleffects.admin.clear` | Remove every effect from a player |
| `/killeffects admin list <player>` | `snkilleffects.admin.list` | Inspect what an online player owns |
| `/killeffects help` | `snkilleffects.use` | Show the help menu |
| `/killeffects reload` | `snkilleffects.admin.reload` | Reload the configuration |

## Durations

The optional duration accepts values like `30m`, `12h`, `7d` or `30d`. Omit it for a permanent grant. Granting an effect a player already owns extends it: a permanent grant stays permanent, and a timed grant adds to the time that is left rather than replacing it.

## The silent flag

`-s` suppresses the notification the TARGET receives. You always get your own confirmation. It requires `snkilleffects.admin.silent`, and you can put it in either order, so both of these work:

```
/killeffects give Steve rocket 7d -s
/killeffects give Steve rocket -s
```

{% hint style="danger" %}
`/killeffects admin clear <player>` removes every effect a player owns and cannot be undone.
{% endhint %}
