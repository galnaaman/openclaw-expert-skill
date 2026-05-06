> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Alibaba Model Studio

OpenClaw ships a bundled `alibaba` plugin that registers a video-generation provider for Wan models on Alibaba Model Studio (the international name for DashScope). The plugin is enabled by default; you only need to set an API key.

| Property         | Value                                                                           |
| ---------------- | ------------------------------------------------------------------------------- |
| Provider id      | `alibaba`                                                                       |
| Plugin           | bundled, `enabledByDefault: true`                                               |
| Auth env vars    | `MODELSTUDIO_API_KEY` → `DASHSCOPE_API_KEY` → `QWEN_API_KEY` (first match wins) |
| Onboarding flag  | `--auth-choice alibaba-model-studio-api-key`                                    |
| Direct CLI flag  | `--alibaba-model-studio-api-key <key>`                                          |
| Default model    | `alibaba/wan2.6-t2v`                                                            |
| Default base URL | `https://dashscope-intl.aliyuncs.com`                                           |

## Getting started

<Steps>
  <Step title="Set an API key">
    Use onboarding to store the key against the `alibaba` provider:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --auth-choice alibaba-model-studio-api-key
    ```

    Or pass the key directly during install/onboarding:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --alibaba-model-studio-api-key <your-key>
    ```

    Or export any of the accepted env vars before starting the Gateway:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    export MODELSTUDIO_API_KEY=sk-...
    # or DASHSCOPE_API_KEY=...
    # or QWEN_API_KEY=...
    ```
  </Step>

  <Step title="Set a default video model">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          videoGenerationModel: {
            primary: "alibaba/wan2.6-t2v",
          },
        },
      },
    }
    ```
  </Step>

  <Step title="Verify the provider is configured">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw models list --provider alibaba
    ```

    The list should include all five bundled Wan models. If `MODELSTUDIO_API_KEY` is unresolved, `openclaw models status --json` reports the missing credential under `auth.unusableProfiles`.
  </Step>
</Steps>

<Note>
  The Alibaba plugin and the [Qwen plugin](/providers/qwen) both authenticate against DashScope and accept overlapping env vars. Use `alibaba/...` model ids to drive the dedicated Wan video surface; use `qwen/...` ids when you want Qwen's chat, embedding, or media-understanding surface.
</Note>

## Built-in Wan models

| Model ref                  | Mode                      |
| -------------------------- | ------------------------- |
| `alibaba/wan2.6-t2v`       | Text-to-video (default)   |
| `alibaba/wan2.6-i2v`       | Image-to-video            |
| `alibaba/wan2.6-r2v`       | Reference-to-video        |
| `alibaba/wan2.6-r2v-flash` | Reference-to-video (fast) |
| `alibaba/wan2.7-r2v`       | Reference-to-video        |

## Capabilities and limits

The bundled provider mirrors DashScope's Wan video API caps. All three modes share the same per-request video count and duration cap; only the input shape differs.

| Mode               | Max output videos | Max input images | Max input videos | Max duration | Supported controls                                        |
| ------------------ | ----------------- | ---------------- | ---------------- | ------------ | --------------------------------------------------------- |
| Text-to-video      | 1                 | n/a              | n/a              | 10 s         | `size`, `aspectRatio`, `resolution`, `audio`, `watermark` |
| Image-to-video     | 1                 | 1                | n/a              | 10 s         | `size`, `aspectRatio`, `resolution`, `audio`, `watermark` |
| Reference-to-video | 1                 | n/a              | 4                | 10 s         | `size`, `aspectRatio`, `resolution`, `audio`, `watermark` |

When a request omits `durationSeconds`, the provider sends DashScope's accepted default of **5 seconds**. Set `durationSeconds` explicitly on the [video generation tool](/tools/video-generation) to extend up to 10 s.

<Warning>
  Reference image and video inputs must be remote `http(s)` URLs. Local file paths are not accepted by DashScope's reference modes; upload to object storage first or use the [media tool](/tools/media-overview) flow that already produces a public URL.
</Warning>

## Advanced configuration

<AccordionGroup>
  <Accordion title="Override the DashScope base URL">
    The provider defaults to the international DashScope endpoint. To target the China-region endpoint, set:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          alibaba: {
            baseUrl: "https://dashscope.aliyuncs.com",
          },
        },
      },
    }
    ```

    The provider strips trailing slashes before constructing AIGC task URLs.
  </Accordion>

  <Accordion title="Auth env priority">
    OpenClaw resolves the Alibaba API key from environment variables in this order, taking the first non-empty value:

    1. `MODELSTUDIO_API_KEY`
    2. `DASHSCOPE_API_KEY`
    3. `QWEN_API_KEY`

    Configured `auth.profiles` entries (set via `openclaw models auth login`) override env-var resolution. See [Auth profiles in the models FAQ](/help/faq-models#what-is-an-auth-profile) for profile rotation, cooldown, and override mechanics.
  </Accordion>

  <Accordion title="Relationship to the Qwen plugin">
    Both bundled plugins talk to DashScope and accept overlapping API keys. Use:

    * `alibaba/wan*.*` ids to drive the dedicated Wan video provider documented on this page.
    * `qwen/*` ids for Qwen chat, embedding, and media understanding (see [Qwen](/providers/qwen)).

    Setting `MODELSTUDIO_API_KEY` once authenticates both plugins because the auth env var list intentionally overlaps; you do not need to onboard each plugin separately.
  </Accordion>
</AccordionGroup>

## Related

<CardGroup cols={2}>
  <Card title="Video generation" href="/tools/video-generation" icon="video">
    Shared video tool parameters and provider selection.
  </Card>

  <Card title="Qwen" href="/providers/qwen" icon="microchip">
    Qwen chat, embedding, and media-understanding setup on the same DashScope auth.
  </Card>

  <Card title="Configuration reference" href="/gateway/config-agents#agent-defaults" icon="gear">
    Agent defaults and model configuration.
  </Card>

  <Card title="Models FAQ" href="/help/faq-models" icon="circle-question">
    Auth profiles, switching models, and resolving "no profile" errors.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Anthropic

Anthropic builds the **Claude** model family. OpenClaw supports two auth routes:

* **API key** — direct Anthropic API access with usage-based billing (`anthropic/*` models)
* **Claude CLI** — reuse an existing Claude CLI login on the same host

<Warning>
  Anthropic staff told us OpenClaw-style Claude CLI usage is allowed again, so
  OpenClaw treats Claude CLI reuse and `claude -p` usage as sanctioned unless
  Anthropic publishes a new policy.

  For long-lived gateway hosts, Anthropic API keys are still the clearest and
  most predictable production path.

  Anthropic's current public docs:

  * [Claude Code CLI reference](https://code.claude.com/docs/en/cli-reference)
  * [Claude Agent SDK overview](https://platform.claude.com/docs/en/agent-sdk/overview)
  * [Using Claude Code with your Pro or Max plan](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan)
  * [Using Claude Code with your Team or Enterprise plan](https://support.anthropic.com/en/articles/11845131-using-claude-code-with-your-team-or-enterprise-plan/)
</Warning>

## Getting started

<Tabs>
  <Tab title="API key">
    **Best for:** standard API access and usage-based billing.

    <Steps>
      <Step title="Get your API key">
        Create an API key in the [Anthropic Console](https://console.anthropic.com/).
      </Step>

      <Step title="Run onboarding">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw onboard
        # choose: Anthropic API key
        ```

        Or pass the key directly:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
        ```
      </Step>

      <Step title="Verify the model is available">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models list --provider anthropic
        ```
      </Step>
    </Steps>

    ### Config example

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      env: { ANTHROPIC_API_KEY: "sk-ant-..." },
      agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
    }
    ```
  </Tab>

  <Tab title="Claude CLI">
    **Best for:** reusing an existing Claude CLI login without a separate API key.

    <Steps>
      <Step title="Ensure Claude CLI is installed and logged in">
        Verify with:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        claude --version
        ```
      </Step>

      <Step title="Run onboarding">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw onboard
        # choose: Claude CLI
        ```

        OpenClaw detects and reuses the existing Claude CLI credentials.
      </Step>

      <Step title="Verify the model is available">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models list --provider anthropic
        ```
      </Step>
    </Steps>

    <Note>
      Setup and runtime details for the Claude CLI backend are in [CLI Backends](/gateway/cli-backends).
    </Note>

    ### Config example

    Prefer the canonical Anthropic model ref plus a CLI runtime override:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          model: { primary: "anthropic/claude-opus-4-7" },
          agentRuntime: { id: "claude-cli" },
        },
      },
    }
    ```

    Legacy `claude-cli/claude-opus-4-7` model refs still work for
    compatibility, but new config should keep provider/model selection as
    `anthropic/*` and put the execution backend in `agentRuntime.id`.

    <Tip>
      If you want the clearest billing path, use an Anthropic API key instead. OpenClaw also supports subscription-style options from [OpenAI Codex](/providers/openai), [Qwen Cloud](/providers/qwen), [MiniMax](/providers/minimax), and [Z.AI / GLM](/providers/glm).
    </Tip>
  </Tab>
</Tabs>

## Thinking defaults (Claude 4.6)

Claude 4.6 models default to `adaptive` thinking in OpenClaw when no explicit thinking level is set.

Override per-message with `/think:<level>` or in model params:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { thinking: "adaptive" },
        },
      },
    },
  },
}
```

<Note>
  Related Anthropic docs:

  * [Adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
  * [Extended thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)
</Note>

## Prompt caching

OpenClaw supports Anthropic's prompt caching feature for API-key auth.

| Value               | Cache duration | Description                            |
| ------------------- | -------------- | -------------------------------------- |
| `"short"` (default) | 5 minutes      | Applied automatically for API-key auth |
| `"long"`            | 1 hour         | Extended cache                         |
| `"none"`            | No caching     | Disable prompt caching                 |

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

<AccordionGroup>
  <Accordion title="Per-agent cache overrides">
    Use model-level params as your baseline, then override specific agents via `agents.list[].params`:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          model: { primary: "anthropic/claude-opus-4-6" },
          models: {
            "anthropic/claude-opus-4-6": {
              params: { cacheRetention: "long" },
            },
          },
        },
        list: [
          { id: "research", default: true },
          { id: "alerts", params: { cacheRetention: "none" } },
        ],
      },
    }
    ```

    Config merge order:

    1. `agents.defaults.models["provider/model"].params`
    2. `agents.list[].params` (matching `id`, overrides by key)

    This lets one agent keep a long-lived cache while another agent on the same model disables caching for bursty/low-reuse traffic.
  </Accordion>

  <Accordion title="Bedrock Claude notes">
    * Anthropic Claude models on Bedrock (`amazon-bedrock/*anthropic.claude*`) accept `cacheRetention` pass-through when configured.
    * Non-Anthropic Bedrock models are forced to `cacheRetention: "none"` at runtime.
    * API-key smart defaults also seed `cacheRetention: "short"` for Claude-on-Bedrock refs when no explicit value is set.
  </Accordion>
</AccordionGroup>

## Advanced configuration

<AccordionGroup>
  <Accordion title="Fast mode">
    OpenClaw's shared `/fast` toggle supports direct Anthropic traffic (API-key and OAuth to `api.anthropic.com`).

    | Command     | Maps to                         |
    | ----------- | ------------------------------- |
    | `/fast on`  | `service_tier: "auto"`          |
    | `/fast off` | `service_tier: "standard_only"` |

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          models: {
            "anthropic/claude-sonnet-4-6": {
              params: { fastMode: true },
            },
          },
        },
      },
    }
    ```

    <Note>
      * Only injected for direct `api.anthropic.com` requests. Proxy routes leave `service_tier` untouched.
      * Explicit `serviceTier` or `service_tier` params override `/fast` when both are set.
      * On accounts without Priority Tier capacity, `service_tier: "auto"` may resolve to `standard`.
    </Note>
  </Accordion>

  <Accordion title="Media understanding (image and PDF)">
    The bundled Anthropic plugin registers image and PDF understanding. OpenClaw
    auto-resolves media capabilities from the configured Anthropic auth — no
    additional config is needed.

    | Property        | Value                 |
    | --------------- | --------------------- |
    | Default model   | `claude-opus-4-6`     |
    | Supported input | Images, PDF documents |

    When an image or PDF is attached to a conversation, OpenClaw automatically
    routes it through the Anthropic media understanding provider.
  </Accordion>

  <Accordion title="1M context window (beta)">
    Anthropic's 1M context window is beta-gated. Enable it per model:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          models: {
            "anthropic/claude-opus-4-6": {
              params: { context1m: true },
            },
          },
        },
      },
    }
    ```

    OpenClaw maps this to `anthropic-beta: context-1m-2025-08-07` on requests.

    `params.context1m: true` also applies to the Claude CLI backend
    (`claude-cli/*`) for eligible Opus and Sonnet models, expanding the runtime
    context window for those CLI sessions to match the direct-API behavior.

    <Warning>
      Requires long-context access on your Anthropic credential. Legacy token auth (`sk-ant-oat-*`) is rejected for 1M context requests — OpenClaw logs a warning and falls back to the standard context window.
    </Warning>
  </Accordion>

  <Accordion title="Claude Opus 4.7 1M context">
    `anthropic/claude-opus-4.7` and its `claude-cli` variant have a 1M context
    window by default — no `params.context1m: true` needed.
  </Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
  <Accordion title="401 errors / token suddenly invalid">
    Anthropic token auth expires and can be revoked. For new setups, use an Anthropic API key instead.
  </Accordion>

  <Accordion title="No API key found for provider &#x22;anthropic&#x22;">
    Anthropic auth is **per agent** — new agents do not inherit the main agent's keys. Re-run onboarding for that agent (or configure an API key on the gateway host), then verify with `openclaw models status`.
  </Accordion>

  <Accordion title="No credentials found for profile &#x22;anthropic:default&#x22;">
    Run `openclaw models status` to see which auth profile is active. Re-run onboarding, or configure an API key for that profile path.
  </Accordion>

  <Accordion title="No available auth profile (all in cooldown)">
    Check `openclaw models status --json` for `auth.unusableProfiles`. Anthropic rate-limit cooldowns can be model-scoped, so a sibling Anthropic model may still be usable. Add another Anthropic profile or wait for cooldown.
  </Accordion>
</AccordionGroup>

<Note>
  More help: [Troubleshooting](/help/troubleshooting) and [FAQ](/help/faq).
</Note>

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="CLI backends" href="/gateway/cli-backends" icon="terminal">
    Claude CLI backend setup and runtime details.
  </Card>

  <Card title="Prompt caching" href="/reference/prompt-caching" icon="database">
    How prompt caching works across providers.
  </Card>

  <Card title="OAuth and auth" href="/gateway/authentication" icon="key">
    Auth details and credential reuse rules.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Amazon Bedrock

OpenClaw can use **Amazon Bedrock** models via pi-ai's **Bedrock Converse**
streaming provider. Bedrock auth uses the **AWS SDK default credential chain**,
not an API key.

| Property | Value                                                       |
| -------- | ----------------------------------------------------------- |
| Provider | `amazon-bedrock`                                            |
| API      | `bedrock-converse-stream`                                   |
| Auth     | AWS credentials (env vars, shared config, or instance role) |
| Region   | `AWS_REGION` or `AWS_DEFAULT_REGION` (default: `us-east-1`) |

## Getting started

Choose your preferred auth method and follow the setup steps.

<Tabs>
  <Tab title="Access keys / env vars">
    **Best for:** developer machines, CI, or hosts where you manage AWS credentials directly.

    <Steps>
      <Step title="Set AWS credentials on the gateway host">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        export AWS_ACCESS_KEY_ID="AKIA..."
        export AWS_SECRET_ACCESS_KEY="..."
        export AWS_REGION="us-east-1"
        # Optional:
        export AWS_SESSION_TOKEN="..."
        export AWS_PROFILE="your-profile"
        # Optional (Bedrock API key/bearer token):
        export AWS_BEARER_TOKEN_BEDROCK="..."
        ```
      </Step>

      <Step title="Add a Bedrock provider and model to your config">
        No `apiKey` is required. Configure the provider with `auth: "aws-sdk"`:

        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          models: {
            providers: {
              "amazon-bedrock": {
                baseUrl: "https://bedrock-runtime.us-east-1.amazonaws.com",
                api: "bedrock-converse-stream",
                auth: "aws-sdk",
                models: [
                  {
                    id: "us.anthropic.claude-opus-4-6-v1:0",
                    name: "Claude Opus 4.6 (Bedrock)",
                    reasoning: true,
                    input: ["text", "image"],
                    cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
                    contextWindow: 200000,
                    maxTokens: 8192,
                  },
                ],
              },
            },
          },
          agents: {
            defaults: {
              model: { primary: "amazon-bedrock/us.anthropic.claude-opus-4-6-v1:0" },
            },
          },
        }
        ```
      </Step>

      <Step title="Verify models are available">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models list
        ```
      </Step>
    </Steps>

    <Tip>
      With env-marker auth (`AWS_ACCESS_KEY_ID`, `AWS_PROFILE`, or `AWS_BEARER_TOKEN_BEDROCK`), OpenClaw auto-enables the implicit Bedrock provider for model discovery without extra config.
    </Tip>
  </Tab>

  <Tab title="EC2 instance roles (IMDS)">
    **Best for:** EC2 instances with an IAM role attached, using the instance metadata service for authentication.

    <Steps>
      <Step title="Enable discovery explicitly">
        When using IMDS, OpenClaw cannot detect AWS auth from env markers alone, so you must opt in:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw config set plugins.entries.amazon-bedrock.config.discovery.enabled true
        openclaw config set plugins.entries.amazon-bedrock.config.discovery.region us-east-1
        ```
      </Step>

      <Step title="Optionally add an env marker for auto mode">
        If you also want the env-marker auto-detection path to work (for example, for `openclaw status` surfaces):

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        export AWS_PROFILE=default
        export AWS_REGION=us-east-1
        ```

        You do **not** need a fake API key.
      </Step>

      <Step title="Verify models are discovered">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models list
        ```
      </Step>
    </Steps>

    <Warning>
      The IAM role attached to your EC2 instance must have the following permissions:

      * `bedrock:InvokeModel`
      * `bedrock:InvokeModelWithResponseStream`
      * `bedrock:ListFoundationModels` (for automatic discovery)
      * `bedrock:ListInferenceProfiles` (for inference profile discovery)

      Or attach the managed policy `AmazonBedrockFullAccess`.
    </Warning>

    <Note>
      You only need `AWS_PROFILE=default` if you specifically want an env marker for auto mode or status surfaces. The actual Bedrock runtime auth path uses the AWS SDK default chain, so IMDS instance-role auth works even without env markers.
    </Note>
  </Tab>
