# SnRankPerks

SnRankPerks gives your donors and ranked players a self-service cosmetics menu: chat color,
glow and a rank prefix, all picked from a GUI or a command, gated behind a LuckPerms group or
a plain permission node.

## Features

- **Two access models.** Gate every feature behind a LuckPerms group, or drop LuckPerms
  entirely and use a single permission node instead. Both models resolve a LuckPerms prefix as
  a fallback display when LuckPerms is present.
- **Chat color picker.** 16 legacy colors, 17 hex colors and 14 gradient and uniform-cycling
  custom colors, browsed across three GUI pages.
- **Glow effects.** 16 solid colors plus an animated rainbow glow, built on scoreboard teams and
  synced to TAB nametags when TAB is installed.
- **Rank prefixes.** 21 configurable gradient prefixes, selectable from a GUI menu.
- **Join system.** Announces a full-access player's connection to the server and hands every
  player a custom interactive item that opens the main menu.
- **A bypass tier.** Grant glow and chat color only, without prefix or the join flow, through a
  single permission node on top of either access model.

## Optional integrations

- **LuckPerms**: powers the GROUP access model and its live sync, and supplies the fallback
  prefix display. Without it, only PERMISSION-mode access is available.
- **PlaceholderAPI**: exposes `%snrankperks_*%` placeholders anywhere on the server. Without it
  the plugin is fully functional; only the placeholders are unavailable.
- **TAB**: mirrors the selected glow color onto the player's TAB nametag and tablist entry.
  Without it, glow still renders through the scoreboard.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
