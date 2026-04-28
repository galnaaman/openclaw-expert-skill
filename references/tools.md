# OpenClaw Tools Documentation

## SearXNG search - OpenClaw
**Source:** https://docs.openclaw.ai/tools/searxng-search

[Skip to main content](https://docs.openclaw.ai/tools/searxng-search#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web tools

SearXNG search

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Setup](https://docs.openclaw.ai/tools/searxng-search#setup)
- [Config](https://docs.openclaw.ai/tools/searxng-search#config)
- [Environment variable](https://docs.openclaw.ai/tools/searxng-search#environment-variable)
- [Plugin config reference](https://docs.openclaw.ai/tools/searxng-search#plugin-config-reference)
- [Notes](https://docs.openclaw.ai/tools/searxng-search#notes)
- [Related](https://docs.openclaw.ai/tools/searxng-search#related)

OpenClaw supports [SearXNG](https://docs.searxng.org/) as a **self-hosted,**
**key-free**`web_search` provider. SearXNG is an open-source meta-search engine
that aggregates results from Google, Bing, DuckDuckGo, and other sources.Advantages:

- **Free and unlimited** — no API key or commercial subscription required
- **Privacy / air-gap** — queries never leave your network
- **Works anywhere** — no region restrictions on commercial search APIs

## [​](https://docs.openclaw.ai/tools/searxng-search\\#setup)  Setup

1

[Navigate to header](https://docs.openclaw.ai/tools/searxng-search#)

Run a SearXNG instance

```
docker run -d -p 8888:8080 searxng/searxng
```

Or use any existing SearXNG deployment you have access to. See the
[SearXNG documentation](https://docs.searxng.org/) for production setup.

2

[Navigate to header](https://docs.openclaw.ai/tools/searxng-search#)

Configure

```
openclaw configure --section web
# Select \"searxng\" as the provider
```

Or set the env var and let auto-detection find it:

```
export SEARXNG_BASE_URL=\"http://localhost:8888\"
```

## [​](https://docs.openclaw.ai/tools/searxng-search\\#config)  Config

```
{
  tools: {
    web: {
      search: {
        provider: \"searxng\",
      },
    },
  },
}
```

Plugin-level settings for the SearXNG instance:

```
{
  plugins: {
    entries: {
      searxng: {
        config: {
          webSearch: {
            baseUrl: \"http://localhost:8888\",
            categories: \"general,news\", // optional
            language: \"en\", // optional
          },
        },
      },
    },
  },
}
```

The `baseUrl` field also accepts SecretRef objects.Transport rules:

- `https://` works for public or private SearXNG hosts
- `http://` is only accepted for trusted private-network or loopback hosts
- public SearXNG hosts must use `https://`

## [​](https://docs.openclaw.ai/tools/searxng-search\\#environment-variable)  Environment variable

Set `SEARXNG_BASE_URL` as an alternative to config:

```
export SEARXNG_BASE_URL=\"http://localhost:8888\"
```

When `SEARXNG_BASE_URL` is set and no explicit provider is configured, auto-detection
picks SearXNG automatically (at the lowest priority — any API-backed provider with a
key wins first).

## [​](https://docs.openclaw.ai/tools/searxng-search\\#plugin-config-reference)  Plugin config reference

| Field | Description |
| --- | --- |
| `baseUrl` | Base URL of your SearXNG instance (required) |
| `categories` | Comma-separated categories such as `general`, `news`, or `science` |
| `language` | Language code for results such as `en`, `de`, or `fr` |

## [​](https://docs.openclaw.ai/tools/searxng-search\\#notes)  Notes

- **JSON API** — uses SearXNG’s native `format=json` endpoint, not HTML scraping
- **No API key** — works with any SearXNG instance out of the box
- **Base URL validation** — `baseUrl` must be a valid `http://` or `https://`
URL; public hosts must use `https://`
- **Auto-detection order** — SearXNG is checked last (order 200) in
auto-detection. API-backed providers with configured keys run first, then
DuckDuckGo (order 100), then Ollama Web Search (order 110)
- **Self-hosted** — you control the instance, queries, and upstream search engines
- **Categories** default to `general` when not configured

For SearXNG JSON API to work, make sure your SearXNG instance has the `json`
format enabled in its `settings.yml` under `search.formats`.

## [​](https://docs.openclaw.ai/tools/searxng-search\\#related)  Related

- [Web Search overview](https://docs.openclaw.ai/tools/web) — all providers and auto-detection
- [DuckDuckGo Search](https://docs.openclaw.ai/tools/duckduckgo-search) — another key-free fallback
- [Brave Search](https://docs.openclaw.ai/tools/brave-search) — structured results with free tier

[Perplexity search](https://docs.openclaw.ai/tools/perplexity-search) [Tavily](https://docs.openclaw.ai/tools/tavily)

Ctrl+I

---

## Web fetch - OpenClaw
**Source:** https://docs.openclaw.ai/tools/web-fetch

[Skip to main content](https://docs.openclaw.ai/tools/web-fetch#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web tools

Web fetch

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/tools/web-fetch#quick-start)
- [Tool parameters](https://docs.openclaw.ai/tools/web-fetch#tool-parameters)
- [How it works](https://docs.openclaw.ai/tools/web-fetch#how-it-works)
- [Config](https://docs.openclaw.ai/tools/web-fetch#config)
- [Firecrawl fallback](https://docs.openclaw.ai/tools/web-fetch#firecrawl-fallback)
- [Limits and safety](https://docs.openclaw.ai/tools/web-fetch#limits-and-safety)
- [Tool profiles](https://docs.openclaw.ai/tools/web-fetch#tool-profiles)
- [Related](https://docs.openclaw.ai/tools/web-fetch#related)

The `web_fetch` tool does a plain HTTP GET and extracts readable content
(HTML to markdown or text). It does **not** execute JavaScript.For JS-heavy sites or login-protected pages, use the
[Web Browser](https://docs.openclaw.ai/tools/browser) instead.

## [​](https://docs.openclaw.ai/tools/web-fetch\\#quick-start)  Quick start

`web_fetch` is **enabled by default** — no configuration needed. The agent can
call it immediately:

```
await web_fetch({ url: \"https://example.com/article\" });
```

## [​](https://docs.openclaw.ai/tools/web-fetch\\#tool-parameters)  Tool parameters

[​](https://docs.openclaw.ai/tools/web-fetch#param-url)

url

string

required

URL to fetch. `http(s)` only.

[​](https://docs.openclaw.ai/tools/web-fetch#param-extract-mode)

extractMode

'markdown' \\| 'text'

default:\"markdown\"

Output format after main-content extraction.

[​](https://docs.openclaw.ai/tools/web-fetch#param-max-chars)

maxChars

number

Truncate output to this many characters.

## [​](https://docs.openclaw.ai/tools/web-fetch\\#how-it-works)  How it works

1

[Navigate to header](https://docs.openclaw.ai/tools/web-fetch#)

Fetch

Sends an HTTP GET with a Chrome-like User-Agent and `Accept-Language`
header. Blocks private/internal hostnames and re-checks redirects.

2

[Navigate to header](https://docs.openclaw.ai/tools/web-fetch#)

Extract

Runs Readability (main-content extraction) on the HTML response.

3

[Navigate to header](https://docs.openclaw.ai/tools/web-fetch#)

Fallback (optional)

If Readability fails and Firecrawl is configured, retries through the
Firecrawl API with bot-circumvention mode.

4

[Navigate to header](https://docs.openclaw.ai/tools/web-fetch#)

Cache

Results are cached for 15 minutes (configurable) to reduce repeated
fetches of the same URL.

## [​](https://docs.openclaw.ai/tools/web-fetch\\#config)  Config

```
{
  tools: {
    web: {
      fetch: {
        enabled: true, // default: true
        provider: \"firecrawl\", // optional; omit for auto-detect
        maxChars: 50000, // max output chars
        maxCharsCap: 50000, // hard cap for maxChars param
        maxResponseBytes: 2000000, // max download size before truncation
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        readability: true, // use Readability extraction
        userAgent: \"Mozilla/5.0 ...\", // override User-Agent
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/tools/web-fetch\\#firecrawl-fallback)  Firecrawl fallback

If Readability extraction fails, `web_fetch` can fall back to
[Firecrawl](https://docs.openclaw.ai/tools/firecrawl) for bot-circumvention and better extraction:

```
{
  tools: {
    web: {
      fetch: {
        provider: \"firecrawl\", // optional; omit for auto-detect from available credentials
      },
    },
  },
  plugins: {
    entries: {
      firecrawl: {
        enabled: true,
        config: {
          webFetch: {
            apiKey: \"fc-...\", // optional if FIRECRAWL_API_KEY is set
            baseUrl: \"https://api.firecrawl.dev\",
            onlyMainContent: true,
            maxAgeMs: 86400000, // cache duration (1 day)
            timeoutSeconds: 60,
          },
        },
      },
    },
  },
}
```

`plugins.entries.firecrawl.config.webFetch.apiKey` supports SecretRef objects.
Legacy `tools.web.fetch.firecrawl.*` config is auto-migrated by `openclaw doctor --fix`.

If Firecrawl is enabled and its SecretRef is unresolved with no
`FIRECRAWL_API_KEY` env fallback, gateway startup fails fast.

Firecrawl `baseUrl` overrides are locked down: they must use `https://` and
the official Firecrawl host (`api.firecrawl.dev`).

Current runtime behavior:

- `tools.web.fetch.provider` selects the fetch fallback provider explicitly.
- If `provider` is omitted, OpenClaw auto-detects the first ready web-fetch
provider from available credentials. Today the bundled provider is Firecrawl.
- If Readability is disabled, `web_fetch` skips straight to the selected
provider fallback. If no provider is available, it fails closed.

## [​](https://docs.openclaw.ai/tools/web-fetch\\#limits-and-safety)  Limits and safety

- `maxChars` is clamped to `tools.web.fetch.maxCharsCap`
- Response body is capped at `maxResponseBytes` before parsing; oversized
responses are truncated with a warning
- Private/internal hostnames are blocked
- Redirects are checked and limited by `maxRedirects`
- `web_fetch` is best-effort — some sites need the [Web Browser](https://docs.openclaw.ai/tools/browser)

## [​](https://docs.openclaw.ai/tools/web-fetch\\#tool-profiles)  Tool profiles

If you use tool profiles or allowlists, add `web_fetch` or `group:web`:

```
{
  tools: {
    allow: [\"web_fetch\"],
    // or: allow: [\"group:web\"]  (includes web_fetch, web_search, and x_search)
  },
}
```

## [​](https://docs.openclaw.ai/tools/web-fetch\\#related)  Related

- [Web Search](https://docs.openclaw.ai/tools/web) — search the web with multiple providers
- [Web Browser](https://docs.openclaw.ai/tools/browser) — full browser automation for JS-heavy sites
- [Firecrawl](https://docs.openclaw.ai/tools/firecrawl) — Firecrawl search and scrape tools

[WSL2 + Windows + remote Chrome CDP troubleshooting](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting) [Web Search](https://docs.openclaw.ai/tools/web)

Ctrl+I

---

## Browser login - OpenClaw
**Source:** https://docs.openclaw.ai/tools/browser-login

[Skip to main content](https://docs.openclaw.ai/tools/browser-login#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web browser

Browser login

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Browser login + X/Twitter posting](https://docs.openclaw.ai/tools/browser-login#browser-login-%2B-x%2Ftwitter-posting)
- [Manual login (recommended)](https://docs.openclaw.ai/tools/browser-login#manual-login-recommended)
- [Which Chrome profile is used?](https://docs.openclaw.ai/tools/browser-login#which-chrome-profile-is-used)
- [X/Twitter: recommended flow](https://docs.openclaw.ai/tools/browser-login#x%2Ftwitter-recommended-flow)
- [Sandboxing + host browser access](https://docs.openclaw.ai/tools/browser-login#sandboxing-%2B-host-browser-access)
- [Related](https://docs.openclaw.ai/tools/browser-login#related)

# [​](https://docs.openclaw.ai/tools/browser-login\\#browser-login-+-x/twitter-posting)  Browser login + X/Twitter posting

## [​](https://docs.openclaw.ai/tools/browser-login\\#manual-login-recommended)  Manual login (recommended)

When a site requires login, **sign in manually** in the **host** browser profile (the openclaw browser).Do **not** give the model your credentials. Automated logins often trigger anti‑bot defenses and can lock the account.Back to the main browser docs: [Browser](https://docs.openclaw.ai/tools/browser).

## [​](https://docs.openclaw.ai/tools/browser-login\\#which-chrome-profile-is-used)  Which Chrome profile is used?

OpenClaw controls a **dedicated Chrome profile** (named `openclaw`, orange‑tinted UI). This is separate from your daily browser profile.For agent browser tool calls:

- Default choice: the agent should use its isolated `openclaw` browser.
- Use `profile=\"user\"` only when existing logged-in sessions matter and the user is at the computer to click/approve any attach prompt.
- If you have multiple user-browser profiles, specify the profile explicitly instead of guessing.

Two easy ways to access it:

1. **Ask the agent to open the browser** and then log in yourself.
2. **Open it via CLI**:

```
openclaw browser start
openclaw browser open https://x.com
```

If you have multiple profiles, pass `--browser-profile <name>` (the default is `openclaw`).

## [​](https://docs.openclaw.ai/tools/browser-login\\#x/twitter-recommended-flow)  X/Twitter: recommended flow

- **Read/search/threads:** use the **host** browser (manual login).
- **Post updates:** use the **host** browser (manual login).

## [​](https://docs.openclaw.ai/tools/browser-login\\#sandboxing-+-host-browser-access)  Sandboxing + host browser access

Sandboxed browser sessions are **more likely** to trigger bot detection. For X/Twitter (and other strict sites), prefer the **host** browser.If the agent is sandboxed, the browser tool defaults to the sandbox. To allow host control:

```
{
  agents: {
    defaults: {
      sandbox: {
        mode: \"non-main\",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Then target the host browser:

```
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Or disable sandboxing for the agent that posts updates.

## [​](https://docs.openclaw.ai/tools/browser-login\\#related)  Related

- [Browser](https://docs.openclaw.ai/tools/browser)
- [Browser Linux troubleshooting](https://docs.openclaw.ai/tools/browser-linux-troubleshooting)
- [Browser WSL2 troubleshooting](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting)

[Browser control API](https://docs.openclaw.ai/tools/browser-control) [Browser troubleshooting](https://docs.openclaw.ai/tools/browser-linux-troubleshooting)

Ctrl+I

---

## Reactions - OpenClaw
**Source:** https://docs.openclaw.ai/tools/reactions

## [​](https://docs.openclaw.ai/tools/reactions\\#reactions) Reactions

Reactions are a way for agents to respond to messages with emojis. They can be used to acknowledge a message, express emotion, or provide a quick response without sending a full message.

## [​](https://docs.openclaw.ai/tools/reactions\\#reaction-policy) Reaction policy

Per-channel `reactionPolicy` config controls which messages the agent reacts to. Values are typically `off`, `own`, or `all`.

- [Telegram reactionPolicy](https://docs.openclaw.ai/channels/telegram#reaction-notifications) — `channels.telegram.reactionPolicy`
- [WhatsApp reactionPolicy](https://docs.openclaw.ai/channels/whatsapp#reaction-policy) — `channels.whatsapp.reactionPolicy`

Set `reactionPolicy` on individual channels to tune how broadly the agent reacts to messages on each platform. `"off"` disables them, `"own"` (default) emits events when users react to bot messages, and `"all"` emits events for all reactions.

## [​](https://docs.openclaw.ai/tools/reactions\\#reaction-level) Reaction level

Per-channel `reactionLevel` config controls how broadly the agent uses reactions. Values are typically `off`, `ack`, `minimal`, or `extensive`.

- [Telegram reactionLevel](https://docs.openclaw.ai/channels/telegram#reaction-notifications) — `channels.telegram.reactionLevel`
- [WhatsApp reactionLevel](https://docs.openclaw.ai/channels/whatsapp#reaction-level) — `channels.whatsapp.reactionLevel`

Set `reactionLevel` on individual channels to tune how actively the agent reacts to messages on each platform.

## [​](https://docs.openclaw.ai/tools/reactions\\#related) Related

- [Agent Send](https://docs.openclaw.ai/tools/agent-send) — the `message` tool that includes `react`
- [Channels](https://docs.openclaw.ai/channels) — channel-specific configuration

[PDF tool](https://docs.openclaw.ai/tools/pdf) [Thinking levels](https://docs.openclaw.ai/tools/thinking)

Ctrl+I

---

## Trajectory bundles - OpenClaw
**Source:** https://docs.openclaw.ai/tools/trajectory

[Skip to main content](https://docs.openclaw.ai/tools/trajectory#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

Trajectory bundles

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/tools/trajectory#quick-start)
- [Access](https://docs.openclaw.ai/tools/trajectory#access)
- [What gets recorded](https://docs.openclaw.ai/tools/trajectory#what-gets-recorded)
- [Bundle files](https://docs.openclaw.ai/tools/trajectory#bundle-files)
- [Capture location](https://docs.openclaw.ai/tools/trajectory#capture-location)
- [Disable capture](https://docs.openclaw.ai/tools/trajectory#disable-capture)
- [Privacy and limits](https://docs.openclaw.ai/tools/trajectory#privacy-and-limits)
- [Troubleshooting](https://docs.openclaw.ai/tools/trajectory#troubleshooting)
- [Related](https://docs.openclaw.ai/tools/trajectory#related)

Trajectory capture is OpenClaw’s per-session flight recorder. It records a
structured timeline for each agent run, then `/export-trajectory` packages the
current session into a redacted support bundle.Use it when you need to answer questions like:

- What prompt, system prompt, and tools were sent to the model?
- Which transcript messages and tool calls led to this answer?
- Did the run time out, abort, compact, or hit a provider error?
- Which model, plugins, skills, and runtime settings were active?
- What usage and prompt-cache metadata did the provider return?

## [​](https://docs.openclaw.ai/tools/trajectory\\#quick-start)  Quick start

Send this in the active session:

```
/export-trajectory
```

Alias:

```
/trajectory
```

OpenClaw writes the bundle under the workspace:

```
.openclaw/trajectory-exports/openclaw-trajectory-<session>-<timestamp>/
```

You can choose a relative output directory name:

```
/export-trajectory bug-1234
```

The custom path is resolved inside `.openclaw/trajectory-exports/`. Absolute
paths and `~` paths are rejected.

## [​](https://docs.openclaw.ai/tools/trajectory\\#access)  Access

Trajectory export is an owner command. The sender must pass the normal command
authorization checks and owner checks for the channel.

## [​](https://docs.openclaw.ai/tools/trajectory\\#what-gets-recorded)  What gets recorded

Trajectory capture is on by default for OpenClaw agent runs.Runtime events include:

- `session.started`
- `trace.metadata`
- `context.compiled`
- `prompt.submitted`
- `model.completed`
- `trace.artifacts`
- `session.ended`

Transcript events are also reconstructed from the active session branch:

- user messages
- assistant messages
- tool calls
- tool results
- compactions
- model changes
- labels and custom session entries

Events are written as JSON Lines with this schema marker:

```
{
  \"traceSchema\": \"openclaw-trajectory\",
  \"schemaVersion\": 1
}
```

## [​](https://docs.openclaw.ai/tools/trajectory\\#bundle-files)  Bundle files

An exported bundle can contain:

| File | Contents |
| --- | --- |
| `manifest.json` | Bundle schema, source files, event counts, and generated file list |
| `events.jsonl` | Ordered runtime and transcript timeline |
| `session-branch.json` | Redacted active transcript branch and session header |
| `metadata.json` | OpenClaw version, OS/runtime, model, config snapshot, plugins, skills, and prompt metadata |
| `artifacts.json` | Final status, errors, usage, prompt cache, compaction count, assistant text, and tool metadata |
| `prompts.json` | Submitted prompts and selected prompt-building details |
| `system-prompt.txt` | Latest compiled system prompt, when captured |
| `tools.json` | Tool definitions sent to the model, when captured |

`manifest.json` lists the files present in that bundle. Some files are omitted
when the session did not capture the corresponding runtime data.

## [​](https://docs.openclaw.ai/tools/trajectory\\#capture-location)  Capture location

By default, runtime trajectory events are written beside the session file:

```
<session>.trajectory.jsonl
```

OpenClaw also writes a best-effort pointer file beside the session:

```
<session>.trajectory-path.json
```

Set `OPENCLAW_TRAJECTORY_DIR` to store runtime trajectory sidecars in a
dedicated directory:

```
export OPENCLAW_TRAJECTORY_DIR=/var/lib/openclaw/trajectories
```

When this variable is set, OpenClaw writes one JSONL file per session id in that
directory.

## [​](https://docs.openclaw.ai/tools/trajectory\\#disable-capture)  Disable capture

Set `OPENCLAW_TRAJECTORY=0` before starting OpenClaw:

```
export OPENCLAW_TRAJECTORY=0
```

This disables runtime trajectory capture. `/export-trajectory` can still export
the transcript branch, but runtime-only files such as compiled context,
provider artifacts, and prompt metadata may be missing.

## [​](https://docs.openclaw.ai/tools/trajectory\\#privacy-and-limits)  Privacy and limits

Trajectory bundles are designed for support and debugging, not public posting.
OpenClaw redacts sensitive values before writing export files:

- credentials and known secret-like payload fields
- image data
- local state paths
- workspace paths, replaced with `$WORKSPACE_DIR`
- home directory paths, where detected

The exporter also bounds input size:

- runtime sidecar files: 50 MiB
- session files: 50 MiB
- runtime events: 200,000
- total exported events: 250,000
- individual runtime event lines are truncated above 256 KiB

Review bundles before sharing them outside your team. Redaction is best-effort
and cannot know every application-specific secret.

## [​](https://docs.openclaw.ai/tools/trajectory\\#troubleshooting)  Troubleshooting

If the export has no runtime events:

- confirm OpenClaw was started without `OPENCLAW_TRAJECTORY=0`
- check whether `OPENCLAW_TRAJECTORY_DIR` points to a writable directory
- run another message in the session, then export again
- inspect `manifest.json` for `runtimeEventCount`

If the command rejects the output path:

- use a relative name like `bug-1234`
- do not pass `/tmp/...` or `~/...`
- keep the export inside `.openclaw/trajectory-exports/`

If the export fails with a size error, the session or sidecar exceeded the
export safety limits. Start a new session or export a smaller reproduction.

## [​](https://docs.openclaw.ai/tools/trajectory\\#related)  Related

- [Diffs](https://docs.openclaw.ai/tools/diffs)
- [Session management](https://docs.openclaw.ai/concepts/session)
- [Exec tool](https://docs.openclaw.ai/tools/exec)

[Tool-loop detection](https://docs.openclaw.ai/tools/loop-detection) [Text to speech (TTS)](https://docs.openclaw.ai/tools/tts)

Ctrl+I

---

## Music generation - OpenClaw
**Source:** https://docs.openclaw.ai/tools/music-generation

[Skip to main content](https://docs.openclaw.ai/tools/music-generation#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

Music generation

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/tools/music-generation#quick-start)
- [Supported providers](https://docs.openclaw.ai/tools/music-generation#supported-providers)
- [Capability matrix](https://docs.openclaw.ai/tools/music-generation#capability-matrix)
- [Tool parameters](https://docs.openclaw.ai/tools/music-generation#tool-parameters)
- [Async behavior](https://docs.openclaw.ai/tools/music-generation#async-behavior)
- [Task lifecycle](https://docs.openclaw.ai/tools/music-generation#task-lifecycle)
- [Configuration](https://docs.openclaw.ai/tools/music-generation#configuration)
- [Model selection](https://docs.openclaw.ai/tools/music-generation#model-selection)
- [Provider selection order](https://docs.openclaw.ai/tools/music-generation#provider-selection-order)
- [Provider notes](https://docs.openclaw.ai/tools/music-generation#provider-notes)
- [Choosing the right path](https://docs.openclaw.ai/tools/music-generation#choosing-the-right-path)
- [Provider capability modes](https://docs.openclaw.ai/tools/music-generation#provider-capability-modes)
- [Live tests](https://docs.openclaw.ai/tools/music-generation#live-tests)
- [Related](https://docs.openclaw.ai/tools/music-generation#related)

The `music_generate` tool lets the agent create music or audio through the
shared music-generation capability with configured providers — Google,
MiniMax, and workflow-configured ComfyUI today.For session-backed agent runs, OpenClaw starts music generation as a
background task, tracks it in the task ledger, then wakes the agent again
when the track is ready so the agent can post the finished audio back into
the original channel.

The built-in shared tool only appears when at least one music-generation
provider is available. If you do not see `music_generate` in your agent’s
tools, configure `agents.defaults.musicGenerationModel` or set up a
provider API key.

## [​](https://docs.openclaw.ai/tools/music-generation\\#quick-start)  Quick start

- Shared provider-backed

- ComfyUI workflow


1

[Navigate to header](https://docs.openclaw.ai/tools/music-generation#)

Configure auth

Set an API key for at least one provider — for example
`GEMINI_API_KEY` or `MINIMAX_API_KEY`.

2

[Navigate to header](https://docs.openclaw.ai/tools/music-generation#)

Pick a default model (optional)

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

3

[Navigate to header](https://docs.openclaw.ai/tools/music-generation#)

Ask the agent

_“Generate an upbeat synthpop track about a night drive through a_
_neon city.”_The agent calls `music_generate` automatically. No tool
allow-listing needed.

For direct synchronous contexts without a session-backed agent run,
the built-in tool still falls back to inline generation and returns
the final media path in the tool result.

1

[Navigate to header](https://docs.openclaw.ai/tools/music-generation#)

Configure the workflow

Configure `plugins.entries.comfy.config.music` with a workflow
JSON and prompt/output nodes.

2

[Navigate to header](https://docs.openclaw.ai/tools/music-generation#)

Cloud auth (optional)

For Comfy Cloud, set `COMFY_API_KEY` or `COMFY_CLOUD_API_KEY`.

3

[Navigate to header](https://docs.openclaw.ai/tools/music-generation#)

Call the tool

```
/tool music_generate prompt=\"Warm ambient synth loop with soft tape texture\"
```

Example prompts:

```
Generate a cinematic piano track with soft strings and no vocals.
```

```
Generate an energetic chiptune loop about launching a rocket at sunrise.
```

## [​](https://docs.openclaw.ai/tools/music-generation\\#supported-providers)  Supported providers

| Provider | Default model | Reference inputs | Supported controls | Auth |
| --- | --- | --- | --- | --- |
| ComfyUI | `workflow` | Up to 1 image | Workflow-defined music or audio | `COMFY_API_KEY`, `COMFY_CLOUD_API_KEY` |
| Google | `lyria-3-clip-preview` | Up to 10 images | `lyrics`, `instrumental`, `format` | `GEMINI_API_KEY`, `GOOGLE_API_KEY` |
| MiniMax | `music-2.6` | None | `lyrics`, `instrumental`, `durationSeconds`, `format=mp3` | `MINIMAX_API_KEY` or MiniMax OAuth |

### [​](https://docs.openclaw.ai/tools/music-generation\\#capability-matrix)  Capability matrix

The explicit mode contract used by `music_generate`, contract tests, and the
shared live sweep:

| Provider | `generate` | `edit` | Edit limit | Shared live lanes |
| --- | --- | --- | --- | --- |
| ComfyUI | ✓ | ✓ | 1 image | Not in the shared sweep; covered by `extensions/comfy/comfy.live.test.ts` |
| Google | ✓ | ✓ | 10 images | `generate`, `edit` |
| MiniMax | ✓ | — | None | `generate` |

Use `action: \"list\"` to inspect available shared providers and models at
runtime:

```
/tool music_generate action=list
```

Use `action: \"status\"` to inspect the active session-backed music task:

```
/tool music_generate action=status
```

Direct generation example:

```
/tool music_generate prompt=\"Dreamy lo-fi hip hop with vinyl texture and gentle rain\" instrumental=true
```

## [​](https://docs.openclaw.ai/tools/music-generation\\#tool-parameters)  Tool parameters

[​](https://docs.openclaw.ai/tools/music-generation#param-prompt)

prompt

string

required

Music generation prompt. Required for `action: \"generate\"`.

[​](https://docs.openclaw.ai/tools/music-generation#param-action)

action

\"generate\" \\| \"status\" \\| \"list\"

default:\"generate\"

`\"status\"` returns the current session task; `\"list\"` inspects providers.

[​](https://docs.openclaw.ai/tools/music-generation#param-model)

model

string

Provider/model override (e.g. `google/lyria-3-pro-preview`,
`comfy/workflow`).

[​](https://docs.openclaw.ai/tools/music-generation#param-lyrics)

lyrics

string

Optional lyrics when the provider supports explicit lyric input.

[​](https://docs.openclaw.ai/tools/music-generation#param-instrumental)

instrumental

boolean

Request instrumental-only output when the provider supports it.

[​](https://docs.openclaw.ai/tools/music-generation#param-image)

image

string

Single reference image path or URL.

[​](https://docs.openclaw.ai/tools/music-generation#param-images)

images

string\[\]

Multiple reference images (up to 10 on supporting providers).

[​](https://docs.openclaw.ai/tools/music-generation#param-duration-seconds)

durationSeconds

number

Target duration in seconds when the provider supports duration hints.

[​](https://docs.openclaw.ai/tools/music-generation#param-format)

format

\"mp3\" \\| \"wav\"

Output format hint when the provider supports it.

[​](https://docs.openclaw.ai/tools/music-generation#param-filename)

filename

string

Output filename hint.

[​](https://docs.openclaw.ai/tools/music-generation#param-timeout-ms)

timeoutMs

number

Optional provider request timeout in milliseconds.

Not all providers support all parameters. OpenClaw still validates hard
limits such as input counts before submission. When a provider supports
duration but uses a shorter maximum than the requested value, OpenClaw
clamps to the closest supported duration. Truly unsupported optional hints
are ignored with a warning when the selected provider or model cannot honor
them. Tool results report applied settings; `details.normalization`
captures any requested-to-applied mapping.

## [​](https://docs.openclaw.ai/tools/music-generation\\#async-behavior)  Async behavior

Session-backed music generation runs as a background task:

- **Background task:**`music_generate` creates a background task, returns a
started/task response immediately, and posts the finished track later in
a follow-up agent message.
- **Duplicate prevention:** while a task is `queued` or `running`, later
`music_generate` calls in the same session return task status instead of
starting another generation. Use `action: \"status\"` to check explicitly.
- **Status lookup:**`openclaw tasks list` or `openclaw tasks show <taskId>`
inspects queued, running, and terminal status.
- **Completion wake:** OpenClaw injects an internal completion event back
into the same session so the model can write the user-facing follow-up
itself.
- **Prompt hint:** later user/manual turns in the same session get a small
runtime hint when a music task is already in flight, so the model can
not blindly call `music_generate` again.
- **No-session fallback:** direct/local contexts without a real agent
session run inline and return the final audio result in the same turn.

### [​](https://docs.openclaw.ai/tools/music-generation\\#task-lifecycle)  Task lifecycle

| State | Meaning |
| --- | --- |
| `queued` | Task created, waiting for the provider to accept it. |
| `running` | Provider is processing (typically 30 seconds to 3 minutes depending on provider and duration). |
| `succeeded` | Track ready; the agent wakes and posts it to the conversation. |
| `failed` | Provider error or timeout; the agent wakes with error details. |

Check status from the CLI:

```
openclaw tasks list
openclaw tasks show <taskId>
openclaw tasks cancel <taskId>
```

## [​](https://docs.openclaw.ai/tools/music-generation\\#configuration)  Configuration

### [​](https://docs.openclaw.ai/tools/music-generation\\#model-selection)  Model selection

```
{
  agents: {
    defaults: {
      musicGenerationModel: {
        primary: \"google/lyria-3-clip-preview\",
        fallbacks: [\"minimax/music-2.6\"],
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/tools/music-generation\\#provider-selection-order)  Provider selection order

OpenClaw tries providers in this order:

1. `model` parameter from the tool call (if the agent specifies one).
2. `musicGenerationModel.primary` from config.
3. `musicGenerationModel.fallbacks` in order.
4. Auto-detection using auth-backed provider defaults only:
   - current default provider first;
   - remaining registered music-generation providers in provider-id order.

If a provider fails, the next candidate is tried automatically. If all
fail, the error includes details from each attempt.Set `agents.defaults.mediaGenerationAutoProviderFallback: false` to use only
explicit `model`, `primary`, and `fallbacks` entries.

## [​](https://docs.openclaw.ai/tools/music-generation\\#provider-notes)  Provider notes

ComfyUI

Workflow-driven and depends on the configured graph plus node mapping
for prompt/output fields. The bundled `comfy` plugin plugs into the
shared `music_generate` tool through the music-generation provider
registry.

Google (Lyria 3)

Uses Lyria 3 batch generation. The current bundled flow supports
prompt, optional lyrics text, and optional reference images.

MiniMax

Uses the batch `music_generation` endpoint. Supports prompt, optional
lyrics, instrumental mode, duration steering, and mp3 output through
either `minimax` API-key auth or `minimax-portal` OAuth.

## [​](https://docs.openclaw.ai/tools/music-generation\\#choosing-the-right-path)  Choosing the right path

- **Shared provider-backed** when you want model selection, provider
failover, and the built-in async task/status flow.
- **Plugin path (ComfyUI)** when you need a custom workflow graph or a
provider that is not part of the shared bundled music capability.

If you are debugging ComfyUI-specific behavior, see
[ComfyUI](https://docs.openclaw.ai/providers/comfy). If you are debugging shared provider
behavior, start with [Google (Gemini)](https://docs.openclaw.ai/providers/google) or
[MiniMax](https://docs.openclaw.ai/providers/minimax).

## [​](https://docs.openclaw.ai/tools/music-generation\\#provider-capability-modes)  Provider capability modes

The shared music-generation contract supports explicit mode declarations:

- `generate` for prompt-only generation.
- `edit` when the request includes one or more reference images.

New provider implementations should prefer explicit mode blocks:

```
capabilities: {
  generate: {
    maxTracks: 1,
    supportsLyrics: true,
    supportsFormat: true,
  },
  edit: {
    enabled: true,
    maxTracks: 1,
    maxInputImages: 1,
    supportsFormat: true,
  },
}
```

Legacy flat fields such as `maxInputImages`, `supportsLyrics`, and
`supportsFormat` are **not** enough to advertise edit support. Providers
should declare `generate` and `edit` explicitly so live tests, contract
tests, and the shared `music_generate` tool can validate mode support
deterministically.

## [​](https://docs.openclaw.ai/tools/music-generation\\#live-tests)  Live tests

Opt-in live coverage for the shared bundled providers:

```
OPENCLAW_LIVE_TEST=1 pnpm test:live -- extensions/music-generation-providers.live.test.ts
```

Repo wrapper:

```
pnpm test:live:media music
```

This live file loads missing provider env vars from `~/.profile`, prefers
live/env API keys ahead of stored auth profiles by default, and runs both
`generate` and declared `edit` coverage when the provider enables edit
mode. Coverage today:

- `google`: `generate` plus `edit`
- `minimax`: `generate` only
- `comfy`: separate Comfy live coverage, not the shared provider sweep

Opt-in live coverage for the bundled ComfyUI music path:

```
OPENCLAW_LIVE_TEST=1 COMFY_LIVE_TEST=1 pnpm test:live -- extensions/comfy/comfy.live.test.ts
```

The Comfy live file also covers comfy image and video workflows when those
sections are configured.

## [​](https://docs.openclaw.ai/tools/music-generation\\#related)  Related

- [Background tasks](https://docs.openclaw.ai/automation/tasks) — task tracking for detached `music_generate` runs
- [ComfyUI](https://docs.openclaw.ai/providers/comfy)
- [Configuration reference](https://docs.openclaw.ai/gateway/config-agents#agent-defaults) — `musicGenerationModel` config
- [Google (Gemini)](https://docs.openclaw.ai/providers/google)
- [MiniMax](https://docs.openclaw.ai/providers/minimax)
- [Models](https://docs.openclaw.ai/concepts/models) — model configuration and failover
- [Tools overview](https://docs.openclaw.ai/tools)

[Media overview](https://docs.openclaw.ai/tools/media-overview) [PDF tool](https://docs.openclaw.ai/tools/pdf)

Ctrl+I

---

## WSL2 + Windows + remote Chrome CDP troubleshooting - OpenClaw
**Source:** https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting

[Skip to main content](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web browser

WSL2 + Windows + remote Chrome CDP troubleshooting

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Choose the right browser mode first](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#choose-the-right-browser-mode-first)
- [Option 1: Raw remote CDP from WSL2 to Windows](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#option-1-raw-remote-cdp-from-wsl2-to-windows)
- [Option 2: Host-local Chrome MCP](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#option-2-host-local-chrome-mcp)
- [Working architecture](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#working-architecture)
- [Why this setup is confusing](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#why-this-setup-is-confusing)
- [Critical rule for the Control UI](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#critical-rule-for-the-control-ui)
- [Validate in layers](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#validate-in-layers)
- [Layer 1: Verify Chrome is serving CDP on Windows](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#layer-1-verify-chrome-is-serving-cdp-on-windows)
- [Layer 2: Verify WSL2 can reach that Windows endpoint](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#layer-2-verify-wsl2-can-reach-that-windows-endpoint)
- [Layer 3: Configure the correct browser profile](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#layer-3-configure-the-correct-browser-profile)
- [Layer 4: Verify the Control UI layer separately](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#layer-4-verify-the-control-ui-layer-separately)
- [Layer 5: Verify end-to-end browser control](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#layer-5-verify-end-to-end-browser-control)
- [Common misleading errors](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#common-misleading-errors)
- [Fast triage checklist](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#fast-triage-checklist)
- [Practical takeaway](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#practical-takeaway)
- [Related](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting#related)

In the common split-host setup, OpenClaw Gateway runs inside WSL2, Chrome runs on Windows, and browser control must cross the WSL2 and Windows boundary. The layered failure pattern from [issue #39369](https://github.com/openclaw/openclaw/issues/39369) means several independent problems can show up at once, which makes the wrong layer look broken first.

## [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#choose-the-right-browser-mode-first)  Choose the right browser mode first

You have two valid patterns:

### [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#option-1-raw-remote-cdp-from-wsl2-to-windows)  Option 1: Raw remote CDP from WSL2 to Windows

Use a remote browser profile that points from WSL2 to a Windows Chrome CDP endpoint.Choose this when:

- the Gateway stays inside WSL2
- Chrome runs on Windows
- you need browser control to cross the WSL2/Windows boundary

### [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#option-2-host-local-chrome-mcp)  Option 2: Host-local Chrome MCP

Use `existing-session` / `user` only when the Gateway itself runs on the same host as Chrome.Choose this when:

- OpenClaw and Chrome are on the same machine
- you want the local signed-in browser state
- you do not need cross-host browser transport
- you do not need advanced managed/raw-CDP-only routes like `responsebody`, PDF
export, download interception, or batch actions

For WSL2 Gateway + Windows Chrome, prefer raw remote CDP. Chrome MCP is host-local, not a WSL2-to-Windows bridge.

## [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#working-architecture)  Working architecture

Reference shape:

- WSL2 runs the Gateway on `127.0.0.1:18789`
- Windows opens the Control UI in a normal browser at `http://127.0.0.1:18789/`
- Windows Chrome exposes a CDP endpoint on port `9222`
- WSL2 can reach that Windows CDP endpoint
- OpenClaw points a browser profile at the address that is reachable from WSL2

## [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#why-this-setup-is-confusing)  Why this setup is confusing

Several failures can overlap:

- WSL2 cannot reach the Windows CDP endpoint
- the Control UI is opened from a non-secure origin
- `gateway.controlUi.allowedOrigins` does not match the page origin
- token or pairing is missing
- the browser profile points at the wrong address

Because of that, fixing one layer can still leave a different error visible.

## [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#critical-rule-for-the-control-ui)  Critical rule for the Control UI

When the UI is opened from Windows, use Windows localhost unless you have a deliberate HTTPS setup.Use:`http://127.0.0.1:18789/`Do not default to a LAN IP for the Control UI. Plain HTTP on a LAN or tailnet address can trigger insecure-origin/device-auth behavior that is unrelated to CDP itself. See [Control UI](https://docs.openclaw.ai/web/control-ui).

## [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#validate-in-layers)  Validate in layers

Work top to bottom. Do not skip ahead.

### [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#layer-1-verify-chrome-is-serving-cdp-on-windows)  Layer 1: Verify Chrome is serving CDP on Windows

Start Chrome on Windows with remote debugging enabled:

```
chrome.exe --remote-debugging-port=9222
```

From Windows, verify Chrome itself first:

```
curl http://127.0.0.1:9222/json/version
curl http://127.0.0.1:9222/json/list
```

If this fails on Windows, OpenClaw is not the problem yet.

### [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#layer-2-verify-wsl2-can-reach-that-windows-endpoint)  Layer 2: Verify WSL2 can reach that Windows endpoint

From WSL2, test the exact address you plan to use in `cdpUrl`:

```
curl http://WINDOWS_HOST_OR_IP:9222/json/version
curl http://WINDOWS_HOST_OR_IP:9222/json/list
```

Good result:

- `/json/version` returns JSON with Browser / Protocol-Version metadata
- `/json/list` returns JSON (empty array is fine if no pages are open)

If this fails:

- Windows is not exposing the port to WSL2 yet
- the address is wrong for the WSL2 side
- firewall / port forwarding / local proxying is still missing

Fix that before touching OpenClaw config.

### [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#layer-3-configure-the-correct-browser-profile)  Layer 3: Configure the correct browser profile

For raw remote CDP, point OpenClaw at the address that is reachable from WSL2:

```
{
  browser: {
    enabled: true,
    defaultProfile: \"remote\",
    profiles: {
      remote: {
        cdpUrl: \"http://WINDOWS_HOST_OR_IP:9222\",
        attachOnly: true,
        color: \"#00AA00\",
      },
    },
  },
}
```

Notes:

- use the WSL2-reachable address, not whatever only works on Windows
- keep `attachOnly: true` for externally managed browsers
- `cdpUrl` can be `http://`, `https://`, `ws://`, or `wss://`
- use HTTP(S) when you want OpenClaw to discover `/json/version`
- use WS(S) only when the browser provider gives you a direct DevTools socket URL
- test the same URL with `curl` before expecting OpenClaw to succeed

### [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#layer-4-verify-the-control-ui-layer-separately)  Layer 4: Verify the Control UI layer separately

Open the UI from Windows:`http://127.0.0.1:18789/`Then verify:

- the page origin matches what `gateway.controlUi.allowedOrigins` expects
- token auth or pairing is configured correctly
- you are not debugging a Control UI auth problem as if it were a browser problem

Helpful page:

- [Control UI](https://docs.openclaw.ai/web/control-ui)

### [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#layer-5-verify-end-to-end-browser-control)  Layer 5: Verify end-to-end browser control

From WSL2:

```
openclaw browser open https://example.com --browser-profile remote
openclaw browser tabs --browser-profile remote
```

Good result:

- the tab opens in Windows Chrome
- `openclaw browser tabs` returns the target
- later actions (`snapshot`, `screenshot`, `navigate`) work from the same profile

## [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#common-misleading-errors)  Common misleading errors

Treat each message as a layer-specific clue:

- `control-ui-insecure-auth`
  - UI origin / secure-context problem, not a CDP transport problem
- `token_missing`
  - auth configuration problem
- `pairing required`
  - device approval problem
- `Remote CDP for profile \"remote\" is not reachable`
  - WSL2 cannot reach the configured `cdpUrl`
- `Browser attachOnly is enabled and CDP websocket for profile \"remote\" is not reachable`
  - the HTTP endpoint answered, but the DevTools WebSocket still could not be opened
- stale viewport / dark-mode / locale / offline overrides after a remote session
  - run `openclaw browser stop --browser-profile remote`
  - this closes the active control session and releases Playwright/CDP emulation state without restarting the gateway or the external browser
- `gateway timeout after 1500ms`
  - often still CDP reachability or a slow/unreachable remote endpoint
- `No Chrome tabs found for profile=\"user\"`
  - local Chrome MCP profile selected where no host-local tabs are available

## [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#fast-triage-checklist)  Fast triage checklist

1. Windows: does `curl http://127.0.0.1:9222/json/version` work?
2. WSL2: does `curl http://WINDOWS_HOST_OR_IP:9222/json/version` work?
3. OpenClaw config: does `browser.profiles.<name>.cdpUrl` use that exact WSL2-reachable address?
4. Control UI: are you opening `http://127.0.0.1:18789/` instead of a LAN IP?
5. Are you trying to use `existing-session` across WSL2 and Windows instead of raw remote CDP?

## [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#practical-takeaway)  Practical takeaway

The setup is usually viable. The hard part is that browser transport, Control UI origin security, and token/pairing can each fail independently while looking similar from the user side.When in doubt:

- verify the Windows Chrome endpoint locally first
- verify the same endpoint from WSL2 second
- only then debug OpenClaw config or Control UI auth

## [​](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting\\#related)  Related

- [Browser](https://docs.openclaw.ai/tools/browser)
- [Browser login](https://docs.openclaw.ai/tools/browser-login)
- [Browser Linux troubleshooting](https://docs.openclaw.ai/tools/browser-linux-troubleshooting)

[Browser troubleshooting](https://docs.openclaw.ai/tools/browser-linux-troubleshooting) [Web Fetch](https://docs.openclaw.ai/tools/web-fetch)

Ctrl+I

---

## Slash commands
**Source:** https://docs.openclaw.ai/tools/slash-commands

[Skip to main content](https://docs.openclaw.ai/tools/slash-commands#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Skills

Slash commands

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Config](https://docs.openclaw.ai/tools/slash-commands#config)
- [Command list](https://docs.openclaw.ai/tools/slash-commands#command-list)
- [Core built-in commands](https://docs.openclaw.ai/tools/slash-commands#core-built-in-commands)
- [Generated dock commands](https://docs.openclaw.ai/tools/slash-commands#generated-dock-commands)
- [Bundled plugin commands](https://docs.openclaw.ai/tools/slash-commands#bundled-plugin-commands)
- [Dynamic skill commands](https://docs.openclaw.ai/tools/slash-commands#dynamic-skill-commands)
- [/tools](https://docs.openclaw.ai/tools/slash-commands#%2Ftools)
- [Usage surfaces (what shows where)](https://docs.openclaw.ai/tools/slash-commands#usage-surfaces-what-shows-where)
- [Model selection (/model)](https://docs.openclaw.ai/tools/slash-commands#model-selection-%2Fmodel)
- [Debug overrides](https://docs.openclaw.ai/tools/slash-commands#debug-overrides)
- [Plugin trace output](https://docs.openclaw.ai/tools/slash-commands#plugin-trace-output)
- [Config updates](https://docs.openclaw.ai/tools/slash-commands#config-updates)
- [MCP updates](https://docs.openclaw.ai/tools/slash-commands#mcp-updates)
- [Plugin updates](https://docs.openclaw.ai/tools/slash-commands#plugin-updates)
- [Surface notes](https://docs.openclaw.ai/tools/slash-commands#surface-notes)
- [BTW side questions](https://docs.openclaw.ai/tools/slash-commands#btw-side-questions)
- [Related](https://docs.openclaw.ai/tools/slash-commands#related)

Commands are handled by the Gateway. Most commands must be sent as a **standalone** message that starts with `/`. The host-only bash chat command uses `! <cmd>` (with `/bash <cmd>` as an alias).When a conversation or thread is bound to an ACP session, normal follow-up text routes to that ACP harness. Gateway management commands still stay local: `/acp ...` always reaches the OpenClaw ACP command handler, and `/status` plus `/unfocus` stay local whenever command handling is enabled for the surface.There are two related systems:\n\nCommands\n\nStandalone `/...` messages.\n\nDirectives\n\n`/think`, `/fast`, `/verbose`, `/trace`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.\n\n- Directives are stripped from the message before the model sees it.\n- In normal chat messages (not directive-only), they are treated as “inline hints” and do **not** persist session settings.\n- In directive-only messages (the message contains only directives), they persist to the session and reply with an acknowledgement.\n- Directives are only applied for **authorized senders**. If `commands.allowFrom` is set, it is the only allowlist used; otherwise authorization comes from channel allowlists/pairing plus `commands.useAccessGroups`. Unauthorized senders see directives treated as plain text.\n\nInline shortcuts\n\nAllowlisted/authorized senders only: `/help`, `/commands`, `/status`, `/whoami` (`/id`).They run immediately, are stripped before the model sees the message, and the remaining text continues through the normal flow.\n\n## [​](https://docs.openclaw.ai/tools/slash-commands\\#config)  Config\n\n```\n{\n  commands: {\n    native: \"auto\",\n    nativeSkills: \"auto\",\n    text: true,\n    bash: false,\n    bashForegroundMs: 2000,\n    config: false,\n    mcp: false,\n    plugins: false,\n    debug: false,\n    restart: true,\n    ownerAllowFrom: [\"discord:123456789012345678\"],\n    ownerDisplay: \"raw\",\n    ownerDisplaySecret: \"${OWNER_ID_HASH_SECRET}\",\n    allowFrom: {\n      \"*\": [\"user1\"],\n      discord: [\"user:123\"],\n    },\n    useAccessGroups: true,\n  },\n}\n```\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-text)\n\ncommands.text\n\nboolean\n\ndefault:\"true\"\n\nEnables parsing `/...` in chat messages. On surfaces without native commands (WhatsApp/WebChat/Signal/iMessage/Google Chat/Microsoft Teams), text commands still work even if you set this to `false`.\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-native)\n\ncommands.native\n\nboolean \\| \"auto\"\n\ndefault:\"\\\\\"auto\\\\\"\"\n\nRegisters native commands. Auto: on for Discord/Telegram; off for Slack (until you add slash commands); ignored for providers without native support. Set `channels.discord.commands.native`, `channels.telegram.commands.native`, or `channels.slack.commands.native` to override per provider (bool or `\"auto\"`). `false` clears previously registered commands on Discord/Telegram at startup. Slack commands are managed in the Slack app and are not removed automatically.\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-native-skills)\n\ncommands.nativeSkills\n\nboolean \\| \"auto\"\n\ndefault:\"\\\\\"auto\\\\\"\"\n\nRegisters **skill** commands natively when supported. Auto: on for Discord/Telegram; off for Slack (Slack requires creating a slash command per skill). Set `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills`, or `channels.slack.commands.nativeSkills` to override per provider (bool or `\"auto\"`).\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-bash)\n\ncommands.bash\n\nboolean\n\ndefault:\"false\"\n\nEnables `! <cmd>` to run host shell commands (`/bash <cmd>` is an alias; requires `tools.elevated` allowlists).\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-bash-foreground-ms)\n\ncommands.bashForegroundMs\n\nnumber\n\ndefault:\"2000\"\n\nControls how long bash waits before switching to background mode (`0` backgrounds immediately).\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-config)\n\ncommands.config\n\nboolean\n\ndefault:\"false\"\n\nEnables `/config` (reads/writes `openclaw.json`).\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-mcp)\n\ncommands.mcp\n\nboolean\n\ndefault:\"false\"\n\nEnables `/mcp` (reads/writes OpenClaw-managed MCP config under `mcp.servers`).\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-plugins)\n\ncommands.plugins\n\nboolean\n\ndefault:\"false\"\n\nEnables `/plugins` (plugin discovery/status plus install + enable/disable controls).\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-debug)\n\ncommands.debug\n\nboolean\n\ndefault:\"false\"\n\nEnables `/debug` (runtime-only overrides).\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-restart)\n\ncommands.restart\n\nboolean\n\ndefault:\"true\"\n\nEnables `/restart` plus gateway restart tool actions.\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-owner-allow-from)\n\ncommands.ownerAllowFrom\n\nstring\\[\\]\n\nSets the explicit owner allowlist for owner-only command/tool surfaces. Separate from `commands.allowFrom`.\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-channels-channel-commands-enforce-owner-for-commands)\n\nchannels.<channel>.commands.enforceOwnerForCommands\n\nboolean\n\ndefault:\"false\"\n\nPer-channel: makes owner-only commands require **owner identity** to run on that surface. When `true`, the sender must either match a resolved owner candidate (for example an entry in `commands.ownerAllowFrom` or provider-native owner metadata) or hold internal `operator.admin` scope on an internal message channel. A wildcard entry in channel `allowFrom`, or an empty/unresolved owner-candidate list, is **not** sufficient — owner-only commands fail closed on that channel. Leave this off if you want owner-only commands gated only by `ownerAllowFrom` and the standard command allowlists.\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-owner-display)\n\ncommands.ownerDisplay\n\n\"raw\" \\| \"hash\"\n\nControls how owner ids appear in the system prompt.\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-owner-display-secret)\n\ncommands.ownerDisplaySecret\n\nstring\n\nOptionally sets the HMAC secret used when `commands.ownerDisplay=\"hash\"`.\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-allow-from)\n\ncommands.allowFrom\n\nobject\n\nPer-provider allowlist for command authorization. When configured, it is the only authorization source for commands and directives (channel allowlists/pairing and `commands.useAccessGroups` are ignored). Use `\"*\"` for a global default; provider-specific keys override it.\n\n[​](https://docs.openclaw.ai/tools/slash-commands#param-commands-use-access-groups)\n\ncommands.useAccessGroups\n\nboolean\n\ndefault:\"true\"\n\nEnforces allowlists/policies for commands when `commands.allowFrom` is not set.\n\n## [​](https://docs.openclaw.ai/tools/slash-commands\\#command-list)  Command list\n\nCurrent source-of-truth:\n\n- core built-ins come from `src/auto-reply/commands-registry.shared.ts`\n- generated dock commands come from `src/auto-reply/commands-registry.data.ts`\n- plugin commands come from plugin `registerCommand()` calls\n- actual availability on your gateway still depends on config flags, channel surface, and installed/enabled plugins\n\n### [​](https://docs.openclaw.ai/tools/slash-commands\\#core-built-in-commands)  Core built-in commands\n\nSessions and runs\n\n- `/new [model]` starts a new session; `/reset` is the reset alias.\n- `/reset soft [message]` keeps the current transcript, drops reused CLI backend session ids, and reruns startup/system-prompt loading in-place.\n- `/compact [instructions]` compacts the session context. See [Compaction](https://docs.openclaw.ai/concepts/compaction).\n- `/stop` aborts the current run.\n- `/session idle <duration|off>` and `/session max-age <duration|off>` manage thread-binding expiry.\n- `/export-session [path]` exports the current session to HTML. Alias: `/export`.\n- `/export-trajectory [path]` exports a JSONL [trajectory bundle](https://docs.openclaw.ai/tools/trajectory) for the current session. Alias: `/trajectory`.\n\nModel and run controls\n\n- `/think <level>` sets the thinking level. Options come from the active model’s provider profile; common levels are `off`, `minimal`, `low`, `medium`, and `high`, with custom levels such as `xhigh`, `adaptive`, `max`, or binary `on` only where supported. Aliases: `/thinking`, `/t`.\n- `/verbose on|off|full` toggles verbose output. Alias: `/v`.\n- `/trace on|off` toggles plugin trace output for the current session.\n- `/fast [status|on|off]` shows or sets fast mode.\n- `/reasoning [on|off|stream]` toggles reasoning visibility. Alias: `/reason`.\n- `/elevated [on|off|ask|full]` toggles elevated mode. Alias: `/elev`.\n- `/exec host=<auto|sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>` shows or sets exec defaults.\n- `/model [name|#|status]` shows or sets the model.\n- `/models [provider] [page] [limit=<n>|size=<n>|all]` lists providers or models for a provider.\n- `/queue <mode>` manages queue behavior (`steer`, `interrupt`, `followup`, `collect`, `steer-backlog`) plus options like `debounce:2s cap:25 drop:summarize`.\n\nDiscovery and status\n\n- `/help` shows the short help summary.\n- `/commands` shows the generated command catalog.\n- `/tools [compact|verbose]` shows what the current agent can use right now.\n- `/status` shows execution/runtime status, including `Execution`/`Runtime` labels and provider usage/quota when available.\n- `/crestodian <request>` runs the Crestodian setup and repair helper from an owner DM.\n- `/tasks` lists active/recent background tasks for the current session.\n- `/context [list|detail|json]` explains how context is assembled.\n- `/whoami` shows your sender id. Alias: `/id`.\n- `/usage off|tokens|full|cost` controls the per-response usage footer or prints a local cost summary.\n\nSkills, allowlists, approvals\n\n- `/skill <name> [input]` runs a skill by name.\n- `/allowlist [list|add|remove] ...` manages allowlist entries. Text-only.\n- `/approve <id> <decision>` resolves exec approval prompts.\n- `/btw <question>` asks a side question without changing future session context. See [BTW](https://docs.openclaw.ai/tools/btw).\n\nSubagents and ACP\n\n- `/subagents list|kill|log|info|send|steer|spawn` manages sub-agent runs for the current session.\n- `/acp spawn|cancel|steer|close|sessions|status|set-mode|set|cwd|permissions|timeout|model|reset-options|doctor|install|help` manages ACP sessions and runtime options.\n- `/focus <target>` binds the current Discord thread or Telegram topic/conversation to a session target.\n- `/unfocus` removes the current binding.\n- `/agents` lists thread-bound agents for the current session.\n- `/kill <id|#|all>` aborts one or all running sub-agents.\n- `/steer <id|#> <message>` sends steering to a running sub-agent. Alias: `/tell`.\n\nOwner-only writes and admin\n\n- `/config show|get|set|unset` reads or writes `openclaw.json`. Owner-only. Requires `commands.config: true`.\n- `/mcp show|get|set|unset` reads or writes OpenClaw-managed MCP server config under `mcp.servers`. Owner-only. Requires `commands.mcp: true`.\n- `/plugins list|inspect|show|get|install|enable|disable` inspects or mutates plugin state. `/plugin` is an alias. Owner-only for writes. Requires `commands.plugins: true`.\n- `/debug show|set|unset|reset` manages runtime-only config overrides. Owner-only. Requires `commands.debug: true`.\n- `/restart` restarts OpenClaw when enabled. Default: enabled; set `commands.restart: false` to disable it.\n- `/send on|off|inherit` sets send policy. Owner-only.\n\nVoice, TTS, channel control\n\n- `/tts on|off|status|chat|latest|provider|limit|summary|audio|help` controls TTS. See [TTS](https://docs.openclaw.ai/tools/tts).\n- `/activation mention|always` sets group activation mode.\n- `/bash <command>` runs a host shell command. Text-only. Alias: `! <command>`. Requires `commands.bash: true` plus `tools.elevated` allowlists.\n- `!poll [sessionId]` checks a background bash job.\n- `!stop [sessionId]` stops a background bash job.\n\n### [​](https://docs.openclaw.ai/tools/slash-commands\\#generated-dock-commands)  Generated dock commands\n\nDock commands switch the current session’s reply route to another linked\nchannel. See [Channel docking](https://docs.openclaw.ai/concepts/channel-docking) for setup,\nexamples, and troubleshooting.Dock commands are generated from channel plugins with native-command support. Current bundled set:\n\n- `/dock-discord` (alias: `/dock_discord`)\n- `/dock-mattermost` (alias: `/dock_mattermost`)\n- `/dock-slack` (alias: `/dock_slack`)\n- `/dock-telegram` (alias: `/dock_telegram`)\n\nUse dock commands from a direct chat to switch the current session’s reply route to another linked channel. The agent keeps the same session context, but future replies for that session are delivered to the selected channel peer.Dock commands require `session.identityLinks`. The source sender and target peer must be in the same identity group, for example `[\"telegram:123\", \"discord:456\"]`. If a Telegram user with id `123` sends `/dock_discord`, OpenClaw stores `lastChannel: \"discord\"` and `lastTo: \"456\"` on the active session. If the sender is not linked to a Discord peer, the command replies with a setup hint instead of falling through to normal chat.Docking changes the active session route only. It does not create channel accounts, grant access, bypass channel allowlists, or move transcript history to another session. Use `/dock-telegram`, `/dock-slack`, `/dock-mattermost`, or another generated dock command to switch the route again.\n\n### [​](https://docs.openclaw.ai/tools/slash-commands\\#bundled-plugin-commands)  Bundled plugin commands\n\nBundled plugins can add more slash commands. Current bundled commands in this repo:\n\n- `/dreaming [on|off|status|help]` toggles memory dreaming. See [Dreaming](https://docs.openclaw.ai/concepts/dreaming).\n- `/pair [qr|status|pending|approve|cleanup|notify]` manages device pairing/setup flow. See [Pairing](https://docs.openclaw.ai/channels/pairing).\n- `/phone status|arm <camera|screen|writes|all> [duration]|disarm` temporarily arms high-risk phone node commands.\n- `/voice status|list [limit]|set <voiceId|name>` manages Talk voice config. On Discord, the native command name is `/talkvoice`.\n- `/card ...` sends LINE rich card presets. See [LINE](https://docs.openclaw.ai/channels/line).\n- `/codex status|models|threads|resume|compact|review|account|mcp|skills` inspects and controls the bundled Codex app-server harness. See [Codex harness](https://docs.openclaw.ai/plugins/codex-harness).\n- QQBot-only commands:\n  - `/bot-ping`\n  - `/bot-version`\n  - `/bot-help`\n  - `/bot-upgrade`\n  - `/bot-logs`\n\n### [​](https://docs.openclaw.ai/tools/slash-commands\\#dynamic-skill-commands)  Dynamic skill commands\n\nUser-invocable skills are also exposed as slash commands:\n\n- `/skill <name> [input]` always works as the generic entrypoint.\n- skills may also appear as direct commands like `/prose` when the skill/plugin registers them.\n- native skill-command registration is controlled by `commands.nativeSkills` and `channels.<provider>.commands.nativeSkills`.\n\nArgument and parser notes\n\n- Commands accept an optional `:` between the command and args (e.g. `/think: high`, `/send: on`, `/help:`).\n- `/new <model>` accepts a model alias, `provider/model`, or a provider name (fuzzy match); if no match, the text is treated as the message body.\n- For full provider usage breakdown, use `openclaw status --usage`.\n- `/allowlist add|remove` requires `commands.config=true` and honors channel `configWrites`.\n- In multi-account channels, config-targeted `/allowlist --account <id>` and `/config set channels.<provider>.accounts.<id>...` also honor the target account’s `configWrites`.\n- `/usage` controls the per-response usage footer; `/usage cost` prints a local cost summary from OpenClaw session logs.\n- `/restart` is enabled by default; set `commands.restart: false` to disable it.\n- `/plugins install <spec>` accepts the same plugin specs as `openclaw plugins install`: local path/archive, npm package, or `clawhub:<pkg>`.\n- `/plugins enable|disable` updates plugin config and may prompt for a restart.\n\nChannel-specific behavior\n\n- Discord-only native command: `/vc join|leave|status` controls voice channels (not available as text). `join` requires a guild and selected voice/stage channel. Requires `channels.discord.voice` and native commands.\n- Discord thread-binding commands (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`) require effective thread bindings to be enabled (`session.threadBindings.enabled` and/or `channels.discord.threadBindings.enabled`).\n- ACP command refere...(content truncated)

---

## LLM task - OpenClaw
**Source:** https://docs.openclaw.ai/tools/llm-task

[Skip to main content](https://docs.openclaw.ai/tools/llm-task#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nTools\n\nLLM task\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Enable the plugin](https://docs.openclaw.ai/tools/llm-task#enable-the-plugin)\n- [Config (optional)](https://docs.openclaw.ai/tools/llm-task#config-optional)\n- [Tool parameters](https://docs.openclaw.ai/tools/llm-task#tool-parameters)\n- [Output](https://docs.openclaw.ai/tools/llm-task#output)\n- [Example: Lobster workflow step](https://docs.openclaw.ai/tools/llm-task#example-lobster-workflow-step)\n- [Safety notes](https://docs.openclaw.ai/tools/llm-task#safety-notes)\n- [Related](https://docs.openclaw.ai/tools/llm-task#related)\n\n`llm-task` is an **optional plugin tool** that runs a JSON-only LLM task and\nreturns structured output (optionally validated against JSON Schema).This is ideal for workflow engines like Lobster: you can add a single LLM step\nwithout writing custom OpenClaw code for each workflow.\n\n## [​](https://docs.openclaw.ai/tools/llm-task\\#enable-the-plugin)  Enable the plugin\n\n1. Enable the plugin:\n\n```\n{\n  \"plugins\": {\n    \"entries\": {\n      \"llm-task\": { \"enabled\": true }\n    }\n  }\n}\n```\n\n2. Allowlist the tool (it is registered with `optional: true`):\n\n```\n{\n  \"agents\": {\n    \"list\": [\\\n      {\\\n        \"id\": \"main\",\\\n        \"tools\": { \"allow\": [\"llm-task\"] }\\\n      }\\\n    ]\n  }\n}\n```\n\n## [​](https://docs.openclaw.ai/tools/llm-task\\#config-optional)  Config (optional)\n\n```\n{\n  \"plugins\": {\n    \"entries\": {\n      \"llm-task\": {\n        \"enabled\": true,\n        \"config\": {\n          \"defaultProvider\": \"openai-codex\",\n          \"defaultModel\": \"gpt-5.5\",\n          \"defaultAuthProfileId\": \"main\",\n          \"allowedModels\": [\"openai/gpt-5.4\"],\n          \"maxTokens\": 800,\n          \"timeoutMs\": 30000\n        }\n      }\n    }\n  }\n}\n```\n\n`allowedModels` is an allowlist of `provider/model` strings. If set, any request\noutside the list is rejected.\n\n## [​](https://docs.openclaw.ai/tools/llm-task\\#tool-parameters)  Tool parameters\n\n- `prompt` (string, required)\n- `input` (any, optional)\n- `schema` (object, optional JSON Schema)\n- `provider` (string, optional)\n- `model` (string, optional)\n- `thinking` (string, optional)\n- `authProfileId` (string, optional)\n- `temperature` (number, optional)\n- `maxTokens` (number, optional)\n- `timeoutMs` (number, optional)\n\n`thinking` accepts the standard OpenClaw reasoning presets, such as `low` or `medium`.\n\n## [​](https://docs.openclaw.ai/tools/llm-task\\#output)  Output\n\nReturns `details.json` containing the parsed JSON (and validates against\n`schema` when provided).\n\n## [​](https://docs.openclaw.ai/tools/llm-task\\#example-lobster-workflow-step)  Example: Lobster workflow step\n\n```\nopenclaw.invoke --tool llm-task --action json --args-json \'{\n  \"prompt\": \"Given the input email, return intent and draft.\",\n  \"thinking\": \"low\",\n  \"input\": {\n    \"subject\": \"Hello\",\n    \"body\": \"Can you help?\"\n  },\n  \"schema\": {\n    \"type\": \"object\",\n    \"properties\": {\n      \"intent\": { \"type\": \"string\" },\n      \"draft\": { \"type\": \"string\" }\n    },\n    \"required\": [\"intent\", \"draft\"],\n    \"additionalProperties\": false\n  }\n}\'\n```\n\n## [​](https://docs.openclaw.ai/tools/llm-task\\#safety-notes)  Safety notes\n\n- The tool is **JSON-only** and instructs the model to output only JSON (no\ncode fences, no commentary).\n- No tools are exposed to the model for this run.\n- Treat output as untrusted unless you validate with `schema`.\n- Put approvals before any side-effecting step (send, post, exec).\n\n## [​](https://docs.openclaw.ai/tools/llm-task\\#related)  Related\n\n- [Thinking levels](https://docs.openclaw.ai/tools/thinking)\n- [Sub-agents](https://docs.openclaw.ai/tools/subagents)\n- [Slash commands](https://docs.openclaw.ai/tools/slash-commands)\n\n[Image generation](https://docs.openclaw.ai/tools/image-generation) [Lobster](https://docs.openclaw.ai/tools/lobster)\n\nCtrl+I

---

## Kimi search - OpenClaw
**Source:** https://docs.openclaw.ai/tools/kimi-search

[Skip to main content](https://docs.openclaw.ai/tools/kimi-search#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web tools

Kimi search

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Get an API key](https://docs.openclaw.ai/tools/kimi-search#get-an-api-key)
- [Config](https://docs.openclaw.ai/tools/kimi-search#config)
- [How it works](https://docs.openclaw.ai/tools/kimi-search#how-it-works)
- [Supported parameters](https://docs.openclaw.ai/tools/kimi-search#supported-parameters)
- [Related](https://docs.openclaw.ai/tools/kimi-search#related)

OpenClaw supports Kimi as a `web_search` provider, using Moonshot web search\nto produce AI-synthesized answers with citations.\n\n## [​](https://docs.openclaw.ai/tools/kimi-search\\#get-an-api-key)  Get an API key\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/tools/kimi-search#)\n\nCreate a key\n\nGet an API key from [Moonshot AI](https://platform.moonshot.cn/).\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/tools/kimi-search#)\n\nStore the key\n\nSet `KIMI_API_KEY` or `MOONSHOT_API_KEY` in the Gateway environment, or\nconfigure via:\n\n```\nopenclaw configure --section web\n```\n\nWhen you choose **Kimi** during `openclaw onboard` or\n`openclaw configure --section web`, OpenClaw can also ask for:\n\n- the Moonshot API region:\n  - `https://api.moonshot.ai/v1`\n  - `https://api.moonshot.cn/v1`\n- the default Kimi web-search model (defaults to `kimi-k2.6`)\n\n## [​](https://docs.openclaw.ai/tools/kimi-search\\#config)  Config\n\n```\n{\n  plugins: {\n    entries: {\n      moonshot: {\n        config: {\n          webSearch: {\n            apiKey: \"sk-...\", // optional if KIMI_API_KEY or MOONSHOT_API_KEY is set\n            baseUrl: \"https://api.moonshot.ai/v1\",\n            model: \"kimi-k2.6\",\n          },\n        },\n      },\n    },\n  },\n  tools: {\n    web: {\n      search: {\n        provider: \"kimi\",\n      },\n    },\n  },\n}\n```\n\nIf you use the China API host for chat (`models.providers.moonshot.baseUrl`:\n`https://api.moonshot.cn/v1`), OpenClaw reuses that same host for Kimi\n`web_search` when `tools.web.search.kimi.baseUrl` is omitted, so keys from\n[platform.moonshot.cn](https://platform.moonshot.cn/) do not hit the\ninternational endpoint by mistake (which often returns HTTP 401). Override\nwith `tools.web.search.kimi.baseUrl` when you need a different search base URL.**Environment alternative:** set `KIMI_API_KEY` or `MOONSHOT_API_KEY` in the\nGateway environment. For a gateway install, put it in `~/.openclaw/.env`.If you omit `baseUrl`, OpenClaw defaults to `https://api.moonshot.ai/v1`.\nIf you omit `model`, OpenClaw defaults to `kimi-k2.6`.\n\n## [​](https://docs.openclaw.ai/tools/kimi-search\\#how-it-works)  How it works\n\nKimi uses Moonshot web search to synthesize answers with inline citations,\nsimilar to Gemini and Grok’s grounded response approach.\n\n## [​](https://docs.openclaw.ai/tools/kimi-search\\#supported-parameters)  Supported parameters\n\nKimi search supports `query`.`count` is accepted for shared `web_search` compatibility, but Kimi still\nreturns one synthesized answer with citations rather than an N-result list.Provider-specific filters are not currently supported.\n\n## [​](https://docs.openclaw.ai/tools/kimi-search\\#related)  Related\n\n- [Web Search overview](https://docs.openclaw.ai/tools/web) — all providers and auto-detection\n- [Moonshot AI](https://docs.openclaw.ai/providers/moonshot) — Moonshot model + Kimi Coding provider docs\n- [Gemini Search](https://docs.openclaw.ai/tools/gemini-search) — AI-synthesized answers via Google grounding\n- [Grok Search](https://docs.openclaw.ai/tools/grok-search) — AI-synthesized answers via xAI grounding\n\n[Grok search](https://docs.openclaw.ai/tools/grok-search) [MiniMax search](https://docs.openclaw.ai/tools/minimax-search)\n\nCtrl+I

---

## Tools and plugins - OpenClaw
**Source:** https://docs.openclaw.ai/tools/index

[Skip to main content](https://docs.openclaw.ai/tools/index#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nOverview\n\nTools and plugins\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Tools, skills, and plugins](https://docs.openclaw.ai/tools/index#tools-skills-and-plugins)\n- [Built-in tools](https://docs.openclaw.ai/tools/index#built-in-tools)\n- [Plugin-provided tools](https://docs.openclaw.ai/tools/index#plugin-provided-tools)\n- [Tool configuration](https://docs.openclaw.ai/tools/index#tool-configuration)\n- [Allow and deny lists](https://docs.openclaw.ai/tools/index#allow-and-deny-lists)\n- [Tool profiles](https://docs.openclaw.ai/tools/index#tool-profiles)\n- [Tool groups](https://docs.openclaw.ai/tools/index#tool-groups)\n- [Provider-specific restrictions](https://docs.openclaw.ai/tools/index#provider-specific-restrictions)\n\nEverything the agent does beyond generating text happens through **tools**.\nTools are how the agent reads files, runs commands, browses the web, sends\nmessages, and interacts with devices.\n\n## [​](https://docs.openclaw.ai/tools/index\\#tools-skills-and-plugins)  Tools, skills, and plugins\n\nOpenClaw has three layers that work together:\n\n1\n\n[Navigate to header](https://docs.openclaw.ai/tools/index#)\n\nTools are what the agent calls\n\nA tool is a typed function the agent can invoke (e.g. `exec`, `browser`,\n`web_search`, `message`). OpenClaw ships a set of **built-in tools** and\nplugins can register additional ones.The agent sees tools as structured function definitions sent to the model API.\n\n2\n\n[Navigate to header](https://docs.openclaw.ai/tools/index#)\n\nSkills teach the agent when and how\n\nA skill is a markdown file (`SKILL.md`) injected into the system prompt.\nSkills give the agent context, constraints, and step-by-step guidance for\nusing tools effectively. Skills live in your workspace, in shared folders,\nor ship inside plugins.[Skills reference](https://docs.openclaw.ai/tools/skills) \\| [Creating skills](https://docs.openclaw.ai/tools/creating-skills)\n\n3\n\n[Navigate to header](https://docs.openclaw.ai/tools/index#)\n\nPlugins package everything together\n\nA plugin is a package that can register any combination of capabilities:\nchannels, model providers, tools, skills, speech, realtime transcription,\nrealtime voice, media understanding, image generation, video generation,\nweb fetch, web search, and more. Some plugins are **core** (shipped with\nOpenClaw), others are **external** (published on npm by the community).[Install and configure plugins](https://docs.openclaw.ai/tools/plugin) \\| [Build your own](https://docs.openclaw.ai/plugins/building-plugins)\n\n## [​](https://docs.openclaw.ai/tools/index\\#built-in-tools)  Built-in tools\n\nThese tools ship with OpenClaw and are available without installing any plugins:\n\n| Tool | What it does | Page |\n| --- | --- | --- |\n| `exec` / `process` | Run shell commands, manage background processes | [Exec](https://docs.openclaw.ai/tools/exec), [Exec Approvals](https://docs.openclaw.ai/tools/exec-approvals) |\n| `code_execution` | Run sandboxed remote Python analysis | [Code Execution](https://docs.openclaw.ai/tools/code-execution) |\n| `browser` | Control a Chromium browser (navigate, click, screenshot) | [Browser](https://docs.openclaw.ai/tools/browser) |\n| `web_search` / `x_search` / `web_fetch` | Search the web, search X posts, fetch page content | [Web](https://docs.openclaw.ai/tools/web), [Web Fetch](https://docs.openclaw.ai/tools/web-fetch) |\n| `read` / `write` / `edit` | File I/O in the workspace |  |\n| `apply_patch` | Multi-hunk file patches | [Apply Patch](https://docs.openclaw.ai/tools/apply-patch) |\n| `message` | Send messages across all channels | [Agent Send](https://docs.openclaw.ai/tools/agent-send) |\n| `canvas` | Drive node Canvas (present, eval, snapshot) |  |\n| `nodes` | Discover and target paired devices |  |\n| `cron` / `gateway` | Manage scheduled jobs; inspect, patch, restart, or update the gateway |  |\n| `image` / `image_generate` | Analyze or generate images | [Image Generation](https://docs.openclaw.ai/tools/image-generation) |\n| `music_generate` | Generate music tracks | [Music Generation](https://docs.openclaw.ai/tools/music-generation) |\n| `video_generate` | Generate videos | [Video Generation](https://docs.openclaw.ai/tools/video-generation) |\n| `tts` | One-shot text-to-speech conversion | [TTS](https://docs.openclaw.ai/tools/tts) |\n| `sessions_*` / `subagents` / `agents_list` | Session management, status, and sub-agent orchestration | [Sub-agents](https://docs.openclaw.ai/tools/subagents) |\n| `session_status` | Lightweight `/status`-style readback and session model override | [Session Tools](https://docs.openclaw.ai/concepts/session-tool) |\n\nFor image work, use `image` for analysis and `image_generate` for generation or editing. If you target `openai/*`, `google/*`, `fal/*`, or another non-default image provider, configure that provider’s auth/API key first.For music work, use `music_generate`. If you target `google/*`, `minimax/*`, or another non-default music provider, configure that provider’s auth/API key first.For video work, use `video_generate`. If you target `qwen/*` or another non-default video provider, configure that provider’s auth/API key first.For workflow-driven audio generation, use `music_generate` when a plugin such as\nComfyUI registers it. This is separate from `tts`, which is text-to-speech.`session_status` is the lightweight status/readback tool in the sessions group.\nIt answers `/status`-style questions about the current session and can\noptionally set a per-session model override; `model=default` clears that\noverride. Like `/status`, it can backfill sparse token/cache counters and the\nactive runtime model label from the latest transcript usage entry.`gateway` is the owner-only runtime tool for gateway operations:\n\n- `config.schema.lookup` for one path-scoped config subtree before edits\n- `config.get` for the current config snapshot + hash\n- `config.patch` for partial config updates with restart\n- `config.apply` only for full-config replacement\n- `update.run` for explicit self-update + restart\n\nFor partial changes, prefer `config.schema.lookup` then `config.patch`. Use\n`config.apply` only when you intentionally replace the entire config.\nFor broader config docs, read [Configuration](https://docs.openclaw.ai/gateway/configuration) and\n[Configuration reference](https://docs.openclaw.ai/gateway/configuration-reference).\nThe tool also refuses to change `tools.exec.ask` or `tools.exec.security`;\nlegacy `tools.bash.*` aliases normalize to the same protected exec paths.\n\n### [​](https://docs.openclaw.ai/tools/index\\#plugin-provided-tools)  Plugin-provided tools\n\nPlugins can register additional tools. Some examples:\n\n- [Diffs](https://docs.openclaw.ai/tools/diffs) — diff viewer and renderer\n- [LLM Task](https://docs.openclaw.ai/tools/llm-task) — JSON-only LLM step for structured output\n- [Lobster](https://docs.openclaw.ai/tools/lobster) — typed workflow runtime with resumable approvals\n- [Music Generation](https://docs.openclaw.ai/tools/music-generation) — shared `music_generate` tool with workflow-backed providers\n- [OpenProse](https://docs.openclaw.ai/prose) — markdown-first workflow orchestration\n- [Tokenjuice](https://docs.openclaw.ai/tools/tokenjuice) — compact noisy `exec` and `bash` tool results\n\n## [​](https://docs.openclaw.ai/tools/index\\#tool-configuration)  Tool configuration\n\n### [​](https://docs.openclaw.ai/tools/index\\#allow-and-deny-lists)  Allow and deny lists\n\nControl which tools the agent can call via `tools.allow` / `tools.deny` in\nconfig. Deny always wins over allow.\n\n```\n{\n  tools: {\n    allow: [\"group:fs\", \"browser\", \"web_search\"],\n    deny: [\"exec\"],\n  },\n}\n```\n\nOpenClaw fails closed when an explicit allowlist resolves to no callable tools.\nFor example, `tools.allow: [\"query_db\"]` only works if a loaded plugin actually\nregisters `query_db`. If no built-in, plugin, or bundled MCP tool matches the\nallowlist, the run stops before the model call instead of continuing as a\ntext-only run that could hallucinate tool results.\n\n### [​](https://docs.openclaw.ai/tools/index\\#tool-profiles)  Tool profiles\n\n`tools.profile` sets a base allowlist before `allow`/`deny` is applied.\nPer-agent override: `agents.list[].tools.profile`.\n\n| Profile | What it includes |\n| --- | --- |\n| `full` | Unrestricted baseline for broader command/control access; same as leaving `tools.profile` unset |\n| `coding` | `group:fs`, `group:runtime`, `group:web`, `group:sessions`, `group:memory`, `cron`, `image`, `image_generate`, `music_generate`, `video_generate` |\n| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |\n| `minimal` | `session_status` only |\n\n`tools.profile: \"messaging\"` is intentionally narrow for channel-focused\nagents. It leaves out broader command/control tools such as filesystem, runtime,\nbrowser, canvas, nodes, cron, and gateway control. Use `tools.profile: \"full\"`\nas the unrestricted baseline for broader command/control access, then trim\naccess with `tools.allow` / `tools.deny` when needed.\n\n`coding` includes lightweight web tools (`web_search`, `web_fetch`, `x_search`)\nbut not the full browser-control tool. Browser automation can drive real\nsessions and logged-in profiles, so add it explicitly with\n`tools.alsoAllow: [\"browser\"]` or a per-agent\n`agents.list[].tools.alsoAllow: [\"browser\"]`.The `coding` and `messaging` profiles also allow configured bundle MCP tools\nunder the plugin key `bundle-mcp`. Add `tools.deny: [\"bundle-mcp\"]` when you\nwant a profile to keep its normal built-ins but hide all configured MCP tools.\nThe `minimal` profile does not include bundle MCP tools.Example (broadest tool surface by default):\n\n```\n{\n  tools: {\n    profile: \"full\",\n  },\n}\n```\n\n### [​](https://docs.openclaw.ai/tools/index\\#tool-groups)  Tool groups\n\nUse `group:*` shorthands in allow/deny lists:\n\n| Group | Tools |\n| --- | --- |\n| `group:runtime` | exec, process, code\\_execution (`bash` is accepted as an alias for `exec`) |\n| `group:fs` | read, write, edit, apply\\_patch |\n| `group:sessions` | sessions\\_list, sessions\\_history, sessions\\_send, sessions\\_spawn, sessions\\_yield, subagents, session\\_status |\n| `group:memory` | memory\\_search, memory\\_get |\n| `group:web` | web\\_search, x\\_search, web\\_fetch |\n| `group:ui` | browser, canvas |\n| `group:automation` | cron, gateway |\n| `group:messaging` | message |\n| `group:nodes` | nodes |\n| `group:agents` | agents\\_list |\n| `group:media` | image, image\\_generate, music\\_generate, video\\_generate, tts |\n| `group:openclaw` | All built-in OpenClaw tools (excludes plugin tools) |\n\n`sessions_history` returns a bounded, safety-filtered recall view. It strips\nthinking tags, `<relevant-memories>` scaffolding, plain-text tool-call XML\npayloads (including `<tool_call>...</tool_call>`,\n`<function_call>...</function_call>`, `<tool_calls>...</tool_calls>`,\n`<function_calls>...</function_calls>`, and truncated tool-call blocks),\ndowngraded tool-call scaffolding, leaked ASCII/full-width model control\ntokens, and malformed MiniMax tool-call XML from assistant text, then applies\nredaction/truncation and possible oversized-row placeholders instead of acting\nas a raw transcript dump.\n\n### [​](https://docs.openclaw.ai/tools/index\\#provider-specific-restrictions)  Provider-specific restrictions\n\nUse `tools.byProvider` to restrict tools for specific providers without\nchanging global defaults:\n\n```\n{\n  tools: {\n    profile: \"coding\",\n    byProvider: {\n      \"google-antigravity\": { profile: \"minimal\" },\n    },\n  },\n}\n```\n\n[Install and Configure](https://docs.openclaw.ai/tools/plugin)\n\nCtrl+I

---

## Firecrawl - OpenClaw
**Source:** https://docs.openclaw.ai/tools/firecrawl

[Skip to main content](https://docs.openclaw.ai/tools/firecrawl#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web tools

Firecrawl

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Get an API key](https://docs.openclaw.ai/tools/firecrawl#get-an-api-key)
- [Configure Firecrawl search](https://docs.openclaw.ai/tools/firecrawl#configure-firecrawl-search)
- [Configure Firecrawl scrape + web\\_fetch fallback](https://docs.openclaw.ai/tools/firecrawl#configure-firecrawl-scrape-%2B-web_fetch-fallback)
- [Firecrawl plugin tools](https://docs.openclaw.ai/tools/firecrawl#firecrawl-plugin-tools)
- [firecrawl\\_search](https://docs.openclaw.ai/tools/firecrawl#firecrawl_search)
- [firecrawl\\_scrape](https://docs.openclaw.ai/tools/firecrawl#firecrawl_scrape)
- [Stealth / bot circumvention](https://docs.openclaw.ai/tools/firecrawl#stealth-%2F-bot-circumvention)
- [How web\\_fetch uses Firecrawl](https://docs.openclaw.ai/tools/firecrawl#how-web_fetch-uses-firecrawl)
- [Related](https://docs.openclaw.ai/tools/firecrawl#related)

OpenClaw can use **Firecrawl** in three ways:

- as the `web_search` provider
- as explicit plugin tools: `firecrawl_search` and `firecrawl_scrape`
- as a fallback extractor for `web_fetch`

It is a hosted extraction/search service that supports bot circumvention and caching,
which helps with JS-heavy sites or pages that block plain HTTP fetches.

## [​](https://docs.openclaw.ai/tools/firecrawl\\#get-an-api-key)  Get an API key

1. Create a Firecrawl account and generate an API key.
2. Store it in config or set `FIRECRAWL_API_KEY` in the gateway environment.

## [​](https://docs.openclaw.ai/tools/firecrawl\\#configure-firecrawl-search)  Configure Firecrawl search

```
{
  tools: {
    web: {
      search: {
        provider: \"firecrawl\",
      },
    },
  },
  plugins: {
    entries: {
      firecrawl: {
        enabled: true,
        config: {
          webSearch: {
            apiKey: \"FIRECRAWL_API_KEY_HERE\",
            baseUrl: \"https://api.firecrawl.dev\",
          },
        },
      },
    },
  },
}
```

Notes:

- Choosing Firecrawl in onboarding or `openclaw configure --section web` enables the bundled Firecrawl plugin automatically.
- `web_search` with Firecrawl supports `query` and `count`.
- For Firecrawl-specific controls like `sources`, `categories`, or result scraping, use `firecrawl_search`.
- `baseUrl` overrides must stay on `https://api.firecrawl.dev`.
- `FIRECRAWL_BASE_URL` is the shared env fallback for Firecrawl search and scrape base URLs.

## [​](https://docs.openclaw.ai/tools/firecrawl\\#configure-firecrawl-scrape-+-web_fetch-fallback)  Configure Firecrawl scrape + web\\_fetch fallback

```
{
  plugins: {
    entries: {
      firecrawl: {
        enabled: true,
        config: {
          webFetch: {
            apiKey: \"FIRECRAWL_API_KEY_HERE\",
            baseUrl: \"https://api.firecrawl.dev\",
            onlyMainContent: true,
            maxAgeMs: 172800000,
            timeoutSeconds: 60,
          },
        },
      },
    },
  },
}
```

Notes:

- Firecrawl fallback attempts run only when an API key is available (`plugins.entries.firecrawl.config.webFetch.apiKey` or `FIRECRAWL_API_KEY`).
- `maxAgeMs` controls how old cached results can be (ms). Default is 2 days.
- Legacy `tools.web.fetch.firecrawl.*` config is auto-migrated by `openclaw doctor --fix`.
- Firecrawl scrape/base URL overrides are restricted to `https://api.firecrawl.dev`.

`firecrawl_scrape` reuses the same `plugins.entries.firecrawl.config.webFetch.*` settings and env vars.

## [​](https://docs.openclaw.ai/tools/firecrawl\\#firecrawl-plugin-tools)  Firecrawl plugin tools

### [​](https://docs.openclaw.ai/tools/firecrawl\\#firecrawl_search)  `firecrawl_search`

Use this when you want Firecrawl-specific search controls instead of generic `web_search`.Core parameters:

- `query`
- `count`
- `sources`
- `categories`
- `scrapeResults`
- `timeoutSeconds`

### [​](https://docs.openclaw.ai/tools/firecrawl\\#firecrawl_scrape)  `firecrawl_scrape`

Use this for JS-heavy or bot-protected pages where plain `web_fetch` is weak.Core parameters:

- `url`
- `extractMode`
- `maxChars`
- `onlyMainContent`
- `maxAgeMs`
- `proxy`
- `storeInCache`
- `timeoutSeconds`

## [​](https://docs.openclaw.ai/tools/firecrawl\\#stealth-/-bot-circumvention)  Stealth / bot circumvention

Firecrawl exposes a **proxy mode** parameter for bot circumvention (`basic`, `stealth`, or `auto`).
OpenClaw always uses `proxy: \"auto\"` plus `storeInCache: true` for Firecrawl requests.
If proxy is omitted, Firecrawl defaults to `auto`. `auto` retries with stealth proxies if a basic attempt fails, which may use more credits\nthan basic-only scraping.

## [​](https://docs.openclaw.ai/tools/firecrawl\\#how-web_fetch-uses-firecrawl)  How `web_fetch` uses Firecrawl

`web_fetch` extraction order:

1. Readability (local)
2. Firecrawl (if selected or auto-detected as the active web-fetch fallback)
3. Basic HTML cleanup (last fallback)

The selection knob is `tools.web.fetch.provider`. If you omit it, OpenClaw\nauto-detects the first ready web-fetch provider from available credentials.\nToday the bundled provider is Firecrawl.

## [​](https://docs.openclaw.ai/tools/firecrawl\\#related)  Related

- [Web Search overview](https://docs.openclaw.ai/tools/web) — all providers and auto-detection
- [Web Fetch](https://docs.openclaw.ai/tools/web-fetch) — web\\_fetch tool with Firecrawl fallback
- [Tavily](https://docs.openclaw.ai/tools/tavily) — search + extract tools

[Exa search](https://docs.openclaw.ai/tools/exa-search) [Gemini search](https://docs.openclaw.ai/tools/gemini-search)

Ctrl+I

---

## Web tools
**Source:** https://docs.openclaw.ai/tools/web

[Skip to main content](https://docs.openclaw.ai/tools/web#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web tools

Web search

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/tools/web#quick-start)
- [Choosing a provider](https://docs.openclaw.ai/tools/web#choosing-a-provider)
- [Provider comparison](https://docs.openclaw.ai/tools/web#provider-comparison)
- [Auto-detection](https://docs.openclaw.ai/tools/web#auto-detection)
- [Native OpenAI web search](https://docs.openclaw.ai/tools/web#native-openai-web-search)
- [Native Codex web search](https://docs.openclaw.ai/tools/web#native-codex-web-search)
- [Setting up web search](https://docs.openclaw.ai/tools/web#setting-up-web-search)
- [Config](https://docs.openclaw.ai/tools/web#config)
- [Storing API keys](https://docs.openclaw.ai/tools/web#storing-api-keys)
- [Tool parameters](https://docs.openclaw.ai/tools/web#tool-parameters)
- [x\\_search](https://docs.openclaw.ai/tools/web#x_search)
- [x\\_search config](https://docs.openclaw.ai/tools/web#x_search-config)
- [x\\_search parameters](https://docs.openclaw.ai/tools/web#x_search-parameters)
- [x\\_search example](https://docs.openclaw.ai/tools/web#x_search-example)
- [Examples](https://docs.openclaw.ai/tools/web#examples)
- [Tool profiles](https://docs.openclaw.ai/tools/web#tool-profiles)
- [Related](https://docs.openclaw.ai/tools/web#related)

The `web_search` tool searches the web using your configured provider and
returns results. Results are cached by query for 15 minutes (configurable).OpenClaw also includes `x_search` for X (formerly Twitter) posts and
`web_fetch` for lightweight URL fetching. In this phase, `web_fetch` stays
local while `web_search` and `x_search` can use xAI Responses under the hood.

`web_search` is a lightweight HTTP tool, not browser automation. For
JS-heavy sites or logins, use the [Web Browser](https://docs.openclaw.ai/tools/browser). For
fetching a specific URL, use [Web Fetch](https://docs.openclaw.ai/tools/web-fetch).

## [​](https://docs.openclaw.ai/tools/web\\#quick-start)  Quick start

1

[Navigate to header](https://docs.openclaw.ai/tools/web#)

Choose a provider

Pick a provider and complete any required setup. Some providers are
key-free, while others use API keys. See the provider pages below for
details.

2

[Navigate to header](https://docs.openclaw.ai/tools/web#)

Configure

```
openclaw configure --section web
```

This stores the provider and any needed credential. You can also set an env
var (for example `BRAVE_API_KEY`) and skip this step for API-backed
providers.

3

[Navigate to header](https://docs.openclaw.ai/tools/web#)

Use it

The agent can now call `web_search`:

```
await web_search({ query: \"OpenClaw plugin SDK\" });
```

For X posts, use:

```
await x_search({ query: \"dinner recipes\" });
```

## [​](https://docs.openclaw.ai/tools/web\\#choosing-a-provider)  Choosing a provider

[**Brave Search** \\\\\n\\\\\nStructured results with snippets. Supports `llm-context` mode, country/language filters. Free tier available.](https://docs.openclaw.ai/tools/brave-search)

[**DuckDuckGo** \\\\\n\\\\\nKey-free fallback. No API key needed. Unofficial HTML-based integration.](https://docs.openclaw.ai/tools/duckduckgo-search)

[**Exa** \\\\\n\\\\\nNeural + keyword search with content extraction (highlights, text, summaries).](https://docs.openclaw.ai/tools/exa-search)

[**Firecrawl** \\\\\n\\\\\nStructured results. Best paired with `firecrawl_search` and `firecrawl_scrape` for deep extraction.](https://docs.openclaw.ai/tools/firecrawl)

[**Gemini** \\\\\n\\\\\nAI-synthesized answers with citations via Google Search grounding.](https://docs.openclaw.ai/tools/gemini-search)

[**Grok** \\\\\n\\\\\nAI-synthesized answers with citations via xAI web grounding.](https://docs.openclaw.ai/tools/grok-search)

[**Kimi** \\\\\n\\\\\nAI-synthesized answers with citations via Moonshot web search.](https://docs.openclaw.ai/tools/kimi-search)

[**MiniMax Search** \\\\\n\\\\\nStructured results via the MiniMax Coding Plan search API.](https://docs.openclaw.ai/tools/minimax-search)

[**Ollama Web Search** \\\\\n\\\\\nSearch via a signed-in local Ollama host or the hosted Ollama API.](https://docs.openclaw.ai/tools/ollama-search)

[**Perplexity** \\\\\n\\\\\nStructured results with content extraction controls and domain filtering.](https://docs.openclaw.ai/tools/perplexity-search)

[**SearXNG** \\\\\n\\\\\nSelf-hosted meta-search. No API key needed. Aggregates Google, Bing, DuckDuckGo, and more.](https://docs.openclaw.ai/tools/searxng-search)

[**Tavily** \\\\\n\\\\\nStructured results with search depth, topic filtering, and `tavily_extract` for URL extraction.](https://docs.openclaw.ai/tools/tavily)

### [​](https://docs.openclaw.ai/tools/web\\#provider-comparison)  Provider comparison

| Provider | Result style | Filters | API key |
| --- | --- | --- | --- |
| [Brave](https://docs.openclaw.ai/tools/brave-search) | Structured snippets | Country, language, time, `llm-context` mode | `BRAVE_API_KEY` |
| [DuckDuckGo](https://docs.openclaw.ai/tools/duckduckgo-search) | Structured snippets | — | None (key-free) |
| [Exa](https://docs.openclaw.ai/tools/exa-search) | Structured + extracted | Neural/keyword mode, date, content extraction | `EXA_API_KEY` |
| [Firecrawl](https://docs.openclaw.ai/tools/firecrawl) | Structured snippets | Via `firecrawl_search` tool | `FIRECRAWL_API_KEY` |
| [Gemini](https://docs.openclaw.ai/tools/gemini-search) | AI-synthesized + citations | — | `GEMINI_API_KEY` |
| [Grok](https://docs.openclaw.ai/tools/grok-search) | AI-synthesized + citations | — | `XAI_API_KEY` |
| [Kimi](https://docs.openclaw.ai/tools/kimi-search) | AI-synthesized + citations | — | `KIMI_API_KEY` / `MOONSHOT_API_KEY` |
| [MiniMax Search](https://docs.openclaw.ai/tools/minimax-search) | Structured snippets | Region (`global` / `cn`) | `MINIMAX_CODE_PLAN_KEY` / `MINIMAX_CODING_API_KEY` |
| [Ollama Web Search](https://docs.openclaw.ai/tools/ollama-search) | Structured snippets | — | None for signed-in local hosts; `OLLAMA_API_KEY` for direct `https://ollama.com` search |
| [Perplexity](https://docs.openclaw.ai/tools/perplexity-search) | Structured snippets | Country, language, time, domains, content limits | `PERPLEXITY_API_KEY` / `OPENROUTER_API_KEY` |
| [SearXNG](https://docs.openclaw.ai/tools/searxng-search) | Structured snippets | Categories, language | None (self-hosted) |
| [Tavily](https://docs.openclaw.ai/tools/tavily) | Structured snippets | Via `tavily_search` tool | `TAVILY_API_KEY` |

## [​](https://docs.openclaw.ai/tools/web\\#auto-detection)  Auto-detection

## [​](https://docs.openclaw.ai/tools/web\\#native-openai-web-search)  Native OpenAI web search

Direct OpenAI Responses models use OpenAI’s hosted `web_search` tool automatically when OpenClaw web search is enabled and no managed provider is pinned. This is provider-owned behavior in the bundled OpenAI plugin and only applies to native OpenAI API traffic, not OpenAI-compatible proxy base URLs or Azure routes. Set `tools.web.search.provider` to another provider such as `brave` to keep the managed `web_search` tool for OpenAI models, or set `tools.web.search.enabled: false` to disable both managed search and native OpenAI search.

## [​](https://docs.openclaw.ai/tools/web\\#native-codex-web-search)  Native Codex web search

Codex-capable models can optionally use the provider-native Responses `web_search` tool instead of OpenClaw’s managed `web_search` function.

- Configure it under `tools.web.search.openaiCodex`
- It only activates for Codex-capable models (`openai-codex/*` or providers using `api: \"openai-codex-responses\"`)
- Managed `web_search` still applies to non-Codex models
- `mode: \"cached\"` is the default and recommended setting
- `tools.web.search.enabled: false` disables both managed and native search

```
{
  tools: {
    web: {
      search: {
        enabled: true,
        openaiCodex: {
          enabled: true,
          mode: \"cached\",
          allowedDomains: [\"example.com\"],
          contextSize: \"high\",
          userLocation: {
            country: \"US\",
            city: \"New York\",
            timezone: \"America/New_York\",
          },
        },
      },
    },
  },
}
```

If native Codex search is enabled but the current model is not Codex-capable, OpenClaw keeps the normal managed `web_search` behavior.

## [​](https://docs.openclaw.ai/tools/web\\#setting-up-web-search)  Setting up web search

Provider lists in docs and setup flows are alphabetical. Auto-detection keeps a
separate precedence order.If no `provider` is set, OpenClaw checks providers in this order and uses the
first one that is ready:API-backed providers first:

1. **Brave** — `BRAVE_API_KEY` or `plugins.entries.brave.config.webSearch.apiKey` (order 10)
2. **MiniMax Search** — `MINIMAX_CODE_PLAN_KEY` / `MINIMAX_CODING_API_KEY` or `plugins.entries.minimax.config.webSearch.apiKey` (order 15)
3. **Gemini** — `GEMINI_API_KEY` or `plugins.entries.google.config.webSearch.apiKey` (order 20)
4. **Grok** — `XAI_API_KEY` or `plugins.entries.xai.config.webSearch.apiKey` (order 30)
5. **Kimi** — `KIMI_API_KEY` / `MOONSHOT_API_KEY` or `plugins.entries.moonshot.config.webSearch.apiKey` (order 40)
6. **Perplexity** — `PERPLEXITY_API_KEY` / `OPENROUTER_API_KEY` or `plugins.entries.perplexity.config.webSearch.apiKey` (order 50)
7. **Firecrawl** — `FIRECRAWL_API_KEY` or `plugins.entries.firecrawl.config.webSearch.apiKey` (order 60)
8. **Exa** — `EXA_API_KEY` or `plugins.entries.exa.config.webSearch.apiKey` (order 65)
9. **Tavily** — `TAVILY_API_KEY` or `plugins.entries.tavily.config.webSearch.apiKey` (order 70)

Key-free fallbacks after that:

10. **DuckDuckGo** — key-free HTML fallback with no account or API key (order 100)
11. **Ollama Web Search** — key-free fallback via your configured local Ollama host when it is reachable and signed in with `ollama signin`; can reuse Ollama provider bearer auth when the host needs it, and can call direct `https://ollama.com` search when configured with `OLLAMA_API_KEY` (order 110)
12. **SearXNG** — `SEARXNG_BASE_URL` or `plugins.entries.searxng.config.webSearch.baseUrl` (order 200)

If no provider is detected, it falls back to Brave (you will get a missing-key
error prompting you to configure one).

All provider key fields support SecretRef objects. Plugin-scoped SecretRefs
under `plugins.entries.<plugin>.config.webSearch.apiKey` are resolved for the
bundled API-backed web search providers, including Brave, Exa, Firecrawl,
Gemini, Grok, Kimi, MiniMax, Perplexity, and Tavily,
whether the provider is picked explicitly via `tools.web.search.provider` or
selected through auto-detect. In auto-detect mode, OpenClaw resolves only the
selected provider key — non-selected SecretRefs stay inactive, so you can
keep multiple providers configured without paying resolution cost for the
ones you are not using.

## [​](https://docs.openclaw.ai/tools/web\\#config)  Config

```
{
  tools: {
    web: {
      search: {
        enabled: true, // default: true
        provider: \"brave\", // or omit for auto-detection
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

Provider-specific config (API keys, base URLs, modes) lives under
`plugins.entries.<plugin>.config.webSearch.*`. See the provider pages for
examples.`web_fetch` fallback provider selection is separate:

- choose it with `tools.web.fetch.provider`
- or omit that field and let OpenClaw auto-detect the first ready web-fetch
provider from available credentials
- today the bundled web-fetch provider is Firecrawl, configured under
`plugins.entries.firecrawl.config.webFetch.*`

When you choose **Kimi** during `openclaw onboard` or
`openclaw configure --section web`, OpenClaw can also ask for:

- the Moonshot API region (`https://api.moonshot.ai/v1` or `https://api.moonshot.cn/v1`)
- the default Kimi web-search model (defaults to `kimi-k2.6`)

For `x_search`, configure `plugins.entries.xai.config.xSearch.*`. It uses the
same `XAI_API_KEY` fallback as Grok web search.
Legacy `tools.web.x_search.*` config is auto-migrated by `openclaw doctor --fix`.
When you choose Grok during `openclaw onboard` or `openclaw configure --section web`,
OpenClaw can also offer optional `x_search` setup with the same key.
This is a separate follow-up step inside the Grok path, not a separate top-level
web-search provider choice. If you pick another provider, OpenClaw does not
show the `x_search` prompt.

### [​](https://docs.openclaw.ai/tools/web\\#storing-api-keys)  Storing API keys

- Config file

- Environment variable


Run `openclaw configure --section web` or set the key directly:

```
{
  plugins: {
    entries: {
      brave: {
        config: {
          webSearch: {
            apiKey: \"YOUR_KEY\", // pragma: allowlist secret
          },
        },
      },
    },
  },
}
```

Set the provider env var in the Gateway process environment:

```
export BRAVE_API_KEY=\"YOUR_KEY\"
```

For a gateway install, put it in `~/.openclaw/.env`.
See [Env vars](https://docs.openclaw.ai/help/faq#env-vars-and-env-loading).

## [​](https://docs.openclaw.ai/tools/web\\#tool-parameters)  Tool parameters

| Parameter | Description |
| --- | --- |
| `query` | Search query (required) |
| `count` | Results to return (1-10, default: 5) |
| `country` | 2-letter ISO country code (e.g. “US”, “DE”) |
| `language` | ISO 639-1 language code (e.g. “en”, “de”) |
| `search_lang` | Search-language code (Brave only) |
| `freshness` | Time filter: `day`, `week`, `month`, or `year` |
| `date_after` | Results after this date (YYYY-MM-DD) |
| `date_before` | Results before this date (YYYY-MM-DD) |
| `ui_lang` | UI language code (Brave only) |
| `domain_filter` | Domain allowlist/denylist array (Perplexity only) |
| `max_tokens` | Total content budget, default 25000 (Perplexity only) |
| `max_tokens_per_page` | Per-page token limit, default 2048 (Perplexity only) |

Not all parameters work with all providers. Brave `llm-context` mode
rejects `ui_lang`, `freshness`, `date_after`, and `date_before`.
Gemini, Grok, and Kimi return one synthesized answer with citations. They
accept `count` for shared-tool compatibility, but it does not change the
grounded answer shape.
Perplexity behaves the same way when you use the Sonar/OpenRouter
compatibility path (`plugins.entries.perplexity.config.webSearch.baseUrl` /
`model` or `OPENROUTER_API_KEY`).
SearXNG accepts `http://` only for trusted private-network or loopback hosts;
public SearXNG endpoints must use `https://`.
Firecrawl and Tavily only support `query` and `count` through `web_search`
— use their dedicated tools for advanced options.

## [​](https://docs.openclaw.ai/tools/web\\#x_search)  x\\_search

`x_search` queries X (formerly Twitter) posts using xAI and returns
AI-synthesized answers with citations. It accepts natural-language queries and
optional structured filters. OpenClaw only enables the built-in xAI `x_search`
tool on the request that serves this tool call.

xAI documents `x_search` as supporting keyword search, semantic search, user
search, and thread fetch. For per-post engagement stats such as reposts,
replies, bookmarks, or views, prefer a targeted lookup for the exact post URL
or status ID. Broad keyword searches may find the right post but return less
complete per-post metadata. A good pattern is: locate the post first, then
run a second `x_search` query focused on that exact post.

### [​](https://docs.openclaw.ai/tools/web\\#x_search-config)  x\\_search config

```
{
  plugins: {
    entries: {
      xai: {
        config: {
          xSearch: {
            enabled: true,
            model: \"grok-4-1-fast-non-reasoning\",
            inlineCitations: false,
            maxTurns: 2,
            timeoutSeconds: 30,
            cacheTtlMinutes: 15,
          },
          webSearch: {
            apiKey: \"xai-...\", // optional if XAI_API_KEY is set
          },
        },
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/tools/web\\#x_search-parameters)  x\\_search parameters

| Parameter | Description |
| --- | --- |
| `query` | Search query (required) |
| `allowed_x_handles` | Restrict results to specific X handles |
| `excluded_x_handles` | Exclude specific X handles |
| `from_date` | Only include posts on or after this date (YYYY-MM-DD) |
| `to_date` | Only include posts on or before this date (YYYY-MM-DD) |
| `enable_image_understanding` | Let xAI inspect images attached to matching posts |
| `enable_video_understanding` | Let xAI inspect videos attached to matching posts |

### [​](https://docs.openclaw.ai/tools/web\\#x_search-example)  x\\_search example

```
await x_search({
  query: \"dinner recipes\",
  allowed_x_handles: [\"nytfood\"],
  from_date: \"2026-03-01\",
});
```

```
// Per-post stats: use the exact status URL or status ID when possible
await x_search({
  query: \"https://x.com/huntharo/status/1905678901234567890\",
});
```

## [​](https://docs.openclaw.ai/tools/web\\#examples)  Examples

```
// Basic search
await web_search({ query: \"OpenClaw plugin SDK\" });

// German-specific search
await web_search({ query: \"TV online schauen\", country: \"DE\", language: \"de\" });

// Recent results (past week)
await web_search({ query: \"AI developments\", freshness: \"week\" });

// Date range
await web_search({
  query: \"climate research\",
  date_after: \"2024-01-01\",
  date_before: \"2024-06-30\",
});

// Domain filtering (Perplexity only)
await web_search({
  query: \"product reviews\",
  domain_filter: [\"-reddit.com\", \"-pinterest.com\"],
});
```

## [​](https://docs.openclaw.ai/tools/web\\#tool-profiles)  Tool profiles

If you use tool profiles or allowlists, add `web_search`, `x_search`, or `group:web`:

```
{
  tools: {
    allow: [\"web_search\", \"x_search\"],
    // or: allow: [\"group:web\"]  (includes web_search, x_search, and web_fetch)
  },
}
```

## [​](https://docs.openclaw.ai/tools/web\\#related)  Related

- [Web Fetch](https://docs.openclaw.ai/tools/web-fetch) — fetch a URL and extract readable content
- [Web Browser](https://docs.openclaw.ai/tools/browser) — full browser automation for JS-heavy sites
- [Grok Search](https://docs.openclaw.ai/tools/grok-search) — Grok as the `web_search` ...

---

## PDF tool - OpenClaw
**Source:** https://docs.openclaw.ai/tools/pdf

[Skip to main content](https://docs.openclaw.ai/tools/pdf#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

PDF tool

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Availability](https://docs.openclaw.ai/tools/pdf#availability)
- [Input reference](https://docs.openclaw.ai/tools/pdf#input-reference)
- [Supported PDF references](https://docs.openclaw.ai/tools/pdf#supported-pdf-references)
- [Execution modes](https://docs.openclaw.ai/tools/pdf#execution-modes)
- [Native provider mode](https://docs.openclaw.ai/tools/pdf#native-provider-mode)
- [Extraction fallback mode](https://docs.openclaw.ai/tools/pdf#extraction-fallback-mode)
- [Config](https://docs.openclaw.ai/tools/pdf#config)
- [Output details](https://docs.openclaw.ai/tools/pdf#output-details)
- [Error behavior](https://docs.openclaw.ai/tools/pdf#error-behavior)
- [Examples](https://docs.openclaw.ai/tools/pdf#examples)
- [Related](https://docs.openclaw.ai/tools/pdf#related)

`pdf` analyzes one or more PDF documents and returns text.Quick behavior:

- Native provider mode for Anthropic and Google model providers.
- Extraction fallback mode for other providers (extract text first, then page images when needed).
- Supports single (`pdf`) or multi (`pdfs`) input, max 10 PDFs per call.

## [​](https://docs.openclaw.ai/tools/pdf\\#availability)  Availability

The tool is only registered when OpenClaw can resolve a PDF-capable model config for the agent:

1. `agents.defaults.pdfModel`
2. fallback to `agents.defaults.imageModel`
3. fallback to the agent’s resolved session/default model
4. if native-PDF providers are auth-backed, prefer them ahead of generic image fallback candidates

If no usable model can be resolved, the `pdf` tool is not exposed.Availability notes:

- The fallback chain is auth-aware. A configured `provider/model` only counts if
OpenClaw can actually authenticate that provider for the agent.
- Native PDF providers are currently **Anthropic** and **Google**.
- If the resolved session/default provider already has a configured vision/PDF
model, the PDF tool reuses that before falling back to other auth-backed
providers.

## [​](https://docs.openclaw.ai/tools/pdf\\#input-reference)  Input reference

[​](https://docs.openclaw.ai/tools/pdf#param-pdf)

pdf

string

One PDF path or URL.

[​](https://docs.openclaw.ai/tools/pdf#param-pdfs)

pdfs

string\[\]

Multiple PDF paths or URLs, up to 10 total.

[​](https://docs.openclaw.ai/tools/pdf#param-prompt)

prompt

string

default:\"Analyze this PDF document.\"

Analysis prompt.

[​](https://docs.openclaw.ai/tools/pdf#param-pages)

pages

string

Page filter like `1-5` or `1,3,7-9`.

[​](https://docs.openclaw.ai/tools/pdf#param-model)

model

string

Optional model override in `provider/model` form.

[​](https://docs.openclaw.ai/tools/pdf#param-max-bytes-mb)

maxBytesMb

number

Per-PDF size cap in MB. Defaults to `agents.defaults.pdfMaxBytesMb` or `10`.

Input notes:

- `pdf` and `pdfs` are merged and deduplicated before loading.
- If no PDF input is provided, the tool errors.
- `pages` is parsed as 1-based page numbers, deduped, sorted, and clamped to the configured max pages.
- `maxBytesMb` defaults to `agents.defaults.pdfMaxBytesMb` or `10`.

## [​](https://docs.openclaw.ai/tools/pdf\\#supported-pdf-references)  Supported PDF references

- local file path (including `~` expansion)
- `file://` URL
- `http://` and `https://` URL
- OpenClaw-managed inbound refs such as `media://inbound/<id>`

Reference notes:

- Other URI schemes (for example `ftp://`) are rejected with `unsupported_pdf_reference`.
- In sandbox mode, remote `http(s)` URLs are rejected.
- With workspace-only file policy enabled, local file paths outside allowed roots are rejected.
- Managed inbound refs and replayed paths under OpenClaw’s inbound media store are allowed with workspace-only file policy.

## [​](https://docs.openclaw.ai/tools/pdf\\#execution-modes)  Execution modes

### [​](https://docs.openclaw.ai/tools/pdf\\#native-provider-mode)  Native provider mode

Native mode is used for provider `anthropic` and `google`.
The tool sends raw PDF bytes directly to provider APIs.Native mode limits:

- `pages` is not supported. If set, the tool returns an error.
- Multi-PDF input is supported; each PDF is sent as a native document block /
inline PDF part before the prompt.

### [​](https://docs.openclaw.ai/tools/pdf\\#extraction-fallback-mode)  Extraction fallback mode

Fallback mode is used for non-native providers.Flow:

1. Extract text from selected pages (up to `agents.defaults.pdfMaxPages`, default `20`).
2. If extracted text length is below `200` chars, render selected pages to PNG images and include them.
3. Send extracted content plus prompt to the selected model.

Fallback details:

- Page image extraction uses a pixel budget of `4,000,000`.
- If the target model does not support image input and there is no extractable text, the tool errors.
- If text extraction succeeds but image extraction would require vision on a\ntext-only model, OpenClaw drops the rendered images and continues with the\nextracted text.
- Extraction fallback uses the bundled `document-extract` plugin. The plugin owns\n`pdfjs-dist`; `@napi-rs/canvas` is used only when image rendering fallback is\navailable.

## [​](https://docs.openclaw.ai/tools/pdf\\#config)  Config

```
{
  agents: {
    defaults: {
      pdfModel: {
        primary: \"anthropic/claude-opus-4-6\",\n        fallbacks: [\"openai/gpt-5.4-mini\"],
      },
      pdfMaxBytesMb: 10,
      pdfMaxPages: 20,
    },
  },
}
```

See [Configuration Reference](https://docs.openclaw.ai/gateway/configuration-reference) for full field details.

## [​](https://docs.openclaw.ai/tools/pdf\\#output-details)  Output details

The tool returns text in `content[0].text` and structured metadata in `details`.Common `details` fields:

- `model`: resolved model ref (`provider/model`)
- `native`: `true` for native provider mode, `false` for fallback
- `attempts`: fallback attempts that failed before success

Path fields:

- single PDF input: `details.pdf`
- multiple PDF inputs: `details.pdfs[]` with `pdf` entries
- sandbox path rewrite metadata (when applicable): `rewrittenFrom`

## [​](https://docs.openclaw.ai/tools/pdf\\#error-behavior)  Error behavior

- Missing PDF input: throws `pdf required: provide a path or URL to a PDF document`
- Too many PDFs: returns structured error in `details.error = \"too_many_pdfs\"`
- Unsupported reference scheme: returns `details.error = \"unsupported_pdf_reference\"`
- Native mode with `pages`: throws clear `pages is not supported with native PDF providers` error

## [​](https://docs.openclaw.ai/tools/pdf\\#examples)  Examples

Single PDF:

```
{
  \"pdf\": \"/tmp/report.pdf\",\n  \"prompt\": \"Summarize this report in 5 bullets\"
}
```

Multiple PDFs:

```
{
  \"pdfs\": [\"/tmp/q1.pdf\", \"/tmp/q2.pdf\"],
  \"prompt\": \"Compare risks and timeline changes across both documents\"
}
```

Page-filtered fallback model:

```
{
  \"pdf\": \"https://example.com/report.pdf\",\n  \"pages\": \"1-3,7\",\n  \"model\": \"openai/gpt-5.4-mini\",\n  \"prompt\": \"Extract only customer-impacting incidents\"
}
```

## [​](https://docs.openclaw.ai/tools/pdf\\#related)  Related

- [Tools Overview](https://docs.openclaw.ai/tools) — all available agent tools
- [Configuration Reference](https://docs.openclaw.ai/gateway/configuration-reference) — pdfMaxBytesMb and pdfMaxPages config

[Music generation](https://docs.openclaw.ai/tools/music-generation) [Reactions](https://docs.openclaw.ai/tools/reactions)

Ctrl+I

---

## Exec approvals — advanced
**Source:** https://docs.openclaw.ai/tools/exec-approvals-advanced

[Skip to main content](https://docs.openclaw.ai/tools/exec-approvals-advanced#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

Exec approvals — advanced

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Safe bins (stdin-only)](https://docs.openclaw.ai/tools/exec-approvals-advanced#safe-bins-stdin-only)
- [Argv validation and denied flags](https://docs.openclaw.ai/tools/exec-approvals-advanced#argv-validation-and-denied-flags)
- [Trusted binary directories](https://docs.openclaw.ai/tools/exec-approvals-advanced#trusted-binary-directories)
- [Shell chaining, wrappers, and multiplexers](https://docs.openclaw.ai/tools/exec-approvals-advanced#shell-chaining-wrappers-and-multiplexers)
- [Safe bins versus allowlist](https://docs.openclaw.ai/tools/exec-approvals-advanced#safe-bins-versus-allowlist)
- [Interpreter/runtime commands](https://docs.openclaw.ai/tools/exec-approvals-advanced#interpreter%2Fruntime-commands)
- [Followup delivery behavior](https://docs.openclaw.ai/tools/exec-approvals-advanced#followup-delivery-behavior)
- [Approval forwarding to chat channels](https://docs.openclaw.ai/tools/exec-approvals-advanced#approval-forwarding-to-chat-channels)
- [Plugin approval forwarding](https://docs.openclaw.ai/tools/exec-approvals-advanced#plugin-approval-forwarding)
- [Same-chat approvals on any channel](https://docs.openclaw.ai/tools/exec-approvals-advanced#same-chat-approvals-on-any-channel)
- [Native approval delivery](https://docs.openclaw.ai/tools/exec-approvals-advanced#native-approval-delivery)
- [macOS IPC flow](https://docs.openclaw.ai/tools/exec-approvals-advanced#macos-ipc-flow)
- [Related](https://docs.openclaw.ai/tools/exec-approvals-advanced#related)

Advanced exec-approval topics: the `safeBins` fast-path, interpreter/runtime\nbinding, and approval-forwarding to chat channels (including native delivery).\nFor the core policy and approval flow, see [Exec approvals](https://docs.openclaw.ai/tools/exec-approvals).\n
## [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#safe-bins-stdin-only)  Safe bins (stdin-only)

`tools.exec.safeBins` defines a small list of **stdin-only** binaries (for\nexample `cut`) that can run in allowlist mode **without** explicit allowlist\nentries. Safe bins reject positional file args and path-like tokens, so they\ncan only operate on the incoming stream. Treat this as a narrow fast-path for\nstream filters, not a general trust list.\n
Do **not** add interpreter or runtime binaries (for example `python3`, `node`,\n`ruby`, `bash`, `sh`, `zsh`) to `safeBins`. If a command can evaluate code,\nexecute subcommands, or read files by design, prefer explicit allowlist entries\nand keep approval prompts enabled. Custom safe bins must define an explicit\nprofile in `tools.exec.safeBinProfiles.<bin>`.\n
Default safe bins:`cut`, `uniq`, `head`, `tail`, `tr`, `wc``grep` and `sort` are not in the default list. If you opt in, keep explicit\nallowlist entries for their non-stdin workflows. For `grep` in safe-bin mode,\nprovide the pattern with `-e`/`--regexp`; positional pattern form is rejected\nso file operands cannot be smuggled as ambiguous positionals.\n
### [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#argv-validation-and-denied-flags)  Argv validation and denied flags\n
Validation is deterministic from argv shape only (no host filesystem existence\nchecks), which prevents file-existence oracle behavior from allow/deny\ndifferences. File-oriented options are denied for default safe bins; long\noptions are validated fail-closed (unknown flags and ambiguous abbreviations are\nrejected).Denied flags by safe-bin profile:\n
- `grep`: `--dereference-recursive`, `--directories`, `--exclude-from`, `--file`, `--recursive`, `-R`, `-d`, `-f`, `-r`\n- `jq`: `--argfile`, `--from-file`, `--library-path`, `--rawfile`, `--slurpfile`, `-L`, `-f`\n- `sort`: `--compress-program`, `--files0-from`, `--output`, `--random-source`, `--temporary-directory`, `-T`, `-o`\n- `wc`: `--files0-from`\n
Safe bins also force argv tokens to be treated as **literal text** at execution\ntime (no globbing and no `$VARS` expansion) for stdin-only segments, so patterns\nlike `*` or `$HOME/...` cannot be used to smuggle file reads.\n
### [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#trusted-binary-directories)  Trusted binary directories\n
Safe bins must resolve from trusted binary directories (system defaults plus\noptional `tools.exec.safeBinTrustedDirs`). `PATH` entries are never auto-trusted.\nDefault trusted directories are intentionally minimal: `/bin`, `/usr/bin`. If\nyour safe-bin executable lives in package-manager/user paths (for example\n`/opt/homebrew/bin`, `/usr/local/bin`, `/opt/local/bin`, `/snap/bin`), add them\nexplicitly to `tools.exec.safeBinTrustedDirs`.\n
### [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#shell-chaining-wrappers-and-multiplexers)  Shell chaining, wrappers, and multiplexers\n
Shell chaining (`&&`, `||`, `;`) is allowed when every top-level segment\nsatisfies the allowlist (including safe bins or skill auto-allow). Redirections\nremain unsupported in allowlist mode. Command substitution (`$()` / backticks) is\nrejected during allowlist parsing, including inside double quotes; use single\nquotes if you need literal `$()` text.On macOS companion-app approvals, raw shell text containing shell control or\nexpansion syntax (`&&`, `||`, `;`, `|`, `````, `$`, `<`, `>`, `(`, `)`) is\ntreated as an allowlist miss unless the shell binary itself is allowlisted.For shell wrappers (`bash|sh|zsh ... -c/-lc`), request-scoped env overrides are\nreduced to a small explicit allowlist (`TERM`, `LANG`, `LC_*`, `COLORTERM`,\n`NO_COLOR`, `FORCE_COLOR`).For `allow-always` decisions in allowlist mode, known dispatch wrappers (`env`,\n`nice`, `nohup`, `stdbuf`, `timeout`) persist the inner executable path instead\nof the wrapper path. Shell multiplexers (`busybox`, `toybox`) are unwrapped for\nshell applets (`sh`, `ash`, etc.) the same way. If a wrapper or multiplexer\ncannot be safely unwrapped, no allowlist entry is persisted automatically.If you allowlist interpreters like `python3` or `node`, prefer\n`tools.exec.strictInlineEval=true` so inline eval still requires an explicit\napproval. In strict mode, `allow-always` can still persist benign\ninterpreter/script invocations, but inline-eval carriers are not persisted\nautomatically.\n
### [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#safe-bins-versus-allowlist)  Safe bins versus allowlist\n
| Topic | `tools.exec.safeBins` | Allowlist (`exec-approvals.json`) |\n| --- | --- | --- |\n| Goal | Auto-allow narrow stdin filters | Explicitly trust specific executables |\n| Match type | Executable name + safe-bin argv policy | Resolved executable path glob, or bare command-name glob for PATH-invoked commands |\n| Argument scope | Restricted by safe-bin profile and literal-token rules | Path match only; arguments are otherwise your responsibility |\n| Typical examples | `head`, `tail`, `tr`, `wc` | `jq`, `python3`, `node`, `ffmpeg`, custom CLIs |\n| Best use | Low-risk text transforms in pipelines | Any tool with broader behavior or side effects |\n
Configuration location:\n
- `safeBins` comes from config (`tools.exec.safeBins` or per-agent `agents.list[].tools.exec.safeBins`).\n- `safeBinTrustedDirs` comes from config (`tools.exec.safeBinTrustedDirs` or per-agent `agents.list[].tools.exec.safeBinTrustedDirs`).\n- `safeBinProfiles` comes from config (`tools.exec.safeBinProfiles` or per-agent `agents.list[].tools.exec.safeBinProfiles`). Per-agent profile keys override global keys.\n- allowlist entries live in host-local `~/.openclaw/exec-approvals.json` under `agents.<id>.allowlist` (or via Control UI / `openclaw approvals allowlist ...`).\n- `openclaw security audit` warns with `tools.exec.safe_bins_interpreter_unprofiled` when interpreter/runtime bins appear in `safeBins` without explicit profiles.\n- `openclaw doctor --fix` can scaffold missing custom `safeBinProfiles.<bin>` entries as `{}` (review and tighten afterward). Interpreter/runtime bins are not auto-scaffolded.\n
Custom profile example:\n
```\n{\n  tools: {\n    exec: {\n      safeBins: [\"jq\", \"myfilter\"],\n      safeBinProfiles: {\n        myfilter: {\n          minPositional: 0,\n          maxPositional: 0,\n          allowedValueFlags: [\"-n\", \"--limit\"],\n          deniedFlags: [\"-f\", \"--file\", \"-c\", \"--command\"],\n        },\n      },\n    },\n  },\n}\n```\n
If you explicitly opt `jq` into `safeBins`, OpenClaw still rejects the `env` builtin in safe-bin\nmode so `jq -n env` cannot dump the host process environment without an explicit allowlist path\nor approval prompt.\n
## [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#interpreter/runtime-commands)  Interpreter/runtime commands\n
Approval-backed interpreter/runtime runs are intentionally conservative:\n
- Exact argv/cwd/env context is always bound.\n- Direct shell script and direct runtime file forms are best-effort bound to one concrete local\nfile snapshot.\n- Common package-manager wrapper forms that still resolve to one direct local file (for example\npnpm exec, pnpm node, npm exec, npx) are unwrapped before binding.\n- If OpenClaw cannot identify exactly one concrete local file for an interpreter/runtime command\n(for example package scripts, eval forms, runtime-specific loader chains, or ambiguous multi-file\nforms), approval-backed execution is denied instead of claiming semantic coverage it does not\nhave.\n- For those workflows, prefer sandboxing, a separate host boundary, or an explicit trusted\nallowlist/full workflow where the operator accepts the broader runtime semantics.\n
When approvals are required, the exec tool returns immediately with an approval id. Use that id to\ncorrelate later system events (`Exec finished` / `Exec denied`). If no decision arrives before the\ntimeout, the request is treated as an approval timeout and surfaced as a denial reason.\n
### [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#followup-delivery-behavior)  Followup delivery behavior\n
After an approved async exec finishes, OpenClaw sends a followup `agent` turn to the same session.\n
- If a valid external delivery target exists (deliverable channel plus target `to`), followup delivery uses that channel.\n- In webchat-only or internal-session flows with no external target, followup delivery stays session-only (`deliver: false`).\n- If a caller explicitly requests strict external delivery with no resolvable external channel, the request fails with `INVALID_REQUEST`.\n- If `bestEffortDeliver` is enabled and no external channel can be resolved, delivery is downgraded to session-only instead of failing.\n
## [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#approval-forwarding-to-chat-channels)  Approval forwarding to chat channels\n
You can forward exec approval prompts to any chat channel (including plugin channels) and approve\nthem with `/approve`. This uses the normal outbound delivery pipeline.Config:\n
```\n{\n  approvals: {\n    exec: {\n      enabled: true,\n      mode: \"session\", // \"session\" | \"targets\" | \"both\"\n      agentFilter: [\"main\"],\n      sessionFilter: [\"discord\"], // substring or regex\n      targets: [\\\n        { channel: \"slack\", to: \"U12345678\" },\\\n        { channel: \"telegram\", to: \"123456789\" },\\\n      ],\n    },\n  },\n}\n```\n
Reply in chat:\n
```\n/approve <id> allow-once\n/approve <id> allow-always\n/approve <id> deny\n```\n
The `/approve` command handles both exec approvals and plugin approvals. If the ID does not match a pending exec approval, it automatically checks plugin approvals instead.\n
### [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#plugin-approval-forwarding)  Plugin approval forwarding\n
Plugin approval forwarding uses the same delivery pipeline as exec approvals but has its own\nindependent config under `approvals.plugin`. Enabling or disabling one does not affect the other.\n
```\n{\n  approvals: {\n    plugin: {\n      enabled: true,\n      mode: \"targets\",\n      agentFilter: [\"main\"],\n      targets: [\\\n        { channel: \"slack\", to: \"U12345678\" },\\\n        { channel: \"telegram\", to: \"123456789\" },\\\n      ],\n    },\n  },\n}\n```\n
The config shape is identical to `approvals.exec`: `enabled`, `mode`, `agentFilter`,\n`sessionFilter`, and `targets` work the same way.Channels that support shared interactive replies render the same approval buttons for both exec and\nplugin approvals. Channels without shared interactive UI fall back to plain text with `/approve`\ninstructions.\n
### [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#same-chat-approvals-on-any-channel)  Same-chat approvals on any channel\n
When an exec or plugin approval request originates from a deliverable chat surface, the same chat\ncan now approve it with `/approve` by default. This applies to channels such as Slack, Matrix, and\nMicrosoft Teams in addition to the existing Web UI and terminal UI flows.This shared text-command path uses the normal channel auth model for that conversation. If the\noriginating chat can already send commands and receive replies, approval requests no longer need a\nseparate native delivery adapter just to stay pending.Discord and Telegram also support same-chat `/approve`, but those channels still use their\nresolved approver list for authorization even when native approval delivery is disabled.For Telegram and other native approval clients that call the Gateway directly,\nthis fallback is intentionally bounded to “approval not found” failures. A real\nexec approval denial/error does not silently retry as a plugin approval.\n
### [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#native-approval-delivery)  Native approval delivery\n
Some channels can also act as native approval clients. Native clients add approver DMs, origin-chat\nfanout, and channel-specific interactive approval UX on top of the shared same-chat `/approve`\nflow.When native approval cards/buttons are available, that native UI is the primary\nagent-facing path. The agent should not also echo a duplicate plain chat\n`/approve` command unless the tool result says chat approvals are unavailable or\nmanual approval is the only remaining path.Generic model:\n
- host exec policy still decides whether exec approval is required\n- `approvals.exec` controls forwarding approval prompts to other chat destinations\n- `channels.<channel>.execApprovals` controls whether that channel acts as a native approval client\n
Native approval clients auto-enable DM-first delivery when all of these are true:\n
- the channel supports native approval delivery\n- approvers can be resolved from explicit `execApprovals.approvers` or that\nchannel’s documented fallback sources\n- `channels.<channel>.execApprovals.enabled` is unset or `\"auto\"`\n
Set `enabled: false` to disable a native approval client explicitly. Set `enabled: true` to force\nit on when approvers resolve. Public origin-chat delivery stays explicit through\n`channels.<channel>.execApprovals.target`.FAQ: [Why are there two exec approval configs for chat approvals?](https://docs.openclaw.ai/help/faq-first-run#why-are-there-two-exec-approval-configs-for-chat-approvals)\n
- Discord: `channels.discord.execApprovals.*`\n- Slack: `channels.slack.execApprovals.*`\n- Telegram: `channels.telegram.execApprovals.*`\n
These native approval clients add DM routing and optional channel fanout on top of the shared\nsame-chat `/approve` flow and shared approval buttons.Shared behavior:\n
- Slack, Matrix, Microsoft Teams, and similar deliverable chats use the normal channel auth model\nfor same-chat `/approve`\n- when a native approval client auto-enables, the default native delivery target is approver DMs\n- for Discord and Telegram, only resolved approvers can approve or deny\n- Discord approvers can be explicit (`execApprovals.approvers`) or inferred from `commands.ownerAllowFrom`\n- Telegram approvers can be explicit (`execApprovals.approvers`) or inferred from existing owner config (`allowFrom`, plus direct-message `defaultTo` where supported)\n- Slack approvers can be explicit (`execApprovals.approvers`) or inferred from `commands.ownerAllowFrom`\n- Slack native buttons preserve approval id kind, so `plugin:` ids can resolve plugin approvals\nwithout a second Slack-local fallback layer\n- Matrix native DM/channel routing and reaction shortcuts handle both exec and plugin approvals;\nplugin authorization still comes from `channels.matrix.dm.allowFrom`\n- the requester does not need to be an approver\n- the originating chat can approve directly with `/approve` when that chat already supports commands and replies\n- native Discord approval buttons route by approval id kind: `plugin:` ids go\nstraight to plugin approvals, everything else goes to exec approvals\n- native Telegram approval buttons follow the same bounded exec-to-plugin fallback as `/approve`\n- when native `target` enables origin-chat delivery, approval prompts include the command text\n- pending exec approvals expire after 30 minutes by default\n- if no operator UI or configured approval client can accept the request, the prompt falls back to `askFallback`\n
Telegram defaults to approver DMs (`target: \"dm\"`). You can switch to `channel` or `both` when you\nwant approval prompts to appear in the originating Telegram chat/topic as well. For Telegram forum\ntopics, OpenClaw preserves the topic for the approval prompt and the post-approval follow-up.See:\n
- [Discord](https://docs.openclaw.ai/channels/discord)\n- [Telegram](https://docs.openclaw.ai/channels/telegram)\n
### [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#macos-ipc-flow)  macOS IPC flow\n
```\nGateway -> Node Service (WS)\n                 |  IPC (UDS + token + HMAC + TTL)\n                 v\n             Mac App (UI + approvals + system.run)\n```\n
Security notes:\n
- Unix socket mode `0600`, token stored in `exec-approvals.json`.\n- Same-UID peer check.\n- Challenge/response (nonce + HMAC token + request hash) + short TTL.\n
## [​](https://docs.openclaw.ai/tools/exec-approvals-advanced\\#related)  Related\n
- [Exec approvals](https://docs.openclaw.ai/tools/exec-approvals) — core policy and approval flow\n- [Exec tool](https://docs.openclaw.ai/tools/exec)\n- [Elevated mode](https://docs.openclaw.ai/tools/elevated)\n- [Skills](https://docs.openclaw.ai/tools/skills) — skill-backed auto-allow behavior\n
[Exec approvals](https://docs.openclaw.ai/tools/exec-approvals) [E...(content truncated)

---

## Elevated mode - OpenClaw
**Source:** https://docs.openclaw.ai/tools/elevated

[Skip to main content](https://docs.openclaw.ai/tools/elevated#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

Elevated mode

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Directives](https://docs.openclaw.ai/tools/elevated#directives)
- [How it works](https://docs.openclaw.ai/tools/elevated#how-it-works)
- [Resolution order](https://docs.openclaw.ai/tools/elevated#resolution-order)
- [Availability and allowlists](https://docs.openclaw.ai/tools/elevated#availability-and-allowlists)
- [What elevated does not control](https://docs.openclaw.ai/tools/elevated#what-elevated-does-not-control)
- [Related](https://docs.openclaw.ai/tools/elevated#related)

When an agent runs inside a sandbox, its `exec` commands are confined to the
sandbox environment. **Elevated mode** lets the agent break out and run commands
outside the sandbox instead, with configurable approval gates.

Elevated mode only changes behavior when the agent is **sandboxed**. For
unsandboxed agents, exec already runs on the host.

## [​](https://docs.openclaw.ai/tools/elevated\\#directives)  Directives

Control elevated mode per-session with slash commands:

| Directive | What it does |
| --- | --- |
| `/elevated on` | Run outside the sandbox on the configured host path, keep approvals |
| `/elevated ask` | Same as `on` (alias) |
| `/elevated full` | Run outside the sandbox on the configured host path and skip approvals |
| `/elevated off` | Return to sandbox-confined execution |

Also available as `/elev on|off|ask|full`.Send `/elevated` with no argument to see the current level.

## [​](https://docs.openclaw.ai/tools/elevated\\#how-it-works)  How it works

1

[Navigate to header](https://docs.openclaw.ai/tools/elevated#)

Check availability

Elevated must be enabled in config and the sender must be on the allowlist:

```
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        discord: [\"user-id-123\"],
        whatsapp: [\"+15555550123\"],
      },
    },
  },
}
```

2

[Navigate to header](https://docs.openclaw.ai/tools/elevated#)

Set the level

Send a directive-only message to set the session default:

```
/elevated full
```

Or use it inline (applies to that message only):

```
/elevated on run the deployment script
```

3

[Navigate to header](https://docs.openclaw.ai/tools/elevated#)

Commands run outside the sandbox

With elevated active, `exec` calls leave the sandbox. The effective host is
`gateway` by default, or `node` when the configured/session exec target is
`node`. In `full` mode, exec approvals are skipped. In `on`/`ask` mode,
configured approval rules still apply.

## [​](https://docs.openclaw.ai/tools/elevated\\#resolution-order)  Resolution order

1. **Inline directive** on the message (applies only to that message)
2. **Session override** (set by sending a directive-only message)
3. **Global default** (`agents.defaults.elevatedDefault` in config)

## [​](https://docs.openclaw.ai/tools/elevated\\#availability-and-allowlists)  Availability and allowlists

- **Global gate**: `tools.elevated.enabled` (must be `true`)
- **Sender allowlist**: `tools.elevated.allowFrom` with per-channel lists
- **Per-agent gate**: `agents.list[].tools.elevated.enabled` (can only further restrict)
- **Per-agent allowlist**: `agents.list[].tools.elevated.allowFrom` (sender must match both global + per-agent)
- **Discord fallback**: if `tools.elevated.allowFrom.discord` is omitted, `channels.discord.allowFrom` is used as fallback
- **All gates must pass**; otherwise elevated is treated as unavailable

Allowlist entry formats:

| Prefix | Matches |
| --- | --- |
| (none) | Sender ID, E.164, or From field |
| `name:` | Sender display name |
| `username:` | Sender username |
| `tag:` | Sender tag |
| `id:`, `from:`, `e164:` | Explicit identity targeting |

## [​](https://docs.openclaw.ai/tools/elevated\\#what-elevated-does-not-control)  What elevated does not control

- **Tool policy**: if `exec` is denied by tool policy, elevated cannot override it
- **Host selection policy**: elevated does not turn `auto` into a free cross-host override. It uses the configured/session exec target rules, choosing `node` only when the target is already `node`.
- **Separate from `/exec`**: the `/exec` directive adjusts per-session exec defaults for authorized senders and does not require elevated mode

## [​](https://docs.openclaw.ai/tools/elevated\\#related)  Related

- [Exec tool](https://docs.openclaw.ai/tools/exec) — shell command execution
- [Exec approvals](https://docs.openclaw.ai/tools/exec-approvals) — approval and allowlist system
- [Sandboxing](https://docs.openclaw.ai/gateway/sandboxing) — sandbox configuration
- [Sandbox vs Tool Policy vs Elevated](https://docs.openclaw.ai/gateway/sandbox-vs-tool-policy-vs-elevated)

[Diffs](https://docs.openclaw.ai/tools/diffs) [Exec approvals](https://docs.openclaw.ai/tools/exec-approvals)

Ctrl+I

---

## Skills config - OpenClaw
**Source:** https://docs.openclaw.ai/tools/skills-config

[Skip to main content](https://docs.openclaw.ai/tools/skills-config#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Skills

Skills config

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Agent skill allowlists](https://docs.openclaw.ai/tools/skills-config#agent-skill-allowlists)
- [Fields](https://docs.openclaw.ai/tools/skills-config#fields)
- [Notes](https://docs.openclaw.ai/tools/skills-config#notes)
- [Sandboxed skills + env vars](https://docs.openclaw.ai/tools/skills-config#sandboxed-skills-%2B-env-vars)
- [Related](https://docs.openclaw.ai/tools/skills-config#related)

Most skills loader/install configuration lives under `skills` in
`~/.openclaw/openclaw.json`. Agent-specific skill visibility lives under
`agents.defaults.skills` and `agents.list[].skills`.

```
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway runtime still Node; bun not recommended)
    },
    entries: {
      "image-lab": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // or plaintext string
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

For built-in image generation/editing, prefer `agents.defaults.imageGenerationModel`
plus the core `image_generate` tool. `skills.entries.*` is only for custom or
third-party skill workflows.If you select a specific image provider/model, also configure that provider’s
auth/API key. Typical examples: `GEMINI_API_KEY` or `GOOGLE_API_KEY` for
`google/*`, `OPENAI_API_KEY` for `openai/*`, and `FAL_KEY` for `fal/*`.Examples:

- Native Nano Banana Pro-style setup: `agents.defaults.imageGenerationModel.primary: "google/gemini-3-pro-image-preview"`
- Native fal setup: `agents.defaults.imageGenerationModel.primary: "fal/fal-ai/flux/dev"`

## [​](https://docs.openclaw.ai/tools/skills-config\\#agent-skill-allowlists)  Agent skill allowlists

Use agent config when you want the same machine/workspace skill roots, but a
different visible skill set per agent.

```
{
  agents: {
    defaults: {
      skills: ["github", "weather"],
    },
    list: [\\
      { id: "writer" }, // inherits defaults -> github, weather\\
      { id: "docs", skills: ["docs-search"] }, // replaces defaults\\
      { id: "locked-down", skills: [] }, // no skills\\
    ],
  },
}
```

Rules:

- `agents.defaults.skills`: shared baseline allowlist for agents that omit
`agents.list[].skills`.
- Omit `agents.defaults.skills` to leave skills unrestricted by default.
- `agents.list[].skills`: explicit final skill set for that agent; it does not
merge with defaults.
- `agents.list[].skills: []`: expose no skills for that agent.

## [​](https://docs.openclaw.ai/tools/skills-config\\#fields)  Fields

- Built-in skill roots always include `~/.openclaw/skills`, `~/.agents/skills`,
`<workspace>/.agents/skills`, and `<workspace>/skills`.
- `allowBundled`: optional allowlist for **bundled** skills only. When set, only
bundled skills in the list are eligible (managed, agent, and workspace skills unaffected).
- `load.extraDirs`: additional skill directories to scan (lowest precedence).
- `load.watch`: watch skill folders and refresh the skills snapshot (default: true).
- `load.watchDebounceMs`: debounce for skill watcher events in milliseconds (default: 250).
- `install.preferBrew`: prefer brew installers when available (default: true).
- `install.nodeManager`: node installer preference (`npm` \\| `pnpm` \\| `yarn` \\| `bun`, default: npm).
This only affects **skill installs**; the Gateway runtime should still be Node
(Bun not recommended for WhatsApp/Telegram).

  - `openclaw setup --node-manager` is narrower and currently accepts `npm`,
    `pnpm`, or `bun`. Set `skills.install.nodeManager: "yarn"` manually if you
    want Yarn-backed skill installs.
- `entries.<skillKey>`: per-skill overrides.
- `agents.defaults.skills`: optional default skill allowlist inherited by agents
that omit `agents.list[].skills`.
- `agents.list[].skills`: optional per-agent final skill allowlist; explicit
lists replace inherited defaults instead of merging.

Per-skill fields:

- `enabled`: set `false` to disable a skill even if it’s bundled/installed.
- `env`: environment variables injected for the agent run (only if not already set).
- `apiKey`: optional convenience for skills that declare a primary env var.
Supports plaintext string or SecretRef object (`{ source, provider, id }`).

## [​](https://docs.openclaw.ai/tools/skills-config\\#notes)  Notes

- Keys under `entries` map to the skill name by default. If a skill defines
`metadata.openclaw.skillKey`, use that key instead.
- Load precedence is `<workspace>/skills` → `<workspace>/.agents/skills` →
`~/.agents/skills` → `~/.openclaw/skills` → bundled skills →
`skills.load.extraDirs`.
- Changes to skills are picked up on the next agent turn when the watcher is enabled.

### [​](https://docs.openclaw.ai/tools/skills-config\\#sandboxed-skills-+-env-vars)  Sandboxed skills + env vars

When a session is **sandboxed**, skill processes run inside the configured
sandbox backend. The sandbox does **not** inherit the host `process.env`.Use one of:

- `agents.defaults.sandbox.docker.env` for the Docker backend (or per-agent `agents.list[].sandbox.docker.env`)
- bake the env into your custom sandbox image or remote sandbox environment

Global `env` and `skills.entries.<skill>.env/apiKey` apply to **host** runs only.

## [​](https://docs.openclaw.ai/tools/skills-config\\#related)  Related

- [Skills](https://docs.openclaw.ai/tools/skills)
- [Creating skills](https://docs.openclaw.ai/tools/creating-skills)
- [Slash commands](https://docs.openclaw.ai/tools/slash-commands)

[Creating skills](https://docs.openclaw.ai/tools/creating-skills) [Slash commands](https://docs.openclaw.ai/tools/slash-commands)

Ctrl+I

---

## Brave search - OpenClaw
**Source:** https://docs.openclaw.ai/tools/brave-search

[Skip to main content](https://docs.openclaw.ai/tools/brave-search#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c652313385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web tools

Brave search

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Brave Search API](https://docs.openclaw.ai/tools/brave-search#brave-search-api)
- [Get an API key](https://docs.openclaw.ai/tools/brave-search#get-an-api-key)
- [Config example](https://docs.openclaw.ai/tools/brave-search#config-example)
- [Tool parameters](https://docs.openclaw.ai/tools/brave-search#tool-parameters)
- [Notes](https://docs.openclaw.ai/tools/brave-search#notes)
- [Related](https://docs.openclaw.ai/tools/brave-search#related)

# [​](https://docs.openclaw.ai/tools/brave-search\\#brave-search-api)  Brave Search API

OpenClaw supports Brave Search API as a `web_search` provider.

## [​](https://docs.openclaw.ai/tools/brave-search\\#get-an-api-key)  Get an API key

1. Create a Brave Search API account at [https://brave.com/search/api/](https://brave.com/search/api/)
2. In the dashboard, choose the **Search** plan and generate an API key.
3. Store the key in config or set `BRAVE_API_KEY` in the Gateway environment.

## [​](https://docs.openclaw.ai/tools/brave-search\\#config-example)  Config example

```
{
  plugins: {
    entries: {
      brave: {
        config: {
          webSearch: {
            apiKey: \"BRAVE_API_KEY_HERE\",
            mode: \"web\", // or \"llm-context\"
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: \"brave\",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

Provider-specific Brave search settings now live under `plugins.entries.brave.config.webSearch.*`.
Legacy `tools.web.search.apiKey` still loads through the compatibility shim, but it is no longer the canonical config path.`webSearch.mode` controls the Brave transport:

- `web` (default): normal Brave web search with titles, URLs, and snippets
- `llm-context`: Brave LLM Context API with pre-extracted text chunks and sources for grounding

## [​](https://docs.openclaw.ai/tools/brave-search\\#tool-parameters)  Tool parameters

[​](https://docs.openclaw.ai/tools/brave-search#param-query)

query

string

required

Search query.

[​](https://docs.openclaw.ai/tools/brave-search#param-count)

count

number

default:\"5\"

Number of results to return (1–10).

[​](https://docs.openclaw.ai/tools/brave-search#param-country)

country

string

2-letter ISO country code (e.g. `US`, `DE`).

[​](https://docs.openclaw.ai/tools/brave-search#param-language)

language

string

ISO 639-1 language code for search results (e.g. `en`, `de`, `fr`).

[​](https://docs.openclaw.ai/tools/brave-search#param-search-lang)

search\\_lang

string

Brave search-language code (e.g. `en`, `en-gb`, `zh-hans`).

[​](https://docs.openclaw.ai/tools/brave-search#param-ui-lang)

ui\\_lang

string

ISO language code for UI elements.

[​](https://docs.openclaw.ai/tools/brave-search#param-freshness)

freshness

'day' \\| 'week' \\| 'month' \\| 'year'

Time filter — `day` is 24 hours.

[​](https://docs.openclaw.ai/tools/brave-search#param-date-after)

date\\_after

string

Only results published after this date (`YYYY-MM-DD`).

[​](https://docs.openclaw.ai/tools/brave-search#param-date-before)

date\\_before

string

Only results published before this date (`YYYY-MM-DD`).

**Examples:**

```
// Country and language-specific search
await web_search({
  query: \"renewable energy\",
  country: \"DE\",
  language: \"de\",
});

// Recent results (past week)
await web_search({
  query: \"AI news\",
  freshness: \"week\",
});

// Date range search
await web_search({
  query: \"AI developments\",
  date_after: \"2024-01-01\",
  date_before: \"2024-06-30\",
});
```

## [​](https://docs.openclaw.ai/tools/brave-search\\#notes)  Notes

- OpenClaw uses the Brave **Search** plan. If you have a legacy subscription (e.g. the original Free plan with 2,000 queries/month), it remains valid but does not include newer features like LLM Context or higher rate limits.
- Each Brave plan includes **$5/month in free credit** (renewing). The Search plan costs $5 per 1,000 requests, so the credit covers 1,000 queries/month. Set your usage limit in the Brave dashboard to avoid unexpected charges. See the [Brave API portal](https://brave.com/search/api/) for current plans.
- The Search plan includes the LLM Context endpoint and AI inference rights. Storing results to train or tune models requires a plan with explicit storage rights. See the Brave [Terms of Service](https://api-dashboard.search.brave.com/terms-of-service).
- `llm-context` mode returns grounded source entries instead of the normal web-search snippet shape.
- `llm-context` mode does not support `ui_lang`, `freshness`, `date_after`, or `date_before`.
- `ui_lang` must include a region subtag like `en-US`.
- Results are cached for 15 minutes by default (configurable via `cacheTtlMinutes`).

## [​](https://docs.openclaw.ai/tools/brave-search\\#related)  Related

- [Web Search overview](https://docs.openclaw.ai/tools/web) — all providers and auto-detection
- [Perplexity Search](https://docs.openclaw.ai/tools/perplexity-search) — structured results with domain filtering
- [Exa Search](https://docs.openclaw.ai/tools/exa-search) — neural search with content extraction

[Web Search](https://docs.openclaw.ai/tools/web) [DuckDuckGo search](https://docs.openclaw.ai/tools/duckduckgo-search)

Ctrl+I

---

## Tokenjuice - OpenClaw
**Source:** https://docs.openclaw.ai/tools/tokenjuice

[Skip to main content](https://docs.openclaw.ai/tools/tokenjuice#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

Tokenjuice

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Enable the plugin](https://docs.openclaw.ai/tools/tokenjuice#enable-the-plugin)
- [What tokenjuice changes](https://docs.openclaw.ai/tools/tokenjuice#what-tokenjuice-changes)
- [Verify it is working](https://docs.openclaw.ai/tools/tokenjuice#verify-it-is-working)
- [Disable the plugin](https://docs.openclaw.ai/tools/tokenjuice#disable-the-plugin)
- [Related](https://docs.openclaw.ai/tools/tokenjuice#related)

`tokenjuice` is an optional bundled plugin that compacts noisy `exec` and `bash`
tool results after the command has already run.It changes the returned `tool_result`, not the command itself. Tokenjuice does
not rewrite shell input, rerun commands, or change exit codes.Today this applies to PI embedded runs and OpenClaw dynamic tools in the Codex
app-server harness. Tokenjuice hooks OpenClaw’s tool-result middleware and
trims the output before it goes back into the active harness session.

## [​](https://docs.openclaw.ai/tools/tokenjuice\\#enable-the-plugin)  Enable the plugin

Fast path:

```
openclaw config set plugins.entries.tokenjuice.enabled true
```

Equivalent:

```
openclaw plugins enable tokenjuice
```

OpenClaw already ships the plugin. There is no separate `plugins install`
or `tokenjuice install openclaw` step.If you prefer editing config directly:

```
{
  plugins: {
    entries: {
      tokenjuice: {
        enabled: true,
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/tools/tokenjuice\\#what-tokenjuice-changes)  What tokenjuice changes

- Compacts noisy `exec` and `bash` results before they are fed back into the session.
- Keeps the original command execution untouched.
- Preserves exact file-content reads and other commands that tokenjuice should leave raw.
- Stays opt-in: disable the plugin if you want verbatim output everywhere.

## [​](https://docs.openclaw.ai/tools/tokenjuice\\#verify-it-is-working)  Verify it is working

1. Enable the plugin.
2. Start a session that can call `exec`.
3. Run a noisy command such as `git status`.
4. Check that the returned tool result is shorter and more structured than the raw shell output.

## [​](https://docs.openclaw.ai/tools/tokenjuice\\#disable-the-plugin)  Disable the plugin

```
openclaw config set plugins.entries.tokenjuice.enabled false
```

Or:

```
openclaw plugins disable tokenjuice
```

## [​](https://docs.openclaw.ai/tools/tokenjuice\\#related)  Related

- [Exec tool](https://docs.openclaw.ai/tools/exec)
- [Thinking levels](https://docs.openclaw.ai/tools/thinking)
- [Context engine](https://docs.openclaw.ai/concepts/context-engine)

[Thinking levels](https://docs.openclaw.ai/tools/thinking) [Tool-loop detection](https://docs.openclaw.ai/tools/loop-detection)

Ctrl+I

---

## Gemini search - OpenClaw
**Source:** https://docs.openclaw.ai/tools/gemini-search

[Skip to main content](https://docs.openclaw.ai/tools/gemini-search#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web tools

Gemini search

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Get an API key](https://docs.openclaw.ai/tools/gemini-search#get-an-api-key)
- [Config](https://docs.openclaw.ai/tools/gemini-search#config)
- [How it works](https://docs.openclaw.ai/tools/gemini-search#how-it-works)
- [Supported parameters](https://docs.openclaw.ai/tools/gemini-search#supported-parameters)
- [Model selection](https://docs.openclaw.ai/tools/gemini-search#model-selection)
- [Related](https://docs.openclaw.ai/tools/gemini-search#related)

OpenClaw supports Gemini models with built-in
[Google Search grounding](https://ai.google.dev/gemini-api/docs/grounding),
which returns AI-synthesized answers backed by live Google Search results with
citations.

## [​](https://docs.openclaw.ai/tools/gemini-search\\#get-an-api-key)  Get an API key

1

[Navigate to header](https://docs.openclaw.ai/tools/gemini-search#)

Create a key

Go to [Google AI Studio](https://aistudio.google.com/apikey) and create an
API key.

2

[Navigate to header](https://docs.openclaw.ai/tools/gemini-search#)

Store the key

Set `GEMINI_API_KEY` in the Gateway environment, or configure via:

```
openclaw configure --section web
```

## [​](https://docs.openclaw.ai/tools/gemini-search\\#config)  Config

```
{
  plugins: {
    entries: {
      google: {
        config: {
          webSearch: {
            apiKey: \"AIza...\", // optional if GEMINI_API_KEY is set
            model: \"gemini-2.5-flash\", // default
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: \"gemini\",
      },
    },
  },
}
```

**Environment alternative:** set `GEMINI_API_KEY` in the Gateway environment.
For a gateway install, put it in `~/.openclaw/.env`.

## [​](https://docs.openclaw.ai/tools/gemini-search\\#how-it-works)  How it works

Unlike traditional search providers that return a list of links and snippets,
Gemini uses Google Search grounding to produce AI-synthesized answers with
inline citations. The results include both the synthesized answer and the source
URLs.

- Citation URLs from Gemini grounding are automatically resolved from Google
redirect URLs to direct URLs.
- Redirect resolution uses the SSRF guard path (HEAD + redirect checks +
http/https validation) before returning the final citation URL.
- Redirect resolution uses strict SSRF defaults, so redirects to
private/internal targets are blocked.

## [​](https://docs.openclaw.ai/tools/gemini-search\\#supported-parameters)  Supported parameters

Gemini search supports `query`.`count` is accepted for shared `web_search` compatibility, but Gemini grounding
still returns one synthesized answer with citations rather than an N-result
list.Provider-specific filters like `country`, `language`, `freshness`, and
`domain_filter` are not supported.

## [​](https://docs.openclaw.ai/tools/gemini-search\\#model-selection)  Model selection

The default model is `gemini-2.5-flash` (fast and cost-effective). Any Gemini
model that supports grounding can be used via
`plugins.entries.google.config.webSearch.model`.

## [​](https://docs.openclaw.ai/tools/gemini-search\\#related)  Related

- [Web Search overview](https://docs.openclaw.ai/tools/web) — all providers and auto-detection
- [Brave Search](https://docs.openclaw.ai/tools/brave-search) — structured results with snippets
- [Perplexity Search](https://docs.openclaw.ai/tools/perplexity-search) — structured results + content extraction

[Firecrawl](https://docs.openclaw.ai/tools/firecrawl) [Grok search](https://docs.openclaw.ai/tools/grok-search)

Ctrl+I

---

## Diffs
**Source:** https://docs.openclaw.ai/tools/diffs

[Skip to main content](https://docs.openclaw.ai/tools/diffs#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

Diffs

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/tools/diffs#quick-start)
- [Disable built-in system guidance](https://docs.openclaw.ai/tools/diffs#disable-built-in-system-guidance)
- [Typical agent workflow](https://docs.openclaw.ai/tools/diffs#typical-agent-workflow)
- [Input examples](https://docs.openclaw.ai/tools/diffs#input-examples)
- [Tool input reference](https://docs.openclaw.ai/tools/diffs#tool-input-reference)
- [Output details contract](https://docs.openclaw.ai/tools/diffs#output-details-contract)
- [Collapsed unchanged sections](https://docs.openclaw.ai/tools/diffs#collapsed-unchanged-sections)
- [Plugin defaults](https://docs.openclaw.ai/tools/diffs#plugin-defaults)
- [Persistent viewer URL config](https://docs.openclaw.ai/tools/diffs#persistent-viewer-url-config)
- [Security config](https://docs.openclaw.ai/tools/diffs#security-config)
- [Artifact lifecycle and storage](https://docs.openclaw.ai/tools/diffs#artifact-lifecycle-and-storage)
- [Viewer URL and network behavior](https://docs.openclaw.ai/tools/diffs#viewer-url-and-network-behavior)
- [Security model](https://docs.openclaw.ai/tools/diffs#security-model)
- [Browser requirements for file mode](https://docs.openclaw.ai/tools/diffs#browser-requirements-for-file-mode)
- [Troubleshooting](https://docs.openclaw.ai/tools/diffs#troubleshooting)
- [Operational guidance](https://docs.openclaw.ai/tools/diffs#operational-guidance)
- [Related](https://docs.openclaw.ai/tools/diffs#related)

`diffs` is an optional plugin tool with short built-in system guidance and a companion skill that turns change content into a read-only diff artifact for agents.It accepts either:

- `before` and `after` text
- a unified `patch`

It can return:

- a gateway viewer URL for canvas presentation
- a rendered file path (PNG or PDF) for message delivery
- both outputs in one call

When enabled, the plugin prepends concise usage guidance into system-prompt space and also exposes a detailed skill for cases where the agent needs fuller instructions.

## [​](https://docs.openclaw.ai/tools/diffs\\#quick-start)  Quick start

1

[Navigate to header](https://docs.openclaw.ai/tools/diffs#)

Enable the plugin

```
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
      },
    },
  },
}
```

2

[Navigate to header](https://docs.openclaw.ai/tools/diffs#)

Pick a mode

- view

- file

- both


Canvas-first flows: agents call `diffs` with `mode: \"view\"` and open `details.viewerUrl` with `canvas present`.

Chat file delivery: agents call `diffs` with `mode: \"file\"` and send `details.filePath` with `message` using `path` or `filePath`.

Combined: agents call `diffs` with `mode: \"both\"` to get both artifacts in one call.

## [​](https://docs.openclaw.ai/tools/diffs\\#disable-built-in-system-guidance)  Disable built-in system guidance

If you want to keep the `diffs` tool enabled but disable its built-in system-prompt guidance, set `plugins.entries.diffs.hooks.allowPromptInjection` to `false`:

```
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        hooks: {
          allowPromptInjection: false,
        },
      },
    },
  },
}
```

This blocks the diffs plugin’s `before_prompt_build` hook while keeping the plugin, tool, and companion skill available.If you want to disable both the guidance and the tool, disable the plugin instead.

## [​](https://docs.openclaw.ai/tools/diffs\\#typical-agent-workflow)  Typical agent workflow

1

[Navigate to header](https://docs.openclaw.ai/tools/diffs#)

Call diffs

Agent calls the `diffs` tool with input.

2

[Navigate to header](https://docs.openclaw.ai/tools/diffs#)

Read details

Agent reads `details` fields from the response.

3

[Navigate to header](https://docs.openclaw.ai/tools/diffs#)

Present

Agent either opens `details.viewerUrl` with `canvas present`, sends `details.filePath` with `message` using `path` or `filePath`, or does both.

## [​](https://docs.openclaw.ai/tools/diffs\\#input-examples)  Input examples

- Before and after

- Patch


```
{
  \"before\": \"# Hello\\n\\nOne\",
  \"after\": \"# Hello\\n\\nTwo\",
  \"path\": \"docs/example.md\",
  \"mode\": \"view\"
}
```

```
{
  \"patch\": \"diff --git a/src/example.ts b/src/example.ts\\n--- a/src/example.ts\\n+++ b/src/example.ts\\n@@ -1 +1 @@\\n-const x = 1;\\n+const x = 2;\\n\",
  \"mode\": \"both\"
}
```

## [​](https://docs.openclaw.ai/tools/diffs\\#tool-input-reference)  Tool input reference

All fields are optional unless noted.

[​](https://docs.openclaw.ai/tools/diffs#param-before)

before

string

Original text. Required with `after` when `patch` is omitted.

[​](https://docs.openclaw.ai/tools/diffs#param-after)

after

string

Updated text. Required with `before` when `patch` is omitted.

[​](https://docs.openclaw.ai/tools/diffs#param-patch)

patch

string

Unified diff text. Mutually exclusive with `before` and `after`.

[​](https://docs.openclaw.ai/tools/diffs#param-path)

path

string

Display filename for before and after mode.

[​](https://docs.openclaw.ai/tools/diffs#param-lang)

lang

string

Language override hint for before and after mode. Unknown values fall back to plain text.

[​](https://docs.openclaw.ai/tools/diffs#param-title)

title

string

Viewer title override.

[​](https://docs.openclaw.ai/tools/diffs#param-mode)

mode

\"view\" \\| \"file\" \\| \"both\"

Output mode. Defaults to plugin default `defaults.mode`. Deprecated alias: `\"image\"` behaves like `\"file\"` and is still accepted for backward compatibility.

[​](https://docs.openclaw.ai/tools/diffs#param-theme)

theme

\"light\" \\| \"dark\"

Viewer theme. Defaults to plugin default `defaults.theme`.

[​](https://docs.openclaw.ai/tools/diffs#param-layout)

layout

\"unified\" \\| \"split\"

Diff layout. Defaults to plugin default `defaults.layout`.

[​](https://docs.openclaw.ai/tools/diffs#param-expand-unchanged)

expandUnchanged

boolean

Expand unchanged sections when full context is available. Per-call option only (not a plugin default key).

[​](https://docs.openclaw.ai/tools/diffs#param-file-format)

fileFormat

\"png\" \\| \"pdf\"

Rendered file format. Defaults to plugin default `defaults.fileFormat`.

[​](https://docs.openclaw.ai/tools/diffs#param-file-quality)

fileQuality

\"standard\" \\| \"hq\" \\| \"print\"

Quality preset for PNG or PDF rendering.

[​](https://docs.openclaw.ai/tools/diffs#param-file-scale)

fileScale

number

Device scale override (`1`-`4`).

[​](https://docs.openclaw.ai/tools/diffs#param-file-max-width)

fileMaxWidth

number

Max render width in CSS pixels (`640`-`2400`).

[​](https://docs.openclaw.ai/tools/diffs#param-ttl-seconds)

ttlSeconds

number

default:\"1800\"

Artifact TTL in seconds for viewer and standalone file outputs. Max 21600.

[​](https://docs.openclaw.ai/tools/diffs#param-base-url)

baseUrl

string

Viewer URL origin override. Overrides plugin `viewerBaseUrl`. Must be `http` or `https`, no query/hash.

Legacy input aliases

Still accepted for backward compatibility:

- `format` -\\> `fileFormat`
- `imageFormat` -\\> `fileFormat`
- `imageQuality` -\\> `fileQuality`
- `imageScale` -\\> `fileScale`
- `imageMaxWidth` -\\> `fileMaxWidth`

Validation and limits

- `before` and `after` each max 512 KiB.
- `patch` max 2 MiB.
- `path` max 2048 bytes.
- `lang` max 128 bytes.
- `title` max 1024 bytes.
- Patch complexity cap: max 128 files and 120000 total lines.
- `patch` and `before` or `after` together are rejected.
- Rendered file safety limits (apply to PNG and PDF):
  - `fileQuality: \"standard\"`: max 8 MP (8,000,000 rendered pixels).
  - `fileQuality: \"hq\"`: max 14 MP (14,000,000 rendered pixels).
  - `fileQuality: \"print\"`: max 24 MP (24,000,000 rendered pixels).
  - PDF also has a max of 50 pages.

## [​](https://docs.openclaw.ai/tools/diffs\\#output-details-contract)  Output details contract

The tool returns structured metadata under `details`.

Viewer fields

Shared fields for modes that create a viewer:

- `artifactId`
- `viewerUrl`
- `viewerPath`
- `title`
- `expiresAt`
- `inputKind`
- `fileCount`
- `mode`
- `context` (`agentId`, `sessionId`, `messageChannel`, `agentAccountId` when available)

File fields

File fields when PNG or PDF is rendered:

- `artifactId`
- `expiresAt`
- `filePath`
- `path` (same value as `filePath`, for message tool compatibility)
- `fileBytes`
- `fileFormat`
- `fileQuality`
- `fileScale`
- `fileMaxWidth`

Compatibility aliases

Also returned for existing callers:

- `format` (same value as `fileFormat`)
- `imagePath` (same value as `filePath`)
- `imageBytes` (same value as `fileBytes`)
- `imageQuality` (same value as `fileQuality`)
- `imageScale` (same value as `fileScale`)
- `imageMaxWidth` (same value as `fileMaxWidth`)

Mode behavior summary:

| Mode | What is returned |
| --- | --- |
| `\"view\"` | Viewer fields only. |
| `\"file\"` | File fields only, no viewer artifact. |
| `\"both\"` | Viewer fields plus file fields. If file rendering fails, viewer still returns with `fileError` and `imageError` alias. |

## [​](https://docs.openclaw.ai/tools/diffs\\#collapsed-unchanged-sections)  Collapsed unchanged sections

- The viewer can show rows like `N unmodified lines`.
- Expand controls on those rows are conditional and not guaranteed for every input kind.
- Expand controls appear when the rendered diff has expandable context data, which is typical for before and after input.
- For many unified patch inputs, omitted context bodies are not available in the parsed patch hunks, so the row can appear without expand controls. This is expected behavior.
- `expandUnchanged` applies only when expandable context exists.

## [​](https://docs.openclaw.ai/tools/diffs\\#plugin-defaults)  Plugin defaults

Set plugin-wide defaults in `~/.openclaw/openclaw.json`:

```
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          defaults: {
            fontFamily: \"Fira Code\",
            fontSize: 15,
            lineSpacing: 1.6,
            layout: \"unified\",
            showLineNumbers: true,
            diffIndicators: \"bars\",
            wordWrap: true,
            background: true,
            theme: \"dark\",
            fileFormat: \"png\",
            fileQuality: \"standard\",
            fileScale: 2,
            fileMaxWidth: 960,
            mode: \"both\",
          },
        },
      },
    },
  },
}
```

Supported defaults:

- `fontFamily`
- `fontSize`
- `lineSpacing`
- `layout`
- `showLineNumbers`
- `diffIndicators`
- `wordWrap`
- `background`
- `theme`
- `fileFormat`
- `fileQuality`
- `fileScale`
- `fileMaxWidth`
- `mode`

Explicit tool parameters override these defaults.

### [​](https://docs.openclaw.ai/tools/diffs\\#persistent-viewer-url-config)  Persistent viewer URL config

[​](https://docs.openclaw.ai/tools/diffs#param-viewer-base-url)

viewerBaseUrl

string

Plugin-owned fallback for returned viewer links when a tool call does not pass `baseUrl`. Must be `http` or `https`, no query/hash.

```
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          viewerBaseUrl: \"https://gateway.example.com/openclaw\",
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/tools/diffs\\#security-config)  Security config

[​](https://docs.openclaw.ai/tools/diffs#param-security-allow-remote-viewer)

security.allowRemoteViewer

boolean

default:\"false\"

`false`: non-loopback requests to viewer routes are denied. `true`: remote viewers are allowed if tokenized path is valid.

```
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          security: {
            allowRemoteViewer: false,
          },
        },
      },
    },
  },
}
```

## [​](https://docs.openclaw.ai/tools/diffs\\#artifact-lifecycle-and-storage)  Artifact lifecycle and storage

- Artifacts are stored under the temp subfolder: `$TMPDIR/openclaw-diffs`.
- Viewer artifact metadata contains:
  - random artifact ID (20 hex chars)
  - random token (48 hex chars)
  - `createdAt` and `expiresAt`
  - stored `viewer.html` path
- Default artifact TTL is 30 minutes when not specified.
- Maximum accepted viewer TTL is 6 hours.
- Cleanup runs opportunistically after artifact creation.
- Expired artifacts are deleted.
- Fallback cleanup removes stale folders older than 24 hours when metadata is missing.

## [​](https://docs.openclaw.ai/tools/diffs\\#viewer-url-and-network-behavior)  Viewer URL and network behavior

Viewer route:

- `/plugins/diffs/view/{artifactId}/{token}`

Viewer assets:

- `/plugins/diffs/assets/viewer.js`
- `/plugins/diffs/assets/viewer-runtime.js`

The viewer document resolves those assets relative to the viewer URL, so an optional `baseUrl` path prefix is preserved for both asset requests too.URL construction behavior:

- If tool-call `baseUrl` is provided, it is used after strict validation.
- Else if plugin `viewerBaseUrl` is configured, it is used.
- Without either override, viewer URL defaults to loopback `127.0.0.1`.
- If gateway bind mode is `custom` and `gateway.customBindHost` is set, that host is used.

`baseUrl` rules:

- Must be `http://` or `https://`.
- Query and hash are rejected.
- Origin plus optional base path is allowed.

## [​](https://docs.openclaw.ai/tools/diffs\\#security-model)  Security model

Viewer hardening

- Loopback-only by default.
- Tokenized viewer paths with strict ID and token validation.
- Viewer response CSP:
  - `default-src 'none'`
  - scripts and assets only from self
  - no outbound `connect-src`
- Remote miss throttling when remote access is enabled:
  - 40 failures per 60 seconds
  - 60 second lockout (`429 Too Many Requests`)

File rendering hardening

- Screenshot browser request routing is deny-by-default.
- Only local viewer assets from `http://127.0.0.1/plugins/diffs/assets/*` are allowed.
- External network requests are blocked.

## [​](https://docs.openclaw.ai/tools/diffs\\#browser-requirements-for-file-mode)  Browser requirements for file mode

`mode: \"file\"` and `mode: \"both\"` need a Chromium-compatible browser.Resolution order:

1

[Navigate to header](https://docs.openclaw.ai/tools/diffs#)

Config

`browser.executablePath` in OpenClaw config.

2

[Navigate to header](https://docs.openclaw.ai/tools/diffs#)

Environment variables

- `OPENCLAW_BROWSER_EXECUTABLE_PATH`
- `BROWSER_EXECUTABLE_PATH`
- `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH`

3

[Navigate to header](https://docs.openclaw.ai/tools/diffs#)

Platform fallback

Platform command/path discovery fallback.

Common failure text:

- `Diff PNG/PDF rendering requires a Chromium-compatible browser...`

Fix by installing Chrome, Chromium, Edge, or Brave, or setting one of the executable path options above.

## [​](https://docs.openclaw.ai/tools/diffs\\#troubleshooting)  Troubleshooting

Input validation errors

- `Provide patch or both before and after text.` — include both `before` and `after`, or provide `patch`.
- `Provide either patch or before/after input, not both.` — do not mix input modes.
- `Invalid baseUrl: ...` — use `http(s)` origin with optional path, no query/hash.
- `{field} exceeds maximum size (...)` — reduce payload size.
- Large patch rejection — reduce patch file count or total lines.

Viewer accessibility

- Viewer URL resolves to `127.0.0.1` by default.
- For remote access scenarios, either:
  - set plugin `viewerBaseUrl`, or
  - pass `baseUrl` per tool call, or
  - use `gateway.bind=custom` and `gateway.customBindHost`
- If `gateway.trustedProxies` includes loopback for a same-host proxy (for example Tailscale Serve), raw loopback viewer requests without forwarded client-IP headers fail closed by design.
- For that proxy topology:
  - prefer `mode: \"file\"` or `mode: \"both\"` when you only need an attachment, or
  - intentionally enable `security.allowRemoteViewer` and set plugin `viewerBaseUrl` or pass a proxy/public `baseUrl` when you need a shareable viewer URL
- Enable `security.allowRemoteViewer` only when you intend external viewer access.

Unmodified-lines row has no expand button

This can happen for patch input when the patch does not carry expandable context. This is expected and does not indicate a viewer failure.

Artifact not found

- Artifact expired due TTL.
- Token or path changed.
- Cleanup removed stale data.

## [​](https://docs.openclaw.ai/tools/diffs\\#operational-guidance)  Operational guidance

- Prefer `mode: \"view\"` for local interactive reviews in canvas.
- Prefer `mode: \"file\"` for outbound chat channels that need an attachment.
- Keep `allowRemoteViewer` disabled unless your deployment requires remote viewer URLs.
- Set explicit short `ttlSeconds` for sensitive diffs.
- Avoid sending secrets in diff input when not required.
- If your channel compresses images aggressively (for example Telegram or WhatsApp), prefer PDF output (`fileFormat: \"pdf\`).

Diff rendering engine powered by [Diffs](https://diffs.com/).

## [​](https://docs.openclaw.ai/tools/diffs\\#related)  Related

- [Browser](https://docs.openclaw.ai/tools/browser)
- [Plugins](https://docs.openclaw.ai/tools/plugin)
- [Tools overview](https://docs.openclaw.ai/tools)

[Code execution](https://docs.openclaw.ai/tools/code-execution) [Elevated mode](https://docs.openclaw.ai/tools/elevated)

Ctrl+I

---

## Creating skills - OpenClaw
**Source:** https://docs.openclaw.ai/tools/creating-skills

[Skip to main content](https://docs.openclaw.ai/tools/creating-skills#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Skills

Creating skills

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Create your first skill](https://docs.openclaw.ai/tools/creating-skills#create-your-first-skill)
- [Skill metadata reference](https://docs.openclaw.ai/tools/creating-skills#skill-metadata-reference)
- [Best practices](https://docs.openclaw.ai/tools/creating-skills#best-practices)
- [Where skills live](https://docs.openclaw.ai/tools/creating-skills#where-skills-live)
- [Related](https://docs.openclaw.ai/tools/creating-skills#related)

Skills teach the agent how and when to use tools. Each skill is a directory
containing a `SKILL.md` file with YAML frontmatter and markdown instructions.For how skills are loaded and prioritized, see [Skills](https://docs.openclaw.ai/tools/skills).

## [​](https://docs.openclaw.ai/tools/creating-skills\\#create-your-first-skill)  Create your first skill

1

[Navigate to header](https://docs.openclaw.ai/tools/creating-skills#)

Create the skill directory

Skills live in your workspace. Create a new folder:

```
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

2

[Navigate to header](https://docs.openclaw.ai/tools/creating-skills#)

Write SKILL.md

Create `SKILL.md` inside that directory. The frontmatter defines metadata,
and the markdown body contains instructions for the agent.

```
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say

\"Hello from your custom skill!\".
```

3

[Navigate to header](https://docs.openclaw.ai/tools/creating-skills#)

Add tools (optional)

You can define custom tool schemas in the frontmatter or instruct the agent
to use existing system tools (like `exec` or `browser`). Skills can also
ship inside plugins alongside the tools they document.

4

[Navigate to header](https://docs.openclaw.ai/tools/creating-skills#)

Load the skill

Start a new session so OpenClaw picks up the skill:

```
# From chat
/new

# Or restart the gateway
openclaw gateway restart
```

Verify the skill loaded:

```
openclaw skills list
```

5

[Navigate to header](https://docs.openclaw.ai/tools/creating-skills#)

Test it

Send a message that should trigger the skill:

```
openclaw agent --message \"give me a greeting\"
```

Or just chat with the agent and ask for a greeting.

## [​](https://docs.openclaw.ai/tools/creating-skills\\#skill-metadata-reference)  Skill metadata reference

The YAML frontmatter supports these fields:

| Field | Required | Description |
| --- | --- | --- |
| `name` | Yes | Unique identifier (snake\\_case) |
| `description` | Yes | One-line description shown to the agent |
| `metadata.openclaw.os` | No | OS filter (`[\"darwin\"]`, `[\"linux\"]`, etc.) |
| `metadata.openclaw.requires.bins` | No | Required binaries on PATH |
| `metadata.openclaw.requires.config` | No | Required config keys |

## [​](https://docs.openclaw.ai/tools/creating-skills\\#best-practices)  Best practices

- **Be concise** — instruct the model on _what_ to do, not how to be an AI
- **Safety first** — if your skill uses `exec`, ensure prompts don’t allow arbitrary command injection from untrusted input
- **Test locally** — use `openclaw agent --message \"...\"` to test before sharing
- **Use ClawHub** — browse and contribute skills at [ClawHub](https://clawhub.ai/)

## [​](https://docs.openclaw.ai/tools/creating-skills\\#where-skills-live)  Where skills live

| Location | Precedence | Scope |
| --- | --- | --- |
| `\\<workspace\\>/skills/` | Highest | Per-agent |
| `\\<workspace\\>/.agents/skills/` | High | Per-workspace agent |
| `~/.agents/skills/` | Medium | Shared agent profile |
| `~/.openclaw/skills/` | Medium | Shared (all agents) |
| Bundled (shipped with OpenClaw) | Low | Global |
| `skills.load.extraDirs` | Lowest | Custom shared folders |

## [​](https://docs.openclaw.ai/tools/creating-skills\\#related)  Related

- [Skills reference](https://docs.openclaw.ai/tools/skills) — loading, precedence, and gating rules
- [Skills config](https://docs.openclaw.ai/tools/skills-config) — `skills.*` config schema
- [ClawHub](https://docs.openclaw.ai/tools/clawhub) — public skill registry
- [Building Plugins](https://docs.openclaw.ai/plugins/building-plugins) — plugins can ship skills

[Skills](https://docs.openclaw.ai/tools/skills) [Skills config](https://docs.openclaw.ai/tools/skills-config)

Ctrl+I

---

## Tavily - OpenClaw
**Source:** https://docs.openclaw.ai/tools/tavily

[Skip to main content](https://docs.openclaw.ai/tools/tavily#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Web tools

Tavily

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Get an API key](https://docs.openclaw.ai/tools/tavily#get-an-api-key)
- [Configure Tavily search](https://docs.openclaw.ai/tools/tavily#configure-tavily-search)
- [Tavily plugin tools](https://docs.openclaw.ai/tools/tavily#tavily-plugin-tools)
- [tavily\\_search](https://docs.openclaw.ai/tools/tavily#tavily_search)
- [tavily\\_extract](https://docs.openclaw.ai/tools/tavily#tavily_extract)
- [Choosing the right tool](https://docs.openclaw.ai/tools/tavily#choosing-the-right-tool)
- [Related](https://docs.openclaw.ai/tools/tavily#related)

OpenClaw can use **Tavily** in two ways:

- as the `web_search` provider
- as explicit plugin tools: `tavily_search` and `tavily_extract`

Tavily is a search API designed for AI applications, returning structured results
optimized for LLM consumption. It supports configurable search depth, topic
filtering, domain filters, AI-generated answer summaries, and content extraction
from URLs (including JavaScript-rendered pages).

## [​](https://docs.openclaw.ai/tools/tavily\\#get-an-api-key)  Get an API key

1. Create a Tavily account at [tavily.com](https://tavily.com/).
2. Generate an API key in the dashboard.
3. Store it in config or set `TAVILY_API_KEY` in the gateway environment.

## [​](https://docs.openclaw.ai/tools/tavily\\#configure-tavily-search)  Configure Tavily search

```
{
  plugins: {
    entries: {
      tavily: {
        enabled: true,
        config: {
          webSearch: {
            apiKey: \"tvly-...\", // optional if TAVILY_API_KEY is set
            baseUrl: \"https://api.tavily.com\",
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: \"tavily\",
      },
    },
  },
}
```

Notes:

- Choosing Tavily in onboarding or `openclaw configure --section web` enables
the bundled Tavily plugin automatically.
- Store Tavily config under `plugins.entries.tavily.config.webSearch.*`.
- `web_search` with Tavily supports `query` and `count` (up to 20 results).
- For Tavily-specific controls like `search_depth`, `topic`, `include_answer`,
or domain filters, use `tavily_search`.

## [​](https://docs.openclaw.ai/tools/tavily\\#tavily-plugin-tools)  Tavily plugin tools

### [​](https://docs.openclaw.ai/tools/tavily\\#tavily_search)  `tavily_search`

Use this when you want Tavily-specific search controls instead of generic
`web_search`.

| Parameter | Description |
| --- | --- |
| `query` | Search query string (keep under 400 characters) |
| `search_depth` | `basic` (default, balanced) or `advanced` (highest relevance, slower) |
| `topic` | `general` (default), `news` (real-time updates), or `finance` |
| `max_results` | Number of results, 1-20 (default: 5) |
| `include_answer` | Include an AI-generated answer summary (default: false) |
| `time_range` | Filter by recency: `day`, `week`, `month`, or `year` |
| `include_domains` | Array of domains to restrict results to |
| `exclude_domains` | Array of domains to exclude from results |

**Search depth:**

| Depth | Speed | Relevance | Best for |
| --- | --- | --- | --- |
| `basic` | Faster | High | General-purpose queries (default) |
| `advanced` | Slower | Highest | Precision, specific facts, research |

### [​](https://docs.openclaw.ai/tools/tavily\\#tavily_extract)  `tavily_extract`

Use this to extract clean content from one or more URLs. Handles
JavaScript-rendered pages and supports query-focused chunking for targeted
extraction.

| Parameter | Description |
| --- | --- |
| `urls` | Array of URLs to extract (1-20 per request) |
| `query` | Rerank extracted chunks by relevance to this query |
| `extract_depth` | `basic` (default, fast) or `advanced` (for JS-heavy pages) |
| `chunks_per_source` | Chunks per URL, 1-5 (requires `query`) |
| `include_images` | Include image URLs in results (default: false) |

**Extract depth:**

| Depth | When to use |
| --- | --- |
| `basic` | Simple pages - try this first |
| `advanced` | JS-rendered SPAs, dynamic content, tables |

Tips:

- Max 20 URLs per request. Batch larger lists into multiple calls.
- Use `query` \\+ `chunks_per_source` to get only relevant content instead of full pages.
- Try `basic` first; fall back to `advanced` if content is missing or incomplete.

## [​](https://docs.openclaw.ai/tools/tavily\\#choosing-the-right-tool)  Choosing the right tool

| Need | Tool |
| --- | --- |
| Quick web search, no special options | `web_search` |
| Search with depth, topic, AI answers | `tavily_search` |
| Extract content from specific URLs | `tavily_extract` |

## [​](https://docs.openclaw.ai/tools/tavily\\#related)  Related

- [Web Search overview](https://docs.openclaw.ai/tools/web) — all providers and auto-detection
- [Firecrawl](https://docs.openclaw.ai/tools/firecrawl) — search + scraping with content extraction
- [Exa Search](https://docs.openclaw.ai/tools/exa-search) — neural search with content extraction

[SearXNG search](https://docs.openclaw.ai/tools/searxng-search) [Agent send](https://docs.openclaw.ai/tools/agent-send)

Ctrl+I

---

## ClawHub - OpenClaw
**Source:** https://docs.openclaw.ai/tools/clawhub

[Skip to main content](https://docs.openclaw.ai/tools/clawhub#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Skills

ClawHub

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/tools/clawhub#quick-start)
- [Native OpenClaw flows](https://docs.openclaw.ai/tools/clawhub#native-openclaw-flows)
- [What ClawHub is](https://docs.openclaw.ai/tools/clawhub#what-clawhub-is)
- [Workspace and skill loading](https://docs.openclaw.ai/tools/clawhub#workspace-and-skill-loading)
- [Service features](https://docs.openclaw.ai/tools/clawhub#service-features)
- [Security and moderation](https://docs.openclaw.ai/tools/clawhub#security-and-moderation)
- [ClawHub CLI](https://docs.openclaw.ai/tools/clawhub#clawhub-cli)
- [Global options](https://docs.openclaw.ai/tools/clawhub#global-options)
- [Commands](https://docs.openclaw.ai/tools/clawhub#commands)
- [Common workflows](https://docs.openclaw.ai/tools/clawhub#common-workflows)
- [Plugin package metadata](https://docs.openclaw.ai/tools/clawhub#plugin-package-metadata)
- [Versioning, lockfile, and telemetry](https://docs.openclaw.ai/tools/clawhub#versioning-lockfile-and-telemetry)
- [Environment variables](https://docs.openclaw.ai/tools/clawhub#environment-variables)
- [Related](https://docs.openclaw.ai/tools/clawhub#related)

ClawHub is the public registry for **OpenClaw skills and plugins**.

- Use native `openclaw` commands to search, install, and update skills, and to install plugins from ClawHub.
- Use the separate `clawhub` CLI for registry auth, publish, delete/undelete, and sync workflows.

Site: [clawhub.ai](https://clawhub.ai/)

## [​](https://docs.openclaw.ai/tools/clawhub\\#quick-start)  Quick start

1

[Navigate to header](https://docs.openclaw.ai/tools/clawhub#)

Search

```
openclaw skills search \"calendar\"
```

2

[Navigate to header](https://docs.openclaw.ai/tools/clawhub#)

Install

```
openclaw skills install <skill-slug>
```

3

[Navigate to header](https://docs.openclaw.ai/tools/clawhub#)

Use

Start a new OpenClaw session — it picks up the new skill.

4

[Navigate to header](https://docs.openclaw.ai/tools/clawhub#)

Publish (optional)

For registry-authenticated workflows (publish, sync, manage), install
the separate `clawhub` CLI:

```
npm i -g clawhub
# or
pnpm add -g clawhub
```

## [​](https://docs.openclaw.ai/tools/clawhub\\#native-openclaw-flows)  Native OpenClaw flows

- Skills

- Plugins


```
openclaw skills search \"calendar\"
openclaw skills install <skill-slug>
openclaw skills update --all
```

Native `openclaw` commands install into your active workspace and
persist source metadata so later `update` calls can stay on ClawHub.

```
openclaw plugins install clawhub:<package>
openclaw plugins update --all
```

Bare npm-safe plugin specs are also tried against ClawHub before npm:

```
openclaw plugins install openclaw-codex-app-server
```

Use `npm:<package>` when you want npm-only resolution without a
ClawHub lookup:

```
openclaw plugins install npm:openclaw-codex-app-server
```

Plugin installs validate advertised `pluginApi` and
`minGatewayVersion` compatibility before archive install runs, so
incompatible hosts fail closed early instead of partially installing
the package.

`openclaw plugins install clawhub:...` only accepts installable plugin
families. If a ClawHub package is actually a skill, OpenClaw stops and
points you at `openclaw skills install <slug>` instead.Anonymous ClawHub plugin installs also fail closed for private packages.
Community or other non-official channels can still install, but OpenClaw
warns so operators can review source and verification before enabling
them.

## [​](https://docs.openclaw.ai/tools/clawhub\\#what-clawhub-is)  What ClawHub is

- A public registry for OpenClaw skills and plugins.
- A versioned store of skill bundles and metadata.
- A discovery surface for search, tags, and usage signals.

A typical skill is a versioned bundle of files that includes:

- A `SKILL.md` file with the primary description and usage.
- Optional configs, scripts, or supporting files used by the skill.
- Metadata such as tags, summary, and install requirements.

ClawHub uses metadata to power discovery and safely expose skill
capabilities. The registry tracks usage signals (stars, downloads) to
improve ranking and visibility. Each publish creates a new semver
version, and the registry keeps version history so users can audit
changes.

## [​](https://docs.openclaw.ai/tools/clawhub\\#workspace-and-skill-loading)  Workspace and skill loading

The separate `clawhub` CLI also installs skills into `./skills` under
your current working directory. If an OpenClaw workspace is configured,
`clawhub` falls back to that workspace unless you override `--workdir`
(or `CLAWHUB_WORKDIR`). OpenClaw loads workspace skills from
`<workspace>/skills` and picks them up in the **next** session.If you already use `~/.openclaw/skills` or bundled skills, workspace
skills take precedence. For more detail on how skills are loaded,
shared, and gated, see [Skills](https://docs.openclaw.ai/tools/skills).

## [​](https://docs.openclaw.ai/tools/clawhub\\#service-features)  Service features

| Feature | Notes |
| --- | --- |
| Public browsing | Skills and their `SKILL.md` content are publicly viewable. |
| Search | Embedding-powered (vector search), not just keywords. |
| Versioning | Semver, changelogs, and tags (including `latest`). |
| Downloads | Zip per version. |
| Stars and comments | Community feedback. |
| Moderation | Approvals and audits. |
| CLI-friendly API | Suitable for automation and scripting. |

## [​](https://docs.openclaw.ai/tools/clawhub\\#security-and-moderation)  Security and moderation

ClawHub is open by default — anyone can upload skills, but a GitHub
account must be **at least one week old** to publish. This slows down
abuse without blocking legitimate contributors.

Reporting

- Any signed-in user can report a skill.
- Report reasons are required and recorded.
- Each user can have up to 20 active reports at a time.
- Skills with more than 3 unique reports are auto-hidden by default.

Moderation

- Moderators can view hidden skills, unhide them, delete them, or ban users.
- Abusing the report feature can result in account bans.
- Interested in becoming a moderator? Ask in the OpenClaw Discord and contact a moderator or maintainer.

## [​](https://docs.openclaw.ai/tools/clawhub\\#clawhub-cli)  ClawHub CLI

You only need this for registry-authenticated workflows such as
publish/sync.

### [​](https://docs.openclaw.ai/tools/clawhub\\#global-options)  Global options

[​](https://docs.openclaw.ai/tools/clawhub#param-workdir-dir)

--workdir <dir>

string

Working directory. Default: current dir; falls back to OpenClaw workspace.

[​](https://docs.openclaw.ai/tools/clawhub#param-dir-dir)

--dir <dir>

string

default:\"skills\"

Skills directory, relative to workdir.

[​](https://openclaw.ai/tools/clawhub#param-site-url)

--site <url>

string

Site base URL (browser login).

[​](https://docs.openclaw.ai/tools/clawhub#param-registry-url)

--registry <url>

string

Registry API base URL.

[​](https://docs.openclaw.ai/tools/clawhub#param-no-input)

--no-input

boolean

Disable prompts (non-interactive).

[​](https://docs.openclaw.ai/tools/clawhub#param-v-cli-version)

-V, --cli-version

boolean

Print CLI version.

### [​](https://docs.openclaw.ai/tools/clawhub\\#commands)  Commands

Auth (login / logout / whoami)

```
clawhub login              # browser flow
clawhub login --token <token>
clawhub logout
clawhub whoami
```

Login options:

- `--token <token>` — paste an API token.
- `--label <label>` — label stored for browser login tokens (default: `CLI token`).
- `--no-browser` — do not open a browser (requires `--token`).

Search

```
clawhub search \"query\"
```

Searches skills. For plugin/package discovery, use `clawhub package explore`.

- `--limit <n>` — max results.

Browse / inspect plugins

```
clawhub package explore --family code-plugin
clawhub package explore \"episodic-claw\" --family code-plugin
clawhub package inspect episodic-claw
```

`package explore` and `package inspect` are the ClawHub CLI surfaces for plugin/package discovery and metadata inspection. Native OpenClaw installs still use `openclaw plugins install clawhub:<package>`.Options:

- `--family skill|code-plugin|bundle-plugin` — filter package family.
- `--official` — show only official packages.
- `--executes-code` — show only packages that execute code.
- `--version <version>` / `--tag <tag>` — inspect a specific package version.
- `--versions`, `--files`, `--file <path>` — inspect package history and files.
- `--json` — machine-readable output.

Install / update / list

```
clawhub install <slug>
clawhub update <slug>
clawhub update --all
clawhub list
```

Options:

- `--version <version>` — install or update to a specific version (single slug only on `update`).
- `--force` — overwrite if the folder already exists, or when local files do not match any published version.
- `clawhub list` reads `.clawhub/lock.json`.

Publish skills

```
clawhub skill publish <path>
```

Options:

- `--slug <slug>` — skill slug.
- `--name <name>` — display name.
- `--version <version>` — semver version.
- `--changelog <text>` — changelog text (can be empty).
- `--tags <tags>` — comma-separated tags (default: `latest`).

Publish plugins

```
clawhub package publish <source>
```

`<source>` can be a local folder, `owner/repo`, `owner/repo@ref`, or a
GitHub URL.Options:

- `--dry-run` — build the exact publish plan without uploading anything.
- `--json` — emit machine-readable output for CI.
- `--source-repo`, `--source-commit`, `--source-ref` — optional overrides when auto-detection is not enough.

Delete / undelete (owner or admin)

```
clawhub delete <slug> --yes
clawhub undelete <slug> --yes
```

Sync (scan local + publish new or updated)

```
clawhub sync
```

Options:

- `--root <dir...>` — extra scan roots.
- `--all` — upload everything without prompts.
- `--dry-run` — show what would be uploaded.
- `--bump <type>` — `patch|minor|major` for updates (default: `patch`).
- `--changelog <text>` — changelog for non-interactive updates.
- `--tags <tags>` — comma-separated tags (default: `latest`).
- `--concurrency <n>` — registry checks (default: `4`).

## [​](https://docs.openclaw.ai/tools/clawhub\\#common-workflows)  Common workflows

- Search

- Find a plugin

- Install

- Update all

- Publish a single skill

- Sync many skills

- Publish a plugin from GitHub


```
clawhub search \"postgres backups\"
```

```
clawhub package explore --family code-plugin
clawhub package explore \"memory\" --family code-plugin
clawhub package inspect episodic-claw
```

```
clawhub install my-skill-pack
```

```
clawhub update --all
```

```
clawhub skill publish ./my-skill --slug my-skill --name \"My Skill\" --version 1.0.0 --tags latest
```

```
clawhub sync --all
```

```
clawhub package publish your-org/your-plugin --dry-run
clawhub package publish your-org/your-plugin
clawhub package publish your-org/your-plugin@v1.0.0
clawhub package publish https://github.com/your-org/your-plugin
```

### [​](https://docs.openclaw.ai/tools/clawhub\\#plugin-package-metadata)  Plugin package metadata

Code plugins must include the required OpenClaw metadata in
`package.json`:

```
{
  \"name\": \"@myorg/openclaw-my-plugin\",
  \"version\": \"1.0.0\",
  \"type\": \"module\",
  \"openclaw\": {
    \"extensions\": [\"./src/index.ts\"],
    \"runtimeExtensions\": [\"./dist/index.js\"],
    \"compat\": {
      \"pluginApi\": \">=2026.3.24-beta.2\",
      \"minGatewayVersion\": \"2026.3.24-beta.2\"
    },
    \"build\": {
      \"openclawVersion\": \"2026.3.24-beta.2\",
      \"pluginSdkVersion\": \"2026.3.24-beta.2\"
    }
  }
}
```

Published packages should ship **built JavaScript** and point
`runtimeExtensions` at that output. Git checkout installs can still fall
back to TypeScript source when no built files exist, but built runtime
entries avoid runtime TypeScript compilation in startup, doctor, and
plugin loading paths.

## [​](https://docs.openclaw.ai/tools/clawhub\\#versioning-lockfile-and-telemetry)  Versioning, lockfile, and telemetry

Versioning and tags

- Each publish creates a new **semver**`SkillVersion`.
- Tags (like `latest`) point to a version; moving tags lets you roll back.
- Changelogs are attached per version and can be empty when syncing or publishing updates.

Local changes vs registry versions

Updates compare the local skill contents to registry versions using a
content hash. If local files do not match any published version, the
CLI asks before overwriting (or requires `--force` in
non-interactive runs).

Sync scanning and fallback roots

`clawhub sync` scans your current workdir first. If no skills are
found, it falls back to known legacy locations (for example
`~/openclaw/skills` and `~/.openclaw/skills`). This is designed to
find older skill installs without extra flags.

Storage and lockfile

- Installed skills are recorded in `.clawhub/lock.json` under your workdir.
- Auth tokens are stored in the ClawHub CLI config file (override via `CLAWHUB_CONFIG_PATH`).

Telemetry (install counts)

When you run `clawhub sync` while logged in, the CLI sends a minimal
snapshot to compute install counts. You can disable this entirely:

```
export CLAWHUB_DISABLE_TELEMETRY=1
```

## [​](https://docs.openclaw.ai/tools/clawhub\\#environment-variables)  Environment variables

| Variable | Effect |
| --- | --- |
| `CLAWHUB_SITE` | Override the site URL. |
| `CLAWHUB_REGISTRY` | Override the registry API URL. |
| `CLAWHUB_CONFIG_PATH` | Override where the CLI stores the token/config. |
| `CLAWHUB_WORKDIR` | Override the default workdir. |
| `CLAWHUB_DISABLE_TELEMETRY=1` | Disable telemetry on `sync`. |

## [​](https://docs.openclaw.ai/tools/clawhub\\#related)  Related

- [Community plugins](https://docs.openclaw.ai/plugins/community)
- [Plugins](https://docs.openclaw.ai/tools/plugin)
- [Skills](https://docs.openclaw.ai/tools/skills)

[Slash commands](https://docs.openclaw.ai/tools/slash-commands) [OpenProse](https://docs.openclaw.ai/prose)

Ctrl+I

---

## Agent send - OpenClaw
**Source:** https://docs.openclaw.ai/tools/agent-send

[Skip to main content](https://docs.openclaw.ai/tools/agent-send#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Agent coordination

Agent send

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/tools/agent-send#quick-start)
- [Flags](https://docs.openclaw.ai/tools/agent-send#flags)
- [Behavior](https://docs.openclaw.ai/tools/agent-send#behavior)
- [Examples](https://docs.openclaw.ai/tools/agent-send#examples)
- [Related](https://docs.openclaw.ai/tools/agent-send#related)

`openclaw agent` runs a single agent turn from the command line without needing
an inbound chat message. Use it for scripted workflows, testing, and
programmatic delivery.

## [​](https://docs.openclaw.ai/tools/agent-send\\#quick-start)  Quick start

1

[Navigate to header](https://docs.openclaw.ai/tools/agent-send#)

Run a simple agent turn

```
openclaw agent --message \"What is the weather today?\"
```

This sends the message through the Gateway and prints the reply.

2

[Navigate to header](https://docs.openclaw.ai/tools/agent-send#)

Target a specific agent or session

```
# Target a specific agent
openclaw agent --agent ops --message \"Summarize logs\"

# Target a phone number (derives session key)
openclaw agent --to +15555550123 --message \"Status update\"

# Reuse an existing session
openclaw agent --session-id abc123 --message \"Continue the task\"
```

3

[Navigate to header](https://docs.openclaw.ai/tools/agent-send#)

Deliver the reply to a channel

```
# Deliver to WhatsApp (default channel)
openclaw agent --to +15555550123 --message \"Report ready\" --deliver

# Deliver to Slack
openclaw agent --agent ops --message \"Generate report\" \\\
  --deliver --reply-channel slack --reply-to \"#reports\"
```

## [​](https://docs.openclaw.ai/tools/agent-send\\#flags)  Flags

| Flag | Description |
| --- | --- |
| `--message \\<text\\>` | Message to send (required) |
| `--to \\<dest\\>` | Derive session key from a target (phone, chat id) |
| `--agent \\<id\\>` | Target a configured agent (uses its `main` session) |
| `--session-id \\<id\\>` | Reuse an existing session by id |
| `--local` | Force local embedded runtime (skip Gateway) |
| `--deliver` | Send the reply to a chat channel |
| `--channel \\<name\\>` | Delivery channel (whatsapp, telegram, discord, slack, etc.) |
| `--reply-to \\<target\\>` | Delivery target override |
| `--reply-channel \\<name\\>` | Delivery channel override |
| `--reply-account \\<id\\>` | Delivery account id override |
| `--thinking \\<level\\>` | Set thinking level for the selected model profile |
| `--verbose \\<on|full|off\\>` | Set verbose level |
| `--timeout \\<seconds\\>` | Override agent timeout |
| `--json` | Output structured JSON |

## [​](https://docs.openclaw.ai/tools/agent-send\\#behavior)  Behavior

- By default, the CLI goes **through the Gateway**. Add `--local` to force the
embedded runtime on the current machine.
- If the Gateway is unreachable, the CLI **falls back** to the local embedded run.
- Session selection: `--to` derives the session key (group/channel targets
preserve isolation; direct chats collapse to `main`).
- Thinking and verbose flags persist into the session store.
- Output: plain text by default, or `--json` for structured payload + metadata.

## [​](https://docs.openclaw.ai/tools/agent-send\\#examples)  Examples

```
# Simple turn with JSON output
openclaw agent --to +15555550123 --message \"Trace logs\" --verbose on --json

# Turn with thinking level
openclaw agent --session-id 1234 --message \"Summarize inbox\" --thinking medium

# Deliver to a different channel than the session
openclaw agent --agent ops --message \"Alert\" --deliver --reply-channel telegram --reply-to \"@admin\"
```

## [​](https://docs.openclaw.ai/tools/agent-send\\#related)  Related

- [Agent CLI reference](https://docs.openclaw.ai/cli/agent)
- [Sub-agents](https://docs.openclaw.ai/tools/subagents) — background sub-agent spawning
- [Sessions](https://docs.openclaw.ai/concepts/session) — how session keys work

[Tavily](https://docs.openclaw.ai/tools/tavily) [Sub-agents](https://docs.openclaw.ai/tools/subagents)

Ctrl+I

---

## Plugins
**Source:** https://docs.openclaw.ai/tools/plugin

[Skip to main content](https://docs.openclaw.ai/tools/plugin#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Plugins

Plugins

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/tools/plugin#quick-start)
- [Plugin types](https://docs.openclaw.ai/tools/plugin#plugin-types)
- [Package entrypoints](https://docs.openclaw.ai/tools/plugin#package-entrypoints)
- [Official plugins](https://docs.openclaw.ai/tools/plugin#official-plugins)
- [Installable (npm)](https://docs.openclaw.ai/tools/plugin#installable-npm)
- [Core (shipped with OpenClaw)](https://docs.openclaw.ai/tools/plugin#core-shipped-with-openclaw)
- [Configuration](https://docs.openclaw.ai/tools/plugin#configuration)
- [Discovery and precedence](https://docs.openclaw.ai/tools/plugin#discovery-and-precedence)
- [Enablement rules](https://docs.openclaw.ai/tools/plugin#enablement-rules)
- [Troubleshooting runtime hooks](https://docs.openclaw.ai/tools/plugin#troubleshooting-runtime-hooks)
- [Duplicate channel or tool ownership](https://docs.openclaw.ai/tools/plugin#duplicate-channel-or-tool-ownership)
- [Plugin slots (exclusive categories)](https://docs.openclaw.ai/tools/plugin#plugin-slots-exclusive-categories)
- [CLI reference](https://docs.openclaw.ai/tools/plugin#cli-reference)
- [Plugin API overview](https://docs.openclaw.ai/tools/plugin#plugin-api-overview)
- [Related](https://docs.openclaw.ai/tools/plugin#related)

Plugins extend OpenClaw with new capabilities: channels, model providers,
agent harnesses, tools, skills, speech, realtime transcription, realtime
voice, media-understanding, image generation, video generation, web fetch, web
search, and more. Some plugins are **core** (shipped with OpenClaw), others
are **external** (published on npm by the community).

## [​](https://docs.openclaw.ai/tools/plugin\\#quick-start)  Quick start

1

[Navigate to header](https://docs.openclaw.ai/tools/plugin#)

See what is loaded

```
openclaw plugins list
```

2

[Navigate to header](https://docs.openclaw.ai/tools/plugin#)

Install a plugin

```
# From npm
openclaw plugins install @openclaw/voice-call

# From a local directory or archive
openclaw plugins install ./my-plugin
openclaw plugins install ./my-plugin.tgz
```

3

[Navigate to header](https://docs.openclaw.ai/tools/plugin#)

Restart the Gateway

```
openclaw gateway restart
```

Then configure under `plugins.entries.\\<id\\>.config` in your config file.

If you prefer chat-native control, enable `commands.plugins: true` and use:

```
/plugin install clawhub:@openclaw/voice-call
/plugin show voice-call
/plugin enable voice-call
```

The install path uses the same resolver as the CLI: local path/archive, explicit
`clawhub:<pkg>`, explicit `npm:<pkg>`, or bare package spec (ClawHub first, then
npm fallback).If config is invalid, install normally fails closed and points you at
`openclaw doctor --fix`. The only recovery exception is a narrow bundled-plugin
reinstall path for plugins that opt into
`openclaw.install.allowInvalidConfigRecovery`.
During Gateway startup, invalid config for one plugin is isolated to that plugin:
startup logs the `plugins.entries.<id>.config` issue, skips that plugin during
load, and keeps other plugins and channels online. Run `openclaw doctor --fix`
to quarantine the bad plugin config by disabling that plugin entry and removing
its invalid config payload; the normal config backup keeps the previous values.
When a channel config references a plugin that is no longer discoverable but the
same stale plugin id remains in plugin config or install records, Gateway startup
logs warnings and skips that channel instead of blocking every other channel.
Run `openclaw doctor --fix` to remove the stale channel/plugin entries; unknown
channel keys without stale-plugin evidence still fail validation so typos stay
visible.
If `plugins.enabled: false` is set, stale plugin references are treated as inert:
Gateway startup skips plugin discovery/load work and `openclaw doctor` preserves
the disabled plugin config instead of auto-removing it. Re-enable plugins before
running doctor cleanup if you want stale plugin ids removed.Packaged OpenClaw installs do not eagerly install every bundled plugin’s
runtime dependency tree. When a bundled OpenClaw-owned plugin is active from
plugin config, legacy channel config, or a default-enabled manifest, startup
repairs only that plugin’s declared runtime dependencies before importing it.
Persisted channel auth state alone does not activate a bundled channel for
Gateway startup runtime-dependency repair.
Explicit disablement still wins: `plugins.entries.<id>.enabled: false`,
`plugins.deny`, `plugins.enabled: false`, and `channels.<id>.enabled: false`
prevent automatic bundled runtime-dependency repair for that plugin/channel.
A non-empty `plugins.allow` also bounds default-enabled bundled runtime-dependency
repair; explicit bundled channel enablement (`channels.<id>.enabled: true`) can
still repair that channel’s plugin dependencies.
External plugins and custom load paths must still be installed through
`openclaw plugins install`.

## [​](https://docs.openclaw.ai/tools/plugin\\#plugin-types)  Plugin types

OpenClaw recognizes two plugin formats:

| Format | How it works | Examples |
| --- | --- | --- |
| **Native** | `openclaw.plugin.json` \\+ runtime module; executes in-process | Official plugins, community npm packages |
| **Bundle** | Codex/Claude/Cursor-compatible layout; mapped to OpenClaw features | `.codex-plugin/`, `.claude-plugin/`, `.cursor-plugin/` |

Both show up under `openclaw plugins list`. See [Plugin Bundles](https://docs.openclaw.ai/plugins/bundles) for bundle details.If you are writing a native plugin, start with [Building Plugins](https://docs.openclaw.ai/plugins/building-plugins)
and the [Plugin SDK Overview](https://docs.openclaw.ai/plugins/sdk-overview).

## [​](https://docs.openclaw.ai/tools/plugin\\#package-entrypoints)  Package entrypoints

Native plugin npm packages must declare `openclaw.extensions` in `package.json`.
Each entry must stay inside the package directory and resolve to a readable
runtime file, or to a TypeScript source file with an inferred built JavaScript
peer such as `src/index.ts` to `dist/index.js`.Use `openclaw.runtimeExtensions` when published runtime files do not live at the
same paths as the source entries. When present, `runtimeExtensions` must contain
exactly one entry for every `extensions` entry. Mismatched lists fail install and
plugin discovery rather than silently falling back to source paths.

```
{
  \"name\": \"@acme/openclaw-plugin\",
  \"openclaw\": {
    \"extensions\": [\"./src/index.ts\"],
    \"runtimeExtensions\": [\"./dist/index.js\"]
  }
}
```

## [​](https://docs.openclaw.ai/tools/plugin\\#official-plugins)  Official plugins

### [​](https://docs.openclaw.ai/tools/plugin\\#installable-npm)  Installable (npm)

| Plugin | Package | Docs |
| --- | --- | --- |
| Matrix | `@openclaw/matrix` | [Matrix](https://docs.openclaw.ai/channels/matrix) |
| Microsoft Teams | `@openclaw/msteams` | [Microsoft Teams](https://docs.openclaw.ai/channels/msteams) |
| Nostr | `@openclaw/nostr` | [Nostr](https://docs.openclaw.ai/channels/nostr) |
| Voice Call | `@openclaw/voice-call` | [Voice Call](https://docs.openclaw.ai/plugins/voice-call) |
| Zalo | `@openclaw/zalo` | [Zalo](https://docs.openclaw.ai/channels/zalo) |
| Zalo Personal | `@openclaw/zalouser` | [Zalo Personal](https://docs.openclaw.ai/plugins/zalouser) |

### [​](https://docs.openclaw.ai/tools/plugin\\#core-shipped-with-openclaw)  Core (shipped with OpenClaw)

Model providers (enabled by default)

`anthropic`, `byteplus`, `cloudflare-ai-gateway`, `github-copilot`, `google`,
`huggingface`, `kilocode`, `kimi-coding`, `minimax`, `mistral`, `qwen`,
`moonshot`, `nvidia`, `openai`, `opencode`, `opencode-go`, `openrouter`,
`qianfan`, `synthetic`, `together`, `venice`,
`vercel-ai-gateway`, `volcengine`, `xiaomi`, `zai`

Memory plugins

- `memory-core` — bundled memory search (default via `plugins.slots.memory`)
- `memory-lancedb` — install-on-demand long-term memory with auto-recall/capture (set `plugins.slots.memory = \"memory-lancedb\"`)

See [Memory LanceDB](https://docs.openclaw.ai/plugins/memory-lancedb) for OpenAI-compatible
embedding setup, Ollama examples, recall limits, and troubleshooting.

Speech providers (enabled by default)

`elevenlabs`, `microsoft`

Other

- `browser` — bundled browser plugin for the browser tool, `openclaw browser` CLI, `browser.request` gateway method, browser runtime, and default browser control service (enabled by default; disable before replacing it)
- `copilot-proxy` — VS Code Copilot Proxy bridge (disabled by default)

Looking for third-party plugins? See [Community Plugins](https://docs.openclaw.ai/plugins/community).

## [​](https://docs.openclaw.ai/tools/plugin\\#configuration)  Configuration

```
{
  plugins: {
    enabled: true,
    allow: [\"voice-call\"],
    deny: [\"untrusted-plugin\"],
    load: { paths: [\"~/Projects/oss/voice-call-plugin\"] },
    entries: {
      \"voice-call\": { enabled: true, config: { provider: \"twilio\" } },
    },
  },
}
```

| Field | Description |
| --- | --- |
| `enabled` | Master toggle (default: `true`) |
| `allow` | Plugin allowlist (optional) |
| `deny` | Plugin denylist (optional; deny wins) |
| `load.paths` | Extra plugin files/directories |
| `slots` | Exclusive slot selectors (e.g. `memory`, `contextEngine`) |
| `entries.\\<id\\>` | Per-plugin toggles + config |

Config changes **require a gateway restart**. If the Gateway is running with config
watch + in-process restart enabled (the default `openclaw gateway` path), that
restart is usually performed automatically a moment after the config write lands.
There is no supported hot-reload path for native plugin runtime code or lifecycle
hooks; restart the Gateway process that is serving the live channel before
expecting updated `register(api)` code, `api.on(...)` hooks, tools, services, or
provider/runtime hooks to run.`openclaw plugins list` is a local plugin registry/config snapshot. An
`enabled` plugin there means the persisted registry and current config allow the
plugin to participate. It does not prove that an already-running remote Gateway
child has restarted into the same plugin code. On VPS/container setups with
wrapper processes, send restarts to the actual `openclaw gateway run` process,
or use `openclaw gateway restart` against the running Gateway.

Plugin states: disabled vs missing vs invalid

- **Disabled**: plugin exists but enablement rules turned it off. Config is preserved.
- **Missing**: config references a plugin id that discovery did not find.
- **Invalid**: plugin exists but its config does not match the declared schema. Gateway startup skips only that plugin; `openclaw doctor --fix` can quarantine the invalid entry by disabling it and removing its config payload.

## [​](https://docs.openclaw.ai/tools/plugin\\#discovery-and-precedence)  Discovery and precedence

OpenClaw scans for plugins in this order (first match wins):

1

[Navigate to header](https://docs.openclaw.ai/tools/plugin#)

Config paths

`plugins.load.paths` — explicit file or directory paths. Paths that point
back at OpenClaw’s own packaged bundled plugin directories are ignored;
run `openclaw doctor --fix` to remove those stale aliases.

2

[Navigate to header](https://docs.openclaw.ai/tools/plugin#)

Workspace plugins

`\\<workspace\\>/.openclaw/<plugin-root>/*.ts` and `\\<workspace\\>/.openclaw/<plugin-root>/*/index.ts`.

3

[Navigate to header](https://docs.openclaw.ai/tools/plugin#)

Global plugins

`~/.openclaw/<plugin-root>/*.ts` and `~/.openclaw/<plugin-root>/*/index.ts`.

4

[Navigate to header](https://docs.openclaw.ai/tools/plugin#)

Bundled plugins

Shipped with OpenClaw. Many are enabled by default (model providers, speech).
Others require explicit enablement.

Packaged installs and Docker images normally resolve bundled plugins from the
compiled `dist/extensions` tree. If a bundled plugin source directory is
bind-mounted over the matching packaged source path, for example
`/app/extensions/synology-chat`, OpenClaw treats that mounted source directory
as a bundled source overlay and discovers it before the packaged
`/app/dist/extensions/synology-chat` bundle. This keeps maintainer container
loops working without switching every bundled plugin back to TypeScript source.
Set `OPENCLAW_DISABLE_BUNDLED_SOURCE_OVERLAYS=1` to force packaged dist bundles
even when source overlay mounts are present.

### [​](https://docs.openclaw.ai/tools/plugin\\#enablement-rules)  Enablement rules

- `plugins.enabled: false` disables all plugins and skips plugin discovery/load work
- `plugins.deny` always wins over allow
- `plugins.entries.\\<id\\>.enabled: false` disables that plugin
- Workspace-origin plugins are **disabled by default** (must be explicitly enabled)
- Bundled plugins follow the built-in default-on set unless overridden
- Exclusive slots can force-enable the selected plugin for that slot
- Some bundled opt-in plugins are enabled automatically when config names a
plugin-owned surface, such as a provider model ref, channel config, or harness
runtim
- Stale plugin config is preserved while `plugins.enabled: false` is active;
re-enable plugins before running doctor cleanup if you want stale ids removed
- OpenAI-family Codex routes keep separate plugin boundaries:
`openai-codex/*` belongs to the OpenAI plugin, while the bundled Codex
app-server plugin is selected by `agentRuntime.id: \"codex\"` or legacy
`codex/*` model refs

## [​](https://docs.openclaw.ai/tools/plugin\\#troubleshooting-runtime-hooks)  Troubleshooting runtime hooks

If a plugin appears in `plugins list` but `register(api)` side effects or hooks
do not run in live chat traffic, check these first:

- Run `openclaw gateway status --deep --require-rpc` and confirm the active
Gateway URL, profile, config path, and process are the ones you are editing.
- Restart the live Gateway after plugin install/config/code changes. In wrapper
containers, PID 1 may only be a supervisor; restart or signal the child
`openclaw gateway run` process.
- Use `openclaw plugins inspect <id> --json` to confirm hook registrations and
diagnostics. Non-bundled conversation hooks such as `llm_input`,
`llm_output`, `before_agent_finalize`, and `agent_end` need
`plugins.entries.<id>.hooks.allowConversationAccess=true`.
- For model switching, prefer `before_model_resolve`. It runs before model
resolution for agent turns; `llm_output` only runs after a model attempt
produces assistant output.
- For proof of the effective session model, use `openclaw sessions` or the
Gateway session/status surfaces and, when debugging provider payloads, start
the Gateway with `--raw-stream --raw-stream-path <path>`.

### [​](https://docs.openclaw.ai/tools/plugin\\#duplicate-channel-or-tool-ownership)  Duplicate channel or tool ownership

Symptoms:

- `channel already registered: <channel-id> (<plugin-id>)`
- `channel setup already registered: <channel-id> (<plugin-id>)`
- `plugin tool name conflict (<plugin-id>): <tool-name>`

These mean more than one enabled plugin is trying to own the same channel,
setup flow, or tool name. The most common cause is an external channel plugin
installed beside a bundled plugin that now provides the same channel id.Debug steps:

- Run `openclaw plugins list --enabled --verbose` to see every enabled plugin
and origin.
- Run `openclaw plugins inspect <id> --json` for each suspected plugin and
compare `channels`, `channelConfigs`, `tools`, and diagnostics.
- Run `openclaw plugins registry --refresh` after installing or removing
plugin packages so persisted metadata reflects the current install.
- Restart the Gateway after install, registry, or config changes.

Fix options:

- If one plugin intentionally replaces another for the same channel id, the
preferred plugin should declare `channelConfigs.<channel-id>.preferOver` with
the lower-priority plugin id. See [/plugins/manifest#replacing-another-channel-plugin](https://docs.openclaw.ai/plugins/manifest#replacing-another-channel-plugin).
- If the duplicate is accidental, disable one side with
`plugins.entries.<plugin-id>.enabled: false` or remove the stale plugin
install.
- If you explicitly enabled both plugins, OpenClaw keeps that request and
reports the conflict. Pick one owner for the channel or rename plugin-owned
tools so the runtime surface is unambiguous.

## [​](https://docs.openclaw.ai/tools/plugin\\#plugin-slots-exclusive-categories)  Plugin slots (exclusive categories)

Some categories are exclusive (only one active at a time):

```
{
  plugins: {
    slots: {
      memory: \"memory-core\", // or \"none\" to disable
      contextEngine: \"legacy\", // or a plugin id
    },
  },
}
```

| Slot | What it controls | Default |
| --- | --- | --- |
| `memory` | Active memory plugin | `memory-core` |
| `contextEngine` | Active context engine | `legacy` (built-in) |

## [​](https://docs.openclaw.ai/tools/plugin\\#cli-reference)  CLI reference

```
openclaw plugins list                       # compact inventory
openclaw plugins list --enabled            # only enabled plugins
openclaw plugins list --verbose            # per-plugin detail lines
openclaw plugins list --json               # machine-readable inventory
openclaw plugins inspect <id>              # deep detail
openclaw plugins inspect <id> --json       # machine-readable
openclaw plugins inspect --all             # fleet-wide table
openclaw plugins info <id>                 # inspect alias
openclaw plugins doctor                    # diagnostics
openclaw plugins registry                  # inspect persisted registry state
openclaw plugins registry --refresh        # rebuild persisted registry
openclaw doctor --fix                      # repair plugin registry state

openclaw plugins install <package>         # install (ClawHub first, then npm)
openclaw plugins install clawhub:<pkg>     # install from ClawHub only
openclaw plugins install npm:<pkg>         # install from npm only
openclaw plugins install <spec> --force    # overwrite existing install
openclaw plugins install <path>            # install from local path
openclaw plugins install -l <path>         # link (no copy) for dev
openclaw plugins install <plugin> --marketplace <source>
openclaw plu...(content truncated)

---

## Image generation
**Source:** https://docs.openclaw.ai/tools/image-generation

[Skip to main content](https://docs.openclaw.ai/tools/image-generation#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

Image generation

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Quick start](https://docs.openclaw.ai/tools/image-generation#quick-start)
- [Common routes](https://docs.openclaw.ai/tools/image-generation#common-routes)
- [Supported providers](https://docs.openclaw.ai/tools/image-generation#supported-providers)
- [Provider capabilities](https://docs.openclaw.ai/tools/image-generation#provider-capabilities)
- [Tool parameters](https://docs.openclaw.ai/tools/image-generation#tool-parameters)
- [Configuration](https://docs.openclaw.ai/tools/image-generation#configuration)
- [Model selection](https://docs.openclaw.ai/tools/image-generation#model-selection)
- [Provider selection order](https://docs.openclaw.ai/tools/image-generation#provider-selection-order)
- [Image editing](https://docs.openclaw.ai/tools/image-generation#image-editing)
- [Provider deep dives](https://docs.openclaw.ai/tools/image-generation#provider-deep-dives)
- [Examples](https://docs.openclaw.ai/tools/image-generation#examples)
- [Related](https://docs.openclaw.ai/tools/image-generation#related)

The `image_generate` tool lets the agent create and edit images using your
configured providers. Generated images are delivered automatically as media
attachments in the agent’s reply.

The tool only appears when at least one image-generation provider is
available. If you do not see `image_generate` in your agent’s tools,
configure `agents.defaults.imageGenerationModel`, set up a provider API key,
or sign in with OpenAI Codex OAuth.

## [​](https://docs.openclaw.ai/tools/image-generation\\#quick-start)  Quick start

1

[Navigate to header](https://docs.openclaw.ai/tools/image-generation#)

Configure auth

Set an API key for at least one provider (for example `OPENAI_API_KEY`,
`GEMINI_API_KEY`, `OPENROUTER_API_KEY`) or sign in with OpenAI Codex OAuth.

2

[Navigate to header](https://docs.openclaw.ai/tools/image-generation#)

Pick a default model (optional)

```
{
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"openai/gpt-image-2\",
        timeoutMs: 180_000,
      },
    },
  },
}
```

Codex OAuth uses the same `openai/gpt-image-2` model ref. When an
`openai-codex` OAuth profile is configured, OpenClaw routes image
requests through that OAuth profile instead of first trying
`OPENAI_API_KEY`. Explicit `models.providers.openai` config (API key,
custom/Azure base URL) opts back into the direct OpenAI Images API
route.

3

[Navigate to header](https://docs.openclaw.ai/tools/image-generation#)

Ask the agent

_“Generate an image of a friendly robot mascot.”_The agent calls `image_generate` automatically. No tool allow-listing
needed — it is enabled by default when a provider is available.

For OpenAI-compatible LAN endpoints such as LocalAI, keep the custom
`models.providers.openai.baseUrl` and explicitly opt in with
`browser.ssrfPolicy.dangerouslyAllowPrivateNetwork: true`. Private and
internal image endpoints remain blocked by default.

## [​](https://docs.openclaw.ai/tools/image-generation\\#common-routes)  Common routes

| Goal | Model ref | Auth |
| --- | --- | --- |
| OpenAI image generation with API billing | `openai/gpt-image-2` | `OPENAI_API_KEY` |
| OpenAI image generation with Codex subscription auth | `openai/gpt-image-2` | OpenAI Codex OAuth |
| OpenAI transparent-background PNG/WebP | `openai/gpt-image-1.5` | `OPENAI_API_KEY` or OpenAI Codex OAuth |
| DeepInfra image generation | `deepinfra/black-forest-labs/FLUX-1-schnell` | `DEEPINFRA_API_KEY` |
| OpenRouter image generation | `openrouter/google/gemini-3.1-flash-image-preview` | `OPENROUTER_API_KEY` |
| LiteLLM image generation | `litellm/gpt-image-2` | `LITELLM_API_KEY` |
| Google Gemini image generation | `google/gemini-3.1-flash-image-preview` | `GEMINI_API_KEY` or `GOOGLE_API_KEY` |

The same `image_generate` tool handles text-to-image and reference-image
editing. Use `image` for one reference or `images` for multiple references.
Provider-supported output hints such as `quality`, `outputFormat`, and
`background` are forwarded when available and reported as ignored when a
provider does not support them. Bundled transparent-background support is
OpenAI-specific; other providers may still preserve PNG alpha if their
backend emits it.

## [​](https://docs.openclaw.ai/tools/image-generation\\#supported-providers)  Supported providers

| Provider | Default model | Edit support | Auth |
| --- | --- | --- | --- |
| ComfyUI | `workflow` | Yes (1 image, workflow-configured) | `COMFY_API_KEY` or `COMFY_CLOUD_API_KEY` for cloud |
| DeepInfra | `black-forest-labs/FLUX-1-schnell` | Yes (1 image) | `DEEPINFRA_API_KEY` |
| fal | `fal-ai/flux/dev` | Yes | `FAL_KEY` |
| Google | `gemini-3.1-flash-image-preview` | Yes | `GEMINI_API_KEY` or `GOOGLE_API_KEY` |
| LiteLLM | `gpt-image-2` | Yes (up to 5 input images) | `LITELLM_API_KEY` |
| MiniMax | `image-01` | Yes (subject reference) | `MINIMAX_API_KEY` or MiniMax OAuth (`minimax-portal`) |
| OpenAI | `gpt-image-2` | Yes (up to 4 images) | `OPENAI_API_KEY` or OpenAI Codex OAuth |
| OpenRouter | `google/gemini-3.1-flash-image-preview` | Yes (up to 5 input images) | `OPENROUTER_API_KEY` |
| Vydra | `grok-imagine` | No | `VYDRA_API_KEY` |
| xAI | `grok-imagine-image` | Yes (up to 5 images) | `XAI_API_KEY` |

Use `action: \"list\"` to inspect available providers and models at runtime:

```
/tool image_generate action=list
```

## [​](https://docs.openclaw.ai/tools/image-generation\\#provider-capabilities)  Provider capabilities

| Capability | ComfyUI | DeepInfra | fal | Google | MiniMax | OpenAI | Vydra | xAI |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Generate (max count) | Workflow-defined | 4 | 4 | 4 | 9 | 4 | 1 | 4 |
| Edit / reference | 1 image (workflow) | 1 image | 1 image | Up to 5 images | 1 image (subject ref) | Up to 5 images | — | Up to 5 images |
| Size control | — | ✓ | ✓ | ✓ | — | Up to 4K | — | — |
| Aspect ratio | — | — | ✓ (generate only) | ✓ | ✓ | — | — | ✓ |
| Resolution (1K/2K/4K) | — | — | ✓ | ✓ | — | — | — | 1K, 2K |

## [​](https://docs.openclaw.ai/tools/image-generation\\#tool-parameters)  Tool parameters

[​](https://docs.openclaw.ai/tools/image-generation#param-prompt)

prompt

string

required

Image generation prompt. Required for `action: \"generate\"`.

[​](https://docs.openclaw.ai/tools/image-generation#param-action)

action

\"generate\" \\| \"list\"

default:\"generate\"

Use `\"list\"` to inspect available providers and models at runtime.

[​](https://docs.openclaw.ai/tools/image-generation#param-model)

model

string

Provider/model override (e.g. `openai/gpt-image-2`). Use
`openai/gpt-image-1.5` for transparent OpenAI backgrounds.

[​](https://docs.openclaw.ai/tools/image-generation#param-image)

image

string

Single reference image path or URL for edit mode.

[​](https://docs.openclaw.ai/tools/image-generation#param-images)

images

string\\[\\]

Multiple reference images for edit mode (up to 5 on supporting providers).

[​](https://docs.openclaw.ai/tools/image-generation#param-size)

size

string

Size hint: `1024x1024`, `1536x1024`, `1024x1536`, `2048x2048`, `3840x2160`.

[​](https://docs.openclaw.ai/tools/image-generation#param-aspect-ratio)

aspectRatio

string

Aspect ratio: `1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `4:5`, `5:4`, `9:16`, `16:9`, `21:9`.

[​](https://docs.openclaw.ai/tools/image-generation#param-resolution)

resolution

\"1K\" \\| \"2K\" \\| \"4K\"

Resolution hint.

[​](https://docs.openclaw.ai/tools/image-generation#param-quality)

quality

\"low\" \\| \"medium\" \\| \"high\" \\| \"auto\"

Quality hint when the provider supports it.

[​](https://docs.openclaw.ai/tools/image-generation#param-output-format)

outputFormat

\"png\" \\| \"jpeg\" \\| \"webp\"

Output format hint when the provider supports it.

[​](https://docs.openclaw.ai/tools/image-generation#param-background)

background

\"transparent\" \\| \"opaque\" \\| \"auto\"

Background hint when the provider supports it. Use `transparent` with
`outputFormat: \"png\"` or `\"webp\"` for transparency-capable providers.

[​](https://docs.openclaw.ai/tools/image-generation#param-count)

count

number

Number of images to generate (1–4).

[​](https://docs.openclaw.ai/tools/image-generation#param-timeout-ms)

timeoutMs

number

Optional provider request timeout in milliseconds.

[​](https://docs.openclaw.ai/tools/image-generation#param-filename)

filename

string

Output filename hint.

[​](https://docs.openclaw.ai/tools/image-generation#param-openai)

openai

object

OpenAI-only hints: `background`, `moderation`, `outputCompression`, and `user`.

Not all providers support all parameters. When a fallback provider supports a
nearby geometry option instead of the exact requested one, OpenClaw remaps to
the closest supported size, aspect ratio, or resolution before submission.
Unsupported output hints are dropped for providers that do not declare
support and reported in the tool result. Tool results report the applied
settings; `details.normalization` captures any requested-to-applied
translation.

## [​](https://docs.openclaw.ai/tools/image-generation\\#configuration)  Configuration

### [​](https://docs.openclaw.ai/tools/image-generation\\#model-selection)  Model selection

```
{
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"openai/gpt-image-2\",
        timeoutMs: 180_000,
        fallbacks: [\\\
          \"openrouter/google/gemini-3.1-flash-image-preview\",\\\
          \"google/gemini-3.1-flash-image-preview\",\\\
          \"fal/fal-ai/flux/dev\",\\\
        ],
      },
    },
  },
}
```

### [​](https://docs.openclaw.ai/tools/image-generation\\#provider-selection-order)  Provider selection order

OpenClaw tries providers in this order:

1. **`model` parameter** from the tool call (if the agent specifies one).
2. **`imageGenerationModel.primary`** from config.
3. **`imageGenerationModel.fallbacks`** in order.
4. **Auto-detection**— auth-backed provider defaults only:

   - current default provider first;
   - remaining registered image-generation providers in provider-id order.

If a provider fails (auth error, rate limit, etc.), the next configured
candidate is tried automatically. If all fail, the error includes details
from each attempt.

Per-call model overrides are exact

A per-call `model` override tries only that provider/model and does
not continue to configured primary/fallback or auto-detected providers.

Auto-detection is auth-aware

A provider default only enters the candidate list when OpenClaw can
actually authenticate that provider. Set
`agents.defaults.mediaGenerationAutoProviderFallback: false` to use only
explicit `model`, `primary`, and `fallbacks` entries.

Timeouts

Set `agents.defaults.imageGenerationModel.timeoutMs` for slow image
backends. A per-call `timeoutMs` tool parameter overrides the configured
default.

Inspect at runtime

Use `action: \"list\"` to inspect the currently registered providers,
their default models, and auth env-var hints.

### [​](https://docs.openclaw.ai/tools/image-generation\\#image-editing)  Image editing

OpenAI, OpenRouter, Google, DeepInfra, fal, MiniMax, ComfyUI, and xAI support editing
reference images. Pass a reference image path or URL:

```
\"Generate a watercolor version of this photo\" + image: \"/path/to/photo.jpg\"
```

OpenAI, OpenRouter, Google, and xAI support up to 5 reference images via the
`images` parameter. fal, MiniMax, and ComfyUI support 1.

## [​](https://docs.openclaw.ai/tools/image-generation\\#provider-deep-dives)  Provider deep dives

OpenAI gpt-image-2 (and gpt-image-1.5)

OpenAI image generation defaults to `openai/gpt-image-2`. If an
`openai-codex` OAuth profile is configured, OpenClaw reuses the same
OAuth profile used by Codex subscription chat models and sends the
image request through the Codex Responses backend. Legacy Codex base
URLs such as `https://chatgpt.com/backend-api` are canonicalized to
`https://chatgpt.com/backend-api/codex` for image requests. OpenClaw
does **not** silently fall back to `OPENAI_API_KEY` for that request —
to force direct OpenAI Images API routing, configure
`models.providers.openai` explicitly with an API key, custom base URL,
or Azure endpoint.The `openai/gpt-image-1.5`, `openai/gpt-image-1`, and
`openai/gpt-image-1-mini` models can still be selected explicitly. Use
`gpt-image-1.5` for transparent-background PNG/WebP output; the current
`gpt-image-2` API rejects `background: \"transparent\"`.`gpt-image-2` supports both text-to-image generation and
reference-image editing through the same `image_generate` tool.
OpenClaw forwards `prompt`, `count`, `size`, `quality`, `outputFormat`,
and reference images to OpenAI. OpenAI does **not** receive
`aspectRatio` or `resolution` directly; when possible OpenClaw maps
those into a supported `size`, otherwise the tool reports them as
ignored overrides.OpenAI-specific options live under the `openai` object:

```
{
  \"quality\": \"low\",
  \"outputFormat\": \"jpeg\",
  \"openai\": {
    \"background\": \"opaque\",
    \"moderation\": \"low\",
    \"outputCompression\": 60,
    \"user\": \"end-user-42\"
  }
}
```

`openai.background` accepts `transparent`, `opaque`, or `auto`;
transparent outputs require `outputFormat``png` or `webp` and a
transparency-capable OpenAI image model. OpenClaw routes default
`gpt-image-2` transparent-background requests to `gpt-image-1.5`.
`openai.outputCompression` applies to JPEG/WebP outputs.The top-level `background` hint is provider-neutral and currently maps
to the same OpenAI `background` request field when the OpenAI provider
is selected. Providers that do not declare background support return
it in `ignoredOverrides` instead of receiving the unsupported parameter.To route OpenAI image generation through an Azure OpenAI deployment
instead of `api.openai.com`, see
[Azure OpenAI endpoints](https://docs.openclaw.ai/providers/openai#azure-openai-endpoints).

OpenRouter image models

OpenRouter image generation uses the same `OPENROUTER_API_KEY` and
routes through OpenRouter’s chat completions image API. Select
OpenRouter image models with the `openrouter/` prefix:

```
{
  agents: {
    defaults: {
      imageGenerationModel: {
        primary: \"openrouter/google/gemini-3.1-flash-image-preview\",
      },
    },
  },
}
```

OpenClaw forwards `prompt`, `count`, reference images, and
Gemini-compatible `aspectRatio` / `resolution` hints to OpenRouter.
Current built-in OpenRouter image model shortcuts include
`google/gemini-3.1-flash-image-preview`,
`google/gemini-3-pro-image-preview`, and `openai/gpt-5.4-image-2`. Use
`action: \"list\"` to see what your configured plugin exposes.

MiniMax dual-auth

MiniMax image generation is available through both bundled MiniMax
auth paths:

- `minimax/image-01` for API-key setups
- `minimax-portal/image-01` for OAuth setups

xAI grok-imagine-image

The bundled xAI provider uses `/v1/images/generations` for prompt-only
requests and `/v1/images/edits` when `image` or `images` is present.

- Models: `xai/grok-imagine-image`, `xai/grok-imagine-image-pro`
- Count: up to 4
- References: one `image` or up to five `images`
- Aspect ratios: `1:1`, `16:9`, `9:16`, `4:3`, `3:4`, `2:3`, `3:2`
- Resolutions: `1K`, `2K`
- Outputs: returned as OpenClaw-managed image attachments

OpenClaw intentionally does not expose xAI-native `quality`, `mask`,
`user`, or extra native-only aspect ratios until those controls exist
in the shared cross-provider `image_generate` contract.

## [​](https://docs.openclaw.ai/tools/image-generation\\#examples)  Examples

- Generate (4K landscape)

- Generate (transparent PNG)

- Generate (two square)

- Edit (one reference)

- Edit (multiple references)


```
/tool image_generate action=generate model=openai/gpt-image-2 prompt=\"A clean editorial poster for OpenClaw image generation\" size=3840x2160 count=1
```

```
/tool image_generate action=generate model=openai/gpt-image-1.5 prompt=\"A simple red circle sticker on a transparent background\" outputFormat=png background=transparent
```

Equivalent CLI:

```
openclaw infer image generate \\\
  --model openai/gpt-image-1.5 \\\
  --output-format png \\\
  --background transparent \\\
  --prompt \"A simple red circle sticker on a transparent background\" \\\
  --json
```

```
/tool image_generate action=generate model=openai/gpt-image-2 prompt=\"Two visual directions for a calm productivity app icon\" size=1024x1024 count=2
```

```
/tool image_generate action=generate model=openai/gpt-image-2 prompt=\"Keep the subject, replace the background with a bright studio setup\" image=/path/to/reference.png size=1024x1536
```

```
/tool image_generate action=generate model=openai/gpt-image-2 prompt=\"Combine the character identity from the first image with the color palette from the second\" images=\'[\"/path/to/character.png\",\"/path/to/palette.jpg\"]\' size=1536x1024
```

The same `--output-format` and `--background` flags are available on
`openclaw infer image edit`; `--openai-background` remains as an
OpenAI-specific alias. Bundled providers other than OpenAI do not declare
explicit background control today, so `background: \"transparent\"` is reported
as ignored for them.

## [​](https://docs.openclaw.ai/tools/image-generation\\#related)  Related

- [Tools overview](https://docs.openclaw.ai/tools) — all available agent tools
- [ComfyUI](https://docs.openclaw.ai/providers/comfy) — local ComfyUI and Comfy Cloud workflow setup
- [fal](https://docs.openclaw.ai/providers/fal) — fal image and video provider setup
- [Google (Gemini)](https://docs.openclaw.ai/providers/google) — Gemini image provider setup
- [MiniMax](https://docs.openclaw.ai/providers/minimax) — MiniMax image provider setup
- [OpenAI](https://docs.openclaw.ai/providers/openai) — OpenAI Images provider setup
- [Vydra](https://docs.openclaw.ai/providers/vydra) — Vydra image, video, and speech setup
- [xAI](https://docs.openclaw.ai/providers/xai) — Grok image, video, search, code execution, and TTS setup
- [Configuration reference](https://docs.openclaw.ai/gateway/config-agents#agent-defaults) — `imageGenerationModel` c...

---

## Exec approvals
**Source:** https://docs.openclaw.ai/tools/exec-approvals

[Skip to main content](https://docs.openclaw.ai/tools/exec-approvals#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

Exec approvals

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Inspecting the effective policy](https://docs.openclaw.ai/tools/exec-approvals#inspecting-the-effective-policy)
- [Where it applies](https://docs.openclaw.ai/tools/exec-approvals#where-it-applies)
- [Trust model](https://docs.openclaw.ai/tools/exec-approvals#trust-model)
- [macOS split](https://docs.openclaw.ai/tools/exec-approvals#macos-split)
- [Settings and storage](https://docs.openclaw.ai/tools/exec-approvals#settings-and-storage)
- [Policy knobs](https://docs.openclaw.ai/tools/exec-approvals#policy-knobs)
- [exec.security](https://docs.openclaw.ai/tools/exec-approvals#exec-security)
- [exec.ask](https://docs.openclaw.ai/tools/exec-approvals#exec-ask)
- [askFallback](https://docs.openclaw.ai/tools/exec-approvals#askfallback)
- [tools.exec.strictInlineEval](https://docs.openclaw.ai/tools/exec-approvals#tools-exec-strictinlineeval)
- [YOLO mode (no-approval)](https://docs.openclaw.ai/tools/exec-approvals#yolo-mode-no-approval)
- [Persistent gateway-host “never prompt” setup](https://docs.openclaw.ai/tools/exec-approvals#persistent-gateway-host-%E2%80%9Cnever-prompt%E2%80%9D-setup)
- [Local shortcut](https://docs.openclaw.ai/tools/exec-approvals#local-shortcut)
- [Node host](https://docs.openclaw.ai/tools/exec-approvals#node-host)
- [Session-only shortcut](https://docs.openclaw.ai/tools/exec-approvals#session-only-shortcut)
- [Allowlist (per agent)](https://docs.openclaw.ai/tools/exec-approvals#allowlist-per-agent)
- [Auto-allow skill CLIs](https://docs.openclaw.ai/tools/exec-approvals#auto-allow-skill-clis)
- [Safe bins and approval forwarding](https://docs.openclaw.ai/tools/exec-approvals#safe-bins-and-approval-forwarding)
- [Control UI editing](https://docs.openclaw.ai/tools/exec-approvals#control-ui-editing)
- [Approval flow](https://docs.openclaw.ai/tools/exec-approvals#approval-flow)
- [System events](https://docs.openclaw.ai/tools/exec-approvals#system-events)
- [Denied approval behavior](https://docs.openclaw.ai/tools/exec-approvals#denied-approval-behavior)
- [Implications](https://docs.openclaw.ai/tools/exec-approvals#implications)
- [Related](https://docs.openclaw.ai/tools/exec-approvals#related)

Exec approvals are the **companion app / node host guardrail** for letting
a sandboxed agent run commands on a real host (`gateway` or `node`). A
safety interlock: commands are allowed only when policy + allowlist +
(optional) user approval all agree. Exec approvals stack **on top of**
tool policy and elevated gating (unless elevated is set to `full`, which
skips approvals).

Effective policy is the **stricter** of `tools.exec.*` and approvals
defaults; if an approvals field is omitted, the `tools.exec` value is
used. Host exec also uses local approvals state on that machine — a
host-local `ask: \"always\"` in `~/.openclaw/exec-approvals.json` keeps
prompting even if session or config defaults request `ask: \"on-miss\"`.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#inspecting-the-effective-policy)  Inspecting the effective policy

| Command | What it shows |
| --- | --- |
| `openclaw approvals get` / `--gateway` / `--node <id|name|ip>` | Requested policy, host policy sources, and the effective result. |
| `openclaw exec-policy show` | Local-machine merged view. |
| `openclaw exec-policy set` / `preset` | Synchronize the local requested policy with the local host approvals file in one step. |

When a local scope requests `host=node`, `exec-policy show` reports that
scope as node-managed at runtime instead of pretending the local
approvals file is the source of truth.If the companion app UI is **not available**, any request that would
normally prompt is resolved by the **ask fallback** (default: `deny`).

Native chat approval clients can seed channel-specific affordances on the
pending approval message. For example, Matrix seeds reaction shortcuts
(`✅` allow once, `❌` deny, `♾️` allow always) while still leaving
`/approve ...` commands in the message as a fallback.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#where-it-applies)  Where it applies

Exec approvals are enforced locally on the execution host:

- **Gateway host** → `openclaw` process on the gateway machine.
- **Node host** → node runner (macOS companion app or headless node host).

### [​](https://docs.openclaw.ai/tools/exec-approvals\\#trust-model)  Trust model

- Gateway-authenticated callers are trusted operators for that Gateway.
- Paired nodes extend that trusted operator capability onto the node host.
- Exec approvals reduce accidental execution risk, but are **not** a per-user auth boundary.
- Approved node-host runs bind canonical execution context: canonical cwd, exact argv, env binding when present, and pinned executable path when applicable.
- For shell scripts and direct interpreter/runtime file invocations, OpenClaw also tries to bind one concrete local file operand. If that bound file changes after approval but before execution, the run is denied instead of executing drifted content.
- File binding is intentionally best-effort, **not** a complete semantic model of every interpreter/runtime loader path. If approval mode cannot identify exactly one concrete local file to bind, it refuses to mint an approval-backed run instead of pretending full coverage.

### [​](https://docs.openclaw.ai/tools/exec-approvals\\#macos-split)  macOS split

- The **node host service** forwards `system.run` to the **macOS app** over local IPC.
- The **macOS app** enforces approvals and executes the command in UI context.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#settings-and-storage)  Settings and storage

Approvals live in a local JSON file on the execution host:

```
~/.openclaw/exec-approvals.json
```

Example schema:

```
{
  \"version\": 1,
  \"socket\": {
    \"path\": \"~/.openclaw/exec-approvals.sock\",
    \"token\": \"base64url-token\"
  },
  \"defaults\": {
    \"security\": \"deny\",
    \"ask\": \"on-miss\",
    \"askFallback\": \"deny\",
    \"autoAllowSkills\": false
  },
  \"agents\": {
    \"main\": {
      \"security\": \"allowlist\",
      \"ask\": \"on-miss\",
      \"askFallback\": \"deny\",
      \"autoAllowSkills\": true,
      \"allowlist\": [\\\n        {\\\n          \"id\": \"B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F\",\\\n          \"pattern\": \"~/Projects/**/bin/rg\",\\\n          \"source\": \"allow-always\",\\\n          \"commandText\": \"rg -n TODO\",\\\n          \"lastUsedAt\": 1737150000000,\\\n          \"lastUsedCommand\": \"rg -n TODO\",\\\n          \"lastResolvedPath\": \"/Users/user/Projects/.../bin/rg\"\\\n        }\\\n      ]
    }
  }
}
```

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#policy-knobs)  Policy knobs

### [​](https://docs.openclaw.ai/tools/exec-approvals\\#exec-security)  `exec.security`

[​](https://docs.openclaw.ai/tools/exec-approvals#param-security)

security

\"deny\" \\| \"allowlist\" \\| \"full\"

- `deny` — block all host exec requests.
- `allowlist` — allow only allowlisted commands.
- `full` — allow everything (equivalent to elevated).

### [​](https://openclaw.ai/tools/exec-approvals\\#exec-ask)  `exec.ask`

[​](https://docs.openclaw.ai/tools/exec-approvals#param-ask)

ask

\"off\" \\| \"on-miss\" \\| \"always\"

- `off` — never prompt.
- `on-miss` — prompt only when the allowlist does not match.
- `always` — prompt on every command. `allow-always` durable trust does **not** suppress prompts when effective ask mode is `always`.

### [​](https://docs.openclaw.ai/tools/exec-approvals\\#askfallback)  `askFallback`

[​](https://docs.openclaw.ai/tools/exec-approvals#param-ask-fallback)

askFallback

\"deny\" \\| \"allowlist\" \\| \"full\"

Resolution when a prompt is required but no UI is reachable.

- `deny` — block.
- `allowlist` — allow only if allowlist matches.
- `full` — allow.

### [​](https://docs.openclaw.ai/tools/exec-approvals\\#tools-exec-strictinlineeval)  `tools.exec.strictInlineEval`

[​](https://docs.openclaw.ai/tools/exec-approvals#param-strict-inline-eval)

strictInlineEval

boolean

When `true`, OpenClaw treats inline code-eval forms as approval-only
even if the interpreter binary itself is allowlisted. Defense-in-depth
for interpreter loaders that do not map cleanly to one stable file
operand.

Examples that strict mode catches:

- `python -c`
- `node -e`, `node --eval`, `node -p`
- `ruby -e`
- `perl -e`, `perl -E`
- `php -r`
- `lua -e`
- `osascript -e`

In strict mode these commands still need explicit approval, and
`allow-always` does not persist new allowlist entries for them
automatically.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#yolo-mode-no-approval)  YOLO mode (no-approval)

If you want host exec to run without approval prompts, you must open
**both** policy layers — requested exec policy in OpenClaw config
(`tools.exec.*`) **and** host-local approvals policy in
`~/.openclaw/exec-approvals.json`.YOLO is the default host behavior unless you tighten it explicitly:

| Layer | YOLO setting |
| --- | --- |
| `tools.exec.security` | `full` on `gateway`/`node` |
| `tools.exec.ask` | `off` |
| Host `askFallback` | `full` |

**Important distinctions:**

- `tools.exec.host=auto` chooses **where** exec runs: sandbox when available, otherwise gateway.
- YOLO chooses **how** host exec is approved: `security=full` plus `ask=off`.
- In YOLO mode, OpenClaw does **not** add a separate heuristic command-obfuscation approval gate or script-preflight rejection layer on top of the configured host exec policy.
- `auto` does not make gateway routing a free override from a sandboxed session. A per-call `host=node` request is allowed from `auto`; `host=gateway` is only allowed from `auto` when no sandbox runtime is active. For a stable non-auto default, set `tools.exec.host` or use `/exec host=...` explicitly.

CLI-backed providers that expose their own noninteractive permission mode
can follow this policy. Claude CLI adds
`--permission-mode bypassPermissions` when OpenClaw’s requested exec
policy is YOLO. Override that backend behavior with explicit Claude args
under `agents.defaults.cliBackends.claude-cli.args` / `resumeArgs` —
for example `--permission-mode default`, `acceptEdits`, or
`bypassPermissions`.If you want a more conservative setup, tighten either layer back to
`allowlist` / `on-miss` or `deny`.

### [​](https://docs.openclaw.ai/tools/exec-approvals\\#persistent-gateway-host-%E2%80%9Cnever-prompt%E2%80%9D-setup)  Persistent gateway-host “never prompt” setup

1

[Navigate to header](https://docs.openclaw.ai/tools/exec-approvals#)

Set the requested config policy

```
openclaw config set tools.exec.host gateway
openclaw config set tools.exec.security full
openclaw config set tools.exec.ask off
openclaw gateway restart
```

2

[Navigate to header](https://docs.openclaw.ai/tools/exec-approvals#)

Match the host approvals file

```
openclaw approvals set --stdin <<\'EOF\'
{
  version: 1,
  defaults: {
    security: \"full\",
    ask: \"off\",
    askFallback: \"full\"
  }
}
EOF
```

### [​](https://docs.openclaw.ai/tools/exec-approvals\\#local-shortcut)  Local shortcut

```
openclaw exec-policy preset yolo
```

That local shortcut updates both:

- Local `tools.exec.host/security/ask`.
- Local `~/.openclaw/exec-approvals.json` defaults.

It is intentionally local-only. To change gateway-host or node-host
approvals remotely, use `openclaw approvals set --gateway` or
`openclaw approvals set --node <id|name|ip>`.

### [​](https://openclaw.ai/tools/exec-approvals\\#node-host)  Node host

For a node host, apply the same approvals file on that node instead:

```
openclaw approvals set --node <id|name|ip> --stdin <<\'EOF\'
{
  version: 1,
  defaults: {
    security: \"full\",
    ask: \"off\",
    askFallback: \"full\"
  }
}
EOF
```

**Local-only limitations:**

- `openclaw exec-policy` does not synchronize node approvals.
- `openclaw exec-policy set --host node` is rejected.
- Node exec approvals are fetched from the node at runtime, so node-targeted updates must use `openclaw approvals --node ...`.

### [​](https://docs.openclaw.ai/tools/exec-approvals\\#session-only-shortcut)  Session-only shortcut

- `/exec security=full ask=off` changes only the current session.
- `/elevated full` is a break-glass shortcut that also skips exec approvals for that session.

If the host approvals file stays stricter than config, the stricter host
policy still wins.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#allowlist-per-agent)  Allowlist (per agent)

Allowlists are **per agent**. If multiple agents exist, switch which agent
you are editing in the macOS app. Patterns are glob matches.Patterns can be resolved binary path globs or bare command-name globs.
Bare names match only commands invoked through `PATH`, so `rg` can match
`/opt/homebrew/bin/rg` when the command is `rg`, but **not**`./rg` or
`/tmp/rg`. Use a path glob when you want to trust one specific binary
location.Legacy `agents.default` entries are migrated to `agents.main` on load.
Shell chains such as `echo ok && pwd` still need every top-level segment
to satisfy allowlist rules.Examples:

- `rg`
- `~/Projects/**/bin/peekaboo`
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

Each allowlist entry tracks:

| Field | Meaning |
| --- | --- |
| `id` | Stable UUID used for UI identity |
| `lastUsedAt` | Last-used timestamp |
| `lastUsedCommand` | Last command that matched |
| `lastResolvedPath` | Last resolved binary path |

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#auto-allow-skill-clis)  Auto-allow skill CLIs

When **Auto-allow skill CLIs** is enabled, executables referenced by
known skills are treated as allowlisted on nodes (macOS node or headless
node host). This uses `skills.bins` over the Gateway RPC to fetch the
skill bin list. Disable this if you want strict manual allowlists.

- This is an **implicit convenience allowlist**, separate from manual path allowlist entries.
- It is intended for trusted operator environments where Gateway and node are in the same trust boundary.
- If you require strict explicit trust, keep `autoAllowSkills: false` and use manual path allowlist entries only.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#safe-bins-and-approval-forwarding)  Safe bins and approval forwarding

For safe bins (the stdin-only fast-path), interpreter binding details, and
how to forward approval prompts to Slack/Discord/Telegram (or run them as
native approval clients), see
[Exec approvals — advanced](https://docs.openclaw.ai/tools/exec-approvals-advanced).

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#control-ui-editing)  Control UI editing

Use the **Control UI → Nodes → Exec approvals** card to edit defaults,
per-agent overrides, and allowlists. Pick a scope (Defaults or an agent),
tweak the policy, add/remove allowlist patterns, then **Save**. The UI
shows last-used metadata per pattern so you can keep the list tidy.The target selector chooses **Gateway** (local approvals) or a **Node**.
Nodes must advertise `system.execApprovals.get/set` (macOS app or
headless node host). If a node does not advertise exec approvals yet,
edit its local `~/.openclaw/exec-approvals.json` directly.CLI: `openclaw approvals` supports gateway or node editing — see
[Approvals CLI](https://docs.openclaw.ai/cli/approvals).

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#approval-flow)  Approval flow

When a prompt is required, the gateway broadcasts
`exec.approval.requested` to operator clients. The Control UI and macOS
app resolve it via `exec.approval.resolve`, then the gateway forwards the
approved request to the node host.For `host=node`, approval requests include a canonical `systemRunPlan`
payload. The gateway uses that plan as the authoritative
command/cwd/session context when forwarding approved `system.run`
requests.That matters for async approval latency:

- The node exec path prepares one canonical plan up front.
- The approval record stores that plan and its binding metadata.
- Once approved, the final forwarded `system.run` call reuses the stored plan instead of trusting later caller edits.
- If the caller changes `command`, `rawCommand`, `cwd`, `agentId`, or `sessionKey` after the approval request was created, the gateway rejects the forwarded run as an approval mismatch.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#system-events)  System events

Exec lifecycle is surfaced as system messages:

- `Exec running` (only if the command exceeds the running notice threshold).
- `Exec finished`.
- `Exec denied`.

These are posted to the agent’s session after the node reports the event.
Gateway-host exec approvals emit the same lifecycle events when the
command finishes (and optionally when running longer than the threshold).
Approval-gated execs reuse the approval id as the `runId` in these
messages for easy correlation.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#denied-approval-behavior)  Denied approval behavior

When an async exec approval is denied, OpenClaw prevents the agent from
reusing output from any earlier run of the same command in the session.
The denial reason is passed with explicit guidance that no command output
is available, which stops the agent from claiming there is new output or
repeating the denied command with stale results from a prior successful
run.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#implications)  Implications

- **`full`** is powerful; prefer allowlists when possible.
- **`ask`** keeps you in the loop while still allowing fast approvals.
- Per-agent allowlists prevent one agent’s approvals from leaking into others.
- Approvals only apply to host exec requests from **authorized senders**. Unauthorized senders cannot issue `/exec`.
- `/exec security=full` is a session-level convenience for authorized operators and skips approvals by design. To hard-block host exec, set approvals security to `deny` or deny the `exec` tool via tool policy.

## [​](https://docs.openclaw.ai/tools/exec-approvals\\#related)  Related

[**Exec approvals — advanced** \\\\\n\\\\\nSafe bins, interpreter binding, and approval forwarding to chat.](https://docs.openclaw.ai/tools/exec-approvals-advanced)



---

## apply_patch tool - OpenClaw
**Source:** https://docs.openclaw.ai/tools/apply-patch

[Skip to main content](https://docs.openclaw.ai/tools/apply-patch#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Tools

apply\_patch tool

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Parameters](https://docs.openclaw.ai/tools/apply-patch#parameters)
- [Notes](https://docs.openclaw.ai/tools/apply-patch#notes)
- [Example](https://docs.openclaw.ai/tools/apply-patch#example)
- [Related](https://docs.openclaw.ai/tools/apply-patch#related)

Apply file changes using a structured patch format. This is ideal for multi-file
or multi-hunk edits where a single `edit` call would be brittle.The tool accepts a single `input` string that wraps one or more file operations:

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```

## [​](https://docs.openclaw.ai/tools/apply-patch\#parameters)  Parameters

- `input` (required): Full patch contents including `*** Begin Patch` and `*** End Patch`.

## [​](https://docs.openclaw.ai/tools/apply-patch\#notes)  Notes

- Patch paths support relative paths (from the workspace directory) and absolute paths.
- `tools.exec.applyPatch.workspaceOnly` defaults to `true` (workspace-contained). Set it to `false` only if you intentionally want `apply_patch` to write/delete outside the workspace directory.
- Use `*** Move to:` within an `*** Update File:` hunk to rename files.
- `*** End of File` marks an EOF-only insert when needed.
- Available by default for OpenAI and OpenAI Codex models. Set
`tools.exec.applyPatch.enabled: false` to disable it.
- Optionally gate by model via
`tools.exec.applyPatch.allowModels`.
- Config is only under `tools.exec`.

## [​](https://docs.openclaw.ai/tools/apply-patch\#example)  Example

```
{
  \"tool\": \"apply_patch\",
  \"input\": \"*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch\"
}
```

## [​](https://docs.openclaw.ai/tools/apply-patch\#related)  Related

- [Diffs](https://docs.openclaw.ai/tools/diffs)
- [Exec tool](https://docs.openclaw.ai/tools/exec)
- [Code execution](https://docs.openclaw.ai/tools/code-execution)

[Hooks](https://docs.openclaw.ai/automation/hooks) [BTW side questions](https://docs.openclaw.ai/tools/btw)

Ctrl+I

---

## Browser troubleshooting - OpenClaw
**Source:** https://docs.openclaw.ai/tools/browser-linux-troubleshooting

[Skip to main content](https://docs.openclaw.ai/tools/browser-linux-troubleshooting#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nWeb browser\n\nBrowser troubleshooting\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Problem: “Failed to start Chrome CDP on port 18800”](https://docs.openclaw.ai/tools/browser-linux-troubleshooting#problem-%E2%80%9Cfailed-to-start-chrome-cdp-on-port-18800%E2%80%9D)\n- [Root cause](https://docs.openclaw.ai/tools/browser-linux-troubleshooting#root-cause)\n- [Solution 1: Install Google Chrome (Recommended)](https://docs.openclaw.ai/tools/browser-linux-troubleshooting#solution-1-install-google-chrome-recommended)\n- [Solution 2: Use Snap Chromium with Attach-Only Mode](https://docs.openclaw.ai/tools/browser-linux-troubleshooting#solution-2-use-snap-chromium-with-attach-only-mode)\n- [Verifying the Browser Works](https://docs.openclaw.ai/tools/browser-linux-troubleshooting#verifying-the-browser-works)\n- [Config reference](https://docs.openclaw.ai/tools/browser-linux-troubleshooting#config-reference)\n- [Problem: “No Chrome tabs found for profile=“user\"\"](https://docs.openclaw.ai/tools/browser-linux-troubleshooting#problem-%E2%80%9Cno-chrome-tabs-found-for-profile%3D%E2%80%9Cuser)\n- [Related](https://docs.openclaw.ai/tools/browser-linux-troubleshooting#related)\n\n## [​](https://docs.openclaw.ai/tools/browser-linux-troubleshooting\\#problem-%E2%80%9Cfailed-to-start-chrome-cdp-on-port-18800%E2%80%9D)  Problem: “Failed to start Chrome CDP on port 18800”\n\nOpenClaw’s browser control server fails to launch Chrome/Brave/Edge/Chromium with the error:\n\n```\n{\"error\":\"Error: Failed to start Chrome CDP on port 18800 for profile \\\"openclaw\\\".\"}\n```\n\n### [​](https://docs.openclaw.ai/tools/browser-linux-troubleshooting\\#root-cause)  Root cause\n\nOn Ubuntu (and many Linux distros), the default Chromium installation is a **snap package**. Snap’s AppArmor confinement interferes with how OpenClaw spawns and monitors the browser process.The `apt install chromium` command installs a stub package that redirects to snap:\n\n```\nNote, selecting \'chromium-browser\' instead of \'chromium\'\nchromium-browser is already the newest version (2:1snap1-0ubuntu2).\n```\n\nThis is NOT a real browser - it’s just a wrapper.Other common Linux launch failures:\n\n- `The profile appears to be in use by another Chromium process` means Chrome\nfound stale `Singleton*` lock files in the managed profile directory. OpenClaw\nremoves those locks and retries once when the lock points at a dead or\ndifferent-host process.\n- `Missing X server or $DISPLAY` means a visible browser was explicitly\nrequested on a host without a desktop session. By default, local managed\nprofiles now fall back to headless mode on Linux when `DISPLAY` and\n`WAYLAND_DISPLAY` are both unset. If you set `OPENCLAW_BROWSER_HEADLESS=0`,\n`browser.headless: false`, or `browser.profiles.<name>.headless: false`,\nremove that headed override, set `OPENCLAW_BROWSER_HEADLESS=1`, start `Xvfb`,\nrun `openclaw browser start --headless` for a one-shot managed launch, or run\nOpenClaw in a real desktop session.\n\n### [​](https://docs.openclaw.ai/tools/browser-linux-troubleshooting\\#solution-1-install-google-chrome-recommended)  Solution 1: Install Google Chrome (Recommended)\n\nInstall the official Google Chrome `.deb` package, which is not sandboxed by snap:\n\n```\nwget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb\nsudo dpkg -i google-chrome-stable_current_amd64.deb\nsudo apt --fix-broken install -y  # if there are dependency errors\n```\n\nThen update your OpenClaw config (`~/.openclaw/openclaw.json`):\n\n```\n{\n  \"browser\": {\n    \"enabled\": true,\n    \"executablePath\": \"/usr/bin/google-chrome-stable\",\n    \"headless\": true,\n    \"noSandbox\": true\n  }\n}\n```\n\n### [​](https://docs.openclaw.ai/tools/browser-linux-troubleshooting\\#solution-2-use-snap-chromium-with-attach-only-mode)  Solution 2: Use Snap Chromium with Attach-Only Mode\n\nIf you must use snap Chromium, configure OpenClaw to attach to a manually-started browser:\n\n1. Update config:\n\n```\n{\n  \"browser\": {\n    \"enabled\": true,\n    \"attachOnly\": true,\n    \"headless\": true,\n    \"noSandbox\": true\n  }\n}\n```\n\n2. Start Chromium manually:\n\n```\nchromium-browser --headless --no-sandbox --disable-gpu \\\n  --remote-debugging-port=18800 \\\n  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \\\n  about:blank &\n```\n\n3. Optionally create a systemd user service to auto-start Chrome:\n\n```\n# ~/.config/systemd/user/openclaw-browser.service\n[Unit]\nDescription=OpenClaw Browser (Chrome CDP)\nAfter=network.target\n\n[Service]\nExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank\nRestart=on-failure\nRestartSec=5\n\n[Install]\nWantedBy=default.target\n```\n\nEnable with: `systemctl --user enable --now openclaw-browser.service`\n\n### [​](https://docs.openclaw.ai/tools/browser-linux-troubleshooting\\#verifying-the-browser-works)  Verifying the Browser Works\n\nCheck status:\n\n```\ncurl -s http://127.0.0.1:18791/ | jq \'{running, pid, chosenBrowser}\'\n```\n\nTest browsing:\n\n```\ncurl -s -X POST http://127.0.0.1:18791/start\ncurl -s http://127.0.0.1:18791/tabs\n```\n\n### [​](https://docs.openclaw.ai/tools/browser-linux-troubleshooting\\#config-reference)  Config reference\n\n| Option | Description | Default |\n| --- | --- | --- |\n| `browser.enabled` | Enable browser control | `true` |\n| `browser.executablePath` | Path to a Chromium-based browser binary (Chrome/Brave/Edge/Chromium) | auto-detected (prefers default browser when Chromium-based) |\n| `browser.headless` | Run without GUI | `false` |\n| `OPENCLAW_BROWSER_HEADLESS` | Per-process override for local managed browser headless mode | unset |\n| `browser.noSandbox` | Add `--no-sandbox` flag (needed for some Linux setups) | `false` |\n| `browser.attachOnly` | Don’t launch browser, only attach to existing | `false` |\n| `browser.cdpPort` | Chrome DevTools Protocol port | `18800` |\n| `browser.localLaunchTimeoutMs` | Local managed Chrome discovery timeout | `15000` |\n| `browser.localCdpReadyTimeoutMs` | Local managed post-launch CDP readiness timeout | `8000` |\n\nOn Raspberry Pi, older VPS hosts, or slow storage, raise\n`browser.localLaunchTimeoutMs` when Chrome needs more time to expose its CDP HTTP\nendpoint. Raise `browser.localCdpReadyTimeoutMs` when launch succeeds but\n`openclaw browser start` still reports `not reachable after start`. Values must\nbe positive integers up to `120000` ms; invalid config values are rejected.\n\n### [​](https://docs.openclaw.ai/tools/browser-linux-troubleshooting\\#problem-%E2%80%9Cno-chrome-tabs-found-for-profile=%E2%80%9Cuser)  Problem: “No Chrome tabs found for profile=“user\"\"\n\nYou’re using an `existing-session` / Chrome MCP profile. OpenClaw can see local Chrome,\nbut there are no open tabs available to attach to.Fix options:\n\n1. **Use the managed browser:**`openclaw browser start --browser-profile openclaw`\n(or set `browser.defaultProfile: \"openclaw\"`).\n2. **Use Chrome MCP:** make sure local Chrome is running with at least one open tab, then retry with `--browser-profile user`.\n\nNotes:\n\n- `user` is host-only. For Linux servers, containers, or remote hosts, prefer CDP profiles.\n- `user` / other `existing-session` profiles keep the current Chrome MCP limits:\nref-driven actions, one-file upload hooks, no dialog timeout overrides, no\n`wait --load networkidle`, and no `responsebody`, PDF export, download\ninterception, or batch actions.\n- Local `openclaw` profiles auto-assign `cdpPort`/`cdpUrl`; only set those for remote CDP.\n- Remote CDP profiles accept `http://`, `https://`, `ws://`, and `wss://`.\nUse HTTP(S) for `/json/version` discovery, or WS(S) when your browser\nservice gives you a direct DevTools socket URL.\n\n## [​](https://docs.openclaw.ai/tools/browser-linux-troubleshooting\\#related)  Related\n\n- [Browser](https://docs.openclaw.ai/tools/browser)\n- [Browser login](https://docs.openclaw.ai/tools/browser-login)\n- [Browser WSL2 troubleshooting](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting)\n\n[Browser login](https://docs.openclaw.ai/tools/browser-login) [WSL2 + Windows + remote Chrome CDP troubleshooting](https://docs.openclaw.ai/tools/browser-wsl2-windows-remote-cdp-troubleshooting)\n\nCtrl+I

---

