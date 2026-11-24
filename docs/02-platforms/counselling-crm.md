# Counselling CRM

A standalone client project, included in this skill because it demonstrates the full pattern
of a Supabase build done carefully and reviewed honestly — not part of the shared estate at
all, and never referenced by it.

## Identity

- Framework: TanStack Start (bun), server functions (`*.functions.ts`), `requireSupabaseAuth` +
  a global auth middleware.
- Supabase: its own standalone project, unrelated to the estate.
- Scale: ~23 commits over a few days — a small, fast build, useful precisely because its whole
  lifecycle (spec → schema → frontend → live account → security review → fixes →
  re-verification) is short enough to read end to end.

## What it is

A workflow CRM for a career-counselling practice that had been running entirely on a shared
spreadsheet by memory. Stages: Inquiry → Payment → Pre-counselling → Career assessment →
Roadmap report → Roadmap meeting → Follow-up, with the stage timing and "what's due next"
logic kept in exactly one module (`src/lib/workflow.ts`, unit-tested) so every page calls
`getNextAction()`/`currentStage()` rather than re-implementing timing.

## Schema

`profiles` (role enum owner/admin/member, auto-created via an `on_auth_user_created`
trigger — **this is the one place in this skill's examples where such a trigger is actually
used**, contrasted with the shared estate's deliberate choice to have none), `programs`
(editable list, not a hardcoded enum), `students` (soft-delete via `deleted_at`, guarded by a
trigger so only owner/admin can set it), `student_stage_events` (normalised stage-completion
history, not a JSON blob), `payment_transactions` (an append-only ledger — fee stays a plain
column on students, received/pending are *derived* from the ledger sum, which naturally
supports instalments without a separate instalment table), `activity_log` + `notifications`
(Realtime-enabled, fed by triggers on student/stage/payment inserts, deliberately kept separate
from the "what's due" countdown logic so stage-timing stays single-sourced).

## Two real bugs that only a live login caught

Code review, `tsc --noEmit` and a passing test suite all looked clean; logging in for real
surfaced two genuine production bugs within a minute each:

1. **Missing table-level GRANTs under otherwise-correct RLS** — every request 403'd with
   "permission denied for table X," which reads exactly like a broken policy but isn't. A
   hand-written SQL migration applied via the CLI does not get Supabase Studio's automatic
   default grants for free. See `docs/01-backend/rls-security-and-grants.md`.
2. **A fixed Realtime channel topic name collided across React's dev-mode double effect
   invocation** — the second mount got handed the first mount's already-subscribed channel
   object, and calling `.on()` on it threw, crashing the whole route. Fixed by suffixing the
   topic with a random value per effect run.

Both are now standing items in this skill's RLS and frontend-auth documents, not
CRM-specific footnotes.

## A security scan the hard way

The project's own AI builder ran a security scan independently and pushed fixes directly to
the connected branch mid-session — a live example of "another agent can push to your branch at
any moment" (see `docs/04-playbooks/working-across-sibling-repos.md`). Every one of its four
fixes was independently re-verified against the live database (function EXECUTE privileges,
RLS policy text, role-table sync) rather than trusted from its summary — and one "still
showing" finding turned out to be the scanner quoting a stale migration file rather than the
live, already-fixed ACL, which is the standing lesson on distinguishing a scanner's static
evidence from a system's live state.