</Tabs>

## Automatic model discovery

OpenClaw can automatically discover Bedrock models that support **streaming**
and **text output**. Discovery uses `bedrock:ListFoundationModels` and
`bedrock:ListInferenceProfiles`, and results are cached (default: 1 hour).

How the implicit provider is enabled:

* If `plugins.entries.amazon-bedrock.config.discovery.enabled` is `true`,
  OpenClaw will try discovery even when no AWS env marker is present.
* If `plugins.entries.amazon-bedrock.config.discovery.enabled` is unset,
  OpenClaw only auto-adds the
  implicit Bedrock provider when it sees one of these AWS auth markers:
  `AWS_BEARER_TOKEN_BEDROCK`, `AWS_ACCESS_KEY_ID` +
  `AWS_SECRET_ACCESS_KEY`, or `AWS_PROFILE`.
* The actual Bedrock runtime auth path still uses the AWS SDK default chain, so
  shared config, SSO, and IMDS instance-role auth can work even when discovery
  needed `enabled: true` to opt in.

<Note>
  For explicit `models.providers["amazon-bedrock"]` entries, OpenClaw can still resolve Bedrock env-marker auth early from AWS env markers such as `AWS_BEARER_TOKEN_BEDROCK` without forcing full runtime auth loading. The actual model-call auth path still uses the AWS SDK default chain.
</Note>

<AccordionGroup>
  <Accordion title="Discovery config options">
    Config options live under `plugins.entries.amazon-bedrock.config.discovery`:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      plugins: {
        entries: {
          "amazon-bedrock": {
            config: {
              discovery: {
                enabled: true,
                region: "us-east-1",
                providerFilter: ["anthropic", "amazon"],
                refreshInterval: 3600,
                defaultContextWindow: 32000,
                defaultMaxTokens: 4096,
              },
            },
          },
        },
      },
    }
    ```

    | Option                 | Default                                           | Description                                                                                                                               |
    | ---------------------- | ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
    | `enabled`              | auto                                              | In auto mode, OpenClaw only enables the implicit Bedrock provider when it sees a supported AWS env marker. Set `true` to force discovery. |
    | `region`               | `AWS_REGION` / `AWS_DEFAULT_REGION` / `us-east-1` | AWS region used for discovery API calls.                                                                                                  |
    | `providerFilter`       | (all)                                             | Matches Bedrock provider names (for example `anthropic`, `amazon`).                                                                       |
    | `refreshInterval`      | `3600`                                            | Cache duration in seconds. Set to `0` to disable caching.                                                                                 |
    | `defaultContextWindow` | `32000`                                           | Context window used for discovered models (override if you know your model limits).                                                       |
    | `defaultMaxTokens`     | `4096`                                            | Max output tokens used for discovered models (override if you know your model limits).                                                    |
  </Accordion>
</AccordionGroup>

## Quick setup (AWS path)

This walkthrough creates an IAM role, attaches Bedrock permissions, associates
the instance profile, and enables OpenClaw discovery on the EC2 host.

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
# 1. Create IAM role and instance profile
aws iam create-role --role-name EC2-Bedrock-Access \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam attach-role-policy --role-name EC2-Bedrock-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonBedrockFullAccess

aws iam create-instance-profile --instance-profile-name EC2-Bedrock-Access
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-Bedrock-Access \
  --role-name EC2-Bedrock-Access

# 2. Attach to your EC2 instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxx \
  --iam-instance-profile Name=EC2-Bedrock-Access

# 3. On the EC2 instance, enable discovery explicitly
openclaw config set plugins.entries.amazon-bedrock.config.discovery.enabled true
openclaw config set plugins.entries.amazon-bedrock.config.discovery.region us-east-1

# 4. Optional: add an env marker if you want auto mode without explicit enable
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. Verify models are discovered
openclaw models list
```

## Advanced configuration

<AccordionGroup>
  <Accordion title="Inference profiles">
    OpenClaw discovers **regional and global inference profiles** alongside
    foundation models. When a profile maps to a known foundation model, the
    profile inherits that model's capabilities (context window, max tokens,
    reasoning, vision) and the correct Bedrock request region is injected
    automatically. This means cross-region Claude profiles work without manual
    provider overrides.

    Inference profile IDs look like `us.anthropic.claude-opus-4-6-v1:0` (regional)
    or `anthropic.claude-opus-4-6-v1:0` (global). If the backing model is already
    in the discovery results, the profile inherits its full capability set;
    otherwise safe defaults apply.

    No extra configuration is needed. As long as discovery is enabled and the IAM
    principal has `bedrock:ListInferenceProfiles`, profiles appear alongside
    foundation models in `openclaw models list`.
  </Accordion>

  <Accordion title="Claude Opus 4.7 temperature">
    Bedrock rejects the `temperature` parameter for Claude Opus 4.7. OpenClaw
    omits `temperature` automatically for any Opus 4.7 Bedrock ref, including
    foundation model ids, named inference profiles, application inference
    profiles whose underlying model resolves to Opus 4.7 via
    `bedrock:GetInferenceProfile`, and dotted `opus-4.7` variants with
    optional region prefixes (`us.`, `eu.`, `ap.`, `apac.`, `au.`, `jp.`,
    `global.`). No config knob is required, and the omission applies to both
    the request options object and the `inferenceConfig` payload field.
  </Accordion>

  <Accordion title="Guardrails">
    You can apply [Amazon Bedrock Guardrails](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html)
    to all Bedrock model invocations by adding a `guardrail` object to the
    `amazon-bedrock` plugin config. Guardrails let you enforce content filtering,
    topic denial, word filters, sensitive information filters, and contextual
    grounding checks.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      plugins: {
        entries: {
          "amazon-bedrock": {
            config: {
              guardrail: {
                guardrailIdentifier: "abc123", // guardrail ID or full ARN
                guardrailVersion: "1", // version number or "DRAFT"
                streamProcessingMode: "sync", // optional: "sync" or "async"
                trace: "enabled", // optional: "enabled", "disabled", or "enabled_full"
              },
            },
          },
        },
      },
    }
    ```

    | Option                 | Required | Description                                                                                                |
    | ---------------------- | -------- | ---------------------------------------------------------------------------------------------------------- |
    | `guardrailIdentifier`  | Yes      | Guardrail ID (e.g. `abc123`) or full ARN (e.g. `arn:aws:bedrock:us-east-1:123456789012:guardrail/abc123`). |
    | `guardrailVersion`     | Yes      | Published version number, or `"DRAFT"` for the working draft.                                              |
    | `streamProcessingMode` | No       | `"sync"` or `"async"` for guardrail evaluation during streaming. If omitted, Bedrock uses its default.     |
    | `trace`                | No       | `"enabled"` or `"enabled_full"` for debugging; omit or set `"disabled"` for production.                    |

    <Warning>
      The IAM principal used by the gateway must have the `bedrock:ApplyGuardrail` permission in addition to the standard invoke permissions.
    </Warning>
  </Accordion>

  <Accordion title="Embeddings for memory search">
    Bedrock can also serve as the embedding provider for
    [memory search](/concepts/memory-search). This is configured separately from the
    inference provider -- set `agents.defaults.memorySearch.provider` to `"bedrock"`:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          memorySearch: {
            provider: "bedrock",
            model: "amazon.titan-embed-text-v2:0", // default
          },
        },
      },
    }
    ```

    Bedrock embeddings use the same AWS SDK credential chain as inference (instance
    roles, SSO, access keys, shared config, and web identity). No API key is
    needed. When `provider` is `"auto"`, Bedrock is auto-detected if that
    credential chain resolves successfully.

    Supported embedding models include Amazon Titan Embed (v1, v2), Amazon Nova
    Embed, Cohere Embed (v3, v4), and TwelveLabs Marengo. See
    [Memory configuration reference -- Bedrock](/reference/memory-config#bedrock-embedding-config)
    for the full model list and dimension options.
  </Accordion>

  <Accordion title="Notes and caveats">
    * Bedrock requires **model access** enabled in your AWS account/region.
    * Automatic discovery needs the `bedrock:ListFoundationModels` and
      `bedrock:ListInferenceProfiles` permissions.
    * If you rely on auto mode, set one of the supported AWS auth env markers on the
      gateway host. If you prefer IMDS/shared-config auth without env markers, set
      `plugins.entries.amazon-bedrock.config.discovery.enabled: true`.
    * OpenClaw surfaces the credential source in this order: `AWS_BEARER_TOKEN_BEDROCK`,
      then `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`, then `AWS_PROFILE`, then the
      default AWS SDK chain.
    * Reasoning support depends on the model; check the Bedrock model card for
      current capabilities.
    * If you prefer a managed key flow, you can also place an OpenAI-compatible
      proxy in front of Bedrock and configure it as an OpenAI provider instead.
  </Accordion>
</AccordionGroup>

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="Memory search" href="/concepts/memory-search" icon="magnifying-glass">
    Bedrock embeddings for memory search configuration.
  </Card>

  <Card title="Memory config reference" href="/reference/memory-config#bedrock-embedding-config" icon="database">
    Full Bedrock embedding model list and dimension options.
  </Card>

  <Card title="Troubleshooting" href="/help/troubleshooting" icon="wrench">
    General troubleshooting and FAQ.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Cerebras

[Cerebras](https://www.cerebras.ai) provides high-speed OpenAI-compatible inference on custom inference hardware. OpenClaw includes a bundled Cerebras provider plugin with a static four-model catalog.

| Property        | Value                                    |
| --------------- | ---------------------------------------- |
| Provider id     | `cerebras`                               |
| Plugin          | bundled, `enabledByDefault: true`        |
| Auth env var    | `CEREBRAS_API_KEY`                       |
| Onboarding flag | `--auth-choice cerebras-api-key`         |
| Direct CLI flag | `--cerebras-api-key <key>`               |
| API             | OpenAI-compatible (`openai-completions`) |
| Base URL        | `https://api.cerebras.ai/v1`             |
| Default model   | `cerebras/zai-glm-4.7`                   |

## Getting started

<Steps>
  <Step title="Get an API key">
    Create an API key in the [Cerebras Cloud Console](https://cloud.cerebras.ai).
  </Step>

  <Step title="Run onboarding">
    <CodeGroup>
      ```bash Onboarding theme={"theme":{"light":"min-light","dark":"min-dark"}}
      openclaw onboard --auth-choice cerebras-api-key
      ```

      ```bash Direct flag theme={"theme":{"light":"min-light","dark":"min-dark"}}
      openclaw onboard --non-interactive \
        --auth-choice cerebras-api-key \
        --cerebras-api-key "$CEREBRAS_API_KEY"
      ```

      ```bash Env only theme={"theme":{"light":"min-light","dark":"min-dark"}}
      export CEREBRAS_API_KEY=csk-...
      ```
    </CodeGroup>
  </Step>

  <Step title="Verify models are available">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw models list --provider cerebras
    ```

    The list should include all four bundled models. If `CEREBRAS_API_KEY` is unresolved, `openclaw models status --json` reports the missing credential under `auth.unusableProfiles`.
  </Step>
</Steps>

## Non-interactive setup

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cerebras-api-key \
  --cerebras-api-key "$CEREBRAS_API_KEY"
```

## Built-in catalog

OpenClaw ships a static Cerebras catalog that mirrors the public OpenAI-compatible endpoint. All four models share a 128k context and 8,192 max-output tokens.

| Model ref                                 | Name                 | Reasoning | Notes                                  |
| ----------------------------------------- | -------------------- | --------- | -------------------------------------- |
| `cerebras/zai-glm-4.7`                    | Z.ai GLM 4.7         | yes       | Default model; preview reasoning model |
| `cerebras/gpt-oss-120b`                   | GPT OSS 120B         | yes       | Production reasoning model             |
| `cerebras/qwen-3-235b-a22b-instruct-2507` | Qwen 3 235B Instruct | no        | Preview non-reasoning model            |
| `cerebras/llama3.1-8b`                    | Llama 3.1 8B         | no        | Production speed-focused model         |

<Warning>
  Cerebras marks `zai-glm-4.7` and `qwen-3-235b-a22b-instruct-2507` as preview models, and `llama3.1-8b` plus `qwen-3-235b-a22b-instruct-2507` are documented for deprecation on May 27, 2026. Check Cerebras' supported-models page before relying on them for production workloads.
</Warning>

## Manual config

The bundled plugin usually means you only need the API key. Use explicit `models.providers.cerebras` config when you want to override model metadata or run in `mode: "merge"` against the static catalog:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  env: { CEREBRAS_API_KEY: "csk-..." },
  agents: {
    defaults: {
      model: { primary: "cerebras/zai-glm-4.7" },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "Z.ai GLM 4.7" },
          { id: "gpt-oss-120b", name: "GPT OSS 120B" },
        ],
      },
    },
  },
}
```

<Note>
  If the Gateway runs as a daemon (launchd, systemd, Docker), make sure `CEREBRAS_API_KEY` is available to that process — for example in `~/.openclaw/.env` or through `env.shellEnv`. A key sitting only in `~/.profile` will not help a managed service unless the env is imported separately.
</Note>

## Related

<CardGroup cols={2}>
  <Card title="Model providers" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="Thinking modes" href="/tools/thinking" icon="brain">
    Reasoning effort levels for the two reasoning-capable Cerebras models.
  </Card>

  <Card title="Configuration reference" href="/gateway/config-agents#agent-defaults" icon="gear">
    Agent defaults and model configuration.
  </Card>

  <Card title="Models FAQ" href="/help/faq-models" icon="circle-question">
    Auth profiles, switching models, and resolving "no profile" errors.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# DeepSeek

[DeepSeek](https://www.deepseek.com) provides powerful AI models with an OpenAI-compatible API.

| Property | Value                      |
| -------- | -------------------------- |
| Provider | `deepseek`                 |
| Auth     | `DEEPSEEK_API_KEY`         |
| API      | OpenAI-compatible          |
| Base URL | `https://api.deepseek.com` |

## Getting started

<Steps>
  <Step title="Get your API key">
    Create an API key at [platform.deepseek.com](https://platform.deepseek.com/api_keys).
  </Step>

  <Step title="Run onboarding">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --auth-choice deepseek-api-key
    ```

    This will prompt for your API key and set `deepseek/deepseek-v4-flash` as the default model.
  </Step>

  <Step title="Verify models are available">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw models list --provider deepseek
    ```

    To inspect the bundled static catalog without requiring a running Gateway,
    use:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw models list --all --provider deepseek
    ```
  </Step>
</Steps>

<AccordionGroup>
  <Accordion title="Non-interactive setup">
    For scripted or headless installations, pass all flags directly:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice deepseek-api-key \
      --deepseek-api-key "$DEEPSEEK_API_KEY" \
      --skip-health \
      --accept-risk
    ```
  </Accordion>
</AccordionGroup>

<Warning>
  If the Gateway runs as a daemon (launchd/systemd), make sure `DEEPSEEK_API_KEY`
  is available to that process (for example, in `~/.openclaw/.env` or via
  `env.shellEnv`).
</Warning>

## Built-in catalog

| Model ref                    | Name              | Input | Context   | Max output | Notes                                      |
| ---------------------------- | ----------------- | ----- | --------- | ---------- | ------------------------------------------ |
| `deepseek/deepseek-v4-flash` | DeepSeek V4 Flash | text  | 1,000,000 | 384,000    | Default model; V4 thinking-capable surface |
| `deepseek/deepseek-v4-pro`   | DeepSeek V4 Pro   | text  | 1,000,000 | 384,000    | V4 thinking-capable surface                |
| `deepseek/deepseek-chat`     | DeepSeek Chat     | text  | 131,072   | 8,192      | DeepSeek V3.2 non-thinking surface         |
| `deepseek/deepseek-reasoner` | DeepSeek Reasoner | text  | 131,072   | 65,536     | Reasoning-enabled V3.2 surface             |

<Tip>
  V4 models support DeepSeek's `thinking` control. OpenClaw also replays
  DeepSeek `reasoning_content` on follow-up turns so thinking sessions with tool
  calls can continue.
  Use `/think xhigh` or `/think max` with DeepSeek V4 models to request DeepSeek's
  maximum `reasoning_effort`.
</Tip>

## Thinking and tools

DeepSeek V4 thinking sessions have a stricter replay contract than most
OpenAI-compatible providers: after a thinking-enabled turn uses tools, DeepSeek
expects replayed assistant messages from that turn to include
`reasoning_content` on follow-up requests. OpenClaw handles this inside the
DeepSeek plugin, so normal multi-turn tool use works with
`deepseek/deepseek-v4-flash` and `deepseek/deepseek-v4-pro`.

If you switch an existing session from another OpenAI-compatible provider to a
DeepSeek V4 model, older assistant tool-call turns may not have native
DeepSeek `reasoning_content`. OpenClaw fills that missing field on replayed
assistant messages for DeepSeek V4 thinking requests so the provider can accept
the history without requiring `/new`.

When thinking is disabled in OpenClaw (including the UI **None** selection),
OpenClaw sends DeepSeek `thinking: { type: "disabled" }` and strips replayed
`reasoning_content` from the outgoing history. This keeps disabled-thinking
sessions on the non-thinking DeepSeek path.

Use `deepseek/deepseek-v4-flash` for the default fast path. Use
`deepseek/deepseek-v4-pro` when you want the stronger V4 model and can accept
higher cost or latency.

## Live testing

The direct live model suite includes DeepSeek V4 in the modern model set. To
run only the DeepSeek V4 direct-model checks:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
OPENCLAW_LIVE_PROVIDERS=deepseek \
OPENCLAW_LIVE_MODELS="deepseek/deepseek-v4-flash,deepseek/deepseek-v4-pro" \
pnpm test:live src/agents/models.profiles.live.test.ts
```

That live check verifies both V4 models can complete and that thinking/tool
follow-up turns preserve the replay payload DeepSeek requires.

## Config example

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  env: { DEEPSEEK_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "deepseek/deepseek-v4-flash" },
    },
  },
}
```

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="Configuration reference" href="/gateway/configuration-reference" icon="gear">
    Full config reference for agents, models, and providers.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# ElevenLabs

OpenClaw uses ElevenLabs for text-to-speech, batch speech-to-text with Scribe
v2, and streaming STT with Scribe v2 Realtime.

| Capability               | OpenClaw surface                                                     | Default                  |
| ------------------------ | -------------------------------------------------------------------- | ------------------------ |
| Text-to-speech           | `messages.tts` / `talk`                                              | `eleven_multilingual_v2` |
| Batch speech-to-text     | `tools.media.audio`                                                  | `scribe_v2`              |
| Streaming speech-to-text | Voice Call streaming or Google Meet `realtime.transcriptionProvider` | `scribe_v2_realtime`     |

## Authentication

Set `ELEVENLABS_API_KEY` in the environment. `XI_API_KEY` is also accepted for
compatibility with existing ElevenLabs tooling.

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
export ELEVENLABS_API_KEY="..."
```

## Text-to-speech

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  messages: {
    tts: {
      providers: {
        elevenlabs: {
          apiKey: "${ELEVENLABS_API_KEY}",
          voiceId: "pMsXgVXv3BLzUgSXRplE",
          modelId: "eleven_multilingual_v2",
        },
      },
    },
  },
}
```

Set `modelId` to `eleven_v3` to use ElevenLabs v3 TTS. OpenClaw keeps
`eleven_multilingual_v2` as the default for existing installs.

## Speech-to-text

Use Scribe v2 for inbound audio attachments and short recorded voice segments:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "elevenlabs", model: "scribe_v2" }],
      },
    },
  },
}
```

OpenClaw sends multipart audio to ElevenLabs `/v1/speech-to-text` with
`model_id: "scribe_v2"`. Language hints map to `language_code` when present.

## Streaming STT

The bundled `elevenlabs` plugin registers Scribe v2 Realtime for Voice Call and
Google Meet agent-mode streaming transcription.

| Setting         | Config path                                                               | Default                                           |
| --------------- | ------------------------------------------------------------------------- | ------------------------------------------------- |
| API key         | `plugins.entries.voice-call.config.streaming.providers.elevenlabs.apiKey` | Falls back to `ELEVENLABS_API_KEY` / `XI_API_KEY` |
| Model           | `...elevenlabs.modelId`                                                   | `scribe_v2_realtime`                              |
| Audio format    | `...elevenlabs.audioFormat`                                               | `ulaw_8000`                                       |
| Sample rate     | `...elevenlabs.sampleRate`                                                | `8000`                                            |
| Commit strategy | `...elevenlabs.commitStrategy`                                            | `vad`                                             |
| Language        | `...elevenlabs.languageCode`                                              | (unset)                                           |

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          streaming: {
            enabled: true,
            provider: "elevenlabs",
            providers: {
              elevenlabs: {
                apiKey: "${ELEVENLABS_API_KEY}",
                audioFormat: "ulaw_8000",
                commitStrategy: "vad",
                languageCode: "en",
              },
            },
          },
        },
      },
    },
  },
}
```

