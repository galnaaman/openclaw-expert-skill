# OpenClaw Providers Documentation

## Volcengine (Doubao) - OpenClaw
**Source:** https://docs.openclaw.ai/providers/volcengine

[Skip to main content](https://docs.openclaw.ai/providers/volcengine#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nProviders\n\nVolcengine (Doubao)\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Getting started](https://docs.openclaw.ai/providers/volcengine#getting-started)\n- [Providers and endpoints](https://docs.openclaw.ai/providers/volcengine#providers-and-endpoints)\n- [Built-in catalog](https://docs.openclaw.ai/providers/volcengine#built-in-catalog)\n- [Text-to-speech](https://docs.openclaw.ai/providers/volcengine#text-to-speech)\n- [Advanced configuration](https://docs.openclaw.ai/providers/volcengine#advanced-configuration)\n- [Related](https://docs.openclaw.ai/providers/volcengine#related)\n\nThe Volcengine provider gives access to Doubao models and third-party models\nhosted on Volcano Engine, with separate endpoints for general and coding\nworkloads. The same bundled plugin can also register Volcengine Speech as a TTS\nprovider.\n\n| Detail | Value |\n| --- | --- |\n| Providers | `volcengine` (general + TTS) + `volcengine-plan` (coding) |\n| Model auth | `VOLCANO_ENGINE_API_KEY` |\n| TTS auth | `VOLCENGINE_TTS_API_KEY` or `BYTEPLUS_SEED_SPEECH_API_KEY` |\n| API | OpenAI-compatible models, BytePlus Seed Speech TTS |\n\n## [​](https://docs.openclaw.ai/providers/volcengine\\#getting-started)  Getting started\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/providers/volcengine#)\n\nSet the API key\n\nRun interactive onboarding:\n\n```\nopenclaw onboard --auth-choice volcengine-api-key\n```\n\nThis registers both the general (`volcengine`) and coding (`volcengine-plan`) providers from a single API key.\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/providers/volcengine#)\n\nSet a default model\n\n```\n{\n  agents: {\n    defaults: {\n      model: { primary: \"volcengine-plan/ark-code-latest\" },\n    },\n  },\n}\n```\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/providers/volcengine#)\n\nVerify the model is available\n\n```\nopenclaw models list --provider volcengine\nopenclaw models list --provider volcengine-plan\n```\n\nFor non-interactive setup (CI, scripting), pass the key directly:\n\n```\nopenclaw onboard --non-interactive \\\n  --mode local \\\n  --auth-choice volcengine-api-key \\\n  --volcengine-api-key \"$VOLCANO_ENGINE_API_KEY\"\n```\n\n## [​](https://docs.openclaw.ai/providers/volcengine\\#providers-and-endpoints)  Providers and endpoints\n\n| Provider | Endpoint | Use case |\n| --- | --- | --- |\n| `volcengine` | `ark.cn-beijing.volces.com/api/v3` | General models |\n| `volcengine-plan` | `ark.cn-beijing.volces.com/api/coding/v3` | Coding models |\n\nBoth providers are configured from a single API key. Setup registers both automatically.\n\n## [​](https://docs.openclaw.ai/providers/volcengine\\#built-in-catalog)  Built-in catalog\n\n- General (volcengine)\n\n- Coding (volcengine-plan)\n\n\n| Model ref | Name | Input | Context |\n| --- | --- | --- | --- |\n| `volcengine/doubao-seed-1-8-251228` | Doubao Seed 1.8 | text, image | 256,000 |\n| `volcengine/doubao-seed-code-preview-251028` | doubao-seed-code-preview-251028 | text, image | 256,000 |\n| `volcengine/kimi-k2-5-260127` | Kimi K2.5 | text, image | 256,000 |\n| `volcengine/glm-4-7-251222` | GLM 4.7 | text, image | 200,000 |\n| `volcengine/deepseek-v3-2-251201` | DeepSeek V3.2 | text, image | 128,000 |\n\n| Model ref | Name | Input | Context |\n| --- | --- | --- | --- |\n| `volcengine-plan/ark-code-latest` | Ark Coding Plan | text | 256,000 |\n| `volcengine-plan/doubao-seed-code` | Doubao Seed Code | text | 256,000 |\n| `volcengine-plan/glm-4.7` | GLM 4.7 Coding | text | 200,000 |\n| `volcengine-plan/kimi-k2-thinking` | Kimi K2 Thinking | text | 256,000 |\n| `volcengine-plan/kimi-k2.5` | Kimi K2.5 Coding | text | 256,000 |\n| `volcengine-plan/doubao-seed-code-preview-251028` | Doubao Seed Code Preview | text | 256,000 |\n\n## [​](https://docs.openclaw.ai/providers/volcengine\\#text-to-speech)  Text-to-speech\n\nVolcengine TTS uses the BytePlus Seed Speech HTTP API and is configured\nseparately from the OpenAI-compatible Doubao model API key. In the BytePlus\nconsole, open Seed Speech > Settings > API Keys and copy the API key, then set:\n\n```\nexport VOLCENGINE_TTS_API_KEY=\"byteplus_seed_speech_api_key\"\nexport VOLCENGINE_TTS_RESOURCE_ID=\"seed-tts-1.0\"\n```\n\nThen enable it in `openclaw.json`:\n\n```\n{\n  messages: {\n    tts: {\n      auto: \"always\",\n      provider: \"volcengine\",\n      providers: {\n        volcengine: {\n          apiKey: \"byteplus_seed_speech_api_key\",\n          voice: \"en_female_anna_mars_bigtts\",\n          speedRatio: 1.0,\n        },\n      },\n    },\n  },\n}\n```\n\nFor voice-note targets, OpenClaw asks Volcengine for provider-native\n`ogg_opus`. For normal audio attachments, it asks for `mp3`. Provider aliases\n`bytedance` and `doubao` also resolve to the same speech provider.The default resource id is `seed-tts-1.0` because that is what BytePlus grants\nto newly created Seed Speech API keys in the default project. If your project\nhas TTS 2.0 entitlement, set `VOLCENGINE_TTS_RESOURCE_ID=seed-tts-2.0`.\n\n`VOLCANO_ENGINE_API_KEY` is for the ModelArk/Doubao model endpoints and is not a\nSeed Speech API key. TTS needs a Seed Speech API key from the BytePlus Speech\nConsole, or a legacy Speech Console AppID/token pair.\n\nLegacy AppID/token auth remains supported for older Speech Console applications:\n\n```\nexport VOLCENGINE_TTS_APPID=\"speech_app_id\"\nexport VOLCENGINE_TTS_TOKEN=\"speech_access_token\"\nexport VOLCENGINE_TTS_CLUSTER=\"volcano_tts\"\n```\n\n## [​](https://docs.openclaw.ai/providers/volcengine\\#advanced-configuration)  Advanced configuration\n\nDefault model after onboarding\n\n`openclaw onboard --auth-choice volcengine-api-key` currently sets\n`volcengine-plan/ark-code-latest` as the default model while also registering\nthe general `volcengine` catalog.\n\nModel picker fallback behavior\n\nDuring onboarding/configure model selection, the Volcengine auth choice prefers\nboth `volcengine/*` and `volcengine-plan/*` rows. If those models are not\nloaded yet, OpenClaw falls back to the unfiltered catalog instead of showing an\nempty provider-scoped picker.\n\nEnvironment variables for daemon processes\n\nIf the Gateway runs as a daemon (launchd/systemd), make sure model and TTS\nenv vars such as `VOLCANO_ENGINE_API_KEY`, `VOLCENGINE_TTS_API_KEY`,\n`BYTEPLUS_SEED_SPEECH_API_KEY`, `VOLCENGINE_TTS_APPID`, and\n`VOLCENGINE_TTS_TOKEN` are available to that process (for example, in\n`~/.openclaw/.env` or via `env.shellEnv`).\n\nWhen running OpenClaw as a background service, environment variables set in your\ninteractive shell are not automatically inherited. See the daemon note above.\n\n## [​](https://docs.openclaw.ai/providers/volcengine\\#related)  Related\n\n[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)\n\n[**Configuration** \\\\\n\\\\\nFull config reference for agents, models, and providers.](https://docs.openclaw.ai/gateway/configuration)\n\n[**Troubleshooting** \\\\\n\\\\\nCommon issues and debugging steps.](https://docs.openclaw.ai/help/troubleshooting)\n\n[**FAQ** \\\\\n\\\\\nFrequently asked questions about OpenClaw setup.](https://docs.openclaw.ai/help/faq)\n\n[vLLM](https://docs.openclaw.ai/providers/vllm) [Vydra](https://docs.openclaw.ai/providers/vydra)\n\nCtrl+I

---

## OpenAI
**Source:** https://docs.openclaw.ai/providers/openai

[Skip to main content](https://docs.openclaw.ai/providers/openai#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

OpenAI

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick choice](https://docs.openclaw.ai/providers/openai#quick-choice)
- [Naming map](https://docs.openclaw.ai/providers/openai#naming-map)
- [OpenClaw feature coverage](https://docs.openclaw.ai/providers/openai#openclaw-feature-coverage)
- [Memory embeddings](https://docs.openclaw.ai/providers/openai#memory-embeddings)
- [Getting started](https://docs.openclaw.ai/providers/openai#getting-started)
- [Route summary](https://docs.openclaw.ai/providers/openai#route-summary)
- [Config example](https://docs.openclaw.ai/providers/openai#config-example)
- [Native Codex app-server auth](https://docs.openclaw.ai/providers/openai#native-codex-app-server-auth)
- [Image generation](https://docs.openclaw.ai/providers/openai#image-generation)
- [Video generation](https://docs.openclaw.ai/providers/openai#video-generation)
- [GPT-5 prompt contribution](https://docs.openclaw.ai/providers/openai#gpt-5-prompt-contribution)
- [Voice and speech](https://docs.openclaw.ai/providers/openai#voice-and-speech)
- [Azure OpenAI endpoints](https://docs.openclaw.ai/providers/openai#azure-openai-endpoints)
- [Configuration](https://docs.openclaw.ai/providers/openai#configuration)
- [API version](https://docs.openclaw.ai/providers/openai#api-version)
- [Model names are deployment names](https://docs.openclaw.ai/providers/openai#model-names-are-deployment-names)
- [Regional availability](https://docs.openclaw.ai/providers/openai#regional-availability)
- [Parameter differences](https://docs.openclaw.ai/providers/openai#parameter-differences)
- [Advanced configuration](https://docs.openclaw.ai/providers/openai#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/openai#related)

OpenAI provides developer APIs for GPT models, and Codex is also available as a
ChatGPT-plan coding agent through OpenAI’s Codex clients. OpenClaw keeps those
surfaces separate so config stays predictable.OpenClaw supports three OpenAI-family routes. The model prefix selects the
provider/auth route; a separate runtime setting selects who executes the
embedded agent loop:

- **API key** — direct OpenAI Platform access with usage-based billing (`openai/*` models)
- **Codex subscription through PI** — ChatGPT/Codex sign-in with subscription access (`openai-codex/*` models)
- **Codex app-server harness** — native Codex app-server execution (`openai/*` models plus `agents.defaults.agentRuntime.id: \"codex\"`)

OpenAI explicitly supports subscription OAuth usage in external tools and workflows like OpenClaw.Provider, model, runtime, and channel are separate layers. If those labels are
getting mixed together, read [Agent runtimes](https://docs.openclaw.ai/concepts/agent-runtimes) before
changing config.

## [​](https://docs.openclaw.ai/providers/openai\\#quick-choice)  Quick choice

| Goal | Use | Notes |
| --- | --- | --- |
| Direct API-key billing | `openai/gpt-5.5` | Set `OPENAI_API_KEY` or run OpenAI API-key onboarding. |
| GPT-5.5 with ChatGPT/Codex subscription auth | `openai-codex/gpt-5.5` | Default PI route for Codex OAuth. Best first choice for subscription setups. |
| GPT-5.5 with native Codex app-server behavior | `openai/gpt-5.5` plus `agentRuntime.id: \"codex\"` | Forces the Codex app-server harness for that model ref. |
| Image generation or editing | `openai/gpt-image-2` | Works with either `OPENAI_API_KEY` or OpenAI Codex OAuth. |
| Transparent-background images | `openai/gpt-image-1.5` | Use `outputFormat=png` or `webp` and `openai.background=transparent`. |

## [​](https://docs.openclaw.ai/providers/openai\\#naming-map)  Naming map

The names are similar but not interchangeable:

| Name you see | Layer | Meaning |
| --- | --- | --- |
| `openai` | Provider prefix | Direct OpenAI Platform API route. |
| `openai-codex` | Provider prefix | OpenAI Codex OAuth/subscription route through the normal OpenClaw PI runner. |
| `codex` plugin | Plugin | Bundled OpenClaw plugin that provides native Codex app-server runtime and `/codex` chat controls. |
| `agentRuntime.id: codex` | Agent runtime | Force the native Codex app-server harness for embedded turns. |
| `/codex ...` | Chat command set | Bind/control Codex app-server threads from a conversation. |
| `runtime: \"acp\", agentId: \"codex\"` | ACP session route | Explicit fallback path that runs Codex through ACP/acpx. |

This means a config can intentionally contain both `openai-codex/*` and the
`codex` plugin. That is valid when you want Codex OAuth through PI and also want
native `/codex` chat controls available. `openclaw doctor` warns about that
combination so you can confirm it is intentional; it does not rewrite it.

GPT-5.5 is available through both direct OpenAI Platform API-key access and
subscription/OAuth routes. Use `openai/gpt-5.5` for direct `OPENAI_API_KEY`
traffic, `openai-codex/gpt-5.5` for Codex OAuth through PI, or
`openai/gpt-5.5` with `agentRuntime.id: \"codex\"` for the native Codex
app-server harness.

Enabling the OpenAI plugin, or selecting an `openai-codex/*` model, does not
enable the bundled Codex app-server plugin. OpenClaw enables that plugin only
when you explicitly select the native Codex harness with
`agentRuntime.id: \"codex\"` or use a legacy `codex/*` model ref.
If the bundled `codex` plugin is enabled but `openai-codex/*` still resolves
through PI, `openclaw doctor` warns and leaves the route unchanged.

## [​](https://docs.openclaw.ai/providers/openai\\#openclaw-feature-coverage)  OpenClaw feature coverage

| OpenAI capability | OpenClaw surface | Status |
| --- | --- | --- |
| Chat / Responses | `openai/<model>` model provider | Yes |
| Codex subscription models | `openai-codex/<model>` with `openai-codex` OAuth | Yes |
| Codex app-server harness | `openai/<model>` with `agentRuntime.id: codex` | Yes |
| Server-side web search | Native OpenAI Responses tool | Yes, when web search is enabled and no provider pinned |
| Images | `image_generate` | Yes |
| Videos | `video_generate` | Yes |
| Text-to-speech | `messages.tts.provider: \"openai\"` / `tts` | Yes |
| Batch speech-to-text | `tools.media.audio` / media understanding | Yes |
| Streaming speech-to-text | Voice Call `streaming.provider: \"openai\"` | Yes |
| Realtime voice | Voice Call `realtime.provider: \"openai\"` / Control UI Talk | Yes |
| Embeddings | memory embedding provider | Yes |

## [​](https://docs.openclaw.ai/providers/openai\\#memory-embeddings)  Memory embeddings

OpenClaw can use OpenAI, or an OpenAI-compatible embedding endpoint, for
`memory_search` indexing and query embeddings:

```
{
  agents: {
    defaults: {
      memorySearch: {
        provider: \"openai\",
        model: \"text-embedding-3-small\",
      },
    },
  },
}
```

For OpenAI-compatible endpoints that require asymmetric embedding labels, set
`queryInputType` and `documentInputType` under `memorySearch`. OpenClaw forwards
those as provider-specific `input_type` request fields: query embeddings use
`queryInputType`; indexed memory chunks and batch indexing use
`documentInputType`. See the [Memory configuration reference](https://docs.openclaw.ai/reference/memory-config#provider-specific-config) for the full example.

## [​](https://docs.openclaw.ai/providers/openai\\#getting-started)  Getting started

Choose your preferred auth method and follow the setup steps.

- API key (OpenAI Platform)

- Codex subscription


**Best for:** direct API access and usage-based billing.

1

[Navigate to header](https://docs.openclaw.ai/providers/openai#)

Get your API key

Create or copy an API key from the [OpenAI Platform dashboard](https://platform.openai.com/api-keys).

2

[Navigate to header](https://docs.openclaw.ai/providers/openai#)

Run onboarding

```
openclaw onboard --auth-choice openai-api-key
```

Or pass the key directly:

```
openclaw onboard --openai-api-key \"$OPENAI_API_KEY\"
```

3

[Navigate to header](https://docs.openclaw.ai/providers/openai#)

Verify the model is available

```
openclaw models list --provider openai
```

### [​](https://docs.openclaw.ai/providers/openai\\#route-summary)  Route summary

| Model ref | Runtime config | Route | Auth |
| --- | --- | --- | --- |
| `openai/gpt-5.5` | omitted / `agentRuntime.id: \"pi\"` | Direct OpenAI Platform API | `OPENAI_API_KEY` |
| `openai/gpt-5.4-mini` | omitted / `agentRuntime.id: \"pi\"` | Direct OpenAI Platform API | `OPENAI_API_KEY` |
| `openai/gpt-5.5` | `agentRuntime.id: \"codex\"` | Codex app-server harness | Codex app-server |

`openai/*` is the direct OpenAI API-key route unless you explicitly force
the Codex app-server harness. Use `openai-codex/*` for Codex OAuth through
the default PI runner, or use `openai/gpt-5.5` with
`agentRuntime.id: \"codex\"` for native Codex app-server execution.

### [​](https://docs.openclaw.ai/providers/openai\\#config-example)  Config example

```
{
  env: { OPENAI_API_KEY: \"sk-...\" },
  agents: { defaults: { model: { primary: \"openai/gpt-5.5\" } } },
}
```

OpenClaw does **not** expose `openai/gpt-5.3-codex-spark`. Live OpenAI API requests reject that model, and the current Codex catalog does not expose it either.

**Best for:** using your ChatGPT/Codex subscription instead of a separate API key. Codex cloud requires ChatGPT sign-in.

1

[Navigate to header](https://docs.openclaw.ai/providers/openai#)

Run Codex OAuth

```
openclaw onboard --auth-choice openai-codex
```

Or run OAuth directly:

```
openclaw models auth login --provider openai-codex
```

For headless or callback-hostile setups, add `--device-code` to sign in with a ChatGPT device-code flow instead of the localhost browser callback:

```
openclaw models auth login --provider openai-codex --device-code
```

2

[Navigate to header](https://docs.openclaw.ai/providers/openai#)

Set the default model

```
openclaw config set agents.defaults.model.primary openai-codex/gpt-5.5
```

3

[Navigate to header](https://docs.openclaw.ai/providers/openai#)

Verify the model is available

```
openclaw models list --provider openai-codex
```

### [​](https://docs.openclaw.ai/providers/openai\\#route-summary-2)  Route summary

| Model ref | Runtime config | Route | Auth |
| --- | --- | --- | --- |
| `openai-codex/gpt-5.5` | omitted / `runtime: \"pi\"` | ChatGPT/Codex OAuth through PI | Codex sign-in |
| `openai-codex/gpt-5.5` | `runtime: \"auto\"` | Still PI unless a plugin explicitly claims `openai-codex` | Codex sign-in |
| `openai/gpt-5.5` | `agentRuntime.id: \"codex\"` | Codex app-server harness | Codex app-server auth |

Keep using the `openai-codex` provider id for auth/profile commands. The
`openai-codex/*` model prefix is also the explicit PI route for Codex OAuth.
It does not select or auto-enable the bundled Codex app-server harness.

`openai-codex/gpt-5.4-mini` is not a supported Codex OAuth route. Use
`openai/gpt-5.4-mini` with an OpenAI API key, or use
`openai-codex/gpt-5.5` with Codex OAuth.

### [​](https://docs.openclaw.ai/providers/openai\\#config-example-2)  Config example

```
{
  agents: { defaults: { model: { primary: \"openai-codex/gpt-5.5\" } } },
}
```

Onboarding no longer imports OAuth material from `~/.codex`. Sign in with browser OAuth (default) or the device-code flow above — OpenClaw manages the resulting credentials in its own agent auth store.

### [​](https://docs.openclaw.ai/providers/openai\\#status-indicator)  Status indicator

Chat `/status` shows which model runtime is active for the current session.
The default PI harness appears as `Runtime: OpenClaw Pi Default`. When the
bundled Codex app-server harness is selected, `/status` shows
`Runtime: OpenAI Codex`. Existing sessions keep their recorded harness id, so use
`/new` or `/reset` after changing `agentRuntime` if you want `/status` to
reflect a new PI/Codex choice.

### [​](https://docs.openclaw.ai/providers/openai\\#doctor-warning)  Doctor warning

If the bundled `codex` plugin is enabled while this tab’s
`openai-codex/*` route is selected, `openclaw doctor` warns that the model
still resolves through PI. Keep the config unchanged when that is the
intended subscription-auth route. Switch to `openai/<model>` plus
`agentRuntime.id: \"codex\"` only when you want native Codex
app-server execution.

### [​](https://openclaw.ai/providers/openai\\#context-window-cap)  Context window cap

OpenClaw treats model metadata and the runtime context cap as separate values.For `openai-codex/gpt-5.5` through Codex OAuth:

- Native `contextWindow`: `1000000`
- Default runtime `contextTokens` cap: `272000`

The smaller default cap has better latency and quality characteristics in practice. Override it with `contextTokens`:

```
{
  models: {
    providers: {
      \"openai-codex\": {
        models: [{ id: \"gpt-5.5\", contextTokens: 160000 }],
      },
    },
  },
}
```

Use `contextWindow` to declare native model metadata. Use `contextTokens` to limit the runtime context budget.

### [​](https://docs.openclaw.ai/providers/openai\\#catalog-recovery)  Catalog recovery

OpenClaw uses upstream Codex catalog metadata for `gpt-5.5` when it is
present. If live Codex discovery omits the `openai-codex/gpt-5.5` row while
the account is authenticated, OpenClaw synthesizes that OAuth model row so
cron, sub-agent, and configured default-model runs do not fail with
`Unknown model`.

## [​](https://docs.openclaw.ai/providers/openai\\#native-codex-app-server-auth)  Native Codex app-server auth

The native Codex app-server harness uses `openai/*` model refs plus
`agentRuntime.id: \"codex\"`, but its auth is still account-based. OpenClaw
selects auth in this order:

1. An explicit OpenClaw `openai-codex` auth profile bound to the agent.
2. The app-server’s existing account, such as a local Codex CLI ChatGPT sign-in.
3. For local stdio app-server launches only, `CODEX_API_KEY`, then
`OPENAI_API_KEY`, when the app-server reports no account and still requires
OpenAI auth.

That means a local ChatGPT/Codex subscription sign-in is not replaced just
because the gateway process also has `OPENAI_API_KEY` for direct OpenAI models
or embeddings. Env API-key fallback is only the local stdio no-account path; it
is not sent to WebSocket app-server connections. When a subscription-style Codex
profile is selected, OpenClaw also keeps `CODEX_API_KEY` and `OPENAI_API_KEY`
out of the spawned stdio app-server child and sends the selected credentials
through the app-server login RPC.

## [​](https://docs.openclaw.ai/providers/openai\\#image-generation)  Image generation

The bundled `openai` plugin registers image generation through the `image_generate` tool.
It supports both OpenAI API-key image generation and Codex OAuth image
generation through the same `openai/gpt-image-2` model ref.

| Capability | OpenAI API key | Codex OAuth |
| --- | --- | --- |
| Model ref | `openai/gpt-image-2` | `openai/gpt-image-2` |
| Auth | `OPENAI_API_KEY` | OpenAI Codex OAuth sign-in |
| Transport | OpenAI Images API | Codex Responses backend |
| Max images per request | 4 | 4 |
| Edit mode | Enabled (up to 5 reference images) | Enabled (up to 5 reference images) |
| Size overrides | Supported, including 2K/4K sizes | Supported, including 2K/4K sizes |
| Aspect ratio / resolution | Not forwarded to OpenAI Images API | Mapped to a supported size when safe |

```
{
  agents: {
    defaults: {
      imageGenerationModel: { primary: \"openai/gpt-image-2\" },
    },
  },
}
```

See [Image Generation](https://docs.openclaw.ai/tools/image-generation) for shared tool parameters, provider selection, and failover behavior.

`gpt-image-2` is the default for both OpenAI text-to-image generation and image
editing. `gpt-image-1.5`, `gpt-image-1`, and `gpt-image-1-mini` remain usable as
explicit model overrides. Use `openai/gpt-image-1.5` for transparent-background
PNG/WebP output; the current `gpt-image-2` API rejects
`background: \"transparent\"`.For a transparent-background request, agents should call `image_generate` with
`model: \"openai/gpt-image-1.5\"`, `outputFormat: \"png\"` or `\"webp\"`, and
`background: \"transparent\"`; the older `openai.background` provider option is
still accepted. OpenClaw also protects the public OpenAI and
OpenAI Codex OAuth routes by rewriting default `openai/gpt-image-2` transparent
requests to `gpt-image-1.5`; Azure and custom OpenAI-compatible endpoints keep
their configured deployment/model names.The same setting is exposed for headless CLI runs:

```
openclaw infer image generate \\\
  --model openai/gpt-image-1.5 \\\
  --output-format png \\\
  --background transparent \\\
  --prompt \"A simple red circle sticker on a transparent background\" \\\
  --json
```

Use the same `--output-format` and `--background` flags with
`openclaw infer image edit` when starting from an input file.
`--openai-background` remains available as an OpenAI-specific alias.For Codex OAuth installs, keep the same `openai/gpt-image-2` ref. When an
`openai-codex` OAuth profile is configured, OpenClaw resolves that stored OAuth
access token and sends image requests through the Codex Responses backend. It
does not first try `OPENAI_API_KEY` or silently fall back to an API key for that
request. Configure `models.providers.openai` explicitly with an API key,
custom base URL, or Azure endpoint when you want the direct OpenAI Images API
route instead.
If that custom image endpoint is on a trusted LAN/private address, also set
`browser.ssrfPolicy.dangerouslyAllowPrivateNetwork: true`; OpenClaw keeps
private/internal OpenAI-compatible image endpoints blocked unless this opt-in is
present.Generate:

```
/tool image_generate model=openai/gpt-image-2 prompt=\"A polished launch poster for OpenClaw on macOS\" size=3840x2160 count=1
```

Generate a transparent PNG:

```
/tool image_generate model=openai/gpt-image-1.5 prompt=\"A simple red circle sticker on a transparent background\" outputFormat=png background=transparent
```

Edit:

```
/tool image_generate model=openai/gpt-image-2 prompt=\"Preserve the object shape, change the material to translucent glass\" image=/path/to/reference.png size=1024x1536
```

## [​](https://docs.openclaw.ai/providers/openai\\#video-generation)  Video generation

The bundled `openai` plugin registers video generation through the `video_generat...(content truncated)

---

## Deepgram - OpenClaw
**Source:** https://docs.openclaw.ai/providers/deepgram

[Skip to main content](https://docs.openclaw.ai/providers/deepgram#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Deepgram

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/deepgram#getting-started)
- [Configuration options](https://docs.openclaw.ai/providers/deepgram#configuration-options)
- [Voice Call streaming STT](https://docs.openclaw.ai/providers/deepgram#voice-call-streaming-stt)
- [Notes](https://docs.openclaw.ai/providers/deepgram#notes)
- [Related](https://docs.openclaw.ai/providers/deepgram#related)

Deepgram is a speech-to-text API. In OpenClaw it is used for inbound
audio/voice-note transcription through `tools.media.audio` and for Voice Call
streaming STT through `plugins.entries.voice-call.config.streaming`.For batch transcription, OpenClaw uploads the complete audio file to Deepgram
and injects the transcript into the reply pipeline (`{{Transcript}}` +
`[Audio]` block). For Voice Call streaming, OpenClaw forwards live G.711
u-law frames over Deepgram’s WebSocket `listen` endpoint and emits partial or
final transcripts as Deepgram returns them.

| Detail | Value |
| --- | --- |
| Website | [deepgram.com](https://deepgram.com/) |
| Docs | [developers.deepgram.com](https://developers.deepgram.com/) |
| Auth | `DEEPGRAM_API_KEY` |
| Default model | `nova-3` |

## [​](https://docs.openclaw.ai/providers/deepgram\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/deepgram#)

Set your API key

Add your Deepgram API key to the environment:

```
DEEPGRAM_API_KEY=dg_...
```

2

[Navigate to header](https://docs.openclaw.ai/providers/deepgram#)

Enable the audio provider

```
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: \"deepgram\", model: \"nova-3\" }],
      },
    },
  },
}
```

3

[Navigate to header](https://docs.openclaw.ai/providers/deepgram#)

Send a voice note

Send an audio message through any connected channel. OpenClaw transcribes it
via Deepgram and injects the transcript into the reply pipeline.

## [​](https://docs.openclaw.ai/providers/deepgram\\#configuration-options)  Configuration options

| Option | Path | Description |
| --- | --- | --- |
| `model` | `tools.media.audio.models[].model` | Deepgram model id (default: `nova-3`) |
| `language` | `tools.media.audio.models[].language` | Language hint (optional) |
| `detect_language` | `tools.media.audio.providerOptions.deepgram.detect_language` | Enable language detection (optional) |
| `punctuate` | `tools.media.audio.providerOptions.deepgram.punctuate` | Enable punctuation (optional) |
| `smart_format` | `tools.media.audio.providerOptions.deepgram.smart_format` | Enable smart formatting (optional) |

- With language hint

- With Deepgram options


```
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: \"deepgram\", model: \"nova-3\", language: \"en\" }],
      },
    },
  },
}
```

```
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true,
          },
        },
        models: [{ provider: \"deepgram\", model: \"nova-3\" }],
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/providers/deepgram\\#voice-call-streaming-stt)  Voice Call streaming STT

The bundled `deepgram` plugin also registers a realtime transcription provider
for the Voice Call plugin.

| Setting | Config path | Default |
| --- | --- | --- |
| API key | `plugins.entries.voice-call.config.streaming.providers.deepgram.apiKey` | Falls back to `DEEPGRAM_API_KEY` |
| Model | `...deepgram.model` | `nova-3` |
| Language | `...deepgram.language` | (unset) |
| Encoding | `...deepgram.encoding` | `mulaw` |
| Sample rate | `...deepgram.sampleRate` | `8000` |
| Endpointing | `...deepgram.endpointingMs` | `800` |
| Interim results | `...deepgram.interimResults` | `true` |

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        config: {
          streaming: {
            enabled: true,
            provider: \"deepgram\",
            providers: {
              deepgram: {
                apiKey: \"${DEEPGRAM_API_KEY}\",
                model: \"nova-3\",
                endpointingMs: 800,
                language: \"en-US\",
              },
            },\n          },\n        },\n      },\n    },\n  },\n}
```

Voice Call receives telephony audio as 8 kHz G.711 u-law. The Deepgram
streaming provider defaults to `encoding: \"mulaw\"` and `sampleRate: 8000`, so
Twilio media frames can be forwarded directly.

## [​](https://docs.openclaw.ai/providers/deepgram\\#notes)  Notes

Authentication

Authentication follows the standard provider auth order. `DEEPGRAM_API_KEY` is
the simplest path.

Proxy and custom endpoints

Override endpoints or headers with `tools.media.audio.baseUrl` and
`tools.media.audio.headers` when using a proxy.

Output behavior

Output follows the same audio rules as other providers (size caps, timeouts,
transcript injection).

## [​](https://docs.openclaw.ai/providers/deepgram\\#related)  Related

[**Media tools** \\\\\n\\\\\nAudio, image, and video processing pipeline overview.](https://docs.openclaw.ai/tools/media-overview)

[**Configuration** \\\\\n\\\\\nFull config reference including media tool settings.](https://docs.openclaw.ai/gateway/configuration)

[**Troubleshooting** \\\\\n\\\\\nCommon issues and debugging steps.](https://docs.openclaw.ai/help/troubleshooting)

[**FAQ** \\\\\n\\\\\nFrequently asked questions about OpenClaw setup.](https://docs.openclaw.ai/help/faq)

[ComfyUI](https://docs.openclaw.ai/providers/comfy) [DeepSeek](https://docs.openclaw.ai/providers/deepseek)

Ctrl+I

---

## Chutes - OpenClaw
**Source:** https://docs.openclaw.ai/providers/chutes

[Skip to main content](https://docs.openclaw.ai/providers/chutes#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Chutes

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/chutes#getting-started)
- [Discovery behavior](https://docs.openclaw.ai/providers/chutes#discovery-behavior)
- [Default aliases](https://docs.openclaw.ai/providers/chutes#default-aliases)
- [Built-in starter catalog](https://docs.openclaw.ai/providers/chutes#built-in-starter-catalog)
- [Config example](https://docs.openclaw.ai/providers/chutes#config-example)
- [Related](https://docs.openclaw.ai/providers/chutes#related)

[Chutes](https://chutes.ai/) exposes open-source model catalogs through an
OpenAI-compatible API. OpenClaw supports both browser OAuth and direct API-key
auth for the bundled `chutes` provider.

| Property | Value |
| --- | --- |
| Provider | `chutes` |
| API | OpenAI-compatible |
| Base URL | `https://llm.chutes.ai/v1` |
| Auth | OAuth or API key (see below) |

## [​](https://docs.openclaw.ai/providers/chutes\\#getting-started)  Getting started

- OAuth

- API key


1

[Navigate to header](https://docs.openclaw.ai/providers/chutes#)

Run the OAuth onboarding flow

```
openclaw onboard --auth-choice chutes
```

OpenClaw launches the browser flow locally, or shows a URL + redirect-paste
flow on remote/headless hosts. OAuth tokens auto-refresh through OpenClaw auth
profiles.

2

[Navigate to header](https://docs.openclaw.ai/providers/chutes#)

Verify the default model

After onboarding, the default model is set to
`chutes/zai-org/GLM-4.7-TEE` and the bundled Chutes catalog is
registered.

1

[Navigate to header](https://docs.openclaw.ai/providers/chutes#)

Get an API key

Create a key at
[chutes.ai/settings/api-keys](https://chutes.ai/settings/api-keys).

2

[Navigate to header](https://docs.openclaw.ai/providers/chutes#)

Run the API key onboarding flow

```
openclaw onboard --auth-choice chutes-api-key
```

3

[Navigate to header](https://docs.openclaw.ai/providers/chutes#)

Verify the default model

After onboarding, the default model is set to
`chutes/zai-org/GLM-4.7-TEE` and the bundled Chutes catalog is
registered.

Both auth paths register the bundled Chutes catalog and set the default model to
`chutes/zai-org/GLM-4.7-TEE`. Runtime environment variables: `CHUTES_API_KEY`,
`CHUTES_OAUTH_TOKEN`.

## [​](https://docs.openclaw.ai/providers/chutes\\#discovery-behavior)  Discovery behavior

When Chutes auth is available, OpenClaw queries the Chutes catalog with that
credential and uses the discovered models. If discovery fails, OpenClaw falls
back to a bundled static catalog so onboarding and startup still work.

## [​](https://docs.openclaw.ai/providers/chutes\\#default-aliases)  Default aliases

OpenClaw registers three convenience aliases for the bundled Chutes catalog:

| Alias | Target model |
| --- | --- |
| `chutes-fast` | `chutes/zai-org/GLM-4.7-FP8` |
| `chutes-pro` | `chutes/deepseek-ai/DeepSeek-V3.2-TEE` |
| `chutes-vision` | `chutes/chutesai/Mistral-Small-3.2-24B-Instruct-2506` |

## [​](https://docs.openclaw.ai/providers/chutes\\#built-in-starter-catalog)  Built-in starter catalog

The bundled fallback catalog includes current Chutes refs:

| Model ref |
| --- |
| `chutes/zai-org/GLM-4.7-TEE` |
| `chutes/zai-org/GLM-5-TEE` |
| `chutes/deepseek-ai/DeepSeek-V3.2-TEE` |
| `chutes/deepseek-ai/DeepSeek-R1-0528-TEE` |
| `chutes/moonshotai/Kimi-K2.5-TEE` |
| `chutes/chutesai/Mistral-Small-3.2-24B-Instruct-2506` |
| `chutes/Qwen/Qwen3-Coder-Next-TEE` |
| `chutes/openai/gpt-oss-120b-TEE` |

## [​](https://docs.openclaw.ai/providers/chutes\\#config-example)  Config example

```
{
  agents: {
    defaults: {
      model: { primary: \"chutes/zai-org/GLM-4.7-TEE\" },
      models: {
        \"chutes/zai-org/GLM-4.7-TEE\": { alias: \"Chutes GLM 4.7\" },
        \"chutes/deepseek-ai/DeepSeek-V3.2-TEE\": { alias: \"Chutes DeepSeek V3.2\" },
      },
    },
  },
}
```

OAuth overrides

You can customize the OAuth flow with optional environment variables:

| Variable | Purpose |
| --- | --- |
| `CHUTES_CLIENT_ID` | Custom OAuth client ID |
| `CHUTES_CLIENT_SECRET` | Custom OAuth client secret |
| `CHUTES_OAUTH_REDIRECT_URI` | Custom redirect URI |
| `CHUTES_OAUTH_SCOPES` | Custom OAuth scopes |

See the [Chutes OAuth docs](https://chutes.ai/docs/sign-in-with-chutes/overview)
for redirect-app requirements and help.

Notes

- API-key and OAuth discovery both use the same `chutes` provider id.
- Chutes models are registered as `chutes/<model-id>`.
- If discovery fails at startup, the bundled static catalog is used automatically.

## [​](https://docs.openclaw.ai/providers/chutes\\#related)  Related

[**Model selection** \\\\\n\\\\\nProvider rules, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Configuration reference** \\\\\n\\\\\nFull config schema including provider settings.](https://docs.openclaw.ai/gateway/configuration-reference)

[**Chutes** \\\\\n\\\\\nChutes dashboard and API docs.](https://chutes.ai/)

[**Chutes API keys** \\\\\n\\\\\nCreate and manage Chutes API keys.](https://chutes.openclaw.ai/settings/api-keys)

[Azure Speech](https://docs.openclaw.ai/providers/azure-speech) [Claude Max API proxy](https://docs.openclaw.ai/providers/claude-max-api-proxy)

Ctrl+I

---

## Fireworks - OpenClaw
**Source:** https://docs.openclaw.ai/providers/fireworks

[Skip to main content](https://docs.openclaw.ai/providers/fireworks#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Fireworks

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/fireworks#getting-started)
- [Non-interactive example](https://docs.openclaw.ai/providers/fireworks#non-interactive-example)
- [Built-in catalog](https://docs.openclaw.ai/providers/fireworks#built-in-catalog)
- [Custom Fireworks model ids](https://docs.openclaw.ai/providers/fireworks#custom-fireworks-model-ids)
- [Related](https://docs.openclaw.ai/providers/fireworks#related)

[Fireworks](https://fireworks.ai/) exposes open-weight and routed models through an OpenAI-compatible API. OpenClaw includes a bundled Fireworks provider plugin.

| Property | Value |
| --- | --- |
| Provider | `fireworks` |
| Auth | `FIREWORKS_API_KEY` |
| API | OpenAI-compatible chat/completions |
| Base URL | `https://api.fireworks.ai/inference/v1` |
| Default model | `fireworks/accounts/fireworks/routers/kimi-k2p5-turbo` |

## [​](https://docs.openclaw.ai/providers/fireworks\\#getting-started) Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/fireworks#)

Set up Fireworks auth through onboarding

```
openclaw onboard --auth-choice fireworks-api-key
```

This stores your Fireworks key in OpenClaw config and sets the Fire Pass starter model as the default.

2

[Navigate to header](https://docs.openclaw.ai/providers/fireworks#)

Verify the model is available

```
openclaw models list --provider fireworks
```

## [​](https://docs.openclaw.ai/providers/fireworks\\#non-interactive-example) Non-interactive example

For scripted or CI setups, pass all values on the command line:

```
openclaw onboard --non-interactive \\
  --mode local \\
  --auth-choice fireworks-api-key \\
  --fireworks-api-key \"$FIREWORKS_API_KEY\" \\
  --skip-health \\
  --accept-risk
```

## [​](https://docs.openclaw.ai/providers/fireworks\\#built-in-catalog) Built-in catalog

| Model ref | Name | Input | Context | Max output | Notes |
| --- | --- | --- | --- | --- | --- |
| `fireworks/accounts/fireworks/models/kimi-k2p6` | Kimi K2.6 | text,image | 262,144 | 262,144 | Latest Kimi model on Fireworks. Thinking is disabled for Fireworks K2.6 requests; route through Moonshot directly if you need Kimi thinking output. |
| `fireworks/accounts/fireworks/routers/kimi-k2p5-turbo` | Kimi K2.5 Turbo (Fire Pass) | text,image | 256,000 | 256,000 | Default bundled starter model on Fireworks |

If Fireworks publishes a newer model such as a fresh Qwen or Gemma release, you can switch to it directly by using its Fireworks model id without waiting for a bundled catalog update.

## [​](https://docs.openclaw.ai/providers/fireworks\\#custom-fireworks-model-ids) Custom Fireworks model ids

OpenClaw accepts dynamic Fireworks model ids too. Use the exact model or router id shown by Fireworks and prefix it with `fireworks/`.

```
{
  agents: {
    defaults: {
      model: {
        primary: \"fireworks/accounts/fireworks/routers/kimi-k2p5-turbo\",
      },
    },
  },
}
```

How model id prefixing works

Every Fireworks model ref in OpenClaw starts with `fireworks/` followed by the exact id or router path from the Fireworks platform. For example:

- Router model: `fireworks/accounts/fireworks/routers/kimi-k2p5-turbo`
- Direct model: `fireworks/accounts/fireworks/models/<model-name>`

OpenClaw strips the `fireworks/` prefix when building the API request and sends the remaining path to the Fireworks endpoint.

Environment note

If the Gateway runs outside your interactive shell, make sure `FIREWORKS_API_KEY` is available to that process too.

A key sitting only in `~/.profile` will not help a launchd/systemd daemon unless that environment is imported there as well. Set the key in `~/.openclaw/.env` or via `env.shellEnv` to ensure the gateway process can read it.

## [​](https://docs.openclaw.ai/providers/fireworks\\#related) Related

[**Model selection** \\\n\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Troubleshooting** \\\n\\nGeneral troubleshooting and FAQ.](https://docs.openclaw.ai/help/troubleshooting)

[Fal](https://docs.openclaw.ai/providers/fal) [GitHub Copilot](https://docs.openclaw.ai/providers/github-copilot)

Ctrl+I

---

## Qwen - OpenClaw
**Source:** https://docs.openclaw.ai/providers/qwen

[Skip to main content](https://docs.openclaw.ai/providers/qwen#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Qwen

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/qwen#getting-started)
- [Plan types and endpoints](https://docs.openclaw.ai/providers/qwen#plan-types-and-endpoints)
- [Built-in catalog](https://docs.openclaw.ai/providers/qwen#built-in-catalog)
- [Thinking Controls](https://docs.openclaw.ai/providers/qwen#thinking-controls)
- [Multimodal add-ons](https://docs.openclaw.ai/providers/qwen#multimodal-add-ons)
- [Advanced configuration](https://docs.openclaw.ai/providers/qwen#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/qwen#related)

**Qwen OAuth has been removed.** The free-tier OAuth integration
(`qwen-portal`) that used `portal.qwen.ai` endpoints is no longer available.
See [Issue #49557](https://github.com/openclaw/openclaw/issues/49557) for
background.

OpenClaw now treats Qwen as a first-class bundled provider with canonical id
`qwen`. The bundled provider targets the Qwen Cloud / Alibaba DashScope and
Coding Plan endpoints and keeps legacy `modelstudio` ids working as a
compatibility alias.

- Provider: `qwen`
- Preferred env var: `QWEN_API_KEY`
- Also accepted for compatibility: `MODELSTUDIO_API_KEY`, `DASHSCOPE_API_KEY`
- API style: OpenAI-compatible

If you want `qwen3.6-plus`, prefer the **Standard (pay-as-you-go)** endpoint.
Coding Plan support can lag behind the public catalog.

## [​](https://docs.openclaw.ai/providers/qwen\\#getting-started)  Getting started

Choose your plan type and follow the setup steps.

- Coding Plan (subscription)

- Standard (pay-as-you-go)


**Best for:** subscription-based access through the Qwen Coding Plan.

1

[Navigate to header](https://docs.openclaw.ai/providers/qwen#)

Get your API key

Create or copy an API key from [home.qwencloud.com/api-keys](https://home.qwencloud.com/api-keys).

2

[Navigate to header](https://docs.openclaw.ai/providers/qwen#)

Run onboarding

For the **Global** endpoint:

```
openclaw onboard --auth-choice qwen-api-key
```

For the **China** endpoint:

```
openclaw onboard --auth-choice qwen-api-key-cn
```

3

[Navigate to header](https://docs.openclaw.ai/providers/qwen#)

Set a default model

```
{
  agents: {
    defaults: {
      model: { primary: \"qwen/qwen3.5-plus\" },
    },
  },
}
```

4

[Navigate to header](https://docs.openclaw.ai/providers/qwen#)

Verify the model is available

```
openclaw models list --provider qwen
```

Legacy `modelstudio-*` auth-choice ids and `modelstudio/...` model refs still
work as compatibility aliases, but new setup flows should prefer the canonical
`qwen-*` auth-choice ids and `qwen/...` model refs. If you define an exact
custom `models.providers.modelstudio` entry with another `api` value, that
custom provider owns `modelstudio/...` refs instead of the Qwen compatibility
alias.

**Best for:** pay-as-you-go access through the Standard Model Studio endpoint, including models like `qwen3.6-plus` that may not be available on the Coding Plan.

1

[Navigate to header](https://docs.openclaw.ai/providers/qwen#)

Get your API key

Create or copy an API key from [home.qwencloud.com/api-keys](https://home.qwencloud.com/api-keys).

2

[Navigate to header](https://docs.openclaw.ai/providers/qwen#)

Run onboarding

For the **Global** endpoint:

```
openclaw onboard --auth-choice qwen-standard-api-key
```

For the **China** endpoint:

```
openclaw onboard --auth-choice qwen-standard-api-key-cn
```

3

[Navigate to header](https://docs.openclaw.ai/providers/qwen#)

Set a default model

```
{
  agents: {
    defaults: {
      model: { primary: \"qwen/qwen3.5-plus\" },
    },
  },
}
```

4

[Navigate to header](https://docs.openclaw.ai/providers/qwen#)

Verify the model is available

```
openclaw models list --provider qwen
```

Legacy `modelstudio-*` auth-choice ids and `modelstudio/...` model refs still
work as compatibility aliases, but new setup flows should prefer the canonical
`qwen-*` auth-choice ids and `qwen/...` model refs. If you define an exact
custom `models.providers.modelstudio` entry with another `api` value, that
custom provider owns `modelstudio/...` refs instead of the Qwen compatibility
alias.

## [​](https://docs.openclaw.ai/providers/qwen\\#plan-types-and-endpoints)  Plan types and endpoints

| Plan | Region | Auth choice | Endpoint |
| --- | --- | --- | --- |
| Standard (pay-as-you-go) | China | `qwen-standard-api-key-cn` | `dashscope.aliyuncs.com/compatible-mode/v1` |
| Standard (pay-as-you-go) | Global | `qwen-standard-api-key` | `dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| Coding Plan (subscription) | China | `qwen-api-key-cn` | `coding.dashscope.aliyuncs.com/v1` |
| Coding Plan (subscription) | Global | `qwen-api-key` | `coding-intl.dashscope.aliyuncs.com/v1` |

The provider auto-selects the endpoint based on your auth choice. Canonical
choices use the `qwen-*` family; `modelstudio-*` remains compatibility-only.
You can override with a custom `baseUrl` in config.

**Manage keys:** [home.qwencloud.com/api-keys](https://home.qwencloud.com/api-keys) \\|\n**Docs:** [docs.qwencloud.com](https://docs.qwencloud.com/developer-guides/getting-started/introduction)

## [​](https://docs.openclaw.ai/providers/qwen\\#built-in-catalog)  Built-in catalog

OpenClaw currently ships this bundled Qwen catalog. The configured catalog is
endpoint-aware: Coding Plan configs omit models that are only known to work on
the Standard endpoint.

| Model ref | Input | Context | Notes |
| --- | --- | --- | --- |
| `qwen/qwen3.5-plus` | text, image | 1,000,000 | Default model |
| `qwen/qwen3.6-plus` | text, image | 1,000,000 | Prefer Standard endpoints when you need this model |
| `qwen/qwen3-max-2026-01-23` | text | 262,144 | Qwen Max line |
| `qwen/qwen3-coder-next` | text | 262,144 | Coding |
| `qwen/qwen3-coder-plus` | text | 1,000,000 | Coding |
| `qwen/MiniMax-M2.5` | text | 1,000,000 | Reasoning enabled |
| `qwen/glm-5` | text | 202,752 | GLM |
| `qwen/glm-4.7` | text | 202,752 | GLM |
| `qwen/kimi-k2.5` | text, image | 262,144 | Moonshot AI via Alibaba |

Availability can still vary by endpoint and billing plan even when a model is
present in the bundled catalog.

## [​](https://docs.openclaw.ai/providers/qwen\\#thinking-controls)  Thinking Controls

For reasoning-enabled Qwen Cloud models, the bundled provider maps OpenClaw
thinking levels to DashScope’s top-level `enable_thinking` request flag. Disabled
thinking sends `enable_thinking: false`; other thinking levels send
`enable_thinking: true`.

## [​](https://docs.openclaw.ai/providers/qwen\\#multimodal-add-ons)  Multimodal add-ons

The `qwen` plugin also exposes multimodal capabilities on the **Standard**
DashScope endpoints (not the Coding Plan endpoints):

- **Video understanding** via `qwen-vl-max-latest`
- **Wan video generation** via `wan2.6-t2v` (default), `wan2.6-i2v`, `wan2.6-r2v`, `wan2.6-r2v-flash`, `wan2.7-r2v`

To use Qwen as the default video provider:

```
{
  agents: {
    defaults: {
      videoGenerationModel: { primary: \"qwen/wan2.6-t2v\" },
    },
  },
}
```

See [Video Generation](https://docs.openclaw.ai/tools/video-generation) for shared tool parameters, provider selection, and failover behavior.

## [​](https://docs.openclaw.ai/providers/qwen\\#advanced-configuration)  Advanced configuration

Image and video understanding

The bundled Qwen plugin registers media understanding for images and video
on the **Standard** DashScope endpoints (not the Coding Plan endpoints).

| Property | Value |
| --- | --- |
| Model | `qwen-vl-max-latest` |
| Supported input | Images, video |

Media understanding is auto-resolved from the configured Qwen auth — no
additional config is needed. Ensure you are using a Standard (pay-as-you-go)
endpoint for media understanding support.

Qwen 3.6 Plus availability

`qwen3.6-plus` is available on the Standard (pay-as-you-go) Model Studio
endpoints:

- China: `dashscope.aliyuncs.com/compatible-mode/v1`
- Global: `dashscope-intl.aliyuncs.com/compatible-mode/v1`

If the Coding Plan endpoints return an “unsupported model” error for
`qwen3.6-plus`, switch to Standard (pay-as-you-go) instead of the Coding Plan
endpoint/key pair.

Capability plan

The `qwen` plugin is being positioned as the vendor home for the full Qwen
Cloud surface, not just coding/text models.

- **Text/chat models:** bundled now
- **Tool calling, structured output, thinking:** inherited from the OpenAI-compatible transport
- **Image generation:** planned at the provider-plugin layer
- **Image/video understanding:** bundled now on the Standard endpoint
- **Speech/audio:** planned at the provider-plugin layer
- **Memory embeddings/reranking:** planned through the embedding adapter surface
- **Video generation:** bundled now through the shared video-generation capability

Video generation details

For video generation, OpenClaw maps the configured Qwen region to the matching
DashScope AIGC host before submitting the job:

- Global/Intl: `https://dashscope-intl.aliyuncs.com`
- China: `https://dashscope.aliyuncs.com`

That means a normal `models.providers.qwen.baseUrl` pointing at either the
Coding Plan or Standard Qwen hosts still keeps video generation on the correct
regional DashScope video endpoint.Current bundled Qwen video-generation limits:

- Up to **1** output video per request
- Up to **1** input image
- Up to **4** input videos
- Up to **10 seconds** duration
- Supports `size`, `aspectRatio`, `resolution`, `audio`, and `watermark`
- Reference image/video mode currently requires **remote http(s) URLs**. Local
file paths are rejected up front because the DashScope video endpoint does not
accept uploaded local buffers for those references.

Streaming usage compatibility

Native Model Studio endpoints advertise streaming usage compatibility on the
shared `openai-completions` transport. OpenClaw keys that off endpoint
capabilities now, so DashScope-compatible custom provider ids targeting the
same native hosts inherit the same streaming-usage behavior instead of
requiring the built-in `qwen` provider id specifically.Native-streaming usage compatibility applies to both the Coding Plan hosts and
the Standard DashScope-compatible hosts:

- `https://coding.dashscope.aliyuncs.com/v1`
- `https://coding-intl.dashscope.aliyuncs.com/v1`
- `https://dashscope.aliyuncs.com/compatible-mode/v1`
- `https://dashscope-intl.aliyuncs.com/compatible-mode/v1`

Multimodal endpoint regions

Multimodal surfaces (video understanding and Wan video generation) use the
**Standard** DashScope endpoints, not the Coding Plan endpoints:

- Global/Intl Standard base URL: `https://dashscope-intl.aliyuncs.com/compatible-mode/v1`
- China Standard base URL: `https://dashscope.aliyuncs.com/compatible-mode/v1`

Environment and daemon setup

If the Gateway runs as a daemon (launchd/systemd), make sure `QWEN_API_KEY` is
available to that process (for example, in `~/.openclaw/.env` or via
`env.shellEnv`).

## [​](https://docs.openclaw.ai/providers/qwen\\#related)  Related

[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Video generation** \\\\\n\\\\\nShared video tool parameters and provider selection.](https://docs.openclaw.ai/tools/video-generation)

[**Alibaba (ModelStudio)** \\\\\n\\\\\nLegacy ModelStudio provider and migration notes.](https://docs.openclaw.ai/providers/alibaba)

[**Troubleshooting** \\\\\n\\\\\nGeneral troubleshooting and FAQ.](https://docs.openclaw.ai/help/troubleshooting)

[Qianfan](https://docs.openclaw.ai/providers/qianfan) [Runway](https://docs.openclaw.ai/providers/runway)

Ctrl+I

---

## Google (Gemini) - OpenClaw
**Source:** https://docs.openclaw.ai/providers/google

[Skip to main content](https://docs.openclaw.ai/providers/google#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Google (Gemini)

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/google#getting-started)
- [Capabilities](https://docs.openclaw.ai/providers/google#capabilities)
- [Image generation](https://docs.openclaw.ai/providers/google#image-generation)
- [Video generation](https://docs.openclaw.ai/providers/google#video-generation)
- [Music generation](https://docs.openclaw.ai/providers/google#music-generation)
- [Text-to-speech](https://docs.openclaw.ai/providers/google#text-to-speech)
- [Realtime voice](https://docs.openclaw.ai/providers/google#realtime-voice)
- [Advanced configuration](https://docs.openclaw.ai/providers/google#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/google#related)

The Google plugin provides access to Gemini models through Google AI Studio, plus
image generation, media understanding (image/audio/video), text-to-speech, and web search via
Gemini Grounding.

- Provider: `google`
- Auth: `GEMINI_API_KEY` or `GOOGLE_API_KEY`
- API: Google Gemini API
- Runtime option: `agents.defaults.agentRuntime.id: \"google-gemini-cli\"`
reuses Gemini CLI OAuth while keeping model refs canonical as `google/*`.

## [​](https://docs.openclaw.ai/providers/google\\#getting-started)  Getting started

Choose your preferred auth method and follow the setup steps.

- API key

- Gemini CLI (OAuth)


**Best for:** standard Gemini API access through Google AI Studio.

1

[Navigate to header](https://docs.openclaw.ai/providers/google#)

Run onboarding

```
openclaw onboard --auth-choice gemini-api-key
```

Or pass the key directly:

```
openclaw onboard --non-interactive \\\
  --mode local \\\
  --auth-choice gemini-api-key \\\
  --gemini-api-key \"$GEMINI_API_KEY\"
```

2

[Navigate to header](https://docs.openclaw.ai/providers/google#)

Set a default model

```
{
  agents: {
    defaults: {
      model: { primary: \"google/gemini-3.1-pro-preview\" },
    },
  },
}
```

3

[Navigate to header](https://docs.openclaw.ai/providers/google#)

Verify the model is available

```
openclaw models list --provider google
```

The environment variables `GEMINI_API_KEY` and `GOOGLE_API_KEY` are both accepted. Use whichever you already have configured.

**Best for:** reusing an existing Gemini CLI login via PKCE OAuth instead of a separate API key.

The `google-gemini-cli` provider is an unofficial integration. Some users
report account restrictions when using OAuth this way. Use at your own risk.

1

[Navigate to header](https://docs.openclaw.ai/providers/google#)

Install the Gemini CLI

The local `gemini` command must be available on `PATH`.

```
# Homebrew
brew install gemini-cli

# or npm
npm install -g @google/gemini-cli
```

OpenClaw supports both Homebrew installs and global npm installs, including
common Windows/npm layouts.

2

[Navigate to header](https://docs.openclaw.ai/providers/google#)

Log in via OAuth

```
openclaw models auth login --provider google-gemini-cli --set-default
```

3

[Navigate to header](https://docs.openclaw.ai/providers/google#)

Verify the model is available

```
openclaw models list --provider google
```

- Default model: `google/gemini-3.1-pro-preview`
- Runtime: `google-gemini-cli`
- Alias: `gemini-cli`

**Environment variables:**

- `OPENCLAW_GEMINI_OAUTH_CLIENT_ID`
- `OPENCLAW_GEMINI_OAUTH_CLIENT_SECRET`

(Or the `GEMINI_CLI_*` variants.)

If Gemini CLI OAuth requests fail after login, set `GOOGLE_CLOUD_PROJECT` or
`GOOGLE_CLOUD_PROJECT_ID` on the gateway host and retry.

If login fails before the browser flow starts, make sure the local `gemini`
command is installed and on `PATH`.

`google-gemini-cli/*` model refs are legacy compatibility aliases. New
configs should use `google/*` model refs plus the `google-gemini-cli`
runtime when they want local Gemini CLI execution.

## [​](https://docs.openclaw.ai/providers/google\\#capabilities)  Capabilities

| Capability | Supported |
| --- | --- |
| Chat completions | Yes |
| Image generation | Yes |
| Music generation | Yes |
| Text-to-speech | Yes |
| Realtime voice | Yes (Google Live API) |
| Image understanding | Yes |
| Audio transcription | Yes |
| Video understanding | Yes |
| Web search (Grounding) | Yes |
| Thinking/reasoning | Yes (Gemini 2.5+ / Gemini 3+) |
| Gemma 4 models | Yes |

Gemini 3 models use `thinkingLevel` rather than `thinkingBudget`. OpenClaw maps
Gemini 3, Gemini 3.1, and `gemini-*-latest` alias reasoning controls to
`thinkingLevel` so default/low-latency runs do not send disabled
`thinkingBudget` values.`/think adaptive` keeps Google’s dynamic thinking semantics instead of choosing
a fixed OpenClaw level. Gemini 3 and Gemini 3.1 omit a fixed `thinkingLevel` so
Google can choose the level; Gemini 2.5 sends Google’s dynamic sentinel
`thinkingBudget: -1`.Gemma 4 models (for example `gemma-4-26b-a4b-it`) support thinking mode. OpenClaw
rewrites `thinkingBudget` to a supported Google `thinkingLevel` for Gemma 4.
Setting thinking to `off` preserves thinking disabled instead of mapping to
`MINIMAL`.

## [​](https://docs.openclaw.ai/providers/google\\#image-generation)  Image generation

The bundled `google` image-generation provider defaults to
`google/gemini-3.1-flash-image-preview`.

- Also supports `google/gemini-3-pro-image-preview`
- Generate: up to 4 images per request
- Edit mode: enabled, up to 5 input images
- Geometry controls: `size`, `aspectRatio`, and `resolution`

To use Google as the default image provider:

```
{
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"google/gemini-3.1-flash-image-preview\",
      },
    },
  },
}
```

See [Image Generation](https://docs.openclaw.ai/tools/image-generation) for shared tool parameters, provider selection, and failover behavior.

## [​](https://docs.openclaw.ai/providers/google\\#video-generation)  Video generation

The bundled `google` plugin also registers video generation through the shared
`video_generate` tool.

- Default video model: `google/veo-3.1-fast-generate-preview`
- Modes: text-to-video, image-to-video, and single-video reference flows
- Supports `aspectRatio`, `resolution`, and `audio`
- Current duration clamp: **4 to 8 seconds**

To use Google as the default video provider:

```
{
  agents: {
    defaults: {
      videoGenerationModel: {
        primary: \"google/veo-3.1-fast-generate-preview\",
      },
    },
  },
}
```

See [Video Generation](https://docs.openclaw.ai/tools/video-generation) for shared tool parameters, provider selection, and failover behavior.

## [​](https://docs.openclaw.ai/providers/google\\#music-generation)  Music generation

The bundled `google` plugin also registers music generation through the shared
`music_generate` tool.

- Default music model: `google/lyria-3-clip-preview`
- Also supports `google/lyria-3-pro-preview`
- Prompt controls: `lyrics` and `instrumental`
- Output format: `mp3` by default, plus `wav` on `google/lyria-3-pro-preview`
- Reference inputs: up to 10 images
- Session-backed runs detach through the shared task/status flow, including `action: \"status\"`

To use Google as the default music provider:

```
{
  agents: {
    defaults: {
      musicGenerationModel: {
        primary: \"google/lyria-3-clip-preview\",
      },
    },
  },
}
```

See [Music Generation](https://docs.openclaw.ai/tools/music-generation) for shared tool parameters, provider selection, and failover behavior.

## [​](https://docs.openclaw.ai/providers/google\\#text-to-speech)  Text-to-speech

The bundled `google` speech provider uses the Gemini API TTS path with
`gemini-3.1-flash-tts-preview`.

- Default voice: `Kore`
- Auth: `messages.tts.providers.google.apiKey`, `models.providers.google.apiKey`, `GEMINI_API_KEY`, or `GOOGLE_API_KEY`
- Output: WAV for regular TTS attachments, Opus for voice-note targets, PCM for Talk/telephony
- Voice-note output: Google PCM is wrapped as WAV and transcoded to 48 kHz Opus with `ffmpeg`

To use Google as the default TTS provider:

```
{
  messages: {
    tts: {
      auto: \"always\",
      provider: \"google\",
      providers: {
        google: {
          model: \"gemini-3.1-flash-tts-preview\",
          voiceName: \"Kore\",
          audioProfile: \"Speak professionally with a calm tone.\",
        },
      },
    },
  },
}
```

Gemini API TTS uses natural-language prompting for style control. Set
`audioProfile` to prepend a reusable style prompt before the spoken text. Set
`speakerName` when your prompt text refers to a named speaker.Gemini API TTS also accepts expressive square-bracket audio tags in the text,\nsuch as `[whispers]` or `[laughs]`. To keep tags out of the visible chat reply\nwhile sending them to TTS, put them inside a `[[tts:text]]...[[/tts:text]]`\nblock:\n
```
Here is the clean reply text.

[[tts:text]][whispers] Here is the spoken version.[[/tts:text]]
```

A Google Cloud Console API key restricted to the Gemini API is valid for this\nprovider. This is not the separate Cloud Text-to-Speech API path.

## [​](https://docs.openclaw.ai/providers/google\\#realtime-voice)  Realtime voice

The bundled `google` plugin registers a realtime voice provider backed by the\nGemini Live API for backend audio bridges such as Voice Call and Google Meet.

| Setting | Config path | Default |
| --- | --- | --- |
| Model | `plugins.entries.voice-call.config.realtime.providers.google.model` | `gemini-2.5-flash-native-audio-preview-12-2025` |
| Voice | `...google.voice` | `Kore` |
| Temperature | `...google.temperature` | (unset) |
| VAD start sensitivity | `...google.startSensitivity` | (unset) |
| VAD end sensitivity | `...google.endSensitivity` | (unset) |
| Silence duration | `...google.silenceDurationMs` | (unset) |
| Activity handling | `...google.activityHandling` | Google default, `start-of-activity-interrupts` |
| Turn coverage | `...google.turnCoverage` | Google default, `only-activity` |
| Disable auto VAD | `...google.automaticActivityDetectionDisabled` | `false` |
| API key | `...google.apiKey` | Falls back to `models.providers.google.apiKey`, `GEMINI_API_KEY`, or `GOOGLE_API_KEY` |

Example Voice Call realtime config:\n
```
{
  plugins: {
    entries: {
      \"voice-call\": {
        enabled: true,
        config: {
          realtime: {
            enabled: true,
            provider: \"google\",
            providers: {
              google: {
                model: \"gemini-2.5-flash-native-audio-preview-12-2025\",
                voice: \"Kore\",
                activityHandling: \"start-of-activity-interrupts\",
                turnCoverage: \"only-activity\",
              },
            },
          },
        },
      },\n    },\n  },\n}
```

Google Live API uses bidirectional audio and function calling over a WebSocket.\nOpenClaw adapts telephony/Meet bridge audio to Gemini’s PCM Live API stream and\nkeeps tool calls on the shared realtime voice contract. Leave `temperature`\nunset unless you need sampling changes; OpenClaw omits non-positive values\nbecause Google Live can return transcripts without audio for `temperature: 0`.\nGemini API transcription is enabled without `languageCodes`; the current Google\nSDK rejects language-code hints on this API path.\n
Control UI Talk supports Google Live browser sessions with constrained one-use\ntokens. Backend-only realtime voice providers can also run through the generic\nGateway relay transport, which keeps provider credentials on the Gateway.\n
For maintainer live verification, run\n`OPENAI_API_KEY=... GEMINI_API_KEY=... node --import tsx scripts/dev/realtime-talk-live-smoke.ts`.\nThe Google leg mints the same constrained Live API token shape used by Control\nUI Talk, opens the browser WebSocket endpoint, sends the initial setup payload,\nand waits for `setupComplete`.

## [​](https://docs.openclaw.ai/providers/google\\#advanced-configuration)  Advanced configuration

Direct Gemini cache reuse

For direct Gemini API runs (`api: \"google-generative-ai\"`), OpenClaw\npasses a configured `cachedContent` handle through to Gemini requests.\n
- Configure per-model or global params with either\n`cachedContent` or legacy `cached_content`\n- If both are present, `cachedContent` wins\n- Example value: `cachedContents/prebuilt-context`\n- Gemini cache-hit usage is normalized into OpenClaw `cacheRead` from\nupstream `cachedContentTokenCount`\n
```
{
  agents: {
    defaults: {
      models: {
        \"google/gemini-2.5-pro\": {
          params: {
            cachedContent: \"cachedContents/prebuilt-context\",
          },\n        },\n      },\n    },\n  },\n}
```

Gemini CLI JSON usage notes

When using the `google-gemini-cli` OAuth provider, OpenClaw normalizes\nthe CLI JSON output as follows:\n
- Reply text comes from the CLI JSON `response` field.\n- Usage falls back to `stats` when the CLI leaves `usage` empty.\n- `stats.cached` is normalized into OpenClaw `cacheRead`.\n- If `stats.input` is missing, OpenClaw derives input tokens from\n`stats.input_tokens - stats.cached`.

Environment and daemon setup

If the Gateway runs as a daemon (launchd/systemd), make sure `GEMINI_API_KEY`\nis available to that process (for example, in `~/.openclaw/.env` or via\n`env.shellEnv`).

## [​](https://docs.openclaw.ai/providers/google\\#related)  Related

[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)\n
[**Image generation** \\\\\n\\\\\nShared image tool parameters and provider selection.](https://docs.openclaw.ai/tools/image-generation)\n
[**Video generation** \\\\\n\\\\\nShared video tool parameters and provider selection.](https://docs.openclaw.ai/tools/video-generation)\n
[**Music generation** \\\\\n\\\\\nShared music tool parameters and provider selection.](https://docs.openclaw.ai/tools/music-generation)\n
[GLM (Zhipu)](https://docs.openclaw.ai/providers/glm) [Gradium](https://openclaw.ai/providers/gradium)\n
Ctrl+I

---

## LiteLLM - OpenClaw
**Source:** https://docs.openclaw.ai/providers/litellm

[Skip to main content](https://docs.openclaw.ai/providers/litellm#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

LiteLLM

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/providers/litellm#quick-start)
- [Configuration](https://docs.openclaw.ai/providers/litellm#configuration)
- [Environment variables](https://docs.openclaw.ai/providers/litellm#environment-variables)
- [Config file](https://docs.openclaw.ai/providers/litellm#config-file)
- [Advanced configuration](https://docs.openclaw.ai/providers/litellm#advanced-configuration)
- [Image generation](https://docs.openclaw.ai/providers/litellm#image-generation)
- [Related](https://docs.openclaw.ai/providers/litellm#related)

[LiteLLM](https://litellm.ai/) is an open-source LLM gateway that provides a unified API to 100+ model providers. Route OpenClaw through LiteLLM to get centralized cost tracking, logging, and the flexibility to switch backends without changing your OpenClaw config.

**Why use LiteLLM with OpenClaw?**

- **Cost tracking** — See exactly what OpenClaw spends across all models
- **Model routing** — Switch between Claude, GPT-4, Gemini, Bedrock without config changes
- **Virtual keys** — Create keys with spend limits for OpenClaw
- **Logging** — Full request/response logs for debugging
- **Fallbacks** — Automatic failover if your primary provider is down

## [​](https://docs.openclaw.ai/providers/litellm\\#quick-start)  Quick start

- Onboarding (recommended)

- Manual setup


**Best for:** fastest path to a working LiteLLM setup.

1

[Navigate to header](https://docs.openclaw.ai/providers/litellm#)

Run onboarding

```
openclaw onboard --auth-choice litellm-api-key
```

For non-interactive setup against a remote proxy, pass the proxy URL explicitly:

```
openclaw onboard --non-interactive --auth-choice litellm-api-key --litellm-api-key \"$LITELLM_API_KEY\" --custom-base-url \"https://litellm.example/v1\"
```

**Best for:** full control over installation and config.

1

[Navigate to header](https://docs.openclaw.ai/providers/litellm#)

Start LiteLLM Proxy

```
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2

[Navigate to header](https://docs.openclaw.ai/providers/litellm#)

Point OpenClaw to LiteLLM

```
export LITELLM_API_KEY=\"your-litellm-key\"

openclaw
```

That’s it. OpenClaw now routes through LiteLLM.

## [​](https://docs.openclaw.ai/providers/litellm\\#configuration)  Configuration

### [​](https://docs.openclaw.ai/providers/litellm\\#environment-variables)  Environment variables

```
export LITELLM_API_KEY=\"sk-litellm-key\"
```

### [​](https://docs.openclaw.ai/providers/litellm\\#config-file)  Config file

```
{
  models: {
    providers: {
      litellm: {
        baseUrl: \"http://localhost:4000\",
        apiKey: \"${LITELLM_API_KEY}\",
        api: \"openai-completions\",
        models: [\\
          {\\
            id: \"claude-opus-4-6\",\\
            name: \"Claude Opus 4.6\",\\
            reasoning: true,\\
            input: [\"text\", \"image\"],\\
            contextWindow: 200000,\\
            maxTokens: 64000,\\
          },\\
          {\\
            id: \"gpt-4o\",\\
            name: \"GPT-4o\",\\
            reasoning: false,\\
            input: [\"text\", \"image\"],\\
            contextWindow: 128000,\\
            maxTokens: 8192,\\
          },\\
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: \"litellm/claude-opus-4-6\" },
    },
  },
}
```

## [​](https://docs.openclaw.ai/providers/litellm\\#advanced-configuration)  Advanced configuration

### [​](https://docs.openclaw.ai/providers/litellm\\#image-generation)  Image generation

LiteLLM can also back the `image_generate` tool through OpenAI-compatible
`/images/generations` and `/images/edits` routes. Configure a LiteLLM image
model under `agents.defaults.imageGenerationModel`:

```
{
  models: {
    providers: {
      litellm: {
        baseUrl: \"http://localhost:4000\",
        apiKey: \"${LITELLM_API_KEY}\",
      },
    },
  },
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"litellm/gpt-image-2\",
        timeoutMs: 180_000,
      },
    },
  },
}
```

Loopback LiteLLM URLs such as `http://localhost:4000` work without a global
private-network override. For a LAN-hosted proxy, set
`models.providers.litellm.request.allowPrivateNetwork: true` because the API key
will be sent to the configured proxy host.

Virtual keys

Create a dedicated key for OpenClaw with spend limits:

```
curl -X POST \"http://localhost:4000/key/generate\" \\\
  -H \"Authorization: Bearer $LITELLM_MASTER_KEY\" \\\
  -H \"Content-Type: application/json\" \\\
  -d '{
    \"key_alias\": \"openclaw\",
    \"max_budget\": 50.00,
    \"budget_duration\": \"monthly\"
  }'
```

Use the generated key as `LITELLM_API_KEY`.

Model routing

LiteLLM can route model requests to different backends. Configure in your LiteLLM `config.yaml`:

```
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw keeps requesting `claude-opus-4-6` — LiteLLM handles the routing.

Viewing usage

Check LiteLLM’s dashboard or API:

```
# Key info
curl \"http://localhost:4000/key/info\" \\\
  -H \"Authorization: Bearer sk-litellm-key\"

# Spend logs
curl \"http://localhost:4000/spend/logs\" \\\
  -H \"Authorization: Bearer $LITELLM_MASTER_KEY\"
```

Proxy behavior notes

- LiteLLM runs on `http://localhost:4000` by default
- OpenClaw connects through LiteLLM’s proxy-style OpenAI-compatible `/v1`
endpoint
- Native OpenAI-only request shaping does not apply through LiteLLM:
no `service_tier`, no Responses `store`, no prompt-cache hints, and no
OpenAI reasoning-compat payload shaping
- Hidden OpenClaw attribution headers (`originator`, `version`, `User-Agent`)
are not injected on custom LiteLLM base URLs

For general provider configuration and failover behavior, see [Model Providers](https://docs.openclaw.ai/concepts/model-providers).

## [​](https://docs.openclaw.ai/providers/litellm\\#related)  Related

[**LiteLLM Docs** \\\\\n\\\\\nOfficial LiteLLM documentation and API reference.](https://docs.litellm.ai/)

[**Model selection** \\\\\n\\\\\nOverview of all providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Configuration** \\\\\n\\\\\nFull config reference.](https://docs.openclaw.ai/gateway/configuration)

[**Model selection** \\\\\n\\\\\nHow to choose and configure models.](https://docs.openclaw.ai/concepts/models)

[Kilocode](https://docs.openclaw.ai/providers/kilocode) [LM Studio](https://docs.openclaw.ai/providers/lmstudio)

Ctrl+I

---

## GLM (Zhipu) - OpenClaw
**Source:** https://docs.openclaw.ai/providers/glm

[Skip to main content](https://docs.openclaw.ai/providers/glm#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

GLM (Zhipu)

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [GLM models](https://docs.openclaw.ai/providers/glm#glm-models)
- [Getting started](https://docs.openclaw.ai/providers/glm#getting-started)
- [Config example](https://docs.openclaw.ai/providers/glm#config-example)
- [Built-in catalog](https://docs.openclaw.ai/providers/glm#built-in-catalog)
- [Advanced configuration](https://docs.openclaw.ai/providers/glm#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/glm#related)

# [​](https://docs.openclaw.ai/providers/glm\\#glm-models)  GLM models

GLM is a **model family** (not a company) available through the Z.AI platform. In OpenClaw, GLM
models are accessed via the `zai` provider and model IDs like `zai/glm-5`.

## [​](https://docs.openclaw.ai/providers/glm\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/glm#)

Choose an auth route and run onboarding

Pick the onboarding choice that matches your Z.AI plan and region:

| Auth choice | Best for |
| --- | --- |
| `zai-api-key` | Generic API-key setup with endpoint auto-detection |
| `zai-coding-global` | Coding Plan users (global) |
| `zai-coding-cn` | Coding Plan users (China region) |
| `zai-global` | General API (global) |
| `zai-cn` | General API (China region) |

```
# Example: generic auto-detect
openclaw onboard --auth-choice zai-api-key

# Example: Coding Plan global
openclaw onboard --auth-choice zai-coding-global
```

2

[Navigate to header](https://docs.openclaw.ai/providers/glm#)

Set GLM as the default model

```
openclaw config set agents.defaults.model.primary \"zai/glm-5.1\"
```

3

[Navigate to header](https://docs.openclaw.ai/providers/glm#)

Verify models are available

```
openclaw models list --provider zai
```

## [​](https://docs.openclaw.ai/providers/glm\\#config-example)  Config example

```
{
  env: { ZAI_API_KEY: \"sk-...\" },
  agents: { defaults: { model: { primary: \"zai/glm-5.1\" } } },
}
```

`zai-api-key` lets OpenClaw detect the matching Z.AI endpoint from the key and
apply the correct base URL automatically. Use the explicit regional choices when
you want to force a specific Coding Plan or general API surface.

## [​](https://docs.openclaw.ai/providers/glm\\#built-in-catalog)  Built-in catalog

OpenClaw currently seeds the bundled `zai` provider with these GLM refs:

| Model | Model |
| --- | --- |
| `glm-5.1` | `glm-4.7` |
| `glm-5` | `glm-4.7-flash` |
| `glm-5-turbo` | `glm-4.7-flashx` |
| `glm-5v-turbo` | `glm-4.6` |
| `glm-4.5` | `glm-4.6v` |
| `glm-4.5-air` |  |
| `glm-4.5-flash` |  |
| `glm-4.5v` |  |

The default bundled model ref is `zai/glm-5.1`. GLM versions and availability
can change; check Z.AI’s docs for the latest.

## [​](https://docs.openclaw.ai/providers/glm\\#advanced-configuration)  Advanced configuration

Endpoint auto-detection

When you use the `zai-api-key` auth choice, OpenClaw inspects the key format
to determine the correct Z.AI base URL. Explicit regional choices
(`zai-coding-global`, `zai-coding-cn`, `zai-global`, `zai-cn`) override
auto-detection and pin the endpoint directly.

Provider details

GLM models are served by the `zai` runtime provider. For full provider
configuration, regional endpoints, and additional capabilities, see
[Z.AI provider docs](https://docs.openclaw.ai/providers/zai).

## [​](https://docs.openclaw.ai/providers/glm\\#related)  Related

[**Z.AI provider** \\\\\n\\\\\nFull Z.AI provider configuration and regional endpoints.](https://docs.openclaw.ai/providers/zai)\n
[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)\n
[GitHub Copilot](https://docs.openclaw.ai/providers/github-copilot) [Google (Gemini)](https://docs.openclaw.ai/providers/google)

Ctrl+I

---

## Moonshot AI - OpenClaw
**Source:** https://docs.openclaw.ai/providers/moonshot

[Skip to main content](https://docs.openclaw.ai/providers/moonshot#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nProviders\n\nMoonshot AI\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Built-in model catalog](https://docs.openclaw.ai/providers/moonshot#built-in-model-catalog)\n- [Getting started](https://docs.openclaw.ai/providers/moonshot#getting-started)\n- [Config example](https://docs.openclaw.ai/providers/moonshot#config-example)\n- [Kimi web search](https://docs.openclaw.ai/providers/moonshot#kimi-web-search)\n- [Advanced configuration](https://docs.openclaw.ai/providers/moonshot#advanced-configuration)\n- [Related](https://docs.openclaw.ai/providers/moonshot#related)\n\nMoonshot provides the Kimi API with OpenAI-compatible endpoints. Configure the\nprovider and set the default model to `moonshot/kimi-k2.6`, or use\nKimi Coding with `kimi/kimi-code`.\n\nMoonshot and Kimi Coding are **separate providers**. Keys are not interchangeable, endpoints differ, and model refs differ (`moonshot/...` vs `kimi/...`).\n\n## [​](https://docs.openclaw.ai/providers/moonshot\\#built-in-model-catalog)  Built-in model catalog\n\n| Model ref | Name | Reasoning | Input | Context | Max output |\n| --- | --- | --- | --- | --- | --- |\n| `moonshot/kimi-k2.6` | Kimi K2.6 | No | text, image | 262,144 | 262,144 |\n| `moonshot/kimi-k2.5` | Kimi K2.5 | No | text, image | 262,144 | 262,144 |\n| `moonshot/kimi-k2-thinking` | Kimi K2 Thinking | Yes | text | 262,144 | 262,144 |\n| `moonshot/kimi-k2-thinking-turbo` | Kimi K2 Thinking Turbo | Yes | text | 262,144 | 262,144 |\n| `moonshot/kimi-k2-turbo` | Kimi K2 Turbo | No | text | 256,000 | 16,384 |\n\nBundled cost estimates for current Moonshot-hosted K2 models use Moonshot’s\npublished pay-as-you-go rates: Kimi K2.6 is 0.16/MTokcachehit,0.16/MTok cache hit,\n0.16/MTokcachehit,0.95/MTok input, and 4.00/MTokoutput;KimiK2.5is4.00/MTok output; Kimi K2.5 is 4.00/MTokoutput;KimiK2.5is0.10/MTok cache hit,\n0.60/MTokinput,and0.60/MTok input, and 0.60/MTokinput,and3.00/MTok output. Other legacy catalog entries keep\nzero-cost placeholders unless you override them in config.\n\n## [​](https://docs.openclaw.ai/providers/moonshot\\#getting-started)  Getting started\n\nChoose your provider and follow the setup steps.\n\n- Moonshot API\n\n- Kimi Coding\n\n\n**Best for:** Kimi K2 models via the Moonshot Open Platform.\n\n1\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nChoose your endpoint region\n\n| Auth choice | Endpoint | Region |\n| --- | --- | --- |\n| `moonshot-api-key` | `https://api.moonshot.ai/v1` | International |\n| `moonshot-api-key-cn` | `https://api.moonshot.cn/v1` | China |\n\n2\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nRun onboarding\n\n```\nopenclaw onboard --auth-choice moonshot-api-key\n```\n\nOr for the China endpoint:\n\n```\nopenclaw onboard --auth-choice moonshot-api-key-cn\n```\n\n3\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nSet a default model\n\n```\n{\n  agents: {\n    defaults: {\n      model: { primary: \"moonshot/kimi-k2.6\" },\n    },\n  },\n}\n```\n\n4\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nVerify models are available\n\n```\nopenclaw models list --provider moonshot\n```\n\n5\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nRun a live smoke test\n\nUse an isolated state dir when you want to verify model access and cost\ntracking without touching your normal sessions:\n\n```\nOPENCLAW_CONFIG_PATH=/tmp/openclaw-kimi/openclaw.json \\\nOPENCLAW_STATE_DIR=/tmp/openclaw-kimi \\\nopenclaw agent --local \\\n  --session-id live-kimi-cost \\\n  --message \'Reply exactly: KIMI_LIVE_OK\' \\\n  --thinking off \\\n  --json\n```\n\nThe JSON response should report `provider: \"moonshot\"` and\n`model: \"kimi-k2.6\"`. The assistant transcript entry stores normalized\ntoken usage plus estimated cost under `usage.cost` when Moonshot returns\nusage metadata.\n\n### [​](https://docs.openclaw.ai/providers/moonshot\\#config-example)  Config example\n\n```\n{\n  env: { MOONSHOT_API_KEY: \"sk-...\" },\n  agents: {\n    defaults: {\n      model: { primary: \"moonshot/kimi-k2.6\" },\n      models: {\n        // moonshot-kimi-k2-aliases:start\n        \"moonshot/kimi-k2.6\": { alias: \"Kimi K2.6\" },\n        \"moonshot/kimi-k2.5\": { alias: \"Kimi K2.5\" },\n        \"moonshot/kimi-k2-thinking\": { alias: \"Kimi K2 Thinking\" },\n        \"moonshot/kimi-k2-thinking-turbo\": { alias: \"Kimi K2 Thinking Turbo\" },\n        \"moonshot/kimi-k2-turbo\": { alias: \"Kimi K2 Turbo\" },\n        // moonshot-kimi-k2-aliases:end\n      },\n    },\n  },\n  models: {\n    mode: \"merge\",\n    providers: {\n      moonshot: {\n        baseUrl: \"https://api.moonshot.ai/v1\",\n        apiKey: \"${MOONSHOT_API_KEY}\",\n        api: \"openai-completions\",\n        models: [\\\n          // moonshot-kimi-k2-models:start\\\n          {\\\n            id: \"kimi-k2.6\",\\\n            name: \"Kimi K2.6\",\\\n            reasoning: false,\\\n            input: [\"text\", \"image\"],\\\n            cost: { input: 0.95, output: 4, cacheRead: 0.16, cacheWrite: 0 },\\\n            contextWindow: 262144,\\\n            maxTokens: 262144,\\\n          },\\\n          {\\\n            id: \"kimi-k2.5\",\\\n            name: \"Kimi K2.5\",\\\n            reasoning: false,\\\n            input: [\"text\", \"image\"],\\\n            cost: { input: 0.6, output: 3, cacheRead: 0.1, cacheWrite: 0 },\\\n            contextWindow: 262144,\\\n            maxTokens: 262144,\\\n          },\\\n          {\\\n            id: \"kimi-k2-thinking\",\\\n            name: \"Kimi K2 Thinking\",\\\n            reasoning: true,\\\n            input: [\"text\"],\\\n            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\n            contextWindow: 262144,\\\n            maxTokens: 262144,\\\n          },\\\n          {\\\n            id: \"kimi-k2-thinking-turbo\",\\\n            name: \"Kimi K2 Thinking Turbo\",\\\n            reasoning: true,\\\n            input: [\"text\"],\\\n            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\n            contextWindow: 262144,\\\n            maxTokens: 262144,\\\n          },\\\n          {\\\n            id: \"kimi-k2-turbo\",\\\n            name: \"Kimi K2 Turbo\",\\\n            reasoning: false,\\\n            input: [\"text\"],\\\n            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\n            contextWindow: 256000,\\\n            maxTokens: 16384,\\\n          },\\\n          // moonshot-kimi-k2-models:end\\\n        ],\n      },\n    },\n  },\n}\n```\n\n**Best for:** code-focused tasks via the Kimi Coding endpoint.\n\nKimi Coding uses a different API key and provider prefix (`kimi/...`) than Moonshot (`moonshot/...`). Legacy model ref `kimi/k2p5` remains accepted as a compatibility id.\n\n1\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nRun onboarding\n\n```\nopenclaw onboard --auth-choice kimi-code-api-key\n```\n\n2\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nSet a default model\n\n```\n{\n  agents: {\n    defaults: {\n      model: { primary: \"kimi/kimi-code\" },\n    },\n  },\n}\n```\n\n3\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nVerify the model is available\n\n```\nopenclaw models list --provider kimi\n```\n\n### [​](https://docs.openclaw.ai/providers/moonshot\\#config-example-2)  Config example\n\n```\n{\n  env: { KIMI_API_KEY: \"sk-...\" },\n  agents: {\n    defaults: {\n      model: { primary: \"kimi/kimi-code\" },\n      models: {\n        \"kimi/kimi-code\": { alias: \"Kimi\" },\n      },\n    },\n  },\n}\n```\n\n## [​](https://docs.openclaw.ai/providers/moonshot\\#kimi-web-search)  Kimi web search\n\nOpenClaw also ships **Kimi** as a `web_search` provider, backed by Moonshot web\nsearch.\n\n1\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nRun interactive web search setup\n\n```\nopenclaw configure --section web\n```\n\nChoose **Kimi** in the web-search section to store\n`plugins.entries.moonshot.config.webSearch.*`.\n\n2\n\n[Navigate to header](https://openclaw.ai/providers/moonshot#)\n\nConfigure the web search region and model\n\nInteractive setup prompts for:\n\n| Setting | Options |\n| --- | --- |\n| API region | `https://api.moonshot.ai/v1` (international) or `https://api.moonshot.cn/v1` (China) |\n| Web search model | Defaults to `kimi-k2.6` |\n\nConfig lives under `plugins.entries.moonshot.config.webSearch`:\n\n```\n{\n  plugins: {\n    entries: {\n      moonshot: {\n        config: {\n          webSearch: {\n            apiKey: \"sk-...\", // or use KIMI_API_KEY / MOONSHOT_API_KEY\n            baseUrl: \"https://api.moonshot.ai/v1\",\n            model: \"kimi-k2.6\",\n          },\n        },\n      },\n    },\n  },\n  tools: {\n    web: {\n      search: {\n        provider: \"kimi\",\n      },\n    },\n  },\n}\n```\n\n## [​](https://docs.openclaw.ai/providers/moonshot\\#advanced-configuration)  Advanced configuration\n\nNative thinking mode\n\nMoonshot Kimi supports binary native thinking:\n\n- `thinking: { type: \"enabled\" }`\n- `thinking: { type: \"disabled\" }`\n\nConfigure it per model via `agents.defaults.models.<provider/model>.params`:\n\n```\n{\n  agents: {\n    defaults: {\n      models: {\n        \"moonshot/kimi-k2.6\": {\n          params: {\n            thinking: { type: \"disabled\" },\n          },\n        },\n      },\n    },\n  },\n}\n```\n\nOpenClaw also maps runtime `/think` levels for Moonshot:\n\n| `/think` level | Moonshot behavior |\n| --- | --- |\n| `/think off` | `thinking.type=disabled` |\n| Any non-off level | `thinking.type=enabled` |\n\nWhen Moonshot thinking is enabled, `tool_choice` must be `auto` or `none`. OpenClaw normalizes incompatible `tool_choice` values to `auto` for compatibility.\n\nKimi K2.6 also accepts an optional `thinking.keep` field that controls\nmulti-turn retention of `reasoning_content`. Set it to `\"all\"` to keep full\nreasoning across turns; omit it (or leave it `null`) to use the server\ndefault strategy. OpenClaw only forwards `thinking.keep` for\n`moonshot/kimi-k2.6` and strips it from other models.\n\n```\n{\n  agents: {\n    defaults: {\n      models: {\n        \"moonshot/kimi-k2.6\": {\n          params: {\n            thinking: { type: \"enabled\", keep: \"all\" },\n          },\n        },\n      },\n    },\n  },\n}\n```\n\nTool call id sanitization\n\nMoonshot Kimi serves tool\\_call ids shaped like `functions.<name>:<index>`. OpenClaw preserves them unchanged so multi-turn tool calls keep working.To force strict sanitization on a custom OpenAI-compatible provider, set `sanitizeToolCallIds: true`:\n\n```\n{\n  models: {\n    providers: {\n      \"my-kimi-proxy\": {\n        api: \"openai-completions\",\n        sanitizeToolCallIds: true,\n      },\n    },\n  },\n}\n```\n\nStreaming usage compatibility\n\nNative Moonshot endpoints (`https://api.moonshot.ai/v1` and\n`https://api.moonshot.cn/v1`) advertise streaming usage compatibility on the\nshared `openai-completions` transport. OpenClaw keys that off endpoint\ncapabilities, so compatible custom provider ids targeting the same native\nMoonshot hosts inherit the same streaming-usage behavior.With the bundled K2.6 pricing, streamed usage that includes input, output,\nand cache-read tokens is also converted into local estimated USD cost for\n`/status`, `/usage full`, `/usage cost`, and transcript-backed session\naccounting.\n\nEndpoint and model ref reference\n\n| Provider | Model ref prefix | Endpoint | Auth env var |\n| --- | --- | --- | --- |\n| Moonshot | `moonshot/` | `https://api.moonshot.ai/v1` | `MOONSHOT_API_KEY` |\n| Moonshot CN | `moonshot/` | `https://api.moonshot.cn/v1` | `MOONSHOT_API_KEY` |\n| Kimi Coding | `kimi/` | Kimi Coding endpoint | `KIMI_API_KEY` |\n| Web search | N/A | Same as Moonshot API region | `KIMI_API_KEY` or `MOONSHOT_API_KEY` |\n\n- Kimi web search uses `KIMI_API_KEY` or `MOONSHOT_API_KEY`, and defaults to `https://api.moonshot.ai/v1` with model `kimi-k2.6`.\n- Override pricing and context metadata in `models.providers` if needed.\n- If Moonshot publishes different context limits for a model, adjust `contextWindow` accordingly.\n\n## [​](https://docs.openclaw.ai/providers/moonshot\\#related)  Related\n\n[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)\n\n[**Web search** \\\\\n\\\\\nConfiguring web search providers including Kimi.](https://docs.openclaw.ai/tools/web)\n\n[**Configuration reference** \\\\\n\\\\\nFull config schema for providers, models, and plugins.](https://docs.openclaw.ai/gateway/configuration-reference)\n\n[**Moonshot Open Platform** \\\\\n\\\\\nMoonshot API key management and documentation.](https://platform.moonshot.ai/)\n\n[Mistral](https://docs.openclaw.ai/providers/mistral) [NVIDIA](https://docs.openclaw.ai/providers/nvidia)\n\nCtrl+I

---

## SenseAudio - OpenClaw
**Source:** https://docs.openclaw.ai/providers/senseaudio

[Skip to main content](https://docs.openclaw.ai/providers/senseaudio#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

SenseAudio

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [SenseAudio](https://docs.openclaw.ai/providers/senseaudio#senseaudio)
- [Getting Started](https://docs.openclaw.ai/providers/senseaudio#getting-started)
- [Options](https://docs.openclaw.ai/providers/senseaudio#options)

# [​](https://docs.openclaw.ai/providers/senseaudio\#senseaudio)  SenseAudio

SenseAudio can transcribe inbound audio/voice-note attachments through
OpenClaw’s shared `tools.media.audio` pipeline. OpenClaw posts multipart audio
to the OpenAI-compatible transcription endpoint and injects the returned text
as `{{Transcript}}` plus an `[Audio]` block.

| Detail | Value |
| --- | --- |
| Website | [senseaudio.cn](https://senseaudio.cn/) |
| Docs | [senseaudio.cn/docs](https://senseaudio.cn/docs) |
| Auth | `SENSEAUDIO_API_KEY` |
| Default model | `senseaudio-asr-pro-1.5-260319` |
| Default URL | `https://api.senseaudio.cn/v1` |

## [​](https://docs.openclaw.ai/providers/senseaudio\#getting-started)  Getting Started

1

[Navigate to header](https://docs.openclaw.ai/providers/senseaudio#)

Set your API key

```
export SENSEAUDIO_API_KEY=\"...\"
```

2

[Navigate to header](https://docs.openclaw.ai/providers/senseaudio#)

Enable the audio provider

```
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: \"senseaudio\", model: \"senseaudio-asr-pro-1.5-260319\" }],
      },
    },
  },
}
```

3

[Navigate to header](https://docs.openclaw.ai/providers/senseaudio#)

Send a voice note

Send an audio message through any connected channel. OpenClaw uploads the
audio to SenseAudio and uses the transcript in the reply pipeline.

## [​](https://docs.openclaw.ai/providers/senseaudio\#options)  Options

| Option | Path | Description |
| --- | --- | --- |
| `model` | `tools.media.audio.models[].model` | SenseAudio ASR model id |
| `language` | `tools.media.audio.models[].language` | Optional language hint |
| `prompt` | `tools.media.audio.prompt` | Optional transcription prompt |
| `baseUrl` | `tools.media.audio.baseUrl` or model | Override the OpenAI-compatible base |
| `headers` | `tools.media.audio.request.headers` | Extra request headers |

SenseAudio is batch STT only in OpenClaw. Voice Call realtime transcription
continues to use providers with streaming STT support.

Ctrl+I

---

## OpenCode Go - OpenClaw
**Source:** https://docs.openclaw.ai/providers/opencode-go

[Skip to main content](https://docs.openclaw.ai/providers/opencode-go#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

OpenCode Go

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Built-in catalog](https://docs.openclaw.ai/providers/opencode-go#built-in-catalog)
- [Getting started](https://docs.openclaw.ai/providers/opencode-go#getting-started)
- [Config example](https://docs.openclaw.ai/providers/opencode-go#config-example)
- [Advanced configuration](https://docs.openclaw.ai/providers/opencode-go#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/opencode-go#related)

OpenCode Go is the Go catalog within [OpenCode](https://docs.openclaw.ai/providers/opencode).
It uses the same `OPENCODE_API_KEY` as the Zen catalog, but keeps the runtime
provider id `opencode-go` so upstream per-model routing stays correct.

| Property | Value |
| --- | --- |
| Runtime provider | `opencode-go` |
| Auth | `OPENCODE_API_KEY` |
| Parent setup | [OpenCode](https://docs.openclaw.ai/providers/opencode) |

## [​](https://docs.openclaw.ai/providers/opencode-go\\#built-in-catalog)  Built-in catalog

OpenClaw sources most Go catalog rows from the bundled pi model registry and
supplements current upstream rows while the registry catches up. Run
`openclaw models list --provider opencode-go` for the current model list.The provider includes:

| Model ref | Name |
| --- | --- |
| `opencode-go/glm-5` | GLM-5 |
| `opencode-go/glm-5.1` | GLM-5.1 |
| `opencode-go/kimi-k2.5` | Kimi K2.5 |
| `opencode-go/kimi-k2.6` | Kimi K2.6 (3x limits) |
| `opencode-go/deepseek-v4-pro` | DeepSeek V4 Pro |
| `opencode-go/deepseek-v4-flash` | DeepSeek V4 Flash |
| `opencode-go/mimo-v2-omni` | MiMo V2 Omni |
| `opencode-go/mimo-v2-pro` | MiMo V2 Pro |
| `opencode-go/minimax-m2.5` | MiniMax M2.5 |
| `opencode-go/minimax-m2.7` | MiniMax M2.7 |
| `opencode-go/qwen3.5-plus` | Qwen3.5 Plus |
| `opencode-go/qwen3.6-plus` | Qwen3.6 Plus |

## [​](https://docs.openclaw.ai/providers/opencode-go\\#getting-started)  Getting started

- Interactive

- Non-interactive


1

[Navigate to header](https://docs.openclaw.ai/providers/opencode-go#)

Run onboarding

```
openclaw onboard --auth-choice opencode-go
```

2

[Navigate to header](https://docs.openclaw.ai/providers/opencode-go#)

Set a Go model as default

```
openclaw config set agents.defaults.model.primary \"opencode-go/kimi-k2.6\"
```

3

[Navigate to header](https://docs.openclaw.ai/providers/opencode-go#)

Verify models are available

```
openclaw models list --provider opencode-go
```

1

[Navigate to header](https://docs.openclaw.ai/providers/opencode-go#)

Pass the key directly

```
openclaw onboard --opencode-go-api-key \"$OPENCODE_API_KEY\"
```

2

[Navigate to header](https://docs.openclaw.ai/providers/opencode-go#)

Verify models are available

```
openclaw models list --provider opencode-go
```

## [​](https://docs.openclaw.ai/providers/opencode-go\\#config-example)  Config example

```
{
  env: { OPENCODE_API_KEY: \"YOUR_API_KEY_HERE\" }, // pragma: allowlist secret
  agents: { defaults: { model: { primary: \"opencode-go/kimi-k2.6\" } } },
}
```

## [​](https://docs.openclaw.ai/providers/opencode-go\\#advanced-configuration)  Advanced configuration

Routing behavior

OpenClaw handles per-model routing automatically when the model ref uses
`opencode-go/...`. No additional provider config is required.

Runtime ref convention

Runtime refs stay explicit: `opencode/...` for Zen, `opencode-go/...` for Go.
This keeps upstream per-model routing correct across both catalogs.

Shared credentials

The same `OPENCODE_API_KEY` is used by both the Zen and Go catalogs. Entering
the key during setup stores credentials for both runtime providers.

See [OpenCode](https://docs.openclaw.ai/providers/opencode) for the shared onboarding overview and the full
Zen + Go catalog reference.

## [​](https://docs.openclaw.ai/providers/opencode-go\\#related)  Related

[**OpenCode (parent)** \\\\\n\\\\\nShared onboarding, catalog overview, and advanced notes.](https://docs.openclaw.ai/providers/opencode)

[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[OpenCode](https://docs.openclaw.ai/providers/opencode) [OpenRouter](https://docs.openclaw.ai/providers/openrouter)

Ctrl+I

---

## Tencent Cloud (TokenHub) - OpenClaw
**Source:** https://docs.openclaw.ai/providers/tencent

[Skip to main content](https://docs.openclaw.ai/providers/tencent#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Tencent Cloud (TokenHub)

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Tencent Cloud TokenHub](https://docs.openclaw.ai/providers/tencent#tencent-cloud-tokenhub)
- [Quick start](https://docs.openclaw.ai/providers/tencent#quick-start)
- [Non-interactive setup](https://docs.openclaw.ai/providers/tencent#non-interactive-setup)
- [Built-in catalog](https://docs.openclaw.ai/providers/tencent#built-in-catalog)
- [Endpoint override](https://docs.openclaw.ai/providers/tencent#endpoint-override)
- [Notes](https://docs.openclaw.ai/providers/tencent#notes)
- [Environment note](https://docs.openclaw.ai/providers/tencent#environment-note)
- [Related documentation](https://docs.openclaw.ai/providers/tencent#related-documentation)

# [​](https://docs.openclaw.ai/providers/tencent\\#tencent-cloud-tokenhub)  Tencent Cloud TokenHub

Tencent Cloud ships as a **bundled provider plugin** in OpenClaw. It gives access to Tencent Hy3 preview through the TokenHub endpoint (`tencent-tokenhub`).The provider uses an OpenAI-compatible API.

| Property | Value |
| --- | --- |
| Provider | `tencent-tokenhub` |
| Default model | `tencent-tokenhub/hy3-preview` |
| Auth | `TOKENHUB_API_KEY` |
| API | OpenAI-compatible chat completions |
| Base URL | `https://tokenhub.tencentmaas.com/v1` |
| Global URL | `https://tokenhub-intl.tencentmaas.com/v1` |

## [​](https://docs.openclaw.ai/providers/tencent\\#quick-start)  Quick start

1

[Navigate to header](https://docs.openclaw.ai/providers/tencent#)

Create a TokenHub API key

Create an API key in Tencent Cloud TokenHub. If you choose a limited access scope for the key, include **Hy3 preview** in the allowed models.

2

[Navigate to header](https://docs.openclaw.ai/providers/tencent#)

Run onboarding

```
openclaw onboard --auth-choice tokenhub-api-key
```

3

[Navigate to header](https://docs.openclaw.ai/providers/tencent#)

Verify the model

```
openclaw models list --provider tencent-tokenhub
```

## [​](https://docs.openclaw.ai/providers/tencent\\#non-interactive-setup)  Non-interactive setup

```
openclaw onboard --non-interactive \\\
  --mode local \\\
  --auth-choice tokenhub-api-key \\\
  --tokenhub-api-key \"$TOKENHUB_API_KEY\" \\\
  --skip-health \\\
  --accept-risk
```

## [​](https://docs.openclaw.ai/providers/tencent\\#built-in-catalog)  Built-in catalog

| Model ref | Name | Input | Context | Max output | Notes |
| --- | --- | --- | --- | --- | --- |
| `tencent-tokenhub/hy3-preview` | Hy3 preview (TokenHub) | text | 256,000 | 64,000 | Default; reasoning-enabled |

Hy3 preview is Tencent Hunyuan’s large MoE language model for reasoning, long-context instruction following, code, and agent workflows. Tencent’s OpenAI-compatible examples use `hy3-preview` as the model id and support standard chat-completions tool calling plus `reasoning_effort`.

The model id is `hy3-preview`. Do not confuse it with Tencent’s `HY-3D-*` models, which are 3D generation APIs and are not the OpenClaw chat model configured by this provider.

## [​](https://docs.openclaw.ai/providers/tencent\\#endpoint-override)  Endpoint override

OpenClaw defaults to Tencent Cloud’s `https://tokenhub.tencentmaas.com/v1` endpoint. Tencent also documents an international TokenHub endpoint:

```
openclaw config set models.providers.tencent-tokenhub.baseUrl \"https://tokenhub-intl.tencentmaas.com/v1\"
```

Only override the endpoint when your TokenHub account or region requires it.

## [​](https://docs.openclaw.ai/providers/tencent\\#notes)  Notes

- TokenHub model refs use `tencent-tokenhub/<modelId>`.
- The bundled catalog currently includes `hy3-preview`.
- The plugin marks Hy3 preview as reasoning-capable and streaming-usage capable.
- The plugin ships with tiered Hy3 pricing metadata, so cost estimates are populated without manual pricing overrides.
- Override pricing, context, or endpoint metadata in `models.providers` only when needed.

## [​](https://docs.openclaw.ai/providers/tencent\\#environment-note)  Environment note

If the Gateway runs as a daemon (launchd/systemd), make sure `TOKENHUB_API_KEY`
is available to that process (for example, in `~/.openclaw/.env` or via
`env.shellEnv`).

## [​](https://docs.openclaw.ai/providers/tencent\\#related-documentation)  Related documentation

- [OpenClaw Configuration](https://docs.openclaw.ai/gateway/configuration)
- [Model Providers](https://docs.openclaw.ai/concepts/model-providers)
- [Tencent TokenHub product page](https://cloud.tencent.com/product/tokenhub)
- [Tencent TokenHub text generation](https://cloud.tencent.com/document/product/1823/130079)
- [Tencent TokenHub Cline setup for Hy3 preview](https://cloud.tencent.com/document/product/1823/130932)
- [Tencent Hy3 preview model card](https://huggingface.co/tencent/Hy3-preview)

[Synthetic](https://docs.openclaw.ai/providers/synthetic) [Together AI](https://docs.openclaw.ai/providers/together)

Ctrl+I

---

## Vydra - OpenClaw
**Source:** https://docs.openclaw.ai/providers/vydra

[Skip to main content](https://docs.openclaw.ai/providers/vydra#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Vydra

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Setup](https://docs.openclaw.ai/providers/vydra#setup)
- [Capabilities](https://docs.openclaw.ai/providers/vydra#capabilities)
- [Related](https://docs.openclaw.ai/providers/vydra#related)

The bundled Vydra plugin adds:

- Image generation via `vydra/grok-imagine`
- Video generation via `vydra/veo3` and `vydra/kling`
- Speech synthesis via Vydra’s ElevenLabs-backed TTS route

OpenClaw uses the same `VYDRA_API_KEY` for all three capabilities.

Use `https://www.vydra.ai/api/v1` as the base URL.Vydra’s apex host (`https://vydra.ai/api/v1`) currently redirects to `www`. Some HTTP clients drop `Authorization` on that cross-host redirect, which turns a valid API key into a misleading auth failure. The bundled plugin uses the `www` base URL directly to avoid that.

## [​](https://docs.openclaw.ai/providers/vydra\\#setup)  Setup

1

[Navigate to header](https://docs.openclaw.ai/providers/vydra#)

Run interactive onboarding

```
openclaw onboard --auth-choice vydra-api-key
```

Or set the env var directly:

```
export VYDRA_API_KEY=\"vydra_live_...\"
```

2

[Navigate to header](https://docs.openclaw.ai/providers/vydra#)

Choose a default capability

Pick one or more of the capabilities below (image, video, or speech) and apply the matching configuration.

## [​](https://docs.openclaw.ai/providers/vydra\\#capabilities)  Capabilities

Image generation

Default image model:

- `vydra/grok-imagine`

Set it as the default image provider:

```
{
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"vydra/grok-imagine\",
      },
    },
  },
}
```

Current bundled support is text-to-image only. Vydra’s hosted edit routes expect remote image URLs, and OpenClaw does not add a Vydra-specific upload bridge in the bundled plugin yet.

See [Image Generation](https://docs.openclaw.ai/tools/image-generation) for shared tool parameters, provider selection, and failover behavior.

Video generation

Registered video models:

- `vydra/veo3` for text-to-video
- `vydra/kling` for image-to-video

Set Vydra as the default video provider:

```
{
  agents: {
    defaults: {
      videoGenerationModel: {
        primary: \"vydra/veo3\",
      },
    },
  },
}
```

Notes:

- `vydra/veo3` is bundled as text-to-video only.
- `vydra/kling` currently requires a remote image URL reference. Local file uploads are rejected up front.
- Vydra’s current `kling` HTTP route has been inconsistent about whether it requires `image_url` or `video_url`; the bundled provider maps the same remote image URL into both fields.
- The bundled plugin stays conservative and does not forward undocumented style knobs such as aspect ratio, resolution, watermark, or generated audio.

See [Video Generation](https://docs.openclaw.ai/tools/video-generation) for shared tool parameters, provider selection, and failover behavior.

Video live tests

Provider-specific live coverage:

```
OPENCLAW_LIVE_TEST=1 \\\
OPENCLAW_LIVE_VYDRA_VIDEO=1 \\\
pnpm test:live -- extensions/vydra/vydra.live.test.ts
```

The bundled Vydra live file now covers:

- `vydra/veo3` text-to-video
- `vydra/kling` image-to-video using a remote image URL

Override the remote image fixture when needed:

```
export OPENCLAW_LIVE_VYDRA_KLING_IMAGE_URL=\"https://example.com/reference.png\"
```

Speech synthesis

Set Vydra as the speech provider:

```
{
  messages: {
    tts: {
      provider: \"vydra\",
      providers: {
        vydra: {
          apiKey: \"${VYDRA_API_KEY}\",
          voiceId: \"21m00Tcm4TlvDq8ikWAM\",
        },
      },
    },
  },
}
```

Defaults:

- Model: `elevenlabs/tts`
- Voice id: `21m00Tcm4TlvDq8ikWAM`

The bundled plugin currently exposes one known-good default voice and returns MP3 audio files.

## [​](https://docs.openclaw.ai/providers/vydra\\#related)  Related

[**Provider directory** \\\\\n\\\\\nBrowse all available providers.](https://docs.openclaw.ai/providers/index)

[**Image generation** \\\\\n\\\\\nShared image tool parameters and provider selection.](https://docs.openclaw.ai/tools/image-generation)

[**Video generation** \\\\\n\\\\\nShared video tool parameters and provider selection.](https://docs.openclaw.ai/tools/video-generation)

[**Configuration reference** \\\\\n\\\\\nAgent defaults and model configuration.](https://docs.openclaw.ai/gateway/config-agents#agent-defaults)

[Volcengine (Doubao)](https://docs.openclaw.ai/providers/volcengine) [xAI](https://docs.openclaw.ai/providers/xai)

Ctrl+I

---

## Cloudflare AI gateway - OpenClaw
**Source:** https://docs.openclaw.ai/providers/cloudflare-ai-gateway

[Skip to main content](https://docs.openclaw.ai/providers/cloudflare-ai-gateway#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

⌘K

Search...

Navigation

Providers

Cloudflare AI gateway

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/cloudflare-ai-gateway#getting-started)
- [Non-interactive example](https://docs.openclaw.ai/providers/cloudflare-ai-gateway#non-interactive-example)
- [Advanced configuration](https://docs.openclaw.ai/providers/cloudflare-ai-gateway#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/cloudflare-ai-gateway#related)

Cloudflare AI Gateway sits in front of provider APIs and lets you add analytics, caching, and controls. For Anthropic, OpenClaw uses the Anthropic Messages API through your Gateway endpoint.

| Property | Value |
| --- | --- |
| Provider | `cloudflare-ai-gateway` |
| Base URL | `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic` |
| Default model | `cloudflare-ai-gateway/claude-sonnet-4-6` |
| API key | `CLOUDFLARE_AI_GATEWAY_API_KEY` (your provider API key for requests through the Gateway) |

For Anthropic models routed through Cloudflare AI Gateway, use your **Anthropic API key** as the provider key.

When thinking is enabled for Anthropic Messages models, OpenClaw strips trailing
assistant prefill turns before sending the payload through Cloudflare AI Gateway.
Anthropic rejects response prefilling with extended thinking, while ordinary
non-thinking prefill remains available.

## [​](https://docs.openclaw.ai/providers/cloudflare-ai-gateway\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/cloudflare-ai-gateway#)

Set the provider API key and Gateway details

Run onboarding and choose the Cloudflare AI Gateway auth option:

```
openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
```

This prompts for your account ID, gateway ID, and API key.

2

[Navigate to header](https://docs.openclaw.ai/providers/cloudflare-ai-gateway#)

Set a default model

Add the model to your OpenClaw config:

```
{
  agents: {
    defaults: {
      model: { primary: \"cloudflare-ai-gateway/claude-sonnet-4-6\" },
    },
  },
}
```

3

[Navigate to header](https://docs.openclaw.ai/providers/cloudflare-ai-gateway#)

Verify the model is available

```
openclaw models list --provider cloudflare-ai-gateway
```

## [​](https://docs.openclaw.ai/providers/cloudflare-ai-gateway\\#non-interactive-example)  Non-interactive example

For scripted or CI setups, pass all values on the command line:

```
openclaw onboard --non-interactive \\\
  --mode local \\\
  --auth-choice cloudflare-ai-gateway-api-key \\\
  --cloudflare-ai-gateway-account-id \"your-account-id\" \\\
  --cloudflare-ai-gateway-gateway-id \"your-gateway-id\" \\\
  --cloudflare-ai-gateway-api-key \"$CLOUDFLARE_AI_GATEWAY_API_KEY\"
```

## [​](https://docs.openclaw.ai/providers/cloudflare-ai-gateway\\#advanced-configuration)  Advanced configuration

Authenticated gateways

If you enabled Gateway authentication in Cloudflare, add the `cf-aig-authorization` header. This is **in addition to** your provider API key.

```
{
  models: {
    providers: {
      \"cloudflare-ai-gateway\": {
        headers: {
          \"cf-aig-authorization\": \"Bearer <cloudflare-ai-gateway-token>\",
        },
      },
    },
  },
}
```

The `cf-aig-authorization` header authenticates with the Cloudflare Gateway itself, while the provider API key (for example, your Anthropic key) authenticates with the upstream provider.

Environment note

If the Gateway runs as a daemon (launchd/systemd), make sure `CLOUDFLARE_AI_GATEWAY_API_KEY` is available to that process.

A key sitting only in `~/.profile` will not help a launchd/systemd daemon unless that environment is imported there as well. Set the key in `~/.openclaw/.env` or via `env.shellEnv` to ensure the gateway process can read it.

## [​](https://docs.openclaw.ai/providers/cloudflare-ai-gateway\\#related)  Related

## Model selection

Choosing providers, model refs, and failover behavior.

## Troubleshooting

General troubleshooting and FAQ.

[Claude Max API proxy](https://docs.openclaw.ai/providers/claude-max-api-proxy) [ComfyUI](https://docs.openclaw.ai/providers/comfy)

⌘I

---

## ComfyUI - OpenClaw
**Source:** https://docs.openclaw.ai/providers/comfy

[Skip to main content](https://docs.openclaw.ai/providers/comfy#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

ComfyUI

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [What it supports](https://docs.openclaw.ai/providers/comfy#what-it-supports)
- [Getting started](https://docs.openclaw.ai/providers/comfy#getting-started)
- [Configuration](https://docs.openclaw.ai/providers/comfy#configuration)
- [Shared keys](https://docs.openclaw.ai/providers/comfy#shared-keys)
- [Per-capability keys](https://docs.openclaw.ai/providers/comfy#per-capability-keys)
- [Workflow details](https://docs.openclaw.ai/providers/comfy#workflow-details)
- [Related](https://docs.openclaw.ai/providers/comfy#related)

OpenClaw ships a bundled `comfy` plugin for workflow-driven ComfyUI runs. The plugin is entirely workflow-driven, so OpenClaw does not try to map generic `size`, `aspectRatio`, `resolution`, `durationSeconds`, or TTS-style controls onto your graph.

| Property | Detail |
| --- | --- |
| Provider | `comfy` |
| Models | `comfy/workflow` |
| Shared surfaces | `image_generate`, `video_generate`, `music_generate` |
| Auth | None for local ComfyUI; `COMFY_API_KEY` or `COMFY_CLOUD_API_KEY` for Comfy Cloud |
| API | ComfyUI `/prompt` / `/history` / `/view` and Comfy Cloud `/api/*` |

## [​](https://docs.openclaw.ai/providers/comfy\\#what-it-supports)  What it supports

- Image generation from a workflow JSON
- Image editing with 1 uploaded reference image
- Video generation from a workflow JSON
- Video generation with 1 uploaded reference image
- Music or audio generation through the shared `music_generate` tool
- Output download from a configured node or all matching output nodes

## [​](https://docs.openclaw.ai/providers/comfy\\#getting-started)  Getting started

Choose between running ComfyUI on your own machine or using Comfy Cloud.

- Local

- Comfy Cloud


**Best for:** running your own ComfyUI instance on your machine or LAN.

1

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Start ComfyUI locally

Make sure your local ComfyUI instance is running (defaults to `http://127.0.0.1:8188`).

2

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Prepare your workflow JSON

Export or create a ComfyUI workflow JSON file. Note the node IDs for the prompt input node and the output node you want OpenClaw to read from.

3

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Configure the provider

Set `mode: \"local\"` and point at your workflow file. Here is a minimal image example:

```
{
  plugins: {
    entries: {
      comfy: {
        config: {
          mode: \"local\",
          baseUrl: \"http://127.0.0.1:8188\",
          image: {
            workflowPath: \"./workflows/flux-api.json\",
            promptNodeId: \"6\",
            outputNodeId: \"9\",
          },
        },
      },
    },
  },
}
```

4

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Set the default model

Point OpenClaw at the `comfy/workflow` model for the capability you configured:

```
{
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"comfy/workflow\",
      },
    },
  },
}
```

5

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Verify

```
openclaw models list --provider comfy
```

**Best for:** running workflows on Comfy Cloud without managing local GPU resources.

1

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Get an API key

Sign up at [comfy.org](https://comfy.org/) and generate an API key from your account dashboard.

2

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Set the API key

Provide your key through one of these methods:

```
# Environment variable (preferred)
export COMFY_API_KEY=\"your-key\"

# Alternative environment variable
export COMFY_CLOUD_API_KEY=\"your-key\"

# Or inline in config
openclaw config set plugins.entries.comfy.config.apiKey \"your-key\"
```

3

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Prepare your workflow JSON

Export or create a ComfyUI workflow JSON file. Note the node IDs for the prompt input node and the output node.

4

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Configure the provider

Set `mode: \"cloud\"` and point at your workflow file:

```
{
  plugins: {
    entries: {
      comfy: {
        config: {
          mode: \"cloud\",
          image: {
            workflowPath: \"./workflows/flux-api.json\",
            promptNodeId: \"6\",
            outputNodeId: \"9\",
          },
        },
      },
    },
  },
}
```

Cloud mode defaults `baseUrl` to `https://cloud.comfy.org`. You only need to set `baseUrl` if you use a custom cloud endpoint.

5

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Set the default model

```
{
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"comfy/workflow\",
      },
    },
  },
}
```

6

[Navigate to header](https://docs.openclaw.ai/providers/comfy#)

Verify

```
openclaw models list --provider comfy
```

## [​](https://docs.openclaw.ai/providers/comfy\\#configuration)  Configuration

Comfy supports shared top-level connection settings plus per-capability workflow sections (`image`, `video`, `music`):

```
{
  plugins: {
    entries: {
      comfy: {
        config: {
          mode: \"local\",
          baseUrl: \"http://127.0.0.1:8188\",
          image: {
            workflowPath: \"./workflows/flux-api.json\",
            promptNodeId: \"6\",
            outputNodeId: \"9\",
          },
          video: {
            workflowPath: \"./workflows/video-api.json\",
            promptNodeId: \"12\",
            outputNodeId: \"21\",
          },
          music: {
            workflowPath: \"./workflows/music-api.json\",
            promptNodeId: \"3\",
            outputNodeId: \"18\",
          },
        },
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/providers/comfy\\#shared-keys)  Shared keys

| Key | Type | Description |
| --- | --- | --- |
| `mode` | `\"local\"` or `\"cloud\"` | Connection mode. |
| `baseUrl` | string | Defaults to `http://127.0.0.1:8188` for local or `https://cloud.comfy.org` for cloud. |
| `apiKey` | string | Optional inline key, alternative to `COMFY_API_KEY` / `COMFY_CLOUD_API_KEY` env vars. |
| `allowPrivateNetwork` | boolean | Allow a private/LAN `baseUrl` in cloud mode. |

### [​](https://docs.openclaw.ai/providers/comfy\\#per-capability-keys)  Per-capability keys

These keys apply inside the `image`, `video`, or `music` sections:

| Key | Required | Default | Description |
| --- | --- | --- | --- |
| `workflow` or `workflowPath` | Yes | — | Path to the ComfyUI workflow JSON file. |
| `promptNodeId` | Yes | — | Node ID that receives the text prompt. |
| `promptInputName` | No | `\"text\"` | Input name on the prompt node. |
| `outputNodeId` | No | — | Node ID to read output from. If omitted, all matching output nodes are used. |
| `pollIntervalMs` | No | — | Polling interval in milliseconds for job completion. |
| `timeoutMs` | No | — | Timeout in milliseconds for the workflow run. |

The `image` and `video` sections also support:

| Key | Required | Default | Description |
| --- | --- | --- | --- |
| `inputImageNodeId` | Yes (when passing a reference image) | — | Node ID that receives the uploaded reference image. |
| `inputImageInputName` | No | `\"image\"` | Input name on the image node. |

## [​](https://docs.openclaw.ai/providers/comfy\\#workflow-details)  Workflow details

Image workflows

Set the default image model to `comfy/workflow`:

```
{
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"comfy/workflow\",
      },
    },
  },
}
```

**Reference-image editing example:**To enable image editing with an uploaded reference image, add `inputImageNodeId` to your image config:

```
{
  plugins: {
    entries: {
      comfy: {
        config: {
          image: {
            workflowPath: \"./workflows/edit-api.json\",
            promptNodeId: \"6\",
            inputImageNodeId: \"7\",
            inputImageInputName: \"image\",
            outputNodeId: \"9\",
          },
        },
      },
    },
  },
}
```

Video workflows

Set the default video model to `comfy/workflow`:

```
{
  agents: {
    defaults: {
      videoGenerationModel: {
        primary: \"comfy/workflow\",
      },
    },
  },
}
```

Comfy video workflows support text-to-video and image-to-video through the configured graph.

OpenClaw does not pass input videos into Comfy workflows. Only text prompts and single reference images are supported as inputs.

Music workflows

The bundled plugin registers a music-generation provider for workflow-defined audio or music outputs, surfaced through the shared `music_generate` tool:

```
/tool music_generate prompt=\"Warm ambient synth loop with soft tape texture\"
```

Use the `music` config section to point at your audio workflow JSON and output node.

Backward compatibility

Existing top-level image config (without the nested `image` section) still works:

```
{
  plugins: {
    entries: {
      comfy: {
        config: {
          workflowPath: \"./workflows/flux-api.json\",
          promptNodeId: \"6\",
          outputNodeId: \"9\",
        },
      },
    },
  },
}
```

OpenClaw treats that legacy shape as the image workflow config. You do not need to migrate immediately, but the nested `image` / `video` / `music` sections are recommended for new setups.

If you only use image generation, the legacy flat config and the new nested `image` section are functionally equivalent.

Live tests

Opt-in live coverage exists for the bundled plugin:

```
OPENCLAW_LIVE_TEST=1 COMFY_LIVE_TEST=1 pnpm test:live -- extensions/comfy/comfy.live.test.ts
```

The live test skips individual image, video, or music cases unless the matching Comfy workflow section is configured.

## [​](https://docs.openclaw.ai/providers/comfy\\#related)  Related

[**Image Generation** \\\\\n\\\\\nImage generation tool configuration and usage.](https://docs.openclaw.ai/tools/image-generation)

[**Video Generation** \\\\\n\\\\\nVideo generation tool configuration and usage.](https://docs.openclaw.ai/tools/video-generation)

[**Music Generation** \\\\\n\\\\\nMusic and audio generation tool setup.](https://docs.openclaw.ai/tools/music-generation)

[**Provider Directory** \\\\\n\\\\\nOverview of all providers and model refs.](https://docs.openclaw.ai/providers/index)

[**Configuration reference** \\\\\n\\\\\nFull config reference including agent defaults.](https://docs.openclaw.ai/gateway/config-agents#agent-defaults)

[Cloudflare AI gateway](https://docs.openclaw.ai/providers/cloudflare-ai-gateway) [Deepgram](https://docs.openclaw.ai/providers/deepgram)

Ctrl+I

---

## Mistral - OpenClaw
**Source:** https://docs.openclaw.ai/providers/mistral

[Skip to main content](https://docs.openclaw.ai/providers/mistral#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Mistral

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/mistral#getting-started)
- [Built-in LLM catalog](https://docs.openclaw.ai/providers/mistral#built-in-llm-catalog)
- [Audio transcription (Voxtral)](https://docs.openclaw.ai/providers/mistral#audio-transcription-voxtral)
- [Voice Call streaming STT](https://docs.openclaw.ai/providers/mistral#voice-call-streaming-stt)
- [Advanced configuration](https://docs.openclaw.ai/providers/mistral#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/mistral#related)

OpenClaw supports Mistral for both text/image model routing (`mistral/...`) and
audio transcription via Voxtral in media understanding.
Mistral can also be used for memory embeddings (`memorySearch.provider = \"mistral\"`).

- Provider: `mistral`
- Auth: `MISTRAL_API_KEY`
- API: Mistral Chat Completions (`https://api.mistral.ai/v1`)

## [​](https://docs.openclaw.ai/providers/mistral\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/mistral#)

Get your API key

Create an API key in the [Mistral Console](https://console.mistral.ai/).

2

[Navigate to header](https://docs.openclaw.ai/providers/mistral#)

Run onboarding

```
openclaw onboard --auth-choice mistral-api-key
```

Or pass the key directly:

```
openclaw onboard --mistral-api-key \"$MISTRAL_API_KEY\"
```

3

[Navigate to header](https://docs.openclaw.ai/providers/mistral#)

Set a default model

```
{
  env: { MISTRAL_API_KEY: \"sk-...\" },
  agents: { defaults: { model: { primary: \"mistral/mistral-large-latest\" } } },
}
```

4

[Navigate to header](https://docs.openclaw.ai/providers/mistral#)

Verify the model is available

```
openclaw models list --provider mistral
```

## [​](https://docs.openclaw.ai/providers/mistral\\#built-in-llm-catalog)  Built-in LLM catalog

OpenClaw currently ships this bundled Mistral catalog:

| Model ref | Input | Context | Max output | Notes |
| --- | --- | --- | --- |
| `mistral/mistral-large-latest` | text, image | 262,144 | 16,384 | Default model |
| `mistral/mistral-medium-2508` | text, image | 262,144 | 8,192 | Mistral Medium 3.1 |
| `mistral/mistral-small-latest` | text, image | 128,000 | 16,384 | Mistral Small 4; adjustable reasoning via API `reasoning_effort` |
| `mistral/pixtral-large-latest` | text, image | 128,000 | 32,768 | Pixtral |
| `mistral/codestral-latest` | text | 256,000 | 4,096 | Coding |
| `mistral/devstral-medium-latest` | text | 262,144 | 32,768 | Devstral 2 |
| `mistral/magistral-small` | text | 128,000 | 40,000 | Reasoning-enabled |

## [​](https://docs.openclaw.ai/providers/mistral\\#audio-transcription-voxtral)  Audio transcription (Voxtral)

Use Voxtral for batch audio transcription through the media understanding
pipeline.

```
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: \"mistral\", model: \"voxtral-mini-latest\" }],
      },
    },
  },
}
```

The media transcription path uses `/v1/audio/transcriptions`. The default audio model for Mistral is `voxtral-mini-latest`.

## [​](https://docs.openclaw.ai/providers/mistral\\#voice-call-streaming-stt)  Voice Call streaming STT

The bundled `mistral` plugin registers Voxtral Realtime as a Voice Call
streaming STT provider.

| Setting | Config path | Default |
| --- | --- | --- |
| API key | `plugins.entries.voice-call.config.streaming.providers.mistral.apiKey` | Falls back to `MISTRAL_API_KEY` |
| Model | `...mistral.model` | `voxtral-mini-transcribe-realtime-2602` |
| Encoding | `...mistral.encoding` | `pcm_mulaw` |
| Sample rate | `...mistral.sampleRate` | `8000` |
| Target delay | `...mistral.targetStreamingDelayMs` | `800` |

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        config: {
          streaming: {
            enabled: true,
            provider: \"mistral\",
            providers: {
              mistral: {
                apiKey: \"${MISTRAL_API_KEY}\",
                targetStreamingDelayMs: 800,
              },
            },
          },
        },
      },
    },
  },
}
```

OpenClaw defaults Mistral realtime STT to `pcm_mulaw` at 8 kHz so Voice Call
can forward Twilio media frames directly. Use `encoding: \"pcm_s16le\"` and a
matching `sampleRate` only if your upstream stream is already raw PCM.

## [​](https://docs.openclaw.ai/providers/mistral\\#advanced-configuration)  Advanced configuration

Adjustable reasoning (mistral-small-latest)

`mistral/mistral-small-latest` maps to Mistral Small 4 and supports [adjustable reasoning](https://docs.mistral.ai/capabilities/reasoning/adjustable) on the Chat Completions API via `reasoning_effort` (`none` minimizes extra thinking in the output; `high` surfaces full thinking traces before the final answer).OpenClaw maps the session **thinking** level to Mistral’s API:

| OpenClaw thinking level | Mistral `reasoning_effort` |
| --- | --- |
| **off** / **minimal** | `none` |
| **low** / **medium** / **high** / **xhigh** / **adaptive** / **max** | `high` |

Other bundled Mistral catalog models do not use this parameter. Keep using `magistral-*` models when you want Mistral’s native reasoning-first behavior.

Memory embeddings

Mistral can serve memory embeddings via `/v1/embeddings` (default model: `mistral-embed`).

```
{
  memorySearch: { provider: \"mistral\" },
}
```

Auth and base URL

- Mistral auth uses `MISTRAL_API_KEY`.
- Provider base URL defaults to `https://api.mistral.ai/v1`.
- Onboarding default model is `mistral/mistral-large-latest`.
- Z.AI uses Bearer auth with your API key.

## [​](https://docs.openclaw.ai/providers/mistral\\#related)  Related

[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Media understanding** \\\\\n\\\\\nAudio transcription setup and provider selection.](https://docs.openclaw.ai/nodes/media-understanding)

[MiniMax](https://docs.openclaw.ai/providers/minimax) [Moonshot AI](https://docs.openclaw.ai/providers/moonshot)

Ctrl+I

---

## Amazon Bedrock - OpenClaw
**Source:** https://docs.openclaw.ai/providers/bedrock

[Skip to main content](https://docs.openclaw.ai/providers/bedrock#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nProviders\n\nAmazon Bedrock\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Getting started](https://docs.openclaw.ai/providers/bedrock#getting-started)\n- [Automatic model discovery](https://docs.openclaw.ai/providers/bedrock#automatic-model-discovery)\n- [Quick setup (AWS path)](https://docs.openclaw.ai/providers/bedrock#quick-setup-aws-path)\n- [Advanced configuration](https://docs.openclaw.ai/providers/bedrock#advanced-configuration)\n- [Related](https://docs.openclaw.ai/providers/bedrock#related)\n\nOpenClaw can use **Amazon Bedrock** models via pi-ai’s **Bedrock Converse**\nstreaming provider. Bedrock auth uses the **AWS SDK default credential chain**,\nnot an API key.\n\n| Property | Value |\n| --- | --- |\n| Provider | `amazon-bedrock` |\n| API | `bedrock-converse-stream` |\n| Auth | AWS credentials (env vars, shared config, or instance role) |\n| Region | `AWS_REGION` or `AWS_DEFAULT_REGION` (default: `us-east-1`) |\n\n## [​](https://docs.openclaw.ai/providers/bedrock\\#getting-started)  Getting started\n\nChoose your preferred auth method and follow the setup steps.\n\n- Access keys / env vars\n\n- EC2 instance roles (IMDS)\n\n\n**Best for:** developer machines, CI, or hosts where you manage AWS credentials directly.\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/providers/bedrock#)\n\nSet AWS credentials on the gateway host\n\n```\nexport AWS_ACCESS_KEY_ID=\"AKIA...\"\nexport AWS_SECRET_ACCESS_KEY=\"...\"\nexport AWS_REGION=\"us-east-1\"\n# Optional:\nexport AWS_SESSION_TOKEN=\"...\"\nexport AWS_PROFILE=\"your-profile\"\n# Optional (Bedrock API key/bearer token):\nexport AWS_BEARER_TOKEN_BEDROCK=\"...\"\n```\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/providers/bedrock#)\n\nAdd a Bedrock provider and model to your config\n\nNo `apiKey` is required. Configure the provider with `auth: \"aws-sdk\"`:\n\n```\n{\n  models: {\n    providers: {\n      \"amazon-bedrock\": {\n        baseUrl: \"https://bedrock-runtime.us-east-1.amazonaws.com\",\n        api: \"bedrock-converse-stream\",\n        auth: \"aws-sdk\",\n        models: [\\\n          {\\\n            id: \"us.anthropic.claude-opus-4-6-v1:0\",\\\n            name: \"Claude Opus 4.6 (Bedrock)\",\\\n            reasoning: true,\\\n            input: [\"text\", \"image\"],\\\n            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\n            contextWindow: 200000,\\\n            maxTokens: 8192,\\\n          },\\\n        ],\n      },\n    },\n  },\n  agents: {\n    defaults: {\n      model: { primary: \"amazon-bedrock/us.anthropic.claude-opus-4-6-v1:0\" },\n    },\n  },\n}\n```\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/providers/bedrock#)\n\nVerify models are available\n\n```\nopenclaw models list\n```\n\nWith env-marker auth (`AWS_ACCESS_KEY_ID`, `AWS_PROFILE`, or `AWS_BEARER_TOKEN_BEDROCK`), OpenClaw auto-enables the implicit Bedrock provider for model discovery without extra config.\n\n**Best for:** EC2 instances with an IAM role attached, using the instance metadata service for authentication.\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/providers/bedrock#)\n\nEnable discovery explicitly\n\nWhen using IMDS, OpenClaw cannot detect AWS auth from env markers alone, so you must opt in:\n\n```\nopenclaw config set plugins.entries.amazon-bedrock.config.discovery.enabled true\nopenclaw config set plugins.entries.amazon-bedrock.config.discovery.region us-east-1\n```\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/providers/bedrock#)\n\nOptionally add an env marker for auto mode\n\nIf you also want the env-marker auto-detection path to work (for example, for `openclaw status` surfaces):\n\n```\nexport AWS_PROFILE=default\nexport AWS_REGION=us-east-1\n```\n\nYou do **not** need a fake API key.\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/providers/bedrock#)\n\nVerify models are discovered\n\n```\nopenclaw models list\n```\n\nThe IAM role attached to your EC2 instance must have the following permissions:\n\n- `bedrock:InvokeModel`\n- `bedrock:InvokeModelWithResponseStream`\n- `bedrock:ListFoundationModels` (for automatic discovery)\n- `bedrock:ListInferenceProfiles` (for inference profile discovery)\n\nOr attach the managed policy `AmazonBedrockFullAccess`.\n\nYou only need `AWS_PROFILE=default` if you specifically want an env marker for auto mode or status surfaces. The actual Bedrock runtime auth path uses the AWS SDK default chain, so IMDS instance-role auth works even without env markers.\n\n## [​](https://docs.openclaw.ai/providers/bedrock\\#automatic-model-discovery)  Automatic model discovery\n\nOpenClaw can automatically discover Bedrock models that support **streaming**\nand **text output**. Discovery uses `bedrock:ListFoundationModels` and\n`bedrock:ListInferenceProfiles`, and results are cached (default: 1 hour).How the implicit provider is enabled:\n\n- If `plugins.entries.amazon-bedrock.config.discovery.enabled` is `true`,\nOpenClaw will try discovery even when no AWS env marker is present.\n- If `plugins.entries.amazon-bedrock.config.discovery.enabled` is unset,\nOpenClaw only auto-adds the\nimplicit Bedrock provider when it sees one of these AWS auth markers:\n`AWS_BEARER_TOKEN_BEDROCK`, `AWS_ACCESS_KEY_ID` +\n`AWS_SECRET_ACCESS_KEY`, or `AWS_PROFILE`.\n- The actual Bedrock runtime auth path still uses the AWS SDK default chain, so\nshared config, SSO, and IMDS instance-role auth can work even when discovery\nneeded `enabled: true` to opt in.\n\nFor explicit `models.providers[\"amazon-bedrock\"]` entries, OpenClaw can still resolve Bedrock env-marker auth early from AWS env markers such as `AWS_BEARER_TOKEN_BEDROCK` without forcing full runtime auth loading. The actual model-call auth path still uses the AWS SDK default chain.\n\nDiscovery config options\n\nConfig options live under `plugins.entries.amazon-bedrock.config.discovery`:\n\n```\n{\n  plugins: {\n    entries: {\n      \"amazon-bedrock\": {\n        config: {\n          discovery: {\n            enabled: true,\n            region: \"us-east-1\",\n            providerFilter: [\"anthropic\", \"amazon\"],\n            refreshInterval: 3600,\n            defaultContextWindow: 32000,\n            defaultMaxTokens: 4096,\n          },\n        },\n      },\n    },\n  },\n}\n```\n\n| Option | Default | Description |\n| --- | --- | --- |\n| `enabled` | auto | In auto mode, OpenClaw only enables the implicit Bedrock provider when it sees a supported AWS env marker. Set `true` to force discovery. |\n| `region` | `AWS_REGION` / `AWS_DEFAULT_REGION` / `us-east-1` | AWS region used for discovery API calls. |\n| `providerFilter` | (all) | Matches Bedrock provider names (for example `anthropic`, `amazon`). |\n| `refreshInterval` | `3600` | Cache duration in seconds. Set to `0` to disable caching. |\n| `defaultContextWindow` | `32000` | Context window used for discovered models (override if you know your model limits). |\n| `defaultMaxTokens` | `4096` | Max output tokens used for discovered models (override if you know your model limits). |\n\n## [​](https://docs.openclaw.ai/providers/bedrock\\#quick-setup-aws-path)  Quick setup (AWS path)\n\nThis walkthrough creates an IAM role, attaches Bedrock permissions, associates\nthe instance profile, and enables OpenClaw discovery on the EC2 host.\n\n```\n# 1. Create IAM role and instance profile\naws iam create-role --role-name EC2-Bedrock-Access \\\n  --assume-role-policy-document \'{\n    \"Version\": \"2012-10-17\",\n    \"Statement\": [{\\\n      \"Effect\": \"Allow\",\\\n      \"Principal\": {\"Service\": \"ec2.amazonaws.com\"},\\\n      \"Action\": \"sts:AssumeRole\"\\\n    }]\n  }\'\n\naws iam attach-role-policy --role-name EC2-Bedrock-Access \\\n  --policy-arn arn:aws:iam::aws:policy/AmazonBedrockFullAccess\n\naws iam create-instance-profile --instance-profile-name EC2-Bedrock-Access\naws iam add-role-to-instance-profile \\\n  --instance-profile-name EC2-Bedrock-Access \\\n  --role-name EC2-Bedrock-Access\n\n# 2. Attach to your EC2 instance\naws ec2 associate-iam-instance-profile \\\n  --instance-id i-xxxxx \\\n  --iam-instance-profile Name=EC2-Bedrock-Access\n\n# 3. On the EC2 instance, enable discovery explicitly\nopenclaw config set plugins.entries.amazon-bedrock.config.discovery.enabled true\nopenclaw config set plugins.entries.amazon-bedrock.config.discovery.region us-east-1\n\n# 4. Optional: add an env marker if you want auto mode without explicit enable\necho \'export AWS_PROFILE=default\' >> ~/.bashrc\necho \'export AWS_REGION=us-east-1\' >> ~/.bashrc\nsource ~/.bashrc\n\n# 5. Verify models are discovered\nopenclaw models list\n```\n\n## [​](https://docs.openclaw.ai/providers/bedrock\\#advanced-configuration)  Advanced configuration\n\nInference profiles\n\nOpenClaw discovers **regional and global inference profiles** alongside\nfoundation models. When a profile maps to a known foundation model, the\nprofile inherits that model’s capabilities (context window, max tokens,\nreasoning, vision) and the correct Bedrock request region is injected\nautomatically. This means cross-region Claude profiles work without manual\nprovider overrides.Inference profile IDs look like `us.anthropic.claude-opus-4-6-v1:0` (regional)\nor `anthropic.claude-opus-4-6-v1:0` (global). If the backing model is already\nin the discovery results, the profile inherits its full capability set;\notherwise safe defaults apply.No extra configuration is needed. As long as discovery is enabled and the IAM\nprincipal has `bedrock:ListInferenceProfiles`, profiles appear alongside\nfoundation models in `openclaw models list`.\n\nGuardrails\n\nYou can apply [Amazon Bedrock Guardrails](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html)\nto all Bedrock model invocations by adding a `guardrail` object to the\n`amazon-bedrock` plugin config. Guardrails let you enforce content filtering,\ntopic denial, word filters, sensitive information filters, and contextual\ngrounding checks.\n\n```\n{\n  plugins: {\n    entries: {\n      \"amazon-bedrock\": {\n        config: {\n          guardrail: {\n            guardrailIdentifier: \"abc123\", // guardrail ID or full ARN\n            guardrailVersion: \"1\", // version number or \"DRAFT\"\n            streamProcessingMode: \"sync\", // optional: \"sync\" or \"async\"\n            trace: \"enabled\", // optional: \"enabled\", \"disabled\", or \"enabled_full\"\n          },\n        },\n      },\n    },\n  },\n}\n```\n\n| Option | Required | Description |\n| --- | --- | --- |\n| `guardrailIdentifier` | Yes | Guardrail ID (e.g. `abc123`) or full ARN (e.g. `arn:aws:bedrock:us-east-1:123456789012:guardrail/abc123`). |\n| `guardrailVersion` | Yes | Published version number, or `\"DRAFT\"` for the working draft. |\n| `streamProcessingMode` | No | `\"sync\"` or `\"async\"` for guardrail evaluation during streaming. If omitted, Bedrock uses its default. |\n| `trace` | No | `\"enabled\"` or `\"enabled_full\"` for debugging; omit or set `\"disabled\"` for production. |\n\nThe IAM principal used by the gateway must have the `bedrock:ApplyGuardrail` permission in addition to the standard invoke permissions.\n\nEmbeddings for memory search\n\nBedrock can also serve as the embedding provider for\n[memory search](https://docs.openclaw.ai/concepts/memory-search). This is configured separately from the\ninference provider — set `agents.defaults.memorySearch.provider` to `\"bedrock\"`:\n\n```\n{\n  agents: {\n    defaults: {\n      memorySearch: {\n        provider: \"bedrock\",\n        model: \"amazon.titan-embed-text-v2:0\", // default\n      },\n    },\n  },\n}\n```\n\nBedrock embeddings use the same AWS SDK credential chain as inference (instance\nroles, SSO, access keys, shared config, and web identity). No API key is\nneeded. When `provider` is `\"auto\"`, Bedrock is auto-detected if that\ncredential chain resolves successfully.Supported embedding models include Amazon Titan Embed (v1, v2), Amazon Nova\nEmbed, Cohere Embed (v3, v4), and TwelveLabs Marengo. See\n[Memory configuration reference — Bedrock](https://docs.openclaw.ai/reference/memory-config#bedrock-embedding-config)\nfor the full model list and dimension options.\n\nNotes and caveats\n\n- Bedrock requires **model access** enabled in your AWS account/region.\n- Automatic discovery needs the `bedrock:ListFoundationModels` and\n`bedrock:ListInferenceProfiles` permissions.\n- If you rely on auto mode, set one of the supported AWS auth env markers on the\ngateway host. If you prefer IMDS/shared-config auth without env markers, set\n`plugins.entries.amazon-bedrock.config.discovery.enabled: true`.\n- OpenClaw surfaces the credential source in this order: `AWS_BEARER_TOKEN_BEDROCK`,\nthen `AWS_ACCESS_KEY_ID` \\+ `AWS_SECRET_ACCESS_KEY`, then `AWS_PROFILE`, then the\ndefault AWS SDK chain.\n- Reasoning support depends on the model; check the Bedrock model card for\ncurrent capabilities.\n- If you prefer a managed key flow, you can also place an OpenAI-compatible\nproxy in front of Bedrock and configure it as an OpenAI provider instead.\n\n## [​](https://docs.openclaw.ai/providers/bedrock\\#related)  Related\n\n[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)\n\n[**Memory search** \\\\\n\\\\\nBedrock embeddings for memory search configuration.](https://docs.openclaw.ai/concepts/memory-search)\n\n[**Memory config reference** \\\\\n\\\\\nFull Bedrock embedding model list and dimension options.](https://docs.openclaw.ai/reference/memory-config#bedrock-embedding-config)\n\n[**Troubleshooting** \\\\\n\\\\\nGeneral troubleshooting and FAQ.](https://docs.openclaw.ai/help/troubleshooting)\n\n[Alibaba Model Studio](https://docs.openclaw.ai/providers/alibaba) [Amazon Bedrock Mantle](https://docs.openclaw.ai/providers/bedrock-mantle)\n\nCtrl+I

---

## Claude Max API proxy - OpenClaw
**Source:** https://docs.openclaw.ai/providers/claude-max-api-proxy

[Skip to main content](https://docs.openclaw.ai/providers/claude-max-api-proxy#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Claude Max API proxy

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Why use this?](https://docs.openclaw.ai/providers/claude-max-api-proxy#why-use-this)
- [How it works](https://docs.openclaw.ai/providers/claude-max-api-proxy#how-it-works)
- [Getting started](https://docs.openclaw.ai/providers/claude-max-api-proxy#getting-started)
- [Built-in catalog](https://docs.openclaw.ai/providers/claude-max-api-proxy#built-in-catalog)
- [Advanced configuration](https://docs.openclaw.ai/providers/claude-max-api-proxy#advanced-configuration)
- [Links](https://docs.openclaw.ai/providers/claude-max-api-proxy#links)
- [Notes](https://docs.openclaw.ai/providers/claude-max-api-proxy#notes)
- [Related](https://docs.openclaw.ai/providers/claude-max-api-proxy#related)

**claude-max-api-proxy** is a community tool that exposes your Claude Max/Pro subscription as an OpenAI-compatible API endpoint. This allows you to use your subscription with any tool that supports the OpenAI API format.

This path is technical compatibility only. Anthropic has blocked some subscription
usage outside Claude Code in the past. You must decide for yourself whether to use
it and verify Anthropic’s current terms before relying on it.

## [​](https://docs.openclaw.ai/providers/claude-max-api-proxy\\#why-use-this)  Why use this?

| Approach | Cost | Best For |
| --- | --- | --- |
| Anthropic API | Pay per token (~15/Minput,15/M input, 15/Minput,75/M output for Opus) | Production apps, high volume |
| Claude Max subscription | $200/month flat | Personal use, development, unlimited usage |

If you have a Claude Max subscription and want to use it with OpenAI-compatible tools, this proxy may reduce cost for some workflows. API keys remain the clearer policy path for production use.

## [​](https://docs.openclaw.ai/providers/claude-max-api-proxy\\#how-it-works)  How it works

```
Your App → claude-max-api-proxy → Claude Code CLI → Anthropic (via subscription)
     (OpenAI format)              (converts format)      (uses your login)
```

The proxy:

1. Accepts OpenAI-format requests at `http://localhost:3456/v1/chat/completions`
2. Converts them to Claude Code CLI commands
3. Returns responses in OpenAI format (streaming supported)

## [​](https://docs.openclaw.ai/providers/claude-max-api-proxy\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/claude-max-api-proxy#)

Install the proxy

Requires Node.js 20+ and Claude Code CLI.

```
npm install -g claude-max-api-proxy

# Verify Claude CLI is authenticated
claude --version
```

2

[Navigate to header](https://docs.openclaw.ai/providers/claude-max-api-proxy#)

Start the server

```
claude-max-api
# Server runs at http://localhost:3456
```

3

[Navigate to header](https://docs.openclaw.ai/providers/claude-max-api-proxy#)

Test the proxy

```
# Health check
curl http://localhost:3456/health

# List models
curl http://localhost:3456/v1/models

# Chat completion
curl http://localhost:3456/v1/chat/completions \\
  -H \"Content-Type: application/json\" \\
  -d \'{
    \"model\": \"claude-opus-4\",
    \"messages\": [{\"role\": \"user\", \"content\": \"Hello!\"}]
  }\'
```

4

[Navigate to header](https://docs.openclaw.ai/providers/claude-max-api-proxy#)

Configure OpenClaw

Point OpenClaw at the proxy as a custom OpenAI-compatible endpoint:

```
{
  env: {
    OPENAI_API_KEY: \"not-needed\",
    OPENAI_BASE_URL: \"http://localhost:3456/v1\",
  },
  agents: {
    defaults: {
      model: { primary: \"openai/claude-opus-4\" },
    },
  },
}
```

## [​](https://docs.openclaw.ai/providers/claude-max-api-proxy\\#built-in-catalog)  Built-in catalog

| Model ID | Maps To |
| --- | --- |
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

## [​](https://docs.openclaw.ai/providers/claude-max-api-proxy\\#advanced-configuration)  Advanced configuration

Proxy-style OpenAI-compatible notes

This path uses the same proxy-style OpenAI-compatible route as other custom
`/v1` backends:

- Native OpenAI-only request shaping does not apply
- No `service_tier`, no Responses `store`, no prompt-cache hints, and no
OpenAI reasoning-compat payload shaping
- Hidden OpenClaw attribution headers (`originator`, `version`, `User-Agent`)
are not injected on the proxy URL

Auto-start on macOS with LaunchAgent

Create a LaunchAgent to run the proxy automatically:

```
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << \'EOF\'
<?xml version=\"1.0\" encoding=\"UTF-8\"?>
<!DOCTYPE plist PUBLIC \"-//Apple//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/PropertyList-1.0.dtd\">
<plist version=\"1.0\">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

## [​](https://docs.openclaw.ai/providers/claude-max-api-proxy\\#links)  Links

- **npm:** [https://www.npmjs.com/package/claude-max-api-proxy](https://www.npmjs.com/package/claude-max-api-proxy)
- **GitHub:** [https://github.com/atalovesyou/claude-max-api-proxy](https://github.com/atalovesyou/claude-max-api-proxy)
- **Issues:** [https://github.com/atalovesyou/claude-max-api-proxy/issues](https://github.com/atalovesyou/claude-max-api-proxy/issues)

## [​](https://docs.openclaw.ai/providers/claude-max-api-proxy\\#notes)  Notes

- This is a **community tool**, not officially supported by Anthropic or OpenClaw
- Requires an active Claude Max/Pro subscription with Claude Code CLI authenticated
- The proxy runs locally and does not send data to any third-party servers
- Streaming responses are fully supported

For native Anthropic integration with Claude CLI or API keys, see [Anthropic provider](https://docs.openclaw.ai/providers/anthropic). For OpenAI/Codex subscriptions, see [OpenAI provider](https://docs.openclaw.ai/providers/openai).

## [​](https://docs.openclaw.ai/providers/claude-max-api-proxy\\#related)  Related

[**Anthropic provider** \\\\
\\\\\nNative OpenClaw integration with Claude CLI or API keys.](https://docs.openclaw.ai/providers/anthropic)

[**OpenAI provider** \\\\
\\\\\nFor OpenAI/Codex subscriptions.](https://docs.openclaw.ai/providers/openai)

[**Model selection** \\\\
\\\\\nOverview of all providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Configuration** \\\\
\\\\\nFull config reference.](https://docs.openclaw.ai/gateway/configuration)

[Chutes](https://docs.openclaw.ai/providers/chutes) [Cloudflare AI gateway](https://docs.openclaw.ai/providers/cloudflare-ai-gateway)

Ctrl+I

---

## Venice AI - OpenClaw
**Source:** https://docs.openclaw.ai/providers/venice

[Skip to main content](https://docs.openclaw.ai/providers/venice#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nProviders\n\nVenice AI\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Why Venice in OpenClaw](https://docs.openclaw.ai/providers/venice#why-venice-in-openclaw)\n- [Privacy modes](https://docs.openclaw.ai/providers/venice#privacy-modes)\n- [Features](https://docs.openclaw.ai/providers/venice#features)\n- [Getting started](https://docs.openclaw.ai/providers/venice#getting-started)\n- [Model selection](https://docs.openclaw.ai/providers/venice#model-selection)\n- [DeepSeek V4 replay behavior](https://docs.openclaw.ai/providers/venice#deepseek-v4-replay-behavior)\n- [Built-in catalog (41 total)](https://docs.openclaw.ai/providers/venice#built-in-catalog-41-total)\n- [Model discovery](https://docs.openclaw.ai/providers/venice#model-discovery)\n- [Streaming and tool support](https://docs.openclaw.ai/providers/venice#streaming-and-tool-support)\n- [Pricing](https://docs.openclaw.ai/providers/venice#pricing)\n- [Venice (anonymized) vs direct API](https://docs.openclaw.ai/providers/venice#venice-anonymized-vs-direct-api)\n- [Usage examples](https://docs.openclaw.ai/providers/venice#usage-examples)\n- [Troubleshooting](https://docs.openclaw.ai/providers/venice#troubleshooting)\n- [Advanced configuration](https://docs.openclaw.ai/providers/venice#advanced-configuration)\n- [Related](https://docs.openclaw.ai/providers/venice#related)\n\nVenice AI provides **privacy-focused AI inference** with support for uncensored models and access to major proprietary models through their anonymized proxy. All inference is private by default — no training on your data, no logging.\n\n## [​](https://docs.openclaw.ai/providers/venice\\#why-venice-in-openclaw)  Why Venice in OpenClaw\n\n- **Private inference** for open-source models (no logging).\n- **Uncensored models** when you need them.\n- **Anonymized access** to proprietary models (Opus/GPT/Gemini) when quality matters.\n- OpenAI-compatible `/v1` endpoints.\n\n## [​](https://docs.openclaw.ai/providers/venice\\#privacy-modes)  Privacy modes\n\nVenice offers two privacy levels — understanding this is key to choosing your model:\n\n| Mode | Description | Models |\n| --- | --- | --- |\n| **Private** | Fully private. Prompts/responses are **never stored or logged**. Ephemeral. | Llama, Qwen, DeepSeek, Kimi, MiniMax, Venice Uncensored, etc. |\n| **Anonymized** | Proxied through Venice with metadata stripped. The underlying provider (OpenAI, Anthropic, Google, xAI) sees anonymized requests. | Claude, GPT, Gemini, Grok |\n\nAnonymized models are **not** fully private. Venice strips metadata before forwarding, but the underlying provider (OpenAI, Anthropic, Google, xAI) still processes the request. Choose **Private** models when full privacy is required.\n\n## [​](https://docs.openclaw.ai/providers/venice\\#features)  Features\n\n- **Privacy-focused**: Choose between “private” (fully private) and “anonymized” (proxied) modes\n- **Uncensored models**: Access to models without content restrictions\n- **Major model access**: Use Claude, GPT, Gemini, and Grok via Venice’s anonymized proxy\n- **OpenAI-compatible API**: Standard `/v1` endpoints for easy integration\n- **Streaming**: Supported on all models\n- **Function calling**: Supported on select models (check model capabilities)\n- **Vision**: Supported on models with vision capability\n- **No hard rate limits**: Fair-use throttling may apply for extreme usage\n\n## [​](https://docs.openclaw.ai/providers/venice\\#getting-started)  Getting started\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/providers/venice#)\n\nGet your API key\n\n1. Sign up at [venice.ai](https://venice.ai/)\n2. Go to **Settings > API Keys > Create new key**\n3. Copy your API key (format: `vapi_xxxxxxxxxxxx`)\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/providers/venice#)\n\nConfigure OpenClaw\n\nChoose your preferred setup method:\n\n- Interactive (recommended)\n\n- Environment variable\n\n- Non-interactive\n\n\n```\nopenclaw onboard --auth-choice venice-api-key\n```\n\nThis will:\n\n1. Prompt for your API key (or use existing `VENICE_API_KEY`)\n2. Show all available Venice models\n3. Let you pick your default model\n4. Configure the provider automatically\n\n```\nexport VENICE_API_KEY=\"vapi_xxxxxxxxxxxx\"\n```\n\n```\nopenclaw onboard --non-interactive \\\n  --auth-choice venice-api-key \\\n  --venice-api-key \"vapi_xxxxxxxxxxxx\"\n```\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/providers/venice#)\n\nVerify setup\n\n```\nopenclaw agent --model venice/kimi-k2-5 --message \"Hello, are you working?\"\n```\n\n## [​](https://docs.openclaw.ai/providers/venice\\#model-selection)  Model selection\n\nAfter setup, OpenClaw shows all available Venice models. Pick based on your needs:\n\n- **Default model**: `venice/kimi-k2-5` for strong private reasoning plus vision.\n- **High-capability option**: `venice/claude-opus-4-6` for the strongest anonymized Venice path.\n- **Privacy**: Choose “private” models for fully private inference.\n- **Capability**: Choose “anonymized” models to access Claude, GPT, Gemini via Venice’s proxy.\n\nChange your default model anytime:\n\n```\nopenclaw models set venice/kimi-k2-5\nopenclaw models set venice/claude-opus-4-6\n```\n\nList all available models:\n\n```\nopenclaw models list | grep venice\n```\n\nYou can also run `openclaw configure`, select **Model/auth**, and choose **Venice AI**.\n\nUse the table below to pick the right model for your use case.\n\n| Use Case | Recommended Model | Why |\n| --- | --- | --- |\n| **General chat (default)** | `kimi-k2-5` | Strong private reasoning plus vision |\n| **Best overall quality** | `claude-opus-4-6` | Strongest anonymized Venice option |\n| **Privacy + coding** | `qwen3-coder-480b-a35b-instruct` | Private coding model with large context |\n| **Private vision** | `kimi-k2-5` | Vision support without leaving private mode |\n| **Fast + cheap** | `qwen3-4b` | Lightweight reasoning model |\n| **Complex private tasks** | `deepseek-v3.2` | Strong reasoning, but no Venice tool support |\n| **Uncensored** | `venice-uncensored` | No content restrictions |\n\n## [​](https://docs.openclaw.ai/providers/venice\\#deepseek-v4-replay-behavior)  DeepSeek V4 replay behavior\n\nIf Venice exposes DeepSeek V4 models such as `venice/deepseek-v4-pro` or\n`venice/deepseek-v4-flash`, OpenClaw fills the required DeepSeek V4\n`reasoning_content` replay placeholder on assistant tool-call turns when the\nproxy omits it. Venice rejects DeepSeek’s native top-level `thinking` control,\nso OpenClaw keeps that provider-specific replay fix separate from the native\nDeepSeek provider’s thinking controls.\n\n## [​](https://docs.openclaw.ai/providers/venice\\#built-in-catalog-41-total)  Built-in catalog (41 total)\n\nPrivate models (26) — fully private, no logging\n\n| Model ID | Name | Context | Features |\n| --- | --- | --- | --- |\n| `kimi-k2-5` | Kimi K2.5 | 256k | Default, reasoning, vision |\n| `kimi-k2-thinking` | Kimi K2 Thinking | 256k | Reasoning |\n| `llama-3.3-70b` | Llama 3.3 70B | 128k | General |\n| `llama-3.2-3b` | Llama 3.2 3B | 128k | General |\n| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 128k | General, tools disabled |\n| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 128k | Reasoning |\n| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 128k | General |\n| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 256k | Coding |\n| `qwen3-coder-480b-a35b-instruct-turbo` | Qwen3 Coder 480B Turbo | 256k | Coding |\n| `qwen3-5-35b-a3b` | Qwen3.5 35B A3B | 256k | Reasoning, vision |\n| `qwen3-next-80b` | Qwen3 Next 80B | 256k | General |\n| `qwen3-vl-235b-a22b` | Qwen3 VL 235B (Vision) | 256k | Vision |\n| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | Fast, reasoning |\n| `deepseek-v3.2` | DeepSeek V3.2 | 160k | Reasoning, tools disabled |\n| `venice-uncensored` | Venice Uncensored (Dolphin-Mistral) | 32k | Uncensored, tools disabled |\n| `mistral-31-24b` | Venice Medium (Mistral) | 128k | Vision |\n| `google-gemma-3-27b-it` | Google Gemma 3 27B Instruct | 198k | Vision |\n| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 128k | General |\n| `nvidia-nemotron-3-nano-30b-a3b` | NVIDIA Nemotron 3 Nano 30B | 128k | General |\n| `olafangensan-glm-4.7-flash-heretic` | GLM 4.7 Flash Heretic | 128k | Reasoning |\n| `zai-org-glm-4.6` | GLM 4.6 | 198k | General |\n| `zai-org-glm-4.7` | GLM 4.7 | 198k | Reasoning |\n| `zai-org-glm-4.7-flash` | GLM 4.7 Flash | 128k | Reasoning |\n| `zai-org-glm-5` | GLM 5 | 198k | Reasoning |\n| `minimax-m21` | MiniMax M2.1 | 198k | Reasoning |\n| `minimax-m25` | MiniMax M2.5 | 198k | Reasoning |\n\nAnonymized models (15) — via Venice proxy\n\n| Model ID | Name | Context | Features |\n| --- | --- | --- | --- |\n| `claude-opus-4-6` | Claude Opus 4.6 (via Venice) | 1M | Reasoning, vision |\n| `claude-opus-4-5` | Claude Opus 4.5 (via Venice) | 198k | Reasoning, vision |\n| `claude-sonnet-4-6` | Claude Sonnet 4.6 (via Venice) | 1M | Reasoning, vision |\n| `claude-sonnet-4-5` | Claude Sonnet 4.5 (via Venice) | 198k | Reasoning, vision |\n| `openai-gpt-54` | GPT-5.4 (via Venice) | 1M | Reasoning, vision |\n| `openai-gpt-53-codex` | GPT-5.3 Codex (via Venice) | 400k | Reasoning, vision, coding |\n| `openai-gpt-52` | GPT-5.2 (via Venice) | 256k | Reasoning |\n| `openai-gpt-52-codex` | GPT-5.2 Codex (via Venice) | 256k | Reasoning, vision, coding |\n| `openai-gpt-4o-2024-11-20` | GPT-4o (via Venice) | 128k | Vision |\n| `openai-gpt-4o-mini-2024-07-18` | GPT-4o Mini (via Venice) | 128k | Vision |\n| `gemini-3-1-pro-preview` | Gemini 3.1 Pro (via Venice) | 1M | Reasoning, vision |\n| `gemini-3-pro-preview` | Gemini 3 Pro (via Venice) | 198k | Reasoning, vision |\n| `gemini-3-flash-preview` | Gemini 3 Flash (via Venice) | 256k | Reasoning, vision |\n| `grok-41-fast` | Grok 4.1 Fast (via Venice) | 1M | Reasoning, vision |\n| `grok-code-fast-1` | Grok Code Fast 1 (via Venice) | 256k | Reasoning, coding |\n\n## [​](https://docs.openclaw.ai/providers/venice\\#model-discovery)  Model discovery\n\nOpenClaw automatically discovers models from the Venice API when `VENICE_API_KEY` is set. If the API is unreachable, it falls back to a static catalog.The `/models` endpoint is public (no auth needed for listing), but inference requires a valid API key.\n\n## [​](https://docs.openclaw.ai/providers/venice\\#streaming-and-tool-support)  Streaming and tool support\n\n| Feature | Support |\n| --- | --- |\n| **Streaming** | All models |\n| **Function calling** | Most models (check `supportsFunctionCalling` in API) |\n| **Vision/Images** | Models marked with “Vision” feature |\n| **JSON mode** | Supported via `response_format` |\n\n## [​](https://docs.openclaw.ai/providers/venice\\#pricing)  Pricing\n\nVenice uses a credit-based system. Check [venice.ai/pricing](https://venice.ai/pricing) for current rates:\n\n- **Private models**: Generally lower cost\n- **Anonymized models**: Similar to direct API pricing + small Venice fee\n\n### [​](https://docs.openclaw.ai/providers/venice\\#venice-anonymized-vs-direct-api)  Venice (anonymized) vs direct API\n\n| Aspect | Venice (Anonymized) | Direct API |\n| --- | --- | --- |\n| **Privacy** | Metadata stripped, anonymized | Your account linked |\n| **Latency** | +10-50ms (proxy) | Direct |\n| **Features** | Most features supported | Full features |\n| **Billing** | Venice credits | Provider billing |\n\n## [​](https://docs.openclaw.ai/providers/venice\\#usage-examples)  Usage examples\n\n```\n# Use the default private model\nopenclaw agent --model venice/kimi-k2-5 --message \"Quick health check\"\n\n# Use Claude Opus via Venice (anonymized)\nopenclaw agent --model venice/claude-opus-4-6 --message \"Summarize this task\"\n\n# Use uncensored model\nopenclaw agent --model venice/venice-uncensored --message \"Draft options\"\n\n# Use vision model with image\nopenclaw agent --model venice/qwen3-vl-235b-a22b --message \"Review attached image\"\n\n# Use coding model\nopenclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message \"Refactor this function\"\n```\n\n## [​](https://docs.openclaw.ai/providers/venice\\#troubleshooting)  Troubleshooting\n\nAPI key not recognized\n\n```\necho $VENICE_API_KEY\nopenclaw models list | grep venice\n```\n\nEnsure the key starts with `vapi_`.\n\nModel not available\n\nThe Venice model catalog updates dynamically. Run `openclaw models list` to see currently available models. Some models may be temporarily offline.\n\nConnection issues\n\nVenice API is at `https://api.venice.ai/api/v1`. Ensure your network allows HTTPS connections.\n\nMore help: [Troubleshooting](https://docs.openclaw.ai/help/troubleshooting) and [FAQ](https://docs.openclaw.ai/help/faq).\n\n## [​](https://docs.openclaw.ai/providers/venice\\#advanced-configuration)  Advanced configuration\n\nConfig file example\n\n```\n{\n  env: { VENICE_API_KEY: \"vapi_...\" },\n  agents: { defaults: { model: { primary: \"venice/kimi-k2-5\" } } },\n  models: {\n    mode: \"merge\",\n    providers: {\n      venice: {\n        baseUrl: \"https://api.venice.ai/api/v1\",\n        apiKey: \"${VENICE_API_KEY}\",\n        api: \"openai-completions\",\n        models: [\\\n          {\\\n            id: \"kimi-k2-5\",\\\n            name: \"Kimi K2.5\",\\\n            reasoning: true,\\\n            input: [\"text\", \"image\"],\\\n            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\n            contextWindow: 256000,\\\n            maxTokens: 65536,\\\n          },\\\n        ],\n      },\n    },\n  },\n}\n```\n\n## [​](https://docs.openclaw.ai/providers/venice\\#related)  Related\n\n[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)\n\n[**Venice AI** \\\\\n\\\\\nVenice AI homepage and account signup.](https://venice.ai/)\n\n[**API documentation** \\\\\n\\\\\nVenice API reference and developer docs.](https://docs.venice.ai/)\n\n[**Pricing** \\\\\n\\\\\nCurrent Venice credit rates and plans.](https://venice.ai/pricing)\n\n[Together AI](https://docs.openclaw.ai/providers/together) [Vercel AI gateway](https://docs.openclaw.ai/providers/vercel-ai-gateway)\n\nCtrl+I

---

## Qianfan - OpenClaw
**Source:** https://docs.openclaw.ai/providers/qianfan

[Skip to main content](https://docs.openclaw.ai/providers/qianfan#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Qianfan

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/qianfan#getting-started)
- [Built-in catalog](https://docs.openclaw.ai/providers/qianfan#built-in-catalog)
- [Config example](https://docs.openclaw.ai/providers/qianfan#config-example)
- [Related](https://docs.openclaw.ai/providers/qianfan#related)

Qianfan is Baidu’s MaaS platform, providing a **unified API** that routes requests to many models behind a single
endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

| Property | Value |
| --- | --- |
| Provider | `qianfan` |
| Auth | `QIANFAN_API_KEY` |
| API | OpenAI-compatible |
| Base URL | `https://qianfan.baidubce.com/v2` |

## [​](https://docs.openclaw.ai/providers/qianfan\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/qianfan#)

Create a Baidu Cloud account

Sign up or log in at the [Qianfan Console](https://console.bce.baidu.com/qianfan/ais/console/apiKey) and ensure you have Qianfan API access enabled.

2

[Navigate to header](https://docs.openclaw.ai/providers/qianfan#)

Generate an API key

Create a new application or select an existing one, then generate an API key. The key format is `bce-v3/ALTAK-...`.

3

[Navigate to header](https://docs.openclaw.ai/providers/qianfan#)

Run onboarding

```
openclaw onboard --auth-choice qianfan-api-key
```

4

[Navigate to header](https://docs.openclaw.ai/providers/qianfan#)

Verify the model is available

```
openclaw models list --provider qianfan
```

## [​](https://docs.openclaw.ai/providers/qianfan\\#built-in-catalog)  Built-in catalog

| Model ref | Input | Context | Max output | Reasoning | Notes |
| --- | --- | --- | --- | --- | --- |
| `qianfan/deepseek-v3.2` | text | 98,304 | 32,768 | Yes | Default model |
| `qianfan/ernie-5.0-thinking-preview` | text, image | 119,000 | 64,000 | Yes | Multimodal |

The default bundled model ref is `qianfan/deepseek-v3.2`. You only need to override `models.providers.qianfan` when you need a custom base URL or model metadata.

## [​](https://docs.openclaw.ai/providers/qianfan\\#config-example)  Config example

```
{
  env: { QIANFAN_API_KEY: \"bce-v3/ALTAK-...\" },
  agents: {
    defaults: {
      model: { primary: \"qianfan/deepseek-v3.2\" },
      models: {
        \"qianfan/deepseek-v3.2\": { alias: \"QIANFAN\" },
      },
    },
  },
  models: {
    providers: {
      qianfan: {
        baseUrl: \"https://qianfan.baidubce.com/v2\",
        api: \"openai-completions\",
        models: [\\\
          {\\\
            id: \"deepseek-v3.2\\\",\\\
            name: \"DEEPSEEK V3.2\\\",\\\
            reasoning: true,\\\n            input: [\"text\"],\\\
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\
            contextWindow: 98304,\\\n            maxTokens: 32768,\\\n          },\\\n          {\\\
            id: \"ernie-5.0-thinking-preview\\\",\\\
            name: \"ERNIE-5.0-Thinking-Preview\\\",\\\
            reasoning: true,\\\n            input: [\"text\", \"image\"],\\\
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },\\\
            contextWindow: 119000,\\\n            maxTokens: 64000,\\\n          },\\\n        ],
      },
    },
  },
}
```

Transport and compatibility

Qianfan runs through the OpenAI-compatible transport path, not native OpenAI request shaping. This means standard OpenAI SDK features work, but provider-specific parameters may not be forwarded.

Catalog and overrides

The bundled catalog currently includes `deepseek-v3.2` and `ernie-5.0-thinking-preview`. Add or override `models.providers.qianfan` only when you need a custom base URL or model metadata.

Model refs use the `qianfan/` prefix (for example `qianfan/deepseek-v3.2`).

Troubleshooting

- Ensure your API key starts with `bce-v3/ALTAK-` and has Qianfan API access enabled in the Baidu Cloud console.
- If models are not listed, confirm your account has the Qianfan service activated.
- The default base URL is `https://qianfan.baidubce.com/v2`. Only change it if you use a custom endpoint or proxy.

## [​](https://docs.openclaw.ai/providers/qianfan\\#related)  Related

[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Configuration reference** \\\\\n\\\\\nFull OpenClaw configuration reference.](https://docs.openclaw.ai/gateway/configuration-reference)

[**Agent setup** \\\\\n\\\\\nConfiguring agent defaults and model assignments.](https://docs.openclaw.ai/concepts/agent)

[**Qianfan API docs** \\\\\n\\\\\nOfficial Qianfan API documentation.](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)

[Perplexity](https://docs.openclaw.ai/providers/perplexity-provider) [Qwen](https://docs.openclaw.ai/providers/qwen)

Ctrl+I

---

## Inferrs - OpenClaw
**Source:** https://docs.openclaw.ai/providers/inferrs

[Skip to main content](https://docs.openclaw.ai/providers/inferrs#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Inferrs

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/inferrs#getting-started)
- [Full config example](https://docs.openclaw.ai/providers/inferrs#full-config-example)
- [Advanced configuration](https://docs.openclaw.ai/providers/inferrs#advanced-configuration)
- [Troubleshooting](https://docs.openclaw.ai/providers/inferrs#troubleshooting)
- [Related](https://docs.openclaw.ai/providers/inferrs#related)

[inferrs](https://github.com/ericcurtin/inferrs) can serve local models behind an
OpenAI-compatible `/v1` API. OpenClaw works with `inferrs` through the generic
`openai-completions` path.`inferrs` is currently best treated as a custom self-hosted OpenAI-compatible
backend, not a dedicated OpenClaw provider plugin.

## [​](https://docs.openclaw.ai/providers/inferrs\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/inferrs#)

Start inferrs with a model

```
inferrs serve google/gemma-4-E2B-it \\\
  --host 127.0.0.1 \\\
  --port 8080 \\\
  --device metal
```

2

[Navigate to header](https://docs.openclaw.ai/providers/inferrs#)

Verify the server is reachable

```
curl http://127.0.0.1:8080/health
curl http://127.0.0.1:8080/v1/models
```

3

[Navigate to header](https://docs.openclaw.ai/providers/inferrs#)

Add an OpenClaw provider entry

Add an explicit provider entry and point your default model at it. See the full config example below.

## [​](https://docs.openclaw.ai/providers/inferrs\\#full-config-example)  Full config example

This example uses Gemma 4 on a local `inferrs` server.

```
{
  agents: {
    defaults: {
      model: { primary: \"inferrs/google/gemma-4-E2B-it\" },
      models: {
        \"inferrs/google/gemma-4-E2B-it\": {
          alias: \"Gemma 4 (inferrs)\",
        },
      },
    },
  },
  models: {
    mode: \"merge\",
    providers: {
      inferrs: {
        baseUrl: \"http://127.0.0.1:8080/v1\",

---

## Inworld - OpenClaw
**Source:** https://docs.openclaw.ai/providers/inworld

[Skip to main content](https://docs.openclaw.ai/providers/inworld#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Inworld

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/inworld#getting-started)
- [Configuration options](https://docs.openclaw.ai/providers/inworld#configuration-options)
- [Notes](https://docs.openclaw.ai/providers/inworld#notes)
- [Related](https://docs.openclaw.ai/providers/inworld#related)

Inworld is a streaming text-to-speech (TTS) provider. In OpenClaw it
synthesizes outbound reply audio (MP3 by default, OGG\\_OPUS for voice notes)
and PCM audio for telephony channels such as Voice Call.OpenClaw posts to Inworld’s streaming TTS endpoint, concatenates the
returned base64 audio chunks into a single buffer, and hands the result to
the standard reply-audio pipeline.

| Detail | Value |
| --- | --- |
| Website | [inworld.ai](https://inworld.ai/) |
| Docs | [docs.inworld.ai/tts/tts](https://docs.inworld.ai/tts/tts) |
| Auth | `INWORLD_API_KEY` (HTTP Basic, Base64 dashboard credential) |
| Default voice | `Sarah` |
| Default model | `inworld-tts-1.5-max` |

## [​](https://docs.openclaw.ai/providers/inworld\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/inworld#)

Set your API key

Copy the credential from your Inworld dashboard (Workspace > API Keys)
and set it as an env var. The value is sent verbatim as the HTTP Basic
credential, so do not Base64-encode it again or convert it to a bearer
token.

```
INWORLD_API_KEY=<base64-credential-from-dashboard>
```

2

[Navigate to header](https://docs.openclaw.ai/providers/inworld#)

Select Inworld in messages.tts

```
{
  messages: {
    tts: {
      auto: \"always\",
      provider: \"inworld\",
      providers: {
        inworld: {
          voiceId: \"Sarah\",
          modelId: \"inworld-tts-1.5-max\",
        },
      },
    },
  },
}
```

3

[Navigate to header](https://docs.openclaw.ai/providers/inworld#)

Send a message

Send a reply through any connected channel. OpenClaw synthesizes the
audio with Inworld and delivers it as MP3 (or OGG\\_OPUS when the channel
expects a voice note).

## [​](https://docs.openclaw.ai/providers/inworld\\#configuration-options)  Configuration options

| Option | Path | Description |
| --- | --- | --- |
| `apiKey` | `messages.tts.providers.inworld.apiKey` | Base64 dashboard credential. Falls back to `INWORLD_API_KEY`. |
| `baseUrl` | `messages.tts.providers.inworld.baseUrl` | Override Inworld API base URL (default `https://api.inworld.ai`). |
| `voiceId` | `messages.tts.providers.inworld.voiceId` | Voice identifier (default `Sarah`). |
| `modelId` | `messages.tts.providers.inworld.modelId` | TTS model id (default `inworld-tts-1.5-max`). |
| `temperature` | `messages.tts.providers.inworld.temperature` | Sampling temperature `0..2` (optional). |

## [​](https://docs.openclaw.ai/providers/inworld\\#notes)  Notes

Authentication

Inworld uses HTTP Basic auth with a single Base64-encoded credential
string. Copy it verbatim from the Inworld dashboard. The provider sends
it as `Authorization: Basic <apiKey>` without any further encoding, so
do not Base64-encode it yourself and do not pass a bearer-style token.
See [TTS auth notes](https://docs.openclaw.ai/tools/tts#inworld-primary) for the same callout.

Models

Supported model ids: `inworld-tts-1.5-max` (default),
`inworld-tts-1.5-mini`, `inworld-tts-1-max`, `inworld-tts-1`.

Audio outputs

Replies use MP3 by default. When the channel target is `voice-note`
OpenClaw asks Inworld for `OGG_OPUS` so the audio plays as a native
voice bubble. Telephony synthesis uses raw `PCM` at 22050 Hz to feed
the telephony bridge.

Custom endpoints

Override the API host with `messages.tts.providers.inworld.baseUrl`.
Trailing slashes are stripped before requests are sent.

## [​](https://docs.openclaw.ai/providers/inworld\\#related)  Related

[**Text-to-speech** \\\\\n\\\\\nTTS overview, providers, and `messages.tts` config.](https://docs.openclaw.ai/tools/tts)

[**Configuration** \\\\\n\\\\\nFull config reference including `messages.tts` settings.](https://docs.openclaw.ai/gateway/configuration)

[**Providers** \\\\\n\\\\\nAll bundled OpenClaw providers.](https://docs.openclaw.ai/providers)

[**Troubleshooting** \\\\\n\\\\\nCommon issues and debugging steps.](https://docs.openclaw.ai/help/troubleshooting)

[Inferrs](https://docs.openclaw.ai/providers/inferrs) [Kilocode](https://docs.openclaw.ai/providers/kilocode)

Ctrl+I

---

## Kilocode - OpenClaw
**Source:** https://docs.openclaw.ai/providers/kilocode

[Skip to main content](https://docs.openclaw.ai/providers/kilocode#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Kilocode

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Kilo Gateway](https://docs.openclaw.ai/providers/kilocode#kilo-gateway)
- [Getting started](https://docs.openclaw.ai/providers/kilocode#getting-started)
- [Default model](https://docs.openclaw.ai/providers/kilocode#default-model)
- [Built-in catalog](https://docs.openclaw.ai/providers/kilocode#built-in-catalog)
- [Config example](https://docs.openclaw.ai/providers/kilocode#config-example)
- [Related](https://docs.openclaw.ai/providers/kilocode#related)

# [​](https://docs.openclaw.ai/providers/kilocode\\#kilo-gateway)  Kilo Gateway

Kilo Gateway provides a **unified API** that routes requests to many models behind a single
endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

| Property | Value |
| --- | --- |
| Provider | `kilocode` |
| Auth | `KILOCODE_API_KEY` |
| API | OpenAI-compatible |
| Base URL | `https://api.kilo.ai/api/gateway/` |

## [​](https://docs.openclaw.ai/providers/kilocode\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/kilocode#)

Create an account

Go to [app.kilo.ai](https://app.kilo.ai/), sign in or create an account, then navigate to API Keys and generate a new key.

2

[Navigate to header](https://docs.openclaw.ai/providers/kilocode#)

Run onboarding

```
openclaw onboard --auth-choice kilocode-api-key
```

Or set the environment variable directly:

```
export KILOCODE_API_KEY=\"<your-kilocode-api-key>\" # pragma: allowlist secret
```

3

[Navigate to header](https://docs.openclaw.ai/providers/kilocode#)

Verify the model is available

```
openclaw models list --provider kilocode
```

## [​](https://docs.openclaw.ai/providers/kilocode\\#default-model)  Default model

The default model is `kilocode/kilo/auto`, a provider-owned smart-routing
model managed by Kilo Gateway.

OpenClaw treats `kilocode/kilo/auto` as the stable default ref, but does not
publish a source-backed task-to-upstream-model mapping for that route. Exact
upstream routing behind `kilocode/kilo/auto` is owned by Kilo Gateway, not
hard-coded in OpenClaw.

## [​](https://docs.openclaw.ai/providers/kilocode\\#built-in-catalog)  Built-in catalog

OpenClaw dynamically discovers available models from the Kilo Gateway at startup. Use
`/models kilocode` to see the full list of models available with your account.Any model available on the gateway can be used with the `kilocode/` prefix:

| Model ref | Notes |
| --- | --- |
| `kilocode/kilo/auto` | Default — smart routing |
| `kilocode/anthropic/claude-sonnet-4` | Anthropic via Kilo |
| `kilocode/openai/gpt-5.5` | OpenAI via Kilo |
| `kilocode/google/gemini-3-pro-preview` | Google via Kilo |
| …and many more | Use `/models kilocode` to list all |

At startup, OpenClaw queries `GET https://api.kilo.ai/api/gateway/models` and merges
discovered models ahead of the static fallback catalog. The bundled fallback always
includes `kilocode/kilo/auto` (`Kilo Auto`) with `input: [\"text\", \"image\"]`,
`reasoning: true`, `contextWindow: 1000000`, and `maxTokens: 128000`.

## [​](https://docs.openclaw.ai/providers/kilocode\\#config-example)  Config example

```
{
  env: { KILOCODE_API_KEY: \"<your-kilocode-api-key>\" }, // pragma: allowlist secret
  agents: {
    defaults: {
      model: { primary: \"kilocode/kilo/auto\" },
    },
  },
}
```

Transport and compatibility

Kilo Gateway is documented in source as OpenRouter-compatible, so it stays on
the proxy-style OpenAI-compatible path rather than native OpenAI request shaping.

- Gemini-backed Kilo refs stay on the proxy-Gemini path, so OpenClaw keeps
Gemini thought-signature sanitation there without enabling native Gemini
replay validation or bootstrap rewrites.
- Kilo Gateway uses a Bearer token with your API key under the hood.

Stream wrapper and reasoning

Kilo’s shared stream wrapper adds the provider app header and normalizes
proxy reasoning payloads for supported concrete model refs.

`kilocode/kilo/auto` and other proxy-reasoning-unsupported hints skip reasoning
injection. If you need reasoning support, use a concrete model ref such as
`kilocode/anthropic/claude-sonnet-4`.

Troubleshooting

- If model discovery fails at startup, OpenClaw falls back to the bundled static catalog containing `kilocode/kilo/auto`.
- Confirm your API key is valid and that your Kilo account has the desired models enabled.
- When the Gateway runs as a daemon, ensure `KILOCODE_API_KEY` is available to that process (for example in `~/.openclaw/.env` or via `env.shellEnv`).

## [​](https://docs.openclaw.ai/providers/kilocode\\#related)  Related

[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Configuration reference** \\\\\n\\\\\nFull OpenClaw configuration reference.](https://docs.openclaw.ai/gateway/configuration-reference)

[**Kilo Gateway** \\\\\n\\\\\nKilo Gateway dashboard, API keys, and account management.](https://app.kilo.ai/)

[Inworld](https://docs.openclaw.ai/providers/inworld) [LiteLLM](https://docs.openclaw.ai/providers/litellm)

Ctrl+I

---

## OpenCode - OpenClaw
**Source:** https://docs.openclaw.ai/providers/opencode

[Skip to main content](https://docs.openclaw.ai/providers/opencode#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

OpenCode

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/opencode#getting-started)
- [Config example](https://docs.openclaw.ai/providers/opencode#config-example)
- [Built-in catalogs](https://docs.openclaw.ai/providers/opencode#built-in-catalogs)
- [Zen](https://docs.openclaw.ai/providers/opencode#zen)
- [Go](https://docs.openclaw.ai/providers/opencode#go)
- [Advanced configuration](https://docs.openclaw.ai/providers/opencode#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/opencode#related)

OpenCode exposes two hosted catalogs in OpenClaw:

| Catalog | Prefix | Runtime provider |
| --- | --- | --- |
| **Zen** | `opencode/...` | `opencode` |
| **Go** | `opencode-go/...` | `opencode-go` |

Both catalogs use the same OpenCode API key. OpenClaw keeps the runtime provider ids
split so upstream per-model routing stays correct, but onboarding and docs treat them
as one OpenCode setup.

## [​](https://docs.openclaw.ai/providers/opencode\\#getting-started)  Getting started

- Zen catalog

- Go catalog


**Best for:** the curated OpenCode multi-model proxy (Claude, GPT, Gemini).

1

[Navigate to header](https://docs.openclaw.ai/providers/opencode#)

Run onboarding

```
openclaw onboard --auth-choice opencode-zen
```

Or pass the key directly:

```
openclaw onboard --opencode-zen-api-key \"$OPENCODE_API_KEY\"
```

2

[Navigate to header](https://docs.openclaw.ai/providers/opencode#)

Set a Zen model as the default

```
openclaw config set agents.defaults.model.primary \"opencode/claude-opus-4-6\"
```

3

[Navigate to header](https://docs.openclaw.ai/providers/opencode#)

Verify models are available

```
openclaw models list --provider opencode
```

**Best for:** the OpenCode-hosted Kimi, GLM, and MiniMax lineup.

1

[Navigate to header](https://docs.openclaw.ai/providers/opencode#)

Run onboarding

```
openclaw onboard --auth-choice opencode-go
```

Or pass the key directly:

```
openclaw onboard --opencode-go-api-key \"$OPENCODE_API_KEY\"
```

2

[Navigate to header](https://docs.openclaw.ai/providers/opencode#)

Set a Go model as the default

```
openclaw config set agents.defaults.model.primary \"opencode-go/kimi-k2.6\"
```

3

[Navigate to header](https://docs.openclaw.ai/providers/opencode#)

Verify models are available

```
openclaw models list --provider opencode-go
```

## [​](https://docs.openclaw.ai/providers/opencode\\#config-example)  Config example

```
{
  env: { OPENCODE_API_KEY: \"sk-...\" },
  agents: { defaults: { model: { primary: \"opencode/claude-opus-4-6\" } } },
}
```

## [​](https://docs.openclaw.ai/providers/opencode\\#built-in-catalogs)  Built-in catalogs

### [​](https://docs.openclaw.ai/providers/opencode\\#zen)  Zen

| Property | Value |
| --- | --- |
| Runtime provider | `opencode` |
| Example models | `opencode/claude-opus-4-6`, `opencode/gpt-5.5`, `opencode/gemini-3-pro` |

### [​](https://docs.openclaw.ai/providers/opencode\\#go)  Go

| Property | Value |
| --- | --- |
| Runtime provider | `opencode-go` |
| Example models | `opencode-go/kimi-k2.6`, `opencode-go/glm-5`, `opencode-go/minimax-m2.5` |

## [​](https://docs.openclaw.ai/providers/opencode\\#advanced-configuration)  Advanced configuration

API key aliases

`OPENCODE_ZEN_API_KEY` is also supported as an alias for `OPENCODE_API_KEY`.

Shared credentials

Entering one OpenCode key during setup stores credentials for both runtime
providers. You do not need to onboard each catalog separately.

Billing and dashboard

You sign in to OpenCode, add billing details, and copy your API key. Billing
and catalog availability are managed from the OpenCode dashboard.

Gemini replay behavior

Gemini-backed OpenCode refs stay on the proxy-Gemini path, so OpenClaw keeps
Gemini thought-signature sanitation there without enabling native Gemini
replay validation or bootstrap rewrites.

Non-Gemini replay behavior

Non-Gemini OpenCode refs keep the minimal OpenAI-compatible replay policy.

Entering one OpenCode key during setup stores credentials for both the Zen and
Go runtime providers, so you only need to onboard once.

## [​](https://docs.openclaw.ai/providers/opencode\\#related)  Related

[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Configuration reference** \\\\\n\\\\\nFull config reference for agents, models, and providers.](https://docs.openclaw.ai/gateway/configuration-reference)

[OpenAI](https://docs.openclaw.ai/providers/openai) [OpenCode Go](https://docs.openclaw.ai/providers/opencode-go)

Ctrl+I

---

## xAI - OpenClaw
**Source:** https://docs.openclaw.ai/providers/xai

[Skip to main content](https://docs.openclaw.ai/providers/xai#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nProviders\n\nxAI\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Getting started](https://docs.openclaw.ai/providers/xai#getting-started)\n- [Built-in catalog](https://docs.openclaw.ai/providers/xai#built-in-catalog)\n- [OpenClaw feature coverage](https://docs.openclaw.ai/providers/xai#openclaw-feature-coverage)\n- [Fast-mode mappings](https://docs.openclaw.ai/providers/xai#fast-mode-mappings)\n- [Legacy compatibility aliases](https://docs.openclaw.ai/providers/xai#legacy-compatibility-aliases)\n- [Features](https://docs.openclaw.ai/providers/xai#features)\n- [Live testing](https://docs.openclaw.ai/providers/xai#live-testing)\n- [Related](https://docs.openclaw.ai/providers/xai#related)\n\nOpenClaw ships a bundled `xai` provider plugin for Grok models.\n\n## [​](https://docs.openclaw.ai/providers/xai\\#getting-started)  Getting started\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/providers/xai#)\n\nCreate an API key\n\nCreate an API key in the [xAI console](https://console.x.ai/).\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/providers/xai#)\n\nSet your API key\n\nSet `XAI_API_KEY`, or run:\n\n```\nopenclaw onboard --auth-choice xai-api-key\n```\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/providers/xai#)\n\nPick a model\n\n```\n{\n  agents: { defaults: { model: { primary: \"xai/grok-4\" } } },\n}\n```\n\nOpenClaw uses the xAI Responses API as the bundled xAI transport. The same\n`XAI_API_KEY` can also power Grok-backed `web_search`, first-class `x_search`,\nand remote `code_execution`.\nIf you store an xAI key under `plugins.entries.xai.config.webSearch.apiKey`,\nthe bundled xAI model provider reuses that key as a fallback too.\n`code_execution` tuning lives under `plugins.entries.xai.config.codeExecution`.\n\n## [​](https://docs.openclaw.ai/providers/xai\\#built-in-catalog)  Built-in catalog\n\nOpenClaw includes these xAI model families out of the box:\n\n| Family | Model ids |\n| --- | --- |\n| Grok 3 | `grok-3`, `grok-3-fast`, `grok-3-mini`, `grok-3-mini-fast` |\n| Grok 4 | `grok-4`, `grok-4-0709` |\n| Grok 4 Fast | `grok-4-fast`, `grok-4-fast-non-reasoning` |\n| Grok 4.1 Fast | `grok-4-1-fast`, `grok-4-1-fast-non-reasoning` |\n| Grok 4.20 Beta | `grok-4.20-beta-latest-reasoning`, `grok-4.20-beta-latest-non-reasoning` |\n| Grok Code | `grok-code-fast-1` |\n\nThe plugin also forward-resolves newer `grok-4*` and `grok-code-fast*` ids when\nthey follow the same API shape.\n\n`grok-4-fast`, `grok-4-1-fast`, and the `grok-4.20-beta-*` variants are the\ncurrent image-capable Grok refs in the bundled catalog.\n\n## [​](https://docs.openclaw.ai/providers/xai\\#openclaw-feature-coverage)  OpenClaw feature coverage\n\nThe bundled plugin maps xAI’s current public API surface onto OpenClaw’s shared\nprovider and tool contracts. Capabilities that don’t fit the shared contract\n(for example streaming TTS and realtime voice) are not exposed — see the table\nbelow.\n\n| xAI capability | OpenClaw surface | Status |\n| --- | --- | --- |\n| Chat / Responses | `xai/<model>` model provider | Yes |\n| Server-side web search | `web_search` provider `grok` | Yes |\n| Server-side X search | `x_search` tool | Yes |\n| Server-side code execution | `code_execution` tool | Yes |\n| Images | `image_generate` | Yes |\n| Videos | `video_generate` | Yes |\n| Batch text-to-speech | `messages.tts.provider: \"xai\"` / `tts` | Yes |\n| Streaming TTS | — | Not exposed; OpenClaw’s TTS contract returns complete audio buffers |\n| Batch speech-to-text | `tools.media.audio` / media understanding | Yes |\n| Streaming speech-to-text | Voice Call `streaming.provider: \"xai\"` | Yes |\n| Realtime voice | — | Not exposed yet; different session/WebSocket contract |\n| Files / batches | Generic model API compatibility only | Not a first-class OpenClaw tool |\n\nOpenClaw uses xAI’s REST image/video/TTS/STT APIs for media generation,\nspeech, and batch transcription, xAI’s streaming STT WebSocket for live\nvoice-call transcription, and the Responses API for model, search, and\ncode-execution tools. Features that need different OpenClaw contracts, such as\nRealtime voice sessions, are documented here as upstream capabilities rather\nthan hidden plugin behavior.\n\n### [​](https://docs.openclaw.ai/providers/xai\\#fast-mode-mappings)  Fast-mode mappings\n\n`/fast on` or `agents.defaults.models[\"xai/<model>\"].params.fastMode: true`\nrewrites native xAI requests as follows:\n\n| Source model | Fast-mode target |\n| --- | --- |\n| `grok-3` | `grok-3-fast` |\n| `grok-3-mini` | `grok-3-mini-fast` |\n| `grok-4` | `grok-4-fast` |\n| `grok-4-0709` | `grok-4-fast` |\n\n### [​](https://docs.openclaw.ai/providers/xai\\#legacy-compatibility-aliases)  Legacy compatibility aliases\n\nLegacy aliases still normalize to the canonical bundled ids:\n\n| Legacy alias | Canonical id |\n| --- | --- |\n| `grok-4-fast-reasoning` | `grok-4-fast` |\n| `grok-4-1-fast-reasoning` | `grok-4-1-fast` |\n| `grok-4.20-reasoning` | `grok-4.20-beta-latest-reasoning` |\n| `grok-4.20-non-reasoning` | `grok-4.20-beta-latest-non-reasoning` |\n\n## [​](https://docs.openclaw.ai/providers/xai\\#features)  Features\n\nWeb search\n\nThe bundled `grok` web-search provider uses `XAI_API_KEY` too:\n\n```\nopenclaw config set tools.web.search.provider grok\n```\n\nVideo generation\n\nThe bundled `xai` plugin registers video generation through the shared\n`video_generate` tool.\n\n- Default video model: `xai/grok-imagine-video`\n- Modes: text-to-video, image-to-video, reference-image generation, remote\nvideo edit, and remote video extension\n- Aspect ratios: `1:1`, `16:9`, `9:16`, `4:3`, `3:4`, `3:2`, `2:3`\n- Resolutions: `480P`, `720P`\n- Duration: 1-15 seconds for generation/image-to-video, 1-10 seconds when\nusing `reference_image` roles, 2-10 seconds for extension\n- Reference-image generation: set `imageRoles` to `reference_image` for\nevery supplied image; xAI accepts up to 7 such images\n\nLocal video buffers are not accepted. Use remote `http(s)` URLs for\nvideo edit/extend inputs. Image-to-video accepts local image buffers because\nOpenClaw can encode those as data URLs for xAI.\n\nTo use xAI as the default video provider:\n\n```\n{\n  agents: {\n    defaults: {\n      videoGenerationModel: {\n        primary: \"xai/grok-imagine-video\",\n      },\n    },\n  },\n}\n```\n\nSee [Video Generation](https://docs.openclaw.ai/tools/video-generation) for shared tool parameters,\nprovider selection, and failover behavior.\n\nImage generation\n\nThe bundled `xai` plugin registers image generation through the shared\n`image_generate` tool.\n\n- Default image model: `xai/grok-imagine-image`\n- Additional model: `xai/grok-imagine-image-pro`\n- Modes: text-to-image and reference-image edit\n- Reference inputs: one `image` or up to five `images`\n- Aspect ratios: `1:1`, `16:9`, `9:16`, `4:3`, `3:4`, `2:3`, `3:2`\n- Resolutions: `1K`, `2K`\n- Count: up to 4 images\n\nOpenClaw asks xAI for `b64_json` image responses so generated media can be\nstored and delivered through the normal channel attachment path. Local\nreference images are converted to data URLs; remote `http(s)` references are\npassed through.To use xAI as the default image provider:\n\n```\n{\n  agents: {\n    defaults: {\n      imageGenerationModel: {\n        primary: \"xai/grok-imagine-image\",\n      },\n    },\n  },\n}\n```\n\nxAI also documents `quality`, `mask`, `user`, and additional native ratios\nsuch as `1:2`, `2:1`, `9:20`, and `20:9`. OpenClaw forwards only the\nshared cross-provider image controls today; unsupported native-only knobs\nare intentionally not exposed through `image_generate`.\n\nText-to-speech\n\nThe bundled `xai` plugin registers text-to-speech through the shared `tts`\nprovider surface.\n\n- Voices: `eve`, `ara`, `rex`, `sal`, `leo`, `una`\n- Default voice: `eve`\n- Formats: `mp3`, `wav`, `pcm`, `mulaw`, `alaw`\n- Language: BCP-47 code or `auto`\n- Speed: provider-native speed override\n- Native Opus voice-note format is not supported\n\nTo use xAI as the default TTS provider:\n\n```\n{\n  messages: {\n    tts: {\n      provider: \"xai\",\n      providers: {\n        xai: {\n          voiceId: \"eve\",\n        },\n      },\n    },\n  },\n}\n```\n\nOpenClaw uses xAI’s batch `/v1/tts` endpoint. xAI also offers streaming TTS\nover WebSocket, but the OpenClaw speech provider contract currently expects\na complete audio buffer before reply delivery.\n\nSpeech-to-text\n\nThe bundled `xai` plugin registers batch speech-to-text through OpenClaw’s\nmedia-understanding transcription surface.\n\n- Default model: `grok-stt`\n- Endpoint: xAI REST `/v1/stt`\n- Input path: multipart audio file upload\n- Supported by OpenClaw wherever inbound audio transcription uses\n`tools.media.audio`, including Discord voice-channel segments and\nchannel audio attachments\n\nTo force xAI for inbound audio transcription:\n\n```\n{\n  tools: {\n    media: {\n      audio: {\n        models: [\\\n          {\\\n            type: \"provider\",\\\n            provider: \"xai\",\\\n            model: \"grok-stt\",\\\n          },\\\n        ],\n      },\n    },\n  },\n}\n```\n\nLanguage can be supplied through the shared audio media config or per-call\ntranscription request. Prompt hints are accepted by the shared OpenClaw\nsurface, but the xAI REST STT integration only forwards file, model, and\nlanguage because those map cleanly to the current public xAI endpoint.\n\nStreaming speech-to-text\n\nThe bundled `xai` plugin also registers a realtime transcription provider\nfor live voice-call audio.\n\n- Endpoint: xAI WebSocket `wss://api.x.ai/v1/stt`\n- Default encoding: `mulaw`\n- Default sample rate: `8000`\n- Default endpointing: `800ms`\n- Interim transcripts: enabled by default\n\nVoice Call’s Twilio media stream sends G.711 µ-law audio frames, so the\nxAI provider can forward those frames directly without transcoding:\n\n```\n{\n  plugins: {\n    entries: {\n      \"voice-call\": {\n        config: {\n          streaming: {\n            enabled: true,\n            provider: \"xai\",\n            providers: {\n              xai: {\n                apiKey: \"${XAI_API_KEY}\",\n                endpointingMs: 800,\n                language: \"en\",\n              },\n            },\n          },\n        },\n      },\n    },\n  },\n}\n```\n\nProvider-owned config lives under\n`plugins.entries.voice-call.config.streaming.providers.xai`. Supported\nkeys are `apiKey`, `baseUrl`, `sampleRate`, `encoding` (`pcm`, `mulaw`, or\n`alaw`), `interimResults`, `endpointingMs`, and `language`.\n\nThis streaming provider is for Voice Call’s realtime transcription path.\nDiscord voice currently records short segments and uses the batch\n`tools.media.audio` transcription path instead.\n\nx\\_search configuration\n\nThe bundled xAI plugin exposes `x_search` as an OpenClaw tool for searching\nX (formerly Twitter) content via Grok.Config path: `plugins.entries.xai.config.xSearch`\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `enabled` | boolean | — | Enable or disable x\\_search |\n| `model` | string | `grok-4-1-fast` | Model used for x\\_search requests |\n| `inlineCitations` | boolean | — | Include inline citations in results |\n| `maxTurns` | number | — | Maximum conversation turns |\n| `timeoutSeconds` | number | — | Request timeout in seconds |\n| `cacheTtlMinutes` | number | — | Cache time-to-live in minutes |\n\n```\n{\n  plugins: {\n    entries: {\n      xai: {\n        config: {\n          xSearch: {\n            enabled: true,\n            model: \"grok-4-1-fast\",\n            inlineCitations: true,\n          },\n        },\n      },\n    },\n  },\n}\n```\n\nCode execution configuration\n\nThe bundled xAI plugin exposes `code_execution` as an OpenClaw tool for\nremote code execution in xAI’s sandbox environment.Config path: `plugins.entries.xai.config.codeExecution`\n\n| Key | Type | Default | Description |\n| --- | --- | --- | --- |\n| `enabled` | boolean | `true` (if key available) | Enable or disable code execution |\n| `model` | string | `grok-4-1-fast` | Model used for code execution requests |\n| `maxTurns` | number | — | Maximum conversation turns |\n| `timeoutSeconds` | number | — | Request timeout in seconds |\n\nThis is remote xAI sandbox execution, not local [`exec`](https://docs.openclaw.ai/tools/exec).\n\n```\n{\n  plugins: {\n    entries: {\n      xai: {\n        config: {\n          codeExecution: {\n            enabled: true,\n            model: \"grok-4-1-fast\",\n          },\n        },\n      },\n    },\n  },\n}\n```\n\nKnown limits\n\n- Auth is API-key only today. There is no xAI OAuth or device-code flow in\nOpenClaw yet.\n- `grok-4.20-multi-agent-experimental-beta-0304` is not supported on the\nnormal xAI provider path because it requires a different upstream API\nsurface than the standard OpenClaw xAI transport.\n- xAI Realtime voice is not registered as an OpenClaw provider yet. It\nneeds a different bidirectional voice session contract than batch STT or\nstreaming transcription.\n- xAI image `quality`, image `mask`, and extra native-only aspect ratios are\nnot exposed until the shared `image_generate` tool has corresponding\ncross-provider controls.\n\nAdvanced notes\n\n- OpenClaw applies xAI-specific tool-schema and tool-call compatibility fixes\nautomatically on the shared runner path.\n- Native xAI requests default `tool_stream: true`. Set\n`agents.defaults.models[\"xai/<model>\"].params.tool_stream` to `false` to\ndisable it.\n- The bundled xAI wrapper strips unsupported strict tool-schema flags and\nreasoning payload keys before sending native xAI requests.\n- `web_search`, `x_search`, and `code_execution` are exposed as OpenClaw\ntools. OpenClaw enables the specific xAI built-in it needs inside each tool\nrequest instead of attaching all native tools to every chat turn.\n- `x_search` and `code_execution` are owned by the bundled xAI plugin rather\nthan hardcoded into the core model runtime.\n- `code_execution` is remote xAI sandbox execution, not local\n[`exec`](https://docs.openclaw.ai/tools/exec).\n\n## [​](https://docs.openclaw.ai/providers/xai\\#live-testing)  Live testing\n\nThe xAI media paths are covered by unit tests and opt-in live suites. The live\ncommands load secrets from your login shell, including `~/.profile`, before\nprobing `XAI_API_KEY`.\n\n```\npnpm test extensions/xai\nOPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_TEST_QUIET=1 pnpm test:live -- extensions/xai/xai.live.test.ts\nOPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_TEST_QUIET=1 OPENCLAW_LIVE_IMAGE_GENERATION_PROVIDERS=xai pnpm test:live -- test/image-generation.runtime.live.test.ts\n```\n\nThe provider-specific live file synthesizes normal TTS, telephony-friendly PCM\nTTS, transcribes audio through xAI batch STT, streams the same PCM through xAI\nrealtime STT, generates text-to-image output, and edits a reference image. The\nshared image live file verifies the same xAI provider through OpenClaw’s\nruntime selection, fallback, normalization, and media attachment path.\n\n## [​](https://docs.openclaw.ai/providers/xai\\#related)  Related\n\n[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)\n\n[**Video generation** \\\\\n\\\\\nShared video tool parameters and provider selection.](https://docs.openclaw.ai/tools/video-generation)\n\n[**All providers** \\\\\n\\\\\nThe broader provider overview.](https://docs.openclaw.ai/providers/index)\n\n[**Troubleshooting** \\\\\n\\\\\nCommon issues and fixes.](https://docs.openclaw.ai/help/troubleshooting)\n\n[Vydra](https://docs.openclaw.ai/providers/vydra) [Xiaomi MiMo](https://docs.openclaw.ai/providers/xiaomi)\n\nCtrl+I

---

## NVIDIA - OpenClaw
**Source:** https://docs.openclaw.ai/providers/nvidia

[Skip to main content](https://docs.openclaw.ai/providers/nvidia#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

NVIDIA

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/nvidia#getting-started)
- [Config example](https://docs.openclaw.ai/providers/nvidia#config-example)
- [Built-in catalog](https://docs.openclaw.ai/providers/nvidia#built-in-catalog)
- [Advanced configuration](https://docs.openclaw.ai/providers/nvidia#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/nvidia#related)

NVIDIA provides an OpenAI-compatible API at `https://integrate.api.nvidia.com/v1` for
open models for free. Authenticate with an API key from
[build.nvidia.com](https://build.nvidia.com/settings/api-keys).

## [​](https://docs.openclaw.ai/providers/nvidia\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/nvidia#)

Get your API key

Create an API key at [build.nvidia.com](https://build.nvidia.com/settings/api-keys).

2

[Navigate to header](https://docs.openclaw.ai/providers/nvidia#)

Export the key and run onboarding

```
export NVIDIA_API_KEY=\"nvapi-...\"
openclaw onboard --auth-choice skip
```

3

[Navigate to header](https://docs.openclaw.ai/providers/nvidia#)

Set an NVIDIA model

```
openclaw models set nvidia/nvidia/nemotron-3-super-120b-a12b
```

If you pass `--token` instead of the env var, the value lands in shell history and
`ps` output. Prefer the `NVIDIA_API_KEY` environment variable when possible.

## [​](https://docs.openclaw.ai/providers/nvidia\\#config-example)  Config example

```
env: { NVIDIA_API_KEY: \"nvapi-...\" },
  models: {
    providers: {
      nvidia: {
        baseUrl: \"https://integrate.api.nvidia.com/v1\",
        api: \"openai-completions\",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: \"nvidia/nvidia/nemotron-3-super-120b-a12b\" },
    },
  },
}
```

## [​](https://docs.openclaw.ai/providers/nvidia\\#built-in-catalog)  Built-in catalog

| Model ref | Name | Context | Max output |
| --- | --- | --- | --- |
| `nvidia/nvidia/nemotron-3-super-120b-a12b` | NVIDIA Nemotron 3 Super 120B | 262,144 | 8,192 |
| `nvidia/moonshotai/kimi-k2.5` | Kimi K2.5 | 262,144 | 8,192 |
| `nvidia/minimaxai/minimax-m2.5` | Minimax M2.5 | 196,608 | 8,192 |
| `nvidia/z-ai/glm5` | GLM 5 | 202,752 | 8,192 |

## [​](https://docs.openclaw.ai/providers/nvidia\\#advanced-configuration)  Advanced configuration

Auto-enable behavior

The provider auto-enables when the `NVIDIA_API_KEY` environment variable is set.
No explicit provider config is required beyond the key.

Catalog and pricing

The bundled catalog is static. Costs default to `0` in source since NVIDIA
currently offers free API access for the listed models.

OpenAI-compatible endpoint

NVIDIA uses the standard `/v1` completions endpoint. Any OpenAI-compatible
tooling should work out of the box with the NVIDIA base URL.

NVIDIA models are currently free to use. Check
[build.nvidia.com](https://build.nvidia.com/) for the latest availability and
rate-limit details.

## [​](https://docs.openclaw.ai/providers/nvidia\\#related)  Related

[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Configuration reference** \\\\\n\\\\\nFull config reference for agents, models, and providers.](https://docs.openclaw.ai/gateway/configuration-reference)

[Moonshot AI](https://docs.openclaw.ai/providers/moonshot) [Ollama](https://docs.openclaw.ai/providers/ollama)

Ctrl+I

---

## Vercel AI gateway - OpenClaw
**Source:** https://docs.openclaw.ai/providers/vercel-ai-gateway

[Skip to main content](https://docs.openclaw.ai/providers/vercel-ai-gateway#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

Vercel AI gateway

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/vercel-ai-gateway#getting-started)
- [Non-interactive example](https://docs.openclaw.ai/providers/vercel-ai-gateway#non-interactive-example)
- [Model ID shorthand](https://docs.openclaw.ai/providers/vercel-ai-gateway#model-id-shorthand)
- [Advanced configuration](https://docs.openclaw.ai/providers/vercel-ai-gateway#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/vercel-ai-gateway#related)

The [Vercel AI Gateway](https://vercel.com/ai-gateway) provides a unified API to
access hundreds of models through a single endpoint.

| Property | Value |
| --- | --- |
| Provider | `vercel-ai-gateway` |
| Auth | `AI_GATEWAY_API_KEY` |
| API | Anthropic Messages compatible |
| Model catalog | Auto-discovered via `/v1/models` |

OpenClaw auto-discovers the Gateway `/v1/models` catalog, so
`/models vercel-ai-gateway` includes current model refs such as
`vercel-ai-gateway/openai/gpt-5.5` and
`vercel-ai-gateway/moonshotai/kimi-k2.6`.

## [​](https://docs.openclaw.ai/providers/vercel-ai-gateway\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/vercel-ai-gateway#)

Set the API key

Run onboarding and choose the AI Gateway auth option:

```
openclaw onboard --auth-choice ai-gateway-api-key
```

2

[Navigate to header](https://docs.openclaw.ai/providers/vercel-ai-gateway#)

Set a default model

Add the model to your OpenClaw config:

```
{
  agents: {
    defaults: {
      model: { primary: \"vercel-ai-gateway/anthropic/claude-opus-4.6\" },
    },
  },
}
```

3

[Navigate to header](https://docs.openclaw.ai/providers/vercel-ai-gateway#)

Verify the model is available

```
openclaw models list --provider vercel-ai-gateway
```

## [​](https://docs.openclaw.ai/providers/vercel-ai-gateway\#non-interactive-example)  Non-interactive example

For scripted or CI setups, pass all values on the command line:

```
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key \"$AI_GATEWAY_API_KEY\"
```

## [​](https://docs.openclaw.ai/providers/vercel-ai-gateway\#model-id-shorthand)  Model ID shorthand

OpenClaw accepts Vercel Claude shorthand model refs and normalizes them at
runtime:

| Shorthand input | Normalized model ref |
| --- | --- |
| `vercel-ai-gateway/claude-opus-4.6` | `vercel-ai-gateway/anthropic/claude-opus-4.6` |
| `vercel-ai-gateway/opus-4.6` | `vercel-ai-gateway/anthropic/claude-opus-4-6` |

You can use either the shorthand or the fully qualified model ref in your
configuration. OpenClaw resolves the canonical form automatically.

## [​](https://docs.openclaw.ai/providers/vercel-ai-gateway\#advanced-configuration)  Advanced configuration

Environment variable for daemon processes

If the OpenClaw Gateway runs as a daemon (launchd/systemd), make sure
`AI_GATEWAY_API_KEY` is available to that process.

A key set only in `~/.profile` will not be visible to a launchd/systemd
daemon unless that environment is explicitly imported. Set the key in
`~/.openclaw/.env` or via `env.shellEnv` to ensure the gateway process can
read it.

Provider routing

Vercel AI Gateway routes requests to the upstream provider based on the model
ref prefix. For example, `vercel-ai-gateway/anthropic/claude-opus-4.6` routes
through Anthropic, while `vercel-ai-gateway/openai/gpt-5.5` routes through
OpenAI and `vercel-ai-gateway/moonshotai/kimi-k2.6` routes through
MoonshotAI. Your single `AI_GATEWAY_API_KEY` handles authentication for all
upstream providers.

## [​](https://docs.openclaw.ai/providers/vercel-ai-gateway\#related)  Related

[**Model selection** \
\
Choosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Troubleshooting** \
\
General troubleshooting and FAQ.](https://docs.openclaw.ai/help/troubleshooting)

[Venice AI](https://docs.openclaw.ai/providers/venice) [vLLM](https://docs.openclaw.ai/providers/vllm)

Ctrl+I

---

## OpenRouter - OpenClaw
**Source:** https://docs.openclaw.ai/providers/openrouter

[Skip to main content](https://docs.openclaw.ai/providers/openrouter#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Providers

OpenRouter

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Getting started](https://docs.openclaw.ai/providers/openrouter#getting-started)
- [Config example](https://docs.openclaw.ai/providers/openrouter#config-example)
- [Model references](https://docs.openclaw.ai/providers/openrouter#model-references)
- [Image generation](https://docs.openclaw.ai/providers/openrouter#image-generation)
- [Video generation](https://docs.openclaw.ai/providers/openrouter#video-generation)
- [Text-to-speech](https://docs.openclaw.ai/providers/openrouter#text-to-speech)
- [Authentication and headers](https://docs.openclaw.ai/providers/openrouter#authentication-and-headers)
- [Advanced configuration](https://docs.openclaw.ai/providers/openrouter#advanced-configuration)
- [Related](https://docs.openclaw.ai/providers/openrouter#related)

OpenRouter provides a **unified API** that routes requests to many models behind a single
endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

## [​](https://docs.openclaw.ai/providers/openrouter\\#getting-started)  Getting started

1

[Navigate to header](https://docs.openclaw.ai/providers/openrouter#)

Get your API key

Create an API key at [openrouter.ai/keys](https://openrouter.ai/keys).

2

[Navigate to header](https://docs.openclaw.ai/providers/openrouter#)

Run onboarding

```
openclaw onboard --auth-choice openrouter-api-key
```

3

[Navigate to header](https://docs.openclaw.ai/providers/openrouter#)

(Optional) Switch to a specific model

Onboarding defaults to `openrouter/auto`. Pick a concrete model later:

```
openclaw models set openrouter/<provider>/<model>
```

## [​](https://docs.openclaw.ai/providers/openrouter\\#config-example)  Config example

```
{
  env: { OPENROUTER_API_KEY: \"sk-or-...\" },
  agents: {
    defaults: {
      model: { primary: \"openrouter/auto\" },
    },
  },
}
```

## [​](https://docs.openclaw.ai/providers/openrouter\\#model-references)  Model references

Model refs follow the pattern `openrouter/<provider>/<model>`. For the full list of
available providers and models, see [/concepts/model-providers](https://docs.openclaw.ai/concepts/model-providers).

Bundled fallback examples:

| Model ref | Notes |
| --- | --- |
| `openrouter/auto` | OpenRouter automatic routing |
| `openrouter/moonshotai/kimi-k2.6` | Kimi K2.6 via MoonshotAI |

## [​](https://docs.openclaw.ai/providers/openrouter\\#image-generation)  Image generation

OpenRouter can also back the `image_generate` tool. Use an OpenRouter image model under `agents.defaults.imageGenerationModel`:

```
{
  env: { OPENROUTER_API_KEY: \"sk-or-...\" },
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"openrouter/google/gemini-3.1-flash-image-preview\",
        timeoutMs: 180_000,
      },
    },
  },
}
```

OpenClaw sends image requests to OpenRouter’s chat completions image API with `modalities: [\"image\", \"text\"]`. Gemini image models receive supported `aspectRatio` and `resolution` hints through OpenRouter’s `image_config`. Use `agents.defaults.imageGenerationModel.timeoutMs` for slower OpenRouter image models; the `image_generate` tool’s per-call `timeoutMs` parameter still wins.

## [​](https://docs.openclaw.ai/providers/openrouter\\#video-generation)  Video generation

OpenRouter can also back the `video_generate` tool through its asynchronous `/videos` API. Use an OpenRouter video model under `agents.defaults.videoGenerationModel`:

```
{
  env: { OPENROUTER_API_KEY: \"sk-or-...\" },
  agents: {
    defaults: {
      videoGenerationModel: {
        primary: \"openrouter/google/veo-3.1-fast\",
      },
    },
  },
}
```

OpenClaw submits text-to-video and image-to-video jobs to OpenRouter, polls
the returned `polling_url`, and downloads the completed video from
OpenRouter’s `unsigned_urls` or the documented job content endpoint.
Reference images are sent as first/last frame images by default; images
tagged with `reference_image` are sent as OpenRouter input references. The
bundled `google/veo-3.1-fast` default advertises the currently supported 4/6/8
second durations, `720P`/`1080P` resolutions, and `16:9`/`9:16` aspect
ratios. Video-to-video is not registered for OpenRouter because the upstream
video generation API currently accepts text and image references.

## [​](https://docs.openclaw.ai/providers/openrouter\\#text-to-speech)  Text-to-speech

OpenRouter can also be used as a TTS provider through its OpenAI-compatible
`/audio/speech` endpoint.

```
{
  messages: {
    tts: {
      auto: \"always\",
      provider: \"openrouter\",
      providers: {
        openrouter: {
          model: \"hexgrad/kokoro-82m\",
          voice: \"af_alloy\",
          responseFormat: \"mp3\",
        },
      },
    },
  },
}
```

If `messages.tts.providers.openrouter.apiKey` is omitted, TTS reuses
`models.providers.openrouter.apiKey`, then `OPENROUTER_API_KEY`.

## [​](https://docs.openclaw.ai/providers/openrouter\\#authentication-and-headers)  Authentication and headers

OpenRouter uses a Bearer token with your API key under the hood.On real OpenRouter requests (`https://openrouter.ai/api/v1`), OpenClaw also adds
OpenRouter’s documented app-attribution headers:

| Header | Value |
| --- | --- |
| `HTTP-Referer` | `https://openclaw.ai` |
| `X-OpenRouter-Title` | `OpenClaw` |
| `X-OpenRouter-Categories` | `cli-agent` |

If you repoint the OpenRouter provider at some other proxy or base URL, OpenClaw
does **not** inject those OpenRouter-specific headers or Anthropic cache markers.

## [​](https://docs.openclaw.ai/providers/openrouter\\#advanced-configuration)  Advanced configuration

Anthropic cache markers

On verified OpenRouter routes, Anthropic model refs keep the
OpenRouter-specific Anthropic `cache_control` markers that OpenClaw uses for
better prompt-cache reuse on system/developer prompt blocks.

Thinking / reasoning injection

On supported non-`auto` routes, OpenClaw maps the selected thinking level to
OpenRouter proxy reasoning payloads. Unsupported model hints and
`openrouter/auto` skip that reasoning injection. Hunter Alpha also skips
proxy reasoning for stale configured model refs because OpenRouter could
return final answer text in reasoning fields for that retired route.

OpenAI-only request shaping

OpenRouter still runs through the proxy-style OpenAI-compatible path, so
native OpenAI-only request shaping such as `serviceTier`, Responses `store`,
OpenAI reasoning-compat payloads, and prompt-cache hints is not forwarded.

Gemini-backed routes

Gemini-backed OpenRouter refs stay on the proxy-Gemini path: OpenClaw keeps
Gemini thought-signature sanitation there, but does not enable native Gemini
replay validation or bootstrap rewrites.

Provider routing metadata

If you pass OpenRouter provider routing under model params, OpenClaw forwards
it as OpenRouter routing metadata before the shared stream wrappers run.

## [​](https://docs.openclaw.ai/providers/openrouter\\#related)  Related

[**Model selection** \\\\\n\\\\\nChoosing providers, model refs, and failover behavior.](https://docs.openclaw.ai/concepts/model-providers)

[**Configuration reference** \\\\\n\\\\\nFull config reference for agents, models, and providers.](https://docs.openclaw.ai/gateway/configuration-reference)

[OpenCode Go](https://docs.openclaw.ai/providers/opencode-go) [Perplexity](https://docs.openclaw.ai/providers/perplexity-provider)

Ctrl+I

---

