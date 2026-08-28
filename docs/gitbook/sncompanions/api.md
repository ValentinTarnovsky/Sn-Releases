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

All events are in `com.sn.companions.api.event` and fire on the main thread.

Cancellable events fire before the action. Cancelling aborts it.

| Event | Fired when | Cancel effect |
|-------|-----------|---------------|
| `CompanionEquipEvent` | A player equips a stored companion from the companion menu | The companion stays in storage |
| `CompanionUnequipEvent` | A player sends an equipped companion back to storage | The companion keeps its slot and its buffs |
| `CompanionBoxOpenEvent` | A player opens one or more companion boxes, before the roll | Every box is handed back |
| `CompanionFuseEvent` | A player commits a fusion, before anything is charged | Nothing is charged and no companion is destroyed |
| `CompanionGroupDeleteEvent` | A player mass-deletes a group from the bulk delete menu | Every companion in the group stays |
| `CompanionRollEvent` | A player rolls a trait or a boost grade, before paying | No currency is spent and no modifier is written |

{% hint style="warning" %}
These fire only for player-driven actions. The `/companions admin` equivalents bypass them on purpose,
so an operator override is never vetoable.
{% endhint %}

Notification events fire after the fact. They cannot be cancelled.

| Event | Fired when | Payload |
|-------|-----------|---------|
| `CompanionBoxRewardEvent` | A box open resolved to a set of companions | Which companions were won, and how many boxes opened |
| `CompanionLevelUpEvent` | An equipped companion gained at least one level | The companion after the level up, and how many levels it gained |
| `CompanionFusedEvent` | A fusion consumed its parents | Whether it succeeded, and the companion it produced |
| `CompanionRolledEvent` | A trait or boost roll was decided and written | One entry per rewritten slot, with the old and new id |

{% hint style="info" %}
Since 1.4.0 a companion box carries a success chance and can fail to open, consuming the box and
granting nothing. A failed open fires **no** event: `CompanionBoxRewardEvent` only ever fires when
there was a reward, and `CompanionBoxOpenEvent` has already fired by then (it runs before the roll,
so it sees the attempt whether or not it succeeds). A dedicated fail event may be added in a
later version; adding one would be a MINOR `API_VERSION` bump, never a breaking change.
{% endhint %}

{% hint style="info" %}
Since 1.7.0 a stored companion can leave the storage as a physical item and be redeemed back into
somebody's storage - since 1.17.0 only by the player who took it out, unless the item predates
that version. Neither half fires an event in this version, and `API_VERSION` stays `1.0.0`
because nothing on the surface changed, the 1.17.0 owner lock included: it is a refusal inside
the redemption, not a new type or method. A companion that arrives by redemption is an ordinary new row,
so every query on the facade sees it immediately; what you cannot do yet is veto or observe the
extraction and the redemption themselves. `CompanionExtractEvent` and `CompanionRedeemEvent` are candidates
for a later version, and adding them would be a MINOR `API_VERSION` bump, never a breaking change.
{% endhint %}

{% hint style="info" %}
Since 1.19.0 a LEVEL scroll raises a companion's level and a RARITY scroll retypes one. Neither fires an
event, and `API_VERSION` stays `1.0.0`. In particular **`CompanionLevelUpEvent` does not fire for a
scroll**: it belongs to the natural experience path, and a scroll is a write in the shape of
`/companions admin setlevel`, which does not fire it either. Both changes are ordinary writes, so every
query on the facade sees the result immediately - a retype produces a new row with a NEW instance
id and the old id stops existing. `CompanionScrollUsedEvent` is a candidate for a later version, and
adding one would be a MINOR `API_VERSION` bump, never a breaking change.
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
thread. None of them touches the database.

| Method | Returns | Notes |
|--------|---------|-------|
| `getStorage(UUID)` | `Optional<CompanionStorageView>` | Counts and capacities. Empty exactly when the player is not loaded |
| `getCompanion(UUID, int)` | `Optional<CompanionView>` | One owned companion by its instance id |
| `getEquippedCompanions(UUID)` | `List<CompanionView>` | The player's equipped companions, in slot order |
| `getStoredCompanions(UUID)` | `List<CompanionView>` | The player's stored companions |
| `getCurrencyBalance(UUID, String)` | `int` | Balance of `TRAIT_TICKET`, `DICE_NORMAL` or `DICE_SPECIAL` |
| `getBuffTotal(UUID, String)` | `double` | Total percentage for `DAMAGE`, `RESISTANCE` or `SPEED` |
| `getCompanionTypes()` | `List<CompanionTypeView>` | Every companion definition the server declares |
| `getCompanionType(String)` | `Optional<CompanionTypeView>` | One companion definition by id |
| `getTraits()` | `List<TraitView>` | Every trait in the global table |
| `getBoostGrades()` | `List<BoostGradeView>` | Every grade in the global ladder |
| `getApiVersion()` | `String` | The API contract version |

{% hint style="warning" %}
SnCompanions loads a player's collection on join and saves it on quit, so these queries only answer for a
player who is currently loaded. For anyone else they return an empty list or an empty `Optional`.
Use `getStorage` to tell "not loaded" apart from "owns nothing".
{% endhint %}

## Views

Views live in `com.sn.companions.api.model` and are immutable snapshots taken when you call the method.
They never change afterwards, so re-query when you need current values.

| View | Describes |
|------|-----------|
| `CompanionView` | One owned companion: level, experience, trait, the three boost slots, equip slot |
| `CompanionTypeView` | One companion definition: display name, level cap, group, buff, fusion target |
| `CompanionStorageView` | One player's counts and capacities |
| `TraitView` | One trait: weight, table share, and what it modifies |
| `BoostGradeView` | One boost grade: weight, percentage range, and the slots it fits |
| `RollChangeView` | One rewritten modifier slot: the old id and the new one |

Companions, traits and boost grades cross the API as their id `String`, and internal enums cross as their
name. Renaming or reordering a config entry can never break your plugin at runtime.

{% hint style="info" %}
An unequipped companion reports `equippedSlot()` as `0`. Slots are numbered from 1, so `equipped()` is the
clearest way to ask.
{% endhint %}

## Versioning

Call `getApiVersion()` for the API contract version. It is independent of the plugin version.
Additions bump the minor component. Existing members are never removed or changed; deprecated
members keep working.