<Note>
  Voice Call receives Twilio media as 8 kHz G.711 u-law. The ElevenLabs realtime
  provider defaults to `ulaw_8000`, so telephony frames can be forwarded without
  transcoding.
</Note>

For Google Meet agent mode, set
`plugins.entries.google-meet.config.realtime.transcriptionProvider` to
`"elevenlabs"` and configure the same provider block under
`plugins.entries.google-meet.config.realtime.providers.elevenlabs`.

## Related

* [Text-to-speech](/tools/tts)
* [Google Meet](/plugins/google-meet)
* [Model selection](/concepts/model-providers)
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# GitHub Copilot

GitHub Copilot is GitHub's AI coding assistant. It provides access to Copilot
models for your GitHub account and plan. OpenClaw can use Copilot as a model
provider in two different ways.

## Two ways to use Copilot in OpenClaw

<Tabs>
  <Tab title="Built-in provider (github-copilot)">
    Use the native device-login flow to obtain a GitHub token, then exchange it for
    Copilot API tokens when OpenClaw runs. This is the **default** and simplest path
    because it does not require VS Code.

    <Steps>
      <Step title="Run the login command">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models auth login-github-copilot
        ```

        You will be prompted to visit a URL and enter a one-time code. Keep the
        terminal open until it completes.
      </Step>

      <Step title="Set a default model">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models set github-copilot/claude-opus-4.7
        ```

        Or in config:

        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          agents: {
            defaults: { model: { primary: "github-copilot/claude-opus-4.7" } },
          },
        }
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="Copilot Proxy plugin (copilot-proxy)">
    Use the **Copilot Proxy** VS Code extension as a local bridge. OpenClaw talks to
    the proxy's `/v1` endpoint and uses the model list you configure there.

    <Note>
      Choose this when you already run Copilot Proxy in VS Code or need to route
      through it. You must enable the plugin and keep the VS Code extension running.
    </Note>
  </Tab>
</Tabs>

## Optional flags

| Flag            | Description                                         |
| --------------- | --------------------------------------------------- |
| `--yes`         | Skip the confirmation prompt                        |
| `--set-default` | Also apply the provider's recommended default model |

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
# Skip confirmation
openclaw models auth login-github-copilot --yes

# Login and set the default model in one step
openclaw models auth login --provider github-copilot --method device --set-default
```

## Non-interactive onboarding

If you already have a GitHub OAuth access token for Copilot, import it during
headless setup with `openclaw onboard --non-interactive`:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw onboard --non-interactive --accept-risk \
  --auth-choice github-copilot \
  --github-copilot-token "$COPILOT_GITHUB_TOKEN" \
  --skip-channels --skip-health
```

You can also omit `--auth-choice`; passing `--github-copilot-token` infers the
GitHub Copilot provider auth choice. If the flag is omitted, onboarding falls
back to `COPILOT_GITHUB_TOKEN`, `GH_TOKEN`, then `GITHUB_TOKEN`. Use
`--secret-input-mode ref` with `COPILOT_GITHUB_TOKEN` set to store an env-backed
`tokenRef` instead of plaintext in `auth-profiles.json`.

<AccordionGroup>
  <Accordion title="Interactive TTY required">
    The device-login flow requires an interactive TTY. Run it directly in a
    terminal, not in a non-interactive script or CI pipeline.
  </Accordion>

  <Accordion title="Model availability depends on your plan">
    Copilot model availability depends on your GitHub plan. If a model is
    rejected, try another ID (for example `github-copilot/gpt-4.1`).
  </Accordion>

  <Accordion title="Transport selection">
    Claude model IDs use the Anthropic Messages transport automatically. GPT,
    o-series, and Gemini models keep the OpenAI Responses transport. OpenClaw
    selects the correct transport based on the model ref.
  </Accordion>

  <Accordion title="Request compatibility">
    OpenClaw sends Copilot IDE-style request headers on Copilot transports,
    including built-in compaction, tool-result, and image follow-up turns. It
    does not enable provider-level Responses continuation for Copilot unless
    that behavior has been verified against Copilot's API.
  </Accordion>

  <Accordion title="Environment variable resolution order">
    OpenClaw resolves Copilot auth from environment variables in the following
    priority order:

    | Priority | Variable               | Notes                              |
    | -------- | ---------------------- | ---------------------------------- |
    | 1        | `COPILOT_GITHUB_TOKEN` | Highest priority, Copilot-specific |
    | 2        | `GH_TOKEN`             | GitHub CLI token (fallback)        |
    | 3        | `GITHUB_TOKEN`         | Standard GitHub token (lowest)     |

    When multiple variables are set, OpenClaw uses the highest-priority one.
    The device-login flow (`openclaw models auth login-github-copilot`) stores
    its token in the auth profile store and takes precedence over all environment
    variables.
  </Accordion>

  <Accordion title="Token storage">
    The login stores a GitHub token in the auth profile store and exchanges it
    for a Copilot API token when OpenClaw runs. You do not need to manage the
    token manually.
  </Accordion>
</AccordionGroup>

<Warning>
  The device-login command requires an interactive TTY. Use non-interactive
  onboarding when you need headless setup.
</Warning>

## Memory search embeddings

GitHub Copilot can also serve as an embedding provider for
[memory search](/concepts/memory-search). If you have a Copilot subscription and
have logged in, OpenClaw can use it for embeddings without a separate API key.

### Auto-detection

When `memorySearch.provider` is `"auto"` (the default), GitHub Copilot is tried
at priority 15 -- after local embeddings but before OpenAI and other paid
providers. If a GitHub token is available, OpenClaw discovers available
embedding models from the Copilot API and picks the best one automatically.

### Explicit config

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "github-copilot",
        // Optional: override the auto-discovered model
        model: "text-embedding-3-small",
      },
    },
  },
}
```

### How it works

1. OpenClaw resolves your GitHub token (from env vars or auth profile).
2. Exchanges it for a short-lived Copilot API token.
3. Queries the Copilot `/models` endpoint to discover available embedding models.
4. Picks the best model (prefers `text-embedding-3-small`).
5. Sends embedding requests to the Copilot `/embeddings` endpoint.

Model availability depends on your GitHub plan. If no embedding models are
available, OpenClaw skips Copilot and tries the next provider.

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="OAuth and auth" href="/gateway/authentication" icon="key">
    Auth details and credential reuse rules.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Groq

[Groq](https://groq.com) provides ultra-fast inference on open-weight models (Llama, Gemma, Kimi, Qwen, GPT OSS, and more) using custom LPU hardware. OpenClaw includes a bundled Groq plugin that registers both an OpenAI-compatible chat provider and an audio media-understanding provider.

| Property               | Value                                    |
| ---------------------- | ---------------------------------------- |
| Provider id            | `groq`                                   |
| Plugin                 | bundled, `enabledByDefault: true`        |
| Auth env var           | `GROQ_API_KEY`                           |
| Onboarding flag        | `--auth-choice groq-api-key`             |
| API                    | OpenAI-compatible (`openai-completions`) |
| Base URL               | `https://api.groq.com/openai/v1`         |
| Audio transcription    | `whisper-large-v3-turbo` (default)       |
| Suggested chat default | `groq/llama-3.3-70b-versatile`           |

## Getting started

<Steps>
  <Step title="Get an API key">
    Create an API key at [console.groq.com/keys](https://console.groq.com/keys).
  </Step>

  <Step title="Set the API key">
    <CodeGroup>
      ```bash Onboarding theme={"theme":{"light":"min-light","dark":"min-dark"}}
      openclaw onboard --auth-choice groq-api-key
      ```

      ```bash Env only theme={"theme":{"light":"min-light","dark":"min-dark"}}
      export GROQ_API_KEY=gsk_...
      ```
    </CodeGroup>
  </Step>

  <Step title="Set a default model">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          model: { primary: "groq/llama-3.3-70b-versatile" },
        },
      },
    }
    ```
  </Step>

  <Step title="Verify the catalog is reachable">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw models list --provider groq
    ```
  </Step>
</Steps>

### Config file example

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  env: { GROQ_API_KEY: "gsk_..." },
  agents: {
    defaults: {
      model: { primary: "groq/llama-3.3-70b-versatile" },
    },
  },
}
```

## Built-in catalog

OpenClaw ships a manifest-backed Groq catalog with both reasoning and non-reasoning entries. Run `openclaw models list --provider groq` to see the bundled rows for your installed version, or check [console.groq.com/docs/models](https://console.groq.com/docs/models) for Groq's authoritative list.

| Model ref                                            | Name                          | Reasoning | Input        | Context |
| ---------------------------------------------------- | ----------------------------- | --------- | ------------ | ------- |
| `groq/llama-3.3-70b-versatile`                       | Llama 3.3 70B Versatile       | no        | text         | 131,072 |
| `groq/llama-3.1-8b-instant`                          | Llama 3.1 8B Instant          | no        | text         | 131,072 |
| `groq/meta-llama/llama-4-maverick-17b-128e-instruct` | Llama 4 Maverick 17B          | no        | text + image | 131,072 |
| `groq/meta-llama/llama-4-scout-17b-16e-instruct`     | Llama 4 Scout 17B             | no        | text + image | 131,072 |
| `groq/llama3-70b-8192`                               | Llama 3 70B                   | no        | text         | 8,192   |
| `groq/llama3-8b-8192`                                | Llama 3 8B                    | no        | text         | 8,192   |
| `groq/gemma2-9b-it`                                  | Gemma 2 9B                    | no        | text         | 8,192   |
| `groq/mistral-saba-24b`                              | Mistral Saba 24B              | no        | text         | 32,768  |
| `groq/moonshotai/kimi-k2-instruct`                   | Kimi K2 Instruct              | no        | text         | 131,072 |
| `groq/moonshotai/kimi-k2-instruct-0905`              | Kimi K2 Instruct 0905         | no        | text         | 262,144 |
| `groq/openai/gpt-oss-120b`                           | GPT OSS 120B                  | yes       | text         | 131,072 |
| `groq/openai/gpt-oss-20b`                            | GPT OSS 20B                   | yes       | text         | 131,072 |
| `groq/openai/gpt-oss-safeguard-20b`                  | Safety GPT OSS 20B            | yes       | text         | 131,072 |
| `groq/qwen-qwq-32b`                                  | Qwen QwQ 32B                  | yes       | text         | 131,072 |
| `groq/qwen/qwen3-32b`                                | Qwen3 32B                     | yes       | text         | 131,072 |
| `groq/deepseek-r1-distill-llama-70b`                 | DeepSeek R1 Distill Llama 70B | yes       | text         | 131,072 |
| `groq/groq/compound`                                 | Compound                      | yes       | text         | 131,072 |
| `groq/groq/compound-mini`                            | Compound Mini                 | yes       | text         | 131,072 |

<Tip>
  The catalog evolves with each OpenClaw release. `openclaw models list --provider groq` shows the rows known to your installed version; cross-check with [console.groq.com/docs/models](https://console.groq.com/docs/models) for newly-added or deprecated models.
</Tip>

## Reasoning models

OpenClaw maps its shared `/think` levels to Groq's model-specific `reasoning_effort` values:

* For `qwen/qwen3-32b`, disabled thinking sends `none` and enabled thinking sends `default`.
* For Groq GPT OSS reasoning models (`openai/gpt-oss-*`), OpenClaw sends `low`, `medium`, or `high` based on `/think` level. Disabled thinking omits `reasoning_effort` because those models do not support a disabled value.
* DeepSeek R1 Distill, Qwen QwQ, and Compound use Groq's native reasoning surface; `/think` controls visibility but the model always reasons.

See [Thinking modes](/tools/thinking) for the shared `/think` levels and how OpenClaw translates them per provider.

## Audio transcription

Groq's bundled plugin also registers an **audio media-understanding provider** so voice messages can be transcribed through the shared `tools.media.audio` surface.

| Property           | Value                                     |
| ------------------ | ----------------------------------------- |
| Shared config path | `tools.media.audio`                       |
| Default base URL   | `https://api.groq.com/openai/v1`          |
| Default model      | `whisper-large-v3-turbo`                  |
| Auto priority      | 20                                        |
| API endpoint       | OpenAI-compatible `/audio/transcriptions` |

To make Groq the default audio backend:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  tools: {
    media: {
      audio: {
        models: [{ provider: "groq" }],
      },
    },
  },
}
```

<AccordionGroup>
  <Accordion title="Environment availability for the daemon">
    If the Gateway runs as a managed service (launchd, systemd, Docker), `GROQ_API_KEY` must be visible to that process — not just to your interactive shell.

    <Warning>
      A key sitting only in `~/.profile` will not help a launchd or systemd daemon unless that environment is imported there too. Set the key in `~/.openclaw/.env` or via `env.shellEnv` to make it readable from the gateway process.
    </Warning>
  </Accordion>

  <Accordion title="Custom Groq model ids">
    OpenClaw accepts any Groq model id at runtime. Use the exact id shown by Groq and prefix it with `groq/`. The bundled catalog covers the common cases; uncatalogued ids fall through to the default OpenAI-compatible template.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          model: { primary: "groq/<your-model-id>" },
        },
      },
    }
    ```
  </Accordion>
</AccordionGroup>

## Related

<CardGroup cols={2}>
  <Card title="Model providers" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="Thinking modes" href="/tools/thinking" icon="brain">
    Reasoning effort levels and provider-policy interaction.
  </Card>

  <Card title="Configuration reference" href="/gateway/configuration-reference" icon="gear">
    Full config schema including provider and audio settings.
  </Card>

  <Card title="Groq Console" href="https://console.groq.com" icon="arrow-up-right-from-square">
    Groq dashboard, API docs, and pricing.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# LiteLLM

[LiteLLM](https://litellm.ai) is an open-source LLM gateway that provides a unified API to 100+ model providers. Route OpenClaw through LiteLLM to get centralized cost tracking, logging, and the flexibility to switch backends without changing your OpenClaw config.

<Tip>
  **Why use LiteLLM with OpenClaw?**

  * **Cost tracking** — See exactly what OpenClaw spends across all models
  * **Model routing** — Switch between Claude, GPT-4, Gemini, Bedrock without config changes
  * **Virtual keys** — Create keys with spend limits for OpenClaw
  * **Logging** — Full request/response logs for debugging
  * **Fallbacks** — Automatic failover if your primary provider is down
</Tip>

## Quick start

<Tabs>
  <Tab title="Onboarding (recommended)">
    **Best for:** fastest path to a working LiteLLM setup.

    <Steps>
      <Step title="Run onboarding">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw onboard --auth-choice litellm-api-key
        ```

        For non-interactive setup against a remote proxy, pass the proxy URL explicitly:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw onboard --non-interactive --auth-choice litellm-api-key --litellm-api-key "$LITELLM_API_KEY" --custom-base-url "https://litellm.example/v1"
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="Manual setup">
    **Best for:** full control over installation and config.

    <Steps>
      <Step title="Start LiteLLM Proxy">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        pip install 'litellm[proxy]'
        litellm --model claude-opus-4-6
        ```
      </Step>

      <Step title="Point OpenClaw to LiteLLM">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        export LITELLM_API_KEY="your-litellm-key"

        openclaw
        ```

        That's it. OpenClaw now routes through LiteLLM.
      </Step>
    </Steps>
  </Tab>
</Tabs>

## Configuration

### Environment variables

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
export LITELLM_API_KEY="sk-litellm-key"
```

### Config file

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## Advanced configuration

### Image generation

LiteLLM can also back the `image_generate` tool through OpenAI-compatible
`/images/generations` and `/images/edits` routes. Configure a LiteLLM image
model under `agents.defaults.imageGenerationModel`:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
      },
    },
  },
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: "litellm/gpt-image-2",
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

