> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Access groups

Access groups are named sender lists you define once and reference from channel allowlists with `accessGroup:<name>`.

Use them when the same people should be allowed across several message channels, or when one trusted set should apply to both DMs and group sender authorization.

Access groups do not grant access by themselves. A group only matters when an allowlist field references it.

## Static message sender groups

Static sender groups use `type: "message.senders"`.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  accessGroups: {
    operators: {
      type: "message.senders",
      members: {
        "*": ["global-owner-id"],
        discord: ["discord:123456789012345678"],
        telegram: ["987654321"],
        whatsapp: ["+15551234567"],
      },
    },
  },
}
```

Member lists are keyed by message-channel id:

| Key        | Meaning                                                                 |
| ---------- | ----------------------------------------------------------------------- |
| `"*"`      | Shared entries checked for every message channel that references group. |
| `discord`  | Entries checked only for Discord allowlist matching.                    |
| `telegram` | Entries checked only for Telegram allowlist matching.                   |
| `whatsapp` | Entries checked only for WhatsApp allowlist matching.                   |

Entries are matched with the destination channel's normal `allowFrom` rules. OpenClaw does not translate sender ids between channels. If Alice has a Telegram id and a Discord id, list both ids under the appropriate keys.

## Reference groups from allowlists

Reference a group with `accessGroup:<name>` anywhere the message channel path supports sender allowlists.

DM allowlist example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  accessGroups: {
    operators: {
      type: "message.senders",
      members: {
        discord: ["discord:123456789012345678"],
        telegram: ["987654321"],
      },
    },
  },
  channels: {
    discord: {
      dmPolicy: "allowlist",
      allowFrom: ["accessGroup:operators"],
    },
    telegram: {
      dmPolicy: "allowlist",
      allowFrom: ["accessGroup:operators"],
    },
  },
}
```

Group sender allowlist example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  accessGroups: {
    oncall: {
      type: "message.senders",
      members: {
        whatsapp: ["+15551234567"],
        googlechat: ["users/1234567890"],
      },
    },
  },
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["accessGroup:oncall"],
    },
    googlechat: {
      spaces: {
        "spaces/AAA": {
          users: ["accessGroup:oncall"],
        },
      },
    },
  },
}
```

You can mix groups and direct entries:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    discord: {
      dmPolicy: "allowlist",
      allowFrom: ["accessGroup:operators", "discord:123456789012345678"],
    },
  },
}
```

## Supported message-channel paths

Access groups are available in shared message-channel authorization paths, including:

* DM sender allowlists such as `channels.<channel>.allowFrom`
* group sender allowlists such as `channels.<channel>.groupAllowFrom`
* channel-specific per-room sender allowlists that use the same sender matching rules
* command authorization paths that reuse message-channel sender allowlists

Channel support depends on whether that channel is wired through the shared OpenClaw sender-authorization helpers. Current bundled support includes Discord, Google Chat, Nostr, WhatsApp, Zalo, and Zalo Personal. Static `message.senders` groups are designed to be channel-agnostic, so new message channels should support them by using the shared plugin SDK helpers instead of custom allowlist expansion.

## Discord channel audiences

Discord also supports a dynamic access group type:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  accessGroups: {
    maintainers: {
      type: "discord.channelAudience",
      guildId: "1456350064065904867",
      channelId: "1456744319972282449",
      membership: "canViewChannel",
    },
  },
  channels: {
    discord: {
      dmPolicy: "allowlist",
      allowFrom: ["accessGroup:maintainers"],
    },
  },
}
```

`discord.channelAudience` means "allow Discord DM senders who can currently view this guild channel." OpenClaw resolves the sender through Discord at authorization time and applies Discord `ViewChannel` permission rules.

Use this when a Discord channel is already the source of truth for a team, such as `#maintainers` or `#on-call`.

Requirements and failure behavior:

* The bot needs access to the guild and channel.
* The bot needs the Discord Developer Portal **Server Members Intent**.
* The access group fails closed when Discord returns `Missing Access`, the sender cannot be resolved as a guild member, or the channel belongs to another guild.

More Discord-specific examples: [Discord access control](/channels/discord#access-control-and-routing)

## Security notes

* Access groups are allowlist aliases, not roles. They do not create owners, approve pairing requests, or grant tool permissions by themselves.
* `dmPolicy: "open"` still requires `"*"` in the effective DM allowlist. Referencing an access group is not the same as public access.
* Missing group names fail closed. If `allowFrom` contains `accessGroup:operators` and `accessGroups.operators` is absent, that entry authorizes nobody.
* Keep channel ids stable. Prefer numeric/user ids over display names when the channel supports both.

## Troubleshooting

If a sender should match but is blocked:

1. Confirm the allowlist field contains the exact `accessGroup:<name>` reference.
2. Confirm `accessGroups.<name>.type` is correct.
3. Confirm the sender id is listed under the matching channel key, or under `"*"`.
4. Confirm the entry uses that channel's normal allowlist syntax.
5. For Discord channel audiences, confirm the bot can see the guild channel and has Server Members Intent enabled.

Run `openclaw doctor` after editing access-control config. It catches many invalid allowlist and policy combinations before runtime.
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Discord

Ready for DMs and guild channels via the official Discord gateway.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DMs default to pairing mode.
  </Card>

  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Native command behavior and command catalog.
  </Card>

  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Cross-channel diagnostics and repair flow.
  </Card>
</CardGroup>

## Quick setup

You will need to create a new application with a bot, add the bot to your server, and pair it to OpenClaw. We recommend adding your bot to your own private server. If you don't have one yet, [create one first](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server) (choose **Create My Own > For me and my friends**).

<Steps>
  <Step title="Create a Discord application and bot">
    Go to the [Discord Developer Portal](https://discord.com/developers/applications) and click **New Application**. Name it something like "OpenClaw".

    Click **Bot** on the sidebar. Set the **Username** to whatever you call your OpenClaw agent.
  </Step>

  <Step title="Enable privileged intents">
    Still on the **Bot** page, scroll down to **Privileged Gateway Intents** and enable:

    * **Message Content Intent** (required)
    * **Server Members Intent** (recommended; required for role allowlists and name-to-ID matching)
    * **Presence Intent** (optional; only needed for presence updates)
  </Step>

  <Step title="Copy your bot token">
    Scroll back up on the **Bot** page and click **Reset Token**.

    <Note>
      Despite the name, this generates your first token — nothing is being "reset."
    </Note>

    Copy the token and save it somewhere. This is your **Bot Token** and you will need it shortly.
  </Step>

  <Step title="Generate an invite URL and add the bot to your server">
    Click **OAuth2** on the sidebar. You'll generate an invite URL with the right permissions to add the bot to your server.

    Scroll down to **OAuth2 URL Generator** and enable:

    * `bot`
    * `applications.commands`

    A **Bot Permissions** section will appear below. Enable at least:

    **General Permissions**

    * View Channels
      **Text Permissions**
    * Send Messages
    * Read Message History
    * Embed Links
    * Attach Files
    * Add Reactions (optional)

    This is the baseline set for normal text channels. If you plan to post in Discord threads, including forum or media channel workflows that create or continue a thread, also enable **Send Messages in Threads**.
    Copy the generated URL at the bottom, paste it into your browser, select your server, and click **Continue** to connect. You should now see your bot in the Discord server.
  </Step>

  <Step title="Enable Developer Mode and collect your IDs">
    Back in the Discord app, you need to enable Developer Mode so you can copy internal IDs.

    1. Click **User Settings** (gear icon next to your avatar) → **Advanced** → toggle on **Developer Mode**
    2. Right-click your **server icon** in the sidebar → **Copy Server ID**
    3. Right-click your **own avatar** → **Copy User ID**

    Save your **Server ID** and **User ID** alongside your Bot Token — you'll send all three to OpenClaw in the next step.
  </Step>

  <Step title="Allow DMs from server members">
    For pairing to work, Discord needs to allow your bot to DM you. Right-click your **server icon** → **Privacy Settings** → toggle on **Direct Messages**.

    This lets server members (including bots) send you DMs. Keep this enabled if you want to use Discord DMs with OpenClaw. If you only plan to use guild channels, you can disable DMs after pairing.
  </Step>

  <Step title="Set your bot token securely (do not send it in chat)">
    Your Discord bot token is a secret (like a password). Set it on the machine running OpenClaw before messaging your agent.

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    export DISCORD_BOT_TOKEN="YOUR_BOT_TOKEN"
    cat > discord.patch.json5 <<'JSON5'
    {
      channels: {
        discord: {
          enabled: true,
          token: { source: "env", provider: "default", id: "DISCORD_BOT_TOKEN" },
        },
      },
    }
    JSON5
    openclaw config patch --file ./discord.patch.json5 --dry-run
    openclaw config patch --file ./discord.patch.json5
    openclaw gateway
    ```

    If OpenClaw is already running as a background service, restart it via the OpenClaw Mac app or by stopping and restarting the `openclaw gateway run` process.
    For managed service installs, run `openclaw gateway install` from a shell where `DISCORD_BOT_TOKEN` is present, or store the variable in `~/.openclaw/.env`, so the service can resolve the env SecretRef after restart.
    If your host is blocked or rate-limited by Discord's startup application lookup, set the Discord application/client ID from the Developer Portal so startup can skip that REST call. Use `channels.discord.applicationId` for the default account, or `channels.discord.accounts.<accountId>.applicationId` when you run multiple Discord bots.
  </Step>

  <Step title="Configure OpenClaw and pair">
    <Tabs>
      <Tab title="Ask your agent">
        Chat with your OpenClaw agent on any existing channel (e.g. Telegram) and tell it. If Discord is your first channel, use the CLI / config tab instead.

        > "I already set my Discord bot token in config. Please finish Discord setup with User ID `<user_id>` and Server ID `<server_id>`."
      </Tab>

      <Tab title="CLI / config">
        If you prefer file-based config, set:

        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          channels: {
            discord: {
              enabled: true,
              token: {
                source: "env",
                provider: "default",
                id: "DISCORD_BOT_TOKEN",
              },
            },
          },
        }
        ```

        Env fallback for the default account:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        DISCORD_BOT_TOKEN=...
        ```

        For scripted or remote setup, write the same JSON5 block with `openclaw config patch --file ./discord.patch.json5 --dry-run` and then rerun without `--dry-run`. Plaintext `token` values are supported. SecretRef values are also supported for `channels.discord.token` across env/file/exec providers. See [Secrets Management](/gateway/secrets).

        For multiple Discord bots, keep each bot token and application ID under its account. A top-level `channels.discord.applicationId` is inherited by accounts, so only set it there when every account should use the same application ID.

        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          channels: {
            discord: {
              enabled: true,
              accounts: {
                personal: {
                  token: { source: "env", provider: "default", id: "DISCORD_PERSONAL_TOKEN" },
                  applicationId: "111111111111111111",
                },
                work: {
                  token: { source: "env", provider: "default", id: "DISCORD_WORK_TOKEN" },
                  applicationId: "222222222222222222",
                },
              },
            },
          },
        }
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="Approve first DM pairing">
    Wait until the gateway is running, then DM your bot in Discord. It will respond with a pairing code.

    <Tabs>
      <Tab title="Ask your agent">
        Send the pairing code to your agent on your existing channel:

        > "Approve this Discord pairing code: `<CODE>`"
      </Tab>

      <Tab title="CLI">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw pairing list discord
        openclaw pairing approve discord <CODE>
        ```
      </Tab>
    </Tabs>

    Pairing codes expire after 1 hour.

    You should now be able to chat with your agent in Discord via DM.
  </Step>
</Steps>

<Note>
  Token resolution is account-aware. Config token values win over env fallback. `DISCORD_BOT_TOKEN` is only used for the default account.
  If two enabled Discord accounts resolve to the same bot token, OpenClaw starts only one gateway monitor for that token. A config-sourced token wins over the default env fallback; otherwise the first enabled account wins and the duplicate account is reported disabled.
  For advanced outbound calls (message tool/channel actions), an explicit per-call `token` is used for that call. This applies to send and read/probe-style actions (for example read/search/fetch/thread/pins/permissions). Account policy/retry settings still come from the selected account in the active runtime snapshot.
</Note>

## Recommended: Set up a guild workspace

Once DMs are working, you can set up your Discord server as a full workspace where each channel gets its own agent session with its own context. This is recommended for private servers where it's just you and your bot.

<Steps>
  <Step title="Add your server to the guild allowlist">
    This enables your agent to respond in any channel on your server, not just DMs.

    <Tabs>
      <Tab title="Ask your agent">
        > "Add my Discord Server ID `<server_id>` to the guild allowlist"
      </Tab>

      <Tab title="Config">
        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          channels: {
            discord: {
              groupPolicy: "allowlist",
              guilds: {
                YOUR_SERVER_ID: {
                  requireMention: true,
                  users: ["YOUR_USER_ID"],
                },
              },
            },
          },
        }
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="Allow responses without @mention">
    By default, your agent only responds in guild channels when @mentioned. For a private server, you probably want it to respond to every message.

    In guild channels, normal assistant final replies stay private by default. Visible Discord output must be sent explicitly with the `message` tool, so the agent can lurk by default and only post when it decides a channel reply is useful.

    This means the selected model must reliably call tools. If Discord shows typing and the logs show token usage but no posted message, check the session log for assistant text with `didSendViaMessagingTool: false`. That means the model produced a private final answer instead of calling `message(action=send)`. Switch to a stronger tool-calling model, or use the config below to restore legacy automatic final replies.

    <Tabs>
      <Tab title="Ask your agent">
        > "Allow my agent to respond on this server without having to be @mentioned"
      </Tab>

      <Tab title="Config">
        Set `requireMention: false` in your guild config:

        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          channels: {
            discord: {
              guilds: {
                YOUR_SERVER_ID: {
                  requireMention: false,
                },
              },
            },
          },
        }
        ```

        To restore legacy automatic final replies for group/channel rooms, set `messages.groupChat.visibleReplies: "automatic"`.
      </Tab>
    </Tabs>
  </Step>

  <Step title="Plan for memory in guild channels">
    By default, long-term memory (MEMORY.md) only loads in DM sessions. Guild channels do not auto-load MEMORY.md.

    <Tabs>
      <Tab title="Ask your agent">
        > "When I ask questions in Discord channels, use memory\_search or memory\_get if you need long-term context from MEMORY.md."
      </Tab>

      <Tab title="Manual">
        If you need shared context in every channel, put the stable instructions in `AGENTS.md` or `USER.md` (they are injected for every session). Keep long-term notes in `MEMORY.md` and access them on demand with memory tools.
      </Tab>
    </Tabs>
  </Step>
</Steps>

Now create some channels on your Discord server and start chatting. Your agent can see the channel name, and each channel gets its own isolated session — so you can set up `#coding`, `#home`, `#research`, or whatever fits your workflow.

## Runtime model

* Gateway owns the Discord connection.
* Reply routing is deterministic: Discord inbound replies back to Discord.
* Discord guild/channel metadata is added to the model prompt as untrusted
  context, not as a user-visible reply prefix. If a model copies that envelope
  back, OpenClaw strips the copied metadata from outbound replies and from
  future replay context.
* By default (`session.dmScope=main`), direct chats share the agent main session (`agent:main:main`).
* Guild channels are isolated session keys (`agent:<agentId>:discord:channel:<channelId>`).
* Group DMs are ignored by default (`channels.discord.dm.groupEnabled=false`).
* Native slash commands run in isolated command sessions (`agent:<agentId>:discord:slash:<userId>`), while still carrying `CommandTargetSessionKey` to the routed conversation session.
* Text-only cron/heartbeat announce delivery to Discord uses the final
  assistant-visible answer once. Media and structured component payloads remain
  multi-message when the agent emits multiple deliverable payloads.

## Forum channels

Discord forum and media channels only accept thread posts. OpenClaw supports two ways to create them:

* Send a message to the forum parent (`channel:<forumId>`) to auto-create a thread. The thread title uses the first non-empty line of your message.
* Use `openclaw message thread create` to create a thread directly. Do not pass `--message-id` for forum channels.

Example: send to forum parent to create a thread

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw message send --channel discord --target channel:<forumId> \
  --message "Topic title\nBody of the post"
```

Example: create a forum thread explicitly

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw message thread create --channel discord --target channel:<forumId> \
  --thread-name "Topic title" --message "Body of the post"
```

Forum parents do not accept Discord components. If you need components, send to the thread itself (`channel:<threadId>`).

## Interactive components

OpenClaw supports Discord components v2 containers for agent messages. Use the message tool with a `components` payload. Interaction results are routed back to the agent as normal inbound messages and follow the existing Discord `replyToMode` settings.

Supported blocks:

* `text`, `section`, `separator`, `actions`, `media-gallery`, `file`
* Action rows allow up to 5 buttons or a single select menu
* Select types: `string`, `user`, `role`, `mentionable`, `channel`

By default, components are single use. Set `components.reusable=true` to allow buttons, selects, and forms to be used multiple times until they expire.

To restrict who can click a button, set `allowedUsers` on that button (Discord user IDs, tags, or `*`). When configured, unmatched users receive an ephemeral denial.

The `/model` and `/models` slash commands open an interactive model picker with provider, model, and compatible runtime dropdowns plus a Submit step. `/models add` is deprecated and now returns a deprecation message instead of registering models from chat. The picker reply is ephemeral and only the invoking user can use it.

File attachments:

* `file` blocks must point to an attachment reference (`attachment://<filename>`)
* Provide the attachment via `media`/`path`/`filePath` (single file); use `media-gallery` for multiple files
* Use `filename` to override the upload name when it should match the attachment reference

Modal forms:

* Add `components.modal` with up to 5 fields
* Field types: `text`, `checkbox`, `radio`, `select`, `role-select`, `user-select`
* OpenClaw adds a trigger button automatically

Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channel: "discord",
  action: "send",
  to: "channel:123456789012345678",
  message: "Optional fallback text",
  components: {
    reusable: true,
    text: "Choose a path",
    blocks: [
      {
        type: "actions",
        buttons: [
          {
            label: "Approve",
            style: "success",
            allowedUsers: ["123456789012345678"],
          },
          { label: "Decline", style: "danger" },
        ],
      },
      {
        type: "actions",
        select: {
          type: "string",
          placeholder: "Pick an option",
          options: [
            { label: "Option A", value: "a" },
            { label: "Option B", value: "b" },
          ],
        },
      },
    ],
    modal: {
      title: "Details",
      triggerLabel: "Open form",
      fields: [
        { type: "text", label: "Requester" },
        {
          type: "select",
          label: "Priority",
          options: [
            { label: "Low", value: "low" },
            { label: "High", value: "high" },
          ],
        },
      ],
    },
  },
}
```

## Access control and routing

<Tabs>
  <Tab title="DM policy">
    `channels.discord.dmPolicy` controls DM access. `channels.discord.allowFrom` is the canonical DM allowlist.

    * `pairing` (default)
    * `allowlist`
    * `open` (requires `channels.discord.allowFrom` to include `"*"`)
    * `disabled`

    If DM policy is not open, unknown users are blocked (or prompted for pairing in `pairing` mode).

    Multi-account precedence:

    * `channels.discord.accounts.default.allowFrom` applies only to the `default` account.
    * For one account, `allowFrom` takes precedence over legacy `dm.allowFrom`.
    * Named accounts inherit `channels.discord.allowFrom` when their own `allowFrom` and legacy `dm.allowFrom` are unset.
    * Named accounts do not inherit `channels.discord.accounts.default.allowFrom`.

    Legacy `channels.discord.dm.policy` and `channels.discord.dm.allowFrom` still read for compatibility. `openclaw doctor --fix` migrates them to `dmPolicy` and `allowFrom` when it can do so without changing access.

    DM target format for delivery:

    * `user:<id>`
    * `<@id>` mention

    Bare numeric IDs normally resolve as channel IDs when a channel default is active, but IDs listed in the account's effective DM `allowFrom` are treated as user DM targets for compatibility.
  </Tab>

  <Tab title="DM access groups">
    Discord DMs can use dynamic `accessGroup:<name>` entries in `channels.discord.allowFrom`.

    Access group names are shared across message channels. Use `type: "message.senders"` for a static group whose members are expressed in each channel's normal `allowFrom` syntax, or `type: "discord.channelAudience"` when a Discord channel's current `ViewChannel` audience should define membership dynamically. Shared access-group behavior is documented here: [Access groups](/channels/access-groups).

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      accessGroups: {
        operators: {
          type: "message.senders",
          members: {
            "*": ["global-owner-id"],
            discord: ["discord:123456789012345678"],
            telegram: ["987654321"],
          },
        },
      },
      channels: {
        discord: {
          dmPolicy: "allowlist",
          allowFrom: ["accessGroup:operators"],
        },
      },
    }
    ```

    A Discord text channel has no separate member list. `type: "discord.channelAudience"` models membership as: the DM sender is a member of the configured guild and currently has effective `ViewChannel` permission on the configured channel after role and channel overwrites are applied.

    Example: allow anyone who can see `#maintainers` to DM the bot, while keeping DMs closed to everyone else.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      accessGroups: {
        maintainers: {
          type: "discord.channelAudience",
          guildId: "1456350064065904867",
          channelId: "1456744319972282449",
          membership: "canViewChannel",
        },
      },
      channels: {
        discord: {
          dmPolicy: "allowlist",
          allowFrom: ["accessGroup:maintainers"],
        },
      },
    }
    ```

    You can mix dynamic and static entries:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      accessGroups: {
        maintainers: {
          type: "discord.channelAudience",
          guildId: "1456350064065904867",
          channelId: "1456744319972282449",
        },
      },
      channels: {
        discord: {
          dmPolicy: "allowlist",
          allowFrom: ["accessGroup:maintainers", "discord:123456789012345678"],
        },
      },
    }
    ```

    Lookups fail closed. If Discord returns `Missing Access`, the member lookup fails, or the channel belongs to a different guild, the DM sender is treated as unauthorized.

    Enable the Discord Developer Portal **Server Members Intent** for the bot when using channel-audience access groups. DMs do not include guild member state, so OpenClaw resolves the member through Discord REST at authorization time.
  </Tab>

  <Tab title="Guild policy">
    Guild handling is controlled by `channels.discord.groupPolicy`:

    * `open`
    * `allowlist`
    * `disabled`

    Secure baseline when `channels.discord` exists is `allowlist`.

    `allowlist` behavior:

    * guild must match `channels.discord.guilds` (`id` preferred, slug accepted)
    * optional sender allowlists: `users` (stable IDs recommended) and `roles` (role IDs only); if either is configured, senders are allowed when they match `users` OR `roles`
    * direct name/tag matching is disabled by default; enable `channels.discord.dangerouslyAllowNameMatching: true` only as break-glass compatibility mode
    * names/tags are supported for `users`, but IDs are safer; `openclaw security audit` warns when name/tag entries are used
    * if a guild has `channels` configured, non-listed channels are denied
    * if a guild has no `channels` block, all channels in that allowlisted guild are allowed

    Example:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          groupPolicy: "allowlist",
          guilds: {
            "123456789012345678": {
              requireMention: true,
              ignoreOtherMentions: true,
              users: ["987654321098765432"],
              roles: ["123456789012345678"],
              channels: {
                general: { allow: true },
                help: { allow: true, requireMention: true },
              },
            },
          },
        },
      },
    }
    ```

    If you only set `DISCORD_BOT_TOKEN` and do not create a `channels.discord` block, runtime fallback is `groupPolicy="allowlist"` (with a warning in logs), even if `channels.defaults.groupPolicy` is `open`.
  </Tab>

  <Tab title="Mentions and group DMs">
    Guild messages are mention-gated by default.

    Mention detection includes:

    * explicit bot mention
    * configured mention patterns (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    * implicit reply-to-bot behavior in supported cases

    When writing outbound Discord messages, use canonical mention syntax: `<@USER_ID>` for users, `<#CHANNEL_ID>` for channels, and `<@&ROLE_ID>` for roles. Do not use the legacy `<@!USER_ID>` nickname mention form.

    `requireMention` is configured per guild/channel (`channels.discord.guilds...`).
    `ignoreOtherMentions` optionally drops messages that mention another user/role but not the bot (excluding @everyone/@here).

    Group DMs:

    * default: ignored (`dm.groupEnabled=false`)
    * optional allowlist via `dm.groupChannels` (channel IDs or slugs)
  </Tab>
</Tabs>

### Role-based agent routing

Use `bindings[].match.roles` to route Discord guild members to different agents by role ID. Role-based bindings accept role IDs only and are evaluated after peer or parent-peer bindings and before guild-only bindings. If a binding also sets other match fields (for example `peer` + `guildId` + `roles`), all configured fields must match.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Native commands and command auth

* `commands.native` defaults to `"auto"` and is enabled for Discord.
* Per-channel override: `channels.discord.commands.native`.
* `commands.native=false` skips Discord slash-command registration and cleanup during startup. Previously registered commands may remain visible in Discord until you remove them from the Discord app.
* Native command auth uses the same Discord allowlists/policies as normal message handling.
* Commands may still be visible in Discord UI for users who are not authorized; execution still enforces OpenClaw auth and returns "not authorized".

See [Slash commands](/tools/slash-commands) for command catalog and behavior.

Default slash command settings:

* `ephemeral: true`

