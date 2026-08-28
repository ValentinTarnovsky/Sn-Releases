# Developer API

SnCompanions exposes a public developer API for other plugins: custom Bukkit events and a read-only
query service. The API lives in the `com.sn.companions.api` package inside the plugin jar. There is no
separate artifact.

{% hint style="warning" %}
Only `com.sn.companions.api` is a stable, kept contract. Everything else in the jar is obfuscated and
internal. Do not call it.
{% endhint %}

## Getting the jar

1. Download the latest release jar.
2. Install it into your local Maven repository:

   ```
   mvn install:install-file -Dfile=SnCompanions-<version>.jar -DgroupId=com.sn \
     -DartifactId=SnCompanions -Dversion=<version> -Dpackaging=jar
   ```

3. Depend on it with `provided` scope. Never shade it.

## Quick start

Declare the dependency in your `plugin.yml`:

```yaml
depend: [SnCompanions]      # or softdepend if optional
```

Resolve the API when you need it:

```java
SnCompanionsAPI api = SnCompanionsProvider.get();
if (api != null) {
    api.getEquippedCompanions(player.getUniqueId())
       .forEach(companion -> getLogger().info(companion.companionId() + " lvl " + companion.level()));
}
```

{% hint style="info" %}
Resolve the reference when you need it. Do not cache it across a SnCompanions reload.
{% endhint %}

## Master switch

API events can be disabled by the server owner with `api-events.enabled: false` in `config.yml`.
Cancellable hooks then report "not cancelled" so gameplay proceeds. The query service stays
available either way.

## Events

All events are in `com.sn.companions.api.event` and fire on the main thread. There are eight: five
cancellable and three notifications.

Cancellable events fire before the action. Cancelling aborts it.

| Event | Fired when | Cancel effect | Payload |
|-------|-----------|---------------|---------|
| `CompanionEquipEvent` | A player equips a stored companion from the companion menu | The companion stays in storage | `getPlayer()`, `getCompanion()` |
| `CompanionUnequipEvent` | A player sends an equipped companion back to storage | The companion keeps its slot and its buffs | `getPlayer()`, `getCompanion()` |
| `CompanionEggOpenEvent` | A player opens one or more companion eggs, after the gates and before anything is charged or granted | Nothing is charged and nothing is granted | `getPlayer()`, `getEggId()`, `getAmount()`, `isCharged()` |
| `CompanionFuseEvent` | A player commits a fusion, before anything is charged | Nothing is charged and no companion is destroyed | `getPlayer()`, `getFirst()`, `getSecond()`, `isBulk()` |
| `CompanionGroupDeleteEvent` | A player mass-deletes a group from the bulk delete menu | Every companion in the group stays | `getPlayer()`, `getGroupId()`, `getCompanionCount()` |

{% hint style="warning" %}
These fire only for player-driven actions. The `/companions admin` equivalents bypass them on purpose,
so an operator override is never vetoable.
{% endhint %}

{% hint style="info" %}
`CompanionFuseEvent` reports a Fuse All with `isBulk()` true, and a bulk fusion carries no pair, so
`getFirst()` and `getSecond()` are then `null`. Cancelling it aborts the whole run.
{% endhint %}

Notification events fire after the fact. They cannot be cancelled.

| Event | Fired when | Payload |
|-------|-----------|---------|
| `CompanionEggRewardEvent` | An egg open finished and its companions are persisted | `getPlayer()`, `getEggId()`, `getCompanionsWon()`, `getTotalCompanions()`, `getEggsOpened()`, `isCharged()` |
| `CompanionLevelUpEvent` | An equipped companion gained at least one level | `getPlayer()`, `getCompanion()` after the level up, `getLevelsGained()` |
| `CompanionFusedEvent` | A fusion consumed its parents | `getPlayer()`, `getResult()`, `getProduced()`, `isBulk()`, `getPairs()`, `getSucceeded()`, `getFailed()` |

{% hint style="info" %}
`CompanionEggRewardEvent` fires only when the open actually produced something, and it fires
**after** the companion rows are persisted, so every companion it names is already queryable through
the facade when your listener runs. Its `companionsWon` map reports what was CREATED, not what the
table decided, so an open that only partly landed reports the smaller number. A refused open (storage
full, the master switch off, a cancelled `CompanionEggOpenEvent`) fires nothing at all.

`isCharged()` tells a purchase from a gift: it is `true` when the open came from a paid button and
`false` for `/companions admin openegg`, which skips the creative gate, the egg's cooldown and the
price entirely. On `CompanionEggOpenEvent` it says what the open IS ABOUT to be, because the event
fires **before** the charge - cancelling it costs the player nothing. On `CompanionEggRewardEvent` it
says what the open WAS.

