# Configuration

Every configuration file lives on the Velocity proxy, in `plugins/sncredits/`. The Paper backends
read no configuration at all. They draw the menus the proxy describes to them and nothing else.

`/credits reload` re-reads every file below and pushes the new values out to the backends. No
restart is needed for a configuration change.

{% hint style="warning" %}
Each file carries a `config-version` key. When a newer jar raises that number, the plugin renames
your file to `old-<name>-v<version>-<timestamp>.yml` and writes a fresh default in its place. The
files are not merged, so copy your values back into the new file by hand after an update.
{% endhint %}

## config.yml

The proxy-wide settings: currency name, database, economy limits, coinflip bounds, the leaderboard
cache and the Discord webhooks.

```yaml
# ============================================================
#                     SnCredits - Configuration
#                   Proxy-based credits economy
#                       Author: Snopeyy
# ============================================================

# Configuration file version - DO NOT MODIFY
config-version: 6

# ─── General ─────────────────────────────────────────────────

# Enable debug logging to console
debug: false

# ─── Currency ────────────────────────────────────────────────

currency:
  # Display name of the currency (also used as the main command name)
  # Changing this will change the command from /credits to /<name>
  name: "credits"

  # Symbol displayed next to amounts
  symbol: "$"

  # Additional command aliases (the currency name is always the primary command)
  command-aliases:
    - "cr"
    - "creds"

# ─── Database ────────────────────────────────────────────────

database:
  # Database backend to use: mysql or sqlite
  # Use sqlite for small/single-server setups. Use mysql for networks.
  type: "mysql"

  # ── SQLite (used when type: sqlite) ────────────────────────
  sqlite:
    # Path to the SQLite database file, relative to the plugin data folder
    file: "sncredits.db"

  # ── MySQL (used when type: mysql) ──────────────────────────
  host: "localhost"
  port: 3306
  database: "sncredits"
  username: "root"
  password: "password"

  # Connection pool settings (HikariCP)
  # For SQLite, max-pool-size is forced to 1 regardless of this value.
  pool:
    # Maximum number of connections in the pool
    max-pool-size: 10

    # Minimum number of idle connections maintained
    min-idle: 2

    # Maximum time (ms) to wait for a connection from the pool
    connection-timeout: 30000

    # Maximum time (ms) a connection can sit idle before being removed
    idle-timeout: 600000

    # Maximum lifetime (ms) of a connection in the pool
    max-lifetime: 1800000

  # Prefix for all database table names
  table-prefix: "sn_credits_"

# ─── Economy ─────────────────────────────────────────────────

economy:
  # Balance given to new players on first join
  starting-balance: 0

  # Maximum balance a player can hold
  max-balance: 1000000000

# ─── Coinflip ────────────────────────────────────────────────

coinflip:
  # Minimum bet amount for a coinflip
  min-bet: 100

  # Maximum bet amount for a coinflip
  max-bet: 1000000

  # Bets above this amount will be broadcast to all players
  broadcast-threshold: 10000

  # Duration of the coinflip spinning animation in ticks (20 ticks = 1 second)
  animation-duration-ticks: 60

  # Maximum number of active coinflip bets per player
  max-active-coinflips: 1

# ─── Leaderboard ─────────────────────────────────────────────

leaderboard:
  # How often (in seconds) the leaderboard cache is refreshed
  cache-refresh-seconds: 300

  # Maximum number of top players cached from the database.
  # The GUI paginates through these to fill the available slots.
  max-entries: 100

  # Permission node that excludes a player from appearing on the leaderboard.
  # Players with this node, the wildcard "*" node, or LuckPerms-backed op
  # status are hidden. If LuckPerms is not installed, only online players
  # with the configured node at cache-refresh time are excluded.
  exempt-permission: "sncredits.leaderboard.exempt"

# ─── Discord Webhooks ────────────────────────────────────────
# Each event type has its own webhook URL. Leave empty to disable.
# Purchase webhooks are configured per-shop in each shop.yml file.

discord-webhooks:
  # Webhook URLs per event type (leave empty to disable that event)
  coinflip-url: ""
  transfer-url: ""
  give-url: ""
  remove-url: ""
  set-url: ""

  # Minimum amount for give commands to trigger a Discord webhook (0 = always send)
  give-webhook-minimum-amount: 100

  # Shared embed appearance settings
  embed:
    # Embed sidebar color in hex (default: gold)
    color: "#FFD700"
    # Footer text shown on all webhook embeds
    footer: "SnCredits"
    # Thumbnail image URL (leave empty for no thumbnail)
    thumbnail: ""

  # Per-event embed templates using Discord fields
  # Each field has: name, value (with placeholders), inline (true/false)
  templates:
    # Placeholders: {winner}, {loser}, {amount}, {currency}, {date}
    coinflip:
      title: "Coinflip Result"
      fields:
        - { name: "Winner", value: "{winner}", inline: true }
        - { name: "Loser", value: "{loser}", inline: true }
        - { name: "Amount", value: "{amount} {currency}", inline: true }
        - { name: "Date", value: "{date}", inline: true }

    # Placeholders: {sender}, {receiver}, {amount}, {currency}, {date}
    transfer:
      title: "Credit Transfer"
      fields:
        - { name: "Sender", value: "{sender}", inline: true }
        - { name: "Receiver", value: "{receiver}", inline: true }
        - { name: "Amount", value: "{amount} {currency}", inline: true }
        - { name: "Date", value: "{date}", inline: true }

    # Placeholders: {admin}, {player}, {amount}, {currency}, {date}
    give:
      title: "Credits Given"
      fields:
        - { name: "Admin", value: "{admin}", inline: true }
        - { name: "Player", value: "{player}", inline: true }
        - { name: "Amount", value: "{amount} {currency}", inline: true }
        - { name: "Date", value: "{date}", inline: true }

    # Placeholders: {admin}, {player}, {amount}, {currency}, {date}
    remove:
      title: "Credits Removed"
      fields:
        - { name: "Admin", value: "{admin}", inline: true }
        - { name: "Player", value: "{player}", inline: true }
        - { name: "Amount", value: "{amount} {currency}", inline: true }
        - { name: "Date", value: "{date}", inline: true }

    # Placeholders: {admin}, {player}, {amount}, {currency}, {date}
    set:
      title: "Credits Set"
      fields:
        - { name: "Admin", value: "{admin}", inline: true }
        - { name: "Player", value: "{player}", inline: true }
        - { name: "Amount", value: "{amount} {currency}", inline: true }
        - { name: "Date", value: "{date}", inline: true }
```

