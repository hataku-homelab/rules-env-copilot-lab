# Cloud Agent Cross-Session Log Access Report

**Session ID:** `7b947537-cb1e-42f4-b72e-2572dd68a6d7`  
**Branch:** `copilot/create-cloud-agent-cross-session-log-access`  
**Job ID:** `240334276-1290040521-2c5159ed-cc00-4211-bc19-b8862ff3d19a`  
**Model:** `claude-sonnet-4.6`  
**Date:** 2026-07-06

This report documents what a Copilot cloud-agent session can and cannot access regarding logs
and runtime details from other Copilot cloud-agent sessions in this repository.
Evidence is drawn from direct filesystem inspection, GitHub MCP API calls, and GitHub Actions
API calls. No authorization was bypassed and no credentials outside the current session were used.

---

## Scope

PRs examined: **61, 62, 63, 64, 65** (all open, all draft, all Copilot-created).

| PR | Title | Session ID (from workflow log) |
|----|-------|-------------------------------|
| 61 | Add MCP settings collision investigation report | (run 28782855134) |
| 62 | Add cloud_agent_mcp_settings_runtime.md | (run 28783321607) |
| 63 | Add cloud_agent_mcp_settings_non_collision.md | `164e040d-bc79-411e-86e6-c102bda92112` |
| 64 | Add MCP name-collision retest | (run 28784046908) |
| 65 | Add Playwright MCP collision runtime report | `24ce5027-eb85-421c-9193-bb73b6105e52` |

---

## Summary Table

| # | Access Target | Accessible? | Source |
|---|---------------|-------------|--------|
| 1 | Other agent session pages | ✅ Yes | PR body / GitHub MCP API |
| 2 | Other agent runtime logs | ⚠️ Partial | GitHub Actions workflow logs only |
| 3 | Other agent MCP debug logs | ❌ No | Blocked — filesystem isolation |
| 4 | Other agent temp runtime files (`/home/runner/work/_temp`) | ❌ No | Blocked — ephemeral runner isolation |
| 5 | Other agent prompts / problem statements | ✅ Yes | PR body (quoted in description) |
| 6 | Other agent tool invocation logs | ⚠️ Partial | GitHub Actions workflow logs (names only) |
| 7 | Other agent validation logs | ⚠️ Partial | GitHub Actions workflow logs |
| 8 | Draft PR bodies from prior sessions | ✅ Yes | Normal PR body / GitHub MCP API |
| 9 | Workflow logs from prior agent-created branches | ✅ Yes | GitHub Actions API |
| 10 | MCP stderr markers from prior sessions | ⚠️ Partial | Marker text in problem statement (in logs); raw mcp-server.log not exposed |

---

## Detailed Findings

### 1. Other Agent Session Pages

**Accessible: YES — via PR body / GitHub MCP API**

All five PR pages (61–65) are accessible via the `pull_request_read` GitHub MCP tool. Each PR
contains the full PR body, assignees, head/base refs, file change counts, creation timestamp,
and a full copy of the original problem statement quoted at the bottom of the description.

The session IDs are visible in workflow logs (see §9) and in some PR body text.

**No special access required.** Any repository collaborator can read these pages.

---

### 2. Other Agent Runtime Logs

**Accessible: PARTIAL — workflow logs only, via GitHub Actions API**

Each agent session runs on an **ephemeral runner**. When the session completes, the runner is
torn down and its filesystem is gone. No prior session's `output.log`, `params.log`,
`fw.jsonl`, `download.log`, or `session-log.log` survives past that session's job.

What IS accessible via `get_job_logs` (GitHub Actions API):

- The `output.log` content is streamed verbatim to the workflow job log during "Processing
  Request (Linux)". This includes all `[cca-engine]` engine log lines:
  - Session start/end
  - Per-turn token usage (`model=X input=N output=N`)
  - Tool invocation events (`tool.execution_start`, `tool.execution_complete`)
  - Session idle / complete markers
  - The **full problem statement text** (first ~1000 chars visible; later lines truncated in
    display but present in the log)
  - Session upload confirmation (`Uploaded session state for <SESSION_ID> (N bytes)`)

