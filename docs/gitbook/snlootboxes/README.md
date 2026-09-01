# SnLootBoxes

Configurable lootbox system with animated GUI opening. You define lootboxes as YAML files or build them in the in-game editor. Players receive physical key items and right-click them to open an animated chest grid.

## Features

- Animated 3x3 chest-grid opening with a super chest that unlocks after all nine normal chests.
- Rewards are rolled and reserved the moment you click; an early close, logout or restart never loses them.
- Weighted reward pools with per-reward enable toggles, NBT item rewards and console-command rewards.
- Full in-game editor: create, rename, duplicate and delete lootboxes, and capture rewards from your inventory.
- Per-lootbox aesthetics: 3-stop gradient names, bullet character, key glow and custom model data.
- Keys of the same lootbox always stack: held keys are re-rendered from the current definition on join and before every grant.
- Fast-open by shift + right-click, denied above a configurable stack size so a full stack cannot be burst-opened.
- Per-lootbox personal or global announcements, and a per-IP giveall cap.

## Optional integrations

- **PlaceholderAPI**: unlocks the `%lootboxes_*%` placeholders and placeholder resolution in messages and titles. Without it, the plugin runs normally and placeholder tokens stay unresolved.
- **WorldGuard**: unlocks the `access.allowed-regions` restriction. Without it, a configured region list fails closed and denies every opening. Leave the list empty when WorldGuard is absent.

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
