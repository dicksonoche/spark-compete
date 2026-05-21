# Day 1 — Installer / Setup hunting session

**Zone:** A — Installer / Setup Flow  
**Duration:** ~60–90 min in Spark (manual) + ~10 min post-session triage in this workspace  
**Phase:** 5 only — candidates and notes, **no fixes**, **no PRs**, **no repo clones**

You run every Spark action yourself. Cursor cannot access Spark for you. Log friction here as you go.

---

## 1. Pre-flight (~5 min)

Do this before touching install/setup flows in Spark.

### Record environment (paste into today’s notes or session log)

| Field | What to capture | Example (redact yours) |
|-------|-----------------|------------------------|
| macOS version | Apple menu → About This Mac | macOS 15.x |
| Architecture | `uname -m` in Terminal | arm64 |
| Node (if installer uses it) | `node -v` | v22.x |
| Spark / Cursor version | In-app About or Settings | Spark x.y / Cursor x.y |
| Install path | Fresh vs upgrade vs re-run | First install today |
| Network | Online / offline / throttled | Online, no VPN |
| Compete workspace | This repo path (generic OK) | `.../spark-compete` |

### Open compete context

- [ ] Spark Compete workflow in mind: **friction → note → (later) repro → fix**
- [ ] This repo open in Cursor: `spark-compete`
- [ ] Skim [`prompts/proof-safety.md`](../prompts/proof-safety.md) — decide what you will **not** capture (API keys, `.env`, tokens, full home paths)

### Proof-safety reminder (non-negotiable)

- Use **invalid** or **revoked** keys only for Block A — never paste a real key into notes or screenshots.
- Blur or crop screenshots that show key fields, emails, or machine-specific paths.
- Log excerpts: **last 20–30 lines** only; redact hostnames and usernames.
- Prefer describing secrets (“invalid key `sk-…REDACTED` rejected”) over raw paste.

### Folders ready (create if missing)

From repo root `spark-compete/`:

```text
notes/                          # dated candidate files
repros/                         # only when repro works twice (usually not Day 1)
screenshots/                    # session captures (optional subfolder per slug)
```

Quick check:

```bash
mkdir -p notes repros screenshots
```

### Session state line (one sentence in a scratch note)

> Day 1 Zone A — targeting 1–3 installer/setup candidates; blocks A–D; no fix scope.

---

## 2. Session goal

**Objective:** Stress first-run and recovery paths where setup can fail partially, claim success incorrectly, or leave Spark unusable until manual cleanup.

