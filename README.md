# Sn-Releases

Shared public releases repo for the Sn Minecraft plugin family.

This repo carries no source code. It exists so every closed-source Sn plugin (obfuscated, sold separately, source kept in a private `Sn<Name>` repo) can publish its releases to **one** public place instead of maintaining a dedicated public releases repo per plugin. Each server's `SnLib` `UpdateChecker` polls this repo and filters by a per-plugin tag prefix, so it never needs a token and never sees anyone else's source.

## Tag convention

Every release is tagged:

```
<pluginId>-vX.Y.Z
```

For example `snclans-v1.4.0` for SnClans, `snoki-v2.0.0` for SnOki. `pluginId` is the plugin's lowercase name (no spaces/`Sn` prefix needed, just something stable and unique across the family).

## How a plugin wires this up

```java
@Override
protected SnSpec buildSpec() {
    return SnSpec.builder()
            .config("config.yml")
            .updates("ValentinTarnovsky/Sn-Releases", "snclans-")   // owner/repo, tag prefix
            .build();
}
```

Requires `com.sn:snlib` 1.4.0 or later (adds the `updates(ownerRepo, tagPrefix)` overload). See the [UpdateChecker docs](https://github.com/ValentinTarnovsky/SnLib/blob/main/docs/gitbook/developers/modules/update-checker.md#shared-releases-repo-one-public-repo-for-many-plugins).

## Publishing a release

```bash
gh release create <pluginId>-vX.Y.Z \
  --repo ValentinTarnovsky/Sn-Releases \
  --title "<PluginName> vX.Y.Z - Brief title" \
  --notes "## Changes
- [Description]

## Installation
Download the JAR below and place it in your plugins folder. Requires SnLib.jar." \
  target/<PluginArtifact>-X.Y.Z.jar
```

The tag stays prefixed with the plugin id; the release title/notes can be human-friendly.
