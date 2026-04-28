# OpenClaw Plugins Documentation

## Building channel plugins
**Source:** https://docs.openclaw.ai/plugins/sdk-channel-plugins

[Skip to main content](https://docs.openclaw.ai/plugins/sdk-channel-plugins#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Building plugins

Building channel plugins

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [How channel plugins work](https://docs.openclaw.ai/plugins/sdk-channel-plugins#how-channel-plugins-work)
- [Approvals and channel capabilities](https://docs.openclaw.ai/plugins/sdk-channel-plugins#approvals-and-channel-capabilities)
- [Inbound mention policy](https://docs.openclaw.ai/plugins/sdk-channel-plugins#inbound-mention-policy)
- [Walkthrough](https://docs.openclaw.ai/plugins/sdk-channel-plugins#walkthrough)
- [File structure](https://docs.openclaw.ai/plugins/sdk-channel-plugins#file-structure)
- [Advanced topics](https://docs.openclaw.ai/plugins/sdk-channel-plugins#advanced-topics)
- [Next steps](https://docs.openclaw.ai/plugins/sdk-channel-plugins#next-steps)
- [Related](https://docs.openclaw.ai/plugins/sdk-channel-plugins#related)

This guide walks through building a channel plugin that connects OpenClaw to a
messaging platform. By the end you will have a working channel with DM security,
pairing, reply threading, and outbound messaging.

If you have not built any OpenClaw plugin before, read
[Getting Started](https://docs.openclaw.ai/plugins/building-plugins) first for the basic package
structure and manifest setup.

## [​](https://docs.openclaw.ai/plugins/sdk-channel-plugins\\#how-channel-plugins-work)  How channel plugins work

Channel plugins do not need their own send/edit/react tools. OpenClaw keeps one
shared `message` tool in core. Your plugin owns:

- **Config** — account resolution and setup wizard
- **Security** — DM policy and allowlists
- **Pairing** — DM approval flow
- **Session grammar** — how provider-specific conversation ids map to base chats, thread ids, and parent fallbacks
- **Outbound** — sending text, media, and polls to the platform
- **Threading** — how replies are threaded
- **Heartbeat typing** — optional typing/busy signals for heartbeat delivery targets

Core owns the shared message tool, prompt wiring, the outer session-key shape,
generic `:thread:` bookkeeping, and dispatch.If your channel supports typing indicators outside inbound replies, expose
`heartbeat.sendTyping(...)` on the channel plugin. Core calls it with the
resolved heartbeat delivery target before the heartbeat model run starts and
uses the shared typing keepalive/cleanup lifecycle. Add `heartbeat.clearTyping(...)`
when the platform needs an explicit stop signal.If your channel adds message-tool params that carry media sources, expose those
param names through `describeMessageTool(...).mediaSourceParams`. Core uses
that explicit list for sandbox path normalization and outbound media-access
policy, so plugins do not need shared-core special cases for provider-specific
avatar, attachment, or cover-image params.
Prefer returning an action-keyed map such as
`{ \"set-profile\": [\"avatarUrl\", \"avatarPath\"] }` so unrelated actions do not
inherit another action’s media args. A flat array still works for params that
are intentionally shared across every exposed action.If your platform stores extra scope inside conversation ids, keep that parsing
in the plugin with `messaging.resolveSessionConversation(...)`. That is the
canonical hook for mapping `rawId` to the base conversation id, optional thread
id, explicit `baseConversationId`, and any `parentConversationCandidates`.\nWhen you return `parentConversationCandidates`, keep them ordered from the\nnarrowest parent to the broadest/base conversation.Use `openclaw/plugin-sdk/channel-route` when plugin code needs to normalize\nroute-like fields, compare a child thread with its parent route, or build a\nstable dedupe key from `{ channel, to, accountId, threadId }`. The helper\nnormalizes numeric thread ids the same way core does, so plugins should prefer\nit over ad hoc `String(threadId)` comparisons.\nPlugins with provider-specific target grammar can inject their parser into\n`resolveChannelRouteTargetWithParser(...)` and still get the same route target\nshape and thread fallback semantics core uses.Bundled plugins that need the same parsing before the channel registry boots\ncan also expose a top-level `session-key-api.ts` file with a matching\n`resolveSessionConversation(...)` export. Core uses that bootstrap-safe surface\nonly when the runtime plugin registry is not available yet.`messaging.resolveParentConversationCandidates(...)` remains available as a\nlegacy compatibility fallback when a plugin only needs parent fallbacks on top\nof the generic/raw id. If both hooks exist, core uses\n`resolveSessionConversation(...).parentConversationCandidates` first and only\nfalls back to `resolveParentConversationCandidates(...)` when the canonical hook\nomits them.\n\n## [​](https://docs.openclaw.ai/plugins/sdk-channel-plugins\\#approvals-and-channel-capabilities)  Approvals and channel capabilities\n\nMost channel plugins do not need approval-specific code.\n\n- Core owns same-chat `/approve`, shared approval button payloads, and generic fallback delivery.\n- Prefer one `approvalCapability` object on the channel plugin when the channel needs approval-specific behavior.\n- `ChannelPlugin.approvals` is removed. Put approval delivery/native/render/auth facts on `approvalCapability`.\n- `plugin.auth` is login/logout only; core no longer reads approval auth hooks from that object.\n- `approvalCapability.authorizeActorAction` and `approvalCapability.getActionAvailabilityState` are the canonical approval-auth seam.\n- Use `approvalCapability.getActionAvailabilityState` for same-chat approval auth availability.\n- If your channel exposes native exec approvals, use `approvalCapability.getExecInitiatingSurfaceState` for the initiating-surface/native-client state when it differs from same-chat approval auth. Core uses that exec-specific hook to distinguish `enabled` vs `disabled`, decide whether the initiating channel supports native exec approvals, and include the channel in native-client fallback guidance. `createApproverRestrictedNativeApprovalCapability(...)` fills this in for the common case.\n- Use `outbound.shouldSuppressLocalPayloadPrompt` or `outbound.beforeDeliverPayload` for channel-specific payload lifecycle behavior such as hiding duplicate local approval prompts or sending typing indicators before delivery.\n- Use `approvalCapability.delivery` only for native approval routing or fallback suppression.\n- Use `approvalCapability.nativeRuntime` for channel-owned native approval facts. Keep it lazy on hot channel entrypoints with `createLazyChannelApprovalNativeRuntimeAdapter(...)`, which can import your runtime module on demand while still letting core assemble the approval lifecycle.\n- Use `approvalCapability.render` only when a channel truly needs custom approval payloads instead of the shared renderer.\n- Use `approvalCapability.describeExecApprovalSetup` when the channel wants the disabled-path reply to explain the exact config knobs needed to enable native exec approvals. The hook receives `{ channel, channelLabel, accountId }`; named-account channels should render account-scoped paths such as `channels.<channel>.accounts.<id>.execApprovals.*` instead of top-level defaults.\n- If a channel can infer stable owner-like DM identities from existing config, use `createResolvedApproverActionAuthAdapter` from `openclaw/plugin-sdk/approval-runtime` to restrict same-chat `/approve` without adding approval-specific core logic.\n- If a channel needs native approval delivery, keep channel code focused on target normalization plus transport/presentation facts. Use `createChannelExecApprovalProfile`, `createChannelNativeOriginTargetResolver`, `createChannelApproverDmTargetResolver`, and `createApproverRestrictedNativeApprovalCapability` from `openclaw/plugin-sdk/approval-runtime`. Put the channel-specific facts behind `approvalCapability.nativeRuntime`, ideally via `createChannelApprovalNativeRuntimeAdapter(...)` or `createLazyChannelApprovalNativeRuntimeAdapter(...)`, so core can assemble the handler and own request filtering, routing, dedupe, expiry, gateway subscription, and routed-elsewhere notices. `nativeRuntime` is split into a few smaller seams:\n- `createChannelNativeOriginTargetResolver` uses the shared channel-route matcher by default for `{ to, accountId, threadId }` targets. Pass `targetsMatch` only when a channel has provider-specific equivalence rules, such as Slack timestamp prefix matching.\n- Pass `normalizeTargetForMatch` to `createChannelNativeOriginTargetResolver` when the channel needs to canonicalize provider ids before the default route matcher or a custom `targetsMatch` callback runs, while preserving the original target for delivery. Use `normalizeTarget` only when the resolved delivery target itself should be canonicalized.\n- `availability` — whether the account is configured and whether a request should be handled\n- `presentation` — map the shared approval view model into pending/resolved/expired native payloads or final actions\n- `transport` — prepare targets plus send/update/delete native approval messages\n- `interactions` — optional bind/unbind/clear-action hooks for native buttons or reactions\n- `observe` — optional delivery diagnostics hooks\n- If the channel needs runtime-owned objects such as a client, token, Bolt app, or webhook receiver, register them through `openclaw/plugin-sdk/channel-runtime-context`. The generic runtime-context registry lets core bootstrap capability-driven handlers from channel startup state without adding approval-specific wrapper glue.\n- Reach for the lower-level `createChannelApprovalHandler` or `createChannelNativeApprovalRuntime` only when the capability-driven seam is not expressive enough yet.\n- Native approval channels must route both `accountId` and `approvalKind` through those helpers. `accountId` keeps multi-account approval policy scoped to the right bot account, and `approvalKind` keeps exec vs plugin approval behavior available to the channel without hardcoded branches in core.\n- Core now owns approval reroute notices too. Channel plugins should not send their own “approval went to DMs / another channel” follow-up messages from `createChannelNativeApprovalRuntime`; instead, expose accurate origin + approver-DM routing through the shared approval capability helpers and let core aggregate actual deliveries before posting any notice back to the initiating chat.\n- Preserve the delivered approval id kind end-to-end. Native clients should not\nguess or rewrite exec vs plugin approval routing from channel-local state.\n- Different approval kinds can intentionally expose different native surfaces.\nCurrent bundled examples:\n  - Slack keeps native approval routing available for both exec and plugin ids.\n  - Matrix keeps the same native DM/channel routing and reaction UX for exec\n    and plugin approvals, while still letting auth differ by approval kind.\n- `createApproverRestrictedNativeApprovalAdapter` still exists as a compatibility wrapper, but new code should prefer the capability builder and expose `approvalCapability` on the plugin.\n\nFor hot channel entrypoints, prefer the narrower runtime subpaths when you only\nneed one part of that family:\n\n- `openclaw/plugin-sdk/approval-auth-runtime`\n- `openclaw/plugin-sdk/approval-client-runtime`\n- `openclaw/plugin-sdk/approval-delivery-runtime`\n- `openclaw/plugin-sdk/approval-gateway-runtime`\n- `openclaw/plugin-sdk/approval-handler-adapter-runtime`\n- `openclaw/plugin-sdk/approval-handler-runtime`\n- `openclaw/plugin-sdk/approval-native-runtime`\n- `openclaw/plugin-sdk/approval-reply-runtime`\n- `openclaw/plugin-sdk/channel-runtime-context`\n\nLikewise, prefer `openclaw/plugin-sdk/setup-runtime`,\n`openclaw/plugin-sdk/setup-adapter-runtime`,\n`openclaw/plugin-sdk/reply-runtime`,\n`openclaw/plugin-sdk/reply-dispatch-runtime`,\n`openclaw/plugin-sdk/reply-reference`, and\n`openclaw/plugin-sdk/reply-chunking` when you do not need the broader umbrella\nsurface.For setup specifically:\n\n- `openclaw/plugin-sdk/setup-runtime` covers the runtime-safe setup helpers:\nimport-safe setup patch adapters (`createPatchedAccountSetupAdapter`,\n`createEnvPatchedAccountSetupAdapter`,\n`createSetupInputPresenceValidator`), lookup-note output,\n`promptResolvedAllowFrom`, `splitSetupEntries`, and the delegated\nsetup-proxy builders\n- `openclaw/plugin-sdk/setup-adapter-runtime` is the narrow env-aware adapter\nseam for `createEnvPatchedAccountSetupAdapter`\n- `openclaw/plugin-sdk/channel-setup` covers the optional-install setup\nbuilders plus a few setup-safe primitives:\n`createOptionalChannelSetupSurface`, `createOptionalChannelSetupAdapter`,\n\nIf your channel supports env-driven setup or auth and generic startup/config\nflows should know those env names before runtime loads, declare them in the\nplugin manifest with `channelEnvVars`. Keep channel runtime `envVars` or local\nconstants for operator-facing copy only.If your channel can appear in `status`, `channels list`, `channels status`, or\nSecretRef scans before the plugin runtime starts, add `openclaw.setupEntry` in\n`package.json`. That entrypoint should be safe to import in read-only command\npaths and should return the channel metadata, setup-safe config adapter, status\nadapter, and channel secret target metadata needed for those summaries. Do not\nstart clients, listeners, or transport runtimes from the setup entry.Keep the main channel entry import path narrow too. Discovery can evaluate the\nentry and the channel plugin module to register capabilities without activating\nthe channel. Files such as `channel-plugin-api.ts` should export the channel\nplugin object without importing setup wizards, transport clients, socket\nlisteners, subprocess launchers, or service startup modules. Put those runtime\npieces in modules loaded from `registerFull(...)`, runtime setters, or lazy\ncapability adapters.`createOptionalChannelSetupWizard`, `DEFAULT_ACCOUNT_ID`,\n`createTopLevelChannelDmPolicy`, `setSetupChannelEnabled`, and\n`splitSetupEntries`\n\n- use the broader `openclaw/plugin-sdk/setup` seam only when you also need the\nheavier shared setup/config helpers such as\n`moveSingleAccountChannelSectionToDefaultAccount(...)`\n\nIf your channel only wants to advertise “install this plugin first” in setup\nsurfaces, prefer `createOptionalChannelSetupSurface(...)`. The generated\nadapter/wizard fail closed on config writes and finalization, and they reuse\nthe same install-required message across validation, finalize, and docs-link\ncopy.For other hot channel paths, prefer the narrow helpers over broader legacy\nsurfaces:\n\n- `openclaw/plugin-sdk/account-core`,\n`openclaw/plugin-sdk/account-id`,\n`openclaw/plugin-sdk/account-resolution`, and\n`openclaw/plugin-sdk/account-helpers` for multi-account config and\ndefault-account fallback\n- `openclaw/plugin-sdk/inbound-envelope` and\n`openclaw/plugin-sdk/inbound-reply-dispatch` for inbound route/envelope and\nrecord-and-dispatch wiring\n- `openclaw/plugin-sdk/messaging-targets` for target parsing/matching\n- `openclaw/plugin-sdk/outbound-media` and\n`openclaw/plugin-sdk/outbound-runtime` for media loading plus outbound\nidentity/send delegates and payload planning\n- `buildThreadAwareOutboundSessionRoute(...)` from\n`openclaw/plugin-sdk/channel-core` when an outbound route should preserve an\nexplicit `replyToId`/`threadId` or recover the current `:thread:` session\nafter the base session key still matches. Provider plugins can override\nprecedence, suffix behavior, and thread id normalization when their platform\nhas native thread delivery semantics.\n- `openclaw/plugin-sdk/thread-bindings-runtime` for thread-binding lifecycle\nand adapter registration\n- `openclaw/plugin-sdk/agent-media-payload` only when a legacy agent/media\npayload field layout is still required\n- `openclaw/plugin-sdk/telegram-command-config` for Telegram custom-command\nnormalization, duplicate/conflict validation, and a fallback-stable command\nconfig contract\n\nAuth-only channels can usually stop at the default path: core handles approvals and the plugin just exposes outbound/auth capabilities. Native approval channels such as Matrix, Slack, Telegram, and custom chat transports should use the shared native helpers instead of rolling their own approval lifecycle.\n\n## [​](https://docs.openclaw.ai/plugins/sdk-channel-plugins\\#inbound-mention-policy)  Inbound mention policy\n\nKeep inbound mention handling split in two layers:\n\n- plugin-owned evidence gathering\n- shared policy evaluation\n\nUse `openclaw/plugin-sdk/channel-mention-gating` for mention-policy decisions.\nUse `openclaw/plugin-sdk/channel-inbound` only when you need the broader inbound\nhelper barrel.Good fit for plugin-local logic:\n\n- reply-to-bot detection\n- quoted-bot detection\n- thread-participation checks\n- service/system-message exclusions\n- platform-native caches needed to prove bot participation\n\nGood fit for the shared helper:\n\n- `requireMention`\n- explicit mention result\n- implicit mention allowlist\n- command bypass\n- final skip decision\n\nPreferred flow:\n\n1. Compute local mention facts.\n2. Pass those facts into `resolveInboundMentionDecision({ facts, policy })`.\n3. Use `decision.effectiveWasMentioned`, `decision.shouldBypassMention`, and `decision.shouldSkip` in your inbound gate.\n\n```\nimport {\n  implicitMentionKindWhen,\n  matchesMentionWithExplicit,\n  resolveInboundMentionDecision,\n} from \"openclaw/plugin-sdk/channel-inbound\";\n\nconst mentionMatch = matchesMentionWithExplicit(text, {
  mentionRegexes,
  mentionPatterns,
});

const facts = {
  canDetectMention: true,
  wasMentioned: mentionMatch.matched,
  hasAnyMention: mentionMatch.hasExplicitMention,
  implicitMentionKinds: [\
    ...implicitMentionKindWhen(\"reply_to_bot\", isReplyToBot),\\
    ...implicitMentionKindWhen(\"quoted_bot\", isQuoteOfBot),\\
  ],
};

const decision = resolveInboundMentionDecision({
  facts,
  policy: {
    isGroup,
    requireMention,
    allowedImplicitMentionKinds: requireExplicitMention ? [] : [\"reply_to_bot\", \"quoted_bot\"],
    allowTextCommands,
    hasControlCommand,
    commandAuthorized,
  },
});

if (decision.shouldSkip) return;
```

`api.runtime.channel.mentions` exposes the same shared mention helpers for\nbundled channel plugins that already depend on runtime injection:\n\n- `buildMentionRegexes`\n- `matchesMentionPatterns`\n- `matchesMentionWithExplicit`\n- `implicitMentionKindWhen`\n- `resolveInboundMentionDecision`\n\nIf you only need `implicitMentionKindWhen` and...(content truncated)

---

## Plugin manifest
**Source:** https://docs.openclaw.ai/plugins/manifest

[Skip to main content](https://docs.openclaw.ai/plugins/manifest#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

SDK reference

Plugin manifest

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [What this file does](https://docs.openclaw.ai/plugins/manifest#what-this-file-does)
- [Minimal example](https://docs.openclaw.ai/plugins/manifest#minimal-example)
- [Rich example](https://docs.openclaw.ai/plugins/manifest#rich-example)
- [Top-level field reference](https://docs.openclaw.ai/plugins/manifest#top-level-field-reference)
- [providerAuthChoices reference](https://docs.openclaw.ai/plugins/manifest#providerauthchoices-reference)
- [commandAliases reference](https://docs.openclaw.ai/plugins/manifest#commandaliases-reference)
- [activation reference](https://docs.openclaw.ai/plugins/manifest#activation-reference)
- [qaRunners reference](https://docs.openclaw.ai/plugins/manifest#qarunners-reference)
- [setup reference](https://docs.openclaw.ai/plugins/manifest#setup-reference)
- [setup.providers reference](https://docs.openclaw.ai/plugins/manifest#setup-providers-reference)
- [setup fields](https://docs.openclaw.ai/plugins/manifest#setup-fields)
- [uiHints reference](https://docs.openclaw.ai/plugins/manifest#uihints-reference)
- [contracts reference](https://docs.openclaw.ai/plugins/manifest#contracts-reference)
- [mediaUnderstandingProviderMetadata reference](https://docs.openclaw.ai/plugins/manifest#mediaunderstandingprovidermetadata-reference)
- [channelConfigs reference](https://docs.openclaw.ai/plugins/manifest#channelconfigs-reference)
- [Replacing another channel plugin](https://docs.openclaw.ai/plugins/manifest#replacing-another-channel-plugin)
- [modelSupport reference](https://docs.openclaw.ai/plugins/manifest#modelsupport-reference)
- [modelCatalog reference](https://docs.openclaw.ai/plugins/manifest#modelcatalog-reference)
- [modelIdNormalization reference](https://docs.openclaw.ai/plugins/manifest#modelidnormalization-reference)
- [providerEndpoints reference](https://docs.openclaw.ai/plugins/manifest#providerendpoints-reference)
- [providerRequest reference](https://docs.openclaw.ai/plugins/manifest#providerrequest-reference)
- [modelPricing reference](https://docs.openclaw.ai/plugins/manifest#modelpricing-reference)
- [OpenClaw Provider Index](https://docs.openclaw.ai/plugins/manifest#openclaw-provider-index)
- [Manifest versus package.json](https://docs.openclaw.ai/plugins/manifest#manifest-versus-package-json)
- [package.json fields that affect discovery](https://docs.openclaw.ai/plugins/manifest#package-json-fields-that-affect-discovery)
- [Discovery precedence (duplicate plugin ids)](https://docs.openclaw.ai/plugins/manifest#discovery-precedence-duplicate-plugin-ids)
- [JSON Schema requirements](https://docs.openclaw.ai/plugins/manifest#json-schema-requirements)
- [Validation behavior](https://docs.openclaw.ai/plugins/manifest#validation-behavior)
- [Notes](https://docs.openclaw.ai/plugins/manifest#notes)
- [Related](https://docs.openclaw.ai/plugins/manifest#related)

This page is for the **native OpenClaw plugin manifest** only.For compatible bundle layouts, see [Plugin bundles](https://docs.openclaw.ai/plugins/bundles).Compatible bundle formats use different manifest files:\n\n- Codex bundle: `.codex-plugin/plugin.json`\n- Claude bundle: `.claude-plugin/plugin.json` or the default Claude component\nlayout without a manifest\n- Cursor bundle: `.cursor-plugin/plugin.json`\n\nOpenClaw auto-detects those bundle layouts too, but they are not validated\nagainst the `openclaw.plugin.json` schema described here.For compatible bundles, OpenClaw currently reads bundle metadata plus declared\nskill roots, Claude command roots, Claude bundle `settings.json` defaults,\nClaude bundle LSP defaults, and supported hook packs when the layout matches\nOpenClaw runtime expectations.Every native OpenClaw plugin **must** ship a `openclaw.plugin.json` file in the\n**plugin root**. OpenClaw uses this manifest to validate configuration\n**without executing plugin code**. Missing or invalid manifests are treated as\nplugin errors and block config validation.See the full plugin system guide: [Plugins](https://docs.openclaw.ai/tools/plugin).\nFor the native capability model and current external-compatibility guidance:\n[Capability model](https://docs.openclaw.ai/plugins/architecture#public-capability-model).\n\n## [​](https://docs.openclaw.ai/plugins/manifest\\#what-this-file-does)  What this file does\n\n`openclaw.plugin.json` is the metadata OpenClaw reads **before it loads your**\n**plugin code**. Everything below must be cheap enough to inspect without booting\nplugin runtime.**Use it for:**\n\n- plugin identity, config validation, and config UI hints\n- auth, onboarding, and setup metadata (alias, auto-enable, provider env vars, auth choices)\n- activation hints for control-plane surfaces\n- shorthand model-family ownership\n- static capability-ownership snapshots (`contracts`)\n- QA runner metadata the shared `openclaw qa` host can inspect\n- channel-specific config metadata merged into catalog and validation surfaces\n\n**Do not use it for:** registering runtime behavior, declaring code entrypoints,\nor npm install metadata. Those belong in your plugin code and `package.json`.\n\n## [​](https://docs.openclaw.ai/plugins/manifest\\#minimal-example)  Minimal example\n\n```\n{\n  \"id\": \"voice-call\",\n  \"configSchema\": {\n    \"type\": \"object\",\n    \"additionalProperties\": false,\n    \"properties\": {}\n  }\n}\n```\n\n## [​](https://docs.openclaw.ai/plugins/manifest\\#rich-example)  Rich example\n\n```\n{\n  \"id\": \"openrouter\",\n  \"name\": \"OpenRouter\",\n  \"description\": \"OpenRouter provider plugin\",\n  \"version\": \"1.0.0\",\n  \"providers\": [\"openrouter\"],\n  \"modelSupport\": {\n    \"modelPrefixes\": [\"router-\"]\n  },\n  \"modelIdNormalization\": {\n    \"providers\": {\n      \"openrouter\": {\n        \"prefixWhenBare\": \"openrouter\"\n      }\n    }\n  },\n  \"providerEndpoints\": [\\\n    {\\\n      \"endpointClass\": \"openrouter\",\\\n      \"hostSuffixes\": [\"openrouter.ai\"]\\\n    }\\\n  ],\n  \"providerRequest\": {\n    \"providers\": {\n      \"openrouter\": {\n        \"family\": \"openrouter\"\n      }\n    }\n  },\n  \"cliBackends\": [\"openrouter-cli\"],\n  \"syntheticAuthRefs\": [\"openrouter-cli\"],\n  \"providerAuthEnvVars\": {\n    \"openrouter\": [\"OPENROUTER_API_KEY\"]\n  },\n  \"providerAuthAliases\": {\n    \"openrouter-coding\": \"openrouter\"\n  },\n  \"channelEnvVars\": {\n    \"openrouter-chatops\": [\"OPENROUTER_CHATOPS_TOKEN\"]\n  },\n  \"providerAuthChoices\": [\\\n    {\\\n      \"provider\": \"openrouter\",\\\n      \"method\": \"api-key\",\\\n      \"choiceId\": \"openrouter-api-key\",\\\n      \"choiceLabel\": \"OpenRouter API key\",\\\n      \"groupId\": \"openrouter\",\\\n      \"groupLabel\": \"OpenRouter\",\\\n      \"optionKey\": \"openrouterApiKey\",\\\n      \"cliFlag\": \"--openrouter-api-key\",\\\n      \"cliOption\": \"--openrouter-api-key <key>\",\\\n      \"cliDescription\": \"OpenRouter API key\",\\\n      \"onboardingScopes\": [\"text-inference\"]\\\n    }\\\n  ],\n  \"uiHints\": {\n    \"apiKey\": {\n      \"label\": \"API key\",\n      \"placeholder\": \"sk-or-v1-...\",\n      \"sensitive\": true\n    }\n  },\n  \"configSchema\": {\n    \"type\": \"object\",\n    \"additionalProperties\": false,\n    \"properties\": {\n      \"apiKey\": {\n        \"type\": \"string\"\n      }\n    }\n  }\n}\n```\n\n## [​](https://docs.openclaw.ai/plugins/manifest\\#top-level-field-reference)  Top-level field reference\n\n| Field | Required | Type | What it means |\n| --- | --- | --- | --- |\n| `id` | Yes | `string` | Canonical plugin id. This is the id used in `plugins.entries.<id>`. |\n| `configSchema` | Yes | `object` | Inline JSON Schema for this plugin’s config. |\n| `enabledByDefault` | No | `true` | Marks a bundled plugin as enabled by default. Omit it, or set any non-`true` value, to leave the plugin disabled by default. |\n| `legacyPluginIds` | No | `string[]` | Legacy ids that normalize to this canonical plugin id. |\n| `autoEnableWhenConfiguredProviders` | No | `string[]` | Provider ids that should auto-enable this plugin when auth, config, or model refs mention them. |\n| `kind` | No | `\"memory\"` \\| `\"context-engine\"` | Declares an exclusive plugin kind used by `plugins.slots.*`. |\n| `channels` | No | `string[]` | Channel ids owned by this plugin. Used for discovery and config validation. |\n| `providers` | No | `string[]` | Provider ids owned by this plugin. |\n| `providerDiscoveryEntry` | No | `string` | Lightweight provider-discovery module path, relative to the plugin root, for manifest-scoped provider catalog metadata that can be loaded without activating the full plugin runtime. |\n| `modelSupport` | No | `object` | Manifest-owned shorthand model-family metadata used to auto-load the plugin before runtime. |\n| `modelCatalog` | No | `object` | Declarative model catalog metadata for providers owned by this plugin. This is the control-plane contract for future read-only listing, onboarding, model pickers, aliases, and suppression without loading plugin runtime. |\n| `modelPricing` | No | `object` | Provider-owned external pricing lookup policy. Use it to opt local/self-hosted providers out of remote pricing catalogs or map provider refs to OpenRouter/LiteLLM catalog ids without hardcoding provider ids in core. |\n| `modelIdNormalization` | No | `object` | Provider-owned model-id alias/prefix cleanup that must run before provider runtime loads. |\n| `providerEndpoints` | No | `object[]` | Manifest-owned endpoint host/baseUrl metadata for provider routes that core must classify before provider runtime loads. |\n| `providerRequest` | No | `object` | Cheap provider-family and request-compatibility metadata used by generic request policy before provider runtime loads. |\n| `cliBackends` | No | `string[]` | CLI inference backend ids owned by this plugin. Used for startup auto-activation from explicit config refs. |\n| `syntheticAuthRefs` | No | `string[]` | Provider or CLI backend refs whose plugin-owned synthetic auth hook should be probed during cold model discovery before runtime loads. |\n| `nonSecretAuthMarkers` | No | `string[]` | Bundled-plugin-owned placeholder API key values that represent non-secret local, OAuth, or ambient credential state. |\n| `commandAliases` | No | `object[]` | Command names owned by this plugin that should produce plugin-aware config and CLI diagnostics before runtime loads. |\n| `providerAuthEnvVars` | No | `Record<string, string[]>` | Deprecated compatibility env metadata for provider auth/status lookup. Prefer `setup.providers[].envVars` for new plugins; OpenClaw still reads this during the deprecation window. |\n| `providerAuthAliases` | No | `Record<string, string>` | Provider ids that should reuse another provider id for auth lookup, for example a coding provider that shares the base provider API key and auth profiles. |\n| `channelEnvVars` | No | `Record<string, string[]>` | Cheap channel env metadata that OpenClaw can inspect without loading plugin code. Use this for env-driven channel setup or auth surfaces that generic startup/config helpers should see. |\n| `providerAuthChoices` | No | `object[]` | Cheap auth-choice metadata for onboarding pickers, preferred-provider resolution, and simple CLI flag wiring. |\n| `activation` | No | `object` | Cheap activation planner metadata for provider, command, channel, route, and capability-triggered loading. Metadata only; plugin runtime still owns actual behavior. |\n| `setup` | No | `object` | Cheap setup/onboarding descriptors that discovery and setup surfaces can inspect without loading plugin runtime. |\n| `qaRunners` | No | `object[]` | Cheap QA runner descriptors used by the shared `openclaw qa` host before plugin runtime loads. |\n| `contracts` | No | `object` | Static bundled capability snapshot for external auth hooks, speech, realtime transcription, realtime voice, media-understanding, image-generation, music-generation, video-generation, web-fetch, web search, and tool ownership. |\n| `mediaUnderstandingProviderMetadata` | No | `Record<string, object>` | Cheap media-understanding defaults for provider ids declared in `contracts.mediaUnderstandingProviders`. |\n| `channelConfigs` | No | `Record<string, object>` | Manifest-owned channel config metadata merged into discovery and validation surfaces before runtime loads. |\n| `skills` | No | `string[]` | Skill directories to load, relative to the plugin root. |\n| `name` | No | `string` | Human-readable plugin name. |\n| `description` | No | `string` | Short summary shown in plugin surfaces. |\n| `version` | No | `string` | Informational plugin version. |\n| `uiHints` | No | `Record<string, object>` | UI labels, placeholders, and sensitivity hints for config fields. |\n\n## [​](https://docs.openclaw.ai/plugins/manifest\\#providerauthchoices-reference)  providerAuthChoices reference\n\nEach `providerAuthChoices` entry describes one onboarding or auth choice.\nOpenClaw reads this before provider runtime loads.\nProvider setup lists use these manifest choices, descriptor-derived setup\nchoices, and install-catalog metadata without loading provider runtime.\n\n| Field | Required | Type | What it means |\n| --- | --- | --- | --- |\n| `provider` | Yes | `string` | Provider id this choice belongs to. |\n| `method` | Yes | `string` | Auth method id to dispatch to. |\n| `choiceId` | Yes | `string` | Stable auth-choice id used by onboarding and CLI flows. |\n| `choiceLabel` | No | `string` | User-facing label. If omitted, OpenClaw falls back to `choiceId`. |\n| `choiceHint` | No | `string` | Short helper text for the picker. |\n| `assistantPriority` | No | `number` | Lower values sort earlier in assistant-driven interactive pickers. |\n| `assistantVisibility` | No | `\"visible\"` \\| `\"manual-only\"` | Hide the choice from assistant pickers while still allowing manual CLI selection. |\n| `deprecatedChoiceIds` | No | `string[]` | Legacy choice ids that should redirect users to this replacement choice. |\n| `groupId` | No | `string` | Optional group id for grouping related choices. |\n| `groupLabel` | No | `string` | User-facing label for that group. |\n| `groupHint` | No | `string` | Short helper text for the group. |\n| `optionKey` | No | `string` | Internal option key for simple one-flag auth flows. |\n| `cliFlag` | No | `string` | CLI flag name, such as `--openrouter-api-key`. |\n| `cliOption` | No | `string` | Full CLI option shape, such as `--openrouter-api-key <key>`. |\n| `cliDescription` | No | `string` | Description used in CLI help. |\n| `onboardingScopes` | No | `Array<\"text-inference\" | \"image-generation\">` | Which onboarding surfaces this choice should appear in. If omitted, it defaults to `[\"text-inference\"]`. |\n\n## [​](https://docs.openclaw.ai/plugins/manifest\\#commandaliases-reference)  commandAliases reference\n\nUse `commandAliases` when a plugin owns a runtime command name that users may\nmistakenly put in `plugins.allow` or try to run as a root CLI command. OpenClaw\nuses this metadata for diagnostics without importing plugin runtime code.\n\n```\n{\n  \"commandAliases\": [\\\n    {\\\n      \"name\": \"dreaming\",\\\n      \"kind\": \"runtime-slash\",\\\n      \"cliCommand\": \"memory\"\\\n    }\\\n  ]\n}\n```\n\n| Field | Required | Type | What it means |\n| --- | --- | --- | --- |\n| `name` | Yes | `string` | Command name that belongs to this plugin. |\n| `kind` | No | `\"runtime-slash\"` | Marks the alias as a chat slash command rather than a root CLI command. |\n| `cliCommand` | No | `string` | Related root CLI command to suggest for CLI operations, if one exists. |\n\n## [​](https://docs.openclaw.ai/plugins/manifest\\#activation-reference)  activation reference\n\nUse `activation` when the plugin can cheaply declare which control-plane events\nshould include it in an activation/load plan.This block is planner metadata, not a lifecycle API. It does not register\nruntime behavior, does not replace `register(...)`, and does not promise that\nplugin code has already executed. The activation planner uses these fields to\nnarrow candidate plugins before falling back to existing manifest ownership\nmetadata such as `providers`, `channels`, `commandAliases`, `setup.providers`,\n`contracts.tools`, and hooks.Prefer the narrowest metadata that already describes ownership. Use\n`providers`, `channels`, `commandAliases`, setup descriptors, or `contracts`\nwhen those fields express the relationship. Use `activation` for extra planner\nhints that cannot be represented by those ownership fields.\nUse top-level `cliBackends` for CLI runtime aliases such as `claude-cli`,\n`codex-cli`, or `google-gemini-cli`; `activation.onAgentHarnesses` is only for\nembedded agent harness ids that do not already have an ownership field.This block is metadata only. It does not register runtime behavior, and it does\nnot replace `register(...)`, `setupEntry`, or other runtime/plugin entrypoints.\nCurrent consumers use it as a narrowing hint before broader plugin loading, so\nmissing activation metadata usually only costs performance; it should not\nchange correctness while legacy manifest ownership fallbacks still exist.\n\n```\n{\n  \"activation\": {\n    \"onProviders\": [\"openai\"],\n    \"onCommands\": [\"models\"],\n    \"onChannels\": [\"web\"],\n    \"onRoutes\": [\"gateway-webhook\"],\n    \"onConfigPaths\": [\"browser\"],\n    \"onCapabilities\": [\"provider\", \"tool\"]\n  }\n}\n```\n\n| Field | Required | Type | What it means |\n| --- | --- | --- | --- |\n| `onProviders` | Yes | `string[]` | Provider ids that should include this plugin in activation/load plans. |\n| `onAgentHarnesses` | Yes | `string[]` | Embedded agent harness runtime ids that should include this plugin in activation/load plans. Use top-level `cliBackends` for CLI backend aliases. |\n| `onCommands` | Yes | `string[]` | Command ids that should include this plugin in activation/load plans. |\n| `onChannels` | Yes | `string[]` | Channel ids that should include this plugin in activation/load plans. |\n| `onRoutes` | Yes | `string[]` | Route kinds that should include this plugin in activation/load plans. |\n| `onConfigPaths` | Yes | `string[]` | Root-relative config paths that should include this plugin in startup/load plans when the path is present and not explicitly disabled. |\n| `onCapabilities` | Yes | `Array<\"provider\" | \"channel\" | \"tool\" | \"hook\">` | Broad capability hints used by control-plane activation planning. Prefer narrower fields when possible. |...(content truncated)

---

## Plugin bundles - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/bundles

[Skip to main content](https://docs.openclaw.ai/plugins/bundles#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Plugin bundles

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Why bundles exist](https://docs.openclaw.ai/plugins/bundles#why-bundles-exist)
- [Install a bundle](https://docs.openclaw.ai/plugins/bundles#install-a-bundle)
- [What OpenClaw maps from bundles](https://docs.openclaw.ai/plugins/bundles#what-openclaw-maps-from-bundles)
- [Supported now](https://docs.openclaw.ai/plugins/bundles#supported-now)
- [Skill content](https://docs.openclaw.ai/plugins/bundles#skill-content)
- [Hook packs](https://docs.openclaw.ai/plugins/bundles#hook-packs)
- [MCP for Pi](https://docs.openclaw.ai/plugins/bundles#mcp-for-pi)
- [Embedded Pi settings](https://docs.openclaw.ai/plugins/bundles#embedded-pi-settings)
- [Embedded Pi LSP](https://docs.openclaw.ai/plugins/bundles#embedded-pi-lsp)
- [Detected but not executed](https://docs.openclaw.ai/plugins/bundles#detected-but-not-executed)
- [Bundle formats](https://docs.openclaw.ai/plugins/bundles#bundle-formats)
- [Detection precedence](https://docs.openclaw.ai/plugins/bundles#detection-precedence)
- [Runtime dependencies and cleanup](https://docs.openclaw.ai/plugins/bundles#runtime-dependencies-and-cleanup)
- [Security](https://docs.openclaw.ai/plugins/bundles#security)
- [Troubleshooting](https://docs.openclaw.ai/plugins/bundles#troubleshooting)
- [Related](https://docs.openclaw.ai/plugins/bundles#related)

OpenClaw can install plugins from three external ecosystems: **Codex**, **Claude**,
and **Cursor**. These are called **bundles** — content and metadata packs that
OpenClaw maps into native features like skills, hooks, and MCP tools.

Bundles are **not** the same as native OpenClaw plugins. Native plugins run
in-process and can register any capability. Bundles are content packs with
selective feature mapping and a narrower trust boundary.

## [​](https://docs.openclaw.ai/plugins/bundles\\#why-bundles-exist)  Why bundles exist

Many useful plugins are published in Codex, Claude, or Cursor format. Instead
of requiring authors to rewrite them as native OpenClaw plugins, OpenClaw
detects these formats and maps their supported content into the native feature
set. This means you can install a Claude command pack or a Codex skill bundle
and use it immediately.

## [​](https://docs.openclaw.ai/plugins/bundles\\#install-a-bundle)  Install a bundle

1

[Navigate to header](https://docs.openclaw.ai/plugins/bundles#)

Install from a directory, archive, or marketplace

```
# Local directory
openclaw plugins install ./my-bundle

# Archive
openclaw plugins install ./my-bundle.tgz

# Claude marketplace
openclaw plugins marketplace list <marketplace-name>
openclaw plugins install <plugin-name>@<marketplace-name>
```

2

[Navigate to header](https://docs.openclaw.ai/plugins/bundles#)

Verify detection

```
openclaw plugins list
openclaw plugins inspect <id>
```

Bundles show as `Format: bundle` with a subtype of `codex`, `claude`, or `cursor`.

3

[Navigate to header](https://docs.openclaw.ai/plugins/bundles#)

Restart and use

```
openclaw gateway restart
```

Mapped features (skills, hooks, MCP tools, LSP defaults) are available in the next session.

## [​](https://docs.openclaw.ai/plugins/bundles\\#what-openclaw-maps-from-bundles)  What OpenClaw maps from bundles

Not every bundle feature runs in OpenClaw today. Here is what works and what
is detected but not yet wired.

### [​](https://docs.openclaw.ai/plugins/bundles\\#supported-now)  Supported now

| Feature | How it maps | Applies to |
| --- | --- | --- |
| Skill content | Bundle skill roots load as normal OpenClaw skills | All formats |
| Commands | `commands/` and `.cursor/commands/` treated as skill roots | Claude, Cursor |
| Hook packs | OpenClaw-style `HOOK.md` \\+ `handler.ts` layouts | Codex |
| MCP tools | Bundle MCP config merged into embedded Pi settings; supported stdio and HTTP servers loaded | All formats |
| LSP servers | Claude `.lsp.json` and manifest-declared `lspServers` merged into embedded Pi LSP defaults | Claude |
| Settings | Claude `settings.json` imported as embedded Pi defaults | Claude |

#### [​](https://docs.openclaw.ai/plugins/bundles\\#skill-content)  Skill content

- bundle skill roots load as normal OpenClaw skill roots
- Claude `commands` roots are treated as additional skill roots
- Cursor `.cursor/commands` roots are treated as additional skill roots

This means Claude markdown command files work through the normal OpenClaw skill
loader. Cursor command markdown works through the same path.

#### [​](https://docs.openclaw.ai/plugins/bundles\\#hook-packs)  Hook packs

- bundle hook roots work **only** when they use the normal OpenClaw hook-pack
layout. Today this is primarily the Codex-compatible case:

  - `HOOK.md`
  - `handler.ts` or `handler.js`

#### [​](https://docs.openclaw.ai/plugins/bundles\\#mcp-for-pi)  MCP for Pi

- enabled bundles can contribute MCP server config
- OpenClaw merges bundle MCP config into the effective embedded Pi settings as
`mcpServers`
- OpenClaw exposes supported bundle MCP tools during embedded Pi agent turns by
launching stdio servers or connecting to HTTP servers
- the `coding` and `messaging` tool profiles include bundle MCP tools by
default; use `tools.deny: [\"bundle-mcp\"]` to opt out for an agent or gateway
- project-local Pi settings still apply after bundle defaults, so workspace
settings can override bundle MCP entries when needed
- bundle MCP tool catalogs are sorted deterministically before registration, so
upstream `listTools()` order changes do not thrash prompt-cache tool blocks

##### Transports

MCP servers can use stdio or HTTP transport:**Stdio** launches a child process:

```
{
  \"mcp\": {
    \"servers\": {
      \"my-server\": {
        \"command\": \"node\",
        \"args\": [\"server.js\"],
        \"env\": { \"PORT\": \"3000\" }
      }
    }
  }
}
```

**HTTP** connects to a running MCP server over `sse` by default, or `streamable-http` when requested:

```
{
  \"mcp\": {
    \"servers\": {
      \"my-server\": {
        \"url\": \"http://localhost:3100/mcp\",
        \"transport\": \"streamable-http\",
        \"headers\": {
          \"Authorization\": \"Bearer ${MY_SECRET_TOKEN}\"
        },
        \"connectionTimeoutMs\": 30000
      }
    }
  }
}
```

- `transport` may be set to `\"streamable-http\"` or `\"sse\"`; when omitted, OpenClaw uses `sse`
- `type: \"http\"` is a CLI-native downstream shape; use `transport: \"streamable-http\"` in OpenClaw config. `openclaw mcp set` and `openclaw doctor --fix` normalize the common alias.
- only `http:` and `https:` URL schemes are allowed
- `headers` values support `${ENV_VAR}` interpolation
- a server entry with both `command` and `url` is rejected
- URL credentials (userinfo and query params) are redacted from tool\ndescriptions and logs
- `connectionTimeoutMs` overrides the default 30-second connection timeout for\nboth stdio and HTTP transports

##### Tool naming

OpenClaw registers bundle MCP tools with provider-safe names in the form\n`serverName__toolName`. For example, a server keyed `\"vigil-harbor\"` exposing a\n`memory_search` tool registers as `vigil-harbor__memory_search`.

- characters outside `A-Za-z0-9_-` are replaced with `-`
- server prefixes are capped at 30 characters
- full tool names are capped at 64 characters
- empty server names fall back to `mcp`
- colliding sanitized names are disambiguated with numeric suffixes
- final exposed tool order is deterministic by safe name to keep repeated Pi\nturns cache-stable
- profile filtering treats all tools from one bundle MCP server as plugin-owned\nby `bundle-mcp`, so profile allowlists and deny lists can include either\nindividual exposed tool names or the `bundle-mcp` plugin key

#### [​](https://docs.openclaw.ai/plugins/bundles\\#embedded-pi-settings)  Embedded Pi settings

- Claude `settings.json` is imported as default embedded Pi settings when the\nbundle is enabled
- OpenClaw sanitizes shell override keys before applying them

Sanitized keys:

- `shellPath`
- `shellCommandPrefix`

#### [​](https://docs.openclaw.ai/plugins/bundles\\#embedded-pi-lsp)  Embedded Pi LSP

- enabled Claude bundles can contribute LSP server config
- OpenClaw loads `.lsp.json` plus any manifest-declared `lspServers` paths
- bundle LSP config is merged into the effective embedded Pi LSP defaults
- only supported stdio-backed LSP servers are runnable today; unsupported\ntransports still show up in `openclaw plugins inspect <id>`

### [​](https://docs.openclaw.ai/plugins/bundles\\#detected-but-not-executed)  Detected but not executed

These are recognized and shown in diagnostics, but OpenClaw does not run them:

- Claude `agents`, `hooks.json` automation, `outputStyles`
- Cursor `.cursor/agents`, `.cursor/hooks.json`, `.cursor/rules`
- Codex inline/app metadata beyond capability reporting

## [​](https://docs.openclaw.ai/plugins/bundles\\#bundle-formats)  Bundle formats

Codex bundles

Markers: `.codex-plugin/plugin.json`Optional content: `skills/`, `hooks/`, `.mcp.json`, `.app.json`Codex bundles fit OpenClaw best when they use skill roots and OpenClaw-style\nhook-pack directories (`HOOK.md` \\+ `handler.ts`).

Claude bundles

Two detection modes:

- **Manifest-based:**`.claude-plugin/plugin.json`
- **Manifestless:** default Claude layout (`skills/`, `commands/`, `agents/`, `hooks/`, `.mcp.json`, `.lsp.json`, `settings.json`)

Claude-specific behavior:

- `commands/` is treated as skill content
- `settings.json` is imported into embedded Pi settings (shell override keys are sanitized)
- `.mcp.json` exposes supported stdio tools to embedded Pi
- `.lsp.json` plus manifest-declared `lspServers` paths load into embedded Pi LSP defaults
- `hooks/hooks.json` is detected but not executed
- Custom component paths in the manifest are additive (they extend defaults, not replace them)

Cursor bundles

Markers: `.cursor-plugin/plugin.json`Optional content: `skills/`, `.cursor/commands/`, `.cursor/agents/`, `.cursor/rules/`, `.cursor/hooks.json`, `.mcp.json`

- `.cursor/commands/` is treated as skill content
- `.cursor/rules/`, `.cursor/agents/`, and `.cursor/hooks.json` are detect-only

## [​](https://docs.openclaw.ai/plugins/bundles\\#detection-precedence)  Detection precedence

OpenClaw checks for native plugin format first:

1. `openclaw.plugin.json` or valid `package.json` with `openclaw.extensions` — treated as **native plugin**
2. Bundle markers (`.codex-plugin/`, `.claude-plugin/`, or default Claude/Cursor layout) — treated as **bundle**

If a directory contains both, OpenClaw uses the native path. This prevents\ndual-format packages from being partially installed as bundles.

## [​](https://openclaw.ai/plugins/bundles\\#runtime-dependencies-and-cleanup)  Runtime dependencies and cleanup

- Bundled plugin runtime dependencies ship inside the OpenClaw package under\n`dist/*`. OpenClaw does **not** run `npm install` at startup for bundled\nplugins; the release pipeline is responsible for shipping a complete bundled\ndependency payload (see the postpublish verification rule in\n[Releasing](https://docs.openclaw.ai/reference/RELEASING)).

## [​](https://docs.openclaw.ai/plugins/bundles\\#security)  Security

Bundles have a narrower trust boundary than native plugins:

- OpenClaw does **not** load arbitrary bundle runtime modules in-process
- Skills and hook-pack paths must stay inside the plugin root (boundary-checked)
- Settings files are read with the same boundary checks
- Supported stdio MCP servers may be launched as subprocesses

This makes bundles safer by default, but you should still treat third-party\nbundles as trusted content for the features they do expose.

## [​](https://docs.openclaw.ai/plugins/bundles\\#troubleshooting)  Troubleshooting

Bundle is detected but capabilities do not run

Run `openclaw plugins inspect <id>`. If a capability is listed but marked as\nnot wired, that is a product limit — not a broken install.

Claude command files do not appear

Make sure the bundle is enabled and the markdown files are inside a detected\n`commands/` or `skills/` root.

Claude settings do not apply

Only embedded Pi settings from `settings.json` are supported. OpenClaw does\nnot treat bundle settings as raw config patches.

Claude hooks do not execute

`hooks/hooks.json` is detect-only. If you need runnable hooks, use the\nOpenClaw hook-pack layout or ship a native plugin.

## [​](https://docs.openclaw.ai/plugins/bundles\\#related)  Related

- [Install and Configure Plugins](https://docs.openclaw.ai/tools/plugin)
- [Building Plugins](https://docs.openclaw.ai/plugins/building-plugins) — create a native plugin
- [Plugin Manifest](https://docs.openclaw.ai/plugins/manifest) — native manifest schema

[Community plugins](https://docs.openclaw.ai/plugins/community) [Codex harness](https://docs.openclaw.ai/plugins/codex-harness)

Ctrl+I

---

## Plugin architecture internals
**Source:** https://docs.openclaw.ai/plugins/architecture-internals

[Skip to main content](https://docs.openclaw.ai/plugins/architecture-internals#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

SDK reference

Plugin architecture internals

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Load pipeline](https://docs.openclaw.ai/plugins/architecture-internals#load-pipeline)
- [Manifest-first behavior](https://docs.openclaw.ai/plugins/architecture-internals#manifest-first-behavior)
- [What the loader caches](https://docs.openclaw.ai/plugins/architecture-internals#what-the-loader-caches)
- [Registry model](https://docs.openclaw.ai/plugins/architecture-internals#registry-model)
- [Conversation binding callbacks](https://docs.openclaw.ai/plugins/architecture-internals#conversation-binding-callbacks)
- [Provider runtime hooks](https://docs.openclaw.ai/plugins/architecture-internals#provider-runtime-hooks)
- [Hook order and usage](https://docs.openclaw.ai/plugins/architecture-internals#hook-order-and-usage)
- [Provider example](https://docs.openclaw.ai/plugins/architecture-internals#provider-example)
- [Built-in examples](https://docs.openclaw.ai/plugins/architecture-internals#built-in-examples)
- [Runtime helpers](https://docs.openclaw.ai/plugins/architecture-internals#runtime-helpers)
- [api.runtime.imageGeneration](https://docs.openclaw.ai/plugins/architecture-internals#api-runtime-imagegeneration)
- [Gateway HTTP routes](https://docs.openclaw.ai/plugins/architecture-internals#gateway-http-routes)
- [Plugin SDK import paths](https://docs.openclaw.ai/plugins/architecture-internals#plugin-sdk-import-paths)
- [Message tool schemas](https://docs.openclaw.ai/plugins/architecture-internals#message-tool-schemas)
- [Channel target resolution](https://docs.openclaw.ai/plugins/architecture-internals#channel-target-resolution)
- [Config-backed directories](https://docs.openclaw.ai/plugins/architecture-internals#config-backed-directories)
- [Provider catalogs](https://docs.openclaw.ai/plugins/architecture-internals#provider-catalogs)
- [Read-only channel inspection](https://docs.openclaw.ai/plugins/architecture-internals#read-only-channel-inspection)
- [Package packs](https://docs.openclaw.ai/plugins/architecture-internals#package-packs)
- [Channel catalog metadata](https://docs.openclaw.ai/plugins/architecture-internals#channel-catalog-metadata)
- [Context engine plugins](https://docs.openclaw.ai/plugins/architecture-internals#context-engine-plugins)
- [Adding a new capability](https://docs.openclaw.ai/plugins/architecture-internals#adding-a-new-capability)
- [Capability checklist](https://docs.openclaw.ai/plugins/architecture-internals#capability-checklist)
- [Capability template](https://docs.openclaw.ai/plugins/architecture-internals#capability-template)
- [Related](https://docs.openclaw.ai/plugins/architecture-internals#related)

For the public capability model, plugin shapes, and ownership/execution
contracts, see [Plugin architecture](https://docs.openclaw.ai/plugins/architecture). This page is the
reference for the internal mechanics: load pipeline, registry, runtime hooks,
Gateway HTTP routes, import paths, and schema tables.

## [​](https://docs.openclaw.ai/plugins/architecture-internals\\#load-pipeline)  Load pipeline

At startup, OpenClaw does roughly this:

1. discover candidate plugin roots
2. read native or compatible bundle manifests and package metadata
3. reject unsafe candidates
4. normalize plugin config (`plugins.enabled`, `allow`, `deny`, `entries`,
`slots`, `load.paths`)
5. decide enablement for each candidate
6. load enabled native modules: built bundled modules use a native loader;
unbuilt native plugins use jiti
7. call native `register(api)` hooks and collect registrations into the plugin registry
8. expose the registry to commands/runtime surfaces

`activate` is a legacy alias for `register` — the loader resolves whichever is present (`def.register ?? def.activate`) and calls it at the same point. All bundled plugins use `register`; prefer `register` for new plugins.

The safety gates happen **before** runtime execution. Candidates are blocked
when the entry escapes the plugin root, the path is world-writable, or path
ownership looks suspicious for non-bundled plugins.

### [​](https://docs.openclaw.ai/plugins/architecture-internals\\#manifest-first-behavior)  Manifest-first behavior

The manifest is the control-plane source of truth. OpenClaw uses it to:

- identify the plugin
- discover declared channels/skills/config schema or bundle capabilities
- validate `plugins.entries.<id>.config`
- augment Control UI labels/placeholders
- show install/catalog metadata
- preserve cheap activation and setup descriptors without loading plugin runtime

For native plugins, the runtime module is the data-plane part. It registers
actual behavior such as hooks, tools, commands, or provider flows.Optional manifest `activation` and `setup` blocks stay on the control plane.
They are metadata-only descriptors for activation planning and setup discovery;
they do not replace runtime registration, `register(...)`, or `setupEntry`.
The first live activation consumers now use manifest command, channel, and provider hints
to narrow plugin loading before broader registry materialization:

- CLI loading narrows to plugins that own the requested primary command
- channel setup/plugin resolution narrows to plugins that own the requested
channel id
- explicit provider setup/runtime resolution narrows to plugins that own the
requested provider id
- Gateway startup planning uses `activation.onStartup` for explicit startup
imports and startup opt-outs; every plugin should declare it as OpenClaw
moves away from implicit startup imports, while plugins without static
capability metadata and without `activation.onStartup` still use the
deprecated implicit startup sidecar fallback for compatibility

The activation planner exposes both an ids-only API for existing callers and a
plan API for new diagnostics. Plan entries report why a plugin was selected,
separating explicit `activation.*` planner hints from manifest ownership
fallback such as `providers`, `channels`, `commandAliases`, `setup.providers`,
`contracts.tools`, and hooks. That reason split is the compatibility boundary:
existing plugin metadata keeps working, while new code can detect broad hints
or fallback behavior without changing runtime loading semantics.Setup discovery now prefers descriptor-owned ids such as `setup.providers` and
`setup.cliBackends` to narrow candidate plugins before it falls back to
`setup-api` for plugins that still need setup-time runtime hooks. Provider
setup lists use manifest `providerAuthChoices`, descriptor-derived setup
choices, and install-catalog metadata without loading provider runtime. Explicit
`setup.requiresRuntime: false` is a descriptor-only cutoff; omitted
`requiresRuntime` keeps the legacy setup-api fallback for compatibility. If more
than one discovered plugin claims the same normalized setup provider or CLI
backend id, setup lookup refuses the ambiguous owner instead of relying on
discovery order. When setup runtime does execute, registry diagnostics report
drift between `setup.providers` / `setup.cliBackends` and the providers or CLI
backends registered by setup-api without blocking legacy plugins.

### [​](https://docs.openclaw.ai/plugins/architecture-internals\\#what-the-loader-caches)  What the loader caches

OpenClaw keeps short in-process caches for:

- discovery results
- manifest registry data
- loaded plugin registries

These caches reduce bursty startup and repeated command overhead. They are safe
to think of as short-lived performance caches, not persistence.Gateway startup hot paths should prefer the current `PluginMetadataSnapshot`,
the derived `PluginLookUpTable`, or an explicit manifest registry passed through
the call chain. Config validation, startup auto-enable, and plugin bootstrap use
the same snapshot when available. For callers that still rebuild manifest
metadata from the persisted installed plugin index, OpenClaw also keeps a small
bounded fallback cache keyed by the installed index, request shape, config
policy, runtime roots, and manifest/package file signatures. That cache is only a
fallback for repeated installed-index reconstruction; it is not a mutable runtime
plugin registry.Performance note:

- Set `OPENCLAW_DISABLE_PLUGIN_DISCOVERY_CACHE=1` or
`OPENCLAW_DISABLE_PLUGIN_MANIFEST_CACHE=1` to disable these caches.
- Set `OPENCLAW_DISABLE_INSTALLED_PLUGIN_MANIFEST_REGISTRY_CACHE=1` to disable
only the installed-index manifest-registry fallback cache.
- Tune cache windows with `OPENCLAW_PLUGIN_DISCOVERY_CACHE_MS` and
`OPENCLAW_PLUGIN_MANIFEST_CACHE_MS`.

## [​](https://docs.openclaw.ai/plugins/architecture-internals\\#registry-model)  Registry model

Loaded plugins do not directly mutate random core globals. They register into a
central plugin registry.The registry tracks:

- plugin records (identity, source, origin, status, diagnostics)
- tools
- legacy hooks and typed hooks
- channels
- providers
- gateway RPC handlers
- HTTP routes
- CLI registrars
- background services
- plugin-owned commands

Core features then read from that registry instead of talking to plugin modules
directly. This keeps loading one-way:

- plugin module -> registry registration
- core runtime -> registry consumption

That separation matters for maintainability. It means most core surfaces only
need one integration point: “read the registry”, not “special-case every plugin
module”.

## [​](https://docs.openclaw.ai/plugins/architecture-internals\\#conversation-binding-callbacks)  Conversation binding callbacks

Plugins that bind a conversation can react when an approval is resolved.Use `api.onConversationBindingResolved(...)` to receive a callback after a bind
request is approved or denied:

```
export default {
  id: \"my-plugin\",
  register(api) {
    api.onConversationBindingResolved(async (event) => {
      if (event.status === \"approved\") {
        // A binding now exists for this plugin + conversation.
        console.log(event.binding?.conversationId);
        return;
      }

      // The request was denied; clear any local pending state.
      console.log(event.request.conversation.conversationId);
    });
  },
};
```

Callback payload fields:

- `status`: `\"approved\"` or `\"denied\"`
- `decision`: `\"allow-once\"`, `\"allow-always\"`, or `\"deny\"`
- `binding`: the resolved binding for approved requests
- `request`: the original request summary, detach hint, sender id, and
conversation metadata

This callback is notification-only. It does not change who is allowed to bind a
conversation, and it runs after core approval handling finishes.

## [​](https://docs.openclaw.ai/plugins/architecture-internals\\#provider-runtime-hooks)  Provider runtime hooks

Provider plugins have three layers:

- **Manifest metadata** for cheap pre-runtime lookup:\n`setup.providers[].envVars`, deprecated compatibility `providerAuthEnvVars`,
`providerAuthAliases`, `providerAuthChoices`, and `channelEnvVars`.
- **Config-time hooks**: `catalog` (legacy `discovery`) plus
`applyConfigDefaults`.
- **Runtime hooks**: 40+ optional hooks covering auth, model resolution,
stream wrapping, thinking levels, replay policy, and usage endpoints. See
the full list under [Hook order and usage](https://docs.openclaw.ai/plugins/architecture-internals#hook-order-and-usage).\n
OpenClaw still owns the generic agent loop, failover, transcript handling, and
tool policy. These hooks are the extension surface for provider-specific
behavior without needing a whole custom inference transport.Use manifest `setup.providers[].envVars` when the provider has env-based
credentials that generic auth/status/model-picker paths should see without
loading plugin runtime. Deprecated `providerAuthEnvVars` is still read by the
compatibility adapter during the deprecation window, and non-bundled plugins
that use it receive a manifest diagnostic. Use manifest `providerAuthAliases`
when one provider id should reuse another provider id’s env vars, auth profiles,
config-backed auth, and API-key onboarding choice. Use manifest
`providerAuthChoices` when onboarding/auth-choice CLI surfaces should know the
provider’s choice id, group labels, and simple one-flag auth wiring without
loading provider runtime. Keep provider runtime
`envVars` for operator-facing hints such as onboarding labels or OAuth
client-id/client-secret setup vars.Use manifest `channelEnvVars` when a channel has env-driven auth or setup that
generic shell-env fallback, config/status checks, or setup prompts should see
without loading channel runtime.\n
### [​](https://docs.openclaw.ai/plugins/architecture-internals\\#hook-order-and-usage)  Hook order and usage

For model/provider plugins, OpenClaw calls hooks in this rough order.\nThe “When to use” column is the quick decision guide.\n\n| # | Hook | What it does | When to use |\n| --- | --- | --- | --- |\n| 1 | `catalog` | Publish provider config into `models.providers` during `models.json` generation | Provider owns a catalog or base URL defaults |\n| 2 | `applyConfigDefaults` | Apply provider-owned global config defaults during config materialization | Defaults depend on auth mode, env, or provider model-family semantics |\n| — | _(built-in model lookup)_ | OpenClaw tries the normal registry/catalog path first | _(not a plugin hook)_ |\n| 3 | `normalizeModelId` | Normalize legacy or preview model-id aliases before lookup | Provider owns alias cleanup before canonical model resolution |\n| 4 | `normalizeTransport` | Normalize provider-family `api` / `baseUrl` before generic model assembly | Provider owns transport cleanup for custom provider ids in the same transport family |\n| 5 | `normalizeConfig` | Normalize `models.providers.<id>` before runtime/provider resolution | Provider needs config cleanup that should live with the plugin; bundled Google-family helpers also backstop supported Google config entries |\n| 6 | `applyNativeStreamingUsageCompat` | Apply native streaming-usage compat rewrites to config providers | Provider needs endpoint-driven native streaming usage metadata fixes |\n| 7 | `resolveConfigApiKey` | Resolve env-marker auth for config providers before runtime auth loading | Provider has provider-owned env-marker API-key resolution; `amazon-bedrock` also has a built-in AWS env-marker resolver here |\n| 8 | `resolveSyntheticAuth` | Surface local/self-hosted or config-backed auth without persisting plaintext | Provider can operate with a synthetic/local credential marker |\n| 9 | `resolveExternalAuthProfiles` | Overlay provider-owned external auth profiles; default `persistence` is `runtime-only` for CLI/app-owned creds | Provider reuses external auth credentials without persisting copied refresh tokens; declare `contracts.externalAuthProviders` in the manifest |\n| 10 | `shouldDeferSyntheticProfileAuth` | Lower stored synthetic profile placeholders behind env/config-backed auth | Provider stores synthetic placeholder profiles that should not win precedence |\n| 11 | `resolveDynamicModel` | Sync fallback for provider-owned model ids not in the local registry yet | Provider accepts arbitrary upstream model ids |\n| 12 | `prepareDynamicModel` | Async warm-up, then `resolveDynamicModel` runs again | Provider needs network metadata before resolving unknown ids |\n| 13 | `normalizeResolvedModel` | Final rewrite before the embedded runner uses the resolved model | Provider needs transport rewrites but still uses a core transport |\n| 14 | `contributeResolvedModelCompat` | Contribute compat flags for vendor models behind another compatible transport | Provider recognizes its own models on proxy transports without taking over the provider |\n| 15 | `capabilities` | Provider-owned transcript/tooling metadata used by shared core logic | Provider needs transcript/provider-family quirks |\n| 16 | `normalizeToolSchemas` | Normalize tool schemas before the embedded runner sees them | Provider needs transport-family schema cleanup |\n| 17 | `inspectToolSchemas` | Surface provider-owned schema diagnostics after normalization | Provider wants keyword warnings without teaching core provider-specific rules |\n| 18 | `resolveReasoningOutputMode` | Select native vs tagged reasoning-output contract | Provider needs tagged reasoning/final output instead of native fields |\n| 19 | `prepareExtraParams` | Request-param normalization before generic stream option wrappers | Provider needs default request params or per-provider param cleanup |\n| 20 | `createStreamFn` | Fully replace the normal stream path with a custom transport | Provider needs a custom wire protocol, not just a wrapper |\n| 21 | `wrapStreamFn` | Stream wrapper after generic wrappers are applied | Provider needs request headers/body/model compat wrappers without a custom transport |\n| 22 | `resolveTransportTurnState` | Attach native per-turn transport headers or metadata | Provider wants generic transports to send provider-native turn identity |\n| 23 | `resolveWebSocketSessionPolicy` | Attach native WebSocket headers or session cool-down policy | Provider wants generic WS transports to tune session headers or fallback policy |\n| 24 | `formatApiKey` | Auth-profile formatter: stored profile becomes the runtime `apiKey` string | Provider stores extra auth metadata and needs a custom runtime token shape |\n| 25 | `refreshOAuth` | OAuth refresh override for custom refresh endpoints or refresh-failure policy | Provider does not fit the shared `pi-ai` refreshers |\n| 26 | `buildAuthDoctorHint` | Repair hint appended when OAuth refresh fails | Provider needs provider-owned auth repair guidance after refresh failure |\n| 27 | `matchesContextOverflowError` | Provider-owned context-window overflow matcher | Provider has raw overflow errors generic heuristics would miss |\n| 28 | `classifyFailoverReason` | Provider-owned failover reason classification | Provider can map raw API/transport errors to rate-limit/overload/etc |\n| 29 | `isCacheTtlEligible` | Prompt-cache policy for proxy/backhaul providers | Provider needs proxy-specific cache TTL gating |\n| 30 | `buildMissingAuthMessage` | Replacement for the generic missing-auth recovery message | Provider needs a provider-specific missing-auth recovery hint |\n| 31 | `suppressBuiltInModel` | Deprecated. Runtime hook is no longer called; use manifest `modelCatalog.suppressions` | Historical hook for hiding stale upstream rows; keep new suppression data in the plugin manifest |
| 32 | `augmen...(content truncated)

---

## Memory wiki - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/memory-wiki

[Skip to main content](https://docs.openclaw.ai/plugins/memory-wiki#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Memory wiki

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [What it adds](https://docs.openclaw.ai/plugins/memory-wiki#what-it-adds)
- [How it fits with memory](https://docs.openclaw.ai/plugins/memory-wiki#how-it-fits-with-memory)
- [Recommended hybrid pattern](https://docs.openclaw.ai/plugins/memory-wiki#recommended-hybrid-pattern)
- [Vault modes](https://docs.openclaw.ai/plugins/memory-wiki#vault-modes)
- [isolated](https://docs.openclaw.ai/plugins/memory-wiki#isolated)
- [bridge](https://docs.openclaw.ai/plugins/memory-wiki#bridge)
- [unsafe-local](https://docs.openclaw.ai/plugins/memory-wiki#unsafe-local)
- [Vault layout](https://docs.openclaw.ai/plugins/memory-wiki#vault-layout)
- [Structured claims and evidence](https://docs.openclaw.ai/plugins/memory-wiki#structured-claims-and-evidence)
- [Compile pipeline](https://docs.openclaw.ai/plugins/memory-wiki#compile-pipeline)
- [Dashboards and health reports](https://docs.openclaw.ai/plugins/memory-wiki#dashboards-and-health-reports)
- [Search and retrieval](https://docs.openclaw.ai/plugins/memory-wiki#search-and-retrieval)
- [Agent tools](https://docs.openclaw.ai/plugins/memory-wiki#agent-tools)
- [Prompt and context behavior](https://docs.openclaw.ai/plugins/memory-wiki#prompt-and-context-behavior)
- [Configuration](https://docs.openclaw.ai/plugins/memory-wiki#configuration)
- [Example: QMD + bridge mode](https://docs.openclaw.ai/plugins/memory-wiki#example-qmd-%2B-bridge-mode)
- [CLI](https://docs.openclaw.ai/plugins/memory-wiki#cli)
- [Obsidian support](https://docs.openclaw.ai/plugins/memory-wiki#obsidian-support)
- [Recommended workflow](https://docs.openclaw.ai/plugins/memory-wiki#recommended-workflow)
- [Related docs](https://docs.openclaw.ai/plugins/memory-wiki#related-docs)

`memory-wiki` is a bundled plugin that turns durable memory into a compiled
knowledge vault.It does **not** replace the active memory plugin. The active memory plugin still
owns recall, promotion, indexing, and dreaming. `memory-wiki` sits beside it
and compiles durable knowledge into a navigable wiki with deterministic pages,
structured claims, provenance, dashboards, and machine-readable digests.Use it when you want memory to behave more like a maintained knowledge layer and
less like a pile of Markdown files.

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#what-it-adds)  What it adds

- A dedicated wiki vault with deterministic page layout
- Structured claim and evidence metadata, not just prose
- Page-level provenance, confidence, contradictions, and open questions
- Compiled digests for agent/runtime consumers
- Wiki-native search/get/apply/lint tools
- Optional bridge mode that imports public artifacts from the active memory plugin
- Optional Obsidian-friendly render mode and CLI integration

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#how-it-fits-with-memory)  How it fits with memory

Think of the split like this:

| Layer | Owns |
| --- | --- |
| Active memory plugin (`memory-core`, QMD, Honcho, etc.) | Recall, semantic search, promotion, dreaming, memory runtime |
| `memory-wiki` | Compiled wiki pages, provenance-rich syntheses, dashboards, wiki-specific search/get/apply |

If the active memory plugin exposes shared recall artifacts, OpenClaw can search
both layers in one pass with `memory_search corpus=all`.When you need wiki-specific ranking, provenance, or direct page access, use the
wiki-native tools instead.

## [​](https://openclaw.ai/plugins/memory-wiki\\#recommended-hybrid-pattern)  Recommended hybrid pattern

A strong default for local-first setups is:

- QMD as the active memory backend for recall and broad semantic search
- `memory-wiki` in `bridge` mode for durable synthesized knowledge pages

That split works well because each layer stays focused:

- QMD keeps raw notes, session exports, and extra collections searchable
- `memory-wiki` compiles stable entities, claims, dashboards, and source pages

Practical rule:

- use `memory_search` when you want one broad recall pass across memory
- use `wiki_search` and `wiki_get` when you want provenance-aware wiki results
- use `memory_search corpus=all` when you want shared search to span both layers

If bridge mode reports zero exported artifacts, the active memory plugin is not
currently exposing public bridge inputs yet. Run `openclaw wiki doctor` first,
then confirm the active memory plugin supports public artifacts.

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#vault-modes)  Vault modes

`memory-wiki` supports three vault modes:

### [​](https://docs.openclaw.ai/plugins/memory-wiki\\#isolated)  `isolated`

Own vault, own sources, no dependency on `memory-core`.Use this when you want the wiki to be its own curated knowledge store.

### [​](https://docs.openclaw.ai/plugins/memory-wiki\\#bridge)  `bridge`

Reads public memory artifacts and memory events from the active memory plugin
through public plugin SDK seams.Use this when you want the wiki to compile and organize the memory plugin’s
exported artifacts without reaching into private plugin internals.Bridge mode can index:

- exported memory artifacts
- dream reports
- daily notes
- memory root files
- memory event logs

### [​](https://docs.openclaw.ai/plugins/memory-wiki\\#unsafe-local)  `unsafe-local`

Explicit same-machine escape hatch for local private paths.This mode is intentionally experimental and non-portable. Use it only when you
understand the trust boundary and specifically need local filesystem access that
bridge mode cannot provide.

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#vault-layout)  Vault layout

The plugin initializes a vault like this:

```
<vault>/
  AGENTS.md
  WIKI.md
  index.md
  inbox.md
  entities/
  concepts/
  syntheses/
  sources/
  reports/
  _attachments/
  _views/
  .openclaw-wiki/
```

Managed content stays inside generated blocks. Human note blocks are preserved.The main page groups are:

- `sources/` for imported raw material and bridge-backed pages
- `entities/` for durable things, people, systems, projects, and objects
- `concepts/` for ideas, abstractions, patterns, and policies
- `syntheses/` for compiled summaries and maintained rollups
- `reports/` for generated dashboards

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#structured-claims-and-evidence)  Structured claims and evidence

Pages can carry structured `claims` frontmatter, not just freeform text.Each claim can include:

- `id`
- `text`
- `status`
- `confidence`
- `evidence[]`
- `updatedAt`

Evidence entries can include:

- `sourceId`
- `path`
- `lines`
- `weight`
- `note`
- `updatedAt`

This is what makes the wiki act more like a belief layer than a passive note
dump. Claims can be tracked, scored, contested, and resolved back to sources.

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#compile-pipeline)  Compile pipeline

The compile step reads wiki pages, normalizes summaries, and emits stable
machine-facing artifacts under:

- `.openclaw-wiki/cache/agent-digest.json`
- `.openclaw-wiki/cache/claims.jsonl`

These digests exist so agents and runtime code do not have to scrape Markdown
pages.Compiled output also powers:

- first-pass wiki indexing for search/get flows
- claim-id lookup back to owning pages
- compact prompt supplements
- report/dashboard generation

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#dashboards-and-health-reports)  Dashboards and health reports

When `render.createDashboards` is enabled, compile maintains dashboards under
`reports/`.Built-in reports include:

- `reports/open-questions.md`
- `reports/contradictions.md`
- `reports/low-confidence.md`
- `reports/claim-health.md`
- `reports/stale-pages.md`

These reports track things like:

- contradiction note clusters
- competing claim clusters
- claims missing structured evidence
- low-confidence pages and claims
- stale or unknown freshness
- pages with unresolved questions

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#search-and-retrieval)  Search and retrieval

`memory-wiki` supports two search backends:

- `shared`
- `local`

It also supports three corpora:

- `wiki`
- `memory`
- `all`

Important behavior:

- `wiki_search` and `wiki_get` use compiled digests as a first pass when possible
- claim ids can resolve back to the owning page
- contested/stale/fresh claims influence ranking
- provenance labels can survive into results

Practical rule:

- use `memory_search corpus=all` for one broad recall pass
- use `wiki_search` \\+ `wiki_get` when you care about wiki-specific ranking,
provenance, or page-level belief structure

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#agent-tools)  Agent tools

The plugin registers these tools:

- `wiki_status`
- `wiki_search`
- `wiki_get`
- `wiki_apply`
- `wiki_lint`

What they do:

- `wiki_status`: current vault mode, health, Obsidian CLI availability
- `wiki_search`: search wiki pages and, when configured, shared memory corpora
- `wiki_get`: read a wiki page by id/path or fall back to shared memory corpus
- `wiki_apply`: narrow synthesis/metadata mutations without freeform page surgery
- `wiki_lint`: structural checks, provenance gaps, contradictions, open questions

The plugin also registers a non-exclusive memory corpus supplement, so shared
`memory_search` and `memory_get` can reach the wiki when the active memory
plugin supports corpus selection.

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#prompt-and-context-behavior)  Prompt and context behavior

When `context.includeCompiledDigestPrompt` is enabled, memory prompt sections
append a compact compiled snapshot from `agent-digest.json`.That snapshot is intentionally small and high-signal:

- top pages only
- top claims only
- contradiction count
- question count
- confidence/freshness qualifiers

This is opt-in because it changes prompt shape and is mainly useful for context
engines or legacy prompt assembly that explicitly consume memory supplements.

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#configuration)  Configuration

Put config under `plugins.entries.memory-wiki.config`:

```
{
  plugins: {
    entries: {
      \"memory-wiki\": {
        enabled: true,
        config: {
          vaultMode: \"isolated\",
          vault: {
            path: \"~/.openclaw/wiki/main\",
            renderMode: \"obsidian\",
          },
          obsidian: {
            enabled: true,
            useOfficialCli: true,
            vaultName: \"OpenClaw Wiki\",
            openAfterWrites: false,
          },
          bridge: {
            enabled: false,
            readMemoryArtifacts: true,
            indexDreamReports: true,
            indexDailyNotes: true,
            indexMemoryRoot: true,
            followMemoryEvents: true,
          },
          ingest: {
            autoCompile: true,
            maxConcurrentJobs: 1,
            allowUrlIngest: true,
          },
          search: {
            backend: \"shared\",
            corpus: \"wiki\",
          },
          context: {
            includeCompiledDigestPrompt: false,
          },
          render: {
            preserveHumanBlocks: true,
            createBacklinks: true,
            createDashboards: true,
          },
        },
      },
    },
  },
}
```

Key toggles:

- `vaultMode`: `isolated`, `bridge`, `unsafe-local`
- `vault.renderMode`: `native` or `obsidian`
- `bridge.readMemoryArtifacts`: import active memory plugin public artifacts
- `bridge.followMemoryEvents`: include event logs in bridge mode
- `search.backend`: `shared` or `local`
- `search.corpus`: `wiki`, `memory`, or `all`
- `context.includeCompiledDigestPrompt`: append compact digest snapshot to memory prompt sections
- `render.createBacklinks`: generate deterministic related blocks
- `render.createDashboards`: generate dashboard pages

### [​](https://docs.openclaw.ai/plugins/memory-wiki\\#example-qmd-+-bridge-mode)  Example: QMD + bridge mode

Use this when you want QMD for recall and `memory-wiki` for a maintained
knowledge layer:

```
{
  memory: {
    backend: \"qmd\",
      \"memory-wiki\": {
        enabled: true,
        config: {
          vaultMode: \"bridge\",
          bridge: {
            enabled: true,
            readMemoryArtifacts: true,
            indexDreamReports: true,
            indexDailyNotes: true,
            indexMemoryRoot: true,
            followMemoryEvents: true,
          },
          search: {
            backend: \"shared\",
            corpus: \"all\",
          },
          context: {
            includeCompiledDigestPrompt: false,
          },
        },
      },
    },
  },
}
```

This keeps:

- QMD in charge of active memory recall
- `memory-wiki` focused on compiled pages and dashboards
- prompt shape unchanged until you intentionally enable compiled digest prompts

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#cli)  CLI

`memory-wiki` also exposes a top-level CLI surface:

```
openclaw wiki status
openclaw wiki doctor
openclaw wiki init
openclaw wiki ingest ./notes/alpha.md
openclaw wiki compile
openclaw wiki lint
openclaw wiki search \"alpha\"
openclaw wiki get entity.alpha
openclaw wiki apply synthesis \"Alpha Summary\" --body \"...\" --source-id source.alpha
openclaw wiki bridge import
openclaw wiki obsidian status
```

See [CLI: wiki](https://docs.openclaw.ai/cli/wiki) for the full command reference.

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#obsidian-support)  Obsidian support

When `vault.renderMode` is `obsidian`, the plugin writes Obsidian-friendly
Markdown and can optionally use the official `obsidian` CLI.Supported workflows include:

- status probing
- vault search
- opening a page
- invoking an Obsidian command
- jumping to the daily note

This is optional. The wiki still works in native mode without Obsidian.

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#recommended-workflow)  Recommended workflow

1. Keep your active memory plugin for recall/promotion/dreaming.
2. Enable `memory-wiki`.
3. Start with `isolated` mode unless you explicitly want bridge mode.
4. Use `wiki_search` / `wiki_get` when provenance matters.
5. Use `wiki_apply` for narrow syntheses or metadata updates.
6. Run `wiki_lint` after meaningful changes.
7. Turn on dashboards if you want stale/contradiction visibility.

## [​](https://docs.openclaw.ai/plugins/memory-wiki\\#related-docs)  Related docs

- [Memory Overview](https://docs.openclaw.ai/concepts/memory)
- [CLI: memory](https://docs.openclaw.ai/cli/memory)
- [CLI: wiki](https://docs.openclaw.ai/cli/wiki)
- [Plugin SDK overview](https://docs.openclaw.ai/plugins/sdk-overview)

[Voice call](https://docs.openclaw.ai/plugins/voice-call) [Message presentation](https://docs.openclaw.ai/plugins/message-presentation)

Ctrl+I

---

## Plugin testing - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/sdk-testing

[Skip to main content](https://docs.openclaw.ai/plugins/sdk-testing#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

SDK reference

Plugin testing

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Test utilities](https://docs.openclaw.ai/plugins/sdk-testing#test-utilities)
- [Available exports](https://docs.openclaw.ai/plugins/sdk-testing#available-exports)
- [Types](https://docs.openclaw.ai/plugins/sdk-testing#types)
- [Testing target resolution](https://docs.openclaw.ai/plugins/sdk-testing#testing-target-resolution)
- [Testing patterns](https://docs.openclaw.ai/plugins/sdk-testing#testing-patterns)
- [Testing registration contracts](https://docs.openclaw.ai/plugins/sdk-testing#testing-registration-contracts)
- [Testing runtime config access](https://docs.openclaw.ai/plugins/sdk-testing#testing-runtime-config-access)
- [Unit testing a channel plugin](https://docs.openclaw.ai/plugins/sdk-testing#unit-testing-a-channel-plugin)
- [Unit testing a provider plugin](https://docs.openclaw.ai/plugins/sdk-testing#unit-testing-a-provider-plugin)
- [Mocking the plugin runtime](https://docs.openclaw.ai/plugins/sdk-testing#mocking-the-plugin-runtime)
- [Testing with per-instance stubs](https://docs.openclaw.ai/plugins/sdk-testing#testing-with-per-instance-stubs)
- [Contract tests (in-repo plugins)](https://docs.openclaw.ai/plugins/sdk-testing#contract-tests-in-repo-plugins)
- [Running scoped tests](https://docs.openclaw.ai/plugins/sdk-testing#running-scoped-tests)
- [Lint enforcement (in-repo plugins)](https://docs.openclaw.ai/plugins/sdk-testing#lint-enforcement-in-repo-plugins)
- [Test configuration](https://docs.openclaw.ai/plugins/sdk-testing#test-configuration)
- [Related](https://docs.openclaw.ai/plugins/sdk-testing#related)

Reference for test utilities, patterns, and lint enforcement for OpenClaw
plugins.

**Looking for test examples?** The how-to guides include worked test examples:
[Channel plugin tests](https://docs.openclaw.ai/plugins/sdk-channel-plugins#step-6-test) and
[Provider plugin tests](https://docs.openclaw.ai/plugins/sdk-provider-plugins#step-6-test).

## [​](https://docs.openclaw.ai/plugins/sdk-testing\\#test-utilities)  Test utilities

**Import:**`openclaw/plugin-sdk/testing`The testing subpath exports a narrow set of helpers for plugin authors:

```
import {
  installCommonResolveTargetErrorCases,
  shouldAckReaction,
  removeAckReactionAfterReply,
} from \"openclaw/plugin-sdk/testing\";
```

### [​](https://docs.openclaw.ai/plugins/sdk-testing\\#available-exports)  Available exports

| Export | Purpose |
| --- | --- |
| `installCommonResolveTargetErrorCases` | Shared test cases for target resolution error handling |
| `shouldAckReaction` | Check whether a channel should add an ack reaction |
| `removeAckReactionAfterReply` | Remove ack reaction after reply delivery |

### [​](https://docs.openclaw.ai/plugins/sdk-testing\\#types)  Types

The testing subpath also re-exports types useful in test files:

```
import type {
  ChannelAccountSnapshot,
  ChannelGatewayContext,
  OpenClawConfig,
  PluginRuntime,
  RuntimeEnv,
  MockFn,
} from \"openclaw/plugin-sdk/testing\";
```

## [​](https://docs.openclaw.ai/plugins/sdk-testing\\#testing-target-resolution)  Testing target resolution

Use `installCommonResolveTargetErrorCases` to add standard error cases for
channel target resolution:

```
import { describe } from \"vitest\";
import { installCommonResolveTargetErrorCases } from \"openclaw/plugin-sdk/testing\";

describe(\"my-channel target resolution\", () => {
  installCommonResolveTargetErrorCases({
    resolveTarget: ({ to, mode, allowFrom }) => {
      // Your channel\'s target resolution logic
      return myChannelResolveTarget({ to, mode, allowFrom });
    },
    implicitAllowFrom: [\"user1\", \"user2\"],
  });

  // Add channel-specific test cases
  it(\"should resolve @username targets\", () => {
    // ...
  });
});
```

## [​](https://docs.openclaw.ai/plugins/sdk-testing\\#testing-patterns)  Testing patterns

### [​](https://docs.openclaw.ai/plugins/sdk-testing\\#testing-registration-contracts)  Testing registration contracts

Unit tests that pass a hand-written `api` mock to `register(api)` do not exercise
OpenClaw’s loader acceptance gates. Add at least one loader-backed smoke test
for each registration surface your plugin depends on, especially hooks and
exclusive capabilities such as memory.The real loader fails plugin registration when required metadata is missing or a
plugin calls a capability API it does not own. For example,
`api.registerHook(...)` requires a hook name, and
`api.registerMemoryCapability(...)` requires the plugin manifest or exported
entry to declare `kind: \"memory\"`.

### [​](https://docs.openclaw.ai/plugins/sdk-testing\\#testing-runtime-config-access)  Testing runtime config access

Prefer the shared plugin runtime mock from the repo test helpers when testing
bundled plugins. Its deprecated `runtime.config.loadConfig()` and
`runtime.config.writeConfigFile(...)` mocks throw by default so tests catch new
usage of compatibility APIs. Override those mocks only when the test is
explicitly covering legacy compatibility behavior.

### [​](https://docs.openclaw.ai/plugins/sdk-testing\\#unit-testing-a-channel-plugin)  Unit testing a channel plugin

```
import { describe, it, expect, vi } from \"vitest\";

describe(\"my-channel plugin\", () => {
  it(\"should resolve account from config\", () => {
    const cfg = {
      channels: {
        \"my-channel\": {
          token: \"test-token\",
          allowFrom: [\"user1\"],
        },
      },
    };

    const account = myPlugin.setup.resolveAccount(cfg, undefined);
    expect(account.token).toBe(\"test-token\");
  });

  it(\"should inspect account without materializing secrets\", () => {
    const cfg = {
      channels: {
        \"my-channel\": { token: \"test-token\" },
      },
    };

    const inspection = myPlugin.setup.inspectAccount(cfg, undefined);
    expect(inspection.configured).toBe(true);
    expect(inspection.tokenStatus).toBe(\"available\");
    // No token value exposed
    expect(inspection).not.toHaveProperty(\"token\");
  });
});
```

### [​](https://docs.openclaw.ai/plugins/sdk-testing\\#unit-testing-a-provider-plugin)  Unit testing a provider plugin

```
import { describe, it, expect } from \"vitest\";

describe(\"my-provider plugin\", () => {
  it(\"should resolve dynamic models\", () => {
    const model = myProvider.resolveDynamicModel({
      modelId: \"custom-model-v2\",

---

## Webhooks plugin - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/webhooks

[Skip to main content](https://docs.openclaw.ai/plugins/webhooks#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Webhooks plugin

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Webhooks (plugin)](https://docs.openclaw.ai/plugins/webhooks#webhooks-plugin)
- [Where it runs](https://docs.openclaw.ai/plugins/webhooks#where-it-runs)
- [Configure routes](https://docs.openclaw.ai/plugins/webhooks#configure-routes)
- [Security model](https://docs.openclaw.ai/plugins/webhooks#security-model)
- [Request format](https://docs.openclaw.ai/plugins/webhooks#request-format)
- [Supported actions](https://docs.openclaw.ai/plugins/webhooks#supported-actions)
- [create\\_flow](https://docs.openclaw.ai/plugins/webhooks#create_flow)
- [run\\_task](https://docs.openclaw.ai/plugins/webhooks#run_task)
- [Response shape](https://docs.openclaw.ai/plugins/webhooks#response-shape)
- [Related docs](https://docs.openclaw.ai/plugins/webhooks#related-docs)

# [​](https://docs.openclaw.ai/plugins/webhooks\\#webhooks-plugin)  Webhooks (plugin)

The Webhooks plugin adds authenticated HTTP routes that bind external
automation to OpenClaw TaskFlows.Use it when you want a trusted system such as Zapier, n8n, a CI job, or an
internal service to create and drive managed TaskFlows without writing a custom
plugin first.

## [​](https://docs.openclaw.ai/plugins/webhooks\\#where-it-runs)  Where it runs

The Webhooks plugin runs inside the Gateway process.If your Gateway runs on another machine, install and configure the plugin on
that Gateway host, then restart the Gateway.

## [​](https://docs.openclaw.ai/plugins/webhooks\\#configure-routes)  Configure routes

Set config under `plugins.entries.webhooks.config`:

```
{
  plugins: {
    entries: {
      webhooks: {
        enabled: true,
        config: {
          routes: {
            zapier: {
              path: \"/plugins/webhooks/zapier\",
              sessionKey: \"agent:main:main\",
              secret: {
                source: \"env\",
                provider: \"default\",
                id: \"OPENCLAW_WEBHOOK_SECRET\",
              },
              controllerId: \"webhooks/zapier\",
              description: \"Zapier TaskFlow bridge\",
            },
          },
        },
      },
    },
  },
}
```

Route fields:

- `enabled`: optional, defaults to `true`
- `path`: optional, defaults to `/plugins/webhooks/<routeId>`
- `sessionKey`: required session that owns the bound TaskFlows
- `secret`: required shared secret or SecretRef
- `controllerId`: optional controller id for created managed flows
- `description`: optional operator note

Supported `secret` inputs:

- Plain string
- SecretRef with `source: \"env\" | \"file\" | \"exec\"`

If a secret-backed route cannot resolve its secret at startup, the plugin skips
that route and logs a warning instead of exposing a broken endpoint.

## [​](https://docs.openclaw.ai/plugins/webhooks\\#security-model)  Security model

Each route is trusted to act with the TaskFlow authority of its configured
`sessionKey`.This means the route can inspect and mutate TaskFlows owned by that session, so
you should:

- Use a strong unique secret per route
- Prefer secret references over inline plaintext secrets
- Bind routes to the narrowest session that fits the workflow
- Expose only the specific webhook path you need

The plugin applies:

- Shared-secret authentication
- Request body size and timeout guards
- Fixed-window rate limiting
- In-flight request limiting
- Owner-bound TaskFlow access through `api.runtime.taskFlow.bindSession(...)`

## [​](https://docs.openclaw.ai/plugins/webhooks\\#request-format)  Request format

Send `POST` requests with:

- `Content-Type: application/json`
- `Authorization: Bearer <secret>` or `x-openclaw-webhook-secret: <secret>`

Example:

```
curl -X POST https://gateway.example.com/plugins/webhooks/zapier \\
  -H 'Content-Type: application/json' \\
  -H 'Authorization: Bearer YOUR_SHARED_SECRET' \\
  -d '{\"action\":\"create_flow\",\"goal\":\"Review inbound queue\"}'
```

## [​](https://docs.openclaw.ai/plugins/webhooks\\#supported-actions)  Supported actions

The plugin currently accepts these JSON `action` values:

- `create_flow`
- `get_flow`
- `list_flows`
- `find_latest_flow`
- `resolve_flow`
- `get_task_summary`
- `set_waiting`
- `resume_flow`
- `finish_flow`
- `fail_flow`
- `request_cancel`
- `cancel_flow`
- `run_task`

### [​](https://docs.openclaw.ai/plugins/webhooks\\#create_flow)  `create_flow`

Creates a managed TaskFlow for the route’s bound session.Example:

```
{
  \"action\": \"create_flow\",
  \"goal\": \"Review inbound queue\",
  \"status\": \"queued\",
  \"notifyPolicy\": \"done_only\"
}
```

### [​](https://docs.openclaw.ai/plugins/webhooks\\#run_task)  `run_task`

Creates a managed child task inside an existing managed TaskFlow.Allowed runtimes are:

- `subagent`
- `acp`

Example:

```
{
  \"action\": \"run_task\",
  \"flowId\": \"flow_123\",
  \"runtime\": \"acp\",
  \"childSessionKey\": \"agent:main:acp:worker\",
  \"task\": \"Inspect the next message batch\"
}
```

## [​](https://docs.openclaw.ai/plugins/webhooks\\#response-shape)  Response shape

Successful responses return:

```
{
  \"ok\": true,
  \"routeId\": \"zapier\",
  \"result\": {}
}
```

Rejected requests return:

```
{
  \"ok\": false,
  \"routeId\": \"zapier\",
  \"code\": \"not_found\",
  \"error\": \"TaskFlow not found.\",
  \"result\": {}
}
```

The plugin intentionally scrubs owner/session metadata from webhook responses.

## [​](https://docs.openclaw.ai/plugins/webhooks\\#related-docs)  Related docs

- [Plugin runtime SDK](https://docs.openclaw.ai/plugins/sdk-runtime)
- [Hooks and webhooks overview](https://docs.openclaw.ai/automation/hooks)
- [CLI webhooks](https://docs.openclaw.ai/cli/webhooks)

[Google Meet plugin](https://docs.openclaw.ai/plugins/google-meet) [Voice call](https://docs.openclaw.ai/plugins/voice-call)

Ctrl+I

---

## Voice call plugin
**Source:** https://docs.openclaw.ai/plugins/voice-call

[Skip to main content](https://docs.openclaw.ai/plugins/voice-call#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Voice call plugin

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/plugins/voice-call#quick-start)
- [Configuration](https://docs.openclaw.ai/plugins/voice-call#configuration)
- [Realtime voice conversations](https://docs.openclaw.ai/plugins/voice-call#realtime-voice-conversations)
- [Tool policy](https://docs.openclaw.ai/plugins/voice-call#tool-policy)
- [Realtime provider examples](https://docs.openclaw.ai/plugins/voice-call#realtime-provider-examples)
- [Streaming transcription](https://docs.openclaw.ai/plugins/voice-call#streaming-transcription)
- [Streaming provider examples](https://docs.openclaw.ai/plugins/voice-call#streaming-provider-examples)
- [TTS for calls](https://docs.openclaw.ai/plugins/voice-call#tts-for-calls)
- [TTS examples](https://docs.openclaw.ai/plugins/voice-call#tts-examples)
- [Inbound calls](https://docs.openclaw.ai/plugins/voice-call#inbound-calls)
- [Spoken output contract](https://docs.openclaw.ai/plugins/voice-call#spoken-output-contract)
- [Conversation startup behavior](https://docs.openclaw.ai/plugins/voice-call#conversation-startup-behavior)
- [Twilio stream disconnect grace](https://docs.openclaw.ai/plugins/voice-call#twilio-stream-disconnect-grace)
- [Stale call reaper](https://docs.openclaw.ai/plugins/voice-call#stale-call-reaper)
- [Webhook security](https://docs.openclaw.ai/plugins/voice-call#webhook-security)
- [CLI](https://docs.openclaw.ai/plugins/voice-call#cli)
- [Agent tool](https://docs.openclaw.ai/plugins/voice-call#agent-tool)
- [Gateway RPC](https://docs.openclaw.ai/plugins/voice-call#gateway-rpc)
- [Related](https://docs.openclaw.ai/plugins/voice-call#related)

Voice calls for OpenClaw via a plugin. Supports outbound notifications,
multi-turn conversations, full-duplex realtime voice, streaming
transcription, and inbound calls with allowlist policies.**Current providers:**`twilio` (Programmable Voice + Media Streams),
`telnyx` (Call Control v2), `plivo` (Voice API + XML transfer + GetInput
speech), `mock` (dev/no network).

The Voice Call plugin runs **inside the Gateway process**. If you use a
remote Gateway, install and configure the plugin on the machine running
the Gateway, then restart the Gateway to load it.

## [​](https://docs.openclaw.ai/plugins/voice-call\\#quick-start)  Quick start

1

[Navigate to header](https://docs.openclaw.ai/plugins/voice-call#)

Install the plugin

- From npm (recommended)

- From a local folder (dev)


```
openclaw plugins install @openclaw/voice-call
```

```
PLUGIN_SRC=./path/to/local/voice-call-plugin
openclaw plugins install \"$PLUGIN_SRC\"
cd \"$PLUGIN_SRC\" && pnpm install
```

Restart the Gateway afterwards so the plugin loads.

2

[Navigate to header](https://docs.openclaw.ai/plugins/voice-call#)

Configure provider and webhook

Set config under `plugins.entries.voice-call.config` (see
[Configuration](https://docs.openclaw.ai/plugins/voice-call#configuration) below for the full shape). At minimum:
`provider`, provider credentials, `fromNumber`, and a publicly
reachable webhook URL.

3

[Navigate to header](https://docs.openclaw.ai/plugins/voice-call#)

Verify setup

```
openclaw voicecall setup
```

The default output is readable in chat logs and terminals. It checks
plugin enablement, provider credentials, webhook exposure, and that
only one audio mode (`streaming` or `realtime`) is active. Use
`--json` for scripts.

4

[Navigate to header](https://docs.openclaw.ai/plugins/voice-call#)

Smoke test

```
openclaw voicecall smoke
openclaw voicecall smoke --to \"+15555550123\"
```

Both are dry runs by default. Add `--yes` to actually place a short
outbound notify call:

```
openclaw voicecall smoke --to \"+15555550123\" --yes
```

For Twilio, Telnyx, and Plivo, setup must resolve to a **public webhook URL**.
If `publicUrl`, the tunnel URL, the Tailscale URL, or the serve fallback
resolves to loopback or private network space, setup fails instead of
starting a provider that cannot receive carrier webhooks.

## [​](https://docs.openclaw.ai/plugins/voice-call\\#configuration)  Configuration

If `enabled: true` but the selected provider is missing credentials,
Gateway startup logs a setup-incomplete warning with the missing keys and
skips starting the runtime. Commands, RPC calls, and agent tools still
return the exact missing provider configuration when used.

Voice-call credentials accept SecretRefs. `plugins.entries.voice-call.config.twilio.authToken` and `plugins.entries.voice-call.config.tts.providers.*.apiKey` resolve through the standard SecretRef surface; see [SecretRef credential surface](https://docs.openclaw.ai/reference/secretref-credential-surface).

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        enabled: true,
        config: {
          provider: \"twilio\", // or \"telnyx\" | \"plivo\" | \"mock\"
          fromNumber: \"+15550001234\", // or TWILIO_FROM_NUMBER for Twilio
          toNumber: \"+15550005678\",

          twilio: {
            accountSid: \"ACxxxxxxxx\",
            authToken: \"...\",
          },
          telnyx: {
            apiKey: \"...\",
            connectionId: \"...\",
            // Telnyx webhook public key from the Mission Control Portal
            // (Base64; can also be set via TELNYX_PUBLIC_KEY).
            publicKey: \"...\",
          },
          plivo: {
            authId: \"MAxxxxxxxxxxxxxxxxxxxx\",
            authToken: \"...\",
          },

          // Webhook server
          serve: {
            port: 3334,
            path: \"/voice/webhook\",
          },

          // Webhook security (recommended for tunnels/proxies)
          webhookSecurity: {
            allowedHosts: [\"voice.example.com\"],
            trustedProxyIPs: [\"100.64.0.1\"],
          },

          // Public exposure (pick one)
          // publicUrl: \"https://example.ngrok.app/voice/webhook\",
          // tunnel: { provider: \"ngrok\" },
          // tailscale: { mode: \"funnel\", path: \"/voice/webhook\" },

          outbound: {
            defaultMode: \"notify\", // notify | conversation
          },

          streaming: { enabled: true /* see Streaming transcription */ },
          realtime: { enabled: false /* see Realtime voice */ },
        },
      },
    },
  },
}
```

Provider exposure and security notes

- Twilio, Telnyx, and Plivo all require a **publicly reachable** webhook URL.
- `mock` is a local dev provider (no network calls).
- Telnyx requires `telnyx.publicKey` (or `TELNYX_PUBLIC_KEY`) unless `skipSignatureVerification` is true.
- `skipSignatureVerification` is for local testing only.
- On ngrok free tier, set `publicUrl` to the exact ngrok URL; signature verification is always enforced.
- `tunnel.allowNgrokFreeTierLoopbackBypass: true` allows Twilio webhooks with invalid signatures **only** when `tunnel.provider=\"ngrok\"` and `serve.bind` is loopback (ngrok local agent). Local dev only.
- Ngrok free-tier URLs can change or add interstitial behaviour; if `publicUrl` drifts, Twilio signatures fail. Production: prefer a stable domain or a Tailscale funnel.

Streaming connection caps

- `streaming.preStartTimeoutMs` closes sockets that never send a valid `start` frame.
- `streaming.maxPendingConnections` caps total unauthenticated pre-start sockets.
- `streaming.maxPendingConnectionsPerIp` caps unauthenticated pre-start sockets per source IP.
- `streaming.maxConnections` caps total open media stream sockets (pending + active).

Legacy config migrations

Older configs using `provider: \"log\"`, `twilio.from`, or legacy
`streaming.*` OpenAI keys are rewritten by `openclaw doctor --fix`.
Runtime fallback still accepts the old voice-call keys for now, but
the rewrite path is `openclaw doctor --fix` and the compat shim is
temporary.Auto-migrated streaming keys:

- `streaming.sttProvider` → `streaming.provider`
- `streaming.openaiApiKey` → `streaming.providers.openai.apiKey`
- `streaming.sttModel` → `streaming.providers.openai.model`
- `streaming.silenceDurationMs` → `streaming.providers.openai.silenceDurationMs`
- `streaming.vadThreshold` → `streaming.providers.openai.vadThreshold`

## [​](https://docs.openclaw.ai/plugins/voice-call\\#realtime-voice-conversations)  Realtime voice conversations

`realtime` selects a full-duplex realtime voice provider for live call
audio. It is separate from `streaming`, which only forwards audio to
realtime transcription providers.

`realtime.enabled` cannot be combined with `streaming.enabled`. Pick one
audio mode per call.

Current runtime behaviour:

- `realtime.enabled` is supported for Twilio Media Streams.
- `realtime.provider` is optional. If unset, Voice Call uses the first registered realtime voice provider.
- Bundled realtime voice providers: Google Gemini Live (`google`) and OpenAI (`openai`), registered by their provider plugins.
- Provider-owned raw config lives under `realtime.providers.<providerId>`.
- Voice Call exposes the shared `openclaw_agent_consult` realtime tool by default. The realtime model can call it when the caller asks for deeper reasoning, current information, or normal OpenClaw tools.
- If `realtime.provider` points at an unregistered provider, or no realtime voice provider is registered at all, Voice Call logs a warning and skips realtime media instead of failing the whole plugin.
- Consult session keys reuse the existing voice session when available, then fall back to the caller/callee phone number so follow-up consult calls keep context during the call.

### [​](https://docs.openclaw.ai/plugins/voice-call\\#tool-policy)  Tool policy

`realtime.toolPolicy` controls the consult run:

| Policy | Behavior |
| --- | --- |
| `safe-read-only` | Expose the consult tool and limit the regular agent to `read`, `web_search`, `web_fetch`, `x_search`, `memory_search`, and `memory_get`. |
| `owner` | Expose the consult tool and let the regular agent use the normal agent tool policy. |
| `none` | Do not expose the consult tool. Custom `realtime.tools` are still passed through to the realtime provider. |

### [​](https://docs.openclaw.ai/plugins/voice-call\\#realtime-provider-examples)  Realtime provider examples

- Google Gemini Live

- OpenAI


Defaults: API key from `realtime.providers.google.apiKey`,
`GEMINI_API_KEY`, or `GOOGLE_GENERATIVE_AI_API_KEY`; model
`gemini-2.5-flash-native-audio-preview-12-2025`; voice `Kore`.

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        config: {
          provider: \"twilio\",
          inboundPolicy: \"allowlist\",
          allowFrom: [\"+15550005678\"],
          inboundGreeting: \"Hello! How can I help?\",
          realtime: {
            enabled: true,
            provider: \"google\",
            instructions: \"Speak briefly. Call openclaw_agent_consult before using deeper tools.\",
            toolPolicy: \"safe-read-only\",
            providers: {
              google: {
                apiKey: \"${GEMINI_API_KEY}\",
                model: \"gemini-2.5-flash-native-audio-preview-12-2025\",
                voice: \"Kore\",
              },
            },
          },
        },
      },
    },
  },
}
```

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        config: {
          realtime: {
            enabled: true,
            provider: \"openai\",
            providers: {
              openai: { apiKey: \"${OPENAI_API_KEY}\" },
            },
          },
        },
      },
    },
  },
}
```

See [Google provider](https://docs.openclaw.ai/providers/google) and
[OpenAI provider](https://docs.openclaw.ai/providers/openai) for provider-specific realtime voice
options.

## [​](https://docs.openclaw.ai/plugins/voice-call\\#streaming-transcription)  Streaming transcription

`streaming` selects a realtime transcription provider for live call audio.Current runtime behavior:

- `streaming.provider` is optional. If unset, Voice Call uses the first registered realtime transcription provider.
- Bundled realtime transcription providers: Deepgram (`deepgram`), ElevenLabs (`elevenlabs`), Mistral (`mistral`), OpenAI (`openai`), and xAI (`xai`), registered by their provider plugins.
- Provider-owned raw config lives under `streaming.providers.<providerId>`.
- If `streaming.provider` points at an unregistered provider, or none is registered, Voice Call logs a warning and skips media streaming instead of failing the whole plugin.

### [​](https://openclaw.ai/plugins/voice-call\\#streaming-provider-examples)  Streaming provider examples

- OpenAI

- xAI


Defaults: API key `streaming.providers.openai.apiKey` or
`OPENAI_API_KEY`; model `gpt-4o-transcribe`; `silenceDurationMs: 800`;
`vadThreshold: 0.5`.

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        config: {
          streaming: {
            enabled: true,
            provider: \"openai\",
            streamPath: \"/voice/stream\",
            providers: {
              openai: {
                apiKey: \"sk-...\", // optional if OPENAI_API_KEY is set
                model: \"gpt-4o-transcribe\",
                silenceDurationMs: 800,
                vadThreshold: 0.5,
              },
            },
          },
        },
      },
    },
  },
}
```

Defaults: API key `streaming.providers.xai.apiKey` or `XAI_API_KEY`;
endpoint `wss://api.x.ai/v1/stt`; encoding `mulaw`; sample rate `8000`;
`endpointingMs: 800`; `interimResults: true`.

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        config: {
          streaming: {
            enabled: true,
            provider: \"xai\",
            streamPath: \"/voice/stream\",
            providers: {
              xai: {
                apiKey: \"${XAI_API_KEY}\", // optional if XAI_API_KEY is set
                endpointingMs: 800,
                language: \"en\",
              },
            },
          },
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/plugins/voice-call\\#tts-for-calls)  TTS for calls

Voice Call uses the core `messages.tts` configuration for streaming
speech on calls. You can override it under the plugin config with the
**same shape** — it deep-merges with `messages.tts`.

```
{
  tts: {
    provider: \"elevenlabs\",
    providers: {
      elevenlabs: {
        voiceId: \"pMsXgVXv3BLzUgSXRplE\",
        modelId: \"eleven_multilingual_v2\",
      },
    },
  },
}
```

**Microsoft speech is ignored for voice calls.** Telephony audio needs PCM;
the current Microsoft transport does not expose telephony PCM output.

Behavior notes:

- Legacy `tts.<provider>` keys inside plugin config (`openai`, `elevenlabs`, `microsoft`, `edge`) are repaired by `openclaw doctor --fix`; committed config should use `tts.providers.<provider>`.
- Core TTS is used when Twilio media streaming is enabled; otherwise calls fall back to provider-native voices.
- If a Twilio media stream is already active, Voice Call does not fall back to TwiML `<Say>`. If telephony TTS is unavailable in that state, the playback request fails instead of mixing two playback paths.
- When telephony TTS falls back to a secondary provider, Voice Call logs a warning with the provider chain (`from`, `to`, `attempts`) for debugging.
- When Twilio barge-in or stream teardown clears the pending TTS queue, queued playback requests settle instead of hanging callers awaiting playback completion.

### [​](https://docs.openclaw.ai/plugins/voice-call\\#tts-examples)  TTS examples

- Core TTS only

- Override to ElevenLabs (calls only)

- OpenAI model override (deep-merge)


```
{
  messages: {
    tts: {
      provider: \"openai\",
      providers: {
        openai: { voice: \"alloy\" },
      },
    },
  },
}
```

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        config: {
          tts: {
            provider: \"elevenlabs\",
            providers: {
              elevenlabs: {
                apiKey: \"elevenlabs_key\",
                voiceId: \"pMsXgVXv3BLzUgSXRplE\",
                modelId: \"eleven_multilingual_v2\",
              },
            },
          },
        },
      },
    },
  },
}
```

```
{
  plugins: {
    entries: {
      \"voice-call\": {
        config: {
          tts: {
            providers: {
              openai: {
                model: \"gpt-4o-mini-tts\",
                voice: \"marin\",
              },
            },
          },
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/plugins/voice-call\\#inbound-calls)  Inbound calls

Inbound policy defaults to `disabled`. To enable inbound calls, set:

```
{
  inboundPolicy: \"allowlist\",
  allowFrom: [\"+15550001234\"],
  inboundGreeting: \"Hello! How can I help?\",
}
```

`inboundPolicy: \"allowlist\"` is a low-assurance caller-ID screen. The
plugin normalizes the provider-supplied `From` value and compares it to
`allowFrom`. Webhook verification authenticates provider delivery and
payload integrity, but it does **not** prove PSTN/VoIP caller-number
ownership. Treat `allowFrom` as caller-ID filtering, not strong caller
identity.

Auto-responses use the agent system. Tune with `responseModel`,
`responseSystemPrompt`, and `responseTimeoutMs`.

### [​](https://docs.openclaw.ai/plugins/voice-call\\#spoken-output-contract)  Spoken output contract

For auto-responses, Voice Call appends a strict spoken-output contract to
the system prompt:

```
{\"spoken\":\"...\"}
```

Voice Call extracts speech text defensively:

- Ignores payloads marked as reasoning/error content.
- Parses direct JSON, fenced JSON, or inline `\"spoken\"` keys.
- Falls back to plain text and removes likely planning/meta lead-in paragraphs.

This keeps spoken playback focused on caller-facing text and avoids
leaking planning text into audio.

### [​](https://openclaw.ai/plugins/voice-call\\#conversation-startup-behavior)  Conversation startup behavior

For outbound `conversation` calls, first-message handling is tied to live
playback state:

- Barge-in queue clear and auto-response are suppressed only while the initial greeting is actively speaking.
- If initial playback fails, the call returns to `listening` and the initial message remains queued for retry.
- Initial playback for Twilio stre...(content truncated)

---

## Community plugins - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/community

[Skip to main content](https://docs.openclaw.ai/plugins/community#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Community plugins

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Listed plugins](https://docs.openclaw.ai/plugins/community#listed-plugins)
- [Apify](https://docs.openclaw.ai/plugins/community#apify)
- [Codex App Server Bridge](https://docs.openclaw.ai/plugins/community#codex-app-server-bridge)
- [DingTalk](https://docs.openclaw.ai/plugins/community#dingtalk)
- [Lossless Claw (LCM)](https://docs.openclaw.ai/plugins/community#lossless-claw-lcm)
- [Opik](https://docs.openclaw.ai/plugins/community#opik)
- [Prometheus Avatar](https://docs.openclaw.ai/plugins/community#prometheus-avatar)
- [QQbot](https://docs.openclaw.ai/plugins/community#qqbot)
- [wecom](https://docs.openclaw.ai/plugins/community#wecom)
- [Submit your plugin](https://docs.openclaw.ai/plugins/community#submit-your-plugin)
- [Quality bar](https://docs.openclaw.ai/plugins/community#quality-bar)
- [Related](https://docs.openclaw.ai/plugins/community#related)

Community plugins are third-party packages that extend OpenClaw with new
channels, tools, providers, or other capabilities. They are built and maintained
by the community, published on [ClawHub](https://docs.openclaw.ai/tools/clawhub) or npm, and
installable with a single command.ClawHub is the canonical discovery surface for community plugins. Do not open
docs-only PRs just to add your plugin here for discoverability; publish it on
ClawHub instead.

```
openclaw plugins install <package-name>
```

OpenClaw checks ClawHub first and falls back to npm automatically.

## [​](https://docs.openclaw.ai/plugins/community\#listed-plugins)  Listed plugins

### [​](https://docs.openclaw.ai/plugins/community\#apify)  Apify

Scrape data from any website with 20,000+ ready-made scrapers. Let your agent
extract data from Instagram, Facebook, TikTok, YouTube, Google Maps, Google
Search, e-commerce sites, and more — just by asking.

- **npm:**`@apify/apify-openclaw-plugin`
- **repo:** [github.com/apify/apify-openclaw-plugin](https://github.com/apify/apify-openclaw-plugin)

```
openclaw plugins install @apify/apify-openclaw-plugin
```

### [​](https://docs.openclaw.ai/plugins/community\#codex-app-server-bridge)  Codex App Server Bridge

Independent OpenClaw bridge for Codex App Server conversations. Bind a chat to
a Codex thread, talk to it with plain text, and control it with chat-native
commands for resume, planning, review, model selection, compaction, and more.

- **npm:**`openclaw-codex-app-server`
- **repo:** [github.com/pwrdrvr/openclaw-codex-app-server](https://github.com/pwrdrvr/openclaw-codex-app-server)

```
openclaw plugins install openclaw-codex-app-server
```

### [​](https://docs.openclaw.ai/plugins/community\#dingtalk)  DingTalk

Enterprise robot integration using Stream mode. Supports text, images, and
file messages via any DingTalk client.

- **npm:**`@largezhou/ddingtalk`
- **repo:** [github.com/largezhou/openclaw-dingtalk](https://github.com/largezhou/openclaw-dingtalk)

```
openclaw plugins install @largezhou/ddingtalk
```

### [​](https://docs.openclaw.ai/plugins/community\#lossless-claw-lcm)  Lossless Claw (LCM)

Lossless Context Management plugin for OpenClaw. DAG-based conversation
summarization with incremental compaction — preserves full context fidelity
while reducing token usage.

- **npm:**`@martian-engineering/lossless-claw`
- **repo:** [github.com/Martian-Engineering/lossless-claw](https://www.github.com/Martian-Engineering/lossless-claw)

```
openclaw plugins install @martian-engineering/lossless-claw
```

### [​](https://docs.openclaw.ai/plugins/community\#opik)  Opik

Official plugin that exports agent traces to Opik. Monitor agent behavior,
cost, tokens, errors, and more.

- **npm:**`@opik/opik-openclaw`
- **repo:** [github.com/comet-ml/opik-openclaw](https://github.com/comet-ml/opik-openclaw)

```
openclaw plugins install @opik/opik-openclaw
```

### [​](https://docs.openclaw.ai/plugins/community\#prometheus-avatar)  Prometheus Avatar

Give your OpenClaw agent a Live2D avatar with real-time lip-sync, emotion
expressions, and text-to-speech. Includes creator tools for AI asset generation
and one-click deployment to the Prometheus Marketplace. Currently in alpha.

- **npm:**`@prometheusavatar/openclaw-plugin`
- **repo:** [github.com/myths-labs/prometheus-avatar](https://github.com/myths-labs/prometheus-avatar)

```
openclaw plugins install @prometheusavatar/openclaw-plugin
```

### [​](https://docs.openclaw.ai/plugins/community\#qqbot)  QQbot

Connect OpenClaw to QQ via the QQ Bot API. Supports private chats, group
mentions, channel messages, and rich media including voice, images, videos,
and files.Current OpenClaw releases bundle QQ Bot. Use the bundled setup in
[QQ Bot](https://docs.openclaw.ai/channels/qqbot) for normal installs; install this external plugin only
when you intentionally want the Tencent-maintained standalone package.

- **npm:**`@tencent-connect/openclaw-qqbot`
- **repo:** [github.com/tencent-connect/openclaw-qqbot](https://github.com/tencent-connect/openclaw-qqbot)

```
openclaw plugins install @tencent-connect/openclaw-qqbot
```

### [​](https://docs.openclaw.ai/plugins/community\#wecom)  wecom

WeCom channel plugin for OpenClaw by the Tencent WeCom team. Powered by
WeCom Bot WebSocket persistent connections, it supports direct messages & group
chats, streaming replies, proactive messaging, image/file processing, Markdown
formatting, built-in access control, and document/meeting/messaging skills.

- **npm:**`@wecom/wecom-openclaw-plugin`
- **repo:** [github.com/WecomTeam/wecom-openclaw-plugin](https://github.com/WecomTeam/wecom-openclaw-plugin)

```
openclaw plugins install @wecom/wecom-openclaw-plugin
```

## [​](https://docs.openclaw.ai/plugins/community\#submit-your-plugin)  Submit your plugin

We welcome community plugins that are useful, documented, and safe to operate.

1

[Navigate to header](https://docs.openclaw.ai/plugins/community#)

Publish to ClawHub or npm

Your plugin must be installable via `openclaw plugins install \<package-name\>`.
Publish to [ClawHub](https://docs.openclaw.ai/tools/clawhub) (preferred) or npm.
See [Building Plugins](https://docs.openclaw.ai/plugins/building-plugins) for the full guide.

2

[Navigate to header](https://docs.openclaw.ai/plugins/community#)

Host on GitHub

Source code must be in a public repository with setup docs and an issue
tracker.

3

[Navigate to header](https://docs.openclaw.ai/plugins/community#)

Use docs PRs only for source-doc changes

You do not need a docs PR just to make your plugin discoverable. Publish it
on ClawHub instead.Open a docs PR only when OpenClaw’s source docs need an actual content
change, such as correcting install guidance or adding cross-repo
documentation that belongs in the main docs set.

## [​](https://docs.openclaw.ai/plugins/community\#quality-bar)  Quality bar

| Requirement | Why |
| --- | --- |
| Published on ClawHub or npm | Users need `openclaw plugins install` to work |
| Public GitHub repo | Source review, issue tracking, transparency |
| Setup and usage docs | Users need to know how to configure it |
| Active maintenance | Recent updates or responsive issue handling |

Low-effort wrappers, unclear ownership, or unmaintained packages may be declined.

## [​](https://docs.openclaw.ai/plugins/community\#related)  Related

- [Install and Configure Plugins](https://docs.openclaw.ai/tools/plugin) — how to install any plugin
- [Building Plugins](https://docs.openclaw.ai/plugins/building-plugins) — create your own
- [Plugin Manifest](https://docs.openclaw.ai/plugins/manifest) — manifest schema

[Install and Configure](https://docs.openclaw.ai/tools/plugin) [Plugin bundles](https://docs.openclaw.ai/plugins/bundles)

Ctrl+I

---

## Plugin SDK migration - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/sdk-migration

[Skip to main content](https://docs.openclaw.ai/plugins/sdk-migration#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Building plugins

Plugin SDK migration

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [What is changing](https://docs.openclaw.ai/plugins/sdk-migration#what-is-changing)
- [Why this changed](https://docs.openclaw.ai/plugins/sdk-migration#why-this-changed)
- [Compatibility policy](https://docs.openclaw.ai/plugins/sdk-migration#compatibility-policy)
- [How to migrate](https://docs.openclaw.ai/plugins/sdk-migration#how-to-migrate)
- [Import path reference](https://docs.openclaw.ai/plugins/sdk-migration#import-path-reference)
- [Active deprecations](https://docs.openclaw.ai/plugins/sdk-migration#active-deprecations)
- [Removal timeline](https://docs.openclaw.ai/plugins/sdk-migration#removal-timeline)
- [Suppressing the warnings temporarily](https://docs.openclaw.ai/plugins/sdk-migration#suppressing-the-warnings-temporarily)
- [Related](https://docs.openclaw.ai/plugins/sdk-migration#related)

OpenClaw has moved from a broad backwards-compatibility layer to a modern plugin
architecture with focused, documented imports. If your plugin was built before
the new architecture, this guide helps you migrate.

## [​](https://docs.openclaw.ai/plugins/sdk-migration\#what-is-changing)  What is changing

The old plugin system provided two wide-open surfaces that let plugins import
anything they needed from a single entry point:

- **`openclaw/plugin-sdk/compat`** — a single import that re-exported dozens of
helpers. It was introduced to keep older hook-based plugins working while the
new plugin architecture was being built.
- **`openclaw/extension-api`** — a bridge that gave plugins direct access to
host-side helpers like the embedded agent runner.
- **`api.registerEmbeddedExtensionFactory(...)`** — a removed Pi-only bundled
extension hook that could observe embedded-runner events such as
`tool_result`.

The broad import surfaces are now **deprecated**. They still work at runtime,
but new plugins must not use them, and existing plugins should migrate before
the next major release removes them. The Pi-only embedded extension factory
registration API has been removed; use tool-result middleware instead.OpenClaw does not remove or reinterpret documented plugin behavior in the same
change that introduces a replacement. Breaking contract changes must first go
through a compatibility adapter, diagnostics, docs, and a deprecation window.
That applies to SDK imports, manifest fields, setup APIs, hooks, and runtime
registration behavior.

The backwards-compatibility layer will be removed in a future major release.
Plugins that still import from these surfaces will break when that happens.
Pi-only embedded extension factory registrations already no longer load.

## [​](https://docs.openclaw.ai/plugins/sdk-migration\#why-this-changed)  Why this changed

The old approach caused problems:

- **Slow startup** — importing one helper loaded dozens of unrelated modules
- **Circular dependencies** — broad re-exports made it easy to create import cycles
- **Unclear API surface** — no way to tell which exports were stable vs internal

The modern plugin SDK fixes this: each import path (`openclaw/plugin-sdk/\<subpath\>`)
is a small, self-contained module with a clear purpose and documented contract.Legacy provider convenience seams for bundled channels are also gone. Imports
such as `openclaw/plugin-sdk/slack`, `openclaw/plugin-sdk/discord`,
`openclaw/plugin-sdk/signal`, `openclaw/plugin-sdk/whatsapp`,
channel-branded helper seams, and
`openclaw/plugin-sdk/telegram-core` were private mono-repo shortcuts, not
stable plugin contracts. Use narrow generic SDK subpaths instead. Inside the
bundled plugin workspace, keep provider-owned helpers in that plugin’s own
`api.ts` or `runtime-api.ts`.Current bundled provider examples:

- Anthropic keeps Claude-specific stream helpers in its own `api.ts` /
`contract-api.ts` seam
- OpenAI keeps provider builders, default-model helpers, and realtime provider
builders in its own `api.ts`
- OpenRouter keeps provider builder and onboarding/config helpers in its own
`api.ts`

## [​](https://docs.openclaw.ai/plugins/sdk-migration\#compatibility-policy)  Compatibility policy

For external plugins, compatibility work follows this order:

1. add the new contract
2. keep the old behavior wired through a compatibility adapter
3. emit a diagnostic or warning that names the old path and replacement
4. cover both paths in tests
5. document the deprecation and migration path
6. remove only after the announced migration window, usually in a major release

If a manifest field is still accepted, plugin authors can keep using it until
the docs and diagnostics say otherwise. New code should prefer the documented
replacement, but existing plugins should not break during ordinary minor
releases.

## [​](https://docs.openclaw.ai/plugins/sdk-migration\#how-to-migrate)  How to migrate

1

[Navigate to header](https://docs.openclaw.ai/plugins/sdk-migration#)

Migrate Pi tool-result extensions to middleware

Bundled plugins must replace Pi-only
`api.registerEmbeddedExtensionFactory(...)` tool-result handlers with
runtime-neutral middleware.

```
// Pi and Codex runtime dynamic tools
api.registerAgentToolResultMiddleware(async (event) => {
  return compactToolResult(event);
}, {
  runtimes: ["pi", "codex"],
});
```

Update the plugin manifest at the same time:

```
{
  "contracts": {
    "agentToolResultMiddleware": ["pi", "codex"]
  }
}
```

External plugins cannot register tool-result middleware because it can
rewrite high-trust tool output before the model sees it.

2

[Navigate to header](https://docs.openclaw.ai/plugins/sdk-migration#)

Migrate approval-native handlers to capability facts

Approval-capable channel plugins now expose native approval behavior through
`approvalCapability.nativeRuntime` plus the shared runtime-context registry.Key changes:

- Replace `approvalCapability.handler.loadRuntime(...)` with
`approvalCapability.nativeRuntime`
- Move approval-specific auth/delivery off legacy `plugin.auth` /
`plugin.approvals` wiring and onto `approvalCapability`
- `ChannelPlugin.approvals` has been removed from the public channel-plugin
contract; move delivery/native/render fields onto `approvalCapability`
- `plugin.auth` remains for channel login/logout flows only; approval auth
hooks there are no longer read by core
- Register channel-owned runtime objects such as clients, tokens, or Bolt
apps through `openclaw/plugin-sdk/channel-runtime-context`
- Do not send plugin-owned reroute notices from native approval handlers;
core now owns routed-elsewhere notices from actual delivery results
- When passing `channelRuntime` into `createChannelManager(...)`, provide a
real `createPluginRuntime().channel` surface. Partial stubs are rejected.

See `/plugins/sdk-channel-plugins` for the current approval capability
layout.

3

[Navigate to header](https://docs.openclaw.ai/plugins/sdk-migration#)

Audit Windows wrapper fallback behavior

If your plugin uses `openclaw/plugin-sdk/windows-spawn`, unresolved Windows
`.cmd`/`.bat` wrappers now fail closed unless you explicitly pass
`allowShellFallback: true`.

```
// Before
const program = applyWindowsSpawnProgramPolicy({ candidate });

// After
const program = applyWindowsSpawnProgramPolicy({
  candidate,
  // Only set this for trusted compatibility callers that intentionally
  // accept shell-mediated fallback.
  allowShellFallback: true,
});
```

If your caller does not intentionally rely on shell fallback, do not set
`allowShellFallback` and handle the thrown error instead.

4

[Navigate to header](https://docs.openclaw.ai/plugins/sdk-migration#)

Find deprecated imports

Search your plugin for imports from either deprecated surface:

```
grep -r "plugin-sdk/compat" my-plugin/
grep -r "openclaw/extension-api" my-plugin/
```

5

[Navigate to header](https://docs.openclaw.ai/plugins/sdk-migration#)

Replace with focused imports

Each export from the old surface maps to a specific modern import path:

```
// Before (deprecated backwards-compatibility layer)
import {
  createChannelReplyPipeline,
  createPluginRuntimeStore,
  resolveControlCommandGate,
} from "openclaw/plugin-sdk/compat";

// After (modern focused imports)
import { createChannelReplyPipeline } from "openclaw/plugin-sdk/channel-reply-pipeline";
import { createPluginRuntimeStore } from "openclaw/plugin-sdk/runtime-store";
import { resolveControlCommandGate } from "openclaw/plugin-sdk/command-auth";
```

For host-side helpers, use the injected plugin runtime instead of importing
directly:

```
// Before (deprecated extension-api bridge)
import { runEmbeddedPiAgent } from "openclaw/extension-api";
const result = await runEmbeddedPiAgent({ sessionId, prompt });

// After (injected runtime)
const result = await api.runtime.agent.runEmbeddedPiAgent({ sessionId, prompt });
```

The same pattern applies to other legacy bridge helpers:

| Old import | Modern equivalent |
| --- | --- |
| `resolveAgentDir` | `api.runtime.agent.resolveAgentDir` |
| `resolveAgentWorkspaceDir` | `api.runtime.agent.resolveAgentWorkspaceDir` |
| `resolveAgentIdentity` | `api.runtime.agent.resolveAgentIdentity` |
| `resolveThinkingDefault` | `api.runtime.agent.resolveThinkingDefault` |
| `resolveAgentTimeoutMs` | `api.runtime.agent.resolveAgentTimeoutMs` |
| `ensureAgentWorkspace` | `api.runtime.agent.ensureAgentWorkspace` |
| session store helpers | `api.runtime.agent.session.*` |

6

[Navigate to header](https://docs.openclaw.ai/plugins/sdk-migration#)

Build and test

```
pnpm build
pnpm test -- my-plugin/
```

## [​](https://docs.openclaw.ai/plugins/sdk-migration\#import-path-reference)  Import path reference

Common import path table

| Import path | Purpose | Key exports |
| --- | --- | --- |
| `plugin-sdk/plugin-entry` | Canonical plugin entry helper | `definePluginEntry` |
| `plugin-sdk/core` | Legacy umbrella re-export for channel entry definitions/builders | `defineChannelPluginEntry`, `createChatChannelPlugin` |
| `plugin-sdk/config-schema` | Root config schema export | `OpenClawSchema` |
| `plugin-sdk/provider-entry` | Single-provider entry helper | `defineSingleProviderPluginEntry` |
| `plugin-sdk/channel-core` | Focused channel entry definitions and builders | `defineChannelPluginEntry`, `defineSetupPluginEntry`, `createChatChannelPlugin`, `createChannelPluginBase` |
| `plugin-sdk/setup` | Shared setup wizard helpers | Allowlist prompts, setup status builders |
| `plugin-sdk/setup-runtime` | Setup-time runtime helpers | Import-safe setup patch adapters, lookup-note helpers, `promptResolvedAllowFrom`, `splitSetupEntries`, delegated setup proxies |
| `plugin-sdk/setup-adapter-runtime` | Setup adapter helpers | `createEnvPatchedAccountSetupAdapter` |
| `plugin-sdk/setup-tools` | Setup tooling helpers | `formatCliCommand`, `detectBinary`, `extractArchive`, `resolveBrewExecutable`, `formatDocsLink`, `CONFIG_DIR` |
| `plugin-sdk/account-core` | Multi-account helpers | Account list/config/action-gate helpers |
| `plugin-sdk/account-id` | Account-id helpers | `DEFAULT_ACCOUNT_ID`, account-id normalization |
| `plugin-sdk/account-resolution` | Account lookup helpers | Account lookup + default-fallback helpers |
| `plugin-sdk/account-helpers` | Narrow account helpers | Account list/account-action helpers |
| `plugin-sdk/channel-setup` | Setup wizard adapters | `createOptionalChannelSetupSurface`, `createOptionalChannelSetupAdapter`, `createOptionalChannelSetupWizard`, plus `DEFAULT_ACCOUNT_ID`, `createTopLevelChannelDmPolicy`, `setSetupChannelEnabled`, `splitSetupEntries` |
| `plugin-sdk/channel-pairing` | DM pairing primitives | `createChannelPairingController` |
| `plugin-sdk/channel-reply-pipeline` | Reply prefix + typing wiring | `createChannelReplyPipeline` |
| `plugin-sdk/channel-config-helpers` | Config adapter factories | `createHybridChannelConfigAdapter` |
| `plugin-sdk/channel-config-schema` | Config schema builders | Shared channel config schema primitives; bundled-channel-named schema exports are legacy compatibility only |
| `plugin-sdk/telegram-command-config` | Telegram command config helpers | Command-name normalization, description trimming, duplicate/conflict validation |
| `plugin-sdk/channel-policy` | Group/DM policy resolution | `resolveChannelGroupRequireMention` |
| `plugin-sdk/channel-lifecycle` | Account status and draft stream lifecycle helpers | `createAccountStatusSink`, draft preview finalization helpers |
| `plugin-sdk/inbound-envelope` | Inbound envelope helpers | Shared route + envelope builder helpers |
| `plugin-sdk/inbound-reply-dispatch` | Inbound reply helpers | Shared record-and-dispatch helpers |
| `plugin-sdk/messaging-targets` | Messaging target parsing | Target parsing/matching helpers |
| `plugin-sdk/outbound-media` | Outbound media helpers | Shared outbound media loading |
| `plugin-sdk/outbound-send-deps` | Outbound send dependency helpers | Lightweight `resolveOutboundSendDep` lookup without importing the full outbound runtime |
| `plugin-sdk/outbound-runtime` | Outbound runtime helpers | Outbound delivery, identity/send delegate, session, formatting, and payload planning helpers |
| `plugin-sdk/thread-bindings-runtime` | Thread-binding helpers | Thread-binding lifecycle and adapter helpers |
| `plugin-sdk/agent-media-payload` | Legacy media payload helpers | Agent media payload builder for legacy field layouts |
| `plugin-sdk/channel-runtime` | Deprecated compatibility shim | Legacy channel runtime utilities only |
| `plugin-sdk/channel-send-result` | Send result types | Reply result types |
| `plugin-sdk/runtime-store` | Persistent plugin storage | `createPluginRuntimeStore` |
| `plugin-sdk/runtime` | Broad runtime helpers | Runtime/logging/backup/plugin-install helpers |
| `plugin-sdk/runtime-env` | Narrow runtime env helpers | Logger/runtime env, timeout, retry, and backoff helpers |
| `plugin-sdk/plugin-runtime` | Shared plugin runtime helpers | Plugin commands/hooks/http/interactive helpers |
| `plugin-sdk/hook-runtime` | Hook pipeline helpers | Shared webhook/internal hook pipeline helpers |
| `plugin-sdk/lazy-runtime` | Lazy runtime helpers | `createLazyRuntimeModule`, `createLazyRuntimeMethod`, `createLazyRuntimeMethodBinder`, `createLazyRuntimeNamedExport`, `createLazyRuntimeSurface` |
| `plugin-sdk/process-runtime` | Process helpers | Shared exec helpers |
| `plugin-sdk/cli-runtime` | CLI runtime helpers | Command formatting, waits, version helpers |
| `plugin-sdk/gateway-runtime` | Gateway helpers | Gateway client and channel-status patch helpers |
| `plugin-sdk/config-runtime` | Config helpers | Config load/write helpers |
| `plugin-sdk/telegram-command-config` | Telegram command helpers | Fallback-stable Telegram command validation helpers when the bundled Telegram contract surface is unavailable |
| `plugin-sdk/approval-runtime` | Approval prompt helpers | Exec/plugin approval payload, approval capability/profile helpers, native approval routing/runtime helpers, and structured approval display path formatting |
| `plugin-sdk/approval-auth-runtime` | Approval auth helpers | Approver resolution, same-chat action auth |
| `plugin-sdk/approval-client-runtime` | Approval client helpers | Native exec approval profile/filter helpers |
| `plugin-sdk/approval-delivery-runtime` | Approval delivery helpers | Native approval capability/delivery adapters |
| `plugin-sdk/approval-gateway-runtime` | Approval gateway helpers | Shared approval gateway-resolution helper |
| `plugin-sdk/approval-handler-adapter-runtime` | Approval adapter helpers | Lightweight native approval adapter loading helpers for hot channel entrypoints |
| `plugin-sdk/approval-handler-runtime` | Approval handler helpers | Broader approval handler runtime helpers; prefer the narrower adapter/gateway seams when they are enough |
| `plugin-sdk/approval-native-runtime` | Approval target helpers | Native approval target/account binding helpers |
| `plugin-sdk/approval-reply-runtime` | Approval reply helpers | Exec/plugin approval reply payload helpers |
| `plugin-sdk/channel-runtime-context` | Channel runtime-context helpers | Generic channel runtime-context register/get/watch helpers |
| `plugin-sdk/security-runtime` | Security helpers | Shared trust, DM gating, external-content, and secret-collection helpers |
| `plugin-sdk/ssrf-policy` | SSRF policy helpers | Host allowlist and private-network policy helpers |
| `plugin-sdk/ssrf-runtime` | SSRF runtime helpers | Pinned-dispatcher, guarded fetch, SSRF policy helpers |
| `plugin-sdk/collection-runtime` | Bounded cache helpers | `pruneMapToMaxSize` |
| `plugin-sdk/diagnostic-runtime` | Diagnostic gating helpers | `isDiagnosticFlagEnabled`, `isDiagnosticsEnabled` |
| `plugin-sdk/error-runtime` | Error formatting helpers | `formatUncaughtError`, `isApprovalNotFoundError`, error graph helpers |
| `plugin-sdk/fetch-runtime` | Wrapped fetch/proxy helpers | `resolveFetch`, proxy helpers |
| `plugin-sdk/host-runtime` | Host normalization helpers | `normalizeHostname`, `normalizeScpRemoteHost` |
| `plugin-sdk/retry-runtime` | Retry helpers | `RetryConfig`, `retryAsync`, policy runners |
| `plugin-sdk/allow-from` | Allowlist formatting | `formatAllowFromLowercase` |
| `plugin-sdk/allowlist-resolution` | Allowlist input mapping | `mapAllowlistResolutionInputs` |
| `plugin-sdk/command-auth` | Command gating and command-surface helpers | `resolveControlCommandGate`, sender-authorization helpers, command registry helpers including dynamic argument menu formatting |
| `plugin-sdk/command-status` | Command status/help renderers | `buildCommandsMessage`, `buildCommandsMessagePaginated`, `buildHelpMessage` |
| `plugin-sdk/secret-input` | Secret input parsing | Secret input helpers |
| `plugin-sdk/webhook-ingress` | Webhook request helpers | Webhook target utilities |
| `plugin-sdk/webhook-request-guards` | Webhook body guard helpers | Request body read/limit helpers |
| `plugin-sdk/reply-runtime` | Shared reply runtime | Inbound dispatch, heartbeat, reply planner, chunking |
| `plugin-sdk/reply-dispatch-runtime` | Narrow reply dispatch helpers | Finalize, provider dispatch, and conversation-label helpers |
| `plugin-sdk/reply-history` | Reply-history helpers | `buildHistoryContext`, `buildPendingHistoryContextFromMap`, `record...(content truncated)

---

## Google Meet plugin
**Source:** https://docs.openclaw.ai/plugins/google-meet

[Skip to main content](https://docs.openclaw.ai/plugins/google-meet#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Google Meet plugin

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/plugins/google-meet#quick-start)
- [Local gateway + Parallels Chrome](https://docs.openclaw.ai/plugins/google-meet#local-gateway-%2B-parallels-chrome)
- [Install notes](https://docs.openclaw.ai/plugins/google-meet#install-notes)
- [Transports](https://docs.openclaw.ai/plugins/google-meet#transports)
- [Chrome](https://docs.openclaw.ai/plugins/google-meet#chrome)
- [Twilio](https://docs.openclaw.ai/plugins/google-meet#twilio)
- [OAuth and preflight](https://docs.openclaw.ai/plugins/google-meet#oauth-and-preflight)
- [Create Google credentials](https://docs.openclaw.ai/plugins/google-meet#create-google-credentials)
- [Mint the refresh token](https://docs.openclaw.ai/plugins/google-meet#mint-the-refresh-token)
- [Verify OAuth with doctor](https://docs.openclaw.ai/plugins/google-meet#verify-oauth-with-doctor)
- [Config](https://docs.openclaw.ai/plugins/google-meet#config)
- [Tool](https://docs.openclaw.ai/plugins/google-meet#tool)
- [Realtime agent consult](https://docs.openclaw.ai/plugins/google-meet#realtime-agent-consult)
- [Live test checklist](https://docs.openclaw.ai/plugins/google-meet#live-test-checklist)
- [Troubleshooting](https://docs.openclaw.ai/plugins/google-meet#troubleshooting)
- [Agent cannot see the Google Meet tool](https://docs.openclaw.ai/plugins/google-meet#agent-cannot-see-the-google-meet-tool)
- [No connected Google Meet-capable node](https://docs.openclaw.ai/plugins/google-meet#no-connected-google-meet-capable-node)
- [Browser opens but agent cannot join](https://docs.openclaw.ai/plugins/google-meet#browser-opens-but-agent-cannot-join)
- [Meeting creation fails](https://docs.openclaw.ai/plugins/google-meet#meeting-creation-fails)
- [Agent joins but does not talk](https://docs.openclaw.ai/plugins/google-meet#agent-joins-but-does-not-talk)
- [Twilio setup checks fail](https://docs.openclaw.ai/plugins/google-meet#twilio-setup-checks-fail)
- [Twilio call starts but never enters the meeting](https://docs.openclaw.ai/plugins/google-meet#twilio-call-starts-but-never-enters-the-meeting)
- [Notes](https://docs.openclaw.ai/plugins/google-meet#notes)
- [Related](https://docs.openclaw.ai/plugins/google-meet#related)

Google Meet participant support for OpenClaw — the plugin is explicit by design:

- It only joins an explicit `https://meet.google.com/...` URL.
- It can create a new Meet space through the Google Meet API, then join the
returned URL.
- `realtime` voice is the default mode.
- Realtime voice can call back into the full OpenClaw agent when deeper
reasoning or tools are needed.
- Agents choose the join behavior with `mode`: use `realtime` for live
listen/talk-back, or `transcribe` to join/control the browser without the
realtime voice bridge.
- Auth starts as personal Google OAuth or an already signed-in Chrome profile.
- There is no automatic consent announcement.
- The default Chrome audio backend is `BlackHole 2ch`.
- Chrome can run locally or on a paired node host.
- Twilio accepts a dial-in number plus optional PIN or DTMF sequence.
- The CLI command is `googlemeet`; `meet` is reserved for broader agent
teleconference workflows.

## [​](https://docs.openclaw.ai/plugins/google-meet\\#quick-start)  Quick start

Install the local audio dependencies and configure a backend realtime voice
provider. OpenAI is the default; Google Gemini Live also works with
`realtime.provider: \"google\"`:

```
brew install blackhole-2ch sox
export OPENAI_API_KEY=sk-...
# or
export GEMINI_API_KEY=...
```

`blackhole-2ch` installs the `BlackHole 2ch` virtual audio device. Homebrew’s
installer requires a reboot before macOS exposes the device:

```
sudo reboot
```

After reboot, verify both pieces:

```
system_profiler SPAudioDataType | grep -i BlackHole
command -v sox
```

Enable the plugin:

```
{
  plugins: {
    entries: {
      \"google-meet\": {
        enabled: true,
        config: {},
      },
    },
  },
}
```

Check setup:

```
openclaw googlemeet setup
```

The setup output is meant to be agent-readable and mode-aware. It reports Chrome
profile, node pinning, and, for realtime Chrome joins, the BlackHole/SoX audio
bridge and delayed realtime intro checks. For observe-only joins, check the same
transport with `--mode transcribe`; that mode skips realtime audio prerequisites
because it does not listen through or speak through the bridge:

```
openclaw googlemeet setup --transport chrome-node --mode transcribe
```

When Twilio delegation is configured, setup also reports whether the
`voice-call` plugin and Twilio credentials are ready. Treat any `ok: false`
check as a blocker for the checked transport and mode before asking an agent to
join. Use `openclaw googlemeet setup --json` for scripts or machine-readable
output. Use `--transport chrome`, `--transport chrome-node`, or `--transport twilio`
to preflight a specific transport before an agent tries it.Join a meeting:

```
openclaw googlemeet join https://meet.google.com/abc-defg-hij
```

Or let an agent join through the `google_meet` tool:

```
{
  \"action\": \"join\",
  \"url\": \"https://meet.google.com/abc-defg-hij\",
  \"transport\": \"chrome-node\",
  \"mode\": \"realtime\"
}
```

Create a new meeting and join it:

```
openclaw googlemeet create --transport chrome-node --mode realtime
```

Create only the URL without joining:

```
openclaw googlemeet create --no-join
```

`googlemeet create` has two paths:

- API create: used when Google Meet OAuth credentials are configured. This is
the most deterministic path and does not depend on browser UI state.
- Browser fallback: used when OAuth credentials are absent. OpenClaw uses the
pinned Chrome node, opens `https://meet.google.com/new`, waits for Google to
redirect to a real meeting-code URL, then returns that URL. This path requires
the OpenClaw Chrome profile on the node to already be signed in to Google.
Browser automation handles Meet’s own first-run microphone prompt; that prompt
is not treated as a Google login failure.
Join and create flows also try to reuse an existing Meet tab before opening a
new one. Matching ignores harmless URL query strings such as `authuser`, so an
agent retry should focus the already-open meeting instead of creating a second
Chrome tab.

The command/tool output includes a `source` field (`api` or `browser`) so agents
can explain which path was used. `create` joins the new meeting by default and
returns `joined: true` plus the join session. To only mint the URL, use
`create --no-join` on the CLI or pass `\"join\": false` to the tool.Or tell an agent: “Create a Google Meet, join it with realtime voice, and send
me the link.” The agent should call `google_meet` with `action: \"create\"` and
then share the returned `meetingUri`.

```
{
  \"action\": \"create\",
  \"transport\": \"chrome-node\",
  \"mode\": \"realtime\"
}
```

For an observe-only/browser-control join, set `\"mode\": \"transcribe\"`. That does
not start the duplex realtime model bridge, does not require BlackHole or SoX,
and will not talk back into the meeting. Chrome joins in this mode also avoid
OpenClaw’s microphone/camera permission grant and avoid the Meet **Use**
**microphone** path. If Meet shows an audio-choice interstitial, automation tries
the no-microphone path and otherwise reports a manual action instead of opening
the local microphone.During realtime sessions, `google_meet` status includes browser and audio bridge
health such as `inCall`, `manualActionRequired`, `providerConnected`,
`realtimeReady`, `audioInputActive`, `audioOutputActive`, last input/output
timestamps, byte counters, and bridge closed state. If a safe Meet page prompt
appears, browser automation handles it when it can. Login, host admission, and
browser/OS permission prompts are reported as manual action with a reason and
message for the agent to relay.Local Chrome joins through the signed-in OpenClaw browser profile. Realtime mode
requires `BlackHole 2ch` for the microphone/speaker path used by OpenClaw. For
clean duplex audio, use separate virtual devices or a Loopback-style graph; a
single BlackHole device is enough for a first smoke test but can echo.

### [​](https://docs.openclaw.ai/plugins/google-meet\\#local-gateway-+-parallels-chrome)  Local gateway + Parallels Chrome

You do **not** need a full OpenClaw Gateway or model API key inside a macOS VM
just to make the VM own Chrome. Run the Gateway and agent locally, then run a
node host in the VM. Enable the bundled plugin on the VM once so the node
advertises the Chrome command:What runs where:

- Gateway host: OpenClaw Gateway, agent workspace, model/API keys, realtime
provider, and the Google Meet plugin config.
- Parallels macOS VM: OpenClaw CLI/node host, Google Chrome, SoX, BlackHole 2ch,
and a Chrome profile signed in to Google.
- Not needed in the VM: Gateway service, agent config, OpenAI/GPT key, or model
provider setup.

Install the VM dependencies:

```
brew install blackhole-2ch sox
```

Reboot the VM after installing BlackHole so macOS exposes `BlackHole 2ch`:

```
sudo reboot
```

After reboot, verify the VM can see the audio device and SoX commands:

```
system_profiler SPAudioDataType | grep -i BlackHole
command -v sox
```

Install or update OpenClaw in the VM, then enable the bundled plugin there:

```
openclaw plugins enable google-meet
```

Start the node host in the VM:

```
openclaw node run --host <gateway-host> --port 18789 --display-name parallels-macos
```

If `<gateway-host>` is a LAN IP and you are not using TLS, the node refuses the
plaintext WebSocket unless you opt in for that trusted private network:

```
OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1 \\
  openclaw node run --host <gateway-lan-ip> --port 18789 --display-name parallels-macos
```

Use the same environment variable when installing the node as a LaunchAgent:

```
OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1 \\
  openclaw node install --host <gateway-lan-ip> --port 18789 --display-name parallels-macos --force
openclaw node restart
```

`OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` is process environment, not an
`openclaw.json` setting. `openclaw node install` stores it in the LaunchAgent
environment when it is present on the install command.Approve the node from the Gateway host:

```
openclaw devices list
openclaw devices approve <requestId>
```

Confirm the Gateway sees the node and that it advertises both `googlemeet.chrome`
and browser capability/`browser.proxy`:

```
openclaw nodes status
```

Route Meet through that node on the Gateway host:

```
{
  gateway: {
    nodes: {
      allowCommands: [\"googlemeet.chrome\", \"browser.proxy\"],
    },
  },
  plugins: {
    entries: {
      \"google-meet\": {
        enabled: true,
        config: {
          defaultTransport: \"chrome-node\",
          chrome: {
            guestName: \"OpenClaw Agent\",
            autoJoin: true,
            reuseExistingTab: true,
          },
          chromeNode: {
            node: \"parallels-macos\",
          },
        },
      },
    },
  },
}
```

Now join normally from the Gateway host:

```
openclaw googlemeet join https://meet.google.com/abc-defg-hij
```

or ask the agent to use the `google_meet` tool with `transport: \"chrome-node\"`.For a one-command smoke test that creates or reuses a session, speaks a known
phrase, and prints session health:

```
openclaw googlemeet test-speech https://meet.google.com/abc-defg-hij
```

During realtime join, OpenClaw browser automation fills the guest name, clicks
Join/Ask to join, and accepts Meet’s first-run “Use microphone” choice when that
prompt appears. During observe-only join or browser-only meeting creation, it
continues past the same prompt without microphone when that choice is available.
If the browser profile is not signed in, Meet is waiting for host admission,
Chrome needs microphone/camera permission for a realtime join, or Meet is stuck
on a prompt automation could not resolve, the join/test-speech result reports
`manualActionRequired: true` with `manualActionReason` and
`manualActionMessage`. Agents should stop retrying the join, report that exact
message plus the current `browserUrl`/`browserTitle`, and retry only after the
manual browser action is complete.If `chromeNode.node` is omitted, OpenClaw auto-selects only when exactly one
connected node advertises both `googlemeet.chrome` and browser control. If
several capable nodes are connected, set `chromeNode.node` to the node id,
display name, or remote IP.Common failure checks:

- `Configured Google Meet node ... is not usable: offline`: the pinned node is
known to the Gateway but unavailable. Agents should treat that node as
diagnostic state, not as a usable Chrome host, and report the setup blocker
instead of falling back to another transport unless the user asked for that.
- `No connected Google Meet-capable node`: start `openclaw node run` in the VM,
approve pairing, and make sure `openclaw plugins enable google-meet` and
`openclaw plugins enable browser` were run in the VM. Also confirm the
Gateway host allows both node commands with
`gateway.nodes.allowCommands: [\"googlemeet.chrome\", \"browser.proxy\"]`.
- `BlackHole 2ch audio device not found`: install `blackhole-2ch` on the host
being checked and reboot before using local Chrome audio.
- `BlackHole 2ch audio device not found on the node`: install `blackhole-2ch`
in the VM and reboot the VM.
- Chrome opens but cannot join: sign in to the browser profile inside the VM, or
keep `chrome.guestName` set for guest join. Guest auto-join uses OpenClaw
browser automation through the node browser proxy; make sure the node browser
config points at the profile you want, for example
`browser.defaultProfile: \"user\"` or a named existing-session profile.
- Duplicate Meet tabs: leave `chrome.reuseExistingTab: true` enabled. OpenClaw
activates an existing tab for the same Meet URL before opening a new one, and
browser meeting creation reuses an in-progress `https://meet.google.com/new`
or Google account prompt tab before opening another one.
- No audio: in Meet, route microphone/speaker through the virtual audio device
path used by OpenClaw; use separate virtual devices or Loopback-style routing
for clean duplex audio.

## [​](https://docs.openclaw.ai/plugins/google-meet\\#install-notes)  Install notes

The Chrome realtime default uses two external tools:

- `sox`: command-line audio utility. The plugin uses explicit CoreAudio
device commands for the default 24 kHz PCM16 audio bridge.
- `blackhole-2ch`: macOS virtual audio driver. It creates the `BlackHole 2ch`
audio device that Chrome/Meet can route through.

OpenClaw does not bundle or redistribute either package. The docs ask users to
install them as host dependencies through Homebrew. SoX is licensed as
`LGPL-2.0-only AND GPL-2.0-only`; BlackHole is GPL-3.0. If you build an
installer or appliance that bundles BlackHole with OpenClaw, review BlackHole’s
upstream licensing terms or get a separate license from Existential Audio.

## [​](https://docs.openclaw.ai/plugins/google-meet\\#transports)  Transports

### [​](https://docs.openclaw.ai/plugins/google-meet\\#chrome)  Chrome

Chrome transport opens the Meet URL through OpenClaw browser control and joins
as the signed-in OpenClaw browser profile. On macOS, the plugin checks for
`BlackHole 2ch` before launch. If configured, it also runs an audio bridge
health command and startup command before opening Chrome. Use `chrome` when
Chrome/audio live on the Gateway host; use `chrome-node` when Chrome/audio live
on a paired node such as a Parallels macOS VM. For local Chrome, choose the
profile with `browser.defaultProfile`; `chrome.browserProfile` is passed to
`chrome-node` hosts.

```
openclaw googlemeet join https://meet.google.com/abc-defg-hij --transport chrome
openclaw googlemeet join https://meet.google.com/abc-defg-hij --transport chrome-node
```

Route Chrome microphone and speaker audio through the local OpenClaw audio
bridge. If `BlackHole 2ch` is not installed, the join fails with a setup error
instead of silently joining without an audio path.

### [​](https://docs.openclaw.ai/plugins/google-meet\\#twilio)  Twilio

Twilio transport is a strict dial plan delegated to the Voice Call plugin. It
does not parse Meet pages for phone numbers.Use this when Chrome participation is not available or you want a phone dial-in
fallback. Google Meet must expose a phone dial-in number and PIN for the
meeting; OpenClaw does not discover those from the Meet page.Enable the Voice Call plugin on the Gateway host, not on the Chrome node:

```
{
  plugins: {
    allow: [\"google-meet\", \"voice-call\"],
    entries: {
      \"google-meet\": {
        enabled: true,
        config: {
          defaultTransport: \"chrome-node\",
          // or set \"twilio\" if Twilio should be the default
        },
      },
      \"voice-call\": {
        enabled: true,
        config: {
          provider: \"twilio\",
        },
      },
    },
  },
}
```

Provide Twilio credentials through environment or config. Environment keeps
secrets out of `openclaw.json`:

```
export TWILIO_ACCOUNT_SID=AC...
export TWILIO_AUTH_TOKEN=...
export TWILIO_FROM_NUMBER=+15550001234
```

Restart or reload the Gateway after enabling `voice-call`; plugin config changes
do not appear in an already running Gateway process until it reloads.Then verify:

```
openclaw config validate
openclaw plugins list | grep -E \'google-meet|voice-call\'
openclaw googlemeet setup
```

When Twilio delegation is wired, `googlemeet setup` includes successful
`twilio-voice-call-plugin` and `twilio-voice-call-credentials` checks.

```
openclaw googlemeet join https://meet.google.com/abc-defg-hij \\
  --transport twilio \\
  --dial-in-number +15551234567 \\
  --pin 123456
```

Use `--dtmf-sequence` when the meeting needs a custom sequence:

```
openclaw googlemeet join https://meet.google.com/abc-defg-hij \\
  --transport twilio \\
  --dial-in-number +15551234567 \\
  --dtmf-sequence ww123456#
```

## [​](https://docs.openclaw.ai/plugins/google-meet\\#oauth-and-preflight)  OAuth and preflight

OAuth is optional for creating a Meet link, but required for the API path. The
plugin uses Google’s OAuth 2.0 flow for web applications. The first time you\nuse `googlemeet create` with the API path, it will open a browser window and ask\nyou to authenticate with Google. After you grant access, it will mint a refresh\ntoken and store it in `~/.openclaw/google-meet-oauth.json`. Subsequent API\ncalls will use this refresh token to get new access tokens without further user\ninteraction.If you need to revoke access or re-authenticate, delete the file\n`~/.openclaw/google-meet-oauth.json` and try again.The plugin requests the following OAuth scopes:\n\n- `https://www.googleapis.com/auth/meetings.space.create`\n- `https://www.googleapis.com/auth/calendar.events.readonly` (for checking existing meetings)\n\n### [​](https://docs.openclaw.ai/plugins/google-meet\\#create-google-credentials)  Create Google credentials\n\n1. Go to the [Google Cloud Console](https://console.cloud.google.com/).\n2. Select an existing project or create a new one.\n3. Go to **APIs & Services > Credentials**.\n4. Click **+ Create Credentials** and choose **OAuth client ID**.\n5. Select **Web application** as the application type.\n6. Enter a name for your OAuth client (e.g., `OpenClaw Google Meet Plugin`).\n7. Add `http://localhost:8080` as an **Authorized redirect URI**.\n8. Click **Create**.\n9. Note down your **Client ID** and **Client Secret**.\n\n### [​](https://docs.openclaw.ai/plugins/google-meet\\#mint-the-refresh-token)  Mint the refresh token\n\nRun the following command, which will open a browser window for authentication:\n\n```\nopenclaw googlemeet oauth mint\n```\n\nAfter successful authentication, the refresh token will be stored in\n`~/.openclaw/google-meet-oauth.json`.\n\n### [​](https://docs.openclaw.ai/plugins/google-meet\\#verify-oauth-with-doctor)  Verify OAuth with doctor\n\n```\nopenclaw googlemeet doctor\n```\n\nThis command checks if the OAuth credentials are valid and if the refresh token\nis present and usable.\n\n## [​](https://docs.openclaw.ai/plugins/google-meet\\#config)  Config\n\n```\nplugins: {\n  entries: {\n    \"google-meet\": {\n      enabled: true,\n      config: {\n        defaultTransport: \"chrome-node\", // or \"chrome\", \"twilio\"\n        chrome: {\n          guestName: \"OpenClaw Agent\",\n          autoJoin: true,\n          reuseExistingTab: true,\n          browserProfile: \"user\", // or a named profile\n        },\n        chromeNode: {\n          node: \"parallels-macos\", // or node id, display name, remote IP\n        },\n        twilio: {\n          accountSid: \"AC...\",\n          authToken: \"...\",\n          fromNumber: \"+15550001234\",\n        },\n      },\n    },\n  },\n}\n```\n\n## [​](https://docs.openclaw.ai/plugins/google-meet\\#tool)  Tool\n\nThe `google_meet` tool is available to agents. Its schema is:\n\n```typescript\ninterface GoogleMeetTool {\n  action: \"join\" | \"create\";\n  url?: string; // Required for join action\n  transport?: \"chrome\" | \"chrome-node\" | \"twilio\";\n  mode?: \"realtime\" | \"transcribe\";\n  join?: boolean; // For create action, defaults to true\n  dialInNumber?: string; // Required for twilio transport\n  pin?: string; // Optional for twilio transport\n  dtmfSequence?: string; // Optional for twilio transport\n}\n```\n\n## [​](https://docs.openclaw.ai/plugins/google-meet\\#realtime-agent-consult)  Realtime agent consult\n\nWhen `realtime` mode is active, the agent can receive and send audio to the Meet\ncall. This allows for live consultation with the agent, where the agent can\nlisten to the conversation and interject as needed. The agent can also use other\ntools during the call to provide real-time information or actions.\n\n## [​](https://docs.openclaw.ai/plugins/google-meet\\#live-test-checklist)  Live test checklist\n\n- [ ] Ensure `blackhole-2ch` and `sox` are installed and working on the host/node.\n- [ ] Verify OAuth credentials are set up and valid if using API create.\n- [ ] Check that the `voice-call` plugin is enabled and Twilio credentials are set if using Twilio transport.\n- [ ] Confirm the Chrome profile is signed in to Google if using browser fallback create or guest join.\n- [ ] Ensure the node is approved and advertises `googlemeet.chrome` and `browser.proxy` if using `chrome-node` transport.\n- [ ] Test with `openclaw googlemeet setup` and `openclaw googlemeet doctor` to preflight the setup.\n- [ ] Try a smoke test with `openclaw googlemeet test-speech` to verify audio and join functionality.\n\n## [​](https://docs.openclaw.ai/plugins/google-meet\\#troubleshooting)  Troubleshooting\n\n### [​](https://docs.openclaw.ai/plugins/google-meet\\#agent-cannot-see-the-google-meet-tool)  Agent cannot see the Google Meet tool\n\n- Ensure the `google-meet` plugin is enabled in your `openclaw.json` config.\n- Verify that the agent has access to the `google_meet` tool in its configuration.\n\n### [​](https://docs.openclaw.ai/plugins/google-meet\\#no-connected-google-meet-capable-node)  No connected Google Meet-capable node\n\n- Start `openclaw node run` on the intended node host.\n- Approve the node from the Gateway host using `openclaw devices approve <requestId>`.\n- Make sure `openclaw plugins enable google-meet` and `openclaw plugins enable browser` were run on the node.\n- Confirm the Gateway host allows both node commands with `gateway.nodes.allowCommands: [\"googlemeet.chrome\", \"browser.proxy\"]` in its `openclaw.json` config.\n\n### [​](https://docs.openclaw.ai/plugins/google-meet\\#browser-opens-but-agent-cannot-join)  Browser opens but agent cannot join\n\n- Sign in to the Chrome profile inside the VM or on the host where Chrome is running.\n- Keep `chrome.guestName` set for guest join. Guest auto-join uses OpenClaw browser automation through the node browser proxy; make sure the node browser config points at the profile you want, for example `browser.defaultProfile: \"user\"` or a named existing-session profile.\n- Check for `manualActionRequired: true` in the join result, and address the reason (e.g., host admission, browser/OS permission prompts).\n\n### [​](https://docs.openclaw.ai/plugins/google-meet\\#meeting-creation-fails)  Meeting creation fails\n\n- If using API create, ensure OAuth credentials are correctly set up and valid.\n- If using browser fallback, ensure the Chrome profile is signed in to Google.\n- Check for any error messages in the `create` command output.\n\n### [​](https://openclaw.ai/plugins/google-meet\\#agent-joins-but-does-not-talk)  Agent joins but does not talk\n\n- Ensure `realtime` mode is selected for the join action.\n- Verify that `blackhole-2ch` and `sox` are correctly installed and configured for audio routing.\n- Check the `google_meet` status for `audioInputActive` and `audioOutputActive` to confirm audio bridge health.\n- Ensure the realtime voice provider (e.g., OpenAI, Google Gemini Live) is correctly configured and has a valid API key.\n\n### [​](https://docs.openclaw.ai/plugins/google-meet\\#twilio-setup-checks-fail)  Twilio setup checks fail\n\n- Ensure the `voice-call` plugin is enabled on the Gateway host.\n- Verify that `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, and `TWILIO_FROM_NUMBER` environment variables are correctly set.\n- Run `openclaw config validate` and `openclaw plugins list` to confirm plugin and credential status.\n\n### [​](https://docs.openclaw.ai/plugins/google-meet\\#twilio-call-starts-but-never-enters-the-meeting)  Twilio call starts but never enters the meeting\n\n- Double-check the `dialInNumber` and `pin` (or `dtmfSequence`) provided are correct for the Google Meet meeting.\n- Ensure the Google Meet meeting actually has a phone dial-in option enabled.\n- Check Twilio logs for any call-related errors.\n\n## [​](https://docs.openclaw.ai/plugins/google-meet\\#notes)  Notes\n\n- The Google Meet plugin is designed for OpenClaw agents to participate in Google Meet calls, either by joining existing calls or creating new ones.\n- It supports both Chrome-based participation (local or via a node) and Twilio dial-in for audio.\n- Realtime voice interaction with the agent is a key feature, allowing for live consultation and tool usage during calls.\n- Proper setup of audio dependencies (BlackHole, SoX) and OAuth credentials (for API-based meeting creation) is crucial for full functionality.\n- Troubleshooting steps are provided for common issues related to plugin visibility, node connectivity, browser join failures, meeting creation, audio problems, and Twilio integration.\n\n## [​](https://docs.openclaw.ai/plugins/google-meet\\#related)  Related\n\n- [Voice Call Plugin](https://docs.openclaw.ai/plugins/voice-call)\n- [Browser Plugin](https://docs.openclaw.ai/plugins/browser)\n- [OpenClaw Gateway](https://docs.openclaw.ai/gateway)\n- [OpenClaw Agents](https://docs.openclaw.ai/concepts/architecture)\n

---

## Codex Computer Use - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/codex-computer-use

[Skip to main content](https://docs.openclaw.ai/plugins/codex-computer-use#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Codex Computer Use

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [OpenClaw.app and Peekaboo](https://docs.openclaw.ai/plugins/codex-computer-use#openclaw-app-and-peekaboo)
- [iOS app](https://docs.openclaw.ai/plugins/codex-computer-use#ios-app)
- [Direct cua-driver MCP](https://docs.openclaw.ai/plugins/codex-computer-use#direct-cua-driver-mcp)
- [Quick setup](https://docs.openclaw.ai/plugins/codex-computer-use#quick-setup)
- [Commands](https://docs.openclaw.ai/plugins/codex-computer-use#commands)
- [Marketplace choices](https://docs.openclaw.ai/plugins/codex-computer-use#marketplace-choices)
- [Bundled macOS marketplace](https://docs.openclaw.ai/plugins/codex-computer-use#bundled-macos-marketplace)
- [Remote catalog limit](https://docs.openclaw.ai/plugins/codex-computer-use#remote-catalog-limit)
- [Configuration reference](https://docs.openclaw.ai/plugins/codex-computer-use#configuration-reference)
- [What OpenClaw checks](https://docs.openclaw.ai/plugins/codex-computer-use#what-openclaw-checks)
- [macOS permissions](https://docs.openclaw.ai/plugins/codex-computer-use#macos-permissions)
- [Troubleshooting](https://docs.openclaw.ai/plugins/codex-computer-use#troubleshooting)

Computer Use is a Codex-native MCP plugin for local desktop control. OpenClaw
does not vendor the desktop app, execute desktop actions itself, or bypass
Codex permissions. The bundled `codex` plugin only prepares Codex app-server:
it enables Codex plugin support, finds or installs the configured Codex
Computer Use plugin, checks that the `computer-use` MCP server is available, and
then lets Codex own the native MCP tool calls during Codex-mode turns.Use this page when OpenClaw is already using the native Codex harness. For the
runtime setup itself, see [Codex harness](https://docs.openclaw.ai/plugins/codex-harness).

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#openclaw-app-and-peekaboo)  OpenClaw.app and Peekaboo

OpenClaw.app’s Peekaboo integration is separate from Codex Computer Use. The
macOS app can host a PeekabooBridge socket so the `peekaboo` CLI can reuse the
app’s local Accessibility and Screen Recording grants for Peekaboo’s own
automation tools. That bridge does not install or proxy Codex Computer Use, and
Codex Computer Use does not call through the PeekabooBridge socket.Use [Peekaboo bridge](https://docs.openclaw.ai/platforms/mac/peekaboo) when you want OpenClaw.app to be
a permission-aware host for Peekaboo CLI automation. Use this page when a
Codex-mode OpenClaw agent should have Codex’s native `computer-use` MCP plugin
available before the turn starts.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#ios-app)  iOS app

The iOS app is separate from Codex Computer Use. It does not install or proxy
the Codex `computer-use` MCP server and it is not a desktop-control backend.
Instead, the iOS app connects as an OpenClaw node and exposes mobile
capabilities through node commands such as `canvas.*`, `camera.*`, `screen.*`,
`location.*`, and `talk.*`.Use [iOS](https://docs.openclaw.ai/platforms/ios) when you want an agent to drive an iPhone node through
the gateway. Use this page when a Codex-mode agent should control the local
macOS desktop through Codex’s native Computer Use plugin.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#direct-cua-driver-mcp)  Direct cua-driver MCP

Codex Computer Use is not the only way to expose desktop control. If you want
OpenClaw-managed runtimes to call TryCua’s driver directly, use the upstream
`cua-driver mcp` server through OpenClaw’s MCP registry instead of the
Codex-specific marketplace flow.After installing `cua-driver`, either ask it for the OpenClaw command:

```
cua-driver mcp-config --client openclaw
```

or register the stdio server yourself:

```
openclaw mcp set cua-driver \'{\"command\":\"cua-driver\",\"args\":[\"mcp\"]}\'
```

That path keeps the upstream MCP tool surface intact, including the driver
schemas and structured MCP responses. Use it when you want the CUA driver
available as a normal OpenClaw MCP server. Use the Codex Computer Use setup on
this page when Codex app-server should own plugin installation, MCP reloads,
and native tool calls inside Codex-mode turns.CUA’s driver is macOS-specific and still requires the local macOS permissions
that its app prompts for, such as Accessibility and Screen Recording. OpenClaw
does not install `cua-driver`, grant those permissions, or bypass the upstream
driver’s safety model.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#quick-setup)  Quick setup

Set `plugins.entries.codex.config.computerUse` when Codex-mode turns must have
Computer Use available before a thread starts:

```
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          computerUse: {
            autoInstall: true,
          },
        },
      },
    },
  },
  agents: {
    defaults: {
      model: \"openai/gpt-5.5\",
      agentRuntime: {
        id: \"codex\",
        fallback: \"none\",
      },
    },
  },
}
```

With this config, OpenClaw checks Codex app-server before each Codex-mode turn.
If Computer Use is missing but Codex app-server has already discovered an
installable marketplace, OpenClaw asks Codex app-server to install or re-enable
the plugin and reload MCP servers. On macOS, when no matching marketplace is
registered and the standard Codex app bundle exists, OpenClaw also tries to
register the bundled Codex marketplace from
`/Applications/Codex.app/Contents/Resources/plugins/openai-bundled` before it
fails. If setup still cannot make the MCP server available, the turn fails
before the thread starts.Existing sessions keep their runtime and Codex thread binding. After changing
`agentRuntime` or Computer Use config, use `/new` or `/reset` in the affected
chat before testing.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#commands)  Commands

Use the `/codex computer-use` commands from any chat surface where the `codex`
plugin command surface is available. These are OpenClaw chat/runtime commands,
not `openclaw codex ...` CLI subcommands:

```
/codex computer-use status
/codex computer-use install
/codex computer-use install --source <marketplace-source>
/codex computer-use install --marketplace-path <path>
/codex computer-use install --marketplace <name>
```

`status` is read-only. It does not add marketplace sources, install plugins, or
enable Codex plugin support.`install` enables Codex app-server plugin support, optionally adds a configured
marketplace source, installs or re-enables the configured plugin through Codex
app-server, reloads MCP servers, and verifies that the MCP server exposes tools.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#marketplace-choices)  Marketplace choices

OpenClaw uses the same app-server API that Codex itself exposes. The
marketplace fields choose where Codex should find `computer-use`.

| Field | Use when | Install support |
| --- | --- | --- |
| No marketplace field | You want Codex app-server to use marketplaces it already knows. | Yes, when app-server returns a local marketplace. |
| `marketplaceSource` | You have a Codex marketplace source app-server can add. | Yes, for explicit `/codex computer-use install`. |
| `marketplacePath` | You already know the local marketplace file path on the host. | Yes, for explicit install and turn-start auto-install. |
| `marketplaceName` | You want to select one already registered marketplace by name. | Yes only when the selected marketplace has a local path. |

Fresh Codex homes may need a short moment to seed their official marketplaces.
During install, OpenClaw polls `plugin/list` for up to
`marketplaceDiscoveryTimeoutMs` milliseconds. The default is 60 seconds.If multiple known marketplaces contain Computer Use, OpenClaw prefers
`openai-bundled`, then `openai-curated`, then `local`. Unknown ambiguous matches
fails closed and ask you to set `marketplaceName` or `marketplacePath`.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#bundled-macos-marketplace)  Bundled macOS marketplace

Recent Codex desktop builds bundle Computer Use here:

```
/Applications/Codex.app/Contents/Resources/plugins/openai-bundled/plugins/computer-use
```

When `computerUse.autoInstall` is true and no marketplace containing
`computer-use` is registered, OpenClaw tries to add the standard bundled
marketplace root automatically:

```
/Applications/Codex.app/Contents/Resources/plugins/openai-bundled
```

You can also register it explicitly from a shell with Codex:

```
codex plugin marketplace add /Applications/Codex.app/Contents/Resources/plugins/openai-bundled
```

If you use a nonstandard Codex app path, set `computerUse.marketplacePath` to a
local marketplace file path or run `/codex computer-use install --source <marketplace-source>` once.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#remote-catalog-limit)  Remote catalog limit

Codex app-server can list and read remote-only catalog entries, but it does not
currently support remote `plugin/install`. That means `marketplaceName` can
select a remote-only marketplace for status checks, but installs and re-enables
still need a local marketplace via `marketplaceSource` or `marketplacePath`.If status says the plugin is available in a remote Codex marketplace but remote
install is unsupported, run install with a local source or path:

```
/codex computer-use install --source <marketplace-source>
/codex computer-use install --marketplace-path <path>
```

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#configuration-reference)  Configuration reference

| Field | Default | Meaning |
| --- | --- | --- |
| `enabled` | inferred | Require Computer Use. Defaults to true when another Computer Use field is set. |
| `autoInstall` | false | Install or re-enable from already discovered marketplaces at turn start. |
| `marketplaceDiscoveryTimeoutMs` | 60000 | How long install waits for Codex app-server marketplace discovery. |
| `marketplaceSource` | unset | Source string passed to Codex app-server `marketplace/add`. |
| `marketplacePath` | unset | Local Codex marketplace file path containing the plugin. |
| `marketplaceName` | unset | Registered Codex marketplace name to select. |
| `pluginName` | `computer-use` | Codex marketplace plugin name. |
| `mcpServerName` | `computer-use` | MCP server name exposed by the installed plugin. |

Turn-start auto-install intentionally refuses configured `marketplaceSource`
values. Adding a new source is an explicit setup operation, so use
`/codex computer-use install --source <marketplace-source>` once, then let
`autoInstall` handle future re-enables from discovered local marketplaces.
Turn-start auto-install can use a configured `marketplacePath`, because that is
already a local path on the host.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#what-openclaw-checks)  What OpenClaw checks

OpenClaw reports a stable setup reason internally and formats the user-facing
status for chat:

| Reason | Meaning | Next step |
| --- | --- | --- |
| `disabled` | `computerUse.enabled` resolved to false. | Set `enabled` or another Computer Use field. |
| `marketplace_missing` | No matching marketplace was available. | Configure source, path, or marketplace name. |
| `plugin_not_installed` | Marketplace exists, but the plugin is not installed. | Run install or enable `autoInstall`. |
| `plugin_disabled` | Plugin is installed but disabled in Codex config. | Run install to re-enable it. |
| `remote_install_unsupported` | Selected marketplace is remote-only. | Use `marketplaceSource` or `marketplacePath`. |
| `mcp_missing` | Plugin is enabled, but the MCP server is unavailable. | Check Codex Computer Use and OS permissions. |
| `ready` | Plugin and MCP tools are available. | Start the Codex-mode turn. |
| `check_failed` | A Codex app-server request failed during status check. | Check app-server connectivity and logs. |
| `auto_install_blocked` | Turn-start setup would need to add a new source. | Run explicit install first. |

The chat output includes the plugin state, MCP server state, marketplace, tools
when available, and the specific message for the failing setup step.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#macos-permissions)  macOS permissions

Computer Use is macOS-specific. The Codex-owned MCP server may need local OS
permissions before it can inspect or control apps. If OpenClaw says Computer Use
is installed but the MCP server is unavailable, verify the Codex-side Computer
Use setup first:

- Codex app-server is running on the same host where desktop control should
happen.
- The Computer Use plugin is enabled in Codex config.
- The `computer-use` MCP server appears in Codex app-server MCP status.
- macOS has granted the required permissions for the desktop-control app.
- The current host session can access the desktop being controlled.

OpenClaw intentionally fails closed when `computerUse.enabled` is true. A
Codex-mode turn should not silently proceed without the native desktop tools
that the config required.

## [​](https://docs.openclaw.ai/plugins/codex-computer-use\\#troubleshooting)  Troubleshooting

**Status says not installed.** Run `/codex computer-use install`. If the
marketplace is not discovered, pass `--source` or `--marketplace-path`.**Status says installed but disabled.** Run `/codex computer-use install` again.
Codex app-server install writes the plugin config back to enabled.**Status says remote install is unsupported.** Use a local marketplace source or
path. Remote-only catalog entries can be inspected but not installed through the
current app-server API.**Status says the MCP server is unavailable.** Re-run install once so MCP
servers reload. If it remains unavailable, fix the Codex Computer Use app,
Codex app-server MCP status, or macOS permissions.**Status or a probe times out on `computer-use.list_apps`.** The plugin and MCP
server are present, but the local Computer Use bridge did not answer. Quit or
restart Codex Computer Use, relaunch Codex Desktop if needed, then retry in a
fresh OpenClaw session.**A Computer Use tool says `Native hook relay unavailable`.** The Codex-native
tool hook reached OpenClaw with a stale or missing relay registration. Start a
fresh OpenClaw session with `/new` or `/reset`. If it keeps happening, restart
the gateway so old app-server threads and hook registrations are dropped, then
retry.**Turn-start auto-install refuses a source.** This is intentional. Add the
source with explicit `/codex computer-use install --source <marketplace-source>`
first, then future turn-start auto-install can use the discovered local
marketplace.

[Codex harness](https://docs.openclaw.ai/plugins/codex-harness) [Google Meet plugin](https://docs.openclaw.ai/plugins/google-meet)

Ctrl+I

---

## Plugin runtime helpers - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/sdk-runtime

[Skip to main content](https://docs.openclaw.ai/plugins/sdk-runtime#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nSDK reference\n\nPlugin runtime helpers\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Runtime namespaces](https://docs.openclaw.ai/plugins/sdk-runtime#runtime-namespaces)\n- [Storing runtime references](https://docs.openclaw.ai/plugins/sdk-runtime#storing-runtime-references)\n- [Other top-level api fields](https://docs.openclaw.ai/plugins/sdk-runtime#other-top-level-api-fields)\n- [Related](https://docs.openclaw.ai/plugins/sdk-runtime#related)\n\nReference for the `api.runtime` object injected into every plugin during registration. Use these helpers instead of importing host internals directly.\n\n[**Channel plugins** \\\\\n\\\\\nStep-by-step guide that uses these helpers in context for channel plugins.](https://docs.openclaw.ai/plugins/sdk-channel-plugins)\n\n[**Provider plugins** \\\\\n\\\\\nStep-by-step guide that uses these helpers in context for provider plugins.](https://docs.openclaw.ai/plugins/sdk-provider-plugins)\n\n```\nregister(api) {\n  const runtime = api.runtime;\n}\n```\n\n## [​](https://docs.openclaw.ai/plugins/sdk-runtime\\#runtime-namespaces)  Runtime namespaces\n\napi.runtime.agent\n\nAgent identity, directories, and session management.\n\n```\n// Resolve the agent\'s working directory\nconst agentDir = api.runtime.agent.resolveAgentDir(cfg);\n\n// Resolve agent workspace\nconst workspaceDir = api.runtime.agent.resolveAgentWorkspaceDir(cfg);\n\n// Get agent identity\nconst identity = api.runtime.agent.resolveAgentIdentity(cfg);\n\n// Get default thinking level\nconst thinking = api.runtime.agent.resolveThinkingDefault(cfg, provider, model);\n\n// Get agent timeout\nconst timeoutMs = api.runtime.agent.resolveAgentTimeoutMs(cfg);\n\n// Ensure workspace exists\nawait api.runtime.agent.ensureAgentWorkspace(cfg);\n\n// Run an embedded agent turn\nconst agentDir = api.runtime.agent.resolveAgentDir(cfg);\nconst result = await api.runtime.agent.runEmbeddedAgent({\n  sessionId: \"my-plugin:task-1\",\n  runId: crypto.randomUUID(),\n  sessionFile: path.join(agentDir, \"sessions\", \"my-plugin-task-1.jsonl\"),\n  workspaceDir: api.runtime.agent.resolveAgentWorkspaceDir(cfg),\n  prompt: \"Summarize the latest changes\",\n  timeoutMs: api.runtime.agent.resolveAgentTimeoutMs(cfg),\n});\n```\n\n`runEmbeddedAgent(...)` is the neutral helper for starting a normal OpenClaw agent turn from plugin code. It uses the same provider/model resolution and agent-harness selection as channel-triggered replies.`runEmbeddedPiAgent(...)` remains as a compatibility alias.**Session store helpers** are under `api.runtime.agent.session`:\n\n```\nconst storePath = api.runtime.agent.session.resolveStorePath(cfg);\nconst store = api.runtime.agent.session.loadSessionStore(cfg);\nawait api.runtime.agent.session.saveSessionStore(cfg, store);\nconst filePath = api.runtime.agent.session.resolveSessionFilePath(cfg, sessionId);\n```\n\napi.runtime.agent.defaults\n\nDefault model and provider constants:\n\n```\nconst model = api.runtime.agent.defaults.model; // e.g. \"anthropic/claude-sonnet-4-6\"\nconst provider = api.runtime.agent.defaults.provider; // e.g. \"anthropic\"\n```\n\napi.runtime.subagent\n\nLaunch and manage background subagent runs.\n\n```\n// Start a subagent run\nconst { runId } = await api.runtime.subagent.run({\n  sessionKey: \"agent:main:subagent:search-helper\",\n  message: \"Expand this query into focused follow-up searches.\",\n  provider: \"openai\", // optional override\n  model: \"gpt-4.1-mini\", // optional override\n  deliver: false,\n});\n\n// Wait for completion\nconst result = await api.runtime.subagent.waitForRun({ runId, timeoutMs: 30000 });\n\n// Read session messages\nconst { messages } = await api.runtime.subagent.getSessionMessages({\n  sessionKey: \"agent:main:subagent:search-helper\",\n  limit: 10,\n});\n\n// Delete a session\nawait api.runtime.subagent.deleteSession({\n  sessionKey: \"agent:main:subagent:search-helper\",\n});\n```\n\nModel overrides (`provider`/`model`) require operator opt-in via `plugins.entries.<id>.subagent.allowModelOverride: true` in config. Untrusted plugins can still run subagents, but override requests are rejected.\n\napi.runtime.nodes\n\nList connected nodes and invoke a node-host command from Gateway-loaded plugin code or from plugin CLI commands. Use this when a plugin owns local work on a paired device, for example a browser or audio bridge on another Mac.\n\n```\nconst { nodes } = await api.runtime.nodes.list({ connected: true });\n\nconst result = await api.runtime.nodes.invoke({\n  nodeId: \"mac-studio\",\n  command: \"my-plugin.command\",\n  params: { action: \"start\" },\n  timeoutMs: 30000,\n});\n```\n\nInside the Gateway this runtime is in-process. In plugin CLI commands it calls the configured Gateway over RPC, so commands such as `openclaw googlemeet recover-tab` can inspect paired nodes from the terminal. Node commands still go through normal Gateway node pairing, command allowlists, and node-local command handling.\n\napi.runtime.taskFlow\n\nBind a Task Flow runtime to an existing OpenClaw session key or trusted tool context, then create and manage Task Flows without passing an owner on every call.\n\n```\nconst taskFlow = api.runtime.taskFlow.fromToolContext(ctx);\n\nconst created = taskFlow.createManaged({\n  controllerId: \"my-plugin/review-batch\",\n  goal: \"Review new pull requests\",\n});\n\nconst child = taskFlow.runTask({\n  flowId: created.flowId,\n  runtime: \"acp\",\n  childSessionKey: \"agent:main:subagent:reviewer\",\n  task: \"Review PR #123\",\n  status: \"running\",\n  startedAt: Date.now(),\n});\n\nconst waiting = taskFlow.setWaiting({\n  flowId: created.flowId,\n  expectedRevision: created.revision,\n  currentStep: \"await-human-reply\",\n  waitJson: { kind: \"reply\", channel: \"telegram\" },\n});\n```\n\nUse `bindSession({ sessionKey, requesterOrigin })` when you already have a trusted OpenClaw session key from your own binding layer. Do not bind from raw user input.\n\napi.runtime.tts\n\nText-to-speech synthesis.\n\n```\n// Standard TTS\nconst clip = await api.runtime.tts.textToSpeech({\n  text: \"Hello from OpenClaw\",\n  cfg: api.config,\n});\n\n// Telephony-optimized TTS\nconst telephonyClip = await api.runtime.tts.textToSpeechTelephony({\n  text: \"Hello from OpenClaw\",\n  cfg: api.config,\n});\n\n// List available voices\nconst voices = await api.runtime.tts.listVoices({\n  provider: \"elevenlabs\",\n  cfg: api.config,\n});\n```\n\nUses core `messages.tts` configuration and provider selection. Returns PCM audio buffer + sample rate.\n\napi.runtime.mediaUnderstanding\n\nImage, audio, and video analysis.\n\n```\n// Describe an image\nconst image = await api.runtime.mediaUnderstanding.describeImageFile({\n  filePath: \"/tmp/inbound-photo.jpg\",\n  cfg: api.config,\n  agentDir: \"/tmp/agent\",\n});\n\n// Transcribe audio\nconst { text } = await api.runtime.mediaUnderstanding.transcribeAudioFile({\n  filePath: \"/tmp/inbound-audio.ogg\",\n  cfg: api.config,\n  mime: \"audio/ogg\", // optional, for when MIME cannot be inferred\n});\n\n// Describe a video\nconst video = await api.runtime.mediaUnderstanding.describeVideoFile({\n  filePath: \"/tmp/inbound-video.mp4\",\n  cfg: api.config,\n});\n\n// Generic file analysis\nconst result = await api.runtime.mediaUnderstanding.runFile({\n  filePath: \"/tmp/inbound-file.pdf\",\n  cfg: api.config,\n});\n```\n\nReturns `{ text: undefined }` when no output is produced (e.g. skipped input).\n\n`api.runtime.stt.transcribeAudioFile(...)` remains as a compatibility alias for `api.runtime.mediaUnderstanding.transcribeAudioFile(...)`.\n\napi.runtime.imageGeneration\n\nImage generation.\n\n```\nconst result = await api.runtime.imageGeneration.generate({\n  prompt: \"A robot painting a sunset\",\n  cfg: api.config,\n});\n\nconst providers = api.runtime.imageGeneration.listProviders({ cfg: api.config });\n```\n\napi.runtime.webSearch\n\nWeb search.\n\n```\nconst providers = api.runtime.webSearch.listProviders({ config: api.config });\n\nconst result = await api.runtime.webSearch.search({\n  config: api.config,\n  args: { query: \"OpenClaw plugin SDK\", count: 5 },\n});\n```\n\napi.runtime.media\n\nLow-level media utilities.\n\n```\nconst webMedia = await api.runtime.media.loadWebMedia(url);\nconst mime = await api.runtime.media.detectMime(buffer);\nconst kind = api.runtime.media.mediaKindFromMime(\"image/jpeg\"); // \"image\"\nconst isVoice = api.runtime.media.isVoiceCompatibleAudio(filePath);\nconst metadata = await api.runtime.media.getImageMetadata(filePath);\nconst resized = await api.runtime.media.resizeToJpeg(buffer, { maxWidth: 800 });\nconst terminalQr = await api.runtime.media.renderQrTerminal(\"https://openclaw.ai\");\nconst pngQr = await api.runtime.media.renderQrPngBase64(\"https://openclaw.ai\", {\n  scale: 6, // 1-12\n  marginModules: 4, // 0-16\n});\nconst pngQrDataUrl = await api.runtime.media.renderQrPngDataUrl(\"https://openclaw.ai\");\nconst tmpRoot = resolvePreferredOpenClawTmpDir();\nconst pngQrFile = await api.runtime.media.writeQrPngTempFile(\"https://openclaw.ai\", {\n  tmpRoot,\n  dirPrefix: \"my-plugin-qr-\",\n  fileName: \"qr.png\",\n});\n```\n\napi.runtime.config\n\nConfig load and write.\n\n```\nconst cfg = await api.runtime.config.loadConfig();\nawait api.runtime.config.writeConfigFile(cfg);\n```\n\napi.runtime.system\n\nSystem-level utilities.\n\n```\nawait api.runtime.system.enqueueSystemEvent(event);\napi.runtime.system.requestHeartbeatNow();\nconst output = await api.runtime.system.runCommandWithTimeout(cmd, args, opts);\nconst hint = api.runtime.system.formatNativeDependencyHint(pkg);\n```\n\napi.runtime.events\n\nEvent subscriptions.\n\n```\napi.runtime.events.onAgentEvent((event) => {\n  /* ... */\n});\napi.runtime.events.onSessionTranscriptUpdate((update) => {\n  /* ... */\n});\n```\n\napi.runtime.logging\n\nLogging.\n\n```\nconst verbose = api.runtime.logging.shouldLogVerbose();\nconst childLogger = api.runtime.logging.getChildLogger({ plugin: \"my-plugin\" }, { level: \"debug\" });\n```\n\napi.runtime.modelAuth\n\nModel and provider auth resolution.\n\n```\nconst auth = await api.runtime.modelAuth.getApiKeyForModel({ model, cfg });\nconst providerAuth = await api.runtime.modelAuth.resolveApiKeyForProvider({\n  provider: \"openai\",\n  cfg,\n});\n```\n\napi.runtime.state\n\nState directory resolution.\n\n```\nconst stateDir = api.runtime.state.resolveStateDir();\n```\n\napi.runtime.tools\n\nMemory tool factories and CLI.\n\n```\nconst getTool = api.runtime.tools.createMemoryGetTool(/* ... */);\nconst searchTool = api.runtime.tools.createMemorySearchTool(/* ... */);\napi.runtime.tools.registerMemoryCli(/* ... */);\n```\n\napi.runtime.channel\n\nChannel-specific runtime helpers (available when a channel plugin is loaded).`api.runtime.channel.mentions` is the shared inbound mention-policy surface for bundled channel plugins that use runtime injection:\n\n```\nconst mentionMatch = api.runtime.channel.mentions.matchesMentionWithExplicit(text, {\n  mentionRegexes,\n  mentionPatterns,\n});\n\nconst decision = api.runtime.channel.mentions.resolveInboundMentionDecision({\n  facts: {\n    canDetectMention: true,\n    wasMentioned: mentionMatch.matched,\n    implicitMentionKinds: api.runtime.channel.mentions.implicitMentionKindWhen(\n      \"reply_to_bot\",\n      isReplyToBot,\n    ),\n  },\n  policy: {\n    isGroup,\n    requireMention,\n    allowTextCommands,\n    hasControlCommand,\n    commandAuthorized,\n  },\n});\n```\n\nAvailable mention helpers:\n\n- `buildMentionRegexes`\n- `matchesMentionPatterns`\n- `matchesMentionWithExplicit`\n- `implicitMentionKindWhen`\n- `resolveInboundMentionDecision`\n\n`api.runtime.channel.mentions` intentionally does not expose the older `resolveMentionGating*` compatibility helpers. Prefer the normalized `{ facts, policy }` path.\n\n## [​](https://docs.openclaw.ai/plugins/sdk-runtime\\#storing-runtime-references)  Storing runtime references\n\nUse `createPluginRuntimeStore` to store the runtime reference for use outside the `register` callback:\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/plugins/sdk-runtime#)\n\nCreate the store\n\n```\nimport { createPluginRuntimeStore } from \"openclaw/plugin-sdk/runtime-store\";\nimport type { PluginRuntime } from \"openclaw/plugin-sdk/runtime-store\";\n\nconst store = createPluginRuntimeStore<PluginRuntime>({\n  pluginId: \"my-plugin\",\n  errorMessage: \"my-plugin runtime not initialized\",\n});\n```\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/plugins/sdk-runtime#)\n\nWire into the entry point\n\n```\nexport default defineChannelPluginEntry({\n  id: \"my-plugin\",\n  name: \"My Plugin\",\n  description: \"Example\",\n  plugin: myPlugin,\n  setRuntime: store.setRuntime,\n});\n```\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/plugins/sdk-runtime#)\n\nAccess from other files\n\n```\nexport function getRuntime() {\n  return store.getRuntime(); // throws if not initialized\n}\n\nexport function tryGetRuntime() {\n  return store.tryGetRuntime(); // returns null if not initialized\n}\n```\n\nPrefer `pluginId` for the runtime-store identity. The lower-level `key` form is for uncommon cases where one plugin intentionally needs more than one runtime slot.\n\n## [​](https://docs.openclaw.ai/plugins/sdk-runtime\\#other-top-level-api-fields)  Other top-level `api` fields\n\nBeyond `api.runtime`, the API object also provides:\n\n[​](https://docs.openclaw.ai/plugins/sdk-runtime#param-api-id)\n\napi.id\n\nstring\n\nPlugin id.\n\n[​](https://docs.openclaw.ai/plugins/sdk-runtime#param-api-name)\n\napi.name\n\nstring\n\nPlugin display name.\n\n[​](https://docs.openclaw.ai/plugins/sdk-runtime#param-api-config)\n\napi.config\n\nOpenClawConfig\n\nCurrent config snapshot (active in-memory runtime snapshot when available).\n\n[​](https://docs.openclaw.ai/plugins/sdk-runtime#param-api-plugin-config)\n\napi.pluginConfig\n\nRecord<string, unknown>\n\nPlugin-specific config from `plugins.entries.<id>.config`.\n\n[​](https://docs.openclaw.ai/plugins/sdk-runtime#param-api-logger)\n\napi.logger\n\nPluginLogger\n\nScoped logger (`debug`, `info`, `warn`, `error`).\n\n[​](https://docs.openclaw.ai/plugins/sdk-runtime#param-api-registration-mode)\n\napi.registrationMode\n\nPluginRegistrationMode\n\nCurrent load mode; `\"setup-runtime\"` is the lightweight pre-full-entry startup/setup window.\n\n[​](https://docs.openclaw.ai/plugins/sdk-runtime#param-api-resolve-path-input)\n\napi.resolvePath(input)\n\n(string) => string\n\nResolve a path relative to the plugin root.\n\n## [​](https://docs.openclaw.ai/plugins/sdk-runtime\\#related)  Related\n\n- [Plugin internals](https://docs.openclaw.ai/plugins/architecture) — capability model and registry\n- [SDK entry points](https://docs.openclaw.ai/plugins/sdk-entrypoints) — `definePluginEntry` options\n- [SDK overview](https://docs.openclaw.ai/plugins/sdk-overview) — subpath reference\n\n[Entry Points](https://docs.openclaw.ai/plugins/sdk-entrypoints) [Agent Harness](https://docs.openclaw.ai/plugins/sdk-agent-harness)\n\nCtrl+I

---

## Skill workshop plugin
**Source:** https://docs.openclaw.ai/plugins/skill-workshop

[Skip to main content](https://docs.openclaw.ai/plugins/skill-workshop#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Skill workshop plugin

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Default state](https://docs.openclaw.ai/plugins/skill-workshop#default-state)
- [Enable](https://docs.openclaw.ai/plugins/skill-workshop#enable)
- [Configuration](https://docs.openclaw.ai/plugins/skill-workshop#configuration)
- [Capture paths](https://docs.openclaw.ai/plugins/skill-workshop#capture-paths)
- [Tool suggestions](https://docs.openclaw.ai/plugins/skill-workshop#tool-suggestions)
- [Heuristic capture](https://docs.openclaw.ai/plugins/skill-workshop#heuristic-capture)
- [LLM reviewer](https://docs.openclaw.ai/plugins/skill-workshop#llm-reviewer)
- [Proposal lifecycle](https://docs.openclaw.ai/plugins/skill-workshop#proposal-lifecycle)
- [Tool reference](https://docs.openclaw.ai/plugins/skill-workshop#tool-reference)
- [status](https://docs.openclaw.ai/plugins/skill-workshop#status)
- [list\\_pending](https://docs.openclaw.ai/plugins/skill-workshop#list_pending)
- [list\\_quarantine](https://docs.openclaw.ai/plugins/skill-workshop#list_quarantine)
- [inspect](https://docs.openclaw.ai/plugins/skill-workshop#inspect)
- [suggest](https://docs.openclaw.ai/plugins/skill-workshop#suggest)
- [apply](https://docs.openclaw.ai/plugins/skill-workshop#apply)
- [reject](https://docs.openclaw.ai/plugins/skill-workshop#reject)
- [write\\_support\\_file](https://docs.openclaw.ai/plugins/skill-workshop#write_support_file)
- [Skill writes](https://docs.openclaw.ai/plugins/skill-workshop#skill-writes)
- [Safety model](https://docs.openclaw.ai/plugins/skill-workshop#safety-model)
- [Prompt guidance](https://docs.openclaw.ai/plugins/skill-workshop#prompt-guidance)
- [Costs and runtime behavior](https://docs.openclaw.ai/plugins/skill-workshop#costs-and-runtime-behavior)
- [Operating patterns](https://docs.openclaw.ai/plugins/skill-workshop#operating-patterns)
- [Debugging](https://docs.openclaw.ai/plugins/skill-workshop#debugging)
- [QA scenarios](https://docs.openclaw.ai/plugins/skill-workshop#qa-scenarios)
- [When not to enable auto apply](https://docs.openclaw.ai/plugins/skill-workshop#when-not-to-enable-auto-apply)
- [Related docs](https://docs.openclaw.ai/plugins/skill-workshop#related-docs)

Skill Workshop is **experimental**. It is disabled by default, its capture\nheuristics and reviewer prompts may change between releases, and automatic\nwrites should be used only in trusted workspaces after reviewing pending-mode\noutput first.Skill Workshop is procedural memory for workspace skills. It lets an agent turn\nreusable workflows, user corrections, hard-won fixes, and recurring pitfalls\ninto `SKILL.md` files under:\n\n```\n<workspace>/skills/<skill-name>/SKILL.md\n```\n
This is different from long-term memory:\n\n- **Memory** stores facts, preferences, entities, and past context.\n- **Skills** store reusable procedures the agent should follow on future tasks.\n- **Skill Workshop** is the bridge from a useful turn to a durable workspace\nskill, with safety checks and optional approval.\n\nSkill Workshop is useful when the agent learns a procedure such as:\n\n- how to validate externally sourced animated GIF assets\n- how to replace screenshot assets and verify dimensions\n- how to run a repo-specific QA scenario\n- how to debug a recurring provider failure\n- how to repair a stale local workflow note\n\nIt is not intended for:\n\n- facts like “the user likes blue”\n- broad autobiographical memory\n- raw transcript archiving\n- secrets, credentials, or hidden prompt text\n- one-off instructions that will not repeat\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#default-state)  Default state\n\nThe bundled plugin is **experimental** and **disabled by default** unless it is\nexplicitly enabled in `plugins.entries.skill-workshop`.The plugin manifest does not set `enabledByDefault: true`. The `enabled: true`\ndefault inside the plugin config schema applies only after the plugin entry has\nalready been selected and loaded.Experimental means:\n\n- the plugin is supported enough for opt-in testing and dogfooding\n- proposal storage, reviewer thresholds, and capture heuristics can evolve\n- pending approval is the recommended starting mode\n- auto apply is for trusted personal/workspace setups, not shared or hostile\ninput-heavy environments\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#enable)  Enable\n\nMinimal safe config:\n\n```\n{\n  plugins: {\n    entries: {\n      \"skill-workshop\": {\n        enabled: true,\n        config: {\n          autoCapture: true,\n          approvalPolicy: \"pending\",\n          reviewMode: \"hybrid\",\n        },\n      },\n    },\n  },\n}\n```\n\nWith this config:\n\n- the `skill_workshop` tool is available\n- explicit reusable corrections are queued as pending proposals\n- threshold-based reviewer passes can propose skill updates\n- no skill file is written until a pending proposal is applied\n\nUse automatic writes only in trusted workspaces:\n\n```\n{\n  plugins: {\n    entries: {\n      \"skill-workshop\": {\n        enabled: true,\n        config: {\n          autoCapture: true,\n          approvalPolicy: \"auto\",\n          reviewMode: \"hybrid\",\n        },\n      },\n    },\n  },\n}\n```\n\n`approvalPolicy: \"auto\"` still uses the same scanner and quarantine path. It\ndoes not apply proposals with critical findings.\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#configuration)  Configuration\n\n| Key | Default | Range / values | Meaning |\n| --- | --- | --- | --- |\n| `enabled` | `true` | boolean | Enables the plugin after the plugin entry is loaded. |\n| `autoCapture` | `true` | boolean | Enables post-turn capture/review on successful agent turns. |\n| `approvalPolicy` | `\"pending\"` | `\"pending\"`, `\"auto\"` | Queue proposals or write safe proposals automatically. |\n| `reviewMode` | `\"hybrid\"` | `\"off\"`, `\"heuristic\"`, `\"llm\"`, `\"hybrid\"` | Chooses explicit correction capture, LLM reviewer, both, or neither. |\n| `reviewInterval` | `15` | `1..200` | Run reviewer after this many successful turns. |\n| `reviewMinToolCalls` | `8` | `1..500` | Run reviewer after this many observed tool calls. |\n| `reviewTimeoutMs` | `45000` | `5000..180000` | Timeout for the embedded reviewer run. |\n| `maxPending` | `50` | `1..200` | Max pending/quarantined proposals kept per workspace. |\n| `maxSkillBytes` | `40000` | `1024..200000` | Max generated skill/support file size. |\n\nRecommended profiles:\n\n```\n// Conservative: explicit tool use only, no automatic capture.\n{\n  autoCapture: false,\n  approvalPolicy: \"pending\",\n  reviewMode: \"off\",\n}\n```\n\n```\n// Review-first: capture automatically, but require approval.\n{\n  autoCapture: true,\n  approvalPolicy: \"pending\",\n  reviewMode: \"hybrid\",\n}\n```\n\n```\n// Trusted automation: write safe proposals immediately.\n{\n  autoCapture: true,\n  approvalPolicy: \"auto\",\n  reviewMode: \"hybrid\",\n}\n```\n\n```\n// Low-cost: no reviewer LLM call, only explicit correction phrases.\n{\n  autoCapture: true,\n  approvalPolicy: \"pending\",\n  reviewMode: \"heuristic\",\n}\n```\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#capture-paths)  Capture paths\n\nSkill Workshop has three capture paths.\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#tool-suggestions)  Tool suggestions\n\nThe model can call `skill_workshop` directly when it sees a reusable procedure\nor when the user asks it to save/update a skill.This is the most explicit path and works even with `autoCapture: false`.\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#heuristic-capture)  Heuristic capture\n\nWhen `autoCapture` is enabled and `reviewMode` is `heuristic` or `hybrid`, the\nplugin scans successful turns for explicit user correction phrases:\n\n- `next time`\n- `from now on`\n- `remember to`\n- `make sure to`\n- `always ... use/check/verify/record/save/prefer`\n- `prefer ... when/for/instead/use`\n- `when asked`\n\nThe heuristic creates a proposal from the latest matching user instruction. It\nuses topic hints to choose skill names for common workflows:\n\n- animated GIF tasks -> `animated-gif-workflow`\n- screenshot or asset tasks -> `screenshot-asset-workflow`\n- QA or scenario tasks -> `qa-scenario-workflow`\n- GitHub PR tasks -> `github-pr-workflow`\n- fallback -> `learned-workflows`\n\nHeuristic capture is intentionally narrow. It is for clear corrections and\nrepeatable process notes, not for general transcript summarization.\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#llm-reviewer)  LLM reviewer\n\nWhen `autoCapture` is enabled and `reviewMode` is `llm` or `hybrid`, the plugin\nruns a compact embedded reviewer after thresholds are reached.The reviewer receives:\n\n- the recent transcript text, capped to the last 12,000 characters\n- up to 12 existing workspace skills\n- up to 2,000 characters from each existing skill\n- JSON-only instructions\n\nThe reviewer has no tools:\n\n- `disableTools: true`\n- `toolsAllow: []`\n- `disableMessageTool: true`\n\nThe reviewer returns either `{ \"action\": \"none\" }` or one proposal. The `action` field is `create`, `append`, or `replace` — prefer `append`/`replace` when a relevant skill already exists; use `create` only when no existing skill fits.Example `create`:\n\n```\n{\n  \"action\": \"create\",\n  \"skillName\": \"media-asset-qa\",\n  \"title\": \"Media Asset QA\",\n  \"reason\": \"Reusable animated media acceptance workflow\",\n  \"description\": \"Validate externally sourced animated media before product use.\",\n  \"body\": \"## Workflow\\n\\n- Verify true animation.\\n- Record attribution.\\n- Store a local approved copy.\\n- Verify in product UI before final reply.\"\n}\n```\n\n`append` adds `section` \\+ `body`. `replace` swaps `oldText` for `newText` in the named skill.\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#proposal-lifecycle)  Proposal lifecycle\n\nEvery generated update becomes a proposal with:\n\n- `id`\n- `createdAt`\n- `updatedAt`\n- `workspaceDir`\n- optional `agentId`\n- optional `sessionId`\n- `skillName`\n- `title`\n- `reason`\n- `source`: `tool`, `agent_end`, or `reviewer`\n- `status`\n- `change`\n- optional `scanFindings`\n- optional `quarantineReason`\n\nProposal statuses:\n\n- `pending` \\- waiting for approval\n- `applied` \\- written to `<workspace>/skills`\n- `rejected` \\- rejected by operator/model\n- `quarantined` \\- blocked by critical scanner findings\n\nState is stored per workspace under the Gateway state directory:\n\n```\n<stateDir>/skill-workshop/<workspace-hash>.json\n```\n\nPending and quarantined proposals are deduplicated by skill name and change\npayload. The store keeps the newest pending/quarantined proposals up to\n`maxPending`.\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#tool-reference)  Tool reference\n\nThe plugin registers one agent tool:\n\n```\nskill_workshop\n```\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#status)  `status`\n\nCount proposals by state for the active workspace.\n\n```\n{ \"action\": \"status\" }\n```\n\nResult shape:\n\n```\n{\n  \"workspaceDir\": \"/path/to/workspace\",\n  \"pending\": 1,\n  \"quarantined\": 0,\n  \"applied\": 3,\n  \"rejected\": 0\n}\n```\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#list_pending)  `list_pending`\n\nList pending proposals.\n\n```\n{ \"action\": \"list_pending\" }\n```\n\nTo list another status:\n\n```\n{ \"action\": \"list_pending\", \"status\": \"applied\" }\n```\n\nValid `status` values:\n\n- `pending`\n- `applied`\n- `rejected`\n- `quarantined`\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#list_quarantine)  `list_quarantine`\n\nList quarantined proposals.\n\n```\n{ \"action\": \"list_quarantine\" }\n```\n\nUse this when automatic capture appears to do nothing and the logs mention\n`skill-workshop: quarantined <skill>`.\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#inspect)  `inspect`\n\nFetch a proposal by id.\n\n```\n{\n  \"action\": \"inspect\",\n  \"id\": \"proposal-id\"\n}\n```\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#suggest)  `suggest`\n\nCreate a proposal. With `approvalPolicy: \"pending\"` (default), this queues instead of writing.\n\n```\n{\n  \"action\": \"suggest\",\n  \"skillName\": \"animated-gif-workflow\",\n  \"title\": \"Animated GIF Workflow\",\n  \"reason\": \"User established reusable GIF validation rules.\",\n  \"description\": \"Validate animated GIF assets before using them.\",\n  \"body\": \"## Workflow\\n\\n- Verify the URL resolves to image/gif.\\n- Confirm it has multiple frames.\\n- Record attribution and license.\\n- Avoid hotlinking when a local asset is needed.\"\n}\n```\n\nForce a safe write (apply: true)\n\n```\n{\n  \"action\": \"suggest\",\n  \"apply\": true,\n  \"skillName\": \"animated-gif-workflow\",\n  \"description\": \"Validate animated GIF assets before using them.\",\n  \"body\": \"## Workflow\\n\\n- Verify true animation.\\n- Record attribution.\"\n}\n```\n\nForce pending under auto policy (apply: false)\n\n```\n{\n  \"action\": \"suggest\",\n  \"apply\": false,\n  \"skillName\": \"screenshot-asset-workflow\",\n  \"description\": \"Screenshot replacement workflow.\",\n  \"body\": \"## Workflow\\n\\n- Verify dimensions.\\n- Optimize the PNG.\\n- Run the relevant gate.\"\n}\n```\n\nAppend to a named section\n\n```\n{\n  \"action\": \"suggest\",\n  \"skillName\": \"qa-scenario-workflow\",\n  \"section\": \"Workflow\",\n  \"description\": \"QA scenario workflow.\",\n  \"body\": \"- For media QA, verify generated assets render and pass final assertions.\"\n}\n```\n\nReplace exact text\n\n```\n{\n  \"action\": \"suggest\",\n  \"skillName\": \"github-pr-workflow\",\n  \"oldText\": \"- Check the PR.\",\n  \"newText\": \"- Check unresolved review threads, CI status, linked issues, and changed files before deciding.\"\n}\n```\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#apply)  `apply`\n\nApply a pending proposal.\n\n```\n{\n  \"action\": \"apply\",\n  \"id\": \"proposal-id\"\n}\n```\n\n`apply` refuses quarantined proposals:\n\n```\nquarantined proposal cannot be applied\n```\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#reject)  `reject`\n\nMark a proposal rejected.\n\n```\n{\n  \"action\": \"reject\",\n  \"id\": \"proposal-id\"\n}\n```\n\n### [​](https://docs.openclaw.ai/plugins/skill-workshop\\#write_support_file)  `write_support_file`\n\nWrite a supporting file inside an existing or proposed skill directory.Allowed top-level support directories:\n\n- `references/`\n- `templates/`\n- `scripts/`\n- `assets/`\n\nExample:\n\n```\n{\n  \"action\": \"write_support_file\",\n  \"skillName\": \"release-workflow\",\n  \"relativePath\": \"references/checklist.md\",\n  \"body\": \"# Release Checklist\\n\\n- Run release docs.\\n- Verify changelog.\\n\"\n}\n```\n\nSupport files are workspace-scoped, path-checked, byte-limited by\n`maxSkillBytes`, scanned, and written atomically.\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#skill-writes)  Skill writes\n\nSkill Workshop writes only under:\n\n```\n<workspace>/skills/<normalized-skill-name>/\n```\n\nSkill names are normalized:\n\n- lowercased\n- non `[a-z0-9_-]` runs become `-`\n- leading/trailing non-alphanumerics are removed\n- max length is 80 characters\n- final name must match `[a-z0-9][a-z0-9_-]{1,79}`\n\nFor `create`:\n\n- if the skill does not exist, Skill Workshop writes a new `SKILL.md`\n- if it already exists, Skill Workshop appends the body to `## Workflow`\n\nFor `append`:\n\n- if the skill exists, Skill Workshop appends to the requested section\n- if it does not exist, Skill Workshop creates a minimal skill then appends\n\nFor `replace`:\n\n- the skill must already exist\n- `oldText` must be present exactly\n- only the first exact match is replaced\n\nAll writes are atomic and refresh the in-memory skills snapshot immediately, so\nthe new or updated skill can become visible without a Gateway restart.\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#safety-model)  Safety model\n\nSkill Workshop has a safety scanner on generated `SKILL.md` content and support\nfiles.Critical findings quarantine proposals:\n\n| Rule id | Blocks content that… |\n| --- | --- |\n| `prompt-injection-ignore-instructions` | tells the agent to ignore prior/higher instructions |\n| `prompt-injection-system` | references system prompts, developer messages, or hidden instructions |\n| `prompt-injection-tool` | encourages bypassing tool permission/approval |\n| `shell-pipe-to-shell` | includes `curl`/`wget` piped into `sh`, `bash`, or `zsh` |\n| `secret-exfiltration` | appears to send env/process env data over the network |\n\nWarn findings are retained but do not block by themselves:\n\n| Rule id | Warns on… |\n| --- | --- |\n| `destructive-delete` | broad `rm -rf` style commands |\n| `unsafe-permissions` | `chmod 777` style permission use |\n\nQuarantined proposals:\n\n- keep `scanFindings`\n- keep `quarantineReason`\n- appear in `list_quarantine`\n- cannot be applied through `apply`\n\nTo recover from a quarantined proposal, create a new safe proposal with the\nunsafe content removed. Do not edit the store JSON by hand.\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#prompt-guidance)  Prompt guidance\n\nWhen enabled, Skill Workshop injects a short prompt section that tells the agent\nto use `skill_workshop` for durable procedural memory.The guidance emphasizes:\n\n- procedures, not facts/preferences\n- user corrections\n- non-obvious successful procedures\n- recurring pitfalls\n- stale/thin/wrong skill repair through append/replace\n- saving reusable procedure after long tool loops or hard fixes\n- short imperative skill text\n- no transcript dumps\n\nThe write mode text changes with `approvalPolicy`:\n\n- pending mode: queue suggestions; apply only after explicit approval\n- auto mode: apply safe workspace-skill updates when clearly reusable\n\n## [​](https://docs.openclaw.ai/plugins/skill-workshop\\#costs-and-runtime-behavior)  Costs and runtime behavior\n\nHeuristic capture does not call a model.LLM review uses an embedded run on the active/default agent model. It is\nthreshold-based so it does not run on every turn by default.The reviewer:\n\n- uses the same configured provider/model context when available\n- falls back to runtime agent defaults\n- has `reviewTimeoutMs`\n- uses lightweight bootstrap context\n- has no tools\n- writes nothing directly\n- can only emit a proposal that goes through the normal s...(content truncated)

---

## Codex harness
**Source:** https://docs.openclaw.ai/plugins/codex-harness

[Skip to main content](https://docs.openclaw.ai/plugins/codex-harness#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Codex harness

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [What this plugin changes](https://docs.openclaw.ai/plugins/codex-harness#what-this-plugin-changes)
- [Route map](https://docs.openclaw.ai/plugins/codex-harness#route-map)
- [Pick the right model prefix](https://docs.openclaw.ai/plugins/codex-harness#pick-the-right-model-prefix)
- [What doctor warnings mean](https://docs.openclaw.ai/plugins/codex-harness#what-doctor-warnings-mean)
- [Requirements](https://docs.openclaw.ai/plugins/codex-harness#requirements)
- [Minimal config](https://docs.openclaw.ai/plugins/codex-harness#minimal-config)
- [Add Codex alongside other models](https://docs.openclaw.ai/plugins/codex-harness#add-codex-alongside-other-models)
- [Agent command routing](https://docs.openclaw.ai/plugins/codex-harness#agent-command-routing)
- [Codex-only deployments](https://docs.openclaw.ai/plugins/codex-harness#codex-only-deployments)
- [Per-agent Codex](https://docs.openclaw.ai/plugins/codex-harness#per-agent-codex)
- [Model discovery](https://docs.openclaw.ai/plugins/codex-harness#model-discovery)
- [App-server connection and policy](https://docs.openclaw.ai/plugins/codex-harness#app-server-connection-and-policy)
- [Computer use](https://docs.openclaw.ai/plugins/codex-harness#computer-use)
- [Common recipes](https://docs.openclaw.ai/plugins/codex-harness#common-recipes)
- [Codex command](https://docs.openclaw.ai/plugins/codex-harness#codex-command)
- [Hook boundaries](https://docs.openclaw.ai/plugins/codex-harness#hook-boundaries)
- [V1 support contract](https://docs.openclaw.ai/plugins/codex-harness#v1-support-contract)
- [Tools, media, and compaction](https://docs.openclaw.ai/plugins/codex-harness#tools-media-and-compaction)
- [Troubleshooting](https://docs.openclaw.ai/plugins/codex-harness#troubleshooting)
- [Related](https://docs.openclaw.ai/plugins/codex-harness#related)

The bundled `codex` plugin lets OpenClaw run embedded agent turns through the
Codex app-server instead of the built-in PI harness.Use this when you want Codex to own the low-level agent session: model
discovery, native thread resume, native compaction, and app-server execution.
OpenClaw still owns chat channels, session files, model selection, tools,
approvals, media delivery, and the visible transcript mirror.If you are trying to orient yourself, start with
[Agent runtimes](https://docs.openclaw.ai/concepts/agent-runtimes). The short version is:
`openai/gpt-5.5` is the model ref, `codex` is the runtime, and Telegram,
Discord, Slack, or another channel remains the communication surface.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#what-this-plugin-changes)  What this plugin changes

The bundled `codex` plugin contributes several separate capabilities:

| Capability | How you use it | What it does |
| --- | --- | --- |
| Native embedded runtime | `agentRuntime.id: \"codex\"` | Runs OpenClaw embedded agent turns through Codex app-server. |
| Native chat-control commands | `/codex bind`, `/codex resume`, `/codex steer`, … | Binds and controls Codex app-server threads from a messaging conversation. |
| Codex app-server provider/catalog | `codex` internals, surfaced through the harness | Lets the runtime discover and validate app-server models. |
| Codex media-understanding path | `codex/*` image-model compatibility paths | Runs bounded Codex app-server turns for supported image understanding models. |
| Native hook relay | Plugin hooks around Codex-native events | Lets OpenClaw observe/block supported Codex-native tool/finalization events. |

Enabling the plugin makes those capabilities available. It does **not**:

- start using Codex for every OpenAI model
- convert `openai-codex/*` model refs into the native runtime
- make ACP/acpx the default Codex path
- hot-switch existing sessions that already recorded a PI runtime
- replace OpenClaw channel delivery, session files, auth-profile storage, or
message routing

The same plugin also owns the native `/codex` chat-control command surface. If
the plugin is enabled and the user asks to bind, resume, steer, stop, or inspect
Codex threads from chat, agents should prefer `/codex ...` over ACP. ACP remains
the explicit fallback when the user asks for ACP/acpx or is testing the ACP
Codex adapter.Native Codex turns keep OpenClaw plugin hooks as the public compatibility layer.
These are in-process OpenClaw hooks, not Codex `hooks.json` command hooks:

- `before_prompt_build`
- `before_compaction`, `after_compaction`
- `llm_input`, `llm_output`
- `before_tool_call`, `after_tool_call`
- `before_message_write` for mirrored transcript records
- `before_agent_finalize` through Codex `Stop` relay
- `agent_end`

Plugins can also register runtime-neutral tool-result middleware to rewrite
OpenClaw dynamic tool results after OpenClaw executes the tool and before the
result is returned to Codex. This is separate from the public
`tool_result_persist` plugin hook, which transforms OpenClaw-owned transcript
tool-result writes.For the plugin hook semantics themselves, see [Plugin hooks](https://docs.openclaw.ai/plugins/hooks)
and [Plugin guard behavior](https://docs.openclaw.ai/tools/plugin).The harness is off by default. New configs should keep OpenAI model refs
canonical as `openai/gpt-*` and explicitly force
`agentRuntime.id: \"codex\"` or `OPENCLAW_AGENT_RUNTIME=codex` when they
want native app-server execution. Legacy `codex/*` model refs still auto-select
the harness for compatibility, but runtime-backed legacy provider prefixes are
not shown as normal model/provider choices.If the `codex` plugin is enabled but the primary model is still
`openai-codex/*`, `openclaw doctor` warns instead of changing the route. That is
intentional: `openai-codex/*` remains the PI Codex OAuth/subscription path, and
native app-server execution stays an explicit runtime choice.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#route-map)  Route map

Use this table before changing config:

| Desired behavior | Model ref | Runtime config | Plugin requirement | Expected status label |
| --- | --- | --- | --- | --- |
| OpenAI API through normal OpenClaw runner | `openai/gpt-*` | omitted or `runtime: \"pi\"` | OpenAI provider | `Runtime: OpenClaw Pi Default` |
| Codex OAuth/subscription through PI | `openai-codex/gpt-*` | omitted or `runtime: \"pi\"` | OpenAI Codex OAuth provider | `Runtime: OpenClaw Pi Default` |
| Native Codex app-server embedded turns | `openai/gpt-*` | `agentRuntime.id: \"codex\"` | `codex` plugin | `Runtime: OpenAI Codex` |
| Mixed providers with conservative auto mode | provider-specific refs | `agentRuntime.id: \"auto\"` | Optional plugin runtimes | Depends on selected runtime |
| Explicit Codex ACP adapter session | ACP prompt/model dependent | `sessions_spawn` with `runtime: \"acp\"` | healthy `acpx` backend | ACP task/session status |

The important split is provider versus runtime:

- `openai-codex/*` answers “which provider/auth route should PI use?”
- `agentRuntime.id: \"codex\"` answers “which loop should execute this
embedded turn?”
- `/codex ...` answers “which native Codex conversation should this chat bind
or control?”
- ACP answers “which external harness process should acpx launch?”

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#pick-the-right-model-prefix)  Pick the right model prefix

OpenAI-family routes are prefix-specific. Use `openai-codex/*` when you want
Codex OAuth through PI; use `openai/*` when you want direct OpenAI API access or
when you are forcing the native Codex app-server harness:

| Model ref | Runtime path | Use when |
| --- | --- | --- |
| `openai/gpt-5.4` | OpenAI provider through OpenClaw/PI plumbing | You want current direct OpenAI Platform API access with `OPENAI_API_KEY`. |
| `openai-codex/gpt-5.5` | OpenAI Codex OAuth through OpenClaw/PI | You want ChatGPT/Codex subscription auth with the default PI runner. |
| `openai/gpt-5.5` \\+ `agentRuntime.id: \"codex\"` | Codex app-server harness | You want native Codex app-server execution for the embedded agent turn. |

GPT-5.5 is currently subscription/OAuth-only in OpenClaw. Use
`openai-codex/gpt-5.5` for PI OAuth, or `openai/gpt-5.5` with the Codex
app-server harness. Direct API-key access for `openai/gpt-5.5` is supported
once OpenAI enables GPT-5.5 on the public API.Legacy `codex/gpt-*` refs remain accepted as compatibility aliases. Doctor
compatibility migration rewrites legacy primary runtime refs to canonical model
refs and records the runtime policy separately, while fallback-only legacy refs
are left unchanged because runtime is configured for the whole agent container.
New PI Codex OAuth configs should use `openai-codex/gpt-*`; new native
app-server harness configs should use `openai/gpt-*` plus
`agentRuntime.id: \"codex\"`.`agents.defaults.imageModel` follows the same prefix split. Use
`openai-codex/gpt-*` when image understanding should run through the OpenAI
Codex OAuth provider path. Use `codex/gpt-*` when image understanding should run
through a bounded Codex app-server turn. The Codex app-server model must
advertise image input support; text-only Codex models fail before the media turn
starts.Use `/status` to confirm the effective harness for the current session. If the
selection is surprising, enable debug logging for the `agents/harness` subsystem
and inspect the gateway’s structured `agent harness selected` record. It
includes the selected harness id, selection reason, runtime/fallback policy, and,
in `auto` mode, each plugin candidate’s support result.

### [​](https://docs.openclaw.ai/plugins/codex-harness\\#what-doctor-warnings-mean)  What doctor warnings mean

`openclaw doctor` warns when all of these are true:

- the bundled `codex` plugin is enabled or allowed
- an agent’s primary model is `openai-codex/*`
- that agent’s effective runtime is not `codex`

That warning exists because users often expect “Codex plugin enabled” to imply
“native Codex app-server runtime.” OpenClaw does not make that leap. The warning
means:

- **No change is required** if you intended ChatGPT/Codex OAuth through PI.
- Change the model to `openai/<model>` and set
`agentRuntime.id: \"codex\"` if you intended native app-server
execution.
- Existing sessions still need `/new` or `/reset` after a runtime change,
because session runtime pins are sticky.

Harness selection is not a live session control. When an embedded turn runs,
OpenClaw records the selected harness id on that session and keeps using it for
later turns in the same session id. Change `agentRuntime` config or
`OPENCLAW_AGENT_RUNTIME` when you want future sessions to use another harness;
use `/new` or `/reset` to start a fresh session before switching an existing
conversation between PI and Codex. This avoids replaying one transcript through
two incompatible native session systems.Legacy sessions created before harness pins are treated as PI-pinned once they
have transcript history. Use `/new` or `/reset` to opt that conversation into
Codex after changing config.`/status` shows the effective model runtime. The default PI harness appears as
`Runtime: OpenClaw Pi Default`, and the Codex app-server harness appears as
`Runtime: OpenAI Codex`.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#requirements)  Requirements

- OpenClaw with the bundled `codex` plugin available.
- Codex app-server `0.125.0` or newer. The bundled plugin manages a compatible
Codex app-server binary by default, so local `codex` commands on `PATH` do
not affect normal harness startup.
- Codex auth available to the app-server process.

The plugin blocks older or unversioned app-server handshakes. That keeps
OpenClaw on the protocol surface it has been tested against.For live and Docker smoke tests, auth usually comes from `OPENAI_API_KEY`, plus
optional Codex CLI files such as `~/.codex/auth.json` and
`~/.codex/config.toml`. Use the same auth material your local Codex app-server
uses.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#minimal-config)  Minimal config

Use `openai/gpt-5.5`, enable the bundled plugin, and force the `codex` harness:

```
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
      },
    },
  },
  agents: {
    defaults: {
      model: \"openai/gpt-5.5\",
      agentRuntime: {
        id: \"codex\",
      },
    },
  },
}
```

If your config uses `plugins.allow`, include `codex` there too:

```
{
  plugins: {
    allow: [\"codex\"],
    entries: {
      codex: {
        enabled: true,
      },
    },
  },
}
```

Legacy configs that set `agents.defaults.model` or an agent model to
`codex/<model>` still auto-enable the bundled `codex` plugin. New configs should
prefer `openai/<model>` plus the explicit `agentRuntime` entry above.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#add-codex-alongside-other-models)  Add Codex alongside other models

Do not set `agentRuntime.id: \"codex\"` globally if the same agent should freely switch
between Codex and non-Codex provider models. A forced runtime applies to every
embedded turn for that agent or session. If you select an Anthropic model while
that runtime is forced, OpenClaw still tries the Codex harness and fails closed
instead of silently routing that turn through PI.Use one of these shapes instead:

- Put Codex on a dedicated agent with `agentRuntime.id: \"codex\"`.
- Keep the default agent on `agentRuntime.id: \"auto\"` and PI fallback for normal mixed
provider usage.
- Use legacy `codex/*` refs only for compatibility. New configs should prefer
`openai/*` plus an explicit Codex runtime policy.

For example, this keeps the default agent on normal automatic selection and
adds a separate Codex agent:

```
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
      },
    },
  },
  agents: {
    defaults: {
      agentRuntime: {
        id: \"auto\",
        fallback: \"pi\",
      },
    },
    list: [\\\
      {\\\
        id: \"main\\\",\\\
        default: true,\\\n        model: \"anthropic/claude-opus-4-6\\\",\\\
      },\\\n      {\\\
        id: \"codex\\\",\\\
        name: \"Codex\\\",\\\
        model: \"openai/gpt-5.5\\\",\\\
        agentRuntime: {\\\
          id: \"codex\\\",\\\
        },\\\n      },\\\n    ],
  },
}
```

With this shape:

- The default `main` agent uses the normal provider path and PI compatibility fallback.
- The `codex` agent uses the Codex app-server harness.
- If Codex is missing or unsupported for the `codex` agent, the turn fails
instead of quietly using PI.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#agent-command-routing)  Agent command routing

Agents should route user requests by intent, not by the word “Codex” alone:

| User asks for… | Agent should use… |
| --- | --- |
| ”Bind this chat to Codex” | `/codex bind` |
| ”Resume Codex thread `<id>` here” | `/codex resume <id>` |
| ”Show Codex threads” | `/codex threads` |
| ”Use Codex as the runtime for this agent” | config change to `agentRuntime.id` |
| ”Use my ChatGPT/Codex subscription with normal OpenClaw” | `openai-codex/*` model refs |
| ”Run Codex through ACP/acpx” | ACP `sessions_spawn({ runtime: \"acp\", ... })` |
| ”Start Claude Code/Gemini/OpenCode/Cursor in a thread” | ACP/acpx, not `/codex` and not native sub-agents |

OpenClaw only advertises ACP spawn guidance to agents when ACP is enabled,
dispatchable, and backed by a loaded runtime backend. If ACP is not available,
the system prompt and plugin skills should not teach the agent about ACP
routing.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#codex-only-deployments)  Codex-only deployments

Force the Codex harness when you need to prove that every embedded agent turn
uses Codex. Explicit plugin runtimes default to no PI fallback, so
`fallback: \"none\"` is optional but often useful as documentation:

```
{
  agents: {
    defaults: {
      model: \"openai/gpt-5.5\",
      agentRuntime: {
        id: \"codex\",
        fallback: \"none\",
      },
    },
  },
}
```

Environment override:

```
OPENCLAW_AGENT_RUNTIME=codex openclaw gateway run
```

With Codex forced, OpenClaw fails early if the Codex plugin is disabled, the
app-server is too old, or the app-server cannot start. Set
`OPENCLAW_AGENT_HARNESS_FALLBACK=pi` only if you intentionally want PI to handle
missing harness selection.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#per-agent-codex)  Per-agent Codex

You can make one agent Codex-only while the default agent keeps normal
auto-selection:

```
{
  agents: {
    defaults: {
      agentRuntime: {
        id: \"auto\",
        fallback: \"pi\",
      },
    },
    list: [\\\
      {\\\\n        id: \"main\\\\",\\\\n        default: true,\\\\n        model: \"anthropic/claude-opus-4-6\\\\",\\\\n      },\\\\n      {\\\\n        id: \"codex\\\\",\\\\n        name: \"Codex\\\\",\\\\n        model: \"openai/gpt-5.5\\\\",\\\\n        agentRuntime: {\\\\n          id: \"codex\\\\",\\\\n          fallback: \"none\\\\",\\\\n        },\\\\n      },\\\\n    ],
  },
}
```

Use normal session commands to switch agents and models. `/new` creates a fresh
OpenClaw session and the Codex harness creates or resumes its sidecar app-server
thread as needed. `/reset` clears the OpenClaw session binding for that thread
and lets the next turn resolve the harness from current config again.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#model-discovery)  Model discovery

By default, the Codex plugin asks the app-server for available models. If
discovery fails or times out, it uses a bundled fallback catalog for:

- GPT-5.5
- GPT-5.4 mini
- GPT-5.2

You can tune discovery under `plugins.entries.codex.config.discovery`:

```
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          discovery: {
            enabled: true,
            timeoutMs: 2500,
          },
        },
      },
    },
  },
}
```

Disable discovery when you want startup to avoid probing Codex and stick to the
fallback catalog:

```
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          discovery: {
            enabled: false,
          },
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#app-server-connection-and-policy)  App-server connection and policy

If the app-server connection fails, OpenClaw falls back to the PI runtime. This
is a soft failure. If the app-server connection succeeds but the app-server
returns an error, OpenClaw fails hard. This is a hard failure. You can tune the
connection timeout under `plugins.entries.codex.config.connection`:

```
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          connection: {
            timeoutMs: 5000,
          },
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#computer-use)  Computer use

Codex app-server turns can use the `computer` tool. This is a special tool that
lets Codex app-server turns execute commands on the local machine. This is a
powerful tool that should be used with caution. You can enable computer use
under `plugins.entries.codex.config.computer`:

```
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          computer: {
            enabled: true,
          },
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#common-recipes)  Common recipes

### [​](https://docs.openclaw.ai/plugins/codex-harness\\#codex-command)  Codex command

Use the `/codex` command to interact with the Codex app-server. This command is
only available when the `codex` plugin is enabled. You can use the following
subcommands:

- `/codex bind`: Binds the current chat to a Codex app-server thread.
- `/codex resume <id>`: Resumes a Codex app-server thread with the given ID.
- `/codex steer`: Steers the Codex app-server thread.
- `/codex threads`: Shows all active Codex app-server threads.

### [​](https://docs.openclaw.ai/plugins/codex-harness\\#hook-boundaries)  Hook boundaries

The `codex` plugin defines several hook boundaries that let you customize its
behavior. These hooks are called at different stages of the Codex app-server
turn. You can use these hooks to:

- Modify the prompt before it is sent to the Codex app-server.
- Modify the response after it is received from the Codex app-server.
- Add custom tools to the Codex app-server.

See [Plugin hooks](https://docs.openclaw.ai/plugins/hooks) for more information.

### [​](https://docs.openclaw.ai/plugins/codex-harness\\#v1-support-contract)  V1 support contract

The `codex` plugin supports the V1 support contract. This means that it is
compatible with older versions of OpenClaw. You can use the `v1` config option
to enable V1 support:

```
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          v1: {
            enabled: true,
          },
        },
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/plugins/codex-harness\\#tools-media-and-compaction)  Tools, media, and compaction

The `codex` plugin supports tools, media, and compaction. You can configure
these options under `plugins.entries.codex.config`:

```
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          tools: {
            enabled: true,
          },
          media: {
            enabled: true,
          },
          compaction: {
            enabled: true,
          },
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#troubleshooting)  Troubleshooting

If you encounter issues with the `codex` plugin, you can try the following:

- Check the OpenClaw logs for error messages.
- Check the Codex app-server logs for error messages.
- Ensure that the Codex app-server is running and accessible.
- Ensure that the Codex app-server is version `0.125.0` or newer.
- Ensure that the Codex app-server has access to the necessary auth material.

## [​](https://docs.openclaw.ai/plugins/codex-harness\\#related)  Related

- [Agent runtimes](https://docs.openclaw.ai/concepts/agent-runtimes)
- [Plugin hooks](https://docs.openclaw.ai/plugins/hooks)
- [Plugin guard behavior](https://docs.openclaw.ai/tools/plugin)

---

## Zalo personal plugin - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/zalouser

[Skip to main content](https://docs.openclaw.ai/plugins/zalouser#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Zalo personal plugin

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Zalo Personal (plugin)](https://docs.openclaw.ai/plugins/zalouser#zalo-personal-plugin)
- [Naming](https://docs.openclaw.ai/plugins/zalouser#naming)
- [Where it runs](https://docs.openclaw.ai/plugins/zalouser#where-it-runs)
- [Install](https://docs.openclaw.ai/plugins/zalouser#install)
- [Option A: install from npm](https://docs.openclaw.ai/plugins/zalouser#option-a-install-from-npm)
- [Option B: install from a local folder (dev)](https://docs.openclaw.ai/plugins/zalouser#option-b-install-from-a-local-folder-dev)
- [Config](https://docs.openclaw.ai/plugins/zalouser#config)
- [CLI](https://docs.openclaw.ai/plugins/zalouser#cli)
- [Agent tool](https://docs.openclaw.ai/plugins/zalouser#agent-tool)
- [Related](https://docs.openclaw.ai/plugins/zalouser#related)

# [​](https://docs.openclaw.ai/plugins/zalouser\\#zalo-personal-plugin)  Zalo Personal (plugin)

Zalo Personal support for OpenClaw via a plugin, using native `zca-js` to automate a normal Zalo user account.

Unofficial automation may lead to account suspension or ban. Use at your own risk.

## [​](https://docs.openclaw.ai/plugins/zalouser\\#naming)  Naming

Channel id is `zalouser` to make it explicit this automates a **personal Zalo user account** (unofficial). We keep `zalo` reserved for a potential future official Zalo API integration.

## [​](https://docs.openclaw.ai/plugins/zalouser\\#where-it-runs)  Where it runs

This plugin runs **inside the Gateway process**.If you use a remote Gateway, install/configure it on the **machine running the Gateway**, then restart the Gateway.No external `zca`/`openzca` CLI binary is required.

## [​](https://docs.openclaw.ai/plugins/zalouser\\#install)  Install

### [​](https://docs.openclaw.ai/plugins/zalouser\\#option-a-install-from-npm)  Option A: install from npm

```
openclaw plugins install @openclaw/zalouser
```

Restart the Gateway afterwards.

### [​](https://docs.openclaw.ai/plugins/zalouser\\#option-b-install-from-a-local-folder (dev))  Option B: install from a local folder (dev)

```
PLUGIN_SRC=./path/to/local/zalouser-plugin
openclaw plugins install \"$PLUGIN_SRC\"
cd \"$PLUGIN_SRC\" && pnpm install
```

Restart the Gateway afterwards.

## [​](https://docs.openclaw.ai/plugins/zalouser\\#config)  Config

Channel config lives under `channels.zalouser` (not `plugins.entries.*`):

```
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: \"pairing\",
    },
  },
}
```

## [​](https://docs.openclaw.ai/plugins/zalouser\\#cli)  CLI

```
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message \"Hello from OpenClaw\"
openclaw directory peers list --channel zalouser --query \"name\"
```

## [​](https://docs.openclaw.ai/plugins/zalouser\\#agent-tool)  Agent tool

Tool name: `zalouser`Actions: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`Channel message actions also support `react` for message reactions.

## [​](https://docs.openclaw.ai/plugins/zalouser\\#related)  Related

- [Building plugins](https://docs.openclaw.ai/plugins/building-plugins)
- [Community plugins](https://docs.openclaw.ai/plugins/community)

[Skill workshop plugin](https://docs.openclaw.ai/plugins/skill-workshop) [Getting Started](https://docs.openclaw.ai/plugins/building-plugins)

Ctrl+I

---

## Message presentation - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/message-presentation

[Skip to main content](https://docs.openclaw.ai/plugins/message-presentation#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Message presentation

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Contract](https://docs.openclaw.ai/plugins/message-presentation#contract)
- [Producer examples](https://docs.openclaw.ai/plugins/message-presentation#producer-examples)
- [Renderer contract](https://docs.openclaw.ai/plugins/message-presentation#renderer-contract)
- [Core render flow](https://docs.openclaw.ai/plugins/message-presentation#core-render-flow)
- [Degradation rules](https://docs.openclaw.ai/plugins/message-presentation#degradation-rules)
- [Provider mapping](https://docs.openclaw.ai/plugins/message-presentation#provider-mapping)
- [Presentation vs InteractiveReply](https://docs.openclaw.ai/plugins/message-presentation#presentation-vs-interactivereply)
- [Delivery pin](https://docs.openclaw.ai/plugins/message-presentation#delivery-pin)
- [Plugin author checklist](https://docs.openclaw.ai/plugins/message-presentation#plugin-author-checklist)
- [Related docs](https://docs.openclaw.ai/plugins/message-presentation#related-docs)

Message presentation is OpenClaw’s shared contract for rich outbound chat UI.
It lets agents, CLI commands, approval flows, and plugins describe the message
intent once, while each channel plugin renders the best native shape it can.Use presentation for portable message UI:

- text sections
- small context/footer text
- dividers
- buttons
- select menus
- card title and tone

Do not add new provider-native fields such as Discord `components`, Slack
`blocks`, Telegram `buttons`, Teams `card`, or Feishu `card` to the shared
message tool. Those are renderer outputs owned by the channel plugin.

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#contract)  Contract

Plugin authors import the public contract from:

```
import type {
  MessagePresentation,
  ReplyPayloadDelivery,
} from \"openclaw/plugin-sdk/interactive-runtime\";
```

Shape:

```
type MessagePresentation = {
  title?: string;
  tone?: \"neutral\" | \"info\" | \"success\" | \"warning\" | \"danger\";
  blocks: MessagePresentationBlock[];
};

type MessagePresentationBlock =
  | { type: \"text\"; text: string }
  | { type: \"context\"; text: string }
  | { type: \"divider\" }
  | { type: \"buttons\"; buttons: MessagePresentationButton[] }
  | { type: \"select\"; placeholder?: string; options: MessagePresentationOption[] };

type MessagePresentationButton = {
  label: string;
  value?: string;
  url?: string;
  style?: \"primary\" | \"secondary\" | \"success\" | \"danger\";
};

type MessagePresentationOption = {
  label: string;
  value: string;
};

type ReplyPayloadDelivery = {
  pin?:
    | boolean
    | {
        enabled: boolean;
        notify?: boolean;
        required?: boolean;
      };
};
```

Button semantics:

- `value` is an application action value routed back through the channel’s
existing interaction path when the channel supports clickable controls.
- `url` is a link button. It can exist without `value`.
- `label` is required and is also used in text fallback.
- `style` is advisory. Renderers should map unsupported styles to a safe
default, not fail the send.

Select semantics:

- `options[].value` is the selected application value.
- `placeholder` is advisory and may be ignored by channels without native
select support.
- If a channel does not support selects, fallback text lists the labels.

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#producer-examples)  Producer examples

Simple card:

```
{
  \"title\": \"Deploy approval\",
  \"tone\": \"warning\",
  \"blocks\": [\\\n    { \"type\": \"text\", \"text\": \"Canary is ready to promote.\" },\\\n    { \"type\": \"context\", \"text\": \"Build 1234, staging passed.\" },\\\n    {\\\n      \"type\": \"buttons\",\\\n      \"buttons\": [\\\n        { \"label\": \"Approve\", \"value\": \"deploy:approve\", \"style\": \"success\" },\\\n        { \"label\": \"Decline\", \"value\": \"deploy:decline\", \"style\": \"danger\" }\\\n      ]\\\n    }\\\n  ]
}
```

URL-only link button:

```
{
  \"blocks\": [\\\n    { \"type\": \"text\", \"text\": \"Release notes are ready.\" },\\\n    {\\\n      \"type\": \"buttons\",\\\n      \"buttons\": [{ \"label\": \"Open notes\", \"url\": \"https://example.com/release\" }]\\\n    }\\\n  ]
}
```

Select menu:

```
{
  \"title\": \"Choose environment\",
  \"blocks\": [\\\n    {\\\n      \"type\": \"select\",\\\n      \"placeholder\": \"Environment\",\\\n      \"options\": [\\\n        { \"label\": \"Canary\", \"value\": \"env:canary\" },\\\n        { \"label\": \"Production\", \"value\": \"env:prod\" }\\\n      ]\\\n    }\\\n  ]
}
```

CLI send:

```
openclaw message send --channel slack \\\n  --target channel:C123 \\\n  --message \"Deploy approval\" \\\n  --presentation \'{\"title\":\"Deploy approval\",\"tone\":\"warning\",\"blocks\":[{\"type\":\"text\",\"text\":\"Canary is ready.\"},{\"type\":\"buttons\",\"buttons\":[{\"label\":\"Approve\",\"value\":\"deploy:approve\",\"style\":\"success\"},{\"label\":\"Decline\",\"value\":\"deploy:decline\",\"style\":\"danger\"}]}]}\
```

Pinned delivery:

```
openclaw message send --channel telegram \\\n  --target -1001234567890 \\\n  --message \"Topic opened\" \\\n  --pin
```

Pinned delivery with explicit JSON:

```
{
  \"pin\": {
    \"enabled\": true,
    \"notify\": true,
    \"required\": false
  }
}
```

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#renderer-contract)  Renderer contract

Channel plugins declare render support on their outbound adapter:

```
const adapter: ChannelOutboundAdapter = {
  deliveryMode: \"direct\",
  presentationCapabilities: {
    supported: true,
    buttons: true,
    selects: true,
    context: true,
    divider: true,
  },
  deliveryCapabilities: {
    pin: true,
  },
  renderPresentation({ payload, presentation, ctx }) {
    return renderNativePayload(payload, presentation, ctx);
  },
  async pinDeliveredMessage({ target, messageId, pin }) {
    await pinNativeMessage(target, messageId, { notify: pin.notify === true });
  },
};
```

Capability fields are intentionally simple booleans. They describe what the
renderer can make interactive, not every native platform limit. Renderers still
own platform-specific limits such as maximum button count, block count, and
card size.

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#core-render-flow)  Core render flow

When a `ReplyPayload` or message action includes `presentation`, core:

1. Normalizes the presentation payload.
2. Resolves the target channel’s outbound adapter.
3. Reads `presentationCapabilities`.
4. Calls `renderPresentation` when the adapter can render the payload.
5. Falls back to conservative text when the adapter is absent or cannot render.
6. Sends the resulting payload through the normal channel delivery path.
7. Applies delivery metadata such as `delivery.pin` after the first successful
sent message.

Core owns fallback behavior so producers can stay channel-agnostic. Channel
plugins own native rendering and interaction handling.

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#degradation-rules)  Degradation rules

Presentation must be safe to send on limited channels.Fallback text includes:

- `title` as the first line
- `text` blocks as normal paragraphs
- `context` blocks as compact context lines
- `divider` blocks as a visual separator
- button labels, including URLs for link buttons
- select option labels

Unsupported native controls should degrade rather than fail the whole send.
Examples:

- Telegram with inline buttons disabled sends text fallback.
- A channel without select support lists select options as text.
- A URL-only button becomes either a native link button or a fallback URL line.
- Optional pin failures do not fail the delivered message.

The main exception is `delivery.pin.required: true`; if pinning is requested as
required and the channel cannot pin the sent message, delivery reports failure.

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#provider-mapping)  Provider mapping

Current bundled renderers:

| Channel | Native render target | Notes |
| --- | --- | --- |
| Discord | Components and component containers | Preserves legacy `channelData.discord.components` for existing provider-native payload producers, but new shared sends should use `presentation`. |
| Slack | Block Kit | Preserves legacy `channelData.slack.blocks` for existing provider-native payload producers, but new shared sends should use `presentation`. |
| Telegram | Text plus inline keyboards | Buttons/selects require inline button capability for the target surface; otherwise text fallback is used. |
| Mattermost | Text plus interactive props | Other blocks degrade to text. |
| Microsoft Teams | Adaptive Cards | Plain `message` text is included with the card when both are provided. |
| Feishu | Interactive cards | Card header can use `title`; body avoids duplicating that title. |
| Plain channels | Text fallback | Channels without a renderer still get readable output. |

Provider-native payload compatibility is a transition affordance for existing
reply producers. It is not a reason to add new shared native fields.

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#presentation-vs-interactivereply)  Presentation vs InteractiveReply

`InteractiveReply` is the older internal subset used by approval and interaction
helpers. It supports:

- text
- buttons
- selects

`MessagePresentation` is the canonical shared send contract. It adds:

- title
- tone
- context
- divider
- URL-only buttons
- generic delivery metadata through `ReplyPayload.delivery`

Use helpers from `openclaw/plugin-sdk/interactive-runtime` when bridging older
code:

```
import {
  interactiveReplyToPresentation,
  normalizeMessagePresentation,
  presentationToInteractiveReply,
  renderMessagePresentationFallbackText,
} from \"openclaw/plugin-sdk/interactive-runtime\";
```

New code should accept or produce `MessagePresentation` directly.

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#delivery-pin)  Delivery pin

Pinning is delivery behavior, not presentation. Use `delivery.pin` instead of
provider-native fields such as `channelData.telegram.pin`.Semantics:

- `pin: true` pins the first successfully delivered message.
- `pin.notify` defaults to `false`.
- `pin.required` defaults to `false`.
- Optional pin failures degrade and leave the sent message intact.
- Required pin failures fail delivery.
- Chunked messages pin the first delivered chunk, not the tail chunk.

Manual `pin`, `unpin`, and `pins` message actions still exist for existing
messages where the provider supports those operations.

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#plugin-author-checklist)  Plugin author checklist

- Declare `presentation` from `describeMessageTool(...)` when the channel can
render or safely degrade semantic presentation.
- Add `presentationCapabilities` to the runtime outbound adapter.
- Implement `renderPresentation` in runtime code, not control-plane plugin
setup code.
- Keep native UI libraries out of hot setup/catalog paths.
- Preserve platform limits in the renderer and tests.
- Add fallback tests for unsupported buttons, selects, URL buttons, title/text
duplication, and mixed `message` plus `presentation` sends.
- Add delivery pin support through `deliveryCapabilities.pin` and
`pinDeliveredMessage` only when the provider can pin the sent message id.
- Do not expose new provider-native card/block/component/button fields through
the shared message action schema.

## [​](https://docs.openclaw.ai/plugins/message-presentation\\#related-docs)  Related docs

- [Message CLI](https://docs.openclaw.ai/cli/message)
- [Plugin SDK Overview](https://docs.openclaw.ai/plugins/sdk-overview)
- [Plugin Architecture](https://docs.openclaw.ai/plugins/architecture-internals#message-tool-schemas)
- [Channel Presentation Refactor Plan](https://docs.openclaw.ai/plan/ui-channels)

[Memory LanceDB](https://docs.openclaw.ai/plugins/memory-lancedb) [Skill workshop plugin](https://docs.openclaw.ai/plugins/skill-workshop)

Ctrl+I

---

## Plugin SDK Overview
**Source:** https://docs.openclaw.ai/plugins/sdk-overview

[Skip to main content](https://docs.openclaw.ai/plugins/sdk-overview#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

SDK reference

Plugin SDK Overview

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Plugin SDK Overview](https://docs.openclaw.ai/plugins/sdk-overview#plugin-sdk-overview)
- [Import convention](https://docs.openclaw.ai/plugins/sdk-overview#import-convention)
- [Subpath reference](https://docs.openclaw.ai/plugins/sdk-overview#subpath-reference)
- [Plugin entry](https://docs.openclaw.ai/plugins/sdk-overview#plugin-entry)
- [Registration API](https://docs.openclaw.ai/plugins/sdk-overview#registration-api)
- [Capability registration](https://docs.openclaw.ai/plugins/sdk-overview#capability-registration)
- [Tools and commands](https://docs.openclaw.ai/plugins/sdk-overview#tools-and-commands)
- [Infrastructure](https://docs.openclaw.ai/plugins/sdk-overview#infrastructure)
- [CLI registration metadata](https://docs.openclaw.ai/plugins/sdk-overview#cli-registration-metadata)
- [CLI backend registration](https://docs.openclaw.ai/plugins/sdk-overview#cli-backend-registration)
- [Exclusive slots](https://docs.openclaw.ai/plugins/sdk-overview#exclusive-slots)
- [Memory embedding adapters](https://docs.openclaw.ai/plugins/sdk-overview#memory-embedding-adapters)
- [Events and lifecycle](https://docs.openclaw.ai/plugins/sdk-overview#events-and-lifecycle)
- [Hook decision semantics](https://docs.openclaw.ai/plugins/sdk-overview#hook-decision-semantics)
- [API object fields](https://docs.openclaw.ai/plugins/sdk-overview#api-object-fields)
- [Internal module convention](https://docs.openclaw.ai/plugins/sdk-overview#internal-module-convention)
- [Related](https://docs.openclaw.ai/plugins/sdk-overview#related)

# [​](https://docs.openclaw.ai/plugins/sdk-overview\\#plugin-sdk-overview)  Plugin SDK Overview

The plugin SDK is the typed contract between plugins and core. This page is the\nreference for **what to import** and **what you can register**.\n\n**Looking for a how-to guide?**\n\n- First plugin? Start with [Getting Started](https://docs.openclaw.ai/plugins/building-plugins)\n- Channel plugin? See [Channel Plugins](https://docs.openclaw.ai/plugins/sdk-channel-plugins)\n- Provider plugin? See [Provider Plugins](https://docs.openclaw.ai/plugins/sdk-provider-plugins)\n\n## [​](https://docs.openclaw.ai/plugins/sdk-overview\\#import-convention)  Import convention\n\nAlways import from a specific subpath:\n\n```\nimport { definePluginEntry } from \"openclaw/plugin-sdk/plugin-entry\";\nimport { defineChannelPluginEntry } from \"openclaw/plugin-sdk/channel-core\";\n```\n\nEach subpath is a small, self-contained module. This keeps startup fast and\nprevents circular dependency issues. For channel-specific entry/build helpers,\nprefer `openclaw/plugin-sdk/channel-core`; keep `openclaw/plugin-sdk/core` for\nthe broader umbrella surface and shared helpers such as\n`buildChannelConfigSchema`.Do not add or depend on provider-named convenience seams such as\n`openclaw/plugin-sdk/slack`, `openclaw/plugin-sdk/discord`,\n`openclaw/plugin-sdk/signal`, `openclaw/plugin-sdk/whatsapp`, or\nchannel-branded helper seams. Bundled plugins should compose generic\nSDK subpaths inside their own `api.ts` or `runtime-api.ts` barrels, and core\nshould either use those plugin-local barrels or add a narrow generic SDK\ncontract when the need is truly cross-channel.The generated export map still contains a small set of bundled-plugin helper\nseams such as `plugin-sdk/feishu`, `plugin-sdk/feishu-setup`,\n`plugin-sdk/zalo`, `plugin-sdk/zalo-setup`, and `plugin-sdk/matrix*`. Those\nsubpaths exist for bundled-plugin maintenance and compatibility only; they are\nintentionally omitted from the common table below and are not the recommended\nimport path for new third-party plugins.\n\n## [​](https://docs.openclaw.ai/plugins/sdk-overview\\#subpath-reference)  Subpath reference\n\nThe most commonly used subpaths, grouped by purpose. The generated full list of\n200+ subpaths lives in `scripts/lib/plugin-sdk-entrypoints.json`.Reserved bundled-plugin helper subpaths still appear in that generated list.\nTreat those as implementation detail/compatibility surfaces unless a doc page\nexplicitly promotes one as public.\n\n### [​](https://docs.openclaw.ai/plugins/sdk-overview\\#plugin-entry)  Plugin entry\n\n| Subpath | Key exports |\n| --- | --- |\n| `plugin-sdk/plugin-entry` | `definePluginEntry` |\n| `plugin-sdk/core` | `defineChannelPluginEntry`, `createChatChannelPlugin`, `createChannelPluginBase`, `defineSetupPluginEntry`, `buildChannelConfigSchema` |\n| `plugin-sdk/config-schema` | `OpenClawSchema` |\n| `plugin-sdk/provider-entry` | `defineSingleProviderPluginEntry` |\n\nChannel subpaths\n\n| Subpath | Key exports |\n| --- | --- |\n| `plugin-sdk/channel-core` | `defineChannelPluginEntry`, `defineSetupPluginEntry`, `createChatChannelPlugin`, `createChannelPluginBase` |\n| `plugin-sdk/config-schema` | Root `openclaw.json` Zod schema export (`OpenClawSchema`) |\n| `plugin-sdk/channel-setup` | `createOptionalChannelSetupSurface`, `createOptionalChannelSetupAdapter`, `createOptionalChannelSetupWizard`, plus `DEFAULT_ACCOUNT_ID`, `createTopLevelChannelDmPolicy`, `setSetupChannelEnabled`, `splitSetupEntries` |\n| `plugin-sdk/setup` | Shared setup wizard helpers, allowlist prompts, setup status builders |\n| `plugin-sdk/setup-runtime` | `createPatchedAccountSetupAdapter`, `createEnvPatchedAccountSetupAdapter`, `createSetupInputPresenceValidator`, `noteChannelLookupFailure`, `noteChannelLookupSummary`, `promptResolvedAllowFrom`, `splitSetupEntries`, `createAllowlistSetupWizardProxy`, `createDelegatedSetupWizardProxy` |\n| `plugin-sdk/setup-adapter-runtime` | `createEnvPatchedAccountSetupAdapter` |\n| `plugin-sdk/setup-tools` | `formatCliCommand`, `detectBinary`, `extractArchive`, `resolveBrewExecutable`, `formatDocsLink`, `CONFIG_DIR` |\n| `plugin-sdk/account-core` | Multi-account config/action-gate helpers, default-account fallback helpers |\n| `plugin-sdk/account-id` | `DEFAULT_ACCOUNT_ID`, account-id normalization helpers |\n| `plugin-sdk/account-resolution` | Account lookup + default-fallback helpers |\n| `plugin-sdk/account-helpers` | Narrow account-list/account-action helpers |\n| `plugin-sdk/channel-pairing` | `createChannelPairingController` |\n| `plugin-sdk/channel-reply-pipeline` | `createChannelReplyPipeline` |\n| `plugin-sdk/channel-config-helpers` | `createHybridChannelConfigAdapter` |\n| `plugin-sdk/channel-config-schema` | Channel config schema types |\n| `plugin-sdk/telegram-command-config` | Telegram custom-command normalization/validation helpers with bundled-contract fallback |\n| `plugin-sdk/command-gating` | Narrow command authorization gate helpers |\n| `plugin-sdk/channel-policy` | `resolveChannelGroupRequireMention` |\n| `plugin-sdk/channel-lifecycle` | `createAccountStatusSink`, draft stream lifecycle/finalization helpers |\n| `plugin-sdk/inbound-envelope` | Shared inbound route + envelope builder helpers |\n| `plugin-sdk/inbound-reply-dispatch` | Shared inbound record-and-dispatch helpers |\n| `plugin-sdk/messaging-targets` | Target parsing/matching helpers |\n| `plugin-sdk/outbound-media` | Shared outbound media loading helpers |\n| `plugin-sdk/outbound-runtime` | Outbound identity, send delegate, and payload planning helpers |\n| `plugin-sdk/poll-runtime` | Narrow poll normalization helpers |\n| `plugin-sdk/thread-bindings-runtime` | Thread-binding lifecycle and adapter helpers |\n| `plugin-sdk/agent-media-payload` | Legacy agent media payload builder |\n| `plugin-sdk/conversation-runtime` | Conversation/thread binding, pairing, and configured-binding helpers |\n| `plugin-sdk/runtime-config-snapshot` | Runtime config snapshot helper |\n| `plugin-sdk/runtime-group-policy` | Runtime group-policy resolution helpers |\n| `plugin-sdk/channel-status` | Shared channel status snapshot/summary helpers |\n| `plugin-sdk/channel-config-primitives` | Narrow channel config-schema primitives |\n| `plugin-sdk/channel-config-writes` | Channel config-write authorization helpers |\n| `plugin-sdk/channel-plugin-common` | Shared channel plugin prelude exports |\n| `plugin-sdk/allowlist-config-edit` | Allowlist config edit/read helpers |\n| `plugin-sdk/group-access` | Shared group-access decision helpers |\n| `plugin-sdk/direct-dm` | Shared direct-DM auth/guard helpers |\n| `plugin-sdk/interactive-runtime` | Semantic message presentation, delivery, and legacy interactive reply helpers. See [Message Presentation](https://docs.openclaw.ai/plugins/message-presentation) |\n| `plugin-sdk/channel-inbound` | Compatibility barrel for inbound debounce, mention matching, mention-policy helpers, and envelope helpers |\n| `plugin-sdk/channel-mention-gating` | Narrow mention-policy helpers without the broader inbound runtime surface |\n| `plugin-sdk/channel-location` | Channel location context and formatting helpers |\n| `plugin-sdk/channel-logging` | Channel logging helpers for inbound drops and typing/ack failures |\n| `plugin-sdk/channel-send-result` | Reply result types |\n| `plugin-sdk/channel-actions` | Channel message-action helpers, plus deprecated native schema helpers kept for plugin compatibility |\n| `plugin-sdk/channel-targets` | Target parsing/matching helpers |\n| `plugin-sdk/channel-contract` | Channel contract types |\n| `plugin-sdk/channel-feedback` | Feedback/reaction wiring |\n| `plugin-sdk/channel-secret-runtime` | Narrow secret-contract helpers such as `collectSimpleChannelFieldAssignments`, `getChannelSurface`, `pushAssignment`, and secret target types |\n\nProvider subpaths\n\n| Subpath | Key exports |\n| --- | --- |\n| `plugin-sdk/provider-entry` | `defineSingleProviderPluginEntry` |\n| `plugin-sdk/provider-setup` | Curated local/self-hosted provider setup helpers |\n| `plugin-sdk/self-hosted-provider-setup` | Focused OpenAI-compatible self-hosted provider setup helpers |\n| `plugin-sdk/cli-backend` | CLI backend defaults + watchdog constants |\n| `plugin-sdk/provider-auth-runtime` | Runtime API-key resolution helpers for provider plugins |\n| `plugin-sdk/provider-auth-api-key` | API-key onboarding/profile-write helpers such as `upsertApiKeyProfile` |\n| `plugin-sdk/provider-auth-result` | Standard OAuth auth-result builder |\n| `plugin-sdk/provider-auth-login` | Shared interactive login helpers for provider plugins |\n| `plugin-sdk/provider-env-vars` | Provider auth env-var lookup helpers |\n| `plugin-sdk/provider-auth` | `createProviderApiKeyAuthMethod`, `ensureApiKeyFromOptionEnvOrPrompt`, `upsertAuthProfile`, `upsertApiKeyProfile`, `writeOAuthCredentials` |\n| `plugin-sdk/provider-model-shared` | `ProviderReplayFamily`, `buildProviderReplayFamilyHooks`, `normalizeModelCompat`, shared replay-policy builders, provider-endpoint helpers, and model-id normalization helpers such as `normalizeNativeXaiModelId` |\n| `plugin-sdk/provider-catalog-shared` | `findCatalogTemplate`, `buildSingleProviderApiKeyCatalog`, `supportsNativeStreamingUsageCompat`, `applyProviderNativeStreamingUsageCompat` |\n| `plugin-sdk/provider-http` | Generic provider HTTP/endpoint capability helpers, including audio transcription multipart form helpers |\n| `plugin-sdk/provider-web-fetch-contract` | Narrow web-fetch config/selection contract helpers such as `enablePluginInConfig` and `WebFetchProviderPlugin` |\n| `plugin-sdk/provider-web-fetch` | Web-fetch provider registration/cache helpers |\n| `plugin-sdk/provider-web-search-config-contract` | Narrow web-search config/credential helpers for providers that do not need plugin-enable wiring |\n| `plugin-sdk/provider-web-search-contract` | Narrow web-search config/credential contract helpers such as `createWebSearchProviderContractFields`, `enablePluginInConfig`, `resolveProviderWebSearchPluginConfig`, and scoped credential setters/getters |\n| `plugin-sdk/provider-web-search` | Web-search provider registration/cache/runtime helpers |\n| `plugin-sdk/provider-tools` | `ProviderToolCompatFamily`, `buildProviderToolCompatFamilyHooks`, Gemini schema cleanup + diagnostics, and xAI compat helpers such as `resolveXaiModelCompatPatch` / `applyXaiModelCompat` |\n| `plugin-sdk/provider-usage` | `fetchClaudeUsage` and similar |\n| `plugin-sdk/provider-stream` | `ProviderStreamFamily`, `buildProviderStreamFamilyHooks`, `composeProviderStreamWrappers`, stream wrapper types, and shared Anthropic/Bedrock/Google/Kilocode/Moonshot/OpenAI/OpenRouter/Z.A.I/MiniMax/Copilot wrapper helpers |\n| `plugin-sdk/provider-transport-runtime` | Native provider transport helpers such as guarded fetch, transport message transforms, and writable transport event streams |\n| `plugin-sdk/provider-onboard` | Onboarding config patch helpers |\n| `plugin-sdk/global-singleton` | Process-local singleton/map/cache helpers |\n\nAuth and security subpaths\n\n| Subpath | Key exports |\n| --- | --- |\n| `plugin-sdk/command-auth` | `resolveControlCommandGate`, command registry helpers, sender-authorization helpers |\n| `plugin-sdk/command-status` | Command/help message builders such as `buildCommandsMessagePaginated` and `buildHelpMessage` |\n| `plugin-sdk/approval-auth-runtime` | Approver resolution and same-chat action-auth helpers |\n| `plugin-sdk/approval-client-runtime` | Native exec approval profile/filter helpers |\n| `plugin-sdk/approval-delivery-runtime` | Native approval capability/delivery adapters |\n| `plugin-sdk/approval-gateway-runtime` | Shared approval gateway-resolution helper |\n| `plugin-sdk/approval-handler-adapter-runtime` | Lightweight native approval adapter loading helpers for hot channel entrypoints |\n| `plugin-sdk/approval-handler-runtime` | Broader approval handler runtime helpers; prefer the narrower adapter/gateway seams when they are enough |\n| `plugin-sdk/approval-native-runtime` | Native approval target + account-binding helpers |\n| `plugin-sdk/approval-reply-runtime` | Exec/plugin approval reply payload helpers |\n| `plugin-sdk/command-auth-native` | Native command auth + native session-target helpers |\n| `plugin-sdk/command-detection` | Shared command detection helpers |\n| `plugin-sdk/command-surface` | Command-body normalization and command-surface helpers |\n| `plugin-sdk/allow-from` | `formatAllowFromLowercase` |\n| `plugin-sdk/channel-secret-runtime` | Narrow secret-contract collection helpers for channel/plugin secret surfaces |\n| `plugin-sdk/secret-ref-runtime` | Narrow `coerceSecretRef` and SecretRef typing helpers for secret-contract/config parsing |\n| `plugin-sdk/security-runtime` | Shared trust, DM gating, external-content, and secret-collection helpers |\n| `plugin-sdk/ssrf-policy` | Host allowlist and private-network SSRF policy helpers |\n| `plugin-sdk/ssrf-dispatcher` | Narrow pinned-dispatcher helpers without the broad infra runtime surface |\n| `plugin-sdk/ssrf-runtime` | Pinned-dispatcher, SSRF-guarded fetch, and SSRF policy helpers |\n| `plugin-sdk/secret-input` | Secret input parsing helpers |\n| `plugin-sdk/webhook-ingress` | Webhook request/target helpers |\n| `plugin-sdk/webhook-request-guards` | Request body size/timeout helpers |\n\nRuntime and storage subpaths\n\n| Subpath | Key exports |\n| --- | --- |\n| `plugin-sdk/runtime` | Broad runtime/logging/backup/plugin-install helpers |\n| `plugin-sdk/runtime-env` | Narrow runtime env, logger, timeout, retry, and backoff helpers |\n| `plugin-sdk/channel-runtime-context` | Generic channel runtime-context registration and lookup helpers |\n| `plugin-sdk/runtime-store` | `createPluginRuntimeStore` |\n| `plugin-sdk/plugin-runtime` | Shared plugin command/hook/http/interactive helpers |\n| `plugin-sdk/hook-runtime` | Shared webhook/internal hook pipeline helpers |\n| `plugin-sdk/lazy-runtime` | Lazy runtime import/binding helpers such as `createLazyRuntimeModule`, `createLazyRuntimeMethod`, and `createLazyRuntimeSurface` |\n| `plugin-sdk/process-runtime` | Process exec helpers |\n| `plugin-sdk/cli-runtime` | CLI formatting, wait, and version helpers |\n| `plugin-sdk/gateway-runtime` | Gateway client and channel-status patch helpers |\n| `plugin-sdk/config-runtime` | Config load/write helpers and plugin-config lookup helpers |\n| `plugin-sdk/telegram-command-config` | Telegram command-name/description normalization and duplicate/conflict checks, even when the bundled Telegram contract surface is unavailable |\n| `plugin-sdk/text-autolink-runtime` | File-reference autolink detection without the broad text-runtime barrel |\n| `plugin-sdk/approval-runtime` | Exec/plugin approval helpers, approval-capability builders, auth/profile helpers, native routing/runtime helpers |\n| `plugin-sdk/reply-runtime` | Shared inbound/reply runtime helpers, chunking, dispatch, heartbeat, reply planner |\n| `plugin-sdk/reply-dispatch-runtime` | Narrow reply dispatch/finalize helpers |\n| `plugin-sdk/reply-history` | Shared short-window reply-history helpers such as `buildHistoryContext`, `recordPendingHistoryEntry`, and `clearHistoryEntriesIfEnabled` |\n| `plugin-sdk/reply-reference` | `createReplyReferencePlanner` |\n| `plugin-sdk/reply-chunking` | Narrow text/markdown chunking helpers |\n| `plugin-sdk/session-store-runtime` | Session store path + updated-at helpers |\n| `plugin-sdk/state-paths` | State/OAuth dir path helpers |\n| `plugin-sdk/routing` | Route/session-key/account binding helpers such as `resolveAgentRoute`, `buildAgentSessionKey`, and `resolveDefaultAgentBoundAccountId` |\n| `plugin-sdk/status-helpers` | Shared channel/account status summary helpers, runtime-state defaults, and issue metadata helpers |\n| `plugin-sdk/target-resolver-runtime` | Shared target resolver helpers |\n| `plugin-sdk/string-normalization-runtime` | Slug/string normalization helpers |\n| `plugin-sdk/request-url` | Extract string URLs from fetch/request-like inputs |\n| `plugin-sdk/run-command` | Timed command runner with normalized stdout/stderr results |\n| `plugin-sdk/param-readers` | Common tool/CLI param readers |\n| `plugin-sdk/tool-payload` | Extract normalized payloads from tool result objects |\n| `plugin-sdk/tool-send` | Extract canonical send target fields from tool args |\n| `plugin-sdk/temp-path` | Shared temp-download path helpers |\n| `plugin-sdk/logging-core` | Subsystem logger and redaction helpers |\n| `plugin-sdk/markdown-table-runtime` | Markdown table mode helpers |\n| `plugin-sdk/json-store` | Small JSON state read/write helpers |\n| `plugin-sdk/file-lock` | Re-entrant file-lock helpers |\n| `plugin-sdk/persistent-dedupe` | Disk-backed dedupe cache helpers |\n| `plugin-sdk/acp-runtime` | ACP runtime/session and reply-dispatch helpers |\n| `plugin-sdk/acp-binding-resolve-runtime` | Read-only ACP binding resolution wi...(content truncated)

---

## Plugin entry points - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/sdk-entrypoints

[Skip to main content](https://docs.openclaw.ai/plugins/sdk-entrypoints#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

SDK reference

Plugin entry points

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [definePluginEntry](https://docs.openclaw.ai/plugins/sdk-entrypoints#definepluginentry)
- [defineChannelPluginEntry](https://docs.openclaw.ai/plugins/sdk-entrypoints#definechannelpluginentry)
- [defineSetupPluginEntry](https://docs.openclaw.ai/plugins/sdk-entrypoints#definesetuppluginentry)
- [Registration mode](https://docs.openclaw.ai/plugins/sdk-entrypoints#registration-mode)
- [Plugin shapes](https://docs.openclaw.ai/plugins/sdk-entrypoints#plugin-shapes)
- [Related](https://docs.openclaw.ai/plugins/sdk-entrypoints#related)

Every plugin exports a default entry object. The SDK provides three helpers for
creating them.For installed plugins, `package.json` should point runtime loading at built
JavaScript when available:

```
{
  \"openclaw\": {
    \"extensions\": [\"./src/index.ts\"],
    \"runtimeExtensions\": [\"./dist/index.js\"],
    \"setupEntry\": \"./src/setup-entry.ts\"", "runtimeSetupEntry": "./dist/setup-entry.js"
  }
}
```

`extensions` and `setupEntry` remain valid source entries for workspace and git
checkout development. `runtimeExtensions` and `runtimeSetupEntry` are preferred
when OpenClaw loads an installed package and let npm packages avoid runtime
TypeScript compilation. If an installed package only declares a TypeScript
source entry, OpenClaw will use a matching built `dist/*.js` peer when one
exists, then fall back to the TypeScript source.All entry paths must stay inside the plugin package directory. Runtime entries
and inferred built JavaScript peers do not make an escaping `extensions` or
`setupEntry` source path valid.

**Looking for a walkthrough?** See [Channel Plugins](https://docs.openclaw.ai/plugins/sdk-channel-plugins)
or [Provider Plugins](https://docs.openclaw.ai/plugins/sdk-provider-plugins) for step-by-step guides.

## [​](https://docs.openclaw.ai/plugins/sdk-entrypoints\\#definepluginentry)  `definePluginEntry`

**Import:**`openclaw/plugin-sdk/plugin-entry`For provider plugins, tool plugins, hook plugins, and anything that is **not**
a messaging channel.

```
import { definePluginEntry } from \"openclaw/plugin-sdk/plugin-entry\";

export default definePluginEntry({
  id: \"my-plugin\",
  name: \"My Plugin\",
  description: \"Short summary\",
  register(api) {
    api.registerProvider({
      /* ... */
    });
    api.registerTool({
      /* ... */
    });
  },
});
```

| Field | Type | Required | Default |
| --- | --- | --- | --- |
| `id` | `string` | Yes | — |
| `name` | `string` | Yes | — |
| `description` | `string` | Yes | — |
| `kind` | `string` | No | — |
| `configSchema` | `OpenClawPluginConfigSchema | () => OpenClawPluginConfigSchema` | No | Empty object schema |
| `register` | `(api: OpenClawPluginApi) => void` | Yes | — |

- `id` must match your `openclaw.plugin.json` manifest.
- `kind` is for exclusive slots: `\"memory\"` or `\"context-engine\"`.
- `configSchema` can be a function for lazy evaluation.
- OpenClaw resolves and memoizes that schema on first access, so expensive schema
builders only run once.

## [​](https://docs.openclaw.ai/plugins/sdk-entrypoints\\#definechannelpluginentry)  `defineChannelPluginEntry`

**Import:**`openclaw/plugin-sdk/channel-core`Wraps `definePluginEntry` with channel-specific wiring. Automatically calls
`api.registerChannel({ plugin })`, exposes an optional root-help CLI metadata
seam, and gates `registerFull` on registration mode.

```
import { defineChannelPluginEntry } from \"openclaw/plugin-sdk/channel-core\";

export default defineChannelPluginEntry({
  id: \"my-channel\",
  name: \"My Channel\",
  description: \"Short summary\",
  plugin: myChannelPlugin,
  setRuntime: setMyRuntime,
  registerCliMetadata(api) {
    api.registerCli(/* ... */);
  },
  registerFull(api) {
    api.registerGatewayMethod(/* ... */);
  },
});
```

| Field | Type | Required | Default |
| --- | --- | --- | --- |
| `id` | `string` | Yes | — |
| `name` | `string` | Yes | — |
| `description` | `string` | Yes | — |
| `plugin` | `ChannelPlugin` | Yes | — |
| `configSchema` | `OpenClawPluginConfigSchema | () => OpenClawPluginConfigSchema` | No | Empty object schema |
| `setRuntime` | `(runtime: PluginRuntime) => void` | No | — |
| `registerCliMetadata` | `(api: OpenClawPluginApi) => void` | No | — |
| `registerFull` | `(api: OpenClawPluginApi) => void` | No | — |

- `setRuntime` is called during registration so you can store the runtime reference
(typically via `createPluginRuntimeStore`). It is skipped during CLI metadata
capture.
- `registerCliMetadata` runs during `api.registrationMode === \"cli-metadata\"`,
`api.registrationMode === \"discovery\"`, and
`api.registrationMode === \"full\"`.
Use it as the canonical place for channel-owned CLI descriptors so root help
stays non-activating, discovery snapshots include static command metadata, and
normal CLI command registration remains compatible with full plugin loads.
- Discovery registration is non-activating, not import-free. OpenClaw may
evaluate the trusted plugin entry and channel plugin module to build the
snapshot, so keep top-level imports side-effect-free and put sockets,
clients, workers, and services behind `\"full\"`-only paths.
- `registerFull` only runs when `api.registrationMode === \"full\"`. It is skipped
during setup-only loading.
- Like `definePluginEntry`, `configSchema` can be a lazy factory and OpenClaw
memoizes the resolved schema on first access.
- For plugin-owned root CLI commands, prefer `api.registerCli(..., { descriptors: [...] })`
when you want the command to stay lazy-loaded without disappearing from the
root CLI parse tree. For channel plugins, prefer registering those descriptors
from `registerCliMetadata(...)` and keep `registerFull(...)` focused on runtime-only work.
- If `registerFull(...)` also registers gateway RPC methods, keep them on a
plugin-specific prefix. Reserved core admin namespaces (`config.*`,
`exec.approvals.*`, `wizard.*`, `update.*`) are always coerced to
`operator.admin`.

## [​](https://docs.openclaw.ai/plugins/sdk-entrypoints\\#definesetuppluginentry)  `defineSetupPluginEntry`

**Import:**`openclaw/plugin-sdk/channel-core`For the lightweight `setup-entry.ts` file. Returns just `{ plugin }` with no
runtime or CLI wiring.

```
import { defineSetupPluginEntry } from \"openclaw/plugin-sdk/channel-core\";

export default defineSetupPluginEntry(myChannelPlugin);
```

OpenClaw loads this instead of the full entry when a channel is disabled,
unconfigured, or when deferred loading is enabled. See
[Setup and Config](https://docs.openclaw.ai/plugins/sdk-setup#setup-entry) for when this matters.In practice, pair `defineSetupPluginEntry(...)` with the narrow setup helper
families:

- `openclaw/plugin-sdk/setup-runtime` for runtime-safe setup helpers such as
import-safe setup patch adapters, lookup-note output,
`promptResolvedAllowFrom`, `splitSetupEntries`, and delegated setup proxies
- `openclaw/plugin-sdk/channel-setup` for optional-install setup surfaces
- `openclaw/plugin-sdk/setup-tools` for setup/install CLI/archive/docs helpers

Keep heavy SDKs, CLI registration, and long-lived runtime services in the full
entry.Bundled workspace channels that split setup and runtime surfaces can use
`defineBundledChannelSetupEntry(...)` from
`openclaw/plugin-sdk/channel-entry-contract` instead. That contract lets the
setup entry keep setup-safe plugin/secrets exports while still exposing a
runtime setter:

```
import { defineBundledChannelSetupEntry } from \"openclaw/plugin-sdk/channel-entry-contract\";

export default defineBundledChannelSetupEntry({
  importMetaUrl: import.meta.url,
  plugin: {
    specifier: \"./channel-plugin-api.js\",
    exportName: \"myChannelPlugin\",
  },
  runtime: {
    specifier: \"./runtime-api.js\",
    exportName: \"setMyChannelRuntime\",
  },
});
```

Use that bundled contract only when setup flows truly need a lightweight runtime
setter before the full channel entry loads.

## [​](https://docs.openclaw.ai/plugins/sdk-entrypoints\\#registration-mode)  Registration mode

`api.registrationMode` tells your plugin how it was loaded:

| Mode | When | What to register |
| --- | --- | --- |
| `\"full\"` | Normal gateway startup | Everything |
| `\"discovery\"` | Read-only capability discovery | Channel registration plus static CLI descriptors; entry code may load, but skip sockets, workers, clients, and services |
| `\"setup-only\"` | Disabled/unconfigured channel | Channel registration only |
| `\"setup-runtime\"` | Setup flow with runtime available | Channel registration plus only the lightweight runtime needed before the full entry loads |
| `\"cli-metadata\"` | Root help / CLI metadata capture | CLI descriptors only |

`defineChannelPluginEntry` handles this split automatically. If you use
`definePluginEntry` directly for a channel, check mode yourself:

```
register(api) {
  if (
    api.registrationMode === \"cli-metadata\" ||
    api.registrationMode === \"discovery\" ||
    api.registrationMode === \"full\"
  ) {
    api.registerCli(/* ... */);
    if (api.registrationMode === \"cli-metadata\") return;
  }

  api.registerChannel({ plugin: myPlugin });
  if (api.registrationMode !== \"full\") return;

  // Heavy runtime-only registrations
  api.registerService(/* ... */);
}
```

Discovery mode builds a non-activating registry snapshot. It may still evaluate
the plugin entry and the channel plugin object so OpenClaw can register channel
capabilities and static CLI descriptors. Treat module evaluation in discovery as
trusted but lightweight: no network clients, subprocesses, listeners, database
connections, background workers, credential reads, or other live runtime side
effects at top level.Treat `\"setup-runtime\"` as the window where setup-only startup surfaces must
exist without re-entering the full bundled channel runtime. Good fits are
channel registration, setup-safe HTTP routes, setup-safe gateway methods, and
delegated setup helpers. Heavy background services, CLI registrars, and
provider/client SDK bootstraps still belong in `\"full\"`.For CLI registrars specifically:

- use `descriptors` when the registrar owns one or more root commands and you
want OpenClaw to lazy-load the real CLI module on first invocation
- make sure those descriptors cover every top-level command root exposed by the
registrar
- keep descriptor command names to letters, numbers, hyphen, and underscore,
starting with a letter or number; OpenClaw rejects descriptor names outside
that shape and strips terminal control sequences from descriptions before
rendering help
- use `commands` alone only for eager compatibility paths

## [​](https://docs.openclaw.ai/plugins/sdk-entrypoints\\#plugin-shapes)  Plugin shapes

OpenClaw classifies loaded plugins by their registration behavior:

| Shape | Description |
| --- | --- |
| **plain-capability** | One capability type (e.g. provider-only) |
| **hybrid-capability** | Multiple capability types (e.g. provider + speech) |
| **hook-only** | Only hooks, no capabilities |
| **non-capability** | Tools/commands/services but no capabilities |

Use `openclaw plugins inspect <id>` to see a plugin’s shape.

## [​](https://docs.openclaw.ai/plugins/sdk-entrypoints\\#related)  Related

- [SDK Overview](https://docs.openclaw.ai/plugins/sdk-overview) — registration API and subpath reference
- [Runtime Helpers](https://docs.openclaw.ai/plugins/sdk-runtime) — `api.runtime` and `createPluginRuntimeStore`
- [Setup and Config](https://docs.openclaw.ai/plugins/sdk-setup) — manifest, setup entry, deferred loading
- [Channel Plugins](https://docs.openclaw.ai/plugins/sdk-channel-plugins) — building the `ChannelPlugin` object
- [Provider Plugins](https://docs.openclaw.ai/plugins/sdk-provider-plugins) — provider registration and hooks

[Plugin SDK subpaths](https://docs.openclaw.ai/plugins/sdk-subpaths) [Runtime helpers](https://docs.openclaw.ai/plugins/sdk-runtime)

Ctrl+I

---

## Plugin compatibility - OpenClaw
**Source:** https://docs.openclaw.ai/plugins/compatibility

[Skip to main content](https://docs.openclaw.ai/plugins/compatibility#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Building plugins

Plugin compatibility

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Compatibility registry](https://docs.openclaw.ai/plugins/compatibility#compatibility-registry)
- [Plugin inspector package](https://docs.openclaw.ai/plugins/compatibility#plugin-inspector-package)
- [Deprecation policy](https://docs.openclaw.ai/plugins/compatibility#deprecation-policy)
- [Current compatibility areas](https://docs.openclaw.ai/plugins/compatibility#current-compatibility-areas)
- [Release notes](https://docs.openclaw.ai/plugins/compatibility#release-notes)

OpenClaw keeps older plugin contracts wired through named compatibility
adapters before removing them. This protects existing bundled and external
plugins while the SDK, manifest, setup, config, and agent runtime contracts
evolve.

## [​](https://docs.openclaw.ai/plugins/compatibility\\#compatibility-registry)  Compatibility registry

Plugin compatibility contracts are tracked in the core registry at
`src/plugins/compat/registry.ts`.Each record has:

- a stable compatibility code
- status: `active`, `deprecated`, `removal-pending`, or `removed`
- owner: SDK, config, setup, channel, provider, plugin execution, agent runtime,
or core
- introduction and deprecation dates when applicable
- replacement guidance
- docs, diagnostics, and tests that cover the old and new behavior

The registry is the source for maintainer planning and future plugin inspector
checks. If a plugin-facing behavior changes, add or update the compatibility
record in the same change that adds the adapter.Doctor repair and migration compatibility is tracked separately at
`src/commands/doctor/shared/deprecation-compat.ts`. Those records cover old
config shapes, install-ledger layouts, and repair shims that may need to stay
available after the runtime compatibility path is removed.Release sweeps should check both registries. Do not delete a doctor migration
just because the matching runtime or config compatibility record expired; first
verify there is no supported upgrade path that still needs the repair. Also
revalidate each replacement annotation during release planning because plugin
ownership and config footprint can change as providers and channels move out of
core.

## [​](https://docs.openclaw.ai/plugins/compatibility\\#plugin-inspector-package)  Plugin inspector package

The plugin inspector should live outside the core OpenClaw repo as a separate
package/repository backed by the versioned compatibility and manifest
contracts.The day-one CLI should be:

```
openclaw-plugin-inspector ./my-plugin
```

It should emit:

- manifest/schema validation
- the contract compatibility version being checked
- install/source metadata checks
- cold-path import checks
- deprecation and compatibility warnings

Use `--json` for stable machine-readable output in CI annotations. OpenClaw
core should expose contracts and fixtures the inspector can consume, but should
not publish the inspector binary from the main `openclaw` package.

## [​](https://docs.openclaw.ai/plugins/compatibility\\#deprecation-policy)  Deprecation policy

OpenClaw should not remove a documented plugin contract in the same release
that introduces its replacement.The migration sequence is:

1. Add the new contract.
2. Keep the old behavior wired through a named compatibility adapter.
3. Emit diagnostics or warnings when plugin authors can act.
4. Document the replacement and timeline.
5. Test both old and new paths.
6. Wait through the announced migration window.
7. Remove only with explicit breaking-release approval.

Deprecated records must include a warning start date, replacement, docs link,
and final removal date no more than three months after the warning starts. Do
not add a deprecated compatibility path with an open-ended removal window unless
maintainers explicitly decide it is permanent compatibility and mark it `active`
instead.

## [​](https://docs.openclaw.ai/plugins/compatibility\\#current-compatibility-areas)  Current compatibility areas

Current compatibility records include:

- legacy broad SDK imports such as `openclaw/plugin-sdk/compat`
- legacy hook-only plugin shapes and `before_agent_start`
- legacy `activate(api)` plugin entrypoints while plugins migrate to
`register(api)`
- legacy SDK aliases such as `openclaw/extension-api`,
`openclaw/plugin-sdk/channel-runtime`, `openclaw/plugin-sdk/command-auth`
status builders, `openclaw/plugin-sdk/test-utils`, and the `ClawdbotConfig` /
`OpenClawSchemaType` type aliases
- bundled plugin allowlist and enablement behavior
- legacy provider/channel env-var manifest metadata
- legacy provider plugin hooks and type aliases while providers move to
explicit catalog, auth, thinking, replay, and transport hooks
- legacy runtime aliases such as `api.runtime.taskFlow`,
`api.runtime.subagent.getSession`, and `api.runtime.stt`
- legacy memory-plugin split registration while memory plugins move to
`registerMemoryCapability`
- legacy channel SDK helpers for native message schemas, mention gating,
inbound envelope formatting, and approval capability nesting
- activation hints that are being replaced by manifest contribution ownership
- `setup-api` runtime fallback while setup descriptors move to cold
`setup.requiresRuntime: false` metadata
- provider `discovery` hooks while provider catalog hooks move to
`catalog.run(...)`
- channel `showConfigured` / `showInSetup` metadata while channel packages move
to `openclaw.channel.exposure`
- legacy runtime-policy config keys while doctor migrates operators to
`agentRuntime`
- generated bundled channel config metadata fallback while registry-first
`channelConfigs` metadata lands
- persisted plugin registry disable and install-migration env flags while
repair flows migrate operators to `openclaw plugins registry --refresh` and
`openclaw doctor --fix`
- legacy plugin-owned web search, web fetch, and x\\_search config paths while
doctor migrates them to `plugins.entries.<plugin>.config`
- legacy `plugins.installs` authored config and bundled plugin load-path
aliases while install metadata moves into the state-managed plugin ledger

New plugin code should prefer the replacement listed in the registry and in the
specific migration guide. Existing plugins can keep using a compatibility path
until the docs, diagnostics, and release notes announce a removal window.

## [​](https://docs.openclaw.ai/plugins/compatibility\\#release-notes)  Release notes

Release notes should include upcoming plugin deprecations with target dates and
links to migration docs. That warning needs to happen before a compatibility
path moves to `removal-pending` or `removed`.

[Provider plugins](https://docs.openclaw.ai/plugins/sdk-provider-plugins) [Migrate to SDK](https://docs.openclaw.ai/plugins/sdk-migration)

Ctrl+I

---

