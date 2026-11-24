# The meeting-intelligence sales pipeline

An internal sales platform ("Sales Command" here) that turns **calendar bookings into
revenue**: Calendly → video call → AI analysis → lead score → drafted follow-up → human (or
WhatsApp agent) send → report. It sits on the data project, schema `sales` (originally built
on its own small project and migrated — see §6), with ~85 edge functions and ~30 shared
modules. This is the most AI-agent-dense codebase in the estate: almost every function either
produces or consumes model output, and the shared module `lwl-knowledge.ts` (here:
`knowledge.ts`) is the single place the organisation's facts, voice rules and scoring rubric
live, so every agent quotes the same truth.

---

## 1. The spine

```
Calendly booking ──webhook──▶ sales.meetings (+ participants)
        │                              │
        │                      score-lead-prep (pre-meeting score)
        ▼
   Zoom call happens ──webhook/sync──▶ transcript attached
        │
        ▼
  analyze-meeting ── AI reads transcript, writes structured notes + interest level
        │
        ▼
     score-lead ── segment-aware rubric → lead_score, lead_temperature, recommended_goal
        │
        ├─▶ dispatch-high-score-webhook (hot leads escalate immediately)
        ├─▶ extract-commitments ── what was promised, by whom, by when
        ▼
  plan-followups ── decides what touch is due
        │
        ▼
  compose-followup / compose-reply / compose-outreach ── drafts the message
        │
        ├─▶ send-draft (human approves) ──▶ send-gmail (OAuth'd mailbox)
        ├─▶ wa-owner-agent / wa-claim-* (WhatsApp agent claims and sends)
        ▼
  gmail-thread-sync / gmail-followup-sync ── watches for replies
        │
        ▼
  classify-inbound ── intent bucket (see §4) ──▶ loops back into plan-followups
        │
        ▼
  weekly-review / send-meeting-report / member-sheet-report / send-attribution-report
```

Supporting sweepers run on cron: `no-show-recovery`, `redecide-attendance`,
`meetings-finalize-sweeper`, `notification-sweeper`, `prep-upcoming`,
`welcome-bot-dispatch`/`welcome-bot-status`/`welcome-greeter-watch`,
`agent-sweeper`, `master-agent-daily`.

---

## 2. Identity and scoring (the knowledge module)

`supabase/functions/_shared/knowledge.ts` is imported by nearly every AI-facing function. It
exports constants every agent must agree on:

```ts
buildSystemPreambleWithKnowledge()   // the org's identity + programmes + voice rules,
                                      // assembled fresh each call so every agent is consistent
ESCALATION_TRIGGERS
INTEREST_LEVELS                      // enum the model's output is validated against
LEAD_SCORING_RUBRIC
PROGRAM_NAMES                        // enum: the only programme names a model may output
```

`analyze-meeting/index.ts` — "Knows the org, programmes, voice rules, and lead-scoring rubric
… reads meeting + transcript, returns structured intelligence and lead grading. **Never
touches lifecycle status**" (its own header comment). Every model output is validated against
an allow-list before being trusted:

```ts
const ALLOWED_INTEREST = new Set<string>(INTEREST_LEVELS);
const ALLOWED_PROGRAMS = new Set<string>(PROGRAM_NAMES);
const ALLOWED_TEMPS = new Set(["hot", "warm", "cold"]);
```

`score-lead/index.ts` — "segment-aware Lead Scoring agent. Runs both pre-meeting (right after
booking) and post-meeting (after `analyze-meeting` writes notes). **Single source of truth**
for `lead_score`, `lead_temperature`, `recommended_goal`, `recommended_program_codes`."
Segments (`school | student | parent | unknown`) each load their own rubric from the knowledge
module, because the business goal differs: a school representative is scored toward "host a
programme at their school," a student/parent toward "enroll." Goals are a closed enum
(`host_program`, `enroll_student`, `nurture`, `disqualify`).

Every AI-facing function checks a kill switch first:
```ts
if (!(await aiEnabled(admin))) { /* skip without touching anything — row picked up normally later */ }
```
(`_shared/ai-gate.ts`). This is the same pattern as the Learner Portal's `vault_enabled` — a
feature that can be switched off without corrupting in-flight state.

---

## 3. Knowledge tables (what an agent may say)

