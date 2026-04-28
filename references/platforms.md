# OpenClaw Platforms Documentation

## Voice wake (macOS) - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/voicewake

[Skip to main content](https://docs.openclaw.ai/platforms/mac/voicewake#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Features

Voice wake (macOS)

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Voice Wake & Push-to-Talk](https://docs.openclaw.ai/platforms/mac/voicewake#voice-wake-%26-push-to-talk)
- [Modes](https://docs.openclaw.ai/platforms/mac/voicewake#modes)
- [Runtime behavior (wake-word)](https://docs.openclaw.ai/platforms/mac/voicewake#runtime-behavior-wake-word)
- [Lifecycle invariants](https://docs.openclaw.ai/platforms/mac/voicewake#lifecycle-invariants)
- [Sticky overlay failure mode (previous)](https://docs.openclaw.ai/platforms/mac/voicewake#sticky-overlay-failure-mode-previous)
- [Push-to-talk specifics](https://docs.openclaw.ai/platforms/mac/voicewake#push-to-talk-specifics)
- [User-facing settings](https://docs.openclaw.ai/platforms/mac/voicewake#user-facing-settings)
- [Forwarding behavior](https://docs.openclaw.ai/platforms/mac/voicewake#forwarding-behavior)
- [Forwarding payload](https://docs.openclaw.ai/platforms/mac/voicewake#forwarding-payload)
- [Quick verification](https://docs.openclaw.ai/platforms/mac/voicewake#quick-verification)
- [Related](https://docs.openclaw.ai/platforms/mac/voicewake#related)

# [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#voice-wake-&-push-to-talk) Voice Wake & Push-to-Talk

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#modes) Modes

- **Wake-word mode** (default): always-on Speech recognizer waits for trigger tokens (`swabbleTriggerWords`). On match it starts capture, shows the overlay with partial text, and auto-sends after silence.
- **Push-to-talk (Right Option hold)**: hold the right Option key to capture immediately—no trigger needed. The overlay appears while held; releasing finalizes and forwards after a short delay so you can tweak text.

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#runtime-behavior-wake-word) Runtime behavior (wake-word)

- Speech recognizer lives in `VoiceWakeRuntime`.
- Trigger only fires when there’s a **meaningful pause** between the wake word and the next word (~0.55s gap). The overlay/chime can start on the pause even before the command begins.
- Silence windows: 2.0s when speech is flowing, 5.0s if only the trigger was heard.
- Hard stop: 120s to prevent runaway sessions.
- Debounce between sessions: 350ms.
- Overlay is driven via `VoiceWakeOverlayController` with committed/volatile coloring.
- After send, recognizer restarts cleanly to listen for the next trigger.

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#lifecycle-invariants) Lifecycle invariants

- If Voice Wake is enabled and permissions are granted, the wake-word recognizer should be listening (except during an explicit push-to-talk capture).
- Overlay visibility (including manual dismiss via the X button) must never prevent the recognizer from resuming.

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#sticky-overlay-failure-mode-previous) Sticky overlay failure mode (previous)

Previously, if the overlay got stuck visible and you manually closed it, Voice Wake could appear “dead” because the runtime’s restart attempt could be blocked by overlay visibility and no subsequent restart was scheduled.Hardening:

- Wake runtime restart is no longer blocked by overlay visibility.
- Overlay dismiss completion triggers a `VoiceWakeRuntime.refresh(...)` via `VoiceSessionCoordinator`, so manual X-dismiss always resumes listening.

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#push-to-talk-specifics) Push-to-talk specifics

- Hotkey detection uses a global `.flagsChanged` monitor for **right Option** (`keyCode 61` \\+ `.option`). We only observe events (no swallowing).
- Capture pipeline lives in `VoicePushToTalk`: starts Speech immediately, streams partials to the overlay, and calls `VoiceWakeForwarder` on release.
- When push-to-talk starts we pause the wake-word runtime to avoid dueling audio taps; it restarts automatically after release.
- Permissions: requires Microphone + Speech; seeing events needs Accessibility/Input Monitoring approval.
- External keyboards: some may not expose right Option as expected—offer a fallback shortcut if users report misses.

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#user-facing-settings) User-facing settings

- **Voice Wake** toggle: enables wake-word runtime.
- **Hold Cmd+Fn to talk**: enables the push-to-talk monitor. Disabled on macOS < 26.
- Language & mic pickers, live level meter, trigger-word table, tester (local-only; does not forward).
- Mic picker preserves the last selection if a device disconnects, shows a disconnected hint, and temporarily falls back to the system default until it returns.
- **Sounds**: chimes on trigger detect and on send; defaults to the macOS “Glass” system sound. You can pick any `NSSound`-loadable file (e.g. MP3/WAV/AIFF) for each event or choose **No Sound**.

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#forwarding-behavior) Forwarding behavior

- When Voice Wake is enabled, transcripts are forwarded to the active gateway/agent (the same local vs remote mode used by the rest of the mac app).
- Replies are delivered to the **last-used main provider** (WhatsApp/Telegram/Discord/WebChat). If delivery fails, the error is logged and the run is still visible via WebChat/session logs.

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#forwarding-payload) Forwarding payload

- `VoiceWakeForwarder.prefixedTranscript(_:)` prepends the machine hint before sending. Shared between wake-word and push-to-talk paths.

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#quick-verification) Quick verification

- Toggle push-to-talk on, hold Cmd+Fn, speak, release: overlay should show partials then send.
- While holding, menu-bar ears should stay enlarged (uses `triggerVoiceEars(ttl:nil)`); they drop after release.

## [​](https://docs.openclaw.ai/platforms/mac/voicewake\\#related) Related

- [Voice wake](https://docs.openclaw.ai/nodes/voicewake)
- [Voice overlay](https://docs.openclaw.ai/platforms/mac/voice-overlay)
- [macOS app](https://docs.openclaw.ai/platforms/macos)

[macOS IPC](https://docs.openclaw.ai/platforms/mac/xpc) [Voice overlay](https://docs.openclaw.ai/platforms/mac/voice-overlay)

Ctrl+I

---

## Skills (macOS) - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/skills

[Skip to main content](https://docs.openclaw.ai/platforms/mac/skills#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Features

Skills (macOS)

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Data source](https://docs.openclaw.ai/platforms/mac/skills#data-source)
- [Install actions](https://docs.openclaw.ai/platforms/mac/skills#install-actions)
- [Env/API keys](https://docs.openclaw.ai/platforms/mac/skills#env%2Fapi-keys)
- [Remote mode](https://docs.openclaw.ai/platforms/mac/skills#remote-mode)
- [Related](https://docs.openclaw.ai/platforms/mac/skills#related)

The macOS app surfaces OpenClaw skills via the gateway; it does not parse skills locally.

## [​](https://docs.openclaw.ai/platforms/mac/skills\\#data-source)  Data source

- `skills.status` (gateway) returns all skills plus eligibility and missing requirements
(including allowlist blocks for bundled skills).
- Requirements are derived from `metadata.openclaw.requires` in each `SKILL.md`.

## [​](https://docs.openclaw.ai/platforms/mac/skills\\#install-actions)  Install actions

- `metadata.openclaw.install` defines install options (brew/node/go/uv).
- The app calls `skills.install` to run installers on the gateway host.
- Built-in dangerous-code `critical` findings block `skills.install` by default; suspicious findings still warn only. The dangerous override exists on the gateway request, but the default app flow stays fail-closed.
- If every install option is `download`, the gateway surfaces all download
choices.
- Otherwise, the gateway picks one preferred installer using the current
install preferences and host binaries: Homebrew first when
`skills.install.preferBrew` is enabled and `brew` exists, then `uv`, then the
configured node manager from `skills.install.nodeManager`, then later
fallbacks like `go` or `download`.
- Node install labels reflect the configured node manager, including `yarn`.

## [​](https://docs.openclaw.ai/platforms/mac/skills\\#env/api-keys)  Env/API keys

- The app stores keys in `~/.openclaw/openclaw.json` under `skills.entries.<skillKey>`.
- `skills.update` patches `enabled`, `apiKey`, and `env`.

## [​](https://docs.openclaw.ai/platforms/mac/skills\\#remote-mode)  Remote mode

- Install + config updates happen on the gateway host (not the local Mac).

## [​](https://docs.openclaw.ai/platforms/mac/skills\\#related)  Related

- [Skills](https://docs.openclaw.ai/tools/skills)
- [macOS app](https://docs.openclaw.ai/platforms/macos)

[Canvas](https://docs.openclaw.ai/platforms/mac/canvas) [Peekaboo bridge](https://docs.openclaw.ai/platforms/mac/peekaboo)

Ctrl+I

---

## macOS permissions - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/permissions

[Skip to main content](https://docs.openclaw.ai/platforms/mac/permissions#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Setup

macOS permissions

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Requirements for stable permissions](https://docs.openclaw.ai/platforms/mac/permissions#requirements-for-stable-permissions)
- [Recovery checklist when prompts disappear](https://docs.openclaw.ai/platforms/mac/permissions#recovery-checklist-when-prompts-disappear)
- [Files and folders permissions (Desktop/Documents/Downloads)](https://docs.openclaw.ai/platforms/mac/permissions#files-and-folders-permissions-desktop%2Fdocuments%2Fdownloads)
- [Related](https://docs.openclaw.ai/platforms/mac/permissions#related)

macOS permission grants are fragile. TCC associates a permission grant with the
app’s code signature, bundle identifier, and on-disk path. If any of those change,
macOS treats the app as new and may drop or hide prompts.

## [​](https://docs.openclaw.ai/platforms/mac/permissions\\#requirements-for-stable-permissions)  Requirements for stable permissions

- Same path: run the app from a fixed location (for OpenClaw, `dist/OpenClaw.app`).
- Same bundle identifier: changing the bundle ID creates a new permission identity.
- Signed app: unsigned or ad-hoc signed builds do not persist permissions.
- Consistent signature: use a real Apple Development or Developer ID certificate
so the signature stays stable across rebuilds.

Ad-hoc signatures generate a new identity every build. macOS will forget previous
grants, and prompts can disappear entirely until the stale entries are cleared.

## [​](https://docs.openclaw.ai/platforms/mac/permissions\\#recovery-checklist-when-prompts-disappear)  Recovery checklist when prompts disappear

1. Quit the app.
2. Remove the app entry in System Settings -> Privacy & Security.
3. Relaunch the app from the same path and re-grant permissions.
4. If the prompt still does not appear, reset TCC entries with `tccutil` and try again.
5. Some permissions only reappear after a full macOS restart.

Example resets (replace bundle ID as needed):

```
sudo tccutil reset Accessibility ai.openclaw.mac
sudo tccutil reset ScreenCapture ai.openclaw.mac
sudo tccutil reset AppleEvents
```

## [​](https://docs.openclaw.ai/platforms/mac/permissions\\#files-and-folders-permissions-desktop/documents/downloads)  Files and folders permissions (Desktop/Documents/Downloads)

macOS may also gate Desktop, Documents, and Downloads for terminal/background processes. If file reads or directory listings hang, grant access to the same process context that performs file operations (for example Terminal/iTerm, LaunchAgent-launched app, or SSH process).Workaround: move files into the OpenClaw workspace (`~/.openclaw/workspace`) if you want to avoid per-folder grants.If you are testing permissions, always sign with a real certificate. Ad-hoc
builds are only acceptable for quick local runs where permissions do not matter.

## [​](https://docs.openclaw.ai/platforms/mac/permissions\\#related)  Related

- [macOS app](https://docs.openclaw.ai/platforms/macos)
- [macOS signing](https://docs.openclaw.ai/platforms/mac/signing)

[Menu bar icon](https://docs.openclaw.ai/platforms/mac/icon) [macOS signing](https://docs.openclaw.ai/platforms/mac/signing)

Ctrl+I

---

## macOS dev setup - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/dev-setup

[Skip to main content](https://docs.openclaw.ai/platforms/mac/dev-setup#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Setup

macOS dev setup

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [macOS developer setup](https://docs.openclaw.ai/platforms/mac/dev-setup#macos-developer-setup)
- [Prerequisites](https://docs.openclaw.ai/platforms/mac/dev-setup#prerequisites)
- [1\. Install Dependencies](https://docs.openclaw.ai/platforms/mac/dev-setup#1-install-dependencies)
- [2\. Build and Package the App](https://docs.openclaw.ai/platforms/mac/dev-setup#2-build-and-package-the-app)
- [3\. Install the CLI](https://docs.openclaw.ai/platforms/mac/dev-setup#3-install-the-cli)
- [Troubleshooting](https://docs.openclaw.ai/platforms/mac/dev-setup#troubleshooting)
- [Build fails: toolchain or SDK mismatch](https://docs.openclaw.ai/platforms/mac/dev-setup#build-fails-toolchain-or-sdk-mismatch)
- [App crashes on permission grant](https://docs.openclaw.ai/platforms/mac/dev-setup#app-crashes-on-permission-grant)
- [Gateway “Starting…” indefinitely](https://docs.openclaw.ai/platforms/mac/dev-setup#gateway-%E2%80%9Cstarting%E2%80%A6%E2%80%9D-indefinitely)
- [Related](https://docs.openclaw.ai/platforms/mac/dev-setup#related)

# [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#macos-developer-setup)  macOS developer setup

Build and run the OpenClaw macOS application from source.

## [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#prerequisites)  Prerequisites

Before building the app, ensure you have the following installed:

1. **Xcode 26.2+**: Required for Swift development.
2. **Node.js 24 & pnpm**: Recommended for the gateway, CLI, and packaging scripts. Node 22 LTS, currently `22.14+`, remains supported for compatibility.

## [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#1-install-dependencies)  1\. Install Dependencies

Install the project-wide dependencies:

```
pnpm install
```

## [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#2-build-and-package-the-app)  2\. Build and Package the App

To build the macOS app and package it into `dist/OpenClaw.app`, run:

```
./scripts/package-mac-app.sh
```

If you don’t have an Apple Developer ID certificate, the script will automatically use **ad-hoc signing** (`-`).For dev run modes, signing flags, and Team ID troubleshooting, see the macOS app README:
[https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md](https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md)

> **Note**: Ad-hoc signed apps may trigger security prompts. If the app crashes immediately with “Abort trap 6”, see the [Troubleshooting](https://docs.openclaw.ai/platforms/mac/dev-setup#troubleshooting) section.

## [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#3-install-the-cli)  3\. Install the CLI

The macOS app expects a global `openclaw` CLI install to manage background tasks.**To install it (recommended):**

1. Open the OpenClaw app.
2. Go to the **General** settings tab.
3. Click **“Install CLI”**.

Alternatively, install it manually:

```
npm install -g openclaw@<version>
```

`pnpm add -g openclaw@<version>` and `bun add -g openclaw@<version>` also work.
For the Gateway runtime, Node remains the recommended path.

## [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#troubleshooting)  Troubleshooting

### [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#build-fails-toolchain-or-sdk-mismatch)  Build fails: toolchain or SDK mismatch

The macOS app build expects the latest macOS SDK and Swift 6.2 toolchain.**System dependencies (required):**

- **Latest macOS version available in Software Update** (required by Xcode 26.2 SDKs)
- **Xcode 26.2** (Swift 6.2 toolchain)

**Checks:**

```
xcodebuild -version
xcrun swift --version
```

If versions don’t match, update macOS/Xcode and re-run the build.

### [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#app-crashes-on-permission-grant)  App crashes on permission grant

If the app crashes when you try to allow **Speech Recognition** or **Microphone** access, it may be due to a corrupted TCC cache or signature mismatch.**Fix:**

1. Reset the TCC permissions:














```
tccutil reset All ai.openclaw.mac.debug
```

2. If that fails, change the `BUNDLE_ID` temporarily in [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) to force a “clean slate” from macOS.

### [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#gateway-%E2%80%9Cstarting%E2%80%A6%E2%80%9D-indefinitely)  Gateway “Starting…” indefinitely

If the gateway status stays on “Starting…”, check if a zombie process is holding the port:

```
openclaw gateway status
openclaw gateway stop

# If you\'re not using a LaunchAgent (dev mode / manual runs), find the listener:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

If a manual run is holding the port, stop that process (Ctrl+C). As a last resort, kill the PID you found above.

## [​](https://docs.openclaw.ai/platforms/mac/dev-setup\#related)  Related

- [macOS app](https://docs.openclaw.ai/platforms/macos)
- [Install overview](https://docs.openclaw.ai/install)

[iOS app](https://docs.openclaw.ai/platforms/ios) [Menu bar](https://docs.openclaw.ai/platforms/mac/menu-bar)

Ctrl+I

---

## macOS signing - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/signing

[Skip to main content](https://docs.openclaw.ai/platforms/mac/signing#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Setup

macOS signing

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [mac signing (debug builds)](https://docs.openclaw.ai/platforms/mac/signing#mac-signing-debug-builds)
- [Usage](https://docs.openclaw.ai/platforms/mac/signing#usage)
- [Ad-hoc Signing Note](https://docs.openclaw.ai/platforms/mac/signing#ad-hoc-signing-note)
- [Build metadata for About](https://docs.openclaw.ai/platforms/mac/signing#build-metadata-for-about)
- [Why](https://docs.openclaw.ai/platforms/mac/signing#why)
- [Related](https://docs.openclaw.ai/platforms/mac/signing#related)

# [​](https://docs.openclaw.ai/platforms/mac/signing\\#mac-signing-debug-builds)  mac signing (debug builds)

This app is usually built from [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), which now:

- sets a stable debug bundle identifier: `ai.openclaw.mac.debug`
- writes the Info.plist with that bundle id (override via `BUNDLE_ID=...`)
- calls [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) to sign the main binary and app bundle so macOS treats each rebuild as the same signed bundle and keeps TCC permissions (notifications, accessibility, screen recording, mic, speech). For stable permissions, use a real signing identity; ad-hoc is opt-in and fragile (see [macOS permissions](https://docs.openclaw.ai/platforms/mac/permissions)).
- uses `CODESIGN_TIMESTAMP=auto` by default; it enables trusted timestamps for Developer ID signatures. Set `CODESIGN_TIMESTAMP=off` to skip timestamping (offline debug builds).
- inject build metadata into Info.plist: `OpenClawBuildTimestamp` (UTC) and `OpenClawGitCommit` (short hash) so the About pane can show build, git, and debug/release channel.
- **Packaging defaults to Node 24**: the script runs TS builds and the Control UI build. Node 22 LTS, currently `22.14+`, remains supported for compatibility.
- reads `SIGN_IDENTITY` from the environment. Add `export SIGN_IDENTITY=\"Apple Development: Your Name (TEAMID)\"` (or your Developer ID Application cert) to your shell rc to always sign with your cert. Ad-hoc signing requires explicit opt-in via `ALLOW_ADHOC_SIGNING=1` or `SIGN_IDENTITY=\"-\"` (not recommended for permission testing).
- runs a Team ID audit after signing and fails if any Mach-O inside the app bundle is signed by a different Team ID. Set `SKIP_TEAM_ID_CHECK=1` to bypass.

## [​](https://docs.openclaw.ai/platforms/mac/signing\\#usage)  Usage

```
# from repo root
scripts/package-mac-app.sh               # auto-selects identity; errors if none found
SIGN_IDENTITY=\"Developer ID Application: Your Name\" scripts/package-mac-app.sh   # real cert
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (permissions will not stick)
SIGN_IDENTITY=\"-\" scripts/package-mac-app.sh        # explicit ad-hoc (same caveat)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # dev-only Sparkle Team ID mismatch workaround
```

### [​](https://docs.openclaw.ai/platforms/mac/signing\\#ad-hoc-signing-note)  Ad-hoc Signing Note

When signing with `SIGN_IDENTITY=\"-\"` (ad-hoc), the script automatically disables the **Hardened Runtime** (`--options runtime`). This is necessary to prevent crashes when the app attempts to load embedded frameworks (like Sparkle) that do not share the same Team ID. Ad-hoc signatures also break TCC permission persistence; see [macOS permissions](https://docs.openclaw.ai/platforms/mac/permissions) for recovery steps.

## [​](https://docs.openclaw.ai/platforms/mac/signing\\#build-metadata-for-about)  Build metadata for About

`package-mac-app.sh` stamps the bundle with:

- `OpenClawBuildTimestamp`: ISO8601 UTC at package time
- `OpenClawGitCommit`: short git hash (or `unknown` if unavailable)

The About tab reads these keys to show version, build date, git commit, and whether it’s a debug build (via `#if DEBUG`). Run the packager to refresh these values after code changes.

## [​](https://docs.openclaw.ai/platforms/mac/signing\\#why)  Why

TCC permissions are tied to the bundle identifier _and_ code signature. Unsigned debug builds with changing UUIDs were causing macOS to forget grants after each rebuild. Signing the binaries (ad‑hoc by default) and keeping a fixed bundle id/path (`dist/OpenClaw.app`) preserves the grants between builds, matching the VibeTunnel approach.

## [​](https://docs.openclaw.ai/platforms/mac/signing\\#related)  Related

- [macOS app](https://docs.openclaw.ai/platforms/macos)
- [macOS permissions](https://docs.openclaw.ai/platforms/mac/permissions)

[macOS permissions](https://docs.openclaw.ai/platforms/mac/permissions) [Gateway on macOS](https://docs.openclaw.ai/platforms/mac/bundled-gateway)

Ctrl+I

---

## Android app - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/android

[Skip to main content](https://docs.openclaw.ai/platforms/android#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Platforms overview

Android app

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Support snapshot](https://docs.openclaw.ai/platforms/android#support-snapshot)
- [System control](https://docs.openclaw.ai/platforms/android#system-control)
- [Connection runbook](https://docs.openclaw.ai/platforms/android#connection-runbook)
- [Prerequisites](https://docs.openclaw.ai/platforms/android#prerequisites)
- [1) Start the Gateway](https://docs.openclaw.ai/platforms/android#1-start-the-gateway)
- [2) Verify discovery (optional)](https://docs.openclaw.ai/platforms/android#2-verify-discovery-optional)
- [Tailnet (Vienna ⇄ London) discovery via unicast DNS-SD](https://docs.openclaw.ai/platforms/android#tailnet-vienna-%E2%87%84-london-discovery-via-unicast-dns-sd)
- [3) Connect from Android](https://docs.openclaw.ai/platforms/android#3-connect-from-android)
- [4) Approve pairing (CLI)](https://docs.openclaw.ai/platforms/android#4-approve-pairing-cli)
- [5) Verify the node is connected](https://docs.openclaw.ai/platforms/android#5-verify-the-node-is-connected)
- [6) Chat + history](https://docs.openclaw.ai/platforms/android#6-chat-%2B-history)
- [7) Canvas + camera](https://docs.openclaw.ai/platforms/android#7-canvas-%2B-camera)
- [Gateway Canvas Host (recommended for web content)](https://docs.openclaw.ai/platforms/android#gateway-canvas-host-recommended-for-web-content)
- [8) Voice + expanded Android command surface](https://docs.openclaw.ai/platforms/android#8-voice-%2B-expanded-android-command-surface)
- [Assistant entrypoints](https://docs.openclaw.ai/platforms/android#assistant-entrypoints)
- [Notification forwarding](https://docs.openclaw.ai/platforms/android#notification-forwarding)
- [Related](https://docs.openclaw.ai/platforms/android#related)

The Android app has not been publicly released yet. The source code is available in the [OpenClaw repository](https://github.com/openclaw/openclaw) under `apps/android`. You can build it yourself using Java 17 and the Android SDK (`./gradlew :app:assemblePlayDebug`). See [apps/android/README.md](https://github.com/openclaw/openclaw/blob/main/apps/android/README.md) for build instructions.

## [​](https://docs.openclaw.ai/platforms/android\\#support-snapshot)  Support snapshot

- Role: companion node app (Android does not host the Gateway).
- Gateway required: yes (run it on macOS, Linux, or Windows via WSL2).
- Install: [Getting Started](https://docs.openclaw.ai/start/getting-started) \\+ [Pairing](https://docs.openclaw.ai/channels/pairing).
- Gateway: [Runbook](https://docs.openclaw.ai/gateway) \\+ [Configuration](https://docs.openclaw.ai/gateway/configuration).

  - Protocols: [Gateway protocol](https://docs.openclaw.ai/gateway/protocol) (nodes + control plane).

## [​](https://docs.openclaw.ai/platforms/android\\#system-control)  System control

System control (launchd/systemd) lives on the Gateway host. See [Gateway](https://docs.openclaw.ai/gateway).

## [​](https://docs.openclaw.ai/platforms/android\\#connection-runbook)  Connection runbook

Android node app ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway**Android connects directly to the Gateway WebSocket and uses device pairing (`role: node`).For Tailscale or public hosts, Android requires a secure endpoint:

- Preferred: Tailscale Serve / Funnel with `https://<magicdns>` / `wss://<magicdns>`
- Also supported: any other `wss://` Gateway URL with a real TLS endpoint
- Cleartext `ws://` remains supported on private LAN addresses / `.local` hosts, plus `localhost`, `127.0.0.1`, and the Android emulator bridge (`10.0.2.2`)

### [​](https://docs.openclaw.ai/platforms/android\\#prerequisites)  Prerequisites

- You can run the Gateway on the “master” machine.
- Android device/emulator can reach the gateway WebSocket:
  - Same LAN with mDNS/NSD, **or**
  - Same Tailscale tailnet using Wide-Area Bonjour / unicast DNS-SD (see below), **or**
  - Manual gateway host/port (fallback)
- Tailnet/public mobile pairing does **not** use raw tailnet IP `ws://` endpoints. Use Tailscale Serve or another `wss://` URL instead.
- You can run the CLI (`openclaw`) on the gateway machine (or via SSH).

### [​](https://docs.openclaw.ai/platforms/android\\#1-start-the-gateway)  1) Start the Gateway

```
openclaw gateway --port 18789 --verbose
```

Confirm in logs you see something like:

- `listening on ws://0.0.0.0:18789`

For remote Android access over Tailscale, prefer Serve/Funnel instead of a raw tailnet bind:

```
openclaw gateway --tailscale serve
```

This gives Android a secure `wss://` / `https://` endpoint. A plain `gateway.bind: \"tailnet\"` setup is not enough for first-time remote Android pairing unless you also terminate TLS separately.

### [​](https://docs.openclaw.ai/platforms/android\\#2-verify-discovery-optional)  2) Verify discovery (optional)

From the gateway machine:

```
dns-sd -B _openclaw-gw._tcp local.
```

More debugging notes: [Bonjour](https://docs.openclaw.ai/gateway/bonjour).If you also configured a wide-area discovery domain, compare against:

```
openclaw gateway discover --json
```

That shows `local.` plus the configured wide-area domain in one pass and uses the resolved\nservice endpoint instead of TXT-only hints.

#### [​](https://docs.openclaw.ai/platforms/android\\#tailnet-vienna-%E2%87%84-london-discovery-via-unicast-dns-sd)  Tailnet (Vienna ⇄ London) discovery via unicast DNS-SD

Android NSD/mDNS discovery won’t cross networks. If your Android node and the gateway are on different networks but connected via Tailscale, use Wide-Area Bonjour / unicast DNS-SD instead.Discovery alone is not sufficient for tailnet/public Android pairing. The discovered route still needs a secure endpoint (`wss://` or Tailscale Serve):

1. Set up a DNS-SD zone (example `openclaw.internal.`) on the gateway host and publish `_openclaw-gw._tcp` records.
2. Configure Tailscale split DNS for your chosen domain pointing at that DNS server.

Details and example CoreDNS config: [Bonjour](https://docs.openclaw.ai/gateway/bonjour).

### [​](https://docs.openclaw.ai/platforms/android\\#3-connect-from-android)  3) Connect from Android

In the Android app:

- The app keeps its gateway connection alive via a **foreground service** (persistent notification).
- Open the **Connect** tab.
- Use **Setup Code** or **Manual** mode.
- If discovery is blocked, use manual host/port in **Advanced controls**. For private LAN hosts, `ws://` still works. For Tailscale/public hosts, turn on TLS and use a `wss://` / Tailscale Serve endpoint.

After the first successful pairing, Android auto-reconnects on launch:

- Manual endpoint (if enabled), otherwise
- The last discovered gateway (best-effort).

### [​](https://docs.openclaw.ai/platforms/android\\#4-approve-pairing-cli)  4) Approve pairing (CLI)

On the gateway machine:

```
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

Pairing details: [Pairing](https://docs.openclaw.ai/channels/pairing).Optional: if the Android node always connects from a tightly controlled subnet,\nyou can opt in to first-time node auto-approval with explicit CIDRs or exact IPs:\n
```
{\n  gateway: {\n    nodes: {\n      pairing: {\n        autoApproveCidrs: [\"192.168.1.0/24\"],\n      },\n    },\n  },\n}\n```

This is disabled by default. It applies only to fresh `role: node` pairing with\nno requested scopes. Operator/browser pairing and any role, scope, metadata, or\npublic-key change still require manual approval.\n
### [​](https://docs.openclaw.ai/platforms/android\\#5-verify-the-node-is-connected)  5) Verify the node is connected

- Via nodes status:\n
\n\n\n\n\n\n\n\n\n\n\n\n\n```
openclaw nodes status
```

- Via Gateway:\n
\n\n\n\n\n\n\n\n\n\n\n\n\n\n```
openclaw gateway call node.list --params \"{}\"\n```

\n### [​](https://docs.openclaw.ai/platforms/android\\#6-chat-+-history)  6) Chat + history

The Android Chat tab supports session selection (default `main`, plus other existing sessions):\n
- History: `chat.history` (display-normalized; inline directive tags are\nstripped from visible text, plain-text tool-call XML payloads (including\n`<tool_call>...</tool_call>`, `<function_call>...</function_call>`,\n`<tool_calls>...</tool_calls>`, `<function_calls>...</function_calls>`, and\ntruncated tool-call blocks) and leaked ASCII/full-width model control tokens\nare stripped, pure silent-token assistant rows such as exact `NO_REPLY` /\n`no_reply` are omitted, and oversized rows can be replaced with placeholders)\n- Send: `chat.send`\n- Push updates (best-effort): `chat.subscribe` → `event:\"chat\"`\n
### [​](https://docs.openclaw.ai/platforms/android\\#7-canvas-+-camera)  7) Canvas + camera

#### [​](https://docs.openclaw.ai/platforms/android\\#gateway-canvas-host-recommended-for-web-content)  Gateway Canvas Host (recommended for web content)

If you want the node to show real HTML/CSS/JS that the agent can edit on disk, point the node at the Gateway canvas host.\n
Nodes load canvas from the Gateway HTTP server (same port as `gateway.port`, default `18789`).\n
1. Create `~/.openclaw/workspace/canvas/index.html` on the gateway host.\n2. Navigate the node to it (LAN):\n
```
openclaw nodes invoke --node \"<Android Node>\" --command canvas.navigate --params \'{\"url\":\"http://<gateway-hostname>.local:18789/__openclaw__/canvas/\"}\'\n```

Tailnet (optional): if both devices are on Tailscale, use a MagicDNS name or tailnet IP instead of `.local`, e.g. `http://<gateway-magicdns>:18789/__openclaw__/canvas/`.This server injects a live-reload client into HTML and reloads on file changes.\nThe A2UI host lives at `http://<gateway-host>:18789/__openclaw__/a2ui/`.Canvas commands (foreground only):\n
- `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (use `{\"url\":\"\"}` or `{\"url\":\"/\"}` to return to the default scaffold). `canvas.snapshot` returns `{ format, base64 }` (default `format=\"jpeg\"`).\n- A2UI: `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` legacy alias)\n
Camera commands (foreground only; permission-gated):\n
- `camera.snap` (jpg)\n- `camera.clip` (mp4)\n
See [Camera node](https://docs.openclaw.ai/nodes/camera) for parameters and CLI helpers.\n
### [​](https://docs.openclaw.ai/platforms/android\\#8-voice-+-expanded-android-command-surface)  8) Voice + expanded Android command surface\n
- Voice tab: Android has two explicit capture modes. **Mic** is a manual Voice-tab session that sends each pause as a chat turn and stops when the app leaves the foreground or the user leaves the Voice tab. **Talk** is continuous Talk Mode and keeps listening until toggled off or the node disconnects.\n- Talk Mode promotes the existing foreground service from `dataSync` to `dataSync|microphone` before capture starts, then demotes it when Talk Mode stops. Android 14+ requires the `FOREGROUND_SERVICE_MICROPHONE` declaration, the `RECORD_AUDIO` runtime grant, and the microphone service type at runtime.\n- Spoken replies use `talk.speak` through the configured gateway Talk provider. Local system TTS is used only when `talk.speak` is unavailable.\n- Voice wake remains disabled in the Android UX/runtime.\n- Additional Android command families (availability depends on device + permissions):\n  - `device.status`, `device.info`, `device.permissions`, `device.health`\n  - `notifications.list`, `notifications.actions` (see [Notification forwarding](https://docs.openclaw.ai/platforms/android#notification-forwarding) below)\n  - `photos.latest`\n  - `contacts.search`, `contacts.add`\n  - `calendar.events`, `calendar.add`\n  - `callLog.search`\n  - `sms.search`\n  - `motion.activity`, `motion.pedometer`\n
## [​](https://docs.openclaw.ai/platforms/android\\#assistant-entrypoints)  Assistant entrypoints

Android supports launching OpenClaw from the system assistant trigger (Google\nAssistant). When configured, holding the home button or saying “Hey Google, ask\nOpenClaw…” opens the app and hands the prompt into the chat composer.This uses Android **App Actions** metadata declared in the app manifest. No\nextra configuration is needed on the gateway side — the assistant intent is\nhandled entirely by the Android app and forwarded as a normal chat message.\n
App Actions availability depends on the device, Google Play Services version,\nand whether the user has set OpenClaw as the default assistant app.\n
## [​](https://openclaw.ai/platforms/android\\#notification-forwarding)  Notification forwarding

Android can forward device notifications to the gateway as events. Several controls let you scope which notifications are forwarded and when.\n
| Key | Type | Description |\n| --- | --- | --- |\n| `notifications.allowPackages` | string\\[\\] | Only forward notifications from these package names. If set, all other packages are ignored. |\n| `notifications.denyPackages` | string\\[\\] | Never forward notifications from these package names. Applied after `allowPackages`. |\n| `notifications.quietHours.start` | string (HH:mm) | Start of quiet hours window (local device time). Notifications are suppressed during this window. |\n| `notifications.quietHours.end` | string (HH:mm) | End of quiet hours window. |\n| `notifications.rateLimit` | number | Maximum forwarded notifications per package per minute. Excess notifications are dropped. |\n
The notification picker also uses safer behavior for forwarded notification events, preventing accidental forwarding of sensitive system notifications.Example configuration:\n
```\n{\n  notifications: {\n    allowPackages: [\"com.slack\", \"com.whatsapp\"],\n    denyPackages: [\"com.android.systemui\"],\n    quietHours: {\n      start: \"22:00\",\n      end: \"07:00\",\n    },\n    rateLimit: 5,\n  },\n}\n```\n
Notification forwarding requires the Android Notification Listener permission. The app prompts for this during setup.\n
## [​](https://docs.openclaw.ai/platforms/android\\#related)  Related\n
- [iOS app](https://docs.openclaw.ai/platforms/ios)\n- [Nodes](https://docs.openclaw.ai/nodes)\n- [Android node troubleshooting](https://docs.openclaw.ai/nodes/troubleshooting)\n
[Windows](https://docs.openclaw.ai/platforms/windows) [iOS app](https://docs.openclaw.ai/platforms/ios)\n
Ctrl+I

---

## Peekaboo bridge - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/peekaboo

[Skip to main content](https://docs.openclaw.ai/platforms/mac/peekaboo#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Features

Peekaboo bridge

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [What this is (and is not)](https://docs.openclaw.ai/platforms/mac/peekaboo#what-this-is-and-is-not)
- [Enable the bridge](https://docs.openclaw.ai/platforms/mac/peekaboo#enable-the-bridge)
- [Client discovery order](https://docs.openclaw.ai/platforms/mac/peekaboo#client-discovery-order)
- [Security & permissions](https://docs.openclaw.ai/platforms/mac/peekaboo#security-%26-permissions)
- [Snapshot behavior (automation)](https://docs.openclaw.ai/platforms/mac/peekaboo#snapshot-behavior-automation)
- [Troubleshooting](https://docs.openclaw.ai/platforms/mac/peekaboo#troubleshooting)
- [Related](https://docs.openclaw.ai/platforms/mac/peekaboo#related)

OpenClaw can host **PeekabooBridge** as a local, permission‑aware UI automation
broker. This lets the `peekaboo` CLI drive UI automation while reusing the
macOS app’s TCC permissions.

## [​](https://docs.openclaw.ai/platforms/mac/peekaboo\\#what-this-is-and-is-not)  What this is (and is not)

- **Host**: OpenClaw.app can act as a PeekabooBridge host.
- **Client**: use the `peekaboo` CLI (no separate `openclaw ui ...` surface).
- **UI**: visual overlays stay in Peekaboo.app; OpenClaw is a thin broker host.

## [​](https://docs.openclaw.ai/platforms/mac/peekaboo\\#enable-the-bridge)  Enable the bridge

In the macOS app:

- Settings → **Enable Peekaboo Bridge**

When enabled, OpenClaw starts a local UNIX socket server. If disabled, the host
is stopped and `peekaboo` will fall back to other available hosts.

## [​](https://docs.openclaw.ai/platforms/mac/peekaboo\\#client-discovery-order)  Client discovery order

Peekaboo clients typically try hosts in this order:

1. Peekaboo.app (full UX)
2. Claude.app (if installed)
3. OpenClaw.app (thin broker)

Use `peekaboo bridge status --verbose` to see which host is active and which
socket path is in use. You can override with:

```
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```

## [​](https://docs.openclaw.ai/platforms/mac/peekaboo\\#security-&-permissions)  Security & permissions

- The bridge validates **caller code signatures**; an allowlist of TeamIDs is
enforced (Peekaboo host TeamID + OpenClaw app TeamID).
- Requests time out after ~10 seconds.
- If required permissions are missing, the bridge returns a clear error message
rather than launching System Settings.

## [​](https://docs.openclaw.ai/platforms/mac/peekaboo\\#snapshot-behavior-automation)  Snapshot behavior (automation)

Snapshots are stored in memory and expire automatically after a short window.
If you need longer retention, re‑capture from the client.

## [​](https://docs.openclaw.ai/platforms/mac/peekaboo\\#troubleshooting)  Troubleshooting

- If `peekaboo` reports “bridge client is not authorized”, ensure the client is
properly signed or run the host with `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`
in **debug** mode only.
- If no hosts are found, open one of the host apps (Peekaboo.app or OpenClaw.app)
and confirm permissions are granted.

## [​](https://docs.openclaw.ai/platforms/mac/peekaboo\\#related)  Related

- [macOS app](https://docs.openclaw.ai/platforms/macos)
- [macOS permissions](https://docs.openclaw.ai/platforms/mac/permissions)

[Skills (macOS)](https://docs.openclaw.ai/platforms/mac/skills)

Ctrl+I

---

## Canvas - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/canvas

[Skip to main content](https://docs.openclaw.ai/platforms/mac/canvas#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Features

Canvas

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Where Canvas lives](https://docs.openclaw.ai/platforms/mac/canvas#where-canvas-lives)
- [Panel behavior](https://docs.openclaw.ai/platforms/mac/canvas#panel-behavior)
- [Agent API surface](https://docs.openclaw.ai/platforms/mac/canvas#agent-api-surface)
- [A2UI in Canvas](https://docs.openclaw.ai/platforms/mac/canvas#a2ui-in-canvas)
- [A2UI commands (v0.8)](https://docs.openclaw.ai/platforms/mac/canvas#a2ui-commands-v0-8)
- [Triggering agent runs from Canvas](https://docs.openclaw.ai/platforms/mac/canvas#triggering-agent-runs-from-canvas)
- [Security notes](https://docs.openclaw.ai/platforms/mac/canvas#security-notes)
- [Related](https://docs.openclaw.ai/platforms/mac/canvas#related)

The macOS app embeds an agent‑controlled **Canvas panel** using `WKWebView`. It
is a lightweight visual workspace for HTML/CSS/JS, A2UI, and small interactive
UI surfaces.

## [​](https://docs.openclaw.ai/platforms/mac/canvas\\#where-canvas-lives)  Where Canvas lives

Canvas state is stored under Application Support:

- `~/Library/Application Support/OpenClaw/canvas/<session>/...`

The Canvas panel serves those files via a **custom URL scheme**:

- `openclaw-canvas://<session>/<path>`

Examples:

- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`
- `openclaw-canvas://main/widgets/todo/` → `<canvasRoot>/main/widgets/todo/index.html`

If no `index.html` exists at the root, the app shows a **built‑in scaffold page**.

## [​](https://docs.openclaw.ai/platforms/mac/canvas\\#panel-behavior)  Panel behavior

- Borderless, resizable panel anchored near the menu bar (or mouse cursor).
- Remembers size/position per session.
- Auto‑reloads when local canvas files change.
- Only one Canvas panel is visible at a time (session is switched as needed).

Canvas can be disabled from Settings → **Allow Canvas**. When disabled, canvas
node commands return `CANVAS_DISABLED`.

## [​](https://docs.openclaw.ai/platforms/mac/canvas\\#agent-api-surface)  Agent API surface

Canvas is exposed via the **Gateway WebSocket**, so the agent can:

- show/hide the panel
- navigate to a path or URL
- evaluate JavaScript
- capture a snapshot image

CLI examples:

```
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url \"/\"
openclaw nodes canvas eval --node <id> --js \"document.title\"
openclaw nodes canvas snapshot --node <id>
```

Notes:

- `canvas.navigate` accepts **local canvas paths**, `http(s)` URLs, and `file://` URLs.
- If you pass `\"/\"`, the Canvas shows the local scaffold or `index.html`.

## [​](https://docs.openclaw.ai/platforms/mac/canvas\\#a2ui-in-canvas)  A2UI in Canvas

A2UI is hosted by the Gateway canvas host and rendered inside the Canvas panel.
When the Gateway advertises a Canvas host, the macOS app auto‑navigates to the
A2UI host page on first open.Default A2UI host URL:

```
http://<gateway-host>:18789/__openclaw__/a2ui/
```

### [​](https://docs.openclaw.ai/platforms/mac/canvas\\#a2ui-commands-v0-8)  A2UI commands (v0.8)

Canvas currently accepts **A2UI v0.8** server→client messages:

- `beginRendering`
- `surfaceUpdate`
- `dataModelUpdate`
- `deleteSurface`

`createSurface` (v0.9) is not supported.CLI example:

```
cat > /tmp/a2ui-v0.8.jsonl <<\'EOFA2\'
{\"surfaceUpdate\":{\"surfaceId\":\"main\",\"components\":[{\"id\":\"root\",\"component\":{\"Column\":{\"children\":{\"explicitList\":[\"title\",\"content\"]}}}},{\"id\":\"title\",\"component\":{\"Text\":{\"text\":{\"literalString\":\"Canvas (A2UI v0.8)\"},\"usageHint\":\"h1\"}}},{\"id\":\"content\",\"component\":{\"Text\":{\"text\":{\"literalString\":\"If you can read this, A2UI push works.\"},\"usageHint\":\"body\"}}}]}}
{\"beginRendering\":{\"surfaceId\":\"main\",\"root\":\"root\"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Quick smoke:

```
openclaw nodes canvas a2ui push --node <id> --text \"Hello from A2UI\"
```

## [​](https://docs.openclaw.ai/platforms/mac/canvas\\#triggering-agent-runs-from-canvas)  Triggering agent runs from Canvas

Canvas can trigger new agent runs via deep links:

- `openclaw://agent?...`

Example (in JS):

```
window.location.href = \"openclaw://agent?message=Review%20this%20design\";
```

The app prompts for confirmation unless a valid key is provided.

## [​](https://docs.openclaw.ai/platforms/mac/canvas\\#security-notes)  Security notes

- Canvas scheme blocks directory traversal; files must live under the session root.
- Local Canvas content uses a custom scheme (no loopback server required).
- External `http(s)` URLs are allowed only when explicitly navigated.

## [​](https://docs.openclaw.ai/platforms/mac/canvas\\#related)  Related

- [macOS app](https://docs.openclaw.ai/platforms/macos)
- [WebChat](https://docs.openclaw.ai/web/webchat)

[WebChat (macOS)](https://docs.openclaw.ai/platforms/mac/webchat) [Skills (macOS)](https://docs.openclaw.ai/platforms/mac/skills)

Ctrl+I

---

## Gateway lifecycle - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/child-process

[Skip to main content](https://docs.openclaw.ai/platforms/mac/child-process#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Runtime

Gateway lifecycle

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Gateway lifecycle on macOS](https://docs.openclaw.ai/platforms/mac/child-process#gateway-lifecycle-on-macos)
- [Default behavior (launchd)](https://docs.openclaw.ai/platforms/mac/child-process#default-behavior-launchd)
- [Unsigned dev builds](https://docs.openclaw.ai/platforms/mac/child-process#unsigned-dev-builds)
- [Attach-only mode](https://docs.openclaw.ai/platforms/mac/child-process#attach-only-mode)
- [Remote mode](https://docs.openclaw.ai/platforms/mac/child-process#remote-mode)
- [Why we prefer launchd](https://docs.openclaw.ai/platforms/mac/child-process#why-we-prefer-launchd)
- [Related](https://docs.openclaw.ai/platforms/mac/child-process#related)

# [​](https://docs.openclaw.ai/platforms/mac/child-process\\#gateway-lifecycle-on-macos)  Gateway lifecycle on macOS

The macOS app **manages the Gateway via launchd** by default and does not spawn
the Gateway as a child process. It first tries to attach to an already‑running
Gateway on the configured port; if none is reachable, it enables the launchd
service via the external `openclaw` CLI (no embedded runtime). This gives you
reliable auto‑start at login and restart on crashes.Child‑process mode (Gateway spawned directly by the app) is **not in use** today.
If you need tighter coupling to the UI, run the Gateway manually in a terminal.

## [​](https://docs.openclaw.ai/platforms/mac/child-process\\#default-behavior-launchd)  Default behavior (launchd)

- The app installs a per‑user LaunchAgent labeled `ai.openclaw.gateway`
(or `ai.openclaw.<profile>` when using `--profile`/`OPENCLAW_PROFILE`; legacy `com.openclaw.*` is supported).
- When Local mode is enabled, the app ensures the LaunchAgent is loaded and
starts the Gateway if needed.
- Logs are written to the launchd gateway log path (visible in Debug Settings).

Common commands:

```
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

Replace the label with `ai.openclaw.<profile>` when running a named profile.

## [​](https://docs.openclaw.ai/platforms/mac/child-process\\#unsigned-dev-builds)  Unsigned dev builds

`scripts/restart-mac.sh --no-sign` is for fast local builds when you don’t have
signing keys. To prevent launchd from pointing at an unsigned relay binary, it:

- Writes `~/.openclaw/disable-launchagent`.

Signed runs of `scripts/restart-mac.sh` clear this override if the marker is
present. To reset manually:

```
rm ~/.openclaw/disable-launchagent
```

## [​](https://docs.openclaw.ai/platforms/mac/child-process\\#attach-only-mode)  Attach-only mode

To force the macOS app to **never install or manage launchd**, launch it with
`--attach-only` (or `--no-launchd`). This sets `~/.openclaw/disable-launchagent`,
so the app only attaches to an already running Gateway. You can toggle the same
behavior in Debug Settings.

## [​](https://docs.openclaw.ai/platforms/mac/child-process\\#remote-mode)  Remote mode

Remote mode never starts a local Gateway. The app uses an SSH tunnel to the
remote host and connects over that tunnel.

## [​](https://docs.openclaw.ai/platforms/mac/child-process\\#why-we-prefer-launchd)  Why we prefer launchd

- Auto‑start at login.
- Built‑in restart/KeepAlive semantics.
- Predictable logs and supervision.

If a true child‑process mode is ever needed again, it should be documented as a
separate, explicit dev‑only mode.

## [​](https://docs.openclaw.ai/platforms/mac/child-process\\#related)  Related

- [macOS app](https://docs.openclaw.ai/platforms/macos)
- [Gateway runbook](https://docs.openclaw.ai/gateway)

[Gateway on macOS](https://docs.openclaw.ai/platforms/mac/bundled-gateway) [Health checks (macOS)](https://docs.openclaw.ai/platforms/mac/health)

Ctrl+I

---

## Gateway on macOS - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/bundled-gateway

[Skip to main content](https://docs.openclaw.ai/platforms/mac/bundled-gateway#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nRuntime\n\nGateway on macOS\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Install the CLI (required for local mode)](https://docs.openclaw.ai/platforms/mac/bundled-gateway#install-the-cli-required-for-local-mode)\n- [Launchd (Gateway as LaunchAgent)](https://docs.openclaw.ai/platforms/mac/bundled-gateway#launchd-gateway-as-launchagent)\n- [Version compatibility](https://docs.openclaw.ai/platforms/mac/bundled-gateway#version-compatibility)\n- [Smoke check](https://docs.openclaw.ai/platforms/mac/bundled-gateway#smoke-check)\n- [Related](https://docs.openclaw.ai/platforms/mac/bundled-gateway#related)\n\nOpenClaw.app no longer bundles Node/Bun or the Gateway runtime. The macOS app\nexpects an **external**`openclaw` CLI install, does not spawn the Gateway as a\nchild process, and manages a per‑user launchd service to keep the Gateway\nrunning (or attaches to an existing local Gateway if one is already running).\n\n## [​](https://docs.openclaw.ai/platforms/mac/bundled-gateway\\#install-the-cli-required-for-local-mode)  Install the CLI (required for local mode)\n\nNode 24 is the default runtime on the Mac. Node 22 LTS, currently `22.14+`, still works for compatibility. Then install `openclaw` globally:\n\n```\nnpm install -g openclaw@<version>\n```\n\nThe macOS app’s **Install CLI** button runs the same global install flow the app\nuses internally: it prefers npm first, then pnpm, then bun if that is the only\ndetected package manager. Node remains the recommended Gateway runtime.\n\n## [​](https://docs.openclaw.ai/platforms/mac/bundled-gateway\\#launchd-gateway-as-launchagent)  Launchd (Gateway as LaunchAgent)\n\nLabel:\n\n- `ai.openclaw.gateway` (or `ai.openclaw.<profile>`; legacy `com.openclaw.*` may remain)\n\nPlist location (per‑user):\n\n- `~/Library/LaunchAgents/ai.openclaw.gateway.plist`\n(or `~/Library/LaunchAgents/ai.openclaw.<profile>.plist`)\n\nManager:\n\n- The macOS app owns LaunchAgent install/update in Local mode.\n- The CLI can also install it: `openclaw gateway install`.\n\nBehavior:\n\n- “OpenClaw Active” enables/disables the LaunchAgent.\n- App quit does **not** stop the gateway (launchd keeps it alive).\n- If a Gateway is already running on the configured port, the app attaches to\nit instead of starting a new one.\n\nLogging:\n\n- launchd stdout/err: `/tmp/openclaw/openclaw-gateway.log`\n\n## [​](https://docs.openclaw.ai/platforms/mac/bundled-gateway\\#version-compatibility)  Version compatibility\n\nThe macOS app checks the gateway version against its own version. If they’re\nincompatible, update the global CLI to match the app version.\n\n## [​](https://docs.openclaw.ai/platforms/mac/bundled-gateway\\#smoke-check)  Smoke check\n\n```\nopenclaw --version\n\nOPENCLAW_SKIP_CHANNELS=1 \\\nOPENCLAW_SKIP_CANVAS_HOST=1 \\\nopenclaw gateway --port 18999 --bind loopback\n```\n\nThen:\n\n```\nopenclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000\n```\n\n## [​](https://docs.openclaw.ai/platforms/mac/bundled-gateway\\#related)  Related\n\n- [macOS app](https://docs.openclaw.ai/platforms/macos)\n- [Gateway runbook](https://docs.openclaw.ai/gateway)\n\n[macOS signing](https://docs.openclaw.ai/platforms/mac/signing) [Gateway lifecycle](https://docs.openclaw.ai/platforms/mac/child-process)\n\nCtrl+I

---

## Windows - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/windows

[Skip to main content](https://docs.openclaw.ai/platforms/windows#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Platforms overview

Windows

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [WSL2 (recommended)](https://docs.openclaw.ai/platforms/windows#wsl2-recommended)
- [Native Windows status](https://docs.openclaw.ai/platforms/windows#native-windows-status)
- [Gateway](https://docs.openclaw.ai/platforms/windows#gateway)
- [Gateway service install (CLI)](https://docs.openclaw.ai/platforms/windows#gateway-service-install-cli)
- [Gateway auto-start before Windows login](https://docs.openclaw.ai/platforms/windows#gateway-auto-start-before-windows-login)
- [1) Keep user services running without login](https://docs.openclaw.ai/platforms/windows#1-keep-user-services-running-without-login)
- [2) Install the OpenClaw gateway user service](https://docs.openclaw.ai/platforms/windows#2-install-the-openclaw-gateway-user-service)
- [3) Start WSL automatically at Windows boot](https://docs.openclaw.ai/platforms/windows#3-start-wsl-automatically-at-windows-boot)
- [Verify startup chain](https://docs.openclaw.ai/platforms/windows#verify-startup-chain)
- [Advanced: expose WSL services over LAN (portproxy)](https://docs.openclaw.ai/platforms/windows#advanced-expose-wsl-services-over-lan-portproxy)
- [Step-by-step WSL2 install](https://docs.openclaw.ai/platforms/windows#step-by-step-wsl2-install)
- [1) Install WSL2 + Ubuntu](https://docs.openclaw.ai/platforms/windows#1-install-wsl2-%2B-ubuntu)
- [2) Enable systemd (required for gateway install)](https://docs.openclaw.ai/platforms/windows#2-enable-systemd-required-for-gateway-install)
- [3) Install OpenClaw (inside WSL)](https://docs.openclaw.ai/platforms/windows#3-install-openclaw-inside-wsl)
- [Windows companion app](https://docs.openclaw.ai/platforms/windows#windows-companion-app)
- [Related](https://docs.openclaw.ai/platforms/windows#related)

OpenClaw supports both **native Windows** and **WSL2**. WSL2 is the more
stable path and recommended for the full experience — the CLI, Gateway, and
tooling run inside Linux with full compatibility. Native Windows works for
core CLI and Gateway use, with some caveats noted below.Native Windows companion apps are planned.

## [​](https://docs.openclaw.ai/platforms/windows\\#wsl2-recommended)  WSL2 (recommended)

- [Getting Started](https://docs.openclaw.ai/start/getting-started) (use inside WSL)
- [Install & updates](https://docs.openclaw.ai/install/updating)
- Official WSL2 guide (Microsoft): [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## [​](https://docs.openclaw.ai/platforms/windows\\#native-windows-status)  Native Windows status

Native Windows CLI flows are improving, but WSL2 is still the recommended path.What works well on native Windows today:

- website installer via `install.ps1`
- local CLI use such as `openclaw --version`, `openclaw doctor`, and `openclaw plugins list --json`
- embedded local-agent/provider smoke such as:

```
openclaw agent --local --agent main --thinking low -m \"Reply with exactly WINDOWS-HATCH-OK.\"
```

Current caveats:

- `openclaw onboard --non-interactive` still expects a reachable local gateway unless you pass `--skip-health`
- `openclaw onboard --non-interactive --install-daemon` and `openclaw gateway install` try Windows Scheduled Tasks first
- if Scheduled Task creation is denied, OpenClaw falls back to a per-user Startup-folder login item and starts the gateway immediately
- if `schtasks` itself wedges or stops responding, OpenClaw now aborts that path quickly and falls back instead of hanging forever
- Scheduled Tasks are still preferred when available because they provide better supervisor status

If you want the native CLI only, without gateway service install, use one of these:

```
openclaw onboard --non-interactive --skip-health
openclaw gateway run
```

If you do want managed startup on native Windows:

```
openclaw gateway install
openclaw gateway status --json
```

If Scheduled Task creation is blocked, the fallback service mode still auto-starts after login through the current user’s Startup folder.

## [​](https://docs.openclaw.ai/platforms/windows\\#gateway)  Gateway

- [Gateway runbook](https://docs.openclaw.ai/gateway)
- [Configuration](https://docs.openclaw.ai/gateway/configuration)

## [​](https://docs.openclaw.ai/platforms/windows\\#gateway-service-install-cli)  Gateway service install (CLI)

Inside WSL2:

```
openclaw onboard --install-daemon
```

Or:

```
openclaw gateway install
```

Or:

```
openclaw configure
```

Select **Gateway service** when prompted.Repair/migrate:

```
openclaw doctor
```

## [​](https://docs.openclaw.ai/platforms/windows\\#gateway-auto-start-before-windows-login)  Gateway auto-start before Windows login

For headless setups, ensure the full boot chain runs even when no one logs into\nWindows.

### [​](https://docs.openclaw.ai/platforms/windows\\#1-keep-user-services-running-without-login)  1) Keep user services running without login\n
Inside WSL:\n
```\nsudo loginctl enable-linger \"$(whoami)\"\n```\n
### [​](https://docs.openclaw.ai/platforms/windows\\#2-install-the-openclaw-gateway-user-service)  2) Install the OpenClaw gateway user service\n
Inside WSL:\n
```\nopenclaw gateway install\n```\n
### [​](https://docs.openclaw.ai/platforms/windows\\#3-start-wsl-automatically-at-windows-boot)  3) Start WSL automatically at Windows boot\n
In PowerShell as Administrator:\n
```\nschtasks /create /tn \"WSL Boot\" /tr \"wsl.exe -d Ubuntu --exec /bin/true\" /sc onstart /ru SYSTEM\n```\n
Replace `Ubuntu` with your distro name from:\n
```\nwsl --list --verbose\n```\n
### [​](https://docs.openclaw.ai/platforms/windows\\#verify-startup-chain)  Verify startup chain\n
After a reboot (before Windows sign-in), check from WSL:\n
```\nsystemctl --user is-enabled openclaw-gateway.service\nsystemctl --user status openclaw-gateway.service --no-pager\n```\n
## [​](https://docs.openclaw.ai/platforms/windows\\#advanced-expose-wsl-services-over-lan-portproxy)  Advanced: expose WSL services over LAN (portproxy)\n
WSL has its own virtual network. If another machine needs to reach a service\nrunning **inside WSL** (SSH, a local TTS server, or the Gateway), you must\nforward a Windows port to the current WSL IP. The WSL IP changes after restarts,\nso you may need to refresh the forwarding rule.Example (PowerShell **as Administrator**):\n
```\n$Distro = \"Ubuntu-24.04\"\n$ListenPort = 2222\n$TargetPort = 22\n\n$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(\" \")[0]\nif (-not $WslIp) { throw \"WSL IP not found.\" }\n\nnetsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `\n  connectaddress=$WslIp connectport=$TargetPort\n```\n
Allow the port through Windows Firewall (one-time):\n
```\nNew-NetFirewallRule -DisplayName \"WSL SSH $ListenPort\" -Direction Inbound `\n  -Protocol TCP -LocalPort $ListenPort -Action Allow\n```\n
Refresh the portproxy after WSL restarts:\n
```\nnetsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null\nnetsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `\n  connectaddress=$WslIp connectport=$TargetPort | Out-Null\n```\n
Notes:\n
- SSH from another machine targets the **Windows host IP** (example: `ssh user@windows-host -p 2222`).\n- Remote nodes must point at a **reachable** Gateway URL (not `127.0.0.1`); use\n`openclaw status --all` to confirm.\n- Use `listenaddress=0.0.0.0` for LAN access; `127.0.0.1` keeps it local only.\n- If you want this automatic, register a Scheduled Task to run the refresh\nstep at login.\n
## [​](https://docs.openclaw.ai/platforms/windows\\#step-by-step-wsl2-install)  Step-by-step WSL2 install\n
### [​](https://docs.openclaw.ai/platforms/windows\\#1-install-wsl2-+-ubuntu)  1) Install WSL2 + Ubuntu\n
Open PowerShell (Admin):\n
```\nwsl --install\n# Or pick a distro explicitly:\nwsl --list --online\nwsl --install -d Ubuntu-24.04\n```\n
Reboot if Windows asks.\n
### [​](https://docs.openclaw.ai/platforms/windows\\#2-enable-systemd-required-for-gateway-install)  2) Enable systemd (required for gateway install)\n
In your WSL terminal:\n
```\nsudo tee /etc/wsl.conf >/dev/null <<\'EOF\'\n[boot]\nsystemd=true\nEOF\n```\n
Then from PowerShell:\n
```\nwsl --shutdown\n```\n
Re-open Ubuntu, then verify:\n
```\nsystemctl --user status\n```\n
### [​](https://docs.openclaw.ai/platforms/windows\\#3-install-openclaw-inside-wsl)  3) Install OpenClaw (inside WSL)\n
For a normal first-time setup inside WSL, follow the Linux Getting Started flow:\n
```\ngit clone https://github.com/openclaw/openclaw.git\ncd openclaw\npnpm install\npnpm build\npnpm ui:build\npnpm openclaw onboard --install-daemon\n```\n
If you are developing from source instead of doing first-time onboarding, use the\nsource dev loop from [Setup](https://docs.openclaw.ai/start/setup):\n
```\npnpm install\n# First run only (or after resetting local OpenClaw config/workspace)\npnpm openclaw setup\npnpm gateway:watch\n```\n
Full guide: [Getting Started](https://docs.openclaw.ai/start/getting-started)\n
## [​](https://docs.openclaw.ai/platforms/windows\\#windows-companion-app)  Windows companion app\n
We do not have a Windows companion app yet. Contributions are welcome if you want\ncontributions to make it happen.\n
## [​](https://docs.openclaw.ai/platforms/windows\\#related)  Related\n
- [Install overview](https://docs.openclaw.ai/install)\n- [Platforms](https://docs.openclaw.ai/platforms)\n
[Linux app](https://docs.openclaw.ai/platforms/linux) [Android app](https://docs.openclaw.ai/platforms/android)\n
Ctrl+I

---

## Voice overlay - OpenClaw
**Source:** https://docs.openclaw.ai/platforms/mac/voice-overlay

[Skip to main content](https://docs.openclaw.ai/platforms/mac/voice-overlay#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nFeatures\n\nVoice overlay\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Voice Overlay Lifecycle (macOS)](https://docs.openclaw.ai/platforms/mac/voice-overlay#voice-overlay-lifecycle-macos)\n- [Current intent](https://docs.openclaw.ai/platforms/mac/voice-overlay#current-intent)\n- [Implemented (Dec 9, 2025)](https://docs.openclaw.ai/platforms/mac/voice-overlay#implemented-dec-9-2025)\n- [Next steps](https://docs.openclaw.ai/platforms/mac/voice-overlay#next-steps)\n- [Debugging checklist](https://docs.openclaw.ai/platforms/mac/voice-overlay#debugging-checklist)\n- [Migration steps (suggested)](https://docs.openclaw.ai/platforms/mac/voice-overlay#migration-steps-suggested)\n- [Related](https://docs.openclaw.ai/platforms/mac/voice-overlay#related)\n\n# [​](https://docs.openclaw.ai/platforms/mac/voice-overlay\\#voice-overlay-lifecycle-macos)  Voice Overlay Lifecycle (macOS)\n\nAudience: macOS app contributors. Goal: keep the voice overlay predictable when wake-word and push-to-talk overlap.\n\n## [​](https://docs.openclaw.ai/platforms/mac/voice-overlay\\#current-intent)  Current intent\n\n- If the overlay is already visible from wake-word and the user presses the hotkey, the hotkey session _adopts_ the existing text instead of resetting it. The overlay stays up while the hotkey is held. When the user releases: send if there is trimmed text, otherwise dismiss.\n- Wake-word alone still auto-sends on silence; push-to-talk sends immediately on release.\n\n## [​](https://docs.openclaw.ai/platforms/mac/voice-overlay\\#implemented-dec-9-2025)  Implemented (Dec 9, 2025)\n\n- Overlay sessions now carry a token per capture (wake-word or push-to-talk). Partial/final/send/dismiss/level updates are dropped when the token doesn’t match, avoiding stale callbacks.\n- Push-to-talk adopts any visible overlay text as a prefix (so pressing the hotkey while the wake overlay is up keeps the text and appends new speech). It waits up to 1.5s for a final transcript before falling back to the current text.\n- Chime/overlay logging is emitted at `info` in categories `voicewake.overlay`, `voicewake.ptt`, and `voicewake.chime` (session start, partial, final, send, dismiss, chime reason).\n\n## [​](https://docs.openclaw.ai/platforms/mac/voice-overlay\\#next-steps)  Next steps\n\n1. **VoiceSessionCoordinator (actor)**\n   - Owns exactly one `VoiceSession` at a time.\n   - API (token-based): `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.\n   - Drops callbacks that carry stale tokens (prevents old recognizers from reopening the overlay).\n2. **VoiceSession (model)**\n   - Fields: `token`, `source` (wakeWord\\|pushToTalk), committed/volatile text, chime flags, timers (auto-send, idle), `overlayMode` (display\\|editing\\|sending), cooldown deadline.\n3. **Overlay binding**\n   - `VoiceSessionPublisher` (`ObservableObject`) mirrors the active session into SwiftUI.\n   - `VoiceWakeOverlayView` renders only via the publisher; it never mutates global singletons directly.\n   - Overlay user actions (`sendNow`, `dismiss`, `edit`) call back into the coordinator with the session token.\n4. **Unified send path**\n   - On `endCapture`: if trimmed text is empty → dismiss; else `performSend(session:)` (plays send chime once, forwards, dismisses).\n   - Push-to-talk: no delay; wake-word: optional delay for auto-send.\n   - Apply a short cooldown to the wake runtime after push-to-talk finishes so wake-word doesn’t immediately retrigger.\n5. **Logging**\n   - Coordinator emits `.info` logs in subsystem `ai.openclaw`, categories `voicewake.overlay` and `voicewake.chime`.\n   - Key events: `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.\n\n## [​](https://docs.openclaw.ai/platforms/mac/voice-overlay\\#debugging-checklist)  Debugging checklist\n\n- Stream logs while reproducing a sticky overlay:\n\n\n\n\n\n\n\n\n\n\n\n\n\n```\nsudo log stream --predicate \'subsystem == \"ai.openclaw\" AND category CONTAINS \"voicewake\"\' --level info --style compact\n```\n\n- Verify only one active session token; stale callbacks should be dropped by the coordinator.\n- Ensure push-to-talk release always calls `endCapture` with the active token; if text is empty, expect `dismiss` without chime or send.\n\n## [​](https://docs.openclaw.ai/platforms/mac/voice-overlay\\#migration-steps-suggested)  Migration steps (suggested)\n\n1. Add `VoiceSessionCoordinator`, `VoiceSession`, and `VoiceSessionPublisher`.\n2. Refactor `VoiceWakeRuntime` to create/update/end sessions instead of touching `VoiceWakeOverlayController` directly.\n3. Refactor `VoicePushToTalk` to adopt existing sessions and call `endCapture` on release; apply runtime cooldown.\n4. Wire `VoiceWakeOverlayController` to the publisher; remove direct calls from runtime/PTT.\n5. Add integration tests for session adoption, cooldown, and empty-text dismissal.\n\n## [​](https://docs.openclaw.ai/platforms/mac/voice-overlay\\#related)  Related\n\n- [macOS app](https://docs.openclaw.ai/platforms/macos)\n- [Voice wake (macOS)](https://docs.openclaw.ai/platforms/mac/voicewake)\n- [Talk mode](https://docs.openclaw.ai/nodes/talk)\n\n[Voice wake (macOS)](https://docs.openclaw.ai/platforms/mac/voicewake) [WebChat (macOS)](https://docs.openclaw.ai/platforms/mac/webchat)\n\nCtrl+I

---