## Feature details

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord supports reply tags in agent output:

    * `[[reply_to_current]]`
    * `[[reply_to:<id>]]`

    Controlled by `channels.discord.replyToMode`:

    * `off` (default)
    * `first`
    * `all`
    * `batched`

    Note: `off` disables implicit reply threading. Explicit `[[reply_to_*]]` tags are still honored.
    `first` always attaches the implicit native reply reference to the first outbound Discord message for the turn.
    `batched` only attaches Discord's implicit native reply reference when the
    inbound turn was a debounced batch of multiple messages. This is useful
    when you want native replies mainly for ambiguous bursty chats, not every
    single-message turn.

    Message IDs are surfaced in context/history so agents can target specific messages.
  </Accordion>

  <Accordion title="Live stream preview">
    OpenClaw can stream draft replies by sending a temporary message and editing it as text arrives. `channels.discord.streaming` takes `off` (default) | `partial` | `block` | `progress`. `progress` keeps one editable status draft and updates it with tool progress until final delivery; `streamMode` is a legacy runtime alias. Run `openclaw doctor --fix` to rewrite persisted config to the canonical key.

    Default stays `off` because Discord preview edits hit rate limits quickly when multiple bots or gateways share an account.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          streaming: "block",
          draftChunk: {
            minChars: 200,
            maxChars: 800,
            breakPreference: "paragraph",
          },
        },
      },
    }
    ```

    * `partial` edits a single preview message as tokens arrive.
    * `block` emits draft-sized chunks (use `draftChunk` to tune size and breakpoints, clamped to `textChunkLimit`).
    * Media, error, and explicit-reply finals cancel pending preview edits.
    * `streaming.preview.toolProgress` (default `true`) controls whether tool/progress updates reuse the preview message.
    * `streaming.preview.commandText` / `streaming.progress.commandText` controls command/exec detail in compact progress lines: `raw` (default) or `status` (tool label only).

    Hide raw command/exec text while keeping compact progress lines:

    ```json theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      "channels": {
        "discord": {
          "streaming": {
            "mode": "progress",
            "progress": {
              "toolProgress": true,
              "commandText": "status"
            }
          }
        }
      }
    }
    ```

    Preview streaming is text-only; media replies fall back to normal delivery. When `block` streaming is explicitly enabled, OpenClaw skips the preview stream to avoid double-streaming.
  </Accordion>

  <Accordion title="History, context, and thread behavior">
    Guild history context:

    * `channels.discord.historyLimit` default `20`
    * fallback: `messages.groupChat.historyLimit`
    * `0` disables

    DM history controls:

    * `channels.discord.dmHistoryLimit`
    * `channels.discord.dms["<user_id>"].historyLimit`

    Thread behavior:

    * Discord threads route as channel sessions and inherit parent channel config unless overridden.
    * Thread sessions inherit the parent channel's session-level `/model` selection as a model-only fallback; thread-local `/model` selections still take precedence and parent transcript history is not copied unless transcript inheritance is enabled.
    * `channels.discord.thread.inheritParent` (default `false`) opts new auto-threads into seeding from the parent transcript. Per-account overrides live under `channels.discord.accounts.<id>.thread.inheritParent`.
    * Message-tool reactions can resolve `user:<id>` DM targets.
    * `guilds.<guild>.channels.<channel>.requireMention: false` is preserved during reply-stage activation fallback.

    Channel topics are injected as **untrusted** context. Allowlists gate who can trigger the agent, not a full supplemental-context redaction boundary.
  </Accordion>

  <Accordion title="Thread-bound sessions for subagents">
    Discord can bind a thread to a session target so follow-up messages in that thread keep routing to the same session (including subagent sessions).

    Commands:

    * `/focus <target>` bind current/new thread to a subagent/session target
    * `/unfocus` remove current thread binding
    * `/agents` show active runs and binding state
    * `/session idle <duration|off>` inspect/update inactivity auto-unfocus for focused bindings
    * `/session max-age <duration|off>` inspect/update hard max age for focused bindings

    Config:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      session: {
        threadBindings: {
          enabled: true,
          idleHours: 24,
          maxAgeHours: 0,
        },
      },
      channels: {
        discord: {
          threadBindings: {
            enabled: true,
            idleHours: 24,
            maxAgeHours: 0,
            spawnSessions: true,
            defaultSpawnContext: "fork",
          },
        },
      },
    }
    ```

    Notes:

    * `session.threadBindings.*` sets global defaults.
    * `channels.discord.threadBindings.*` overrides Discord behavior.
    * `spawnSessions` controls auto-create/bind threads for `sessions_spawn({ thread: true })` and ACP thread spawns. Default: `true`.
    * `defaultSpawnContext` controls native subagent context for thread-bound spawns. Default: `"fork"`.
    * Deprecated `spawnSubagentSessions`/`spawnAcpSessions` keys are migrated by `openclaw doctor --fix`.
    * If thread bindings are disabled for an account, `/focus` and related thread binding operations are unavailable.

    See [Sub-agents](/tools/subagents), [ACP Agents](/tools/acp-agents), and [Configuration Reference](/gateway/configuration-reference).
  </Accordion>

  <Accordion title="Persistent ACP channel bindings">
    For stable "always-on" ACP workspaces, configure top-level typed ACP bindings targeting Discord conversations.

    Config path:

    * `bindings[]` with `type: "acp"` and `match.channel: "discord"`

    Example:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        list: [
          {
            id: "codex",
            runtime: {
              type: "acp",
              acp: {
                agent: "codex",
                backend: "acpx",
                mode: "persistent",
                cwd: "/workspace/openclaw",
              },
            },
          },
        ],
      },
      bindings: [
        {
          type: "acp",
          agentId: "codex",
          match: {
            channel: "discord",
            accountId: "default",
            peer: { kind: "channel", id: "222222222222222222" },
          },
          acp: { label: "codex-main" },
        },
      ],
      channels: {
        discord: {
          guilds: {
            "111111111111111111": {
              channels: {
                "222222222222222222": {
                  requireMention: false,
                },
              },
            },
          },
        },
      },
    }
    ```

    Notes:

    * `/acp spawn codex --bind here` binds the current channel or thread in place and keeps future messages on the same ACP session. Thread messages inherit the parent channel binding.
    * In a bound channel or thread, `/new` and `/reset` reset the same ACP session in place. Temporary thread bindings can override target resolution while active.
    * `spawnSessions` gates child thread creation/binding via `--thread auto|here`.

    See [ACP Agents](/tools/acp-agents) for binding behavior details.
  </Accordion>

  <Accordion title="Reaction notifications">
    Per-guild reaction notification mode:

    * `off`
    * `own` (default)
    * `all`
    * `allowlist` (uses `guilds.<id>.users`)

    Reaction events are turned into system events and attached to the routed Discord session.
  </Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` sends an acknowledgement emoji while OpenClaw is processing an inbound message.

    Resolution order:

    * `channels.discord.accounts.<accountId>.ackReaction`
    * `channels.discord.ackReaction`
    * `messages.ackReaction`
    * agent identity emoji fallback (`agents.list[].identity.emoji`, else "👀")

    Notes:

    * Discord accepts unicode emoji or custom emoji names.
    * Use `""` to disable the reaction for a channel or account.
  </Accordion>

  <Accordion title="Config writes">
    Channel-initiated config writes are enabled by default.

    This affects `/config set|unset` flows (when command features are enabled).

    Disable:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          configWrites: false,
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Gateway proxy">
    Route Discord gateway WebSocket traffic and startup REST lookups (application ID + allowlist resolution) through an HTTP(S) proxy with `channels.discord.proxy`.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          proxy: "http://proxy.example:8080",
        },
      },
    }
    ```

    Per-account override:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          accounts: {
            primary: {
              proxy: "http://proxy.example:8080",
            },
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="PluralKit support">
    Enable PluralKit resolution to map proxied messages to system member identity:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          pluralkit: {
            enabled: true,
            token: "pk_live_...", // optional; needed for private systems
          },
        },
      },
    }
    ```

    Notes:

    * allowlists can use `pk:<memberId>`
    * member display names are matched by name/slug only when `channels.discord.dangerouslyAllowNameMatching: true`
    * lookups use original message ID and are time-window constrained
    * if lookup fails, proxied messages are treated as bot messages and dropped unless `allowBots=true`
  </Accordion>

  <Accordion title="Outbound mention aliases">
    Use `mentionAliases` when agents need deterministic outbound mentions for known Discord users. Keys are handles without the leading `@`; values are Discord user IDs. Unknown handles, `@everyone`, `@here`, and mentions inside Markdown code spans are left unchanged.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          mentionAliases: {
            Vladislava: "123456789012345678",
          },
          accounts: {
            ops: {
              mentionAliases: {
                OpsLead: "234567890123456789",
              },
            },
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Presence configuration">
    Presence updates are applied when you set a status or activity field, or when you enable auto presence.

    Status only example:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          status: "idle",
        },
      },
    }
    ```

    Activity example (custom status is the default activity type):

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          activity: "Focus time",
          activityType: 4,
        },
      },
    }
    ```

    Streaming example:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          activity: "Live coding",
          activityType: 1,
          activityUrl: "https://twitch.tv/openclaw",
        },
      },
    }
    ```

    Activity type map:

    * 0: Playing
    * 1: Streaming (requires `activityUrl`)
    * 2: Listening
    * 3: Watching
    * 4: Custom (uses the activity text as the status state; emoji is optional)
    * 5: Competing

    Auto presence example (runtime health signal):

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          autoPresence: {
            enabled: true,
            intervalMs: 30000,
            minUpdateIntervalMs: 15000,
            exhaustedText: "token exhausted",
          },
        },
      },
    }
    ```

    Auto presence maps runtime availability to Discord status: healthy => online, degraded or unknown => idle, exhausted or unavailable => dnd. Optional text overrides:

    * `autoPresence.healthyText`
    * `autoPresence.degradedText`
    * `autoPresence.exhaustedText` (supports `{reason}` placeholder)
  </Accordion>

  <Accordion title="Approvals in Discord">
    Discord supports button-based approval handling in DMs and can optionally post approval prompts in the originating channel.

    Config path:

    * `channels.discord.execApprovals.enabled`
    * `channels.discord.execApprovals.approvers` (optional; falls back to `commands.ownerAllowFrom` when possible)
    * `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, default: `dm`)
    * `agentFilter`, `sessionFilter`, `cleanupAfterResolve`

    Discord auto-enables native exec approvals when `enabled` is unset or `"auto"` and at least one approver can be resolved, either from `execApprovals.approvers` or from `commands.ownerAllowFrom`. Discord does not infer exec approvers from channel `allowFrom`, legacy `dm.allowFrom`, or direct-message `defaultTo`. Set `enabled: false` to disable Discord as a native approval client explicitly.

    For sensitive owner-only group commands such as `/diagnostics` and `/export-trajectory`, OpenClaw sends approval prompts and final results privately. It tries Discord DM first when the invoking owner has a Discord owner route; if that is not available, it falls back to the first available owner route from `commands.ownerAllowFrom`, such as Telegram.

    When `target` is `channel` or `both`, the approval prompt is visible in the channel. Only resolved approvers can use the buttons; other users receive an ephemeral denial. Approval prompts include the command text, so only enable channel delivery in trusted channels. If the channel ID cannot be derived from the session key, OpenClaw falls back to DM delivery.

    Discord also renders the shared approval buttons used by other chat channels. The native Discord adapter mainly adds approver DM routing and channel fanout.
    When those buttons are present, they are the primary approval UX; OpenClaw
    should only include a manual `/approve` command when the tool result says
    chat approvals are unavailable or manual approval is the only path.
    If the Discord native approval runtime is not active, OpenClaw keeps the
    local deterministic `/approve <id> <decision>` prompt visible. If the
    runtime is active but a native card cannot be delivered to any target,
    OpenClaw sends a same-chat fallback notice with the exact `/approve`
    command from the pending approval.

    Gateway auth and approval resolution follow the shared Gateway client contract (`plugin:` IDs resolve through `plugin.approval.resolve`; other IDs through `exec.approval.resolve`). Approvals expire after 30 minutes by default.

    See [Exec approvals](/tools/exec-approvals).
  </Accordion>
</AccordionGroup>

## Tools and action gates

Discord message actions include messaging, channel admin, moderation, presence, and metadata actions.

Core examples:

* messaging: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
* reactions: `react`, `reactions`, `emojiList`
* moderation: `timeout`, `kick`, `ban`
* presence: `setPresence`

The `event-create` action accepts an optional `image` parameter (URL or local file path) to set the scheduled event cover image.

Action gates live under `channels.discord.actions.*`.

Default gate behavior:

| Action group                                                                                                                                                             | Default  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | enabled  |
| roles                                                                                                                                                                    | disabled |
| moderation                                                                                                                                                               | disabled |
| presence                                                                                                                                                                 | disabled |

## Components v2 UI

OpenClaw uses Discord components v2 for exec approvals and cross-context markers. Discord message actions can also accept `components` for custom UI (advanced; requires constructing a component payload via the discord tool), while legacy `embeds` remain available but are not recommended.

* `channels.discord.ui.components.accentColor` sets the accent color used by Discord component containers (hex).
* Set per account with `channels.discord.accounts.<id>.ui.components.accentColor`.
* `embeds` are ignored when components v2 are present.

Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```

## Voice

Discord has two distinct voice surfaces: realtime **voice channels** (continuous conversations) and **voice message attachments** (the waveform preview format). The gateway supports both.

### Voice channels

Setup checklist:

1. Enable Message Content Intent in the Discord Developer Portal.
2. Enable Server Members Intent when role/user allowlists are used.
3. Invite the bot with `bot` and `applications.commands` scopes.
4. Grant Connect, Speak, Send Messages, and Read Message History in the target voice channel.
5. Enable native commands (`commands.native` or `channels.discord.commands.native`).
6. Configure `channels.discord.voice`.

Use `/vc join|leave|status` to control sessions. The command uses the account default agent and follows the same allowlist and group policy rules as other Discord commands.

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
/vc join channel:<voice-channel-id>
/vc status
/vc leave
```

Auto-join example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    discord: {
      voice: {
        enabled: true,
        model: "openai/gpt-5.4-mini",
        autoJoin: [
          {
            guildId: "123456789012345678",
            channelId: "234567890123456789",
          },
        ],
        daveEncryption: true,
        decryptionFailureTolerance: 24,
        connectTimeoutMs: 30000,
        reconnectGraceMs: 15000,
        tts: {
          provider: "openai",
          openai: { voice: "onyx" },
        },
      },
    },
  },
}
```

Notes:

* `voice.tts` overrides `messages.tts` for voice playback only.
* `voice.model` overrides the LLM used for Discord voice channel responses only. Leave it unset to inherit the routed agent model.
* STT uses `tools.media.audio`; `voice.model` does not affect transcription.
* Per-channel Discord `systemPrompt` overrides apply to voice transcript turns for that voice channel.
* Voice transcript turns derive owner status from Discord `allowFrom` (or `dm.allowFrom`); non-owner speakers cannot access owner-only tools (for example `gateway` and `cron`).
* Discord voice is opt-in for text-only configs; set `channels.discord.voice.enabled=true` (or keep an existing `channels.discord.voice` block) to enable `/vc` commands, the voice runtime, and the `GuildVoiceStates` gateway intent.
* `channels.discord.intents.voiceStates` can explicitly override voice-state intent subscription. Leave it unset for the intent to follow effective voice enablement.
* `voice.daveEncryption` and `voice.decryptionFailureTolerance` pass through to `@discordjs/voice` join options.
* `@discordjs/voice` defaults are `daveEncryption=true` and `decryptionFailureTolerance=24` if unset.
* `voice.connectTimeoutMs` controls the initial `@discordjs/voice` Ready wait for `/vc join` and auto-join attempts. Default: `30000`.
* `voice.reconnectGraceMs` controls how long OpenClaw waits for a disconnected voice session to begin reconnecting before destroying it. Default: `15000`.
* OpenClaw also watches receive decrypt failures and auto-recovers by leaving/rejoining the voice channel after repeated failures in a short window.
* If receive logs repeatedly show `DecryptionFailed(UnencryptedWhenPassthroughDisabled)` after updating, collect a dependency report and logs. The bundled `@discordjs/voice` line includes the upstream padding fix from discord.js PR #11449, which closed discord.js issue #11419.

Voice channel pipeline:

* Discord PCM capture is converted to a WAV temp file.
* `tools.media.audio` handles STT, for example `openai/gpt-4o-mini-transcribe`.
* The transcript is sent through Discord ingress and routing while the response LLM runs with a voice-output policy that hides the agent `tts` tool and asks for returned text, because Discord voice owns final TTS playback.
* `voice.model`, when set, overrides only the response LLM for this voice-channel turn.
* `voice.tts` is merged over `messages.tts`; the resulting audio is played in the joined channel.

Credentials are resolved per component: LLM route auth for `voice.model`, STT auth for `tools.media.audio`, and TTS auth for `messages.tts`/`voice.tts`.

### Voice messages

Discord voice messages show a waveform preview and require OGG/Opus audio. OpenClaw generates the waveform automatically, but needs `ffmpeg` and `ffprobe` on the gateway host to inspect and convert.

* Provide a **local file path** (URLs are rejected).
* Omit text content (Discord rejects text + voice message in the same payload).
* Any audio format is accepted; OpenClaw converts to OGG/Opus as needed.

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Troubleshooting

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">
    * enable Message Content Intent
    * enable Server Members Intent when you depend on user/member resolution
    * restart gateway after changing intents
  </Accordion>

  <Accordion title="Guild messages blocked unexpectedly">
    * verify `groupPolicy`
    * verify guild allowlist under `channels.discord.guilds`
    * if guild `channels` map exists, only listed channels are allowed
    * verify `requireMention` behavior and mention patterns

    Useful checks:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw doctor
    openclaw channels status --probe
    openclaw logs --follow
    ```
  </Accordion>

  <Accordion title="Require mention false but still blocked">
    Common causes:

    * `groupPolicy="allowlist"` without matching guild/channel allowlist
    * `requireMention` configured in the wrong place (must be under `channels.discord.guilds` or channel entry)
    * sender blocked by guild/channel `users` allowlist
  </Accordion>

  <Accordion title="Long-running Discord turns or duplicate replies">
    Typical logs:

    * `Slow listener detected ...`
    * `stuck session: sessionKey=agent:...:discord:... state=processing ...`

    Discord gateway queue knobs:

    * single-account: `channels.discord.eventQueue.listenerTimeout`
    * multi-account: `channels.discord.accounts.<accountId>.eventQueue.listenerTimeout`
    * this only controls Discord gateway listener work, not agent turn lifetime

    Discord does not apply a channel-owned timeout to queued agent turns. Message listeners hand off immediately, and queued Discord runs preserve per-session ordering until the session/tool/runtime lifecycle completes or aborts the work.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          accounts: {
            default: {
              eventQueue: {
                listenerTimeout: 120000,
              },
            },
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Gateway metadata lookup timeout warnings">
    OpenClaw fetches Discord `/gateway/bot` metadata before connecting. Transient failures fall back to Discord's default gateway URL and are rate-limited in logs.

    Metadata timeout knobs:

    * single-account: `channels.discord.gatewayInfoTimeoutMs`
    * multi-account: `channels.discord.accounts.<accountId>.gatewayInfoTimeoutMs`
    * env fallback when config is unset: `OPENCLAW_DISCORD_GATEWAY_INFO_TIMEOUT_MS`
    * default: `30000` (30 seconds), max: `120000`
  </Accordion>

  <Accordion title="Gateway READY timeout restarts">
    OpenClaw waits for Discord's gateway `READY` event during startup and after runtime reconnects. Multi-account setups with startup staggering can need a longer startup READY window than the default.

    READY timeout knobs:

    * startup single-account: `channels.discord.gatewayReadyTimeoutMs`
    * startup multi-account: `channels.discord.accounts.<accountId>.gatewayReadyTimeoutMs`
    * startup env fallback when config is unset: `OPENCLAW_DISCORD_READY_TIMEOUT_MS`
    * startup default: `15000` (15 seconds), max: `120000`
    * runtime single-account: `channels.discord.gatewayRuntimeReadyTimeoutMs`
    * runtime multi-account: `channels.discord.accounts.<accountId>.gatewayRuntimeReadyTimeoutMs`
    * runtime env fallback when config is unset: `OPENCLAW_DISCORD_RUNTIME_READY_TIMEOUT_MS`
    * runtime default: `30000` (30 seconds), max: `120000`
  </Accordion>

  <Accordion title="Permissions audit mismatches">
    `channels status --probe` permission checks only work for numeric channel IDs.

    If you use slug keys, runtime matching can still work, but probe cannot fully verify permissions.
  </Accordion>

  <Accordion title="DM and pairing issues">
    * DM disabled: `channels.discord.dm.enabled=false`
    * DM policy disabled: `channels.discord.dmPolicy="disabled"` (legacy: `channels.discord.dm.policy`)
    * awaiting pairing approval in `pairing` mode
  </Accordion>

  <Accordion title="Bot to bot loops">
    By default bot-authored messages are ignored.

    If you set `channels.discord.allowBots=true`, use strict mention and allowlist rules to avoid loop behavior.
    Prefer `channels.discord.allowBots="mentions"` to only accept bot messages that mention the bot.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        discord: {
          accounts: {
            mantis: {
              // Mantis listens to other bots only when they mention her.
              allowBots: "mentions",
            },
            molty: {
              // Molty listens to all bot-authored Discord messages.
              allowBots: true,
              mentionAliases: {
                // Lets Molty write "@Mantis" and send a real Discord mention.
                Mantis: "MANTIS_DISCORD_USER_ID",
              },
            },
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Voice STT drops with DecryptionFailed(...)">
    * keep OpenClaw current (`openclaw update`) so the Discord voice receive recovery logic is present
    * confirm `channels.discord.voice.daveEncryption=true` (default)
    * start from `channels.discord.voice.decryptionFailureTolerance=24` (upstream default) and tune only if needed
    * watch logs for:
      * `discord voice: DAVE decrypt failures detected`
      * `discord voice: repeated decrypt failures; attempting rejoin`
    * if failures continue after automatic rejoin, collect logs and compare against the upstream DAVE receive history in [discord.js #11419](https://github.com/discordjs/discord.js/issues/11419) and [discord.js #11449](https://github.com/discordjs/discord.js/pull/11449)
  </Accordion>
</AccordionGroup>

## Configuration reference

Primary reference: [Configuration reference - Discord](/gateway/config-channels#discord).

<Accordion title="High-signal Discord fields">
  * startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
  * policy: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
  * command: `commands.native`, `commands.useAccessGroups`, `configWrites`, `slashCommand.*`
  * event queue: `eventQueue.listenerTimeout` (listener budget), `eventQueue.maxQueueSize`, `eventQueue.maxConcurrency`
  * gateway: `gatewayInfoTimeoutMs`, `gatewayReadyTimeoutMs`, `gatewayRuntimeReadyTimeoutMs`
  * reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  * delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
  * streaming: `streaming` (legacy alias: `streamMode`), `streaming.preview.toolProgress`, `draftChunk`, `blockStreaming`, `blockStreamingCoalesce`
  * media/retry: `mediaMaxMb` (caps outbound Discord uploads, default `100MB`), `retry`
  * actions: `actions.*`
  * presence: `activity`, `status`, `activityType`, `activityUrl`
  * UI: `ui.components.accentColor`
  * features: `threadBindings`, top-level `bindings[]` (`type: "acp"`), `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`
</Accordion>

## Safety and operations

* Treat bot tokens as secrets (`DISCORD_BOT_TOKEN` preferred in supervised environments).
* Grant least-privilege Discord permissions.
* If command deploy/state is stale, restart gateway and re-check with `openclaw channels status --probe`.

## Related

<CardGroup cols={2}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Pair a Discord user to the gateway.
  </Card>

  <Card title="Groups" icon="users" href="/channels/groups">
    Group chat and allowlist behavior.
  </Card>

  <Card title="Channel routing" icon="route" href="/channels/channel-routing">
    Route inbound messages to agents.
  </Card>

  <Card title="Security" icon="shield" href="/gateway/security">
    Threat model and hardening.
  </Card>

  <Card title="Multi-agent routing" icon="sitemap" href="/concepts/multi-agent">
    Map guilds and channels to agents.
  </Card>

  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Native command behavior.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Feishu

Feishu/Lark is an all-in-one collaboration platform where teams chat, share documents, manage calendars, and get work done together.

**Status:** production-ready for bot DMs + group chats. WebSocket is the default mode; webhook mode is optional.

***

## Quick start

<Note>
  Requires OpenClaw 2026.4.25 or above. Run `openclaw --version` to check. Upgrade with `openclaw update`.
</Note>

<Steps>
  <Step title="Run the channel setup wizard">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw channels login --channel feishu
    ```

    Scan the QR code with your Feishu/Lark mobile app to create a Feishu/Lark bot automatically.
  </Step>

  <Step title="After setup completes, restart the gateway to apply the changes">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw gateway restart
    ```
  </Step>
</Steps>

***

## Access control

### Direct messages

Configure `dmPolicy` to control who can DM the bot:

* `"pairing"` - unknown users receive a pairing code; approve via CLI
* `"allowlist"` - only users listed in `allowFrom` can chat (default: bot owner only)
* `"open"` - allow public DMs only when `allowFrom` includes `"*"`; with restrictive entries, only matching users can chat
* `"disabled"` - disable all DMs

**Approve a pairing request:**

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw pairing list feishu
openclaw pairing approve feishu <CODE>
```

### Group chats

**Group policy** (`channels.feishu.groupPolicy`):

| Value         | Behavior                                                                                     |
| ------------- | -------------------------------------------------------------------------------------------- |
| `"open"`      | Respond to all messages in groups                                                            |
| `"allowlist"` | Only respond to groups in `groupAllowFrom` or explicitly configured under `groups.<chat_id>` |
| `"disabled"`  | Disable all group messages; explicit `groups.<chat_id>` entries do not override this         |

Default: `allowlist`

**Mention requirement** (`channels.feishu.requireMention`):

* `true` - require @mention (default)
* `false` - respond without @mention
* Per-group override: `channels.feishu.groups.<chat_id>.requireMention`
* Broadcast-only `@all` and `@_all` are not treated as bot mentions. A message that mentions both `@all` and the bot directly still counts as a bot mention.

***

## Group configuration examples

### Allow all groups, no @mention required

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    feishu: {
      groupPolicy: "open",
    },
  },
}
```

### Allow all groups, still require @mention

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    feishu: {
      groupPolicy: "open",
      requireMention: true,
    },
  },
}
```

### Allow specific groups only

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      // Group IDs look like: oc_xxx
      groupAllowFrom: ["oc_xxx", "oc_yyy"],
    },
  },
}
```

In `allowlist` mode, you can also admit a group by adding an explicit `groups.<chat_id>` entry. Explicit entries do not override `groupPolicy: "disabled"`. Wildcard defaults under `groups.*` configure matching groups, but they do not admit groups by themselves.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groups: {
        oc_xxx: {
          requireMention: false,
        },
      },
    },
  },
}
```

### Restrict senders within a group

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxx"],
      groups: {
        oc_xxx: {
          // User open_ids look like: ou_xxx
          allowFrom: ["ou_user1", "ou_user2"],
        },
      },
    },
  },
}
```

***

<a id="get-groupuser-ids" />

## Get group/user IDs

### Group IDs (`chat_id`, format: `oc_xxx`)

Open the group in Feishu/Lark, click the menu icon in the top-right corner, and go to **Settings**. The group ID (`chat_id`) is listed on the settings page.

<img src="https://mintcdn.com/clawdhub/0NpU6wNaI7exeaOE/images/feishu-get-group-id.png?fit=max&auto=format&n=0NpU6wNaI7exeaOE&q=85&s=1c9b41e1f9743621dfdd3abf7e952405" alt="Get Group ID" width="1636" height="1764" data-path="images/feishu-get-group-id.png" />

### User IDs (`open_id`, format: `ou_xxx`)

Start the gateway, send a DM to the bot, then check the logs:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw logs --follow
```

Look for `open_id` in the log output. You can also check pending pairing requests:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw pairing list feishu
```

***

## Common commands

| Command   | Description                 |
| --------- | --------------------------- |
| `/status` | Show bot status             |
| `/reset`  | Reset the current session   |
| `/model`  | Show or switch the AI model |

<Note>
  Feishu/Lark does not support native slash-command menus, so send these as plain text messages.
</Note>

***

## Troubleshooting

### Bot does not respond in group chats

1. Ensure the bot is added to the group
2. Ensure you @mention the bot (required by default)
3. Verify `groupPolicy` is not `"disabled"`
4. Check logs: `openclaw logs --follow`

### Bot does not receive messages

1. Ensure the bot is published and approved in Feishu Open Platform / Lark Developer
2. Ensure event subscription includes `im.message.receive_v1`
3. Ensure **persistent connection** (WebSocket) is selected
4. Ensure all required permission scopes are granted
5. Ensure the gateway is running: `openclaw gateway status`
6. Check logs: `openclaw logs --follow`

### App Secret leaked

1. Reset the App Secret in Feishu Open Platform / Lark Developer
2. Update the value in your config
3. Restart the gateway: `openclaw gateway restart`

***

## Advanced configuration

### Multiple accounts

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          name: "Primary bot",
          tts: {
            providers: {
              openai: { voice: "shimmer" },
            },
          },
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          name: "Backup bot",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount` controls which account is used when outbound APIs do not specify an `accountId`.
`accounts.<id>.tts` uses the same shape as `messages.tts` and deep-merges over
global TTS config, so multi-bot Feishu setups can keep shared provider
credentials globally while overriding only voice, model, persona, or auto mode
per account.

### Message limits

* `textChunkLimit` - outbound text chunk size (default: `2000` chars)
* `mediaMaxMb` - media upload/download limit (default: `30` MB)

### Streaming

Feishu/Lark supports streaming replies via interactive cards. When enabled, the bot updates the card in real time as it generates text.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    feishu: {
      streaming: true, // enable streaming card output (default: true)
      blockStreaming: true, // opt into completed-block streaming
    },
  },
}
```

Set `streaming: false` to send the complete reply in one message. `blockStreaming` is off by default; enable it only when you want completed assistant blocks flushed before the final reply.

### Quota optimization

Reduce the number of Feishu/Lark API calls with two optional flags:

* `typingIndicator` (default `true`): set `false` to skip typing reaction calls
* `resolveSenderNames` (default `true`): set `false` to skip sender profile lookups

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    feishu: {
      typingIndicator: false,
      resolveSenderNames: false,
    },
  },
}
```

### ACP sessions

Feishu/Lark supports ACP for DMs and group thread messages. Feishu/Lark ACP is text-command driven - there are no native slash-command menus, so use `/acp ...` messages directly in the conversation.

#### Persistent ACP binding

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "feishu",
        accountId: "default",
        peer: { kind: "direct", id: "ou_1234567890" },
      },
    },
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "feishu",
        accountId: "default",
        peer: { kind: "group", id: "oc_group_chat:topic:om_topic_root" },
      },
      acp: { label: "codex-feishu-topic" },
    },
  ],
}
```

#### Spawn ACP from chat

In a Feishu/Lark DM or thread:

```text theme={"theme":{"light":"min-light","dark":"min-dark"}}
/acp spawn codex --thread here
```

`--thread here` works for DMs and Feishu/Lark thread messages. Follow-up messages in the bound conversation route directly to that ACP session.

### Multi-agent routing

Use `bindings` to route Feishu/Lark DMs or groups to different agents.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    list: [
      { id: "main" },
      { id: "agent-a", workspace: "/home/user/agent-a" },
      { id: "agent-b", workspace: "/home/user/agent-b" },
    ],
  },
  bindings: [
    {
      agentId: "agent-a",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "agent-b",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

Routing fields:

* `match.channel`: `"feishu"`
* `match.peer.kind`: `"direct"` (DM) or `"group"` (group chat)
* `match.peer.id`: user Open ID (`ou_xxx`) or group ID (`oc_xxx`)

See [Get group/user IDs](#get-groupuser-ids) for lookup tips.

***

## Configuration reference

Full configuration: [Gateway configuration](/gateway/configuration)

| Setting                                           | Description                                                                      | Default          |
| ------------------------------------------------- | -------------------------------------------------------------------------------- | ---------------- |
| `channels.feishu.enabled`                         | Enable/disable the channel                                                       | `true`           |
| `channels.feishu.domain`                          | API domain (`feishu` or `lark`)                                                  | `feishu`         |
| `channels.feishu.connectionMode`                  | Event transport (`websocket` or `webhook`)                                       | `websocket`      |
| `channels.feishu.defaultAccount`                  | Default account for outbound routing                                             | `default`        |
| `channels.feishu.verificationToken`               | Required for webhook mode                                                        | -                |
| `channels.feishu.encryptKey`                      | Required for webhook mode                                                        | -                |
| `channels.feishu.webhookPath`                     | Webhook route path                                                               | `/feishu/events` |
| `channels.feishu.webhookHost`                     | Webhook bind host                                                                | `127.0.0.1`      |
| `channels.feishu.webhookPort`                     | Webhook bind port                                                                | `3000`           |
| `channels.feishu.accounts.<id>.appId`             | App ID                                                                           | -                |
| `channels.feishu.accounts.<id>.appSecret`         | App Secret                                                                       | -                |
| `channels.feishu.accounts.<id>.domain`            | Per-account domain override                                                      | `feishu`         |
| `channels.feishu.accounts.<id>.tts`               | Per-account TTS override                                                         | `messages.tts`   |
| `channels.feishu.dmPolicy`                        | DM policy                                                                        | `allowlist`      |
| `channels.feishu.allowFrom`                       | DM allowlist (open\_id list)                                                     | \[BotOwnerId]    |
| `channels.feishu.groupPolicy`                     | Group policy                                                                     | `allowlist`      |
| `channels.feishu.groupAllowFrom`                  | Group allowlist                                                                  | -                |
| `channels.feishu.requireMention`                  | Require @mention in groups                                                       | `true`           |
| `channels.feishu.groups.<chat_id>.requireMention` | Per-group @mention override; explicit IDs also admit the group in allowlist mode | inherited        |
| `channels.feishu.groups.<chat_id>.enabled`        | Enable/disable a specific group                                                  | `true`           |
| `channels.feishu.textChunkLimit`                  | Message chunk size                                                               | `2000`           |
| `channels.feishu.mediaMaxMb`                      | Media size limit                                                                 | `30`             |
| `channels.feishu.streaming`                       | Streaming card output                                                            | `true`           |
| `channels.feishu.blockStreaming`                  | Completed-block reply streaming                                                  | `false`          |
| `channels.feishu.typingIndicator`                 | Send typing reactions                                                            | `true`           |
| `channels.feishu.resolveSenderNames`              | Resolve sender display names                                                     | `true`           |

***

## Supported message types

### Receive

* ✅ Text
* ✅ Rich text (post)
* ✅ Images
* ✅ Files
* ✅ Audio
* ✅ Video/media
* ✅ Stickers

Inbound Feishu/Lark audio messages are normalized as media placeholders instead
of raw `file_key` JSON. When `tools.media.audio` is configured, OpenClaw
downloads the voice-note resource and runs shared audio transcription before the
agent turn, so the agent receives the spoken transcript. If Feishu includes
transcript text directly in the audio payload, that text is used without another
ASR call. Without an audio transcription provider, the agent still receives a
`<media:audio>` placeholder plus the saved attachment, not the raw Feishu
resource payload.

### Send

* ✅ Text
* ✅ Images
* ✅ Files
* ✅ Audio
* ✅ Video/media
* ✅ Interactive cards (including streaming updates)
* ⚠️ Rich text (post-style formatting; doesn't support full Feishu/Lark authoring capabilities)

Native Feishu/Lark audio bubbles use the Feishu `audio` message type and require
Ogg/Opus upload media (`file_type: "opus"`). Existing `.opus` and `.ogg` media
is sent directly as native audio. MP3/WAV/M4A and other likely audio formats are
transcoded to 48kHz Ogg/Opus with `ffmpeg` only when the reply requests voice
delivery (`audioAsVoice` / message tool `asVoice`, including TTS voice-note
replies). Ordinary MP3 attachments stay regular files. If `ffmpeg` is missing or
conversion fails, OpenClaw falls back to a file attachment and logs the reason.

### Threads and replies

* ✅ Inline replies
* ✅ Thread replies
* ✅ Media replies stay thread-aware when replying to a thread message

For `groupSessionScope: "group_topic"` and `"group_topic_sender"`, native
Feishu/Lark topic groups use the event `thread_id` (`omt_*`) as the canonical
topic session key. If a native topic starter event omits `thread_id`, OpenClaw
hydrates it from Feishu before routing the turn. Normal group replies that
OpenClaw turns into threads keep using the reply root message ID (`om_*`) so the
first turn and follow-up turn stay in the same session.

***

## Related

* [Channels Overview](/channels) - all supported channels
* [Pairing](/channels/pairing) - DM authentication and pairing flow
* [Groups](/channels/groups) - group chat behavior and mention gating
* [Channel Routing](/channels/channel-routing) - session routing for messages
* [Security](/gateway/security) - access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Groups

OpenClaw treats group chats consistently across surfaces: Discord, iMessage, Matrix, Microsoft Teams, Signal, Slack, Telegram, WhatsApp, Zalo.

## Beginner intro (2 minutes)

OpenClaw "lives" on your own messaging accounts. There is no separate WhatsApp bot user. If **you** are in a group, OpenClaw can see that group and respond there.

Default behavior:

* Groups are restricted (`groupPolicy: "allowlist"`).
* Replies require a mention unless you explicitly disable mention gating.
* Normal final replies in groups/channels are private by default. Visible room output uses the `message` tool.

Translation: allowlisted senders can trigger OpenClaw by mentioning it.

<Note>
  **TL;DR**

  * **DM access** is controlled by `*.allowFrom`.
  * **Group access** is controlled by `*.groupPolicy` + allowlists (`*.groups`, `*.groupAllowFrom`).
  * **Reply triggering** is controlled by mention gating (`requireMention`, `/activation`).
</Note>

Quick flow (what happens to a group message):

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

## Visible replies

For group/channel rooms, OpenClaw defaults to `messages.groupChat.visibleReplies: "message_tool"`.
`openclaw doctor --fix` writes this default into configured-channel configs that omit it.
That means the agent still processes the turn and can update memory/session state, but its normal final answer is not automatically posted back into the room. To speak visibly, the agent uses `message(action=send)`.

This default depends on a model/runtime that reliably calls tools. If logs show
assistant text but `didSendViaMessagingTool: false`, the model answered
privately instead of calling the message tool. That is not a
Discord/Slack/Telegram send failure. Use a tool-call-reliable model for
group/channel sessions, or set
`messages.groupChat.visibleReplies: "automatic"` to restore legacy visible
final replies.

If the message tool is unavailable under the active tool policy, OpenClaw falls
back to automatic visible replies instead of silently suppressing the response.
`openclaw doctor` warns about this mismatch.

For direct chats and any other source turn, use `messages.visibleReplies: "message_tool"` to apply the same tool-only visible-reply behavior globally. Harnesses can also choose this as their unset default; the Codex harness does this for Codex-mode direct chats. `messages.groupChat.visibleReplies` remains the more specific override for group/channel rooms.

This replaces the old pattern of forcing the model to answer `NO_REPLY` for most lurk-mode turns. In tool-only mode, doing nothing visible simply means not calling the message tool.

Typing indicators are still sent while the agent works in tool-only mode. The default group typing mode is upgraded from "message" to "instant" for these turns because there may never be normal assistant message text before the agent decides whether to call the message tool. Explicit typing-mode config still wins.

To restore legacy automatic final replies for group/channel rooms:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  messages: {
    groupChat: {
      visibleReplies: "automatic",
    },
  },
}
```

