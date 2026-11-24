# The Brain RPC gate: RLS enabled + zero policies + SECURITY DEFINER `brain_api_*`

> Source: `lwl-data-outreach` repo (Group Research & Outreach), Supabase project "Brain"
> `<BRAIN_REF>`. Real SQL below is copied from the one committed migration
> `supabase/migrations/20261029121253_<uuid>.sql`; everything else is
> reconstructed from call sites in `src/lib/*.functions.ts` and from commit messages (11-05/11-06), and is
> marked as such. No secrets appear here.

---

## 1. What the pattern is

```
 browser ──bearer JWT──▶ TanStack server fn ──supabase-js (publishable key + caller JWT)──▶ PostgREST
                                                                                            │
                                                              SELECT public.brain_api_x(...)   ← only entry point
                                                                                            │
                                                     SECURITY DEFINER, search_path pinned, actor := auth.uid()
                                                     role check on brain_ops.member_roles
                                                     work on tables that have RLS ON and NO policies
                                                     INSERT brain_ops.action_audit (workstream, tool_name, action, ...)
                                                     RETURN jsonb {status: 'ok' | 'access_denied' | ...}
```

- Every research/campaign table has **RLS enabled and zero policies**. For the `authenticated` and `anon`
  roles that means `SELECT` returns no rows and writes are rejected – the table is effectively invisible
  through the Data API even with a valid session.
- The **only** way in is a function named `public.brain_api_<verb>_<noun>` that is `SECURITY DEFINER`
  (runs as its owner, typically `postgres`, so RLS is bypassed inside), `REVOKE`d from `PUBLIC`/`anon`,
  `GRANT EXECUTE`d to `authenticated`, and that re-derives the caller from `auth.uid()`.
- No service-role key exists in the app (`src/integrations/supabase/client.server.ts` throws on access);
  `LWL_BRAIN_PUBLISHABLE_KEY` is rejected if it starts with `sb_secret_`.

## 2. Why it exists

1. **Fail-closed data privacy** – the README (cutover 10-29) explicitly refuses to "widen RLS as a workaround"
   and says effective per-user row visibility has not been changed. Zero policies + definer functions means
   every exposure is an explicit, reviewable function.
2. **Single authorization source** – `brain_ops.member_roles` is the one place roles live; both the app
   (`brain_api_my_roles`) and the ChatGPT MCP go through it.
3. **Auditability** – mutating RPCs insert into `brain_ops.action_audit` in the same transaction.
4. **Shape stability** – RPCs return a jsonb envelope with a `status` field so clients never depend on
   table columns; the generated `types.ts` is deliberately unused for Brain queries.
5. **No privileged key in a Lovable-hosted app** – the whole design assumes the browser-safe key only.

## 3. Canonical template (real SQL from the committed migration)

### 3.1 Read-only admin RPC

```sql
CREATE OR REPLACE FUNCTION public.brain_api_admin_list_members()
RETURNS jsonb LANGUAGE plpgsql STABLE SECURITY DEFINER SET search_path TO 'pg_catalog'
AS $$
DECLARE v_actor uuid := auth.uid();
BEGIN
 IF v_actor IS NULL OR NOT EXISTS(SELECT 1 FROM brain_ops.member_roles m WHERE m.user_id=v_actor
   AND m.active AND m.revoked_at IS NULL AND m.role='admin') THEN
   RETURN jsonb_build_object('status','access_denied');
 END IF;
 RETURN jsonb_build_object('status','ok','members',(
  SELECT coalesce(jsonb_agg(jsonb_build_object(
    'user_id',u.id,'email',u.email,
    'display_name',u.raw_user_meta_data->>'display_name',
    'last_sign_in_at',u.last_sign_in_at,'created_at',u.created_at,
    'roles',(SELECT coalesce(jsonb_agg(m.role ORDER BY m.role),'[]'::jsonb) FROM brain_ops.member_roles m
             WHERE m.user_id=u.id AND m.active AND m.revoked_at IS NULL)
  ) ORDER BY lower(u.email)),'[]'::jsonb) FROM auth.users u WHERE u.deleted_at IS NULL));
END $$;
```

