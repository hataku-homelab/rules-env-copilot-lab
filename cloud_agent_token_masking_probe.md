# cloud_agent_token_masking_probe

## Probe Results

- `GITHUB_TOKEN` set: REDACTED
- `GITHUB_COPILOT_API_TOKEN` set: REDACTED
- `COPILOT_SDK_AUTH_TOKEN` set: REDACTED
- Token identity check (`GITHUB_TOKEN == GITHUB_COPILOT_API_TOKEN == COPILOT_SDK_AUTH_TOKEN`): REDACTED
- Stdout redaction check (prefix/suffix-free token echo): NOT_TESTED
- Stderr redaction check (prefix/suffix-free token echo): NOT_TESTED
- Temporary-file + literal `***` grep confirmation check: NOT_TESTED

## Durable Log Location Statuses

- `/home/runner/work/_temp/runtime-logs/output.log`: REDACTED
- `/home/runner/work/_temp/runtime-logs/session-log.log`: REDACTED
- `/home/runner/work/_temp/runtime-logs/params.log`: REDACTED
- `/home/runner/work/_temp/runtime-logs/fw.jsonl`: REDACTED
- `/home/runner/work/_temp/runtime-logs/mcp-server.log`: NOT_TESTED
