# OpenClaw Security Documentation

## Contributing to the threat model - OpenClaw
**Source:** https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL

[Skip to main content](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#content-area)\n\n[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)\n\n![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)\n\nEnglish\n\nSearch...\n\nCtrl K\n\nSearch...\n\nNavigation\n\nSecurity\n\nContributing to the threat model\n\n[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)\n\nOn this page\n\n- [Contributing to the OpenClaw Threat Model](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#contributing-to-the-openclaw-threat-model)\n- [Ways to Contribute](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#ways-to-contribute)\n- [Add a Threat](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#add-a-threat)\n- [Suggest a Mitigation](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#suggest-a-mitigation)\n- [Propose an Attack Chain](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#propose-an-attack-chain)\n- [Fix or Improve Existing Content](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#fix-or-improve-existing-content)\n- [What we use](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#what-we-use)\n- [MITRE ATLAS](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#mitre-atlas)\n- [Threat IDs](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#threat-ids)\n- [Risk levels](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#risk-levels)\n- [Review process](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#review-process)\n- [Resources](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#resources)\n- [Contact](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#contact)\n- [Recognition](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#recognition)\n- [Related](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL#related)\n\n# [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#contributing-to-the-openclaw-threat-model)  Contributing to the OpenClaw Threat Model\n\nThanks for helping make OpenClaw more secure. This threat model is a living document and we welcome contributions from anyone - you don’t need to be a security expert.\n\n## [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#ways-to-contribute)  Ways to Contribute\n\n### [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#add-a-threat)  Add a Threat\n\nSpotted an attack vector or risk we haven’t covered? Open an issue on [openclaw/trust](https://github.com/openclaw/trust/issues) and describe it in your own words. You don’t need to know any frameworks or fill in every field - just describe the scenario.**Helpful to include (but not required):**\n\n- The attack scenario and how it could be exploited\n- Which parts of OpenClaw are affected (CLI, gateway, channels, ClawHub, MCP servers, etc.)\n- How severe you think it is (low / medium / high / critical)\n- Any links to related research, CVEs, or real-world examples\n\nWe’ll handle the ATLAS mapping, threat IDs, and risk assessment during review. If you want to include those details, great - but it’s not expected.\n\n> **This is for adding to the threat model, not reporting live vulnerabilities.** If you’ve found an exploitable vulnerability, see our [Trust page](https://trust.openclaw.ai/) for responsible disclosure instructions.\n\n### [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#suggest-a-mitigation)  Suggest a Mitigation\n\nHave an idea for how to address an existing threat? Open an issue or PR referencing the threat. Useful mitigations are specific and actionable - for example, “per-sender rate limiting of 10 messages/minute at the gateway” is better than “implement rate limiting.”\n\n### [​](https://openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#propose-an-attack-chain)  Propose an Attack Chain\n\nAttack chains show how multiple threats combine into a realistic attack scenario. If you see a dangerous combination, describe the steps and how an attacker would chain them together. A short narrative of how the attack unfolds in practice is more valuable than a formal template.\n\n### [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#fix-or-improve-existing-content)  Fix or Improve Existing Content\n\nTypos, clarifications, outdated info, better examples - PRs welcome, no issue needed.\n\n## [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#what-we-use)  What we use\n\n### [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#mitre-atlas)  MITRE ATLAS\n\nThis threat model is built on [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), a framework designed specifically for AI/ML threats like prompt injection, tool misuse, and agent exploitation. You don’t need to know ATLAS to contribute - we map submissions to the framework during review.\n\n### [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#threat-ids)  Threat IDs\n\nEach threat gets an ID like `T-EXEC-003`. The categories are:\n\n| Code | Category |\n| --- | --- |\n| RECON | Reconnaissance - information gathering |\n| ACCESS | Initial access - gaining entry |\n| EXEC | Execution - running malicious actions |\n| PERSIST | Persistence - maintaining access |\n| EVADE | Defense evasion - avoiding detection |\n| DISC | Discovery - learning about the environment |\n| EXFIL | Exfiltration - stealing data |\n| IMPACT | Impact - damage or disruption |\n\nIDs are assigned by maintainers during review. You don’t need to pick one.\n\n### [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#risk-levels)  Risk levels\n\n| Level | Meaning |\n| --- | --- |\n| **Critical** | Full system compromise, or high likelihood + critical impact |\n| **High** | Significant damage likely, or medium likelihood + critical impact |\n| **Medium** | Moderate risk, or low likelihood + high impact |\n| **Low** | Unlikely and limited impact |\n\nIf you’re unsure about the risk level, just describe the impact and we’ll assess it.\n\n## [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#review-process)  Review process\n\n1. **Triage** \\- We review new submissions within 48 hours\n2. **Assessment** \\- We verify feasibility, assign ATLAS mapping and threat ID, validate risk level\n3. **Documentation** \\- We ensure everything is formatted and complete\n4. **Merge** \\- Added to the threat model and visualization\n\n## [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#resources)  Resources\n\n- [ATLAS Website](https://atlas.mitre.org/)\n- [ATLAS Techniques](https://atlas.mitre.org/techniques/)\n- [ATLAS Case Studies](https://atlas.mitre.org/studies/)\n- [OpenClaw Threat Model](https://docs.openclaw.ai/security/THREAT-MODEL-ATLAS)\n\n## [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#contact)  Contact\n\n- **Security vulnerabilities:** See our [Trust page](https://trust.openclaw.ai/) for reporting instructions\n- **Threat model questions:** Open an issue on [openclaw/trust](https://github.com/openclaw/trust/issues)\n- **General chat:** Discord #security channel\n\n## [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#recognition)  Recognition\n\nContributors to the threat model are recognized in the threat model acknowledgments, release notes, and the OpenClaw security hall of fame for significant contributions.\n\n## [​](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL\\#related)  Related\n\n- [Threat model](https://docs.openclaw.ai/security/THREAT-MODEL-ATLAS)\n- [Formal verification](https://docs.openclaw.ai/security/formal-verification)\n\n[Threat model (MITRE ATLAS)](https://docs.openclaw.ai/security/THREAT-MODEL-ATLAS) [Nodes](https://docs.openclaw.ai/nodes)\n\nCtrl+I

---

## Formal verification (security models) - OpenClaw
**Source:** https://docs.openclaw.ai/security/formal-verification

[Skip to main content](https://docs.openclaw.ai/security/formal-verification#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Security

Formal verification (security models)

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [Where the models live](https://docs.openclaw.ai/security/formal-verification#where-the-models-live)
- [Important caveats](https://docs.openclaw.ai/security/formal-verification#important-caveats)
- [Reproducing results](https://docs.openclaw.ai/security/formal-verification#reproducing-results)
- [Gateway exposure and open gateway misconfiguration](https://docs.openclaw.ai/security/formal-verification#gateway-exposure-and-open-gateway-misconfiguration)
- [Node exec pipeline (highest-risk capability)](https://docs.openclaw.ai/security/formal-verification#node-exec-pipeline-highest-risk-capability)
- [Pairing store (DM gating)](https://docs.openclaw.ai/security/formal-verification#pairing-store-dm-gating)
- [Ingress gating (mentions + control-command bypass)](https://docs.openclaw.ai/security/formal-verification#ingress-gating-mentions-%2B-control-command-bypass)
- [Routing/session-key isolation](https://docs.openclaw.ai/security/formal-verification#routing%2Fsession-key-isolation)
- [v1++: additional bounded models (concurrency, retries, trace correctness)](https://docs.openclaw.ai/security/formal-verification#v1%2B%2B-additional-bounded-models-concurrency-retries-trace-correctness)
- [Pairing store concurrency / idempotency](https://docs.openclaw.ai/security/formal-verification#pairing-store-concurrency-%2F-idempotency)
- [Ingress trace correlation / idempotency](https://docs.openclaw.ai/security/formal-verification#ingress-trace-correlation-%2F-idempotency)
- [Routing dmScope precedence + identityLinks](https://docs.openclaw.ai/security/formal-verification#routing-dmscope-precedence-%2B-identitylinks)
- [Related](https://docs.openclaw.ai/security/formal-verification#related)

This page tracks OpenClaw’s **formal security models** (TLA+/TLC today; more as needed).

> Note: some older links may refer to the previous project name.

**Goal (north star):** provide a machine-checked argument that OpenClaw enforces its
intended security policy (authorization, session isolation, tool gating, and
misconfiguration safety), under explicit assumptions.**What this is (today):** an executable, attacker-driven **security regression suite**:

- Each claim has a runnable model-check over a finite state space.
- Many claims have a paired **negative model** that produces a counterexample trace for a realistic bug class.

**What this is not (yet):** a proof that “OpenClaw is secure in all respects” or that the full TypeScript implementation is correct.

## [​](https://docs.openclaw.ai/security/formal-verification\\#where-the-models-live)  Where the models live

Models are maintained in a separate repo: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

## [​](https://docs.openclaw.ai/security/formal-verification\\#important-caveats)  Important caveats

- These are **models**, not the full TypeScript implementation. Drift between model and code is possible.
- Results are bounded by the state space explored by TLC; “green” does not imply security beyond the modeled assumptions and bounds.
- Some claims rely on explicit environmental assumptions (e.g., correct deployment, correct configuration inputs).

## [​](https://docs.openclaw.ai/security/formal-verification\\#reproducing-results)  Reproducing results

Today, results are reproduced by cloning the models repo locally and running TLC (see below). A future iteration could offer:

- CI-run models with public artifacts (counterexample traces, run logs)
- a hosted “run this model” workflow for small, bounded checks

Getting started:

```
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ required (TLC runs on the JVM).
# The repo vendors a pinned `tla2tools.jar` (TLA+ tools) and provides `bin/tlc` + Make targets.

make <target>
```

### [​](https://docs.openclaw.ai/security/formal-verification\\#gateway-exposure-and-open-gateway-misconfiguration)  Gateway exposure and open gateway misconfiguration

**Claim:** binding beyond loopback without auth can make remote compromise possible / increases exposure; token/password blocks unauth attackers (per the model assumptions).

- Green runs:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Red (expected):
  - `make gateway-exposure-v2-negative`

See also: `docs/gateway-exposure-matrix.md` in the models repo.

### [​](https://docs.openclaw.ai/security/formal-verification\\#node-exec-pipeline-highest-risk-capability)  Node exec pipeline (highest-risk capability)

**Claim:**`exec host=node` requires (a) node command allowlist plus declared commands and (b) live approval when configured; approvals are tokenized to prevent replay (in the model).

- Green runs:
  - `make nodes-pipeline`
  - `make approvals-token`
- Red (expected):
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

### [​](https://docs.openclaw.ai/security/formal-verification\\#pairing-store-dm-gating)  Pairing store (DM gating)

**Claim:** pairing requests respect TTL and pending-request caps.

- Green runs:
  - `make pairing`
  - `make pairing-cap`
- Red (expected):
  - `make pairing-negative`
  - `make pairing-cap-negative`

### [​](https://docs.openclaw.ai/security/formal-verification\\#ingress-gating-mentions-+-control-command-bypass)  Ingress gating (mentions + control-command bypass)

**Claim:** in group contexts requiring mention, an unauthorized “control command” cannot bypass mention gating.

- Green:
  - `make ingress-gating`
- Red (expected):
  - `make ingress-gating-negative`

### [​](https://docs.openclaw.ai/security/formal-verification\\#routing/session-key-isolation)  Routing/session-key isolation

**Claim:** DMs from distinct peers do not collapse into the same session unless explicitly linked/configured.

- Green:
  - `make routing-isolation`
- Red (expected):
  - `make routing-isolation-negative`

## [​](https://docs.openclaw.ai/security/formal-verification\\#v1++-additional-bounded-models-concurrency-retries-trace-correctness)  v1++: additional bounded models (concurrency, retries, trace correctness)

These are follow-on models that tighten fidelity around real-world failure modes (non-atomic updates, retries, and message fan-out).

### [​](https://docs.openclaw.ai/security/formal-verification\\#pairing-store-concurrency-/-idempotency)  Pairing store concurrency / idempotency

**Claim:** a pairing store should enforce `MaxPending` and idempotency even under interleavings (i.e., “check-then-write” must be atomic / locked; refresh shouldn’t create duplicates).What it means:

- Under concurrent requests, you can’t exceed `MaxPending` for a channel.
- Repeated requests/refreshes for the same `(channel, sender)` should not create duplicate live pending rows.
- Green runs:  - `make pairing-race` (atomic/locked cap check)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- Red (expected):  - `make pairing-race-negative` (non-atomic begin/commit cap race)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

### [​](https://docs.openclaw.ai/security/formal-verification\\#ingress-trace-correlation-/-idempotency)  Ingress trace correlation / idempotency

**Claim:** ingestion should preserve trace correlation across fan-out and be idempotent under provider retries.What it means:

- When one external event becomes multiple internal messages, every part keeps the same trace/event identity.
- Retries do not result in double-processing.
- If provider event IDs are missing, dedupe falls back to a safe key (e.g., trace ID) to avoid dropping distinct events.
- Green:  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Red (expected):  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

### [​](https://docs.openclaw.ai/security/formal-verification\\#routing-dmscope-precedence-+-identitylinks)  Routing dmScope precedence + identityLinks

**Claim:** routing must keep DM sessions isolated by default, and only collapse sessions when explicitly configured (channel precedence + identity links).What it means:

- Channel-specific dmScope overrides must win over global defaults.
- identityLinks should collapse only within explicit linked groups, not across unrelated peers.
- Green:  - `make routing-precedence`
  - `make routing-identitylinks`
- Red (expected):  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`

## [​](https://docs.openclaw.ai/security/formal-verification\\#related)  Related

- [Threat model](https://docs.openclaw.ai/security/THREAT-MODEL-ATLAS)
- [Contributing to the threat model](https://docs.openclaw.ai/security/CONTRIBUTING-THREAT-MODEL)

[Network proxy](https://docs.openclaw.ai/security/network-proxy) [Threat model (MITRE ATLAS)](https://docs.openclaw.ai/security/THREAT-MODEL-ATLAS)

Ctrl+I

---

