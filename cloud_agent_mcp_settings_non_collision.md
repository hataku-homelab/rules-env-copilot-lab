# Cloud Agent MCP Settings Non-Collision Report

**Date:** 2026-07-06  
**Repository:** hataku-homelab/rules-env-copilot-lab  
**Branch:** copilot/create-cloud-agent-mcp-settings

## Summary

This report documents the runtime behavior of the Copilot cloud agent with respect to an MCP server defined in repository settings:

- **Server name:** `hataku-local-settings-probe`
- **Type:** local
- **Command:** `node`
- **Args:** `-e "console.error('HATAKU_SETTINGS_MCP_LOCAL_STDERR_20260706'); process.exit(1);"`

The `.github/mcp.json` file does not exist in this repository; no file-based MCP configuration was present.

---

## Findings

### 1. Was `hataku-local-settings-probe` initialized?

**Yes.** The cloud agent attempted to start the server. The following sequence was recorded in `/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log`:

```
Starting MCP client for hataku-local-settings-probe with
command: node
args: -e console.error('HATAKU_SETTINGS_MCP_LOCAL_STDERR_20260706'); process.exit(1);
cwd: /home/runner/work/rules-env-copilot-lab/rules-env-copilot-lab
Creating MCP client for hataku-local-settings-probe...
Connecting MCP client for hataku-local-settings-probe...
```

The process exited immediately (exit code 1), causing the connection to close:

```
MCP transport for hataku-local-settings-probe closed
Failed to start MCP client for hataku-local-settings-probe: McpError: MCP error -32000: Connection closed
Recorded failure for server hataku-local-settings-probe: MCP error -32000: Connection closed
```

The server was initialized (launched) but failed to establish a functioning MCP connection.

### 2. Did `HATAKU_SETTINGS_MCP_LOCAL_STDERR_20260706` appear in logs, stderr, tool output, environment, or runtime files?

**Yes, in the MCP debug log.** The stderr output of the spawned process was captured and recorded in `/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log`:

```
[mcp server hataku-local-settings-probe stderr] HATAKU_SETTINGS_MCP_LOCAL_STDERR_20260706
```

The marker did **not** appear in the process environment, in `/tmp/mcp-server.log`, or in any other runtime file beyond the MCP debug log.

### 3. Were `GITHUB_COPILOT_MCP_JSON_FROM_INPUT` or `GITHUB_COPILOT_MCP_JSON` set?

**No.** Neither variable was present in the agent's environment. A full scan of the environment confirmed only the following MCP-related variables were set:

| Variable | Value |
|---|---|
| `COPILOT_MCP_ENABLED` | `true` |
| `COPILOT_MCP_READ_ONLY_MODE` | `false` |
| `COPILOT_AGENT_MCP_SERVER_TEMP` | `/home/runner/work/_temp/mcp-server` |

`GITHUB_COPILOT_MCP_JSON_FROM_INPUT` and `GITHUB_COPILOT_MCP_JSON` were both unset.

### 4. Was any user-provided MCP config loaded?

**Yes.** The MCP debug log confirms that user-provided MCP configuration was loaded from repository settings (not from an environment variable or `.github/mcp.json`):

```
User-provided MCPs are enabled, checking for environment variable
Adding default MCP servers to configuration
...
Starting MCP client for hataku-local-settings-probe with command: node ...
```

The repository-settings-defined server `hataku-local-settings-probe` was loaded and launched alongside the default servers.

### 5. Were only default MCP servers initialized?

**No.** In addition to the two default servers (`playwright` and `github-mcp-server`), the repository-settings-provided server `hataku-local-settings-probe` was also initialized:

| Server | Source | Status |
|---|---|---|
| `playwright` | Default | Connected successfully |
| `github-mcp-server` | Default | Connected successfully (36 tools) |
| `hataku-local-settings-probe` | Repository settings | Launch attempted; failed (connection closed) |

---

## Conclusion

Repository-settings MCP servers are loaded and launched by the Copilot cloud agent. The `hataku-local-settings-probe` server defined in repository settings was initialized, its stderr output (`HATAKU_SETTINGS_MCP_LOCAL_STDERR_20260706`) was captured in the MCP debug log, and its failure was recorded. No environment-variable-based MCP JSON override was in effect; the server was sourced exclusively from repository settings.
