# OpenClaw Install Documentation

## exe.dev - OpenClaw
**Source:** https://docs.openclaw.ai/install/exe-dev

[Skip to main content](https://docs.openclaw.ai/install/exe-dev#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nHosting\n\nexe.dev\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Beginner quick path](https://docs.openclaw.ai/install/exe-dev#beginner-quick-path)\n- [What you need](https://docs.openclaw.ai/install/exe-dev#what-you-need)\n- [Automated install with Shelley](https://docs.openclaw.ai/install/exe-dev#automated-install-with-shelley)\n- [Manual installation](https://docs.openclaw.ai/install/exe-dev#manual-installation)\n- [1) Create the VM](https://docs.openclaw.ai/install/exe-dev#1-create-the-vm)\n- [2) Install prerequisites (on the VM)](https://docs.openclaw.ai/install/exe-dev#2-install-prerequisites-on-the-vm)\n- [3) Install OpenClaw](https://docs.openclaw.ai/install/exe-dev#3-install-openclaw)\n- [4) Setup nginx to proxy OpenClaw to port 8000](https://docs.openclaw.ai/install/exe-dev#4-setup-nginx-to-proxy-openclaw-to-port-8000)\n- [5) Access OpenClaw and grant privileges](https://docs.openclaw.ai/install/exe-dev#5-access-openclaw-and-grant-privileges)\n- [Remote access](https://docs.openclaw.ai/install/exe-dev#remote-access)\n- [Updating](https://docs.openclaw.ai/install/exe-dev#updating)\n- [Related](https://docs.openclaw.ai/install/exe-dev#related)\n\nGoal: OpenClaw Gateway running on an exe.dev VM, reachable from your laptop via: `https://<vm-name>.exe.xyz`This page assumes exe.dev’s default **exeuntu** image. If you picked a different distro, map packages accordingly.\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#beginner-quick-path)  Beginner quick path\n\n1. [https://exe.new/openclaw](https://exe.new/openclaw)\n2. Fill in your auth key/token as needed\n3. Click on “Agent” next to your VM and wait for Shelley to finish provisioning\n4. Open `https://<vm-name>.exe.xyz/` and authenticate with the configured shared secret (this guide uses token auth by default, but password auth works too if you switch `gateway.auth.mode`)\n5. Approve any pending device pairing requests with `openclaw devices approve <requestId>`\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#what-you-need)  What you need\n\n- exe.dev account\n- `ssh exe.dev` access to [exe.dev](https://exe.dev/) virtual machines (optional)\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#automated-install-with-shelley)  Automated install with Shelley\n\nShelley, [exe.dev](https://exe.dev/)’s agent, can install OpenClaw instantly with our\nprompt. The prompt used is as below:\n\n```\nSet up OpenClaw (https://docs.openclaw.ai/install) on this VM. Use the non-interactive and accept-risk flags for openclaw onboarding. Add the supplied auth or token as needed. Configure nginx to forward from the default port 18789 to the root location on the default enabled site config, making sure to enable Websocket support. Pairing is done by \"openclaw devices list\" and \"openclaw devices approve <request id>\". Make sure the dashboard shows that OpenClaw\'s health is OK. exe.dev handles forwarding from port 8000 to port 80/443 and HTTPS for us, so the final \"reachable\" should be <vm-name>.exe.xyz, without port specification.\n```\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#manual-installation)  Manual installation\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#1-create-the-vm)  1) Create the VM\n\nFrom your device:\n\n```\nssh exe.dev new\n```\n\nThen connect:\n\n```\nssh <vm-name>.exe.xyz\n```\n\nKeep this VM **stateful**. OpenClaw stores `openclaw.json`, per-agent `auth-profiles.json`, sessions, and channel/provider state under `~/.openclaw/`, plus the workspace under `~/.openclaw/workspace/`.\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#2-install-prerequisites-on-the-vm)  2) Install prerequisites (on the VM)\n\n```\nsudo apt-get update\nsudo apt-get install -y git curl jq ca-certificates openssl\n```\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#3-install-openclaw)  3) Install OpenClaw\n\nRun the OpenClaw install script:\n\n```\ncurl -fsSL https://openclaw.ai/install.sh | bash\n```\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#4-setup-nginx-to-proxy-openclaw-to-port-8000)  4) Setup nginx to proxy OpenClaw to port 8000\n\nEdit `/etc/nginx/sites-enabled/default` with\n\n```\nserver {\n    listen 80 default_server;\n    listen [::]:80 default_server;\n    listen 8000;\n    listen [::]:8000;\n\n    server_name _;\n\n    location / {\n        proxy_pass http://127.0.0.1:18789;\n        proxy_http_version 1.1;\n\n        # WebSocket support\n        proxy_set_header Upgrade $http_upgrade;\n        proxy_set_header Connection \"upgrade\";\n\n        # Standard proxy headers\n        proxy_set_header Host $host;\n        proxy_set_header X-Real-IP $remote_addr;\n        proxy_set_header X-Forwarded-For $remote_addr;\n        proxy_set_header X-Forwarded-Proto $scheme;\n\n        # Timeout settings for long-lived connections\n        proxy_read_timeout 86400s;\n        proxy_send_timeout 86400s;\n    }\n}\n```\n\nOverwrite forwarding headers instead of preserving client-supplied chains.\nOpenClaw trusts forwarded IP metadata only from explicitly configured proxies,\nand append-style `X-Forwarded-For` chains are treated as a hardening risk.\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#5-access-openclaw-and-grant-privileges)  5) Access OpenClaw and grant privileges\n\nAccess `https://<vm-name>.exe.xyz/` (see the Control UI output from onboarding). If it prompts for auth, paste the\nconfigured shared secret from the VM. This guide uses token auth, so retrieve `gateway.auth.token`\nwith `openclaw config get gateway.auth.token` (or generate one with `openclaw doctor --generate-gateway-token`).\nIf you changed the gateway to password auth, use `gateway.auth.password` / `OPENCLAW_GATEWAY_PASSWORD` instead.\nApprove devices with `openclaw devices list` and `openclaw devices approve <requestId>`. When in doubt, use Shelley from your browser!\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#remote-access)  Remote access\n\nRemote access is handled by [exe.dev](https://exe.dev/)’s authentication. By\ndefault, HTTP traffic from port 8000 is forwarded to `https://<vm-name>.exe.xyz`\nwith email auth.\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#updating)  Updating\n\n```\nnpm i -g openclaw@latest\nopenclaw doctor\nopenclaw gateway restart\nopenclaw health\n```\n\nGuide: [Updating](https://docs.openclaw.ai/install/updating)\n\n## [​](https://docs.openclaw.ai/install/exe-dev\\#related)  Related\n\n- [Remote gateway](https://docs.openclaw.ai/gateway/remote)\n- [Install overview](https://docs.openclaw.ai/install)\n\n[Docker VM runtime](https://docs.openclaw.ai/install/docker-vm-runtime) [Fly.io](https://docs.openclaw.ai/install/fly)\n\nCtrl+I

---

## Migration guide - OpenClaw
**Source:** https://docs.openclaw.ai/install/migrating

[Skip to main content](https://docs.openclaw.ai/install/migrating/#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Migrating

Migration guide

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Import from another agent system](https://docs.openclaw.ai/install/migrating/#import-from-another-agent-system)
- [Move OpenClaw to a new machine](https://docs.openclaw.ai/install/migrating/#move-openclaw-to-a-new-machine)
- [Migration steps](https://docs.openclaw.ai/install/migrating/#migration-steps)
- [Common pitfalls](https://docs.openclaw.ai/install/migrating/#common-pitfalls)
- [Verification checklist](https://docs.openclaw.ai/install/migrating/#verification-checklist)
- [Upgrade a plugin in place](https://docs.openclaw.ai/install/migrating/#upgrade-a-plugin-in-place)
- [Related](https://docs.openclaw.ai/install/migrating/#related)

OpenClaw supports three migration paths: importing from another agent system, moving an existing install to a new machine, and upgrading a plugin in place.

## [​](https://docs.openclaw.ai/install/migrating/\\#import-from-another-agent-system)  Import from another agent system

Use the bundled migration providers to bring instructions, MCP servers, skills, model config, and (opt-in) API keys into OpenClaw. Plans are previewed before any change, secrets are redacted in reports, and apply is backed by a verified backup.

[**Migrating from Claude** \\\\\n\\\\\nImport Claude Code and Claude Desktop state, including `CLAUDE.md`, MCP servers, skills, and project commands.](https://docs.openclaw.ai/install/migrating-claude)

[**Migrating from Hermes** \\\\\n\\\\\nImport Hermes config, providers, MCP servers, memory, skills, and supported `.env` keys.](https://docs.openclaw.ai/install/migrating-hermes)

The CLI entry point is [`openclaw migrate`](https://docs.openclaw.ai/cli/migrate). Onboarding can also offer migration when it detects a known source (`openclaw onboard --flow import`).

## [​](https://docs.openclaw.ai/install/migrating/\\#move-openclaw-to-a-new-machine)  Move OpenClaw to a new machine

Copy the **state directory** (`~/.openclaw/` by default) and your **workspace** to preserve:

- **Config** — `openclaw.json` and all gateway settings.
- **Auth** — per-agent `auth-profiles.json` (API keys plus OAuth), plus any channel or provider state under `credentials/`.
- **Sessions** — conversation history and agent state.
- **Channel state** — WhatsApp login, Telegram session, and similar.
- **Workspace files** — `MEMORY.md`, `USER.md`, skills, and prompts.

Run `openclaw status` on the old machine to confirm your state directory path. Custom profiles use `~/.openclaw-<profile>/` or a path set via `OPENCLAW_STATE_DIR`.

### [​](https://docs.openclaw.ai/install/migrating/\\#migration-steps)  Migration steps

1

[Navigate to header](https://docs.openclaw.ai/install/migrating/#)

Stop the gateway and back up

On the **old** machine, stop the gateway so files are not changing mid-copy, then archive:

```
openclaw gateway stop
cd ~
tar -czf openclaw-state.tgz .openclaw
```

If you use multiple profiles (for example `~/.openclaw-work`), archive each separately.

2

[Navigate to header](https://docs.openclaw.ai/install/migrating/#)

Install OpenClaw on the new machine

[Install](https://docs.openclaw.ai/install) the CLI (and Node if needed) on the new machine. It is fine if onboarding creates a fresh `~/.openclaw/`. You will overwrite it next.

3

[Navigate to header](https://docs.openclaw.ai/install/migrating/#)

Copy state directory and workspace

Transfer the archive via `scp`, `rsync -a`, or an external drive, then extract:

```
cd ~
tar -xzf openclaw-state.tgz
```

Ensure hidden directories were included and file ownership matches the user that will run the gateway.

4

[Navigate to header](https://docs.openclaw.ai/install/migrating/#)

Run doctor and verify

On the new machine, run [Doctor](https://docs.openclaw.ai/gateway/doctor) to apply config migrations and repair services:

```
openclaw doctor
openclaw gateway restart
openclaw status
```

### [​](https://docs.openclaw.ai/install/migrating/\\#common-pitfalls)  Common pitfalls

Profile or state-dir mismatch

If the old gateway used `--profile` or `OPENCLAW_STATE_DIR` and the new one does not, channels will appear logged out and sessions will be empty. Launch the gateway with the **same** profile or state-dir you migrated, then rerun `openclaw doctor`.

Copying only openclaw.json

The config file alone is not enough. Model auth profiles live under `agents/<agentId>/agent/auth-profiles.json`, and channel and provider state lives under `credentials/`. Always migrate the **entire** state directory.

Permissions and ownership

If you copied as root or switched users, the gateway may fail to read credentials. Ensure the state directory and workspace are owned by the user running the gateway.

Remote mode

If your UI points at a **remote** gateway, the remote host owns sessions and workspace. Migrate the gateway host itself, not your local laptop. See [FAQ](https://docs.openclaw.ai/help/faq#where-things-live-on-disk).

Secrets in backups

The state directory contains auth profiles, channel credentials, and other provider state. Store backups encrypted, avoid insecure transfer channels, and rotate keys if you suspect exposure.

### [​](https://docs.openclaw.ai/install/migrating/\\#verification-checklist)  Verification checklist

On the new machine, confirm:

- [ ] `openclaw status` shows the gateway running.
- [ ]  Channels are still connected (no re-pairing needed).
- [ ]  The dashboard opens and shows existing sessions.
- [ ]  Workspace files (memory, configs) are present.

## [​](https://docs.openclaw.ai/install/migrating/\\#upgrade-a-plugin-in-place)  Upgrade a plugin in place

In-place plugin upgrades preserve the same plugin id and config keys but may move on-disk state into the current layout. Plugin-specific upgrade guides live alongside their channels:

- [Matrix migration](https://docs.openclaw.ai/channels/matrix-migration): encrypted-state recovery limits, automatic snapshot behavior, and manual recovery commands.

## [​](https://docs.openclaw.ai/install/migrating/\\#related)  Related

- [`openclaw migrate`](https://docs.openclaw.ai/cli/migrate): CLI reference for cross-system imports.
- [Install overview](https://docs.openclaw.ai/install): all installation methods.
- [Doctor](https://docs.openclaw.ai/gateway/doctor): post-migration health check.
- [Uninstall](https://docs.openclaw.ai/install/uninstall): removing OpenClaw cleanly.

[Updating](https://docs.openclaw.ai/install/updating) [Migrating from Claude](https://docs.openclaw.ai/install/migrating-claude)

Ctrl+I

---

## Northflank - OpenClaw
**Source:** https://docs.openclaw.ai/install/northflank

[Skip to main content](https://docs.openclaw.ai/install/northflank#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

Northflank

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Northflank](https://docs.openclaw.ai/install/northflank#northflank)
- [How to get started](https://docs.openclaw.ai/install/northflank#how-to-get-started)
- [What you get](https://docs.openclaw.ai/install/northflank#what-you-get)
- [Connect a channel](https://docs.openclaw.ai/install/northflank#connect-a-channel)
- [Next steps](https://docs.openclaw.ai/install/northflank#next-steps)

# [​](https://docs.openclaw.ai/install/northflank\\#northflank)  Northflank

Deploy OpenClaw on Northflank with a one-click template and access it through the web Control UI.
This is the easiest “no terminal on the server” path: Northflank runs the Gateway for you.

## [​](https://docs.openclaw.ai/install/northflank\\#how-to-get-started)  How to get started

1. Click [Deploy OpenClaw](https://northflank.com/stacks/deploy-openclaw) to open the template.
2. Create an [account on Northflank](https://app.northflank.com/signup) if you don’t already have one.
3. Click **Deploy OpenClaw now**.
4. Set the required environment variable: `OPENCLAW_GATEWAY_TOKEN` (use a strong random value).
5. Click **Deploy stack** to build and run the OpenClaw template.
6. Wait for the deployment to complete, then click **View resources**.
7. Open the OpenClaw service.
8. Open the public OpenClaw URL at `/openclaw` and connect using the configured shared secret. This template uses `OPENCLAW_GATEWAY_TOKEN` by default; if you replace it with password auth, use that password instead.

## [​](https://docs.openclaw.ai/install/northflank\\#what-you-get)  What you get

- Hosted OpenClaw Gateway + Control UI
- Persistent storage via Northflank Volume (`/data`) so `openclaw.json`,
per-agent `auth-profiles.json`, channel/provider state, sessions, and
workspace survive redeploys

## [​](https://docs.openclaw.ai/install/northflank\\#connect-a-channel)  Connect a channel

Use the Control UI at `/openclaw` or run `openclaw onboard` via SSH for channel setup instructions:

- [Telegram](https://docs.openclaw.ai/channels/telegram) (fastest — just a bot token)
- [Discord](https://docs.openclaw.ai/channels/discord)
- [All channels](https://docs.openclaw.ai/channels)

## [​](https://docs.openclaw.ai/install/northflank\\#next-steps)  Next steps

- Set up messaging channels: [Channels](https://docs.openclaw.ai/channels)
- Configure the Gateway: [Gateway configuration](https://docs.openclaw.ai/gateway/configuration)
- Keep OpenClaw up to date: [Updating](https://docs.openclaw.ai/install/updating)

[macOS VMs](https://docs.openclaw.ai/install/macos-vm) [Oracle Cloud](https://docs.openclaw.ai/install/oracle)

Ctrl+I

---

## GCP - OpenClaw
**Source:** https://docs.openclaw.ai/install/gcp

[Skip to main content](https://docs.openclaw.ai/install/gcp#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

GCP

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [OpenClaw on GCP Compute Engine (Docker, Production VPS Guide)](https://docs.openclaw.ai/install/gcp#openclaw-on-gcp-compute-engine-docker-production-vps-guide)
- [Goal](https://docs.openclaw.ai/install/gcp#goal)
- [What are we doing (simple terms)?](https://docs.openclaw.ai/install/gcp#what-are-we-doing-simple-terms-)
- [Quick path (experienced operators)](https://docs.openclaw.ai/install/gcp#quick-path-experienced-operators)
- [What you need](https://docs.openclaw.ai/install/gcp#what-you-need)
- [Troubleshooting](https://docs.openclaw.ai/install/gcp#troubleshooting)
- [Service accounts (security best practice)](https://docs.openclaw.ai/install/gcp#service-accounts-security-best-practice)
- [Next steps](https://docs.openclaw.ai/install/gcp#next-steps)
- [Related](https://docs.openclaw.ai/install/gcp#related)

# [​](https://docs.openclaw.ai/install/gcp\\#openclaw-on-gcp-compute-engine-docker-production-vps-guide)  OpenClaw on GCP Compute Engine (Docker, Production VPS Guide)

## [​](https://docs.openclaw.ai/install/gcp\\#goal)  Goal

Run a persistent OpenClaw Gateway on a GCP Compute Engine VM using Docker, with durable state, baked-in binaries, and safe restart behavior.If you want “OpenClaw 24/7 for ~$5-12/mo”, this is a reliable setup on Google Cloud.
Pricing varies by machine type and region; pick the smallest VM that fits your workload and scale up if you hit OOMs.

## [​](https://docs.openclaw.ai/install/gcp\\#what-are-we-doing-simple-terms-)  What are we doing (simple terms)?

- Create a GCP project and enable billing
- Create a Compute Engine VM
- Install Docker (isolated app runtime)
- Start the OpenClaw Gateway in Docker
- Persist `~/.openclaw` \\+ `~/.openclaw/workspace` on the host (survives restarts/rebuilds)
- Access the Control UI from your laptop via an SSH tunnel

That mounted `~/.openclaw` state includes `openclaw.json`, per-agent\n`agents/<agentId>/agent/auth-profiles.json`, and `.env`.The Gateway can be accessed via:

- SSH port forwarding from your laptop
- Direct port exposure if you manage firewalling and tokens yourself

This guide uses Debian on GCP Compute Engine.
Ubuntu also works; map packages accordingly.
For the generic Docker flow, see [Docker](https://docs.openclaw.ai/install/docker).

* * *

## [​](https://docs.openclaw.ai/install/gcp\\#quick-path-experienced-operators)  Quick path (experienced operators)

1. Create GCP project + enable Compute Engine API
2. Create Compute Engine VM (e2-small, Debian 12, 20GB)
3. SSH into the VM
4. Install Docker
5. Clone OpenClaw repository
6. Create persistent host directories
7. Configure `.env` and `docker-compose.yml`
8. Bake required binaries, build, and launch

* * *

## [​](https://docs.openclaw.ai/install/gcp\\#what-you-need)  What you need

- GCP account (free tier eligible for e2-micro)
- gcloud CLI installed (or use Cloud Console)
- SSH access from your laptop
- Basic comfort with SSH + copy/paste
- ~20-30 minutes
- Docker and Docker Compose
- Model auth credentials
- Optional provider credentials
  - WhatsApp QR
  - Telegram bot token
  - Gmail OAuth

* * *

1

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Install gcloud CLI (or use Console)

**Option A: gcloud CLI** (recommended for automation)Install from [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install)Initialize and authenticate:

```
gcloud init
gcloud auth login
```

**Option B: Cloud Console**All steps can be done via the web UI at [https://console.cloud.google.com](https://console.cloud.google.com/)

2

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Create a GCP project

**CLI:**

```
gcloud projects create my-openclaw-project --name=\"OpenClaw Gateway\"\ngcloud config set project my-openclaw-project
```

Enable billing at [https://console.cloud.google.com/billing](https://console.cloud.google.com/billing) (required for Compute Engine).Enable the Compute Engine API:

```
gcloud services enable compute.googleapis.com
```

**Console:**

1. Go to IAM & Admin > Create Project
2. Name it and create
3. Enable billing for the project
4. Navigate to APIs & Services > Enable APIs > search “Compute Engine API” > Enable

3

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Create the VM

**Machine types:**

| Type | Specs | Cost | Notes |
| --- | --- | --- | --- |
| e2-medium | 2 vCPU, 4GB RAM | ~$25/mo | Most reliable for local Docker builds |
| e2-small | 2 vCPU, 2GB RAM | ~$12/mo | Minimum recommended for Docker build |
| e2-micro | 2 vCPU (shared), 1GB RAM | Free tier eligible | Often fails with Docker build OOM (exit 137) |

**CLI:**

```
gcloud compute instances create openclaw-gateway \\\
  --zone=us-central1-a \\\
  --machine-type=e2-small \\\
  --boot-disk-size=20GB \\\
  --image-family=debian-12 \\\
  --image-project=debian-cloud
```

**Console:**

1. Go to Compute Engine > VM instances > Create instance
2. Name: `openclaw-gateway`
3. Region: `us-central1`, Zone: `us-central1-a`
4. Machine type: `e2-small`
5. Boot disk: Debian 12, 20GB
6. Create

4

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

SSH into the VM

**CLI:**

```
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console:**Click the “SSH” button next to your VM in the Compute Engine dashboard.Note: SSH key propagation can take 1-2 minutes after VM creation. If connection is refused, wait and retry.

5

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Install Docker (on the VM)

```
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Log out and back in for the group change to take effect:

```
exit
```

Then SSH back in:

```
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

Verify:

```
docker --version
docker compose version
```

6

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Clone the OpenClaw repository

```
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

This guide assumes you will build a custom image to guarantee binary persistence.

7

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Create persistent host directories

Docker containers are ephemeral.
All long-lived state must live on the host.

```
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

8

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Configure environment variables

Create `.env` in the repository root.

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=
XDG_CONFIG_HOME=/home/node/.openclaw
```

Leave `OPENCLAW_GATEWAY_TOKEN` blank unless you explicitly want to\nmanage it through `.env`; OpenClaw writes a random gateway token to\nconfig on first start. Generate a keyring password and paste it into\n`GOG_KEYRING_PASSWORD`:\n
```
openssl rand -hex 32
```

**Do not commit this file.**This `.env` file is for container/runtime env such as `OPENCLAW_GATEWAY_TOKEN`.\nStored provider OAuth/API-key auth lives in the mounted\n`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`.

9

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Docker Compose configuration

Create or update `docker-compose.yml`.

```
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recommended: keep the Gateway loopback-only on the VM; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - \"127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789\"
    command:
      [\\\
        \"node\\\",\\\
        \"dist/index.js\\\",\\\
        \"gateway\\\",\\\
        \"--bind\\\",\\\
        \"${OPENCLAW_GATEWAY_BIND}\\\",\\\
        \"--port\\\",\\\
        \"${OPENCLAW_GATEWAY_PORT}\\\",\\\
        \"--allow-unconfigured\\\",\\\
      ]
```

`--allow-unconfigured` is only for bootstrap convenience, it is not a replacement for a proper gateway configuration. Still set auth (`gateway.auth.token` or password) and use safe bind settings for your deployment.

10

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Shared Docker VM runtime steps

Use the shared runtime guide for the common Docker host flow:

- [Bake required binaries into the image](https://docs.openclaw.ai/install/docker-vm-runtime#bake-required-binaries-into-the-image)
- [Build and launch](https://docs.openclaw.ai/install/docker-vm-runtime#build-and-launch)
- [What persists where](https://docs.openclaw.ai/install/docker-vm-runtime#what-persists-where)
- [Updates](https://docs.openclaw.ai/install/docker-vm-runtime#updates)

11

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

GCP-specific launch notes

On GCP, if build fails with `Killed` or `exit code 137` during `pnpm install --frozen-lockfile`, the VM is out of memory. Use `e2-small` minimum, or `e2-medium` for more reliable first builds.When binding to LAN (`OPENCLAW_GATEWAY_BIND=lan`), configure a trusted browser origin before continuing:\n
```
docker compose run --rm openclaw-cli config set gateway.controlUi.allowedOrigins '[\"http://127.0.0.1:18789\"]' --strict-json
```

If you changed the gateway port, replace `18789` with your configured port.

12

[Navigate to header](https://docs.openclaw.ai/install/gcp#)

Access from your laptop

Create an SSH tunnel to forward the Gateway port:\n
```
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

Open in your browser:`http://127.0.0.1:18789/`Reprint a clean dashboard link:\n
```
docker compose run --rm openclaw-cli dashboard --no-open
```

If the UI prompts for shared-secret auth, paste the configured token or\npassword into Control UI settings. This Docker flow writes a token by\ndefault; if you switch the container config to password auth, use that\npassword instead.If Control UI shows `unauthorized` or `disconnected (1008): pairing required`, approve the browser device:\n
```
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

Need the shared persistence and update reference again?\nSee [Docker VM Runtime](https://docs.openclaw.ai/install/docker-vm-runtime#what-persists-where) and [Docker VM Runtime updates](https://docs.openclaw.ai/install/docker-vm-runtime#updates).

* * *

## [​](https://docs.openclaw.ai/install/gcp\\#troubleshooting)  Troubleshooting

**SSH connection refused**SSH key propagation can take 1-2 minutes after VM creation. Wait and retry.**OS Login issues**Check your OS Login profile:\n
```
gcloud compute os-login describe-profile
```

Ensure your account has the required IAM permissions (Compute OS Login or Compute OS Admin Login).**Out of memory (OOM)**If Docker build fails with `Killed` and `exit code 137`, the VM was OOM-killed. Upgrade to e2-small (minimum) or e2-medium (recommended for reliable local builds):\n
```
# Stop the VM first
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# Change machine type
gcloud compute instances set-machine-type openclaw-gateway \\\
  --zone=us-central1-a \\\
  --machine-type=e2-small

# Start the VM
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

* * *

## [​](https://docs.openclaw.ai/install/gcp\\#service-accounts-security-best-practice)  Service accounts (security best practice)

For personal use, your default user account works fine.For automation or CI/CD pipelines, create a dedicated service account with minimal permissions:\n
1. Create a service account:\n













```
gcloud iam service-accounts create openclaw-deploy \\\
     --display-name=\"OpenClaw Deployment\"
```

2. Grant Compute Instance Admin role (or narrower custom role):\n













```
gcloud projects add-iam-policy-binding my-openclaw-project \\\
     --member=\"serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com\" \\\
     --role=\"roles/compute.instanceAdmin.v1\"
```

\nAvoid using the Owner role for automation. Use the principle of least privilege.See [https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles) for IAM role details.\n
* * *

## [​](https://docs.openclaw.ai/install/gcp\\#next-steps)  Next steps

- Set up messaging channels: [Channels](https://docs.openclaw.ai/channels)
- Pair local devices as nodes: [Nodes](https://docs.openclaw.ai/nodes)
- Configure the Gateway: [Gateway configuration](https://docs.openclaw.ai/gateway/configuration)

## [​](https://docs.openclaw.ai/install/gcp\\#related)  Related

- [Install overview](https://docs.openclaw.ai/install)
- [Azure](https://docs.openclaw.ai/install/azure)
- [VPS hosting](https://docs.openclaw.ai/vps)

[Fly.io](https://docs.openclaw.ai/install/fly) [Hetzner](https://docs.openclaw.ai/install/hetzner)

Ctrl+I

---

## Fly.io - OpenClaw
**Source:** https://docs.openclaw.ai/install/fly

[Skip to main content](https://docs.openclaw.ai/install/fly#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

Fly.io

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Fly.io Deployment](https://docs.openclaw.ai/install/fly#fly-io-deployment)
- [What you need](https://docs.openclaw.ai/install/fly#what-you-need)
- [Beginner quick path](https://docs.openclaw.ai/install/fly#beginner-quick-path)
- [Control UI](https://docs.openclaw.ai/install/fly#control-ui)
- [Logs](https://docs.openclaw.ai/install/fly#logs)
- [SSH Console](https://docs.openclaw.ai/install/fly#ssh-console)
- [Troubleshooting](https://docs.openclaw.ai/install/fly#troubleshooting)
- [”App is not listening on expected address”](https://docs.openclaw.ai/install/fly#%E2%80%9Dapp-is-not-listening-on-expected-address%E2%80%9D)
- [Health checks failing / connection refused](https://docs.openclaw.ai/install/fly#health-checks-failing-%2F-connection-refused)
- [OOM / Memory Issues](https://docs.openclaw.ai/install/fly#oom-%2F-memory-issues)
- [Gateway lock issues](https://docs.openclaw.ai/install/fly#gateway-lock-issues)
- [Config not being read](https://docs.openclaw.ai/install/fly#config-not-being-read)
- [Writing config via SSH](https://docs.openclaw.ai/install/fly#writing-config-via-ssh)
- [State not persisting](https://docs.openclaw.ai/install/fly#state-not-persisting)
- [Updates](https://docs.openclaw.ai/install/fly#updates)
- [Updating machine command](https://docs.openclaw.ai/install/fly#updating-machine-command)
- [Private deployment (hardened)](https://docs.openclaw.ai/install/fly#private-deployment-hardened)
- [When to use private deployment](https://docs.openclaw.ai/install/fly#when-to-use-private-deployment)
- [Setup](https://docs.openclaw.ai/install/fly#setup)
- [Accessing a private deployment](https://docs.openclaw.ai/install/fly#accessing-a-private-deployment)
- [Webhooks with private deployment](https://docs.openclaw.ai/install/fly#webhooks-with-private-deployment)
- [Security benefits](https://docs.openclaw.ai/install/fly#security-benefits)
- [Notes](https://docs.openclaw.ai/install/fly#notes)
- [Cost](https://docs.openclaw.ai/install/fly#cost)
- [Next steps](https://docs.openclaw.ai/install/fly#next-steps)
- [Related](https://docs.openclaw.ai/install/fly#related)

# [​](https://docs.openclaw.ai/install/fly\\#fly-io-deployment)  Fly.io Deployment

**Goal:** OpenClaw Gateway running on a [Fly.io](https://fly.io/) machine with persistent storage, automatic HTTPS, and Discord/channel access.

## [​](https://docs.openclaw.ai/install/fly\\#what-you-need)  What you need

- [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/) installed
- Fly.io account (free tier works)
- Model auth: API key for your chosen model provider
- Channel credentials: Discord bot token, Telegram token, etc.

## [​](https://docs.openclaw.ai/install/fly\\#beginner-quick-path)  Beginner quick path

1. Clone repo → customize `fly.toml`
2. Create app + volume → set secrets
3. Deploy with `fly deploy`
4. SSH in to create config or use Control UI

1

[Navigate to header](https://docs.openclaw.ai/install/fly#)

Create the Fly app

```
# Clone the repo
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Create a new Fly app (pick your own name)
fly apps create my-openclaw

# Create a persistent volume (1GB is usually enough)
fly volumes create openclaw_data --size 1 --region iad
```

**Tip:** Choose a region close to you. Common options: `lhr` (London), `iad` (Virginia), `sjc` (San Jose).

2

[Navigate to header](https://docs.openclaw.ai/install/fly#)

Configure fly.toml

Edit `fly.toml` to match your app name and requirements.**Security note:** The default config exposes a public URL. For a hardened deployment with no public IP, see [Private Deployment](https://docs.openclaw.ai/install/fly#private-deployment-hardened) or use `fly.private.toml`.

```
app = \"my-openclaw\"  # Your app name
primary_region = \"iad\"

[build]
  dockerfile = \"Dockerfile\"

[env]
  NODE_ENV = \"production\"
  OPENCLAW_PREFER_PNPM = \"1\"
  OPENCLAW_STATE_DIR = \"/data\"
  NODE_OPTIONS = \"--max-old-space-size=1536\"

[processes]
  app = \"node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan\"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = [\"app\"]

[[vm]]
  size = \"shared-cpu-2x\"
  memory = \"2048mb\"

[mounts]
  source = \"openclaw_data\"
  destination = \"/data\"
```

**Key settings:**

| Setting | Why |
| --- | --- |
| `--bind lan` | Binds to `0.0.0.0` so Fly’s proxy can reach the gateway |
| `--allow-unconfigured` | Starts without a config file (you’ll create one after) |
| `internal_port = 3000` | Must match `--port 3000` (or `OPENCLAW_GATEWAY_PORT`) for Fly health checks |
| `memory = \"2048mb\"` | 512MB is too small; 2GB recommended |
| `OPENCLAW_STATE_DIR = \"/data\"` | Persists state on the volume |

3

[Navigate to header](https://docs.openclaw.ai/install/fly#)

Set secrets

```
# Required: Gateway token (for non-loopback binding)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Model provider API keys
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Optional: Other providers
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Channel tokens
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**Notes:**

- Non-loopback binds (`--bind lan`) require a valid gateway auth path. This Fly.io example uses `OPENCLAW_GATEWAY_TOKEN`, but `gateway.auth.password` or a correctly configured non-loopback `trusted-proxy` deployment also satisfy the requirement.
- Treat these tokens like passwords.
- **Prefer env vars over config file** for all API keys and tokens. This keeps secrets out of `openclaw.json` where they could be accidentally exposed or logged.

4

[Navigate to header](https://docs.openclaw.ai/install/fly#)

Deploy

```
fly deploy
```

First deploy builds the Docker image (~2-3 minutes). Subsequent deploys are faster.After deployment, verify:

```
fly status
fly logs
```

You should see:

```
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```

5

[Navigate to header](https://docs.openclaw.ai/install/fly#)

Create config file

SSH into the machine to create a proper config:

```
fly ssh console
```

Create the config directory and file:

```
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  \"agents\": {
    \"defaults\": {
      \"model\": {
        \"primary\": \"anthropic/claude-opus-4-6\",
        \"fallbacks\": [\"anthropic/claude-sonnet-4-6\", \"openai/gpt-5.4\"]
      },
      \"maxConcurrent\": 4
    },
    \"list\": [\\\
      {\\\
        \"id\": \"main\\\",\\\
        \"default\": true\\\
      }\\\
    ]
  },
  \"auth\": {
    \"profiles\": {
      \"anthropic:default\": { \"mode\": \"token\", \"provider\": \"anthropic\" },
      \"openai:default\": { \"mode\": \"token\", \"provider\": \"openai\" }
    }
  },
  \"bindings\": [\\\
    {\\\
      \"agentId\": \"main\\\",\\\
      \"match\": { \"channel\": \"discord\" }\\\
    }\\\
  ],
  \"channels\": {
    \"discord\": {
      \"enabled\": true,
      \"groupPolicy\": \"allowlist\",
      \"guilds\": {
        \"YOUR_GUILD_ID\": {
          \"channels\": { \"general\": { \"allow\": true } },
          \"requireMention\": false
        }
      }
    }
  },
  \"gateway\": {
    \"mode\": \"local\",
    \"bind\": \"auto\",
    \"controlUi\": {
      \"allowedOrigins\": [\\\
        \"https://my-openclaw.fly.dev\\\",\\\
        \"http://localhost:3000\\\",\\\
        \"http://127.0.0.1:3000\\\"
      ]
    }
  },
  \"meta\": {}
}
EOF
```

**Note:** With `OPENCLAW_STATE_DIR=/data`, the config path is `/data/openclaw.json`.**Note:** Replace `https://my-openclaw.fly.dev` with your real Fly app
origin. Gateway startup seeds local Control UI origins from the runtime
`--bind` and `--port` values so first boot can proceed before config exists,
but browser access through Fly still needs the exact HTTPS origin listed in
`gateway.controlUi.allowedOrigins`.**Note:** The Discord token can come from either:

- Environment variable: `DISCORD_BOT_TOKEN` (recommended for secrets)
- Config file: `channels.discord.token`

If using env var, no need to add token to config. The gateway reads `DISCORD_BOT_TOKEN` automatically.Restart to apply:

```
exit
fly machine restart <machine-id>
```

6

[Navigate to header](https://docs.openclaw.ai/install/fly#)

Access the Gateway

### [​](https://docs.openclaw.ai/install/fly\\#control-ui)  Control UI

Open in browser:

```
fly open
```

Or visit `https://my-openclaw.fly.dev/`Authenticate with the configured shared secret. This guide uses the gateway
token from `OPENCLAW_GATEWAY_TOKEN`; if you switched to password auth, use
that password instead.

### [​](https://docs.openclaw.ai/install/fly\\#logs)  Logs

```
fly logs              # Live logs
fly logs --no-tail    # Recent logs
```

### [​](https://docs.openclaw.ai/install/fly\\#ssh-console)  SSH Console

```
fly ssh console
```

## [​](https://docs.openclaw.ai/install/fly\\#troubleshooting)  Troubleshooting

### [​](https://docs.openclaw.ai/install/fly\\#%E2%80%9Dapp-is-not-listening-on-expected-address%E2%80%9D)  ”App is not listening on expected address”

The gateway is binding to `127.0.0.1` instead of `0.0.0.0`.**Fix:** Add `--bind lan` to your process command in `fly.toml`.

### [​](https://docs.openclaw.ai/install/fly\\#health-checks-failing-/-connection-refused)  Health checks failing / connection refused

Fly can’t reach the gateway on the configured port.**Fix:** Ensure `internal_port` matches the gateway port (set `--port 3000` or `OPENCLAW_GATEWAY_PORT=3000`).

### [​](https://docs.openclaw.ai/install/fly\\#oom-/-memory-issues)  OOM / Memory Issues

Container keeps restarting or getting killed. Signs: `SIGABRT`, `v8::internal::Runtime_AllocateInYoungGeneration`, or silent restarts.**Fix:** Increase memory in `fly.toml`:

```
[[vm]]
  memory = \"2048mb\"
```

Or update an existing machine:

```
fly machine update <machine-id> --vm-memory 2048 -y
```

**Note:** 512MB is too small. 1GB may work but can OOM under load or with verbose logging. **2GB is recommended.**

### [​](https://docs.openclaw.ai/install/fly\\#gateway-lock-issues)  Gateway lock issues

Gateway refuses to start with “already running” errors.This happens when the container restarts but the PID lock file persists on the volume.**Fix:** Delete the lock file:

```
fly ssh console --command \"rm -f /data/gateway.*.lock\"
fly machine restart <machine-id>
```

The lock file is at `/data/gateway.*.lock` (not in a subdirectory).

### [​](https://docs.openclaw.ai/install/fly\\#config-not-being-read)  Config not being read

`--allow-unconfigured` only bypasses the startup guard. It does not create or repair `/data/openclaw.json`, so make sure your real config exists and includes `gateway.mode=\"local\"` when you want a normal local gateway start.Verify the config exists:

```
fly ssh console --command \"cat /data/openclaw.json\"
```

### [​](https://docs.openclaw.ai/install/fly\\#writing-config-via-ssh)  Writing config via SSH

The `fly ssh console -C` command doesn’t support shell redirection. To write a config file:

```
# Use echo + tee (pipe from local to remote)
echo \'{\"your\":\"config\"}\' | fly ssh console -C \"tee /data/openclaw.json\"

# Or use sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**Note:**`fly sftp` may fail if the file already exists. Delete first:

```
fly ssh console --command \"rm /data/openclaw.json\"
```

### [​](https://docs.openclaw.ai/install/fly\\#state-not-persisting)  State not persisting

If you lose auth profiles, channel/provider state, or sessions after a restart,
the state dir is writing to the container filesystem.**Fix:** Ensure `OPENCLAW_STATE_DIR=/data` is set in `fly.toml` and redeploy.

## [​](https://docs.openclaw.ai/install/fly\\#updates)  Updates

```
# Pull latest changes
git pull

# Redeploy
fly deploy

# Check health
fly status
fly logs
```

### [​](https://docs.openclaw.ai/install/fly\\#updating-machine-command)  Updating machine command

If you need to change the startup command without a full redeploy:

```
# Get machine ID
fly machines list

# Update command
fly machine update <machine-id> --command \"node dist/index.js gateway --port 3000 --bind lan\" -y

# Or with memory increase
fly machine update <machine-id> --vm-memory 2048 --command \"node dist/index.js gateway --port 3000 --bind lan\" -y
```

**Note:** After `fly deploy`, the machine command may reset to what’s in `fly.toml`. If you made manual changes, re-apply them after deploy.

## [​](https://docs.openclaw.ai/install/fly\\#private-deployment-hardened)  Private deployment (hardened)

By default, Fly allocates public IPs, making your gateway accessible at `https://your-app.fly.dev`. This is convenient but means your deployment is discoverable by internet scanners (Shodan, Censys, etc.).For a hardened deployment with **no public exposure**, use the private template.

### [​](https://docs.openclaw.ai/install/fly\\#when-to-use-private-deployment)  When to use private deployment

- You only make **outbound** calls/messages (no inbound webhooks)
- You use **ngrok or Tailscale** tunnels for any webhook callbacks
- You access the gateway via **SSH, proxy, or WireGuard** instead of browser
- You want the deployment **hidden from internet scanners**

### [​](https://openclaw.ai/install/fly#setup)  Setup

Use `fly.private.toml` instead of the standard config:

```
# Deploy with private config
fly deploy -c fly.private.toml
```

Or convert an existing deployment:

```
# List current IPs
fly ips list -a my-openclaw

# Release public IPs
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# Switch to private config so future deploys don\'t re-allocate public IPs
# (remove [http_service] or deploy with the private template)
fly deploy -c fly.private.toml

# Allocate private-only IPv6
fly ips allocate-v6 --private -a my-openclaw
```

After this, `fly ips list` should show only a `private` type IP:

```
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```

### [​](https://docs.openclaw.ai/install/fly\\#accessing-a-private-deployment)  Accessing a private deployment

Since there’s no public URL, use one of these methods:**Option 1: Local proxy (simplest)**

```
# Forward local port 3000 to the app
fly proxy 3000:3000 -a my-openclaw

# Then open http://localhost:3000 in browser
```

**Option 2: WireGuard VPN**

```
# Create WireGuard config (one-time)
fly wireguard create

# Import to WireGuard client, then access via internal IPv6
# Example: http://[fdaa:x:x:x:x::x]:3000
```

**Option 3: SSH only**

```
fly ssh console -a my-openclaw
```

### [​](https://docs.openclaw.ai/install/fly\\#webhooks-with-private-deployment)  Webhooks with private deployment

If you need webhook callbacks (Twilio, Telnyx, etc.) without public exposure:

1. **ngrok tunnel** \\- Run ngrok inside the container or as a sidecar
2. **Tailscale Funnel** \\- Expose specific paths via Tailscale
3. **Outbound-only** \\- Some providers (Twilio) work fine for outbound calls without webhooks

Example voice-call config with ngrok:

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        enabled: true,
        config: {
          provider: \"twilio\",
          tunnel: { provider: \"ngrok\" },
          webhookSecurity: {
            allowedHosts: [\"example.ngrok.app\"],
          },
        },
      },
    },
  },
}
```

The ngrok tunnel runs inside the container and provides a public webhook URL without exposing the Fly app itself. Set `webhookSecurity.allowedHosts` to the public tunnel hostname so forwarded host headers are accepted.

### [​](https://docs.openclaw.ai/install/fly\\#security-benefits)  Security benefits

| Aspect | Public | Private |
| --- | --- | --- |
| Internet scanners | Discoverable | Hidden |
| Direct attacks | Possible | Blocked |
| Control UI access | Browser | Proxy/VPN |
| Webhook delivery | Direct | Via tunnel |

## [​](https://docs.openclaw.ai/install/fly\\#notes)  Notes

- Fly.io uses **x86 architecture** (not ARM)
- The Dockerfile is compatible with both architectures
- For WhatsApp/Telegram onboarding, use `fly ssh console`
- Persistent data lives on the volume at `/data`
- Signal requires Java + signal-cli; use a custom image and keep memory at 2GB+.

## [​](https://docs.openclaw.ai/install/fly\\#cost)  Cost

With the recommended config (`shared-cpu-2x`, 2GB RAM):

- ~$10-15/month depending on usage
- Free tier includes some allowance

See [Fly.io pricing](https://fly.io/docs/about/pricing/) for details.

## [​](https://docs.openclaw.ai/install/fly\\#next-steps)  Next steps

- Set up messaging channels: [Channels](https://docs.openclaw.ai/channels)
- Configure the Gateway: [Gateway configuration](https://docs.openclaw.ai/gateway/configuration)
- Keep OpenClaw up to date: [Updating](https://docs.openclaw.ai/install/updating)

## [​](https://docs.openclaw.ai/install/fly\\#related)  Related

- [Install overview](https://docs.openclaw.ai/install)
- [Hetzner](https://docs.openclaw.ai/install/hetzner)
- [Docker](https://docs.openclaw.ai/install/docker)
- [VPS hosting](https://docs.openclaw.ai/vps)

[exe.dev](https://docs.openclaw.ai/install/exe-dev) [GCP](https://docs.openclaw.ai/install/gcp)

Ctrl+I

---

## Installer internals - OpenClaw
**Source:** https://docs.openclaw.ai/install/installer

[Skip to main content](https://docs.openclaw.ai/install/installer#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Install overview

Installer internals

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick commands](https://docs.openclaw.ai/install/installer#quick-commands)
- [install.sh](https://docs.openclaw.ai/install/installer#install-sh)
- [Flow (install.sh)](https://docs.openclaw.ai/install/installer#flow-install-sh)
- [Source checkout detection](https://docs.openclaw.ai/install/installer#source-checkout-detection)
- [Examples (install.sh)](https://docs.openclaw.ai/install/installer#examples-install-sh)
- [install-cli.sh](https://docs.openclaw.ai/install/installer#install-cli-sh)
- [Flow (install-cli.sh)](https://docs.openclaw.ai/install/installer#flow-install-cli-sh)
- [Examples (install-cli.sh)](https://docs.openclaw.ai/install/installer#examples-install-cli-sh)
- [install.ps1](https://docs.openclaw.ai/install/installer#install-ps1)
- [Flow (install.ps1)](https://docs.openclaw.ai/install/installer#flow-install-ps1)
- [Examples (install.ps1)](https://docs.openclaw.ai/install/installer#examples-install-ps1)
- [CI and automation](https://docs.openclaw.ai/install/installer#ci-and-automation)
- [Troubleshooting](https://docs.openclaw.ai/install/installer#troubleshooting)
- [Related](https://docs.openclaw.ai/install/installer#related)

OpenClaw ships three installer scripts, served from `openclaw.ai`.

| Script | Platform | What it does |
| --- | --- | --- |
| [`install.sh`](https://docs.openclaw.ai/install/installer#installsh) | macOS / Linux / WSL | Installs Node if needed, installs OpenClaw via npm (default) or git, and can run onboarding. |
| [`install-cli.sh`](https://docs.openclaw.ai/install/installer#install-clish) | macOS / Linux / WSL | Installs Node + OpenClaw into a local prefix (`~/.openclaw`) with npm or git checkout modes. No root required. |
| [`install.ps1`](https://docs.openclaw.ai/install/installer#installps1) | Windows (PowerShell) | Installs Node if needed, installs OpenClaw via npm (default) or git, and can run onboarding. |

## [​](https://docs.openclaw.ai/install/installer\\#quick-commands)  Quick commands

- install.sh

- install-cli.sh

- install.ps1


```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --help
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --help
```

```
iwr -useb https://openclaw.ai/install.ps1 | iex
```

```
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -Tag beta -NoOnboard -DryRun
```

If install succeeds but `openclaw` is not found in a new terminal, see [Node.js troubleshooting](https://docs.openclaw.ai/install/node#troubleshooting).

* * *

## [​](https://docs.openclaw.ai/install/installer\\#install-sh)  install.sh

Recommended for most interactive installs on macOS/Linux/WSL.

### [​](https://docs.openclaw.ai/install/installer\\#flow-install-sh)  Flow (install.sh)

1

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Detect OS

Supports macOS and Linux (including WSL). If macOS is detected, installs Homebrew if missing.

2

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Ensure Node.js 24 by default

Checks Node version and installs Node 24 if needed (Homebrew on macOS, NodeSource setup scripts on Linux apt/dnf/yum). OpenClaw still supports Node 22 LTS, currently `22.14+`, for compatibility.

3

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Ensure Git

Installs Git if missing.

4

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Install OpenClaw

- `npm` method (default): global npm install
- `git` method: clone/update repo, install deps with pnpm, build, then install wrapper at `~/.local/bin/openclaw`

5

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Post-install tasks

- Refreshes a loaded gateway service best-effort (`openclaw gateway install --force`, then restart)
- Runs `openclaw doctor --non-interactive` on upgrades and git installs (best effort)
- Attempts onboarding when appropriate (TTY available, onboarding not disabled, and bootstrap/config checks pass)
- Defaults `SHARP_IGNORE_GLOBAL_LIBVIPS=1`

### [​](https://docs.openclaw.ai/install/installer\\#source-checkout-detection)  Source checkout detection

If run inside an OpenClaw checkout (`package.json` \\+ `pnpm-workspace.yaml`), the script offers:

- use checkout (`git`), or
- use global install (`npm`)

If no TTY is available and no install method is set, it defaults to `npm` and warns.The script exits with code `2` for invalid method selection or invalid `--install-method` values.

### [​](https://docs.openclaw.ai/install/installer\\#examples-install-sh)  Examples (install.sh)

- Default

- Skip onboarding

- Git install

- GitHub main via npm

- Dry run


```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --no-onboard
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --install-method git
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --version main
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --dry-run
```

Flags reference

| Flag | Description |
| --- | --- |
| `--install-method npm|git` | Choose install method (default: `npm`). Alias: `--method` |
| `--npm` | Shortcut for npm method |
| `--git` | Shortcut for git method. Alias: `--github` |
| `--version <version|dist-tag|spec>` | npm version, dist-tag, or package spec (default: `latest`) |
| `--beta` | Use beta dist-tag if available, else fallback to `latest` |
| `--git-dir <path>` | Checkout directory (default: `~/openclaw`). Alias: `--dir` |
| `--no-git-update` | Skip `git pull` for existing checkout |
| `--no-prompt` | Disable prompts |
| `--no-onboard` | Skip onboarding |
| `--onboard` | Enable onboarding |
| `--dry-run` | Print actions without applying changes |
| `--verbose` | Enable debug output (`set -x`, npm notice-level logs) |
| `--help` | Show usage (`-h`) |

Environment variables reference

| Variable | Description |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git|npm` | Install method |
| `OPENCLAW_VERSION=latest|next|main|<semver>|<spec>` | npm version, dist-tag, or package spec |
| `OPENCLAW_BETA=0|1` | Use beta if available |
| `OPENCLAW_GIT_DIR=<path>` | Checkout directory |
| `OPENCLAW_GIT_UPDATE=0|1` | Toggle git updates |
| `OPENCLAW_NO_PROMPT=1` | Disable prompts |
| `OPENCLAW_NO_ONBOARD=1` | Skip onboarding |
| `OPENCLAW_DRY_RUN=1` | Dry run mode |
| `OPENCLAW_VERBOSE=1` | Debug mode |
| `OPENCLAW_NPM_LOGLEVEL=error|warn|notice` | npm log level |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0|1` | Control sharp/libvips behavior (default: `1`) |

* * *

## [​](https://docs.openclaw.ai/install/installer\\#install-cli-sh)  install-cli.sh

Designed for environments where you want everything under a local prefix
(default `~/.openclaw`) and no system Node dependency. Supports npm installs
by default, plus git-checkout installs under the same prefix flow.

### [​](https://docs.openclaw.ai/install/installer\\#flow-install-cli-sh)  Flow (install-cli.sh)

1

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Install local Node runtime

Downloads a pinned supported Node LTS tarball (the version is embedded in the script and updated independently) to `<prefix>/tools/node-v<version>` and verifies SHA-256.

2

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Ensure Git

If Git is missing, attempts install via apt/dnf/yum on Linux or Homebrew on macOS.

3

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Install OpenClaw under prefix

- `npm` method (default): installs under the prefix with npm, then writes wrapper to `<prefix>/bin/openclaw`
- `git` method: clones/updates a checkout (default `~/openclaw`) and still writes the wrapper to `<prefix>/bin/openclaw`

4

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Refresh loaded gateway service

If a gateway service is already loaded from that same prefix, the script runs
`openclaw gateway install --force`, then `openclaw gateway restart`, and
probes gateway health best-effort.

### [​](https://docs.openclaw.ai/install/installer\\#examples-install-cli-sh)  Examples (install-cli.sh)

- Default

- Custom prefix + version

- Git install

- Automation JSON output

- Run onboarding


```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --prefix /opt/openclaw --version latest
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --install-method git --git-dir ~/openclaw
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --json --prefix /opt/openclaw
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --onboard
```

Flags reference

| Flag | Description |
| --- | --- |
| `--prefix <path>` | Install prefix (default: `~/.openclaw`) |
| `--install-method npm|git` | Choose install method (default: `npm`). Alias: `--method` |
| `--npm` | Shortcut for npm method |
| `--git`, `--github` | Shortcut for git method |
| `--git-dir <path>` | Git checkout directory (default: `~/openclaw`). Alias: `--dir` |
| `--version <ver>` | OpenClaw version or dist-tag (default: `latest`) |
| `--node-version <ver>` | Node version (default: `22.22.0`) |
| `--json` | Emit NDJSON events |
| `--onboard` | Run `openclaw onboard` after install |
| `--no-onboard` | Skip onboarding (default) |
| `--set-npm-prefix` | On Linux, force npm prefix to `~/.npm-global` if current prefix is not writable |
| `--help` | Show usage (`-h`) |

Environment variables reference

| Variable | Description |
| --- | --- |
| `OPENCLAW_PREFIX=<path>` | Install prefix |
| `OPENCLAW_INSTALL_METHOD=git|npm` | Install method |
| `OPENCLAW_VERSION=<ver>` | OpenClaw version or dist-tag |
| `OPENCLAW_NODE_VERSION=<ver>` | Node version |
| `OPENCLAW_GIT_DIR=<path>` | Git checkout directory for git installs |
| `OPENCLAW_GIT_UPDATE=0|1` | Toggle git updates for existing checkouts |
| `OPENCLAW_NO_ONBOARD=1` | Skip onboarding |
| `OPENCLAW_NPM_LOGLEVEL=error|warn|notice` | npm log level |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0|1` | Control sharp/libvips behavior (default: `1`) |

* * *

## [​](https://docs.openclaw.ai/install/installer\\#install-ps1)  install.ps1

### [​](https://docs.openclaw.ai/install/installer\\#flow-install-ps1)  Flow (install.ps1)

1

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Ensure PowerShell + Windows environment

Requires PowerShell 5+.

2

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Ensure Node.js 24 by default

If missing, attempts install via winget, then Chocolatey, then Scoop. Node 22 LTS, currently `22.14+`, remains supported for compatibility.

3

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Install OpenClaw

- `npm` method (default): global npm install using selected `-Tag`
- `git` method: clone/update repo, install/build with pnpm, and install wrapper at `%USERPROFILE%\\.local\\bin\\openclaw.cmd`

4

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Post-install tasks

- Adds needed bin directory to user PATH when possible
- Refreshes a loaded gateway service best-effort (`openclaw gateway install --force`, then restart)
- Runs `openclaw doctor --non-interactive` on upgrades and git installs (best effort)

5

[Navigate to header](https://docs.openclaw.ai/install/installer#)

Handle failures

`iwr ... | iex` and scriptblock installs report a terminating error without closing the current PowerShell session. Direct `powershell -File` / `pwsh -File` installs still exit non-zero for automation.

### [​](https://docs.openclaw.ai/install/installer\\#examples-install-ps1)  Examples (install.ps1)

- Default

- Git install

- GitHub main via npm

- Custom git directory

- Dry run

- Debug trace


```
iwr -useb https://openclaw.ai/install.ps1 | iex
```

```
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -InstallMethod git
```

```
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -Tag main
```

```
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -InstallMethod git -GitDir \"C:\\openclaw\"\n```

```
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -DryRun
```

```
# install.ps1 has no dedicated -Verbose flag yet.
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

Flags reference

| Flag | Description |
| --- | --- |
| `-InstallMethod npm|git` | Install method (default: `npm`) |
| `-Tag <tag|version|spec>` | npm dist-tag, version, or package spec (default: `latest`) |
| `-GitDir <path>` | Checkout directory (default: `%USERPROFILE%\\openclaw`) |
| `-NoOnboard` | Skip onboarding |
| `-NoGitUpdate` | Skip `git pull` |
| `-DryRun` | Print actions only |

Environment variables reference

| Variable | Description |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git|npm` | Install method |
| `OPENCLAW_GIT_DIR=<path>` | Checkout directory |
| `OPENCLAW_NO_ONBOARD=1` | Skip onboarding |
| `OPENCLAW_GIT_UPDATE=0` | Disable git pull |
| `OPENCLAW_DRY_RUN=1` | Dry run mode |

If `-InstallMethod git` is used and Git is missing, the script exits and prints the Git for Windows link.

* * *

## [​](https://docs.openclaw.ai/install/installer\\#ci-and-automation)  CI and automation

Use non-interactive flags/env vars for predictable runs.

- install.sh (non-interactive npm)

- install.sh (non-interactive git)

- install-cli.sh (JSON)

- install.ps1 (skip onboarding)


```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --no-prompt --no-onboard
```

```
OPENCLAW_INSTALL_METHOD=git OPENCLAW_NO_PROMPT=1 \\\n  curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

```
curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --json --prefix /opt/openclaw
```

```
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
```

* * *

## [​](https://docs.openclaw.ai/install/installer\\#troubleshooting)  Troubleshooting

Why is Git required?

Git is required for `git` install method. For `npm` installs, Git is still checked/installed to avoid `spawn git ENOENT` failures when dependencies use git URLs.

Why does npm hit EACCES on Linux?

Some Linux setups point npm global prefix to root-owned paths. `install.sh` can switch prefix to `~/.npm-global` and append PATH exports to shell rc files (when those files exist).

sharp/libvips issues

The scripts default `SHARP_IGNORE_GLOBAL_LIBVIPS=1` to avoid sharp building against system libvips. To override:

```
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL --proto \'=https\' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

Windows: \"npm error spawn git / ENOENT\"

Install Git for Windows, reopen PowerShell, rerun installer.

Windows: \"openclaw is not recognized\"

Run `npm config get prefix` and add that directory to your user PATH (no `\\bin` suffix needed on Windows), then reopen PowerShell.

Windows: how to get verbose installer output

`install.ps1` does not currently expose a `-Verbose` flag yet.
Use PowerShell tracing for script-level diagnostics:

```
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

openclaw not found after install

Usually a PATH issue. See [Node.js troubleshooting](https://docs.openclaw.ai/install/node#troubleshooting).

## [​](https://docs.openclaw.ai/install/installer\\#related)  Related

- [Install overview](https://docs.openclaw.ai/install)
- [Updating](https://docs.openclaw.ai/install/updating)
- [Uninstall](https://docs.openclaw.ai/install/uninstall)

[Install](https://docs.openclaw.ai/install) [Node.js](https://docs.openclaw.ai/install/node)

Ctrl+I

---

## Render - OpenClaw
**Source:** https://docs.openclaw.ai/install/render

[Skip to main content](https://docs.openclaw.ai/install/render#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

Render

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Render](https://docs.openclaw.ai/install/render#render)
- [Prerequisites](https://docs.openclaw.ai/install/render#prerequisites)
- [Deploy with a Render Blueprint](https://docs.openclaw.ai/install/render#deploy-with-a-render-blueprint)
- [Understanding the Blueprint](https://docs.openclaw.ai/install/render#understanding-the-blueprint)
- [Choosing a plan](https://docs.openclaw.ai/install/render#choosing-a-plan)
- [After deployment](https://docs.openclaw.ai/install/render#after-deployment)
- [Access the Control UI](https://docs.openclaw.ai/install/render#access-the-control-ui)
- [Render Dashboard features](https://docs.openclaw.ai/install/render#render-dashboard-features)
- [Logs](https://docs.openclaw.ai/install/render#logs)
- [Shell access](https://docs.openclaw.ai/install/render#shell-access)
- [Environment variables](https://docs.openclaw.ai/install/render#environment-variables)
- [Auto-deploy](https://docs.openclaw.ai/install/render#auto-deploy)
- [Custom domain](https://docs.openclaw.ai/install/render#custom-domain)
- [Scaling](https://docs.openclaw.ai/install/render#scaling)
- [Backups and migration](https://docs.openclaw.ai/install/render#backups-and-migration)
- [Troubleshooting](https://docs.openclaw.ai/install/render#troubleshooting)
- [Service will not start](https://docs.openclaw.ai/install/render#service-will-not-start)
- [Slow cold starts (free tier)](https://docs.openclaw.ai/install/render#slow-cold-starts-free-tier)
- [Data loss after redeploy](https://docs.openclaw.ai/install/render#data-loss-after-redeploy)
- [Health check failures](https://docs.openclaw.ai/install/render#health-check-failures)
- [Next steps](https://docs.openclaw.ai/install/render#next-steps)

# [​](https://docs.openclaw.ai/install/render\\#render)  Render

Deploy OpenClaw on Render using Infrastructure as Code. The included `render.yaml` Blueprint defines your entire stack declaratively, service, disk, environment variables, so you can deploy with a single click and version your infrastructure alongside your code.

## [​](https://docs.openclaw.ai/install/render\\#prerequisites)  Prerequisites

- A [Render account](https://render.com/) (free tier available)
- An API key from your preferred [model provider](https://docs.openclaw.ai/providers)

## [​](https://docs.openclaw.ai/install/render\\#deploy-with-a-render-blueprint)  Deploy with a Render Blueprint

[Deploy to Render](https://render.com/deploy?repo=https://github.com/openclaw/openclaw)Clicking this link will:

1. Create a new Render service from the `render.yaml` Blueprint at the root of this repo.
2. Build the Docker image and deploy

Once deployed, your service URL follows the pattern `https://<service-name>.onrender.com`.

## [​](https://docs.openclaw.ai/install/render\\#understanding-the-blueprint)  Understanding the Blueprint

Render Blueprints are YAML files that define your infrastructure. The `render.yaml` in this
repository configures everything needed to run OpenClaw:

```
services:
  - type: web
    name: openclaw
    runtime: docker
    plan: starter
    healthCheckPath: /health
    envVars:
      - key: OPENCLAW_GATEWAY_PORT
        value: \"8080\"
      - key: OPENCLAW_STATE_DIR
        value: /data/.openclaw
      - key: OPENCLAW_WORKSPACE_DIR
        value: /data/workspace
      - key: OPENCLAW_GATEWAY_TOKEN
        generateValue: true # auto-generates a secure token
    disk:
      name: openclaw-data
      mountPath: /data
      sizeGB: 1
```

Key Blueprint features used:

| Feature | Purpose |
| --- | --- |
| `runtime: docker` | Builds from the repo’s Dockerfile |
| `healthCheckPath` | Render monitors `/health` and restarts unhealthy instances |
| `generateValue: true` | Auto-generates a cryptographically secure value |
| `disk` | Persistent storage that survives redeploys |

## [​](https://docs.openclaw.ai/install/render\\#choosing-a-plan)  Choosing a plan

| Plan | Spin-down | Disk | Best for |
| --- | --- | --- | --- |
| Free | After 15 min idle | Not available | Testing, demos |
| Starter | Never | 1GB+ | Personal use, small teams |
| Standard+ | Never | 1GB+ | Production, multiple channels |

The Blueprint defaults to `starter`. To use free tier, change `plan: free` in
your fork’s `render.yaml` (but note: no persistent disk means OpenClaw state
resets on each deploy).

## [​](https://docs.openclaw.ai/install/render\\#after-deployment)  After deployment

### [​](https://docs.openclaw.ai/install/render\\#access-the-control-ui)  Access the Control UI

The web dashboard is available at `https://<your-service>.onrender.com/`.Connect using the configured shared secret. This deploy template auto-generates
`OPENCLAW_GATEWAY_TOKEN` (find it in **Dashboard → your service →**\n**Environment**); if you replace it with password auth, use that password\ninstead.

## [​](https://docs.openclaw.ai/install/render\\#render-dashboard-features)  Render Dashboard features

### [​](https://docs.openclaw.ai/install/render\\#logs)  Logs

View real-time logs in **Dashboard → your service → Logs**. Filter by:

- Build logs (Docker image creation)
- Deploy logs (service startup)
- Runtime logs (application output)

### [​](https://docs.openclaw.ai/install/render\\#shell-access)  Shell access

For debugging, open a shell session via **Dashboard → your service → Shell**. The persistent disk is mounted at `/data`.

### [​](https://docs.openclaw.ai/install/render\\#environment-variables)  Environment variables

Modify variables in **Dashboard → your service → Environment**. Changes trigger an automatic redeploy.

### [​](https://docs.openclaw.ai/install/render\\#auto-deploy)  Auto-deploy

If you use the original OpenClaw repository, Render will not auto-deploy your OpenClaw. To update it, run a manual Blueprint sync from the dashboard.

## [​](https://docs.openclaw.ai/install/render\\#custom-domain)  Custom domain

1. Go to **Dashboard → your service → Settings → Custom Domains**
2. Add your domain
3. Configure DNS as instructed (CNAME to `*.onrender.com`)
4. Render provisions a TLS certificate automatically

## [​](https://docs.openclaw.ai/install/render\\#scaling)  Scaling

Render supports horizontal and vertical scaling:

- **Vertical**: Change the plan to get more CPU/RAM
- **Horizontal**: Increase instance count (Standard plan and above)

For OpenClaw, vertical scaling is usually sufficient. Horizontal scaling requires sticky sessions or external state management.

## [​](https://docs.openclaw.ai/install/render\\#backups-and-migration)  Backups and migration

Export your state, config, auth profiles, and workspace at any time using the\nshell access in the Render Dashboard:\n\n```\nopenclaw backup create\n```\n
This creates a portable backup archive with OpenClaw state plus any configured\nworkspace. See [Backup](https://docs.openclaw.ai/cli/backup) for details.\n
## [​](https://docs.openclaw.ai/install/render\\#troubleshooting)  Troubleshooting

### [​](https://docs.openclaw.ai/install/render\\#service-will-not-start)  Service will not start

Check the deploy logs in the Render Dashboard. Common issues:\n\n- Missing `OPENCLAW_GATEWAY_TOKEN` — verify it is set in **Dashboard → Environment**\n- Port mismatch — ensure `OPENCLAW_GATEWAY_PORT=8080` is set so the gateway binds to the port Render expects\n\n### [​](https://docs.openclaw.ai/install/render\\#slow-cold-starts-free-tier)  Slow cold starts (free tier)\n
Free tier services spin down after 15 minutes of inactivity. The first request after spin-down takes a few seconds while the container starts. Upgrade to Starter plan for always-on.\n
### [​](https://docs.openclaw.ai/install/render\\#data-loss-after-redeploy)  Data loss after redeploy\n
This happens on free tier (no persistent disk). Upgrade to a paid plan, or\nregularly export a full backup via `openclaw backup create` in the Render shell.\n
### [​](https://docs.openclaw.ai/install/render\\#health-check-failures)  Health check failures\n
Render expects a 200 response from `/health` within 30 seconds. If builds succeed but deploys fail, the service may be taking too long to start. Check:\n\n- Build logs for errors\n- Whether the container runs locally with `docker build && docker run`\n\n## [​](https://docs.openclaw.ai/install/render\\#next-steps)  Next steps\n
- Set up messaging channels: [Channels](https://docs.openclaw.ai/channels)\n- Configure the Gateway: [Gateway configuration](https://docs.openclaw.ai/gateway/configuration)\n- Keep OpenClaw up to date: [Updating](https://docs.openclaw.ai/install/updating)\n
[Raspberry Pi](https://docs.openclaw.ai/install/raspberry-pi) [Setup](https://docs.openclaw.ai/start/setup)\n
Ctrl+I

---

## Raspberry Pi - OpenClaw
**Source:** https://docs.openclaw.ai/install/raspberry-pi

[Skip to main content](https://docs.openclaw.ai/install/raspberry-pi#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nHosting\n\nRaspberry Pi\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Prerequisites](https://docs.openclaw.ai/install/raspberry-pi#prerequisites)\n- [Setup](https://docs.openclaw.ai/install/raspberry-pi#setup)\n- [Performance tips](https://docs.openclaw.ai/install/raspberry-pi#performance-tips)\n- [Troubleshooting](https://docs.openclaw.ai/install/raspberry-pi#troubleshooting)\n- [Next steps](https://docs.openclaw.ai/install/raspberry-pi#next-steps)\n- [Related](https://docs.openclaw.ai/install/raspberry-pi#related)\n\nRun a persistent, always-on OpenClaw Gateway on a Raspberry Pi. Since the Pi is just the gateway (models run in the cloud via API), even a modest Pi handles the workload well.\n\n## [​](https://docs.openclaw.ai/install/raspberry-pi\\#prerequisites)  Prerequisites\n\n- Raspberry Pi 4 or 5 with 2 GB+ RAM (4 GB recommended)\n- MicroSD card (16 GB+) or USB SSD (better performance)\n- Official Pi power supply\n- Network connection (Ethernet or WiFi)\n- 64-bit Raspberry Pi OS (required — do not use 32-bit)\n- About 30 minutes\n\n## [​](https://docs.openclaw.ai/install/raspberry-pi\\#setup)  Setup\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/install/raspberry-pi#)\n\nFlash the OS\n\nUse **Raspberry Pi OS Lite (64-bit)** — no desktop needed for a headless server.\n\n1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/).\n2. Choose OS: **Raspberry Pi OS Lite (64-bit)**.\n3. In the settings dialog, pre-configure:\n   - Hostname: `gateway-host`\n   - Enable SSH\n   - Set username and password\n   - Configure WiFi (if not using Ethernet)\n4. Flash to your SD card or USB drive, insert it, and boot the Pi.\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/install/raspberry-pi#)\n\nConnect via SSH\n\n```\nssh user@gateway-host\n```\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/install/raspberry-pi#)\n\nUpdate the system\n\n```\nsudo apt update && sudo apt upgrade -y\nsudo apt install -y git curl build-essential\n\n# Set timezone (important for cron and reminders)\nsudo timedatectl set-timezone America/Chicago\n```\n\n4\n\n[Navigate to header](https://docs.openclaw.ai/install/raspberry-pi#)\n\nInstall Node.js 24\n\n```\ncurl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -\nsudo apt install -y nodejs\nnode --version\n```\n\n5\n\n[Navigate to header](https://docs.openclaw.ai/install/raspberry-pi#)\n\nAdd swap (important for 2 GB or less)\n\n```\nsudo fallocate -l 2G /swapfile\nsudo chmod 600 /swapfile\nsudo mkswap /swapfile\nsudo swapon /swapfile\necho \'/swapfile none swap sw 0 0\' | sudo tee -a /etc/fstab\n\n# Reduce swappiness for low-RAM devices\necho \'vm.swappiness=10\' | sudo tee -a /etc/sysctl.conf\nsudo sysctl -p\n```\n\n6\n\n[Navigate to header](https://docs.openclaw.ai/install/raspberry-pi#)\n\nInstall OpenClaw\n\n```\ncurl -fsSL https://openclaw.ai/install.sh | bash\n```\n\n7\n\n[Navigate to header](https://docs.openclaw.ai/install/raspberry-pi#)\n\nRun onboarding\n\n```\nopenclaw onboard --install-daemon\n```\n\nFollow the wizard. API keys are recommended over OAuth for headless devices. Telegram is the easiest channel to start with.\n\n8\n\n[Navigate to header](https://docs.openclaw.ai/install/raspberry-pi#)\n\nVerify\n\n```\nopenclaw status\nsystemctl --user status openclaw-gateway.service\njournalctl --user -u openclaw-gateway.service -f\n```\n\n9\n\n[Navigate to header](https://docs.openclaw.ai/install/raspberry-pi#)\n\nAccess the Control UI\n\nOn your computer, get a dashboard URL from the Pi:\n\n```\nssh user@gateway-host \'openclaw dashboard --no-open\'\n```\n\nThen create an SSH tunnel in another terminal:\n\n```\nssh -N -L 18789:127.0.0.1:18789 user@gateway-host\n```\n\nOpen the printed URL in your local browser. For always-on remote access, see [Tailscale integration](https://docs.openclaw.ai/gateway/tailscale).\n\n## [​](https://docs.openclaw.ai/install/raspberry-pi\\#performance-tips)  Performance tips\n\n**Use a USB SSD** — SD cards are slow and wear out. A USB SSD dramatically improves performance. See the [Pi USB boot guide](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot).**Enable module compile cache** — Speeds up repeated CLI invocations on lower-power Pi hosts:\n\n```\ngrep -q \'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache\' ~/.bashrc || cat >> ~/.bashrc <<\'EOF\' # pragma: allowlist secret\nexport NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache\nmkdir -p /var/tmp/openclaw-compile-cache\nexport OPENCLAW_NO_RESPAWN=1\nEOF\nsource ~/.bashrc\n```\n\n**Reduce memory usage** — For headless setups, free GPU memory and disable unused services:\n\n```\necho \'gpu_mem=16\' | sudo tee -a /boot/config.txt\nsudo systemctl disable bluetooth\n```\n\n## [​](https://docs.openclaw.ai/install/raspberry-pi\\#troubleshooting)  Troubleshooting\n\n**Out of memory** — Verify swap is active with `free -h`. Disable unused services (`sudo systemctl disable cups bluetooth avahi-daemon`). Use API-based models only.**Slow performance** — Use a USB SSD instead of an SD card. Check for CPU throttling with `vcgencmd get_throttled` (should return `0x0`).**Service will not start** — Check logs with `journalctl --user -u openclaw-gateway.service --no-pager -n 100` and run `openclaw doctor --non-interactive`. If this is a headless Pi, also verify lingering is enabled: `sudo loginctl enable-linger \"$(whoami)\"`.**ARM binary issues** — If a skill fails with “exec format error”, check whether the binary has an ARM64 build. Verify architecture with `uname -m` (should show `aarch64`).**WiFi drops** — Disable WiFi power management: `sudo iwconfig wlan0 power off`.\n\n## [​](https://docs.openclaw.ai/install/raspberry-pi\\#next-steps)  Next steps\n\n- [Channels](https://docs.openclaw.ai/channels) — connect Telegram, WhatsApp, Discord, and more\n- [Gateway configuration](https://docs.openclaw.ai/gateway/configuration) — all config options\n- [Updating](https://docs.openclaw.ai/install/updating) — keep OpenClaw up to date\n\n## [​](https://docs.openclaw.ai/install/raspberry-pi\\#related)  Related\n\n- [Install overview](https://docs.openclaw.ai/install)\n- [Linux server](https://docs.openclaw.ai/vps)\n- [Platforms](https://docs.openclaw.ai/platforms)\n\n[Railway](https://docs.openclaw.ai/install/railway) [Render](https://docs.openclaw.ai/install/render)\n\nCtrl+I

---

## Kubernetes - OpenClaw
**Source:** https://docs.openclaw.ai/install/kubernetes

[Skip to main content](https://docs.openclaw.ai/install/kubernetes#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

Kubernetes

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [OpenClaw on Kubernetes](https://docs.openclaw.ai/install/kubernetes#openclaw-on-kubernetes)
- [Why not Helm?](https://docs.openclaw.ai/install/kubernetes#why-not-helm)
- [What you need](https://docs.openclaw.ai/install/kubernetes#what-you-need)
- [Quick start](https://docs.openclaw.ai/install/kubernetes#quick-start)
- [Local testing with Kind](https://docs.openclaw.ai/install/kubernetes#local-testing-with-kind)
- [Step by step](https://docs.openclaw.ai/install/kubernetes#step-by-step)
- [1) Deploy](https://docs.openclaw.ai/install/kubernetes#1-deploy)
- [2) Access the gateway](https://docs.openclaw.ai/install/kubernetes#2-access-the-gateway)
- [What gets deployed](https://docs.openclaw.ai/install/kubernetes#what-gets-deployed)
- [Customization](https://docs.openclaw.ai/install/kubernetes#customization)
- [Agent instructions](https://docs.openclaw.ai/install/kubernetes#agent-instructions)
- [Gateway config](https://docs.openclaw.ai/install/kubernetes#gateway-config)
- [Add providers](https://docs.openclaw.ai/install/kubernetes#add-providers)
- [Custom namespace](https://docs.openclaw.ai/install/kubernetes#custom-namespace)
- [Custom image](https://docs.openclaw.ai/install/kubernetes#custom-image)
- [Expose beyond port-forward](https://docs.openclaw.ai/install/kubernetes#expose-beyond-port-forward)
- [Re-deploy](https://docs.openclaw.ai/install/kubernetes#re-deploy)
- [Teardown](https://docs.openclaw.ai/install/kubernetes#teardown)
- [Architecture notes](https://docs.openclaw.ai/install/kubernetes#architecture-notes)
- [File structure](https://docs.openclaw.ai/install/kubernetes#file-structure)
- [Related](https://docs.openclaw.ai/install/kubernetes#related)

# [​](https://docs.openclaw.ai/install/kubernetes\\#openclaw-on-kubernetes)  OpenClaw on Kubernetes

A minimal starting point for running OpenClaw on Kubernetes — not a production-ready deployment. It covers the core resources and is meant to be adapted to your environment.

## [​](https://docs.openclaw.ai/install/kubernetes\\#why-not-helm)  Why not Helm?

OpenClaw is a single container with some config files. The interesting customization is in agent content (markdown files, skills, config overrides), not infrastructure templating. Kustomize handles overlays without the overhead of a Helm chart. If your deployment grows more complex, a Helm chart can be layered on top of these manifests.

## [​](https://docs.openclaw.ai/install/kubernetes\\#what-you-need)  What you need

- A running Kubernetes cluster (AKS, EKS, GKE, k3s, kind, OpenShift, etc.)
- `kubectl` connected to your cluster
- An API key for at least one model provider

## [​](https://docs.openclaw.ai/install/kubernetes\\#quick-start)  Quick start

```
# Replace with your provider: ANTHROPIC, GEMINI, OPENAI, or OPENROUTER
export <PROVIDER>_API_KEY=\"...\"
./scripts/k8s/deploy.sh

kubectl port-forward svc/openclaw 18789:18789 -n openclaw
open http://localhost:18789
```

Retrieve the configured shared secret for the Control UI. This deploy script
creates token auth by default:

```
kubectl get secret openclaw-secrets -n openclaw -o jsonpath=\'{ .data.OPENCLAW_GATEWAY_TOKEN }\' | base64 -d
```

For local debugging, `./scripts/k8s/deploy.sh --show-token` prints the token after deploy.

## [​](https://docs.openclaw.ai/install/kubernetes\\#local-testing-with-kind)  Local testing with Kind

If you don’t have a cluster, create one locally with [Kind](https://kind.sigs.k8s.io/):

```
./scripts/k8s/create-kind.sh           # auto-detects docker or podman
./scripts/k8s/create-kind.sh --delete  # tear down
```

Then deploy as usual with `./scripts/k8s/deploy.sh`.

## [​](https://docs.openclaw.ai/install/kubernetes\\#step-by-step)  Step by step

### [​](https://docs.openclaw.ai/install/kubernetes\\#1-deploy)  1) Deploy

**Option A** — API key in environment (one step):

```
# Replace with your provider: ANTHROPIC, GEMINI, OPENAI, or OPENROUTER
export <PROVIDER>_API_KEY=\"...\"
./scripts/k8s/deploy.sh
```

The script creates a Kubernetes Secret with the API key and an auto-generated gateway token, then deploys. If the Secret already exists, it preserves the current gateway token and any provider keys not being changed.**Option B** — create the secret separately:

```
export <PROVIDER>_API_KEY=\"...\"
./scripts/k8s/deploy.sh --create-secret
./scripts/k8s/deploy.sh
```

Use `--show-token` with either command if you want the token printed to stdout for local testing.

### [​](https://docs.openclaw.ai/install/kubernetes\\#2-access-the-gateway)  2) Access the gateway

```
kubectl port-forward svc/openclaw 18789:18789 -n openclaw
open http://localhost:18789
```

## [​](https://docs.openclaw.ai/install/kubernetes\\#what-gets-deployed)  What gets deployed

```
Namespace: openclaw (configurable via OPENCLAW_NAMESPACE)
├── Deployment/openclaw        # Single pod, init container + gateway
├── Service/openclaw           # ClusterIP on port 18789
├── PersistentVolumeClaim      # 10Gi for agent state and config
├── ConfigMap/openclaw-config  # openclaw.json + AGENTS.md
└── Secret/openclaw-secrets    # Gateway token + API keys
```

## [​](https://docs.openclaw.ai/install/kubernetes\\#customization)  Customization

### [​](https://docs.openclaw.ai/install/kubernetes\\#agent-instructions)  Agent instructions

Edit the `AGENTS.md` in `scripts/k8s/manifests/configmap.yaml` and redeploy:

```
./scripts/k8s/deploy.sh
```

### [​](https://docs.openclaw.ai/install/kubernetes\\#gateway-config)  Gateway config

Edit `openclaw.json` in `scripts/k8s/manifests/configmap.yaml`. See [Gateway configuration](https://docs.openclaw.ai/gateway/configuration) for the full reference.

### [​](https://docs.openclaw.ai/install/kubernetes\\#add-providers)  Add providers

Re-run with additional keys exported:

```
export ANTHROPIC_API_KEY=\"...\"
export OPENAI_API_KEY=\"...\"
./scripts/k8s/deploy.sh --create-secret
./scripts/k8s/deploy.sh
```

Existing provider keys stay in the Secret unless you overwrite them.Or patch the Secret directly:

```
kubectl patch secret openclaw-secrets -n openclaw \\
  -p \'{\"stringData\":{\"<PROVIDER>_API_KEY\":\"...\"}}\'
kubectl rollout restart deployment/openclaw -n openclaw
```

### [​](https://docs.openclaw.ai/install/kubernetes\\#custom-namespace)  Custom namespace

```
OPENCLAW_NAMESPACE=my-namespace ./scripts/k8s/deploy.sh
```

### [​](https://docs.openclaw.ai/install/kubernetes\\#custom-image)  Custom image

Edit the `image` field in `scripts/k8s/manifests/deployment.yaml`:

```
image: ghcr.io/openclaw/openclaw:latest # or pin to a specific version from https://github.com/openclaw/openclaw/releases
```

### [​](https://docs.openclaw.ai/install/kubernetes\\#expose-beyond-port-forward)  Expose beyond port-forward

The default manifests bind the gateway to loopback inside the pod. That works with `kubectl port-forward`, but it does not work with a Kubernetes `Service` or Ingress path that needs to reach the pod IP.If you want to expose the gateway through an Ingress or load balancer:

- Change the gateway bind in `scripts/k8s/manifests/configmap.yaml` from `loopback` to a non-loopback bind that matches your deployment model
- Keep gateway auth enabled and use a proper TLS-terminated entrypoint
- Configure the Control UI for remote access using the supported web security model (for example HTTPS/Tailscale Serve and explicit allowed origins when needed)

## [​](https://docs.openclaw.ai/install/kubernetes\\#re-deploy)  Re-deploy

```
./scripts/k8s/deploy.sh
```

This applies all manifests and restarts the pod to pick up any config or secret changes.

## [​](https://docs.openclaw.ai/install/kubernetes\\#teardown)  Teardown

```
./scripts/k8s/deploy.sh --delete
```

This deletes the namespace and all resources in it, including the PVC.

## [​](https://docs.openclaw.ai/install/kubernetes\\#architecture-notes)  Architecture notes

- The gateway binds to loopback inside the pod by default, so the included setup is for `kubectl port-forward`
- No cluster-scoped resources — everything lives in a single namespace
- Security: `readOnlyRootFilesystem`, `drop: ALL` capabilities, non-root user (UID 1000)
- The default config keeps the Control UI on the safer local-access path: loopback bind plus `kubectl port-forward` to `http://127.0.0.1:18789`
- If you move beyond localhost access, use the supported remote model: HTTPS/Tailscale plus the appropriate gateway bind and Control UI origin settings
- Secrets are generated in a temp directory and applied directly to the cluster — no secret material is written to the repo checkout

## [​](https://docs.openclaw.ai/install/kubernetes\\#file-structure)  File structure

```
scripts/k8s/
├── deploy.sh                   # Creates namespace + secret, deploys via kustomize
├── create-kind.sh              # Local Kind cluster (auto-detects docker/podman)
└── manifests/
    ├── kustomization.yaml      # Kustomize base
    ├── configmap.yaml          # openclaw.json + AGENTS.md
    ├── deployment.yaml         # Pod spec with security hardening
    ├── pvc.yaml                # 10Gi persistent storage
    └── service.yaml            # ClusterIP on 18789
```

## [​](https://docs.openclaw.ai/install/kubernetes\\#related)  Related

- [Docker](https://docs.openclaw.ai/install/docker)
- [Docker VM runtime](https://docs.openclaw.ai/install/docker-vm-runtime)
- [Install overview](https://docs.openclaw.ai/install)

[Hostinger](https://docs.openclaw.ai/install/hostinger) [Linux Server](https://docs.openclaw.ai/vps)

Ctrl+I

---

## Release channels - OpenClaw
**Source:** https://docs.openclaw.ai/install/development-channels

[Skip to main content](https://docs.openclaw.ai/install/development-channels#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nMaintenance\n\nRelease channels\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Development channels](https://docs.openclaw.ai/install/development-channels#development-channels)\n- [Switching channels](https://docs.openclaw.ai/install/development-channels#switching-channels)\n- [One-off version or tag targeting](https://docs.openclaw.ai/install/development-channels#one-off-version-or-tag-targeting)\n- [Dry run](https://docs.openclaw.ai/install/development-channels#dry-run)\n- [Plugins and channels](https://docs.openclaw.ai/install/development-channels#plugins-and-channels)\n- [Checking current status](https://docs.openclaw.ai/install/development-channels#checking-current-status)\n- [Tagging best practices](https://docs.openclaw.ai/install/development-channels#tagging-best-practices)\n- [macOS app availability](https://docs.openclaw.ai/install/development-channels#macos-app-availability)\n- [Related](https://docs.openclaw.ai/install/development-channels#related)\n\n# [​](https://docs.openclaw.ai/install/development-channels\\#development-channels)  Development channels\n\nOpenClaw ships three update channels:\n\n- **stable**: npm dist-tag `latest`. Recommended for most users.\n- **beta**: npm dist-tag `beta` when it is current; if beta is missing or older than\nthe latest stable release, the update flow falls back to `latest`.\n- **dev**: moving head of `main` (git). npm dist-tag: `dev` (when published).\nThe `main` branch is for experimentation and active development. It may contain\nincomplete features or breaking changes. Do not use it for production gateways.\n\nWe usually ship stable builds to **beta** first, test them there, then run an\nexplicit promotion step that moves the vetted build to `latest` without\nchanging the version number. Maintainers can also publish a stable release\ndirectly to `latest` when needed. Dist-tags are the source of truth for npm\ninstalls.\n\n## [​](https://docs.openclaw.ai/install/development-channels\\#switching-channels)  Switching channels\n\n```\nopenclaw update --channel stable\nopenclaw update --channel beta\nopenclaw update --channel dev\n```\n\n`--channel` persists your choice in config (`update.channel`) and aligns the\ninstall method:\n\n- **`stable`** (package installs): updates via npm dist-tag `latest`.\n- **`beta`** (package installs): prefers npm dist-tag `beta`, but falls back to\n`latest` when `beta` is missing or older than the current stable tag.\n- **`stable`** (git installs): checks out the latest stable git tag.\n- **`beta`** (git installs): prefers the latest beta git tag, but falls back to\nthe latest stable git tag when beta is missing or older.\n- **`dev`**: ensures a git checkout (default `~/openclaw`, override with\n`OPENCLAW_GIT_DIR`), switches to `main`, rebases on upstream, builds, and\ninstalls the global CLI from that checkout.\n\nIf you want stable and dev in parallel, keep two clones and point your gateway at the stable one.\n\n## [​](https://docs.openclaw.ai/install/development-channels\\#one-off-version-or-tag-targeting)  One-off version or tag targeting\n\nUse `--tag` to target a specific dist-tag, version, or package spec for a single\nupdate **without** changing your persisted channel:\n\n```\n# Install a specific version\nopenclaw update --tag 2026.4.1-beta.1\n\n# Install from the beta dist-tag (one-off, does not persist)\nopenclaw update --tag beta\n\n# Install from GitHub main branch (npm tarball)\nopenclaw update --tag main\n\n# Install a specific npm package spec\nopenclaw update --tag openclaw@2026.4.1-beta.1\n```\n\nNotes:\n\n- `--tag` applies to **package (npm) installs only**. Git installs ignore it.\n- The tag is not persisted. Your next `openclaw update` uses your configured\nchannel as usual.\n- Downgrade protection: if the target version is older than your current version,\nOpenClaw prompts for confirmation (skip with `--yes`).\n- `--channel beta` is different from `--tag beta`: the channel flow can fall back\nto stable/latest when beta is missing or older, while `--tag beta` targets the\nraw `beta` dist-tag for that one run.\n\n## [​](https://docs.openclaw.ai/install/development-channels\\#dry-run)  Dry run\n\nPreview what `openclaw update` would do without making changes:\n\n```\nopenclaw update --dry-run\nopenclaw update --channel beta --dry-run\nopenclaw update --tag 2026.4.1-beta.1 --dry-run\nopenclaw update --dry-run --json\n```\n\nThe dry run shows the effective channel, target version, planned actions, and\nwhether a downgrade confirmation would be required.\n\n## [​](https://docs.openclaw.ai/install/development-channels\\#plugins-and-channels)  Plugins and channels\n\nWhen you switch channels with `openclaw update`, OpenClaw also syncs plugin\nsources:\n\n- `dev` prefers bundled plugins from the git checkout.\n- `stable` and `beta` restore npm-installed plugin packages.\n- npm-installed plugins are updated after the core update completes.\n\n## [​](https://docs.openclaw.ai/install/development-channels\\#checking-current-status)  Checking current status\n\n```\nopenclaw update status\n```\n\nShows the active channel, install kind (git or package), current version, and\nsource (config, git tag, git branch, or default).\n\n## [​](https://docs.openclaw.ai/install/development-channels\\#tagging-best-practices)  Tagging best practices\n\n- Tag releases you want git checkouts to land on (`vYYYY.M.D` for stable,\n`vYYYY.M.D-beta.N` for beta).\n- `vYYYY.M.D.beta.N` is also recognized for compatibility, but prefer `-beta.N`.\n- Legacy `vYYYY.M.D-<patch>` tags are still recognized as stable (non-beta).\n- Keep tags immutable: never move or reuse a tag.\n- npm dist-tags remain the source of truth for npm installs:\n  - `latest` -\\> stable\n  - `beta` -\\> candidate build or beta-first stable build\n  - `dev` -\\> main snapshot (optional)\n\n## [​](https://docs.openclaw.ai/install/development-channels\\#macos-app-availability)  macOS app availability\n\nBeta and dev builds may **not** include a macOS app release. That is OK:\n\n- The git tag and npm dist-tag can still be published.\n- Call out “no macOS build for this beta” in release notes or changelog.\n\n## [​](https://docs.openclaw.ai/install/development-channels\\#related)  Related\n\n- [Updating](https://docs.openclaw.ai/install/updating)\n- [Installer internals](https://docs.openclaw.ai/install/installer)\n\n[Uninstall](https://docs.openclaw.ai/install/uninstall) [Ansible](https://docs.openclaw.ai/install/ansible)\n\nCtrl+I

---

## Matrix migration
**Source:** https://docs.openclaw.ai/install/migrating-matrix

[Skip to main content](https://docs.openclaw.ai/install/migrating-matrix#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Maintenance

Matrix migration

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [What the migration does automatically](https://docs.openclaw.ai/install/migrating-matrix#what-the-migration-does-automatically)
- [What the migration cannot do automatically](https://docs.openclaw.ai/install/migrating-matrix#what-the-migration-cannot-do-automatically)
- [Recommended upgrade flow](https://docs.openclaw.ai/install/migrating-matrix#recommended-upgrade-flow)
- [How encrypted migration works](https://docs.openclaw.ai/install/migrating-matrix#how-encrypted-migration-works)
- [Common messages and what they mean](https://docs.openclaw.ai/install/migrating-matrix#common-messages-and-what-they-mean)
- [Upgrade and detection messages](https://docs.openclaw.ai/install/migrating-matrix#upgrade-and-detection-messages)
- [Encrypted-state recovery messages](https://docs.openclaw.ai/install/migrating-matrix#encrypted-state-recovery-messages)
- [Manual recovery messages](https://docs.openclaw.ai/install/migrating-matrix#manual-recovery-messages)
- [Custom plugin install messages](https://docs.openclaw.ai/install/migrating-matrix#custom-plugin-install-messages)
- [If encrypted history still does not come back](https://docs.openclaw.ai/install/migrating-matrix#if-encrypted-history-still-does-not-come-back)
- [If you want to start fresh for future messages](https://docs.openclaw.ai/install/migrating-matrix#if-you-want-to-start-fresh-for-future-messages)
- [Related pages](https://docs.openclaw.ai/install/migrating-matrix#related-pages)

Upgrade from the previous public `matrix` plugin to the current implementation.For most users, the upgrade is in place:

- the plugin stays `@openclaw/matrix`
- the channel stays `matrix`
- your config stays under `channels.matrix`
- cached credentials stay under `~/.openclaw/credentials/matrix/`
- runtime state stays under `~/.openclaw/matrix/`

You do not need to rename config keys or reinstall the plugin under a new name.

## [​](https://docs.openclaw.ai/install/migrating-matrix\\#what-the-migration-does-automatically)  What the migration does automatically

When the gateway starts, and when you run [`openclaw doctor --fix`](https://docs.openclaw.ai/gateway/doctor), OpenClaw tries to repair old Matrix state automatically.
Before any actionable Matrix migration step mutates on-disk state, OpenClaw creates or reuses a focused recovery snapshot.When you use `openclaw update`, the exact trigger depends on how OpenClaw is installed:

- source installs run `openclaw doctor --fix` during the update flow, then restart the gateway by default
- package-manager installs update the package, run a non-interactive doctor pass, then rely on the default gateway restart so startup can finish Matrix migration
- if you use `openclaw update --no-restart`, startup-backed Matrix migration is deferred until you later run `openclaw doctor --fix` and restart the gateway

Automatic migration covers:

- creating or reusing a pre-migration snapshot under `~/Backups/openclaw-migrations/`
- reusing your cached Matrix credentials
- keeping the same account selection and `channels.matrix` config
- moving the oldest flat Matrix sync store into the current account-scoped location
- moving the oldest flat Matrix crypto store into the current account-scoped location when the target account can be resolved safely
- extracting a previously saved Matrix room-key backup decryption key from the old rust crypto store, when that key exists locally
- reusing the most complete existing token-hash storage root for the same Matrix account, homeserver, and user when the access token changes later
- scanning sibling token-hash storage roots for pending encrypted-state restore metadata when the Matrix access token changed but the account/device identity stayed the same
- restoring backed-up room keys into the new crypto store on the next Matrix startup

Snapshot details:

- OpenClaw writes a marker file at `~/.openclaw/matrix/migration-snapshot.json` after a successful snapshot so later startup and repair passes can reuse the same archive.
- These automatic Matrix migration snapshots back up config + state only (`includeWorkspace: false`).
- If Matrix only has warning-only migration state, for example because `userId` or `accessToken` is still missing, OpenClaw does not create the snapshot yet because no Matrix mutation is actionable.
- If the snapshot step fails, OpenClaw skips Matrix migration for that run instead of mutating state without a recovery point.

About multi-account upgrades:

- the oldest flat Matrix store (`~/.openclaw/matrix/bot-storage.json` and `~/.openclaw/matrix/crypto/`) came from a single-store layout, so OpenClaw can only migrate it into one resolved Matrix account target
- already account-scoped legacy Matrix stores are detected and prepared per configured Matrix account

## [​](https://docs.openclaw.ai/install/migrating-matrix\\#what-the-migration-cannot-do-automatically)  What the migration cannot do automatically

The previous public Matrix plugin did **not** automatically create Matrix room-key backups. It persisted local crypto state and requested device verification, but it did not guarantee that your room keys were backed up to the homeserver.That means some encrypted installs can only be migrated partially.OpenClaw cannot automatically recover:

- local-only room keys that were never backed up
- encrypted state when the target Matrix account cannot be resolved yet because `homeserver`, `userId`, or `accessToken` are still unavailable
- automatic migration of one shared flat Matrix store when multiple Matrix accounts are configured but `channels.matrix.defaultAccount` is not set
- custom plugin path installs that are pinned to a repo path instead of the standard Matrix package
- a missing recovery key when the old store had backed-up keys but did not keep the decryption key locally

Current warning scope:

- custom Matrix plugin path installs are surfaced by both gateway startup and `openclaw doctor`

If your old installation had local-only encrypted history that was never backed up, some older encrypted messages may remain unreadable after the upgrade.

## [​](https://docs.openclaw.ai/install/migrating-matrix\\#recommended-upgrade-flow)  Recommended upgrade flow

1. Update OpenClaw and the Matrix plugin normally.
Prefer plain `openclaw update` without `--no-restart` so startup can finish the Matrix migration immediately.
2. Run:















```
openclaw doctor --fix
```











If Matrix has actionable migration work, doctor will create or reuse the pre-migration snapshot first and print the archive path.
3. Start or restart the gateway.
4. Check current verification and backup state:















```
openclaw matrix verify status
openclaw matrix verify backup status
```

5. Put the recovery key for the Matrix account you are repairing in an account-specific environment variable. For a single default account, `MATRIX_RECOVERY_KEY` is fine. For multiple accounts, use one variable per account, for example `MATRIX_RECOVERY_KEY_ASSISTANT`, and add `--account assistant` to the command.
6. If OpenClaw tells you a recovery key is needed, run the command for the matching account:















```
printf '%s\n' \"$MATRIX_RECOVERY_KEY\" | openclaw matrix verify backup restore --recovery-key-stdin\nprintf '%s\n' \"$MATRIX_RECOVERY_KEY_ASSISTANT\" | openclaw matrix verify backup restore --recovery-key-stdin --account assistant\n```

7. If this device is still unverified, run the command for the matching account:















```
printf '%s\n' \"$MATRIX_RECOVERY_KEY\" | openclaw matrix verify device --recovery-key-stdin\nprintf '%s\n' \"$MATRIX_RECOVERY_KEY_ASSISTANT\" | openclaw matrix verify device --recovery-key-stdin --account assistant\n```











If the recovery key is accepted and backup is usable, but `Cross-signing verified`
is still `no`, complete self-verification from another Matrix client:















```
openclaw matrix verify self
```











Accept the request in another Matrix client, compare the emoji or decimals,
and type `yes` only when they match. The command exits successfully only
after `Cross-signing verified` becomes `yes`.
8. If you are intentionally abandoning unrecoverable old history and want a fresh backup baseline for future messages, run:















```
openclaw matrix verify backup reset --yes
```

9. If no server-side key backup exists yet, create one for future recoveries:















```
openclaw matrix verify bootstrap
```


## [​](https://docs.openclaw.ai/install/migrating-matrix\\#how-encrypted-migration-works)  How encrypted migration works

Encrypted migration is a two-stage process:

1. Startup or `openclaw doctor --fix` creates or reuses the pre-migration snapshot if encrypted migration is actionable.
2. Startup or `openclaw doctor --fix` inspects the old Matrix crypto store through the active Matrix plugin install.
3. If a backup decryption key is found, OpenClaw writes it into the new recovery-key flow and marks room-key restore as pending.
4. On the next Matrix startup, OpenClaw restores backed-up room keys into the new crypto store automatically.

If the old store reports room keys that were never backed up, OpenClaw warns instead of pretending recovery succeeded.

## [​](https://docs.openclaw.ai/install/migrating-matrix\\#common-messages-and-what-they-mean)  Common messages and what they mean

### [​](https://docs.openclaw.ai/install/migrating-matrix\\#upgrade-and-detection-messages)  Upgrade and detection messages

`Matrix plugin upgraded in place.`

- Meaning: the old on-disk Matrix state was detected and migrated into the current layout.
- What to do: nothing unless the same output also includes warnings.

`Matrix migration snapshot created before applying Matrix upgrades.`

- Meaning: OpenClaw created a recovery archive before mutating Matrix state.
- What to do: keep the printed archive path until you confirm migration succeeded.

`Matrix migration snapshot reused before applying Matrix upgrades.`

- Meaning: OpenClaw found an existing Matrix migration snapshot marker and reused that archive instead of creating a duplicate backup.
- What to do: keep the printed archive path until you confirm migration succeeded.

`Legacy Matrix state detected at ... but channels.matrix is not configured yet.`

- Meaning: old Matrix state exists, but OpenClaw cannot map it to a current Matrix account because Matrix is not configured.
- What to do: configure `channels.matrix`, then rerun `openclaw doctor --fix` or restart the gateway.

`Legacy Matrix state detected at ... but the new account-scoped target could not be resolved yet (need homeserver, userId, and access token for channels.matrix...).`

- Meaning: OpenClaw found old state, but it still cannot determine the exact current account/device root.
- What to do: start the gateway once with a working Matrix login, or rerun `openclaw doctor --fix` after cached credentials exist.

`Legacy Matrix state detected at ... but multiple Matrix accounts are configured and channels.matrix.defaultAccount is not set.`

- Meaning: OpenClaw found one shared flat Matrix store, but it refuses to guess which named Matrix account should receive it.
- What to do: set `channels.matrix.defaultAccount` to the intended account, then rerun `openclaw doctor --fix` or restart the gateway.

`Matrix legacy sync store not migrated because the target already exists (...)`

- Meaning: the new account-scoped location already has a sync or crypto store, so OpenClaw did not overwrite it automatically.
- What to do: verify that the current account is the correct one before manually removing or moving the conflicting target.

`Failed migrating Matrix legacy sync store (...)` or `Failed migrating Matrix legacy crypto store (...)`

- Meaning: OpenClaw tried to move old Matrix state but the filesystem operation failed.
- What to do: inspect filesystem permissions and disk state, then rerun `openclaw doctor --fix`.

`Legacy Matrix encrypted state detected at ... but channels.matrix is not configured yet.`

- Meaning: OpenClaw found an old encrypted Matrix store, but there is no current Matrix config to attach it to.
- What to do: configure `channels.matrix`, then rerun `openclaw doctor --fix` or restart the gateway.

`Legacy Matrix encrypted state detected at ... but the account-scoped target could not be resolved yet (need homeserver, userId, and access token for channels.matrix...).`

- Meaning: the encrypted store exists, but OpenClaw cannot safely decide which current account/device it belongs to.
- What to do: start the gateway once with a working Matrix login, or rerun `openclaw doctor --fix` after cached credentials are available.

`Legacy Matrix encrypted state detected at ... but multiple Matrix accounts are configured and channels.matrix.defaultAccount is not set.`

- Meaning: OpenClaw found one shared flat legacy crypto store, but it refuses to guess which named Matrix account should receive it.
- What to do: set `channels.matrix.defaultAccount` to the intended account, then rerun `openclaw doctor --fix` or restart the gateway.

`Matrix migration warnings are present, but no on-disk Matrix mutation is actionable yet. No pre-migration snapshot was needed.`

- Meaning: OpenClaw detected old Matrix state, but the migration is still blocked on missing identity or credential data.
- What to do: finish Matrix login or config setup, then rerun `openclaw doctor --fix` or restart the gateway.

`Legacy Matrix encrypted state was detected, but the Matrix plugin helper is unavailable. Install or repair @openclaw/matrix so OpenClaw can inspect the old rust crypto store before upgrading.`

- Meaning: OpenClaw found old encrypted Matrix state, but it could not load the helper entrypoint from the Matrix plugin that normally inspects that store.
- What to do: reinstall or repair the Matrix plugin (`openclaw plugins install @openclaw/matrix`, or `openclaw plugins install ./path/to/local/matrix-plugin` for a repo checkout), then rerun `openclaw doctor --fix` or restart the gateway.

`Matrix plugin helper path is unsafe: ... Reinstall @openclaw/matrix and try again.`

- Meaning: OpenClaw found a helper file path that escapes the plugin root or fails plugin boundary checks, so it refused to import it.
- What to do: reinstall the Matrix plugin from a trusted path, then rerun `openclaw doctor --fix` or restart the gateway.

`- Failed creating a Matrix migration snapshot before repair: ...``- Skipping Matrix migration changes for now. Resolve the snapshot failure, then rerun \"openclaw doctor --fix\".`

- Meaning: OpenClaw refused to mutate Matrix state because it could not create the recovery snapshot first.
- What to do: resolve the backup error, then rerun `openclaw doctor --fix` or restart the gateway.

`Failed migrating legacy Matrix client storage: ...`

- Meaning: the Matrix client-side fallback found old flat storage, but the move failed. OpenClaw now aborts that fallback instead of silently starting with a fresh store.
- What to do: inspect filesystem permissions or conflicts, keep the old state intact, and retry after fixing the error.

`Matrix is installed from a custom path: ...`

- Meaning: Matrix is pinned to a path install, so mainline updates do not automatically replace it with the repo’s standard Matrix package.
- What to do: reinstall with `openclaw plugins install @openclaw/matrix` when you want to return to the default Matrix plugin.`

---

## Migrating from Hermes - OpenClaw
**Source:** https://docs.openclaw.ai/install/migrating-hermes

[Skip to main content](https://docs.openclaw.ai/install/migrating-hermes#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Migrating

Migrating from Hermes

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Two ways to import](https://docs.openclaw.ai/install/migrating-hermes#two-ways-to-import)
- [What gets imported](https://docs.openclaw.ai/install/migrating-hermes#what-gets-imported)
- [What stays archive-only](https://docs.openclaw.ai/install/migrating-hermes#what-stays-archive-only)
- [Recommended flow](https://docs.openclaw.ai/install/migrating-hermes#recommended-flow)
- [Conflict handling](https://docs.openclaw.ai/install/migrating-hermes#conflict-handling)
- [Secrets](https://docs.openclaw.ai/install/migrating-hermes#secrets)
- [JSON output for automation](https://docs.openclaw.ai/install/migrating-hermes#json-output-for-automation)
- [Troubleshooting](https://docs.openclaw.ai/install/migrating-hermes#troubleshooting)
- [Related](https://docs.openclaw.ai/install/migrating-hermes#related)

OpenClaw imports Hermes state through a bundled migration provider. The provider previews everything before changing state, redacts secrets in plans and reports, and creates a verified backup before apply.

Imports require a fresh OpenClaw setup. If you already have local OpenClaw state, reset config, credentials, sessions, and the workspace first, or use `openclaw migrate` directly with `--overwrite` after reviewing the plan.

## [​](https://docs.openclaw.ai/install/migrating-hermes\\#two-ways-to-import)  Two ways to import

- Onboarding wizard

- CLI


The fastest path. The wizard detects Hermes at `~/.hermes` and shows a preview before applying.

```
openclaw onboard --flow import
```

Or point at a specific source:

```
openclaw onboard --import-from hermes --import-source ~/.hermes
```

Use `openclaw migrate` for scripted or repeatable runs. See [`openclaw migrate`](https://docs.openclaw.ai/cli/migrate) for the full reference.

```
openclaw migrate hermes --dry-run    # preview only
openclaw migrate apply hermes --yes  # apply with confirmation skipped
```

Add `--from <path>` when Hermes lives outside `~/.hermes`.

## [​](https://docs.openclaw.ai/install/migrating-hermes\\#what-gets-imported)  What gets imported

Model configuration

- Default model selection from Hermes `config.yaml`.
- Configured model providers and custom OpenAI-compatible endpoints from `providers` and `custom_providers`.

MCP servers

MCP server definitions from `mcp_servers` or `mcp.servers`.

Workspace files

- `SOUL.md` and `AGENTS.md` are copied into the OpenClaw agent workspace.
- `memories/MEMORY.md` and `memories/USER.md` are **appended** to the matching OpenClaw memory files instead of overwriting them.

Memory configuration

Memory config defaults for OpenClaw file memory. External memory providers such as Honcho are recorded as archive or manual-review items so you can move them deliberately.

Skills

Skills with a `SKILL.md` file under `skills/<name>/` are copied, along with per-skill config values from `skills.config`.

API keys (opt-in)

Set `--include-secrets` to import supported `.env` keys: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY`, `GOOGLE_API_KEY`, `GEMINI_API_KEY`, `GROQ_API_KEY`, `XAI_API_KEY`, `MISTRAL_API_KEY`, `DEEPSEEK_API_KEY`. Without the flag, secrets are never copied.

## [​](https://docs.openclaw.ai/install/migrating-hermes\\#what-stays-archive-only)  What stays archive-only

The provider copies these into the migration report directory for manual review, but does **not** load them into live OpenClaw config or credentials:

- `plugins/`
- `sessions/`
- `logs/`
- `cron/`
- `mcp-tokens/`
- `auth.json`
- `state.db`

OpenClaw refuses to execute or trust this state automatically because the formats and trust assumptions can drift between systems. Move what you need by hand after reviewing the archive.

## [​](https://docs.openclaw.ai/install/migrating-hermes\\#recommended-flow)  Recommended flow

1

[Navigate to header](https://docs.openclaw.ai/install/migrating-hermes#)

Preview the plan

```
openclaw migrate hermes --dry-run
```

The plan lists everything that will change, including conflicts, skipped items, and any sensitive items. Plan output redacts nested secret-looking keys.

2

[Navigate to header](https://docs.openclaw.ai/install/migrating-hermes#)

Apply with backup

```
openclaw migrate apply hermes --yes
```

OpenClaw creates and verifies a backup before applying. If you need API keys imported, add `--include-secrets`.

3

[Navigate to header](https://docs.openclaw.ai/install/migrating-hermes#)

Run doctor

```
openclaw doctor
```

[Doctor](https://docs.openclaw.ai/gateway/doctor) reapplies any pending config migrations and checks for issues introduced during the import.

4

[Navigate to header](https://docs.openclaw.ai/install/migrating-hermes#)

Restart and verify

```
openclaw gateway restart
openclaw status
```

Confirm the gateway is healthy and your imported model, memory, and skills are loaded.

## [​](https://docs.openclaw.ai/install/migrating-hermes\\#conflict-handling)  Conflict handling

Apply refuses to continue when the plan reports conflicts (a file or config value already exists at the target).

Rerun with `--overwrite` only when replacing the existing target is intentional. Providers may still write item-level backups for overwritten files in the migration report directory.

For a fresh OpenClaw install, conflicts are unusual. They typically appear when you re-run the import on a setup that already has user edits.If a conflict surfaces mid-apply (for example, an unexpected race on a config file), Hermes marks remaining dependent config items as `skipped` with reason `blocked by earlier apply conflict` instead of writing them partially. The migration report records each blocked item so you can resolve the original conflict and rerun the import.

## [​](https://docs.openclaw.ai/install/migrating-hermes\\#secrets)  Secrets

Secrets are never imported by default.

- Run `openclaw migrate apply hermes --yes` first to import non-secret state.
- If you also want supported `.env` keys copied across, rerun with `--include-secrets`.
- For SecretRef-managed credentials, configure the SecretRef source after the import completes.

## [​](https://docs.openclaw.ai/install/migrating-hermes\\#json-output-for-automation)  JSON output for automation

```
openclaw migrate hermes --dry-run --json
openclaw migrate apply hermes --json --yes
```

With `--json` and no `--yes`, apply prints the plan and does not mutate state. This is the safest mode for CI and shared scripts.

## [​](https://docs.openclaw.ai/install/migrating-hermes\\#troubleshooting)  Troubleshooting

Apply refuses with conflicts

Inspect the plan output. Each conflict identifies the source path and the existing target. Decide per item whether to skip, edit the target, or rerun with `--overwrite`.

Hermes lives outside ~/.hermes

Pass `--from /actual/path` (CLI) or `--import-source /actual/path` (onboarding).

Onboarding refuses to import on an existing setup

Onboarding imports require a fresh setup. Either reset state and re-onboard, or use `openclaw migrate apply hermes` directly, which supports `--overwrite` and explicit backup control.

API keys did not import

`--include-secrets` is required, and only the keys listed above are recognized. Other variables in `.env` are ignored.

## [​](https://docs.openclaw.ai/install/migrating-hermes\\#related)  Related

- [`openclaw migrate`](https://docs.openclaw.ai/cli/migrate): full CLI reference, plugin contract, and JSON shapes.
- [Onboarding](https://docs.openclaw.ai/cli/onboard): wizard flow and non-interactive flags.
- [Migrating](https://docs.openclaw.ai/install/migrating): move an OpenClaw install between machines.
- [Doctor](https://docs.openclaw.ai/gateway/doctor): post-migration health check.
- [Agent workspace](https://docs.openclaw.ai/concepts/agent-workspace): where `SOUL.md`, `AGENTS.md`, and memory files live.

[Migrating from Claude](https://docs.openclaw.ai/install/migrating-claude) [Uninstall](https://docs.openclaw.ai/install/uninstall)

Ctrl+I

---

## Hetzner - OpenClaw
**Source:** https://docs.openclaw.ai/install/hetzner

[Skip to main content](https://docs.openclaw.ai/install/hetzner#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

Hetzner

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [OpenClaw on Hetzner (Docker, Production VPS Guide)](https://docs.openclaw.ai/install/hetzner#openclaw-on-hetzner-docker-production-vps-guide)
- [Goal](https://docs.openclaw.ai/install/hetzner#goal)
- [What are we doing (simple terms)?](https://docs.openclaw.ai/install/hetzner#what-are-we-doing-simple-terms-)
- [Quick path (experienced operators)](https://docs.openclaw.ai/install/hetzner#quick-path-experienced-operators)
- [What you need](https://docs.openclaw.ai/install/hetzner#what-you-need)
- [Infrastructure as Code (Terraform)](https://docs.openclaw.ai/install/hetzner#infrastructure-as-code-terraform)
- [Next steps](https://docs.openclaw.ai/install/hetzner#next-steps)
- [Related](https://docs.openclaw.ai/install/hetzner#related)

# [​](https://docs.openclaw.ai/install/hetzner\\#openclaw-on-hetzner-docker-production-vps-guide)  OpenClaw on Hetzner (Docker, Production VPS Guide)

## [​](https://docs.openclaw.ai/install/hetzner\\#goal)  Goal

Run a persistent OpenClaw Gateway on a Hetzner VPS using Docker, with durable state, baked-in binaries, and safe restart behavior.If you want “OpenClaw 24/7 for ~$5”, this is the simplest reliable setup.\nHetzner pricing changes; pick the smallest Debian/Ubuntu VPS and scale up if you hit OOMs.Security model reminder:\n\n- Company-shared agents are fine when everyone is in the same trust boundary and the runtime is business-only.\n- Keep strict separation: dedicated VPS/runtime + dedicated accounts; no personal Apple/Google/browser/password-manager profiles on that host.\n- If users are adversarial to each other, split by gateway/host/OS user.\n
See [Security](https://docs.openclaw.ai/gateway/security) and [VPS hosting](https://docs.openclaw.ai/vps).\n
## [​](https://docs.openclaw.ai/install/hetzner\\#what-are-we-doing-simple-terms-)  What are we doing (simple terms)?\n\n- Rent a small Linux server (Hetzner VPS)\n- Install Docker (isolated app runtime)\n- Start the OpenClaw Gateway in Docker\n- Persist `~/.openclaw` \\+ `~/.openclaw/workspace` on the host (survives restarts/rebuilds)\n- Access the Control UI from your laptop via an SSH tunnel\n
That mounted `~/.openclaw` state includes `openclaw.json`, per-agent\n`agents/<agentId>/agent/auth-profiles.json`, and `.env`.The Gateway can be accessed via:\n\n- SSH port forwarding from your laptop\n- Direct port exposure if you manage firewalling and tokens yourself\n
This guide assumes Ubuntu or Debian on Hetzner.\n
If you are on another Linux VPS, map packages accordingly.\nFor the generic Docker flow, see [Docker](https://docs.openclaw.ai/install/docker).\n
* * *\n\n## [​](https://docs.openclaw.ai/install/hetzner\\#quick-path-experienced-operators)  Quick path (experienced operators)\n\n1. Provision Hetzner VPS\n2. Install Docker\n3. Clone OpenClaw repository\n4. Create persistent host directories\n5. Configure `.env` and `docker-compose.yml`\n6. Bake required binaries into the image\n7. `docker compose up -d`\n8. Verify persistence and Gateway access\n
* * *\n\n## [​](https://docs.openclaw.ai/install/hetzner\\#what-you-need)  What you need\n\n- Hetzner VPS with root access\n- SSH access from your laptop\n- Basic comfort with SSH + copy/paste\n- ~20 minutes\n- Docker and Docker Compose\n- Model auth credentials\n- Optional provider credentials\n  - WhatsApp QR\n  - Telegram bot token\n  - Gmail OAuth\n
* * *\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/install/hetzner#)\n\nProvision the VPS\n\nCreate an Ubuntu or Debian VPS in Hetzner.Connect as root:\n\n```\nssh root@YOUR_VPS_IP\n```\n\nThis guide assumes the VPS is stateful.\nDo not treat it as disposable infrastructure.\n
2\n
[Navigate to header](https://docs.openclaw.ai/install/hetzner#)\n\nInstall Docker (on the VPS)\n\n```\napt-get update\napt-get install -y git curl ca-certificates\ncurl -fsSL https://get.docker.com | sh\n```\n\nVerify:\n\n```\ndocker --version\ndocker compose version\n```\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/install/hetzner#)\n\nClone the OpenClaw repository\n\n```\ngit clone https://github.com/openclaw/openclaw.git\ncd openclaw\n```\n\nThis guide assumes you will build a custom image to guarantee binary persistence.\n
4\n
[Navigate to header](https://docs.openclaw.ai/install/hetzner#)\n\nCreate persistent host directories\n\nDocker containers are ephemeral.\nAll long-lived state must live on the host.\n\n```\nmkdir -p /root/.openclaw/workspace\n\n# Set ownership to the container user (uid 1000):\nchown -R 1000:1000 /root/.openclaw\n```\n\n5\n\n[Navigate to header](https://docs.openclaw.ai/install/hetzner#)\n\nConfigure environment variables\n\nCreate `.env` in the repository root.\n\n```\nOPENCLAW_IMAGE=openclaw:latest\nOPENCLAW_GATEWAY_TOKEN=\nOPENCLAW_GATEWAY_BIND=lan\nOPENCLAW_GATEWAY_PORT=18789\n\nOPENCLAW_CONFIG_DIR=/root/.openclaw\nOPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace\n\nGOG_KEYRING_PASSWORD=\nXDG_CONFIG_HOME=/home/node/.openclaw\n```\n\nLeave `OPENCLAW_GATEWAY_TOKEN` blank unless you explicitly want to\nmanage it through `.env`; OpenClaw writes a random gateway token to\nconfig on first start. Generate a keyring password and paste it into\n`GOG_KEYRING_PASSWORD`:\n\n```\nopenssl rand -hex 32\n```\n\n**Do not commit this file.**This `.env` file is for container/runtime env such as `OPENCLAW_GATEWAY_TOKEN`.\nStored provider OAuth/API-key auth lives in the mounted\n`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`.\n
6\n
[Navigate to header](https://docs.openclaw.ai/install/hetzner#)\n\nDocker Compose configuration\n\nCreate or update `docker-compose.yml`.\n\n```\nservices:\n  openclaw-gateway:\n    image: ${OPENCLAW_IMAGE}\n    build: .\n    restart: unless-stopped\n    env_file:\n      - .env\n    environment:\n      - HOME=/home/node\n      - NODE_ENV=production\n      - TERM=xterm-256color\n      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}\n      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}\n      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}\n      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}\n      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}\n      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\n    volumes:\n      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw\n      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace\n    ports:\n      # Recommended: keep the Gateway loopback-only on the VPS; access via SSH tunnel.\n      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.\n      - \"127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789\"\n    command:\n      [\\\n        \"node\",\\\n        \"dist/index.js\",\\\n        \"gateway\",\\\n        \"--bind\",\\\n        \"${OPENCLAW_GATEWAY_BIND}\",\\\n        \"--port\",\\\n        \"${OPENCLAW_GATEWAY_PORT}\",\\\n        \"--allow-unconfigured\",\\\n      ]\n```\n\n`--allow-unconfigured` is only for bootstrap convenience, it is not a replacement for a proper gateway configuration. Still set auth (`gateway.auth.token` or password) and use safe bind settings for your deployment.\n
7\n
[Navigate to header](https://docs.openclaw.ai/install/hetzner#)\n\nShared Docker VM runtime steps\n\nUse the shared runtime guide for the common Docker host flow:\n\n- [Bake required binaries into the image](https://docs.openclaw.ai/install/docker-vm-runtime#bake-required-binaries-into-the-image)\n- [Build and launch](https://docs.openclaw.ai/install/docker-vm-runtime#build-and-launch)\n- [What persists where](https://docs.openclaw.ai/install/docker-vm-runtime#what-persists-where)\n- [Updates](https://docs.openclaw.ai/install/docker-vm-runtime#updates)\n
8\n
[Navigate to header](https://docs.openclaw.ai/install/hetzner#)\n\nHetzner-specific access\n\nAfter the shared build and launch steps, tunnel from your laptop:\n\n```\nssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP\n```\n\nOpen:`http://127.0.0.1:18789/`Paste the configured shared secret. This guide uses the gateway token by\ndefault; if you switched to password auth, use that password instead.\n\nThe shared persistence map lives in [Docker VM Runtime](https://docs.openclaw.ai/install/docker-vm-runtime#what-persists-where).\n
## [​](https://docs.openclaw.ai/install/hetzner\\#infrastructure-as-code-terraform)  Infrastructure as Code (Terraform)\n\nFor teams preferring infrastructure-as-code workflows, a community-maintained Terraform setup provides:\n\n- Modular Terraform configuration with remote state management\n- Automated provisioning via cloud-init\n- Deployment scripts (bootstrap, deploy, backup/restore)\n- Security hardening (firewall, UFW, SSH-only access)\n- SSH tunnel configuration for gateway access\n
**Repositories:**\n\n- Infrastructure: [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)\n- Docker config: [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)\n
This approach complements the Docker setup above with reproducible deployments, version-controlled infrastructure, and automated disaster recovery.\n
Community-maintained. For issues or contributions, see the repository links above.\n
## [​](https://docs.openclaw.ai/install/hetzner\\#next-steps)  Next steps\n\n- Set up messaging channels: [Channels](https://docs.openclaw.ai/channels)\n- Configure the Gateway: [Gateway configuration](https://docs.openclaw.ai/gateway/configuration)\n- Keep OpenClaw up to date: [Updating](https://docs.openclaw.ai/install/updating)\n
## [​](https://docs.openclaw.ai/install/hetzner\\#related)  Related\n\n- [Install overview](https://docs.openclaw.ai/install)\n- [Fly.io](https://docs.openclaw.ai/install/fly)\n- [Docker](https://docs.openclaw.ai/install/docker)\n- [VPS hosting](https://docs.openclaw.ai/vps)\n
[GCP](https://docs.openclaw.ai/install/gcp) [Hostinger](https://docs.openclaw.ai/install/hostinger)\n
Ctrl+I

---

## DigitalOcean - OpenClaw
**Source:** https://docs.openclaw.ai/install/digitalocean

[Skip to main content](https://docs.openclaw.ai/install/digitalocean#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

DigitalOcean

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Prerequisites](https://docs.openclaw.ai/install/digitalocean#prerequisites)
- [Setup](https://docs.openclaw.ai/install/digitalocean#setup)
- [Troubleshooting](https://docs.openclaw.ai/install/digitalocean#troubleshooting)
- [Next steps](https://docs.openclaw.ai/install/digitalocean#next-steps)
- [Related](https://docs.openclaw.ai/install/digitalocean#related)

Run a persistent OpenClaw Gateway on a DigitalOcean Droplet.

## [​](https://docs.openclaw.ai/install/digitalocean\\#prerequisites) Prerequisites

- DigitalOcean account ( [signup](https://cloud.digitalocean.com/registrations/new))
- SSH key pair (or willingness to use password auth)
- About 20 minutes

## [​](https://docs.openclaw.ai/install/digitalocean\\#setup) Setup

1

[Navigate to header](https://docs.openclaw.ai/install/digitalocean#)

Create a Droplet

Use a clean base image (Ubuntu 24.04 LTS). Avoid third-party Marketplace 1-click images unless you have reviewed their startup scripts and firewall defaults.

1. Log into [DigitalOcean](https://cloud.digitalocean.com/).
2. Click **Create > Droplets**.
3. Choose:
   - **Region:** Closest to you
   - **Image:** Ubuntu 24.04 LTS
   - **Size:** Basic, Regular, 1 vCPU / 1 GB RAM / 25 GB SSD
   - **Authentication:** SSH key (recommended) or password
4. Click **Create Droplet** and note the IP address.

2

[Navigate to header](https://docs.openclaw.ai/install/digitalocean#)

Connect and install

```
ssh root@YOUR_DROPLET_IP

apt update && apt upgrade -y

# Install Node.js 24
curl -fsSL https://deb.nodesource.com/setup_24.x | bash -\napt install -y nodejs

# Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw --version
```

3

[Navigate to header](https://docs.openclaw.ai/install/digitalocean#)

Run onboarding

```
openclaw onboard --install-daemon
```

The wizard walks you through model auth, channel setup, gateway token generation, and daemon installation (systemd).

4

[Navigate to header](https://docs.openclaw.ai/install/digitalocean#)

Add swap (recommended for 1 GB Droplets)

```
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo 
'/swapfile none swap sw 0 0' >> /etc/fstab
```

5

[Navigate to header](https://docs.openclaw.ai/install/digitalocean#)

Verify the gateway

```
openclaw status
systemctl --user status openclaw-gateway.service
journalctl --user -u openclaw-gateway.service -f
```

6

[Navigate to header](https://docs.openclaw.ai/install/digitalocean#)

Access the Control UI

The gateway binds to loopback by default. Pick one of these options.**Option A: SSH tunnel (simplest)**

```
# From your local machine
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP
```

Then open `http://localhost:18789`.**Option B: Tailscale Serve**

```
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Then open `https://<magicdns>/` from any device on your tailnet.**Option C: Tailnet bind (no Serve)**

```
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Then open `http://<tailscale-ip>:18789` (token required).

## [​](https://docs.openclaw.ai/install/digitalocean\\#troubleshooting) Troubleshooting

**Gateway will not start** — Run `openclaw doctor --non-interactive` and check logs with `journalctl --user -u openclaw-gateway.service -n 50`.**Port already in use** — Run `lsof -i :18789` to find the process, then stop it.**Out of memory** — Verify swap is active with `free -h`. If still hitting OOM, use API-based models (Claude, GPT) rather than local models, or upgrade to a 2 GB Droplet.

## [​](https://openclaw.ai/install/digitalocean\\#next-steps) Next steps

- [Channels](https://docs.openclaw.ai/channels) — connect Telegram, WhatsApp, Discord, and more
- [Gateway configuration](https://docs.openclaw.ai/gateway/configuration) — all config options
- [Updating](https://docs.openclaw.ai/install/updating) — keep OpenClaw up to date

## [​](https://docs.openclaw.ai/install/digitalocean\\#related) Related

- [Install overview](https://docs.openclaw.ai/install)
- [Fly.io](https://docs.openclaw.ai/install/fly)
- [Hetzner](https://docs.openclaw.ai/install/hetzner)
- [VPS hosting](https://docs.openclaw.ai/vps)

[Azure](https://docs.openclaw.ai/install/azure) [Docker VM runtime](https://docs.openclaw.ai/install/docker-vm-runtime)

Ctrl+I

---

## macOS VMs - OpenClaw
**Source:** https://docs.openclaw.ai/install/macos-vm

[Skip to main content](https://docs.openclaw.ai/install/macos-vm#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

macOS VMs

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [OpenClaw on macOS VMs (Sandboxing)](https://docs.openclaw.ai/install/macos-vm#openclaw-on-macos-vms-sandboxing)
- [Recommended default (most users)](https://docs.openclaw.ai/install/macos-vm#recommended-default-most-users)
- [macOS VM options](https://docs.openclaw.ai/install/macos-vm#macos-vm-options)
- [Local VM on your Apple Silicon Mac (Lume)](https://docs.openclaw.ai/install/macos-vm#local-vm-on-your-apple-silicon-mac-lume)
- [Hosted Mac providers (cloud)](https://docs.openclaw.ai/install/macos-vm#hosted-mac-providers-cloud)
- [Quick path (Lume, experienced users)](https://docs.openclaw.ai/install/macos-vm#quick-path-lume-experienced-users)
- [What you need (Lume)](https://docs.openclaw.ai/install/macos-vm#what-you-need-lume)
- [1) Install Lume](https://docs.openclaw.ai/install/macos-vm#1-install-lume)
- [2) Create the macOS VM](https://docs.openclaw.ai/install/macos-vm#2-create-the-macos-vm)
- [3) Complete Setup Assistant](https://docs.openclaw.ai/install/macos-vm#3-complete-setup-assistant)
- [4) Get the VM IP address](https://docs.openclaw.ai/install/macos-vm#4-get-the-vm-ip-address)
- [5) SSH into the VM](https://docs.openclaw.ai/install/macos-vm#5-ssh-into-the-vm)
- [6) Install OpenClaw](https://docs.openclaw.ai/install/macos-vm#6-install-openclaw)
- [7) Configure channels](https://docs.openclaw.ai/install/macos-vm#7-configure-channels)
- [8) Run the VM headlessly](https://docs.openclaw.ai/install/macos-vm#8-run-the-vm-headlessly)
- [Bonus: iMessage integration](https://docs.openclaw.ai/install/macos-vm#bonus-imessage-integration)
- [Save a golden image](https://docs.openclaw.ai/install/macos-vm#save-a-golden-image)
- [Running 24/7](https://docs.openclaw.ai/install/macos-vm#running-24%2F7)
- [Troubleshooting](https://docs.openclaw.ai/install/macos-vm#troubleshooting)
- [Related docs](https://docs.openclaw.ai/install/macos-vm#related-docs)

# [​](https://docs.openclaw.ai/install/macos-vm\\#openclaw-on-macos-vms-sandboxing)  OpenClaw on macOS VMs (Sandboxing)

## [​](https://docs.openclaw.ai/install/macos-vm\\#recommended-default-most-users)  Recommended default (most users)

- **Small Linux VPS** for an always-on Gateway and low cost. See [VPS hosting](https://docs.openclaw.ai/vps).
- **Dedicated hardware** (Mac mini or Linux box) if you want full control and a **residential IP** for browser automation. Many sites block data center IPs, so local browsing often works better.
- **Hybrid:** keep the Gateway on a cheap VPS, and connect your Mac as a **node** when you need browser/UI automation. See [Nodes](https://docs.openclaw.ai/nodes) and [Gateway remote](https://docs.openclaw.ai/gateway/remote).

Use a macOS VM when you specifically need macOS-only capabilities (iMessage/BlueBubbles) or want strict isolation from your daily Mac.

## [​](https://docs.openclaw.ai/install/macos-vm\\#macos-vm-options)  macOS VM options

### [​](https://docs.openclaw.ai/install/macos-vm\\#local-vm-on-your-apple-silicon-mac-lume)  Local VM on your Apple Silicon Mac (Lume)

Run OpenClaw in a sandboxed macOS VM on your existing Apple Silicon Mac using [Lume](https://cua.ai/docs/lume).This gives you:

- Full macOS environment in isolation (your host stays clean)
- iMessage support via BlueBubbles (impossible on Linux/Windows)
- Instant reset by cloning VMs
- No extra hardware or cloud costs

### [​](https://docs.openclaw.ai/install/macos-vm\\#hosted-mac-providers-cloud)  Hosted Mac providers (cloud)

If you want macOS in the cloud, hosted Mac providers work too:

- [MacStadium](https://www.macstadium.com/) (hosted Macs)
- Other hosted Mac vendors also work; follow their VM + SSH docs

Once you have SSH access to a macOS VM, continue at step 6 below.

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#quick-path-lume-experienced-users)  Quick path (Lume, experienced users)

1. Install Lume
2. `lume create openclaw --os macos --ipsw latest`
3. Complete Setup Assistant, enable Remote Login (SSH)
4. `lume run openclaw --no-display`
5. SSH in, install OpenClaw, configure channels
6. Done

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#what-you-need-lume)  What you need (Lume)

- Apple Silicon Mac (M1/M2/M3/M4)
- macOS Sequoia or later on the host
- ~60 GB free disk space per VM
- ~20 minutes

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#1-install-lume)  1) Install Lume

```
/bin/bash -c \"$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)\"
```

If `~/.local/bin` isn’t in your PATH:

```
echo 'export PATH=\"$PATH:$HOME/.local/bin\"' >> ~/.zshrc && source ~/.zshrc
```

Verify:

```
lume --version
```

Docs: [Lume Installation](https://cua.ai/docs/lume/guide/getting-started/installation)

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#2-create-the-macos-vm)  2) Create the macOS VM

```
lume create openclaw --os macos --ipsw latest
```

This downloads macOS and creates the VM. A VNC window opens automatically.

The download can take a while depending on your connection.

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#3-complete-setup-assistant)  3) Complete Setup Assistant

In the VNC window:

1. Select language and region
2. Skip Apple ID (or sign in if you want iMessage later)
3. Create a user account (remember the username and password)
4. Skip all optional features

After setup completes, enable SSH:

1. Open System Settings → General → Sharing
2. Enable “Remote Login”

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#4-get-the-vm-ip-address)  4) Get the VM IP address

```
lume get openclaw
```

Look for the IP address (usually `192.168.64.x`).

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#5-ssh-into-the-vm)  5) SSH into the VM

```
ssh youruser@192.168.64.X
```

Replace `youruser` with the account you created, and the IP with your VM’s IP.

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#6-install-openclaw)  6) Install OpenClaw

Inside the VM:

```
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Follow the onboarding prompts to set up your model provider (Anthropic, OpenAI, etc.).

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#7-configure-channels)  7) Configure channels

Edit the config file:

```
nano ~/.openclaw/openclaw.json
```

Add your channels:

```
{
  channels: {
    whatsapp: {
      dmPolicy: \"allowlist\",
      allowFrom: [\"+15551234567\"],
    },
    telegram: {
      botToken: \"YOUR_BOT_TOKEN\",
    },
  },
}
```

Then login to WhatsApp (scan QR):

```
openclaw channels login
```

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#8-run-the-vm-headlessly)  8) Run the VM headlessly

Stop the VM and restart without display:

```
lume stop openclaw
lume run openclaw --no-display
```

The VM runs in the background. OpenClaw’s daemon keeps the gateway running.To check status:

```
ssh youruser@192.168.64.X \"openclaw status\"
```

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#bonus-imessage-integration)  Bonus: iMessage integration

This is the killer feature of running on macOS. Use [BlueBubbles](https://bluebubbles.app/) to add iMessage to OpenClaw.Inside the VM:

1. Download BlueBubbles from bluebubbles.app
2. Sign in with your Apple ID
3. Enable the Web API and set a password
4. Point BlueBubbles webhooks at your gateway (example: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`)

Add to your OpenClaw config:

```
{
  channels: {
    bluebubbles: {
      serverUrl: \"http://localhost:1234\",
      password: \"your-api-password\",
      webhookPath: \"/bluebubbles-webhook\",
    },
  },
}
```

Restart the gateway. Now your agent can send and receive iMessages.Full setup details: [BlueBubbles channel](https://docs.openclaw.ai/channels/bluebubbles)

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#save-a-golden-image)  Save a golden image

Before customizing further, snapshot your clean state:

```
lume stop openclaw
lume clone openclaw openclaw-golden
```

Reset anytime:

```
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#running-24/7)  Running 24/7

Keep the VM running by:

- Keeping your Mac plugged in
- Disabling sleep in System Settings → Energy Saver
- Using `caffeinate` if needed

For true always-on, consider a dedicated Mac mini or a small VPS. See [VPS hosting](https://docs.openclaw.ai/vps).

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#troubleshooting)  Troubleshooting

| Problem | Solution |
| --- | --- |
| Can’t SSH into VM | Check “Remote Login” is enabled in VM’s System Settings |
| VM IP not showing | Wait for VM to fully boot, run `lume get openclaw` again |
| Lume command not found | Add `~/.local/bin` to your PATH |
| WhatsApp QR not scanning | Ensure you’re logged into the VM (not host) when running `openclaw channels login` |

* * *

## [​](https://docs.openclaw.ai/install/macos-vm\\#related-docs)  Related docs

- [VPS hosting](https://docs.openclaw.ai/vps)
- [Nodes](https://docs.openclaw.ai/nodes)
- [Gateway remote](https://docs.openclaw.ai/gateway/remote)
- [BlueBubbles channel](https://docs.openclaw.ai/channels/bluebubbles)
- [Lume Quickstart](https://cua.ai/docs/lume/guide/getting-started/quickstart)
- [Lume CLI Reference](https://cua.ai/docs/lume/reference/cli-reference)
- [Unattended VM Setup](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (advanced)
- [Docker Sandboxing](https://docs.openclaw.ai/install/docker) (alternative isolation approach)

[Linux Server](https://docs.openclaw.ai/vps) [Northflank](https://docs.openclaw.ai/install/northflank)

Ctrl+I

---

## Ansible - OpenClaw
**Source:** https://docs.openclaw.ai/install/ansible

[Skip to main content](https://docs.openclaw.ai/install/ansible#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Containers

Ansible

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Ansible Installation](https://docs.openclaw.ai/install/ansible#ansible-installation)
- [Prerequisites](https://docs.openclaw.ai/install/ansible#prerequisites)
- [What you get](https://docs.openclaw.ai/install/ansible#what-you-get)
- [Quick start](https://docs.openclaw.ai/install/ansible#quick-start)
- [What gets installed](https://docs.openclaw.ai/install/ansible#what-gets-installed)
- [Post-Install Setup](https://docs.openclaw.ai/install/ansible#post-install-setup)
- [Quick commands](https://docs.openclaw.ai/install/ansible#quick-commands)
- [Security architecture](https://docs.openclaw.ai/install/ansible#security-architecture)
- [Manual installation](https://docs.openclaw.ai/install/ansible#manual-installation)
- [Updating](https://docs.openclaw.ai/install/ansible#updating)
- [Troubleshooting](https://docs.openclaw.ai/install/ansible#troubleshooting)
- [Advanced configuration](https://docs.openclaw.ai/install/ansible#advanced-configuration)
- [Related](https://docs.openclaw.ai/install/ansible#related)

# [​](https://docs.openclaw.ai/install/ansible\\#ansible-installation)  Ansible Installation

Deploy OpenClaw to production servers with **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — an automated installer with security-first architecture.

The [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) repo is the source of truth for Ansible deployment. This page is a quick overview.

## [​](https://docs.openclaw.ai/install/ansible\\#prerequisites)  Prerequisites

| Requirement | Details |
| --- | --- |
| **OS** | Debian 11+ or Ubuntu 20.04+ |
| **Access** | Root or sudo privileges |
| **Network** | Internet connection for package installation |
| **Ansible** | 2.14+ (installed automatically by the quick-start script) |

## [​](https://docs.openclaw.ai/install/ansible\\#what-you-get)  What you get

- **Firewall-first security** — UFW + Docker isolation (only SSH + Tailscale accessible)
- **Tailscale VPN** — secure remote access without exposing services publicly
- **Docker** — isolated sandbox containers, localhost-only bindings
- **Defense in depth** — 4-layer security architecture
- **Systemd integration** — auto-start on boot with hardening
- **One-command setup** — complete deployment in minutes

## [​](https://docs.openclaw.ai/install/ansible\\#quick-start)  Quick start

One-command install:

```
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

## [​](https://docs.openclaw.ai/install/ansible\\#what-gets-installed)  What gets installed

The Ansible playbook installs and configures:

1. **Tailscale** — mesh VPN for secure remote access
2. **UFW firewall** — SSH + Tailscale ports only
3. **Docker CE + Compose V2** — for the default agent sandbox backend
4. **Node.js 24 + pnpm** — runtime dependencies (Node 22 LTS, currently `22.14+`, remains supported)
5. **OpenClaw** — host-based, not containerized
6. **Systemd service** — auto-start with security hardening

The gateway runs directly on the host (not in Docker). Agent sandboxing is
optional; this playbook installs Docker because it is the default sandbox
backend. See [Sandboxing](https://docs.openclaw.ai/gateway/sandboxing) for details and other backends.

## [​](https://docs.openclaw.ai/install/ansible\\#post-install-setup)  Post-Install Setup

1

[Navigate to header](https://docs.openclaw.ai/install/ansible#)

Switch to the openclaw user

```
sudo -i -u openclaw
```

2

[Navigate to header](https://docs.openclaw.ai/install/ansible#)

Run the onboarding wizard

The post-install script guides you through configuring OpenClaw settings.

3

[Navigate to header](https://docs.openclaw.ai/install/ansible#)

Connect messaging providers

Log in to WhatsApp, Telegram, Discord, or Signal:

```
openclaw channels login
```

4

[Navigate to header](https://docs.openclaw.ai/install/ansible#)

Verify the installation

```
sudo systemctl status openclaw
sudo journalctl -u openclaw -f
```

5

[Navigate to header](https://docs.openclaw.ai/install/ansible#)

Connect to Tailscale

Join your VPN mesh for secure remote access.

### [​](https://docs.openclaw.ai/install/ansible\\#quick-commands)  Quick commands

```
# Check service status
sudo systemctl status openclaw

# View live logs
sudo journalctl -u openclaw -f

# Restart gateway
sudo systemctl restart openclaw

# Provider login (run as openclaw user)
sudo -i -u openclaw
openclaw channels login
```

## [​](https://docs.openclaw.ai/install/ansible\\#security-architecture)  Security architecture

The deployment uses a 4-layer defense model:

1. **Firewall (UFW)** — only SSH (22) + Tailscale (41641/udp) exposed publicly
2. **VPN (Tailscale)** — gateway accessible only via VPN mesh
3. **Docker isolation** — DOCKER-USER iptables chain prevents external port exposure
4. **Systemd hardening** — NoNewPrivileges, PrivateTmp, unprivileged user

To verify your external attack surface:

```
nmap -p- YOUR_SERVER_IP
```

Only port 22 (SSH) should be open. All other services (gateway, Docker) are locked down.Docker is installed for agent sandboxes (isolated tool execution), not for running the gateway itself. See [Multi-Agent Sandbox and Tools](https://docs.openclaw.ai/tools/multi-agent-sandbox-tools) for sandbox configuration.

## [​](https://docs.openclaw.ai/install/ansible\\#manual-installation)  Manual installation

If you prefer manual control over the automation:

1

[Navigate to header](https://docs.openclaw.ai/install/ansible#)

Install prerequisites

```
sudo apt update && sudo apt install -y ansible git
```

2

[Navigate to header](https://docs.openclaw.ai/install/ansible#)

Clone the repository

```
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible
```

3

[Navigate to header](https://docs.openclaw.ai/install/ansible#)

Install Ansible collections

```
ansible-galaxy collection install -r requirements.yml
```

4

[Navigate to header](https://docs.openclaw.ai/install/ansible#)

Run the playbook

```
./run-playbook.sh
```

Alternatively, run directly and then manually execute the setup script afterward:

```
ansible-playbook playbook.yml --ask-become-pass
# Then run: /tmp/openclaw-setup.sh
```

## [​](https://docs.openclaw.ai/install/ansible\\#updating)  Updating

The Ansible installer sets up OpenClaw for manual updates. See [Updating](https://docs.openclaw.ai/install/updating) for the standard update flow.To re-run the Ansible playbook (for example, for configuration changes):

```
cd openclaw-ansible
./run-playbook.sh
```

This is idempotent and safe to run multiple times.

## [​](https://docs.openclaw.ai/install/ansible\\#troubleshooting)  Troubleshooting

Firewall blocks my connection

- Ensure you can access via Tailscale VPN first
- SSH access (port 22) is always allowed
- The gateway is only accessible via Tailscale by design

Service will not start

```
# Check logs
sudo journalctl -u openclaw -n 100

# Verify permissions
sudo ls -la /opt/openclaw

# Test manual start
sudo -i -u openclaw
cd ~/openclaw
openclaw gateway run
```

Docker sandbox issues

```
# Verify Docker is running
sudo systemctl status docker

# Check sandbox image
sudo docker images | grep openclaw-sandbox

# Build sandbox image if missing
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

Provider login fails

Make sure you are running as the `openclaw` user:

```
sudo -i -u openclaw
openclaw channels login
```

## [​](https://docs.openclaw.ai/install/ansible\\#advanced-configuration)  Advanced configuration

For detailed security architecture and troubleshooting, see the openclaw-ansible repo:

- [Security Architecture](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [Technical Details](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [Troubleshooting Guide](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## [​](https://docs.openclaw.ai/install/ansible\\#related)  Related

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — full deployment guide
- [Docker](https://docs.openclaw.ai/install/docker) — containerized gateway setup
- [Sandboxing](https://docs.openclaw.ai/gateway/sandboxing) — agent sandbox configuration
- [Multi-Agent Sandbox and Tools](https://docs.openclaw.ai/tools/multi-agent-sandbox-tools) — per-agent isolation

[Node.js](https://docs.openclaw.ai/install/node) [Bun (experimental)](https://docs.openclaw.ai/install/bun)

Ctrl+I

---

## Railway - OpenClaw
**Source:** https://docs.openclaw.ai/install/railway

[Skip to main content](https://docs.openclaw.ai/install/railway#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nHosting\n\nRailway\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Railway](https://docs.openclaw.ai/install/railway#railway)\n- [Quick checklist (new users)](https://docs.openclaw.ai/install/railway#quick-checklist-new-users)\n- [One-click deploy](https://docs.openclaw.ai/install/railway#one-click-deploy)\n- [What you get](https://docs.openclaw.ai/install/railway#what-you-get)\n- [Required Railway settings](https://docs.openclaw.ai/install/railway#required-railway-settings)\n- [Public Networking](https://docs.openclaw.ai/install/railway#public-networking)\n- [Volume (required)](https://docs.openclaw.ai/install/railway#volume-required)\n- [Variables](https://docs.openclaw.ai/install/railway#variables)\n- [Connect a channel](https://docs.openclaw.ai/install/railway#connect-a-channel)\n- [Backups & migration](https://docs.openclaw.ai/install/railway#backups-%26-migration)\n- [Next steps](https://docs.openclaw.ai/install/railway#next-steps)\n\n# [​](https://docs.openclaw.ai/install/railway\\#railway)  Railway\n\nDeploy OpenClaw on Railway with a one-click template and access it through the web Control UI.\nThis is the easiest “no terminal on the server” path: Railway runs the Gateway for you.\n\n## [​](https://docs.openclaw.ai/install/railway\\#quick-checklist-new-users)  Quick checklist (new users)\n\n1. Click **Deploy on Railway** (below).\n2. Add a **Volume** mounted at `/data`.\n3. Set the required **Variables** (at least `OPENCLAW_GATEWAY_PORT` and `OPENCLAW_GATEWAY_TOKEN`).\n4. Enable **HTTP Proxy** on port `8080`.\n5. Open `https://<your-railway-domain>/openclaw` and connect using the configured shared secret. This template uses `OPENCLAW_GATEWAY_TOKEN` by default; if you replace it with password auth, use that password instead.\n\n## [​](https://docs.openclaw.ai/install/railway\\#one-click-deploy)  One-click deploy\n\n[Deploy on Railway](https://railway.com/deploy/clawdbot-railway-template) After deploy, find your public URL in **Railway → your service → Settings → Domains**.Railway will either:\n\n- give you a generated domain (often `https://<something>.up.railway.app`), or\n- use your custom domain if you attached one.\n\nThen open:\n\n- `https://<your-railway-domain>/openclaw` — Control UI\n\n## [​](https://docs.openclaw.ai/install/railway\\#what-you-get)  What you get\n\n- Hosted OpenClaw Gateway + Control UI\n- Persistent storage via Railway Volume (`/data`) so `openclaw.json`,\nper-agent `auth-profiles.json`, channel/provider state, sessions, and\nworkspace survive redeploys\n\n## [​](https://docs.openclaw.ai/install/railway\\#required-railway-settings)  Required Railway settings\n\n### [​](https://docs.openclaw.ai/install/railway\\#public-networking)  Public Networking\n\nEnable **HTTP Proxy** for the service.\n\n- Port: `8080`\n\n### [​](https://docs.openclaw.ai/install/railway\\#volume-required)  Volume (required)\n\nAttach a volume mounted at:\n\n- `/data`\n\n### [​](https://docs.openclaw.ai/install/railway\\#variables)  Variables\n\nSet these variables on the service:\n\n- `OPENCLAW_GATEWAY_PORT=8080` (required — must match the port in Public Networking)\n- `OPENCLAW_GATEWAY_TOKEN` (required; treat as an admin secret)\n- `OPENCLAW_STATE_DIR=/data/.openclaw` (recommended)\n- `OPENCLAW_WORKSPACE_DIR=/data/workspace` (recommended)\n\n## [​](https://docs.openclaw.ai/install/railway\\#connect-a-channel)  Connect a channel\n\nUse the Control UI at `/openclaw` or run `openclaw onboard` via Railway’s shell for channel setup instructions:\n\n- [Telegram](https://docs.openclaw.ai/channels/telegram) (fastest — just a bot token)\n- [Discord](https://docs.openclaw.ai/channels/discord)\n- [All channels](https://docs.openclaw.ai/channels)\n\n## [​](https://docs.openclaw.ai/install/railway\\#backups-&-migration)  Backups & migration\n\nExport your state, config, auth profiles, and workspace:\n\n```\nopenclaw backup create\n```\n\nThis creates a portable backup archive with OpenClaw state plus any configured\nworkspace. See [Backup](https://docs.openclaw.ai/cli/backup) for details.\n\n## [​](https://docs.openclaw.ai/install/railway\\#next-steps)  Next steps\n\n- Set up messaging channels: [Channels](https://docs.openclaw.ai/channels)\n- Configure the Gateway: [Gateway configuration](https://docs.openclaw.ai/gateway/configuration)\n- Keep OpenClaw up to date: [Updating](https://docs.openclaw.ai/install/updating)\n\n[Oracle Cloud](https://docs.openclaw.ai/install/oracle) [Raspberry Pi](https://docs.openclaw.ai/install/raspberry-pi)\n\nCtrl+I

---

## Updating - OpenClaw
**Source:** https://docs.openclaw.ai/install/updating

[Skip to main content](https://docs.openclaw.ai/install/updating#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Maintenance

Updating

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Recommended: openclaw update](https://docs.openclaw.ai/install/updating#recommended-openclaw-update)
- [Switch between npm and git installs](https://docs.openclaw.ai/install/updating#switch-between-npm-and-git-installs)
- [Alternative: re-run the installer](https://docs.openclaw.ai/install/updating#alternative-re-run-the-installer)
- [Alternative: manual npm, pnpm, or bun](https://docs.openclaw.ai/install/updating#alternative-manual-npm-pnpm-or-bun)
- [Advanced npm install topics](https://docs.openclaw.ai/install/updating#advanced-npm-install-topics)
- [Auto-updater](https://docs.openclaw.ai/install/updating#auto-updater)
- [After updating](https://docs.openclaw.ai/install/updating#after-updating)
- [Run doctor](https://docs.openclaw.ai/install/updating#run-doctor)
- [Restart the gateway](https://docs.openclaw.ai/install/updating#restart-the-gateway)
- [Verify](https://docs.openclaw.ai/install/updating#verify)
- [Rollback](https://docs.openclaw.ai/install/updating#rollback)
- [Pin a version (npm)](https://docs.openclaw.ai/install/updating#pin-a-version-npm)
- [Pin a commit (source)](https://docs.openclaw.ai/install/updating#pin-a-commit-source)
- [If you are stuck](https://docs.openclaw.ai/install/updating#if-you-are-stuck)
- [Related](https://docs.openclaw.ai/install/updating#related)

Keep OpenClaw up to date.

## [​](https://docs.openclaw.ai/install/updating\\#recommended-openclaw-update)  Recommended: `openclaw update`

The fastest way to update. It detects your install type (npm or git), fetches the latest version, runs `openclaw doctor`, and restarts the gateway.

```
openclaw update
```

To switch channels or target a specific version:

```
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag main
openclaw update --dry-run   # preview without applying
```

`--channel beta` prefers beta, but the runtime falls back to stable/latest when
the beta tag is missing or older than the latest stable release. Use `--tag beta`
if you want the raw npm beta dist-tag for a one-off package update.See [Development channels](https://docs.openclaw.ai/install/development-channels) for channel semantics.

## [​](https://docs.openclaw.ai/install/updating\\#switch-between-npm-and-git-installs)  Switch between npm and git installs

Use channels when you want to change the install type. The updater keeps your
state, config, credentials, and workspace in `~/.openclaw`; it only changes
which OpenClaw code install the CLI and gateway use.

```
# npm package install -> editable git checkout
openclaw update --channel dev

# git checkout -> npm package install
openclaw update --channel stable
```

Run with `--dry-run` first to preview the exact install-mode switch:

```
openclaw update --channel dev --dry-run
openclaw update --channel stable --dry-run
```

The `dev` channel ensures a git checkout, builds it, and installs the global CLI
from that checkout. The `stable` and `beta` channels use package installs. If the
gateway is already installed, `openclaw update` refreshes the service metadata
and restarts it unless you pass `--no-restart`.

## [​](https://docs.openclaw.ai/install/updating\\#alternative-re-run-the-installer)  Alternative: re-run the installer

```
curl -fsSL https://openclaw.ai/install.sh | bash
```

Add `--no-onboard` to skip onboarding. To force a specific install type through
the installer, pass `--install-method git --no-onboard` or
`--install-method npm --no-onboard`.If `openclaw update` fails after the npm package install phase, re-run the
installer. The installer does not call the old updater; it runs the global
package install directly and can recover a partially updated npm install.

```
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method npm
```

To pin the recovery to a specific version or dist-tag, add `--version`:

```
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method npm --version <version-or-dist-tag>
```

## [​](https://docs.openclaw.ai/install/updating\\#alternative-manual-npm-pnpm-or-bun)  Alternative: manual npm, pnpm, or bun

```
npm i -g openclaw@latest
```

When `openclaw update` manages a global npm install, it installs the target into
a temporary npm prefix first, verifies the packaged `dist` inventory, then swaps
the clean package tree into the real global prefix. That avoids npm overlaying a
new package onto stale files from the old package. If the install command fails,
OpenClaw retries once with `--omit=optional`. That retry helps hosts where native
optional dependencies cannot compile, while keeping the original failure visible
if the fallback also fails.

```
pnpm add -g openclaw@latest
```

```
bun add -g openclaw@latest
```

### [​](https://docs.openclaw.ai/install/updating\\#advanced-npm-install-topics)  Advanced npm install topics

Read-only package tree

OpenClaw treats packaged global installs as read-only at runtime, even when the global package directory is writable by the current user. Bundled plugin runtime dependencies are staged into a writable runtime directory instead of mutating the package tree. This keeps `openclaw update` from racing with a running gateway or local agent that is repairing plugin dependencies during the same install.Some Linux npm setups install global packages under root-owned directories such as `/usr/lib/node_modules/openclaw`. OpenClaw supports that layout through the same external staging path.

Hardened systemd units

Set a writable stage directory that is included in `ReadWritePaths`:

```
Environment=OPENCLAW_PLUGIN_STAGE_DIR=/var/lib/openclaw/plugin-runtime-deps
ReadWritePaths=/var/lib/openclaw /home/openclaw/.openclaw /tmp
```

`OPENCLAW_PLUGIN_STAGE_DIR` also accepts a path list. OpenClaw resolves bundled plugin runtime dependencies left-to-right across the listed roots, treats earlier roots as read-only preinstalled layers, and installs or repairs only into the final writable root:

```
Environment=OPENCLAW_PLUGIN_STAGE_DIR=/opt/openclaw/plugin-runtime-deps:/var/lib/openclaw/plugin-runtime-deps
ReadWritePaths=/var/lib/openclaw /home/openclaw/.openclaw /tmp
```

If `OPENCLAW_PLUGIN_STAGE_DIR` is not set, OpenClaw uses `$STATE_DIRECTORY` when systemd provides it, then falls back to `~/.openclaw/plugin-runtime-deps`. The repair step treats that stage as an OpenClaw-owned local package root and ignores user npm prefix and global settings, so global-install npm config does not redirect bundled plugin dependencies into `~/node_modules` or the global package tree.

Disk-space preflight

Before package updates and bundled runtime-dependency repairs, OpenClaw tries a best-effort disk-space check for the target volume. Low space produces a warning with the checked path, but does not block the update because filesystem quotas, snapshots, and network volumes can change after the check. The actual npm install, copy, and post-install verification remain authoritative.

Bundled plugin runtime dependencies

Packaged installs keep bundled plugin runtime dependencies out of the read-only package tree. On startup and during `openclaw doctor --fix`, OpenClaw repairs runtime dependencies only for bundled plugins that are active in config, active through legacy channel config, or enabled by their bundled manifest default. Persisted channel auth state alone does not trigger Gateway startup runtime-dependency repair.Explicit disablement wins. A disabled plugin or channel does not get its runtime dependencies repaired just because it exists in the package. External plugins and custom load paths still use `openclaw plugins install` or `openclaw plugins update`.

## [​](https://docs.openclaw.ai/install/updating\\#auto-updater)  Auto-updater

The auto-updater is off by default. Enable it in `~/.openclaw/openclaw.json`:

```
{
  update: {
    channel: \"stable\",
    auto: {
      enabled: true,
      stableDelayHours: 6,
      stableJitterHours: 12,
      betaCheckIntervalHours: 1,
    },
  },
}
```

| Channel | Behavior |
| --- | --- |
| `stable` | Waits `stableDelayHours`, then applies with deterministic jitter across `stableJitterHours` (spread rollout). |
| `beta` | Checks every `betaCheckIntervalHours` (default: hourly) and applies immediately. |
| `dev` | No automatic apply. Use `openclaw update` manually. |

The gateway also logs an update hint on startup (disable with `update.checkOnStart: false`).
For downgrade or incident recovery, set `OPENCLAW_NO_AUTO_UPDATE=1` in the gateway environment to block automatic applies even when `update.auto.enabled` is configured. Startup update hints can still run unless `update.checkOnStart` is also disabled.

## [​](https://docs.openclaw.ai/install/updating\\#after-updating)  After updating

1

[Navigate to header](https://docs.openclaw.ai/install/updating#run-doctor)

Run doctor

2

[Navigate to header](https://docs.openclaw.ai/install/updating#)

```
openclaw doctor
```

3

[Navigate to header](https://docs.openclaw.ai/install/updating#)

Migrates config, audits DM policies, and checks gateway health. Details: [Doctor](https://docs.openclaw.ai/gateway/doctor)

4

[Navigate to header](https://docs.openclaw.ai/install/updating#restart-the-gateway)

Restart the gateway

5

[Navigate to header](https://docs.openclaw.ai/install/updating#)

```
openclaw gateway restart
```

6

[Navigate to header](https://docs.openclaw.ai/install/updating#verify)

Verify

7

[Navigate to header](https://docs.openclaw.ai/install/updating#)

```
openclaw health
```

## [​](https://docs.openclaw.ai/install/updating\\#rollback)  Rollback

### [​](https://docs.openclaw.ai/install/updating\\#pin-a-version-npm)  Pin a version (npm)

```
npm i -g openclaw@<version>
openclaw doctor
openclaw gateway restart
```

`npm view openclaw version` shows the current published version.

### [​](https://docs.openclaw.ai/install/updating\\#pin-a-commit-source)  Pin a commit (source)

```
git fetch origin
git checkout \"$(git rev-list -n 1 --before=\\\"2026-01-01\\\" origin/main)\"
pnpm install && pnpm build
openclaw gateway restart
```

To return to latest: `git checkout main && git pull`.

## [​](https://docs.openclaw.ai/install/updating\\#if-you-are-stuck)  If you are stuck

- Run `openclaw doctor` again and read the output carefully.
- For `openclaw update --channel dev` on source checkouts, the updater auto-bootstraps `pnpm` when needed. If you see a pnpm/corepack bootstrap error, install `pnpm` manually (or re-enable `corepack`) and rerun the update.
- Check: [Troubleshooting](https://docs.openclaw.ai/gateway/troubleshooting)
- Ask in Discord: [https://discord.gg/clawd](https://discord.gg/clawd)

## [​](https://docs.openclaw.ai/install/updating\\#related)  Related

- [Install overview](https://docs.openclaw.ai/install): all installation methods.
- [Doctor](https://docs.openclaw.ai/gateway/doctor): health checks after updates.
- [Migrating](https://docs.openclaw.ai/install/migrating): major version migration guides.

[Node.js](https://docs.openclaw.ai/install/node) [Migration guide](https://docs.openclaw.ai/install/migrating)

---

## Docker VM runtime - OpenClaw
**Source:** https://docs.openclaw.ai/install/docker-vm-runtime

[Skip to main content](https://docs.openclaw.ai/install/docker-vm-runtime#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

Docker VM runtime

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Bake required binaries into the image](https://docs.openclaw.ai/install/docker-vm-runtime#bake-required-binaries-into-the-image)
- [Build and launch](https://docs.openclaw.ai/install/docker-vm-runtime#build-and-launch)
- [What persists where](https://docs.openclaw.ai/install/docker-vm-runtime#what-persists-where)
- [Updates](https://docs.openclaw.ai/install/docker-vm-runtime#updates)
- [Related](https://docs.openclaw.ai/install/docker-vm-runtime#related)

Shared runtime steps for VM-based Docker installs such as GCP, Hetzner, and similar VPS providers.

## [​](https://docs.openclaw.ai/install/docker-vm-runtime\#bake-required-binaries-into-the-image)  Bake required binaries into the image

Installing binaries inside a running container is a trap.
Anything installed at runtime will be lost on restart.All external binaries required by skills must be installed at image build time.The examples below show three common binaries only:

- `gog` for Gmail access
- `goplaces` for Google Places
- `wacli` for WhatsApp

These are examples, not a complete list.
You may install as many binaries as needed using the same pattern.If you add new skills later that depend on additional binaries, you must:

1. Update the Dockerfile
2. Rebuild the image
3. Restart the containers

**Example Dockerfile**

```
FROM node:24-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Example binary 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \\
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Example binary 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \\
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Example binary 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \\
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Add more binaries below using the same pattern

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD [\"node\",\"dist/index.js\"]
```

The download URLs above are for x86\_64 (amd64). For ARM-based VMs (e.g. Hetzner ARM, GCP Tau T2A), replace the download URLs with the appropriate ARM64 variants from each tool’s release page.

## [​](https://docs.openclaw.ai/install/docker-vm-runtime\#build-and-launch)  Build and launch

```
docker compose build
docker compose up -d openclaw-gateway
```

If build fails with `Killed` or `exit code 137` during `pnpm install --frozen-lockfile`, the VM is out of memory.
Use a larger machine class before retrying.Verify binaries:

```
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Expected output:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

Verify Gateway:

```
docker compose logs -f openclaw-gateway
```

Expected output:

```
[gateway] listening on ws://0.0.0.0:18789
```

## [​](https://docs.openclaw.ai/install/docker-vm-runtime\#what-persists-where)  What persists where

OpenClaw runs in Docker, but Docker is not the source of truth.
All long-lived state must survive restarts, rebuilds, and reboots.

| Component | Location | Persistence mechanism | Notes |
| --- | --- | --- | --- |
| Gateway config | `/home/node/.openclaw/` | Host volume mount | Includes `openclaw.json`, `.env` |
| Model auth profiles | `/home/node/.openclaw/agents/` | Host volume mount | `agents/<agentId>/agent/auth-profiles.json` (OAuth, API keys) |
| Skill configs | `/home/node/.openclaw/skills/` | Host volume mount | Skill-level state |
| Agent workspace | `/home/node/.openclaw/workspace/` | Host volume mount | Code and agent artifacts |
| WhatsApp session | `/home/node/.openclaw/` | Host volume mount | Preserves QR login |
| Gmail keyring | `/home/node/.openclaw/` | Host volume + password | Requires `GOG_KEYRING_PASSWORD` |
| External binaries | `/usr/local/bin/` | Docker image | Must be baked at build time |
| Node runtime | Container filesystem | Docker image | Rebuilt every image build |
| OS packages | Container filesystem | Docker image | Do not install at runtime |
| Docker container | Ephemeral | Restartable | Safe to destroy |

## [​](https://docs.openclaw.ai/install/docker-vm-runtime\#updates)  Updates

To update OpenClaw on the VM:

```
git pull
docker compose build
docker compose up -d
```

## [​](https://docs.openclaw.ai/install/docker-vm-runtime\#related)  Related

- [Docker](https://docs.openclaw.ai/install/docker)
- [Podman](https://docs.openclaw.ai/install/podman)
- [ClawDock](https://docs.openclaw.ai/install/clawdock)

[DigitalOcean](https://docs.openclaw.ai/install/digitalocean) [exe.dev](https://docs.openclaw.ai/install/exe-dev)

Ctrl+I

---

## Hostinger - OpenClaw
**Source:** https://docs.openclaw.ai/install/hostinger

[Skip to main content](https://docs.openclaw.ai/install/hostinger#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

Hostinger

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Prerequisites](https://docs.openclaw.ai/install/hostinger#prerequisites)
- [Option A: 1-Click OpenClaw](https://docs.openclaw.ai/install/hostinger#option-a-1-click-openclaw)
- [Option B: OpenClaw on VPS](https://docs.openclaw.ai/install/hostinger#option-b-openclaw-on-vps)
- [Verify your setup](https://docs.openclaw.ai/install/hostinger#verify-your-setup)
- [Troubleshooting](https://docs.openclaw.ai/install/hostinger#troubleshooting)
- [Next steps](https://docs.openclaw.ai/install/hostinger#next-steps)
- [Related](https://docs.openclaw.ai/install/hostinger#related)

Run a persistent OpenClaw Gateway on [Hostinger](https://www.hostinger.com/openclaw) via a **1-Click** managed deployment or a **VPS** install.

## [​](https://docs.openclaw.ai/install/hostinger\\#prerequisites)  Prerequisites

- Hostinger account ( [signup](https://www.hostinger.com/openclaw))
- About 5-10 minutes

## [​](https://docs.openclaw.ai/install/hostinger\\#option-a-1-click-openclaw)  Option A: 1-Click OpenClaw

The fastest way to get started. Hostinger handles infrastructure, Docker, and automatic updates.

1

[Navigate to header](https://docs.openclaw.ai/install/hostinger#)

Purchase and launch

1. From the [Hostinger OpenClaw page](https://www.hostinger.com/openclaw), choose a Managed OpenClaw plan and complete checkout.

During checkout you can select **Ready-to-Use AI** credits that are pre-purchased and integrated instantly inside OpenClaw — no external accounts or API keys from other providers needed. You can start chatting right away. Alternatively, provide your own key from Anthropic, OpenAI, Google Gemini, or xAI during setup.

2

[Navigate to header](https://docs.openclaw.ai/install/hostinger#)

Select a messaging channel

Choose one or more channels to connect:

- **WhatsApp** — scan the QR code shown in the setup wizard.
- **Telegram** — paste the bot token from [BotFather](https://t.me/BotFather).

3

[Navigate to header](https://docs.openclaw.ai/install/hostinger#)

Complete installation

Click **Finish** to deploy the instance. Once ready, access the OpenClaw dashboard from **OpenClaw Overview** in hPanel.

## [​](https://docs.openclaw.ai/install/hostinger\\#option-b-openclaw-on-vps)  Option B: OpenClaw on VPS

More control over your server. Hostinger deploys OpenClaw via Docker on your VPS and you manage it through the **Docker Manager** in hPanel.

1

[Navigate to header](https://docs.openclaw.ai/install/hostinger#)

Purchase a VPS

1. From the [Hostinger OpenClaw page](https://www.hostinger.com/openclaw), choose an OpenClaw on VPS plan and complete checkout.

You can select **Ready-to-Use AI** credits during checkout — these are pre-purchased and integrated instantly inside OpenClaw, so you can start chatting without any external accounts or API keys from other providers.

2

[Navigate to header](https://docs.openclaw.ai/install/hostinger#)

Configure OpenClaw

Once the VPS is provisioned, fill in the configuration fields:

- **Gateway token** — auto-generated; save it for later use.
- **WhatsApp number** — your number with country code (optional).
- **Telegram bot token** — from [BotFather](https://t.me/BotFather) (optional).
- **API keys** — only needed if you did not select Ready-to-Use AI credits during checkout.

3

[Navigate to header](https://docs.openclaw.ai/install/hostinger#)

Start OpenClaw

Click **Deploy**. Once running, open the OpenClaw dashboard from the hPanel by clicking on **Open**.

Logs, restarts, and updates are managed directly from the Docker Manager interface in hPanel. To update, press on **Update** in Docker Manager and that will pull the latest image.

## [​](https://docs.openclaw.ai/install/hostinger\\#verify-your-setup)  Verify your setup

Send “Hi” to your assistant on the channel you connected. OpenClaw will reply and walk you through initial preferences.

## [​](https://docs.openclaw.ai/install/hostinger\\#troubleshooting)  Troubleshooting

**Dashboard not loading** — Wait a few minutes for the container to finish provisioning. Check the Docker Manager logs in hPanel.**Docker container keeps restarting** — Open Docker Manager logs and look for configuration errors (missing tokens, invalid API keys).**Telegram bot not responding** — Send your pairing code message from Telegram directly as a message inside your OpenClaw chat to complete the connection.

## [​](https://docs.openclaw.ai/install/hostinger\\#next-steps)  Next steps

- [Channels](https://docs.openclaw.ai/channels) — connect Telegram, WhatsApp, Discord, and more
- [Gateway configuration](https://docs.openclaw.ai/gateway/configuration) — all config options

## [​](https://docs.openclaw.ai/install/hostinger\\#related)  Related

- [Install overview](https://docs.openclaw.ai/install)
- [VPS hosting](https://docs.openclaw.ai/vps)
- [DigitalOcean](https://docs.openclaw.ai/install/digitalocean)

[Hetzner](https://docs.openclaw.ai/install/hetzner) [Kubernetes](https://docs.openclaw.ai/install/kubernetes)

Ctrl+I

---

## Azure - OpenClaw
**Source:** https://docs.openclaw.ai/install/azure

[Skip to main content](https://docs.openclaw.ai/install/azure#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Hosting

Azure

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [OpenClaw on Azure Linux VM](https://docs.openclaw.ai/install/azure#openclaw-on-azure-linux-vm)
- [What you will do](https://docs.openclaw.ai/install/azure#what-you-will-do)
- [What you need](https://docs.openclaw.ai/install/azure#what-you-need)
- [Configure deployment](https://docs.openclaw.ai/install/azure#configure-deployment)
- [Deploy Azure resources](https://docs.openclaw.ai/install/azure#deploy-azure-resources)
- [Install OpenClaw](https://docs.openclaw.ai/install/azure#install-openclaw)
- [Cost considerations](https://docs.openclaw.ai/install/azure#cost-considerations)
- [Cleanup](https://docs.openclaw.ai/install/azure#cleanup)
- [Next steps](https://docs.openclaw.ai/install/azure#next-steps)
- [Related](https://docs.openclaw.ai/install/azure#related)

# [​](https://docs.openclaw.ai/install/azure\\#openclaw-on-azure-linux-vm)  OpenClaw on Azure Linux VM

This guide sets up an Azure Linux VM with the Azure CLI, applies Network Security Group (NSG) hardening, configures Azure Bastion for SSH access, and installs OpenClaw.

## [​](https://docs.openclaw.ai/install/azure\\#what-you-will-do)  What you will do

- Create Azure networking (VNet, subnets, NSG) and compute resources with the Azure CLI
- Apply Network Security Group rules so VM SSH is allowed only from Azure Bastion
- Use Azure Bastion for SSH access (no public IP on the VM)
- Install OpenClaw with the installer script
- Verify the Gateway

## [​](https://docs.openclaw.ai/install/azure\\#what-you-need)  What you need

- An Azure subscription with permission to create compute and network resources
- Azure CLI installed (see [Azure CLI install steps](https://learn.microsoft.com/cli/azure/install-azure-cli) if needed)
- An SSH key pair (the guide covers generating one if needed)
- ~20-30 minutes

## [​](https://docs.openclaw.ai/install/azure\\#configure-deployment)  Configure deployment

1

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Sign in to Azure CLI

```
az login
az extension add -n ssh
```

The `ssh` extension is required for Azure Bastion native SSH tunneling.

2

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Register required resource providers (one-time)

```
az provider register --namespace Microsoft.Compute
az provider register --namespace Microsoft.Network
```

Verify registration. Wait until both show `Registered`.

```
az provider show --namespace Microsoft.Compute --query registrationState -o tsv
az provider show --namespace Microsoft.Network --query registrationState -o tsv
```

3

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Set deployment variables

```
RG=\"rg-openclaw\"
LOCATION=\"westus2\"
VNET_NAME=\"vnet-openclaw\"
VNET_PREFIX=\"10.40.0.0/16\"
VM_SUBNET_NAME=\"snet-openclaw-vm\"
VM_SUBNET_PREFIX=\"10.40.2.0/24\"
BASTION_SUBNET_PREFIX=\"10.40.1.0/26\"
NSG_NAME=\"nsg-openclaw-vm\"
VM_NAME=\"vm-openclaw\"
ADMIN_USERNAME=\"openclaw\"
BASTION_NAME=\"bas-openclaw\"
BASTION_PIP_NAME=\"pip-openclaw-bastion\"
```

Adjust names and CIDR ranges to fit your environment. The Bastion subnet must be at least `/26`.

4

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Select SSH key

Use your existing public key if you have one:

```
SSH_PUB_KEY=\"$(cat ~/.ssh/id_ed25519.pub)\"
```

If you don’t have an SSH key yet, generate one:

```
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519 -C \"you@example.com\"
SSH_PUB_KEY=\"$(cat ~/.ssh/id_ed25519.pub)\"
```

5

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Select VM size and OS disk size

```
VM_SIZE=\"Standard_B2as_v2\"
OS_DISK_SIZE_GB=64
```

Choose a VM size and OS disk size available in your subscription and region:

- Start smaller for light usage and scale up later
- Use more vCPU/RAM/disk for heavier automation, more channels, or larger model/tool workloads
- If a VM size is unavailable in your region or subscription quota, pick the closest available SKU

List VM sizes available in your target region:

```
az vm list-skus --location \"${LOCATION}\" --resource-type virtualMachines -o table
```

Check your current vCPU and disk usage/quota:

```
az vm list-usage --location \"${LOCATION}\" -o table
```

## [​](https://docs.openclaw.ai/install/azure\\#deploy-azure-resources)  Deploy Azure resources

1

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Create the resource group

```
az group create -n \"${RG}\" -l \"${LOCATION}\"
```

2

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Create the network security group

Create the NSG and add rules so only the Bastion subnet can SSH into the VM.

```
az network nsg create \\
  -g \"${RG}\" -n \"${NSG_NAME}\" -l \"${LOCATION}\"

# Allow SSH from the Bastion subnet only
az network nsg rule create \\
  -g \"${RG}\" --nsg-name \"${NSG_NAME}\" \\
  -n AllowSshFromBastionSubnet --priority 100 \\
  --access Allow --direction Inbound --protocol Tcp \\
  --source-address-prefixes \"${BASTION_SUBNET_PREFIX}\" \\
  --destination-port-ranges 22

# Deny SSH from the public internet
az network nsg rule create \\
  -g \"${RG}\" --nsg-name \"${NSG_NAME}\" \\
  -n DenyInternetSsh --priority 110 \\
  --access Deny --direction Inbound --protocol Tcp \\
  --source-address-prefixes Internet \\
  --destination-port-ranges 22

# Deny SSH from other VNet sources
az network nsg rule create \\
  -g \"${RG}\" --nsg-name \"${NSG_NAME}\" \\
  -n DenyVnetSsh --priority 120 \\
  --access Deny --direction Inbound --protocol Tcp \\
  --source-address-prefixes VirtualNetwork \\
  --destination-port-ranges 22
```

The rules are evaluated by priority (lowest number first): Bastion traffic is allowed at 100, then all other SSH is blocked at 110 and 120.

3

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Create the virtual network and subnets

Create the VNet with the VM subnet (NSG attached), then add the Bastion subnet.

```
az network vnet create \\
  -g \"${RG}\" -n \"${VNET_NAME}\" -l \"${LOCATION}\" \\
  --address-prefixes \"${VNET_PREFIX}\" \\
  --subnet-name \"${VM_SUBNET_NAME}\" \\
  --subnet-prefixes \"${VM_SUBNET_PREFIX}\"

# Attach the NSG to the VM subnet
az network vnet subnet update \\
  -g \"${RG}\" --vnet-name \"${VNET_NAME}\" \\
  -n \"${VM_SUBNET_NAME}\" --nsg \"${NSG_NAME}\"

# AzureBastionSubnet — name is required by Azure
az network vnet subnet create \\
  -g \"${RG}\" --vnet-name \"${VNET_NAME}\" \\
  -n AzureBastionSubnet \\
  --address-prefixes \"${BASTION_SUBNET_PREFIX}\"
```

4

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Create the VM

The VM has no public IP. SSH access is exclusively through Azure Bastion.

```
az vm create \\
  -g \"${RG}\" -n \"${VM_NAME}\" -l \"${LOCATION}\" \\
  --image \"Canonical:ubuntu-24_04-lts:server:latest\" \\
  --size \"${VM_SIZE}\" \\
  --os-disk-size-gb \"${OS_DISK_SIZE_GB}\" \\
  --storage-sku StandardSSD_LRS \\
  --admin-username \"${ADMIN_USERNAME}\" \\
  --ssh-key-values \"${SSH_PUB_KEY}\" \\
  --vnet-name \"${VNET_NAME}\" \\
  --subnet \"${VM_SUBNET_NAME}\" \\
  --public-ip-address \"\" \\
  --nsg \"\"
```

`--public-ip-address \"\"` prevents a public IP from being assigned. `--nsg \"\"` skips creating a per-NIC NSG (the subnet-level NSG handles security).**Reproducibility:** The command above uses `latest` for the Ubuntu image. To pin a specific version, list available versions and replace `latest`:

```
az vm image list \\
  --publisher Canonical --offer ubuntu-24_04-lts \\
  --sku server --all -o table
```

Check your current vCPU and disk usage/quota:

```
az vm list-usage --location \"${LOCATION}\" -o table
```

5

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Create Azure Bastion

Azure Bastion provides managed SSH access to the VM without exposing a public IP. Standard SKU with tunneling is required for CLI-based `az network bastion ssh`.

```
az network public-ip create \\
  -g \"${RG}\" -n \"${BASTION_PIP_NAME}\" -l \"${LOCATION}\" \\
  --sku Standard --allocation-method Static

az network bastion create \\
  -g \"${RG}\" -n \"${BASTION_NAME}\" -l \"${LOCATION}\" \\
  --vnet-name \"${VNET_NAME}\" \\
  --public-ip-address \"${BASTION_PIP_NAME}\" \\
  --sku Standard --enable-tunneling true
```

Bastion provisioning typically takes 5-10 minutes but can take up to 15-30 minutes in some regions.

## [​](https://docs.openclaw.ai/install/azure\\#install-openclaw)  Install OpenClaw

1

[Navigate to header](https://docs.openclaw.ai/install/azure#)

SSH into the VM through Azure Bastion

```
VM_ID=\"$(az vm show -g \"${RG}\" -n \"${VM_NAME}\" --query id -o tsv)\"

az network bastion ssh \\
  --name \"${BASTION_NAME}\" \\
  --resource-group \"${RG}\" \\
  --target-resource-id \"${VM_ID}\" \\
  --auth-type ssh-key \\
  --username \"${ADMIN_USERNAME}\" \\
  --ssh-key ~/.ssh/id_ed25519
```

2

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Install OpenClaw (in the VM shell)

```
curl -fsSL https://openclaw.ai/install.sh -o /tmp/install.sh
bash /tmp/install.sh
rm -f /tmp/install.sh
```

The installer installs Node LTS and dependencies if not already present, installs OpenClaw, and launches the onboarding wizard. See [Install](https://docs.openclaw.ai/install) for details.

3

[Navigate to header](https://docs.openclaw.ai/install/azure#)

Verify the Gateway

After onboarding completes:

```
openclaw gateway status
```

Most enterprise Azure teams already have GitHub Copilot licenses. If that is your case, we recommend choosing the GitHub Copilot provider in the OpenClaw onboarding wizard. See [GitHub Copilot provider](https://docs.openclaw.ai/providers/github-copilot).

## [​](https://docs.openclaw.ai/install/azure\\#cost-considerations)  Cost considerations

Azure Bastion Standard SKU runs approximately **$140/month** and the VM (Standard\\_B2as\\_v2) runs approximately **$55/month**.To reduce costs:

- **Deallocate the VM** when not in use (stops compute billing; disk charges remain). The OpenClaw Gateway will not be reachable while the VM is deallocated — restart it when you need it live again:















```
az vm deallocate -g \"${RG}\" -n \"${VM_NAME}\"
az vm start -g \"${RG}\" -n \"${VM_NAME}\"   # restart later
```

- **Delete Bastion when not needed** and recreate it when you need SSH access. Bastion is the largest cost component and takes only a few minutes to provision.
- **Use the Basic Bastion SKU** (~$38/month) if you only need Portal-based SSH and don’t require CLI tunneling (`az network bastion ssh`).

## [​](https://docs.openclaw.ai/install/azure\\#cleanup)  Cleanup

To delete all resources created by this guide:

```
az group delete -n \"${RG}\" --yes --no-wait
```

This removes the resource group and everything inside it (VM, VNet, NSG, Bastion, public IP).

## [​](https://docs.openclaw.ai/install/azure\\#next-steps)  Next steps

- Set up messaging channels: [Channels](https://docs.openclaw.ai/channels)
- Pair local devices as nodes: [Nodes](https://docs.openclaw.ai/nodes)
- Configure the Gateway: [Gateway configuration](https://docs.openclaw.ai/gateway/configuration)
- For more details on OpenClaw Azure deployment with the GitHub Copilot model provider: [OpenClaw on Azure with GitHub Copilot](https://github.com/johnsonshi/openclaw-azure-github-copilot)

## [​](https://docs.openclaw.ai/install/azure\\#related)  Related

- [Install overview](https://docs.openclaw.ai/install)
- [GCP](https://docs.openclaw.ai/install/gcp)
- [DigitalOcean](https://docs.openclaw.ai/install/digitalocean)

[Podman](https://docs.openclaw.ai/install/podman) [DigitalOcean](https://docs.openclaw.ai/install/digitalocean)

Ctrl+I

---