### 3.2 Mutating admin RPC with audit

```sql
CREATE OR REPLACE FUNCTION public.brain_api_admin_set_member_role(p_user_id uuid, p_role text, p_active boolean)
RETURNS jsonb LANGUAGE plpgsql SECURITY DEFINER SET search_path TO 'pg_catalog'
AS $$
DECLARE v_actor uuid := auth.uid();
BEGIN
 IF v_actor IS NULL OR NOT EXISTS(SELECT 1 FROM brain_ops.member_roles m WHERE m.user_id=v_actor
   AND m.active AND m.revoked_at IS NULL AND m.role='admin') THEN
   RETURN jsonb_build_object('status','access_denied');
 END IF;
 IF p_user_id IS NULL OR p_active IS NULL OR p_role NOT IN ('admin','researcher')
    OR NOT EXISTS(SELECT 1 FROM auth.users WHERE id=p_user_id) THEN
   RETURN jsonb_build_object('status','invalid_input');
 END IF;
 IF p_user_id=v_actor AND p_role='admin' AND NOT p_active THEN
   RETURN jsonb_build_object('status','cannot_remove_own_admin');
 END IF;
 IF p_active THEN
   INSERT INTO brain_ops.member_roles(user_id,role,active,granted_by,granted_at,revoked_at)
   VALUES(p_user_id,p_role,true,v_actor,now(),NULL)
   ON CONFLICT(user_id,role) DO UPDATE SET active=true,granted_by=v_actor,granted_at=now(),revoked_at=NULL;
 ELSE
   UPDATE brain_ops.member_roles SET active=false,revoked_at=now()
   WHERE user_id=p_user_id AND role=p_role AND active;
 END IF;
 INSERT INTO brain_ops.action_audit(actor_user_id,workstream,tool_name,action,entity_kind,entity_id,result_status,details)
 VALUES(v_actor,'admin','set_member_role',CASE WHEN p_active THEN 'grant' ELSE 'revoke' END,
   'member_role',p_user_id::text,'completed',jsonb_build_object('role',p_role,'active',p_active));
 RETURN jsonb_build_object('status','ok');
END $$;
```

### 3.3 Grants (always the last block of the migration)

```sql
REVOKE ALL ON FUNCTION public.brain_api_admin_list_members() FROM PUBLIC, anon;
REVOKE ALL ON FUNCTION public.brain_api_admin_set_member_role(uuid,text,boolean) FROM PUBLIC, anon;
GRANT EXECUTE ON FUNCTION public.brain_api_admin_list_members() TO authenticated;
GRANT EXECUTE ON FUNCTION public.brain_api_admin_set_member_role(uuid,text,boolean) TO authenticated;
```

Things to notice in the real code:
- `SET search_path TO 'pg_catalog'` – deliberately *empty of user schemas*, so every table reference is
  schema-qualified (`brain_ops.member_roles`, `auth.users`). This blocks search-path hijacking for a
  definer function. (`auth.uid()` still resolves because it is schema-qualified by name.)
- `STABLE` on the read-only function, default `VOLATILE` on the writer.
- The role check reads `active AND revoked_at IS NULL` – both flags, not just one.
- Status strings are the API: `access_denied`, `invalid_input`, `cannot_remove_own_admin`, `ok`.
- Identity: `auth.uid()`, never a `p_actor` parameter. (Older Brain functions in the types snapshot –
  `brain_api_create_draft_campaign(p_actor, ...)`, `_add_draft_recipient(p_actor, ...)` – trust a
  client-supplied actor and are therefore grant-locked to `postgres`/`service_role`; commit 2e2c72d chose
  not to reuse them. Several still-used read RPCs accept `p_actor` (`list_schools`, `research_metrics`,
  Instantly settings); the app passes `context.userId`, but the server side should still validate it
  against `auth.uid()` – treat the parameter as advisory.)