## messages.yml

Every line a player can see. Colors accept `&` codes and `&#RRGGBB` hex.

```yaml
# ============================================================
#                    SnCredits - Messages
#                  All user-facing messages
#                       Author: Snopeyy
# ============================================================
#
# Supports:
#   - &#RRGGBB hex colors and & color codes
#   - Placeholders: {prefix}, {player}, {amount}, {balance},
#     {currency}, {target}, {rank}, {name}, {winner}, {loser}
#
# ============================================================

# Configuration file version - DO NOT MODIFY
config-version: 11

# Plugin prefix prepended to most messages
prefix: "&#9945FF&lCredits &8» &r"

# ─── Error Messages ─────────────────────────────────────────

errors:
  players-only: "{prefix}&cThis command can only be used by players."
  no-permission: "{prefix}&cYou don't have permission to do that."
  player-not-found: "{prefix}&cPlayer &f{player} &cnot found or has never joined."
  invalid-amount: "{prefix}&cInvalid amount. Please enter a valid number."
  decimal-amount: "{prefix}&cAmounts must be whole numbers. Decimals are not allowed."
  insufficient-funds: "{prefix}&cYou don't have enough {currency}. &7(Balance: &f{balance}&7)"
  self-pay: "{prefix}&cYou cannot send {currency} to yourself."
  data-not-loaded: "{prefix}&cYour data hasn't loaded yet. Please try again in a moment."
  max-balance: "{prefix}&cThat would exceed the maximum balance of &f{amount} {currency}&c."
  command-error: "{prefix}&cAn internal error occurred. Check the proxy console for details."

# ─── Success Messages ───────────────────────────────────────

success:
  balance-self: "{prefix}&7Your balance: &#9945FF{balance} {currency}"
  balance-other: "{prefix}&#9945FF{player}&7's balance: &#9945FF{balance} {currency}"
  pay-sent: "{prefix}&7You sent &#9945FF{amount} {currency} &7to &#9945FF{target}&7."
  pay-received: "{prefix}&#9945FF{player} &7sent you &#9945FF{amount} {currency}&7."
  give: "{prefix}&7Gave &#9945FF{amount} {currency} &7to &#9945FF{player}&7. New balance: &#9945FF{balance}"
  give-received: "{prefix}&7You received &#9945FF{amount} {currency} &7from &#9945FF{player}&7."
  set: "{prefix}&7Set &#9945FF{player}&7's balance to &#9945FF{amount} {currency}&7."
  set-received: "{prefix}&7Your balance has been set to &#9945FF{amount} {currency}&7."
  remove: "{prefix}&7Removed &#9945FF{amount} {currency} &7from &#9945FF{player}&7. New balance: &#9945FF{balance}"
  remove-received: "{prefix}&#9945FF{amount} {currency} &7has been removed from your balance."
  reset: "{prefix}&7Reset &#9945FF{player}&7's balance to &#9945FF{amount} {currency}&7."
  reset-received: "{prefix}&7Your balance has been reset to &#9945FF{amount} {currency}&7."

# ─── Economy ─────────────────────────────────────────────────

economy:
  starting-balance: "{prefix}&7You received &#9945FF{amount} {currency} &7as a starting balance!"

# ─── Coinflip ────────────────────────────────────────────────

coinflip:
  created: "{prefix}&7You created a coinflip for &#9945FF{amount} {currency}&7. Waiting for an opponent..."
  accepted: "{prefix}&7You accepted &#9945FF{player}&7's coinflip for &#9945FF{amount} {currency}&7!"
  won: "{prefix}&#55FF55You won the coinflip! &7+&#9945FF{amount} {currency} &7(Balance: &#9945FF{balance}&7)"
  lost: "{prefix}&#FF5555You lost the coinflip. &7-&#9945FF{amount} {currency} &7(Balance: &#9945FF{balance}&7)"
  cancelled: "{prefix}&7Your coinflip for &#9945FF{amount} {currency} &7has been cancelled."
  max-active: "{prefix}&cYou already have an active coinflip. Cancel it first with &f/credits coinflip cancel&c."
  min-bet: "{prefix}&cMinimum bet is &#9945FF{amount} {currency}&c."
  max-bet: "{prefix}&cMaximum bet is &#9945FF{amount} {currency}&c."
  no-active-self: "{prefix}&7You don't have any active coinflips to cancel."
  insufficient-funds: "{prefix}&cYou need &#9945FF{amount} {currency} &cto accept this coinflip. &7(Balance: &f{balance}&7)"

  # Global announcements (broadcast to all players on all servers)
  announce-creation: "{prefix}&#9945FF{player} &7created a coinflip for &#FFD700{amount} {currency}&7!"
  announce-winner: "{prefix}&#55FF55{winner} &7won &#FFD700{amount} {currency} &7in a coinflip against &#FF5555{loser}&7!"

# ─── Shop ────────────────────────────────────────────────────

shop:
  purchased: "{prefix}&7You purchased &#9945FF{name} &7for &#9945FF{amount} {currency}&7!"
  not-enough: "{prefix}&cYou need &#9945FF{amount} {currency} &cto purchase this item. &7(Balance: &f{balance}&7)"
  announce-purchase: "{prefix}&#9945FF{player} &7purchased &#FFD700{item} &7for &#9945FF{amount} {currency}&7!"
  # Sent when a player clicks an item whose 'requires-any' list is not satisfied
  # on the current backend. {requirements} is the comma-joined list of tokens
  # as authored in the YAML (e.g. "vip" or "ranks/elite").
  requires-not-met: "{prefix}&cYou must first purchase: &f{requirements}&c."

  # ─── Bypass (admin testing mode) ─────────────────────────────
  # Toggled via /credits bypass. While enabled, shop purchases for the toggling
  # admin execute item commands only - no cost, no log, no webhook, no broadcast.
  bypass-enabled: "{prefix}&aBypass mode &lON&a. Shop purchases will skip cost, logs, webhooks and broadcast."
  bypass-disabled: "{prefix}&cBypass mode &lOFF&c. Shop purchases work normally."
  bypass-purchased: "{prefix}&7[BYPASS] Triggered &#9945FF{name}&7 - no cost, no log, no webhook."

# ─── Leaderboard ─────────────────────────────────────────────

leaderboard:
  header: "&8&m─────────────&r &#9945FF&lCredits Top &8&m─────────────"
  entry: " &#FFD700#{rank} &f{player} &8- &#9945FF{balance} {currency}"
  empty: "{prefix}&7No leaderboard data available yet."
  footer: "&8&m──────────────────────────────────────────"

# ─── Commands ────────────────────────────────────────────────

commands:
  help-header: "&8&m─────────────&r &#9945FF&lCredits Help &8&m─────────────"
  help-balance: " &#9945FF/{currency} &7- Check your balance"
  help-pay: " &#9945FF/{currency} pay <player> <amount> &7- Send {currency} to a player"
  help-shop: " &#9945FF/{currency} shop &7- Open the credits shop"
  help-coinflip: " &#9945FF/{currency} coinflip <amount|cancel> &7- Create, view, or cancel coinflips"
  help-top: " &#9945FF/{currency} top &7- View the leaderboard"
  help-admin-header: "&8&m─────────────&r &#FF5555&lAdmin Commands &8&m─────────────"
  help-give: " &#FF5555/{currency} give <player> <amount> &7- Give {currency} to a player"
  help-set: " &#FF5555/{currency} set <player> <amount> &7- Set a player's balance"
  help-remove: " &#FF5555/{currency} remove <player> <amount> &7- Remove {currency} from a player"
  help-reset: " &#FF5555/{currency} reset <player> &7- Reset a player's balance"
  help-reload: " &#FF5555/{currency} reload &7- Reload all configuration files"
  help-history: " &#FF5555/{currency} history <player> [page] &7- View purchase history"
  help-resend: " &#FF5555/{currency} resend <backend> <fromId> &7- Re-execute purchases from ID"
  help-summary: " &#FF5555/{currency} summary <backend> [timeframe] &7- Purchase statistics"
  help-bypass: " &#FF5555/{currency} bypass [on|off] &7- Toggle test-purchase mode (no cost/log/webhook)"
  reload-success: "{prefix}&#55FF55Configuration reloaded successfully."
  unknown-subcommand: "{prefix}&cUnknown subcommand. Use &f/{currency} help &cfor a list of commands."

  # ─── History ────────────────────────────────────────────────
  history-header: "&8&m─────────────&r &#9945FF&lPurchase History: {player} &8&m─────────────"
  history-entry: " &7#{id} &8| &#9945FF{amount} {currency} &8| &7{details} &8| &7{date}"
  history-footer: "&8&m──────────────────────────────────────────────────"
  history-empty: "{prefix}&7No purchase history found for &#9945FF{player}&7."
  history-page-info: "{prefix}&7Page &#9945FF{page}&7/&#9945FF{total_pages} &8| &7Use &#9945FF/{currency} history {player} <page>"

  # ─── Resend ─────────────────────────────────────────────────
  resend-start: "{prefix}&7Starting resend from ID &#9945FF#{id} &7on backend &#9945FF{backend}&7..."
  resend-progress: "{prefix}&7Re-executing: &f#{id} &8- &7{item} &7for &f{player}"
  resend-complete: "{prefix}&#55FF55Resend complete. &7{executed} executed, {queued} queued for offline players."
  resend-no-transactions: "{prefix}&7No matching purchases found from ID &#9945FF#{id} &7on &#9945FF{backend}&7."
  resend-server-not-found: "{prefix}&cBackend server '{backend}' not found."
  resend-queued: "{prefix}&7Queued {count} commands for offline player &#9945FF{player}&7."

  # ─── Summary ────────────────────────────────────────────────
  summary-header: "&8&m─────────────&r &#9945FF&lPurchase Summary: {backend} &8&m─────────────"
  summary-total-purchases: " &7Total Purchases: &#9945FF{count}"
  summary-total-spent: " &7Total Spent: &#9945FF{amount} {currency}"
  summary-top-items-header: " &7Top Items:"
  summary-top-item-entry: "   &#FFD700{rank}. &f{item} &8- &7{count} purchases"
  summary-footer: "&8&m──────────────────────────────────────────────────"
  summary-no-data: "{prefix}&7No purchase data found for the specified criteria."
  summary-invalid-timeframe: "{prefix}&cInvalid timeframe. Use: today, week, month, year, YYYY-MM, or fromId:<id>"
```

