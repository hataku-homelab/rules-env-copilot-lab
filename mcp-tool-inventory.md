# MCP Tool Inventory

This repository can be inspected with approved GitHub/MCP tools using harmless read, list, and status operations. This document intentionally excludes secrets, credentials, and mutating operations.

## Actions visibility

Verified against the repository workflow `.github/workflows/marker-artifact.yml` and its successful `Marker Artifact Test` run on 2026-07-05.

| Capability | Approved tool(s) | Visible? | Notes |
| --- | --- | --- | --- |
| List workflows | `github-mcp-server-actions_list` (`list_workflows`) | Yes | Shows workflow IDs, names, paths, and state. |
| Get workflow details | `github-mcp-server-actions_get` (`get_workflow`) | Yes | Returns metadata for a specific workflow ID or file name. |
| List workflow runs | `github-mcp-server-actions_list` (`list_workflow_runs`) | Yes | Shows run IDs, branch, event, status, conclusion, timestamps, and related URLs. |
| Get workflow run details | `github-mcp-server-actions_get` (`get_workflow_run`) | Yes | Supported by the approved tool set for run-specific metadata. |
| List workflow jobs | `github-mcp-server-actions_list` (`list_workflow_jobs`) | Yes | Shows job IDs, status, conclusion, runner info, and step names. |
| Get job details | `github-mcp-server-actions_get` (`get_workflow_job`) | Yes | Supported for a specific job ID. |
| Get workflow-run logs URL | `github-mcp-server-actions_get` (`get_workflow_run_logs_url`) | Yes | Returns a temporary ZIP download URL for full run logs. |
| Get job log content | `github-mcp-server-get_job_logs` | Yes | Returned the harmless `H1_FAKE_ACTIONS_LOG_MARKER` line from the `marker` job. |
| List workflow run artifacts | `github-mcp-server-actions_list` (`list_workflow_run_artifacts`) | Yes | Showed artifact metadata for `h1-fake-actions-artifact`. |
| Download workflow artifact | `github-mcp-server-actions_get` (`download_workflow_run_artifact`) | Yes | Returned a temporary ZIP download URL for the artifact. |

### Actions conclusion

Approved GitHub/MCP tools can see:

- workflow definitions and metadata
- workflow runs and their status/conclusion
- run jobs and job steps
- job log content and full-run log download URLs
- artifact metadata and artifact download URLs

In other words, **Actions workflow runs, job logs, and artifacts are visible through approved tools in this repository**.

## Other approved GitHub/MCP read-only capabilities

These GitHub/MCP tools are also available for harmless inspection of this repository:

| Area | Approved tool(s) |
| --- | --- |
| Repository files | `github-mcp-server-get_file_contents` |
| Branches | `github-mcp-server-list_branches` |
| Commits | `github-mcp-server-list_commits`, `github-mcp-server-get_commit`, `github-mcp-server-search_commits` |
| Tags and releases | `github-mcp-server-list_tags`, `github-mcp-server-get_tag`, `github-mcp-server-list_releases`, `github-mcp-server-get_latest_release`, `github-mcp-server-get_release_by_tag` |
| Pull requests | `github-mcp-server-list_pull_requests`, `github-mcp-server-pull_request_read`, `github-mcp-server-search_pull_requests` |
| Issues | `github-mcp-server-list_issues`, `github-mcp-server-issue_read`, `github-mcp-server-search_issues` |
| Discussions | `github-mcp-server-list_discussions`, `github-mcp-server-get_discussion`, `github-mcp-server-get_discussion_comments`, `github-mcp-server-list_discussion_categories` |
| Labels | `github-mcp-server-list_label`, `github-mcp-server-get_label` |
| Collaborators | `github-mcp-server-list_repository_collaborators` |
| Code and repositories | `github-mcp-server-search_code`, `github-mcp-server-search_repositories`, `github-mcp-server-search_users` |
| Security alerts | `github-mcp-server-list_code_scanning_alerts`, `github-mcp-server-get_code_scanning_alert`, `github-mcp-server-list_secret_scanning_alerts`, `github-mcp-server-get_secret_scanning_alert` |

## Safety notes

- This inventory is limited to read/list/status-style operations.
- Do not record or publish secrets from workflow logs, artifacts, or repository contents.
- Temporary log and artifact download URLs should be treated as sensitive operational data even when the underlying content is harmless.
