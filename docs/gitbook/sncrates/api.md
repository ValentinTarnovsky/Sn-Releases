# Developer API

SnCrates fires two Bukkit events so other plugins can react to crate openings and to reward
delivery. The surface is deliberately small: **two events, both notifications**. Neither is
cancellable, and ignoring one changes nothing about the open, the key spend or the reward.

Package: `com.sn.crates.api.event`

| Event | Fired when |
|---|---|
| `CrateOpenEvent` | A player opens a crate: the key has been consumed, the reward has not been delivered yet |
| `CrateRewardEvent` | A won reward has been committed to the player. Exactly once per win |

Both are fired on the main thread, so the Bukkit API is safe to call directly inside a handler.

{% hint style="warning" %}
Only `com.sn.crates.api.**` and the model types its signatures expose are a kept contract.
Everything else in the jar is obfuscated and internal. Do not call it.
{% endhint %}

## Depending on SnCrates

The jar is obfuscated, but the classes under `com.sn.crates.api.**` keep their real names, and so do
the model types their signatures expose (`Crate`, `Reward`, `KeyType`, `PhysicalCrateBlock`).
Compile against the published jar.

Install it into your local Maven repository:

```
mvn install:install-file -Dfile=SnCrates-<version>.jar -DgroupId=com.sn \
  -DartifactId=SnCrates -Dversion=<version> -Dpackaging=jar
```

Then depend on it with `provided` scope - SnCrates is already installed on the server and must not
be bundled into your jar:

```xml
<dependency>
    <groupId>com.sn</groupId>
    <artifactId>SnCrates</artifactId>
    <version>REPLACE_ME</version>
    <scope>provided</scope>
</dependency>
```

Declare it in your `plugin.yml` so it loads first:

```yaml
softdepend: [SnCrates]   # use 'depend' if your plugin cannot work without SnCrates
```

{% hint style="info" %}
Both events can be switched off server-side with `api-events.enabled: false` in `config.yml`. A
plugin that hard-depends on them should say so in its own documentation.
{% endhint %}

## CrateOpenEvent

Fired once per successful open, on every open path: the animated spin, the instant open, and each
crate of a mass-open. It fires **after** the key is consumed and **before** the reward is delivered.

It deliberately carries no reward, which is what keeps "opened" and "won" separate. Listen to
`CrateRewardEvent` for the reward.

```java
public class CrateOpenEvent extends org.bukkit.event.Event {
    public CrateOpenEvent(Player player, Crate crate, KeyType keyType,
                          PhysicalCrateBlock source, boolean massOpen);
    public Player getPlayer();
    public Crate getCrate();
    public KeyType getKeyType();
    public PhysicalCrateBlock getSource();
    public boolean isMassOpen();
    public static HandlerList getHandlerList();
}
```

| Method | Type | Returns |
|---|---|---|
| `getPlayer()` | `org.bukkit.entity.Player` | The player opening the crate |
| `getCrate()` | `com.sn.crates.model.Crate` | The crate being opened |
| `getKeyType()` | `com.sn.crates.model.KeyType` | The key type spent: `PHYSICAL`, `VIRTUAL` or `PERMISSION` |
| `getSource()` | `com.sn.crates.model.PhysicalCrateBlock` | The physical block the open came from, or `null` for a virtual or menu open |
| `isMassOpen()` | `boolean` | `true` when this open is one crate of a bulk mass-open |

## CrateRewardEvent

Fired exactly once per won reward, at the moment the win is committed: the animation settling, an
early close or a disconnect mid-spin, an instant open, or each win of a mass-open. By the time it
fires the reward has already been granted (the item handed over and/or the commands run).

```java
public class CrateRewardEvent extends org.bukkit.event.Event {
    public CrateRewardEvent(Player player, Crate crate, Reward reward,
                            KeyType keyType, boolean massOpen);
    public Player getPlayer();
    public Crate getCrate();
    public Reward getReward();
    public KeyType getKeyType();
    public boolean isMassOpen();
    public static HandlerList getHandlerList();
}
```