## shops/&lt;backend&gt;.yml

Each backend registered on the proxy gets its own shop file, in `plugins/sncredits/shops/`. The
file is named after the server as Velocity knows it, so `survival` becomes `shops/survival.yml`.

The proxy copies the template below into a new file the first time it sees a server. Edit each
copy freely: a different catalogue, different prices and its own purchase webhook per backend.

{% hint style="info" %}
`requires-any` lists the items a player must already own before this one unlocks. Ownership is
counted on the backend the player is standing on. A bare id points inside the same category, and
`<categoryId>/<itemId>` points at another category. An empty or missing list means no prerequisite.
{% endhint %}

```yaml
# ============================================================
#                     SnCredits - Shop
#                  Category & item definitions
#                       Author: Snopeyy
# ============================================================

# Configuration file version - DO NOT MODIFY
config-version: 8

shop:
  # Title displayed at the top of the shop GUI
  # Supports &#RRGGBB hex colors and & color codes
  title: "&#FFD700&lCredits Shop"

  # Announce purchases in chat to all players on this backend
  announce: false

  # Discord webhook for purchase notifications (per-shop)
  # Each shop file can have its own webhook URL, allowing different backends
  # to send purchase notifications to different Discord channels.
  # Placeholders: {player}, {item_id}, {item_name}, {price}, {currency},
  #               {server}, {category}, {date}, {transaction_id},
  #               {balance_before}, {balance_after}
  webhook:
    url: ""
    title: "Shop Purchase"
    fields:
      - { name: "Nickname", value: "{player}", inline: true }
      - { name: "Item", value: "{item_id}", inline: true }
      - { name: "Category", value: "{category}", inline: true }
      - { name: "Purchase Info", value: "───────────────", inline: false }
      - { name: "ID", value: "#{transaction_id}", inline: true }
      - { name: "Date", value: "{date}", inline: true }
      - { name: "Price", value: "{price} {currency}", inline: true }
      - { name: "Balance Before", value: "{balance_before}", inline: true }
      - { name: "Balance After", value: "{balance_after}", inline: true }

  # Number of rows in the shop GUI (1-6)
  rows: 6

  # Filler item to fill empty slots
  filler:
    material: BLACK_STAINED_GLASS_PANE

  # ─── Display-only items (main shop GUI) ──────────────────
  # Informational/decorative items shown in the main shop GUI alongside
  # the category icons. By default they do nothing when clicked; add a
  # 'commands' list to run commands on click (console-dispatched on the
  # backend server where the player clicked).
  #
  # Fields per entry:
  #   slot      - single slot (14), or range/list string ("13-24,27,29-34"),
  #               or YAML list ([1,3,5]). Ranges and lists let one entry
  #               fill several slots with the same icon.
  #   material  - Bukkit material name
  #   name      - display name (supports &codes and &#RRGGBB hex)
  #   lore      - list of lore lines
  #   commands  - (optional) commands to run on click (placeholders:
  #               {player}, %player%, {uuid}, %uuid%)
  display-items:
    info:
      slot: 4
      material: BOOK
      name: "&#FFD700&lInformación"
      lore:
        - "&7Bienvenido a la tienda."
        - "&7Haz click en una categoría para comprar."

  # ─── Categories ──────────────────────────────────────────

  categories:
    # Each category is displayed as an icon in the main shop GUI
    # Players click a category to see the items inside it

    ranks:
      display:
        material: DIAMOND
        name: "&#55FFFF&lRanks"
        lore:
          - "&7Purchase exclusive ranks"
          - "&7to unlock special perks!"
          - ""
          - "&eClick to browse"
      # Slot in the main shop GUI (0-indexed)
      slot: 11
      # Display-only items shown inside this category GUI (optional)
      # Not paginated - always shown on every page.
      display-items:
        header:
          slot: 4
          material: PAPER
          name: "&#55FFFF&lMejoras de rangos"
          lore:
            - "&7Tienda de mejoras de rangos."
      items:
        vip:
          display:
            material: EMERALD
            name: "&#55FF55&lVIP"
            lore:
              - "&7Unlock VIP perks including"
              - "&7cosmetics and priority queue!"
              - ""
              - "&ePrice: &6{price} {currency}"
          slot: 10
          price: 5000
          # Commands executed on purchase ({player} = buyer name)
          commands:
            - "lp user {player} parent set vip"
        mvp:
          display:
            material: DIAMOND
            name: "&#55FFFF&lMVP"
            lore:
              - "&7The ultimate rank with all"
              - "&7perks and exclusive features!"
              - ""
              - "&ePrice: &6{price} {currency}"
          slot: 12
          price: 15000
          commands:
            - "lp user {player} parent set mvp"
          # Optional purchase prerequisites. The player must own at least ONE
          # of the listed items on the SAME backend server to be able to buy
          # this item. Tokens are either:
          #   - bare item id, e.g. "vip"          → resolved within this category
          #   - "<categoryId>/<itemId>"           → cross-category reference
          # Empty list / omitted = no prerequisite.
          requires-any:
            - "vip"
        elite:
          display:
            material: NETHER_STAR
            name: "&#FF5555&lElite"
            lore:
              - "&7The most prestigious rank"
              - "&7with unique abilities!"
              - ""
              - "&ePrice: &6{price} {currency}"
          slot: 14
          price: 30000
          commands:
            - "lp user {player} parent set elite"
          # Owning either VIP or MVP unlocks Elite.
          requires-any:
            - "vip"
            - "mvp"

    items:
      display:
        material: CHEST
        name: "&#FFAA00&lItems"
        lore:
          - "&7Purchase special items"
          - "&7and consumables!"
          - ""
          - "&eClick to browse"
      slot: 15
      items:
        fly_token:
          display:
            material: FEATHER
            name: "&#AAFFFF&lFly Token"
            lore:
              - "&71 hour of creative flight"
              - ""
              - "&ePrice: &6{price} {currency}"
          slot: 10
          price: 2500
          commands:
            - "tempfly give {player} 3600"
        crate_key:
          display:
            material: TRIPWIRE_HOOK
            name: "&#FFD700&lLegendary Key"
            lore:
              - "&7A key to the Legendary Crate"
              - "&7containing rare rewards!"
              - ""
              - "&ePrice: &6{price} {currency}"
          slot: 12
          price: 1000
          commands:
            - "crate give {player} legendary 1"
```

