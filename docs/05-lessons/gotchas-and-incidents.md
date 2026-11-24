# Gotchas, incidents and the rules they produced

A year of building on Supabase + Lovable + many AI agents produced a specific set of scars. Each entry: what happened, why it was hard to see, the rule. Entries are grouped; dates are when the lesson was recorded.

---

## A. Postgres / Supabase

### A1. RLS policies with no table GRANT are a silent no-op (2026-11-21)
Postgres checks `GRANT`s **before** RLS. Supabase Studio's table editor applies default grants automatically; a hand-written SQL migration via the CLI does not. Symptom: every request 403s with `permission denied for table X` — looks like a policy bug, isn't.
**Rule:** every hand-written migration includes `GRANT USAGE ON SCHEMA <s> TO authenticated;` and `GRANT SELECT/INSERT/UPDATE/DELETE ON <table> TO authenticated` scoped to what the policies intend. When a fresh table 403s, check grants before policy logic. Re-check on every new table.

### A2. RLS enabled + zero policies = silently empty, not an error (2026-11-05)
On Brain, `research_campaigns`, `research_contacts`, `research_schools`, `brain_ops.npf_jobs`, `campaign_recipients`, `campaign_links`, `research_allocations`, `research_claims`, `research_tracking_events` all had RLS on with no policies. Direct `.from(table).select()` from `authenticated` returned `[]`. The dashboard showed "Schools 0 / Contacts 0" with 22,979 schools in the table.
**Rule:** on Brain, every read/write is a `SECURITY DEFINER` RPC (`public.brain_api_*`). Zero `.from(` calls remain in `src/lib`. On the estate, the 192 RLS-enabled-no-policy tables are the intended default-deny state for service-role-only tables — not a vulnerability, but also not readable from a browser.

### A3. A CHECK constraint inside an audit insert rolled back whole transactions (2026-11-05)
Eleven new Brain RPCs inserted `workstream='campaigns'`; the live `action_audit_workstream_check` only allowed singular `campaign`. Every function raised at the audit line and rolled back everything, while earlier calls had *appeared* to succeed.
**Rule:** after writing an RPC, do a live save → re-query → cleanup round-trip. Never trust the return value of a function you have not re-queried.

### A4. Immutability trigger on live campaigns (2026-11-05)
`brain_ops.guard_campaign_tracking_identity_v2` makes `research_campaigns.attribution_source_slug` immutable once `status` is `live`/`done`. All 70 existing campaigns were `done`, so source mapping only works for new drafts. The user chose to respect the guard rather than backfill. **Do not build a bypass without asking again.**

### A5. `ON CONFLICT` does not collapse NULLs
`campaign_links` has a nullable `step_id` in its unique key; plain unique constraints never treat two NULLs as equal, so idempotency had to be an explicit SELECT-then-INSERT.

### A6. Postgres grants EXECUTE to PUBLIC on every new function (2026-11-21)
`notify_owners_and_admins` was callable via PostgREST RPC by anyone with arbitrary title/body.
**Rule:** `REVOKE EXECUTE ON FUNCTION ... FROM PUBLIC;` and re-grant only to the roles that need it; verify with `has_function_privilege`.

### A7. Migration files ≠ live state
LMS and Pre-Med Platform migration history both contain triggers (e.g. a `user_roles`-populating trigger on `auth.users`) that do not exist live. **Rule:** `select * from pg_trigger` / `pg_policies` / `has_function_privilege` before claiming anything exists; fetch the deployed edge function body, not just the git file.

### A8. Security scanner wording vs live ACLs (2026-11-21)
Lovable's scan kept flagging "misleading alerts" because its evidence cited the original migration (implicit PUBLIC execute at creation), not the live ACL after the later REVOKE. **Rule:** when a scanner is ambiguous about live vs source, verify the live system directly; treat the finding as resolved only after a fresh `has_function_privilege`/`pg_policies` query.

### A9. Realtime channel topic collision (2026-11-21)
Fixed topic `notifications:${userId}` + React dev double effect-invoke → second mount got an already-subscribed channel; `.on()` threw; root error boundary crashed the route. **Rule:** suffix every Realtime topic with a random string per effect run.

