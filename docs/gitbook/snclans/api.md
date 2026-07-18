# Developer API

SnClans exposes a public developer API for other plugins: custom Bukkit events, a
read-only query service, and extension points. The API lives in the
`com.sn.clans.api` package inside the plugin jar. There is no separate artifact.

{% hint style="warning" %}
Only `com.sn.clans.api` is a stable, kept contract. Everything else in the jar is
obfuscated and internal. Do not call it.
{% endhint %}

## Getting the jar

1. Download the latest release jar.
2. Install it into your local Maven repository:

   ```
   mvn install:install-file -Dfile=SnClans-<version>.jar -DgroupId=com.sn \
     -DartifactId=SnClans -Dversion=<version> -Dpackaging=jar
   ```

3. Depend on it with `provided` scope. Never shade it.

## Quick start

Declare the dependency in your `plugin.yml`:

```yaml
depend: [SnClans]      # or softdepend if optional
```

Resolve the API when you need it:

```java
SnClansAPI api = SnClansProvider.get();
if (api != null) {
    // use the api
}
```

{% hint style="info" %}
Resolve the reference when you need it. Do not cache it across a SnClans reload.
{% endhint %}

## Master switch

API events can be disabled by the server owner with `api-events.enabled: false` in
`config.yml`. The query service stays available either way.

## Events

Cancellable events fire before the action. Cancelling aborts it.

| Event | Fired when | Cancel effect |
|-------|-----------|---------------|
| `ClanCreateEvent` | A player is about to create a clan | The clan is not created |
| `ClanDisbandEvent` | A clan is about to disband: command, menu, admin, or last member leaving | The clan stays |
| `ClanMemberJoinEvent` | A player is about to join a clan: open join, invite accept, or admin add | The player does not join |
| `ClanMemberLeaveEvent` | A member is about to leave, be kicked, or be banned | The member stays |
| `ClanAllyFormEvent` | Two clans are about to become allies | The alliance is not formed |

Notification events fire after the fact. They cannot be cancelled.

| Event | Fired when | Thread |
|-------|-----------|--------|
| `ClanPointsChangeEvent` | A clan point total changed: kills, mob kills, death penalty, or admin grant | Main |
| `ClanRoleChangeEvent` | A member role changed: promote, demote, or leader transfer | Main |
| `ClanAllyBreakEvent` | An alliance was broken: unally command or a disband cascade | Main |

Listen like any Bukkit event:

```java
@EventHandler
public void on(ClanCreateEvent event) {
    // read the payload; cancellable events support event.setCancelled(true)
}
```

Event payloads are read-only. To influence values, use the extension points below instead of
looking for setters.

## Query service

Synchronous methods read in-memory state. Call them on the main thread.

| Method | Returns | Notes |
|--------|---------|-------|
| `clan(id)` | `ClanView` or `null` | Lookup by clan id |
| `clanByName(name)` | `ClanView` or `null` | Lookup by display name, styling ignored |
| `clanOf(player)` | `ClanView` or `null` | The clan of a player UUID |
| `isInClan(player)` | `boolean` | Membership check for a player UUID |
| `clans()` | `List<ClanView>` | Immutable snapshot of every clan |
| `top(pointId, limit)` | `List<ClanView>` | Rank-ordered leaderboard for a point type |
| `rankOf(clanId, pointId)` | `int` | Rank of a clan, `0` when unknown |
| `areAllied(a, b)` | `boolean` | Alliance check by clan ids |
| `getApiVersion()` | `String` | The API contract version |

Views are immutable snapshots. `ClanView` copies its collections and clones its home
`Location`. `MemberView` is a plain record. Roles and join policies cross as `String` names.

## Extension points

| Interface | What it influences | Neutral return |
|-----------|--------------------|----------------|
| `ClanPointsModifier` | Point deltas from kills, mob kills, death penalties, and admin grants | The unchanged `context.delta()` |

Register through `registerPointModifier(plugin, modifier)` on the API facade. Modifiers run
in registration order before the total is clamped at zero. Providers must be fast; a provider
that throws contributes the neutral value and never breaks gameplay. The master switch does
not affect modifiers.

## Versioning

Call `getApiVersion()` for the API contract version. It is independent of the plugin version.
Additions bump the minor component. Existing members are never removed or changed; deprecated
members keep working.
