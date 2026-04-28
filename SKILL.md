---
name: openclaw-expert
description: Comprehensive expert knowledge base for OpenClaw. Use for configuring OpenClaw gateways, developing channel plugins, setting up agents, managing tools, troubleshooting ops, and understanding OpenClaw architecture.
---

# OpenClaw Expert Skill

This skill provides comprehensive documentation and expert knowledge for working with OpenClaw, an open-source framework for running autonomous, agentic AI systems.

## When to Use This Skill

Use this skill whenever you need to:
- Configure or deploy an OpenClaw Gateway
- Build or configure Channel Plugins (WhatsApp, Telegram, Slack, etc.)
- Set up and manage Agents and Multi-Agent routing
- Develop or integrate Tools and Plugins
- Troubleshoot OpenClaw operations (CLI, memory, security, telemetry)
- Understand OpenClaw's core concepts (Delegate architecture, QMD memory, Dreaming)

## Reference Documentation

The complete OpenClaw documentation has been scraped and organized into the following reference files. Read the relevant file based on the user's request:

### 1. Core Concepts (`references/concepts.md`)
Read this file to understand OpenClaw's architecture and core mechanisms.
- **Topics covered:** Delegate architecture, Agent runtime, Multi-agent routing, QMD memory engine, Memory search, Compaction, Dreaming, Command queue, Retry policy, Session management, System prompts, and Model providers.

### 2. Gateway & Operations (`references/gateway.md`)
Read this file for configuring and running the OpenClaw Gateway.
- **Topics covered:** Configuration reference, Secrets management, Security audit checks, Multiple gateways, Heartbeat, Health checks, Prometheus metrics, OpenTelemetry export, Bridge protocol, and Logging.

### 3. Channels (`references/channels.md`)
Read this file when configuring messaging platforms and channel integrations.
- **Topics covered:** WhatsApp, WeChat, IRC, Nostr, QQ bot, Broadcast groups, Location parsing, and Channel troubleshooting.

### 4. Agents & CLI (`references/cli.md`)
Read this file for managing agents and using the OpenClaw CLI.
- **Topics covered:** Agent configuration, CLI commands (agent, memory, browser, models, dashboard, status, completion, update), and TUI.

### 5. Tools (`references/tools.md`)
Read this file when working with built-in tools or creating new ones.
- **Topics covered:** Web search, Firecrawl, Browser login, PDF tool, Music generation, Reactions, Slash commands, LLM tasks, Exec approvals, Trajectory bundles, Subagents, and Kimi search.

### 6. Plugins (`references/plugins.md`)
Read this file for developing and managing OpenClaw plugins.
- **Topics covered:** Plugin manifest, Plugin bundles, SDK channel plugins, Architecture internals, Webhooks, Voice calls, Memory wiki, and SDK testing.

### 7. Model Providers (`references/providers.md`)
Read this file for configuring LLM providers.
- **Topics covered:** OpenAI, Google (Gemini), Deepgram, LiteLLM, Fireworks, Qwen, and Chutes.

### 8. Installation (`references/install.md`)
Read this file for setup and deployment guides.
- **Topics covered:** Getting started, Onboarding wizard, Migration guide, Deployment (Fly.io, GCP, Northflank, Azure, DigitalOcean, Render, Railway, Hetzner, Hostinger).

### 9. Automation (`references/automation.md`)
Read this file for automation and scheduled tasks.
- **Topics covered:** Taskflow, Tasks, Standing orders, Cron jobs.

### 10. Nodes (`references/nodes.md`)
Read this file for node-specific documentation.
- **Topics covered:** Media understanding, Images, Audio, Voicewake, Troubleshooting.

### 11. Security (`references/security.md`)
Read this file for security guidelines and models.
- **Topics covered:** Threat model, Formal verification, Audit checks.

### 12. Platforms (`references/platforms.md`)
Read this file for platform-specific guides.
- **Topics covered:** macOS (Peekaboo, Canvas, Child process, Voice overlay, Bundled gateway), Windows, Android, Raspberry Pi.

### 13. Reference (`references/reference.md`)
Read this file for technical reference material.
- **Topics covered:** RPC adapters, SecretRef credential surface, Memory config, Prompt caching, Tests, Templates (TOOLS, SOUL, HEARTBEAT), Credits, Releasing.

### 14. Help (`references/help.md`)
Read this file for troubleshooting and FAQs.
- **Topics covered:** General troubleshooting, FAQ models, FAQ first-run, Testing, Debugging, Scripts.

### 15. Other (`references/other.md`)
Read this file for miscellaneous documentation.
- **Topics covered:** Network, CI, Prose, VPS, Pi-dev.

## Best Practices for OpenClaw

1. **Always check the Gateway configuration:** Many issues stem from incorrect `config.yaml` settings. Refer to `references/gateway.md` for the exact schema.
2. **Understand the Delegate Architecture:** OpenClaw uses a unique delegate model for handling messages. Review `references/concepts.md` before building complex multi-agent setups.
3. **Use the CLI for debugging:** The `openclaw status` and `openclaw memory` commands are essential for troubleshooting. See `references/cli.md`.
4. **Secure your Secrets:** Never hardcode API keys. Use OpenClaw's secrets management and SecretRef system (`references/gateway.md` and `references/reference.md`).

## How to Proceed

1. Identify the specific area of OpenClaw the user needs help with.
2. Use the `file` tool to read the corresponding reference markdown file from `/home/ubuntu/skills/openclaw-expert/references/`.
3. Synthesize the information and provide a detailed, accurate response or solution based on the official documentation.