The gateway hot-reloads `messages` config after the file is saved. Restart only
when file watching or config reload is disabled in the deployment.

To require visible output to go through the message tool for every source chat:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  messages: {
    visibleReplies: "message_tool",
  },
}
```

Native slash commands (Discord, Telegram, and other surfaces with native command support) bypass `visibleReplies: "message_tool"` and always reply visibly so the channel-native command UI gets the response it expects. This applies to validated native command turns only; text-typed `/...` commands and ordinary chat turns still follow the configured group default.

## Context visibility and allowlists

Two different controls are involved in group safety:

* **Trigger authorization**: who can trigger the agent (`groupPolicy`, `groups`, `groupAllowFrom`, channel-specific allowlists).
* **Context visibility**: what supplemental context is injected into the model (reply text, quotes, thread history, forwarded metadata).

By default, OpenClaw prioritizes normal chat behavior and keeps context mostly as received. This means allowlists primarily decide who can trigger actions, not a universal redaction boundary for every quoted or historical snippet.

<AccordionGroup>
  <Accordion title="Current behavior is channel-specific">
    * Some channels already apply sender-based filtering for supplemental context in specific paths (for example Slack thread seeding, Matrix reply/thread lookups).
    * Other channels still pass quote/reply/forward context through as received.
  </Accordion>

  <Accordion title="Hardening direction (planned)">
    * `contextVisibility: "all"` (default) keeps current as-received behavior.
    * `contextVisibility: "allowlist"` filters supplemental context to allowlisted senders.
    * `contextVisibility: "allowlist_quote"` is `allowlist` plus one explicit quote/reply exception.

    Until this hardening model is implemented consistently across channels, expect differences by surface.
  </Accordion>
</AccordionGroup>

<img src="https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/images/groups-flow.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=eeb387df91a967fbbe8bf8f80ae41dd7" alt="Group message flow" width="960" height="260" data-path="images/groups-flow.svg" />

If you want...

| Goal                                         | What to set                                                |
| -------------------------------------------- | ---------------------------------------------------------- |
| Allow all groups but only reply on @mentions | `groups: { "*": { requireMention: true } }`                |
| Disable all group replies                    | `groupPolicy: "disabled"`                                  |
| Only specific groups                         | `groups: { "<group-id>": { ... } }` (no `"*"` key)         |
| Only you can trigger in groups               | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |
| Reuse one trusted sender set across channels | `groupAllowFrom: ["accessGroup:operators"]`                |

For reusable sender allowlists, see [Access groups](/channels/access-groups).

## Session keys

* Group sessions use `agent:<agentId>:<channel>:group:<id>` session keys (rooms/channels use `agent:<agentId>:<channel>:channel:<id>`).
* Telegram forum topics add `:topic:<threadId>` to the group id so each topic has its own session.
* Direct chats use the main session (or per-sender if configured).
* Heartbeats are skipped for group sessions.

<a id="pattern-personal-dms-public-groups-single-agent" />

## Pattern: personal DMs + public groups (single agent)

Yes — this works well if your "personal" traffic is **DMs** and your "public" traffic is **groups**.

Why: in single-agent mode, DMs typically land in the **main** session key (`agent:main:main`), while groups always use **non-main** session keys (`agent:main:<channel>:group:<id>`). If you enable sandboxing with `mode: "non-main"`, those group sessions run in the configured sandbox backend while your main DM session stays on-host. Docker is the default backend if you do not choose one.

This gives you one agent "brain" (shared workspace + memory), but two execution postures:

* **DMs**: full tools (host)
* **Groups**: sandbox + restricted tools

<Note>
  If you need truly separate workspaces/personas ("personal" and "public" must never mix), use a second agent + bindings. See [Multi-Agent Routing](/concepts/multi-agent).
</Note>

<Tabs>
  <Tab title="DMs on host, groups sandboxed">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main", // groups/channels are non-main -> sandboxed
            scope: "session", // strongest isolation (one container per group/channel)
            workspaceAccess: "none",
          },
        },
      },
      tools: {
        sandbox: {
          tools: {
            // If allow is non-empty, everything else is blocked (deny still wins).
            allow: ["group:messaging", "group:sessions"],
            deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
          },
        },
      },
    }
    ```
  </Tab>

  <Tab title="Groups see only an allowlisted folder">
    Want "groups can only see folder X" instead of "no host access"? Keep `workspaceAccess: "none"` and mount only allowlisted paths into the sandbox:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",
            scope: "session",
            workspaceAccess: "none",
            docker: {
              binds: [
                // hostPath:containerPath:mode
                "/home/user/FriendsShared:/data:ro",
              ],
            },
          },
        },
      },
    }
    ```
  </Tab>
</Tabs>

Related:

* Configuration keys and defaults: [Gateway configuration](/gateway/config-agents#agentsdefaultssandbox)
* Debugging why a tool is blocked: [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated)
* Bind mounts details: [Sandboxing](/gateway/sandboxing#custom-bind-mounts)

## Display labels

* UI labels use `displayName` when available, formatted as `<channel>:<token>`.
* `#room` is reserved for rooms/channels; group chats use `g-<slug>` (lowercase, spaces -> `-`, keep `#@+._-`).

## Group policy

Control how group/room messages are handled per channel:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // numeric Telegram user id (wizard can resolve @username)
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: { channels: { help: { allow: true } } },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { enabled: true },
        "#alias:example.org": { enabled: true },
      },
    },
  },
}
```

| Policy        | Behavior                                                     |
| ------------- | ------------------------------------------------------------ |
| `"open"`      | Groups bypass allowlists; mention-gating still applies.      |
| `"disabled"`  | Block all group messages entirely.                           |
| `"allowlist"` | Only allow groups/rooms that match the configured allowlist. |

<AccordionGroup>
  <Accordion title="Per-channel notes">
    * `groupPolicy` is separate from mention-gating (which requires @mentions).
    * WhatsApp/Telegram/Signal/iMessage/Microsoft Teams/Zalo: use `groupAllowFrom` (fallback: explicit `allowFrom`).
    * Signal: `groupAllowFrom` can match either the inbound Signal group id or the sender phone/UUID.
    * DM pairing approvals (`*-allowFrom` store entries) apply to DM access only; group sender authorization stays explicit to group allowlists.
    * Discord: allowlist uses `channels.discord.guilds.<id>.channels`.
    * Slack: allowlist uses `channels.slack.channels`.
    * Matrix: allowlist uses `channels.matrix.groups`. Prefer room IDs or aliases; joined-room name lookup is best-effort, and unresolved names are ignored at runtime. Use `channels.matrix.groupAllowFrom` to restrict senders; per-room `users` allowlists are also supported.
    * Group DMs are controlled separately (`channels.discord.dm.*`, `channels.slack.dm.*`).
    * Telegram allowlist can match user IDs (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) or usernames (`"@alice"` or `"alice"`); prefixes are case-insensitive.
    * Default is `groupPolicy: "allowlist"`; if your group allowlist is empty, group messages are blocked.
    * Runtime safety: when a provider block is completely missing (`channels.<provider>` absent), group policy falls back to a fail-closed mode (typically `allowlist`) instead of inheriting `channels.defaults.groupPolicy`.
  </Accordion>
</AccordionGroup>

Quick mental model (evaluation order for group messages):

<Steps>
  <Step title="groupPolicy">
    `groupPolicy` (open/disabled/allowlist).
  </Step>

  <Step title="Group allowlists">
    Group allowlists (`*.groups`, `*.groupAllowFrom`, channel-specific allowlist).
  </Step>

  <Step title="Mention gating">
    Mention gating (`requireMention`, `/activation`).
  </Step>
</Steps>

## Mention gating (default)

Group messages require a mention unless overridden per group. Defaults live per subsystem under `*.groups."*"`.

Replying to a bot message counts as an implicit mention when the channel supports reply metadata. Quoting a bot message can also count as an implicit mention on channels that expose quote metadata. Current built-in cases include Telegram, WhatsApp, Slack, Discord, Microsoft Teams, and ZaloUser.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false },
      },
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false },
      },
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50,
        },
      },
    ],
  },
}
```

<AccordionGroup>
  <Accordion title="Mention gating notes">
    * `mentionPatterns` are case-insensitive safe regex patterns; invalid patterns and unsafe nested-repetition forms are ignored.
    * Surfaces that provide explicit mentions still pass; patterns are a fallback.
    * Per-agent override: `agents.list[].groupChat.mentionPatterns` (useful when multiple agents share a group).
    * Mention gating is only enforced when mention detection is possible (native mentions or `mentionPatterns` are configured).
    * Allowlisting a group or sender does not disable mention gating; set that group's `requireMention` to `false` when all messages should trigger.
    * Group chat prompt context carries the resolved silent-reply instruction every turn; workspace files should not duplicate `NO_REPLY` mechanics.
    * Groups where silent replies are allowed treat clean empty or reasoning-only model turns as silent, equivalent to `NO_REPLY`. Direct chats do the same only when direct silent replies are explicitly allowed; otherwise empty replies remain failed agent turns.
    * Discord defaults live in `channels.discord.guilds."*"` (overridable per guild/channel).
    * Group history context is wrapped uniformly across channels and is **pending-only** (messages skipped due to mention gating); use `messages.groupChat.historyLimit` for the global default and `channels.<channel>.historyLimit` (or `channels.<channel>.accounts.*.historyLimit`) for overrides. Set `0` to disable.
  </Accordion>
</AccordionGroup>

## Group/channel tool restrictions (optional)

Some channel configs support restricting which tools are available **inside a specific group/room/channel**.

* `tools`: allow/deny tools for the whole group.
* `toolsBySender`: per-sender overrides within the group. Use explicit key prefixes: `id:<senderId>`, `e164:<phone>`, `username:<handle>`, `name:<displayName>`, and `"*"` wildcard. Legacy unprefixed keys are still accepted and matched as `id:` only.

Resolution order (most specific wins):

<Steps>
  <Step title="Group toolsBySender">
    Group/channel `toolsBySender` match.
  </Step>

  <Step title="Group tools">
    Group/channel `tools`.
  </Step>

  <Step title="Default toolsBySender">
    Default (`"*"`) `toolsBySender` match.
  </Step>

  <Step title="Default tools">
    Default (`"*"`) `tools`.
  </Step>
</Steps>

Example (Telegram):

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "id:123456789": { alsoAllow: ["exec"] },
          },
        },
      },
    },
  },
}
```

<Note>
  Group/channel tool restrictions are applied in addition to global/agent tool policy (deny still wins). Some channels use different nesting for rooms/channels (e.g., Discord `guilds.*.channels.*`, Slack `channels.*`, Microsoft Teams `teams.*.channels.*`).
</Note>

## Group allowlists

When `channels.whatsapp.groups`, `channels.telegram.groups`, or `channels.imessage.groups` is configured, the keys act as a group allowlist. Use `"*"` to allow all groups while still setting default mention behavior.

<Warning>
  Common confusion: DM pairing approval is not the same as group authorization. For channels that support DM pairing, the pairing store unlocks DMs only. Group commands still require explicit group sender authorization from config allowlists such as `groupAllowFrom` or the documented config fallback for that channel.
</Warning>

Common intents (copy/paste):

<Tabs>
  <Tab title="Disable all group replies">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: { whatsapp: { groupPolicy: "disabled" } },
    }
    ```
  </Tab>

  <Tab title="Allow only specific groups (WhatsApp)">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        whatsapp: {
          groups: {
            "123@g.us": { requireMention: true },
            "456@g.us": { requireMention: false },
          },
        },
      },
    }
    ```
  </Tab>

  <Tab title="Allow all groups but require mention">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```
  </Tab>

  <Tab title="Owner-only triggers (WhatsApp)">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```
  </Tab>
</Tabs>

## Activation (owner-only)

Group owners can toggle per-group activation:

* `/activation mention`
* `/activation always`

Owner is determined by `channels.whatsapp.allowFrom` (or the bot's self E.164 when unset). Send the command as a standalone message. Other surfaces currently ignore `/activation`.

## Context fields

Group inbound payloads set:

* `ChatType=group`
* `GroupSubject` (if known)
* `GroupMembers` (if known)
* `WasMentioned` (mention gating result)
* Telegram forum topics also include `MessageThreadId` and `IsForum`.

Channel-specific notes:

* BlueBubbles can optionally enrich unnamed macOS group participants from the local Contacts database before populating `GroupMembers`. This is off by default and only runs after normal group gating passes.

The agent system prompt includes a group intro on the first turn of a new group session. It reminds the model to respond like a human, avoid Markdown tables, minimize empty lines and follow normal chat spacing, and avoid typing literal `\n` sequences. Channel-sourced group names and participant labels are rendered as fenced untrusted metadata, not inline system instructions.

## iMessage specifics

* Prefer `chat_id:<id>` when routing or allowlisting.
* List chats: `imsg chats --limit 20`.
* Group replies always go back to the same `chat_id`.

## WhatsApp system prompts

See [WhatsApp](/channels/whatsapp#system-prompts) for the canonical WhatsApp system prompt rules, including group and direct prompt resolution, wildcard behavior, and account override semantics.

## WhatsApp specifics

See [Group messages](/channels/group-messages) for WhatsApp-only behavior (history injection, mention handling details).

## Related

* [Broadcast groups](/channels/broadcast-groups)
* [Channel routing](/channels/channel-routing)
* [Group messages](/channels/group-messages)
* [Pairing](/channels/pairing)
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# iMessage

<Warning>
  For new iMessage deployments, use <a href="/channels/bluebubbles">BlueBubbles</a>.

  The `imsg` integration is legacy and may be removed in a future release.
</Warning>

Status: legacy external CLI integration. Gateway spawns `imsg rpc` and communicates over JSON-RPC on stdio (no separate daemon/port).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Preferred iMessage path for new setups.
  </Card>

  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage DMs default to pairing mode.
  </Card>

  <Card title="Configuration reference" icon="settings" href="/gateway/config-channels#imessage">
    Full iMessage field reference.
  </Card>
</CardGroup>

## Quick setup

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        brew install steipete/tap/imsg
        imsg rpc --help
        ```
      </Step>

      <Step title="Configure OpenClaw">
        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          channels: {
            imessage: {
              enabled: true,
              cliPath: "/usr/local/bin/imsg",
              dbPath: "/Users/user/Library/Messages/chat.db",
            },
          },
        }
        ```
      </Step>

      <Step title="Start gateway">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw gateway
        ```
      </Step>

      <Step title="Approve first DM pairing (default dmPolicy)">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw pairing list imessage
        openclaw pairing approve imessage <CODE>
        ```

        Pairing requests expire after 1 hour.
      </Step>
    </Steps>
  </Tab>

  <Tab title="Remote Mac over SSH">
    OpenClaw only requires a stdio-compatible `cliPath`, so you can point `cliPath` at a wrapper script that SSHes to a remote Mac and runs `imsg`.

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    #!/usr/bin/env bash
    exec ssh -T gateway-host imsg "$@"
    ```

    Recommended config when attachments are enabled:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        imessage: {
          enabled: true,
          cliPath: "~/.openclaw/scripts/imsg-ssh",
          remoteHost: "user@gateway-host", // used for SCP attachment fetches
          includeAttachments: true,
          // Optional: override allowed attachment roots.
          // Defaults include /Users/*/Library/Messages/Attachments
          attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
          remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
        },
      },
    }
    ```

    If `remoteHost` is not set, OpenClaw attempts to auto-detect it by parsing the SSH wrapper script.
    `remoteHost` must be `host` or `user@host` (no spaces or SSH options).
    OpenClaw uses strict host-key checking for SCP, so the relay host key must already exist in `~/.ssh/known_hosts`.
    Attachment paths are validated against allowed roots (`attachmentRoots` / `remoteAttachmentRoots`).
  </Tab>
</Tabs>

## Requirements and permissions (macOS)

* Messages must be signed in on the Mac running `imsg`.
* Full Disk Access is required for the process context running OpenClaw/`imsg` (Messages DB access).
* Automation permission is required to send messages through Messages.app.

<Tip>
  Permissions are granted per process context. If gateway runs headless (LaunchAgent/SSH), run a one-time interactive command in that same context to trigger prompts:

  ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
  imsg chats --limit 1
  # or
  imsg send <handle> "test"
  ```
</Tip>

## Access control and routing

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` controls direct messages:

    * `pairing` (default)
    * `allowlist`
    * `open` (requires `allowFrom` to include `"*"`)
    * `disabled`

    Allowlist field: `channels.imessage.allowFrom`.

    Allowlist entries can be handles or chat targets (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).
  </Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` controls group handling:

    * `allowlist` (default when configured)
    * `open`
    * `disabled`

    Group sender allowlist: `channels.imessage.groupAllowFrom`.

    Runtime fallback: if `groupAllowFrom` is unset, iMessage group sender checks fall back to `allowFrom` when available.
    Runtime note: if `channels.imessage` is completely missing, runtime falls back to `groupPolicy="allowlist"` and logs a warning (even if `channels.defaults.groupPolicy` is set).

    Mention gating for groups:

    * iMessage has no native mention metadata
    * mention detection uses regex patterns (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    * with no configured patterns, mention gating cannot be enforced

    Control commands from authorized senders can bypass mention gating in groups.
  </Tab>

  <Tab title="Sessions and deterministic replies">
    * DMs use direct routing; groups use group routing.
    * With default `session.dmScope=main`, iMessage DMs collapse into the agent main session.
    * Group sessions are isolated (`agent:<agentId>:imessage:group:<chat_id>`).
    * Replies route back to iMessage using originating channel/target metadata.

    Group-ish thread behavior:

    Some multi-participant iMessage threads can arrive with `is_group=false`.
    If that `chat_id` is explicitly configured under `channels.imessage.groups`, OpenClaw treats it as group traffic (group gating + group session isolation).
  </Tab>
</Tabs>

## ACP conversation bindings

Legacy iMessage chats can also be bound to ACP sessions.

Fast operator flow:

* Run `/acp spawn codex --bind here` inside the DM or allowed group chat.
* Future messages in that same iMessage conversation route to the spawned ACP session.
* `/new` and `/reset` reset the same bound ACP session in place.
* `/acp close` closes the ACP session and removes the binding.

Configured persistent bindings are supported through top-level `bindings[]` entries with `type: "acp"` and `match.channel: "imessage"`.

`match.peer.id` can use:

* normalized DM handle such as `+15555550123` or `user@example.com`
* `chat_id:<id>` (recommended for stable group bindings)
* `chat_guid:<guid>`
* `chat_identifier:<identifier>`

Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: { agent: "codex", backend: "acpx", mode: "persistent" },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "imessage",
        accountId: "default",
        peer: { kind: "group", id: "chat_id:123" },
      },
      acp: { label: "codex-group" },
    },
  ],
}
```

See [ACP Agents](/tools/acp-agents) for shared ACP binding behavior.

## Deployment patterns

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    Use a dedicated Apple ID and macOS user so bot traffic is isolated from your personal Messages profile.

    Typical flow:

    1. Create/sign in a dedicated macOS user.
    2. Sign into Messages with the bot Apple ID in that user.
    3. Install `imsg` in that user.
    4. Create SSH wrapper so OpenClaw can run `imsg` in that user context.
    5. Point `channels.imessage.accounts.<id>.cliPath` and `.dbPath` to that user profile.

    First run may require GUI approvals (Automation + Full Disk Access) in that bot user session.
  </Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Common topology:

    * gateway runs on Linux/VM
    * iMessage + `imsg` runs on a Mac in your tailnet
    * `cliPath` wrapper uses SSH to run `imsg`
    * `remoteHost` enables SCP attachment fetches

    Example:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        imessage: {
          enabled: true,
          cliPath: "~/.openclaw/scripts/imsg-ssh",
          remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
          includeAttachments: true,
          dbPath: "/Users/bot/Library/Messages/chat.db",
        },
      },
    }
    ```

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    #!/usr/bin/env bash
    exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
    ```

    Use SSH keys so both SSH and SCP are non-interactive.
    Ensure the host key is trusted first (for example `ssh bot@mac-mini.tailnet-1234.ts.net`) so `known_hosts` is populated.
  </Accordion>

  <Accordion title="Multi-account pattern">
    iMessage supports per-account config under `channels.imessage.accounts`.

    Each account can override fields such as `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, history settings, and attachment root allowlists.
  </Accordion>
</AccordionGroup>

## Media, chunking, and delivery targets

<AccordionGroup>
  <Accordion title="Attachments and media">
    * inbound attachment ingestion is optional: `channels.imessage.includeAttachments`
    * remote attachment paths can be fetched via SCP when `remoteHost` is set
    * attachment paths must match allowed roots:
      * `channels.imessage.attachmentRoots` (local)
      * `channels.imessage.remoteAttachmentRoots` (remote SCP mode)
      * default root pattern: `/Users/*/Library/Messages/Attachments`
    * SCP uses strict host-key checking (`StrictHostKeyChecking=yes`)
    * outbound media size uses `channels.imessage.mediaMaxMb` (default 16 MB)
  </Accordion>

  <Accordion title="Outbound chunking">
    * text chunk limit: `channels.imessage.textChunkLimit` (default 4000)
    * chunk mode: `channels.imessage.chunkMode`
      * `length` (default)
      * `newline` (paragraph-first splitting)
  </Accordion>

  <Accordion title="Addressing formats">
    Preferred explicit targets:

    * `chat_id:123` (recommended for stable routing)
    * `chat_guid:...`
    * `chat_identifier:...`

    Handle targets are also supported:

    * `imessage:+1555...`
    * `sms:+1555...`
    * `user@example.com`

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    imsg chats --limit 20
    ```
  </Accordion>
</AccordionGroup>

## Config writes

iMessage allows channel-initiated config writes by default (for `/config set|unset` when `commands.config: true`).

Disable:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Troubleshooting

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Validate the binary and RPC support:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    imsg rpc --help
    openclaw channels status --probe
    ```

    If probe reports RPC unsupported, update `imsg`.
  </Accordion>

  <Accordion title="DMs are ignored">
    Check:

    * `channels.imessage.dmPolicy`
    * `channels.imessage.allowFrom`
    * pairing approvals (`openclaw pairing list imessage`)
  </Accordion>

  <Accordion title="Group messages are ignored">
    Check:

    * `channels.imessage.groupPolicy`
    * `channels.imessage.groupAllowFrom`
    * `channels.imessage.groups` allowlist behavior
    * mention pattern configuration (`agents.list[].groupChat.mentionPatterns`)
  </Accordion>

  <Accordion title="Remote attachments fail">
    Check:

    * `channels.imessage.remoteHost`
    * `channels.imessage.remoteAttachmentRoots`
    * SSH/SCP key auth from the gateway host
    * host key exists in `~/.ssh/known_hosts` on the gateway host
    * remote path readability on the Mac running Messages
  </Accordion>

  <Accordion title="macOS permission prompts were missed">
    Re-run in an interactive GUI terminal in the same user/session context and approve prompts:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    imsg chats --limit 1
    imsg send <handle> "test"
    ```

    Confirm Full Disk Access + Automation are granted for the process context that runs OpenClaw/`imsg`.
  </Accordion>
</AccordionGroup>

## Configuration reference pointers

* [Configuration reference - iMessage](/gateway/config-channels#imessage)
* [Gateway configuration](/gateway/configuration)
* [Pairing](/channels/pairing)
* [BlueBubbles](/channels/bluebubbles)

## Related

* [Channels Overview](/channels) — all supported channels
* [Pairing](/channels/pairing) — DM authentication and pairing flow
* [Groups](/channels/groups) — group chat behavior and mention gating
* [Channel Routing](/channels/channel-routing) — session routing for messages
* [Security](/gateway/security) — access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# LINE

LINE connects to OpenClaw via the LINE Messaging API. The plugin runs as a webhook
receiver on the gateway and uses your channel access token + channel secret for
authentication.

Status: downloadable plugin. Direct messages, group chats, media, locations, Flex
messages, template messages, and quick replies are supported. Reactions and threads
are not supported.

## Install

Install LINE before configuring the channel:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw plugins install @openclaw/line
```

Local checkout (when running from a git repo):

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw plugins install ./path/to/local/line-plugin
```

## Setup

1. Create a LINE Developers account and open the Console:
   [https://developers.line.biz/console/](https://developers.line.biz/console/)
2. Create (or pick) a Provider and add a **Messaging API** channel.
3. Copy the **Channel access token** and **Channel secret** from the channel settings.
4. Enable **Use webhook** in the Messaging API settings.
5. Set the webhook URL to your gateway endpoint (HTTPS required):

```
https://gateway-host/line/webhook
```

The gateway responds to LINE's webhook verification (GET) and inbound events (POST).
If you need a custom path, set `channels.line.webhookPath` or
`channels.line.accounts.<id>.webhookPath` and update the URL accordingly.

Security note:

* LINE signature verification is body-dependent (HMAC over the raw body), so OpenClaw applies strict pre-auth body limits and timeout before verification.
* OpenClaw processes webhook events from the verified raw request bytes. Upstream middleware-transformed `req.body` values are ignored for signature-integrity safety.

## Configure

Minimal config:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

Public DM config:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "open",
      allowFrom: ["*"],
    },
  },
}
```

Env vars (default account only):

* `LINE_CHANNEL_ACCESS_TOKEN`
* `LINE_CHANNEL_SECRET`

Token/secret files:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

`tokenFile` and `secretFile` must point to regular files. Symlinks are rejected.

Multiple accounts:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## Access control

Direct messages default to pairing. Unknown senders get a pairing code and their
messages are ignored until approved.

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Allowlists and policies:

* `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
* `channels.line.allowFrom`: allowlisted LINE user IDs for DMs; `dmPolicy: "open"` requires `["*"]`
* `channels.line.groupPolicy`: `allowlist | open | disabled`
* `channels.line.groupAllowFrom`: allowlisted LINE user IDs for groups
* Per-group overrides: `channels.line.groups.<groupId>.allowFrom`
* Runtime note: if `channels.line` is completely missing, runtime falls back to `groupPolicy="allowlist"` for group checks (even if `channels.defaults.groupPolicy` is set).

LINE IDs are case-sensitive. Valid IDs look like:

* User: `U` + 32 hex chars
* Group: `C` + 32 hex chars
* Room: `R` + 32 hex chars

## Message behavior

* Text is chunked at 5000 characters.
* Markdown formatting is stripped; code blocks and tables are converted into Flex
  cards when possible.
* Streaming responses are buffered; LINE receives full chunks with a loading
  animation while the agent works.
* Media downloads are capped by `channels.line.mediaMaxMb` (default 10).
* Inbound media is saved under `~/.openclaw/media/inbound/` before it is passed
  to the agent, matching the shared media store used by other bundled channel
  plugins.

## Channel data (rich messages)

Use `channelData.line` to send quick replies, locations, Flex cards, or template
messages.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Status card",
        contents: {
          /* Flex payload */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no",
      },
    },
  },
}
```

The LINE plugin also ships a `/card` command for Flex message presets:

```
/card info "Welcome" "Thanks for joining!"
```

## ACP support

LINE supports ACP (Agent Communication Protocol) conversation bindings:

* `/acp spawn <agent> --bind here` binds the current LINE chat to an ACP session without creating a child thread.
* Configured ACP bindings and active conversation-bound ACP sessions work on LINE like other conversation channels.

See [ACP agents](/tools/acp-agents) for details.

## Outbound media

The LINE plugin supports sending images, videos, and audio files through the agent message tool. Media is sent via the LINE-specific delivery path with appropriate preview and tracking handling:

* **Images**: sent as LINE image messages with automatic preview generation.
* **Videos**: sent with explicit preview and content-type handling.
* **Audio**: sent as LINE audio messages.

Outbound media URLs must be public HTTPS URLs. OpenClaw validates the target hostname before handing the URL to LINE and rejects loopback, link-local, and private-network targets.

Generic media sends fall back to the existing image-only route when a LINE-specific path is not available.

## Troubleshooting

* **Webhook verification fails:** ensure the webhook URL is HTTPS and the
  `channelSecret` matches the LINE console.
* **No inbound events:** confirm the webhook path matches `channels.line.webhookPath`
  and that the gateway is reachable from LINE.
* **Media download errors:** raise `channels.line.mediaMaxMb` if media exceeds the
  default limit.

## Related

* [Channels Overview](/channels) — all supported channels
* [Pairing](/channels/pairing) — DM authentication and pairing flow
* [Groups](/channels/groups) — group chat behavior and mention gating
* [Channel Routing](/channels/channel-routing) — session routing for messages
* [Security](/gateway/security) — access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Mattermost