## 4. Recipe for a new `brain_api_*` RPC

### 4.1 Decide the gate

| Gate | Check | Used by |
|---|---|---|
| self / caller-scoped | `auth.uid()` not null; return only rows where `owner = v_actor` | `my_roles`, `my_research_dashboard`, `school_coverage_self` |
| member | any row in `brain_ops.member_roles` with `active AND revoked_at IS NULL` | `list_schools`, `list_contacts`, `get_school`, `list_research_ops`, `list_tracking_events` |
| lead+ | role IN ('admin','lead') | all `campaign*`, `data_health`, `sources_summary`, templates, link generation |
| admin | role = 'admin' | `admin_*`, `set_campaign_source` (per commit), Instantly settings (section `integrations`) |

The app-side `SECTIONS` map in `src/lib/permissions.ts` must agree with the DB gate; both are enforced.

### 4.2 Skeleton (reconstructed from the committed migration + 11-05 commits; adapt names)

```sql
CREATE OR REPLACE FUNCTION public.brain_api_<verb>_<noun>(p_campaign_id uuid, p_limit int DEFAULT 25, p_offset int DEFAULT 0)
RETURNS jsonb
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path TO 'pg_catalog'
AS $$
DECLARE
  v_actor uuid := auth.uid();
  v_total bigint;
  v_rows  jsonb;
BEGIN
  -- 1. identity + role gate (lead or admin)
  IF v_actor IS NULL OR NOT EXISTS (
       SELECT 1 FROM brain_ops.member_roles m
       WHERE m.user_id = v_actor AND m.active AND m.revoked_at IS NULL
         AND m.role IN ('admin','lead')) THEN
    RETURN jsonb_build_object('status','access_denied');
  END IF;

  -- 2. input validation -> explicit statuses, never exceptions for expected cases
  IF p_campaign_id IS NULL THEN RETURN jsonb_build_object('status','invalid_input'); END IF;
  p_limit  := LEAST(GREATEST(coalesce(p_limit,25),1),500);
  p_offset := GREATEST(coalesce(p_offset,0),0);

  -- 3. work (schema-qualified everything)
  SELECT count(*) INTO v_total FROM brain_ops.campaign_links l WHERE l.campaign_id = p_campaign_id;
  SELECT coalesce(jsonb_agg(to_jsonb(x)),'[]'::jsonb) INTO v_rows
  FROM (SELECT l.id AS link_id, l.link_kind, l.destination_url, l.is_active, l.created_at
        FROM brain_ops.campaign_links l
        WHERE l.campaign_id = p_campaign_id
        ORDER BY l.created_at DESC
        LIMIT p_limit OFFSET p_offset) x;

  -- 4. (writers only) audit in the same transaction – see 4.3 for the workstream value
  -- INSERT INTO brain_ops.action_audit(actor_user_id,workstream,tool_name,action,entity_kind,entity_id,result_status,details)
  -- VALUES (v_actor,'campaign','generate_tracking_link','create','campaign_link',v_link_id::text,'completed',jsonb_build_object(...));

  -- 5. envelope
  RETURN jsonb_build_object('status','ok','total',v_total,'rows',v_rows);
END $$;

REVOKE ALL ON FUNCTION public.brain_api_<verb>_<noun>(uuid,int,int) FROM PUBLIC, anon;
GRANT EXECUTE ON FUNCTION public.brain_api_<verb>_<noun>(uuid,int,int) TO authenticated;
```

### 4.3 The `action_audit.workstream` CHECK constraint – look it up first

Commit fb83e12 (2026-11-05):

> every write-RPC built today used workstream='campaigns' (plural) in its action_audit insert, but the
> table's check constraint only allows the singular 'campaign' -- so every one of those functions was
> silently rolling back its entire write the moment it reached that line.

Values proven allowed by the repo: **`'admin'`** (committed migration) and **`'campaign'`** (fix commit).
The complete list is **not** in the repo; before writing a new audit insert run (read-only):