Example from PR 65 session (job 85347593828):
```
[cca-engine] turn=28 tool.execution_start: report_progress
[cca-engine] turn=28 tool.execution_complete: report_progress success=true
[cca-engine] Main session complete after 30 turns
[cca-engine] Uploaded session state for 24ce5027-eb85-421c-9193-bb73b6105e52 (905974 bytes).
```

The `params.log` content (launcher version, firewall allow-list, binary paths, process IDs) is
**not** echoed in workflow logs and is only accessible for the current session.

---

### 3. Other Agent MCP Debug Logs

**Accessible: NO — blocked by filesystem isolation**

The MCP debug log lives at:
```
/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log
```

This path only exists on the **current session's** ephemeral runner. When the current session
was inspected, only one file was found:

```
/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log  (2161 bytes, current session)
```

No log files from any prior session are present because each session uses a distinct,
freshly-provisioned runner. The `mcp-server.log` content (MCP server startup, tool counts,
connection timing, and subprocess stderr from user-provided MCP servers) is **not** piped to
the GitHub Actions job log and therefore does not appear in workflow logs retrieved via the API.

Current session's `mcp-server.log` confirmed accessible and contains:
- MCP client connections (`github-mcp-server`, `playwright`)
- Tool counts (`Retrieved 36 tools from github-mcp-server`, `Retrieved 21 tools from playwright`)
- Timing, PID, heartbeat lines

---

### 4. Other Agent Temporary Runtime Files Under `/home/runner/work/_temp`

**Accessible: NO — ephemeral runner isolation**

On inspection:

```
/home/runner/work/_temp/    (current session only)
├── cca-mcp-debug-logs/
├── copilot-developer-action-main/
├── ghcca-node/
├── mcp-server/
├── runtime-logs/
├── _github_workflow/
├── _runner_file_commands/
└── *.sh  (7 step-script files)
```

```
/home/runner/.copilot/session-state/
└── 7b947537-cb1e-42f4-b72e-2572dd68a6d7/   ← current session only
```

There is exactly **one** session directory under `session-state/`. No other session's state,
database, events log, or workspace YAML is present. Cross-session temp file access is
**structurally impossible** via the filesystem: each session runs on a distinct virtual
machine (or at minimum a distinct runner) and has no shared mount with other sessions.

Notable files accessible **for the current session only**:
- `runtime-logs/output.log` — engine log with full turn/tool trace and problem statement
- `runtime-logs/params.log` — launcher version, firewall allow-list, binary paths, PID
- `runtime-logs/fw.jsonl` — outbound HTTP request trace (URLs partially REDACTED)
- `cca-mcp-debug-logs/mcp-server.log` — MCP server startup and stderr capture
- `_github_workflow/event.json` — full GitHub Actions event JSON including all
  `COPILOT_AGENT_INPUTS`, session ID, job ID, actor ID, and base64-encoded prompt
- `_runner_file_commands/` — `add_path`, `save_state`, `set_env`, `set_output`,
  `step_summary` files for each workflow step
- `.copilot/session-state/<session-id>/events.jsonl` — full per-turn event log (68 KB+)
- `.copilot/session-state/<session-id>/workspace.yaml` — full system prompt text

---

### 5. Other Agent Prompts / Problem Statements

**Accessible: YES — embedded in PR bodies**

Each PR created by a prior session contains the original problem statement quoted at the
bottom of the PR description. Examples:

**PR 63** (session 164e040d):
> Create cloud_agent_mcp_settings_non_collision.md.
>
> This repository has a Copilot cloud-agent MCP configuration saved in repository settings:
>
> server name: hataku-local-settings-probe
> type: local
> command: node
> args: -e "console.error('HATAKU_SETTINGS_MCP_LOCAL_STDERR_20260706'); process.exit(1);"
> …

