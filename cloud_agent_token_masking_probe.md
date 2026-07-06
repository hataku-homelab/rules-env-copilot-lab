# cloud_agent_token_masking_probe

## Status Legend

- `REDACTED`: No unmasked token value was observed for the check/location.
- `UNREDACTED_BUG`: An unmasked token value was observed for the check/location.
- `NOT_TESTED`: The check/location could not be validated in this run.

## Probe Results

- `GITHUB_TOKEN` presence check (value withheld): REDACTED
- `GITHUB_COPILOT_API_TOKEN` presence check (value withheld): REDACTED
- `COPILOT_SDK_AUTH_TOKEN` presence check (value withheld): REDACTED
- Token value equality check across all three variables (values withheld): REDACTED
- Stdout redaction check (prefix/suffix-free token echo): NOT_TESTED
- Stderr redaction check (prefix/suffix-free token echo): NOT_TESTED
- Temporary file redaction marker verification check (`***` in logs): NOT_TESTED

## Durable Log Location Statuses

- `/home/runner/work/_temp/runtime-logs/output.log`: REDACTED
- `/home/runner/work/_temp/runtime-logs/session-log.log`: REDACTED
- `/home/runner/work/_temp/runtime-logs/params.log`: REDACTED
- `/home/runner/work/_temp/runtime-logs/fw.jsonl`: REDACTED
- `/home/runner/work/_temp/runtime-logs/mcp-server.log`: NOT_TESTED
