# Phase 5: Systematic Failure Hunting Plan

Spark Compete workflow (full pipeline): **Spark → friction → Cursor repro → fix → verify → PR → proof packet**.  
Phase 5 stops at **documented candidates** — no fixes, no giant PRs, no speculative filing yet.

---

## 1. Objective

Phase 5 turns intentional friction into **reviewer-ready bug candidates** stored under `notes/` and `repros/`, not shipped fixes.

By the end of this phase you should have:

- A small backlog of **reproducible** failures with clear steps, expected vs actual, and safe evidence
- Enough triage signal to pick **one focused issue** for Phase 6 (Cursor repro → minimal fix → PR)
- Zero pressure to “win” with volume — quality and reproducibility beat count

**Output artifact:** dated note files (`notes/YYYY-MM-DD-<slug>.md`) and, when stable, a repro folder under `repros/<slug>/` with logs/screenshots redacted per `prompts/proof-safety.md`.

---

## 2. Daily loop (hunting-only)

Aligned to a lightweight Phase 10 rhythm, scoped to discovery only.

| Block | Time | Activity |
|-------|------|----------|
| **Morning hunt** | 45–60 min | One hunting zone (see §4). Run 2–3 concrete test ideas. Log friction immediately — don’t polish. |
| **Midday prioritize** | 15 min | Score new candidates with triage rubric (§5). Drop duplicates and “maybe someday.” Pick **at most one** to deepen today. |
| **Afternoon investigate** | 30–45 min | Turn the best candidate into a structured repro: copy `prompts/reproduction.md`, fill Environment/Steps/Expected/Actual, attach safe screenshots/logs. |

**End of day:** one sentence in the day’s top note — “best candidate today: `<slug>` because …”

---

## 3. Session checklist (60–90 min Spark session)

Use this every hunting session before opening Cursor for deep repro work.

- [ ] **Zone picked** — one of the four zones (§4); don’t mix zones in one session
- [ ] **Clean baseline** — known OS, Spark/Cursor versions, fresh or documented dirty state
- [ ] **Proof-safety first** — skim `prompts/proof-safety.md`; decide what you will *not* capture (keys, `.env`, PII)
- [ ] **2–3 test ideas executed** — from the zone list; note pass/fail/friction even on “soft” UX issues
- [ ] **Friction logged** — each hit gets a slug: `notes/YYYY-MM-DD-<slug>.md` (stub is fine mid-session)
- [ ] **One candidate deepened** — best item gets full repro sections (Environment, Steps, Expected, Actual)
- [ ] **Evidence bounded** — screenshot or log excerpt; redact; no full terminal dumps
- [ ] **Triage scored** — rubric filled for today’s top candidate (§5)
- [ ] **No fix scope** — resist editing upstream repos; hunting only
- [ ] **Handoff line** — “Ready for Phase 6?” only if repro steps work twice

---

## 4. Four hunting zones

File pattern: **`notes/YYYY-MM-DD-<slug>.md`** for discovery and triage.  
When repro is stable (same steps, same failure twice), add **`repros/<slug>/`** with `reproduction.md` (from template), `screenshots/`, `logs/` (redacted).

### Zone A — Installer / Setup Flow

**What to test:** First-run and recovery paths where setup can fail partially, lie about success, or leave the tool unusable until manual cleanup.

| # | Concrete test idea |
|---|-------------------|
| 1 | Invalid API key at install — submit, dismiss error, retry with valid key |
| 2 | Kill network mid-download/install; resume when back online |
| 3 | Interrupt install (force quit) at 50%; relaunch — partial state vs clean retry |
| 4 | Missing dependency (simulate: block binary, rename tool) — error clarity |
| 5 | Install without write permissions to config dir — failure message and exit code |
| 6 | Restart machine after failed install; open app — stale lock files or duplicate setup |
| 7 | Run installer twice — idempotent or destructive overwrite? |
| 8 | Offline install attempt — does UI explain offline vs bad key? |

