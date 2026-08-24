# Deployment checklist — Jarvis dashboard + inbox triage agent

Living checklist. Update it as items complete; don't delete history — strike through with
`~~text~~` so the review schedule can see what changed between check-ins.

Last updated: 2026-08-24 (see review log at bottom for cadence).

## Phase 1 — Dashboard (live)

- [x] `index.html` built: email/action-items/calendar/markets/news panels, live Gmail + Calendar via `mcp`, checklist persistence via `artifact`
- [x] Published to `https://claude.ai/code/artifact/323f876d-297b-4d92-a367-767509724bb0`
- [x] Committed to `claude/interactive-animated-dashboard-ih4ynk` (`b6aa09d`, `638a6cc`)
- [x] Market-refresh Routine created and fixed (push-access bug found + patched)
- [x] First unattended weekday auto-fire observed: 2026-08-24 11:20:43 UTC, found genuine new data (10Y yield 4.71%, gold $4,645.74) and published successfully.
- [ ] **Recurring bug: the fire's `git push` did not land, a second time**, despite the `access: "push"` fix applied after the first occurrence (fixed 02:44 UTC, this fire was 11:20 UTC — well after the fix). Repo and artifact drifted again; manually reconciled during this review (commit follows). Root cause still unconfirmed — this session has no way to read the fired session's own command output/logs to diagnose further. **Needs a decision**: try a more explicit push-verification instruction, or investigate a different way (e.g. fire once interactively and watch it directly).

## Phase 2 — Inbox triage: rubric & taxonomy

- [x] `agent/inbox-triage.md` written — taxonomy, label IDs, safety rails, 14 worked examples
- [x] Six new Gmail labels created: `Job/Response`, `Job/Leads`, `Payments/Due`, `Payments/Disputed`, `Payments/Renewal`, `Triaged`
- [x] Dry run completed over first 50 threads in the 7-day window — verdicts matched expectations (74% noise, 1 real payment dispute, 7 job alerts correctly separated from real responses)
- [ ] Note in rubric: `Job` and `Payments` parent labels were auto-created by Gmail's nested-label mechanic (`Job/Response` → parent `Job`). Harmless, but the rubric should say so explicitly so a future run doesn't try to "fix" it.

## Phase 3 — Inbox triage: dashboard integration

- [x] `index.html` updated: `CATEGORY_LABELS` map, category-aware `scoreThread`, badges, noise collapsing, new filter chips, payment/job action items, job-pipeline stat tile
- [x] Scoring logic unit-tested against 5 representative threads — ranks correctly
- [x] Committed and pushed (`9fcfd97`)
- [x] Republished to the live artifact — category badges and noise collapsing are now live

## Phase 4 — Inbox triage: applying to real mail

- [x] 18 of 50 dry-run threads labeled: 1 `Payments/Disputed`+`Action-Required` (Mimo), 2 `Personal`, 7 `Job/Leads`, 8 `Newsletters`
- [ ] **32 of the 50 were denied/interrupted mid-batch** — not an error, a deliberate stop. These still need a verdict applied (or a decision to skip them).
- [ ] **~150 threads in the 7-day window never reached** (only the first page of 50 was pulled)
- [ ] The single most important unverified case — the Deloitte human recruiter reply (`ldowdell@deloitte.com`) — is in that unreached remainder. This is the one thread that proves the agent's whole reason for existing (human response vs. robot alert). **Should be checked before trusting the rubric on more mail.**
- [ ] Blast-radius re-check after any further batch: `INBOX` unread should stay at ~53,240 (only moves with real mail arriving, never drops from agent activity)

## Phase 5 — Production automation

- [ ] **Blocked on this org's connector policy**: `create_trigger`'s `connectors` param is rejected here, so a fresh-session Routine can't reach Gmail. A twice-daily triage Routine must be created in the **claude.ai Routines UI** instead (connectors attach there), pointed at `agent/inbox-triage.md` in the repo.
- [ ] Decide: does production triage run unattended once trusted, or does every batch get a human pass first (given how Phase 4 actually went)? This affects whether the Routine should ever leave dry-run mode.

## Phase 4.5 — Not yet done

- [ ] 32 of the original 50 dry-run threads still need a verdict (interrupted mid-batch, see Phase 4)
- [ ] ~150 threads in the 7-day window never reached, including the Deloitte calibration case
- [ ] Blast-radius re-check after the next labeling batch

## Phase 6 — Documentation

- [ ] `README.md` updated with the label taxonomy, safety rails, and how to hand-tune the rubric via git
- [ ] This checklist itself committed to the repo so it isn't only visible in chat

---

## Review log

| When | What changed | Next gate |
|---|---|---|
| 2026-08-24 (this session) | Checklist created; state audited via `list_labels` + `git status` | See review schedule below |
| 2026-08-24 13:19 UTC (auto review #1) | First unattended market-refresh fire happened and published real data, but its git push failed again (recurrence — see Phase 1). Reconciled repo to match published artifact. Gmail label counts unchanged since last check (no drift); INBOX unread +12, normal incoming mail, not a red flag. | **User decision needed on the recurring push failure** before trusting this Routine unattended long-term. Phase 3/4 (dashboard commit, remaining triage) still open. |

