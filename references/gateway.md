# OpenClaw Gateway Documentation

## Multiple gateways - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/multiple-gateways

[Skip to main content](https://docs.openclaw.ai/gateway/multiple-gateways#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Scaling and operations

Multiple gateways

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Best recommended setup](https://docs.openclaw.ai/gateway/multiple-gateways#best-recommended-setup)
- [Rescue-Bot Quickstart](https://docs.openclaw.ai/gateway/multiple-gateways#rescue-bot-quickstart)
- [Why this works](https://docs.openclaw.ai/gateway/multiple-gateways#why-this-works)
- [What --profile rescue onboard Changes](https://docs.openclaw.ai/gateway/multiple-gateways#what-profile-rescue-onboard-changes)
- [General multi-gateway setup](https://docs.openclaw.ai/gateway/multiple-gateways#general-multi-gateway-setup)
- [Isolation checklist](https://docs.openclaw.ai/gateway/multiple-gateways#isolation-checklist)
- [Port mapping (derived)](https://docs.openclaw.ai/gateway/multiple-gateways#port-mapping-derived)
- [Browser/CDP notes (common footgun)](https://docs.openclaw.ai/gateway/multiple-gateways#browser%2Fcdp-notes-common-footgun)
- [Manual env example](https://docs.openclaw.ai/gateway/multiple-gateways#manual-env-example)
- [Quick checks](https://docs.openclaw.ai/gateway/multiple-gateways#quick-checks)
- [Related](https://docs.openclaw.ai/gateway/multiple-gateways#related)

Most setups should use one Gateway because a single Gateway can handle multiple messaging connections and agents. If you need stronger isolation or redundancy (e.g., a rescue bot), run separate Gateways with isolated profiles/ports.

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#best-recommended-setup)  Best recommended setup

For most users, the simplest rescue-bot setup is:

- keep the main bot on the default profile
- run the rescue bot on `--profile rescue`
- use a completely separate Telegram bot for the rescue account
- keep the rescue bot on a different base port such as `19789`

This keeps the rescue bot isolated from the main bot so it can debug or apply
config changes if the primary bot is down. Leave at least 20 ports between
base ports so the derived browser/canvas/CDP ports never collide.

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#rescue-bot-quickstart)  Rescue-Bot Quickstart

Use this as the default path unless you have a strong reason to do something
else:

```
# Rescue bot (separate Telegram bot, separate profile, port 19789)
openclaw --profile rescue onboard
openclaw --profile rescue gateway install --port 19789
```

If your main bot is already running, that is usually all you need.During `openclaw --profile rescue onboard`:

- use the separate Telegram bot token
- keep the `rescue` profile
- use a base port at least 20 higher than the main bot
- accept the default rescue workspace unless you already manage one yourself

If onboarding already installed the rescue service for you, the final
`gateway install` is not needed.

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#why-this-works)  Why this works

The rescue bot stays independent because it has its own:

- profile/config
- state directory
- workspace
- base port (plus derived ports)
- Telegram bot token

For most setups, use a completely separate Telegram bot for the rescue profile:

- easy to keep operator-only
- separate bot token and identity
- independent from the main bot’s channel/app install
- simple DM-based recovery path when the main bot is broken

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#what-profile-rescue-onboard-changes)  What `--profile rescue onboard` Changes

`openclaw --profile rescue onboard` uses the normal onboarding flow, but it
writes everything into a separate profile.In practice, that means the rescue bot gets its own:

- config file
- state directory
- workspace (by default `~/.openclaw/workspace-rescue`)
- managed service name

The prompts are otherwise the same as normal onboarding.

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#general-multi-gateway-setup)  General multi-gateway setup

The rescue-bot layout above is the easiest default, but the same isolation
pattern works for any pair or group of Gateways on one host.For a more general setup, give each extra Gateway its own named profile and its
own base port:

```
# main (default profile)
openclaw setup
openclaw gateway --port 18789

# extra gateway
openclaw --profile ops setup
openclaw --profile ops gateway --port 19789
```

If you want both Gateways to use named profiles, that also works:

```
openclaw --profile main setup
openclaw --profile main gateway --port 18789

openclaw --profile ops setup
openclaw --profile ops gateway --port 19789
```

Services follow the same pattern:

```
openclaw gateway install
openclaw --profile ops gateway install --port 19789
```

Use the rescue-bot quickstart when you want a fallback operator lane. Use the
general profile pattern when you want multiple long-lived Gateways for
different channels, tenants, workspaces, or operational roles.

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#isolation-checklist)  Isolation checklist

Keep these unique per Gateway instance:

- `OPENCLAW_CONFIG_PATH` — per-instance config file
- `OPENCLAW_STATE_DIR` — per-instance sessions, creds, caches
- `agents.defaults.workspace` — per-instance workspace root
- `gateway.port` (or `--port`) — unique per instance
- derived browser/canvas/CDP ports

If these are shared, you will hit config races and port conflicts.

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#port-mapping-derived)  Port mapping (derived)

Base port = `gateway.port` (or `OPENCLAW_GATEWAY_PORT` / `--port`).

- browser control service port = base + 2 (loopback only)
- canvas host is served on the Gateway HTTP server (same port as `gateway.port`)
- Browser profile CDP ports auto-allocate from `browser.controlPort + 9 .. + 108`

If you override any of these in config or env, you must keep them unique per instance.

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#browser/cdp-notes-common-footgun)  Browser/CDP notes (common footgun)

- Do **not** pin `browser.cdpUrl` to the same values on multiple instances.
- Each instance needs its own browser control port and CDP range (derived from its gateway port).
- If you need explicit CDP ports, set `browser.profiles.<name>.cdpPort` per instance.
- Remote Chrome: use `browser.profiles.<name>.cdpUrl` (per profile, per instance).

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#manual-env-example)  Manual env example

```
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \\
OPENCLAW_STATE_DIR=~/.openclaw \\
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \\
OPENCLAW_STATE_DIR=~/.openclaw-rescue \\
openclaw gateway --port 19789
```

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#quick-checks)  Quick checks

```
openclaw gateway status --deep
openclaw --profile rescue gateway status --deep
openclaw --profile rescue gateway probe
openclaw status
openclaw --profile rescue status
openclaw --profile rescue browser status
```

Interpretation:

- `gateway status --deep` helps catch stale launchd/systemd/schtasks services from older installs.
- `gateway probe` warning text such as `multiple reachable gateways detected` is expected only when you intentionally run more than one isolated gateway.

## [​](https://docs.openclaw.ai/gateway/multiple-gateways\\#related)  Related

- [Gateway runbook](https://docs.openclaw.ai/gateway)
- [Gateway lock](https://docs.openclaw.ai/gateway/gateway-lock)
- [Configuration](https://docs.openclaw.ai/gateway/configuration)

[Background exec and process tool](https://docs.openclaw.ai/gateway/background-process) [Security](https://docs.openclaw.ai/gateway/security)

Ctrl+I

---

## Heartbeat
**Source:** https://docs.openclaw.ai/gateway/heartbeat

[Skip to main content](https://docs.openclaw.ai/gateway/heartbeat#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Health and diagnostics

Heartbeat

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start (beginner)](https://docs.openclaw.ai/gateway/heartbeat#quick-start-beginner)
- [Defaults](https://docs.openclaw.ai/gateway/heartbeat#defaults)
- [What the heartbeat prompt is for](https://docs.openclaw.ai/gateway/heartbeat#what-the-heartbeat-prompt-is-for)
- [Response contract](https://docs.openclaw.ai/gateway/heartbeat#response-contract)
- [Config](https://docs.openclaw.ai/gateway/heartbeat#config)
- [Scope and precedence](https://docs.openclaw.ai/gateway/heartbeat#scope-and-precedence)
- [Per-agent heartbeats](https://docs.openclaw.ai/gateway/heartbeat#per-agent-heartbeats)
- [Active hours example](https://docs.openclaw.ai/gateway/heartbeat#active-hours-example)
- [24/7 setup](https://docs.openclaw.ai/gateway/heartbeat#24%2F7-setup)
- [Multi account example](https://docs.openclaw.ai/gateway/heartbeat#multi-account-example)
- [Field notes](https://docs.openclaw.ai/gateway/heartbeat#field-notes)
- [Delivery behavior](https://docs.openclaw.ai/gateway/heartbeat#delivery-behavior)
- [Visibility controls](https://docs.openclaw.ai/gateway/heartbeat#visibility-controls)
- [What each flag does](https://docs.openclaw.ai/gateway/heartbeat#what-each-flag-does)
- [Per-channel vs per-account examples](https://docs.openclaw.ai/gateway/heartbeat#per-channel-vs-per-account-examples)
- [Common patterns](https://docs.openclaw.ai/gateway/heartbeat#common-patterns)
- [HEARTBEAT.md (optional)](https://docs.openclaw.ai/gateway/heartbeat#heartbeat-md-optional)
- [tasks: blocks](https://docs.openclaw.ai/gateway/heartbeat#tasks-blocks)
- [Can the agent update HEARTBEAT.md?](https://docs.openclaw.ai/gateway/heartbeat#can-the-agent-update-heartbeat-md)
- [Manual wake (on-demand)](https://docs.openclaw.ai/gateway/heartbeat#manual-wake-on-demand)
- [Reasoning delivery (optional)](https://docs.openclaw.ai/gateway/heartbeat#reasoning-delivery-optional)
- [Cost awareness](https://docs.openclaw.ai/gateway/heartbeat#cost-awareness)
- [Related](https://docs.openclaw.ai/gateway/heartbeat#related)

> **Heartbeat vs Cron?** See [Automation & Tasks](https://docs.openclaw.ai/automation) for guidance on when to use each.

Heartbeat runs **periodic agent turns** in the main session so the model can
surface anything that needs attention without spamming you.Heartbeat is a scheduled main-session turn — it does **not** create [background task](https://docs.openclaw.ai/automation/tasks) records.
Task records are for detached work (ACP runs, subagents, isolated cron jobs).Troubleshooting: [Scheduled Tasks](https://docs.openclaw.ai/automation/cron-jobs#troubleshooting)

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#quick-start-beginner)  Quick start (beginner)

1. Leave heartbeats enabled (default is `30m`, or `1h` for Anthropic OAuth/token auth, including Claude CLI reuse) or set your own cadence.
2. Create a tiny `HEARTBEAT.md` checklist or `tasks:` block in the agent workspace (optional but recommended).
3. Decide where heartbeat messages should go (`target: \"none\"` is the default; set `target: \"last\"` to route to the last contact).
4. Optional: enable heartbeat reasoning delivery for transparency.
5. Optional: use lightweight bootstrap context if heartbeat runs only need `HEARTBEAT.md`.
6. Optional: enable isolated sessions to avoid sending full conversation history each heartbeat.
7. Optional: restrict heartbeats to active hours (local time).

Example config:

```
{
  agents: {
    defaults: {
      heartbeat: {
        every: \"30m\",
        target: \"last\", // explicit delivery to last contact (default is \"none\")
        directPolicy: \"allow\", // default: allow direct/DM targets; set \"block\" to suppress
        lightContext: true, // optional: only inject HEARTBEAT.md from bootstrap files
        isolatedSession: true, // optional: fresh session each run (no conversation history)
        // activeHours: { start: \"08:00\", end: \"24:00\" },
        // includeReasoning: true, // optional: send separate `Reasoning:` message too
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#defaults)  Defaults

- Interval: `30m` (or `1h` when Anthropic OAuth/token auth is the detected auth mode, including Claude CLI reuse). Set `agents.defaults.heartbeat.every` or per-agent `agents.list[].heartbeat.every`; use `0m` to disable.
- Prompt body (configurable via `agents.defaults.heartbeat.prompt`):
`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
- The heartbeat prompt is sent **verbatim** as the user message. The system
prompt includes a “Heartbeat” section only when heartbeats are enabled for the
default agent, and the run is flagged internally.
- When heartbeats are disabled with `0m`, normal runs also omit `HEARTBEAT.md`
from bootstrap context so the model does not see heartbeat-only instructions.
- Active hours (`heartbeat.activeHours`) are checked in the configured timezone.
Outside the window, heartbeats are skipped until the next tick inside the window.

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#what-the-heartbeat-prompt-is-for)  What the heartbeat prompt is for

The default prompt is intentionally broad:

- **Background tasks**: “Consider outstanding tasks” nudges the agent to review
follow-ups (inbox, calendar, reminders, queued work) and surface anything urgent.
- **Human check-in**: “Checkup sometimes on your human during day time” nudges an
occasional lightweight “anything you need?” message, but avoids night-time spam
by using your configured local timezone (see [/concepts/timezone](https://docs.openclaw.ai/concepts/timezone)).

Heartbeat can react to completed [background tasks](https://docs.openclaw.ai/automation/tasks), but a heartbeat run itself does not create a task record.If you want a heartbeat to do something very specific (e.g. “check Gmail PubSub
stats” or “verify gateway health”), set `agents.defaults.heartbeat.prompt` (or
`agents.list[].heartbeat.prompt`) to a custom body (sent verbatim).

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#response-contract)  Response contract

- If nothing needs attention, reply with **`HEARTBEAT_OK`**.
- During heartbeat runs, OpenClaw treats `HEARTBEAT_OK` as an ack when it appears
at the **start or end** of the reply. The token is stripped and the reply is
dropped if the remaining content is **≤ `ackMaxChars`** (default: 300).
- If `HEARTBEAT_OK` appears in the **middle** of a reply, it is not treated
specially.
- For alerts, **do not** include `HEARTBEAT_OK`; return only the alert text.

Outside heartbeats, stray `HEARTBEAT_OK` at the start/end of a message is stripped
and logged; a message that is only `HEARTBEAT_OK` is dropped.

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#config)  Config

```
{
  agents: {
    defaults: {
      heartbeat: {
        every: \"30m\", // default: 30m (0m disables)
        model: \"anthropic/claude-opus-4-6\",
        includeReasoning: false, // default: false (deliver separate Reasoning: message when available)
        lightContext: false, // default: false; true keeps only HEARTBEAT.md from workspace bootstrap files
        isolatedSession: false, // default: false; true runs each heartbeat in a fresh session (no conversation history)
        target: \"last\", // default: none | options: last | none | <channel id> (core or plugin, e.g. \"bluebubbles\")
        to: \"+15551234567\", // optional channel-specific override
        accountId: \"ops-bot\", // optional multi-account channel id
        prompt: \"Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.\",
        ackMaxChars: 300, // max chars allowed after HEARTBEAT_OK
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/heartbeat\\#scope-and-precedence)  Scope and precedence

- `agents.defaults.heartbeat` sets global heartbeat behavior.
- `agents.list[].heartbeat` merges on top; if any agent has a `heartbeat` block, **only those agents** run heartbeats.
- `channels.defaults.heartbeat` sets visibility defaults for all channels.
- `channels.<channel>.heartbeat` overrides channel defaults.
- `channels.<channel>.accounts.<id>.heartbeat` (multi-account channels) overrides per-channel settings.

### [​](https://docs.openclaw.ai/gateway/heartbeat\\#per-agent-heartbeats)  Per-agent heartbeats

If any `agents.list[]` entry includes a `heartbeat` block, **only those agents**
run heartbeats. The per-agent block merges on top of `agents.defaults.heartbeat`
(so you can set shared defaults once and override per agent).Example: two agents, only the second agent runs heartbeats.

```
{
  agents: {
    defaults: {
      heartbeat: {
        every: \"30m\",
        target: \"last\", // explicit delivery to last contact (default is \"none\")
      },
    },
    list: [\\\n      { id: \"main\", default: true },\\\n      {\\\n        id: \"ops\",\\\n        heartbeat: {\\\n          every: \"1h\",\\\n          target: \"whatsapp\",\\\n          to: \"+15551234567\",\\\n          timeoutSeconds: 45,\\\n          prompt: \"Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.\",\\\n        },\\\n      },\\\n    ],
  },
}
```

### [​](https://docs.openclaw.ai/gateway/heartbeat\\#active-hours-example)  Active hours example

Restrict heartbeats to business hours in a specific timezone:

```
{
  agents: {
    defaults: {
      heartbeat: {
        every: \"30m\",
        target: \"last\", // explicit delivery to last contact (default is \"none\")
        activeHours: {
          start: \"09:00\",
          end: \"22:00\",
          timezone: \"America/New_York\", // optional; uses your userTimezone if set, otherwise host tz
        },
      },
    },
  },
}
```

Outside this window (before 9am or after 10pm Eastern), heartbeats are skipped. The next scheduled tick inside the window will run normally.

### [​](https://docs.openclaw.ai/gateway/heartbeat\\#24/7-setup)  24/7 setup

If you want heartbeats to run all day, use one of these patterns:

- Omit `activeHours` entirely (no time-window restriction; this is the default behavior).
- Set a full-day window: `activeHours: { start: \"00:00\", end: \"24:00\" }`.

Do not set the same `start` and `end` time (for example `08:00` to `08:00`).
That is treated as a zero-width window, so heartbeats are always skipped.

### [​](https://docs.openclaw.ai/gateway/heartbeat\\#multi-account-example)  Multi account example

Use `accountId` to target a specific account on multi-account channels like Telegram:

```
{
  agents: {
    list: [\\\n      {\\\n        id: \"ops\",\\\n        heartbeat: {\\\n          every: \"1h\",\\\n          target: \"telegram\",\\\n          to: \"12345678:topic:42\", // optional: route to a specific topic/thread\\\n          accountId: \"ops-bot\",\\\n        },\\\n      },\\\n    ],
  },
  channels: {
    telegram: {
      accounts: {
        \"ops-bot\": { botToken: \"YOUR_TELEGRAM_BOT_TOKEN\" },
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/heartbeat\\#field-notes)  Field notes

- `every`: heartbeat interval (duration string; default unit = minutes).
- `model`: optional model override for heartbeat runs (`provider/model`).
- `includeReasoning`: when enabled, also deliver the separate `Reasoning:` message when available (same shape as `/reasoning on`).
- `lightContext`: when true, heartbeat runs use lightweight bootstrap context and keep only `HEARTBEAT.md` from workspace bootstrap files.
- `isolatedSession`: when true, each heartbeat runs in a fresh session with no prior conversation history. Uses the same isolation pattern as cron `sessionTarget: \"isolated\"`. Dramatically reduces per-heartbeat token cost. Combine with `lightContext: true` for maximum savings. Delivery routing still uses the main session context.
- `session`: optional session key for heartbeat runs.

  - `main` (default): agent main session.
  - Explicit session key (copy from `openclaw sessions --json` or the [sessions CLI](https://docs.openclaw.ai/cli/sessions)).
  - Session key formats: see [Sessions](https://docs.openclaw.ai/concepts/session) and [Groups](https://docs.openclaw.ai/channels/groups).
- `target`:

  - `last`: deliver to the last used external channel.
  - explicit channel: any configured channel or plugin id, for example `discord`, `matrix`, `telegram`, or `whatsapp`.
  - `none` (default): run the heartbeat but **do not deliver** externally.
- `directPolicy`: controls direct/DM delivery behavior:

  - `allow` (default): allow direct/DM heartbeat delivery.
  - `block`: suppress direct/DM delivery (`reason=dm-blocked`).
- `to`: optional recipient override (channel-specific id, e.g. E.164 for WhatsApp or a Telegram chat id). For Telegram topics/threads, use `<chatId>:topic:<messageThreadId>`.
- `accountId`: optional account id for multi-account channels. When `target: \"last\"`, the account id applies to the resolved last channel if it supports accounts; otherwise it is ignored. If the account id does not match a configured account for the resolved channel, delivery is skipped.
- `prompt`: overrides the default prompt body (not merged).
- `ackMaxChars`: max chars allowed after `HEARTBEAT_OK` before delivery.
- `suppressToolErrorWarnings`: when true, suppresses tool error warning payloads during heartbeat runs.
- `activeHours`: restricts heartbeat runs to a time window. Object with `start` (HH:MM, inclusive; use `00:00` for start-of-day), `end` (HH:MM exclusive; `24:00` allowed for end-of-day), and optional `timezone`.

  - Omitted or `\"user\"`: uses your `agents.defaults.userTimezone` if set, otherwise falls back to the host system timezone.
  - `\"local\"`: always uses the host system timezone.
  - Any IANA identifier (e.g. `America/New_York`): used directly; if invalid, falls back to the `\"user\"` behavior above.
  - `start` and `end` must not be equal for an active window; equal values are treated as zero-width (always outside the window).
  - Outside the active window, heartbeats are skipped until the next tick inside the window.

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#delivery-behavior)  Delivery behavior

- Heartbeats run in the agent’s main session by default (`agent:<id>:<mainKey>`),\nor `global` when `session.scope = \"global\"`. Set `session` to override to a\nspecific channel session (Discord/WhatsApp/etc.).
- `session` only affects the run context; delivery is controlled by `target` and `to`.
- To deliver to a specific channel/recipient, set `target` \\+ `to`. With\n`target: \"last\"`, delivery uses the last external channel for that session.
- Heartbeat deliveries allow direct/DM targets by default. Set `directPolicy: \"block\"` to suppress direct-target sends while still running the heartbeat turn.
- If the main queue is busy, the heartbeat is skipped and retried later.
- If `target` resolves to no external destination, the run still happens but no\noutbound message is sent.
- If `showOk`, `showAlerts`, and `useIndicator` are all disabled, the run is skipped up front as `reason=alerts-disabled`.
- If only alert delivery is disabled, OpenClaw can still run the heartbeat, update due-task timestamps, restore the session idle timestamp, and suppress the outward alert payload.
- If the resolved heartbeat target supports typing, OpenClaw shows typing while\nthe heartbeat run is active. This uses the same target the heartbeat would\nsend chat output to, and it is disabled by `typingMode: \"never\"`.
- Heartbeat-only replies do **not** keep the session alive. Heartbeat metadata\nmay update the session row, but idle expiry uses `lastInteractionAt` from the\nlast real user/channel message, and daily expiry uses `sessionStartedAt`.
- Control UI and WebChat history hide heartbeat prompts and OK-only\nacknowledgments. The underlying session transcript can still contain those\nturns for audit/replay.
- Detached [background tasks](https://docs.openclaw.ai/automation/tasks) can enqueue a system event and wake heartbeat when the main session should notice something quickly. That wake does not make the heartbeat run a background task.

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#visibility-controls)  Visibility controls

By default, `HEARTBEAT_OK` acknowledgments are suppressed while alert content is\ndelivered. You can adjust this per channel or per account:\n
```
channels:
  defaults:
    heartbeat:
      showOk: false # Hide HEARTBEAT_OK (default)
      showAlerts: true # Show alert messages (default)
      useIndicator: true # Emit indicator events (default)
  telegram:
    heartbeat:
      showOk: true # Show OK acknowledgments on Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Suppress alert delivery for this account
```

Precedence: per-account → per-channel → channel defaults → built-in defaults.

### [​](https://docs.openclaw.ai/gateway/heartbeat\\#what-each-flag-does)  What each flag does

- `showOk`: sends a `HEARTBEAT_OK` acknowledgment when the model returns an OK-only reply.
- `showAlerts`: sends the alert content when the model returns a non-OK reply.
- `useIndicator`: emits indicator events for UI status surfaces.

If **all three** are false, OpenClaw skips the heartbeat run entirely (no model call).

### [​](https://docs.openclaw.ai/gateway/heartbeat\\#per-channel-vs-per-account-examples)  Per-channel vs per-account examples

```
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # all Slack accounts
    accounts:
      ops:
        heartbeat:
          showAlerts: false # suppress alerts for the ops account only
  telegram:
    heartbeat:
      showOk: true
```

### [​](https://docs.openclaw.ai/gateway/heartbeat\\#common-patterns)  Common patterns

| Goal | Config |
| --- | --- |
| Default behavior (silent OKs, alerts on) | _(no config needed)_ |
| Fully silent (no messages, no indicator) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Always send OKs | `channels.defaults.heartbeat: { showOk: true }` |
| Always send alerts | `channels.defaults.heartbeat: { showAlerts: true }` |
| Always send indicator | `channels.defaults.heartbeat: { useIndicator: true }` |
| Disable all heartbeats | `agents.defaults.heartbeat.every: \"0m\"` |
| Per-agent overrides | `agents.list[].heartbeat` (merges on top of defaults) |
| Per-channel overrides | `channels.<channel>.heartbeat` (merges on top of defaults) |
| Per-account overrides | `channels.<channel>.accounts.<id>.heartbeat` (merges on top of channel) |
| Only run heartbeats for specific agents | Set `heartbeat` block on `agents.list[]` entries; omit from `agents.defaults` |
| Only run heartbeats for specific channels | Set `heartbeat` block on `channels.<channel>` entries; omit from `channels.defaults` |
| Only run heartbeats for specific accounts | Set `heartbeat` block on `channels.<channel>.accounts.<id>` entries; omit from `channels.<channel>` |
| Restrict to active hours | `agents.defaults.heartbeat.activeHours: { start: \"09:00\", end: \"17:00\", timezone: \"America/New_York\" }` |
| Lightweight context (only HEARTBEAT.md) | `agents.defaults.heartbeat.lightContext: true` |
| Isolated sessions (no conversation history) | `agents.defaults.heartbeat.isolatedSession: true` |
| Deliver to last contact | `agents.defaults.heartbeat.target: \"last\"` |
| Deliver to specific channel | `agents.defaults.heartbeat.target: \"discord\"` |
| Deliver to specific recipient | `agents.defaults.heartbeat.target: \"whatsapp\", to: \"+15551234567\"` |
| Deliver to specific account | `agents.defaults.heartbeat.target: \"telegram\", accountId: \"ops-bot\"` |
| Custom prompt | `agents.defaults.heartbeat.prompt: \"Check the server logs and report any errors.\"` |
| Suppress tool error warnings | `agents.defaults.heartbeat.suppressToolErrorWarnings: true` |

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#heartbeat-md-optional)  HEARTBEAT.md (optional)

If a `HEARTBEAT.md` file exists in the agent’s workspace (or any bootstrap file), it is injected into the heartbeat run context. This is useful for providing a dynamic checklist or instructions to the agent during its periodic turns.The agent is instructed to follow `HEARTBEAT.md` strictly. This means it should not infer or repeat old tasks from prior chats, but rather focus on the current instructions in the `HEARTBEAT.md` file.If `lightContext: true` is set, only `HEARTBEAT.md` from bootstrap files is injected, significantly reducing token cost.

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#tasks-blocks)  tasks: blocks

`HEARTBEAT.md` can contain `tasks:` blocks, which are parsed and presented to the agent as a structured list of tasks. This allows for more precise control over what the agent should be doing during its heartbeat turns.Example `HEARTBEAT.md`:

```markdown
# Heartbeat Checklist

tasks:
- Check for new emails in the \"urgent\" inbox.
- Review the latest server logs for errors.
- Summarize any critical alerts from the past hour.
- If no critical issues, reply HEARTBEAT_OK.
```

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#can-the-agent-update-heartbeat-md)  Can the agent update HEARTBEAT.md?

Yes, the agent can update `HEARTBEAT.md` during a heartbeat run. This allows for dynamic adjustments to the agent’s periodic tasks. For example, if the agent completes a task, it can mark it as done in `HEARTBEAT.md` so it doesn’t repeat it in the next heartbeat.This is particularly useful for long-running, iterative tasks that require periodic check-ins and updates.

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#manual-wake-on-demand)  Manual wake (on-demand)

To manually trigger a heartbeat run for an agent, use the `openclaw agent heartbeat <agent-id>` CLI command. This is useful for testing configurations or for immediate check-ins outside the scheduled cadence.

## [​](https://openclaw.ai/gateway/heartbeat\\#reasoning-delivery-optional)  Reasoning delivery (optional)

If `includeReasoning: true` is set in the heartbeat config, OpenClaw will also deliver a separate `Reasoning:` message when available. This provides transparency into the agent’s thought process during its heartbeat turns, similar to the `/reasoning on` command.This is useful for debugging or for understanding why the agent took certain actions or made specific observations.

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#cost-awareness)  Cost awareness

Heartbeat runs contribute to token usage. To minimize costs:

- Use `lightContext: true` to inject only `HEARTBEAT.md` from bootstrap files.
- Use `isolatedSession: true` to run each heartbeat in a fresh session, avoiding the cost of sending full conversation history.
- Keep `HEARTBEAT.md` concise.
- Adjust `every` interval to run less frequently if appropriate.

## [​](https://docs.openclaw.ai/gateway/heartbeat\\#related)  Related

- [Automation & Tasks](https://docs.openclaw.ai/automation)
- [Cron Jobs](https://docs.openclaw.ai/automation/cron-jobs)
- [Sessions](https://docs.openclaw.ai/concepts/session)
- [Groups](https://docs.openclaw.ai/channels/groups)
- [Timezone](https://docs.openclaw.ai/concepts/timezone)
- [CLI](https://docs.openclaw.ai/cli)

---

## Security audit checks - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/security/audit-checks

[Skip to main content](https://docs.openclaw.ai/gateway/security/audit-checks#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c652313385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Security and sandboxing

Security audit checks

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Related](https://docs.openclaw.ai/gateway/security/audit-checks#related)

`openclaw security audit` emits structured findings keyed by `checkId`. This
page is the reference catalog for those IDs. For the high-level threat model
and hardening guidance, see [Security](https://docs.openclaw.ai/gateway/security).High-signal `checkId` values you will most likely see in real deployments (not
exhaustive):

| `checkId` | Severity | Why it matters | Primary fix key/path | Auto-fix |
| --- | --- | --- | --- | --- |
| `fs.state_dir.perms_world_writable` | critical | Other users/processes can modify full OpenClaw state | filesystem perms on `~/.openclaw` | yes |
| `fs.state_dir.perms_group_writable` | warn | Group users can modify full OpenClaw state | filesystem perms on `~/.openclaw` | yes |
| `fs.state_dir.perms_readable` | warn | State dir is readable by others | filesystem perms on `~/.openclaw` | yes |
| `fs.state_dir.symlink` | warn | State dir target becomes another trust boundary | state dir filesystem layout | no |
| `fs.config.perms_writable` | critical | Others can change auth/tool policy/config | filesystem perms on `~/.openclaw/openclaw.json` | yes |
| `fs.config.symlink` | warn | Symlinked config files are unsupported for writes and add another trust boundary | replace with a regular config file or point `OPENCLAW_CONFIG_PATH` at the real file | no |
| `fs.config.perms_group_readable` | warn | Group users can read config tokens/settings | filesystem perms on config file | yes |
| `fs.config.perms_world_readable` | critical | Config can expose tokens/settings | filesystem perms on config file | yes |
| `fs.config_include.perms_writable` | critical | Config include file can be modified by others | include-file perms referenced from `openclaw.json` | yes |
| `fs.config_include.perms_group_readable` | warn | Group users can read included secrets/settings | include-file perms referenced from `openclaw.json` | yes |
| `fs.config_include.perms_world_readable` | critical | Included secrets/settings are world-readable | include-file perms referenced from `openclaw.json` | yes |
| `fs.auth_profiles.perms_writable` | critical | Others can inject or replace stored model credentials | `agents/<agentId>/agent/auth-profiles.json` perms | yes |
| `fs.auth_profiles.perms_readable` | warn | Others can read API keys and OAuth tokens | `agents/<agentId>/agent/auth-profiles.json` perms | yes |
| `fs.credentials_dir.perms_writable` | critical | Others can modify channel pairing/credential state | filesystem perms on `~/.openclaw/credentials` | yes |
| `fs.credentials_dir.perms_readable` | warn | Others can read channel credential state | filesystem perms on `~/.openclaw/credentials` | yes |
| `fs.sessions_store.perms_readable` | warn | Others can read session transcripts/metadata | session store perms | yes |
| `fs.log_file.perms_readable` | warn | Others can read redacted-but-still-sensitive logs | gateway log file perms | yes |
| `fs.synced_dir` | warn | State/config in iCloud/Dropbox/Drive broadens token/transcript exposure | move config/state off synced folders | no |
| `gateway.bind_no_auth` | critical | Remote bind without shared secret | `gateway.bind`, `gateway.auth.*` | no |
| `gateway.loopback_no_auth` | critical | Reverse-proxied loopback may become unauthenticated | `gateway.auth.*`, proxy setup | no |
| `gateway.trusted_proxies_missing` | warn | Reverse-proxy headers are present but not trusted | `gateway.trustedProxies` | no |
| `gateway.http.no_auth` | warn/critical | Gateway HTTP APIs reachable with `auth.mode=\"none\"` | `gateway.auth.mode`, `gateway.http.endpoints.*` | no |
| `gateway.http.session_key_override_enabled` | info | HTTP API callers can override `sessionKey` | `gateway.http.allowSessionKeyOverride` | no |
| `gateway.tools_invoke_http.dangerous_allow` | warn/critical | Re-enables dangerous tools over HTTP API | `gateway.tools.allow` | no |
| `gateway.nodes.allow_commands_dangerous` | warn/critical | Enables high-impact node commands (camera/screen/contacts/calendar/SMS) | `gateway.nodes.allowCommands` | no |
| `gateway.nodes.deny_commands_ineffective` | warn | Pattern-like deny entries do not match shell text or groups | `gateway.nodes.denyCommands` | no |
| `gateway.tailscale_funnel` | critical | Public internet exposure | `gateway.tailscale.mode` | no |
| `gateway.tailscale_serve` | info | Tailnet exposure is enabled via Serve | `gateway.tailscale.mode` | no |
| `gateway.control_ui.allowed_origins_required` | critical | Non-loopback Control UI without explicit browser-origin allowlist | `gateway.controlUi.allowedOrigins` | no |
| `gateway.control_ui.allowed_origins_wildcard` | warn/critical | `allowedOrigins=[\"*\"]` disables browser-origin allowlisting | `gateway.controlUi.allowedOrigins` | no |
| `gateway.control_ui.host_header_origin_fallback` | warn/critical | Enables Host-header origin fallback (DNS rebinding hardening downgrade) | `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` | no |
| `gateway.control_ui.insecure_auth` | warn | Insecure-auth compatibility toggle enabled | `gateway.controlUi.allowInsecureAuth` | no |
| `gateway.control_ui.device_auth_disabled` | critical | Disables device identity check | `gateway.controlUi.dangerouslyDisableDeviceAuth` | no |
| `gateway.real_ip_fallback_enabled` | warn/critical | Trusting `X-Real-IP` fallback can enable source-IP spoofing via proxy misconfig | `gateway.allowRealIpFallback`, `gateway.trustedProxies` | no |
| `gateway.token_too_short` | warn | Short shared token is easier to brute force | `gateway.auth.token` | no |
| `gateway.auth_no_rate_limit` | warn | Exposed auth without rate limiting increases brute-force risk | `gateway.auth.rateLimit` | no |
| `gateway.trusted_proxy_auth` | critical | Proxy identity now becomes the auth boundary | `gateway.auth.mode=\"trusted-proxy\"` | no |
| `gateway.trusted_proxy_no_proxies` | critical | Trusted-proxy auth without trusted proxy IPs is unsafe | `gateway.trustedProxies` | no |
| `gateway.trusted_proxy_no_user_header` | critical | Trusted-proxy auth cannot resolve user identity safely | `gateway.auth.trustedProxy.userHeader` | no |
| `gateway.trusted_proxy_no_allowlist` | warn | Trusted-proxy auth accepts any authenticated upstream user | `gateway.auth.trustedProxy.allowUsers` | no |
| `gateway.probe_auth_secretref_unavailable` | warn | Deep probe could not resolve auth SecretRefs in this command path | deep-probe auth source / SecretRef availability | no |
| `gateway.probe_failed` | warn/critical | Live Gateway probe failed | gateway reachability/auth | no |
| `discovery.mdns_full_mode` | warn/critical | mDNS full mode advertises `cliPath`/`sshPort` metadata on local network | `discovery.mdns.mode`, `gateway.bind` | no |
| `config.insecure_or_dangerous_flags` | warn | Any insecure/dangerous debug flags enabled | multiple keys (see finding detail) | no |
| `config.secrets.gateway_password_in_config` | warn | Gateway password is stored directly in config | `gateway.auth.password` | no |
| `config.secrets.hooks_token_in_config` | warn | Hook bearer token is stored directly in config | `hooks.token` | no |
| `hooks.token_reuse_gateway_token` | critical | Hook ingress token also unlocks Gateway auth | `hooks.token`, `gateway.auth.token` | no |
| `hooks.token_too_short` | warn | Easier brute force on hook ingress | `hooks.token` | no |
| `hooks.default_session_key_unset` | warn | Hook agent runs fan out into generated per-request sessions | `hooks.defaultSessionKey` | no |
| `hooks.allowed_agent_ids_unrestricted` | warn/critical | Authenticated hook callers may route to any configured agent | `hooks.allowedAgentIds` | no |
| `hooks.request_session_key_enabled` | warn/critical | External caller can choose sessionKey | `hooks.allowRequestSessionKey` | no |
| `hooks.request_session_key_prefixes_missing` | warn/critical | No bound on external session key shapes | `hooks.allowedSessionKeyPrefixes` | no |
| `hooks.path_root` | critical | Hook path is `/`, making ingress easier to collide or misroute | `hooks.path` | no |
| `hooks.installs_unpinned_npm_specs` | warn | Hook install records are not pinned to immutable npm specs | hook install metadata | no |
| `hooks.installs_missing_integrity` | warn | Hook install records lack integrity metadata | hook install metadata | no |
| `hooks.installs_version_drift` | warn | Hook install records drift from installed packages | hook install metadata | no |
| `logging.redact_off` | warn | Sensitive values leak to logs/status | `logging.redactSensitive` | yes |
| `browser.control_invalid_config` | warn | Browser control config is invalid before runtime | `browser.*` | no |
| `browser.control_no_auth` | critical | Browser control exposed without token/password auth | `gateway.auth.*` | no |
| `browser.remote_cdp_http` | warn | Remote CDP over plain HTTP lacks transport encryption | browser profile `cdpUrl` | no |
| `browser.remote_cdp_private_host` | warn | Remote CDP targets a private/internal host | browser profile `cdpUrl`, `browser.ssrfPolicy.*` | no |
| `sandbox.docker_config_mode_off` | warn | Sandbox Docker config present but inactive | `agents.*.sandbox.mode` | no |
| `sandbox.bind_mount_non_absolute` | warn | Relative bind mounts can resolve unpredictably | `agents.*.sandbox.docker.binds[]` | no |
| `sandbox.dangerous_bind_mount` | critical | Sandbox bind mount targets blocked system, credential, or Docker socket paths | `agents.*.sandbox.docker.binds[]` | no |
| `sandbox.dangerous_network_mode` | critical | Sandbox Docker network uses `host` or `container:*` namespace-join mode | `agents.*.sandbox.docker.network` | no |
| `sandbox.dangerous_seccomp_profile` | critical | Sandbox seccomp profile weakens container isolation | `agents.*.sandbox.docker.securityOpt` | no |
| `sandbox.dangerous_apparmor_profile` | critical | Sandbox AppArmor profile weakens container isolation | `agents.*.sandbox.docker.securityOpt` | no |
| `sandbox.browser_cdp_bridge_unrestricted` | warn | Sandbox browser bridge is exposed without source-range restriction | `sandbox.browser.cdpSourceRange` | no |
| `sandbox.browser_container.non_loopback_publish` | critical | Existing browser container publishes CDP on non-loopback interfaces | browser sandbox container publish config | no |
| `sandbox.browser_container.hash_label_missing` | warn | Existing browser container predates current config-hash labels | `openclaw sandbox recreate --browser --all` | no |
| `sandbox.browser_container.hash_epoch_stale` | warn | Existing browser container predates current browser config epoch | `openclaw sandbox recreate --browser --all` | no |
| `tools.exec.host_sandbox_no_sandbox_defaults` | warn | `exec host=sandbox` fails closed when sandbox is off | `tools.exec.host`, `agents.defaults.sandbox.mode` | no |
| `tools.exec.host_sandbox_no_sandbox_agents` | warn | Per-agent `exec host=sandbox` fails closed when sandbox is off | `agents.list[].tools.exec.host`, `agents.list[].sandbox.mode` | no |
| `tools.exec.security_full_configured` | warn/critical | Host exec is running with `security=\"full\"` | `tools.exec.security`, `agents.list[].tools.exec.security` | no |
| `tools.exec.auto_allow_skills_enabled` | warn | Exec approvals trust skill bins implicitly | `~/.openclaw/exec-approvals.json` | no |
| `tools.exec.allowlist_interpreter_without_strict_inline_eval` | warn | Interpreter allowlists permit inline eval without forced reapproval | `tools.exec.strictInlineEval`, `agents.list[].tools.exec.strictInlineEval`, exec approvals allowlist | no |
| `tools.exec.safe_bins_interpreter_unprofiled` | warn | Interpreter/runtime bins in `safeBins` without explicit profiles broaden exec risk | `tools.exec.safeBins`, `tools.exec.safeBinProfiles`, `agents.list[].tools.exec.*` | no |
| `tools.exec.safe_bins_broad_behavior` | warn | Broad-behavior tools in `safeBins` weaken the low-risk stdin-filter trust model | `tools.exec.safeBins`, `agents.list[].tools.exec.safeBins` | no |
| `tools.exec.safe_bin_trusted_dirs_risky` | warn | `safeBinTrustedDirs` includes mutable or risky directories | `tools.exec.safeBinTrustedDirs`, `agents.list[].tools.exec.safeBinTrustedDirs` | no |
| `skills.workspace.symlink_escape` | warn | Workspace `skills/**/SKILL.md` resolves outside workspace root (symlink-chain drift) | workspace `skills/**` filesystem state | no |
| `plugins.extensions_no_allowlist` | warn | Plugins are installed without an explicit plugin allowlist | `plugins.allowlist` | no |
| `plugins.installs_unpinned_npm_specs` | warn | Plugin index records are not pinned to immutable npm specs | plugin install metadata | no |
| `plugins.installs_missing_integrity` | warn | Plugin index records lack integrity metadata | plugin install metadata | no |
| `plugins.installs_version_drift` | warn | Plugin index records drift from installed packages | plugin install metadata | no |
| `plugins.code_safety` | warn/critical | Plugin code scan found suspicious or dangerous patterns | plugin code / install source | no |
| `plugins.code_safety.entry_path` | warn | Plugin entry path points into hidden or `node_modules` locations | plugin manifest `entry` | no |
| `plugins.code_safety.entry_escape` | critical | Plugin entry escapes the plugin directory | plugin manifest `entry` | no |
| `plugins.code_safety.scan_failed` | warn | Plugin code scan could not complete | plugin path / scan environment | no |
| `skills.code_safety` | warn/critical | Skill installer metadata/code contains suspicious or dangerous patterns | skill install source | no |
| `skills.code_safety.scan_failed` | warn | Skill code scan could not complete | skill scan environment | no |
| `security.exposure.open_channels_with_exec` | warn/critical | Shared/public rooms can reach exec-enabled agents | `channels.*.dmPolicy`, `channels.*.groupPolicy`, `tools.exec.*`, `agents.list[].tools.exec.*` | no |
| `security.exposure.open_groups_with_elevated` | critical | Open groups + elevated tools create high-impact prompt-injection paths | `channels.*.groupPolicy`, `tools.elevated.*` | no |
| `security.exposure.open_groups_with_runtime_or_fs` | critical/warn | Open groups can reach command/file tools without sandbox/workspace guards | `channels.*.groupPolicy`, `tools.profile/deny`, `tools.fs.workspaceOnly`, `agents.*.sandbox.mode` | no |
| `security.trust_model.multi_user_heuristic` | warn | Config looks multi-user while gateway trust model is personal-assistant | split trust boundaries, or shared-user hardening (`sandbox.mode`, tool deny/workspace scoping\\`) | no |
| `tools.profile_minimal_overridden` | warn | Agent overrides bypass global minimal profile | `agents.list[].tools.profile` | no |
| `plugins.tools_reachable_permissive_policy` | warn | Extension tools reachable in permissive contexts | `tools.profile` \\+ tool allow/deny | no |
| `models.legacy` | warn | Legacy model families are still configured | model selection | no |
| `models.weak_tier` | warn | Configured models are below current recommended tiers | model selection | no |
| `models.small_params` | critical/info | Small models + unsafe tool surfaces raise injection risk | model choice + sandbox/tool policy | no |
| `summary.attack_surface` | info | Roll-up summary of auth, channel, tool, and exposure posture | multiple keys (see finding detail) | no |

## [​](https://docs.openclaw.ai/gateway/security/audit-checks\\#related)  Related

- [Security](https://docs.openclaw.ai/gateway/security)
- [Configuration](https://docs.openclaw.ai/gateway/configuration)
- [Trusted proxy auth](https://docs.openclaw.ai/gateway/trusted-proxy-auth)

[Security](https://docs.openclaw.ai/gateway/security) [Sandboxing](https://docs.openclaw.ai/gateway/sandboxing)

Ctrl+I

---

## Secrets management
**Source:** https://docs.openclaw.ai/gateway/secrets

[Skip to main content](https://docs.openclaw.ai/gateway/secrets#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Authentication and secrets

Secrets management

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Goals and runtime model](https://docs.openclaw.ai/gateway/secrets#goals-and-runtime-model)
- [Active-surface filtering](https://docs.openclaw.ai/gateway/secrets#active-surface-filtering)
- [Gateway auth surface diagnostics](https://docs.openclaw.ai/gateway/secrets#gateway-auth-surface-diagnostics)
- [Onboarding reference preflight](https://docs.openclaw.ai/gateway/secrets#onboarding-reference-preflight)
- [SecretRef contract](https://docs.openclaw.ai/gateway/secrets#secretref-contract)
- [Provider config](https://docs.openclaw.ai/gateway/secrets#provider-config)
- [Exec integration examples](https://docs.openclaw.ai/gateway/secrets#exec-integration-examples)
- [MCP server environment variables](https://docs.openclaw.ai/gateway/secrets#mcp-server-environment-variables)
- [Sandbox SSH auth material](https://docs.openclaw.ai/gateway/secrets#sandbox-ssh-auth-material)
- [Supported credential surface](https://docs.openclaw.ai/gateway/secrets#supported-credential-surface)
- [Required behavior and precedence](https://docs.openclaw.ai/gateway/secrets#required-behavior-and-precedence)
- [Activation triggers](https://docs.openclaw.ai/gateway/secrets#activation-triggers)
- [Degraded and recovered signals](https://docs.openclaw.ai/gateway/secrets#degraded-and-recovered-signals)
- [Command-path resolution](https://docs.openclaw.ai/gateway/secrets#command-path-resolution)
- [Audit and configure workflow](https://docs.openclaw.ai/gateway/secrets#audit-and-configure-workflow)
- [One-way safety policy](https://docs.openclaw.ai/gateway/secrets#one-way-safety-policy)
- [Legacy auth compatibility notes](https://docs.openclaw.ai/gateway/secrets#legacy-auth-compatibility-notes)
- [Web UI note](https://docs.openclaw.ai/gateway/secrets#web-ui-note)
- [Related](https://docs.openclaw.ai/gateway/secrets#related)

OpenClaw supports additive SecretRefs so supported credentials do not need to be stored as plaintext in configuration.

Plaintext still works. SecretRefs are opt-in per credential.

## [​](https://docs.openclaw.ai/gateway/secrets\\#goals-and-runtime-model)  Goals and runtime model

Secrets are resolved into an in-memory runtime snapshot.

- Resolution is eager during activation, not lazy on request paths.
- Startup fails fast when an effectively active SecretRef cannot be resolved.
- Reload uses atomic swap: full success, or keep the last-known-good snapshot.
- SecretRef policy violations (for example OAuth-mode auth profiles combined with SecretRef input) fail activation before runtime swap.
- Runtime requests read from the active in-memory snapshot only.
- After the first successful config activation/load, runtime code paths keep reading that active in-memory snapshot until a successful reload swaps it.
- Outbound delivery paths also read from that active snapshot (for example Discord reply/thread delivery and Telegram action sends); they do not re-resolve SecretRefs on each send.

This keeps secret-provider outages off hot request paths.

## [​](https://docs.openclaw.ai/gateway/secrets\\#active-surface-filtering)  Active-surface filtering

SecretRefs are validated only on effectively active surfaces.

- Enabled surfaces: unresolved refs block startup/reload.
- Inactive surfaces: unresolved refs do not block startup/reload.
- Inactive refs emit non-fatal diagnostics with code `SECRETS_REF_IGNORED_INACTIVE_SURFACE`.

Examples of inactive surfaces

- Disabled channel/account entries.
- Top-level channel credentials that no enabled account inherits.
- Disabled tool/feature surfaces.
- Web search provider-specific keys that are not selected by `tools.web.search.provider`. In auto mode (provider unset), keys are consulted by precedence for provider auto-detection until one resolves. After selection, non-selected provider keys are treated as inactive until selected.
- Sandbox SSH auth material (`agents.defaults.sandbox.ssh.identityData`, `certificateData`, `knownHostsData`, plus per-agent overrides) is active only when the effective sandbox backend is `ssh` for the default agent or an enabled agent.
- `gateway.remote.token` / `gateway.remote.password` SecretRefs are active if one of these is true:

  - `gateway.mode=remote`
  - `gateway.remote.url` is configured
  - `gateway.tailscale.mode` is `serve` or `funnel`
  - In local mode without those remote surfaces:
    - `gateway.remote.token` is active when token auth can win and no env/auth token is configured.
    - `gateway.remote.password` is active only when password auth can win and no env/auth password is configured.
- `gateway.auth.token` SecretRef is inactive for startup auth resolution when `OPENCLAW_GATEWAY_TOKEN` is set, because env token input wins for that runtime.

## [​](https://docs.openclaw.ai/gateway/secrets\\#gateway-auth-surface-diagnostics)  Gateway auth surface diagnostics

When a SecretRef is configured on `gateway.auth.token`, `gateway.auth.password`, `gateway.remote.token`, or `gateway.remote.password`, gateway startup/reload logs the surface state explicitly:

- `active`: the SecretRef is part of the effective auth surface and must resolve.
- `inactive`: the SecretRef is ignored for this runtime because another auth surface wins, or because remote auth is disabled/not active.

These entries are logged with `SECRETS_GATEWAY_AUTH_SURFACE` and include the reason used by the active-surface policy, so you can see why a credential was treated as active or inactive.

## [​](https://docs.openclaw.ai/gateway/secrets\\#onboarding-reference-preflight)  Onboarding reference preflight

When onboarding runs in interactive mode and you choose SecretRef storage, OpenClaw runs preflight validation before saving:

- Env refs: validates env var name and confirms a non-empty value is visible during setup.
- Provider refs (`file` or `exec`): validates provider selection, resolves `id`, and checks resolved value type.
- Quickstart reuse path: when `gateway.auth.token` is already a SecretRef, onboarding resolves it before probe/dashboard bootstrap (for `env`, `file`, and `exec` refs) using the same fail-fast gate.

If validation fails, onboarding shows the error and lets you retry.

## [​](https://docs.openclaw.ai/gateway/secrets\\#secretref-contract)  SecretRef contract

Use one object shape everywhere:

```
{
  source: \"env\" | \"file\" | \"exec\",
  provider: \"default\",
  id: \"...\"
}
```

- env

- file

- exec


```
{
  source: \"env\",
  provider: \"default\",
  id: \"OPENAI_API_KEY\"
}
```

Validation:

- `provider` must match `^[a-z][a-z0-9_-]{0,63}$`
- `id` must match `^[A-Z][A-Z0-9_]{0,127}$`

```
{
  source: \"file\",
  provider: \"filemain\",
  id: \"/providers/openai/apiKey\"
}
```

Validation:

- `provider` must match `^[a-z][a-z0-9_-]{0,63}$`
- `id` must be an absolute JSON pointer (`/...`)
- RFC6901 escaping in segments: `~` => `~0`, `/` => `~1`

```
{
  source: \"exec\",
  provider: \"vault\",
  id: \"providers/openai/apiKey\"
}
```

Validation:

- `provider` must match `^[a-z][a-z0-9_-]{0,63}$`
- `id` must match `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$`
- `id` must not contain `.` or `..` as slash-delimited path segments (for example `a/../b` is rejected)

## [​](https://docs.openclaw.ai/gateway/secrets\\#provider-config)  Provider config

Define providers under `secrets.providers`:

```
{
  secrets: {
    providers: {
      default: { source: \"env\" },
      filemain: {
        source: \"file\",
        path: \"~/.openclaw/secrets.json\",
        mode: \"json\", // or \"singleValue\"
      },
      vault: {
        source: \"exec\",
        command: \"/usr/local/bin/openclaw-vault-resolver\",
        args: [\"--profile\", \"prod\"],
        passEnv: [\"PATH\", \"VAULT_ADDR\"],
        jsonOnly: true,
      },
    },
    defaults: {
      env: \"default\",
      file: \"filemain\",
      exec: \"vault\",
    },
    resolution: {
      maxProviderConcurrency: 4,
      maxRefsPerProvider: 512,
      maxBatchBytes: 262144,
    },
  },
}
```

Env provider

- Optional allowlist via `allowlist`.
- Missing/empty env values fail resolution.

File provider

- Reads local file from `path`.
- `mode: \"json\"` expects JSON object payload and resolves `id` as pointer.
- `mode: \"singleValue\"` expects ref id `\"value\"` and returns file contents.
- Path must pass ownership/permission checks.
- Windows fail-closed note: if ACL verification is unavailable for a path, resolution fails. For trusted paths only, set `allowInsecurePath: true` on that provider to bypass path security checks.

Exec provider

- Runs configured absolute binary path, no shell.
- By default, `command` must point to a regular file (not a symlink).
- Set `allowSymlinkCommand: true` to allow symlink command paths (for example Homebrew shims). OpenClaw validates the resolved target path.
- Pair `allowSymlinkCommand` with `trustedDirs` for package-manager paths (for example `[\"/opt/homebrew\"]`).
- Supports timeout, no-output timeout, output byte limits, env allowlist, and trusted dirs.
- Windows fail-closed note: if ACL verification is unavailable for the command path, resolution fails. For trusted paths only, set `allowInsecurePath: true` on that provider to bypass path security checks.

Request payload (stdin):

```
{
  \"protocolVersion\": 1,
  \"provider\": \"vault\",
  \"ids\": [\"providers/openai/apiKey\"]
}
```

Response payload (stdout):

```
{
  \"protocolVersion\": 1,
  \"values\": { \"providers/openai/apiKey\": \"<openai-api-key>\" }
} // pragma: allowlist secret
```

Optional per-id errors:

```
{
  \"protocolVersion\": 1,
  \"values\": {},
  \"errors\": { \"providers/openai/apiKey\": { \"message\": \"not found\" } }
}
```

## [​](https://docs.openclaw.ai/gateway/secrets\\#exec-integration-examples)  Exec integration examples

1Password CLI

```
{
  secrets: {
    providers: {
      onepassword_openai: {
        source: \"exec\",
        command: \"/opt/homebrew/bin/op\",
        allowSymlinkCommand: true, // required for Homebrew symlinked binaries
        trustedDirs: [\"/opt/homebrew\"],
        args: [\"read\", \"op://Personal/OpenClaw QA API Key/password\"],
        passEnv: [\"HOME\"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: \"https://api.openai.com/v1\",
        models: [{ id: \"gpt-5\", name: \"gpt-5\" }],
        apiKey: { source: \"exec\", provider: \"onepassword_openai\", id: \"value\" },
      },
    },
  },
}
```

HashiCorp Vault CLI

```
{
  secrets: {
    providers: {
      vault_openai: {
        source: \"exec\",
        command: \"/opt/homebrew/bin/vault\",
        allowSymlinkCommand: true, // required for Homebrew symlinked binaries
        trustedDirs: [\"/opt/homebrew\"],
        args: [\"kv\", \"get\", \"-field=OPENAI_API_KEY\", \"secret/openclaw\"],
        passEnv: [\"VAULT_ADDR\", \"VAULT_TOKEN\"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: \"https://api.openai.com/v1\",
        models: [{ id: \"gpt-5\", name: \"gpt-5\" }],
        apiKey: { source: \"exec\", provider: \"vault_openai\", id: \"value\" },
      },
    },
  },
}
```

sops

```
{
  secrets: {
    providers: {
      sops_openai: {
        source: \"exec\",
        command: \"/opt/homebrew/bin/sops\",
        allowSymlinkCommand: true, // required for Homebrew symlinked binaries
        trustedDirs: [\"/opt/homebrew\"],
        args: [\"-d\", \"--extract\", '[\"providers\"][\"openai\"][\"apiKey\"]', \"/path/to/secrets.enc.json\"],
        passEnv: [\"SOPS_AGE_KEY_FILE\"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: \"https://api.openai.com/v1\",
        models: [{ id: \"gpt-5\", name: \"gpt-5\" }],
        apiKey: { source: \"exec\", provider: \"sops_openai\", id: \"value\" },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/gateway/secrets\\#mcp-server-environment-variables)  MCP server environment variables

MCP server env vars configured via `plugins.entries.acpx.config.mcpServers` support SecretInput. This keeps API keys and tokens out of plaintext config:

```
{
  plugins: {
    entries: {
      acpx: {
        enabled: true,
        config: {
          mcpServers: {
            github: {
              command: \"npx\",
              args: [\"-y\", \"@modelcontextprotocol/server-github\"],
              env: {
                GITHUB_PERSONAL_ACCESS_TOKEN: {
                  source: \"env\",
                  provider: \"default\",
                  id: \"MCP_GITHUB_PAT\",
                },
              },
            },
          },
        },
      },
    },
  },
}
```

Plaintext string values still work. Env-template refs like `${MCP_SERVER_API_KEY}` and SecretRef objects are resolved during gateway activation before the MCP server process is spawned. As with other SecretRef surfaces, unresolved refs only block activation when the `acpx` plugin is effectively active.

## [​](https://docs.openclaw.ai/gateway/secrets\\#sandbox-ssh-auth-material)  Sandbox SSH auth material

The core `ssh` sandbox backend also supports SecretRefs for SSH auth material:

```
{
  agents: {
    defaults: {
      sandbox: {
        mode: \"all\",
        backend: \"ssh\",
        ssh: {
          target: \"user@gateway-host:22\",
          identityData: { source: \"env\", provider: \"default\", id: \"SSH_IDENTITY\" },
          certificateData: { source: \"env\", provider: \"default\", id: \"SSH_CERTIFICATE\" },
          knownHostsData: { source: \"env\", provider: \"default\", id: \"SSH_KNOWN_HOSTS\" },
        },
      },
    },
  },
}
```

Runtime behavior:

- OpenClaw resolves these refs during sandbox activation, not lazily during each SSH call.
- Resolved values are written to temp files with restrictive permissions and used in generated SSH config.
- If the effective sandbox backend is not `ssh`, these refs stay inactive and do not block startup.

## [​](https://docs.openclaw.ai/gateway/secrets\\#supported-credential-surface)  Supported credential surface

Canonical supported and unsupported credentials are listed in:

- [SecretRef Credential Surface](https://docs.openclaw.ai/reference/secretref-credential-surface)

Runtime-minted or rotating credentials and OAuth refresh material are intentionally excluded from read-only SecretRef resolution.

## [​](https://docs.openclaw.ai/gateway/secrets\\#required-behavior-and-precedence)  Required behavior and precedence

- Field without a ref: unchanged.
- Field with a ref: required on active surfaces during activation.
- If both plaintext and ref are present, ref takes precedence on supported precedence paths.
- The redaction sentinel `__OPENCLAW_REDACTED__` is reserved for internal config redaction/restore and is rejected as literal submitted config data.

Warning and audit signals:

- `SECRETS_REF_OVERRIDES_PLAINTEXT` (runtime warning)
- `REF_SHADOWED` (audit finding when `auth-profiles.json` credentials take precedence over `openclaw.json` refs)

Google Chat compatibility behavior:

- `serviceAccountRef` takes precedence over plaintext `serviceAccount`.
- Plaintext value is ignored when sibling ref is set.

## [​](https://docs.openclaw.ai/gateway/secrets\\#activation-triggers)  Activation triggers

Secret activation runs on:

- Startup (preflight plus final activation)
- Config reload hot-apply path
- Config reload restart-check path
- Manual reload via `secrets.reload`
- Gateway config write RPC preflight (`config.set` / `config.apply` / `config.patch`) for active-surface SecretRef resolvability within the submitted config payload before persisting edits

Activation contract:

- Success swaps the snapshot atomically.
- Startup failure aborts gateway startup.
- Runtime reload failure keeps the last-known-good snapshot.
- Write-RPC preflight failure rejects the submitted config and keeps both disk config and active runtime snapshot unchanged.
- Providing an explicit per-call channel token to an outbound helper/tool call does not trigger SecretRef activation; activation points remain startup, reload, and explicit `secrets.reload`.

## [​](https://docs.openclaw.ai/gateway/secrets\\#degraded-and-recovered-signals)  Degraded and recovered signals

When reload-time activation fails after a healthy state, OpenClaw enters degraded secrets state.One-shot system event and log codes:

- `SECRETS_RELOADER_DEGRADED`
- `SECRETS_RELOADER_RECOVERED`

Behavior:

- Degraded: runtime keeps last-known-good snapshot.
- Recovered: emitted once after the next successful activation.
- Repeated failures while already degraded log warnings but do not spam events.
- Startup fail-fast does not emit degraded events because runtime never became active.

## [​](https://docs.openclaw.ai/gateway/secrets\\#command-path-resolution)  Command-path resolution

Command paths can opt into supported SecretRef resolution via gateway snapshot RPC.There are two broad behaviors:

- Strict command paths

- Read-only command paths


For example `openclaw memory` remote-memory paths and `openclaw qr --remote` when it needs remote shared-secret refs. They read from the active snapshot and fail fast when a required SecretRef is unavailable.

For example `openclaw status`, `openclaw status --all`, `openclaw channels status`, `openclaw channels resolve`, `openclaw security audit`, and read-only doctor/config repair flows. They also prefer the active snapshot, but degrade instead of aborting when a targeted SecretRef is unavailable in that command path.Read-only behavior:

- When the gateway is running, these commands read from the active snapshot first.
- If gateway resolution is incomplete or the gateway is unavailable, they attempt targeted local fallback for the specific command surface.
- If a targeted SecretRef is still unavailable, the command continues with degraded read-only output and explicit diagnostics such as “configured but unavailable in this command path”.
- This degraded behavior is command-local only. It does not weaken runtime startup, reload, or send/auth paths.

Other notes:

- Snapshot refresh after backend secret rotation is handled by `openclaw secrets reload`.
- Gate...(content truncated)

---

## OpenTelemetry export
**Source:** https://docs.openclaw.ai/gateway/opentelemetry

[Skip to main content](https://docs.openclaw.ai/gateway/opentelemetry#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Health and diagnostics

OpenTelemetry export

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [How it fits together](https://docs.openclaw.ai/gateway/opentelemetry#how-it-fits-together)
- [Quick start](https://docs.openclaw.ai/gateway/opentelemetry#quick-start)
- [Signals exported](https://docs.openclaw.ai/gateway/opentelemetry#signals-exported)
- [Configuration reference](https://docs.openclaw.ai/gateway/opentelemetry#configuration-reference)
- [Environment variables](https://docs.openclaw.ai/gateway/opentelemetry#environment-variables)
- [Privacy and content capture](https://docs.openclaw.ai/gateway/opentelemetry#privacy-and-content-capture)
- [Sampling and flushing](https://docs.openclaw.ai/gateway/opentelemetry#sampling-and-flushing)
- [Exported metrics](https://docs.openclaw.ai/gateway/opentelemetry#exported-metrics)
- [Model usage](https://docs.openclaw.ai/gateway/opentelemetry#model-usage)
- [Message flow](https://docs.openclaw.ai/gateway/opentelemetry#message-flow)
- [Queues and sessions](https://docs.openclaw.ai/gateway/opentelemetry#queues-and-sessions)
- [Harness lifecycle](https://docs.openclaw.ai/gateway/opentelemetry#harness-lifecycle)
- [Exec](https://docs.openclaw.ai/gateway/opentelemetry#exec)
- [Diagnostics internals (memory and tool loop)](https://docs.openclaw.ai/gateway/opentelemetry#diagnostics-internals-memory-and-tool-loop)
- [Exported spans](https://docs.openclaw.ai/gateway/opentelemetry#exported-spans)
- [Diagnostic event catalog](https://docs.openclaw.ai/gateway/opentelemetry#diagnostic-event-catalog)
- [Without an exporter](https://docs.openclaw.ai/gateway/opentelemetry#without-an-exporter)
- [Disable](https://docs.openclaw.ai/gateway/opentelemetry#disable)
- [Related](https://docs.openclaw.ai/gateway/opentelemetry#related)

OpenClaw exports diagnostics through the bundled `diagnostics-otel` plugin
using **OTLP/HTTP (protobuf)**. Any collector or backend that accepts OTLP/HTTP
works without code changes. For local file logs and how to read them, see
[Logging](https://docs.openclaw.ai/logging).

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#how-it-fits-together)  How it fits together

- **Diagnostics events** are structured, in-process records emitted by the
Gateway and bundled plugins for model runs, message flow, sessions, queues,
and exec.
- **`diagnostics-otel` plugin** subscribes to those events and exports them as
OpenTelemetry **metrics**, **traces**, and **logs** over OTLP/HTTP.
- **Provider calls** receive a W3C `traceparent` header from OpenClaw’s
trusted model-call span context when the provider transport accepts custom
headers. Plugin-emitted trace context is not propagated.
- Exporters only attach when both the diagnostics surface and the plugin are
enabled, so the in-process cost stays near zero by default.

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#quick-start)  Quick start

```
{
  plugins: {
    allow: [\"diagnostics-otel\"],
    entries: {
      \"diagnostics-otel\": { enabled: true },
    },
  },
  diagnostics: {
    enabled: true,
    otel: {
      enabled: true,
      endpoint: \"http://otel-collector:4318\",
      protocol: \"http/protobuf\",
      serviceName: \"openclaw-gateway\",
      traces: true,
      metrics: true,
      logs: true,
      sampleRate: 0.2,
      flushIntervalMs: 60000,
    },
  },
}
```

You can also enable the plugin from the CLI:

```
openclaw plugins enable diagnostics-otel
```

`protocol` currently supports `http/protobuf` only. `grpc` is ignored.

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#signals-exported)  Signals exported

| Signal | What goes in it |
| --- | --- |
| **Metrics** | Counters and histograms for token usage, cost, run duration, message flow, queue lanes, session state, exec, and memory pressure. |
| **Traces** | Spans for model usage, model calls, harness lifecycle, tool execution, exec, webhook/message processing, context assembly, and tool loops. |
| **Logs** | Structured `logging.file` records exported over OTLP when `diagnostics.otel.logs` is enabled. |

Toggle `traces`, `metrics`, and `logs` independently. All three default to on
when `diagnostics.otel.enabled` is true.

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#configuration-reference)  Configuration reference

```
{
  diagnostics: {
    enabled: true,
    otel: {
      enabled: true,
      endpoint: \"http://otel-collector:4318\",
      tracesEndpoint: \"http://otel-collector:4318/v1/traces\",
      metricsEndpoint: \"http://otel-collector:4318/v1/metrics\",
      logsEndpoint: \"http://otel-collector:4318/v1/logs\",
      protocol: \"http/protobuf\", // grpc is ignored
      serviceName: \"openclaw-gateway\",
      headers: { \"x-collector-token\": \"...\" },
      traces: true,
      metrics: true,
      logs: true,
      sampleRate: 0.2, // root-span sampler, 0.0..1.0
      flushIntervalMs: 60000, // metric export interval (min 1000ms)
      captureContent: {
        enabled: false,
        inputMessages: false,
        outputMessages: false,
        toolInputs: false,
        toolOutputs: false,
        systemPrompt: false,
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/opentelemetry\\#environment-variables)  Environment variables

| Variable | Purpose |
| --- | --- |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | Override `diagnostics.otel.endpoint`. If the value already contains `/v1/traces`, `/v1/metrics`, or `/v1/logs`, it is used as-is. |
| `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` / `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT` / `OTEL_EXPORTER_OTLP_LOGS_ENDPOINT` | Signal-specific endpoint overrides used when the matching `diagnostics.otel.*Endpoint` config key is unset. Signal-specific env, which wins over the shared endpoint. |
| `OTEL_SERVICE_NAME` | Override `diagnostics.otel.serviceName`. |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | Override the wire protocol (only `http/protobuf` is honored today). |
| `OTEL_SEMCONV_STABILITY_OPT_IN` | Set to `gen_ai_latest_experimental` to emit the latest experimental GenAI span attribute (`gen_ai.provider.name`) instead of the legacy `gen_ai.system`. GenAI metrics always use bounded, low-cardinality semantic attributes regardless. |
| `OPENCLAW_OTEL_PRELOADED` | Set to `1` when another preload or host process already registered the global OpenTelemetry SDK. The plugin then skips its own NodeSDK lifecycle but still wires diagnostic listeners and honors `traces`/`metrics`/`logs`. |

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#privacy-and-content-capture)  Privacy and content capture

Raw model/tool content is **not** exported by default. Spans carry bounded
identifiers (channel, provider, model, error category, hash-only request ids)
and never include prompt text, response text, tool inputs, tool outputs, or
session keys.Outbound model requests may include a W3C `traceparent` header. That header is
generated only from OpenClaw-owned diagnostic trace context for the active model
call. Existing caller-supplied `traceparent` headers are replaced, so plugins or
custom provider options cannot spoof cross-service trace ancestry.Set `diagnostics.otel.captureContent.*` to `true` only when your collector and
retention policy are approved for prompt, response, tool, or system-prompt
text. Each subkey is opt-in independently:

- `inputMessages` — user prompt content.
- `outputMessages` — model response content.
- `toolInputs` — tool argument payloads.
- `toolOutputs` — tool result payloads.
- `systemPrompt` — assembled system/developer prompt.

When any subkey is enabled, model and tool spans get bounded, redacted
`openclaw.content.*` attributes for that class only.

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#sampling-and-flushing)  Sampling and flushing

- **Traces:**`diagnostics.otel.sampleRate` (root-span only, `0.0` drops all,
`1.0` keeps all).
- **Metrics:**`diagnostics.otel.flushIntervalMs` (minimum `1000`).
- **Logs:** OTLP logs respect `logging.level` (file log level). They use the
diagnostic log-record redaction path, not console formatting. High-volume
installs should prefer OTLP collector sampling/filtering over local sampling.
- **File-log correlation:** JSONL file logs include top-level `traceId`,
`spanId`, `parentSpanId`, and `traceFlags` when the log call carries a valid
diagnostic trace context, which lets log processors join local log lines with
exported spans.
- **Request correlation:** Gateway HTTP requests and WebSocket frames create an
internal request trace scope. Logs and diagnostic events inside that scope
inherit the request trace by default, while agent run and model-call spans are
created as children so provider `traceparent` headers stay on the same trace.

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#exported-metrics)  Exported metrics

### [​](https://docs.openclaw.ai/gateway/opentelemetry\\#model-usage)  Model usage

- `openclaw.tokens` (counter, attrs: `openclaw.token`, `openclaw.channel`, `openclaw.provider`, `openclaw.model`, `openclaw.agent`)
- `openclaw.cost.usd` (counter, attrs: `openclaw.channel`, `openclaw.provider`, `openclaw.model`)
- `openclaw.run.duration_ms` (histogram, attrs: `openclaw.channel`, `openclaw.provider`, `openclaw.model`)
- `openclaw.context.tokens` (histogram, attrs: `openclaw.context`, `openclaw.channel`, `openclaw.provider`, `openclaw.model`)
- `gen_ai.client.token.usage` (histogram, GenAI semantic-conventions metric, attrs: `gen_ai.token.type` = `input`/`output`, `gen_ai.provider.name`, `gen_ai.operation.name`, `gen_ai.request.model`)
- `gen_ai.client.operation.duration` (histogram, seconds, GenAI semantic-conventions metric, attrs: `gen_ai.provider.name`, `gen_ai.operation.name`, `gen_ai.request.model`, optional `error.type`)
- `openclaw.model_call.duration_ms` (histogram, attrs: `openclaw.provider`, `openclaw.model`, `openclaw.api`, `openclaw.transport`, plus `openclaw.errorCategory` and `openclaw.failureKind` on classified errors)
- `openclaw.model_call.request_bytes` (histogram, UTF-8 byte size of the final model request payload; no raw payload content)
- `openclaw.model_call.response_bytes` (histogram, UTF-8 byte size of streamed model response events; no raw response content)
- `openclaw.model_call.time_to_first_byte_ms` (histogram, elapsed time before the first streamed response event)

### [​](https://docs.openclaw.ai/gateway/opentelemetry\\#message-flow)  Message flow

- `openclaw.webhook.received` (counter, attrs: `openclaw.channel`, `openclaw.webhook`)
- `openclaw.webhook.error` (counter, attrs: `openclaw.channel`, `openclaw.webhook`)
- `openclaw.webhook.duration_ms` (histogram, attrs: `openclaw.channel`, `openclaw.webhook`)
- `openclaw.message.queued` (counter, attrs: `openclaw.channel`, `openclaw.source`)
- `openclaw.message.processed` (counter, attrs: `openclaw.channel`, `openclaw.outcome`)
- `openclaw.message.duration_ms` (histogram, attrs: `openclaw.channel`, `openclaw.outcome`)
- `openclaw.message.delivery.started` (counter, attrs: `openclaw.channel`, `openclaw.delivery.kind`)
- `openclaw.message.delivery.duration_ms` (histogram, attrs: `openclaw.channel`, `openclaw.delivery.kind`, `openclaw.outcome`, `openclaw.errorCategory`)

### [​](https://docs.openclaw.ai/gateway/opentelemetry\\#queues-and-sessions)  Queues and sessions

- `openclaw.queue.lane.enqueue` (counter, attrs: `openclaw.lane`)
- `openclaw.queue.lane.dequeue` (counter, attrs: `openclaw.lane`)
- `openclaw.queue.depth` (histogram, attrs: `openclaw.lane` or `openclaw.channel=heartbeat`)
- `openclaw.queue.wait_ms` (histogram, attrs: `openclaw.lane`)
- `openclaw.session.state` (counter, attrs: `openclaw.state`, `openclaw.reason`)
- `openclaw.session.stuck` (counter, attrs: `openclaw.state`)
- `openclaw.session.stuck_age_ms` (histogram, attrs: `openclaw.state`)
- `openclaw.run.attempt` (counter, attrs: `openclaw.attempt`)

### [​](https://docs.openclaw.ai/gateway/opentelemetry\\#harness-lifecycle)  Harness lifecycle

- `openclaw.harness.duration_ms` (histogram, attrs: `openclaw.harness.id`, `openclaw.harness.plugin`, `openclaw.outcome`, `openclaw.harness.phase` on errors)

### [​](https://docs.openclaw.ai/gateway/opentelemetry\\#exec)  Exec

- `openclaw.exec.duration_ms` (histogram, attrs: `openclaw.exec.target`, `openclaw.exec.mode`, `openclaw.outcome`, `openclaw.failureKind`)

### [​](https://docs.openclaw.ai/gateway/opentelemetry\\#diagnostics-internals-memory-and-tool-loop)  Diagnostics internals (memory and tool loop)

- `openclaw.memory.heap_used_bytes` (histogram, attrs: `openclaw.memory.kind`)
- `openclaw.memory.rss_bytes` (histogram)
- `openclaw.memory.pressure` (counter, attrs: `openclaw.memory.level`)
- `openclaw.tool.loop.iterations` (counter, attrs: `openclaw.toolName`, `openclaw.outcome`)
- `openclaw.tool.loop.duration_ms` (histogram, attrs: `openclaw.toolName`, `openclaw.outcome`)

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#exported-spans)  Exported spans

- `openclaw.model.usage`
  - `openclaw.channel`, `openclaw.provider`, `openclaw.model`
  - `openclaw.tokens.*` (input/output/cache\\_read/cache\\_write/total)
  - `gen_ai.system` by default, or `gen_ai.provider.name` when the latest GenAI semantic conventions are opted in
  - `gen_ai.request.model`, `gen_ai.operation.name`, `gen_ai.usage.*`
- `openclaw.run`
  - `openclaw.outcome`, `openclaw.channel`, `openclaw.provider`, `openclaw.model`, `openclaw.errorCategory`
- `openclaw.model.call`
  - `gen_ai.system` by default, or `gen_ai.provider.name` when the latest GenAI semantic conventions are opted in
  - `gen_ai.request.model`, `gen_ai.operation.name`, `openclaw.provider`, `openclaw.model`, `openclaw.api`, `openclaw.transport`
  - `openclaw.errorCategory` and optional `openclaw.failureKind` on errors
  - `openclaw.model_call.request_bytes`, `openclaw.model_call.response_bytes`, `openclaw.model_call.time_to_first_byte_ms`
  - `openclaw.provider.request_id_hash` (bounded SHA-based hash of the upstream provider request id; raw ids are not exported)
- `openclaw.harness.run`
  - `openclaw.harness.id`, `openclaw.harness.plugin`, `openclaw.outcome`, `openclaw.provider`, `openclaw.model`, `openclaw.channel`
  - On completion: `openclaw.harness.result_classification`, `openclaw.harness.yield_detected`, `openclaw.harness.items.started`, `openclaw.harness.items.completed`, `openclaw.harness.items.active`
  - On error: `openclaw.harness.phase`, `openclaw.errorCategory`, optional `openclaw.harness.cleanup_failed`
- `openclaw.tool.execution`
  - `gen_ai.tool.name`, `openclaw.toolName`, `openclaw.errorCategory`, `openclaw.tool.params.*`
- `openclaw.exec`
  - `openclaw.exec.target`, `openclaw.exec.mode`, `openclaw.outcome`, `openclaw.failureKind`, `openclaw.exec.command_length`, `openclaw.exec.exit_code`, `openclaw.exec.timed_out`
- `openclaw.webhook.processed`
  - `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`
- `openclaw.webhook.error`
  - `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`, `openclaw.error`
- `openclaw.message.processed`
  - `openclaw.channel`, `openclaw.outcome`, `openclaw.chatId`, `openclaw.messageId`, `openclaw.reason`
- `openclaw.message.delivery`
  - `openclaw.channel`, `openclaw.delivery.kind`, `openclaw.outcome`, `openclaw.errorCategory`, `openclaw.delivery.result_count`
- `openclaw.session.stuck`
  - `openclaw.state`, `openclaw.ageMs`, `openclaw.queueDepth`
- `openclaw.context.assembled`
  - `openclaw.prompt.size`, `openclaw.history.size`, `openclaw.context.tokens`, `openclaw.errorCategory` (no prompt, history, response, or session-key content)
- `openclaw.tool.loop`
  - `openclaw.toolName`, `openclaw.outcome`, `openclaw.iterations`, `openclaw.errorCategory` (no loop messages, params, or tool output)
- `openclaw.memory.pressure`
  - `openclaw.memory.level`, `openclaw.memory.heap_used_bytes`, `openclaw.memory.rss_bytes`

When content capture is explicitly enabled, model and tool spans can also
include bounded, redacted `openclaw.content.*` attributes for the specific
content classes you opted into.

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#diagnostic-event-catalog)  Diagnostic event catalog

The events below back the metrics and spans above. Plugins can also subscribe
to them directly without OTLP export.**Model usage**

- `model.usage` — tokens, cost, duration, context, provider/model/channel,
session ids. `usage` is provider/turn accounting for cost and telemetry;
`context.used` is the current prompt/context snapshot and can be lower than
provider `usage.total` when cached input or tool-loop calls are involved.

**Message flow**

- `webhook.received` / `webhook.processed` / `webhook.error`
- `message.queued` / `message.processed`
- `message.delivery.started` / `message.delivery.completed` / `message.delivery.error`

**Queue and session**

- `queue.lane.enqueue` / `queue.lane.dequeue`
- `session.state` / `session.stuck`
- `run.attempt`
- `diagnostic.heartbeat` (aggregate counters: webhooks/queue/session)

**Harness lifecycle**

- `harness.run.started` / `harness.run.completed` / `harness.run.error` —
per-run lifecycle for the agent harness. Includes `harnessId`, optional
`pluginId`, provider/model/channel, and run id. Completion adds
`durationMs`, `outcome`, optional `resultClassification`, `yieldDetected`,
and `itemLifecycle` counts. Errors add `phase`
(`prepare`/`start`/`send`/`resolve`/`cleanup`), `errorCategory`, and
optional `cleanupFailed`.

**Exec**

- `exec.process.completed` — terminal outcome, duration, target, mode, exit
code, and failure kind. Command text and working directories are not
included.

## [​](https://docs.openclaw.ai/gateway/opentelemetry\\#without-an-exporter)  Without an exporter

You can keep diagnostics events available to plugins or custom sinks without
running `diagnostics-otel`:

```
{
  diagnostics: { enabled: true },
}
```

For targeted debug output without raising `logging.level`, use diagnostics
flags. Flags are case-insensitive and support wildcards (e.g. `telegram.*` or
`*`):

```
{
  diagnostics: { flags: [\"telegram.http\"] },
}
```

Or as a one-off env override:

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload openclaw gateway
```

Flag output goes to the standard log file (`logging.file`) and is still
redacted by `logging.redactSensitive`. Ful...(content truncated)

---

## Prometheus metrics - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/prometheus

[Skip to main content](https://docs.openclaw.ai/gateway/prometheus#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Health and diagnostics

Prometheus metrics

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/gateway/prometheus#quick-start)
- [Metrics exported](https://docs.openclaw.ai/gateway/prometheus#metrics-exported)
- [Label policy](https://docs.openclaw.ai/gateway/prometheus#label-policy)
- [PromQL recipes](https://docs.openclaw.ai/gateway/prometheus#promql-recipes)
- [Choosing between Prometheus and OpenTelemetry export](https://docs.openclaw.ai/gateway/prometheus#choosing-between-prometheus-and-opentelemetry-export)
- [Troubleshooting](https://docs.openclaw.ai/gateway/prometheus#troubleshooting)
- [Related](https://docs.openclaw.ai/gateway/prometheus#related)

OpenClaw can expose diagnostics metrics through the bundled `diagnostics-prometheus` plugin. It listens to trusted internal diagnostics and renders a Prometheus text endpoint at:

```
GET /api/diagnostics/prometheus
```

Content type is `text/plain; version=0.0.4; charset=utf-8`, the standard Prometheus exposition format.

The route uses Gateway authentication (operator scope). Do not expose it as a public unauthenticated `/metrics` endpoint. Scrape it through the same auth path you use for other operator APIs.

For traces, logs, OTLP push, and OpenTelemetry GenAI semantic attributes, see [OpenTelemetry export](https://docs.openclaw.ai/gateway/opentelemetry).

## [​](https://docs.openclaw.ai/gateway/prometheus\\#quick-start)  Quick start

1

[Navigate to header](https://docs.openclaw.ai/gateway/prometheus#)

Enable the plugin

- Config

- CLI


```
{
  plugins: {
    allow: ["diagnostics-prometheus"],
    entries: {
      "diagnostics-prometheus": { enabled: true },
    },
  },
  diagnostics: {
    enabled: true,
  },
}
```

```
openclaw plugins enable diagnostics-prometheus
```

2

[Navigate to header](https://docs.openclaw.ai/gateway/prometheus#)

Restart the Gateway

The HTTP route is registered at plugin startup, so reload after enabling.

3

[Navigate to header](https://docs.openclaw.ai/gateway/prometheus#)

Scrape the protected route

Send the same gateway auth your operator clients use:

```
curl -H \"Authorization: Bearer $OPENCLAW_GATEWAY_TOKEN\" \\
  http://127.0.0.1:18789/api/diagnostics/prometheus
```

4

[Navigate to header](https://docs.openclaw.ai/gateway/prometheus#)

Wire Prometheus

```
# prometheus.yml
scrape_configs:
  - job_name: openclaw
    scrape_interval: 30s
    metrics_path: /api/diagnostics/prometheus
    authorization:
      credentials_file: /etc/prometheus/openclaw-gateway-token
    static_configs:
      - targets: [\"openclaw-gateway:18789\"]
```

`diagnostics.enabled: true` is required. Without it, the plugin still registers the HTTP route but no diagnostic events flow into the exporter, so the response is empty.

## [​](https://docs.openclaw.ai/gateway/prometheus\\#metrics-exported)  Metrics exported

| Metric | Type | Labels |
| --- | --- | --- |
| `openclaw_run_completed_total` | counter | `channel`, `model`, `outcome`, `provider`, `trigger` |
| `openclaw_run_duration_seconds` | histogram | `channel`, `model`, `outcome`, `provider`, `trigger` |
| `openclaw_model_call_total` | counter | `api`, `error_category`, `model`, `outcome`, `provider`, `transport` |
| `openclaw_model_call_duration_seconds` | histogram | `api`, `error_category`, `model`, `outcome`, `provider`, `transport` |
| `openclaw_model_tokens_total` | counter | `agent`, `channel`, `model`, `provider`, `token_type` |
| `openclaw_gen_ai_client_token_usage` | histogram | `model`, `provider`, `token_type` |
| `openclaw_model_cost_usd_total` | counter | `agent`, `channel`, `model`, `provider` |
| `openclaw_tool_execution_total` | counter | `error_category`, `outcome`, `params_kind`, `tool` |
| `openclaw_tool_execution_duration_seconds` | histogram | `error_category`, `outcome`, `params_kind`, `tool` |
| `openclaw_harness_run_total` | counter | `channel`, `error_category`, `harness`, `model`, `outcome`, `phase`, `plugin`, `provider` |
| `openclaw_harness_run_duration_seconds` | histogram | `channel`, `error_category`, `harness`, `model`, `outcome`, `phase`, `plugin`, `provider` |
| `openclaw_message_processed_total` | counter | `channel`, `outcome`, `reason` |
| `openclaw_message_processed_duration_seconds` | histogram | `channel`, `outcome`, `reason` |
| `openclaw_message_delivery_total` | counter | `channel`, `delivery_kind`, `error_category`, `outcome` |
| `openclaw_message_delivery_duration_seconds` | histogram | `channel`, `delivery_kind`, `error_category`, `outcome` |
| `openclaw_queue_lane_size` | gauge | `lane` |
| `openclaw_queue_lane_wait_seconds` | histogram | `lane` |
| `openclaw_session_state_total` | counter | `reason`, `state` |
| `openclaw_session_queue_depth` | gauge | `state` |
| `openclaw_memory_bytes` | gauge | `kind` |
| `openclaw_memory_rss_bytes` | histogram | none |
| `openclaw_memory_pressure_total` | counter | `level`, `reason` |
| `openclaw_telemetry_exporter_total` | counter | `exporter`, `reason`, `signal`, `status` |
| `openclaw_prometheus_series_dropped_total` | counter | none |

## [​](https://docs.openclaw.ai/gateway/prometheus\\#label-policy)  Label policy

Bounded, low-cardinality labels

Prometheus labels stay bounded and low-cardinality. The exporter does not emit raw diagnostic identifiers such as `runId`, `sessionKey`, `sessionId`, `callId`, `toolCallId`, message IDs, chat IDs, or provider request IDs.Label values are redacted and must match OpenClaw’s low-cardinality character policy. Values that fail the policy are replaced with `unknown`, `other`, or `none`, depending on the metric.\n\nSeries cap and overflow accounting\n\nThe exporter caps retained time series in memory at **2048** series across counters, gauges, and histograms combined. New series beyond that cap are dropped, and `openclaw_prometheus_series_dropped_total` increments by one each time.Watch this counter as a hard signal that an attribute upstream is leaking high-cardinality values. The exporter never lifts the cap automatically; if it climbs, fix the source rather than disabling the cap.\n\nWhat never appears in Prometheus output\n\n- prompt text, response text, tool inputs, tool outputs, system prompts\n- raw provider request IDs (only bounded hashes, where applicable, on spans — never on metrics)\n- session keys and session IDs\n- hostnames, file paths, secret values\n\n## [​](https://openclaw.ai/gateway/prometheus\\#promql-recipes)  PromQL recipes

```
# Tokens per minute, split by provider
sum by (provider) (rate(openclaw_model_tokens_total[1m]))

# Spend (USD) over the last hour, by model
sum by (model) (increase(openclaw_model_cost_usd_total[1h]))

# 95th percentile model run duration
histogram_quantile(
  0.95,
  sum by (le, provider, model)
    (rate(openclaw_run_duration_seconds_bucket[5m]))
)

# Queue wait time SLO (95p under 2s)
histogram_quantile(
  0.95,
  sum by (le, lane) (rate(openclaw_queue_lane_wait_seconds_bucket[5m]))
) < 2

# Dropped Prometheus series (cardinality alarm)
increase(openclaw_prometheus_series_dropped_total[15m]) > 0
```

Prefer `gen_ai_client_token_usage` for cross-provider dashboards: it follows the OpenTelemetry GenAI semantic conventions and is consistent with metrics from non-OpenClaw GenAI services.\n\n## [​](https://docs.openclaw.ai/gateway/prometheus\\#choosing-between-prometheus-and-opentelemetry-export)  Choosing between Prometheus and OpenTelemetry export

OpenClaw supports both surfaces independently. You can run either, both, or neither.\n\n- diagnostics-prometheus\n\n- diagnostics-otel\n

- **Pull** model: Prometheus scrapes `/api/diagnostics/prometheus`.\n- No external collector required.\n- Authenticated through normal Gateway auth.\n- Surface is metrics only (no traces or logs).\n- Best for stacks already standardized on Prometheus + Grafana.\n
- **Push** model: OpenClaw sends OTLP/HTTP to a collector or OTLP-compatible backend.\n- Surface includes metrics, traces, and logs.\n- Bridges to Prometheus through an OpenTelemetry Collector (`prometheus` or `prometheusremotewrite` exporter) when you need both.\n- See [OpenTelemetry export](https://docs.openclaw.ai/gateway/opentelemetry) for the full catalog.\n
## [​](https://docs.openclaw.ai/gateway/prometheus\\#troubleshooting)  Troubleshooting

Empty response body\n\n- Check `diagnostics.enabled: true` in config.\n- Confirm the plugin is enabled and loaded with `openclaw plugins list --enabled`.\n- Generate some traffic; counters and histograms only emit lines after at least one event.\n
401 / unauthorized\n
The endpoint requires the Gateway operator scope (`auth: \"gateway\"` with `gatewayRuntimeScopeSurface: \"trusted-operator\"`). Use the same token or password Prometheus uses for any other Gateway operator route. There is no public unauthenticated mode.\n
\\`openclaw\\_prometheus\\_series\\_dropped\\_total\\` is climbing\n
A new attribute is exceeding the **2048**-series cap. Inspect recent metrics for an unexpectedly high-cardinality label and fix it at the source. The exporter intentionally drops new series instead of silently rewriting labels.\n
Prometheus shows stale series after a restart\n
The plugin keeps state in memory only. After a Gateway restart, counters reset to zero and gauges restart at their next reported value. Use PromQL `rate()` and `increase()` to handle resets cleanly.\n
## [​](https://docs.openclaw.ai/gateway/prometheus\\#related)  Related\n
- [Diagnostics export](https://docs.openclaw.ai/gateway/diagnostics) — local diagnostics zip for support bundles\n- [Health and readiness](https://docs.openclaw.ai/gateway/health) — `/healthz` and `/readyz` probes\n- [Logging](https://docs.openclaw.ai/logging) — file-based logging\n- [OpenTelemetry export](https://docs.openclaw.ai/gateway/opentelemetry) — OTLP push for traces, metrics, and logs\n
[OpenTelemetry export](https://docs.openclaw.ai/gateway/opentelemetry) [Gateway logging](https://docs.openclaw.ai/gateway/logging)\n
Ctrl+I

---

## Configuration — agents
**Source:** https://docs.openclaw.ai/gateway/config-agents

[Skip to main content](https://docs.openclaw.ai/gateway/config-agents#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Configuration

Configuration — agents

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Agent defaults](https://docs.openclaw.ai/gateway/config-agents#agent-defaults)
- [agents.defaults.workspace](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-workspace)
- [agents.defaults.repoRoot](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-reporoot)
- [agents.defaults.skills](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-skills)
- [agents.defaults.skipBootstrap](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-skipbootstrap)
- [agents.defaults.contextInjection](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-contextinjection)
- [agents.defaults.bootstrapMaxChars](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-bootstrapmaxchars)
- [agents.defaults.bootstrapTotalMaxChars](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-bootstraptotalmaxchars)
- [agents.defaults.bootstrapPromptTruncationWarning](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-bootstrapprompttruncationwarning)
- [Context budget ownership map](https://docs.openclaw.ai/gateway/config-agents#context-budget-ownership-map)
- [agents.defaults.startupContext](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-startupcontext)
- [agents.defaults.contextLimits](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-contextlimits)
- [agents.list\\[\\].contextLimits](https://docs.openclaw.ai/gateway/config-agents#agents-list-contextlimits)
- [skills.limits.maxSkillsPromptChars](https://docs.openclaw.ai/gateway/config-agents#skills-limits-maxskillspromptchars)
- [agents.list\\[\\].skillsLimits.maxSkillsPromptChars](https://docs.openclaw.ai/gateway/config-agents#agents-list-skillslimits-maxskillspromptchars)
- [agents.defaults.imageMaxDimensionPx](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-imagemaxdimensionpx)
- [agents.defaults.userTimezone](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-usertimezone)
- [agents.defaults.timeFormat](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-timeformat)
- [agents.defaults.model](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-model)
- [agents.defaults.agentRuntime](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-agentruntime)
- [agents.defaults.cliBackends](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-clibackends)
- [agents.defaults.systemPromptOverride](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-systempromptoverride)
- [agents.defaults.promptOverlays](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-promptoverlays)
- [agents.defaults.heartbeat](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-heartbeat)
- [agents.defaults.compaction](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-compaction)
- [agents.defaults.contextPruning](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-contextpruning)
- [Block streaming](https://docs.openclaw.ai/gateway/config-agents#block-streaming)
- [Typing indicators](https://docs.openclaw.ai/gateway/config-agents#typing-indicators)
- [agents.defaults.sandbox](https://docs.openclaw.ai/gateway/config-agents#agents-defaults-sandbox)
- [agents.list (per-agent overrides)](https://docs.openclaw.ai/gateway/config-agents#agents-list-per-agent-overrides)
- [Multi-agent routing](https://docs.openclaw.ai/gateway/config-agents#multi-agent-routing)
- [Binding match fields](https://docs.openclaw.ai/gateway/config-agents#binding-match-fields)
- [Per-agent access profiles](https://docs.openclaw.ai/gateway/config-agents#per-agent-access-profiles)
- [Session](https://docs.openclaw.ai/gateway/config-agents#session)
- [Messages](https://docs.openclaw.ai/gateway/config-agents#messages)
- [Response prefix](https://docs.openclaw.ai/gateway/config-agents#response-prefix)
- [Ack reaction](https://docs.openclaw.ai/gateway/config-agents#ack-reaction)
- [Inbound debounce](https://docs.openclaw.ai/gateway/config-agents#inbound-debounce)
- [TTS (text-to-speech)](https://docs.openclaw.ai/gateway/config-agents#tts-text-to-speech)
- [Talk](https://docs.openclaw.ai/gateway/config-agents#talk)
- [Related](https://docs.openclaw.ai/gateway/config-agents#related)

Agent-scoped configuration keys under `agents.*`, `multiAgent.*`, `session.*`,
`messages.*`, and `talk.*`. For channels, tools, gateway runtime, and other
top-level keys, see [Configuration reference](https://docs.openclaw.ai/gateway/configuration-reference).

## [​](https://docs.openclaw.ai/gateway/config-agents\\#agent-defaults)  Agent defaults

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-workspace)  `agents.defaults.workspace`

Default: `~/.openclaw/workspace`.

```
{
  agents: { defaults: { workspace: \"~/.openclaw/workspace\" } },
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-reporoot)  `agents.defaults.repoRoot`

Optional repository root shown in the system prompt’s Runtime line. If unset, OpenClaw auto-detects by walking upward from the workspace.

```
{
  agents: { defaults: { repoRoot: \"~/Projects/openclaw\" } },
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-skills)  `agents.defaults.skills`

Optional default skill allowlist for agents that do not set
`agents.list[].skills`.

```
{
  agents: {
    defaults: { skills: [\"github\", \"weather\"] },
    list: [\\\n      { id: \"writer\" }, // inherits github, weather\\\n      { id: \"docs\", skills: [\"docs-search\"] }, // replaces defaults\\\n      { id: \"locked-down\", skills: [] }, // no skills\\\n    ],
  },
}
```

- Omit `agents.defaults.skills` for unrestricted skills by default.
- Omit `agents.list[].skills` to inherit the defaults.
- Set `agents.list[].skills: []` for no skills.
- A non-empty `agents.list[].skills` list is the final set for that agent; it
does not merge with defaults.

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-skipbootstrap)  `agents.defaults.skipBootstrap`

Disables automatic creation of workspace bootstrap files (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-contextinjection)  `agents.defaults.contextInjection`

Controls when workspace bootstrap files are injected into the system prompt. Default: `\"always\"`.

- `\"continuation-skip\"`: safe continuation turns (after a completed assistant response) skip workspace bootstrap re-injection, reducing prompt size. Heartbeat runs and post-compaction retries still rebuild context.
- `\"never\"`: disable workspace bootstrap and context-file injection on every turn. Use this only for agents that fully own their prompt lifecycle (custom context engines, native runtimes that build their own context, or specialized bootstrap-free workflows). Heartbeat and compaction-recovery turns also skip injection.

```
{
  agents: { defaults: { contextInjection: \"continuation-skip\" } },
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-bootstrapmaxchars)  `agents.defaults.bootstrapMaxChars`

Max characters per workspace bootstrap file before truncation. Default: `12000`.

```
{
  agents: { defaults: { bootstrapMaxChars: 12000 } },
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-bootstraptotalmaxchars)  `agents.defaults.bootstrapTotalMaxChars`

Max total characters injected across all workspace bootstrap files. Default: `60000`.

```
{
  agents: { defaults: { bootstrapTotalMaxChars: 60000 } },
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-bootstrapprompttruncationwarning)  `agents.defaults.bootstrapPromptTruncationWarning`

Controls agent-visible warning text when bootstrap context is truncated.
Default: `\"once\"`.

- `\"off\"`: never inject warning text into the system prompt.
- `\"once\"`: inject warning once per unique truncation signature (recommended).
- `\"always\"`: inject warning on every run when truncation exists.

```
{
  agents: { defaults: { bootstrapPromptTruncationWarning: \"once\" } }, // off | once | always
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#context-budget-ownership-map)  Context budget ownership map

OpenClaw has multiple high-volume prompt/context budgets, and they are
intentionally split by subsystem instead of all flowing through one generic
knob.

- `agents.defaults.bootstrapMaxChars` /
`agents.defaults.bootstrapTotalMaxChars`:
normal workspace bootstrap injection.
- `agents.defaults.startupContext.*`:
one-shot `/new` and `/reset` startup prelude, including recent daily
`memory/*.md` files.
- `skills.limits.*`:
the compact skills list injected into the system prompt.
- `agents.defaults.contextLimits.*`:
bounded runtime excerpts and injected runtime-owned blocks.
- `memory.qmd.limits.*`:
indexed memory-search snippet and injection sizing.

Use the matching per-agent override only when one agent needs a different
budget:

- `agents.list[].skillsLimits.maxSkillsPromptChars`
- `agents.list[].contextLimits.*`

#### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-startupcontext)  `agents.defaults.startupContext`

Controls the first-turn startup prelude injected on bare `/new` and `/reset`
runs.

```
{
  agents: {
    defaults: {
      startupContext: {
        enabled: true,
        applyOn: [\"new\", \"reset\"],
        dailyMemoryDays: 2,
        maxFileBytes: 16384,
        maxFileChars: 1200,
        maxTotalChars: 2800,
      },
    },
  },
}
```

#### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-contextlimits)  `agents.defaults.contextLimits`

Shared defaults for bounded runtime context surfaces.

```
{
  agents: {
    defaults: {
      contextLimits: {
        memoryGetMaxChars: 12000,
        memoryGetDefaultLines: 120,
        toolResultMaxChars: 16000,
        postCompactionMaxChars: 1800,
      },
    },
  },
}
```

- `memoryGetMaxChars`: default `memory_get` excerpt cap before truncation
metadata and continuation notice are added.
- `memoryGetDefaultLines`: default `memory_get` line window when `lines` is
omitted.
- `toolResultMaxChars`: live tool-result cap used for persisted results and
overflow recovery.
- `postCompactionMaxChars`: AGENTS.md excerpt cap used during post-compaction
refresh injection.

#### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-list-contextlimits)  `agents.list[].contextLimits`

Per-agent override for the shared `contextLimits` knobs. Omitted fields inherit
from `agents.defaults.contextLimits`.

```
{
  agents: {
    defaults: {
      contextLimits: {
        memoryGetMaxChars: 12000,
        toolResultMaxChars: 16000,
      },
    },
    list: [\\\n      {\\\n        id: \"tiny-local\",\\\n        contextLimits: {\\\n          memoryGetMaxChars: 6000,\\\n          toolResultMaxChars: 8000,\\\n        },\\\n      },\\\n    ],
  },
}
```

#### [​](https://docs.openclaw.ai/gateway/config-agents\\#skills-limits-maxskillspromptchars)  `skills.limits.maxSkillsPromptChars`

Global cap for the compact skills list injected into the system prompt. This
does not affect reading `SKILL.md` files on demand.

```
{
  skills: {
    limits: {
      maxSkillsPromptChars: 18000,
    },
  },
}
```

#### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-list-skillslimits-maxskillspromptchars)  `agents.list[].skillsLimits.maxSkillsPromptChars`

Per-agent override for the skills prompt budget.

```
{
  agents: {
    list: [\\\n      {\\\n        id: \"tiny-local\",\\\n        skillsLimits: {\\\n          maxSkillsPromptChars: 6000,\\\n        },\\\n      },\\\n    ],
  },
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-imagemaxdimensionpx)  `agents.defaults.imageMaxDimensionPx`

Max pixel size for the longest image side in transcript/tool image blocks before provider calls.
Default: `1200`.Lower values usually reduce vision-token usage and request payload size for screenshot-heavy runs.
Higher values preserve more visual detail.

```
{
  agents: { defaults: { imageMaxDimensionPx: 1200 } },
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-usertimezone)  `agents.defaults.userTimezone`

Timezone for system prompt context (not message timestamps). Falls back to host timezone.

```
{
  agents: { defaults: { userTimezone: \"America/Chicago\" } },
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-timeformat)  `agents.defaults.timeFormat`

Time format in system prompt. Default: `auto` (OS preference).

```
{
  agents: { defaults: { timeFormat: \"auto\" } }, // auto | 12 | 24
}
```

### [​](https://docs.openclaw.ai/gateway/config-agents\\#agents-defaults-model)  `agents.defaults.model`

```
{
  agents: {
    defaults: {
      models: {
        \"anthropic/claude-opus-4-6\": { alias: \"opus\" },
        \"minimax/MiniMax-M2.7\": { alias: \"minimax\" },
      },
      model: {
        primary: \"anthropic/claude-opus-4-6\",
        fallbacks: [\"minimax/MiniMax-M2.7\"],
      },
      imageModel: {
        primary: \"openrouter/qwen/qwen-2.5-vl-72b-instruct:free\",
        fallbacks: [\"openrouter/google/gemini-2.0-flash-vision:free\"],
      },
      imageGenerationModel: {
        primary: \"openai/gpt-image-2\",
        fallbacks: [\"google/gemini-3.1-flash-image-preview\"],
      },
      videoGenerationModel: {
        primary: \"qwen/wan2.6-t2v\",
        fallbacks: [\"qwen/wan2.6-i2v\"],
      },
      pdfModel: {
        primary: \"anthropic/claude-opus-4-6\",
        fallbacks: [\"openai/gpt-5.4-mini\"],
      },
      params: { cacheRetention: \"long\" }, // global default provider params
      agentRuntime: {
        id: \"pi\", // pi | auto | registered harness id, e.g. codex
        fallback: \"pi\", // pi | none
      },
      pdfMaxBytesMb: 10,
      pdfMaxPages: 20,
      thinkingDefault: \"low\",
      verboseDefault: \"off\",
      elevatedDefault: \"on\",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model`: accepts either a string (`\"provider/model\"`) or an object (`{ primary, fallbacks }`).

  - String form sets only the primary model.
  - Object form sets primary plus ordered failover models.
- `imageModel`: accepts either a string (`\"provider/model\"`) or an object (`{ primary, fallbacks }`).

  - Used by the `image` tool path as its vision-model config.
  - Also used as fallback routing when the selected/default model cannot accept image input.
- `imageGenerationModel`: accepts either a string (`\"provider/model\"`) or an object (`{ primary, fallbacks }`).

  - Used by the shared image-generation capability and any future tool/plugin surface that generates images.
  - Typical values: `google/gemini-3.1-flash-image-preview` for native Gemini image generation, `fal/fal-ai/flux/dev` for fal, `openai/gpt-image-2` for OpenAI Images, or `openai/gpt-image-1.5` for transparent-background OpenAI PNG/WebP output.
  - If you select a provider/model directly, configure matching provider auth too (for example `GEMINI_API_KEY` or `GOOGLE_API_KEY` for `google/*`, `OPENAI_API_KEY` or OpenAI Codex OAuth for `openai/gpt-image-2` / `openai/gpt-image-1.5`, `FAL_KEY` for `fal/*`).
  - If omitted, `image_generate` can still infer an auth-backed provider default. It tries the current default provider first, then the remaining registered image-generation providers in provider-id order.
- `musicGenerationModel`: accepts either a string (`\"provider/model\"`) or an object (`{ primary, fallbacks }`).

  - Used by the shared music-generation capability and the built-in `music_generate` tool.
  - Typical values: `google/lyria-3-clip-preview`, `google/lyria-3-pro-preview`, or `minimax/music-2.6`.
  - If omitted, `music_generate` can still infer an auth-backed provider default. It tries the current default provider first, then the remaining registered music-generation providers in provider-id order.
  - If you select a provider/model directly, configure the matching provider auth/API key too.
- `videoGenerationModel`: accepts either a string (`\"provider/model\"`) or an object (`{ primary, fallbacks }`).

  - Used by the shared video-generation capability and the built-in `video_generate` tool.
  - Typical values: `qwen/wan2.6-t2v`, `qwen/wan2.6-i2v`, `qwen/wan2.6-r2v`, `qwen/wan2.6-r2v-flash`, or `qwen/wan2.7-r2v`.
  - If omitted, `video_generate` can still infer an auth-backed provider default. It tries the current default provider first, then the remaining registered video-generation providers in provider-id order.
  - If you select a provider/model directly, configure the matching provider auth/API key too.
  - The bundled Qwen video-generation provider supports up to 1 output video, 1 input image, 4 input videos, 10 seconds duration, and provider-level `size`, `aspectRatio`, `resolution`, `audio`, and `watermark` options.
- `pdfModel`: accepts either a string (`\"provider/model\"`) or an object (`{ primary, fallbacks }`).

  - Used by the `pdf` tool for model routing.
  - If omitted, the PDF tool falls back to `imageModel`, then to the resolved session/default model.
- `pdfMaxBytesMb`: default PDF size limit for the `pdf` tool when `maxBytesMb` is not passed at call time.
- `pdfMaxPages`: default maximum pages considered by extraction fallback mode in the `pdf` tool.
- `verboseDefault`: default verbose level for agents. Values: `\"off\"`, `\"on\"`, `\"full\"`. Default: `\"off\"`.
- `elevatedDefault`: default elevated-output level for agents. Values: `\"off\"`, `\"on\"`, `\"ask\"`, `\"full\"`. Default: `\"on\"`.
- `model.primary`: format `provider/model` (e.g. `openai/gpt-5.5` for API-key access or `openai-codex/gpt-5.5` for Codex OAuth). If you omit the provider, OpenClaw tries an alias first, then a unique configured-provider match for that exact model id, and only then falls back to the configured default provider (deprecated compatibility behavior...(content truncated)

---

## Health checks - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/health

[Skip to main content](https://docs.openclaw.ai/gateway/health#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Health and diagnostics

Health checks

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick checks](https://docs.openclaw.ai/gateway/health#quick-checks)
- [Deep diagnostics](https://docs.openclaw.ai/gateway/health#deep-diagnostics)
- [Health monitor config](https://docs.openclaw.ai/gateway/health#health-monitor-config)
- [When something fails](https://docs.openclaw.ai/gateway/health#when-something-fails)
- [Dedicated “health” command](https://docs.openclaw.ai/gateway/health#dedicated-%E2%80%9Chealth%E2%80%9D-command)
- [Related](https://docs.openclaw.ai/gateway/health#related)

Short guide to verify channel connectivity without guessing.

## [​](https://docs.openclaw.ai/gateway/health\\#quick-checks)  Quick checks

- `openclaw status` — local summary: gateway reachability/mode, update hint, linked channel auth age, sessions + recent activity.
- `openclaw status --all` — full local diagnosis (read-only, color, safe to paste for debugging).
- `openclaw status --deep` — asks the running gateway for a live health probe (`health` with `probe:true`), including per-account channel probes when supported.
- `openclaw health` — asks the running gateway for its health snapshot (WS-only; no direct channel sockets from the CLI).
- `openclaw health --verbose` — forces a live health probe and prints gateway connection details.
- `openclaw health --json` — machine-readable health snapshot output.
- Send `/status` as a standalone message in WhatsApp/WebChat to get a status reply without invoking the agent.
- Logs: tail `/tmp/openclaw/openclaw-*.log` and filter for `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound`.

## [​](https://docs.openclaw.ai/gateway/health\\#deep-diagnostics)  Deep diagnostics

- Creds on disk: `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (mtime should be recent).
- Session store: `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (path can be overridden in config). Count and recent recipients are surfaced via `status`.
- Relink flow: `openclaw channels logout && openclaw channels login --verbose` when status codes 409–515 or `loggedOut` appear in logs. (Note: the QR login flow auto-restarts once for status 515 after pairing.)
- Diagnostics are enabled by default. The gateway records operational facts unless `diagnostics.enabled: false` is set. Memory events record RSS/heap byte counts, threshold pressure, and growth pressure. Oversized-payload events record what was rejected, truncated, or chunked, plus sizes and limits when available. They do not record the message text, attachment contents, webhook body, raw request or response body, tokens, cookies, or secret values. The same heartbeat starts the bounded stability recorder, which is available through `openclaw gateway stability` or the `diagnostics.stability` Gateway RPC. Fatal Gateway exits, shutdown timeouts, and restart startup failures persist the latest recorder snapshot under `~/.openclaw/logs/stability/` when events exist; inspect the newest saved bundle with `openclaw gateway stability --bundle latest`.
- For bug reports, run `openclaw gateway diagnostics export` and attach the generated zip. The export combines a Markdown summary, the newest stability bundle, sanitized log metadata, sanitized Gateway status/health snapshots, and config shape. It is meant to be shared: chat text, webhook bodies, tool outputs, credentials, cookies, account/message identifiers, and secret values are omitted or redacted. See [Diagnostics Export](https://docs.openclaw.ai/gateway/diagnostics).

## [​](https://docs.openclaw.ai/gateway/health\\#health-monitor-config)  Health monitor config

- `gateway.channelHealthCheckMinutes`: how often the gateway checks channel health. Default: `5`. Set `0` to disable health-monitor restarts globally.
- `gateway.channelStaleEventThresholdMinutes`: how long a connected channel can stay idle before the health monitor treats it as stale and restarts it. Default: `30`. Keep this greater than or equal to `gateway.channelHealthCheckMinutes`.
- `gateway.channelMaxRestartsPerHour`: rolling one-hour cap for health-monitor restarts per channel/account. Default: `10`.
- `channels.<provider>.healthMonitor.enabled`: disable health-monitor restarts for a specific channel while leaving global monitoring enabled.
- `channels.<provider>.accounts.<accountId>.healthMonitor.enabled`: multi-account override that wins over the channel-level setting.
- These per-channel overrides apply to the built-in channel monitors that expose them today: Discord, Google Chat, iMessage, Microsoft Teams, Signal, Slack, Telegram, and WhatsApp.

## [​](https://docs.openclaw.ai/gateway/health\\#when-something-fails)  When something fails

- `logged out` or status 409–515 → relink with `openclaw channels logout` then `openclaw channels login`.
- Gateway unreachable → start it: `openclaw gateway --port 18789` (use `--force` if the port is busy).
- No inbound messages → confirm linked phone is online and the sender is allowed (`channels.whatsapp.allowFrom`); for group chats, ensure allowlist + mention rules match (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

## [​](https://docs.openclaw.ai/gateway/health\\#dedicated-%E2%80%9Chealth%E2%80%9D-command)  Dedicated “health” command

`openclaw health` asks the running gateway for its health snapshot (no direct channel\nsockets from the CLI). By default it can return a fresh cached gateway snapshot; the\ngateway then refreshes that cache in the background. `openclaw health --verbose` forces\na live probe instead. The command reports linked creds/auth age when available,\nper-channel probe summaries, session-store summary, and a probe duration. It exits\nnon-zero if the gateway is unreachable or the probe fails/timeouts.Options:\n\n- `--json`: machine-readable JSON output\n- `--timeout <ms>`: override the default 10s probe timeout\n- `--verbose`: force a live probe and print gateway connection details\n- `--debug`: alias for `--verbose`\n\nThe health snapshot includes: `ok` (boolean), `ts` (timestamp), `durationMs` (probe time), per-channel status, agent availability, and session-store summary.\n
## [​](https://docs.openclaw.ai/gateway/health\\#related)  Related\n
- [Gateway runbook](https://docs.openclaw.ai/gateway)\n- [Diagnostics export](https://docs.openclaw.ai/gateway/diagnostics)\n- [Gateway troubleshooting](https://docs.openclaw.ai/gateway/troubleshooting)\n\n[Trusted proxy auth](https://docs.openclaw.ai/gateway/trusted-proxy-auth) [Heartbeat](https://docs.openclaw.ai/gateway/heartbeat)\n\nCtrl+I

---

## Bridge protocol - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/bridge-protocol

[Skip to main content](https://docs.openclaw.ai/gateway/bridge-protocol#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Protocols and APIs

Bridge protocol

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Why it existed](https://docs.openclaw.ai/gateway/bridge-protocol#why-it-existed)
- [Transport](https://docs.openclaw.ai/gateway/bridge-protocol#transport)
- [Handshake + pairing](https://docs.openclaw.ai/gateway/bridge-protocol#handshake-%2B-pairing)
- [Frames](https://docs.openclaw.ai/gateway/bridge-protocol#frames)
- [Exec lifecycle events](https://docs.openclaw.ai/gateway/bridge-protocol#exec-lifecycle-events)
- [Historical tailnet usage](https://docs.openclaw.ai/gateway/bridge-protocol#historical-tailnet-usage)
- [Versioning](https://docs.openclaw.ai/gateway/bridge-protocol#versioning)
- [Related](https://docs.openclaw.ai/gateway/bridge-protocol#related)

The TCP bridge has been **removed**. Current OpenClaw builds do not ship the bridge listener and `bridge.*` config keys are no longer in the schema. This page is kept for historical reference only. Use the [Gateway Protocol](https://docs.openclaw.ai/gateway/protocol) for all node/operator clients.

## [​](https://docs.openclaw.ai/gateway/bridge-protocol\\#why-it-existed)  Why it existed

- **Security boundary**: the bridge exposes a small allowlist instead of the
full gateway API surface.
- **Pairing + node identity**: node admission is owned by the gateway and tied
to a per-node token.
- **Discovery UX**: nodes can discover gateways via Bonjour on LAN, or connect
directly over a tailnet.
- **Loopback WS**: the full WS control plane stays local unless tunneled via SSH.

## [​](https://docs.openclaw.ai/gateway/bridge-protocol\\#transport)  Transport

- TCP, one JSON object per line (JSONL).
- Optional TLS (when `bridge.tls.enabled` is true).
- Historical default listener port was `18790` (current builds do not start a
TCP bridge).

When TLS is enabled, discovery TXT records include `bridgeTls=1` plus
`bridgeTlsSha256` as a non-secret hint. Note that Bonjour/mDNS TXT records are
unauthenticated; clients must not treat the advertised fingerprint as an
authoritative pin without explicit user intent or other out-of-band verification.

## [​](https://docs.openclaw.ai/gateway/bridge-protocol\\#handshake-+-pairing)  Handshake + pairing

1. Client sends `hello` with node metadata + token (if already paired).
2. If not paired, gateway replies `error` (`NOT_PAIRED`/`UNAUTHORIZED`).
3. Client sends `pair-request`.
4. Gateway waits for approval, then sends `pair-ok` and `hello-ok`.

Historically, `hello-ok` returned `serverName` and could include
`canvasHostUrl`.

## [​](https://docs.openclaw.ai/gateway/bridge-protocol\\#frames)  Frames

Client → Gateway:

- `req` / `res`: scoped gateway RPC (chat, sessions, config, health, voicewake, skills.bins)
- `event`: node signals (voice transcript, agent request, chat subscribe, exec lifecycle)

Gateway → Client:

- `invoke` / `invoke-res`: node commands (`canvas.*`, `camera.*`, `screen.record`,
`location.get`, `sms.send`)
- `event`: chat updates for subscribed sessions
- `ping` / `pong`: keepalive

Legacy allowlist enforcement lived in `src/gateway/server-bridge.ts` (removed).

## [​](https://docs.openclaw.ai/gateway/bridge-protocol\\#exec-lifecycle-events)  Exec lifecycle events

Nodes can emit `exec.finished` or `exec.denied` events to surface system.run activity.
These are mapped to system events in the gateway. (Legacy nodes may still emit `exec.started`.)Payload fields (all optional unless noted):

- `sessionKey` (required): agent session to receive the system event.
- `runId`: unique exec id for grouping.
- `command`: raw or formatted command string.
- `exitCode`, `timedOut`, `success`, `output`: completion details (finished only).
- `reason`: denial reason (denied only).

## [​](https://docs.openclaw.ai/gateway/bridge-protocol\\#historical-tailnet-usage)  Historical tailnet usage

- Bind the bridge to a tailnet IP: `bridge.bind: \"tailnet\"` in
`~/.openclaw/openclaw.json` (historical only; `bridge.*` is no longer valid).
- Clients connect via MagicDNS name or tailnet IP.
- Bonjour does **not** cross networks; use manual host/port or wide-area DNS‑SD
when needed.

## [​](https://docs.openclaw.ai/gateway/bridge-protocol\\#versioning)  Versioning

The bridge was **implicit v1** (no min/max negotiation). This section is
historical reference only; current node/operator clients use the WebSocket
[Gateway Protocol](https://docs.openclaw.ai/gateway/protocol).

## [​](https://docs.openclaw.ai/gateway/bridge-protocol\\#related)  Related

- [Gateway protocol](https://docs.openclaw.ai/gateway/protocol)
- [Nodes](https://docs.openclaw.ai/nodes)

[Gateway protocol](https://docs.openclaw.ai/gateway/protocol) [OpenAI chat completions](https://docs.openclaw.ai/gateway/openai-http-api)

Ctrl+I

---

## Troubleshooting
**Source:** https://docs.openclaw.ai/gateway/troubleshooting

[Skip to main content](https://docs.openclaw.ai/gateway/troubleshooting#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Health and diagnostics

Troubleshooting

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Command ladder](https://docs.openclaw.ai/gateway/troubleshooting#command-ladder)
- [Split brain installs and newer config guard](https://docs.openclaw.ai/gateway/troubleshooting#split-brain-installs-and-newer-config-guard)
- [Anthropic 429 extra usage required for long context](https://docs.openclaw.ai/gateway/troubleshooting#anthropic-429-extra-usage-required-for-long-context)
- [Local OpenAI-compatible backend passes direct probes but agent runs fail](https://docs.openclaw.ai/gateway/troubleshooting#local-openai-compatible-backend-passes-direct-probes-but-agent-runs-fail)
- [No replies](https://docs.openclaw.ai/gateway/troubleshooting#no-replies)
- [Dashboard control UI connectivity](https://docs.openclaw.ai/gateway/troubleshooting#dashboard-control-ui-connectivity)
- [Auth detail codes quick map](https://docs.openclaw.ai/gateway/troubleshooting#auth-detail-codes-quick-map)
- [Gateway service not running](https://docs.openclaw.ai/gateway/troubleshooting#gateway-service-not-running)
- [Gateway restored last-known-good config](https://docs.openclaw.ai/gateway/troubleshooting#gateway-restored-last-known-good-config)
- [Gateway probe warnings](https://docs.openclaw.ai/gateway/troubleshooting#gateway-probe-warnings)
- [Channel connected, messages not flowing](https://docs.openclaw.ai/gateway/troubleshooting#channel-connected-messages-not-flowing)
- [Cron and heartbeat delivery](https://docs.openclaw.ai/gateway/troubleshooting#cron-and-heartbeat-delivery)
- [Node paired, tool fails](https://docs.openclaw.ai/gateway/troubleshooting#node-paired-tool-fails)
- [Browser tool fails](https://docs.openclaw.ai/gateway/troubleshooting#browser-tool-fails)
- [If you upgraded and something suddenly broke](https://docs.openclaw.ai/gateway/troubleshooting#if-you-upgraded-and-something-suddenly-broke)
- [Related](https://docs.openclaw.ai/gateway/troubleshooting#related)

This page is the deep runbook. Start at [/help/troubleshooting](https://docs.openclaw.ai/help/troubleshooting) if you want the fast triage flow first.

## [​](https://docs.openclaw.ai/gateway/troubleshooting\\#command-ladder)  Command ladder

Run these first, in this order:

```
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Expected healthy signals:

- `openclaw gateway status` shows `Runtime: running`, `Connectivity probe: ok`, and a `Capability: ...` line.
- `openclaw doctor` reports no blocking config/service issues.
- `openclaw channels status --probe` shows live per-account transport status and, where supported, probe/audit results such as `works` or `audit ok`.

## [​](https://docs.openclaw.ai/gateway/troubleshooting\\#split-brain-installs-and-newer-config-guard)  Split brain installs and newer config guard

Use this when a gateway service unexpectedly stops after an update, or logs show that one `openclaw` binary is older than the version that last wrote `openclaw.json`.OpenClaw stamps config writes with `meta.lastTouchedVersion`. Read-only commands can still inspect a config written by a newer OpenClaw, but process and service mutations refuse to continue from an older binary. Blocked actions include gateway service start, stop, restart, uninstall, forced service reinstall, service-mode gateway startup, and `gateway --force` port cleanup.

```
which openclaw
openclaw --version
openclaw gateway status --deep
openclaw config get meta.lastTouchedVersion
```

1

[Navigate to header](https://docs.openclaw.ai/gateway/troubleshooting#)

Fix PATH

Fix `PATH` so `openclaw` resolves to the newer install, then rerun the action.

2

[Navigate to header](https://docs.openclaw.ai/gateway/troubleshooting#)

Reinstall the gateway service

Reinstall the intended gateway service from the newer install:

```
openclaw gateway install --force
openclaw gateway restart
```

3

[Navigate to header](https://docs.openclaw.ai/gateway/troubleshooting#)

Remove stale wrappers

Remove stale system package or old wrapper entries that still point at an old `openclaw` binary.

For intentional downgrade or emergency recovery only, set `OPENCLAW_ALLOW_OLDER_BINARY_DESTRUCTIVE_ACTIONS=1` for the single command. Leave it unset for normal operation.

## [​](https://docs.openclaw.ai/gateway/troubleshooting\\#anthropic-429-extra-usage-required-for-long-context)  Anthropic 429 extra usage required for long context

Use this when logs/errors include: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`.

```
openclaw logs --follow
openclaw models status
openclaw config get agents.defaults.models
```

Look for:

- Selected Anthropic Opus/Sonnet model has `params.context1m: true`.
- Current Anthropic credential is not eligible for long-context usage.
- Requests fail only on long sessions/model runs that need the 1M beta path.

Fix options:

1

[Navigate to header](https://docs.openclaw.ai/gateway/troubleshooting#)

Disable context1m

Disable `context1m` for that model to fall back to the normal context window.

2

[Navigate to header](https://docs.openclaw.ai/gateway/troubleshooting#)

Use an eligible credential

Use an Anthropic credential that is eligible for long-context requests, or switch to an Anthropic API key.

3

[Navigate to header](https://docs.openclaw.ai/gateway/troubleshooting#)

Configure fallback models

Configure fallback models so runs continue when Anthropic long-context requests are rejected.

Related:

- [Anthropic](https://docs.openclaw.ai/providers/anthropic)
- [Token use and costs](https://docs.openclaw.ai/reference/token-use)
- [Why am I seeing HTTP 429 from Anthropic?](https://docs.openclaw.ai/help/faq-first-run#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)

## [​](https://docs.openclaw.ai/gateway/troubleshooting\\#local-openai-compatible-backend-passes-direct-probes-but-agent-runs-fail)  Local OpenAI-compatible backend passes direct probes but agent runs fail

Use this when:

- `curl ... /v1/models` works
- tiny direct `/v1/chat/completions` calls work
- OpenClaw model runs fail only on normal agent turns

```
curl http://127.0.0.1:1234/v1/models
curl http://127.0.0.1:1234/v1/chat/completions \\
  -H 'content-type: application/json' \\
  -d '{"model":"<id>","messages":[{"role":"user","content":"hi"}],"stream":false}'
openclaw infer model run --model <provider/model> --prompt "hi" --json
openclaw logs --follow
```

Look for:

- direct tiny calls succeed, but OpenClaw runs fail only on larger prompts
- `model_not_found` or 404 errors even though direct `/v1/chat/completions`
works with the same bare model id
- backend errors about `messages[].content` expecting a string
- intermittent `incomplete turn detected ... stopReason=stop payloads=0` warnings with an OpenAI-compatible local backend
- backend crashes that appear only with larger prompt-token counts or full agent runtime prompts

Common signatures

- `model_not_found` with a local MLX/vLLM-style server → verify `baseUrl` includes `/v1`, `api` is `\"openai-completions\"` for `/v1/chat/completions` backends, and `models.providers.<provider>.models[].id` is the bare provider-local id. Select it with the provider prefix once, for example `mlx/mlx-community/Qwen3-30B-A3B-6bit`; keep the catalog entry as `mlx-community/Qwen3-30B-A3B-6bit`.
- `messages[...].content: invalid type: sequence, expected a string` → backend rejects structured Chat Completions content parts. Fix: set `models.providers.<provider>.models[].compat.requiresStringContent: true`.
- `incomplete turn detected ... stopReason=stop payloads=0` → the backend completed the Chat Completions request but returned no user-visible assistant text for that turn. OpenClaw retries replay-safe empty OpenAI-compatible turns once; persistent failures usually mean the backend is emitting empty/non-text content or suppressing final-answer text.
- direct tiny requests succeed, but OpenClaw agent runs fail with backend/model crashes (for example Gemma on some `inferrs` builds) → OpenClaw transport is likely already correct; the backend is failing on the larger agent-runtime prompt shape.
- failures shrink after disabling tools but do not disappear → tool schemas were part of the pressure, but the remaining issue is still upstream model/server capacity or a backend bug.

Fix options

1. Set `compat.requiresStringContent: true` for string-only Chat Completions backends.
2. Set `compat.supportsTools: false` for models/backends that cannot handle OpenClaw’s tool schema surface reliably.
3. Lower prompt pressure where possible: smaller workspace bootstrap, shorter session history, lighter local model, or a backend with stronger long-context support.
4. If tiny direct requests keep passing while OpenClaw agent turns still crash inside the backend, treat it as an upstream server/model limitation and file a repro there with the accepted payload shape.

Related:

- [Configuration](https://docs.openclaw.ai/gateway/configuration)
- [Local models](https://docs.openclaw.ai/gateway/local-models)
- [OpenAI-compatible endpoints](https://docs.openclaw.ai/gateway/configuration-reference#openai-compatible-endpoints)

## [​](https://docs.openclaw.ai/gateway/troubleshooting\\#no-replies)  No replies

If channels are up but nothing answers, check routing and policy before reconnecting anything.

```
openclaw status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw config get channels
openclaw logs --follow
```

Look for:

- Pairing pending for DM senders.
- Group mention gating (`requireMention`, `mentionPatterns`).
- Channel/group allowlist mismatches.

Common signatures:

- `drop guild message (mention required` → group message ignored until mention.
- `pairing request` → sender needs approval.
- `blocked` / `allowlist` → sender/channel was filtered by policy.

Related:

- [Channel troubleshooting](https://docs.openclaw.ai/channels/troubleshooting)
- [Groups](https://docs.openclaw.ai/channels/groups)
- [Pairing](https://docs.openclaw.ai/channels/pairing)

## [​](https://docs.openclaw.ai/gateway/troubleshooting\\#dashboard-control-ui-connectivity)  Dashboard control UI connectivity

When dashboard/control UI will not connect, validate URL, auth mode, and secure context assumptions.

```
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

Look for:

- Correct probe URL and dashboard URL.
- Auth mode/token mismatch between client and gateway.
- HTTP usage where device identity is required.

Connect / auth signatures

- `device identity required` → non-secure context or missing device auth.
- `origin not allowed` → browser `Origin` is not in `gateway.controlUi.allowedOrigins` (or you are connecting from a non-loopback browser origin without an explicit allowlist).
- `device nonce required` / `device nonce mismatch` → client is not completing the challenge-based device auth flow (`connect.challenge` \\+ `device.nonce`).
- `device signature invalid` / `device signature expired` → client signed the wrong payload (or stale timestamp) for the current handshake.
- `AUTH_TOKEN_MISMATCH` with `canRetryWithDeviceToken=true` → client can do one trusted retry with cached device token.
- That cached-token retry reuses the cached scope set stored with the paired device token. Explicit `deviceToken` / explicit `scopes` callers keep their requested scope set instead.
- Outside that retry path, connect auth precedence is explicit shared token/password first, then explicit `deviceToken`, then stored device token, then bootstrap token.
- On the async Tailscale Serve Control UI path, failed attempts for the same `{scope, ip}` are serialized before the limiter records the failure. Two bad concurrent retries from the same client can therefore surface `retry later` on the second attempt instead of two plain mismatches.
- `too many failed authentication attempts (retry later)` from a browser-origin loopback client → repeated failures from that same normalized `Origin` are locked out temporarily; another localhost origin uses a separate bucket.
- repeated `unauthorized` after that retry → shared token/device token drift; refresh token config and re-approve/rotate device token if needed.
- `gateway connect failed:` → wrong host/port/url target.

### [​](https://docs.openclaw.ai/gateway/troubleshooting\\#auth-detail-codes-quick-map)  Auth detail codes quick map

| Detail code | Meaning | Recommended action |
| --- | --- | --- |
| `AUTH_TOKEN_MISSING` | Client did not send a required shared token. | Paste/set token in the client and retry. For dashboard paths: `openclaw config get gateway.auth.token` then paste into Control UI settings. |
| `AUTH_TOKEN_MISMATCH` | Shared token did not match gateway auth token. | If `canRetryWithDeviceToken=true`, allow one trusted retry. Cached-token retries reuse stored approved scopes; explicit `deviceToken` / `scopes` callers keep requested scopes. If still failing, run the [token drift recovery checklist](https://docs.openclaw.ai/cli/devices#token-drift-recovery-checklist). |
| `AUTH_DEVICE_TOKEN_MISMATCH` | Cached per-device token is stale or revoked. | Rotate/re-approve device token using [devices CLI](https://docs.openclaw.ai/cli/devices), then reconnect. |
| `PAIRING_REQUIRED` | Device identity needs approval. Check `error.details.reason` for `not-paired`, `scope-upgrade`, `role-upgrade`, or `metadata-upgrade`, and use `requestId` / `remediationHint` when present. | Approve pending request: `openclaw devices list` then `openclaw devices approve <requestId>`. Scope/role upgrades use the same flow after you review the requested access. |

Direct loopback backend RPCs authenticated with the shared gateway token/password should not depend on the CLI’s paired-device scope baseline. If subagents or other internal calls still fail with `scope-upgrade`, verify the caller is using `client.id: \"gateway-client\"` and `client.mode: \"backend\"` and is not forcing an explicit `deviceIdentity` or device token.

Device auth v2 migration check:

```
openclaw --version
openclaw doctor
openclaw gateway status
```

If logs show nonce/signature errors, update the connecting client and verify it:

1

[Navigate to header](https://docs.openclaw.ai/gateway/troubleshooting#)

Wait for connect.challenge

Client waits for the gateway-issued `connect.challenge`.

2

[Navigate to header](https://docs.openclaw.ai/gateway/troubleshooting#)

Sign the payload

Client signs the challenge-bound payload.

3

[Navigate to header](https://docs.openclaw.ai/gateway/troubleshooting#)

Send the device nonce

Client sends `connect.params.device.nonce` with the same challenge nonce.

If `openclaw devices rotate` / `revoke` / `remove` is denied unexpectedly:

- paired-device token sessions can manage only **their own** device unless the caller also has `operator.admin`
- `openclaw devices rotate --scope ...` can only request operator scopes that the caller session already holds

Related:

- [Configuration](https://docs.openclaw.ai/gateway/configuration) (gateway auth modes)
- [Control UI](https://docs.openclaw.ai/web/control-ui)
- [Devices](https://docs.openclaw.ai/cli/devices)
- [Remote access](https://docs.openclaw.ai/gateway/remote)
- [Trusted proxy auth](https://docs.openclaw.ai/gateway/trusted-proxy-auth)

## [​](https://docs.openclaw.ai/gateway/troubleshooting\\#gateway-service-not-running)  Gateway service not running

Use this when service is installed but process does not stay up.

```
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep   # also scan system-level services
```

Look for:

- `Runtime: stopped` with exit hints.
- Service config mismatch (`Config (cli)` vs `Config (service)`).
- Port/listener conflicts.
- Extra launchd/systemd/schtasks installs when `--deep` is used.
- `Other gateway-like services detected (best effort)` cleanup hints.

Common signatures

- `Gateway start blocked: set gateway.mode=local` or `existing config is missing gateway.mode` → local gateway mode is not enabled, or the config file was clobbered and lost `gateway.mode`. Fix: set `gateway.mode=\"local\"` in your config, or re-run `openclaw onboard --mode local` / `openclaw setup` to restamp the expected local-mode config. If you are running OpenClaw via Podman, the default config path is `~/.openclaw/openclaw.json`.
- `refusing to bind gateway ... without auth` → non-loopback bind without a valid gateway auth path (token/password, or trusted-proxy where configured).
- `another gateway instance is already listening` / `EADDRINUSE` → port conflict.
- `Other gateway-like services detected (best effort)` → stale or parallel launchd/systemd/schtasks units exist. Most setups should keep one gateway per machine; if you do need more than one, isolate ports + config/state/workspace. See [/gateway#multiple-gateways-same-host](https://docs.openclaw.ai/gateway#multiple-gateways-same-host).
- `System-level OpenClaw gateway service detected` from doctor → a systemd system unit exists while the user-level service is missing. Remove or disable the duplicate before allowing doctor to install a user service, or set `OPENCLAW_SERVICE_REPAIR_POLICY=external` if the system unit is the intended supervisor.
- `Gateway service port does not match current gateway config` → the installed supervisor still pins the old `--port`. Run `openclaw doctor --fix` or `openclaw gateway install --force`, then restart the gateway service.

Related:

- [Background exec and process tool](https://docs.openclaw.ai/gateway/background-process)
- [Configuration](https://docs.openclaw.ai/gateway/configuration)
- [Doctor](https://docs.openclaw.ai/gateway/doctor)

## [​](https://docs.openclaw.ai/gateway/troubleshooting\\#gateway-restored-last-known-good-config)  Gateway restored last-known-good config

Use this when the Gateway starts, but logs say it restored `openclaw.json`.

```
openclaw logs --follow
openclaw config file
openclaw config validate
openclaw doctor
```

Look for:

- `Config auto-restored from last-known-good`
- `g...(content truncated)

---

## Trusted proxy auth
**Source:** https://docs.openclaw.ai/gateway/trusted-proxy-auth

[Skip to main content](https://docs.openclaw.ai/gateway/trusted-proxy-auth#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Authentication and secrets

Trusted proxy auth

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [When to use](https://docs.openclaw.ai/gateway/trusted-proxy-auth#when-to-use)
- [When NOT to use](https://docs.openclaw.ai/gateway/trusted-proxy-auth#when-not-to-use)
- [How it works](https://docs.openclaw.ai/gateway/trusted-proxy-auth#how-it-works)
- [Control UI pairing behavior](https://docs.openclaw.ai/gateway/trusted-proxy-auth#control-ui-pairing-behavior)
- [Configuration](https://docs.openclaw.ai/gateway/trusted-proxy-auth#configuration)
- [Configuration reference](https://docs.openclaw.ai/gateway/trusted-proxy-auth#configuration-reference)
- [TLS termination and HSTS](https://docs.openclaw.ai/gateway/trusted-proxy-auth#tls-termination-and-hsts)
- [Rollout guidance](https://docs.openclaw.ai/gateway/trusted-proxy-auth#rollout-guidance)
- [Proxy setup examples](https://docs.openclaw.ai/gateway/trusted-proxy-auth#proxy-setup-examples)
- [Mixed token configuration](https://docs.openclaw.ai/gateway/trusted-proxy-auth#mixed-token-configuration)
- [Operator scopes header](https://docs.openclaw.ai/gateway/trusted-proxy-auth#operator-scopes-header)
- [Security checklist](https://docs.openclaw.ai/gateway/trusted-proxy-auth#security-checklist)
- [Security audit](https://docs.openclaw.ai/gateway/trusted-proxy-auth#security-audit)
- [Troubleshooting](https://docs.openclaw.ai/gateway/trusted-proxy-auth#troubleshooting)
- [Migration from token auth](https://docs.openclaw.ai/gateway/trusted-proxy-auth#migration-from-token-auth)
- [Related](https://docs.openclaw.ai/gateway/trusted-proxy-auth#related)

**Security-sensitive feature.** This mode delegates authentication entirely to your reverse proxy. Misconfiguration can expose your Gateway to unauthorized access. Read this page carefully before enabling.

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#when-to-use)  When to use

Use `trusted-proxy` auth mode when:

- You run OpenClaw behind an **identity-aware proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth).
- Your proxy handles all authentication and passes user identity via headers.
- You’re in a Kubernetes or container environment where the proxy is the only path to the Gateway.
- You’re hitting WebSocket `1008 unauthorized` errors because browsers can’t pass tokens in WS payloads.

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#when-not-to-use)  When NOT to use

- If your proxy doesn’t authenticate users (just a TLS terminator or load balancer).
- If there’s any path to the Gateway that bypasses the proxy (firewall holes, internal network access).
- If you’re unsure whether your proxy correctly strips/overwrites forwarded headers.
- If you only need personal single-user access (consider Tailscale Serve + loopback for simpler setup).

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#how-it-works)  How it works

1

[Navigate to header](https://docs.openclaw.ai/gateway/trusted-proxy-auth#)

Proxy authenticates the user

Your reverse proxy authenticates users (OAuth, OIDC, SAML, etc.).

2

[Navigate to header](https://docs.openclaw.ai/gateway/trusted-proxy-auth#)

Proxy adds an identity header

Proxy adds a header with the authenticated user identity (e.g., `x-forwarded-user: nick@example.com`).

3

[Navigate to header](https://docs.openclaw.ai/gateway/trusted-proxy-auth#)

Gateway verifies trusted source

OpenClaw checks that the request came from a **trusted proxy IP** (configured in `gateway.trustedProxies`).

4

[Navigate to header](https://docs.openclaw.ai/gateway/trusted-proxy-auth#)

Gateway extracts identity

OpenClaw extracts the user identity from the configured header.

5

[Navigate to header](https://docs.openclaw.ai/gateway/trusted-proxy-auth#)

Authorize

If everything checks out, the request is authorized.

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#control-ui-pairing-behavior)  Control UI pairing behavior

When `gateway.auth.mode = \"trusted-proxy\"` is active and the request passes trusted-proxy checks, Control UI WebSocket sessions can connect without device pairing identity.Implications:

- Pairing is no longer the primary gate for Control UI access in this mode.
- Your reverse proxy auth policy and `allowUsers` become the effective access control.
- Keep gateway ingress locked to trusted proxy IPs only (`gateway.trustedProxies` \\+ firewall).

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#configuration)  Configuration

```
{
  gateway: {
    // Trusted-proxy auth expects requests from a non-loopback trusted proxy source by default
    bind: \"lan\",

    // CRITICAL: Only add your proxy\'s IP(s) here
    trustedProxies: [\"10.0.0.1\", \"172.17.0.1\"],

    auth: {
      mode: \"trusted-proxy\",
      trustedProxy: {
        // Header containing authenticated user identity (required)
        userHeader: \"x-forwarded-user\",

        // Optional: headers that MUST be present (proxy verification)
        requiredHeaders: [\"x-forwarded-proto\", \"x-forwarded-host\"],

        // Optional: restrict to specific users (empty = allow all)
        allowUsers: [\"nick@example.com\", \"admin@company.org\"],

        // Optional: allow a same-host loopback proxy after explicit opt-in
        allowLoopback: false,
      },
    },
  },
}
```

**Important runtime rules**

- Trusted-proxy auth rejects loopback-source requests (`127.0.0.1`, `::1`, loopback CIDRs) by default.
- Same-host loopback reverse proxies do **not** satisfy trusted-proxy auth unless you explicitly set `gateway.auth.trustedProxy.allowLoopback = true` and include the loopback address in `gateway.trustedProxies`.
- `allowLoopback` trusts local processes on the Gateway host to the same degree as the reverse proxy. Enable it only when the Gateway is still firewalled from direct remote access and the local proxy strips or overwrites client-supplied identity headers.
- Internal Gateway clients that do not travel through the reverse proxy should use `gateway.auth.password` / `OPENCLAW_GATEWAY_PASSWORD`, not trusted-proxy identity headers.
- Non-loopback Control UI deployments still need explicit `gateway.controlUi.allowedOrigins`.
- **Forwarded-header evidence overrides loopback locality for local direct fallback.** If a request arrives on loopback but carries `X-Forwarded-For` / `X-Forwarded-Host` / `X-Forwarded-Proto` headers pointing at a non-local origin, that evidence disqualifies local-direct password fallback and device-identity gating. With `allowLoopback: true`, trusted-proxy auth can still accept the request as a same-host proxy request, while `requiredHeaders` and `allowUsers` continue to apply.

### [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#configuration-reference)  Configuration reference

[​](https://docs.openclaw.ai/gateway/trusted-proxy-auth#param-gateway-trusted-proxies)

gateway.trustedProxies

string\\[\\]

required

Array of proxy IP addresses to trust. Requests from other IPs are rejected.

[​](https://docs.openclaw.ai/gateway/trusted-proxy-auth#param-gateway-auth-mode)

gateway.auth.mode

string

required

Must be `\"trusted-proxy\"`.

[​](https://docs.openclaw.ai/gateway/trusted-proxy-auth#param-gateway-auth-trusted-proxy-user-header)

gateway.auth.trustedProxy.userHeader

string

required

Header name containing the authenticated user identity.

[​](https://docs.openclaw.ai/gateway/trusted-proxy-auth#param-gateway-auth-trusted-proxy-required-headers)

gateway.auth.trustedProxy.requiredHeaders

string\\[\\]

Additional headers that must be present for the request to be trusted.

[​](https://docs.openclaw.ai/gateway/trusted-proxy-auth#param-gateway-auth-trusted-proxy-allow-users)

gateway.auth.trustedProxy.allowUsers

string\\[\\]

Allowlist of user identities. Empty means allow all authenticated users.

[​](https://docs.openclaw.ai/gateway/trusted-proxy-auth#param-gateway-auth-trusted-proxy-allow-loopback)

gateway.auth.trustedProxy.allowLoopback

boolean

Opt-in support for same-host loopback reverse proxies. Defaults to `false`.

Only enable `allowLoopback` when the local reverse proxy is the intended trust boundary. Any local process that can connect to the Gateway can try to send proxy identity headers, so keep direct Gateway access private to the host and require proxy-owned headers such as `x-forwarded-proto` or a signed assertion header where your proxy supports one.

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#tls-termination-and-hsts)  TLS termination and HSTS

Use one TLS termination point and apply HSTS there.

- Proxy TLS termination (recommended)

- Gateway TLS termination


When your reverse proxy handles HTTPS for `https://control.example.com`, set `Strict-Transport-Security` at the proxy for that domain.

- Good fit for internet-facing deployments.
- Keeps certificate + HTTP hardening policy in one place.
- OpenClaw can stay on loopback HTTP behind the proxy.

Example header value:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

If OpenClaw itself serves HTTPS directly (no TLS-terminating proxy), set:

```
{
  gateway: {
    tls: { enabled: true },
    http: {
      securityHeaders: {
        strictTransportSecurity: \"max-age=31536000; includeSubDomains\",
      },
    },
  },
}
```

`strictTransportSecurity` accepts a string header value, or `false` to disable explicitly.

### [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#rollout-guidance)  Rollout guidance

- Start with a short max age first (for example `max-age=300`) while validating traffic.
- Increase to long-lived values (for example `max-age=31536000`) only after confidence is high.
- Add `includeSubDomains` only if every subdomain is HTTPS-ready.
- Use preload only if you intentionally meet preload requirements for your full domain set.
- Loopback-only local development does not benefit from HSTS.

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#proxy-setup-examples)  Proxy setup examples

Pomerium

Pomerium passes identity in `x-pomerium-claim-email` (or other claim headers) and a JWT in `x-pomerium-jwt-assertion`.

```
{
  gateway: {
    bind: \"lan\",
    trustedProxies: [\"10.0.0.1\"], // Pomerium\'s IP
    auth: {
      mode: \"trusted-proxy\",
      trustedProxy: {
        userHeader: \"x-pomerium-claim-email\",
        requiredHeaders: [\"x-pomerium-jwt-assertion\"],
      },
    },
  },
}
```

Pomerium config snippet:

```
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

Caddy with OAuth

Caddy with the `caddy-security` plugin can authenticate users and pass identity headers.

```
{
  gateway: {
    bind: \"lan\",
    trustedProxies: [\"10.0.0.1\"], // Caddy/sidecar proxy IP
    auth: {
      mode: \"trusted-proxy\",
      trustedProxy: {
        userHeader: \"x-forwarded-user\",
      },
    },
  },
}
```

Caddyfile snippet:

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

nginx + oauth2-proxy

oauth2-proxy authenticates users and passes identity in `x-auth-request-email`.

```
{
  gateway: {
    bind: \"lan\",
    trustedProxies: [\"10.0.0.1\"], // nginx/oauth2-proxy IP
    auth: {
      mode: \"trusted-proxy\",
      trustedProxy: {
        userHeader: \"x-auth-request-email\",
      },
    },
  },
}
```

nginx config snippet:

```
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection \"upgrade\";
}
```

Traefik with forward auth

```
{
  gateway: {
    bind: \"lan\",
    trustedProxies: [\"172.17.0.1\"], // Traefik container IP
    auth: {
      mode: \"trusted-proxy\",
      trustedProxy: {
        userHeader: \"x-forwarded-user\",
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#mixed-token-configuration)  Mixed token configuration

OpenClaw rejects ambiguous configurations where both a `gateway.auth.token` (or `OPENCLAW_GATEWAY_TOKEN`) and `trusted-proxy` mode are active at the same time. Mixed token configs can cause loopback requests to silently authenticate on the wrong auth path.If you see a `mixed_trusted_proxy_token` error on startup:

- Remove the shared token when using trusted-proxy mode, or
- Switch `gateway.auth.mode` to `\"token\"` if you intend token-based auth.

Loopback trusted-proxy identity headers still fail closed: same-host callers are not silently authenticated as proxy users. Internal OpenClaw callers that bypass the proxy may authenticate with `gateway.auth.password` / `OPENCLAW_GATEWAY_PASSWORD` instead. Token fallback remains intentionally unsupported in trusted-proxy mode.

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#operator-scopes-header)  Operator scopes header

Trusted-proxy auth is an **identity-bearing** HTTP mode, so callers may optionally declare operator scopes with `x-openclaw-scopes`.Examples:

- `x-openclaw-scopes: operator.read`
- `x-openclaw-scopes: operator.read,operator.write`
- `x-openclaw-scopes: operator.admin,operator.write`

Behavior:

- When the header is present, OpenClaw honors the declared scope set.
- When the header is present but empty, the request declares **no** operator scopes.
- When the header is absent, normal identity-bearing HTTP APIs fall back to the standard operator default scope set.
- Gateway-auth **plugin HTTP routes** are narrower by default: when `x-openclaw-scopes` is absent, their runtime scope falls back to `operator.write`.
- Browser-origin HTTP requests still have to pass `gateway.controlUi.allowedOrigins` (or deliberate Host-header fallback mode) even after trusted-proxy auth succeeds.

Practical rule: send `x-openclaw-scopes` explicitly when you want a trusted-proxy request to be narrower than the defaults, or when a gateway-auth plugin route needs something stronger than write scope.

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#security-checklist)  Security checklist

Before enabling trusted-proxy auth, verify:

- [ ] **Proxy is the only path**: The Gateway port is firewalled from everything except your proxy.
- [ ] **trustedProxies is minimal**: Only your actual proxy IPs, not entire subnets.
- [ ] **Loopback proxy source is deliberate**: trusted-proxy auth fails closed for loopback-source requests unless `gateway.auth.trustedProxy.allowLoopback` is explicitly enabled for a same-host proxy.
- [ ] **Proxy strips headers**: Your proxy overwrites (not appends) `x-forwarded-*` headers from clients.
- [ ] **TLS termination**: Your proxy handles TLS; users connect via HTTPS.
- [ ] **allowedOrigins is explicit**: Non-loopback Control UI uses explicit `gateway.controlUi.allowedOrigins`.
- [ ] **allowUsers is set** (recommended): Restrict to known users rather than allowing anyone authenticated.
- [ ] **No mixed token config**: Do not set both `gateway.auth.token` and `gateway.auth.mode: \"trusted-proxy\"`.
- [ ] **Local password fallback is private**: If you configure `gateway.auth.password` for internal direct callers, keep the Gateway port firewalled so non-proxy remote clients cannot reach it directly.

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#security-audit)  Security audit

`openclaw security audit` will flag trusted-proxy auth with a **critical** severity finding. This is intentional — it’s a reminder that you’re delegating security to your proxy setup.The audit checks for:

- Base `gateway.trusted_proxy_auth` warning/critical reminder
- Missing `trustedProxies` configuration
- Missing `userHeader` configuration
- Empty `allowUsers` (allows any authenticated user)
- Enabled `allowLoopback` for same-host proxy sources
- Wildcard or missing browser-origin policy on exposed Control UI surfaces

## [​](https://docs.openclaw.ai/gateway/trusted-proxy-auth\\#troubleshooting)  Troubleshooting

trusted\\_proxy\\_untrusted\\_source

The request didn’t come from an IP in `gateway.trustedProxies`. Check:

- Is the proxy IP correct? (Docker container IPs can change.)
- Is there a load balancer in front of your proxy?
- Use `docker inspect` or `kubectl get pods -o wide` to find actual IPs.

trusted\\_proxy\\_loopback\\_source

OpenClaw rejected a loopback-source trusted-proxy request.Check:

- Is the proxy connecting from `127.0.0.1` / `::1`?
- Are you trying to use trusted-proxy auth with a same-host loopback reverse proxy?

Fix:

- Prefer token/password auth for internal same-host clients that do not go through the proxy, or
- Route through a non-loopback trusted proxy address and keep that IP in `gateway.trustedProxies`, or
- For a deliberate same-host reverse proxy, set `gateway.auth.trustedProxy.allowLoopback = true`, keep the loopback address in `gateway.trustedProxies`, and make sure the proxy strips or overwrites identity headers.

trusted\\_proxy\\_user\\_missing

The user header was empty or missing. Check:

- Is your proxy configured to pass identity headers?
- Is the header name correct? (case-insensitive, but spelling matters)
- Is the user actually authenticated at the proxy?

trusted\\_proxy\\_missing\\_header\\_\\*\n\nA required header wasn’t present. Check:

- Your proxy configuration for those specific headers.
- Whether headers are being stripped somewhere in the chain.
\ntrusted\\_proxy\\_user\\_not\\_allowed

The user is authenticated but not in `allowUsers`. Either add them or remove the allowlist.

trusted\\_proxy\\_origin\\_not\\_allowed

Trusted-proxy auth succeeded, but the browser `Origin` header did not pass Control UI origin checks.Check:

- `gateway.controlUi.allowedOrigins` includes the exact browser origin.
- You are not relying on wildcard origins unless you intentionally want allow-all behavior.
- If you intentionally use Host-header fallback mode, you must explicitly configure `gateway.controlUi.allowedOrigins: [\"*\"]`.

---

## Gateway lock - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/gateway-lock

[Skip to main content](https://docs.openclaw.ai/gateway/gateway-lock#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Scaling and operations

Gateway lock

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Why](https://docs.openclaw.ai/gateway/gateway-lock#why)
- [Mechanism](https://docs.openclaw.ai/gateway/gateway-lock#mechanism)
- [Error surface](https://docs.openclaw.ai/gateway/gateway-lock#error-surface)
- [Operational notes](https://docs.openclaw.ai/gateway/gateway-lock#operational-notes)
- [Related](https://docs.openclaw.ai/gateway/gateway-lock#related)

## [​](https://docs.openclaw.ai/gateway/gateway-lock\\#why)  Why

- Ensure only one gateway instance runs per base port on the same host; additional gateways must use isolated profiles and unique ports.
- Survive crashes/SIGKILL without leaving stale lock files.
- Fail fast with a clear error when the control port is already occupied.

## [​](https://docs.openclaw.ai/gateway/gateway-lock\\#mechanism)  Mechanism

- The gateway binds the WebSocket listener (default `ws://127.0.0.1:18789`) immediately on startup using an exclusive TCP listener.
- If the bind fails with `EADDRINUSE`, startup throws `GatewayLockError(\"another gateway instance is already listening on ws://127.0.0.1:<port>\")`.
- The OS releases the listener automatically on any process exit, including crashes and SIGKILL—no separate lock file or cleanup step is needed.
- On shutdown the gateway closes the WebSocket server and underlying HTTP server to free the port promptly.

## [​](https://docs.openclaw.ai/gateway/gateway-lock\\#error-surface)  Error surface

- If another process holds the port, startup throws `GatewayLockError(\"another gateway instance is already listening on ws://127.0.0.1:<port>\")`.
- Other bind failures surface as `GatewayLockError(\"failed to bind gateway socket on ws://127.0.0.1:<port>: …\")`.

## [​](https://docs.openclaw.ai/gateway/gateway-lock\\#operational-notes)  Operational notes

- If the port is occupied by _another_ process, the error is the same; free the port or choose another with `openclaw gateway --port <port>`.
- The macOS app still maintains its own lightweight PID guard before spawning the gateway; the runtime lock is enforced by the WebSocket bind.

## [​](https://docs.openclaw.ai/gateway/gateway-lock\\#related)  Related

- [Multiple Gateways](https://docs.openclaw.ai/gateway/multiple-gateways) — running multiple instances with unique ports
- [Troubleshooting](https://docs.openclaw.ai/gateway/troubleshooting) — diagnosing `EADDRINUSE` and port conflicts

[Troubleshooting](https://docs.openclaw.ai/gateway/troubleshooting) [Background exec and process tool](https://docs.openclaw.ai/gateway/background-process)

Ctrl+I

---

## Authentication - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/authentication

[Skip to main content](https://docs.openclaw.ai/gateway/authentication#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Authentication and secrets

Authentication

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Recommended setup (API key, any provider)](https://docs.openclaw.ai/gateway/authentication#recommended-setup-api-key-any-provider)
- [Anthropic: Claude CLI and token compatibility](https://docs.openclaw.ai/gateway/authentication#anthropic-claude-cli-and-token-compatibility)
- [Anthropic note](https://docs.openclaw.ai/gateway/authentication#anthropic-note)
- [Checking model auth status](https://docs.openclaw.ai/gateway/authentication#checking-model-auth-status)
- [API key rotation behavior (gateway)](https://docs.openclaw.ai/gateway/authentication#api-key-rotation-behavior-gateway)
- [Controlling which credential is used](https://docs.openclaw.ai/gateway/authentication#controlling-which-credential-is-used)
- [Per-session (chat command)](https://docs.openclaw.ai/gateway/authentication#per-session-chat-command)
- [Per-agent (CLI override)](https://docs.openclaw.ai/gateway/authentication#per-agent-cli-override)
- [Troubleshooting](https://docs.openclaw.ai/gateway/authentication#troubleshooting)
- [”No credentials found”](https://docs.openclaw.ai/gateway/authentication#%E2%80%9Dno-credentials-found%E2%80%9D)
- [Token expiring/expired](https://docs.openclaw.ai/gateway/authentication#token-expiring%2Fexpired)
- [Related](https://docs.openclaw.ai/gateway/authentication#related)

This page is the **model provider** authentication reference (API keys, OAuth, Claude CLI reuse, and Anthropic setup-token). For **gateway connection** authentication (token, password, trusted-proxy), see [Configuration](https://docs.openclaw.ai/gateway/configuration) and [Trusted Proxy Auth](https://docs.openclaw.ai/gateway/trusted-proxy-auth).

OpenClaw supports OAuth and API keys for model providers. For always-on gateway
hosts, API keys are usually the most predictable option. Subscription/OAuth
flows are also supported when they match your provider account model.See [/concepts/oauth](https://docs.openclaw.ai/concepts/oauth) for the full OAuth flow and storage
layout.
For SecretRef-based auth (`env`/`file`/`exec` providers), see [Secrets Management](https://docs.openclaw.ai/gateway/secrets).
For credential eligibility/reason-code rules used by `models status --probe`, see
[Auth Credential Semantics](https://docs.openclaw.ai/auth-credential-semantics).

## [​](https://docs.openclaw.ai/gateway/authentication\\#recommended-setup-api-key-any-provider)  Recommended setup (API key, any provider)

If you’re running a long-lived gateway, start with an API key for your chosen
provider.
For Anthropic specifically, API key auth is still the most predictable server
setup, but OpenClaw also supports reusing a local Claude CLI login.

1. Create an API key in your provider console.
2. Put it on the **gateway host** (the machine running `openclaw gateway`).

```
export <PROVIDER>_API_KEY=\"...\"
openclaw models status
```

3. If the Gateway runs under systemd/launchd, prefer putting the key in
`~/.openclaw/.env` so the daemon can read it:

```
cat >> ~/.openclaw/.env <<\'EOF\'
<PROVIDER>_API_KEY=...\nEOF\n```

Then restart the daemon (or restart your Gateway process) and re-check:

```
openclaw models status
openclaw doctor
```

If you’d rather not manage env vars yourself, onboarding can store
API keys for daemon use: `openclaw onboard`.See [Help](https://docs.openclaw.ai/help) for details on env inheritance (`env.shellEnv`,
`~/.openclaw/.env`, systemd/launchd).

## [​](https://docs.openclaw.ai/gateway/authentication\\#anthropic-claude-cli-and-token-compatibility)  Anthropic: Claude CLI and token compatibility

Anthropic setup-token auth is still available in OpenClaw as a supported token
path. Anthropic staff has since told us that OpenClaw-style Claude CLI usage is
allowed again, so OpenClaw treats Claude CLI reuse and `claude -p` usage as
sanctioned for this integration unless Anthropic publishes a new policy. When
Claude CLI reuse is available on the host, that is now the preferred path.For long-lived gateway hosts, an Anthropic API key is still the most predictable
setup. If you want to reuse an existing Claude login on the same host, use the
Anthropic Claude CLI path in onboarding/configure.Recommended host setup for Claude CLI reuse:

```
# Run on the gateway host
claude auth login
claude auth status --text
openclaw models auth login --provider anthropic --method cli --set-default
```

This is a two-step setup:

1. Log Claude Code itself into Anthropic on the gateway host.
2. Tell OpenClaw to switch Anthropic model selection to the local `claude-cli`
backend and store the matching OpenClaw auth profile.

If `claude` is not on `PATH`, either install Claude Code first or set
`agents.defaults.cliBackends.claude-cli.command` to the real binary path.Manual token entry (any provider; writes `auth-profiles.json` \\+ updates config):

```
openclaw models auth paste-token --provider openrouter
```

`auth-profiles.json` stores credentials only. The canonical shape is:

```
{
  \"version\": 1,\n  \"profiles\": {
    \"openrouter:default\": {
      \"type\": \"api_key\",\n      \"provider\": \"openrouter\",\n      \"key\": \"OPENROUTER_API_KEY\"
    }
  }
}
```

OpenClaw expects the canonical `version` \\+ `profiles` shape at runtime. If an older install still has a flat file such as `{ \"openrouter\": { \"apiKey\": \"...\" } }`, run `openclaw doctor --fix` to rewrite it as an `openrouter:default` API-key profile; doctor keeps a `.legacy-flat.*.bak` copy beside the original. Endpoint details such as `baseUrl`, `api`, model ids, headers, and timeouts belong under `models.providers.<id>` in `openclaw.json` or `models.json`, not in `auth-profiles.json`.Auth profile refs are also supported for static credentials:

- `api_key` credentials can use `keyRef: { source, provider, id }`
- `token` credentials can use `tokenRef: { source, provider, id }`
- OAuth-mode profiles do not support SecretRef credentials; if `auth.profiles.<id>.mode` is set to `\"oauth\"`, SecretRef-backed `keyRef`/`tokenRef` input for that profile is rejected.

Automation-friendly check (exit `1` when expired/missing, `2` when expiring):

```
openclaw models status --check
```

Live auth probes:

```
openclaw models status --probe
```

Notes:

- Probe rows can come from auth profiles, env credentials, or `models.json`.
- If explicit `auth.order.<provider>` omits a stored profile, probe reports
`excluded_by_auth_order` for that profile instead of trying it.
- If auth exists but OpenClaw cannot resolve a probeable model candidate for
that provider, probe reports `status: no_model`.
- Rate-limit cooldowns can be model-scoped. A profile cooling down for one
model can still be usable for a sibling model on the same provider.

Optional ops scripts (systemd/Termux) are documented here:
[Auth monitoring scripts](https://docs.openclaw.ai/help/scripts#auth-monitoring-scripts)

## [​](https://docs.openclaw.ai/gateway/authentication\\#anthropic-note)  Anthropic note

The Anthropic `claude-cli` backend is supported again.

- Anthropic staff told us this OpenClaw integration path is allowed again.
- OpenClaw therefore treats Claude CLI reuse and `claude -p` usage as sanctioned
for Anthropic-backed runs unless Anthropic publishes a new policy.
- Anthropic API keys remain the most predictable choice for long-lived gateway
hosts and explicit server-side billing control.

## [​](https://docs.openclaw.ai/gateway/authentication\\#checking-model-auth-status)  Checking model auth status

```
openclaw models status
openclaw doctor
```

## [​](https://docs.openclaw.ai/gateway/authentication\\#api-key-rotation-behavior-gateway)  API key rotation behavior (gateway)

Some providers support retrying a request with alternative keys when an API call
hits a provider rate limit.

- Priority order:
  - `OPENCLAW_LIVE_<PROVIDER>_KEY` (single override)
  - `<PROVIDER>_API_KEYS`
  - `<PROVIDER>_API_KEY`
  - `<PROVIDER>_API_KEY_*`
- Google providers also include `GOOGLE_API_KEY` as an additional fallback.
- The same key list is deduplicated before use.
- OpenClaw retries with the next key only for rate-limit errors (for example
`429`, `rate_limit`, `quota`, `resource exhausted`, `Too many concurrent requests`, `ThrottlingException`, `concurrency limit reached`, or
`workers_ai ... quota limit exceeded`).
- Non-rate-limit errors are not retried with alternate keys.
- If all keys fail, the final error from the last attempt is returned.

## [​](https://docs.openclaw.ai/gateway/authentication\\#controlling-which-credential-is-used)  Controlling which credential is used

### [​](https://docs.openclaw.ai/gateway/authentication\\#per-session-chat-command)  Per-session (chat command)

Use `/model <alias-or-id>@<profileId>` to pin a specific provider credential for the current session (example profile ids: `anthropic:default`, `anthropic:work`).Use `/model` (or `/model list`) for a compact picker; use `/model status` for the full view (candidates + next auth profile, plus provider endpoint details when configured).

### [​](https://docs.openclaw.ai/gateway/authentication\\#per-agent-cli-override)  Per-agent (CLI override)

Set an explicit auth profile order override for an agent (stored in that agent’s `auth-state.json`):

```
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Use `--agent <id>` to target a specific agent; omit it to use the configured default agent.
When you debug order issues, `openclaw models status --probe` shows omitted
stored profiles as `excluded_by_auth_order` instead of silently skipping them.
When you debug cooldown issues, remember that rate-limit cooldowns can be tied
to one model id rather than the whole provider profile.

## [​](https://docs.openclaw.ai/gateway/authentication\\#troubleshooting)  Troubleshooting

### [​](https://docs.openclaw.ai/gateway/authentication\\#%E2%80%9Dno-credentials-found%E2%80%9D)  ”No credentials found”

If the Anthropic profile is missing, configure an Anthropic API key on the
**gateway host** or set up the Anthropic setup-token path, then re-check:

```
openclaw models status
```

### [​](https://docs.openclaw.ai/gateway/authentication\\#token-expiring/expired)  Token expiring/expired

Run `openclaw models status` to confirm which profile is expiring. If an
Anthropic token profile is missing or expired, refresh that setup via
setup-token or migrate to an Anthropic API key.

## [​](https://docs.openclaw.ai/gateway/authentication\\#related)  Related

- [Secrets management](https://docs.openclaw.ai/gateway/secrets)
- [Remote access](https://docs.openclaw.ai/gateway/remote)
- [Auth storage](https://docs.openclaw.ai/concepts/oauth)

[Configuration examples](https://docs.openclaw.ai/gateway/configuration-examples) [Auth credential semantics](https://docs.openclaw.ai/auth-credential-semantics)

Ctrl+I

---

## Configuration examples
**Source:** https://docs.openclaw.ai/gateway/configuration-examples

[Skip to main content](https://docs.openclaw.ai/gateway/configuration-examples#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Configuration

Configuration examples

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/gateway/configuration-examples#quick-start)
- [Absolute minimum](https://docs.openclaw.ai/gateway/configuration-examples#absolute-minimum)
- [Recommended starter](https://docs.openclaw.ai/gateway/configuration-examples#recommended-starter)
- [Expanded example (major options)](https://docs.openclaw.ai/gateway/configuration-examples#expanded-example-major-options)
- [Common patterns](https://docs.openclaw.ai/gateway/configuration-examples#common-patterns)
- [Shared skill baseline with one override](https://docs.openclaw.ai/gateway/configuration-examples#shared-skill-baseline-with-one-override)
- [Multi-platform setup](https://docs.openclaw.ai/gateway/configuration-examples#multi-platform-setup)
- [Trusted node network auto-approval](https://docs.openclaw.ai/gateway/configuration-examples#trusted-node-network-auto-approval)
- [Secure DM mode (shared inbox / multi-user DMs)](https://docs.openclaw.ai/gateway/configuration-examples#secure-dm-mode-shared-inbox-%2F-multi-user-dms)
- [Anthropic API key + MiniMax fallback](https://docs.openclaw.ai/gateway/configuration-examples#anthropic-api-key-%2B-minimax-fallback)
- [Work bot (restricted access)](https://docs.openclaw.ai/gateway/configuration-examples#work-bot-restricted-access)
- [Local models only](https://docs.openclaw.ai/gateway/configuration-examples#local-models-only)
- [Tips](https://docs.openclaw.ai/gateway/configuration-examples#tips)
- [Related](https://docs.openclaw.ai/gateway/configuration-examples#related)

Examples below are aligned with the current config schema. For the exhaustive reference and per-field notes, see [Configuration](https://docs.openclaw.ai/gateway/configuration).

## [​](https://docs.openclaw.ai/gateway/configuration-examples\\#quick-start)  Quick start

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#absolute-minimum)  Absolute minimum

```
{
  agent: { workspace: \"~/.openclaw/workspace\" },
  channels: { whatsapp: { allowFrom: [\"+15555550123\"] } },
}
```

Save to `~/.openclaw/openclaw.json` and you can DM the bot from that number.

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#recommended-starter)  Recommended starter

```
{
  identity: {
    name: \"Clawd\",
    theme: \"helpful assistant\",
    emoji: \"🦞\",
  },
  agent: {
    workspace: \"~/.openclaw/workspace\",
    model: { primary: \"anthropic/claude-sonnet-4-6\" },
  },
  channels: {
    whatsapp: {
      allowFrom: [\"+15555550123\"],
      groups: { \"*\": { requireMention: true } },
    },
  },
  messages: {
    groupChat: {
      visibleReplies: \"message_tool\", // default; use \"automatic\" for legacy room replies
    },
  },
}
```

## [​](https://docs.openclaw.ai/gateway/configuration-examples\\#expanded-example-major-options)  Expanded example (major options)

> JSON5 lets you use comments and trailing commas. Regular JSON works too.

```
{
  // Environment + shell
  env: {
    OPENROUTER_API_KEY: \"sk-or-...\",
    vars: {
      GROQ_API_KEY: \"gsk-...\",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },

  // Auth profile metadata (secrets live in auth-profiles.json)
  auth: {
    profiles: {
      \"anthropic:default\": { provider: \"anthropic\", mode: \"api_key\" },
      \"anthropic:work\": { provider: \"anthropic\", mode: \"api_key\" },
      \"openai:default\": { provider: \"openai\", mode: \"api_key\" },
      \"openai-codex:personal\": { provider: \"openai-codex\", mode: \"oauth\" },
    },
    order: {
      anthropic: [\"anthropic:default\", \"anthropic:work\"],
      openai: [\"openai:default\"],
      \"openai-codex\": [\"openai-codex:personal\"],
    },
  },

  // Identity
  identity: {
    name: \"Samantha\",
    theme: \"helpful sloth\",
    emoji: \"🦥\",
  },

  // Logging
  logging: {
    level: \"info\",
    file: \"/tmp/openclaw/openclaw.log\",
    consoleLevel: \"info\",
    consoleStyle: \"pretty\",
    redactSensitive: \"tools\",
  },

  // Message formatting
  messages: {
    messagePrefix: \"[openclaw]\",
    responsePrefix: \">\",
    ackReaction: \"👀\",
    ackReactionScope: \"group-mentions\",
    groupChat: {
      historyLimit: 50,
      visibleReplies: \"message_tool\", // normal final replies stay private in groups/channels
    },
    queue: {
      mode: \"collect\",
      debounceMs: 1000,
      cap: 20,
      drop: \"summarize\",
      byChannel: {
        whatsapp: \"collect\",
        telegram: \"collect\",
        discord: \"collect\",
        slack: \"collect\",
        signal: \"collect\",
        imessage: \"collect\",
        webchat: \"collect\",
      },
    },
  },

  // Tooling
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [\\\
          { provider: \"openai\", model: \"gpt-4o-mini-transcribe\" },\\\
          // Optional CLI fallback (Whisper binary):\\\
          // { type: \"cli\", command: \"whisper\", args: [\"--model\", \"base\", \"{{MediaPath}}\"] }\\\
        ],
        timeoutSeconds: 120,
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: \"google\", model: \"gemini-3-flash-preview\" }],
      },
    },
  },

  // Session behavior
  session: {
    scope: \"per-sender\",
    dmScope: \"per-channel-peer\", // recommended for multi-user inboxes
    reset: {
      mode: \"daily\",
      atHour: 4,
      idleMinutes: 60,
    },
    resetByChannel: {
      discord: { mode: \"idle\", idleMinutes: 10080 },
    },
    resetTriggers: [\"/new\", \"/reset\"],
    store: \"~/.openclaw/agents/default/sessions/sessions.json\",
    maintenance: {
      mode: \"warn\",
      pruneAfter: \"30d\",
      maxEntries: 500,
      resetArchiveRetention: \"30d\", // duration or false
      maxDiskBytes: \"500mb\", // optional
      highWaterBytes: \"400mb\", // optional (defaults to 80% of maxDiskBytes)
    },
    typingIntervalSeconds: 5,
    sendPolicy: {
      default: \"allow\",
      rules: [{ action: \"deny\", match: { channel: \"discord\", chatType: \"group\" } }],
    },
  },

  // Channels
  channels: {
    whatsapp: {
      dmPolicy: \"pairing\",
      allowFrom: [\"+15555550123\"],
      groupPolicy: \"allowlist\",
      groupAllowFrom: [\"+15555550123\"],
      groups: { \"*\": { requireMention: true } },
    },

    telegram: {
      enabled: true,
      botToken: \"YOUR_TELEGRAM_BOT_TOKEN\",
      allowFrom: [\"123456789\"],
      groupPolicy: \"allowlist\",
      groupAllowFrom: [\"123456789\"],
      groups: { \"*\": { requireMention: true } },
    },

    discord: {
      enabled: true,
      token: \"YOUR_DISCORD_BOT_TOKEN\",
      dm: { enabled: true, allowFrom: [\"123456789012345678\"] },
      guilds: {
        \"123456789012345678\": {
          slug: \"friends-of-openclaw\",
          requireMention: false,
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },

    slack: {
      enabled: true,
      botToken: \"xoxb-REPLACE_ME\",
      appToken: \"xapp-REPLACE_ME\",
      channels: {
        \"#general\": { allow: true, requireMention: true },
      },
      dm: { enabled: true, allowFrom: [\"U123\"] },
      slashCommand: {
        enabled: true,
        name: \"openclaw\",
        sessionPrefix: \"slack:slash\",
        ephemeral: true,
      },
    },
  },

  // Agent runtime
  agents: {
    defaults: {
      workspace: \"~/.openclaw/workspace\",
      userTimezone: \"America/Chicago\",
      model: {
        primary: \"anthropic/claude-sonnet-4-6\",
        fallbacks: [\"anthropic/claude-opus-4-6\", \"openai/gpt-5.4\"],
      },
      imageModel: {
        primary: \"openrouter/anthropic/claude-sonnet-4-6\",
      },
      models: {
        \"anthropic/claude-opus-4-6\": { alias: \"opus\" },
        \"anthropic/claude-sonnet-4-6\": { alias: \"sonnet\" },
        \"openai/gpt-5.4\": { alias: \"gpt\" },
      },
      skills: [\"github\", \"weather\"], // inherited by agents that omit list[].skills
      thinkingDefault: \"low\",
      verboseDefault: \"off\",
      elevatedDefault: \"on\",
      blockStreamingDefault: \"off\",
      blockStreamingBreak: \"text_end\",
      blockStreamingChunk: {
        minChars: 800,
        maxChars: 1200,
        breakPreference: \"paragraph\",
      },
      blockStreamingCoalesce: {
        idleMs: 1000,
      },
      humanDelay: {
        mode: \"natural\",
      },
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      typingIntervalSeconds: 5,
      maxConcurrent: 3,
      heartbeat: {
        every: \"30m\",
        model: \"anthropic/claude-sonnet-4-6\",
        target: \"last\",
        directPolicy: \"allow\", // allow (default) | block
        to: \"+15555550123\",
        prompt: \"HEARTBEAT\",
        ackMaxChars: 300,
      },
      memorySearch: {
        provider: \"gemini\",
        model: \"gemini-embedding-001\",
        remote: {
          apiKey: \"${GEMINI_API_KEY}\",
        },
        extraPaths: [\"../team-docs\", \"/srv/shared-notes\"],
      },
      sandbox: {
        mode: \"non-main\",
        scope: \"session\", // preferred over legacy perSession: true
        workspaceRoot: \"~/.openclaw/sandboxes\",
        docker: {
          image: \"openclaw-sandbox:bookworm-slim\",
          workdir: \"/workspace\",
          readOnlyRoot: true,
          tmpfs: [\"/tmp\", \"/var/tmp\", \"/run\"],
          network: \"none\",
          user: \"1000:1000\",
        },
        browser: {
          enabled: false,
        },
      },
    },
    list: [\\\
      {\\\n        id: \"main\",\\\
        default: true,\\\n        // inherits defaults.skills -> github, weather\\\
        groupChat: {\\\
          mentionPatterns: [\"@openclaw\", \"openclaw\"],\\\
        },\\\n        thinkingDefault: \"high\", // per-agent thinking override\\\
        reasoningDefault: \"on\", // per-agent reasoning visibility\\\
        fastModeDefault: false, // per-agent fast mode\\\
      },\\\n      {\\\n        id: \"quick\",\\\
        skills: [], // no skills for this agent\\\
        fastModeDefault: true, // this agent always runs fast\\\
        thinkingDefault: \"off\",\\\
      },\\\n    ],
  },

  tools: {
    allow: [\"exec\", \"process\", \"read\", \"write\", \"edit\", \"apply_patch\"],
    deny: [\"browser\", \"canvas\"],
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
    },
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: [\"+15555550123\"],
        telegram: [\"123456789\"],
        discord: [\"123456789012345678\"],
        slack: [\"U123\"],
        signal: [\"+15555550123\"],
        imessage: [\"user@example.com\"],
        webchat: [\"session:demo\"],
      },
    },
  },

  // Custom model providers
  models: {
    mode: \"merge\",
    providers: {
      \"custom-proxy\": {
        baseUrl: \"http://localhost:4000/v1\",
        apiKey: \"LITELLM_KEY\",
        api: \"openai-responses\",
        authHeader: true,
        headers: { \"X-Proxy-Region\": \"us-west\" },
        models: [\\\
          {\\\n            id: \"llama-3.1-8b\",\\\
            name: \"Llama 3.1 8B\",\\\
            api: \"openai-responses\",\\\
            reasoning: false,\\\n            input: [\"text\"],\\\
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\
            contextWindow: 128000,\\\n            maxTokens: 32000,\\\n          },\\\n        ],
      },
    },
  },

  // Cron jobs
  cron: {
    enabled: true,
    store: \"~/.openclaw/cron/cron.json\",
    maxConcurrentRuns: 2, // cron dispatch + isolated cron agent-turn execution
    sessionRetention: \"24h\",
    runLog: {
      maxBytes: \"2mb\",
      keepLines: 2000,
    },
  },

  // Webhooks
  hooks: {
    enabled: true,
    path: \"/hooks\",
    token: \"shared-secret\",
    presets: [\"gmail\"],
    transformsDir: \"~/.openclaw/hooks/transforms\",
    mappings: [\\\
      {\\\n        id: \"gmail-hook\",\\\
        match: { path: \"gmail\" },\\\
        action: \"agent\",\\\
        wakeMode: \"now\",\\\
        name: \"Gmail\",\\\
        sessionKey: \"hook:gmail:{{messages[0].id}}\",\\\
        messageTemplate: \"From: {{messages[0].from}}\\nSubject: {{messages[0].subject}}\",\\\
        textTemplate: \"{{messages[0].snippet}}\",\\\
        deliver: true,\\\n        channel: \"last\",\\\
        to: \"+15555550123\",\\\
        thinking: \"low\",\\\
        timeoutSeconds: 300,\\\n        transform: {\\\
          module: \"gmail.js\",\\\
          export: \"transformGmail\",\\\
        },\\\n      },\\\n    ],
    gmail: {
      account: \"openclaw@gmail.com\",
      label: \"INBOX\",
      topic: \"projects/<project-id>/topics/gog-gmail-watch\",
      subscription: \"gog-gmail-watch-push\",
      pushToken: \"shared-push-token\",
      hookUrl: \"http://127.0.0.1:18789/hooks/gmail\",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: \"127.0.0.1\", port: 8788, path: \"/\" },
      tailscale: { mode: \"funnel\", path: \"/gmail-pubsub\" },
    },
  },

  // Gateway + networking
  gateway: {
    mode: \"local\",
    port: 18789,
    bind: \"loopback\",
    controlUi: { enabled: true, basePath: \"/openclaw\" },
    auth: {
      mode: \"token\",
      token: \"gateway-token\",
      allowTailscale: true,
    },
    tailscale: { mode: \"serve\", resetOnExit: false },
    remote: { url: \"ws://gateway.tailnet:18789\", token: \"remote-token\" },
    reload: { mode: \"hybrid\", debounceMs: 300 },
  },

  skills: {
    allowBundled: [\"gemini\", \"peekaboo\"],
    load: {
      extraDirs: [\"~/Projects/agent-scripts/skills\"],
    },
    install: {
      preferBrew: true,
      nodeManager: \"npm\", // npm | pnpm | yarn | bun
    },
    entries: {
      \"image-lab\": {
        enabled: true,
        apiKey: \"GEMINI_KEY_HERE\",
        env: { GEMINI_API_KEY: \"GEMINI_KEY_HERE\" },
      },
      peekaboo: { enabled: true },
    },
  },
}
```

## [​](https://docs.openclaw.ai/gateway/configuration-examples\\#common-patterns)  Common patterns

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#shared-skill-baseline-with-one-override)  Shared skill baseline with one override

```
{
  agents: {
    defaults: {
      workspace: \"~/.openclaw/workspace\",
      skills: [\"github\", \"weather\"],
    },
    list: [\\\
      { id: \"main\", default: true },\\\
      { id: \"docs\", workspace: \"~/.openclaw/workspace-docs\", skills: [\"docs-search\"] },\\\
    ],
  },
}
```

- `agents.defaults.skills` is the shared baseline.
- `agents.list[].skills` replaces that baseline for one agent.
- Use `skills: []` when an agent should see no skills.

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#multi-platform-setup)  Multi-platform setup

```
{
  agent: { workspace: \"~/.openclaw/workspace\" },
  channels: {
    whatsapp: { allowFrom: [\"+15555550123\"] },
    telegram: {
      enabled: true,
      botToken: \"YOUR_TOKEN\",
      allowFrom: [\"123456789\"],
    },
    discord: {
      enabled: true,
      token: \"YOUR_TOKEN\",
      dm: { allowFrom: [\"123456789012345678\"] },
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#trusted-node-network-auto-approval)  Trusted node network auto-approval

Keep device pairing manual unless you control the network path. For a dedicated
lab or tailnet subnet, you can opt in to first-time node device auto-approval
with exact CIDRs or IPs:

```
{
  gateway: {
    nodes: {
      pairing: {
        autoApproveCidrs: [\"192.168.1.0/24\", \"fd00:1234:5678::/64\"],
      },
    },
  },
}
```

This remains off when unset. It only applies to fresh `role: node` pairing with
no requested scopes. Operator/browser clients and role, scope, metadata, or
public-key upgrades still require manual approval.

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#secure-dm-mode-shared-inbox-/-multi-user-dms)  Secure DM mode (recommended for multi-user or sensitive DM agents)

If more than one person can DM your bot (multiple entries in `allowFrom`, pairing approvals for multiple people, or `dmPolicy: \"open\"`), enable **secure DM mode** so DMs from different senders don’t share one context by default:

```
{
  // Secure DM mode (recommended for multi-user or sensitive DM agents)
  session: { dmScope: \"per-channel-peer\" },

  channels: {
    // Example: WhatsApp multi-user inbox
    whatsapp: {
      dmPolicy: \"allowlist\",
      allowFrom: [\"+15555550123\", \"+15555550124\"],
    },

    // Example: Discord multi-user inbox
    discord: {
      enabled: true,
      token: \"YOUR_DISCORD_BOT_TOKEN\",
      dm: { enabled: true, allowFrom: [\"123456789012345678\", \"987654321098765432\"] },
    },
  },
}
```

For Discord/Slack/Google Chat/Microsoft Teams/Mattermost/IRC, sender authorization is ID-first by default.
Only enable direct mutable name/email/nick matching with each channel’s `dangerouslyAllowNameMatching: true` if you explicitly accept that risk.

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#anthropic-api-key-+-minimax-fallback)  Anthropic API key + MiniMax fallback

```
{
  auth: {
    profiles: {
      \"anthropic:api\": {
        provider: \"anthropic\",
        mode: \"api_key\",
      },
    },
    order: {
      anthropic: [\"anthropic:api\"],
    },
  },
  models: {
    providers: {
      minimax: {
        baseUrl: \"https://api.minimax.io/anthropic\",
        api: \"anthropic-messages\",
        apiKey: \"${MINIMAX_API_KEY}\",
      },
    },
  },
  agent: {
    workspace: \"~/.openclaw/workspace\",
    model: {
      primary: \"anthropic/claude-opus-4-6\",
      fallbacks: [\"minimax/MiniMax-M2.7\"],
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#work-bot-restricted-access)  Work bot (restricted access)

```
{
  identity: {
    name: \"Work Bot\",
    theme: \"focused assistant\",
    emoji: \"👷\",
  },
  agent: {
    workspace: \"~/.openclaw/workspace-work\",
    model: { primary: \"anthropic/claude-sonnet-4-6\" },
  },
  channels: {
    whatsapp: {
      dmPolicy: \"allowlist\",
      allowFrom: [\"+15555550123\"],
      groupPolicy: \"allowlist\",
      groupAllowFrom: [\"+15555550123\"],
      groups: { \"*\": { requireMention: true } },
    },
  },
  tools: {
    allow: [\"exec\", \"read\", \"write\"],
    deny: [\"browser\", \"canvas\"],
    elevated: { enabled: true, allowFrom: { whatsapp: [\"+15555550123\"] } },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#local-models-only)  Local models only

```
{
  agent: {
    workspace: \"~/.openclaw/workspace\",
    model: { primary: \"ollama/llama3\" },
  },
  models: {
    providers: {
      ollama: {
        baseUrl: \"http://localhost:11434/api\",
        apiKey: \"ollama\",
        api: \"ollama\",
        models: [\\\
          {\\\n            id: \"llama3\",\\\
            name: \"Llama 3\",\\\
            api: \"ollama\",\\\
            reasoning: false,\\\n            input: [\"text\"],\\\
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\
            contextWindow: 8192,\\\n            maxTokens: 4096,\\
          },\\
        ],
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#tips)  Tips

- Use JSON5 for comments and trailing commas.
- Use `dmPolicy: \"per-channel-peer\"` for secure multi-user DMs.
- Use `agents.list[].skills` to override shared skill baselines.
- Use `firecrawl_map` to discover URLs before scraping.

### [​](https://docs.openclaw.ai/gateway/configuration-examples\\#related)  Related

- [Configuration](https://docs.openclaw.ai/gateway/configuration)
- [Gateway](https://docs.openclaw.ai/gateway)
- [Agents](https://docs.openclaw.ai/concepts/architecture)
- [Channels](https://docs.openclaw.ai/channels)
- [Tools & Plugins](https://docs.openclaw.ai/tools)
- [Models](https://docs.openclaw.ai/providers)
- [Platforms](https://docs.openclaw.ai/platforms)
- [CLI](https://docs.openclaw.ai/cli)
- [Help](https://docs.openclaw.ai/help)



---

## Gateway logging - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/logging

[Skip to main content](https://docs.openclaw.ai/gateway/logging#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Health and diagnostics

Gateway logging

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Logging](https://docs.openclaw.ai/gateway/logging#logging)
- [File-based logger](https://docs.openclaw.ai/gateway/logging#file-based-logger)
- [Console capture](https://docs.openclaw.ai/gateway/logging#console-capture)
- [Redaction](https://docs.openclaw.ai/gateway/logging#redaction)
- [Gateway WebSocket logs](https://docs.openclaw.ai/gateway/logging#gateway-websocket-logs)
- [WS log style](https://docs.openclaw.ai/gateway/logging#ws-log-style)
- [Console formatting (subsystem logging)](https://docs.openclaw.ai/gateway/logging#console-formatting-subsystem-logging)
- [Related](https://docs.openclaw.ai/gateway/logging#related)

# [​](https://docs.openclaw.ai/gateway/logging\\#logging)  Logging

For a user-facing overview (CLI + Control UI + config), see [/logging](https://docs.openclaw.ai/logging).OpenClaw has two log “surfaces”:

- **Console output** (what you see in the terminal / Debug UI).
- **File logs** (JSON lines) written by the gateway logger.

## [​](https://docs.openclaw.ai/gateway/logging\\#file-based-logger)  File-based logger

- Default rolling log file is under `/tmp/openclaw/` (one file per day): `openclaw-YYYY-MM-DD.log`
  - Date uses the gateway host’s local timezone.
- Active log files rotate at `logging.maxFileBytes` (default: 100 MB), keeping
up to five numbered archives and continuing to write a fresh active file.
- The log file path and level can be configured via `~/.openclaw/openclaw.json`:

  - `logging.file`
  - `logging.level`

The file format is one JSON object per line.The Control UI Logs tab tails this file via the gateway (`logs.tail`).
CLI can do the same:

```
openclaw logs --follow
```

**Verbose vs. log levels**

- **File logs** are controlled exclusively by `logging.level`.
- `--verbose` only affects **console verbosity** (and WS log style); it does **not**
raise the file log level.
- To capture verbose-only details in file logs, set `logging.level` to `debug` or
trace.

## [​](https://docs.openclaw.ai/gateway/logging\\#console-capture)  Console capture

The CLI captures `console.log/info/warn/error/debug/trace` and writes them to file logs,
while still printing to stdout/stderr.You can tune console verbosity independently via:

- `logging.consoleLevel` (default `info`)
- `logging.consoleStyle` (`pretty` \\| `compact` \\| `json`)

## [​](https://docs.openclaw.ai/gateway/logging\\#redaction)  Redaction

OpenClaw can mask sensitive tokens before log or transcript output leaves the
process. The same redaction policy is applied at console, file-log, OTLP
log-record, and session transcript text sinks, so matching secret values are
masked before JSONL lines or messages are written to disk.

- `logging.redactSensitive`: `off` \\| `tools` (default: `tools`)
- `logging.redactPatterns`: array of regex strings (overrides defaults)

  - Use raw regex strings (auto `gi`), or `/pattern/flags` if you need custom flags.
  - Matches are masked by keeping the first 6 + last 4 chars (length >= 18), otherwise `***`.
  - Defaults cover common key assignments, CLI flags, JSON fields, bearer headers, PEM blocks, and popular token prefixes.

## [​](https://docs.openclaw.ai/gateway/logging\\#gateway-websocket-logs)  Gateway WebSocket logs

The gateway prints WebSocket protocol logs in two modes:

- **Normal mode (no `--verbose`)**: only “interesting” RPC results are printed:

  - errors (`ok=false`)
  - slow calls (default threshold: `>= 50ms`)
  - parse errors
- **Verbose mode (`--verbose`)**: prints all WS request/response traffic.

### [​](https://docs.openclaw.ai/gateway/logging\\#ws-log-style)  WS log style

`openclaw gateway` supports a per-gateway style switch:

- `--ws-log auto` (default): normal mode is optimized; verbose mode uses compact output
- `--ws-log compact`: compact output (paired request/response) when verbose
- `--ws-log full`: full per-frame output when verbose
- `--compact`: alias for `--ws-log compact`

Examples:

```
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# show all WS traffic (full meta)
openclaw gateway --verbose --ws-log full
```

## [​](https://docs.openclaw.ai/gateway/logging\\#console-formatting-subsystem-logging)  Console formatting (subsystem logging)

The console formatter is **TTY-aware** and prints consistent, prefixed lines.
Subsystem loggers keep output grouped and scannable.Behavior:

- **Subsystem prefixes** on every line (e.g. `[gateway]`, `[canvas]`, `[tailscale]`)
- **Subsystem colors** (stable per subsystem) plus level coloring
- **Color when output is a TTY or the environment looks like a rich terminal** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), respects `NO_COLOR`
- **Shortened subsystem prefixes**: drops leading `gateway/` \\+ `channels/`, keeps last 2 segments (e.g. `whatsapp/outbound`)
- **Sub-loggers by subsystem** (auto prefix + structured field `{ subsystem }`)
- **`logRaw()`** for QR/UX output (no prefix, no formatting)
- **Console styles** (e.g. `pretty | compact | json`)
- **Console log level** separate from file log level (file keeps full detail when `logging.level` is set to `debug`/`trace`)
- **WhatsApp message bodies** are logged at `debug` (use `--verbose` to see them)

This keeps existing file logs stable while making interactive output scannable.

## [​](https://docs.openclaw.ai/gateway/logging\\#related)  Related

- [Logging](https://docs.openclaw.ai/logging)
- [OpenTelemetry export](https://docs.openclaw.ai/gateway/opentelemetry)
- [Diagnostics export](https://docs.openclaw.ai/gateway/diagnostics)

[Prometheus](https://docs.openclaw.ai/gateway/prometheus) [Diagnostics export](https://docs.openclaw.ai/gateway/diagnostics)

Ctrl+I

---

## Doctor
**Source:** https://docs.openclaw.ai/gateway/doctor

[Skip to main content](https://docs.openclaw.ai/gateway/doctor#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Health and diagnostics

Doctor

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/gateway/doctor#quick-start)
- [Headless and automation modes](https://docs.openclaw.ai/gateway/doctor#headless-and-automation-modes)
- [What it does (summary)](https://docs.openclaw.ai/gateway/doctor#what-it-does-summary)
- [Dreams UI backfill and reset](https://docs.openclaw.ai/gateway/doctor#dreams-ui-backfill-and-reset)
- [Detailed behavior and rationale](https://docs.openclaw.ai/gateway/doctor#detailed-behavior-and-rationale)
- [Related](https://docs.openclaw.ai/gateway/doctor#related)

`openclaw doctor` is the repair + migration tool for OpenClaw. It fixes stale config/state, checks health, and provides actionable repair steps.

## [​](https://docs.openclaw.ai/gateway/doctor\\#quick-start)  Quick start

```
openclaw doctor
```

### [​](https://docs.openclaw.ai/gateway/doctor\\#headless-and-automation-modes)  Headless and automation modes

- --yes

- --repair

- --repair --force

- --non-interactive

- --deep


```
openclaw doctor --yes
```

Accept defaults without prompting (including restart/service/sandbox repair steps when applicable).

```
openclaw doctor --repair
```

Apply recommended repairs without prompting (repairs + restarts where safe).

```
openclaw doctor --repair --force
```

Apply aggressive repairs too (overwrites custom supervisor configs).

```
openclaw doctor --non-interactive
```

Run without prompts and only apply safe migrations (config normalization + on-disk state moves). Skips restart/service/sandbox actions that require human confirmation. Legacy state migrations run automatically when detected.

```
openclaw doctor --deep
```

Scan system services for extra gateway installs (launchd/systemd/schtasks).

If you want to review changes before writing, open the config file first:

```
cat ~/.openclaw/openclaw.json
```

## [​](https://docs.openclaw.ai/gateway/doctor\\#what-it-does-summary)  What it does (summary)

Health, UI, and updates

- Optional pre-flight update for git installs (interactive only).
- UI protocol freshness check (rebuilds Control UI when the protocol schema is newer).
- Health check + restart prompt.
- Skills status summary (eligible/missing/blocked) and plugin status.

Config and migrations

- Config normalization for legacy values.
- Talk config migration from legacy flat `talk.*` fields into `talk.provider` \\+ `talk.providers.<provider>`.
- Browser migration checks for legacy Chrome extension configs and Chrome MCP readiness.
- OpenCode provider override warnings (`models.providers.opencode` / `models.providers.opencode-go`).
- Codex OAuth shadowing warnings (`models.providers.openai-codex`).
- OAuth TLS prerequisites check for OpenAI Codex OAuth profiles.
- Legacy on-disk state migration (sessions/agent dir/WhatsApp auth).
- Legacy plugin manifest contract key migration (`speechProviders`, `realtimeTranscriptionProviders`, `realtimeVoiceProviders`, `mediaUnderstandingProviders`, `imageGenerationProviders`, `videoGenerationProviders`, `webFetchProviders`, `webSearchProviders` → `contracts`).
- Legacy cron store migration (`jobId`, `schedule.cron`, top-level delivery/payload fields, payload `provider`, simple `notify: true` webhook fallback jobs).
- Legacy agent runtime-policy migration to `agents.defaults.agentRuntime` and `agents.list[].agentRuntime`.
- Stale plugin config cleanup when plugins are enabled; when `plugins.enabled=false`, stale plugin references are treated as inert containment config and are preserved.

State and integrity

- Session lock file inspection and stale lock cleanup.
- Session transcript repair for duplicated prompt-rewrite branches created by affected 2026.4.24 builds.
- State integrity and permissions checks (sessions, transcripts, state dir).
- Config file permission checks (chmod 600) when running locally.
- Model auth health: checks OAuth expiry, can refresh expiring tokens, and reports auth-profile cooldown/disabled states.
- Extra workspace dir detection (`~/openclaw`).

Gateway, services, and supervisors

- Sandbox image repair when sandboxing is enabled.
- Legacy service migration and extra gateway detection.
- Matrix channel legacy state migration (in `--fix` / `--repair` mode).
- Gateway runtime checks (service installed but not running; cached launchd label).
- Channel status warnings (probed from the running gateway).
- Supervisor config audit (launchd/systemd/schtasks) with optional repair.
- Embedded proxy environment cleanup for gateway services that captured shell `HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY` values during install or update.
- Gateway runtime best-practice checks (Node vs Bun, version-manager paths).
- Gateway port collision diagnostics (default `18789`).

Auth, security, and pairing

- Security warnings for open DM policies.
- Gateway auth checks for local token mode (offers token generation when no token source exists; does not overwrite token SecretRef configs).
- Device pairing trouble detection (pending first-time pair requests, pending role/scope upgrades, stale local device-token cache drift, and paired-record auth drift).

Workspace and shell

- systemd linger check on Linux.
- Workspace bootstrap file size check (truncation/near-limit warnings for context files).
- Shell completion status check and auto-install/upgrade.
- Memory search embedding provider readiness check (local model, remote API key, or QMD binary).
- Source install checks (pnpm workspace mismatch, missing UI assets, missing tsx binary).
- Writes updated config + wizard metadata.

## [​](https://docs.openclaw.ai/gateway/doctor\\#dreams-ui-backfill-and-reset)  Dreams UI backfill and reset

The Control UI Dreams scene includes **Backfill**, **Reset**, and **Clear Grounded** actions for the grounded dreaming workflow. These actions use gateway doctor-style RPC methods, but they are **not** part of `openclaw doctor` CLI repair/migration.What they do:

- **Backfill** scans historical `memory/YYYY-MM-DD.md` files in the active workspace, runs the grounded REM diary pass, and writes reversible backfill entries into `DREAMS.md`.
- **Reset** removes only those marked backfill diary entries from `DREAMS.md`.
- **Clear Grounded** removes only staged grounded-only short-term entries that came from historical replay and have not accumulated live recall or daily support yet.

What they do **not** do by themselves:

- they do not edit `MEMORY.md`
- they do not run full doctor migrations
- they do not automatically stage grounded candidates into the live short-term promotion store unless you explicitly run the staged CLI path first

If you want grounded historical replay to influence the normal deep promotion lane, use the CLI flow instead:

```
openclaw memory rem-backfill --path ./memory --stage-short-term
```

That stages grounded durable candidates into the short-term dreaming store while keeping `DREAMS.md` as the review surface.

## [​](https://docs.openclaw.ai/gateway/doctor\\#detailed-behavior-and-rationale)  Detailed behavior and rationale

0\\. Optional update (git installs)

If this is a git checkout and doctor is running interactively, it offers to update (fetch/rebase/build) before running doctor.

1\\. Config normalization

If the config contains legacy value shapes (for example `messages.ackReaction` without a channel-specific override), doctor normalizes them into the current schema.That includes legacy Talk flat fields. Current public Talk config is `talk.provider` \\+ `talk.providers.<provider>`. Doctor rewrites old `talk.voiceId` / `talk.voiceAliases` / `talk.modelId` / `talk.outputFormat` / `talk.apiKey` shapes into the provider map.

2\\. Legacy config key migrations

When the config contains deprecated keys, other commands refuse to run and ask you to run `openclaw doctor`.Doctor will:

- Explain which legacy keys were found.
- Show the migration it applied.
- Rewrite `~/.openclaw/openclaw.json` with the updated schema.

The Gateway also auto-runs doctor migrations on startup when it detects a legacy config format, so stale configs are repaired without manual intervention. Cron job store migrations are handled by `openclaw doctor --fix`.Current migrations:

- `routing.allowFrom` → `channels.whatsapp.allowFrom`
- `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups.\"*\".requireMention`
- `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
- `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
- `routing.queue` → `messages.queue`
- `routing.bindings` → top-level `bindings`
- `routing.agents`/`routing.defaultAgentId` → `agents.list` \\+ `agents.list[].default`
- legacy `talk.voiceId`/`talk.voiceAliases`/`talk.modelId`/`talk.outputFormat`/`talk.apiKey` → `talk.provider` \\+ `talk.providers.<provider>`
- `routing.agentToAgent` → `tools.agentToAgent`
- `routing.transcribeAudio` → `tools.media.audio.models`
- `messages.tts.<provider>` (`openai`/`elevenlabs`/`microsoft`/`edge`) → `messages.tts.providers.<provider>`
- `messages.tts.provider: \"edge\"` and `messages.tts.providers.edge` → `messages.tts.provider: \"microsoft\"` and `messages.tts.providers.microsoft`
- `channels.discord.voice.tts.<provider>` (`openai`/`elevenlabs`/`microsoft`/`edge`) → `channels.discord.voice.tts.providers.<provider>`
- `channels.discord.accounts.<id>.voice.tts.<provider>` (`openai`/`elevenlabs`/`microsoft`/`edge`) → `channels.discord.accounts.<id>.voice.tts.providers.<provider>`
- `plugins.entries.voice-call.config.tts.<provider>` (`openai`/`elevenlabs`/`microsoft`/`edge`) → `plugins.entries.voice-call.config.tts.providers.<provider>`
- `plugins.entries.voice-call.config.tts.provider: \"edge\"` and `plugins.entries.voice-call.config.tts.providers.edge` → `provider: \"microsoft\"` and `providers.microsoft`
- `plugins.entries.voice-call.config.provider: \"log\"` → `\"mock\"`
- `plugins.entries.voice-call.config.twilio.from` → `plugins.entries.voice-call.config.fromNumber`
- `plugins.entries.voice-call.config.streaming.sttProvider` → `plugins.entries.voice-call.config.streaming.provider`
- `plugins.entries.voice-call.config.streaming.openaiApiKey|sttModel|silenceDurationMs|vadThreshold` → `plugins.entries.voice-call.config.streaming.providers.openai.*`
- `bindings[].match.accountID` → `bindings[].match.accountId`
- For channels with named `accounts` but lingering single-account top-level channel values, move those account-scoped values into the promoted account chosen for that channel (`accounts.default` for most channels; Matrix can preserve an existing matching named/default target)
- `identity` → `agents.list[].identity`
- `agent.*` → `agents.defaults` \\+ `tools.*` (tools/elevated/exec/sandbox/subagents)
- `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks` → `agents.defaults.models` \\+ `agents.defaults.model.primary/fallbacks` \\+ `agents.defaults.imageModel.primary/fallbacks`
- remove `agents.defaults.llm`; use `models.providers.<id>.timeoutSeconds` for slow provider/model timeouts
- `browser.ssrfPolicy.allowPrivateNetwork` → `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`
- `browser.profiles.*.driver: \"extension\"` → `\"existing-session\"`
- remove `browser.relayBindHost` (legacy extension relay setting)
- legacy `models.providers.*.api: \"openai\"` → `\"openai-completions\"` (gateway startup also skips providers whose `api` is set to a future or unknown enum value rather than failing closed)

Doctor warnings also include account-default guidance for multi-account channels:

- If two or more `channels.<channel>.accounts` entries are configured without `channels.<channel>.defaultAccount` or `accounts.default`, doctor warns that fallback routing can pick an unexpected account.
- If `channels.<channel>.defaultAccount` is set to an unknown account ID, doctor warns and lists configured account IDs.

2b. OpenCode provider overrides

If you’ve added `models.providers.opencode`, `opencode-zen`, or `opencode-go` manually, it overrides the built-in OpenCode catalog from `@mariozechner/pi-ai`. That can force models onto the wrong API or zero out costs. Doctor warns so you can remove the override and restore per-model API routing + costs.

2c. Browser migration and Chrome MCP readiness

If your browser config still points at the removed Chrome extension path, doctor normalizes it to the current host-local Chrome MCP attach model:

- `browser.profiles.*.driver: \"extension\"` becomes `\"existing-session\"`
- `browser.relayBindHost` is removed

Doctor also audits the host-local Chrome MCP path when you use `defaultProfile: \"user\"` or a configured `existing-session` profile:

- checks whether Google Chrome is installed on the same host for default auto-connect profiles
- checks the detected Chrome version and warns when it is below Chrome 144
- reminds you to enable remote debugging in the browser inspect page (for example `chrome://inspect/#remote-debugging`, `brave://inspect/#remote-debugging`, or `edge://inspect/#remote-debugging`)

Doctor cannot enable the Chrome-side setting for you. Host-local Chrome MCP still requires:

- a Chromium-based browser 144+ on the gateway/node host
- the browser running locally
- remote debugging enabled in that browser
- approving the first attach consent prompt in the browser

Readiness here is only about local attach prerequisites. Existing-session keeps the current Chrome MCP route limits; advanced routes like `responsebody`, PDF export, download interception, and batch actions still require a managed browser or raw CDP profile.This check does **not** apply to Docker, sandbox, remote-browser, or other headless flows. Those continue to use raw CDP.

2d. OAuth TLS prerequisites

When an OpenAI Codex OAuth profile is configured, doctor probes the OpenAI authorization endpoint to verify that the local Node/OpenSSL TLS stack can validate the certificate chain. If the probe fails with a certificate error (for example `UNABLE_TO_GET_ISSUER_CERT_LOCALLY`, expired cert, or self-signed cert), doctor prints platform-specific fix guidance. On macOS with a Homebrew Node, the fix is usually `brew postinstall ca-certificates`. With `--deep`, the probe runs even if the gateway is healthy.

2e. Codex OAuth provider overrides

If you previously added legacy OpenAI transport settings under `models.providers.openai-codex`, they can shadow the built-in Codex OAuth provider path that newer releases use automatically. Doctor warns when it sees those old transport settings alongside Codex OAuth so you can remove or rewrite the stale transport override and get the built-in routing/fallback behavior back. Custom proxies and header-only overrides are still supported and do not trigger this warning.

2f. Codex plugin route warnings

When the bundled Codex plugin is enabled, doctor also checks whether `openai-codex/*` primary model refs still resolve through the default PI runner. That combination is valid when you want Codex OAuth/subscription auth through PI, but it is easy to confuse with the native Codex app-server harness. Doctor warns and points to the explicit app-server shape: `openai/*` plus `agentRuntime.id: \"codex\"` or `OPENCLAW_AGENT_RUNTIME=codex`.Doctor does not repair this automatically because both routes are valid:

- `openai-codex/*` \\+ PI means “use Codex OAuth/subscription auth through the normal OpenClaw runner.”
- `openai/*` \\+ `runtime: \"codex\"` means “run the embedded turn through native Codex app-server.”
- `/codex ...` means “control or bind a native Codex conversation from chat.”
- `/acp ...` or `runtime: \"acp\"` means “use the external ACP/acpx adapter.”

If the warning appears, choose the route you intended and edit config manually. Keep the warning as-is when PI Codex OAuth is intentional.

3\\. Legacy state migrations (disk layout)

Doctor can migrate older on-disk layouts into the current structure:

- Sessions store + transcripts:
  - from `~/.openclaw/sessions/` to `~/.openclaw/agents/<agentId>/sessions/`
- Agent dir:
  - from `~/.openclaw/agent/` to `~/.openclaw/agents/<agentId>/agent/`
- WhatsApp auth state (Baileys):
  - from legacy `~/.openclaw/credentials/*.json` (except `oauth.json`)
  - to `~/.openclaw/credentials/whatsapp/<accountId>/...` (default account id: `default`)

These migrations are best-effort and idempotent; doctor will emit warnings when it leaves any legacy folders behind as backups. The Gateway/CLI also auto-migrates the legacy sessions + agent dir on startup so history/auth/models land in the per-agent path without a manual doctor run. WhatsApp auth is intentionally only migrated via `openclaw doctor`. Talk provider/provider-map normalization now compares by structural equality, so key-order-only diffs no longer trigger repeat no-op `doctor --fix` changes.

3a. Legacy plugin manifest migrations

Doctor scans all installed plugin manifests for deprecated top-level capability keys (`speechProviders`, `realtimeTranscriptionProviders`, `realtimeVoiceProviders`, `mediaUnderstandingProviders`, `imageGenerationProviders`, `videoGenerationProviders`, `webFetchProviders`, `webSearchProviders`). When found, it offers to move them into the `contracts` object and rewrite the manifest file in-place. This migration is idempotent; if the `contracts` key already has the same values, the legacy key is removed without duplicating the data.

3b. Legacy cron store migrations

Doctor also checks the cron job store (`~/.openclaw/cron/jobs.json` by default, or `cron.store` when overridden) for old job shapes that the scheduler still accepts for compatibility.Current cron cleanups include:

- `jobId` → `id`
- `schedule.cron` → `schedule.expr`
- top-level payload fields (`message`, `model`, `thinking`, …) → `payload`
- top-level delivery fields (`deliver`, `channel`, `to`, `provider`, …) → `delivery`
- payload `provider` delivery aliases → explicit `delivery.channel`
- simple legacy `notify: true` webhook fallback jobs → explicit `delivery.mode=\"webhook\"` with `delivery.to=cron.webhook`

Doctor only auto-migrates `notify: true` jobs when it can do so without changing behavior. If a job combines legacy notify fallback with an existing non-webhook delivery mode, doctor warns and leaves that job for manual review.

3c. Session lock cleanup

Doctor scans every agent session directory fo...

---

## Bonjour discovery - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/bonjour

[Skip to main content](https://docs.openclaw.ai/gateway/bonjour#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Networking and discovery

Bonjour discovery

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Bonjour / mDNS discovery](https://docs.openclaw.ai/gateway/bonjour#bonjour-%2F-mdns-discovery)
- [Wide-area Bonjour (Unicast DNS-SD) over Tailscale](https://docs.openclaw.ai/gateway/bonjour#wide-area-bonjour-unicast-dns-sd-over-tailscale)
- [Gateway config (recommended)](https://docs.openclaw.ai/gateway/bonjour#gateway-config-recommended)
- [One-time DNS server setup (gateway host)](https://docs.openclaw.ai/gateway/bonjour#one-time-dns-server-setup-gateway-host)
- [Tailscale DNS settings](https://docs.openclaw.ai/gateway/bonjour#tailscale-dns-settings)
- [Gateway listener security (recommended)](https://docs.openclaw.ai/gateway/bonjour#gateway-listener-security-recommended)
- [What advertises](https://docs.openclaw.ai/gateway/bonjour#what-advertises)
- [Service types](https://docs.openclaw.ai/gateway/bonjour#service-types)
- [TXT keys (non-secret hints)](https://docs.openclaw.ai/gateway/bonjour#txt-keys-non-secret-hints)
- [Debugging on macOS](https://docs.openclaw.ai/gateway/bonjour#debugging-on-macos)
- [Debugging in Gateway logs](https://docs.openclaw.ai/gateway/bonjour#debugging-in-gateway-logs)
- [Debugging on iOS node](https://docs.openclaw.ai/gateway/bonjour#debugging-on-ios-node)
- [When to disable Bonjour](https://docs.openclaw.ai/gateway/bonjour#when-to-disable-bonjour)
- [Docker gotchas](https://docs.openclaw.ai/gateway/bonjour#docker-gotchas)
- [Troubleshooting disabled Bonjour](https://docs.openclaw.ai/gateway/bonjour#troubleshooting-disabled-bonjour)
- [Common failure modes](https://docs.openclaw.ai/gateway/bonjour#common-failure-modes)
- [Escaped instance names (\\\\032)](https://docs.openclaw.ai/gateway/bonjour#escaped-instance-names-%5C032)
- [Disabling / configuration](https://docs.openclaw.ai/gateway/bonjour#disabling-%2F-configuration)
- [Related docs](https://docs.openclaw.ai/gateway/bonjour#related-docs)

# [​](https://docs.openclaw.ai/gateway/bonjour\\#bonjour-/-mdns-discovery)  Bonjour / mDNS discovery

OpenClaw uses Bonjour (mDNS / DNS‑SD) to discover an active Gateway (WebSocket endpoint).
Multicast `local.` browsing is a **LAN-only convenience**. The bundled `bonjour`
plugin owns LAN advertising and is enabled by default. For cross-network discovery,
the same beacon can also be published through a configured wide-area DNS-SD domain.
Discovery is still best-effort and does **not** replace SSH or Tailnet-based connectivity.

## [​](https://docs.openclaw.ai/gateway/bonjour\\#wide-area-bonjour-unicast-dns-sd-over-tailscale)  Wide-area Bonjour (Unicast DNS-SD) over Tailscale

If the node and gateway are on different networks, multicast mDNS won’t cross the
boundary. You can keep the same discovery UX by switching to **unicast DNS‑SD**
(“Wide‑Area Bonjour”) over Tailscale.High‑level steps:

1. Run a DNS server on the gateway host (reachable over Tailnet).
2. Publish DNS‑SD records for `_openclaw-gw._tcp` under a dedicated zone
(example: `openclaw.internal.`).
3. Configure Tailscale **split DNS** so your chosen domain resolves via that
DNS server for clients (including iOS).

OpenClaw supports any discovery domain; `openclaw.internal.` is just an example.
iOS/Android nodes browse both `local.` and your configured wide‑area domain.

### [​](https://openclaw.ai/gateway/bonjour\\#gateway-config-recommended)  Gateway config (recommended)

```
{
  gateway: { bind: \"tailnet\" }, // tailnet-only (recommended)
  discovery: { wideArea: { enabled: true } }, // enables wide-area DNS-SD publishing
}
```

### [​](https://docs.openclaw.ai/gateway/bonjour\\#one-time-dns-server-setup-gateway-host)  One-time DNS server setup (gateway host)

```
openclaw dns setup --apply
```

This installs CoreDNS and configures it to:

- listen on port 53 only on the gateway’s Tailscale interfaces
- serve your chosen domain (example: `openclaw.internal.`) from `~/.openclaw/dns/<domain>.db`

Validate from a tailnet‑connected machine:

```
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### [​](https://docs.openclaw.ai/gateway/bonjour\\#tailscale-dns-settings)  Tailscale DNS settings

In the Tailscale admin console:

- Add a nameserver pointing at the gateway’s tailnet IP (UDP/TCP 53).
- Add split DNS so your discovery domain uses that nameserver.

Once clients accept tailnet DNS, iOS nodes and CLI discovery can browse
`_openclaw-gw._tcp` in your discovery domain without multicast.

### [​](https://docs.openclaw.ai/gateway/bonjour\\#gateway-listener-security-recommended)  Gateway listener security (recommended)

The Gateway WS port (default `18789`) binds to loopback by default. For LAN/tailnet
access, bind explicitly and keep auth enabled.For tailnet‑only setups:

- Set `gateway.bind: \"tailnet\"` in `~/.openclaw/openclaw.json`.
- Restart the Gateway (or restart the macOS menubar app).

## [​](https://docs.openclaw.ai/gateway/bonjour\\#what-advertises)  What advertises

Only the Gateway advertises `_openclaw-gw._tcp`. LAN multicast advertising is
provided by the bundled `bonjour` plugin; wide-area DNS-SD publishing remains
Gateway-owned.

## [​](https://docs.openclaw.ai/gateway/bonjour\\#service-types)  Service types

- `_openclaw-gw._tcp` — gateway transport beacon (used by macOS/iOS/Android nodes).

## [​](https://docs.openclaw.ai/gateway/bonjour\\#txt-keys-non-secret-hints)  TXT keys (non-secret hints)

The Gateway advertises small non‑secret hints to make UI flows convenient:

- `role=gateway`
- `displayName=<friendly name>`
- `lanHost=<hostname>.local`
- `gatewayPort=<port>` (Gateway WS + HTTP)
- `gatewayTls=1` (only when TLS is enabled)
- `gatewayTlsSha256=<sha256>` (only when TLS is enabled and fingerprint is available)
- `canvasPort=<port>` (only when the canvas host is enabled; currently the same as `gatewayPort`)
- `transport=gateway`
- `tailnetDns=<magicdns>` (mDNS full mode only, optional hint when Tailnet is available)
- `sshPort=<port>` (mDNS full mode only; wide-area DNS-SD may omit it)
- `cliPath=<path>` (mDNS full mode only; wide-area DNS-SD still writes it as a remote-install hint)

Security notes:

- Bonjour/mDNS TXT records are **unauthenticated**. Clients must not treat TXT as authoritative routing.
- Clients should route using the resolved service endpoint (SRV + A/AAAA). Treat `lanHost`, `tailnetDns`, `gatewayPort`, and `gatewayTlsSha256` as hints only.
- SSH auto-targeting should likewise use the resolved service host, not TXT-only hints.
- TLS pinning must never allow an advertised `gatewayTlsSha256` to override a previously stored pin.
- iOS/Android nodes should treat discovery-based direct connects as **TLS-only** and require explicit user confirmation before trusting a first-time fingerprint.

## [​](https://docs.openclaw.ai/gateway/bonjour\\#debugging-on-macos)  Debugging on macOS

Useful built‑in tools:

- Browse instances:














```
dns-sd -B _openclaw-gw._tcp local.
```

- Resolve one instance (replace `<instance>`):














```
dns-sd -L \"<instance>\" _openclaw-gw._tcp local.
```


If browsing works but resolving fails, you’re usually hitting a LAN policy or
mDNS resolver issue.

## [​](https://docs.openclaw.ai/gateway/bonjour\\#debugging-in-gateway-logs)  Debugging in Gateway logs

The Gateway writes a rolling log file (printed on startup as
`gateway log file: ...`). Look for `bonjour:` lines, especially:

- `bonjour: advertise failed ...`
- `bonjour: ... name conflict resolved` / `hostname conflict resolved`
- `bonjour: watchdog detected non-announced service ...`
- `bonjour: disabling advertiser after ... failed restarts ...`

Bonjour uses the system hostname for the advertised `.local` host when it is a
valid DNS label. If the system hostname contains spaces, underscores, or another
invalid DNS-label character, OpenClaw falls back to `openclaw.local`. Set
`OPENCLAW_MDNS_HOSTNAME=<name>` before starting the Gateway when you need an
explicit host label.

## [​](https://docs.openclaw.ai/gateway/bonjour\\#debugging-on-ios-node)  Debugging on iOS node

The iOS node uses `NWBrowser` to discover `_openclaw-gw._tcp`.To capture logs:

- Settings → Gateway → Advanced → **Discovery Debug Logs**
- Settings → Gateway → Advanced → **Discovery Logs** → reproduce → **Copy**

The log includes browser state transitions and result‑set changes.

## [​](https://docs.openclaw.ai/gateway/bonjour\\#when-to-disable-bonjour)  When to disable Bonjour

Disable Bonjour only when LAN multicast advertising is unavailable or harmful.
The common case is a Gateway running behind Docker bridge networking, WSL, or a
network policy that drops mDNS multicast. In those environments the Gateway is
still reachable through its published URL, SSH, Tailnet, or wide-area DNS-SD,
but LAN auto-discovery is not reliable.Prefer the existing environment override when the problem is deployment-scoped:

```
OPENCLAW_DISABLE_BONJOUR=1
```

That disables LAN multicast advertising without changing plugin configuration.
It is safe for Docker images, service files, launch scripts, and one-off
debugging because the setting disappears when the environment does.Use plugin configuration only when you intentionally want to turn off the
bundled LAN discovery plugin for that OpenClaw config:

```
openclaw plugins disable bonjour
```

## [​](https://docs.openclaw.ai/gateway/bonjour\\#docker-gotchas)  Docker gotchas

The bundled Bonjour plugin auto-disables LAN multicast advertising in detected
containers when `OPENCLAW_DISABLE_BONJOUR` is unset. Docker bridge networks
usually do not forward mDNS multicast (`224.0.0.251:5353`) between the container
and the LAN, so advertising from the container rarely makes discovery work.Important gotchas:

- Disabling Bonjour does not stop the Gateway. It only stops LAN multicast
advertising.
- Disabling Bonjour does not change `gateway.bind`; Docker still defaults to
`OPENCLAW_GATEWAY_BIND=lan` so the published host port can work.
- Disabling Bonjour does not disable wide-area DNS-SD. Use wide-area discovery
or Tailnet when the Gateway and node are not on the same LAN.
- Reusing the same `OPENCLAW_CONFIG_DIR` outside Docker does not persist the
container auto-disable policy.
- Set `OPENCLAW_DISABLE_BONJOUR=0` only for host networking, macvlan, or another
network where mDNS multicast is known to pass; set it to `1` to force-disable.

## [​](https://docs.openclaw.ai/gateway/bonjour\\#troubleshooting-disabled-bonjour)  Troubleshooting disabled Bonjour

If a node no longer auto-discovers the Gateway after Docker setup:

1. Confirm whether the Gateway is running in auto, forced-on, or forced-off mode:














```
docker compose config | grep OPENCLAW_DISABLE_BONJOUR
```

2. Confirm the Gateway itself is reachable through the published port:














```
curl -fsS http://127.0.0.1:18789/healthz
```

3. Use a direct target when Bonjour is disabled:   - Control UI or local tools: `http://127.0.0.1:18789`
   - LAN clients: `http://<gateway-host>:18789`
   - Cross-network clients: Tailnet MagicDNS, Tailnet IP, SSH tunnel, or
     wide-area DNS-SD
4. If you deliberately enabled Bonjour in Docker with
`OPENCLAW_DISABLE_BONJOUR=0`, test multicast from the host:














```
dns-sd -B _openclaw-gw._tcp local.
```










If browsing is empty or the Gateway logs show repeated ciao watchdog
cancellations, restore `OPENCLAW_DISABLE_BONJOUR=1` and use a direct or
Tailnet route.

## [​](https://docs.openclaw.ai/gateway/bonjour\\#common-failure-modes)  Common failure modes

- **Bonjour doesn’t cross networks**: use Tailnet or SSH.
- **Multicast blocked**: some Wi‑Fi networks disable mDNS.
- **Advertiser stuck in probing/announcing**: hosts with blocked multicast,
container bridges, WSL, or interface churn can leave the ciao advertiser in a
non-announced state. OpenClaw retries a few times and then disables Bonjour
for the current Gateway process instead of restarting the advertiser forever.
- **Docker bridge networking**: Bonjour auto-disables in detected containers.
Set `OPENCLAW_DISABLE_BONJOUR=0` only for host, macvlan, or another
mDNS-capable network.
- **Sleep / interface churn**: macOS may temporarily drop mDNS results; retry.
- **Browse works but resolve fails**: keep machine names simple (avoid emojis or
punctuation), then restart the Gateway. The service instance name derives from
the host name, so overly complex names can confuse some resolvers.

## [​](https://docs.openclaw.ai/gateway/bonjour\\#escaped-instance-names-\\032)  Escaped instance names (`\\032`)

Bonjour/DNS‑SD often escapes bytes in service instance names as decimal `\\DDD`
sequences (e.g. spaces become `\\032`).

- This is normal at the protocol level.
- UIs should decode for display (iOS uses `BonjourEscapes.decode`).

## [​](https://docs.openclaw.ai/gateway/bonjour\\#disabling-/-configuration)  Disabling / configuration

- `openclaw plugins disable bonjour` disables LAN multicast advertising by disabling the bundled plugin.
- `openclaw plugins enable bonjour` restores the default LAN discovery plugin.
- `OPENCLAW_DISABLE_BONJOUR=1` disables LAN multicast advertising without changing plugin config; accepted truthy values are `1`, `true`, `yes`, and `on` (legacy: `OPENCLAW_DISABLE_BONJOUR`).
- `OPENCLAW_DISABLE_BONJOUR=0` forces LAN multicast advertising on, including inside detected containers; accepted falsy values are `0`, `false`, `no`, and `off`.
- When `OPENCLAW_DISABLE_BONJOUR` is unset, Bonjour advertises on normal hosts and auto-disables inside detected containers.
- `gateway.bind` in `~/.openclaw/openclaw.json` controls the Gateway bind mode.
- `OPENCLAW_SSH_PORT` overrides the SSH port when `sshPort` is advertised (legacy: `OPENCLAW_SSH_PORT`).
- `OPENCLAW_TAILNET_DNS` publishes a MagicDNS hint in TXT when mDNS full mode is enabled (legacy: `OPENCLAW_TAILNET_DNS`).
- `OPENCLAW_CLI_PATH` overrides the advertised CLI path (legacy: `OPENCLAW_CLI_PATH`).

## [​](https://docs.openclaw.ai/gateway/bonjour\\#related-docs)  Related docs

- Discovery policy and transport selection: [Discovery](https://docs.openclaw.ai/gateway/discovery)
- Node pairing + approvals: [Gateway pairing](https://docs.openclaw.ai/gateway/pairing)

[Discovery and transports](https://docs.openclaw.ai/gateway/discovery) [Remote access](https://docs.openclaw.ai/gateway/remote)

Ctrl+I

---

## CLI backends - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/cli-backends

[Skip to main content](https://docs.openclaw.ai/gateway/cli-backends#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c652313385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Protocols and APIs

CLI backends

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Beginner-friendly quick start](https://docs.openclaw.ai/gateway/cli-backends#beginner-friendly-quick-start)
- [Using it as a fallback](https://docs.openclaw.ai/gateway/cli-backends#using-it-as-a-fallback)
- [Configuration overview](https://docs.openclaw.ai/gateway/cli-backends#configuration-overview)
- [Example configuration](https://docs.openclaw.ai/gateway/cli-backends#example-configuration)
- [How it works](https://docs.openclaw.ai/gateway/cli-backends#how-it-works)
- [Sessions](https://docs.openclaw.ai/gateway/cli-backends#sessions)
- [Images (pass-through)](https://docs.openclaw.ai/gateway/cli-backends#images-pass-through)
- [Inputs / outputs](https://docs.openclaw.ai/gateway/cli-backends#inputs-%2F-outputs)
- [Defaults (plugin-owned)](https://docs.openclaw.ai/gateway/cli-backends#defaults-plugin-owned)
- [Plugin-owned defaults](https://docs.openclaw.ai/gateway/cli-backends#plugin-owned-defaults)
- [Bundle MCP overlays](https://docs.openclaw.ai/gateway/cli-backends#bundle-mcp-overlays)
- [Limitations](https://docs.openclaw.ai/gateway/cli-backends#limitations)
- [Troubleshooting](https://docs.openclaw.ai/gateway/cli-backends#troubleshooting)
- [Related](https://docs.openclaw.ai/gateway/cli-backends#related)

OpenClaw can run **local AI CLIs** as a **text-only fallback** when API providers are down,
rate-limited, or temporarily misbehaving. This is intentionally conservative:

- **OpenClaw tools are not injected directly**, but backends with `bundleMcp: true`
can receive gateway tools via a loopback MCP bridge.
- **JSONL streaming** for CLIs that support it.
- **Sessions are supported** (so follow-up turns stay coherent).
- **Images can be passed through** if the CLI accepts image paths.

This is designed as a **safety net** rather than a primary path. Use it when you
want “always works” text responses without relying on external APIs.If you want a full harness runtime with ACP session controls, background tasks,
thread/conversation binding, and persistent external coding sessions, use
[ACP Agents](https://docs.openclaw.ai/tools/acp-agents) instead. CLI backends are not ACP.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#beginner-friendly-quick-start)  Beginner-friendly quick start

You can use Codex CLI **without any config** (the bundled OpenAI plugin
registers a default backend):

```
openclaw agent --message \"hi\" --model codex-cli/gpt-5.5
```

If your gateway runs under launchd/systemd and PATH is minimal, add just the
command path:

```
{
  agents: {
    defaults: {
      cliBackends: {
        \"codex-cli\": {
          command: \"/opt/homebrew/bin/codex\",
        },
      },
    },
  },
}
```

That’s it. No keys, no extra auth config needed beyond the CLI itself.If you use a bundled CLI backend as the **primary message provider** on a
gateway host, OpenClaw now auto-loads the owning bundled plugin when your config
explicitly references that backend in a model ref or under
`agents.defaults.cliBackends`.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#using-it-as-a-fallback)  Using it as a fallback

Add a CLI backend to your fallback list so it only runs when primary models fail:

```
{
  agents: {
    defaults: {
      model: {
        primary: \"anthropic/claude-opus-4-6\",
        fallbacks: [\"codex-cli/gpt-5.5\"],
      },
      models: {
        \"anthropic/claude-opus-4-6\": { alias: \"Opus\" },
        \"codex-cli/gpt-5.5\": {},
      },
    },
  },
}
```

Notes:

- If you use `agents.defaults.models` (allowlist), you must include your CLI backend models there too.
- If the primary provider fails (auth, rate limits, timeouts), OpenClaw will
try the CLI backend next.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#configuration-overview)  Configuration overview

All CLI backends live under:

```
agents.defaults.cliBackends
```

Each entry is keyed by a **provider id** (e.g. `codex-cli`, `my-cli`).
The provider id becomes the left side of your model ref:

```
<provider>/<model>
```

### [​](https://docs.openclaw.ai/gateway/cli-backends\\#example-configuration)  Example configuration

```
{
  agents: {
    defaults: {
      cliBackends: {
        \"codex-cli\": {
          command: \"/opt/homebrew/bin/codex\",
        },
        \"my-cli\": {
          command: \"my-cli\",
          args: [\"--json\"],
          output: \"json\",
          input: \"arg\",
          modelArg: \"--model\",
          modelAliases: {
            \"claude-opus-4-6\": \"opus\",
            \"claude-sonnet-4-6\": \"sonnet\",
          },
          sessionArg: \"--session\",
          sessionMode: \"existing\",
          sessionIdFields: [\"session_id\", \"conversation_id\"],
          systemPromptArg: \"--system\",
          // For CLIs with a dedicated prompt-file flag:
          // systemPromptFileArg: \"--system-file\",
          // Codex-style CLIs can point at a prompt file instead:
          // systemPromptFileConfigArg: \"-c\",
          // systemPromptFileConfigKey: \"model_instructions_file\",
          systemPromptWhen: \"first\",
          imageArg: \"--image\",
          imageMode: \"repeat\",
          serialize: true,
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#how-it-works)  How it works

1. **Selects a backend** based on the provider prefix (`codex-cli/...`).
2. **Builds a system prompt** using the same OpenClaw prompt + workspace context.
3. **Executes the CLI** with a session id (if supported) so history stays consistent.
The bundled `claude-cli` backend keeps a Claude stdio process alive per
OpenClaw session and sends follow-up turns over stream-json stdin.
4. **Parses output** (JSON or plain text) and returns the final text.
5. **Persists session ids** per backend, so follow-ups reuse the same CLI session.

The bundled Anthropic `claude-cli` backend is supported again. Anthropic staff
told us OpenClaw-style Claude CLI usage is allowed again, so OpenClaw treats
`claude -p` usage as sanctioned for this integration unless Anthropic publishes
a new policy.

The bundled OpenAI `codex-cli` backend passes OpenClaw’s system prompt through
Codex’s `model_instructions_file` config override (`-c model_instructions_file=\"...\"`). Codex does not expose a Claude-style
`--append-system-prompt` flag, so OpenClaw writes the assembled prompt to a
temporary file for each fresh Codex CLI session.The bundled Anthropic `claude-cli` backend receives the OpenClaw skills snapshot
two ways: the compact OpenClaw skills catalog in the appended system prompt, and
a temporary Claude Code plugin passed with `--plugin-dir`. The plugin contains
only the eligible skills for that agent/session, so Claude Code’s native skill
resolver sees the same filtered set that OpenClaw would otherwise advertise in
the prompt. Skill env/API key overrides are still applied by OpenClaw to the
child process environment for the run.Claude CLI also has its own noninteractive permission mode. OpenClaw maps that
to the existing exec policy instead of adding Claude-specific config: when the
effective requested exec policy is YOLO (`tools.exec.security: \"full\"` and
`tools.exec.ask: \"off\"`), OpenClaw adds `--permission-mode bypassPermissions`.\nPer-agent `agents.list[].tools.exec` settings override global `tools.exec` for
that agent. To force a different Claude mode, set explicit raw backend args
such as `--permission-mode default` or `--permission-mode acceptEdits` under
`agents.defaults.cliBackends.claude-cli.args` and matching `resumeArgs`.Before OpenClaw can use the bundled `claude-cli` backend, Claude Code itself
must already be logged in on the same host:

```
claude auth login
claude auth status --text
openclaw models auth login --provider anthropic --method cli --set-default
```

Use `agents.defaults.cliBackends.claude-cli.command` only when the `claude`
binary is not already on `PATH`.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#sessions)  Sessions

- If the CLI supports sessions, set `sessionArg` (e.g. `--session-id`) or
`sessionArgs` (placeholder `{sessionId}`) when the ID needs to be inserted
into multiple flags.
- If the CLI uses a **resume subcommand** with different flags, set
`resumeArgs` (replaces `args` when resuming) and optionally `resumeOutput`
(for non-JSON resumes).
- `sessionMode`:

  - `always`: always send a session id (new UUID if none stored).
  - `existing`: only send a session id if one was stored before.
  - `none`: never send a session id.
- `claude-cli` defaults to `liveSession: \"claude-stdio\"`, `output: \"jsonl\"`,
and `input: \"stdin\"` so follow-up turns reuse the live Claude process while
it is active. Warm stdio is the default now, including for custom configs
that omit transport fields. If the Gateway restarts or the idle process
exits, OpenClaw resumes from the stored Claude session id. Stored session
ids are verified against an existing readable project transcript before
resume, so phantom bindings are cleared with `reason=transcript-missing`
instead of silently starting a fresh Claude CLI session under `--resume`.
- Stored CLI sessions are provider-owned continuity. The implicit daily session
reset does not cut them; `/reset` and explicit `session.reset` policies still
do.

Serialization notes:

- `serialize: true` keeps same-lane runs ordered.
- Most CLIs serialize on one provider lane.
- OpenClaw drops stored CLI session reuse when the selected auth identity changes,
including a changed auth profile id, static API key, static token, or OAuth
account identity when the CLI exposes one. OAuth access and refresh token
rotation does not cut the stored CLI session. If a CLI does not expose a
stable OAuth account id, OpenClaw lets that CLI enforce resume permissions.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#images-pass-through)  Images (pass-through)

If your CLI accepts image paths, set `imageArg`:

```
imageArg: \"--image\",
imageMode: \"repeat\"
```

OpenClaw will write base64 images to temp files. If `imageArg` is set, those
paths are passed as CLI args. If `imageArg` is missing, OpenClaw appends the
file paths to the prompt (path injection), which is enough for CLIs that auto-
load local files from plain paths.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#inputs-/-outputs)  Inputs / outputs

- `output: \"json\"` (default) tries to parse JSON and extract text + session id.
- For Gemini CLI JSON output, OpenClaw reads reply text from `response` and
usage from `stats` when `usage` is missing or empty.
- `output: \"jsonl\"` parses JSONL streams (for example Codex CLI `--json`) and extracts the final agent message plus session
identifiers when present.
- `output: \"text\"` treats stdout as the final response.

Input modes:

- `input: \"arg\"` (default) passes the prompt as the last CLI arg.
- `input: \"stdin\"` sends the prompt via stdin.
- If the prompt is very long and `maxPromptArgChars` is set, stdin is used.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#defaults-plugin-owned)  Defaults (plugin-owned)

The bundled OpenAI plugin also registers a default for `codex-cli`:

- `command: \"codex\"`
- `args: [\"exec\",\"--json\",\"--color\",\"never\",\"--sandbox\",\"workspace-write\",\"--skip-git-repo-check\"]`
- `resumeArgs: [\"exec\",\"resume\",\"{sessionId}\",\"-c\",\"sandbox_mode=\\\"workspace-write\\\"\",\"--skip-git-repo-check\"]`
- `output: \"jsonl\"`
- `resumeOutput: \"text\"`
- `modelArg: \"--model\"`
- `imageArg: \"--image\"`
- `sessionMode: \"existing\"`

The bundled Google plugin also registers a default for `google-gemini-cli`:

- `command: \"gemini\"`
- `args: [\"--output-format\", \"json\", \"--prompt\", \"{prompt}\"]`
- `resumeArgs: [\"--resume\", \"{sessionId}\", \"--output-format\", \"json\", \"--prompt\", \"{prompt}\"]`
- `imageArg: \"@\"`
- `imagePathScope: \"workspace\"`
- `modelArg: \"--model\"`
- `sessionMode: \"existing\"`
- `sessionIdFields: [\"session_id\", \"sessionId\"]`

Prerequisite: the local Gemini CLI must be installed and available as
`gemini` on `PATH` (`brew install gemini-cli` or
`npm install -g @google/gemini-cli`).Gemini CLI JSON notes:

- Reply text is read from the JSON `response` field.
- Usage falls back to `stats` when `usage` is absent or empty.
- `stats.cached` is normalized into OpenClaw `cacheRead`.
- If `stats.input` is missing, OpenClaw derives input tokens from
`stats.input_tokens - stats.cached`.

Override only if needed (common: absolute `command` path).

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#plugin-owned-defaults)  Plugin-owned defaults

CLI backend defaults are now part of the plugin surface:

- Plugins register them with `api.registerCliBackend(...)`.
- The backend `id` becomes the provider prefix in model refs.
- User config in `agents.defaults.cliBackends.<id>` still overrides the plugin default.
- Backend-specific config cleanup stays plugin-owned through the optional
`normalizeConfig` hook.

Plugins that need tiny prompt/message compatibility shims can declare
bidirectional text transforms without replacing a provider or CLI backend:

```
api.registerTextTransforms({
  input: [\\\n    { from: /red basket/g, to: \"blue basket\" },\\\n    { from: /paper ticket/g, to: \"digital ticket\" },\\\n    { from: /left shelf/g, to: \"right shelf\" },\\\n  ],\n  output: [\\\n    { from: /blue basket/g, to: \"red basket\" },\\\n    { from: /digital ticket/g, to: \"paper ticket\" },\\\n    { from: /right shelf/g, to: \"left shelf\" },\\\n  ],
});
```

`input` rewrites the system prompt and user prompt passed to the CLI. `output`
rewrites streamed assistant deltas and parsed final text before OpenClaw handles
its own control markers and channel delivery.For CLIs that emit Claude Code stream-json compatible JSONL, set
`jsonlDialect: \"claude-stream-json\"` on that backend’s config.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#bundle-mcp-overlays)  Bundle MCP overlays

CLI backends do **not** receive OpenClaw tool calls directly, but a backend can
opt into a generated MCP config overlay with `bundleMcp: true`.Current bundled behavior:

- `claude-cli`: generated strict MCP config file
- `codex-cli`: inline config overrides for `mcp_servers`; the generated
OpenClaw loopback server is marked with Codex’s per-server tool approval mode
so MCP calls cannot stall on local approval prompts
- `google-gemini-cli`: generated Gemini system settings file

When bundle MCP is enabled, OpenClaw:

- spawns a loopback HTTP MCP server that exposes gateway tools to the CLI process
- authenticates the bridge with a per-session token (`OPENCLAW_MCP_TOKEN`)
- scopes tool access to the current session, account, and channel context
- loads enabled bundle-MCP servers for the current workspace
- merges them with any existing backend MCP config/settings shape
- rewrites the launch config using the backend-owned integration mode from the owning extension

If no MCP servers are enabled, OpenClaw still injects a strict config when a
backend opts into bundle MCP so background runs stay isolated.Session-scoped bundled MCP runtimes are cached for reuse within a session, then
reaped after `mcp.sessionIdleTtlMs` milliseconds of idle time (default 10
minutes; set `0` to disable). One-shot embedded runs such as auth probes,
slug generation, and active-memory recall request cleanup at run end so stdio
children and Streamable HTTP/SSE streams do not outlive the run.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#limitations)  Limitations

- **No direct OpenClaw tool calls.** OpenClaw does not inject tool calls into
the CLI backend protocol. Backends only see gateway tools when they opt into
`bundleMcp: true`.
- **Streaming is backend-specific.** Some backends stream JSONL; others buffer
until exit.
- **Structured outputs** depend on the CLI’s JSON format.
- **Codex CLI sessions** resume via text output (no JSONL), which is less
structured than the initial `--json` run. OpenClaw sessions still work
normally.

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#troubleshooting)  Troubleshooting

- **CLI not found**: set `command` to a full path.
- **Wrong model name**: use `modelAliases` to map `provider/model` → CLI model.
- **No session continuity**: ensure `sessionArg` is set and `sessionMode` is not
`none` (Codex CLI currently cannot resume with JSON output).
- **Images ignored**: set `imageArg` (and verify CLI supports file paths).

## [​](https://docs.openclaw.ai/gateway/cli-backends\\#related)  Related

- [Gateway runbook](https://docs.openclaw.ai/gateway)
- [Local models](https://docs.openclaw.ai/gateway/local-models)

[Tools invoke API](https://docs.openclaw.ai/gateway/tools-invoke-http-api) [Local models](https://docs.openclaw.ai/gateway/local-models)

Ctrl+I

---

## Secrets apply plan contract - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/secrets-plan-contract

[Skip to main content](https://docs.openclaw.ai/gateway/secrets-plan-contract#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Authentication and secrets

Secrets apply plan contract

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Plan file shape](https://docs.openclaw.ai/gateway/secrets-plan-contract#plan-file-shape)
- [Supported target scope](https://docs.openclaw.ai/gateway/secrets-plan-contract#supported-target-scope)
- [Target type behavior](https://docs.openclaw.ai/gateway/secrets-plan-contract#target-type-behavior)
- [Path validation rules](https://docs.openclaw.ai/gateway/secrets-plan-contract#path-validation-rules)
- [Failure behavior](https://docs.openclaw.ai/gateway/secrets-plan-contract#failure-behavior)
- [Exec provider consent behavior](https://docs.openclaw.ai/gateway/secrets-plan-contract#exec-provider-consent-behavior)
- [Runtime and audit scope notes](https://docs.openclaw.ai/gateway/secrets-plan-contract#runtime-and-audit-scope-notes)
- [Operator checks](https://docs.openclaw.ai/gateway/secrets-plan-contract#operator-checks)
- [Related docs](https://docs.openclaw.ai/gateway/secrets-plan-contract#related-docs)

This page defines the strict contract enforced by `openclaw secrets apply`.If a target does not match these rules, apply fails before mutating configuration.

## [​](https://docs.openclaw.ai/gateway/secrets-plan-contract\#plan-file-shape)  Plan file shape

`openclaw secrets apply --from <plan.json>` expects a `targets` array of plan targets:

```
{
  version: 1,
  protocolVersion: 1,
  targets: [\
    {\
      type: \"models.providers.apiKey\",\
      path: \"models.providers.openai.apiKey\",\
      pathSegments: [\"models\", \"providers\", \"openai\", \"apiKey\"],\
      providerId: \"openai\",\
      ref: { source: \"env\", provider: \"default\", id: \"OPENAI_API_KEY\" },\
    },\\
    {\
      type: \"auth-profiles.api_key.key\",\
      path: \"profiles.openai:default.key\",\
      pathSegments: [\"profiles\", \"openai:default\", \"key\"],\
      agentId: \"main\",\
      ref: { source: \"env\", provider: \"default\", id: \"OPENAI_API_KEY\" },\
    },\\
  ],
}
```

## [​](https://docs.openclaw.ai/gateway/secrets-plan-contract\#supported-target-scope)  Supported target scope

Plan targets are accepted for supported credential paths in:

- [SecretRef Credential Surface](https://docs.openclaw.ai/reference/secretref-credential-surface)

## [​](https://docs.openclaw.ai/gateway/secrets-plan-contract\#target-type-behavior)  Target type behavior

General rule:

- `target.type` must be recognized and must match the normalized `target.path` shape.

Compatibility aliases remain accepted for existing plans:

- `models.providers.apiKey`
- `skills.entries.apiKey`
- `channels.googlechat.serviceAccount`

## [​](https://docs.openclaw.ai/gateway/secrets-plan-contract\#path-validation-rules)  Path validation rules

Each target is validated with all of the following:

- `type` must be a recognized target type.
- `path` must be a non-empty dot path.
- `pathSegments` can be omitted. If provided, it must normalize to exactly the same path as `path`.
- Forbidden segments are rejected: `__proto__`, `prototype`, `constructor`.
- The normalized path must match the registered path shape for the target type.
- If `providerId` or `accountId` is set, it must match the id encoded in the path.
- `auth-profiles.json` targets require `agentId`.
- When creating a new `auth-profiles.json` mapping, include `authProfileProvider`.

## [​](https://docs.openclaw.ai/gateway/secrets-plan-contract\#failure-behavior)  Failure behavior

If a target fails validation, apply exits with an error like:

```
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

No writes are committed for an invalid plan.

## [​](https://docs.openclaw.ai/gateway/secrets-plan-contract\#exec-provider-consent-behavior)  Exec provider consent behavior

- `--dry-run` skips exec SecretRef checks by default.
- Plans containing exec SecretRefs/providers are rejected in write mode unless `--allow-exec` is set.
- When validating/applying exec-containing plans, pass `--allow-exec` in both dry-run and write commands.

## [​](https://docs.openclaw.ai/gateway/secrets-plan-contract\#runtime-and-audit-scope-notes)  Runtime and audit scope notes

- Ref-only `auth-profiles.json` entries (`keyRef`/`tokenRef`) are included in runtime resolution and audit coverage.
- `secrets apply` writes supported `openclaw.json` targets, supported `auth-profiles.json` targets, and optional scrub targets.

## [​](https://docs.openclaw.ai/gateway/secrets-plan-contract\#operator-checks)  Operator checks

```
# Validate plan without writes
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# Then apply for real
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json

# For exec-containing plans, opt in explicitly in both modes
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run --allow-exec
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --allow-exec
```

If apply fails with an invalid target path message, regenerate the plan with `openclaw secrets configure` or fix the target path to a supported shape above.

## [​](https://docs.openclaw.ai/gateway/secrets-plan-contract\#related-docs)  Related docs

- [Secrets Management](https://docs.openclaw.ai/gateway/secrets)
- [CLI `secrets`](https://docs.openclaw.ai/cli/secrets)
- [SecretRef Credential Surface](https://docs.openclaw.ai/reference/secretref-credential-surface)
- [Configuration Reference](https://docs.openclaw.ai/gateway/configuration-reference)

[Secrets management](https://docs.openclaw.ai/gateway/secrets) [Trusted proxy auth](https://docs.openclaw.ai/gateway/trusted-proxy-auth)

Ctrl+I

---

## Local models - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/local-models

[Skip to main content](https://docs.openclaw.ai/gateway/local-models#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Protocols and APIs

Local models

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Recommended: LM Studio + large local model (Responses API)](https://docs.openclaw.ai/gateway/local-models#recommended-lm-studio-%2B-large-local-model-responses-api)
- [Hybrid config: hosted primary, local fallback](https://docs.openclaw.ai/gateway/local-models#hybrid-config-hosted-primary-local-fallback)
- [Local-first with hosted safety net](https://docs.openclaw.ai/gateway/local-models#local-first-with-hosted-safety-net)
- [Regional hosting / data routing](https://docs.openclaw.ai/gateway/local-models#regional-hosting-%2F-data-routing)
- [Other OpenAI-compatible local proxies](https://docs.openclaw.ai/gateway/local-models#other-openai-compatible-local-proxies)
- [Troubleshooting](https://docs.openclaw.ai/gateway/local-models#troubleshooting)
- [Related](https://docs.openclaw.ai/gateway/local-models#related)

Local is doable, but OpenClaw expects large context + strong defenses against prompt injection. Small cards truncate context and leak safety. Aim high: **≥2 maxed-out Mac Studios or equivalent GPU rig (~$30k+)**. A single **24 GB** GPU works only for lighter prompts with higher latency. Use the **largest / full-size model variant you can run**; aggressively quantized or “small” checkpoints raise prompt-injection risk (see [Security](https://docs.openclaw.ai/gateway/security)).If you want the lowest-friction local setup, start with [LM Studio](https://docs.openclaw.ai/providers/lmstudio) or [Ollama](https://docs.openclaw.ai/providers/ollama) and `openclaw onboard`. This page is the opinionated guide for higher-end local stacks and custom OpenAI-compatible local servers.

**WSL2 + Ollama + NVIDIA/CUDA users:** The official Ollama Linux installer enables a systemd service with `Restart=always`. On WSL2 GPU setups, autostart can reload the last model during boot and pin host memory. If your WSL2 VM repeatedly restarts after enabling Ollama, see [WSL2 crash loop](https://docs.openclaw.ai/providers/ollama#wsl2-crash-loop-repeated-reboots).

## [​](https://docs.openclaw.ai/gateway/local-models\\#recommended-lm-studio-+-large-local-model-responses-api)  Recommended: LM Studio + large local model (Responses API)

Best current local stack. Load a large model in LM Studio (for example, a full-size Qwen, DeepSeek, or Llama build), enable the local server (default `http://127.0.0.1:1234`), and use Responses API to keep reasoning separate from final text.

```
{
  agents: {
    defaults: {
      model: { primary: \"lmstudio/my-local-model\" },
      models: {
        \"anthropic/claude-opus-4-6\": { alias: \"Opus\" },
        \"lmstudio/my-local-model\": { alias: \"Local\" },
      },
    },
  },
  models: {
    mode: \"merge\",
    providers: {
      lmstudio: {
        baseUrl: \"http://127.0.0.1:1234/v1\",
        apiKey: \"lmstudio\",
        api: \"openai-responses\",
        models: [\\\
          {\\\
            id: \"my-local-model\\\",\\\
            name: \"Local Model\\\",\\\
            reasoning: false,\\\n            input: [\\\"text\\\"],\\\
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\
            contextWindow: 196608,\\\n            maxTokens: 8192,\\\n          },\\\n        ],
      },
    },
  },
}
```

**Setup checklist**

- Install LM Studio: [https://lmstudio.ai](https://lmstudio.ai/)
- In LM Studio, download the **largest model build available** (avoid “small”/heavily quantized variants), start the server, confirm `http://127.0.0.1:1234/v1/models` lists it.
- Replace `my-local-model` with the actual model ID shown in LM Studio.
- Keep the model loaded; cold-load adds startup latency.
- Adjust `contextWindow`/`maxTokens` if your LM Studio build differs.
- For WhatsApp, stick to Responses API so only final text is sent.

Keep hosted models configured even when running local; use `models.mode: \"merge\"` so fallbacks stay available.

### [​](https://docs.openclaw.ai/gateway/local-models\\#hybrid-config-hosted-primary-local-fallback)  Hybrid config: hosted primary, local fallback

```
{
  agents: {
    defaults: {
      model: {
        primary: \"anthropic/claude-sonnet-4-6\",
        fallbacks: [\"lmstudio/my-local-model\", \"anthropic/claude-opus-4-6\"],
      },
      models: {
        \"anthropic/claude-sonnet-4-6\": { alias: \"Sonnet\" },
        \"lmstudio/my-local-model\": { alias: \"Local\" },
        \""anthropic/claude-opus-4-6\": { alias: \"Opus\" },
      },
    },
  },
  models: {
    mode: \"merge\",
    providers: {
      lmstudio: {
        baseUrl: \"http://127.0.0.1:1234/v1\",
        apiKey: \"lmstudio\",
        api: \"openai-responses\",
        models: [\\\
          {\\\
            id: \"my-local-model\\\",\\\
            name: \"Local Model\\\",\\\
            reasoning: false,\\\n            input: [\\\"text\\\"],\\\
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\
            contextWindow: 196608,\\\n            maxTokens: 8192,\\\n          },\\\n        ],
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/local-models\\#local-first-with-hosted-safety-net)  Local-first with hosted safety net

Swap the primary and fallback order; keep the same providers block and `models.mode: \"merge\"` so you can fall back to Sonnet or Opus when the local box is down.

### [​](https://docs.openclaw.ai/gateway/local-models\\#regional-hosting-/-data-routing)  Regional hosting / data routing

- Hosted MiniMax/Kimi/GLM variants also exist on OpenRouter with region-pinned endpoints (e.g., US-hosted). Pick the regional variant there to keep traffic in your chosen jurisdiction while still using `models.mode: \"merge\"` for Anthropic/OpenAI fallbacks.
- Local-only remains the strongest privacy path; hosted regional routing is the middle ground when you need provider features but want control over data flow.

## [​](https://docs.openclaw.ai/gateway/local-models\\#other-openai-compatible-local-proxies)  Other OpenAI-compatible local proxies

MLX (`mlx_lm.server`), vLLM, SGLang, LiteLLM, OAI-proxy, or custom
gateways work if they expose an OpenAI-style `/v1/chat/completions`
endpoint. Use the Chat Completions adapter unless the backend explicitly
documents `/v1/responses` support. Replace the provider block above with your
endpoint and model ID:

```
{
  agents: {
    defaults: {
      model: { primary: \"local/my-local-model\" },
    },
  },
  models: {
    mode: \"merge\",
    providers: {
      local: {
        baseUrl: \"http://127.0.0.1:8000/v1\",
        apiKey: \"sk-local\",
        api: \"openai-completions\",
        timeoutSeconds: 300,
        models: [\\\
          {\\\
            id: \"my-local-model\\\",\\\
            name: \"Local Model\\\",\\\
            reasoning: false,\\\n            input: [\\\"text\\\"],\\\
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\
            contextWindow: 120000,\\\n            maxTokens: 8192,\\\n          },\\\n        ],
      },
    },
  },
}
```

If `api` is omitted on a custom provider with a `baseUrl`, OpenClaw defaults to
`openai-completions`. Loopback endpoints such as `127.0.0.1` are trusted
automatically; LAN, tailnet, and private DNS endpoints still need
`request.allowPrivateNetwork: true`.The `models.providers.<id>.models[].id` value is provider-local. Do not
include the provider prefix there. For example, an MLX server started with
`mlx_lm.server --model mlx-community/Qwen3-30B-A3B-6bit` should use this
catalog id and model ref:

- `models.providers.mlx.models[].id: \"mlx-community/Qwen3-30B-A3B-6bit\"`
- `agents.defaults.model.primary: \"mlx/mlx-community/Qwen3-30B-A3B-6bit\"`

Set `input: [\"text\", \"image\"]` on local or proxied vision models so image
attachments are injected into agent turns. Interactive custom-provider
onboarding infers common vision model IDs and asks only for unknown names.
Non-interactive onboarding uses the same inference; use `--custom-image-input`
for unknown vision IDs or `--custom-text-input` when a known-looking model is
text-only behind your endpoint.Keep `models.mode: \"merge\"` so hosted models stay available as fallbacks.
Use `models.providers.<id>.timeoutSeconds` for slow local or remote model
servers before raising `agents.defaults.timeoutSeconds`. The provider timeout
applies only to model HTTP requests, including connect, headers, body streaming,
and the total guarded-fetch abort.

For custom OpenAI-compatible providers, persisting a non-secret local marker such as `apiKey: \"ollama-local\"` is accepted when `baseUrl` resolves to loopback, a private LAN, `.local`, or a bare hostname. OpenClaw treats it as a valid local credential instead of reporting a missing key. Use a real value for any provider that accepts a public hostname.

Behavior note for local/proxied `/v1` backends:

- OpenClaw treats these as proxy-style OpenAI-compatible routes, not native
OpenAI endpoints
- native OpenAI-only request shaping does not apply here: no
`service_tier`, no Responses `store`, no OpenAI reasoning-compat payload
shaping, and no prompt-cache hints
- hidden OpenClaw attribution headers (`originator`, `version`, `User-Agent`)
are not injected on these custom proxy URLs

Compatibility notes for stricter OpenAI-compatible backends:

- Some servers accept only string `messages[].content` on Chat Completions, not
structured content-part arrays. Set
`models.providers.<provider>.models[].compat.requiresStringContent: true` for
those endpoints.
- Some local models emit standalone bracketed tool requests as text, such as
`[tool_name]` followed by JSON and `[END_TOOL_REQUEST]`. OpenClaw promotes
those into real tool calls only when the name exactly matches a registered
tool for the turn; otherwise the block is treated as unsupported text and is
hidden from user-visible replies.
- If a model emits JSON, XML, or ReAct-style text that looks like a tool call
but the provider did not emit a structured invocation, OpenClaw leaves it as
text and logs a warning with the run id, provider/model, detected pattern, and
tool name when available. Treat that as provider/model tool-call
incompatibility, not a completed tool run.
- If tools appear as assistant text instead of running, for example raw JSON,
XML, ReAct syntax, or an empty `tool_calls` array in the provider response,
first verify the server is using a tool-call-capable chat template/parser. For
OpenAI-compatible Chat Completions backends whose parser works only when tool
use is forced, set a per-model request override instead of relying on text
parsing:














```
{
    agents: {
      defaults: {
        models: {
          \"local/my-local-model\": {
            params: {
              extra_body: {
                tool_choice: \"required\",
              },
            },
          },
        },
      },
    },
}
```














```
openclaw config set agents.defaults.models \'{\"local/my-local-model\":{\"params\":{\"extra_body\":{\"tool_choice\":\"required\"}}}}\' --strict-json --merge
```

- Some smaller or stricter local backends are unstable with OpenClaw’s full
agent-runtime prompt shape, especially when tool schemas are included. First
verify the provider path with the lean local probe:














```
openclaw infer model run --local --model <provider/model> --prompt \"Reply with exactly: pong\" --json
```










If that succeeds but normal OpenClaw agent turns fail, first try
`agents.defaults.experimental.localModelLean: true` to drop heavyweight
default tools like `browser`, `cron`, and `message`; this is an experimental
flag, not a stable default-mode setting. See
[Experimental Features](https://docs.openclaw.ai/concepts/experimental-features). If that still fails, try
`models.providers.<provider>.models[].compat.supportsTools: false`.
- If the backend still fails only on larger OpenClaw runs, the remaining issue
is usually upstream model/server capacity or a backend bug, not OpenClaw’s
transport layer.

## [​](https://docs.openclaw.ai/gateway/local-models\\#troubleshooting)  Troubleshooting

- Gateway can reach the proxy? `curl http://127.0.0.1:1234/v1/models`.
- LM Studio model unloaded? Reload; cold start is a common “hanging” cause.
- Local server says `terminated`, `ECONNRESET`, or closes the stream mid-turn?
OpenClaw records a low-cardinality `model.call.error.failureKind` plus the
OpenClaw process RSS/heap snapshot in diagnostics. For LM Studio/Ollama
memory pressure, match that timestamp against the server log or macOS crash /
jetsam log to confirm whether the model server was killed.
- OpenClaw warns when the detected context window is below **32k** and blocks below **16k**. If you hit that preflight, raise the server/model context limit or choose a larger model.
- Context errors? Lower `contextWindow` or raise your server limit.
- OpenAI-compatible server returns `messages[].content ... expected a string`?
Add `compat.requiresStringContent: true` on that model entry.
- Direct tiny `/v1/chat/completions` calls work, but `openclaw infer model run --local`
fails on Gemma or another local model? Check the provider URL, model ref, auth
marker, and server logs first; local `model run` does not include agent tools.
If local `model run` succeeds but larger agent turns fail, reduce the agent
tool surface with `localModelLean` or `compat.supportsTools: false`.
- Tool calls show up as raw JSON/XML/ReAct text, or the provider returns an
empty `tool_calls` array? Do not add a proxy that blindly converts assistant
text into tool execution. Fix the server chat template/parser first. If the
model only works when tool use is forced, add the per-model
`params.extra_body.tool_choice: \"required\"` override above and use that model
entry only for sessions where a tool call is expected on every turn.
- Safety: local models skip provider-side filters; keep agents narrow and compaction on to limit prompt injection blast radius.

## [​](https://docs.openclaw.ai/gateway/local-models\\#related)  Related

- [Configuration reference](https://docs.openclaw.ai/gateway/configuration-reference)
- [Model failover](https://docs.openclaw.ai/concepts/model-failover)

[CLI backends](https://docs.openclaw.ai/gateway/cli-backends) [Network](https://docs.openclaw.ai/network)

Ctrl+I

---

## Configuration reference
**Source:** https://docs.openclaw.ai/gateway/configuration-reference

[Skip to main content](https://docs.openclaw.ai/gateway/configuration-reference#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nConfiguration\n\nConfiguration reference\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Channels](https://docs.openclaw.ai/gateway/configuration-reference#channels)\n- [Agent defaults, multi-agent, sessions, and messages](https://docs.openclaw.ai/gateway/configuration-reference#agent-defaults-multi-agent-sessions-and-messages)\n- [Tools and custom providers](https://docs.openclaw.ai/gateway/configuration-reference#tools-and-custom-providers)\n- [Models](https://docs.openclaw.ai/gateway/configuration-reference#models)\n- [MCP](https://docs.openclaw.ai/gateway/configuration-reference#mcp)\n- [Skills](https://docs.openclaw.ai/gateway/configuration-reference#skills)\n- [Plugins](https://docs.openclaw.ai/gateway/configuration-reference#plugins)\n- [Browser](https://docs.openclaw.ai/gateway/configuration-reference#browser)\n- [UI](https://docs.openclaw.ai/gateway/configuration-reference#ui)\n- [Gateway](https://docs.openclaw.ai/gateway/configuration-reference#gateway)\n- [OpenAI-compatible endpoints](https://docs.openclaw.ai/gateway/configuration-reference#openai-compatible-endpoints)\n- [Multi-instance isolation](https://docs.openclaw.ai/gateway/configuration-reference#multi-instance-isolation)\n- [gateway.tls](https://docs.openclaw.ai/gateway/configuration-reference#gateway-tls)\n- [gateway.reload](https://docs.openclaw.ai/gateway/configuration-reference#gateway-reload)\n- [Hooks](https://docs.openclaw.ai/gateway/configuration-reference#hooks)\n- [Gmail integration](https://docs.openclaw.ai/gateway/configuration-reference#gmail-integration)\n- [Canvas host](https://docs.openclaw.ai/gateway/configuration-reference#canvas-host)\n- [Discovery](https://docs.openclaw.ai/gateway/configuration-reference#discovery)\n- [mDNS (Bonjour)](https://docs.openclaw.ai/gateway/configuration-reference#mdns-bonjour)\n- [Wide-area (DNS-SD)](https://docs.openclaw.ai/gateway/configuration-reference#wide-area-dns-sd)\n- [Environment](https://docs.openclaw.ai/gateway/configuration-reference#environment)\n- [env (inline env vars)](https://docs.openclaw.ai/gateway/configuration-reference#env-inline-env-vars)\n- [Env var substitution](https://docs.openclaw.ai/gateway/configuration-reference#env-var-substitution)\n- [Secrets](https://docs.openclaw.ai/gateway/configuration-reference#secrets)\n- [SecretRef](https://docs.openclaw.ai/gateway/configuration-reference#secretref)\n- [Supported credential surface](https://docs.openclaw.ai/gateway/configuration-reference#supported-credential-surface)\n- [Secret providers config](https://docs.openclaw.ai/gateway/configuration-reference#secret-providers-config)\n- [Auth storage](https://docs.openclaw.ai/gateway/configuration-reference#auth-storage)\n- [auth.cooldowns](https://docs.openclaw.ai/gateway/configuration-reference#auth-cooldowns)\n- [Logging](https://docs.openclaw.ai/gateway/configuration-reference#logging)\n- [Diagnostics](https://docs.openclaw.ai/gateway/configuration-reference#diagnostics)\n- [Update](https://docs.openclaw.ai/gateway/configuration-reference#update)\n- [ACP](https://docs.openclaw.ai/gateway/configuration-reference#acp)\n- [CLI](https://docs.openclaw.ai/gateway/configuration-reference#cli)\n- [Wizard](https://docs.openclaw.ai/gateway/configuration-reference#wizard)\n- [Identity](https://docs.openclaw.ai/gateway/configuration-reference#identity)\n- [Bridge (legacy, removed)](https://docs.openclaw.ai/gateway/configuration-reference#bridge-legacy-removed)\n- [Cron](https://docs.openclaw.ai/gateway/configuration-reference#cron)\n- [cron.retry](https://docs.openclaw.ai/gateway/configuration-reference#cron-retry)\n- [cron.failureAlert](https://docs.openclaw.ai/gateway/configuration-reference#cron-failurealert)\n- [cron.failureDestination](https://docs.openclaw.ai/gateway/configuration-reference#cron-failuredestination)\n- [Media model template variables](https://docs.openclaw.ai/gateway/configuration-reference#media-model-template-variables)\n- [Config includes ($include)](https://docs.openclaw.ai/gateway/configuration-reference#config-includes-%24include)\n- [Related](https://docs.openclaw.ai/gateway/configuration-reference#related)\n\nCore config reference for `~/.openclaw/openclaw.json`. For a task-oriented overview, see [Configuration](https://docs.openclaw.ai/gateway/configuration).Covers the main OpenClaw config surfaces and links out when a subsystem has its own deeper reference. Channel- and plugin-owned command catalogs and deep memory/QMD knobs live on their own pages rather than on this one.Code truth:\n\n- `openclaw config schema` prints the live JSON Schema used for validation and Control UI, with bundled/plugin/channel metadata merged in when available\n- `config.schema.lookup` returns one path-scoped schema node for drill-down tooling\n- `pnpm config:docs:check` / `pnpm config:docs:gen` validate the config-doc baseline hash against the current schema surface\n\nAgent lookup path: use the `gateway` tool action `config.schema.lookup` for\nexact field-level docs and constraints before edits. Use\n[Configuration](https://docs.openclaw.ai/gateway/configuration) for task-oriented guidance and this page\nfor the broader field map, defaults, and links to subsystem references.Dedicated deep references:\n\n- [Memory configuration reference](https://docs.openclaw.ai/reference/memory-config) for `agents.defaults.memorySearch.*`, `memory.qmd.*`, `memory.citations`, and dreaming config under `plugins.entries.memory-core.config.dreaming`\n- [Slash commands](https://docs.openclaw.ai/tools/slash-commands) for the current built-in + bundled command catalog\n- owning channel/plugin pages for channel-specific command surfaces\n\nConfig format is **JSON5** (comments + trailing commas allowed). All fields are optional — OpenClaw uses safe defaults when omitted.\n\n* * *\n\n## [​](https://docs.openclaw.ai/gateway/configuration-reference\\#channels)  Channels\n\nPer-channel config keys moved to a dedicated page — see\n[Configuration — channels](https://docs.openclaw.ai/gateway/config-channels) for `channels.*`,\nincluding Slack, Discord, Telegram, WhatsApp, Matrix, iMessage, and other\nbundled channels (auth, access control, multi-account, mention gating).\n\n## [​](https://docs.openclaw.ai/gateway/configuration-reference\\#agent-defaults-multi-agent-sessions-and-messages)  Agent defaults, multi-agent, sessions, and messages\n\nMoved to a dedicated page — see\n[Configuration — agents](https://docs.openclaw.ai/gateway/config-agents) for:\n\n- `agents.defaults.*` (workspace, model, thinking, heartbeat, memory, media, skills, sandbox)\n- `multiAgent.*` (multi-agent routing and bindings)\n- `session.*` (session lifecycle, compaction, pruning)\n- `messages.*` (message delivery, TTS, markdown rendering)\n- `talk.*`(Talk mode)\n\n  - `talk.speechLocale`: optional BCP 47 locale id for Talk speech recognition on iOS/macOS\n  - `talk.silenceTimeoutMs`: when unset, Talk keeps the platform default pause window before sending the transcript (`700 ms on macOS and Android, 900 ms on iOS`)\n\n## [​](https://docs.openclaw.ai/gateway/configuration-reference\\#tools-and-custom-providers)  Tools and custom providers\n\nTool policy, experimental toggles, provider-backed tool config, and custom\nprovider / base-URL setup moved to a dedicated page — see\n[Configuration — tools and custom providers](https://docs.openclaw.ai/gateway/config-tools).\n\n## [​](https://docs.openclaw.ai/gateway/configuration-reference\\#models)  Models\n\nProvider definitions, model allowlists, and custom provider setup live in\n[Configuration — tools and custom providers](https://docs.openclaw.ai/gateway/config-tools#custom-providers-and-base-urls).\nThe `models` root also owns global model-catalog behavior.\n\n```\n{\n  models: {\n    // Optional. Default: true. Requires a Gateway restart when changed.\n    pricing: { enabled: false },\n  },\n}\n```\n\n- `models.mode`: provider catalog behavior (`merge` or `replace`).\n- `models.providers`: custom provider map keyed by provider id.\n- `models.pricing.enabled`: controls the background pricing bootstrap. When\n`false`, Gateway startup skips OpenRouter and LiteLLM pricing-catalog fetches;\nconfigured `models.providers.*.models[].cost` values still work for local cost\nestimates.\n\n## [​](https://docs.openclaw.ai/gateway/configuration-reference\\#mcp)  MCP\n\nOpenClaw-managed MCP server definitions live under `mcp.servers` and are\nconsumed by embedded Pi and other runtime adapters. The `openclaw mcp list`,\n`show`, `set`, and `unset` commands manage this block without connecting to the\ntarget server during config edits.\n\n```\n{\n  mcp: {\n    // Optional. Default: 600000 ms (10 minutes). Set 0 to disable idle eviction.\n    sessionIdleTtlMs: 600000,\n    servers: {\n      docs: {\n        command: \"npx\",\n        args: [\"-y\", \"@modelcontextprotocol/server-fetch\"],\n      },\n      remote: {\n        url: \"https://example.com/mcp\",\n        transport: \"streamable-http\", // streamable-http | sse\n        headers: {\n          Authorization: \"Bearer ${MCP_REMOTE_TOKEN}\",\n        },\n      },\n    },\n  },\n}\n```\n\n- `mcp.servers`: named stdio or remote MCP server definitions for runtimes that\nexpose configured MCP tools.\nRemote entries use `transport: \"streamable-http\"` or `transport: \"sse\"`;\n`type: \"http\"` is a CLI-native alias that `openclaw mcp set` and\n`openclaw doctor --fix` normalize into the canonical `transport` field.\n- `mcp.sessionIdleTtlMs`: idle TTL for session-scoped bundled MCP runtimes.\nOne-shot embedded runs request run-end cleanup; this TTL is the backstop for\nlong-lived sessions and future callers.\n- Changes under `mcp.*` hot-apply by disposing cached session MCP runtimes.\nThe next tool discovery/use recreates them from the new config, so removed\n`mcp.servers` entries are reaped immediately instead of waiting for idle TTL.\n\nSee [MCP](https://docs.openclaw.ai/cli/mcp#openclaw-as-an-mcp-client-registry) and\n[CLI backends](https://docs.openclaw.ai/gateway/cli-backends#bundle-mcp-overlays) for runtime behavior.\n\n## [​](https://docs.openclaw.ai/gateway/configuration-reference\\#skills)  Skills\n\n```\n{\n  skills: {\n    allowBundled: [\"gemini\", \"peekaboo\"],\n    load: {\n      extraDirs: [\"~/Projects/agent-scripts/skills\"],\n    },\n    install: {\n      preferBrew: true,\n      nodeManager: \"npm\", // npm | pnpm | yarn | bun\n    },\n    entries: {\n      \"image-lab\": {\n        apiKey: { source: \"env\", provider: \"default\", id: \"GEMINI_API_KEY\" }, // or plaintext string\n        env: { GEMINI_API_KEY: \"GEMINI_KEY_HERE\" },\n      },\n      peekaboo: { enabled: true },\n      sag: { enabled: false },\n    },\n  },\n}\n```\n\n- `allowBundled`: optional allowlist for bundled skills only (managed/workspace skills unaffected).\n- `load.extraDirs`: extra shared skill roots (lowest precedence).\n- `install.preferBrew`: when true, prefer Homebrew installers when `brew` is\navailable before falling back to other installer kinds.\n- `install.nodeManager`: node installer preference for `metadata.openclaw.install`\nspecs (`npm` \\| `pnpm` \\| `yarn` \\| `bun`).\n- `entries.<skillKey>.enabled: false` disables a skill even if bundled/installed.\n- `entries.<skillKey>.apiKey`: convenience for skills declaring a primary env var (plaintext string or SecretRef object).\n\n* * *\n\n## [​](https://docs.openclaw.ai/gateway/configuration-reference\\#plugins)  Plugins\n\n```\n{\n  plugins: {\n    enabled: true,\n    allow: [\"voice-call\"],\n    deny: [],\n    load: {\n      paths: [\"~/Projects/oss/voice-call-plugin\"],\n    },\n    entries: {\n      \"voice-call\": {\n        enabled: true,\n        hooks: {\n          allowPromptInjection: false,\n        },\n        config: { provider: \"twilio\" },\n      },\n    },\n  },\n}\n```\n\n- Loaded from `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, plus `plugins.load.paths`.\n- Discovery accepts native OpenClaw plugins plus compatible Codex bundles and Claude bundles, including manifestless Claude default-layout bundles.\n- **Config changes require a gateway restart.**\n- `allow`: optional allowlist (only listed plugins load). `deny` wins.\n- `plugins.entries.<id>.apiKey`: plugin-level API key convenience field (when supported by the plugin).\n- `plugins.entries.<id>.env`: plugin-scoped env var map.\n- `plugins.entries.<id>.hooks.allowPromptInjection`: when `false`, core blocks `before_prompt_build` and ignores prompt-mutating fields from legacy `before_agent_start`, while preserving legacy `modelOverride` and `providerOverride`. Applies to native plugin hooks and supported bundle-provided hook directories.\n- `plugins.entries.<id>.hooks.allowConversationAccess`: when `true`, trusted non-bundled plugins may read raw conversation content from typed hooks such as `llm_input`, `llm_output`, `before_agent_finalize`, and `agent_end`.\n- `plugins.entries.<id>.subagent.allowModelOverride`: explicitly trust this plugin to request per-run `provider` and `model` overrides for background subagent runs.\n- `plugins.entries.<id>.subagent.allowedModels`: optional allowlist of canonical `provider/model` targets for trusted subagent overrides. Use `\"*\"` only when you intentionally want to allow any model.\n- `plugins.entries.<id>.config`: plugin-defined config object (validated by native OpenClaw plugin schema when available).\n- Channel plugin account/runtime settings live under `channels.<id>` and should be described by the owning plugin’s manifest `channelConfigs` metadata, not by a central OpenClaw option registry.\n- `plugins.entries.firecrawl.config.webFetch`: Firecrawl web-fetch provider settings.\n\n  - `apiKey`: Firecrawl API key (accepts SecretRef). Falls back to `plugins.entries.firecrawl.config.webSearch.apiKey`, legacy `tools.web.fetch.firecrawl.apiKey`, or `FIRECRAWL_API_KEY` env var.\n  - `baseUrl`: Firecrawl API base URL (default: `https://api.firecrawl.dev`).\n  - `onlyMainContent`: extract only the main content from pages (default: `true`).\n  - `maxAgeMs`: maximum cache age in milliseconds (default: `172800000` / 2 days).\n  - `timeoutSeconds`: scrape request timeout in seconds (default: `60`).\n- `plugins.entries.xai.config.xSearch`: xAI X Search (Grok web search) settings.\n\n  - `enabled`: enable the X Search provider.\n  - `model`: Grok model to use for search (e.g. `\"grok-4-1-fast\"`).\n- `plugins.entries.memory-core.config.dreaming`: memory dreaming settings. See [Dreaming](https://docs.openclaw.ai/concepts/dreaming) for phases and thresholds.\n\n  - `enabled`: master dreaming switch (default `false`).\n  - `frequency`: cron cadence for each full dreaming sweep (`\"0 3 * * *\"` by default).\n  - `model`: optional Dream Diary subagent model override. Requires `plugins.entries.memory-core.subagent.allowModelOverride: true`; pair with `allowedModels` to restrict targets. Model-unavailable errors retry once with the session default model; trust or allowlist failures do not fall back silently.\n  - phase policy and thresholds are implementation details (not user-facing config keys).\n- Full memory config lives in [Memory configuration reference](https://docs.openclaw.ai/reference/memory-config):\n\n  - `agents.defaults.memorySearch.*`\n  - `memory.backend`\n  - `memory.citations`\n  - `memory.qmd.*`\n  - `plugins.entries.memory-core.config.dreaming`\n- Enabled Claude bundle plugins can also contribute embedded Pi defaults from `settings.json`; OpenClaw applies those as sanitized agent settings, not as raw OpenClaw config patches.\n- `plugins.slots.memory`: pick the active memory plugin id, or `\"none\"` to disable memory plugins.\n- `plugins.slots.contextEngine`: pick the active context engine plugin id; defaults to `\"legacy\"` unless you install and select another engine.\n\nSee [Plugins](https://docs.openclaw.ai/tools/plugin).\n\n* * *\n\n## [​](https://docs.openclaw.ai/gateway/configuration-reference\\#browser)  Browser\n\n```\n{\n  browser: {\n    enabled: true,\n    evaluateEnabled: true,\n    defaultProfile: \"user\",\n    ssrfPolicy: {\n      // dangerouslyAllowPrivateNetwork: true, // opt in only for trusted private-network access\n      // allowPrivateNetwork: true, // legacy alias\n      // hostnameAllowlist: [\"*.example.com\", \"example.com\"],\n      // allowedHostnames: [\"localhost\"],\n    },\n    tabCleanup: {\n      enabled: true,\n      idleMinutes: 120,\n      maxTabsPerSession: 8,\n      sweepMinutes: 5,\n    },\n    profiles: {\n      openclaw: { cdpPort: 18800, color: \"#FF4500\" },\n      work: {\n        cdpPort: 18801,\n        color: \"#0066CC\",\n        executablePath: \"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome\",\n      },\n      user: { driver: \"existing-session\", attachOnly: true, color: \"#00AA00\" },\n      brave: {\n        driver: \"existing-session\",\n        attachOnly: true,\n        userDataDir: \"~/Library/Application Support/BraveSoftware/Brave-Browser\",\n        color: \"#FB542B\",\n      },\n      remote: { cdpUrl: \"http://10.0.0.42:9222\", color: \"#00AA00\" },\n    },\n    color: \"#FF4500\",\n    // headless: false,\n    // noSandbox: false,\n    // extraArgs: [],\n    // executablePath: \"/Applications/Brave Browser.app/Contents/MacOS/Brave Browser\",\n    // attachOnly: false,\n  },\n}\n```\n\n- `evaluateEnabled: false` disables `act:evaluate` and `wait --fn`.\n- `tabCleanup` reclaims tracked primary-agent tabs after idle time or when a\nsession exceeds its cap. Set `idleMinutes: 0` or `maxTabsPerSession: 0` to\ndisable those individual cleanup modes.\n- `ssrfPolicy.dangerouslyAllowPrivateNetwork` is disabled when unset, so browser navigation stays strict by default.\n- Set `ssrfPolicy.dangerouslyAllowPrivateNetwork: true` only when you intentionally trust private-network browser navigation.\n- In strict mode, remote CDP profile endpoints (`profiles.*.cdpUrl`) are subject to the same private-network blocking during reachability/discovery checks.\n- `ssrfPolicy.allowPrivateNetwork` remains supported as a legacy alias.\n- In strict mode, use `ssrfPolicy.hostnameAllowlist` and `ssrfPolicy.allowedHostnames` for explicit exceptions.\n- Remote profiles are attach-only (start/stop/reset disabled).\n- `profiles.*.cdpUrl` accepts `http://`, `https://`, `ws://`, and `wss://`.\nUse HTTP(S) when you want OpenClaw to discover `/json/version`; use WS(S)\nwhen your provider gives you a direct DevTools WebSocket URL.\n- `remoteCdpTimeoutMs` and `remoteCdpHandshakeTimeoutMs` apply to remote and\n`attachOnly` CDP reachability plus tab-opening request...(content truncated)

---

## Configuration — tools and custom providers
**Source:** https://docs.openclaw.ai/gateway/config-tools

[Skip to main content](https://docs.openclaw.ai/gateway/config-tools#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Configuration

Configuration — tools and custom providers

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Tools](https://docs.openclaw.ai/gateway/config-tools#tools)
- [Tool profiles](https://docs.openclaw.ai/gateway/config-tools#tool-profiles)
- [Tool groups](https://docs.openclaw.ai/gateway/config-tools#tool-groups)
- [tools.allow / tools.deny](https://docs.openclaw.ai/gateway/config-tools#tools-allow-%2F-tools-deny)
- [tools.byProvider](https://docs.openclaw.ai/gateway/config-tools#tools-byprovider)
- [tools.elevated](https://docs.openclaw.ai/gateway/config-tools#tools-elevated)
- [tools.exec](https://docs.openclaw.ai/gateway/config-tools#tools-exec)
- [tools.loopDetection](https://docs.openclaw.ai/gateway/config-tools#tools-loopdetection)
- [tools.web](https://docs.openclaw.ai/gateway/config-tools#tools-web)
- [tools.media](https://docs.openclaw.ai/gateway/config-tools#tools-media)
- [tools.agentToAgent](https://docs.openclaw.ai/gateway/config-tools#tools-agenttoagent)
- [tools.sessions](https://docs.openclaw.ai/gateway/config-tools#tools-sessions)
- [tools.sessions\\_spawn](https://docs.openclaw.ai/gateway/config-tools#tools-sessions_spawn)
- [tools.experimental](https://docs.openclaw.ai/gateway/config-tools#tools-experimental)
- [agents.defaults.subagents](https://docs.openclaw.ai/gateway/config-tools#agents-defaults-subagents)
- [Custom providers and base URLs](https://docs.openclaw.ai/gateway/config-tools#custom-providers-and-base-urls)
- [Provider field details](https://docs.openclaw.ai/gateway/config-tools#provider-field-details)
- [Provider examples](https://docs.openclaw.ai/gateway/config-tools#provider-examples)
- [Related](https://docs.openclaw.ai/gateway/config-tools#related)

`tools.*` config keys and custom provider / base-URL setup. For agents, channels, and other top-level config keys, see [Configuration reference](https://docs.openclaw.ai/gateway/configuration-reference).

## [​](https://docs.openclaw.ai/gateway/config-tools\\#tools)  Tools

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tool-profiles)  Tool profiles

`tools.profile` sets a base allowlist before `tools.allow`/`tools.deny`:

Local onboarding defaults new local configs to `tools.profile: \"coding\"` when unset (existing explicit profiles are preserved).

| Profile | Includes |
| --- | --- |
| `minimal` | `session_status` only |
| `coding` | `group:fs`, `group:runtime`, `group:web`, `group:sessions`, `group:memory`, `cron`, `image`, `image_generate`, `video_generate` |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full` | No restriction (same as unset) |

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tool-groups)  Tool groups

| Group | Tools |
| --- | --- |
| `group:runtime` | `exec`, `process`, `code_execution` (`bash` is accepted as an alias for `exec`) |
| `group:fs` | `read`, `write`, `edit`, `apply_patch` |
| `group:sessions` | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `sessions_yield`, `subagents`, `session_status` |
| `group:memory` | `memory_search`, `memory_get` |
| `group:web` | `web_search`, `x_search`, `web_fetch` |
| `group:ui` | `browser`, `canvas` |
| `group:automation` | `cron`, `gateway` |
| `group:messaging` | `message` |
| `group:nodes` | `nodes` |
| `group:agents` | `agents_list` |
| `group:media` | `image`, `image_generate`, `video_generate`, `tts` |
| `group:openclaw` | All built-in tools (excludes provider plugins) |

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-allow-/-tools-deny)  `tools.allow` / `tools.deny`

Global tool allow/deny policy (deny wins). Case-insensitive, supports `*` wildcards. Applied even when Docker sandbox is off.

```
{
  tools: { deny: [\"browser\", \"canvas\"] },
}
```

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-byprovider)  `tools.byProvider`

Further restrict tools for specific providers or models. Order: base profile → provider profile → allow/deny.

```
{
  tools: {
    profile: \"coding\",
    byProvider: {
      \"google-antigravity\": { profile: \"minimal\" },
      \"openai/gpt-5.4\": { allow: [\"group:fs\", \"sessions_list\"] },
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-elevated)  `tools.elevated`

Controls elevated exec access outside the sandbox:

```
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: [\"+15555550123\"],
        discord: [\"1234567890123\", \"987654321098765432\"],
      },
    },
  },
}
```

- Per-agent override (`agents.list[].tools.elevated`) can only further restrict.
- `/elevated on|off|ask|full` stores state per session; inline directives apply to single message.
- Elevated `exec` bypasses sandboxing and uses the configured escape path (`gateway` by default, or `node` when the exec target is `node`).

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-exec)  `tools.exec`

```
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: [\"gpt-5.5\"],
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-loopdetection)  `tools.loopDetection`

Tool-loop safety checks are **disabled by default**. Set `enabled: true` to activate detection. Settings can be defined globally in `tools.loopDetection` and overridden per-agent at `agents.list[].tools.loopDetection`.

```
{
  tools: {
    loopDetection: {
      enabled: true,
      historySize: 30,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

[​](https://docs.openclaw.ai/gateway/config-tools#param-history-size)

historySize

number

Max tool-call history retained for loop analysis.

[​](https://docs.openclaw.ai/gateway/config-tools#param-warning-threshold)

warningThreshold

number

Repeating no-progress pattern threshold for warnings.

[​](https://docs.openclaw.ai/gateway/config-tools#param-critical-threshold)

criticalThreshold

number

Higher repeating threshold for blocking critical loops.

[​](https://docs.openclaw.ai/gateway/config-tools#param-global-circuit-breaker-threshold)

globalCircuitBreakerThreshold

number

Hard stop threshold for any no-progress run.

[​](https://docs.openclaw.ai/gateway/config-tools#param-detectors-generic-repeat)

detectors.genericRepeat

boolean

Warn on repeated same-tool/same-args calls.

[​](https://docs.openclaw.ai/gateway/config-tools#param-detectors-known-poll-no-progress)

detectors.knownPollNoProgress

boolean

Warn/block on known poll tools (`process.poll`, `command_status`, etc.).

[​](https://docs.openclaw.ai/gateway/config-tools#param-detectors-ping-pong)

detectors.pingPong

boolean

Warn/block on alternating no-progress pair patterns.

If `warningThreshold >= criticalThreshold` or `criticalThreshold >= globalCircuitBreakerThreshold`, validation fails.

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-web)  `tools.web`

```
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: \"brave_api_key\", // or BRAVE_API_KEY env
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        provider: \"firecrawl\", // optional; omit for auto-detect
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        readability: true,
        userAgent: \"custom-ua\",
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-media)  `tools.media`

Configures inbound media understanding (image/audio/video):

```
{
  tools: {
    media: {
      concurrency: 2,
      asyncCompletion: {
        directSend: false, // opt-in: send finished async music/video directly to the channel
      },
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: \"deny\",
          rules: [{ action: \"allow\", match: { chatType: \"direct\" } }],
        },
        models: [\
          { provider: \"openai\", model: \"gpt-4o-mini-transcribe\" },\
          { type: \"cli\", command: \"whisper\", args: [\"--model\", \"base\", \"{{MediaPath}}\"] },\
        ],
      },
      image: {
        enabled: true,
        timeoutSeconds: 180,
        models: [{ provider: \"ollama\", model: \"gemma4:26b\", timeoutSeconds: 300 }],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: \"google\", model: \"gemini-3-flash-preview\" }],
      },
    },
  },
}
```

Media model entry fields

**Provider entry** (`type: \"provider\"` or omitted):

- `provider`: API provider id (`openai`, `anthropic`, `google`/`gemini`, `groq`, etc.)
- `model`: model id override
- `profile` / `preferredProfile`: `auth-profiles.json` profile selection

**CLI entry** (`type: \"cli\"`):

- `command`: executable to run
- `args`: templated args (supports `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, etc.; `openclaw doctor --fix` migrates deprecated `{input}` placeholders to `{{MediaPath}}`)

**Common fields:**

- `capabilities`: optional list (`image`, `audio`, `video`). Defaults: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: per-entry overrides.
- `tools.media.image.timeoutSeconds` and matching image model `timeoutSeconds` entries also apply when the agent calls the explicit `image` tool.
- Failures fall back to the next entry.

Provider auth follows standard order: `auth-profiles.json` → env vars → `models.providers.*.apiKey`.**Async completion fields:**

- `asyncCompletion.directSend`: when `true`, completed async `music_generate` and `video_generate` tasks try direct channel delivery first. Default: `false` (legacy requester-session wake/model-delivery path).

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-agenttoagent)  `tools.agentToAgent`

```
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: [\"home\", \"work\"],
    },
  },
}
```

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-sessions)  `tools.sessions`

Controls which sessions can be targeted by the session tools (`sessions_list`, `sessions_history`, `sessions_send`).Default: `tree` (current session + sessions spawned by it, such as subagents).

```
{
  tools: {
    sessions: {
      // \"self\" | \"tree\" | \"agent\" | \"all\"
      visibility: \"tree\",
    },
  },
}
```

Visibility scopes

- `self`: only the current session key.
- `tree`: current session + sessions spawned by the current session (subagents).
- `agent`: any session belonging to the current agent id (can include other users if you run per-sender sessions under the same agent id).
- `all`: any session. Cross-agent targeting still requires `tools.agentToAgent`.
- Sandbox clamp: when the current session is sandboxed and `agents.defaults.sandbox.sessionToolsVisibility=\"spawned\"`, visibility is forced to `tree` even if `tools.sessions.visibility=\"all\"`.

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-sessions_spawn)  `tools.sessions_spawn`

Controls inline attachment support for `sessions_spawn`.

```
{
  tools: {
    sessions_spawn: {
      attachments: {
        enabled: false, // opt-in: set true to allow inline file attachments
        maxTotalBytes: 5242880, // 5 MB total across all files
        maxFiles: 50,
        maxFileBytes: 1048576, // 1 MB per file
        retainOnSessionKeep: false, // keep attachments when cleanup=\"keep\"
      },
    },
  },
}
```

Attachment notes

- Attachments are only supported for `runtime: \"subagent\"`. ACP runtime rejects them.
- Files are materialized into the child workspace at `.openclaw/attachments/<uuid>/` with a `.manifest.json`.
- Attachment content is automatically redacted from transcript persistence.
- Base64 inputs are validated with strict alphabet/padding checks and a pre-decode size guard.
- File permissions are `0700` for directories and `0600` for files.
- Cleanup follows the `cleanup` policy: `delete` always removes attachments; `keep` retains them only when `retainOnSessionKeep: true`.

### [​](https://docs.openclaw.ai/gateway/config-tools\\#tools-experimental)  `tools.experimental`

Experimental built-in tool flags. Default off unless a strict-agentic GPT-5 auto-enable rule applies.

```
{
  tools: {
    experimental: {
      planTool: true, // enable experimental update_plan
    },
  },
}
```

- `planTool`: enables the structured `update_plan` tool for non-trivial multi-step work tracking.
- Default: `false` unless `agents.defaults.embeddedPi.executionContract` (or a per-agent override) is set to `\"strict-agentic\"` for an OpenAI or OpenAI Codex GPT-5-family run. Set `true` to force the tool on outside that scope, or `false` to keep it off even for strict-agentic GPT-5 runs.
- When enabled, the system prompt also adds usage guidance so the model only uses it for substantial work and keeps at most one step `in_progress`.

### [​](https://docs.openclaw.ai/gateway/config-tools\\#agents-defaults-subagents)  `agents.defaults.subagents`

```
{
  agents: {
    defaults: {
      subagents: {
        allowAgents: [\"research\"],
        model: \"minimax/MiniMax-M2.7\",
        maxConcurrent: 8,
        runTimeoutSeconds: 900,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: default model for spawned sub-agents. If omitted, sub-agents inherit the caller’s model.
- `allowAgents`: default allowlist of target agent ids for `sessions_spawn` when the requester agent does not set its own `subagents.allowAgents` (`[\"*\"]` = any; default: same agent only).
- `runTimeoutSeconds`: default timeout (seconds) for `sessions_spawn` when the tool call omits `runTimeoutSeconds`. `0` means no timeout.
- Per-subagent tool policy: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

* * *

## [​](https://docs.openclaw.ai/gateway/config-tools\\#custom-providers-and-base-urls)  Custom providers and base URLs

OpenClaw uses the built-in model catalog. Add custom providers via `models.providers` in config or `~/.openclaw/agents/<agentId>/agent/models.json`.

```
{
  models: {
    mode: \"merge\", // merge (default) | replace
    providers: {
      \"custom-proxy\": {
        baseUrl: \"http://localhost:4000/v1\",
        apiKey: \"LITELLM_KEY\",
        api: \"openai-completions\", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [\
          {\
            id: \"llama-3.1-8b\",\
            name: \"Llama 3.1 8B\",\
            reasoning: false,\"

---

## OpenResponses API - OpenClaw
**Source:** https://docs.openclaw.ai/gateway/openresponses-http-api

[Skip to main content](https://docs.openclaw.ai/gateway/openresponses-http-api#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Protocols and APIs

OpenResponses API

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Authentication, security, and routing](https://docs.openclaw.ai/gateway/openresponses-http-api#authentication-security-and-routing)
- [Session behavior](https://docs.openclaw.ai/gateway/openresponses-http-api#session-behavior)
- [Request shape (supported)](https://docs.openclaw.ai/gateway/openresponses-http-api#request-shape-supported)
- [Items (input)](https://docs.openclaw.ai/gateway/openresponses-http-api#items-input)
- [message](https://docs.openclaw.ai/gateway/openresponses-http-api#message)
- [function\\_call\\_output (turn-based tools)](https://docs.openclaw.ai/gateway/openresponses-http-api#function_call_output-turn-based-tools)
- [reasoning and item\\_reference](https://docs.openclaw.ai/gateway/openresponses-http-api#reasoning-and-item_reference)
- [Tools (client-side function tools)](https://docs.openclaw.ai/gateway/openresponses-http-api#tools-client-side-function-tools)
- [Images (input\\_image)](https://docs.openclaw.ai/gateway/openresponses-http-api#images-input_image)
- [Files (input\\_file)](https://docs.openclaw.ai/gateway/openresponses-http-api#files-input_file)
- [File + image limits (config)](https://docs.openclaw.ai/gateway/openresponses-http-api#file-%2B-image-limits-config)
- [Streaming (SSE)](https://docs.openclaw.ai/gateway/openresponses-http-api#streaming-sse)
- [Usage](https://docs.openclaw.ai/gateway/openresponses-http-api#usage)
- [Errors](https://docs.openclaw.ai/gateway/openresponses-http-api#errors)
- [Examples](https://docs.openclaw.ai/gateway/openresponses-http-api#examples)
- [Related](https://docs.openclaw.ai/gateway/openresponses-http-api#related)

OpenClaw’s Gateway can serve an OpenResponses-compatible `POST /v1/responses` endpoint.This endpoint is **disabled by default**. Enable it in config first.

- `POST /v1/responses`
- Same port as the Gateway (WS + HTTP multiplex): `http://<gateway-host>:<port>/v1/responses`

Under the hood, requests are executed as a normal Gateway agent run (same codepath as
`openclaw agent`), so routing/permissions/config match your Gateway.

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#authentication-security-and-routing)  Authentication, security, and routing

Operational behavior matches [OpenAI Chat Completions](https://docs.openclaw.ai/gateway/openai-http-api):

- use the matching Gateway HTTP auth path:
  - shared-secret auth (`gateway.auth.mode=\"token\"` or `\"password\"`): `Authorization: Bearer <token-or-password>`
  - trusted-proxy auth (`gateway.auth.mode=\"trusted-proxy\"`): identity-aware proxy headers from a configured trusted proxy source; same-host loopback proxies require explicit `gateway.auth.trustedProxy.allowLoopback = true`
  - private-ingress open auth (`gateway.auth.mode=\"none\"`): no auth header
- treat the endpoint as full operator access for the gateway instance
- for shared-secret auth modes (`token` and `password`), ignore narrower bearer-declared `x-openclaw-scopes` values and restore the normal full operator defaults
- for trusted identity-bearing HTTP modes (for example trusted proxy auth or `gateway.auth.mode=\"none\"`), honor `x-openclaw-scopes` when present and otherwise fall back to the normal operator default scope set
- select agents with `model: \"openclaw\"`, `model: \"openclaw/default\"`, `model: \"openclaw/<agentId>\"`, or `x-openclaw-agent-id`
- use `x-openclaw-model` when you want to override the selected agent’s backend model
- use `x-openclaw-session-key` for explicit session routing
- use `x-openclaw-message-channel` when you want a non-default synthetic ingress channel context

Auth matrix:

- `gateway.auth.mode=\"token\"` or `\"password\"` \\+ `Authorization: Bearer ...`
  - proves possession of the shared gateway operator secret
  - ignores narrower `x-openclaw-scopes`
  - restores the full default operator scope set:
    `operator.admin`, `operator.approvals`, `operator.pairing`,
    `operator.read`, `operator.talk.secrets`, `operator.write`
  - treats chat turns on this endpoint as owner-sender turns
- trusted identity-bearing HTTP modes (for example trusted proxy auth, or `gateway.auth.mode=\"none\"` on private ingress)

  - honor `x-openclaw-scopes` when the header is present
  - fall back to the normal operator default scope set when the header is absent
  - only lose owner semantics when the caller explicitly narrows scopes and omits `operator.admin`

Enable or disable this endpoint with `gateway.http.endpoints.responses.enabled`.The same compatibility surface also includes:

- `GET /v1/models`
- `GET /v1/models/{id}`
- `POST /v1/embeddings`
- `POST /v1/chat/completions`

For the canonical explanation of how agent-target models, `openclaw/default`, embeddings pass-through, and backend model overrides fit together, see [OpenAI Chat Completions](https://docs.openclaw.ai/gateway/openai-http-api#agent-first-model-contract) and [Model list and agent routing](https://docs.openclaw.ai/gateway/openai-http-api#model-list-and-agent-routing).

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#session-behavior)  Session behavior

By default the endpoint is **stateless per request** (a new session key is generated each call).If the request includes an OpenResponses `user` string, the Gateway derives a stable session key
from it, so repeated calls can share an agent session.

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#request-shape-supported)  Request shape (supported)

The request follows the OpenResponses API with item-based input. Current support:

- `input`: string or array of item objects.
- `instructions`: merged into the system prompt.
- `tools`: client tool definitions (function tools).
- `tool_choice`: filter or require client tools.
- `stream`: enables SSE streaming.
- `max_output_tokens`: best-effort output limit (provider dependent).
- `user`: stable session routing.

Accepted but **currently ignored**:

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `truncation`

Supported:

- `previous_response_id`: OpenClaw reuses the earlier response session when the request stays within the same agent/user/requested-session scope.

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#items-input)  Items (input)

### [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#message)  `message`

Roles: `system`, `developer`, `user`, `assistant`.

- `system` and `developer` are appended to the system prompt.
- The most recent `user` or `function_call_output` item becomes the “current message.”
- Earlier user/assistant messages are included as history for context.

### [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#function_call_output-turn-based-tools)  `function_call_output` (turn-based tools)

Send tool results back to the model:

```
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#reasoning-and-item_reference)  `reasoning` and `item_reference`

Accepted for schema compatibility but ignored when building the prompt.

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#tools-client-side-function-tools)  Tools (client-side function tools)

Provide tools with `tools: [{ type: \"function\", function: { name, description?, parameters? } }]`.If the agent decides to call a tool, the response returns a `function_call` output item.
You then send a follow-up request with `function_call_output` to continue the turn.

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#images-input_image)  Images (`input_image`)

Supports base64 or URL sources:

```
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Allowed MIME types (current): `image/jpeg`, `image/png`, `image/gif`, `image/webp`, `image/heic`, `image/heif`.
Max size (current): 10MB.

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#files-input_file)  Files (`input_file`)

Supports base64 or URL sources:

```
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Allowed MIME types (current): `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.Max size (current): 5MB.Current behavior:

- File content is decoded and added to the **system prompt**, not the user message,
so it stays ephemeral (not persisted in session history).
- Decoded file text is wrapped as **untrusted external content** before it is added,
so file bytes are treated as data, not trusted instructions.
- The injected block uses explicit boundary markers like
`<<<EXTERNAL_UNTRUSTED_CONTENT id=\"...\">>>` /
`<<<END_EXTERNAL_UNTRUSTED_CONTENT id=\"...\">>>` and includes a
`Source: External` metadata line.
- This file-input path intentionally omits the long `SECURITY NOTICE:` banner to
preserve prompt budget; the boundary markers and metadata still stay in place.
- PDFs are parsed for text first. If little text is found, the first pages are
rasterized into images and passed to the model, and the injected file block uses
the placeholder `[PDF content rendered to images]`.

PDF parsing is provided by the bundled `document-extract` plugin, which uses the
Node-friendly `pdfjs-dist` legacy build (no worker). The modern PDF.js build
expects browser workers/DOM globals, so it is not used in the Gateway.URL fetch defaults:

- `files.allowUrl`: `true`
- `images.allowUrl`: `true`
- `maxUrlParts`: `8` (total URL-based `input_file` \\+ `input_image` parts per request)
- Requests are guarded (DNS resolution, private IP blocking, redirect caps, timeouts).
- Optional hostname allowlists are supported per input type (`files.urlAllowlist`, `images.urlAllowlist`).

  - Exact host: `\"cdn.example.com\"`
  - Wildcard subdomains: `\"*.assets.example.com\"` (does not match apex)
  - Empty or omitted allowlists mean no hostname allowlist restriction.
- To disable URL-based fetches entirely, set `files.allowUrl: false` and/or `images.allowUrl: false`.

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#file-+-image-limits-config)  File + image limits (config)

Defaults can be tuned under `gateway.http.endpoints.responses`:

```
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          maxUrlParts: 8,
          files: {
            allowUrl: true,
            urlAllowlist: [\"cdn.example.com\", \"*.assets.example.com\"],
            allowedMimes: [\\\
              \"text/plain\",\\\
              \"text/markdown\",\\\
              \"text/html\",\\\
              \"text/csv\",\\\
              \"application/json\",\\\
              \"application/pdf\",\\\
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            urlAllowlist: [\"images.example.com\"],
            allowedMimes: [\\\
              \"image/jpeg\",\\\
              \"image/png\",\\\
              \"image/gif\",\\\
              \"image/webp\",\\\
              \"image/heic\",\\\
              \"image/heif\",\\\
            ],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

Defaults when omitted:

- `maxBodyBytes`: 20MB
- `maxUrlParts`: 8
- `files.maxBytes`: 5MB
- `files.maxChars`: 200k
- `files.maxRedirects`: 3
- `files.timeoutMs`: 10s
- `files.pdf.maxPages`: 4
- `files.pdf.maxPixels`: 4,000,000
- `files.pdf.minTextChars`: 200
- `images.maxBytes`: 10MB
- `images.maxRedirects`: 3
- `images.timeoutMs`: 10s
- HEIC/HEIF `input_image` sources are accepted and normalized to JPEG before provider delivery.

Security note:

- URL allowlists are enforced before fetch and on redirect hops.
- Allowlisting a hostname does not bypass private/internal IP blocking.
- For internet-exposed gateways, apply network egress controls in addition to app-level guards.
See [Security](https://docs.openclaw.ai/gateway/security).

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#streaming-sse)  Streaming (SSE)

Set `stream: true` to receive Server-Sent Events (SSE):

- `Content-Type: text/event-stream`
- Each event line is `event: <type>` and `data: <json>`
- Stream ends with `data: [DONE]`

Event types currently emitted:

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed` (on error)

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#usage)  Usage

`usage` is populated when the underlying provider reports token counts.
OpenClaw normalizes common OpenAI-style aliases before those counters reach
downstream status/session surfaces, including `input_tokens` / `output_tokens`
and `prompt_tokens` / `completion_tokens`.

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#errors)  Errors

Errors use a JSON object like:

```
{ \"error\": { \"message\": \"...\", \"type\": \"invalid_request_error\" } }
```

Common cases:

- `401` missing/invalid auth
- `400` invalid request body
- `405` wrong method

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#examples)  Examples

Non-streaming:

```
curl -sS http://127.0.0.1:18789/v1/responses \\\n  -H \'Authorization: Bearer YOUR_TOKEN\' \\\n  -H \'Content-Type: application/json\' \\\n  -H \'x-openclaw-agent-id: main\' \\\n  -d \'{\n    \"model\": \"openclaw\",\n    \"input\": \"hi\"\n  }\'
```

Streaming:

```
curl -N http://127.0.0.1:18789/v1/responses \\\n  -H \'Authorization: Bearer YOUR_TOKEN\' \\\n  -H \'Content-Type: application/json\' \\\n  -H \'x-openclaw-agent-id: main\' \\\n  -d \'{\n    \"model\": \"openclaw\",\n    \"stream\": true,\n    \"input\": \"hi\"\n  }\'
```

## [​](https://docs.openclaw.ai/gateway/openresponses-http-api\\#related)  Related

- [OpenAI chat completions](https://docs.openclaw.ai/gateway/openai-http-api)
- [OpenAI](https://docs.openclaw.ai/providers/openai)

[OpenAI chat completions](https://docs.openclaw.ai/gateway/openai-http-api) [Tools invoke API](https://docs.openclaw.ai/gateway/tools-invoke-http-api)

Ctrl+I

---

