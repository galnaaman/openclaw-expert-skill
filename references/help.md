# OpenClaw Help Documentation

## FAQ: first-run setup
**Source:** https://docs.openclaw.ai/help/faq-first-run

[Skip to main content](https://docs.openclaw.ai/help/faq-first-run#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

FAQ

FAQ: first-run setup

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start and first-run setup](https://docs.openclaw.ai/help/faq-first-run#quick-start-and-first-run-setup)
- [Related](https://docs.openclaw.ai/help/faq-first-run#related)

Quick-start and first-run Q&A. For everyday operations, models, auth, sessions,
and troubleshooting see the main [FAQ](https://docs.openclaw.ai/help/faq).

## [​](https://docs.openclaw.ai/help/faq-first-run\\#quick-start-and-first-run-setup)  Quick start and first-run setup

I am stuck, fastest way to get unstuck

Use a local AI agent that can **see your machine**. That is far more effective than asking
in Discord, because most “I’m stuck” cases are **local config or environment issues** that
remote helpers cannot inspect.

- **Claude Code**: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
- **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

These tools can read the repo, run commands, inspect logs, and help fix your machine-level
setup (PATH, services, permissions, auth files). Give them the **full source checkout** via
the hackable (git) install:

```
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

This installs OpenClaw **from a git checkout**, so the agent can read the code + docs and
reason about the exact version you are running. You can always switch back to stable later
by re-running the installer without `--install-method git`.Tip: ask the agent to **plan and supervise** the fix (step-by-step), then execute only the
necessary commands. That keeps changes small and easier to audit.If you discover a real bug or fix, please file a GitHub issue or send a PR:
[https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues) [https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls)Start with these commands (share outputs when asking for help):

```
openclaw status
openclaw models status
openclaw doctor
```

What they do:

- `openclaw status`: quick snapshot of gateway/agent health + basic config.
- `openclaw models status`: checks provider auth + model availability.
- `openclaw doctor`: validates and repairs common config/state issues.

Other useful CLI checks: `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.Quick debug loop: [First 60 seconds if something is broken](https://docs.openclaw.ai/help/faq-first-run#first-60-seconds-if-something-is-broken).
Install docs: [Install](https://docs.openclaw.ai/install), [Installer flags](https://docs.openclaw.ai/install/installer), [Updating](https://docs.openclaw.ai/install/updating).

Heartbeat keeps skipping. What do the skip reasons mean?

Common heartbeat skip reasons:

- `quiet-hours`: outside the configured active-hours window
- `empty-heartbeat-file`: `HEARTBEAT.md` exists but only contains blank/header-only scaffolding
- `no-tasks-due`: `HEARTBEAT.md` task mode is active but none of the task intervals are due yet
- `alerts-disabled`: all heartbeat visibility is disabled (`showOk`, `showAlerts`, and `useIndicator` are all off)

In task mode, due timestamps are only advanced after a real heartbeat run
completes. Skipped runs do not mark tasks as completed.Docs: [Heartbeat](https://docs.openclaw.ai/gateway/heartbeat), [Automation & Tasks](https://docs.openclaw.ai/automation).

Recommended way to install and set up OpenClaw

The repo recommends running from source and using onboarding:

```
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

The wizard can also build UI assets automatically. After onboarding, you typically run the Gateway on port **18789**.From source (contributors/dev):

```
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build
openclaw onboard
```

If you don’t have a global install yet, run it via `pnpm openclaw onboard`.

How do I open the dashboard after onboarding?

The wizard opens your browser with a clean (non-tokenized) dashboard URL right after onboarding and also prints the link in the summary. Keep that tab open; if it didn’t launch, copy/paste the printed URL on the same machine.

How do I authenticate the dashboard on localhost vs remote?

**Localhost (same machine):**

- Open `http://127.0.0.1:18789/`.
- If it asks for shared-secret auth, paste the configured token or password into Control UI settings.
- Token source: `gateway.auth.token` (or `OPENCLAW_GATEWAY_TOKEN`).
- Password source: `gateway.auth.password` (or `OPENCLAW_GATEWAY_PASSWORD`).
- If no shared secret is configured yet, generate a token with `openclaw doctor --generate-gateway-token`.

**Not on localhost:**

- **Tailscale Serve** (recommended): keep bind loopback, run `openclaw gateway --tailscale serve`, open `https://<magicdns>/`. If `gateway.auth.allowTailscale` is `true`, identity headers satisfy Control UI/WebSocket auth (no pasted shared secret, assumes trusted gateway host); HTTP APIs still require shared-secret auth unless you deliberately use private-ingress `none` or trusted-proxy HTTP auth.
Bad concurrent Serve auth attempts from the same client are serialized before the failed-auth limiter records them, so the second bad retry can already show `retry later`.
- **Tailnet bind**: run `openclaw gateway --bind tailnet --token \"<token>\"` (or configure password auth), open `http://<tailscale-ip>:18789/`, then paste the matching shared secret in dashboard settings.
- **Identity-aware reverse proxy**: keep the Gateway behind a non-loopback trusted proxy, configure `gateway.auth.mode: \"trusted-proxy\"`, then open the proxy URL.
- **SSH tunnel**: `ssh -N -L 18789:127.0.0.1:18789 user@host` then open `http://127.0.0.1:18789/`. Shared-secret auth still applies over the tunnel; paste the configured token or password if prompted.

See [Dashboard](https://docs.openclaw.ai/web/dashboard) and [Web surfaces](https://docs.openclaw.ai/web) for bind modes and auth details.

Why are there two exec approval configs for chat approvals?

They control different layers:

- `approvals.exec`: forwards approval prompts to chat destinations
- `channels.<channel>.execApprovals`: makes that channel act as a native approval client for exec approvals

The host exec policy is still the real approval gate. Chat config only controls where approval
prompts appear and how people can answer them.In most setups you do **not** need both:

- If the chat already supports commands and replies, same-chat `/approve` works through the shared path.
- If a supported native channel can infer approvers safely, OpenClaw now auto-enables DM-first native approvals when `channels.<channel>.execApprovals.enabled` is unset or `\"auto\"`.
- When native approval cards/buttons are available, that native UI is the primary path; the agent should only include a manual `/approve` command if the tool result says chat approvals are unavailable or manual approval is the only path.
- Use `approvals.exec` only when prompts must also be forwarded to other chats or explicit ops rooms.
- Use `channels.<channel>.execApprovals.target: \"channel\"` or `\"both\"` only when you explicitly want approval prompts posted back into the originating room/topic.
- Plugin approvals are separate again: they use same-chat `/approve` by default, optional `approvals.plugin` forwarding, and only some native channels keep plugin-approval-native handling on top.

Short version: forwarding is for routing, native client config is for richer channel-specific UX.
See [Exec Approvals](https://docs.openclaw.ai/tools/exec-approvals).

What runtime do I need?

Node **>= 22** is required. `pnpm` is recommended. Bun is **not recommended** for the Gateway.

Does it run on Raspberry Pi?

Yes. The Gateway is lightweight - docs list **512MB-1GB RAM**, **1 core**, and about **500MB**
disk as enough for personal use, and note that a **Raspberry Pi 4 can run it**.If you want extra headroom (logs, media, other services), **2GB is recommended**, but it’s
not a hard minimum.Tip: a small Pi/VPS can host the Gateway, and you can pair **nodes** on your laptop/phone for
local screen/camera/canvas or command execution. See [Nodes](https://docs.openclaw.ai/nodes).

Any tips for Raspberry Pi installs?

Short version: it works, but expect rough edges.

- Use a **64-bit** OS and keep Node >= 22.
- Prefer the **hackable (git) install** so you can see logs and update fast.
- Start without channels/skills, then add them one by one.
- If you hit weird binary issues, it is usually an **ARM compatibility** problem.

Docs: [Linux](https://docs.openclaw.ai/platforms/linux), [Install](https://docs.openclaw.ai/install).

It is stuck on wake up my friend / onboarding will not hatch. What now?

That screen depends on the Gateway being reachable and authenticated. The TUI also sends
“Wake up, my friend!” automatically on first hatch. If you see that line with **no reply**
and tokens stay at 0, the agent never ran.

1. Restart the Gateway:

```
openclaw gateway restart
```

2. Check status + auth:

```
openclaw status
openclaw models status
openclaw logs --follow
```

3. If it still hangs, run:

```
openclaw doctor
```

If the Gateway is remote, ensure the tunnel/Tailscale connection is up and that the UI
is pointed at the right Gateway. See [Remote access](https://docs.openclaw.ai/gateway/remote).

Can I migrate my setup to a new machine (Mac mini) without redoing onboarding?

Yes. Copy the **state directory** and **workspace**, then run Doctor once. This
keeps your bot “exactly the same” (memory, session history, auth, and channel
state) as long as you copy **both** locations:

1. Install OpenClaw on the new machine.
2. Copy `$OPENCLAW_STATE_DIR` (default: `~/.openclaw`) from the old machine.
3. Copy your workspace (default: `~/.openclaw/workspace`).
4. Run `openclaw doctor` and restart the Gateway service.

That preserves config, auth profiles, WhatsApp creds, sessions, and memory. If you’re in
remote mode, remember the gateway host owns the session store and workspace.**Important:** if you only commit/push your workspace to GitHub, you’re backing
up **memory + bootstrap files**, but **not** session history or auth. Those live
under `~/.openclaw/` (for example `~/.openclaw/agents/<agentId>/sessions/`).Related: [Migrating](https://docs.openclaw.ai/install/migrating), [Where things live on disk](https://docs.openclaw.ai/help/faq-first-run#where-things-live-on-disk),
[Agent workspace](https://docs.openclaw.ai/concepts/agent-workspace), [Doctor](https://docs.openclaw.ai/gateway/doctor),
[Remote mode](https://docs.openclaw.ai/gateway/remote).

Where do I see what is new in the latest version?

Check the GitHub changelog:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)Newest entries are at the top. If the top section is marked **Unreleased**, the next dated
section is the latest shipped version. Entries are grouped by **Highlights**, **Changes**, and
**Fixes** (plus docs/other sections when needed).

Cannot access docs.openclaw.ai (SSL error)

Some Comcast/Xfinity connections incorrectly block `docs.openclaw.ai` via Xfinity
Advanced Security. Disable it or allowlist `docs.openclaw.ai`, then retry.
Please help us unblock it by reporting here: [https://spa.xfinity.com/check\\_url\\_status](https://spa.xfinity.com/check_url_status).If you still can’t reach the site, the docs are mirrored on GitHub:
[https://github.com/openclaw/openclaw/tree/main/docs](https://github.com/openclaw/openclaw/tree/main/docs)

Difference between stable and beta

**Stable** and **beta** are **npm dist-tags**, not separate code lines:

- `latest` = stable
- `beta` = early build for testing

Usually, a stable release lands on **beta** first, then an explicit
promotion step moves that same version to `latest`. Maintainers can also
publish straight to `latest` when needed. That’s why beta and stable can
point at the **same version** after promotion.See what changed:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)For install one-liners and the difference between beta and dev, see the accordion below.

How do I install the beta version and what is the difference between beta and dev?

**Beta** is the npm dist-tag `beta` (may match `latest` after promotion).
**Dev** is the moving head of `main` (git); when published, it uses the npm dist-tag `dev`.One-liners (macOS/Linux):

```
curl -fsSL --proto =https --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --beta
```

```
curl -fsSL --proto =https --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Windows installer (PowerShell):
[https://openclaw.ai/install.ps1](https://openclaw.ai/install.ps1)More detail: [Development channels](https://docs.openclaw.ai/install/development-channels) and [Installer flags](https://docs.openclaw.ai/install/installer).

How do I try the latest bits?

Two options:

1. **Dev channel (git checkout):**

```
openclaw update --channel dev
```

This switches to the `main` branch and updates from source.

2. **Hackable install (from the installer site):**

```
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

That gives you a local repo you can edit, then update via git.If you prefer a clean clone manually, use:

```
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

Docs: [Update](https://docs.openclaw.ai/cli/update), [Development channels](https://docs.openclaw.ai/install/development-channels),
[Install](https://docs.openclaw.ai/install).

How long does install and onboarding usually take?

Rough guide:

- **Install:** 2-5 minutes
- **Onboarding:** 5-15 minutes depending on how many channels/models you configure

If it hangs, use [Installer stuck](https://docs.openclaw.ai/help/faq-first-run#quick-start-and-first-run-setup)
and the fast debug loop in [I am stuck](https://docs.openclaw.ai/help/faq-first-run#quick-start-and-first-run-setup).

Installer stuck? How do I get more feedback?

Re-run the installer with **verbose output**:

```
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

Beta install with verbose:

```
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta --verbose
```

For a hackable (git) install:

```
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --verbose
```

Windows installer (PowerShell) equivalent:

```
# install.ps1 has no dedicated -Verbose flag yet.
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

More options: [Installer flags](https://docs.openclaw.ai/install/installer).

Windows install says git not found or openclaw not recognized

Two common Windows issues:**1) npm error spawn git / git not found**

- Install **Git for Windows** and make sure `git` is on your PATH.
- Close and reopen PowerShell, then re-run the installer.

**2) openclaw is not recognized after install**

- Your npm global bin folder is not on PATH.
- Check the path:














```
npm config get prefix
```

- Add that directory to your user PATH (no `\\bin` suffix needed on Windows; on most systems it is `%AppData%\\npm`).
- Close and reopen PowerShell after updating PATH.

If you want the smoothest Windows setup, use **WSL2** instead of native Windows.
Docs: [Windows](https://docs.openclaw.ai/platforms/windows).

Windows exec output shows garbled Chinese text - what should I do?

This is usually a console code page mismatch on native Windows shells.Symptoms:

- `system.run`/`exec` output renders Chinese as mojibake
- The same command looks fine in another terminal profile

Quick workaround in PowerShell:

```
chcp 65001
[Console]::InputEncoding = [System.Text.UTF8Encoding]::new($false)
[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new($false)
$OutputEncoding = [System.Text.UTF8Encoding]::new($false)
```

Then restart the Gateway and retry your command:

```
openclaw gateway restart
```

If you still reproduce this on latest OpenClaw, track/report it in:

- [Issue #30640](https://github.com/openclaw/openclaw/issues/30640)

The docs did not answer my question - how do I get a better answer?

Use the **hackable (git) install** so you have the full source and docs locally, then ask
your bot (or Claude/Codex) _from that folder_ so it can read the repo and answer precisely.

```
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

More detail: [Install](https://docs.openclaw.ai/install) and [Installer flags](https://docs.openclaw.ai/install/installer).

How do I install OpenClaw on Linux?

Short answer: follow the Linux guide, then run onboarding.

- Linux quick path + service install: [Linux](https://docs.openclaw.ai/platforms/linux).
- Full walkthrough: [Getting Started](https://docs.openclaw.ai/start/getting-started).
- Installer + updates: [Install & updates](https://docs.openclaw.ai/install/updating).

How do I install OpenClaw on a VPS?

Any Linux VPS works. Install on the server, then use SSH/Tailscale to reach the Gateway.Guides: [exe.dev](https://docs.openclaw.ai/install/exe-dev), [Hetzner](https://docs.openclaw.ai/install/hetzner), [Fly.io](https://docs.openclaw.ai/install/fly).
Remote access: [Gateway remote](https://docs.openclaw.ai/gateway/remote).

Where are the cloud/VPS install guides?

We keep a **hosting hub** with the common providers. Pick one and follow the guide:

- [VPS hosting](https://docs.openclaw.ai/vps) (all providers in one place)
- [Fly.io](https://docs.openclaw.ai/install/fly)
- [Hetzner](https://docs.openclaw.ai/install/hetzner)
- [exe.dev](https://docs.openclaw.ai/install/exe-dev)

How it works in the cloud: the **Gateway runs on the server**, and you access it
from your laptop/phone via the Control UI (or Tailscale/SSH). Your state + workspace
live on the server, so treat the host as the source of truth and back it up.You can pair **nodes** (Mac/iOS/Android/headless) to that cloud Gateway to access
local screen/camera/canvas or run commands on ...

---

## Scripts - OpenClaw
**Source:** https://docs.openclaw.ai/help/scripts

[Skip to main content](https://docs.openclaw.ai/help/scripts#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Release and CI

Scripts

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Conventions](https://docs.openclaw.ai/help/scripts#conventions)
- [Auth monitoring scripts](https://docs.openclaw.ai/help/scripts#auth-monitoring-scripts)
- [GitHub read helper](https://docs.openclaw.ai/help/scripts#github-read-helper)
- [When adding scripts](https://docs.openclaw.ai/help/scripts#when-adding-scripts)
- [Related](https://docs.openclaw.ai/help/scripts#related)

The `scripts/` directory contains helper scripts for local workflows and ops tasks.
Use these when a task is clearly tied to a script; otherwise prefer the CLI.

## [​](https://docs.openclaw.ai/help/scripts\\#conventions)  Conventions

- Scripts are **optional** unless referenced in docs or release checklists.
- Prefer CLI surfaces when they exist (example: auth monitoring uses `openclaw models status --check`).
- Assume scripts are host‑specific; read them before running on a new machine.

## [​](https://docs.openclaw.ai/help/scripts\\#auth-monitoring-scripts)  Auth monitoring scripts

Auth monitoring is covered in [Authentication](https://docs.openclaw.ai/gateway/authentication). The scripts under `scripts/` are optional extras for systemd/Termux phone workflows.

## [​](https://docs.openclaw.ai/help/scripts\\#github-read-helper)  GitHub read helper

Use `scripts/gh-read` when you want `gh` to use a GitHub App installation token for repo-scoped read calls while leaving normal `gh` on your personal login for write actions.Required env:

- `OPENCLAW_GH_READ_APP_ID`
- `OPENCLAW_GH_READ_PRIVATE_KEY_FILE`

Optional env:

- `OPENCLAW_GH_READ_INSTALLATION_ID` when you want to skip repo-based installation lookup
- `OPENCLAW_GH_READ_PERMISSIONS` as a comma-separated override for the read permission subset to request

Repo resolution order:

- `gh ... -R owner/repo`
- `GH_REPO`
- `git remote origin`

Examples:

- `scripts/gh-read pr view 123`
- `scripts/gh-read run list -R openclaw/openclaw`
- `scripts/gh-read api repos/openclaw/openclaw/pulls/123`

## [​](https://docs.openclaw.ai/help/scripts\\#when-adding-scripts)  When adding scripts

- Keep scripts focused and documented.
- Add a short entry in the relevant doc (or create one if missing).

## [​](https://docs.openclaw.ai/help/scripts\\#related)  Related

- [Testing](https://docs.openclaw.ai/help/testing)
- [Testing live](https://docs.openclaw.ai/help/testing-live)

[CI pipeline](https://docs.openclaw.ai/ci)

Ctrl+I

---

## Testing
**Source:** https://docs.openclaw.ai/help/testing

[Skip to main content](https://docs.openclaw.ai/help/testing#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nTesting\n\nTesting\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Quick start](https://docs.openclaw.ai/help/testing#quick-start)\n- [QA-specific runners](https://docs.openclaw.ai/help/testing#qa-specific-runners)\n- [Shared Telegram credentials via Convex (v1)](https://docs.openclaw.ai/help/testing#shared-telegram-credentials-via-convex-v1)\n- [Adding a channel to QA](https://docs.openclaw.ai/help/testing#adding-a-channel-to-qa)\n- [Test suites (what runs where)](https://docs.openclaw.ai/help/testing#test-suites-what-runs-where)\n- [Unit / integration (default)](https://docs.openclaw.ai/help/testing#unit-%2F-integration-default)\n- [Stability (gateway)](https://docs.openclaw.ai/help/testing#stability-gateway)\n- [E2E (gateway smoke)](https://docs.openclaw.ai/help/testing#e2e-gateway-smoke)\n- [E2E: OpenShell backend smoke](https://docs.openclaw.ai/help/testing#e2e-openshell-backend-smoke)\n- [Live (real providers + real models)](https://docs.openclaw.ai/help/testing#live-real-providers-%2B-real-models)\n- [Which suite should I run?](https://docs.openclaw.ai/help/testing#which-suite-should-i-run)\n- [Live (network-touching) tests](https://docs.openclaw.ai/help/testing#live-network-touching-tests)\n- [Docker runners (optional “works in Linux” checks)](https://docs.openclaw.ai/help/testing#docker-runners-optional-%E2%80%9Cworks-in-linux%E2%80%9D-checks)\n- [Docs sanity](https://docs.openclaw.ai/help/testing#docs-sanity)\n- [Offline regression (CI-safe)](https://docs.openclaw.ai/help/testing#offline-regression-ci-safe)\n- [Agent reliability evals (skills)](https://docs.openclaw.ai/help/testing#agent-reliability-evals-skills)\n- [Contract tests (plugin and channel shape)](https://docs.openclaw.ai/help/testing#contract-tests-plugin-and-channel-shape)\n- [Commands](https://docs.openclaw.ai/help/testing#commands)\n- [Channel contracts](https://docs.openclaw.ai/help/testing#channel-contracts)\n- [Provider status contracts](https://docs.openclaw.ai/help/testing#provider-status-contracts)\n- [Provider contracts](https://docs.openclaw.ai/help/testing#provider-contracts)\n- [When to run](https://docs.openclaw.ai/help/testing#when-to-run)\n- [Adding regressions (guidance)](https://docs.openclaw.ai/help/testing#adding-regressions-guidance)\n- [Related](https://docs.openclaw.ai/help/testing#related)\n\nOpenClaw has three Vitest suites (unit/integration, e2e, live) and a small set\nof Docker runners. This doc is a “how we test” guide:\n\n- What each suite covers (and what it deliberately does _not_ cover).\n- Which commands to run for common workflows (local, pre-push, debugging).\n- How live tests discover credentials and select models/providers.\n- How to add regressions for real-world model/provider issues.\n\n**QA stack (qa-lab, qa-channel, live transport lanes)** is documented separately:\n\n- [QA overview](https://docs.openclaw.ai/concepts/qa-e2e-automation) — architecture, command surface, scenario authoring.\n- [Matrix QA](https://docs.openclaw.ai/concepts/qa-matrix) — reference for `pnpm openclaw qa matrix`.\n- [QA channel](https://docs.openclaw.ai/channels/qa-channel) — the synthetic transport plugin used by repo-backed scenarios.\n\nThis page covers running the regular test suites and Docker/Parallels runners. The QA-specific runners section below ( [QA-specific runners](https://docs.openclaw.ai/help/testing#qa-specific-runners)) lists the concrete `qa` invocations and points back at the references above.\n\n## [​](https://docs.openclaw.ai/help/testing\\#quick-start)  Quick start\n\nMost days:\n\n- Full gate (expected before push): `pnpm build && pnpm check && pnpm check:test-types && pnpm test`\n- Faster local full-suite run on a roomy machine: `pnpm test:max`\n- Direct Vitest watch loop: `pnpm test:watch`\n- Direct file targeting now routes extension/channel paths too: `pnpm test extensions/discord/src/monitor/message-handler.preflight.test.ts`\n- Prefer targeted runs first when you are iterating on a single failure.\n- Docker-backed QA site: `pnpm qa:lab:up`\n- Linux VM-backed QA lane: `pnpm openclaw qa suite --runner multipass --scenario channel-chat-baseline`\n\nWhen you touch tests or want extra confidence:\n\n- Coverage gate: `pnpm test:coverage`\n- E2E suite: `pnpm test:e2e`\n\nWhen debugging real providers/models (requires real creds):\n\n- Live suite (models + gateway tool/image probes): `pnpm test:live`\n- Target one live file quietly: `pnpm test:live -- src/agents/models.profiles.live.test.ts`\n- Docker live model sweep: `pnpm test:docker:live-models`\n  - Each selected model now runs a text turn plus a small file-read-style probe.\n    Models whose metadata advertises `image` input also run a tiny image turn.\n    Disable the extra probes with `OPENCLAW_LIVE_MODEL_FILE_PROBE=0` or\n    `OPENCLAW_LIVE_MODEL_IMAGE_PROBE=0` when isolating provider failures.\n  - CI coverage: daily `OpenClaw Scheduled Live And E2E Checks` and manual\n    `OpenClaw Release Checks` both call the reusable live/E2E workflow with\n    `include_live_suites: true`, which includes separate Docker live model\n    matrix jobs sharded by provider.\n  - For focused CI reruns, dispatch `OpenClaw Live And E2E Checks (Reusable)`\n    with `include_live_suites: true` and `live_models_only: true`.\n  - Add new high-signal provider secrets to `scripts/ci-hydrate-live-auth.sh`\n    plus `.github/workflows/openclaw-live-and-e2e-checks-reusable.yml` and its\n    scheduled/release callers.\n- Native Codex bound-chat smoke: `pnpm test:docker:live-codex-bind`\n  - Runs a Docker live lane against the Codex app-server path, binds a synthetic\n    Slack DM with `/codex bind`, exercises `/codex fast` and\n    `/codex permissions`, then verifies a plain reply and an image attachment\n    route through the native plugin binding instead of ACP.\n- Codex app-server harness smoke: `pnpm test:docker:live-codex-harness`\n  - Runs gateway agent turns through the plugin-owned Codex app-server harness,\n    verifies `/codex status` and `/codex models`, and by default exercises image,\n    cron MCP, sub-agent, and Guardian probes. Disable the sub-agent probe with\n    `OPENCLAW_LIVE_CODEX_HARNESS_SUBAGENT_PROBE=0` when isolating other Codex\n    app-server failures. For a focused sub-agent check, disable the other probes:\n    `OPENCLAW_LIVE_CODEX_HARNESS_IMAGE_PROBE=0 OPENCLAW_LIVE_CODEX_HARNESS_MCP_PROBE=0 OPENCLAW_LIVE_CODEX_HARNESS_GUARDIAN_PROBE=0 OPENCLAW_LIVE_CODEX_HARNESS_SUBAGENT_PROBE=1 pnpm test:docker:live-codex-harness`.\n    This exits after the sub-agent probe unless\n    `OPENCLAW_LIVE_CODEX_HARNESS_SUBAGENT_ONLY=0` is set.\n- Crestodian rescue command smoke: `pnpm test:live:crestodian-rescue-channel`\n  - Opt-in belt-and-suspenders check for the message-channel rescue command\n    surface. It exercises `/crestodian status`, queues a persistent model\n    change, replies `/crestodian yes`, and verifies the audit/config write path.\n- Crestodian planner Docker smoke: `pnpm test:docker:crestodian-planner`\n  - Runs Crestodian in a configless container with a fake Claude CLI on `PATH`\n    and verifies the fuzzy planner fallback translates into an audited typed\n    config write.\n- Crestodian first-run Docker smoke: `pnpm test:docker:crestodian-first-run`\n  - Starts from an empty OpenClaw state dir, routes bare `openclaw` to\n    Crestodian, applies setup/model/agent/Discord plugin + SecretRef writes,\n    validates config, and verifies audit entries. The same Ring 0 setup path is\n    also covered in QA Lab by\n    `pnpm openclaw qa suite --scenario crestodian-ring-zero-setup`.\n- Moonshot/Kimi cost smoke: with `MOONSHOT_API_KEY` set, run\n`openclaw models list --provider moonshot --json`, then run an isolated\n`openclaw agent --local --session-id live-kimi-cost --message 'Reply exactly: KIMI_LIVE_OK' --thinking off --json`\nagainst `moonshot/kimi-k2.6`. Verify the JSON reports Moonshot/K2.6 and the\nassistant transcript stores normalized `usage.cost`.\n\nWhen you only need one failing case, prefer narrowing live tests via the allowlist env vars described below.\n\n## [​](https://docs.openclaw.ai/help/testing\\#qa-specific-runners)  QA-specific runners\n\nThese commands sit beside the main test suites when you need QA-lab realism:CI runs QA Lab in dedicated workflows. `Parity gate` runs on matching PRs and\nfrom manual dispatch with mock providers. `QA-Lab - All Lanes` runs nightly on\n`main` and from manual dispatch with the mock parity gate, live Matrix lane,\nConvex-managed live Telegram lane, and Convex-managed live Discord lane as\nparallel jobs. Scheduled QA and release checks pass Matrix `--profile fast`\nexplicitly, while the Matrix CLI and manual workflow input default remain\n`all`; manual dispatch can shard `all` into `transport`, `media`, `e2ee-smoke`,\n`e2ee-deep`, and `e2ee-cli` jobs. `OpenClaw Release Checks` runs parity plus\nthe fast Matrix and Telegram lanes before release approval.\n\n- `pnpm openclaw qa suite`\n  - Runs repo-backed QA scenarios directly on the host.\n  - Runs multiple selected scenarios in parallel by default with isolated\n    gateway workers. `qa-channel` defaults to concurrency 4 (bounded by the\n    selected scenario count). Use `--concurrency <count>` to tune the worker\n    count, or `--concurrency 1` for the older serial lane.\n  - Exits non-zero when any scenario fails. Use `--allow-failures` when you\n    want artifacts without a failing exit code.\n  - Supports provider modes `live-frontier`, `mock-openai`, and `aimock`.\n    `aimock` starts a local AIMock-backed provider server for experimental\n    fixture and protocol-mock coverage without replacing the scenario-aware\n    `mock-openai` lane.\n- `pnpm openclaw qa suite --runner multipass`\n  - Runs the same QA suite inside a disposable Multipass Linux VM.\n  - Keeps the same scenario-selection behavior as `qa suite` on the host.\n  - Reuses the same provider/model selection flags as `qa suite`.\n  - Live runs forward the supported QA auth inputs that are practical for the guest:\n    env-based provider keys, the QA live provider config path, and `CODEX_HOME`\n    when present.\n  - Output dirs must stay under the repo root so the guest can write back through\n    the mounted workspace.\n  - Writes the normal QA report + summary plus Multipass logs under\n    `.artifacts/qa-e2e/...`.\n- `pnpm qa:lab:up`\n  - Starts the Docker-backed QA site for operator-style QA work.\n- `pnpm test:docker:npm-onboard-channel-agent`\n  - Builds an npm tarball from the current checkout, installs it globally in\n    Docker, runs non-interactive OpenAI API-key onboarding, configures Telegram\n    by default, verifies enabling the plugin installs runtime dependencies on\n    demand, runs doctor, and runs one local agent turn against a mocked OpenAI\n    endpoint.\n  - Use `OPENCLAW_NPM_ONBOARD_CHANNEL=discord` to run the same packaged-install\n    lane with Discord.\n- `pnpm test:docker:session-runtime-context`\n  - Runs a deterministic built-app Docker smoke for embedded runtime context\n    transcripts. It verifies hidden OpenClaw runtime context is persisted as a\n    non-display custom message instead of leaking into the visible user turn,\n    then seeds an affected broken session JSONL and verifies\n    `openclaw doctor --fix` rewrites it to the active branch with a backup.\n- `pnpm test:docker:npm-telegram-live`\n  - Installs an OpenClaw package candidate in Docker, runs installed-package\n    onboarding, configures Telegram through the installed CLI, then reuses the\n    live Telegram QA lane with that installed package as the SUT Gateway.\n  - Defaults to `OPENCLAW_NPM_TELEGRAM_PACKAGE_SPEC=openclaw@beta`; set\n    `OPENCLAW_NPM_TELEGRAM_PACKAGE_TGZ=/path/to/openclaw-current.tgz` or\n    `OPENCLAW_CURRENT_PACKAGE_TGZ` to test a resolved local tarball instead of\n    installing from the registry.\n  - Uses the same Telegram env credentials or Convex credential source as\n    `pnpm openclaw qa telegram`. For CI/release automation, set\n    `OPENCLAW_QA_CREDENTIAL_SOURCE=convex` plus\n    `OPENCLAW_QA_CONVEX_SITE_URL` and the role secret. If\n    `OPENCLAW_QA_CONVEX_SITE_URL` and a Convex role secret are present in CI,\n    the Docker wrapper selects Convex automatically.\n  - `OPENCLAW_NPM_TELEGRAM_CREDENTIAL_ROLE=ci|maintainer` overrides the shared\n    `OPENCLAW_QA_CREDENTIAL_ROLE` for this lane only.\n  - GitHub Actions exposes this lane as the manual maintainer workflow\n    `NPM Telegram Beta E2E`. It does not run on merge. The workflow uses the\n    `qa-live-shared` environment and Convex CI credential leases.\n- GitHub Actions also exposes `Package Acceptance` for side-run product proof\nagainst one candidate package. It accepts a trusted ref, published npm spec,\nHTTPS tarball URL plus SHA-256, or tarball artifact from another run, uploads\nthe normalized `openclaw-current.tgz` as `package-under-test`, then runs the\nexisting Docker E2E scheduler with smoke, package, product, full, or custom\nlane profiles. Set `telegram_mode=mock-openai` or `live-frontier` to run the\nTelegram QA workflow against the same `package-under-test` artifact.\n\n  - Latest beta product proof:\n\n```\ngh workflow run package-acceptance.yml --ref main \\\n  -f source=npm \\\n  -f package_spec=openclaw@beta \\\n  -f suite_profile=product \\\n  -f telegram_mode=mock-openai\n```\n\n- Exact tarball URL proof requires a digest:\n\n```\ngh workflow run package-acceptance.yml --ref main \\\n  -f source=url \\\n  -f package_url=https://registry.npmjs.org/openclaw/-/openclaw-VERSION.tgz \\\n  -f package_sha256=<sha256> \\\n  -f suite_profile=package\n```\n\n- Artifact proof downloads a tarball artifact from another Actions run:\n\n```\ngh workflow run package-acceptance.yml --ref main \\\n  -f source=artifact \\\n  -f artifact_run_id=<run-id> \\\n  -f artifact_name=<artifact-name> \\\n  -f suite_profile=smoke\n```\n\n- `pnpm test:docker:bundled-channel-deps`  - Packs and installs the current OpenClaw build in Docker, starts the Gateway\n      with OpenAI configured, then enables bundled channel/plugins via config\n      edits.\n  - Verifies setup discovery leaves unconfigured plugin runtime dependencies\n    absent, the first configured Gateway or doctor run installs each bundled\n    plugin’s runtime dependencies on demand, and a second restart does not\n    reinstall dependencies that were already activated.\n  - Also installs a known older npm baseline, enables Telegram before running\n    `openclaw update --tag <candidate>`, and verifies the candidate’s\n    post-update doctor repairs bundled channel runtime dependencies without a\n    harness-side postinstall repair.\n- `pnpm test:parallels:npm-update`  - Runs the native packaged-install update smoke across Parallels guests. Each\n      selected platform first installs the requested baseline package, then runs\n      the installed `openclaw update` command in the same guest and verifies the\n      installed version, update status, gateway readiness, and one local agent\n      turn.\n  - Use `--platform macos`, `--platform windows`, or `--platform linux` while\n    iterating on one guest. Use `--json` for the summary artifact path and\n    per-lane status.\n  - The OpenAI lane uses `openai/gpt-5.5` for the live agent-turn proof by\n    default. Pass `--model <provider/model>` or set\n    `OPENCLAW_PARALLELS_OPENAI_MODEL` when deliberately validating another\n    OpenAI model.\n  - Wrap long local runs in a host timeout so Parallels transport stalls cannot\n    consume the rest of the testing window:\n\n\n\n\n\n\n\n\n\n\n\n\n\n    ```\n    timeout --foreground 150m pnpm test:parallels:npm-update -- --json\n    timeout --foreground 90m pnpm test:parallels:npm-update -- --platform windows --json\n    ```\n\n  - The script writes nested lane logs under `/tmp/openclaw-parallels-npm-update.*`.\n    Inspect `windows-update.log`, `macos-update.log`, or `linux-update.log`\n    before assuming the outer wrapper is hung.\n  - Windows update can spend 10 to 15 minutes in post-update doctor/runtime\n    dependency repair on a cold guest; that is still healthy when the nested\n    npm debug log is advancing.\n  - Do not run this aggregate wrapper in parallel with individual Parallels\n    macOS, Windows, or Linux smoke lanes. They share VM state and can collide on\n    snapshot restore, package serving, or guest gateway state.\n  - The post-update proof runs the normal bundled plugin surface because\n    capability facades such as speech, image generation, and media\n    understanding are loaded through bundled runtime APIs even when the agent\n    turn itself only checks a simple text response.\n- `pnpm openclaw qa aimock`  - Starts only the local AIMock provider server for direct protocol smoke\n      testing.\n- `pnpm openclaw qa matrix`  - Runs the Matrix live QA lane against a disposable Docker-backed Tuwunel homeserver. Source-checkout only — packaged installs do not ship `qa-lab`.\n  - Full CLI, profile/scenario catalog, env vars, and artifact layout: [Matrix QA](https://docs.openclaw.ai/concepts/qa-matrix).\n- `pnpm openclaw qa telegram`  - Runs the Telegram live QA lane against a real private group using the driver and SUT bot tokens from env.\n  - Requires `OPENCLAW_QA_TELEGRAM_GROUP_ID`, `OPENCLAW_QA_TELEGRAM_DRIVER_BOT_TOKEN`, and `OPENCLAW_QA_TELEGRAM_SUT_BOT_TOKEN`. The group id must be the numeric Telegram chat id.\n  - Supports `--credential-source convex` for shared pooled credentials. Use env mode by default, or set `OPENCLAW_QA_CREDENTIAL_SOURCE=convex` to opt into pooled leases.\n  - Exits non-zero when any scenario fails. Use `--allow-failures` when you\n    want artifacts without a failing exit code.\n  - Requires two distinct bots in the same private group, with the SUT bot exposing a Telegram username.\n  - For stable bot-to-bot observation, enable Bot-to-Bot Communication Mode in `@BotFather` for both bots and ensure the driver bot can observe group bot traffic.\n  - Writes a Telegram QA report, summary, and observed-messages artifact under `.artifacts/qa-e2e/...`. Replying scenarios include RTT from driver send request to observed SUT reply.\n\nLive transport lanes share one standard contract so new transports do not drift; the per-lane coverage matrix lives in [QA overview → Live transport coverage](https://docs.openclaw.ai/concepts/qa-e2e-automation#live-transport-coverage). `qa-channel` is the broad synthetic suite and is not part of that matrix.\n\n### [​](https://docs.openclaw.ai/help/testing\\#shared-telegram-credentials-via-convex-v1)  Shared Telegram credentials via Conve...(content truncated)

---

## Testing: live suites
**Source:** https://docs.openclaw.ai/help/testing-live

[Skip to main content](https://docs.openclaw.ai/help/testing-live#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Testing

Testing: live suites

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Live: local profile smoke commands](https://docs.openclaw.ai/help/testing-live#live-local-profile-smoke-commands)
- [Live: Android node capability sweep](https://docs.openclaw.ai/help/testing-live#live-android-node-capability-sweep)
- [Live: model smoke (profile keys)](https://docs.openclaw.ai/help/testing-live#live-model-smoke-profile-keys)
- [Layer 1: Direct model completion (no gateway)](https://docs.openclaw.ai/help/testing-live#layer-1-direct-model-completion-no-gateway)
- [Layer 2: Gateway + dev agent smoke (what “@openclaw” actually does)](https://docs.openclaw.ai/help/testing-live#layer-2-gateway-%2B-dev-agent-smoke-what-%E2%80%9C%40openclaw%E2%80%9D-actually-does)
- [Live: CLI backend smoke (Claude, Codex, Gemini, or other local CLIs)](https://docs.openclaw.ai/help/testing-live#live-cli-backend-smoke-claude-codex-gemini-or-other-local-clis)
- [Live: ACP bind smoke (/acp spawn ... --bind here)](https://docs.openclaw.ai/help/testing-live#live-acp-bind-smoke-%2Facp-spawn-bind-here)
- [Live: Codex app-server harness smoke](https://docs.openclaw.ai/help/testing-live#live-codex-app-server-harness-smoke)
- [Recommended live recipes](https://docs.openclaw.ai/help/testing-live#recommended-live-recipes)
- [Live: model matrix (what we cover)](https://docs.openclaw.ai/help/testing-live#live-model-matrix-what-we-cover)
- [Modern smoke set (tool calling + image)](https://docs.openclaw.ai/help/testing-live#modern-smoke-set-tool-calling-%2B-image)
- [Baseline: tool calling (Read + optional Exec)](https://docs.openclaw.ai/help/testing-live#baseline-tool-calling-read-%2B-optional-exec)
- [Vision: image send (attachment → multimodal message)](https://docs.openclaw.ai/help/testing-live#vision-image-send-attachment-%E2%86%92-multimodal-message)
- [Aggregators / alternate gateways](https://docs.openclaw.ai/help/testing-live#aggregators-%2F-alternate-gateways)
- [Credentials (never commit)](https://docs.openclaw.ai/help/testing-live#credentials-never-commit)
- [Deepgram live (audio transcription)](https://docs.openclaw.ai/help/testing-live#deepgram-live-audio-transcription)
- [BytePlus coding plan live](https://docs.openclaw.ai/help/testing-live#byteplus-coding-plan-live)
- [ComfyUI workflow media live](https://docs.openclaw.ai/help/testing-live#comfyui-workflow-media-live)
- [Image generation live](https://docs.openclaw.ai/help/testing-live#image-generation-live)
- [Music generation live](https://docs.openclaw.ai/help/testing-live#music-generation-live)
- [Video generation live](https://docs.openclaw.ai/help/testing-live#video-generation-live)
- [Media live harness](https://docs.openclaw.ai/help/testing-live#media-live-harness)
- [Related](https://docs.openclaw.ai/help/testing-live#related)

For quick start, QA runners, unit/integration suites, and Docker flows, see
[Testing](https://docs.openclaw.ai/help/testing). This page covers the **live** (network-touching) test
suites: model matrix, CLI backends, ACP, and media-provider live tests, plus
credential handling.

## [​](https://docs.openclaw.ai/help/testing-live\\#live-local-profile-smoke-commands)  Live: local profile smoke commands

Source `~/.profile` before ad hoc live checks so provider keys and local tool
paths match your shell:

```
source ~/.profile
```

Safe media smoke:

```
pnpm openclaw infer tts convert --local --json \\
  --text \"OpenClaw live smoke.\" \\
  --output /tmp/openclaw-live-smoke.mp3
```

Safe voice-call readiness smoke:

```
pnpm openclaw voicecall setup --json
pnpm openclaw voicecall smoke --to \"+15555550123\"
```

`voicecall smoke` is a dry run unless `--yes` is also present. Use `--yes` only
when you intentionally want to place a real notify call. For Twilio, Telnyx, and
Plivo, a successful readiness check requires a public webhook URL; local-only
loopback/private fallbacks are rejected by design.

## [​](https://docs.openclaw.ai/help/testing-live\\#live-android-node-capability-sweep)  Live: Android node capability sweep

- Test: `src/gateway/android-node.capabilities.live.test.ts`
- Script: `pnpm android:test:integration`
- Goal: invoke **every command currently advertised** by a connected Android node and assert command contract behavior.
- Scope:
  - Preconditioned/manual setup (the suite does not install/run/pair the app).
  - Command-by-command gateway `node.invoke` validation for the selected Android node.
- Required pre-setup:
  - Android app already connected + paired to the gateway.
  - App kept in foreground.
  - Permissions/capture consent granted for capabilities you expect to pass.
- Optional target overrides:
  - `OPENCLAW_ANDROID_NODE_ID` or `OPENCLAW_ANDROID_NODE_NAME`.
  - `OPENCLAW_ANDROID_GATEWAY_URL` / `OPENCLAW_ANDROID_GATEWAY_TOKEN` / `OPENCLAW_ANDROID_GATEWAY_PASSWORD`.
- Full Android setup details: [Android App](https://docs.openclaw.ai/platforms/android)

## [​](https://docs.openclaw.ai/help/testing-live\\#live-model-smoke-profile-keys)  Live: model smoke (profile keys)

Live tests are split into two layers so we can isolate failures:

- “Direct model” tells us the provider/model can answer at all with the given key.
- “Gateway smoke” tells us the full gateway+agent pipeline works for that model (sessions, history, tools, sandbox policy, etc.).

### [​](https://docs.openclaw.ai/help/testing-live\\#layer-1-direct-model-completion-no-gateway)  Layer 1: Direct model completion (no gateway)

- Test: `src/agents/models.profiles.live.test.ts`
- Goal:
  - Enumerate discovered models
  - Use `getApiKeyForModel` to select models you have creds for
  - Run a small completion per model (and targeted regressions where needed)
- How to enable:
  - `pnpm test:live` (or `OPENCLAW_LIVE_TEST=1` if invoking Vitest directly)
- Set `OPENCLAW_LIVE_MODELS=modern` (or `all`, alias for modern) to actually run this suite; otherwise it skips to keep `pnpm test:live` focused on gateway smoke
- How to select models:
  - `OPENCLAW_LIVE_MODELS=modern` to run the modern allowlist (Opus/Sonnet 4.6+, GPT-5.2 + Codex, Gemini 3, DeepSeek V4, GLM 4.7, MiniMax M2.7, Grok 4)
  - `OPENCLAW_LIVE_MODELS=all` is an alias for the modern allowlist
  - or `OPENCLAW_LIVE_MODELS=\"openai/gpt-5.2,openai-codex/gpt-5.2,anthropic/claude-opus-4-6,...\"` (comma allowlist)
  - Modern/all sweeps default to a curated high-signal cap; set `OPENCLAW_LIVE_MAX_MODELS=0` for an exhaustive modern sweep or a positive number for a smaller cap.
  - Exhaustive sweeps use `OPENCLAW_LIVE_TEST_TIMEOUT_MS` for the whole direct-model test timeout. Default: 60 minutes.
  - Direct-model probes run with 20-way parallelism by default; set `OPENCLAW_LIVE_MODEL_CONCURRENCY` to override.
- How to select providers:
  - `OPENCLAW_LIVE_PROVIDERS=\"google,google-antigravity,google-gemini-cli\"` (comma allowlist)
- Where keys come from:
  - By default: profile store and env fallbacks
  - Set `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` to enforce **profile store** only
- Why this exists:
  - Separates “provider API is broken / key is invalid” from “gateway agent pipeline is broken”
  - Contains small, isolated regressions (example: OpenAI Responses/Codex Responses reasoning replay + tool-call flows)

### [​](https://docs.openclaw.ai/help/testing-live\\#layer-2-gateway-+-dev-agent-smoke-what-%E2%80%9C@openclaw%E2%80%9D-actually-does)  Layer 2: Gateway + dev agent smoke (what “@openclaw” actually does)

- Test: `src/gateway/gateway-models.profiles.live.test.ts`
- Goal:
  - Spin up an in-process gateway
  - Create/patch a `agent:dev:*` session (model override per run)
  - Iterate models-with-keys and assert:
    - “meaningful” response (no tools)
    - a real tool invocation works (read probe)
    - optional extra tool probes (exec+read probe)
    - OpenAI regression paths (tool-call-only → follow-up) keep working
- Probe details (so you can explain failures quickly):
  - `read` probe: the test writes a nonce file in the workspace and asks the agent to `read` it and echo the nonce back.
  - `exec+read` probe: the test asks the agent to `exec`-write a nonce into a temp file, then `read` it back.
  - image probe: the test attaches a generated PNG (cat + randomized code) and expects the model to return `cat <CODE>`.
  - Implementation reference: `src/gateway/gateway-models.profiles.live.test.ts` and `src/gateway/live-image-probe.ts`.
- How to enable:
  - `pnpm test:live` (or `OPENCLAW_LIVE_TEST=1` if invoking Vitest directly)
- How to select models:
  - Default: modern allowlist (Opus/Sonnet 4.6+, GPT-5.2 + Codex, Gemini 3, DeepSeek V4, GLM 4.7, MiniMax M2.7, Grok 4)
  - `OPENCLAW_LIVE_GATEWAY_MODELS=all` is an alias for the modern allowlist
  - Or set `OPENCLAW_LIVE_GATEWAY_MODELS=\"provider/model\"` (or comma list) to narrow
  - Modern/all gateway sweeps default to a curated high-signal cap; set `OPENCLAW_LIVE_GATEWAY_MAX_MODELS=0` for an exhaustive modern sweep or a positive number for a smaller cap.
- How to select providers (avoid “OpenRouter everything”):
  - `OPENCLAW_LIVE_GATEWAY_PROVIDERS=\"google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax\"` (comma allowlist)
- Tool + image probes are always on in this live test:
  - `read` probe + `exec+read` probe (tool stress)
  - image probe runs when the model advertises image input support
  - Flow (high level):
    - Test generates a tiny PNG with “CAT” + random code (`src/gateway/live-image-probe.ts`)
    - Sends it via `agent``attachments: [{ mimeType: \"image/png\", content: \"<base64>\" }]`
    - Gateway parses attachments into `images[]` (`src/gateway/server-methods/agent.ts` \\+ `src/gateway/chat-attachments.ts`)
    - Embedded agent forwards a multimodal user message to the model
    - Assertion: reply contains `cat` \\+ the code (OCR tolerance: minor mistakes allowed)

To see what you can test on your machine (and the exact `provider/model` ids), run:

```
openclaw models list
openclaw models list --json
```

## [​](https://docs.openclaw.ai/help/testing-live\\#live-cli-backend-smoke-claude-codex-gemini-or-other-local-clis)  Live: CLI backend smoke (Claude, Codex, Gemini, or other local CLIs)

- Test: `src/gateway/gateway-cli-backend.live.test.ts`
- Goal: validate the Gateway + agent pipeline using a local CLI backend, without touching your default config.
- Backend-specific smoke defaults live with the owning extension’s `cli-backend.ts` definition.
- Enable:
  - `pnpm test:live` (or `OPENCLAW_LIVE_TEST=1` if invoking Vitest directly)
  - `OPENCLAW_LIVE_CLI_BACKEND=1`
- Defaults:
  - Default provider/model: `claude-cli/claude-sonnet-4-6`
  - Command/args/image behavior come from the owning CLI backend plugin metadata.
- Overrides (optional):
  - `OPENCLAW_LIVE_CLI_BACKEND_MODEL=\"codex-cli/gpt-5.2\"`
  - `OPENCLAW_LIVE_CLI_BACKEND_COMMAND=\"/full/path/to/codex\"`
  - `OPENCLAW_LIVE_CLI_BACKEND_ARGS=\'[\"exec\",\"--json\",\"--color\",\"never\",\"--sandbox\",\"read-only\",\"--skip-git-repo-check\"]\'`
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` to send a real image attachment (paths are injected into the prompt). Docker recipes default this off unless explicitly requested.
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG=\"--image\"` to pass image file paths as CLI args instead of prompt injection.
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE=\"repeat\"` (or `\"list\"`) to control how image args are passed when `IMAGE_ARG` is set.
  - `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` to send a second turn and validate resume flow.
  - `OPENCLAW_LIVE_CLI_BACKEND_MODEL_SWITCH_PROBE=1` to opt into the Claude Sonnet -> Opus same-session continuity probe when the selected model supports a switch target. Docker recipes default this off for aggregate reliability.
  - `OPENCLAW_LIVE_CLI_BACKEND_MCP_PROBE=1` to opt into the MCP/tool loopback probe. Docker recipes default this off unless explicitly requested.

Example:

```
OPENCLAW_LIVE_CLI_BACKEND=1 \\
  OPENCLAW_LIVE_CLI_BACKEND_MODEL=\"codex-cli/gpt-5.2\" \\
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

Cheap Gemini MCP config smoke:

```
OPENCLAW_LIVE_TEST=1 \\
  pnpm test:live src/agents/cli-runner/bundle-mcp.gemini.live.test.ts
```

This does not ask Gemini to generate a response. It writes the same system
settings OpenClaw gives Gemini, then runs `gemini --debug mcp list` to prove a
saved `transport: \"streamable-http\"` server is normalized to Gemini’s HTTP MCP
shape and can connect to a local streamable-HTTP MCP server.Docker recipe:

```
pnpm test:docker:live-cli-backend
```

Single-provider Docker recipes:

```
pnpm test:docker:live-cli-backend:claude
pnpm test:docker:live-cli-backend:claude-subscription
pnpm test:docker:live-cli-backend:codex
pnpm test:docker:live-cli-backend:gemini
```

Notes:

- The Docker runner lives at `scripts/test-live-cli-backend-docker.sh`.
- It runs the live CLI-backend smoke inside the repo Docker image as the non-root `node` user.
- It resolves CLI smoke metadata from the owning extension, then installs the matching Linux CLI package (`@anthropic-ai/claude-code`, `@openai/codex`, or `@google/gemini-cli`) into a cached writable prefix at `OPENCLAW_DOCKER_CLI_TOOLS_DIR` (default: `~/.cache/openclaw/docker-cli-tools`).
- `pnpm test:docker:live-cli-backend:claude-subscription` requires portable Claude Code subscription OAuth through either `~/.claude/.credentials.json` with `claudeAiOauth.subscriptionType` or `CLAUDE_CODE_OAUTH_TOKEN` from `claude setup-token`. It first proves direct `claude -p` in Docker, then runs two Gateway CLI-backend turns without preserving Anthropic API-key env vars. This subscription lane disables the Claude MCP/tool and image probes by default because Claude currently routes third-party app usage through extra-usage billing instead of normal subscription plan limits.
- The live CLI-backend smoke now exercises the same end-to-end flow for Claude, Codex, and Gemini: text turn, image classification turn, then MCP `cron` tool call verified through the gateway CLI.
- Claude’s default smoke also patches the session from Sonnet to Opus and verifies the resumed session still remembers an earlier note.

## [​](https://docs.openclaw.ai/help/testing-live\\#live-acp-bind-smoke-/acp-spawn-bind-here)  Live: ACP bind smoke (`/acp spawn ... --bind here`)

- Test: `src/gateway/gateway-acp-bind.live.test.ts`
- Goal: validate the real ACP conversation-bind flow with a live ACP agent:
  - send `/acp spawn <agent> --bind here`
  - bind a synthetic message-channel conversation in place
  - send a normal follow-up on that same conversation
  - verify the follow-up lands in the bound ACP session transcript
- Enable:
  - `pnpm test:live src/gateway/gateway-acp-bind.live.test.ts`
  - `OPENCLAW_LIVE_ACP_BIND=1`
- Defaults:
  - ACP agents in Docker: `claude,codex,gemini`
  - ACP agent for direct `pnpm test:live ...`: `claude`
  - Synthetic channel: Slack DM-style conversation context
  - ACP backend: `acpx`
- Overrides:
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=claude`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=codex`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=droid`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=gemini`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=opencode`
  - `OPENCLAW_LIVE_ACP_BIND_AGENTS=claude,codex,gemini`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT_COMMAND=\'npx -y @agentclientprotocol/claude-agent-acp@<version>\'`
  - `OPENCLAW_LIVE_ACP_BIND_CODEX_MODEL=gpt-5.2`
  - `OPENCLAW_LIVE_ACP_BIND_OPENCODE_MODEL=opencode/kimi-k2.6`
  - `OPENCLAW_LIVE_ACP_BIND_REQUIRE_TRANSCRIPT=1`
  - `OPENCLAW_LIVE_ACP_BIND_REQUIRE_CRON=1`
  - `OPENCLAW_LIVE_ACP_BIND_PARENT_MODEL=openai/gpt-5.2`
- Notes:
  - This lane uses the gateway `chat.send` surface with admin-only synthetic originating-route fields so tests can attach message-channel context without pretending to deliver externally.
  - When `OPENCLAW_LIVE_ACP_BIND_AGENT_COMMAND` is unset, the test uses the embedded `acpx` plugin’s built-in agent registry for the selected ACP harness agent.
  - Bound-session cron MCP creation is best-effort by default because external ACP harnesses can cancel MCP calls after the bind/image proof has passed; set `OPENCLAW_LIVE_ACP_BIND_REQUIRE_CRON=1` to make that post-bind cron probe strict.

Example:

```
OPENCLAW_LIVE_ACP_BIND=1 \\
  OPENCLAW_LIVE_ACP_BIND_AGENT=claude \\
  pnpm test:live src/gateway/gateway-acp-bind.live.test.ts
```

Docker recipe:

```
pnpm test:docker:live-acp-bind
```

Single-agent Docker recipes:

```
pnpm test:docker:live-acp-bind:claude
pnpm test:docker:live-acp-bind:codex
pnpm test:docker:live-acp-bind:droid
pnpm test:docker:live-acp-bind:gemini
pnpm test:docker:live-acp-bind:opencode
```

Docker notes:

- The Docker runner lives at `scripts/test-live-acp-bind-docker.sh`.
- By default, it runs the ACP bind smoke against the aggregate live CLI agents in sequence: `claude`, `codex`, then `gemini`.
- Use `OPENCLAW_LIVE_ACP_BIND_AGENTS=claude`, `OPENCLAW_LIVE_ACP_BIND_AGENTS=codex`, `OPENCLAW_LIVE_ACP_BIND_AGENTS=droid`, `OPENCLAW_LIVE_ACP_BIND_AGENTS=gemini`, or `OPENCLAW_LIVE_ACP_BIND_AGENTS=opencode` to narrow the matrix.
- It sources `~/.profile`, stages the matching CLI auth material into the container, then installs the requested live CLI (`@anthropic-ai/claude-code`, `@openai/codex`, Factory Droid via `https://app.factory.ai/cli`, `@google/gemini-cli`, or `opencode-ai`) if missing. The ACP backend itself is the bundled embedded `acpx/runtime` package from the `acpx` plugin.
- The Droid Docker variant stages `~/.factory` for settings, forwards `FACTORY_API_KEY`, and requires that API key because local Factory OAuth/keyring auth is not portable into the container. It uses ACPX’s built-in `droid exec --output-format acp` registry entry.
- The OpenCode Docker variant is a strict single-agent regression lane. It writes a temporary `OPENCODE_CONFIG_CONTENT` default model from `OPENCLAW_LIVE_ACP_BIND_OPENCODE_MODEL` (default `opencode/kimi-k2.6`) after sourcing `~/.profile`, and `pnpm test:docker:live-acp-bind:opencode` requires a bound assistant transcript instead of accepting the generic post-bind skip.
- Direct `acpx` CLI calls are only a manual/workaround path for comparing behavior outside the Gateway. The Docker ACP bind smoke exercises OpenClaw’s embedded `acpx` runtime backend.

## [​](https://docs.openclaw.ai/help/testing-live\\#live-codex-app-server-harness-smoke)  Live: Codex app-server harnes...(content truncated)

---

## FAQ: models and auth
**Source:** https://docs.openclaw.ai/help/faq-models

[Skip to main content](https://docs.openclaw.ai/help/faq-models#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

FAQ

FAQ: models and auth

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Models: defaults, selection, aliases, switching](https://docs.openclaw.ai/help/faq-models#models-defaults-selection-aliases-switching)
- [Model failover and “All models failed”](https://docs.openclaw.ai/help/faq-models#model-failover-and-%E2%80%9Call-models-failed%E2%80%9D)
- [Auth profiles: what they are and how to manage them](https://docs.openclaw.ai/help/faq-models#auth-profiles-what-they-are-and-how-to-manage-them)
- [Related](https://docs.openclaw.ai/help/faq-models#related)

Model- and auth-profile Q&A. For setup, sessions, gateway, channels, and
troubleshooting, see the main [FAQ](https://docs.openclaw.ai/help/faq).

## [​](https://docs.openclaw.ai/help/faq-models\\#models-defaults-selection-aliases-switching)  Models: defaults, selection, aliases, switching

What is the \"default model\"?

OpenClaw’s default model is whatever you set as:

```
agents.defaults.model.primary
```

Models are referenced as `provider/model` (example: `openai/gpt-5.5` or `openai-codex/gpt-5.5`). If you omit the provider, OpenClaw first tries an alias, then a unique configured-provider match for that exact model id, and only then falls back to the configured default provider as a deprecated compatibility path. If that provider no longer exposes the configured default model, OpenClaw falls back to the first configured provider/model instead of surfacing a stale removed-provider default. You should still **explicitly** set `provider/model`.

What model do you recommend?

**Recommended default:** use the strongest latest-generation model available in your provider stack.
**For tool-enabled or untrusted-input agents:** prioritize model strength over cost.
**For routine/low-stakes chat:** use cheaper fallback models and route by agent role.MiniMax has its own docs: [MiniMax](https://docs.openclaw.ai/providers/minimax) and
[Local models](https://docs.openclaw.ai/gateway/local-models).Rule of thumb: use the **best model you can afford** for high-stakes work, and a cheaper
model for routine chat or summaries. You can route models per agent and use sub-agents to
parallelize long tasks (each sub-agent consumes tokens). See [Models](https://docs.openclaw.ai/concepts/models) and
[Sub-agents](https://docs.openclaw.ai/tools/subagents).Strong warning: weaker/over-quantized models are more vulnerable to prompt
injection and unsafe behavior. See [Security](https://docs.openclaw.ai/gateway/security).More context: [Models](https://docs.openclaw.ai/concepts/models).

How do I switch models without wiping my config?

Use **model commands** or edit only the **model** fields. Avoid full config replaces.Safe options:

- `/model` in chat (quick, per-session)
- `openclaw models set ...` (updates just model config)
- `openclaw configure --section model` (interactive)
- edit `agents.defaults.model` in `~/.openclaw/openclaw.json`

Avoid `config.apply` with a partial object unless you intend to replace the whole config.
For RPC edits, inspect with `config.schema.lookup` first and prefer `config.patch`. The lookup payload gives you the normalized path, shallow schema docs/constraints, and immediate child summaries.
for partial updates.
If you did overwrite config, restore from backup or re-run `openclaw doctor` to repair.Docs: [Models](https://docs.openclaw.ai/concepts/models), [Configure](https://docs.openclaw.ai/cli/configure), [Config](https://docs.openclaw.ai/cli/config), [Doctor](https://docs.openclaw.ai/gateway/doctor).

Can I use self-hosted models (llama.cpp, vLLM, Ollama)?

Yes. Ollama is the easiest path for local models.Quickest setup:

1. Install Ollama from `https://ollama.com/download`
2. Pull a local model such as `ollama pull gemma4`
3. If you want cloud models too, run `ollama signin`
4. Run `openclaw onboard` and choose `Ollama`
5. Pick `Local` or `Cloud + Local`

Notes:

- `Cloud + Local` gives you cloud models plus your local Ollama models
- cloud models such as `kimi-k2.5:cloud` do not need a local pull
- for manual switching, use `openclaw models list` and `openclaw models set ollama/<model>`

Security note: smaller or heavily quantized models are more vulnerable to prompt
injection. We strongly recommend **large models** for any bot that can use tools.
If you still want small models, enable sandboxing and strict tool allowlists.Docs: [Ollama](https://docs.openclaw.ai/providers/ollama), [Local models](https://docs.openclaw.ai/gateway/local-models),
[Model providers](https://docs.openclaw.ai/concepts/model-providers), [Security](https://docs.openclaw.ai/gateway/security),
[Sandboxing](https://docs.openclaw.ai/gateway/sandboxing).

What do OpenClaw, Flawd, and Krill use for models?

- These deployments can differ and may change over time; there is no fixed provider recommendation.
- Check the current runtime setting on each gateway with `openclaw models status`.
- For security-sensitive/tool-enabled agents, use the strongest latest-generation model available.

How do I switch models on the fly (without restarting)?

Use the `/model` command as a standalone message:

```
/model sonnet
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
/model gemini-flash-lite
```

These are the built-in aliases. Custom aliases can be added via `agents.defaults.models`.You can list available models with `/model`, `/model list`, or `/model status`.`/model` (and `/model list`) shows a compact, numbered picker. Select by number:

```
/model 3
```

You can also force a specific auth profile for the provider (per session):

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Tip: `/model status` shows which agent is active, which `auth-profiles.json` file is being used, and which auth profile will be tried next.
It also shows the configured provider endpoint (`baseUrl`) and API mode (`api`) when available.**How do I unpin a profile I set with @profile?**Re-run `/model` **without** the `@profile` suffix:

```
/model anthropic/claude-opus-4-6
```

If you want to return to the default, pick it from `/model` (or send `/model <default provider/model>`).
Use `/model status` to confirm which auth profile is active.

Can I use GPT 5.5 for daily tasks and Codex 5.5 for coding?

Yes. Set one as default and switch as needed:

- **Quick switch (per session):**`/model openai/gpt-5.5` for current direct OpenAI API-key tasks or `/model openai-codex/gpt-5.5` for GPT-5.5 Codex OAuth tasks.
- **Default:** set `agents.defaults.model.primary` to `openai/gpt-5.5` for API-key usage or `openai-codex/gpt-5.5` for GPT-5.5 Codex OAuth usage.
- **Sub-agents:** route coding tasks to sub-agents with a different default model.

See [Models](https://docs.openclaw.ai/concepts/models) and [Slash commands](https://docs.openclaw.ai/tools/slash-commands).

How do I configure fast mode for GPT 5.5?

Use either a session toggle or a config default:

- **Per session:** send `/fast on` while the session is using `openai/gpt-5.5` or `openai-codex/gpt-5.5`.
- **Per model default:** set `agents.defaults.models[\"openai/gpt-5.5\"].params.fastMode` or `agents.defaults.models[\"openai-codex/gpt-5.5\"].params.fastMode` to `true`.

Example:

```
{
  agents: {
    defaults: {
      models: {
        \"openai/gpt-5.5\": {
          params: {
            fastMode: true,
          },
        },
      },
    },
  },
}
```

For OpenAI, fast mode maps to `service_tier = \"priority\"` on supported native Responses requests. Session `/fast` overrides beat config defaults.See [Thinking and fast mode](https://docs.openclaw.ai/tools/thinking) and [OpenAI fast mode](https://docs.openclaw.ai/providers/openai#fast-mode).

Why do I see \"Model ... is not allowed\" and then no reply?

If `agents.defaults.models` is set, it becomes the **allowlist** for `/model` and any
session overrides. Choosing a model that isn’t in that list returns:

```
Model \"provider/model\" is not allowed. Use /model to list available models.
```

That error is returned **instead of** a normal reply. Fix: add the model to
`agents.defaults.models`, remove the allowlist, or pick a model from `/model list`.

Why do I see \"Unknown model: minimax/MiniMax-M2.7\"?

This means the **provider isn’t configured** (no MiniMax provider config or auth
profile was found), so the model can’t be resolved.Fix checklist:

1. Upgrade to a current OpenClaw release (or run from source `main`), then restart the gateway.
2. Make sure MiniMax is configured (wizard or JSON), or that MiniMax auth
exists in env/auth profiles so the matching provider can be injected
(`MINIMAX_API_KEY` for `minimax`, `MINIMAX_OAUTH_TOKEN` or stored MiniMax
OAuth for `minimax-portal`).
3. Use the exact model id (case-sensitive) for your auth path:
`minimax/MiniMax-M2.7` or `minimax/MiniMax-M2.7-highspeed` for API-key
setup, or `minimax-portal/MiniMax-M2.7` /
`minimax-portal/MiniMax-M2.7-highspeed` for OAuth setup.
4. Run:














```
openclaw models list
```










and pick from the list (or `/model list` in chat).

See [MiniMax](https://docs.openclaw.ai/providers/minimax) and [Models](https://docs.openclaw.ai/concepts/models).

Can I use MiniMax as my default and OpenAI for complex tasks?

Yes. Use **MiniMax as the default** and switch models **per session** when needed.
Fallbacks are for **errors**, not “hard tasks,” so use `/model` or a separate agent.**Option A: switch per session**

```
{
  env: { MINIMAX_API_KEY: \"sk-...\", OPENAI_API_KEY: \"sk-...\" },
  agents: {
    defaults: {
      model: { primary: \"minimax/MiniMax-M2.7\" },
      models: {
        \"minimax/MiniMax-M2.7\": { alias: \"minimax\" },
        \"openai/gpt-5.5\": { alias: \"gpt\" },
      },
    },
  },
}
```

Then:

```
/model gpt
```

**Option B: separate agents**

- Agent A default: MiniMax
- Agent B default: OpenAI
- Route by agent or use `/agent` to switch

Docs: [Models](https://docs.openclaw.ai/concepts/models), [Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent), [MiniMax](https://docs.openclaw.ai/providers/minimax), [OpenAI](https://docs.openclaw.ai/providers/openai).

Are opus / sonnet / gpt built-in shortcuts?

Yes. OpenClaw ships a few default shorthands (only applied when the model exists in `agents.defaults.models`):

- `opus` → `anthropic/claude-opus-4-6`
- `sonnet` → `anthropic/claude-sonnet-4-6`
- `gpt` → `openai/gpt-5.5` for API-key setups, or `openai-codex/gpt-5.5` when configured for Codex OAuth
- `gpt-mini` → `openai/gpt-5.4-mini`
- `gpt-nano` → `openai/gpt-5.4-nano`
- `gemini` → `google/gemini-3.1-pro-preview`
- `gemini-flash` → `google/gemini-3-flash-preview`
- `gemini-flash-lite` → `google/gemini-3.1-flash-lite-preview`

If you set your own alias with the same name, your value wins.

How do I define/override model shortcuts (aliases)?

Aliases come from `agents.defaults.models.<modelId>.alias`. Example:

```
{
  agents: {
    defaults: {
      model: { primary: \"anthropic/claude-opus-4-6\" },
      models: {
        \"anthropic/claude-opus-4-6\": { alias: \"opus\" },
        \"anthropic/claude-sonnet-4-6\": { alias: \"sonnet\" },
        \"anthropic/claude-haiku-4-5\": { alias: \"haiku\" },
      },
    },
  },
}
```

Then `/model sonnet` (or `/<alias>` when supported) resolves to that model ID.

How do I add models from other providers like OpenRouter or Z.AI?

OpenRouter (pay-per-token; many models):

```
{
  agents: {
    defaults: {
      model: { primary: \"openrouter/anthropic/claude-sonnet-4-6\" },
      models: { \"openrouter/anthropic/claude-sonnet-4-6\": {} },
    },
  },
  env: { OPENROUTER_API_KEY: \"sk-or-...\" },
}
```

Z.AI (GLM models):

```
{
  agents: {
    defaults: {
      model: { primary: \"zai/glm-5\" },
      models: { \"zai/glm-5\": {} },
    },
  },
  env: { ZAI_API_KEY: \"...\" },
}
```

If you reference a provider/model but the required provider key is missing, you’ll get a runtime auth error (e.g. `No API key found for provider \"zai\"`).**No API key found for provider after adding a new agent**This usually means the **new agent** has an empty auth store. Auth is per-agent and
stored in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Fix options:

- Run `openclaw agents add <id>` and configure auth during the wizard.
- Or copy `auth-profiles.json` from the main agent’s `agentDir` into the new agent’s `agentDir`.

Do **not** reuse `agentDir` across agents; it causes auth/session collisions.

## [​](https://docs.openclaw.ai/help/faq-models\\#model-failover-and-%E2%80%9Call-models-failed%E2%80%9D)  Model failover and “All models failed”

How does failover work?

Failover happens in two stages:

1. **Auth profile rotation** within the same provider.
2. **Model fallback** to the next model in `agents.defaults.model.fallbacks`.

Cooldowns apply to failing profiles (exponential backoff), so OpenClaw can keep responding even when a provider is rate-limited or temporarily failing.The rate-limit bucket includes more than plain `429` responses. OpenClaw
also treats messages like `Too many concurrent requests`,
`ThrottlingException`, `concurrency limit reached`,
`workers_ai ... quota limit exceeded`, `resource exhausted`, and periodic
usage-window limits (`weekly/monthly limit reached`) as failover-worthy
rate limits.Some billing-looking responses are not `402`, and some HTTP `402`
responses also stay in that transient bucket. If a provider returns
explicit billing text on `401` or `403`, OpenClaw can still keep that in
the billing lane, but provider-specific text matchers stay scoped to the
provider that owns them (for example OpenRouter `Key limit exceeded`). If a `402`
message instead looks like a retryable usage-window or
organization/workspace spend limit (`daily limit reached, resets tomorrow`,
`organization spending limit exceeded`), OpenClaw treats it as
`rate_limit`, not a long billing disable.Context-overflow errors are different: signatures such as
`request_too_large`, `input exceeds the maximum number of tokens`,
`input token count exceeds the maximum number of input tokens`,
`input is too long for the model`, or `ollama error: context length         exceeded` stay on the compaction/retry path instead of advancing model
fallback.Generic server-error text is intentionally narrower than “anything with
unknown/error in it”. OpenClaw does treat provider-scoped transient shapes
such as Anthropic bare `An unknown error occurred`, OpenRouter bare
`Provider returned error`, stop-reason errors like `Unhandled stop reason:         error`, JSON `api_error` payloads with transient server text
(`internal server error`, `unknown error, 520`, `upstream error`, `backend         error`), and provider-busy errors such as `ModelNotReadyException` as
failover-worthy timeout/overloaded signals when the provider context
matches.
Generic internal fallback text like `LLM request failed with an unknown         error.` stays conservative and does not trigger model fallback by itself.

What does \"No credentials found for profile anthropic:default\" mean?\n
It means the system attempted to use the auth profile ID `anthropic:default`, but could not find credentials for it in the expected auth store.**Fix checklist:**

- **Confirm where auth profiles live**(new vs legacy paths)

  - Current: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  - Legacy: `~/.openclaw/agent/*` (migrated by `openclaw doctor`)
- **Confirm your env var is loaded by the Gateway**
  - If you set `ANTHROPIC_API_KEY` in your shell but run the Gateway via systemd/launchd, it may not inherit it. Put it in `~/.openclaw/.env` or enable `env.shellEnv`.
- **Make sure you’re editing the correct agent**
  - Multi-agent setups mean there can be multiple `auth-profiles.json` files.
- **Sanity-check model/auth status**
  - Use `openclaw models status` to see configured models and whether providers are authenticated.

**Fix checklist for “No credentials found for profile anthropic”**This means the run is pinned to an Anthropic auth profile, but the Gateway
can’t find it in its auth store.

- **Use Claude CLI**  - Run `openclaw models auth login --provider anthropic --method cli --set-default` on the gateway host.
- **If you want to use an API key instead**  - Put `ANTHROPIC_API_KEY` in `~/.openclaw/.env` on the **gateway host**.
  - Clear any pinned order that forces a missing profile:














    ```
    openclaw models auth order clear --provider anthropic
    ```
- **Confirm you’re running commands on the gateway host**  - In remote mode, auth profiles live on the gateway machine, not your laptop.

Why did it also try Google Gemini and fail?\n
If your model config includes Google Gemini as a fallback (or you switched to a Gemini shorthand), OpenClaw will try it during model fallback. If you haven’t configured Google credentials, you’ll see `No API key found for provider \"google\"`.Fix: either provide Google auth, or remove/avoid Google models in `agents.defaults.model.fallbacks` / aliases so fallback doesn’t route there.**LLM request rejected: thinking signature required (Google Antigravity)**Cause: the session history contains **thinking blocks without signatures** (often from\nan aborted/partial stream). Google Antigravity requires signatures for thinking blocks.Fix: OpenClaw now strips unsigned thinking blocks for Google Antigravity Claude. If it still appears, start a **new session** or set `/thinking off` for that agent.\n\n## [​](https://docs.openclaw.ai/help/faq-models\\#auth-profiles-what-they-are-and-how-to-manage-them)  Auth profiles: what they are and how to manage them\n
Related: [/concepts/oauth](https://docs.openclaw.ai/concepts/oauth) (OAuth flows, token storage, multi-account patterns)\n
What is an auth profile?\n
An auth profile is a named credential record (OAuth or API key) tied to a provider. Profiles live in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

What are typical profile IDs?\n
OpenClaw uses provider-prefixed IDs like:

- `anthropic:default` (common when no email identity exists)\n- `anthropic:<email>` for OAuth identities\n- custom IDs you choose (e.g. `anthropic:work`)\n
Can I control which auth profile is tried first?\n
Yes. Config supports optional metadata for profil...(content truncated)

---

## Debugging - OpenClaw
**Source:** https://docs.openclaw.ai/help/debugging

[Skip to main content](https://docs.openclaw.ai/help/debugging#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Start here

Debugging

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Runtime debug overrides](https://docs.openclaw.ai/help/debugging#runtime-debug-overrides)
- [Session trace output](https://docs.openclaw.ai/help/debugging#session-trace-output)
- [Temporary CLI debug timing](https://docs.openclaw.ai/help/debugging#temporary-cli-debug-timing)
- [Add temporary spans](https://docs.openclaw.ai/help/debugging#add-temporary-spans)
- [Run with readable output](https://docs.openclaw.ai/help/debugging#run-with-readable-output)
- [Run with JSON output](https://docs.openclaw.ai/help/debugging#run-with-json-output)
- [Clean up before landing](https://docs.openclaw.ai/help/debugging#clean-up-before-landing)
- [Gateway watch mode](https://docs.openclaw.ai/help/debugging#gateway-watch-mode)
- [Dev profile + dev gateway (—dev)](https://docs.openclaw.ai/help/debugging#dev-profile-%2B-dev-gateway-%E2%80%94dev)
- [Raw stream logging (OpenClaw)](https://docs.openclaw.ai/help/debugging#raw-stream-logging-openclaw)
- [Raw chunk logging (pi-mono)](https://docs.openclaw.ai/help/debugging#raw-chunk-logging-pi-mono)
- [Safety notes](https://docs.openclaw.ai/help/debugging#safety-notes)
- [Related](https://docs.openclaw.ai/help/debugging#related)

Debugging helpers for streaming output, especially when a provider mixes reasoning into normal text.

## [​](https://docs.openclaw.ai/help/debugging\\#runtime-debug-overrides)  Runtime debug overrides

Use `/debug` in chat to set **runtime-only** config overrides (memory, not disk).
`/debug` is disabled by default; enable with `commands.debug: true`.
This is handy when you need to toggle obscure settings without editing `openclaw.json`.Examples:

```
/debug show
/debug set messages.responsePrefix=\"[openclaw]\"\n/debug unset messages.responsePrefix\n/debug reset
```

`/debug reset` clears all overrides and returns to the on-disk config.

## [​](https://docs.openclaw.ai/help/debugging\\#session-trace-output)  Session trace output

Use `/trace` when you want to see plugin-owned trace/debug lines in one session
without turning on full verbose mode.Examples:

```
/trace
/trace on
/trace off
```

Use `/trace` for plugin diagnostics such as Active Memory debug summaries.
Keep using `/verbose` for normal verbose status/tool output, and keep using
`/debug` for runtime-only config overrides.

## [​](https://docs.openclaw.ai/help/debugging\\#temporary-cli-debug-timing)  Temporary CLI debug timing

OpenClaw keeps `src/cli/debug-timing.ts` as a small helper for local
investigation. It is intentionally not wired into CLI startup, command routing,
nor any command by default. Use it only while debugging a slow command, then
remove the import and spans before landing the behavior change.Use this when a command is slow and you need a quick phase breakdown before
deciding whether to use a CPU profiler or fix a specific subsystem.

### [​](https://docs.openclaw.ai/help/debugging\\#add-temporary-spans)  Add temporary spans

Add the helper near the code you are investigating. For example, while debugging
`openclaw models list`, a temporary patch in
`src/commands/models/list.list-command.ts` might look like this:

```
// Temporary debugging only. Remove before landing.
import { createCliDebugTiming } from \"../../cli/debug-timing.js\";

const timing = createCliDebugTiming({ command: \"models list\" });

const authStore = timing.time(\"debug:models:list:auth_store\", () => ensureAuthProfileStore());

const loaded = await timing.timeAsync(\n  \"debug:models:list:registry\",\n  () => loadListModelRegistry(cfg, { sourceConfig }),\n  (result) => ({\n    models: result.models.length,\n    discoveredKeys: result.discoveredKeys.size,\n  }),\n);
```

Guidelines:

- Prefix temporary phase names with `debug:`.
- Add only a few spans around suspected slow sections.
- Prefer broad phases such as `registry`, `auth_store`, or `rows` over helper
names.
- Use `time()` for synchronous work and `timeAsync()` for promises.
- Keep stdout clean. The helper writes to stderr, so command JSON output stays
parseable.
- Remove temporary imports and spans before opening the final fix PR.
- Include the timing output or a short summary in the issue or PR that explains
the optimization.

### [​](https://docs.openclaw.ai/help/debugging\\#run-with-readable-output)  Run with readable output

Readable mode is best for live debugging:

```
OPENCLAW_DEBUG_TIMING=1 pnpm openclaw models list --all --provider moonshot
```

Example output from a temporary `models list` investigation:

```
OpenClaw CLI debug timing: models list
     0ms     +0ms start all=true json=false local=false plain=false provider=\"moonshot\"
     2ms     +2ms debug:models:list:import_runtime duration=2ms
    17ms    +14ms debug:models:list:load_config duration=14ms sourceConfig=true
  20.3s  +20.3s debug:models:list:auth_store duration=20.3s
  20.3s     +0ms debug:models:list:resolve_agent_dir duration=0ms agentDir=true
  20.3s     +0ms debug:models:list:resolve_provider_filter duration=0ms
  25.3s   +5.0s debug:models:list:ensure_models_json duration=5.0s
  31.2s   +5.9s debug:models:list:load_model_registry duration=5.9s models=869 availableKeys=38 discoveredKeys=868 availabilityError=false
  31.2s     +0ms debug:models:list:resolve_configured_entries duration=0ms entries=1
  31.2s     +0ms debug:models:list:build_configured_lookup duration=0ms entries=1
  33.6s   +2.4s debug:models:list:read_registry_models duration=2.4s models=871
  35.2s   +1.5s debug:models:list:append_discovered_rows duration=1.5s seenKeys=0 rows=0
  36.9s   +1.7s debug:models:list:append_catalog_supplement_rows duration=1.7s seenKeys=5 rows=5

Model                                      Input       Ctx   Local Auth  Tags
moonshot/kimi-k2-thinking                  text        256k  no    no
moonshot/kimi-k2-thinking-turbo            text        256k  no    no
moonshot/kimi-k2-turbo                     text        250k  no    no
moonshot/kimi-k2.5                         text+image  256k  no    no
moonshot/kimi-k2.6                         text+image  256k  no    no

  36.9s     +0ms debug:models:list:print_model_table duration=0ms rows=5
  36.9s     +0ms complete rows=5
```

Findings from this output:

| Phase | Time | What it means |
| --- | --- | --- |
| `debug:models:list:auth_store` | 20.3s | The auth-profile store load is the largest cost and should be investigated first. |
| `debug:models:list:ensure_models_json` | 5.0s | Syncing `models.json` is expensive enough to inspect for caching or skip conditions. |
| `debug:models:list:load_model_registry` | 5.9s | Registry construction and provider availability work are also meaningful costs. |
| `debug:models:list:read_registry_models` | 2.4s | Reading all registry models is not free and may matter for `--all`. |
| row append phases | 3.2s total | Building five displayed rows still takes several seconds, so the filtering path deserves a closer look. |
| `debug:models:list:print_model_table` | 0ms | Rendering is not the bottleneck. |

Those findings are enough to guide the next patch without keeping timing code in
production paths.

### [​](https://docs.openclaw.ai/help/debugging\\#run-with-json-output)  Run with JSON output

Use JSON mode when you want to save or compare timing data:

```
OPENCLAW_DEBUG_TIMING=json pnpm openclaw models list --all --provider moonshot \\\n  2> .artifacts/models-list-timing.jsonl
```

Each stderr line is one JSON object:

```
{
  \"command\": \"models list\",
  \"phase\": \"debug:models:list:registry\",
  \"elapsedMs\": 31200,
  \"deltaMs\": 5900,
  \"durationMs\": 5900,
  \"models\": 869,
  \"discoveredKeys\": 868
}
```

### [​](https://docs.openclaw.ai/help/debugging\\#clean-up-before-landing)  Clean up before landing

Before opening the final PR:

```
rg \'createCliDebugTiming|debug:[a-z0-9_-]+:\' src/commands src/cli \\\n  --glob \'!src/cli/debug-timing.*\' \\\n  --glob \'!*.test.ts\'
```

The command should return no temporary instrumentation call sites unless the PR
is explicitly adding a permanent diagnostics surface. For normal performance
fixes, keep only the behavior change, tests, and a short note with the timing
evidence.For deeper CPU hotspots, use Node profiling (`--cpu-prof`) or an external
profiler instead of adding more timing wrappers.

## [​](https://docs.openclaw.ai/help/debugging\\#gateway-watch-mode)  Gateway watch mode

For fast iteration, run the gateway under the file watcher:

```
pnpm gateway:watch
```

This maps to:

```
node scripts/watch-node.mjs gateway --force
```

The watcher restarts on build-relevant files under `src/`, extension source files,
extension `package.json` and `openclaw.plugin.json` metadata, `tsconfig.json`,
`package.json`, and `tsdown.config.ts`. Extension metadata changes restart the
gateway without forcing a `tsdown` rebuild; source and config changes still
rebuild `dist` first.Add any gateway CLI flags after `gateway:watch` and they will be passed through on
each restart. Re-running the same watch command for the same repo/flag set now
replaces the older watcher instead of leaving duplicate watcher parents behind.

## [​](https://docs.openclaw.ai/help/debugging\\#dev-profile-+-dev-gateway-%E2%80%94dev)  Dev profile + dev gateway (—dev)

Use the dev profile to isolate state and spin up a safe, disposable setup for
debugging. There are **two**`--dev` flags:

- **Global `--dev` (profile):** isolates state under `~/.openclaw-dev` and
defaults the gateway port to `19001` (derived ports shift with it).
- **`gateway --dev`: tells the Gateway to auto-create a default config +**
**workspace** when missing (and skip BOOTSTRAP.md).

Recommended flow (dev profile + dev bootstrap):

```
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

If you don’t have a global install yet, run the CLI via `pnpm openclaw ...`.What this does:

1. **Profile isolation** (global `--dev`)   - `OPENCLAW_PROFILE=dev`
   - `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   - `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   - `OPENCLAW_GATEWAY_PORT=19001` (browser/canvas shift accordingly)
2. **Dev bootstrap** (`gateway --dev`)   - Writes a minimal config if missing (`gateway.mode=local`, bind loopback).
   - Sets `agent.workspace` to the dev workspace.
   - Sets `agent.skipBootstrap=true` (no BOOTSTRAP.md).
   - Seeds the workspace files if missing:
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
   - Default identity: **C3‑PO** (protocol droid).
   - Skips channel providers in dev mode (`OPENCLAW_SKIP_CHANNELS=1`).

Reset flow (fresh start):

```
pnpm gateway:dev:reset
```

`--dev` is a **global** profile flag and gets eaten by some runners. If you need to spell it out, use the env var form:

```
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` wipes config, credentials, sessions, and the dev workspace (using
`trash`, not `rm`), then recreates the default dev setup.

If a non-dev gateway is already running (launchd or systemd), stop it first:

```
openclaw gateway stop
```

## [​](https://docs.openclaw.ai/help/debugging\\#raw-stream-logging-openclaw)  Raw stream logging (OpenClaw)

OpenClaw can log the **raw assistant stream** before any filtering/formatting.
This is the best way to see whether reasoning is arriving as plain text deltas
(or as separate thinking blocks).Enable it via CLI:

```
pnpm gateway:watch --raw-stream
```

Optional path override:

```
pnpm gateway:watch --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

Equivalent env vars:

```
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

Default file:`~/.openclaw/logs/raw-stream.jsonl`

## [​](https://docs.openclaw.ai/help/debugging\\#raw-chunk-logging-pi-mono)  Raw chunk logging (pi-mono)

To capture **raw OpenAI-compat chunks** before they are parsed into blocks,
pi-mono exposes a separate logger:

```
PI_RAW_STREAM=1
```

Optional path:

```
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

Default file:`~/.pi-mono/logs/raw-openai-completions.jsonl`

> Note: this is only emitted by processes using pi-mono’s
> `openai-completions` provider.

## [​](https://docs.openclaw.ai/help/debugging\\#safety-notes)  Safety notes

- Raw stream logs can include full prompts, tool output, and user data.
- Keep logs local and delete them after debugging.
- If you share logs, scrub secrets and PII first.

## [​](https://docs.openclaw.ai/help/debugging\\#related)  Related

- [Troubleshooting](https://docs.openclaw.ai/help/troubleshooting)
- [FAQ](https://docs.openclaw.ai/help/faq)

[General troubleshooting](https://docs.openclaw.ai/help/troubleshooting) [FAQ](https://docs.openclaw.ai/help/faq)

Ctrl+I

---

## GPT-5.5 / Codex agentic parity - OpenClaw
**Source:** https://docs.openclaw.ai/help/gpt55-codex-agentic-parity

[Skip to main content](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Concept internals

GPT-5.5 / Codex agentic parity

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [GPT-5.5 / Codex Agentic Parity in OpenClaw](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#gpt-5-5-%2F-codex-agentic-parity-in-openclaw)
- [What changed](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#what-changed)
- [PR A: strict-agentic execution](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#pr-a-strict-agentic-execution)
- [PR B: runtime truthfulness](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#pr-b-runtime-truthfulness)
- [PR C: execution correctness](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#pr-c-execution-correctness)
- [PR D: parity harness](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#pr-d-parity-harness)
- [Why this improves GPT-5.5 in practice](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#why-this-improves-gpt-5-5-in-practice)
- [Before vs after for GPT-5.5 users](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#before-vs-after-for-gpt-5-5-users)
- [Architecture](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#architecture)
- [Release flow](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#release-flow)
- [Scenario pack](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#scenario-pack)
- [approval-turn-tool-followthrough](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#approval-turn-tool-followthrough)
- [model-switch-tool-continuity](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#model-switch-tool-continuity)
- [source-docs-discovery-report](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#source-docs-discovery-report)
- [image-understanding-attachment](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#image-understanding-attachment)
- [compaction-retry-mutating-tool](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#compaction-retry-mutating-tool)
- [Scenario matrix](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#scenario-matrix)
- [Release gate](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#release-gate)
- [Goal-to-evidence matrix](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#goal-to-evidence-matrix)
- [How to read the parity verdict](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#how-to-read-the-parity-verdict)
- [Who should enable strict-agentic](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#who-should-enable-strict-agentic)
- [Related](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity#related)

# [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#gpt-5-5-/-codex-agentic-parity-in-openclaw)  GPT-5.5 / Codex Agentic Parity in OpenClaw

OpenClaw already worked well with tool-using frontier models, but GPT-5.5 and Codex-style models were still underperforming in a few practical ways:

- they could stop after planning instead of doing the work
- they could use strict OpenAI/Codex tool schemas incorrectly
- they could ask for `/elevated full` even when full access was impossible
- they could lose long-running task state during replay or compaction
- parity claims against Claude Opus 4.6 were based on anecdotes instead of repeatable scenarios

This parity program fixes those gaps in four reviewable slices.

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#what-changed)  What changed

### [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#pr-a-strict-agentic-execution)  PR A: strict-agentic execution

This slice adds an opt-in `strict-agentic` execution contract for embedded Pi GPT-5 runs.When enabled, OpenClaw stops accepting plan-only turns as “good enough” completion. If the model only says what it intends to do and does not actually use tools or make progress, OpenClaw retries with an act-now steer and then fails closed with an explicit blocked state instead of silently ending the task.This improves the GPT-5.5 experience most on:

- short “ok do it” follow-ups
- code tasks where the first step is obvious
- flows where `update_plan` should be progress tracking rather than filler text

### [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#pr-b-runtime-truthfulness)  PR B: runtime truthfulness

This slice makes OpenClaw tell the truth about two things:

- why the provider/runtime call failed
- whether `/elevated full` is actually available

That means GPT-5.5 gets better runtime signals for missing scope, auth refresh failures, HTML 403 auth failures, proxy issues, DNS or timeout failures, and blocked full-access modes. The model is less likely to hallucinate the wrong remediation or keep asking for a permission mode the runtime cannot provide.

### [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#pr-c-execution-correctness)  PR C: execution correctness

This slice improves two kinds of correctness:

- provider-owned OpenAI/Codex tool-schema compatibility
- replay and long-task liveness surfacing

The tool-compat work reduces schema friction for strict OpenAI/Codex tool registration, especially around parameter-free tools and strict object-root expectations. The replay/liveness work makes long-running tasks more observable, so paused, blocked, and abandoned states are visible instead of disappearing into generic failure text.

### [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#pr-d-parity-harness)  PR D: parity harness

This slice adds the first-wave QA-lab parity pack so GPT-5.5 and Opus 4.6 can be exercised through the same scenarios and compared using shared evidence.The parity pack is the proof layer. It does not change runtime behavior by itself.After you have two `qa-suite-summary.json` artifacts, generate the release-gate comparison with:

```
pnpm openclaw qa parity-report \\
  --repo-root . \\
  --candidate-summary .artifacts/qa-e2e/gpt55/qa-suite-summary.json \\
  --baseline-summary .artifacts/qa-e2e/opus46/qa-suite-summary.json \\
  --output-dir .artifacts/qa-e2e/parity
```

That command writes:

- a human-readable Markdown report
- a machine-readable JSON verdict
- an explicit `pass` / `fail` gate result

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#why-this-improves-gpt-5-5-in-practice)  Why this improves GPT-5.5 in practice

Before this work, GPT-5.5 on OpenClaw could feel less agentic than Opus in real coding sessions because the runtime tolerated behaviors that are especially harmful for GPT-5-style models:

- commentary-only turns
- schema friction around tools
- vague permission feedback
- silent replay or compaction breakage

The goal is not to make GPT-5.5 imitate Opus. The goal is to give GPT-5.5 a runtime contract that rewards real progress, supplies cleaner tool and permission semantics, and turns failure modes into explicit machine- and human-readable states.That changes the user experience from:

- “the model had a good plan but stopped”

to:

- “the model either acted, or OpenClaw surfaced the exact reason it could not”

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#before-vs-after-for-gpt-5-5-users)  Before vs after for GPT-5.5 users

| Before this program | After PR A-D |
| --- | --- |
| GPT-5.5 could stop after a reasonable plan without taking the next tool step | PR A turns “plan only” into “act now or surface a blocked state” |
| Strict tool schemas could reject parameter-free or OpenAI/Codex-shaped tools in confusing ways | PR C makes provider-owned tool registration and invocation more predictable |
| `/elevated full` guidance could be vague or wrong in blocked runtimes | PR B gives GPT-5.5 and the user truthful runtime and permission hints |
| Replay or compaction failures could feel like the task silently disappeared | PR C surfaces paused, blocked, abandoned, and replay-invalid outcomes explicitly |
| “GPT-5.5 feels worse than Opus” was mostly anecdotal | PR D turns that into the same scenario pack, the same metrics, and a hard pass/fail gate |

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#architecture)  Architecture

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#release-flow)  Release flow

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#scenario-pack)  Scenario pack

The first-wave parity pack currently covers five scenarios:

### [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#approval-turn-tool-followthrough)  `approval-turn-tool-followthrough`

Checks that the model does not stop at “I’ll do that” after a short approval. It should take the first concrete action in the same turn.

### [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#model-switch-tool-continuity)  `model-switch-tool-continuity`

Checks that tool-using work remains coherent across model/runtime switching boundaries instead of resetting into commentary or losing execution context.

### [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#source-docs-discovery-report)  `source-docs-discovery-report`

Checks that the model can read source and docs, synthesize findings, and continue the task agentically rather than producing a thin summary and stopping early.

### [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#image-understanding-attachment)  `image-understanding-attachment`

Checks that mixed-mode tasks involving attachments remain actionable and do not collapse into vague narration.

### [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#compaction-retry-mutating-tool)  `compaction-retry-mutating-tool`

Checks that a task with a real mutating write keeps replay-unsafety explicit instead of quietly looking replay-safe if the run compacts, retries, or loses reply state under pressure.

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#scenario-matrix)  Scenario matrix

| Scenario | What it tests | Good GPT-5.5 behavior | Failure signal |
| --- | --- | --- | --- |
| `approval-turn-tool-followthrough` | Short approval turns after a plan | Starts the first concrete tool action immediately instead of restating intent | plan-only follow-up, no tool activity, or blocked turn without a real blocker |
| `model-switch-tool-continuity` | Runtime/model switching under tool use | Preserves task context and continues acting coherently | resets into commentary, loses tool context, or stops after switch |
| `source-docs-discovery-report` | Source reading + synthesis + action | Finds sources, uses tools, and produces a useful report without stalling | thin summary, missing tool work, or incomplete-turn stop |
| `image-understanding-attachment` | Attachment-driven agentic work | Interprets the attachment, connects it to tools, and continues the task | vague narration, attachment ignored, or no concrete next action |
| `compaction-retry-mutating-tool` | Mutating work under compaction pressure | Performs a real write and keeps replay-unsafety explicit after the side effect | mutating write happens but replay safety is implied, missing, or contradictory |

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#release-gate)  Release gate

GPT-5.5 can only be considered at parity or better when the merged runtime passes the parity pack and the runtime-truthfulness regressions at the same time.Required outcomes:

- no plan-only stall when the next tool action is clear
- no fake completion without real execution
- no incorrect `/elevated full` guidance
- no silent replay or compaction abandonment
- parity-pack metrics that are at least as strong as the agreed Opus 4.6 baseline

For the first-wave harness, the gate compares:

- completion rate
- unintended-stop rate
- valid-tool-call rate
- fake-success count

Parity evidence is intentionally split across two layers:

- PR D proves same-scenario GPT-5.5 vs Opus 4.6 behavior with QA-lab
- PR B deterministic suites prove auth, proxy, DNS, and `/elevated full` truthfulness outside the harness

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#goal-to-evidence-matrix)  Goal-to-evidence matrix

| Completion gate item | Owning PR | Evidence source | Pass signal |
| --- | --- | --- | --- |
| GPT-5.5 no longer stalls after planning | PR A | `approval-turn-tool-followthrough` plus PR A runtime suites | approval turns trigger real work or an explicit blocked state |
| GPT-5.5 no longer fakes progress or fake tool completion | PR A + PR D | parity report scenario outcomes and fake-success count | no suspicious pass results and no commentary-only completion |
| GPT-5.5 no longer gives false `/elevated full` guidance | PR B | deterministic truthfulness suites | blocked reasons and full-access hints stay runtime-accurate |
| Replay/liveness failures stay explicit | PR C + PR D | PR C lifecycle/replay suites plus `compaction-retry-mutating-tool` | mutating work keeps replay-unsafety explicit instead of silently disappearing |
| GPT-5.5 matches or beats Opus 4.6 on the agreed metrics | PR D | `qa-agentic-parity-report.md` and `qa-agentic-parity-summary.json` | same scenario coverage and no regression on completion, stop behavior, or valid tool use |

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#how-to-read-the-parity-verdict)  How to read the parity verdict

Use the verdict in `qa-agentic-parity-summary.json` as the final machine-readable decision for the first-wave parity pack.

- `pass` means GPT-5.5 covered the same scenarios as Opus 4.6 and did not regress on the agreed aggregate metrics.
- `fail` means at least one hard gate tripped: weaker completion, worse unintended stops, weaker valid tool use, any fake-success case, or mismatched scenario coverage.
- “shared/base CI issue” is not itself a parity result. If CI noise outside PR D blocks a run, the verdict should wait for a clean merged-runtime execution instead of being inferred from branch-era logs.
- Auth, proxy, DNS, and `/elevated full` truthfulness still come from PR B’s deterministic suites, so the final release claim needs both: a passing PR D parity verdict and green PR B truthfulness coverage.

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#who-should-enable-strict-agentic)  Who should enable `strict-agentic`

Use `strict-agentic` when:

- the agent is expected to act immediately when a next step is obvious
- GPT-5.5 or Codex-family models are the primary runtime
- you prefer explicit blocked states over “helpful” recap-only replies

Keep the default contract when:

- you want the existing looser behavior
- you are not using GPT-5-family models
- you are testing prompts rather than runtime enforcement

## [​](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity\#related)  Related

- [GPT-5.5 / Codex parity maintainer notes](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity-maintainers)

[Timezones](https://docs.openclaw.ai/concepts/timezone) [GPT-5.5 / Codex parity maintainer notes](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity-maintainers)

Ctrl+I

---

