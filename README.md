# rules-env-copilot-lab

## Issue #4 diagnostic note

Diagnostic result for "Deployment tool boundary test" (`#4`):

| Area | Visibility through available GitHub tools in this environment | Evidence |
| --- | --- | --- |
| Repository deployments | Not directly visible | No deployment-specific GitHub MCP tool is available in this environment's toolset. |
| Repository environments | Not directly visible | No environment listing/read GitHub MCP tool is available in this environment's toolset. |
| Environment protection settings | Not directly visible | Protection settings depend on environments and no environment/protection GitHub MCP tool is available here. |

Related repository data that is visible with available tools:
- Branches are visible (via `github-mcp-server-list_branches`).
- Workflows are visible (via `github-mcp-server-actions_list` with `method: list_workflows`).
