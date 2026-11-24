# RLS, grants and the estate's security posture

Sources: a live metadata audit of the shared estate project (353 tables, 78 views, 436
functions, 385 policies, 222 triggers, 8 event triggers across 14 business schemas — see
`reference/sql/estate-*.md`), the estate master architecture document's security section, and
the incidents in `docs/05-lessons/gotchas-and-incidents.md`.

---

## 1. The default posture

- **RLS is on everywhere** — enforced by an event trigger (`ensure_rls`) that fires whenever a
  new table is created, so a forgotten `ENABLE ROW LEVEL SECURITY` is structurally hard to ship.
- **RLS enabled with zero policies is the default-deny state**, not a bug, for the majority of
  tables in several schemas (most of the fellowship schema, most of the finance schema, 14
  tables in `public` including the central payment and promo tables). These are
  service-role-only or `SECURITY DEFINER`-RPC-only by design.
- `anon` and `authenticated` have **no schema `USAGE`** grant at all on at least two platform
  schemas — not just no table grants, a step further back, meaning no client call into that
  schema can succeed regardless of RLS.
- A handful of **custom login roles** exist outside the usual `anon`/`authenticated`/
  `service_role` triad — one platform's own site writes through a dedicated login role with its
  own narrow RLS policies rather than the service-role client every other platform uses. One
  role carries `BYPASSRLS` for a foreign-data-wrapper reader. Check `pg_roles` before assuming
  only the standard three exist.
- Explicit `service_role` policies exist on a few tables even though `service_role` already
  bypasses RLS — redundant, but harmless; don't be alarmed by them, and don't assume their
  presence means RLS is actually gating that role.

## 2. The two silent-failure modes (memorise both)

### 2.1 RLS policy without a table GRANT

Postgres checks `GRANT`s **before** it evaluates any RLS policy. Supabase Studio's table editor
applies sane default grants automatically when a table is created through the UI. A raw SQL
migration applied via the CLI or the Management API does **not** get this for free. The
symptom is deceptive: every request 403s with `permission denied for table X`, which reads
exactly like a broken RLS policy — the actual fix is a plain
`GRANT SELECT/INSERT/UPDATE/DELETE ON <table> TO authenticated;` scoped to whatever the
policies already intend to allow.

**Checklist for every hand-written migration:**
```sql
GRANT USAGE ON SCHEMA <schema> TO authenticated;
GRANT SELECT, INSERT, UPDATE, DELETE ON <schema>.<table> TO authenticated;  -- only what policies allow
-- anon only if the table is genuinely public-writable (e.g. a lead-capture INSERT)
```

### 2.2 RLS enabled with zero policies

This is the deliberate default-deny shape for most of the estate's tables — but on a platform
that expected normal client reads to work, it produces a **silent empty result**, not an error.
A dashboard that shows "0 schools / 0 contacts" when 20,000+ rows exist in the table is this
bug, not a data problem. The fix when the no-policy state was unintentional is either real RLS
policies, or — the estate's preferred pattern where fine-grained access logic is complex — a
`SECURITY DEFINER` RPC that the client calls instead of `.from(table).select()` directly (see
`docs/01-backend/research-attribution-and-rpc-gate.md` for the fullest worked example of this
pattern).

## 3. Function privilege hygiene

- Postgres grants `EXECUTE` to `PUBLIC` on every newly created function **by default**. A
  function meant to be internal (e.g. a notification-sender) is therefore callable by any
  authenticated client over PostgREST's RPC endpoint unless this default is explicitly revoked.
  **Always**: `REVOKE EXECUTE ON FUNCTION <fn>(<args>) FROM PUBLIC;` then grant only to the
  roles that actually need it, and verify live with
  `select has_function_privilege('anon', '<schema>.<fn>(<args>)', 'execute');`.
- Keep `SECURITY DEFINER` functions thin — a point lookup, a validated write, an audit insert —
  never a general-purpose query surface. Pin `search_path = <schema>, public` on every one, or
  a caller-controlled search path can redirect an unqualified identifier to a different table.
