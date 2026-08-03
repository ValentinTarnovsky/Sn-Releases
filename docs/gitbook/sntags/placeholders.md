# Placeholders

Requires **PlaceholderAPI**. Without it SnTags runs normally and the expansion is simply not registered.

Expansion identifier: `sntags`.

| Placeholder | Returns |
|-------------|---------|
| `%sntags_tag%` | The player's active tag display text, **raw and uncolorized** |
| `%sntags_has_tag%` | `true` or `false` |

## Raw output is the contract

`%sntags_tag%` returns the display text with its `&` and `&#RRGGBB` codes intact, so the consuming plugin renders them itself. This has been true since 1.0.0 and is what makes the placeholder safe to use across a network in chat formats, TAB, scoreboards and holograms.

For a player with no tag it returns `placeholders.no-tag` from `config.yml`, which is empty by default. Branch on `%sntags_has_tag%` rather than on the text being empty - a tag whose display is styling only would otherwise read as "no tag".

## Both placeholders need a viewer

{% hint style="warning" %}
Parsed with no player - a hologram line, a console-context parse - both placeholders are left as the literal text `%sntags_tag%` / `%sntags_has_tag%` instead of resolving.
{% endhint %}

This is decided in the PlaceholderAPI bridge, above the plugin, so it cannot be changed from SnTags.

It matters most where you branch: a condition on `%sntags_has_tag%` in a viewer-less context receives the literal string, which is neither `true` nor `false`, so the condition takes the else branch **for every player, including the ones who do have a tag**. Wherever you use these, make sure the parse carries a player.

In 1.x, `%sntags_tag%` returned an empty string in that situation. That is the one behaviour change in the placeholder contract for 2.0.0.

## Examples

Chat format, with a separator that only appears when the player has a tag:

```yaml
format: "%sntags_tag% &7%player_name%&f: %message%"
```

Conditional, in a plugin that supports it:

```yaml
prefix: "%sntags_has_tag%" == "true" ? "%sntags_tag% " : ""
```
