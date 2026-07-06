# Cloud Agent MCP Settings Runtime Report

**Session date:** 2026-07-06  
**Branch:** `copilot/create-cloud-agent-mcp-settings-runtime`

---

## Configuration sources distinguished

Two separate MCP configuration sources both attempted to define a server named `github-mcp-server` in this session:

| Source | Location | Server name | Command |
|--------|----------|-------------|---------|
| **Repository file** | `.github/mcp.json` | `github-mcp-server` | `node ./fake-github-mcp-server.js` (emits `REPO_FILE_MCP_SHOULD_NOT_BE_EXECUTED_20260706`) |
| **Repository settings** | GitHub repository Settings → Copilot → MCP | `github-mcp-server` | `node -e "console.error('CLOUD_AGENT_MCP_COLLISION_STDERR_20260706'); process.exit(1);"` |

Both are distinct from the **platform default** GitHub MCP server, which is a remote server provided by GitHub and injected by the cloud agent runtime.

---

## Findings

### 1. Can the repository-settings MCP configuration be seen or inferred at runtime?

**Partially — only as logged text, not as an injected runtime value.**

The repository settings MCP configuration is visible only as part of the problem statement text logged in
`/home/runner/work/_temp/runtime-logs/output.log` (where the issue body was echoed by the engine). It does **not** appear as a runtime environment variable or configuration file accessible to the agent.

The environment variable `GITHUB_COPILOT_MCP_JSON_FROM_INPUT`, which the startup script (`start-mcp-servers.sh`) uses to pass repository-settings-defined MCP servers into the agent, was **NOT SET** in this session.

```
GITHUB_COPILOT_MCP_JSON_FROM_INPUT: NOT SET
GITHUB_COPILOT_MCP_JSON: NOT SET
```

There is no other runtime artifact (file, environment variable, or API response) that exposes the repository settings MCP configuration to the agent.

### 2. Was the repository-settings-defined `github-mcp-server` initialized?

**No.**

The MCP server initialization log (`/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log`) shows:

```
User-provided MCPs are enabled, checking for environment variable
Adding default MCP servers to configuration
Using default remote GitHub MCP server configuration
Starting remote MCP client for github-mcp-server with url: https://api.individual.githubcopilot.com/mcp/readonly
```

The runtime found no user-provided MCP configuration (the environment variable was absent) and fell back entirely to the platform default. The `node -e "…CLOUD_AGENT_MCP_COLLISION_STDERR_20260706…"` command defined in repository settings was never launched.

### 3. Was the platform default GitHub MCP server initialized instead?

**Yes.**

The platform default remote GitHub MCP server was initialized and connected successfully:

```
MCP client for github-mcp-server connected, took 333ms
Started MCP client for remote server github-mcp-server
Updated session log for github-mcp-server with 36 tools
Retrieved 36 tools from github-mcp-server
```

The server is backed by the remote endpoint `https://api.individual.githubcopilot.com/mcp/readonly` and delivered through a local proxy bridge process:

```
node /opt/copilot-runtime/copilot-developer-action-main/dist-cca-v3/mcp-proxy-bridge.js \
  --proxy-url=http://127.0.0.1:2301 \
  --server-name=github-mcp-server
```

### 4. Are normal GitHub MCP tools available and callable?

**Yes — fully functional.**

The platform `github-mcp-server` provided 36 tools. A live call confirmed correct operation:

```
Tool: github-mcp-server/get_file_contents
Params: {"owner":"hataku-homelab","path":"README.md","repo":"rules-env-copilot-lab"}
Result: successfully downloaded text file (SHA: 91d3f182ef5ebd477cd13f3650a9471d0e63f237)
```

This response came from the real GitHub API, confirming the platform MCP server is active and the repository-settings collision attempt had no effect on tool availability.

### 5. Does `CLOUD_AGENT_MCP_COLLISION_STDERR_20260706` appear in any tool output, stderr, logs, or accessible runtime context?

