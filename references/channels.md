# OpenClaw Channels Documentation

## Nostr - OpenClaw
**Source:** https://docs.openclaw.ai/channels/nostr

[Skip to main content](https://docs.openclaw.ai/channels/nostr#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Developer and self-hosted

Nostr

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Bundled plugin](https://docs.openclaw.ai/channels/nostr#bundled-plugin)
- [Older/custom installs](https://docs.openclaw.ai/channels/nostr#older%2Fcustom-installs)
- [Non-interactive setup](https://docs.openclaw.ai/channels/nostr#non-interactive-setup)
- [Quick setup](https://docs.openclaw.ai/channels/nostr#quick-setup)
- [Configuration reference](https://docs.openclaw.ai/channels/nostr#configuration-reference)
- [Profile metadata](https://docs.openclaw.ai/channels/nostr#profile-metadata)
- [Access control](https://docs.openclaw.ai/channels/nostr#access-control)
- [DM policies](https://docs.openclaw.ai/channels/nostr#dm-policies)
- [Allowlist example](https://docs.openclaw.ai/channels/nostr#allowlist-example)
- [Key formats](https://docs.openclaw.ai/channels/nostr#key-formats)
- [Relays](https://docs.openclaw.ai/channels/nostr#relays)
- [Protocol support](https://docs.openclaw.ai/channels/nostr#protocol-support)
- [Testing](https://docs.openclaw.ai/channels/nostr#testing)
- [Local relay](https://docs.openclaw.ai/channels/nostr#local-relay)
- [Manual test](https://docs.openclaw.ai/channels/nostr#manual-test)
- [Troubleshooting](https://docs.openclaw.ai/channels/nostr#troubleshooting)
- [Not receiving messages](https://docs.openclaw.ai/channels/nostr#not-receiving-messages)
- [Not sending responses](https://docs.openclaw.ai/channels/nostr#not-sending-responses)
- [Duplicate responses](https://docs.openclaw.ai/channels/nostr#duplicate-responses)
- [Security](https://docs.openclaw.ai/channels/nostr#security)
- [Limitations (MVP)](https://docs.openclaw.ai/channels/nostr#limitations-mvp)
- [Related](https://docs.openclaw.ai/channels/nostr#related)

**Status:** Optional bundled plugin (disabled by default until configured).Nostr is a decentralized protocol for social networking. This channel enables OpenClaw to receive and respond to encrypted direct messages (DMs) via NIP-04.

## [​](https://docs.openclaw.ai/channels/nostr\\#bundled-plugin)  Bundled plugin

Current OpenClaw releases ship Nostr as a bundled plugin, so normal packaged
builds do not need a separate install.

### [​](https://docs.openclaw.ai/channels/nostr\\#older/custom-installs)  Older/custom installs

- Onboarding (`openclaw onboard`) and `openclaw channels add` still surface
Nostr from the shared channel catalog.
- If your build excludes bundled Nostr, install it manually.

```
openclaw plugins install @openclaw/nostr
```

Use a local checkout (dev workflows):

```
openclaw plugins install --link <path-to-local-nostr-plugin>
```

Restart the Gateway after installing or enabling plugins.

### [​](https://docs.openclaw.ai/channels/nostr\\#non-interactive-setup)  Non-interactive setup

```
openclaw channels add --channel nostr --private-key \"$NOSTR_PRIVATE_KEY\"
openclaw channels add --channel nostr --private-key \"$NOSTR_PRIVATE_KEY\" --relay-urls \"wss://relay.damus.io,wss://relay.primal.net\"
```

Use `--use-env` to keep `NOSTR_PRIVATE_KEY` in the environment instead of storing the key in config.

## [​](https://docs.openclaw.ai/channels/nostr\\#quick-setup)  Quick setup

1. Generate a Nostr keypair (if needed):

```
# Using nak
nak key generate
```

2. Add to config:

```
{
  channels: {
    nostr: {
      privateKey: \"${NOSTR_PRIVATE_KEY}\",
    },
  },
}
```

3. Export the key:

```
export NOSTR_PRIVATE_KEY=\"nsec1...\"
```

4. Restart the Gateway.

## [​](https://docs.openclaw.ai/channels/nostr\\#configuration-reference)  Configuration reference

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `privateKey` | string | required | Private key in `nsec` or hex format |
| `relays` | string\\[\\] | `["wss://relay.damus.io", "wss://nos.lol"]` | Relay URLs (WebSocket) |
| `dmPolicy` | string | `pairing` | DM access policy |
| `allowFrom` | string\\[\\] | `[]` | Allowed sender pubkeys |
| `enabled` | boolean | `true` | Enable/disable channel |
| `name` | string | - | Display name |
| `profile` | object | - | NIP-01 profile metadata |

## [​](https://docs.openclaw.ai/channels/nostr\\#profile-metadata)  Profile metadata

Profile data is published as a NIP-01 `kind:0` event. You can manage it from the Control UI (Channels -> Nostr -> Profile) or set it directly in config.Example:

```
{
  channels: {
    nostr: {
      privateKey: \"${NOSTR_PRIVATE_KEY}\",
      profile: {
        name: \"openclaw\",
        displayName: \"OpenClaw\",
        about: \"Personal assistant DM bot\",
        picture: \"https://example.com/avatar.png\",
        banner: \"https://example.com/banner.png\",
        website: \"https://example.com\",
        nip05: \"openclaw@example.com\",
        lud16: \"openclaw@example.com\",
      },
    },
  },
}
```

Notes:

- Profile URLs must use `https://`.
- Importing from relays merges fields and preserves local overrides.

## [​](https://docs.openclaw.ai/channels/nostr\\#access-control)  Access control

### [​](https://docs.openclaw.ai/channels/nostr\\#dm-policies)  DM policies

- **pairing** (default): unknown senders get a pairing code.
- **allowlist**: only pubkeys in `allowFrom` can DM.
- **open**: public inbound DMs (requires `allowFrom: [\"*\"]`).
- **disabled**: ignore inbound DMs.

Enforcement notes:

- Inbound event signatures are verified before sender policy and NIP-04 decryption, so forged events are rejected early.
- Pairing replies are sent without processing the original DM body.
- Inbound DMs are rate-limited and oversized payloads are dropped before decrypt.

### [​](https://docs.openclaw.ai/channels/nostr\\#allowlist-example)  Allowlist example

```
{
  channels: {
    nostr: {
      privateKey: \"${NOSTR_PRIVATE_KEY}\",
      dmPolicy: \"allowlist\",
      allowFrom: [\"npub1abc...\", \"npub1xyz\"],
    },
  },
}
```

## [​](https://docs.openclaw.ai/channels/nostr\\#key-formats)  Key formats

Accepted formats:

- **Private key:**`nsec...` or 64-char hex
- **Pubkeys (`allowFrom`):**`npub...` or hex

## [​](https://docs.openclaw.ai/channels/nostr\\#relays)  Relays

Defaults: `relay.damus.io` and `nos.lol`.

```
{
  channels: {
    nostr: {
      privateKey: \"${NOSTR_PRIVATE_KEY}\",
      relays: [\"wss://relay.damus.io\", \"wss://relay.primal.net\", \"wss://nostr.wine\"],
    },
  },
}
```

Tips:

- Use 2-3 relays for redundancy.
- Avoid too many relays (latency, duplication).
- Paid relays can improve reliability.
- Local relays are fine for testing (`ws://localhost:7777`).

## [​](https://docs.openclaw.ai/channels/nostr\\#protocol-support)  Protocol support

| NIP | Status | Description |
| --- | --- | --- |
| NIP-01 | Supported | Basic event format + profile metadata |
| NIP-04 | Supported | Encrypted DMs (`kind:4`) |
| NIP-17 | Planned | Gift-wrapped DMs |
| NIP-44 | Planned | Versioned encryption |

## [​](https://docs.openclaw.ai/channels/nostr\\#testing)  Testing

### [​](https://docs.openclaw.ai/channels/nostr\\#local-relay)  Local relay

```
# Start strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```
{
  channels: {
    nostr: {
      privateKey: \"${NOSTR_PRIVATE_KEY}\",
      relays: [\"ws://localhost:7777\"],
    },
  },
}
```

### [​](https://docs.openclaw.ai/channels/nostr\\#manual-test)  Manual test

1. Note the bot pubkey (npub) from logs.
2. Open a Nostr client (Damus, Amethyst, etc.).
3. DM the bot pubkey.
4. Verify the response.

## [​](https://docs.openclaw.ai/channels/nostr\\#troubleshooting)  Troubleshooting

### [​](https://docs.openclaw.ai/channels/nostr\\#not-receiving-messages)  Not receiving messages

- Verify the private key is valid.
- Ensure relay URLs are reachable and use `wss://` (or `ws://` for local).
- Confirm `enabled` is not `false`.
- Check Gateway logs for relay connection errors.

### [​](https://openclaw.ai/channels/nostr\\#not-sending-responses)  Not sending responses

- Check relay accepts writes.
- Verify outbound connectivity.
- Watch for relay rate limits.

### [​](https://docs.openclaw.ai/channels/nostr\\#duplicate-responses)  Duplicate responses

- Expected when using multiple relays.
- Messages are deduplicated by event ID; only the first delivery triggers a response.

## [​](https://docs.openclaw.ai/channels/nostr\\#security)  Security

- Never commit private keys.
- Use environment variables for keys.
- Consider `allowlist` for production bots.
- Signatures are verified before sender policy, and sender policy is enforced before decrypt, so forged events are rejected early and unknown senders cannot force full crypto work.

## [​](https://docs.openclaw.ai/channels/nostr\\#limitations-mvp)  Limitations (MVP)

- Direct messages only (no group chats).
- No media attachments.
- NIP-04 only (NIP-17 gift-wrap planned).

## [​](https://docs.openclaw.ai/channels/nostr\\#related)  Related

- [Channels Overview](https://docs.openclaw.ai/channels) — all supported channels
- [Pairing](https://docs.openclaw.ai/channels/pairing) — DM authentication and pairing flow
- [Groups](https://docs.openclaw.ai/channels/groups) — group chat behavior and mention gating
- [Channel Routing](https://docs.openclaw.ai/channels/channel-routing) — session routing for messages
- [Security](https://docs.openclaw.ai/gateway/security) — access model and hardening

[Nextcloud Talk](https://docs.openclaw.ai/channels/nextcloud-talk) [Tlon](https://docs.openclaw.ai/channels/tlon)

---

## Channel location parsing - OpenClaw
**Source:** https://docs.openclaw.ai/channels/location

[Skip to main content](https://docs.openclaw.ai/channels/location#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Configuration

Channel location parsing

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Text formatting](https://docs.openclaw.ai/channels/location#text-formatting)
- [Context fields](https://docs.openclaw.ai/channels/location#context-fields)
- [Channel notes](https://docs.openclaw.ai/channels/location#channel-notes)
- [Related](https://docs.openclaw.ai/channels/location#related)

OpenClaw normalizes shared locations from chat channels into:

- terse coordinate text appended to the inbound body, and
- structured fields in the auto-reply context payload. Channel-provided labels, addresses, and captions/comments are rendered into the prompt by the shared untrusted metadata JSON block, not inline in the user body.

Currently supported:

- **Telegram** (location pins + venues + live locations)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` with `geo_uri`)

## [​](https://docs.openclaw.ai/channels/location\\#text-formatting)  Text formatting

Locations are rendered as friendly lines without brackets:

- Pin:
  - `📍 48.858844, 2.294351 ±12m`
- Named place:
  - `📍 48.858844, 2.294351 ±12m`
- Live share:
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

If the channel includes a label, address, or caption/comment, it is preserved in the context payload and appears in the prompt as fenced untrusted JSON:

````
Location (untrusted metadata):
```json
{
\"latitude\": 48.858844,\n\"longitude\": 2.294351,\n\"name\": \"Eiffel Tower\",\n\"address\": \"Champ de Mars, Paris\",\n\"caption\": \"Meet here\"\n}
```
````

## [​](https://docs.openclaw.ai/channels/location\\#context-fields)  Context fields

When a location is present, these fields are added to `ctx`:

- `LocationLat` (number)
- `LocationLon` (number)
- `LocationAccuracy` (number, meters; optional)
- `LocationName` (string; optional)
- `LocationAddress` (string; optional)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (boolean)
- `LocationCaption` (string; optional)

The prompt renderer treats `LocationName`, `LocationAddress`, and `LocationCaption` as untrusted metadata and serializes them through the same bounded JSON path used for other channel context.

## [​](https://docs.openclaw.ai/channels/location\\#channel-notes)  Channel notes

- **Telegram**: venues map to `LocationName/LocationAddress`; live locations use `live_period`.
- **WhatsApp**: `locationMessage.comment` and `liveLocationMessage.caption` populate `LocationCaption`.
- **Matrix**: `geo_uri` is parsed as a pin location; altitude is ignored and `LocationIsLive` is always false.

## [​](https://docs.openclaw.ai/channels/location\\#related)  Related

- [Location command (nodes)](https://docs.openclaw.ai/nodes/location-command)
- [Camera capture](https://docs.openclaw.ai/nodes/camera)
- [Media understanding](https://docs.openclaw.ai/nodes/media-understanding)

[Channel routing](https://docs.openclaw.ai/channels/channel-routing) [Channel troubleshooting](https://docs.openclaw.ai/channels/troubleshooting)

Ctrl+I

---

## QQ bot - OpenClaw
**Source:** https://docs.openclaw.ai/channels/qqbot

[Skip to main content](https://docs.openclaw.ai/channels/qqbot#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Regional platforms

QQ bot

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Bundled plugin](https://docs.openclaw.ai/channels/qqbot#bundled-plugin)
- [Setup](https://docs.openclaw.ai/channels/qqbot#setup)
- [Configure](https://docs.openclaw.ai/channels/qqbot#configure)
- [Multi-account setup](https://docs.openclaw.ai/channels/qqbot#multi-account-setup)
- [Group chats](https://docs.openclaw.ai/channels/qqbot#group-chats)
- [Voice (STT / TTS)](https://docs.openclaw.ai/channels/qqbot#voice-stt-%2F-tts)
- [Target formats](https://docs.openclaw.ai/channels/qqbot#target-formats)
- [Slash commands](https://docs.openclaw.ai/channels/qqbot#slash-commands)
- [Engine architecture](https://docs.openclaw.ai/channels/qqbot#engine-architecture)
- [QR-code onboarding](https://docs.openclaw.ai/channels/qqbot#qr-code-onboarding)
- [Troubleshooting](https://docs.openclaw.ai/channels/qqbot#troubleshooting)
- [Related](https://docs.openclaw.ai/channels/qqbot#related)

QQ Bot connects to OpenClaw via the official QQ Bot API (WebSocket gateway). The
plugin supports C2C private chat, group @messages, and guild channel messages with
rich media (images, voice, video, files).Status: bundled plugin. Direct messages, group chats, guild channels, and
media are supported. Reactions and threads are not supported.

## [​](https://docs.openclaw.ai/channels/qqbot\\#bundled-plugin)  Bundled plugin

Current OpenClaw releases bundle QQ Bot, so normal packaged builds do not need
a separate `openclaw plugins install` step.

## [​](https://docs.openclaw.ai/channels/qqbot\\#setup)  Setup

1. Go to the [QQ Open Platform](https://q.qq.com/) and scan the QR code with your
phone QQ to register / log in.
2. Click **Create Bot** to create a new QQ bot.
3. Find **AppID** and **AppSecret** on the bot’s settings page and copy them.

> AppSecret is not stored in plaintext — if you leave the page without saving it,
> you’ll have to regenerate a new one.

4. Add the channel:

```
openclaw channels add --channel qqbot --token \"AppID:AppSecret\"
```

5. Restart the Gateway.

Interactive setup paths:

```
openclaw channels add
openclaw configure --section channels
```

## [​](https://docs.openclaw.ai/channels/qqbot\\#configure)  Configure

Minimal config:

```
{
  channels: {
    qqbot: {
      enabled: true,
      appId: \"YOUR_APP_ID\",
      clientSecret: \"YOUR_APP_SECRET\",
    },
  },
}
```

Default-account env vars:

- `QQBOT_APP_ID`
- `QQBOT_CLIENT_SECRET`

File-backed AppSecret:

```
{
  channels: {
    qqbot: {
      enabled: true,
      appId: \"YOUR_APP_ID\",
      clientSecretFile: \"/path/to/qqbot-secret.txt\",
    },
  },
}
```

Notes:

- Env fallback applies to the default QQ Bot account only.
- `openclaw channels add --channel qqbot --token-file ...` provides the
AppSecret only; the AppID must already be set in config or `QQBOT_APP_ID`.
- `clientSecret` also accepts SecretRef input, not just a plaintext string.

### [​](https://docs.openclaw.ai/channels/qqbot\\#multi-account-setup)  Multi-account setup

Run multiple QQ bots under a single OpenClaw instance:

```
{
  channels: {
    qqbot: {
      enabled: true,
      appId: \"111111111\",
      clientSecret: \"secret-of-bot-1\",
      accounts: {
        bot2: {
          enabled: true,
          appId: \"222222222\",
          clientSecret: \"secret-of-bot-2\",
        },
      },
    },
  },
}
```

Each account launches its own WebSocket connection and maintains an independent
token cache (isolated by `appId`).Add a second bot via CLI:

```
openclaw channels add --channel qqbot --account bot2 --token \"222222222:secret-of-bot-2\"
```

### [​](https://docs.openclaw.ai/channels/qqbot\\#group-chats)  Group chats

QQ Bot group chat support uses QQ group OpenIDs, not display names. Add the bot
to a group, then mention it or configure the group to run without a mention.

```
{
  channels: {
    qqbot: {
      groupPolicy: \"allowlist\",
      groupAllowFrom: [\"member_openid\"],
      groups: {
        \"*\": {
          requireMention: true,
          historyLimit: 50,
          toolPolicy: \"restricted\",
        },
        GROUP_OPENID: {
          name: \"Release room\",
          requireMention: false,
          ignoreOtherMentions: true,
          historyLimit: 20,
          prompt: \"Keep replies short and operational.\",
        },
      },
    },
  },
}
```

`groups[\"*\"]` sets defaults for every group, and a concrete
`groups.GROUP_OPENID` entry overrides those defaults for one group. Group
settings include:

- `requireMention`: require an @mention before the bot replies. Default: `true`.
- `ignoreOtherMentions`: drop messages that mention someone else but not the bot.
- `historyLimit`: keep recent non-mention group messages as context for the next mentioned turn. Set `0` to disable.
- `toolPolicy`: `full`, `restricted`, or `none` for group-scoped tools.
- `name`: friendly label used in logs and group context.
- `prompt`: per-group behavior prompt appended to the agent context.

Activation modes are `mention` and `always`. `requireMention: true` maps to
`mention`; `requireMention: false` maps to `always`. A session-level activation
override, when present, wins over config.The inbound queue is per peer. Group peers get a larger queue cap, keep human
messages ahead of bot-authored chatter when full, and merge bursts of normal
group messages into one attributed turn. Slash commands still run one by one.

### [​](https://docs.openclaw.ai/channels/qqbot\\#voice-stt-/-tts)  Voice (STT / TTS)

STT and TTS support two-level configuration with priority fallback:

| Setting | Plugin-specific | Framework fallback |
| --- | --- | --- |
| STT | `channels.qqbot.stt` | `tools.media.audio.models[0]` |
| TTS | `channels.qqbot.tts`, `channels.qqbot.accounts.<id>.tts` | `messages.tts` |

```
{
  channels: {
    qqbot: {
      stt: {
        provider: \"your-provider\",
        model: \"your-stt-model\",
      },
      tts: {
        provider: \"your-provider\",
        model: \"your-tts-model\",
        voice: \"your-voice\",
      },
      accounts: {
        qq-main: {
          tts: {
            providers: {
              openai: { voice: \"shimmer\" },
            },
          },
        },
      },
    },
  },
}
```

Set `enabled: false` on either to disable.
Account-level TTS overrides use the same shape as `messages.tts` and deep-merge
over the channel/global TTS config.Inbound QQ voice attachments are exposed to agents as audio media metadata while
keeping raw voice files out of generic `MediaPaths`. `[[audio_as_voice]]` plain
text replies synthesize TTS and send a native QQ voice message when TTS is
configured.Outbound audio upload/transcode behavior can also be tuned with
`channels.qqbot.audioFormatPolicy`:

- `sttDirectFormats`
- `uploadDirectFormats`
- `transcodeEnabled`

## [​](https://docs.openclaw.ai/channels/qqbot\\#target-formats)  Target formats

| Format | Description |
| --- | --- |
| `qqbot:c2c:OPENID` | Private chat (C2C) |
| `qqbot:group:GROUP_OPENID` | Group chat |
| `qqbot:channel:CHANNEL_ID` | Guild channel |

> Each bot has its own set of user OpenIDs. An OpenID received by Bot A **cannot**
> be used to send messages via Bot B.

## [​](https://docs.openclaw.ai/channels/qqbot\\#slash-commands)  Slash commands

Built-in commands intercepted before the AI queue:

| Command | Description |
| --- | --- |
| `/bot-ping` | Latency test |
| `/bot-version` | Show the OpenClaw framework version |
| `/bot-help` | List all commands |
| `/bot-upgrade` | Show the QQBot upgrade guide link |
| `/bot-logs` | Export recent gateway logs as a file |
| `/bot-approve` | Approve a pending QQ Bot action (for example, confirming a C2C or group upload) through the native flow. |

Append `?` to any command for usage help (for example `/bot-upgrade ?`).

## [​](https://docs.openclaw.ai/channels/qqbot\\#engine-architecture)  Engine architecture

QQ Bot ships as a self-contained engine inside the plugin:

- Each account owns an isolated resource stack (WebSocket connection, API client, token cache, media storage root) keyed by `appId`. Accounts never share inbound/outbound state.
- The multi-account logger tags log lines with the owning account so diagnostics stay separable when you run several bots under one gateway.
- Inbound, outbound, and gateway bridge paths share a single media payload root under `~/.openclaw/media`, so uploads, downloads, and transcode caches land under one guarded directory instead of a per-subsystem tree.
- Rich media delivery goes through one `sendMedia` path for C2C and group targets. Local files and buffers above the large-file threshold use QQ’s chunked upload endpoints, while smaller payloads use the one-shot media API.
- Credentials can be backed up and restored as part of standard OpenClaw credential snapshots; the engine re-attaches each account’s resource stack on restore without requiring a fresh QR-code pair.

## [​](https://docs.openclaw.ai/channels/qqbot\\#qr-code-onboarding)  QR-code onboarding

As an alternative to pasting `AppID:AppSecret` manually, the engine supports a QR-code onboarding flow for linking a QQ Bot to OpenClaw:

1. Run the QQ Bot setup path (for example `openclaw channels add --channel qqbot`) and pick the QR-code flow when prompted.
2. Scan the generated QR code with the phone app tied to the target QQ Bot.
3. Approve the pairing on the phone. OpenClaw persists the returned credentials into `credentials/` under the right account scope.

Approval prompts generated by the bot itself (for example, “allow this action?” flows exposed by the QQ Bot API) surface as native OpenClaw prompts that you can accept with `/bot-approve` rather than replying through the raw QQ client.

## [​](https://docs.openclaw.ai/channels/qqbot\\#troubleshooting)  Troubleshooting

- **Bot replies “gone to Mars”:** credentials not configured or Gateway not started.
- **No inbound messages:** verify `appId` and `clientSecret` are correct, and the
bot is enabled on the QQ Open Platform.
- **Repeated self-replies:** OpenClaw records QQ outbound ref indexes as
bot-authored and ignores inbound events whose current `msgIdx` matches that
same bot account. This prevents platform echo loops while still allowing users
to quote or reply to previous bot messages.
- **Setup with `--token-file` still shows unconfigured:**`--token-file` only sets
the AppSecret. You still need `appId` in config or `QQBOT_APP_ID`.
- **Proactive messages not arriving:** QQ may intercept bot-initiated messages if
the user hasn’t interacted recently.
- **Voice not transcribed:** ensure STT is configured and the provider is reachable.

## [​](https://docs.openclaw.ai/channels/qqbot\\#related)  Related

- [Pairing](https://docs.openclaw.ai/channels/pairing)
- [Groups](https://docs.openclaw.ai/channels/groups)
- [Channel troubleshooting](https://docs.openclaw.ai/channels/troubleshooting)

[WeChat](https://docs.openclaw.ai/channels/wechat) [Feishu](https://docs.openclaw.ai/channels/feishu)

Ctrl+I

---

## IRC - OpenClaw
**Source:** https://docs.openclaw.ai/channels/irc

[Skip to main content](https://docs.openclaw.ai/channels/irc#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Developer and self-hosted

IRC

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/channels/irc#quick-start)
- [Security defaults](https://docs.openclaw.ai/channels/irc#security-defaults)
- [Access control](https://docs.openclaw.ai/channels/irc#access-control)
- [Common gotcha: allowFrom is for DMs, not channels](https://docs.openclaw.ai/channels/irc#common-gotcha-allowfrom-is-for-dms-not-channels)
- [Reply triggering (mentions)](https://docs.openclaw.ai/channels/irc#reply-triggering-mentions)
- [Security note (recommended for public channels)](https://docs.openclaw.ai/channels/irc#security-note-recommended-for-public-channels)
- [Same tools for everyone in the channel](https://docs.openclaw.ai/channels/irc#same-tools-for-everyone-in-the-channel)
- [Different tools per sender (owner gets more power)](https://docs.openclaw.ai/channels/irc#different-tools-per-sender-owner-gets-more-power)
- [NickServ](https://docs.openclaw.ai/channels/irc#nickserv)
- [Environment variables](https://docs.openclaw.ai/channels/irc#environment-variables)
- [Troubleshooting](https://docs.openclaw.ai/channels/irc#troubleshooting)
- [Related](https://docs.openclaw.ai/channels/irc#related)

Use IRC when you want OpenClaw in classic channels (`#room`) and direct messages.
IRC ships as a bundled plugin, but it is configured in the main config under `channels.irc`.

## [​](https://docs.openclaw.ai/channels/irc\\#quick-start)  Quick start

1. Enable IRC config in `~/.openclaw/openclaw.json`.
2. Set at least:

```
{
  channels: {
    irc: {
      enabled: true,
      host: \"irc.example.com\",
      port: 6697,
      tls: true,
      nick: \"openclaw-bot\",
      channels: [\"#openclaw\"],
    },
  },
}
```

Prefer a private IRC server for bot coordination. If you intentionally use a public IRC network, common choices include Libera.Chat, OFTC, and Snoonet. Avoid predictable public channels for bot or swarm backchannel traffic.

3. Start/restart gateway:

```
openclaw gateway run
```

## [​](https://docs.openclaw.ai/channels/irc\\#security-defaults)  Security defaults

- `channels.irc.dmPolicy` defaults to `\"pairing\"`.
- `channels.irc.groupPolicy` defaults to `\"allowlist\"`.
- With `groupPolicy=\"allowlist\"`, set `channels.irc.groups` to define allowed channels.
- Use TLS (`channels.irc.tls=true`) unless you intentionally accept plaintext transport.

## [​](https://docs.openclaw.ai/channels/irc\\#access-control)  Access control

There are two separate “gates” for IRC channels:

1. **Channel access** (`groupPolicy` \\+ `groups`): whether the bot accepts messages from a channel at all.
2. **Sender access** (`groupAllowFrom` / per-channel `groups[\"#channel\"].allowFrom`): who is allowed to trigger the bot inside that channel.

Config keys:

- DM allowlist (DM sender access): `channels.irc.allowFrom`
- Group sender allowlist (channel sender access): `channels.irc.groupAllowFrom`
- Per-channel controls (channel + sender + mention rules): `channels.irc.groups[\"#channel\"]`
- `channels.irc.groupPolicy=\"open\"` allows unconfigured channels ( **still mention-gated by default**)

Allowlist entries should use stable sender identities (`nick!user@host`).
Bare nick matching is mutable and only enabled when `channels.irc.dangerouslyAllowNameMatching: true`.

### [​](https://docs.openclaw.ai/channels/irc\\#common-gotcha-allowfrom-is-for-dms-not-channels)  Common gotcha: `allowFrom` is for DMs, not channels

If you see logs like:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…it means the sender wasn’t allowed for **group/channel** messages. Fix it by either:

- setting `channels.irc.groupAllowFrom` (global for all channels), or
- setting per-channel sender allowlists: `channels.irc.groups[\"#channel\"].allowFrom`

Example (allow anyone in `#tuirc-dev` to talk to the bot):

```
{
  channels: {
    irc: {
      groupPolicy: \"allowlist\",
      groups: {
        \"#tuirc-dev\": { allowFrom: [\"*\"] },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/channels/irc\\#reply-triggering-mentions)  Reply triggering (mentions)

Even if a channel is allowed (via `groupPolicy` \\+ `groups`) and the sender is allowed, OpenClaw defaults to **mention-gating** in group contexts.That means you may see logs like `drop channel … (missing-mention)` unless the message includes a mention pattern that matches the bot.To make the bot reply in an IRC channel **without needing a mention**, disable mention gating for that channel:

```
{
  channels: {
    irc: {
      groupPolicy: \"allowlist\",
      groups: {
        \"#tuirc-dev\": {
          requireMention: false,
          allowFrom: [\"*\"],
        },
      },
    },
  },
}
```

Or to allow **all** IRC channels (no per-channel allowlist) and still reply without mentions:

```
{
  channels: {
    irc: {
      groupPolicy: \"open\",
      groups: {
        \"*\": { requireMention: false, allowFrom: [\"*\"] },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/channels/irc\\#security-note-recommended-for-public-channels)  Security note (recommended for public channels)

If you allow `allowFrom: [\"*\"]` in a public channel, anyone can prompt the bot.
To reduce risk, restrict tools for that channel.

### [​](https://docs.openclaw.ai/channels/irc\\#same-tools-for-everyone-in-the-channel)  Same tools for everyone in the channel

```
{
  channels: {
    irc: {
      groups: {
        \"#tuirc-dev\": {
          allowFrom: [\"*\"],
          tools: {
            deny: [\"group:runtime\", \"group:fs\", \"gateway\", \"nodes\", \"cron\", \"browser\"],
          },
        },
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/channels/irc\\#different-tools-per-sender-owner-gets-more-power)  Different tools per sender (owner gets more power)

Use `toolsBySender` to apply a stricter policy to `\"*\"` and a looser one to your nick:

```
{
  channels: {
    irc: {
      groups: {
        \"#tuirc-dev\": {
          allowFrom: [\"*\"],
          toolsBySender: {
            \"*\": {
              deny: [\"group:runtime\", \"group:fs\", \"gateway\", \"nodes\", \"cron\", \"browser\"],
            },
            \"id:eigen\": {
              deny: [\"gateway\", \"nodes\", \"cron\"],
            },
          },
        },
      },
    },
  },
}
```

Notes:

- `toolsBySender` keys should use `id:` for IRC sender identity values:
`id:eigen` or `id:eigen!~eigen@174.127.248.171` for stronger matching.
- Legacy unprefixed keys are still accepted and matched as `id:` only.
- The first matching sender policy wins; `\"*\"` is the wildcard fallback.

For more on group access vs mention-gating (and how they interact), see: [/channels/groups](https://docs.openclaw.ai/channels/groups).

## [​](https://docs.openclaw.ai/channels/irc\\#nickserv)  NickServ

To identify with NickServ after connect:

```
{
  channels: {
    irc: {
      nickserv: {
        enabled: true,
        service: \"NickServ\",
        password: \"your-nickserv-password\",
      },
    },
  },
}
```

Optional one-time registration on connect:

```
{
  channels: {
    irc: {
      nickserv: {
        register: true,
        registerEmail: \"bot@example.com\",
      },
    },
  },
}
```

Disable `register` after the nick is registered to avoid repeated REGISTER attempts.

## [​](https://docs.openclaw.ai/channels/irc\\#environment-variables)  Environment variables

Default account supports:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (comma-separated)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

`IRC_HOST` cannot be set from a workspace `.env`; see [Workspace `.env` files](https://docs.openclaw.ai/gateway/security).

## [​](https://docs.openclaw.ai/channels/irc\\#troubleshooting)  Troubleshooting

- If the bot connects but never replies in channels, verify `channels.irc.groups` **and** whether mention-gating is dropping messages (`missing-mention`). If you want it to reply without pings, set `requireMention:false` for the channel.
- If login fails, verify nick availability and server password.
- If TLS fails on a custom network, verify host/port and certificate setup.

## [​](https://docs.openclaw.ai/channels/irc\\#related)  Related

- [Channels Overview](https://docs.openclaw.ai/channels) — all supported channels
- [Pairing](https://docs.openclaw.ai/channels/pairing) — DM authentication and pairing flow
- [Groups](https://docs.openclaw.ai/channels/groups) — group chat behavior and mention gating
- [Channel Routing](https://docs.openclaw.ai/channels/channel-routing) — session routing for messages
- [Security](https://docs.openclaw.ai/gateway/security) — access model and hardening

[Matrix push rules for quiet previews](https://docs.openclaw.ai/channels/matrix-push-rules) [Mattermost](https://docs.openclaw.ai/channels/mattermost)

Ctrl+I

---

## Broadcast groups - OpenClaw
**Source:** https://docs.openclaw.ai/channels/broadcast-groups

[Skip to main content](https://docs.openclaw.ai/channels/broadcast-groups#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Configuration

Broadcast groups

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Overview](https://docs.openclaw.ai/channels/broadcast-groups#overview)
- [Use cases](https://docs.openclaw.ai/channels/broadcast-groups#use-cases)
- [Configuration](https://docs.openclaw.ai/channels/broadcast-groups#configuration)
- [Basic setup](https://docs.openclaw.ai/channels/broadcast-groups#basic-setup)
- [Processing strategy](https://docs.openclaw.ai/channels/broadcast-groups#processing-strategy)
- [Complete example](https://docs.openclaw.ai/channels/broadcast-groups#complete-example)
- [How it works](https://docs.openclaw.ai/channels/broadcast-groups#how-it-works)
- [Message flow](https://docs.openclaw.ai/channels/broadcast-groups#message-flow)
- [Session isolation](https://docs.openclaw.ai/channels/broadcast-groups#session-isolation)
- [Example: isolated sessions](https://docs.openclaw.ai/channels/broadcast-groups#example-isolated-sessions)
- [Best practices](https://docs.openclaw.ai/channels/broadcast-groups#best-practices)
- [Compatibility](https://docs.openclaw.ai/channels/broadcast-groups#compatibility)
- [Providers](https://docs.openclaw.ai/channels/broadcast-groups#providers)
- [Routing](https://docs.openclaw.ai/channels/broadcast-groups#routing)
- [Troubleshooting](https://docs.openclaw.ai/channels/broadcast-groups#troubleshooting)
- [Examples](https://docs.openclaw.ai/channels/broadcast-groups#examples)
- [API reference](https://docs.openclaw.ai/channels/broadcast-groups#api-reference)
- [Config schema](https://docs.openclaw.ai/channels/broadcast-groups#config-schema)
- [Fields](https://docs.openclaw.ai/channels/broadcast-groups#fields)
- [Limitations](https://docs.openclaw.ai/channels/broadcast-groups#limitations)
- [Future enhancements](https://docs.openclaw.ai/channels/broadcast-groups#future-enhancements)
- [Related](https://docs.openclaw.ai/channels/broadcast-groups#related)

**Status:** Experimental. Added in 2026.1.9.

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#overview) Overview

Broadcast Groups enable multiple agents to process and respond to the same message simultaneously. This allows you to create specialized agent teams that work together in a single WhatsApp group or DM — all using one phone number.Current scope: **WhatsApp only** (web channel).Broadcast groups are evaluated after channel allowlists and group activation rules. In WhatsApp groups, this means broadcasts happen when OpenClaw would normally reply (for example: on mention, depending on your group settings).

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#use-cases) Use cases

1\. Specialized agent teams

Deploy multiple agents with atomic, focused responsibilities:

```
Group: \"Development Team\"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Each agent processes the same message and provides its specialized perspective.

2\. Multi-language support

```
Group: \"International Support\"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

3\. Quality assurance workflows

```
Group: \"Customer Support\"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

4\. Task automation

```
Group: \"Project Management\"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#configuration) Configuration

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#basic-setup) Basic setup

Add a top-level `broadcast` section (next to `bindings`). Keys are WhatsApp peer ids:

- group chats: group JID (e.g. `120363403215116621@g.us`)
- DMs: E.164 phone number (e.g. `+15551234567`)

```
{
  \"broadcast\": {
    \"120363403215116621@g.us\": [\"alfred\", \"baerbel\", \"assistant3\"]
  }
}
```

**Result:** When OpenClaw would reply in this chat, it will run all three agents.

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#processing-strategy) Processing strategy

Control how agents process messages:

- parallel (default)

- sequential


All agents process simultaneously:

```
{
  \"broadcast\": {
    \"strategy\": \"parallel\",
    \"120363403215116621@g.us\": [\"alfred\", \"baerbel\"]
  }
}
```

Agents process in order (one waits for previous to finish):

```
{
  \"broadcast\": {
    \"strategy\": \"sequential\",
    \"120363403215116621@g.us\": [\"alfred\", \"baerbel\"]
  }
}
```

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#complete-example) Complete example

```
{
  \"agents\": {
    \"list\": [\\\
      {\\\
        \"id\": \"code-reviewer\",\\\
        \"name\": \"Code Reviewer\",\\\
        \"workspace\": \"/path/to/code-reviewer\",\\\
        \"sandbox\": { \"mode\": \"all\" }\\\
      },\\\n      {\\\
        \"id\": \"security-auditor\",\\\
        \"name\": \"Security Auditor\",\\\
        \"workspace\": \"/path/to/security-auditor\",\\\
        \"sandbox\": { \"mode\": \"all\" }\\\
      },\\\n      {\\\
        \"id\": \"docs-generator\",\\\
        \"name\": \"Documentation Generator\",\\\
        \"workspace\": \"/path/to/docs-generator\",\\\
        \"sandbox\": { \"mode\": \"all\" }\\\
      }\\\
    ]
  },
  \"broadcast\": {
    \"strategy\": \"parallel\",
    \"120363403215116621@g.us\": [\"code-reviewer\", \"security-auditor\", \"docs-generator\"],
    \"120363424282127706@g.us\": [\"support-en\", \"support-de\"],
    \"+15555550123\": [\"assistant\", \"logger\"]
  }
}
```

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#how-it-works) How it works

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#message-flow) Message flow

1

[Navigate to header](https://docs.openclaw.ai/channels/broadcast-groups#)

Incoming message arrives

A WhatsApp group or DM message arrives.

2

[Navigate to header](https://docs.openclaw.ai/channels/broadcast-groups#)

Broadcast check

System checks if peer ID is in `broadcast`.

3

[Navigate to header](https://docs.openclaw.ai/channels/broadcast-groups#)

If in broadcast list

- All listed agents process the message.
- Each agent has its own session key and isolated context.
- Agents process in parallel (default) or sequentially.

4

[Navigate to header](https://docs.openclaw.ai/channels/broadcast-groups#)

If not in broadcast list

Normal routing applies (first matching binding).

Broadcast groups do not bypass channel allowlists or group activation rules (mentions/commands/etc). They only change _which agents run_ when a message is eligible for processing.

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#session-isolation) Session isolation

Each agent in a broadcast group maintains completely separate:

- **Session keys** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
- **Conversation history** (agent doesn’t see other agents’ messages)
- **Workspace** (separate sandboxes if configured)
- **Tool access** (different allow/deny lists)
- **Memory/context** (separate IDENTITY.md, SOUL.md, etc.)
- **Group context buffer** (recent group messages used for context) is shared per peer, so all broadcast agents see the same context when triggered

This allows each agent to have:

- Different personalities
- Different tool access (e.g., read-only vs. read-write)
- Different models (e.g., opus vs. sonnet)
- Different skills installed

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#example-isolated-sessions) Example: isolated sessions

In group `120363403215116621@g.us` with agents `[\"alfred\", \"baerbel\"]`:

- Alfred\'s context

- Bärbel\'s context


```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred\'s previous responses]
Workspace: /Users/user/openclaw-alfred/
Tools: read, write, exec
```

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel\'s previous responses]
Workspace: /Users/user/openclaw-baerbel/
Tools: read only
```

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#best-practices) Best practices

1\. Keep agents focused

Design each agent with a single, clear responsibility:

```
{
  \"broadcast\": {
    \"DEV_GROUP\": [\"formatter\", \"linter\", \"tester\"]
  }
}
```

✅ **Good:** Each agent has one job. ❌ **Bad:** One generic “dev-helper” agent.

2\. Use descriptive names

Make it clear what each agent does:

```
{
  \"agents\": {
    \"security-scanner\": { \"name\": \"Security Scanner\" },
    \"code-formatter\": { \"name\": \"Code Formatter\" },
    \"test-generator\": { \"name\": \"Test Generator\" }
  }
}
```

3\. Configure different tool access

Give agents only the tools they need:

```
{
  \"agents\": {
    \"reviewer\": {
      \"tools\": { \"allow\": [\"read\", \"exec\"] } // Read-only
    },
    \"fixer\": {
      \"tools\": { \"allow\": [\"read\", \"write\", \"edit\", \"exec\"] } // Read-write
    }
  }
}
```

4\. Monitor performance

With many agents, consider:

- Using `\"strategy\": \"parallel\"` (default) for speed
- Limiting broadcast groups to 5-10 agents
- Using faster models for simpler agents

5\. Handle failures gracefully

Agents fail independently. One agent’s error doesn’t block others:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#compatibility) Compatibility

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#providers) Providers

Broadcast groups currently work with:

- ✅ WhatsApp (implemented)
- 🚧 Telegram (planned)
- 🚧 Discord (planned)
- 🚧 Slack (planned)

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#routing) Routing

Broadcast groups work alongside existing routing:

```
{
  \"bindings\": [\\\
    {\\\\n      \"match\": { \"channel\": \"whatsapp\", \"peer\": { \"kind\": \"group\", \"id\": \"GROUP_A\" } },\\\
      \"agentId\": \"alfred\"\\\
    }\\\\n  ],
  \"broadcast\": {
    \"GROUP_B\": [\"agent1\", \"agent2\"]
  }
}
```

- `GROUP_A`: Only alfred responds (normal routing).
- `GROUP_B`: agent1 AND agent2 respond (broadcast).

**Precedence:**`broadcast` takes priority over `bindings`.

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#troubleshooting) Troubleshooting

Agents not responding

**Check:**

1. Agent IDs exist in `agents.list`.
2. Peer ID format is correct (e.g., `120363403215116621@g.us`).
3. Agents are not in deny lists.

**Debug:**

```
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

Only one agent responding

**Cause:** Peer ID might be in `bindings` but not `broadcast`.**Fix:** Add to broadcast config or remove from bindings.

Performance issues

If slow with many agents:

- Reduce number of agents per group.
- Use lighter models (sonnet instead of opus).
- Check sandbox startup time.

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#examples) Examples

Example 1: Code review team

```
{
  \"broadcast\": {
    \"strategy\": \"parallel\",
    \"120363403215116621@g.us\": [\\\
      \"code-formatter\",\\\
      \"security-scanner\",\\\
      \"test-coverage\",\\\
      \"docs-checker\"\\\
    ]
  },
  \"agents\": {
    \"list\": [\\\
      {\\\\n        \"id\": \"code-formatter\",\\\
        \"workspace\": \"~/agents/formatter\",\\\
        \"tools\": { \"allow\": [\"read\", \"write\"] }\\\
      },\\\n      {\\\\n        \"id\": \"security-scanner\",\\\
        \"workspace\": \"~/agents/security\",\\\
        \"tools\": { \"allow\": [\"read\", \"exec\"] }\\\
      },\\\n      {\\\\n        \"id\": \"test-coverage\",\\\
        \"workspace\": \"~/agents/testing\",\\\
        \"tools\": { \"allow\": [\"read\", \"exec\"] }\\\
      },\\\n      { \"id\": \"docs-checker\", \"workspace\": \"~/agents/docs\", \"tools\": { \"allow\": [\"read\"] } }\\\
    ]
  }
}
```

**User sends:** Code snippet.**Responses:**

- code-formatter: “Fixed indentation and added type hints”
- security-scanner: “⚠️ SQL injection vulnerability in line 12”
- test-coverage: “Coverage is 45%, missing tests for error cases”
- docs-checker: “Missing docstring for function `process_data`”

Example 2: Multi-language support

```
{
  \"broadcast\": {
    \"strategy\": \"sequential\",
    \"+15555550123\": [\"detect-language\", \"translator-en\", \"translator-de\"]
  },
  \"agents\": {
    \"list\": [\\\
      { \"id\": \"detect-language\", \"workspace\": \"~/agents/lang-detect\" },\\\
      { \"id\": \"translator-en\", \"workspace\": \"~/agents/translate-en\" },\\\
      { \"id\": \"translator-de\", \"workspace\": \"~/agents/translate-de\" }\\\
    ]
  }
}
```

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#api-reference) API reference

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#config-schema) Config schema

```
interface OpenClawConfig {
  broadcast?: {
    strategy?: \"parallel\" | \"sequential\";
    [peerId: string]: string[];
  };
}
```

### [​](https://docs.openclaw.ai/channels/broadcast-groups\\#fields) Fields

[​](https://docs.openclaw.ai/channels/broadcast-groups#param-strategy)

strategy

\"parallel\" \\| \"sequential\"

default:\"\\\\\"parallel\\\\\"\"

How to process agents. `parallel` runs all agents simultaneously; `sequential` runs them in array order.

[​](https://docs.openclaw.ai/channels/broadcast-groups#param-peer-id)

\\[peerId\\]

string\\[\\]

WhatsApp group JID, E.164 number, or other peer ID. Value is the array of agent IDs that should process messages.

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#limitations) Limitations

1. **Max agents:** No hard limit, but 10+ agents may be slow.
2. **Shared context:** Agents don’t see each other’s responses (by design).
3. **Message ordering:** Parallel responses may arrive in any order.
4. **Rate limits:** All agents count toward WhatsApp rate limits.

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#future-enhancements) Future enhancements

Planned features:

- [ ] Shared context mode (agents see each other’s responses)
- [ ] Agent coordination (agents can signal each other)
- [ ] Dynamic agent selection (choose agents based on message content)
- [ ] Agent priorities (some agents respond before others)

## [​](https://docs.openclaw.ai/channels/broadcast-groups\\#related) Related

- [Channel routing](https://docs.openclaw.ai/channels/channel-routing)
- [Groups](https://docs.openclaw.ai/channels/groups)
- [Multi-agent sandbox tools](https://docs.openclaw.ai/tools/multi-agent-sandbox-tools)
- [Pairing](https://docs.openclaw.ai/channels/pairing)
- [Session management](https://docs.openclaw.ai/concepts/session)

[Groups](https://docs.openclaw.ai/channels/groups) [Channel routing](https://docs.openclaw.ai/channels/channel-routing)

Ctrl+I

---

## WhatsApp
**Source:** https://docs.openclaw.ai/channels/whatsapp

[Skip to main content](https://docs.openclaw.ai/channels/whatsapp#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Mainstream messaging

WhatsApp

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Install (on demand)](https://docs.openclaw.ai/channels/whatsapp#install-on-demand)
- [Quick setup](https://docs.openclaw.ai/channels/whatsapp#quick-setup)
- [Deployment patterns](https://docs.openclaw.ai/channels/whatsapp#deployment-patterns)
- [Runtime model](https://docs.openclaw.ai/channels/whatsapp#runtime-model)
- [Plugin hooks and privacy](https://docs.openclaw.ai/channels/whatsapp#plugin-hooks-and-privacy)
- [Access control and activation](https://docs.openclaw.ai/channels/whatsapp#access-control-and-activation)
- [Personal-number and self-chat behavior](https://docs.openclaw.ai/channels/whatsapp#personal-number-and-self-chat-behavior)
- [Message normalization and context](https://docs.openclaw.ai/channels/whatsapp#message-normalization-and-context)
- [Delivery, chunking, and media](https://docs.openclaw.ai/channels/whatsapp#delivery-chunking-and-media)
- [Reply quoting](https://docs.openclaw.ai/channels/whatsapp#reply-quoting)
- [Reaction level](https://docs.openclaw.ai/channels/whatsapp#reaction-level)
- [Acknowledgment reactions](https://docs.openclaw.ai/channels/whatsapp#acknowledgment-reactions)
- [Multi-account and credentials](https://docs.openclaw.ai/channels/whatsapp#multi-account-and-credentials)
- [Tools, actions, and config writes](https://docs.openclaw.ai/channels/whatsapp#tools-actions-and-config-writes)
- [Troubleshooting](https://docs.openclaw.ai/channels/whatsapp#troubleshooting)
- [System prompts](https://docs.openclaw.ai/channels/whatsapp#system-prompts)
- [Configuration reference pointers](https://docs.openclaw.ai/channels/whatsapp#configuration-reference-pointers)
- [Related](https://docs.openclaw.ai/channels/whatsapp#related)

Status: production-ready via WhatsApp Web (Baileys). Gateway owns linked session(s).

## [​](https://docs.openclaw.ai/channels/whatsapp\\#install-on-demand)  Install (on demand)

- Onboarding (`openclaw onboard`) and `openclaw channels add --channel whatsapp`
prompt to install the WhatsApp plugin the first time you select it.
- `openclaw channels login --channel whatsapp` also offers the install flow when
the plugin is not present yet.
- Dev channel + git checkout: defaults to the local plugin path.
- Stable/Beta: defaults to the npm package `@openclaw/whatsapp`.

Manual install stays available:

```
openclaw plugins install @openclaw/whatsapp
```

[**Pairing** \\\\\n\\\\\nDefault DM policy is pairing for unknown senders.](https://docs.openclaw.ai/channels/pairing)

[**Channel troubleshooting** \\\\\n\\\\\nCross-channel diagnostics and repair playbooks.](https://docs.openclaw.ai/channels/troubleshooting)

[**Gateway configuration** \\\\\n\\\\\nFull channel config patterns and examples.](https://docs.openclaw.ai/gateway/configuration)

## [​](https://docs.openclaw.ai/channels/whatsapp\\#quick-setup)  Quick setup

1

[Navigate to header](https://docs.openclaw.ai/channels/whatsapp#)

Configure WhatsApp access policy

```
{
  channels: {
    whatsapp: {
      dmPolicy: \"pairing\",
      allowFrom: [\"+15551234567\"],
      groupPolicy: \"allowlist\",
      groupAllowFrom: [\"+15551234567\"],
    },
  },
}
```

2

[Navigate to header](https://docs.openclaw.ai/channels/whatsapp#)

Link WhatsApp (QR)

```
openclaw channels login --channel whatsapp
```

For a specific account:

```
openclaw channels login --channel whatsapp --account work
```

To attach an existing/custom WhatsApp Web auth directory before login:

```
openclaw channels add --channel whatsapp --account work --auth-dir /path/to/wa-auth
openclaw channels login --channel whatsapp --account work
```

3

[Navigate to header](https://docs.openclaw.ai/channels/whatsapp#)

Start the gateway

```
openclaw gateway
```

4

[Navigate to header](https://docs.openclaw.ai/channels/whatsapp#)

Approve first pairing request (if using pairing mode)

```
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

Pairing requests expire after 1 hour. Pending requests are capped at 3 per channel.

OpenClaw recommends running WhatsApp on a separate number when possible. (The channel metadata and setup flow are optimized for that setup, but personal-number setups are also supported.)

## [​](https://docs.openclaw.ai/channels/whatsapp\\#deployment-patterns)  Deployment patterns

Dedicated number (recommended)

This is the cleanest operational mode:

- separate WhatsApp identity for OpenClaw
- clearer DM allowlists and routing boundaries
- lower chance of self-chat confusion

Minimal policy pattern:

```
{
  channels: {
    whatsapp: {
      dmPolicy: \"allowlist\",
      allowFrom: [\"+15551234567\"],
    },
  },
}
```

Personal-number fallback

Onboarding supports personal-number mode and writes a self-chat-friendly baseline:

- `dmPolicy: \"allowlist\"`
- `allowFrom` includes your personal number
- `selfChatMode: true`

In runtime, self-chat protections key off the linked self number and `allowFrom`.

WhatsApp Web-only channel scope

The messaging platform channel is WhatsApp Web-based (`Baileys`) in current OpenClaw channel architecture.There is no separate Twilio WhatsApp messaging channel in the built-in chat-channel registry.

## [​](https://docs.openclaw.ai/channels/whatsapp\\#runtime-model)  Runtime model

- Gateway owns the WhatsApp socket and reconnect loop.
- The reconnect watchdog uses WhatsApp Web transport activity, not only inbound app-message volume, so a quiet linked-device session is not restarted solely because nobody has sent a message recently. A longer application-silence cap still forces a reconnect if transport frames keep arriving but no application messages are handled for the watchdog window.
- Outbound sends require an active WhatsApp listener for the target account.
- Status and broadcast chats are ignored (`@status`, `@broadcast`).
- Direct chats use DM session rules (`session.dmScope`; default `main` collapses DMs to the agent main session).
- Group sessions are isolated (`agent:<agentId>:whatsapp:group:<jid>`).
- WhatsApp Web transport honors standard proxy environment variables on the gateway host (`HTTPS_PROXY`, `HTTP_PROXY`, `NO_PROXY` / lowercase variants). Prefer host-level proxy config over channel-specific WhatsApp proxy settings.
- When `messages.removeAckAfterReply` is enabled, OpenClaw clears the WhatsApp ack reaction after a visible reply is delivered.

## [​](https://docs.openclaw.ai/channels/whatsapp\\#plugin-hooks-and-privacy)  Plugin hooks and privacy

WhatsApp inbound messages can contain personal message content, phone numbers,\ngroup identifiers, sender names, and session correlation fields. For that reason,\nWhatsApp does not broadcast inbound `message_received` hook payloads to plugins\nunless you explicitly opt in:\n
```
{
  channels: {
    whatsapp: {
      pluginHooks: {
        messageReceived: true,
      },
    },
  },
}
```

You can scope the opt-in to one account:

```
{
  channels: {
    whatsapp: {
      accounts: {
        work: {
          pluginHooks: {
            messageReceived: true,
          },
        },
      },
    },
  },
}
```

Only enable this for plugins you trust to receive inbound WhatsApp message\ncontent and identifiers.

## [​](https://docs.openclaw.ai/channels/whatsapp\\#access-control-and-activation)  Access control and activation

- DM policy

- Group policy + allowlists

- Mentions + /activation


`channels.whatsapp.dmPolicy` controls direct chat access:

- `pairing` (default)
- `allowlist`
- `open` (requires `allowFrom` to include `\"*\"`)
- `disabled`

`allowFrom` accepts E.164-style numbers (normalized internally).Multi-account override: `channels.whatsapp.accounts.<id>.dmPolicy` (and `allowFrom`) take precedence over channel-level defaults for that account.Runtime behavior details:

- pairings are persisted in channel allow-store and merged with configured `allowFrom`
- if no allowlist is configured, the linked self number is allowed by default
- OpenClaw never auto-pairs outbound `fromMe` DMs (messages you send to yourself from the linked device)

Group access has two layers:

1. **Group membership allowlist** (`channels.whatsapp.groups`)   - if `groups` is omitted, all groups are eligible
   - if `groups` is present, it acts as a group allowlist (`\"*\"` allowed)
2. **Group sender policy** (`channels.whatsapp.groupPolicy` \\+ `groupAllowFrom`)   - `open`: sender allowlist bypassed
   - `allowlist`: sender must match `groupAllowFrom` (or `*`)
   - `disabled`: block all group inbound

Sender allowlist fallback:

- if `groupAllowFrom` is unset, runtime falls back to `allowFrom` when available
- sender allowlists are evaluated before mention/reply activation

Note: if no `channels.whatsapp` block exists at all, runtime group-policy fallback is `allowlist` (with a warning log), even if `channels.defaults.groupPolicy` is set.

Group replies require mention by default.Mention detection includes:

- explicit WhatsApp mentions of the bot identity
- configured mention regex patterns (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
- inbound voice-note transcripts for authorized group messages
- implicit reply-to-bot detection (reply sender matches bot identity)

Security note:

- quote/reply only satisfies mention gating; it does **not** grant sender authorization
- with `groupPolicy: \"allowlist\"`, non-allowlisted senders are still blocked even if they reply to an allowlisted user’s message

Session-level activation command:

- `/activation mention`
- `/activation always`

`activation` updates session state (not global config). It is owner-gated.

## [​](https://docs.openclaw.ai/channels/whatsapp\\#personal-number-and-self-chat-behavior)  Personal-number and self-chat behavior

When the linked self number is also present in `allowFrom`, WhatsApp self-chat safeguards activate:

- skip read receipts for self-chat turns
- ignore mention-JID auto-trigger behavior that would otherwise ping yourself
- if `messages.responsePrefix` is unset, self-chat replies default to `[{identity.name}]` or `[openclaw]`

## [​](https://docs.openclaw.ai/channels/whatsapp\\#message-normalization-and-context)  Message normalization and context

Inbound envelope + reply context

Incoming WhatsApp messages are wrapped in the shared inbound envelope.If a quoted reply exists, context is appended in this form:

```
[Replying to <sender> id:<stanzaId>]
<quoted body or media placeholder>
[/Replying]
```

Reply metadata fields are also populated when available (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).

Media placeholders and location/contact extraction

Media-only inbound messages are normalized with placeholders such as:

- `<media:image>`
- `<media:video>`
- `<media:audio>`
- `<media:document>`
- `<media:sticker>`

Authorized group voice notes are transcribed before mention gating when the\nbody is only `<media:audio>`, so saying the bot mention in the voice note can\ntrigger the reply. If the transcript still does not mention the bot, the\ntranscript is kept in pending group history instead of the raw placeholder.Location bodies use terse coordinate text. Location labels/comments and contact/vCard details are rendered as fenced untrusted metadata, not inline prompt text.

Pending group history injection

For groups, unprocessed messages can be buffered and injected as context when the bot is finally triggered.

- default limit: `50`
- config: `channels.whatsapp.historyLimit`
- fallback: `messages.groupChat.historyLimit`
- `0` disables

Injection markers:

- `[Chat messages since your last reply - for context]`
- `[Current message - respond to this]`

Read receipts

Read receipts are enabled by default for accepted inbound WhatsApp messages.Disable globally:

```
{
  channels: {
    whatsapp: {
      sendReadReceipts: false,
    },
  },
}
```

Per-account override:

```
{
  channels: {
    whatsapp: {
      accounts: {
        work: {
          sendReadReceipts: false,
        },
      },
    },
  },
}
```

Self-chat turns skip read receipts even when globally enabled.

## [​](https://docs.openclaw.ai/channels/whatsapp\\#delivery-chunking-and-media)  Delivery, chunking, and media

Text chunking

- default chunk limit: `channels.whatsapp.textChunkLimit = 4000`
- `channels.whatsapp.chunkMode = \"length\" | \"newline\"`
- `newline` mode prefers paragraph boundaries (blank lines), then falls back to length-safe chunking

Outbound media behavior

- supports image, video, audio (PTT voice-note), and document payloads
- audio media is sent through the Baileys `audio` payload with `ptt: true`, so WhatsApp clients render it as a push-to-talk voice note
- reply payloads preserve `audioAsVoice`; TTS voice-note output for WhatsApp stays on this PTT path even when the provider returns MP3 or WebM
- native Ogg/Opus audio is sent as `audio/ogg; codecs=opus` for voice-note compatibility
- non-Ogg audio, including Microsoft Edge TTS MP3/WebM output, is transcoded with `ffmpeg` to 48 kHz mono Ogg/Opus before PTT delivery
- `/tts latest` sends the latest assistant reply as one voice note and suppresses repeat sends for the same reply; `/tts chat on|off|default` controls auto-TTS for the current WhatsApp chat
- animated GIF playback is supported via `gifPlayback: true` on video sends
- captions are applied to the first media item when sending multi-media reply payloads, except PTT voice notes send the audio first and visible text separately because WhatsApp clients do not render voice-note captions consistently
- media source can be HTTP(S), `file://`, or local paths

Media size limits and fallback behavior

- inbound media save cap: `channels.whatsapp.mediaMaxMb` (default `50`)
- outbound media send cap: `channels.whatsapp.mediaMaxMb` (default `50`)
- per-account overrides use `channels.whatsapp.accounts.<accountId>.mediaMaxMb`
- images are auto-optimized (resize/quality sweep) to fit limits
- on media send failure, first-item fallback sends text warning instead of dropping the response silently

## [​](https://docs.openclaw.ai/channels/whatsapp\\#reply-quoting)  Reply quoting

WhatsApp supports native reply quoting, where outbound replies visibly quote the inbound message. Control it with `channels.whatsapp.replyToMode`.

| Value | Behavior |
| --- | --- |
| `\"off\"` | Never quote; send as a plain message |
| `\"first\"` | Quote only the first outbound reply chunk |
| `\"all\"` | Quote every outbound reply chunk |
| `\"batched\"` | Quote queued batched replies while leaving immediate replies unquoted |

Default is `\"off\"`. Per-account overrides use `channels.whatsapp.accounts.<id>.replyToMode`.

```
{
  channels: {
    whatsapp: {
      replyToMode: \"first\",
    },
  },
}
```

## [​](https://docs.openclaw.ai/channels/whatsapp\\#reaction-level)  Reaction level

`channels.whatsapp.reactionLevel` controls how broadly the agent uses emoji reactions on WhatsApp:

| Level | Ack reactions | Agent-initiated reactions | Description |
| --- | --- | --- | --- |
| `\"off\"` | No | No | No reactions at all |
| `\"ack\"` | Yes | No | Ack reactions only (pre-reply receipt) |
| `\"minimal\"` | Yes | Yes (conservative) | Ack + agent reactions with conservative guidance |
| `\"extensive\"` | Yes | Yes (encouraged) | Ack + agent reactions with encouraged guidance |

Default: `\"minimal\"`.Per-account overrides use `channels.whatsapp.accounts.<id>.reactionLevel`.

```
{
  channels: {
    whatsapp: {
      reactionLevel: \"ack\",
    },
  },
}
```

## [​](https://docs.openclaw.ai/channels/whatsapp\\#acknowledgment-reactions)  Acknowledgment reactions

WhatsApp supports immediate ack reactions on inbound receipt via `channels.whatsapp.ackReaction`.
Ack reactions are gated by `reactionLevel` — they are suppressed when `reactionLevel` is `\"off\"`.

```
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: \"👀\",
        direct: true,
        group: \"mentions\", // always | mentions | never
      },
    },
  },
}
```

Behavior notes:

- sent immediately after inbound is accepted (pre-reply)
- failures are logged but do not block normal reply delivery
- group mode `mentions` reacts on mention-triggered turns; group activation `always` acts as bypass for this check
- WhatsApp uses `channels.whatsapp.ackReaction` (legacy `messages.ackReaction` is not used here)

## [​](https://docs.openclaw.ai/channels/whatsapp\\#multi-account-and-credentials)  Multi-account and credentials

Account selection and defaults

- account ids come from `channels.whatsapp.accounts`
- default account selection: `default` if present, otherwise first configured account id (sorted)
- account ids are normalized internally for lookup

Credential paths and legacy compatibility

- current auth path: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
- backup file: `creds.json.bak`
- legacy default auth in `~/.openclaw/credentials/` is still recognized/migrated for default-account flows

Logout behavior

`openclaw channels logout --channel whatsapp [--account <id>]` clears WhatsApp auth state for that account.In legacy auth directories, `oauth.json` is preserved while Baileys auth files are removed.

## [​](https://docs.openclaw.ai/channels/whatsapp\\#tools-actions-and-config-writes)  Tools, actions, and config writes

- Agent tool support includes WhatsApp reaction action (`react`).
- Action gates:
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- Channel-initiated config writes are enabled by default (disable via `channels.whatsapp.configWrites=false`).

## [​](https://docs.openclaw.ai/channels/whatsapp\\#troubleshooting)  Troubleshooting

Not linked (QR required)

Symptom: channel status reports not linked.Fix:

```
openclaw channels login --channel whatsapp
openclaw channels status
```

Linked but disconnected / reconnect loop

Symptom: linked account with repeated disconnects or reconnect attempts.Quiet accounts can stay connected past the normal message timeout; the watchdog\nrestarts when WhatsApp Web transport activity stops, the socket closes, or\napplication-level activity stays silent beyond the longer safety window.Fi...(content truncated)

---

## WeChat - OpenClaw
**Source:** https://docs.openclaw.ai/channels/wechat

[Skip to main content](https://docs.openclaw.ai/channels/wechat#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Regional platforms

WeChat

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Naming](https://docs.openclaw.ai/channels/wechat#naming)
- [How it works](https://docs.openclaw.ai/channels/wechat#how-it-works)
- [Install](https://docs.openclaw.ai/channels/wechat#install)
- [Login](https://docs.openclaw.ai/channels/wechat#login)
- [Access control](https://docs.openclaw.ai/channels/wechat#access-control)
- [Compatibility](https://docs.openclaw.ai/channels/wechat#compatibility)
- [Sidecar process](https://docs.openclaw.ai/channels/wechat#sidecar-process)
- [Troubleshooting](https://docs.openclaw.ai/channels/wechat#troubleshooting)
- [Related docs](https://docs.openclaw.ai/channels/wechat#related-docs)

OpenClaw connects to WeChat through Tencent’s external
`@tencent-weixin/openclaw-weixin` channel plugin.Status: external plugin. Direct chats and media are supported. Group chats are not
advertised by the current plugin capability metadata.

## [​](https://docs.openclaw.ai/channels/wechat\\#naming)  Naming

- **WeChat** is the user-facing name in these docs.
- **Weixin** is the name used by Tencent’s package and by the plugin id.
- `openclaw-weixin` is the OpenClaw channel id.
- `@tencent-weixin/openclaw-weixin` is the npm package.

Use `openclaw-weixin` in CLI commands and config paths.

## [​](https://docs.openclaw.ai/channels/wechat\\#how-it-works)  How it works

The WeChat code does not live in the OpenClaw core repo. OpenClaw provides the
generic channel plugin contract, and the external plugin provides the
WeChat-specific runtime:

1. `openclaw plugins install` installs `@tencent-weixin/openclaw-weixin`.
2. The Gateway discovers the plugin manifest and loads the plugin entrypoint.
3. The plugin registers channel id `openclaw-weixin`.
4. `openclaw channels login --channel openclaw-weixin` starts QR login.
5. The plugin stores account credentials under the OpenClaw state directory.
6. When the Gateway starts, the plugin starts its Weixin monitor for each
configured account.
7. Inbound WeChat messages are normalized through the channel contract, routed to
the selected OpenClaw agent, and sent back through the plugin outbound path.

That separation matters: OpenClaw core should stay channel-agnostic. WeChat login,
Tencent iLink API calls, media upload/download, context tokens, and account
monitoring are owned by the external plugin.

## [​](https://docs.openclaw.ai/channels/wechat\\#install)  Install

Quick install:

```
npx -y @tencent-weixin/openclaw-weixin-cli install
```

Manual install:

```
openclaw plugins install \"@tencent-weixin/openclaw-weixin\"
openclaw config set plugins.entries.openclaw-weixin.enabled true
```

Restart the Gateway after install:

```
openclaw gateway restart
```

## [​](https://docs.openclaw.ai/channels/wechat\\#login)  Login

Run QR login on the same machine that runs the Gateway:

```
openclaw channels login --channel openclaw-weixin
```

Scan the QR code with WeChat on your phone and confirm the login. The plugin saves
the account token locally after a successful scan.To add another WeChat account, run the same login command again. For multiple
accounts, isolate direct-message sessions by account, channel, and sender:

```
openclaw config set session.dmScope per-account-channel-peer
```

## [​](https://docs.openclaw.ai/channels/wechat\\#access-control)  Access control

Direct messages use the normal OpenClaw pairing and allowlist model for channel
plugins.Approve new senders:

```
openclaw pairing list openclaw-weixin
openclaw pairing approve openclaw-weixin <CODE>
```

For the full access-control model, see [Pairing](https://docs.openclaw.ai/channels/pairing).

## [​](https://docs.openclaw.ai/channels/wechat\\#compatibility)  Compatibility

The plugin checks the host OpenClaw version at startup.

| Plugin line | OpenClaw version | npm tag |
| --- | --- | --- |
| `2.x` | `>=2026.3.22` | `latest` |
| `1.x` | `>=2026.1.0 <2026.3.22` | `legacy` |

If the plugin reports that your OpenClaw version is too old, either update
OpenClaw or install the legacy plugin line:

```
openclaw plugins install @tencent-weixin/openclaw-weixin@legacy
```

## [​](https://docs.openclaw.ai/channels/wechat\\#sidecar-process)  Sidecar process

The WeChat plugin can run helper work beside the Gateway while it monitors the
Tencent iLink API. In issue #68451, that helper path exposed a bug in OpenClaw’s
generic stale-Gateway cleanup: a child process could try to clean up the parent
Gateway process, causing restart loops under process managers such as systemd.Current OpenClaw startup cleanup excludes the current process and its ancestors,
so a channel helper must not kill the Gateway that launched it. This fix is
generic; it is not a WeChat-specific path in core.

## [​](https://docs.openclaw.ai/channels/wechat\\#troubleshooting)  Troubleshooting

Check install and status:

```
openclaw plugins list
openclaw channels status --probe
openclaw --version
```

If the channel shows as installed but does not connect, confirm that the plugin is
enabled and restart:

```
openclaw config set plugins.entries.openclaw-weixin.enabled true
openclaw gateway restart
```

If the Gateway restarts repeatedly after enabling WeChat, update both OpenClaw and
the plugin:

```
npm view @tencent-weixin/openclaw-weixin version
openclaw plugins install \"@tencent-weixin/openclaw-weixin\" --force
openclaw gateway restart
```

Temporary disable:

```
openclaw config set plugins.entries.openclaw-weixin.enabled false
openclaw gateway restart
```

## [​](https://docs.openclaw.ai/channels/wechat\\#related-docs)  Related docs

- Channel overview: [Chat Channels](https://docs.openclaw.ai/channels)
- Pairing: [Pairing](https://docs.openclaw.ai/channels/pairing)
- Channel routing: [Channel Routing](https://docs.openclaw.ai/channels/channel-routing)
- Plugin architecture: [Plugin Architecture](https://docs.openclaw.ai/plugins/architecture)
- Channel plugin SDK: [Channel Plugin SDK](https://docs.openclaw.ai/plugins/sdk-channel-plugins)
- External package: [@tencent-weixin/openclaw-weixin](https://www.npmjs.com/package/@tencent-weixin/openclaw-weixin)

[LINE](https://docs.openclaw.ai/channels/line) [QQ bot](https://docs.openclaw.ai/channels/qqbot)

Ctrl+I

---

## BlueBubbles
**Source:** https://docs.openclaw.ai/channels/bluebubbles

[Skip to main content](https://docs.openclaw.ai/channels/bluebubbles#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Mainstream messaging

BlueBubbles

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Bundled plugin](https://docs.openclaw.ai/channels/bluebubbles#bundled-plugin)
- [Overview](https://docs.openclaw.ai/channels/bluebubbles#overview)
- [Quick start](https://docs.openclaw.ai/channels/bluebubbles#quick-start)
- [Keeping Messages.app alive (VM / headless setups)](https://docs.openclaw.ai/channels/bluebubbles#keeping-messages-app-alive-vm-%2F-headless-setups)
- [1) Save the AppleScript](https://docs.openclaw.ai/channels/bluebubbles#1-save-the-applescript)
- [2) Install a LaunchAgent](https://docs.openclaw.ai/channels/bluebubbles#2-install-a-launchagent)
- [Onboarding](https://docs.openclaw.ai/channels/bluebubbles#onboarding)
- [Access control (DMs + groups)](https://docs.openclaw.ai/channels/bluebubbles#access-control-dms-%2B-groups)
- [Contact name enrichment (macOS, optional)](https://docs.openclaw.ai/channels/bluebubbles#contact-name-enrichment-macos-optional)
- [Mention gating (groups)](https://docs.openclaw.ai/channels/bluebubbles#mention-gating-groups)
- [Command gating](https://docs.openclaw.ai/channels/bluebubbles#command-gating)
- [Per-group system prompt](https://docs.openclaw.ai/channels/bluebubbles#per-group-system-prompt)
- [Worked example: threaded replies and tapback reactions (Private API)](https://docs.openclaw.ai/channels/bluebubbles#worked-example-threaded-replies-and-tapback-reactions-private-api)
- [ACP conversation bindings](https://docs.openclaw.ai/channels/bluebubbles#acp-conversation-bindings)
- [Typing + read receipts](https://docs.openclaw.ai/channels/bluebubbles#typing-%2B-read-receipts)
- [Advanced actions](https://docs.openclaw.ai/channels/bluebubbles#advanced-actions)
- [Message IDs (short vs full)](https://docs.openclaw.ai/channels/bluebubbles#message-ids-short-vs-full)
- [Coalescing split-send DMs (command + URL in one composition)](https://docs.openclaw.ai/channels/bluebubbles#coalescing-split-send-dms-command-%2B-url-in-one-composition)
- [When to enable](https://docs.openclaw.ai/channels/bluebubbles#when-to-enable)
- [Enabling](https://docs.openclaw.ai/channels/bluebubbles#enabling)
- [Trade-offs](https://docs.openclaw.ai/channels/bluebubbles#trade-offs)
- [Scenarios and what the agent sees](https://docs.openclaw.ai/channels/bluebubbles#scenarios-and-what-the-agent-sees)
- [Split-send coalescing troubleshooting](https://docs.openclaw.ai/channels/bluebubbles#split-send-coalescing-troubleshooting)
- [Block streaming](https://docs.openclaw.ai/channels/bluebubbles#block-streaming)
- [Media + limits](https://docs.openclaw.ai/channels/bluebubbles#media-%2B-limits)
- [Configuration reference](https://docs.openclaw.ai/channels/bluebubbles#configuration-reference)
- [Addressing / delivery targets](https://docs.openclaw.ai/channels/bluebubbles#addressing-%2F-delivery-targets)
- [iMessage vs SMS routing](https://docs.openclaw.ai/channels/bluebubbles#imessage-vs-sms-routing)
- [Security](https://docs.openclaw.ai/channels/bluebubbles#security)
- [Troubleshooting](https://docs.openclaw.ai/channels/bluebubbles#troubleshooting)
- [Related](https://docs.openclaw.ai/channels/bluebubbles#related)

Status: bundled plugin that talks to the BlueBubbles macOS server over HTTP. **Recommended for iMessage integration** due to its richer API and easier setup compared to the legacy imsg channel.

## [​](https://docs.openclaw.ai/channels/bluebubbles\\#bundled-plugin)  Bundled plugin

Current OpenClaw releases bundle BlueBubbles, so normal packaged builds do not
need a separate `openclaw plugins install` step.

## [​](https://docs.openclaw.ai/channels/bluebubbles\\#overview)  Overview

- Runs on macOS via the BlueBubbles helper app ( [bluebubbles.app](https://bluebubbles.app/)).
- Recommended/tested: macOS Sequoia (15). macOS Tahoe (26) works; edit is currently broken on Tahoe, and group icon updates may report success but not sync.
- OpenClaw talks to it through its REST API (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
- Incoming messages arrive via webhooks; outgoing replies, typing indicators, read receipts, and tapbacks are REST calls.
- Attachments and stickers are ingested as inbound media (and surfaced to the agent when possible).
- Auto-TTS replies that synthesize MP3 or CAF audio are delivered as iMessage
voice memo bubbles instead of plain file attachments.
- Pairing/allowlist works the same way as other channels (`/channels/pairing` etc) with `channels.bluebubbles.allowFrom` \\+ pairing codes.
- Reactions are surfaced as system events just like Slack/Telegram so agents can “mention” them before replying.
- Advanced features: edit, unsend, reply threading, message effects, group management.

## [​](https://docs.openclaw.ai/channels/bluebubbles\\#quick-start)  Quick start

1. Install the BlueBubbles server on your Mac (follow the instructions at [bluebubbles.app/install](https://bluebubbles.app/install)).
2. In the BlueBubbles config, enable the web API and set a password.
3. Run `openclaw onboard` and select BlueBubbles, or configure manually:














```
{
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: \"http://192.168.1.100:1234\",
         password: \"example-password\",
         webhookPath: \"/bluebubbles-webhook\",
       },
     },
}
```

4. Point BlueBubbles webhooks to your gateway (example: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`).
5. Start the gateway; it will register the webhook handler and start pairing.

Security note:

- Always set a webhook password.
- Webhook authentication is always required. OpenClaw rejects BlueBubbles webhook requests unless they include a password/guid that matches `channels.bluebubbles.password` (for example `?password=<password>` or `x-password`), regardless of loopback/proxy topology.
- Password authentication is checked before reading/parsing full webhook bodies.

## [​](https://docs.openclaw.ai/channels/bluebubbles\\#keeping-messages-app-alive-vm-/-headless-setups)  Keeping Messages.app alive (VM / headless setups)

Some macOS VM / always-on setups can end up with Messages.app going “idle” (incoming events stop until the app is opened/foregrounded). A simple workaround is to **poke Messages every 5 minutes** using an AppleScript + LaunchAgent.

### [​](https://docs.openclaw.ai/channels/bluebubbles\\#1-save-the-applescript)  1) Save the AppleScript

Save this as:

- `~/Scripts/poke-messages.scpt`

Example script (non-interactive; does not steal focus):

```
try
  tell application \"Messages\"
    if not running then
      launch
    end if

    -- Touch the scripting interface to keep the process responsive.
    set _chatCount to (count of chats)
  end tell
on error
  -- Ignore transient failures (first-run prompts, locked session, etc).
end try
```

### [​](https://docs.openclaw.ai/channels/bluebubbles\\#2-install-a-launchagent)  2) Install a LaunchAgent

Save this as:

- `~/Library/LaunchAgents/com.user.poke-messages.plist`

```
<?xml version=\"1.0\" encoding=\"UTF-8\"?>
<!DOCTYPE plist PUBLIC \"-//Apple//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/PropertyList-1.0.dtd\">
<plist version=\"1.0\">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript &quot;$HOME/Scripts/poke-messages.scpt&quot;</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>StartInterval</key>
    <integer>300</integer>

    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```

Notes:

- This runs **every 300 seconds** and **on login**.
- The first run may trigger macOS **Automation** prompts (`osascript` → Messages). Approve them in the same user session that runs the LaunchAgent.

Load it:

```
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

## [​](https://docs.openclaw.ai/channels/bluebubbles\\#onboarding)  Onboarding

BlueBubbles is available in interactive onboarding:

```
openclaw onboard
```

The wizard prompts for:

- **Server URL** (required): BlueBubbles server address (e.g., `http://192.168.1.100:1234`)
- **Password** (required): API password from BlueBubbles Server settings
- **Webhook path** (optional): Defaults to `/bluebubbles-webhook`
- **DM policy**: pairing, allowlist, open, or disabled
- **Allow list**: Phone numbers, emails, or chat targets

You can also add BlueBubbles via CLI:

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## [​](https://docs.openclaw.ai/channels/bluebubbles\\#access-control-dms-+-groups)  Access control (DMs + groups)

DMs:

- Default: `channels.bluebubbles.dmPolicy = \"pairing\"`.
- Unknown senders receive a pairing code; messages are ignored until approved (codes expire after 1 hour).
- Approve via:
  - `openclaw pairing list bluebubbles`
  - `openclaw pairing approve bluebubbles <CODE>`
- Pairing is the default token exchange. Details: [Pairing](https://docs.openclaw.ai/channels/pairing)

Groups:

- `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (default: `allowlist`).
- `channels.bluebubbles.groupAllowFrom` controls who can trigger in groups when `allowlist` is set.

### [​](https://docs.openclaw.ai/channels/bluebubbles\\#contact-name-enrichment-macos-optional)  Contact name enrichment (macOS, optional)

BlueBubbles group webhooks often only include raw participant addresses. If you want `GroupMembers` context to show local contact names instead, you can opt in to local Contacts enrichment on macOS:

- `channels.bluebubbles.enrichGroupParticipantsFromContacts = true` enables the lookup. Default: `false`.
- Lookups run only after group access, command authorization, and mention gating have allowed the message through.
- Only unnamed phone participants are enriched.
- Raw phone numbers remain as the fallback when no local match is found.

```
{
  channels: {
    bluebubbles: {
      enrichGroupParticipantsFromContacts: true,
    },
  },
}
```

### [​](https://docs.openclaw.ai/channels/bluebubbles\\#mention-gating-groups)  Mention gating (groups)

BlueBubbles supports mention gating for group chats, matching iMessage/WhatsApp behavior:

- Uses `agents.list[].groupChat.mentionPatterns` (or `messages.groupChat.mentionPatterns`) to detect mentions.
- When `requireMention` is enabled for a group, the agent only responds when mentioned.
- Control commands from authorized senders bypass mention gating.

Per-group configuration:

```
{
  channels: {
    bluebubbles: {
      groupPolicy: \"allowlist\",
      groupAllowFrom: [\"+15555550123\"],
      groups: {
        \"*\": { requireMention: true }, // default for all groups
        \"iMessage;-;chat123\": { requireMention: false }, // override for specific group
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/channels/bluebubbles\\#command-gating)  Command gating

- Control commands (e.g., `/config`, `/model`) require authorization.
- Uses `allowFrom` and `groupAllowFrom` to determine command authorization.
- Authorized senders can run control commands even without mentioning in groups.

### [​](https://docs.openclaw.ai/channels/bluebubbles\\#per-group-system-prompt)  Per-group system prompt

Each entry under `channels.bluebubbles.groups.*` accepts an optional `systemPrompt` string. The value is injected into the agent’s system prompt on every turn that handles a message in that group, so you can set per-group persona or behavioral rules without editing agent prompts:

```
{
  channels: {
    bluebubbles: {
      groups: {
        \"iMessage;-;chat123\": {
          systemPrompt: \"Keep responses under 3 sentences. Mirror the group's casual tone.\",
        },
      },
    },
  },
}
```

The key matches whatever BlueBubbles reports as `chatGuid` / `chatIdentifier` / numeric `chatId` for the group, and a `\"*\"` wildcard entry provides a default for every group without an exact match (same pattern used by `requireMention` and per-group tool policies). Exact matches always win over the wildcard. DMs ignore this field; use agent-level or account-level prompt customization instead.

#### [​](https://docs.openclaw.ai/channels/bluebubbles\\#worked-example-threaded-replies-and-tapback-reactions-private-api)  Worked example: threaded replies and tapback reactions (Private API)

With the BlueBubbles Private API enabled, inbound messages arrive with short message IDs (for example `[[reply_to:5]]`) and the agent can call `action=reply` to thread into a specific message or `action=react` to drop a tapback. A per-group `systemPrompt` is a reliable way to keep the agent choosing the right tool:

```
{
  channels: {
    bluebubbles: {
      groups: {
        \"iMessage;+;chat-family\": {
          systemPrompt: [\\\n            \"When replying in this group, always call action=reply with the\",\\\n            \"[[reply_to:N]] messageId from context so your response threads\",\\\n            \"under the triggering message. Never send a new unlinked message.\",\\\n            \"\",\\\n            \"For short acknowledgements ('ok', 'got it', 'on it'), use\",\\\n            \"action=react with an appropriate tapback emoji (❤️, 👍, 😂, ‼️, ❓)\",\\\n            \"instead of sending a text reply.\",\\\n          ].join(\" \"),
        },
      },
    },
  },
}
```

Tapback reactions and threaded replies both require the BlueBubbles Private API; see [Advanced actions](https://docs.openclaw.ai/channels/bluebubbles#advanced-actions) and [Message IDs](https://docs.openclaw.ai/channels/bluebubbles#message-ids-short-vs-full) for the underlying mechanics.

## [​](https://docs.openclaw.ai/channels/bluebubbles\\#acp-conversation-bindings)  ACP conversation bindings

BlueBubbles chats can be turned into durable ACP workspaces without changing the transport layer.Fast operator flow:

- Run `/acp spawn codex --bind here` inside the DM or allowed group chat.
- Future messages in that same BlueBubbles conversation route to the spawned ACP session.
- `/new` and `/reset` reset the same bound ACP session in place.
- `/acp close` closes the ACP session and removes the binding.

Configured persistent bindings are also supported through top-level `bindings[]` entries with `type: \"acp\"` and `match.channel: \"bluebubbles\"`.`match.peer.id` can use any supported BlueBubbles target form:

- normalized DM handle such as `+15555550123` or `user@example.com`
- `chat_id:<id>`
- `chat_guid:<guid>`
- `chat_identifier:<identifier>`

For stable group bindings, prefer `chat_id:*` or `chat_identifier:*`.Example:

```
{
  agents: {
    list: [\\\n      {\\\
        id: \"codex\",\\\n        runtime: {\\\n          type: \"acp\",\\\n          acp: { agent: \"codex\", backend: \"acpx\", mode: \"persistent\" },\\\n        },\\\n      },\\\n    ],
  },
  bindings: [\\\n    {\\\
      type: \"acp\",\\\n      agentId: \"codex\",\\\n      match: {\\\n        channel: \"bluebubbles\",\\\n        accountId: \"default\",\\\n        peer: { kind: \"dm\", id: \"+15555550123\" },\\\n      },\\\n      acp: { label: \"codex-imessage\" },\\\n    },\\\n  ],
}
```

See [ACP Agents](https://docs.openclaw.ai/tools/acp-agents) for shared ACP binding behavior.

## [​](https://docs.openclaw.ai/channels/bluebubbles\\#typing-+-read-receipts)  Typing + read receipts

- **Typing indicators**: Sent automatically before and during response generation.
- **Read receipts**: Controlled by `channels.bluebubbles.sendReadReceipts` (default: `true`).
- **Typing indicators**: OpenClaw sends typing start events; BlueBubbles clears typing automatically on send or timeout (manual stop via DELETE is unreliable).

```
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // disable read receipts
    },
  },
}
```

## [​](https://docs.openclaw.ai/channels/bluebubbles\\#advanced-actions)  Advanced actions

BlueBubbles supports advanced message actions when enabled in config:

```
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // tapbacks (default: true)
        edit: true, // edit sent messages (macOS 13+, broken on macOS 26 Tahoe)
        unsend: true, // unsend messages (macOS 13+)
        reply: true, // reply threading by message GUID
        sendWithEffect: true, // message effects (slam, loud, etc.)
        renameGroup: true, // rename group chats
        setGroupIcon: true, // set group chat icon/photo (flaky on macOS 26 Tahoe)
        addParticipant: true, // add participants to groups
        removeParticipant: true, // remove participants from groups
        leaveGroup: true, // leave group chats
        sendAttachment: true, // send attachments/media
      },
    },
  },
}
```

Available actions:

- **react**: Add/remove tapback reactions (`messageId`, `emoji`, `remove`). iMessage’s native tapback set is `love`, `like`, `dislike`, `laugh`, `emphasize`, and `question`. When an agent picks an emoji outside that set (for example `👀`), the reaction tool falls back to `love` so the tapback still renders instead of failing the whole request. Configured ack reactions still validate strictly and error on unknown values.
- **edit**: Edit a sent message (`messageId`, `text`)
- **unsend**: Unsend a message (`messageId`)
- **reply**: Reply to a specific message (`messageId`, `text`, `to`)
- **sendWithEffect**: Send with iMessage effect (`text`, `to`, `effectId`)
- **renameGroup**: Rename a group chat (`chatGuid`, `displayName`)
- **setGroupIcon**: Set a group chat’s icon/photo (`chatGuid`, `media`) — flaky on macOS 26 Tahoe (API may return success but the icon does not sync).
- **addParticipant**: Add someone to a group (`chatGuid`, `address`)
- **removeParticipant**: Remove someone from a group (`chatGuid`, `address`)
- **leaveGroup**: Leave a group chat (`chatGuid`)
- **upload-file**: Send media/files (`to`, `buffer`, `filename`, `asVoice`)

  - Voice memos: set `asVoice: true` with **MP3** or **CAF** audio to send as an iMessage voice message. BlueBubbles ...(content truncated)

---

## Google Chat - OpenClaw
**Source:** https://docs.openclaw.ai/channels/googlechat

[Skip to main content](https://docs.openclaw.ai/channels/googlechat#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Mainstream messaging

Google Chat

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick setup (beginner)](https://docs.openclaw.ai/channels/googlechat#quick-setup-beginner)
- [Add to Google Chat](https://docs.openclaw.ai/channels/googlechat#add-to-google-chat)
- [Public URL (Webhook-only)](https://docs.openclaw.ai/channels/googlechat#public-url-webhook-only)
- [Option A: Tailscale Funnel (Recommended)](https://docs.openclaw.ai/channels/googlechat#option-a-tailscale-funnel-recommended)
- [Option B: Reverse Proxy (Caddy)](https://docs.openclaw.ai/channels/googlechat#option-b-reverse-proxy-caddy)
- [Option C: Cloudflare Tunnel](https://docs.openclaw.ai/channels/googlechat#option-c-cloudflare-tunnel)
- [How it works](https://docs.openclaw.ai/channels/googlechat#how-it-works)
- [Targets](https://docs.openclaw.ai/channels/googlechat#targets)
- [Config highlights](https://docs.openclaw.ai/channels/googlechat#config-highlights)
- [Troubleshooting](https://docs.openclaw.ai/channels/googlechat#troubleshooting)
- [405 Method Not Allowed](https://docs.openclaw.ai/channels/googlechat#405-method-not-allowed)
- [Other issues](https://docs.openclaw.ai/channels/googlechat#other-issues)
- [Related](https://docs.openclaw.ai/channels/googlechat#related)

Status: ready for DMs + spaces via Google Chat API webhooks (HTTP only).

## [​](https://docs.openclaw.ai/channels/googlechat\\#quick-setup-beginner)  Quick setup (beginner)

1. Create a Google Cloud project and enable the **Google Chat API**.

   - Go to: [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   - Enable the API if it is not already enabled.
2. Create a **Service Account**:

   - Press **Create Credentials** \\> **Service Account**.
   - Name it whatever you want (e.g., `openclaw-chat`).
   - Leave permissions blank (press **Continue**).
   - Leave principals with access blank (press **Done**).
3. Create and download the **JSON Key**:

   - In the list of service accounts, click on the one you just created.
   - Go to the **Keys** tab.
   - Click **Add Key** \\> **Create new key**.
   - Select **JSON** and press **Create**.
4. Store the downloaded JSON file on your gateway host (e.g., `~/.openclaw/googlechat-service-account.json`).
5. Create a Google Chat app in the [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat):

   - Fill in the **Application info**:

     - **App name**: (e.g. `OpenClaw`)
     - **Avatar URL**: (e.g. `https://openclaw.ai/logo.png`)
     - **Description**: (e.g. `Personal AI Assistant`)
   - Enable **Interactive features**.
   - Under **Functionality**, check **Join spaces and group conversations**.
   - Under **Connection settings**, select **HTTP endpoint URL**.
   - Under **Triggers**, select **Use a common HTTP endpoint URL for all triggers** and set it to your gateway’s public URL followed by `/googlechat`.

     - _Tip: Run `openclaw status` to find your gateway’s public URL._
   - Under **Visibility**, check **Make this Chat app available to specific people and groups in `<Your Domain>`**.
   - Enter your email address (e.g., `user@example.com`) in the text box.
   - Click **Save** at the bottom.
6. **Enable the app status**:

   - After saving, **refresh the page**.
   - Look for the **App status** section (usually near the top or bottom after saving).
   - Change the status to **Live - available to users**.
   - Click **Save** again.
7. Configure OpenClaw with the service account path + webhook audience:
   - Env: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   - Or config: `channels.googlechat.serviceAccountFile: \"/path/to/service-account.json\"`.
8. Set the webhook audience type + value (matches your Chat app config).
9. Start the gateway. Google Chat will POST to your webhook path.

## [​](https://docs.openclaw.ai/channels/googlechat\\#add-to-google-chat)  Add to Google Chat

Once the gateway is running and your email is added to the visibility list:

1. Go to [Google Chat](https://chat.google.com/).
2. Click the **+** (plus) icon next to **Direct Messages**.
3. In the search bar (where you usually add people), type the **App name** you configured in the Google Cloud Console.

   - **Note**: The bot will _not_ appear in the “Marketplace” browse list because it is a private app. You must search for it by name.
4. Select your bot from the results.
5. Click **Add** or **Chat** to start a 1:1 conversation.
6. Send “Hello” to trigger the assistant!

## [​](https://docs.openclaw.ai/channels/googlechat\\#public-url-webhook-only)  Public URL (Webhook-only)

Google Chat webhooks require a public HTTPS endpoint. For security, **only expose the `/googlechat` path** to the internet. Keep the OpenClaw dashboard and other sensitive endpoints on your private network.

### [​](https://docs.openclaw.ai/channels/googlechat\\#option-a-tailscale-funnel-recommended)  Option A: Tailscale Funnel (Recommended)

Use Tailscale Serve for the private dashboard and Funnel for the public webhook path. This keeps `/` private while exposing only `/googlechat`.

1. **Check what address your gateway is bound to:**














```
ss -tlnp | grep 18789
```











Note the IP address (e.g., `127.0.0.1`, `0.0.0.0`, or your Tailscale IP like `100.x.x.x`).
2. **Expose the dashboard to the tailnet only (port 8443):**














```
# If bound to localhost (127.0.0.1 or 0.0.0.0):
tailscale serve --bg --https 8443 http://127.0.0.1:18789

# If bound to Tailscale IP only (e.g., 100.106.161.80):
tailscale serve --bg --https 8443 http://100.106.161.80:18789
```

3. **Expose only the webhook path publicly:**














```
# If bound to localhost (127.0.0.1 or 0.0.0.0):
tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

# If bound to Tailscale IP only (e.g., 100.106.161.80):
tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
```

4. **Authorize the node for Funnel access:**
If prompted, visit the authorization URL shown in the output to enable Funnel for this node in your tailnet policy.
5. **Verify the configuration:**














```
tailscale serve status
tailscale funnel status
```


Your public webhook URL will be:
`https://<node-name>.<tailnet>.ts.net/googlechat`Your private dashboard stays tailnet-only:
`https://<node-name>.<tailnet>.ts.net:8443/`Use the public URL (without `:8443`) in the Google Chat app config.

> Note: This configuration persists across reboots. To remove it later, run `tailscale funnel reset` and `tailscale serve reset`.

### [​](https://docs.openclaw.ai/channels/googlechat\\#option-b-reverse-proxy-caddy)  Option B: Reverse Proxy (Caddy)

If you use a reverse proxy like Caddy, only proxy the specific path:

```
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

With this config, any request to `your-domain.com/` will be ignored or returned as 404, while `your-domain.com/googlechat` is safely routed to OpenClaw.

### [​](https://docs.openclaw.ai/channels/googlechat\\#option-c-cloudflare-tunnel)  Option C: Cloudflare Tunnel

Configure your tunnel’s ingress rules to only route the webhook path:

- **Path**: `/googlechat` -\\> `http://localhost:18789/googlechat`
- **Default Rule**: HTTP 404 (Not Found)

## [​](https://docs.openclaw.ai/channels/googlechat\\#how-it-works)  How it works

1. Google Chat sends webhook POSTs to the gateway. Each request includes an `Authorization: Bearer <token>` header.

   - OpenClaw verifies bearer auth before reading/parsing full webhook bodies when the header is present.
   - Google Workspace Add-on requests that carry `authorizationEventObject.systemIdToken` in the body are supported via a stricter pre-auth body budget.
2. OpenClaw verifies the token against the configured `audienceType` \\+ `audience`:

   - `audienceType: \"app-url\"` → audience is your HTTPS webhook URL.
   - `audienceType: \"project-number\"` → audience is the Cloud project number.
3. Messages are routed by space:
   - DMs use session key `agent:<agentId>:googlechat:direct:<spaceId>`.
   - Spaces use session key `agent:<agentId>:googlechat:group:<spaceId>`.
4. DM access is pairing by default. Unknown senders receive a pairing code; approve with:
   - `openclaw pairing approve googlechat <code>`
5. Group spaces require @-mention by default. Use `botUser` if mention detection needs the app’s user name.

## [​](https://docs.openclaw.ai/channels/googlechat\\#targets)  Targets

Use these identifiers for delivery and allowlists:

- Direct messages: `users/<userId>` (recommended).
- Raw email `name@example.com` is mutable and only used for direct allowlist matching when `channels.googlechat.dangerouslyAllowNameMatching: true`.
- Deprecated: `users/<email>` is treated as a user id, not an email allowlist.
- Spaces: `spaces/<spaceId>`.

## [​](https://docs.openclaw.ai/channels/googlechat\\#config-highlights)  Config highlights

```
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: \"/path/to/service-account.json\",
      // or serviceAccountRef: { source: \"file\", provider: \"filemain\", id: \"/channels/googlechat/serviceAccount\" }
      audienceType: \"app-url\",
      audience: \"https://gateway.example.com/googlechat\",
      webhookPath: \"/googlechat\",
      botUser: \"users/1234567890\", // optional; helps mention detection
      dm: {
        policy: \"pairing\",
        allowFrom: [\"users/1234567890\"],
      },
      groupPolicy: \"allowlist\",
      groups: {
        \"spaces/AAAA\": {
          allow: true,
          requireMention: true,
          users: [\"users/1234567890\"],
          systemPrompt: \"Short answers only.\",
        },
      },
      actions: { reactions: true },
      typingIndicator: \"message\",
      mediaMaxMb: 20,
    },
  },
}
```

Notes:

- Service account credentials can also be passed inline with `serviceAccount` (JSON string).
- `serviceAccountRef` is also supported (env/file SecretRef), including per-account refs under `channels.googlechat.accounts.<id>.serviceAccountRef`.
- Default webhook path is `/googlechat` if `webhookPath` isn’t set.
- `dangerouslyAllowNameMatching` re-enables mutable email principal matching for allowlists (break-glass compatibility mode).
- Reactions are available via the `reactions` tool and `channels action` when `actions.reactions` is enabled.
- Message actions expose `send` for text and `upload-file` for explicit attachment sends. `upload-file` accepts `media` / `filePath` / `path` plus optional `message`, `filename`, and thread targeting.
- `typingIndicator` supports `none`, `message` (default), and `reaction` (reaction requires user OAuth).
- Attachments are downloaded through the Chat API and stored in the media pipeline (size capped by `mediaMaxMb`).

Secrets reference details: [Secrets Management](https://docs.openclaw.ai/gateway/secrets).

## [​](https://docs.openclaw.ai/channels/googlechat\\#troubleshooting)  Troubleshooting

### [​](https://docs.openclaw.ai/channels/googlechat\\#405-method-not-allowed)  405 Method Not Allowed

If Google Cloud Logs Explorer shows errors like:

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

This means the webhook handler isn’t registered. Common causes:

1. **Channel not configured**: The `channels.googlechat` section is missing from your config. Verify with:














```
openclaw config get channels.googlechat
```











If it returns “Config path not found”, add the configuration (see [Config highlights](https://docs.openclaw.ai/channels/googlechat#config-highlights)).
2. **Plugin not enabled**: Check plugin status:














```
openclaw plugins list | grep googlechat
```











If it shows “disabled”, add `plugins.entries.googlechat.enabled: true` to your config.
3. **Gateway not restarted**: After adding config, restart the gateway:














```
openclaw gateway restart
```


Verify the channel is running:

```
openclaw channels status
# Should show: Google Chat default: enabled, configured, ...
```

### [​](https://docs.openclaw.ai/channels/googlechat\\#other-issues)  Other issues

- Check `openclaw channels status --probe` for auth errors or missing audience config.
- If no messages arrive, confirm the Chat app’s webhook URL + event subscriptions.
- If mention gating blocks replies, set `botUser` to the app’s user resource name and verify `requireMention`.
- Use `openclaw logs --follow` while sending a test message to see if requests reach the gateway.

Related docs:

- [Gateway configuration](https://docs.openclaw.ai/gateway/configuration)
- [Security](https://docs.openclaw.ai/gateway/security)
- [Reactions](https://docs.openclaw.ai/tools/reactions)

## [​](https://docs.openclaw.ai/channels/googlechat\\#related)  Related

- [Channels Overview](https://docs.openclaw.ai/channels) — all supported channels
- [Pairing](https://docs.openclaw.ai/channels/pairing) — DM authentication and pairing flow
- [Groups](https://docs.openclaw.ai/channels/groups) — group chat behavior and mention gating
- [Channel Routing](https://docs.openclaw.ai/channels/channel-routing) — session routing for messages
- [Security](https://docs.openclaw.ai/gateway/security) — access model and hardening

[Microsoft Teams](https://docs.openclaw.ai/channels/msteams) [iMessage](https://docs.openclaw.ai/channels/imessage)

Ctrl+I

---

## Matrix
**Source:** https://docs.openclaw.ai/channels/matrix

[Skip to main content](https://docs.openclaw.ai/channels/matrix#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Mainstream messaging

Matrix

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Bundled plugin](https://docs.openclaw.ai/channels/matrix#bundled-plugin)
- [Setup](https://docs.openclaw.ai/channels/matrix#setup)
- [Interactive setup](https://docs.openclaw.ai/channels/matrix#interactive-setup)
- [Minimal config](https://docs.openclaw.ai/channels/matrix#minimal-config)
- [Auto-join](https://docs.openclaw.ai/channels/matrix#auto-join)
- [Allowlist target formats](https://docs.openclaw.ai/channels/matrix#allowlist-target-formats)
- [Account ID normalization](https://docs.openclaw.ai/channels/matrix#account-id-normalization)
- [Cached credentials](https://docs.openclaw.ai/channels/matrix#cached-credentials)
- [Environment variables](https://docs.openclaw.ai/channels/matrix#environment-variables)
- [Configuration example](https://docs.openclaw.ai/channels/matrix#configuration-example)
- [Streaming previews](https://docs.openclaw.ai/channels/matrix#streaming-previews)
- [Self-hosted push rules for quiet finalized previews](https://docs.openclaw.ai/channels/matrix#self-hosted-push-rules-for-quiet-finalized-previews)
- [Bot-to-bot rooms](https://docs.openclaw.ai/channels/matrix#bot-to-bot-rooms)
- [Encryption and verification](https://docs.openclaw.ai/channels/matrix#encryption-and-verification)
- [Enable encryption](https://docs.openclaw.ai/channels/matrix#enable-encryption)
- [Status and trust signals](https://docs.openclaw.ai/channels/matrix#status-and-trust-signals)
- [Verify this device with a recovery key](https://docs.openclaw.ai/channels/matrix#verify-this-device-with-a-recovery-key)
- [Bootstrap or repair cross-signing](https://docs.openclaw.ai/channels/matrix#bootstrap-or-repair-cross-signing)
- [Room-key backup](https://docs.openclaw.ai/channels/matrix#room-key-backup)
- [Listing, requesting, and responding to verifications](https://docs.openclaw.ai/channels/matrix#listing-requesting-and-responding-to-verifications)
- [Multi-account notes](https://docs.openclaw.ai/channels/matrix#multi-account-notes)
- [Profile management](https://docs.openclaw.ai/channels/matrix#profile-management)
- [Threads](https://docs.openclaw.ai/channels/matrix#threads)
- [Session routing (sessionScope)](https://docs.openclaw.ai/channels/matrix#session-routing-sessionscope)
- [Reply threading (threadReplies)](https://docs.openclaw.ai/channels/matrix#reply-threading-threadreplies)
- [Thread inheritance and slash commands](https://docs.openclaw.ai/channels/matrix#thread-inheritance-and-slash-commands)
- [ACP conversation bindings](https://docs.openclaw.ai/channels/matrix#acp-conversation-bindings)
- [Thread binding config](https://docs.openclaw.ai/channels/matrix#thread-binding-config)
- [Reactions](https://docs.openclaw.ai/channels/matrix#reactions)
- [History context](https://docs.openclaw.ai/channels/matrix#history-context)
- [Context visibility](https://docs.openclaw.ai/channels/matrix#context-visibility)
- [DM and room policy](https://docs.openclaw.ai/channels/matrix#dm-and-room-policy)
- [Direct room repair](https://docs.openclaw.ai/channels/matrix#direct-room-repair)
- [Exec approvals](https://docs.openclaw.ai/channels/matrix#exec-approvals)
- [Slash commands](https://docs.openclaw.ai/channels/matrix#slash-commands)
- [Multi-account](https://docs.openclaw.ai/channels/matrix#multi-account)
- [Private/LAN homeservers](https://docs.openclaw.ai/channels/matrix#private%2Flan-homeservers)
- [Proxying Matrix traffic](https://docs.openclaw.ai/channels/matrix#proxying-matrix-traffic)
- [Target resolution](https://docs.openclaw.ai/channels/matrix#target-resolution)
- [Configuration reference](https://docs.openclaw.ai/channels/matrix#configuration-reference)
- [Account and connection](https://docs.openclaw.ai/channels/matrix#account-and-connection)
- [Encryption](https://docs.openclaw.ai/channels/matrix#encryption)
- [Access and policy](https://docs.openclaw.ai/channels/matrix#access-and-policy)
- [Reply behavior](https://docs.openclaw.ai/channels/matrix#reply-behavior)
- [Reaction settings](https://docs.openclaw.ai/channels/matrix#reaction-settings)
- [Tooling and per-room overrides](https://docs.openclaw.ai/channels/matrix#tooling-and-per-room-overrides)
- [Exec approval settings](https://docs.openclaw.ai/channels/matrix#exec-approval-settings)
- [Related](https://docs.openclaw.ai/channels/matrix#related)

Matrix is a bundled channel plugin for OpenClaw.
It uses the official `matrix-js-sdk` and supports DMs, rooms, threads, media, reactions, polls, location, and E2EE.

## [​](https://docs.openclaw.ai/channels/matrix\\#bundled-plugin)  Bundled plugin

Current packaged OpenClaw releases ship the Matrix plugin in the box. You do not need to install anything; configuring `channels.matrix.*` (see [Setup](https://docs.openclaw.ai/channels/matrix#setup)) is what activates it.For older builds or custom installs that exclude Matrix, install manually first:

```
openclaw plugins install @openclaw/matrix
# or, from a local checkout
openclaw plugins install ./path/to/local/matrix-plugin
```

`plugins install` registers and enables the plugin, so no separate `openclaw plugins enable matrix` step is needed. The plugin still does nothing until you configure the channel below. See [Plugins](https://docs.openclaw.ai/tools/plugin) for general plugin behavior and install rules.

## [​](https://docs.openclaw.ai/channels/matrix\\#setup)  Setup

1. Create a Matrix account on your homeserver.
2. Configure `channels.matrix` with either `homeserver` \\+ `accessToken`, or `homeserver` \\+ `userId` \\+ `password`.
3. Restart the gateway.
4. Start a DM with the bot, or invite it to a room (see [auto-join](https://docs.openclaw.ai/channels/matrix#auto-join) — fresh invites only land when `autoJoin` allows them).

### [​](https://docs.openclaw.ai/channels/matrix\\#interactive-setup)  Interactive setup

```
openclaw channels add
openclaw configure --section channels
```

The wizard asks for: homeserver URL, auth method (access token or password), user ID (password auth only), optional device name, whether to enable E2EE, and whether to configure room access and auto-join.If matching `MATRIX_*` env vars already exist and the selected account has no saved auth, the wizard offers an env-var shortcut. To resolve room names before saving an allowlist, run `openclaw channels resolve --channel matrix \"Project Room\"`. When E2EE is enabled, the wizard writes the config and runs the same bootstrap as [`openclaw matrix encryption setup`](https://docs.openclaw.ai/channels/matrix#encryption-and-verification).

### [​](https://docs.openclaw.ai/channels/matrix\\#minimal-config)  Minimal config

Token-based:

```
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: \"https://matrix.example.org\",
      accessToken: \"syt_xxx\",
      dm: { policy: \"pairing\" },
    },
  },
}
```

Password-based (the token is cached after first login):

```
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: \"https://matrix.example.org\",
      userId: \"@bot:example.org\",
      password: \"replace-me\", // pragma: allowlist secret
      deviceName: \"OpenClaw Gateway\",
    },
  },
}
```

### [​](https://docs.openclaw.ai/channels/matrix\\#auto-join)  Auto-join

`channels.matrix.autoJoin` defaults to `off`. With the default, the bot will not appear in new rooms or DMs from fresh invites until you join manually.OpenClaw cannot tell at invite time whether an invited room is a DM or a group, so all invites — including DM-style invites — go through `autoJoin` first. `dm.policy` only applies later, after the bot has joined and the room has been classified.

Set `autoJoin: \"allowlist\"` plus `autoJoinAllowlist` to restrict which invites the bot accepts, or `autoJoin: \"always\"` to accept every invite.`autoJoinAllowlist` only accepts stable targets: `!roomId:server`, `#alias:server`, or `*`. Plain room names are rejected; alias entries are resolved against the homeserver, not against state claimed by the invited room.

```
{
  channels: {
    matrix: {
      autoJoin: \"allowlist\",
      autoJoinAllowlist: [\"!ops:example.org\", \"#support:example.org\"],
      groups: {
        \"!ops:example.org\": { requireMention: true },
      },
    },
  },
}
```

To accept every invite, use `autoJoin: \"always\"`.

### [​](https://docs.openclaw.ai/channels/matrix\\#allowlist-target-formats)  Allowlist target formats

DM and room allowlists are best populated with stable IDs:

- DMs (`dm.allowFrom`, `groupAllowFrom`, `groups.<room>.users`): use `@user:server`. Display names only resolve when the homeserver directory returns exactly one match.
- Rooms (`groups`, `autoJoinAllowlist`): use `!room:server` or `#alias:server`. Names are resolved best-effort against joined rooms; unresolved entries are ignored at runtime.

### [​](https://docs.openclaw.ai/channels/matrix\\#account-id-normalization)  Account ID normalization

The wizard converts a friendly name into a normalized account ID. For example, `Ops Bot` becomes `ops-bot`. Punctuation is escaped in scoped env-var names so that two accounts cannot collide: `-` → `_X2D_`, so `ops-prod` maps to `MATRIX_OPS_X2D_PROD_*`.

### [​](https://openclaw.ai/channels/matrix\\#cached-credentials)  Cached credentials

Matrix stores cached credentials under `~/.openclaw/credentials/matrix/`:

- default account: `credentials.json`
- named accounts: `credentials-<account>.json`

When cached credentials exist there, OpenClaw treats Matrix as configured even if the access token is not in the config file — that covers setup, `openclaw doctor`, and channel-status probes.

### [​](https://docs.openclaw.ai/channels/matrix\\#environment-variables)  Environment variables

Used when the equivalent config key is not set. The default account uses unprefixed names; named accounts use the account ID inserted before the suffix.

| Default account | Named account (`<ID>` is the normalized account ID) |
| --- | --- |
| `MATRIX_HOMESERVER` | `MATRIX_<ID>_HOMESERVER` |
| `MATRIX_ACCESS_TOKEN` | `MATRIX_<ID>_ACCESS_TOKEN` |
| `MATRIX_USER_ID` | `MATRIX_<ID>_USER_ID` |
| `MATRIX_PASSWORD` | `MATRIX_<ID>_PASSWORD` |
| `MATRIX_DEVICE_ID` | `MATRIX_<ID>_DEVICE_ID` |
| `MATRIX_DEVICE_NAME` | `MATRIX_<ID>_DEVICE_NAME` |
| `MATRIX_RECOVERY_KEY` | `MATRIX_<ID>_RECOVERY_KEY` |

For account `ops`, the names become `MATRIX_OPS_HOMESERVER`, `MATRIX_OPS_ACCESS_TOKEN`, and so on. The recovery-key env vars are read by recovery-aware CLI flows (`verify backup restore`, `verify device`, `verify bootstrap`) when you pipe the key in via `--recovery-key-stdin`.`MATRIX_HOMESERVER` cannot be set from a workspace `.env`; see [Workspace `.env` files](https://docs.openclaw.ai/gateway/security).

## [​](https://docs.openclaw.ai/channels/matrix\\#configuration-example)  Configuration example

A practical baseline with DM pairing, room allowlist, and E2EE:

```
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: \"https://matrix.example.org\",
      accessToken: \"syt_xxx\",
      encryption: true,

      dm: {
        policy: \"pairing\",
        sessionScope: \"per-room\",
        threadReplies: \"off\",
      },

      groupPolicy: \"allowlist\",
      groupAllowFrom: [\"@admin:example.org\"],
      groups: {
        \"!roomid:example.org\": { requireMention: true },
      },

      autoJoin: \"allowlist\",
      autoJoinAllowlist: [\"!roomid:example.org\"],
      threadReplies: \"inbound\",
      replyToMode: \"off\",
      streaming: \"partial\",
    },
  },
}
```

## [​](https://docs.openclaw.ai/channels/matrix\\#streaming-previews)  Streaming previews

Matrix reply streaming is opt-in. `streaming` controls how OpenClaw delivers the in-flight assistant reply; `blockStreaming` controls whether each completed block is preserved as its own Matrix message.

```
{
  channels: {
    matrix: {
      streaming: \"partial\",
    },
  },
}
```

| `streaming` | Behavior |
| --- | --- |
| `\"off\"` (default) | Wait for the full reply, send once. `true` ↔ `\"partial\"`, `false` ↔ `\"off\"`. |
| `\"partial\"` | Edit one normal text message in place as the model writes the current block. Stock Matrix clients may notify on the first preview, not the final edit. |
| `\"quiet\"` | Same as `\"partial\"` but the message is a non-notifying notice. Recipients only get a notification once a per-user push rule matches the finalized edit (see below). |

`blockStreaming` is independent of `streaming`:

| `streaming` | `blockStreaming: true` | `blockStreaming: false` (default) |
| --- | --- | --- |
| `\"partial\"` / `\"quiet\"` | Live draft for the current block, completed blocks kept as messages | Live draft for the current block, finalized in place |
| `\"off\"` | One notifying Matrix message per finished block | One notifying Matrix message for the full reply |

Notes:

- If a preview grows past Matrix’s per-event size limit, OpenClaw stops preview streaming and falls back to final-only delivery.
- Media replies always send attachments normally. If a stale preview can no longer be reused safely, OpenClaw redacts it before sending the final media reply.
- Preview edits cost extra Matrix API calls. Leave `streaming: \"off\"` if you want the most conservative rate-limit profile.

### [​](https://docs.openclaw.ai/channels/matrix\\#self-hosted-push-rules-for-quiet-finalized-previews)  Self-hosted push rules for quiet finalized previews

`streaming: \"quiet\"` only notifies recipients once a block or turn is finalized — a per-user push rule has to match the finalized preview marker. See [Matrix push rules for quiet previews](https://docs.openclaw.ai/channels/matrix-push-rules) for the full recipe (recipient token, pusher check, rule install, per-homeserver notes).

## [​](https://docs.openclaw.ai/channels/matrix\\#bot-to-bot-rooms)  Bot-to-bot rooms

By default, Matrix messages from other configured OpenClaw Matrix accounts are ignored.Use `allowBots` when you intentionally want inter-agent Matrix traffic:

```
{
  channels: {
    matrix: {
      allowBots: \"mentions\", // true | \"mentions\"
      groups: {
        \"!roomid:example.org\": {
          requireMention: true,
        },
      },
    },
  },
}
```

- `allowBots: true` accepts messages from other configured Matrix bot accounts in allowed rooms and DMs.
- `allowBots: \"mentions\"` accepts those messages only when they visibly mention this bot in rooms. DMs are still allowed.
- `groups.<room>.allowBots` overrides the account-level setting for one room.
- OpenClaw still ignores messages from the same Matrix user ID to avoid self-reply loops.
- Matrix does not expose a native bot flag here; OpenClaw treats “bot-authored” as “sent by another configured Matrix account on this OpenClaw gateway”.

Use strict room allowlists and mention requirements when enabling bot-to-bot traffic in shared rooms.

## [​](https://docs.openclaw.ai/channels/matrix\\#encryption-and-verification)  Encryption and verification

In encrypted (E2EE) rooms, outbound image events use `thumbnail_file` so image previews are encrypted alongside the full attachment. Unencrypted rooms still use plain `thumbnail_url`. No configuration is needed — the plugin detects E2EE state automatically.All `openclaw matrix` commands accept `--verbose` (full diagnostics), `--json` (machine-readable output), and `--account <id>` (multi-account setups). Output is concise by default with quiet internal SDK logging. The examples below show the canonical form; add the flags as needed.

### [​](https://docs.openclaw.ai/channels/matrix\\#enable-encryption)  Enable encryption

```
openclaw matrix encryption setup
```

Bootstraps secret storage and cross-signing, creates a room-key backup if needed, then prints status and next steps. Useful flags:

- `--recovery-key <key>` apply a recovery key before bootstrapping (prefer the stdin form documented below)
- `--force-reset-cross-signing` discard the current cross-signing identity and create a new one (use only intentionally)

For a new account, enable E2EE at creation time:

```
openclaw matrix account add \\\n  --homeserver https://matrix.example.org \\\n  --access-token syt_xxx \\\n  --enable-e2ee
```

`--encryption` is an alias for `--enable-e2ee`.Manual config equivalent:

```
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: \"https://matrix.example.org\",
      accessToken: \"syt_xxx\",
      encryption: true,
      dm: { policy: \"pairing\" },
    },
  },
}
```

### [​](https://docs.openclaw.ai/channels/matrix\\#status-and-trust-signals)  Status and trust signals

```
openclaw matrix verify status
openclaw matrix verify status --include-recovery-key --json
```

`verify status` reports three independent trust signals (`--verbose` shows all of them):

- `Locally trusted`: trusted by this client only
- `Cross-signing verified`: the SDK reports verification via cross-signing
- `Signed by owner`: signed by your own self-signing key (diagnostic only)

`Verified by owner` becomes `yes` only when `Cross-signing verified` is `yes`. Local trust or an owner signature alone is not enough.`--allow-degraded-local-state` returns best-effort diagnostics without preparing the Matrix account first; useful for offline or partially-configured probes.

### [​](https://docs.openclaw.ai/channels/matrix\\#verify-this-device-with-a-recovery-key)  Verify this device with a recovery key

The recovery key is sensitive — pipe it via stdin instead of passing it on the command line. Set `MATRIX_RECOVERY_KEY` (or `MATRIX_<ID>_RECOVERY_KEY` for a named account):

```
printf ‘%s\\n’ \"$MATRIX_RECOVERY_KEY\" | openclaw matrix verify device --recovery-key-stdin
```

The command reports three states:

- `Recovery key accepted`: Matrix accepted the key for secret storage or device trust.
- `Backup usable`: room-key backup can be loaded with the trusted recovery material.
- `Device verified by owner`: this device has full Matrix cross-signing identity trust.

It exits non-zero when full identity trust is incomplete, even if the recovery key unlocked backup material. In that case, finish self-verification from another Matrix client:

```
openclaw matrix verify self
```

`verify self` waits for `Cross-signing verified: yes` before it exits successfully. Use `--timeout-ms <ms>` to tune the wai...(content truncated)

---

## Channel routing - OpenClaw
**Source:** https://docs.openclaw.ai/channels/channel-routing

[Skip to main content](https://docs.openclaw.ai/channels/channel-routing#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Configuration

Channel routing

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Channels & routing](https://docs.openclaw.ai/channels/channel-routing#channels-%26-routing)
- [Key terms](https://docs.openclaw.ai/channels/channel-routing#key-terms)
- [Session key shapes (examples)](https://docs.openclaw.ai/channels/channel-routing#session-key-shapes-examples)
- [Main DM route pinning](https://docs.openclaw.ai/channels/channel-routing#main-dm-route-pinning)
- [Routing rules (how an agent is chosen)](https://docs.openclaw.ai/channels/channel-routing#routing-rules-how-an-agent-is-chosen)
- [Broadcast groups (run multiple agents)](https://docs.openclaw.ai/channels/channel-routing#broadcast-groups-run-multiple-agents)
- [Config overview](https://docs.openclaw.ai/channels/channel-routing#config-overview)
- [Session storage](https://docs.openclaw.ai/channels/channel-routing#session-storage)
- [WebChat behavior](https://docs.openclaw.ai/channels/channel-routing#webchat-behavior)
- [Reply context](https://docs.openclaw.ai/channels/channel-routing#reply-context)
- [Related](https://docs.openclaw.ai/channels/channel-routing#related)

# [​](https://docs.openclaw.ai/channels/channel-routing\\#channels-&-routing)  Channels & routing

OpenClaw routes replies **back to the channel where a message came from**. The
model does not choose a channel; routing is deterministic and controlled by the
host configuration.

## [​](https://docs.openclaw.ai/channels/channel-routing\\#key-terms)  Key terms

- **Channel**: `telegram`, `whatsapp`, `discord`, `irc`, `googlechat`, `slack`, `signal`, `imessage`, `line`, plus plugin channels. `webchat` is the internal WebChat UI channel and is not a configurable outbound channel.
- **AccountId**: per‑channel account instance (when supported).
- Optional channel default account: `channels.<channel>.defaultAccount` chooses
which account is used when an outbound path does not specify `accountId`.

  - In multi-account setups, set an explicit default (`defaultAccount` or `accounts.default`) when two or more accounts are configured. Without it, fallback routing may pick the first normalized account ID.
- **AgentId**: an isolated workspace + session store (“brain”).
- **SessionKey**: the bucket key used to store context and control concurrency.

## [​](https://docs.openclaw.ai/channels/channel-routing\\#session-key-shapes-examples)  Session key shapes (examples)

Direct messages collapse to the agent’s **main** session by default:

- `agent:<agentId>:<mainKey>` (default: `agent:main:main`)

Even when direct-message conversation history is shared with main, sandbox and
tool policy use a derived per-account direct-chat runtime key for external DMs
so channel-originated messages are not treated like local main-session runs.Groups and channels remain isolated per channel:

- Groups: `agent:<agentId>:<channel>:group:<id>`
- Channels/rooms: `agent:<agentId>:<channel>:channel:<id>`

Threads:

- Slack/Discord threads append `:thread:<threadId>` to the base key.
- Telegram forum topics embed `:topic:<topicId>` in the group key.

Examples:

- `agent:main:telegram:group:-1001234567890:topic:42`
- `agent:main:discord:channel:123456:thread:987654`

## [​](https://docs.openclaw.ai/channels/channel-routing\\#main-dm-route-pinning)  Main DM route pinning

When `session.dmScope` is `main`, direct messages may share one main session.
To prevent the session’s `lastRoute` from being overwritten by non-owner DMs,
OpenClaw infers a pinned owner from `allowFrom` when all of these are true:

- `allowFrom` has exactly one non-wildcard entry.
- The entry can be normalized to a concrete sender ID for that channel.
- The inbound DM sender does not match that pinned owner.

In that mismatch case, OpenClaw still records inbound session metadata, but it
skips updating the main session `lastRoute`.

## [​](https://docs.openclaw.ai/channels/channel-routing\\#routing-rules-how-an-agent-is-chosen)  Routing rules (how an agent is chosen)

Routing picks **one agent** for each inbound message:

1. **Exact peer match** (`bindings` with `peer.kind` \\+ `peer.id`).
2. **Parent peer match** (thread inheritance).
3. **Guild + roles match** (Discord) via `guildId` \\+ `roles`.
4. **Guild match** (Discord) via `guildId`.
5. **Team match** (Slack) via `teamId`.
6. **Account match** (`accountId` on the channel).
7. **Channel match** (any account on that channel, `accountId: \"*\"`).
8. **Default agent** (`agents.list[].default`, else first list entry, fallback to `main`).

When a binding includes multiple match fields (`peer`, `guildId`, `teamId`, `roles`), **all provided fields must match** for that binding to apply.The matched agent determines which workspace and session store are used.

## [​](https://docs.openclaw.ai/channels/channel-routing\\#broadcast-groups-run-multiple-agents)  Broadcast groups (run multiple agents)

Broadcast groups let you run **multiple agents** for the same peer **when OpenClaw would normally reply** (for example: in WhatsApp groups, after mention/activation gating).Config:

```
{
  broadcast: {
    strategy: \"parallel\",
    \"120363403215116621@g.us\": [\"alfred\", \"baerbel\"],
    \"+15555550123\": [\"support\", \"logger\"],
  },
}
```

See: [Broadcast Groups](https://docs.openclaw.ai/channels/broadcast-groups).

## [​](https://docs.openclaw.ai/channels/channel-routing\\#config-overview)  Config overview

- `agents.list`: named agent definitions (workspace, model, etc.).
- `bindings`: map inbound channels/accounts/peers to agents.

Example:

```
{
  agents: {
    list: [{ id: \"support\", name: \"Support\", workspace: \"~/.openclaw/workspace-support\" }],
  },
  bindings: [\\\
    { match: { channel: \"slack\", teamId: \"T123\" }, agentId: \"support\" },\\\
    { match: { channel: \"telegram\", peer: { kind: \"group\", id: \"-100123\" } }, agentId: \"support\" },\\\
  ],
}
```

## [​](https://docs.openclaw.ai/channels/channel-routing\\#session-storage)  Session storage

Session stores live under the state directory (default `~/.openclaw`):

- `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- JSONL transcripts live alongside the store

You can override the store path via `session.store` and `{agentId}` templating.Gateway and ACP session discovery also scans disk-backed agent stores under the
default `agents/` root and under templated `session.store` roots. Discovered
stores must stay inside that resolved agent root and use a regular
`sessions.json` file. Symlinks and out-of-root paths are ignored.

## [​](https://docs.openclaw.ai/channels/channel-routing\\#webchat-behavior)  WebChat behavior

WebChat attaches to the **selected agent** and defaults to the agent’s main
session. Because of this, WebChat lets you see cross‑channel context for that
agent in one place.

## [​](https://docs.openclaw.ai/channels/channel-routing\\#reply-context)  Reply context

Inbound replies include:

- `ReplyToId`, `ReplyToBody`, and `ReplyToSender` when available.
- Quoted context is appended to `Body` as a `[Replying to ...]` block.

This is consistent across channels.

## [​](https://docs.openclaw.ai/channels/channel-routing\\#related)  Related

- [Groups](https://docs.openclaw.ai/channels/groups)
- [Broadcast groups](https://docs.openclaw.ai/channels/broadcast-groups)
- [Pairing](https://docs.openclaw.ai/channels/pairing)

[Broadcast groups](https://docs.openclaw.ai/channels/broadcast-groups) [Channel location parsing](https://docs.openclaw.ai/channels/location)

Ctrl+I

---

## Signal - OpenClaw
**Source:** https://docs.openclaw.ai/channels/signal

[Skip to main content](https://docs.openclaw.ai/channels/signal#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nMainstream messaging\n\nSignal\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Prerequisites](https://docs.openclaw.ai/channels/signal#prerequisites)\n- [Quick setup (beginner)](https://docs.openclaw.ai/channels/signal#quick-setup-beginner)\n- [What it is](https://docs.openclaw.ai/channels/signal#what-it-is)\n- [Config writes](https://docs.openclaw.ai/channels/signal#config-writes)\n- [The number model (important)](https://docs.openclaw.ai/channels/signal#the-number-model-important)\n- [Setup path A: link existing Signal account (QR)](https://docs.openclaw.ai/channels/signal#setup-path-a-link-existing-signal-account-qr)\n- [Setup path B: register dedicated bot number (SMS, Linux)](https://docs.openclaw.ai/channels/signal#setup-path-b-register-dedicated-bot-number-sms-linux)\n- [External daemon mode (httpUrl)](https://docs.openclaw.ai/channels/signal#external-daemon-mode-httpurl)\n- [Access control (DMs + groups)](https://docs.openclaw.ai/channels/signal#access-control-dms-%2B-groups)\n- [How it works (behavior)](https://docs.openclaw.ai/channels/signal#how-it-works-behavior)\n- [Media + limits](https://docs.openclaw.ai/channels/signal#media-%2B-limits)\n- [Typing + read receipts](https://docs.openclaw.ai/channels/signal#typing-%2B-read-receipts)\n- [Reactions (message tool)](https://docs.openclaw.ai/channels/signal#reactions-message-tool)\n- [Delivery targets (CLI/cron)](https://docs.openclaw.ai/channels/signal#delivery-targets-cli%2Fcron)\n- [Troubleshooting](https://docs.openclaw.ai/channels/signal#troubleshooting)\n- [Security notes](https://docs.openclaw.ai/channels/signal#security-notes)\n- [Configuration reference (Signal)](https://docs.openclaw.ai/channels/signal#configuration-reference-signal)\n- [Related](https://docs.openclaw.ai/channels/signal#related)\n\nStatus: external CLI integration. Gateway talks to `signal-cli` over HTTP JSON-RPC + SSE.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#prerequisites)  Prerequisites\n\n- OpenClaw installed on your server (Linux flow below tested on Ubuntu 24).\n- `signal-cli` available on the host where the gateway runs.\n- A phone number that can receive one verification SMS (for SMS registration path).\n- Browser access for Signal captcha (`signalcaptchas.org`) during registration.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#quick-setup-beginner)  Quick setup (beginner)\n\n1. Use a **separate Signal number** for the bot (recommended).\n2. Install `signal-cli` (Java required if you use the JVM build).\n3. Choose one setup path:\n   - **Path A (QR link):**`signal-cli link -n \"OpenClaw\"` and scan with Signal.\n   - **Path B (SMS register):** register a dedicated number with captcha + SMS verification.\n4. Configure OpenClaw and restart the gateway.\n5. Send a first DM and approve pairing (`openclaw pairing approve signal <CODE>`).\n\nMinimal config:\n\n```\n{\n  channels: {\n    signal: {\n      enabled: true,\n      account: \"+15551234567\",\n      cliPath: \"signal-cli\",\n      dmPolicy: \"pairing\",\n      allowFrom: [\"+15557654321\"],\n    },\n  },\n}\n```\n\nField reference:\n\n| Field | Description |\n| --- | --- |\n| `account` | Bot phone number in E.164 format (`+15551234567`) |\n| `cliPath` | Path to `signal-cli` (`signal-cli` if on `PATH`) |\n| `dmPolicy` | DM access policy (`pairing` recommended) |\n| `allowFrom` | Phone numbers or `uuid:<id>` values allowed to DM |\n\n## [​](https://docs.openclaw.ai/channels/signal\\#what-it-is)  What it is\n\n- Signal channel via `signal-cli` (not embedded libsignal).\n- Deterministic routing: replies always go back to Signal.\n- DMs share the agent’s main session; groups are isolated (`agent:<agentId>:signal:group:<groupId>`).\n\n## [​](https://docs.openclaw.ai/channels/signal\\#config-writes)  Config writes\n\nBy default, Signal is allowed to write config updates triggered by `/config set|unset` (requires `commands.config: true`).Disable with:\n\n```\n{\n  channels: { signal: { configWrites: false } },\n}\n```\n\n## [​](https://openclaw.ai/channels/signal\\#the-number-model-important)  The number model (important)\n\n- The gateway connects to a **Signal device** (the `signal-cli` account).\n- If you run the bot on **your personal Signal account**, it will ignore your own messages (loop protection).\n- For “I text the bot and it replies,” use a **separate bot number**.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#setup-path-a-link-existing-signal-account-qr)  Setup path A: link existing Signal account (QR)\n\n1. Install `signal-cli` (JVM or native build).\n2. Link a bot account:\n   - `signal-cli link -n \"OpenClaw\"` then scan the QR in Signal.\n3. Configure Signal and start the gateway.\n\nExample:\n\n```\n{\n  channels: {\n    signal: {\n      enabled: true,\n      account: \"+15551234567\",\n      cliPath: \"signal-cli\",\n      dmPolicy: \"pairing\",\n      allowFrom: [\"+15557654321\"],\n    },\n  },\n}\n```\n\nMulti-account support: use `channels.signal.accounts` with per-account config and optional `name`. See [`gateway/configuration`](https://docs.openclaw.ai/gateway/config-channels#multi-account-all-channels) for the shared pattern.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#setup-path-b-register-dedicated-bot-number-sms-linux)  Setup path B: register dedicated bot number (SMS, Linux)\n\nUse this when you want a dedicated bot number instead of linking an existing Signal app account.\n\n1. Get a number that can receive SMS (or voice verification for landlines).\n   - Use a dedicated bot number to avoid account/session conflicts.\n2. Install `signal-cli` on the gateway host:\n\n```\nVERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e \'s/^.*\\/v//\')\ncurl -L -O \"https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz\"\nsudo tar xf \"signal-cli-${VERSION}-Linux-native.tar.gz\" -C /opt\nsudo ln -sf /opt/signal-cli /usr/local/bin/\nsignal-cli --version\n```\n\nIf you use the JVM build (`signal-cli-${VERSION}.tar.gz`), install JRE 25+ first.\nKeep `signal-cli` updated; upstream notes that old releases can break as Signal server APIs change.\n\n3. Register and verify the number:\n\n```\nsignal-cli -a +<BOT_PHONE_NUMBER> register\n```\n\nIf captcha is required:\n\n1. Open `https://signalcaptchas.org/registration/generate.html`.\n2. Complete captcha, copy the `signalcaptcha://...` link target from “Open Signal”.\n3. Run from the same external IP as the browser session when possible.\n4. Run registration again immediately (captcha tokens expire quickly):\n\n```\nsignal-cli -a +<BOT_PHONE_NUMBER> register --captcha \'<SIGNALCAPTCHA_URL>\''\nsignal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>\n```\n\n4. Configure OpenClaw, restart gateway, verify channel:\n\n```\n# If you run the gateway as a user systemd service:\nsystemctl --user restart openclaw-gateway.service\n\n# Then verify:\nopenclaw doctor\nopenclaw channels status --probe\n```\n\n5. Pair your DM sender:\n   - Send any message to the bot number.\n   - Approve code on the server: `openclaw pairing approve signal <PAIRING_CODE>`.\n   - Save the bot number as a contact on your phone to avoid “Unknown contact”.\n\nRegistering a phone number account with `signal-cli` can de-authenticate the main Signal app session for that number. Prefer a dedicated bot number, or use QR link mode if you need to keep your existing phone app setup.\n\nUpstream references:\n\n- `signal-cli` README: `https://github.com/AsamK/signal-cli`\n- Captcha flow: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`\n- Linking flow: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`\n\n## [​](https://docs.openclaw.ai/channels/signal\\#external-daemon-mode-httpurl)  External daemon mode (httpUrl)\n\nIf you want to manage `signal-cli` yourself (slow JVM cold starts, container init, or shared CPUs), run the daemon separately and point OpenClaw at it:\n\n```\n{\n  channels: {\n    signal: {\n      httpUrl: \"http://127.0.0.1:8080\",\n      autoStart: false,\n    },\n  },\n}\n```\n\nThis skips auto-spawn and the startup wait inside OpenClaw. For slow starts when auto-spawning, set `channels.signal.startupTimeoutMs`.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#access-control-dms-+-groups)  Access control (DMs + groups)\n\nDMs:\n\n- Default: `channels.signal.dmPolicy = \"pairing\"`.\n- Unknown senders receive a pairing code; messages are ignored until approved (codes expire after 1 hour).\n- Approve via:\n  - `openclaw pairing list signal`\n  - `openclaw pairing approve signal <CODE>`\n- Pairing is the default token exchange for Signal DMs. Details: [Pairing](https://docs.openclaw.ai/channels/pairing)\n- UUID-only senders (from `sourceUuid`) are stored as `uuid:<id>` in `channels.signal.allowFrom`.\n\nGroups:\n\n- `channels.signal.groupPolicy = open | allowlist | disabled`.\n- `channels.signal.groupAllowFrom` controls who can trigger in groups when `allowlist` is set.\n- `channels.signal.groups[\"<group-id>\" | \"*\"]` can override group behavior with `requireMention`, `tools`, and `toolsBySender`.\n- Use `channels.signal.accounts.<id>.groups` for per-account overrides in multi-account setups.\n- Runtime note: if `channels.signal` is completely missing, runtime falls back to `groupPolicy=\"allowlist\"` for group checks (even if `channels.defaults.groupPolicy` is set).\n\n## [​](https://docs.openclaw.ai/channels/signal\\#how-it-works-behavior)  How it works (behavior)\n\n- `signal-cli` runs as a daemon; the gateway reads events via SSE.\n- Inbound messages are normalized into the shared channel envelope.\n- Replies always route back to the same number or group.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#media-+-limits)  Media + limits\n\n- Outbound text is chunked to `channels.signal.textChunkLimit` (default 4000).\n- Optional newline chunking: set `channels.signal.chunkMode=\"newline\"` to split on blank lines (paragraph boundaries) before length chunking.\n- Attachments supported (base64 fetched from `signal-cli`).\n- Voice-note attachments use the `signal-cli` filename as a MIME fallback when `contentType` is missing, so audio transcription can still classify AAC voice memos.\n- Default media cap: `channels.signal.mediaMaxMb` (default 8).\n- Use `channels.signal.ignoreAttachments` to skip downloading media.\n- Group history context uses `channels.signal.historyLimit` (or `channels.signal.accounts.*.historyLimit`), falling back to `messages.groupChat.historyLimit`. Set `0` to disable (default 50).\n\n## [​](https://docs.openclaw.ai/channels/signal\\#typing-+-read-receipts)  Typing + read receipts\n\n- **Typing indicators**: OpenClaw sends typing signals via `signal-cli sendTyping` and refreshes them while a reply is running.\n- **Read receipts**: when `channels.signal.sendReadReceipts` is true, OpenClaw forwards read receipts for allowed DMs.\n- Signal-cli does not expose read receipts for groups.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#reactions-message-tool)  Reactions (message tool)\n\n- Use `message action=react` with `channel=signal`.\n- Targets: sender E.164 or UUID (use `uuid:<id>` from pairing output; bare UUID works too).\n- `messageId` is the Signal timestamp for the message you’re reacting to.\n- Group reactions require `targetAuthor` or `targetAuthorUuid`.\n\nExamples:\n\n```\nmessage action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥\nmessage action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true\nmessage action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅\n```\n\nConfig:\n\n- `channels.signal.actions.reactions`: enable/disable reaction actions (default true).\n- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.\n\n  - `off`/`ack` disables agent reactions (message tool `react` will error).\n  - `minimal`/`extensive` enables agent reactions and sets the guidance level.\n- Per-account overrides: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#delivery-targets-cli/cron)  Delivery targets (CLI/cron)\n\n- DMs: `signal:+15551234567` (or plain E.164).\n- UUID DMs: `uuid:<id>` (or bare UUID).\n- Groups: `signal:group:<groupId>`.\n- Usernames: `username:<name>` (if supported by your Signal account).\n\n## [​](https://docs.openclaw.ai/channels/signal\\#troubleshooting)  Troubleshooting\n\nRun this ladder first:\n\n```\nopenclaw status\nopenclaw gateway status\nopenclaw logs --follow\nopenclaw doctor\nopenclaw channels status --probe\n```\n\nThen confirm DM pairing state if needed:\n\n```\nopenclaw pairing list signal\n```\n\nCommon failures:\n\n- Daemon reachable but no replies: verify account/daemon settings (`httpUrl`, `account`) and receive mode.\n- DMs ignored: sender is pending pairing approval.\n- Group messages ignored: group sender/mention gating blocks delivery.\n- Config validation errors after edits: run `openclaw doctor --fix`.\n- Signal missing from diagnostics: confirm `channels.signal.enabled: true`.\n\nExtra checks:\n\n```\nopenclaw pairing list signal\npgrep -af signal-cli\ngrep -i \"signal\" \"/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log\" | tail -20\n```\n\nFor triage flow: [/channels/troubleshooting](https://docs.openclaw.ai/channels/troubleshooting).\n\n## [​](https://docs.openclaw.ai/channels/signal\\#security-notes)  Security notes\n\n- `signal-cli` stores account keys locally (typically `~/.local/share/signal-cli/data/`).\n- Back up Signal account state before server migration or rebuild.\n- Keep `channels.signal.dmPolicy: \"pairing\"` unless you explicitly want broader DM access.\n- SMS verification is only needed for registration or recovery flows, but losing control of the number/account can complicate re-registration.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#configuration-reference-signal)  Configuration reference (Signal)\n\nFull configuration: [Configuration](https://docs.openclaw.ai/gateway/configuration)Provider options:\n\n- `channels.signal.enabled`: enable/disable channel startup.\n- `channels.signal.account`: E.164 for the bot account.\n- `channels.signal.cliPath`: path to `signal-cli`.\n- `channels.signal.httpUrl`: full daemon URL (overrides host/port).\n- `channels.signal.httpHost`, `channels.signal.httpPort`: daemon bind (default 127.0.0.1:8080).\n- `channels.signal.autoStart`: auto-spawn daemon (default true if `httpUrl` unset).\n- `channels.signal.startupTimeoutMs`: startup wait timeout in ms (cap 120000).\n- `channels.signal.receiveMode`: `on-start | manual`.\n- `channels.signal.ignoreAttachments`: skip attachment downloads.\n- `channels.signal.ignoreStories`: ignore stories from the daemon.\n- `channels.signal.sendReadReceipts`: forward read receipts.\n- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (default: pairing).\n- `channels.signal.allowFrom`: DM allowlist (E.164 or `uuid:<id>`). `open` requires `\"*\"`. Signal has no usernames; use phone/UUID ids.\n- `channels.signal.groupPolicy`: `open | allowlist | disabled` (default: allowlist).\n- `channels.signal.groupAllowFrom`: group sender allowlist.\n- `channels.signal.groups`: per-group overrides keyed by Signal group id (or `\"*\"`). Supported fields: `requireMention`, `tools`, `toolsBySender`.\n- `channels.signal.accounts.<id>.groups`: per-account version of `channels.signal.groups` for multi-account setups.\n- `channels.signal.historyLimit`: max group messages to include as context (0 disables).\n- `channels.signal.dmHistoryLimit`: DM history limit in user turns. Per-user overrides: `channels.signal.dms[\"<phone_or_uuid>\"].historyLimit`.\n- `channels.signal.textChunkLimit`: outbound chunk size (chars).\n- `channels.signal.chunkMode`: `length` (default) or `newline` to split on blank lines (paragraph boundaries) before length chunking.\n- `channels.signal.mediaMaxMb`: inbound/outbound media cap (MB).\n\nRelated global options:\n\n- `agents.list[].groupChat.mentionPatterns` (Signal does not support native mentions).\n- `messages.groupChat.mentionPatterns` (global fallback).\n- `messages.responsePrefix`.\n\n## [​](https://docs.openclaw.ai/channels/signal\\#related)  Related\n\n- [Channels Overview](https://docs.openclaw.ai/channels) — all supported channels\n- [Pairing](https://docs.openclaw.ai/channels/pairing) — DM authentication and pairing flow\n- [Groups](https://docs.openclaw.ai/channels/groups) — group chat behavior and mention gating\n- [Channel Routing](https://docs.openclaw.ai/channels/channel-routing) — session routing for messages\n- [Security](https://docs.openclaw.ai/gateway/security) — access model and hardening\n\n[WhatsApp](https://docs.openclaw.ai/channels/whatsapp) [Microsoft Teams](https://docs.openclaw.ai/channels/msteams)\n\nCtrl+I

---

## Zalo - OpenClaw
**Source:** https://docs.openclaw.ai/channels/zalo

[Skip to main content](https://docs.openclaw.ai/channels/zalo#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Regional platforms

Zalo

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Bundled plugin](https://docs.openclaw.ai/channels/zalo#bundled-plugin)
- [Quick setup (beginner)](https://docs.openclaw.ai/channels/zalo#quick-setup-beginner)
- [What it is](https://docs.openclaw.ai/channels/zalo#what-it-is)
- [Setup (fast path)](https://docs.openclaw.ai/channels/zalo#setup-fast-path)
- [1) Create a bot token (Zalo Bot Platform)](https://docs.openclaw.ai/channels/zalo#1-create-a-bot-token-zalo-bot-platform)
- [2) Configure the token (env or config)](https://docs.openclaw.ai/channels/zalo#2-configure-the-token-env-or-config)
- [How it works (behavior)](https://docs.openclaw.ai/channels/zalo#how-it-works-behavior)
- [Limits](https://docs.openclaw.ai/channels/zalo#limits)
- [Access control (DMs)](https://docs.openclaw.ai/channels/zalo#access-control-dms)
- [DM access](https://docs.openclaw.ai/channels/zalo#dm-access)
- [Access control (Groups)](https://docs.openclaw.ai/channels/zalo#access-control-groups)
- [Long-polling vs webhook](https://docs.openclaw.ai/channels/zalo#long-polling-vs-webhook)
- [Supported message types](https://docs.openclaw.ai/channels/zalo#supported-message-types)
- [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities)
- [Delivery targets (CLI/cron)](https://docs.openclaw.ai/channels/zalo#delivery-targets-cli%2Fcron)
- [Troubleshooting](https://docs.openclaw.ai/channels/zalo#troubleshooting)
- [Configuration reference (Zalo)](https://docs.openclaw.ai/channels/zalo#configuration-reference-zalo)
- [Related](https://docs.openclaw.ai/channels/zalo#related)

Status: experimental. DMs are supported. The [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities) section below reflects current Marketplace-bot behavior.

## [​](https://docs.openclaw.ai/channels/zalo\\#bundled-plugin)  Bundled plugin

Zalo ships as a bundled plugin in current OpenClaw releases, so normal packaged
builds do not need a separate install.If you are on an older build or a custom install that excludes Zalo, install it
manually:

- Install via CLI: `openclaw plugins install @openclaw/zalo`
- Or from a source checkout: `openclaw plugins install ./path/to/local/zalo-plugin`
- Details: [Plugins](https://docs.openclaw.ai/tools/plugin)

## [​](https://docs.openclaw.ai/channels/zalo\\#quick-setup-beginner)  Quick setup (beginner)

1. Ensure the Zalo plugin is available.
   - Current packaged OpenClaw releases already bundle it.
   - Older/custom installs can add it manually with the commands above.
2. Set the token:
   - Env: `ZALO_BOT_TOKEN=...`
   - Or config: `channels.zalo.accounts.default.botToken: \"...\"`.
3. Restart the gateway (or finish setup).
4. DM access is pairing by default; approve the pairing code on first contact.

Minimal config:

```
{
  channels: {
    zalo: {
      enabled: true,
      accounts: {
        default: {
          botToken: \"12345689:abc-xyz\",
          dmPolicy: \"pairing\",
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/channels/zalo\\#what-it-is)  What it is

Zalo is a Vietnam-focused messaging app; its Bot API lets the Gateway run a bot for 1:1 conversations.
It is a good fit for support or notifications where you want deterministic routing back to Zalo.This page reflects current OpenClaw behavior for **Zalo Bot Creator / Marketplace bots**.
**Zalo Official Account (OA) bots** are a different Zalo product surface and may behave differently.

- A Zalo Bot API channel owned by the Gateway.
- Deterministic routing: replies go back to Zalo; the model never chooses channels.
- DMs share the agent’s main session.
- The [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities) section below shows current Marketplace-bot support.

## [​](https://docs.openclaw.ai/channels/zalo\\#setup-fast-path)  Setup (fast path)

### [​](https://docs.openclaw.ai/channels/zalo\\#1-create-a-bot-token-zalo-bot-platform)  1) Create a bot token (Zalo Bot Platform)

1. Go to [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com/) and sign in.
2. Create a new bot and configure its settings.
3. Copy the full bot token (typically `numeric_id:secret`). For Marketplace bots, the usable runtime token may appear in the bot’s welcome message after creation.

### [​](https://docs.openclaw.ai/channels/zalo\\#2-configure-the-token-env-or-config)  2) Configure the token (env or config)

Example:

```
{
  channels: {
    zalo: {
      enabled: true,
      accounts: {
        default: {
          botToken: \"12345689:abc-xyz\",
          dmPolicy: \"pairing\",
        },
      },
    },
  },
}
```

If you later move to a Zalo bot surface where groups are available, you can add group-specific config such as `groupPolicy` and `groupAllowFrom` explicitly. For current Marketplace-bot behavior, see [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities).Env option: `ZALO_BOT_TOKEN=...` (works for the default account only).Multi-account support: use `channels.zalo.accounts` with per-account tokens and optional `name`.

3. Restart the gateway. Zalo starts when a token is resolved (env or config).
4. DM access defaults to pairing. Approve the code when the bot is first contacted.

## [​](https://docs.openclaw.ai/channels/zalo\\#how-it-works-behavior)  How it works (behavior)

- Inbound messages are normalized into the shared channel envelope with media placeholders.
- Replies always route back to the same Zalo chat.
- Long-polling by default; webhook mode available with `channels.zalo.webhookUrl`.

## [​](https://docs.openclaw.ai/channels/zalo\\#limits)  Limits

- Outbound text is chunked to 2000 characters (Zalo API limit).
- Media downloads/uploads are capped by `channels.zalo.mediaMaxMb` (default 5).
- Streaming is blocked by default due to the 2000 char limit making streaming less useful.

## [​](https://docs.openclaw.ai/channels/zalo\\#access-control-dms)  Access control (DMs)

### [​](https://docs.openclaw.ai/channels/zalo\\#dm-access)  DM access

- Default: `channels.zalo.dmPolicy = \"pairing\"`. Unknown senders receive a pairing code; messages are ignored until approved (codes expire after 1 hour).
- Approve via:
  - `openclaw pairing list zalo`
  - `openclaw pairing approve zalo <CODE>`
- Pairing is the default token exchange. Details: [Pairing](https://docs.openclaw.ai/channels/pairing)
- `channels.zalo.allowFrom` accepts numeric user IDs (no username lookup available).

## [​](https://docs.openclaw.ai/channels/zalo\\#access-control-groups)  Access control (Groups)

For **Zalo Bot Creator / Marketplace bots**, group support was not available in practice because the bot could not be added to a group at all.That means the group-related config keys below exist in the schema, but were not usable for Marketplace bots:

- `channels.zalo.groupPolicy` controls group inbound handling: `open | allowlist | disabled`.
- `channels.zalo.groupAllowFrom` restricts which sender IDs can trigger the bot in groups.
- If `groupAllowFrom` is unset, Zalo falls back to `allowFrom` for sender checks.
- Runtime note: if `channels.zalo` is missing entirely, runtime still falls back to `groupPolicy=\"allowlist\"` for safety.

The group policy values (when group access is available on your bot surface) are:

- `groupPolicy: \"disabled\"` — blocks all group messages.
- `groupPolicy: \"open\"` — allows any group member (mention-gated).
- `groupPolicy: \"allowlist\"` — fail-closed default; only allowed senders are accepted.

If you are using a different Zalo bot product surface and have verified working group behavior, document that separately rather than assuming it matches the Marketplace-bot flow.

## [​](https://docs.openclaw.ai/channels/zalo\\#long-polling-vs-webhook)  Long-polling vs webhook

- Default: long-polling (no public URL required).
- Webhook mode: set `channels.zalo.webhookUrl` and `channels.zalo.webhookSecret`.

  - The webhook secret must be 8-256 characters.
  - Webhook URL must use HTTPS.
  - Zalo sends events with `X-Bot-Api-Secret-Token` header for verification.
  - Gateway HTTP handles webhook requests at `channels.zalo.webhookPath` (defaults to the webhook URL path).
  - Requests must use `Content-Type: application/json` (or `+json` media types).
  - Duplicate events (`event_name + message_id`) are ignored for a short replay window.
  - Burst traffic is rate-limited per path/source and may return HTTP 429.

**Note:** getUpdates (polling) and webhook are mutually exclusive per Zalo API docs.

## [​](https://docs.openclaw.ai/channels/zalo\\#supported-message-types)  Supported message types

For a quick support snapshot, see [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities). The notes below add detail where the behavior needs extra context.

- **Text messages**: Full support with 2000 character chunking.
- **Plain URLs in text**: Behave like normal text input.
- **Link previews / rich link cards**: See the Marketplace-bot status in [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities); they did not reliably trigger a reply.
- **Image messages**: See the Marketplace-bot status in [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities); inbound image handling was unreliable (typing indicator without a final reply).
- **Stickers**: See the Marketplace-bot status in [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities).
- **Voice notes / audio files / video / generic file attachments**: See the Marketplace-bot status in [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities).
- **Unsupported types**: Logged (for example, messages from protected users).

## [​](https://docs.openclaw.ai/channels/zalo\\#capabilities)  Capabilities

This table summarizes current **Zalo Bot Creator / Marketplace bot** behavior in OpenClaw.

| Feature | Status |
| --- | --- |
| Direct messages | ✅ Supported |
| Groups | ❌ Not available for Marketplace bots |
| Media (inbound images) | ⚠️ Limited / verify in your environment |
| Media (outbound images) | ⚠️ Not re-tested for Marketplace bots |
| Plain URLs in text | ✅ Supported |
| Link previews | ⚠️ Unreliable for Marketplace bots |
| Reactions | ❌ Not supported |
| Stickers | ⚠️ No agent reply for Marketplace bots |
| Voice notes / audio / video | ⚠️ No agent reply for Marketplace bots |
| File attachments | ⚠️ No agent reply for Marketplace bots |
| Threads | ❌ Not supported |
| Polls | ❌ Not supported |
| Native commands | ❌ Not supported |
| Streaming | ⚠️ Blocked (2000 char limit) |

## [​](https://docs.openclaw.ai/channels/zalo\\#delivery-targets-cli/cron)  Delivery targets (CLI/cron)

- Use a chat id as the target.
- Example: `openclaw message send --channel zalo --target 123456789 --message \"hi\"`.

## [​](https://docs.openclaw.ai/channels/zalo\\#troubleshooting)  Troubleshooting

**Bot doesn’t respond:**

- Check that the token is valid: `openclaw channels status --probe`
- Verify the sender is approved (pairing or allowFrom)
- Check gateway logs: `openclaw logs --follow`

**Webhook not receiving events:**

- Ensure webhook URL uses HTTPS
- Verify secret token is 8-256 characters
- Confirm the gateway HTTP endpoint is reachable on the configured path
- Check that getUpdates polling is not running (they’re mutually exclusive)

## [​](https://docs.openclaw.ai/channels/zalo\\#configuration-reference-zalo)  Configuration reference (Zalo)

Full configuration: [Configuration](https://docs.openclaw.ai/gateway/configuration)The flat top-level keys (`channels.zalo.botToken`, `channels.zalo.dmPolicy`, and similar) are a legacy single-account shorthand. Prefer `channels.zalo.accounts.<id>.*` for new configs. Both forms are still documented here because they exist in the schema.Provider options:

- `channels.zalo.enabled`: enable/disable channel startup.
- `channels.zalo.botToken`: bot token from Zalo Bot Platform.
- `channels.zalo.tokenFile`: read token from a regular file path. Symlinks are rejected.
- `channels.zalo.dmPolicy`: `pairing | allowlist | open | disabled` (default: pairing).
- `channels.zalo.allowFrom`: DM allowlist (user IDs). `open` requires `\"*\"`. The wizard will ask for numeric IDs.
- `channels.zalo.groupPolicy`: `open | allowlist | disabled` (default: allowlist). Present in config; see [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities) and [Access control (Groups)](https://docs.openclaw.ai/channels/zalo#access-control-groups) for current Marketplace-bot behavior.
- `channels.zalo.groupAllowFrom`: group sender allowlist (user IDs). Falls back to `allowFrom` when unset.
- `channels.zalo.mediaMaxMb`: inbound/outbound media cap (MB, default 5).
- `channels.zalo.webhookUrl`: enable webhook mode (HTTPS required).
- `channels.zalo.webhookSecret`: webhook secret (8-256 chars).
- `channels.zalo.webhookPath`: webhook path on the gateway HTTP server.
- `channels.zalo.proxy`: proxy URL for API requests.

Multi-account options:

- `channels.zalo.accounts.<id>.botToken`: per-account token.
- `channels.zalo.accounts.<id>.tokenFile`: per-account regular token file. Symlinks are rejected.
- `channels.zalo.accounts.<id>.name`: display name.
- `channels.zalo.accounts.<id>.enabled`: enable/disable account.
- `channels.zalo.accounts.<id>.dmPolicy`: per-account DM policy.
- `channels.zalo.accounts.<id>.allowFrom`: per-account allowlist.
- `channels.zalo.accounts.<id>.groupPolicy`: per-account group policy. Present in config; see [Capabilities](https://docs.openclaw.ai/channels/zalo#capabilities) and [Access control (Groups)](https://docs.openclaw.ai/channels/zalo#access-control-groups) for current Marketplace-bot behavior.
- `channels.zalo.accounts.<id>.groupAllowFrom`: per-account group sender allowlist.
- `channels.zalo.accounts.<id>.webhookUrl`: per-account webhook URL.
- `channels.zalo.accounts.<id>.webhookSecret`: per-account webhook secret.
- `channels.zalo.accounts.<id>.webhookPath`: per-account webhook path.
- `channels.zalo.accounts.<id>.proxy`: per-account proxy URL.

## [​](https://docs.openclaw.ai/channels/zalo\\#related)  Related

- [Channels Overview](https://docs.openclaw.ai/channels) — all supported channels
- [Pairing](https://docs.openclaw.ai/channels/pairing) — DM authentication and pairing flow
- [Groups](https://docs.openclaw.ai/channels/groups) — group chat behavior and mention gating
- [Channel Routing](https://docs.openclaw.ai/channels/channel-routing) — session routing for messages
- [Security](https://docs.openclaw.ai/gateway/security) — access model and hardening

[Feishu](https://docs.openclaw.ai/channels/feishu) [Zalo personal](https://docs.openclaw.ai/channels/zalouser)

Ctrl+I

---

## Telegram Channel - OpenClaw Documentation
**Source:** https://docs.openclaw.ai/channels/telegram

[Skip to main content](https://docs.openclaw.ai/channels/telegram#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nMainstream messaging\n\nTelegram\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Quick setup](https://docs.openclaw.ai/channels/telegram#quick-setup)\n- [Telegram side settings](https://docs.openclaw.ai/channels/telegram#telegram-side-settings)\n- [Access control and activation](https://docs.openclaw.ai/channels/telegram#access-control-and-activation)\n- [Finding your Telegram user ID](https://docs.openclaw.ai/channels/telegram#finding-your-telegram-user-id)\n- [Runtime behavior](https://docs.openclaw.ai/channels/telegram#runtime-behavior)\n- [Feature reference](https://docs.openclaw.ai/channels/telegram#feature-reference)\n- [Error reply controls](https://docs.openclaw.ai/channels/telegram#error-reply-controls)\n- [Troubleshooting](https://docs.openclaw.ai/channels/telegram#troubleshooting)\n- [Configuration reference](https://docs.openclaw.ai/channels/telegram#configuration-reference)\n- [Related](https://docs.openclaw.ai/channels/telegram#related)\n\nProduction-ready for bot DMs and groups via grammY. Long polling is the default mode; webhook mode is optional.\n\n[**Pairing** \\\\\n\\\\\nDefault DM policy for Telegram is pairing.](https://docs.openclaw.ai/channels/pairing)\n\n[**Channel troubleshooting** \\\\\n\\\\\nCross-channel diagnostics and repair playbooks.](https://docs.openclaw.ai/channels/troubleshooting)\n\n[**Gateway configuration** \\\\\n\\\\\nFull channel config patterns and examples.](https://docs.openclaw.ai/gateway/configuration)\n\n## [​](https://docs.openclaw.ai/channels/telegram\\#quick-setup)  Quick setup\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/channels/telegram#)\n\nCreate the bot token in BotFather\n\nOpen Telegram and chat with **@BotFather** (confirm the handle is exactly `@BotFather`).Run `/newbot`, follow prompts, and save the token.\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/channels/telegram#)\n\nConfigure token and DM policy\n\n```\n{\n  channels: {\n    telegram: {\n      enabled: true,\n      botToken: \"123:abc\",\n      dmPolicy: \"pairing\",\n      groups: { \"*\": { requireMention: true } },\n    },\n  },\n}\n```\n\nEnv fallback: `TELEGRAM_BOT_TOKEN=...` (default account only).\nTelegram does **not** use `openclaw channels login telegram`; configure token in config/env, then start gateway.\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/channels/telegram#)\n\nStart gateway and approve first DM\n\n```\nopenclaw gateway\nopenclaw pairing list telegram\nopenclaw pairing approve telegram <CODE>\n```\n\nPairing codes expire after 1 hour.\n\n4\n\n[Navigate to header](https://docs.openclaw.ai/channels/telegram#)\n\nAdd the bot to a group\n\nAdd the bot to your group, then set `channels.telegram.groups` and `groupPolicy` to match your access model.\n\nToken resolution order is account-aware. In practice, config values win over env fallback, and `TELEGRAM_BOT_TOKEN` only applies to the default account.\n\n## [​](https://docs.openclaw.ai/channels/telegram\\#telegram-side-settings)  Telegram side settings\n\nPrivacy mode and group visibility\n\nTelegram bots default to **Privacy Mode**, which limits what group messages they receive.If the bot must see all group messages, either:\n\n- disable privacy mode via `/setprivacy`, or\n- make the bot a group admin.\n\nWhen toggling privacy mode, remove + re-add the bot in each group so Telegram applies the change.\n\nGroup permissions\n\nAdmin status is controlled in Telegram group settings.Admin bots receive all group messages, which is useful for always-on group behavior.\n\nHelpful BotFather toggles\n\n- `/setjoingroups` to allow/deny group adds\n- `/setprivacy` for group visibility behavior\n\n## [​](https://docs.openclaw.ai/channels/telegram\\#access-control-and-activation)  Access control and activation\n\n- DM policy\n\n- Group policy and allowlists\n\n- Mention behavior\n\n\n`channels.telegram.dmPolicy` controls direct message access:\n\n- `pairing` (default)\n- `allowlist` (requires at least one sender ID in `allowFrom`)\n- `open` (requires `allowFrom` to include `\"*\"`)\n- `disabled`\n\n`channels.telegram.allowFrom` accepts numeric Telegram user IDs. `telegram:` / `tg:` prefixes are accepted and normalized.\n`dmPolicy: \"allowlist\"` with empty `allowFrom` blocks all DMs and is rejected by config validation.\nSetup asks for numeric user IDs only.\nIf you upgraded and your config contains `@username` allowlist entries, run `openclaw doctor --fix` to resolve them (best-effort; requires a Telegram bot token).\nIf you previously relied on pairing-store allowlist files, `openclaw doctor --fix` can recover entries into `channels.telegram.allowFrom` in allowlist flows (for example when `dmPolicy: \"allowlist\"` has no explicit IDs yet).For one-owner bots, prefer `dmPolicy: \"allowlist\"` with explicit numeric `allowFrom` IDs to keep access policy durable in config (instead of depending on previous pairing approvals).Common confusion: DM pairing approval does not mean “this sender is authorized everywhere”.\nPairing grants DM access only. Group sender authorization still comes from explicit config allowlists.\nIf you want “I am authorized once and both DMs and group commands work”, put your numeric Telegram user ID in `channels.telegram.allowFrom`.\n\n### [​](https://docs.openclaw.ai/channels/telegram\\#finding-your-telegram-user-id)  Finding your Telegram user ID\n\nSafer (no third-party bot):\n\n1. DM your bot.\n2. Run `openclaw logs --follow`.\n3. Read `from.id`.\n\nOfficial Bot API method:\n\n```\ncurl \"https://api.telegram.org/bot<bot_token>/getUpdates\"\n```\n\nThird-party method (less private): `@userinfobot` or `@getidsbot`.\n\nTwo controls apply together:\n\n1. **Which groups are allowed** (`channels.telegram.groups`)   - no `groups` config:\n\n     - with `groupPolicy: \"open\"`: any group can pass group-ID checks\n     - with `groupPolicy: \"allowlist\"` (default): groups are blocked until you add `groups` entries (or `\"*\"`)\n   - `groups` configured: acts as allowlist (explicit IDs or `\"*\"`)\n2. **Which senders are allowed in groups** (`channels.telegram.groupPolicy`)   - `open`\n   - `allowlist` (default)\n   - `disabled`\n\n`groupAllowFrom` is used for group sender filtering. If not set, Telegram falls back to `allowFrom`.\n`groupAllowFrom` entries should be numeric Telegram user IDs (`telegram:` / `tg:` prefixes are normalized).\nDo not put Telegram group or supergroup chat IDs in `groupAllowFrom`. Negative chat IDs belong under `channels.telegram.groups`.\nNon-numeric entries are ignored for sender authorization.\nSecurity boundary (`2026.2.25+`): group sender auth does **not** inherit DM pairing-store approvals.\nPairing stays DM-only. For groups, set `groupAllowFrom` or per-group/per-topic `allowFrom`.\nIf `groupAllowFrom` is unset, Telegram falls back to config `allowFrom`, not the pairing store.\nPractical pattern for one-owner bots: set your user ID in `channels.telegram.allowFrom`, leave `groupAllowFrom` unset, and allow the target groups under `channels.telegram.groups`.\nRuntime note: if `channels.telegram` is completely missing, runtime defaults to fail-closed `groupPolicy=\"allowlist\"` unless `channels.defaults.groupPolicy` is explicitly set.Example: allow any member in one specific group:\n\n```\n{\n  channels: {\n    telegram: {\n      groups: {\n        \"-1001234567890\": {\n          groupPolicy: \"open\",\n          requireMention: false,\n        },\n      },\n    },\n  },\n}\n```\n\nExample: allow only specific users inside one specific group:\n\n```\n{\n  channels: {\n    telegram: {\n      groups: {\n        \"-1001234567890\": {\n          requireMention: true,\n          allowFrom: [\"8734062810\", \"745123456\"],\n        },\n      },\n    },\n  },\n}\n```\n\nCommon mistake: `groupAllowFrom` is not a Telegram group allowlist.\n\n- Put negative Telegram group or supergroup chat IDs like `-1001234567890` under `channels.telegram.groups`.\n- Put Telegram user IDs like `8734062810` under `groupAllowFrom` when you want to limit which people inside an allowed group can trigger the bot.\n- Use `groupAllowFrom: [\"*\"]` only when you want any member of an allowed group to be able to talk to the bot.\n\nGroup replies require mention by default.Mention can come from:\n\n- native `@botusername` mention, or\n- mention patterns in:\n  - `agents.list[].groupChat.mentionPatterns`\n  - `messages.groupChat.mentionPatterns`\n\nSession-level command toggles:\n\n- `/activation always`\n- `/activation mention`\n\nThese update session state only. Use config for persistence.Persistent config example:\n\n```\n{\n  channels: {\n    telegram: {\n      groups: {\n        \"*\": { requireMention: false },\n      },\n    },\n  },\n}\n```\n\nGetting the group chat ID:\n\n- forward a group message to `@userinfobot` / `@getidsbot`\n- or read `chat.id` from `openclaw logs --follow`\n- or inspect Bot API `getUpdates`\n\n## [​](https://docs.openclaw.ai/channels/telegram\\#runtime-behavior)  Runtime behavior\n\n- Telegram is owned by the gateway process.\n- Routing is deterministic: Telegram inbound replies back to Telegram (the model does not pick channels).\n- Inbound messages normalize into the shared channel envelope with reply metadata and media placeholders.\n- Group sessions are isolated by group ID. Forum topics append `:topic:<threadId>` to keep topics isolated.\n- DM messages can carry `message_thread_id`; OpenClaw routes them with thread-aware session keys and preserves thread ID for replies.\n- Long polling uses grammY runner with per-chat/per-thread sequencing. Overall runner sink concurrency uses `agents.defaults.maxConcurrent`.\n- Long polling is guarded inside each gateway process so only one active poller can use a bot token at a time. If you still see `getUpdates` 409 conflicts, another OpenClaw gateway, script, or external poller is likely using the same token.\n- Long-polling watchdog restarts trigger after 120 seconds without completed `getUpdates` liveness by default. Increase `channels.telegram.pollingStallThresholdMs` only if your deployment still sees false polling-stall restarts during long-running work. The value is in milliseconds and is allowed from `30000` to `600000`; per-account overrides are supported.\n- Telegram Bot API has no read-receipt support (`sendReadReceipts` does not apply).\n\n## [​](https://docs.openclaw.ai/channels/telegram\\#feature-reference)  Feature reference\n\nLive stream preview (message edits)\n\nOpenClaw can stream partial replies in real time:\n\n- direct chats: preview message + `editMessageText`\n- groups/topics: preview message + `editMessageText`\n\nRequirement:\n\n- `channels.telegram.streaming` is `off | partial | block | progress` (default: `partial`)\n- `progress` maps to `partial` on Telegram (compat with cross-channel naming)\n- `streaming.preview.toolProgress` controls whether tool/progress updates reuse the same edited preview message (default: `true` when preview streaming is active)\n- legacy `channels.telegram.streamMode` and boolean `streaming` values are detected; run `openclaw doctor --fix` to migrate them to `channels.telegram.streaming.mode`\n\nTool-progress preview updates are the short “Working…” lines shown while tools run, for example command execution, file reads, planning updates, or patch summaries. Telegram keeps these enabled by default to match released OpenClaw behavior from `v2026.4.22` and later. To keep the edited preview for answer text but hide tool-progress lines, set:\n\n```\n{\n  \"channels\": {\n    \"telegram\": {\n      \"streaming\": {\n        \"mode\": \"partial\",\n        \"preview\": {\n          \"toolProgress\": false\n        }\n      }\n    }\n  }\n}\n```\n\nUse `streaming.mode: \"off\"` only when you want to disable Telegram preview edits entirely. Use `streaming.preview.toolProgress: false` when you only want to disable the tool-progress status lines.For text-only replies:\n\n- short DM/group/topic previews: OpenClaw keeps the same preview message and performs a final edit in place\n- previews older than about one minute: OpenClaw sends the completed reply as a fresh final message and then cleans up the preview, so Telegram’s visible timestamp reflects completion time instead of the preview creation time\n\nFor complex replies (for example media payloads), OpenClaw falls back to normal final delivery and then cleans up the preview message.Preview streaming is separate from block streaming. When block streaming is explicitly enabled for Telegram, OpenClaw skips the preview stream to avoid double-streaming.If native draft transport is unavailable/rejected, OpenClaw automatically falls back to `sendMessage` \\+ `editMessageText`.Telegram-only reasoning stream:\n\n- `/reasoning stream` sends reasoning to the live preview while generating\n- final answer is sent without reasoning text\n\nFormatting and HTML fallback\n\nOutbound text uses Telegram `parse_mode: \"HTML\"`.\n\n- Markdown-ish text is rendered to Telegram-safe HTML.\n- Raw model HTML is escaped to reduce Telegram parse failures.\n- If Telegram rejects parsed HTML, OpenClaw retries as plain text.\n\nLink previews are enabled by default and can be disabled with `channels.telegram.linkPreview: false`.\n\nNative commands and custom commands\n\nTelegram command menu registration is handled at startup with `setMyCommands`.Native command defaults:\n\n- `commands.native: \"auto\"` enables native commands for Telegram\n\nAdd custom command menu entries:\n\n```\n{\n  channels: {\n    telegram: {\n      customCommands: [\\\n        { command: \"backup\", description: \"Git backup\" },\\\n        { command: \"generate\", description: \"Create an image\" },\\\n      ],\n    },\n  },\n}\n```\n\nRules:\n\n- names are normalized (strip leading `/`, lowercase)\n- valid pattern: `a-z`, `0-9`, `_`, length `1..32`\n- custom commands cannot override native commands\n- conflicts/duplicates are skipped and logged\n\nNotes:\n\n- custom commands are menu entries only; they do not auto-implement behavior\n- plugin/skill commands can still work when typed even if not shown in Telegram menu\n\nIf native commands are disabled, built-ins are removed. Custom/plugin commands may still register if configured.Common setup failures:\n\n- `setMyCommands failed` with `BOT_COMMANDS_TOO_MUCH` means the Telegram menu still overflowed after trimming; reduce plugin/skill/custom commands or disable `channels.telegram.commands.native`.\n- `deleteWebhook`, `deleteMyCommands`, or `setMyCommands` failing with `404: Not Found` while direct Bot API curl commands work can mean `channels.telegram.apiRoot` was set to the full `/bot<TOKEN>` endpoint. `apiRoot` must be only the Bot API root, and `openclaw doctor --fix` removes an accidental trailing `/bot<TOKEN>`.\n- `getMe returned 401` means Telegram rejected the configured bot token. Update `botToken`, `tokenFile`, or `TELEGRAM_BOT_TOKEN` with the current BotFather token; OpenClaw stops before polling so this is not reported as a webhook cleanup failure.\n- `setMyCommands failed` with network/fetch errors usually means outbound DNS/HTTPS to `api.telegram.org` is blocked.\n\n### [​](https://docs.openclaw.ai/channels/telegram\\#device-pairing-commands-device-pair-plugin)  Device pairing commands (`device-pair` plugin)\n\nWhen the `device-pair` plugin is installed:\n\n1. `/pair` generates setup code\n2. paste code in iOS app\n3. `/pair pending` lists pending requests (including role/scopes)\n4. approve the request:\n   - `/pair approve <requestId>` for explicit approval\n   - `/pair approve` when there is only one pending request\n   - `/pair approve latest` for most recent\n\nThe setup code carries a short-lived bootstrap token. Built-in bootstrap handoff keeps the primary node token at `scopes: []`; any handed-off operator token stays bounded to `operator.approvals`, `operator.read`, `operator.talk.secrets`, and `operator.write`. Bootstrap scope checks are role-prefixed, so that operator allowlist only satisfies operator requests; non-operator roles still need scopes under their own role prefix.If a device retries with changed auth details (for example role/scopes/public key), the previous pending request is superseded and the new request uses a different `requestId`. Re-run `/pair pending` before approving.More details: [Pairing](https://docs.openclaw.ai/channels/pairing#pair-via-telegram-recommended-for-ios).\n\nInline buttons\n\nConfigure inline keyboard scope:\n\n```\n{\n  channels: {\n    telegram: {\n      capabilities: {\n        inlineButtons: \"allowlist\",\n      },\n    },\n  },\n}\n```\n\nPer-account override:\n\n```\n{\n  channels: {\n    telegram: {\n      accounts: {\n        main: {\n          capabilities: {\n            inlineButtons: \"allowlist\",\n          },\n        },\n      },\n    },\n  },\n}\n```\n\nScopes:\n\n- `off`\n- `dm`\n- `group`\n- `all`\n- `allowlist` (default)\n\nLegacy `capabilities: [\"inlineButtons\"]` maps to `inlineButtons: \"all\"`.Message action example:\n\n```\n{\n  action: \"send\",\n  channel: \"telegram\",\n  to: \"123456789\",\n  message: \"Choose an option:\",\n  buttons: [\\\n    [\\\n      { text: \"Yes\", callback_data: \"yes\" },\\\n      { text: \"No\", callback_data: \"no\" },\\\n    ],\\\n    [{ text: \"Cancel\", callback_data: \"cancel\" }],\\\n  ],\n}\n```\n\nCallback clicks are passed to the agent as text:\n`callback_data: <value>`\n\nTelegram message actions for agents and automation\n\nTelegram tool actions include:\n\n- `sendMessage` (`to`, `content`, optional `mediaUrl`, `replyToMessageId`, `messageThreadId`)\n- `react` (`chatId`, `messageId`, `emoji`)\n- `deleteMessage` (`chatId`, `messageId`)\n- `editMessage` (`chatId`, `messageId`, `content`)\n- `createForumTopic` (`chatId`, `name`, optional `iconColor`, `iconCustomEmojiId`)\n\nChannel message actions expose ergonomic aliases (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`, `topic-create`).Gating controls:\n\n- `channels.telegram.actions.sendMessage`\n- `channels.telegram.actions.deleteMessage`\n- `channels.telegram.actions.reactions`\n- `channels.telegram.actions.sticker` (default: disabled)\n\nNote: `edit` and `topic-create` are currently enabled by default and do not have separate `channels.telegram.actions.*` toggles.\nRuntime sends use the active config/secrets snapshot (startup/reload), so action paths do not perform ad-hoc SecretRef re-resolution per send.Reaction removal semantics: [/tools/reactions](https://docs.openclaw.ai/tools/reactions)\n\nReply threading tags\n\nTelegram supports explicit reply threading tags in generated output:\n\n- `[[reply_to_curr...

---