Status: downloadable plugin (bot token + WebSocket events). Channels, groups, and DMs are supported. Mattermost is a self-hostable team messaging platform; see the official site at [mattermost.com](https://mattermost.com) for product details and downloads.

## Install

Install Mattermost before configuring the channel:

<Tabs>
  <Tab title="npm registry">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw plugins install @openclaw/mattermost
    ```
  </Tab>

  <Tab title="Local checkout">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw plugins install ./path/to/local/mattermost-plugin
    ```
  </Tab>
</Tabs>

Details: [Plugins](/tools/plugin)

## Quick setup

<Steps>
  <Step title="Ensure plugin is available">
    Current packaged OpenClaw releases already bundle it. Older/custom installs can add it manually with the commands above.
  </Step>

  <Step title="Create a Mattermost bot">
    Create a Mattermost bot account and copy the **bot token**.
  </Step>

  <Step title="Copy the base URL">
    Copy the Mattermost **base URL** (e.g., `https://chat.example.com`).
  </Step>

  <Step title="Configure OpenClaw and start the gateway">
    Minimal config:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        mattermost: {
          enabled: true,
          botToken: "mm-token",
          baseUrl: "https://chat.example.com",
          dmPolicy: "pairing",
        },
      },
    }
    ```
  </Step>
</Steps>

## Native slash commands

Native slash commands are opt-in. When enabled, OpenClaw registers `oc_*` slash commands via the Mattermost API and receives callback POSTs on the gateway HTTP server.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    mattermost: {
      commands: {
        native: true,
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // Use when Mattermost cannot reach the gateway directly (reverse proxy/public URL).
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
    },
  },
}
```

<AccordionGroup>
  <Accordion title="Behavior notes">
    * `native: "auto"` defaults to disabled for Mattermost. Set `native: true` to enable.
    * If `callbackUrl` is omitted, OpenClaw derives one from gateway host/port + `callbackPath`.
    * For multi-account setups, `commands` can be set at the top level or under `channels.mattermost.accounts.<id>.commands` (account values override top-level fields).
    * Command callbacks are validated with the per-command tokens returned by Mattermost when OpenClaw registers `oc_*` commands.
    * OpenClaw refreshes current Mattermost command registration before accepting each callback so stale tokens from deleted or regenerated slash commands stop being accepted without a gateway restart.
    * Callback validation fails closed if the Mattermost API cannot confirm the command is still current; failed validations are cached briefly, concurrent lookups are coalesced, and fresh lookup starts are rate-limited per command to bound replay pressure.
    * Slash callbacks fail closed when registration failed, startup was partial, or the callback token does not match the resolved command's registered token (a token valid for one command cannot reach upstream validation for a different command).
  </Accordion>

  <Accordion title="Reachability requirement">
    The callback endpoint must be reachable from the Mattermost server.

    * Do not set `callbackUrl` to `localhost` unless Mattermost runs on the same host/network namespace as OpenClaw.
    * Do not set `callbackUrl` to your Mattermost base URL unless that URL reverse-proxies `/api/channels/mattermost/command` to OpenClaw.
    * A quick check is `curl https://<gateway-host>/api/channels/mattermost/command`; a GET should return `405 Method Not Allowed` from OpenClaw, not `404`.
  </Accordion>

  <Accordion title="Mattermost egress allowlist">
    If your callback targets private/tailnet/internal addresses, set Mattermost `ServiceSettings.AllowedUntrustedInternalConnections` to include the callback host/domain.

    Use host/domain entries, not full URLs.

    * Good: `gateway.tailnet-name.ts.net`
    * Bad: `https://gateway.tailnet-name.ts.net`
  </Accordion>
</AccordionGroup>

## Environment variables (default account)

Set these on the gateway host if you prefer env vars:

* `MATTERMOST_BOT_TOKEN=...`
* `MATTERMOST_URL=https://chat.example.com`

<Note>
  Env vars apply only to the **default** account (`default`). Other accounts must use config values.

  `MATTERMOST_URL` cannot be set from a workspace `.env`; see [Workspace `.env` files](/gateway/security).
</Note>

## Chat modes

Mattermost responds to DMs automatically. Channel behavior is controlled by `chatmode`:

<Tabs>
  <Tab title="oncall (default)">
    Respond only when @mentioned in channels.
  </Tab>

  <Tab title="onmessage">
    Respond to every channel message.
  </Tab>

  <Tab title="onchar">
    Respond when a message starts with a trigger prefix.
  </Tab>
</Tabs>

Config example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

Notes:

* `onchar` still responds to explicit @mentions.
* `channels.mattermost.requireMention` is honored for legacy configs but `chatmode` is preferred.

## Threading and sessions

Use `channels.mattermost.replyToMode` to control whether channel and group replies stay in the main channel or start a thread under the triggering post.

* `off` (default): only reply in a thread when the inbound post is already in one.
* `first`: for top-level channel/group posts, start a thread under that post and route the conversation to a thread-scoped session.
* `all`: same behavior as `first` for Mattermost today.
* Direct messages ignore this setting and stay non-threaded.

Config example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    mattermost: {
      replyToMode: "all",
    },
  },
}
```

Notes:

* Thread-scoped sessions use the triggering post id as the thread root.
* `first` and `all` are currently equivalent because once Mattermost has a thread root, follow-up chunks and media continue in that same thread.

## Access control (DMs)

* Default: `channels.mattermost.dmPolicy = "pairing"` (unknown senders get a pairing code).
* Approve via:
  * `openclaw pairing list mattermost`
  * `openclaw pairing approve mattermost <CODE>`
* Public DMs: `channels.mattermost.dmPolicy="open"` plus `channels.mattermost.allowFrom=["*"]`.

## Channels (groups)

* Default: `channels.mattermost.groupPolicy = "allowlist"` (mention-gated).
* Allowlist senders with `channels.mattermost.groupAllowFrom` (user IDs recommended).
* Per-channel mention overrides live under `channels.mattermost.groups.<channelId>.requireMention` or `channels.mattermost.groups["*"].requireMention` for a default.
* `@username` matching is mutable and only enabled when `channels.mattermost.dangerouslyAllowNameMatching: true`.
* Open channels: `channels.mattermost.groupPolicy="open"` (mention-gated).
* Runtime note: if `channels.mattermost` is completely missing, runtime falls back to `groupPolicy="allowlist"` for group checks (even if `channels.defaults.groupPolicy` is set).

Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    mattermost: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: true },
        "team-channel-id": { requireMention: false },
      },
    },
  },
}
```

## Targets for outbound delivery

Use these target formats with `openclaw message send` or cron/webhooks:

* `channel:<id>` for a channel
* `user:<id>` for a DM
* `@username` for a DM (resolved via the Mattermost API)

<Warning>
  Bare opaque IDs (like `64ifufp...`) are **ambiguous** in Mattermost (user ID vs channel ID).

  OpenClaw resolves them **user-first**:

  * If the ID exists as a user (`GET /api/v4/users/<id>` succeeds), OpenClaw sends a **DM** by resolving the direct channel via `/api/v4/channels/direct`.
  * Otherwise the ID is treated as a **channel ID**.

  If you need deterministic behavior, always use the explicit prefixes (`user:<id>` / `channel:<id>`).
</Warning>

## DM channel retry

When OpenClaw sends to a Mattermost DM target and needs to resolve the direct channel first, it retries transient direct-channel creation failures by default.

Use `channels.mattermost.dmChannelRetry` to tune that behavior globally for the Mattermost plugin, or `channels.mattermost.accounts.<id>.dmChannelRetry` for one account.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    mattermost: {
      dmChannelRetry: {
        maxRetries: 3,
        initialDelayMs: 1000,
        maxDelayMs: 10000,
        timeoutMs: 30000,
      },
    },
  },
}
```

Notes:

* This applies only to DM channel creation (`/api/v4/channels/direct`), not every Mattermost API call.
* Retries apply to transient failures such as rate limits, 5xx responses, and network or timeout errors.
* 4xx client errors other than `429` are treated as permanent and are not retried.

## Preview streaming

Mattermost streams thinking, tool activity, and partial reply text into a single **draft preview post** that finalizes in place when the final answer is safe to send. The preview updates on the same post id instead of spamming the channel with per-chunk messages. Media/error finals cancel pending preview edits and use normal delivery instead of flushing a throwaway preview post.

Enable via `channels.mattermost.streaming`:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    mattermost: {
      streaming: "partial", // off | partial | block | progress
    },
  },
}
```

<AccordionGroup>
  <Accordion title="Streaming modes">
    * `partial` is the usual choice: one preview post that is edited as the reply grows, then finalized with the complete answer.
    * `block` uses append-style draft chunks inside the preview post.
    * `progress` shows a status preview while generating and only posts the final answer at completion.
    * `off` disables preview streaming.
  </Accordion>

  <Accordion title="Streaming behavior notes">
    * If the stream cannot be finalized in place (for example the post was deleted mid-stream), OpenClaw falls back to sending a fresh final post so the reply is never lost.
    * Reasoning-only payloads are suppressed from channel posts, including text that arrives as a `> Reasoning:` blockquote. Set `/reasoning on` to see thinking in other surfaces; the Mattermost final post keeps the answer only.
    * See [Streaming](/concepts/streaming#preview-streaming-modes) for the channel-mapping matrix.
  </Accordion>
</AccordionGroup>

## Reactions (message tool)

* Use `message action=react` with `channel=mattermost`.
* `messageId` is the Mattermost post id.
* `emoji` accepts names like `thumbsup` or `:+1:` (colons are optional).
* Set `remove=true` (boolean) to remove a reaction.
* Reaction add/remove events are forwarded as system events to the routed agent session.

Examples:

```
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

Config:

* `channels.mattermost.actions.reactions`: enable/disable reaction actions (default true).
* Per-account override: `channels.mattermost.accounts.<id>.actions.reactions`.

## Interactive buttons (message tool)

Send messages with clickable buttons. When a user clicks a button, the agent receives the selection and can respond.

Enable buttons by adding `inlineButtons` to the channel capabilities:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    mattermost: {
      capabilities: ["inlineButtons"],
    },
  },
}
```

Use `message action=send` with a `buttons` parameter. Buttons are a 2D array (rows of buttons):

```
message action=send channel=mattermost target=channel:<channelId> buttons=[[{"text":"Yes","callback_data":"yes"},{"text":"No","callback_data":"no"}]]
```

Button fields:

<ParamField path="text" type="string" required>
  Display label.
</ParamField>

<ParamField path="callback_data" type="string" required>
  Value sent back on click (used as the action ID).
</ParamField>

<ParamField path="style" type="&#x22;default&#x22; | &#x22;primary&#x22; | &#x22;danger&#x22;">
  Button style.
</ParamField>

When a user clicks a button:

<Steps>
  <Step title="Buttons replaced with confirmation">
    All buttons are replaced with a confirmation line (e.g., "✓ **Yes** selected by @user").
  </Step>

  <Step title="Agent receives the selection">
    The agent receives the selection as an inbound message and responds.
  </Step>
</Steps>

<AccordionGroup>
  <Accordion title="Implementation notes">
    * Button callbacks use HMAC-SHA256 verification (automatic, no config needed).
    * Mattermost strips callback data from its API responses (security feature), so all buttons are removed on click - partial removal is not possible.
    * Action IDs containing hyphens or underscores are sanitized automatically (Mattermost routing limitation).
  </Accordion>

  <Accordion title="Config and reachability">
    * `channels.mattermost.capabilities`: array of capability strings. Add `"inlineButtons"` to enable the buttons tool description in the agent system prompt.
    * `channels.mattermost.interactions.callbackBaseUrl`: optional external base URL for button callbacks (for example `https://gateway.example.com`). Use this when Mattermost cannot reach the gateway at its bind host directly.
    * In multi-account setups, you can also set the same field under `channels.mattermost.accounts.<id>.interactions.callbackBaseUrl`.
    * If `interactions.callbackBaseUrl` is omitted, OpenClaw derives the callback URL from `gateway.customBindHost` + `gateway.port`, then falls back to `http://localhost:<port>`.
    * Reachability rule: the button callback URL must be reachable from the Mattermost server. `localhost` only works when Mattermost and OpenClaw run on the same host/network namespace.
    * If your callback target is private/tailnet/internal, add its host/domain to Mattermost `ServiceSettings.AllowedUntrustedInternalConnections`.
  </Accordion>
</AccordionGroup>

### Direct API integration (external scripts)

External scripts and webhooks can post buttons directly via the Mattermost REST API instead of going through the agent's `message` tool. Use `buildButtonAttachments()` from the plugin when possible; if posting raw JSON, follow these rules:

**Payload structure:**

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channel_id: "<channelId>",
  message: "Choose an option:",
  props: {
    attachments: [
      {
        actions: [
          {
            id: "mybutton01", // alphanumeric only - see below
            type: "button", // required, or clicks are silently ignored
            name: "Approve", // display label
            style: "primary", // optional: "default", "primary", "danger"
            integration: {
              url: "https://gateway.example.com/mattermost/interactions/default",
              context: {
                action_id: "mybutton01", // must match button id (for name lookup)
                action: "approve",
                // ... any custom fields ...
                _token: "<hmac>", // see HMAC section below
              },
            },
          },
        ],
      },
    ],
  },
}
```

<Warning>
  **Critical rules**

  1. Attachments go in `props.attachments`, not top-level `attachments` (silently ignored).
  2. Every action needs `type: "button"` - without it, clicks are swallowed silently.
  3. Every action needs an `id` field - Mattermost ignores actions without IDs.
  4. Action `id` must be **alphanumeric only** (`[a-zA-Z0-9]`). Hyphens and underscores break Mattermost's server-side action routing (returns 404). Strip them before use.
  5. `context.action_id` must match the button's `id` so the confirmation message shows the button name (e.g., "Approve") instead of a raw ID.
  6. `context.action_id` is required - the interaction handler returns 400 without it.
</Warning>

**HMAC token generation**

The gateway verifies button clicks with HMAC-SHA256. External scripts must generate tokens that match the gateway's verification logic:

<Steps>
  <Step title="Derive the secret from the bot token">
    `HMAC-SHA256(key="openclaw-mattermost-interactions", data=botToken)`
  </Step>

  <Step title="Build the context object">
    Build the context object with all fields **except** `_token`.
  </Step>

  <Step title="Serialize with sorted keys">
    Serialize with **sorted keys** and **no spaces** (the gateway uses `JSON.stringify` with sorted keys, which produces compact output).
  </Step>

  <Step title="Sign the payload">
    `HMAC-SHA256(key=secret, data=serializedContext)`
  </Step>

  <Step title="Add the token">
    Add the resulting hex digest as `_token` in the context.
  </Step>
</Steps>

Python example:

```python theme={"theme":{"light":"min-light","dark":"min-dark"}}
import hmac, hashlib, json

secret = hmac.new(
    b"openclaw-mattermost-interactions",
    bot_token.encode(), hashlib.sha256
).hexdigest()

ctx = {"action_id": "mybutton01", "action": "approve"}
payload = json.dumps(ctx, sort_keys=True, separators=(",", ":"))
token = hmac.new(secret.encode(), payload.encode(), hashlib.sha256).hexdigest()

context = {**ctx, "_token": token}
```

<AccordionGroup>
  <Accordion title="Common HMAC pitfalls">
    * Python's `json.dumps` adds spaces by default (`{"key": "val"}`). Use `separators=(",", ":")` to match JavaScript's compact output (`{"key":"val"}`).
    * Always sign **all** context fields (minus `_token`). The gateway strips `_token` then signs everything remaining. Signing a subset causes silent verification failure.
    * Use `sort_keys=True` - the gateway sorts keys before signing, and Mattermost may reorder context fields when storing the payload.
    * Derive the secret from the bot token (deterministic), not random bytes. The secret must be the same across the process that creates buttons and the gateway that verifies.
  </Accordion>
</AccordionGroup>

## Directory adapter

The Mattermost plugin includes a directory adapter that resolves channel and user names via the Mattermost API. This enables `#channel-name` and `@username` targets in `openclaw message send` and cron/webhook deliveries.

No configuration is needed - the adapter uses the bot token from the account config.

## Multi-account

Mattermost supports multiple accounts under `channels.mattermost.accounts`:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## Troubleshooting

<AccordionGroup>
  <Accordion title="No replies in channels">
    Ensure the bot is in the channel and mention it (oncall), use a trigger prefix (onchar), or set `chatmode: "onmessage"`.
  </Accordion>

  <Accordion title="Auth or multi-account errors">
    * Check the bot token, base URL, and whether the account is enabled.
    * Multi-account issues: env vars only apply to the `default` account.
  </Accordion>

  <Accordion title="Native slash commands fail">
    * `Unauthorized: invalid command token.`: OpenClaw did not accept the callback token. Typical causes:
      * slash command registration failed or only partially completed at startup
      * the callback is hitting the wrong gateway/account
      * Mattermost still has old commands pointing at a previous callback target
      * the gateway restarted without reactivating slash commands
    * If native slash commands stop working, check logs for `mattermost: failed to register slash commands` or `mattermost: native slash commands enabled but no commands could be registered`.
    * If `callbackUrl` is omitted and logs warn that the callback resolved to `http://127.0.0.1:18789/...`, that URL is probably only reachable when Mattermost runs on the same host/network namespace as OpenClaw. Set an explicit externally reachable `commands.callbackUrl` instead.
  </Accordion>

  <Accordion title="Buttons issues">
    * Buttons appear as white boxes: the agent may be sending malformed button data. Check that each button has both `text` and `callback_data` fields.
    * Buttons render but clicks do nothing: verify `AllowedUntrustedInternalConnections` in Mattermost server config includes `127.0.0.1 localhost`, and that `EnablePostActionIntegration` is `true` in ServiceSettings.
    * Buttons return 404 on click: the button `id` likely contains hyphens or underscores. Mattermost's action router breaks on non-alphanumeric IDs. Use `[a-zA-Z0-9]` only.
    * Gateway logs `invalid _token`: HMAC mismatch. Check that you sign all context fields (not a subset), use sorted keys, and use compact JSON (no spaces). See the HMAC section above.
    * Gateway logs `missing _token in context`: the `_token` field is not in the button's context. Ensure it is included when building the integration payload.
    * Confirmation shows raw ID instead of button name: `context.action_id` does not match the button's `id`. Set both to the same sanitized value.
    * Agent doesn't know about buttons: add `capabilities: ["inlineButtons"]` to the Mattermost channel config.
  </Accordion>
</AccordionGroup>

## Related

* [Channel Routing](/channels/channel-routing) - session routing for messages
* [Channels Overview](/channels) - all supported channels
* [Groups](/channels/groups) - group chat behavior and mention gating
* [Pairing](/channels/pairing) - DM authentication and pairing flow
* [Security](/gateway/security) - access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Microsoft Teams

