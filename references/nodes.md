# OpenClaw Nodes Documentation

## Media Understanding - Inbound (2026-01-17)
**Source:** https://docs.openclaw.ai/nodes/media-understanding

[Skip to main content](https://docs.openclaw.ai/nodes/media-understanding#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Media capabilities

Media understanding

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Media Understanding - Inbound (2026-01-17)](https://docs.openclaw.ai/nodes/media-understanding#media-understanding-inbound-2026-01-17)
- [Goals](https://docs.openclaw.ai/nodes/media-understanding#goals)
- [High-level behavior](https://docs.openclaw.ai/nodes/media-understanding#high-level-behavior)
- [Config overview](https://docs.openclaw.ai/nodes/media-understanding#config-overview)
- [Model entries](https://docs.openclaw.ai/nodes/media-understanding#model-entries)
- [Defaults and limits](https://docs.openclaw.ai/nodes/media-understanding#defaults-and-limits)
- [Auto-detect media understanding (default)](https://docs.openclaw.ai/nodes/media-understanding#auto-detect-media-understanding-default)
- [Proxy environment support (provider models)](https://docs.openclaw.ai/nodes/media-understanding#proxy-environment-support-provider-models)
- [Capabilities (optional)](https://docs.openclaw.ai/nodes/media-understanding#capabilities-optional)
- [Provider support matrix (OpenClaw integrations)](https://docs.openclaw.ai/nodes/media-understanding#provider-support-matrix-openclaw-integrations)
- [Model selection guidance](https://docs.openclaw.ai/nodes/media-understanding#model-selection-guidance)
- [Attachment policy](https://docs.openclaw.ai/nodes/media-understanding#attachment-policy)
- [Config examples](https://docs.openclaw.ai/nodes/media-understanding#config-examples)
- [1) Shared models list + overrides](https://docs.openclaw.ai/nodes/media-understanding#1-shared-models-list-%2B-overrides)
- [2) Audio + Video only (image off)](https://docs.openclaw.ai/nodes/media-understanding#2-audio-%2B-video-only-image-off)
- [3) Optional image understanding](https://docs.openclaw.ai/nodes/media-understanding#3-optional-image-understanding)
- [4) Multi-modal single entry (explicit capabilities)](https://docs.openclaw.ai/nodes/media-understanding#4-multi-modal-single-entry-explicit-capabilities)
- [Status output](https://docs.openclaw.ai/nodes/media-understanding#status-output)
- [Notes](https://docs.openclaw.ai/nodes/media-understanding#notes)
- [Related docs](https://docs.openclaw.ai/nodes/media-understanding#related-docs)

# [​](https://docs.openclaw.ai/nodes/media-understanding\\#media-understanding-inbound-2026-01-17)  Media Understanding - Inbound (2026-01-17)

OpenClaw can **summarize inbound media** (image/audio/video) before the reply pipeline runs. It auto‑detects when local tools or provider keys are available, and can be disabled or customized. If understanding is off, models still receive the original files/URLs as usual.Vendor-specific media behavior is registered by vendor plugins, while OpenClaw\ncore owns the shared `tools.media` config, fallback order, and reply-pipeline\nintegration.\n\n## [​](https://docs.openclaw.ai/nodes/media-understanding\\#goals)  Goals\n\n- Optional: pre‑digest inbound media into short text for faster routing + better command parsing.\n- Preserve original media delivery to the model (always).\n- Support **provider APIs** and **CLI fallbacks**.\n- Allow multiple models with ordered fallback (error/size/timeout).\n\n## [​](https://openclaw.ai/nodes/media-understanding\\#high-level-behavior)  High-level behavior\n\n1. Collect inbound attachments (`MediaPaths`, `MediaUrls`, `MediaTypes`).\n2. For each enabled capability (image/audio/video), select attachments per policy (default: **first**).\n3. Choose the first eligible model entry (size + capability + auth).\n4. If a model fails or the media is too large, **fall back to the next entry**.\n5. On success:\n   - `Body` becomes `[Image]`, `[Audio]`, or `[Video]` block.\n   - Audio sets `{{Transcript}}`; command parsing uses caption text when present,\n     otherwise the transcript.\n   - Captions are preserved as `User text:` inside the block.\n\nIf understanding fails or is disabled, **the reply flow continues** with the original body + attachments.\n\n## [​](https://docs.openclaw.ai/nodes/media-understanding\\#config-overview)  Config overview\n\n`tools.media` supports **shared models** plus per‑capability overrides:\n\n- `tools.media.models`: shared model list (use `capabilities` to gate).\n- `tools.media.image` / `tools.media.audio` / `tools.media.video`:\n\n  - defaults (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)\n  - provider overrides (`baseUrl`, `headers`, `providerOptions`)\n  - Deepgram audio options via `tools.media.audio.providerOptions.deepgram`\n  - audio transcript echo controls (`echoTranscript`, default `false`; `echoFormat`)\n  - optional **per‑capability `models` list** (preferred before shared models)\n  - `attachments` policy (`mode`, `maxAttachments`, `prefer`)\n  - `scope` (optional gating by channel/chatType/session key)\n- `tools.media.concurrency`: max concurrent capability runs (default **2**).\n\n```\n{\n  tools: {\n    media: {\n      models: [\\\n        /* shared list */\\\n      ],\n      image: {\n        /* optional overrides */\n      },\n      audio: {\n        /* optional overrides */\n        echoTranscript: true,\n        echoFormat: '📝 \"{transcript}\"',\n      },\n      video: {\n        /* optional overrides */\n      },\n    },\n  },\n}\n```\n\n### [​](https://docs.openclaw.ai/nodes/media-understanding\\#model-entries)  Model entries\n\nEach `models[]` entry can be **provider** or **CLI**:\n\n```\n{\n  type: \"provider\", // default if omitted\n  provider: \"openai\",\n  model: \"gpt-5.5\",\n  prompt: \"Describe the image in <= 500 chars.\",\n  maxChars: 500,\n  maxBytes: 10485760,\n  timeoutSeconds: 60,\n  capabilities: [\"image\"], // optional, used for multi‑modal entries\n  profile: \"vision-profile\",\n  preferredProfile: \"vision-fallback\",\n}\n```\n\n```\n{\n  type: \"cli\",\n  command: \"gemini\",\n  args: [\\\n    \"-m\",\\\n    \"gemini-3-flash\",\\\n    \"--allowed-tools\",\\\n    \"read_file\",\\\n    \"Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.\",\\\n  ],\n  maxChars: 500,\n  maxBytes: 52428800,\n  timeoutSeconds: 120,\n  capabilities: [\"video\", \"image\"],\n}\n```\n\nCLI templates can also use:\n\n- `{{MediaDir}}` (directory containing the media file)\n- `{{OutputDir}}` (scratch dir created for this run)\n- `{{OutputBase}}` (scratch file base path, no extension)\n\n## [​](https://docs.openclaw.ai/nodes/media-understanding\\#defaults-and-limits)  Defaults and limits\n\nRecommended defaults:\n\n- `maxChars`: **500** for image/video (short, command‑friendly)\n- `maxChars`: **unset** for audio (full transcript unless you set a limit)\n- `maxBytes`:\n\n  - image: **10MB**\n  - audio: **20MB**\n  - video: **50MB**\n\nRules:\n\n- If media exceeds `maxBytes`, that model is skipped and the **next model is tried**.\n- Audio files smaller than **1024 bytes** are treated as empty/corrupt and skipped before provider/CLI transcription; inbound reply context receives a deterministic placeholder transcript so the agent knows the note was too small.\n- If the model returns more than `maxChars`, output is trimmed.\n- `prompt` defaults to simple “Describe the .” plus the `maxChars` guidance (image/video only).\n- If the active primary image model already supports vision natively, OpenClaw\nskips the `[Image]` summary block and passes the original image into the\nmodel instead.\n- If a Gateway/WebChat primary model is text-only, image attachments are\npreserved as offloaded `media://inbound/*` refs so the image/PDF tools or\nconfigured image model can still inspect them instead of losing the attachment.\n- Explicit `openclaw infer image describe --model <provider/model>` requests\nare different: they run that image-capable provider/model directly, including\nOllama refs such as `ollama/qwen2.5vl:7b`.\n- If `<capability>.enabled: true` but no models are configured, OpenClaw tries the\n**active reply model** when its provider supports the capability.\n\n### [​](https://docs.openclaw.ai/nodes/media-understanding\\#auto-detect-media-understanding-default)  Auto-detect media understanding (default)\n\nIf `tools.media.<capability>.enabled` is **not** set to `false` and you haven’t\nconfigured models, OpenClaw auto-detects in this order and **stops at the first**\n**working option**:\n\n1. **Active reply model** when its provider supports the capability.\n2. **`agents.defaults.imageModel`** primary/fallback refs (image only).\n3. **Local CLIs**(audio only; if installed)\n\n   - `sherpa-onnx-offline` (requires `SHERPA_ONNX_MODEL_DIR` with encoder/decoder/joiner/tokens)\n   - `whisper-cli` (`whisper-cpp`; uses `WHISPER_CPP_MODEL` or the bundled tiny model)\n   - `whisper` (Python CLI; downloads models automatically)\n4. **Gemini CLI** (`gemini`) using `read_many_files`\n5. **Provider auth**\n   - Configured `models.providers.*` entries that support the capability are\n     tried before the bundled fallback order.\n   - Image-only config providers with an image-capable model auto-register for\n     media understanding even when they are not a bundled vendor plugin.\n   - Ollama image understanding is available when selected explicitly, for\n     example through `agents.defaults.imageModel` or\n     `openclaw infer image describe --model ollama/<vision-model>`.\n   - Bundled fallback order:\n     - Audio: OpenAI → Groq → xAI → Deepgram → Google → SenseAudio → ElevenLabs → Mistral\n     - Image: OpenAI → Anthropic → Google → MiniMax → MiniMax Portal → Z.AI\n     - Video: Google → Qwen → Moonshot\n\nTo disable auto-detection, set:\n\n```\n{\n  tools: {\n    media: {\n      audio: {\n        enabled: false,\n      },\n    },\n  },\n}\n```\n\nNote: Binary detection is best-effort across macOS/Linux/Windows; ensure the CLI is on `PATH` (we expand `~`), or set an explicit CLI model with a full command path.\n\n### [​](https://docs.openclaw.ai/nodes/media-understanding\\#proxy-environment-support-provider-models)  Proxy environment support (provider models)\n\nWhen provider-based **audio** and **video** media understanding is enabled, OpenClaw\nhonors standard outbound proxy environment variables for provider HTTP calls:\n\n- `HTTPS_PROXY`\n- `HTTP_PROXY`\n- `https_proxy`\n- `http_proxy`\n\nIf no proxy env vars are set, media understanding uses direct egress.\nIf the proxy value is malformed, OpenClaw logs a warning and falls back to direct\nfetch.\n\n## [​](https://docs.openclaw.ai/nodes/media-understanding\\#capabilities-optional)  Capabilities (optional)\n\nIf you set `capabilities`, the entry only runs for those media types. For shared\nlists, OpenClaw can infer defaults:\n\n- `openai`, `anthropic`, `minimax`: **image**\n- `minimax-portal`: **image**\n- `moonshot`: **image + video**\n- `openrouter`: **image**\n- `google` (Gemini API): **image + audio + video**\n- `qwen`: **image + video**\n- `mistral`: **audio**\n- `zai`: **image**\n- `groq`: **audio**\n- `xai`: **audio**\n- `deepgram`: **audio**\n- Any `models.providers.<id>.models[]` catalog with an image-capable model:\n**image**\n\nFor CLI entries, **set `capabilities` explicitly** to avoid surprising matches.\nIf you omit `capabilities`, the entry is eligible for the list it appears in.\n\n## [​](https://docs.openclaw.ai/nodes/media-understanding\\#provider-support-matrix-openclaw-integrations)  Provider support matrix (OpenClaw integrations)\n\n| Capability | Provider integration | Notes |\n| --- | --- | --- |\n| Image | OpenAI, OpenAI Codex OAuth, Codex app-server, OpenRouter, Anthropic, Google, MiniMax, Moonshot, Qwen, Z.AI, config providers | Vendor plugins register image support; `openai-codex/*` uses OAuth provider plumbing; `codex/*` uses a bounded Codex app-server turn; MiniMax and MiniMax OAuth both use `MiniMax-VL-01`; image-capable config providers auto-register. |\n| Audio | OpenAI, Groq, xAI, Deepgram, Google, SenseAudio, ElevenLabs, Mistral | Provider transcription (Whisper/Groq/xAI/Deepgram/Gemini/SenseAudio/Scribe/Voxtral). |\n| Video | Google, Qwen, Moonshot | Provider video understanding via vendor plugins; Qwen video understanding uses the Standard DashScope endpoints. |\n\nMiniMax note:\n\n- `minimax` and `minimax-portal` image understanding comes from the plugin-owned\n`MiniMax-VL-01` media provider.\n- The bundled MiniMax text catalog still starts text-only; explicit\n`models.providers.minimax` entries materialize image-capable M2.7 chat refs.\n\n## [​](https://docs.openclaw.ai/nodes/media-understanding\\#model-selection-guidance)  Model selection guidance\n\n- Prefer the strongest latest-generation model available for each media capability when quality and safety matter.\n- For tool-enabled agents handling untrusted inputs, avoid older/weaker media models.\n- Keep at least one fallback per capability for availability (quality model + faster/cheaper model).\n- CLI fallbacks (`whisper-cli`, `whisper`, `gemini`) are useful when provider APIs are unavailable.\n- `parakeet-mlx` note: with `--output-dir`, OpenClaw reads `<output-dir>/<media-basename>.txt` when output format is `txt` (or unspecified); non-`txt` formats fall back to stdout.\n\n## [​](https://docs.openclaw.ai/nodes/media-understanding\\#attachment-policy)  Attachment policy\n\nPer‑capability `attachments` controls which attachments are processed:\n\n- `mode`: `first` (default) or `all`\n- `maxAttachments`: cap the number processed (default **1**)\n- `prefer`: `first`, `last`, `path`, `url`\n\nWhen `mode: \"all\"`, outputs are labeled `[Image 1/2]`, `[Audio 2/2]`, etc.File-attachment extraction behavior:\n\n- Extracted file text is wrapped as **untrusted external content** before it is\nappended to the media prompt.\n- The injected block uses explicit boundary markers like\n`<<<EXTERNAL_UNTRUSTED_CONTENT id=\"...\">>>` /\n`<<<END_EXTERNAL_UNTRUSTED_CONTENT id=\"...\">>>` and includes a\n`Source: External` metadata line.\n- This attachment-extraction path intentionally omits the long\n`SECURITY NOTICE:` banner to avoid bloating the media prompt; the boundary\nmarkers and metadata still remain.\n- If a file has no extractable text, OpenClaw injects `[No extractable text]`.\n- If a PDF falls back to rendered page images in this path, the media prompt keeps\nthe placeholder `[PDF content rendered to images; images not forwarded to model]`\nbecause this attachment-extraction step forwards text blocks, not the rendered PDF images.\n\n## [​](https://docs.openclaw.ai/nodes/media-understanding\\#config-examples)  Config examples\n\n### [​](https://docs.openclaw.ai/nodes/media-understanding\\#1-shared-models-list-+-overrides)  1) Shared models list + overrides\n\n```\n{\n  tools: {\n    media: {\n      models: [\\\n        { provider: \"openai\", model: \"gpt-5.5\", capabilities: [\"image\"] },\\\n        {\\\n          provider: \"google\",\\\n          model: \"gemini-3-flash-preview\",\\\n          capabilities: [\"image\", \"audio\", \"video\"],\\\n        },\\\n        {\\\n          type: \"cli\",\\\n          command: \"gemini\",\\\n          args: [\\\n            \"-m\",\\\n            \"gemini-3-flash\",\\\n            \"--allowed-tools\",\\\n            \"read_file\",\\\n            \"Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.\",\\\n          ],\\\n          capabilities: [\"image\", \"video\"],\\\n        },\\\n      ],\n      audio: {\n        attachments: { mode: \"all\", maxAttachments: 2 },\n      },\n      video: {\n        maxChars: 500,\n      },\n    },\n  },\n}\n```\n\n### [​](https://docs.openclaw.ai/nodes/media-understanding\\#2-audio-+-video-only-image-off)  2) Audio + Video only (image off)\n\n```\n{\n  tools: {\n    media: {\n      audio: {\n        enabled: true,\n        models: [\\\n          { provider: \"openai\", model: \"gpt-4o-mini-transcribe\" },\\\n          {\\\n            type: \"cli\",\\\n            command: \"whisper\",\\\n            args: [\"--model\", \"base\", \"{{MediaPath}}\"],\\\n          },\\\n        ],\n      },\n      video: {\n        enabled: true,\n        maxChars: 500,\n        models: [\\\n          { provider: \"google\", model: \"gemini-3-flash-preview\" },\\\n          {\\\n            type: \"cli\",\\\n            command: \"gemini\",\\\n            args: [\\\n              \"-m\",\\\n              \"gemini-3-flash\",\\\n              \"--allowed-tools\",\\\n              \"read_file\",\\\n              \"Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.\",\\\n            ],\\\n          },\\\n        ],\n      },\n    },\n  },\n}\n```\n\n### [​](https://docs.openclaw.ai/nodes/media-understanding\\#3-optional-image-understanding)  3) Optional image understanding\n\n```\n{\n  tools: {\n    media: {\n      image: {\n        enabled: true,\n        maxBytes: 10485760,\n        maxChars: 500,\n        models: [\\\n          { provider: \"openai\", model: \"gpt-5.5\" },\\\n          { provider: \"anthropic\", model: \"claude-opus-4-6\" },\\\n          {\\\n            type: \"cli\",\\\n            command: \"gemini\",\\\n            args: [\\\n              \"-m\",\\\n              \"gemini-3-flash\",\\\n              \"--allowed-tools\",\\\n              \"read_file\",\\\n              \"Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.\",\\\n            ],\\\n          },\\\n        ],\n      },\n    },\n  },\n}\n```\n\n### [​](https://docs.openclaw.ai/nodes/media-understanding\\#4-multi-modal-single-entry-explicit-capabilities)  4) Multi-modal single entry (explicit capabilities)\n\n```\n{\n  tools: {\n    media: {\n      image: {\n        models: [\\\n          {\\\n            provider: \"google\",\\\n            model: \"gemini-3.1-pro-preview\",\\\n            capabilities: [\"image\", \"video\", \"audio\"],\\\n          },\\\n        ],\n      },\n      audio: {\n        models: [\\\n          {\\\n            provider: \"google\",\\\n            model: \"gemini-3.1-pro-preview\",\\\n            capabilities: [\"image\", \"video\", \"audio\"],\\\n          },\\\n        ],\n      },\n      video: {\n        models: [\\\n          {\\\n            provider: \"google\",\\\n            model: \"gemini-3.1-pro-preview\",\\\n            capabilities: [\"image\", \"video\", \"audio\"],\\\n          },\\\n        ],\n      },\n    },\n  },\n}\n```\n\n## [​](https://docs.openclaw.ai/nodes/media-understanding\\#status-output)  Status output\n\nWhen media understanding runs, `/status` includes a short summary line:\n\n```\n📎 Media: image ok (openai/gpt-5.4) · audio skipped (maxBytes)\n```\n\nThis shows per‑capability...(content truncated)

---

## Node troubleshooting - OpenClaw
**Source:** https://docs.openclaw.ai/nodes/troubleshooting

[Skip to main content](https://docs.openclaw.ai/nodes/troubleshooting#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Nodes and media

Node troubleshooting

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Command ladder](https://docs.openclaw.ai/nodes/troubleshooting#command-ladder)
- [Foreground requirements](https://docs.openclaw.ai/nodes/troubleshooting#foreground-requirements)
- [Permissions matrix](https://docs.openclaw.ai/nodes/troubleshooting#permissions-matrix)
- [Pairing versus approvals](https://docs.openclaw.ai/nodes/troubleshooting#pairing-versus-approvals)
- [Common node error codes](https://docs.openclaw.ai/nodes/troubleshooting#common-node-error-codes)
- [Fast recovery loop](https://docs.openclaw.ai/nodes/troubleshooting#fast-recovery-loop)
- [Related](https://docs.openclaw.ai/nodes/troubleshooting#related)

Use this page when a node is visible in status but node tools fail.

## [​](https://docs.openclaw.ai/nodes/troubleshooting\\#command-ladder)  Command ladder

```
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Then run node specific checks:

```
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

Healthy signals:

- Node is connected and paired for role `node`.
- `nodes describe` includes the capability you are calling.
- Exec approvals show expected mode/allowlist.

## [​](https://docs.openclaw.ai/nodes/troubleshooting\\#foreground-requirements)  Foreground requirements

`canvas.*`, `camera.*`, and `screen.*` are foreground only on iOS/Android nodes.Quick check and fix:

```
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

If you see `NODE_BACKGROUND_UNAVAILABLE`, bring the node app to the foreground and retry.

## [​](https://docs.openclaw.ai/nodes/troubleshooting\\#permissions-matrix)  Permissions matrix

| Capability | iOS | Android | macOS node app | Typical failure code |
| --- | --- | --- | --- | --- |
| `camera.snap`, `camera.clip` | Camera (+ mic for clip audio) | Camera (+ mic for clip audio) | Camera (+ mic for clip audio) | `*_PERMISSION_REQUIRED` |
| `screen.record` | Screen Recording (+ mic optional) | Screen capture prompt (+ mic optional) | Screen Recording | `*_PERMISSION_REQUIRED` |
| `location.get` | While Using or Always (depends on mode) | Foreground/Background location based on mode | Location permission | `LOCATION_PERMISSION_REQUIRED` |
| `system.run` | n/a (node host path) | n/a (node host path) | Exec approvals required | `SYSTEM_RUN_DENIED` |

## [​](https://docs.openclaw.ai/nodes/troubleshooting\\#pairing-versus-approvals)  Pairing versus approvals

These are different gates:

1. **Device pairing**: can this node connect to the gateway?
2. **Gateway node command policy**: is the RPC command ID allowed by `gateway.nodes.allowCommands` / `denyCommands` and platform defaults?
3. **Exec approvals**: can this node run a specific shell command locally?

Quick checks:

```
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> \"/usr/bin/uname\"\n```

If pairing is missing, approve the node device first.
If `nodes describe` is missing a command, check the gateway node command policy and whether the node actually declared that command on connect.
If pairing is fine but `system.run` fails, fix exec approvals/allowlist on that node.Node pairing is an identity/trust gate, not a per-command approval surface. For `system.run`, the per-node policy lives in that node’s exec approvals file (`openclaw approvals get --node ...`), not in the gateway pairing record.For approval-backed `host=node` runs, the gateway also binds execution to the\nprepared canonical `systemRunPlan`. If a later caller mutates command/cwd or\nsession metadata before the approved run is forwarded, the gateway rejects the\nrun as an approval mismatch instead of trusting the edited payload.\n
## [​](https://docs.openclaw.ai/nodes/troubleshooting\\#common-node-error-codes)  Common node error codes

- `NODE_BACKGROUND_UNAVAILABLE` → app is backgrounded; bring it foreground.
- `CAMERA_DISABLED` → camera toggle disabled in node settings.
- `*_PERMISSION_REQUIRED` → OS permission missing/denied.
- `LOCATION_DISABLED` → location mode is off.
- `LOCATION_PERMISSION_REQUIRED` → requested location mode not granted.
- `LOCATION_BACKGROUND_UNAVAILABLE` → app is backgrounded but only While Using permission exists.
- `SYSTEM_RUN_DENIED: approval required` → exec request needs explicit approval.
- `SYSTEM_RUN_DENIED: allowlist miss` → command blocked by allowlist mode.
On Windows node hosts, shell-wrapper forms like `cmd.exe /c ...` are treated as allowlist misses in\nallowlist mode unless approved via ask flow.\n
## [​](https://docs.openclaw.ai/nodes/troubleshooting\\#fast-recovery-loop)  Fast recovery loop

```
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

If still stuck:\n
- Re-approve device pairing.
- Re-open node app (foreground).
- Re-grant OS permissions.
- Recreate/adjust exec approval policy.

Related:\n
- [/nodes/index](https://docs.openclaw.ai/nodes/index)
- [/nodes/camera](https://docs.openclaw.ai/nodes/camera)
- [/nodes/location-command](https://docs.openclaw.ai/nodes/location-command)
- [/tools/exec-approvals](https://docs.openclaw.ai/tools/exec-approvals)
- [/gateway/pairing](https://docs.openclaw.ai/gateway/pairing)

## [​](https://docs.openclaw.ai/nodes/troubleshooting\\#related)  Related

- [Nodes overview](https://docs.openclaw.ai/nodes)
- [Gateway troubleshooting](https://docs.openclaw.ai/gateway/troubleshooting)
- [Channel troubleshooting](https://docs.openclaw.ai/channels/troubleshooting)

[Nodes](https://docs.openclaw.ai/nodes) [Media understanding](https://docs.openclaw.ai/nodes/media-understanding)

Ctrl+I

---

## Voice wake - OpenClaw
**Source:** https://docs.openclaw.ai/nodes/voicewake

[Skip to main content](https://docs.openclaw.ai/nodes/voicewake#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Node features

Voice wake

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Storage (Gateway host)](https://docs.openclaw.ai/nodes/voicewake#storage-gateway-host)
- [Protocol](https://docs.openclaw.ai/nodes/voicewake#protocol)
- [Methods](https://docs.openclaw.ai/nodes/voicewake#methods)
- [Routing methods (trigger → target)](https://docs.openclaw.ai/nodes/voicewake#routing-methods-trigger-%E2%86%92-target)
- [Events](https://docs.openclaw.ai/nodes/voicewake#events)
- [Client behavior](https://docs.openclaw.ai/nodes/voicewake#client-behavior)
- [macOS app](https://docs.openclaw.ai/nodes/voicewake#macos-app)
- [iOS node](https://docs.openclaw.ai/nodes/voicewake#ios-node)
- [Android node](https://docs.openclaw.ai/nodes/voicewake#android-node)
- [Related](https://docs.openclaw.ai/nodes/voicewake#related)

OpenClaw treats **wake words as a single global list** owned by the **Gateway**.

- There are **no per-node custom wake words**.
- **Any node/app UI may edit** the list; changes are persisted by the Gateway and broadcast to everyone.
- macOS and iOS keep local **Voice Wake enabled/disabled** toggles (local UX + permissions differ).
- Android currently keeps Voice Wake off and uses a manual mic flow in the Voice tab.

## [​](https://docs.openclaw.ai/nodes/voicewake\\#storage-gateway-host)  Storage (Gateway host)

Wake words are stored on the gateway machine at:

- `~/.openclaw/settings/voicewake.json`

Shape:

```
{ \"triggers\": [\"openclaw\", \"claude\", \"computer\"], \"updatedAtMs\": 1730000000000 }
```

## [​](https://docs.openclaw.ai/nodes/voicewake\\#protocol)  Protocol

### [​](https://docs.openclaw.ai/nodes/voicewake\\#methods)  Methods

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` with params `{ triggers: string[] }` → `{ triggers: string[] }`

Notes:

- Triggers are normalized (trimmed, empties dropped). Empty lists fall back to defaults.
- Limits are enforced for safety (count/length caps).

### [​](https://docs.openclaw.ai/nodes/voicewake\\#routing-methods-trigger-%E2%86%92-target)  Routing methods (trigger → target)

- `voicewake.routing.get` → `{ config: VoiceWakeRoutingConfig }`
- `voicewake.routing.set` with params `{ config: VoiceWakeRoutingConfig }` → `{ config: VoiceWakeRoutingConfig }`

`VoiceWakeRoutingConfig` shape:

```
{
  \"version\": 1,
  \"defaultTarget\": { \"mode\": \"current\" },
  \"routes\": [{ \"trigger\": \"robot wake\", \"target\": { \"sessionKey\": \"agent:main:main\" } }],
  \"updatedAtMs\": 1730000000000
}
```

Route targets support exactly one of:

- `{ \"mode\": \"current\" }`
- `{ \"agentId\": \"main\" }`
- `{ \"sessionKey\": \"agent:main:main\" }`

### [​](https://docs.openclaw.ai/nodes/voicewake\\#events)  Events

- `voicewake.changed` payload `{ triggers: string[] }`
- `voicewake.routing.changed` payload `{ config: VoiceWakeRoutingConfig }`

Who receives it:

- All WebSocket clients (macOS app, WebChat, etc.)
- All connected nodes (iOS/Android), and also on node connect as an initial “current state” push.

## [​](https://docs.openclaw.ai/nodes/voicewake\\#client-behavior)  Client behavior

### [​](https://docs.openclaw.ai/nodes/voicewake\\#macos-app)  macOS app

- Uses the global list to gate `VoiceWakeRuntime` triggers.
- Editing “Trigger words” in Voice Wake settings calls `voicewake.set` and then relies on the broadcast to keep other clients in sync.

### [​](https://openclaw.ai/nodes/voicewake#ios-node)  iOS node

- Uses the global list for `VoiceWakeManager` trigger detection.
- Editing Wake Words in Settings calls `voicewake.set` (over the Gateway WS) and also keeps local wake-word detection responsive.

### [​](https://docs.openclaw.ai/nodes/voicewake\\#android-node)  Android node

- Voice Wake is currently disabled in Android runtime/Settings.
- Android voice uses manual mic capture in the Voice tab instead of wake-word triggers.

## [​](https://docs.openclaw.ai/nodes/voicewake\\#related)  Related

- [Talk mode](https://docs.openclaw.ai/nodes/talk)
- [Audio and voice notes](https://docs.openclaw.ai/nodes/audio)
- [Media understanding](https://docs.openclaw.ai/nodes/media-understanding)

[Talk mode](https://docs.openclaw.ai/nodes/talk) [Location command](https://docs.openclaw.ai/nodes/location-command)

Ctrl+I

---

## Image and media support - OpenClaw
**Source:** https://docs.openclaw.ai/nodes/images

[Skip to main content](https://docs.openclaw.ai/nodes/images#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Media capabilities

Image and media support

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Image & Media Support (2025-12-05)](https://docs.openclaw.ai/nodes/images#image-%26-media-support-2025-12-05)
- [Goals](https://docs.openclaw.ai/nodes/images#goals)
- [CLI Surface](https://docs.openclaw.ai/nodes/images#cli-surface)
- [WhatsApp Web channel behavior](https://docs.openclaw.ai/nodes/images#whatsapp-web-channel-behavior)
- [Auto-Reply Pipeline](https://docs.openclaw.ai/nodes/images#auto-reply-pipeline)
- [Inbound media to commands (Pi)](https://docs.openclaw.ai/nodes/images#inbound-media-to-commands-pi)
- [Limits & Errors](https://docs.openclaw.ai/nodes/images#limits-%26-errors)
- [Notes for Tests](https://docs.openclaw.ai/nodes/images#notes-for-tests)
- [Related](https://docs.openclaw.ai/nodes/images#related)

# [​](https://docs.openclaw.ai/nodes/images\\#image-&-media-support-2025-12-05)  Image & Media Support (2025-12-05)

The WhatsApp channel runs via **Baileys Web**. This document captures the current media handling rules for send, gateway, and agent replies.

## [​](https://docs.openclaw.ai/nodes/images\\#goals)  Goals

- Send media with optional captions via `openclaw message send --media`.
- Allow auto-replies from the web inbox to include media alongside text.
- Keep per-type limits sane and predictable.

## [​](https://docs.openclaw.ai/nodes/images\\#cli-surface)  CLI Surface

- `openclaw message send --media <path-or-url> [--message <caption>]`
  - `--media` optional; caption can be empty for media-only sends.
  - `--dry-run` prints the resolved payload; `--json` emits `{ channel, to, messageId, mediaUrl, caption }`.

## [​](https://docs.openclaw.ai/nodes/images\\#whatsapp-web-channel-behavior)  WhatsApp Web channel behavior

- Input: local file path **or** HTTP(S) URL.
- Flow: load into a Buffer, detect media kind, and build the correct payload:
  - **Images:** resize & recompress to JPEG (max side 2048px) targeting `channels.whatsapp.mediaMaxMb` (default: 50 MB).
  - **Audio/Voice/Video:** pass-through up to 16 MB; audio is sent as a voice note (`ptt: true`).
  - **Documents:** anything else, up to 100 MB, with filename preserved when available.
- WhatsApp GIF-style playback: send an MP4 with `gifPlayback: true` (CLI: `--gif-playback`) so mobile clients loop inline.
- MIME detection prefers magic bytes, then headers, then file extension.
- Caption comes from `--message` or `reply.text`; empty caption is allowed.
- Logging: non-verbose shows `↩️`/`✅`; verbose includes size and source path/URL.

## [​](https://docs.openclaw.ai/nodes/images\\#auto-reply-pipeline)  Auto-Reply Pipeline

- `getReplyFromConfig` returns `{ text?, mediaUrl?, mediaUrls? }`.
- When media is present, the web sender resolves local paths or URLs using the same pipeline as `openclaw message send`.
- Multiple media entries are sent sequentially if provided.

## [​](https://openclaw.ai/nodes/images\\#inbound-media-to-commands-pi)  Inbound media to commands (Pi)

- When inbound web messages include media, OpenClaw downloads to a temp file and exposes templating variables:
  - `{{MediaUrl}}` pseudo-URL for the inbound media.
  - `{{MediaPath}}` local temp path written before running the command.
- When a per-session Docker sandbox is enabled, inbound media is copied into the sandbox workspace and `MediaPath`/`MediaUrl` are rewritten to a relative path like `media/inbound/<filename>`.
- Media understanding (if configured via `tools.media.*` or shared `tools.media.models`) runs before templating and can insert `[Image]`, `[Audio]`, and `[Video]` blocks into `Body`.

  - Audio sets `{{Transcript}}` and uses the transcript for command parsing so slash commands still work.
  - Video and image descriptions preserve any caption text for command parsing.
  - If the active primary image model already supports vision natively, OpenClaw skips the `[Image]` summary block and passes the original image to the model instead.
- By default only the first matching image/audio/video attachment is processed; set `tools.media.<cap>.attachments` to process multiple attachments.

## [​](https://docs.openclaw.ai/nodes/images\\#limits-&-errors)  Limits & Errors

**Outbound send caps (WhatsApp web send)**

- Images: up to `channels.whatsapp.mediaMaxMb` (default: 50 MB) after recompression.
- Audio/voice/video: 16 MB cap; documents: 100 MB cap.
- Oversize or unreadable media → clear error in logs and the reply is skipped.

**Media understanding caps (transcription/description)**

- Image default: 10 MB (`tools.media.image.maxBytes`).
- Audio default: 20 MB (`tools.media.audio.maxBytes`).
- Video default: 50 MB (`tools.media.video.maxBytes`).
- Oversize media skips understanding, but replies still go through with the original body.

## [​](https://docs.openclaw.ai/nodes/images\\#notes-for-tests)  Notes for Tests)

- Cover send + reply flows for image/audio/document cases.
- Validate recompression for images (size bound) and voice-note flag for audio.
- Ensure multi-media replies fan out as sequential sends.

## [​](https://docs.openclaw.ai/nodes/images\\#related)  Related

- [Camera capture](https://docs.openclaw.ai/nodes/camera)
- [Media understanding](https://docs.openclaw.ai/nodes/media-understanding)
- [Audio and voice notes](https://docs.openclaw.ai/nodes/audio)

[Media understanding](https://docs.openclaw.ai/nodes/media-understanding) [Audio and voice notes](https://docs.openclaw.ai/nodes/audio)

Ctrl+I

---

