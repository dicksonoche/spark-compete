# Proof safety checklist

Before sharing logs, screenshots, or excerpts with Cursor or in a PR:

- [ ] No API keys, tokens, passwords, or session cookies
- [ ] No `.env` values, credentials files, or connection strings
- [ ] No machine-specific local paths (use `...` or generic labels)
- [ ] No internal hostnames, IPs, or employee/customer PII unless required and approved
- [ ] Excerpts are bounded (relevant lines only, not full dumps)
- [ ] Stack traces trimmed to the failing frame and immediate context

When in doubt, redact and describe instead of pasting raw secrets.