Status: text + DM attachments are supported; channel/group file sending requires `sharePointSiteId` + Graph permissions (see [Sending files in group chats](#sending-files-in-group-chats)). Polls are sent via Adaptive Cards. Message actions expose explicit `upload-file` for file-first sends.

## Bundled plugin

Microsoft Teams ships as a bundled plugin in current OpenClaw releases, so no
separate install is required in the normal packaged build.

If you are on an older build or a custom install that excludes bundled Teams,
install the npm package directly:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw plugins install @openclaw/msteams
```

Use the bare package to follow the current official release tag. Pin an exact
version only when you need a reproducible install.

Local checkout (when running from a git repo):

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw plugins install ./path/to/local/msteams-plugin
```

Details: [Plugins](/tools/plugin)

## Quick setup

The [`@microsoft/teams.cli`](https://www.npmjs.com/package/@microsoft/teams.cli) handles bot registration, manifest creation, and credential generation in a single command.

**1. Install and log in**

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
npm install -g @microsoft/teams.cli@preview
teams login
teams status   # verify you're logged in and see your tenant info
```

<Note>
  The Teams CLI is currently in preview. Commands and flags may change between releases.
</Note>

**2. Start a tunnel** (Teams can't reach localhost)

Install and authenticate the devtunnel CLI if you haven't already ([getting started guide](https://learn.microsoft.com/en-us/azure/developer/dev-tunnels/get-started)).

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
# One-time setup (persistent URL across sessions):
devtunnel create my-openclaw-bot --allow-anonymous
devtunnel port create my-openclaw-bot -p 3978 --protocol auto

# Each dev session:
devtunnel host my-openclaw-bot
# Your endpoint: https://<tunnel-id>.devtunnels.ms/api/messages
```

<Note>
  `--allow-anonymous` is required because Teams cannot authenticate with devtunnels. Each incoming bot request is still validated by the Teams SDK automatically.
</Note>

Alternatives: `ngrok http 3978` or `tailscale funnel 3978` (but these may change URLs each session).

**3. Create the app**

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
teams app create \
  --name "OpenClaw" \
  --endpoint "https://<your-tunnel-url>/api/messages"
```

This single command:

* Creates an Entra ID (Azure AD) application
* Generates a client secret
* Builds and uploads a Teams app manifest (with icons)
* Registers the bot (Teams-managed by default - no Azure subscription needed)

The output will show `CLIENT_ID`, `CLIENT_SECRET`, `TENANT_ID`, and a **Teams App ID** - note these for the next steps. It also offers to install the app in Teams directly.

**4. Configure OpenClaw** using the credentials from the output:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<CLIENT_ID>",
      appPassword: "<CLIENT_SECRET>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

Or use environment variables directly: `MSTEAMS_APP_ID`, `MSTEAMS_APP_PASSWORD`, `MSTEAMS_TENANT_ID`.

**5. Install the app in Teams**

`teams app create` will prompt you to install the app - select "Install in Teams". If you skipped it, you can get the link later:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
teams app get <teamsAppId> --install-link
```

**6. Verify everything works**

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
teams app doctor <teamsAppId>
```

This runs diagnostics across bot registration, AAD app config, manifest validity, and SSO setup.

For production deployments, consider using [federated authentication](/channels/msteams#federated-authentication-certificate-plus-managed-identity) (certificate or managed identity) instead of client secrets.

<Note>
  Group chats are blocked by default (`channels.msteams.groupPolicy: "allowlist"`). To allow group replies, set `channels.msteams.groupAllowFrom`, or use `groupPolicy: "open"` to allow any member (mention-gated).
</Note>

## Goals

* Talk to OpenClaw via Teams DMs, group chats, or channels.
* Keep routing deterministic: replies always go back to the channel they arrived on.
* Default to safe channel behavior (mentions required unless configured otherwise).

## Config writes

By default, Microsoft Teams is allowed to write config updates triggered by `/config set|unset` (requires `commands.config: true`).

Disable with:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: { msteams: { configWrites: false } },
}
```

## Access control (DMs + groups)

**DM access**

* Default: `channels.msteams.dmPolicy = "pairing"`. Unknown senders are ignored until approved.
* `channels.msteams.allowFrom` should use stable AAD object IDs.
* Do not rely on UPN/display-name matching for allowlists - they can change. OpenClaw disables direct name matching by default; opt in explicitly with `channels.msteams.dangerouslyAllowNameMatching: true`.
* The wizard can resolve names to IDs via Microsoft Graph when credentials allow.

**Group access**

* Default: `channels.msteams.groupPolicy = "allowlist"` (blocked unless you add `groupAllowFrom`). Use `channels.defaults.groupPolicy` to override the default when unset.
* `channels.msteams.groupAllowFrom` controls which senders can trigger in group chats/channels (falls back to `channels.msteams.allowFrom`).
* Set `groupPolicy: "open"` to allow any member (still mention-gated by default).
* To allow **no channels**, set `channels.msteams.groupPolicy: "disabled"`.

Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Teams + channel allowlist**

* Scope group/channel replies by listing teams and channels under `channels.msteams.teams`.
* Keys should use stable Teams conversation IDs from Teams links, not mutable display names.
* When `groupPolicy="allowlist"` and a teams allowlist is present, only listed teams/channels are accepted (mention-gated).
* The configure wizard accepts `Team/Channel` entries and stores them for you.
* On startup, OpenClaw resolves team/channel and user allowlist names to IDs (when Graph permissions allow)
  and logs the mapping; unresolved team/channel names are kept as typed but ignored for routing by default unless `channels.msteams.dangerouslyAllowNameMatching: true` is enabled.

Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```

<details>
  <summary><strong>Manual setup (without the Teams CLI)</strong></summary>

  If you can't use the Teams CLI, you can set up the bot manually through the Azure Portal.

  ### How it works

  1. Ensure the Microsoft Teams plugin is available (bundled in current releases).
  2. Create an **Azure Bot** (App ID + secret + tenant ID).
  3. Build a **Teams app package** that references the bot and includes the RSC permissions below.
  4. Upload/install the Teams app into a team (or personal scope for DMs).
  5. Configure `msteams` in `~/.openclaw/openclaw.json` (or env vars) and start the gateway.
  6. The gateway listens for Bot Framework webhook traffic on `/api/messages` by default.

  ### Step 1: Create Azure Bot

  1. Go to [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
  2. Fill in the **Basics** tab:

     | Field              | Value                                                    |
     | ------------------ | -------------------------------------------------------- |
     | **Bot handle**     | Your bot name, e.g., `openclaw-msteams` (must be unique) |
     | **Subscription**   | Select your Azure subscription                           |
     | **Resource group** | Create new or use existing                               |
     | **Pricing tier**   | **Free** for dev/testing                                 |
     | **Type of App**    | **Single Tenant** (recommended - see note below)         |
     | **Creation type**  | **Create new Microsoft App ID**                          |

  <Warning>
    Creation of new multi-tenant bots was deprecated after 2025-07-31. Use **Single Tenant** for new bots.
  </Warning>

  3. Click **Review + create** → **Create** (wait \~1-2 minutes)

  ### Step 2: Get Credentials

  1. Go to your Azure Bot resource → **Configuration**
  2. Copy **Microsoft App ID** → this is your `appId`
  3. Click **Manage Password** → go to the App Registration
  4. Under **Certificates & secrets** → **New client secret** → copy the **Value** → this is your `appPassword`
  5. Go to **Overview** → copy **Directory (tenant) ID** → this is your `tenantId`

  ### Step 3: Configure Messaging Endpoint

  1. In Azure Bot → **Configuration**
  2. Set **Messaging endpoint** to your webhook URL:
     * Production: `https://your-domain.com/api/messages`
     * Local dev: Use a tunnel (see [Local Development](#local-development-tunneling) below)

  ### Step 4: Enable Teams Channel

  1. In Azure Bot → **Channels**
  2. Click **Microsoft Teams** → Configure → Save
  3. Accept the Terms of Service

  ### Step 5: Build Teams App Manifest

  * Include a `bot` entry with `botId = <App ID>`.
  * Scopes: `personal`, `team`, `groupChat`.
  * `supportsFiles: true` (required for personal scope file handling).
  * Add RSC permissions (see [RSC Permissions](#current-teams-rsc-permissions-manifest)).
  * Create icons: `outline.png` (32x32) and `color.png` (192x192).
  * Zip all three files together: `manifest.json`, `outline.png`, `color.png`.

  ### Step 6: Configure OpenClaw

  ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
  {
    channels: {
      msteams: {
        enabled: true,
        appId: "<APP_ID>",
        appPassword: "<APP_PASSWORD>",
        tenantId: "<TENANT_ID>",
        webhook: { port: 3978, path: "/api/messages" },
      },
    },
  }
  ```

  Environment variables: `MSTEAMS_APP_ID`, `MSTEAMS_APP_PASSWORD`, `MSTEAMS_TENANT_ID`.

  ### Step 7: Run the Gateway

  The Teams channel starts automatically when the plugin is available and `msteams` config exists with credentials.
</details>

## Federated authentication (certificate plus managed identity)

> Added in 2026.4.11

For production deployments, OpenClaw supports **federated authentication** as a more secure alternative to client secrets. Two methods are available:

### Option A: Certificate-based authentication

Use a PEM certificate registered with your Entra ID app registration.

**Setup:**

1. Generate or obtain a certificate (PEM format with private key).
2. In Entra ID → App Registration → **Certificates & secrets** → **Certificates** → Upload the public certificate.

**Config:**

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      tenantId: "<TENANT_ID>",
      authType: "federated",
      certificatePath: "/path/to/cert.pem",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

**Env vars:**

* `MSTEAMS_AUTH_TYPE=federated`
* `MSTEAMS_CERTIFICATE_PATH=/path/to/cert.pem`

### Option B: Azure Managed Identity

Use Azure Managed Identity for passwordless authentication. This is ideal for deployments on Azure infrastructure (AKS, App Service, Azure VMs) where a managed identity is available.

**How it works:**

1. The bot pod/VM has a managed identity (system-assigned or user-assigned).
2. A **federated identity credential** links the managed identity to the Entra ID app registration.
3. At runtime, OpenClaw uses `@azure/identity` to acquire tokens from the Azure IMDS endpoint (`169.254.169.254`).
4. The token is passed to the Teams SDK for bot authentication.

**Prerequisites:**

* Azure infrastructure with managed identity enabled (AKS workload identity, App Service, VM)
* Federated identity credential created on the Entra ID app registration
* Network access to IMDS (`169.254.169.254:80`) from the pod/VM

**Config (system-assigned managed identity):**

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      tenantId: "<TENANT_ID>",
      authType: "federated",
      useManagedIdentity: true,
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

**Config (user-assigned managed identity):**

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      tenantId: "<TENANT_ID>",
      authType: "federated",
      useManagedIdentity: true,
      managedIdentityClientId: "<MI_CLIENT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

**Env vars:**

* `MSTEAMS_AUTH_TYPE=federated`
* `MSTEAMS_USE_MANAGED_IDENTITY=true`
* `MSTEAMS_MANAGED_IDENTITY_CLIENT_ID=<client-id>` (only for user-assigned)

### AKS Workload Identity Setup

For AKS deployments using workload identity:

1. **Enable workload identity** on your AKS cluster.

2. **Create a federated identity credential** on the Entra ID app registration:

   ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
   az ad app federated-credential create --id <APP_OBJECT_ID> --parameters '{
     "name": "my-bot-workload-identity",
     "issuer": "<AKS_OIDC_ISSUER_URL>",
     "subject": "system:serviceaccount:<NAMESPACE>:<SERVICE_ACCOUNT>",
     "audiences": ["api://AzureADTokenExchange"]
   }'
   ```

3. **Annotate the Kubernetes service account** with the app client ID:

   ```yaml theme={"theme":{"light":"min-light","dark":"min-dark"}}
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: my-bot-sa
     annotations:
       azure.workload.identity/client-id: "<APP_CLIENT_ID>"
   ```

4. **Label the pod** for workload identity injection:

   ```yaml theme={"theme":{"light":"min-light","dark":"min-dark"}}
   metadata:
     labels:
       azure.workload.identity/use: "true"
   ```

5. **Ensure network access** to IMDS (`169.254.169.254`) - if using NetworkPolicy, add an egress rule allowing traffic to `169.254.169.254/32` on port 80.

### Auth type comparison

| Method               | Config                                         | Pros                               | Cons                                  |
| -------------------- | ---------------------------------------------- | ---------------------------------- | ------------------------------------- |
| **Client secret**    | `appPassword`                                  | Simple setup                       | Secret rotation required, less secure |
| **Certificate**      | `authType: "federated"` + `certificatePath`    | No shared secret over network      | Certificate management overhead       |
| **Managed Identity** | `authType: "federated"` + `useManagedIdentity` | Passwordless, no secrets to manage | Azure infrastructure required         |

**Default behavior:** When `authType` is not set, OpenClaw defaults to client secret authentication. Existing configurations continue to work without changes.

## Local development (tunneling)

Teams can't reach `localhost`. Use a persistent dev tunnel so your URL stays the same across sessions:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
# One-time setup:
devtunnel create my-openclaw-bot --allow-anonymous
devtunnel port create my-openclaw-bot -p 3978 --protocol auto

# Each dev session:
devtunnel host my-openclaw-bot
```

Alternatives: `ngrok http 3978` or `tailscale funnel 3978` (URLs may change each session).

If your tunnel URL changes, update the endpoint:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
teams app update <teamsAppId> --endpoint "https://<new-url>/api/messages"
```

## Testing the Bot

**Run diagnostics:**

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
teams app doctor <teamsAppId>
```

Checks bot registration, AAD app, manifest, and SSO configuration in one pass.

**Send a test message:**

1. Install the Teams app (use the install link from `teams app get <id> --install-link`)
2. Find the bot in Teams and send a DM
3. Check gateway logs for incoming activity

## Environment variables

All config keys can be set via environment variables instead:

* `MSTEAMS_APP_ID`
* `MSTEAMS_APP_PASSWORD`
* `MSTEAMS_TENANT_ID`
* `MSTEAMS_AUTH_TYPE` (optional: `"secret"` or `"federated"`)
* `MSTEAMS_CERTIFICATE_PATH` (federated + certificate)
* `MSTEAMS_CERTIFICATE_THUMBPRINT` (optional, not required for auth)
* `MSTEAMS_USE_MANAGED_IDENTITY` (federated + managed identity)
* `MSTEAMS_MANAGED_IDENTITY_CLIENT_ID` (user-assigned MI only)

## Member info action

OpenClaw exposes a Graph-backed `member-info` action for Microsoft Teams so agents and automations can resolve channel member details (display name, email, role) directly from Microsoft Graph.

Requirements:

* `Member.Read.Group` RSC permission (already in the recommended manifest)
* For cross-team lookups: `User.Read.All` Graph Application permission with admin consent

The action is gated by `channels.msteams.actions.memberInfo` (default: enabled when Graph credentials are available).

## History context

* `channels.msteams.historyLimit` controls how many recent channel/group messages are wrapped into the prompt.
* Falls back to `messages.groupChat.historyLimit`. Set `0` to disable (default 50).
* Fetched thread history is filtered by sender allowlists (`allowFrom` / `groupAllowFrom`), so thread context seeding only includes messages from allowed senders.
* Quoted attachment context (`ReplyTo*` derived from Teams reply HTML) is currently passed as received.
* In other words, allowlists gate who can trigger the agent; only specific supplemental context paths are filtered today.
* DM history can be limited with `channels.msteams.dmHistoryLimit` (user turns). Per-user overrides: `channels.msteams.dms["<user_id>"].historyLimit`.

## Current Teams RSC permissions (manifest)

These are the **existing resourceSpecific permissions** in our Teams app manifest. They only apply inside the team/chat where the app is installed.

**For channels (team scope):**

* `ChannelMessage.Read.Group` (Application) - receive all channel messages without @mention
* `ChannelMessage.Send.Group` (Application)
* `Member.Read.Group` (Application)
* `Owner.Read.Group` (Application)
* `ChannelSettings.Read.Group` (Application)
* `TeamMember.Read.Group` (Application)
* `TeamSettings.Read.Group` (Application)

**For group chats:**

* `ChatMessage.Read.Chat` (Application) - receive all group chat messages without @mention

To add RSC permissions via the Teams CLI:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
teams app rsc add <teamsAppId> ChannelMessage.Read.Group --type Application
```

## Example Teams manifest (redacted)

Minimal, valid example with the required fields. Replace IDs and URLs.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  $schema: "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  manifestVersion: "1.23",
  version: "1.0.0",
  id: "00000000-0000-0000-0000-000000000000",
  name: { short: "OpenClaw" },
  developer: {
    name: "Your Org",
    websiteUrl: "https://example.com",
    privacyUrl: "https://example.com/privacy",
    termsOfUseUrl: "https://example.com/terms",
  },
  description: { short: "OpenClaw in Teams", full: "OpenClaw in Teams" },
  icons: { outline: "outline.png", color: "color.png" },
  accentColor: "#5B6DEF",
  bots: [
    {
      botId: "11111111-1111-1111-1111-111111111111",
      scopes: ["personal", "team", "groupChat"],
      isNotificationOnly: false,
      supportsCalling: false,
      supportsVideo: false,
      supportsFiles: true,
    },
  ],
  webApplicationInfo: {
    id: "11111111-1111-1111-1111-111111111111",
  },
  authorization: {
    permissions: {
      resourceSpecific: [
        { name: "ChannelMessage.Read.Group", type: "Application" },
        { name: "ChannelMessage.Send.Group", type: "Application" },
        { name: "Member.Read.Group", type: "Application" },
        { name: "Owner.Read.Group", type: "Application" },
        { name: "ChannelSettings.Read.Group", type: "Application" },
        { name: "TeamMember.Read.Group", type: "Application" },
        { name: "TeamSettings.Read.Group", type: "Application" },
        { name: "ChatMessage.Read.Chat", type: "Application" },
      ],
    },
  },
}
```

### Manifest caveats (must-have fields)

* `bots[].botId` **must** match the Azure Bot App ID.
* `webApplicationInfo.id` **must** match the Azure Bot App ID.
* `bots[].scopes` must include the surfaces you plan to use (`personal`, `team`, `groupChat`).
* `bots[].supportsFiles: true` is required for file handling in personal scope.
* `authorization.permissions.resourceSpecific` must include channel read/send if you want channel traffic.

### Updating an existing app

To update an already-installed Teams app (e.g., to add RSC permissions):

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
# Download, edit, and re-upload the manifest
teams app manifest download <teamsAppId> manifest.json
# Edit manifest.json locally...
teams app manifest upload manifest.json <teamsAppId>
# Version is auto-bumped if content changed
```

After updating, reinstall the app in each team for new permissions to take effect, and **fully quit and relaunch Teams** (not just close the window) to clear cached app metadata.

<details>
  <summary>Manual manifest update (without CLI)</summary>

  1. Update your `manifest.json` with the new settings
  2. **Increment the `version` field** (e.g., `1.0.0` → `1.1.0`)
  3. **Re-zip** the manifest with icons (`manifest.json`, `outline.png`, `color.png`)
  4. Upload the new zip:
     * **Teams Admin Center:** Teams apps → Manage apps → find your app → Upload new version
     * **Sideload:** In Teams → Apps → Manage your apps → Upload a custom app
</details>

## Capabilities: RSC only vs Graph

### With **Teams RSC only** (app installed, no Graph API permissions)

Works:

* Read channel message **text** content.
* Send channel message **text** content.
* Receive **personal (DM)** file attachments.

Does NOT work:

* Channel/group **image or file contents** (payload only includes HTML stub).
* Downloading attachments stored in SharePoint/OneDrive.
* Reading message history (beyond the live webhook event).

### With **Teams RSC + Microsoft Graph Application permissions**

Adds:

* Downloading hosted contents (images pasted into messages).
* Downloading file attachments stored in SharePoint/OneDrive.
* Reading channel/chat message history via Graph.

### RSC vs Graph API

| Capability              | RSC Permissions      | Graph API                           |
| ----------------------- | -------------------- | ----------------------------------- |
| **Real-time messages**  | Yes (via webhook)    | No (polling only)                   |
| **Historical messages** | No                   | Yes (can query history)             |
| **Setup complexity**    | App manifest only    | Requires admin consent + token flow |
| **Works offline**       | No (must be running) | Yes (query anytime)                 |

**Bottom line:** RSC is for real-time listening; Graph API is for historical access. For catching up on missed messages while offline, you need Graph API with `ChannelMessage.Read.All` (requires admin consent).

## Graph-enabled media + history (required for channels)

If you need images/files in **channels** or want to fetch **message history**, you must enable Microsoft Graph permissions and grant admin consent.

1. In Entra ID (Azure AD) **App Registration**, add Microsoft Graph **Application permissions**:
   * `ChannelMessage.Read.All` (channel attachments + history)
   * `Chat.Read.All` or `ChatMessage.Read.All` (group chats)
2. **Grant admin consent** for the tenant.
3. Bump the Teams app **manifest version**, re-upload, and **reinstall the app in Teams**.
4. **Fully quit and relaunch Teams** to clear cached app metadata.

**Additional permission for user mentions:** User @mentions work out of the box for users in the conversation. However, if you want to dynamically search and mention users who are **not in the current conversation**, add `User.Read.All` (Application) permission and grant admin consent.

## Known limitations

### Webhook timeouts

Teams delivers messages via HTTP webhook. If processing takes too long (e.g., slow LLM responses), you may see:

* Gateway timeouts
* Teams retrying the message (causing duplicates)
* Dropped replies

OpenClaw handles this by returning quickly and sending replies proactively, but very slow responses may still cause issues.

### Formatting

Teams markdown is more limited than Slack or Discord:

* Basic formatting works: **bold**, *italic*, `code`, links
* Complex markdown (tables, nested lists) may not render correctly
* Adaptive Cards are supported for polls and semantic presentation sends (see below)

## Configuration

Key settings (see `/gateway/configuration` for shared channel patterns):

* `channels.msteams.enabled`: enable/disable the channel.
* `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`: bot credentials.
* `channels.msteams.webhook.port` (default `3978`)
* `channels.msteams.webhook.path` (default `/api/messages`)
* `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled` (default: pairing)
* `channels.msteams.allowFrom`: DM allowlist (AAD object IDs recommended). The wizard resolves names to IDs during setup when Graph access is available.
* `channels.msteams.dangerouslyAllowNameMatching`: break-glass toggle to re-enable mutable UPN/display-name matching and direct team/channel name routing.
* `channels.msteams.textChunkLimit`: outbound text chunk size.
* `channels.msteams.chunkMode`: `length` (default) or `newline` to split on blank lines (paragraph boundaries) before length chunking.
* `channels.msteams.mediaAllowHosts`: allowlist for inbound attachment hosts (defaults to Microsoft/Teams domains).
* `channels.msteams.mediaAuthAllowHosts`: allowlist for attaching Authorization headers on media retries (defaults to Graph + Bot Framework hosts).
* `channels.msteams.requireMention`: require @mention in channels/groups (default true).
* `channels.msteams.replyStyle`: `thread | top-level` (see [Reply Style](#reply-style-threads-vs-posts)).
* `channels.msteams.teams.<teamId>.replyStyle`: per-team override.
* `channels.msteams.teams.<teamId>.requireMention`: per-team override.
* `channels.msteams.teams.<teamId>.tools`: default per-team tool policy overrides (`allow`/`deny`/`alsoAllow`) used when a channel override is missing.
* `channels.msteams.teams.<teamId>.toolsBySender`: default per-team per-sender tool policy overrides (`"*"` wildcard supported).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle`: per-channel override.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention`: per-channel override.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.tools`: per-channel tool policy overrides (`allow`/`deny`/`alsoAllow`).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender`: per-channel per-sender tool policy overrides (`"*"` wildcard supported).
* `toolsBySender` keys should use explicit prefixes:
  `id:`, `e164:`, `username:`, `name:` (legacy unprefixed keys still map to `id:` only).
* `channels.msteams.actions.memberInfo`: enable or disable the Graph-backed member info action (default: enabled when Graph credentials are available).
* `channels.msteams.authType`: authentication type - `"secret"` (default) or `"federated"`.
* `channels.msteams.certificatePath`: path to PEM certificate file (federated + certificate auth).
* `channels.msteams.certificateThumbprint`: certificate thumbprint (optional, not required for auth).
* `channels.msteams.useManagedIdentity`: enable managed identity auth (federated mode).
* `channels.msteams.managedIdentityClientId`: client ID for user-assigned managed identity.
* `channels.msteams.sharePointSiteId`: SharePoint site ID for file uploads in group chats/channels (see [Sending files in group chats](#sending-files-in-group-chats)).

## Routing and sessions

* Session keys follow the standard agent format (see [/concepts/session](/concepts/session)):
  * Direct messages share the main session (`agent:<agentId>:<mainKey>`).
  * Channel/group messages use conversation id:
    * `agent:<agentId>:msteams:channel:<conversationId>`
    * `agent:<agentId>:msteams:group:<conversationId>`

## Reply style: threads vs posts

Teams recently introduced two channel UI styles over the same underlying data model:

| Style                    | Description                                               | Recommended `replyStyle` |
| ------------------------ | --------------------------------------------------------- | ------------------------ |
| **Posts** (classic)      | Messages appear as cards with threaded replies underneath | `thread` (default)       |
| **Threads** (Slack-like) | Messages flow linearly, more like Slack                   | `top-level`              |

**The problem:** The Teams API does not expose which UI style a channel uses. If you use the wrong `replyStyle`:

* `thread` in a Threads-style channel → replies appear nested awkwardly
* `top-level` in a Posts-style channel → replies appear as separate top-level posts instead of in-thread

**Solution:** Configure `replyStyle` per-channel based on how the channel is set up:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    msteams: {
      replyStyle: "thread",
      teams: {
        "19:abc...@thread.tacv2": {
          channels: {
            "19:xyz...@thread.tacv2": {
              replyStyle: "top-level",
            },
          },
        },
      },
    },
  },
}
```

## Attachments and images

**Current limitations:**

* **DMs:** Images and file attachments work via Teams bot file APIs.
* **Channels/groups:** Attachments live in M365 storage (SharePoint/OneDrive). The webhook payload only includes an HTML stub, not the actual file bytes. **Graph API permissions are required** to download channel attachments.
* For explicit file-first sends, use `action=upload-file` with `media` / `filePath` / `path`; optional `message` becomes the accompanying text/comment, and `filename` overrides the uploaded name.

Without Graph permissions, channel messages with images will be received as text-only (the image content is not accessible to the bot).
By default, OpenClaw only downloads media from Microsoft/Teams hostnames. Override with `channels.msteams.mediaAllowHosts` (use `["*"]` to allow any host).
Authorization headers are only attached for hosts in `channels.msteams.mediaAuthAllowHosts` (defaults to Graph + Bot Framework hosts). Keep this list strict (avoid multi-tenant suffixes).

## Sending files in group chats

Bots can send files in DMs using the FileConsentCard flow (built-in). However, **sending files in group chats/channels** requires additional setup:

| Context                  | How files are sent                           | Setup needed                                    |
| ------------------------ | -------------------------------------------- | ----------------------------------------------- |
| **DMs**                  | FileConsentCard → user accepts → bot uploads | Works out of the box                            |
| **Group chats/channels** | Upload to SharePoint → share link            | Requires `sharePointSiteId` + Graph permissions |
| **Images (any context)** | Base64-encoded inline                        | Works out of the box                            |

### Why group chats need SharePoint

Bots don't have a personal OneDrive drive (the `/me/drive` Graph API endpoint doesn't work for application identities). To send files in group chats/channels, the bot uploads to a **SharePoint site** and creates a sharing link.

### Setup

1. **Add Graph API permissions** in Entra ID (Azure AD) → App Registration:
   * `Sites.ReadWrite.All` (Application) - upload files to SharePoint
   * `Chat.Read.All` (Application) - optional, enables per-user sharing links

2. **Grant admin consent** for the tenant.

3. **Get your SharePoint site ID:**

   ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
   # Via Graph Explorer or curl with a valid token:
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # Example: for a site at "contoso.sharepoint.com/sites/BotFiles"
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # Response includes: "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **Configure OpenClaw:**

   ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
   {
     channels: {
       msteams: {
         // ... other config ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2",
       },
     },
   }
   ```

### Sharing behavior

| Permission                              | Sharing behavior                                          |
| --------------------------------------- | --------------------------------------------------------- |
| `Sites.ReadWrite.All` only              | Organization-wide sharing link (anyone in org can access) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | Per-user sharing link (only chat members can access)      |

Per-user sharing is more secure as only the chat participants can access the file. If `Chat.Read.All` permission is missing, the bot falls back to organization-wide sharing.

### Fallback behavior

| Scenario                                          | Result                                             |
| ------------------------------------------------- | -------------------------------------------------- |
| Group chat + file + `sharePointSiteId` configured | Upload to SharePoint, send sharing link            |
| Group chat + file + no `sharePointSiteId`         | Attempt OneDrive upload (may fail), send text only |
| Personal chat + file                              | FileConsentCard flow (works without SharePoint)    |
| Any context + image                               | Base64-encoded inline (works without SharePoint)   |

### Files stored location

Uploaded files are stored in a `/OpenClawShared/` folder in the configured SharePoint site's default document library.

## Polls (Adaptive Cards)

OpenClaw sends Teams polls as Adaptive Cards (there is no native Teams poll API).

* CLI: `openclaw message poll --channel msteams --target conversation:<id> ...`
* Votes are recorded by the gateway in `~/.openclaw/msteams-polls.json`.
* The gateway must stay online to record votes.
* Polls do not auto-post result summaries yet (inspect the store file if needed).

## Presentation cards

Send semantic presentation payloads to Teams users or conversations using the `message` tool or CLI. OpenClaw renders them as Teams Adaptive Cards from the generic presentation contract.

The `presentation` parameter accepts semantic blocks. When `presentation` is provided, the message text is optional.

**Agent tool:**

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  action: "send",
  channel: "msteams",
  target: "user:<id>",
  presentation: {
    title: "Hello",
    blocks: [{ type: "text", text: "Hello!" }],
  },
}
```

**CLI:**

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --presentation '{"title":"Hello","blocks":[{"type":"text","text":"Hello!"}]}'
```

For target format details, see [Target formats](#target-formats) below.

## Target formats

MSTeams targets use prefixes to distinguish between users and conversations:

| Target type         | Format                           | Example                                             |
| ------------------- | -------------------------------- | --------------------------------------------------- |
| User (by ID)        | `user:<aad-object-id>`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197`         |
| User (by name)      | `user:<display-name>`            | `user:John Smith` (requires Graph API)              |
| Group/channel       | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2`            |
| Group/channel (raw) | `<conversation-id>`              | `19:abc123...@thread.tacv2` (if contains `@thread`) |

**CLI examples:**

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
# Send to a user by ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# Send to a user by display name (triggers Graph API lookup)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Send to a group chat or channel
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Send a presentation card to a conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --presentation '{"title":"Hello","blocks":[{"type":"text","text":"Hello"}]}'
```

**Agent tool examples:**

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  action: "send",
  channel: "msteams",
  target: "user:John Smith",
  message: "Hello!",
}
```

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  action: "send",
  channel: "msteams",
  target: "conversation:19:abc...@thread.tacv2",
  presentation: {
    title: "Hello",
    blocks: [{ type: "text", text: "Hello" }],
  },
}
```

<Note>
  Without the `user:` prefix, names default to group or team resolution. Always use `user:` when targeting people by display name.
</Note>

## Proactive messaging

* Proactive messages are only possible **after** a user has interacted, because we store conversation references at that point.
* See `/gateway/configuration` for `dmPolicy` and allowlist gating.

## Team and Channel IDs (Common Gotcha)

The `groupId` query parameter in Teams URLs is **NOT** the team ID used for configuration. Extract IDs from the URL path instead:

**Team URL:**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    Team conversation ID (URL-decode this)
```

**Channel URL:**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      Channel ID (URL-decode this)
```

**For config:**

* Team key = path segment after `/team/` (URL-decoded, e.g., `19:Bk4j...@thread.tacv2`; older tenants may show `@thread.skype`, which is also valid)
* Channel key = path segment after `/channel/` (URL-decoded)
* **Ignore** the `groupId` query parameter for OpenClaw routing. It is the Microsoft Entra group ID, not the Bot Framework conversation ID used in incoming Teams activities.

## Private channels

Bots have limited support in private channels:

| Feature                      | Standard Channels | Private Channels       |
| ---------------------------- | ----------------- | ---------------------- |
| Bot installation             | Yes               | Limited                |
| Real-time messages (webhook) | Yes               | May not work           |
| RSC permissions              | Yes               | May behave differently |
| @mentions                    | Yes               | If bot is accessible   |
| Graph API history            | Yes               | Yes (with permissions) |

**Workarounds if private channels don't work:**

1. Use standard channels for bot interactions
2. Use DMs - users can always message the bot directly
3. Use Graph API for historical access (requires `ChannelMessage.Read.All`)

## Troubleshooting

### Common issues

* **Images not showing in channels:** Graph permissions or admin consent missing. Reinstall the Teams app and fully quit/reopen Teams.
* **No responses in channel:** mentions are required by default; set `channels.msteams.requireMention=false` or configure per team/channel.
* **Version mismatch (Teams still shows old manifest):** remove + re-add the app and fully quit Teams to refresh.
* **401 Unauthorized from webhook:** Expected when testing manually without Azure JWT - means endpoint is reachable but auth failed. Use Azure Web Chat to test properly.

### Manifest upload errors

* **"Icon file cannot be empty":** The manifest references icon files that are 0 bytes. Create valid PNG icons (32x32 for `outline.png`, 192x192 for `color.png`).
* **"webApplicationInfo.Id already in use":** The app is still installed in another team/chat. Find and uninstall it first, or wait 5-10 minutes for propagation.
* **"Something went wrong" on upload:** Upload via [https://admin.teams.microsoft.com](https://admin.teams.microsoft.com) instead, open browser DevTools (F12) → Network tab, and check the response body for the actual error.
* **Sideload failing:** Try "Upload an app to your org's app catalog" instead of "Upload a custom app" - this often bypasses sideload restrictions.

### RSC permissions not working

1. Verify `webApplicationInfo.id` matches your bot's App ID exactly
2. Re-upload the app and reinstall in the team/chat
3. Check if your org admin has blocked RSC permissions
4. Confirm you're using the right scope: `ChannelMessage.Read.Group` for teams, `ChatMessage.Read.Chat` for group chats

## References

* [Create Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) - Azure Bot setup guide
* [Teams Developer Portal](https://dev.teams.microsoft.com/apps) - create/manage Teams apps
* [Teams app manifest schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
* [Receive channel messages with RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
* [RSC permissions reference](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
* [Teams bot file handling](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4) (channel/group requires Graph)
* [Proactive messaging](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)
* [@microsoft/teams.cli](https://www.npmjs.com/package/@microsoft/teams.cli) - Teams CLI for bot management

## Related

* [Channels Overview](/channels) - all supported channels
* [Pairing](/channels/pairing) - DM authentication and pairing flow
* [Groups](/channels/groups) - group chat behavior and mention gating
* [Channel Routing](/channels/channel-routing) - session routing for messages
* [Security](/gateway/security) - access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Nextcloud Talk

Status: bundled plugin (webhook bot). Direct messages, rooms, reactions, and markdown messages are supported.

## Bundled plugin

Nextcloud Talk ships as a bundled plugin in current OpenClaw releases, so
normal packaged builds do not need a separate install.

If you are on an older build or a custom install that excludes Nextcloud Talk,
install the npm package directly:

Install via CLI (npm registry):

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw plugins install @openclaw/nextcloud-talk
```

Use the bare package to follow the current official release tag. Pin an exact
version only when you need a reproducible install.

Local checkout (when running from a git repo):

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw plugins install ./path/to/local/nextcloud-talk-plugin
```

Details: [Plugins](/tools/plugin)

## Quick setup (beginner)

1. Ensure the Nextcloud Talk plugin is available.
   * Current packaged OpenClaw releases already bundle it.
   * Older/custom installs can add it manually with the commands above.

2. On your Nextcloud server, create a bot:

   ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```

3. Enable the bot in the target room settings.

4. Configure OpenClaw:

   * Config: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   * Or env: `NEXTCLOUD_TALK_BOT_SECRET` (default account only)

   CLI setup:

   ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
   openclaw channels add --channel nextcloud-talk \
     --url https://cloud.example.com \
     --token "<shared-secret>"
   ```

   Equivalent explicit fields:

   ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
   openclaw channels add --channel nextcloud-talk \
     --base-url https://cloud.example.com \
     --secret "<shared-secret>"
   ```

   File-backed secret:

   ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
   openclaw channels add --channel nextcloud-talk \
     --base-url https://cloud.example.com \
     --secret-file /path/to/nextcloud-talk-secret
   ```

5. Restart the gateway (or finish setup).

Minimal config:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## Notes

* Bots cannot initiate DMs. The user must message the bot first.
* Webhook URL must be reachable by the Gateway; set `webhookPublicUrl` if behind a proxy.
* Media uploads are not supported by the bot API; media is sent as URLs.
* The webhook payload does not distinguish DMs vs rooms; set `apiUser` + `apiPassword` to enable room-type lookups (otherwise DMs are treated as rooms).

## Access control (DMs)

* Default: `channels.nextcloud-talk.dmPolicy = "pairing"`. Unknown senders get a pairing code.
* Approve via:
  * `openclaw pairing list nextcloud-talk`
  * `openclaw pairing approve nextcloud-talk <CODE>`
* Public DMs: `channels.nextcloud-talk.dmPolicy="open"` plus `channels.nextcloud-talk.allowFrom=["*"]`.
* `allowFrom` matches Nextcloud user IDs only; display names are ignored.

## Rooms (groups)

* Default: `channels.nextcloud-talk.groupPolicy = "allowlist"` (mention-gated).
* Allowlist rooms with `channels.nextcloud-talk.rooms`:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

* To allow no rooms, keep the allowlist empty or set `channels.nextcloud-talk.groupPolicy="disabled"`.

## Capabilities

| Feature         | Status        |
| --------------- | ------------- |
| Direct messages | Supported     |
| Rooms           | Supported     |
| Threads         | Not supported |
| Media           | URL-only      |
| Reactions       | Supported     |
| Native commands | Not supported |

## Configuration reference (Nextcloud Talk)

Full configuration: [Configuration](/gateway/configuration)

Provider options:

* `channels.nextcloud-talk.enabled`: enable/disable channel startup.
* `channels.nextcloud-talk.baseUrl`: Nextcloud instance URL.
* `channels.nextcloud-talk.botSecret`: bot shared secret.
* `channels.nextcloud-talk.botSecretFile`: regular-file secret path. Symlinks are rejected.
* `channels.nextcloud-talk.apiUser`: API user for room lookups (DM detection).
* `channels.nextcloud-talk.apiPassword`: API/app password for room lookups.
* `channels.nextcloud-talk.apiPasswordFile`: API password file path.
* `channels.nextcloud-talk.webhookPort`: webhook listener port (default: 8788).
* `channels.nextcloud-talk.webhookHost`: webhook host (default: 0.0.0.0).
* `channels.nextcloud-talk.webhookPath`: webhook path (default: /nextcloud-talk-webhook).
* `channels.nextcloud-talk.webhookPublicUrl`: externally reachable webhook URL.
* `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`.
* `channels.nextcloud-talk.allowFrom`: DM allowlist (user IDs). `open` requires `"*"`.
* `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`.
* `channels.nextcloud-talk.groupAllowFrom`: group allowlist (user IDs).
* `channels.nextcloud-talk.rooms`: per-room settings and allowlist.
* `channels.nextcloud-talk.historyLimit`: group history limit (0 disables).
* `channels.nextcloud-talk.dmHistoryLimit`: DM history limit (0 disables).
* `channels.nextcloud-talk.dms`: per-DM overrides (historyLimit).
* `channels.nextcloud-talk.textChunkLimit`: outbound text chunk size (chars).
* `channels.nextcloud-talk.chunkMode`: `length` (default) or `newline` to split on blank lines (paragraph boundaries) before length chunking.
* `channels.nextcloud-talk.blockStreaming`: disable block streaming for this channel.
* `channels.nextcloud-talk.blockStreamingCoalesce`: block streaming coalesce tuning.
* `channels.nextcloud-talk.mediaMaxMb`: inbound media cap (MB).

## Related

* [Channels Overview](/channels) — all supported channels
* [Pairing](/channels/pairing) — DM authentication and pairing flow
* [Groups](/channels/groups) — group chat behavior and mention gating
* [Channel Routing](/channels/channel-routing) — session routing for messages
* [Security](/gateway/security) — access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Pairing

"Pairing" is OpenClaw's explicit access approval step.
It is used in two places:

1. **DM pairing** (who is allowed to talk to the bot)
2. **Node pairing** (which devices/nodes are allowed to join the gateway network)

Security context: [Security](/gateway/security)

## 1) DM pairing (inbound chat access)

When a channel is configured with DM policy `pairing`, unknown senders get a short code and their message is **not processed** until you approve.

Default DM policies are documented in: [Security](/gateway/security)

`dmPolicy: "open"` is public only when the effective DM allowlist includes `"*"`.
Setup and validation require that wildcard for public-open configs. If existing
state contains `open` with concrete `allowFrom` entries, runtime still admits
only those senders, and pairing-store approvals do not widen `open` access.

Pairing codes:

* 8 characters, uppercase, no ambiguous chars (`0O1I`).
* **Expire after 1 hour**. The bot only sends the pairing message when a new request is created (roughly once per hour per sender).
* Pending DM pairing requests are capped at **3 per channel** by default; additional requests are ignored until one expires or is approved.

### Approve a sender

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

If no command owner is configured yet, approving a DM pairing code also bootstraps
`commands.ownerAllowFrom` to the approved sender, such as `telegram:123456789`.
That gives first-time setups an explicit owner for privileged commands and exec
approval prompts. After an owner exists, later pairing approvals only grant DM
access; they do not add more owners.

Supported channels: `bluebubbles`, `discord`, `feishu`, `googlechat`, `imessage`, `irc`, `line`, `matrix`, `mattermost`, `msteams`, `nextcloud-talk`, `nostr`, `openclaw-weixin`, `signal`, `slack`, `synology-chat`, `telegram`, `twitch`, `whatsapp`, `zalo`, `zalouser`.

### Reusable sender groups

Use top-level `accessGroups` when the same trusted sender set should apply to
multiple message channels or to both DM and group allowlists.

Static groups use `type: "message.senders"` and are referenced with
`accessGroup:<name>` from channel allowlists:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  accessGroups: {
    operators: {
      type: "message.senders",
      members: {
        discord: ["discord:123456789012345678"],
        telegram: ["987654321"],
        whatsapp: ["+15551234567"],
      },
    },
  },
  channels: {
    telegram: { dmPolicy: "allowlist", allowFrom: ["accessGroup:operators"] },
    whatsapp: { groupPolicy: "allowlist", groupAllowFrom: ["accessGroup:operators"] },
  },
}
```

Access groups are documented in detail here: [Access groups](/channels/access-groups)

### Where the state lives

Stored under `~/.openclaw/credentials/`:

* Pending requests: `<channel>-pairing.json`
* Approved allowlist store:
  * Default account: `<channel>-allowFrom.json`
  * Non-default account: `<channel>-<accountId>-allowFrom.json`

Account scoping behavior:

* Non-default accounts read/write only their scoped allowlist file.
* Default account uses the channel-scoped unscoped allowlist file.

Treat these as sensitive (they gate access to your assistant).

<Note>
  The pairing allowlist store is for DM access. Group authorization is separate.
  Approving a DM pairing code does not automatically allow that sender to run group
  commands or control the bot in groups. First-owner bootstrap is separate config
  state in `commands.ownerAllowFrom`, and group chat delivery still follows the
  channel's group allowlists (for example `groupAllowFrom`, `groups`, or per-group
  or per-topic overrides depending on the channel).
</Note>

## 2) Node device pairing (iOS/Android/macOS/headless nodes)

Nodes connect to the Gateway as **devices** with `role: node`. The Gateway
creates a device pairing request that must be approved.

### Pair via Telegram (recommended for iOS)

If you use the `device-pair` plugin, you can do first-time device pairing entirely from Telegram:

1. In Telegram, message your bot: `/pair`
2. The bot replies with two messages: an instruction message and a separate **setup code** message (easy to copy/paste in Telegram).
3. On your phone, open the OpenClaw iOS app → Settings → Gateway.
4. Scan the QR code or paste the setup code and connect.
5. Back in Telegram: `/pair pending` (review request IDs, role, and scopes), then approve.

The setup code is a base64-encoded JSON payload that contains:

* `url`: the Gateway WebSocket URL (`ws://...` or `wss://...`)
* `bootstrapToken`: a short-lived single-device bootstrap token used for the initial pairing handshake

That bootstrap token carries the built-in pairing bootstrap profile:

* primary handed-off `node` token stays `scopes: []`
* any handed-off `operator` token stays bounded to the bootstrap allowlist:
  `operator.approvals`, `operator.read`, `operator.talk.secrets`, `operator.write`
* bootstrap scope checks are role-prefixed, not one flat scope pool:
  operator scope entries only satisfy operator requests, and non-operator roles
  must still request scopes under their own role prefix
* later token rotation/revocation remains bounded by both the device's approved
  role contract and the caller session's operator scopes

Treat the setup code like a password while it is valid.

For Tailscale, public, or other remote mobile pairing, use Tailscale Serve/Funnel
or another `wss://` Gateway URL. Plaintext `ws://` setup codes are accepted only
for loopback, private LAN addresses, `.local` Bonjour hosts, and the Android
emulator host. Tailnet CGNAT addresses, `.ts.net` names, and public hosts still
fail closed before QR/setup-code issuance.

### Approve a node device

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

When an explicit approval is denied because the approving paired-device session
was opened with pairing-only scope, the CLI retries the same request with
`operator.admin`. This lets an existing admin-capable paired device recover a new
Control UI/browser pairing without editing `devices/paired.json` by hand. The
Gateway still validates the retried connection; tokens that cannot authenticate
with `operator.admin` remain blocked.

If the same device retries with different auth details (for example different
role/scopes/public key), the previous pending request is superseded and a new
`requestId` is created.

<Note>
  An already paired device does not get broader access silently. If it reconnects asking for more scopes or a broader role, OpenClaw keeps the existing approval as-is and creates a fresh pending upgrade request. Use `openclaw devices list` to compare the currently approved access with the newly requested access before you approve.
</Note>

### Optional trusted-CIDR node auto-approve

Device pairing remains manual by default. For tightly controlled node networks,
you can opt in to first-time node auto-approval with explicit CIDRs or exact IPs:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  gateway: {
    nodes: {
      pairing: {
        autoApproveCidrs: ["192.168.1.0/24"],
      },
    },
  },
}
```

This only applies to fresh `role: node` pairing requests with no requested
scopes. Operator, browser, Control UI, and WebChat clients still require manual
approval. Role, scope, metadata, and public-key changes still require manual
approval.

### Node pairing state storage

Stored under `~/.openclaw/devices/`:

* `pending.json` (short-lived; pending requests expire)
* `paired.json` (paired devices + tokens)

### Notes

* The legacy `node.pair.*` API (CLI: `openclaw nodes pending|approve|reject|remove|rename`) is a
  separate gateway-owned pairing store. WS nodes still require device pairing.
* The pairing record is the durable source of truth for approved roles. Active
  device tokens stay bounded to that approved role set; a stray token entry
  outside the approved roles does not create new access.

## Related docs

* Security model + prompt injection: [Security](/gateway/security)
* Updating safely (run doctor): [Updating](/install/updating)
* Channel configs:
  * Telegram: [Telegram](/channels/telegram)
  * WhatsApp: [WhatsApp](/channels/whatsapp)
  * Signal: [Signal](/channels/signal)
  * BlueBubbles (iMessage): [BlueBubbles](/channels/bluebubbles)
  * iMessage (legacy): [iMessage](/channels/imessage)
  * Discord: [Discord](/channels/discord)
  * Slack: [Slack](/channels/slack)
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Slack

Production-ready for DMs and channels via Slack app integrations. Default mode is Socket Mode; HTTP Request URLs are also supported.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DMs default to pairing mode.
  </Card>

  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Native command behavior and command catalog.
  </Card>

  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Cross-channel diagnostics and repair playbooks.
  </Card>
</CardGroup>

## Choosing Socket Mode or HTTP Request URLs

Both transports are production-ready and reach feature parity for messaging, slash commands, App Home, and interactivity. Pick by deployment shape, not features.

| Concern                      | Socket Mode (default)                                                                | HTTP Request URLs                                                                                              |
| ---------------------------- | ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| Public Gateway URL           | Not required                                                                         | Required (DNS, TLS, reverse proxy or tunnel)                                                                   |
| Outbound network             | Outbound WSS to `wss-primary.slack.com` must be reachable                            | No outbound WS; inbound HTTPS only                                                                             |
| Tokens needed                | Bot token (`xoxb-...`) + App-Level Token (`xapp-...`) with `connections:write`       | Bot token (`xoxb-...`) + Signing Secret                                                                        |
| Dev laptop / behind firewall | Works as-is                                                                          | Needs a public tunnel (ngrok, Cloudflare Tunnel, Tailscale Funnel) or staging Gateway                          |
| Horizontal scaling           | One Socket Mode session per app per host; multiple Gateways need separate Slack apps | Stateless POST handler; multiple Gateway replicas can share one app behind a load balancer                     |
| Multi-account on one Gateway | Supported; each account opens its own WS                                             | Supported; each account needs a unique `webhookPath` (default `/slack/events`) so registrations do not collide |
| Slash command transport      | Delivered over the WS connection; `slash_commands[].url` is ignored                  | Slack POSTs to `slash_commands[].url`; field is required for the command to dispatch                           |
| Request signing              | Not used (auth is the App-Level Token)                                               | Slack signs every request; OpenClaw verifies with `signingSecret`                                              |
| Recovery on connection drop  | Slack SDK auto-reconnects; the gateway's pong-timeout transport tuning applies       | No persistent connection to drop; retries are per-request from Slack                                           |

<Note>
  **Pick Socket Mode** for single-Gateway hosts, dev laptops, and on-prem networks that can reach `*.slack.com` outbound but cannot accept inbound HTTPS.

  **Pick HTTP Request URLs** when running multiple Gateway replicas behind a load balancer, when outbound WSS is blocked but inbound HTTPS is allowed, or when you already terminate Slack webhooks at a reverse proxy.
</Note>

## Quick setup

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create a new Slack app">
        Open [api.slack.com/apps](https://api.slack.com/apps/new) → **Create New App** → **From a manifest** → select your workspace → paste one of the manifests below → **Next** → **Create**.

        <CodeGroup>
          ```json Recommended theme={"theme":{"light":"min-light","dark":"min-dark"}}
          {
            "display_information": {
              "name": "OpenClaw",
              "description": "Slack connector for OpenClaw"
            },
            "features": {
              "bot_user": { "display_name": "OpenClaw", "always_online": true },
              "app_home": {
                "home_tab_enabled": true,
                "messages_tab_enabled": true,
                "messages_tab_read_only_enabled": false
              },
              "slash_commands": [
                {
                  "command": "/openclaw",
                  "description": "Send a message to OpenClaw",
                  "should_escape": false
                }
              ]
            },
            "oauth_config": {
              "scopes": {
                "bot": [
                  "app_mentions:read",
                  "assistant:write",
                  "channels:history",
                  "channels:read",
                  "chat:write",
                  "commands",
                  "emoji:read",
                  "files:read",
                  "files:write",
                  "groups:history",
                  "groups:read",
                  "im:history",
                  "im:read",
                  "im:write",
                  "mpim:history",
                  "mpim:read",
                  "mpim:write",
                  "pins:read",
                  "pins:write",
                  "reactions:read",
                  "reactions:write",
                  "usergroups:read",
                  "users:read"
                ]
              }
            },
            "settings": {
              "socket_mode_enabled": true,
              "event_subscriptions": {
                "bot_events": [
                  "app_home_opened",
                  "app_mention",
                  "channel_rename",
                  "member_joined_channel",
                  "member_left_channel",
                  "message.channels",
                  "message.groups",
                  "message.im",
                  "message.mpim",
                  "pin_added",
                  "pin_removed",
                  "reaction_added",
                  "reaction_removed"
                ]
              }
            }
          }
          ```

          ```json Minimal theme={"theme":{"light":"min-light","dark":"min-dark"}}
          {
            "display_information": {
              "name": "OpenClaw",
              "description": "Slack connector for OpenClaw"
            },
            "features": {
              "bot_user": { "display_name": "OpenClaw", "always_online": true },
              "app_home": {
                "home_tab_enabled": true,
                "messages_tab_enabled": true,
                "messages_tab_read_only_enabled": false
              },
              "slash_commands": [
                {
                  "command": "/openclaw",
                  "description": "Send a message to OpenClaw",
                  "should_escape": false
                }
              ]
            },
            "oauth_config": {
              "scopes": {
                "bot": [
                  "app_mentions:read",
                  "assistant:write",
                  "channels:history",
                  "channels:read",
                  "chat:write",
                  "commands",
                  "groups:history",
                  "groups:read",
                  "im:history",
                  "im:read",
                  "im:write",
                  "users:read"
                ]
              }
            },
            "settings": {
              "socket_mode_enabled": true,
              "event_subscriptions": {
                "bot_events": [
                  "app_home_opened",
                  "app_mention",
                  "message.channels",
                  "message.groups",
                  "message.im"
                ]
              }
            }
          }
          ```
        </CodeGroup>

        <Note>
          **Recommended** matches the bundled Slack plugin's full feature set: App Home, slash commands, files, reactions, pins, group DMs, and emoji/usergroup reads. Pick **Minimal** when workspace policy restricts scopes — it covers DMs, channel/group history, mentions, and slash commands but drops files, reactions, pins, group-DM (`mpim:*`), `emoji:read`, and `usergroups:read`. See [Manifest and scope checklist](#manifest-and-scope-checklist) for per-scope rationale and additive options like extra slash commands.
        </Note>

        After Slack creates the app:

        * **Basic Information → App-Level Tokens → Generate Token and Scopes**: add `connections:write`, save, copy the `xapp-...` value.
        * **Install App → Install to Workspace**: copy the `xoxb-...` Bot User OAuth Token.
      </Step>

      <Step title="Configure OpenClaw">
        Recommended SecretRef setup:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        export SLACK_APP_TOKEN=xapp-...
        export SLACK_BOT_TOKEN=xoxb-...
        cat > slack.socket.patch.json5 <<'JSON5'
        {
          channels: {
            slack: {
              enabled: true,
              mode: "socket",
              appToken: { source: "env", provider: "default", id: "SLACK_APP_TOKEN" },
              botToken: { source: "env", provider: "default", id: "SLACK_BOT_TOKEN" },
            },
          },
        }
        JSON5
        openclaw config patch --file ./slack.socket.patch.json5 --dry-run
        openclaw config patch --file ./slack.socket.patch.json5
        ```

        Env fallback (default account only):

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        SLACK_APP_TOKEN=xapp-...
        SLACK_BOT_TOKEN=xoxb-...
        ```
      </Step>

      <Step title="Start gateway">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw gateway
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="HTTP Request URLs">
    <Steps>
      <Step title="Create a new Slack app">
        Open [api.slack.com/apps](https://api.slack.com/apps/new) → **Create New App** → **From a manifest** → select your workspace → paste one of the manifests below → replace `https://gateway-host.example.com/slack/events` with your public Gateway URL → **Next** → **Create**.

        <CodeGroup>
          ```json Recommended theme={"theme":{"light":"min-light","dark":"min-dark"}}
          {
            "display_information": {
              "name": "OpenClaw",
              "description": "Slack connector for OpenClaw"
            },
            "features": {
              "bot_user": { "display_name": "OpenClaw", "always_online": true },
              "app_home": {
                "home_tab_enabled": true,
                "messages_tab_enabled": true,
                "messages_tab_read_only_enabled": false
              },
              "slash_commands": [
                {
                  "command": "/openclaw",
                  "description": "Send a message to OpenClaw",
                  "should_escape": false,
                  "url": "https://gateway-host.example.com/slack/events"
                }
              ]
            },
            "oauth_config": {
              "scopes": {
                "bot": [
                  "app_mentions:read",
                  "assistant:write",
                  "channels:history",
                  "channels:read",
                  "chat:write",
                  "commands",
                  "emoji:read",
                  "files:read",
                  "files:write",
                  "groups:history",
                  "groups:read",
                  "im:history",
                  "im:read",
                  "im:write",
                  "mpim:history",
                  "mpim:read",
                  "mpim:write",
                  "pins:read",
                  "pins:write",
                  "reactions:read",
                  "reactions:write",
                  "usergroups:read",
                  "users:read"
                ]
              }
            },
            "settings": {
              "event_subscriptions": {
                "request_url": "https://gateway-host.example.com/slack/events",
                "bot_events": [
                  "app_home_opened",
                  "app_mention",
                  "channel_rename",
                  "member_joined_channel",
                  "member_left_channel",
                  "message.channels",
                  "message.groups",
                  "message.im",
                  "message.mpim",
                  "pin_added",
                  "pin_removed",
                  "reaction_added",
                  "reaction_removed"
                ]
              },
              "interactivity": {
                "is_enabled": true,
                "request_url": "https://gateway-host.example.com/slack/events",
                "message_menu_options_url": "https://gateway-host.example.com/slack/events"
              }
            }
          }
          ```

          ```json Minimal theme={"theme":{"light":"min-light","dark":"min-dark"}}
          {
            "display_information": {
              "name": "OpenClaw",
              "description": "Slack connector for OpenClaw"
            },
            "features": {
              "bot_user": { "display_name": "OpenClaw", "always_online": true },
              "app_home": {
                "home_tab_enabled": true,
                "messages_tab_enabled": true,
                "messages_tab_read_only_enabled": false
              },
              "slash_commands": [
                {
                  "command": "/openclaw",
                  "description": "Send a message to OpenClaw",
                  "should_escape": false,
                  "url": "https://gateway-host.example.com/slack/events"
                }
              ]
            },
            "oauth_config": {
              "scopes": {
                "bot": [
                  "app_mentions:read",
                  "assistant:write",
                  "channels:history",
                  "channels:read",
                  "chat:write",
                  "commands",
                  "groups:history",
                  "groups:read",
                  "im:history",
                  "im:read",
                  "im:write",
                  "users:read"
                ]
              }
            },
            "settings": {
              "event_subscriptions": {
                "request_url": "https://gateway-host.example.com/slack/events",
                "bot_events": [
                  "app_home_opened",
                  "app_mention",
                  "message.channels",
                  "message.groups",
                  "message.im"
                ]
              },
              "interactivity": {
                "is_enabled": true,
                "request_url": "https://gateway-host.example.com/slack/events",
                "message_menu_options_url": "https://gateway-host.example.com/slack/events"
              }
            }
          }
          ```
        </CodeGroup>

        <Note>
          **Recommended** matches the bundled Slack plugin's full feature set; **Minimal** drops files, reactions, pins, group-DM (`mpim:*`), `emoji:read`, and `usergroups:read` for restrictive workspaces. See [Manifest and scope checklist](#manifest-and-scope-checklist) for per-scope rationale.
        </Note>

        <Info>
          The three URL fields (`slash_commands[].url`, `event_subscriptions.request_url`, and `interactivity.request_url` / `message_menu_options_url`) all point at the same OpenClaw endpoint. Slack's manifest schema requires them named separately, but OpenClaw routes by payload type so a single `webhookPath` (default `/slack/events`) is enough. Slash commands without `slash_commands[].url` will silently no-op in HTTP mode.
        </Info>

        After Slack creates the app:

        * **Basic Information → App Credentials**: copy the **Signing Secret** for request verification.
        * **Install App → Install to Workspace**: copy the `xoxb-...` Bot User OAuth Token.
      </Step>

      <Step title="Configure OpenClaw">
        Recommended SecretRef setup:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        export SLACK_BOT_TOKEN=xoxb-...
        export SLACK_SIGNING_SECRET=...
        cat > slack.http.patch.json5 <<'JSON5'
        {
          channels: {
            slack: {
              enabled: true,
              mode: "http",
              botToken: { source: "env", provider: "default", id: "SLACK_BOT_TOKEN" },
              signingSecret: { source: "env", provider: "default", id: "SLACK_SIGNING_SECRET" },
              webhookPath: "/slack/events",
            },
          },
        }
        JSON5
        openclaw config patch --file ./slack.http.patch.json5 --dry-run
        openclaw config patch --file ./slack.http.patch.json5
        ```

        <Note>
          Use unique webhook paths for multi-account HTTP

          Give each account a distinct `webhookPath` (default `/slack/events`) so registrations do not collide.
        </Note>
      </Step>

      <Step title="Start gateway">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw gateway
        ```
      </Step>
    </Steps>
  </Tab>
</Tabs>

## Socket Mode transport tuning

OpenClaw sets the Slack SDK client pong timeout to 15 seconds by default for Socket Mode. Override the transport settings only when you need workspace- or host-specific tuning:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    slack: {
      mode: "socket",
      socketMode: {
        clientPingTimeout: 20000,
        serverPingTimeout: 30000,
        pingPongLoggingEnabled: false,
      },
    },
  },
}
```

Use this only for Socket Mode workspaces that log Slack websocket pong/server-ping timeouts or run on hosts with known event-loop starvation. `clientPingTimeout` is the pong wait after the SDK sends a client ping; `serverPingTimeout` is the wait for Slack server pings. App messages and events remain application state, not transport liveness signals.

## Manifest and scope checklist

The base Slack app manifest is the same for Socket Mode and HTTP Request URLs. Only the `settings` block (and the slash command `url`) differs.

Base manifest (Socket Mode default):

```json theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": { "display_name": "OpenClaw", "always_online": true },
    "app_home": {
      "home_tab_enabled": true,
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "app_mentions:read",
        "assistant:write",
        "channels:history",
        "channels:read",
        "chat:write",
        "commands",
        "emoji:read",
        "files:read",
        "files:write",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "pins:read",
        "pins:write",
        "reactions:read",
        "reactions:write",
        "usergroups:read",
        "users:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_home_opened",
        "app_mention",
        "channel_rename",
        "member_joined_channel",
        "member_left_channel",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "pin_added",
        "pin_removed",
        "reaction_added",
        "reaction_removed"
      ]
    }
  }
}
```

For **HTTP Request URLs mode**, replace `settings` with the HTTP variant and add `url` to each slash command. Public URL required:

```json theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  "features": {
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false,
        "url": "https://gateway-host.example.com/slack/events"
      }
    ]
  },
  "settings": {
    "event_subscriptions": {
      "request_url": "https://gateway-host.example.com/slack/events",
      "bot_events": [
        "app_home_opened",
        "app_mention",
        "channel_rename",
        "member_joined_channel",
        "member_left_channel",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "pin_added",
        "pin_removed",
        "reaction_added",
        "reaction_removed"
      ]
    },
    "interactivity": {
      "is_enabled": true,
      "request_url": "https://gateway-host.example.com/slack/events",
      "message_menu_options_url": "https://gateway-host.example.com/slack/events"
    }
  }
}
```

### Additional manifest settings

Surface different features that extend the above defaults.

The default manifest enables the Slack App Home **Home** tab and subscribes to `app_home_opened`. When a workspace member opens the Home tab, OpenClaw publishes a safe default Home view with `views.publish`; no conversation payload or private configuration is included. The **Messages** tab remains enabled for Slack DMs.

<AccordionGroup>
  <Accordion title="Optional native slash commands">
    Multiple [native slash commands](#commands-and-slash-behavior) can be used instead of a single configured command with nuance:

    * Use `/agentstatus` instead of `/status` because the `/status` command is reserved.
    * No more than 25 slash commands can be made available at once.

    Replace your existing `features.slash_commands` section with a subset of [available commands](/tools/slash-commands#command-list):

    <Tabs>
      <Tab title="Socket Mode (default)">
        ```json theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          "slash_commands": [
            {
              "command": "/new",
              "description": "Start a new session",
              "usage_hint": "[model]"
            },
            {
              "command": "/reset",
              "description": "Reset the current session"
            },
            {
              "command": "/compact",
              "description": "Compact the session context",
              "usage_hint": "[instructions]"
            },
            {
              "command": "/stop",
              "description": "Stop the current run"
            },
            {
              "command": "/session",
              "description": "Manage thread-binding expiry",
              "usage_hint": "idle <duration|off> or max-age <duration|off>"
            },
            {
              "command": "/think",
              "description": "Set the thinking level",
              "usage_hint": "<level>"
            },
            {
              "command": "/verbose",
              "description": "Toggle verbose output",
              "usage_hint": "on|off|full"
            },
            {
              "command": "/fast",
              "description": "Show or set fast mode",
              "usage_hint": "[status|on|off]"
            },
            {
              "command": "/reasoning",
              "description": "Toggle reasoning visibility",
              "usage_hint": "[on|off|stream]"
            },
            {
              "command": "/elevated",
              "description": "Toggle elevated mode",
              "usage_hint": "[on|off|ask|full]"
            },
            {
              "command": "/exec",
              "description": "Show or set exec defaults",
              "usage_hint": "host=<auto|sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>"
            },
            {
              "command": "/model",
              "description": "Show or set the model",
              "usage_hint": "[name|#|status]"
            },
            {
              "command": "/models",
              "description": "List providers/models",
              "usage_hint": "[provider] [page] [limit=<n>|size=<n>|all]"
            },
            {
              "command": "/help",
              "description": "Show the short help summary"
            },
            {
              "command": "/commands",
              "description": "Show the generated command catalog"
            },
            {
              "command": "/tools",
              "description": "Show what the current agent can use right now",
              "usage_hint": "[compact|verbose]"
            },
            {
              "command": "/agentstatus",
              "description": "Show runtime status, including provider usage/quota when available"
            },
            {
              "command": "/tasks",
              "description": "List active/recent background tasks for the current session"
            },
            {
              "command": "/context",
              "description": "Explain how context is assembled",
              "usage_hint": "[list|detail|json]"
            },
            {
              "command": "/whoami",
              "description": "Show your sender identity"
            },
            {
              "command": "/skill",
              "description": "Run a skill by name",
              "usage_hint": "<name> [input]"
            },
            {
              "command": "/btw",
              "description": "Ask a side question without changing session context",
              "usage_hint": "<question>"
            },
            {
              "command": "/side",
              "description": "Ask a side question without changing session context",
              "usage_hint": "<question>"
            },
            {
              "command": "/usage",
              "description": "Control the usage footer or show cost summary",
              "usage_hint": "off|tokens|full|cost"
            }
          ]
        }
        ```
      </Tab>

      <Tab title="HTTP Request URLs">
        Use the same `slash_commands` list as Socket Mode above, and add `"url": "https://gateway-host.example.com/slack/events"` to every entry. Example:

        ```json theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          "slash_commands": [
            {
              "command": "/new",
              "description": "Start a new session",
              "usage_hint": "[model]",
              "url": "https://gateway-host.example.com/slack/events"
            },
            {
              "command": "/help",
              "description": "Show the short help summary",
              "url": "https://gateway-host.example.com/slack/events"
            }
          ]
        }
        ```

        Repeat that `url` value on every command in the list.
      </Tab>
    </Tabs>
  </Accordion>

  <Accordion title="Optional authorship scopes (write operations)">
    Add the `chat:write.customize` bot scope if you want outgoing messages to use the active agent identity (custom username and icon) instead of the default Slack app identity.

    If you use an emoji icon, Slack expects `:emoji_name:` syntax.
  </Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    If you configure `channels.slack.userToken`, typical read scopes are:

    * `channels:history`, `groups:history`, `im:history`, `mpim:history`
    * `channels:read`, `groups:read`, `im:read`, `mpim:read`
    * `users:read`
    * `reactions:read`
    * `pins:read`
    * `emoji:read`
    * `search:read` (if you depend on Slack search reads)
  </Accordion>
</AccordionGroup>

## Token model

* `botToken` + `appToken` are required for Socket Mode.
* HTTP mode requires `botToken` + `signingSecret`.
* `botToken`, `appToken`, `signingSecret`, and `userToken` accept plaintext
  strings or SecretRef objects.
* Config tokens override env fallback.
* `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` env fallback applies only to the default account.
* `userToken` (`xoxp-...`) is config-only (no env fallback) and defaults to read-only behavior (`userTokenReadOnly: true`).

Status snapshot behavior:

* Slack account inspection tracks per-credential `*Source` and `*Status`
  fields (`botToken`, `appToken`, `signingSecret`, `userToken`).
* Status is `available`, `configured_unavailable`, or `missing`.
* `configured_unavailable` means the account is configured through SecretRef
  or another non-inline secret source, but the current command/runtime path
  could not resolve the actual value.
* In HTTP mode, `signingSecretStatus` is included; in Socket Mode, the
  required pair is `botTokenStatus` + `appTokenStatus`.

<Tip>
  For actions/directory reads, user token can be preferred when configured. For writes, bot token remains preferred; user-token writes are only allowed when `userTokenReadOnly: false` and bot token is unavailable.
</Tip>

## Actions and gates

Slack actions are controlled by `channels.slack.actions.*`.

Available action groups in current Slack tooling:

| Group      | Default |
| ---------- | ------- |
| messages   | enabled |
| reactions  | enabled |
| pins       | enabled |
| memberInfo | enabled |
| emojiList  | enabled |

Current Slack message actions include `send`, `upload-file`, `download-file`, `read`, `edit`, `delete`, `pin`, `unpin`, `list-pins`, `member-info`, and `emoji-list`. `download-file` accepts Slack file IDs shown in inbound file placeholders and returns image previews for images or local file metadata for other file types.

## Access control and routing

<Tabs>
  <Tab title="DM policy">
    `channels.slack.dmPolicy` controls DM access. `channels.slack.allowFrom` is the canonical DM allowlist.

    * `pairing` (default)
    * `allowlist`
    * `open` (requires `channels.slack.allowFrom` to include `"*"`)
    * `disabled`

    DM flags:

    * `dm.enabled` (default true)
    * `channels.slack.allowFrom`
    * `dm.allowFrom` (legacy)
    * `dm.groupEnabled` (group DMs default false)
    * `dm.groupChannels` (optional MPIM allowlist)

    Multi-account precedence:

    * `channels.slack.accounts.default.allowFrom` applies only to the `default` account.
    * Named accounts inherit `channels.slack.allowFrom` when their own `allowFrom` is unset.
    * Named accounts do not inherit `channels.slack.accounts.default.allowFrom`.

    Legacy `channels.slack.dm.policy` and `channels.slack.dm.allowFrom` still read for compatibility. `openclaw doctor --fix` migrates them to `dmPolicy` and `allowFrom` when it can do so without changing access.

    Pairing in DMs uses `openclaw pairing approve slack <code>`.
  </Tab>

  <Tab title="Channel policy">
    `channels.slack.groupPolicy` controls channel handling:

    * `open`
    * `allowlist`
    * `disabled`

    Channel allowlist lives under `channels.slack.channels` and **must use stable Slack channel IDs** (for example `C12345678`) as config keys.

    Runtime note: if `channels.slack` is completely missing (env-only setup), runtime falls back to `groupPolicy="allowlist"` and logs a warning (even if `channels.defaults.groupPolicy` is set).

    Name/ID resolution:

    * channel allowlist entries and DM allowlist entries are resolved at startup when token access allows
    * unresolved channel-name entries are kept as configured but ignored for routing by default
    * inbound authorization and channel routing are ID-first by default; direct username/slug matching requires `channels.slack.dangerouslyAllowNameMatching: true`

    <Warning>
      Name-based keys (`#channel-name` or `channel-name`) do **not** match under `groupPolicy: "allowlist"`. The channel lookup is ID-first by default, so a name-based key will never route successfully and all messages in that channel will be silently blocked. This differs from `groupPolicy: "open"`, where the channel key is not required for routing and a name-based key appears to work.

      Always use the Slack channel ID as the key. To find it: right-click the channel in Slack → **Copy link** — the ID (`C...`) appears at the end of the URL.

      Correct:

      ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
      {
        channels: {
          slack: {
            groupPolicy: "allowlist",
            channels: {
              C12345678: { allow: true, requireMention: true },
            },
          },
        },
      }
      ```

      Incorrect (silently blocked under `groupPolicy: "allowlist"`):

      ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
      {
        channels: {
          slack: {
            groupPolicy: "allowlist",
            channels: {
              "#eng-my-channel": { allow: true, requireMention: true },
            },
          },
        },
      }
      ```
    </Warning>
  </Tab>

  <Tab title="Mentions and channel users">
    Channel messages are mention-gated by default.

    Mention sources:

    * explicit app mention (`<@botId>`)
    * Slack user-group mention (`<!subteam^S...>`) when the bot user is a member of that user group; requires `usergroups:read`
    * mention regex patterns (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    * implicit reply-to-bot thread behavior (disabled when `thread.requireExplicitMention` is `true`)

    Per-channel controls (`channels.slack.channels.<id>`; names only via startup resolution or `dangerouslyAllowNameMatching`):

    * `requireMention`
    * `users` (allowlist)
    * `allowBots`
    * `skills`
    * `systemPrompt`
    * `tools`, `toolsBySender`
    * `toolsBySender` key format: `id:`, `e164:`, `username:`, `name:`, or `"*"` wildcard
      (legacy unprefixed keys still map to `id:` only)

    `allowBots` is conservative for channels and private channels: bot-authored room messages are accepted only when the sending bot is explicitly listed in that room's `users` allowlist, or when at least one explicit Slack owner ID from `channels.slack.allowFrom` is currently a room member. Wildcards and display-name owner entries do not satisfy owner presence. Owner presence uses Slack `conversations.members`; make sure the app has the matching read scope for the room type (`channels:read` for public channels, `groups:read` for private channels). If the member lookup fails, OpenClaw drops the bot-authored room message.
  </Tab>
</Tabs>

## Threading, sessions, and reply tags

* DMs route as `direct`; channels as `channel`; MPIMs as `group`.
* Slack route bindings accept raw peer IDs plus Slack target forms such as `channel:C12345678`, `user:U12345678`, and `<@U12345678>`.
* With default `session.dmScope=main`, Slack DMs collapse to agent main session.
* Channel sessions: `agent:<agentId>:slack:channel:<channelId>`.
* Thread replies can create thread session suffixes (`:thread:<threadTs>`) when applicable.
* `channels.slack.thread.historyScope` default is `thread`; `thread.inheritParent` default is `false`.
* `channels.slack.thread.initialHistoryLimit` controls how many existing thread messages are fetched when a new thread session starts (default `20`; set `0` to disable).
* `channels.slack.thread.requireExplicitMention` (default `false`): when `true`, suppress implicit thread mentions so the bot only responds to explicit `@bot` mentions inside threads, even when the bot already participated in the thread. Without this, replies in a bot-participated thread bypass `requireMention` gating.

Reply threading controls:

* `channels.slack.replyToMode`: `off|first|all|batched` (default `off`)
* `channels.slack.replyToModeByChatType`: per `direct|group|channel`
* legacy fallback for direct chats: `channels.slack.dm.replyToMode`

Manual reply tags are supported:

* `[[reply_to_current]]`
* `[[reply_to:<id>]]`

<Note>
  `replyToMode="off"` disables **all** reply threading in Slack, including explicit `[[reply_to_*]]` tags. This differs from Telegram, where explicit tags are still honored in `"off"` mode. Slack threads hide messages from the channel while Telegram replies stay visible inline.
</Note>

## Ack reactions

`ackReaction` sends an acknowledgement emoji while OpenClaw is processing an inbound message.

Resolution order:

* `channels.slack.accounts.<accountId>.ackReaction`
* `channels.slack.ackReaction`
* `messages.ackReaction`
* agent identity emoji fallback (`agents.list[].identity.emoji`, else "👀")

Notes:

* Slack expects shortcodes (for example `"eyes"`).
* Use `""` to disable the reaction for the Slack account or globally.

## Text streaming

`channels.slack.streaming` controls live preview behavior:

* `off`: disable live preview streaming.
* `partial` (default): replace preview text with the latest partial output.
* `block`: append chunked preview updates.
* `progress`: show progress status text while generating, then send final text.
* `streaming.preview.toolProgress`: when draft preview is active, route tool/progress updates into the same edited preview message (default: `true`). Set `false` to keep separate tool/progress messages.
* `streaming.preview.commandText` / `streaming.progress.commandText`: set to `status` to keep compact tool-progress lines while hiding raw command/exec text (default: `raw`).

Hide raw command/exec text while keeping compact progress lines:

```json theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  "channels": {
    "slack": {
      "streaming": {
        "mode": "progress",
        "progress": {
          "toolProgress": true,
          "commandText": "status"
        }
      }
    }
  }
}
```

`channels.slack.streaming.nativeTransport` controls Slack native text streaming when `channels.slack.streaming.mode` is `partial` (default: `true`).

* A reply thread must be available for native text streaming and Slack assistant thread status to appear. Thread selection still follows `replyToMode`.
* Channel, group-chat, and top-level DM roots can still use the normal draft preview when native streaming is unavailable or no reply thread exists.
* Top-level Slack DMs stay off-thread by default, so they do not show Slack's thread-style native stream/status preview; OpenClaw posts and edits a draft preview in the DM instead.
* Media and non-text payloads fall back to normal delivery.
* Media/error finals cancel pending preview edits; eligible text/block finals flush only when they can edit the preview in place.
* If streaming fails mid-reply, OpenClaw falls back to normal delivery for remaining payloads.

Use draft preview instead of Slack native text streaming:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    slack: {
      streaming: {
        mode: "partial",
        nativeTransport: false,
      },
    },
  },
}
```

Legacy keys:

* `channels.slack.streamMode` (`replace | status_final | append`) is a legacy runtime alias for `channels.slack.streaming.mode`.
* boolean `channels.slack.streaming` is a legacy runtime alias for `channels.slack.streaming.mode` and `channels.slack.streaming.nativeTransport`.
* legacy `channels.slack.nativeStreaming` is a runtime alias for `channels.slack.streaming.nativeTransport`.
* Run `openclaw doctor --fix` to rewrite persisted Slack streaming config to the canonical keys.

## Typing reaction fallback

`typingReaction` adds a temporary reaction to the inbound Slack message while OpenClaw is processing a reply, then removes it when the run finishes. This is most useful outside of thread replies, which use a default "is typing..." status indicator.

Resolution order:

* `channels.slack.accounts.<accountId>.typingReaction`
* `channels.slack.typingReaction`

Notes:

* Slack expects shortcodes (for example `"hourglass_flowing_sand"`).
* The reaction is best-effort and cleanup is attempted automatically after the reply or failure path completes.

## Media, chunking, and delivery

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack file attachments are downloaded from Slack-hosted private URLs (token-authenticated request flow) and written to the media store when fetch succeeds and size limits permit. File placeholders include the Slack `fileId` so agents can fetch the original file with `download-file`.

    Downloads use bounded idle and total timeouts. If Slack file retrieval stalls or fails, OpenClaw keeps processing the message and falls back to the file placeholder.

    Runtime inbound size cap defaults to `20MB` unless overridden by `channels.slack.mediaMaxMb`.
  </Accordion>

  <Accordion title="Outbound text and files">
    * text chunks use `channels.slack.textChunkLimit` (default 4000)
    * `channels.slack.chunkMode="newline"` enables paragraph-first splitting
    * file sends use Slack upload APIs and can include thread replies (`thread_ts`)
    * outbound media cap follows `channels.slack.mediaMaxMb` when configured; otherwise channel sends use MIME-kind defaults from media pipeline
  </Accordion>

  <Accordion title="Delivery targets">
    Preferred explicit targets:

    * `user:<id>` for DMs
    * `channel:<id>` for channels

    Text/block-only Slack DMs can post directly to user IDs; file uploads and threaded sends open the DM via Slack conversation APIs first because those paths require a concrete conversation ID.
  </Accordion>
</AccordionGroup>

## Commands and slash behavior

Slash commands appear in Slack as either a single configured command or multiple native commands. Configure `channels.slack.slashCommand` to change command defaults:

* `enabled: false`
* `name: "openclaw"`
* `sessionPrefix: "slack:slash"`
* `ephemeral: true`

```txt theme={"theme":{"light":"min-light","dark":"min-dark"}}
/openclaw /help
```

Native commands require [additional manifest settings](#additional-manifest-settings) in your Slack app and are enabled with `channels.slack.commands.native: true` or `commands.native: true` in global configurations instead.

* Native command auto-mode is **off** for Slack so `commands.native: "auto"` does not enable Slack native commands.

```txt theme={"theme":{"light":"min-light","dark":"min-dark"}}
/help
```

Native argument menus use an adaptive rendering strategy that shows a confirmation modal before dispatching a selected option value:

* up to 5 options: button blocks
* 6-100 options: static select menu
* more than 100 options: external select with async option filtering when interactivity options handlers are available
* exceeded Slack limits: encoded option values fall back to buttons

```txt theme={"theme":{"light":"min-light","dark":"min-dark"}}
/think
```

Slash sessions use isolated keys like `agent:<agentId>:slack:slash:<userId>` and still route command executions to the target conversation session using `CommandTargetSessionKey`.

## Interactive replies

Slack can render agent-authored interactive reply controls, but this feature is disabled by default.

Enable it globally:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    slack: {
      capabilities: {
        interactiveReplies: true,
      },
    },
  },
}
```

