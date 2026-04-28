# openclaw-expert-skill

> Comprehensive OpenClaw Expert skill for AI agents — covering gateway, channels, agents, tools, plugins, providers, ops, and more.

[![skills.sh](https://img.shields.io/badge/skills.sh-openclaw--expert--skill-green)](https://skills.sh/galnaaman/openclaw-expert-skill)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Install

```bash
npx skills add galnaaman/openclaw-expert-skill
```

This installs the skill and makes it available to your AI agent automatically.

## What It Does

Once installed, your AI agent will have expert-level knowledge of the entire [OpenClaw](https://openclaw.ai) platform, sourced from **252 pages** scraped directly from [docs.openclaw.ai](https://docs.openclaw.ai). The skill covers all major areas:

| Reference File | Topics Covered |
|---|---|
| `gateway.md` | Config reference, secrets, security, auth, logging, metrics, protocol, bridge |
| `tools.md` | Web search, browser, PDF, image gen, exec approvals, skills, diffs, firecrawl |
| `plugins.md` | SDK overview, channel plugins, webhooks, voice calls, Codex, manifest, bundles |
| `providers.md` | OpenAI, Gemini, Anthropic, Bedrock, LiteLLM, Mistral, xAI, Nvidia, 20+ more |
| `cli.md` | All CLI commands: agent, memory, browser, models, MCP, webhooks, DNS, flows |
| `channels.md` | WhatsApp, Telegram, Signal, Matrix, WeChat, Zalo, IRC, Nostr, QQ, BlueBubbles |
| `concepts.md` | Delegate arch, QMD memory, multi-agent, dreaming, streaming, sessions, retry |
| `install.md` | Fly.io, GCP, Azure, DigitalOcean, Kubernetes, Docker, Ansible, Railway, Render |
| `automation.md` | Taskflow, tasks, standing orders, cron jobs |
| `nodes.md` | Media understanding, images, audio, voicewake, troubleshooting |
| `security.md` | Threat model, formal verification, audit checks |
| `platforms.md` | macOS, Windows, Android, Raspberry Pi |
| `reference.md` | RPC, SecretRef, memory config, prompt caching, templates |
| `help.md` | Troubleshooting, FAQs, debugging, testing, scripts |
| `other.md` | Network, VPS, CI, prose, Pi-dev, web/TUI, webchat |

**Total: ~2.7 MB of structured documentation across 49,809 lines.**

## Example Prompts

After installing, ask your agent things like:

- *"How do I configure a WhatsApp channel in OpenClaw?"*
- *"Show me the full gateway configuration reference"*
- *"How do I build a custom channel plugin with the SDK?"*
- *"Set up multi-agent routing in OpenClaw"*
- *"How do I connect OpenClaw to Anthropic Claude via Bedrock?"*
- *"Troubleshoot a broken OpenClaw gateway"*
- *"What CLI commands are available for managing memory?"*
- *"How do I deploy OpenClaw to Kubernetes?"*

## Skill Structure

```
openclaw-expert-skill/
├── SKILL.md              # Agent instructions and skill index
└── references/
    ├── gateway.md        # Gateway & ops documentation
    ├── channels.md       # Channel integrations
    ├── tools.md          # Built-in tools
    ├── plugins.md        # Plugin SDK & development
    ├── cli.md            # CLI reference
    ├── providers.md      # Model providers
    ├── concepts.md       # Core concepts & architecture
    ├── install.md        # Installation & deployment
    ├── automation.md     # Automation & scheduling
    ├── nodes.md          # Node documentation
    ├── security.md       # Security guidelines
    ├── platforms.md      # Platform-specific guides
    ├── reference.md      # Technical reference
    ├── help.md           # Help & troubleshooting
    └── other.md          # Miscellaneous docs
```

## Which AI Agents Support This Skill?

This skill works with any AI agent that supports the [skills.sh](https://skills.sh) format, including Claude Code, Cursor, Windsurf, Manus, and others.

## Source

- **OpenClaw Docs:** [docs.openclaw.ai](https://docs.openclaw.ai)
- **skills.sh:** [skills.sh/galnaaman/openclaw-expert-skill](https://skills.sh/galnaaman/openclaw-expert-skill)

## License

MIT