## gui.yml

The layout of the purchase confirmation menu, the shop category pages and the leaderboard.

```yaml
# ============================================================
#                   SnCredits - GUI Config
#              All GUI layout and appearance
#                       Author: Snopeyy
# ============================================================

# Configuration file version - DO NOT MODIFY
config-version: 5

# ─── Confirm Purchase GUI ───────────────────────────────────

confirm-purchase:
  # Title of the confirmation inventory
  title: "&8Confirm Purchase"

  # Number of rows (1-6)
  rows: 3

  # Filler material for empty slots
  filler-material: BLACK_STAINED_GLASS_PANE

  # Confirm button
  confirm:
    slot: 11
    material: GREEN_WOOL
    name: "&a&lConfirm"
    lore:
      - "&7Price: &6{price} {currency}"
      - ""
      - "&aClick to confirm purchase"

  # Item display (center)
  item-display:
    slot: 13
    material: PAPER
    lore:
      - "&7Cost: &6{price} {currency}"

  # Cancel button
  cancel:
    slot: 15
    material: RED_WOOL
    name: "&c&lCancel"
    lore:
      - "&7Click to cancel"

# ─── Shop Category GUI ─────────────────────────────────────

shop-category:
  # Number of rows in the category GUI (1-6).
  # Make sure prev-page / next-page / back-button slots and your item
  # slots all fit within rows * 9 - 1, or they will be clipped.
  rows: 6

  # Filler material for empty slots
  filler-material: BLACK_STAINED_GLASS_PANE

  # Previous page button (only shown when there is a previous page)
  prev-page:
    slot: 45
    material: ARROW
    name: "&ePrevious Page"

  # Next page button (only shown when there is a next page)
  next-page:
    slot: 53
    material: ARROW
    name: "&eNext Page"

  # Back button - returns to the main shop menu without closing the inventory
  back-button:
    slot: 49
    material: BARRIER
    name: "&cBack"
    lore:
      - "&7Return to the shop menu"

# ─── Leaderboard GUI ───────────────────────────────────────

leaderboard:
  # Title of the leaderboard inventory
  title: "&6Credits Leaderboard"

  # Number of rows (1-6). The bottom row is reserved for pagination;
  # the remaining rows hold leaderboard entries.
  rows: 6

  # Filler material for empty slots
  filler-material: BLACK_STAINED_GLASS_PANE

  # Previous page button (only shown when there is a previous page)
  prev-page:
    slot: 45
    material: ARROW
    name: "&ePrevious Page"

  # Next page button (only shown when there is a next page)
  next-page:
    slot: 53
    material: ARROW
    name: "&eNext Page"

  # Page indicator (placeholders: {page}, {total})
  page-indicator:
    slot: 49
    material: PAPER
    name: "&fPage &e{page}&7/&e{total}"

  # Rank color codes (hex or & codes)
  rank-colors:
    1: "&#FFD700"
    2: "&#C0C0C0"
    3: "&#CD7F32"
    default: "&f"

  # Lore for each leaderboard entry head
  entry-lore:
    - "&7Balance: &6{balance} {currency}"
```