Or enable it for one Slack account only:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    slack: {
      accounts: {
        ops: {
          capabilities: {
            interactiveReplies: true,
          },
        },
      },
    },
  },
}
```

When enabled, agents can emit Slack-only reply directives:

* `[[slack_buttons: Approve:approve, Reject:reject]]`
* `[[slack_select: Choose a target | Canary:canary, Production:production]]`

These directives compile into Slack Block Kit and route clicks or selections back through the existing Slack interaction event path.

Notes:

* This is Slack-specific UI. Other channels do not translate Slack Block Kit directives into their own button systems.
* The interactive callback values are OpenClaw-generated opaque tokens, not raw agent-authored values.
* If generated interactive blocks would exceed Slack Block Kit limits, OpenClaw falls back to the original text reply instead of sending an invalid blocks payload.

## Exec approvals in Slack

Slack can act as a native approval client with interactive buttons and interactions, instead of falling back to the Web UI or terminal.

* Exec approvals use `channels.slack.execApprovals.*` for native DM/channel routing.
* Plugin approvals can still resolve through the same Slack-native button surface when the request already lands in Slack and the approval id kind is `plugin:`.
* Approver authorization is still enforced: only users identified as approvers can approve or deny requests through Slack.

This uses the same shared approval button surface as other channels. When `interactivity` is enabled in your Slack app settings, approval prompts render as Block Kit buttons directly in the conversation.
When those buttons are present, they are the primary approval UX; OpenClaw
should only include a manual `/approve` command when the tool result says chat
approvals are unavailable or manual approval is the only path.

Config path:

* `channels.slack.execApprovals.enabled`
* `channels.slack.execApprovals.approvers` (optional; falls back to `commands.ownerAllowFrom` when possible)
* `channels.slack.execApprovals.target` (`dm` | `channel` | `both`, default: `dm`)
* `agentFilter`, `sessionFilter`

Slack auto-enables native exec approvals when `enabled` is unset or `"auto"` and at least one
approver resolves. Set `enabled: false` to disable Slack as a native approval client explicitly.
Set `enabled: true` to force native approvals on when approvers resolve.

Default behavior with no explicit Slack exec approval config:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  commands: {
    ownerAllowFrom: ["slack:U12345678"],
  },
}
```

