# Cloud Agent Token Permission Probe

| Endpoint category | Expected permission | Actual HTTP status | Matched declared permission model? |
|---|---|---:|---|
| 1. GET current repo metadata | metadata:read (baseline read access) | 403 | Inconclusive (request blocked before permission evaluation) |
| 2. GET issues | issues:read | 403 | Inconclusive (request blocked before permission evaluation) |
| 3. GET pull requests | pull-requests:read | 403 | Inconclusive (request blocked before permission evaluation) |
| 4. GET workflow runs | actions:read | 403 | Inconclusive (request blocked before permission evaluation) |
| 5. GET workflow logs | actions:read | 403 | Inconclusive (request blocked before permission evaluation) |
| 6. GET repository rulesets | administration/rules:read | 403 | Inconclusive (request blocked before permission evaluation) |
| 7. GET environments | actions:read (environments metadata) | 403 | Inconclusive (request blocked before permission evaluation) |
| 8. GET Actions variables metadata | actions:read | 403 | Inconclusive (request blocked before permission evaluation) |
| 9. GET Actions secrets metadata | actions:read (metadata only) | 403 | Inconclusive (request blocked before permission evaluation) |
| 10. GET code scanning alerts | security-events:read | 403 | Inconclusive (request blocked before permission evaluation) |
| 11. GET secret scanning alerts | secret-scanning:read | 403 | Inconclusive (request blocked before permission evaluation) |
| 12. POST create label `TOKEN_PERMISSION_SHOULD_NOT_CREATE_20260706` | issues:write (should be denied) | 403 | Yes (write did not proceed/succeed) |
