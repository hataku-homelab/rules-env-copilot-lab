# Cloud Agent MCP `playwright` Collision Report

## Scope and evidence
Primary evidence was collected from:
- `/home/runner/work/_temp/cca-mcp-debug-logs/mcp-server.log`
- `/home/runner/work/_temp/runtime-logs/output.log`
- `/home/runner/work/_temp/runtime-logs/params.log`
- Runtime environment (`env`)

## Findings
1. **Was a repository-settings-defined `playwright` local command launched?**
   - **No evidence of launch.**
   - `mcp-server.log` shows only remote startup for `playwright`: `Starting remote MCP client for playwright with url: http://localhost:3100/mcp` and `Started MCP client for remote server playwright` (lines 9, 16).
   - There is no `Starting local MCP client for playwright` line and no `[mcp server playwright stderr] ...` emission.

2. **Did `CLOUD_AGENT_PLAYWRIGHT_COLLISION_STDERR_20260706` appear in MCP debug logs, process stderr, tool output, environment, or runtime files?**
   - **Not as process stderr from a launched custom server.**
   - It appears as prompt/tool text content:
     - `output.log` line 13 (problem statement text)
     - `mcp-server.log` line 49 (tool invocation result containing PR body text)
   - It does **not** appear as an MCP subprocess stderr marker (no `[mcp server ... stderr] CLOUD_AGENT_PLAYWRIGHT_COLLISION_STDERR_20260706`).
   - It does **not** appear in environment output.

3. **Was the platform default `playwright` MCP server launched?**
   - **Yes.**
   - `mcp-server.log` shows remote `playwright` initialization and connection (lines 9, 15-16), plus tool discovery: `Retrieved 21 tools from playwright` (line 32).

4. **Did platform Playwright MCP tools remain available and callable?**
   - **Yes.**
   - `mcp-server.log` shows 21 tools retrieved (line 32).
   - Tool invocations were accepted and executed (`playwright/browser_tabs`, `playwright/browser_close`; lines 42-48, 50-67).
   - One `browser_tabs` result reported browser lock contention (`Browser is already in use`), which still confirms the platform Playwright tool endpoint was callable.

5. **Did the platform GitHub MCP server remain available?**
   - **Yes.**
   - `mcp-server.log` shows default remote GitHub MCP startup: `https://api.individual.githubcopilot.com/mcp/readonly` (line 12), connected (line 17), and `Retrieved 36 tools from github-mcp-server` (line 30).
   - A live GitHub MCP call (`list_pull_requests`) succeeded (lines 35-41, 49; also `output.log` line 166 success).

6. **Any log line indicating duplicate-name handling, skipping, overriding, replacement, merge, or precedence?**
   - **No explicit duplicate-handling log line found.**
   - Startup proceeds directly with default server wiring and remote connections.

7. **Did the custom same-name MCP server shadow the platform Playwright MCP server?**
   - **No.**
   - Observed behavior is consistent with platform default `playwright` remaining active; no custom-local-launch stderr evidence was observed.

8. **Does this match the previous `github-mcp-server` collision behavior?**
   - **Yes.**
   - Same pattern: platform default server remains active, same-name custom server shows no launch evidence, and no explicit duplicate-collision log messaging is emitted.