A paid open that could not write its companions is refunded in full and fires no reward event, so
a listener never sees a purchase that produced nothing. The hatch animation only delays the FEEDBACK
of an open that already happened, so both events fire at exactly the same points whether or not the
player watches it.
{% endhint %}

{% hint style="info" %}
A stored companion can leave the storage as a physical item and be redeemed back into the storage of
the player who took it out. Neither half fires an event: a companion that arrives by redemption is an
ordinary new row, so every query on the facade sees it immediately, but you cannot veto or observe
the extraction and the redemption themselves. `CompanionExtractEvent` and `CompanionRedeemEvent` are
candidates for a later version; adding them would be a MINOR `API_VERSION` bump, never a breaking
change.
{% endhint %}

Listen like any Bukkit event:

```java
@EventHandler
public void on(CompanionGroupDeleteEvent event) {
    if (isProtected(event.getPlayer())) {
        event.setCancelled(true);
    }
}
```

Event payloads are read-only. `setCancelled` is the only setter any event carries.

{% hint style="info" %}
A failed fusion is a normal outcome, not an error. It destroys both parents and still charges the
price, so `CompanionFusedEvent` reports `FAILED` with no produced companion.
{% endhint %}

## Query service

Every method is synchronous and reads in-memory state, so all of them are safe to call on the main
thread. None of them touches the database. There are eight.

| Method | Returns | Notes |
|--------|---------|-------|
| `getStorage(UUID)` | `Optional<CompanionStorageView>` | Counts and capacities. Empty exactly when the player is not loaded |
| `getCompanion(UUID, int)` | `Optional<CompanionView>` | One owned companion by its instance id |
| `getEquippedCompanions(UUID)` | `List<CompanionView>` | The player's equipped companions, in slot order |
| `getStoredCompanions(UUID)` | `List<CompanionView>` | The player's stored companions |
| `getBuffTotal(UUID, String)` | `double` | Total percentage for `DAMAGE`, `RESISTANCE` or `SPEED` |
| `getCompanionTypes()` | `List<CompanionTypeView>` | Every companion definition the server declares |
| `getCompanionType(String)` | `Optional<CompanionTypeView>` | One companion definition by id |
| `getApiVersion()` | `String` | The API contract version |

{% hint style="warning" %}
SnCompanions loads a player's collection on join and saves it on quit, so these queries only answer for a
player who is currently loaded. For anyone else they return an empty list or an empty `Optional`.
Use `getStorage` to tell "not loaded" apart from "owns nothing".
{% endhint %}

## Views

Views live in `com.sn.companions.api.model` and are immutable snapshots taken when you call the method.
They never change afterwards, so re-query when you need current values. There are three.

| View | Components |
|------|-----------|
| `CompanionView` | `instance`, `owner`, `companionId`, `level`, `exp`, `equippedSlot`, `obtainedAt`, plus the derived `equipped()` |
| `CompanionTypeView` | `id`, `displayName`, `lore`, `maxLevel`, `groupId`, `buffType`, `buffInitial`, `buffPerLevel`, `fusionTargetId`, `fusionChance`, `fusionCost`, `incompatible`, `placeholder` |
| `CompanionStorageView` | `owner`, `storedCount`, `storageCapacity`, `equippedCount`, `equipSlots` |

Companions cross the API as their id `String`, and internal enums cross as their
name. Renaming or reordering a config entry can never break your plugin at runtime.

{% hint style="info" %}
An unequipped companion reports `equippedSlot()` as `0`. Slots are numbered from 1, so `equipped()` is the
clearest way to ask.
{% endhint %}

## Versioning

Call `getApiVersion()` for the API contract version. It is independent of the plugin version, and it
reports `1.0.0`.

{% hint style="success" %}
**The surface documented on this page is the baseline, and it is additions-only from here.** Nothing
public is removed, renamed, or changed in signature or observable behaviour. A change that needs
different behaviour arrives as a new method, event, or view; the old one keeps working. Every
addition bumps the MINOR component of `API_VERSION`, so a consumer can gate on it.
{% endhint %}

{% hint style="warning" %}
**Plugin versions 1.0.0 through 1.8.0 did not follow that rule, and `API_VERSION` did not move.**
Four features were cut out of the plugin in that window and their surface went with them, removed
outright rather than deprecated, while there was still no adopter to break. The version was held at
`1.0.0` on purpose across those releases: bumping it would have claimed a stability the contract did
not have yet, so for that window **the version does not tell you whether surface went away**.

If you compiled against a jar older than 1.8.1, check the surface you use against this page before
upgrading. From 1.8.1 on, that check is never needed again.
{% endhint %}
