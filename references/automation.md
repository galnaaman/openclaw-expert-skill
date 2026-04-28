# OpenClaw Automation Documentation

## Task flow - OpenClaw
**Source:** https://docs.openclaw.ai/automation/taskflow

[Skip to main content](https://docs.openclaw.ai/automation/taskflow#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Automation and tasks

Task flow

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [When to use Task Flow](https://docs.openclaw.ai/automation/taskflow#when-to-use-task-flow)
- [Reliable scheduled workflow pattern](https://docs.openclaw.ai/automation/taskflow#reliable-scheduled-workflow-pattern)
- [Sync modes](https://docs.openclaw.ai/automation/taskflow#sync-modes)
- [Managed mode](https://docs.openclaw.ai/automation/taskflow#managed-mode)
- [Mirrored mode](https://docs.openclaw.ai/automation/taskflow#mirrored-mode)
- [Durable state and revision tracking](https://docs.openclaw.ai/automation/taskflow#durable-state-and-revision-tracking)
- [Cancel behavior](https://docs.openclaw.ai/automation/taskflow#cancel-behavior)
- [CLI commands](https://docs.openclaw.ai/automation/taskflow#cli-commands)
- [How flows relate to tasks](https://docs.openclaw.ai/automation/taskflow#how-flows-relate-to-tasks)
- [Related](https://docs.openclaw.ai/automation/taskflow#related)

Task Flow is the flow orchestration substrate that sits above [background tasks](https://docs.openclaw.ai/automation/tasks). It manages durable multi-step flows with their own state, revision tracking, and sync semantics while individual tasks remain the unit of detached work.

## [​](https://docs.openclaw.ai/automation/taskflow\#when-to-use-task-flow)  When to use Task Flow

Use Task Flow when work spans multiple sequential or branching steps and you need durable progress tracking across gateway restarts. For single background operations, a plain [task](https://docs.openclaw.ai/automation/tasks) is sufficient.

| Scenario | Use |
| --- | --- |
| Single background job | Plain task |
| Multi-step pipeline (A then B then C) | Task Flow (managed) |
| Observe externally created tasks | Task Flow (mirrored) |
| One-shot reminder | Cron job |

## [​](https://docs.openclaw.ai/automation/taskflow\#reliable-scheduled-workflow-pattern)  Reliable scheduled workflow pattern

For recurring workflows such as market intelligence briefings, treat the schedule, orchestration, and reliability checks as separate layers:

1. Use [Scheduled Tasks](https://docs.openclaw.ai/automation/cron-jobs) for timing.
2. Use a persistent cron session when the workflow should build on prior context.
3. Use [Lobster](https://docs.openclaw.ai/tools/lobster) for deterministic steps, approval gates, and resume tokens.
4. Use Task Flow to track the multi-step run across child tasks, waits, retries, and gateway restarts.

Example cron shape:

```
openclaw cron add \\
  --name \"Market intelligence brief\" \\
  --cron \"0 7 * * 1-5\" \\
  --tz \"America/New_York\" \\
  --session session:market-intel \\
  --message \"Run the market-intel Lobster workflow. Verify source freshness before summarizing.\" \\
  --announce \\
  --channel slack \\
  --to \"channel:C1234567890\"
```

Use `session:<id>` instead of `isolated` when the recurring workflow needs deliberate history, previous run summaries, or standing context. Use `isolated` when each run should start fresh and all required state is explicit in the workflow.Inside the workflow, put reliability checks before the LLM summary step:

```
name: market-intel-brief
steps:
  - id: preflight
    command: market-intel check --json
  - id: collect
    command: market-intel collect --json
    stdin: $preflight.json
  - id: summarize
    command: market-intel summarize --json
    stdin: $collect.json
  - id: approve
    command: market-intel deliver --preview
    stdin: $summarize.json
    approval: required
  - id: deliver
    command: market-intel deliver --execute
    stdin: $summarize.json
    condition: $approve.approved
```

Recommended preflight checks:

- Browser availability and profile choice, for example `openclaw` for managed state or `user` when a signed-in Chrome session is required. See [Browser](https://docs.openclaw.ai/tools/browser).
- API credentials and quota for each source.
- Network reachability for required endpoints.
- Required tools enabled for the agent, such as `lobster`, `browser`, and `llm-task`.
- Failure destination configured for cron so preflight failures are visible. See [Scheduled Tasks](https://docs.openclaw.ai/automation/cron-jobs#delivery-and-output).

Recommended data provenance fields for every collected item:

```
{
  \"sourceUrl\": \"https://example.com/report\",\n  \"retrievedAt\": \"2026-04-24T12:00:00Z\",\n  \"asOf\": \"2026-04-24\",\n  \"title\": \"Example report\",\n  \"content\": \"...\"
}
```

Have the workflow reject or mark stale items before summarization. The LLM step should receive only structured JSON and should be asked to preserve `sourceUrl`, `retrievedAt`, and `asOf` in its output. Use [LLM Task](https://docs.openclaw.ai/tools/llm-task) when you need a schema-validated model step inside the workflow.For reusable team or community workflows, package the CLI, `.lobster` files, and any setup notes as a skill or plugin and publish it through [ClawHub](https://docs.openclaw.ai/tools/clawhub). Keep workflow-specific guardrails in that package unless the plugin API is missing a needed generic capability.\n
## [​](https://docs.openclaw.ai/automation/taskflow\#sync-modes)  Sync modes

### [​](https://docs.openclaw.ai/automation/taskflow\#managed-mode)  Managed mode

Task Flow owns the lifecycle end-to-end. It creates tasks as flow steps, drives them to completion, and advances the flow state automatically.Example: a weekly report flow that (1) gathers data, (2) generates the report, and (3) delivers it. Task Flow creates each step as a background task, waits for completion, then moves to the next step.

```
Flow: weekly-report
  Step 1: gather-data     → task created → succeeded
  Step 2: generate-report → task created → succeeded
  Step 3: deliver         → task created → running
```

### [​](https://docs.openclaw.ai/automation/taskflow\#mirrored-mode)  Mirrored mode

Task Flow observes externally created tasks and keeps flow state in sync without taking ownership of task creation. This is useful when tasks originate from cron jobs, CLI commands, or other sources and you want a unified view of their progress as a flow.Example: three independent cron jobs that together form a “morning ops” routine. A mirrored flow tracks their collective progress without controlling when or how they run.

## [​](https://docs.openclaw.ai/automation/taskflow\#durable-state-and-revision-tracking)  Durable state and revision tracking

Each flow persists its own state and tracks revisions so progress survives gateway restarts. Revision tracking enables conflict detection when multiple sources attempt to advance the same flow concurrently.
The flow registry uses SQLite with bounded write-ahead-log maintenance, including
periodic and shutdown checkpoints, so long-running gateways do not retain
unbounded `registry.sqlite-wal` sidecar files.

## [​](https://docs.openclaw.ai/automation/taskflow\#cancel-behavior)  Cancel behavior

`openclaw tasks flow cancel` sets a sticky cancel intent on the flow. Active tasks within the flow are cancelled, and no new steps are started. The cancel intent persists across restarts, so a cancelled flow stays cancelled even if the gateway restarts before all child tasks have terminated.

## [​](https://docs.openclaw.ai/automation/taskflow\#cli-commands)  CLI commands

```
# List active and recent flows
openclaw tasks flow list

# Show details for a specific flow
openclaw tasks flow show <lookup>

# Cancel a running flow and its active tasks
openclaw tasks flow cancel <lookup>
```

| Command | Description |
| --- | --- |
| `openclaw tasks flow list` | Shows tracked flows with status and sync mode |
| `openclaw tasks flow show <id>` | Inspect one flow by flow id or lookup key |
| `openclaw tasks flow cancel <id>` | Cancel a running flow and its active tasks |

## [​](https://docs.openclaw.ai/automation/taskflow\#how-flows-relate-to-tasks)  How flows relate to tasks

Flows coordinate tasks, not replace them. A single flow may drive multiple background tasks over its lifetime. Use `openclaw tasks` to inspect individual task records and `openclaw tasks flow` to inspect the orchestrating flow.

## [​](https://docs.openclaw.ai/automation/taskflow\#related)  Related

- [Background Tasks](https://docs.openclaw.ai/automation/tasks) — the detached work ledger that flows coordinate
- [CLI: tasks](https://docs.openclaw.ai/cli/tasks) — CLI command reference for `openclaw tasks flow`
- [Automation Overview](https://docs.openclaw.ai/automation) — all automation mechanisms at a glance
- [Cron Jobs](https://docs.openclaw.ai/automation/cron-jobs) — scheduled jobs that may feed into flows

[Background tasks](https://docs.openclaw.ai/automation/tasks) [Standing orders](https://docs.openclaw.ai/automation/standing-orders)

Ctrl+I

---

## Automation and tasks
**Source:** https://docs.openclaw.ai/automation/tasks

[Skip to main content](https://docs.openclaw.ai/automation/tasks#content-area)

[OpenClaw home page![light logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)![dark logo](https://mintcdn.com/clawdhub/dpADRo8IUoiDztzJ/assets/pixel-lobster.svg?fit=max&auto=format&n=dpADRo8IUoiDztzJ&q=85&s=8fdf719fb6d3eaad7c65231385bf28e5)](https://docs.openclaw.ai/)

![US](https://d3gk2c5xim1je2.cloudfront.net/flags/US.svg)

English

Search...

Ctrl K

Search...

Navigation

Automation and tasks

Background tasks

[Get started](https://docs.openclaw.ai/) [Install](https://docs.openclaw.ai/install) [Channels](https://docs.openclaw.ai/channels) [Agents](https://docs.openclaw.ai/concepts/architecture) [Tools & Plugins](https://docs.openclaw.ai/tools) [Models](https://docs.openclaw.ai/providers) [Platforms](https://docs.openclaw.ai/platforms) [Gateway & Ops](https://docs.openclaw.ai/gateway) [Reference](https://docs.openclaw.ai/cli) [Help](https://docs.openclaw.ai/help)

On this page

- [TL;DR](https://docs.openclaw.ai/automation/tasks#tldr)
- [Quick start](https://docs.openclaw.ai/automation/tasks#quick-start)
- [What creates a task](https://docs.openclaw.ai/automation/tasks#what-creates-a-task)
- [Task lifecycle](https://docs.openclaw.ai/automation/tasks#task-lifecycle)
- [Delivery and notifications](https://docs.openclaw.ai/automation/tasks#delivery-and-notifications)
- [Notification policies](https://docs.openclaw.ai/automation/tasks#notification-policies)
- [CLI reference](https://docs.openclaw.ai/automation/tasks#cli-reference)
- [Chat task board (/tasks)](https://docs.openclaw.ai/automation/tasks#chat-task-board-%2Ftasks)
- [Status integration (task pressure)](https://docs.openclaw.ai/automation/tasks#status-integration-task-pressure)
- [Storage and maintenance](https://docs.openclaw.ai/automation/tasks#storage-and-maintenance)
- [Where tasks live](https://docs.openclaw.ai/automation/tasks#where-tasks-live)
- [Automatic maintenance](https://docs.openclaw.ai/automation/tasks#automatic-maintenance)
- [How tasks relate to other systems](https://docs.openclaw.ai/automation/tasks#how-tasks-relate-to-other-systems)
- [Related](https://docs.openclaw.ai/automation/tasks#related)

Looking for scheduling? See [Automation and tasks](https://docs.openclaw.ai/automation) for choosing the right mechanism. This page is the activity ledger for background work, not the scheduler.

Background tasks track work that runs **outside your main conversation session**: ACP runs, subagent spawns, isolated cron job executions, and CLI-initiated operations.Tasks do **not** replace sessions, cron jobs, or heartbeats — they are the **activity ledger** that records what detached work happened, when, and whether it succeeded.

Not every agent run creates a task. Heartbeat turns and normal interactive chat do not. All cron executions, ACP spawns, subagent spawns, and CLI agent commands do.

## [​](https://docs.openclaw.ai/automation/tasks\\#tldr)  TL;DR

- Tasks are **records**, not schedulers — cron and heartbeat decide _when_ work runs, tasks track _what happened_.
- ACP, subagents, all cron jobs, and CLI operations create tasks. Heartbeat turns do not.
- Each task moves through `queued → running → terminal` (succeeded, failed, timed\\_out, cancelled, or lost).
- Cron tasks stay live while the cron runtime still owns the job; if the\nin-memory runtime state is gone, task maintenance first checks durable cron\nrun history before marking a task lost.
- Completion is push-driven: detached work can notify directly or wake the\nrequester session/heartbeat when it finishes, so status polling loops are\nusually the wrong shape.
- Isolated cron runs and subagent completions best-effort clean up tracked browser tabs/processes for their child session before final cleanup bookkeeping.
- Isolated cron delivery suppresses stale interim parent replies while descendant subagent work is still draining, and it prefers final descendant output when that arrives before delivery.
- Completion notifications are delivered directly to a channel or queued for the next heartbeat.
- `openclaw tasks list` shows all tasks; `openclaw tasks audit` surfaces issues.
- Terminal records are kept for 7 days, then automatically pruned.

## [​](https://docs.openclaw.ai/automation/tasks\\#quick-start)  Quick start

- List and filter

- Inspect

- Cancel and notify

- Audit and maintenance

- Task flow


```\n# List all tasks (newest first)\nopenclaw tasks list\n\n# Filter by runtime or status\nopenclaw tasks list --runtime acp\nopenclaw tasks list --status running\n```

```\n# Show details for a specific task (by ID, run ID, or session key)\nopenclaw tasks show <lookup>\n```

```\n# Cancel a running task (kills the child session)\nopenclaw tasks cancel <lookup>\n\n# Change notification policy for a task\nopenclaw tasks notify <lookup> state_changes\n```

```\n# Run a health audit\nopenclaw tasks audit\n\n# Preview or apply maintenance\nopenclaw tasks maintenance\nopenclaw tasks maintenance --apply\n```

```\n# Inspect TaskFlow state\nopenclaw tasks flow list\nopenclaw tasks flow show <lookup>\nopenclaw tasks flow cancel <lookup>\n```

## [​](https://docs.openclaw.ai/automation/tasks\\#what-creates-a-task)  What creates a task

| Source | Runtime type | When a task record is created | Default notify policy |
| --- | --- | --- | --- |
| ACP background runs | `acp` | Spawning a child ACP session | `done_only` |
| Subagent orchestration | `subagent` | Spawning a subagent via `sessions_spawn` | `done_only` |
| Cron jobs (all types) | `cron` | Every cron execution (main-session and isolated) | `silent` |
| CLI operations | `cli` | `openclaw agent` commands that run through the gateway | `silent` |
| Agent media jobs | `cli` | Session-backed `video_generate` runs | `silent` |

Notify defaults for cron and media

Main-session cron tasks use `silent` notify policy by default — they create records for tracking but do not generate notifications. Isolated cron tasks also default to `silent` but are more visible because they run in their own session.Session-backed `video_generate` runs also use `silent` notify policy. They still create task records, but completion is handed back to the original agent session as an internal wake so the agent can write the follow-up message and attach the finished video itself. If you opt into `tools.media.asyncCompletion.directSend`, async `music_generate` and `video_generate` completions try direct channel delivery first before falling back to the requester-session wake path.

Concurrent video\\_generate guardrail

While a session-backed `video_generate` task is still active, the tool also acts as a guardrail: repeated `video_generate` calls in that same session return the active task status instead of starting a second concurrent generation. Use `action: \"status\"` when you want an explicit progress/status lookup from the agent side.

What does not create tasks

- Heartbeat turns — main-session; see [Heartbeat](https://docs.openclaw.ai/gateway/heartbeat)
- Normal interactive chat turns
- Direct `/command` responses

## [​](https://docs.openclaw.ai/automation/tasks\\#task-lifecycle)  Task lifecycle

agent starts

completes ok

error

timeout exceeded

operator cancels

session gone > 5 min

session gone > 5 min

queued

running

succeeded

failed

timed\\_out

cancelled

lost

| Status | What it means |
| --- | --- |
| `queued` | Created, waiting for the agent to start |
| `running` | Agent turn is actively executing |
| `succeeded` | Completed successfully |
| `failed` | Completed with an error |
| `timed_out` | Exceeded the configured timeout |
| `cancelled` | Stopped by the operator via `openclaw tasks cancel` |
| `lost` | The runtime lost authoritative backing state after a 5-minute grace period |

Transitions happen automatically — when the associated agent run ends, the task status updates to match.Agent run completion is authoritative for active task records. A successful detached run finalizes as `succeeded`, ordinary run errors finalize as `failed`, and timeout or abort outcomes finalize as `timed_out`. If an operator already cancelled the task, or the runtime already recorded a stronger terminal state such as `failed`, `timed_out`, or `lost`, a later success signal does not downgrade that terminal status.`lost` is runtime-aware:\n
- ACP tasks: backing ACP child session metadata disappeared.\n- Subagent tasks: backing child session disappeared from the target agent store.\n- Cron tasks: the cron runtime no longer tracks the job as active and durable\ncron run history does not show a terminal result for that run. Offline CLI\naudit does not treat its own empty in-process cron runtime state as authority.\n- CLI tasks: isolated child-session tasks use the child session; chat-backed\nCLI tasks use the live run context instead, so lingering\nchannel/group/direct session rows do not keep them alive. Gateway-backed\n`openclaw agent` runs also finalize from their run result, so completed runs\ndo not sit active until the sweeper marks them `lost`.\n
## [​](https://docs.openclaw.ai/automation/tasks\\#delivery-and-notifications)  Delivery and notifications

When a task reaches a terminal state, OpenClaw notifies you. There are two delivery paths:**Direct delivery** — if the task has a channel target (the `requesterOrigin`), the completion message goes straight to that channel (Telegram, Discord, Slack, etc.). For subagent completions, OpenClaw also preserves bound thread/topic routing when available and can fill a missing `to` / account from the requester session’s stored route (`lastChannel` / `lastTo` / `lastAccountId`) before giving up on direct delivery.**Session-queued delivery** — if direct delivery fails or no origin is set, the update is queued as a system event in the requester’s session and surfaces on the next heartbeat.

Task completion triggers an immediate heartbeat wake so you see the result quickly — you do not have to wait for the next scheduled heartbeat tick.

That means the usual workflow is push-based: start detached work once, then let the runtime wake or notify you on completion. Poll task state only when you need debugging, intervention, or an explicit audit.

### [​](https://docs.openclaw.ai/automation/tasks\\#notification-policies)  Notification policies

Control how much you hear about each task:\n
| Policy | What is delivered |
| --- | --- |
| `done_only` (default) | Only terminal state (succeeded, failed, etc.) — **this is the default** |
| `state_changes` | Every state transition and progress update |
| `silent` | Nothing at all |

Change the policy while a task is running:\n
```\nopenclaw tasks notify <lookup> state_changes\n```

## [​](https://docs.openclaw.ai/automation/tasks\\#cli-reference)  CLI reference

tasks list

```\nopenclaw tasks list [--runtime <acp|subagent|cron|cli>] [--status <status>] [--json]\n```

Output columns: Task ID, Kind, Status, Delivery, Run ID, Child Session, Summary.\n
tasks show

```\nopenclaw tasks show <lookup>\n```

The lookup token accepts a task ID, run ID, or session key. Shows the full record including timing, delivery state, error, and terminal summary.\n
tasks cancel

```\nopenclaw tasks cancel <lookup>\n```

For ACP and subagent tasks, this kills the child session. For CLI-tracked tasks, cancellation is recorded in the task registry (there is no separate child runtime handle). Status transitions to `cancelled` and a delivery notification is sent when applicable.\n
tasks notify

```\nopenclaw tasks notify <lookup> <done_only|state_changes|silent>\n```

tasks audit

```\nopenclaw tasks audit [--json]\n```

Surfaces operational issues. Findings also appear in `openclaw status` when issues are detected.\n
| Finding | Severity | Trigger |
| --- | --- | --- |
| `stale_queued` | warn | Queued for more than 10 minutes |
| `stale_running` | error | Running for more than 30 minutes |
| `lost` | warn/error | Runtime-backed task ownership disappeared; retained lost tasks warn until `cleanupAfter`, then become errors |
| `delivery_failed` | warn | Delivery failed and notify policy is not `silent` |
| `missing_cleanup` | warn | Terminal task with no cleanup timestamp |
| `inconsistent_timestamps` | warn | Timeline violation (for example ended before started) |

tasks maintenance

```\nopenclaw tasks maintenance [--json]\nopenclaw tasks maintenance --apply [--json]\n```

Use this to preview or apply reconciliation, cleanup stamping, and pruning for tasks and Task Flow state.Reconciliation is runtime-aware:\n
- ACP/subagent tasks check their backing child session.\n- Cron tasks check whether the cron runtime still owns the job, then recover terminal status from persisted cron run logs/job state before falling back to `lost`. Only the Gateway process is authoritative for the in-memory cron active-job set; offline CLI audit uses durable history but does not mark a cron task lost solely because that local Set is empty.\n- Chat-backed CLI tasks check the owning live run context, not just the chat session row.\n
Completion cleanup is also runtime-aware:\n
- Subagent completion best-effort closes tracked browser tabs/processes for the child session before announce cleanup continues.\n- Isolated cron completion best-effort closes tracked browser tabs/processes for the cron session before the run fully tears down.\n- Isolated cron delivery waits out descendant subagent follow-up when needed and suppresses stale parent acknowledgement text instead of announcing it.\n- Subagent completion delivery prefers the latest visible assistant text; if that is empty it falls back to sanitized latest tool/toolResult text, and timeout-only tool-call runs can collapse to a short partial-progress summary. Terminal failed runs announce failure status without replaying captured reply text.\n- Cleanup failures do not mask the real task outcome.\n
tasks flow list \\| show \\| cancel\n
```\nopenclaw tasks flow list [--status <status>] [--json]\nopenclaw tasks flow show <lookup> [--json]\nopenclaw tasks flow cancel <lookup>\n```

Use these when the orchestrating Task Flow is the thing you care about rather than one individual background task record.\n
## [​](https://docs.openclaw.ai/automation/tasks\\#chat-task-board-/tasks)  Chat task board (`/tasks`)

Use `/tasks` in any chat session to see background tasks linked to that session. The board shows active and recently completed tasks with runtime, status, timing, and progress or error detail.When the current session has no visible linked tasks, `/tasks` falls back to agent-local task counts so you still get an overview without leaking other-session details.For the full operator ledger, use the CLI: `openclaw tasks list`.

## [​](https://openclaw.ai/automation/tasks\\#status-integration-task-pressure)  Status integration (task pressure)

`openclaw status` includes an at-a-glance task summary:\n
```\nTasks: 3 queued · 2 running · 1 issues\n```

The summary reports:\n
- **active** — count of `queued` \\+ `running`\n- **failures** — count of `failed` \\+ `timed_out` \\+ `lost`\n- **byRuntime** — breakdown by `acp`, `subagent`, `cron`, `cli`\n
Both `/status` and the `session_status` tool use a cleanup-aware task snapshot: active tasks are preferred, stale completed rows are hidden, and recent failures only surface when no active work remains. This keeps the status card focused on what matters right now.\n
## [​](https://openclaw.ai/automation/tasks\\#storage-and-maintenance)  Storage and maintenance\n
### [​](https://docs.openclaw.ai/automation/tasks\\#where-tasks-live)  Where tasks live

Task records persist in SQLite at:\n
```\n$OPENCLAW_STATE_DIR/tasks/runs.sqlite\n```

The registry loads into memory at gateway start and syncs writes to SQLite for durability across restarts.\nThe Gateway keeps the SQLite write-ahead log bounded by using SQLite’s default\nautocheckpoint threshold plus periodic and shutdown `TRUNCATE` checkpoints.\n
### [​](https://docs.openclaw.ai/automation/tasks\\#automatic-maintenance)  Automatic maintenance\n
A sweeper runs every **60 seconds** and handles three things:\n
1\n
[Navigate to header](https://docs.openclaw.ai/automation/tasks#)\n
Reconciliation\n
Checks whether active tasks still have authoritative runtime backing. ACP/subagent tasks use child-session state, cron tasks use active-job ownership, and chat-backed CLI tasks use the owning run context. If that backing state is gone for more than 5 minutes, the task is marked `lost`.\n
2\n
[Navigate to header](https://docs.openclaw.ai/automation/tasks#)\n
Cleanup stamping\n
Sets a `cleanupAfter` timestamp on terminal tasks (endedAt + 7 days). During retention, lost tasks still appear in audit as warnings; after `cleanupAfter` expires or when cleanup metadata is missing, they are errors.\n
3\n
[Navigate to header](https://docs.openclaw.ai/automation/tasks#)\n
Pruning\n
Deletes records past their `cleanupAfter` date.\n
**Retention:** terminal task records are kept for **7 days**, then automatically pruned. No configuration needed.\n
## [​](https://docs.openclaw.ai/automation/tasks\\#how-tasks-relate-to-other-systems)  How tasks relate to other systems\n
Tasks and Task Flow\n
[Task Flow](https://docs.openclaw.ai/automation/taskflow) is the flow orchestration layer above background tasks. A single flow may coordinate multiple tasks over its lifetime using managed or mirrored sync modes. Use `openclaw tasks` to inspect individual task records and `openclaw tasks flow` to inspect the orchestrating flow.See [Task Flow](https://docs.openclaw.ai/automation/taskflow) for details.\n
Tasks and cron\n
A cron job **definition** lives in `~/.openclaw/cron/jobs.json`; runtime execution state lives beside it in `~/.openclaw/cron/jobs-state.json`. **Every** cron execution creates a task record — both main-session and isolated. Main-session cron tasks default to `silent` notify policy so they track without generating notifications.See [Cron Jobs](https://docs.openclaw.ai/automation/cron-jobs).\n
Tasks and heartbeat\n
A heartbeat runs are main-session turns — they do not create task records. When a task completes, it can trigger a heartbeat wake so you see the result promptly.See [Heartbeat](https://docs.openclaw.ai/gateway/heartbeat).\n
Tasks and sessions\n
A task may reference a `childSessionKey` (where work runs) and a `requesterSessionKey` (who started it). Sessions are conversation context; tasks are activity tracking on top of that.\n
Tasks and agent runs\n
A task’s `runId` links to the agent run doing the work. Agent lifecycle events (start, end, error) automatically update the task status — you do not need to manage the lifecycle manually.\n
## [​](https://docs.openclaw.ai/automation/tasks\\#related)  Related\n
- [Automation & Tasks](https://docs.openclaw.ai/automation) — all automation mechanisms at a glance\n- [CLI: Tasks](https://docs.openclaw.ai/cli/tasks) — CLI command reference\n- [Heartbeat](https://docs.openclaw.ai/gateway/heartbeat) — periodic main-session turns\n- [Scheduled Tasks](https://docs.openclaw.ai/automation/cron-jobs) — scheduling background work\n- [Task Flow](https://docs.openclaw.ai/automation/taskflow) — flow orchestration above tasks\n\n[Scheduled tasks](https://docs.openclaw.ai/automation/cron-jobs) [Task flo...(content truncated)

---

