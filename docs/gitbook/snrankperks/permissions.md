# Permissions

`snrankperks.admin` and its children default to operators. `snrankperks.bypass` is not granted
by default.

| Permission | Default | Description |
|-----------|---------|-------------|
| `snrankperks.admin` | op | Full administrative access of SnRankPerks |
| `snrankperks.admin.reload` | op | Allows `/rankperks reload` |
| `snrankperks.admin.update` | op | Receive update notifications of SnRankPerks |
| `snrankperks.bypass` | false | Grants access to glow and chat color only (not prefix or the join flow), in both access modes |

{% hint style="info" %}
There is no `snrankperks.use` permission. The plugin's real access gate is `access.mode` in
`config.yml` (a LuckPerms group or a plain permission node), checked per feature - not a fixed
Bukkit permission scheme.
{% endhint %}