**Capture:** installer UI states, CLI exit codes, config dir listing (redacted), setup log tail (last 30 lines).  
**File:** `notes/YYYY-MM-DD-setup-<issue>.md` → `repros/setup-<issue>/` when repeatable.

---

### Zone B — Context / Memory Drift

**What to test:** Instructions and repo context that contradict, go stale, or bleed across projects/sessions.

| # | Concrete test idea |
|---|-------------------|
| 1 | Give rule “never modify X”; later ask to modify X — does Spark obey prior rule? |
| 2 | Switch git repo mid-session — does memory still cite old paths/files? |
| 3 | Rename branch or delete file referenced in earlier turn — stale refs in suggestions |
| 4 | Two conflicting user rules in same session — which wins; is conflict surfaced? |
| 5 | New session same project — does old “remembered” fact persist incorrectly? |
| 6 | Open monorepo subfolder vs root — wrong package or wrong `package.json` context |
| 7 | Large pasted doc then “ignore above” — does later behavior still follow revoked text? |
| 8 | Cross-project: Project A facts mentioned while workspace is Project B |

**Capture:** transcript excerpts (trimmed), wrong file paths cited, before/after project switch screenshots.  
**File:** `notes/YYYY-MM-DD-context-<issue>.md` → `repros/context-<issue>/`.

---

### Zone C — CLI Error Recovery

**What to test:** Malformed input, bad auth, missing repo, timeouts, and giant/empty payloads — recovery without silent corruption.

| # | Concrete test idea |
|---|-------------------|
| 1 | Malformed flag combo (`--help` + invalid subcommand) — stderr vs exit code |
| 2 | Empty stdin / zero-byte file as input — hang vs clear error |
| 3 | Very large input (e.g. 5MB paste or file) — truncate, reject, or OOM message |
| 4 | Command with no git repo / wrong `cwd` — message names missing repo |
| 5 | Expired or wrong auth token — retry guidance without leaking token |
| 6 | Invalid config JSON/TOML — line number or key in error |
| 7 | Timeout (slow network or `sleep` in hooked command) — partial output handling |
| 8 | Interrupt running command (Ctrl+C) — child process cleanup, shell state |

**Capture:** command invoked, exit code, stderr (redacted), config snippet (no secrets).  
**File:** `notes/YYYY-MM-DD-cli-<issue>.md` → `repros/cli-<issue>/`.

---

### Zone D — Non-coder / UX

**What to test:** Onboarding, docs, and errors that block non-coders or send them in circles.

| # | Concrete test idea |
|---|-------------------|
| 1 | Follow README/setup doc literally on clean machine — first blocker step |
| 2 | Error message with no “what to do next” — can a non-coder recover in one try? |
| 3 | Contradictory docs (README vs in-app hint) for same action |
| 4 | Impossible onboarding step (requires tool not mentioned earlier) |
| 5 | Jargon-heavy error (stack trace only, no plain-language summary) |
| 6 | Success UI but background task failed — false confidence |
| 7 | Destructive action without confirm or with unclear confirm copy |
| 8 | Help/search for common task (“connect API”, “open project”) — dead ends |

**Capture:** screenshot of UI + exact error string; doc links; 1–2 sentence “user story” (“I’m new, I tried X”).  
**File:** `notes/YYYY-MM-DD-ux-<issue>.md` → `repros/ux-<issue>/`.

---

## 5. Triage rubric

Score each candidate **1–3** per dimension (3 = best for Compete). Drop or park if total **&lt; 8** or duplicate risk is 3.

| Dimension | 1 (weak) | 2 (ok) | 3 (strong) |
|-----------|----------|--------|------------|
| **Reproducible** | Happened once, vague steps | Mostly repeatable | Same steps, same failure twice |
| **Focused** | Systemic / needs huge refactor | Touches 1–2 files/areas | Single clear failure surface |
| **Reviewer-friendly** | Needs long context to understand | Clear with note + screenshot | Minimal repro, obvious expected vs actual |
| **Duplicate risk** | Known issue / already filed | Similar but different root cause | Novel or clearly distinct |

**Actions:**