| Target | Detail |
|--------|--------|
| Candidates | **1–3** filed or stubbed — quality over volume |
| Primary tests (plan Day 1) | Invalid API key (#1), interrupt install (#3), permissions (#5) |
| Stretch | Partial dependency (#4), network drop (#2) — Block D |
| Out of scope | Fixes, upstream PRs, cloning extra repos, Phase 6 analysis |

**Success today:** At least one `notes/YYYY-MM-DD-setup-<slug>.md` with real friction **or** an explicit clean-run note (see §7).

---

## 3. Time-boxed blocks (~60–90 min total)

| Block | Topic | Time | Plan test # |
|-------|--------|------|-------------|
| **A** | Invalid API key / false success | 15–20 min | 1 |
| **B** | Interrupted install / restart after failure | 15–20 min | 3, 6 |
| **C** | Partial setup / missing dependency / permissions | 15–20 min | 4, 5 |
| **D** | Network drop or cancel mid-step | 10–15 min | 2 (if feasible) |
| **Buffer** | File stubs, one screenshot polish | 5–10 min | — |

**Rule:** If a block yields nothing in ~15 min, log `clean on block X` and move on — do not force bugs.

---

## Block A — Invalid API key / false success (15–20 min)

### Concrete steps in Spark

1. Start from a **clean or documented** install state (fresh VM/user, or note “upgrade from yesterday”).
2. Open Spark/Cursor **setup or sign-in** where an API key (or provider token) is requested.
3. Enter an **obviously invalid** key (e.g. `sk-invalid-test-000` or provider’s documented bad format). Submit.
4. Observe error UI: message, dismiss, retry affordance.
5. **Without** fully quitting, enter a **valid** key (your real key — do not screenshot it). Submit.
6. Optional: repeat — invalid → dismiss → invalid again — watch for cached “success” or stuck state.
7. Navigate away and back to settings/account — does UI still show connected?

### Watch for (expected vs actual)

| Expected (good) | Actual (file if wrong) |
|-----------------|-------------------------|
| Clear error: invalid key, no silent success | Toast says “connected” or setup completes with bad key |
| After valid key, authenticated state is consistent | Settings still show error or “not configured” |
| Retry after error works without restart | Must kill app or delete config to recover |
| Error does not leak validation internals | Stack trace, raw HTTP body, or key echoed in log |

### Capture checklist

- [ ] Screenshot: error state **after** invalid submit (crop key field)
- [ ] Screenshot: post-valid state OR stuck state
- [ ] Log excerpt: installer/setup log tail, 20–30 lines, redacted → `screenshots/` or note body
- [ ] Record exact button labels and screen order in note Steps

### When to stop and file

**File** `notes/YYYY-MM-DD-setup-invalid-key-false-success.md` (adjust slug) if:

- False success, ambiguous error, or recovery requires manual cleanup, **or**
- Same failure happens twice with same steps → add `repros/setup-invalid-key-false-success/` later

**Stop block** after one solid candidate or 20 min — do not exhaust all sub-steps.

---

## Block B — Interrupted install / restart after failure (15–20 min)

### Concrete steps in Spark

1. Begin install or first-run setup (download/update if applicable). Note progress indicator.
2. At **~50%** (or mid-download), **force quit** Spark/Cursor (Activity Monitor → Force Quit, or `kill -9` on installer PID if CLI-driven — note which you used).
3. Relaunch app immediately — what wizard/state appears?
4. If offered “Resume” / “Retry” / “Start over”, try **Resume** first; record outcome.
5. Quit normally, relaunch again — idempotent or duplicate setup?
6. **Restart macOS** (or skip if timeboxed — note “skipped restart” in env), open app — stale lock, duplicate wizard, or broken half-config?
7. If Block A left a failed key attempt, combine: failed install → force quit → relaunch (recovery interaction).

### Watch for

| Expected | Actual (file) |
|----------|----------------|
| Detect partial install; offer clean retry | Crash on launch, blank window, infinite spinner |
| Resume continues or cleanly restarts | Corrupt config, two accounts, duplicate entries |
| No orphaned lock files blocking second run | “Already installing” with no way to clear |
| After OS restart, stable launch | Must delete `~/...` paths manually (describe generically in note) |

### Capture checklist

- [ ] Screenshot: mid-install before kill
- [ ] Screenshot: first screen after relaunch
- [ ] Screenshot: after Resume vs after second relaunch (if different)
- [ ] Config dir listing: **names only**, no secrets (`ls` redacted output in note)
- [ ] Log tail after relaunch

### When to stop and file

**File** `notes/YYYY-MM-DD-setup-interrupt-relaunch.md` if partial state, crash, or unclear recovery.

**Stop block** after reproducible interrupt path once or 20 min.

---

## Block C — Partial setup / missing dependency / permissions (15–20 min)

Pick **one** path per session if time is tight; permissions is Day 1 priority per plan.

### Path C1 — Permissions (plan test #5)

1. Identify Spark/Cursor config directory (generic: application support folder for the product).
2. **Simulate** no write access: rename config dir temporarily **or** `chmod -w` on the dir (note exact action in Environment — revert after session).
3. Run setup / first-run / save settings that writes config.
4. Observe error: permission denied, exit code if CLI, in-app message.

### Path C2 — Missing dependency (plan test #4)

1. If install uses a bundled CLI, note its path from docs or Activity Monitor.
2. **Simulate** missing binary: rename binary (backup first) or block execution — only if you can restore safely.
3. Trigger step that needs that binary (install complete, extension load, etc.).
4. Read error: does it name the missing tool and remediation?

### Path C3 — Partial setup (general)

1. Complete setup until **one step before finish**, then force quit (lighter than Block B).
2. Relaunch — is app usable with half-filled wizard? Can you skip remaining steps incorrectly?

### Watch for

| Expected | Actual (file) |
|----------|----------------|
| Actionable error (“cannot write to …”, “install …”) | Silent failure or generic “something went wrong” |
| Non-zero exit for CLI installer | Exit 0 with broken state |
| User can fix permissions and retry without manual doc hunt | Requires forum/deleting unknown files |
| Missing dep names the tool | Opaque crash |

### Capture checklist

- [ ] Screenshot: permission or missing-dep error
- [ ] CLI exit code if applicable: `echo $?`
- [ ] Before/after `ls` of config dir (names only)
- [ ] One-line “what a non-coder would try next” in Friction section

### When to stop and file

**File** e.g. `notes/YYYY-MM-DD-setup-permissions-opaque-error.md` or `...-missing-dep-no-guidance.md`.

**Revert** chmod/rename immediately after capture.

---

## Block D — Network drop or cancel mid-step (10–15 min, if feasible)

Skip if no download/update step in your install path — log `Block D skipped — no network install step`.

### Concrete steps

1. Start install or update that shows **download progress**.
2. Disable network: Wi‑Fi off, or `sudo dscacheutil -flushcache` not needed — simple airplane mode / disconnect Ethernet.
3. Wait for UI reaction (timeout, error, spinner).
4. Re-enable network — does **Resume** work?
5. Alternate: click **Cancel** mid-download if UI offers it — then retry install.

### Watch for

| Expected | Actual (file) |
|----------|----------------|
| Distinguish offline vs bad auth vs server error | Same message for offline and invalid key |
| Resume download without full reinstall | Stuck progress bar, corrupt cache |
| Cancel leaves system clean for retry | Broken half-installed bundle |

### Capture checklist

- [ ] Screenshot: offline/error state
- [ ] Screenshot: after reconnect + resume
- [ ] Log lines mentioning network (redacted)

### When to stop and file

**File** `notes/YYYY-MM-DD-setup-network-resume-fail.md` if resume/cancel path is broken or misleading.

---

## 4. Filing convention

### Note filename

```text
notes/YYYY-MM-DD-<slug>.md
```

**Day 1 slugs (examples):**

- `setup-invalid-key-false-success`
- `setup-interrupt-relaunch`
- `setup-permissions-opaque-error`
- `setup-missing-dep-no-guidance`
- `setup-network-resume-fail`
- `setup-clean-run-day1` — if all blocks clean

### Note stub (start immediately when friction appears)

```markdown
# <slug>
Zone: A
Date: YYYY-MM-DD
Status: stub | investigating | repro-ready

## Friction (one line)


## Triage
Reproducible: /3 | Focused: /3 | Reviewer-friendly: /3 | Duplicate risk: /3 | Total:

## Next step
```

When repeatable (**same failure twice**), promote to:

```text
repros/<slug>/
  reproduction.md    # copy from prompts/reproduction.md
  screenshots/
  logs/              # redacted excerpts only
```

Full reproduction sections: [`prompts/reproduction.md`](../prompts/reproduction.md).

### Screenshot rules

| Rule | Detail |
|------|--------|
| Location | `screenshots/YYYY-MM-DD-<slug>-<n>.png` or under `repros/<slug>/screenshots/` |
| Content | UI state, progress bars, error dialogs — crop secrets |
| Count | 1–2 per candidate usually enough |

### Log excerpt rules

- Last **20–30 lines** of setup/install log only
- Replace home dir with `~/...` or `$HOME/...`
- No API keys, tokens, cookies, emails in paste
- Stack traces: failing frame + 2–3 lines context

---

## 5. Post-session triage (~10 min)

Do this in `spark-compete` after Spark session ends.

### Score each candidate (1–3 per dimension, 3 = best)

| Dimension | 1 (weak) | 2 (ok) | 3 (strong) |
|-----------|----------|--------|------------|
| **Reproducible** | Once, vague | Mostly repeatable | Same steps, same failure **twice** |
| **Focused** | Huge/systemic | 1–2 areas | Single clear failure surface |
| **Reviewer-friendly** | Long context needed | Note + screenshot enough | Minimal repro, obvious expected vs actual |
| **Duplicate risk** | Known/filed | Similar but different | Novel / distinct |

**Total:** sum of four scores (max 12).

| Total | Action |
|-------|--------|
| **≥ 10** | Promote toward `repros/<slug>/`, top pick for Phase 6 |
| **8–9** | Keep in `notes/`, one more hunt session |
| **< 8** or duplicate risk **3** | Drop or merge; don’t invest more today |

### Score table (fill in)

| Slug | Repro | Focus | Reviewer | Dup risk | Total | Phase 6? |
|------|-------|-------|----------|----------|-------|----------|
| | /3 | /3 | /3 | /3 | | |
| | /3 | /3 | /3 | /3 | | |
| | /3 | /3 | /3 | /3 | | |

### Pick top 1 for Phase 6 (one sentence)

Add to the best note or here:

> **Best candidate today:** `<slug>` because ___

Only mark **Ready for Phase 6?** if repro steps failed **twice** the same way.

---

## 6. Exit checklist

Session is **done** when all apply:

- [ ] Pre-flight env recorded
- [ ] Blocks A–C attempted (D attempted or explicitly skipped with reason)
- [ ] **At least one** of:
  - [ ] ≥ 1 candidate filed under `notes/YYYY-MM-DD-setup-*.md` with friction + triage stub, **or**
  - [ ] `notes/YYYY-MM-DD-setup-clean-run-day1.md` listing blocks/tests run and “clean on …”
- [ ] Proof-safety pass on every screenshot/log attached
- [ ] Post-session triage table filled; top 1 named for Phase 6 (or “none ≥ 8”)
- [ ] No fixes, no PRs, no extra repo clones

**Handoff:** Phase 6 only when top candidate is repro-ready (twice verified).

---

## 7. Clean-run note template (if no bugs)

If all blocks behave correctly, create one short note:

```markdown
# setup-clean-run-day1
Zone: A
Date: YYYY-MM-DD
Status: investigating

## Friction (one line)
No filing-grade defects on Day 1 installer tests.

## Blocks run
- A invalid key: clean / friction: ___
- B interrupt: clean / friction: ___
- C permissions/dep: clean / friction: ___
- D network: skipped | clean / friction: ___

## Triage
N/A — no candidate promoted.

## Next step
Retry Day 5 installer recovery tests or deepen Block B with OS restart.
```

---

## 8. Copy-paste reproduction template

Use when a candidate is **repro-ready** (failure twice). Full template: [`prompts/reproduction.md`](../prompts/reproduction.md).

```markdown
# Bug reproduction

## Environment

- OS / runtime:
- Spark / Cursor version:
- Install state: (fresh | upgrade | recovery)
- Relevant config or flags:

## Steps

1.
2.
3.

## Expected

-

## Actual

-
```

---

## Quick reference

| Item | Location |
|------|----------|
| Hunting plan | [`notes/phase-5-hunting-plan.md`](phase-5-hunting-plan.md) |
| Reproduction | [`prompts/reproduction.md`](../prompts/reproduction.md) |
| Proof safety | [`prompts/proof-safety.md`](../prompts/proof-safety.md) |
| Session output | `notes/YYYY-MM-DD-setup-<issue>.md` |
| Stable repro | `repros/setup-<issue>/` |