Explicit Slack-native config is only needed when you want to override approvers, add filters, or
opt into origin-chat delivery:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    slack: {
      execApprovals: {
        enabled: true,
        approvers: ["U12345678"],
        target: "both",
      },
    },
  },
}
```

Shared `approvals.exec` forwarding is separate. Use it only when exec approval prompts must also
route to other chats or explicit out-of-band targets. Shared `approvals.plugin` forwarding is also
separate; Slack-native buttons can still resolve plugin approvals when those requests already land
in Slack.

Same-chat `/approve` also works in Slack channels and DMs that already support commands. See [Exec approvals](/tools/exec-approvals) for the full approval forwarding model.

## Events and operational behavior

* Message edits/deletes are mapped into system events.
* Thread broadcasts ("Also send to channel" thread replies) are processed as normal user messages.
* Reaction add/remove events are mapped into system events.
* Member join/leave, channel created/renamed, and pin add/remove events are mapped into system events.
* `channel_id_changed` can migrate channel config keys when `configWrites` is enabled.
* Channel topic/purpose metadata is treated as untrusted context and can be injected into routing context.
* Thread starter and initial thread-history context seeding are filtered by configured sender allowlists when applicable.
* Block actions and modal interactions emit structured `Slack interaction: ...` system events with rich payload fields:
  * block actions: selected values, labels, picker values, and `workflow_*` metadata
  * modal `view_submission` and `view_closed` events with routed channel metadata and form inputs

## Configuration reference

Primary reference: [Configuration reference - Slack](/gateway/config-channels#slack).

<Accordion title="High-signal Slack fields">
  * mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  * DM access: `dm.enabled`, `dmPolicy`, `allowFrom` (legacy: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  * compatibility toggle: `dangerouslyAllowNameMatching` (break-glass; keep off unless needed)
  * channel access: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  * threading/history: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  * delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `streaming`, `streaming.nativeTransport`, `streaming.preview.toolProgress`
  * ops/features: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`
</Accordion>

## Troubleshooting

<AccordionGroup>
  <Accordion title="No replies in channels">
    Check, in order:

    * `groupPolicy`
    * channel allowlist (`channels.slack.channels`) — **keys must be channel IDs** (`C12345678`), not names (`#channel-name`). Name-based keys silently fail under `groupPolicy: "allowlist"` because channel routing is ID-first by default. To find an ID: right-click the channel in Slack → **Copy link** — the `C...` value at the end of the URL is the channel ID.
    * `requireMention`
    * per-channel `users` allowlist

    Useful commands:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw channels status --probe
    openclaw logs --follow
    openclaw doctor
    ```
  </Accordion>

  <Accordion title="DM messages ignored">
    Check:

    * `channels.slack.dm.enabled`
    * `channels.slack.dmPolicy` (or legacy `channels.slack.dm.policy`)
    * pairing approvals / allowlist entries
    * Slack Assistant DM events: verbose logs mentioning `drop message_changed`
      usually mean Slack sent an edited Assistant-thread event without a
      recoverable human sender in message metadata

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw pairing list slack
    ```
  </Accordion>

  <Accordion title="Socket mode not connecting">
    Validate bot + app tokens and Socket Mode enablement in Slack app settings.

    If `openclaw channels status --probe --json` shows `botTokenStatus` or
    `appTokenStatus: "configured_unavailable"`, the Slack account is
    configured but the current runtime could not resolve the SecretRef-backed
    value.
  </Accordion>

  <Accordion title="HTTP mode not receiving events">
    Validate:

    * signing secret
    * webhook path
    * Slack Request URLs (Events + Interactivity + Slash Commands)
    * unique `webhookPath` per HTTP account

    If `signingSecretStatus: "configured_unavailable"` appears in account
    snapshots, the HTTP account is configured but the current runtime could not
    resolve the SecretRef-backed signing secret.
  </Accordion>

  <Accordion title="Native/slash commands not firing">
    Verify whether you intended:

    * native command mode (`channels.slack.commands.native: true`) with matching slash commands registered in Slack
    * or single slash command mode (`channels.slack.slashCommand.enabled: true`)

    Also check `commands.useAccessGroups` and channel/user allowlists.
  </Accordion>
</AccordionGroup>

## Attachment vision reference

Slack can attach downloaded media to the agent turn when Slack file downloads succeed and size limits permit. Image files can be passed through the media understanding path or directly to a vision-capable reply model; other files are retained as downloadable file context rather than treated as image input.

### Supported media types

| Media type                     | Source               | Current behavior                                                                  | Notes                                                                     |
| ------------------------------ | -------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| JPEG / PNG / GIF / WebP images | Slack file URL       | Downloaded and attached to the turn for vision-capable handling                   | Per-file cap: `channels.slack.mediaMaxMb` (default 20 MB)                 |
| PDF files                      | Slack file URL       | Downloaded and exposed as file context for tools such as `download-file` or `pdf` | Slack inbound does not convert PDFs into image-vision input automatically |
| Other files                    | Slack file URL       | Downloaded when possible and exposed as file context                              | Binary files are not treated as image input                               |
| Thread replies                 | Thread starter files | Root-message files can be hydrated as context when the reply has no direct media  | File-only starters use an attachment placeholder                          |
| Multi-image messages           | Multiple Slack files | Each file is evaluated independently                                              | Slack processing is capped at eight files per message                     |

### Inbound pipeline

When a Slack message with file attachments arrives:

1. OpenClaw downloads the file from Slack's private URL using the bot token (`xoxb-...`).
2. The file is written to the media store on success.
3. Downloaded media paths and content types are added to the inbound context.
4. Image-capable model/tool paths can use image attachments from that context.
5. Non-image files remain available as file metadata or media references for tools that can handle them.

### Thread-root attachment inheritance

When a message arrives in a thread (has a `thread_ts` parent):

* If the reply itself has no direct media and the included root message has files, Slack can hydrate the root files as thread-starter context.
* Direct reply attachments take precedence over root-message attachments.
* A root message that has only files and no text is represented with an attachment placeholder so the fallback can still include its files.

### Multi-attachment handling

When a single Slack message contains multiple file attachments:

* Each attachment is processed independently through the media pipeline.
* Downloaded media references are aggregated into the message context.
* Processing order follows Slack's file order in the event payload.
* A failure in one attachment's download does not block others.

### Size, download, and model limits

* **Size cap**: Default 20 MB per file. Configurable via `channels.slack.mediaMaxMb`.
* **Download failures**: Files that Slack cannot serve, expired URLs, inaccessible files, oversize files, and Slack auth/login HTML responses are skipped instead of being reported as unsupported formats.
* **Vision model**: Image analysis uses the active reply model when it supports vision, or the image model configured at `agents.defaults.imageModel`.

### Known limits

| Scenario                               | Current behavior                                                             | Workaround                                                                 |
| -------------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Expired Slack file URL                 | File skipped; no error shown                                                 | Re-upload the file in Slack                                                |
| Vision model not configured            | Image attachments are stored as media references, but not analyzed as images | Configure `agents.defaults.imageModel` or use a vision-capable reply model |
| Very large images (> 20 MB by default) | Skipped per size cap                                                         | Increase `channels.slack.mediaMaxMb` if Slack allows                       |
| Forwarded/shared attachments           | Text and Slack-hosted image/file media are best-effort                       | Re-share directly in the OpenClaw thread                                   |
| PDF attachments                        | Stored as file/media context, not automatically routed through image vision  | Use `download-file` for file metadata or the `pdf` tool for PDF analysis   |

### Related documentation

* [Media understanding pipeline](/nodes/media-understanding)
* [PDF tool](/tools/pdf)
* Epic: [#51349](https://github.com/openclaw/openclaw/issues/51349) — Slack attachment vision enablement
* Regression tests: [#51353](https://github.com/openclaw/openclaw/issues/51353)
* Live verification: [#51354](https://github.com/openclaw/openclaw/issues/51354)

## Related

<CardGroup cols={2}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Pair a Slack user to the gateway.
  </Card>

  <Card title="Groups" icon="users" href="/channels/groups">
    Channel and group DM behavior.
  </Card>

  <Card title="Channel routing" icon="route" href="/channels/channel-routing">
    Route inbound messages to agents.
  </Card>

  <Card title="Security" icon="shield" href="/gateway/security">
    Threat model and hardening.
  </Card>

  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Config layout and precedence.
  </Card>

  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Command catalog and behavior.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Synology Chat

Status: bundled plugin direct-message channel using Synology Chat webhooks.
The plugin accepts inbound messages from Synology Chat outgoing webhooks and sends replies
through a Synology Chat incoming webhook.

## Bundled plugin

Synology Chat ships as a bundled plugin in current OpenClaw releases, so normal
packaged builds do not need a separate install.

If you are on an older build or a custom install that excludes Synology Chat,
install it manually:

Install from a local checkout:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw plugins install ./path/to/local/synology-chat-plugin
```

Details: [Plugins](/tools/plugin)

## Quick setup

1. Ensure the Synology Chat plugin is available.
   * Current packaged OpenClaw releases already bundle it.
   * Older/custom installs can add it manually from a source checkout with the command above.
   * `openclaw onboard` now shows Synology Chat in the same channel setup list as `openclaw channels add`.
   * Non-interactive setup: `openclaw channels add --channel synology-chat --token <token> --url <incoming-webhook-url>`
2. In Synology Chat integrations:
   * Create an incoming webhook and copy its URL.
   * Create an outgoing webhook with your secret token.
3. Point the outgoing webhook URL to your OpenClaw gateway:
   * `https://gateway-host/webhook/synology` by default.
   * Or your custom `channels.synology-chat.webhookPath`.
4. Finish setup in OpenClaw.
   * Guided: `openclaw onboard`
   * Direct: `openclaw channels add --channel synology-chat --token <token> --url <incoming-webhook-url>`
5. Restart gateway and send a DM to the Synology Chat bot.

Webhook auth details:

* OpenClaw accepts the outgoing webhook token from `body.token`, then
  `?token=...`, then headers.
* Accepted header forms:
  * `x-synology-token`
  * `x-webhook-token`
  * `x-openclaw-token`
  * `Authorization: Bearer <token>`
* Empty or missing tokens fail closed.

Minimal config:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```

## Environment variables

For the default account, you can use env vars:

* `SYNOLOGY_CHAT_TOKEN`
* `SYNOLOGY_CHAT_INCOMING_URL`
* `SYNOLOGY_NAS_HOST`
* `SYNOLOGY_ALLOWED_USER_IDS` (comma-separated)
* `SYNOLOGY_RATE_LIMIT`
* `OPENCLAW_BOT_NAME`

Config values override env vars.

`SYNOLOGY_CHAT_INCOMING_URL` cannot be set from a workspace `.env`; see [Workspace `.env` files](/gateway/security).

## DM policy and access control

* `dmPolicy: "allowlist"` is the recommended default.
* `allowedUserIds` accepts a list (or comma-separated string) of Synology user IDs.
* In `allowlist` mode, an empty `allowedUserIds` list is treated as misconfiguration and the webhook route will not start (use `dmPolicy: "open"` with `allowedUserIds: ["*"]` for allow-all).
* `dmPolicy: "open"` allows public DMs only when `allowedUserIds` includes `"*"`; with restrictive entries, only matching users can chat.
* `dmPolicy: "disabled"` blocks DMs.
* Reply recipient binding stays on stable numeric `user_id` by default. `channels.synology-chat.dangerouslyAllowNameMatching: true` is break-glass compatibility mode that re-enables mutable username/nickname lookup for reply delivery.
* Pairing approvals work with:
  * `openclaw pairing list synology-chat`
  * `openclaw pairing approve synology-chat <CODE>`

## Outbound delivery

Use numeric Synology Chat user IDs as targets.

Examples:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
openclaw message send --channel synology-chat --target synology:123456 --text "Short prefix"
```

Media sends are supported by URL-based file delivery.
Outbound file URLs must use `http` or `https`, and private or otherwise blocked network targets are rejected before OpenClaw forwards the URL to the NAS webhook.

## Multi-account

Multiple Synology Chat accounts are supported under `channels.synology-chat.accounts`.
Each account can override token, incoming URL, webhook path, DM policy, and limits.
Direct-message sessions are isolated per account and user, so the same numeric `user_id`
on two different Synology accounts does not share transcript state.
Give each enabled account a distinct `webhookPath`. OpenClaw now rejects duplicate exact paths
and refuses to start named accounts that only inherit a shared webhook path in multi-account setups.
If you intentionally need legacy inheritance for a named account, set
`dangerouslyAllowInheritedWebhookPath: true` on that account or at `channels.synology-chat`,
but duplicate exact paths are still rejected fail-closed. Prefer explicit per-account paths.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```

## Security notes

* Keep `token` secret and rotate it if leaked.
* Keep `allowInsecureSsl: false` unless you explicitly trust a self-signed local NAS cert.
* Inbound webhook requests are token-verified and rate-limited per sender.
* Invalid token checks use constant-time secret comparison and fail closed.
* Prefer `dmPolicy: "allowlist"` for production.
* Keep `dangerouslyAllowNameMatching` off unless you explicitly need legacy username-based reply delivery.
* Keep `dangerouslyAllowInheritedWebhookPath` off unless you explicitly accept shared-path routing risk in a multi-account setup.

## Troubleshooting

* `Missing required fields (token, user_id, text)`:
  * the outgoing webhook payload is missing one of the required fields
  * if Synology sends the token in headers, make sure the gateway/proxy preserves those headers
* `Invalid token`:
  * the outgoing webhook secret does not match `channels.synology-chat.token`
  * the request is hitting the wrong account/webhook path
  * a reverse proxy stripped the token header before the request reached OpenClaw
* `Rate limit exceeded`:
  * too many invalid token attempts from the same source can temporarily lock that source out
  * authenticated senders also have a separate per-user message rate limit
* `Allowlist is empty. Configure allowedUserIds or use dmPolicy=open with allowedUserIds=["*"].`:
  * `dmPolicy="allowlist"` is enabled but no users are configured
* `User not authorized`:
  * the sender's numeric `user_id` is not in `allowedUserIds`

## Related

* [Channels Overview](/channels) — all supported channels
* [Pairing](/channels/pairing) — DM authentication and pairing flow
* [Groups](/channels/groups) — group chat behavior and mention gating
* [Channel Routing](/channels/channel-routing) — session routing for messages
* [Security](/gateway/security) — access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Tlon

Tlon is a decentralized messenger built on Urbit. OpenClaw connects to your Urbit ship and can
respond to DMs and group chat messages. Group replies require an @ mention by default and can
be further restricted via allowlists.

Status: bundled plugin. DMs, group mentions, thread replies, rich text formatting, and
image uploads are supported. Reactions and polls are not yet supported.

## Bundled plugin

Tlon ships as a bundled plugin in current OpenClaw releases, so normal packaged
builds do not need a separate install.

If you are on an older build or a custom install that excludes Tlon, install a
current npm package:

Install via CLI (npm registry):

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw plugins install @openclaw/tlon
```

Use the bare package to follow the current official release tag. Pin an exact
version only when you need a reproducible install.

Local checkout (when running from a git repo):

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw plugins install ./path/to/local/tlon-plugin
```

Details: [Plugins](/tools/plugin)

## Setup

1. Ensure the Tlon plugin is available.
   * Current packaged OpenClaw releases already bundle it.
   * Older/custom installs can add it manually with the commands above.
2. Gather your ship URL and login code.
3. Configure `channels.tlon`.
4. Restart the gateway.
5. DM the bot or mention it in a group channel.

Minimal config (single account):

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
      ownerShip: "~your-main-ship", // recommended: your ship, always allowed
    },
  },
}
```

## Private/LAN ships

By default, OpenClaw blocks private/internal hostnames and IP ranges for SSRF protection.
If your ship is running on a private network (localhost, LAN IP, or internal hostname),
you must explicitly opt in:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    tlon: {
      url: "http://localhost:8080",
      allowPrivateNetwork: true,
    },
  },
}
```

This applies to URLs like:

* `http://localhost:8080`
* `http://192.168.x.x:8080`
* `http://my-ship.local:8080`

⚠️ Only enable this if you trust your local network. This setting disables SSRF protections
for requests to your ship URL.

## Group channels

Auto-discovery is enabled by default. You can also pin channels manually:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

Disable auto-discovery:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## Access control

DM allowlist (empty = no DMs allowed, use `ownerShip` for approval flow):

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

Group authorization (restricted by default):

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## Owner and approval system

Set an owner ship to receive approval requests when unauthorized users try to interact:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    tlon: {
      ownerShip: "~your-main-ship",
    },
  },
}
```

The owner ship is **automatically authorized everywhere** — DM invites are auto-accepted and
channel messages are always allowed. You don't need to add the owner to `dmAllowlist` or
`defaultAuthorizedShips`.

When set, the owner receives DM notifications for:

* DM requests from ships not in the allowlist
* Mentions in channels without authorization
* Group invite requests

## Auto-accept settings

Auto-accept DM invites (for ships in dmAllowlist):

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    tlon: {
      autoAcceptDmInvites: true,
    },
  },
}
```

Auto-accept group invites from trusted ships:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    tlon: {
      autoAcceptGroupInvites: true,
      groupInviteAllowlist: ["~zod"],
    },
  },
}
```

`autoAcceptGroupInvites` fails closed when `groupInviteAllowlist` is empty. Set the
allowlist to the ships whose group invites should be accepted automatically.

## Delivery targets (CLI/cron)

Use these with `openclaw message send` or cron delivery:

* DM: `~sampel-palnet` or `dm/~sampel-palnet`
* Group: `chat/~host-ship/channel` or `group:~host-ship/channel`

## Bundled skill

The Tlon plugin includes a bundled skill ([`@tloncorp/tlon-skill`](https://github.com/tloncorp/tlon-skill))
that provides CLI access to Tlon operations:

* **Contacts**: get/update profiles, list contacts
* **Channels**: list, create, post messages, fetch history
* **Groups**: list, create, manage members
* **DMs**: send messages, react to messages
* **Reactions**: add/remove emoji reactions to posts and DMs
* **Settings**: manage plugin permissions via slash commands

The skill is automatically available when the plugin is installed.

## Capabilities

| Feature         | Status                                 |
| --------------- | -------------------------------------- |
| Direct messages | ✅ Supported                            |
| Groups/channels | ✅ Supported (mention-gated by default) |
| Threads         | ✅ Supported (auto-replies in thread)   |
| Rich text       | ✅ Markdown converted to Tlon format    |
| Images          | ✅ Uploaded to Tlon storage             |
| Reactions       | ✅ Via [bundled skill](#bundled-skill)  |
| Polls           | ❌ Not yet supported                    |
| Native commands | ✅ Supported (owner-only by default)    |

## Troubleshooting

Run this ladder first:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
```

Common failures:

* **DMs ignored**: sender not in `dmAllowlist` and no `ownerShip` configured for approval flow.
* **Group messages ignored**: channel not discovered or sender not authorized.
* **Connection errors**: check ship URL is reachable; enable `allowPrivateNetwork` for local ships.
* **Auth errors**: verify login code is current (codes rotate).

## Configuration reference

Full configuration: [Configuration](/gateway/configuration)

Provider options:

* `channels.tlon.enabled`: enable/disable channel startup.
* `channels.tlon.ship`: bot's Urbit ship name (e.g. `~sampel-palnet`).
* `channels.tlon.url`: ship URL (e.g. `https://sampel-palnet.tlon.network`).
* `channels.tlon.code`: ship login code.
* `channels.tlon.allowPrivateNetwork`: allow localhost/LAN URLs (SSRF bypass).
* `channels.tlon.ownerShip`: owner ship for approval system (always authorized).
* `channels.tlon.dmAllowlist`: ships allowed to DM (empty = none).
* `channels.tlon.autoAcceptDmInvites`: auto-accept DMs from allowlisted ships.
* `channels.tlon.autoAcceptGroupInvites`: auto-accept group invites from allowlisted ships.
* `channels.tlon.groupInviteAllowlist`: ships whose group invites may be auto-accepted.
* `channels.tlon.autoDiscoverChannels`: auto-discover group channels (default: true).
* `channels.tlon.groupChannels`: manually pinned channel nests.
* `channels.tlon.defaultAuthorizedShips`: ships authorized for all channels.
* `channels.tlon.authorization.channelRules`: per-channel auth rules.
* `channels.tlon.showModelSignature`: append model name to messages.

## Notes

* Group replies require a mention (e.g. `~your-bot-ship`) to respond.
* Thread replies: if the inbound message is in a thread, OpenClaw replies in-thread.
* Rich text: Markdown formatting (bold, italic, code, headers, lists) is converted to Tlon's native format.
* Images: URLs are uploaded to Tlon storage and embedded as image blocks.

## Related

* [Channels Overview](/channels) — all supported channels
* [Pairing](/channels/pairing) — DM authentication and pairing flow
* [Groups](/channels/groups) — group chat behavior and mention gating
* [Channel Routing](/channels/channel-routing) — session routing for messages
* [Security](/gateway/security) — access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Channel troubleshooting

Use this page when a channel connects but behavior is wrong.

## Command ladder

Run these in order first:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Healthy baseline:

* `Runtime: running`
* `Connectivity probe: ok`
* `Capability: read-only`, `write-capable`, or `admin-capable`
* Channel probe shows transport connected and, where supported, `works` or `audit ok`

## WhatsApp

### WhatsApp failure signatures

| Symptom                             | Fastest check                                       | Fix                                                                                                                              |
| ----------------------------------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Connected but no DM replies         | `openclaw pairing list whatsapp`                    | Approve sender or switch DM policy/allowlist.                                                                                    |
| Group messages ignored              | Check `requireMention` + mention patterns in config | Mention the bot or relax mention policy for that group.                                                                          |
| QR login times out with 408         | Check gateway `HTTPS_PROXY` / `HTTP_PROXY` env      | Set a reachable proxy; use `NO_PROXY` only for bypasses.                                                                         |
| Random disconnect/relogin loops     | `openclaw channels status --probe` + logs           | Recent reconnects are flagged even when currently connected; watch logs, restart the gateway, then relink if flapping continues. |
| Replies arrive seconds/minutes late | `openclaw doctor --fix`                             | Doctor stops verified stale local TUI clients when they are degrading the Gateway event loop.                                    |