- **≥ 10:** Promote to `repros/<slug>/`, candidate for Phase 6
- **8–9:** Keep in `notes/`, one more investigation session
- **&lt; 8 or duplicate risk = 3:** Drop or merge into existing note; don’t invest afternoon block

---

## 6. Templates usage

| Template | When to use in Phase 5 |
|----------|-------------------------|
| **`prompts/proof-safety.md`** | **Before** any screenshot, log paste, or sharing with Cursor — every session |
| **`prompts/reproduction.md`** | When a candidate is **repeatable** — copy into `repros/<slug>/reproduction.md` or into the bottom of the dated note |
| **`prompts/bug-analysis.md`** | **Not** during hunting — reserve for Phase 6 when Cursor analyzes root cause |
| **`prompts/fix-and-pr.md`** | **Not** during Phase 5 — only after a fix exists (Phase 6+) |

**Note stub** (start of every `notes/YYYY-MM-DD-<slug>.md`):

```markdown
# <slug>
Zone: A|B|C|D
Date:
Status: stub | investigating | repro-ready

## Friction (one line)

## Triage
Reproducible: /3 | Focused: /3 | Reviewer-friendly: /3 | Duplicate risk: /3 | Total:

## Next step
```

Fill **Environment / Steps / Expected / Actual** from `reproduction.md` when promoting to repro-ready.

---

## 7. First week schedule

Practical rotation — one primary zone per day; afternoon block only on the best candidate from that day.

| Day | Primary zone | Focus |
|-----|----------------|-------|
| **Day 1** | A — Installer / Setup | Tests 1, 3, 5 (invalid key, interrupt, permissions) |
| **Day 2** | C — CLI Error Recovery | Tests 1, 4, 6 (malformed cmd, no repo, bad config) |
| **Day 3** | B — Context / Memory Drift | Tests 2, 4, 5 (repo switch, conflicting rules, new session) |
| **Day 4** | D — Non-coder / UX | Tests 1, 2, 5 (README path, unhelpful errors, jargon) |
| **Day 5** | A — Installer (recovery) | Tests 2, 6, 7 (network drop, restart after fail, double install) |
| **Day 6** | C — CLI (edge input) | Tests 2, 3, 7 (empty/giant input, timeout) |
| **Day 7** | **Buffer / triage** | No new zone — score all notes, promote top 3 to `repros/`, write exit check (§9) |

If a day finds nothing solid, log **“clean on tests X,Y,Z”** in a short note — that’s still valid hunting output.

---

## 8. Do NOT (Phase 5)

- Open PRs or land fixes “while you’re here”
- File speculative bugs without repro steps
- Ship giant PRs or drive-by refactors in upstream repos
- Paste secrets, full `.env`, or unbounded logs — see proof-safety
- Chase every soft UX nit — triage ruthlessly
- Clone extra repos unless needed for a **specific** repro (prefer this compete workspace + target app)

---

## 9. Exit criteria for Phase 5

Phase 5 is complete when **all** are true:

1. **≥ 3** candidates in `repros/<slug>/` each with:
   - Filled `reproduction.md` (Environment, Steps, Expected, Actual)
   - Triage total **≥ 10**
   - Safe evidence (screenshot and/or log) passing proof-safety
2. Each repro verified **twice** on separate runs (same steps, same failure)
3. Prioritized list in `notes/` (or a single `notes/phase-5-exit-summary.md`) naming the **#1 pick for Phase 6** and why (focused, reviewer-friendly, low duplicate risk)
4. No open PRs required — handoff is documentation, not code

Then move to Phase 6: Spark friction → Cursor repro (bug-analysis) → smallest fix → verify → PR → proof packet.

---

## Quick reference

| Item | Location |
|------|----------|
| Daily friction | `notes/YYYY-MM-DD-<slug>.md` |
| Stable repro | `repros/<slug>/` + `reproduction.md` |
| Safety | `prompts/proof-safety.md` |
| Deep analysis (later) | `prompts/bug-analysis.md` |
| PR writeup (later) | `prompts/fix-and-pr.md` |
