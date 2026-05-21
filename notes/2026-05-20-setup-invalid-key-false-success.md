# setup-invalid-key-false-success

Zone: A — Block A (in progress)
Date: 2026-05-20
Status: stub

## Environment

| Field | Value |
|-------|-------|
| macOS version | |
| Architecture (`uname -m`) | |
| Node (`node -v`) | |
| Spark / Cursor version | |
| Install state | fresh / upgrade / recovery |
| Network | |

## Friction (one line)



## Steps (fill as you go)

1. Install state noted above
2. Opened setup / sign-in where API key is requested
3. Submitted invalid key: `sk-invalid-test-000` (do not paste real keys)
4. Observed error UI:
5. Submitted valid key (not logged here)
6. Checked settings/account after navigation:

## Expected vs actual

| Expected | Actual |
|----------|--------|
| Clear invalid-key error; no silent success | |
| After valid key, consistent authenticated state | |
| Retry works without restart | |
| No stack trace / raw HTTP / key echoed in logs | |

## Capture

- [ ] Screenshot after invalid submit → `screenshots/2026-05-20-setup-invalid-key-1.png`
- [ ] Screenshot post-valid or stuck state → `screenshots/2026-05-20-setup-invalid-key-2.png`
- [ ] Log excerpt (20–30 lines, redacted):

## Triage

| Dimension | Score /3 |
|-----------|----------|
| Reproducible | |
| Focused | |
| Reviewer-friendly | |
| Duplicate risk | |
| **Total** | |

## Next step

- If strong friction: re-run same steps once; if fails twice → promote to `repros/setup-invalid-key-false-success/`
- If clean after 20 min: note "clean on Block A" and move to Block B in runbook
