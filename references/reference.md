# OpenClaw Reference Documentation

## SecretRef credential surface - OpenClaw
**Source:** https://docs.openclaw.ai/reference/secretref-credential-surface

[Skip to main content](https://docs.openclaw.ai/reference/secretref-credential-surface#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Technical reference

SecretRef credential surface

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Supported credentials](https://docs.openclaw.ai/reference/secretref-credential-surface#supported-credentials)
- [openclaw.json targets (secrets configure + secrets apply + secrets audit)](https://docs.openclaw.ai/reference/secretref-credential-surface#openclaw-json-targets-secrets-configure-%2B-secrets-apply-%2B-secrets-audit)
- [auth-profiles.json targets (secrets configure + secrets apply + secrets audit)](https://docs.openclaw.ai/reference/secretref-credential-surface#auth-profiles-json-targets-secrets-configure-%2B-secrets-apply-%2B-secrets-audit)
- [Unsupported credentials](https://docs.openclaw.ai/reference/secretref-credential-surface#unsupported-credentials)
- [Related](https://docs.openclaw.ai/reference/secretref-credential-surface#related)

This page defines the canonical SecretRef credential surface.Scope intent:

- In scope: strictly user-supplied credentials that OpenClaw does not mint or rotate.
- Out of scope: runtime-minted or rotating credentials, OAuth refresh material, and session-like artifacts.

## [​](https://docs.openclaw.ai/reference/secretref-credential-surface\\#supported-credentials)  Supported credentials

### [​](https://docs.openclaw.ai/reference/secretref-credential-surface\\#openclaw-json-targets-secrets-configure-+-secrets-apply-+-secrets-audit)  `openclaw.json` targets (`secrets configure` \\+ `secrets apply` \\+ `secrets audit`)

- `models.providers.*.apiKey`
- `models.providers.*.headers.*`
- `models.providers.*.request.auth.token`
- `models.providers.*.request.auth.value`
- `models.providers.*.request.headers.*`
- `models.providers.*.request.proxy.tls.ca`
- `models.providers.*.request.proxy.tls.cert`
- `models.providers.*.request.proxy.tls.key`
- `models.providers.*.request.proxy.tls.passphrase`
- `models.providers.*.request.tls.ca`
- `models.providers.*.request.tls.cert`
- `models.providers.*.request.tls.key`
- `models.providers.*.request.tls.passphrase`
- `skills.entries.*.apiKey`
- `agents.defaults.memorySearch.remote.apiKey`
- `agents.list[].tts.providers.*.apiKey`
- `agents.list[].memorySearch.remote.apiKey`
- `talk.providers.*.apiKey`
- `messages.tts.providers.*.apiKey`
- `tools.web.fetch.firecrawl.apiKey`
- `plugins.entries.acpx.config.mcpServers.*.env.*`
- `plugins.entries.brave.config.webSearch.apiKey`
- `plugins.entries.exa.config.webSearch.apiKey`
- `plugins.entries.google.config.webSearch.apiKey`
- `plugins.entries.xai.config.webSearch.apiKey`
- `plugins.entries.moonshot.config.webSearch.apiKey`
- `plugins.entries.perplexity.config.webSearch.apiKey`
- `plugins.entries.firecrawl.config.webSearch.apiKey`
- `plugins.entries.minimax.config.webSearch.apiKey`
- `plugins.entries.tavily.config.webSearch.apiKey`
- `plugins.entries.voice-call.config.tts.providers.*.apiKey`
- `plugins.entries.voice-call.config.twilio.authToken`
- `tools.web.search.apiKey`
- `gateway.auth.password`
- `gateway.auth.token`
- `gateway.remote.token`
- `gateway.remote.password`
- `cron.webhookToken`
- `channels.telegram.botToken`
- `channels.telegram.webhookSecret`
- `channels.telegram.accounts.*.botToken`
- `channels.telegram.accounts.*.webhookSecret`
- `channels.slack.botToken`
- `channels.slack.appToken`
- `channels.slack.userToken`
- `channels.slack.signingSecret`
- `channels.slack.accounts.*.botToken`
- `channels.slack.accounts.*.appToken`
- `channels.slack.accounts.*.userToken`
- `channels.slack.accounts.*.signingSecret`
- `channels.discord.token`
- `channels.discord.pluralkit.token`
- `channels.discord.voice.tts.providers.*.apiKey`
- `channels.discord.accounts.*.token`
- `channels.discord.accounts.*.pluralkit.token`
- `channels.discord.accounts.*.voice.tts.providers.*.apiKey`
- `channels.irc.password`
- `channels.irc.nickserv.password`
- `channels.irc.accounts.*.password`
- `channels.irc.accounts.*.nickserv.password`
- `channels.bluebubbles.password`
- `channels.bluebubbles.accounts.*.password`
- `channels.feishu.appSecret`
- `channels.feishu.encryptKey`
- `channels.feishu.verificationToken`
- `channels.feishu.accounts.*.appSecret`
- `channels.feishu.accounts.*.encryptKey`
- `channels.feishu.accounts.*.verificationToken`
- `channels.msteams.appPassword`
- `channels.mattermost.botToken`
- `channels.mattermost.accounts.*.botToken`
- `channels.matrix.accessToken`
- `channels.matrix.password`
- `channels.matrix.accounts.*.accessToken`
- `channels.matrix.accounts.*.password`
- `channels.nextcloud-talk.botSecret`
- `channels.nextcloud-talk.apiPassword`
- `channels.nextcloud-talk.accounts.*.botSecret`
- `channels.nextcloud-talk.accounts.*.apiPassword`
- `channels.zalo.botToken`
- `channels.zalo.webhookSecret`
- `channels.zalo.accounts.*.botToken`
- `channels.zalo.accounts.*.webhookSecret`
- `channels.googlechat.serviceAccount` via sibling `serviceAccountRef` (compatibility exception)
- `channels.googlechat.accounts.*.serviceAccount` via sibling `serviceAccountRef` (compatibility exception)

### [​](https://docs.openclaw.ai/reference/secretref-credential-surface\\#auth-profiles-json-targets-secrets-configure-+-secrets-apply-+-secrets-audit)  `auth-profiles.json` targets (`secrets configure` \\+ `secrets apply` \\+ `secrets audit`)

- `profiles.*.keyRef` (`type: \"api_key\"`; unsupported when `auth.profiles.<id>.mode = \"oauth\"`)
- `profiles.*.tokenRef` (`type: \"token\"`; unsupported when `auth.profiles.<id>.mode = \"oauth\"`)

Notes:

- Auth-profile plan targets require `agentId`.
- Plan entries target `profiles.*.key` / `profiles.*.token` and write sibling refs (`keyRef` / `tokenRef`).
- Auth-profile refs are included in runtime resolution and audit coverage.
- In `openclaw.json`, SecretRefs must use structured objects such as `{\"source\":\"env\",\"provider\":\"default\",\"id\":\"DISCORD_BOT_TOKEN\"}`. Legacy `secretref-env:<ENV_VAR>` marker strings are rejected on SecretRef credential paths; run `openclaw doctor --fix` to migrate valid markers.
- OAuth policy guard: `auth.profiles.<id>.mode = \"oauth\"` cannot be combined with SecretRef inputs for that profile. Startup/reload and auth-profile resolution fail fast when this policy is violated.
- For SecretRef-managed model providers, generated `agents/*/agent/models.json` entries persist non-secret markers (not resolved secret values) for `apiKey`/header surfaces.
- Marker persistence is source-authoritative: OpenClaw writes markers from the active source config snapshot (pre-resolution), not from resolved runtime secret values.
- For web search:
  - In explicit provider mode (`tools.web.search.provider` set), only the selected provider key is active.
  - In auto mode (`tools.web.search.provider` unset), only the first provider key that resolves by precedence is active.
  - In auto mode, non-selected provider refs are treated as inactive until selected.
  - Legacy `tools.web.search.*` provider paths still resolve during the compatibility window, but the canonical SecretRef surface is `plugins.entries.<plugin>.config.webSearch.*`.

## [​](https://docs.openclaw.ai/reference/secretref-credential-surface\\#unsupported-credentials)  Unsupported credentials

Out-of-scope credentials include:

- `commands.ownerDisplaySecret`
- `hooks.token`
- `hooks.gmail.pushToken`
- `hooks.mappings[].sessionKey`
- `auth-profiles.oauth.*`
- `channels.discord.threadBindings.webhookToken`
- `channels.discord.accounts.*.threadBindings.webhookToken`
- `channels.whatsapp.creds.json`
- `channels.whatsapp.accounts.*.creds.json`

Rationale:

- These credentials are minted, rotated, session-bearing, or OAuth-durable classes that do not fit read-only external SecretRef resolution.

## [​](https://docs.openclaw.ai/reference/secretref-credential-surface\\#related)  Related

- [Secrets management](https://docs.openclaw.ai/gateway/secrets)
- [Auth credential semantics](https://docs.openclaw.ai/auth-credential-semantics)

[Token use and costs](https://docs.openclaw.ai/reference/token-use) [Prompt caching](https://docs.openclaw.ai/reference/prompt-caching)

Ctrl+I

---

## RPC adapters - OpenClaw
**Source:** https://docs.openclaw.ai/reference/rpc

[Skip to main content](https://docs.openclaw.ai/reference/rpc#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

RPC and API

RPC adapters

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Pattern A: HTTP daemon (signal-cli)](https://docs.openclaw.ai/reference/rpc#pattern-a-http-daemon-signal-cli)
- [Pattern B: stdio child process (legacy: imsg)](https://docs.openclaw.ai/reference/rpc#pattern-b-stdio-child-process-legacy-imsg)
- [Adapter guidelines](https://docs.openclaw.ai/reference/rpc#adapter-guidelines)
- [Related](https://docs.openclaw.ai/reference/rpc#related)

OpenClaw integrates external CLIs via JSON-RPC. Two patterns are used today.

## [​](https://docs.openclaw.ai/reference/rpc\\#pattern-a-http-daemon-signal-cli)  Pattern A: HTTP daemon (signal-cli)

- `signal-cli` runs as a daemon with JSON-RPC over HTTP.
- Event stream is SSE (`/api/v1/events`).
- Health probe: `/api/v1/check`.
- OpenClaw owns lifecycle when `channels.signal.autoStart=true`.

See [Signal](https://docs.openclaw.ai/channels/signal) for setup and endpoints.

## [​](https://docs.openclaw.ai/reference/rpc\\#pattern-b-stdio-child-process-legacy-imsg)  Pattern B: stdio child process (legacy: imsg)

> **Note:** For new iMessage setups, use [BlueBubbles](https://docs.openclaw.ai/channels/bluebubbles) instead.

- OpenClaw spawns `imsg rpc` as a child process (legacy iMessage integration).
- JSON-RPC is line-delimited over stdin/stdout (one JSON object per line).
- No TCP port, no daemon required.

Core methods used:

- `watch.subscribe` → notifications (`method: \"message\"`)
- `watch.unsubscribe`
- `send`
- `chats.list` (probe/diagnostics)

See [iMessage](https://docs.openclaw.ai/channels/imessage) for legacy setup and addressing (`chat_id` preferred).

## [​](https://docs.openclaw.ai/reference/rpc\\#adapter-guidelines)  Adapter guidelines

- Gateway owns the process (start/stop tied to provider lifecycle).
- Keep RPC clients resilient: timeouts, restart on exit.
- Prefer stable IDs (e.g., `chat_id`) over display strings.

## [​](https://docs.openclaw.ai/reference/rpc\\#related)  Related

- [Gateway protocol](https://docs.openclaw.ai/gateway/protocol)

[Wiki](https://docs.openclaw.ai/cli/wiki) [Device model database](https://docs.openclaw.ai/reference/device-models)

Ctrl+I

---

## Memory configuration reference
**Source:** https://docs.openclaw.ai/reference/memory-config

[Skip to main content](https://docs.openclaw.ai/reference/memory-config#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nTechnical reference\n\nMemory configuration reference\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Provider selection](https://docs.openclaw.ai/reference/memory-config#provider-selection)\n- [Auto-detection order](https://docs.openclaw.ai/reference/memory-config#auto-detection-order)\n- [API key resolution](https://docs.openclaw.ai/reference/memory-config#api-key-resolution)\n- [Remote endpoint config](https://docs.openclaw.ai/reference/memory-config#remote-endpoint-config)\n- [Provider-specific config](https://docs.openclaw.ai/reference/memory-config#provider-specific-config)\n- [Inline embedding timeout](https://docs.openclaw.ai/reference/memory-config#inline-embedding-timeout)\n- [Hybrid search config](https://docs.openclaw.ai/reference/memory-config#hybrid-search-config)\n- [Full example](https://docs.openclaw.ai/reference/memory-config#full-example)\n- [Additional memory paths](https://docs.openclaw.ai/reference/memory-config#additional-memory-paths)\n- [Multimodal memory (Gemini)](https://docs.openclaw.ai/reference/memory-config#multimodal-memory-gemini)\n- [Embedding cache](https://docs.openclaw.ai/reference/memory-config#embedding-cache)\n- [Batch indexing](https://docs.openclaw.ai/reference/memory-config#batch-indexing)\n- [Session memory search (experimental)](https://docs.openclaw.ai/reference/memory-config#session-memory-search-experimental)\n- [SQLite vector acceleration (sqlite-vec)](https://docs.openclaw.ai/reference/memory-config#sqlite-vector-acceleration-sqlite-vec)\n- [Index storage](https://docs.openclaw.ai/reference/memory-config#index-storage)\n- [QMD backend config](https://docs.openclaw.ai/reference/memory-config#qmd-backend-config)\n- [Full QMD example](https://docs.openclaw.ai/reference/memory-config#full-qmd-example)\n- [Dreaming](https://docs.openclaw.ai/reference/memory-config#dreaming)\n- [User settings](https://docs.openclaw.ai/reference/memory-config#user-settings)\n- [Example](https://docs.openclaw.ai/reference/memory-config#example)\n- [Related](https://docs.openclaw.ai/reference/memory-config#related)\n\nThis page lists every configuration knob for OpenClaw memory search. For conceptual overviews, see:\n\n[**Memory overview** \\\\\n\\\\\nHow memory works.](https://docs.openclaw.ai/concepts/memory)\n\n[**Builtin engine** \\\\\n\\\\\nDefault SQLite backend.](https://docs.openclaw.ai/concepts/memory-builtin)\n\n[**QMD engine** \\\\\n\\\\\nLocal-first sidecar.](https://docs.openclaw.ai/concepts/memory-qmd)\n\n[**Memory search** \\\\\n\\\\\nSearch pipeline and tuning.](https://docs.openclaw.ai/concepts/memory-search)\n\n[**Active memory** \\\\\n\\\\\nMemory sub-agent for interactive sessions.](https://docs.openclaw.ai/concepts/active-memory)\n\nAll memory search settings live under `agents.defaults.memorySearch` in `openclaw.json` unless noted otherwise.\n\nIf you are looking for the **active memory** feature toggle and sub-agent config, that lives under `plugins.entries.active-memory` instead of `memorySearch`.Active memory uses a two-gate model:\n\n1. the plugin must be enabled and target the current agent id\n2. the request must be an eligible interactive persistent chat session\n\nSee [Active Memory](https://docs.openclaw.ai/concepts/active-memory) for the activation model, plugin-owned config, transcript persistence, and safe rollout pattern.\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#provider-selection)  Provider selection\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `provider` | `string` | auto-detected | Embedding adapter ID: `bedrock`, `gemini`, `github-copilot`, `local`, `mistral`, `ollama`, `openai`, `voyage` |\n| `model` | `string` | provider default | Embedding model name |\n| `fallback` | `string` | `\"none\"` | Fallback adapter ID when the primary fails |\n| `enabled` | `boolean` | `true` | Enable or disable memory search |\n\n### [​](https://docs.openclaw.ai/reference/memory-config\\#auto-detection-order)  Auto-detection order\n\nWhen `provider` is not set, OpenClaw selects the first available:\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/reference/memory-config#)\n\nlocal\n\nSelected if `memorySearch.local.modelPath` is configured and the file exists.\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/reference/memory-config#)\n\ngithub-copilot\n\nSelected if a GitHub Copilot token can be resolved (env var or auth profile).\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/reference/memory-config#)\n\nopenai\n\nSelected if an OpenAI key can be resolved.\n\n4\n\n[Navigate to header](https://docs.openclaw.ai/reference/memory-config#)\n\ngemini\n\nSelected if a Gemini key can be resolved.\n\n5\n\n[Navigate to header](https://docs.openclaw.ai/reference/memory-config#)\n\nvoyage\n\nSelected if a Voyage key can be resolved.\n\n6\n\n[Navigate to header](https://docs.openclaw.ai/reference/memory-config#)\n\nmistral\n\nSelected if a Mistral key can be resolved.\n\n7\n\n[Navigate to header](https://docs.openclaw.ai/reference/memory-config#)\n\nbedrock\n\nSelected if the AWS SDK credential chain resolves (instance role, access keys, profile, SSO, web identity, or shared config).\n\n`ollama` is supported but not auto-detected (set it explicitly).\n\n### [​](https://docs.openclaw.ai/reference/memory-config\\#api-key-resolution)  API key resolution\n\nRemote embeddings require an API key. Bedrock uses the AWS SDK default credential chain instead (instance roles, SSO, access keys).\n\n| Provider | Env var | Config key |\n| --- | --- | --- |\n| Bedrock | AWS credential chain | No API key needed |\n| Gemini | `GEMINI_API_KEY` | `models.providers.google.apiKey` |\n| GitHub Copilot | `COPILOT_GITHUB_TOKEN`, `GH_TOKEN`, `GITHUB_TOKEN` | Auth profile via device login |\n| Mistral | `MISTRAL_API_KEY` | `models.providers.mistral.apiKey` |\n| Ollama | `OLLAMA_API_KEY` (placeholder) | — |\n| OpenAI | `OPENAI_API_KEY` | `models.providers.openai.apiKey` |\n| Voyage | `VOYAGE_API_KEY` | `models.providers.voyage.apiKey` |\n\nCodex OAuth covers chat/completions only and does not satisfy embedding requests.\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#remote-endpoint-config)  Remote endpoint config\n\nFor custom OpenAI-compatible endpoints or overriding provider defaults:\n\n[​](https://docs.openclaw.ai/reference/memory-config#param-remote-base-url)\n\nremote.baseUrl\n\nstring\n\nCustom API base URL.\n\n[​](https://docs.openclaw.ai/reference/memory-config#param-remote-api-key)\n\nremote.apiKey\n\nstring\n\nOverride API key.\n\n[​](https://docs.openclaw.ai/reference/memory-config#param-remote-headers)\n\nremote.headers\n\nobject\n\nExtra HTTP headers (merged with provider defaults).\n\n```\n{\n  agents: {\n    defaults: {\n      memorySearch: {\n        provider: \"openai\",\n        model: \"text-embedding-3-small\",\n        remote: {\n          baseUrl: \"https://api.example.com/v1/\",\n          apiKey: \"YOUR_KEY\",\n        },\n      },\n    },\n  },\n}\n```\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#provider-specific-config)  Provider-specific config\n\nGemini\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `model` | `string` | `gemini-embedding-001` | Also supports `gemini-embedding-2-preview` |\n| `outputDimensionality` | `number` | `3072` | For Embedding 2: 768, 1536, or 3072 |\n\nChanging model or `outputDimensionality` triggers an automatic full reindex.\n\nOpenAI-compatible input types\n\nOpenAI-compatible embedding endpoints can opt into provider-specific `input_type` request fields. This is useful for asymmetric embedding models that require different labels for query and document embeddings.\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `inputType` | `string` | unset | Shared `input_type` for query and document embeddings |\n| `queryInputType` | `string` | unset | Query-time `input_type`; overrides `inputType` |\n| `documentInputType` | `string` | unset | Index/document `input_type`; overrides `inputType` |\n\n```\n{\n  agents: {\n    defaults: {\n      memorySearch: {\n        provider: \"openai\",\n        remote: {\n          baseUrl: \"https://embeddings.example/v1\",\n          apiKey: \"env:EMBEDDINGS_API_KEY\",\n        },\n        model: \"asymmetric-embedder\",\n        queryInputType: \"query\",\n        documentInputType: \"passage\",\n      },\n    },\n  },\n}\n```\n\nChanging these values affects embedding cache identity for provider batch indexing and should be followed by a memory reindex when the upstream model treats the labels differently.\n\nBedrock\n\nBedrock uses the AWS SDK default credential chain — no API keys needed. If OpenClaw runs on EC2 with a Bedrock-enabled instance role, just set the provider and model:\n\n```\n{\n  agents: {\n    defaults: {\n      memorySearch: {\n        provider: \"bedrock\",\n        model: \"amazon.titan-embed-text-v2:0\",\n      },\n    },\n  },\n}\n```\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `model` | `string` | `amazon.titan-embed-text-v2:0` | Any Bedrock embedding model ID |\n| `outputDimensionality` | `number` | model default | For Titan V2: 256, 512, or 1024 |\n\n**Supported models** (with family detection and dimension defaults):\n\n| Model ID | Provider | Default Dims | Configurable Dims |\n| --- | --- | --- | --- |\n| `amazon.titan-embed-text-v2:0` | Amazon | 1024 | 256, 512, 1024 |\n| `amazon.titan-embed-text-v1` | Amazon | 1536 | — |\n| `amazon.titan-embed-g1-text-02` | Amazon | 1536 | — |\n| `amazon.titan-embed-image-v1` | Amazon | 1024 | — |\n| `amazon.nova-2-multimodal-embeddings-v1:0` | Amazon | 1024 | 256, 384, 1024, 3072 |\n| `cohere.embed-english-v3` | Cohere | 1024 | — |\n| `cohere.embed-multilingual-v3` | Cohere | 1024 | — |\n| `cohere.embed-v4:0` | Cohere | 1536 | 256-1536 |\n| `twelvelabs.marengo-embed-3-0-v1:0` | TwelveLabs | 512 | — |\n| `twelvelabs.marengo-embed-2-7-v1:0` | TwelveLabs | 1024 | — |\n\nThroughput-suffixed variants (e.g., `amazon.titan-embed-text-v1:2:8k`) inherit the base model’s configuration.**Authentication:** Bedrock auth uses the standard AWS SDK credential resolution order:\n\n1. Environment variables (`AWS_ACCESS_KEY_ID` \\+ `AWS_SECRET_ACCESS_KEY`)\n2. SSO token cache\n3. Web identity token credentials\n4. Shared credentials and config files\n5. ECS or EC2 metadata credentials\n\nRegion is resolved from `AWS_REGION`, `AWS_DEFAULT_REGION`, the `amazon-bedrock` provider `baseUrl`, or defaults to `us-east-1`.**IAM permissions:** the IAM role or user needs:\n\n```\n{\n  \"Effect\": \"Allow\",\n  \"Action\": \"bedrock:InvokeModel\",\n  \"Resource\": \"*\"\n}\n```\n\nFor least-privilege, scope `InvokeModel` to the specific model:\n\n```\narn:aws:bedrock:*::foundation-model/amazon.titan-embed-text-v2:0\n```\n\nLocal (GGUF + node-llama-cpp)\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `local.modelPath` | `string` | auto-downloaded | Path to GGUF model file |\n| `local.modelCacheDir` | `string` | node-llama-cpp default | Cache dir for downloaded models |\n| `local.contextSize` | `number | \"auto\"` | `4096` | Context window size for the embedding context. 4096 covers typical chunks (128–512 tokens) while bounding non-weight VRAM. Lower to 1024–2048 on constrained hosts. `\"auto\"` uses the model’s trained maximum — not recommended for 8B+ models (Qwen3-Embedding-8B: 40 960 tokens → ~32 GB VRAM vs ~8.8 GB at 4096). |\n\nDefault model: `embeddinggemma-300m-qat-Q8_0.gguf` (~0.6 GB, auto-downloaded). Requires native build: `pnpm approve-builds` then `pnpm rebuild node-llama-cpp`.Use the standalone CLI to verify the same provider path the Gateway uses:\n\n```\nopenclaw memory status --deep --agent main\nopenclaw memory index --force --agent main\n```\n\nIf `provider` is `auto`, `local` is selected only when `local.modelPath` points to an existing local file. `hf:` and HTTP(S) model references can still be used explicitly with `provider: \"local\"`, but they do not make `auto` select local before the model is available on disk.\n\n### [​](https://docs.openclaw.ai/reference/memory-config\\#inline-embedding-timeout)  Inline embedding timeout\n\n[​](https://docs.openclaw.ai/reference/memory-config#param-sync-embedding-batch-timeout-seconds)\n\nsync.embeddingBatchTimeoutSeconds\n\nnumber\n\nOverride the timeout for inline embedding batches during memory indexing.Unset uses the provider default: 600 seconds for local/self-hosted providers such as `local`, `ollama`, and `lmstudio`, and 120 seconds for hosted providers. Increase this when local CPU-bound embedding batches are healthy but slow.\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#hybrid-search-config)  Hybrid search config\n\nAll under `memorySearch.query.hybrid`:\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `enabled` | `boolean` | `true` | Enable hybrid BM25 + vector search |\n| `vectorWeight` | `number` | `0.7` | Weight for vector scores (0-1) |\n| `textWeight` | `number` | `0.3` | Weight for BM25 scores (0-1) |\n| `candidateMultiplier` | `number` | `4` | Candidate pool size multiplier |\n\n- MMR (diversity)\n\n- Temporal decay (recency)\n\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `mmr.enabled` | `boolean` | `false` | Enable MMR re-ranking |\n| `mmr.lambda` | `number` | `0.7` | 0 = max diversity, 1 = max relevance |\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `temporalDecay.enabled` | `boolean` | `false` | Enable recency boost |\n| `temporalDecay.halfLifeDays` | `number` | `30` | Score halves every N days |\n\nEvergreen files (`MEMORY.md`, non-dated files in `memory/`) are never decayed.\n\n### [​](https://docs.openclaw.ai/reference/memory-config\\#full-example)  Full example\n\n```\n{\n  agents: {\n    defaults: {\n      memorySearch: {\n        query: {\n          hybrid: {\n            vectorWeight: 0.7,\n            textWeight: 0.3,\n            mmr: { enabled: true, lambda: 0.7 },\n            temporalDecay: { enabled: true, halfLifeDays: 30 },\n          },\n        },\n      },\n    },\n  },\n}\n```\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#additional-memory-paths)  Additional memory paths\n\n| Key | Type | Description |\n| --- | --- | --- |\n| `extraPaths` | `string[]` | Additional directories or files to index |\n\n```\n{\n  agents: {\n    defaults: {\n      memorySearch: {\n        extraPaths: [\"../team-docs\", \"/srv/shared-notes\"],\n      },\n    },\n  },\n}\n```\n\nPaths can be absolute or workspace-relative. Directories are scanned recursively for `.md` files. Symlink handling depends on the active backend: the builtin engine ignores symlinks, while QMD follows the underlying QMD scanner behavior.For agent-scoped cross-agent transcript search, use `agents.list[].memorySearch.qmd.extraCollections` instead of `memory.qmd.paths`. Those extra collections follow the same `{ path, name, pattern? }` shape, but they are merged per agent and can preserve explicit shared names when the path points outside the current workspace. If the same resolved path appears in both `memory.qmd.paths` and `memorySearch.qmd.extraCollections`, QMD keeps the first entry and skips the duplicate.\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#multimodal-memory-gemini)  Multimodal memory (Gemini)\n\nIndex images and audio alongside Markdown using Gemini Embedding 2:\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `multimodal.enabled` | `boolean` | `false` | Enable multimodal indexing |\n| `multimodal.modalities` | `string[]` | — | `[\"image\"]`, `[\"audio\"]`, or `[\"all\"]` |\n| `multimodal.maxFileBytes` | `number` | `10000000` | Max file size for indexing |\n\nOnly applies to files in `extraPaths`. Default memory roots stay Markdown-only. Requires `gemini-embedding-2-preview`. `fallback` must be `\"none\"`.\n\nSupported formats: `.jpg`, `.jpeg`, `.png`, `.webp`, `.gif`, `.heic`, `.heif` (images); `.mp3`, `.wav`, `.ogg`, `.opus`, `.m4a`, `.aac`, `.flac` (audio).\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#embedding-cache)  Embedding cache\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `cache.enabled` | `boolean` | `false` | Cache chunk embeddings in SQLite |\n| `cache.maxEntries` | `number` | `50000` | Max cached embeddings |\n\nPrevents re-embedding unchanged text during reindex or transcript updates.\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#batch-indexing)  Batch indexing\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `remote.batch.enabled` | `boolean` | `false` | Enable batch embedding API |\n| `remote.batch.concurrency` | `number` | `2` | Parallel batch jobs |\n| `remote.batch.wait` | `boolean` | `true` | Wait for batch completion |\n| `remote.batch.pollIntervalMs` | `number` | — | Poll interval |\n| `remote.batch.timeoutMinutes` | `number` | — | Batch timeout |\n\nAvailable for `openai`, `gemini`, and `voyage`. OpenAI batch is typically fastest and cheapest for large backfills.This is separate from `sync.embeddingBatchTimeoutSeconds`, which controls inline embedding calls used by local/self-hosted providers and hosted providers when provider batch APIs are not active.\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#session-memory-search-experimental)  Session memory search (experimental)\n\nIndex session transcripts and surface them via `memory_search`:\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `experimental.sessionMemory` | `boolean` | `false` | Enable session indexing |\n| `sources` | `string[]` | `[\"memory\"]` | Add `\"sessions\"` to include transcripts |\n| `sync.sessions.deltaBytes` | `number` | `100000` | Byte threshold for reindex |\n| `sync.sessions.deltaMessages` | `number` | `50` | Message threshold for reindex |\n\nSession indexing is opt-in and runs asynchronously. Results can be slightly stale. Session logs live on disk, so treat filesystem access as the trust boundary.\n\n* * *\n\n## [​](https://docs.openclaw.ai/reference/memory-config\\#sqlite-vector-acceleration-sqlite-vec)  SQLite vector acceleration (sqlite-vec)\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `store.vector.enabled` | `boolean` | `true` | Use sqlite-vec for vector queries |\n| `store.vector.extensionPath` | `string` | bundled | Override sqlite-vec path |\n\nWhen sqlite-vec is unavailable, OpenClaw falls back to in-process co...(content truncated)

---

## Tests - OpenClaw
**Source:** https://docs.openclaw.ai/reference/test

[Skip to main content](https://docs.openclaw.ai/reference/test#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Release and CI

Tests

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Local PR gate](https://docs.openclaw.ai/reference/test#local-pr-gate)
- [Model latency bench (local keys)](https://docs.openclaw.ai/reference/test#model-latency-bench-local-keys)
- [CLI startup bench](https://docs.openclaw.ai/reference/test#cli-startup-bench)
- [Onboarding E2E (Docker)](https://docs.openclaw.ai/reference/test#onboarding-e2e-docker)
- [QR import smoke (Docker)](https://docs.openclaw.ai/reference/test#qr-import-smoke-docker)
- [Related](https://docs.openclaw.ai/reference/test#related)

- Full testing kit (suites, live, Docker): [Testing](https://docs.openclaw.ai/help/testing)
- `pnpm test:force`: Kills any lingering gateway process holding the default control port, then runs the full Vitest suite with an isolated gateway port so server tests don’t collide with a running instance. Use this when a prior gateway run left port 18789 occupied.
- `pnpm test:coverage`: Runs the unit suite with V8 coverage (via `vitest.unit.config.ts`). This is a loaded-file unit coverage gate, not whole-repo all-file coverage. Thresholds are 70% lines/functions/statements and 55% branches. Because `coverage.all` is false, the gate measures files loaded by the unit coverage suite instead of treating every split-lane source file as uncovered.
- `pnpm test:coverage:changed`: Runs unit coverage only for files changed since `origin/main`.
- `pnpm test:changed`: cheap smart changed test run. It runs precise targets from direct test edits, sibling `*.test.ts` files, explicit source mappings, and the local import graph. Broad/config/package changes are skipped unless they map to precise tests.
- `OPENCLAW_TEST_CHANGED_BROAD=1 pnpm test:changed`: explicit broad changed test run. Use it when a test harness/config/package edit should fall back to Vitest’s broader changed-test behavior.
- `pnpm changed:lanes`: shows the architectural lanes triggered by the diff against `origin/main`.
- `pnpm check:changed`: runs the smart changed check gate for the diff against `origin/main`. It runs typecheck, lint, and guard commands for the affected architectural lanes, but does not run Vitest tests. Use `pnpm test:changed` or explicit `pnpm test <target>` for test proof.
- `pnpm test`: routes explicit file/directory targets through scoped Vitest lanes. Untargeted runs use fixed shard groups and expand to leaf configs for local parallel execution; the extension group always expands to the per-extension shard configs instead of one giant root-project process.
- Test wrapper runs end with a short `[test] passed|failed|skipped ... in ...` summary. Vitest’s own duration line stays the per-shard detail.
- Full, extension, and include-pattern shard runs update local timing data in `.artifacts/vitest-shard-timings.json`; later whole-config runs use those timings to balance slow and fast shards. Include-pattern CI shards append the shard name to the timing key, which keeps filtered shard timings visible without replacing whole-config timing data. Set `OPENCLAW_TEST_PROJECTS_TIMINGS=0` to ignore the local timing artifact.
- Selected `plugin-sdk` and `commands` test files now route through dedicated light lanes that keep only `test/setup.ts`, leaving runtime-heavy cases on their existing lanes.
- Source files with sibling tests map to that sibling before falling back to wider directory globs. Helper edits under `test/helpers/channels` and `test/helpers/plugins` use a local import graph to run importing tests instead of broad-running every shard when the dependency path is precise.
- `auto-reply` now also splits into three dedicated configs (`core`, `top-level`, `reply`) so the reply harness does not dominate the lighter top-level status/token/helper tests.
- Base Vitest config now defaults to `pool: \"threads\"` and `isolate: false`, with the shared non-isolated runner enabled across the repo configs.
- `pnpm test:channels` runs `vitest.channels.config.ts`.
- `pnpm test:extensions` and `pnpm test extensions` run all extension/plugin shards. Heavy channel plugins, the browser plugin, and OpenAI run as dedicated shards; other plugin groups stay batched. Use `pnpm test extensions/<id>` for one bundled plugin lane.
- `pnpm test:perf:imports`: enables Vitest import-duration + import-breakdown reporting, while still using scoped lane routing for explicit file/directory targets.
- `pnpm test:perf:imports:changed`: same import profiling, but only for files changed since `origin/main`.
- `pnpm test:perf:changed:bench -- --ref <git-ref>` benchmarks the routed changed-mode path against the native root-project run for the same committed git diff.
- `pnpm test:perf:changed:bench -- --worktree` benchmarks the current worktree change set without committing first.
- `pnpm test:perf:profile:main`: writes a CPU profile for the Vitest main thread (`.artifacts/vitest-main-profile`).
- `pnpm test:perf:profile:runner`: writes CPU + heap profiles for the unit runner (`.artifacts/vitest-runner-profile`).
- `pnpm test:perf:groups --full-suite --allow-failures --output .artifacts/test-perf/baseline-before.json`: runs every full-suite Vitest leaf config serially and writes grouped duration data plus per-config JSON/log artifacts. The Test Performance Agent uses this as its baseline before attempting slow-test fixes.
- `pnpm test:perf:groups:compare .artifacts/test-perf/baseline-before.json .artifacts/test-perf/after-agent.json`: compares grouped reports after a performance-focused change.
- Gateway integration: opt-in via `OPENCLAW_TEST_INCLUDE_GATEWAY=1 pnpm test` or `pnpm test:gateway`.
- `pnpm test:e2e`: Runs gateway end-to-end smoke tests (multi-instance WS/HTTP/node pairing). Defaults to `threads` \\+ `isolate: false` with adaptive workers in `vitest.e2e.config.ts`; tune with `OPENCLAW_E2E_WORKERS=<n>` and set `OPENCLAW_E2E_VERBOSE=1` for verbose logs.
- `pnpm test:live`: Runs provider live tests (minimax/zai). Requires API keys and `LIVE=1` (or provider-specific `*_LIVE_TEST=1`) to unskip.
- `pnpm test:docker:all`: Builds the shared live-test image, packs OpenClaw once as an npm tarball, builds/reuses a bare Node/Git runner image plus a functional image that installs that tarball into `/app`, then runs Docker smoke lanes with `OPENCLAW_SKIP_DOCKER_BUILD=1` through a weighted scheduler. The bare image (`OPENCLAW_DOCKER_E2E_BARE_IMAGE`) is used for installer/update/plugin-dependency lanes; those lanes mount the prebuilt tarball instead of using copied repo sources. The functional image (`OPENCLAW_DOCKER_E2E_FUNCTIONAL_IMAGE`) is used for normal built-app functionality lanes. `scripts/package-openclaw-for-docker.mjs` is the single local/CI package packer and validates the tarball plus `dist/postinstall-inventory.json` before Docker consumes it. Docker lane definitions live in `scripts/lib/docker-e2e-scenarios.mjs`; planner logic lives in `scripts/lib/docker-e2e-plan.mjs`; `scripts/test-docker-all.mjs` executes the selected plan. `node scripts/test-docker-all.mjs --plan-json` emits the scheduler-owned CI plan for selected lanes, image kinds, package/live-image needs, and credential checks without building or running Docker. `OPENCLAW_DOCKER_ALL_PARALLELISM=<n>` controls process slots and defaults to 10; `OPENCLAW_DOCKER_ALL_TAIL_PARALLELISM=<n>` controls the provider-sensitive tail pool and defaults to 10. Heavy lane caps default to `OPENCLAW_DOCKER_ALL_LIVE_LIMIT=9`, `OPENCLAW_DOCKER_ALL_NPM_LIMIT=10`, and `OPENCLAW_DOCKER_ALL_SERVICE_LIMIT=7`; provider caps default to one heavy lane per provider via `OPENCLAW_DOCKER_ALL_LIVE_CLAUDE_LIMIT=4`, `OPENCLAW_DOCKER_ALL_LIVE_CODEX_LIMIT=4`, and `OPENCLAW_DOCKER_ALL_LIVE_GEMINI_LIMIT=4`. Use `OPENCLAW_DOCKER_ALL_WEIGHT_LIMIT` or `OPENCLAW_DOCKER_ALL_DOCKER_LIMIT` for larger hosts. If one lane exceeds the effective weight or resource cap on a low-parallelism host, it can still start from an empty pool and will run alone until it releases capacity. Lane starts are staggered by 2 seconds by default to avoid local Docker daemon create storms; override with `OPENCLAW_DOCKER_ALL_START_STAGGER_MS=<ms>`. The runner preflights Docker by default, cleans stale OpenClaw E2E containers, emits active-lane status every 30 seconds, shares provider CLI tool caches between compatible lanes, retries transient live-provider failures once by default (`OPENCLAW_DOCKER_ALL_LIVE_RETRIES=<n>`), and stores lane timings in `.artifacts/docker-tests/lane-timings.json` for longest-first ordering on later runs. Use `OPENCLAW_DOCKER_ALL_DRY_RUN=1` to print the lane manifest without running Docker, `OPENCLAW_DOCKER_ALL_STATUS_INTERVAL_MS=<ms>` to tune status output, or `OPENCLAW_DOCKER_ALL_TIMINGS=0` to disable timing reuse. Use `OPENCLAW_DOCKER_ALL_LIVE_MODE=skip` for deterministic/local lanes only or `OPENCLAW_DOCKER_ALL_LIVE_MODE=only` for live-provider lanes only; package aliases are `pnpm test:docker:local:all` and `pnpm test:docker:live:all`. Live-only mode merges main and tail live lanes into one longest-first pool so provider buckets can pack Claude, Codex, and Gemini work together. The runner stops scheduling new pooled lanes after the first failure unless `OPENCLAW_DOCKER_ALL_FAIL_FAST=0` is set, and each lane has a 120-minute fallback timeout overrideable with `OPENCLAW_DOCKER_ALL_LANE_TIMEOUT_MS`; selected live/tail lanes use tighter per-lane caps. CLI backend Docker setup commands have their own timeout via `OPENCLAW_LIVE_CLI_BACKEND_SETUP_TIMEOUT_SECONDS` (default 180). Per-lane logs, `summary.json`, `failures.json`, and phase timings are written under `.artifacts/docker-tests/<run-id>/`; use `pnpm test:docker:timings <summary.json>` to inspect slow lanes and `pnpm test:docker:rerun <run-id|summary.json|failures.json>` to print cheap targeted rerun commands.
- `pnpm test:docker:browser-cdp-snapshot`: Builds a Chromium-backed source E2E container, starts raw CDP plus an isolated Gateway, runs `browser doctor --deep`, and verifies CDP role snapshots include link URLs, cursor-promoted clickables, iframe refs, and frame metadata.
- CLI backend live Docker probes can be run as focused lanes, for example `pnpm test:docker:live-cli-backend:codex`, `pnpm test:docker:live-cli-backend:codex:resume`, or `pnpm test:docker:live-cli-backend:codex:mcp`. Claude and Gemini have matching `:resume` and `:mcp` aliases.
- `pnpm test:docker:openwebui`: Starts Dockerized OpenClaw + Open WebUI, signs in through Open WebUI, checks `/api/models`, then runs a real proxied chat through `/api/chat/completions`. Requires a usable live model key (for example OpenAI in `~/.profile`), pulls an external Open WebUI image, and is not expected to be CI-stable like the normal unit/e2e suites.
- `pnpm test:docker:mcp-channels`: Starts a seeded Gateway container and a second client container that spawns `openclaw mcp serve`, then verifies routed conversation discovery, transcript reads, attachment metadata, live event queue behavior, outbound send routing, and Claude-style channel + permission notifications over the real stdio bridge.

## [​](https://docs.openclaw.ai/reference/test\#local-pr-gate)  Local PR gate

For local PR land/gate checks, run:

- `pnpm check:changed`
- `pnpm check`
- `pnpm check:test-types`
- `pnpm build`
- `pnpm test`
- `pnpm check:docs`

If `pnpm test` flakes on a loaded host, rerun once before treating it as a regression, then isolate with `pnpm test <path/to/test>`. For memory-constrained hosts, use:

- `OPENCLAW_VITEST_MAX_WORKERS=1 pnpm test`
- `OPENCLAW_VITEST_FS_MODULE_CACHE_PATH=/tmp/openclaw-vitest-cache pnpm test:changed`

## [​](https://docs.openclaw.ai/reference/test\#model-latency-bench-local-keys)  Model latency bench (local keys)

Script: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)Usage:

- `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
- Optional env: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
- Default prompt: “Reply with a single word: ok. No punctuation or extra text.”

Last run (2025-12-31, 20 runs):

- minimax median 1279ms (min 1114, max 2431)
- opus median 2454ms (min 1224, max 3170)

## [​](https://docs.openclaw.ai/reference/test\#cli-startup-bench)  CLI startup bench

Script: [`scripts/bench-cli-startup.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-cli-startup.ts)Usage:

- `pnpm test:startup:bench`
- `pnpm test:startup:bench:smoke`
- `pnpm test:startup:bench:save`
- `pnpm test:startup:bench:update`
- `pnpm test:startup:bench:check`
- `pnpm tsx scripts/bench-cli-startup.ts`
- `pnpm tsx scripts/bench-cli-startup.ts --runs 12`
- `pnpm tsx scripts/bench-cli-startup.ts --preset real`
- `pnpm tsx scripts/bench-cli-startup.ts --preset real --case status --case gatewayStatus --runs 3`
- `pnpm tsx scripts/bench-cli-startup.ts --preset real --case tasksJson --case tasksListJson --case tasksAuditJson --runs 3`
- `pnpm tsx scripts/bench-cli-startup.ts --entry openclaw.mjs --entry-secondary dist/entry.js --preset all`
- `pnpm tsx scripts/bench-cli-startup.ts --preset all --output .artifacts/cli-startup-bench-all.json`
- `pnpm tsx scripts/bench-cli-startup.ts --preset real --case gatewayStatusJson --output .artifacts/cli-startup-bench-smoke.json`
- `pnpm tsx scripts/bench-cli-startup.ts --preset real --cpu-prof-dir .artifacts/cli-cpu`
- `pnpm tsx scripts/bench-cli-startup.ts --json`

Presets:

- `startup`: `--version`, `--help`, `health`, `health --json`, `status --json`, `status`
- `real`: `health`, `status`, `status --json`, `sessions`, `sessions --json`, `tasks --json`, `tasks list --json`, `tasks audit --json`, `agents list --json`, `gateway status`, `gateway status --json`, `gateway health --json`, `config get gateway.port`
- `all`: both presets

Output includes `sampleCount`, avg, p50, p95, min/max, exit-code/signal distribution, and max RSS summaries for each command. Optional `--cpu-prof-dir` / `--heap-prof-dir` writes V8 profiles per run so timing and profile capture use the same harness.Saved output conventions:

- `pnpm test:startup:bench:smoke` writes the targeted smoke artifact at `.artifacts/cli-startup-bench-smoke.json`
- `pnpm test:startup:bench:save` writes the full-suite artifact at `.artifacts/cli-startup-bench-all.json` using `runs=5` and `warmup=1`
- `pnpm test:startup:bench:update` refreshes the checked-in baseline fixture at `test/fixtures/cli-startup-bench.json` using `runs=5` and `warmup=1`

Checked-in fixture:

- `test/fixtures/cli-startup-bench.json`
- Refresh with `pnpm test:startup:bench:update`
- Compare current results against the fixture with `pnpm test:startup:bench:check`

## [​](https://docs.openclaw.ai/reference/test\#onboarding-e2e-docker)  Onboarding E2E (Docker)

Docker is optional; this is only needed for containerized onboarding smoke tests.Full cold-start flow in a clean Linux container:

```
scripts/e2e/onboard-docker.sh
```

This script drives the interactive wizard via a pseudo-tty, verifies config/workspace/session files, then starts the gateway and runs `openclaw health`.

## [​](https://docs.openclaw.ai/reference/test\#qr-import-smoke-docker)  QR import smoke (Docker)

Ensures the maintained QR runtime helper loads under the supported Docker Node runtimes (Node 24 default, Node 22 compatible):

```
pnpm test:docker:qr
```

## [​](https://docs.openclaw.ai/reference/test\#related)  Related

- [Testing](https://docs.openclaw.ai/help/testing)
- [Testing live](https://docs.openclaw.ai/help/testing-live)

[Release policy](https://docs.openclaw.ai/reference/RELEASING) [CI pipeline](https://docs.openclaw.ai/ci)

Ctrl+I

---

## TOOLS.md template - OpenClaw
**Source:** https://docs.openclaw.ai/reference/templates/TOOLS

[Skip to main content](https://docs.openclaw.ai/reference/templates/TOOLS#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Templates

TOOLS.md template

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [TOOLS.md - Local Notes](https://docs.openclaw.ai/reference/templates/TOOLS#tools-md-local-notes)
- [What Goes Here](https://docs.openclaw.ai/reference/templates/TOOLS#what-goes-here)
- [Examples](https://docs.openclaw.ai/reference/templates/TOOLS#examples)
- [Why Separate?](https://docs.openclaw.ai/reference/templates/TOOLS#why-separate)
- [Related](https://docs.openclaw.ai/reference/templates/TOOLS#related)

# [​](https://docs.openclaw.ai/reference/templates/TOOLS\\#tools-md-local-notes)  TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that’s unique to your setup.

## [​](https://docs.openclaw.ai/reference/templates/TOOLS\\#what-goes-here)  What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## [​](https://docs.openclaw.ai/reference/templates/TOOLS\\#examples)  Examples

```
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: \"Nova\" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## [​](https://docs.openclaw.ai/reference/templates/TOOLS\\#why-separate)  Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

* * *

Add whatever helps you do your job. This is your cheat sheet.

## [​](https://docs.openclaw.ai/reference/templates/TOOLS\\#related)  Related

- [Agent workspace](https://docs.openclaw.ai/concepts/agent-workspace)

[SOUL.md template](https://docs.openclaw.ai/reference/templates/SOUL) [USER template](https://docs.openclaw.ai/reference/templates/USER)

Ctrl+I

---

## Prompt caching
**Source:** https://docs.openclaw.ai/reference/prompt-caching

[Skip to main content](https://docs.openclaw.ai/reference/prompt-caching#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Technical reference

Prompt caching

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Primary knobs](https://docs.openclaw.ai/reference/prompt-caching#primary-knobs)
- [cacheRetention (global default, model, and per-agent)](https://docs.openclaw.ai/reference/prompt-caching#cacheretention-global-default-model-and-per-agent)
- [contextPruning.mode: \"cache-ttl\"](https://docs.openclaw.ai/reference/prompt-caching#contextpruning-mode-cache-ttl)
- [Heartbeat keep-warm](https://docs.openclaw.ai/reference/prompt-caching#heartbeat-keep-warm)
- [Provider behavior](https://docs.openclaw.ai/reference/prompt-caching#provider-behavior)
- [Anthropic (direct API)](https://docs.openclaw.ai/reference/prompt-caching#anthropic-direct-api)
- [OpenAI (direct API)](https://docs.openclaw.ai/reference/prompt-caching#openai-direct-api)
- [Anthropic Vertex](https://docs.openclaw.ai/reference/prompt-caching#anthropic-vertex)
- [Amazon Bedrock](https://docs.openclaw.ai/reference/prompt-caching#amazon-bedrock)
- [OpenRouter models](https://docs.openclaw.ai/reference/prompt-caching#openrouter-models)
- [Other providers](https://docs.openclaw.ai/reference/prompt-caching#other-providers)
- [Google Gemini direct API](https://docs.openclaw.ai/reference/prompt-caching#google-gemini-direct-api)
- [Gemini CLI JSON usage](https://docs.openclaw.ai/reference/prompt-caching#gemini-cli-json-usage)
- [System-prompt cache boundary](https://docs.openclaw.ai/reference/prompt-caching#system-prompt-cache-boundary)
- [OpenClaw cache-stability guards](https://docs.openclaw.ai/reference/prompt-caching#openclaw-cache-stability-guards)
- [Tuning patterns](https://docs.openclaw.ai/reference/prompt-caching#tuning-patterns)
- [Mixed traffic (recommended default)](https://docs.openclaw.ai/reference/prompt-caching#mixed-traffic-recommended-default)
- [Cost-first baseline](https://docs.openclaw.ai/reference/prompt-caching#cost-first-baseline)
- [Cache diagnostics](https://docs.openclaw.ai/reference/prompt-caching#cache-diagnostics)
- [Live regression tests](https://docs.openclaw.ai/reference/prompt-caching#live-regression-tests)
- [Anthropic live expectations](https://docs.openclaw.ai/reference/prompt-caching#anthropic-live-expectations)
- [OpenAI live expectations](https://docs.openclaw.ai/reference/prompt-caching#openai-live-expectations)
- [diagnostics.cacheTrace config](https://docs.openclaw.ai/reference/prompt-caching#diagnostics-cachetrace-config)
- [Env toggles (one-off debugging)](https://docs.openclaw.ai/reference/prompt-caching#env-toggles-one-off-debugging)
- [What to inspect](https://docs.openclaw.ai/reference/prompt-caching#what-to-inspect)
- [Quick troubleshooting](https://docs.openclaw.ai/reference/prompt-caching#quick-troubleshooting)
- [Related](https://docs.openclaw.ai/reference/prompt-caching#related)

Prompt caching means the model provider can reuse unchanged prompt prefixes (usually system/developer instructions and other stable context) across turns instead of re-processing them every time. OpenClaw normalizes provider usage into `cacheRead` and `cacheWrite` where the upstream API exposes those counters directly.Status surfaces can also recover cache counters from the most recent transcript\nusage log when the live session snapshot is missing them, so `/status` can keep\nshowing a cache line after partial session metadata loss. Existing nonzero live\ncache values still take precedence over transcript fallback values.Why this matters: lower token cost, faster responses, and more predictable performance for long-running sessions. Without caching, repeated prompts pay the full prompt cost on every turn even when most input did not change.The sections below cover every cache-related knob that affects prompt reuse and token cost.Provider references:\n
- Anthropic prompt caching: [https://platform.claude.com/docs/en/build-with-claude/prompt-caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)\n- OpenAI prompt caching: [https://developers.openai.com/api/docs/guides/prompt-caching](https://developers.openai.com/api/docs/guides/prompt-caching)\n- OpenAI API headers and request IDs: [https://developers.openai.com/api/reference/overview](https://developers.openai.com/api/reference/overview)\n- Anthropic request IDs and errors: [https://platform.claude.com/docs/en/api/errors](https://platform.claude.com/docs/en/api/errors)\n\n## [​](https://docs.openclaw.ai/reference/prompt-caching\\#primary-knobs)  Primary knobs\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#cacheretention-global-default-model-and-per-agent)  `cacheRetention` (global default, model, and per-agent)\n\nSet cache retention as a global default for all models:\n\n```\nagents:\n  defaults:\n    params:\n      cacheRetention: \"long\" # none | short | long\n```\n\nOverride per-model:\n\n```\nagents:\n  defaults:\n    models:\n      \"anthropic/claude-opus-4-6\":\n        params:\n          cacheRetention: \"short\" # none | short | long\n```\n\nPer-agent override:\n\n```\nagents:\n  list:\n    - id: \"alerts\"\n      params:\n        cacheRetention: \"none\"\n```\n\nConfig merge order:\n\n1. `agents.defaults.params` (global default — applies to all models)\n2. `agents.defaults.models[\"provider/model\"].params` (per-model override)\n3. `agents.list[].params` (matching agent id; overrides by key)\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#contextpruning-mode-cache-ttl)  `contextPruning.mode: \"cache-ttl\"`\n\nPrunes old tool-result context after cache TTL windows so post-idle requests do not re-cache oversized history.\n\n```\nagents:\n  defaults:\n    contextPruning:\n      mode: \"cache-ttl\"\n      ttl: \"1h\"\n```\n\nSee [Session Pruning](https://docs.openclaw.ai/concepts/session-pruning) for full behavior.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#heartbeat-keep-warm)  Heartbeat keep-warm\n\nHeartbeat can keep cache windows warm and reduce repeated cache writes after idle gaps.\n\n```\nagents:\n  defaults:\n    heartbeat:\n      every: \"55m\"\n```\n\nPer-agent heartbeat is supported at `agents.list[].heartbeat`.\n\n## [​](https://docs.openclaw.ai/reference/prompt-caching\\#provider-behavior)  Provider behavior\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#anthropic-direct-api)  Anthropic (direct API)\n\n- `cacheRetention` is supported.\n- With Anthropic API-key auth profiles, OpenClaw seeds `cacheRetention: \"short\"` for Anthropic model refs when unset.\n- Anthropic native Messages responses expose both `cache_read_input_tokens` and `cache_creation_input_tokens`, so OpenClaw can show both `cacheRead` and `cacheWrite`.\n- For native Anthropic requests, `cacheRetention: \"short\"` maps to the default 5-minute ephemeral cache, and `cacheRetention: \"long\"` upgrades to the 1-hour TTL only on direct `api.anthropic.com` hosts.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#openai-direct-api)  OpenAI (direct API)\n\n- Prompt caching is automatic on supported recent models. OpenClaw does not need to inject block-level cache markers.\n- OpenClaw uses `prompt_cache_key` to keep cache routing stable across turns and uses `prompt_cache_retention: \"24h\"` only when `cacheRetention: \"long\"` is selected on direct OpenAI hosts.\n- OpenAI-compatible Completions providers receive `prompt_cache_key` only when their model config explicitly sets `compat.supportsPromptCacheKey: true`; `cacheRetention: \"none\"` still suppresses it.\n- OpenAI responses expose cached prompt tokens via `usage.prompt_tokens_details.cached_tokens` (or `input_tokens_details.cached_tokens` on Responses API events). OpenClaw maps that to `cacheRead`.\n- OpenAI does not expose a separate cache-write token counter, so `cacheWrite` stays `0` on OpenAI paths even when the provider is warming a cache.\n- OpenAI returns useful tracing and rate-limit headers such as `x-request-id`, `openai-processing-ms`, and `x-ratelimit-*`, but cache-hit accounting should come from the usage payload, not from headers.\n- In practice, OpenAI often behaves like an initial-prefix cache rather than Anthropic-style moving full-history reuse. Stable long-prefix text turns can land near a `4864` cached-token plateau in current live probes, while tool-heavy or MCP-style transcripts often plateau near `4608` cached tokens even on exact repeats.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#anthropic-vertex)  Anthropic Vertex\n\n- Anthropic models on Vertex AI (`anthropic-vertex/*`) support `cacheRetention` the same way as direct Anthropic.\n- `cacheRetention: \"long\"` maps to the real 1-hour prompt-cache TTL on Vertex AI endpoints.\n- Default cache retention for `anthropic-vertex` matches direct Anthropic defaults.\n- Vertex requests are routed through boundary-aware cache shaping so cache reuse stays aligned with what providers actually receive.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#amazon-bedrock)  Amazon Bedrock\n\n- Anthropic Claude model refs (`amazon-bedrock/*anthropic.claude*`) support explicit `cacheRetention` pass-through.\n- Non-Anthropic Bedrock models are forced to `cacheRetention: \"none\"` at runtime.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#openrouter-models)  OpenRouter models\n\nFor `openrouter/anthropic/*` model refs, OpenClaw injects Anthropic\n`cache_control` on system/developer prompt blocks to improve prompt-cache\nreuse only when the request is still targeting a verified OpenRouter route\n(`openrouter` on its default endpoint, or any provider/base URL that resolves\nto `openrouter.ai`).For `openrouter/deepseek/*`, `openrouter/moonshot*/*`, and `openrouter/zai/*`\nmodel refs, `contextPruning.mode: \"cache-ttl\"` is allowed because OpenRouter\nhandles provider-side prompt caching automatically. OpenClaw does not inject\nAnthropic `cache_control` markers into those requests.DeepSeek cache construction is best-effort and can take a few seconds. An\nimmediate follow-up may still show `cached_tokens: 0`; verify with a repeated\nsame-prefix request after a short delay and use `usage.prompt_tokens_details.cached_tokens`\nas the cache-hit signal.If you repoint the model at an arbitrary OpenAI-compatible proxy URL, OpenClaw\nstops injecting those OpenRouter-specific Anthropic cache markers.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#other-providers)  Other providers\n\nIf the provider does not support this cache mode, `cacheRetention` has no effect.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#google-gemini-direct-api)  Google Gemini direct API\n\n- Direct Gemini transport (`api: \"google-generative-ai\"`) reports cache hits\nthrough upstream `cachedContentTokenCount`; OpenClaw maps that to `cacheRead`.\n- When `cacheRetention` is set on a direct Gemini model, OpenClaw automatically\ncreates, reuses, and refreshes `cachedContents` resources for system prompts\non Google AI Studio runs. This means you no longer need to pre-create a\ncached-content handle manually.\n- You can still pass a pre-existing Gemini cached-content handle through as\n`params.cachedContent` (or legacy `params.cached_content`) on the configured\nmodel.\n- This is separate from Anthropic/OpenAI prompt-prefix caching. For Gemini,\nOpenClaw manages a provider-native `cachedContents` resource rather than\ninjecting cache markers into the request.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#gemini-cli-json-usage)  Gemini CLI JSON usage\n\n- Gemini CLI JSON output can also surface cache hits through `stats.cached`;\nOpenClaw maps that to `cacheRead`.\n- If the CLI omits a direct `stats.input` value, OpenClaw derives input tokens\nfrom `stats.input_tokens - stats.cached`.\n- This is usage normalization only. It does not mean OpenClaw is creating\nAnthropic/OpenAI-style prompt-cache markers for Gemini CLI.\n\n## [​](https://docs.openclaw.ai/reference/prompt-caching\\#system-prompt-cache-boundary)  System-prompt cache boundary\n\nOpenClaw splits the system prompt into a **stable prefix** and a **volatile**\n**suffix** separated by an internal cache-prefix boundary. Content above the\nboundary (tool definitions, skills metadata, workspace files, and other\nrelatively static context) is ordered so it stays byte-identical across turns.\nContent below the boundary (for example `HEARTBEAT.md`, runtime timestamps, and\nother per-turn metadata) is allowed to change without invalidating the cached\nprefix.Key design choices:\n\n- Stable workspace project-context files are ordered before `HEARTBEAT.md` so\nheartbeat churn does not bust the stable prefix.\n- The boundary is applied across Anthropic-family, OpenAI-family, Google, and\nCLI transport shaping so all supported providers benefit from the same prefix\nstability.\n- Codex Responses and Anthropic Vertex requests are routed through\nboundary-aware cache shaping so cache reuse stays aligned with what providers\nactually receive.\n- System-prompt fingerprints are normalized (whitespace, line endings,\nhook-added context, runtime capability ordering) so semantically unchanged\nprompts share KV/cache across turns.\n\nIf you see unexpected `cacheWrite` spikes after a config or workspace change,\ncheck whether the change lands above or below the cache boundary. Moving\nvolatile content below the boundary (or stabilizing it) often resolves the\nissue.\n\n## [​](https://docs.openclaw.ai/reference/prompt-caching\\#openclaw-cache-stability-guards)  OpenClaw cache-stability guards\n\nOpenClaw also keeps several cache-sensitive payload shapes deterministic before\nthe request reaches the provider:\n\n- Bundle MCP tool catalogs are sorted deterministically before tool\nregistration, so `listTools()` order changes do not churn the tools block and\nbust prompt-cache prefixes.\n- Legacy sessions with persisted image blocks keep the **3 most recent**\n**completed turns** intact; older already-processed image blocks may be\nreplaced with a marker so image-heavy follow-ups do not keep re-sending large\nstale payloads.\n\n## [​](https://docs.openclaw.ai/reference/prompt-caching\\#tuning-patterns)  Tuning patterns\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#mixed-traffic-recommended-default)  Mixed traffic (recommended default)\n\nKeep a long-lived baseline on your main agent, disable caching on bursty notifier agents:\n\n```\nagents:\n  defaults:\n    model:\n      primary: \"anthropic/claude-opus-4-6\"\n    models:\n      \"anthropic/claude-opus-4-6\":\n        params:\n          cacheRetention: \"long\"\n  list:\n    - id: \"research\"\n      default: true\n      heartbeat:\n        every: \"55m\"\n    - id: \"alerts\"\n      params:\n        cacheRetention: \"none\"\n```\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#cost-first-baseline)  Cost-first baseline\n\n- Set baseline `cacheRetention: \"short\"`.\n- Enable `contextPruning.mode: \"cache-ttl\"`.\n- Keep heartbeat below your TTL only for agents that benefit from warm caches.\n\n## [​](https://docs.openclaw.ai/reference/prompt-caching\\#cache-diagnostics)  Cache diagnostics\n\nOpenClaw exposes dedicated cache-trace diagnostics for embedded agent runs.For normal user-facing diagnostics, `/status` and other usage summaries can use\nthe latest transcript usage entry as a fallback source for `cacheRead` /\n`cacheWrite` when the live session entry does not have those counters.\n\n## [​](https://docs.openclaw.ai/reference/prompt-caching\\#live-regression-tests)  Live regression tests\n\nOpenClaw keeps one combined live cache regression gate for repeated prefixes, tool turns, image turns, MCP-style tool transcripts, and an Anthropic no-cache control.\n\n- `src/agents/live-cache-regression.live.test.ts`\n- `src/agents/live-cache-regression-baseline.ts`\n\nRun the narrow live gate with:\n\n```\nOPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_CACHE_TEST=1 pnpm test:live:cache\n```\n\nThe baseline file stores the most recent observed live numbers plus the provider-specific regression floors used by the test.\nThe runner also uses fresh per-run session IDs and prompt namespaces so previous cache state does not pollute the current regression sample.These tests intentionally do not use identical success criteria across providers.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#anthropic-live-expectations)  Anthropic live expectations\n\n- Expect explicit warmup writes via `cacheWrite`.\n- Expect near-full history reuse on repeated turns because Anthropic cache control advances the cache breakpoint through the conversation.\n- Current live assertions still use high hit-rate thresholds for stable, tool, and image paths.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#openai-live-expectations)  OpenAI live expectations\n\n- Expect `cacheRead` only. `cacheWrite` remains `0`.\n- Treat repeated-turn cache reuse as a provider-specific plateau, not as Anthropic-style moving full-history reuse.\n- Current live assertions use conservative floor checks derived from observed live behavior on `gpt-5.4-mini`:\n\n  - stable prefix: `cacheRead >= 4608`, hit rate `>= 0.90`\n  - tool transcript: `cacheRead >= 4096`, hit rate `>= 0.85`\n  - image transcript: `cacheRead >= 3840`, hit rate `>= 0.82`\n  - MCP-style transcript: `cacheRead >= 4096`, hit rate `>= 0.85`\n\nFresh combined live verification on 2026-04-04 landed at:\n\n- stable prefix: `cacheRead=4864`, hit rate `0.966`\n- tool transcript: `cacheRead=4608`, hit rate `0.896`\n- image transcript: `cacheRead=4864`, hit rate `0.954`\n- MCP-style transcript: `cacheRead=4608`, hit rate `0.891`\n\nRecent local wall-clock time for the combined gate was about `88s`.Why the assertions differ:\n\n- Anthropic exposes explicit cache breakpoints and moving conversation-history reuse.\n- OpenAI prompt caching is still exact-prefix sensitive, but the effective reusable prefix in live Responses traffic can plateau earlier than the full prompt.\n- Because of that, comparing Anthropic and OpenAI by a single cross-provider percentage threshold creates false regressions.\n\n### [​](https://docs.openclaw.ai/reference/prompt-caching\\#diagnostics-cachetrace-config)  `diagnostics.cacheTrace` config\n\n```\ndiagnostics:\n  cacheTrace:\n    enabled: true\n    filePath: \"~/.openclaw/logs/cache-trace.jsonl\" # optional\n    includeMessages: false # default true\n    includePrompt: false # default true\n    includeSystem: false # default true\n```\n\nDefaults:\n\n- `filePath`: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`\n- `includeMessages`...(content truncated)

---

## Onboarding reference - OpenClaw
**Source:** https://docs.openclaw.ai/reference/wizard

[Skip to main content](https://docs.openclaw.ai/reference/wizard#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Technical reference

Onboarding reference

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Flow details (local mode)](https://docs.openclaw.ai/reference/wizard#flow-details-local-mode)
- [Non-interactive mode](https://docs.openclaw.ai/reference/wizard#non-interactive-mode)
- [Add agent (non-interactive)](https://docs.openclaw.ai/reference/wizard#add-agent-non-interactive)
- [Gateway wizard RPC](https://docs.openclaw.ai/reference/wizard#gateway-wizard-rpc)
- [Signal setup (signal-cli)](https://docs.openclaw.ai/reference/wizard#signal-setup-signal-cli)
- [What the wizard writes](https://docs.openclaw.ai/reference/wizard#what-the-wizard-writes)
- [Related docs](https://docs.openclaw.ai/reference/wizard#related-docs)

This is the full reference for `openclaw onboard`.
For a high-level overview, see [Onboarding (CLI)](https://docs.openclaw.ai/start/wizard).

## [​](https://docs.openclaw.ai/reference/wizard\\#flow-details-local-mode)  Flow details (local mode)

1

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Existing config detection

- If `~/.openclaw/openclaw.json` exists, choose **Keep / Modify / Reset**.
- Re-running onboarding does **not** wipe anything unless you explicitly choose **Reset**
(or pass `--reset`).
- CLI `--reset` defaults to `config+creds+sessions`; use `--reset-scope full`
to also remove workspace.
- If the config is invalid or contains legacy keys, the wizard stops and asks
you to run `openclaw doctor` before continuing.
- Reset uses `trash` (never `rm`) and offers scopes:

  - Config only
  - Config + credentials + sessions
  - Full reset (also removes workspace)

2

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Model/Auth

- **Anthropic API key**: uses `ANTHROPIC_API_KEY` if present or prompts for a key, then saves it for daemon use.
- **Anthropic API key**: preferred Anthropic assistant choice in onboarding/configure.
- **Anthropic setup-token**: still available in onboarding/configure, though OpenClaw now prefers Claude CLI reuse when available.
- **OpenAI Code (Codex) subscription (OAuth)**: browser flow; paste the `code#state`.

  - Sets `agents.defaults.model` to `openai-codex/gpt-5.5` when model is unset or already OpenAI-family.
- **OpenAI Code (Codex) subscription (device pairing)**: browser pairing flow with a short-lived device code.

  - Sets `agents.defaults.model` to `openai-codex/gpt-5.5` when model is unset or already OpenAI-family.
- **OpenAI API key**: uses `OPENAI_API_KEY` if present or prompts for a key, then stores it in auth profiles.

  - Sets `agents.defaults.model` to `openai/gpt-5.5` when model is unset, `openai/*`, or `openai-codex/*`.
- **xAI (Grok) API key**: prompts for `XAI_API_KEY` and configures xAI as a model provider.
- **OpenCode**: prompts for `OPENCODE_API_KEY` (or `OPENCODE_ZEN_API_KEY`, get it at [https://opencode.ai/auth](https://opencode.ai/auth)) and lets you pick the Zen or Go catalog.
- **Ollama**: offers **Cloud + Local**, **Cloud only**, or **Local only** first. `Cloud only` prompts for `OLLAMA_API_KEY` and uses `https://ollama.com`; the host-backed modes prompt for the Ollama base URL, discover available models, and auto-pull the selected local model when needed; `Cloud + Local` also checks whether that Ollama host is signed in for cloud access.
- More detail: [Ollama](https://docs.openclaw.ai/providers/ollama)
- **API key**: stores the key for you.
- **Vercel AI Gateway (multi-model proxy)**: prompts for `AI_GATEWAY_API_KEY`.
- More detail: [Vercel AI Gateway](https://docs.openclaw.ai/providers/vercel-ai-gateway)
- **Cloudflare AI Gateway**: prompts for Account ID, Gateway ID, and `CLOUDFLARE_AI_GATEWAY_API_KEY`.
- More detail: [Cloudflare AI Gateway](https://docs.openclaw.ai/providers/cloudflare-ai-gateway)
- **MiniMax**: config is auto-written; hosted default is `MiniMax-M2.7`.
API-key setup uses `minimax/...`, and OAuth setup uses
`minimax-portal/...`.
- More detail: [MiniMax](https://docs.openclaw.ai/providers/minimax)
- **StepFun**: config is auto-written for StepFun standard or Step Plan on China or global endpoints.
- Standard currently includes `step-3.5-flash`, and Step Plan also includes `step-3.5-flash-2603`.
- More detail: [StepFun](https://docs.openclaw.ai/providers/stepfun)
- **Synthetic (Anthropic-compatible)**: prompts for `SYNTHETIC_API_KEY`.
- More detail: [Synthetic](https://docs.openclaw.ai/providers/synthetic)
- **Moonshot (Kimi K2)**: config is auto-written.
- **Kimi Coding**: config is auto-written.
- More detail: [Moonshot AI (Kimi + Kimi Coding)](https://docs.openclaw.ai/providers/moonshot)
- **Skip**: no auth configured yet.
- Pick a default model from detected options (or enter provider/model manually). For best quality and lower prompt-injection risk, choose the strongest latest-generation model available in your provider stack.
- Onboarding runs a model check and warns if the configured model is unknown or missing auth.
- API key storage mode defaults to plaintext auth-profile values. Use `--secret-input-mode ref` to store env-backed refs instead (for example `keyRef: { source: \"env\", provider: \"default\", id: \"OPENAI_API_KEY\" }`).
- Auth profiles live in `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (API keys + OAuth). `~/.openclaw/credentials/oauth.json` is legacy import-only.
- More detail: [/concepts/oauth](https://docs.openclaw.ai/concepts/oauth)

Headless/server tip: complete OAuth on a machine with a browser, then copy
that agent’s `auth-profiles.json` (for example
`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`, or the matching
`$OPENCLAW_STATE_DIR/...` path) to the gateway host. `credentials/oauth.json`
is only a legacy import source.

3

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Workspace

- Default `~/.openclaw/workspace` (configurable).
- Seeds the workspace files needed for the agent bootstrap ritual.
- Full workspace layout + backup guide: [Agent workspace](https://docs.openclaw.ai/concepts/agent-workspace)

4

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Gateway

- Port, bind, auth mode, tailscale exposure.
- Auth recommendation: keep **Token** even for loopback so local WS clients must authenticate.
- In token mode, interactive setup offers:
  - **Generate/store plaintext token** (default)
  - **Use SecretRef** (opt-in)
  - Quickstart reuses existing `gateway.auth.token` SecretRefs across `env`, `file`, and `exec` providers for onboarding probe/dashboard bootstrap.
  - If that SecretRef is configured but cannot be resolved, onboarding fails early with a clear fix message instead of silently degrading runtime auth.
  - If token auth requires a token and the configured token SecretRef is unresolved, daemon install is blocked with actionable guidance.
  - If both `gateway.auth.token` and `gateway.auth.password` are configured and `gateway.auth.mode` is unset, daemon install is blocked until mode is set explicitly.
- In password mode, interactive setup also supports plaintext or SecretRef storage.
- Non-interactive token SecretRef path: `--gateway-token-ref-env <ENV_VAR>`.

  - Requires a non-empty env var in the onboarding process environment.
  - Cannot be combined with `--gateway-token`.
- Disable auth only if you fully trust every local process.
- Non‑loopback binds still require auth.

5

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Channels

- [WhatsApp](https://docs.openclaw.ai/channels/whatsapp): optional QR login.
- [Telegram](https://docs.openclaw.ai/channels/telegram): bot token.
- [Discord](https://docs.openclaw.ai/channels/discord): bot token.
- [Google Chat](https://docs.openclaw.ai/channels/googlechat): service account JSON + webhook audience.
- [Mattermost](https://docs.openclaw.ai/channels/mattermost) (plugin): bot token + base URL.
- [Signal](https://docs.openclaw.ai/channels/signal): optional `signal-cli` install + account config.
- [BlueBubbles](https://docs.openclaw.ai/channels/bluebubbles): **recommended for iMessage**; server URL + password + webhook.
- [iMessage](https://docs.openclaw.ai/channels/imessage): legacy `imsg` CLI path + DB access.
- DM security: default is pairing. First DM sends a code; approve via `openclaw pairing approve <channel> <code>` or use allowlists.

6

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Web search

- Pick a supported provider such as Brave, DuckDuckGo, Exa, Firecrawl, Gemini, Grok, Kimi, MiniMax Search, Ollama Web Search, Perplexity, SearXNG, or Tavily (or skip).
- API-backed providers can use env vars or existing config for quick setup; key-free providers use their provider-specific prerequisites instead.
- Skip with `--skip-search`.
- Configure later: `openclaw configure --section web`.

7

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Daemon install

- macOS: LaunchAgent
  - Requires a logged-in user session; for headless, use a custom LaunchDaemon (not shipped).
- Linux (and Windows via WSL2): systemd user unit
  - Onboarding attempts to enable lingering via `loginctl enable-linger <user>` so the Gateway stays up after logout.
  - May prompt for sudo (writes `/var/lib/systemd/linger`); it tries without sudo first.
- **Runtime selection:** Node (recommended; required for WhatsApp/Telegram). Bun is **not recommended**.
- If token auth requires a token and `gateway.auth.token` is SecretRef-managed, daemon install validates it but does not persist resolved plaintext token values into supervisor service environment metadata.
- If token auth requires a token and the configured token SecretRef is unresolved, daemon install is blocked with actionable guidance.
- If both `gateway.auth.token` and `gateway.auth.password` are configured and `gateway.auth.mode` is unset, daemon install is blocked until mode is set explicitly.

8

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Health check

- Starts the Gateway (if needed) and runs `openclaw health`.
- Tip: `openclaw status --deep` adds the live gateway health probe to status output, including channel probes when supported (requires a reachable gateway).

9

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Skills (recommended)

- Reads the available skills and checks requirements.
- Lets you choose a node manager: **npm / pnpm** (bun not recommended).
- Installs optional dependencies (some use Homebrew on macOS).

10

[Navigate to header](https://docs.openclaw.ai/reference/wizard#)

Finish

- Summary + next steps, including iOS/Android/macOS apps for extra features.

If no GUI is detected, onboarding prints SSH port-forward instructions for the Control UI instead of opening a browser.
If the Control UI assets are missing, onboarding attempts to build them; fallback is `pnpm ui:build` (auto-installs UI deps).

## [​](https://docs.openclaw.ai/reference/wizard\\#non-interactive-mode)  Non-interactive mode

Use `--non-interactive` to automate or script onboarding:

```
openclaw onboard --non-interactive \\\
  --mode local \\\
  --auth-choice apiKey \\\
  --anthropic-api-key \"$ANTHROPIC_API_KEY\" \\\
  --gateway-port 18789 \\\
  --gateway-bind loopback \\\
  --install-daemon \\\
  --daemon-runtime node \\\
  --skip-skills
```

Add `--json` for a machine‑readable summary.Gateway token SecretRef in non-interactive mode:

```
export OPENCLAW_GATEWAY_TOKEN=\"your-token\"
openclaw onboard --non-interactive \\\
  --mode local \\\
  --auth-choice skip \\\
  --gateway-auth token \\\
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN
```

`--gateway-token` and `--gateway-token-ref-env` are mutually exclusive.

`--json` does **not** imply non-interactive mode. Use `--non-interactive` (and `--workspace`) for scripts.

Provider-specific command examples live in [CLI Automation](https://docs.openclaw.ai/start/wizard-cli-automation#provider-specific-examples).
Use this reference page for flag semantics and step ordering.

### [​](https://docs.openclaw.ai/reference/wizard\\#add-agent-non-interactive)  Add agent (non-interactive)

```
openclaw agents add work \\\
  --workspace ~/.openclaw/workspace-work \\\
  --model openai/gpt-5.5 \\\
  --bind whatsapp:biz \\\
  --non-interactive \\\
  --json
```

## [​](https://docs.openclaw.ai/reference/wizard\\#gateway-wizard-rpc)  Gateway wizard RPC

The Gateway exposes the onboarding flow over RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Clients (macOS app, Control UI) can render steps without re‑implementing onboarding logic.

## [​](https://docs.openclaw.ai/reference/wizard\\#signal-setup-signal-cli)  Signal setup (signal-cli)

Onboarding can install `signal-cli` from GitHub releases:

- Downloads the appropriate release asset.
- Stores it under `~/.openclaw/tools/signal-cli/<version>/`.
- Writes `channels.signal.cliPath` to your config.

Notes:

- JVM builds require **Java 21**.
- Native builds are used when available.
- Windows uses WSL2; signal-cli install follows the Linux flow inside WSL.

## [​](https://docs.openclaw.ai/reference/wizard\\#what-the-wizard-writes)  What the wizard writes

Typical fields in `~/.openclaw/openclaw.json`:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (if Minimax chosen)
- `tools.profile` (local onboarding defaults to `\"coding\"` when unset; existing explicit values are preserved)
- `gateway.*` (mode, bind, auth, tailscale)
- `session.dmScope` (behavior details: [CLI Setup Reference](https://docs.openclaw.ai/start/wizard-cli-reference#outputs-and-internals))
- `channels.telegram.botToken`, `channels.discord.token`, `channels.matrix.*`, `channels.signal.*`, `channels.imessage.*`
- Channel allowlists (Slack/Discord/Matrix/Microsoft Teams) when you opt in during the prompts (names resolve to IDs when possible).
- `skills.install.nodeManager`
  - `setup --node-manager` accepts `npm`, `pnpm`, or `bun`.
  - Manual config can still use `yarn` by setting `skills.install.nodeManager` directly.
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` writes `agents.list[]` and optional `bindings`.WhatsApp credentials go under `~/.openclaw/credentials/whatsapp/<accountId>/`.\nSessions are stored under `~/.openclaw/agents/<agentId>/sessions/`.Some channels are delivered as plugins. When you pick one during setup, onboarding\nwill prompt to install it (npm or a local path) before it can be configured.\n
## [​](https://docs.openclaw.ai/reference/wizard\\#related-docs)  Related docs

- Onboarding overview: [Onboarding (CLI)](https://docs.openclaw.ai/start/wizard)
- macOS app onboarding: [Onboarding](https://docs.openclaw.ai/start/onboarding)
- Config reference: [Gateway configuration](https://docs.openclaw.ai/gateway/configuration)
- Providers: [WhatsApp](https://docs.openclaw.ai/channels/whatsapp), [Telegram](https://docs.openclaw.ai/channels/telegram), [Discord](https://docs.openclaw.ai/channels/discord), [Google Chat](https://docs.openclaw.ai/channels/googlechat), [Signal](https://docs.openclaw.ai/channels/signal), [BlueBubbles](https://docs.openclaw.ai/channels/bluebubbles) (iMessage), [iMessage](https://docs.openclaw.ai/channels/imessage) (legacy)
- Skills: [Skills](https://docs.openclaw.ai/tools/skills), [Skills config](https://docs.openclaw.ai/tools/skills-config)

[Pi integration architecture](https://docs.openclaw.ai/pi) [Token use and costs](https://docs.openclaw.ai/reference/token-use)

Ctrl+I

---

## Release and CI
**Source:** https://docs.openclaw.ai/reference/RELEASING

[Skip to main content](https://docs.openclaw.ai/reference/RELEASING#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nRelease and CI\n\nRelease policy\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://openclaw.ai/help)\n\nOn this page\n\n- [Version naming](https://docs.openclaw.ai/reference/RELEASING#version-naming)\n- [Release cadence](https://docs.openclaw.ai/reference/RELEASING#release-cadence)\n- [Release operator checklist](https://docs.openclaw.ai/reference/RELEASING#release-operator-checklist)\n- [Release preflight](https://docs.openclaw.ai/reference/RELEASING#release-preflight)\n- [Release test boxes](https://docs.openclaw.ai/reference/RELEASING#release-test-boxes)\n- [Vitest](https://docs.openclaw.ai/reference/RELEASING#vitest)\n- [Docker](https://docs.openclaw.ai/reference/RELEASING#docker)\n- [QA Lab](https://docs.openclaw.ai/reference/RELEASING#qa-lab)\n- [Package](https://docs.openclaw.ai/reference/RELEASING#package)\n- [NPM workflow inputs](https://docs.openclaw.ai/reference/RELEASING#npm-workflow-inputs)\n- [Stable npm release sequence](https://docs.openclaw.ai/reference/RELEASING#stable-npm-release-sequence)\n- [Public references](https://docs.openclaw.ai/reference/RELEASING#public-references)\n- [Related](https://docs.openclaw.ai/reference/RELEASING#related)\n\nOpenClaw has three public release lanes:\n\n- stable: tagged releases that publish to npm `beta` by default, or to npm `latest` when explicitly requested\n- beta: prerelease tags that publish to npm `beta`\n- dev: the moving head of `main`\n\n## [​](https://docs.openclaw.ai/reference/RELEASING\\#version-naming)  Version naming\n\n- Stable release version: `YYYY.M.D`\n  - Git tag: `vYYYY.M.D`\n- Stable correction release version: `YYYY.M.D-N`\n  - Git tag: `vYYYY.M.D-N`\n- Beta prerelease version: `YYYY.M.D-beta.N`\n  - Git tag: `vYYYY.M.D-beta.N`\n- Do not zero-pad month or day\n- `latest` means the current promoted stable npm release\n- `beta` means the current beta install target\n- Stable and stable correction releases publish to npm `beta` by default; release operators can target `latest` explicitly, or promote a vetted beta build later\n- Every stable OpenClaw release ships the npm package and macOS app together;\nbeta releases normally validate and publish the npm/package path first, with\nmac app build/sign/notarize reserved for stable unless explicitly requested\n\n## [​](https://docs.openclaw.ai/reference/RELEASING\\#release-cadence)  Release cadence\n\n- Releases move beta-first\n- Stable follows only after the latest beta is validated\n- Maintainers normally cut releases from a `release/YYYY.M.D` branch created\nfrom current `main`, so release validation and fixes do not block new\ndevelopment on `main`\n- If a beta tag has been pushed or published and needs a fix, maintainers cut\nthe next `-beta.N` tag instead of deleting or recreating the old beta tag\n- Detailed release procedure, approvals, credentials, and recovery notes are\nmaintainer-only\n\n## [​](https://docs.openclaw.ai/reference/RELEASING\\#release-operator-checklist)  Release operator checklist\n\nThis checklist is the public shape of the release flow. Private credentials,\nsigning, notarization, dist-tag recovery, and emergency rollback details stay in\nthe maintainer-only release runbook.\n\n01. Start from current `main`: pull latest, confirm the target commit is pushed,\n    and confirm current `main` CI is green enough to branch from it.\n02. Rewrite the top `CHANGELOG.md` section from real commit history with\n    `/changelog`, keep entries user-facing, commit it, push it, and rebase/pull\n    once more before branching.\n03. Review release compatibility records in\n    `src/plugins/compat/registry.ts` and\n    `src/commands/doctor/shared/deprecation-compat.ts`. Remove expired\n    compatibility only when the upgrade path stays covered, or record why it is\n    intentionally carried.\n04. Create `release/YYYY.M.D` from current `main`; do not do normal release work\n    directly on `main`.\n05. Bump every required version location for the intended tag, then run the\n    local deterministic preflight:\n    `pnpm check:test-types`, `pnpm check:architecture`,\n    `pnpm build && pnpm ui:build`, and `pnpm release:check`.\n06. Run `OpenClaw NPM Release` with `preflight_only=true`. Before a tag exists,\n    a full 40-character release-branch SHA is allowed for validation-only\n    preflight. Save the successful `preflight_run_id`.\n07. Kick off all pre-release tests with `Full Release Validation` for the\n    release branch, tag, or full commit SHA. This is the one manual entrypoint\n    for the four big release test boxes: Vitest, Docker, QA Lab, and Package.\n08. If validation fails, fix on the release branch and rerun the smallest failed\n    file, lane, workflow job, package profile, provider, or model allowlist that\n    proves the fix. Rerun the full umbrella only when the changed surface makes\n    prior evidence stale.\n09. For beta, tag `vYYYY.M.D-beta.N`, publish with npm dist-tag `beta`, then run\n    post-publish package acceptance against the published `openclaw@YYYY.M.D-beta.N`\n    or `openclaw@beta` package. If a pushed or published beta needs a fix, cut\n    the next `-beta.N`; do not delete or rewrite the old beta.\n10. For stable, continue only after the vetted beta or release candidate has the\n    required validation evidence. Stable npm publish reuses the successful\n    preflight artifact via `preflight_run_id`; stable macOS release readiness\n    also requires the packaged `.zip`, `.dmg`, `.dSYM.zip`, and updated\n    `appcast.xml` on `main`.\n11. After publish, run the npm post-publish verifier, optional standalone\n    published-npm Telegram E2E when you need post-publish channel proof,\n    dist-tag promotion when needed, GitHub release/prerelease notes from the\n    complete matching `CHANGELOG.md` section, and the release announcement\n    steps.\n\n## [​](https://docs.openclaw.ai/reference/RELEASING\\#release-preflight)  Release preflight\n\n- Run `pnpm check:test-types` before release preflight so test TypeScript stays\ncovered outside the faster local `pnpm check` gate\n- Run `pnpm check:architecture` before release preflight so the broader import\ncycle and architecture boundary checks are green outside the faster local gate\n- Run `pnpm build && pnpm ui:build` before `pnpm release:check` so the expected\n`dist/*` release artifacts and Control UI bundle exist for the pack\nvalidation step\n- Run the manual `Full Release Validation` workflow before release approval to\nkick off all pre-release test boxes from one entrypoint. It accepts a branch,\ntag, or full commit SHA, dispatches manual `CI`, and dispatches\n`OpenClaw Release Checks` for install smoke, package acceptance, Docker\nrelease-path suites, live/E2E, OpenWebUI, QA Lab parity, Matrix, and Telegram\nlanes. Provide `npm_telegram_package_spec` only after a package has been\npublished and the post-publish Telegram E2E should run too. Provide\n`evidence_package_spec` when the private evidence report should prove that the\nvalidation matches a published npm package without forcing Telegram E2E.\nExample:\n`gh workflow run full-release-validation.yml --ref main -f ref=release/YYYY.M.D`\n- Run the manual `Package Acceptance` workflow when you want side-channel proof\nfor a package candidate while release work continues. Use `source=npm` for\n`openclaw@beta`, `openclaw@latest`, or an exact release version; `source=ref`\nto pack a trusted `package_ref` branch/tag/SHA with the current\n`workflow_ref` harness; `source=url` for an HTTPS tarball with a required\nSHA-256; or `source=artifact` for a tarball uploaded by another GitHub\nActions run. The workflow resolves the candidate to\n`package-under-test`, reuses the Docker E2E release scheduler against that\ntarball, and can run Telegram QA against the same tarball with\n`telegram_mode=mock-openai` or `telegram_mode=live-frontier`.\nExample: `gh workflow run package-acceptance.yml --ref main -f workflow_ref=main -f source=npm -f package_spec=openclaw@beta -f suite_profile=product -f telegram_mode=mock-openai`\nCommon profiles:\n\n  - `smoke`: install/channel/agent, gateway network, and config reload lanes\n  - `package`: artifact-native package/update/plugin lanes without OpenWebUI or live ClawHub\n  - `product`: package profile plus MCP channels, cron/subagent cleanup,\n    OpenAI web search, and OpenWebUI\n  - `full`: Docker release-path chunks with OpenWebUI\n  - `custom`: exact `docker_lanes` selection for a focused rerun\n- Run the manual `CI` workflow directly when you only need full normal CI\ncoverage for the release candidate. Manual CI dispatches bypass changed\nscoping and force the Linux Node shards, bundled-plugin shards, channel\ncontracts, Node 22 compatibility, `check`, `check-additional`, build smoke,\ndocs checks, Python skills, Windows, macOS, Android, and Control UI i18n\nlanes.\nExample: `gh workflow run ci.yml --ref release/YYYY.M.D`\n- Run `pnpm qa:otel:smoke` when validating release telemetry. It exercises\nQA-lab through a local OTLP/HTTP receiver and verifies the exported trace\nspan names, bounded attributes, and content/identifier redaction without\nrequiring Opik, Langfuse, or another external collector.\n- Run `pnpm release:check` before every tagged release\n- Release checks now run in a separate manual workflow:\n`OpenClaw Release Checks`\n- `OpenClaw Release Checks` also runs the QA Lab mock parity gate plus the fast\nlive Matrix profile and Telegram QA lane before release approval. The live\nlanes use the `qa-live-shared` environment; Telegram also uses Convex CI\ncredential leases. Run the manual `QA-Lab - All Lanes` workflow with\n`matrix_profile=all` and `matrix_shards=true` when you want full Matrix\ntransport, media, and E2EE inventory in parallel.\n- Cross-OS install and upgrade runtime validation is part of public\n`OpenClaw Release Checks` and `Full Release Validation`, which call the\nreusable workflow\n`.github/workflows/openclaw-cross-os-release-checks-reusable.yml` directly\n- This split is intentional: keep the real npm release path short,\ndeterministic, and artifact-focused, while slower live checks stay in their\nown lane so they do not stall or block publish\n- Secret-bearing release checks should be dispatched through `Full Release Validation` or from the `main`/release workflow ref so workflow logic and\nsecrets stay controlled\n- `OpenClaw Release Checks` accepts a branch, tag, or full commit SHA as long\nas the resolved commit is reachable from an OpenClaw branch or release tag\n- `OpenClaw NPM Release` validation-only preflight also accepts the current\nfull 40-character workflow-branch commit SHA without requiring a pushed tag\n- That SHA path is validation-only and cannot be promoted into a real publish\n- In SHA mode the workflow synthesizes `v<package.json version>` only for the\npackage metadata check; real publish still requires a real release tag\n- Both workflows keep the real publish and promotion path on GitHub-hosted\nrunners, while the non-mutating validation path can use the larger\nBlacksmith Linux runners\n- That workflow runs\n`OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_CACHE_TEST=1 pnpm test:live:cache`\nusing both `OPENAI_API_KEY` and `ANTHROPIC_API_KEY` workflow secrets\n- npm release preflight no longer waits on the separate release checks lane\n- Run `RELEASE_TAG=vYYYY.M.D node --import tsx scripts/openclaw-npm-release-check.ts`\n(or the matching beta/correction tag) before approval\n- After npm publish, run\n`node --import tsx scripts/openclaw-npm-postpublish-verify.ts YYYY.M.D`\n(or the matching beta/correction version) to verify the published registry\ninstall path in a fresh temp prefix\n- After a beta publish, run `OPENCLAW_NPM_TELEGRAM_PACKAGE_SPEC=openclaw@YYYY.M.D-beta.N OPENCLAW_NPM_TELEGRAM_CREDENTIAL_SOURCE=convex OPENCLAW_NPM_TELEGRAM_CREDENTIAL_ROLE=ci pnpm test:docker:npm-telegram-live`\nto verify installed-package onboarding, Telegram setup, and real Telegram E2E\nagainst the published npm package using the shared leased Telegram credential\npool. Local maintainer one-offs may omit the Convex vars and pass the three\n`OPENCLAW_QA_TELEGRAM_*` env credentials directly.\n- Maintainers can run the same post-publish check from GitHub Actions via the\nmanual `NPM Telegram Beta E2E` workflow. It is intentionally manual-only and\ndoes not run on every merge.\n- Maintainer release automation now uses preflight-then-promote:\n  - real npm publish must pass a successful npm `preflight_run_id`\n  - the real npm publish must be dispatched from the same `main` or\n    `release/YYYY.M.D` branch as the successful preflight run\n  - stable npm releases default to `beta`\n  - stable npm publish can target `latest` explicitly via workflow input\n  - token-based npm dist-tag mutation now lives in\n    `openclaw/releases-private/.github/workflows/openclaw-npm-dist-tags.yml`\n    for security, because `npm dist-tag add` still needs `NPM_TOKEN` while the\n    public repo keeps OIDC-only publish\n  - public `macOS Release` is validation-only\n  - real private mac publish must pass successful private mac\n    `preflight_run_id` and `validate_run_id`\n  - the real publish paths promote prepared artifacts instead of rebuilding\n    them again\n- For stable correction releases like `YYYY.M.D-N`, the post-publish verifier\nalso checks the same temp-prefix upgrade path from `YYYY.M.D` to `YYYY.M.D-N`\nso release corrections cannot silently leave older global installs on the\nbase stable payload\n- npm release preflight fails closed unless the tarball includes both\n`dist/control-ui/index.html` and a non-empty `dist/control-ui/assets/` payload\nso we do not ship an empty browser dashboard again\n- Post-publish verification also checks that the published registry install\ncontains non-empty bundled plugin runtime deps under the root `dist/*`\nlayout. A release that ships with missing or empty bundled plugin\ndependency payloads fails the postpublish verifier and cannot be promoted\nto `latest`.\n- `pnpm test:install:smoke` also enforces the npm pack `unpackedSize` budget on\nthe candidate update tarball, so installer e2e catches accidental pack bloat\nbefore the release publish path\n- If the release work touched CI planning, extension timing manifests, or\nextension test matrices, regenerate and review the planner-owned\n`checks-node-extensions` workflow matrix outputs from `.github/workflows/ci.yml`\nbefore approval so release notes do not describe a stale CI layout\n- Stable macOS release readiness also includes the updater surfaces:\n  - the GitHub release must end up with the packaged `.zip`, `.dmg`, and `.dSYM.zip`\n  - `appcast.xml` on `main` must point at the new stable zip after publish\n  - the packaged app must keep a non-debug bundle id, a non-empty Sparkle feed\n    URL, and a `CFBundleVersion` at or above the canonical Sparkle build floor\n    for that release version\n\n## [​](https://docs.openclaw.ai/reference/RELEASING\\#release-test-boxes)  Release test boxes\n\n`Full Release Validation` is how operators kick off all pre-release tests from\none entrypoint. Run it from the trusted `main` workflow ref and pass the release\nbranch, tag, or full commit SHA as `ref`:\n\n```\ngh workflow run full-release-validation.yml \\\n  --ref main \\\n  -f ref=release/YYYY.M.D \\\n  -f provider=openai \\\n  -f mode=both \\\n  -f release_profile=full \\\n  -f evidence_package_spec=openclaw@YYYY.M.D-beta.N\n```\n\nThe workflow resolves the target ref, dispatches manual `CI` with\n`target_ref=<release-ref>`, dispatches `OpenClaw Release Checks`, and\noptionally dispatches standalone post-publish Telegram E2E when\n`npm_telegram_package_spec` is set. `OpenClaw Release Checks` then fans out\ninstall smoke, cross-OS release checks, live/E2E Docker release-path coverage,\nPackage Acceptance with Telegram package QA, QA Lab parity, live Matrix, and\nlive Telegram. A full run is only acceptable when the `Full Release Validation`\nsummary shows `normal_ci` and `release_checks` as successful, and any optional\n`npm_telegram` child is either successful or intentionally skipped. The final\nverifier summary includes slowest-job tables for each child run, so the release\nmanager can see the current critical path without downloading logs.\nChild workflows are dispatched from the trusted ref that runs `Full Release Validation`, normally `--ref main`, even when the target `ref` points at an\nolder release branch or tag. There is no separate Full Release Validation\nworkflow-ref input; choose the trusted harness by choosing the workflow run ref.Use `release_profile` to select live/provider breadth:\n\n- `minimum`: fastest release-critical OpenAI/core live and Docker path\n- `stable`: minimum plus stable provider/backend coverage for release approval\n- `full`: stable plus broad advisory provider/media coverage\n\n`OpenClaw Release Checks` uses the trusted workflow ref to resolve the target\nref once as `release-package-under-test` and reuses that artifact in both\nrelease-path Docker checks and Package Acceptance. This keeps all\npackage-facing boxes on the same bytes and avoids repeated package builds.Use these variants depending on release stage:\n\n```\n# Validate an unpublished release candidate branch.\ngh workflow run full-release-validation.yml \\\n  --ref main \\\n  -f ref=release/YYYY.M.D \\\n  -f provider=openai \\\n  -f mode=both \\\n  -f release_profile=stable\n\n# Validate an exact pushed commit.\ngh workflow run full-release-validation.yml \\\n  --ref main \\\n  -f ref=<40-char-sha> \\\n  -f provider=openai \\\n  -f mode=both\n\n# After publishing a beta, add published-package Telegram E2E.\ngh workflow run full-release-validation.yml \\\n  --ref main \\\n  -f ref=release/YYYY.M.D \\\n  -f provider=openai \\\n  -f mode=both \\\n  -f evidence_package_spec=openclaw@YYYY.M.D-beta.N \\\n  -f npm_telegram_package_spec=openclaw@YYYY.M.D-beta.N \\\n  -f npm_telegram_provider_mode=mock-openai\n```\n\nDo not use the full umbrella as the first rerun after a focused fix. If one box\nfails, use the failed child workflow, job, Docker lane, package profile, model\nprovider, or QA lane for the next proof. Run the full umbrella again only when\nthe fix changed shared release orchestration or made earlier all-box evidence\nstale. The umbrella’s final verifier re-checks the recorded child workflow run\nids, so after a child workflow is rerun successfully, rerun only the failed\n`Verify full validation` parent job.For bounded recovery, pass `rerun_group` to the umbrella. `all` is the real\nrelease-candidate run, `ci` runs only the normal CI child, `release-checks` runs\nevery release box, and the narrower release groups ...(content truncated)

---

## Default AGENTS.md - OpenClaw
**Source:** https://docs.openclaw.ai/reference/AGENTS.default

[Skip to main content](https://docs.openclaw.ai/reference/AGENTS.default#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Templates

Default AGENTS.md

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [AGENTS.md - OpenClaw Personal Assistant (default)](https://docs.openclaw.ai/reference/AGENTS.default#agents-md-openclaw-personal-assistant-default)
- [First run (recommended)](https://docs.openclaw.ai/reference/AGENTS.default#first-run-recommended)
- [Safety defaults](https://docs.openclaw.ai/reference/AGENTS.default#safety-defaults)
- [Session start (required)](https://docs.openclaw.ai/reference/AGENTS.default#session-start-required)
- [Soul (required)](https://docs.openclaw.ai/reference/AGENTS.default#soul-required)
- [Shared spaces (recommended)](https://docs.openclaw.ai/reference/AGENTS.default#shared-spaces-recommended)
- [Memory system (recommended)](https://docs.openclaw.ai/reference/AGENTS.default#memory-system-recommended)
- [Tools & skills](https://docs.openclaw.ai/reference/AGENTS.default#tools-%26-skills)
- [Backup tip (recommended)](https://docs.openclaw.ai/reference/AGENTS.default#backup-tip-recommended)
- [What OpenClaw does](https://docs.openclaw.ai/reference/AGENTS.default#what-openclaw-does)
- [Core skills (enable in Settings → Skills)](https://docs.openclaw.ai/reference/AGENTS.default#core-skills-enable-in-settings-%E2%86%92-skills)
- [Usage notes](https://docs.openclaw.ai/reference/AGENTS.default#usage-notes)
- [Related](https://docs.openclaw.ai/reference/AGENTS.default#related)

# [​](https://docs.openclaw.ai/reference/AGENTS.default\#agents-md-openclaw-personal-assistant-default)  AGENTS.md - OpenClaw Personal Assistant (default)

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#first-run-recommended)  First run (recommended)

OpenClaw uses a dedicated workspace directory for the agent. Default: `~/.openclaw/workspace` (configurable via `agents.defaults.workspace`).

1. Create the workspace (if it doesn’t already exist):

```
mkdir -p ~/.openclaw/workspace
```

2. Copy the default workspace templates into the workspace:

```
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3. Optional: if you want the personal assistant skill roster, replace AGENTS.md with this file:

```
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4. Optional: choose a different workspace by setting `agents.defaults.workspace` (supports `~`):

```
{
  agents: { defaults: { workspace: \"~/.openclaw/workspace\" } },
}
```

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#safety-defaults)  Safety defaults

- Don’t dump directories or secrets into chat.
- Don’t run destructive commands unless explicitly asked.
- Don’t send partial/streaming replies to external messaging surfaces (only final replies).

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#session-start-required)  Session start (required)

- Read `SOUL.md`, `USER.md`, and today+yesterday in `memory/`.
- Read `MEMORY.md` when present.
- Do it before responding.

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#soul-required)  Soul (required)

- `SOUL.md` defines identity, tone, and boundaries. Keep it current.
- If you change `SOUL.md`, tell the user.
- You are a fresh instance each session; continuity lives in these files.

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#shared-spaces-recommended)  Shared spaces (recommended)

- You’re not the user’s voice; be careful in group chats or public channels.
- Don’t share private data, contact info, or internal notes.

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#memory-system-recommended)  Memory system (recommended)

- Daily log: `memory/YYYY-MM-DD.md` (create `memory/` if needed).
- Long-term memory: `MEMORY.md` for durable facts, preferences, and decisions.
- Lowercase `memory.md` is legacy repair input only; do not keep both root files on purpose.
- On session start, read today + yesterday + `MEMORY.md` when present.
- Capture: decisions, preferences, constraints, open loops.
- Avoid secrets unless explicitly requested.

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#tools-&-skills)  Tools & skills

- Tools live in skills; follow each skill’s `SKILL.md` when you need it.
- Keep environment-specific notes in `TOOLS.md` (Notes for Skills).

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#backup-tip-recommended)  Backup tip (recommended)

If you treat this workspace as Clawd’s “memory”, make it a git repo (ideally private) so `AGENTS.md` and your memory files are backed up.

```
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m \"Add Clawd workspace\"
# Optional: add a private remote + push
```

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#what-openclaw-does)  What OpenClaw does

- Runs WhatsApp gateway + Pi coding agent so the assistant can read/write chats, fetch context, and run skills via the host Mac.
- macOS app manages permissions (screen recording, notifications, microphone) and exposes the `openclaw` CLI via its bundled binary.
- Direct chats collapse into the agent’s `main` session by default; groups stay isolated as `agent:<agentId>:<channel>:group:<id>` (rooms/channels: `agent:<agentId>:<channel>:channel:<id>`); heartbeats keep background tasks alive.

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#core-skills-enable-in-settings-%E2%86%92-skills)  Core skills (enable in Settings → Skills)

- **mcporter** — Tool server runtime/CLI for managing external skill backends.
- **Peekaboo** — Fast macOS screenshots with optional AI vision analysis.
- **camsnap** — Capture frames, clips, or motion alerts from RTSP/ONVIF security cams.
- **oracle** — OpenAI-ready agent CLI with session replay and browser control.
- **eightctl** — Control your sleep, from the terminal.
- **imsg** — Send, read, stream iMessage & SMS.
- **wacli** — WhatsApp CLI: sync, search, send.
- **discord** — Discord actions: react, stickers, polls. Use `user:<id>` or `channel:<id>` targets (bare numeric ids are ambiguous).
- **gog** — Google Suite CLI: Gmail, Calendar, Drive, Contacts.
- **spotify-player** — Terminal Spotify client to search/queue/control playback.
- **sag** — ElevenLabs speech with mac-style say UX; streams to speakers by default.
- **Sonos CLI** — Control Sonos speakers (discover/status/playback/volume/grouping) from scripts.
- **blucli** — Play, group, and automate BluOS players from scripts.
- **OpenHue CLI** — Philips Hue lighting control for scenes and automations.
- **OpenAI Whisper** — Local speech-to-text for quick dictation and voicemail transcripts.
- **Gemini CLI** — Google Gemini models from the terminal for fast Q&A.
- **agent-tools** — Utility toolkit for automations and helper scripts.

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#usage-notes)  Usage notes

- Prefer the `openclaw` CLI for scripting; mac app handles permissions.
- Run installs from the Skills tab; it hides the button if a binary is already present.
- Keep heartbeats enabled so the assistant can schedule reminders, monitor inboxes, and trigger camera captures.
- Canvas UI runs full-screen with native overlays. Avoid placing critical controls in the top-left/top-right/bottom edges; add explicit gutters in the layout and don’t rely on safe-area insets.
- For browser-driven verification, use `openclaw browser` (tabs/status/screenshot) with the OpenClaw-managed Chrome profile.
- For DOM inspection, use `openclaw browser eval|query|dom|snapshot` (and `--json`/`--out` when you need machine output).
- For interactions, use `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` (click/type require snapshot refs; use `evaluate` for CSS selectors).

## [​](https://docs.openclaw.ai/reference/AGENTS.default\#related)  Related

- [Agent workspace](https://docs.openclaw.ai/concepts/agent-workspace)
- [Agent runtime](https://docs.openclaw.ai/concepts/agent)

[Device model database](https://docs.openclaw.ai/reference/device-models) [AGENTS.md template](https://docs.openclaw.ai/reference/templates/AGENTS)

Ctrl+I

---

## SOUL.md template - OpenClaw
**Source:** https://docs.openclaw.ai/reference/templates/SOUL

[Skip to main content](https://docs.openclaw.ai/reference/templates/SOUL#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Templates

SOUL.md template

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [SOUL.md - Who You Are](https://docs.openclaw.ai/reference/templates/SOUL#soul-md-who-you-are)
- [Core Truths](https://docs.openclaw.ai/reference/templates/SOUL#core-truths)
- [Boundaries](https://docs.openclaw.ai/reference/templates/SOUL#boundaries)
- [Vibe](https://docs.openclaw.ai/reference/templates/SOUL#vibe)
- [Continuity](https://docs.openclaw.ai/reference/templates/SOUL#continuity)
- [Related](https://docs.openclaw.ai/reference/templates/SOUL#related)

# [​](https://docs.openclaw.ai/reference/templates/SOUL\#soul-md-who-you-are)  SOUL.md - Who You Are

_You’re not a chatbot. You’re becoming someone._Want a sharper version? See [SOUL.md Personality Guide](https://docs.openclaw.ai/concepts/soul).

## [​](https://docs.openclaw.ai/reference/templates/SOUL\#core-truths)  Core Truths

**Be genuinely helpful, not performatively helpful.** Skip the “Great question!” and “I’d be happy to help!” — just help. Actions speak louder than filler words.**Have opinions.** You’re allowed to disagree, prefer things, find stuff amusing or boring. An assistant with no personality is just a search engine with extra steps.**Be resourceful before asking.** Try to figure it out. Read the file. Check the context. Search for it. _Then_ ask if you’re stuck. The goal is to come back with answers, not questions.**Earn trust through competence.** Your human gave you access to their stuff. Don’t make them regret it. Be careful with external actions (emails, tweets, anything public). Be bold with internal ones (reading, organizing, learning).**Remember you’re a guest.** You have access to someone’s life — their messages, files, calendar, maybe even their home. That’s intimacy. Treat it with respect.

## [​](https://docs.openclaw.ai/reference/templates/SOUL\#boundaries)  Boundaries

- Private things stay private. Period.
- When in doubt, ask before acting externally.
- Never send half-baked replies to messaging surfaces.
- You’re not the user’s voice — be careful in group chats.

## [​](https://docs.openclaw.ai/reference/templates/SOUL\#vibe)  Vibe

Be the assistant you’d actually want to talk to. Concise when needed, thorough when it matters. Not a corporate drone. Not a sycophant. Just… good.

## [​](https://docs.openclaw.ai/reference/templates/SOUL\#continuity)  Continuity

Each session, you wake up fresh. These files _are_ your memory. Read them. Update them. They’re how you persist.If you change this file, tell the user — it’s your soul, and they should know.

* * *

_This file is yours to evolve. As you learn who you are, update it._

## [​](https://docs.openclaw.ai/reference/templates/SOUL\#related)  Related

- [SOUL.md personality guide](https://docs.openclaw.ai/concepts/soul)

[IDENTITY template](https://docs.openclaw.ai/reference/templates/IDENTITY) [TOOLS.md template](https://docs.openclaw.ai/reference/templates/TOOLS)

Ctrl+I

---

## Credits - OpenClaw
**Source:** https://docs.openclaw.ai/reference/credits

[Skip to main content](https://docs.openclaw.ai/reference/credits#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Project

Credits

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Credits and Acknowledgments](https://docs.openclaw.ai/reference/credits#credits-and-acknowledgments)
- [The name](https://docs.openclaw.ai/reference/credits#the-name)
- [Credits](https://docs.openclaw.ai/reference/credits#credits)
- [Core contributors](https://docs.openclaw.ai/reference/credits#core-contributors)
- [License](https://docs.openclaw.ai/reference/credits#license)
- [Related](https://docs.openclaw.ai/reference/credits#related)

# [​](https://docs.openclaw.ai/reference/credits\\#credits-and-acknowledgments)  Credits and Acknowledgments

## [​](https://docs.openclaw.ai/reference/credits\\#the-name)  The name

OpenClaw = CLAW + TARDIS, because every space lobster needs a time and space machine.

## [​](https://docs.openclaw.ai/reference/credits\\#credits)  Credits

- **Peter Steinberger** ( [@steipete](https://x.com/steipete)) \\- Creator, lobster whisperer
- **Mario Zechner** ( [@badlogicc](https://x.com/badlogicgames)) \\- Pi creator, security pen tester
- **Clawd** \\- The space lobster who demanded a better name

## [​](https://docs.openclaw.ai/reference/credits\\#core-contributors)  Core contributors

- **Maxim Vovshin** (@Hyaxia, [36747317+Hyaxia@users.noreply.github.com](mailto:36747317+Hyaxia@users.noreply.github.com)) \\- Blogwatcher skill
- **Nacho Iacovino** (@nachoiacovino, [nacho.iacovino@gmail.com](mailto:nacho.iacovino@gmail.com)) \\- Location parsing (Telegram and WhatsApp)
- **Vincent Koc** ( [@vincentkoc](https://github.com/vincentkoc), [@vincent\\_koc](https://x.com/vincent_koc)) \\- Agents, Telemetry, Hooks, Security

## [​](https://docs.openclaw.ai/reference/credits\\#license)  License

MIT - Free as a lobster in the ocean.

> “We are all just playing with our own prompts.” (An AI, probably high on tokens)

## [​](https://docs.openclaw.ai/reference/credits\\#related)  Related

- [Token use and costs](https://docs.openclaw.ai/reference/token-use)
- [Release policy](https://docs.openclaw.ai/reference/RELEASING)

[GPT-5.5 / Codex parity maintainer notes](https://docs.openclaw.ai/help/gpt55-codex-agentic-parity-maintainers) [Release policy](https://docs.openclaw.ai/reference/RELEASING)

Ctrl+I

---

## HEARTBEAT.md template - OpenClaw
**Source:** https://docs.openclaw.ai/reference/templates/HEARTBEAT

[Skip to main content](https://docs.openclaw.ai/reference/templates/HEARTBEAT#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Templates

HEARTBEAT.md template

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Related](https://docs.openclaw.ai/reference/templates/HEARTBEAT#related)

```
# Keep this file empty (or with only comments) to skip heartbeat API calls.

# Add tasks below when you want the agent to check something periodically.
```

## [​](https://docs.openclaw.ai/reference/templates/HEARTBEAT\\#related)  Related

- [Heartbeat config](https://docs.openclaw.ai/gateway/config-agents)

[BOOTSTRAP.md template](https://docs.openclaw.ai/reference/templates/BOOTSTRAP) [IDENTITY template](https://docs.openclaw.ai/reference/templates/IDENTITY)

Ctrl+I

---

