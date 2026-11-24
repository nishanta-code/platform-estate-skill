# Playbook: write and apply a Supabase migration safely

Applies to every repo. The Lovable block in each `AGENTS.md` says "do not add or edit `supabase/migrations/`; ask Lovable" — that is the default. The user has repeatedly overridden it for specific sessions (Counselling CRM CRM via CLI with a transient PAT; estate hardening via MCP `apply_migration`/`execute_sql`). When overridden, this is the discipline.

## Before
1. `git fetch` and compare `main` vs `origin/main` (or `newrepo/main`) — another agent may have pushed migrations. `supabase migration list --remote` must match local.
2. Inspect the live object you are changing: `pg_tables`, `pg_policies`, `pg_trigger`, `pg_proc`, `information_schema.columns`. Migration files are not the truth.
3. Decide the schema. Never touch another platform's schema. Cross-schema writes into shared tables (`public.promo_codes`, `portal.payment_links`) require explicit founder sign-off and a note in `AGENTS.md` (the Growth Partner Portal precedent).
4. Additive only unless explicitly authorised: new tables/columns/functions/policies. No `ALTER`/`DROP` on anything pre-existing without the dependency chain checked one level deeper than feels necessary.

## The migration file
Name: `YYYYMMDDHHMMSS_<snake_description>.sql` in `supabase/migrations/`. Contents checklist:

```sql
-- 1. objects
create table if not exists <schema>.<table> (
  id uuid primary key default gen_random_uuid(),
  ...,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
comment on table <schema>.<table> is '<who writes it, who reads it, why it exists>';

-- 2. RLS
alter table <schema>.<table> enable row level security;
create policy "<table>_owner_select" on <schema>.<table>
  for select to authenticated using ((select auth.uid()) = user_id);   -- (select auth.uid()) evaluates once per query
-- insert policies must bind created_by = auth.uid(); never `using (true)`

-- 3. GRANTS (hand-written migrations get none automatically)
grant usage on schema <schema> to authenticated;
grant select, insert, update on <schema>.<table> to authenticated;   -- only what the policies allow
-- anon only if genuinely public

-- 4. functions
create or replace function <schema>.<fn>(...) returns ... language plpgsql
security definer set search_path = <schema>, public as $$ ... $$;
revoke execute on function <schema>.<fn>(...) from public;
grant execute on function <schema>.<fn>(...) to service_role;        -- or authenticated when it is the API surface

-- 5. triggers (updated_at, guards, audit) — verify with pg_trigger after apply

-- 6. indexes for every FK and every RLS predicate column
```

Rules baked in:
- Comments on tables encode design decisions (the data project does this exhaustively — "D-014: refunds are rows, never a status string"). Write them.
- Audit/log tables append-only; the log outlives the staff row.
- Store money as minor units + currency code (data project) or whole units in the catalogue (`sticker_price`) — never mix; never convert at rest.
- Soft-delete via `deleted_at` guarded by a trigger when only some roles may delete (Counselling CRM `students`).
- Role authority in one table (`user_roles`) with a guard trigger for the last owner; mirrors are read-only columns.
- Idempotent inserts: `on conflict` only works when no nullable column is in the unique key; otherwise SELECT-then-INSERT.
- CHECK constraints on enumerations (`workstream in (...)`) — and read them before inserting.

## Apply
- Preferred: `supabase db push` with the CLI (`SUPABASE_ACCESS_TOKEN` as a transient env var; never written to a file) or the MCP `apply_migration` (records it in `supabase_migrations.schema_migrations`). Raw `execute_sql` is for reads, verification and one-off repairs — if you use it for DDL, also add the file so the repo matches.
- Never run DDL from a forked/recon agent. One session serialises all schema changes.

## After (live verification, every time)
```sql
select tablename, rowsecurity from pg_tables where schemaname='<schema>';
select * from pg_policies where schemaname='<schema>' and tablename='<table>';
select grantee, privilege_type from information_schema.role_table_grants
 where table_schema='<schema>' and table_name='<table>';
select tgname from pg_trigger where tgrelid='<schema>.<table>'::regclass and not tgisinternal;
select has_function_privilege('anon','<schema>.<fn>(<args>)','execute');
```
- Then a live round-trip: insert → select → delete through the real client (authenticated, not service role).
- `get_advisors` (security + performance) — no new findings.
- Log in for real if auth/RLS is involved (A1/A9 in the lessons file were only caught this way).
- Commit the migration file; `git fetch` again before pushing; push to a feature branch unless the user authorised `main`.
