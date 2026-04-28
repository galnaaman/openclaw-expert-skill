# OpenClaw Cli Documentation

## Health - OpenClaw
**Source:** https://docs.openclaw.ai/cli/health

[Skip to main content](https://docs.openclaw.ai/cli/health#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Gateway and service

Health

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw health](https://docs.openclaw.ai/cli/health#openclaw-health)
- [Related](https://docs.openclaw.ai/cli/health#related)

# [​](https://docs.openclaw.ai/cli/health\\#openclaw-health)  `openclaw health`

Fetch health from the running Gateway.Options:

- `--json`: machine-readable output
- `--timeout <ms>`: connection timeout in milliseconds (default `10000`)
- `--verbose`: verbose logging
- `--debug`: alias for `--verbose`

Examples:

```
openclaw health
openclaw health --json
openclaw health --timeout 2500
openclaw health --verbose
openclaw health --debug
```

Notes:

- Default `openclaw health` asks the running gateway for its health snapshot. When the
gateway already has a fresh cached snapshot, it can return that cached payload and
refresh in the background.
- `--verbose` forces a live probe, prints gateway connection details, and expands the
human-readable output across all configured accounts and agents.
- Output includes per-agent session stores when multiple agents are configured.

## [​](https://docs.openclaw.ai/cli/health\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Gateway health](https://docs.openclaw.ai/gateway/health)

[Gateway](https://docs.openclaw.ai/cli/gateway) [Logs](https://docs.openclaw.ai/cli/logs)

Ctrl+I

---

## Node - OpenClaw
**Source:** https://docs.openclaw.ai/cli/node

[Skip to main content](https://docs.openclaw.ai/cli/node#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools and execution

Node

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw node](https://docs.openclaw.ai/cli/node#openclaw-node)
- [Why use a node host?](https://docs.openclaw.ai/cli/node#why-use-a-node-host)
- [Browser proxy (zero-config)](https://docs.openclaw.ai/cli/node#browser-proxy-zero-config)
- [Run (foreground)](https://docs.openclaw.ai/cli/node#run-foreground)
- [Gateway auth for node host](https://docs.openclaw.ai/cli/node#gateway-auth-for-node-host)
- [Service (background)](https://docs.openclaw.ai/cli/node#service-background)
- [Pairing](https://docs.openclaw.ai/cli/node#pairing)
- [Exec approvals](https://docs.openclaw.ai/cli/node#exec-approvals)
- [Related](https://docs.openclaw.ai/cli/node#related)

# [​](https://docs.openclaw.ai/cli/node\\#openclaw-node)  `openclaw node`

Run a **headless node host** that connects to the Gateway WebSocket and exposes
`system.run` / `system.which` on this machine.

## [​](https://docs.openclaw.ai/cli/node\\#why-use-a-node-host)  Why use a node host?

Use a node host when you want agents to **run commands on other machines** in your
network without installing a full macOS companion app there.Common use cases:

- Run commands on remote Linux/Windows boxes (build servers, lab machines, NAS).
- Keep exec **sandboxed** on the gateway, but delegate approved runs to other hosts.
- Provide a lightweight, headless execution target for automation or CI nodes.

Execution is still guarded by **exec approvals** and per‑agent allowlists on the
node host, so you can keep command access scoped and explicit.

## [​](https://docs.openclaw.ai/cli/node\\#browser-proxy-zero-config)  Browser proxy (zero-config)

Node hosts automatically advertise a browser proxy if `browser.enabled` is not
disabled on the node. This lets the agent use browser automation on that node
without extra configuration.By default, the proxy exposes the node’s normal browser profile surface. If you
set `nodeHost.browserProxy.allowProfiles`, the proxy becomes restrictive:
non-allowlisted profile targeting is rejected, and persistent profile
create/delete routes are blocked through the proxy.Disable it on the node if needed:

```
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```

## [​](https://docs.openclaw.ai/cli/node\\#run-foreground)  Run (foreground)

```
openclaw node run --host <gateway-host> --port 18789
```

Options:

- `--host <host>`: Gateway WebSocket host (default: `127.0.0.1`)
- `--port <port>`: Gateway WebSocket port (default: `18789`)
- `--tls`: Use TLS for the gateway connection
- `--tls-fingerprint <sha256>`: Expected TLS certificate fingerprint (sha256)
- `--node-id <id>`: Override node id (clears pairing token)
- `--display-name <name>`: Override the node display name

## [​](https://docs.openclaw.ai/cli/node\\#gateway-auth-for-node-host)  Gateway auth for node host

`openclaw node run` and `openclaw node install` resolve gateway auth from config/env (no `--token`/`--password` flags on node commands):

- `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD` are checked first.
- Then local config fallback: `gateway.auth.token` / `gateway.auth.password`.
- In local mode, node host intentionally does not inherit `gateway.remote.token` / `gateway.remote.password`.
- If `gateway.auth.token` / `gateway.auth.password` is explicitly configured via SecretRef and unresolved, node auth resolution fails closed (no remote fallback masking).
- In `gateway.mode=remote`, remote client fields (`gateway.remote.token` / `gateway.remote.password`) are also eligible per remote precedence rules.
- Node host auth resolution only honors `OPENCLAW_GATEWAY_*` env vars.

For a node connecting to a non-loopback `ws://` Gateway on a trusted private
network, set `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1`. Without it, node startup
fails closed and asks you to use `wss://`, an SSH tunnel, or Tailscale.
This is a process-environment opt-in, not an an `openclaw.json` config key.
`openclaw node install` persists it into the supervised node service when it is
present in the install command environment.

## [​](https://docs.openclaw.ai/cli/node\\#service-background)  Service (background)

Install a headless node host as a user service.

```
openclaw node install --host <gateway-host> --port 18789
```

Options:

- `--host <host>`: Gateway WebSocket host (default: `127.0.0.1`)
- `--port <port>`: Gateway WebSocket port (default: `18789`)
- `--tls`: Use TLS for the gateway connection
- `--tls-fingerprint <sha256>`: Expected TLS certificate fingerprint (sha256)
- `--node-id <id>`: Override node id (clears pairing token)
- `--display-name <name>`: Override the node display name
- `--runtime <runtime>`: Service runtime (`node` or `bun`)
- `--force`: Reinstall/overwrite if already installed

Manage the service:

```
openclaw node status
openclaw node start
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Use `openclaw node run` for a foreground node host (no service).Service commands accept `--json` for machine-readable output.The node host retries Gateway restart and network closes in-process. If the
Gateway reports a terminal token/password/bootstrap auth pause, the node host
logs the close detail and exits non-zero so launchd/systemd can restart it with
fresh config and credentials. Pairing-required pauses stay in the foreground
flow so the pending request can be approved.

## [​](https://docs.openclaw.ai/cli/node\\#pairing)  Pairing

The first connection creates a pending device pairing request (`role: node`) on the Gateway.
Approve it via:

```
openclaw devices list
openclaw devices approve <requestId>
```

On tightly controlled node networks, the Gateway operator can explicitly opt in
to auto-approving first-time node pairing from trusted CIDRs:

```
{
  gateway: {
    nodes: {
      pairing: {
        autoApproveCidrs: [\"192.168.1.0/24\"],
      },
    },
  },
}
```

This is disabled by default. It only applies to fresh `role: node` pairing with
no requested scopes. Operator/browser clients, Control UI, WebChat, and role,
scope, metadata, or public-key upgrades still require manual approval.If the node retries pairing with changed auth details (role/scopes/public key),
the previous pending request is superseded and a new `requestId` is created.
Run `openclaw devices list` again before approval.The node host stores its node id, token, display name, and gateway connection info in
`~/.openclaw/node.json`.

## [​](https://docs.openclaw.ai/cli/node\\#exec-approvals)  Exec approvals

`system.run` is gated by local exec approvals:

- `~/.openclaw/exec-approvals.json`
- [Exec approvals](https://docs.openclaw.ai/tools/exec-approvals)
- `openclaw approvals --node <id|name|ip>` (edit from the Gateway)

For approved async node exec, OpenClaw prepares a canonical `systemRunPlan`
before prompting. The later approved `system.run` forward reuses that stored
plan, so edits to command/cwd/session fields after the approval request was
created are rejected instead of changing what the node executes.

## [​](https://docs.openclaw.ai/cli/node\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Nodes](https://docs.openclaw.ai/nodes)

[Flows (redirect)](https://docs.openclaw.ai/cli/flows) [Nodes](https://docs.openclaw.ai/cli/nodes)

Ctrl+I

---

## QR - OpenClaw
**Source:** https://docs.openclaw.ai/cli/qr

[Skip to main content](https://docs.openclaw.ai/cli/qr#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Channels and messaging

QR

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw qr](https://docs.openclaw.ai/cli/qr#openclaw-qr)
- [Usage](https://docs.openclaw.ai/cli/qr#usage)
- [Options](https://docs.openclaw.ai/cli/qr#options)
- [Notes](https://docs.openclaw.ai/cli/qr#notes)
- [Related](https://docs.openclaw.ai/cli/qr#related)

# [​](https://docs.openclaw.ai/cli/qr\\#openclaw-qr)  `openclaw qr`

Generate a mobile pairing QR and setup code from your current Gateway configuration.

## [​](https://docs.openclaw.ai/cli/qr\\#usage)  Usage

```
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws
```

## [​](https://docs.openclaw.ai/cli/qr\\#options)  Options

- `--remote`: prefer `gateway.remote.url`; if it is unset, `gateway.tailscale.mode=serve|funnel` can still provide the remote public URL
- `--url <url>`: override gateway URL used in payload
- `--public-url <url>`: override public URL used in payload
- `--token <token>`: override which gateway token the bootstrap flow authenticates against
- `--password <password>`: override which gateway password the bootstrap flow authenticates against
- `--setup-code-only`: print only setup code
- `--no-ascii`: skip ASCII QR rendering
- `--json`: emit JSON (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## [​](https://docs.openclaw.ai/cli/qr\\#notes)  Notes

- `--token` and `--password` are mutually exclusive.
- The setup code itself now carries an opaque short-lived `bootstrapToken`, not the shared gateway token/password.
- In the built-in node/operator bootstrap flow, the primary node token still lands with `scopes: []`.
- If bootstrap handoff also issues an operator token, it stays bounded to the bootstrap allowlist: `operator.approvals`, `operator.read`, `operator.talk.secrets`, `operator.write`.
- Bootstrap scope checks are role-prefixed. That operator allowlist only satisfies operator requests; non-operator roles still need scopes under their own role prefix.
- Mobile pairing fails closed for Tailscale/public `ws://` gateway URLs. Private LAN `ws://` remains supported, but Tailscale/public mobile routes should use Tailscale Serve/Funnel or a `wss://` gateway URL.
- With `--remote`, OpenClaw requires either `gateway.remote.url` or
`gateway.tailscale.mode=serve|funnel`.
- With `--remote`, if effectively active remote credentials are configured as SecretRefs and you do not pass `--token` or `--password`, the command resolves them from the active gateway snapshot. If gateway is unavailable, the command fails fast.
- Without `--remote`, local gateway auth SecretRefs are resolved when no CLI auth override is passed:

  - `gateway.auth.token` resolves when token auth can win (explicit `gateway.auth.mode=\"token\"` or inferred mode where no password source wins).
  - `gateway.auth.password` resolves when password auth can win (explicit `gateway.auth.mode=\"password\"` or inferred mode with no winning token from auth/env).
- If both `gateway.auth.token` and `gateway.auth.password` are configured (including SecretRefs) and `gateway.auth.mode` is unset, setup-code resolution fails until mode is set explicitly.
- Gateway version skew note: this command path requires a gateway that supports `secrets.resolve`; older gateways return an unknown-method error.
- After scanning, approve device pairing with:
  - `openclaw devices list`
  - `openclaw devices approve <requestId>`

## [​](https://docs.openclaw.ai/cli/qr\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Pairing](https://docs.openclaw.ai/cli/pairing)

[Pairing](https://docs.openclaw.ai/cli/pairing) [Voicecall](https://docs.openclaw.ai/cli/voicecall)

Ctrl+I

---

## Agent - OpenClaw
**Source:** https://docs.openclaw.ai/cli/agent

[Skip to main content](https://docs.openclaw.ai/cli/agent#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Agents and sessions

Agent

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw agent](https://docs.openclaw.ai/cli/agent#openclaw-agent)
- [Options](https://docs.openclaw.ai/cli/agent#options)
- [Examples](https://docs.openclaw.ai/cli/agent#examples)
- [Notes](https://docs.openclaw.ai/cli/agent#notes)
- [Related](https://docs.openclaw.ai/cli/agent#related)

# [​](https://docs.openclaw.ai/cli/agent\\#openclaw-agent)  `openclaw agent`

Run an agent turn via the Gateway (use `--local` for embedded).\nUse `--agent <id>` to target a configured agent directly.Pass at least one session selector:\n
- `--to <dest>`\n- `--session-id <id>`\n- `--agent <id>`\n
Related:\n
- Agent send tool: [Agent send](https://docs.openclaw.ai/tools/agent-send)\n
## [​](https://docs.openclaw.ai/cli/agent\\#options)  Options\n
- `-m, --message <text>`: required message body\n- `-t, --to <dest>`: recipient used to derive the session key\n- `--session-id <id>`: explicit session id\n- `--agent <id>`: agent id; overrides routing bindings\n- `--model <id>`: model override for this run (`provider/model` or model id)\n- `--thinking <level>`: agent thinking level (`off`, `minimal`, `low`, `medium`, `high`, plus provider-supported custom levels such as `xhigh`, `adaptive`, or `max`)\n- `--verbose <on|off>`: persist verbose level for the session\n- `--channel <channel>`: delivery channel; omit to use the main session channel\n- `--reply-to <target>`: delivery target override\n- `--reply-channel <channel>`: delivery channel override\n- `--reply-account <id>`: delivery account override\n- `--local`: run the embedded agent directly (after plugin registry preload)\n- `--deliver`: send the reply back to the selected channel/target\n- `--timeout <seconds>`: override agent timeout (default 600 or config value)\n- `--json`: output JSON\n
## [​](https://docs.openclaw.ai/cli/agent\\#examples)  Examples\n
```\nopenclaw agent --to +15555550123 --message \"status update\" --deliver\nopenclaw agent --agent ops --message \"Summarize logs\"\nopenclaw agent --agent ops --model openai/gpt-5.4 --message \"Summarize logs\"\nopenclaw agent --session-id 1234 --message \"Summarize inbox\" --thinking medium\nopenclaw agent --to +15555550123 --message \"Trace logs\" --verbose on --json\nopenclaw agent --agent ops --message \"Generate report\" --deliver --reply-channel slack --reply-to \"#reports\"\nopenclaw agent --agent ops --message \"Run locally\" --local\n```\n
## [​](https://docs.openclaw.ai/cli/agent\\#notes)  Notes\n
- Gateway mode falls back to the embedded agent when the Gateway request fails. Use `--local` to force embedded execution up front.\n- `--local` still preloads the plugin registry first, so plugin-provided providers, tools, and channels stay available during embedded runs.\n- Each `openclaw agent` invocation is treated as a one-shot run. Bundled or user-configured MCP servers opened for that run are retired after the reply, even when the command uses the Gateway path, so stdio MCP child processes do not stay alive between scripted invocations.\n- `--channel`, `--reply-channel`, and `--reply-account` affect reply delivery, not session routing.\n- `--json` keeps stdout reserved for the JSON response. Gateway, plugin, and embedded-fallback diagnostics are routed to stderr so scripts can parse stdout directly.\n- Embedded fallback JSON includes `meta.transport: \"embedded\"` and `meta.fallbackFrom: \"gateway\"` so scripts can distinguish fallback runs from Gateway runs.\n- When this command triggers `models.json` regeneration, SecretRef-managed provider credentials are persisted as non-secret markers (for example env var names, `secretref-env:ENV_VAR_NAME`, or `secretref-managed`), not resolved secret plaintext.\n- Marker writes are source-authoritative: OpenClaw persists markers from the active source config snapshot, not from resolved runtime secret values.\n
## [​](https://docs.openclaw.ai/cli/agent\\#related)  Related\n
- [CLI reference](https://docs.openclaw.ai/cli)\n- [Agent runtime](https://docs.openclaw.ai/concepts/agent)\n
[Update](https://docs.openclaw.ai/cli/update) [Agents](https://docs.openclaw.ai/cli/agents)\n
Ctrl+I

---

## Memory - OpenClaw
**Source:** https://docs.openclaw.ai/cli/memory

[Skip to main content](https://docs.openclaw.ai/cli/memory#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nAgents and sessions\n\nMemory\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [openclaw memory](https://docs.openclaw.ai/cli/memory#openclaw-memory)\n- [Examples](https://docs.openclaw.ai/cli/memory#examples)\n- [Options](https://docs.openclaw.ai/cli/memory#options)\n- [Dreaming](https://docs.openclaw.ai/cli/memory#dreaming)\n- [Related](https://docs.openclaw.ai/cli/memory#related)\n\n# [​](https://docs.openclaw.ai/cli/memory\\#openclaw-memory)  `openclaw memory`\n\nManage semantic memory indexing and search.\nProvided by the active memory plugin (default: `memory-core`; set `plugins.slots.memory = \"none\"` to disable).Related:\n\n- Memory concept: [Memory](https://docs.openclaw.ai/concepts/memory)\n- Memory wiki: [Memory Wiki](https://docs.openclaw.ai/plugins/memory-wiki)\n- Wiki CLI: [wiki](https://docs.openclaw.ai/cli/wiki)\n- Plugins: [Plugins](https://docs.openclaw.ai/tools/plugin)\n\n## [​](https://docs.openclaw.ai/cli/memory\\#examples)  Examples\n\n```\nopenclaw memory status\nopenclaw memory status --deep\nopenclaw memory status --fix\nopenclaw memory index --force\nopenclaw memory search \"meeting notes\"\nopenclaw memory search --query \"deployment\" --max-results 20\nopenclaw memory promote --limit 10 --min-score 0.75\nopenclaw memory promote --apply\nopenclaw memory promote --json --min-recall-count 0 --min-unique-queries 0\nopenclaw memory promote-explain \"router vlan\"\nopenclaw memory promote-explain \"router vlan\" --json\nopenclaw memory rem-harness\nopenclaw memory rem-harness --json\nopenclaw memory status --json\nopenclaw memory status --deep --index\nopenclaw memory status --deep --index --verbose\nopenclaw memory status --agent main\nopenclaw memory index --agent main --verbose\n```\n\n## [​](https://docs.openclaw.ai/cli/memory\\#options)  Options\n\n`memory status` and `memory index`:\n\n- `--agent <id>`: scope to a single agent. Without it, these commands run for each configured agent; if no agent list is configured, they fall back to the default agent.\n- `--verbose`: emit detailed logs during probes and indexing.\n\n`memory status`:\n\n- `--deep`: probe vector + embedding availability.\n- `--index`: run a reindex if the store is dirty (implies `--deep`).\n- `--fix`: repair stale recall locks and normalize promotion metadata.\n- `--json`: print JSON output.\n\nIf `memory status` shows `Dreaming status: blocked`, the managed dreaming cron is enabled but the heartbeat that drives it is not firing for the default agent. See [Dreaming never runs](https://docs.openclaw.ai/concepts/dreaming#dreaming-never-runs-status-shows-blocked) for the two common causes.`memory index`:\n\n- `--force`: force a full reindex.\n\n`memory search`:\n\n- Query input: pass either positional `[query]` or `--query <text>`.\n- If both are provided, `--query` wins.\n- If neither is provided, the command exits with an error.\n- `--agent <id>`: scope to a single agent (default: the default agent).\n- `--max-results <n>`: limit the number of results returned.\n- `--min-score <n>`: filter out low-score matches.\n- `--json`: print JSON results.\n\n`memory promote`:Preview and apply short-term memory promotions.\n\n```\nopenclaw memory promote [--apply] [--limit <n>] [--include-promoted]\n```\n\n- `--apply` — write promotions to `MEMORY.md` (default: preview only).\n- `--limit <n>` — cap the number of candidates shown.\n- `--include-promoted` — include entries already promoted in previous cycles.\n\nFull options:\n\n- Ranks short-term candidates from `memory/YYYY-MM-DD.md` using weighted promotion signals (`frequency`, `relevance`, `query diversity`, `recency`, `consolidation`, `conceptual richness`).\n- Uses short-term signals from both memory recalls and daily-ingestion passes, plus light/REM phase reinforcement signals.\n- When dreaming is enabled, `memory-core` auto-manages one cron job that runs a full sweep (`light -> REM -> deep`) in the background (no manual `openclaw cron add` required).\n- `--agent <id>`: scope to a single agent (default: the default agent).\n- `--limit <n>`: max candidates to return/apply.\n- `--min-score <n>`: minimum weighted promotion score.\n- `--min-recall-count <n>`: minimum recall count required for a candidate.\n- `--min-unique-queries <n>`: minimum distinct query count required for a candidate.\n- `--apply`: append selected candidates into `MEMORY.md` and mark them promoted.\n- `--include-promoted`: include already promoted candidates in output.\n- `--json`: print JSON output.\n\n`memory promote-explain`:Explain a specific promotion candidate and its score breakdown.\n\n```\nopenclaw memory promote-explain <selector> [--agent <id>] [--include-promoted] [--json]\n```\n\n- `<selector>`: candidate key, path fragment, or snippet fragment to look up.\n- `--agent <id>`: scope to a single agent (default: the default agent).\n- `--include-promoted`: include already promoted candidates.\n- `--json`: print JSON output.\n\n`memory rem-harness`:Preview REM reflections, candidate truths, and deep promotion output without writing anything.\n\n```\nopenclaw memory rem-harness [--agent <id>] [--include-promoted] [--json]\n```\n\n- `--agent <id>`: scope to a single agent (default: the default agent).\n- `--include-promoted`: include already promoted deep candidates.\n- `--json`: print JSON output.\n\n## [​](https://docs.openclaw.ai/cli/memory\\#dreaming)  Dreaming\n\nDreaming is the background memory consolidation system with three cooperative\nphases: **light** (sort/stage short-term material), **deep** (promote durable\nfacts into `MEMORY.md`), and **REM** (reflect and surface themes).\n\n- Enable with `plugins.entries.memory-core.config.dreaming.enabled: true`.\n- Toggle from chat with `/dreaming on|off` (or inspect with `/dreaming status`).\n- Dreaming runs on one managed sweep schedule (`dreaming.frequency`) and executes phases in order: light, REM, deep.\n- Only the deep phase writes durable memory to `MEMORY.md`.\n- Human-readable phase output and diary entries are written to `DREAMS.md` (or existing `dreams.md`), with optional per-phase reports in `memory/dreaming/<phase>/YYYY-MM-DD.md`.\n- Ranking uses weighted signals: recall frequency, retrieval relevance, query diversity, temporal recency, cross-day consolidation, and derived concept richness.\n- Promotion re-reads the live daily note before writing to `MEMORY.md`, so edited or deleted short-term snippets do not get promoted from stale recall-store snapshots.\n- Scheduled and manual `memory promote` runs share the same deep phase defaults unless you pass CLI threshold overrides.\n- Automatic runs fan out across configured memory workspaces.\n\nDefault scheduling:\n\n- **Sweep cadence**: `dreaming.frequency = 0 3 * * *`\n- **Deep thresholds**: `minScore=0.8`, `minRecallCount=3`, `minUniqueQueries=3`, `recencyHalfLifeDays=14`, `maxAgeDays=30`\n\nExample:\n\n```\n{\n  \"plugins\": {\n    \"entries\": {\n      \"memory-core\": {\n        \"config\": {\n          \"dreaming\": {\n            \"enabled\": true\n          }\n        }\n      }\n    }\n  }\n}\n```\n\nNotes:\n\n- `memory index --verbose` prints per-phase details (provider, model, sources, batch activity).\n- `memory status` includes any extra paths configured via `memorySearch.extraPaths`.\n- If effectively active memory remote API key fields are configured as SecretRefs, the command resolves those values from the active gateway snapshot. If gateway is unavailable, the command fails fast.\n- Gateway version skew note: this command path requires a gateway that supports `secrets.resolve`; older gateways return an unknown-method error.\n- Tune scheduled sweep cadence with `dreaming.frequency`. Deep promotion policy is otherwise internal; use CLI flags on `memory promote` when you need one-off manual overrides.\n- `memory rem-harness --path <file-or-dir> --grounded` previews grounded `What Happened`, `Reflections`, and `Possible Lasting Updates` from historical daily notes without writing anything.\n- `memory rem-backfill --path <file-or-dir>` writes reversible grounded diary entries into `DREAMS.md` for UI review.\n- `memory rem-backfill --path <file-or-dir> --stage-short-term` also seeds grounded durable candidates into the live short-term promotion store so the normal deep phase can rank them.\n- `memory rem-backfill --rollback` removes previously written grounded diary entries, and `memory rem-backfill --rollback-short-term` removes previously staged grounded short-term candidates.\n- See [Dreaming](https://docs.openclaw.ai/concepts/dreaming) for full phase descriptions and configuration reference.\n\n## [​](https://docs.openclaw.ai/cli/memory\\#related)  Related\n\n- [CLI reference](https://docs.openclaw.ai/cli)\n- [Memory overview](https://docs.openclaw.ai/concepts/memory)\n\n[Inference CLI](https://docs.openclaw.ai/cli/infer) [Message](https://docs.openclaw.ai/cli/message)\n\nCtrl+I

---

## Completion - OpenClaw
**Source:** https://docs.openclaw.ai/cli/completion

[Skip to main content](https://docs.openclaw.ai/cli/completion#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Utility

Completion

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw completion](https://docs.openclaw.ai/cli/completion#openclaw-completion)
- [Usage](https://docs.openclaw.ai/cli/completion#usage)
- [Options](https://docs.openclaw.ai/cli/completion#options)
- [Notes](https://docs.openclaw.ai/cli/completion#notes)
- [Related](https://docs.openclaw.ai/cli/completion#related)

# [​](https://docs.openclaw.ai/cli/completion\\#openclaw-completion)  `openclaw completion`

Generate shell completion scripts and optionally install them into your shell profile.

## [​](https://docs.openclaw.ai/cli/completion\\#usage)  Usage

```
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## [​](https://docs.openclaw.ai/cli/completion\\#options)  Options

- `-s, --shell <shell>`: shell target (`zsh`, `bash`, `powershell`, `fish`; default: `zsh`)
- `-i, --install`: install completion by adding a source line to your shell profile
- `--write-state`: write completion script(s) to `$OPENCLAW_STATE_DIR/completions` without printing to stdout
- `-y, --yes`: skip install confirmation prompts

## [​](https://docs.openclaw.ai/cli/completion\\#notes)  Notes

- `--install` writes a small “OpenClaw Completion” block into your shell profile and points it at the cached script.
- Without `--install` or `--write-state`, the command prints the script to stdout.
- Completion generation eagerly loads command trees so nested subcommands are included.

## [​](https://docs.openclaw.ai/cli/completion\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)

[Clawbot](https://docs.openclaw.ai/cli/clawbot) [DNS](https://docs.openclaw.ai/cli/dns)

Ctrl+I

---

## Browser - OpenClaw
**Source:** https://docs.openclaw.ai/cli/browser

[Skip to main content](https://docs.openclaw.ai/cli/browser#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools and execution

Browser

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw browser](https://docs.openclaw.ai/cli/browser#openclaw-browser)
- [Common flags](https://docs.openclaw.ai/cli/browser#common-flags)
- [Quick start (local)](https://docs.openclaw.ai/cli/browser#quick-start-local)
- [Quick troubleshooting](https://docs.openclaw.ai/cli/browser#quick-troubleshooting)
- [Lifecycle](https://docs.openclaw.ai/cli/browser#lifecycle)
- [If the command is missing](https://docs.openclaw.ai/cli/browser#if-the-command-is-missing)
- [Profiles](https://docs.openclaw.ai/cli/browser#profiles)
- [Tabs](https://docs.openclaw.ai/cli/browser#tabs)
- [Snapshot / screenshot / actions](https://docs.openclaw.ai/cli/browser#snapshot-%2F-screenshot-%2F-actions)
- [State and storage](https://docs.openclaw.ai/cli/browser#state-and-storage)
- [Debugging](https://docs.openclaw.ai/cli/browser#debugging)
- [Existing Chrome via MCP](https://docs.openclaw.ai/cli/browser#existing-chrome-via-mcp)
- [Remote browser control (node host proxy)](https://docs.openclaw.ai/cli/browser#remote-browser-control-node-host-proxy)
- [Related](https://docs.openclaw.ai/cli/browser#related)

# [​](https://docs.openclaw.ai/cli/browser\\#openclaw-browser)  `openclaw browser`

Manage OpenClaw’s browser control surface and run browser actions (lifecycle, profiles, tabs, snapshots, screenshots, navigation, input, state emulation, and debugging).Related:

- Browser tool + API: [Browser tool](https://docs.openclaw.ai/tools/browser)

## [​](https://docs.openclaw.ai/cli/browser\\#common-flags)  Common flags

- `--url <gatewayWsUrl>`: Gateway WebSocket URL (defaults to config).
- `--token <token>`: Gateway token (if required).
- `--timeout <ms>`: request timeout (ms).
- `--expect-final`: wait for a final Gateway response.
- `--browser-profile <name>`: choose a browser profile (default from config).
- `--json`: machine-readable output (where supported).

## [​](https://docs.openclaw.ai/cli/browser\\#quick-start-local)  Quick start (local)

```
openclaw browser profiles
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Agents can run the same readiness check with `browser({ action: \"doctor\" })`.

## [​](https://docs.openclaw.ai/cli/browser\\#quick-troubleshooting)  Quick troubleshooting

If `start` fails with `not reachable after start`, troubleshoot CDP readiness first. If `start` and `tabs` succeed but `open` or `navigate` fails, the browser control plane is healthy and the failure is usually navigation SSRF policy.Minimal sequence:

```
openclaw browser --browser-profile openclaw doctor
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw tabs
openclaw browser --browser-profile openclaw open https://example.com
```

Detailed guidance: [Browser troubleshooting](https://docs.openclaw.ai/tools/browser#cdp-startup-failure-vs-navigation-ssrf-block)

## [​](https://docs.openclaw.ai/cli/browser\\#lifecycle)  Lifecycle

```
openclaw browser status
openclaw browser doctor
openclaw browser doctor --deep
openclaw browser start
openclaw browser start --headless
openclaw browser stop
openclaw browser --browser-profile openclaw reset-profile
```

Notes:

- `doctor --deep` adds a live snapshot probe. It is useful when basic CDP
readiness is green but you want proof that the current tab can be inspected.
- For `attachOnly` and remote CDP profiles, `openclaw browser stop` closes the
active control session and clears temporary emulation overrides even when
OpenClaw did not launch the browser process itself.
- For local managed profiles, `openclaw browser stop` stops the spawned browser
process.
- `openclaw browser start --headless` applies only to that start request and
only when OpenClaw launches a local managed browser. It does not rewrite
`browser.headless` or profile config, and it is a no-op for an already-running
browser.
- On Linux hosts without `DISPLAY` or `WAYLAND_DISPLAY`, local managed profiles
run headless automatically unless `OPENCLAW_BROWSER_HEADLESS=0`,
`browser.headless=false`, or `browser.profiles.<name>.headless=false`
explicitly requests a visible browser.

## [​](https://docs.openclaw.ai/cli/browser\\#if-the-command-is-missing)  If the command is missing

If `openclaw browser` is an unknown command, check `plugins.allow` in
`~/.openclaw/openclaw.json`.When `plugins.allow` is present, list the bundled browser plugin explicitly
unless the config already has a root `browser` block:

```
{
  plugins: {
    allow: [\"telegram\", \"browser\"],
  },
}
```

An explicit root `browser` block, for example `browser.enabled=true` or
`browser.profiles.<name>`, also activates the bundled browser plugin under a
restrictive plugin allowlist.Related: [Browser tool](https://docs.openclaw.ai/tools/browser#missing-browser-command-or-tool)

## [​](https://docs.openclaw.ai/cli/browser\\#profiles)  Profiles

Profiles are named browser routing configs. In practice:

- `openclaw`: launches or attaches to a dedicated OpenClaw-managed Chrome instance (isolated user data dir).
- `user`: controls your existing signed-in Chrome session via Chrome DevTools MCP.
- custom CDP profiles: point at a local or remote CDP endpoint.

```
openclaw browser profiles
openclaw browser create-profile --name work --color \"#FF5A36\"
openclaw browser create-profile --name chrome-live --driver existing-session
openclaw browser create-profile --name remote --cdp-url https://browser-host.example.com
openclaw browser delete-profile --name work
```

Use a specific profile:

```
openclaw browser --browser-profile work tabs
```

## [​](https://docs.openclaw.ai/cli/browser\\#tabs)  Tabs

```
openclaw browser tabs
openclaw browser tab new --label docs
openclaw browser tab label t1 docs
openclaw browser tab select 2
openclaw browser tab close 2
openclaw browser open https://docs.openclaw.ai --label docs
openclaw browser focus docs
openclaw browser close t1
```

`tabs` returns `suggestedTargetId` first, then the stable `tabId` such as `t1`,
the optional label, and the raw `targetId`. Agents should pass
`suggestedTargetId` back into `focus`, `close`, snapshots, and actions. You can
assign a label with `open --label`, `tab new --label`, or `tab label`; labels,
tab ids, raw target ids, and unique target-id prefixes are all accepted.
When Chromium replaces the underlying raw target during a navigation or form
submit, OpenClaw keeps the stable `tabId`/label attached to the replacement tab
when it can prove the match. Raw target ids remain volatile; prefer
`suggestedTargetId`.

## [​](https://docs.openclaw.ai/cli/browser\\#snapshot-/-screenshot-/-actions)  Snapshot / screenshot / actions

Snapshot:

```
openclaw browser snapshot
openclaw browser snapshot --urls
```

Screenshot:

```
openclaw browser screenshot
openclaw browser screenshot --full-page
openclaw browser screenshot --ref e12
openclaw browser screenshot --labels
```

Notes:

- `--full-page` is for page captures only; it cannot be combined with `--ref`
or `--element`.
- `existing-session` / `user` profiles support page screenshots and `--ref`
screenshots from snapshot output, but not CSS `--element` screenshots.
- `--labels` overlays current snapshot refs on the screenshot.
- `snapshot --urls` appends discovered link destinations to AI snapshots so
agents can choose direct navigation targets instead of guessing from link
text alone.

Navigate/click/type (ref-based UI automation):

```
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser click-coords 120 340
openclaw browser type <ref> \"hello\"
openclaw browser press Enter
openclaw browser hover <ref>
openclaw browser scrollintoview <ref>
openclaw browser drag <startRef> <endRef>
openclaw browser select <ref> OptionA OptionB
openclaw browser fill --fields \'[{\"ref\":\"1\",\"value\":\"Ada\"}]\'
openclaw browser wait --text \"Done\"
openclaw browser evaluate --fn \'(el) => el.textContent\' --ref <ref>
```

Action responses return the current raw `targetId` after action-triggered page
replacement when OpenClaw can prove the replacement tab. Scripts should still
store and pass `suggestedTargetId`/labels for long-lived workflows.File + dialog helpers:

```
openclaw browser upload /tmp/openclaw/uploads/file.pdf --ref <ref>
openclaw browser waitfordownload
openclaw browser download <ref> report.pdf
openclaw browser dialog --accept
```

Managed Chrome profiles save ordinary click-triggered downloads into the OpenClaw
downloads directory (`/tmp/openclaw/downloads` by default, or the configured temp
root). Use `waitfordownload` or `download` when the agent needs to wait for a
specific file and return its path; those explicit waiters own the next download.

## [​](https://docs.openclaw.ai/cli/browser\\#state-and-storage)  State and storage

Viewport + emulation:

```
openclaw browser resize 1280 720
openclaw browser set viewport 1280 720
openclaw browser set offline on
openclaw browser set media dark
openclaw browser set timezone Europe/London
openclaw browser set locale en-GB
openclaw browser set geo 51.5074 -0.1278 --accuracy 25
openclaw browser set device \"iPhone 14\"
openclaw browser set headers \'{\"x-test\":\"1\"}\'
openclaw browser set credentials myuser mypass
```

Cookies + storage:

```
openclaw browser cookies
openclaw browser cookies set session abc123 --url https://example.com
openclaw browser cookies clear
openclaw browser storage local get
openclaw browser storage local set token abc123
openclaw browser storage session clear
```

## [​](https://docs.openclaw.ai/cli/browser\\#debugging)  Debugging

```
openclaw browser console --level error
openclaw browser pdf
openclaw browser responsebody \"**/api\"
openclaw browser highlight <ref>
openclaw browser errors --clear
openclaw browser requests --filter api
openclaw browser trace start
openclaw browser trace stop --out trace.zip
```

## [​](https://docs.openclaw.ai/cli/browser\\#existing-chrome-via-mcp)  Existing Chrome via MCP

Use the built-in `user` profile, or create your own `existing-session` profile:

```
openclaw browser --browser-profile user tabs
openclaw browser create-profile --name chrome-live --driver existing-session
openclaw browser create-profile --name brave-live --driver existing-session --user-data-dir \"~/Library/Application Support/BraveSoftware/Brave-Browser\"
openclaw browser --browser-profile chrome-live tabs
```

This path is host-only. For Docker, headless servers, Browserless, or other remote setups, use a CDP profile instead.Current existing-session limits:

- snapshot-driven actions use refs, not CSS selectors
- `browser.actionTimeoutMs` defaults supported `act` requests to 60000 ms when
callers omit `timeoutMs`; per-call `timeoutMs` still wins.
- `click` is left-click only
- `type` does not support `slowly=true`
- `press` does not support `delayMs`
- `hover`, `scrollintoview`, `drag`, `select`, and `fill`, and `evaluate` reject
per-call timeout overrides
- `select` supports one value only
- `wait --load networkidle` is not supported
- file uploads require `--ref` / `--input-ref`, do not support CSS
`--element`, and currently support one file at a time
- dialog hooks do not support `--timeout`
- screenshots support page captures and `--ref`, but not CSS `--element`
- `responsebody`, download interception, PDF export, and batch actions still
require a managed browser or raw CDP profile

## [​](https://docs.openclaw.ai/cli/browser\\#remote-browser-control-node-host-proxy)  Remote browser control (node host proxy)

If the Gateway runs on a different machine than the browser, run a **node host** on the machine that has Chrome/Brave/Edge/Chromium. The Gateway will proxy browser actions to that node (no separate browser control server required).Use `gateway.nodes.browser.mode` to control auto-routing and `gateway.nodes.browser.node` to pin a specific node if multiple are connected.Security + remote setup: [Browser tool](https://docs.openclaw.ai/tools/browser), [Remote access](https://docs.openclaw.ai/gateway/remote), [Tailscale](https://docs.openclaw.ai/gateway/tailscale), [Security](https://docs.openclaw.ai/gateway/security)

## [​](https://docs.openclaw.ai/cli/browser\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Browser](https://docs.openclaw.ai/tools/browser)

[Approvals](https://docs.openclaw.ai/cli/approvals) [Cron](https://docs.openclaw.ai/cli/cron)

Ctrl+I

---

## Models - OpenClaw
**Source:** https://docs.openclaw.ai/cli/models

[Skip to main content](https://docs.openclaw.ai/cli/models#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Agents and sessions

Models

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw models](https://docs.openclaw.ai/cli/models#openclaw-models)
- [Common commands](https://docs.openclaw.ai/cli/models#common-commands)
- [Models scan](https://docs.openclaw.ai/cli/models#models-scan)
- [Models status](https://docs.openclaw.ai/cli/models#models-status)
- [Aliases + fallbacks](https://docs.openclaw.ai/cli/models#aliases-%2B-fallbacks)
- [Auth profiles](https://docs.openclaw.ai/cli/models#auth-profiles)
- [Related](https://docs.openclaw.ai/cli/models#related)

# [​](https://docs.openclaw.ai/cli/models\\#openclaw-models)  `openclaw models`

Model discovery, scanning, and configuration (default model, fallbacks, auth profiles).Related:

- Providers + models: [Models](https://docs.openclaw.ai/providers/models)
- Model selection concepts + `/models` slash command: [Models concept](https://docs.openclaw.ai/concepts/models)
- Provider auth setup: [Getting started](https://docs.openclaw.ai/start/getting-started)

## [​](https://docs.openclaw.ai/cli/models\\#common-commands)  Common commands

```
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` shows the resolved default/fallbacks plus an auth overview.
When provider usage snapshots are available, the OAuth/API-key status section includes
provider usage windows and quota snapshots.
Current usage-window providers: Anthropic, GitHub Copilot, Gemini CLI, OpenAI
Codex, MiniMax, Xiaomi, and z.ai. Usage auth comes from provider-specific hooks
when available; otherwise OpenClaw falls back to matching OAuth/API-key
credentials from auth profiles, env, or config.
In `--json` output, `auth.providers` is the env/config/store-aware provider
overview, while `auth.oauth` is auth-store profile health only.
Add `--probe` to run live auth probes against each configured provider profile.
Probes are real requests (may consume tokens and trigger rate limits).
Use `--agent <id>` to inspect a configured agent’s model/auth state. When omitted,
the command uses `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` if set, otherwise the
configured default agent.
Probe rows can come from auth profiles, env credentials, or `models.json`.Notes:

- `models set <model-or-alias>` accepts `provider/model` or an alias.
- `models list` is read-only: it reads config, auth profiles, existing catalog
state, and provider-owned catalog rows, but it does not rewrite
`models.json`.
- `models list --all --provider <id>` can include provider-owned static catalog
rows from plugin manifests or bundled provider catalog metadata even when you
have not authenticated with that provider yet. Those rows still show as
unavailable until matching auth is configured.
- `models list` keeps native model metadata and runtime caps distinct. In table
output, `Ctx` shows `contextTokens/contextWindow` when an effective runtime
cap differs from the native context window; JSON rows include `contextTokens`
when a provider exposes that cap.
- `models list --provider <id>` filters by provider id, such as `moonshot` or
`openai-codex`. It does not accept display labels from interactive provider
pickers, such as `Moonshot AI`.
- Model refs are parsed by splitting on the **first**`/`. If the model ID includes `/` (OpenRouter-style), include the provider prefix (example: `openrouter/moonshotai/kimi-k2`).
- If you omit the provider, OpenClaw resolves the input as an alias first, then
as a unique configured-provider match for that exact model id, and only then
falls back to the configured default provider with a deprecation warning.
If that provider no longer exposes the configured default model, OpenClaw
falls back to the first configured provider/model instead of surfacing a
stale removed-provider default.
- `models status` may show `marker(<value>)` in auth output for non-secret placeholders (for example `OPENAI_API_KEY`, `secretref-managed`, `minimax-oauth`, `oauth:chutes`, `ollama-local`) instead of masking them as secrets.

### [​](https://docs.openclaw.ai/cli/models\\#models-scan)  Models scan

`models scan` reads OpenRouter’s public `:free` catalog and ranks candidates for
fallback use. The catalog itself is public, so metadata-only scans do not need
an OpenRouter key.By default OpenClaw tries to probe tool and image support with live model calls.
If no OpenRouter key is configured, the command falls back to metadata-only
output and explains that `:free` models still require `OPENROUTER_API_KEY` for
probes and inference.Options:

- `--no-probe` (metadata only; no config/secrets lookup)
- `--min-params <b>`
- `--max-age-days <days>`
- `--provider <name>`
- `--max-candidates <n>`
- `--timeout <ms>` (catalog request and per-probe timeout)
- `--concurrency <n>`
- `--yes`
- `--no-input`
- `--set-default`
- `--set-image`
- `--json`

`--set-default` and `--set-image` require live probes; metadata-only scan
results are informational and are not applied to config.

### [​](https://docs.openclaw.ai/cli/models\\#models-status)  Models status

Options:

- `--json`
- `--plain`
- `--check` (exit 1=expired/missing, 2=expiring)
- `--probe` (live probe of configured auth profiles)
- `--probe-provider <name>` (probe one provider)
- `--probe-profile <id>` (repeat or comma-separated profile ids)
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`
- `--agent <id>` (configured agent id; overrides `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)

`--json` keeps stdout reserved for the JSON payload. Auth-profile, provider,
and startup diagnostics are routed to stderr so scripts can pipe stdout directly
into tools such as `jq`.Probe status buckets:

- `ok`
- `auth`
- `rate_limit`
- `billing`
- `timeout`
- `format`
- `unknown`
- `no_model`

Probe detail/reason-code cases to expect:

- `excluded_by_auth_order`: a stored profile exists, but explicit
`auth.order.<provider>` omitted it, so probe reports the exclusion instead of
trying it.
- `missing_credential`, `invalid_expires`, `expired`, `unresolved_ref`:
profile is present but not eligible/resolvable.
- `no_model`: provider auth exists, but OpenClaw could not resolve a probeable
model candidate for that provider.

## [​](https://docs.openclaw.ai/cli/models\\#aliases-+-fallbacks)  Aliases + fallbacks

```
openclaw models aliases list
openclaw models fallbacks list
```

## [​](https://docs.openclaw.ai/cli/models\\#auth-profiles)  Auth profiles

```
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token --provider <id>
openclaw models auth paste-token
```

`models auth add` is the interactive auth helper. It can launch a provider auth
flow (OAuth/API key) or guide you into manual token paste, depending on the
provider you choose.`models auth login` runs a provider plugin’s auth flow (OAuth/API key). Use
`openclaw plugins list` to see which providers are installed.
Use `openclaw models auth --agent <id> <subcommand>` to write auth results to a
specific configured agent store. The parent `--agent` flag is honored by
`add`, `login`, `setup-token`, `paste-token`, and `login-github-copilot`.Examples:

```
openclaw models auth login --provider openai-codex --set-default
```

Notes:

- `setup-token` and `paste-token` remain generic token commands for providers
that expose token auth methods.
- `setup-token` requires an interactive TTY and runs the provider’s token-auth
method (defaulting to that provider’s `setup-token` method when it exposes
one).
- `paste-token` accepts a token string generated elsewhere or from automation.
- `paste-token` requires `--provider`, prompts for the token value, and writes
it to the default profile id `<provider>:manual` unless you pass
`--profile-id`.
- `paste-token --expires-in <duration>` stores an absolute token expiry from a
relative duration such as `365d` or `12h`.
- Anthropic note: Anthropic staff told us OpenClaw-style Claude CLI usage is allowed again, so OpenClaw treats Claude CLI reuse and `claude -p` usage as sanctioned for this integration unless Anthropic publishes a new policy.
- Anthropic `setup-token` / `paste-token` remain available as a supported OpenClaw token path, but OpenClaw now prefers Claude CLI reuse and `claude -p` when available.

## [​](https://docs.openclaw.ai/cli/models\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Model selection](https://docs.openclaw.ai/concepts/model-providers)
- [Model failover](https://docs.openclaw.ai/concepts/model-failover)

[Message](https://docs.openclaw.ai/cli/message) [Sessions](https://docs.openclaw.ai/cli/sessions)

Ctrl+I

---

## Status - OpenClaw
**Source:** https://docs.openclaw.ai/cli/status

[Skip to main content](https://docs.openclaw.ai/cli/status#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Gateway and service

Status

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw status](https://docs.openclaw.ai/cli/status#openclaw-status)
- [Related](https://docs.openclaw.ai/cli/status#related)

# [​](https://docs.openclaw.ai/cli/status\\#openclaw-status)  `openclaw status`

Diagnostics for channels + sessions.

```
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Notes:

- `--deep` runs live probes (WhatsApp Web + Telegram + Discord + Slack + Signal).
- `--usage` prints normalized provider usage windows as `X% left`.
- Session status output separates `Execution:` from `Runtime:`. `Execution` is the sandbox path (`direct`, `docker/*`), while `Runtime` tells you whether the session is using `OpenClaw Pi Default`, `OpenAI Codex`, a CLI backend, or an ACP backend such as `codex (acp/acpx)`. See [Agent runtimes](https://docs.openclaw.ai/concepts/agent-runtimes) for the provider/model/runtime distinction.
- MiniMax’s raw `usage_percent` / `usagePercent` fields are remaining quota, so OpenClaw inverts them before display; count-based fields win when present. `model_remains` responses prefer the chat-model entry, derive the window label from timestamps when needed, and include the model name in the plan label.
- When the current session snapshot is sparse, `/status` can backfill token and cache counters from the most recent transcript usage log. Existing nonzero live values still win over transcript fallback values.
- Transcript fallback can also recover the active runtime model label when the live session entry is missing it. If that transcript model differs from the selected model, status resolves the context window against the recovered runtime model instead of the selected one.
- For prompt-size accounting, transcript fallback prefers the larger prompt-oriented total when session metadata is missing or smaller, so custom-provider sessions do not collapse to `0` token displays.
- Output includes per-agent session stores when multiple agents are configured.
- Overview includes Gateway + node host service install/runtime status when available.
- Overview includes update channel + git SHA (for source checkouts).
- Update info surfaces in the Overview; if an update is available, status prints a hint to run `openclaw update` (see [Updating](https://docs.openclaw.ai/install/updating)).
- Read-only status surfaces (`status`, `status --json`, `status --all`) resolve supported SecretRefs for their targeted config paths when possible.
- If a supported channel SecretRef is configured but unavailable in the current command path, status stays read-only and reports degraded output instead of crashing. Human output shows warnings such as “configured token unavailable in this command path”, and JSON output includes `secretDiagnostics`.
- When command-local SecretRef resolution succeeds, status prefers the resolved snapshot and clears transient “secret unavailable” channel markers from the final output.
- `status --all` includes a Secrets overview row and a diagnosis section that summarizes secret diagnostics (truncated for readability) without stopping report generation.

## [​](https://docs.openclaw.ai/cli/status\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Doctor](https://docs.openclaw.ai/gateway/doctor)

[Setup](https://docs.openclaw.ai/cli/setup) [Uninstall](https://docs.openclaw.ai/cli/uninstall)

Ctrl+I

---

## Dashboard - OpenClaw
**Source:** https://docs.openclaw.ai/cli/dashboard

[Skip to main content](https://docs.openclaw.ai/cli/dashboard#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Interfaces

Dashboard

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw dashboard](https://docs.openclaw.ai/cli/dashboard#openclaw-dashboard)
- [Related](https://docs.openclaw.ai/cli/dashboard#related)

# [​](https://docs.openclaw.ai/cli/dashboard\\#openclaw-dashboard)  `openclaw dashboard`

Open the Control UI using your current auth.

```
openclaw dashboard
openclaw dashboard --no-open
```

Notes:

- `dashboard` resolves configured `gateway.auth.token` SecretRefs when possible.
- `dashboard` follows `gateway.tls.enabled`: TLS-enabled gateways print/open
`https://` Control UI URLs and connect over `wss://`.
- For SecretRef-managed tokens (resolved or unresolved), `dashboard` prints/copies/opens a non-tokenized URL to avoid exposing external secrets in terminal output, clipboard history, or browser-launch arguments.
- If `gateway.auth.token` is SecretRef-managed but unresolved in this command path, the command prints a non-tokenized URL and explicit remediation guidance instead of embedding an invalid token placeholder.

## [​](https://docs.openclaw.ai/cli/dashboard\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Dashboard](https://docs.openclaw.ai/web/dashboard)

[Skills](https://docs.openclaw.ai/cli/skills) [TUI](https://docs.openclaw.ai/cli/tui)

Ctrl+I

---

## Update - OpenClaw
**Source:** https://docs.openclaw.ai/cli/update

[Skip to main content](https://docs.openclaw.ai/cli/update#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Gateway and service

Update

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw update](https://docs.openclaw.ai/cli/update#openclaw-update)
- [Usage](https://docs.openclaw.ai/cli/update#usage)
- [Options](https://docs.openclaw.ai/cli/update#options)
- [update status](https://docs.openclaw.ai/cli/update#update-status)
- [update wizard](https://docs.openclaw.ai/cli/update#update-wizard)
- [What it does](https://docs.openclaw.ai/cli/update#what-it-does)
- [Git checkout flow](https://docs.openclaw.ai/cli/update#git-checkout-flow)
- [Channel selection](https://docs.openclaw.ai/cli/update#channel-selection)
- [Update steps](https://docs.openclaw.ai/cli/update#update-steps)
- [--update shorthand](https://docs.openclaw.ai/cli/update#update-shorthand)
- [Related](https://docs.openclaw.ai/cli/update#related)

# [​](https://docs.openclaw.ai/cli/update\\#openclaw-update)  `openclaw update`

Safely update OpenClaw and switch between stable/beta/dev channels.If you installed via **npm/pnpm/bun** (global install, no git metadata),
updates happen via the package-manager flow in [Updating](https://docs.openclaw.ai/install/updating).

## [​](https://docs.openclaw.ai/cli/update\\#usage)  Usage

```
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --tag main
openclaw update --dry-run
openclaw update --no-restart
openclaw update --yes
openclaw update --json
openclaw --update
```

## [​](https://docs.openclaw.ai/cli/update\\#options)  Options

- `--no-restart`: skip restarting the Gateway service after a successful update. Package-manager updates that do restart the Gateway verify the restarted service reports the expected updated version before the command succeeds.
- `--channel <stable|beta|dev>`: set the update channel (git + npm; persisted in config).
- `--tag <dist-tag|version|spec>`: override the package target for this update only. For package installs, `main` maps to `github:openclaw/openclaw#main`.
- `--dry-run`: preview planned update actions (channel/tag/target/restart flow) without writing config, installing, syncing plugins, or restarting.
- `--json`: print machine-readable `UpdateRunResult` JSON, including
`postUpdate.plugins.integrityDrifts` when npm plugin artifact drift is
detected during post-update plugin sync.
- `--timeout <seconds>`: per-step timeout (default is 1800s).
- `--yes`: skip confirmation prompts (for example downgrade confirmation).

Downgrades require confirmation because older versions can break configuration.

## [​](https://docs.openclaw.ai/cli/update\\#update-status)  `update status`

Show the active update channel + git tag/branch/SHA (for source checkouts), plus update availability.

```
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Options:

- `--json`: print machine-readable status JSON.
- `--timeout <seconds>`: timeout for checks (default is 3s).

## [​](https://docs.openclaw.ai/cli/update\\#update-wizard)  `update wizard`

Interactive flow to pick an update channel and confirm whether to restart the Gateway
after updating (default is to restart). If you select `dev` without a git checkout, it
offers to create one.Options:

- `--timeout <seconds>`: timeout for each update step (default `1800`)

## [​](https://docs.openclaw.ai/cli/update\\#what-it-does)  What it does

When you switch channels explicitly (`--channel ...`), OpenClaw also keeps the
install method aligned:

- `dev` → ensures a git checkout (default: `~/openclaw`, override with `OPENCLAW_GIT_DIR`),
updates it, and installs the global CLI from that checkout.
- `stable` → installs from npm using `latest`.
- `beta` → prefers npm dist-tag `beta`, but falls back to `latest` when beta is
missing or older than the current stable release.

The Gateway core auto-updater (when enabled via config) reuses this same update path.For package-manager installs, `openclaw update` resolves the target package
version before invoking the package manager. npm global installs use a staged
install: OpenClaw installs the new package into a temporary npm prefix, verifies
the packaged `dist` inventory there, then swaps that clean package tree into the
real global prefix. If verification fails, post-update doctor, plugin sync, and
restart work do not run from the suspect tree. Even when the installed version
already matches the target, the command refreshes the global package install,
then runs plugin sync, a core-command completion refresh, and restart work. This
keeps packaged sidecars and channel-owned plugin records aligned with the
installed OpenClaw build while leaving full plugin-command completion rebuilds to
explicit `openclaw completion --write-state` runs.When a local managed Gateway service is installed and restart is enabled,
package-manager updates stop the running service before replacing the package
tree, then refresh the service metadata from the updated install, restart the
service, and verify the restarted Gateway reports the expected version. With
`--no-restart`, package replacement still runs but the managed service is not
stopped or restarted, so the running Gateway may keep old code until you restart
it manually.

## [​](https://docs.openclaw.ai/cli/update\\#git-checkout-flow)  Git checkout flow

### [​](https://docs.openclaw.ai/cli/update\\#channel-selection)  Channel selection

- `stable`: checkout the latest non-beta tag, then build and doctor.
- `beta`: prefer the latest `-beta` tag, but fall back to the latest stable tag when beta is missing or older.
- `dev`: checkout `main`, then fetch and rebase.

### [​](https://docs.openclaw.ai/cli/update\\#update-steps)  Update steps

1

[Navigate to header](https://docs.openclaw.ai/cli/update#)

Verify clean worktree

Requires no uncommitted changes.

2

[Navigate to header](https://docs.openclaw.ai/cli/update#)

Switch channel

Switches to the selected channel (tag or branch).

3

[Navigate to header](https://docs.openclaw.ai/cli/update#)

Fetch upstream

Dev only.

4

[Navigate to header](https://docs.openclaw.ai/cli/update#)

Preflight build (dev only)

Runs lint and TypeScript build in a temp worktree. If the tip fails, walks back up to 10 commits to find the newest clean build.

5

[Navigate to header](https://docs.openclaw.ai/cli/update#)

Rebase

Rebases onto the selected commit (dev only).

6

[Navigate to header](https://docs.openclaw.ai/cli/update#)

Install dependencies

Uses the repo package manager. For pnpm checkouts, the updater bootstraps `pnpm` on demand (via `corepack` first, then a temporary `npm install pnpm@10` fallback) instead of running `npm run build` inside a pnpm workspace.

7

[Navigate to header](https://docs.openclaw.ai/cli/update#)

Build Control UI

Builds the gateway and the Control UI.

8

[Navigate to header](https://docs.openclaw.ai/cli/update#)

Run doctor

`openclaw doctor` runs as the final safe-update check.

9

[Navigate to header](https://docs.openclaw.ai/cli/update#)

Sync plugins

Syncs plugins to the active channel. Dev uses bundled plugins; stable and beta use npm. Updates npm-installed plugins.

If an exact pinned npm plugin update resolves to an artifact whose integrity differs from the stored install record, `openclaw update` aborts that plugin artifact update instead of installing it. Reinstall or update the plugin explicitly only after verifying that you trust the new artifact.

Post-update plugin sync failures fail the update result and stop restart follow-up work. Fix the plugin install or update error, then rerun `openclaw update`.When the updated Gateway starts, enabled bundled plugin runtime dependencies are staged before plugin activation. Update-triggered restarts drain any active runtime-dependency staging before closing the Gateway, so service-manager restarts do not interrupt an in-flight npm install.If pnpm bootstrap still fails, the updater stops early with a package-manager-specific error instead of trying `npm run build` inside the checkout.

## [​](https://docs.openclaw.ai/cli/update\\#update-shorthand)  `--update` shorthand

`openclaw --update` rewrites to `openclaw update` (useful for shells and launcher scripts).

## [​](https://docs.openclaw.ai/cli/update\\#related)  Related

- `openclaw doctor` (offers to run update first on git checkouts)
- [Development channels](https://docs.openclaw.ai/install/development-channels)
- [Updating](https://docs.openclaw.ai/install/updating)
- [CLI reference](https://docs.openclaw.ai/cli)

[Uninstall](https://docs.openclaw.ai/cli/uninstall) [Agent](https://docs.openclaw.ai/cli/agent)

Ctrl+I

---

## Logs - OpenClaw
**Source:** https://docs.openclaw.ai/cli/logs

[Skip to main content](https://docs.openclaw.ai/cli/logs#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Gateway and service

Logs

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw logs](https://docs.openclaw.ai/cli/logs#openclaw-logs)
- [Options](https://docs.openclaw.ai/cli/logs#options)
- [Shared Gateway RPC options](https://docs.openclaw.ai/cli/logs#shared-gateway-rpc-options)
- [Examples](https://docs.openclaw.ai/cli/logs#examples)
- [Notes](https://docs.openclaw.ai/cli/logs#notes)
- [Related](https://docs.openclaw.ai/cli/logs#related)

# [​](https://docs.openclaw.ai/cli/logs\\#openclaw-logs)  `openclaw logs`

Tail Gateway file logs over RPC (works in remote mode).Related:

- Logging overview: [Logging](https://docs.openclaw.ai/logging)
- Gateway CLI: [gateway](https://docs.openclaw.ai/cli/gateway)

## [​](https://docs.openclaw.ai/cli/logs\\#options)  Options

- `--limit <n>`: maximum number of log lines to return (default `200`)
- `--max-bytes <n>`: maximum bytes to read from the log file (default `250000`)
- `--follow`: follow the log stream
- `--interval <ms>`: polling interval while following (default `1000`)
- `--json`: emit line-delimited JSON events
- `--plain`: plain text output without styled formatting
- `--no-color`: disable ANSI colors
- `--local-time`: render timestamps in your local timezone

## [​](https://docs.openclaw.ai/cli/logs\\#shared-gateway-rpc-options)  Shared Gateway RPC options

`openclaw logs` also accepts the standard Gateway client flags:

- `--url <url>`: Gateway WebSocket URL
- `--token <token>`: Gateway token
- `--timeout <ms>`: timeout in ms (default `30000`)
- `--expect-final`: wait for a final response when the Gateway call is agent-backed

When you pass `--url`, the CLI does not auto-apply config or environment credentials. Include `--token` explicitly if the target Gateway requires auth.

## [​](https://docs.openclaw.ai/cli/logs\\#examples)  Examples

```
openclaw logs
openclaw logs --follow
openclaw logs --follow --interval 2000
openclaw logs --limit 500 --max-bytes 500000
openclaw logs --json
openclaw logs --plain
openclaw logs --no-color
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
openclaw logs --url ws://127.0.0.1:18789 --token \"$OPENCLAW_GATEWAY_TOKEN\"
```

## [​](https://docs.openclaw.ai/cli/logs\\#notes)  Notes

- Use `--local-time` to render timestamps in your local timezone.
- If the local loopback Gateway asks for pairing, `openclaw logs` falls back to the configured local log file automatically. Explicit `--url` targets do not use this fallback.

## [​](https://docs.openclaw.ai/cli/logs\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Gateway logging](https://docs.openclaw.ai/gateway/logging)

[Health](https://docs.openclaw.ai/cli/health) [Migrate](https://docs.openclaw.ai/cli/migrate)

Ctrl+I

---

## Wiki - OpenClaw
**Source:** https://docs.openclaw.ai/cli/wiki

[Skip to main content](https://docs.openclaw.ai/cli/wiki#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Utility

Wiki

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw wiki](https://docs.openclaw.ai/cli/wiki#openclaw-wiki)
- [What it is for](https://docs.openclaw.ai/cli/wiki#what-it-is-for)
- [Common commands](https://docs.openclaw.ai/cli/wiki#common-commands)
- [Commands](https://docs.openclaw.ai/cli/wiki#commands)
- [wiki status](https://docs.openclaw.ai/cli/wiki#wiki-status)
- [wiki doctor](https://docs.openclaw.ai/cli/wiki#wiki-doctor)
- [wiki init](https://docs.openclaw.ai/cli/wiki#wiki-init)
- [wiki ingest <path-or-url>](https://docs.openclaw.ai/cli/wiki#wiki-ingest-%3Cpath-or-url%3E)
- [wiki compile](https://docs.openclaw.ai/cli/wiki#wiki-compile)
- [wiki lint](https://docs.openclaw.ai/cli/wiki#wiki-lint)
- [wiki search <query>](https://docs.openclaw.ai/cli/wiki#wiki-search-%3Cquery%3E)
- [wiki get <lookup>](https://docs.openclaw.ai/cli/wiki#wiki-get-%3Clookup%3E)
- [wiki apply](https://docs.openclaw.ai/cli/wiki#wiki-apply)
- [wiki bridge import](https://docs.openclaw.ai/cli/wiki#wiki-bridge-import)
- [wiki unsafe-local import](https://docs.openclaw.ai/cli/wiki#wiki-unsafe-local-import)
- [wiki obsidian ...](https://docs.openclaw.ai/cli/wiki#wiki-obsidian)
- [Practical usage guidance](https://docs.openclaw.ai/cli/wiki#practical-usage-guidance)
- [Configuration tie-ins](https://docs.openclaw.ai/cli/wiki#configuration-tie-ins)
- [Related](https://docs.openclaw.ai/cli/wiki#related)

# [​](https://docs.openclaw.ai/cli/wiki\#openclaw-wiki)  `openclaw wiki`

Inspect and maintain the `memory-wiki` vault.Provided by the bundled `memory-wiki` plugin.Related:

- [Memory Wiki plugin](https://docs.openclaw.ai/plugins/memory-wiki)
- [Memory Overview](https://docs.openclaw.ai/concepts/memory)
- [CLI: memory](https://docs.openclaw.ai/cli/memory)

## [​](https://docs.openclaw.ai/cli/wiki\#what-it-is-for)  What it is for

Use `openclaw wiki` when you want a compiled knowledge vault with:

- wiki-native search and page reads
- provenance-rich syntheses
- contradiction and freshness reports
- bridge imports from the active memory plugin
- optional Obsidian CLI helpers

## [​](https://docs.openclaw.ai/cli/wiki\#common-commands)  Common commands

```
openclaw wiki status
openclaw wiki doctor
openclaw wiki init
openclaw wiki ingest ./notes/alpha.md
openclaw wiki compile
openclaw wiki lint
openclaw wiki search \"alpha\"
openclaw wiki get entity.alpha --from 1 --lines 80

openclaw wiki apply synthesis \"Alpha Summary\" \\
  --body \"Short synthesis body\" \\
  --source-id source.alpha

openclaw wiki apply metadata entity.alpha \\
  --source-id source.alpha \\
  --status review \\
  --question \"Still active?\"

openclaw wiki bridge import
openclaw wiki unsafe-local import

openclaw wiki obsidian status
openclaw wiki obsidian search \"alpha\"
openclaw wiki obsidian open syntheses/alpha-summary.md
openclaw wiki obsidian command workspace:quick-switcher
openclaw wiki obsidian daily
```

## [​](https://docs.openclaw.ai/cli/wiki\#commands)  Commands

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-status)  `wiki status`

Inspect current vault mode, health, and Obsidian CLI availability.Use this first when you are unsure whether the vault is initialized, bridge mode\nis healthy, or Obsidian integration is available.

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-doctor)  `wiki doctor`

Run wiki health checks and surface configuration or vault problems.Typical issues include:

- bridge mode enabled without public memory artifacts
- invalid or missing vault layout
- missing external Obsidian CLI when Obsidian mode is expected

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-init)  `wiki init`

Create the wiki vault layout and starter pages.This initializes the root structure, including top-level indexes and cache\ndirectories.

### [​](https://openclaw.ai/cli/wiki\#wiki-ingest-%3Cpath-or-url%3E)  `wiki ingest <path-or-url>`

Import content into the wiki source layer.Notes:

- URL ingest is controlled by `ingest.allowUrlIngest`
- imported source pages keep provenance in frontmatter
- auto-compile can run after ingest when enabled

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-compile)  `wiki compile`

Rebuild indexes, related blocks, dashboards, and compiled digests.This writes stable machine-facing artifacts under:

- `.openclaw-wiki/cache/agent-digest.json`
- `.openclaw-wiki/cache/claims.jsonl`

If `render.createDashboards` is enabled, compile also refreshes report pages.

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-lint)  `wiki lint`

Lint the vault and report:

- structural issues
- provenance gaps
- contradictions
- open questions
- low-confidence pages/claims
- stale pages/claims

Run this after meaningful wiki updates.

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-search-%3Cquery%3E)  `wiki search <query>`

Search wiki content.Behavior depends on config:

- `search.backend`: `shared` or `local`
- `search.corpus`: `wiki`, `memory`, or `all`

Use `wiki search` when you want wiki-specific ranking or provenance details.\nFor one broad shared recall pass, prefer `openclaw memory search` when the\nactive memory plugin exposes shared search.

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-get-%3Clookup%3E)  `wiki get <lookup>`

Read a wiki page by id or relative path.Examples:

```
openclaw wiki get entity.alpha
openclaw wiki get syntheses/alpha-summary.md --from 1 --lines 80
```

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-apply)  `wiki apply`

Apply narrow mutations without freeform page surgery.Supported flows include:

- create/update a synthesis page
- update page metadata
- attach source ids
- add questions
- add contradictions
- update confidence/status
- write structured claims

This command exists so the wiki can evolve safely without manually editing\nmanaged blocks.

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-bridge-import)  `wiki bridge import`

Import public memory artifacts from the active memory plugin into bridge-backed\nsource pages.Use this in `bridge` mode when you want the latest exported memory artifacts\npulled into the wiki vault.

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-unsafe-local-import)  `wiki unsafe-local import`

Import from explicitly configured local paths in `unsafe-local` mode.This is intentionally experimental and same-machine only.

### [​](https://docs.openclaw.ai/cli/wiki\#wiki-obsidian)  `wiki obsidian ...`

Obsidian helper commands for vaults running in Obsidian-friendly mode.Subcommands:

- `status`
- `search`
- `open`
- `command`
- `daily`

These require the official `obsidian` CLI on `PATH` when\n`obsidian.useOfficialCli` is enabled.

## [​](https://docs.openclaw.ai/cli/wiki\#practical-usage-guidance)  Practical usage guidance

- Use `wiki search` \\+ `wiki get` when provenance and page identity matter.
- Use `wiki apply` instead of hand-editing managed generated sections.
- Use `wiki lint` before trusting contradictory or low-confidence content.
- Use `wiki compile` after bulk imports or source changes when you want fresh\ndashboards and compiled digests immediately.
- Use `wiki bridge import` when bridge mode depends on newly exported memory\nartifacts.

## [​](https://docs.openclaw.ai/cli/wiki\#configuration-tie-ins)  Configuration tie-ins

`openclaw wiki` behavior is shaped by:

- `plugins.entries.memory-wiki.config.vaultMode`
- `plugins.entries.memory-wiki.config.search.backend`
- `plugins.entries.memory-wiki.config.search.corpus`
- `plugins.entries.memory-wiki.config.bridge.*`
- `plugins.entries.memory-wiki.config.obsidian.*`
- `plugins.entries.memory-wiki.config.render.*`
- `plugins.entries.memory-wiki.config.context.includeCompiledDigestPrompt`

See [Memory Wiki plugin](https://docs.openclaw.ai/plugins/memory-wiki) for the full config model.

## [​](https://docs.openclaw.ai/cli/wiki\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Memory wiki](https://docs.openclaw.ai/plugins/memory-wiki)

[Proxy](https://docs.openclaw.ai/cli/proxy) [RPC adapters](https://docs.openclaw.ai/reference/rpc)

Ctrl+I

---

## Backup - OpenClaw
**Source:** https://docs.openclaw.ai/cli/backup

[Skip to main content](https://docs.openclaw.ai/cli/backup#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Gateway and service

Backup

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw backup](https://docs.openclaw.ai/cli/backup#openclaw-backup)
- [Notes](https://docs.openclaw.ai/cli/backup#notes)
- [What gets backed up](https://docs.openclaw.ai/cli/backup#what-gets-backed-up)
- [Invalid config behavior](https://docs.openclaw.ai/cli/backup#invalid-config-behavior)
- [Size and performance](https://docs.openclaw.ai/cli/backup#size-and-performance)
- [Related](https://docs.openclaw.ai/cli/backup#related)

# [​](https://docs.openclaw.ai/cli/backup\\#openclaw-backup)  `openclaw backup`

Create a local backup archive for OpenClaw state, config, auth profiles, channel/provider credentials, sessions, and optionally workspaces.

```
openclaw backup create
openclaw backup create --output ~/Backups
openclaw backup create --dry-run --json
openclaw backup create --verify
openclaw backup create --no-include-workspace
openclaw backup create --only-config
openclaw backup verify ./2026-03-09T00-00-00.000Z-openclaw-backup.tar.gz
```

## [​](https://docs.openclaw.ai/cli/backup\\#notes)  Notes

- The archive includes a `manifest.json` file with the resolved source paths and archive layout.
- Default output is a timestamped `.tar.gz` archive in the current working directory.
- If the current working directory is inside a backed-up source tree, OpenClaw falls back to your home directory for the default archive location.
- Existing archive files are never overwritten.
- Output paths inside the source state/workspace trees are rejected to avoid self-inclusion.
- `openclaw backup verify <archive>` validates that the archive contains exactly one root manifest, rejects traversal-style archive paths, and checks that every manifest-declared payload exists in the tarball.
- `openclaw backup create --verify` runs that validation immediately after writing the archive.
- `openclaw backup create --only-config` backs up just the active JSON config file.

## [​](https://docs.openclaw.ai/cli/backup\\#what-gets-backed-up)  What gets backed up

`openclaw backup create` plans backup sources from your local OpenClaw install:

- The state directory returned by OpenClaw’s local state resolver, usually `~/.openclaw`
- The active config file path
- The resolved `credentials/` directory when it exists outside the state directory
- Workspace directories discovered from the current config, unless you pass `--no-include-workspace`

Model auth profiles are already part of the state directory under
`agents/<agentId>/agent/auth-profiles.json`, so they are normally covered by the
state backup entry.If you use `--only-config`, OpenClaw skips state, credentials-directory, and workspace discovery and archives only the active config file path.OpenClaw canonicalizes paths before building the archive. If config, the
credentials directory, or a workspace already live inside the state directory,
they are not duplicated as separate top-level backup sources. Missing paths are
skipped.The archive payload stores file contents from those source trees, and the embedded `manifest.json` records the resolved absolute source paths plus the archive layout used for each asset.

## [​](https://docs.openclaw.ai/cli/backup\\#invalid-config-behavior)  Invalid config behavior

`openclaw backup` intentionally bypasses the normal config preflight so it can still help during recovery. Because workspace discovery depends on a valid config, `openclaw backup create` now fails fast when the config file exists but is invalid and workspace backup is still enabled.If you still want a partial backup in that situation, rerun:

```
openclaw backup create --no-include-workspace
```

That keeps state, config, and the external credentials directory in scope while
skipping workspace discovery entirely.If you only need a copy of the config file itself, `--only-config` also works when the config is malformed because it does not rely on parsing the config for workspace discovery.

## [​](https://docs.openclaw.ai/cli/backup\\#size-and-performance)  Size and performance

OpenClaw does not enforce a built-in maximum backup size or per-file size limit.Practical limits come from the local machine and destination filesystem:

- Available space for the temporary archive write plus the final archive
- Time to walk large workspace trees and compress them into a `.tar.gz`
- Time to rescan the archive if you use `openclaw backup create --verify` or run `openclaw backup verify`
- Filesystem behavior at the destination path. OpenClaw prefers a no-overwrite hard-link publish step and falls back to exclusive copy when hard links are unsupported

Large workspaces are usually the main driver of archive size. If you want a smaller or faster backup, use `--no-include-workspace`.For the smallest archive, use `--only-config`.

## [​](https://docs.openclaw.ai/cli/backup\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)

[CLI reference](https://docs.openclaw.ai/cli) [Daemon](https://docs.openclaw.ai/cli/daemon)

Ctrl+I

---

## Inference CLI - OpenClaw
**Source:** https://docs.openclaw.ai/cli/infer

[Skip to main content](https://docs.openclaw.ai/cli/infer#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Agents and sessions

Inference CLI

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Turn infer into a skill](https://docs.openclaw.ai/cli/infer#turn-infer-into-a-skill)
- [Why use infer](https://docs.openclaw.ai/cli/infer#why-use-infer)
- [Command tree](https://docs.openclaw.ai/cli/infer#command-tree)
- [Common tasks](https://docs.openclaw.ai/cli/infer#common-tasks)
- [Behavior](https://docs.openclaw.ai/cli/infer#behavior)
- [Model](https://docs.openclaw.ai/cli/infer#model)
- [Image](https://docs.openclaw.ai/cli/infer#image)
- [Audio](https://docs.openclaw.ai/cli/infer#audio)
- [TTS](https://docs.openclaw.ai/cli/infer#tts)
- [Video](https://docs.openclaw.ai/cli/infer#video)
- [Web](https://docs.openclaw.ai/cli/infer#web)
- [Embedding](https://docs.openclaw.ai/cli/infer#embedding)
- [JSON output](https://docs.openclaw.ai/cli/infer#json-output)
- [Common pitfalls](https://docs.openclaw.ai/cli/infer#common-pitfalls)
- [Notes](https://docs.openclaw.ai/cli/infer#notes)
- [Related](https://docs.openclaw.ai/cli/infer#related)

`openclaw infer` is the canonical headless surface for provider-backed inference workflows.It intentionally exposes capability families, not raw gateway RPC names and not raw agent tool ids.

## [​](https://docs.openclaw.ai/cli/infer\\#turn-infer-into-a-skill)  Turn infer into a skill

Copy and paste this to an agent:

```
Read https://docs.openclaw.ai/cli/infer, then create a skill that routes my common workflows to `openclaw infer`.
Focus on model runs, image generation, video generation, audio transcription, TTS, web search, and embeddings.
```

A good infer-based skill should:

- map common user intents to the correct infer subcommand
- include a few canonical infer examples for the workflows it covers
- prefer `openclaw infer ...` in examples and suggestions
- avoid re-documenting the entire infer surface inside the skill body

Typical infer-focused skill coverage:

- `openclaw infer model run`
- `openclaw infer image generate`
- `openclaw infer audio transcribe`
- `openclaw infer tts convert`
- `openclaw infer web search`
- `openclaw infer embedding create`

## [​](https://docs.openclaw.ai/cli/infer\\#why-use-infer)  Why use infer

`openclaw infer` provides one consistent CLI for provider-backed inference tasks inside OpenClaw.Benefits:

- Use the providers and models already configured in OpenClaw instead of wiring up one-off wrappers for each backend.
- Keep model, image, audio transcription, TTS, video, web, and embedding workflows under one command tree.
- Use a stable `--json` output shape for scripts, automation, and agent-driven workflows.
- Prefer a first-party OpenClaw surface when the task is fundamentally “run inference.”
- Use the normal local path without requiring the gateway for most infer commands.

For end-to-end provider checks, prefer `openclaw infer ...` once lower-level
provider tests are green. It exercises the shipped CLI, config loading,
default-agent resolution, bundled plugin activation, runtime-dependency repair,
and the shared capability runtime before the provider request is made.

## [​](https://docs.openclaw.ai/cli/infer\\#command-tree)  Command tree

```
 openclaw infer
  list
  inspect

  model
    run
    list
    inspect
    providers
    auth login
    auth logout
    auth status

  image
    generate
    edit
    describe
    describe-many
    providers

  audio
    transcribe
    providers

  tts
    convert
    voices
    providers
    status
    enable
    disable
    set-provider

  video
    generate
    describe
    providers

  web
    search
    fetch
    providers

  embedding
    create
    providers
```

## [​](https://docs.openclaw.ai/cli/infer\\#common-tasks)  Common tasks

This table maps common inference tasks to the corresponding infer command.

| Task | Command | Notes |
| --- | --- | --- |
| Run a text/model prompt | `openclaw infer model run --prompt \"...\" --json` | Uses the normal local path by default |
| Generate an image | `openclaw infer image generate --prompt \"...\" --json` | Use `image edit` when starting from an existing file |
| Describe an image file | `openclaw infer image describe --file ./image.png --json` | `--model` must be an image-capable `<provider/model>` |
| Transcribe audio | `openclaw infer audio transcribe --file ./memo.m4a --json` | `--model` must be `<provider/model>` |
| Synthesize speech | `openclaw infer tts convert --text \"...\" --output ./speech.mp3 --json` | `tts status` is gateway-oriented |
| Generate a video | `openclaw infer video generate --prompt \"...\" --json` | Supports provider hints such as `--resolution` |
| Describe a video file | `openclaw infer video describe --file ./clip.mp4 --json` | `--model` must be `<provider/model>` |
| Search the web | `openclaw infer web search --query \"...\" --json` |  |
| Fetch a web page | `openclaw infer web fetch --url https://example.com --json` |  |
| Create embeddings | `openclaw infer embedding create --text \"...\" --json` |  |

## [​](https://docs.openclaw.ai/cli/infer\\#behavior)  Behavior

- `openclaw infer ...` is the primary CLI surface for these workflows.
- Use `--json` when the output will be consumed by another command or script.
- Use `--provider` or `--model provider/model` when a specific backend is required.
- For `image describe`, `audio transcribe`, and `video describe`, `--model` must use the form `<provider/model>`.
- For `image describe`, an explicit `--model` runs that provider/model directly. The model must be image-capable in the model catalog or provider config. `codex/<model>` runs a bounded Codex app-server image-understanding turn; `openai-codex/<model>` uses the OpenAI Codex OAuth provider path.
- Stateless execution commands default to local.
- Gateway-managed state commands default to gateway.
- The normal local path does not require the gateway to be running.
- `model run` is one-shot. MCP servers opened through the agent runtime for that command are retired after the reply for both local and `--gateway` execution, so repeated scripted invocations do not keep stdio MCP child processes alive.

## [​](https://docs.openclaw.ai/cli/infer\\#model)  Model

Use `model` for provider-backed text inference and model/provider inspection.

```
openclaw infer model run --prompt \"Reply with exactly: smoke-ok\" --json
openclaw infer model run --prompt \"Summarize this changelog entry\" --provider openai --json
openclaw infer model providers --json
openclaw infer model inspect --name gpt-5.5 --json
```

Notes:

- `model run` reuses the agent runtime so provider/model overrides behave like normal agent execution.
- Because `model run` is intended for headless automation, it does not retain per-session bundled MCP runtimes after the command finishes.
- `model auth login`, `model auth logout`, and `model auth status` manage saved provider auth state.

## [​](https://docs.openclaw.ai/cli/infer\\#image)  Image

Use `image` for generation, edit, and description.

```
openclaw infer image generate --prompt \"friendly lobster illustration\" --json
openclaw infer image generate --prompt \"cinematic product photo of headphones\" --json
openclaw infer image generate --model openai/gpt-image-1.5 --output-format png --background transparent --prompt \"simple red circle sticker on a transparent background\" --json
openclaw infer image generate --prompt \"slow image backend\" --timeout-ms 180000 --json
openclaw infer image edit --file ./logo.png --model openai/gpt-image-1.5 --output-format png --background transparent --prompt \"keep the logo, remove the background\" --json
openclaw infer image edit --file ./poster.png --prompt \"make this a vertical story ad\" --size 2160x3840 --aspect-ratio 9:16 --resolution 4K --json
openclaw infer image describe --file ./photo.jpg --json
openclaw infer image describe --file ./ui-screenshot.png --model openai/gpt-4.1-mini --json
openclaw infer image describe --file ./photo.jpg --model ollama/qwen2.5vl:7b --json
```

Notes:

- Use `image edit` when starting from existing input files.
- Use `--size`, `--aspect-ratio`, or `--resolution` with `image edit` for
providers/models that support geometry hints on reference-image edits.
- Use `--output-format png --background transparent` with
`--model openai/gpt-image-1.5` for transparent-background OpenAI PNG output;
`--openai-background` remains available as an OpenAI-specific alias. Providers
that do not declare background support report the hint as an ignored override.
- Use `image providers --json` to verify which bundled image providers are
discoverable, configured, selected, and which generation/edit capabilities
each provider exposes.
- Use `image generate --model <provider/model> --json` as the narrowest live
CLI smoke for image generation changes. Example:














```
openclaw infer image providers --json
openclaw infer image generate \\\
    --model google/gemini-3.1-flash-image-preview \\\
    --prompt \"Minimal flat test image: one blue square on a white background, no text.\" \\\
    --output ./openclaw-infer-image-smoke.png \\\
    --json
```










The JSON response reports `ok`, `provider`, `model`, `attempts`, and written
output paths. When `--output` is set, the final extension may follow the
provider’s returned MIME type.
- For `image describe`, `--model` must be an image-capable `<provider/model>`.
- For local Ollama vision models, pull the model first and set `OLLAMA_API_KEY` to any placeholder value, for example `ollama-local`. See [Ollama](https://docs.openclaw.ai/providers/ollama#vision-and-image-description).

## [​](https://docs.openclaw.ai/cli/infer\\#audio)  Audio

Use `audio` for file transcription.

```
openclaw infer audio transcribe --file ./memo.m4a --json
openclaw infer audio transcribe --file ./team-sync.m4a --language en --prompt \"Focus on names and action items\" --json
openclaw infer audio transcribe --file ./memo.m4a --model openai/whisper-1 --json
```

Notes:

- `audio transcribe` is for file transcription, not realtime session management.
- `--model` must be `<provider/model>`.

## [​](https://docs.openclaw.ai/cli/infer\\#tts)  TTS

Use `tts` for speech synthesis and TTS provider state.

```
openclaw infer tts convert --text \"hello from openclaw\" --output ./hello.mp3 --json
openclaw infer tts convert --text \"Your build is complete\" --output ./build-complete.mp3 --json
openclaw infer tts providers --json
openclaw infer tts status --json
```

Notes:

- `tts status` defaults to gateway because it reflects gateway-managed TTS state.
- Use `tts providers`, `tts voices`, and `tts set-provider` to inspect and configure TTS behavior.

## [​](https://docs.openclaw.ai/cli/infer\\#video)  Video

Use `video` for generation and description.

```
openclaw infer video generate --prompt \"cinematic sunset over the ocean\" --json
openclaw infer video generate --prompt \"slow drone shot over a forest lake\" --resolution 768P --duration 6 --json
openclaw infer video describe --file ./clip.mp4 --json
openclaw infer video describe --file ./clip.mp4 --model openai/gpt-4.1-mini --json
```

Notes:

- `video generate` accepts `--size`, `--aspect-ratio`, `--resolution`, `--duration`, `--audio`, `--watermark`, and `--timeout-ms` and forwards them to the video-generation runtime.
- `--model` must be `<provider/model>` for `video describe`.

## [​](https://docs.openclaw.ai/cli/infer\\#web)  Web

Use `web` for search and fetch workflows.

```
openclaw infer web search --query \"OpenClaw docs\" --json
openclaw infer web search --query \"OpenClaw infer web providers\" --json
openclaw infer web fetch --url https://docs.openclaw.ai/cli/infer --json
openclaw infer web providers --json
```

Notes:

- Use `web providers` to inspect available, configured, and selected providers.

## [​](https://docs.openclaw.ai/cli/infer\\#embedding)  Embedding

Use `embedding` for vector creation and embedding provider inspection.

```
openclaw infer embedding create --text \"friendly lobster\" --json
openclaw infer embedding create --text \"customer support ticket: delayed shipment\" --model openai/text-embedding-3-large --json
openclaw infer embedding providers --json
```

## [​](https://docs.openclaw.ai/cli/infer\\#json-output)  JSON output

Infer commands normalize JSON output under a shared envelope:

```
{
  \"ok\": true,
  \"capability\": \"image.generate\",
  \"transport\": \"local\",
  \"provider\": \"openai\",
  \"model\": \"gpt-image-2\",
  \"attempts\": [],
  \"outputs\": []
}
```

Top-level fields are stable:

- `ok`
- `capability`
- `transport`
- `provider`
- `model`
- `attempts`
- `outputs`
- `error`

For generated media commands, `outputs` contains files written by OpenClaw. Use
the `path`, `mimeType`, `size`, and any media-specific dimensions in that array
for automation instead of parsing human-readable stdout.

## [​](https://docs.openclaw.ai/cli/infer\\#common-pitfalls)  Common pitfalls

```
# Bad
openclaw infer media image generate --prompt \"friendly lobster\"

# Good
openclaw infer image generate --prompt \"friendly lobster\"
```

```
# Bad
openclaw infer audio transcribe --file ./memo.m4a --model whisper-1 --json

# Good
openclaw infer audio transcribe --file ./memo.m4a --model openai/whisper-1 --json
```

## [​](https://docs.openclaw.ai/cli/infer\\#notes)  Notes

- `openclaw capability ...` is an alias for `openclaw infer ...`.

## [​](https://docs.openclaw.ai/cli/infer\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Models](https://docs.openclaw.ai/concepts/models)

[Hooks](https://docs.openclaw.ai/cli/hooks) [Memory](https://docs.openclaw.ai/cli/memory)

Ctrl+I

---

## Directory - OpenClaw
**Source:** https://docs.openclaw.ai/cli/directory

[Skip to main content](https://docs.openclaw.ai/cli/directory#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nChannels and messaging\n\nDirectory\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [openclaw directory](https://docs.openclaw.ai/cli/directory#openclaw-directory)\n- [Common flags](https://docs.openclaw.ai/cli/directory#common-flags)\n- [Notes](https://docs.openclaw.ai/cli/directory#notes)\n- [Using results with message send](https://docs.openclaw.ai/cli/directory#using-results-with-message-send)\n- [ID formats (by channel)](https://docs.openclaw.ai/cli/directory#id-formats-by-channel)\n- [Self (“me”)](https://docs.openclaw.ai/cli/directory#self-%E2%80%9Cme%E2%80%9D)\n- [Peers (contacts/users)](https://docs.openclaw.ai/cli/directory#peers-contacts%2Fusers)\n- [Groups](https://docs.openclaw.ai/cli/directory#groups)\n- [Related](https://docs.openclaw.ai/cli/directory#related)\n\n# [​](https://docs.openclaw.ai/cli/directory\\#openclaw-directory)  `openclaw directory`\n\nDirectory lookups for channels that support it (contacts/peers, groups, and “me”).\n\n## [​](https://docs.openclaw.ai/cli/directory\\#common-flags)  Common flags\n\n- `--channel <name>`: channel id/alias (required when multiple channels are configured; auto when only one is configured)\n- `--account <id>`: account id (default: channel default)\n- `--json`: output JSON\n\n## [​](https://docs.openclaw.ai/cli/directory\\#notes)  Notes\n\n- `directory` is meant to help you find IDs you can paste into other commands (especially `openclaw message send --target ...`).\n- For many channels, results are config-backed (allowlists / configured groups) rather than a live provider directory.\n- Default output is `id` (and sometimes `name`) separated by a tab; use `--json` for scripting.\n\n## [​](https://docs.openclaw.ai/cli/directory\\#using-results-with-message-send)  Using results with `message send`\n\n```\nopenclaw directory peers list --channel slack --query \"U0\"\nopenclaw message send --channel slack --target user:U012ABCDEF --message \"hello\"\n```\n\n## [​](https://docs.openclaw.ai/cli/directory\\#id-formats-by-channel)  ID formats (by channel)\n\n- WhatsApp: `+15551234567` (DM), `1234567890-1234567890@g.us` (group)\n- Telegram: `@username` or numeric chat id; groups are numeric ids\n- Slack: `user:U…` and `channel:C…`\n- Discord: `user:<id>` and `channel:<id>`\n- Matrix (plugin): `user:@user:server`, `room:!roomId:server`, or `#alias:server`\n- Microsoft Teams (plugin): `user:<id>` and `conversation:<id>`\n- Zalo (plugin): user id (Bot API)\n- Zalo Personal / `zalouser` (plugin): thread id (DM/group) from `zca` (`me`, `friend list`, `group list`)\n\n## [​](https://docs.openclaw.ai/cli/directory\\#self-%E2%80%9Cme%E2%80%9D)  Self (“me”)\n\n```\nopenclaw directory self --channel zalouser\n```\n\n## [​](https://docs.openclaw.ai/cli/directory\\#peers-contacts/users)  Peers (contacts/users)\n\n```\nopenclaw directory peers list --channel zalouser\nopenclaw directory peers list --channel zalouser --query \"name\"\nopenclaw directory peers list --channel zalouser --limit 50\n```\n\n## [​](https://docs.openclaw.ai/cli/directory\\#groups)  Groups\n\n```\nopenclaw directory groups list --channel zalouser\nopenclaw directory groups list --channel zalouser --query \"work\"\nopenclaw directory groups members --channel zalouser --group-id <id>\n```\n\n## [​](https://docs.openclaw.ai/cli/directory\\#related)  Related\n\n- [CLI reference](https://docs.openclaw.ai/cli)\n\n[Devices](https://docs.openclaw.ai/cli/devices) [Pairing](https://docs.openclaw.ai/cli/pairing)\n\nCtrl+I

---

## System - OpenClaw
**Source:** https://docs.openclaw.ai/cli/system

# System\n\n`openclaw system`\n\nThis command is used to interact with the Gateway system services.\n\n## `system heartbeat`\n\nManage the Gateway's heartbeat service. The heartbeat service sends periodic\nstatus updates to the Gateway. This is used to keep track of the Gateway's\nhealth and availability.\n\nUsage:\n\n```bash\nopenclaw system heartbeat <command>\n```\n\nCommands:\n\n- `last`: show the last heartbeat event.\n- `enable`: turn heartbeats back on (use this if they were disabled).\n- `disable`: pause heartbeats.\n\nHeartbeat controls:\n\n- `last`: show the last heartbeat event.\n- `enable`: turn heartbeats back on (use this if they were disabled).\n- `disable`: pause heartbeats.\n\nFlags:\n\n- `--json`: machine-readable output.\n- `--url`, `--token`, `--timeout`, `--expect-final`: shared Gateway RPC flags.\n\n## `system presence`\n\nList the current system presence entries the Gateway knows about (nodes,\ninstances, and similar status lines).Flags:\n\n- `--json`: machine-readable output.\n- `--url`, `--token`, `--timeout`, `--expect-final`: shared Gateway RPC flags.\n\n## Notes\n\n- Requires a running Gateway reachable by your current config (local or remote).\n- System events are ephemeral and not persisted across restarts.\n\n## Related\n\n- [CLI reference](https://docs.openclaw.ai/cli)\n\n[Sessions](https://docs.openclaw.ai/cli/sessions) [`openclaw tasks`](https://docs.openclaw.ai/cli/tasks)

---

## Flows (redirect) - OpenClaw
**Source:** https://docs.openclaw.ai/cli/flows

[Skip to main content](https://docs.openclaw.ai/cli/flows#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools and execution

Flows (redirect)

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw tasks flow](https://docs.openclaw.ai/cli/flows#openclaw-tasks-flow)
- [Related](https://docs.openclaw.ai/cli/flows#related)

# [​](https://docs.openclaw.ai/cli/flows\\#openclaw-tasks-flow)  `openclaw tasks flow`

Flow commands are subcommands of `openclaw tasks`, not a standalone `flows` command.

```
openclaw tasks flow list [--json]
openclaw tasks flow show <lookup>
openclaw tasks flow cancel <lookup>
```

For full documentation see [Task Flow](https://docs.openclaw.ai/automation/taskflow) and the [tasks CLI reference](https://docs.openclaw.ai/cli/tasks).

## [​](https://docs.openclaw.ai/cli/flows\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Automation](https://docs.openclaw.ai/automation)

[Cron](https://docs.openclaw.ai/cli/cron) [Node](https://docs.openclaw.ai/cli/node)

Ctrl+I

---

## TUI - OpenClaw
**Source:** https://docs.openclaw.ai/cli/tui

[Skip to main content](https://docs.openclaw.ai/cli/tui#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Interfaces

TUI

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw tui](https://docs.openclaw.ai/cli/tui#openclaw-tui)
- [Examples](https://docs.openclaw.ai/cli/tui#examples)
- [Config repair loop](https://docs.openclaw.ai/cli/tui#config-repair-loop)
- [Related](https://docs.openclaw.ai/cli/tui#related)

# [​](https://docs.openclaw.ai/cli/tui\\#openclaw-tui)  `openclaw tui`

Open the terminal UI connected to the Gateway, or run it in local embedded
mode.Related:

- TUI guide: [TUI](https://docs.openclaw.ai/web/tui)

Notes:

- `chat` and `terminal` are aliases for `openclaw tui --local`.
- `--local` cannot be combined with `--url`, `--token`, or `--password`.
- `tui` resolves configured gateway auth SecretRefs for token/password auth when possible (`env`/`file`/`exec` providers).
- When launched from inside a configured agent workspace directory, TUI auto-selects that agent for the session key default (unless `--session` is explicitly `agent:<id>:...`).
- Local mode uses the embedded agent runtime directly. Most local tools work, but Gateway-only features are unavailable.
- Local mode adds `/auth [provider]` inside the TUI command surface.
- Plugin approval gates still apply in local mode. Tools that require approval prompt for a decision in the terminal; nothing is silently auto-approved because the Gateway is not involved.

## [​](https://docs.openclaw.ai/cli/tui\\#examples)  Examples

```
openclaw chat
openclaw tui --local
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
openclaw chat --message \"Compare my config to the docs and tell me what to fix\"
# when run inside an agent workspace, infers that agent automatically
openclaw tui --session bugfix
```

## [​](https://docs.openclaw.ai/cli/tui\\#config-repair-loop)  Config repair loop

Use local mode when the current config already validates and you want the
embedded agent to inspect it, compare it against the docs, and help repair it
from the same terminal:If `openclaw config validate` is already failing, use `openclaw configure` or
`openclaw doctor --fix` first. `openclaw chat` does not bypass the invalid-
config guard.

```
openclaw chat
```

Then inside the TUI:

```
!openclaw config file
!openclaw docs gateway auth token secretref
!openclaw config validate
!openclaw doctor
```

Apply targeted fixes with `openclaw config set` or `openclaw configure`, then
rerun `openclaw config validate`. See [TUI](https://docs.openclaw.ai/web/tui) and [Config](https://docs.openclaw.ai/cli/config).

## [​](https://docs.openclaw.ai/cli/tui\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [TUI](https://docs.openclaw.ai/web/tui)

[Dashboard](https://docs.openclaw.ai/cli/dashboard) [ACP](https://docs.openclaw.ai/cli/acp)

Ctrl+I

---

## MCP
**Source:** https://docs.openclaw.ai/cli/mcp

[Skip to main content](https://docs.openclaw.ai/cli/mcp#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nUtility\n\nMCP\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [OpenClaw as an MCP server](https://docs.openclaw.ai/cli/mcp#openclaw-as-an-mcp-server)\n- [When to use serve](https://docs.openclaw.ai/cli/mcp#when-to-use-serve)\n- [How it works](https://docs.openclaw.ai/cli/mcp#how-it-works)\n- [Choose a client mode](https://docs.openclaw.ai/cli/mcp#choose-a-client-mode)\n- [What serve exposes](https://docs.openclaw.ai/cli/mcp#what-serve-exposes)\n- [Usage](https://docs.openclaw.ai/cli/mcp#usage)\n- [Bridge tools](https://docs.openclaw.ai/cli/mcp#bridge-tools)\n- [Event model](https://docs.openclaw.ai/cli/mcp#event-model)\n- [Claude channel notifications](https://docs.openclaw.ai/cli/mcp#claude-channel-notifications)\n- [MCP client config](https://docs.openclaw.ai/cli/mcp#mcp-client-config)\n- [Options](https://docs.openclaw.ai/cli/mcp#options)\n- [Security and trust boundary](https://docs.openclaw.ai/cli/mcp#security-and-trust-boundary)\n- [Testing](https://docs.openclaw.ai/cli/mcp#testing)\n- [Troubleshooting](https://docs.openclaw.ai/cli/mcp#troubleshooting)\n- [OpenClaw as an MCP client registry](https://docs.openclaw.ai/cli/mcp#openclaw-as-an-mcp-client-registry)\n- [Saved MCP server definitions](https://docs.openclaw.ai/cli/mcp#saved-mcp-server-definitions)\n- [Stdio transport](https://docs.openclaw.ai/cli/mcp#stdio-transport)\n- [SSE / HTTP transport](https://docs.openclaw.ai/cli/mcp#sse-/-http-transport)\n- [Streamable HTTP transport](https://docs.openclaw.ai/cli/mcp#streamable-http-transport)\n- [Current limits](https://docs.openclaw.ai/cli/mcp#current-limits)\n- [Related](https://docs.openclaw.ai/cli/mcp#related)\n\n`openclaw mcp` has two jobs:\n\n- run OpenClaw as an MCP server with `openclaw mcp serve`\n- manage OpenClaw-owned outbound MCP server definitions with `list`, `show`, `set`, and `unset`\n\nIn other words:\n\n- `serve` is OpenClaw acting as an MCP server\n- `list` / `show` / `set` / `unset` is OpenClaw acting as an MCP client-side registry for other MCP servers its runtimes may consume later\n\nUse [`openclaw acp`](https://docs.openclaw.ai/cli/acp) when OpenClaw should host a coding harness session itself and route that runtime through ACP.\n\n## [​](https://docs.openclaw.ai/cli/mcp\\#openclaw-as-an-mcp-server)  OpenClaw as an MCP server\n\nThis is the `openclaw mcp serve` path.\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#when-to-use-serve)  When to use `serve`\n\nUse `openclaw mcp serve` when:\n\n- Codex, Claude Code, or another MCP client should talk directly to OpenClaw-backed channel conversations\n- you already have a local or remote OpenClaw Gateway with routed sessions\n- you want one MCP server that works across OpenClaw’s channel backends instead of running separate per-channel bridges\n\nUse [`openclaw acp`](https://docs.openclaw.ai/cli/acp) instead when OpenClaw should host the coding runtime itself and keep the agent session inside OpenClaw.\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#how-it-works)  How it works\n\n`openclaw mcp serve` starts a stdio MCP server. The MCP client owns that process. While the client keeps the stdio session open, the bridge connects to a local or remote OpenClaw Gateway over WebSocket and exposes routed channel conversations over MCP.\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/cli/mcp#)\n\nClient spawns the bridge\n\nThe MCP client spawns `openclaw mcp serve`.\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/cli/mcp#)\n\nBridge connects to Gateway\n\nThe bridge connects to the OpenClaw Gateway over WebSocket.\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/cli/mcp#)\n\nSessions become MCP conversations\n\nRouted sessions become MCP conversations and transcript/history tools.\n\n4\n\n[Navigate to header](https://docs.openclaw.ai/cli/mcp#)\n\nLive events queue\n\nLive events are queued in memory while the bridge is connected.\n\n5\n\n[Navigate to header](https://docs.openclaw.ai/cli/mcp#)\n\nOptional Claude push\n\nIf Claude channel mode is enabled, the same session can also receive Claude-specific push notifications.\n\nImportant behavior\n\n- live queue state starts when the bridge connects\n- older transcript history is read with `messages_read`\n- Claude push notifications only exist while the MCP session is alive\n- when the client disconnects, the bridge exits and the live queue is gone\n- one-shot agent entry points such as `openclaw agent` and `openclaw infer model run` retire any bundled MCP runtimes they open when the reply completes, so repeated scripted runs do not accumulate stdio MCP child processes\n- stdio MCP servers launched by OpenClaw (bundled or user-configured) are torn down as a process tree on shutdown, so child subprocesses started by the server do not survive after the parent stdio client exits\n- deleting or resetting a session disposes that session’s MCP clients through the shared runtime cleanup path, so there are no lingering stdio connections tied to a removed session\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#choose-a-client-mode)  Choose a client mode\n\nUse the same bridge in two different ways:\n\n- Generic MCP clients\n\n- Claude Code\n\n\nStandard MCP tools only. Use `conversations_list`, `messages_read`, `events_poll`, `events_wait`, `messages_send`, and the approval tools.\n\nStandard MCP tools plus the Claude-specific channel adapter. Enable `--claude-channel-mode on` or leave the default `auto`.\n\nToday, `auto` behaves the same as `on`. There is no client capability detection yet.\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#what-serve-exposes)  What `serve` exposes\n\nThe bridge uses existing Gateway session route metadata to expose channel-backed conversations. A conversation appears when OpenClaw already has session state with a known route such as:\n\n- `channel`\n- recipient or destination metadata\n- optional `accountId`\n- optional `threadId`\n\nThis gives MCP clients one place to:\n\n- list recent routed conversations\n- read recent transcript history\n- wait for new inbound events\n- send a reply back through the same route\n- see approval requests that arrive while the bridge is connected\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#usage)  Usage\n\n- Local Gateway\n\n- Remote Gateway (token)\n\n- Remote Gateway (password)\n\n- Verbose / Claude off\n\n\n```\nopenclaw mcp serve\n```\n\n```\nopenclaw mcp serve --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token\n```\n\n```\nopenclaw mcp serve --url wss://gateway-host:18789 --password-file ~/.openclaw/gateway.password\n```\n\n```\nopenclaw mcp serve --verbose\nopenclaw mcp serve --claude-channel-mode off\n```\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#bridge-tools)  Bridge tools\n\nThe current bridge exposes these MCP tools:\n\nconversations\\_list\n\nLists recent session-backed conversations that already have route metadata in Gateway session state.Useful filters:\n\n- `limit`\n- `search`\n- `channel`\n- `includeDerivedTitles`\n- `includeLastMessage`\n\nconversation\\_get\n\nReturns one conversation by `session_key`.\n\nmessages\\_read\n\nReads recent transcript messages for one session-backed conversation.\n\nattachments\\_fetch\n\nExtracts non-text message content blocks from one transcript message. This is a metadata view over transcript content, not a standalone durable attachment blob store.\n\nevents\\_poll\n\nReads queued live events since a numeric cursor.\n\nevents\\_wait\n\nLong-polls until the next matching queued event arrives or a timeout expires.Use this when a generic MCP client needs near-real-time delivery without a Claude-specific push protocol.\n\nmessages\\_send\n\nSends text back through the same route already recorded on the session.Current behavior:\n\n- requires an existing conversation route\n- uses the session’s channel, recipient, account id, and thread id\n- sends text only\n\npermissions\\_list\\_open\n\nLists pending exec/plugin approval requests the bridge has observed since it connected to the Gateway.\n\npermissions\\_respond\n\nResolves one pending exec/plugin approval request with:\n\n- `allow-once`\n- `allow-always`\n- `deny`\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#event-model)  Event model\n\nThe bridge keeps an in-memory event queue while it is connected.Current event types:\n\n- `message`\n- `exec_approval_requested`\n- `exec_approval_resolved`\n- `plugin_approval_requested`\n- `plugin_approval_resolved`\n- `claude_permission_request`\n\n- the queue is live-only; it starts when the MCP bridge starts\n- `events_poll` and `events_wait` do not replay older Gateway history by themselves\n- durable backlog should be read with `messages_read`\n\n### [​](https://openclaw.ai/cli/mcp\\#claude-channel-notifications)  Claude channel notifications\n\nThe bridge can also expose Claude-specific channel notifications. This is the OpenClaw equivalent of a Claude Code channel adapter: standard MCP tools remain available, but live inbound messages can also arrive as Claude-specific MCP notifications.\n\n- off\n\n- on\n\n- auto (default)\n\n\n`--claude-channel-mode off`: standard MCP tools only.\n\n`--claude-channel-mode on`: enable Claude channel notifications.\n\n`--claude-channel-mode auto`: current default; same bridge behavior as `on`.\n\nWhen Claude channel mode is enabled, the server advertises Claude experimental capabilities and can emit:\n\n- `notifications/claude/channel`\n- `notifications/claude/channel/permission`\n\nCurrent bridge behavior:\n\n- inbound `user` transcript messages are forwarded as `notifications/claude/channel`\n- Claude permission requests received over MCP are tracked in-memory\n- if the linked conversation later sends `yes abcde` or `no abcde`, the bridge converts that to `notifications/claude/channel/permission`\n- these notifications are live-session only; if the MCP client disconnects, there is no push target\n\nThis is intentionally client-specific. Generic MCP clients should rely on the standard polling tools.\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#mcp-client-config)  MCP client config\n\nExample stdio client config:\n\n```\n{\n  \"mcpServers\": {\n    \"openclaw\": {\n      \"command\": \"openclaw\",\n      \"args\": [\\\n        \"mcp\",\\\n        \"serve\",\\\n        \"--url\",\\\n        \"wss://gateway-host:18789\",\\\n        \"--token-file\",\\\n        \"/path/to/gateway.token\"\\\n      ]\n    }\n  }\n}\n```\n\nFor most generic MCP clients, start with the standard tool surface and ignore Claude mode. Turn Claude mode on only for clients that actually understand the Claude-specific notification methods.\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#options)  Options\n\n`openclaw mcp serve` supports:\n\n[​](https://docs.openclaw.ai/cli/mcp#param-url)\n\n--url\n\nstring\n\nGateway WebSocket URL.\n\n[​](https://docs.openclaw.ai/cli/mcp#param-token)\n\n--token\n\nstring\n\nGateway token.\n\n[​](https://docs.openclaw.ai/cli/mcp#param-token-file)\n\n--token-file\n\nstring\n\nRead token from file.\n\n[​](https://docs.openclaw.ai/cli/mcp#param-password)\n\n--password\n\nstring\n\nGateway password.\n\n[​](https://docs.openclaw.ai/cli/mcp#param-password-file)\n\n--password-file\n\nstring\n\nRead password from file.\n\n[​](https://docs.openclaw.ai/cli/mcp#param-claude-channel-mode)\n\n--claude-channel-mode\n\n\"auto\" \\| \"on\" \\| \"off\"\n\nClaude notification mode.\n\n[​](https://docs.openclaw.ai/cli/mcp#param-v-verbose)\n\n-v, --verbose\n\nboolean\n\nVerbose logs on stderr.\n\nPrefer `--token-file` or `--password-file` over inline secrets when possible.\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#security-and-trust-boundary)  Security and trust boundary\n\nThe bridge does not invent routing. It only exposes conversations that Gateway already knows how to route.That means:\n\n- sender allowlists, pairing, and channel-level trust still belong to the underlying OpenClaw channel configuration\n- `messages_send` can only reply through an existing stored route\n- approval state is live/in-memory only for the current bridge session\n- bridge auth should use the same Gateway token or password controls you would trust for any other remote Gateway client\n\nIf a conversation is missing from `conversations_list`, the usual cause is not MCP configuration. It is missing or incomplete route metadata in the underlying Gateway session.\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#testing)  Testing\n\nOpenClaw ships a deterministic Docker smoke for this bridge:\n\n```\npnpm test:docker:mcp-channels\n```\n\nThat smoke:\n\n- starts a seeded Gateway container\n- starts a second container that spawns `openclaw mcp serve`\n- verifies conversation discovery, transcript reads, attachment metadata reads, live event queue behavior, and outbound send routing\n- validates Claude-style channel and permission notifications over the real stdio MCP bridge\n- This is the fastest way to prove the bridge works without wiring a real Telegram, Discord, or iMessage account into the test run.For broader testing context, see [Testing](https://docs.openclaw.ai/help/testing).\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#troubleshooting)  Troubleshooting\n\nNo conversations returned\n\nUsually means the Gateway session is not already routable. Confirm that the underlying session has stored channel/provider, recipient, and optional account/thread route metadata.\n\nevents\\_poll or events\\_wait misses older messages\n\nExpected. The live queue starts when the bridge connects. Read older transcript history with `messages_read`.\n\nClaude notifications do not show up\n\nCheck all of these:\n\n- the client kept the stdio MCP session open\n- `--claude-channel-mode` is `on` or `auto`\n- the client actually understands the Claude-specific notification methods\n- the inbound message happened after the bridge connected\n\nApprovals are missing\n\n`permissions_list_open` only shows approval requests observed while the bridge was connected. It is not a durable approval history API.\n\n## [​](https://docs.openclaw.ai/cli/mcp\\#openclaw-as-an-mcp-client-registry)  OpenClaw as an MCP client registry\n\nThis is the `openclaw mcp list`, `show`, `set`, and `unset` path.These commands do not expose OpenClaw over MCP. They manage OpenClaw-owned MCP server definitions under `mcp.servers` in OpenClaw config.Those saved definitions are for runtimes that OpenClaw launches or configures later, such as embedded Pi and other runtime adapters. OpenClaw stores the definitions centrally so those runtimes do not need to keep their own duplicate MCP server lists.\n\nImportant behavior\n\n- these commands only read or write OpenClaw config\n- they do not connect to the target MCP server\n- they do not validate whether the command, URL, or remote transport is reachable right now\n- runtime adapters decide which transport shapes they actually support at execution time\n- embedded Pi exposes configured MCP tools in normal `coding` and `messaging` tool profiles; `minimal` still hides them, and `tools.deny: [\"bundle-mcp\"]` disables them explicitly\n- session-scoped bundled MCP runtimes are reaped after `mcp.sessionIdleTtlMs` milliseconds of idle time (default 10 minutes; set `0` to disable) and one-shot embedded runs clean them up at run end\n\nRuntime adapters may normalize this shared registry into the shape their downstream client expects. For example, embedded Pi consumes OpenClaw `transport` values directly, while Claude Code and Gemini receive CLI-native `type` values such as `http`, `sse`, or `stdio`.\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#saved-mcp-server-definitions)  Saved MCP server definitions\n\nOpenClaw also stores a lightweight MCP server registry in config for surfaces that want OpenClaw-managed MCP definitions.Commands:\n\n- `openclaw mcp list`\n- `openclaw mcp show [name]`\n- `openclaw mcp set <name> <json>`\n- `openclaw mcp unset <name>`\n\nNotes:\n\n- `list` sorts server names.\n- `show` without a name prints the full configured MCP server object.\n- `set` expects one JSON object value on the command line.\n- Use `transport: \"streamable-http\"` for Streamable HTTP MCP servers. `openclaw mcp set` also normalizes CLI-native `type: \"http\"` to the same canonical config shape for compatibility.\n- `unset` fails if the named server does not exist.\n\nExamples:\n\n```\nopenclaw mcp list\nopenclaw mcp show context7 --json\nopenclaw mcp set context7 \'{\"command\":\"uvx\",\"args\":[\"context7-mcp\"]}\'\nopenclaw mcp set docs \'{\"url\":\"https://mcp.example.com\",\"transport\":\"streamable-http\"}\'\nopenclaw mcp unset context7\n```\n\nExample config shape:\n\n```\n{\n  \"mcp\": {\n    \"servers\": {\n      \"context7\": {\n        \"command\": \"uvx\",\n        \"args\": [\"context7-mcp\"]\n      },\n      \"docs\": {\n        \"url\": \"https://mcp.example.com\",\n        \"transport\": \"streamable-http\"\n      }\n    }\n  }\n}\n```\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#stdio-transport)  Stdio transport\n\nLaunches a local child process and communicates over stdin/stdout.\n\n| Field | Description |\n| --- | --- |\n| `command` | Executable to spawn (required) |\n| `args` | Array of command-line arguments |\n| `env` | Extra environment variables |\n| `cwd` / `workingDirectory` | Working directory for the process |\n\n**Stdio env safety filter**OpenClaw rejects interpreter-startup env keys that can alter how a stdio MCP server starts up before the first RPC, even if they appear in a server’s `env` block. Blocked keys include `NODE_OPTIONS`, `PYTHONSTARTUP`, `PYTHONPATH`, `PERL5OPT`, `RUBYOPT`, `SHELLOPTS`, `PS4`, and similar runtime-control variables. Startup rejects these with a configuration error so they cannot inject an implicit prelude, swap the interpreter, or enable a debugger against the stdio process. Ordinary credential, proxy, and server-specific env vars (`GITHUB_TOKEN`, `HTTP_PROXY`, custom `*_API_KEY`, etc.) are unaffected.If your MCP server genuinely needs one of the blocked variables, set it on the gateway host process instead of under the stdio server’s `env`.\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#sse-/-http-transport)  SSE / HTTP transport\n\nConnects to a remote MCP server over HTTP Server-Sent Events.\n\n| Field | Description |\n| --- | --- |\n| `url` | HTTP or HTTPS URL of the remote server (required) |\n| `headers` | Optional key-value map of HTTP headers (for example auth tokens) |\n| `connectionTimeoutMs` | Per-server connection timeout in ms (optional) |\n\nExample:\n\n```\n{\n  \"mcp\": {\n    \"servers\": {\n      \"remote-tools\": {\n        \"url\": \"https://mcp.example.com\",\n        \"headers\": {\n          \"Authorization\": \"Bearer <token>\"\n        }\n      }\n    }\n  }\n}\n```\n\n### [​](https://docs.openclaw.ai/cli/mcp\\#streamable-http-transport)  Streamable HTTP transport\n\nConnects to a remote MCP server over HTTP with streaming capabilities.\n\n| Field | Description |\n| --- | --- |\n| `url` | HTTP or HTTPS URL of the remote server (required) |\n| `headers` | Optional key-value map of HTTP headers (for example auth tokens) |\n| `connectionTimeoutMs` | Per-server connection timeout in ms (optional) |\n\nExample:\n\n```\n{\n  \"mcp\": {\n    \"servers\": {\n      \"remote-tools\": {\n        \"url\": \"https://mcp.example.com\",\n        \"headers\": {\n          \"Authorization\": \"Bearer <token>\"\n        }\n      }\n    }\n  }\n}\n```\n\n## [​](https://docs.openclaw.ai/cli/mcp\\#current-limits)  Current limits\n\n- The stdio bridge is currently limited to one active MCP client process at a time.\n- Only one client can be connected to a Gateway session at a time.\n- The bridge does not currently support multiple concurrent MCP conversations over a single stdio session.\n\n## [​](https://docs.openclaw.ai/cli/mcp\\#related)  Related\n\n- [CLI](https://docs.openclaw.ai/cli)\n- [ACP](https://docs.openclaw.ai/cli/acp)\n- [Gateway](https://docs.openclaw.ai/gateway)\n- [Tools](https://docs.openclaw.ai/tools)\n- [Agents](https://docs.openclaw.ai/concepts/architecture)\n- [Channels](https://docs.openclaw.ai/channels)\n- [Models](https://docs.openclaw.ai/providers)\n- [Platforms](https://docs.openclaw.ai/platforms)\n- [Install](https://docs.openclaw.ai/install)\n- [Help](https://docs.openclaw.ai/help)\n- [Get started](https://docs.openclaw.ai/)

---

## Pairing - OpenClaw
**Source:** https://docs.openclaw.ai/cli/pairing

[Skip to main content](https://docs.openclaw.ai/cli/pairing#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Channels and messaging

Pairing

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw pairing](https://docs.openclaw.ai/cli/pairing#openclaw-pairing)
- [Commands](https://docs.openclaw.ai/cli/pairing#commands)
- [pairing list](https://docs.openclaw.ai/cli/pairing#pairing-list)
- [pairing approve](https://docs.openclaw.ai/cli/pairing#pairing-approve)
- [Notes](https://docs.openclaw.ai/cli/pairing#notes)
- [Related](https://docs.openclaw.ai/cli/pairing#related)

# [​](https://docs.openclaw.ai/cli/pairing\\#openclaw-pairing)  `openclaw pairing`

Approve or inspect DM pairing requests (for channels that support pairing).Related:

- Pairing flow: [Pairing](https://docs.openclaw.ai/channels/pairing)

## [​](https://docs.openclaw.ai/cli/pairing\\#commands)  Commands

```
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve <code>
openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## [​](https://docs.openclaw.ai/cli/pairing\\#pairing-list)  `pairing list`

List pending pairing requests for one channel.Options:

- `[channel]`: positional channel id
- `--channel <channel>`: explicit channel id
- `--account <accountId>`: account id for multi-account channels
- `--json`: machine-readable output

Notes:

- If multiple pairing-capable channels are configured, you must provide a channel either positionally or with `--channel`.
- Extension channels are allowed as long as the channel id is valid.

## [​](https://docs.openclaw.ai/cli/pairing\\#pairing-approve)  `pairing approve`

Approve a pending pairing code and allow that sender.Usage:

- `openclaw pairing approve <channel> <code>`
- `openclaw pairing approve --channel <channel> <code>`
- `openclaw pairing approve <code>` when exactly one pairing-capable channel is configured

Options:

- `--channel <channel>`: explicit channel id
- `--account <accountId>`: account id for multi-account channels
- `--notify`: send a confirmation back to the requester on the same channel

## [​](https://docs.openclaw.ai/cli/pairing\\#notes)  Notes

- Channel input: pass it positionally (`pairing list telegram`) or with `--channel <channel>`.
- `pairing list` supports `--account <accountId>` for multi-account channels.
- `pairing approve` supports `--account <accountId>` and `--notify`.
- If only one pairing-capable channel is configured, `pairing approve <code>` is allowed.

## [​](https://docs.openclaw.ai/cli/pairing\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Channel pairing](https://docs.openclaw.ai/channels/pairing)

[Directory](https://docs.openclaw.ai/cli/directory) [QR](https://docs.openclaw.ai/cli/qr)

Ctrl+I

---

## Skills - OpenClaw
**Source:** https://docs.openclaw.ai/cli/skills

[Skip to main content](https://docs.openclaw.ai/cli/skills#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins and skills

Skills

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw skills](https://docs.openclaw.ai/cli/skills#openclaw-skills)
- [Commands](https://docs.openclaw.ai/cli/skills#commands)
- [Related](https://docs.openclaw.ai/cli/skills#related)

# [​](https://docs.openclaw.ai/cli/skills\\#openclaw-skills)  `openclaw skills`

Inspect local skills and install/update skills from ClawHub.Related:

- Skills system: [Skills](https://docs.openclaw.ai/tools/skills)
- Skills config: [Skills config](https://docs.openclaw.ai/tools/skills-config)
- ClawHub installs: [ClawHub](https://docs.openclaw.ai/tools/clawhub)

## [​](https://docs.openclaw.ai/cli/skills\\#commands)  Commands

```
openclaw skills search \"calendar\"
openclaw skills search --limit 20 --json
openclaw skills install <slug>
openclaw skills install <slug> --version <version>
openclaw skills install <slug> --force
openclaw skills update <slug>
openclaw skills update --all
openclaw skills list
openclaw skills list --eligible
openclaw skills list --json
openclaw skills list --verbose
openclaw skills info <name>
openclaw skills info <name> --json
openclaw skills check
openclaw skills check --json
```

`search`/`install`/`update` use ClawHub directly and install into the active
workspace `skills/` directory. `list`/`info`/`check` still inspect the local
skills visible to the current workspace and config.This CLI `install` command downloads skill folders from ClawHub. Gateway-backed
skill dependency installs triggered from onboarding or Skills settings use the
separate `skills.install` request path instead.Notes:

- `search [query...]` accepts an optional query; omit it to browse the default
ClawHub search feed.
- `search --limit <n>` caps returned results.
- `install --force` overwrites an existing workspace skill folder for the same
slug.
- `update --all` only updates tracked ClawHub installs in the active workspace.
- `list` is the default action when no subcommand is provided.
- `list`, `info`, and `check` write their rendered output to stdout. With
`--json`, that means the machine-readable payload stays on stdout for pipes
and scripts.

## [​](https://docs.openclaw.ai/cli/skills\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Skills](https://docs.openclaw.ai/tools/skills)

[Plugins](https://docs.openclaw.ai/cli/plugins) [Dashboard](https://docs.openclaw.ai/cli/dashboard)

Ctrl+I

---

## Doctor - OpenClaw
**Source:** https://docs.openclaw.ai/cli/doctor

[Skip to main content](https://docs.openclaw.ai/cli/doctor#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Gateway and service

Doctor

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw doctor](https://docs.openclaw.ai/cli/doctor#openclaw-doctor)
- [Examples](https://docs.openclaw.ai/cli/doctor#examples)
- [Options](https://docs.openclaw.ai/cli/doctor#options)
- [macOS: launchctl env overrides](https://docs.openclaw.ai/cli/doctor#macos-launchctl-env-overrides)
- [Related](https://docs.openclaw.ai/cli/doctor#related)

# [​](https://docs.openclaw.ai/cli/doctor\\#openclaw-doctor)  `openclaw doctor`

Health checks + quick fixes for the gateway and channels.Related:

- Troubleshooting: [Troubleshooting](https://docs.openclaw.ai/gateway/troubleshooting)
- Security audit: [Security](https://docs.openclaw.ai/gateway/security)

## [​](https://docs.openclaw.ai/cli/doctor\\#examples)  Examples

```
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
openclaw doctor --repair --non-interactive
openclaw doctor --generate-gateway-token
```

## [​](https://docs.openclaw.ai/cli/doctor\\#options)  Options

- `--no-workspace-suggestions`: disable workspace memory/search suggestions
- `--yes`: accept defaults without prompting
- `--repair`: apply recommended repairs without prompting
- `--fix`: alias for `--repair`
- `--force`: apply aggressive repairs, including overwriting custom service config when needed
- `--non-interactive`: run without prompts; safe migrations only
- `--generate-gateway-token`: generate and configure a gateway token
- `--deep`: scan system services for extra gateway installs

Notes:

- Interactive prompts (like keychain/OAuth fixes) only run when stdin is a TTY and `--non-interactive` is **not** set. Headless runs (cron, Telegram, no terminal) will skip prompts.
- Performance: non-interactive `doctor` runs skip eager plugin loading so headless health checks stay fast. Interactive sessions still fully load plugins when a check needs their contribution.
- `--fix` (alias for `--repair`) writes a backup to `~/.openclaw/openclaw.json.bak` and drops unknown config keys, listing each removal.
- State integrity checks now detect orphan transcript files in the sessions directory and can archive them as `.deleted.<timestamp>` to reclaim space safely.
- Doctor also scans `~/.openclaw/cron/jobs.json` (or `cron.store`) for legacy cron job shapes and can rewrite them in place before the scheduler has to auto-normalize them at runtime.
- Doctor repairs missing bundled plugin runtime dependencies without writing into packaged global installs. For root-owned npm installs or hardened systemd units, set `OPENCLAW_PLUGIN_STAGE_DIR` to a writable directory such as `/var/lib/openclaw/plugin-runtime-deps`; it can also be a path-list such as `/opt/openclaw/plugin-runtime-deps:/var/lib/openclaw/plugin-runtime-deps`, where earlier roots are read-only lookup layers and the final root is the repair target.
- Set `OPENCLAW_SERVICE_REPAIR_POLICY=external` when another supervisor owns the gateway lifecycle. Doctor still reports gateway/service health and applies non-service repairs, but skips service install/start/restart/bootstrap and legacy service cleanup.
- Doctor auto-migrates legacy flat Talk config (`talk.voiceId`, `talk.modelId`, and friends) into `talk.provider` \\+ `talk.providers.<provider>`.
- Repeat `doctor --fix` runs no longer report/apply Talk normalization when the only difference is object key order.
- Doctor includes a memory-search readiness check and can recommend `openclaw configure --section model` when embedding credentials are missing.
- If sandbox mode is enabled but Docker is unavailable, doctor reports a high-signal warning with remediation (`install Docker` or `openclaw config set agents.defaults.sandbox.mode off`).
- If `gateway.auth.token`/`gateway.auth.password` are SecretRef-managed and unavailable in the current command path, doctor reports a read-only warning and does not write plaintext fallback credentials.
- If channel SecretRef inspection fails in a fix path, doctor continues and reports a warning instead of exiting early.
- Telegram `allowFrom` username auto-resolution (`doctor --fix`) requires a resolvable Telegram token in the current command path. If token inspection is unavailable, doctor reports a warning and skips auto-resolution for that pass.

## [​](https://docs.openclaw.ai/cli/doctor\\#macos-launchctl-env-overrides)  macOS: `launchctl` env overrides

If you previously ran `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (or `...PASSWORD`), that value overrides your config file and can cause persistent “unauthorized” errors.

```
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

## [​](https://docs.openclaw.ai/cli/doctor\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Gateway doctor](https://docs.openclaw.ai/gateway/doctor)

[Daemon](https://docs.openclaw.ai/cli/daemon) [Gateway](https://docs.openclaw.ai/cli/gateway)

Ctrl+I

---

## Message - OpenClaw
**Source:** https://docs.openclaw.ai/cli/message

[Skip to main content](https://docs.openclaw.ai/cli/message#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Agents and sessions

Message

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw message](https://docs.openclaw.ai/cli/message#openclaw-message)
- [Usage](https://docs.openclaw.ai/cli/message#usage)
- [Common flags](https://docs.openclaw.ai/cli/message#common-flags)
- [SecretRef behavior](https://docs.openclaw.ai/cli/message#secretref-behavior)
- [Actions](https://docs.openclaw.ai/cli/message#actions)
- [Core](https://docs.openclaw.ai/cli/message#core)
- [Threads](https://docs.openclaw.ai/cli/message#threads)
- [Emojis](https://docs.openclaw.ai/cli/message#emojis)
- [Stickers](https://docs.openclaw.ai/cli/message#stickers)
- [Roles / Channels / Members / Voice](https://docs.openclaw.ai/cli/message#roles-%2F-channels-%2F-members-%2F-voice)
- [Events](https://docs.openclaw.ai/cli/message#events)
- [Moderation (Discord)](https://docs.openclaw.ai/cli/message#moderation-discord)
- [Broadcast](https://docs.openclaw.ai/cli/message#broadcast)
- [Examples](https://docs.openclaw.ai/cli/message#examples)
- [Related](https://docs.openclaw.ai/cli/message#related)

# [​](https://docs.openclaw.ai/cli/message\\#openclaw-message)  `openclaw message`

Single outbound command for sending messages and channel actions
(Discord/Google Chat/iMessage/Matrix/Mattermost (plugin)/Microsoft Teams/Signal/Slack/Telegram/WhatsApp).

## [​](https://docs.openclaw.ai/cli/message\\#usage)  Usage

```
openclaw message <subcommand> [flags]
```

Channel selection:

- `--channel` required if more than one channel is configured.
- If exactly one channel is configured, it becomes the default.
- Values: `discord|googlechat|imessage|matrix|mattermost|msteams|signal|slack|telegram|whatsapp` (Mattermost requires plugin)

Target formats (`--target`):

- WhatsApp: E.164 or group JID
- Telegram: chat id or `@username`
- Discord: `channel:<id>` or `user:<id>` (or `<@id>` mention; raw numeric ids are treated as channels)
- Google Chat: `spaces/<spaceId>` or `users/<userId>`
- Slack: `channel:<id>` or `user:<id>` (raw channel id is accepted)
- Mattermost (plugin): `channel:<id>`, `user:<id>`, or `@username` (bare ids are treated as channels)
- Signal: `+E.164`, `group:<id>`, `signal:+E.164`, `signal:group:<id>`, or `username:<name>`/`u:<name>`
- iMessage: handle, `chat_id:<id>`, `chat_guid:<guid>`, or `chat_identifier:<id>`
- Matrix: `@user:server`, `!room:server`, or `#alias:server`
- Microsoft Teams: conversation id (`19:...@thread.tacv2`) or `conversation:<id>` or `user:<aad-object-id>`

Name lookup:

- For supported providers (Discord/Slack/etc), channel names like `Help` or `#help` are resolved via the directory cache.
- On cache miss, OpenClaw will attempt a live directory lookup when the provider supports it.

## [​](https://docs.openclaw.ai/cli/message\\#common-flags)  Common flags

- `--channel <name>`
- `--account <id>`
- `--target <dest>` (target channel or user for send/poll/read/etc)
- `--targets <name>` (repeat; broadcast only)
- `--json`
- `--dry-run`
- `--verbose`

## [​](https://docs.openclaw.ai/cli/message\\#secretref-behavior)  SecretRef behavior

- `openclaw message` resolves supported channel SecretRefs before running the selected action.
- Resolution is scoped to the active action target when possible:
  - channel-scoped when `--channel` is set (or inferred from prefixed targets like `discord:...`)
  - account-scoped when `--account` is set (channel globals + selected account surfaces)
  - when `--account` is omitted, OpenClaw does not force a `default` account SecretRef scope
- Unresolved SecretRefs on unrelated channels do not block a targeted message action.
- If the selected channel/account SecretRef is unresolved, the command fails closed for that action.

## [​](https://docs.openclaw.ai/cli/message\\#actions)  Actions

### [​](https://docs.openclaw.ai/cli/message\\#core)  Core

- `send`  - Channels: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/Matrix/Microsoft Teams
  - Required: `--target`, plus `--message`, `--media`, or `--presentation`
  - Optional: `--media`, `--presentation`, `--delivery`, `--pin`, `--reply-to`, `--thread-id`, `--gif-playback`, `--force-document`, `--silent`
  - Shared presentation payloads: `--presentation` sends semantic blocks (`text`, `context`, `divider`, `buttons`, `select`) that core renders through the selected channel’s declared capabilities. See [Message Presentation](https://docs.openclaw.ai/plugins/message-presentation).
  - Generic delivery preferences: `--delivery` accepts delivery hints such as `{ \"pin\": true }`; `--pin` is shorthand for pinned delivery when the channel supports it.
  - Telegram only: `--force-document` (send images and GIFs as documents to avoid Telegram compression)
  - Telegram only: `--thread-id` (forum topic id)
  - Slack only: `--thread-id` (thread timestamp; `--reply-to` uses the same field)
  - Telegram + Discord: `--silent`
  - WhatsApp only: `--gif-playback`
- `poll`  - Channels: WhatsApp/Telegram/Discord/Matrix/Microsoft Teams
  - Required: `--target`, `--poll-question`, `--poll-option` (repeat)
  - Optional: `--poll-multi`
  - Discord only: `--poll-duration-hours`, `--silent`, `--message`
  - Telegram only: `--poll-duration-seconds` (5-600), `--silent`, `--poll-anonymous` / `--poll-public`, `--thread-id`
- `react`  - Channels: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/Matrix
  - Required: `--message-id`, `--target`
  - Optional: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - Note: `--remove` requires `--emoji` (omit `--emoji` to clear own reactions where supported; see /tools/reactions)
  - WhatsApp only: `--participant`, `--from-me`
  - Signal group reactions: `--target-author` or `--target-author-uuid` required
- `reactions`  - Channels: Discord/Google Chat/Slack/Matrix
  - Required: `--message-id`, `--target`
  - Optional: `--limit`
- `read`  - Channels: Discord/Slack/Matrix
  - Required: `--target`
  - Optional: `--limit`, `--before`, `--after`
  - Discord only: `--around`
- `edit`  - Channels: Discord/Slack/Matrix
  - Required: `--message-id`, `--message`, `--target`
- `delete`  - Channels: Discord/Slack/Telegram/Matrix
  - Required: `--message-id`, `--target`
- `pin` / `unpin`  - Channels: Discord/Slack/Matrix
  - Required: `--message-id`, `--target`
- `pins` (list)  - Channels: Discord/Slack/Matrix
  - Required: `--target`
- `permissions`  - Channels: Discord/Matrix
  - Required: `--target`
  - Matrix only: available when Matrix encryption is enabled and verification actions are allowed
- `search`  - Channels: Discord
  - Required: `--guild-id`, `--query`
  - Optional: `--channel-id`, `--channel-ids` (repeat), `--author-id`, `--author-ids` (repeat), `--limit`

### [​](https://docs.openclaw.ai/cli/message\\#threads)  Threads

- `thread create`  - Channels: Discord
  - Required: `--thread-name`, `--target` (channel id)
  - Optional: `--message-id`, `--message`, `--auto-archive-min`
- `thread list`  - Channels: Discord
  - Required: `--guild-id`
  - Optional: `--channel-id`, `--include-archived`, `--before`, `--limit`
- `thread reply`  - Channels: Discord
  - Required: `--target` (thread id), `--message`
  - Optional: `--media`, `--reply-to`

### [​](https://docs.openclaw.ai/cli/message\\#emojis)  Emojis

- `emoji list`  - Discord: `--guild-id`
  - Slack: no extra flags
- `emoji upload`  - Channels: Discord
  - Required: `--guild-id`, `--emoji-name`, `--media`
  - Optional: `--role-ids` (repeat)

### [​](https://docs.openclaw.ai/cli/message\\#stickers)  Stickers

- `sticker send`  - Channels: Discord
  - Required: `--target`, `--sticker-id` (repeat)
  - Optional: `--message`
- `sticker upload`  - Channels: Discord
  - Required: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

### [​](https://docs.openclaw.ai/cli/message\\#roles-/-channels-/-members-/-voice)  Roles / Channels / Members / Voice

- `role info` (Discord): `--guild-id`
- `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
- `channel info` (Discord): `--target`
- `channel list` (Discord): `--guild-id`
- `member info` (Discord/Slack): `--user-id` (\\+ `--guild-id` for Discord)
- `voice status` (Discord): `--guild-id`, `--user-id`

### [​](https://docs.openclaw.ai/cli/message\\#events)  Events

- `event list` (Discord): `--guild-id`
- `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
  - Optional: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

### [​](https://docs.openclaw.ai/cli/message\\#moderation-discord)  Moderation (Discord)

- `timeout`: `--guild-id`, `--user-id` (optional `--duration-min` or `--until`; omit both to clear timeout)
- `kick`: `--guild-id`, `--user-id` (\\+ `--reason`)
- `ban`: `--guild-id`, `--user-id` (\\+ `--delete-days`, `--reason`)

  - `timeout` also supports `--reason`

### [​](https://docs.openclaw.ai/cli/message\\#broadcast)  Broadcast

- `broadcast`
  - Channels: any configured channel; use `--channel all` to target all providers
  - Required: `--targets <target...>`
  - Optional: `--message`, `--media`, `--dry-run`

## [​](https://docs.openclaw.ai/cli/message\\#examples)  Examples

Send a Discord reply:

```
openclaw message send --channel discord \\
  --target channel:123 --message \"hi\" --reply-to 456
```

Send a message with semantic buttons:

```
openclaw message send --channel discord \\
  --target channel:123 --message \"Choose:\" \\
  --presentation \'{\"blocks\":[{\"type\":\"buttons\",\"buttons\":[{\"label\":\"Approve\",\"value\":\"approve\",\"style\":\"success\"},{\"label\":\"Decline\",\"value\":\"decline\",\"style\":\"danger\"}]}]}\
```

Core renders the same `presentation` payload into Discord components, Slack blocks, Telegram inline buttons, Mattermost props, or Teams/Feishu cards depending on channel capability. See [Message Presentation](https://docs.openclaw.ai/plugins/message-presentation) for the full contract and fallback rules.Send a richer presentation payload:

```
openclaw message send --channel googlechat --target spaces/AAA... \\
  --message \"Choose:\" \\
  --presentation \'{\"title\":\"Deploy approval\",\"tone\":\"warning\",\"blocks\":[{\"type\":\"text\",\"text\":\"Choose a path\"},{\"type\":\"buttons\",\"buttons\":[{\"label\":\"Approve\",\"value\":\"approve\"},{\"label\":\"Decline\",\"value\":\"decline\"}]}]}\
```

Create a Discord poll:

```
openclaw message poll --channel discord \\
  --target channel:123 \\
  --poll-question \"Snack?\" \\
  --poll-option Pizza --poll-option Sushi \\
  --poll-multi --poll-duration-hours 48
```

Create a Telegram poll (auto-close in 2 minutes):

```
openclaw message poll --channel telegram \\
  --target @mychat \\
  --poll-question \"Lunch?\" \\
  --poll-option Pizza --poll-option Sushi \\
  --poll-duration-seconds 120 --silent
```

Send a Teams proactive message:

```
openclaw message send --channel msteams \\
  --target conversation:19:abc@thread.tacv2 --message \"hi\"
```

Create a Teams poll:

```
openclaw message poll --channel msteams \\
  --target conversation:19:abc@thread.tacv2 \\
  --poll-question \"Lunch?\" \\
  --poll-option Pizza --poll-option Sushi
```

React in Slack:

```
openclaw message react --channel slack \\
  --target C123 --message-id 456 --emoji \"✅\"
```

React in a Signal group:

```
openclaw message react --channel signal \\
  --target signal:group:abc123 --message-id 1737630212345 \\
  --emoji \"✅\" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Send Telegram inline buttons through generic presentation:

```
openclaw message send --channel telegram --target @mychat --message \"Choose:\" \\
  --presentation \'{\"blocks\":[{\"type\":\"buttons\",\"buttons\":[{\"label\":\"Yes\",\"value\":\"cmd:yes\"},{\"label\":\"No\",\"value\":\"cmd:no\"}]}]}\
```

Send a Teams card through generic presentation:

```
openclaw message send --channel msteams \\
  --target conversation:19:abc@thread.tacv2 \\
  --presentation \'{\"title\":\"Status update\",\"blocks\":[{\"type\":\"text\",\"text\":\"Build completed\"}]}\
```

Send a Telegram image as a document to avoid compression:

```
openclaw message send --channel telegram --target @mychat \\
  --media ./diagram.png --force-document
```

## [​](https://docs.openclaw.ai/cli/message\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Agent send](https://docs.openclaw.ai/tools/agent-send)

[Memory](https://docs.openclaw.ai/cli/memory) [Models](https://docs.openclaw.ai/cli/models)

Ctrl+I

---

## DNS - OpenClaw
**Source:** https://docs.openclaw.ai/cli/dns

[Skip to main content](https://docs.openclaw.ai/cli/dns#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Utility

DNS

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw dns](https://docs.openclaw.ai/cli/dns#openclaw-dns)
- [Setup](https://docs.openclaw.ai/cli/dns#setup)
- [dns setup](https://docs.openclaw.ai/cli/dns#dns-setup)
- [Related](https://docs.openclaw.ai/cli/dns#related)

# [​](https://docs.openclaw.ai/cli/dns\\#openclaw-dns)  `openclaw dns`

DNS helpers for wide-area discovery (Tailscale + CoreDNS). Currently focused on macOS + Homebrew CoreDNS.Related:

- Gateway discovery: [Discovery](https://docs.openclaw.ai/gateway/discovery)
- Wide-area discovery config: [Configuration](https://docs.openclaw.ai/gateway/configuration)

## [​](https://docs.openclaw.ai/cli/dns\\#setup)  Setup

```
openclaw dns setup
openclaw dns setup --domain openclaw.internal
openclaw dns setup --apply
```

## [​](https://docs.openclaw.ai/cli/dns\\#dns-setup)  `dns setup`

Plan or apply CoreDNS setup for unicast DNS-SD discovery.Options:

- `--domain <domain>`: wide-area discovery domain (for example `openclaw.internal`)
- `--apply`: install or update CoreDNS config and restart the service (requires sudo; macOS only)

What it shows:

- resolved discovery domain
- zone file path
- current tailnet IPs
- recommended `openclaw.json` discovery config
- the Tailscale Split DNS nameserver/domain values to set

Notes:

- Without `--apply`, the command is a planning helper only and prints the recommended setup.
- If `--domain` is omitted, OpenClaw uses `discovery.wideArea.domain` from config.
- `--apply` currently supports macOS only and expects Homebrew CoreDNS.
- `--apply` bootstraps the zone file if needed, ensures the CoreDNS import stanza exists, and restarts the `coredns` brew service.

## [​](https://docs.openclaw.ai/cli/dns\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Discovery](https://docs.openclaw.ai/gateway/discovery)

[Completion](https://docs.openclaw.ai/cli/completion) [Docs](https://docs.openclaw.ai/cli/docs)

Ctrl+I

---

## Gateway
**Source:** https://docs.openclaw.ai/cli/gateway

[Skip to main content](https://docs.openclaw.ai/cli/gateway#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Gateway and service

Gateway

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Run the Gateway](https://docs.openclaw.ai/cli/gateway#run-the-gateway)
- [Options](https://docs.openclaw.ai/cli/gateway#options)
- [Startup profiling](https://docs.openclaw.ai/cli/gateway#startup-profiling)
- [Query a running Gateway](https://docs.openclaw.ai/cli/gateway#query-a-running-gateway)
- [gateway health](https://docs.openclaw.ai/cli/gateway#gateway-health)
- [gateway usage-cost](https://docs.openclaw.ai/cli/gateway#gateway-usage-cost)
- [gateway stability](https://docs.openclaw.ai/cli/gateway#gateway-stability)
- [gateway diagnostics export](https://docs.openclaw.ai/cli/gateway#gateway-diagnostics-export)
- [gateway status](https://docs.openclaw.ai/cli/gateway#gateway-status)
- [gateway probe](https://docs.openclaw.ai/cli/gateway#gateway-probe)
- [Remote over SSH (Mac app parity)](https://docs.openclaw.ai/cli/gateway#remote-over-ssh-mac-app-parity)
- [gateway call <method>](https://docs.openclaw.ai/cli/gateway#gateway-call-%3Cmethod%3E)
- [Manage the Gateway service](https://docs.openclaw.ai/cli/gateway#manage-the-gateway-service)
- [Install with a wrapper](https://docs.openclaw.ai/cli/gateway#install-with-a-wrapper)
- [Discover gateways (Bonjour)](https://docs.openclaw.ai/cli/gateway#discover-gateways-bonjour)
- [gateway discover](https://docs.openclaw.ai/cli/gateway#gateway-discover)
- [Related](https://docs.openclaw.ai/cli/gateway#related)

The Gateway is OpenClaw’s WebSocket server (channels, nodes, sessions, hooks). Subcommands in this page live under `openclaw gateway …`.

[**Bonjour discovery** \\\\\n\\\\\nLocal mDNS + wide-area DNS-SD setup.](https://docs.openclaw.ai/gateway/bonjour)

[**Discovery overview** \\\\\n\\\\\nHow OpenClaw advertises and finds gateways.](https://docs.openclaw.ai/gateway/discovery)

[**Configuration** \\\\\n\\\\\nTop-level gateway config keys.](https://docs.openclaw.ai/gateway/configuration)

## [​](https://docs.openclaw.ai/cli/gateway\\#run-the-gateway)  Run the Gateway

Run a local Gateway process:

```
openclaw gateway
```

Foreground alias:

```
openclaw gateway run
```

Startup behavior

- By default, the Gateway refuses to start unless `gateway.mode=local` is set in `~/.openclaw/openclaw.json`. Use `--allow-unconfigured` for ad-hoc/dev runs.
- `openclaw onboard --mode local` and `openclaw setup` are expected to write `gateway.mode=local`. If the file exists but `gateway.mode` is missing, treat that as a broken or clobbered config and repair it instead of assuming local mode implicitly.
- If the file exists and `gateway.mode` is missing, the Gateway treats that as suspicious config damage and refuses to “guess local” for you.
- Binding beyond loopback without auth is blocked (safety guardrail).
- `SIGUSR1` triggers an in-process restart when authorized (`commands.restart` is enabled by default; set `commands.restart: false` to block manual restart, while gateway tool/config apply/update remain allowed).
- `SIGINT`/`SIGTERM` handlers stop the gateway process, but they don’t restore any custom terminal state. If you wrap the CLI with a TUI or raw-mode input, restore the terminal before exit.

### [​](https://docs.openclaw.ai/cli/gateway\\#options)  Options

[​](https://docs.openclaw.ai/cli/gateway#param-port-port)

--port <port>

number

WebSocket port (default comes from config/env; usually `18789`).

[​](https://docs.openclaw.ai/cli/gateway#param-bind-loopback-lan-tailnet-auto-custom)

--bind <loopback\\|lan\\|tailnet\\|auto\\|custom>

string

Listener bind mode.

[​](https://docs.openclaw.ai/cli/gateway#param-auth-token-password)

--auth <token\\|password>

string

Auth mode override.

[​](https://docs.openclaw.ai/cli/gateway#param-token-token)

--token <token>

string

Token override (also sets `OPENCLAW_GATEWAY_TOKEN` for the process).

[​](https://docs.openclaw.ai/cli/gateway#param-password-password)

--password <password>

string

Password override.

[​](https://docs.openclaw.ai/cli/gateway#param-password-file-path)

--password-file <path>

string

Read the gateway password from a file.

[​](https://docs.openclaw.ai/cli/gateway#param-tailscale-off-serve-funnel)

--tailscale <off\\|serve\\|funnel>

string

Expose the Gateway via Tailscale.

[​](https://docs.openclaw.ai/cli/gateway#param-tailscale-reset-on-exit)

--tailscale-reset-on-exit

boolean

Reset Tailscale serve/funnel config on shutdown.

[​](https://docs.openclaw.ai/cli/gateway#param-allow-unconfigured)

--allow-unconfigured

boolean

Allow gateway start without `gateway.mode=local` in config. Bypasses the startup guard for ad-hoc/dev bootstrap only; does not write or repair the config file.

[​](https://docs.openclaw.ai/cli/gateway#param-dev)

--dev

boolean

Create a dev config + workspace if missing (skips BOOTSTRAP.md).

[​](https://docs.openclaw.ai/cli/gateway#param-reset)

--reset

boolean

Reset dev config + credentials + sessions + workspace (requires `--dev`).

[​](https://docs.openclaw.ai/cli/gateway#param-force)

--force

boolean

Kill any existing listener on the selected port before starting.

[​](https://docs.openclaw.ai/cli/gateway#param-verbose)

--verbose

boolean

Verbose logs.

[​](https://docs.openclaw.ai/cli/gateway#param-cli-backend-logs)

--cli-backend-logs

boolean

Only show CLI backend logs in the console (and enable stdout/stderr).

[​](https://docs.openclaw.ai/cli/gateway#param-ws-log-auto-full-compact)

--ws-log <auto\\|full\\|compact>

string

default:\"auto\"\n
Websocket log style.

[​](https://docs.openclaw.ai/cli/gateway#param-compact)

--compact

boolean

Alias for `--ws-log compact`.

[​](https://docs.openclaw.ai/cli/gateway#param-raw-stream)

--raw-stream

boolean

Log raw model stream events to jsonl.

[​](https://docs.openclaw.ai/cli/gateway#param-raw-stream-path-path)

--raw-stream-path <path>

string

Raw stream jsonl path.

Inline `--password` can be exposed in local process listings. Prefer `--password-file`, env, or a SecretRef-backed `gateway.auth.password`.

### [​](https://docs.openclaw.ai/cli/gateway\\#startup-profiling)  Startup profiling

- Set `OPENCLAW_GATEWAY_STARTUP_TRACE=1` to log phase timings during Gateway startup, including per-phase `eventLoopMax` delay and plugin lookup-table timings for installed-index, manifest registry, startup planning, and owner-map work.
- Run `pnpm test:startup:gateway -- --runs 5 --warmup 1` to benchmark Gateway startup. The benchmark records first process output, `/healthz`, `/readyz`, startup trace timings, event-loop delay, and plugin lookup-table timing details.

## [​](https://docs.openclaw.ai/cli/gateway\\#query-a-running-gateway)  Query a running Gateway

All query commands use WebSocket RPC.

- Output modes

- Shared options


- Default: human-readable (colored in TTY).
- `--json`: machine-readable JSON (no styling/spinner).
- `--no-color` (or `NO_COLOR=1`): disable ANSI while keeping human layout.

- `--url <url>`: Gateway WebSocket URL.
- `--token <token>`: Gateway token.
- `--password <password>`: Gateway password.
- `--timeout <ms>`: timeout/budget (varies per command).
- `--expect-final`: wait for a “final” response (agent calls).

When you set `--url`, the CLI does not fall back to config or environment credentials. Pass `--token` or `--password` explicitly. Missing explicit credentials is an error.

### [​](https://docs.openclaw.ai/cli/gateway\\#gateway-health)  `gateway health`

```
openclaw gateway health --url ws://127.0.0.1:18789
```

The HTTP `/healthz` endpoint is a liveness probe: it returns once the server can answer HTTP. The HTTP `/readyz` endpoint is stricter and stays red while startup sidecars, channels, or configured hooks are still settling. Local or authenticated detailed readiness responses include an `eventLoop` diagnostic block with event-loop delay, event-loop utilization, CPU core ratio, and a `degraded` flag.

### [​](https://docs.openclaw.ai/cli/gateway\\#gateway-usage-cost)  `gateway usage-cost`

Fetch usage-cost summaries from session logs.

```
openclaw gateway usage-cost
openclaw gateway usage-cost --days 7
openclaw gateway usage-cost --json
```

[​](https://docs.openclaw.ai/cli/gateway#param-days-days)

--days <days>

number

default:\"30\"\n
Number of days to include.

### [​](https://docs.openclaw.ai/cli/gateway\\#gateway-stability)  `gateway stability`

Fetch the recent diagnostic stability recorder from a running Gateway.

```
openclaw gateway stability
openclaw gateway stability --type payload.large
openclaw gateway stability --bundle latest
openclaw gateway stability --bundle latest --export
openclaw gateway stability --json
```

[​](https://docs.openclaw.ai/cli/gateway#param-limit-limit)

--limit <limit>

number

default:\"25\"\n
Maximum number of recent events to include (max `1000`).

[​](https://docs.openclaw.ai/cli/gateway#param-type-type)

--type <type>

string

Filter by diagnostic event type, such as `payload.large` or `diagnostic.memory.pressure`.

[​](https://docs.openclaw.ai/cli/gateway#param-since-seq-seq)

--since-seq <seq>

number

Include only events after a diagnostic sequence number.

[​](https://docs.openclaw.ai/cli/gateway#param-bundle-path)

--bundle \\[path\\]

string

Read a persisted stability bundle instead of calling the running Gateway. Use `--bundle latest` (or just `--bundle`) for the newest bundle under the state directory, or pass a bundle JSON path directly.

[​](https://docs.openclaw.ai/cli/gateway#param-export)

--export

boolean

Write a shareable support diagnostics zip instead of printing stability details.

[​](https://docs.openclaw.ai/cli/gateway#param-output-path)

--output <path>

string

Output path for `--export`.

Privacy and bundle behavior

- Records keep operational metadata: event names, counts, byte sizes, memory readings, queue/session state, channel/plugin names, and redacted session summaries. They do not keep chat text, webhook bodies, tool outputs, raw request or response bodies, tokens, cookies, secret values, hostnames, or raw session ids. Set `diagnostics.enabled: false` to disable the recorder entirely.
- On fatal Gateway exits, shutdown timeouts, and restart startup failures, OpenClaw writes the same diagnostic snapshot to `~/.openclaw/logs/stability/openclaw-stability-*.json` when the recorder has events. Inspect the newest bundle with `openclaw gateway stability --bundle latest`; `--limit`, `--type`, and `--since-seq` also apply to bundle output.

### [​](https://docs.openclaw.ai/cli/gateway\\#gateway-diagnostics-export)  `gateway diagnostics export`

Write a local diagnostics zip that is designed to attach to bug reports. For the privacy model and bundle contents, see [Diagnostics Export](https://docs.openclaw.ai/gateway/diagnostics).

```
openclaw gateway diagnostics export
openclaw gateway diagnostics export --output openclaw-diagnostics.zip
openclaw gateway diagnostics export --json
```

[​](https://docs.openclaw.ai/cli/gateway#param-output-path-1)

--output <path>

string

Output zip path. Defaults to a support export under the state directory.

[​](https://docs.openclaw.ai/cli/gateway#param-log-lines-count)

--log-lines <count>

number

default:\"5000\"\n
Maximum sanitized log lines to include.

[​](https://docs.openclaw.ai/cli/gateway#param-log-bytes-bytes)

--log-bytes <bytes>

number

default:\"1000000\"\n
Maximum log bytes to inspect.

[​](https://docs.openclaw.ai/cli/gateway#param-url-url)

--url <url>

string

Gateway WebSocket URL for the health snapshot.

[​](https://docs.openclaw.ai/cli/gateway#param-token-token-1)

--token <token>

string

Gateway token for the health snapshot.

[​](https://docs.openclaw.ai/cli/gateway#param-password-password-1)

--password <password>

string

Gateway password for the health snapshot.

[​](https://docs.openclaw.ai/cli/gateway#param-timeout-ms)

--timeout <ms>

number

default:\"3000\"\n
Status/health snapshot timeout.

[​](https://docs.openclaw.ai/cli/gateway#param-no-stability-bundle)

--no-stability-bundle

boolean

Skip persisted stability bundle lookup.

[​](https://docs.openclaw.ai/cli/gateway#param-json)

--json

boolean

Print the written path, size, and manifest as JSON.

The export contains a manifest, a Markdown summary, config shape, sanitized config details, sanitized log summaries, sanitized Gateway status/health snapshots, and the newest stability bundle when one exists.It is meant to be shared. It keeps operational details that help debugging, such as safe OpenClaw log fields, subsystem names, status codes, durations, configured modes, ports, plugin ids, provider ids, non-secret feature settings, and redacted operational log messages. It omits or redacts chat text, webhook bodies, tool outputs, credentials, cookies, account/message identifiers, prompt/instruction text, hostnames, and secret values. When a LogTape-style message looks like user/chat/tool payload text, the export keeps only that a message was omitted plus its byte count.\n\n### [​](https://docs.openclaw.ai/cli/gateway\\#gateway-status)  `gateway status`

`gateway status` shows the Gateway service (launchd/systemd/schtasks) plus an optional probe of connectivity/auth capability.

```
openclaw gateway status
openclaw gateway status --json
openclaw gateway status --require-rpc
```

[​](https://docs.openclaw.ai/cli/gateway#param-url-url-1)

--url <url>

string

Add an explicit probe target. Configured remote + localhost are still probed.

[​](https://docs.openclaw.ai/cli/gateway#param-token-token-2)

--token <token>

string

Token auth for the probe.

[​](https://docs.openclaw.ai/cli/gateway#param-password-password-2)

--password <password>

string

Password auth for the probe.

[​](https://docs.openclaw.ai/cli/gateway#param-timeout-ms-1)

--timeout <ms>

number

default:\"10000\"\n
Probe timeout.

[​](https://docs.openclaw.ai/cli/gateway#param-no-probe)

--no-probe

boolean

Skip the connectivity probe (service-only view).

[​](https://docs.openclaw.ai/cli/gateway#param-deep)

--deep

boolean

Scan system-level services too.

[​](https://docs.openclaw.ai/cli/gateway#param-require-rpc)

--require-rpc

boolean

Upgrade the default connectivity probe to a read probe and exit non-zero when that read probe fails. Cannot be combined with `--no-probe`.

Status semantics

- `gateway status` stays available for diagnostics even when the local CLI config is missing or invalid.
- Default `gateway status` proves service state, WebSocket connect, and the auth capability visible at handshake time. It does not prove read/write/admin operations.
- Diagnostic probes are non-mutating for first-time device auth: they reuse an existing cached device token when one exists, but they do not create a new CLI device identity or read-only device pairing record just to check status.
- `gateway status` resolves configured auth SecretRefs for probe auth when possible.
- If a required auth SecretRef is unresolved in this command path, `gateway status --json` reports `rpc.authWarning` when probe connectivity/auth fails; pass `--token`/`--password` explicitly or resolve the secret source first.
- If the probe succeeds, unresolved auth-ref warnings are suppressed to avoid false positives.
- Use `--require-rpc` in scripts and automation when a listening service is not enough and you need read-scope RPC calls to be healthy too.
- `--deep` adds a best-effort scan for extra launchd/systemd/schtasks installs. When multiple gateway-like services are detected, human output prints cleanup hints and warns that most setups should run one gateway per machine.
- Human output includes the resolved file log path plus the CLI-vs-service config paths/validity snapshot to help diagnose profile or state-dir drift.

Linux systemd auth-drift checks

- On Linux systemd installs, service auth drift checks read both `Environment=` and `EnvironmentFile=` values from the unit (including `%h`, quoted paths, multiple files, and optional `-` files).
- Drift checks resolve `gateway.auth.token` SecretRefs using merged runtime env (service command env first, then process env fallback).
- If token auth is not effectively active (explicit `gateway.auth.mode` of `password`/`none`/`trusted-proxy`, or mode unset where password can win and no token candidate can win), token-drift checks skip config token resolution.

### [​](https://docs.openclaw.ai/cli/gateway\\#gateway-probe)  `gateway probe`

`gateway probe` is the “debug everything” command. It always probes:

- your configured remote gateway (if set), and
- localhost (loopback) **even if remote is configured**.

If you pass `--url`, that explicit target is added ahead of both. Human output labels the targets as:

- `URL (explicit)`
- `Remote (configured)` or `Remote (configured, inactive)`
- `Local loopback`

If multiple gateways are reachable, it prints all of them. Multiple gateways are supported when you use isolated profiles/ports (e.g., a rescue bot), but most installs still run a single gateway.

```
openclaw gateway probe
openclaw gateway probe --json
```

Interpretation

- `Reachable: yes` means at least one target accepted a WebSocket connect.
- `Capability: read-only|write-capable|admin-capable|pairing-pending|connect-only` reports what the probe could prove about auth. It is separate from reachability.
- `Read probe: ok` means read-scope detail RPC calls (`health`/`status`/`system-presence`/`config.get`) also succeeded.
- `Read probe: limited - missing scope: operator.read` means connect succeeded but read-scope RPC is limited. This is reported as **degraded** reachability, not full failure.
- Like `gateway status`, probe reuses existing cached device auth but does not create first-time device identity or pairing state.
- Exit code is non-zero only when no probed target is reachable.

JSON output

Top level:

- `ok`: at least one target is reachable.
- `degraded`: at least one target had scope-limited detail RPC.
- `capability`: best capability seen across reachable targets (`read_only`, `write_capable`, `admin_capable`, `pairing_pending`, `connected_no_operator_scope`, or `unknown`).
- `primaryTargetId`: best target to treat as the active winner in this order: explicit...(content truncated)

---

## Configure - OpenClaw
**Source:** https://docs.openclaw.ai/cli/configure

[Skip to main content](https://docs.openclaw.ai/cli/configure#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Configuration

Configure

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw configure](https://docs.openclaw.ai/cli/configure#openclaw-configure)
- [Options](https://docs.openclaw.ai/cli/configure#options)
- [Examples](https://docs.openclaw.ai/cli/configure#examples)
- [Related](https://docs.openclaw.ai/cli/configure#related)

# [​](https://docs.openclaw.ai/cli/configure\\#openclaw-configure)  `openclaw configure`

Interactive prompt to set up credentials, devices, and agent defaults.

The **Model** section includes a multi-select for the `agents.defaults.models` allowlist (what shows up in `/model` and the model picker). Provider-scoped setup choices merge their selected models into the existing allowlist instead of replacing unrelated providers already in the config. Re-running provider auth from configure preserves an existing `agents.defaults.model.primary`. Use `openclaw models auth login --provider <id> --set-default` or `openclaw models set <model>` when you intentionally want to change the default model.

When configure starts from a provider auth choice, the default-model and allowlist pickers prefer that provider automatically. For paired providers such as Volcengine and BytePlus, the same preference also matches their coding-plan variants (`volcengine-plan/*`, `byteplus-plan/*`). If the preferred-provider filter would produce an empty list, configure falls back to the unfiltered catalog instead of showing a blank picker.

`openclaw config` without a subcommand opens the same wizard. Use `openclaw config get|set|unset` for non-interactive edits.

For web search, `openclaw configure --section web` lets you choose a provider
and configure its credentials. Some providers also show provider-specific
follow-up prompts:

- **Grok** can offer optional `x_search` setup with the same `XAI_API_KEY` and
let you pick an `x_search` model.
- **Kimi** can ask for the Moonshot API region (`api.moonshot.ai` vs
`api.moonshot.cn`) and the default Kimi web-search model.

Related:

- Gateway configuration reference: [Configuration](https://docs.openclaw.ai/gateway/configuration)
- Config CLI: [Config](https://docs.openclaw.ai/cli/config)

## [​](https://docs.openclaw.ai/cli/configure\\#options)  Options

- `--section <section>`: repeatable section filter

Available sections:

- `workspace`
- `model`
- `web`
- `gateway`
- `daemon`
- `channels`
- `plugins`
- `skills`
- `health`

Notes:

- Choosing where the Gateway runs always updates `gateway.mode`. You can select “Continue” without other sections if that is all you need.
- Channel-oriented services (Slack/Discord/Matrix/Microsoft Teams) prompt for channel/room allowlists during setup. You can enter names or IDs; the wizard resolves names to IDs when possible.
- If you run the daemon install step, token auth requires a token, and `gateway.auth.token` is SecretRef-managed, configure validates the SecretRef but does not persist resolved plaintext token values into supervisor service environment metadata.
- If token auth requires a token and the configured token SecretRef is unresolved, configure blocks daemon install with actionable remediation guidance.
- If both `gateway.auth.token` and `gateway.auth.password` are configured and `gateway.auth.mode` is unset, configure blocks daemon install until mode is set explicitly.

## [​](https://docs.openclaw.ai/cli/configure\\#examples)  Examples

```
openclaw configure
openclaw configure --section web
openclaw configure --section model --section channels
openclaw configure --section gateway --section daemon
```

## [​](https://docs.openclaw.ai/cli/configure\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [Configuration](https://docs.openclaw.ai/gateway/configuration)

[Config](https://docs.openclaw.ai/cli/config) [Webhooks](https://docs.openclaw.ai/cli/webhooks)

Ctrl+I

---

## Onboard - OpenClaw
**Source:** https://docs.openclaw.ai/cli/onboard

[Skip to main content](https://docs.openclaw.ai/cli/onboard#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Gateway and service

Onboard

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw onboard](https://docs.openclaw.ai/cli/onboard#openclaw-onboard)
- [Related guides](https://docs.openclaw.ai/cli/onboard#related-guides)
- [Examples](https://docs.openclaw.ai/cli/onboard#examples)
- [Non-interactive Z.AI endpoint choices](https://docs.openclaw.ai/cli/onboard#non-interactive-z-ai-endpoint-choices)
- [Flow notes](https://docs.openclaw.ai/cli/onboard#flow-notes)
- [Common follow-up commands](https://docs.openclaw.ai/cli/onboard#common-follow-up-commands)

# [​](https://docs.openclaw.ai/cli/onboard\\#openclaw-onboard)  `openclaw onboard`

Interactive onboarding for local or remote Gateway setup.

## [​](https://docs.openclaw.ai/cli/onboard\\#related-guides)  Related guides

[**CLI onboarding hub** \\\\\n\\\\\nWalkthrough of the interactive CLI flow.](https://docs.openclaw.ai/start/wizard)

[**Onboarding overview** \\\\\n\\\\\nHow OpenClaw onboarding fits together.](https://docs.openclaw.ai/start/onboarding-overview)

[**CLI setup reference** \\\\\n\\\\\nOutputs, internals, and per-step behavior.](https://docs.openclaw.ai/start/wizard-cli-reference)

[**CLI automation** \\\\\n\\\\\nNon-interactive flags and scripted setups.](https://docs.openclaw.ai/start/wizard-cli-automation)

[**macOS app onboarding** \\\\\n\\\\\nOnboarding flow for the macOS menu bar app.](https://docs.openclaw.ai/start/onboarding)

## [​](https://docs.openclaw.ai/cli/onboard\\#examples)  Examples

```
openclaw onboard
openclaw onboard --modern
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --flow import
openclaw onboard --import-from hermes --import-source ~/.hermes
openclaw onboard --skip-bootstrap
openclaw onboard --mode remote --remote-url wss://gateway-host:18789
```

`--flow import` uses plugin-owned migration providers such as Hermes. It only runs against a fresh OpenClaw setup; if existing config, credentials, sessions, or workspace memory/identity files are present, reset or choose a fresh setup before importing.`--modern` starts the Crestodian conversational onboarding preview. Without
`--modern`, `openclaw onboard` keeps the classic onboarding flow.For plaintext private-network `ws://` targets (trusted networks only), set
`OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` in the onboarding process environment.
There is no `openclaw.json` equivalent for this client-side transport
break-glass.Non-interactive custom provider:

```
openclaw onboard --non-interactive \\\
  --auth-choice custom-api-key \\\
  --custom-base-url \"https://llm.example.com/v1\" \\\
  --custom-model-id \"foo-large\" \\\
  --custom-api-key \"$CUSTOM_API_KEY\" \\\
  --secret-input-mode plaintext \\\
  --custom-compatibility openai
```

`--custom-api-key` is optional in non-interactive mode. If omitted, onboarding checks `CUSTOM_API_KEY`.LM Studio also supports a provider-specific key flag in non-interactive mode:

```
openclaw onboard --non-interactive \\\
  --auth-choice lmstudio \\\
  --custom-base-url \"http://localhost:1234/v1\" \\\
  --custom-model-id \"qwen/qwen3.5-9b\" \\\
  --lmstudio-api-key \"$LM_API_TOKEN\" \\\
  --accept-risk
```

Non-interactive Ollama:

```
openclaw onboard --non-interactive \\\
  --auth-choice ollama \\\
  --custom-base-url \"http://ollama-host:11434\" \\\
  --custom-model-id \"qwen3.5:27b\" \\\
  --accept-risk
```

`--custom-base-url` defaults to `http://127.0.0.1:11434`. `--custom-model-id` is optional; if omitted, onboarding uses Ollama’s suggested defaults. Cloud model IDs such as `kimi-k2.5:cloud` also work here.Store provider keys as refs instead of plaintext:

```
openclaw onboard --non-interactive \\\
  --auth-choice openai-api-key \\\
  --secret-input-mode ref \\\
  --accept-risk
```

With `--secret-input-mode ref`, onboarding writes env-backed refs instead of plaintext key values.
For auth-profile backed providers this writes `keyRef` entries; for custom providers this writes `models.providers.<id>.apiKey` as an env ref (for example `{ source: \"env\", provider: \"default\", id: \"CUSTOM_API_KEY\" }`).Non-interactive `ref` mode contract:

- Set the provider env var in the onboarding process environment (for example `OPENAI_API_KEY`).
- Do not pass inline key flags (for example `--openai-api-key`) unless that env var is also set.
- If an inline key flag is passed without the required env var, onboarding fails fast with guidance.

Gateway token options in non-interactive mode:

- `--gateway-auth token --gateway-token <token>` stores a plaintext token.
- `--gateway-auth token --gateway-token-ref-env <name>` stores `gateway.auth.token` as an env SecretRef.
- `--gateway-token` and `--gateway-token-ref-env` are mutually exclusive.
- `--gateway-token-ref-env` requires a non-empty env var in the onboarding process environment.
- With `--install-daemon`, when token auth requires a token, SecretRef-managed gateway tokens are validated but not persisted as resolved plaintext in supervisor service environment metadata.
- With `--install-daemon`, if token mode requires a token and the configured token SecretRef is unresolved, onboarding fails closed with remediation guidance.
- With `--install-daemon`, if both `gateway.auth.token` and `gateway.auth.password` are configured and `gateway.auth.mode` is unset, onboarding blocks install until mode is set explicitly.
- Local onboarding writes `gateway.mode=\"local\"` into the config. If a later config file is missing `gateway.mode`, treat that as config damage or an incomplete manual edit, not as a valid local-mode shortcut.
- `--allow-unconfigured` is a separate gateway runtime escape hatch. It does not mean onboarding may omit `gateway.mode`.

Example:

```
export OPENCLAW_GATEWAY_TOKEN=\"your-token\"
openclaw onboard --non-interactive \\\
  --mode local \\\
  --auth-choice skip \\\
  --gateway-auth token \\\
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN \\\
  --accept-risk
```

Non-interactive local gateway health:

- Unless you pass `--skip-health`, onboarding waits for a reachable local gateway before it exits successfully.
- `--install-daemon` starts the managed gateway install path first. Without it, you must already have a local gateway running, for example `openclaw gateway run`.
- If you only want config/workspace/bootstrap writes in automation, use `--skip-health`.
- If you manage workspace files yourself, pass `--skip-bootstrap` to set `agents.defaults.skipBootstrap: true` and skip creating `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, and `BOOTSTRAP.md`.
- On native Windows, `--install-daemon` tries Scheduled Tasks first and falls back to a per-user Startup-folder login item if task creation is denied.

Interactive onboarding behavior with reference mode:

- Choose **Use secret reference** when prompted.
- Then choose either:
  - Environment variable
  - Configured secret provider (`file` or `exec`)
- Onboarding performs a fast preflight validation before saving the ref.
  - If validation fails, onboarding shows the error and lets you retry.

### [​](https://docs.openclaw.ai/cli/onboard\\#non-interactive-z-ai-endpoint-choices)  Non-interactive Z.AI endpoint choices

`--auth-choice zai-api-key` auto-detects the best Z.AI endpoint for your key (prefers the general API with `zai/glm-5.1`). If you specifically want the GLM Coding Plan endpoints, pick `zai-coding-global` or `zai-coding-cn`.

```
# Promptless endpoint selection
openclaw onboard --non-interactive \\\
  --auth-choice zai-coding-global \\\
  --zai-api-key \"$ZAI_API_KEY\"

# Other Z.AI endpoint choices:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Non-interactive Mistral example:

```
openclaw onboard --non-interactive \\\
  --auth-choice mistral-api-key \\\
  --mistral-api-key \"$MISTRAL_API_KEY\"
```

## [​](https://docs.openclaw.ai/cli/onboard\\#flow-notes)  Flow types

- `quickstart`: minimal prompts, auto-generates a gateway token.
- `manual`: full prompts for port, bind, and auth (alias of `advanced`).
- `import`: runs a detected migration provider, previews the plan, then applies after confirmation.

Provider prefiltering

When an auth choice implies a preferred provider, onboarding prefilters the default-model and allowlist pickers to that provider. For Volcengine and BytePlus, this also matches the coding-plan variants (`volcengine-plan/*`, `byteplus-plan/*`).If the preferred-provider filter yields no loaded models yet, onboarding falls back to the unfiltered catalog instead of leaving the picker empty.

Web-search follow-ups

Some web-search providers trigger provider-specific follow-up prompts:

- **Grok** can offer optional `x_search` setup with the same `XAI_API_KEY` and an `x_search` model choice.
- **Kimi** can ask for the Moonshot API region (`api.moonshot.ai` vs `api.moonshot.cn`) and the default Kimi web-search model.

Other behaviors

- Local onboarding DM scope behavior: [CLI setup reference](https://docs.openclaw.ai/start/wizard-cli-reference#outputs-and-internals).
- Fastest first chat: `openclaw dashboard` (Control UI, no channel setup).
- Custom provider: connect any OpenAI or Anthropic compatible endpoint, including hosted providers not listed. Use Unknown to auto-detect.
- If Hermes state is detected, onboarding offers a migration flow. Use [Migrate](https://docs.openclaw.ai/cli/migrate) for dry-run plans, overwrite mode, reports, and exact mappings.

## [​](https://docs.openclaw.ai/cli/onboard\\#common-follow-up-commands)  Common follow-up commands

```
openclaw configure
openclaw agents add <name>
```

`--json` does not imply non-interactive mode. Use `--non-interactive` for scripts.

[Migrate](https://docs.openclaw.ai/cli/migrate) [Reset](https://docs.openclaw.ai/cli/reset)

Ctrl+I

---

## Plugins
**Source:** https://docs.openclaw.ai/cli/plugins

[Skip to main content](https://docs.openclaw.ai/cli/plugins#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins and skills

Plugins

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Commands](https://docs.openclaw.ai/cli/plugins#commands)
- [Install](https://docs.openclaw.ai/cli/plugins#install)
- [Marketplace shorthand](https://docs.openclaw.ai/cli/plugins#marketplace-shorthand)
- [List](https://docs.openclaw.ai/cli/plugins#list)
- [Plugin index](https://docs.openclaw.ai/cli/plugins#plugin-index)
- [Uninstall](https://docs.openclaw.ai/cli/plugins#uninstall)
- [Update](https://docs.openclaw.ai/cli/plugins#update)
- [Inspect](https://docs.openclaw.ai/cli/plugins#inspect)
- [Doctor](https://docs.openclaw.ai/cli/plugins#doctor)
- [Registry](https://docs.openclaw.ai/cli/plugins#registry)
- [Marketplace](https://docs.openclaw.ai/cli/plugins#marketplace)
- [Related](https://docs.openclaw.ai/cli/plugins#related)

Manage Gateway plugins, hook packs, and compatible bundles.

[**Plugin system** \\\\\n\\\\\nEnd-user guide for installing, enabling, and troubleshooting plugins.](https://docs.openclaw.ai/tools/plugin)

[**Plugin bundles** \\\\\n\\\\\nBundle compatibility model.](https://docs.openclaw.ai/plugins/bundles)

[**Plugin manifest** \\\\\n\\\\\nManifest fields and config schema.](https://docs.openclaw.ai/plugins/manifest)

[**Security** \\\\\n\\\\\nSecurity hardening for plugin installs.](https://docs.openclaw.ai/gateway/security)

## [​](https://docs.openclaw.ai/cli/plugins\\#commands)  Commands

```
openclaw plugins list
openclaw plugins list --enabled
openclaw plugins list --verbose
openclaw plugins list --json
openclaw plugins install <path-or-spec>
openclaw plugins inspect <id>
openclaw plugins inspect <id> --json
openclaw plugins inspect --all
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins registry
openclaw plugins registry --refresh
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id-or-npm-spec>
openclaw plugins update --all
openclaw plugins marketplace list <marketplace>
openclaw plugins marketplace list <marketplace> --json
```

Bundled plugins ship with OpenClaw. Some are enabled by default (for example bundled model providers, bundled speech providers, and the bundled browser plugin); others require `plugins enable`.Native OpenClaw plugins must ship `openclaw.plugin.json` with an inline JSON Schema (`configSchema`, even if empty). Compatible bundles use their own bundle manifests instead.`plugins list` shows `Format: openclaw` or `Format: bundle`. Verbose list/info output also shows the bundle subtype (`codex`, `claude`, or `cursor`) plus detected bundle capabilities.

### [​](https://docs.openclaw.ai/cli/plugins\\#install)  Install

```
openclaw plugins install <package>                      # ClawHub first, then npm
openclaw plugins install clawhub:<package>              # ClawHub only
openclaw plugins install npm:<package>                  # npm only
openclaw plugins install <package> --force              # overwrite existing install
openclaw plugins install <package> --pin                # pin version
openclaw plugins install <package> --dangerously-force-unsafe-install
openclaw plugins install <path>                         # local path
openclaw plugins install <plugin>@<marketplace>         # marketplace
openclaw plugins install <plugin> --marketplace <name>  # marketplace (explicit)
openclaw plugins install <plugin> --marketplace https://github.com/<owner>/<repo>
```

Bare package names are checked against ClawHub first, then npm. Treat plugin installs like running code. Prefer pinned versions.

Config includes and invalid-config recovery

If your `plugins` section is backed by a single-file `$include`, `plugins install/update/enable/disable/uninstall` write through to that included file and leave `openclaw.json` untouched. Root includes, include arrays, and includes with sibling overrides fail closed instead of flattening. See [Config includes](https://docs.openclaw.ai/gateway/configuration) for the supported shapes.If config is invalid during install, `plugins install` normally fails closed and tells you to run `openclaw doctor --fix` first. During Gateway startup, invalid config for one plugin is isolated to that plugin so other channels and plugins can keep running; `openclaw doctor --fix` can quarantine the invalid plugin entry. The only documented install-time exception is a narrow bundled-plugin recovery path for plugins that explicitly opt into `openclaw.install.allowInvalidConfigRecovery`.

--force and reinstall vs update

`--force` reuses the existing install target and overwrites an already-installed plugin or hook pack in place. Use it when you are intentionally reinstalling the same id from a new local path, archive, ClawHub package, or npm artifact. For routine upgrades of an already tracked npm plugin, prefer `openclaw plugins update <id-or-npm-spec>`.If you run `plugins install` for a plugin id that is already installed, OpenClaw stops and points you at `plugins update <id-or-npm-spec>` for a normal upgrade, or at `plugins install <package> --force` when you genuinely want to overwrite the current install from a different source.

--pin scope

`--pin` applies to npm installs only. It is not supported with `--marketplace`, because marketplace installs persist marketplace source metadata instead of an npm spec.

--dangerously-force-unsafe-install

`--dangerously-force-unsafe-install` is a break-glass option for false positives in the built-in dangerous-code scanner. It allows the install to continue even when the built-in scanner reports `critical` findings, but it does **not** bypass plugin `before_install` hook policy blocks and does **not** bypass scan failures.This CLI flag applies to plugin install/update flows. Gateway-backed skill dependency installs use the matching `dangerouslyForceUnsafeInstall` request override, while `openclaw skills install` remains a separate ClawHub skill download/install flow.

Hook packs and npm specs

`plugins install` is also the install surface for hook packs that expose `openclaw.hooks` in `package.json`. Use `openclaw hooks` for filtered hook visibility and per-hook enablement, not package installation.Npm specs are **registry-only** (package name + optional **exact version** or **dist-tag**). Git/URL/file specs and semver ranges are rejected. Dependency installs run project-local with `--ignore-scripts` for safety, even when your shell has global npm install settings.Use `npm:<package>` when you want to skip ClawHub lookup and install directly from npm. Bare package specs still prefer ClawHub and only fall back to npm when ClawHub does not have that package or version.Bare specs and `@latest` stay on the stable track. If npm resolves either of those to a prerelease, OpenClaw stops and asks you to opt in explicitly with a prerelease tag such as `@beta`/`@rc` or an exact prerelease version such as `@1.2.3-beta.4`.If a bare install spec matches a bundled plugin id (for example `diffs`), OpenClaw installs the bundled plugin directly. To install an npm package with the same name, use an explicit scoped spec (for example `@scope/diffs`).

Archives

Supported archives: `.zip`, `.tgz`, `.tar.gz`, `.tar`. Native OpenClaw plugin archives must contain a valid `openclaw.plugin.json` at the extracted plugin root; archives that only contain `package.json` are rejected before OpenClaw writes install records.Claude marketplace installs are also supported.

ClawHub installs use an explicit `clawhub:<package>` locator:

```
openclaw plugins install clawhub:openclaw-codex-app-server
openclaw plugins install clawhub:openclaw-codex-app-server@1.2.3
```

OpenClaw now also prefers ClawHub for bare npm-safe plugin specs. It only falls back to npm if ClawHub does not have that package or version:

```
openclaw plugins install openclaw-codex-app-server
```

Use `npm:` to force npm-only resolution, for example when ClawHub is unreachable or you know the package exists only on npm:

```
openclaw plugins install npm:openclaw-codex-app-server
openclaw plugins install npm:@scope/plugin-name@1.0.1
```

OpenClaw downloads the package archive from ClawHub, checks the advertised plugin API / minimum gateway compatibility, then installs it through the normal archive path. Recorded installs keep their ClawHub source metadata for later updates.
Unversioned ClawHub installs keep an unversioned recorded spec so `openclaw plugins update` can follow newer ClawHub releases; explicit version or tag selectors such as `clawhub:pkg@1.2.3` and `clawhub:pkg@beta` remain pinned to that selector.

#### [​](https://docs.openclaw.ai/cli/plugins\\#marketplace-shorthand)  Marketplace shorthand

Use `plugin@marketplace` shorthand when the marketplace name exists in Claude’s local registry cache at `~/.claude/plugins/known_marketplaces.json`:

```
openclaw plugins marketplace list <marketplace-name>
openclaw plugins install <plugin-name>@<marketplace-name>
```

Use `--marketplace` when you want to pass the marketplace source explicitly:

```
openclaw plugins install <plugin-name> --marketplace <marketplace-name>
openclaw plugins install <plugin-name> --marketplace <owner/repo>
openclaw plugins install <plugin-name> --marketplace https://github.com/<owner>/<repo>
openclaw plugins install <plugin-name> --marketplace ./my-marketplace
```

- Marketplace sources

- Remote marketplace rules


- a Claude known-marketplace name from `~/.claude/plugins/known_marketplaces.json`
- a local marketplace root or `marketplace.json` path
- a GitHub repo shorthand such as `owner/repo`
- a GitHub repo URL such as `https://github.com/owner/repo`
- a git URL

For remote marketplaces loaded from GitHub or git, plugin entries must stay inside the cloned marketplace repo. OpenClaw accepts relative path sources from that repo and rejects HTTP(S), absolute-path, git, GitHub, and other non-path plugin sources from remote manifests.

For local paths and archives, OpenClaw auto-detects:

- native OpenClaw plugins (`openclaw.plugin.json`)
- Codex-compatible bundles (`.codex-plugin/plugin.json`)
- Claude-compatible bundles (`.claude-plugin/plugin.json` or the default Claude component layout)
- Cursor-compatible bundles (`.cursor-plugin/plugin.json`)

Compatible bundles install into the normal plugin root and participate in the same list/info/enable/disable flow. Today, bundle skills, Claude command-skills, Claude `settings.json` defaults, Claude `.lsp.json` / manifest-declared `lspServers` defaults, Cursor command-skills, and compatible Codex hook directories are supported; other detected bundle capabilities are shown in diagnostics/info but are not yet wired into runtime execution.

### [​](https://docs.openclaw.ai/cli/plugins\\#list)  List

```
openclaw plugins list
openclaw plugins list --enabled
openclaw plugins list --verbose
openclaw plugins list --json
```

[​](https://docs.openclaw.ai/cli/plugins#param-enabled)

--enabled

boolean

Show only enabled plugins.

[​](https://docs.openclaw.ai/cli/plugins#param-verbose)

--verbose

boolean

Switch from the table view to per-plugin detail lines with source/origin/version/activation metadata.

[​](https://docs.openclaw.ai/cli/plugins#param-json)

--json

boolean

Machine-readable inventory plus registry diagnostics.

`plugins list` reads the persisted local plugin registry first, with a manifest-only derived fallback when the registry is missing or invalid. It is useful for checking whether a plugin is installed, enabled, and visible to cold startup planning, but it is not a live runtime probe of an already-running Gateway process. After changing plugin code, enablement, hook policy, or `plugins.load.paths`, restart the Gateway that serves the channel before expecting new `register(api)` code or hooks to run. For remote/container deployments, verify you are restarting the actual `openclaw gateway run` child, not only a wrapper process.

For bundled plugin work inside a packaged Docker image, bind-mount the plugin\nsource directory over the matching packaged source path, such as\n`/app/extensions/synology-chat`. OpenClaw will discover that mounted source\noverlay before `/app/dist/extensions/synology-chat`; a plain copied source\ndirectory remains inert so normal packaged installs still use compiled dist.For runtime hook debugging:

- `openclaw plugins inspect <id> --json` shows registered hooks and diagnostics from a module-loaded inspection pass.
- `openclaw gateway status --deep --require-rpc` confirms the reachable Gateway, service/process hints, config path, and RPC health.
- Non-bundled conversation hooks (`llm_input`, `llm_output`, `before_agent_finalize`, `agent_end`) require `plugins.entries.<id>.hooks.allowConversationAccess=true`.

Use `--link` to avoid copying a local directory (adds to `plugins.load.paths`):

```
openclaw plugins install -l ./my-plugin
```

`--force` is not supported with `--link` because linked installs reuse the source path instead of copying over a managed install target.Use `--pin` on npm installs to save the resolved exact spec (`name@version`) in the managed plugin index while keeping the default behavior unpinned.

### [​](https://docs.openclaw.ai/cli/plugins\\#plugin-index)  Plugin index

Plugin install metadata is machine-managed state, not user config. Installs and updates write it to `plugins/installs.json` under the active OpenClaw state directory. Its top-level `installRecords` map is the durable source of install metadata, including records for broken or missing plugin manifests. The `plugins` array is the manifest-derived cold registry cache. The file includes a do-not-edit warning and is used by `openclaw plugins update`, uninstall, diagnostics, and the cold plugin registry.When OpenClaw sees shipped legacy `plugins.installs` records in config, it moves them into the plugin index and removes the config key; if either write fails, the config records are kept so the install metadata is not lost.

### [​](https://docs.openclaw.ai/cli/plugins\\#uninstall)  Uninstall

```
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` removes plugin records from `plugins.entries`, the persisted plugin index, plugin allow/deny list entries, and linked `plugins.load.paths` entries when applicable. Unless `--keep-files` is set, uninstall also removes the tracked managed install directory when it is inside OpenClaw’s plugin extensions root. For active memory plugins, the memory slot resets to `memory-core`.

`--keep-config` is supported as a deprecated alias for `--keep-files`.

### [​](https://docs.openclaw.ai/cli/plugins\\#update)  Update

```
openclaw plugins update <id-or-npm-spec>
openclaw plugins update --all
openclaw plugins update <id-or-npm-spec> --dry-run
openclaw plugins update @openclaw/voice-call@beta
openclaw plugins update openclaw-codex-app-server --dangerously-force-unsafe-install
```

Updates apply to tracked plugin installs in the managed plugin index and tracked hook-pack installs in `hooks.internal.installs`.

Resolving plugin id vs npm spec

When you pass a plugin id, OpenClaw reuses the recorded install spec for that plugin. That means previously stored dist-tags such as `@beta` and exact pinned versions continue to be used on later `update <id>` runs.For npm installs, you can also pass an explicit npm package spec with a dist-tag or exact version. OpenClaw resolves that package name back to the tracked plugin record, updates that installed plugin, and records the new npm spec for future id-based updates.Passing the npm package name without a version or tag also resolves back to the tracked plugin record. Use this when a plugin was pinned to an exact version and you want to move it back to the registry’s default release line.

Version checks and integrity drift

Before a live npm update, OpenClaw checks the installed package version against the npm registry metadata. If the installed version and recorded artifact identity already match the resolved target, the update is skipped without downloading, reinstalling, or rewriting `openclaw.json`.When a stored integrity hash exists and the fetched artifact hash changes, OpenClaw treats that as npm artifact drift. The interactive `openclaw plugins update` command prints the expected and actual hashes and asks for confirmation before proceeding. Non-interactive update helpers fail closed unless the caller supplies an explicit continuation policy.

--dangerously-force-unsafe-install on update

`--dangerously-force-unsafe-install` is also available on `plugins update` as a break-glass override for built-in dangerous-code scan false positives during plugin updates. It still does not bypass plugin `before_install` policy blocks or scan-failure blocking, and it only applies to plugin updates, not hook-pack updates.

### [​](https://docs.openclaw.ai/cli/plugins\\#inspect)  Inspect

```
openclaw plugins inspect <id>
openclaw plugins inspect <id> --json
```

Deep introspection for a single plugin. Shows identity, load status, source, registered capabilities, hooks, tools, commands, services, gateway methods, HTTP routes, policy flags, diagnostics, install metadata, bundle capabilities, and any detected MCP or LSP server support.Each plugin is classified by what it actually registers at runtime:

- **plain-capability** — one capability type (e.g. a provider-only plugin)
- **hybrid-capability** — multiple capability types (e.g. text + speech + images)
- **hook-only** — only hooks, no capabilities or surfaces
- **non-capability** — tools/commands/services but no capabilities

See [Plugin shapes](https://docs.openclaw.ai/plugins/architecture#plugin-shapes) for more on the capability model.

The `--json` flag outputs a machine-readable report suitable for scripting and auditing. `inspect --all` renders a fleet-wide table with shape, capability kinds, compatibility notices, bundle capabilities, and hook summary columns. `info` is an alias for `inspect`.

### [​](https://docs.openclaw.ai/cli/plugins\\#doctor)  Doctor

```
openclaw plugins doctor
```

`doctor` reports plugin load errors, manifest/discovery diagnostics, and compatibility notices. When everything is clean it prints `No plugin issues detected.`For module-shape failures such as missing `register`/`activate` exports, rerun with `OPENCLAW_PLUGIN_LOAD_DEBUG=1` to include a compact export-shap...(content truncated)

---

## Webhooks - OpenClaw
**Source:** https://docs.openclaw.ai/cli/webhooks

[Skip to main content](https://docs.openclaw.ai/cli/webhooks#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nConfiguration\n\nWebhooks\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [openclaw webhooks](https://docs.openclaw.ai/cli/webhooks#openclaw-webhooks)\n- [Gmail](https://docs.openclaw.ai/cli/webhooks#gmail)\n- [webhooks gmail setup](https://docs.openclaw.ai/cli/webhooks#webhooks-gmail-setup)\n- [webhooks gmail run](https://docs.openclaw.ai/cli/webhooks#webhooks-gmail-run)\n- [Related](https://docs.openclaw.ai/cli/webhooks#related)\n\n# [​](https://docs.openclaw.ai/cli/webhooks\\#openclaw-webhooks)  `openclaw webhooks`\n\nWebhook helpers and integrations (Gmail Pub/Sub, webhook helpers).Related:\n\n- Webhooks: [Webhooks](https://docs.openclaw.ai/automation/cron-jobs#webhooks)\n- Gmail Pub/Sub: [Gmail Pub/Sub](https://docs.openclaw.ai/automation/cron-jobs#gmail-pubsub-integration)\n\n## [​](https://docs.openclaw.ai/cli/webhooks\\#gmail)  Gmail\n\n```\nopenclaw webhooks gmail setup --account you@example.com\nopenclaw webhooks gmail run\n```\n\n### [​](https://docs.openclaw.ai/cli/webhooks\\#webhooks-gmail-setup)  `webhooks gmail setup`\n\nConfigure Gmail watch, Pub/Sub, and OpenClaw webhook delivery.Required:\n\n- `--account <email>`\n\nOptions:\n\n- `--project <id>`\n- `--topic <name>`\n- `--subscription <name>`\n- `--label <label>`\n- `--hook-url <url>`\n- `--hook-token <token>`\n- `--push-token <token>`\n- `--bind <host>`\n- `--port <port>`\n- `--path <path>`\n- `--include-body`\n- `--max-bytes <n>`\n- `--renew-minutes <n>`\n- `--tailscale <funnel|serve|off>`\n- `--tailscale-path <path>`\n- `--tailscale-target <target>`\n- `--push-endpoint <url>`\n- `--json`\n\nExamples:\n\n```\nopenclaw webhooks gmail setup --account you@example.com\nopenclaw webhooks gmail setup --account you@example.com --project my-gcp-project --json\nopenclaw webhooks gmail setup --account you@example.com --hook-url https://gateway.example.com/hooks/gmail\n```\n\n### [​](https://docs.openclaw.ai/cli/webhooks\\#webhooks-gmail-run)  `webhooks gmail run`\n\nRun `gog watch serve` plus the watch auto-renew loop.Options:\n\n- `--account <email>`\n- `--topic <topic>`\n- `--subscription <name>`\n- `--label <label>`\n- `--hook-url <url>`\n- `--hook-token <token>`\n- `--push-token <token>`\n- `--bind <host>`\n- `--port <port>`\n- `--path <path>`\n- `--include-body`\n- `--max-bytes <n>`\n- `--renew-minutes <n>`\n- `--tailscale <funnel|serve|off>`\n- `--tailscale-path <path>`\n- `--tailscale-target <target>`\n\nExample:\n\n```\nopenclaw webhooks gmail run --account you@example.com\n```\n\nSee [Gmail Pub/Sub documentation](https://docs.openclaw.ai/automation/cron-jobs#gmail-pubsub-integration) for the end-to-end setup flow and operational details.\n\n## [​](https://docs.openclaw.ai/cli/webhooks\\#related)  Related\n\n- [CLI reference](https://docs.openclaw.ai/cli)\n- [Webhook automation](https://docs.openclaw.ai/automation/webhook)\n\n[Configure](https://docs.openclaw.ai/cli/configure) [Plugins](https://docs.openclaw.ai/cli/plugins)\n\nCtrl+I

---

## ACP - OpenClaw
**Source:** https://docs.openclaw.ai/cli/acp

[Skip to main content](https://docs.openclaw.ai/cli/acp#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Utility

ACP

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [What this is not](https://docs.openclaw.ai/cli/acp#what-this-is-not)
- [Compatibility Matrix](https://docs.openclaw.ai/cli/acp#compatibility-matrix)
- [Known Limitations](https://docs.openclaw.ai/cli/acp#known-limitations)
- [Usage](https://docs.openclaw.ai/cli/acp#usage)
- [ACP client (debug)](https://docs.openclaw.ai/cli/acp#acp-client-debug)
- [How to use this](https://docs.openclaw.ai/cli/acp#how-to-use-this)
- [Selecting agents](https://docs.openclaw.ai/cli/acp#selecting-agents)
- [Use from acpx (Codex, Claude, other ACP clients)](https://docs.openclaw.ai/cli/acp#use-from-acpx-codex-claude-other-acp-clients)
- [Zed editor setup](https://docs.openclaw.ai/cli/acp#zed-editor-setup)
- [Session mapping](https://docs.openclaw.ai/cli/acp#session-mapping)
- [Options](https://docs.openclaw.ai/cli/acp#options)
- [acp client options](https://docs.openclaw.ai/cli/acp#acp-client-options)
- [Related](https://docs.openclaw.ai/cli/acp#related)

Run the [Agent Client Protocol (ACP)](https://agentclientprotocol.com/) bridge that talks to an OpenClaw Gateway.This command speaks ACP over stdio for IDEs and forwards prompts to the Gateway
over WebSocket. It keeps ACP sessions mapped to Gateway session keys.`openclaw acp` is a Gateway-backed ACP bridge, not a full ACP-native editor
runtime. It focuses on session routing, prompt delivery, and basic streaming
updates.If you want an external MCP client to talk directly to OpenClaw channel
conversations instead of hosting an ACP harness session, use
[`openclaw mcp serve`](https://docs.openclaw.ai/cli/mcp) instead.

## [​](https://docs.openclaw.ai/cli/acp\\#what-this-is-not)  What this is not

This page is often confused with ACP harness sessions.`openclaw acp` means:

- OpenClaw acts as an ACP server
- an IDE or ACP client connects to OpenClaw
- OpenClaw forwards that work into a Gateway session

This is different from [ACP Agents](https://docs.openclaw.ai/tools/acp-agents), where OpenClaw runs an
external harness such as Codex or Claude Code through `acpx`.Quick rule:

- editor/client wants to talk ACP to OpenClaw: use `openclaw acp`
- OpenClaw should launch Codex/Claude/Gemini as an ACP harness: use `/acp spawn` and [ACP Agents](https://docs.openclaw.ai/tools/acp-agents)

## [​](https://docs.openclaw.ai/cli/acp\\#compatibility-matrix)  Compatibility Matrix

| ACP area | Status | Notes |
| --- | --- | --- |
| `initialize`, `newSession`, `prompt`, `cancel` | Implemented | Core bridge flow over stdio to Gateway chat/send + abort. |
| `listSessions`, slash commands | Implemented | Session list works against Gateway session state; commands are advertised via `available_commands_update`. |
| `loadSession` | Partial | Rebinds the ACP session to a Gateway session key and replays stored user/assistant text history. Tool/system history is not reconstructed yet. |
| Prompt content (`text`, embedded `resource`, images) | Partial | Text/resources are flattened into chat input; images become Gateway attachments. |
| Session modes | Partial | `session/set_mode` is supported and the bridge exposes initial Gateway-backed session controls for thought level, tool verbosity, reasoning, usage detail, and elevated actions. Broader ACP-native mode/config surfaces are still out of scope. |
| Session info and usage updates | Partial | The bridge emits `session_info_update` and best-effort `usage_update` notifications from cached Gateway session snapshots. Usage is approximate and only sent when Gateway token totals are marked fresh. |
| Tool streaming | Partial | `tool_call` / `tool_call_update` events include raw I/O, text content, and best-effort file locations when Gateway tool args/results expose them. Embedded terminals and richer diff-native output are still not exposed. |
| Per-session MCP servers (`mcpServers`) | Unsupported | Bridge mode rejects per-session MCP server requests. Configure MCP on the OpenClaw gateway or agent instead. |
| Client filesystem methods (`fs/read_text_file`, `fs/write_text_file`) | Unsupported | The bridge does not call ACP client filesystem methods. |
| Client terminal methods (`terminal/*`) | Unsupported | The bridge does not create ACP client terminals or stream terminal ids through tool calls. |
| Session plans / thought streaming | Unsupported | The bridge currently emits output text and tool status, not ACP plan or thought updates. |

## [​](https://docs.openclaw.ai/cli/acp\\#known-limitations)  Known Limitations

- `loadSession` replays stored user and assistant text history, but it does not
reconstruct historic tool calls, system notices, or richer ACP-native event
types.
- If multiple ACP clients share the same Gateway session key, event and cancel
routing are best-effort rather than strictly isolated per client. Prefer the
default isolated `acp:<uuid>` sessions when you need clean editor-local
turns.
- Gateway stop states are translated into ACP stop reasons, but that mapping is
less expressive than a fully ACP-native runtime.
- Initial session controls currently surface a focused subset of Gateway knobs:
thought level, tool verbosity, reasoning, usage detail, and elevated
actions. Model selection and exec-host controls are not yet exposed as ACP
config options.
- `session_info_update` and `usage_update` are derived from Gateway session
snapshots, not live ACP-native runtime accounting. Usage is approximate,
carries no cost data, and is only emitted when the Gateway marks total token
data as fresh.
- Tool follow-along data is best-effort. The bridge can surface file paths that
appear in known tool args/results, but it does not yet emit ACP terminals or
structured file diffs.

## [​](https://docs.openclaw.ai/cli/acp\\#usage)  Usage

```
openclaw acp

# Remote Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# Remote Gateway (token from file)
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Attach to an existing session key
openclaw acp --session agent:main:main

# Attach by label (must already exist)
openclaw acp --session-label \"support inbox\"

# Reset the session key before the first prompt
openclaw acp --session agent:main:main --reset-session
```

## [​](https://docs.openclaw.ai/cli/acp\\#acp-client-debug)  ACP client (debug)

Use the built-in ACP client to sanity-check the bridge without an IDE.
It spawns the ACP bridge and lets you type prompts interactively.

```
openclaw acp client

# Point the spawned bridge at a remote Gateway
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Override the server command (default: openclaw)
openclaw acp client --server \"node\" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

Permission model (client debug mode):

- Auto-approval is allowlist-based and only applies to trusted core tool IDs.
- `read` auto-approval is scoped to the current working directory (`--cwd` when set).
- ACP only auto-approves narrow readonly classes: scoped `read` calls under the active cwd plus readonly search tools (`search`, `web_search`, `memory_search`). Unknown/non-core tools, out-of-scope reads, exec-capable tools, control-plane tools, mutating tools, and interactive flows always require explicit prompt approval.
- Server-provided `toolCall.kind` is treated as untrusted metadata (not an authorization source).
- This ACP bridge policy is separate from ACPX harness permissions. If you run OpenClaw through the `acpx` backend, `plugins.entries.acpx.config.permissionMode=approve-all` is the break-glass “yolo” switch for that harness session.

## [​](https://docs.openclaw.ai/cli/acp\\#how-to-use-this)  How to use this

Use ACP when an IDE (or other client) speaks Agent Client Protocol and you want
it to drive an OpenClaw Gateway session.

1. Ensure the Gateway is running (local or remote).
2. Configure the Gateway target (config or flags).
3. Point your IDE to run `openclaw acp` over stdio.

Example config (persisted):

```
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

Example direct run (no config write):

```
openclaw acp --url wss://gateway-host:18789 --token <token>
# preferred for local process safety
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```

## [​](https://docs.openclaw.ai/cli/acp\\#selecting-agents)  Selecting agents

ACP does not pick agents directly. It routes by the Gateway session key.Use agent-scoped session keys to target a specific agent:

```
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

Each ACP session maps to a single Gateway session key. One agent can have many
sessions; ACP defaults to an isolated `acp:<uuid>` session unless you override
the key or label.Per-session `mcpServers` are not supported in bridge mode. If an ACP client
sends them during `newSession` or `loadSession`, the bridge returns a clear
error instead of silently ignoring them.If you want ACPX-backed sessions to see OpenClaw plugin tools or selected
built-in tools such as `cron`, enable the gateway-side ACPX MCP bridges instead
of trying to pass per-session `mcpServers`. See
[ACP Agents](https://docs.openclaw.ai/tools/acp-agents-setup#plugin-tools-mcp-bridge) and
[OpenClaw tools MCP bridge](https://docs.openclaw.ai/tools/acp-agents-setup#openclaw-tools-mcp-bridge).

## [​](https://docs.openclaw.ai/cli/acp\\#use-from-acpx-codex-claude-other-acp-clients)  Use from `acpx` (Codex, Claude, other ACP clients)

If you want a coding agent such as Codex or Claude Code to talk to your
OpenClaw bot over ACP, use `acpx` with its built-in `openclaw` target.Typical flow:

1. Run the Gateway and make sure the ACP bridge can reach it.
2. Point `acpx openclaw` at `openclaw acp`.
3. Target the OpenClaw session key you want the coding agent to use.

Examples:

```
# One-shot request into your default OpenClaw ACP session
acpx openclaw exec \"Summarize the active OpenClaw session state.\"

# Persistent named session for follow-up turns
acpx openclaw sessions ensure --name codex-bridge
acpx openclaw -s codex-bridge --cwd /path/to/repo \\\
  \"Ask my OpenClaw work agent for recent context relevant to this repo.\"
```

If you want `acpx openclaw` to target a specific Gateway and session key every
time, override the `openclaw` agent command in `~/.acpx/config.json`:

```
{
  \"agents\": {
    \"openclaw\": {
      \"command\": \"env OPENCLAW_HIDE_BANNER=1 OPENCLAW_SUPPRESS_NOTES=1 openclaw acp --url ws://127.0.0.1:18789 --token-file ~/.openclaw/gateway.token --session agent:main:main\"
    }
  }
}
```

For a repo-local OpenClaw checkout, use the direct CLI entrypoint instead of the
dev runner so the ACP stream stays clean. For example:

```
env OPENCLAW_HIDE_BANNER=1 OPENCLAW_SUPPRESS_NOTES=1 node openclaw.mjs acp ...
```

This is the easiest way to let Codex, Claude Code, or another ACP-aware client
pull contextual information from an OpenClaw agent without scraping a terminal.

## [​](https://docs.openclaw.ai/cli/acp\\#zed-editor-setup)  Zed editor setup

Add a custom ACP agent in `~/.config/zed/settings.json` (or use Zed’s Settings UI):

```
{
  \"agent_servers\": {
    \"OpenClaw ACP\": {
      \"type\": \"custom\",
      \"command\": \"openclaw\",
      \"args\": [\"acp\"],
      \"env\": {}
    }
  }
}
```

To target a specific Gateway or agent:

```
{
  \"agent_servers\": {
    \"OpenClaw ACP\": {
      \"type\": \"custom\",
      \"command\": \"openclaw\",
      \"args\": [\\\
        \"acp\",\\\
        \"--url\",\\\
        \"wss://gateway-host:18789\",\\\
        \"--token\",\\\
        \"<token>\",\\\
        \"--session\",\\\
        \"agent:design:main\"\\\
      ],
      \"env\": {}
    }
  }
}
```

In Zed, open the Agent panel and select “OpenClaw ACP” to start a thread.

## [​](https://docs.openclaw.ai/cli/acp\\#session-mapping)  Session mapping

By default, ACP sessions get an isolated Gateway session key with an `acp:` prefix.
To reuse a known session, pass a session key or label:

- `--session <key>`: use a specific Gateway session key.
- `--session-label <label>`: resolve an existing session by label.
- `--reset-session`: mint a fresh session id for that key (same key, new transcript).

If your ACP client supports metadata, you can override per session:

```
{
  \"_meta\": {
    \"sessionKey\": \"agent:main:main\",
    \"sessionLabel\": \"support inbox\",
    \"resetSession\": true
  }
}
```

Learn more about session keys at [/concepts/session](https://docs.openclaw.ai/concepts/session).

## [​](https://docs.openclaw.ai/cli/acp\\#options)  Options

- `--url <url>`: Gateway WebSocket URL (defaults to gateway.remote.url when configured).
- `--token <token>`: Gateway auth token.
- `--token-file <path>`: read Gateway auth token from file.
- `--password <password>`: Gateway auth password.
- `--password-file <path>`: read Gateway auth password from file.
- `--session <key>`: default session key.
- `--session-label <label>`: default session label to resolve.
- `--require-existing`: fail if the session key/label does not exist.
- `--reset-session`: reset the session key before first use.
- `--no-prefix-cwd`: do not prefix prompts with the working directory.
- `--provenance <off|meta|meta+receipt>`: include ACP provenance metadata or receipts.
- `--verbose, -v`: verbose logging to stderr.

Security note:

- `--token` and `--password` can be visible in local process listings on some systems.
- Prefer `--token-file`/`--password-file` or environment variables (`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`).
- Gateway auth resolution follows the shared contract used by other Gateway clients:
  - local mode: env (`OPENCLAW_GATEWAY_*`) -\\> `gateway.auth.*` -\\> `gateway.remote.*` fallback only when `gateway.auth.*` is unset (configured-but-unresolved local SecretRefs fail closed)
  - remote mode: `gateway.remote.*` with env/config fallback per remote precedence rules
  - `--url` is override-safe and does not reuse implicit config/env credentials; pass explicit `--token`/`--password` (or file variants)
- ACP runtime backend child processes receive `OPENCLAW_SHELL=acp`, which can be used for context-specific shell/profile rules.
- `openclaw acp client` sets `OPENCLAW_SHELL=acp-client` on the spawned bridge process.

### [​](https://docs.openclaw.ai/cli/acp\\#acp-client-options)  `acp client` options

- `--cwd <dir>`: working directory for the ACP session.
- `--server <command>`: ACP server command (default: `openclaw`).
- `--server-args <args...>`: extra arguments passed to the ACP server.
- `--server-verbose`: enable verbose logging on the ACP server.
- `--verbose, -v`: verbose client logging.

## [​](https://docs.openclaw.ai/cli/acp\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)
- [ACP agents](https://docs.openclaw.ai/tools/acp-agents)

[TUI](https://docs.openclaw.ai/cli/tui) [Clawbot](https://docs.openclaw.ai/cli/clawbot)

Ctrl+I

---

## Config - OpenClaw
**Source:** https://docs.openclaw.ai/cli/config

[Skip to main content](https://docs.openclaw.ai/cli/config#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nConfiguration\n\nConfig\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Root options](https://docs.openclaw.ai/cli/config#root-options)\n- [Examples](https://docs.openclaw.ai/cli/config#examples)\n- [config schema](https://docs.openclaw.ai/cli/config#config-schema)\n- [Paths](https://docs.openclaw.ai/cli/config#paths)\n- [Values](https://docs.openclaw.ai/cli/config#values)\n- [config set modes](https://docs.openclaw.ai/cli/config#config-set-modes)\n- [Provider builder flags](https://docs.openclaw.ai/cli/config#provider-builder-flags)\n- [Dry run](https://docs.openclaw.ai/cli/config#dry-run)\n- [JSON output shape](https://docs.openclaw.ai/cli/config#json-output-shape)\n- [Write safety](https://docs.openclaw.ai/cli/config#write-safety)\n- [Subcommands](https://docs.openclaw.ai/cli/config#subcommands)\n- [Validate](https://docs.openclaw.ai/cli/config#validate)\n- [Related](https://docs.openclaw.ai/cli/config#related)\n\nConfig helpers for non-interactive edits in `openclaw.json`: get/set/unset/file/schema/validate values by path and print the active config file. Run without a subcommand to open the configure wizard (same as `openclaw configure`).\n\n## [​](https://docs.openclaw.ai/cli/config\\#root-options)  Root options\n\n[​](https://docs.openclaw.ai/cli/config#param-section-section)\n\n--section <section>\n\nstring\n\nRepeatable guided-setup section filter when you run `openclaw config` without a subcommand.\n\nSupported guided sections: `workspace`, `model`, `web`, `gateway`, `daemon`, `channels`, `plugins`, `skills`, `health`.\n\n## [​](https://docs.openclaw.ai/cli/config\\#examples)  Examples\n\n```\nopenclaw config file\nopenclaw config --section model\nopenclaw config --section gateway --section daemon\nopenclaw config schema\nopenclaw config get browser.executablePath\nopenclaw config set browser.executablePath \"/usr/bin/google-chrome\"\nopenclaw config set browser.profiles.work.executablePath \"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome\"\nopenclaw config set agents.defaults.heartbeat.every \"2h\"\nopenclaw config set agents.list[0].tools.exec.node \"node-id-or-name\"\nopenclaw config set agents.defaults.models \'{\"openai/gpt-5.4\":{}}\' --strict-json --merge\nopenclaw config set channels.discord.token --ref-provider default --ref-source env --ref-id DISCORD_BOT_TOKEN\nopenclaw config set secrets.providers.vaultfile --provider-source file --provider-path /etc/openclaw/secrets.json --provider-mode json\nopenclaw config unset plugins.entries.brave.config.webSearch.apiKey\nopenclaw config set channels.discord.token --ref-provider default --ref-source env --ref-id DISCORD_BOT_TOKEN --dry-run\nopenclaw config validate\nopenclaw config validate --json\n```\n\n### [​](https://docs.openclaw.ai/cli/config\\#config-schema)  `config schema`\n\nPrint the generated JSON schema for `openclaw.json` to stdout as JSON.\n\nWhat it includes\n\n- The current root config schema, plus a root `$schema` string field for editor tooling.\n- Field `title` and `description` docs metadata used by the Control UI.\n- Nested object, wildcard (`*`), and array-item (`[]`) nodes inherit the same `title` / `description` metadata when matching field documentation exists.\n- `anyOf` / `oneOf` / `allOf` branches inherit the same docs metadata too when matching field documentation exists.\n- Best-effort live plugin + channel schema metadata when runtime manifests can be loaded.\n- A clean fallback schema even when the current config is invalid.\n\nRelated runtime RPC\n\n`config.schema.lookup` returns one normalized config path with a shallow schema node (`title`, `description`, `type`, `enum`, `const`, common bounds), matched UI hint metadata, and immediate child summaries. Use it for path-scoped drill-down in Control UI or custom clients.\n\n```\nopenclaw config schema\n```\n\nPipe it into a file when you want to inspect or validate it with other tools:\n\n```\nopenclaw config schema > openclaw.schema.json\n```\n\n### [​](https://docs.openclaw.ai/cli/config\\#paths)  Paths\n\nPaths use dot or bracket notation:\n\n```\nopenclaw config get agents.defaults.workspace\nopenclaw config get agents.list[0].id\n```\n\nUse the agent list index to target a specific agent:\n\n```\nopenclaw config get agents.list\nopenclaw config set agents.list[1].tools.exec.node \"node-id-or-name\"\n```\n\n## [​](https://docs.openclaw.ai/cli/config\\#values)  Values\n\nValues are parsed as JSON5 when possible; otherwise they are treated as strings. Use `--strict-json` to require JSON5 parsing. `--json` remains supported as a legacy alias.\n\n```\nopenclaw config set agents.defaults.heartbeat.every \"0m\"\nopenclaw config set gateway.port 19001 --strict-json\nopenclaw config set channels.whatsapp.groups \'[\"*\"]\' --strict-json\n```\n\n`config get <path> --json` prints the raw value as JSON instead of terminal-formatted text.\n\nObject assignment replaces the target path by default. Protected map/list paths that commonly hold user-added entries, such as `agents.defaults.models`, `models.providers`, `models.providers.<id>.models`, `plugins.entries`, and `auth.profiles`, refuse replacements that would remove existing entries unless you pass `--replace`.\n\nUse `--merge` when adding entries to those maps:\n\n```\nopenclaw config set agents.defaults.models \'{\"openai/gpt-5.4\":{}}\' --strict-json --merge\nopenclaw config set models.providers.ollama.models \'[{\"id\":\"llama3.2\",\"name\":\"Llama 3.2\"}]\' --strict-json --merge\n```\n\nUse `--replace` only when you intentionally want the provided value to become the complete target value.\n\n## [​](https://docs.openclaw.ai/cli/config\\#config-set-modes)  `config set` modes\n\n`openclaw config set` supports four assignment styles:\n\n- Value mode\n\n- SecretRef builder mode\n\n- Provider builder mode\n\n- Batch mode\n\n\n```\nopenclaw config set <path> <value>\n```\n\n```\nopenclaw config set channels.discord.token \\\n  --ref-provider default \\\n  --ref-source env \\\n  --ref-id DISCORD_BOT_TOKEN\n```\n\nProvider builder mode targets `secrets.providers.<alias>` paths only:\n\n```\nopenclaw config set secrets.providers.vault \\\n  --provider-source exec \\\n  --provider-command /usr/local/bin/openclaw-vault \\\n  --provider-arg read \\\n  --provider-arg openai/api-key \\\n  --provider-timeout-ms 5000\n```\n\n```\nopenclaw config set --batch-json \'[\\\n  {\\\n    \"path\": \"secrets.providers.default\",\\\n    \"provider\": { \"source\": \"env\" }\\\n  },\\\n  {\\\n    \"path\": \"channels.discord.token\",\\\n    \"ref\": { \"source\": \"env\", \"provider\": \"default\", \"id\": \"DISCORD_BOT_TOKEN\" }\\\n  }\\\n]\'\n```\n\n```\nopenclaw config set --batch-file ./config-set.batch.json --dry-run\n```\n\nSecretRef assignments are rejected on unsupported runtime-mutable surfaces (for example `hooks.token`, `commands.ownerDisplaySecret`, Discord thread-binding webhook tokens, and WhatsApp creds JSON). See [SecretRef Credential Surface](https://docs.openclaw.ai/reference/secretref-credential-surface).\n\nBatch parsing always uses the batch payload (`--batch-json`/`--batch-file`) as the source of truth. `--strict-json` / `--json` do not change batch parsing behavior.JSON path/value mode remains supported for both SecretRefs and providers:\n\n```\nopenclaw config set channels.discord.token \\\n  \'{\"source\":\"env\",\"provider\":\"default\",\"id\":\"DISCORD_BOT_TOKEN\"}\' \\\n  --strict-json\n\nopenclaw config set secrets.providers.vaultfile \\\n  \'{\"source\":\"file\",\"path\":\"/etc/openclaw/secrets.json\",\"mode\":\"json\"}\' \\\n  --strict-json\n```\n\n## [​](https://docs.openclaw.ai/cli/config\\#provider-builder-flags)  Provider builder flags\n\nProvider builder targets must use `secrets.providers.<alias>` as the path.\n\nCommon flags\n\n- `--provider-source <env|file|exec>`\n- `--provider-timeout-ms <ms>` (`file`, `exec`)\n\nEnv provider (--provider-source env)\n\n- `--provider-allowlist <ENV_VAR>` (repeatable)\n\nFile provider (--provider-source file)\n\n- `--provider-path <path>` (required)\n- `--provider-mode <singleValue|json>`\n- `--provider-max-bytes <bytes>`\n- `--provider-allow-insecure-path`\n\nExec provider (--provider-source exec)\n\n- `--provider-command <path>` (required)\n- `--provider-arg <arg>` (repeatable)\n- `--provider-no-output-timeout-ms <ms>`\n- `--provider-max-output-bytes <bytes>`\n- `--provider-json-only`\n- `--provider-env <KEY=VALUE>` (repeatable)\n- `--provider-pass-env <ENV_VAR>` (repeatable)\n- `--provider-trusted-dir <path>` (repeatable)\n- `--provider-allow-insecure-path`\n- `--provider-allow-symlink-command`\n\nHardened exec provider example:\n\n```\nopenclaw config set secrets.providers.vault \\\n  --provider-source exec \\\n  --provider-command /usr/local/bin/openclaw-vault \\\n  --provider-arg read \\\n  --provider-arg openai/api-key \\\n  --provider-json-only \\\n  --provider-pass-env VAULT_TOKEN \\\n  --provider-trusted-dir /usr/local/bin \\\n  --provider-timeout-ms 5000\n```\n\n## [​](https://docs.openclaw.ai/cli/config\\#dry-run)  Dry run\n\nUse `--dry-run` to validate changes without writing `openclaw.json`.\n\n```\nopenclaw config set channels.discord.token \\\n  --ref-provider default \\\n  --ref-source env \\\n  --ref-id DISCORD_BOT_TOKEN \\\n  --dry-run\n\nopenclaw config set channels.discord.token \\\n  --ref-provider default \\\n  --ref-source env \\\n  --ref-id DISCORD_BOT_TOKEN \\\n  --dry-run \\\n  --json\n\nopenclaw config set channels.discord.token \\\n  --ref-provider vault \\\n  --ref-source exec \\\n  --ref-id discord/token \\\n  --dry-run \\\n  --allow-exec\n```\n\nDry-run behavior\n\n- Builder mode: runs SecretRef resolvability checks for changed refs/providers.\n- JSON mode (`--strict-json`, `--json`, or batch mode): runs schema validation plus SecretRef resolvability checks.\n- Policy validation also runs for known unsupported SecretRef target surfaces.\n- Policy checks evaluate the full post-change config, so parent-object writes (for example setting `hooks` as an object) cannot bypass unsupported-surface validation.\n- Exec SecretRef checks are skipped by default during dry-run to avoid command side effects.\n- Use `--allow-exec` with `--dry-run` to opt in to exec SecretRef checks (this may execute provider commands).\n- `--allow-exec` is dry-run only and errors if used without `--dry-run`.\n\n--dry-run --json fields\n\n`--dry-run --json` prints a machine-readable report:\n\n- `ok`: whether dry-run passed\n- `operations`: number of assignments evaluated\n- `checks`: whether schema/resolvability checks ran\n- `checks.resolvabilityComplete`: whether resolvability checks ran to completion (false when exec refs are skipped)\n- `refsChecked`: number of refs actually resolved during dry-run\n- `skippedExecRefs`: number of exec refs skipped because `--allow-exec` was not set\n- `errors`: structured schema/resolvability failures when `ok=false`\n\n### [​](https://docs.openclaw.ai/cli/config\\#json-output-shape)  JSON output shape\n\n```\n{\n  ok: boolean,\n  operations: number,\n  configPath: string,\n  inputModes: [\"value\" | \"json\" | \"builder\", ...],\n  checks: {\n    schema: boolean,\n    resolvability: boolean,\n    resolvabilityComplete: boolean,\n  },\n  refsChecked: number,\n  skippedExecRefs: number,\n  errors?: [\\\n    {\\\n      kind: \"schema\" | \"resolvability\",\\\n      message: string,\\\n      ref?: string, // present for resolvability errors\\\n    },\\\n  ],\n}\n```\n\n- Success example\n\n- Failure example\n\n\n```\n{\n  \"ok\": true,\n  \"operations\": 1,\n  \"configPath\": \"~/.openclaw/openclaw.json\",\n  \"inputModes\": [\"builder\"],\n  \"checks\": {\n    \"schema\": false,\n    \"resolvability\": true,\n    \"resolvabilityComplete\": true\n  },\n  \"refsChecked\": 1,\n  \"skippedExecRefs\": 0\n}\n```\n\n```\n{\n  \"ok\": false,\n  \"operations\": 1,\n  \"configPath\": \"~/.openclaw/openclaw.json\",\n  \"inputModes\": [\"builder\"],\n  \"checks\": {\n    \"schema\": false,\n    \"resolvability\": true,\n    \"resolvabilityComplete\": true\n  },\n  \"refsChecked\": 1,\n  \"skippedExecRefs\": 0,\n  \"errors\": [\\\n    {\\\n      \"kind\": \"resolvability\",\\\n      \"message\": \"Error: Environment variable \\\"MISSING_TEST_SECRET\\\" is not set.\",\\\n      \"ref\": \"env:default:MISSING_TEST_SECRET\"\\\n    }\\\n  ]\n}\n```\n\nIf dry-run fails\n\n- `config schema validation failed`: your post-change config shape is invalid; fix path/value or provider/ref object shape.\n- `Config policy validation failed: unsupported SecretRef usage`: move that credential back to plaintext/string input and keep SecretRefs on supported surfaces only.\n- `SecretRef assignment(s) could not be resolved`: referenced provider/ref currently cannot resolve (missing env var, invalid file pointer, exec provider failure, or provider/source mismatch).\n- `Dry run note: skipped <n> exec SecretRef resolvability check(s)`: dry-run skipped exec refs; rerun with `--allow-exec` if you need exec resolvability validation.\n- For batch mode, fix failing entries and rerun `--dry-run` before writing.\n\n## [​](https://docs.openclaw.ai/cli/config\\#write-safety)  Write safety\n\n`openclaw config set` and other OpenClaw-owned config writers validate the full post-change config before committing it to disk. If the new payload fails schema validation or looks like a destructive clobber, the active config is left alone and the rejected payload is saved beside it as `openclaw.json.rejected.*`.\n\nThe active config path must be a regular file. Symlinked `openclaw.json` layouts are unsupported for writes; use `OPENCLAW_CONFIG_PATH` to point directly at the real file instead.\n\nPrefer CLI writes for small edits:\n\n```\nopenclaw config set gateway.reload.mode hybrid --dry-run\nopenclaw config set gateway.reload.mode hybrid\nopenclaw config validate\n```\n\nIf a write is rejected, inspect the saved payload and fix the full config shape:\n\n```\nCONFIG=\"$(openclaw config file)\"\nls -lt \"$CONFIG\".rejected.* 2>/dev/null | head\nopenclaw config validate\n```\n\nDirect editor writes are still allowed, but the running Gateway treats them as untrusted until they validate. Invalid direct edits can be restored from the last-known-good backup during startup or hot reload. See [Gateway troubleshooting](https://docs.openclaw.ai/gateway/troubleshooting#gateway-restored-last-known-good-config).Whole-file recovery is reserved for globally broken config, such as parse errors, root-level schema failures, legacy migration failures, or mixed plugin and root failures. If validation fails only under `plugins.entries.<id>...`, OpenClaw keeps the active `openclaw.json` in place and reports the plugin-local issue instead of restoring `.last-good`. This prevents plugin schema changes or `minHostVersion` skew from rolling back unrelated user settings such as models, providers, auth profiles, channels, gateway exposure, tools, memory, browser, or cron config.\n\n## [​](https://docs.openclaw.ai/cli/config\\#subcommands)  Subcommands\n\n- `config file`: Print the active config file path (resolved from `OPENCLAW_CONFIG_PATH` or default location). The path should name a regular file, not a symlink.\n\nRestart the gateway after edits.\n\n## [​](https://docs.openclaw.ai/cli/config\\#validate)  Validate\n\nValidate the current config against the active schema without starting the gateway.\n\n```\nopenclaw config validate\nopenclaw config validate --json\n```\n\nAfter `openclaw config validate` is passing, you can use the local TUI to have an embedded agent compare the active config against the docs while you validate each change from the same terminal:\n\nIf validation is already failing, start with `openclaw configure` or `openclaw doctor --fix`. `openclaw chat` does not bypass the invalid-config guard.\n\n```\nopenclaw chat\n```\n\nThen inside the TUI:\n\n```\n!openclaw config file\n!openclaw docs gateway auth token secretref\n!openclaw config validate\n!openclaw doctor\n```\n\nTypical repair loop:\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/cli/config#)\n\nCompare with docs\n\nAsk the agent to compare your current config with the relevant docs page and suggest the smallest fix.\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/cli/config#)\n\nApply targeted edits\n\nApply targeted edits with `openclaw config set` or `openclaw configure`.\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/cli/config#)\n\nRe-validate\n\nRerun `openclaw config validate` after each change.\n\n4\n\n[Navigate to header](https://docs.openclaw.ai/cli/config#)\n\nDoctor for runtime issues\n\nIf validation passes but the runtime is still unhealthy, run `openclaw doctor` or `openclaw doctor --fix` for migration and repair help.\n\n## [​](https://docs.openclaw.ai/cli/config\\#related)  Related\n\n- [CLI reference](https://docs.openclaw.ai/cli)\n- [Configuration](https://docs.openclaw.ai/gateway/configuration)\n\n[Sandbox CLI](https://docs.openclaw.ai/cli/sandbox) [Configure](https://docs.openclaw.ai/cli/configure)\n\nCtrl+I

---

## Docs - OpenClaw
**Source:** https://docs.openclaw.ai/cli/docs

[Skip to main content](https://docs.openclaw.ai/cli/docs#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Utility

Docs

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [openclaw docs](https://docs.openclaw.ai/cli/docs#openclaw-docs)
- [Related](https://docs.openclaw.ai/cli/docs#related)

# [​](https://docs.openclaw.ai/cli/docs\\#openclaw-docs)  `openclaw docs`

Search the live docs index.Arguments:

- `[query...]`: search terms to send to the live docs index

Examples:

```
openclaw docs
openclaw docs browser existing-session
openclaw docs sandbox allowHostControl
openclaw docs gateway token secretref
```

Notes:

- With no query, `openclaw docs` opens the live docs search entrypoint.
- Multi-word queries are passed through as one search request.

## [​](https://docs.openclaw.ai/cli/docs\\#related)  Related

- [CLI reference](https://docs.openclaw.ai/cli)

[DNS](https://docs.openclaw.ai/cli/dns) [MCP](https://docs.openclaw.ai/cli/mcp)

Ctrl+I

---