Full troubleshooting: [WhatsApp troubleshooting](/channels/whatsapp#troubleshooting)

## Telegram

### Telegram failure signatures

| Symptom                              | Fastest check                                    | Fix                                                                                                                        |
| ------------------------------------ | ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| `/start` but no usable reply flow    | `openclaw pairing list telegram`                 | Approve pairing or change DM policy.                                                                                       |
| Bot online but group stays silent    | Verify mention requirement and bot privacy mode  | Disable privacy mode for group visibility or mention bot.                                                                  |
| Send failures with network errors    | Inspect logs for Telegram API call failures      | Fix DNS/IPv6/proxy routing to `api.telegram.org`.                                                                          |
| Startup reports `getMe returned 401` | Check configured token source                    | Re-copy or regenerate the BotFather token and update `botToken`, `tokenFile`, or default-account `TELEGRAM_BOT_TOKEN`.     |
| Polling stalls or reconnects slowly  | `openclaw logs --follow` for polling diagnostics | Upgrade; if restarts are false positives, tune `pollingStallThresholdMs`. Persistent stalls still point to proxy/DNS/IPv6. |
| `setMyCommands` rejected at startup  | Inspect logs for `BOT_COMMANDS_TOO_MUCH`         | Reduce plugin/skill/custom Telegram commands or disable native menus.                                                      |
| Upgraded and allowlist blocks you    | `openclaw security audit` and config allowlists  | Run `openclaw doctor --fix` or replace `@username` with numeric sender IDs.                                                |

Full troubleshooting: [Telegram troubleshooting](/channels/telegram#troubleshooting)

## Discord

### Discord failure signatures

| Symptom                                   | Fastest check                                                          | Fix                                                                                                                                                                     |
| ----------------------------------------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Bot online but no guild replies           | `openclaw channels status --probe`                                     | Allow guild/channel and verify message content intent.                                                                                                                  |
| Group messages ignored                    | Check logs for mention gating drops                                    | Mention bot or set guild/channel `requireMention: false`.                                                                                                               |
| Typing/token usage but no Discord message | Session log shows assistant text with `didSendViaMessagingTool: false` | The model answered privately instead of calling the message tool. Use a tool-call-reliable model, or set `messages.groupChat.visibleReplies: "automatic"` to auto-post. |
| DM replies missing                        | `openclaw pairing list discord`                                        | Approve DM pairing or adjust DM policy.                                                                                                                                 |

Full troubleshooting: [Discord troubleshooting](/channels/discord#troubleshooting)

## Slack

### Slack failure signatures

| Symptom                                | Fastest check                             | Fix                                                                                                                                                  |
| -------------------------------------- | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Socket mode connected but no responses | `openclaw channels status --probe`        | Verify app token + bot token and required scopes; watch for `botTokenStatus` / `appTokenStatus = configured_unavailable` on SecretRef-backed setups. |
| DMs blocked                            | `openclaw pairing list slack`             | Approve pairing or relax DM policy.                                                                                                                  |
| Channel message ignored                | Check `groupPolicy` and channel allowlist | Allow the channel or switch policy to `open`.                                                                                                        |

Full troubleshooting: [Slack troubleshooting](/channels/slack#troubleshooting)

## iMessage and BlueBubbles

### iMessage and BlueBubbles failure signatures

| Symptom                          | Fastest check                                                           | Fix                                                   |
| -------------------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------- |
| No inbound events                | Verify webhook/server reachability and app permissions                  | Fix webhook URL or BlueBubbles server state.          |
| Can send but no receive on macOS | Check macOS privacy permissions for Messages automation                 | Re-grant TCC permissions and restart channel process. |
| DM sender blocked                | `openclaw pairing list imessage` or `openclaw pairing list bluebubbles` | Approve pairing or update allowlist.                  |

Full troubleshooting:

* [iMessage troubleshooting](/channels/imessage#troubleshooting)
* [BlueBubbles troubleshooting](/channels/bluebubbles#troubleshooting)

## Signal

### Signal failure signatures

| Symptom                         | Fastest check                              | Fix                                                      |
| ------------------------------- | ------------------------------------------ | -------------------------------------------------------- |
| Daemon reachable but bot silent | `openclaw channels status --probe`         | Verify `signal-cli` daemon URL/account and receive mode. |
| DM blocked                      | `openclaw pairing list signal`             | Approve sender or adjust DM policy.                      |
| Group replies do not trigger    | Check group allowlist and mention patterns | Add sender/group or loosen gating.                       |

Full troubleshooting: [Signal troubleshooting](/channels/signal#troubleshooting)

## QQ Bot

### QQ Bot failure signatures

| Symptom                         | Fastest check                               | Fix                                                             |
| ------------------------------- | ------------------------------------------- | --------------------------------------------------------------- |
| Bot replies "gone to Mars"      | Verify `appId` and `clientSecret` in config | Set credentials or restart the gateway.                         |
| No inbound messages             | `openclaw channels status --probe`          | Verify credentials on the QQ Open Platform.                     |
| Voice not transcribed           | Check STT provider config                   | Configure `channels.qqbot.stt` or `tools.media.audio`.          |
| Proactive messages not arriving | Check QQ platform interaction requirements  | QQ may block bot-initiated messages without recent interaction. |

Full troubleshooting: [QQ Bot troubleshooting](/channels/qqbot#troubleshooting)

## Matrix

### Matrix failure signatures

| Symptom                             | Fastest check                          | Fix                                                                       |
| ----------------------------------- | -------------------------------------- | ------------------------------------------------------------------------- |
| Logged in but ignores room messages | `openclaw channels status --probe`     | Check `groupPolicy`, room allowlist, and mention gating.                  |
| DMs do not process                  | `openclaw pairing list matrix`         | Approve sender or adjust DM policy.                                       |
| Encrypted rooms fail                | `openclaw matrix verify status`        | Re-verify the device, then check `openclaw matrix verify backup status`.  |
| Backup restore is pending/broken    | `openclaw matrix verify backup status` | Run `openclaw matrix verify backup restore` or rerun with a recovery key. |
| Cross-signing/bootstrap looks wrong | `openclaw matrix verify bootstrap`     | Repair secret storage, cross-signing, and backup state in one pass.       |

Full setup and config: [Matrix](/channels/matrix)

## Related

* [Pairing](/channels/pairing)
* [Channel routing](/channels/channel-routing)
* [Gateway troubleshooting](/gateway/troubleshooting)
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Twitch

Twitch chat support via IRC connection. OpenClaw connects as a Twitch user (bot account) to receive and send messages in channels.

## Bundled plugin

<Note>
  Twitch ships as a bundled plugin in current OpenClaw releases, so normal packaged builds do not need a separate install.
</Note>

If you are on an older build or a custom install that excludes Twitch, install the npm package directly:

<Tabs>
  <Tab title="npm registry">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw plugins install @openclaw/twitch
    ```
  </Tab>

  <Tab title="Local checkout">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw plugins install ./path/to/local/twitch-plugin
    ```
  </Tab>
</Tabs>

Use the bare package to follow the current official release tag. Pin an exact
version only when you need a reproducible install.

Details: [Plugins](/tools/plugin)

## Quick setup (beginner)

<Steps>
  <Step title="Ensure plugin is available">
    Current packaged OpenClaw releases already bundle it. Older/custom installs can add it manually with the commands above.
  </Step>

  <Step title="Create a Twitch bot account">
    Create a dedicated Twitch account for the bot (or use an existing account).
  </Step>

  <Step title="Generate credentials">
    Use [Twitch Token Generator](https://twitchtokengenerator.com/):

    * Select **Bot Token**
    * Verify scopes `chat:read` and `chat:write` are selected
    * Copy the **Client ID** and **Access Token**
  </Step>

  <Step title="Find your Twitch user ID">
    Use [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/) to convert a username to a Twitch user ID.
  </Step>

  <Step title="Configure the token">
    * Env: `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (default account only)
    * Or config: `channels.twitch.accessToken`

    If both are set, config takes precedence (env fallback is default-account only).
  </Step>

  <Step title="Start the gateway">
    Start the gateway with the configured channel.
  </Step>
</Steps>

<Warning>
  Add access control (`allowFrom` or `allowedRoles`) to prevent unauthorized users from triggering the bot. `requireMention` defaults to `true`.
</Warning>

Minimal config:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // Bot's Twitch account
      accessToken: "oauth:abc123...", // OAuth Access Token (or use OPENCLAW_TWITCH_ACCESS_TOKEN env var)
      clientId: "xyz789...", // Client ID from Token Generator
      channel: "vevisk", // Which Twitch channel's chat to join (required)
      allowFrom: ["123456789"], // (recommended) Your Twitch user ID only - get it from https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    },
  },
}
```

## What it is

* A Twitch channel owned by the Gateway.
* Deterministic routing: replies always go back to Twitch.
* Each account maps to an isolated session key `agent:<agentId>:twitch:<accountName>`.
* `username` is the bot's account (who authenticates), `channel` is which chat room to join.

## Setup (detailed)

### Generate credentials

Use [Twitch Token Generator](https://twitchtokengenerator.com/):

* Select **Bot Token**
* Verify scopes `chat:read` and `chat:write` are selected
* Copy the **Client ID** and **Access Token**

<Note>
  No manual app registration needed. Tokens expire after several hours.
</Note>

### Configure the bot

<Tabs>
  <Tab title="Env var (default account only)">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
    ```
  </Tab>

  <Tab title="Config">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        twitch: {
          enabled: true,
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
      },
    }
    ```
  </Tab>
</Tabs>

If both env and config are set, config takes precedence.

### Access control (recommended)

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // (recommended) Your Twitch user ID only
    },
  },
}
```

Prefer `allowFrom` for a hard allowlist. Use `allowedRoles` instead if you want role-based access.

**Available roles:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`.

<Note>
  **Why user IDs?** Usernames can change, allowing impersonation. User IDs are permanent.

  Find your Twitch user ID: [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/) (Convert your Twitch username to ID)
</Note>

## Token refresh (optional)

Tokens from [Twitch Token Generator](https://twitchtokengenerator.com/) cannot be automatically refreshed - regenerate when expired.

For automatic token refresh, create your own Twitch application at [Twitch Developer Console](https://dev.twitch.tv/console) and add to config:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token",
    },
  },
}
```

The bot automatically refreshes tokens before expiration and logs refresh events.

## Multi-account support

Use `channels.twitch.accounts` with per-account tokens. See [Configuration](/gateway/configuration) for the shared pattern.

Example (one bot account in two channels):

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel",
        },
      },
    },
  },
}
```

<Note>
  Each account needs its own token (one token per channel).
</Note>

## Access control

<Tabs>
  <Tab title="User ID allowlist (most secure)">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        twitch: {
          accounts: {
            default: {
              allowFrom: ["123456789", "987654321"],
            },
          },
        },
      },
    }
    ```
  </Tab>

  <Tab title="Role-based">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        twitch: {
          accounts: {
            default: {
              allowedRoles: ["moderator", "vip"],
            },
          },
        },
      },
    }
    ```

    `allowFrom` is a hard allowlist. When set, only those user IDs are allowed. If you want role-based access, leave `allowFrom` unset and configure `allowedRoles` instead.
  </Tab>

  <Tab title="Disable @mention requirement">
    By default, `requireMention` is `true`. To disable and respond to all messages:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      channels: {
        twitch: {
          accounts: {
            default: {
              requireMention: false,
            },
          },
        },
      },
    }
    ```
  </Tab>
</Tabs>

## Troubleshooting

First, run diagnostic commands:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw doctor
openclaw channels status --probe
```

<AccordionGroup>
  <Accordion title="Bot does not respond to messages">
    * **Check access control:** Ensure your user ID is in `allowFrom`, or temporarily remove `allowFrom` and set `allowedRoles: ["all"]` to test.
    * **Check the bot is in the channel:** The bot must join the channel specified in `channel`.
  </Accordion>

  <Accordion title="Token issues">
    "Failed to connect" or authentication errors:

    * Verify `accessToken` is the OAuth access token value (typically starts with `oauth:` prefix)
    * Check token has `chat:read` and `chat:write` scopes
    * If using token refresh, verify `clientSecret` and `refreshToken` are set
  </Accordion>

  <Accordion title="Token refresh not working">
    Check logs for refresh events:

    ```
    Using env token source for mybot
    Access token refreshed for user 123456 (expires in 14400s)
    ```

    If you see "token refresh disabled (no refresh token)":

    * Ensure `clientSecret` is provided
    * Ensure `refreshToken` is provided
  </Accordion>
</AccordionGroup>

## Config

### Account config

<ParamField path="username" type="string">
  Bot username.
</ParamField>

<ParamField path="accessToken" type="string">
  OAuth access token with `chat:read` and `chat:write`.
</ParamField>

<ParamField path="clientId" type="string">
  Twitch Client ID (from Token Generator or your app).
</ParamField>

<ParamField path="channel" type="string" required>
  Channel to join.
</ParamField>

<ParamField path="enabled" type="boolean" default="true">
  Enable this account.
</ParamField>

<ParamField path="clientSecret" type="string">
  Optional: for automatic token refresh.
</ParamField>

<ParamField path="refreshToken" type="string">
  Optional: for automatic token refresh.
</ParamField>

<ParamField path="expiresIn" type="number">
  Token expiry in seconds.
</ParamField>

<ParamField path="obtainmentTimestamp" type="number">
  Token obtained timestamp.
</ParamField>

<ParamField path="allowFrom" type="string[]">
  User ID allowlist.
</ParamField>

<ParamField path="allowedRoles" type="Array<&#x22;moderator&#x22; | &#x22;owner&#x22; | &#x22;vip&#x22; | &#x22;subscriber&#x22; | &#x22;all&#x22;>">
  Role-based access control.
</ParamField>

<ParamField path="requireMention" type="boolean" default="true">
  Require @mention.
</ParamField>

### Provider options

* `channels.twitch.enabled` - Enable/disable channel startup
* `channels.twitch.username` - Bot username (simplified single-account config)
* `channels.twitch.accessToken` - OAuth access token (simplified single-account config)
* `channels.twitch.clientId` - Twitch Client ID (simplified single-account config)
* `channels.twitch.channel` - Channel to join (simplified single-account config)
* `channels.twitch.accounts.<accountName>` - Multi-account config (all account fields above)

Full example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

## Tool actions

The agent can call `twitch` with action:

* `send` - Send a message to a channel

Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  action: "twitch",
  params: {
    message: "Hello Twitch!",
    to: "#mychannel",
  },
}
```

## Safety and ops

* **Treat tokens like passwords** — Never commit tokens to git.
* **Use automatic token refresh** for long-running bots.
* **Use user ID allowlists** instead of usernames for access control.
* **Monitor logs** for token refresh events and connection status.
* **Scope tokens minimally** — Only request `chat:read` and `chat:write`.
* **If stuck**: Restart the gateway after confirming no other process owns the session.

## Limits

* **500 characters** per message (auto-chunked at word boundaries).
* Markdown is stripped before chunking.
* No rate limiting (uses Twitch's built-in rate limits).

## Related

* [Channel Routing](/channels/channel-routing) — session routing for messages
* [Channels Overview](/channels) — all supported channels
* [Groups](/channels/groups) — group chat behavior and mention gating
* [Pairing](/channels/pairing) — DM authentication and pairing flow
* [Security](/gateway/security) — access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Yuanbao

Tencent Yuanbao is Tencent's AI assistant platform. The OpenClaw channel plugin
connects Yuanbao bots to OpenClaw over WebSocket so they can interact with users
through direct messages and group chats.

**Status:** production-ready for bot DMs + group chats. WebSocket is the only supported connection mode.

***

## Quick start

> **Requires OpenClaw 2026.4.10 or above.** Run `openclaw --version` to check. Upgrade with `openclaw update`.

<Steps>
  <Step title="Add the Yuanbao channel with your credentials">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw channels add --channel yuanbao --token "appKey:appSecret"
    ```

    The `--token` value uses colon-separated `appKey:appSecret` format. You can obtain these from the Yuanbao app by creating a robot in your application settings.
  </Step>

  <Step title="After setup completes, restart the gateway to apply the changes">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw gateway restart
    ```
  </Step>
</Steps>

### Interactive setup (alternative)

You can also use the interactive wizard:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw channels login --channel yuanbao
```

Follow the prompts to enter your App ID and App Secret.

***

## Access control

### Direct messages

Configure `dmPolicy` to control who can DM the bot:

* `"pairing"` - unknown users receive a pairing code; approve via CLI
* `"allowlist"` - only users listed in `allowFrom` can chat
* `"open"` - allow all users (default)
* `"disabled"` - disable all DMs

**Approve a pairing request:**

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw pairing list yuanbao
openclaw pairing approve yuanbao <CODE>
```

### Group chats

**Mention requirement** (`channels.yuanbao.requireMention`):

* `true` - require @mention (default)
* `false` - respond without @mention

Replying to the bot's message in a group chat is treated as an implicit mention.

***

## Configuration examples

### Basic setup with open DM policy

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      appKey: "your_app_key",
      appSecret: "your_app_secret",
      dm: {
        policy: "open",
      },
    },
  },
}
```

### Restrict DMs to specific users

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      appKey: "your_app_key",
      appSecret: "your_app_secret",
      dm: {
        policy: "allowlist",
        allowFrom: ["user_id_1", "user_id_2"],
      },
    },
  },
}
```

### Disable @mention requirement in groups

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      requireMention: false,
    },
  },
}
```

### Optimize outbound message delivery

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      // Send each chunk immediately without buffering
      outboundQueueStrategy: "immediate",
    },
  },
}
```

### Tune merge-text strategy

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      outboundQueueStrategy: "merge-text",
      minChars: 2800, // buffer until this many chars
      maxChars: 3000, // force split above this limit
      idleMs: 5000, // auto-flush after idle timeout (ms)
    },
  },
}
```

***

## Common commands

| Command    | Description                 |
| ---------- | --------------------------- |
| `/help`    | Show available commands     |
| `/status`  | Show bot status             |
| `/new`     | Start a new session         |
| `/stop`    | Stop the current run        |
| `/restart` | Restart OpenClaw            |
| `/compact` | Compact the session context |

> Yuanbao supports native slash-command menus. Commands are synced to the platform automatically when the gateway starts.

***

## Troubleshooting

### Bot does not respond in group chats

1. Ensure the bot is added to the group
2. Ensure you @mention the bot (required by default)
3. Check logs: `openclaw logs --follow`

### Bot does not receive messages

1. Ensure the bot is created and approved in the Yuanbao app
2. Ensure `appKey` and `appSecret` are correctly configured
3. Ensure the gateway is running: `openclaw gateway status`
4. Check logs: `openclaw logs --follow`

### Bot sends empty or fallback replies

1. Check if the AI model is returning valid content
2. The default fallback reply is: "暂时无法解答，你可以换个问题问问我哦"
3. Customize it via `channels.yuanbao.fallbackReply`

### App Secret leaked

1. Reset the App Secret in YuanBao APP
2. Update the value in your config
3. Restart the gateway: `openclaw gateway restart`

***

## Advanced configuration

### Multiple accounts

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      defaultAccount: "main",
      accounts: {
        main: {
          appKey: "key_xxx",
          appSecret: "secret_xxx",
          name: "Primary bot",
        },
        backup: {
          appKey: "key_yyy",
          appSecret: "secret_yyy",
          name: "Backup bot",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount` controls which account is used when outbound APIs do not specify an `accountId`.

### Message limits

* `maxChars` - single message max character count (default: `3000` chars)
* `mediaMaxMb` - media upload/download limit (default: `20` MB)
* `overflowPolicy` - behavior when message exceeds limit: `"split"` (default) or `"stop"`

### Streaming

Yuanbao supports block-level streaming output. When enabled, the bot sends text in chunks as it generates.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      disableBlockStreaming: false, // block streaming enabled (default)
    },
  },
}
```

Set `disableBlockStreaming: true` to send the complete reply in one message.

### Group chat history context

Control how many historical messages are included in the AI context for group chats:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      historyLimit: 100, // default: 100, set 0 to disable
    },
  },
}
```

### Reply-to mode

Control how the bot quotes messages when replying in group chats:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      replyToMode: "first", // "off" | "first" | "all" (default: "first")
    },
  },
}
```

| Value     | Behavior                                                 |
| --------- | -------------------------------------------------------- |
| `"off"`   | No quote reply                                           |
| `"first"` | Quote only the first reply per inbound message (default) |
| `"all"`   | Quote every reply                                        |

### Markdown hint injection

By default, the bot injects instructions in the system prompt to prevent the AI model from wrapping the entire reply in markdown code blocks.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      markdownHintEnabled: true, // default: true
    },
  },
}
```

### Debug mode

Enable unsanitized log output for specific bot IDs:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    yuanbao: {
      debugBotIds: ["bot_user_id_1", "bot_user_id_2"],
    },
  },
}
```

### Multi-agent routing

Use `bindings` to route Yuanbao DMs or groups to different agents.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    list: [
      { id: "main" },
      { id: "agent-a", workspace: "/home/user/agent-a" },
      { id: "agent-b", workspace: "/home/user/agent-b" },
    ],
  },
  bindings: [
    {
      agentId: "agent-a",
      match: {
        channel: "yuanbao",
        peer: { kind: "direct", id: "user_xxx" },
      },
    },
    {
      agentId: "agent-b",
      match: {
        channel: "yuanbao",
        peer: { kind: "group", id: "group_zzz" },
      },
    },
  ],
}
```

Routing fields:

* `match.channel`: `"yuanbao"`
* `match.peer.kind`: `"direct"` (DM) or `"group"` (group chat)
* `match.peer.id`: user ID or group code

***

## Configuration reference

Full configuration: [Gateway configuration](/gateway/configuration)

| Setting                                    | Description                                       | Default              |
| ------------------------------------------ | ------------------------------------------------- | -------------------- |
| `channels.yuanbao.enabled`                 | Enable/disable the channel                        | `true`               |
| `channels.yuanbao.defaultAccount`          | Default account for outbound routing              | `default`            |
| `channels.yuanbao.accounts.<id>.appKey`    | App Key (used for signing and ticket generation)  | -                    |
| `channels.yuanbao.accounts.<id>.appSecret` | App Secret (used for signing)                     | -                    |
| `channels.yuanbao.accounts.<id>.token`     | Pre-signed token (skips automatic ticket signing) | -                    |
| `channels.yuanbao.accounts.<id>.name`      | Account display name                              | -                    |
| `channels.yuanbao.accounts.<id>.enabled`   | Enable/disable a specific account                 | `true`               |
| `channels.yuanbao.dm.policy`               | DM policy                                         | `open`               |
| `channels.yuanbao.dm.allowFrom`            | DM allowlist (user ID list)                       | -                    |
| `channels.yuanbao.requireMention`          | Require @mention in groups                        | `true`               |
| `channels.yuanbao.overflowPolicy`          | Long message handling (`split` or `stop`)         | `split`              |
| `channels.yuanbao.replyToMode`             | Group reply-to strategy (`off`, `first`, `all`)   | `first`              |
| `channels.yuanbao.outboundQueueStrategy`   | Outbound strategy (`merge-text` or `immediate`)   | `merge-text`         |
| `channels.yuanbao.minChars`                | Merge-text: min chars to trigger send             | `2800`               |
| `channels.yuanbao.maxChars`                | Merge-text: max chars per message                 | `3000`               |
| `channels.yuanbao.idleMs`                  | Merge-text: idle timeout before auto-flush (ms)   | `5000`               |
| `channels.yuanbao.mediaMaxMb`              | Media size limit (MB)                             | `20`                 |
| `channels.yuanbao.historyLimit`            | Group chat history context entries                | `100`                |
| `channels.yuanbao.disableBlockStreaming`   | Disable block-level streaming output              | `false`              |
| `channels.yuanbao.fallbackReply`           | Fallback reply when AI returns no content         | `暂时无法解答，你可以换个问题问问我哦` |
| `channels.yuanbao.markdownHintEnabled`     | Inject markdown anti-wrapping instructions        | `true`               |
| `channels.yuanbao.debugBotIds`             | Debug whitelist bot IDs (unsanitized logs)        | `[]`                 |

***

## Supported message types

### Receive

* ✅ Text
* ✅ Images
* ✅ Files
* ✅ Audio / Voice
* ✅ Video
* ✅ Stickers / Custom emoji
* ✅ Custom elements (link cards, etc.)

### Send

* ✅ Text (with markdown support)
* ✅ Images
* ✅ Files
* ✅ Audio
* ✅ Video
* ✅ Stickers

### Threads and replies

* ✅ Quote replies (configurable via `replyToMode`)
* ❌ Thread replies (not supported by platform)

***

## Related

* [Channels Overview](/channels) - all supported channels
* [Pairing](/channels/pairing) - DM authentication and pairing flow
* [Groups](/channels/groups) - group chat behavior and mention gating
* [Channel Routing](/channels/channel-routing) - session routing for messages
* [Security](/gateway/security) - access model and hardening
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Zalo personal

Status: experimental. This integration automates a **personal Zalo account** via native `zca-js` inside OpenClaw.

<Warning>
  This is an unofficial integration and may result in account suspension or ban. Use at your own risk.
</Warning>

## Bundled plugin

Zalo Personal ships as a bundled plugin in current OpenClaw releases, so normal
packaged builds do not need a separate install.

If you are on an older build or a custom install that excludes Zalo Personal,
install the npm package directly:

* Install via CLI: `openclaw plugins install @openclaw/zalouser`
* Pinned version: `openclaw plugins install @openclaw/zalouser@2026.5.2`
* Or from a source checkout: `openclaw plugins install ./path/to/local/zalouser-plugin`
* Details: [Plugins](/tools/plugin)

No external `zca`/`openzca` CLI binary is required.

## Quick setup (beginner)

1. Ensure the Zalo Personal plugin is available.
   * Current packaged OpenClaw releases already bundle it.
   * Older/custom installs can add it manually with the commands above.
2. Login (QR, on the Gateway machine):
   * `openclaw channels login --channel zalouser`
   * Scan the QR code with the Zalo mobile app.
3. Enable the channel:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

4. Restart the Gateway (or finish setup).
5. DM access defaults to pairing; approve the pairing code on first contact.

## What it is

* Runs entirely in-process via `zca-js`.
* Uses native event listeners to receive inbound messages.
* Sends replies directly through the JS API (text/media/link).
* Designed for "personal account" use cases where Zalo Bot API is not available.

## Naming

Channel id is `zalouser` to make it explicit this automates a **personal Zalo user account** (unofficial). We keep `zalo` reserved for a potential future official Zalo API integration.

## Finding IDs (directory)

Use the directory CLI to discover peers/groups and their IDs:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

## Limits

* Outbound text is chunked to \~2000 characters (Zalo client limits).
* Streaming is blocked by default.

## Access control (DMs)

`channels.zalouser.dmPolicy` supports: `pairing | allowlist | open | disabled` (default: `pairing`).

`channels.zalouser.allowFrom` should use stable Zalo user IDs. During interactive setup, entered names can be resolved to IDs using the plugin's in-process contact lookup.

If a raw name remains in config, startup resolves it only when `channels.zalouser.dangerouslyAllowNameMatching: true` is enabled. Without that opt-in, runtime sender checks are ID-only and raw names are ignored for authorization.

Approve via:

* `openclaw pairing list zalouser`
* `openclaw pairing approve zalouser <code>`

## Group access (optional)

* Default: `channels.zalouser.groupPolicy = "open"` (groups allowed). Use `channels.defaults.groupPolicy` to override the default when unset.
* Restrict to an allowlist with:
  * `channels.zalouser.groupPolicy = "allowlist"`
  * `channels.zalouser.groups` (keys should be stable group IDs; names are resolved to IDs on startup only when `channels.zalouser.dangerouslyAllowNameMatching: true` is enabled)
  * `channels.zalouser.groupAllowFrom` (controls which senders in allowed groups can trigger the bot)
* Block all groups: `channels.zalouser.groupPolicy = "disabled"`.
* The configure wizard can prompt for group allowlists.
* On startup, OpenClaw resolves group/user names in allowlists to IDs and logs the mapping only when `channels.zalouser.dangerouslyAllowNameMatching: true` is enabled.
* Group allowlist matching is ID-only by default. Unresolved names are ignored for auth unless `channels.zalouser.dangerouslyAllowNameMatching: true` is enabled.
* `channels.zalouser.dangerouslyAllowNameMatching: true` is a break-glass compatibility mode that re-enables mutable startup name resolution and runtime group-name matching.
* If `groupAllowFrom` is unset, runtime falls back to `allowFrom` for group sender checks.
* Sender checks apply to both normal group messages and control commands (for example `/new`, `/reset`).

Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["1471383327500481391"],
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```

### Group mention gating

* `channels.zalouser.groups.<group>.requireMention` controls whether group replies require a mention.
* Resolution order: exact group id/name -> normalized group slug -> `*` -> default (`true`).
* This applies both to allowlisted groups and open group mode.
* Quoting a bot message counts as an implicit mention for group activation.
* Authorized control commands (for example `/new`) can bypass mention gating.
* When a group message is skipped because mention is required, OpenClaw stores it as pending group history and includes it on the next processed group message.
* Group history limit defaults to `messages.groupChat.historyLimit` (fallback `50`). You can override per account with `channels.zalouser.historyLimit`.

Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "*": { allow: true, requireMention: true },
        "Work Chat": { allow: true, requireMention: false },
      },
    },
  },
}
```

## Multi-account

Accounts map to `zalouser` profiles in OpenClaw state. Example:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```

## Typing, reactions, and delivery acknowledgements

* OpenClaw sends a typing event before dispatching a reply (best-effort).
* Message reaction action `react` is supported for `zalouser` in channel actions.
  * Use `remove: true` to remove a specific reaction emoji from a message.
  * Reaction semantics: [Reactions](/tools/reactions)
* For inbound messages that include event metadata, OpenClaw sends delivered + seen acknowledgements (best-effort).

## Troubleshooting

**Login doesn't stick:**

* `openclaw channels status --probe`
* Re-login: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`

**Allowlist/group name didn't resolve:**

* Use numeric IDs in `allowFrom`/`groupAllowFrom` and stable group IDs in `groups`. If you intentionally need exact friend/group names, enable `channels.zalouser.dangerouslyAllowNameMatching: true`.

**Upgraded from old CLI-based setup:**

* Remove any old external `zca` process assumptions.
* The channel now runs fully in OpenClaw without external CLI binaries.

## Related

* [Channels Overview](/channels) — all supported channels
* [Pairing](/channels/pairing) — DM authentication and pairing flow
* [Groups](/channels/groups) — group chat behavior and mention gating
* [Channel Routing](/channels/channel-routing) — session routing for messages
* [Security](/gateway/security) — access model and hardening
