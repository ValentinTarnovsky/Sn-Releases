# Commands

Root command `/envoy`, alias `/envoys`.

| Command | Permission | Description |
|---------|-----------|-------------|
| `/envoy` | `snenvoys.use` | Show time remaining until the next envoy event, or the number of unclaimed envoys while one is active |
| `/envoy help` | `snenvoys.use` | Show the generated help menu |
| `/envoy editor` | `snenvoys.admin.editor` | Toggle the location editor: place a diamond block to register a spawn location, break one to remove it |
| `/envoy start` | `snenvoys.admin.start` | Force-start an envoy event now |
| `/envoy stop` | `snenvoys.admin.stop` | Force-end the active envoy event |
| `/envoy drop [here]` | `snenvoys.admin.drop` | Force a Supply Drop now; `here` drops at your own location (feature must be enabled) |
| `/envoy reload` | `snenvoys.admin.reload` | Reload `config.yml` and the language file |
| `/envoy debug` | `snenvoys.admin.debug` | Toggle runtime debug logging |
