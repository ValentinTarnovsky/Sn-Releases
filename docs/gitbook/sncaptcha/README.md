# SnCaptcha

SnCaptcha is a time-based anti-macro captcha for EdTools farming worlds. It measures how long each player actually spends farming, and once that total crosses a threshold drawn privately for them, it asks them to prove they are at the keyboard.

The challenge is a menu of four number heads. The player runs `/captcha` and clicks them in ascending order. Failing to answer in time, or clicking out of order too often, runs a ladder of console commands you configure.

## Features

- **Accumulated farming time is the only trigger.** No behavioural scoring, no pattern analysis, no response-timing guesswork, so there is nothing to tune and nothing to produce a false accusation.
- **A private threshold per player**, drawn from a range you set and re-drawn after every captcha. Two players farming side by side are never asked at once, and nobody can learn the interval by timing it.
- **Only players who are actually farming are tested.** Accrued time never decays, so the plugin refuses to challenge somebody who has stopped.
- **The challenge cannot be answered in chat**, by design: a proxy-level mute would stop a chat answer from ever reaching your server. Inventory clicks and commands are the only channels.
- **The heads carry no readable data.** No name, no lore, no hidden tags. Which digit sits in which cell and which skin is drawn are both re-randomized on every open, so a generic auto-clicker has nothing in the menu to read.
- **A configurable sanction ladder** by failure count, with an optional decay so an old failure is forgiven.
- **Staff alerts** in game and to a log file, for a captcha sent, unanswered, failed or solved.
- **It refuses to challenge anybody when it cannot present a solvable board**, and says why in the console. A broken menu file or an unreadable head texture never turns into a punishment.

## Optional integrations

- **WorldGuard**: lets you list regions where the farming timer pauses and no captcha fires. Without WorldGuard the region list is simply ignored and nothing is exempt.
- **Floodgate**: identifies Bedrock players, who are never sent a captcha. Without Floodgate the plugin falls back to the Bedrock UUID prefix, and any failure to detect is treated as "not Bedrock".

## Links

- Source and releases: part of the shared [Sn-Releases](https://github.com/ValentinTarnovsky/Sn-Releases) repo