<AccordionGroup>
  <Accordion title="Virtual keys">
    Create a dedicated key for OpenClaw with spend limits:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    curl -X POST "http://localhost:4000/key/generate" \
      -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "key_alias": "openclaw",
        "max_budget": 50.00,
        "budget_duration": "monthly"
      }'
    ```

    Use the generated key as `LITELLM_API_KEY`.
  </Accordion>

  <Accordion title="Model routing">
    LiteLLM can route model requests to different backends. Configure in your LiteLLM `config.yaml`:

    ```yaml theme={"theme":{"light":"min-light","dark":"min-dark"}}
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
  </Accordion>

  <Accordion title="Viewing usage">
    Check LiteLLM's dashboard or API:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    # Key info
    curl "http://localhost:4000/key/info" \
      -H "Authorization: Bearer sk-litellm-key"

    # Spend logs
    curl "http://localhost:4000/spend/logs" \
      -H "Authorization: Bearer $LITELLM_MASTER_KEY"
    ```
  </Accordion>

  <Accordion title="Proxy behavior notes">
    * LiteLLM runs on `http://localhost:4000` by default
    * OpenClaw connects through LiteLLM's proxy-style OpenAI-compatible `/v1`
      endpoint
    * Native OpenAI-only request shaping does not apply through LiteLLM:
      no `service_tier`, no Responses `store`, no prompt-cache hints, and no
      OpenAI reasoning-compat payload shaping
    * Hidden OpenClaw attribution headers (`originator`, `version`, `User-Agent`)
      are not injected on custom LiteLLM base URLs
  </Accordion>
</AccordionGroup>

<Note>
  For general provider configuration and failover behavior, see [Model Providers](/concepts/model-providers).
</Note>

## Related

<CardGroup cols={2}>
  <Card title="LiteLLM Docs" href="https://docs.litellm.ai" icon="book">
    Official LiteLLM documentation and API reference.
  </Card>

  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Overview of all providers, model refs, and failover behavior.
  </Card>

  <Card title="Configuration" href="/gateway/configuration" icon="gear">
    Full config reference.
  </Card>

  <Card title="Model selection" href="/concepts/models" icon="brain">
    How to choose and configure models.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Mistral

OpenClaw includes a bundled Mistral plugin that registers four contracts: chat completions, media understanding (Voxtral batch transcription), realtime STT for Voice Call (Voxtral Realtime), and memory embeddings (`mistral-embed`).

| Property         | Value                                       |
| ---------------- | ------------------------------------------- |
| Provider id      | `mistral`                                   |
| Plugin           | bundled, `enabledByDefault: true`           |
| Auth env var     | `MISTRAL_API_KEY`                           |
| Onboarding flag  | `--auth-choice mistral-api-key`             |
| Direct CLI flag  | `--mistral-api-key <key>`                   |
| API              | OpenAI-compatible (`openai-completions`)    |
| Base URL         | `https://api.mistral.ai/v1`                 |
| Default model    | `mistral/mistral-large-latest`              |
| Embedding model  | `mistral-embed`                             |
| Voxtral batch    | `voxtral-mini-latest` (audio transcription) |
| Voxtral realtime | `voxtral-mini-transcribe-realtime-2602`     |

## Getting started

<Steps>
  <Step title="Get your API key">
    Create an API key in the [Mistral Console](https://console.mistral.ai/).
  </Step>

  <Step title="Run onboarding">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --auth-choice mistral-api-key
    ```

    Or pass the key directly:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
    ```
  </Step>

  <Step title="Set a default model">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      env: { MISTRAL_API_KEY: "sk-..." },
      agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
    }
    ```
  </Step>

  <Step title="Verify the model is available">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw models list --provider mistral
    ```
  </Step>
</Steps>

## Built-in LLM catalog

OpenClaw currently ships this bundled Mistral catalog:

| Model ref                        | Input       | Context | Max output | Notes                                                            |
| -------------------------------- | ----------- | ------- | ---------- | ---------------------------------------------------------------- |
| `mistral/mistral-large-latest`   | text, image | 262,144 | 16,384     | Default model                                                    |
| `mistral/mistral-medium-2508`    | text, image | 262,144 | 8,192      | Mistral Medium 3.1                                               |
| `mistral/mistral-small-latest`   | text, image | 128,000 | 16,384     | Mistral Small 4; adjustable reasoning via API `reasoning_effort` |
| `mistral/pixtral-large-latest`   | text, image | 128,000 | 32,768     | Pixtral                                                          |
| `mistral/codestral-latest`       | text        | 256,000 | 4,096      | Coding                                                           |
| `mistral/devstral-medium-latest` | text        | 262,144 | 32,768     | Devstral 2                                                       |
| `mistral/magistral-small`        | text        | 128,000 | 40,000     | Reasoning-enabled                                                |

## Audio transcription (Voxtral)

Use Voxtral for batch audio transcription through the media understanding
pipeline.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

<Tip>
  The media transcription path uses `/v1/audio/transcriptions`. The default audio model for Mistral is `voxtral-mini-latest`.
</Tip>

## Voice Call streaming STT

The bundled `mistral` plugin registers Voxtral Realtime as a Voice Call
streaming STT provider.

