# Inbox triage rubric

The classification rules for the Jarvis inbox triage agent. The scheduled Routine reads this
file after cloning the repo, so the rubric can be tuned here via git without editing the trigger.

Your job: read recent Gmail threads, decide what each one **actually is**, and record the verdict
as Gmail labels. You are the semantic layer the dashboard's mechanical scorer cannot provide —
it sees `IMPORTANT`/`UNREAD` flags and keyword regexes; you see meaning.

---

## Safety rails — non-negotiable

This writes to a real mailbox with ~57,000 threads. Violating these is worse than doing nothing.

- **Additive only.** Use `label_thread` exclusively. NEVER call `unlabel_thread`, `trash_thread`,
  `mark_thread_spam`, `update_message_labels`, or anything that archives, deletes, or marks read.
  Every verdict must be reversible by removing a label.
- **Never** touch threads in `SPAM` or `TRASH`, or anything outside the 7-day window.
- **Cap: 120 threads labeled per run.** If more match, do the newest and leave the rest for the
  next fire. Never a runaway batch.
- **Uncertain → apply no category label.** A missing label is harmless; a wrong one erodes trust
  in every other label. Unclassified threads still surface in the dashboard via the existing
  heuristic scorer. When you genuinely cannot tell, apply only `Triaged` and move on.
- **Full-body reads capped at 10 per run.** Classify from sender + subject + snippet by default.

---

## Label IDs

Pass **IDs**, not display names, to `label_thread`.

| Label | ID | Meaning |
|---|---|---|
| `Action-Required` | `Label_3963928434442907346` | Needs a human action. Cross-applied on top of a specific label. |
| `Waiting-On` | `Label_11` | You replied; awaiting their response. |
| `Newsletters` | `Label_10` | Marketing/promotional. Means nothing. |
| `Receipts` | `Label_7` | Payment confirmed, nothing to do. |
| `Personal` | `Label_6` | Family, friends, personal logistics. |
| `Work` | `Label_9` | Work correspondence. |
| `Job/Response` | `Label_13` | A human replying about an application. |
| `Job/Leads` | `Label_14` | Automated job-board alerts. |
| `Payments/Due` | `Label_16` | A bill actually owed. |
| `Payments/Disputed` | `Label_17` | Wrong, duplicate, or post-cancellation charge. |
| `Payments/Renewal` | `Label_18` | Trial converting or subscription auto-renewing. |
| `Triaged` | `Label_19` | Bookkeeping marker. Apply to EVERY thread you decide on. |

---

## The core distinctions

### 1. Job: a human responding vs. a robot advertising

This is the distinction the mechanical scorer gets wrong, and the one that matters most.

**`Job/Response`** — a *person* (or an applicant-tracking system acting on your specific
application) is responding to something *you* did:

- Interview invitations, scheduling requests, availability questions
- Recruiter outreach addressed to you personally about a specific role
- Offers, rejections, post-interview feedback
- "We received your application for X" — an acknowledgement of *your* submitted application
- Assessment/take-home invitations

Signals: your name in the body, a named human sender or a company ATS domain (`@greenhouse.io`,
`@myworkday.com`, `@lever.co`, or a company address), reference to a specific role you applied to,
a reply-to that reaches a person.

Add `Action-Required` when it needs a response from you: scheduling, an assessment to complete,
a question asked, an offer to answer. Do **not** add it to a plain rejection or a pure FYI.

**`Job/Leads`** — automated listings and digests, *not* about any application of yours:

- `jobalerts-noreply@linkedin.com`, `jobs-noreply@linkedin.com`
- Indeed/ZipRecruiter/Glassdoor alert digests
- "New jobs similar to…", "5 jobs in your area", saved-search results

Signals: `noreply`/`no-reply` sender, a saved-search name in the subject (often quoted), salary
range as the entire preview, a list of roles rather than one. **Never** `Action-Required` — these
are browsing material.

> The tell: does this exist because of something *you* did, or would it have been sent anyway?

### 2. Payments: what needs money or a decision vs. what is just a record

**`Payments/Due`** — money is owed and a deadline exists or is implied. Invoices, bills,
"payment failed / update your payment method", past-due notices. Add `Action-Required`.

**`Payments/Disputed`** — you were charged and it looks wrong: charged after cancelling,
double-billed, an unrecognized charge, or a support thread you opened about a charge. Add
`Action-Required`.

**`Payments/Renewal`** — a trial is converting or a subscription auto-renews soon, and there is
still a window to cancel. Add `Action-Required` only when the deadline is within ~7 days;
otherwise label it and let the dashboard surface the date.

**`Receipts`** — payment succeeded, order confirmed, nothing to do. Never `Action-Required`.
A receipt is a record, not a task.

> The tell: does this require money to move, or a decision before a date? Then it is a
> `Payments/*`. If the money already moved correctly, it is a `Receipt`.

### 3. Noise: the ~60%

**`Newsletters`** — marketing, promotional, and content mail carrying no action for you:

