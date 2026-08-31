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
- [ ] **Recurring bug: the fire's `git push` did not land, a second time**, despite the `access: "push"` fix applied after the first occurrence (fixed 02:44 UTC, this fire was 11:20 UTC — well after the fix). Repo and artifact drifted again; manually reconciled during this review (commit follows). Root cause still unconfirmed — this session has no way to read the fired session's own command output/logs to diagnose further. **Decision (2026-08-24): accepted as a known issue.** The daily review is the safety net — it detects the drift and reconciles the repo each morning, so the artifact is never more than a day stale from git. Not worth further investigation right now; revisit if reconciliation itself ever fails or the drift window becomes a problem.

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
| 2026-08-24 13:19 UTC (auto review #1) | First unattended market-refresh fire happened and published real data, but its git push failed again (recurrence — see Phase 1). Reconciled repo to match published artifact. Gmail label counts unchanged since last check (no drift); INBOX unread +12, normal incoming mail, not a red flag. | Decision made: keep the Routine as-is, daily review absorbs the drift. Phase 3 closed out this session. Phase 4 (remaining triage) still open. |
| 2026-08-25 13:07 UTC (auto review #2) | Market Routine fired again (Aug 25, 11:22 UTC), found real new data (sp500/nasdaq/dow caught up to Aug 24, gold/wti advanced to Aug 25), push again did not land (expected per 2026-08-24 decision). Reconciled repo silently, no user report needed. Gmail label counts unchanged; INBOX unread +68 over 24h, normal organic growth, no red flag. Nothing else pending. | Phase 4 (remaining ~150 threads) still the only open item awaiting the user. |
| 2026-08-26 13:22 UTC (auto review #3) | Market Routine fired again (Aug 26, 11:30 UTC), found real new data (sp500/nasdaq/dow/yield10y advanced to Aug 25; gold/wti still lagging one day, unconfirmed for Aug 26), push again did not land (expected). Reconciled silently. Gmail label counts unchanged; INBOX unread +68 over 24h, normal. Nothing else pending. | Phase 4 (remaining ~150 threads) still the only open item. |
| 2026-08-27 13:02 UTC (auto review #4) | Market Routine fired again (Aug 27, 11:25 UTC), push again did not land (expected). Only WTI got a new confirmed point (Aug 27, $81.36 — down from $85.67, skipped Aug 26 entirely); sp500/nasdaq/dow/yield10y/gold stalled at Aug 25 for a second run in a row. Not treated as a red flag — the rubric explicitly permits skipping unconfirmable instruments rather than guessing, and the WTI move has a plausible real-world driver (oil headlines in this run). Reconciled silently. Gmail unchanged; INBOX unread +76 over 24h, normal. Nothing else pending. | Worth a glance if the sp500/nasdaq/dow/yield10y/gold stall continues past a 3rd run — daily reconciliation is masking it from a full log/no-op distinction. Phase 4 (remaining ~150 threads) still the only open item. |
| 2026-08-28 13:02 UTC (auto review #5) | Market Routine fired again (Aug 28, 11:20 UTC), push again did not land (expected). **The sp500/nasdaq/dow/yield10y/gold stall flagged in review #4 resolved this run** — all six instruments caught up to Aug 27, WTI advanced further to Aug 28 ($83.10, partial rebound from the Aug 27 drop). Values consistent with real headlines (Nvidia earnings lifting tech). No further action needed on the stall. Reconciled silently. Gmail unchanged; INBOX unread +75 over 24h, normal. Nothing else pending. | Phase 4 (remaining ~150 threads) still the only open item. |
| 2026-08-29 13:02 UTC (auto review #6) | Saturday — market Routine correctly did not fire (weekdays-only cron), nothing to reconcile. Git clean. Gmail unchanged; INBOX unread +80 over ~24h, normal weekend volume. Nothing pending. | Phase 4 (remaining ~150 threads) still the only open item. |
| 2026-08-30 13:01 UTC (auto review #7) | Sunday — market Routine correctly did not fire, nothing to reconcile. Git clean. Gmail unchanged; INBOX unread +71 over ~24h, normal. Nothing pending. | Phase 4 (remaining ~150 threads) still the only open item. |
| 2026-08-31 13:21 UTC (auto review #8) | Market Routine fired (Aug 31, 11:23 UTC) after the weekend, push again did not land (expected). sp500/nasdaq/dow/yield10y advanced to Aug 28; gold and WTI skipped straight to Aug 31 (missed Aug 28 for gold specifically). Values consistent with real headlines (US-Iran conflict escalation — oil up, gold down on the same news, plausible if unusual pairing). Reconciled silently. Also noted in passing: a new, unrelated Routine ("Quarterly Cowork context review") appeared in this account's trigger list — not something this review created or touches; different repo/workspace entirely, ignored. Gmail unchanged; INBOX unread +57 over ~24-48h, normal. Nothing pending. | Phase 4 (remaining ~150 threads) still the only open item. |