| Setting      | Config path                                                            | Default                                 |
| ------------ | ---------------------------------------------------------------------- | --------------------------------------- |
| API key      | `plugins.entries.voice-call.config.streaming.providers.mistral.apiKey` | Falls back to `MISTRAL_API_KEY`         |
| Model        | `...mistral.model`                                                     | `voxtral-mini-transcribe-realtime-2602` |
| Encoding     | `...mistral.encoding`                                                  | `pcm_mulaw`                             |
| Sample rate  | `...mistral.sampleRate`                                                | `8000`                                  |
| Target delay | `...mistral.targetStreamingDelayMs`                                    | `800`                                   |

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          streaming: {
            enabled: true,
            provider: "mistral",
            providers: {
              mistral: {
                apiKey: "${MISTRAL_API_KEY}",
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

<Note>
  OpenClaw defaults Mistral realtime STT to `pcm_mulaw` at 8 kHz so Voice Call
  can forward Twilio media frames directly. Use `encoding: "pcm_s16le"` and a
  matching `sampleRate` only if your upstream stream is already raw PCM.
</Note>

## Advanced configuration

<AccordionGroup>
  <Accordion title="Adjustable reasoning (mistral-small-latest)">
    `mistral/mistral-small-latest` maps to Mistral Small 4 and supports [adjustable reasoning](https://docs.mistral.ai/capabilities/reasoning/adjustable) on the Chat Completions API via `reasoning_effort` (`none` minimizes extra thinking in the output; `high` surfaces full thinking traces before the final answer).

    OpenClaw maps the session **thinking** level to Mistral's API:

    | OpenClaw thinking level                                              | Mistral `reasoning_effort` |
    | -------------------------------------------------------------------- | -------------------------- |
    | **off** / **minimal**                                                | `none`                     |
    | **low** / **medium** / **high** / **xhigh** / **adaptive** / **max** | `high`                     |

    <Note>
      Other bundled Mistral catalog models do not use this parameter. Keep using `magistral-*` models when you want Mistral's native reasoning-first behavior.
    </Note>
  </Accordion>

  <Accordion title="Memory embeddings">
    Mistral can serve memory embeddings via `/v1/embeddings` (default model: `mistral-embed`).

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      memorySearch: { provider: "mistral" },
    }
    ```
  </Accordion>

  <Accordion title="Auth and base URL">
    * Mistral auth uses `MISTRAL_API_KEY` (Bearer header).
    * Provider base URL defaults to `https://api.mistral.ai/v1` and accepts the standard OpenAI-compatible chat-completions request shape.
    * Onboarding default model is `mistral/mistral-large-latest`.
    * Override the base URL under `models.providers.mistral.baseUrl` only when Mistral explicitly publishes a regional endpoint you need.
  </Accordion>
</AccordionGroup>

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="Media understanding" href="/nodes/media-understanding" icon="microphone">
    Audio transcription setup and provider selection.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Model provider quickstart

OpenClaw can use many LLM providers. Pick one, authenticate, then set the default
model as `provider/model`.

## Quick start (two steps)

1. Authenticate with the provider (usually via `openclaw onboard`).
2. Set the default model:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Supported providers (starter set)

* [Alibaba Model Studio](/providers/alibaba)
* [Amazon Bedrock](/providers/bedrock)
* [Anthropic (API + Claude CLI)](/providers/anthropic)
* [BytePlus (International)](/concepts/model-providers#byteplus-international)
* [Chutes](/providers/chutes)
* [ComfyUI](/providers/comfy)
* [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
* [DeepInfra](/providers/deepinfra)
* [fal](/providers/fal)
* [Fireworks](/providers/fireworks)
* [GLM models](/providers/glm)
* [MiniMax](/providers/minimax)
* [Mistral](/providers/mistral)
* [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
* [OpenAI (API + Codex)](/providers/openai)
* [OpenCode (Zen + Go)](/providers/opencode)
* [OpenRouter](/providers/openrouter)
* [Qianfan](/providers/qianfan)
* [Qwen](/providers/qwen)
* [Runway](/providers/runway)
* [StepFun](/providers/stepfun)
* [Synthetic](/providers/synthetic)
* [Vercel AI Gateway](/providers/vercel-ai-gateway)
* [Venice (Venice AI)](/providers/venice)
* [xAI](/providers/xai)
* [Z.AI](/providers/zai)

## Additional bundled provider variants

* `anthropic-vertex` - implicit Anthropic on Google Vertex support when Vertex credentials are available; no separate onboarding auth choice
* `copilot-proxy` - local VS Code Copilot Proxy bridge; use `openclaw onboard --auth-choice copilot-proxy`
* `google-gemini-cli` - unofficial Gemini CLI OAuth flow; requires a local `gemini` install (`brew install gemini-cli` or `npm install -g @google/gemini-cli`); default model `google-gemini-cli/gemini-3-flash-preview`; use `openclaw onboard --auth-choice google-gemini-cli` or `openclaw models auth login --provider google-gemini-cli --set-default`

For the full provider catalog (xAI, Groq, Mistral, etc.) and advanced configuration,
see [Model providers](/concepts/model-providers).

## Related

* [Model selection](/concepts/model-providers)
* [Model failover](/concepts/model-failover)
* [Models CLI](/cli/models)
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Moonshot AI

Moonshot provides the Kimi API with OpenAI-compatible endpoints. Configure the
provider and set the default model to `moonshot/kimi-k2.6`, or use
Kimi Coding with `kimi/kimi-code`.

<Warning>
  Moonshot and Kimi Coding are **separate providers**. Keys are not interchangeable, endpoints differ, and model refs differ (`moonshot/...` vs `kimi/...`).
</Warning>

## Built-in model catalog

[//]: # "moonshot-kimi-k2-ids:start"

| Model ref                         | Name                   | Reasoning | Input       | Context | Max output |
| --------------------------------- | ---------------------- | --------- | ----------- | ------- | ---------- |
| `moonshot/kimi-k2.6`              | Kimi K2.6              | No        | text, image | 262,144 | 262,144    |
| `moonshot/kimi-k2.5`              | Kimi K2.5              | No        | text, image | 262,144 | 262,144    |
| `moonshot/kimi-k2-thinking`       | Kimi K2 Thinking       | Yes       | text        | 262,144 | 262,144    |
| `moonshot/kimi-k2-thinking-turbo` | Kimi K2 Thinking Turbo | Yes       | text        | 262,144 | 262,144    |
| `moonshot/kimi-k2-turbo`          | Kimi K2 Turbo          | No        | text        | 256,000 | 16,384     |

[//]: # "moonshot-kimi-k2-ids:end"

Bundled cost estimates for current Moonshot-hosted K2 models use Moonshot's
published pay-as-you-go rates: Kimi K2.6 is $0.16/MTok cache hit,
$0.95/MTok input, and $4.00/MTok output; Kimi K2.5 is $0.10/MTok cache hit,
$0.60/MTok input, and $3.00/MTok output. Other legacy catalog entries keep
zero-cost placeholders unless you override them in config.

## Getting started

Choose your provider and follow the setup steps.

<Tabs>
  <Tab title="Moonshot API">
    **Best for:** Kimi K2 models via the Moonshot Open Platform.

    <Steps>
      <Step title="Choose your endpoint region">
        | Auth choice           | Endpoint                     | Region        |
        | --------------------- | ---------------------------- | ------------- |
        | `moonshot-api-key`    | `https://api.moonshot.ai/v1` | International |
        | `moonshot-api-key-cn` | `https://api.moonshot.cn/v1` | China         |
      </Step>

      <Step title="Run onboarding">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw onboard --auth-choice moonshot-api-key
        ```

        Or for the China endpoint:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw onboard --auth-choice moonshot-api-key-cn
        ```
      </Step>

      <Step title="Set a default model">
        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          agents: {
            defaults: {
              model: { primary: "moonshot/kimi-k2.6" },
            },
          },
        }
        ```
      </Step>

      <Step title="Verify models are available">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models list --provider moonshot
        ```
      </Step>

      <Step title="Run a live smoke test">
        Use an isolated state dir when you want to verify model access and cost
        tracking without touching your normal sessions:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        OPENCLAW_CONFIG_PATH=/tmp/openclaw-kimi/openclaw.json \
        OPENCLAW_STATE_DIR=/tmp/openclaw-kimi \
        openclaw agent --local \
          --session-id live-kimi-cost \
          --message 'Reply exactly: KIMI_LIVE_OK' \
          --thinking off \
          --json
        ```

        The JSON response should report `provider: "moonshot"` and
        `model: "kimi-k2.6"`. The assistant transcript entry stores normalized
        token usage plus estimated cost under `usage.cost` when Moonshot returns
        usage metadata.
      </Step>
    </Steps>

    ### Config example

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      env: { MOONSHOT_API_KEY: "sk-..." },
      agents: {
        defaults: {
          model: { primary: "moonshot/kimi-k2.6" },
          models: {
            // moonshot-kimi-k2-aliases:start
            "moonshot/kimi-k2.6": { alias: "Kimi K2.6" },
            "moonshot/kimi-k2.5": { alias: "Kimi K2.5" },
            "moonshot/kimi-k2-thinking": { alias: "Kimi K2 Thinking" },
            "moonshot/kimi-k2-thinking-turbo": { alias: "Kimi K2 Thinking Turbo" },
            "moonshot/kimi-k2-turbo": { alias: "Kimi K2 Turbo" },
            // moonshot-kimi-k2-aliases:end
          },
        },
      },
      models: {
        mode: "merge",
        providers: {
          moonshot: {
            baseUrl: "https://api.moonshot.ai/v1",
            apiKey: "${MOONSHOT_API_KEY}",
            api: "openai-completions",
            models: [
              // moonshot-kimi-k2-models:start
              {
                id: "kimi-k2.6",
                name: "Kimi K2.6",
                reasoning: false,
                input: ["text", "image"],
                cost: { input: 0.95, output: 4, cacheRead: 0.16, cacheWrite: 0 },
                contextWindow: 262144,
                maxTokens: 262144,
              },
              {
                id: "kimi-k2.5",
                name: "Kimi K2.5",
                reasoning: false,
                input: ["text", "image"],
                cost: { input: 0.6, output: 3, cacheRead: 0.1, cacheWrite: 0 },
                contextWindow: 262144,
                maxTokens: 262144,
              },
              {
                id: "kimi-k2-thinking",
                name: "Kimi K2 Thinking",
                reasoning: true,
                input: ["text"],
                cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
                contextWindow: 262144,
                maxTokens: 262144,
              },
              {
                id: "kimi-k2-thinking-turbo",
                name: "Kimi K2 Thinking Turbo",
                reasoning: true,
                input: ["text"],
                cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
                contextWindow: 262144,
                maxTokens: 262144,
              },
              {
                id: "kimi-k2-turbo",
                name: "Kimi K2 Turbo",
                reasoning: false,
                input: ["text"],
                cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
                contextWindow: 256000,
                maxTokens: 16384,
              },
              // moonshot-kimi-k2-models:end
            ],
          },
        },
      },
    }
    ```
  </Tab>

  <Tab title="Kimi Coding">
    **Best for:** code-focused tasks via the Kimi Coding endpoint.

    <Note>
      Kimi Coding uses a different API key and provider prefix (`kimi/...`) than Moonshot (`moonshot/...`). Legacy model ref `kimi/k2p5` remains accepted as a compatibility id.
    </Note>

    <Steps>
      <Step title="Run onboarding">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw onboard --auth-choice kimi-code-api-key
        ```
      </Step>

      <Step title="Set a default model">
        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          agents: {
            defaults: {
              model: { primary: "kimi/kimi-code" },
            },
          },
        }
        ```
      </Step>

      <Step title="Verify the model is available">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models list --provider kimi
        ```
      </Step>
    </Steps>

    ### Config example

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      env: { KIMI_API_KEY: "sk-..." },
      agents: {
        defaults: {
          model: { primary: "kimi/kimi-code" },
          models: {
            "kimi/kimi-code": { alias: "Kimi" },
          },
        },
      },
    }
    ```
  </Tab>
</Tabs>

## Kimi web search

OpenClaw also ships **Kimi** as a `web_search` provider, backed by Moonshot web
search.

<Steps>
  <Step title="Run interactive web search setup">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw configure --section web
    ```

    Choose **Kimi** in the web-search section to store
    `plugins.entries.moonshot.config.webSearch.*`.
  </Step>

  <Step title="Configure the web search region and model">
    Interactive setup prompts for:

    | Setting          | Options                                                                              |
    | ---------------- | ------------------------------------------------------------------------------------ |
    | API region       | `https://api.moonshot.ai/v1` (international) or `https://api.moonshot.cn/v1` (China) |
    | Web search model | Defaults to `kimi-k2.6`                                                              |
  </Step>
</Steps>

Config lives under `plugins.entries.moonshot.config.webSearch`:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  plugins: {
    entries: {
      moonshot: {
        config: {
          webSearch: {
            apiKey: "sk-...", // or use KIMI_API_KEY / MOONSHOT_API_KEY
            baseUrl: "https://api.moonshot.ai/v1",
            model: "kimi-k2.6",
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "kimi",
      },
    },
  },
}
```

## Advanced configuration

<AccordionGroup>
  <Accordion title="Native thinking mode">
    Moonshot Kimi supports binary native thinking:

    * `thinking: { type: "enabled" }`
    * `thinking: { type: "disabled" }`

    Configure it per model via `agents.defaults.models.<provider/model>.params`:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          models: {
            "moonshot/kimi-k2.6": {
              params: {
                thinking: { type: "disabled" },
              },
            },
          },
        },
      },
    }
    ```

    OpenClaw also maps runtime `/think` levels for Moonshot:

    | `/think` level    | Moonshot behavior        |
    | ----------------- | ------------------------ |
    | `/think off`      | `thinking.type=disabled` |
    | Any non-off level | `thinking.type=enabled`  |

    <Warning>
      When Moonshot thinking is enabled, `tool_choice` must be `auto` or `none`. OpenClaw normalizes incompatible `tool_choice` values to `auto` for compatibility.
    </Warning>

    Kimi K2.6 also accepts an optional `thinking.keep` field that controls
    multi-turn retention of `reasoning_content`. Set it to `"all"` to keep full
    reasoning across turns; omit it (or leave it `null`) to use the server
    default strategy. OpenClaw only forwards `thinking.keep` for
    `moonshot/kimi-k2.6` and strips it from other models.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          models: {
            "moonshot/kimi-k2.6": {
              params: {
                thinking: { type: "enabled", keep: "all" },
              },
            },
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Tool call id sanitization">
    Moonshot Kimi serves tool\_call ids shaped like `functions.<name>:<index>`. OpenClaw preserves them unchanged so multi-turn tool calls keep working.

    To force strict sanitization on a custom OpenAI-compatible provider, set `sanitizeToolCallIds: true`:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          "my-kimi-proxy": {
            api: "openai-completions",
            sanitizeToolCallIds: true,
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Streaming usage compatibility">
    Native Moonshot endpoints (`https://api.moonshot.ai/v1` and
    `https://api.moonshot.cn/v1`) advertise streaming usage compatibility on the
    shared `openai-completions` transport. OpenClaw keys that off endpoint
    capabilities, so compatible custom provider ids targeting the same native
    Moonshot hosts inherit the same streaming-usage behavior.

    With the bundled K2.6 pricing, streamed usage that includes input, output,
    and cache-read tokens is also converted into local estimated USD cost for
    `/status`, `/usage full`, `/usage cost`, and transcript-backed session
    accounting.
  </Accordion>

  <Accordion title="Endpoint and model ref reference">
    | Provider    | Model ref prefix | Endpoint                     | Auth env var                         |
    | ----------- | ---------------- | ---------------------------- | ------------------------------------ |
    | Moonshot    | `moonshot/`      | `https://api.moonshot.ai/v1` | `MOONSHOT_API_KEY`                   |
    | Moonshot CN | `moonshot/`      | `https://api.moonshot.cn/v1` | `MOONSHOT_API_KEY`                   |
    | Kimi Coding | `kimi/`          | Kimi Coding endpoint         | `KIMI_API_KEY`                       |
    | Web search  | N/A              | Same as Moonshot API region  | `KIMI_API_KEY` or `MOONSHOT_API_KEY` |

    * Kimi web search uses `KIMI_API_KEY` or `MOONSHOT_API_KEY`, and defaults to `https://api.moonshot.ai/v1` with model `kimi-k2.6`.
    * Override pricing and context metadata in `models.providers` if needed.
    * If Moonshot publishes different context limits for a model, adjust `contextWindow` accordingly.
  </Accordion>
</AccordionGroup>

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="Web search" href="/tools/web" icon="magnifying-glass">
    Configuring web search providers including Kimi.
  </Card>

  <Card title="Configuration reference" href="/gateway/configuration-reference" icon="gear">
    Full config schema for providers, models, and plugins.
  </Card>

  <Card title="Moonshot Open Platform" href="https://platform.moonshot.ai" icon="globe">
    Moonshot API key management and documentation.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# NVIDIA

NVIDIA provides an OpenAI-compatible API at `https://integrate.api.nvidia.com/v1` for
open models for free. Authenticate with an API key from
[build.nvidia.com](https://build.nvidia.com/settings/api-keys).

## Getting started

<Steps>
  <Step title="Get your API key">
    Create an API key at [build.nvidia.com](https://build.nvidia.com/settings/api-keys).
  </Step>

  <Step title="Export the key and run onboarding">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    export NVIDIA_API_KEY="nvapi-..."
    openclaw onboard --auth-choice nvidia-api-key
    ```
  </Step>

  <Step title="Set an NVIDIA model">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw models set nvidia/nvidia/nemotron-3-super-120b-a12b
    ```
  </Step>
</Steps>

<Warning>
  If you pass `--nvidia-api-key` instead of the env var, the value lands in shell
  history and `ps` output. Prefer the `NVIDIA_API_KEY` environment variable when
  possible.
</Warning>

For non-interactive setup, you can also pass the key directly:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw onboard --auth-choice nvidia-api-key --nvidia-api-key "nvapi-..."
```

## Config example

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/nemotron-3-super-120b-a12b" },
    },
  },
}
```

## Built-in catalog

| Model ref                                  | Name                         | Context | Max output |
| ------------------------------------------ | ---------------------------- | ------- | ---------- |
| `nvidia/nvidia/nemotron-3-super-120b-a12b` | NVIDIA Nemotron 3 Super 120B | 262,144 | 8,192      |
| `nvidia/moonshotai/kimi-k2.5`              | Kimi K2.5                    | 262,144 | 8,192      |
| `nvidia/minimaxai/minimax-m2.5`            | Minimax M2.5                 | 196,608 | 8,192      |
| `nvidia/z-ai/glm5`                         | GLM 5                        | 202,752 | 8,192      |

## Advanced configuration

<AccordionGroup>
  <Accordion title="Auto-enable behavior">
    The provider auto-enables when the `NVIDIA_API_KEY` environment variable is set.
    No explicit provider config is required beyond the key.
  </Accordion>

  <Accordion title="Catalog and pricing">
    The bundled catalog is static. Costs default to `0` in source since NVIDIA
    currently offers free API access for the listed models.
  </Accordion>

  <Accordion title="OpenAI-compatible endpoint">
    NVIDIA uses the standard `/v1` completions endpoint. Any OpenAI-compatible
    tooling should work out of the box with the NVIDIA base URL.
  </Accordion>
</AccordionGroup>

<Tip>
  NVIDIA models are currently free to use. Check
  [build.nvidia.com](https://build.nvidia.com/) for the latest availability and
  rate-limit details.
</Tip>

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="Configuration reference" href="/gateway/configuration-reference" icon="gear">
    Full config reference for agents, models, and providers.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Ollama

OpenClaw integrates with Ollama's native API (`/api/chat`) for hosted cloud models and local/self-hosted Ollama servers. You can use Ollama in three modes: `Cloud + Local` through a reachable Ollama host, `Cloud only` against `https://ollama.com`, or `Local only` against a reachable Ollama host.

<Warning>
  **Remote Ollama users**: Do not use the `/v1` OpenAI-compatible URL (`http://host:11434/v1`) with OpenClaw. This breaks tool calling and models may output raw tool JSON as plain text. Use the native Ollama API URL instead: `baseUrl: "http://host:11434"` (no `/v1`).
</Warning>

Ollama provider config uses `baseUrl` as the canonical key. OpenClaw also accepts `baseURL` for compatibility with OpenAI SDK-style examples, but new config should prefer `baseUrl`.

## Auth rules

<AccordionGroup>
  <Accordion title="Local and LAN hosts">
    Local and LAN Ollama hosts do not need a real bearer token. OpenClaw uses the local `ollama-local` marker only for loopback, private-network, `.local`, and bare-hostname Ollama base URLs.
  </Accordion>

  <Accordion title="Remote and Ollama Cloud hosts">
    Remote public hosts and Ollama Cloud (`https://ollama.com`) require a real credential through `OLLAMA_API_KEY`, an auth profile, or the provider's `apiKey`.
  </Accordion>

  <Accordion title="Custom provider ids">
    Custom provider ids that set `api: "ollama"` follow the same rules. For example, an `ollama-remote` provider that points at a private LAN Ollama host can use `apiKey: "ollama-local"` and sub-agents will resolve that marker through the Ollama provider hook instead of treating it as a missing credential. Memory search can also set `agents.defaults.memorySearch.provider` to that custom provider id so embeddings use the matching Ollama endpoint.
  </Accordion>

  <Accordion title="Auth profiles">
    `auth-profiles.json` stores the credential for a provider id. Put endpoint settings (`baseUrl`, `api`, model ids, headers, timeouts) in `models.providers.<id>`. Older flat auth-profile files such as `{ "ollama-windows": { "apiKey": "ollama-local" } }` are not a runtime format; run `openclaw doctor --fix` to rewrite them to the canonical `ollama-windows:default` API-key profile with a backup. `baseUrl` in that file is compatibility noise and should be moved to provider config.
  </Accordion>

  <Accordion title="Memory embedding scope">
    When Ollama is used for memory embeddings, bearer auth is scoped to the host where it was declared:

    * A provider-level key is sent only to that provider's Ollama host.
    * `agents.*.memorySearch.remote.apiKey` is sent only to its remote embedding host.
    * A pure `OLLAMA_API_KEY` env value is treated as the Ollama Cloud convention, not sent to local or self-hosted hosts by default.
  </Accordion>
</AccordionGroup>

## Getting started

Choose your preferred setup method and mode.

<Tabs>
  <Tab title="Onboarding (recommended)">
    **Best for:** fastest path to a working Ollama cloud or local setup.

    <Steps>
      <Step title="Run onboarding">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw onboard
        ```

        Select **Ollama** from the provider list.
      </Step>

      <Step title="Choose your mode">
        * **Cloud + Local** — local Ollama host plus cloud models routed through that host
        * **Cloud only** — hosted Ollama models via `https://ollama.com`
        * **Local only** — local models only
      </Step>

      <Step title="Select a model">
        `Cloud only` prompts for `OLLAMA_API_KEY` and suggests hosted cloud defaults. `Cloud + Local` and `Local only` ask for an Ollama base URL, discover available models, and auto-pull the selected local model if it is not available yet. When Ollama reports an installed `:latest` tag such as `gemma4:latest`, setup shows that installed model once instead of showing both `gemma4` and `gemma4:latest` or pulling the bare alias again. `Cloud + Local` also checks whether that Ollama host is signed in for cloud access.
      </Step>

      <Step title="Verify the model is available">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models list --provider ollama
        ```
      </Step>
    </Steps>

    ### Non-interactive mode

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --non-interactive \
      --auth-choice ollama \
      --accept-risk
    ```

    Optionally specify a custom base URL or model:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --non-interactive \
      --auth-choice ollama \
      --custom-base-url "http://ollama-host:11434" \
      --custom-model-id "qwen3.5:27b" \
      --accept-risk
    ```
  </Tab>

  <Tab title="Manual setup">
    **Best for:** full control over cloud or local setup.

    <Steps>
      <Step title="Choose cloud or local">
        * **Cloud + Local**: install Ollama, sign in with `ollama signin`, and route cloud requests through that host
        * **Cloud only**: use `https://ollama.com` with an `OLLAMA_API_KEY`
        * **Local only**: install Ollama from [ollama.com/download](https://ollama.com/download)
      </Step>

      <Step title="Pull a local model (local only)">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        ollama pull gemma4
        # or
        ollama pull gpt-oss:20b
        # or
        ollama pull llama3.3
        ```
      </Step>

      <Step title="Enable Ollama for OpenClaw">
        For `Cloud only`, use your real `OLLAMA_API_KEY`. For host-backed setups, any placeholder value works:

        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        # Cloud
        export OLLAMA_API_KEY="your-ollama-api-key"

        # Local-only
        export OLLAMA_API_KEY="ollama-local"

        # Or configure in your config file
        openclaw config set models.providers.ollama.apiKey "OLLAMA_API_KEY"
        ```
      </Step>

      <Step title="Inspect and set your model">
        ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
        openclaw models list
        openclaw models set ollama/gemma4
        ```

        Or set the default in config:

        ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
        {
          agents: {
            defaults: {
              model: { primary: "ollama/gemma4" },
            },
          },
        }
        ```
      </Step>
    </Steps>
  </Tab>
</Tabs>

## Cloud models

<Tabs>
  <Tab title="Cloud + Local">
    `Cloud + Local` uses a reachable Ollama host as the control point for both local and cloud models. This is Ollama's preferred hybrid flow.

    Use **Cloud + Local** during setup. OpenClaw prompts for the Ollama base URL, discovers local models from that host, and checks whether the host is signed in for cloud access with `ollama signin`. When the host is signed in, OpenClaw also suggests hosted cloud defaults such as `kimi-k2.5:cloud`, `minimax-m2.7:cloud`, and `glm-5.1:cloud`.

    If the host is not signed in yet, OpenClaw keeps the setup local-only until you run `ollama signin`.
  </Tab>

  <Tab title="Cloud only">
    `Cloud only` runs against Ollama's hosted API at `https://ollama.com`.

    Use **Cloud only** during setup. OpenClaw prompts for `OLLAMA_API_KEY`, sets `baseUrl: "https://ollama.com"`, and seeds the hosted cloud model list. This path does **not** require a local Ollama server or `ollama signin`.

    The cloud model list shown during `openclaw onboard` is populated live from `https://ollama.com/api/tags`, capped at 500 entries, so the picker reflects the current hosted catalog rather than a static seed. If `ollama.com` is unreachable or returns no models at setup time, OpenClaw falls back to the previous hardcoded suggestions so onboarding still completes.
  </Tab>

  <Tab title="Local only">
    In local-only mode, OpenClaw discovers models from the configured Ollama instance. This path is for local or self-hosted Ollama servers.

    OpenClaw currently suggests `gemma4` as the local default.
  </Tab>
</Tabs>

## Model discovery (implicit provider)

When you set `OLLAMA_API_KEY` (or an auth profile) and **do not** define `models.providers.ollama` or another custom remote provider with `api: "ollama"`, OpenClaw discovers models from the local Ollama instance at `http://127.0.0.1:11434`.

| Behavior             | Detail                                                                                                                                                               |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Catalog query        | Queries `/api/tags`                                                                                                                                                  |
| Capability detection | Uses best-effort `/api/show` lookups to read `contextWindow`, expanded `num_ctx` Modelfile parameters, and capabilities including vision/tools                       |
| Vision models        | Models with a `vision` capability reported by `/api/show` are marked as image-capable (`input: ["text", "image"]`), so OpenClaw auto-injects images into the prompt  |
| Reasoning detection  | Uses `/api/show` capabilities when available, including `thinking`; falls back to a model-name heuristic (`r1`, `reasoning`, `think`) when Ollama omits capabilities |
| Token limits         | Sets `maxTokens` to the default Ollama max-token cap used by OpenClaw                                                                                                |
| Costs                | Sets all costs to `0`                                                                                                                                                |

This avoids manual model entries while keeping the catalog aligned with the local Ollama instance. You can use a full ref such as `ollama/<pulled-model>:latest` in local `infer model run`; OpenClaw resolves that installed model from Ollama's live catalog without requiring a hand-written `models.json` entry.

For signed-in Ollama hosts, some `:cloud` models may be usable through `/api/chat`
and `/api/show` before they appear in `/api/tags`. When you explicitly select a
full `ollama/<model>:cloud` ref, OpenClaw validates that exact missing model with
`/api/show` and adds it to the runtime catalog only if Ollama confirms model
metadata. Typos still fail as unknown models instead of being auto-created.

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
# See what models are available
ollama list
openclaw models list
```

For a narrow text-generation smoke test that avoids the full agent tool surface,
use local `infer model run` with a full Ollama model ref:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
OLLAMA_API_KEY=ollama-local \
  openclaw infer model run \
    --local \
    --model ollama/llama3.2:latest \
    --prompt "Reply with exactly: pong" \
    --json
```

That path still uses OpenClaw's configured provider, auth, and native Ollama
transport, but it does not start a chat-agent turn or load MCP/tool context. If
this succeeds while normal agent replies fail, troubleshoot the model's agent
prompt/tool capacity next.

For a narrow vision-model smoke test on the same lean path, add one or more
image files to `infer model run`. This sends the prompt and image directly to
the selected Ollama vision model without loading chat tools, memory, or prior
session context:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
OLLAMA_API_KEY=ollama-local \
  openclaw infer model run \
    --local \
    --model ollama/qwen2.5vl:7b \
    --prompt "Describe this image in one sentence." \
    --file ./photo.jpg \
    --json
```

`model run --file` accepts files detected as `image/*`, including common PNG,
JPEG, and WebP inputs. Non-image files are rejected before Ollama is called.
For speech recognition, use `openclaw infer audio transcribe` instead.

When you switch a conversation with `/model ollama/<model>`, OpenClaw treats
that as an exact user selection. If the configured Ollama `baseUrl` is
unreachable, the next reply fails with the provider error instead of silently
answering from another configured fallback model.

Isolated cron jobs do one extra local safety check before they start the agent
turn. If the selected model resolves to a local, private-network, or `.local`
Ollama provider and `/api/tags` is unreachable, OpenClaw records that cron run
as `skipped` with the selected `ollama/<model>` in the error text. The endpoint
preflight is cached for 5 minutes, so multiple cron jobs pointed at the same
stopped Ollama daemon do not all launch failing model requests.

Live-verify the local text path, native stream path, and embeddings against
local Ollama with:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_OLLAMA=1 OPENCLAW_LIVE_OLLAMA_WEB_SEARCH=0 \
  pnpm test:live -- extensions/ollama/ollama.live.test.ts
```

To add a new model, simply pull it with Ollama:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
ollama pull mistral
```

The new model will be automatically discovered and available to use.

<Note>
  If you set `models.providers.ollama` explicitly, or configure a custom remote provider such as `models.providers.ollama-cloud` with `api: "ollama"`, auto-discovery is skipped and you must define models manually. Loopback custom providers such as `http://127.0.0.2:11434` are still treated as local. See the explicit config section below.
</Note>

## Vision and image description

The bundled Ollama plugin registers Ollama as an image-capable media-understanding provider. This lets OpenClaw route explicit image-description requests and configured image-model defaults through local or hosted Ollama vision models.

For local vision, pull a model that supports images:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
ollama pull qwen2.5vl:7b
export OLLAMA_API_KEY="ollama-local"
```

Then verify with the infer CLI:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw infer image describe \
  --file ./photo.jpg \
  --model ollama/qwen2.5vl:7b \
  --json
```

`--model` must be a full `<provider/model>` ref. When it is set, `openclaw infer image describe` runs that model directly instead of skipping description because the model supports native vision.

Use `infer image describe` when you want OpenClaw's image-understanding provider flow, configured `agents.defaults.imageModel`, and image-description output shape. Use `infer model run --file` when you want a raw multimodal model probe with a custom prompt and one or more images.

To make Ollama the default image-understanding model for inbound media, configure `agents.defaults.imageModel`:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    defaults: {
      imageModel: {
        primary: "ollama/qwen2.5vl:7b",
      },
    },
  },
}
```

Prefer the full `ollama/<model>` ref. If the same model is listed under `models.providers.ollama.models` with `input: ["text", "image"]` and no other configured image provider exposes that bare model ID, OpenClaw also normalizes a bare `imageModel` ref such as `qwen2.5vl:7b` to `ollama/qwen2.5vl:7b`. If more than one configured image provider has the same bare ID, use the provider prefix explicitly.

Slow local vision models can need a longer image-understanding timeout than cloud models. They can also crash or stop when Ollama tries to allocate the full advertised vision context on constrained hardware. Set a capability timeout, and cap `num_ctx` on the model entry when you only need a normal image-description turn:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  models: {
    providers: {
      ollama: {
        models: [
          {
            id: "qwen2.5vl:7b",
            name: "qwen2.5vl:7b",
            input: ["text", "image"],
            params: { num_ctx: 2048, keep_alive: "1m" },
          },
        ],
      },
    },
  },
  tools: {
    media: {
      image: {
        timeoutSeconds: 180,
        models: [{ provider: "ollama", model: "qwen2.5vl:7b", timeoutSeconds: 300 }],
      },
    },
  },
}
```

This timeout applies to inbound image understanding and to the explicit `image` tool the agent can call during a turn. Provider-level `models.providers.ollama.timeoutSeconds` still controls the underlying Ollama HTTP request guard for normal model calls.

Live-verify the explicit image tool against local Ollama with:

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_OLLAMA_IMAGE=1 \
  pnpm test:live -- src/agents/tools/image-tool.ollama.live.test.ts
```

If you define `models.providers.ollama.models` manually, mark vision models with image input support:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  id: "qwen2.5vl:7b",
  name: "qwen2.5vl:7b",
  input: ["text", "image"],
  contextWindow: 128000,
  maxTokens: 8192,
}
```

OpenClaw rejects image-description requests for models that are not marked image-capable. With implicit discovery, OpenClaw reads this from Ollama when `/api/show` reports a vision capability.

## Configuration

<Tabs>
  <Tab title="Basic (implicit discovery)">
    The simplest local-only enablement path is via environment variable:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    export OLLAMA_API_KEY="ollama-local"
    ```

    <Tip>
      If `OLLAMA_API_KEY` is set, you can omit `apiKey` in the provider entry and OpenClaw will fill it for availability checks.
    </Tip>
  </Tab>

  <Tab title="Explicit (manual models)">
    Use explicit config when you want hosted cloud setup, Ollama runs on another host/port, you want to force specific context windows or model lists, or you want fully manual model definitions.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            baseUrl: "https://ollama.com",
            apiKey: "OLLAMA_API_KEY",
            api: "ollama",
            models: [
              {
                id: "kimi-k2.5:cloud",
                name: "kimi-k2.5:cloud",
                reasoning: false,
                input: ["text", "image"],
                cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
                contextWindow: 128000,
                maxTokens: 8192
              }
            ]
          }
        }
      }
    }
    ```
  </Tab>

  <Tab title="Custom base URL">
    If Ollama is running on a different host or port (explicit config disables auto-discovery, so define models manually):

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            apiKey: "ollama-local",
            baseUrl: "http://ollama-host:11434", // No /v1 - use native Ollama API URL
            api: "ollama", // Set explicitly to guarantee native tool-calling behavior
            timeoutSeconds: 300, // Optional: give cold local models longer to connect and stream
            models: [
              {
                id: "qwen3:32b",
                name: "qwen3:32b",
                params: {
                  keep_alive: "15m", // Optional: keep the model loaded between turns
                },
              },
            ],
          },
        },
      },
    }
    ```

    <Warning>
      Do not add `/v1` to the URL. The `/v1` path uses OpenAI-compatible mode, where tool calling is not reliable. Use the base Ollama URL without a path suffix.
    </Warning>
  </Tab>
</Tabs>

## Common recipes

Use these as starting points and replace model IDs with the exact names from `ollama list` or `openclaw models list --provider ollama`.

<AccordionGroup>
  <Accordion title="Local model with auto-discovery">
    Use this when Ollama runs on the same machine as the Gateway and you want OpenClaw to discover the installed models automatically.

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    ollama serve
    ollama pull gemma4
    export OLLAMA_API_KEY="ollama-local"
    openclaw models list --provider ollama
    openclaw models set ollama/gemma4
    ```

    This path keeps config minimal. Do not add a `models.providers.ollama` block unless you want to define models manually.
  </Accordion>

  <Accordion title="LAN Ollama host with manual models">
    Use native Ollama URLs for LAN hosts. Do not add `/v1`.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            baseUrl: "http://gpu-box.local:11434",
            apiKey: "ollama-local",
            api: "ollama",
            timeoutSeconds: 300,
            contextWindow: 32768,
            maxTokens: 8192,
            models: [
              {
                id: "qwen3.5:9b",
                name: "qwen3.5:9b",
                reasoning: true,
                input: ["text"],
                params: {
                  num_ctx: 32768,
                  thinking: false,
                  keep_alive: "15m",
                },
              },
            ],
          },
        },
      },
      agents: {
        defaults: {
          model: { primary: "ollama/qwen3.5:9b" },
        },
      },
    }
    ```

    `contextWindow` is the OpenClaw-side context budget. `params.num_ctx` is sent to Ollama for the request. Keep them aligned when your hardware cannot run the model's full advertised context.
  </Accordion>

  <Accordion title="Ollama Cloud only">
    Use this when you do not run a local daemon and want hosted Ollama models directly.

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    export OLLAMA_API_KEY="your-ollama-api-key"
    ```

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            baseUrl: "https://ollama.com",
            apiKey: "OLLAMA_API_KEY",
            api: "ollama",
            models: [
              {
                id: "kimi-k2.5:cloud",
                name: "kimi-k2.5:cloud",
                reasoning: false,
                input: ["text", "image"],
                contextWindow: 128000,
                maxTokens: 8192,
              },
            ],
          },
        },
      },
      agents: {
        defaults: {
          model: { primary: "ollama/kimi-k2.5:cloud" },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Cloud plus local through a signed-in daemon">
    Use this when a local or LAN Ollama daemon is signed in with `ollama signin` and should serve both local models and `:cloud` models.

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    ollama signin
    ollama pull gemma4
    ```

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            baseUrl: "http://127.0.0.1:11434",
            apiKey: "ollama-local",
            api: "ollama",
            timeoutSeconds: 300,
            models: [
              { id: "gemma4", name: "gemma4", input: ["text"] },
              { id: "kimi-k2.5:cloud", name: "kimi-k2.5:cloud", input: ["text", "image"] },
            ],
          },
        },
      },
      agents: {
        defaults: {
          model: {
            primary: "ollama/gemma4",
            fallbacks: ["ollama/kimi-k2.5:cloud"],
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Multiple Ollama hosts">
    Use custom provider IDs when you have more than one Ollama server. Each provider gets its own host, models, auth, timeout, and model refs.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          "ollama-fast": {
            baseUrl: "http://mini.local:11434",
            apiKey: "ollama-local",
            api: "ollama",
            contextWindow: 32768,
            models: [{ id: "gemma4", name: "gemma4", input: ["text"] }],
          },
          "ollama-large": {
            baseUrl: "http://gpu-box.local:11434",
            apiKey: "ollama-local",
            api: "ollama",
            timeoutSeconds: 420,
            contextWindow: 131072,
            maxTokens: 16384,
            models: [{ id: "qwen3.5:27b", name: "qwen3.5:27b", input: ["text"] }],
          },
        },
      },
      agents: {
        defaults: {
          model: {
            primary: "ollama-fast/gemma4",
            fallbacks: ["ollama-large/qwen3.5:27b"],
          },
        },
      },
    }
    ```

    When OpenClaw sends the request, the active provider prefix is stripped so `ollama-large/qwen3.5:27b` reaches Ollama as `qwen3.5:27b`.
  </Accordion>

  <Accordion title="Lean local model profile">
    Some local models can answer simple prompts but struggle with the full agent tool surface. Start by limiting tools and context before changing global runtime settings.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          experimental: {
            localModelLean: true,
          },
          model: { primary: "ollama/gemma4" },
        },
      },
      models: {
        providers: {
          ollama: {
            baseUrl: "http://127.0.0.1:11434",
            apiKey: "ollama-local",
            api: "ollama",
            contextWindow: 32768,
            models: [
              {
                id: "gemma4",
                name: "gemma4",
                input: ["text"],
                params: { num_ctx: 32768 },
                compat: { supportsTools: false },
              },
            ],
          },
        },
      },
    }
    ```

    Use `compat.supportsTools: false` only when the model or server reliably fails on tool schemas. It trades agent capability for stability.
    `localModelLean` removes the browser, cron, and message tools from the agent surface, but it does not change Ollama's runtime context or thinking mode. Pair it with explicit `params.num_ctx` and `params.thinking: false` for small Qwen-style thinking models that loop or spend their response budget on hidden reasoning.
  </Accordion>
</AccordionGroup>

### Model selection

Once configured, all your Ollama models are available:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

Custom Ollama provider ids are also supported. When a model ref uses the active
provider prefix, such as `ollama-spark/qwen3:32b`, OpenClaw strips only that
prefix before calling Ollama so the server receives `qwen3:32b`.

For slow local models, prefer provider-scoped request tuning before raising the
whole agent runtime timeout:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  models: {
    providers: {
      ollama: {
        timeoutSeconds: 300,
        models: [
          {
            id: "gemma4:26b",
            name: "gemma4:26b",
            params: { keep_alive: "15m" },
          },
        ],
      },
    },
  },
}
```

`timeoutSeconds` applies to the model HTTP request, including connection setup,
headers, body streaming, and the total guarded-fetch abort. `params.keep_alive`
is forwarded to Ollama as top-level `keep_alive` on native `/api/chat` requests;
set it per model when first-turn load time is the bottleneck.

### Quick verification

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
# Ollama daemon visible to this machine
curl http://127.0.0.1:11434/api/tags

# OpenClaw catalog and selected model
openclaw models list --provider ollama
openclaw models status

# Direct model smoke
openclaw infer model run \
  --model ollama/gemma4 \
  --prompt "Reply with exactly: ok"
```

For remote hosts, replace `127.0.0.1` with the host used in `baseUrl`. If `curl` works but OpenClaw does not, check whether the Gateway runs on a different machine, container, or service account.

## Ollama Web Search

OpenClaw supports **Ollama Web Search** as a bundled `web_search` provider.

| Property    | Detail                                                                                                                                                               |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Host        | Uses your configured Ollama host (`models.providers.ollama.baseUrl` when set, otherwise `http://127.0.0.1:11434`); `https://ollama.com` uses the hosted API directly |
| Auth        | Key-free for signed-in local Ollama hosts; `OLLAMA_API_KEY` or configured provider auth for direct `https://ollama.com` search or auth-protected hosts               |
| Requirement | Local/self-hosted hosts must be running and signed in with `ollama signin`; direct hosted search requires `baseUrl: "https://ollama.com"` plus a real Ollama API key |

Choose **Ollama Web Search** during `openclaw onboard` or `openclaw configure --section web`, or set:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  tools: {
    web: {
      search: {
        provider: "ollama",
      },
    },
  },
}
```

For direct hosted search through Ollama Cloud:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  models: {
    providers: {
      ollama: {
        baseUrl: "https://ollama.com",
        apiKey: "OLLAMA_API_KEY",
        api: "ollama",
        models: [{ id: "kimi-k2.5:cloud", name: "kimi-k2.5:cloud", input: ["text"] }],
      },
    },
  },
  tools: {
    web: {
      search: { provider: "ollama" },
    },
  },
}
```

For a signed-in local daemon, OpenClaw uses the daemon's `/api/experimental/web_search` proxy. For `https://ollama.com`, it calls the hosted `/api/web_search` endpoint directly.

<Note>
  For the full setup and behavior details, see [Ollama Web Search](/tools/ollama-search).
</Note>

## Advanced configuration

<AccordionGroup>
  <Accordion title="Legacy OpenAI-compatible mode">
    <Warning>
      **Tool calling is not reliable in OpenAI-compatible mode.** Use this mode only if you need OpenAI format for a proxy and do not depend on native tool calling behavior.
    </Warning>

    If you need to use the OpenAI-compatible endpoint instead (for example, behind a proxy that only supports OpenAI format), set `api: "openai-completions"` explicitly:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            baseUrl: "http://ollama-host:11434/v1",
            api: "openai-completions",
            injectNumCtxForOpenAICompat: true, // default: true
            apiKey: "ollama-local",
            models: [...]
          }
        }
      }
    }
    ```

    This mode may not support streaming and tool calling simultaneously. You may need to disable streaming with `params: { streaming: false }` in model config.

    When `api: "openai-completions"` is used with Ollama, OpenClaw injects `options.num_ctx` by default so Ollama does not silently fall back to a 4096 context window. If your proxy/upstream rejects unknown `options` fields, disable this behavior:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            baseUrl: "http://ollama-host:11434/v1",
            api: "openai-completions",
            injectNumCtxForOpenAICompat: false,
            apiKey: "ollama-local",
            models: [...]
          }
        }
      }
    }
    ```
  </Accordion>

  <Accordion title="Context windows">
    For auto-discovered models, OpenClaw uses the context window reported by Ollama when available, including larger `PARAMETER num_ctx` values from custom Modelfiles. Otherwise it falls back to the default Ollama context window used by OpenClaw.

    You can set provider-level `contextWindow`, `contextTokens`, and `maxTokens` defaults for every model under that Ollama provider, then override them per model when needed. `contextWindow` is OpenClaw's prompt and compaction budget. Native Ollama requests leave `options.num_ctx` unset unless you explicitly configure `params.num_ctx`, so Ollama can apply its own model, `OLLAMA_CONTEXT_LENGTH`, or VRAM-based default. To cap or force Ollama's per-request runtime context without rebuilding a Modelfile, set `params.num_ctx`; invalid, zero, negative, and non-finite values are ignored. The OpenAI-compatible Ollama adapter still injects `options.num_ctx` by default from the configured `params.num_ctx` or `contextWindow`; disable that with `injectNumCtxForOpenAICompat: false` if your upstream rejects `options`.

    Native Ollama model entries also accept the common Ollama runtime options under `params`, including `temperature`, `top_p`, `top_k`, `min_p`, `num_predict`, `stop`, `repeat_penalty`, `num_batch`, `num_thread`, and `use_mmap`. OpenClaw forwards only Ollama request keys, so OpenClaw runtime params such as `streaming` are not leaked to Ollama. Use `params.think` or `params.thinking` to send top-level Ollama `think`; `false` disables API-level thinking for Qwen-style thinking models.

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            contextWindow: 32768,
            models: [
              {
                id: "llama3.3",
                contextWindow: 131072,
                maxTokens: 65536,
                params: {
                  num_ctx: 32768,
                  temperature: 0.7,
                  top_p: 0.9,
                  thinking: false,
                },
              }
            ]
          }
        }
      }
    }
    ```

    Per-model `agents.defaults.models["ollama/<model>"].params.num_ctx` works too. If both are configured, the explicit provider model entry wins over the agent default.
  </Accordion>

  <Accordion title="Thinking control">
    For native Ollama models, OpenClaw forwards thinking control as Ollama expects it: top-level `think`, not `options.think`. Auto-discovered models whose `/api/show` response includes the `thinking` capability expose `/think low`, `/think medium`, `/think high`, and `/think max`; non-thinking models expose only `/think off`.

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw agent --model ollama/gemma4 --thinking off
    openclaw agent --model ollama/gemma4 --thinking low
    ```

    You can also set a model default:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          models: {
            "ollama/gemma4": {
              thinking: "low",
            },
          },
        },
      },
    }
    ```

    Per-model `params.think` or `params.thinking` can disable or force Ollama API thinking for a specific configured model. OpenClaw preserves those explicit model params when the active run only has the implicit default `off`; non-off runtime commands such as `/think medium` still override the active run.
  </Accordion>

  <Accordion title="Reasoning models">
    OpenClaw treats models with names such as `deepseek-r1`, `reasoning`, or `think` as reasoning-capable by default.

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    ollama pull deepseek-r1:32b
    ```

    No additional configuration is needed. OpenClaw marks them automatically.
  </Accordion>

  <Accordion title="Model costs">
    Ollama is free and runs locally, so all model costs are set to \$0. This applies to both auto-discovered and manually defined models.
  </Accordion>

  <Accordion title="Memory embeddings">
    The bundled Ollama plugin registers a memory embedding provider for
    [memory search](/concepts/memory). It uses the configured Ollama base URL
    and API key, calls Ollama's current `/api/embed` endpoint, and batches
    multiple memory chunks into one `input` request when possible.

    | Property      | Value                                                                    |
    | ------------- | ------------------------------------------------------------------------ |
    | Default model | `nomic-embed-text`                                                       |
    | Auto-pull     | Yes — the embedding model is pulled automatically if not present locally |

    Query-time embeddings use retrieval prefixes for models that require or recommend them, including `nomic-embed-text`, `qwen3-embedding`, and `mxbai-embed-large`. Memory document batches stay raw so existing indexes do not need a format migration.

    To select Ollama as the memory search embedding provider:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          memorySearch: {
            provider: "ollama",
            remote: {
              // Default for Ollama. Raise on larger hosts if reindexing is too slow.
              nonBatchConcurrency: 1,
            },
          },
        },
      },
    }
    ```

    For a remote embedding host, keep auth scoped to that host:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          memorySearch: {
            provider: "ollama",
            model: "nomic-embed-text",
            remote: {
              baseUrl: "http://gpu-box.local:11434",
              apiKey: "ollama-local",
              nonBatchConcurrency: 2,
            },
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Streaming configuration">
    OpenClaw's Ollama integration uses the **native Ollama API** (`/api/chat`) by default, which fully supports streaming and tool calling simultaneously. No special configuration is needed.

    For native `/api/chat` requests, OpenClaw also forwards thinking control directly to Ollama: `/think off` and `openclaw agent --thinking off` send top-level `think: false` unless an explicit model `params.think`/`params.thinking` value is configured, while `/think low|medium|high` send the matching top-level `think` effort string. `/think max` maps to Ollama's highest native effort, `think: "high"`.

    <Tip>
      If you need to use the OpenAI-compatible endpoint, see the "Legacy OpenAI-compatible mode" section above. Streaming and tool calling may not work simultaneously in that mode.
    </Tip>
  </Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
  <Accordion title="WSL2 crash loop (repeated reboots)">
    On WSL2 with NVIDIA/CUDA, the official Ollama Linux installer creates an `ollama.service` systemd unit with `Restart=always`. If that service autostarts and loads a GPU-backed model during WSL2 boot, Ollama can pin host memory while the model loads. Hyper-V memory reclaim cannot always reclaim those pinned pages, so Windows can terminate the WSL2 VM, systemd starts Ollama again, and the loop repeats.

    Common evidence:

    * repeated WSL2 reboots or terminations from the Windows side
    * high CPU in `app.slice` or `ollama.service` shortly after WSL2 startup
    * SIGTERM from systemd rather than a Linux OOM-killer event

    OpenClaw logs a startup warning when it detects WSL2, `ollama.service` enabled with `Restart=always`, and visible CUDA markers.

    Mitigation:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    sudo systemctl disable ollama
    ```

    Add this to `%USERPROFILE%\.wslconfig` on the Windows side, then run `wsl --shutdown`:

    ```ini theme={"theme":{"light":"min-light","dark":"min-dark"}}
    [experimental]
    autoMemoryReclaim=disabled
    ```

    Set a shorter keep-alive in the Ollama service environment, or start Ollama manually only when you need it:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    export OLLAMA_KEEP_ALIVE=5m
    ollama serve
    ```

    See [ollama/ollama#11317](https://github.com/ollama/ollama/issues/11317).
  </Accordion>

  <Accordion title="Ollama not detected">
    Make sure Ollama is running and that you set `OLLAMA_API_KEY` (or an auth profile), and that you did **not** define an explicit `models.providers.ollama` entry:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    ollama serve
    ```

    Verify that the API is accessible:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    curl http://localhost:11434/api/tags
    ```
  </Accordion>

  <Accordion title="No models available">
    If your model is not listed, either pull the model locally or define it explicitly in `models.providers.ollama`.

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    ollama list  # See what's installed
    ollama pull gemma4
    ollama pull gpt-oss:20b
    ollama pull llama3.3     # Or another model
    ```
  </Accordion>

  <Accordion title="Connection refused">
    Check that Ollama is running on the correct port:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    # Check if Ollama is running
    ps aux | grep ollama

    # Or restart Ollama
    ollama serve
    ```
  </Accordion>

  <Accordion title="Remote host works with curl but not OpenClaw">
    Verify from the same machine and runtime that runs the Gateway:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw gateway status --deep
    curl http://ollama-host:11434/api/tags
    ```

    Common causes:

    * `baseUrl` points at `localhost`, but the Gateway runs in Docker or on another host.
    * The URL uses `/v1`, which selects OpenAI-compatible behavior instead of native Ollama.
    * The remote host needs firewall or LAN binding changes on the Ollama side.
    * The model is present on your laptop's daemon but not on the remote daemon.
  </Accordion>

  <Accordion title="Model outputs tool JSON as text">
    This usually means the provider is using OpenAI-compatible mode or the model cannot handle tool schemas.

    Prefer native Ollama mode:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            baseUrl: "http://ollama-host:11434",
            api: "ollama",
          },
        },
      },
    }
    ```

    If a small local model still fails on tool schemas, set `compat.supportsTools: false` on that model entry and retest.
  </Accordion>

  <Accordion title="Kimi or GLM returns garbled symbols">
    Hosted Kimi/GLM responses that are long, non-linguistic symbol runs are treated as failed provider output instead of a successful assistant answer. That lets normal retry, fallback, or error handling take over without persisting the corrupted text into the session.

    If it happens repeatedly, capture the raw model name, the current session file, and whether the run used `Cloud + Local` or `Cloud only`, then try a fresh session and a fallback model:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw infer model run --model ollama/kimi-k2.5:cloud --prompt "Reply with exactly: ok" --json
    openclaw models set ollama/gemma4
    ```
  </Accordion>

  <Accordion title="Cold local model times out">
    Large local models can need a long first load before streaming begins. Keep the timeout scoped to the Ollama provider, and optionally ask Ollama to keep the model loaded between turns:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            timeoutSeconds: 300,
            models: [
              {
                id: "gemma4:26b",
                name: "gemma4:26b",
                params: { keep_alive: "15m" },
              },
            ],
          },
        },
      },
    }
    ```

    If the host itself is slow to accept connections, `timeoutSeconds` also extends the guarded Undici connect timeout for this provider.
  </Accordion>

  <Accordion title="Large-context model is too slow or runs out of memory">
    Many Ollama models advertise contexts that are larger than your hardware can run comfortably. Native Ollama uses Ollama's own runtime context default unless you set `params.num_ctx`. Cap both OpenClaw's budget and Ollama's request context when you want predictable first-token latency:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      models: {
        providers: {
          ollama: {
            contextWindow: 32768,
            maxTokens: 8192,
            models: [
              {
                id: "qwen3.5:9b",
                name: "qwen3.5:9b",
                params: { num_ctx: 32768, thinking: false },
              },
            ],
          },
        },
      },
    }
    ```

    Lower `contextWindow` first if OpenClaw is sending too much prompt. Lower `params.num_ctx` if Ollama is loading a runtime context that is too large for the machine. Lower `maxTokens` if generation runs too long.
  </Accordion>
</AccordionGroup>

<Note>
  More help: [Troubleshooting](/help/troubleshooting) and [FAQ](/help/faq).
</Note>

## Related

<CardGroup cols={2}>
  <Card title="Model providers" href="/concepts/model-providers" icon="layers">
    Overview of all providers, model refs, and failover behavior.
  </Card>

  <Card title="Model selection" href="/concepts/models" icon="brain">
    How to choose and configure models.
  </Card>

  <Card title="Ollama Web Search" href="/tools/ollama-search" icon="magnifying-glass">
    Full setup and behavior details for Ollama-powered web search.
  </Card>

  <Card title="Configuration" href="/gateway/configuration" icon="gear">
    Full config reference.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# OpenRouter

OpenRouter provides a **unified API** that routes requests to many models behind a single
endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

## Getting started

<Steps>
  <Step title="Get your API key">
    Create an API key at [openrouter.ai/keys](https://openrouter.ai/keys).
  </Step>

  <Step title="Run onboarding">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --auth-choice openrouter-api-key
    ```
  </Step>

  <Step title="(Optional) Switch to a specific model">
    Onboarding defaults to `openrouter/auto`. Pick a concrete model later:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw models set openrouter/<provider>/<model>
    ```
  </Step>
</Steps>

## Config example

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/auto" },
    },
  },
}
```

## Model references

<Note>
  Model refs follow the pattern `openrouter/<provider>/<model>`. For the full list of
  available providers and models, see [/concepts/model-providers](/concepts/model-providers).
</Note>

Bundled fallback examples:

| Model ref                         | Notes                        |
| --------------------------------- | ---------------------------- |
| `openrouter/auto`                 | OpenRouter automatic routing |
| `openrouter/moonshotai/kimi-k2.6` | Kimi K2.6 via MoonshotAI     |

## Image generation

OpenRouter can also back the `image_generate` tool. Use an OpenRouter image model under `agents.defaults.imageGenerationModel`:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: "openrouter/google/gemini-3.1-flash-image-preview",
        timeoutMs: 180_000,
      },
    },
  },
}
```

OpenClaw sends image requests to OpenRouter's chat completions image API with `modalities: ["image", "text"]`. Gemini image models receive supported `aspectRatio` and `resolution` hints through OpenRouter's `image_config`. Use `agents.defaults.imageGenerationModel.timeoutMs` for slower OpenRouter image models; the `image_generate` tool's per-call `timeoutMs` parameter still wins.

## Video generation

OpenRouter can also back the `video_generate` tool through its asynchronous `/videos` API. Use an OpenRouter video model under `agents.defaults.videoGenerationModel`:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      videoGenerationModel: {
        primary: "openrouter/google/veo-3.1-fast",
      },
    },
  },
}
```

OpenClaw submits text-to-video and image-to-video jobs to OpenRouter, polls
the returned `polling_url`, and downloads the completed video from
OpenRouter's `unsigned_urls` or the documented job content endpoint.
Reference images are sent as first/last frame images by default; images
tagged with `reference_image` are sent as OpenRouter input references. The
bundled `google/veo-3.1-fast` default advertises the currently supported 4/6/8
second durations, `720P`/`1080P` resolutions, and `16:9`/`9:16` aspect
ratios. Video-to-video is not registered for OpenRouter because the upstream
video generation API currently accepts text and image references.

## Text-to-speech

OpenRouter can also be used as a TTS provider through its OpenAI-compatible
`/audio/speech` endpoint.

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  messages: {
    tts: {
      auto: "always",
      provider: "openrouter",
      providers: {
        openrouter: {
          model: "hexgrad/kokoro-82m",
          voice: "af_alloy",
          responseFormat: "mp3",
        },
      },
    },
  },
}
```

If `messages.tts.providers.openrouter.apiKey` is omitted, TTS reuses
`models.providers.openrouter.apiKey`, then `OPENROUTER_API_KEY`.

## Authentication and headers

OpenRouter uses a Bearer token with your API key under the hood.

On real OpenRouter requests (`https://openrouter.ai/api/v1`), OpenClaw also adds
OpenRouter's documented app-attribution headers:

| Header                    | Value                                                                                                  |
| ------------------------- | ------------------------------------------------------------------------------------------------------ |
| `HTTP-Referer`            | `https://openclaw.ai`                                                                                  |
| `X-OpenRouter-Title`      | `OpenClaw`                                                                                             |
| `X-OpenRouter-Categories` | `cli-agent,cloud-agent,programming-app,creative-writing,writing-assistant,general-chat,personal-agent` |

<Warning>
  If you repoint the OpenRouter provider at some other proxy or base URL, OpenClaw
  does **not** inject those OpenRouter-specific headers or Anthropic cache markers.
</Warning>

## Advanced configuration

<AccordionGroup>
  <Accordion title="Response caching">
    OpenRouter response caching is opt-in. Enable it per OpenRouter model with
    model params:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          models: {
            "openrouter/auto": {
              params: {
                responseCache: true,
                responseCacheTtlSeconds: 300,
              },
            },
          },
        },
      },
    }
    ```

    OpenClaw sends `X-OpenRouter-Cache: true` and, when configured,
    `X-OpenRouter-Cache-TTL`. `responseCacheClear: true` forces a refresh for
    the current request and stores the replacement response. Snake\_case aliases
    (`response_cache`, `response_cache_ttl_seconds`, and
    `response_cache_clear`) are also accepted.

    This is separate from provider prompt caching and from OpenRouter's
    Anthropic `cache_control` markers. It is only applied on verified
    `openrouter.ai` routes, not custom proxy base URLs.
  </Accordion>

  <Accordion title="Anthropic cache markers">
    On verified OpenRouter routes, Anthropic model refs keep the
    OpenRouter-specific Anthropic `cache_control` markers that OpenClaw uses for
    better prompt-cache reuse on system/developer prompt blocks.
  </Accordion>

  <Accordion title="Anthropic reasoning prefill">
    On verified OpenRouter routes, Anthropic model refs with reasoning enabled
    drop trailing assistant prefill turns before the request reaches OpenRouter,
    matching Anthropic's requirement that reasoning conversations end with a user
    turn.
  </Accordion>

  <Accordion title="Thinking / reasoning injection">
    On supported non-`auto` routes, OpenClaw maps the selected thinking level to
    OpenRouter proxy reasoning payloads. Unsupported model hints and
    `openrouter/auto` skip that reasoning injection. Hunter Alpha also skips
    proxy reasoning for stale configured model refs because OpenRouter could
    return final answer text in reasoning fields for that retired route.
  </Accordion>

  <Accordion title="DeepSeek V4 reasoning replay">
    On verified OpenRouter routes, `openrouter/deepseek/deepseek-v4-flash` and
    `openrouter/deepseek/deepseek-v4-pro` fill missing `reasoning_content` on
    replayed assistant turns so thinking/tool conversations keep DeepSeek V4's
    required follow-up shape. OpenClaw sends OpenRouter-supported
    `reasoning_effort` values for these routes; `xhigh` is the highest advertised
    level, and stale `max` overrides are mapped to `xhigh`.
  </Accordion>

  <Accordion title="OpenAI-only request shaping">
    OpenRouter still runs through the proxy-style OpenAI-compatible path, so
    native OpenAI-only request shaping such as `serviceTier`, Responses `store`,
    OpenAI reasoning-compat payloads, and prompt-cache hints is not forwarded.
  </Accordion>

  <Accordion title="Gemini-backed routes">
    Gemini-backed OpenRouter refs stay on the proxy-Gemini path: OpenClaw keeps
    Gemini thought-signature sanitation there, but does not enable native Gemini
    replay validation or bootstrap rewrites.
  </Accordion>

  <Accordion title="Provider routing metadata">
    If you pass OpenRouter provider routing under model params, OpenClaw forwards
    it as OpenRouter routing metadata before the shared stream wrappers run.
  </Accordion>
</AccordionGroup>

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="Configuration reference" href="/gateway/configuration-reference" icon="gear">
    Full config reference for agents, models, and providers.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Perplexity

The Perplexity plugin provides web search capabilities through the Perplexity
Search API or Perplexity Sonar via OpenRouter.

<Note>
  This page is the Perplexity **provider** setup. For the Perplexity **tool** (how the agent uses it), see [Perplexity tool](/tools/perplexity-search).
</Note>

| Property    | Value                                                                  |
| ----------- | ---------------------------------------------------------------------- |
| Type        | Web search provider (not a model provider)                             |
| Auth        | `PERPLEXITY_API_KEY` (direct) or `OPENROUTER_API_KEY` (via OpenRouter) |
| Config path | `plugins.entries.perplexity.config.webSearch.apiKey`                   |

## Getting started

<Steps>
  <Step title="Set the API key">
    Run the interactive web-search configuration flow:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw configure --section web
    ```

    Or set the key directly:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw config set plugins.entries.perplexity.config.webSearch.apiKey "pplx-xxxxxxxxxxxx"
    ```
  </Step>

  <Step title="Start searching">
    The agent will automatically use Perplexity for web searches once the key is
    configured. No additional steps are required.
  </Step>
</Steps>

## Search modes

The plugin auto-selects the transport based on API key prefix:

<Tabs>
  <Tab title="Native Perplexity API (pplx-)">
    When your key starts with `pplx-`, OpenClaw uses the native Perplexity Search
    API. This transport returns structured results and supports domain, language,
    and date filters (see filtering options below).
  </Tab>

  <Tab title="OpenRouter / Sonar (sk-or-)">
    When your key starts with `sk-or-`, OpenClaw routes through OpenRouter using
    the Perplexity Sonar model. This transport returns AI-synthesized answers with
    citations.
  </Tab>
</Tabs>

| Key prefix | Transport                    | Features                                         |
| ---------- | ---------------------------- | ------------------------------------------------ |
| `pplx-`    | Native Perplexity Search API | Structured results, domain/language/date filters |
| `sk-or-`   | OpenRouter (Sonar)           | AI-synthesized answers with citations            |

## Native API filtering

<Note>
  Filtering options are only available when using the native Perplexity API
  (`pplx-` key). OpenRouter/Sonar searches do not support these parameters.
</Note>

When using the native Perplexity API, searches support the following filters:

| Filter         | Description                            | Example                             |
| -------------- | -------------------------------------- | ----------------------------------- |
| Country        | 2-letter country code                  | `us`, `de`, `jp`                    |
| Language       | ISO 639-1 language code                | `en`, `fr`, `zh`                    |
| Date range     | Recency window                         | `day`, `week`, `month`, `year`      |
| Domain filters | Allowlist or denylist (max 20 domains) | `example.com`                       |
| Content budget | Token limits per response / per page   | `max_tokens`, `max_tokens_per_page` |

## Advanced configuration

<AccordionGroup>
  <Accordion title="Environment variable for daemon processes">
    If the OpenClaw Gateway runs as a daemon (launchd/systemd), make sure
    `PERPLEXITY_API_KEY` is available to that process.

    <Warning>
      A key set only in `~/.profile` will not be visible to a launchd/systemd
      daemon unless that environment is explicitly imported. Set the key in
      `~/.openclaw/.env` or via `env.shellEnv` to ensure the gateway process can
      read it.
    </Warning>
  </Accordion>

  <Accordion title="OpenRouter proxy setup">
    If you prefer to route Perplexity searches through OpenRouter, set an
    `OPENROUTER_API_KEY` (prefix `sk-or-`) instead of a native Perplexity key.
    OpenClaw will detect the prefix and switch to the Sonar transport
    automatically.

    <Tip>
      The OpenRouter transport is useful if you already have an OpenRouter account
      and want consolidated billing across multiple providers.
    </Tip>
  </Accordion>
</AccordionGroup>

## Related

<CardGroup cols={2}>
  <Card title="Perplexity search tool" href="/tools/perplexity-search" icon="magnifying-glass">
    How the agent invokes Perplexity searches and interprets results.
  </Card>

  <Card title="Configuration reference" href="/gateway/configuration-reference" icon="gear">
    Full configuration reference including plugin entries.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# Together AI

[Together AI](https://together.ai) provides access to leading open-source
models including Llama, DeepSeek, Kimi, and more through a unified API.

| Property | Value                         |
| -------- | ----------------------------- |
| Provider | `together`                    |
| Auth     | `TOGETHER_API_KEY`            |
| API      | OpenAI-compatible             |
| Base URL | `https://api.together.xyz/v1` |

## Getting started

<Steps>
  <Step title="Get an API key">
    Create an API key at
    [api.together.ai/settings/api-keys](https://api.together.ai/settings/api-keys).
  </Step>

  <Step title="Run onboarding">
    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --auth-choice together-api-key
    ```
  </Step>

  <Step title="Set a default model">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          model: { primary: "together/moonshotai/Kimi-K2.5" },
        },
      },
    }
    ```
  </Step>
</Steps>

### Non-interactive example

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

<Note>
  The onboarding preset sets `together/moonshotai/Kimi-K2.5` as the default
  model.
</Note>

## Built-in catalog

OpenClaw ships this bundled Together catalog:

| Model ref                                                    | Name                                   | Input       | Context    | Notes                            |
| ------------------------------------------------------------ | -------------------------------------- | ----------- | ---------- | -------------------------------- |
| `together/moonshotai/Kimi-K2.5`                              | Kimi K2.5                              | text, image | 262,144    | Default model; reasoning enabled |
| `together/zai-org/GLM-4.7`                                   | GLM 4.7 Fp8                            | text        | 202,752    | General-purpose text model       |
| `together/meta-llama/Llama-3.3-70B-Instruct-Turbo`           | Llama 3.3 70B Instruct Turbo           | text        | 131,072    | Fast instruction model           |
| `together/meta-llama/Llama-4-Scout-17B-16E-Instruct`         | Llama 4 Scout 17B 16E Instruct         | text, image | 10,000,000 | Multimodal                       |
| `together/meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | Llama 4 Maverick 17B 128E Instruct FP8 | text, image | 20,000,000 | Multimodal                       |
| `together/deepseek-ai/DeepSeek-V3.1`                         | DeepSeek V3.1                          | text        | 131,072    | General text model               |
| `together/deepseek-ai/DeepSeek-R1`                           | DeepSeek R1                            | text        | 131,072    | Reasoning model                  |
| `together/moonshotai/Kimi-K2-Instruct-0905`                  | Kimi K2-Instruct 0905                  | text        | 262,144    | Secondary Kimi text model        |

## Video generation

The bundled `together` plugin also registers video generation through the
shared `video_generate` tool.

| Property             | Value                                 |
| -------------------- | ------------------------------------- |
| Default video model  | `together/Wan-AI/Wan2.2-T2V-A14B`     |
| Modes                | text-to-video, single-image reference |
| Supported parameters | `aspectRatio`, `resolution`           |

To use Together as the default video provider:

```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
{
  agents: {
    defaults: {
      videoGenerationModel: {
        primary: "together/Wan-AI/Wan2.2-T2V-A14B",
      },
    },
  },
}
```

<Tip>
  See [Video Generation](/tools/video-generation) for the shared tool parameters,
  provider selection, and failover behavior.
</Tip>

<AccordionGroup>
  <Accordion title="Environment note">
    If the Gateway runs as a daemon (launchd/systemd), make sure
    `TOGETHER_API_KEY` is available to that process (for example, in
    `~/.openclaw/.env` or via `env.shellEnv`).

    <Warning>
      Keys set only in your interactive shell are not visible to daemon-managed
      gateway processes. Use `~/.openclaw/.env` or `env.shellEnv` config for
      persistent availability.
    </Warning>
  </Accordion>

  <Accordion title="Troubleshooting">
    * Verify your key works: `openclaw models list --provider together`
    * If models are not appearing, confirm the API key is set in the correct
      environment for your Gateway process.
    * Model refs use the form `together/<model-id>`.
  </Accordion>
</AccordionGroup>

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Provider rules, model refs, and failover behavior.
  </Card>

  <Card title="Video generation" href="/tools/video-generation" icon="video">
    Shared video generation tool parameters and provider selection.
  </Card>

  <Card title="Configuration reference" href="/gateway/configuration-reference" icon="gear">
    Full config schema including provider settings.
  </Card>

  <Card title="Together AI" href="https://together.ai" icon="arrow-up-right-from-square">
    Together AI dashboard, API docs, and pricing.
  </Card>
</CardGroup>
> ## Documentation Index
> Fetch the complete documentation index at: https://docs.openclaw.ai/llms.txt
> Use this file to discover all available pages before exploring further.

# xAI

OpenClaw ships a bundled `xai` provider plugin for Grok models.

## Getting started

<Steps>
  <Step title="Create an API key">
    Create an API key in the [xAI console](https://console.x.ai/).
  </Step>

  <Step title="Set your API key">
    Set `XAI_API_KEY`, or run:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw onboard --auth-choice xai-api-key
    ```
  </Step>

  <Step title="Pick a model">
    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: { defaults: { model: { primary: "xai/grok-4.3" } } },
    }
    ```
  </Step>
</Steps>

<Note>
  OpenClaw uses the xAI Responses API as the bundled xAI transport. The same
  `XAI_API_KEY` can also power Grok-backed `web_search`, first-class `x_search`,
  and remote `code_execution`.
  If you store an xAI key under `plugins.entries.xai.config.webSearch.apiKey`,
  the bundled xAI model provider reuses that key as a fallback too.
  Set `plugins.entries.xai.config.webSearch.baseUrl` to route Grok `web_search`
  and, by default, `x_search` through an operator xAI Responses proxy.
  `code_execution` tuning lives under `plugins.entries.xai.config.codeExecution`.
</Note>

## Built-in catalog

OpenClaw includes these xAI model families out of the box:

| Family         | Model ids                                                                |
| -------------- | ------------------------------------------------------------------------ |
| Grok 3         | `grok-3`, `grok-3-fast`, `grok-3-mini`, `grok-3-mini-fast`               |
| Grok 4.3       | `grok-4.3`                                                               |
| Grok 4         | `grok-4`, `grok-4-0709`                                                  |
| Grok 4 Fast    | `grok-4-fast`, `grok-4-fast-non-reasoning`                               |
| Grok 4.1 Fast  | `grok-4-1-fast`, `grok-4-1-fast-non-reasoning`                           |
| Grok 4.20 Beta | `grok-4.20-beta-latest-reasoning`, `grok-4.20-beta-latest-non-reasoning` |
| Grok Code      | `grok-code-fast-1`                                                       |

The plugin also forward-resolves newer `grok-4*` and `grok-code-fast*` ids when
they follow the same API shape.

<Tip>
  `grok-4.3`, `grok-4-fast`, `grok-4-1-fast`, and the `grok-4.20-beta-*`
  variants are the current image-capable Grok refs in the bundled catalog.
</Tip>

## OpenClaw feature coverage

The bundled plugin maps xAI's current public API surface onto OpenClaw's shared
provider and tool contracts. Capabilities that don't fit the shared contract
(for example streaming TTS and realtime voice) are not exposed - see the table
below.

| xAI capability             | OpenClaw surface                          | Status                                                              |
| -------------------------- | ----------------------------------------- | ------------------------------------------------------------------- |
| Chat / Responses           | `xai/<model>` model provider              | Yes                                                                 |
| Server-side web search     | `web_search` provider `grok`              | Yes                                                                 |
| Server-side X search       | `x_search` tool                           | Yes                                                                 |
| Server-side code execution | `code_execution` tool                     | Yes                                                                 |
| Images                     | `image_generate`                          | Yes                                                                 |
| Videos                     | `video_generate`                          | Yes                                                                 |
| Batch text-to-speech       | `messages.tts.provider: "xai"` / `tts`    | Yes                                                                 |
| Streaming TTS              | -                                         | Not exposed; OpenClaw's TTS contract returns complete audio buffers |
| Batch speech-to-text       | `tools.media.audio` / media understanding | Yes                                                                 |
| Streaming speech-to-text   | Voice Call `streaming.provider: "xai"`    | Yes                                                                 |
| Realtime voice             | -                                         | Not exposed yet; different session/WebSocket contract               |
| Files / batches            | Generic model API compatibility only      | Not a first-class OpenClaw tool                                     |

<Note>
  OpenClaw uses xAI's REST image/video/TTS/STT APIs for media generation,
  speech, and batch transcription, xAI's streaming STT WebSocket for live
  voice-call transcription, and the Responses API for model, search, and
  code-execution tools. Features that need different OpenClaw contracts, such as
  Realtime voice sessions, are documented here as upstream capabilities rather
  than hidden plugin behavior.
</Note>

### Fast-mode mappings

`/fast on` or `agents.defaults.models["xai/<model>"].params.fastMode: true`
rewrites native xAI requests as follows:

| Source model  | Fast-mode target   |
| ------------- | ------------------ |
| `grok-3`      | `grok-3-fast`      |
| `grok-3-mini` | `grok-3-mini-fast` |
| `grok-4`      | `grok-4-fast`      |
| `grok-4-0709` | `grok-4-fast`      |

### Legacy compatibility aliases

Legacy aliases still normalize to the canonical bundled ids:

| Legacy alias              | Canonical id                          |
| ------------------------- | ------------------------------------- |
| `grok-4-fast-reasoning`   | `grok-4-fast`                         |
| `grok-4-1-fast-reasoning` | `grok-4-1-fast`                       |
| `grok-4.20-reasoning`     | `grok-4.20-beta-latest-reasoning`     |
| `grok-4.20-non-reasoning` | `grok-4.20-beta-latest-non-reasoning` |

## Features

<AccordionGroup>
  <Accordion title="Web search">
    The bundled `grok` web-search provider uses `XAI_API_KEY` too:

    ```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
    openclaw config set tools.web.search.provider grok
    ```
  </Accordion>

  <Accordion title="Video generation">
    The bundled `xai` plugin registers video generation through the shared
    `video_generate` tool.

    * Default video model: `xai/grok-imagine-video`
    * Modes: text-to-video, image-to-video, reference-image generation, remote
      video edit, and remote video extension
    * Aspect ratios: `1:1`, `16:9`, `9:16`, `4:3`, `3:4`, `3:2`, `2:3`
    * Resolutions: `480P`, `720P`
    * Duration: 1-15 seconds for generation/image-to-video, 1-10 seconds when
      using `reference_image` roles, 2-10 seconds for extension
    * Reference-image generation: set `imageRoles` to `reference_image` for
      every supplied image; xAI accepts up to 7 such images

    <Warning>
      Local video buffers are not accepted. Use remote `http(s)` URLs for
      video edit/extend inputs. Image-to-video accepts local image buffers because
      OpenClaw can encode those as data URLs for xAI.
    </Warning>

    To use xAI as the default video provider:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          videoGenerationModel: {
            primary: "xai/grok-imagine-video",
          },
        },
      },
    }
    ```

    <Note>
      See [Video Generation](/tools/video-generation) for shared tool parameters,
      provider selection, and failover behavior.
    </Note>
  </Accordion>

  <Accordion title="Image generation">
    The bundled `xai` plugin registers image generation through the shared
    `image_generate` tool.

    * Default image model: `xai/grok-imagine-image`
    * Additional model: `xai/grok-imagine-image-pro`
    * Modes: text-to-image and reference-image edit
    * Reference inputs: one `image` or up to five `images`
    * Aspect ratios: `1:1`, `16:9`, `9:16`, `4:3`, `3:4`, `2:3`, `3:2`
    * Resolutions: `1K`, `2K`
    * Count: up to 4 images

    OpenClaw asks xAI for `b64_json` image responses so generated media can be
    stored and delivered through the normal channel attachment path. Local
    reference images are converted to data URLs; remote `http(s)` references are
    passed through.

    To use xAI as the default image provider:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      agents: {
        defaults: {
          imageGenerationModel: {
            primary: "xai/grok-imagine-image",
          },
        },
      },
    }
    ```

    <Note>
      xAI also documents `quality`, `mask`, `user`, and additional native ratios
      such as `1:2`, `2:1`, `9:20`, and `20:9`. OpenClaw forwards only the
      shared cross-provider image controls today; unsupported native-only knobs
      are intentionally not exposed through `image_generate`.
    </Note>
  </Accordion>

  <Accordion title="Text-to-speech">
    The bundled `xai` plugin registers text-to-speech through the shared `tts`
    provider surface.

    * Voices: `eve`, `ara`, `rex`, `sal`, `leo`, `una`
    * Default voice: `eve`
    * Formats: `mp3`, `wav`, `pcm`, `mulaw`, `alaw`
    * Language: BCP-47 code or `auto`
    * Speed: provider-native speed override
    * Native Opus voice-note format is not supported

    To use xAI as the default TTS provider:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      messages: {
        tts: {
          provider: "xai",
          providers: {
            xai: {
              voiceId: "eve",
            },
          },
        },
      },
    }
    ```

    <Note>
      OpenClaw uses xAI's batch `/v1/tts` endpoint. xAI also offers streaming TTS
      over WebSocket, but the OpenClaw speech provider contract currently expects
      a complete audio buffer before reply delivery.
    </Note>
  </Accordion>

  <Accordion title="Speech-to-text">
    The bundled `xai` plugin registers batch speech-to-text through OpenClaw's
    media-understanding transcription surface.

    * Default model: `grok-stt`
    * Endpoint: xAI REST `/v1/stt`
    * Input path: multipart audio file upload
    * Supported by OpenClaw wherever inbound audio transcription uses
      `tools.media.audio`, including Discord voice-channel segments and
      channel audio attachments

    To force xAI for inbound audio transcription:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      tools: {
        media: {
          audio: {
            models: [
              {
                type: "provider",
                provider: "xai",
                model: "grok-stt",
              },
            ],
          },
        },
      },
    }
    ```

    Language can be supplied through the shared audio media config or per-call
    transcription request. Prompt hints are accepted by the shared OpenClaw
    surface, but the xAI REST STT integration only forwards file, model, and
    language because those map cleanly to the current public xAI endpoint.
  </Accordion>

  <Accordion title="Streaming speech-to-text">
    The bundled `xai` plugin also registers a realtime transcription provider
    for live voice-call audio.

    * Endpoint: xAI WebSocket `wss://api.x.ai/v1/stt`
    * Default encoding: `mulaw`
    * Default sample rate: `8000`
    * Default endpointing: `800ms`
    * Interim transcripts: enabled by default

    Voice Call's Twilio media stream sends G.711 µ-law audio frames, so the
    xAI provider can forward those frames directly without transcoding:

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      plugins: {
        entries: {
          "voice-call": {
            config: {
              streaming: {
                enabled: true,
                provider: "xai",
                providers: {
                  xai: {
                    apiKey: "${XAI_API_KEY}",
                    endpointingMs: 800,
                    language: "en",
                  },
                },
              },
            },
          },
        },
      },
    }
    ```

    Provider-owned config lives under
    `plugins.entries.voice-call.config.streaming.providers.xai`. Supported
    keys are `apiKey`, `baseUrl`, `sampleRate`, `encoding` (`pcm`, `mulaw`, or
    `alaw`), `interimResults`, `endpointingMs`, and `language`.

    <Note>
      This streaming provider is for Voice Call's realtime transcription path.
      Discord voice currently records short segments and uses the batch
      `tools.media.audio` transcription path instead.
    </Note>
  </Accordion>

  <Accordion title="x_search configuration">
    The bundled xAI plugin exposes `x_search` as an OpenClaw tool for searching
    X (formerly Twitter) content via Grok.

    Config path: `plugins.entries.xai.config.xSearch`

    | Key               | Type    | Default         | Description                         |
    | ----------------- | ------- | --------------- | ----------------------------------- |
    | `enabled`         | boolean | -               | Enable or disable x\_search         |
    | `model`           | string  | `grok-4-1-fast` | Model used for x\_search requests   |
    | `baseUrl`         | string  | -               | xAI Responses base URL override     |
    | `inlineCitations` | boolean | -               | Include inline citations in results |
    | `maxTurns`        | number  | -               | Maximum conversation turns          |
    | `timeoutSeconds`  | number  | -               | Request timeout in seconds          |
    | `cacheTtlMinutes` | number  | -               | Cache time-to-live in minutes       |

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      plugins: {
        entries: {
          xai: {
            config: {
              xSearch: {
                enabled: true,
                model: "grok-4-1-fast",
                baseUrl: "https://api.x.ai/v1",
                inlineCitations: true,
              },
            },
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Code execution configuration">
    The bundled xAI plugin exposes `code_execution` as an OpenClaw tool for
    remote code execution in xAI's sandbox environment.

    Config path: `plugins.entries.xai.config.codeExecution`

    | Key              | Type    | Default                   | Description                            |
    | ---------------- | ------- | ------------------------- | -------------------------------------- |
    | `enabled`        | boolean | `true` (if key available) | Enable or disable code execution       |
    | `model`          | string  | `grok-4-1-fast`           | Model used for code execution requests |
    | `maxTurns`       | number  | -                         | Maximum conversation turns             |
    | `timeoutSeconds` | number  | -                         | Request timeout in seconds             |

    <Note>
      This is remote xAI sandbox execution, not local [`exec`](/tools/exec).
    </Note>

    ```json5 theme={"theme":{"light":"min-light","dark":"min-dark"}}
    {
      plugins: {
        entries: {
          xai: {
            config: {
              codeExecution: {
                enabled: true,
                model: "grok-4-1-fast",
              },
            },
          },
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Known limits">
    * Auth is API-key only today. There is no xAI OAuth or device-code flow in
      OpenClaw yet.
    * `grok-4.20-multi-agent-experimental-beta-0304` is not supported on the
      normal xAI provider path because it requires a different upstream API
      surface than the standard OpenClaw xAI transport.
    * xAI Realtime voice is not registered as an OpenClaw provider yet. It
      needs a different bidirectional voice session contract than batch STT or
      streaming transcription.
    * xAI image `quality`, image `mask`, and extra native-only aspect ratios are
      not exposed until the shared `image_generate` tool has corresponding
      cross-provider controls.
  </Accordion>

  <Accordion title="Advanced notes">
    * OpenClaw applies xAI-specific tool-schema and tool-call compatibility fixes
      automatically on the shared runner path.
    * Native xAI requests default `tool_stream: true`. Set
      `agents.defaults.models["xai/<model>"].params.tool_stream` to `false` to
      disable it.
    * The bundled xAI wrapper strips unsupported strict tool-schema flags and
      reasoning payload keys before sending native xAI requests.
    * `web_search`, `x_search`, and `code_execution` are exposed as OpenClaw
      tools. OpenClaw enables the specific xAI built-in it needs inside each tool
      request instead of attaching all native tools to every chat turn.
    * Grok `web_search` reads `plugins.entries.xai.config.webSearch.baseUrl`.
      `x_search` reads `plugins.entries.xai.config.xSearch.baseUrl`, then
      falls back to the Grok web-search base URL.
    * `x_search` and `code_execution` are owned by the bundled xAI plugin rather
      than hardcoded into the core model runtime.
    * `code_execution` is remote xAI sandbox execution, not local
      [`exec`](/tools/exec).
  </Accordion>
</AccordionGroup>

## Live testing

The xAI media paths are covered by unit tests and opt-in live suites. The live
commands load secrets from your login shell, including `~/.profile`, before
probing `XAI_API_KEY`.

```bash theme={"theme":{"light":"min-light","dark":"min-dark"}}
pnpm test extensions/xai
OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_TEST_QUIET=1 pnpm test:live -- extensions/xai/xai.live.test.ts
OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_TEST_QUIET=1 OPENCLAW_LIVE_IMAGE_GENERATION_PROVIDERS=xai pnpm test:live -- test/image-generation.runtime.live.test.ts
```

The provider-specific live file synthesizes normal TTS, telephony-friendly PCM
TTS, transcribes audio through xAI batch STT, streams the same PCM through xAI
realtime STT, generates text-to-image output, and edits a reference image. The
shared image live file verifies the same xAI provider through OpenClaw's
runtime selection, fallback, normalization, and media attachment path.

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>

  <Card title="Video generation" href="/tools/video-generation" icon="video">
    Shared video tool parameters and provider selection.
  </Card>

  <Card title="All providers" href="/providers/index" icon="grid-2">
    The broader provider overview.
  </Card>

  <Card title="Troubleshooting" href="/help/troubleshooting" icon="wrench">
    Common issues and fixes.
  </Card>
</CardGroup>
