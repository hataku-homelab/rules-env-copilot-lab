# Cloud Agent MCP Settings Collision Retest

**Date:** 2026-07-06  
**Branch:** `copilot/create-cloud-agent-mcp-settings-collision-retest-m`  
**Repository settings MCP server under test:** `github-mcp-server` (local, `node -e "console.error('CLOUD_AGENT_MCP_COLLISION_STDERR_20260706'); process.exit(1);"`)

---

## Summary

When a repository settings MCP server is named `github-mcp-server` — identical to the platform default — the repository-settings-defined local command is **not launched**. The platform default remote `github-mcp-server` runs normally, all 36 GitHub MCP tools remain available, and the collision marker `CLOUD_AGENT_MCP_COLLISION_STDERR_20260706` does not appear anywhere in runtime output.

---

## Evidence

### Primary log: `/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log`

```
MCP Client Server starting on platform: linux, arch: x64, node: v24.17.0
Loaded 5 secrets
Created MCP Registry instance
MCPServer: useNewOutOfProcMcp=true
MCPServer: readOnlyMode=false
User-provided MCPs are enabled, checking for environment variable
Adding default MCP servers to configuration
Using default remote GitHub MCP server configuration
Starting remote MCP client for github-mcp-server with url: https://api.individual.githubcopilot.com/mcp/readonly
Creating MCP client for github-mcp-server...
Connecting MCP client for github-mcp-server...
...
MCP client for github-mcp-server connected, took 603ms
MCP server github-mcp-server provided instructions: The GitHub MCP Server provides tools to interact with GitHub platform.
Started MCP client for remote server github-mcp-server
...
Updated session log for github-mcp-server with 36 tools
Retrieved 36 tools from github-mcp-server
```

### MCP server temp directory: `/home/runner/work/_temp/mcp-server/`

The directory is **empty**. No local MCP server configuration files were written there. This is the directory pointed to by `COPILOT_AGENT_MCP_SERVER_TEMP` and is where user-provided local server configs would be deposited before launch.

---

## Findings per Question

### 1. Was the repository-settings-defined `github-mcp-server` local command launched?

**No.** No local `node` process running the collision command was started. The `COPILOT_AGENT_MCP_SERVER_TEMP` directory is empty, meaning no local server config was written for the repository settings entry. The MCP debug log contains no trace of a local process being spawned for `github-mcp-server`.

### 2. Did `CLOUD_AGENT_MCP_COLLISION_STDERR_20260706` appear in any log, stderr, or runtime file?

**No.** The marker string does not appear in:
- `/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log`
- `/home/runner/work/_temp/runtime-logs/output.log` (except as quoted text in the echoed problem statement)
- `/home/runner/work/_temp/runtime-logs/session-log.log`
- The environment
- Any file under `/home/runner/work/_temp/`

The command that would have emitted the marker was never executed.

### 3. Was the platform default remote `github-mcp-server` launched?

**Yes.** The platform default remote GitHub MCP server connected successfully:

```
Starting remote MCP client for github-mcp-server with url: https://api.individual.githubcopilot.com/mcp/readonly
...
MCP client for github-mcp-server connected, took 603ms
Started MCP client for remote server github-mcp-server
```

### 4. Are the platform GitHub MCP tools available and callable?

**Yes.** 36 tools were retrieved from the platform `github-mcp-server` and were active throughout this session. All standard GitHub MCP tools (e.g. `github-mcp-server-issue_read`, `github-mcp-server-search_code`, etc.) functioned normally.

### 5. Was there any log line indicating duplicate-name handling, skipping, overriding, or precedence?

**No explicit message.** The log transitions directly from:

```
User-provided MCPs are enabled, checking for environment variable
Adding default MCP servers to configuration
Using default remote GitHub MCP server configuration
```

There is no "skipped duplicate", "collision detected", "overriding user config", or similar line. The collision resolution is silent — the repository settings server is dropped without any warning or notice in the observable logs.

### 6. Did the custom same-name MCP server shadow the platform GitHub MCP server?

**No.** The platform default remote `github-mcp-server` ran unaffected. The repository settings `github-mcp-server` was silently ignored; it did not replace, override, or shadow the platform default.

### 7. Does this differ from the non-colliding `hataku-local-settings-probe` behavior?

**Yes, significantly.** The prior `hataku-local-settings-probe` test demonstrated that repository settings MCP servers with names that do not collide with platform defaults are launched: the local command was executed and its stderr marker appeared in the debug log.

In this run, the name collision with `github-mcp-server` caused the repository settings entry to be silently dropped. The platform default took exclusive ownership of that name. No stderr output was produced, no local process was started, and no error was logged.

---

## Mechanism (inferred)

The MCP initialization sequence appears to be:

1. Check for user-provided MCP servers (via environment variable mechanism; the `COPILOT_AGENT_MCP_SERVER_TEMP` dir is the staging area).
2. Add platform default servers (`github-mcp-server` remote, `playwright` remote, etc.).
3. When a user-provided server name matches a platform default name, the platform default wins — the user-provided entry is silently discarded, not written to the staging directory, and not launched.

The collision resolution happens before or during configuration assembly and is not reflected in any logged message. This is distinct from the non-colliding case where the user-provided server config is written to the staging directory and launched.

---

## Conclusion

A repository settings MCP server named `github-mcp-server` does **not** override, shadow, or replace the platform default `github-mcp-server`. The platform default takes unconditional precedence. The custom local command is never executed, its stderr is never produced, and no log message announces this suppression. The platform GitHub MCP tools remain fully available and unaffected.