| Method | Type | Returns |
|---|---|---|
| `getPlayer()` | `org.bukkit.entity.Player` | The winning player |
| `getCrate()` | `com.sn.crates.model.Crate` | The crate that was opened |
| `getReward()` | `com.sn.crates.model.Reward` | The reward that was won |
| `getKeyType()` | `com.sn.crates.model.KeyType` | The key type spent on the open that produced this win |
| `isMassOpen()` | `boolean` | `true` when this win is one crate of a bulk mass-open |

## The model types the events expose

Read-only value objects. Their getters report crate state; none of them mutates anything inside
SnCrates.

| Type | What it is | Useful getters |
|---|---|---|
| `com.sn.crates.model.Crate` | A crate definition | `getId()`, `getDisplayName()`, `getRewards()`, `getReward(String id)`, `getAnimationType()`, `getAcceptedKeyTypes()`, `accepts(KeyType)` |
| `com.sn.crates.model.Reward` | One entry of a crate's reward pool | `getId()`, `getWeight()`, `isEnabled()`, `getDisplayItemClone()`, `getAmount()`, `getCommands()`, `isBroadcast()`, `isGiveItem()`, `getPerPlayerLimit()`, `getGlobalLimit()` |
| `com.sn.crates.model.KeyType` | Enum: `PHYSICAL`, `VIRTUAL`, `PERMISSION` | - |
| `com.sn.crates.model.PhysicalCrateBlock` | A crate bound to a block in the world | `getCrateId()`, `getWorld()`, `getX()`, `getY()`, `getZ()` |

`Reward.getDisplayItemClone()` hands back a copy, so mutating it cannot corrupt the crate.

## Listening

Nothing special: they are ordinary Bukkit events.

```java
public final class MyPlugin extends JavaPlugin {
    @Override
    public void onEnable() {
        getServer().getPluginManager().registerEvents(new CrateListener(this), this);
    }
}

public final class CrateListener implements Listener {

    private final MyPlugin plugin;

    public CrateListener(MyPlugin plugin) {
        this.plugin = plugin;
    }

    @EventHandler
    public void onOpen(CrateOpenEvent event) {
        plugin.getLogger().info(event.getPlayer().getName()
                + " opened " + event.getCrate().getId()
                + " (key=" + event.getKeyType() + ", mass=" + event.isMassOpen() + ")");
    }

    @EventHandler
    public void onReward(CrateRewardEvent event) {
        // Announce only the rewards the crate marks as broadcast-worthy, and only for a
        // single open: a mass-open would otherwise spam one line per crate.
        if (event.isMassOpen() || !event.getReward().isBroadcast()) {
            return;
        }
        plugin.getServer().broadcastMessage(event.getPlayer().getName()
                + " won " + event.getReward().getId()
                + " from " + event.getCrate().getId());
    }
}
```

## Guarantees

- **Additions only.** The class names, the packages, the constructors, every getter and every return
  type above are frozen. The API grows by adding, never by renaming or re-signaturing, so a plugin
  compiled against it keeps working.
- **Not cancellable.** Neither event implements `Cancellable`. A listener cannot stop an open, a key
  spend or a reward delivery; they are notifications.
- **`CrateRewardEvent` is exactly once.** Closing the animation, disconnecting mid-spin or the plugin
  disabling mid-spin each deliver the reward once and fire the event once.
- **`getSource()` may be `null`** on `CrateOpenEvent`: a virtual open, a menu open and a mass-open
  have no originating block.
- **Main thread.** Both events are fired on the main thread.
- **Obfuscation.** `com.sn.crates.api.**` is kept unobfuscated, which is also what preserves the
  static `getHandlerList()` Bukkit resolves reflectively, and the model types reachable from these
  signatures keep their names with it.