- Keep views `security_invoker = true` unless there is a specific reason for definer semantics;
  this is how a table safely moved between schemas while leaving a compatibility view behind
  without re-exposing more privilege than the original table had (see the applications-table
  relocation in the payments document).

## 4. Secrets management

- No secret values belong in a migration file, an edge function's source, or a cron job's
  literal command text. Two real findings in a live audit of this estate: scheduled cron jobs
  on one platform embedded a literal cron-auth header value directly in `cron.job.command`,
  and two function bodies on another platform had a literal bearer token hard-coded — both
  should instead read from `Deno.env.get(...)` backed by a Supabase secret, or from
  `supabase_vault`, exactly as a sibling platform's cron jobs already correctly do.
- `supabase_vault` holds genuinely sensitive long-lived values (a service-role key kept for a
  dependent platform, for example) — audit vault contents for orphaned secrets after a platform
  migrates away from needing them, but don't delete one without confirming nothing still reads
  it.
- Storage buckets: check `public` + "listing allowed" together — a bucket can be `public` by
  design (served assets) but should almost never also allow directory listing.

## 5. Housekeeping that is also a security posture

- **Log tables bloat fast and never prune themselves.** A pg_net response-log table was found
  at tens of megabytes for a handful of rows (dead tuples never vacuumed); a cron-run-history
  table reached well over a hundred megabytes across hundreds of thousands of rows never
  purged. Schedule a trim; it costs nothing and the data has no operational value past a short
  window.
- **Database webhook triggers that fire-and-forget to an external endpoint** (Apps-Script style
  integrations, third-party automations) should log their outcome somewhere queryable, not just
  fire. A matching row count on the receiving end is not proof the webhook actually succeeded
  every time — see the "verify the remote write" lesson.
- **Autovacuum/analyze gaps** on tables that grow fast but are rarely read directly (append-only
  audit/event tables, mirrors) degrade query planning silently over months; check
  `pg_stat_user_tables.last_autovacuum`/`last_analyze` periodically on the biggest tables.

## 6. Auth config that must not be "helpfully" tightened

Verified live and worth restating because it looks like it should be hardened and must not be:
`sessions_timebox = 0`, `sessions_inactivity_timeout = 0`, `sessions_single_per_user = false`,
refresh-token rotation on with a short reuse interval. A shared learning/portal product
depends on indefinite, multi-device sessions; enabling any of the three zeroed settings would
silently log out real users mid-course. If a security review recommends tightening session
policy here, treat it as a product decision to escalate, not a default-safe change to apply.

## 7. Advisor-driven findings worth treating as a standing checklist

From a real advisor pass across the estate, roughly a tenth of the findings were genuinely
worth acting on and the rest were the expected shape of the system (RPC-exposed `SECURITY
DEFINER` functions, default-deny no-policy tables). The signal-worth-acting-on categories:
- A table with RLS fully **disabled** (not just no-policy) — always critical, always fix before
  anything else, but check what policies are needed before enabling (enabling with none locks
  it to nothing, which can itself be an outage).
- A secret value or column (`join_url`, `passcode`-equivalents) leaking through a view, two
  platforms deep, to `anon`.
- `anon` holding DML grants on a shared table "just in case," with RLS as the only thing
  actually stopping a write — revoke the grant too, don't rely on RLS alone to be the whole
  defence.
- Policies that call a volatile function like `auth.uid()` once per row instead of once per
  query — wrap as `(select auth.uid())` for the performance win, not a security fix, but cheap
  and worth doing as part of the same pass.

## 8. The verification loop (repeat for every change)

```sql
select tablename, rowsecurity from pg_tables where schemaname = '<schema>';
select * from pg_policies where schemaname = '<schema>' and tablename = '<table>';
select grantee, privilege_type from information_schema.role_table_grants
 where table_schema = '<schema>' and table_name = '<table>';
select tgname from pg_trigger where tgrelid = '<schema>.<table>'::regclass and not tgisinternal;
select has_function_privilege('anon', '<schema>.<fn>(<args>)', 'execute');
```
Then a real round-trip through the actual authenticated client (not service role), and — for
anything touching auth or RLS — a real login. Migration files, chat summaries and even this
document are not the live system; the live queries above are.