### A10. Log bloat is a real cost
`net._http_response` was 91 MB for 6 rows and `cron.job_run_details` 67 MB on the estate (158 of 487 MB). Neither prunes itself. **Rule:** schedule trims.

### A11. 261 RLS policies call `auth.uid()` per row
Performance advisor on the estate: wrap as `(select auth.uid())` so it is evaluated once per query; 167 overlapping permissive policies; 153 unused indexes; 64 unindexed FKs. Known, not urgent at current size.

### A12. Dropped column with CASCADE
Caused alarm but no damage. **Rule:** check the dependency chain one level deeper than feels necessary before and after.

---

## B. Shared-estate behaviour

### B1. Agents write central rows with guessed values
Fellowship Platform's agent wrote programmes with a guessed 399 price. **Rule:** never write a central row without confirmed price/code/dates; deactivate rather than delete when reverting.

### B2. Agents act ahead of instructions
Test-Prep Platform's agent executed a full migration after being told to assess. A recon-only fork in the 2026-11-05 pricing audit made a live edit (correct, but unasked). **Rule:** assessment and migration are separate documents; recon-only prompts must say "do not edit" forcefully; verify state directly rather than trusting an agent's report.

### B3. Secrets leaked through views
Four platform views exposed `join_url`/`passcode`, two to `anon`. **Rule:** no secret column in any view; verify by column list; clients read `all_masterclasses_public` only.

### B4. `anon` had full DML behind RLS
Revoked to SELECT on the five shared tables. **Rule:** revoke what is not needed even when RLS would block it.

### B5. Fire-and-forget replication
Four platforms queued an HTTP call and assumed success. **Rule:** sync must verify the remote write and record failures (Pre-Med Platform, Founders Platform, Purpose Platform, EDGE built logged, retryable mechanisms — `premed.side_effects_queue`, `research.replication_log`).

### B6. Founders Platform's loose payment matching emailed strangers
9 unmatched $499/£499 payments received a pending-registration email. **Rule:** route on `metadata.program_code`; unattributed payments go to manual review (`<schema>.unmatched_payments`); never auto-match on amount or timing.

### B7. Schema renames looked like lost data
`sales` → `finance`, `main` → `portal`. **Rule:** renames are announced in the estate table; any prompt still saying the old name is stale.

### B8. Early-bird pricing drifted across six checkouts (2026-11-05)
Financial Literacy Platform and Pre-Law Platform never fetched `early_bird_price` (always charged sticker). Pre-Med Platform gated it on a hardcoded date constant. Research Platform had a decoy `sticker_major` always truthy and no strike UI. Challenge Platform's "at a glance" tile ignored the deadline. Fellowship Platform was correct (tested `_shared/cataloguePricing.ts`). **Rule (reference = Application Portal):** `amount = early_bird_price` when populated and > 0, else `sticker_price`; no date-based expiry; strike-through only when a real discount exists; promo and early-bird strikes never shown simultaneously.

### B9. Payment links have no price floor
The Growth Partner spec said links are "never below the floor we set" — aspiration, not reality. `admin-api` only checks `amount > 0`. Promo codes do have a cap (checked at creation and at checkout). **Rule:** read the code before trusting a spec's description of current behaviour.

### B10. `portal` is read more widely than documented
`portal.payment_links` and `app_settings` (`key='extra_charge_payment_links'`) are read by Financial Literacy Platform, Pre-Med Platform, Pre-Law Platform, Fellowship Platform, Research Platform checkouts read-only; only the Application Portal's `admin-api` writes them. The master docs understate this.

### B11. `promo_codes` live in `public`, not `portal`
Not in either master architecture doc. Written by multiple partner platforms' edge functions via service role; RLS enabled, zero policies; browsers never touch them.

### B12. Stale table comments mislead
`affiliate.partners` claimed its `auth_user_id` referenced a different project's `auth.users` after the migration to the shared estate. Fixed 2026-11-23. **Rule:** when a schema moves projects, sweep `COMMENT ON` text too.

---

## C. Lovable coexistence and git

