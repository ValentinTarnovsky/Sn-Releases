# SnRotatinHeads

Decorative rotating, bouncing player heads and custom item models for lobbies and hubs. Place a
textured head or any resource-pack model anywhere, give it a DecentHolograms text label, and bind
left and right click actions to it. One shared animation task drives every head with client-side
interpolation, so hundreds of heads cost almost nothing.

## Features

- Heads are `ItemDisplay` entities with an `Interaction` hitbox: they spin, bounce above their anchor
  and stay clickable at every point of the bounce.
- Per-head texture (base64 textures value or skin URL), size, rotation speed, bounce speed, bounce
  height and view range in blocks. Every setter applies live, no restart.
- Custom models instead of a skull: any item with a `custom-model-data` or (1.21.2+) `item_model`
  selector, typed as `<material>[:<selector>]` or read straight off the item in your hand with
  `/rh model <id> hand`, so an ItemsAdder, Nexo or plain resource-pack item becomes a head. The item
  display context (`ground`, `fixed`, `head`, ...) is per head, so a model renders with the pivot it
  was made for.
- Optional DecentHolograms label per head with any number of lines; the label can follow the bounce.
- Left and right click action lists in the SnLib `[tag] argument` syntax: run commands as the player
  or the console, send messages, play sounds, show titles, and more.
- Heads spawn and despawn with their chunks, survive restarts and unclean shutdowns, and reload live
  with `/rotatinheads reload`, which also re-reads a hand-edited `heads.yml`.

## Optional integrations

- **DecentHolograms**: text labels above heads. Without it, heads work exactly the same and simply
  show no label; the plugin tells you so when you add a label line.
- **PlaceholderAPI**: placeholders in messages, in click action payloads and (through DecentHolograms)
  in label lines. Without it, placeholders are left as typed.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