`sales.knowledge_programs` — what an agent may say about a programme (distinct from the
*sellable catalogue* with prices and seats: same subject, different job).
`sales.knowledge_org`, `knowledge_intents` ("what a reply means, how hot it is, what to do and
how long you have" — 12 rows distilled from 27,236 real emails, ported unchanged because
rebuilding means re-reading all of them), `knowledge_scenarios`, `knowledge_voice_rules`
(`always` / `never` / `positioning` / `hard_rule` — the kind decides whether a violation is
rewritten silently or escalated to a human), `knowledge_cadence` (timing as data: says when a
step is **due**, never that it may be **sent** — the gate's own frequency check always
overrides this table), `knowledge_base` (key/value facts, **text primary key on purpose: the
key IS the identity** — read-only to `authenticated` by design because a wrong value here is
quoted to a parent), `knowledge_faq`, `knowledge_assets` (brochures/links the agent can attach;
documented as empty in the source, so its type CHECK list is untested against real data).

This "facts live in the database, not in a prompt string" design is what lets
`buildSystemPreambleWithKnowledge()` be regenerated fresh on every call instead of drifting out
of sync across dozens of functions.

---

## 4. Email intake, classification and the reply playbook

The Email Intelligence Agent SOP (`docs/SOP_Email_Agent_Playbook.md`, "Version 1.0 — Built
from 27,236 real email interactions across 12,543 threads") is a complete operating manual for
an AI persona that runs outbound and inbound email. It is worth copying in full for any
outreach-agent build:

### Intent tiers (every inbound email classified before any reply is generated)

| Tier | SLA | Intent codes |
|---|---|---|
| **Hot** | respond within 2 hours | `MEETING_ACCEPT`, `ENROLLMENT_READY`, `WARM_INTEREST`, `REFERRAL_OFFER` |
| **Warm** | respond within 24 hours | `INFO_REQUEST`, `DELEGATE`, `TIMING_DEFER`, `CONDITIONAL_YES` |
| **Cold** | respond within 48 hours or archive | `SOFT_DECLINE`, `HARD_DECLINE`, `AUTO_REPLY`, `WRONG_CONTACT` |

Each intent code is defined by concrete signal phrases and a real example line, not an
abstract description — this is what makes the classifier (`classify-inbound`) reliable: it is
matching against patterns a human already observed in 27k real emails, not inventing a
taxonomy from scratch.

### Scenario playbook

For each conversation state (cold first touch, positive reply / meeting request, info request,
objection, decline, delegate, timing-deferred, etc.) the SOP gives: when to use the scenario, a
fixed message **structure** (hook → credibility → value-for-them → soft CTA), explicit tone
rules ("never lead with 'I hope this email finds you well'", "specificity beats generality"),
and a worked example. `compose-outreach`, `compose-followup` and `compose-reply` are the
functions that execute these scenarios, reading the matching rubric out of the knowledge
tables rather than the markdown file — the SOP is the design document; the knowledge tables
are the deployed, model-readable version of it.

### Why this shape generalises

Any outreach/support agent benefits from the same three-layer design:
1. **A closed classification scheme** with concrete signal phrases, not vibes.
2. **A fixed structural template per scenario**, so output quality does not depend on prompt
   luck.
3. **A single escalation SLA table** so "hot" always means the same urgency everywhere the
   agent is deployed.

---

## 5. The rest of the pipeline

- **Calendly**: `calendly-webhook` (inbound booking events), `calendly-register-webhook`,
  `calendly-backfill`, `calendly-refresh-times`, `calendly-day-reconcile`,
  `calendly-schema-watch` (detects upstream schema drift before it silently breaks parsing),
  `calendly-map.ts` / `calendly-questions.ts` (shared mapping of custom questions to internal
  fields, registry-driven so a new question doesn't need a code change).
- **Zoom**: `zoom-sync`, `zoom-sync-worker`, `zoom-webhook`, `zoom-taker-backfill`,
  `backfill-hosts`, `host-resolver.ts` — resolving who actually hosted a call when the
  calendar owner and the Zoom host differ.
- **Attendance**: `redecide-attendance`, `attendance-decider.ts`, `no-show-recovery`,
  `meetings-finalize-sweeper` — a meeting's final status is **decided**, not just recorded,
  because a Zoom join event alone does not prove a meaningful meeting happened.
- **Gmail**: `connect-gmail-account` / `disconnect-gmail-account` / `set-primary-gmail-account`
  (OAuth per staff mailbox, `sales.user_gmail_accounts`, 24 connected mailboxes),
  `gmail-thread-sync`, `gmail-followup-sync`, `backfill-gmail-history`, `send-gmail`,
  `gmail-crypto.ts` (token encryption at rest), `gmail-mime.ts` (message construction),
  `thread-root.ts` (collapses a reply chain to its root for continuity).
- **WhatsApp agent**: `wa-owner-agent` (the agent that actually decides what to send),
  `wa-claim-webhook` / `wa-claim-poll` / `wa-new-meeting-claim` (a human staff member "claims"
  a conversation so two people don't double-message the same lead), `wa-day-start`,
  `wa-founder-brief`, `wa-meeting-reminder`, `wa-unassigned-nudge`, `wa-delivery-check`,
  `wa-assignment.ts` (who owns this WhatsApp thread).
- **School matching**: `match-school`, `school-reconciler`, `school-domain-verify`,
  `city-resolve.ts` — resolving a lead's free-text school name to a canonical school record,
  including the "school name collision is not proof of duplication" caution from the catalogue
  rules.
- **Reports**: `send-meeting-report`, `send-followup-report`, `member-sheet-report`,
  `meetings-sheet-sync` (push to a spreadsheet for non-technical staff), `weekly-review`,
  `send-attribution-report`.
- **Core**: `master-agent` / `master-agent-daily` (the daily orchestration run),
  `agent-runner` / `agent-sweeper` (generic agent-job execution + retry sweep, backed by
  `agent-queue.ts` and `agent-gate.ts`), `classify-lead-type`, `score-lead-prep`,
  `sync-applications` (bridges paid applications into this platform's own view of the funnel),
  `lwl-core-sync` → `core-sync` (keeps `sales.core_payments`/`core_applications` as a read
  mirror of the central payments/applications tables, explicitly a mirror — the central tables
  stay authoritative).
- **Accounts**: `invite-user`, `accept-invite`, `admin-manage-user`, `auth-email`,
  `send-account-credentials`, `auth-guard.ts` (`guard()`, `canAccessMeetings()`,
  `isAdminCaller()`, `FORBIDDEN_MEETING` — a function-level deny-list for meetings that must
  never be surfaced through the general API, e.g. ones under legal hold).

---

## 6. The migration this platform went through

The repo contains a `supabase/migrate-to-own/` toolkit (`02_export_data.sh`,
`03_load_data.sh`, `04_auth_users.sql` + a Python generator, `05_cron.sql`, `06_rls.sql`,
`06_secrets_checklist.md`, `07_verify.sql`) and a `deploy-all.sh` that deploys every edge
function to a target project, reading each function's `verify_jwt` setting straight out of
`supabase/config.toml` so the deploy step never has to duplicate that decision:

```bash
mapfile -t NO_JWT < <(awk '
  /^\[functions\./ { name=$0; sub(/^\[functions\./,"",name); sub(/\]$/,"",name); next }
  /^verify_jwt[[:space:]]*=[[:space:]]*false/ { if (name != "") print name; name="" }
' "$CONFIG")
```

This is the evidence trail for a real platform migration: the repo's own original Supabase
project was superseded by the shared data project's `sales` schema, with every edge function,
every cron schedule and every RLS policy carried across deliberately rather than organically
drifting. **Worth copying whenever a platform outgrows its own project**: export data, generate
`auth.users` rows on the target, apply RLS, re-point cron, verify, then redeploy every function
with the exact same `verify_jwt` posture it had before — never re-decide auth posture during a
migration, only relocate it.

---

## 7. What to copy from this design

1. **A single knowledge module** assembling the system preamble fresh per call, so every agent
   — analysis, scoring, drafting — is provably consistent, instead of N copies of a prompt that
   drift.
2. **Validate every model output against a closed enum** before it is trusted or written.
3. **A kill switch checked first, every time**, with "skip without touching anything, pick it
   up again later" as the disabled behaviour — never a silent partial write.
4. **Facts as data, not prompt text**: a `knowledge_base` keyed by a meaningful text primary
   key, read-only to normal callers, because a wrong fact here is said to a real customer.
5. **Classification before generation**: a closed set of intent codes with concrete example
   phrases, each with its own SLA.
6. **A fixed message structure per scenario**, documented with a worked example, not a vague
   style guide.
7. **Decide attendance/outcome, don't just log an event** — a join event is not proof a
   meaningful interaction happened.
8. **A claim/assignment layer on top of an agent** so a human and an automation never
   double-act on the same conversation.
9. **Mirror, never duplicate authority** — a read mirror of central data states plainly that
   the source system stays authoritative.
10. **Carry `verify_jwt` posture through a migration programmatically**, reading it from config
    rather than re-deciding it by hand per function.