```sql
SELECT conname, pg_get_constraintdef(oid)
FROM pg_constraint
WHERE conrelid = 'brain_ops.action_audit'::regclass AND contype = 'c';
```

and use exactly one of the listed literals. Do the same for `brain_ops.member_roles` before adding a role
(`lead` exists in the app's role list but the committed RPC only inserts `admin`/`researcher`).

### 4.4 Checklist before `apply_migration`

- [ ] Name `public.brain_api_<verb>_<noun>`; parameters prefixed `p_`; defaults for limit/offset.
- [ ] `SECURITY DEFINER` + `SET search_path TO 'pg_catalog'` (or `''`) + all objects schema-qualified.
- [ ] Actor from `auth.uid()`; `IF v_actor IS NULL` -> `access_denied`.
- [ ] Role check against `brain_ops.member_roles` with `active AND revoked_at IS NULL`.
- [ ] Returns `jsonb` with `status`; expected failures are statuses, not `RAISE`.
- [ ] Clamp `p_limit`; include `total` for paged results.
- [ ] Writers: audit insert with a valid `workstream`; verify with a live round trip (save -> read back -> clean up) as fb83e12 did.
- [ ] `REVOKE ALL ... FROM PUBLIC, anon; GRANT EXECUTE ... TO authenticated;` with the **exact argument type list**.
- [ ] If replacing a signature, `DROP FUNCTION` the old one (see pitfalls 6.5).
- [ ] Commit the SQL to `supabase/migrations/` (the 11-05/11-06 functions were applied via MCP only and are not in git – avoid repeating that).

## 5. Client side

### 5.1 Authentication + authorization helpers (`src/lib/brain.server.ts`)

```ts
export async function authenticateBrain(authorization: string | null) {
  const token = authorization?.startsWith("Bearer ") ? authorization.slice(7) : null;
  if (!token || token.split(".").length !== 3) throw new Error("Unauthorized");
  const key = brainKey();                                   // LWL_BRAIN_PUBLISHABLE_KEY, refuses sb_secret_*
  const db = createClient(BRAIN_URL, key, {
    auth: { persistSession: false, autoRefreshToken: false, detectSessionInUrl: false },
    global: { headers: { Authorization: `Bearer ${token}` }, fetch: /* re-sets apikey + Authorization */ },
  });
  const { data, error } = await db.auth.getUser(token);     // JWT must verify against Brain
  if (error || !data.user) throw new Error("Unauthorized");
  return { db, userId: data.user.id, user: data.user, token };
}

export async function brainAccess(context) {
  const { data, error } = await context.db.rpc("brain_api_my_roles");   // no args: auth.uid() inside
  if (error || !Array.isArray(data)) return decideBrainAccess(null);        // -> "unavailable" (fail closed)
  return decideBrainAccess(parseBrainRoles(data));                           // known roles only
}

export async function authorizeBrain(context, section: Section) {
  const access = await brainAccess(context);
  if (access.status === "unavailable") throw new Error("Brain access verification unavailable");
  const role = access.status === "approved" ? brainWorkspaceRole(access.roles) : null;
  if (!role || !can(role, section)) throw new Error("Forbidden");
  return { db: context.db, role };
}
```

`requireBrainAuth` (`src/lib/brain-auth.ts`) is a TanStack `createMiddleware({type:"function"})` that reads the
`authorization` header, calls `authenticateBrain`, sets `Cache-Control: no-store`, and passes `{db,userId,user,token}`
as context. The browser attaches the header automatically through `attachSupabaseAuth` in `src/start.ts`.

### 5.2 Calling an RPC from a server function

```ts
// src/lib/campaigns.functions.ts
type Envelope<T> = { status: string; total?: number; rows?: T[] } & Record<string, unknown>;

async function callRpc<T>(context: BrainContext, fnName: string, args: Record<string, unknown>): Promise<Envelope<T>> {
  const { authorizeBrain } = await import("./brain.server");      // lazy: keeps server-only code out of the client bundle
  const { db } = await authorizeBrain(context, "campaigns");
  const r = await db.rpc(fnName, args);
  if (r.error) throw new Error("Brain reporting data not yet available");   // transport/permission error
  return r.data as Envelope<T>;
}

export const listCampaignLinks = createServerFn({ method: "POST" }).middleware([requireBrainAuth])
  .inputValidator((i) => z.object({ id: z.string().uuid() }).parse(i))
  .handler(async ({ data, context }) => {
    const e = await callRpc<CampaignLinkRow>(context, "brain_api_campaign_links_list",
      { p_campaign_id: data.id, p_limit: 200, p_offset: 0 });
    if (e.status === "access_denied") throw new Error("Forbidden");          // UI shows "Your role doesn't have access"
    return { rows: (e.rows ?? []) as CampaignLinkRow[], total: e.total ?? 0 };
  });
```

Conventions:
- Argument object keys are the SQL parameter names (`p_*`); `null` (not `undefined`) for optional params
  (`data.q || null`) – `undefined` keys are dropped and can change overload resolution.
- `db.rpc(name)` with **no second argument** for zero-arg functions (`brain_api_my_roles`,
  `brain_api_status`, `brain_api_data_health`).
- Lazy `await import("./brain.server")` inside handlers so `@supabase/supabase-js` server code never ships to the browser.

### 5.3 Typing the envelope

Types live in `src/lib/campaigns-data.ts` as discriminated unions keyed on `status`:

```ts
export type SetCampaignSourceResult =
  | { status: "ok"; campaign_id: string; attribution_source_slug: string }
  | { status: "access_denied" | "invalid_source_slug" | "not_found" | "immutable_after_launch" };

export type RpcEnvelope<T> = { status: "ok"; total?: number; rows?: T[] } | { status: "access_denied" | "not_found" };
export function isOk<T>(e: RpcEnvelope<T>): e is { status: "ok"; total?: number; rows?: T[] } { return e.status === "ok"; }
```

For caller-scoped research RPCs the parsers in `src/lib/brain-research.ts` go further: they only accept
`status === "ok"` with every expected numeric field present (`num()`), map `access_denied` to
`{state:"denied"}` and anything else to `{state:"unavailable"}`, and whitelist fields so canonical IDs /
verification flags / emails are dropped before reaching the browser. Unit tests pin these behaviours.

### 5.4 Error-handling contract

| Layer | Condition | Result |
|---|---|---|
| RPC | not signed in / wrong role | `{status:'access_denied'}` (HTTP 200) |
| RPC | bad input / not found / rule violation | `{status:'invalid_input'|'not_found'|'immutable_after_launch'|...}` |
| PostgREST | function not granted / not found / ambiguous | `r.error` (42501 / PGRST202 / PGRST203) |
| server fn | `r.error` | `throw new Error("Brain reporting data not yet available")` |
| server fn | `access_denied` | `throw new Error("Forbidden")` |
| `brainAccess` | `my_roles` errored | `{status:"unavailable"}` -> "Brain access verification unavailable" (never treated as "no roles") |
| UI | `Error("Forbidden")` | `LoadError` renders "Your role doesn't have access to this." |
| UI | any other error | "Couldn't load this right now." – counts are never shown as 0 |

### 5.5 Pagination

- Server fn validates `page` (`z.number().int().min(0).max(1000)`), computes `p_offset = page * PAGE`,
  passes `p_limit = PAGE` (50 for schools/contacts, 25 for campaigns/recipients/coverage, 200 for
  per-campaign lists), and returns `{rows, total, pageSize}`.
- RPC returns `total` (count) alongside `rows`; `Pager` derives page count. Caller-scoped coverage returns
  `has_more` + `offset` instead of total-only (`parseCoverage`).
- Search inputs are length-capped (`q` ≤ 100/200) and the DB side is expected to `ILIKE`; the old client
  `like()` escaper in `data.functions.ts` is a leftover from the `.from()` era and is no longer used.

## 6. Pitfalls (each one happened in this repo)

### 6.1 "Empty data" that is really RLS
RLS on + zero policies returns **no rows and no error** for `.from()` under a user JWT. Commits 8c3f205 /
4f50de0 found Schools/Contacts/Research/Outreach silently empty for every user. Symptom: counts 0, no
console error. Fix: RPC. Rule in this repo: **never call `.from()` on a Brain table from the app**.

### 6.2 Missing `GRANT EXECUTE` looks like an auth bug
PostgREST returns `42501 permission denied for function ...`. The app maps any `my_roles` error to
"verification unavailable" and any other RPC error to "Brain reporting data not yet available". Before
touching policies, run:

```sql
SELECT p.proname, pg_get_function_identity_arguments(p.oid) AS args, p.prosecdef, p.proconfig,
       (SELECT string_agg(a::text, ',') FROM unnest(p.proacl) a) AS acl
FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname = 'public' AND p.proname LIKE 'brain_api_%'
ORDER BY 1;
```

Expect `prosecdef = true`, `proconfig = {search_path=pg_catalog}` and an ACL that includes `authenticated=X/postgres`.

### 6.3 CHECK constraint on `action_audit.workstream` rolls back the whole write
See 4.3. The function does not fail at definition time; it fails at first execution, inside the
transaction, after the "real" write – so the UI shows an error (or, worse, a caught exception) and nothing
persists. Eleven RPCs were regenerated on 11-05 for this.

### 6.4 Immutability trigger on live campaigns
`guard_campaign_tracking_identity_v2` (pre-existing on Brain) makes `attribution_source_slug` immutable once
`status IN ('live','done')`. `brain_api_set_campaign_source` surfaces this as
`{status:'immutable_after_launch'}`; the UI locks the control. Do not "fix" by dropping the trigger – it is
the attribution integrity guarantee. Set sources on `draft`/`paused` campaigns only.

### 6.5 Function overload ambiguity
PostgREST resolves `rpc(name, args)` by argument **names**; if two functions share a name and the supplied
names fit both (e.g. after adding `p_design_json DEFAULT NULL` to `save_email_template` without dropping the
old signature), you get `PGRST203 Could not choose the best candidate function`. Always `DROP FUNCTION
public.brain_api_x(old, types)` in the same migration, and regenerate grants for the new signature
(grants are per-signature).

### 6.6 `RETURNS SETOF`/`TABLE` vs `RETURNS jsonb`
This codebase standardises on **`RETURNS jsonb`** envelopes. Reasons visible in the code: a `status` field
can be returned alongside data without a second round trip; `access_denied` is a 200 (no PostgREST error
masking); nested structures (`campaign`, `reporting`, `npf_jobs`) come back in one call; `total` rides
along with `rows`. Exceptions: `brain_api_my_roles()` and `member_roles()` return `text[]` (the client
checks `Array.isArray(data)`), and the resolver returns a flat object. If you write `RETURNS SETOF`,
`data` is an array and you lose the `status` channel – do not mix the two styles in one feature.

### 6.7 Actor parameters
Older functions take `p_actor uuid`. If you must keep one for compatibility, compare it to `auth.uid()`
inside the function and deny on mismatch; never trust it for authorization. New functions: no `p_actor`.

### 6.8 `search_path`
Without `SET search_path`, a SECURITY DEFINER function resolves unqualified names using the caller's
search path – a classic privilege-escalation vector. Pin it (`'pg_catalog'` or `''`) and qualify every
object. The committed migration does this; copy it.

### 6.9 `STABLE` vs `VOLATILE`
Mark pure readers `STABLE` (lets PostgREST use GET semantics and the planner inline); anything that
inserts into `action_audit` must stay `VOLATILE` (default).

### 6.10 Role drift between app and DB
`src/lib/permissions.ts` knows `researcher | lead | admin`; the committed `admin_set_member_role` only
accepts `admin | researcher`; `ASSIGNABLE_ROLES` in `admin.functions.ts` mirrors the DB. If `lead` is
added to the DB CHECK, update all three – AGENTS.md rule: "assignable roles must exist in both the app
role list and Brain's member_roles constraint".

### 6.11 Uncommitted SQL
Everything created on 11-05/11-06 lives only in Brain. When you need the current definition:

```sql
SELECT pg_get_functiondef('public.brain_api_campaigns_list'::regproc);
```

and paste it into a new migration file before changing it, so the repo regains a history.

### 6.12 Lovable editor binding is not the runtime
`SUPABASE_*` / `VITE_SUPABASE_*` and `src/integrations/supabase/client.ts` are Lovable's binding. Runtime
reads only `LWL_BRAIN_PUBLISHABLE_KEY`. A working editor preview does not prove the deployed app can reach
Brain, and vice-versa.

## 7. Inventory of RPCs following this pattern (as of 2026-11-06)

Called by the app (35): `admin_list_members, admin_set_member_role, campaign_contacts, campaign_detail,
campaign_export_rows, campaign_links_list, campaign_preflight_self, campaign_recipients, campaigns_list,
check_contact_rows, create_campaign, data_health, delete_email_template, generate_tracking_link,
generate_tracking_links_bulk, generate_tracking_links_for_codes, get_instantly_integration, get_school,
import_npf_report, instantly_campaign_summary, instantly_campaigns_list, instantly_unmapped, list_contacts,
list_email_templates, list_research_ops, list_schools, list_tracking_events, map_campaign_job,
my_research_dashboard, my_roles, outreach_overview, research_metrics, resolve_three_utm,
rotate_instantly_webhook_token, save_email_template, save_instantly_api_key, school_coverage_self,
set_campaign_source, set_instantly_workspace, status` (all prefixed `brain_api_`).

Present on Brain but not used by this app (from the 2026-11-03 types snapshot): the research-submission,
review, promotion, queue, import-run and migration families (`submit_research`, `claim_research_school(_v2)`,
`review_research_submission`, `promote_research_*`, `research_queue`, `register_research_file`,
`begin_import_run`, `stage_import_rows`, `ingest_legacy_batch`, `master_migration_status`,
`verify_master_sync`, `operating_rules`, `outreach_check_v1`, `record_funnel_v1`, `revenue_report_v1`,
`ingest_instantly_v1(p_token)`, `create_draft_campaign`, `add_draft_recipient`, `campaign_preflight`,
`historical_school_coverage(_page)`, `member_roles(p_user_id)`), plus non-prefixed helpers
`three_utm_url_v3`, `brain_ops.campaign_preflight_unified_v2`, `brain_ops.sync_outreach_suppressions`,
`npf_server_reconcile_job`, `lwl_auto_review_*`, `lwl_migration_*_v2`.

## 8. Quick reference

```
Name        public.brain_api_<verb>_<noun>(p_... )
Returns     jsonb  { status: 'ok' | 'access_denied' | 'invalid_input' | 'not_found' | <rule>, ...payload }
Header      LANGUAGE plpgsql [STABLE] SECURITY DEFINER SET search_path TO 'pg_catalog'
Actor       v_actor uuid := auth.uid();  NULL -> access_denied
Gate        EXISTS(SELECT 1 FROM brain_ops.member_roles WHERE user_id=v_actor AND active AND revoked_at IS NULL AND role IN (...))
Audit       INSERT INTO brain_ops.action_audit(actor_user_id, workstream, tool_name, action, entity_kind, entity_id, result_status, details)
            workstream must satisfy the CHECK (known: 'admin', 'campaign'); query pg_constraint first
Grants      REVOKE ALL ... FROM PUBLIC, anon;  GRANT EXECUTE ... TO authenticated;   (per exact signature)
Client      createServerFn + requireBrainAuth -> authorizeBrain(context, section) -> db.rpc(name, {p_...}) -> status switch
Never       .from() on Brain tables, service-role keys, p_actor as authority, unpinned search_path, plural 'campaigns'
```