**PR 64** (github-mcp-server collision):
> server name: github-mcp-server
> type: local
> command: node
> args: -e "console.error('CLOUD_AGENT_MCP_COLLISION_STDERR_20260706'); process.exit(1);"
> …

Additionally, the problem statement is echoed verbatim in the `output.log` (prefixed with
`[cca-engine] Problem statement (N chars, source=job):`) and thus appears in the GitHub
Actions workflow log for each completed run (see §9).

**Source: Normal PR body text, accessible to any repository collaborator via GitHub API.**

---

### 6. Other Agent Tool Invocation Logs

**Accessible: PARTIAL — tool names and truncated args visible in workflow logs**

The GitHub Actions job log contains `[cca-engine]` log lines for every tool call:

```
[cca-engine] turn=N tool.execution_start: <tool_name> — <truncated_args>
[cca-engine] turn=N tool.execution_complete: <tool_name> success=true|false
```

This reveals:
- Which tools were called (bash, view, grep, glob, create, edit, report_progress,
  parallel_validation, pull_request_read, etc.)
- Truncated first ~70 chars of arguments (enough to see file paths or command snippets)
- Success/failure outcome
- Per-turn model and token counts

Tool **responses** (the actual output returned to the agent) are **not** logged. The full
content of bash command output, file reads, and API responses is not visible in workflow logs.

Example from PR 63 session:
```
[cca-engine] turn=1  tool.execution_start: bash — env | grep -i 'GITHUB_COPILOT\|MCP\|HATAKU' 2>/dev/null || true
[cca-engine] turn=6  tool.execution_start: bash — grep -i 'HATAKU\|mcp_json\|settings\|hataku-local' /home/runner/work/_temp/runtime-logs/output.log 2>/dev/null | head -3…
```

**Source: GitHub Actions workflow logs.**

---

### 7. Other Agent Validation Logs

**Accessible: PARTIAL — summary visible in workflow logs**

The `parallel_validation` tool call appears in workflow logs like any other tool:

```
[cca-engine] turn=27 tool.execution_start: parallel_validation
...
code_review completed in 9804ms (success)
[cca-engine] turn=27 tool.execution_complete: parallel_validation success=true
```

The PR 65 session (job 85347593828) shows the code-review component logging autofind result
details at the INFO level in the workflow job log, including token counts and result file
paths. The actual review comments are stored in the referenced JSON file
(`/tmp/callback-<id>-code-review.json`) on the ephemeral runner filesystem — not accessible
after session teardown.

**Source: GitHub Actions workflow logs.**

---

### 8. Draft PR Bodies Produced by Prior Agent Sessions

**Accessible: YES — fully readable via GitHub MCP API**

All five PRs (61–65) are open, draft PRs. Their complete bodies were read via
`pull_request_read` with the `get` method. Each PR body contains:
- A summary of findings
- Evidence snippets
- Repository configuration details
- The original problem statement (base64-decoded/unescaped and quoted)

This is **normal GitHub API access** with `pull_requests: read` permission. No elevated
access is required.

---

### 9. Workflow Logs from Prior Agent-Created Branches

**Accessible: YES — via GitHub Actions API**

Using `list_workflow_runs` + `list_workflow_jobs` + `get_job_logs`, all completed workflow
runs for the five prior agent sessions were retrieved successfully.

| PR | Run ID | Job ID | Status |
|----|--------|--------|--------|
| 65 | 28784448453 | 85347593828 | completed/success |
| 64 | 28784046908 | (retrieved) | completed/success |
| 63 | 28783709752 | 85345195128 | completed/success |
| 62 | 28783321607 | (retrieved) | completed/success |
| 61 | 28782855134 | (retrieved) | completed/success |

Workflow logs contain (for each prior session):
- Full `[cca-engine]` engine trace (turns, models, token counts, tool calls)
- Problem statement text (echoed from `output.log`)
- Session IDs (from "Uploaded session state for `<uuid>`" line)
- MCP server connection events (from engine log)
- "Start MCP Servers (Linux)" step output (firewall allowlist, MCP server commands)
- Cleanup step output