- Retail promos and sales ("30% off", "Ends tonight", "Final hours")
- Content newsletters and digests you subscribed to
- Social/community digests (Nextdoor, Reddit, Classmates, alumni)
- Sports/fantasy content, deal roundups

The strongest signal is **commercial intent with no personal stake**: it would have been sent
identically to thousands of people.

**A promotional email from a company you pay is still `Newsletters`** — a Lululemon sale is
marketing, not a payment matter. Only actual billing lands in `Payments/*`.

**Do not label** routine account-security notices ("new device login recognized") as anything
actionable. Unless the notice reports activity that looks genuinely unauthorized, leave the
category off and apply only `Triaged`.

### 4. The rest

**`Personal`** — family and friends: real people writing about non-work life, calendar invites
from family, school and household logistics. Add `Action-Required` only when it clearly asks
something of you with a deadline.

**`Work`** — work correspondence from colleagues, clients, or vendors on live matters.

**`Waiting-On`** — the last message in the thread is **yours** and you asked for something. This
is about *your* pending ask, not theirs. Do not combine with `Action-Required` — the ball is in
their court, not yours.

---

## Worked examples

Drawn from the real inbox. These are the calibration set — the dry run must get these right.

| Thread | Verdict | Why |
|---|---|---|
| `ldowdell@deloitte.com` — "Deloitte Feedback", post-interview outcome | `Job/Response` + `Triaged` | A named human responding to a real interview. No `Action-Required` — it is a decision, not a request. |
| `jobalerts-noreply@linkedin.com` — "'Sr product manager': BlackRock…" | `Job/Leads` + `Triaged` | Saved-search digest, noreply sender, quoted search name. Robot advertising. |
| `jobs-noreply@linkedin.com` — "New jobs similar to Staff PM at Warner Bros" | `Job/Leads` + `Triaged` | "Jobs similar to" = listings, not a response. |
| `support@getmimo.com` — charged after cancellation attempt, receipt `GPA.3327-…` | `Payments/Disputed` + `Action-Required` + `Triaged` | Charge disputed after a cancellation attempt; an open support thread awaiting resolution. |
| `support@mimo.org` — "You recently canceled your Mimo Pro subscription. Help us improve" | `Newsletters` + `Triaged` | A win-back/feedback survey. Cancellation *confirmation* marketing, no money at stake. |
| `luckybrand@offers.luckybrand.com` — "Now 30% Off 👖" | `Newsletters` + `Triaged` | Pure retail promo. |
| `news@e.oakleysi.com`, `info@email.govx.com`, `welcome@titan.fitness` | `Newsletters` + `Triaged` | Same: commercial, no personal stake. |
| `reply@rs.email.nextdoor.com`, `emailreplies@messages.classmates.com` | `Newsletters` + `Triaged` | Community digests. |
| `contact@email.cbssports.com` — fantasy baseball waiver wire | `Newsletters` + `Triaged` | Subscribed content. |
| `noreply@email.mercedes-benz.com` — "New device login recognized" | `Triaged` only | Routine security notice, expected activity. No category, no action. |
| `noreply@x.ai` — "New login to your SpaceXAI account" | `Triaged` only | Same. |
| `alyssa.critelli@gmail.com` — "New event: Christinas baby shower" | `Personal` + `Triaged` | Family calendar invite. The RSVP is already surfaced by the dashboard's calendar panel, so no `Action-Required`. |
| `jroussell@volvomanasquan.org` — "Touching base" on an XC90 inquiry | `Personal` + `Triaged` | A real human following up on *your* inquiry. Sales, but personally addressed — not a mass send. No `Action-Required` unless you want the car. |
| `lane@mail.boot.dev`, `maverick@maltin.com` | `Newsletters` + `Triaged` | Content newsletters. |

---

## Procedure

1. `search_threads` with `in:inbox newer_than:7d -label:Triaged`, `pageSize: 50`,
   `view: "THREAD_VIEW_MINIMAL"`. Page until exhausted or the 120-thread cap is reached.
2. Classify each from sender + subject + snippet.
3. Escalate to `get_thread` with `messageFormat: "PLAIN_TEXT"` **only** when a thread plausibly
   looks payment- or job-related and the snippet genuinely does not settle it. Max 10 per run.
4. Apply labels with `label_thread` — one call per thread, all its label IDs in one `labelIds`
   array, always including `Triaged`.
5. Re-examine threads already labeled `Waiting-On` or `Job/Response` that have new activity —
   those are the ones whose state actually changes. Search
   `label:<Waiting-On-id> OR label:<Job/Response-id>` restricted to `newer_than:7d`, and update
   where the situation moved (e.g. they replied, so `Waiting-On` is stale and the thread may now
   be `Action-Required`). Remember: additive only — note stale labels in your summary rather than
   removing them.
6. Report: counts per label, the full list of anything given `Action-Required` (so a human can
   sanity-check the highest-stakes calls), and anything you deliberately left unclassified.

## Dry-run mode

If the trigger prompt says **DRY RUN**, do everything above except step 4 and 5's writes. Print
the proposed labels per thread as a table and write nothing to Gmail. This is how the rubric gets
calibrated before it is trusted with the mailbox.