## coinflip.yml

The bet list menu, the spinning animation, its sounds and the two global announcements.

```yaml
# ============================================================
#                  SnCredits - Coinflip GUI
#              All coinflip GUI configuration
#                       Author: Snopeyy
# ============================================================

# Configuration file version - DO NOT MODIFY
config-version: 2

# ─── Coinflip List GUI ──────────────────────────────────────

gui:
  list:
    # Title of the coinflip list inventory
    title: "&6Coinflip Bets"

    # Number of rows (1-6)
    rows: 6

    # Filler material for empty slots
    filler-material: BLACK_STAINED_GLASS_PANE

    # Create coinflip button
    create-button:
      slot: 49
      material: GOLD_INGOT
      name: "&6Create Coinflip"
      lore:
        - "&7Click to create a new coinflip"
        - "&7Use: &e/{currency} coinflip <amount>"

    # Display name format for each coinflip entry head
    entry-name: "&6{player}"

    # Lore for each coinflip entry head
    entry-lore:
      - "&7Wager: &6{amount} {currency}"
      - ""
      - "&eClick to accept this coinflip!"

  # ─── Coinflip Animation GUI ─────────────────────────────────

  animation:
    # Title of the animation inventory
    title: "&6&lCoinflip!"

    # Number of rows (1-6)
    rows: 3

    # Center slot where heads spin
    center-slot: 13

    # Total animation ticks before reveal
    max-ticks: 60

    # Ticks to show the winner before closing
    reveal-delay-ticks: 40

    # Base interval for the animation task (in ticks)
    base-interval-ticks: 2

    # Glass pane colors during spinning phase
    pane-color-1: RED_STAINED_GLASS_PANE
    pane-color-2: BLUE_STAINED_GLASS_PANE

    # Glass pane color for reveal phase (viewer won)
    win-pane: LIME_STAINED_GLASS_PANE

    # Glass pane color for reveal phase (viewer lost)
    lose-pane: RED_STAINED_GLASS_PANE

    # Winner head display name format
    winner-name: "&a&lWINNER! {player}"

    # Winner head lore
    winner-lore:
      - "&7Won: &6{amount} {currency}"

# ─── Sounds ─────────────────────────────────────────────────
# Bukkit sound names. Leave empty to disable a sound.

sounds:
  # Played each tick during the animation spin
  tick: "BLOCK_NOTE_BLOCK_HAT"
  # Played on reveal for the winner
  win: "ENTITY_PLAYER_LEVELUP"
  # Played on reveal for the loser
  lose: "BLOCK_NOTE_BLOCK_BASS"

# ─── Global Announcements ───────────────────────────────────

announcements:
  # Broadcast when a coinflip is created
  creation:
    enabled: true

  # Broadcast when a coinflip has a winner
  winner:
    enabled: true
```