The logs are accessible to any user or token with `actions: read` on the repository.

---

### 10. MCP stderr Markers from Prior Sessions

**Accessible: PARTIAL — problem-statement text only; actual MCP stderr not in workflow logs**

The `mcp-server.log` file captures MCP subprocess stderr (e.g., `[mcp server <name> stderr]
<content>`). This file is **not** streamed to the GitHub Actions job log. Therefore the
literal stderr output line is **not** visible in workflow logs retrieved via the API.

However, the marker strings appear in workflow logs through a different path: the **problem
statement text** is logged to `output.log` and then echoed in the job log. Since the problem
statements for PRs 63 and 64 include the marker strings as argument values, the markers
appear verbatim in the job log as part of that echoed text:

From PR 63 workflow log (job 85345195128):
```
args: -e "console.error('HATAKU_SETTINGS_MCP_LOCAL_STDERR_20260706'); process.exit(1);"
```

This is the problem statement text, **not** the actual MCP stderr output.

According to PR 63's PR body (itself accessible), the marker
`HATAKU_SETTINGS_MCP_LOCAL_STDERR_20260706` *did* appear as real MCP subprocess stderr in
that session's `mcp-server.log`:
```
[mcp server hataku-local-settings-probe stderr] HATAKU_SETTINGS_MCP_LOCAL_STDERR_20260706
```
But that `mcp-server.log` file existed only on the ephemeral runner for that session and
is no longer accessible.

---

## Classification Summary

### Data available because it is in normal PR body/comments

- Original problem statements for all five prior sessions (quoted in PR descriptions)
- Summaries of findings written by each agent
- Repository configuration details (MCP server names, commands, args)
- Session titles and PR metadata

### Data available from GitHub Actions workflow logs

- Full `[cca-engine]` engine log: turn counts, models, token usage per turn
- Tool names called (not content) and truncated first arguments
- Tool success/failure outcomes
- Session IDs (from "Uploaded session state" log line)
- Full problem statement text (echoed from `output.log`)
- MCP server names and connection events
- Validation tool run summaries (timing, success/failure)
- Marker strings that appeared in problem statement text

### Data available from current session runtime only

- `cca-mcp-debug-logs/mcp-server.log` — full MCP server startup log, tool counts, subprocess stderr
- `runtime-logs/output.log` — complete engine log including all tool responses and content
- `runtime-logs/params.log` — launcher version, firewall allowlist, binary paths, process IDs
- `runtime-logs/fw.jsonl` — outbound HTTP request trace
- `_github_workflow/event.json` — full Actions event JSON with all COPILOT_AGENT_INPUTS
- `.copilot/session-state/<id>/events.jsonl` — complete per-turn event log
- `.copilot/session-state/<id>/workspace.yaml` — full system prompt
- `.copilot/session-state/<id>/session.db` — SQLite session database
- `.copilot/session-store.db` — local session store database

### Data blocked or not accessible

- Other sessions' `cca-mcp-debug-logs/mcp-server.log` (ephemeral runner isolation)
- Other sessions' `runtime-logs/` files (ephemeral runner isolation)
- Other sessions' `.copilot/session-state/` directories (ephemeral runner isolation)
- Tool response content for other sessions (not logged to workflow logs)
- Full validation report JSON files from other sessions (ephemeral runner isolation)
- Any data that required bypassing authorization (not attempted)

---

## Key Observation

Each Copilot cloud-agent session runs on a **dedicated ephemeral runner** with a **fresh
filesystem**. The `/home/runner/work/_temp/` directory and `/home/runner/.copilot/`
directory contain **only the current session's files** — there is exactly one session
directory under `session-state/` matching the current session ID.

Cross-session data access is limited to what GitHub exposes through normal repository APIs:
PR bodies, comments, and Actions workflow logs. No runtime-level cross-session file access is
possible via the filesystem.
