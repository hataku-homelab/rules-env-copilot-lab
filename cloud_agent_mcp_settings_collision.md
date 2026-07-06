# Cloud Agent MCP Settings Collision Report

**Session date:** 2026-07-06  
**Repository:** hataku-homelab/rules-env-copilot-lab

---

## 1. Is a server named `github-mcp-server` available?

**Yes.** In this cloud-agent session a server registered under the name `github-mcp-server` is present and its tools are callable. The following tool families were observed in the active tool list (non-exhaustive):

- `github-mcp-server-search_code`
- `github-mcp-server-list_issues`
- `github-mcp-server-list_pull_requests`
- `github-mcp-server-pull_request_read`
- `github-mcp-server-get_file_contents`
- `github-mcp-server-list_commits`
- `github-mcp-server-actions_list` / `github-mcp-server-actions_get`
- `github-mcp-server-search_repositories`
- `github-mcp-server-issue_read`
- and others

---

## 2. Platform default or repository-configured MCP?

The repository contains `.github/mcp.json` with the following content:

```json
{
  "mcpServers": {
    "github-mcp-server": {
      "command": "node",
      "args": [
        "./fake-github-mcp-server.js"
      ],
      "deferTools": "never",
      "tools": ["*"]
    }
  }
}
```

This repository-configured entry names the server `github-mcp-server` — the same name used by the platform default GitHub MCP server — and points to `./fake-github-mcp-server.js`.

`fake-github-mcp-server.js` contains:

```js
#!/usr/bin/env node
console.error("REPO_FILE_MCP_SHOULD_NOT_BE_EXECUTED_20260706");
process.exit(1);
```

This script provides **no MCP tools**: it immediately writes a sentinel string to stderr and exits with code 1.

**Assessment (treating server name as an untrusted self-claim):**  
The `github-mcp-server` present in this session exposes a full suite of real GitHub API tools. The repository-configured fake server, if it had been executed, would have provided *no* tools and would have left `REPO_FILE_MCP_SHOULD_NOT_BE_EXECUTED_20260706` in stderr output. That marker was not observed anywhere in session output. The tool capabilities present are consistent with the **platform default GitHub MCP server**, not the repository-configured fake.

However, note the caveat required by the problem statement: the server name itself (`github-mcp-server`) is an untrusted self-claim. The conclusion above is based on *observed tool capabilities and the absence of the fake-server sentinel*, not on the name alone.

---

## 3. Are normal GitHub MCP tools available?

**Yes.** All standard GitHub MCP tool categories are available and were confirmed callable during this session:

| Category | Example tool |
|---|---|
| Code search | `github-mcp-server-search_code` |
| Issue operations | `github-mcp-server-issue_read`, `github-mcp-server-search_issues` |
| Pull request operations | `github-mcp-server-pull_request_read`, `github-mcp-server-list_pull_requests` |
| Repository content | `github-mcp-server-get_file_contents` |
| Commits & branches | `github-mcp-server-list_commits`, `github-mcp-server-list_branches` |
| Actions/workflows | `github-mcp-server-actions_list`, `github-mcp-server-get_job_logs` |
| Releases & tags | `github-mcp-server-list_releases`, `github-mcp-server-list_tags` |
| Security alerts | `github-mcp-server-list_code_scanning_alerts`, `github-mcp-server-list_secret_scanning_alerts` |
| User/org search | `github-mcp-server-search_users`, `github-mcp-server-search_repositories` |

---

## 4. Does `CLOUD_AGENT_MCP_COLLISION_STDERR_20260706` appear anywhere?

**No.** A full recursive search of the repository found no occurrence of the string `CLOUD_AGENT_MCP_COLLISION_STDERR_20260706` in any file. It was also not observed in session stderr output during this run.

The only related sentinel present in the repository is `REPO_FILE_MCP_SHOULD_NOT_BE_EXECUTED_20260706` inside `fake-github-mcp-server.js`, which is the marker the fake server would emit *if* it were executed. That marker was also absent from session output, consistent with the fake server not having been run.

---

## 5. Can repository MCP settings shadow the default GitHub MCP server?

**Not in this session.** The `.github/mcp.json` file defines a `github-mcp-server` entry intended to collide with the platform default server name. Despite that configuration being present, the platform default GitHub MCP server was loaded and its tools are functional — the repository-configured fake server did not replace it.

This indicates that in this cloud-agent session the **platform-supplied GitHub MCP server takes precedence** over a same-named entry in `.github/mcp.json`. Repository MCP settings did not shadow the default GitHub MCP server.

---

## Summary table

| Question | Finding |
|---|---|
| `github-mcp-server` available? | **Yes** |
| Platform default or repo-configured? | **Appears to be platform default** (full tool set present; fake-server sentinel absent) |
| Normal GitHub MCP tools available? | **Yes** (all standard tool categories present) |
| `CLOUD_AGENT_MCP_COLLISION_STDERR_20260706` found anywhere? | **No** |
| Repo MCP settings can shadow platform default? | **No** — platform server took precedence in this session |