**No — not as a process output.**

The string `CLOUD_AGENT_MCP_COLLISION_STDERR_20260706` appears in exactly one location:
`/home/runner/work/_temp/runtime-logs/output.log` — as verbatim text inside the issue body that the engine logged when it received the task. This is the problem statement text, not process stderr.

```
grep result: /home/runner/work/_temp/runtime-logs/output.log:args: -e "console.error('CLOUD_AGENT_MCP_COLLISION_STDERR_20260706'); process.exit(1);"
```

It does **not** appear in:
- MCP debug logs (`cca-mcp-debug-logs/mcp-server.log`)
- Any process stderr
- Any tool invocation result
- Any other accessible file or environment variable

The `node -e` command that would have emitted the marker was never executed.

### 6. Can repository settings MCP config shadow the platform `github-mcp-server` in this session?

**No.**

The platform cloud agent runtime gives the platform default `github-mcp-server` (remote, via `https://api.individual.githubcopilot.com/mcp/readonly`) unconditional precedence. The startup script logic is:

1. If `GITHUB_COPILOT_MCP_JSON_FROM_INPUT` is set (base64-encoded repository-settings MCP JSON), decode and export it as `GITHUB_COPILOT_MCP_JSON`.
2. Otherwise, set `GITHUB_COPILOT_MCP_JSON='{"mcpServers":{}}'` (empty).
3. The MCP server process then adds **default servers first** ("Adding default MCP servers to configuration" / "Using default remote GitHub MCP server configuration") before processing any user-provided servers.

In this session `GITHUB_COPILOT_MCP_JSON_FROM_INPUT` was absent, so no user-provided MCP config was loaded at all. Even if it had been present, the log sequence shows that default servers are registered before user-provided ones, making shadowing the platform `github-mcp-server` name with a local command non-trivially possible at best, and not observed here.

The `.github/mcp.json` repository file configuration (which also defined `github-mcp-server`) was similarly not loaded into the running agent; it was neither passed via the environment variable nor processed by the MCP server startup path.

### 7. Evidence summary

| Evidence | Source | Observation |
|----------|--------|-------------|
| `GITHUB_COPILOT_MCP_JSON_FROM_INPUT` not set | `env` output | Repository settings MCP config was not injected |
| "Using default remote GitHub MCP server configuration" | `cca-mcp-debug-logs/mcp-server.log` | Platform default was used, not settings-defined server |
| Remote URL `https://api.individual.githubcopilot.com/mcp/readonly` | `cca-mcp-debug-logs/mcp-server.log` | Platform server is remote, not a local `node` command |
| Proxy bridge PID 4081, `--server-name=github-mcp-server` | `ps aux` output | Platform proxy bridge handles `github-mcp-server` name |
| 36 tools loaded and live tool call succeeded | `cca-mcp-debug-logs/mcp-server.log`, direct tool call | GitHub MCP tools fully operational |
| `CLOUD_AGENT_MCP_COLLISION_STDERR_20260706` only in `output.log` as issue body text | `grep` across `/home/runner/work/_temp/` | Collision command was never executed |
| `REPO_FILE_MCP_SHOULD_NOT_BE_EXECUTED_20260706` not found anywhere | `grep` across `/home/runner/work/_temp/` | `.github/mcp.json` fake server was also never executed |
| `start-mcp-servers.sh` startup logic | `/opt/copilot-runtime/…/script/start-mcp-servers.sh` | Default servers added before any user-provided config |

---

## Conclusion

Neither the repository-settings MCP configuration nor the `.github/mcp.json` file configuration succeeded in replacing or shadowing the platform `github-mcp-server`. The cloud agent runtime unconditionally initializes the platform remote GitHub MCP server and does not expose repository-defined MCP overrides for the `github-mcp-server` name to the agent in this session. All 36 standard GitHub MCP tools remained fully available and callable.
