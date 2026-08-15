# Installation

## Requirements

| | |
|---|---|
| Java | 21 or newer |
| Server | Paper 1.20.4+ (1.20.x and 1.21.x are both supported) |
| SnLib | 1.27.0 or newer, in `plugins/` |
| Licence | A key in `plugins/.Sn-License/license.yml` |
| Optional | SnClans, for clan-based teams in rooms of more than one player per side |

## Steps

1. Download `SnRooms-x.y.z.jar` from the release tagged `snrooms-v*`.
2. Put it in `plugins/`, alongside `SnLib.jar`.
3. Start the server once. SnRooms creates `plugins/.Sn-License/license.yml` and then stops
   itself with `[Sn] License: FAIL`. **This is expected on a first run.**
4. Open `plugins/.Sn-License/license.yml` and replace `PASTE-YOUR-LICENSE-KEY-HERE` with your
   key.
5. Restart.

### The licence file is shared

Every Sn bundle plugin on the server reads the same `plugins/.Sn-License/license.yml`, so one
key unlocks all of them and step 4 is only ever done once, however many you install.

The check runs once, at startup, and needs outbound HTTPS. If it cannot be completed the plugin
refuses to enable rather than running half-licensed, so `[Sn] License: FAIL` **after** a key is
in place almost always means the server could not reach the licence backend.

## First room

```
/rooms wand
```
Left click one corner of the region, right click the opposite one. The edges are drawn in
particles while you select.

```
/rooms create arena1
```
Turns the selection into a room using the `default-room` template from `config.yml`.

```
/rooms setexit arena1
```
Run it standing where survivors should be sent when a round ends. Optional - without an exit,
survivors are simply left where they are.

```
/rooms set arena1 teams 2
/rooms set arena1 team-size 1
```
Anything the template got wrong. With `teams: 2` and `team-size: 1` the room is a 1v1: it starts
the moment two players are inside.

That is the whole setup. Players never type anything - they walk in.

## Files it creates

| File | Kind | Contents |
|---|---|---|
| `config.yml` | managed | The wand, the room template, teams, round, anti-glitch, shell and feedback settings |
| `lang/messages_en.yml` | managed | Every message and state word, in English |
| `lang/messages_es.yml` | managed | The Spanish translation; select it with `lang: es` |
| `rooms.yml` | data | One section per room |
| `shell-state.yml` | data | Crash net: the blocks a sealed room replaced |

**Managed** files gain the keys a new version adds without losing your values or your comments.
The two **data** files are written by the plugin and never merged.

`shell-state.yml` is normally empty. It only holds entries while a room is actually sealed. If a
room's world is not loaded when the server starts - common when a world manager loads worlds
after plugins - its entry is deliberately kept and the blocks go back once that world is
available, so it is not a file to clear by hand.

## Updating

Replace the jar and restart. New config and language keys are merged in; your values and
comments survive. Set `update-configs: false` to freeze the managed files, after which SnLib
only warns about missing keys instead of inserting them.