### C1. Lovable's own agent pushes while you work
It fixed 3 of 4 scan findings and pushed 4 commits mid-session (Counselling CRM). The Application Portal once gained 45 upstream commits in minutes. **Rule:** `git fetch` immediately before any merge/push, not just early; read every diff; re-check `supabase migration list --remote`.

### C2. `git checkout <branch>` does not update from origin
A locally-stale `master` merged with a feature branch fast-forwards past what origin really has; the push is rejected and the diff looks like Lovable reverted your work. **Rule:** fetch + reset/merge explicitly before the final merge.

### C3. Remote naming
Application Portal's remote is `newrepo`, not `origin`; `git fetch origin` fails with a generic access error.

### C4. Map a local repo to its Lovable project by `latest_commit_sha`
"Challenge Platform" is Lovable project "Global Future Innovators Challenge: Challenge Platform"; name search finds nothing.

### C5. `set_project_knowledge` replaces, 10 KB limit
Read existing content first, measure bytes with `wc -c`, write an appendix if near the cap. Workspace-level knowledge is 403 for collaborators.

### C6. npm `EOVERRIDE` vs bun tolerance
Same package in `overrides` and `dependencies` with non-identical ranges: `npm install` fails, `bun install` is silent. Lovable's pipeline likely uses npm. **Rule:** `npm install --dry-run` when "build errors" can't be reproduced with bun.

### C7. CRLF checkouts make `bun run lint` fail everywhere
`prettier/prettier "Delete ␍"` on untouched files. Filter lint to changed files; do not mass-fix.

### C8. Stale `node_modules/.vite` cache → spurious "Invalid hook call"
Delete the cache and restart.

### C9. Lovable project knowledge must carry the shared-estate rules
Otherwise Lovable's agent reverts hardened registration back to bare `supabase.auth.signUp()`.

---

## D. Process and verification

### D1. "tsc clean + tests pass + code review" ≠ "works"
Both A1 and A9 were found within a minute of a real login. **Rule:** for any auth/RLS feature, log in for real before calling it done.

### D2. A route-table audit misses components
The 2026-11-23 responsive audit covered 51 files from the route list; the user found `SchoolStats.tsx` clipping on a phone the next day; a re-sweep found 4 more of the same class. **Rule:** build audit file lists from `find src/pages` and actual imports, not from the routes a fork happens to think of; when a live bug is reported, re-sweep for its whole class.

### D3. Cross-repo work: one fork per repo, scope-locked, diff back for review
Never let a fork run git; serialise git yourself to avoid races.

### D4. Verify independently — even Lovable's own summaries
Spot-check LMS shell claims against `RoleLayout.tsx`, colour claims against `styles.css`, schema claims against `execute_sql`.

### D5. Decline literal clones of other sites
`a site-cloning CLI` scrape / "exact same design" requests were declined twice; built an original in the same spirit with Group content and brand. The user accepted that framing.

### D6. Reduced-motion fallback is not what the user sees
Describe the default full-3D experience, not the `prefers-reduced-motion` 2D path.

### D7. Memory corrections
Several memory notes were later corrected (Lovable project id for Growth Partner Portal; "each repo is its own project" → "six share one"; "not cloned locally" → cloned). **Rule:** verify "which project does this repo use" against the imported client file, not `.env` alone (LMS has two clients).

---

## E. Frontend

### E1. `flex-1` search input in a toolbar must have `min-w-0`
A text input's intrinsic min-width ignores `flex-1`; it pushed the Download button off-screen on `SchoolStats.tsx`.

### E2. Toolbar rows need `flex-wrap` / `flex-col` fallbacks; dialog grids need a mobile stacking fallback; long emails need `break-words`.

### E3. `overflow-hidden` on a table wrapper silently clips; use `overflow-x-auto`.

### E4. Touch targets 44px on signature pads and mobile close buttons.

### E5. Design restraint (Counselling CRM): saturated solid gold blocks + `text-5xl` + `rounded-3xl` read as playful; the corrected scale is `h-9` buttons, `rounded-xl/2xl`, headings ≤ `text-2xl`, gold as accent.
