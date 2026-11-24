# Brain project: user-defined functions

Live snapshot taken 2026-11-24 from <DATA_REF>. Every function/procedure (pg_proc prokind f/p) in the user schemas. Format per function:

`### name(identity args) -> return type` then one metadata line: language | SECURITY DEFINER or invoker | volatility | config (proconfig, e.g. search_path=...) | exec anon=Y/n authenticated=Y/n (has_function_privilege EXECUTE) | body line count; then the body (full if <= 40 lines, otherwise the first 15 lines with a truncation note).

Counts: public 397, group 233, revenue_brain 75, leads 48, migration 11, attribution 4, quarantine 2, brain 0, staging 0 (total 770). Trigger functions, cron bodies, RLS helpers (is_admin_or_owner, is_active_staff, has_grant, has_scope_all, can_see_person, auth_staff_id, research_role, sop_actor) and the migration.sync_* family are all included.

## Summary statistics (computed from this dump)

Total functions parsed: 770

| schema | functions | SECURITY DEFINER | SECURITY DEFINER without search_path set | EXECUTE granted to anon | EXECUTE granted to authenticated | SECURITY DEFINER and anon-executable |
|---|---|---|---|---|---|---|
| attribution | 4 | 0 | 0 | 4 | 4 | 0 |
| leads | 48 | 33 | 0 | 10 | 30 | 1 |
| group | 233 | 168 | 0 | 46 | 136 | 2 |
| migration | 11 | 7 | 0 | 11 | 11 | 7 |
| public | 397 | 291 | 0 | 71 | 180 | 8 |
| quarantine | 2 | 1 | 0 | 2 | 2 | 1 |
| revenue_brain | 75 | 70 | 0 | 9 | 9 | 8 |

### SECURITY DEFINER functions executable by anon (27)

These run with the owner's privileges and can be called by anyone holding the publishable key through PostgREST RPC if their schema is exposed; review each for input validation.

- group.meeting_school_timeline(p_ids uuid[]) -> TABLE(meeting_id uuid, apps_before integer, paid_before integer, apps_after integer, paid_after integer, window_end date)
- group.utm_member_id(p_source text, p_medium text, p_campaign text, p_ref text) -> uuid
- public.research_bump_short_link(_slug text) -> void
- public.research_country_from_domain(p_actor text, p_dry boolean) -> jsonb
- public.research_dup_place_ok(p_field text, p_value text, p_contact uuid) -> boolean
- public.research_role(_user_id uuid) -> text
- public.research_us_state_for(p_domain text, p_school text) -> jsonb
- public.research_us_state_for(p_domain text, p_school text, p_city text) -> jsonb
- public.schools_public(p_system text, p_ingest_key text, p_q text, p_limit integer) -> jsonb
- public.submit_inbound(p_system text, p_ingest_key text, p_kind text, p_payload jsonb, p_idempotency_key text) -> jsonb
- revenue_brain.bind_yale_test_campaign(p_run_id uuid, p_campaign_id text, p_campaign_name text) -> jsonb
- revenue_brain.create_yale_test_run() -> jsonb
- revenue_brain.enforce_instantly_campaign_ingress() -> trigger
- revenue_brain.enforce_production_kill_switch() -> trigger
- revenue_brain.get_yale_sequence() -> jsonb
- revenue_brain.protect_paid_customer_state() -> trigger
- revenue_brain.yale_apply_gate() -> void
- revenue_brain.yale_reconcile() -> jsonb
- leads.get_invitation_by_token(p_token text) -> TABLE(email text, role leads.app_role, full_name text, title text, expires_at timestamp with time zone, accepted boolean)
- migration.sync_applications(p_force boolean) -> TABLE(rows_seen bigint, rows_changed bigint, table_total bigint)
- migration.sync_bank_refunds() -> integer
- migration.sync_dbs_payments() -> TABLE(rows_seen bigint, rows_changed bigint, provider_total bigint)
- migration.sync_programme_codes() -> integer
- migration.sync_razorpay_payments(p_force boolean) -> TABLE(rows_seen bigint, rows_changed bigint, provider_total bigint, flagged_future bigint)
- migration.sync_refunds_from_payments() -> integer
- migration.sync_stripe_payments(p_force boolean) -> TABLE(rows_seen bigint, rows_changed bigint, provider_total bigint)
- quarantine.refresh_research_schools() -> integer

### SECURITY DEFINER functions without a fixed search_path (0)



## Notable functions index (83; full bodies in the per-schema sections below)

| function | returns | lang | security | search_path | anon | authenticated | body lines |
|---|---|---|---|---|---|---|---|
| leads.deals_flag_price_conflict() | trigger | plpgsql | invoker | leads, public | Y | Y | 16 |
| leads.deals_stage_stamp() | trigger | plpgsql | invoker | leads, public | Y | Y | 80 |
| leads.is_admin_or_owner(_user_id uuid) | boolean | sql | DEFINER | leads, public, extensions | n | Y | 6 |
| leads.is_assigned_to_meeting(_meeting_id uuid, _user_id uuid) | boolean | sql | DEFINER | leads, public, extensions | n | Y | 6 |
| leads.sop_actor() | uuid | sql | DEFINER | leads, public | n | Y | 3 |
| leads.sop_sweep() | jsonb | plpgsql | DEFINER | leads, public | n | n | 213 |
| leads.sync_lead_segment_from_designation() | trigger | plpgsql | invoker | leads, public, extensions | Y | Y | 9 |
| leads.sync_lifecycle_from_status() | trigger | plpgsql | invoker | leads, public, extensions | Y | Y | 22 |
| group.application_school_matcher_dispatch(p_body jsonb) | void | plpgsql | DEFINER | group, pg_temp | n | n | 9 |
| group.auto_link_application_schools(p_limit integer) | jsonb | plpgsql | DEFINER | group, public, extensions, pg_temp | n | n | 59 |
| group.automations_apply_cron() | trigger | plpgsql | DEFINER | group, public, extensions, pg_temp | n | n | 18 |
| group.deals_flag_price_conflict() | trigger | plpgsql | invoker | group, public, extensions, pg_temp | Y | Y | 12 |
| group.deals_stage_stamp() | trigger | plpgsql | invoker | group, public, extensions, pg_temp | Y | Y | 54 |
| group.handle_new_user_if_invited() | trigger | plpgsql | DEFINER | group, public | n | Y | 14 |
| group.is_active_staff(_uid uuid) | boolean | sql | DEFINER | group, public, extensions, pg_temp | n | Y | 8 |
| group.is_admin_or_owner(_user_id uuid) | boolean | sql | DEFINER | group, public, extensions, pg_temp | n | Y | 6 |
| group.is_assigned_to_meeting(_meeting_id uuid, _user_id uuid) | boolean | sql | DEFINER | group, public, extensions, pg_temp | n | Y | 6 |
| group.meeting_manual_lock() | trigger | plpgsql | DEFINER | group, public, extensions, pg_temp | n | Y | 39 |
| group.member_daily_email_dispatch_all(p_body jsonb) | integer | plpgsql | DEFINER | group, pg_temp | n | n | 12 |
| group.member_email_agent_dispatch(p_body jsonb) | void | plpgsql | DEFINER | group, pg_temp | n | n | 9 |
| group.member_email_agent_dispatch_all(p_body jsonb) | integer | plpgsql | DEFINER | group, pg_temp | n | n | 9 |
| group.my_assigned_meeting_ids() | SETOF uuid | sql | DEFINER | group, public, extensions, pg_temp | n | Y | 3 |
| group.pull_bank_refunds_from_core() | integer | plpgsql | invoker | group, public, extensions, pg_temp | Y | Y | 18 |
| group.pull_from_core(p_full boolean) | TABLE(apps_pulled bigint, pays_pulled bigint, lwl_updated bigint, schools_resolved bigint) | plpgsql | invoker | group, public, extensions, pg_temp | n | n | 74 |
| group.release_recovery_backlog(p_limit integer) | integer | plpgsql | DEFINER | group, public, extensions, pg_temp | n | n | 26 |
| group.resolve_school_codes() | TABLE(meetings_resolved integer, apps_resolved integer) | plpgsql | DEFINER | group, public, extensions, pg_temp | n | n | 20 |
| group.run_health_checks() | integer | plpgsql | DEFINER | group, public, extensions, pg_temp, cron | n | n | 112 |
| group.sop_actor() | uuid | sql | DEFINER | group, public, extensions, pg_temp | n | Y | 3 |
| group.sop_sweep() | jsonb | plpgsql | DEFINER | group, public, extensions, pg_temp | n | n | 102 |
| group.sop_sweep_all() | jsonb | sql | DEFINER | group, public, extensions, pg_temp | n | n | 3 |
| group.sop_sweep_tracks() | jsonb | plpgsql | DEFINER | group, public, extensions, pg_temp | n | n | 86 |
| group.sync_lead_segment_from_designation() | trigger | plpgsql | invoker | group, public, extensions, pg_temp | n | n | 9 |
| group.sync_lead_type_from_designation() | trigger | plpgsql | invoker | group, public, extensions, pg_temp | n | n | 21 |
| group.sync_lifecycle_from_status() | trigger | plpgsql | invoker | group, public, extensions, pg_temp | n | n | 22 |
| group.sync_meeting_assigned_to(_meeting_id uuid) | void | plpgsql | DEFINER | group, public, extensions, pg_temp | n | n | 24 |
| group.sync_meeting_attribution() | integer | plpgsql | DEFINER | group, public, extensions, pg_temp | n | n | 67 |
| group.welcome_bot_expire_stale() | integer | plpgsql | DEFINER | group, public, extensions, pg_temp | n | n | 20 |
| migration.sync_applications(p_force boolean) | TABLE(rows_seen bigint, rows_changed bigint, table_total bigint) | plpgsql | DEFINER | public, migration, legacy_ro | Y | Y | 67 |
| migration.sync_bank_refunds() | integer | plpgsql | DEFINER | public, migration, legacy_ro | Y | Y | 51 |
| migration.sync_dbs_payments() | TABLE(rows_seen bigint, rows_changed bigint, provider_total bigint) | plpgsql | DEFINER | public, migration, legacy_ro | Y | Y | 58 |
| migration.sync_programme_codes() | integer | plpgsql | DEFINER | public, migration, legacy_ro | Y | Y | 30 |
| migration.sync_razorpay_payments(p_force boolean) | TABLE(rows_seen bigint, rows_changed bigint, provider_total bigint, flagged_future bigint) | plpgsql | DEFINER | public, migration, legacy_ro | Y | Y | 81 |
| migration.sync_refunds_from_payments() | integer | sql | DEFINER | public, migration | Y | Y | 14 |
| migration.sync_stripe_payments(p_force boolean) | TABLE(rows_seen bigint, rows_changed bigint, provider_total bigint) | plpgsql | DEFINER | public, migration, legacy_ro | Y | Y | 81 |
| public.accept_staff_invitation() | trigger | plpgsql | DEFINER | public | n | n | 73 |
| public.auth_staff_id() | uuid | sql | DEFINER | public | n | Y | 6 |
| public.can_see_person(p_person_id uuid) | boolean | sql | DEFINER | public | n | Y | 28 |
| public.contact_ownership(p_email text) | jsonb | plpgsql | DEFINER | public, quarantine, extensions, pg_temp | n | Y | 8 |
| public.contact_ownership_body(p_email text, p_actor uuid) | jsonb | sql | invoker | public, quarantine, extensions, pg_temp | n | n | 109 |
| public.email_sendability(p_email text) | jsonb | plpgsql | DEFINER | public, quarantine, pg_temp | n | Y | 5 |
| public.geo_apply(p_limit integer) | jsonb | plpgsql | DEFINER | public, quarantine, pg_temp | n | Y | 35 |
| public.geo_propose_body(p_by text) | jsonb | sql | invoker | public, quarantine, pg_temp | n | n | 118 |
| public.guard_source_code() | trigger | plpgsql | invoker | public | n | n | 11 |
| public.has_grant(g staff_grant) | boolean | sql | DEFINER | public | n | Y | 6 |
| public.has_scope_all() | boolean | sql | DEFINER | public | n | Y | 4 |
| public.meeting_dataset_pairs() | TABLE(o_meeting_id uuid, o_dataset_id uuid, o_contact_id uuid, o_code text, o_built_by uuid, o_why text) | sql | invoker | public, quarantine, pg_temp | n | n | 27 |
| public.org_classify_body(p_by text) | jsonb | sql | invoker | public, quarantine, pg_temp | n | n | 121 |
| public.org_groups_seed() | jsonb | plpgsql | DEFINER | public, quarantine, pg_temp | n | Y | 15 |
| public.org_promote_candidates(p_limit integer) | jsonb | plpgsql | DEFINER | public, quarantine, pg_temp | n | Y | 88 |
| public.promote_research_contact(p_contact_id uuid) | jsonb | plpgsql | DEFINER | public, quarantine, pg_temp | n | Y | 84 |
| public.promote_research_contacts(p_country text, p_state text, p_city text, p_limit integer) | jsonb | plpgsql | DEFINER | public, quarantine, pg_temp | n | Y | 31 |
| public.promote_research_school(p_name_key text, p_note text) | jsonb | plpgsql | DEFINER | public, quarantine, pg_temp | n | Y | 25 |
| public.promote_school(p_candidate_id uuid, p_note text) | jsonb | plpgsql | DEFINER | public, quarantine, pg_temp | n | Y | 43 |
| public.provider_tick() | bigint | plpgsql | invoker | public, pg_temp | n | n | 6 |
| public.research_auto_merge_duplicates(p_limit integer, p_actor text) | jsonb | plpgsql | DEFINER | public | n | n | 24 |
| public.research_contrib_tick(p_pieces integer) | jsonb | plpgsql | DEFINER | public | n | n | 23 |
| public.research_identity_hygiene_cron(p_actor text, p_link_limit integer) | jsonb | plpgsql | DEFINER | public | n | n | 36 |
| public.research_library_tick(p_slices integer) | jsonb | plpgsql | DEFINER | public | n | n | 20 |
| public.research_link_by_domain(p_limit integer) | jsonb | plpgsql | DEFINER | public, quarantine, pg_temp | n | Y | 42 |
| public.research_refresh_all_contributions() | integer | plpgsql | DEFINER | public | n | n | 9 |
| public.research_refresh_contrib_chunk(p_tab uuid, p_chunk integer) | integer | plpgsql | DEFINER | public | n | n | 63 |
| public.research_refresh_contributions(p_tab uuid) | integer | plpgsql | DEFINER | public | n | n | 58 |
| public.research_refresh_domain_index() | jsonb | plpgsql | DEFINER | public | n | n | 29 |
| public.research_refresh_email_check_stats() | void | sql | DEFINER | public | n | n | 13 |
| public.research_refresh_folder_slices(p_folder uuid, p_from integer, p_to integer) | integer | plpgsql | DEFINER | public | n | n | 9 |
| public.research_refresh_folders_slice(p_prefix text) | integer | plpgsql | DEFINER | public | n | n | 46 |
| public.research_refresh_health_slice(p_prefix text) | integer | plpgsql | DEFINER | public | n | n | 62 |
| public.research_refresh_trust_totals() | integer | plpgsql | DEFINER | public | n | n | 16 |
| public.research_retry_tick(p_limit integer, p_base_minutes integer, p_cap_minutes integer) | jsonb | plpgsql | DEFINER | public | n | n | 111 |
| public.research_review_autoclean() | jsonb | plpgsql | DEFINER | public | n | n | 7 |
| public.research_role(_user_id uuid) | text | sql | DEFINER | public | Y | Y | 3 |
| public.validator_tick(p_stage text, p_limit integer) | bigint | plpgsql | invoker | public, quarantine, pg_temp | n | n | 19 |
| quarantine.refresh_research_schools() | integer | plpgsql | DEFINER | quarantine, public, pg_temp | Y | Y | 55 |


## Schema: group (233 functions) - part 1 of 2

### _school_acronym_run(tok text, arr text[]) -> boolean
- plpgsql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 15 lines
```sql

declare i int; j int; acr text; n int;
begin
  if tok is null or length(tok) < 3 or arr is null then return false; end if;
  n := coalesce(array_length(arr,1),0);
  for i in 1..n loop
    acr := '';
    for j in i..n loop
      acr := acr || left(arr[j],1);
      if acr = tok and (j - i + 1) >= 2 then return true; end if;
      if length(acr) > length(tok) then exit; end if;
    end loop;
  end loop;
  return false;
end 
```

### _school_generic_tokens() -> text[]
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 7 lines
```sql
 select array[
  'the','of','and','a','school','schools','academy','college','university','institute',
  'international','public','private','high','higher','senior','secondary','sec','primary',
  'elementary','middle','district','education','educational','educacional','educativo',
  'campus','centre','center','trust','society','group','global','world','national','city',
  'branch','co','ed','coed','convent','matric','matriculation','cbse','icse','ib','vidyalaya'
]::text[] 
```

### _school_tok_match(tok text, arr text[]) -> boolean
- plpgsql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 25 lines
```sql

declare i int; j int; acr text; n int;
begin
  if tok is null or tok = '' or arr is null then return false; end if;
  if tok = any(arr) then return true; end if;
  if length(tok) >= 5 then
    if exists (
      select 1 from unnest(arr) x
      where (x like tok || '%' or tok like x || '%') and abs(length(x) - length(tok)) <= 3
    ) then return true; end if;
  end if;
  n := coalesce(array_length(arr,1),0);
  -- initialisms only from 3 letters up: "CS" is far too weak to identify a school
  if length(tok) between 3 and 8 then
    for i in 1..n loop
      acr := '';
      for j in i..n loop
        acr := acr || left(arr[j],1);
        if acr = tok then return true; end if;
        if length(acr) > length(tok) then exit; end if;
      end loop;
    end loop;
  end if;
  return false;
end 
```

### activity_detail(p_activity_id text) -> TABLE(activity_id text, occurred_at timestamp with time zone, kind text, actor_type text, actor_name text, title text, detail text, meeting_id uuid, lead_email text, school_code text, source_table text, source_id text, payload jsonb)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

begin
  if not group.is_active_staff(auth.uid()) then raise exception 'not authorised'; end if;
  return query select * from group.activity_detail__impl(p_activity_id);
end 
```

### activity_detail__impl(p_activity_id text) -> TABLE(activity_id text, occurred_at timestamp with time zone, kind text, actor_type text, actor_name text, title text, detail text, meeting_id uuid, lead_email text, school_code text, source_table text, source_id text, payload jsonb)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 8 lines
```sql

  select a.activity_id, a.occurred_at, a.kind, a.actor_type, a.actor_name,
         a.title, a.detail, a.meeting_id, a.lead_email, a.school_code,
         a.source_table, a.source_id::text, a.payload
    from group.v_activity a
   where a.activity_id = p_activity_id
   limit 1;

```

### ai_set_enabled(p_on boolean, p_note text) -> boolean
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 14 lines
```sql

begin
  if auth.uid() is not null and not group.is_admin_or_owner(auth.uid()) then
    raise exception 'Only an owner or admin can switch AI on or off';
  end if;
  insert into group.ai_settings (id, enabled, note, updated_by, updated_at)
  values (true, p_on, p_note, auth.uid(), now())
  on conflict (id) do update
    set enabled = excluded.enabled,
        note = coalesce(excluded.note, group.ai_settings.note),
        updated_by = excluded.updated_by,
        updated_at = now();
  return p_on;
end 
```

### application_school_matcher_dispatch(p_body jsonb) -> void
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 9 lines
```sql

declare cmd text;
begin
  select command into cmd from cron.job where jobname='group-member-sheet-report' limit 1;
  if cmd is null then raise exception 'Scheduler credential not found'; end if;
  cmd := replace(cmd, '/functions/v1/member-sheet-report', '/functions/v1/application-school-matcher');
  cmd := replace(cmd, '), timeout_milliseconds', ') || ' || quote_literal(p_body::text) || '::jsonb, timeout_milliseconds');
  execute cmd;
end 
```

### apply_trusted_application_schools() -> integer
- plpgsql | invoker | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 28 lines
```sql

declare n integer;
begin
  with pending as (
    select p.id,
           regexp_replace(regexp_replace(lower(btrim(p.school_raw)), '[.,\-/\\#!$%^&*;:{}=_~()]+', ' ', 'g'), '\s+', ' ', 'g') as norm,
           group.conversion_school_key(p.school_raw) as k
      from group.core_applications p
     where p.school_code is null and nullif(btrim(p.school_raw), '') is not null
  ), cand as (
    select pending.id, a.school_code
      from pending join group.school_aliases a on lower(btrim(a.raw_text)) = pending.norm or a.raw_text = pending.k
     where a.school_code is not null and a.match_confidence = 1
       and a.match_source = any (array['seed','bulk_import','registry_exact_norm','domain','sm_20261003','human','founder_ai'])
    union all
    select pending.id, l.school_code
      from pending cross join lateral (
        select m.school_code, m.method from group.application_school_matches m
         where m.name_key = pending.k order by m.created_at desc limit 1) l
     where l.school_code is not null and l.method in ('match_ai','new_school','email_domain','human','founder_ai','founder_ai_new')
  ), one as (
    select id, min(school_code) as sc from cand group by id having count(distinct school_code) = 1
  )
  update group.core_applications c set school_code = one.sc
    from one where c.id = one.id and c.school_code is null;
  get diagnostics n = row_count;
  return n;
end 
```

### attendance_review_queue(p_from date, p_limit integer) -> TABLE(meeting_id uuid, meeting_date date, scheduled_time time without time zone, full_name text, school_name text, taker text, strength text, matched_words text, guests text, guest_minutes integer, summary text)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 44 lines
```sql

  with staff as (
    select distinct lower(split_part(btrim(full_name), ' ', 1)) fn from group.user_profiles where full_name is not null
    union select unnest(array['learn','leaders','zoom','team'])
  ),
  ns as (
    select m.id, m.meeting_date, m.scheduled_time, m.full_name, m.school_name, m.zoom_ai_summary s
      from group.meetings m
     where m.meeting_status = 'No Show' and m.meeting_date >= p_from
       and not (m.manual_overrides ? 'meeting_status')
       and coalesce(btrim(m.zoom_ai_summary), '') <> ''
       and not (m.zoom_ai_summary ~* '^\s*Meeting Summary for .{0,120}: \d+ ?Minute' and length(m.zoom_ai_summary) < 160)
  ),
  words as (
    select ns.id, string_agg(distinct w, ', ') filter (where lower(ns.s) ~ ('\m' || w || '\M')) hit
-- ... truncated (44 lines total)
```

### attribution_report(p_day date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp; statement_timeout=120s | exec anon=n authenticated=n | 70 lines
```sql

DECLARE
  t0 timestamptz := (p_day::timestamp AT TIME ZONE 'Asia/Kolkata');
  t1 timestamptz := ((p_day + 1)::timestamp AT TIME ZONE 'Asia/Kolkata');
  res jsonb;
BEGIN
  WITH apps AS (
    SELECT a.id, a.applied_at, a.paid_at, a.programme_code, a.student_name, a.school_raw, a.country,
      lower(nullif(trim(a.student_email),'')) se, lower(nullif(trim(a.parent_email),'')) pe,
      coalesce(a.school_code, (SELECT sa.school_code FROM school_aliases sa
        WHERE lower(sa.raw_text) = lower(trim(a.school_raw)) ORDER BY sa.match_confidence DESC NULLS LAST LIMIT 1)) sc
    FROM core_applications a WHERE a.applied_at >= t0 AND a.applied_at < t1
  ),
  schools AS (SELECT sc, max(applied_at) cutoff FROM apps WHERE sc IS NOT NULL GROUP BY sc),
  direct AS (
-- ... truncated (70 lines total)
```

### attribution_report_admin(p_day date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp; statement_timeout=120s | exec anon=n authenticated=Y | 8 lines
```sql

BEGIN
  IF NOT (group.has_role(auth.uid(), 'owner') OR group.has_role(auth.uid(), 'admin')) THEN
    RAISE EXCEPTION 'forbidden';
  END IF;
  RETURN group.attribution_report(p_day);
END

```

### auto_confirm_school_matches() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 48 lines
```sql

DECLARE
  v_meetings_reviewed INT := 0;
  v_apps_reviewed     INT := 0;
  v_schools_approved  INT := 0;
BEGIN
  -- Meetings: auto-confirm any match that is either
  --   (a) an AI/alias match at >= 85% confidence, or
  --   (b) an AI auto-created school (no ambiguity — we made the row ourselves)
  UPDATE group.meetings
     SET school_match_reviewed = true
   WHERE school_code IS NOT NULL
     AND school_match_reviewed = false
     AND (
       school_match_source = 'ai_auto_create'
-- ... truncated (48 lines total)
```

### auto_link_application_schools(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp; statement_timeout=240s | exec anon=n authenticated=n | 59 lines
```sql

declare v_items jsonb; v_res jsonb;
begin
  if not pg_try_advisory_xact_lock(hashtext('group.auto_link_application_schools')) then
    return jsonb_build_object('skipped', 'already running');
  end if;
  with names as (
    select x.k, min(c.school_raw) raw, mode() within group (order by c.school_raw) best_raw,
           group.norm_school_country(mode() within group (order by c.country)) country,
           mode() within group (order by c.city) city, count(*) n
      from group.core_applications c cross join lateral (select btrim(group.conversion_school_key(c.school_raw)) k) x
     where c.school_code is null and nullif(btrim(c.school_raw),'') is not null and coalesce(x.k,'') <> ''
       and not exists (select 1 from group.application_school_matches m where m.name_key = x.k
                        and m.method = 'not_identifiable' and m.created_at > now() - interval '30 days')
     group by x.k order by count(*) desc limit p_limit
-- ... truncated (59 lines total)
```

### automations_apply_cron() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 18 lines
```sql

begin
  if new.state is not distinct from old.state then
    return new;
  end if;

  if new.slug = 'welcome_greeter' then
    perform cron.alter_job(jobid, active := (new.state = 'on'))
      from cron.job where jobname in ('welcome-bot-dispatch-minutely','welcome-bot-expire',
                                      'group-welcome-bot-dispatch-minutely','group-welcome-bot-expire');
  elsif new.slug = 'new_meeting_claim' then
    perform cron.alter_job(jobid, active := (new.state = 'on'))
      from cron.job where jobname in ('wa-new-meeting-claim-every-5-min','wa-claim-poll-every-5-min',
                                      'group-wa-new-meeting-claim-every-5-min','group-wa-claim-poll-every-5-min');
  end if;

  return new;
end 
```

### bump_zoom_backfill(_id uuid, _increment boolean, _note text) -> void
- sql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 7 lines
```sql

  UPDATE group.meetings
     SET zoom_backfill_attempts = zoom_backfill_attempts + CASE WHEN _increment THEN 1 ELSE 0 END,
         zoom_backfill_last_at  = now(),
         zoom_backfill_note     = _note
   WHERE id = _id;

```

### can_work_deal(p_deal_id uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=group, public, pg_temp | exec anon=n authenticated=n | 5 lines
```sql

  select group.is_admin_or_owner(auth.uid())
      or exists (select 1 from group.deals d where d.id = p_deal_id
                 and (d.owner = auth.uid() or (d.owner is null and group.is_active_staff(auth.uid()))))

```

### claim_meeting(_meeting_id uuid) -> void
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 46 lines
```sql

DECLARE
  v_uid uuid := auth.uid();
  v_who text;
  v_name text;
BEGIN
  IF v_uid IS NULL THEN RAISE EXCEPTION 'Not signed in'; END IF;
  IF NOT group.is_active_staff(v_uid) THEN
    RAISE EXCEPTION 'Only active team members can take meetings';
  END IF;
  IF group.meeting_has_assignee(_meeting_id) THEN
    RAISE EXCEPTION 'Someone has already taken this meeting';
  END IF;

  INSERT INTO group.meeting_assignees (meeting_id, user_id, assigned_by)
-- ... truncated (46 lines total)
```

### classify_lead_group(_designation text, _email text, _school_name text, _school_code text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 18 lines
```sql

  SELECT CASE
    WHEN coalesce(btrim(lower(_designation)), '') IN ('', '.', '-', '--', 'n/a', 'na', 'none', 'nil', 'group', 'other', 'test')
      THEN CASE
        WHEN coalesce(_email,'') ~ '@'
         AND lower(split_part(_email,'@',2)) !~ '(gmail|yahoo|outlook|hotmail|icloud|live\.|proton|aol|rediff|ymail|qq\.com|163\.com|me\.com|mail\.com|zoho)'
        THEN 'school'
        ELSE 'unknown'
      END
    WHEN lower(_designation) ~ '(universit|professor|provost|chancellor)' THEN 'university'
    WHEN lower(_designation) ~ '(\mcorporate\M|\mcompany\M|\mceo\M|\mcto\M|\mcoo\M|\mcfo\M|\mhr\M|human resource|recruit|business development)' THEN 'corporate'
    WHEN lower(_designation) ~ '(\mpartner\M|\magent\M|consultant|franchis|reseller)' THEN 'partner'
    WHEN lower(_designation) ~ '(school|principal|representative|coordinator|teacher|head|dean|admin|counsel|leader|manager|director|founder|owner|headmistress|headmaster|educator|faculty|academic|superintendent)' THEN 'school'
    WHEN lower(_designation) ~ '(parent|guardian|mother|father)' THEN 'parent'
    WHEN lower(_designation) ~ '(student|learner|pupil|alumn)' THEN 'student'
    ELSE 'other'
  END

```

### clear_score_on_non_completion() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 11 lines
```sql

BEGIN
  IF NEW.meeting_status IN ('No Show','Cancelled')
     AND (OLD.meeting_status IS DISTINCT FROM NEW.meeting_status
          OR TG_OP = 'INSERT') THEN
    NEW.lead_score := NULL;
    NEW.lead_temperature := NULL;
  END IF;
  RETURN NEW;
END;

```

### confirm_zoom_takers() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 42 lines
```sql

DECLARE
  n integer;
BEGIN
  -- Same fault as release_meeting: called with no argument, so it failed for
  -- everyone rather than only for the unauthorised.
  IF NOT group.is_admin_or_owner(auth.uid()) THEN
    RAISE EXCEPTION 'not authorised';
  END IF;

  WITH cand AS (
    SELECT m.id AS meeting_id,
           (p->>'user_id')::uuid AS user_id,
           sum((p->>'duration')::int) AS secs
    FROM group.meetings m,
-- ... truncated (42 lines total)
```

### conversion_evidence(p_from date, p_to date, p_member uuid, p_basis text, p_offset integer, p_limit integer) -> SETOF group.v_conversion_evidence
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 9 lines
```sql
 BEGIN
 IF NOT group.is_admin_or_owner(auth.uid()) THEN RAISE EXCEPTION 'Not authorised' USING ERRCODE='42501'; END IF;
 IF p_basis NOT IN ('meeting','received','utm') THEN RAISE EXCEPTION 'Invalid reporting basis'; END IF;
 RETURN QUERY SELECT e.* FROM group.v_conversion_evidence e WHERE (p_member IS NULL OR e.credited_user_id=p_member) AND
 CASE WHEN p_basis='received' THEN (e.applied_at>=p_from::timestamp AT TIME ZONE 'Asia/Kolkata' AND e.applied_at<(p_to+1)::timestamp AT TIME ZONE 'Asia/Kolkata') OR (e.paid_at>=p_from::timestamp AT TIME ZONE 'Asia/Kolkata' AND e.paid_at<(p_to+1)::timestamp AT TIME ZONE 'Asia/Kolkata')
 WHEN p_basis='utm' THEN e.utm_member_id IS NOT NULL AND ((e.applied_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to OR (e.paid_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to)
 ELSE (e.meeting_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to AND e.credit_reason<>'utm_member' END
 ORDER BY e.applied_at DESC,e.application_id OFFSET greatest(p_offset,0) LIMIT least(greatest(p_limit,1),500);
 END 
```

### conversion_future_payment_review() -> TABLE(payment_id uuid, provider_payment_id text, payer_name text, occurred_at timestamp with time zone, currency text, amount_minor bigint)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 1 lines
```sql
 BEGIN IF NOT group.is_admin_or_owner(auth.uid()) THEN RAISE EXCEPTION 'Not authorised'; END IF; RETURN QUERY SELECT p.id,p.provider_payment_id,p.payer_name,p.occurred_at,p.currency,p.amount_minor FROM group.core_payments p WHERE p.occurred_at>now() AND p.status IN ('paid','partially_refunded') ORDER BY p.occurred_at,p.id LIMIT 500; END 
```

### conversion_meeting_counts(p_ids uuid[]) -> TABLE(meeting_id uuid, apps_after integer, paid_after integer, first_app_at timestamp with time zone, first_paid_at timestamp with time zone)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 3 lines
```sql
 BEGIN
 IF NOT group.is_admin_or_owner(auth.uid()) THEN RAISE EXCEPTION 'Not authorised' USING ERRCODE='42501'; END IF;
 RETURN QUERY SELECT e.meeting_id,count(*) FILTER(WHERE e.application_after_meeting)::integer,count(*) FILTER(WHERE e.paid_after_meeting)::integer,min(e.applied_at) FILTER(WHERE e.application_after_meeting),min(e.paid_at) FILTER(WHERE e.paid_after_meeting) FROM group.v_conversion_evidence e WHERE e.meeting_id=ANY(p_ids) AND e.credit_reason<>'utm_member' GROUP BY e.meeting_id; END 
```

### conversion_member_profile(p_member uuid) -> SETOF group.v_member_performance
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 1 lines
```sql
 BEGIN IF NOT group.is_admin_or_owner(auth.uid()) THEN RAISE EXCEPTION 'Not authorised'; END IF; RETURN QUERY SELECT * FROM group.v_member_performance WHERE user_id=p_member; END 
```

### conversion_reconciliation(p_from date, p_to date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql
 DECLARE result jsonb; BEGIN
 IF NOT group.is_admin_or_owner(auth.uid()) THEN RAISE EXCEPTION 'Not authorised' USING ERRCODE='42501'; END IF;
 WITH e AS MATERIALIZED (SELECT * FROM group.v_conversion_evidence WHERE (applied_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to OR (paid_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to), a AS (SELECT * FROM e WHERE (applied_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to), members AS (SELECT e.credited_user_id user_id,coalesce(p.full_name,'Unattributed') member_name,count(*) FILTER(WHERE (e.applied_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to) applications,count(*) FILTER(WHERE (e.paid_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to) paid FROM e LEFT JOIN group.user_profiles p ON p.id=e.credited_user_id GROUP BY 1,2), campaigns AS (SELECT utm_source,utm_medium,utm_campaign,count(*) applications,count(*) FILTER(WHERE is_paid) paid FROM a GROUP BY 1,2,3)
 SELECT jsonb_build_object('applications',(SELECT count(*) FROM a),'credited',(SELECT count(*) FROM a WHERE credited_user_id IS NOT NULL),'unattributed',(SELECT count(*) FROM a WHERE credited_user_id IS NULL),'missing_school',(SELECT count(*) FROM a WHERE school_code IS NULL),'missing_utm',(SELECT count(*) FROM a WHERE missing_utm),'paid',(SELECT count(*) FROM e WHERE (paid_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to),'payment_exceptions',(SELECT count(*) FROM e WHERE is_paid AND payment_evidence<>'matched'),'future_payments',(SELECT count(*) FROM group.core_payments WHERE occurred_at>now() AND status IN ('paid','partially_refunded')),'latest_pull',(SELECT max(pulled_at) FROM group.core_applications),'members',coalesce((SELECT jsonb_agg(m ORDER BY applications DESC) FROM members m),'[]'::jsonb),'campaigns',coalesce((SELECT jsonb_agg(c ORDER BY applications DESC) FROM campaigns c),'[]'::jsonb)) INTO result;
 RETURN result; END 
```

### conversion_school_key(p_raw text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 1 lines
```sql
 SELECT regexp_replace(regexp_replace(lower(btrim(p_raw)), '[.,\-/\\#!$%^&*;:{}=_~()]+', ' ', 'g'), '\s+', ' ', 'g') 
```

### core_applications_set_utm_member() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 5 lines
```sql

begin
  new.utm_member_id := group.utm_member_id(new.utm_source, new.utm_medium, new.utm_campaign, new.referral_source);
  return new;
end 
```

### create_manual_meeting(p_full_name text, p_email text, p_meeting_date date, p_scheduled_time time without time zone, p_duration_minutes integer, p_meeting_link text, p_school_name text, p_designation text, p_phone text, p_country text, p_city text, p_query text, p_assignee uuid) -> uuid
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 39 lines
```sql

declare
  caller uuid := auth.uid(); v_id uuid; v_start timestamptz; v_zoom text; v_locks jsonb;
begin
  if caller is null or not group.is_admin_or_owner(caller) then
    raise exception 'Only an admin can add a meeting by hand';
  end if;
  if nullif(btrim(p_full_name), '') is null then raise exception 'Name is required'; end if;
  if p_meeting_date is null or p_scheduled_time is null then raise exception 'Date and time are required'; end if;

  v_start := (p_meeting_date + p_scheduled_time) at time zone 'Asia/Kolkata';
  v_zoom := (regexp_match(coalesce(p_meeting_link, ''), '/j/(\d{9,12})'))[1];
  select jsonb_object_agg(f, jsonb_build_object('by', caller, 'at', now(), 'reason', 'created by hand'))
    into v_locks
    from unnest(array['meeting_date','scheduled_time','start_time_utc','end_time_utc','meeting_link']
                || case when v_zoom is not null then array['zoom_meeting_id'] else array[]::text[] end) f;

  insert into group.meetings (
    full_name, email, school_name, designation, phone, country, city, query,
    meeting_date, scheduled_time, start_time_utc, end_time_utc, duration_minutes,
    meeting_link, zoom_meeting_id, meeting_status, meeting_type, host_source, manual_overrides)
  values (
    btrim(p_full_name), lower(nullif(btrim(p_email), '')), nullif(btrim(p_school_name), ''),
    nullif(btrim(p_designation), ''), nullif(btrim(p_phone), ''), nullif(btrim(p_country), ''),
    nullif(btrim(p_city), ''), nullif(btrim(p_query), ''),
    p_meeting_date, p_scheduled_time, v_start, v_start + make_interval(mins => coalesce(p_duration_minutes, 30)),
    coalesce(p_duration_minutes, 30), nullif(btrim(p_meeting_link), ''), v_zoom,
    'Scheduled', 'New Lead', 'manual', v_locks)
  returning id into v_id;

  if p_assignee is not null then
    insert into group.meeting_assignees (meeting_id, user_id, assigned_by)
    values (v_id, p_assignee, caller::text) on conflict do nothing;
  end if;

  insert into group.meeting_edit_log (meeting_id, field, old_value, new_value, changed_by)
  values (v_id, 'created', null, 'Added by hand (not from Calendly)', caller);
  return v_id;
end 
```

### create_stale_followup_tasks(_days integer) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 60 lines
```sql

DECLARE
  v_count integer := 0;
BEGIN
  WITH last_out AS (
    SELECT es.meeting_id,
           max(COALESCE(es.sent_at, es.created_at)) AS last_sent
      FROM group.email_sends es
     WHERE es.direction = 'outbound'
       AND es.status = 'sent'
       AND es.meeting_id IS NOT NULL
     GROUP BY es.meeting_id
  ),
  candidates AS (
    SELECT lo.meeting_id, lo.last_sent, m.full_name, m.school_name
-- ... truncated (60 lines total)
```

### deal_advance_stage(p_deal_id uuid, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 42 lines
```sql

declare d group.deals; blockers text[]; nxt smallint;
begin
  if group.sop_actor() is null then raise exception 'not authorised'; end if;
  if auth.uid() is not null and not group.can_work_deal(p_deal_id) then raise exception 'You do not own this record'; end if;
  select * into d from group.deals where id = p_deal_id;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;
  nxt := least(coalesce(d.stage,0) + 1, 7);
  if nxt = d.stage then raise exception 'This record is already at the final stage'; end if;

  select coalesce(array_agg(reason), '{}') into blockers from (
    select 'A proposal must be sent and dated' reason
      where nxt >= 2 and d.proposal_sent_date is null
    union all select 'A primary stream must be tagged'
      where nxt >= 2 and d.stream is null
-- ... truncated (42 lines total)
```

### deal_create_repeat(p_deal_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 24 lines
```sql

declare d group.deals; nid uuid;
begin
  if group.sop_actor() is null then raise exception 'not authorised'; end if;
  if auth.uid() is not null and not group.can_work_deal(p_deal_id) then raise exception 'You do not own this record'; end if;
  select * into d from group.deals where id = p_deal_id;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;
  if exists (select 1 from group.deals where repeat_of = p_deal_id) then
    return jsonb_build_object('ok', true, 'already', true); end if;
  insert into group.deals (school_code, school_name, contact_name, contact_email, contact_phone,
    designation, decision_maker_name, decision_maker_role, country, region, campaign_tag,
    lead_source_type, stream, programme, stage, status, owner, currency, price_quoted_minor,
    minimum_required, next_action, next_action_owner, next_action_due, proposal_sent_date,
    repeat_of, created_by)
  values (d.school_code, d.school_name, d.contact_name, d.contact_email, d.contact_phone,
    d.designation, d.decision_maker_name, d.decision_maker_role, d.country, d.region,
    'repeat','repeat', d.stream, d.programme, 2, 'active', d.owner, d.currency,
    d.price_quoted_minor, d.minimum_required,
    'Propose the next cohort date for this school', d.owner, current_date + 14,
    current_date, p_deal_id, group.sop_actor())
  returning id into nid;
  update group.deals set post_delivery = post_delivery || '{"repeat_record":true}'::jsonb where id = p_deal_id;
  return jsonb_build_object('ok', true, 'repeat_deal_id', nid);
end 
```

### deal_gate_recommendation(p_deal_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

begin
  if not group.is_active_staff(auth.uid()) then raise exception 'not authorised'; end if;
  return group.deal_gate_recommendation__impl(p_deal_id);
end 
```

### deal_gate_recommendation__impl(p_deal_id uuid) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 15 lines
```sql

  select jsonb_build_object(
    'deal_id', d.id, 'school', d.school_name, 'programme_start', d.programme_start,
    'days_out', d.programme_start - current_date,
    'registered', coalesce(d.registered,0), 'minimum', coalesce(d.minimum_required,15),
    'has_count', d.registered is not null,
    'recommendation', case
      when d.registered is null then 'no_count'
      when d.registered >= ceil(coalesce(d.minimum_required,15) * 0.8) then 'confirmed'
      when d.registered >= ceil(coalesce(d.minimum_required,15) * 0.53) then 'rescue'
      else 'postponed' end,
    'gate_decision', d.gate_decision, 'gate_decision_at', d.gate_decision_at,
    'gate_due_on', d.programme_start - 45)
  from group.deals d where d.id = p_deal_id;

```

### deal_heat(p_stage smallint, p_follow_up_booked_at date, p_last_touch_at timestamp with time zone, p_next_action_due date, p_stage_entered_at timestamp with time zone, p_created_at timestamp with time zone) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 9 lines
```sql

  SELECT CASE
    WHEN p_stage >= 3 THEN 'hot'
    WHEN p_follow_up_booked_at IS NOT NULL AND p_follow_up_booked_at >= current_date THEN 'hot'
    WHEN p_last_touch_at >= date_trunc('week', now()) AND p_next_action_due IS NOT NULL THEN 'hot'
    WHEN COALESCE(p_last_touch_at, p_created_at) < now() - interval '21 days' THEN 'cold'
    WHEN p_stage IN (1,2) AND p_stage_entered_at < now() - interval '30 days' THEN 'cold'
    ELSE 'warm' END

```

### deal_post_delivery_tick(p_deal_id uuid, p_item text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 13 lines
```sql

declare pd jsonb;
begin
  if group.sop_actor() is null then raise exception 'not authorised'; end if;
  if auth.uid() is not null and not group.can_work_deal(p_deal_id) then raise exception 'You do not own this record'; end if;
  if p_item not in ('feedback','testimonial','repeat_ask','referral_ask','repeat_record') then
    raise exception 'unknown post-delivery item %', p_item; end if;
  update group.deals set post_delivery = post_delivery || jsonb_build_object(p_item, true)
   where id = p_deal_id returning post_delivery into pd;
  if pd is null then raise exception 'unknown deal %', p_deal_id; end if;
  if p_item = 'repeat_ask' then perform group.deal_create_repeat(p_deal_id); end if;
  return jsonb_build_object('ok', true, 'post_delivery', pd);
end 
```

### deal_work_action(p_deal_id uuid, p_action text, p_note text, p_date date, p_next_action text, p_channel text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 37 lines
```sql

DECLARE v_actor uuid := auth.uid(); v_owner uuid; v_kind text;
BEGIN
  SELECT owner INTO v_owner FROM deals WHERE id = p_deal_id AND merged_into IS NULL;
  IF NOT FOUND THEN RAISE EXCEPTION 'Deal not found'; END IF;
  IF NOT (group.is_admin_or_owner(v_actor) OR v_owner = v_actor OR (v_owner IS NULL AND group.is_active_staff(v_actor))) THEN
    RAISE EXCEPTION 'You do not own this record';
  END IF;

  IF p_action = 'done' THEN
    UPDATE deals SET last_touch_at = now(), touch_count = touch_count + 1,
      waiting_until = NULL, waiting_note = NULL,
      next_action = p_next_action, next_action_due = p_date,
      next_action_owner = CASE WHEN p_date IS NULL THEN next_action_owner ELSE COALESCE(next_action_owner, v_actor) END,
      updated_at = now()
    WHERE id = p_deal_id;
    v_kind := 'action_done';
  ELSIF p_action = 'snooze' THEN
    IF p_date IS NULL THEN RAISE EXCEPTION 'A new date is required to snooze'; END IF;
    UPDATE deals SET next_action_due = p_date, updated_at = now() WHERE id = p_deal_id;
    v_kind := 'action_snoozed';
  ELSIF p_action = 'waiting' THEN
    IF p_date IS NULL THEN RAISE EXCEPTION 'A chase date is required'; END IF;
    UPDATE deals SET waiting_until = p_date, waiting_note = p_note,
      next_action = COALESCE(NULLIF(p_next_action,''), 'Chase: ' || COALESCE(p_note, 'waiting on school')),
      next_action_due = p_date, last_touch_at = now(), touch_count = touch_count + 1, updated_at = now()
    WHERE id = p_deal_id;
    v_kind := 'waiting_on_school';
  ELSE
    RAISE EXCEPTION 'Unknown action %', p_action;
  END IF;

  INSERT INTO deal_events (deal_id, kind, channel, actor, payload)
  VALUES (p_deal_id, v_kind, p_channel, v_actor,
          jsonb_strip_nulls(jsonb_build_object('note', p_note, 'date', p_date, 'next_action', p_next_action)));
  RETURN jsonb_build_object('ok', true, 'kind', v_kind);
END 
```

### deals_flag_price_conflict() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 12 lines
```sql

begin
  if new.price_quoted_minor is not null and exists (
    select 1 from group.deals x
     where x.id <> new.id
       and coalesce(x.school_code, lower(x.school_name)) = coalesce(new.school_code, lower(new.school_name))
       and coalesce(x.programme,'') = coalesce(new.programme,'')
       and x.price_quoted_minor is not null and x.price_quoted_minor <> new.price_quoted_minor
       and x.created_at > now() - interval '365 days')
  then new.needs_review := true; end if;
  return new;
end 
```

### deals_guard_merge() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 14 lines
```sql

declare t group.deals;
begin
  if new.merged_into is null then return new; end if;
  if new.merged_into = new.id then raise exception 'A record cannot be merged into itself.'; end if;
  select * into t from group.deals where id = new.merged_into;
  if t.id is null then raise exception 'The target record does not exist.'; end if;
  if t.merged_into is not null then raise exception 'That target is itself merged. Use the surviving record.'; end if;
  if coalesce(t.school_code, lower(t.school_name)) is distinct from coalesce(new.school_code, lower(new.school_name))
    then raise exception 'SOP section 6: only records for the same school may be merged.'; end if;
  if coalesce(t.programme,'~') is distinct from coalesce(new.programme,'~')
    then raise exception 'SOP section 6: one record per lead per programme. Different programmes stay as two records; SOP 2.1 logs a cross-sell separately.'; end if;
  return new;
end 
```

### deals_stage_stamp() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 54 lines
```sql

declare actor uuid := group.sop_actor();
begin
  new.updated_at := now();
  if tg_op = 'INSERT' then
    new.stage_entered_at := coalesce(new.stage_entered_at, now());
    new.stage_history := jsonb_build_array(jsonb_build_object(
      'stage', new.stage, 'status', new.status, 'at', new.stage_entered_at, 'by', actor));
    return new;
  end if;
  if new.stage is not distinct from old.stage and new.status is not distinct from old.status then
    return new;
  end if;
  if new.status = 'lost' and old.status = 'active' and old.stage not between 1 and 5 then
    raise exception 'SOP: a deal at stage % cannot be marked Lost (allowed from stages 1-5)', old.stage;
-- ... truncated (54 lines total)
```

### derive_lead_segment(_designation text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 10 lines
```sql

  SELECT CASE
    WHEN _designation IS NULL OR btrim(_designation) = '' THEN 'unknown'
    WHEN lower(_designation) IN ('student') THEN 'student'
    WHEN lower(_designation) IN ('parent','guardian','mother','father') THEN 'parent'
    WHEN lower(_designation) ~ '(principal|representative|coordinator|teacher|founder|director|head|owner|dean|admin|counsell?or|leader|manager)'
      THEN 'school'
    ELSE 'unknown'
  END

```

### email_auto_drafts_set_enabled(p_on boolean) -> void
- sql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 1 lines
```sql
 update group.email_settings set auto_drafts_enabled = p_on where id = true 
```

### email_auto_set_enabled(p_on boolean, p_note text) -> boolean
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 14 lines
```sql

begin
  if auth.uid() is not null and not group.is_admin_or_owner(auth.uid()) then
    raise exception 'Only an owner or admin can switch automated emails on or off';
  end if;
  insert into group.email_settings (id, auto_enabled, note, updated_by, updated_at)
  values (true, p_on, p_note, auth.uid(), now())
  on conflict (id) do update
    set auto_enabled = excluded.auto_enabled,
        note = coalesce(excluded.note, group.email_settings.note),
        updated_by = excluded.updated_by,
        updated_at = now();
  return p_on;
end 
```

### email_body_repair_queue(p_limit integer) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=group, pg_temp | exec anon=n authenticated=n | 25 lines
```sql

  with s as (
    select 'email_sends'::text kind, e.id, e.message_id rfc_id, null::text gmail_id, e.direction, lower(e.from_email) from_email, lower(e.to_email) to_email,
      a.email account_email, m.meeting_date, coalesce(lower(m.email),'') lead_email, coalesce(e.received_at, e.sent_at, e.created_at) at
    from group.email_sends e
    left join group.meetings m on m.id = e.meeting_id
    left join group.user_gmail_accounts a on a.id = e.sent_via_gmail_account_id
    where e.status in ('sent','received') and e.message_id is not null
      and coalesce(e.body_text,'') = '' and coalesce(e.body_html,'') = '' and coalesce(e.rendered_html,'') = ''
    union all
    select 'meeting_followups', f.id, null, f.gmail_msgid, f.direction, lower(f.from_email), lower(f.to_emails),
      a.email, m.meeting_date, coalesce(lower(m.email),''), f.sent_at
    from group.meeting_followups f
    join group.user_gmail_accounts a on a.id = f.account_id
    left join group.meetings m on m.id = f.meeting_id
    where coalesce(f.body_text,'') = '' and f.gmail_msgid is not null
  ), due as (
    select s.*, (s.meeting_date >= '2026-11-02' and s.lead_email not like '%@example-group.com%') priority
    from s left join group.email_body_repair_attempts r on r.kind = s.kind and r.row_id = s.id
    where r.row_id is null or r.attempted_at < now() - interval '7 days'
    order by priority desc, at desc nulls last
    limit greatest(1, least(p_limit, 100))
  )
  select coalesce(jsonb_agg(to_jsonb(due)), '[]'::jsonb) from due;

```

### email_body_repair_status(p_ids uuid[]) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=group, pg_temp | exec anon=n authenticated=n | 2 lines
```sql
 select coalesce(jsonb_object_agg(row_id::text, jsonb_build_object('at', attempted_at, 'result', result)), '{}'::jsonb)
      from group.email_body_repair_attempts where row_id = any(p_ids) 
```

### email_domain(p_email text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 5 lines
```sql

  select nullif(
    regexp_replace(lower(btrim(coalesce(p_email,''))), '^.*@', ''),
  '')

```

### email_report(p_from date, p_to date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 70 lines
```sql

declare v jsonb;
begin
  if not (coalesce(auth.role(), '') = 'service_role' or group.is_admin_or_owner(auth.uid())) then
    raise exception 'Only owners and admins can see the email report';
  end if;
  with mt as (
    select c.id, lower(btrim(c.email)) em, coalesce(nullif(btrim(c.utm_source), ''), 'Direct / none') utm
      from group.v_meeting_clean c
     where c.meeting_date between p_from and p_to
  ),
  per as (
    select mt.*,
      exists (select 1 from group.email_sends s where s.meeting_id = mt.id and s.status = 'sent'
                and coalesce(s.direction, 'outbound') = 'outbound') got,
-- ... truncated (70 lines total)
```

### founder_ai_link_schools(p_items jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp; statement_timeout=120s | exec anon=n authenticated=n | 50 lines
```sql

#variable_conflict use_column
-- Founder-approved (21 Nov 2026) AI school linking for application school names.
-- item: {key, code} existing school; {key, new_name, country, city} cleaned new school; {key, skip_reason} not identifiable.
declare it jsonb; k text; v_code text; base text; i int; created int := 0; skipped int := 0; linked int := 0;
begin
  create temp table if not exists _fai(k text primary key, code text, is_new boolean) on commit drop;
  truncate _fai;
  for it in select * from jsonb_array_elements(p_items) loop
    k := it->>'key'; v_code := nullif(it->>'code','');
    if coalesce(k,'') = '' then continue; end if;
    if it ? 'skip_reason' then
      insert into group.application_school_matches(name_key, raw_name, school_code, confidence, method, evidence, applications_linked)
      values (k, k, null, 0, 'not_identifiable', 'Founder-approved AI review: '||(it->>'skip_reason'), 0);
      skipped := skipped + 1; continue;
-- ... truncated (50 lines total)
```

### founder_brief(p_from date, p_to date, p_first_name text, p_app_link text) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 209 lines
```sql

with live as (
  select m.*,
         case when m.lead_group is null or btrim(m.lead_group) = '' or m.lead_group = 'unknown' then 'unknown'
              when m.lead_group in ('student','parent') then 'b2c' else 'b2b' end as ltype,
         (coalesce(m.start_time_utc,
                   (m.meeting_date + coalesce(m.scheduled_time, '00:00'::time)) at time zone 'Asia/Kolkata')
          > now()) as upcoming
  from group.meetings m
  where m.meeting_date between p_from and p_to
    and lower(coalesce(m.meeting_status,'')) not in ('cancelled','rescheduled','no show')
),
b2b as (select * from live where ltype = 'b2b'),
app as (
  select school_code, count(*) filter (where is_paid) as paid, count(*) as applied,
-- ... truncated (209 lines total)
```

### founder_brief_history(p_school_code text, p_before date) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 18 lines
```sql

  select to_jsonb(x) from (
    select m.meeting_date as at,
           m.host_name as taken_by,
           nullif(btrim(m.next_step), '') as next_step,
           nullif(btrim(m.program_pitched), '') as pitched,
           nullif(btrim(m.objections), '') as objections,
           nullif(btrim(m.commitments), '') as commitments,
           (select array_agg(btrim(t)) from unnest(string_to_array(coalesce(m.key_quotes,''), '|')) t
             where btrim(t) <> '') as quotes,
           coalesce(group.founder_brief_points(m.ai_notes), nullif(btrim(m.meeting_notes),'')) as notes
    from group.meetings m
    where m.school_code = p_school_code and m.meeting_date < p_before
      and (m.key_quotes is not null or m.objections is not null or m.commitments is not null
           or nullif(btrim(m.next_step),'') is not null or m.ai_notes is not null)
    order by m.meeting_date desc limit 1
  ) x;

```

### founder_brief_points(notes text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 18 lines
```sql

  -- The gist of the last call, on one line and short enough to read on a phone.
  -- Two guards: a note under 30 characters is a placeholder, not a discussion
  -- point ("Akshita's meeting" is a real example); and some notes are a single
  -- unbroken paragraph, so taking "the first two lines" can return a thousand
  -- characters. Both would land in a WhatsApp template parameter.
  select case
    when length(btrim(coalesce(notes,''))) < 30 then null
    else (
      select case when length(t) > 200 then left(t, 199) || '…' else t end
      from (
        select btrim(regexp_replace(
                 array_to_string((string_to_array(btrim(notes), E'\n'))[1:2], ' · '),
                 '^[\s\-•*]+', '')) as t
      ) x
    )
  end

```

### founder_brief_recap(h jsonb, prior integer, budget integer) -> text
- plpgsql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 33 lines
```sql

declare out text := ''; part text; nothing text; ns text;
begin
  nothing := case when coalesce(prior, 0) > 0 then 'No notes from the last call.'
                  else 'First time we are meeting this school.' end;
  if h is null or h = 'null'::jsonb then return nothing; end if;

  ns := nullif(btrim(h->>'next_step'), '');

  -- Whole sentences, added while they fit and dropped whole when they do not,
  -- so the founder never reads half a thought followed by an ellipsis. They are
  -- written to read on from "Last call: " without stacking a second label —
  -- next_step is always an imperative ("Send…", "Follow…", "Confirm…"), so it
  -- folds into a sentence by lowercasing its first letter.
  foreach part in array array[
      case when nullif(btrim(h->>'pitched'), '') is not null
           then 'We pitched ' || btrim(h->>'pitched') || '.' end,
      case when ns is not null
           then 'We agreed to ' || lower(left(ns, 1)) || substr(ns, 2)
                || (case when right(ns, 1) in ('.', '!', '?') then '' else '.' end) end,
      case when nullif(btrim(h->>'objections'), '') is not null
           then 'Their concern: ' || btrim(h->>'objections') end
    ]
  loop
    if part is not null and length(out) + length(part) + 1 <= budget then
      out := case when out = '' then part else out || ' ' || part end;
    end if;
  end loop;

  if out = '' then return nothing; end if;
  if right(out, 1) not in ('.', '!', '?') then out := out || '.'; end if;
  return out;
end 
```

### founder_brief_text(p jsonb) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 7 lines
```sql

  select 'Hello ' || (p->>'first_name') || ',' || E'\n\n'
      || 'Hope you''re doing well!' || E'\n\n'
      || (p->>'summary_line') || E'\n\n'
      || coalesce(nullif(p->>'meetings_block_lines',''), p->>'meetings_block') || E'\n\n'
      || 'Thank you!'

```

### get_invitation_by_token(p_token text) -> TABLE(email text, role group.app_role, full_name text, title text, expires_at timestamp with time zone, accepted boolean)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 6 lines
```sql

  SELECT i.email, i.role, i.full_name, i.title, i.expires_at, i.accepted
  FROM group.invitations i
  WHERE i.token = p_token
  LIMIT 1;

```

### guard_member_meeting_update() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 45 lines
```sql

DECLARE
  caller uuid := auth.uid();
  is_admin boolean;
BEGIN
  IF caller IS NULL THEN RETURN NEW; END IF; -- service_role bypass
  SELECT group.is_admin_or_owner(caller) INTO is_admin;
  IF is_admin THEN RETURN NEW; END IF;

  -- PL/pgSQL resolves NEW.<field> at run time, so a column name that does not
  -- exist on `meetings` raises 42703 the moment a member updates any row — and
  -- only a member, because admins, owners and the service role all return
  -- above. That is why this stayed invisible until the team started using the
  -- platform. Two names were wrong and broke every member update, including
  -- "Assign to me":
-- ... truncated (45 lines total)
```

### guard_profile_role_change() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 22 lines
```sql

DECLARE
  caller uuid := auth.uid();
  caller_is_admin boolean;
BEGIN
  IF NEW.role IS DISTINCT FROM OLD.role OR NEW.is_active IS DISTINCT FROM OLD.is_active THEN
    -- service_role bypasses auth.uid(); allow that path (edge functions)
    IF caller IS NULL THEN
      RETURN NEW;
    END IF;
    SELECT group.is_admin_or_owner(caller) INTO caller_is_admin;
    IF NOT caller_is_admin THEN
      RAISE EXCEPTION 'Only admins or owners can change role or active status';
    END IF;
    -- Prevent self-demotion of the last remaining owner
    IF OLD.role = 'owner' AND NEW.role <> 'owner' AND caller = OLD.id THEN
      RAISE EXCEPTION 'Owners cannot demote themselves';
    END IF;
  END IF;
  RETURN NEW;
END;

```

### handle_new_user() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 23 lines
```sql

DECLARE
  v_inv record;
BEGIN
  SELECT * INTO v_inv
    FROM group.invitations
   WHERE lower(email) = lower(NEW.email)
     AND (expires_at IS NULL OR expires_at > now() OR accepted = true)
   ORDER BY created_at DESC
   LIMIT 1;

  INSERT INTO group.user_profiles (id, email, full_name, role, is_active)
  VALUES (
    NEW.id,
    NEW.email,
    COALESCE(v_inv.full_name, NEW.raw_user_meta_data->>'full_name', split_part(NEW.email, '@', 1)),
    COALESCE(v_inv.role, 'member'::app_role),
    v_inv.id IS NOT NULL
  )
  ON CONFLICT (id) DO NOTHING;
  RETURN NEW;
END;

```

### handle_new_user_if_invited() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public | exec anon=n authenticated=Y | 14 lines
```sql

declare v_inv record;
begin
  select * into v_inv from group.invitations
   where lower(email) = lower(new.email)
     and (expires_at is null or expires_at > now() or accepted = true)
   order by created_at desc limit 1;
  if v_inv.id is null then return new; end if;
  insert into group.user_profiles (id, email, full_name, role, is_active)
  values (new.id, new.email, coalesce(v_inv.full_name, new.raw_user_meta_data->>'full_name', split_part(new.email,'@',1)),
          coalesce(v_inv.role, 'member'::group.app_role), true)
  on conflict (id) do nothing;
  return new;
end 
```

### has_role(_user_id uuid, _role group.app_role) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 6 lines
```sql

  SELECT EXISTS (
    SELECT 1 FROM group.user_profiles
    WHERE id = _user_id AND role = _role
  )

```

### is_active_staff(_uid uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

  SELECT EXISTS (
    SELECT 1 FROM group.user_profiles
    WHERE id = _uid
      AND is_active = true
      AND role::text IN ('owner', 'admin', 'member')
  );

```

### is_admin_or_owner(_user_id uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 6 lines
```sql

  SELECT EXISTS (
    SELECT 1 FROM group.user_profiles
    WHERE id = _user_id AND role IN ('owner','admin') AND is_active = true
  )

```

### is_assigned_to_meeting(_meeting_id uuid, _user_id uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 6 lines
```sql

  SELECT EXISTS (
    SELECT 1 FROM group.meeting_assignees
    WHERE meeting_id = _meeting_id AND user_id = _user_id
  )

```

### is_internal_email(p_email text) -> boolean
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 6 lines
```sql

  SELECT coalesce(
    lower(trim(p_email)) ~ '@(example-group.com|example-group.in|partner-a.com|partner-b.in|example-group.com|example-group.in|partner-a.com|partner-b.in|example-group.com|example-group.in|partner-a.com|partner-b.in|example-group.com|example-group.in|partner-a.com|partner-b.in)$'
    OR lower(trim(p_email)) IN ('jojo.kearney@corporatednaconsulting.com'),
  false)

```

### is_internal_lead(p_email text, p_school text, p_name text) -> boolean
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 6 lines
```sql

  select coalesce(p_email,'')  ilike '%@example-group.com'
      or coalesce(p_school,'') ilike 'group%'
      or coalesce(p_school,'') ilike '%example group%'
      or coalesce(p_name,'')   ilike 'group%';

```

### is_meeting_host(_meeting_id uuid, _user_id uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 3 lines
```sql

  select exists (select 1 from group.meetings m where m.id = _meeting_id and m.host_user_id = _user_id)

```

### lead_segment_of_group(_grp text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 7 lines
```sql

  SELECT CASE
    WHEN _grp IN ('student','parent') THEN _grp
    WHEN _grp IN ('school','university','corporate','partner','other') THEN 'school'
    ELSE 'unknown'
  END

```

### lead_type_of_group(_grp text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 7 lines
```sql

  SELECT CASE
    WHEN _grp IN ('student','parent') THEN 'b2c'
    WHEN _grp IN ('school','university','corporate','partner','other') THEN 'b2b'
    ELSE 'unknown'
  END

```

### link_application_school(p_key text, p_code text, p_new_name text) -> text
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 42 lines
```sql

declare k text := group.conversion_school_key(p_key); v_code text := p_code; base text; i int := 1;
begin
  if not (select group.is_admin_or_owner((select auth.uid()))) then
    raise exception 'Not authorised' using errcode = '42501';
  end if;
  if coalesce(k, '') = '' then raise exception 'School name required'; end if;
  if v_code is null and nullif(btrim(p_new_name), '') is not null then
    select r.code into v_code from group.school_registry r
     where r.is_active and group.conversion_school_key(r.canonical_name) = group.conversion_school_key(p_new_name)
     order by r.code limit 1;
  end if;
  if v_code is null and nullif(btrim(p_new_name), '') is not null then
    base := left(coalesce(nullif(array_to_string(array(
      select left(w, 4) from regexp_split_to_table(
-- ... truncated (42 lines total)
```

### link_application_school_key(p_key text, p_school text, p_confidence double precision, p_method text, p_evidence text, p_raw text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 12 lines
```sql

declare n integer := 0;
begin
  if p_school is not null then
    update group.core_applications c set school_code = p_school
     where c.school_code is null and group.conversion_school_key(c.school_raw) = p_key;
    get diagnostics n = row_count;
  end if;
  insert into group.application_school_matches (name_key, raw_name, school_code, confidence, method, evidence, applications_linked)
  values (p_key, p_raw, p_school, p_confidence, p_method, p_evidence, n);
  return n;
end 
```

### match_applications_by_meeting_email() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 21 lines
```sql

declare n integer;
begin
  with hits as (
    select c.id, min(m.school_code) sc, min(m.id::text) mid
      from group.core_applications c
      join group.meetings m on m.merged_into is null and m.school_code is not null
       and nullif(btrim(m.email),'') is not null
       and lower(btrim(m.email)) in (lower(btrim(c.student_email)), lower(btrim(c.parent_email)))
     where c.school_code is null
     group by c.id
    having count(distinct m.school_code) = 1
  ), upd as (
    update group.core_applications c set school_code = h.sc from hits h where c.id = h.id
    returning c.id, h.sc, h.mid, c.school_raw
  )
  insert into group.application_school_matches (name_key, raw_name, application_id, school_code, confidence, method, evidence, applications_linked)
  select coalesce(group.conversion_school_key(school_raw),''), school_raw, id, sc, 1, 'meeting_email', 'Student or parent email matches meeting ' || mid, 1 from upd;
  get diagnostics n = row_count;
  return n;
end 
```

### meeting_app_counts(p_ids uuid[]) -> TABLE(meeting_id uuid, apps_earlier integer, apps_after integer, paid_after integer)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 29 lines
```sql

declare v_ids uuid[] := p_ids;
begin
  -- Decide access once, not per row (service role and staff see all; members only their assigned meetings).
  if not (coalesce(auth.role(),'') = 'service_role' or auth.uid() is null
          or group.is_admin_or_owner(auth.uid()) or group.is_active_staff(auth.uid())) then
    select array_agg(x) into v_ids from unnest(p_ids) x
     where x in (select group.my_assigned_meeting_ids());
  end if;
  return query
  select m.id,
    case
      when nullif(btrim(m.school_code), '') is not null
           and (m.lead_group = 'school' or m.lead_segment = 'school') then
        (select count(*)::int from group.core_applications c
          where c.school_code = m.school_code
            and c.applied_at < (m.meeting_date::timestamp at time zone 'Asia/Kolkata'))
      when coalesce(m.lead_group, 'unknown') <> 'school' and nullif(btrim(m.email), '') is not null then
        (select count(*)::int from group.core_applications c
          where (lower(c.student_email) = lower(btrim(m.email)) or lower(c.parent_email) = lower(btrim(m.email)))
            and c.applied_at < (m.meeting_date::timestamp at time zone 'Asia/Kolkata'))
      else null
    end,
    o.apps_after, o.paid_after
  from group.meetings m
  left join group.mv_meeting_outcomes o on o.meeting_id = m.id
  where m.id = any(coalesce(v_ids, '{}'::uuid[]));
end

```

### meeting_bullets_extract(p text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 10 lines
```sql

  with b as (
    select regexp_replace(btrim(l), '^([-•*]|\d+[.)])\s*', '') t, i
      from unnest(string_to_array(regexp_replace(coalesce(p, ''), E'\r', '', 'g'), E'\n')) with ordinality u(l, i)
     where length(btrim(l)) > 3
     order by i limit 2
  ), j as (select string_agg(t, ' · ' order by i) x from b)
  select case when x is null then null when length(x) <= 280 then x
              else regexp_replace(left(x, 280), '\s+\S*$', '') || '…' end from j

```

### meeting_close_out(p_meeting_id uuid, p_outcome text, p_qualified text, p_summary text, p_decision_maker text, p_decision_maker_role text, p_programme text, p_students integer, p_blocker text, p_proposal text, p_next_meeting date, p_no_next_reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 110 lines
```sql

declare v_actor uuid := auth.uid();
        v_deal uuid; v_owner uuid; v_school text; v_stage smallint;
        v_next_action text; v_next_due date; v_result text;
begin
  if v_actor is null then raise exception 'not authorised'; end if;
  if coalesce(btrim(p_summary),'') = '' then
    raise exception 'A one-line summary is required. If the call is not worth a sentence it is not worth a record.'; end if;
  if p_outcome not in ('interested','needs_approval','not_now','not_a_fit','no_show') then
    raise exception 'Pick an outcome'; end if;
  if p_next_meeting is null and coalesce(btrim(p_no_next_reason),'') = '' and p_outcome <> 'no_show' then
    raise exception 'Either book the next meeting or say why there is not one'; end if;

  select m.deal_id into v_deal from group.meetings m where m.id = p_meeting_id;
  if not found then raise exception 'Meeting not found'; end if;
-- ... truncated (110 lines total)
```

### meeting_export_rows(p_ids uuid[]) -> TABLE(meeting_id uuid, year integer, meeting_date date, ist_time text, dubai_time text, full_name text, school_name text, email text, phone text, city text, state text, country text, designation text, meeting_type text, utm text, query text, meeting_link text, linkedin text, apps_earlier integer, taken_by text, meeting_status text, transcript text, ai_notes text, meeting_notes text, apps_after integer, category text, previous_count integer, previous_meetings text, utm_source text, utm_medium text, utm_campaign text, booked_via text)
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 71 lines
```sql

  with m as (
    select m.*,
           coalesce((m.meeting_date + m.scheduled_time) at time zone 'Asia/Kolkata', m.start_time_utc) starts_at
      from group.meetings m where m.id = any(p_ids)
  )
  select m.id,
    extract(year from m.meeting_date)::int,
    m.meeting_date,
    to_char(m.starts_at at time zone 'Asia/Kolkata', 'HH12:MI AM'),
    to_char(m.starts_at at time zone 'Asia/Dubai', 'HH12:MI AM'),
    m.full_name, m.school_name, m.email, m.phone, m.city, m.state, m.country, m.designation,
    m.meeting_type,
    m.utm_source,
    m.query, m.meeting_link, m.linkedin_url,
-- ... truncated (71 lines total)
```

### meeting_followup_status(p_ids uuid[]) -> TABLE(meeting_id uuid, held_at timestamp with time zone, owner_connected boolean, sent_count integer, reply_count integer, first_sent_at timestamp with time zone, hours_to_first numeric, state text)
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 25 lines
```sql

  WITH m AS (
    SELECT mt.id, COALESCE(mt.actual_start, mt.start_time_utc) AS held_at
    FROM meetings mt WHERE mt.id = ANY(p_ids)
  ), f AS (
    SELECT f.meeting_id,
      count(*) FILTER (WHERE direction='out')::int s,
      count(*) FILTER (WHERE direction='in')::int r,
      min(sent_at) FILTER (WHERE direction='out') fs
    FROM meeting_followups f WHERE f.meeting_id = ANY(p_ids) GROUP BY 1
  ), c AS (
    SELECT DISTINCT ma.meeting_id FROM meeting_assignees ma
    JOIN user_gmail_accounts g ON g.user_id = ma.user_id AND g.status='connected'
    WHERE ma.meeting_id = ANY(p_ids)
  )
  SELECT m.id, m.held_at, (c.meeting_id IS NOT NULL), COALESCE(f.s,0), COALESCE(f.r,0), f.fs,
    round(extract(epoch FROM f.fs - m.held_at)/3600.0, 1),
    CASE WHEN c.meeting_id IS NULL AND f.s IS NULL THEN 'not_connected'
         WHEN f.fs IS NULL AND now() - m.held_at <= interval '2 days' THEN 'pending'
         WHEN f.fs IS NULL THEN 'not_sent'
         WHEN f.fs - m.held_at <= interval '2 days' THEN 'on_time'
         WHEN f.fs - m.held_at <= interval '7 days' THEN 'late'
         ELSE 'very_late' END
  FROM m LEFT JOIN f ON f.meeting_id=m.id LEFT JOIN c ON c.meeting_id=m.id;

```

### meeting_for_zoom_id(p_zoom_meeting_id text) -> uuid
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

  select coalesce(
    (select id from group.meetings where zoom_meeting_id = p_zoom_meeting_id order by meeting_date desc nulls last limit 1),
    (select meeting_id from group.meeting_zoom_links where zoom_meeting_id = p_zoom_meeting_id))

```

### meeting_has_assignee(_meeting_id uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 3 lines
```sql

  SELECT EXISTS (SELECT 1 FROM group.meeting_assignees WHERE meeting_id = _meeting_id)

```

### meeting_manual_lock() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 39 lines
```sql

declare
  lockable constant text[] := array['meeting_status','meeting_date','scheduled_time','start_time_utc',
                                    'end_time_utc','duration_minutes','meeting_link','zoom_meeting_id'];
  caller uuid := auth.uid();
  o jsonb; n jsonb; f text; keep jsonb := '{}'::jsonb; dur interval;
begin
  o := to_jsonb(old); n := to_jsonb(new);
  if caller is not null and group.is_admin_or_owner(caller) then
    if (new.meeting_date is distinct from old.meeting_date or new.scheduled_time is distinct from old.scheduled_time)
       and new.start_time_utc is not distinct from old.start_time_utc
       and new.meeting_date is not null and new.scheduled_time is not null then
      dur := coalesce(old.end_time_utc - old.start_time_utc, make_interval(mins => coalesce(old.duration_minutes, 30)));
      new.start_time_utc := (new.meeting_date + new.scheduled_time) at time zone 'Asia/Kolkata';
      new.end_time_utc := new.start_time_utc + dur;
      n := to_jsonb(new);
    end if;
    foreach f in array lockable loop
      if (n -> f) is distinct from (o -> f) then
        new.manual_overrides := coalesce(new.manual_overrides, '{}'::jsonb)
          || jsonb_build_object(f, jsonb_build_object('by', caller, 'at', now()));
        insert into group.meeting_edit_log (meeting_id, field, old_value, new_value, changed_by)
        values (old.id, f, o ->> f, n ->> f, caller);
      end if;
    end loop;
    return new;
  end if;
  new.manual_overrides := old.manual_overrides;
  if old.manual_overrides = '{}'::jsonb then return new; end if;
  for f in select jsonb_object_keys(old.manual_overrides) loop
    if f = any(lockable) and (n -> f) is distinct from (o -> f) then
      keep := keep || jsonb_build_object(f, o -> f);
    end if;
  end loop;
  if keep <> '{}'::jsonb then
    new := jsonb_populate_record(new, keep);
  end if;
  return new;
end 
```

### meeting_quick_note_of(p_short text, p_zoom text, p_ai_notes text, p_notes text, p_status text) -> TABLE(note text, source text)
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 14 lines
```sql

  select regexp_replace(x.note, '^[^[:alnum:]]*(quick recap|meeting recap|recap|summary)\s*[:.\-–—]?\s*', '', 'i'), x.source from (
    select nullif(btrim(p_short), '') note, 'ai'::text source, 1 ord
    union all select group.meeting_zoom_extract(p_zoom), 'zoom', 2
    union all select group.meeting_bullets_extract(p_ai_notes), 'ai_notes', 3
    union all select case when length(btrim(p_notes)) <= 280 then nullif(btrim(p_notes), '')
                          else regexp_replace(left(btrim(p_notes), 280), '\s+\S*$', '') || '…' end, 'team_notes', 4
    union all select case p_status
                       when 'No Show' then 'Did not join (marked no-show); nothing was recorded.'
                       when 'Cancelled' then 'Cancelled; the meeting did not take place.'
                       when 'Rescheduled' then 'Rescheduled to another slot.'
                     end, 'status', 5
  ) x where x.note is not null order by x.ord limit 1

```

### meeting_quick_notes(p_ids uuid[]) -> TABLE(meeting_id uuid, note text, source text, status_doubt boolean, booking_query text)
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 9 lines
```sql

  select m.id, q.note, q.source,
         (m.meeting_status = 'No Show' and length(coalesce(m.zoom_ai_summary, '')) >= 400),
         case when length(btrim(m.query)) <= 240 then nullif(btrim(m.query), '')
              else regexp_replace(left(btrim(m.query), 240), '\s+\S*$', '') || '…' end
    from group.meetings m
    left join lateral group.meeting_quick_note_of(m.ai_summary_short, m.zoom_ai_summary, m.ai_notes, m.meeting_notes, m.meeting_status) q on true
   where m.id = any(p_ids)

```

### meeting_report(p_from date, p_to date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 99 lines
```sql

declare
  v_now timestamp := now() at time zone 'Asia/Kolkata';
  r jsonb;
begin
  if coalesce(auth.role(), '') <> 'service_role' and not group.is_admin_or_owner(auth.uid()) then
    raise exception 'Only owners and admins can see the meeting report' using errcode = '42501';
  end if;
  if p_from is null or p_to is null or p_to < p_from then
    raise exception 'meeting_report needs a from date on or before the to date';
  end if;

  with base as (
    select m.id, m.meeting_date, m.scheduled_time, m.full_name, m.school_name, m.school_code, m.meeting_status,
      case mm.lead_group when 'school' then 'School' when 'student' then 'Student' when 'parent' then 'Parent'
-- ... truncated (99 lines total)
```

### meeting_school_timeline(p_ids uuid[]) -> TABLE(meeting_id uuid, apps_before integer, paid_before integer, apps_after integer, paid_after integer, window_end date)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 26 lines
```sql

  with me as (
    select m.id, m.meeting_date d, m.school_code sc, lower(btrim(m.email)) em
      from group.meetings m
     where m.id = any(p_ids) and m.meeting_date is not null
       and (select group.is_admin_or_owner((select auth.uid())))
  ), w as (
    select me.*, least(me.d + 90, coalesce((
        select min(n.meeting_date) from group.meetings n
         where n.merged_into is null and n.meeting_date > me.d
           and n.meeting_status = 'Completed'
           and (case when me.sc is not null then n.school_code = me.sc else lower(btrim(n.email)) = me.em end)
      ), me.d + 90)) e
      from me
  )
  select w.id,
    count(c.id) filter (where c.applied_at < w.d)::int,
    count(c.id) filter (where c.paid_at < w.d)::int,
    count(c.id) filter (where c.applied_at >= w.d and c.applied_at < w.e)::int,
    count(c.id) filter (where c.paid_at >= w.d and c.paid_at < w.e)::int,
    w.e
  from w
  left join group.core_applications c
    on (case when w.sc is not null then c.school_code = w.sc else lower(c.student_email) = w.em end)
  group by w.id, w.e

```

### meeting_staff_present(p_meeting_id uuid, p_asof timestamp with time zone) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 9 lines
```sql

  select case
    when count(*) filter (where participant_kind = 'staff') > 0 then true
    when count(*) filter (where participant_kind = 'guest') > 0 then false
    else null
  end
  from group.meeting_participants
  where meeting_id = p_meeting_id and joined_at <= p_asof

```

### meeting_zoom_extract(p text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 17 lines
```sql

  with para as (
    select (select btrim(l)
              from unnest(string_to_array(regexp_replace(coalesce(p, ''), E'\r', '', 'g'), E'\n')) with ordinality u(l, i)
             where length(btrim(l)) >= 80
               and btrim(l) !~* '^(meeting summary for|quick recap|summary|next steps)'
             order by i limit 1) t
  ), s as (
    select t,
      (regexp_match(t, '^((?:[^.!?]|\.(?=\S)|\m(?:Mr|Ms|Mrs|Dr|Prof|St|vs)\.)*[.!?])'))[1] s1,
      (regexp_match(t, '^((?:[^.!?]|\.(?=\S)|\m(?:Mr|Ms|Mrs|Dr|Prof|St|vs)\.)*[.!?](?:\s+(?:[^.!?]|\.(?=\S)|\m(?:Mr|Ms|Mrs|Dr|Prof|St|vs)\.)*[.!?])?)'))[1] s2
    from para where t is not null
  ), pick as (
    select case when length(coalesce(s1, '')) >= 90 then s1 else coalesce(s2, s1, t) end x from s
  )
  select case when length(x) <= 280 then x else regexp_replace(left(x, 280), '\s+\S*$', '') || '…' end from pick

```

### meetings_set_booking_rep() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 11 lines
```sql

BEGIN
  IF NEW.calendly_host_email IS NOT NULL
     AND lower(btrim(NEW.calendly_host_email)) <> 'team@example-group.com'
     AND NEW.calendly_host_email NOT LIKE '%@deleted.calendly.com'
  THEN
    NEW.booking_rep_id := COALESCE(group.resolve_booking_rep(NEW.calendly_host_email), NEW.booking_rep_id);
  END IF;
  RETURN NEW;
END;

```

### meetings_sheet_changes(p_after timestamp with time zone, p_after_id uuid, p_limit integer) -> TABLE(id uuid, changed_at timestamp with time zone)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, pg_temp | exec anon=n authenticated=n | 11 lines
```sql

  select x.id, x.changed_at from (
    select m.id, greatest(m.updated_at, m.created_at,
      (select max(f.created_at) from group.meeting_followups f where f.meeting_id = m.id),
      (select max(e.created_at) from group.email_sends e where e.meeting_id = m.id),
      (select max(a.assigned_at) from group.meeting_assignees a where a.meeting_id = m.id)) changed_at
    from group.meetings m where m.meeting_date is not null
  ) x
  where p_after is null or (x.changed_at, x.id) > (p_after, coalesce(p_after_id, '00000000-0000-0000-0000-000000000000'::uuid))
  order by x.changed_at, x.id limit least(greatest(p_limit,1),1000)

```

### meetings_sheet_kick() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, pg_temp | exec anon=n authenticated=n | 25 lines
```sql

declare
  today date := (now() at time zone 'Asia/Kolkata')::date + 1; -- sheet runs one day ahead
  relevant boolean;
  cmd text;
begin
  if tg_op = 'INSERT' then
    select exists (select 1 from new_rows where meeting_date is not null and meeting_date <= today) into relevant;
  else
    select exists (select 1 from new_rows where meeting_date is not null and meeting_date <= today)
        or exists (select 1 from old_rows where meeting_date is not null and meeting_date <= today) into relevant;
  end if;
  if not relevant then return null; end if;
  update group.meetings_sheet_sync_state set last_kick_at = now()
   where id = true and (last_kick_at is null or last_kick_at < now() - interval '30 seconds');
  if not found then
    update group.meetings_sheet_sync_state set kick_pending = true where id = true and not kick_pending;
    return null;
  end if;
  select command into cmd from cron.job where jobname = 'group-meetings-sheet-sync' and active;
  if cmd is not null then execute cmd; end if;
  return null;
exception when others then
  return null;
end 
```

### meetings_sheet_kick_mail() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, pg_temp | exec anon=n authenticated=n | 16 lines
```sql

declare cmd text;
begin
  if not exists (select 1 from new_rows n join group.meetings m on m.id = n.meeting_id
                 where n.status in ('sent','received') and m.meeting_date <= (now() at time zone 'Asia/Kolkata')::date + 1) then return null; end if;
  update group.meetings_sheet_sync_state set last_kick_at = now()
   where id = true and (last_kick_at is null or last_kick_at < now() - interval '30 seconds');
  if not found then
    update group.meetings_sheet_sync_state set kick_pending = true where id = true and not kick_pending;
    return null;
  end if;
  select command into cmd from cron.job where jobname = 'group-meetings-sheet-sync' and active;
  if cmd is not null then execute cmd; end if;
  return null;
exception when others then return null;
end 
```

### meetings_sheet_lock(p_token uuid) -> boolean
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, pg_temp | exec anon=n authenticated=n | 1 lines
```sql
 begin update group.meetings_sheet_sync_state set lock_token=p_token,locked_until=now()+interval '5 minutes',updated_at=now() where id=true and (locked_until is null or locked_until<now()); return found; end 
```

### meetings_sheet_owner_edit(p_meeting uuid, p_users uuid[], p_previous uuid[], p_actor text) -> boolean
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, pg_temp | exec anon=n authenticated=n | 1 lines
```sql
 declare current_ids uuid[]; begin perform 1 from group.meetings where id=p_meeting for update; if not found then return false; end if; select coalesce(array_agg(user_id order by user_id),'{}'::uuid[]) into current_ids from group.meeting_assignees where meeting_id=p_meeting; if current_ids is distinct from p_previous then return false; end if; if cardinality(p_users)=0 or exists(select 1 from unnest(p_users) u where not exists(select 1 from group.user_profiles p where p.id=u and p.is_active)) then return false; end if; delete from group.meeting_assignees where meeting_id=p_meeting and not(user_id=any(p_users)); insert into group.meeting_assignees(meeting_id,user_id,assigned_by) select p_meeting,u,p_actor from unnest(p_users) u on conflict(meeting_id,user_id) do nothing; return true; end 
```

### meetings_sheet_unlock(p_token uuid, p_result jsonb, p_cursor_date date, p_cursor_time time without time zone, p_cursor_id uuid) -> void
- sql | SECURITY DEFINER | volatile | config: search_path=group, public, pg_temp | exec anon=n authenticated=n | 1 lines
```sql
 update group.meetings_sheet_sync_state set locked_until=null,lock_token=null,last_result=p_result,cursor_date=p_cursor_date,cursor_time=p_cursor_time,cursor_id=p_cursor_id,updated_at=now() where id=true and lock_token=p_token 
```

### member_activity(p_user_id uuid, p_kinds text[], p_limit integer) -> TABLE(activity_id text, occurred_at timestamp with time zone, kind text, actor_type text, actor_name text, title text, detail text, meeting_id uuid, lead_email text, school_code text, source_table text)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

begin
  if not group.is_active_staff(auth.uid()) then raise exception 'not authorised'; end if;
  return query select * from group.member_activity__impl(p_user_id, p_kinds, p_limit);
end 
```

### member_activity__impl(p_user_id uuid, p_kinds text[], p_limit integer) -> TABLE(activity_id text, occurred_at timestamp with time zone, kind text, actor_type text, actor_name text, title text, detail text, meeting_id uuid, lead_email text, school_code text, source_table text)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 19 lines
```sql

  with me as materialized (
    select p.id as user_id, p.full_name from group.user_profiles p where p.id = p_user_id
  ),
  mine as materialized (
    select vm.meeting_id from group.v_member_meeting vm where vm.user_id = p_user_id
  ),
  ev as materialized (
    select a.activity_id, a.occurred_at, a.kind, a.actor_type, a.actor_name,
           a.title, a.detail, a.meeting_id, a.lead_email, a.school_code, a.source_table
      from group.v_activity a
     where a.meeting_id in (select meeting_id from mine)
        or a.actor_name = (select full_name from me)
  )
  select * from ev
   where p_kinds is null or ev.kind = any(p_kinds)
   order by ev.occurred_at desc
   limit p_limit;

```

### member_daily_email_dispatch(p_body jsonb) -> void
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 9 lines
```sql

declare cmd text;
begin
  select command into cmd from cron.job where jobname = 'group-member-sheet-report' limit 1;
  if cmd is null then raise exception 'Scheduler credential not found'; end if;
  cmd := replace(cmd, '/functions/v1/member-sheet-report', '/functions/v1/send-member-daily');
  cmd := replace(cmd, '), timeout_milliseconds', ') || ' || quote_literal(p_body::text) || '::jsonb, timeout_milliseconds');
  execute cmd;
end 
```

### member_daily_email_dispatch_all(p_body jsonb) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 12 lines
```sql

declare r record; n int := 0;
begin
  for r in select s.member_id from group.member_sheet_reports s join group.user_profiles u on u.id = s.member_id
            where s.enabled and coalesce(u.is_active, true)
              and not (s.member_id = any (coalesce((select excluded_member_ids from group.member_daily_email_settings where id), '{}')))
            order by s.tab_name loop
    perform group.member_daily_email_dispatch(p_body || jsonb_build_object('member_id', r.member_id));
    n := n + 1;
  end loop;
  return n;
end 
```

### member_email_agent_dispatch(p_body jsonb) -> void
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 9 lines
```sql

declare cmd text;
begin
  select command into cmd from cron.job where jobname='group-member-sheet-report' limit 1;
  if cmd is null then raise exception 'Scheduler credential not found'; end if;
  cmd := replace(cmd, '/functions/v1/member-sheet-report', '/functions/v1/member-email-agent');
  cmd := replace(cmd, '), timeout_milliseconds', ') || ' || quote_literal(p_body::text) || '::jsonb, timeout_milliseconds');
  execute cmd;
end
```

### member_email_agent_dispatch_all(p_body jsonb) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 9 lines
```sql

declare r record; n int := 0;
begin
  for r in select member_id from group.member_email_agent where enabled order by last_run_at nulls first loop
    perform group.member_email_agent_dispatch(p_body || jsonb_build_object('member_id', r.member_id));
    n := n + 1;
  end loop;
  return n;
end
```

### member_followup_stats(p_from date) -> TABLE(user_id uuid, connected boolean, held integer, fu1_sent integer, late integer, not_sent integer, replies integer, avg_hours numeric)
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 18 lines
```sql

  WITH mm AS (
    SELECT ma.user_id, m.id, COALESCE(m.actual_start, m.start_time_utc) held_at
    FROM meetings m JOIN meeting_assignees ma ON ma.meeting_id=m.id
    WHERE m.meeting_status='Completed' AND m.meeting_date >= p_from AND m.meeting_date <= current_date
  ), f AS (
    SELECT meeting_id, min(sent_at) FILTER (WHERE direction='out') fs, count(*) FILTER (WHERE direction='in') r
    FROM meeting_followups WHERE meeting_id IN (SELECT id FROM mm) GROUP BY 1
  )
  SELECT mm.user_id,
    EXISTS(SELECT 1 FROM user_gmail_accounts g WHERE g.user_id=mm.user_id AND g.status='connected'),
    count(*)::int, count(f.fs)::int,
    count(*) FILTER (WHERE f.fs - mm.held_at > interval '2 days')::int,
    count(*) FILTER (WHERE f.fs IS NULL AND now()-mm.held_at > interval '2 days')::int,
    COALESCE(sum(f.r),0)::int,
    round(avg(extract(epoch FROM f.fs - mm.held_at)/3600.0)::numeric, 1)
  FROM mm LEFT JOIN f ON f.meeting_id=mm.id GROUP BY mm.user_id;

```

### member_meetings(p_user_id uuid, p_from date, p_to date, p_limit integer) -> TABLE(meeting_id uuid, meeting_date date, scheduled_time text, lead_name text, lead_email text, school_name text, school_code text, country text, programme text, meeting_status text, outcome_state text, apps_after integer, paid_after integer, days_to_pay integer, analysed boolean, whatsapp_captured boolean, next_meeting_agreed boolean, docs_promised integer, open_questions integer, actions_open integer)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

begin
  if not group.is_active_staff(auth.uid()) then raise exception 'not authorised'; end if;
  return query select * from group.member_meetings__impl(p_user_id, p_from, p_to, p_limit);
end 
```

### member_meetings__impl(p_user_id uuid, p_from date, p_to date, p_limit integer) -> TABLE(meeting_id uuid, meeting_date date, scheduled_time text, lead_name text, lead_email text, school_name text, school_code text, country text, programme text, meeting_status text, outcome_state text, apps_after integer, paid_after integer, days_to_pay integer, analysed boolean, whatsapp_captured boolean, next_meeting_agreed boolean, docs_promised integer, open_questions integer, actions_open integer)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 37 lines
```sql

declare v_ids uuid[];
begin
  select array_agg(x.id) into v_ids from (
    select m.id
      from group.v_member_meeting vm
      join group.meetings m on m.id = vm.meeting_id
     where vm.user_id = p_user_id
       and m.meeting_date between p_from and p_to
       and m.merged_into is null
     order by m.meeting_date desc, m.scheduled_time desc nulls last
     limit p_limit) x;
  if v_ids is null then return; end if;
  return query execute $q$
    with led as materialized (
      select l.meeting_id, l.outcome_state, l.apps_after, l.paid_after, l.days_to_pay,
             l.analysed, l.whatsapp_captured, l.next_meeting_agreed, l.docs_promised, l.open_questions
        from group.v_meeting_ledger l where l.meeting_id = any($1)
    ),
    t as (
      select t.meeting_id, count(*)::int n from group.tasks t
       where t.status in ('pending','needs_info') and coalesce(t.archived,false) = false
         and t.meeting_id = any($1)
       group by 1
    )
    select m.id, m.meeting_date, m.scheduled_time::text, m.full_name, lower(btrim(m.email)),
           m.school_name, m.school_code, m.country, m.program_pitched, m.meeting_status,
           led.outcome_state, led.apps_after::int, led.paid_after::int, led.days_to_pay::int,
           led.analysed, led.whatsapp_captured, led.next_meeting_agreed,
           led.docs_promised::int, led.open_questions::int, coalesce(t.n, 0)
      from group.meetings m
      left join led on led.meeting_id = m.id
      left join t on t.meeting_id = m.id
     where m.id = any($1)
     order by m.meeting_date desc, m.scheduled_time desc nulls last
  $q$ using v_ids;
end 
```

### member_performance() -> SETOF group.v_member_performance
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp; statement_timeout=45s | exec anon=n authenticated=Y | 7 lines
```sql

BEGIN
  IF NOT group.is_admin_or_owner(auth.uid()) THEN
    RAISE EXCEPTION 'not allowed';
  END IF;
  RETURN QUERY SELECT * FROM group.v_member_performance;
END 
```

### member_report_assignment_dirty() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 4 lines
```sql
declare day date; begin
  select meeting_date into day from group.meetings where id=case when TG_OP='DELETE' then old.meeting_id else new.meeting_id end;
  update group.member_report_days set next_refresh_at=least(next_refresh_at, now()+interval '3 minutes'), revision=revision+1 where meeting_day=day;
  return null; end
```

### member_report_day_claim(p_token uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 31 lines
```sql

declare r group.member_report_days; today date := (now() at time zone 'Asia/Kolkata')::date; pick date;
begin
  if not pg_try_advisory_xact_lock(20461120,2) then return null; end if;
  if exists(select 1 from group.member_report_days where locked_until>now()) then return null; end if;
  insert into group.member_report_days(meeting_day,target_key,member_id,spreadsheet_id,tab_name)
    select d::date,c.member_id::text,c.member_id,c.spreadsheet_id,c.tab_name
    from generate_series('2026-11-02'::date,today,interval '1 day') d cross join group.member_sheet_reports c where c.enabled on conflict do nothing;
  insert into group.member_report_days(meeting_day,target_key,spreadsheet_id,tab_name)
    select d::date,'unassigned',c.spreadsheet_id,'Unassigned'
    from generate_series('2026-11-02'::date,today,interval '1 day') d
    cross join (select spreadsheet_id from group.member_sheet_reports where enabled order by tab_name limit 1) c on conflict do nothing;
  -- First pass: dates never written, newest first (today's live work must never wait behind history).
  select max(meeting_day) into pick from group.member_report_days
    where meeting_day<=today and last_success_at is null and next_refresh_at<=now() and (locked_until is null or locked_until<now());
  if pick is not null then
    select * into r from group.member_report_days
      where meeting_day=pick and last_success_at is null and next_refresh_at<=now() and (locked_until is null or locked_until<now())
      order by tab_name for update skip locked limit 1;
  else
    -- Refreshes: today and yesterday first, then whichever has waited longest.
    select * into r from group.member_report_days
      where meeting_day<=today and next_refresh_at<=now() and (locked_until is null or locked_until<now())
      order by (meeting_day >= today - 1) desc, next_refresh_at, meeting_day desc, tab_name for update skip locked limit 1;
  end if;
  if not found or r.meeting_day is null then return null; end if;
  update group.member_report_days set lock_token=p_token, locked_until=now()+interval '5 minutes', claimed_revision=revision,
    next_refresh_at=now()+interval '10 minutes'
    where meeting_day=r.meeting_day and target_key=r.target_key;
  return to_jsonb(r)||jsonb_build_object('lock_token',p_token);
end
```

### member_report_day_dispatch() -> void
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 1 lines
```sql
declare cmd text; begin if exists(select 1 from group.member_report_days where next_refresh_at<=now() and (locked_until is null or locked_until<now())) then select command into cmd from cron.job where jobname='group-member-sheet-report' and active limit 1; if cmd is not null then execute cmd; end if; end if; end
```

### member_report_day_finish(p_day date, p_target text, p_token uuid, p_result jsonb, p_success boolean) -> boolean
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 12 lines
```sql
begin
  update group.member_report_days set locked_until=null, lock_token=null, last_result=p_result,
    last_success_at=case when p_success then now() else last_success_at end,
    next_refresh_at=case
      when p_success and revision<>claimed_revision then now()
      when p_success and p_day >= (now() at time zone 'Asia/Kolkata')::date - 1 then now()+interval '2 hours'
      when p_success and p_day >= (now() at time zone 'Asia/Kolkata')::date - 14 then now()+interval '1 day'
      when p_success then now()+interval '7 days'
      else now()+interval '5 minutes' end
  where meeting_day=p_day and target_key=p_target and lock_token=p_token and locked_until>now();
  return found;
end
```

### member_report_email_dirty() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 4 lines
```sql
declare day date; begin
  select meeting_date into day from group.meetings where id=case when TG_OP='DELETE' then old.meeting_id else new.meeting_id end;
  update group.member_report_days set next_refresh_at=least(next_refresh_at, now()+interval '3 minutes'), revision=revision+1 where meeting_day=day;
  return null; end
```

### member_report_meeting_dirty() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 4 lines
```sql
begin
  update group.member_report_days set next_refresh_at=least(next_refresh_at, now()+interval '3 minutes'), revision=revision+1
  where meeting_day=case when TG_OP='DELETE' then old.meeting_date else new.meeting_date end or (TG_OP='UPDATE' and meeting_day=old.meeting_date);
  return null; end
```

### member_sheet_report_kick() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 1 lines
```sql
declare cmd text; begin if not exists(select 1 from group.member_sheet_reports where enabled) then return null; end if; if not pg_try_advisory_xact_lock(20461101,1) then return null; end if; update group.member_sheet_reports set last_result=last_result||jsonb_build_object('last_kick_at',now()) where enabled and coalesce((last_result->>'last_kick_at')::timestamptz,'epoch'::timestamptz)<now()-interval '30 seconds'; if not found then return null; end if; select command into cmd from cron.job where jobname='group-meetings-sheet-sync' and active limit 1; if cmd is not null then execute replace(cmd,'/functions/v1/meetings-sheet-sync','/functions/v1/member-sheet-report'); end if; return null; end
```

### member_sheet_report_lock(p_member uuid, p_token uuid) -> boolean
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 1 lines
```sql
begin update group.member_sheet_reports set lock_token=p_token,locked_until=now()+interval '10 minutes' where member_id=p_member and enabled and (locked_until is null or locked_until<now()); return found; end
```

### member_sheet_report_touch() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, pg_temp | exec anon=n authenticated=n | 1 lines
```sql
begin new.updated_at=now(); return new; end
```

### member_utm_credit(p_from date, p_to date) -> TABLE(user_id uuid, applications integer, paid_students integer)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 1 lines
```sql
 BEGIN IF NOT group.is_admin_or_owner(auth.uid()) THEN RAISE EXCEPTION 'Not authorised'; END IF; RETURN QUERY SELECT c.utm_member_id,count(*) FILTER(WHERE (c.applied_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to)::integer,count(*) FILTER(WHERE (c.paid_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to)::integer FROM group.core_applications c WHERE c.utm_member_id IS NOT NULL AND ((c.applied_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to OR (c.paid_at AT TIME ZONE 'Asia/Kolkata')::date BETWEEN p_from AND p_to) GROUP BY 1; END 
```

### merge_school(_source text, _target text, _reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 38 lines
```sql

declare m int; a int; al int; src record; tgt record;
begin
  if not group.is_admin_or_owner(auth.uid()) then
    raise exception 'not authorised';
  end if;
  if _source = _target then raise exception 'source and target must differ'; end if;
  select * into src from group.school_registry where code = _source;
  select * into tgt from group.school_registry where code = _target;
  if src is null or tgt is null then raise exception 'unknown school code'; end if;

  update group.meetings set school_code = _target, school_match_source = 'merged', school_match_confidence = 1.0, school_match_reviewed = true
  where school_code = _source;
  get diagnostics m = row_count;

  update group.applications set school_code = _target, match_source = 'merged', match_confidence = 1.0, match_reviewed = true
  where school_code = _source;
  get diagnostics a = row_count;

  update group.school_aliases set school_code = _target, match_source = 'merged', match_confidence = 1.0
  where school_code = _source
    and not exists (select 1 from group.school_aliases s2 where s2.school_code = _target and s2.raw_text = group.school_aliases.raw_text);
  get diagnostics al = row_count;
  delete from group.school_aliases where school_code = _source;

  insert into group.school_aliases (raw_text, school_code, match_confidence, match_source, times_seen)
  values (group.norm_school_name(src.canonical_name), _target, 1.0, 'merged', 1)
  on conflict (raw_text) do update set school_code = _target, match_confidence = 1.0, match_source = 'merged';

  insert into group.school_merge_log (source_code, target_code, reason, confidence, rows_repointed)
  values (_source, _target, coalesce(_reason, 'manual merge'), 1.0,
          jsonb_build_object('meetings', m, 'applications', a, 'aliases', al));

  delete from group.school_registry where code = _source;

  return jsonb_build_object('ok', true, 'meetings', m, 'applications', a, 'aliases', al,
                            'source', src.canonical_name, 'target', tgt.canonical_name);
end 
```

### metrics_by_member(p_from date, p_to date) -> TABLE(user_id uuid, member_name text, meetings integer, held integer, no_shows integer, apps_attributed integer, paid_attributed integer, converted integer, conv_pct numeric)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

begin
  if not group.is_active_staff(auth.uid()) then raise exception 'not authorised'; end if;
  return query select * from group.metrics_by_member__impl(p_from, p_to);
end 
```

### metrics_by_member__impl(p_from date, p_to date) -> TABLE(user_id uuid, member_name text, meetings integer, held integer, no_shows integer, apps_attributed integer, paid_attributed integer, converted integer, conv_pct numeric)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 5 lines
```sql

  select s.user_id, s.member_name, s.meetings, s.held, s.no_shows,
         s.applications, s.paid_students, s.converted, s.conv_pct
  from group.owner_member_scorecard(p_from, p_to) s;

```

### metrics_funnel(p_from date, p_to date) -> TABLE(stage text, sort smallint, value integer)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

begin
  if not group.is_active_staff(auth.uid()) then raise exception 'not authorised'; end if;
  return query select * from group.metrics_funnel__impl(p_from, p_to);
end 
```

### metrics_funnel__impl(p_from date, p_to date) -> TABLE(stage text, sort smallint, value integer)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 22 lines
```sql

WITH entered AS (
  SELECT d.id,
         d.stage,
         min((event ->> 'at')::timestamptz)::date AS entered_at
  FROM group.deals d
  LEFT JOIN LATERAL jsonb_array_elements(coalesce(d.stage_history, '[]'::jsonb)) event ON true
  WHERE d.merged_into IS NULL
  GROUP BY d.id, d.stage
)
SELECT l.label,
       l.sort::smallint,
       count(*) FILTER (
         WHERE e.entered_at BETWEEN p_from AND p_to
           AND e.stage >= l.value::smallint
       )::int
FROM group.sop_lists l
LEFT JOIN entered e ON true
WHERE l.list = 'stage' AND l.active
GROUP BY l.label, l.sort, l.value
ORDER BY l.sort;

```

### metrics_summary(p_from date, p_to date) -> TABLE(meetings integer, held integer, no_shows integer, no_show_pct numeric, held_pct numeric, apps_attributed integer, paid_attributed integer, apps_total integer, paid_total integer, converted integer, conv_pct numeric)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

begin
  if not group.is_active_staff(auth.uid()) then raise exception 'not authorised'; end if;
  return query select * from group.metrics_summary__impl(p_from, p_to);
end 
```

### metrics_summary__impl(p_from date, p_to date) -> TABLE(meetings integer, held integer, no_shows integer, no_show_pct numeric, held_pct numeric, apps_attributed integer, paid_attributed integer, apps_total integer, paid_total integer, converted integer, conv_pct numeric)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 25 lines
```sql

  with vol as (
    select count(*)::int m,
           count(*) filter (where is_held)::int h,
           count(*) filter (where is_no_show)::int ns
    from group.v_meeting_clean
    where meeting_date between p_from and p_to
  ), agg as (
    select coalesce(sum(applications),0)::int a,
           coalesce(sum(paid_students),0)::int p,
           coalesce(sum(converted),0)::int c
    from group.owner_member_scorecard__impl(p_from, p_to)
  )
  select vol.m, vol.h, vol.ns,
    case when vol.h + vol.ns > 0 then round(100.0*vol.ns/(vol.h+vol.ns),1) else 0 end,
    case when vol.h + vol.ns > 0 then round(100.0*vol.h /(vol.h+vol.ns),1) else 0 end,
    agg.a, agg.p,
    (select count(*)::int from group.core_applications
      where applied_at is not null and applied_at::date between p_from and p_to),
    (select count(*)::int from group.core_applications
      where paid_at is not null and paid_at::date between p_from and p_to),
    agg.c,
    case when vol.h > 0 then round(100.0*agg.c/vol.h,1) else 0 end
  from vol, agg;

```

### my_assigned_meeting_ids() -> SETOF uuid
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 3 lines
```sql

  SELECT meeting_id FROM group.meeting_assignees WHERE user_id = auth.uid()

```

### my_profile_role_active() -> TABLE(role text, is_active boolean)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 3 lines
```sql

  SELECT role::text, is_active FROM group.user_profiles WHERE id = auth.uid()

```

### norm_school_country(p text) -> text
- sql | invoker | immutable | config: search_path=group, pg_temp | exec anon=Y authenticated=Y | 7 lines
```sql

  select case x when '' then null when 'unknown' then null when 'uae' then 'united arab emirates' when 'u.a.e' then 'united arab emirates'
    when 'usa' then 'united states' when 'us' then 'united states' when 'united states of america' then 'united states'
    when 'uk' then 'united kingdom' when 'england' then 'united kingdom' when 'ksa' then 'saudi arabia' when 'türkiye' then 'turkey'
    when 'turkiye' then 'turkey' when 'viet nam' then 'vietnam' when 'korea, republic of' then 'south korea' when 'korea' then 'south korea' else x end
  from (select lower(btrim(coalesce(p,''))) x) s

```

## Schema: group - part 2 of 2

### norm_school_name(t text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 13 lines
```sql

  select nullif(
    btrim(
      regexp_replace(
        regexp_replace(
          regexp_replace(lower(coalesce(t,'')), '&', ' and ', 'g'),
          '[^a-z0-9]+', ' ', 'g'
        ),
        '\s+(ltd|llc|inc|pvt|private limited|co)\s*$', ' ', 'g'
      )
    ), ''
  );

```

### notify_meeting_freed() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 25 lines
```sql

DECLARE m record; v_when text;
BEGIN
  IF EXISTS (SELECT 1 FROM group.meeting_assignees WHERE meeting_id = OLD.meeting_id) THEN
    RETURN OLD;
  END IF;
  SELECT * INTO m FROM group.meetings WHERE id = OLD.meeting_id;
  IF m.id IS NULL OR m.meeting_date IS NULL
     OR m.meeting_date < (now() AT TIME ZONE 'Asia/Kolkata')::date
     OR COALESCE(m.meeting_status, 'Scheduled') IN ('Cancelled', 'Rescheduled', 'Completed', 'No Show') THEN
    RETURN OLD;
  END IF;
  v_when := to_char(m.meeting_date, 'DD Mon') || COALESCE(' at ' || to_char(m.scheduled_time, 'HH24:MI'), '');
  INSERT INTO group.notifications (user_id, type, title, body, meeting_id, link_url, severity, dedupe_key)
  SELECT p.id,
         'meeting_unassigned',
         COALESCE('Free again - ' || NULLIF(m.school_name, ''), 'Meeting free again'),
         v_when || ' - this meeting has no owner. Take it if you can.',
         m.id, '/meetings', 'warning',
         'freed:' || m.id || ':' || p.id || ':' || to_char(now(), 'YYYY-MM-DD"T"HH24')
    FROM group.user_profiles p
   WHERE p.is_active AND p.id <> COALESCE(OLD.user_id, '00000000-0000-0000-0000-000000000000'::uuid)
  ON CONFLICT DO NOTHING;
  RETURN OLD;
END 
```

### notify_on_assignment() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 41 lines
```sql

DECLARE
  v_school TEXT;
  v_date   DATE;
  v_time   TIME;
  v_by     TEXT;
  v_by_uuid UUID;
  v_assignee TEXT;
BEGIN
  SELECT school_name, meeting_date, scheduled_time
    INTO v_school, v_date, v_time
    FROM group.meetings WHERE id = NEW.meeting_id;

  SELECT full_name INTO v_assignee FROM group.user_profiles WHERE id = NEW.user_id;

-- ... truncated (41 lines total)
```

### notify_unassigned_meeting() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 25 lines
```sql

DECLARE v_when text;
BEGIN
  IF NEW.meeting_date IS NULL OR NEW.meeting_date < (now() AT TIME ZONE 'Asia/Kolkata')::date THEN
    RETURN NEW;
  END IF;
  IF COALESCE(NEW.meeting_status, 'Scheduled') IN ('Cancelled', 'Rescheduled', 'Completed', 'No Show') THEN
    RETURN NEW;
  END IF;
  v_when := to_char(NEW.meeting_date, 'DD Mon')
            || COALESCE(' at ' || to_char(NEW.scheduled_time, 'HH24:MI'), '');
  INSERT INTO group.notifications (user_id, type, title, body, meeting_id, link_url, severity, dedupe_key)
  SELECT p.id,
         'meeting_unassigned',
         COALESCE('Unassigned meeting - ' || NULLIF(NEW.school_name, ''), 'Unassigned meeting'),
         v_when || ' - nobody has picked this up yet.',
         NEW.id,
         '/meetings',
         'warning',
         'unassigned:' || NEW.id || ':' || p.id
    FROM group.user_profiles p
   WHERE p.is_active
  ON CONFLICT DO NOTHING;
  RETURN NEW;
END 
```

### owner_campaigns(p_from date, p_to date, p_limit integer) -> TABLE(campaign text, meetings integer, applications integer, paid_students integer, meetings_converted integer, conv_pct numeric)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

BEGIN
  IF NOT group.is_active_staff(auth.uid()) THEN
    RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
  END IF;
  RETURN QUERY SELECT * FROM group.owner_campaigns__impl(p_from, p_to, p_limit);
END

```

### owner_campaigns__impl(p_from date, p_to date, p_limit integer) -> TABLE(campaign text, meetings integer, applications integer, paid_students integer, meetings_converted integer, conv_pct numeric)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 15 lines
```sql

  SELECT COALESCE(NULLIF(btrim(m.utm_campaign), ''), 'none')::text,
         count(*)::int,
         COALESCE(sum(l.apps_after), 0)::int,
         COALESCE(sum(l.paid_after), 0)::int,
         count(*) FILTER (WHERE l.outcome_state = 'converted')::int,
         round(100.0 * count(*) FILTER (WHERE l.outcome_state = 'converted')
               / GREATEST(count(*) FILTER (WHERE m.meeting_status = 'Completed'), 1), 1)
  FROM group.meetings m
  LEFT JOIN group.v_meeting_ledger l ON l.meeting_id = m.id
  WHERE m.meeting_date BETWEEN p_from AND p_to
  GROUP BY 1
  ORDER BY 4 DESC, 3 DESC, 2 DESC
  LIMIT GREATEST(p_limit, 1);

```

### owner_country_ledger(p_from date, p_to date) -> TABLE(name text, meetings integer, converted integer, applications integer, paid_students integer, conv_pct numeric)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_admin_or_owner(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_country_ledger__impl(p_from, p_to);
      END
      
```

### owner_country_ledger__impl(p_from date, p_to date) -> TABLE(name text, meetings integer, converted integer, applications integer, paid_students integer, conv_pct numeric)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 11 lines
```sql

SELECT COALESCE(NULLIF(btrim(l.country), ''), 'Not recorded'),
       count(*)::int,
       count(*) FILTER (WHERE l.outcome_state = 'converted')::int,
       COALESCE(sum(l.apps_after),0)::int,
       COALESCE(sum(l.paid_after),0)::int,
       round(100.0 * count(*) FILTER (WHERE l.outcome_state = 'converted') / GREATEST(count(*),1), 1)
FROM group.v_meeting_ledger l
WHERE l.meeting_date BETWEEN p_from AND p_to
GROUP BY 1 ORDER BY 5 DESC, 2 DESC;

```

### owner_cut(p_dim text, p_from date, p_to date) -> TABLE(bucket text, label text, meetings integer, converted integer, applications integer, paid_students integer, conv_pct numeric, revenue jsonb)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_admin_or_owner(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_cut__impl(p_dim, p_from, p_to);
      END

```

### owner_cut__impl(p_dim text, p_from date, p_to date) -> TABLE(bucket text, label text, meetings integer, converted integer, applications integer, paid_students integer, conv_pct numeric, revenue jsonb)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 53 lines
```sql

with l as (
  select ml.meeting_id, ml.meeting_date, ml.country, ml.programme,
         ml.outcome_state, ml.apps_after, ml.paid_after,
         coalesce(nullif(btrim(ml.school_name),''), nullif(btrim(ml.school_code),''), 'Not recorded') skey,
         coalesce(r.label, initcap(replace(coalesce(group.utm_key(m.utm_source),'unrecorded'),'-',' '))) camp
    from group.v_meeting_ledger ml
    join group.meetings m on m.id = ml.meeting_id
    left join group.campaign_registry r on r.utm_key = group.utm_key(m.utm_source)
   where ml.meeting_date between p_from and p_to
), keyed as (
  select case p_dim
           when 'month'     then to_char(l.meeting_date,'YYYY-MM')
           when 'country'   then coalesce(nullif(btrim(l.country),''),'Not recorded')
           when 'programme' then coalesce(nullif(btrim(l.programme),''),'Not recorded')
-- ... truncated (53 lines total)
```

### owner_cut_bucket_meetings(p_dim text, p_bucket text, p_from date, p_to date, p_limit integer) -> TABLE(meeting_id uuid, meeting_date date, lead_name text, school_name text, country text, programme text, campaign text, host_name text, member_name text, lead_segment text, lead_score integer, outcome_state text, apps_after integer, paid_after integer, days_to_pay integer, paid_amounts jsonb)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

BEGIN
  IF NOT group.is_admin_or_owner(auth.uid()) THEN
    RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
  END IF;
  RETURN QUERY SELECT * FROM group.owner_cut_bucket_meetings__impl(p_dim, p_bucket, p_from, p_to, p_limit);
END

```

### owner_cut_bucket_meetings__impl(p_dim text, p_bucket text, p_from date, p_to date, p_limit integer) -> TABLE(meeting_id uuid, meeting_date date, lead_name text, school_name text, country text, programme text, campaign text, host_name text, member_name text, lead_segment text, lead_score integer, outcome_state text, apps_after integer, paid_after integer, days_to_pay integer, paid_amounts jsonb)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 33 lines
```sql

with l as (
  select ml.*, coalesce(r.label,
           initcap(replace(coalesce(group.utm_key(m.utm_source),'unrecorded'),'-',' '))) camp,
         coalesce(nullif(btrim(ml.school_name),''), nullif(btrim(ml.school_code),''), 'Not recorded') skey,
         coalesce((select vm.member_name from group.v_meeting_member vm
                    where vm.meeting_id = ml.meeting_id and vm.is_primary limit 1),'Unassigned') mem
    from group.v_meeting_ledger ml
    join group.meetings m on m.id = ml.meeting_id
    left join group.campaign_registry r on r.utm_key = group.utm_key(m.utm_source)
   where ml.meeting_date between p_from and p_to
), rev as (
  select vr.meeting_id, jsonb_object_agg(vr.currency, vr.revenue_minor order by vr.revenue_minor desc) j
    from group.v_meeting_revenue vr
   group by vr.meeting_id
)
select l.meeting_id, l.meeting_date, l.lead_name, l.school_name, l.country, l.programme,
       l.camp, l.host_name, l.mem, l.lead_segment, l.lead_score::int, l.outcome_state,
       l.apps_after, l.paid_after, l.days_to_pay, coalesce(rev.j, '{}'::jsonb)
  from l
  left join rev on rev.meeting_id = l.meeting_id
 where case p_dim
         when 'month'     then to_char(l.meeting_date,'YYYY-MM') = p_bucket
         when 'country'   then coalesce(nullif(btrim(l.country),''),'Not recorded') = p_bucket
         when 'programme' then coalesce(nullif(btrim(l.programme),''),'Not recorded') = p_bucket
         when 'campaign'  then l.camp = p_bucket
         when 'school'    then l.skey = p_bucket
         when 'member'    then l.mem = p_bucket
         when 'outcome'   then coalesce(l.outcome_state,'no_outcome') = p_bucket
         else true end
 order by l.meeting_date desc, l.lead_name
 limit greatest(p_limit, 1);

```

### owner_cut_meetings(p_dim text, p_bucket text, p_from date, p_to date, p_limit integer) -> TABLE(meeting_id uuid, meeting_date date, lead_name text, school_name text, country text, programme text, host_name text, campaign text, outcome_state text, apps_after integer, paid_after integer, days_to_pay integer, analysed boolean, whatsapp_captured boolean, next_meeting_agreed boolean, paid_amounts jsonb)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_admin_or_owner(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_cut_meetings__impl(p_dim, p_bucket, p_from, p_to, p_limit);
      END

```

### owner_cut_meetings__impl(p_dim text, p_bucket text, p_from date, p_to date, p_limit integer) -> TABLE(meeting_id uuid, meeting_date date, lead_name text, school_name text, country text, programme text, host_name text, campaign text, outcome_state text, apps_after integer, paid_after integer, days_to_pay integer, analysed boolean, whatsapp_captured boolean, next_meeting_agreed boolean, paid_amounts jsonb)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 34 lines
```sql

with l as (
  select ml.*, coalesce(r.label,
           initcap(replace(coalesce(group.utm_key(m.utm_source),'unrecorded'),'-',' '))) camp,
         coalesce(nullif(btrim(ml.school_name),''), nullif(btrim(ml.school_code),''), 'Not recorded') skey
    from group.v_meeting_ledger ml
    join group.meetings m on m.id = ml.meeting_id
    left join group.campaign_registry r on r.utm_key = group.utm_key(m.utm_source)
   where ml.meeting_date between p_from and p_to
), rev as (
  select vr.meeting_id, jsonb_object_agg(vr.currency, vr.revenue_minor order by vr.revenue_minor desc) j
    from group.v_meeting_revenue vr
   group by vr.meeting_id
)
select l.meeting_id, l.meeting_date, l.lead_name, l.school_name, l.country, l.programme,
       l.host_name, l.camp, l.outcome_state, l.apps_after, l.paid_after, l.days_to_pay,
       l.analysed, l.whatsapp_captured, l.next_meeting_agreed,
       coalesce(rev.j, '{}'::jsonb)
  from l
  left join rev on rev.meeting_id = l.meeting_id
 where case p_dim
         when 'month'     then to_char(l.meeting_date,'YYYY-MM') = p_bucket
         when 'country'   then coalesce(nullif(btrim(l.country),''),'Not recorded') = p_bucket
         when 'programme' then coalesce(nullif(btrim(l.programme),''),'Not recorded') = p_bucket
         when 'campaign'  then l.camp = p_bucket
         when 'school'    then l.skey = p_bucket
         when 'member'    then exists (select 1 from group.v_member_meeting vm
                                        where vm.meeting_id = l.meeting_id
                                          and vm.member_name = p_bucket)
         when 'outcome'   then l.outcome_state = p_bucket
         else true end
 order by (l.paid_after > 0) desc, (l.apps_after > 0) desc, l.meeting_date desc
 limit greatest(p_limit, 1);

```

### owner_cut_sub(p_dim text, p_bucket text, p_sub_dim text, p_from date, p_to date) -> TABLE(bucket text, label text, meetings integer, converted integer, applications integer, paid_students integer, conv_pct numeric, revenue jsonb)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

BEGIN
  IF NOT group.is_admin_or_owner(auth.uid()) THEN
    RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
  END IF;
  RETURN QUERY SELECT * FROM group.owner_cut_sub__impl(p_dim, p_bucket, p_sub_dim, p_from, p_to);
END

```

### owner_cut_sub__impl(p_dim text, p_bucket text, p_sub_dim text, p_from date, p_to date) -> TABLE(bucket text, label text, meetings integer, converted integer, applications integer, paid_students integer, conv_pct numeric, revenue jsonb)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 68 lines
```sql

with l as (
  select ml.meeting_id, ml.meeting_date, ml.country, ml.programme,
         ml.outcome_state, ml.apps_after, ml.paid_after,
         coalesce(nullif(btrim(ml.school_name),''), nullif(btrim(ml.school_code),''), 'Not recorded') skey,
         coalesce(r.label, initcap(replace(coalesce(group.utm_key(m.utm_source),'unrecorded'),'-',' '))) camp
    from group.v_meeting_ledger ml
    join group.meetings m on m.id = ml.meeting_id
    left join group.campaign_registry r on r.utm_key = group.utm_key(m.utm_source)
   where ml.meeting_date between p_from and p_to
), f as (
  select l.* from l
   where case p_dim
           when 'month'     then to_char(l.meeting_date,'YYYY-MM') = p_bucket
           when 'country'   then coalesce(nullif(btrim(l.country),''),'Not recorded') = p_bucket
-- ... truncated (68 lines total)
```

### owner_data_gaps() -> TABLE(ord integer, gap text, n integer, detail text, href text)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_admin_or_owner(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_data_gaps__impl();
      END
      
```

### owner_data_gaps__impl() -> TABLE(ord integer, gap text, n integer, detail text, href text)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 62 lines
```sql

with unmatched as (
  select regexp_replace(lower(btrim(school_name)), '[^a-z0-9]+', ' ', 'g') nm, count(*) c
  from group.meetings
  where merged_into is null
    and school_code is null
    and nullif(btrim(school_name), '') is not null
    and btrim(regexp_replace(lower(btrim(school_name)), '[^a-z0-9]+', ' ', 'g'))
        not in ('', 'na', 'n a', 'group', 'test', 'none', 'nil')
  group by 1
), ranked as (
  select c, row_number() over (order by c desc) rk from unmatched
)
select 1, 'Meetings not yet analysed',
       count(*) filter (where not analysed)::int,
-- ... truncated (62 lines total)
```

### owner_funnel(p_from date, p_to date) -> TABLE(ord integer, stage text, n integer, tone text)
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 9 lines
```sql

with o as (select * from group.mv_meeting_outcomes where meeting_date between p_from and p_to)
select 1, 'Booked',              (select count(*) from o)::int, 'blue'
union all select 2, 'Held',      (select count(*) from o where meeting_status='Completed')::int, 'blue'
union all select 3, 'Applied after', (select count(*) from o where meeting_status='Completed' and apps_after>0)::int, 'teal'
union all select 4, 'Paid after',    (select count(*) from o where meeting_status='Completed' and paid_after>0)::int, 'teal'
union all select 5, 'Already in pipeline', (select count(*) from o where meeting_status='Completed' and outcome_state='in_pipeline')::int, 'grey'
order by 1;

```

### owner_kpis(p_from date, p_to date) -> TABLE(meetings_held integer, no_shows integer, no_show_pct numeric, meetings_with_app integer, applications_after integer, paid_students_after integer, app_rate_pct numeric, paid_rate_pct numeric, median_days_to_apply integer, median_days_to_pay integer, paid_students_total integer, paid_students_met integer, roi_pct numeric)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_admin_or_owner(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_kpis__impl(p_from, p_to);
      END
      
```

### owner_kpis__impl(p_from date, p_to date) -> TABLE(meetings_held integer, no_shows integer, no_show_pct numeric, meetings_with_app integer, applications_after integer, paid_students_after integer, app_rate_pct numeric, paid_rate_pct numeric, median_days_to_apply integer, median_days_to_pay integer, paid_students_total integer, paid_students_met integer, roi_pct numeric)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 40 lines
```sql

with held as (
  select * from group.mv_meeting_outcomes
   where meeting_status='Completed' and meeting_date between p_from and p_to),
allm as (
  select meeting_status from group.mv_meeting_outcomes
   where meeting_date between p_from and p_to),
paidset as (
  select c.id, c.student_email, c.school_code, c.paid_at
    from group.core_applications c
   where c.paid_at::date between p_from and p_to),
met as (
  select p.id from paidset p
    join group.meetings m on lower(btrim(m.email)) = lower(p.student_email)
   where m.meeting_status='Completed' and m.meeting_date is not null
     and m.meeting_date <= p.paid_at::date
  union
  select p.id from paidset p
    join group.meetings m on m.school_code = p.school_code
   where p.school_code is not null and m.meeting_status='Completed'
     and m.meeting_date is not null and m.meeting_date <= p.paid_at::date)
select
  (select count(*) from held)::int,
  (select count(*) from allm where meeting_status='No Show')::int,
  round(100.0*(select count(*) from allm where meeting_status='No Show')
        / nullif((select count(*) from allm where meeting_status in ('Completed','No Show')),0),1),
  (select count(*) from held where apps_after>0)::int,
  (select coalesce(sum(apps_after),0) from held)::int,
  (select coalesce(sum(paid_after),0) from held)::int,
  round(100.0*(select count(*) from held where apps_after>0)/nullif((select count(*) from held),0),1),
  round(100.0*(select coalesce(sum(paid_after),0) from held)
        / nullif((select coalesce(sum(apps_after),0) from held),0),1),
  (select percentile_cont(0.5) within group (order by extract(day from first_app_at - meeting_date::timestamptz))
     from held where first_app_at is not null)::int,
  (select percentile_cont(0.5) within group (order by extract(day from first_paid_at - meeting_date::timestamptz))
     from held where first_paid_at is not null)::int,
  (select count(*) from paidset)::int,
  (select count(*) from met)::int,
  round(100.0*(select count(*) from met)/nullif((select count(*) from paidset),0),1);

```

### owner_member_scorecard(p_from date, p_to date) -> TABLE(user_id uuid, member_name text, meetings integer, held integer, no_shows integer, converted integer, applications integer, paid_students integer, conv_pct numeric, median_days_to_pay integer, followups_sent integer, replies_in integer, tasks_open integer, tasks_done integer, followthrough_pct numeric)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_active_staff(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_member_scorecard__impl(p_from, p_to);
      END
      
```

### owner_member_scorecard__impl(p_from date, p_to date) -> TABLE(user_id uuid, member_name text, meetings integer, held integer, no_shows integer, converted integer, applications integer, paid_students integer, conv_pct numeric, median_days_to_pay integer, followups_sent integer, replies_in integer, tasks_open integer, tasks_done integer, followthrough_pct numeric)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 50 lines
```sql

WITH base AS (
  SELECT vm.user_id AS uid, vm.member_name AS mname, m.id AS mid, m.meeting_status AS mstatus
  FROM group.v_meeting_clean m
  JOIN group.v_meeting_member vm ON vm.meeting_id = m.id AND vm.is_primary
  WHERE m.meeting_date BETWEEN p_from AND p_to
), joined AS (
  SELECT b.uid, b.mname, b.mid, b.mstatus,
         l.outcome_state, l.apps_after, l.paid_after, l.days_to_pay
  FROM base b
  LEFT JOIN group.v_meeting_ledger l ON l.meeting_id = b.mid
), em AS (
  SELECT b.uid,
         count(*) FILTER (WHERE e.direction <> 'inbound' AND e.status = 'sent')::int AS out_n,
         count(*) FILTER (WHERE e.direction = 'inbound')::int AS in_n
-- ... truncated (50 lines total)
```

### owner_payment_summary(p_from date, p_to date) -> TABLE(payments integer, from_meeting integer, no_meeting integer, revenue jsonb, revenue_from_meeting jsonb)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

BEGIN
  IF NOT group.is_admin_or_owner(auth.uid()) THEN
    RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
  END IF;
  RETURN QUERY SELECT * FROM group.owner_payment_summary__impl(p_from, p_to);
END

```

### owner_payment_summary__impl(p_from date, p_to date) -> TABLE(payments integer, from_meeting integer, no_meeting integer, revenue jsonb, revenue_from_meeting jsonb)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 25 lines
```sql

with v as (
  select * from group.v_payment_attribution
   where occurred_at >= p_from::timestamptz and occurred_at < (p_to + 1)::timestamptz
), unlinked_bank_refunds as (
  select currency, -sum(amount_minor) amt from group.core_bank_refunds
   where payment_id is null and refunded_on >= p_from and refunded_on <= p_to
   group by 1
), all_rev as (
  select jsonb_object_agg(currency, amt order by amt desc) j
    from (select currency, sum(amt) amt
            from (select currency, sum(net_minor) amt from v group by 1
                  union all select currency, amt from unlinked_bank_refunds) u
           group by 1) x
), meet_rev as (
  select jsonb_object_agg(currency, amt order by amt desc) j
    from (select currency, sum(net_minor) amt from v where from_meeting group by 1) y
)
select count(*)::int,
       count(*) filter (where from_meeting)::int,
       count(*) filter (where not from_meeting)::int,
       coalesce((select j from all_rev), '{}'::jsonb),
       coalesce((select j from meet_rev), '{}'::jsonb)
  from v;

```

### owner_payments(p_from date, p_to date, p_scope text, p_limit integer) -> TABLE(payment_id uuid, occurred_at timestamp with time zone, currency text, amount_minor bigint, net_minor bigint, status text, payer_name text, payer_email text, programme_code text, method text, meeting_id uuid, meeting_date date, lead_name text, school_name text, host_name text, days_to_pay integer, from_meeting boolean, match_route text)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

begin
  if not group.is_admin_or_owner(auth.uid()) then
    raise exception 'Not authorised' using errcode = '42501';
  end if;
  return query select * from group.owner_payments__impl(p_from, p_to, p_scope, p_limit);
end

```

### owner_payments__impl(p_from date, p_to date, p_scope text, p_limit integer) -> TABLE(payment_id uuid, occurred_at timestamp with time zone, currency text, amount_minor bigint, net_minor bigint, status text, payer_name text, payer_email text, programme_code text, method text, meeting_id uuid, meeting_date date, lead_name text, school_name text, host_name text, days_to_pay integer, from_meeting boolean, match_route text)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 13 lines
```sql

  select v.payment_id, v.occurred_at, v.currency, v.amount_minor, v.net_minor, v.status,
         v.payer_name, v.payer_email, v.programme_code, v.method, v.meeting_id, v.meeting_date,
         v.lead_name, v.school_name, v.host_name, v.days_to_pay::int, v.from_meeting, v.match_route
    from group.v_payment_attribution v
   where v.occurred_at >= p_from::timestamptz
     and v.occurred_at < (p_to + 1)::timestamptz
     and (p_scope = 'all'
          or (p_scope = 'meeting' and v.from_meeting)
          or (p_scope = 'none' and not v.from_meeting))
   order by v.occurred_at desc
   limit greatest(p_limit, 1);

```

### owner_programme_ledger(p_from date, p_to date) -> TABLE(name text, meetings integer, converted integer, applications integer, paid_students integer, conv_pct numeric)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_admin_or_owner(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_programme_ledger__impl(p_from, p_to);
      END
      
```

### owner_programme_ledger__impl(p_from date, p_to date) -> TABLE(name text, meetings integer, converted integer, applications integer, paid_students integer, conv_pct numeric)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 11 lines
```sql

SELECT COALESCE(NULLIF(btrim(l.programme), ''), 'Not recorded'),
       count(*)::int,
       count(*) FILTER (WHERE l.outcome_state = 'converted')::int,
       COALESCE(sum(l.apps_after),0)::int,
       COALESCE(sum(l.paid_after),0)::int,
       round(100.0 * count(*) FILTER (WHERE l.outcome_state = 'converted') / GREATEST(count(*),1), 1)
FROM group.v_meeting_ledger l
WHERE l.meeting_date BETWEEN p_from AND p_to
GROUP BY 1 ORDER BY 5 DESC, 2 DESC;

```

### owner_pulse(p_months integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_admin_or_owner(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN group.owner_pulse__impl(p_months);
      END
      
```

### owner_pulse__impl(p_months integer) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 90 lines
```sql

with months as (
  select generate_series(
           date_trunc('month', current_date) - ((greatest(p_months, 1) - 1) || ' months')::interval,
           date_trunc('month', current_date),
           interval '1 month')::date ms
),
since as (select min(ms) d from months),
mt as (
  select id, meeting_date, scheduled_time, meeting_status, host_name, host_email
  from group.meetings
  where merged_into is null
),
led as (
  select meeting_id, apps_after, paid_after from group.v_meeting_ledger
-- ... truncated (90 lines total)
```

### owner_roi_trend(p_months integer) -> TABLE(ym text, month_start date, held integer, applications integer, paid_students integer, converted integer, conv_pct numeric, analysed integer, wa_pct numeric)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_active_staff(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_roi_trend__impl(p_months);
      END
      
```

### owner_roi_trend__impl(p_months integer) -> TABLE(ym text, month_start date, held integer, applications integer, paid_students integer, converted integer, conv_pct numeric, analysed integer, wa_pct numeric)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 25 lines
```sql

with months as (
  select generate_series(
           date_trunc('month', current_date) - ((greatest(p_months,1) - 1) || ' months')::interval,
           date_trunc('month', current_date), interval '1 month')::date ms
), l as materialized (
  select date_trunc('month', v.meeting_date)::date ms, v.meeting_id, v.apps_after, v.paid_after,
         v.outcome_state, v.analysed, v.whatsapp_captured
    from group.v_meeting_ledger v
   where v.meeting_date >= (select min(ms) from months)
)
select to_char(m.ms, 'Mon YYYY'), m.ms,
       count(l.meeting_id)::int,
       coalesce(sum(l.apps_after),0)::int,
       coalesce(sum(l.paid_after),0)::int,
       count(*) filter (where l.outcome_state = 'converted')::int,
       round(100.0 * count(*) filter (where l.outcome_state='converted')
             / nullif(count(l.meeting_id),0), 1),
       count(*) filter (where l.analysed)::int,
       round(100.0 * count(*) filter (where l.whatsapp_captured)
             / nullif(count(*) filter (where l.analysed),0), 1)
  from months m
  left join l on l.ms = m.ms
 group by m.ms order by m.ms;

```

### owner_school_ledger(p_limit integer) -> TABLE(school_code text, school_name text, country text, meetings integer, last_meeting date, hosts text, applications integer, paid_students integer, programmes integer, first_app date, last_app date, status text)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_admin_or_owner(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_school_ledger__impl(p_limit);
      END
      
```

### owner_school_ledger__impl(p_limit integer) -> TABLE(school_code text, school_name text, country text, meetings integer, last_meeting date, hosts text, applications integer, paid_students integer, programmes integer, first_app date, last_app date, status text)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 26 lines
```sql

with mtg as (
  select school_code, count(*)::int n, max(meeting_date) last_m,
         string_agg(distinct host_name, ', ') hosts
    from group.v_meeting_ledger where school_code is not null group by school_code
), app as (
  select school_code, count(*)::int n,
         count(*) filter (where paid_at is not null)::int paid,
         count(distinct programme_code)::int progs,
         min(applied_at)::date f, max(applied_at)::date l
    from group.core_applications where school_code is not null group by school_code
)
select r.code, r.canonical_name, r.country,
       coalesce(mtg.n,0), mtg.last_m, mtg.hosts,
       coalesce(app.n,0), coalesce(app.paid,0), coalesce(app.progs,0),
       app.f, app.l,
       case when coalesce(app.n,0) = 0 then 'no applications'
            when coalesce(app.progs,0) >= 2 then 'returning'
            else 'first programme' end
  from group.school_registry r
  left join mtg on mtg.school_code = r.code
  left join app on app.school_code = r.code
 where coalesce(mtg.n,0) > 0 or coalesce(app.n,0) > 0
 order by coalesce(app.paid,0) desc, coalesce(app.n,0) desc, coalesce(mtg.n,0) desc
 limit greatest(p_limit, 1);

```

### owner_source_ledger(p_from date, p_to date) -> TABLE(name text, medium text, campaign text, meetings integer, held integer, converted integer, applications integer, paid_students integer, conv_pct numeric)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

      BEGIN
        IF NOT group.is_active_staff(auth.uid()) THEN
          RAISE EXCEPTION 'Not authorised' USING ERRCODE = '42501';
        END IF;
        RETURN QUERY SELECT * FROM group.owner_source_ledger__impl(p_from, p_to);
      END
      
```

### owner_source_ledger__impl(p_from date, p_to date) -> TABLE(name text, medium text, campaign text, meetings integer, held integer, converted integer, applications integer, paid_students integer, conv_pct numeric)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 16 lines
```sql

SELECT COALESCE(NULLIF(btrim(m.utm_source), ''), 'Not recorded'),
       COALESCE(NULLIF(btrim(m.utm_medium), ''), 'none'),
       COALESCE(NULLIF(btrim(m.utm_campaign), ''), 'none'),
       count(*)::int,
       count(*) FILTER (WHERE m.meeting_status = 'Completed')::int,
       count(*) FILTER (WHERE l.outcome_state = 'converted')::int,
       COALESCE(sum(l.apps_after),0)::int,
       COALESCE(sum(l.paid_after),0)::int,
       round(100.0 * count(*) FILTER (WHERE l.outcome_state = 'converted')
             / GREATEST(count(*) FILTER (WHERE m.meeting_status = 'Completed'),1), 1)
FROM group.meetings m
LEFT JOIN group.v_meeting_ledger l ON l.meeting_id = m.id
WHERE m.meeting_date BETWEEN p_from AND p_to
GROUP BY 1,2,3 ORDER BY 8 DESC, 4 DESC;

```

### owner_summary(p_from date, p_to date) -> TABLE(held integer, held_prev integer, held_delta integer, applications integer, applications_prev integer, applications_delta integer, paid integer, paid_prev integer, paid_delta integer, paid_students_total integer, paid_students_met integer, roi_pct numeric, unpaid_apps integer, stalled_recent integer, no_show_pct numeric, median_days_to_pay integer, paid_rate_pct numeric, has_outcomes boolean)
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 21 lines
```sql

with span as (select (p_to - p_from) d),
prev as (select (p_from - (select d from span) - 1)::date f, (p_from - 1)::date t),
cur  as (select * from group.owner_kpis(p_from, p_to)),
pre  as (select * from group.owner_kpis((select f from prev), (select t from prev))),
act as (
  select count(*) filter (where apps_after > 0 and paid_after = 0
                            and meeting_date <= current_date - 7)::int unpaid,
         count(*) filter (where coalesce(lead_score,0) >= 75 and apps_after = 0
                            and paid_after = 0
                            and meeting_date between current_date - 30 and current_date - 14)::int stalled
    from group.mv_meeting_outcomes where meeting_status='Completed')
select c.meetings_held, p.meetings_held, c.meetings_held - p.meetings_held,
       c.applications_after, p.applications_after, c.applications_after - p.applications_after,
       c.paid_students_after, p.paid_students_after, c.paid_students_after - p.paid_students_after,
       c.paid_students_total, c.paid_students_met, c.roi_pct,
       (select unpaid from act), (select stalled from act),
       c.no_show_pct, c.median_days_to_pay, c.paid_rate_pct,
       (c.applications_after > 0 or c.paid_students_after > 0)
  from cur c cross join pre p;

```

### person_emails(p_email text) -> TABLE(id uuid, direction text, kind text, subject text, status text, at timestamp with time zone, open_count integer, first_opened_at timestamp with time zone, click_count integer, replied_at timestamp with time zone, error text, meeting_id uuid)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 15 lines
```sql

  select s.id, coalesce(s.direction, 'outbound'),
         coalesce(s.scenario_slug, s.template_slug, 'email'), s.subject, s.status,
         coalesce(s.sent_at, s.received_at, s.created_at),
         coalesce(s.open_count, 0), s.first_opened_at, coalesce(s.click_count, 0), s.replied_at,
         s.error_message, s.meeting_id
    from group.email_sends s
   where nullif(btrim(p_email), '') is not null
     and (lower(btrim(s.to_email)) = lower(btrim(p_email)) or lower(btrim(s.from_email)) = lower(btrim(p_email)))
     and (group.is_admin_or_owner(auth.uid())
          or exists (select 1 from group.meetings m
                      where lower(btrim(m.email)) = lower(btrim(p_email))
                        and group.is_assigned_to_meeting(m.id, auth.uid())))
   order by 6 desc

```

### propagate_assignee_to_tasks() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 10 lines
```sql

BEGIN
  UPDATE group.tasks
     SET assigned_to = NEW.user_id
   WHERE meeting_id = NEW.meeting_id
     AND assigned_to IS NULL
     AND status IN ('open','pending','snoozed','needs_info','revision_requested');
  RETURN NEW;
END;

```

### pull_bank_refunds_from_core() -> integer
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 18 lines
```sql

declare v int;
begin
  insert into group.core_bank_refunds (id, provider, programme_code, currency, amount_minor, refunded_on,
                                        payment_id, link_basis, needs_review, synced_at, pulled_at)
  select v.id, v.provider, v.programme_code, v.currency, v.amount_minor, v.refunded_on,
         v.payment_id, v.link_basis, v.needs_review, v.synced_at, now()
    from core_ro.v_puller_bank_refunds v
  on conflict (id) do update set
    provider = excluded.provider, programme_code = excluded.programme_code, currency = excluded.currency,
    amount_minor = excluded.amount_minor, refunded_on = excluded.refunded_on, payment_id = excluded.payment_id,
    link_basis = excluded.link_basis, needs_review = excluded.needs_review, synced_at = excluded.synced_at, pulled_at = now();
  get diagnostics v = row_count;
  -- The table is small; drop anything Group Core no longer has.
  delete from group.core_bank_refunds b
   where not exists (select 1 from core_ro.v_puller_bank_refunds v where v.id = b.id);
  return v;
end 
```

### pull_from_core(p_full boolean) -> TABLE(apps_pulled bigint, pays_pulled bigint, lwl_updated bigint, schools_resolved bigint)
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 74 lines
```sql

declare v_app_from timestamptz; v_pay_from timestamptz;
        v_apps bigint; v_pays bigint; v_lwl bigint; v_sch bigint;
begin
  select case when p_full then '-infinity'::timestamptz
              else coalesce(last_seen,'-infinity') - interval '10 minutes' end
    into v_app_from from group.core_pull_state where scope='applications';
  v_app_from := coalesce(v_app_from,'-infinity');
  select case when p_full then '-infinity'::timestamptz
              else coalesce(last_seen,'-infinity') - interval '10 minutes' end
    into v_pay_from from group.core_pull_state where scope='payments';
  v_pay_from := coalesce(v_pay_from,'-infinity');

  insert into group.core_applications
  select v.*, now() from core_ro.v_puller_applications v where v.synced_at > v_app_from
-- ... truncated (74 lines total)
```

### record_zoom_participant(p_event text, p_zoom_meeting_id text, p_participant_uuid text, p_zoom_user_id text, p_display_name text, p_email text, p_at timestamp with time zone) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 48 lines
```sql

declare v_meeting uuid; v_staff uuid; v_kind text; v_id uuid; v_auto boolean := false; v_hits uuid[];
begin
  v_meeting := group.meeting_for_zoom_id(p_zoom_meeting_id);
  v_kind  := group.zoom_classify_participant(p_display_name, p_email);
  v_staff := group.zoom_resolve_staff(p_display_name, p_email);
  if v_meeting is null and p_event = 'joined' and v_staff is null and v_kind <> 'staff'
     and nullif(btrim(p_email), '') is not null and nullif(btrim(p_zoom_meeting_id), '') is not null then
    select array_agg(m.id) into v_hits
      from group.meetings m
     where lower(btrim(m.email)) = lower(btrim(p_email))
       and m.start_time_utc between p_at - interval '4 hours' and p_at + interval '30 minutes'
       and coalesce(m.meeting_status, '') not in ('Cancelled', 'Rescheduled')
       and m.zoom_meeting_id is distinct from p_zoom_meeting_id;
    if cardinality(v_hits) = 1 then
-- ... truncated (48 lines total)
```

### recovery_send_blocked(p_meeting_id uuid) -> text
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 15 lines
```sql

  WITH m AS (SELECT id, lower(trim(email)) em, meeting_date FROM meetings WHERE id = p_meeting_id)
  SELECT CASE
    WHEN (SELECT em FROM m) IS NULL OR (SELECT em FROM m) = '' THEN 'no_email'
    WHEN group.is_internal_email((SELECT em FROM m)) THEN 'internal'
    WHEN EXISTS (SELECT 1 FROM email_suppression s, m WHERE lower(s.email) = m.em) THEN 'suppressed'
    WHEN EXISTS (SELECT 1 FROM meetings x, m WHERE lower(trim(x.email)) = m.em AND x.id <> m.id
                 AND x.meeting_date > m.meeting_date) THEN 'booked_again'
    WHEN EXISTS (SELECT 1 FROM scheduled_emails q JOIN meetings x ON x.id = q.meeting_id, m
                 WHERE lower(trim(x.email)) = m.em AND q.status IN ('sent','sending')) THEN 'recovery_already_sent'
    WHEN EXISTS (SELECT 1 FROM email_sends e, m WHERE lower(e.to_email) = m.em AND e.status = 'sent'
                 AND e.created_at > now() - interval '3 days'
                 AND coalesce(e.scenario_slug,'') NOT ILIKE '%thank%') THEN 'emailed_recently'
    ELSE NULL END

```

### register_calendly_questions(_rows jsonb) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 28 lines
```sql

DECLARE n integer := 0;
BEGIN
  INSERT INTO group.calendly_question_registry AS r
    (question_norm, question_label, event_type_name, target_field, status, source,
     sample_answer, last_seen)
  SELECT
    x.question_norm, x.question_label, x.event_type_name, x.target_field,
    COALESCE(x.status, 'pending'), x.source, x.sample_answer,
    COALESCE(x.last_seen, now())
  FROM jsonb_to_recordset(_rows) AS x(
    question_norm text, question_label text, event_type_name text,
    target_field text, status text, source text, sample_answer text,
    last_seen timestamptz)
  WHERE x.question_norm IS NOT NULL AND x.question_norm <> ''
  ON CONFLICT (question_norm) DO UPDATE SET
    hits = r.hits + 1,
    last_seen = GREATEST(r.last_seen, EXCLUDED.last_seen),
    question_label = EXCLUDED.question_label,
    event_type_name = COALESCE(EXCLUDED.event_type_name, r.event_type_name),
    sample_answer = COALESCE(r.sample_answer, EXCLUDED.sample_answer),
    target_field = CASE WHEN r.status = 'pending' THEN EXCLUDED.target_field ELSE r.target_field END,
    status = CASE WHEN r.status = 'pending' THEN EXCLUDED.status ELSE r.status END,
    source = CASE WHEN r.status = 'pending' THEN EXCLUDED.source ELSE r.source END;
  GET DIAGNOSTICS n = ROW_COUNT;
  RETURN n;
END;

```

### release_meeting(_meeting_id uuid) -> void
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 26 lines
```sql

DECLARE
  v_uid uuid := auth.uid();
  v_at timestamptz;
  v_n int;
BEGIN
  IF v_uid IS NULL THEN RAISE EXCEPTION 'Not signed in'; END IF;
  SELECT count(*) INTO v_n FROM group.meeting_assignees WHERE meeting_id = _meeting_id;
  IF v_n <> 1 THEN RAISE EXCEPTION 'Only a sole owner can hand a meeting back'; END IF;
  SELECT assigned_at INTO v_at FROM group.meeting_assignees
   WHERE meeting_id = _meeting_id AND user_id = v_uid;
  IF v_at IS NULL THEN RAISE EXCEPTION 'This meeting is not yours'; END IF;

  -- is_admin_or_owner takes a uuid. It was called with no arguments, and
  -- Postgres does not guarantee short-circuit evaluation of AND, so the missing
  -- signature was resolved on every call regardless of the 30-minute test —
  -- which made handing a meeting back fail for everyone, admins included.
  IF v_at < now() - interval '30 minutes' AND NOT group.is_admin_or_owner(v_uid) THEN
    RAISE EXCEPTION 'The 30 minute window has passed - ask an admin to move it';
  END IF;

  DELETE FROM group.meeting_assignees WHERE meeting_id = _meeting_id AND user_id = v_uid;
  UPDATE group.meetings
     SET assignment_confirmed_at = NULL, assignment_confirmed_by = NULL
   WHERE id = _meeting_id;
END 
```

### release_recovery_backlog(p_limit integer) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 26 lines
```sql

DECLARE n int;
BEGIN
  -- Do not release more while a previous batch is still waiting to send.
  IF EXISTS (SELECT 1 FROM scheduled_emails WHERE status IN ('scheduled','sending')
             AND cancelled_reason LIKE 'backlog_release%') THEN RETURN 0; END IF;

  WITH cand AS (
    SELECT DISTINCT ON (lower(trim(mt.email))) q.id
    FROM scheduled_emails q JOIN meetings mt ON mt.id = q.meeting_id
    WHERE q.status = 'cancelled' AND q.cancelled_reason LIKE 'stale_backlog%'
      AND ((q.email_type = 'no_show_recovery' AND mt.meeting_status = 'No Show')
        OR (q.email_type = 'cancelled_recovery' AND mt.meeting_status = 'Cancelled'))
      AND group.recovery_send_blocked(mt.id) IS NULL
    ORDER BY lower(trim(mt.email)), mt.meeting_date DESC
  ), pick AS (
    SELECT c.id FROM cand c JOIN scheduled_emails q ON q.id = c.id
    JOIN meetings mt ON mt.id = q.meeting_id
    ORDER BY mt.meeting_date DESC LIMIT greatest(1, least(p_limit, 25))
  )
  UPDATE scheduled_emails s SET status = 'scheduled', send_after = now(),
         cancelled_reason = 'backlog_release ' || to_char(now(), 'YYYY-MM-DD HH24:MI')
  FROM pick WHERE s.id = pick.id;
  GET DIAGNOSTICS n = ROW_COUNT;
  RETURN n;
END 
```

### resolve_booking_rep(_email text) -> uuid
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 13 lines
```sql

  SELECT p.id
  FROM group.user_profiles p
  WHERE _email IS NOT NULL
    AND p.email IS NOT NULL
    AND lower(p.email) = lower(
      CASE lower(btrim(_email))
        WHEN 'om@example-group.com' THEN 'omkar@example-group.com'
        ELSE btrim(_email)
      END
    )
  LIMIT 1

```

### resolve_school_codes() -> TABLE(meetings_resolved integer, apps_resolved integer)
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 20 lines
```sql

declare v_m integer := 0; v_a integer := 0;
begin
  with al as (
    select lower(btrim(raw_text)) n, school_code
      from group.school_aliases where school_code is not null
  ), upd as (
    update group.meetings m set school_code = al.school_code
      from al
     where m.school_code is null and m.school_name is not null
       and al.n = regexp_replace(regexp_replace(lower(btrim(m.school_name)),
             '[.,\-/\\#!$%^&*;:{}=_~()]+',' ','g'),'\s+',' ','g')
    returning 1
  ) select count(*) into v_m from upd;

  v_a := group.apply_trusted_application_schools();

  return query select v_m, v_a;
end;

```

### run_health_checks() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp, cron | exec anon=n authenticated=n | 112 lines
```sql

declare
  r record; n int := 0;
  ist_now timestamp := now() at time zone 'Asia/Kolkata';
  v_sent int; v_failed int; v_err text; v_paused boolean; v_x int; v_t timestamptz; v_txt text; v_last_err text;
  v_rec int;
begin
  create temp table if not exists _hc (k text, label text, status text, detail text) on commit drop;
  truncate _hc;

  select count(*) filter (where status = 'sent'), count(*) filter (where status = 'failed'),
         (array_agg(left(error_message, 160) order by created_at desc) filter (where status = 'failed'))[1]
    into v_sent, v_failed, v_err
    from email_sends where created_at > now() - interval '24 hours' and coalesce(direction, 'outbound') = 'outbound';
  select not coalesce(auto_enabled, true) into v_paused from email_settings limit 1;
-- ... truncated (112 lines total)
```

### school_code_for(name text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 7 lines
```sql

  select left(regexp_replace(
    upper(array_to_string(array(
      select left(x,4) from unnest(string_to_array(group.norm_school_name(name),' ')) x
    ), '-')),
  '[^A-Z0-9-]','','g'), 20);

```

### school_domain_review() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 36 lines
```sql

declare result jsonb;
begin
  if auth.uid() is not null and not group.is_active_staff(auth.uid()) then
    raise exception 'Staff only';
  end if;

  select jsonb_build_object(
    'known_domains', (select count(*) from group.school_domains where kind = 'school' and sightings >= 2),
    'districts',     (select count(*) from group.school_domains where kind = 'multi_school'),
    'fillable', coalesce((select jsonb_agg(x) from (
        select jsonb_build_object(
          'meeting_id', m.id, 'lead', m.full_name, 'typed', m.school_name,
          'domain', group.email_domain(m.email),
          'should_be', r.canonical_name, 'code', r.code) as x
        from group.meetings m
        join group.school_domains d on d.domain = group.email_domain(m.email)
        join group.school_registry r on r.code = d.school_code
        where m.school_code is null and d.kind = 'school' and d.sightings >= 2
        limit 200) a), '[]'::jsonb),
    'contradicted', coalesce((select jsonb_agg(x) from (
        select jsonb_build_object(
          'meeting_id', m.id, 'lead', m.full_name, 'typed', m.school_name,
          'domain', group.email_domain(m.email),
          'filed_as', cur.canonical_name, 'domain_says', r.canonical_name,
          'code', r.code, 'sightings', d.sightings) as x
        from group.meetings m
        join group.school_domains d on d.domain = group.email_domain(m.email)
        join group.school_registry r on r.code = d.school_code
        left join group.school_registry cur on cur.code = m.school_code
        where m.school_code is not null and m.school_code <> d.school_code
          and d.kind = 'school' and d.sightings >= 2
        limit 200) b), '[]'::jsonb)
  ) into result;
  return result;
end 
```

### school_domains_learn() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 74 lines
```sql

declare v_school int := 0; v_multi int := 0; v_single int := 0;
begin
  with seen as (
    select group.email_domain(m.email) as domain, m.school_code, count(*)::int as n
    from group.meetings m
    join group.school_registry r on r.code = m.school_code
    where m.email is not null and m.school_code is not null
    group by 1, 2
  ),
  usable as (
    select s.* from seen s
    where s.domain is not null and s.domain like '%.%'
      and not exists (
        select 1 from group.school_domains g
-- ... truncated (74 lines total)
```

### school_duplicate_candidates() -> TABLE(code_a text, name_a text, usage_a bigint, code_b text, name_b text, usage_b bigint, sim double precision)
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 19 lines
```sql

  with usage as (
    select r.code,
           (select count(*) from group.meetings m where m.school_code = r.code)
           + (select count(*) from group.applications l where l.school_code = r.code) as n
    from group.school_registry r
  )
  select a.code, a.canonical_name, ua.n,
         b.code, b.canonical_name, ub.n,
         public.similarity(group.norm_school_name(a.canonical_name), group.norm_school_name(b.canonical_name))::double precision
  from group.school_registry a
  join group.school_registry b on a.code < b.code
  join usage ua on ua.code = a.code
  join usage ub on ub.code = b.code
  where a.is_active and b.is_active
    and group.school_names_agree(a.canonical_name, b.canonical_name)
  order by 7 desc, 3 + 6 desc
  limit 300

```

### school_from_email(p_email text) -> text
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 9 lines
```sql

  select d.school_code
  from group.school_domains d
  where d.domain = group.email_domain(p_email)
    and d.kind = 'school'
    and d.school_code is not null
    and d.sightings >= 2
  limit 1

```

### school_name_shape(p text, OUT dist text, OUT gen text) -> record
- sql | invoker | immutable | config: search_path=group, pg_temp | exec anon=Y authenticated=Y | 7 lines
```sql

  with t as (select unnest(group.school_name_tokens(p)) w),
  g as (select string_to_array('the of and de la le les el del des du school schools high secondary senior junior middle primary elementary international academy college colegio college lycee escuela ecole scuola schule gymnasium institute institucion education educational centre center campus private public state national global world group prep preparatory day boarding girls boys co ed coed english british american community model new modern upper lower grammar academic study learning in at for a an y e i', ' ') a,
              string_to_array('the of and school schools high secondary senior private de la el del in at a an y e i', ' ') f)
  select (select string_agg(distinct w, ' ' order by w) from t, g where not (w = any(g.a)) and length(w) > 1),
         (select string_agg(distinct w, ' ' order by w) from t, g where w = any(g.a) and not (w = any(g.f)))

```

### school_name_tokens(p text) -> text[]
- sql | invoker | immutable | config: search_path=group, pg_temp | exec anon=Y authenticated=Y | 9 lines
```sql

  select coalesce(array_agg(case w when 'hs' then 'high' when 'intl' then 'international' when 'int' then 'international'
      when 'sch' then 'school' when 'acad' then 'academy' when 'sec' then 'secondary' when 'sr' then 'senior' when 'jr' then 'junior'
      when 'st' then 'saint' when 'mt' then 'mount' when 'govt' then 'government' when 'pvt' then 'private' when 'eng' then 'english'
      when 'amer' then 'american' when 'coll' then 'college' when 'univ' then 'university' else w end), '{}')
  from regexp_split_to_table(btrim(regexp_replace(translate(lower(coalesce(p,'')),
      'áàâäãåéèêëíìîïóòôöõúùûüñçýÿ','aaaaaaeeeeiiiiooooouuuuncyy'), '[^a-z0-9 ]', ' ', 'g')), '\s+') w
  where w <> ''

```

### school_names_agree(a text, b text) -> boolean
- plpgsql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 46 lines
```sql

declare
  na text; nb text; ta text[]; tb text[]; sa text[]; sb text[];
  short_sig text[]; long_all text[]; long_sig text[]; hit int := 0; tok text; need int; n int;
  extras int := 0;
begin
  na := group.norm_school_name(a);
  nb := group.norm_school_name(b);
  if na is null or nb is null then return false; end if;
  if na = nb or replace(na,' ','') = replace(nb,' ','') then return true; end if;

  ta := string_to_array(na,' ');
  tb := string_to_array(nb,' ');
  sa := array(select x from unnest(ta) x where not (x = any(group._school_generic_tokens())));
  sb := array(select x from unnest(tb) x where not (x = any(group._school_generic_tokens())));
-- ... truncated (46 lines total)
```

### school_resolve_name(_raw text) -> TABLE(code text, canonical_name text, exact boolean, sim double precision)
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 11 lines
```sql

  select r.code, r.canonical_name,
         (group.norm_school_name(_raw) = group.norm_school_name(r.canonical_name)) as exact,
         public.similarity(group.norm_school_name(_raw), group.norm_school_name(r.canonical_name))::double precision as sim
  from group.school_registry r
  where r.is_active
    and group.norm_school_name(_raw) is not null
    and group.school_names_agree(_raw, r.canonical_name)
  order by 3 desc, 4 desc
  limit 5

```

### set_lead_type(_meeting_id uuid, _group text) -> text
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 39 lines
```sql

DECLARE g text; d text; e text; s text; c text;
BEGIN
  IF NOT group.is_active_staff(auth.uid()) THEN
    RAISE EXCEPTION 'not allowed';
  END IF;

  IF _group IS NULL THEN
    SELECT designation, email, school_name, school_code INTO d, e, s, c
      FROM group.meetings WHERE id = _meeting_id;
    g := group.classify_lead_group(d, e, s, c);
    UPDATE group.meetings
       SET lead_type_locked = false,
           lead_group = g,
           lead_type = group.lead_type_of_group(g),
           lead_segment = group.lead_segment_of_group(g),
           lead_type_source = 'rule',
           lead_type_set_by = NULL,
           lead_type_at = now()
     WHERE id = _meeting_id;
    RETURN g;
  END IF;

  IF _group NOT IN ('student','parent','school','university','corporate','partner','other') THEN
    RAISE EXCEPTION 'invalid lead group %', _group;
  END IF;

  UPDATE group.meetings
     SET lead_type_locked = true,
         lead_group = _group,
         lead_type = group.lead_type_of_group(_group),
         lead_segment = group.lead_segment_of_group(_group),
         lead_type_source = 'manual',
         lead_type_set_by = auth.uid(),
         lead_type_at = now()
   WHERE id = _meeting_id;
  RETURN _group;
END;

```

### sheet_apply_assignments(p_since date, p_dry_run boolean) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 44 lines
```sql

declare v_pairs int; v_new int := 0; v_unresolved int;
begin
  create temp table _pairs on commit drop as
  select distinct s.matched_meeting_id as meeting_id,
         group.sheet_resolve_owner(nm) as user_id,
         coalesce(m.start_time_utc,
                  (m.meeting_date + coalesce(m.scheduled_time,'00:00'::time)) at time zone 'Asia/Kolkata',
                  s.mdate::timestamptz) as at
  from group.sheet_import_stage s
  join group.meetings m on m.id = s.matched_meeting_id
  cross join lateral (
    select btrim(x) as nm from jsonb_array_elements_text(
      case when jsonb_typeof(s.owners) = 'array' and jsonb_array_length(s.owners) > 0
           then s.owners
-- ... truncated (44 lines total)
```

### sheet_resolve_owner(p_name text) -> uuid
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 24 lines
```sql

declare n text; hit uuid; c int;
begin
  -- Owners are typed by hand: full names, bare first names, and honorifics
  -- ("Gunjan Ma'am", "Shubham Sir").
  n := btrim(regexp_replace(coalesce(p_name,''), '(?i)\s*\b(ma.?am|sir|madam)\b\.?', '', 'g'));
  if n = '' then return null; end if;

  select id into hit from group.user_profiles
   where is_active and lower(btrim(full_name)) = lower(n) limit 1;
  if hit is not null then return hit; end if;

  -- A bare first name only counts when exactly one active person answers to it.
  -- Guessing between two would put the wrong name on a meeting.
  select count(*) into c from group.user_profiles
   where is_active and lower(split_part(btrim(full_name),' ',1)) = lower(split_part(n,' ',1));
  if c = 1 then
    select id into hit from group.user_profiles
     where is_active and lower(split_part(btrim(full_name),' ',1)) = lower(split_part(n,' ',1)) limit 1;
    return hit;
  end if;

  return null;
end 
```

### sm_alias_norm(t text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 2 lines
```sql

  select regexp_replace(regexp_replace(lower(btrim(t)), '[.,\-/\\#!$%^&*;:{}=_~()]+', ' ', 'g'), '\s+', ' ', 'g') 
```

### sm_generic_domain(d text) -> boolean
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select coalesce(d, '') = '' or d ~ '^(gmail|googlemail|yahoo|ymail|rocketmail|hotmail|outlook|live|msn|icloud|me|mac|aol|verizon|comcast|att|sbcglobal|protonmail|proton|pm|gmx|mail|email|rediffmail|zoho|yandex|qq|163|126|naver|hanmail|daum|orange|free|wanadoo|laposte|sfr|btinternet|sky|bigpond|optusnet|shaw|rogers|sympatico|cox|charter|earthlink|bellsouth|frontier|t-online|web|libero|tiscali|virgilio|alice|bol|uol|terra|ig|seznam|wp|o2|interia|mailinator|duck|tutanota)\.'

```

### sm_names_agree(a text, b text) -> boolean
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 10 lines
```sql

  with t as (select group.sm_sig_tokens(a) sa, group.sm_sig_tokens(b) sb),
  small as (select case when cardinality(sa) <= cardinality(sb) then sa else sb end s,
                   (cardinality(sa) > 0 and cardinality(sb) > 0 and (sa <@ sb or sb <@ sa)) contained from t)
  select contained and (
           cardinality(s) >= 2
           or (cardinality(s) = 1 and (select count(*) from group.school_registry r
                                       where r.is_active and s[1] = any(group.sm_sig_tokens(r.canonical_name))) <= 1))
  from small

```

### sm_norm_country(t text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 13 lines
```sql

  select case
    when x is null or x in ('', '?', 'unknown', 'na', 'n a', 'other') then null
    when x in ('uae', 'u a e', 'united arab emirates', 'dubai', 'abu dhabi') then 'united arab emirates'
    when x in ('us', 'usa', 'u s a', 'united states', 'united states of america', 'america') then 'united states'
    when x in ('uk', 'u k', 'united kingdom', 'england', 'great britain') then 'united kingdom'
    when x in ('ksa', 'saudi arabia', 'saaudi arabia', 'saudi') then 'saudi arabia'
    when x in ('cote d ivoire', 'côte d ivoire', 'ivory coast') then 'cote d ivoire'
    when x in ('the bahamas', 'bahamas') then 'bahamas'
    when x in ('mexcio', 'mexico') then 'mexico'
    when x in ('south korea', 'korea', 'republic of korea') then 'south korea'
    else x end
  from (select btrim(regexp_replace(regexp_replace(lower(btrim(t)), '[.,\-/\\#!$%^&*;:{}=_~()'']+', ' ', 'g'), '\s+', ' ', 'g')) x) s 
```

### sm_norm_name(t text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 2 lines
```sql

  select nullif(btrim(regexp_replace(regexp_replace(lower(btrim(t)), '[.,\-/\\#!$%^&*;:{}=_~()'']+', ' ', 'g'), '\s+', ' ', 'g')), '') 
```

### sm_sig_tokens(t text) -> text[]
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 6 lines
```sql

  select coalesce(array_agg(distinct w), '{}') from regexp_split_to_table(coalesce(group.sm_norm_name(t), ''), ' ') w
  where length(w) >= 2 and w not in ('school','schools','international','intl','academy','college','high','secondary','senior','public','private',
    'the','of','de','la','le','el','al','and','campus','institute','education','educational','world','global','group','lycee','lycée','colegio',
    'escola','ecole','école','sr','sec','higher','sch','for','girls','boys','in','at','a','an','du','des','da','do','di','y','e','und')

```

### sop_actor() -> uuid
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 3 lines
```sql

  select p.id from group.user_profiles p where p.id = auth.uid() and p.is_active

```

### sop_campaign_tag_ok(p_tag text) -> boolean
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 3 lines
```sql

  select p_tag is not null and p_tag ~ '^[a-z0-9_]+(-[a-z0-9_]+){3}$';

```

### sop_is_red(p_status text, p_stage smallint, p_last timestamp with time zone, p_created timestamp with time zone) -> boolean
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 4 lines
```sql

  select p_status = 'active' and p_stage between 1 and 5
     and coalesce(p_last, p_created) < now() - interval '14 days';

```

### sop_map_designation(p_raw text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 16 lines
```sql

  select case
    when p_raw is null or btrim(p_raw) in ('','.','Group') then null
    when p_raw ilike 'principal%'                                     then 'principal'
    when p_raw ilike '%head of school%' or p_raw ilike 'head%'
      or p_raw ilike '%director%'                                     then 'head_of_school'
    when p_raw ilike '%counsel%'                                      then 'counsellor'
    when p_raw ilike 'school rep%' or p_raw ilike '%teacher%'
      or p_raw ilike '%coordinator%' or p_raw ilike 'school'
      or p_raw ilike '%practitioner%'                                 then 'teacher_coordinator'
    when p_raw ilike 'parent%'                                        then 'parent'
    when p_raw ilike 'student%'                                       then 'student'
    when p_raw ilike 'universit%'                                     then 'university'
    when p_raw ilike 'partner%' or p_raw ilike 'corporate%'           then 'partner_agency'
    else null end;

```

### sop_map_programme(p_raw text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 37 lines
```sql

  select case
    when p_raw is null then null
    -- short codes used by the application feed
    when upper(btrim(p_raw)) = 'ONWD' then 'onwards'
    when upper(btrim(p_raw)) = 'SLC'  then 'fellowship_research'
    when upper(btrim(p_raw)) = 'CMFL' then 'fellowship_capital_minds'
    when upper(btrim(p_raw)) = 'SATI' then 'sat_instinct'
    when upper(btrim(p_raw)) = 'FDF'  then 'fellowship_doctors'
    when upper(btrim(p_raw)) = 'FEDG' then 'future_edge'
    when upper(btrim(p_raw)) = 'FLF'  then 'fellowship_lawyers'
    when upper(btrim(p_raw)) = 'HCUS' then 'hcus_day_camp'
    -- long names used by the meeting history
    when p_raw ilike '%future edge%' then 'future_edge'
    when p_raw ilike '%sir luther%' then 'fellowship_research'
    when p_raw ilike '%fellowship%' then 'onwards'
    when p_raw ilike '%capital minds%' or p_raw ilike '%finedu%' then 'fellowship_capital_minds'
    when p_raw ilike '%hcus%' or p_raw ilike '%harvard summer%' then 'hcus_day_camp'
    when p_raw ilike '%harvard%winter%' then 'harvard_winter'
    when p_raw ilike '%lawyer%' then 'fellowship_lawyers'
    when p_raw ilike '%testprep%' then 'sat_instinct'
    when p_raw ilike '%hpair%' then 'hpair_ai'
    when p_raw ilike '%hwics%' then 'hwics'
    when p_raw ilike '%nyu%' then 'nyu_entrepreneurship'
    when p_raw ilike '%doctor%' then 'fellowship_doctors'
    when p_raw ilike '%research%' then 'fellowship_research'
    when p_raw ilike '%hmun%' then 'hmun'
    when p_raw ilike '%cambridge%' then 'cambridge_premed'
    when p_raw ilike '%geneva%' then 'geneva'
    when p_raw ilike '%salamanca%' then 'salamanca'
    when p_raw ilike '%wolf%' or p_raw ilike '%wow%' then 'competition_wow'
    when p_raw ilike '%huso%' then 'competition_huso'
    when p_raw ilike '%yale%' then 'competition_yale_essay'
    when p_raw ilike '%ai leader%' then 'fellowship_ai_leaders'
    when p_raw ilike '%entrepreneur%' then 'fellowship_entrepreneurs'
    else null end;

```

### sop_meeting_held(p_status text) -> boolean
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select lower(coalesce(p_status,'')) in ('completed','done','held');

```

### sop_metrics(p_from date, p_to date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 46 lines
```sql

declare d_to date := coalesce(p_to, current_date);
        d_from date := coalesce(p_from, coalesce(p_to, current_date) - 89);
begin
  if group.sop_actor() is null then raise exception 'not authorised'; end if;
  return (
    with live as (select * from group.deals where merged_into is null)
    select jsonb_build_object(
    'window', jsonb_build_object('from', d_from, 'to', d_to),
    'meeting_done_rate', (
      select case when count(*) filter (where lower(coalesce(meeting_status,''))
                     in ('completed','done','held','no show','cancelled')) = 0 then null
             else round(100.0*count(*) filter (where group.sop_meeting_held(meeting_status))
               / count(*) filter (where lower(coalesce(meeting_status,''))
                 in ('completed','done','held','no show','cancelled'))) end
-- ... truncated (46 lines total)
```

### sop_routes_to(p_designation text) -> text
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 4 lines
```sql

  select coalesce((select l.extra->>'routes_to' from group.sop_lists l
                    where l.list='designation' and l.value = group.sop_map_designation(p_designation)), 'unknown');

```

### sop_sla_days(p_value text, p_default integer) -> integer
- sql | invoker | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  SELECT COALESCE((SELECT (extra->>'days')::int FROM sop_lists WHERE list='sla' AND value=p_value AND active LIMIT 1), p_default)

```

### sop_sweep() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 102 lines
```sql

declare made int := 0; n int := 0; woke int := 0; det jsonb := '{}'::jsonb;
begin
  with c(day_no, label, prio) as (values
    (4,'Day 4 touch: short check-in, one question','med'),
    (10,'Day 10 touch: second email with one new piece of value','med'),
    (18,'Day 18 touch: direct ask for a yes, no or a date','high')),
  ins as (
    insert into group.tasks (type, sop_kind, title, description, deal_id, assigned_to,
      due_at, due_date, priority, status, created_by_agent, sop_key)
    select 'schedule_followup','cadence_day'||c.day_no, c.label||' - '||d.school_name,
           'SOP 3.4 follow-up cadence, day '||c.day_no||' after the proposal.',
           d.id, d.owner, (d.proposal_sent_date + c.day_no)::timestamptz,
           (d.proposal_sent_date + c.day_no), c.prio::group.task_priority, 'pending', true,
           'cadence:'||d.id::text||':day'||c.day_no
-- ... truncated (102 lines total)
```

### sop_sweep_all() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 3 lines
```sql

  select jsonb_build_object('core', group.sop_sweep(), 'tracks', group.sop_sweep_tracks());

```

### sop_sweep_tracks() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 86 lines
```sql

declare made int := 0; n int := 0; det jsonb := '{}'::jsonb;
begin
  with ins as (
    insert into group.tasks (type, sop_kind, title, description, deal_id, assigned_to,
      due_at, due_date, priority, status, created_by_agent, sop_key)
    select 'manual','ops_kickoff','Operations kick-off due for '||d.school_name,
           'SOP 5.1.1: brochure with school logo, registration link, mentor availability.',
           d.id, d.owner, d.stage_entered_at + interval '3 days',
           (d.stage_entered_at + interval '3 days')::date,'high','pending', true,
           'kickoff:'||d.id::text
      from group.deals d
     where d.status='active' and d.stage=4 and d.stream in ('in_person','hosting')
       and d.stage_entered_at < now() - interval '3 days' and d.registration_opened_at is null
    on conflict (sop_key) do nothing returning 1)
-- ... truncated (86 lines total)
```

### sop_touch_rag(p_last timestamp with time zone, p_created timestamp with time zone) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

  select case when extract(day from now() - coalesce(p_last,p_created))::int >= 14 then 'red'
              when extract(day from now() - coalesce(p_last,p_created))::int >= 8 then 'amber'
              else 'green' end;

```

### sync_lead_segment_from_designation() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 9 lines
```sql

BEGIN
  IF TG_OP = 'INSERT' OR NEW.designation IS DISTINCT FROM OLD.designation THEN
    IF NEW.lead_segment IS NULL OR NEW.lead_segment = '' OR NEW.lead_segment = 'unknown' THEN
      NEW.lead_segment := group.derive_lead_segment(NEW.designation);
    END IF;
  END IF;
  RETURN NEW;
END 
```

### sync_lead_type_from_designation() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 21 lines
```sql

DECLARE g text;
BEGIN
  IF coalesce(NEW.lead_type_locked, false) THEN
    RETURN NEW;
  END IF;

  g := group.classify_lead_group(NEW.designation, NEW.email, NEW.school_name, NEW.school_code);

  IF g = 'unknown' AND NEW.lead_type_source = 'ai' AND NEW.lead_group IS NOT NULL THEN
    RETURN NEW;
  END IF;

  NEW.lead_group := g;
  NEW.lead_type := group.lead_type_of_group(g);
  NEW.lead_type_source := 'rule';
  NEW.lead_type_at := now();
  NEW.lead_segment := group.lead_segment_of_group(g);
  RETURN NEW;
END;

```

### sync_lifecycle_from_status() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 22 lines
```sql

BEGIN
  IF NEW.meeting_status = 'Completed'
     AND OLD.meeting_status IS DISTINCT FROM 'Completed'
     AND NEW.lifecycle_stage IN ('new_lead', 'meeting_scheduled') THEN
    NEW.lifecycle_stage = 'meeting_done';
  END IF;

  IF NEW.meeting_status = 'Scheduled'
     AND NEW.lifecycle_stage = 'new_lead' THEN
    NEW.lifecycle_stage = 'meeting_scheduled';
  END IF;

  IF NEW.conversion_status = 'Converted'
     AND (OLD.conversion_status IS NULL
          OR OLD.conversion_status <> 'Converted') THEN
    NEW.lifecycle_stage = 'converted';
  END IF;

  RETURN NEW;
END;

```

### sync_meeting_assigned_to(_meeting_id uuid) -> void
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 24 lines
```sql

DECLARE
  _name text;
BEGIN
  select p.full_name into _name
  from group.meeting_assignees a
  join group.user_profiles p on p.id = a.user_id
  left join lateral (
    select sum(extract(epoch from (coalesce(mp.left_at, mp.joined_at)) - mp.joined_at)) as secs
    from group.meeting_participants mp
    where mp.meeting_id = a.meeting_id and mp.staff_user_id = a.user_id
  ) d on true
  where a.meeting_id = _meeting_id
  order by coalesce(d.secs, -1) desc, a.assigned_at asc nulls last
  limit 1;

  PERFORM set_config('app.assignee_sync', 'on', true);
  UPDATE group.meetings
     SET assigned_to = _name
   WHERE id = _meeting_id
     AND assigned_to IS DISTINCT FROM _name;
  PERFORM set_config('app.assignee_sync', 'off', true);
END;

```

### sync_meeting_attribution() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 67 lines
```sql

DECLARE n integer;
BEGIN
  WITH cand_all AS (
    SELECT m.id AS mid, c.id AS aid, c.applied_at, c.paid_at, m.meeting_date
      FROM meetings m
      JOIN core_applications c
        ON lower(c.student_email) = lower(btrim(m.email))
     WHERE COALESCE(m.lead_segment, 'unknown') IN ('student', 'parent', 'unknown')
       AND m.email IS NOT NULL
       AND m.meeting_date IS NOT NULL
       AND c.applied_at >= (m.meeting_date - 90)::timestamptz
       AND c.applied_at <  (m.meeting_date + 90)::timestamptz
    UNION
    SELECT m.id, c.id, c.applied_at, c.paid_at, m.meeting_date
-- ... truncated (67 lines total)
```

### tasks_on_email_draft() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 61 lines
```sql

DECLARE
  v_assignee_id  uuid;
  v_assignee_nm  text;
  v_school       text;
  v_lead         text;
  v_md           date;
  v_prio         task_priority;
  v_due          date;
  v_type         task_type;
  v_title        text;
BEGIN
  IF NEW.status = 'draft' AND COALESCE(NEW.direction,'outbound') = 'outbound' THEN
    SELECT m.school_name, m.full_name, m.meeting_date, m.assigned_to
      INTO v_school, v_lead, v_md, v_assignee_nm
-- ... truncated (61 lines total)
```

### tasks_on_inbound_reply() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 45 lines
```sql

DECLARE
  v_assignee uuid;
  v_lead text;
  v_school text;
BEGIN
  IF COALESCE(NEW.direction, 'outbound') <> 'inbound' THEN
    RETURN NEW;
  END IF;

  SELECT m.full_name, m.school_name
    INTO v_lead, v_school
    FROM group.meetings m
   WHERE m.id = NEW.meeting_id;

-- ... truncated (45 lines total)
```

### tasks_resolve_on_email_change() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 11 lines
```sql

BEGIN
  IF OLD.status = 'draft' AND NEW.status <> 'draft' THEN
    UPDATE group.tasks
       SET status = 'done', completed_at = now()
     WHERE email_send_id = NEW.id
       AND status IN ('open','pending');
  END IF;
  RETURN NEW;
END

```

### tasks_resolve_on_email_delete() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 8 lines
```sql

BEGIN
  UPDATE group.tasks
     SET status='dismissed', completed_at=now()
   WHERE email_send_id = OLD.id
     AND status IN ('open','pending','snoozed');
  RETURN OLD;
END 
```

### tasks_set_completed_at() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 13 lines
```sql

BEGIN
  IF NEW.status IN ('done','dismissed') AND OLD.status IN ('open','pending','snoozed') THEN
    NEW.completed_at := now();
    IF NEW.completed_by IS NULL THEN NEW.completed_by := auth.uid(); END IF;
  END IF;
  IF NEW.status IN ('open','pending') AND OLD.status NOT IN ('open','pending') THEN
    NEW.completed_at := NULL;
    NEW.completed_by := NULL;
  END IF;
  RETURN NEW;
END

```

### touch_followup_updated_at() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 6 lines
```sql

BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;

```

### touch_meeting_welcome_jobs() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 6 lines
```sql

begin
  new.updated_at := now();
  return new;
end;

```

### trg_sync_meeting_assigned_to() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 13 lines
```sql

BEGIN
  IF TG_OP = 'DELETE' THEN
    PERFORM group.sync_meeting_assigned_to(OLD.meeting_id);
    RETURN OLD;
  END IF;
  PERFORM group.sync_meeting_assigned_to(NEW.meeting_id);
  IF TG_OP = 'UPDATE' AND OLD.meeting_id IS DISTINCT FROM NEW.meeting_id THEN
    PERFORM group.sync_meeting_assigned_to(OLD.meeting_id);
  END IF;
  RETURN NEW;
END;

```

### trg_zoom_autoassign() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 11 lines
```sql

begin
  -- Only an identified person assigns a meeting. The shared host account joins
  -- 100% of meetings including every No Show, so it can never own one.
  if NEW.participant_kind = 'staff' and NEW.staff_user_id is not null and NEW.meeting_id is not null then
    perform group.zoom_autoassign(NEW.meeting_id);
  elsif TG_OP = 'UPDATE' and NEW.left_at is distinct from OLD.left_at and NEW.meeting_id is not null then
    perform group.sync_meeting_assigned_to(NEW.meeting_id);
  end if;
  return NEW;
end 
```

### unassigned_health() -> TABLE(open_unassigned integer, avg_hours_to_claim numeric, late_unowned integer)
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 13 lines
```sql

  SELECT
    (SELECT count(*)::int FROM group.meetings m
      WHERE m.meeting_date >= (now() AT TIME ZONE 'Asia/Kolkata')::date
        AND COALESCE(m.meeting_status,'Scheduled') NOT IN ('Cancelled','Rescheduled','Completed','No Show')
        AND NOT EXISTS (SELECT 1 FROM group.meeting_assignees a WHERE a.meeting_id = m.id)),
    (SELECT round(avg(EXTRACT(epoch FROM (a.assigned_at - m.created_at)) / 3600)::numeric, 1)
       FROM group.meeting_assignees a JOIN group.meetings m ON m.id = a.meeting_id
      WHERE a.assigned_at >= now() - interval '30 days'),
    (SELECT count(*)::int FROM group.meetings m
      WHERE m.start_time_utc BETWEEN now() AND now() + interval '1 hour'
        AND NOT EXISTS (SELECT 1 FROM group.meeting_assignees a WHERE a.meeting_id = m.id))

```

### unlinked_application_schools(p_min integer) -> TABLE(school_key text, school_raw text, applications integer, paid integer, latest_applied timestamp with time zone)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 18 lines
```sql

begin
  if not (select group.is_admin_or_owner((select auth.uid()))) then
    raise exception 'Not authorised' using errcode = '42501';
  end if;
  return query
  select k.key, k.raw, k.n, k.p, k.latest from (
    select group.conversion_school_key(c.school_raw) key, mode() within group (order by btrim(c.school_raw)) raw,
           count(*)::int n, count(*) filter (where c.paid_at is not null)::int p, max(c.applied_at) latest
      from group.v_core_applications_unique c
     where c.school_code is null and coalesce(btrim(c.school_raw), '') <> '' and c.applied_at >= '2026-01-01'
     group by 1) k
  where k.n >= greatest(p_min, 1) and k.key <> ''
    and not exists (select 1 from group.school_aliases a
                     where a.raw_text = k.key and (a.match_source = 'human_skip' or (a.school_code is not null and a.match_confidence = 1)))
  order by k.n desc, k.key
  limit 300;
end 
```

### unmatched_application_schools(p_limit integer) -> TABLE(name_key text, raw_name text, country text, city text, emails text[], apps integer, paid integer)
- sql | SECURITY DEFINER | stable | config: search_path=group, pg_temp | exec anon=n authenticated=n | 12 lines
```sql

  select k, min(c.school_raw), mode() within group (order by c.country), mode() within group (order by c.city),
         (array_agg(distinct lower(coalesce(c.student_email, c.parent_email))) filter (where coalesce(c.student_email,c.parent_email) is not null))[1:5],
         count(*)::int, count(*) filter (where c.paid_at is not null)::int
    from group.core_applications c
    cross join lateral (select group.conversion_school_key(c.school_raw) k) x
   where c.school_code is null and nullif(btrim(c.school_raw),'') is not null and k is not null and k <> ''
     and not exists (select 1 from group.application_school_matches a where a.name_key = x.k and a.method <> 'check_failed' and a.created_at > now() - interval '7 days')
   group by k
   order by max(c.applied_at) desc, count(*) filter (where c.paid_at is not null) desc
   limit p_limit

```

### update_updated_at_column() -> trigger
- plpgsql | invoker | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 6 lines
```sql

BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;

```

### utm_key(p text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select nullif(regexp_replace(lower(btrim(coalesce(p,''))), '[^a-z0-9]+', '-', 'g'), '')

```

### utm_member_for_apps(p_ids uuid[]) -> TABLE(id uuid, utm_member_id uuid)
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

begin
  if not group.is_active_staff(auth.uid()) then raise exception 'not authorised'; end if;
  return query select c.id, c.utm_member_id from group.core_applications c where c.id = any(p_ids) and c.utm_member_id is not null;
end 
```

### utm_member_id(p_source text, p_medium text, p_campaign text, p_ref text) -> uuid
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 16 lines
```sql

  with platform as (
    select (lower(concat_ws(' ', p_source, p_medium, p_campaign)) ~ '(lead-mgmt|auto-email)') as is_platform
  ), toks as (
    select distinct t from regexp_split_to_table(lower(concat_ws(' ', p_source, p_medium, p_campaign, p_ref)), '[^a-z]+') t where length(t) >= 2
  ), keys as (
    select p.id, lower(split_part(btrim(p.full_name), ' ', 1)) k from group.user_profiles p
     where coalesce(p.email,'') ilike '%@example-group.com' and length(split_part(btrim(p.full_name),' ',1)) >= 3
    union all
    select p.id, a.alias from group.user_profiles p
      join (values ('om','omkar'),('abhi','abhijeet'),('bhavya','bhaavya')) a(alias, first)
        on lower(split_part(btrim(p.full_name),' ',1)) = a.first
  ), hits as (select distinct k.id from keys k join toks on toks.t = k.k)
  select case when (select is_platform from platform) then null
              when (select count(*) from hits) = 1 then (select id from hits) end

```

### wa_activity(p_days integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 68 lines
```sql

declare result jsonb;
begin
  if auth.uid() is not null and not group.is_active_staff(auth.uid()) then
    raise exception 'Staff only';
  end if;

  select jsonb_build_object(
    'enabled', coalesce((select enabled from group.wa_settings), false),
    'now_ist', to_char(now() at time zone 'Asia/Kolkata','DD Mon HH24:MI'),
    'kinds', jsonb_build_object(
      'daystart_member',      'Morning message to every member',
      'daystart_leader',      'Morning numbers to owners and admins',
      'founder_brief',        'The 8 AM founder brief, owners only',
      'meeting_reminder_5min','Your meeting starts in 5 minutes',
-- ... truncated (68 lines total)
```

### wa_can_send(p_user_id uuid, p_kind text) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 33 lines
```sql

declare v_role text; v_zone text; v_cap int; v_sent int; v_today date;
begin
  select role, coalesce(nullif(btrim(timezone),''),'Asia/Kolkata')
    into v_role, v_zone
  from group.user_profiles where id = p_user_id;

  if v_role is null then
    return jsonb_build_object('allowed', false, 'reason', 'unknown_recipient');
  end if;

  if p_kind = 'team_takenby' then
    return jsonb_build_object('allowed', true, 'reason', 'stand_down_exempt');
  end if;

  select daily_cap into v_cap from group.wa_limits where role = v_role;
  if v_cap is null then
    return jsonb_build_object('allowed', true, 'reason', 'no_cap_for_role');
  end if;

  v_today := (now() at time zone v_zone)::date;

  select count(*) into v_sent
  from group.wa_sends s
  where s.recipient_user_id = p_user_id
    and s.status <> 'failed'
    and (s.created_at at time zone v_zone)::date = v_today;

  return jsonb_build_object(
    'allowed', v_sent < v_cap,
    'reason',  case when v_sent < v_cap then 'within_cap' else 'daily_cap_reached' end,
    'role', v_role, 'cap', v_cap, 'sent_today', v_sent);
end 
```

### wa_claim_from_whatsapp(p_phone text, p_context_id text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 115 lines
```sql

declare
  v_digits      text;
  v_user        uuid;
  v_user_name   text;
  v_role        text;
  v_meeting     uuid;
  v_status      text;
  v_school      text;
  v_contact     text;
  v_when        timestamptz;
  v_holder      uuid;
  v_holder_name text;
  v_mine        boolean;
begin
-- ... truncated (115 lines total)
```

### wa_param(t text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 5 lines
```sql

  select nullif(btrim(regexp_replace(
           regexp_replace(coalesce(t, ''), '[\n\r\t]+', ' ', 'g'),
           ' {4,}', '   ', 'g')), '')

```

### welcome_bot_expire_stale() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 20 lines
```sql

declare n integer;
begin
  with closed as (
    update group.meeting_welcome_jobs j
       set state = case
             when exists (select 1 from group.meeting_participants p
                           where p.meeting_id = j.meeting_id and p.is_staff and p.joined_at is not null)
               then 'handed_over'
             else 'nobody_joined'
           end,
           skip_reason = null,
           last_error = null
     where j.state in ('pending','waiting','rep_alerted','founder_alerted','dispatched')
       and j.join_at < now() - interval '3 hours'
     returning 1
  )
  select count(*) from closed into n;
  return n;
end 
```

### welcome_bot_greeting(p_meeting_id uuid) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=n | 48 lines
```sql

with m as (select * from group.meetings where id = p_meeting_id),
-- Who the bot waits for. Assignees and assigned_to are the canonical takers,
-- but they are set late: 5 of the scheduled meetings have a real host_name and
-- no assignee at all, and the nearest 9 have neither. host_name is therefore
-- folded in — except when it is the shared "the Group" Calendly
-- account, which is a round-robin inbox, not a person, and is also the name the
-- bot itself joins under.
named as (
  select distinct p.full_name as n
  from group.user_profiles p
  where p.id::text in (select user_id::text from group.meeting_assignees where meeting_id = p_meeting_id)
     or p.id::text = (select assigned_to::text from m)
  union
  select host_name from m where host_name is not null
-- ... truncated (48 lines total)
```

### welcome_bot_overview() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 58 lines
```sql

declare result jsonb; nxt uuid;
begin
  if auth.uid() is not null and not group.is_active_staff(auth.uid()) then
    raise exception 'Staff only';
  end if;

  select j.meeting_id into nxt
    from group.meeting_welcome_jobs j
   where j.state in ('waiting','rep_alerted','founder_alerted')
   order by j.lead_seen_at limit 1;

  select jsonb_build_object(
    'enabled', coalesce((select bool_or(active) from cron.job where jobname = 'welcome-bot-dispatch-minutely'), false),
    'scheduled', exists (select 1 from cron.job where jobname = 'welcome-bot-dispatch-minutely'),
-- ... truncated (58 lines total)
```

### welcome_bot_set_enabled(p_on boolean) -> boolean
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 11 lines
```sql

begin
  if not group.is_admin_or_owner(auth.uid()) then
    raise exception 'Only an owner or admin can switch the welcome greeter';
  end if;
  update group.automations
     set state = case when p_on then 'on' else 'off' end,
         updated_by = auth.uid(), updated_at = now()
   where slug = 'welcome_greeter';
  return p_on;
end 
```

### zoom_autoassign(p_meeting_id uuid) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 20 lines
```sql

declare n int := 0;
begin
  if p_meeting_id is null then return 0; end if;

  with ins as (
    insert into group.meeting_assignees (meeting_id, user_id, assigned_by, assigned_at)
    select p.meeting_id, p.staff_user_id, 'zoom-attendance', min(p.joined_at)
    from group.meeting_participants p
    where p.meeting_id = p_meeting_id
      and p.participant_kind = 'staff'
      and p.staff_user_id is not null
    group by p.meeting_id, p.staff_user_id
    on conflict (meeting_id, user_id) do nothing
    returning 1)
  select count(*) into n from ins;

  perform group.sync_meeting_assigned_to(p_meeting_id);
  return n;
end 
```

### zoom_classify_participant(p_name text, p_email text) -> text
- sql | invoker | immutable | config: search_path=group, public, extensions, pg_temp | exec anon=Y authenticated=Y | 18 lines
```sql

  select case
    -- Calendly and Fireflies notetakers. Some carry Cyrillic homoglyphs in
    -- "Nоtеtаkеr", so match on the ASCII vendor words instead.
    when coalesce(p_name,'') ~* '(calendly|notetaker|fireflies|otter\.ai|read\.ai|recording)' then 'bot'
    -- The bare host account, with or without the possessive.
    when btrim(regexp_replace(coalesce(p_name,''), '(?i)''s$', '')) ~* '^learn\s*with\s*leaders$' then 'shared_account'
    when exists (
      select 1 from group.user_profiles p
      where p.is_active
        and length(split_part(btrim(p.full_name),' ',1)) > 3
        and btrim(regexp_replace(coalesce(p_name,''), '(?i)learn\s*with\s*leaders''?s?', '', 'g'))
            ~* ('\y' || split_part(btrim(p.full_name),' ',1) || '\y')
    ) then 'staff'
    when lower(coalesce(p_email,'')) ~ '@(example-group.com|example-group.in|partner-a.com|partner-b.in)$' then 'staff'
    else 'guest'
  end

```

### zoom_resolve_staff(p_name text, p_email text) -> uuid
- sql | SECURITY DEFINER | stable | config: search_path=group, public, extensions, pg_temp | exec anon=n authenticated=Y | 16 lines
```sql

  with cleaned as (
    select btrim(regexp_replace(coalesce(p_name,''), '(?i)learn\s*with\s*leaders''?s?', '', 'g')) as n
  )
  select p.id from group.user_profiles p, cleaned c
  where p.is_active
    and group.zoom_classify_participant(p_name, p_email) = 'staff'
    and (
      lower(btrim(p.email)) = lower(btrim(coalesce(p_email,'')))
      or (length(split_part(btrim(p.full_name),' ',1)) > 3
          and c.n ~* ('\y' || split_part(btrim(p.full_name),' ',1) || '\y'))
    )
  order by (lower(btrim(p.email)) = lower(btrim(coalesce(p_email,'')))) desc,
           length(p.full_name) desc
  limit 1

```

## Schema: public (397 functions) - part 1 of 4

### accept_staff_invitation() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 73 lines
- comment: Runs when an Auth user is created. Applies a live staff_invitations row, or links a staff row an owner already made, or does nothing. It NEVER creates authorisation that an owner did not write down first. Swallows its own errors on purpose: a trigger on auth.users that raises stops all sign-up, including the owner trying to fix it.
```sql

DECLARE
  v_email text := lower(btrim(NEW.email));
  v_inv   staff_invitations%ROWTYPE;
  v_staff uuid;
  v_role  staff_role;
  v_grant staff_grant;
  v_kind  text;
  v_val   text;
  v_meta  jsonb := coalesce(NEW.raw_user_meta_data, '{}'::jsonb);
BEGIN
  IF v_email IS NULL OR v_email = '' THEN RETURN NEW; END IF;

  SELECT * INTO v_inv FROM staff_invitations
   WHERE lower(email) = v_email AND accepted_at IS NULL AND revoked_at IS NULL AND expires_at > now()
-- ... truncated (73 lines total)
```

### access_log(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 18 lines
```sql

BEGIN
  IF NOT (has_role('owner') OR has_role('admin')) THEN RETURN jsonb_build_object('permitted', false); END IF;
  RETURN jsonb_build_object('permitted', true,
    'total', (SELECT count(*) FROM sensitive_access_log),
    'last_7_days', (SELECT count(*) FROM sensitive_access_log WHERE at > now() - interval '7 days'),
    'by_staff', coalesce((SELECT jsonb_agg(x ORDER BY (x->>'views')::int DESC) FROM (
        SELECT jsonb_build_object('staff', coalesce(s.full_name,s.email,'unattributed'),
                 'grant_key', l.grant_key, 'views', count(*),
                 'distinct_subjects', count(DISTINCT l.subject_id), 'last_at', max(l.at)) AS x
          FROM sensitive_access_log l LEFT JOIN staff s ON s.id=l.staff_id
         GROUP BY coalesce(s.full_name,s.email,'unattributed'), l.grant_key) y), '[]'::jsonb),
    'recent', coalesce((SELECT jsonb_agg(x) FROM (
        SELECT jsonb_build_object('at',l.at,'staff',coalesce(s.full_name,s.email,'UNATTRIBUTED'),
                 'grant_key',l.grant_key,'subject_kind',l.subject_kind,'context',l.context) AS x
          FROM sensitive_access_log l LEFT JOIN staff s ON s.id=l.staff_id
         ORDER BY l.at DESC LIMIT greatest(1, least(coalesce(p_limit,100),500))) y), '[]'::jsonb));
END; 
```

### activity_is_person_event(p_type text) -> boolean
- sql | invoker | immutable | config: search_path=pg_catalog, pg_temp | exec anon=Y authenticated=Y | 8 lines
```sql

  SELECT lower(btrim(coalesce(p_type,''))) IN (
    'connect_requested','connect_unavailable','connection_accepted',
    'send_invite','invite_deferred','send_message','reply_interested',
    'draft_created','draft_personalized','draft_ready',
    'draft_approved','draft_auto_approved','draft_lost',
    'lead_disqualified','lead_requalified','lead_held_by_gate','lead_rejected_by_gate');

```

### agent_setting(p_key text) -> numeric
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

BEGIN
  IF auth.role() IN ('anon','authenticated') AND public.auth_staff_id() IS NULL THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE = '42501';
  END IF;
  RETURN (SELECT value FROM public.agent_settings WHERE key = p_key);
END;

```

### agent_setting_set(p_key text, p_value numeric, p_reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 22 lines
```sql

DECLARE me uuid; old_v numeric;
BEGIN
  me := auth_staff_id();
  IF me IS NULL OR NOT (has_role('owner') OR has_role('admin')) THEN
    RAISE EXCEPTION 'Only an owner or admin may change an agent setting' USING ERRCODE='42501'; END IF;
  IF nullif(btrim(coalesce(p_reason,'')),'') IS NULL THEN
    RAISE EXCEPTION 'A setting needs a reason. A number with no reason is a guess, and the next person cannot review it'
      USING ERRCODE='22023'; END IF;

  SELECT value INTO old_v FROM agent_settings WHERE key = p_key;
  IF old_v IS NULL THEN
    RAISE EXCEPTION 'No such setting %. Settings are added by a migration, not by a screen, so that the default and its reasoning are in version control', p_key
      USING ERRCODE='P0002'; END IF;

  UPDATE agent_settings SET value = p_value, reason = p_reason, set_by = me::text, set_at = now()
   WHERE key = p_key;
  INSERT INTO agent_setting_history (key, was, became, reason, changed_by)
  VALUES (p_key, old_v, p_value, p_reason, me::text);

  RETURN jsonb_build_object('ok', true, 'key', p_key, 'was', old_v, 'now', p_value);
END 
```

### agent_settings_list() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 19 lines
```sql

DECLARE out jsonb;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  SELECT jsonb_build_object(
    'settings', coalesce((SELECT jsonb_agg(jsonb_build_object(
        'key', key, 'value', value, 'unit', unit, 'reason', reason,
        'set_by', set_by, 'set_at', set_at,
        'changes', (SELECT count(*) FROM agent_setting_history h WHERE h.key = s.key))
        ORDER BY key) FROM agent_settings s), '[]'::jsonb),
    'history', coalesce((SELECT jsonb_agg(jsonb_build_object(
        'key', key, 'was', was, 'became', became, 'reason', reason,
        'by', changed_by, 'at', changed_at) ORDER BY changed_at DESC)
      FROM (SELECT * FROM agent_setting_history ORDER BY changed_at DESC LIMIT 20) z), '[]'::jsonb),
    'not_here',
      'Sending autonomy is deliberately absent from this table. Non-negotiable 3 keeps every send threshold at 0, and it is not a dial on a screen.')
    INTO out;
  RETURN out;
END 
```

### allocation_board(p_from date, p_to date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 25 lines
```sql

DECLARE f date; t date; me uuid; out jsonb;
BEGIN
  me := auth_staff_id();
  IF me IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;

  f := coalesce(p_from, date_trunc('week', now())::date);
  t := coalesce(p_to, now()::date);
  IF t < f THEN RAISE EXCEPTION 'The window ends before it starts' USING ERRCODE='22023'; END IF;

  SELECT jsonb_agg(jsonb_build_object(
      'staff_id', o_staff_id, 'name', o_name, 'email', o_email,
      'roles', to_jsonb(o_roles), 'places', o_places, 'target', o_target,
      'added', o_added, 'promoted', o_promoted, 'meetings', o_meetings)
    ORDER BY o_name) INTO out
   FROM allocation_board_body(f, t)
   WHERE has_role('owner') OR has_role('admin') OR has_role('manager') OR o_staff_id = me;

  RETURN jsonb_build_object(
    'from', f, 'to', t,
    'is_manager_view', (has_role('owner') OR has_role('admin') OR has_role('manager')),
    'people', coalesce(out, '[]'::jsonb),
    'rows_with_no_author', (SELECT count(*) FROM quarantine.research_contacts WHERE added_by IS NULL),
    'why_no_author', 'Rows that predate the desk carry no author. They are counted nowhere here.');
END 
```

### allocation_board_body(p_from date, p_to date) -> TABLE(o_staff_id uuid, o_name text, o_email text, o_roles text[], o_places jsonb, o_target jsonb, o_added integer, o_promoted integer, o_meetings integer)
- sql | invoker | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 28 lines
```sql

  WITH who AS (
    SELECT s.id, s.full_name, s.email
      FROM public.staff s
     WHERE EXISTS (SELECT 1 FROM public.allocations a WHERE a.staff_id = s.id AND a.is_active)
        OR EXISTS (SELECT 1 FROM public.staff_role_assignments r
                    WHERE r.staff_id = s.id AND r.role = 'researcher')
  )
  SELECT w.id, w.full_name, w.email,
    ARRAY(SELECT r.role::text FROM public.staff_role_assignments r
           WHERE r.staff_id = w.id ORDER BY r.role::text),
    coalesce((SELECT jsonb_agg(jsonb_build_object(
                'id', a.id, 'country', a.location_country,
                'state', a.location_state, 'city', a.location_city,
                'since', a.allocated_at) ORDER BY a.allocated_at)
              FROM public.allocations a WHERE a.staff_id = w.id AND a.is_active), '[]'::jsonb),
    coalesce((SELECT to_jsonb(t) - 'staff_id'::text - 'set_by'::text
                FROM public.staff_targets t WHERE t.staff_id = w.id), 'null'::jsonb),
    (SELECT count(*)::int FROM quarantine.research_contacts rc
      WHERE rc.added_by = w.id AND rc.landed_at::date BETWEEN p_from AND p_to),
    (SELECT count(*)::int FROM quarantine.research_contacts rc
      WHERE rc.added_by = w.id AND rc.landed_at::date BETWEEN p_from AND p_to
        AND rc.promoted_person_id IS NOT NULL),
    (SELECT count(*)::int FROM meeting_dataset_pairs() p
       JOIN public.meetings m ON m.id = p.o_meeting_id
      WHERE p.o_built_by = w.id AND m.starts_at::date BETWEEN p_from AND p_to)
    FROM who w

```

### allocation_covers(p_staff_id uuid, p_country text, p_state text, p_city text) -> boolean
- sql | invoker | stable | config: search_path=public, pg_temp | exec anon=n authenticated=n | 11 lines
- comment: Internal. Answers "may this person work this place" for callers that are already SECURITY DEFINER — research_add() is the only one. Not reachable from the API: it is a plain STABLE function, so a direct call meets allocations RLS and is refused.
```sql

  SELECT EXISTS (
    SELECT 1 FROM public.allocations a
     WHERE a.staff_id = p_staff_id AND a.is_active
       AND lower(btrim(a.location_country)) = lower(btrim(coalesce(p_country,'')))
       AND (a.location_state IS NULL
            OR lower(btrim(a.location_state)) = lower(btrim(coalesce(p_state,''))))
       AND (a.location_city IS NULL
            OR lower(btrim(a.location_city)) = lower(btrim(coalesce(p_city,''))))
  )

```

### allocation_end(p_allocation_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 15 lines
```sql

DECLARE me uuid; n int;
BEGIN
  me := auth_staff_id();
  IF me IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')) THEN
    RAISE EXCEPTION 'Only an owner, admin or manager may end an allocation' USING ERRCODE='42501'; END IF;

  UPDATE public.allocations
     SET is_active = false, ended_at = now()
   WHERE id = p_allocation_id AND is_active;
  GET DIAGNOSTICS n = ROW_COUNT;
  IF n = 0 THEN
    RAISE EXCEPTION 'No active allocation with that id' USING ERRCODE='02000'; END IF;
  RETURN jsonb_build_object('ok', true, 'ended', n);
END 
```

### allocation_no_overlap() -> trigger
- plpgsql | invoker | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=n | 26 lines
```sql

DECLARE clash record;
BEGIN
  IF NOT NEW.is_active THEN RETURN NEW; END IF;

  SELECT a.id, s.full_name,
         concat_ws(', ', a.location_city, a.location_state, a.location_country) AS place
    INTO clash
    FROM public.allocations a
    LEFT JOIN public.staff s ON s.id = a.staff_id
   WHERE a.is_active
     AND a.id IS DISTINCT FROM NEW.id
     AND lower(btrim(a.location_country)) = lower(btrim(NEW.location_country))
     AND (a.location_state IS NULL OR NEW.location_state IS NULL
          OR lower(btrim(a.location_state)) = lower(btrim(NEW.location_state)))
     AND (a.location_city IS NULL OR NEW.location_city IS NULL
          OR lower(btrim(a.location_city)) = lower(btrim(NEW.location_city)))
   LIMIT 1;

  IF clash.id IS NOT NULL THEN
    RAISE EXCEPTION '% already has %. Two people on one place is the thing this screen exists to stop — end that allocation first',
      coalesce(clash.full_name, 'somebody'), clash.place
      USING ERRCODE = '23505';
  END IF;
  RETURN NEW;
END 
```

### allocation_set(p_staff_id uuid, p_country text, p_state text, p_city text, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 20 lines
```sql

DECLARE me uuid; a uuid;
BEGIN
  me := auth_staff_id();
  IF me IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')) THEN
    RAISE EXCEPTION 'Only an owner, admin or manager may allocate a place' USING ERRCODE='42501'; END IF;
  IF nullif(btrim(coalesce(p_country,'')),'') IS NULL THEN
    RAISE EXCEPTION 'An allocation needs at least a country' USING ERRCODE='23502'; END IF;
  IF NOT EXISTS (SELECT 1 FROM public.staff WHERE id = p_staff_id) THEN
    RAISE EXCEPTION 'No such staff member' USING ERRCODE='02000'; END IF;

  INSERT INTO public.allocations (staff_id, location_country, location_state, location_city,
                                  allocated_by, note)
  VALUES (p_staff_id, btrim(p_country),
          nullif(btrim(coalesce(p_state,'')),''), nullif(btrim(coalesce(p_city,'')),''),
          me, nullif(btrim(coalesce(p_note,'')),''))
  RETURNING id INTO a;

  RETURN jsonb_build_object('ok', true, 'allocation_id', a);
END 
```

### approve_access_request(p_staff_id uuid, p_roles staff_role[], p_grants staff_grant[], p_scopes jsonb, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=Y | 42 lines
```sql

DECLARE v_me uuid := auth_staff_id(); v_role staff_role; v_grant staff_grant;
        v_kind text; v_val text; v_target staff%ROWTYPE;
BEGIN
  IF v_me IS NULL OR NOT (has_role('owner') OR has_role('admin')) THEN
    RAISE EXCEPTION 'only an owner or admin may approve access requests'; END IF;
  IF p_roles IS NULL OR cardinality(p_roles) = 0 THEN
    RAISE EXCEPTION 'a role is required — approving without one creates an account that can sign in and reach nothing'; END IF;
  IF 'owner' = ANY (p_roles) AND NOT has_role('owner') THEN
    RAISE EXCEPTION 'only an owner may grant the owner role'; END IF;

  SELECT * INTO v_target FROM staff WHERE id = p_staff_id FOR UPDATE;
  IF NOT FOUND THEN RAISE EXCEPTION 'no such staff row: %', p_staff_id; END IF;
  IF v_target.status <> 'requested' THEN
    RAISE EXCEPTION 'staff % is %, not requested — nothing to approve', p_staff_id, v_target.status; END IF;
-- ... truncated (42 lines total)
```

### auth_staff_id() -> uuid
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 6 lines
- comment: The staff row for the current Auth user, or NULL. NULL for every parent and student, who share the same Auth pool. Policies key off this, so NULL means sees nothing.
```sql

  SELECT s.id FROM staff s
   WHERE s.auth_user_id = auth.uid()
     AND s.status = 'active'
     AND (s.ends_on IS NULL OR s.ends_on >= current_date)

```

### can_see_person(p_person_id uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 28 lines
- comment: The row rule for every person-linked table. Scope only — it says nothing about which COLUMNS are visible; that is the grants, applied in the _v views.
```sql

  SELECT auth_staff_id() IS NOT NULL
     AND (
       has_scope_all()
       -- own / team: a lead I own, or one owned by someone in my subtree.
       OR EXISTS (SELECT 1 FROM memberships m
                   WHERE m.person_id = p_person_id
                     AND m.owner_staff_id IN (SELECT staff_visible_ids()))
       -- programme:CODE — anything attached to a named programme.
       --
       -- Four hops, not three: campaigns has no programme_id. The chain is
       -- memberships → campaigns.cohort_id → cohorts.programme_id → programmes,
       -- because a campaign targets a COHORT (a specific intake with its own
       -- dates and price), and the programme is the cohort's parent. Verified
       -- against the live foreign keys on 11 August 2026.
       OR EXISTS (SELECT 1 FROM memberships m
                    JOIN campaigns  c  ON c.id  = m.campaign_id
                    JOIN cohorts    co ON co.id = c.cohort_id
                    JOIN programmes pr ON pr.id = co.programme_id
                   WHERE m.person_id = p_person_id
                     AND has_scope('programme', pr.code))
       -- region:CODE — anything in a market. people.country is the ISO code.
       OR EXISTS (SELECT 1 FROM people p
                   WHERE p.id = p_person_id
                     AND p.country IS NOT NULL
                     AND has_scope('region', p.country))
     )

```

### candidates_awaiting_name(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 32 lines
```sql

DECLARE r jsonb; total int;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  SELECT count(*) INTO total FROM quarantine.org_candidates c
   WHERE NOT c.name_agreement AND c.promoted_organization_id IS NULL;

  SELECT coalesce(jsonb_agg(x ORDER BY (x->>'people_count')::int DESC, x->>'company_name'), '[]'::jsonb)
    INTO r FROM (
    SELECT jsonb_build_object('id', c.id, 'company_name', c.company_name, 'domain', c.domain,
             'website', c.website, 'industry', c.industry, 'linkedin_url', c.linkedin_url,
             'country', c.country_raw, 'city', c.city_raw, 'people_count', c.people_count,
             'source', c.source_system, 'note', c.request_note,
             'already_in_register', (c.existing_organization_id IS NOT NULL),
             'existing_organization_id', c.existing_organization_id,
             'triage', quarantine.school_triage(c.company_name),
             -- The signal the first two samples exposed and school_triage() cannot see:
             -- "Bryan Adams High School" sits on dallasisd.org (a district) while stating baaa.org,
             -- and "MVP Video Productions" sits on stpsb.org (a school board). When the email domain
             -- and the stated website disagree, the domain usually belongs to somebody else.
             'domain_agrees_with_website',
               CASE WHEN c.domain IS NULL OR c.website IS NULL THEN NULL
                    ELSE lower(c.domain) = lower(regexp_replace(c.website,
                           '^https?://(www\.)?|/.*$', '', 'g')) END,
             'landed_at', c.landed_at) AS x
      FROM quarantine.org_candidates c
     WHERE NOT c.name_agreement AND c.promoted_organization_id IS NULL
     ORDER BY c.people_count DESC, c.company_name
     LIMIT least(greatest(coalesce(p_limit,50),1),200)) s;

  RETURN jsonb_build_object('total', total, 'showing', jsonb_array_length(r), 'candidates', r);
END 
```

### channel_of(p_raw text) -> channel
- plpgsql | invoker | immutable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 6 lines
```sql

BEGIN
  RETURN lower(btrim(coalesce(p_raw,'')))::channel;
EXCEPTION WHEN invalid_text_representation THEN
  RETURN NULL;
END 
```

### cohort_command() -> jsonb
- sql | invoker | stable | config: search_path=public | exec anon=n authenticated=Y | 33 lines
```sql

  SELECT jsonb_build_object(
    'readiness', jsonb_build_object(
      'cohorts',            (SELECT count(*) FROM cohorts),
      'with_capacity',      (SELECT count(*) FROM cohorts WHERE capacity IS NOT NULL AND capacity > 0),
      'with_seats',         (SELECT count(*) FROM cohorts WHERE coalesce(seats_taken,0) > 0),
      'with_price',         (SELECT count(*) FROM cohorts WHERE price_minor IS NOT NULL AND price_minor > 0),
      'with_payment_link',  (SELECT count(*) FROM cohorts WHERE payment_link_url IS NOT NULL),
      'memberships',        (SELECT count(*) FROM memberships)),
    'cohorts', coalesce((
      SELECT jsonb_agg(x) FROM (
        SELECT jsonb_build_object(
                 'cohort_id',    c.id,
                 'programme_id', c.programme_id,
                 'label',        c.label,
                 'programme',    p.name,
                 'partner',      p.partner_name,
                 'category',     p.category,
                 'starts_on',    c.starts_on,
                 'ends_on',      c.ends_on,
                 'deadline',     c.application_deadline,
                 'days_to_deadline', (c.application_deadline - current_date),
                 'days_to_start',    (c.starts_on - current_date),
                 'price_minor',  c.price_minor,
                 'currency',     c.currency,
                 'capacity',     c.capacity,
                 'seats_taken',  c.seats_taken,
                 'is_open',      c.is_open,
                 'has_link',     (c.payment_link_url IS NOT NULL)) AS x
          FROM cohorts c
          LEFT JOIN programmes p ON p.id = c.programme_id
         ORDER BY c.starts_on NULLS LAST) y), '[]'::jsonb));

```

### confirm_candidate_name(p_candidate_id uuid, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 27 lines
```sql

DECLARE c quarantine.org_candidates%ROWTYPE;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')) THEN
    RAISE EXCEPTION 'Only an owner, admin or manager may confirm a candidate''s name' USING ERRCODE='42501'; END IF;
  SELECT * INTO c FROM quarantine.org_candidates WHERE id = p_candidate_id;
  IF c.id IS NULL THEN RAISE EXCEPTION 'No such candidate %', p_candidate_id USING ERRCODE='P0002'; END IF;
  IF c.promoted_organization_id IS NOT NULL THEN
    RAISE EXCEPTION 'This candidate has already been promoted' USING ERRCODE='22023'; END IF;

  UPDATE quarantine.org_candidates
     SET name_agreement = true, reviewed_by = auth_staff_id(), reviewed_at = now(),
         review_note = coalesce(nullif(btrim(p_note),''), review_note)
   WHERE id = p_candidate_id;

  INSERT INTO activities (occurred_at, person_id, organization_id, kind, actor, payload)
  VALUES (now(), NULL, c.existing_organization_id, 'candidate_name_confirmed',
          'staff:'||auth_staff_id()::text,
          jsonb_build_object('candidate_id',p_candidate_id,'company_name',c.company_name,
                             'domain',c.domain,'note',p_note));

  RETURN jsonb_build_object('ok',true,'candidate_id',p_candidate_id,'company_name',c.company_name,
    'promotable_now',(c.existing_organization_id IS NULL),
    'blocked_because', CASE WHEN c.existing_organization_id IS NOT NULL
      THEN 'this domain or name already exists in the register - link it instead of promoting a second row' END);
END 
```

### contact_ownership(p_email text) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

DECLARE me uuid;
BEGIN
  me := auth_staff_id();
  IF me IS NULL AND coalesce(auth.role(),'') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  RETURN contact_ownership_body(p_email, me);
END 
```

### contact_ownership_body(p_email text, p_actor uuid) -> jsonb
- sql | invoker | stable | config: search_path=public, quarantine, extensions, pg_temp | exec anon=n authenticated=n | 109 lines
```sql

  WITH k AS (SELECT identity_key('email', p_email) AS e),
  hold_days AS (SELECT coalesce(agent_setting('recent_reply_hold_days'), 90)::int AS d),
  who AS (
    SELECT pi.person_id FROM person_identities pi, k
     WHERE pi.kind = 'email' AND pi.value_norm = k.e LIMIT 1),
  supp AS (
    SELECT s.reason::text AS reason, s.suppressed_at FROM suppressions s, k
     WHERE s.kind = 'email' AND s.identity_hash = digest(k.e, 'sha256')
     ORDER BY s.suppressed_at LIMIT 1),
  grp AS (
    SELECT g.id, g.name, g.kind, g.campaign_owner, g.schools_counted,
           g.name_is_placeholder, st.full_name AS owner_name,
           (SELECT count(*) FROM org_group_domains d2 WHERE d2.group_id = g.id) AS domains_held
      FROM org_groups g
-- ... truncated (109 lines total)
```

### create_activities_partition(month_start date) -> text
- plpgsql | invoker | volatile | config: search_path=public | exec anon=n authenticated=n | 24 lines
- comment: Creates next month's activities partition WITH row level security and the scoped policy from 008. Never create a partition by hand — one made with CREATE TABLE inherits no RLS and no policy.
```sql

DECLARE
  part_name  text := 'activities_' || to_char(month_start, 'YYYY_MM');
  next_month date := (month_start + interval '1 month')::date;
BEGIN
  IF EXISTS (SELECT 1 FROM pg_class WHERE relname = part_name) THEN
    RETURN part_name || ' (already exists)';
  END IF;
  EXECUTE format(
    'CREATE TABLE %I PARTITION OF activities FOR VALUES FROM (%L) TO (%L)',
    part_name, month_start, next_month);
  -- The whole point of the helper: a hand-made partition inherits no row-level
  -- security, which is one of the three bugs 001's smoke test caught.
  EXECUTE format('ALTER TABLE %I ENABLE ROW LEVEL SECURITY', part_name);
  EXECUTE format('REVOKE ALL ON %I FROM anon, authenticated', part_name);
  EXECUTE format('GRANT SELECT ON %I TO authenticated', part_name);
  EXECUTE format('GRANT ALL ON %I TO service_role', part_name);
  EXECUTE format(
    'CREATE POLICY %I ON %I FOR SELECT TO authenticated
       USING (person_id IS NOT NULL AND can_see_person(person_id))',
    'staff_scoped_' || part_name, part_name);
  RETURN part_name || ' (created)';
END;

```

### dashboard_metrics() -> jsonb
- sql | invoker | stable | config: search_path=public | exec anon=n authenticated=Y | 40 lines
```sql

  SELECT jsonb_build_object(
    'people', jsonb_build_object(
      'total', (SELECT count(*) FROM people_v),
      'minors', (SELECT count(*) FROM people_v WHERE is_minor),
      'names_masked', (SELECT count(*) FROM people_v WHERE full_name = '—'),
      'needs_review', (SELECT count(*) FROM people_v WHERE needs_review),
      'identities', (SELECT count(*) FROM person_identities_v),
      'contactable', (SELECT count(*) FROM people_v WHERE zone <> 'raw')),
    'households', jsonb_build_object(
      'total', (SELECT count(*) FROM households_v),
      'members', (SELECT count(*) FROM household_members),
      'minors_unlinked', (SELECT count(*) FROM people_v p WHERE p.is_minor
                            AND NOT EXISTS (SELECT 1 FROM household_members m WHERE m.person_id=p.id))),
    'organisations', jsonb_build_object(
      'total', (SELECT count(*) FROM organizations),
      'with_domain', (SELECT count(*) FROM organizations WHERE domain IS NOT NULL),
      'with_website', (SELECT count(*) FROM organizations WHERE website IS NOT NULL),
      'needs_review', (SELECT count(*) FROM organizations WHERE needs_review),
      'affiliations', (SELECT count(*) FROM affiliations),
      'people_unaffiliated', (SELECT count(*) FROM people_v p
                               WHERE NOT EXISTS (SELECT 1 FROM affiliations a WHERE a.person_id=p.id))),
    'programmes', jsonb_build_object(
      'total', (SELECT count(*) FROM programmes),
      'cohorts', (SELECT count(*) FROM cohorts),
      'memberships', (SELECT count(*) FROM memberships)),
    'governance', jsonb_build_object(
      'suppressions', (SELECT count(*) FROM suppressions),
      'consents', (SELECT count(*) FROM consents),
      'source_codes', (SELECT count(*) FROM source_codes),
      'with_lawful_basis', (SELECT count(*) FROM source_codes WHERE lawful_basis IS NOT NULL),
      'approved_facts', (SELECT count(*) FROM approved_facts),
      'activities', (SELECT count(*) FROM activities)),
    'by_source', coalesce((
      SELECT jsonb_agg(jsonb_build_object('code', s.code, 'people', s.n, 'basis', s.basis) ORDER BY s.n DESC)
        FROM (SELECT replace(p.source_code,'SRC-LEGACY-202608-','') AS code, count(*) AS n,
                     max(sc.lawful_basis::text) AS basis
                FROM people_v p LEFT JOIN source_codes sc ON sc.code = p.source_code
               GROUP BY 1) s), '[]'::jsonb));

```

### dashboard_today() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 7 lines
- comment: The business today, in one call: what arrived and from where, what is waiting on a human with somewhere to act on it, meetings, money per currency, what the agents did and how much a human has read, and what is refused. Aggregates only, so every staff member may read it.
```sql

BEGIN
  -- Aggregates only: counts, sums and queue sizes. No name, no address, nothing a grant
  -- governs — which is why every staff member may read it, exactly like dashboard_metrics.
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  RETURN dashboard_today_body();
END 
```

### dashboard_today_body() -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 111 lines
```sql

  SELECT jsonb_build_object(

    'arriving', jsonb_build_object(
      'today',     (SELECT count(*) FROM inbound_records WHERE received_at >= current_date),
      'last_7d',   (SELECT count(*) FROM inbound_records WHERE received_at >= current_date - 6),
      'pending',   (SELECT count(*) FROM inbound_records WHERE processed_at IS NULL AND rejected_at IS NULL),
      'by_system', coalesce((
        SELECT jsonb_agg(jsonb_build_object(
                 'system', x.system_key, 'label', ss.label,
                 'last_7d', x.n, 'pending', x.pending, 'refused', x.refused,
                 'last_seen', x.last_seen) ORDER BY x.n DESC)
          FROM (SELECT ir.system_key, count(*) AS n,
                       count(*) FILTER (WHERE ir.processed_at IS NULL AND ir.rejected_at IS NULL) AS pending,
                       count(*) FILTER (WHERE ir.rejected_at IS NOT NULL) AS refused,
-- ... truncated (111 lines total)
```

### dataset_create(p_code text, p_country text, p_state text, p_city text, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 19 lines
```sql

DECLARE sid uuid; d uuid;
BEGIN
  sid := auth_staff_id();
  IF sid IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach')) THEN
    RAISE EXCEPTION 'You do not have a role that may build a dataset' USING ERRCODE='42501'; END IF;

  IF nullif(btrim(coalesce(p_country,'')),'') IS NULL THEN
    RAISE EXCEPTION 'A dataset is a list for a place. Name the country' USING ERRCODE='23502'; END IF;

  INSERT INTO quarantine.datasets (code, location_country, location_state, location_city, built_by, note)
  VALUES (upper(btrim(coalesce(p_code,''))), btrim(p_country),
          nullif(btrim(coalesce(p_state,'')),''), nullif(btrim(coalesce(p_city,'')),''),
          sid, nullif(btrim(coalesce(p_note,'')),''))
  RETURNING id INTO d;

  RETURN dataset_detail(d, 25, 0);
END 
```

### dataset_detail(p_dataset_id uuid, p_limit integer, p_offset integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 49 lines
```sql

DECLARE d quarantine.datasets%ROWTYPE; lim int; off int;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  SELECT * INTO d FROM quarantine.datasets WHERE id = p_dataset_id;
  IF d.id IS NULL THEN RAISE EXCEPTION 'No such dataset' USING ERRCODE='02000'; END IF;

  lim := least(greatest(coalesce(p_limit, 25), 1), 200);
  off := greatest(coalesce(p_offset, 0), 0);

  RETURN jsonb_build_object(
    'dataset', jsonb_build_object(
      'id', d.id, 'code', d.code, 'status', d.status,
      'country', d.location_country, 'state', d.location_state, 'city', d.location_city,
      'built_at', d.built_at, 'campaign_done_at', d.campaign_done_at, 'note', d.note,
-- ... truncated (49 lines total)
```

### dataset_fill(p_dataset_id uuid, p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 31 lines
```sql

DECLARE sid uuid; d quarantine.datasets%ROWTYPE; lim int; added int;
BEGIN
  sid := auth_staff_id();
  IF sid IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach')) THEN
    RAISE EXCEPTION 'You do not have a role that may build a dataset' USING ERRCODE='42501'; END IF;

  SELECT * INTO d FROM quarantine.datasets WHERE id = p_dataset_id FOR UPDATE;
  IF d.id IS NULL THEN RAISE EXCEPTION 'No such dataset' USING ERRCODE='02000'; END IF;
  IF d.status IN ('superseded','campaign_done') THEN
    RAISE EXCEPTION 'That version is % and cannot take new rows. A changed list is a new version',
      d.status USING ERRCODE='42501'; END IF;

  lim := least(greatest(coalesce(p_limit, 2000), 1), 50000);

  INSERT INTO quarantine.dataset_rows (dataset_id, research_contact_id)
  SELECT d.id, rc.id
    FROM quarantine.research_contacts rc
   WHERE lower(btrim(rc.country_raw)) = lower(btrim(d.location_country))
     AND (d.location_state IS NULL OR lower(btrim(rc.state_raw)) = lower(btrim(d.location_state)))
     AND (d.location_city  IS NULL OR lower(btrim(rc.city_raw))  = lower(btrim(d.location_city)))
     AND rc.is_valid IS NOT FALSE
     AND rc.email_norm IS NOT NULL
   ORDER BY rc.is_valid DESC NULLS LAST, rc.landed_at
   LIMIT lim
  ON CONFLICT DO NOTHING;
  GET DIAGNOSTICS added = ROW_COUNT;

  RETURN dataset_detail(p_dataset_id, 25, 0) || jsonb_build_object('rows_added', added);
END 
```

### dataset_list(p_stale_months integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 26 lines
```sql

DECLARE out jsonb; stale int;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  stale := greatest(coalesce(p_stale_months, 18), 1);

  SELECT jsonb_agg(jsonb_build_object(
      'id', d.id, 'code', d.code, 'status', d.status,
      'country', d.location_country, 'state', d.location_state, 'city', d.location_city,
      'built_at', d.built_at, 'built_by', s.full_name,
      'redo_of', (SELECT p.code FROM quarantine.datasets p WHERE p.id = d.redo_of),
      'campaign_done_at', d.campaign_done_at,
      'rows_total', (SELECT count(*) FROM quarantine.dataset_rows dr WHERE dr.dataset_id = d.id),
      'meetings', (SELECT count(*) FROM meeting_dataset_pairs() mp WHERE mp.o_dataset_id = d.id),
      'months_since_built', floor(extract(epoch FROM (now() - d.built_at)) / 2629800)::int,
      'needs_redo', (d.status <> 'superseded'
                     AND d.built_at < now() - make_interval(months => stale))
    ) ORDER BY d.built_at DESC) INTO out
    FROM quarantine.datasets d LEFT JOIN public.staff s ON s.id = d.built_by;

  RETURN jsonb_build_object(
    'datasets', coalesce(out, '[]'::jsonb),
    'stale_after_months', stale,
    'meetings_total', (SELECT count(*) FROM public.meetings),
    'meetings_attributable', (SELECT count(*) FROM meeting_dataset_pairs()));
END 
```

### dataset_redo(p_dataset_id uuid, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 30 lines
```sql

DECLARE sid uuid; parent quarantine.datasets%ROWTYPE; d uuid; copied int;
BEGIN
  sid := auth_staff_id();
  IF sid IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach')) THEN
    RAISE EXCEPTION 'You do not have a role that may build a dataset' USING ERRCODE='42501'; END IF;

  SELECT * INTO parent FROM quarantine.datasets WHERE id = p_dataset_id FOR UPDATE;
  IF parent.id IS NULL THEN RAISE EXCEPTION 'No such dataset' USING ERRCODE='02000'; END IF;
  IF parent.status = 'superseded' THEN
    RAISE EXCEPTION 'That version has already been redone as a newer one. Redo the newest, not this'
      USING ERRCODE='42501'; END IF;

  INSERT INTO quarantine.datasets (
    code, location_country, location_state, location_city, built_by, redo_of, note)
  VALUES (next_dataset_code(parent.code), parent.location_country, parent.location_state,
          parent.location_city, parent.built_by, parent.id,
          nullif(btrim(coalesce(p_note,'')),''))
  RETURNING id INTO d;

  INSERT INTO quarantine.dataset_rows (dataset_id, research_contact_id)
  SELECT d, dr.research_contact_id FROM quarantine.dataset_rows dr WHERE dr.dataset_id = parent.id
  ON CONFLICT DO NOTHING;
  GET DIAGNOSTICS copied = ROW_COUNT;

  UPDATE quarantine.datasets SET status = 'superseded' WHERE id = parent.id;

  RETURN dataset_detail(d, 25, 0) || jsonb_build_object('rows_carried_over', copied);
END 
```

### dataset_set_status(p_dataset_id uuid, p_status text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 24 lines
```sql

DECLARE sid uuid; d quarantine.datasets%ROWTYPE;
BEGIN
  sid := auth_staff_id();
  IF sid IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach')) THEN
    RAISE EXCEPTION 'You do not have a role that may build a dataset' USING ERRCODE='42501'; END IF;

  SELECT * INTO d FROM quarantine.datasets WHERE id = p_dataset_id FOR UPDATE;
  IF d.id IS NULL THEN RAISE EXCEPTION 'No such dataset' USING ERRCODE='02000'; END IF;

  IF p_status NOT IN ('building','ready','campaign_done') THEN
    RAISE EXCEPTION 'A version may be moved to building, ready or campaign_done. Superseded is set by a redo'
      USING ERRCODE='22023'; END IF;
  IF d.status = 'superseded' THEN
    RAISE EXCEPTION 'That version has been superseded by a newer one' USING ERRCODE='42501'; END IF;

  UPDATE quarantine.datasets
     SET status = p_status,
         campaign_done_at = CASE WHEN p_status = 'campaign_done' THEN now() ELSE NULL END
   WHERE id = p_dataset_id;

  RETURN dataset_detail(p_dataset_id, 25, 0);
END 
```

### email_check_windows() -> jsonb
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=Y | 17 lines
```sql

  SELECT jsonb_build_object(
    'positive_check_valid_days', 90,
    'cooling_days',              90,
    'at_risk_days',              180,
    'suppress_after_days',       365,
    'annual_decay_pct',          23,
    'two_year_scrape_invalid_pct_low',  40,
    'two_year_scrape_invalid_pct_high', 60,
    'bounce_alarm_pct',          2,
    'bounce_stop_pct',           5,
    'complaint_alarm_pct',       0.08,
    'complaint_ceiling_pct',     0.10,
    'complaint_cliff_pct',       0.30,
    'trap_alarm_hits',           1,
    'source', 'docs/fixing-the-system/research/01-thresholds.md')

```

### email_domain_queue(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 37 lines
```sql

DECLARE lim int; out jsonb; total int; done int;
BEGIN
  IF auth_staff_id() IS NULL AND coalesce(auth.role(), '') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  lim := least(greatest(coalesce(p_limit, 500), 1), 5000);

  WITH dom AS (
    SELECT lower(split_part(rc.email_norm,'@',2)) AS d, count(*) AS n
      FROM quarantine.research_contacts rc
     WHERE rc.email_norm IS NOT NULL AND rc.promoted_person_id IS NULL
     GROUP BY lower(split_part(rc.email_norm,'@',2))
  ),
  due AS (
    SELECT dom.d, dom.n FROM dom
     WHERE NOT EXISTS (SELECT 1 FROM quarantine.email_domain_evidence v
                        WHERE v.domain = dom.d AND v.method = 'dns_resolve')
     ORDER BY dom.n DESC
     LIMIT lim
  )
  SELECT jsonb_agg(jsonb_build_object('domain', d, 'addresses', n) ORDER BY n DESC),
         (SELECT count(*) FROM dom),
         (SELECT count(*) FROM dom
           WHERE EXISTS (SELECT 1 FROM quarantine.email_domain_evidence v
                          WHERE v.domain = dom.d AND v.method = 'dns_resolve'))
    INTO out, total, done
    FROM due;

  RETURN jsonb_build_object(
    'due', coalesce(out, '[]'::jsonb),
    'domains_total', total,
    'domains_checked', done,
    'addresses_behind_the_batch', (SELECT coalesce(sum((x->>'addresses')::int), 0)
                                     FROM jsonb_array_elements(coalesce(out,'[]'::jsonb)) x),
    'how', 'DNS over HTTPS against 1.1.1.1 or 8.8.8.8. Free, no port 25, works from an edge '
           || 'function or from pg_net. One answer decides every address on the domain.');
END 
```

### email_needs_check(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 35 lines
```sql

DECLARE lim int; out jsonb; fresh int;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  lim   := least(greatest(coalesce(p_limit, 500), 1), 5000);
  fresh := (email_check_windows()->>'positive_check_valid_days')::int;

  SELECT jsonb_agg(jsonb_build_object('email', e, 'contacts', n, 'oldest_landing', l)
         ORDER BY n DESC, l) INTO out
    FROM (
      SELECT rc.email_norm e, count(*) n, min(rc.landed_at) l
        FROM quarantine.research_contacts rc
       WHERE rc.email_norm IS NOT NULL
         AND rc.promoted_person_id IS NULL
         AND NOT EXISTS (SELECT 1 FROM quarantine.email_evidence v
                          WHERE v.email_norm = rc.email_norm
                            AND v.method IN ('bounce_hard','complaint','trap_hit','disposable'))
         AND NOT EXISTS (SELECT 1 FROM quarantine.email_evidence v
                          WHERE v.email_norm = rc.email_norm AND v.result = 'pass'
                            AND v.method IN ('provider_api','smtp_probe','engagement')
                            AND v.checked_at > now() - make_interval(days => fresh))
       GROUP BY rc.email_norm
       ORDER BY count(*) DESC, min(rc.landed_at)
       LIMIT lim) z;

  RETURN jsonb_build_object(
    'windows', email_check_windows(),
    'due', coalesce(out, '[]'::jsonb),
    'total_due', (SELECT count(DISTINCT rc.email_norm) FROM quarantine.research_contacts rc
                   WHERE rc.email_norm IS NOT NULL AND rc.promoted_person_id IS NULL
                     AND NOT EXISTS (SELECT 1 FROM quarantine.email_evidence v
                                      WHERE v.email_norm = rc.email_norm AND v.result = 'pass'
                                        AND v.method IN ('provider_api','smtp_probe','engagement')
                                        AND v.checked_at > now() - make_interval(days => fresh))));
END 
```

### email_provider_queue(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 35 lines
```sql

DECLARE lim int; out jsonb; fresh int;
BEGIN
  IF auth_staff_id() IS NULL AND coalesce(auth.role(), '') <> 'service_role' THEN
    RAISE EXCEPTION 'Not authorized' USING ERRCODE='42501';
  END IF;
  lim   := least(greatest(coalesce(p_limit, 100), 1), 5000);
  fresh := (email_check_windows()->>'positive_check_valid_days')::int;

  SELECT jsonb_agg(jsonb_build_object('email', e, 'contacts', n, 'oldest_landing', l)
         ORDER BY n DESC, l) INTO out
    FROM (
      SELECT rc.email_norm e, count(*) n, min(rc.landed_at) l
        FROM quarantine.research_contacts rc
        JOIN quarantine.email_domain_evidence dv
          ON lower(split_part(rc.email_norm, '@', 2)) = dv.domain
       WHERE rc.email_norm IS NOT NULL
         AND rc.promoted_person_id IS NULL
         AND dv.method = 'dns_resolve' AND dv.result = 'pass'
         AND NOT EXISTS (SELECT 1 FROM quarantine.email_evidence v
                          WHERE v.email_norm = rc.email_norm
                            AND v.method IN ('bounce_hard','complaint','trap_hit','disposable'))
         AND NOT EXISTS (SELECT 1 FROM quarantine.email_evidence v
                          WHERE v.email_norm = rc.email_norm AND v.result = 'pass'
                            AND v.method IN ('provider_api','smtp_probe','engagement')
                            AND v.checked_at > now() - make_interval(days => fresh))
       GROUP BY rc.email_norm
       ORDER BY count(*) DESC, min(rc.landed_at)
       LIMIT lim) z;

  RETURN jsonb_build_object(
    'due', coalesce(out, '[]'::jsonb),
    'windows', email_check_windows()
  );
END 
```

### email_record_domain_evidence(p_domain text, p_method text, p_result text, p_detail jsonb, p_checked_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 26 lines
```sql

DECLARE me uuid; d text; who text; reach int; is_agent boolean;
BEGIN
  me := auth_staff_id();
  is_agent := coalesce(auth.role(), '') = 'service_role';
  IF NOT is_agent AND (me IS NULL OR NOT (has_role('owner') OR has_role('admin')
      OR has_role('manager') OR has_role('engineer') OR has_role('outreach'))) THEN
    RAISE EXCEPTION 'You do not have a role that may record domain evidence' USING ERRCODE='42501'; END IF;

  d := lower(btrim(coalesce(p_domain,'')));
  IF d = '' OR d NOT LIKE '%.%' THEN
    RAISE EXCEPTION 'Not a domain' USING ERRCODE='22023'; END IF;
  who := coalesce(nullif(btrim(coalesce(p_checked_by,'')),''),
                  CASE WHEN is_agent THEN 'agent:unnamed' ELSE me::text END);

  INSERT INTO quarantine.email_domain_evidence (domain, method, result, detail, checked_by, checked_at)
  VALUES (d, p_method, p_result, p_detail, who, now())
  ON CONFLICT (domain, method) DO UPDATE
    SET result = excluded.result, detail = excluded.detail,
        checked_by = excluded.checked_by, checked_at = now();

  SELECT count(*) INTO reach FROM quarantine.research_contacts
   WHERE lower(split_part(email_norm,'@',2)) = d AND promoted_person_id IS NULL;

  RETURN jsonb_build_object('ok', true, 'domain', d, 'addresses_decided', reach);
END 
```

### email_record_domain_evidence_batch(p_rows jsonb, p_checked_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 48 lines
- comment: Record DNS evidence for many domains in one call. Same guards as the single-row writer; a malformed row is skipped with its reason instead of failing the batch.
```sql

DECLARE
  me uuid; who text; is_agent boolean;
  r jsonb; d text; written int := 0; skipped jsonb := '[]'::jsonb;
BEGIN
  me := auth_staff_id();
  is_agent := coalesce(auth.role(), '') = 'service_role';
  IF NOT is_agent AND (me IS NULL OR NOT (has_role('owner') OR has_role('admin')
      OR has_role('manager') OR has_role('engineer') OR has_role('outreach'))) THEN
    RAISE EXCEPTION 'You do not have a role that may record domain evidence' USING ERRCODE='42501';
  END IF;

  who := coalesce(nullif(btrim(coalesce(p_checked_by,'')),''),
                  CASE WHEN is_agent THEN 'agent:unnamed' ELSE me::text END);

-- ... truncated (48 lines total)
```

### email_record_evidence(p_email text, p_method text, p_result text, p_detail jsonb, p_checked_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, extensions, pg_temp | exec anon=n authenticated=Y | 30 lines
```sql

DECLARE me uuid; ek text; who text; sup_reason suppression_reason; is_agent boolean;
BEGIN
  me := auth_staff_id();
  is_agent := coalesce(auth.role(), '') = 'service_role';
  IF NOT is_agent AND (me IS NULL OR NOT (has_role('owner') OR has_role('admin')
      OR has_role('manager') OR has_role('engineer') OR has_role('outreach'))) THEN
    RAISE EXCEPTION 'You do not have a role that may record deliverability evidence' USING ERRCODE='42501'; END IF;

  ek := identity_key('email', p_email);
  IF ek IS NULL THEN
    RAISE EXCEPTION 'Not an address this database can key on' USING ERRCODE='22023'; END IF;

  who := coalesce(nullif(btrim(coalesce(p_checked_by,'')),''),
                  CASE WHEN is_agent THEN 'agent:unnamed' ELSE me::text END);

  INSERT INTO quarantine.email_evidence (email_norm, method, result, detail, checked_by)
  VALUES (ek, p_method, p_result, p_detail, who);

  IF p_method IN ('bounce_hard','complaint','trap_hit') THEN
    sup_reason := CASE p_method
                    WHEN 'complaint' THEN 'complained'::suppression_reason
                    ELSE 'bounced_hard'::suppression_reason END;
    INSERT INTO public.suppressions (kind, identity_hash, reason, suppressed_at)
    VALUES ('email'::identity_kind, digest(ek, 'sha256'), sup_reason, now())
    ON CONFLICT (kind, identity_hash) DO NOTHING;
  END IF;

  RETURN email_sendability_body(ek);
END 
```

### email_record_evidence_batch(p_rows jsonb, p_checked_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 46 lines
- comment: Record evidence for many addresses in one call.
```sql

DECLARE
  me uuid; who text; is_agent boolean;
  r jsonb; e text; m text; res text;
  written int := 0; skipped jsonb := '[]'::jsonb;
BEGIN
  me := auth_staff_id();
  is_agent := coalesce(auth.role(), '') = 'service_role';
  IF NOT is_agent AND (me IS NULL OR NOT (has_role('owner') OR has_role('admin')
      OR has_role('manager') OR has_role('engineer') OR has_role('outreach'))) THEN
    RAISE EXCEPTION 'You do not have a role that may record evidence' USING ERRCODE='42501';
  END IF;

  who := coalesce(nullif(btrim(coalesce(p_checked_by,'')),''),
                  CASE WHEN is_agent THEN 'agent:unnamed' ELSE me::text END);
-- ... truncated (46 lines total)
```

### email_sendability(p_email text) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 5 lines
```sql

BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  RETURN email_sendability_body(p_email);
END 
```

### email_sendability_body(p_email text) -> jsonb
- sql | invoker | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 145 lines
```sql

  WITH k AS (SELECT identity_key('email', p_email) AS e),
  dm AS (SELECT lower(split_part((SELECT e FROM k), '@', 2)) AS d),
  w AS (SELECT (email_check_windows()->>'positive_check_valid_days')::int AS fresh_days),
  ev AS (SELECT v.* FROM quarantine.email_evidence v, k WHERE v.email_norm = k.e),
  dv AS (SELECT v.* FROM quarantine.email_domain_evidence v, dm WHERE v.domain = dm.d),
  dead_domain AS (
    SELECT method, checked_at FROM dv
     WHERE (method = 'dns_resolve' AND result = 'fail')
        OR (method = 'domain_mx'   AND result = 'fail')
        OR (method = 'disposable'  AND result = 'pass')
        OR (method = 'parked'      AND result = 'pass')
     ORDER BY checked_at LIMIT 1),
  fatal AS (
    SELECT method, checked_at, detail FROM ev
-- ... truncated (145 lines total)
```

### geo_apply(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 35 lines
```sql

DECLARE me uuid; is_sched boolean; lim int; n_aliases int; n_rows int;
BEGIN
  me       := auth_staff_id();
  is_sched := session_user IN ('postgres','supabase_admin');
  IF me IS NULL AND NOT is_sched AND coalesce(auth.role(),'') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  lim := least(greatest(coalesce(p_limit, 200), 1), 5000);

  WITH due AS (
    SELECT * FROM quarantine.geo_aliases
     WHERE verdict = 'approved' AND applied_at IS NULL
     ORDER BY rows_affected DESC LIMIT lim),
  written AS (
    UPDATE quarantine.research_contacts rc
       SET state_norm   = due.canonical_state,
           country_norm = coalesce(due.canonical_country,
                                   nullif(btrim(coalesce(rc.country_raw,'')),'')),
           geo_norm_at  = now()
      FROM due
     WHERE lower(btrim(rc.state_raw)) = due.raw_state
       AND rc.promoted_person_id IS NULL
    RETURNING 1),
  marked AS (
    UPDATE quarantine.geo_aliases a SET applied_at = now()
      FROM due WHERE a.raw_state = due.raw_state
    RETURNING 1)
  SELECT (SELECT count(*) FROM marked), (SELECT count(*) FROM written)
    INTO n_aliases, n_rows;

  RETURN jsonb_build_object('ok', true, 'aliases_applied', n_aliases, 'rows_written', n_rows,
    'still_approved_unapplied', (SELECT count(*) FROM quarantine.geo_aliases
                                  WHERE verdict = 'approved' AND applied_at IS NULL),
    'how', 'state_norm and country_norm are written beside the raw values, never over them, so a better rule later can simply run again.');
END 
```

### geo_decide(p_raw_state text, p_verdict text, p_state text, p_country text, p_reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 36 lines
```sql

DECLARE me uuid; n int;
BEGIN
  me := auth_staff_id();
  IF me IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
                        OR has_role('engineer')) THEN
    RAISE EXCEPTION 'Only an owner, admin, manager or engineer may decide a geography alias'
      USING ERRCODE='42501'; END IF;
  IF p_verdict NOT IN ('approved','rejected','held') THEN
    RAISE EXCEPTION 'Verdict must be approved, rejected or held' USING ERRCODE='22023'; END IF;

  UPDATE quarantine.geo_aliases
     SET verdict           = p_verdict,
         canonical_state   = CASE WHEN p_verdict = 'approved'
                                  THEN nullif(btrim(coalesce(p_state,'')),'')
                                  ELSE canonical_state END,
         canonical_country = CASE WHEN p_verdict = 'approved'
                                  THEN coalesce(nullif(btrim(coalesce(p_country,'')),''),
                                                canonical_country)
                                  ELSE canonical_country END,
         confidence        = CASE WHEN p_verdict = 'approved' THEN 1.000 ELSE confidence END,
         reason            = coalesce(nullif(btrim(coalesce(p_reason,'')),''), reason)
                             || ' | ' || p_verdict || ' by a person on '
                             || to_char(now(), 'DD Mon YYYY'),
         decided_by        = me,
         decided_at        = now()
   WHERE raw_state = lower(btrim(p_raw_state));
  GET DIAGNOSTICS n = ROW_COUNT;
  IF n = 0 THEN RAISE EXCEPTION 'No such raw state value %', p_raw_state USING ERRCODE='P0002'; END IF;

  RETURN jsonb_build_object('ok', true, 'raw_state', lower(btrim(p_raw_state)),
    'verdict', p_verdict,
    'rows_it_settles', (SELECT rows_affected FROM quarantine.geo_aliases
                         WHERE raw_state = lower(btrim(p_raw_state))),
    'next', 'geo_apply() writes it onto the rows. Nothing is written until it runs.');
END 
```

### geo_propose() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 9 lines
```sql

DECLARE me uuid; is_sched boolean;
BEGIN
  me       := auth_staff_id();
  is_sched := session_user IN ('postgres','supabase_admin');
  IF me IS NULL AND NOT is_sched AND coalesce(auth.role(),'') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  RETURN geo_propose_body(CASE WHEN me IS NULL THEN 'agent:geo-hygiene' ELSE me::text END);
END 
```

### geo_propose_body(p_by text) -> jsonb
- sql | invoker | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 118 lines
```sql

  WITH floor_n AS (SELECT coalesce(agent_setting('geo_country_min_states'), 50)::int AS min_states),
  r AS (
    SELECT lower(btrim(rc.state_raw)) AS st,
           lower(btrim(coalesce(rc.country_raw,''))) AS co,
           nullif(btrim(coalesce(rc.city_raw,'')),'') AS city
      FROM quarantine.research_contacts rc
     WHERE rc.promoted_person_id IS NULL
       AND nullif(btrim(coalesce(rc.state_raw,'')),'') IS NOT NULL
  ),
  as_country AS (
    SELECT co AS v, count(DISTINCT st) FILTER (WHERE st <> co) AS states_under_it
      FROM r WHERE co <> '' GROUP BY co
  ),
  agg AS (
-- ... truncated (118 lines total)
```

### geo_queue(p_kind text, p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 29 lines
```sql

DECLARE out jsonb; lim int;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  lim := least(greatest(coalesce(p_limit, 50), 1), 500);

  SELECT jsonb_build_object(
    'rows', coalesce((SELECT jsonb_agg(jsonb_build_object(
        'raw_state', raw_state, 'kind', kind, 'rows', rows_affected,
        'countries', countries, 'cities', sample_cities, 'reason', reason)
        ORDER BY rows_affected DESC)
      FROM (SELECT * FROM quarantine.geo_aliases
             WHERE verdict IS NULL AND (p_kind IS NULL OR kind = p_kind)
             ORDER BY rows_affected DESC LIMIT lim) q), '[]'::jsonb),
    'by_kind', coalesce((SELECT jsonb_object_agg(kind, jsonb_build_object('values', v, 'rows', n))
      FROM (SELECT kind, count(*) AS v, sum(rows_affected) AS n
              FROM quarantine.geo_aliases WHERE verdict IS NULL GROUP BY kind) z), '{}'::jsonb),
    'waiting',        (SELECT count(*) FROM quarantine.geo_aliases WHERE verdict IS NULL),
    'rows_waiting',   (SELECT coalesce(sum(rows_affected),0) FROM quarantine.geo_aliases
                        WHERE verdict IS NULL),
    'decided',        (SELECT count(*) FROM quarantine.geo_aliases WHERE verdict IS NOT NULL),
    'rows_decided',   (SELECT coalesce(sum(rows_affected),0) FROM quarantine.geo_aliases
                        WHERE verdict IS NOT NULL),
    'applied',        (SELECT count(*) FROM quarantine.geo_aliases WHERE applied_at IS NOT NULL),
    'why_ordered_this_way',
      'By rows, because the top fifty values are 57% of the data and the eight thousand rarest are 3.5% between them. Answering ten of these is worth more than a week on the tail.')
    INTO out;
  RETURN out;
END 
```

### gin_extract_query_trgm(text, internal, smallint, internal, internal, internal, internal) -> internal
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gin_extract_query_trgm
```

### gin_extract_value_trgm(text, internal) -> internal
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gin_extract_value_trgm
```

### gin_trgm_consistent(internal, smallint, text, integer, internal, internal, internal, internal) -> boolean
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gin_trgm_consistent
```

### gin_trgm_triconsistent(internal, smallint, text, integer, internal, internal, internal) -> "char"
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gin_trgm_triconsistent
```

### governance_summary() -> jsonb
- sql | invoker | stable | config: search_path=public | exec anon=n authenticated=Y | 25 lines
```sql

  SELECT jsonb_build_object(
    'consents', jsonb_build_object(
      'total', (SELECT count(*) FROM consents),
      'granted', (SELECT count(*) FROM consents WHERE granted_at IS NOT NULL AND revoked_at IS NULL),
      'revoked', (SELECT count(*) FROM consents WHERE revoked_at IS NOT NULL),
      'by_basis', coalesce((SELECT jsonb_agg(jsonb_build_object('basis',b.basis,'n',b.n) ORDER BY b.n DESC)
                    FROM (SELECT basis::text AS basis, count(*) AS n FROM consents GROUP BY 1) b),'[]'::jsonb)),
    'suppressions', jsonb_build_object(
      'total', (SELECT count(*) FROM suppressions),
      'by_reason', coalesce((SELECT jsonb_agg(jsonb_build_object('reason',r.reason,'n',r.n) ORDER BY r.n DESC)
                    FROM (SELECT reason::text AS reason, count(*) AS n FROM suppressions GROUP BY 1) r),'[]'::jsonb)),
    'autonomy', jsonb_build_object(
      'rows', (SELECT count(*) FROM autonomy_thresholds),
      'above_zero', (SELECT count(*) FROM autonomy_thresholds WHERE threshold > 0),
      'agents', coalesce((SELECT jsonb_agg(jsonb_build_object('agent',a.agent,'segment',a.segment,
                    'threshold',a.threshold,'approval_rate',a.approval_rate,'runs_behind',a.runs_behind)
                    ORDER BY a.agent) FROM autonomy_thresholds a),'[]'::jsonb)),
    'approved_facts', jsonb_build_object(
      'total', (SELECT count(*) FROM approved_facts),
      'live', (SELECT count(*) FROM approved_facts WHERE expires_on IS NULL OR expires_on >= current_date),
      'expired', (SELECT count(*) FROM approved_facts WHERE expires_on < current_date)),
    'minors_without_parent', (SELECT count(*) FROM people_v p WHERE p.is_minor
       AND NOT EXISTS (SELECT 1 FROM household_members m WHERE m.person_id=p.id)));

```

### gtrgm_compress(internal) -> internal
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_compress
```

### gtrgm_consistent(internal, text, smallint, oid, internal) -> boolean
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_consistent
```

### gtrgm_decompress(internal) -> internal
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_decompress
```

### gtrgm_distance(internal, text, smallint, oid, internal) -> double precision
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_distance
```

### gtrgm_in(cstring) -> gtrgm
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_in
```

### gtrgm_options(internal) -> void
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_options
```

### gtrgm_out(gtrgm) -> cstring
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_out
```

### gtrgm_penalty(internal, internal, internal) -> internal
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_penalty
```

### gtrgm_picksplit(internal, internal) -> internal
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_picksplit
```

### gtrgm_same(gtrgm, gtrgm, internal) -> internal
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_same
```

### gtrgm_union(internal, internal) -> gtrgm
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
gtrgm_union
```

### guard_source_code() -> trigger
- plpgsql | invoker | volatile | config: search_path=public | exec anon=n authenticated=n | 11 lines
```sql

BEGIN
  IF TG_OP = 'UPDATE' AND OLD.source_code IS DISTINCT FROM NEW.source_code THEN
    RAISE EXCEPTION 'source_code is immutable (was %, tried %)', OLD.source_code, NEW.source_code;
  END IF;
  IF NEW.source_code !~ '^SRC-[A-Z0-9]+-[0-9]{6}-[A-Z0-9-]+$' THEN
    RAISE EXCEPTION 'malformed source_code: %', NEW.source_code;
  END IF;
  RETURN NEW;
END;

```

### has_grant(g staff_grant) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 6 lines
```sql

  SELECT EXISTS (SELECT 1 FROM staff_grants x
                  WHERE x.staff_id = auth_staff_id()
                    AND x.grant_key = g
                    AND x.expires_on >= current_date)

```

### has_role(r staff_role) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 4 lines
```sql

  SELECT EXISTS (SELECT 1 FROM staff_role_assignments a
                  WHERE a.staff_id = auth_staff_id() AND a.role = r)

```

### has_scope(kind text, val text) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 6 lines
```sql

  SELECT EXISTS (SELECT 1 FROM staff_scopes s
                  WHERE s.staff_id = auth_staff_id()
                    AND s.scope_kind = kind
                    AND s.scope_value = val)

```

### has_scope_all() -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 4 lines
```sql

  SELECT EXISTS (SELECT 1 FROM staff_scopes s
                  WHERE s.staff_id = auth_staff_id() AND s.scope_kind = 'all')

```

### identity_key(p_kind text, p_value text) -> text
- sql | invoker | immutable | config: search_path=public, pg_temp | exec anon=Y authenticated=Y | 49 lines
- comment: The single definition of "the same identity", matching packages/core/src/identity/normalise.ts byte for byte. The parity test in identityKey.test.ts is the proof; if the two ever diverge, one human becomes two rows. 038.
```sql

  WITH t AS (
    SELECT nullif(btrim(lower(coalesce(p_value, '')), E' \t\n\r\f\v'), '') AS x
  )
  SELECT CASE
    WHEN (SELECT x FROM t) IS NULL THEN NULL

    -- normEmail: EMAIL_SHAPE = /^[^\s@]+@[^\s@.]+(?:\.[^\s@.]+)+$/
    WHEN p_kind = 'email' THEN
      CASE WHEN (SELECT x FROM t) ~ '^[^\s@]+@[^\s@.]+(\.[^\s@.]+)+$'
           THEN (SELECT x FROM t) END

    -- normPhone, without a default country. SQL has no per-row country to pass, so only the
    -- already-international forms resolve; anything else is refused rather than guessed, which
    -- is what the TypeScript does when defaultCountry is undefined.
-- ... truncated (49 lines total)
```

### import_commit(p_batch_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 98 lines
```sql

DECLARE
  sid uuid; b quarantine.import_batches%ROWTYPE; v record; cm jsonb;
  raw_row jsonb; res jsonb; expect int; got int; cand uuid;
BEGIN
  sid := auth_staff_id();
  IF sid IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'You do not have a role that may import contacts' USING ERRCODE='42501'; END IF;

  SELECT * INTO b FROM quarantine.import_batches WHERE id = p_batch_id FOR UPDATE;
  IF b.id IS NULL THEN RAISE EXCEPTION 'No such import batch' USING ERRCODE='02000'; END IF;
  IF b.committed_at IS NOT NULL THEN
    RAISE EXCEPTION 'That batch was already committed at %', b.committed_at USING ERRCODE='42501'; END IF;

-- ... truncated (98 lines total)
```

### import_create(p_label text, p_file_name text, p_headers text[], p_rows jsonb, p_kind text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 52 lines
```sql

DECLARE sid uuid; fp text; b uuid; remembered jsonb; n int; kd text;
BEGIN
  sid := auth_staff_id();
  IF sid IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'You do not have a role that may import contacts' USING ERRCODE='42501'; END IF;

  IF p_headers IS NULL OR array_length(p_headers, 1) IS NULL THEN
    RAISE EXCEPTION 'That file has no header row, so no column can be mapped' USING ERRCODE='22023'; END IF;
  IF jsonb_typeof(p_rows) <> 'array' THEN
    RAISE EXCEPTION 'Rows must be a JSON array' USING ERRCODE='22023'; END IF;

  n := jsonb_array_length(p_rows);
  IF n = 0 THEN
-- ... truncated (52 lines total)
```

### import_detail(p_batch_id uuid, p_limit integer, p_offset integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 69 lines
```sql

DECLARE b quarantine.import_batches%ROWTYPE; lim int; off int;
        counts jsonb; rows_out jsonb; missing text[]; unmapped text[];
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;

  SELECT * INTO b FROM quarantine.import_batches WHERE id = p_batch_id;
  IF b.id IS NULL THEN RAISE EXCEPTION 'No such import batch' USING ERRCODE='02000'; END IF;

  lim := least(greatest(coalesce(p_limit, 50), 1), 200);
  off := greatest(coalesce(p_offset, 0), 0);

  SELECT array_agg(f->>'field' ORDER BY f->>'field') INTO missing
    FROM jsonb_array_elements(import_field_list(b.kind)) f
   WHERE (f->>'required')::boolean
-- ... truncated (69 lines total)
```

### import_field_list(p_kind text) -> jsonb
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=Y | 22 lines
```sql

  SELECT jsonb_agg(jsonb_build_object('field', f, 'label', l, 'required', req) ORDER BY ord)
    FROM (VALUES
      ('research_contact','school',      'School',            true,  1),
      ('research_contact','city',        'City',              false, 2),
      ('research_contact','state',       'State / province',  false, 3),
      ('research_contact','country',     'Country',           false, 4),
      ('research_contact','contact',     'Contact name',      true,  5),
      ('research_contact','designation', 'Designation',       true,  6),
      ('research_contact','email',       'Email',             true,  7),
      ('research_contact','domain',      'School domain',     false, 8),
      ('research_contact','phone',       'Phone',             false, 9),
      ('organisation','school',   'School name',      true,  1),
      ('organisation','country',  'Country',          false, 2),
      ('organisation','city',     'City',             false, 3),
      ('organisation','domain',   'Domain',           false, 4),
      ('organisation','website',  'Website',          false, 5),
      ('organisation','linkedin', 'LinkedIn URL',     false, 6),
      ('organisation','industry', 'Type or industry', false, 7)
    ) AS v(kind, f, l, req, ord)
   WHERE kind = coalesce(nullif(btrim(coalesce(p_kind,'')),''), 'research_contact')

```

### import_guess_kind(p_headers text[]) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=Y | 8 lines
```sql

  WITH h AS (SELECT regexp_replace(lower(btrim(x)), '[^a-z0-9]+', ' ', 'g') AS n FROM unnest(p_headers) x)
  SELECT CASE
    WHEN EXISTS (SELECT 1 FROM h WHERE n ~ '^(e ?mail|mail id|mail)') THEN 'research_contact'
    WHEN EXISTS (SELECT 1 FROM h WHERE n ~ '(website|linkedin|employee count|follower)') THEN 'organisation'
    ELSE 'research_contact'
  END

```

### import_guess_map(p_headers text[], p_kind text) -> jsonb
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 33 lines
```sql

  WITH h AS (
    SELECT ord, raw, regexp_replace(lower(btrim(raw)), '[^a-z0-9]+', ' ', 'g') AS n
      FROM unnest(p_headers) WITH ORDINALITY AS t(raw, ord)
  ),
  pat(kind, field, rx, pri) AS (VALUES
    ('research_contact','phone',       '^(phone|mobile|cell|telephone|whatsapp|contact (number|no|phone))', 1),
    ('research_contact','email',       '^(e ?mail|mail id|mail)',                                           1),
    ('research_contact','designation', '^(designation|job title|title|role|position)',                      1),
    ('research_contact','contact',     '^(contact name|contact person|full name|person name|name|contact)$',1),
    ('research_contact','school',      '^(school|institution|organisation|organization|college|academy|org|company)( name)?$', 1),
    ('research_contact','domain',      '(domain|website|web site|url)',                                     1),
    ('research_contact','country',     '^country',                                                          1),
    ('research_contact','state',       '^(state|province|region)',                                          1),
    ('research_contact','city',        '^(city|town|district)',                                             1),
    ('research_contact','school',      '^(school|institution|organisation|organization)',                   3),
    ('organisation','school',   '^(school|company|organisation|organization|institution|name)( name)?$',    1),
    ('organisation','domain',   '^domain$',                                                                 1),
    ('organisation','linkedin', 'linkedin',                                                                 1),
    ('organisation','website',  '^(website|web site|url|site|homepage)',                                    1),
    ('organisation','country',  '^country',                                                                 1),
    ('organisation','city',     '^(city|town)',                                                             1),
    ('organisation','industry', '^(industry|school type|type|category|segment|sector)',                     1),
    ('organisation','school',   '^(school|company|organisation|organization|institution)',                  3)
  ),
  m AS (
    SELECT DISTINCT ON (p.field) p.field, h.raw
      FROM h JOIN pat p ON h.n ~ p.rx
     WHERE p.kind = coalesce(nullif(btrim(coalesce(p_kind,'')),''), 'research_contact')
     ORDER BY p.field, p.pri, h.ord
  )
  SELECT coalesce(jsonb_object_agg(field, raw), '{}'::jsonb) FROM m

```

### import_list(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 22 lines
```sql

DECLARE out jsonb;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;

  SELECT jsonb_agg(jsonb_build_object(
      'id', b.id, 'label', b.label, 'file_name', b.file_name, 'kind', b.kind,
      'uploaded_at', b.uploaded_at, 'uploaded_by', s.full_name,
      'committed_at', b.committed_at, 'stopped_reason', b.stopped_reason,
      'map_source', b.map_source,
      'rows_total', (SELECT count(*) FROM quarantine.import_rows ir WHERE ir.batch_id = b.id),
      'saved',      (SELECT count(*) FROM quarantine.import_rows ir
                      WHERE ir.batch_id = b.id AND ir.verdict = 'saved'),
      'refused',    (SELECT count(*) FROM quarantine.import_rows ir
                      WHERE ir.batch_id = b.id AND ir.verdict IS NOT NULL AND ir.verdict <> 'saved')
    ) ORDER BY b.uploaded_at DESC) INTO out
    FROM (SELECT * FROM quarantine.import_batches
           ORDER BY uploaded_at DESC LIMIT least(greatest(coalesce(p_limit, 25), 1), 100)) b
    LEFT JOIN public.staff s ON s.id = b.uploaded_by;

  RETURN coalesce(out, '[]'::jsonb);
END 
```

### import_row_verdicts(p_batch_id uuid) -> TABLE(o_row_no integer, o_row_id uuid, o_key text, o_verdict text, o_reason text, o_person_id uuid, o_contact_id uuid, o_org_id uuid, o_first_row integer)
- sql | invoker | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 100 lines
```sql

  WITH b AS (SELECT column_map AS cm, kind FROM quarantine.import_batches WHERE id = p_batch_id),
  r AS (
    SELECT ir.id, ir.row_no,
           nullif(btrim(coalesce(ir.raw ->> (b.cm ->> 'email'),       '')), '') AS email,
           nullif(btrim(coalesce(ir.raw ->> (b.cm ->> 'school'),      '')), '') AS school,
           nullif(btrim(coalesce(ir.raw ->> (b.cm ->> 'contact'),     '')), '') AS contact,
           nullif(btrim(coalesce(ir.raw ->> (b.cm ->> 'designation'), '')), '') AS designation
      FROM quarantine.import_rows ir CROSS JOIN b
     WHERE ir.batch_id = p_batch_id AND b.kind = 'research_contact'
  ),
  k  AS (SELECT r.*, identity_key('email', r.email) AS ek FROM r),
  ff AS (SELECT k.ek AS ek, min(k.row_no) AS first_row FROM k WHERE k.ek IS NOT NULL GROUP BY k.ek),
  j  AS (
    SELECT k.*, ff.first_row,
-- ... truncated (100 lines total)
```

### import_set_kind(p_batch_id uuid, p_kind text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 28 lines
```sql

DECLARE sid uuid; b quarantine.import_batches%ROWTYPE; fp text; remembered jsonb;
BEGIN
  sid := auth_staff_id();
  IF sid IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'You do not have a role that may import contacts' USING ERRCODE='42501'; END IF;
  IF p_kind NOT IN ('research_contact','organisation') THEN
    RAISE EXCEPTION 'A sheet is either contacts or schools' USING ERRCODE='22023'; END IF;

  SELECT * INTO b FROM quarantine.import_batches WHERE id = p_batch_id FOR UPDATE;
  IF b.id IS NULL THEN RAISE EXCEPTION 'No such import batch' USING ERRCODE='02000'; END IF;
  IF b.committed_at IS NOT NULL THEN
    RAISE EXCEPTION 'That batch is committed. What its rows became is part of what happened'
      USING ERRCODE='42501'; END IF;

  fp := md5(p_kind || '|' || array_to_string(
          ARRAY(SELECT lower(btrim(h)) FROM unnest(b.headers) h ORDER BY 1), '|'));
  SELECT column_map INTO remembered FROM quarantine.import_header_maps WHERE header_fingerprint = fp;

  UPDATE quarantine.import_batches
     SET kind = p_kind, header_fingerprint = fp,
         column_map = coalesce(remembered, import_guess_map(b.headers, p_kind)),
         map_source = CASE WHEN remembered IS NULL THEN 'guessed' ELSE 'remembered' END
   WHERE id = p_batch_id;

  RETURN import_detail(p_batch_id, 50, 0);
END 
```

### import_set_map(p_batch_id uuid, p_map jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 37 lines
```sql

DECLARE sid uuid; b quarantine.import_batches%ROWTYPE; bad text[];
BEGIN
  sid := auth_staff_id();
  IF sid IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'You do not have a role that may import contacts' USING ERRCODE='42501'; END IF;

  SELECT * INTO b FROM quarantine.import_batches WHERE id = p_batch_id FOR UPDATE;
  IF b.id IS NULL THEN RAISE EXCEPTION 'No such import batch' USING ERRCODE='02000'; END IF;
  IF b.committed_at IS NOT NULL THEN
    RAISE EXCEPTION 'That batch is committed. Its mapping is part of what happened, so it stays'
      USING ERRCODE='42501'; END IF;
  IF jsonb_typeof(p_map) <> 'object' THEN
    RAISE EXCEPTION 'The mapping must be an object of core field to column' USING ERRCODE='22023'; END IF;

  SELECT array_agg(k) INTO bad FROM jsonb_object_keys(p_map) k
   WHERE k NOT IN (SELECT f->>'field' FROM jsonb_array_elements(import_field_list()) f);
  IF bad IS NOT NULL THEN
    RAISE EXCEPTION 'Not core fields: %', array_to_string(bad, ', ') USING ERRCODE='22023'; END IF;

  SELECT array_agg(z.v) INTO bad
    FROM (SELECT p_map ->> k AS v FROM jsonb_object_keys(p_map) k) z
   WHERE z.v IS NOT NULL AND NOT (z.v = ANY(b.headers));
  IF bad IS NOT NULL THEN
    RAISE EXCEPTION 'Not columns in that file: %', array_to_string(bad, ', ') USING ERRCODE='22023'; END IF;

  UPDATE quarantine.import_batches
     SET column_map = p_map, map_source = 'set_by_hand' WHERE id = p_batch_id;

  INSERT INTO quarantine.import_header_maps (header_fingerprint, headers, column_map, updated_by)
  VALUES (b.header_fingerprint, b.headers, p_map, sid)
  ON CONFLICT (header_fingerprint) DO UPDATE
    SET column_map = excluded.column_map, updated_by = excluded.updated_by, updated_at = now();

  RETURN import_detail(p_batch_id, 50, 0);
END 
```

### inbound_queue() -> jsonb
- sql | invoker | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 28 lines
- comment: Per-app ingest state and queue depth for the console. resolver_present is false until packages/core/identity exists. 030.
```sql

  SELECT jsonb_build_object(
    'systems', coalesce((
      SELECT jsonb_agg(jsonb_build_object(
               'key', s.key, 'label', s.label,
               'has_key', (s.ingest_key_hash IS NOT NULL),
               'key_set_at', s.ingest_key_set_at,
               'ingest_enabled', s.ingest_enabled,
               'pending', coalesce(q.pending, 0),
               'processed', coalesce(q.processed, 0),
               'rejected', coalesce(q.rejected, 0),
               'last_received', q.last_received)
             ORDER BY s.sort_order)
        FROM public.source_systems s
        LEFT JOIN (
          SELECT system_key,
                 count(*) FILTER (WHERE processed_at IS NULL AND rejected_at IS NULL) AS pending,
                 count(*) FILTER (WHERE processed_at IS NOT NULL) AS processed,
                 count(*) FILTER (WHERE rejected_at IS NOT NULL) AS rejected,
                 max(received_at) AS last_received
            FROM public.inbound_records GROUP BY system_key) q ON q.system_key = s.key
       WHERE s.is_expected), '[]'::jsonb),
    'totals', jsonb_build_object(
      'pending',   (SELECT count(*) FROM public.inbound_records WHERE processed_at IS NULL AND rejected_at IS NULL),
      'processed', (SELECT count(*) FROM public.inbound_records WHERE processed_at IS NOT NULL),
      'rejected',  (SELECT count(*) FROM public.inbound_records WHERE rejected_at IS NOT NULL),
      'resolver_present', false));

```

### link_meeting_attendees(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 46 lines
```sql

DECLARE m record; v_person uuid; v_hits int; v_src text;
        created int:=0; linked int:=0; ambiguous int:=0; minors int:=0;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may link meeting attendees' USING ERRCODE='42501'; END IF;

  FOR m IN SELECT id, attendee_email_norm, attendee_name_raw, attendee_designation,
                  attendee_country, source_code
             FROM meetings
            WHERE person_id IS NULL AND attendee_email_norm IS NOT NULL
            ORDER BY starts_at LIMIT least(greatest(coalesce(p_limit,200),1),1000) LOOP
    v_person := NULL;
    SELECT count(DISTINCT pi.person_id) INTO v_hits FROM person_identities pi
-- ... truncated (46 lines total)
```

### link_pitched_programme() -> trigger
- plpgsql | invoker | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=n | 7 lines
```sql

BEGIN
  IF NEW.programme_pitched_raw IS NOT NULL AND NEW.programme_pitched_id IS NULL THEN
    NEW.programme_pitched_id := programme_for_pitch(NEW.programme_pitched_raw);
  END IF;
  RETURN NEW;
END 
```

### log_sensitive_access(p_grant_key staff_grant, p_subject_kind text, p_subject_id uuid, p_context text) -> void
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=Y | 9 lines
- comment: The only way to write sensitive_access_log. Stamps staff_id from the session, so a caller cannot attribute an access to somebody else.
```sql

BEGIN
  IF auth.role() IN ('anon','authenticated') AND public.auth_staff_id() IS NULL THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE = '42501';
  END IF;
  INSERT INTO sensitive_access_log (staff_id, grant_key, subject_kind, subject_id, context)
  VALUES (auth_staff_id(), p_grant_key, p_subject_kind, p_subject_id, left(p_context, 500));
END;

```

### me() -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 20 lines
- comment: The caller's own identity. Zero arguments, so it cannot be pointed at anybody else. For a non-active account it returns is_staff false plus request_status, so the console can tell a pending joiner they are pending. A caller with no staff row gets exactly {"is_staff": false}.
```sql

  WITH mine AS (SELECT s.* FROM staff s WHERE s.auth_user_id = auth.uid())
  SELECT CASE
    WHEN a.id IS NOT NULL THEN jsonb_build_object(
      'is_staff', true, 'staff_id', a.id, 'email', a.email, 'full_name', a.full_name,
      'job_title', a.job_title, 'status', a.status,
      'roles', coalesce((SELECT jsonb_agg(ra.role ORDER BY ra.role) FROM staff_role_assignments ra WHERE ra.staff_id=a.id),'[]'::jsonb),
      'grants', coalesce((SELECT jsonb_agg(jsonb_build_object('key',g.grant_key,'expires_on',g.expires_on,'days_left',g.expires_on-current_date) ORDER BY g.grant_key)
                            FROM staff_grants g WHERE g.staff_id=a.id AND g.expires_on >= current_date),'[]'::jsonb),
      'scopes', coalesce((SELECT jsonb_agg(jsonb_build_object('kind',sc.scope_kind,'value',sc.scope_value) ORDER BY sc.scope_kind, sc.scope_value)
                            FROM staff_scopes sc WHERE sc.staff_id=a.id),'[]'::jsonb))
    ELSE jsonb_strip_nulls(jsonb_build_object(
      'is_staff', false,
      'request_status', (SELECT m.status::text FROM mine m),
      'requested_at', (SELECT m.requested_at FROM mine m),
      'decision_note', (SELECT m.decision_note FROM mine m WHERE m.status='rejected')))
  END
  FROM (SELECT NULL::uuid AS dummy) d
  LEFT JOIN staff a ON a.id = auth_staff_id();

```

### meeting_dataset(p_meeting_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 29 lines
```sql

DECLARE r record; m_starts timestamptz; any_hit boolean;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;

  SELECT * INTO r FROM meeting_dataset_pairs() WHERE o_meeting_id = p_meeting_id;
  IF r.o_dataset_id IS NOT NULL THEN
    RETURN jsonb_build_object(
      'attributed', true,
      'dataset_id', r.o_dataset_id, 'code', r.o_code,
      'built_by', (SELECT full_name FROM public.staff WHERE id = r.o_built_by),
      'contact_id', r.o_contact_id, 'why', r.o_why);
  END IF;

  SELECT m.starts_at INTO m_starts FROM public.meetings m WHERE m.id = p_meeting_id;
  IF m_starts IS NULL THEN RAISE EXCEPTION 'No such meeting' USING ERRCODE='02000'; END IF;

  SELECT EXISTS (
    SELECT 1 FROM public.meetings m
      JOIN quarantine.research_contacts rc ON rc.email_norm = m.attendee_email_norm
     WHERE m.id = p_meeting_id) INTO any_hit;

  RETURN jsonb_build_object('attributed', false,
    'why', CASE WHEN any_hit
      THEN 'This attendee is a researched contact, but every version of their list was built after '
        || 'the meeting happened, so none of them can have produced it.'
      ELSE 'This attendee does not appear in the research data, so no dataset can claim the meeting.'
      END);
END 
```

### meeting_dataset_pairs() -> TABLE(o_meeting_id uuid, o_dataset_id uuid, o_contact_id uuid, o_code text, o_built_by uuid, o_why text)
- sql | invoker | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 27 lines
```sql

  WITH hit AS (
    SELECT m.id AS meeting_id, m.starts_at, rc.id AS contact_id
      FROM public.meetings m
      JOIN quarantine.research_contacts rc ON rc.email_norm = m.attendee_email_norm
     WHERE m.attendee_email_norm IS NOT NULL
    UNION
    SELECT m.id, m.starts_at, rc.id
      FROM public.meetings m
      JOIN public.person_identities pi ON pi.person_id = m.person_id AND pi.kind = 'email'
      JOIN quarantine.research_contacts rc ON rc.email_norm = pi.value_norm
     WHERE m.person_id IS NOT NULL
  ),
  cand AS (
    SELECT h.meeting_id, h.contact_id, d.id AS dataset_id, d.built_at, d.code, d.built_by
      FROM hit h
      JOIN quarantine.dataset_rows dr ON dr.research_contact_id = h.contact_id
      JOIN quarantine.datasets d      ON d.id = dr.dataset_id
     WHERE d.built_at <= h.starts_at
  )
  SELECT DISTINCT ON (c.meeting_id)
         c.meeting_id, c.dataset_id, c.contact_id, c.code, c.built_by,
         'Newest version of this list that already existed when the meeting happened — '
           || c.code || ', built ' || to_char(c.built_at, 'DD Mon YYYY')
    FROM cand c
   ORDER BY c.meeting_id, c.built_at DESC, c.dataset_id

```

### meeting_detail(p_meeting_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 106 lines
- comment: Everything on one meeting, in one call. Who the attendee is needs pii_contact; what was said needs comms_read; the operational assessments need neither.
```sql

DECLARE
  m meetings%ROWTYPE; see_pii boolean; see_comms boolean;
  hidden constant text := '—';
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  SELECT * INTO m FROM meetings WHERE id = p_meeting_id;
  IF m.id IS NULL THEN RAISE EXCEPTION 'No such meeting %', p_meeting_id USING ERRCODE='P0002'; END IF;
  see_pii := has_grant('pii_contact'); see_comms := has_grant('comms_read');

  RETURN jsonb_build_object(
    'can_see_attendee', see_pii, 'can_see_comms', see_comms,
    'meeting', jsonb_build_object(
      'id', m.id, 'starts_at', m.starts_at, 'ends_at', m.ends_at,
      'duration_minutes', m.duration_minutes,
-- ... truncated (106 lines total)
```

### meeting_status_of(p_raw text) -> meeting_status
- sql | invoker | immutable | config: search_path=public, pg_temp | exec anon=Y authenticated=Y | 9 lines
```sql

  SELECT CASE lower(btrim(replace(coalesce(p_raw,''), '_', ' ')))
    WHEN 'completed' THEN 'completed'::meeting_status
    WHEN 'complete' THEN 'completed' WHEN 'attended' THEN 'completed'
    WHEN 'no show' THEN 'no_show'    WHEN 'noshow' THEN 'no_show'
    WHEN 'scheduled' THEN 'scheduled' WHEN 'upcoming' THEN 'scheduled'
    WHEN 'rescheduled' THEN 'rescheduled'
    WHEN 'cancelled' THEN 'cancelled' WHEN 'canceled' THEN 'cancelled'
  END; 
```

### meeting_to_activity() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=n | 18 lines
```sql

BEGIN
  IF NEW.person_id IS NULL OR NEW.status NOT IN ('completed','no_show') THEN RETURN NEW; END IF;
  IF TG_OP = 'UPDATE' AND OLD.status = NEW.status
     AND OLD.person_id IS NOT DISTINCT FROM NEW.person_id THEN RETURN NEW; END IF;

  INSERT INTO activities (occurred_at, person_id, organization_id, kind, actor, source_code, payload)
  SELECT NEW.starts_at, NEW.person_id, NEW.organization_id,
         CASE NEW.status WHEN 'completed' THEN 'meeting_held' ELSE 'meeting_no_show' END,
         'system:meetings', NEW.source_code,
         jsonb_build_object('meeting_id', NEW.id, 'source', NEW.source::text,
                            'duration_minutes', NEW.duration_minutes,
                            'programme_pitched', NEW.programme_pitched_raw)
   WHERE NOT EXISTS (SELECT 1 FROM activities a
                      WHERE a.person_id = NEW.person_id AND a.occurred_at = NEW.starts_at
                        AND a.payload->>'meeting_id' = NEW.id::text);
  RETURN NEW;
END 
```

### meetings_calendar(p_from date, p_to date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 37 lines
```sql

DECLARE see_pii boolean; see_comms boolean;
  lo date := coalesce(p_from, (now() - interval '30 days')::date);
  hi date := coalesce(p_to,   (now() + interval '60 days')::date);
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  IF hi < lo OR (hi - lo) > 400 THEN
    RAISE EXCEPTION 'Range must be between 0 and 400 days' USING ERRCODE='22023'; END IF;
  see_pii := has_grant('pii_contact'); see_comms := has_grant('comms_read');
  RETURN jsonb_build_object('from', lo, 'to', hi,
    'can_see_attendee', see_pii, 'can_see_notes', see_comms,
    'meetings', coalesce((
      SELECT jsonb_agg(jsonb_build_object(
        'id', m.id, 'starts_at', m.starts_at,
        'date', (m.starts_at AT TIME ZONE 'UTC')::date,
        'duration_minutes', m.duration_minutes, 'status', m.status, 'source', m.source,
        'person_id', m.person_id, 'organization_id', m.organization_id,
        'attendee', CASE WHEN see_pii THEN coalesce(m.attendee_name_raw,'—') ELSE '—' END,
        'school',   CASE WHEN see_pii THEN m.school_name_raw ELSE '—' END,
        -- `notes` is the human note (0 rows have one); the agent's summary lives in ai_summary
        -- since 045, and both follow comms_read.
        'notes',    CASE WHEN see_comms THEN coalesce(m.notes, m.ai_summary) ELSE NULL END,
        'notes_origin', CASE WHEN m.notes IS NOT NULL THEN m.notes_origin
                             WHEN m.ai_summary IS NOT NULL THEN 'agent'::fact_origin END,
        'programme', m.programme_pitched_raw, 'programme_origin', m.programme_origin,
        -- The agent's assessment. Deliberately behind NO grant: a manager deciding who to call
        -- needs the score, and a number is not a conversation. See the note in 047.
        'interest_level', m.ai_interest_level,
        'lead_score',     m.ai_lead_score,
        'lead_temperature', m.ai_lead_temperature,
        'escalation_needed', coalesce(m.ai_escalation_needed, false),
        'attendance_review', m.attendance_review_required,
        'unresolved', (m.person_id IS NULL), 'needs_review', m.needs_review)
        ORDER BY m.starts_at)
      FROM meetings m
     WHERE (m.starts_at AT TIME ZONE 'UTC')::date BETWEEN lo AND hi), '[]'::jsonb));
END 
```

### meetings_summary() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 40 lines
```sql

DECLARE r jsonb;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  SELECT jsonb_build_object(
    'total', count(*),
    'completed', count(*) FILTER (WHERE status='completed'),
    'no_show', count(*) FILTER (WHERE status='no_show'),
    'scheduled', count(*) FILTER (WHERE status='scheduled'),
    'rescheduled', count(*) FILTER (WHERE status='rescheduled'),
    'cancelled', count(*) FILTER (WHERE status='cancelled'),
    'upcoming', count(*) FILTER (WHERE starts_at >= now()),
    'unresolved', count(*) FILTER (WHERE person_id IS NULL),
    'needs_review', count(*) FILTER (WHERE needs_review),
    'with_notes', count(*) FILTER (WHERE notes IS NOT NULL OR ai_summary IS NOT NULL),
    'notes_by_agent', count(*) FILTER (WHERE ai_summary IS NOT NULL),
    'notes_by_human', count(*) FILTER (WHERE notes IS NOT NULL),
    'with_host', count(*) FILTER (WHERE host_staff_id IS NOT NULL),
    'escalated', count(*) FILTER (WHERE ai_escalation_needed),
    'attendance_review', count(*) FILTER (WHERE attendance_review_required),
    'no_show_rate', CASE WHEN count(*) FILTER (WHERE status IN ('completed','no_show'))=0 THEN NULL
      ELSE round(100.0*count(*) FILTER (WHERE status='no_show')
               / count(*) FILTER (WHERE status IN ('completed','no_show')),1) END,
    'programme_named', count(programme_pitched_raw),
    'programme_matched', count(programme_pitched_id),
    'earliest', min(starts_at)::date, 'latest', max(starts_at)::date)
    INTO r FROM meetings;

  -- Each unlinked pitched name WITH the reason it is unlinked. 052 recorded those reasons, so the
  -- screen no longer has to say "not in the catalogue" about a name whose eight pathways all exist.
  RETURN r || jsonb_build_object('pitched_unlinked', coalesce((
    SELECT jsonb_agg(jsonb_build_object('name', x.raw, 'meetings', x.n,
                                        'kind', coalesce(a.kind, 'undecided'),
                                        'why',  a.note) ORDER BY x.n DESC)
      FROM (SELECT programme_pitched_raw AS raw, count(*) AS n
              FROM meetings
             WHERE programme_pitched_raw IS NOT NULL AND programme_pitched_id IS NULL
             GROUP BY 1) x
      LEFT JOIN programme_aliases a ON a.name_key = programme_name_key(x.raw)), '[]'::jsonb));
END 
```

### minor_for_designation(p_designation text) -> boolean
- sql | invoker | immutable | config: search_path=public, pg_temp | exec anon=Y authenticated=Y | 4 lines
```sql

  SELECT role_for_designation(p_designation) NOT IN
         ('parent'::person_role, 'educator', 'partner', 'alumni');

```

### next_dataset_code(p_code text) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=Y | 7 lines
```sql

  SELECT CASE
    WHEN p_code ~ '-[A-Y]$'
      THEN left(p_code, length(p_code) - 1) || chr(ascii(right(p_code, 1)) + 1)
    ELSE p_code || '-B'
  END

```

### norm_email(v text) -> text
- sql | invoker | immutable | config: search_path=public | exec anon=n authenticated=n | 3 lines
```sql

  SELECT lower(btrim(v));

```

### norm_phone(v text) -> text
- sql | invoker | immutable | config: search_path=public | exec anon=n authenticated=n | 3 lines
```sql

  SELECT regexp_replace(v, '[^0-9+]', '', 'g');

```

### normalise_message_direction() -> trigger
- plpgsql | invoker | volatile | config: search_path=pg_catalog, pg_temp | exec anon=n authenticated=n | 15 lines
```sql

BEGIN
  NEW.direction := CASE lower(btrim(coalesce(NEW.direction,'')))
    WHEN 'out'      THEN 'outbound'
    WHEN 'outbound' THEN 'outbound'
    WHEN 'sent'     THEN 'outbound'
    WHEN 'send'     THEN 'outbound'
    WHEN 'in'       THEN 'inbound'
    WHEN 'inbound'  THEN 'inbound'
    WHEN 'received' THEN 'inbound'
    WHEN 'reply'    THEN 'inbound'
    ELSE NEW.direction   -- unrecognised falls through to the CHECK, which refuses it
  END;
  RETURN NEW;
END 
```

### org_candidate_summary() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 24 lines
```sql

DECLARE out jsonb;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  SELECT jsonb_build_object(
    'by_kind', coalesce((SELECT jsonb_object_agg(coalesce(kind,'unclassified'),
                          jsonb_build_object('domains', d, 'contacts', c, 'confidence', conf))
        FROM (SELECT kind, count(*) AS d, sum(people_count) AS c, max(confidence) AS conf
                FROM quarantine.org_candidates GROUP BY kind) z), '{}'::jsonb),
    'promoted', (SELECT count(*) FROM quarantine.org_candidates
                  WHERE promoted_organization_id IS NOT NULL),
    'reviewed', (SELECT count(*) FROM quarantine.org_candidates WHERE reviewed_at IS NOT NULL),
    'already_an_organisation', (SELECT count(*) FROM quarantine.org_candidates
                                WHERE existing_organization_id IS NOT NULL),
    'biggest_districts', coalesce((SELECT jsonb_agg(jsonb_build_object(
          'domain', domain, 'name', company_name,
          'contacts', people_count, 'schools_on_it', name_variants) ORDER BY people_count DESC)
        FROM (SELECT * FROM quarantine.org_candidates
               WHERE kind = 'district' ORDER BY people_count DESC LIMIT 10) d), '[]'::jsonb),
    'door_out',
      'None of these become organisations yet. Linking an exact domain match needs no judgement and already happens. Creating tens of thousands of organisations at confidence 0.850 is a threshold, and there is no record setting it, so it is a decision, not a default.')
    INTO out;
  RETURN out;
END 
```

### org_classify() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 9 lines
```sql

DECLARE me uuid; who text;
BEGIN
  me := auth_staff_id();
  IF me IS NULL AND coalesce(auth.role(), '') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  who := CASE WHEN me IS NULL THEN 'agent:org-classifier' ELSE me::text END;
  RETURN org_classify_body(who);
END 
```

### org_classify_body(p_by text) -> jsonb
- sql | invoker | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 121 lines
```sql

  WITH due AS (
    SELECT EXISTS (
      SELECT 1 FROM quarantine.research_contacts rc
       WHERE rc.organization_id IS NULL
         AND rc.promoted_person_id IS NULL
         AND rc.landed_at > coalesce(
               (SELECT max(c.classified_at) FROM quarantine.org_candidates c),
               '-infinity'::timestamptz)
    ) AS work
  ),
  lim AS (SELECT coalesce(agent_setting('group_domain_min_schools'), 11)::int AS min_schools),
  unlinked AS (
    SELECT lower(split_part(rc.email_norm,'@',2)) AS dom,
           btrim(rc.school_name_raw) AS nm,
-- ... truncated (121 lines total)
```

### org_group_assign(p_group_id uuid, p_staff_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 20 lines
```sql

DECLARE me uuid; nm text;
BEGIN
  me := auth_staff_id();
  IF me IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')) THEN
    RAISE EXCEPTION 'Only an owner, admin or manager may assign a group' USING ERRCODE='42501'; END IF;
  IF p_staff_id IS NOT NULL AND NOT EXISTS (SELECT 1 FROM staff WHERE id = p_staff_id) THEN
    RAISE EXCEPTION 'No such staff member' USING ERRCODE='P0002'; END IF;

  UPDATE org_groups SET campaign_owner = p_staff_id, updated_at = now()
   WHERE id = p_group_id RETURNING name INTO nm;
  IF nm IS NULL THEN
    RAISE EXCEPTION 'No such group' USING ERRCODE='P0002'; END IF;

  RETURN jsonb_build_object('ok', true, 'group', nm,
    'owner', (SELECT full_name FROM staff WHERE id = p_staff_id),
    'effect', CASE WHEN p_staff_id IS NULL
      THEN 'Unassigned. Every school under this group is held again.'
      ELSE 'Assigned. Only they may approach this group; everybody else is held with their name.' END);
END 
```

### org_group_recount() -> integer
- plpgsql | invoker | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 20 lines
- comment: Re-sum schools_counted and contacts_counted across every domain a group holds. Called by the seeder, because a stored total that only a migration maintains is a stored total that drifts.
```sql

DECLARE n int;
BEGIN
  UPDATE public.org_groups g
     SET schools_counted  = t.schools,
         contacts_counted = t.contacts,
         updated_at       = now()
    FROM (
      SELECT d.group_id,
             sum(c.name_variants) AS schools,
             sum(c.people_count)  AS contacts
        FROM public.org_group_domains d
        JOIN quarantine.org_candidates c ON c.domain = d.domain
       GROUP BY d.group_id) t
   WHERE t.group_id = g.id
     AND (g.schools_counted IS DISTINCT FROM t.schools
          OR g.contacts_counted IS DISTINCT FROM t.contacts);
  GET DIAGNOSTICS n = ROW_COUNT;
  RETURN n;
END 
```

## Schema: public - part 2 of 4

### org_groups_list() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 30 lines
```sql

DECLARE out jsonb;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  SELECT jsonb_build_object(
    'groups', coalesce((SELECT jsonb_agg(jsonb_build_object(
        'id', g.id, 'name', g.name, 'kind', g.kind, 'domain', g.shared_domain,
        'name_is_placeholder', g.name_is_placeholder,
        'domains', (SELECT count(*) FROM org_group_domains d WHERE d.group_id = g.id),
        'schools', g.schools_counted, 'contacts', g.contacts_counted,
        'owner', st.full_name, 'owner_id', g.campaign_owner,
        'organisations_linked', (SELECT count(*) FROM organizations o WHERE o.group_id = g.id))
        ORDER BY g.schools_counted DESC NULLS LAST)
      FROM (SELECT * FROM org_groups ORDER BY schools_counted DESC NULLS LAST LIMIT 200) g
      LEFT JOIN staff st ON st.id = g.campaign_owner), '[]'::jsonb),
    'groups_total',  (SELECT count(*) FROM org_groups),
    'domains_total', (SELECT count(*) FROM org_group_domains),
    'unowned',       (SELECT count(*) FROM org_groups WHERE campaign_owner IS NULL),
    'need_a_name',   (SELECT count(*) FROM org_groups WHERE name_is_placeholder),
    'schools_held',  (SELECT coalesce(sum(schools_counted),0) FROM org_groups
                       WHERE campaign_owner IS NULL),
    'candidates_not_yet_grouped', (SELECT count(*) FROM quarantine.org_candidates c
                                    WHERE c.kind = 'district'
                                      AND NOT EXISTS (SELECT 1 FROM org_group_domains d
                                                       WHERE d.domain = c.domain)),
    'why_unowned_matters',
      'Every school under an unowned group is held. That is the point — an unowned group is the state that produced the complaint — but it means the count above is the number of relationships nobody can approach until somebody takes them.')
    INTO out;
  RETURN out;
END 
```

### org_groups_seed() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 15 lines
```sql

DECLARE me uuid; is_sched boolean; out jsonb; recounted int;
BEGIN
  me       := auth_staff_id();
  is_sched := session_user IN ('postgres','supabase_admin');
  IF me IS NULL AND NOT is_sched AND coalesce(auth.role(),'') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  IF me IS NOT NULL AND NOT (has_role('owner') OR has_role('admin') OR has_role('manager')) THEN
    RAISE EXCEPTION 'Only an owner, admin or manager may create groups' USING ERRCODE='42501'; END IF;

  out := org_groups_seed_from_districts(
           CASE WHEN me IS NULL THEN 'agent:org-classifier' ELSE me::text END);
  recounted := org_group_recount();
  RETURN out || jsonb_build_object('groups_recounted', recounted);
END 
```

### org_groups_seed_from_districts(p_by text) -> jsonb
- sql | invoker | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 66 lines
```sql

  WITH cand AS (
    SELECT c.domain, btrim(c.company_name) AS voted, c.name_variants, c.people_count,
           (public.org_name_shape(btrim(c.company_name)) = 'district') AS voted_is_usable,
           (SELECT g.id FROM org_groups g
             WHERE lower(btrim(g.name)) = lower(btrim(c.company_name)) LIMIT 1) AS same_name_group
      FROM quarantine.org_candidates c
     WHERE c.kind = 'district'
       AND nullif(btrim(coalesce(c.company_name,'')),'') IS NOT NULL
       AND NOT EXISTS (SELECT 1 FROM org_group_domains d WHERE d.domain = c.domain)
  ),
  attached AS (
    INSERT INTO org_group_domains (domain, group_id, added_by, note)
    SELECT domain, same_name_group, coalesce(p_by, 'agent:org-classifier'),
           'The same district name as this group, reached under another domain.'
-- ... truncated (66 lines total)
```

### org_name_shape(p_name text) -> text
- sql | invoker | immutable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 15 lines
- comment: The single answer to "is this the name of one school". Returns ok, district, generic or unusable. Called by the classifier, the promoter and the group seeder, so a district cannot be one thing to one of them and another to the next.
```sql

  SELECT CASE
    WHEN nullif(btrim(coalesce(p_name,'')),'') IS NULL              THEN 'unusable'
    WHEN btrim(p_name) !~ '[A-Za-z]'                                 THEN 'unusable'
    WHEN length(btrim(p_name)) < 4                                   THEN 'unusable'
    WHEN lower(btrim(p_name)) IN (
      'students','student','staff','school','schools','info','admin','administrator','office',
      'teachers','teacher','parents','parent','faculty','mail','email','contact','contacts',
      'n/a','na','none','null','unknown','test','user','users','principal','reception','library'
    )                                                                THEN 'generic'
    WHEN btrim(p_name) ~* '(district|board of education|school board|county schools|public schools|department of education|ministry of education|unified school|\yisd\y|\ycsd\y|\yusd\y|education authority|schools division)'
                                                                     THEN 'district'
    ELSE 'ok'
  END

```

### org_promote_candidates(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 88 lines
```sql

DECLARE
  me uuid; who text; is_sched boolean;
  floor_c numeric; lim int; src text;
  n_orgs int := 0; n_links int := 0; n_alias int := 0; sample jsonb;
  made_domains text[];
BEGIN
  me       := auth_staff_id();
  is_sched := session_user IN ('postgres', 'supabase_admin');
  IF me IS NULL AND NOT is_sched AND coalesce(auth.role(), '') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  IF me IS NOT NULL AND NOT (has_role('owner') OR has_role('admin') OR has_role('manager')) THEN
    RAISE EXCEPTION 'Only an owner, admin or manager may promote organisation candidates'
      USING ERRCODE='42501'; END IF;

-- ... truncated (88 lines total)
```

### org_unpromote(p_domain text, p_reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 34 lines
```sql

DECLARE me uuid; d text; org uuid; n_people int; n_unlinked int;
BEGIN
  me := auth_staff_id();
  IF me IS NULL OR NOT (has_role('owner') OR has_role('admin')) THEN
    RAISE EXCEPTION 'Only an owner or admin may undo a promotion' USING ERRCODE='42501'; END IF;
  IF nullif(btrim(coalesce(p_reason,'')),'') IS NULL THEN
    RAISE EXCEPTION 'Undoing needs a reason, the same as doing' USING ERRCODE='22023'; END IF;

  d := lower(btrim(coalesce(p_domain,'')));
  SELECT promoted_organization_id INTO org FROM quarantine.org_candidates WHERE domain = d;
  IF org IS NULL THEN
    RAISE EXCEPTION 'No promoted organisation for domain %', d USING ERRCODE='P0002'; END IF;

  SELECT count(*) INTO n_people FROM affiliations WHERE organization_id = org;
  IF n_people > 0 THEN
    RAISE EXCEPTION 'This organisation has % affiliated people. Undoing would remove a real link, so it is a merge decision rather than an undo', n_people
      USING ERRCODE='22023'; END IF;

  UPDATE quarantine.research_contacts SET organization_id = NULL
   WHERE organization_id = org AND promoted_person_id IS NULL;
  GET DIAGNOSTICS n_unlinked = ROW_COUNT;

  DELETE FROM org_aliases WHERE organization_id = org;
  UPDATE quarantine.org_candidates
     SET promoted_organization_id = NULL, promoted_at = NULL,
         review_note = coalesce(review_note || ' | ', '') || 'unpromoted: ' || btrim(p_reason),
         reviewed_by = me, reviewed_at = now()
   WHERE domain = d;
  DELETE FROM organizations WHERE id = org;

  RETURN jsonb_build_object('ok', true, 'domain', d, 'contacts_unlinked', n_unlinked,
    'organisation_deleted', true, 'reason', btrim(p_reason));
END 
```

### pending_access_requests() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 10 lines
```sql

BEGIN
  IF NOT (has_role('owner') OR has_role('admin')) THEN RETURN '[]'::jsonb; END IF;
  RETURN coalesce((SELECT jsonb_agg(jsonb_build_object(
             'staff_id', s.id, 'email', s.email, 'email_domain', split_part(s.email,'@',2),
             'full_name', s.full_name, 'job_title', s.job_title, 'requested_at', s.requested_at)
           ORDER BY s.requested_at)
      FROM staff s WHERE s.status='requested'), '[]'::jsonb);
END;

```

### people_growth(p_months integer) -> jsonb
- sql | invoker | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 46 lines
- comment: Monthly arrival buckets from people_v. Separates bulk-import volume from organic arrival and marks the current partial month. INVOKER, so scope applies. 029.
```sql

  WITH bounds AS (
    SELECT date_trunc('month', current_date)::date AS this_month,
           (date_trunc('month', current_date) - ((greatest(p_months, 1) - 1) * interval '1 month'))::date AS first_month
  ), mine AS (
    SELECT date_trunc('month', p.created_at)::date AS month,
           date_trunc('day',   p.created_at)::date AS day
      FROM people_v p, bounds b
     WHERE p.created_at IS NOT NULL AND p.created_at >= b.first_month
  ), per_day AS (
    SELECT month, day, count(*) AS n FROM mine GROUP BY 1, 2
  ), thresholds AS (
    SELECT month, 5 * percentile_cont(0.5) WITHIN GROUP (ORDER BY n) AS bulk_over
      FROM per_day GROUP BY month
  ), per_month AS (
-- ... truncated (46 lines total)
```

### person_360(p_person_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 133 lines
- comment: Everything about one person in one call. Masking is 047's line: pii_contact is who they are, comms_read is what was said, the assessments are visible to any staff. Scope check is people_v's, reproduced because this is DEFINER.
```sql

DECLARE p people%ROWTYPE; see_pii boolean; see_comms boolean;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;

  SELECT * INTO p FROM people WHERE id = p_person_id;
  IF p.id IS NULL THEN RAISE EXCEPTION 'No such person' USING ERRCODE='P0002'; END IF;

  IF NOT (has_scope_all() OR can_see_person(p_person_id)) THEN
    RAISE EXCEPTION 'No such person' USING ERRCODE='P0002'; END IF;

  see_pii   := CASE WHEN p.is_minor THEN has_grant('pii_minor') ELSE has_grant('pii_contact') END;
  see_comms := has_grant('comms_read');

  RETURN jsonb_build_object(
-- ... truncated (133 lines total)
```

### person_360_timeline(p_person_id uuid, p_comms boolean, p_pii boolean) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public, pg_temp | exec anon=n authenticated=n | 44 lines
```sql

  SELECT coalesce(jsonb_agg(t ORDER BY t->>'at' DESC), '[]'::jsonb) FROM (

    SELECT jsonb_build_object('at', a.occurred_at, 'stream', 'activity', 'kind', a.kind,
             'channel', a.channel, 'actor', a.actor, 'title', a.payload->>'title',
             'ref', jsonb_build_object('table','activities','id',a.id)) AS t
      FROM activities a WHERE a.person_id = p_person_id

    UNION ALL
    SELECT jsonb_build_object('at', mt.starts_at, 'stream', 'meeting',
             'kind', 'meeting_' || mt.status::text, 'channel', NULL, 'actor', mt.source::text,
             'title', CASE WHEN p_comms THEN left(coalesce(mt.notes, mt.ai_summary, 'Meeting'), 160)
                           ELSE 'Meeting' END,
             'ref', jsonb_build_object('table','meetings','id',mt.id))
      FROM meetings mt WHERE mt.person_id = p_person_id
-- ... truncated (44 lines total)
```

### postgres_fdw_disconnect(text) -> boolean
- c | invoker | volatile | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
postgres_fdw_disconnect
```

### postgres_fdw_disconnect_all() -> boolean
- c | invoker | volatile | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
postgres_fdw_disconnect_all
```

### postgres_fdw_get_connections(OUT server_name text, OUT valid boolean) -> SETOF record
- c | invoker | volatile | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
postgres_fdw_get_connections
```

### postgres_fdw_handler() -> fdw_handler
- c | invoker | volatile | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
postgres_fdw_handler
```

### postgres_fdw_validator(text[], oid) -> void
- c | invoker | volatile | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
postgres_fdw_validator
```

### programme_for_pitch(p_raw text) -> uuid
- sql | invoker | stable | config: search_path=public, pg_temp | exec anon=Y authenticated=Y | 7 lines
```sql

  SELECT coalesce(
    (SELECT p.id FROM programmes p
      WHERE programme_name_key(p.name) = programme_name_key(p_raw) LIMIT 1),
    (SELECT a.programme_id FROM programme_aliases a
      WHERE a.name_key = programme_name_key(p_raw) AND a.kind='programme' LIMIT 1));

```

### programme_mix() -> jsonb
- sql | invoker | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 22 lines
- comment: Programmes per category with the active count and cohort count. Labels resolved here so every caller renders the same words. 029.
```sql

  SELECT jsonb_build_object(
    'categories', coalesce((
      SELECT jsonb_agg(jsonb_build_object(
               'category', x.category,
               'label',    initcap(replace(x.category, '_', ' ')),
               'total',    x.total,
               'active',   x.active,
               'cohorts',  x.cohorts)
             ORDER BY x.total DESC, x.category)
        FROM (SELECT pr.category::text AS category,
                     count(*) AS total,
                     count(*) FILTER (WHERE pr.is_active) AS active,
                     (SELECT count(*) FROM cohorts c
                       WHERE c.programme_id IN (SELECT id FROM programmes p2 WHERE p2.category = pr.category)) AS cohorts
                FROM programmes pr GROUP BY pr.category) x), '[]'::jsonb),
    'totals', jsonb_build_object(
      'programmes', (SELECT count(*) FROM programmes),
      'active',     (SELECT count(*) FROM programmes WHERE is_active),
      'cohorts',    (SELECT count(*) FROM cohorts))
  );

```

### programme_name_key(p_name text) -> text
- sql | invoker | immutable | config: search_path=pg_catalog, pg_temp | exec anon=Y authenticated=Y | 5 lines
```sql

  SELECT nullif(btrim(regexp_replace(
           regexp_replace(lower(coalesce(p_name,'')), '[^a-z0-9]+', ' ', 'g'),
           '\s+', ' ', 'g')), '');

```

### promote_research_contact(p_contact_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 84 lines
```sql

DECLARE
  rc quarantine.research_contacts%ROWTYPE;
  v_person uuid; v_hits int; v_src text; v_new boolean := false;
  v_send jsonb; v_verdict text;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may promote a researched contact' USING ERRCODE='42501'; END IF;

  SELECT * INTO rc FROM quarantine.research_contacts WHERE id = p_contact_id;
  IF rc.id IS NULL THEN RAISE EXCEPTION 'No such research contact %', p_contact_id USING ERRCODE='P0002'; END IF;
  IF rc.promoted_person_id IS NOT NULL THEN
    RETURN jsonb_build_object('ok',true,'person_id',rc.promoted_person_id,'created',false,'already_promoted',true); END IF;

-- ... truncated (84 lines total)
```

### promote_research_contacts(p_country text, p_state text, p_city text, p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 31 lines
```sql

DECLARE r record; created int:=0; linked int:=0; refused int:=0; reasons jsonb:='[]'::jsonb; res jsonb;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach')) THEN
    RAISE EXCEPTION 'Only an owner, manager or outreach may promote researched contacts in bulk' USING ERRCODE='42501'; END IF;

  -- California-first as a constraint, not a note: leaving the country out would promote every row.
  IF nullif(btrim(coalesce(p_country,'')),'') IS NULL THEN
    RAISE EXCEPTION 'A country is required. This runs one location at a time on purpose - the trial is one state first, and an unscoped call would promote every researched contact in the table'
      USING ERRCODE='22023'; END IF;

  FOR r IN SELECT rc.id FROM quarantine.research_contacts rc
            WHERE rc.is_valid IS TRUE AND rc.promoted_person_id IS NULL
              AND lower(btrim(rc.country_raw)) = lower(btrim(p_country))
              AND (p_state IS NULL OR lower(btrim(rc.state_raw)) = lower(btrim(p_state)))
              AND (p_city  IS NULL OR lower(btrim(rc.city_raw))  = lower(btrim(p_city)))
            ORDER BY rc.verified_at NULLS LAST
            LIMIT least(greatest(coalesce(p_limit,200),1),1000) LOOP
    BEGIN
      res := promote_research_contact(r.id);
      IF (res->>'created')::boolean THEN created := created+1; ELSE linked := linked+1; END IF;
    EXCEPTION WHEN others THEN
      refused := refused+1;
      reasons := reasons || jsonb_build_object('contact_id',r.id,'reason',left(SQLERRM,160));
    END;
  END LOOP;

  RETURN jsonb_build_object('scope', jsonb_build_object('country',p_country,'state',p_state,'city',p_city),
    'people_created',created,'linked_to_existing',linked,'refused',refused,'reasons',reasons);
END 
```

### promote_research_school(p_name_key text, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 25 lines
```sql

DECLARE me uuid := public.auth_staff_id(); rs quarantine.research_schools; req jsonb; out_ jsonb;
BEGIN
  IF me IS NULL OR NOT EXISTS (SELECT 1 FROM public.staff_role_assignments
                                WHERE staff_id = me AND role IN ('owner','admin')) THEN
    RAISE EXCEPTION 'Only an owner or admin may promote a school';
  END IF;
  SELECT * INTO rs FROM quarantine.research_schools WHERE name_key = lower(btrim(p_name_key));
  IF rs.name_key IS NULL THEN RAISE EXCEPTION 'No such research school'; END IF;
  IF rs.organization_id IS NOT NULL THEN
    RETURN jsonb_build_object('ok', true, 'already_promoted', true,
                              'organization_id', rs.organization_id);
  END IF;
  IF rs.is_provider THEN RAISE EXCEPTION '% is %, not a school', rs.name, rs.triage_reason; END IF;

  req := public.school_request(rs.name, rs.country_raw, rs.city_raw, NULL, p_note);
  IF (req->>'already_exists')::boolean THEN
    UPDATE quarantine.research_schools SET organization_id = (req->>'organization_id')::uuid
     WHERE name_key = rs.name_key;
    RETURN jsonb_build_object('ok', true, 'already_promoted', true,
                              'organization_id', (req->>'organization_id')::uuid);
  END IF;
  out_ := public.promote_school((req->>'candidate_id')::uuid, p_note);
  RETURN out_ || jsonb_build_object('name', rs.name, 'contacts', rs.contacts);
END 
```

### promote_school(p_candidate_id uuid, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 43 lines
- comment: Owner/admin promotes a reviewed candidate into organizations, writes its alias, and links the research rows that named it. Idempotent. Resolves no people. 036.
```sql

DECLARE me uuid := public.auth_staff_id(); cand quarantine.org_candidates; oid uuid; n_rc integer := 0;
BEGIN
  IF me IS NULL OR NOT EXISTS (SELECT 1 FROM public.staff_role_assignments
                                WHERE staff_id = me AND role IN ('owner','admin')) THEN
    RAISE EXCEPTION 'Only an owner or admin may promote a school into the registry';
  END IF;
  SELECT * INTO cand FROM quarantine.org_candidates WHERE id = p_candidate_id;
  IF cand.id IS NULL THEN RAISE EXCEPTION 'No such candidate'; END IF;
  IF cand.promoted_organization_id IS NOT NULL THEN
    RETURN jsonb_build_object('ok', true, 'already_promoted', true,
                              'organization_id', cand.promoted_organization_id);
  END IF;
  IF cand.existing_organization_id IS NOT NULL THEN
    RAISE EXCEPTION 'This candidate points at an existing organisation; merge it rather than promoting';
-- ... truncated (43 lines total)
```

### provider_tick() -> bigint
- plpgsql | invoker | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=n | 6 lines
- comment: Reads provider_batch from agent_settings and fires one provider-verification batch at the edge function. Called only by the spine_validator_provider cron job — SECURITY INVOKER so only postgres (via pg_cron) can run it. Raise provider_batch via agent_setting_set when a campaign slice needs verifying faster than the daily default.
```sql

DECLARE batch int;
BEGIN
  batch := coalesce(agent_setting('provider_batch')::int, 200);
  RETURN validator_tick('provider', batch);
END 
```

### rb_apply_ai_reply_classification(p_handoff_id uuid, p_reply_class text, p_confidence numeric, p_reason text, p_next_action text, p_priority text, p_model text, p_ai_response jsonb) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 5 lines
```sql

  select revenue_brain.apply_ai_reply_classification(
    p_handoff_id,p_reply_class,p_confidence,p_reason,p_next_action,p_priority,p_model,p_ai_response
  );

```

### rb_approve_pilot_preflight_case(p_case_id uuid, p_decision text, p_approved_by text, p_notes text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.approve_pilot_preflight_case(p_case_id,p_decision,p_approved_by,p_notes); 
```

### rb_approve_programme_batch(p_batch_id uuid, p_approved_by text, p_reject_emails text[]) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.approve_programme_batch(p_batch_id, p_approved_by, p_reject_emails); 
```

### rb_arm_wow_pilot_launch(p_approved_by text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.arm_wow_pilot_launch(p_approved_by); 
```

### rb_authorize_wow_real_send(p_approved_by text, p_confirmation text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.authorize_wow_real_send(p_approved_by,p_confirmation); 
```

### rb_bind_yale_test_campaign(p_run_id uuid, p_campaign_id text, p_campaign_name text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.bind_yale_test_campaign(p_run_id,p_campaign_id,p_campaign_name); 
```

### rb_claim_handoff_notification(p_handoff_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 3 lines
```sql

  select revenue_brain.claim_handoff_notification(p_handoff_id);

```

### rb_claim_wow_pilot_dispatch() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.claim_wow_pilot_dispatch(); 
```

### rb_create_autonomous_test_run() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.create_autonomous_test_run(); 
```

### rb_create_conversion_test_run() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.create_conversion_test_run(); 
```

### rb_create_pilot_copy_review() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.create_pilot_copy_review(); 
```

### rb_create_pilot_preflight_run() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.create_pilot_preflight_run(); 
```

### rb_create_programme_batch(p_programme_code text, p_size integer, p_region text, p_include_needs_zerobounce boolean, p_by text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.create_programme_batch(p_programme_code, p_size, p_region, p_include_needs_zerobounce, p_by); 
```

### rb_create_safety_test_run() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.create_safety_test_run(); 
```

### rb_create_shadow_run() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.create_shadow_run(); 
```

### rb_create_wow_internal_cert_run() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.create_wow_internal_cert_run(); 
```

### rb_create_yale_test_run() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.create_yale_test_run(); 
```

### rb_evaluate_final_approval(p_queue_id uuid, p_route_key text, p_requested_at timestamp with time zone) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 3 lines
```sql

  select revenue_brain.evaluate_final_approval(p_queue_id,p_route_key,p_requested_at);

```

### rb_finalize_autonomous_test_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.finalize_autonomous_test_run(p_run_id); 
```

### rb_finalize_conversion_test_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.finalize_conversion_test_run(p_run_id); 
```

### rb_finalize_pilot_preflight_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.finalize_pilot_preflight_run(p_run_id); 
```

### rb_finalize_safety_test_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.finalize_safety_test_run(p_run_id); 
```

### rb_finalize_shadow_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.finalize_shadow_run(p_run_id); 
```

### rb_get_all_polling_campaign_routes() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_all_polling_campaign_routes(); 
```

### rb_get_autonomous_case_send_status(p_run_id uuid, p_case_key text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 3 lines
```sql

  select revenue_brain.get_autonomous_case_send_status(p_run_id,p_case_key);

```

### rb_get_autonomous_test_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_autonomous_test_run(p_run_id); 
```

### rb_get_conversion_test_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_conversion_test_run(p_run_id); 
```

### rb_get_pilot_copy_review(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_pilot_copy_review(p_run_id); 
```

### rb_get_pilot_preflight_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_pilot_preflight_run(p_run_id); 
```

### rb_get_polling_campaign_routes() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_polling_campaign_routes(); 
```

### rb_get_reply_handoff(p_handoff_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 3 lines
```sql

  select revenue_brain.get_reply_handoff(p_handoff_id);

```

### rb_get_safety_test_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_safety_test_run(p_run_id); 
```

### rb_get_shadow_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_shadow_run(p_run_id); 
```

### rb_get_wow_internal_cert_status(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_wow_internal_cert_status(p_run_id); 
```

### rb_get_wow_pilot_bundle() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_wow_pilot_bundle(); 
```

### rb_get_wow_pilot_controller_state() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_wow_pilot_controller_state(); 
```

### rb_get_yale_sequence() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_yale_sequence(); 
```

### rb_get_yale_sequence_by_code(p_sequence_code text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.get_yale_sequence_by_code(p_sequence_code); 
```

### rb_ingest_instantly_webhook(p_payload jsonb) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 3 lines
```sql

  select revenue_brain.ingest_instantly_webhook(p_payload);

```

### rb_kill_wow_pilot(p_requested_by text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.kill_wow_pilot(p_requested_by); 
```

### rb_list_open_reply_handoffs(p_owner text, p_limit integer) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 27 lines
```sql

  select coalesce(jsonb_agg(to_jsonb(x)),'[]'::jsonb)
  from (
    select
      h.id,h.school_id,h.school_name,h.contact_name,h.contact_email,h.programme_code,
      h.reply_class,h.classification_confidence,h.priority,h.owner,h.next_action,
      h.due_at,h.reply_subject,h.reply_snippet,h.unibox_url,h.created_at,h.is_test,
      h.notification_status,h.notification_attempts
    from revenue_brain.reply_handoffs h
    where h.handoff_status in ('NEW','IN_PROGRESS')
      and (p_owner is null or h.owner=p_owner)
      and (
        h.notification_status='PENDING'
        or (h.notification_status='FAILED' and coalesce(h.notification_attempts,0) < 3)
        or (
          h.notification_status='PROCESSING'
          and h.updated_at < now()-interval '10 minutes'
          and coalesce(h.notification_attempts,0) < 3
        )
      )
    order by
      case h.priority when 'CRITICAL' then 1 when 'HIGH' then 2 when 'MEDIUM' then 3 else 4 end,
      h.due_at nulls last,
      h.created_at
    limit greatest(1,least(coalesce(p_limit,100),500))
  ) x;

```

### rb_mark_ai_reply_failure(p_handoff_id uuid, p_model text, p_error_text text, p_ai_response jsonb) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 5 lines
```sql

  select revenue_brain.mark_ai_reply_failure(
    p_handoff_id,p_model,p_error_text,p_ai_response
  );

```

### rb_mark_handoff_notification_result(p_handoff_id uuid, p_success boolean, p_channel text, p_recipients text, p_error_text text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 5 lines
```sql

  select revenue_brain.mark_handoff_notification_result(
    p_handoff_id,p_success,p_channel,p_recipients,p_error_text
  );

```

### rb_prepare_dispatch(p_queue_id uuid, p_route_key text, p_requested_at timestamp with time zone) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 3 lines
```sql

  select revenue_brain.prepare_dispatch(p_queue_id, p_route_key, p_requested_at);

```

### rb_programme_pipeline_summary(p_programme_code text, p_region text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.programme_pipeline_summary(p_programme_code, p_region); 
```

### rb_programme_readiness() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select coalesce(jsonb_agg(to_jsonb(r) order by r.outreach_enabled desc, r.priority, r.display_name), '[]'::jsonb) from revenue_brain.programme_readiness r; 
```

### rb_record_autonomous_test_enrollment(p_case_id uuid, p_provider_lead_id text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.record_autonomous_test_enrollment(p_case_id,p_provider_lead_id); 
```

### rb_record_conversion_test_event(p_run_id uuid, p_case_key text, p_event_type text, p_payload jsonb) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.record_conversion_test_event(p_run_id,p_case_key,p_event_type,p_payload); 
```

### rb_record_dispatch_result(p_run_id uuid, p_success boolean, p_provider_lead_id text, p_response jsonb, p_error_text text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 5 lines
```sql

  select revenue_brain.record_dispatch_result(
    p_run_id, p_success, p_provider_lead_id, p_response, p_error_text
  );

```

### rb_record_pilot_provider_snapshot(p_run_id uuid, p_snapshot jsonb) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.record_pilot_provider_snapshot(p_run_id,p_snapshot); 
```

### rb_record_shadow_ai_decision(p_run_id uuid, p_case_key text, p_ai_decision jsonb) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.record_shadow_ai_decision(p_run_id,p_case_key,p_ai_decision); 
```

### rb_record_wow_pilot_dispatch(p_case_release_id uuid, p_success boolean, p_provider_lead_id text, p_response jsonb, p_error_text text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 3 lines
```sql
 select revenue_brain.record_wow_pilot_dispatch(
  p_case_release_id,p_success,p_provider_lead_id,p_response,p_error_text
); 
```

### rb_resolve_reply_handoff(p_handoff_id uuid, p_resolved_by text, p_resolution text, p_human_classification text, p_unlock_school boolean, p_next_eligible_at timestamp with time zone) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 5 lines
```sql

  select revenue_brain.resolve_reply_handoff(
    p_handoff_id,p_resolved_by,p_resolution,p_human_classification,p_unlock_school,p_next_eligible_at
  );

```

### rb_revoke_wow_real_send(p_requested_by text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.revoke_wow_real_send(p_requested_by); 
```

### rb_set_autonomous_test_campaign(p_run_id uuid, p_provider_campaign_id text, p_campaign_name text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 5 lines
```sql

  select revenue_brain.set_autonomous_test_campaign(
    p_run_id,p_provider_campaign_id,p_campaign_name
  );

```

### rb_set_programme_outreach(p_programme_code text, p_enabled boolean, p_priority integer, p_by text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.set_programme_outreach(p_programme_code, p_enabled, p_priority, p_by); 
```

### rb_set_wow_case_approval(p_preflight_case_id uuid, p_decision text, p_approved_by text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.set_wow_case_approval(p_preflight_case_id,p_decision,p_approved_by); 
```

### rb_set_wow_copy_approval(p_decision text, p_approved_by text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.set_wow_copy_approval(p_decision,p_approved_by); 
```

### rb_stage_autonomous_test_decision(p_case_id uuid, p_ai_decision jsonb) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.stage_autonomous_test_decision(p_case_id,p_ai_decision); 
```

### rb_stage_programme_batch(p_batch_id uuid, p_by text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.stage_programme_batch(p_batch_id, p_by); 
```

### rb_yale_bind_route(p_route_key text, p_campaign_id text, p_campaign_name text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.yale_bind_route(p_route_key, p_campaign_id, p_campaign_name); 
```

### rb_yale_dispatch_route(p_route_key text, p_limit integer) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.yale_pilot_dispatch_batch(p_limit, p_route_key); 
```

### rb_yale_pilot_bind_campaign(p_campaign_id text, p_campaign_name text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.yale_pilot_bind_campaign(p_campaign_id, p_campaign_name); 
```

### rb_yale_pilot_dispatch_batch(p_limit integer) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.yale_pilot_dispatch_batch(p_limit, 'PILOT_US'); 
```

### rb_yale_pilot_record(p_n integer, p_success boolean, p_provider_lead_id text, p_response jsonb, p_error text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=public, revenue_brain | exec anon=n authenticated=n | 1 lines
```sql
 select revenue_brain.yale_pilot_record(p_n, p_success, p_provider_lead_id, p_response, p_error); 
```

### record_source_arrival() -> integer
- sql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=n | 17 lines
- comment: Stamps source_systems.connected_at from observed people.source_code. Idempotent, never clears. 028.
```sql

  WITH observed AS (
    SELECT split_part(source_code, '-', 2) AS key, min(created_at) AS first_at
      FROM public.people
     WHERE source_code IS NOT NULL
     GROUP BY 1
  ), stamped AS (
    UPDATE public.source_systems s
       SET connected_at = o.first_at
      FROM observed o
     WHERE o.key = s.key
       AND s.connected_at IS NULL
       AND s.is_expected
     RETURNING 1
  )
  SELECT count(*)::integer FROM stamped;

```

### refund_not_before_payment() -> trigger
- plpgsql | invoker | volatile | config: search_path=public | exec anon=n authenticated=n | 8 lines
```sql

BEGIN
  IF NEW.occurred_at IS NOT NULL
     AND NEW.occurred_at < (SELECT p.occurred_at FROM payments p WHERE p.id = NEW.payment_id) THEN
    RAISE EXCEPTION 'refund % predates its payment', NEW.id;
  END IF;
  RETURN NEW;
END; 
```

### refund_within_payment() -> trigger
- plpgsql | invoker | volatile | config: search_path=public | exec anon=n authenticated=n | 14 lines
```sql

DECLARE paid_minor bigint; paid_ccy text; already bigint;
BEGIN
  SELECT p.amount_minor, p.currency INTO paid_minor, paid_ccy FROM payments p WHERE p.id = NEW.payment_id;
  IF NEW.currency <> paid_ccy THEN
    RAISE EXCEPTION 'refund currency % does not match payment currency %', NEW.currency, paid_ccy;
  END IF;
  SELECT coalesce(sum(r.amount_minor),0) INTO already FROM refunds r
   WHERE r.payment_id = NEW.payment_id AND r.id <> NEW.id;
  IF already + NEW.amount_minor > paid_minor THEN
    RAISE EXCEPTION 'refunds (% + %) would exceed payment of %', already, NEW.amount_minor, paid_minor;
  END IF;
  RETURN NEW;
END; 
```

### reject_access_request(p_staff_id uuid, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=Y | 15 lines
```sql

DECLARE v_me uuid := auth_staff_id(); v_status staff_status;
BEGIN
  IF v_me IS NULL OR NOT (has_role('owner') OR has_role('admin')) THEN
    RAISE EXCEPTION 'only an owner or admin may reject access requests'; END IF;
  IF nullif(btrim(coalesce(p_note,'')),'') IS NULL THEN
    RAISE EXCEPTION 'a reason is required to reject a request'; END IF;
  SELECT status INTO v_status FROM staff WHERE id = p_staff_id FOR UPDATE;
  IF NOT FOUND THEN RAISE EXCEPTION 'no such staff row: %', p_staff_id; END IF;
  IF v_status <> 'requested' THEN RAISE EXCEPTION 'staff % is %, not requested', p_staff_id, v_status; END IF;
  UPDATE staff SET status='rejected', decided_by=v_me, decided_at=now(),
         decision_note=p_note, updated_at=now() WHERE id = p_staff_id;
  RETURN jsonb_build_object('rejected', p_staff_id, 'by', v_me);
END;

```

### reject_inbound(p_id uuid, p_reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 32 lines
- comment: Dismiss a queued inbound record with a written reason. Owner or admin. Never deletes. 032.
```sql

DECLARE
  me  uuid := public.auth_staff_id();
  rec public.inbound_records;
BEGIN
  IF me IS NULL OR NOT EXISTS (
    SELECT 1 FROM public.staff_role_assignments
     WHERE staff_id = me AND role IN ('owner', 'admin')
  ) THEN
    RAISE EXCEPTION 'Only an owner or admin may reject an inbound record';
  END IF;

  IF btrim(coalesce(p_reason, '')) = '' THEN
    RAISE EXCEPTION 'A written reason is required';
  END IF;

  SELECT * INTO rec FROM public.inbound_records WHERE id = p_id;
  IF rec.id IS NULL THEN
    RAISE EXCEPTION 'No such inbound record';
  END IF;
  IF rec.processed_at IS NOT NULL THEN
    RAISE EXCEPTION 'That record was already resolved into a person; rejecting it now would be a lie about history';
  END IF;

  UPDATE public.inbound_records
     SET rejected_at = now(),
         rejected_reason = btrim(p_reason) || ' — ' ||
           coalesce((SELECT full_name FROM public.staff WHERE id = me), 'unknown staff')
   WHERE id = p_id AND rejected_at IS NULL;

  RETURN jsonb_build_object('ok', true, 'id', p_id);
END 
```

### reject_research_school(p_name_key text, p_reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 26 lines
```sql

DECLARE me uuid := public.auth_staff_id(); n int;
BEGIN
  IF me IS NULL OR NOT EXISTS (SELECT 1 FROM public.staff_role_assignments
                                WHERE staff_id = me AND role IN ('owner','admin')) THEN
    RAISE EXCEPTION 'Only an owner or admin may reject a research school';
  END IF;
  -- A reason is required for the same reason it is on every other rejection in this schema:
  -- "not a school" in three months tells nobody why.
  IF btrim(coalesce(p_reason, '')) = '' THEN
    RAISE EXCEPTION 'A written reason is required';
  END IF;

  UPDATE quarantine.research_schools
     SET rejected_at = now(), rejected_by = me, reject_reason = btrim(p_reason)
   WHERE name_key = lower(btrim(p_name_key))
     AND organization_id IS NULL      -- already promoted: rejecting it now would be a lie
     AND rejected_at IS NULL;
  GET DIAGNOSTICS n = ROW_COUNT;

  IF n = 0 THEN
    RETURN jsonb_build_object('ok', false,
      'error', 'not found, already promoted, or already rejected');
  END IF;
  RETURN jsonb_build_object('ok', true, 'name_key', lower(btrim(p_name_key)));
END 
```

### research_add(p_email text, p_school text, p_contact text, p_domain text, p_country text, p_state text, p_city text, p_phone text, p_designation text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 41 lines
```sql

DECLARE chk jsonb; v_id uuid; ek text; me uuid; is_researcher boolean;
BEGIN
  me := auth_staff_id();
  IF me IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach')
             OR has_role('counsellor') OR has_role('researcher')) THEN
    RAISE EXCEPTION 'You do not have a role that may add research contacts' USING ERRCODE='42501'; END IF;

  is_researcher := has_role('researcher')
                   AND NOT (has_role('owner') OR has_role('admin') OR has_role('manager'));
  IF is_researcher AND NOT allocation_covers(me, p_country, p_state, p_city) THEN
    RAISE EXCEPTION 'That place is not allocated to you. Ask an owner or manager to allocate %, or add rows for the place you hold',
      coalesce(nullif(btrim(coalesce(p_country,'')),''), 'nowhere')
      USING ERRCODE='42501'; END IF;
-- ... truncated (41 lines total)
```

### research_agent_spend_today() -> numeric
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 3 lines
```sql

  select coalesce(sum(cost_usd), 0) from public.research_tasks where started_at >= date_trunc('day', now());

```

### research_ai_check() -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 21 lines
```sql

  with a as (
    select c.id, c.email, c.contact_name, c.designation, c.school, c.website, c.country, c.source_file, c.research_tool,
           coalesce(nullif(array_to_string(c.researcher_names, ', '), ''), 'Not recorded') researcher, h.tier, h.reasons, h.email_status
      from research_contacts c
      join research_source_tools st on st.key = c.research_tool and st.needs_verification
      left join research_contact_health h on h.contact_id = c.id
     where c.deleted_at is null)
  select jsonb_build_object(
    'groups', (select coalesce(jsonb_agg(g order by g.rows_n desc), '[]') from (
      select source_file sheet, researcher, research_tool tool, count(*) rows_n,
             count(*) filter (where not ('ai_sourced_unverified' = any(coalesce(reasons, '{}')))) proven,
             count(*) filter (where 'ai_email_domain_not_school' = any(coalesce(reasons, '{}'))) guessed_domain,
             count(*) filter (where email_status = 'do_not_send') bad_email,
             count(*) filter (where email_status is null) email_unchecked
        from a group by 1, 2, 3) g),
    'rows', (select coalesce(jsonb_agg(r), '[]') from (
      select id, contact_name, designation, school, website, email, country, source_file, researcher, research_tool, tier, reasons, email_status
        from a where 'ai_sourced_unverified' = any(coalesce(reasons, '{}')) or 'ai_email_domain_not_school' = any(coalesce(reasons, '{}'))
        order by source_file, school limit 5000) r));

```

### research_allocation_progress(p_researcher uuid) -> TABLE(allocation_id uuid, today_count bigint, week_count bigint, done_rows bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 23 lines
```sql

  with a as (
    select al.id,
           al.researcher_id,
           nullif(lo.state, '') as state,
           coalesce(al.started_on, al.created_at::date) as started_on
      from public.research_allocations al
      left join public.research_locations lo on lo.id = al.location_id
     where p_researcher is null or al.researcher_id = p_researcher
  )
  select
    a.id,
    count(*) filter (where c.created_at >= date_trunc('day', now())),
    count(*) filter (where c.created_at >= date_trunc('day', now()) - interval '6 days'),
    count(*) filter (where c.created_at::date >= a.started_on)
  from a
  left join public.research_contacts c
    on c.created_by = a.researcher_id
   and c.deleted_at is null
   and c.person_type = 'staff'
   and (a.state is null or c.state = a.state)
  group by a.id;

```

### research_alumni_facets(p_top integer) -> TABLE(facet text, label text, n bigint, distinct_labels bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 22 lines
```sql

  with live as (select * from public.research_alumni where deleted_at is null),
  raw as (
    select 'year' as facet, coalesce(program_year::text, 'Not set') as label, count(*) as n from live group by 2
    union all select 'program', coalesce(nullif(trim(program_name), ''), 'Not set'), count(*) from live group by 2
    union all select 'program_type', coalesce(nullif(program_type, ''), 'Not set'), count(*) from live group by 2
    union all select 'payment', coalesce(nullif(payment_status, ''), 'Not recorded'), count(*) from live group by 2
    union all select 'grade', coalesce(nullif(grade, ''), 'Not set'), count(*) from live group by 2
    union all select 'state', coalesce(nullif(trim(state), ''), 'Not set'), count(*) from live group by 2
    union all select 'school', coalesce(nullif(trim(school_name), ''), 'Not set'), count(*) from live group by 2
    union all select 'reach',
      case when school_contact_id is not null then 'School we already hold' else 'School not in the database' end,
      count(*) from live group by 2
  ),
  ranked as (
    select facet, label, n,
           row_number() over (partition by facet order by n desc, label) as rn,
           count(*) over (partition by facet) as distinct_labels
    from raw
  )
  select facet, label, n, distinct_labels from ranked where rn <= p_top order by facet, n desc;

```

### research_alumni_schools() -> TABLE(school_key text, school_name text, students bigint, participations bigint, paid bigint, last_year integer, programs text, we_hold_staff boolean)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 14 lines
```sql

  select
    lower(school_name), min(school_name),
    count(distinct coalesce(nullif(lower(student_email), ''), lower(student_name))),
    count(*),
    count(*) filter (where payment_status = 'Paid'),
    max(program_year),
    string_agg(distinct nullif(trim(program_name), ''), ', ' order by nullif(trim(program_name), '')),
    bool_or(school_contact_id is not null)
  from public.research_alumni
  where deleted_at is null and school_name <> ''
  group by lower(school_name)
  order by count(*) desc;

```

## Schema: public - part 3 of 4

### research_alumni_summary() -> TABLE(participations bigint, students bigint, schools bigint, schools_we_hold bigint, paid bigint, repeat_students bigint, first_year integer, last_year integer)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 12 lines
```sql

  with live as (select * from public.research_alumni where deleted_at is null)
  select
    (select count(*) from live),
    (select count(distinct coalesce(nullif(lower(student_email), ''), lower(student_name))) from live),
    (select count(distinct lower(school_name)) from live where school_name <> ''),
    (select count(distinct lower(school_name)) from live where school_contact_id is not null),
    (select count(*) from live where payment_status = 'Paid'),
    (select count(*) from (select 1 from live where student_email <> '' group by lower(student_email) having count(*) > 1) r),
    (select min(program_year) from live),
    (select max(program_year) from live);

```

### research_apply_city_targets(p_limit integer, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 30 lines
```sql

declare b uuid := gen_random_uuid(); n_city int := 0; n_state int := 0; n_t int;
begin
  create temp table _t on commit drop as
    select * from research_city_targets
     where method in ('read', 'structured_ok') and city is not null and applied_at is null
     order by contacts_no_city desc limit p_limit;
  get diagnostics n_t = row_count;
  create temp table _c on commit drop as
    select c.id, t.domain, t.city,
           case when lower(t.country) in ('united states', 'usa', 'us')
                then coalesce((select u.name from research_us_states u where u.code = upper(btrim(t.state))), t.state) else t.state end state,
           t.method, t.evidence, t.url,
           (btrim(coalesce(c.state,'')) = '' or research_place_junk(c.state)) no_state, c.city old_city, c.state old_state
      from _t t
      join research_contacts c on lower(split_part(c.email,'@',2)) = t.domain and c.deleted_at is null
       and research_school_name_key(c.school) = t.school_key
       and (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city));
  with u as (update research_contacts c set city = x.city from _c x where c.id = x.id returning c.id, x.city, x.old_city)
  insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
  select id, 'city', coalesce(old_city,''), city, 'filled', 'school website (read)', p_actor, b from u;
  get diagnostics n_city = row_count;
  with u as (update research_contacts c set state = x.state from _c x where c.id = x.id and x.no_state and btrim(coalesce(x.state,'')) <> ''
             returning c.id, x.state, x.old_state)
  insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
  select id, 'state', coalesce(old_state,''), state, 'filled', 'school website (read)', p_actor, b from u;
  get diagnostics n_state = row_count;
  update research_city_targets t set applied_at = now() from _t where t.domain = _t.domain;
  return jsonb_build_object('domains', n_t, 'cities', n_city, 'states', n_state, 'batch', b);
end 
```

### research_apply_fact_fill(p_limit integer, p_actor text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 20 lines
```sql

declare n int; b uuid := gen_random_uuid(); col text;
begin
  create temp table _p on commit drop as
    select * from research_fact_fill_plan where done_at is null order by id limit p_limit;
  get diagnostics n = row_count;
  if n = 0 then perform cron.unschedule('group-fact-fill-apply'); return 0; end if;
  foreach col in array array['city','state','country','address','website','accreditation','association','grade','fee_standard','fee_maximum','fee_currency','curriculum_raw'] loop
    execute format($q$
      with u as (
        update research_contacts c set %1$I = p.value from _p p
         where p.col = %2$L and c.id = p.contact_id
           and (btrim(coalesce(c.%1$I::text, '')) = '' or (%2$L in ('city','state','country') and research_place_junk(c.%1$I::text)))
        returning c.id, p.value)
      insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
      select id, %2$L, '', value, 'filled', 'school master (verified facts)', %3$L, %4$L::uuid from u$q$, col, col, p_actor, b);
  end loop;
  update research_fact_fill_plan f set done_at = now() from _p where f.id = _p.id;
  return n;
end 
```

### research_apply_fix_plan(p_limit integer, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 28 lines
```sql

declare b uuid := gen_random_uuid(); f text; n_done int := 0; n_skip int := 0; k int;
begin
  create temp table _p on commit drop as
    select * from research_fix_plan where applied_at is null and skipped is null order by id limit p_limit;
  for f in select distinct field from _p loop
    execute format($q$
      with u as (
        update research_contacts c set %1$I = p.new_value
          from _p p
         where p.field = %2$L and c.id = p.contact_id and c.deleted_at is null
           and coalesce(c.%1$I, '') = p.old_value
        returning c.id, p.old_value, p.new_value, p.rule)
      insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
      select id, %2$L, old_value, new_value,
             case when old_value = '' then 'filled' when new_value = '' then 'cleared' else 'corrected' end,
             'common-sense cleanup', %3$L, %4$L::uuid from u $q$, f, f, p_actor, b);
    get diagnostics k = row_count;
    n_done := n_done + k;
  end loop;
  update research_fix_plan r set applied_at = now()
    from _p p join research_field_changes fc on fc.batch_id = b and fc.contact_id = p.contact_id and fc.field = p.field
   where r.id = p.id;
  update research_fix_plan r set skipped = 'value changed since the plan was made'
    from _p p where r.id = p.id and r.applied_at is null;
  get diagnostics n_skip = row_count;
  return jsonb_build_object('applied', n_done, 'skipped', n_skip, 'batch', b);
end 
```

### research_apply_legacy_value(p_value text, p_from integer, p_to integer, p_actor text, p_batch uuid) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 19 lines
```sql

declare tk text; n int;
begin
  select tool_key into tk from research_legacy_source_map where value = p_value;
  if tk is null then return 0; end if;
  with u as (
    update research_contacts c set research_tool = tk, research_tool_set_by = 'legacy_map'
     where c.id between (lpad(to_hex(p_from), 2, '0') || '000000-0000-0000-0000-000000000000')::uuid
                    and (lpad(to_hex(p_to), 2, '0') || 'ffffff-ffff-ffff-ffff-ffffffffffff')::uuid
       and c.source_detail = p_value and c.research_tool is null and c.deleted_at is null
    returning c.id)
  insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
  select id, 'research_tool', '', tk, 'filled', 'researcher/source backfill', p_actor, p_batch from u;
  get diagnostics n = row_count;
  update research_legacy_source_map set applied_at = now(), applied_by = p_actor, applied_batch = p_batch,
         applied_rows = case when p_from = 0 then n else coalesce(applied_rows, 0) + n end
   where value = p_value;
  return n;
end 
```

### research_apply_place_fill(p_limit integer, p_actor text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 26 lines
```sql

declare n int;
begin
  create temp table _p on commit drop as
    select * from research_place_fill_plan where done_at is null limit p_limit;
  get diagnostics n = row_count;
  if n = 0 then
    perform cron.unschedule('group-place-fill-apply');
    return 0;
  end if;
  -- Only blanks/placeholders are filled; a value someone set meanwhile wins. Each fill is logged.
  with u as (
    update research_contacts c set city = p.new_city from _p p
     where c.id = p.contact_id and p.new_city is not null and (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city))
    returning c.id, p.old_city, p.new_city, p.batch)
  insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
  select id, 'city', coalesce(old_city, ''), new_city, 'filled', 'same-school', p_actor, batch from u;
  with u as (
    update research_contacts c set state = p.new_state from _p p
     where c.id = p.contact_id and p.new_state is not null and (btrim(coalesce(c.state,'')) = '' or research_place_junk(c.state))
    returning c.id, p.old_state, p.new_state, p.batch)
  insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
  select id, 'state', coalesce(old_state, ''), new_state, 'filled', 'same-school', p_actor, batch from u;
  update research_place_fill_plan f set done_at = now() from _p where f.contact_id = _p.contact_id;
  return n;
end 
```

### research_apply_registry_matches(p_actor text, p_dry boolean) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 145 lines
```sql

declare v_batch uuid := gen_random_uuid(); n_match int; n_facts int; n_fill int; n_fee int;
begin
  create temp table _us on commit drop as
  select s.id, s.city,
         coalesce((select code from research_us_states st where lower(st.name) = lower(btrim(s.state)) or st.code = upper(btrim(s.state))),
                  (select st.code from research_contacts c join research_us_states st on lower(st.name) = lower(btrim(c.state))
                    where c.school_id = s.id and c.deleted_at is null group by st.code order by count(*) desc limit 1), '') sc,
         research_school_name_key(s.name) k, research_school_name_key3(s.name) k3,
         coalesce(research_domain_state(s.domain),
                  (select research_domain_state(split_part(lower(c.email), '@', 2)) from research_contacts c
                    where c.school_id = s.id and c.deleted_at is null
                      and research_domain_state(split_part(lower(c.email), '@', 2)) is not null limit 1)) dstate,
         (s.domain ~ '(k12|\.us$|isd|usd|cisd|district|schools\.|sd[0-9]*\.|ps\.(org|net)|\.gov$)'
          or exists (select 1 from research_contacts c where c.school_id = s.id and c.deleted_at is null
-- ... truncated (145 lines total)
```

### research_apply_school_links(p_limit integer, p_actor text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 23 lines
```sql

declare n int; b uuid := gen_random_uuid();
begin
  create temp table _p on commit drop as
    select * from research_school_link_plan where done_at is null order by lower(school), lower(city), lower(country) limit p_limit;
  get diagnostics n = row_count;
  if n = 0 then perform cron.unschedule('group-school-link-apply'); return 0; end if;
  insert into research_schools (name, branch, city, state, country, domain, created_by)
  select distinct on (lower(school), lower(city), lower(country)) school, '', city, state, country,
         case when rule like 'new school%' then d else '' end, p_actor
    from _p where sid is null
   order by lower(school), lower(city), lower(country)
  on conflict do nothing;
  update _p p set sid = s.id from research_schools s
   where p.sid is null and lower(s.name) = lower(p.school) and s.branch = '' and lower(s.city) = lower(p.city)
     and lower(s.country) = lower(p.country) and s.merged_into is null;
  with u as (update research_contacts c set school_id = p.sid from _p p
              where c.id = p.contact_id and c.school_id is null and p.sid is not null returning c.id, p.sid, p.rule)
  insert into research_school_links (contact_id, school_id, rule, created_school, batch, actor)
  select id, sid, rule, false, b, p_actor from u;
  update research_school_link_plan f set done_at = now() from _p where f.contact_id = _p.contact_id;
  return n;
end 
```

### research_apply_sop(p jsonb) -> integer
- sql | SECURITY DEFINER | volatile | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 32 lines
```sql

  with v as (
    select * from jsonb_to_recordset(p) as x(
      id uuid,
      sop_role text, sop_priority text, sop_category text, sop_level text, sop_signal text,
      subject text,
      school_type text, school_type_signal text, school_type_confidence text,
      grade_band text, grade_band_signal text, grade_band_confidence text,
      classifier_version int)
  ), u as (
    update public.research_contacts c set
      sop_role = v.sop_role,
      sop_priority = v.sop_priority,
      sop_category = v.sop_category,
      sop_level = v.sop_level,
      sop_signal = v.sop_signal,
      -- Coalesced rather than assigned: a null in the payload (an older caller
      -- that does not send the column) must not blank a real value.
      subject = coalesce(v.subject, c.subject),
      school_type = v.school_type,
      school_type_signal = v.school_type_signal,
      school_type_confidence = v.school_type_confidence,
      grade_band = v.grade_band,
      grade_band_signal = v.grade_band_signal,
      grade_band_confidence = v.grade_band_confidence,
      classifier_version = v.classifier_version
    from v
    where c.id = v.id
    returning 1
  )
  select count(*)::int from u;

```

### research_apply_trust(verdicts jsonb) -> integer
- sql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 14 lines
```sql

  with v as (
    select (x->>'id')::uuid id, x->>'state' state,
           coalesce(array(select jsonb_array_elements_text(x->'reasons')), '{}') reasons,
           coalesce(x->'tried', '[]'::jsonb) tried
      from jsonb_array_elements(verdicts) x
  ), u as (
    update public.research_contacts c
       set trust_state = v.state, trust_reasons = v.reasons, trust_tried = v.tried, trust_checked_at = now()
      from v where c.id = v.id
    returning 1
  )
  select count(*)::int from u;

```

### research_assign_school_from_domain(p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 52 lines
```sql

declare
  v_batch uuid := gen_random_uuid();
  n_new int; n int;
begin
  create temp table _a on commit drop as
  select c.id, c.person_id, c.person_type, c.designation, c.school old_school, c.school_id old_id,
         btrim(coalesce(c.country, '')) country, split_part(lower(btrim(c.email)), '@', 2) d,
         case when ds.kind = 'school' and ds.school_name <> '' then ds.school_name else ds.ai_name end new_name,
         case when ds.kind = 'school' and ds.school_name <> '' then '' else ds.ai_city end new_city
    from research_contacts c
    left join research_schools s on s.id = c.school_id
    join research_domain_sites ds on ds.domain = split_part(lower(btrim(c.email)), '@', 2)
   where c.deleted_at is null
     and (c.school_id is null or btrim(coalesce(c.school, '')) = ''
-- ... truncated (52 lines total)
```

### research_auto_merge_duplicates(p_limit integer, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 24 lines
```sql

declare r record; o text; b uuid := gen_random_uuid();
  merged int := 0; closed int := 0; no_match int := 0; failed int := 0;
begin
  for r in
    select id from research_duplicate_log
     where decision = 'pending'
     order by checked_at nulls first, created_at
     limit p_limit
  loop
    begin
      o := research_merge_duplicate(r.id, p_actor, b);
    exception when others then
      -- One bad row never stops the batch; its error is on the row for anyone to see.
      update research_duplicate_log set checked_at = now(), auto_note = 'Merge error: ' || sqlerrm where id = r.id;
      o := 'failed';
    end;
    if o = 'merged' then merged := merged + 1;
    elsif o = 'closed' then closed := closed + 1;
    elsif o = 'no_match' then no_match := no_match + 1;
    elsif o = 'failed' then failed := failed + 1; end if;
  end loop;
  return jsonb_build_object('merged', merged, 'closed', closed, 'no_match', no_match, 'failed', failed, 'batch', b);
end 
```

### research_backfill_provenance_slice(p_prefix text, p_actor text, p_batch uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 43 lines
```sql

declare lo uuid := (rpad(p_prefix, 8, '0') || '-0000-0000-0000-000000000000')::uuid;
        hi uuid := (rpad(p_prefix, 8, 'f') || '-ffff-ffff-ffff-ffffffffffff')::uuid;
        n_tool int; n_ids int;
begin
  -- Tool: only from evidence — a mapped legacy value, the school's own page, or the research agent.
  with pick as (
    select c.id, coalesce(
             case when c.research_source ilike '%research agent%' or 'Research agent' = any(c.researcher_names) then 'research-agent' end,
             m.tool_key,
             case when c.source_detail ~* '^https?://' and c.website <> ''
                   and lower(regexp_replace(substring(c.source_detail from '^https?://([^/:]+)'), '^www\.', ''))
                     = lower(regexp_replace(substring(c.website from '^(?:https?://)?([^/:]+)'), '^www\.', ''))
                  then 'school-website' end) tool,
           case when c.source_detail ~* '^https?://' then c.source_detail else '' end note
-- ... truncated (43 lines total)
```

### research_backfill_tick(p_slices integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 17 lines
```sql

declare cur record; i int; r jsonb; t int := 0; ids int := 0;
begin
  select * into cur from research_backfill_cursor where id = 1 for update;
  if cur.pos >= 256 then
    perform cron.unschedule('group-provenance-backfill');
    return jsonb_build_object('done', true);
  end if;
  for i in cur.pos .. least(cur.pos + p_slices, 256) - 1 loop
    r := research_backfill_provenance_slice(lpad(to_hex(i), 2, '0'), cur.actor, cur.batch);
    t := t + (r->>'tool')::int; ids := ids + (r->>'researcher_ids')::int;
  end loop;
  update research_backfill_cursor set pos = least(cur.pos + p_slices, 256),
    totals = jsonb_build_object('tool', coalesce((totals->>'tool')::int, 0) + t, 'researcher_ids', coalesce((totals->>'researcher_ids')::int, 0) + ids)
   where id = 1;
  return jsonb_build_object('pos', least(cur.pos + p_slices, 256), 'tool', t, 'researcher_ids', ids);
end 
```

### research_build_city_targets() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 21 lines
```sql

declare n int;
begin
  create temp table _x on commit drop as
    select lower(split_part(c.email,'@',2)) d, research_school_name_key(c.school) k, btrim(c.school) school,
           lower(btrim(c.country)) country, (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)) no_city
      from research_contacts c
     where c.deleted_at is null and c.email like '%@%' and btrim(coalesce(c.school,'')) <> '' and not research_dup_junk(c.school)
       and lower(split_part(c.email,'@',2)) !~ '^(gmail|googlemail|yahoo|ymail|hotmail|outlook|aol|icloud|me|live|msn|rediffmail|protonmail|zoho|gmx|mail|yandex|qq|163)\.'
       and not coalesce(c.school ~* '\m(district|public schools|county schools|city schools|board of education|unified|isd|usd|school corporation|department of education|ministry of education)\M', false);
  create temp table _need on commit drop as select distinct d from _x where no_city;
  insert into research_city_targets (domain, school_key, school, country, contacts_no_city, total, share)
  select g.d, g.k, g.school, g.country, g.no_city_n, t.total, round(g.n::numeric / t.total, 3)
    from (select d, k, max(school) school, max(country) country, count(*) n, count(*) filter (where no_city) no_city_n
            from _x where d in (select d from _need) group by d, k) g
    join (select d, count(*) total from _x where d in (select d from _need) group by d) t on t.d = g.d
   where g.n::numeric / t.total >= 0.9 and g.no_city_n > 0
  on conflict (domain) do nothing;
  get diagnostics n = row_count;
  return n;
end 
```

### research_build_review_domain_school() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 39 lines
```sql

declare v_rows int;
begin
  create temp table _doms on commit drop as
    select distinct lower(split_part(btrim(fields->>'email'), '@', 2)) d from research_review_rows
     where btrim(coalesce(fields->>'email','')) like '%@%';
  create index on _doms (d);
  create temp table _c on commit drop as
    select lower(split_part(c.email, '@', 2)) d, research_school_name_key(c.school) k, btrim(c.school) school,
           btrim(coalesce(c.city,'')) city, btrim(coalesce(c.state,'')) state, btrim(coalesce(c.country,'')) country, c.school_id
      from research_contacts c join _doms on _doms.d = lower(split_part(c.email, '@', 2))
     where c.deleted_at is null and btrim(coalesce(c.school,'')) <> '' and not research_dup_junk(c.school);
  insert into research_review_domain_school (domain, school_key, school, support, total, share, city, city_share, state, state_share, country, country_share, school_id)
  with g as (select x.d, x.k, count(*) cnt from _c x where x.k <> '' group by 1, 2),
  t as (select g.d, sum(g.cnt) total from g group by 1),
  best as (select distinct on (g.d) g.d, g.k, g.cnt, t.total from g join t using (d) order by g.d, g.cnt desc, g.k),
  top as (select x.* from _c x join best b on b.d = x.d and b.k = x.k),
  pick as (
    select top.d, 'school' f, top.school v, count(*) cnt from top group by 1, 3
    union all select top.d, 'city', top.city, count(*) from top where top.city <> '' group by 1, 3
    union all select top.d, 'state', top.state, count(*) from top where top.state <> '' group by 1, 3
    union all select top.d, 'country', top.country, count(*) from top where top.country <> '' group by 1, 3
    union all select top.d, 'school_id', top.school_id::text, count(*) from top where top.school_id is not null group by 1, 3),
  win as (select distinct on (p.d, p.f) p.d, p.f, p.v, p.cnt, sum(p.cnt) over (partition by p.d, p.f) tot from pick p order by p.d, p.f, p.cnt desc, p.v)
  select b.d, b.k,
    max(w.v) filter (where w.f = 'school'), b.cnt, b.total, round(b.cnt::numeric / b.total, 3),
    max(w.v) filter (where w.f = 'city'), max(round(w.cnt::numeric / w.tot, 3)) filter (where w.f = 'city'),
    max(w.v) filter (where w.f = 'state'), max(round(w.cnt::numeric / w.tot, 3)) filter (where w.f = 'state'),
    max(w.v) filter (where w.f = 'country'), max(round(w.cnt::numeric / w.tot, 3)) filter (where w.f = 'country'),
    (max(w.v) filter (where w.f = 'school_id'))::uuid
  from best b left join win w on w.d = b.d
  group by b.d, b.k, b.cnt, b.total
  on conflict (domain) do update set school_key = excluded.school_key, school = excluded.school, support = excluded.support,
    total = excluded.total, share = excluded.share, city = excluded.city, city_share = excluded.city_share,
    state = excluded.state, state_share = excluded.state_share, country = excluded.country,
    country_share = excluded.country_share, school_id = excluded.school_id, built_at = now();
  get diagnostics v_rows = row_count;
  return v_rows;
end 
```

### research_bump_short_link(_slug text) -> void
- sql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=Y authenticated=Y | 5 lines
```sql

  update public.research_short_links
     set hits = hits + 1, last_hit_at = now()
   where slug = _slug;

```

### research_bump_tab_totals(p_tab uuid, p_imported integer, p_merged integer, p_duplicate integer, p_review integer, p_failed integer) -> void
- sql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 9 lines
```sql

  update public.research_upload_tabs
     set imported_rows = coalesce(imported_rows, 0) + p_imported,
         merged_rows = coalesce(merged_rows, 0) + p_merged,
         duplicate_rows = coalesce(duplicate_rows, 0) + p_duplicate,
         review_rows = coalesce(review_rows, 0) + p_review,
         failed_rows = coalesce(failed_rows, 0) + p_failed
   where id = p_tab;

```

### research_change_summary(p_days integer) -> TABLE(day date, source text, action text, field text, changes bigint, contacts bigint, reverted bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 7 lines
```sql

  select (at at time zone 'UTC')::date, source, action, field, count(*), count(distinct contact_id), count(*) filter (where reverted_at is not null)
    from research_field_changes
   where at >= now() - make_interval(days => greatest(p_days, 1))
   group by 1, 2, 3, 4
   order by 1 desc, 5 desc;

```

### research_check(p_email text, p_school text, p_contact text, p_domain text) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 88 lines
```sql

DECLARE
  ek text; edom text;
  dup_res quarantine.research_contacts%ROWTYPE; dup_pid uuid;
  dup_pair quarantine.research_contacts%ROWTYPE;
  checks jsonb := '[]'::jsonb; verdict text := 'ok';
  message text := 'Clean and unique - this can be saved.'; missing text[] := '{}';
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;

  ek := identity_key('email', p_email);
  edom := lower(split_part(coalesce(p_email,''),'@',2));

  -- ::text on every append. Without it Postgres reads `text[] || 'school'` as array || array and
  -- tries to parse the word as an array literal - and being plpgsql, it fails on the first CALL
-- ... truncated (88 lines total)
```

### research_claim_raw_rows(p_tab uuid, p_limit integer) -> TABLE(row_number integer, data jsonb)
- sql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 12 lines
```sql

  update public.research_raw_rows r
     set claimed_at = now()
   where r.id in (
     select id from public.research_raw_rows
      where upload_tab_id = p_tab and outcome is null
        and (claimed_at is null or claimed_at < now() - interval '10 minutes')
      order by row_number
      limit p_limit
      for update skip locked)
  returning r.row_number, r.data;

```

### research_clean_pass1(p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=Y | 90 lines
```sql

declare
  b uuid := gen_random_uuid();
  out jsonb := jsonb_build_object('batch', b);
  n int;
begin
  with t as (
    select id, contact_name from public.research_contacts
     where deleted_at is null and public.research_is_placeholder_name(contact_name)
  ), log as (
    insert into public.research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
    select id, 'contact_name', contact_name, '', 'updated', 'cleaning: role placeholder in the name column, not a person', p_actor, b from t
  )
  update public.research_contacts c
     set contact_name = '', generic_contact = true,
-- ... truncated (90 lines total)
```

### research_compute_data_health() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 79 lines
```sql

declare r jsonb;
begin
  create temp table _h on commit drop as
  with base as (
    select c.id,
      lower(btrim(coalesce(c.country,''))) country_l,
      (btrim(coalesce(c.contact_name,'')) = '' or research_dup_junk(c.contact_name)) no_name,
      coalesce(c.contact_name ~ '[/&]|\sand\s', false) two_people,
      (btrim(coalesce(c.designation,'')) = '' or research_dup_junk(c.designation)) no_title,
      (c.school_id is null) no_school_id,
      (btrim(coalesce(c.school,'')) = '' or research_dup_junk(c.school)) no_school,
      coalesce(c.school ~* '\m(district|public schools|county schools|city schools|board of education|unified|isd|usd|school corporation|department of education|ministry of education)\M', false) district_as_school,
      (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)) no_city,
      (btrim(coalesce(c.state,'')) = '' or research_place_junk(c.state)) no_state,
-- ... truncated (79 lines total)
```

### research_contact_decide(p_contact_id uuid, p_decision text, p_reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 29 lines
```sql

DECLARE me uuid; rc quarantine.research_contacts%ROWTYPE; res jsonb;
BEGIN
  me := auth_staff_id();
  IF me IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'You do not have a role that may decide on a researched contact' USING ERRCODE='42501'; END IF;
  IF p_decision NOT IN ('approve','reject','hold') THEN
    RAISE EXCEPTION 'The decision is approve, reject or hold' USING ERRCODE='22023'; END IF;
  IF p_decision <> 'approve' AND nullif(btrim(coalesce(p_reason,'')),'') IS NULL THEN
    RAISE EXCEPTION 'A rejection with a reason teaches the system. A silent one teaches it nothing'
      USING ERRCODE='23502'; END IF;

  SELECT * INTO rc FROM quarantine.research_contacts WHERE id = p_contact_id FOR UPDATE;
  IF rc.id IS NULL THEN RAISE EXCEPTION 'No such research contact' USING ERRCODE='P0002'; END IF;

  IF p_decision = 'approve' THEN
    res := promote_research_contact(p_contact_id);
  END IF;

  UPDATE quarantine.research_contacts
     SET review_verdict = CASE p_decision WHEN 'approve' THEN 'approved'
                                          WHEN 'reject'  THEN 'rejected' ELSE 'held' END,
         review_reason = nullif(btrim(coalesce(p_reason,'')),''),
         reviewed_by = me, reviewed_at = now()
   WHERE id = p_contact_id;

  RETURN jsonb_build_object('ok', true, 'decision', p_decision, 'promotion', res);
END 
```

### research_contact_detail(p_contact_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 9 lines
```sql

DECLARE me uuid; out jsonb;
BEGIN
  me := auth_staff_id();
  IF me IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  out := research_contact_detail_body(p_contact_id);
  IF out IS NULL THEN RAISE EXCEPTION 'No such research contact' USING ERRCODE='P0002'; END IF;
  RETURN out;
END 
```

### research_contact_detail_body(p_contact_id uuid) -> jsonb
- sql | invoker | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 63 lines
```sql

  SELECT jsonb_build_object(
    'contact', to_jsonb(rc) - 'continent_raw'::text,
    'added_by', (SELECT full_name FROM public.staff s WHERE s.id = rc.added_by),
    'reviewed_by', (SELECT full_name FROM public.staff s WHERE s.id = rc.reviewed_by),
    'promoted_person', (SELECT jsonb_build_object('id', p.id, 'name', p.full_name)
                          FROM public.people p WHERE p.id = rc.promoted_person_id),

    'gate', jsonb_build_object(
      'named_person', jsonb_build_object(
        'ok', nullif(btrim(coalesce(rc.full_name,'')),'') IS NOT NULL,
        'enforced', 'promotion refuses without a name'),
      'verified_address', jsonb_build_object(
        'ok', rc.is_valid IS TRUE AND rc.verified_at IS NOT NULL,
        'enforced', 'promotion refuses unless validated'),
-- ... truncated (63 lines total)
```

### research_contact_facets_sync() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 19 lines
```sql

begin
  if tg_op = 'DELETE' then
    delete from public.research_contact_facets where id = old.id;
    return old;
  end if;
  insert into public.research_contact_facets values (
    new.id, new.deleted_at, new.category, new.person_type, new.school, new.country, new.state, new.city, new.email,
    new.subject, new.seniority, new.sop_priority, new.grade_band, new.school_type, new.school_type_confidence,
    new.grade_band_confidence, new.sop_level, new.curriculum_row, new.geo_region)
  on conflict (id) do update set
    deleted_at = excluded.deleted_at, category = excluded.category, person_type = excluded.person_type,
    school = excluded.school, country = excluded.country, state = excluded.state, city = excluded.city,
    email = excluded.email, subject = excluded.subject, seniority = excluded.seniority,
    sop_priority = excluded.sop_priority, grade_band = excluded.grade_band, school_type = excluded.school_type,
    school_type_confidence = excluded.school_type_confidence, grade_band_confidence = excluded.grade_band_confidence,
    sop_level = excluded.sop_level, curriculum_row = excluded.curriculum_row, geo_region = excluded.geo_region;
  return new;
end 
```

### research_contact_facets_trust_sync() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 7 lines
```sql

begin
  update public.research_contact_facets
     set trust_state = new.trust_state, trust_reasons = coalesce(new.trust_reasons, '{}')
   where id = new.id;
  return new;
end 
```

### research_contact_provenance(p_contact uuid) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 22 lines
```sql

  select jsonb_build_object(
    'researcher_names', c.researcher_names,
    'researchers', (select coalesce(jsonb_agg(jsonb_build_object('id', s.id, 'name', s.name) order by x.o), '[]')
                      from unnest(c.researcher_ids) with ordinality x(id, o) join research_staff_profiles s on s.id = x.id),
    'researched_on', c.researched_on, 'researched_until', c.researched_until,
    'research_source', c.research_source, 'source', c.source, 'source_detail', c.source_detail,
    'tool', c.research_tool, 'tool_label', st.label, 'tool_kind', st.kind, 'tool_note', c.research_tool_note,
    'tool_set_by', c.research_tool_set_by, 'ai_tool', coalesce(st.needs_verification, false),
    'health', (select jsonb_build_object('tier', h.tier, 'reasons', h.reasons, 'email_status', h.email_status, 'computed_at', h.computed_at)
                 from research_contact_health h where h.contact_id = c.id),
    'sheets', (select coalesce(jsonb_agg(jsonb_build_object('file', u.file_name, 'tab', t.tab_name, 'upload_id', u.id, 'row', rr.row_number,
                                                            'outcome', rr.outcome, 'at', u.uploaded_at,
                                                            'researcher', rr.data->>(select k from jsonb_each_text(t.mapping) e(k, v) where v = 'researcher' limit 1))
                                         order by u.uploaded_at), '[]')
                 from research_raw_rows rr join research_upload_tabs t on t.id = rr.upload_tab_id join research_uploads u on u.id = t.upload_id
                where rr.contact_id = c.id),
    'folders', (select coalesce(jsonb_agg(jsonb_build_object('key', f.key, 'name', f.name, 'method', m.method, 'removed', m.removed) order by f.sort), '[]')
                  from research_folder_members m join research_folders f on f.id = m.folder_id where m.contact_id = c.id))
  from research_contacts c left join research_source_tools st on st.key = c.research_tool
  where c.id = p_contact;

```

### research_contact_readiness(p_school_ids uuid[]) -> TABLE(contact_id uuid, school_id uuid, tier text, missing text[])
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 26 lines
```sql

  select c.id, c.school_id,
    case
      when c.generic_contact then 'NOT_USABLE'
      when btrim(coalesce(c.contact_name, '')) = '' then 'NOT_USABLE'
      when c.school_id is null then 'NOT_USABLE'
      when cardinality(m.missing) = 0 then 'SALES_READY'
      else 'NEEDS_RESEARCH'
    end,
    case when c.generic_contact then array['is a generic inbox, not a person'] else m.missing end
  from research_contacts c
  cross join lateral (
    select array_remove(array[
      case when btrim(coalesce(c.contact_name, '')) = '' then 'name' end,
      -- "Maria Ferrer/Lynnette Williams": two people on one row.
      case when c.contact_name ~ '(/|&|\s+and\s+)' then 'one person per contact' end,
      case when btrim(coalesce(c.designation, '')) = '' then 'designation' end,
      case when btrim(coalesce(c.city, '')) = '' then 'city' end,
      case when btrim(coalesce(c.state, '')) = ''
            and c.country in ('United States', 'India', 'Canada', 'Australia') then 'state' end,
      case when btrim(coalesce(c.country, '')) = '' then 'country' end,
      case when btrim(coalesce(c.email, '')) = '' then 'email' end
    ], null) missing
  ) m
  where c.deleted_at is null and c.school_id = any(p_school_ids);

```

### research_contact_update(p_contact_id uuid, p_patch jsonb, p_reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 71 lines
```sql

DECLARE
  me uuid; rc quarantine.research_contacts%ROWTYPE; k text; allowed text[];
  newv text; oldv text; n int := 0; touched text[] := '{}';
BEGIN
  me := auth_staff_id();
  IF me IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
      OR has_role('outreach') OR has_role('counsellor') OR has_role('researcher')) THEN
    RAISE EXCEPTION 'You do not have a role that may correct a research contact' USING ERRCODE='42501'; END IF;

  IF nullif(btrim(coalesce(p_reason,'')),'') IS NULL THEN
    RAISE EXCEPTION 'Say why you are changing it. A correction with no reason teaches the system nothing'
      USING ERRCODE='23502'; END IF;
  IF jsonb_typeof(p_patch) <> 'object' OR p_patch = '{}'::jsonb THEN
    RAISE EXCEPTION 'Nothing to change' USING ERRCODE='22023'; END IF;
-- ... truncated (71 lines total)
```

### research_contacts_count_by_country() -> TABLE(country text, cnt bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 10 lines
```sql

  select
    country,
    count(*) as cnt
  from public.research_contacts
  where deleted_at is null
    and country is not null
    and country <> ''
  group by country;

```

### research_contacts_count_by_dataset() -> TABLE(dataset_id uuid, cnt bigint, latest_at timestamp with time zone)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 9 lines
```sql

  select
    dataset_id,
    count(*)            as cnt,
    max(created_at)     as latest_at
  from public.research_contacts
  where deleted_at is null
  group by dataset_id;

```

### research_contacts_count_by_state() -> TABLE(state text, cnt bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 10 lines
```sql

  select
    state,
    count(*) as cnt
  from public.research_contacts
  where deleted_at is null
    and state is not null
    and state <> ''
  group by state;

```

### research_contacts_tool_in() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 23 lines
```sql

declare k text; d record;
begin
  if new.research_tool is not null and not exists (select 1 from research_source_tools where key = new.research_tool) then
    k := research_tool_for(new.research_tool);
    new.research_tool_note := case when k is null then left(new.research_tool, 300) else coalesce(new.research_tool_note, '') end;
    new.research_tool := coalesce(k, 'other');
    if coalesce(new.research_tool_set_by, '') = '' then new.research_tool_set_by := 'sheet'; end if;
  end if;
  if tg_op = 'INSERT' and new.research_tool is null then
    if new.research_source ilike '%research agent%' then
      new.research_tool := 'research-agent'; new.research_tool_set_by := 'agent';
    elsif new.upload_tab_id is not null then
      select default_research_tool, default_research_tool_note into d from research_upload_tabs where id = new.upload_tab_id;
      if d.default_research_tool is not null then
        new.research_tool := d.default_research_tool;
        new.research_tool_note := coalesce(d.default_research_tool_note, '');
        new.research_tool_set_by := 'tab_default';
      end if;
    end if;
  end if;
  return new;
end 
```

### research_contacts_upload_folders() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 11 lines
```sql

begin
  if new.upload_tab_id is not null then
    insert into research_folder_members (folder_id, contact_id, method, added_by, added_at)
    select f.id, new.id, 'upload', 'upload', now()
      from research_upload_tabs t join research_folders f on f.key = any(t.folder_keys)
     where t.id = new.upload_tab_id and cardinality(t.folder_keys) > 0
    on conflict (folder_id, contact_id) do nothing;
  end if;
  return null;
end 
```

### research_contrib_tick(p_pieces integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 23 lines
```sql

declare cur int; total int; p record; done int := 0;
begin
  select pos into cur from research_contrib_cursor where id = 1 for update;
  create temp table if not exists _pieces on commit drop as
    select row_number() over (order by t.id, g) - 1 idx, t.id tab, g chunk
      from research_upload_tabs t
      cross join lateral generate_series(0, greatest(0, (t.row_count + 1) / 20000)) g
     where t.target_lane = 'contact';
  select count(*) into total from _pieces;
  for p in select * from _pieces where idx >= cur order by idx limit p_pieces loop
    perform research_refresh_contrib_chunk(p.tab, p.chunk);
    done := done + 1;
  end loop;
  cur := cur + done;
  if cur >= total then
    update research_contrib_cursor set pos = 0, last_cycle_at = now() where id = 1;
    update research_stats_contributions_meta set computed_at = now() where id = 1;
  else
    update research_contrib_cursor set pos = cur where id = 1;
  end if;
  return jsonb_build_object('pos', cur, 'total', total, 'done', done);
end 
```

### research_country_from_domain(p_actor text, p_dry boolean) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=Y authenticated=Y | 86 lines
```sql

declare
  v_batch uuid := gen_random_uuid();
  n int;
begin
  create temp table _tld(tld text primary key, country text) on commit drop;
  insert into _tld values
    ('us','United States'),('ca','Canada'),('uk','United Kingdom'),('au','Australia'),('in','India'),
    ('ae','United Arab Emirates'),('sg','Singapore'),('nz','New Zealand'),('za','South Africa'),('ie','Ireland'),
    ('hk','Hong Kong'),('my','Malaysia'),('ng','Nigeria'),('ke','Kenya'),('pk','Pakistan'),('ph','Philippines'),
    ('qa','Qatar'),('sa','Saudi Arabia'),('nl','Netherlands'),('de','Germany'),('fr','France'),('es','Spain'),
    ('it','Italy'),('ch','Switzerland'),('jp','Japan'),('cn','China'),('br','Brazil'),('mx','Mexico'),
    ('ni','Nicaragua'),('kr','South Korea'),('th','Thailand'),('id','Indonesia'),('vn','Vietnam'),('eg','Egypt'),
    ('bh','Bahrain'),('om','Oman'),('kw','Kuwait'),('lk','Sri Lanka'),('bd','Bangladesh'),('np','Nepal'),
    ('gh','Ghana'),('ug','Uganda'),('tz','Tanzania'),('se','Sweden'),('no','Norway'),('dk','Denmark'),
-- ... truncated (86 lines total)
```

### research_desk_summary(p_days integer) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 18 lines
```sql

  SELECT jsonb_build_object(
    'mine_today', (SELECT count(*) FROM quarantine.research_contacts
                    WHERE source_table='research_desk' AND landed_at >= current_date),
    'desk_total', (SELECT count(*) FROM quarantine.research_contacts WHERE source_table='research_desk'),
    'desk_last_n_days', (SELECT count(*) FROM quarantine.research_contacts
                          WHERE source_table='research_desk'
                            AND landed_at >= current_date - least(greatest(coalesce(p_days,7),1),90)),
    'by_day', coalesce((
      SELECT jsonb_agg(jsonb_build_object('day',d.day,'entries',d.n) ORDER BY d.day)
        FROM (SELECT landed_at::date AS day, count(*) AS n
                FROM quarantine.research_contacts
               WHERE source_table='research_desk'
                 AND landed_at >= current_date - least(greatest(coalesce(p_days,7),1),90)
               GROUP BY 1) d), '[]'::jsonb),
    'awaiting_promotion', (SELECT count(*) FROM quarantine.research_contacts
                            WHERE source_table='research_desk' AND promoted_person_id IS NULL));

```

### research_district_lead_targets(p_limit integer) -> TABLE(domain text, leads bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 8 lines
```sql

  select lower(split_part(o.fields->>'email','@',2)) d, count(*)
    from research_review_outcomes o
   where o.outcome = 'lead' and o.note like 'Our records put %' or (o.outcome = 'lead' and o.note like '%homepage is a district%')
   group by 1
  having lower(split_part(min(o.fields->>'email'),'@',2)) not in (select s.domain from research_district_scans s)
   order by 2 desc limit p_limit;

```

### research_domain_state(d text) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select upper(coalesce(substring(lower(d) from '\.([a-z]{2})\.us$'), substring(lower(d) from '(?:^|\.)([a-z]{2})schools\.us$')));

```

### research_dup_bare_name(v text) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select btrim(regexp_replace(coalesce(v, ''), '^\s*(mr|mrs|ms|miss|mx|dr|prof)\.?\s+', '', 'i'))

```

### research_dup_generic_inbox(e text) -> boolean
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 4 lines
```sql

  select split_part(lower(btrim(coalesce(e, ''))), '@', 1) ~ ('^(info|office|admin|admins|administration|admission|admissions|contact|contactus|enquiry|enquiries|inquiry|inquiries|hello|hi|mail|email|school|principal|principaloffice|headteacher|headmaster|head|reception|frontdesk|secretary|registrar|accounts|accounting|finance|hr|careers|jobs|support|help|team|general|communications|media|pr|marketing)([._-]?[a-z0-9]{0,6})?$')
      or split_part(lower(btrim(coalesce(e, ''))), '@', 1) ~ '(principal|office|admission|reception|enquir|inquir|secretary|registrar|headteacher|headmaster|contact|frontdesk)'

```

### research_dup_junk(v text) -> boolean
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select coalesce(btrim(v), '') ~* '^(general|n/?a|na|none|null|nil|unknown|tbd|tba|tbc|ytbc|other|others|misc|mixed|various|multiple|-+|\.+|0|x|\?+)$'

```

### research_dup_merge_on_insert() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 8 lines
```sql

begin
  begin
    perform research_merge_duplicate(new.id, 'Auto-merge (instant)', gen_random_uuid());
  exception when others then null;
  end;
  return null;
end 
```

### research_dup_place_ok(p_field text, p_value text, p_contact uuid) -> boolean
- plpgsql | SECURITY DEFINER | stable | config: search_path=public | exec anon=Y authenticated=Y | 16 lines
```sql

declare t record;
begin
  select school_id, school, country into t from research_contacts where id = p_contact;
  if not found or research_place_junk(p_value) then return false; end if;
  if p_field = 'state' and lower(btrim(coalesce(t.country, ''))) in ('united states', 'usa', 'us', 'united states of america')
     and exists (select 1 from research_us_states s where lower(s.name) = lower(btrim(p_value)) or lower(s.code) = lower(btrim(p_value))) then
    return true;
  end if;
  return exists (
    select 1 from research_contacts c
     where c.id <> p_contact and c.deleted_at is null
       and ((t.school_id is not null and c.school_id = t.school_id)
            or (btrim(coalesce(t.school, '')) <> '' and lower(btrim(c.school)) = lower(btrim(t.school))))
       and lower(btrim(case p_field when 'state' then c.state when 'city' then c.city else c.country end)) = lower(btrim(p_value)));
end 
```

### research_editable_fields() -> jsonb
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=Y | 13 lines
```sql

  SELECT jsonb_agg(jsonb_build_object('field', f, 'label', l, 'kind', k) ORDER BY ord)
    FROM (VALUES
      ('full_name',       'Contact name',      'text', 1),
      ('designation_raw', 'Designation',       'text', 2),
      ('school_name_raw', 'School',            'text', 3),
      ('country_raw',     'Country',           'text', 4),
      ('state_raw',       'State / province',  'text', 5),
      ('city_raw',        'City',              'text', 6),
      ('phone_raw',       'Phone',             'text', 7),
      ('organization_id', 'Linked school',     'uuid', 8)
    ) AS v(f, l, k, ord)

```

### research_elementary_backlog(p_staff uuid, p_top integer, p_reason text) -> TABLE(school text, city text, state text, country text, contacts bigint, mine bigint, reason text, total_schools bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 56 lines
```sql

  with mine_schools as (
    select distinct
      lower(school) || '|' || lower(coalesce(country, '')) || '|' || lower(coalesce(state, '')) as key
      from public.research_contacts
     where deleted_at is null
       and person_type = 'staff'
       and school <> ''
       and created_by = p_staff
  ), scoped as (
    select c.*,
      lower(c.school) || '|' || lower(coalesce(c.country, '')) || '|' || lower(coalesce(c.state, '')) as key
      from public.research_contacts c
     where c.deleted_at is null and c.person_type = 'staff' and c.school <> ''
  ), per_school as (
-- ... truncated (56 lines total)
```

### research_email_check_totals() -> TABLE(ready bigint, risky bigint, do_not_send bigint, unchecked bigint, used_today bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 5 lines
```sql

  select coalesce(s.ready, 0), coalesce(s.risky, 0), coalesce(s.do_not_send, 0), coalesce(s.unchecked, 0),
         (select coalesce(sum(credits_used), 0) from research_email_checks where checked_at >= date_trunc('day', now()))
    from (select 1) one left join research_stats_email_checks s on s.id = 1;

```

### research_estimate_fixable() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 28 lines
```sql

declare r jsonb;
begin
  create temp table _s on commit drop as
    with x as (
      select lower(btrim(school)) sk, lower(btrim(country)) ck, btrim(city) city, btrim(state) state
        from research_contacts where deleted_at is null and btrim(coalesce(school,'')) <> '' and not research_dup_junk(school)),
    cities as (select sk, ck, city, count(*) n from x where city <> '' and not research_place_junk(city) group by 1,2,3),
    ct as (select sk, ck, sum(n) tot, max(n) top from cities group by 1,2),
    states as (select sk, ck, state, count(*) n from x where state <> '' and not research_place_junk(state) group by 1,2,3),
    st as (select sk, ck, sum(n) tot, max(n) top from states group by 1,2)
    select coalesce(ct.sk, st.sk) sk, coalesce(ct.ck, st.ck) ck,
           (ct.tot >= 2 and ct.top::numeric / ct.tot >= 0.9) city_ok, (st.tot >= 2 and st.top::numeric / st.tot >= 0.9) state_ok
      from ct full join st on st.sk = ct.sk and st.ck = ct.ck;
  create index on _s (sk, ck);
  select jsonb_build_object('kind', 'fixable',
    'missing_city', count(*) filter (where btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)),
    'city_from_same_school', count(*) filter (where (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)) and s.city_ok),
    'missing_state', count(*) filter (where btrim(coalesce(c.state,'')) = '' or research_place_junk(c.state)),
    'state_from_same_school', count(*) filter (where (btrim(coalesce(c.state,'')) = '' or research_place_junk(c.state)) and s.state_ok),
    'no_school_id', count(*) filter (where c.school_id is null),
    'no_school_id_with_school_name', count(*) filter (where c.school_id is null and btrim(coalesce(c.school,'')) <> '' and not research_dup_junk(c.school)))
  into r
  from research_contacts c left join _s s on s.sk = lower(btrim(c.school)) and s.ck = lower(btrim(c.country))
  where c.deleted_at is null;
  insert into research_data_health (summary) values (r);
  return r;
end 
```

### research_fill_place_from_school(p_apply boolean, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 53 lines
```sql

declare b uuid := gen_random_uuid(); r jsonb; n_city int := 0; n_state int := 0;
begin
  -- Contacts on a real school email domain, keyed by school name + country + domain.
  create temp table _x on commit drop as
    select c.id, research_school_name_key(c.school) k, lower(btrim(coalesce(c.country,''))) ck, lower(split_part(c.email,'@',2)) d,
           btrim(coalesce(c.city,'')) city, btrim(coalesce(c.state,'')) state,
           (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)) no_city,
           (btrim(coalesce(c.state,'')) = '' or research_place_junk(c.state)) no_state
      from research_contacts c
     where c.deleted_at is null and btrim(coalesce(c.school,'')) <> '' and not research_dup_junk(c.school)
       and btrim(coalesce(c.country,'')) <> '' and not research_place_junk(c.country)
       and c.email like '%@%'
       and lower(split_part(c.email,'@',2)) !~ '^(gmail|googlemail|yahoo|ymail|hotmail|outlook|aol|icloud|me|live|msn|rediffmail|protonmail|zoho|gmx|mail|yandex|qq|163)\.';
  create index on _x (k, ck, d);
-- ... truncated (53 lines total)
```

### research_fix_mojibake(t text) -> text
- plpgsql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 18 lines
```sql

declare r text;
begin
  if t is null or t !~ '(Ã.|â€|Ä.|Å.|Â.)' then return t; end if;
  begin
    r := convert_from(convert_to(t, 'WIN1252'), 'UTF8');
  exception when others then
    return t;
  end;
  if r ~ '(Ã.|â€|Ä.|Å.)' then
    begin
      r := convert_from(convert_to(r, 'WIN1252'), 'UTF8');  -- encoded twice
    exception when others then
      return t;
    end;
  end if;
  return r;
end 
```

### research_fix_mojibake_pass(p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 62 lines
```sql

declare
  v_batch uuid := gen_random_uuid();
  col text; n int; total int := 0; pairs jsonb; merged jsonb := '{}'::jsonb; renamed int; cand int; dupes int := 0;
begin
  foreach col in array array['school', 'contact_name', 'designation', 'city', 'state', 'country', 'address'] loop
    execute format(
      'create temp table %2$I on commit drop as
       select c.id, c.%1$I old, research_fix_mojibake(c.%1$I) new,
              lower(case when %1$L = ''school'' then research_fix_mojibake(c.school) else c.school end) k_school,
              lower(case when %1$L = ''contact_name'' then research_fix_mojibake(c.contact_name) else c.contact_name end) k_name,
              btrim(coalesce(c.contact_name, '''')) <> '''' named
         from research_contacts c
        where c.deleted_at is null and c.%1$I ~ ''(Ã.|â€|Ä.|Å.|Â.)''
          and research_fix_mojibake(c.%1$I) is distinct from c.%1$I', col, '_c_' || col);
-- ... truncated (62 lines total)
```

### research_folder_contacts(p_folder uuid, p_tier text, p_researcher text, p_tool text, p_tab uuid, p_country text, p_email text, p_ai text, p_search text, p_from date, p_to date, p_export boolean, p_limit integer, p_offset integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 37 lines
```sql

declare ids uuid[]; page uuid[]; res jsonb;
begin
  -- Match once, then count and page from the id list (an ORDER BY … LIMIT over the
  -- join made the planner walk the whole contacts index).
  ids := array(
    select c.id
      from research_folder_members m
      join research_contacts c on c.id = m.contact_id and c.deleted_at is null
      left join research_contact_health h on h.contact_id = c.id
      left join research_source_tools st on st.key = c.research_tool
     where m.folder_id = p_folder and not m.removed
       and (p_tier = '' or h.tier = p_tier)
       and (p_researcher = '' or p_researcher ilike any(c.researcher_names))
       and (p_tool = '' or (p_tool = '-' and c.research_tool is null) or c.research_tool = p_tool)
       and (p_tab is null or c.upload_tab_id = p_tab)
       and (p_country = '' or c.country = p_country)
       and (p_email = '' or coalesce(h.email_status, 'unchecked') = p_email)
       and (p_ai = '' or (p_ai = 'yes') = coalesce(st.needs_verification, false))
       and (p_search = '' or c.contact_name ilike '%' || p_search || '%' or c.school ilike '%' || p_search || '%' or c.email ilike '%' || p_search || '%')
       and (p_from is null or c.researched_on >= p_from)
       and (p_to is null or c.researched_on <= p_to)
       and (not p_export or coalesce(c.legal_state, 'allowed') = 'allowed')
     order by c.id);
  page := ids[p_offset + 1 : p_offset + least(p_limit, 1000)];
  select coalesce(jsonb_agg(r order by r.id), '[]') into res from (
    select c.id, c.contact_name, c.designation, c.school, c.city, c.state, c.country, c.email, c.phone, c.website,
           c.linkedin_url, c.seniority, c.sop_role, c.researcher_names, c.researched_on, c.research_tool, st.label tool_label,
           coalesce(st.needs_verification, false) ai_tool, c.research_tool_note, c.source_detail, c.source_file,
           h.tier, h.email_status, h.reasons, m.method, m.added_at
      from unnest(page) p(id)
      join research_contacts c on c.id = p.id
      join research_folder_members m on m.folder_id = p_folder and m.contact_id = c.id
      left join research_contact_health h on h.contact_id = c.id
      left join research_source_tools st on st.key = c.research_tool) r;
  return jsonb_build_object('total', coalesce(cardinality(ids), 0), 'rows', res);
end 
```

### research_folder_facets(p_folder uuid) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 12 lines
```sql

  with m as (
    select c.* from research_folder_members fm join research_contacts c on c.id = fm.contact_id and c.deleted_at is null
     where fm.folder_id = p_folder and not fm.removed)
  select jsonb_build_object(
    'researchers', (select coalesce(jsonb_agg(jsonb_build_object('v', v, 'n', n) order by n desc), '[]') from (select v, count(*) n from m, unnest(m.researcher_names) v group by v order by 2 desc limit 60) x),
    'tools', (select coalesce(jsonb_agg(jsonb_build_object('v', coalesce(research_tool, '-'), 'n', n) order by n desc), '[]') from (select research_tool, count(*) n from m group by 1) x),
    'countries', (select coalesce(jsonb_agg(jsonb_build_object('v', country, 'n', n) order by n desc), '[]') from (select country, count(*) n from m where country <> '' group by 1 order by 2 desc limit 80) x),
    'tabs', (select coalesce(jsonb_agg(jsonb_build_object('v', upload_tab_id, 'label', coalesce(u.file_name, '') || ' · ' || coalesce(t.tab_name, ''), 'n', n) order by n desc), '[]')
               from (select upload_tab_id, count(*) n from m where upload_tab_id is not null group by 1 order by 2 desc limit 60) x
               left join research_upload_tabs t on t.id = x.upload_tab_id left join research_uploads u on u.id = t.upload_id));

```

### research_folder_title_regex(p_titles jsonb) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 6 lines
```sql

  select case when count(*) = 0 then null else '\m(' || string_agg(
    case when length(w) <= 4 then regexp_replace(w, '([.+*?^$()\[\]{}|\\])', '\\\1', 'g') || '\M'
         else regexp_replace(w, '([.+*?^$()\[\]{}|\\])', '\\\1', 'g') end, '|') || ')' end
  from (select lower(btrim(x)) w from jsonb_array_elements_text(coalesce(p_titles, '[]'::jsonb)) x where btrim(x) <> '') s

```

### research_founder_facts() -> TABLE(country text, state text, city text, week date, sellable boolean, held_by_verification boolean, do_not_contact boolean, not_allowed boolean, n bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 16 lines
```sql

  with c as (
    select btrim(coalesce(country, '')) country, btrim(coalesce(state, '')) state, btrim(coalesce(city, '')) city,
           date_trunc('week', created_at)::date week,
           coalesce(sop_priority, '') = 'do_not_contact' dnc,
           coalesce(legal_state, 'allowed') not in ('', 'allowed') na,
           coalesce(essentials, 0) >= 100 ess,
           coalesce(verify_state, '') vs, reverify_due
      from public.research_contacts where deleted_at is null
  )
  select country, state, city, week,
         ess and not dnc and not na,
         (ess and not dnc and not na) and ((vs <> '' and vs <> 'verified') or (reverify_due is not null and reverify_due < current_date)),
         dnc, na, count(*)
    from c group by 1, 2, 3, 4, 5, 6, 7, 8;

```

### research_founder_facts_v2() -> TABLE(country text, state text, city text, week date, month_start date, sellable boolean, held_by_verification boolean, do_not_contact boolean, not_allowed boolean, n bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 16 lines
```sql

  with c as (
    select btrim(coalesce(country, '')) c_country, btrim(coalesce(state, '')) c_state, btrim(coalesce(city, '')) c_city,
           date_trunc('week', created_at)::date c_week, date_trunc('month', created_at)::date c_month,
           coalesce(sop_priority, '') = 'do_not_contact' dnc,
           coalesce(legal_state, 'allowed') not in ('', 'allowed') na,
           coalesce(essentials, 0) >= 100 ess,
           coalesce(verify_state, '') vs, reverify_due
      from public.research_contacts where deleted_at is null
  )
  select c_country, c_state, c_city, c_week, c_month,
         ess and not dnc and not na,
         (ess and not dnc and not na) and ((vs <> '' and vs <> 'verified') or (reverify_due is not null and reverify_due < current_date)),
         dnc, na, count(*)
    from c group by 1, 2, 3, 4, 5, 6, 7, 8, 9;

```

### research_gen_code(prefix text, tbl text, col text) -> text
- plpgsql | invoker | volatile | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 15 lines
```sql

declare
  alphabet constant text := 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  code text; taken boolean; i int;
begin
  loop
    code := prefix;
    for i in 1..7 loop
      code := code || substr(alphabet, 1 + floor(random() * 36)::int, 1);
    end loop;
    execute format('select exists(select 1 from public.%I where %I = $1)', tbl, col) into taken using code;
    exit when not taken;
  end loop;
  return code;
end 
```

### research_generate_school_tasks() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 30 lines
```sql

declare n int;
begin
  with s as (
    select c.school_id,
           count(*) contacts,
           count(*) filter (where c.generic_contact) generic,
           count(*) filter (where not c.generic_contact and btrim(coalesce(c.contact_name, '')) <> '' and btrim(coalesce(c.designation, '')) = '') no_title,
           count(*) filter (where btrim(coalesce(c.city, '')) = '') no_city,
           min(lower(split_part(c.email, '@', 2))) domain
      from public.research_contacts c
     where c.deleted_at is null and c.school_id is not null and c.person_type = 'staff'
     group by c.school_id
  )
  insert into public.research_tasks (kind, dedupe_key, priority, title, reason, school_id, domain, created_by)
  select 'research_school', 'research_school:' || s.school_id,
         s.contacts + 10 * s.generic + 5 * s.no_title,
         sc.name,
         array_to_string(array_remove(array[
           case when s.generic > 0 then s.generic || ' generic inbox(es) with no named person' end,
           case when s.no_title > 0 then s.no_title || ' contact(s) without a job title' end,
           case when s.no_city > 0 then 'city missing on ' || s.no_city || ' contact(s)' end
         ], null), '; '),
         s.school_id, coalesce(s.domain, ''), 'Task generator'
    from s join public.research_schools sc on sc.id = s.school_id
   where s.generic > 0 or s.no_title > 0 or s.no_city > 0
  on conflict (dedupe_key) do nothing;
  get diagnostics n = row_count;
  return n;
end 
```

### research_generate_school_tasks(p_prefix text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 33 lines
```sql

declare n int;
begin
  with s as (
    select c.school_id,
           count(*) contacts,
           count(*) filter (where c.generic_contact) generic,
           count(*) filter (where not c.generic_contact and btrim(coalesce(c.contact_name, '')) <> '' and btrim(coalesce(c.designation, '')) = '') no_title,
           count(*) filter (where btrim(coalesce(c.city, '')) = '') no_city,
           min(lower(split_part(c.email, '@', 2))) domain
      from public.research_contacts c
     where c.deleted_at is null and c.school_id is not null and c.person_type = 'staff'
       -- a School ID range (index), not a text prefix (full scan)
       and c.school_id between (rpad(p_prefix, 8, '0') || '-0000-0000-0000-000000000000')::uuid
                           and (rpad(p_prefix, 8, 'f') || '-ffff-ffff-ffff-ffffffffffff')::uuid
     group by c.school_id
  )
  insert into public.research_tasks (kind, dedupe_key, priority, title, reason, school_id, domain, created_by)
  select 'research_school', 'research_school:' || s.school_id,
         s.contacts + 10 * s.generic + 5 * s.no_title,
         sc.name,
         array_to_string(array_remove(array[
           case when s.generic > 0 then s.generic || ' generic inbox(es) with no named person' end,
           case when s.no_title > 0 then s.no_title || ' contact(s) without a job title' end,
           case when s.no_city > 0 then 'city missing on ' || s.no_city || ' contact(s)' end
         ], null), '; '),
         s.school_id, coalesce(s.domain, ''), 'Task generator'
    from s join public.research_schools sc on sc.id = s.school_id and sc.merged_into is null
   where s.generic > 0 or s.no_title > 0 or s.no_city > 0
  on conflict (dedupe_key) do nothing;
  get diagnostics n = row_count;
  return n;
end 
```

### research_has_role_word(v text) -> boolean
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select public.research_norm_words(v) ~ '(principal|head|teacher|director|counsel|coordinator|dean|admin|manager|officer|secretary|president|chair|founder|owner|librar|registrar|superintendent|leader|advis|specialist|assistant|associate|vice|chief|ceo|cfo|coo|board|trustee|member|staff|faculty|nurse|coach|lecturer|professor|instructor|tutor|incharge|in charge|controller|executive|supervisor|consultant|psycholog|social|marketing|sales|hr|finance|account|education|academic|admission|campus|department|grade|science|math|english|history|art|music|sport|physical|curriculum|programme|program|correspondent|rector|provost|mentor|guidance|career|college|university|school|trainer|analyst|engineer|designer|developer|clerk|reception|technician|aide|intern|pastor|curator|custodian|educator|parent|guardian|language|general)';

```

### research_id_health() -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 12 lines
```sql

  select jsonb_build_object(
    'contacts', (select count(*) from research_contacts where deleted_at is null),
    'contacts_without_id', (select count(*) from research_contacts where deleted_at is null and person_id is null),
    'contacts_sharing_id', (select coalesce(sum(n), 0) from (select count(*) n from research_contacts
                              where deleted_at is null and person_id is not null group by person_id having count(*) > 1) x),
    'contacts_without_school', (select count(*) from research_contacts where deleted_at is null and school_id is null),
    'schools', (select count(*) from research_schools),
    'duplicate_school_codes', (select count(*) - count(distinct school_code) from research_schools),
    'duplicate_contact_codes', (select count(*) - count(distinct contact_code) from research_people)
  );

```

### research_identity_hygiene(p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 18 lines
```sql

declare
  out jsonb := '{}'::jsonb;
begin
  out := out || jsonb_build_object('link', research_link_identities(50000, p_actor));
  out := out || jsonb_build_object('clean', research_clean_pass1(p_actor));
  out := out || jsonb_build_object('characters', research_fix_mojibake_pass(p_actor));
  out := out || jsonb_build_object('school_from_website', research_assign_school_from_domain(p_actor));
  out := out || jsonb_build_object('country', research_country_from_domain(p_actor, false));
  out := out || jsonb_build_object('split_bundles', research_split_school_bundles(10000, p_actor));
  out := out || jsonb_build_object('school_country', research_sync_school_country(p_actor));
  out := out || jsonb_build_object('registry', research_apply_registry_matches(p_actor, false));
  out := out || jsonb_build_object('split_people', research_split_shared_people(p_actor));
  out := out || jsonb_build_object('new_tasks', research_generate_school_tasks());
  out := out || jsonb_build_object('health', research_id_health());
  insert into research_cleaning_runs (actor, result) values (p_actor, out);
  return out;
end 
```

### research_identity_hygiene_cron(p_actor text, p_link_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 36 lines
- comment: Bounded cron-safe identity linker. Heavy/unbounded cleanup and registry passes are intentionally excluded to prevent statement timeout and resource spikes.
```sql

DECLARE
  v_link jsonb;
  v_out jsonb;
BEGIN
  IF p_link_limit < 1 OR p_link_limit > 5000 THEN
    RAISE EXCEPTION 'p_link_limit must be between 1 and 5000';
  END IF;

  v_link := public.research_link_identities(p_link_limit, p_actor);

  v_out := jsonb_build_object(
    'mode', 'cron_safe',
    'actor', p_actor,
    'link_limit', p_link_limit,
    'link', v_link,
    'deferred', jsonb_build_array(
      'research_clean_pass1',
      'research_fix_mojibake_pass',
      'research_assign_school_from_domain',
      'research_country_from_domain',
      'research_split_school_bundles',
      'research_sync_school_country',
      'research_apply_registry_matches',
      'research_split_shared_people',
      'research_generate_school_tasks'
    ),
    'note', 'Heavy/unbounded reconciliation passes are intentionally excluded from the cron-safe identity job.'
  );

  INSERT INTO public.research_cleaning_runs(actor, result)
  VALUES (p_actor, v_out);

  RETURN v_out;
END

```

### research_insert_contacts(p_rows jsonb) -> TABLE(id uuid, source_row_number integer)
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 13 lines
```sql

declare cols text;
begin
  select string_agg(quote_ident(k), ', ')
    into cols
    from (select distinct jsonb_object_keys(x) k from jsonb_array_elements(p_rows) x) keys
   where exists (select 1 from information_schema.columns c
                  where c.table_schema = 'public' and c.table_name = 'research_contacts' and c.column_name = keys.k);
  return query execute format(
    'insert into public.research_contacts (%1$s) select %1$s from jsonb_populate_recordset(null::public.research_contacts, $1)
     on conflict do nothing returning research_contacts.id, research_contacts.source_row_number', cols)
  using p_rows;
end 
```

### research_is_placeholder_name(v text) -> boolean
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 12 lines
```sql

  select public.research_norm_words(v) = any (array[
    'school leader','school leaders','school head','leader','principal','the principal','vice principal',
    'head','head master','headmaster','head mistress','headmistress','head of school','head teacher','headteacher',
    'teacher','teachers','counsellor','counselor','career counsellor','career counselor','admin','administrator',
    'administration','director','coordinator','co ordinator','librarian','office','school office','front office',
    'reception','receptionist','admission','admissions','admission office','info','information','contact',
    'contact person','general','general contact','staff','faculty','secretary','registrar','dean','president',
    'superintendent','management','manager','owner','founder','trustee','chairman','chairperson','correspondent',
    'hr','accounts','it','na','n a','nil','none','unknown','not available','not known','tbd','sir','madam','mam',
    'ma am','mr','mrs','ms','dr','team','school','enquiry','enquiries','inquiry','hello']);

```

### research_library_overview() -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 15 lines
```sql

  select jsonb_build_object(
    'tiers', (select jsonb_object_agg(tier, n) from (select tier, count(*) n from research_contact_health group by tier) t),
    'folders', (select coalesce(jsonb_agg(f order by f.sort, f.name), '[]') from (
      select fo.id, fo.key, fo.name, fo.description, fo.kind, fo.programme_family, fo.rules, fo.color, fo.sort, fo.archived, fo.refreshed_at,
             count(m.contact_id) filter (where not m.removed) members,
             count(m.contact_id) filter (where not m.removed and h.tier = 'SALES_READY') clean,
             count(m.contact_id) filter (where not m.removed and h.email_status = 'ready') email_ready,
             count(m.contact_id) filter (where not m.removed and m.method = 'manual') manual
        from research_folders fo
        left join research_folder_members m on m.folder_id = fo.id
        left join research_contact_health h on h.contact_id = m.contact_id
       group by fo.id) f),
    'refreshed', (select last_cycle_at from research_library_cursor where id = 1));

```

### research_library_report(p_by text, p_from date, p_to date) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 25 lines
```sql

  with s as (
    select s.*, case p_by
        when 'researcher' then s.researcher_label || case when s.researcher_basis = 'tab_default' then ' (set at upload)' else '' end
        when 'tool' then coalesce(st.label, nullif(s.tool_key, ''), 'Not recorded')
        else coalesce(u.file_name, '?') || ' · ' || coalesce(t.tab_name, '?') end grp
      from research_stats_contributions s
      left join research_source_tools st on st.key = s.tool_key
      left join research_upload_tabs t on t.id = s.upload_tab_id
      left join research_uploads u on u.id = t.upload_id
     where s.rows_n > 0
       and (p_from is null or s.researched_month >= date_trunc('month', p_from))
       and (p_to is null or s.researched_month <= p_to))
  select jsonb_build_object(
    'computed_at', (select computed_at from research_stats_contributions_meta where id = 1),
    'rows', coalesce(jsonb_agg(g order by g.rows_n desc), '[]'))
  from (
    select grp, sum(rows_n)::int rows_n,
      sum(rows_n) filter (where outcome = 'imported')::int new_rows,
      sum(rows_n) filter (where outcome in ('merged', 'duplicate'))::int dup_rows,
      sum(rows_n) filter (where outcome = 'review')::int review_rows,
      sum(rows_n) filter (where outcome not in ('imported', 'merged', 'duplicate', 'review'))::int other_rows,
      sum(clean_now)::int clean, sum(email_ready_now)::int email_ready, sum(bad_email_now)::int bad_email, sum(ai_rows)::int ai_rows
    from s group by grp) g;

```

### research_library_tick(p_slices integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 20 lines
```sql

declare cur int; i int; pfx text; h int := 0; m int := 0;
begin
  select pos into cur from research_library_cursor where id = 1 for update;
  if cur = 0 then update research_library_cursor set cycle_started_at = now() where id = 1; end if;
  for i in 0 .. p_slices - 1 loop
    exit when cur + i > 255;
    pfx := lpad(to_hex(cur + i), 2, '0');
    h := h + research_refresh_health_slice(pfx);
    m := m + research_refresh_folders_slice(pfx);
  end loop;
  cur := cur + p_slices;
  if cur > 255 then
    update research_library_cursor set pos = 0, last_cycle_at = now() where id = 1;
    update research_folders set refreshed_at = now() where not archived;
  else
    update research_library_cursor set pos = cur where id = 1;
  end if;
  return jsonb_build_object('next', cur, 'health_rows', h, 'member_changes', m);
end 
```

### research_link_by_domain(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 42 lines
```sql

DECLARE me uuid; who text; lim int; n_linked int; n_alias int; is_sched boolean;
BEGIN
  me       := auth_staff_id();
  is_sched := session_user IN ('postgres', 'supabase_admin');

  IF me IS NULL AND NOT is_sched AND coalesce(auth.role(), '') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  IF me IS NOT NULL AND NOT (has_role('owner') OR has_role('admin') OR has_role('manager')
                             OR has_role('engineer')) THEN
    RAISE EXCEPTION 'You do not have a role that may link contacts to organisations'
      USING ERRCODE='42501'; END IF;

  who := CASE WHEN me IS NULL THEN 'agent:org-classifier' ELSE me::text END;
  lim := least(greatest(coalesce(p_limit, 5000), 1), 50000);
-- ... truncated (42 lines total)
```

### research_link_identities(p_limit integer, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 69 lines
```sql

declare n_schools int; n_school_links int; n_known int; n_people int;
begin
  create temp table _b on commit drop as
  select c.id, btrim(coalesce(c.school, '')) school, btrim(coalesce(c.city, '')) city, btrim(coalesce(c.state, '')) state,
         btrim(coalesce(c.country, '')) country, lower(btrim(coalesce(c.email, ''))) email, btrim(coalesce(c.phone, '')) phone,
         btrim(coalesce(c.contact_name, '')) name, btrim(coalesce(c.designation, '')) designation,
         case when c.person_type in ('staff', 'student', 'parent', 'counsellor') then c.person_type else 'staff' end ptype,
         btrim(coalesce(c.website, '')) website
    from public.research_contacts c
   where c.deleted_at is null
     and ((c.person_id is null and btrim(coalesce(c.email, '')) <> '')
          or (c.school_id is null and btrim(coalesce(c.school, '')) <> ''))
   limit p_limit;

-- ... truncated (69 lines total)
```

### research_link_schools_safe(p_apply boolean, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 71 lines
```sql

declare b uuid := gen_random_uuid(); r jsonb; v_new int := 0; v_linked int := 0;
begin
  create temp table _c on commit drop as
    select c.id, btrim(c.school) school, research_school_name_key(c.school) k,
           btrim(coalesce(c.city,'')) city, btrim(coalesce(c.state,'')) state, btrim(coalesce(c.country,'')) country,
           lower(split_part(c.email,'@',2)) d,
           (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)) no_city,
           lower(split_part(c.email,'@',2)) ~ '^(gmail|googlemail|yahoo|ymail|hotmail|outlook|aol|icloud|me|live|msn|rediffmail|protonmail|zoho|gmx|mail|yandex|qq|163)\.' free
      from research_contacts c
     where c.deleted_at is null and c.school_id is null
       and btrim(coalesce(c.school,'')) <> '' and not research_dup_junk(c.school)
       and btrim(coalesce(c.country,'')) <> '' and not research_place_junk(c.country);
  create index on _c (k, country);
  create temp table _s on commit drop as
-- ... truncated (71 lines total)
```

### research_mark_touched(_ids uuid[], _fields text[], _action text, _actor text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 19 lines
```sql

declare
  _touched integer;
  _map jsonb;
begin
  select coalesce(jsonb_object_agg(f, _action), '{}'::jsonb)
    into _map
    from unnest(coalesce(_fields, array[]::text[])) as f;

  update public.research_contacts
     set touched_action = _action,
         touched_at = now(),
         touched_by = coalesce(_actor, ''),
         origin = coalesce(origin, '{}'::jsonb) || _map
   where id = any(_ids);

  get diagnostics _touched = row_count;
  return _touched;
end 
```

### research_match_schools(p_names text[]) -> TABLE(school_key text, contact_id uuid)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 8 lines
```sql

  select distinct on (lower(c.school)) lower(c.school), c.id
  from public.research_contacts c
  where c.deleted_at is null
    and c.person_type = 'staff'
    and lower(c.school) = any (p_names)
  order by lower(c.school), c.serial;

```

### research_merge_duplicate(p_id uuid, p_actor text, p_batch uuid) -> text
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 114 lines
```sql

declare
  d record; t record; tid uuid; pid uuid; owner uuid;
  k text; inc text; cur text; em text; n int;
  filled text[] := '{}'; notes text[] := '{}';
  us boolean;
begin
  select * into d from research_duplicate_log where id = p_id and decision = 'pending' for update skip locked;
  if not found then return 'skipped'; end if;
  em := lower(btrim(coalesce(nullif(d.attempted_email, ''), d.attempted_fields->>'email', '')));
  select id into tid from research_contacts where id = d.existing_contact_id and deleted_at is null;
  if tid is null and em <> '' then
    select id into tid from research_contacts where lower(email) = em and deleted_at is null;
  end if;
  if tid is null and btrim(coalesce(d.contact_name, '')) ~ '\S+\s+\S+' and btrim(coalesce(d.school, '')) <> '' then
-- ... truncated (114 lines total)
```

### research_merge_schools(p_pairs jsonb, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 49 lines
```sql

declare
  v_batch uuid := gen_random_uuid();
  n_pairs int; n_contacts int;
begin
  create temp table _mp on commit drop as
  select distinct on ((p->>'drop')::uuid) (p->>'drop')::uuid drop_id, (p->>'keep')::uuid keep_id, coalesce(p->>'reason', '') reason
    from jsonb_array_elements(p_pairs) p
   where (p->>'drop')::uuid <> (p->>'keep')::uuid;
  update _mp a set keep_id = b.keep_id from _mp b where a.keep_id = b.drop_id;
  select count(*) into n_pairs from _mp;

  insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
  select c.id, 'school_id', m.drop_id::text, m.keep_id::text, 'merged', 'merge-schools: ' || m.reason, p_actor, v_batch
    from research_contacts c join _mp m on c.school_id = m.drop_id where c.deleted_at is null;
-- ... truncated (49 lines total)
```

### research_needs_work_schools() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 26 lines
```sql

declare r jsonb;
begin
  create temp table _n on commit drop as
    select c.id, c.school_id, lower(split_part(c.email,'@',2)) d,
      (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)) no_city,
      (btrim(coalesce(c.designation,'')) = '' or research_dup_junk(c.designation)) no_title,
      coalesce(c.school ~* '\m(district|public schools|county schools|city schools|board of education|unified|isd|usd|school corporation|department of education|ministry of education)\M', false) district
    from research_contacts c
    where c.deleted_at is null and not coalesce(c.generic_contact, false);
  select jsonb_build_object('kind', 'needs_work_schools',
    'no_city_contacts', count(*) filter (where no_city),
    'no_city_schools', count(distinct school_id) filter (where no_city),
    'no_city_no_school_id', count(*) filter (where no_city and school_id is null),
    'no_city_domains', count(distinct d) filter (where no_city),
    'no_city_domains_with_site', (select count(*) from research_domain_sites s where s.url <> '' and s.domain in (select distinct d from _n where no_city)),
    'no_title_contacts', count(*) filter (where no_title),
    'no_title_schools', count(distinct school_id) filter (where no_title),
    'district_contacts', count(*) filter (where district),
    'district_domains', count(distinct d) filter (where district),
    'open_tasks', (select count(*) from research_tasks where status in ('open','queued','pending')),
    'open_task_schools_with_no_city_contacts', (select count(distinct t.school_id) from research_tasks t where t.status in ('open','queued','pending') and t.school_id in (select school_id from _n where no_city)))
  into r from _n;
  insert into research_data_health (summary) values (r);
  return r;
end 
```

### research_norm_words(v text) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select btrim(regexp_replace(regexp_replace(lower(coalesce(v, '')), '[^a-z\s]', ' ', 'g'), '\s+', ' ', 'g'));

```

### research_place_junk(v text) -> boolean
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 6 lines
```sql

  select research_dup_junk(v)
      or coalesce(v, '') ~* '\mmix\M'
      or btrim(coalesce(v, '')) ~* '^(africa|asia|europe|oceania|australasia|north america|south america|latin america|middle east|gcc|apac|emea|international|global|worldwide|online|remote)$'
      or coalesce(v, '') ~* '(counsel|educator|principal|teacher|director|coordinator|superintendent|headmaster|admission|school|college|academy)'

```

### research_plan_fact_fill() -> integer
- sql | invoker | volatile | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 1 lines
```sql
 select 0 
```

### research_plan_fact_fill_next() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 18 lines
```sql

declare c record; n int;
begin
  select * into c from research_fact_fill_cols where done_at is null order by col limit 1;
  if not found then perform cron.unschedule('group-fact-fill-plan2'); return jsonb_build_object('done', true); end if;
  execute format($q$
    insert into research_fact_fill_plan (contact_id, col, value, school_id, source)
    select ct.id, %1$L, btrim(f.value), f.school_id, coalesce(nullif(f.method,''), 'verified') || ': ' || coalesce(f.source_url, '')
      from research_school_facts f
      join research_fact_fill_schools e on e.school_id = f.school_id
      join research_contacts ct on ct.school_id = f.school_id and ct.deleted_at is null
     where f.field = %2$L and f.status = 'VERIFIED' and btrim(coalesce(f.value,'')) <> ''
       and (btrim(coalesce(ct.%1$I, '')) = '' or (%1$L in ('city','state','country') and research_place_junk(ct.%1$I)))
    on conflict (contact_id, col) do nothing$q$, c.col, c.field);
  get diagnostics n = row_count;
  update research_fact_fill_cols set planned = n, done_at = now() where col = c.col;
  return jsonb_build_object('col', c.col, 'planned', n);
end 
```

### research_plan_place_fill(p_apply boolean, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 52 lines
```sql

declare b uuid := gen_random_uuid(); r jsonb; n_city int := 0; n_state int := 0;
begin
  -- Contacts on a real school email domain, keyed by school name + country + domain.
  create temp table _x on commit drop as
    select c.id, research_school_name_key(c.school) k, lower(btrim(coalesce(c.country,''))) ck, lower(split_part(c.email,'@',2)) d,
           btrim(coalesce(c.city,'')) city, btrim(coalesce(c.state,'')) state,
           (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)) no_city,
           (btrim(coalesce(c.state,'')) = '' or research_place_junk(c.state)) no_state
      from research_contacts c
     where c.deleted_at is null and btrim(coalesce(c.school,'')) <> '' and not research_dup_junk(c.school)
       and btrim(coalesce(c.country,'')) <> '' and not research_place_junk(c.country)
       and c.email like '%@%'
       and lower(split_part(c.email,'@',2)) !~ '^(gmail|googlemail|yahoo|ymail|hotmail|outlook|aol|icloud|me|live|msn|rediffmail|protonmail|zoho|gmx|mail|yandex|qq|163)\.';
  create index on _x (k, ck, d);
-- ... truncated (52 lines total)
```

### research_plan_school_links(p_apply boolean, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 57 lines
```sql

declare b uuid := gen_random_uuid(); r jsonb; v_new int := 0; v_linked int := 0;
begin
  create temp table _c on commit drop as
    select c.id, btrim(c.school) school, research_school_name_key(c.school) k,
           btrim(coalesce(c.city,'')) city, btrim(coalesce(c.state,'')) state, btrim(coalesce(c.country,'')) country,
           lower(split_part(c.email,'@',2)) d,
           (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)) no_city,
           lower(split_part(c.email,'@',2)) ~ '^(gmail|googlemail|yahoo|ymail|hotmail|outlook|aol|icloud|me|live|msn|rediffmail|protonmail|zoho|gmx|mail|yandex|qq|163)\.' free
      from research_contacts c
     where c.deleted_at is null and c.school_id is null
       and btrim(coalesce(c.school,'')) <> '' and not research_dup_junk(c.school)
       and btrim(coalesce(c.country,'')) <> '' and not research_place_junk(c.country);
  create index on _c (k, country);
  create temp table _s on commit drop as
-- ... truncated (57 lines total)
```

### research_promotable_by_location(p_limit integer) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 13 lines
```sql

  SELECT coalesce(jsonb_agg(x ORDER BY (x->>'promotable')::int DESC), '[]'::jsonb)
    FROM (SELECT jsonb_build_object('country',t.country,'state',t.state,
                   'promotable',t.n,'schools',t.schools) AS x
            FROM (SELECT coalesce(nullif(btrim(country_raw),''),'(no country)') AS country,
                         coalesce(nullif(btrim(state_raw),''),'(no state)')     AS state,
                         count(*) AS n,
                         count(DISTINCT lower(btrim(school_name_raw))) AS schools
                    FROM quarantine.research_contacts
                   WHERE is_valid IS TRUE AND promoted_person_id IS NULL
                   GROUP BY 1,2 ORDER BY 3 DESC
                   LIMIT least(greatest(coalesce(p_limit,40),1),200)) t) s;

```

### research_reach_explore(p jsonb, p_offset integer, p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public; work_mem=128MB | exec anon=n authenticated=n | 60 lines
```sql

declare result jsonb;
begin
  execute format($q$
  with f as materialized (
    select c.school_key sk, c.school, c.country, c.state, c.city, btrim(coalesce(c.email, '')) <> '' has_email,
           c.subject, c.seniority, c.sop_priority, c.grade_band, c.school_type, c.sop_level, c.curriculum_row, c.geo_region,
           (c.school_type_confidence = 'low' or c.grade_band_confidence = 'low') low_row
      from public.research_contact_facets c where %s),
  tally as (
    select case when grouping(geo_region) = 0 then 'region' when grouping(country) = 0 then 'country'
                when grouping(state) = 0 then 'state' when grouping(city) = 0 then 'city'
                when grouping(subj) = 0 then 'subject' when grouping(seniority) = 0 then 'seniority'
                when grouping(prio) = 0 then 'priority' else 'grade_contacts' end facet,
           coalesce(geo_region, country, state, city, subj, seniority, prio, grade) k, count(*) n
-- ... truncated (60 lines total)
```

### research_reach_filtered(p jsonb) -> SETOF research_contacts
- sql | invoker | stable | config: search_path=public | exec anon=n authenticated=n | 24 lines
```sql

  select c.* from public.research_contacts c
   where c.deleted_at is null
     and c.person_type = 'staff'
     and c.category = coalesce(nullif(p->>'category', ''), 'school')
     and (coalesce(p->>'region', '') = '' or coalesce(c.geo_region, '') = p->>'region')
     and (coalesce(p->>'country', '') = '' or lower(btrim(c.country)) = lower(btrim(p->>'country')))
     and (coalesce(p->>'state', '') = '' or lower(btrim(c.state)) = lower(btrim(p->>'state')))
     and (coalesce(p->>'city', '') = '' or lower(btrim(c.city)) = lower(btrim(p->>'city')))
     and (coalesce(p->>'school', '') = '' or lower(btrim(c.school)) = lower(btrim(p->>'school')))
     and (coalesce(p->>'curriculum', '') = '' or c.curriculum_row = p->>'curriculum')
     and (coalesce(p->>'subject', '') = '' or coalesce(nullif(c.subject, ''), 'Not set') = p->>'subject')
     and (coalesce(p->>'priority', '') = ''
          or (p->>'priority' = 'unmatched' and coalesce(c.sop_priority, '') = '')
          or (p->>'priority' <> 'unmatched' and coalesce(c.sop_priority, '') = p->>'priority'))
     and (coalesce(p->>'gradeLevel', '') = '' or coalesce(nullif(c.grade_band, ''), 'unknown') = p->>'gradeLevel')
     and (coalesce(p->>'schoolType', '') = '' or coalesce(nullif(c.school_type, ''), 'unknown') = p->>'schoolType')
     and (coalesce(p->>'seniority', '') = '' or coalesce(c.seniority, '') = p->>'seniority')
     and (coalesce(p->>'quality', '') = ''
          or (p->>'quality' = 'unknown_type' and coalesce(nullif(c.school_type, ''), 'unknown') = 'unknown')
          or (p->>'quality' = 'low_confidence' and (c.school_type_confidence = 'low' or c.grade_band_confidence = 'low'))
          or (p->>'quality' = 'unmatched_title' and coalesce(c.sop_priority, '') = '')
          or (p->>'quality' = 'no_city' and btrim(coalesce(c.city, '')) = ''));

```

### research_reach_rows(p jsonb, p_schools text[], p_offset integer, p_limit integer) -> SETOF research_contacts
- plpgsql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 7 lines
```sql

begin
  return query execute format(
    'select c.* from public.research_contacts c where %s and ($1 is null or lower(btrim(c.school)) = any ($1)) order by c.id offset $2 limit $3',
    public.research_reach_where(p))
  using p_schools, p_offset, least(greatest(p_limit, 1), 5000);
end 
```

### research_reach_slim(p jsonb) -> TABLE(school text, country text, state text, city text, email text, subject text, seniority text, sop_priority text, grade_band text, school_type text, school_type_confidence text, grade_band_confidence text, sop_level text, curriculum_row text, geo_region text)
- sql | invoker | stable | config: search_path=public | exec anon=n authenticated=n | 26 lines
```sql

  select c.school, c.country, c.state, c.city, c.email, c.subject, c.seniority, c.sop_priority, c.grade_band,
         c.school_type, c.school_type_confidence, c.grade_band_confidence, c.sop_level, c.curriculum_row, c.geo_region
    from public.research_contacts c
   where c.deleted_at is null
     and c.person_type = 'staff'
     and c.category = coalesce(nullif(p->>'category', ''), 'school')
     and (coalesce(p->>'region', '') = '' or coalesce(c.geo_region, '') = p->>'region')
     and (coalesce(p->>'country', '') = '' or lower(btrim(c.country)) = lower(btrim(p->>'country')))
     and (coalesce(p->>'state', '') = '' or lower(btrim(c.state)) = lower(btrim(p->>'state')))
     and (coalesce(p->>'city', '') = '' or lower(btrim(c.city)) = lower(btrim(p->>'city')))
     and (coalesce(p->>'school', '') = '' or lower(btrim(c.school)) = lower(btrim(p->>'school')))
     and (coalesce(p->>'curriculum', '') = '' or c.curriculum_row = p->>'curriculum')
     and (coalesce(p->>'subject', '') = '' or coalesce(nullif(c.subject, ''), 'Not set') = p->>'subject')
     and (coalesce(p->>'priority', '') = ''
          or (p->>'priority' = 'unmatched' and coalesce(c.sop_priority, '') = '')
          or (p->>'priority' <> 'unmatched' and coalesce(c.sop_priority, '') = p->>'priority'))
     and (coalesce(p->>'gradeLevel', '') = '' or coalesce(nullif(c.grade_band, ''), 'unknown') = p->>'gradeLevel')
     and (coalesce(p->>'schoolType', '') = '' or coalesce(nullif(c.school_type, ''), 'unknown') = p->>'schoolType')
     and (coalesce(p->>'seniority', '') = '' or coalesce(c.seniority, '') = p->>'seniority')
     and (coalesce(p->>'quality', '') = ''
          or (p->>'quality' = 'unknown_type' and coalesce(nullif(c.school_type, ''), 'unknown') = 'unknown')
          or (p->>'quality' = 'low_confidence' and (c.school_type_confidence = 'low' or c.grade_band_confidence = 'low'))
          or (p->>'quality' = 'unmatched_title' and coalesce(c.sop_priority, '') = '')
          or (p->>'quality' = 'no_city' and btrim(coalesce(c.city, '')) = ''));

```

### research_reach_where(p jsonb) -> text
- plpgsql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 23 lines
```sql

declare w text[] := array['c.deleted_at is null', $$c.person_type = 'staff'$$,
                          format('c.category = %L', coalesce(nullif(p->>'category', ''), 'school'))];
begin
  if coalesce(p->>'region', '') <> '' then w := w || format('c.geo_region = %L', p->>'region'); end if;
  if coalesce(p->>'country', '') <> '' then w := w || format('lower(btrim(c.country)) = lower(btrim(%L))', p->>'country'); end if;
  if coalesce(p->>'state', '') <> '' then w := w || format('lower(btrim(c.state)) = lower(btrim(%L))', p->>'state'); end if;
  if coalesce(p->>'city', '') <> '' then w := w || format('lower(btrim(c.city)) = lower(btrim(%L))', p->>'city'); end if;
  if coalesce(p->>'school', '') <> '' then w := w || format('lower(btrim(c.school)) = lower(btrim(%L))', p->>'school'); end if;
  if coalesce(p->>'curriculum', '') <> '' then w := w || format('c.curriculum_row = %L', p->>'curriculum'); end if;
  if coalesce(p->>'subject', '') <> '' then w := w || format($$coalesce(nullif(c.subject, ''), 'Not set') = %L$$, p->>'subject'); end if;
  if p->>'priority' = 'unmatched' then w := w || $$coalesce(c.sop_priority, '') = ''$$;
  elsif coalesce(p->>'priority', '') <> '' then w := w || format($$coalesce(c.sop_priority, '') = %L$$, p->>'priority'); end if;
  if coalesce(p->>'gradeLevel', '') <> '' then w := w || format($$coalesce(nullif(c.grade_band, ''), 'unknown') = %L$$, p->>'gradeLevel'); end if;
  if coalesce(p->>'schoolType', '') <> '' then w := w || format($$coalesce(nullif(c.school_type, ''), 'unknown') = %L$$, p->>'schoolType'); end if;
  if coalesce(p->>'seniority', '') <> '' then w := w || format($$coalesce(c.seniority, '') = %L$$, p->>'seniority'); end if;
  if p->>'quality' = 'unknown_type' then w := w || $$coalesce(nullif(c.school_type, ''), 'unknown') = 'unknown'$$;
  elsif p->>'quality' = 'low_confidence' then w := w || $$(c.school_type_confidence = 'low' or c.grade_band_confidence = 'low')$$;
  elsif p->>'quality' = 'unmatched_title' then w := w || $$coalesce(c.sop_priority, '') = ''$$;
  elsif p->>'quality' = 'no_city' then w := w || $$btrim(coalesce(c.city, '')) = ''$$;
  end if;
  return array_to_string(w, ' and ');
end 
```

### research_refined_pool() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 6 lines
- comment: The pool outreach may draw from, derived set-wise from the evidence ledger. Reports what is blocking it when it is empty, because a pool that silently reads zero looks like one that is still filling.
```sql

BEGIN
  IF auth_staff_id() IS NULL AND coalesce(auth.role(), '') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  RETURN research_refined_pool_body();
END 
```

### research_refined_pool_body() -> jsonb
- sql | invoker | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 70 lines
```sql

  WITH w AS (SELECT (email_check_windows()->>'positive_check_valid_days')::int AS fresh_days),
  dead_dom AS (
    SELECT DISTINCT domain FROM quarantine.email_domain_evidence
     WHERE (method='dns_resolve' AND result='fail')
        OR (method='domain_mx'   AND result='fail')
        OR (method='disposable'  AND result='pass')
        OR (method='parked'      AND result='pass')),
  seen_dom AS (
    SELECT DISTINCT domain FROM quarantine.email_domain_evidence WHERE method='dns_resolve'),
  soft_dom AS (
    SELECT DISTINCT domain FROM quarantine.email_domain_evidence
     WHERE result='pass' AND method IN ('freemail','catch_all')),
  dead_addr AS (
    SELECT DISTINCT email_norm FROM quarantine.email_evidence
-- ... truncated (70 lines total)
```

### research_refresh_all_contributions() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 9 lines
```sql

declare t uuid; n int := 0;
begin
  for t in select id from research_upload_tabs where target_lane = 'contact' loop
    n := n + research_refresh_contributions(t);
  end loop;
  update research_stats_contributions_meta set computed_at = now() where id = 1;
  return n;
end 
```

### research_refresh_contrib_chunk(p_tab uuid, p_chunk integer) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 63 lines
```sql

declare t record; rcol text; tcol text; now_ts timestamptz := clock_timestamp(); n int;
        def_ids uuid[]; def_tool text; lo int := p_chunk * 20000; hi int := p_chunk * 20000 + 19999;
begin
  select ut.*, u.researcher_ids up_ids into t from research_upload_tabs ut join research_uploads u on u.id = ut.upload_id where ut.id = p_tab;
  if t is null then return 0; end if;
  select k into rcol from jsonb_each_text(t.mapping) e(k, v) where v = 'researcher' limit 1;
  select k into tcol from jsonb_each_text(t.mapping) e(k, v) where v = 'research_tool' limit 1;
  def_ids := coalesce(array(select jsonb_array_elements_text(coalesce((t.mapping->>'__attribution')::jsonb->'researcherIds','[]'))::uuid), '{}');
  if cardinality(def_ids) = 0 then def_ids := coalesce(t.up_ids, '{}'); end if;
  def_tool := t.default_research_tool;

  insert into research_stats_contributions (upload_tab_id, chunk_no, researcher_label, researcher_staff_id, researcher_basis, tool_key,
    researched_month, outcome, rows_n, clean_now, email_ready_now, bad_email_now, ai_rows, computed_at)
  with r as (
-- ... truncated (63 lines total)
```

### research_refresh_contributions(p_tab uuid) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 58 lines
```sql

declare t record; rcol text; tcol text; now_ts timestamptz := clock_timestamp(); n int;
        def_ids uuid[]; def_tool text;
begin
  select ut.*, u.researcher_ids up_ids into t from research_upload_tabs ut join research_uploads u on u.id = ut.upload_id where ut.id = p_tab;
  if t is null then return 0; end if;
  select k into rcol from jsonb_each_text(t.mapping) e(k, v) where v = 'researcher' limit 1;
  select k into tcol from jsonb_each_text(t.mapping) e(k, v) where v = 'research_tool' limit 1;
  def_ids := coalesce(array(select jsonb_array_elements_text(coalesce((t.mapping->>'__attribution')::jsonb->'researcherIds','[]'))::uuid), '{}');
  if cardinality(def_ids) = 0 then def_ids := coalesce(t.up_ids, '{}'); end if;
  def_tool := t.default_research_tool;

  insert into research_stats_contributions (upload_tab_id, researcher_label, researcher_staff_id, researcher_basis, tool_key,
    researched_month, outcome, rows_n, clean_now, email_ready_now, ai_rows, computed_at)
  with r as (
-- ... truncated (58 lines total)
```

### research_refresh_domain_index() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 29 lines
```sql

declare v_names int; v_known int; v_gone int;
begin
  create temp table _d on commit drop as
  select c.id, btrim(coalesce(c.school, '')) school,
         coalesce(nullif(lower(btrim(split_part(c.email, '@', 2))), ''), lower(btrim(coalesce(c.domain, '')))) d
    from research_contacts c
   where c.deleted_at is null;
  create temp table _k on commit drop as select d, count(*)::int n from _d where d <> '' group by d;
  create temp table _n on commit drop as
  select d, school, count(*)::int n, (array_agg(id order by id))[1:3] ids from _d where d <> '' and school <> '' group by d, school;

  insert into research_domain_known as t (domain, n, refreshed_at)
  select d, n, now() from _k
  on conflict (domain) do update set n = excluded.n, refreshed_at = excluded.refreshed_at where t.n is distinct from excluded.n;
  get diagnostics v_known = row_count;
  update research_domain_known t set n = 0, refreshed_at = now() where t.n > 0 and not exists (select 1 from _k where _k.d = t.domain);

  insert into research_domain_names as t (domain, school, n, ids, refreshed_at)
  select d, school, n, ids, now() from _n
  on conflict (domain, school) do update set n = excluded.n, ids = excluded.ids, refreshed_at = excluded.refreshed_at
   where t.n is distinct from excluded.n;
  get diagnostics v_names = row_count;
  update research_domain_names t set n = 0, refreshed_at = now()
   where t.n > 0 and not exists (select 1 from _n where _n.d = t.domain and _n.school = t.school);
  get diagnostics v_gone = row_count;

  return jsonb_build_object('domains_changed', v_known, 'pairs_changed', v_names, 'pairs_gone', v_gone);
end 
```

### research_refresh_email_check_stats() -> void
- sql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 13 lines
```sql

  insert into research_stats_email_checks (id, ready, risky, do_not_send, unchecked, refreshed_at)
  select 1,
         count(*) filter (where k.status = 'ready'),
         count(*) filter (where k.status = 'risky'),
         count(*) filter (where k.status = 'do_not_send'),
         count(*) filter (where k.email is null or k.recheck_requested),
         now()
    from (select distinct email from research_email_targets) e
    left join research_email_checks k using (email)
  on conflict (id) do update set ready = excluded.ready, risky = excluded.risky, do_not_send = excluded.do_not_send,
    unchecked = excluded.unchecked, refreshed_at = excluded.refreshed_at;

```

### research_refresh_folder_slices(p_folder uuid, p_from integer, p_to integer) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 9 lines
```sql

declare i int; total int := 0;
begin
  for i in p_from .. least(p_to, 255) loop
    total := total + research_refresh_folders_slice(lpad(to_hex(i), 2, '0'));
  end loop;
  update research_folders set refreshed_at = now() where id = p_folder and p_to >= 255;
  return total;
end 
```

### research_refresh_folders_slice(p_prefix text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 46 lines
```sql

declare lo uuid := (rpad(p_prefix, 8, '0') || '-0000-0000-0000-000000000000')::uuid;
        hi uuid := (rpad(p_prefix, 8, 'f') || '-ffff-ffff-ffff-ffffffffffff')::uuid;
        f record; rx text; subj text[]; ids uuid[]; total int := 0; k int;
        lim_countries text[]; lim_states text[]; lim_curricula text[]; lim_grades text[]; lim_types text[]; lim_roles text[]; lim_sen text[];
begin
  for f in select * from research_folders where not archived
             and (jsonb_array_length(coalesce(rules->'title_any','[]')) > 0 or jsonb_array_length(coalesce(rules->'subjects_any','[]')) > 0) loop
    rx := research_folder_title_regex(f.rules->'title_any');
    subj := array(select jsonb_array_elements_text(coalesce(f.rules->'subjects_any','[]')));
    lim_countries := array(select lower(x) from jsonb_array_elements_text(coalesce(f.rules->'countries','[]')) x);
    lim_states := array(select lower(x) from jsonb_array_elements_text(coalesce(f.rules->'states','[]')) x);
    lim_curricula := array(select lower(x) from jsonb_array_elements_text(coalesce(f.rules->'curricula','[]')) x);
    lim_grades := array(select x from jsonb_array_elements_text(coalesce(f.rules->'grade_bands','[]')) x);
    lim_types := array(select x from jsonb_array_elements_text(coalesce(f.rules->'school_types','[]')) x);
-- ... truncated (46 lines total)
```

### research_refresh_health_slice(p_prefix text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 62 lines
```sql

declare lo uuid := (rpad(p_prefix, 8, '0') || '-0000-0000-0000-000000000000')::uuid;
        hi uuid := (rpad(p_prefix, 8, 'f') || '-ffff-ffff-ffff-ffffffffffff')::uuid;
        n int;
begin
  with b as (
    select c.id, c.deleted_at is not null deleted,
      lower(btrim(coalesce(c.country,''))) country_l,
      (btrim(coalesce(c.contact_name,'')) = '' or research_dup_junk(c.contact_name)) no_name,
      coalesce(c.contact_name ~ '[/&]|\sand\s', false) two_people,
      (btrim(coalesce(c.designation,'')) = '' or research_dup_junk(c.designation)) no_title,
      (c.school_id is null) no_school_id,
      (btrim(coalesce(c.school,'')) = '' or research_dup_junk(c.school)) no_school,
      coalesce(c.school ~* '\m(district|public schools|county schools|city schools|board of education|unified|isd|usd|school corporation|department of education|ministry of education)\M', false) district_as_school,
      (btrim(coalesce(c.city,'')) = '' or research_place_junk(c.city)) no_city,
-- ... truncated (62 lines total)
```

## Schema: public - part 4 of 4

### research_refresh_trust_totals() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 16 lines
```sql

declare v_rows int;
begin
  create temp table _tt on commit drop as
  select coalesce(f.trust_state, 'unchecked') st, ''::text rs, count(*) cnt from research_contact_facets f where f.deleted_at is null group by 1
  union all
  select 'quarantined', r, count(*) from research_contact_facets f, unnest(f.trust_reasons) r
   where f.deleted_at is null and f.trust_state = 'quarantined' group by 2;
  insert into research_trust_totals_cache as c (state, reason, n, computed_at)
  select st, rs, cnt, now() from _tt
  on conflict (state, reason) do update set n = excluded.n, computed_at = excluded.computed_at;
  update research_trust_totals_cache c set n = 0, computed_at = now()
   where not exists (select 1 from _tt t where t.st = c.state and t.rs = c.reason);
  select count(*) into v_rows from _tt;
  return v_rows;
end 
```

### research_registry_grade(g integer) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select case when g is null then '' when g = -1 then 'PK' when g = 0 then 'K' when g between 1 and 12 then g::text else '' end;

```

### research_restore_review_batch(p_batch uuid) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 11 lines
```sql

declare n int;
begin
  with back as (delete from research_review_outcomes where batch = p_batch and outcome = 'archived'
                returning id, dataset_id, dataset_code, line, fields, reason, reason_bucket, source, origin, queued_by, queued_at)
  insert into research_review_rows (id, dataset_id, dataset_code, line, fields, reason, reason_bucket, source, origin, queued_by, queued_at, state, missing, autofilled)
  select id, dataset_id, dataset_code, line, fields, reason, reason_bucket, source, origin, queued_by, coalesce(queued_at, now()), 'hold', '{}', '{}' from back
  on conflict (id) do nothing;
  get diagnostics n = row_count;
  return n;
end 
```

### research_resume_billing_blocked(p_limit integer, p_delay_minutes integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 42 lines
- comment: Manual-only bounded resume for tasks parked because OpenAI credits were exhausted. Use only after credits are restored.
```sql

declare
  v_resumed integer := 0;
begin
  if p_limit < 1 or p_limit > 25 then
    raise exception 'p_limit must be between 1 and 25';
  end if;
  if p_delay_minutes < 5 or p_delay_minutes > 240 then
    raise exception 'p_delay_minutes must be between 5 and 240';
  end if;

  with picked as (
    select id
      from public.research_tasks
     where status='failed'
-- ... truncated (42 lines total)
```

### research_retry_tick(p_limit integer, p_base_minutes integer, p_cap_minutes integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 111 lines
- comment: Classifies billing-blocked failures, applies bounded exponential backoff to transient 429s, and releases only a small due batch. Billing-blocked tasks remain parked until credits are restored.
```sql

declare
  v_billing integer := 0;
  v_scheduled integer := 0;
  v_exhausted integer := 0;
  v_released integer := 0;
begin
  if p_limit < 1 or p_limit > 25 then
    raise exception 'p_limit must be between 1 and 25';
  end if;
  if p_base_minutes < 5 or p_base_minutes > 240 then
    raise exception 'p_base_minutes must be between 5 and 240';
  end if;
  if p_cap_minutes < p_base_minutes or p_cap_minutes > 1440 then
    raise exception 'p_cap_minutes must be between p_base_minutes and 1440';
-- ... truncated (111 lines total)
```

### research_review_archive_unusable(p_limit integer, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 29 lines
```sql

declare b uuid := gen_random_uuid(); a int := 0; n int := 0; ids uuid[];
begin
  -- A: personal mailbox and no school — nothing on the row says which school.
  select array_agg(id) into ids from (
    select id from research_review_rows
     where (btrim(coalesce(fields->>'school_name', '')) = '' or research_dup_junk(fields->>'school_name'))
       and lower(split_part(btrim(fields->>'email'), '@', 2)) ~ '^(gmail|googlemail|yahoo|ymail|hotmail|outlook|aol|icloud|me|live|msn|rediffmail|protonmail|zoho|gmx|mail|yandex|qq|163)\.'
     limit p_limit) x;
  if ids is not null then
    a := research_settle_review_rows(ids, 'archived',
      'Personal email (gmail, yahoo …) and no school name — nothing on the row says which school this person is at',
      'Free-mail domain; school cell empty or a placeholder', p_actor, b);
  end if;
  -- B: no person's name — a contact is one named person.
  select array_agg(id) into ids from (
    select id from research_review_rows
     where btrim(coalesce(fields->>'contact_name', '')) = '' or research_dup_junk(fields->>'contact_name')
     limit p_limit) x;
  if ids is not null then
    n := research_settle_review_rows(ids, 'archived',
      'No person''s name on the row — a contact has to be one named person',
      case when false then '' else 'Name cell empty or a placeholder' end, p_actor, b);
    update research_review_outcomes
       set note = note || ' (this address is a school office inbox)'
     where batch = b and research_dup_generic_inbox(fields->>'email') and note like 'No person%';
  end if;
  return jsonb_build_object('personal_no_school', a, 'no_name', n, 'batch', b);
end 
```

### research_review_autoclean() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 7 lines
```sql

begin
  return jsonb_build_object(
    'copies', research_review_merge_copies(5000, 'Review autoclean'),
    'unusable', research_review_archive_unusable(5000, 'Review autoclean'),
    'rest', research_review_settle_rest(5000, 'Review autoclean'));
end 
```

### research_review_backfill_from_raw(p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 34 lines
```sql

declare b uuid := gen_random_uuid(); r record; col text; v text; cur text; n int := 0; contacts int := 0; touched boolean;
begin
  create temp table _src on commit drop as
  with m as (
    select t.id tab_id, t.dataset_id, e.key header, e.value #>> '{}' col,
           (select k.key from jsonb_each(t.mapping) k where k.value #>> '{}' = 'email' limit 1) email_header
      from research_upload_tabs t, jsonb_each(t.mapping) e
     where t.dataset_id in (select distinct dataset_id from research_review_outcomes where contact_id is not null)
       and e.value #>> '{}' in ('website', 'phone', 'linkedin_url'))
  select o.contact_id, jsonb_object_agg(m.col, btrim(w.data->>m.header)) filter (where btrim(coalesce(w.data->>m.header, '')) <> '') add
    from research_review_outcomes o
    join m on m.dataset_id = o.dataset_id
    join research_raw_rows w on w.upload_tab_id = m.tab_id and w.row_number = o.line
   where o.contact_id is not null and o.outcome in ('filed', 'merged')
     and lower(btrim(coalesce(w.data->>m.email_header, ''))) = lower(btrim(coalesce(o.fields->>'email', '')))
   group by o.contact_id;
  for r in select s.contact_id, s.add, to_jsonb(c) cj from _src s join research_contacts c on c.id = s.contact_id and c.deleted_at is null where s.add is not null loop
    touched := false;
    foreach col in array array['website', 'phone', 'linkedin_url'] loop
      v := btrim(coalesce(r.add->>col, ''));
      cur := btrim(coalesce(r.cj->>col, ''));
      if v = '' or cur <> '' or research_dup_junk(v) then continue; end if;
      if col = 'website' and v !~* '^(https?://)?[a-z0-9.-]+\.[a-z]{2,}' then continue; end if;
      if col = 'linkedin_url' and v !~* 'linkedin\.com/' then continue; end if;
      execute format('update research_contacts set %I = $1 where id = $2', col) using v, r.contact_id;
      insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
      values (r.contact_id, col, '', v, 'filled', 'review-cleanup', p_actor, b);
      n := n + 1; touched := true;
    end loop;
    if touched then contacts := contacts + 1; end if;
  end loop;
  return jsonb_build_object('fields', n, 'contacts', contacts, 'batch', b);
end 
```

### research_review_bucket(reason text) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 7 lines
```sql

  select case
    when reason ~* 'school name' then 'No school name'
    when reason ~* 'email' then 'Broken email'
    when reason ~* 'is not a (country|state|city)' then 'Shifted columns'
    else 'Other' end;

```

### research_review_bucket_counts() -> TABLE(bucket text, n bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 4 lines
```sql

  select reason_bucket, count(*) from public.research_review_rows
   where state = 'hold' and quarantined_at is null group by 1;

```

### research_review_district_name_matches() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 18 lines
```sql

declare r jsonb;
begin
  create temp table _dr on commit drop as
    select q.id, lower(split_part(btrim(q.fields->>'email'),'@',2)) d, lower(research_dup_bare_name(q.fields->>'contact_name')) nm
      from research_review_rows q join research_review_domain_school s on s.domain = lower(split_part(btrim(q.fields->>'email'),'@',2))
     where s.total >= 3 and s.share < 0.9;
  create temp table _dc on commit drop as
    select lower(split_part(c.email,'@',2)) d, lower(research_dup_bare_name(c.contact_name)) nm, c.id, c.school
      from research_contacts c
     where c.deleted_at is null and lower(split_part(c.email,'@',2)) in (select distinct d from _dr);
  create index on _dc (d, nm);
  select jsonb_build_object(
    'district_rows', (select count(*) from _dr),
    'one_match', (select count(*) from _dr where (select count(*) from _dc where _dc.d = _dr.d and _dc.nm = _dr.nm) = 1),
    'many_matches', (select count(*) from _dr where (select count(*) from _dc where _dc.d = _dr.d and _dc.nm = _dr.nm) > 1)) into r;
  return r;
end 
```

### research_review_enrich_from_raw() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 22 lines
```sql

declare v_rows int;
begin
  with m as (
    select t.id tab_id, t.dataset_id, e.key header, e.value #>> '{}' col,
           (select k.key from jsonb_each(t.mapping) k where k.value #>> '{}' = 'email' limit 1) email_header
      from research_upload_tabs t, jsonb_each(t.mapping) e
     where t.dataset_id in (select distinct dataset_id from research_review_rows)
       and e.value #>> '{}' in ('website', 'phone', 'linkedin_url', 'curriculum', 'grade', 'fee_standard')),
  extra as (
    select q.id, jsonb_object_agg(m.col, btrim(w.data->>m.header)) filter (where btrim(coalesce(w.data->>m.header, '')) <> '') add
      from research_review_rows q
      join m on m.dataset_id = q.dataset_id
      join research_raw_rows w on w.upload_tab_id = m.tab_id and w.row_number = q.line
     -- the source row is this row only when its email is this row's email
     where lower(btrim(coalesce(w.data->>m.email_header, ''))) = lower(btrim(coalesce(q.fields->>'email', '')))
     group by q.id)
  update research_review_rows q set fields = extra.add || q.fields
    from extra where extra.id = q.id and extra.add is not null;
  get diagnostics v_rows = row_count;
  return v_rows;
end 
```

### research_review_merge_copies(p_limit integer, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 21 lines
```sql

declare r record; did uuid; b uuid := gen_random_uuid(); n int := 0; note text;
begin
  for r in
    select q.id, q.fields, q.dataset_id, c.id cid
      from research_review_rows q
      join research_contacts c on lower(c.email) = lower(btrim(q.fields->>'email')) and c.deleted_at is null
     where btrim(coalesce(q.fields->>'email', '')) <> ''
     limit p_limit
  loop
    insert into research_duplicate_log (attempted_email, school, contact_name, reason, existing_contact_id, dataset_id, attempted_fields, decision)
    values (lower(btrim(r.fields->>'email')), coalesce(r.fields->>'school_name', ''), coalesce(r.fields->>'contact_name', ''),
            'From Needs Fixing: this email is already a contact', r.cid, r.dataset_id, r.fields, 'pending')
    returning id into did;
    -- The insert trigger has merged it; read what it did.
    select coalesce(nullif(auto_note, ''), 'Merged onto the existing contact') into note from research_duplicate_log where id = did;
    perform research_settle_review_rows(array[r.id], 'merged', note, 'Same email as an existing contact', p_actor, b, r.cid);
    n := n + 1;
  end loop;
  return jsonb_build_object('merged', n, 'batch', b);
end 
```

### research_review_merge_district_names(p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 28 lines
```sql

declare r record; did uuid; b uuid := gen_random_uuid(); n int := 0; note text;
begin
  create temp table _dr on commit drop as
    select q.id, q.fields, q.dataset_id, lower(split_part(btrim(q.fields->>'email'),'@',2)) d, lower(research_dup_bare_name(q.fields->>'contact_name')) nm
      from research_review_rows q join research_review_domain_school s on s.domain = lower(split_part(btrim(q.fields->>'email'),'@',2))
     where s.total >= 3 and s.share < 0.9 and research_dup_bare_name(q.fields->>'contact_name') ~ '\S+\s+\S+';
  create temp table _dc on commit drop as
    select lower(split_part(c.email,'@',2)) d, lower(research_dup_bare_name(c.contact_name)) nm, c.id, c.school
      from research_contacts c
     where c.deleted_at is null and lower(split_part(c.email,'@',2)) in (select distinct d from _dr);
  create index on _dc (d, nm);
  for r in
    select _dr.*, m.id cid, m.school from _dr
      join lateral (select array_agg(_dc.id) ids, max(_dc.school) school, min(_dc.id::text)::uuid id from _dc where _dc.d = _dr.d and _dc.nm = _dr.nm) m on cardinality(m.ids) = 1
  loop
    insert into research_duplicate_log (attempted_email, school, contact_name, reason, existing_contact_id, dataset_id, attempted_fields, decision)
    values (lower(btrim(r.fields->>'email')), r.school, coalesce(r.fields->>'contact_name', ''),
            'From Needs Fixing: same name on the same district domain as one existing contact', r.cid, r.dataset_id,
            r.fields || jsonb_build_object('school_name', ''), 'pending')
    returning id into did;
    select coalesce(nullif(auto_note, ''), 'Merged onto the existing contact') into note from research_duplicate_log where id = did;
    perform research_settle_review_rows(array[r.id], 'merged', note,
      'Same full name on the same email domain as exactly one existing contact (' || r.school || ')', p_actor, b, r.cid);
    n := n + 1;
  end loop;
  return jsonb_build_object('merged', n, 'batch', b);
end 
```

### research_review_page(p_bucket text, p_limit integer) -> SETOF research_review_rows
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 7 lines
```sql

  select * from public.research_review_rows
   where state = 'hold' and quarantined_at is null
     and (p_bucket is null or p_bucket = '' or reason_bucket = p_bucket)
   order by queued_at desc
   limit least(greatest(p_limit, 1), 1000);

```

### research_review_queue(p_bucket text, p_country text, p_state text, p_city text, p_q text, p_limit integer, p_offset integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 25 lines
```sql

DECLARE me uuid; b text; lim int; off int; researcher_only boolean;
BEGIN
  me := auth_staff_id();
  IF me IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;

  b := CASE coalesce(p_bucket, 'ready')
         WHEN 'promotable'      THEN 'ready'
         WHEN 'never_validated' THEN 'awaiting_check'
         WHEN 'invalid'         THEN 'removed'
         ELSE coalesce(p_bucket, 'ready')
       END;

  IF b NOT IN ('ready','needs_school','awaiting_check','removed','unnamed','no_email',
               'promoted','reviewed') THEN
    RAISE EXCEPTION 'Unknown bucket %', p_bucket USING ERRCODE='22023'; END IF;

  lim := least(greatest(coalesce(p_limit, 25), 1), 100);
  off := greatest(coalesce(p_offset, 0), 0);
  researcher_only := has_role('researcher')
                     AND NOT (has_role('owner') OR has_role('admin') OR has_role('manager'));

  RETURN research_review_queue_body(b, p_country, p_state, p_city, p_q, lim, off,
                                    me, researcher_only);
END 
```

### research_review_queue_body(p_bucket text, p_country text, p_state text, p_city text, p_q text, p_limit integer, p_offset integer, p_staff uuid, p_researcher_only boolean) -> jsonb
- sql | invoker | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 81 lines
```sql

  WITH dead_dom AS (
    SELECT DISTINCT domain FROM quarantine.email_domain_evidence
     WHERE (method='dns_resolve' AND result='fail')
        OR (method='domain_mx'   AND result='fail')
        OR (method='disposable'  AND result='pass')
        OR (method='parked'      AND result='pass')),
  seen_dom AS (
    SELECT DISTINCT domain FROM quarantine.email_domain_evidence WHERE method='dns_resolve'),
  dead_addr AS (
    SELECT DISTINCT email_norm FROM quarantine.email_evidence
     WHERE method IN ('bounce_hard','complaint','trap_hit')
        OR (method='plausibility' AND result='fail')
        OR (method='disposable'   AND result='pass')),
  live_addr AS (
-- ... truncated (81 lines total)
```

### research_review_raw_stats() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 17 lines
```sql

declare r jsonb;
begin
  create temp table _x on commit drop as
    select q.id, lower(split_part(btrim(q.fields->>'email'),'@',2)) d,
           lower(regexp_replace(regexp_replace(btrim(coalesce(w.data->>'Website','')), '^https?://(www\.)?', '', 'i'), '[/?#].*$', '')) host
      from research_review_rows q
      join research_upload_tabs t on t.dataset_id = q.dataset_id
      join research_raw_rows w on w.upload_tab_id = t.id and w.row_number = q.line;
  select jsonb_build_object(
    'joined', count(*),
    'with_website', count(*) filter (where host <> ''),
    'website_other_than_email_domain', count(*) filter (where host <> '' and host <> d and host not like '%.' || d and d not like '%.' || host),
    'district_rows_with_school_site', count(*) filter (where host <> '' and host <> d and exists (select 1 from research_review_domain_school s where s.domain = _x.d and s.total >= 3 and s.share < 0.9)))
  into r from _x;
  return r;
end 
```

### research_review_settle_rest(p_limit integer, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 46 lines
```sql

declare b uuid := gen_random_uuid(); v_arch int := 0; v_lead int := 0;
begin
  create temp table _rest on commit drop as
  select q.id, lower(split_part(btrim(q.fields->>'email'),'@',2)) d, q.fields,
         btrim(coalesce(q.fields->>'email','')) !~* '^[^\s@]+@[^\s@]+\.[a-z]{2,}$' bad_email,
         p.total, p.share, p.school p_school, s.url s_url, s.ai_kind, s.kind s_kind, coalesce(nullif(s.ai_name,''), s.school_name) s_name
    from (select * from research_review_rows where queued_at < now() - interval '2 days' limit p_limit) q
    left join research_review_domain_school p on p.domain = lower(split_part(btrim(q.fields->>'email'),'@',2))
    left join research_domain_sites s on s.domain = lower(split_part(btrim(q.fields->>'email'),'@',2));

  -- An email that is not an email, and could not be repaired: no way to reach the person.
  with moved as (delete from research_review_rows q using _rest r where q.id = r.id and r.bad_email
                 returning q.*)
  insert into research_review_outcomes (id, dataset_id, dataset_code, line, fields, reason, reason_bucket, source, origin, queued_by, queued_at, outcome, note, evidence, batch, settled_by)
-- ... truncated (46 lines total)
```

### research_role(_user_id uuid) -> text
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=Y authenticated=Y | 3 lines
```sql

  select role from public.research_staff_profiles where user_id = _user_id and active limit 1

```

### research_schema_ddl(p_part integer) -> text
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, pg_catalog | exec anon=n authenticated=n | 140 lines
```sql

declare
  tabs oid[]; fns oid[]; prev int := -1; out text := ''; r record; s text; deferred text := '';
  fn_names text[];
begin
  -- Tables: every research_* table plus the support tables research code depends on.
  tabs := array(select c.oid from pg_class c where c.relnamespace = 'public'::regnamespace and c.relkind in ('r','p')
                  and (c.relname like 'research\_%' or c.relname in ('staff','staff_role_assignments','allocations','people','person_identities','affiliations','organizations','org_groups'))
                order by c.relname);
  -- Functions: research_*, trigger functions of those tables, and everything they call (closure).
  fns := array(select p.oid from pg_proc p where p.pronamespace = 'public'::regnamespace and p.prokind in ('f','p')
                 and (p.proname like 'research%' or p.oid in (select t.tgfoid from pg_trigger t where t.tgrelid = any(tabs) and not t.tgisinternal)));
  while cardinality(fns) <> prev loop
    prev := cardinality(fns);
    s := (select string_agg(pg_get_functiondef(x), ' ') from unnest(fns) x)
-- ... truncated (140 lines total)
```

### research_schema_export_try() -> text
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 25 lines
```sql

declare src text; msg text; ctx text;
begin
  select sql into src from research_schema_export_test where id = 1;
  begin
    execute 'create schema zzx_export_test';
    perform set_config('search_path', 'zzx_export_test, public, extensions', true);
    execute replace(replace(src, 'public.', 'zzx_export_test.'), 'set check_function_bodies = off;', 'set local check_function_bodies = off;');
    select format('OK tables=%s views=%s functions=%s indexes=%s triggers=%s policies=%s types=%s',
      (select count(*) from pg_class where relnamespace = 'zzx_export_test'::regnamespace and relkind = 'r'),
      (select count(*) from pg_class where relnamespace = 'zzx_export_test'::regnamespace and relkind = 'v'),
      (select count(*) from pg_proc where pronamespace = 'zzx_export_test'::regnamespace),
      (select count(*) from pg_class where relnamespace = 'zzx_export_test'::regnamespace and relkind = 'i'),
      (select count(*) from pg_trigger t join pg_class c on c.oid = t.tgrelid where c.relnamespace = 'zzx_export_test'::regnamespace and not t.tgisinternal),
      (select count(*) from pg_policies where schemaname = 'zzx_export_test'),
      (select count(*) from pg_type where typnamespace = 'zzx_export_test'::regnamespace and typtype = 'e')) into msg;
    raise exception 'ROLLBACK:%', msg;
  exception when others then
    get stacked diagnostics ctx = pg_exception_context;
    msg := sqlerrm;
    if msg not like 'ROLLBACK:%' then msg := 'FAILED: ' || msg || ' | ' || left(ctx, 600); else msg := substr(msg, 10); end if;
  end;
  update research_schema_export_test set result = msg where id = 1;
  return msg;
end 
```

### research_school_name_key(n text) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 4 lines
```sql

  select btrim(regexp_replace(regexp_replace(regexp_replace(regexp_replace(
    lower(coalesce(n, '')), '&', ' and ', 'g'), '\msaint\M', 'st', 'g'), '[^a-z0-9]+', ' ', 'g'), '\s+', ' ', 'g'));

```

### research_school_name_key2(n text) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 5 lines
```sql

  select btrim(regexp_replace(regexp_replace(research_school_name_key(n),
    '\m(school|the|senior|sr|jr|junior|elem|elementary|middle|high|academy|center|centre|of)\M', ' ', 'g'), '\s+', ' ', 'g'))
     || '|' || coalesce((regexp_match(research_school_name_key(n), '\m(elementary|elem|middle|high|academy)\M'))[1], '');

```

### research_school_name_key3(n text) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 3 lines
```sql

  select btrim(regexp_replace(regexp_replace(research_school_name_key(n), '\m(school|the|senior|sr)\M', ' ', 'g'), '\s+', ' ', 'g'));

```

### research_schools_page(p_q text, p_country text, p_only_valid boolean, p_include_providers boolean, p_limit integer, p_offset integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 39 lines
```sql

DECLARE t text; ctry text; ov boolean; inc boolean; lim int; off_ int; res jsonb;
BEGIN
  IF public.auth_staff_id() IS NULL THEN
    RAISE EXCEPTION 'Only a staff member may read the research queue';
  END IF;
  t := nullif(lower(btrim(coalesce(p_q, ''))), '');
  IF t IS NOT NULL AND length(t) < 2 THEN t := NULL; END IF;
  ctry := nullif(lower(btrim(coalesce(p_country, ''))), '');
  ov := coalesce(p_only_valid, false); inc := coalesce(p_include_providers, false);
  lim := least(greatest(coalesce(p_limit, 50), 1), 200);
  off_ := greatest(coalesce(p_offset, 0), 0);

  WITH hit AS (
    SELECT rs.* FROM quarantine.research_schools rs
     WHERE rs.organization_id IS NULL
       AND rs.rejected_at IS NULL
       AND (inc OR NOT rs.is_provider)
       AND (t IS NULL OR rs.name_key % t)
       AND (ctry IS NULL OR lower(btrim(coalesce(rs.country_raw, ''))) = ctry)
       AND (NOT ov OR rs.valid_contacts > 0)
  )
  SELECT jsonb_build_object(
    'total', (SELECT count(*) FROM hit),
    'rows', coalesce((SELECT jsonb_agg(jsonb_build_object(
        'name_key', name_key, 'name', name, 'country', country_raw, 'city', city_raw,
        'contacts', contacts, 'valid_contacts', valid_contacts,
        'country_variants', country_variants, 'country_agreement', country_agreement,
        'is_provider', is_provider, 'triage_reason', triage_reason)
        ORDER BY contacts DESC, name)
      FROM (SELECT * FROM hit ORDER BY contacts DESC, name LIMIT lim OFFSET off_) p), '[]'::jsonb),
    'countries', coalesce((SELECT jsonb_agg(jsonb_build_object('country', c, 'schools', n) ORDER BY n DESC)
      FROM (SELECT coalesce(nullif(btrim(country_raw), ''), 'unknown') c, count(*) n
              FROM quarantine.research_schools
             WHERE organization_id IS NULL AND rejected_at IS NULL AND NOT is_provider
             GROUP BY 1 ORDER BY n DESC LIMIT 15) x), '[]'::jsonb))
  INTO res;
  RETURN res;
END 
```

### research_set_derived(p_column text, p_map jsonb) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 17 lines
```sql

declare n int;
begin
  if p_column = 'geo_region' then
    update public.research_contacts c set geo_region = m.value #>> '{}'
      from jsonb_each(p_map) m
     where lower(btrim(coalesce(c.country, ''))) = m.key and c.geo_region is distinct from (m.value #>> '{}');
  elsif p_column = 'seniority' then
    update public.research_contacts c set seniority = m.value #>> '{}'
      from jsonb_each(p_map) m
     where lower(btrim(coalesce(c.designation, ''))) = m.key and c.seniority is distinct from (m.value #>> '{}');
  else
    raise exception 'unknown column %', p_column;
  end if;
  get diagnostics n = row_count;
  return n;
end 
```

### research_set_derived_batch(p_column text, p_map jsonb, p_limit integer) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 15 lines
```sql

declare n int;
begin
  if p_column = 'geo_region' then
    update public.research_contacts c set geo_region = coalesce(p_map ->> lower(btrim(coalesce(c.country, ''))), '')
     where c.id in (select id from public.research_contacts where geo_region is null limit p_limit);
  elsif p_column = 'seniority' then
    update public.research_contacts c set seniority = coalesce(p_map ->> lower(btrim(coalesce(c.designation, ''))), '')
     where c.id in (select id from public.research_contacts where seniority is null limit p_limit);
  else
    raise exception 'unknown column %', p_column;
  end if;
  get diagnostics n = row_count;
  return n;
end 
```

### research_set_raw_outcomes(p_tab uuid, p_rows jsonb) -> integer
- sql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 12 lines
```sql

  with v as (
    select (x->>'n')::int n, x->>'outcome' outcome, nullif(x->>'reason', '') reason,
           nullif(x->>'contact_id', '')::uuid contact_id
      from jsonb_array_elements(p_rows) x
  ), u as (
    update public.research_raw_rows r
       set outcome = v.outcome, reason = v.reason, contact_id = v.contact_id, claimed_at = null
      from v where r.upload_tab_id = p_tab and r.row_number = v.n
    returning 1
  ) select count(*)::int from u;

```

### research_settle_review_many(p_rows jsonb, p_outcome text, p_actor text, p_batch uuid) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 19 lines
```sql

declare v_rows int;
begin
  with x as (
    select (e->>'id')::uuid id, nullif(e->>'contact_id', '')::uuid contact_id,
           coalesce(e->>'note', '') note, coalesce(e->>'evidence', '') evidence
      from jsonb_array_elements(p_rows) e),
  moved as (
    delete from research_review_rows q using x where q.id = x.id
    returning q.id, q.dataset_id, q.dataset_code, q.line, q.fields, q.reason, q.reason_bucket, q.source, q.origin, q.queued_by, q.queued_at)
  insert into research_review_outcomes (id, dataset_id, dataset_code, line, fields, reason, reason_bucket, source, origin, queued_by, queued_at, outcome, note, evidence, contact_id, batch, settled_by)
  select m.id, m.dataset_id, coalesce(m.dataset_code, ''), coalesce(m.line, 0), m.fields, coalesce(m.reason, ''), coalesce(m.reason_bucket, ''),
         coalesce(m.source, ''), coalesce(m.origin, ''), coalesce(m.queued_by, ''), m.queued_at,
         p_outcome, x.note, x.evidence, x.contact_id, p_batch, p_actor
    from moved m join x on x.id = m.id
  on conflict (id) do nothing;
  get diagnostics v_rows = row_count;
  return v_rows;
end 
```

### research_settle_review_rows(p_ids uuid[], p_outcome text, p_note text, p_evidence text, p_actor text, p_batch uuid, p_contact uuid) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 15 lines
```sql

declare n int;
begin
  with moved as (
    delete from research_review_rows where id = any(p_ids)
    returning id, dataset_id, dataset_code, line, fields, reason, reason_bucket, source, origin, queued_by, queued_at
  )
  insert into research_review_outcomes (id, dataset_id, dataset_code, line, fields, reason, reason_bucket, source, origin, queued_by, queued_at, outcome, note, evidence, contact_id, batch, settled_by)
  select id, dataset_id, coalesce(dataset_code, ''), coalesce(line, 0), fields, coalesce(reason, ''), coalesce(reason_bucket, ''), coalesce(source, ''), coalesce(origin, ''), coalesce(queued_by, ''), queued_at,
         p_outcome, p_note, coalesce(p_evidence, ''), p_contact, p_batch, p_actor
    from moved
  on conflict (id) do nothing;
  get diagnostics n = row_count;
  return n;
end 
```

### research_sheet_facets(p_dataset uuid, p_top integer) -> TABLE(facet text, label text, n bigint, distinct_labels bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 39 lines
```sql

  with live as (
    select * from public.research_contacts
    where dataset_id = p_dataset and deleted_at is null
  ),
  raw as (
    select 'state' as facet, coalesce(nullif(trim(state), ''), 'Not set') as label, count(*) as n from live group by 2
    union all
    select 'city', coalesce(nullif(trim(city), ''), 'Not set'), count(*) from live group by 2
    union all
    select 'curriculum', coalesce(nullif(trim(curriculum), ''), 'Not set'), count(*) from live group by 2
    union all
    select 'region', coalesce(nullif(trim(region), ''), 'Not set'), count(*) from live group by 2
    union all
    select 'designation', coalesce(nullif(trim(designation), ''), 'Not set'), count(*) from live group by 2
    union all
    select 'validation',
           case when validated then 'Validated'
                when authenticity = 'generic' then 'Personal inbox'
                else 'Needs checking' end,
           count(*) from live group by 2
    union all
    select 'keep',
           case when keep_flag is true then 'KEEP'
                when keep_flag is false then 'Marked old'
                else 'Not marked' end,
           count(*) from live group by 2
    union all
    select 'school_group', coalesce(nullif(trim(g.name), ''), 'No group'), count(*)
      from live left join public.research_groups g on g.id = live.group_id group by 2
  ),
  ranked as (
    select facet, label, n,
           row_number() over (partition by facet order by n desc, label) as rn,
           count(*) over (partition by facet) as distinct_labels
    from raw
  )
  select facet, label, n, distinct_labels from ranked where rn <= p_top order by facet, n desc;

```

### research_sheet_people(p_dataset uuid) -> TABLE(staff_id uuid, name text, total bigint, today bigint, week bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 13 lines
```sql

  select
    c.created_by,
    coalesce(s.name, 'Unattributed'),
    count(*),
    count(*) filter (where c.created_at >= date_trunc('day', now())),
    count(*) filter (where c.created_at >= now() - interval '7 days')
  from public.research_contacts c
  left join public.research_staff_profiles s on s.id = c.created_by
  where c.dataset_id = p_dataset and c.deleted_at is null
  group by 1, 2
  order by 3 desc;

```

### research_site_host(p text) -> text
- sql | invoker | immutable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 2 lines
```sql

    select regexp_replace(lower(substring(btrim(coalesce(p,'')) from '^(?:https?://)?([^/:?#]+)')), '^www\.', '') 
```

### research_sop_compliance(p_country text, p_state text, p_city text) -> TABLE(schools_total bigint, schools_with_gap bigint, schools_no_high_priority bigint, schools_need_elementary bigint, schools_missing_elementary bigint, contacts_total bigint, contacts_high bigint, contacts_do_not_contact bigint, contacts_unmatched bigint, contacts_blank_designation bigint, contacts_missing_linkedin bigint, contacts_missing_phone bigint, contacts_missing_website bigint, contacts_missing_address bigint, contacts_missing_accreditation bigint, contacts_missing_association bigint, contacts_missing_fee bigint, contacts_missing_fee_maximum bigint, contacts_missing_direct_link bigint, contacts_missing_resources bigint, schools_type_unknown bigint, schools_band_unknown bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 46 lines
```sql

  with scoped as (
    select * from public.research_contacts
     where deleted_at is null and person_type = 'staff'
       and (p_country = '' or lower(country) = lower(p_country))
       and (p_state   = '' or lower(state)   = lower(p_state))
       and (p_city    = '' or lower(city)    = lower(p_city))
  ), per_school as (
    select
      lower(school) || '|' || lower(coalesce(country, '')) || '|' || lower(coalesce(state, '')) as key,
      case
        when bool_or(grade_band = 'k12') then 'k12'
        when count(distinct grade_band) filter (where grade_band <> 'unknown') > 1 then 'k12'
        else coalesce(max(grade_band) filter (where grade_band <> 'unknown'), 'unknown')
      end as grade_band,
-- ... truncated (46 lines total)
```

### research_sop_gaps(p_country text, p_state text, p_city text, p_top integer) -> TABLE(school text, city text, state text, country text, contacts bigint, grade_band text, school_type text, has_high_priority boolean, has_elementary boolean, needs_elementary boolean, do_not_contact bigint, blank_designation bigint, missing_linkedin bigint, blank_sop_fields bigint, total_gap_schools bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 47 lines
```sql

  with per_school as (
    select
      min(school) as school,
      min(nullif(city, '')) as city,
      min(nullif(state, '')) as state,
      min(nullif(country, '')) as country,
      count(*) as contacts,
      case
        when bool_or(grade_band = 'k12') then 'k12'
        when count(distinct grade_band) filter (where grade_band <> 'unknown') > 1 then 'k12'
        else coalesce(max(grade_band) filter (where grade_band <> 'unknown'), 'unknown')
      end as grade_band,
      coalesce(max(school_type) filter (where school_type <> 'unknown'), 'unknown') as school_type,
      bool_or(sop_priority = 'high') as has_high_priority,
-- ... truncated (47 lines total)
```

### research_sop_role_counts() -> TABLE(sop_role text, contacts bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 9 lines
```sql

  select sop_role, count(*)
    from public.research_contacts
   where deleted_at is null
     and person_type = 'staff'
     and sop_role <> ''
   group by sop_role
   order by count(*) desc;

```

### research_sop_stages(p_country text, p_state text, p_top integer) -> TABLE(school text, city text, state text, country text, contacts bigint, grade_band text, has_leadership boolean, has_elementary boolean, has_middle boolean, has_high boolean, needs_elementary boolean, needs_middle boolean, needs_high boolean, teams_needed integer, teams_covered integer, verified bigint, incomplete bigint, total_schools bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 53 lines
```sql

  with scoped as (
    select * from public.research_contacts
     where deleted_at is null and person_type = 'staff' and school <> ''
       and (p_country = '' or lower(country) = lower(p_country))
       and (p_state   = '' or lower(state)   = lower(p_state))
  ), per_school as (
    select
      lower(school) || '|' || lower(coalesce(country, '')) || '|' || lower(coalesce(state, '')) as key,
      min(school) as school, min(nullif(city, '')) as city,
      min(nullif(state, '')) as state, min(nullif(country, '')) as country,
      count(*) as contacts,
      case
        when bool_or(grade_band = 'k12') then 'k12'
        when count(distinct grade_band) filter (where grade_band <> 'unknown') > 1 then 'k12'
-- ... truncated (53 lines total)
```

### research_sop_territories() -> TABLE(country text, state text, schools bigint, contacts bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 13 lines
```sql

  select
    coalesce(nullif(country, ''), 'Not recorded'),
    coalesce(nullif(state, ''), 'Not recorded'),
    count(distinct lower(school)) filter (where school <> ''),
    count(*)
  from public.research_contacts
  where deleted_at is null and person_type = 'staff'
  group by 1, 2
  having count(*) > 0
  order by count(*) desc
  limit 300;

```

### research_sop_unknown_designations(p_top integer) -> TABLE(designation text, n bigint, schools bigint, sample_school text, sample_country text, distinct_unknown bigint, total_unknown_rows bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 21 lines
```sql

  with raw as (
    select lower(trim(designation)) as designation,
           count(*) as n,
           count(distinct lower(school)) as schools,
           min(school) as sample_school,
           min(country) as sample_country
      from public.research_contacts
     where deleted_at is null and person_type = 'staff'
       and sop_role = '' and trim(designation) <> ''
     group by 1
  ), ranked as (
    select *, row_number() over (order by n desc, designation) as rn,
              count(*) over () as distinct_unknown,
              sum(n) over () as total_unknown_rows
      from raw
  )
  select designation, n, schools, sample_school, sample_country,
         distinct_unknown, total_unknown_rows
    from ranked where rn <= p_top order by n desc, designation;

```

### research_split_school_bundles(p_limit integer, p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 75 lines
```sql

declare
  v_batch uuid := gen_random_uuid();
  n_bundles int; n_new int; n_moved int;
begin
  create temp table _o on commit drop as
  select c.school_id, split_part(lower(btrim(c.email)), '@', 2) d, count(*) n
    from research_contacts c join research_schools s on s.id = c.school_id
   where c.deleted_at is null and btrim(coalesce(s.city, '')) = '' and s.merged_into is null
     and s.name !~* '^(mixed|various|multiple|misc|other|others|n/?a|na|none|null|unknown|test|school|tbd|-+|\d+)$'
     and split_part(lower(btrim(c.email)), '@', 2) like '%.%'
     and split_part(lower(btrim(c.email)), '@', 2) !~ '^(gmail|googlemail|yahoo|ymail|hotmail|outlook|aol|icloud|me|live|msn|rediffmail|protonmail|proton|zoho|gmx|mail|yandex|qq|163|comcast|verizon|att|sbcglobal)\.'
   group by 1, 2;
  create index on _o (school_id);
  analyze _o;
-- ... truncated (75 lines total)
```

### research_split_shared_people(p_actor text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 48 lines
```sql

declare
  r record;
  new_id uuid;
  old_code text;
  n int := 0;
begin
  for r in
    select c.id, c.person_id, c.school_id, c.contact_name, c.designation, lower(trim(c.email)) as email
    from (
      select c.*, row_number() over (partition by c.person_id order by c.created_at, c.id) as rn
      from research_contacts c
      where c.deleted_at is null and c.person_id is not null
        and c.person_id in (
          select person_id from research_contacts
-- ... truncated (48 lines total)
```

### research_summary() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 22 lines
- comment: Counts for the research review screen. resolver_present is false until packages/core/identity exists. 036.
```sql

BEGIN
  IF public.auth_staff_id() IS NULL THEN
    RAISE EXCEPTION 'Only a staff member may read the research summary';
  END IF;
  RETURN jsonb_build_object(
    'contacts', jsonb_build_object(
      'total',        (SELECT count(*) FROM quarantine.research_contacts),
      'valid',        (SELECT count(*) FROM quarantine.research_contacts WHERE is_valid),
      'linked_to_org',(SELECT count(*) FROM quarantine.research_contacts WHERE organization_id IS NOT NULL),
      'promoted',     (SELECT count(*) FROM quarantine.research_contacts WHERE promoted_person_id IS NOT NULL)),
    'schools', jsonb_build_object(
      'distinct_names',    (SELECT count(*) FROM quarantine.research_schools),
      'reviewable',        (SELECT count(*) FROM quarantine.research_schools
                             WHERE organization_id IS NULL AND NOT is_provider),
      'not_schools',       (SELECT count(*) FROM quarantine.research_schools WHERE is_provider),
      'disputed_location', (SELECT count(*) FROM quarantine.research_schools WHERE country_variants > 1),
      'approved',          (SELECT count(*) FROM public.organizations),
      'candidates',        (SELECT count(*) FROM quarantine.org_candidates WHERE promoted_organization_id IS NULL),
      'promoted',          (SELECT count(*) FROM quarantine.org_candidates WHERE promoted_organization_id IS NOT NULL)),
    'resolver_present', false);
END 
```

### research_sync_school_country(p_actor text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 43 lines
```sql

declare
  n_updated int;
  merged jsonb := '{}'::jsonb;
  pairs jsonb;
begin
  create temp table _t on commit drop as
  select s.id, s.name, s.branch, s.city, lower(s.domain) domain, min(btrim(c.country)) new_country
    from research_schools s join research_contacts c on c.school_id = s.id and c.deleted_at is null
   where s.merged_into is null and btrim(coalesce(c.country, '')) <> ''
   group by s.id, s.name, s.branch, s.city, s.domain
  having count(distinct lower(btrim(c.country))) = 1
     and lower(min(btrim(c.country))) <> lower(coalesce(min(s.country), ''));

  select jsonb_agg(p) into pairs from (
-- ... truncated (43 lines total)
```

### research_tab_progress(p_tab uuid) -> TABLE(staged bigint, pending bigint, processing bigint, done bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 7 lines
```sql

  select count(*),
         count(*) filter (where outcome is null and (claimed_at is null or claimed_at < now() - interval '10 minutes')),
         count(*) filter (where outcome is null and claimed_at >= now() - interval '10 minutes'),
         count(*) filter (where outcome is not null)
    from public.research_raw_rows where upload_tab_id = p_tab;

```

### research_task_counts() -> TABLE(kind text, status text, n bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 3 lines
```sql

  select kind, status, count(*) from public.research_tasks group by 1, 2;

```

### research_tool_for(p text) -> text
- sql | invoker | stable | config: search_path=public | exec anon=Y authenticated=Y | 6 lines
```sql

  select t.key from research_source_tools t
   where t.active and (lower(regexp_replace(btrim(coalesce(p,'')), '\s+', ' ', 'g')) = any(t.aliases)
                       or lower(btrim(coalesce(p,''))) = t.key or lower(btrim(coalesce(p,''))) = lower(t.label))
   limit 1

```

### research_trust_totals() -> TABLE(state text, reason text, n bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 3 lines
```sql

  select c.state, c.reason, c.n from public.research_trust_totals_cache c where c.n > 0;

```

### research_undo_city_fill(p_domain text, p_reason text) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 17 lines
```sql

declare n int;
begin
  create temp table _b on commit drop as
    select fc.id, fc.contact_id, fc.field, fc.old_value, fc.new_value
      from research_field_changes fc join research_contacts c on c.id = fc.contact_id
     where fc.source = 'school website (read)' and fc.reverted_at is null
       and lower(split_part(c.email,'@',2)) = lower(p_domain)
       and lower(btrim(case fc.field when 'city' then c.city else c.state end)) = lower(btrim(fc.new_value));
  update research_contacts c set city = b.old_value from _b b where b.field = 'city' and c.id = b.contact_id;
  update research_contacts c set state = b.old_value from _b b where b.field = 'state' and c.id = b.contact_id;
  update research_field_changes f set reverted_at = now(), reverted_by = p_reason from _b b where f.id = b.id;
  insert into research_field_changes (contact_id, field, old_value, new_value, action, source, actor, batch_id)
  select contact_id, field, new_value, old_value, 'restored', 'undo', p_reason, gen_random_uuid() from _b;
  get diagnostics n = row_count;
  return n;
end 
```

### research_undo_provenance_batch(p_batch uuid) -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public | exec anon=n authenticated=n | 14 lines
```sql

declare n int := 0; k int;
begin
  update research_contacts c set research_tool = null, research_tool_note = '', research_tool_set_by = ''
    from research_field_changes f
   where f.batch_id = p_batch and f.field = 'research_tool' and f.contact_id = c.id and c.research_tool = f.new_value;
  get diagnostics k = row_count; n := n + k;
  update research_contacts c set researcher_ids = '{}'
    from research_field_changes f
   where f.batch_id = p_batch and f.field = 'researcher_ids' and f.contact_id = c.id and array_to_string(c.researcher_ids, ',') = f.new_value;
  get diagnostics k = row_count; n := n + k;
  update research_legacy_source_map set applied_at = null, applied_batch = null, applied_rows = null where applied_batch = p_batch;
  return n;
end 
```

### research_us_state_for(p_domain text, p_school text) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public | exec anon=Y authenticated=Y | 13 lines
```sql

declare v_code text; n int;
begin
  v_code := research_domain_state(lower(btrim(p_domain)));
  if v_code is not null then
    return jsonb_build_object('state', (select s.name from research_us_states s where s.code = upper(v_code)), 'evidence', 'the email domain is a ' || upper(v_code) || ' state school domain');
  end if;
  select count(distinct r.state), min(r.state) into n, v_code from research_registry_schools r where r.name_key = research_school_name_key(p_school);
  if n = 1 then
    return jsonb_build_object('state', (select s.name from research_us_states s where s.code = upper(v_code)), 'evidence', 'the NCES school registry lists "' || p_school || '" only in ' || upper(v_code));
  end if;
  return null;
end 
```

### research_us_state_for(p_domain text, p_school text, p_city text) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public | exec anon=Y authenticated=Y | 15 lines
```sql

declare v_code text; n int;
begin
  v_code := research_domain_state(lower(btrim(p_domain)));
  if v_code is not null then
    return jsonb_build_object('state', (select s.name from research_us_states s where s.code = upper(v_code)), 'evidence', 'the email domain is a ' || upper(v_code) || ' state school domain');
  end if;
  if btrim(coalesce(p_city, '')) = '' then return null; end if;
  select count(distinct r.state), min(r.state) into n, v_code from research_registry_schools r
   where r.name_key = research_school_name_key(p_school) and lower(btrim(r.city)) = lower(btrim(p_city));
  if n = 1 then
    return jsonb_build_object('state', (select s.name from research_us_states s where s.code = upper(v_code)), 'evidence', 'the NCES school registry lists "' || p_school || '" in ' || p_city || ', ' || upper(v_code));
  end if;
  return null;
end 
```

### research_verification_summary(p_country text, p_state text) -> TABLE(contacts_total bigint, verified bigint, unverified bigint, mismatch bigint, not_found bigint, stale bigint, no_linkedin bigint, cross_checked bigint, incomplete bigint, outreach_ready bigint, essentials_missing bigint)
- sql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=n authenticated=n | 29 lines
```sql

  with scoped as (
    select *
      from public.research_contacts
     where deleted_at is null
       and person_type = 'staff'
       and (p_country = '' or lower(country) = lower(p_country))
       and (p_state   = '' or lower(state)   = lower(p_state))
  )
  select
    count(*),
    count(*) filter (where verify_state = 'verified'),
    count(*) filter (where verify_state = 'unverified'),
    count(*) filter (where verify_state = 'mismatch'),
    count(*) filter (where verify_state = 'not_found'),
    count(*) filter (where verify_state = 'verified' and reverify_due is not null and reverify_due < current_date),
    count(*) filter (where coalesce(linkedin_url, '') = ''),
    count(*) filter (where cross_checked),
    count(*) filter (where completeness < 100),
    count(*) filter (
      where verify_state = 'verified'
        and essentials = 100
        and (reverify_due is null or reverify_due >= current_date)
        and coalesce(legal_state, 'allowed') in ('allowed', 'released')
        and coalesce(sop_priority, '') <> 'do_not_contact'
    ),
    count(*) filter (where essentials < 100)
    from scoped

```

### resolve_activities_pending(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 17 lines
```sql

DECLARE r record; placed int:=0; already int:=0; refused int:=0; res jsonb;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may resolve activities' USING ERRCODE='42501'; END IF;
  FOR r IN SELECT id FROM inbound_records
            WHERE kind='activity' AND processed_at IS NULL AND rejected_at IS NULL
            ORDER BY received_at LIMIT least(greatest(coalesce(p_limit,500),1),3000) LOOP
    BEGIN res := resolve_activity(r.id);
      IF (res->>'already_present')::boolean THEN already := already+1; ELSE placed := placed+1; END IF;
    EXCEPTION WHEN others THEN refused := refused+1;
      UPDATE inbound_records SET rejected_at=now(), rejected_reason=left(SQLERRM,500) WHERE id=r.id;
    END;
  END LOOP;
  RETURN jsonb_build_object('placed',placed,'already_present',already,'refused',refused);
END 
```

### resolve_activity(p_inbound_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 47 lines
```sql

DECLARE rec inbound_records%ROWTYPE; pay jsonb; v_type text; v_person uuid;
        v_at timestamptz; v_id uuid; v_channel channel;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may resolve an activity' USING ERRCODE='42501'; END IF;
  SELECT * INTO rec FROM inbound_records WHERE id = p_inbound_id;
  IF rec.id IS NULL THEN RAISE EXCEPTION 'No such inbound record %', p_inbound_id USING ERRCODE='P0002'; END IF;
  IF rec.kind <> 'activity' THEN
    RAISE EXCEPTION 'Inbound record % is a % claim, not an activity', p_inbound_id, rec.kind USING ERRCODE='22023'; END IF;

  pay := rec.payload; v_type := nullif(btrim(pay->>'type'),'');
  IF v_type IS NULL THEN RAISE EXCEPTION 'An activity with no type cannot be placed' USING ERRCODE='22023'; END IF;

-- ... truncated (47 lines total)
```

### resolve_meeting(p_inbound_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 238 lines
```sql

DECLARE
  rec inbound_records%ROWTYPE; pay jsonb; ai jsonb;
  v_status meeting_status; v_source meeting_source;
  v_email text; v_person uuid; v_org uuid; v_dom text; v_prog uuid; v_id uuid; v_src text;
  matched text := 'none'; item jsonb; n_mail int := 0; n_task int := 0; n_dec int := 0;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may resolve a meeting' USING ERRCODE='42501';
  END IF;
  SELECT * INTO rec FROM inbound_records WHERE id = p_inbound_id;
  IF rec.id IS NULL THEN RAISE EXCEPTION 'No such inbound record %', p_inbound_id USING ERRCODE='P0002'; END IF;
  IF rec.kind <> 'meeting' THEN
    RAISE EXCEPTION 'Inbound record % is a % claim, not a meeting', p_inbound_id, rec.kind USING ERRCODE='22023'; END IF;
-- ... truncated (238 lines total)
```

### resolve_meetings_pending(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 18 lines
```sql

DECLARE r record; ok_n int:=0; fail_n int:=0; review_n int:=0; fails jsonb:='[]'::jsonb; res jsonb;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may resolve meetings' USING ERRCODE='42501'; END IF;
  FOR r IN SELECT id FROM inbound_records
            WHERE kind='meeting' AND processed_at IS NULL AND rejected_at IS NULL
            ORDER BY received_at LIMIT least(greatest(coalesce(p_limit,100),1),1000) LOOP
    BEGIN res := resolve_meeting(r.id); ok_n := ok_n+1;
      IF (res->>'needs_review')::boolean THEN review_n := review_n+1; END IF;
    EXCEPTION WHEN others THEN fail_n := fail_n+1;
      fails := fails || jsonb_build_object('inbound_id',r.id,'error',left(SQLERRM,120));
      UPDATE inbound_records SET rejected_at=now(), rejected_reason=left(SQLERRM,500) WHERE id=r.id;
    END;
  END LOOP;
  RETURN jsonb_build_object('resolved',ok_n,'needs_review',review_n,'failed',fail_n,'failures',fails);
END 
```

### resolve_organisation(p_inbound_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 96 lines
```sql

DECLARE rec inbound_records%ROWTYPE; pay jsonb; v_name text; v_dom text; v_li text;
        v_existing uuid; v_how text := 'nothing'; v_id uuid; v_people int; v_merged boolean := false;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may resolve an organisation' USING ERRCODE='42501'; END IF;
  SELECT * INTO rec FROM inbound_records WHERE id = p_inbound_id;
  IF rec.id IS NULL THEN RAISE EXCEPTION 'No such inbound record %', p_inbound_id USING ERRCODE='P0002'; END IF;
  IF rec.kind <> 'organisation' THEN
    RAISE EXCEPTION 'Inbound record % is a % claim, not an organisation', p_inbound_id, rec.kind USING ERRCODE='22023'; END IF;

  pay := rec.payload; v_name := nullif(btrim(pay->>'name'),'');
  IF v_name IS NULL THEN RAISE EXCEPTION 'An organisation with no name cannot be reviewed' USING ERRCODE='22023'; END IF;

-- ... truncated (96 lines total)
```

### resolve_organisations_pending(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 19 lines
```sql

DECLARE r record; landed int:=0; matched int:=0; fail int:=0; fails jsonb:='[]'::jsonb; res jsonb;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may resolve organisations' USING ERRCODE='42501'; END IF;
  FOR r IN SELECT id FROM inbound_records
            WHERE kind='organisation' AND processed_at IS NULL AND rejected_at IS NULL
            ORDER BY received_at LIMIT least(greatest(coalesce(p_limit,200),1),1000) LOOP
    BEGIN res := resolve_organisation(r.id); landed := landed+1;
      IF (res->>'matched_existing')::boolean THEN matched := matched+1; END IF;
    EXCEPTION WHEN others THEN fail := fail+1;
      fails := fails || jsonb_build_object('inbound_id',r.id,'error',left(SQLERRM,160));
      UPDATE inbound_records SET rejected_at=now(), rejected_reason=left(SQLERRM,500) WHERE id=r.id;
    END;
  END LOOP;
  RETURN jsonb_build_object('landed_for_review',landed,'already_in_register',matched,
                            'failed',fail,'failures',fails);
END 
```

### resolve_person(p_inbound_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 76 lines
```sql

DECLARE
  rec inbound_records%ROWTYPE; pay jsonb;
  v_li text; v_em text; v_ph text; v_person uuid; v_hits int; v_src text;
  v_new boolean := false; n_ident int := 0;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may resolve a person' USING ERRCODE='42501';
  END IF;
  SELECT * INTO rec FROM inbound_records WHERE id = p_inbound_id;
  IF rec.id IS NULL THEN RAISE EXCEPTION 'No such inbound record %', p_inbound_id USING ERRCODE='P0002'; END IF;
  IF rec.kind <> 'person' THEN
    RAISE EXCEPTION 'Inbound record % is a % claim, not a person', p_inbound_id, rec.kind USING ERRCODE='22023'; END IF;

-- ... truncated (76 lines total)
```

### resolve_persons_pending(p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 18 lines
```sql

DECLARE r record; created int:=0; linked int:=0; fail int:=0; fails jsonb:='[]'::jsonb; res jsonb;
BEGIN
  IF auth_staff_id() IS NULL
     OR NOT (has_role('owner') OR has_role('manager') OR has_role('outreach') OR has_role('counsellor')) THEN
    RAISE EXCEPTION 'Only an owner, manager, outreach or counsellor may resolve people' USING ERRCODE='42501'; END IF;
  FOR r IN SELECT id FROM inbound_records
            WHERE kind='person' AND processed_at IS NULL AND rejected_at IS NULL
            ORDER BY received_at LIMIT least(greatest(coalesce(p_limit,100),1),1000) LOOP
    BEGIN res := resolve_person(r.id);
      IF (res->>'created')::boolean THEN created := created+1; ELSE linked := linked+1; END IF;
    EXCEPTION WHEN others THEN fail := fail+1;
      fails := fails || jsonb_build_object('inbound_id',r.id,'error',left(SQLERRM,160));
      UPDATE inbound_records SET rejected_at=now(), rejected_reason=left(SQLERRM,500) WHERE id=r.id;
    END;
  END LOOP;
  RETURN jsonb_build_object('created',created,'linked',linked,'failed',fail,'failures',fails);
END 
```

### review_queues() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 31 lines
```sql

BEGIN
  IF NOT (has_role('owner') OR has_role('admin')) THEN
    RETURN jsonb_build_object('permitted', false); END IF;
  RETURN jsonb_build_object(
    'permitted', true,
    'org_candidates', jsonb_build_object(
      'total', (SELECT count(*) FROM quarantine.org_candidates),
      'unanimous', (SELECT count(*) FROM quarantine.org_candidates WHERE name_agreement),
      'linkable', (SELECT count(*) FROM quarantine.org_candidates WHERE existing_organization_id IS NOT NULL),
      'promotable', (SELECT count(*) FROM quarantine.org_candidates
                      WHERE name_agreement AND existing_organization_id IS NULL
                        AND promoted_organization_id IS NULL AND people_count BETWEEN 5 AND 49),
      'promoted', (SELECT count(*) FROM quarantine.org_candidates WHERE promoted_organization_id IS NOT NULL)),
    'research', jsonb_build_object(
      'total', (SELECT count(*) FROM quarantine.research_contacts),
      'promoted', (SELECT count(*) FROM quarantine.research_contacts WHERE promoted_person_id IS NOT NULL)),
    'linkable_list', coalesce((
      SELECT jsonb_agg(jsonb_build_object('domain', c.domain, 'company_name', c.company_name,
               'people_count', c.people_count, 'existing_name', o.name, 'existing_id', o.id)
             ORDER BY c.people_count DESC)
        FROM quarantine.org_candidates c JOIN organizations o ON o.id = c.existing_organization_id), '[]'::jsonb),
    'promotable_list', coalesce((SELECT jsonb_agg(x) FROM (
        SELECT jsonb_build_object('domain', c.domain, 'company_name', c.company_name,
                 'people_count', c.people_count, 'corroborated', (c.website = c.domain)) AS x
          FROM quarantine.org_candidates c
         WHERE c.name_agreement AND c.existing_organization_id IS NULL AND c.promoted_organization_id IS NULL
         ORDER BY (c.website = c.domain) DESC, c.people_count DESC LIMIT 40) y), '[]'::jsonb),
    'promotable_list_capped_at', 40);
END;

```

### rls_auto_enable() -> event_trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=pg_catalog | exec anon=n authenticated=n | 24 lines
- comment: Event-trigger body for `ensure_rls`: switches row level security on for every new table in public. Never called directly — REVOKEd from PUBLIC, anon and authenticated because a SECURITY DEFINER function auto-exposed at /rest/v1/rpc/ has no business being reachable without signing in. 008 said so, 099 enforces it.
```sql

DECLARE
  cmd record;
BEGIN
  FOR cmd IN
    SELECT *
    FROM pg_event_trigger_ddl_commands()
    WHERE command_tag IN ('CREATE TABLE', 'CREATE TABLE AS', 'SELECT INTO')
      AND object_type IN ('table','partitioned table')
  LOOP
     IF cmd.schema_name IS NOT NULL AND cmd.schema_name IN ('public') AND cmd.schema_name NOT IN ('pg_catalog','information_schema') AND cmd.schema_name NOT LIKE 'pg_toast%' AND cmd.schema_name NOT LIKE 'pg_temp%' THEN
      BEGIN
        EXECUTE format('alter table if exists %s enable row level security', cmd.object_identity);
        RAISE LOG 'rls_auto_enable: enabled RLS on %', cmd.object_identity;
      EXCEPTION
        WHEN OTHERS THEN
          RAISE LOG 'rls_auto_enable: failed to enable RLS on %', cmd.object_identity;
      END;
     ELSE
        RAISE LOG 'rls_auto_enable: skip % (either system schema or not in enforced list: %.)', cmd.object_identity, cmd.schema_name;
     END IF;
  END LOOP;
END;

```

### role_for_designation(p_designation text) -> person_role
- sql | invoker | immutable | config: search_path=public, pg_temp | exec anon=Y authenticated=Y | 14 lines
```sql

  SELECT CASE lower(btrim(coalesce(p_designation,'')))
    WHEN 'student' THEN 'student'::person_role
    WHEN 'parent' THEN 'parent'
    WHEN 'principal' THEN 'educator'
    WHEN 'school representative' THEN 'educator'
    WHEN 'teacher' THEN 'educator'
    WHEN 'counsellor' THEN 'educator'
    WHEN 'counselor' THEN 'educator'
    WHEN 'corporate' THEN 'partner'
    WHEN 'university' THEN 'partner'
    WHEN 'partner' THEN 'partner'
    ELSE 'unknown' END;

```

### rotate_ingest_key(p_system text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 16 lines
```sql

DECLARE me uuid := public.auth_staff_id(); raw text;
BEGIN
  IF me IS NULL OR NOT EXISTS (SELECT 1 FROM public.staff_role_assignments WHERE staff_id = me AND role = 'owner') THEN
    RAISE EXCEPTION 'Only an owner may issue an ingest key';
  END IF;
  IF NOT EXISTS (SELECT 1 FROM public.source_systems WHERE key = upper(btrim(p_system))) THEN
    RAISE EXCEPTION 'Unknown source system %', p_system;
  END IF;
  raw := 'lwl_ingest_' || replace(encode(extensions.gen_random_bytes(32), 'base64'), '/', '_');
  UPDATE public.source_systems
     SET ingest_key_hash = sha256(convert_to(raw, 'UTF8')), ingest_key_set_at = now()
   WHERE key = upper(btrim(p_system));
  RETURN jsonb_build_object('system', upper(btrim(p_system)), 'ingest_key', raw,
    'note', 'Shown once. Not recoverable — rotate again if lost. Ingest is still disabled until an owner turns it on.');
END 
```

### school_request(p_name text, p_country text, p_city text, p_domain text, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 50 lines
- comment: Staff ask for a school that is not in the registry. Writes to quarantine only — never to organizations. 036.
```sql

DECLARE
  me  uuid := public.auth_staff_id();
  nm  text := nullif(btrim(coalesce(p_name, '')), '');
  ctry text := nullif(btrim(coalesce(p_country, '')), '');
  hit uuid;
  cid uuid;
BEGIN
  IF me IS NULL THEN RAISE EXCEPTION 'Only a staff member may request a school'; END IF;
  IF nm IS NULL OR length(nm) < 3 THEN
    RAISE EXCEPTION 'A school name of at least 3 characters is required';
  END IF;

  SELECT o.id INTO hit FROM organizations o
   WHERE lower(btrim(o.name)) = lower(nm)
-- ... truncated (50 lines total)
```

### schools_public(p_system text, p_ingest_key text, p_q text, p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, extensions, pg_temp | exec anon=Y authenticated=Y | 29 lines
- comment: The shared school dropdown for the six apps. Approved organisations only, key-guarded like submit_inbound. A school that is missing is requested via submit_inbound(kind=organisation).
```sql

DECLARE
  sys public.source_systems%ROWTYPE;
  q   text := nullif(btrim(lower(coalesce(p_q, ''))), '');
  lim int  := least(greatest(coalesce(p_limit, 20), 1), 100);
  items jsonb;
BEGIN
  SELECT * INTO sys FROM public.source_systems WHERE key = upper(btrim(coalesce(p_system, '')));
  IF sys.key IS NULL OR NOT coalesce(sys.ingest_enabled, false) OR sys.ingest_key_hash IS NULL
     OR p_ingest_key IS NULL OR sha256(convert_to(p_ingest_key,'UTF8')) <> sys.ingest_key_hash THEN
    RAISE EXCEPTION 'Ingest is not authorised for this system' USING ERRCODE = '28000';
  END IF;

  SELECT coalesce(jsonb_agg(x ORDER BY x->>'name'), '[]'::jsonb) INTO items
  FROM (
    SELECT DISTINCT ON (o.id)
           jsonb_build_object('organization_id', o.id, 'name', o.name,
                              'city', o.city, 'country', o.country) AS x
      FROM public.organizations o
      LEFT JOIN public.org_aliases a ON a.organization_id = o.id
     WHERE q IS NULL
        OR lower(o.name) LIKE '%'||q||'%'
        OR lower(coalesce(o.city,'')) LIKE '%'||q||'%'
        OR lower(coalesce(a.raw_name,'')) LIKE '%'||q||'%'
     LIMIT lim) s;

  RETURN jsonb_build_object('schools', items, 'count', jsonb_array_length(items),
                            'truncated', jsonb_array_length(items) >= lim);
END 
```

### schools_search(p_q text, p_limit integer) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 44 lines
- comment: The shared school picker. Approved organisations are selectable; research names are returned marked unverified so the caller offers to request rather than to select. Refuses queries shorter than 2 characters. 036.
```sql

DECLARE t text; lim int; res jsonb;
BEGIN
  IF public.auth_staff_id() IS NULL THEN
    RAISE EXCEPTION 'Only a staff member may search schools';
  END IF;
  t := nullif(lower(btrim(coalesce(p_q, ''))), '');
  lim := least(greatest(coalesce(p_limit, 20), 1), 50);
  IF t IS NULL OR length(t) < 2 THEN
    RETURN jsonb_build_object('query', t, 'too_short', true,
                              'approved', '[]'::jsonb, 'unverified', '[]'::jsonb);
  END IF;
  WITH approved AS (
    SELECT o.id, o.name, o.city, o.country, o.domain,
           greatest(similarity(lower(btrim(o.name)), t),
-- ... truncated (44 lines total)
```

### set_cohort_capacity(p_cohort_id uuid, p_capacity integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 22 lines
- comment: Sets the one cohort number no record contains. Owner or manager only, logged to activities.
```sql

DECLARE c cohorts%ROWTYPE;
BEGIN
  IF auth_staff_id() IS NULL OR NOT (has_role('owner') OR has_role('manager')) THEN
    RAISE EXCEPTION 'Only an owner or manager may set a cohort capacity' USING ERRCODE='42501'; END IF;
  SELECT * INTO c FROM cohorts WHERE id = p_cohort_id;
  IF c.id IS NULL THEN RAISE EXCEPTION 'No such cohort %', p_cohort_id USING ERRCODE='P0002'; END IF;
  IF p_capacity IS NOT NULL AND p_capacity <= 0 THEN
    RAISE EXCEPTION 'Capacity must be a positive number, or empty to clear it. Use is_open to close a cohort.'
      USING ERRCODE='22023'; END IF;
  IF p_capacity IS NOT NULL AND p_capacity > 100000 THEN
    RAISE EXCEPTION 'Capacity of % looks like a typo rather than a cohort size', p_capacity
      USING ERRCODE='22023'; END IF;

  UPDATE cohorts SET capacity = p_capacity, updated_at = now() WHERE id = p_cohort_id;

  INSERT INTO activities (occurred_at, person_id, organization_id, kind, actor, payload)
  VALUES (now(), NULL, NULL, 'cohort_capacity_set', 'staff:'||auth_staff_id()::text,
          jsonb_build_object('cohort_id',p_cohort_id,'label',c.label,'was',c.capacity,'now',p_capacity));

  RETURN jsonb_build_object('ok',true,'cohort_id',p_cohort_id,'capacity',p_capacity,'was',c.capacity);
END 
```

### set_identity_key() -> trigger
- plpgsql | invoker | volatile | config: search_path=public, pg_temp | exec anon=Y authenticated=Y | 8 lines
```sql

BEGIN
  NEW.value_norm := public.identity_key(NEW.kind::text, NEW.value_raw);
  IF NEW.value_norm IS NULL THEN
    RAISE EXCEPTION 'identity of kind % has no usable value (%)', NEW.kind, NEW.value_raw;
  END IF;
  RETURN NEW;
END 
```

### set_ingest_enabled(p_system text, p_on boolean) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 12 lines
```sql

DECLARE me uuid := public.auth_staff_id();
BEGIN
  IF me IS NULL OR NOT EXISTS (SELECT 1 FROM public.staff_role_assignments WHERE staff_id = me AND role = 'owner') THEN
    RAISE EXCEPTION 'Only an owner may enable or disable ingest';
  END IF;
  IF p_on AND NOT EXISTS (SELECT 1 FROM public.source_systems WHERE key = upper(btrim(p_system)) AND ingest_key_hash IS NOT NULL) THEN
    RAISE EXCEPTION 'Issue an ingest key for % first', p_system;
  END IF;
  UPDATE public.source_systems SET ingest_enabled = p_on WHERE key = upper(btrim(p_system));
  RETURN jsonb_build_object('system', upper(btrim(p_system)), 'ingest_enabled', p_on);
END 
```

### set_limit(real) -> real
- c | invoker | volatile | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
set_limit
```

### show_limit() -> real
- c | invoker | stable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
show_limit
```

### show_trgm(text) -> text[]
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
show_trgm
```

### similarity(text, text) -> real
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
similarity
```

### similarity_dist(text, text) -> real
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
similarity_dist
```

### similarity_op(text, text) -> boolean
- c | invoker | stable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
similarity_op
```

### source_provenance() -> jsonb
- sql | invoker | stable | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 51 lines
- comment: Per-system arrival for the console. INVOKER: people counts run through people_v so a scoped role sees scoped totals. 028.
```sql

  WITH mine AS (
    SELECT split_part(p.source_code, '-', 2)             AS system_key,
           split_part(p.source_code, '-', 3)             AS period,
           nullif(split_part(p.source_code, '-', 4), '') AS dataset,
           p.source_code
      FROM people_v p
     WHERE p.source_code IS NOT NULL
  ), per_system AS (
    SELECT system_key,
           count(*)                    AS people,
           count(DISTINCT source_code)  AS codes,
           min(period)                  AS first_period,
           max(period)                  AS last_period
      FROM mine GROUP BY 1
-- ... truncated (51 lines total)
```

### spine_agent_health() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, cron, pg_temp | exec anon=n authenticated=Y | 40 lines
```sql

BEGIN
  IF auth.role() IN ('anon','authenticated') AND public.auth_staff_id() IS NULL THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE = '42501';
  END IF;
  RETURN (
    WITH runs AS (
      SELECT j.jobname, j.schedule, j.active,
             count(d.*) FILTER (WHERE d.status = 'succeeded')  AS ok,
             count(d.*) FILTER (WHERE d.status <> 'succeeded') AS failed,
             max(d.end_time)                                   AS last_run,
             max(d.end_time) FILTER (WHERE d.status <> 'succeeded') AS last_failure,
             (array_agg(d.return_message ORDER BY d.end_time DESC)
                FILTER (WHERE d.status <> 'succeeded'))[1]     AS last_error
        FROM cron.job j
        LEFT JOIN cron.job_run_details d ON d.jobid = j.jobid
       WHERE j.jobname LIKE 'spine\_%'
       GROUP BY j.jobname, j.schedule, j.active
    )
    SELECT jsonb_build_object(
      'jobs', coalesce(jsonb_agg(jsonb_build_object(
                'job',          jobname,
                'schedule',     schedule,
                'active',       active,
                'ok',           ok,
                'failed',       failed,
                'last_run',     last_run,
                'last_failure', last_failure,
                'last_error',   last_error,
                'healthy',      (failed = 0)
              ) ORDER BY failed DESC, jobname), '[]'::jsonb),
      'jobs_failing',   count(*) FILTER (WHERE failed > 0),
      'failures_total', coalesce(sum(failed), 0),
      'how', 'Read straight from cron.job_run_details, which pg_cron has written since the first '
          || 'spine job was scheduled. No agent writes this and none can forget to: it is the '
          || 'scheduler''s own record. A job that fails is visible here on its next run.')
      FROM runs
  );
END;

```

### staff_directory() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=Y | 22 lines
```sql

BEGIN
  IF NOT (has_role('owner') OR has_role('admin')) THEN RETURN jsonb_build_object('permitted', false); END IF;
  RETURN jsonb_build_object('permitted', true,
    'staff', coalesce((SELECT jsonb_agg(jsonb_build_object(
        'staff_id', s.id, 'email', s.email, 'full_name', s.full_name, 'job_title', s.job_title,
        'status', s.status, 'linked', (s.auth_user_id IS NOT NULL), 'starts_on', s.starts_on,
        'decided_by', (SELECT d.full_name FROM staff d WHERE d.id = s.decided_by),
        'decided_at', s.decided_at, 'decision_note', s.decision_note,
        'roles', coalesce((SELECT jsonb_agg(ra.role ORDER BY ra.role) FROM staff_role_assignments ra WHERE ra.staff_id=s.id),'[]'::jsonb),
        'grants', coalesce((SELECT jsonb_agg(jsonb_build_object('key',g.grant_key,'expires_on',g.expires_on,
                    'days_left',g.expires_on-current_date,'expired',(g.expires_on<current_date),'reason',g.reason)
                    ORDER BY g.grant_key) FROM staff_grants g WHERE g.staff_id=s.id),'[]'::jsonb),
        'scopes', coalesce((SELECT jsonb_agg(sc.scope_kind ORDER BY sc.scope_kind) FROM staff_scopes sc WHERE sc.staff_id=s.id),'[]'::jsonb))
      ORDER BY s.status, s.email) FROM staff s), '[]'::jsonb),
    'invitations', coalesce((SELECT jsonb_agg(jsonb_build_object(
        'email',i.email,'full_name',i.full_name,'job_title',i.job_title,
        'roles',to_jsonb(i.roles),'grants',to_jsonb(i.grants),'expires_at',i.expires_at,
        'state', CASE WHEN i.accepted_at IS NOT NULL THEN 'accepted' WHEN i.revoked_at IS NOT NULL THEN 'revoked'
                      WHEN i.expires_at < now() THEN 'expired' ELSE 'open' END)
      ORDER BY i.created_at DESC) FROM staff_invitations i), '[]'::jsonb));
END; 
```

### staff_visible_ids() -> SETOF uuid
- sql | SECURITY DEFINER | stable | config: search_path=public | exec anon=n authenticated=n | 15 lines
```sql

  WITH RECURSIVE me AS (
    SELECT auth_staff_id() AS id
  ), team AS (
    SELECT m.id FROM me m WHERE m.id IS NOT NULL
    UNION
    -- Only expands if the caller actually holds team scope. Without it this
    -- would hand every manager's subtree to anyone at scope 'own'.
    SELECT s.id FROM staff s
      JOIN team t ON s.manager_id = t.id
     WHERE EXISTS (SELECT 1 FROM staff_scopes x
                    WHERE x.staff_id = auth_staff_id() AND x.scope_kind = 'team')
  )
  SELECT id FROM team

```

### strict_word_similarity(text, text) -> real
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
strict_word_similarity
```

### strict_word_similarity_commutator_op(text, text) -> boolean
- c | invoker | stable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
strict_word_similarity_commutator_op
```

### strict_word_similarity_dist_commutator_op(text, text) -> real
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
strict_word_similarity_dist_commutator_op
```

### strict_word_similarity_dist_op(text, text) -> real
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
strict_word_similarity_dist_op
```

### strict_word_similarity_op(text, text) -> boolean
- c | invoker | stable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
strict_word_similarity_op
```

### submit_inbound(p_system text, p_ingest_key text, p_kind text, p_payload jsonb, p_idempotency_key text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=Y authenticated=Y | 49 lines
- comment: The only write path for the six apps. Validates a per-app key, mints provenance, appends to inbound_records. Resolves nothing. 030.
```sql

DECLARE
  sys  public.source_systems;
  code text;
  rec  public.inbound_records;
  idem text;
BEGIN
  SELECT * INTO sys FROM public.source_systems WHERE key = upper(btrim(p_system));

  IF sys.key IS NULL OR NOT sys.ingest_enabled OR sys.ingest_key_hash IS NULL
     OR p_ingest_key IS NULL
     OR sha256(convert_to(p_ingest_key, 'UTF8')) <> sys.ingest_key_hash THEN
    RETURN jsonb_build_object('ok', false, 'error', 'not_authorised');
  END IF;

-- ... truncated (49 lines total)
```

### target_set(p_staff_id uuid, p_rows_per_day integer, p_rows_per_week integer, p_meetings_per_month integer, p_is_part_time boolean) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, pg_temp | exec anon=n authenticated=Y | 27 lines
```sql

DECLARE me uuid; pt boolean;
BEGIN
  me := auth_staff_id();
  IF me IS NULL OR NOT (has_role('owner') OR has_role('admin') OR has_role('manager')) THEN
    RAISE EXCEPTION 'Only an owner, admin or manager may set a target' USING ERRCODE='42501'; END IF;

  pt := coalesce(p_is_part_time, false);

  INSERT INTO public.staff_targets AS t
    (staff_id, rows_per_day, rows_per_week, meetings_per_month, is_part_time, set_by)
  VALUES (p_staff_id,
          coalesce(p_rows_per_day, 100),
          coalesce(p_rows_per_week, 700),
          coalesce(p_meetings_per_month, CASE WHEN pt THEN 10 ELSE 20 END),
          pt, me)
  ON CONFLICT (staff_id) DO UPDATE SET
    rows_per_day       = coalesce(p_rows_per_day,       t.rows_per_day),
    rows_per_week      = coalesce(p_rows_per_week,      t.rows_per_week),
    is_part_time       = coalesce(p_is_part_time,       t.is_part_time),
    meetings_per_month = coalesce(p_meetings_per_month,
                                  CASE WHEN coalesce(p_is_part_time, t.is_part_time)
                                       THEN 10 ELSE 20 END),
    set_by = me, updated_at = now();

  RETURN jsonb_build_object('ok', true);
END 
```

### touch_updated_at() -> trigger
- plpgsql | invoker | volatile | config: search_path=public | exec anon=n authenticated=n | 3 lines
```sql

BEGIN NEW.updated_at = now(); RETURN NEW; END;

```

### validator_progress() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 28 lines
```sql

DECLARE out jsonb;
BEGIN
  IF auth_staff_id() IS NULL THEN RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  SELECT jsonb_build_object(
    'domains_total',   (SELECT count(DISTINCT lower(split_part(email_norm,'@',2)))
                          FROM quarantine.research_contacts
                         WHERE email_norm IS NOT NULL AND promoted_person_id IS NULL),
    'domains_checked', (SELECT count(*) FROM quarantine.email_domain_evidence
                         WHERE method = 'dns_resolve'),
    'domains_dead',    (SELECT count(*) FROM quarantine.email_domain_evidence
                         WHERE method = 'dns_resolve' AND result = 'fail'),
    'addresses_killed_by_domain', (
      SELECT count(*) FROM quarantine.research_contacts rc
       WHERE rc.promoted_person_id IS NULL
         AND EXISTS (SELECT 1 FROM quarantine.email_domain_evidence v
                      WHERE v.domain = lower(split_part(rc.email_norm,'@',2))
                        AND ((v.method='dns_resolve' AND v.result='fail')
                          OR (v.method='domain_mx'   AND v.result='fail')
                          OR (v.method='disposable'  AND v.result='pass')))),
    'recent_runs', coalesce((SELECT jsonb_agg(jsonb_build_object(
        'agent', agent, 'stage', stage, 'at', started_at,
        'domains', domains_checked, 'addresses', addresses_hit,
        'passed', passed, 'failed', failed, 'errors', errors) ORDER BY started_at DESC)
      FROM (SELECT * FROM quarantine.validator_runs ORDER BY started_at DESC LIMIT 10) z), '[]'::jsonb)
  ) INTO out;
  RETURN out;
END 
```

### validator_run_log(p_agent text, p_stage text, p_domains integer, p_addresses integer, p_passed integer, p_failed integer, p_errors integer, p_detail jsonb) -> uuid
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=Y | 12 lines
```sql

DECLARE v uuid;
BEGIN
  IF auth_staff_id() IS NULL AND coalesce(auth.role(), '') <> 'service_role' THEN
    RAISE EXCEPTION 'Not staff' USING ERRCODE='42501'; END IF;
  INSERT INTO quarantine.validator_runs
    (agent, stage, finished_at, domains_checked, addresses_hit, passed, failed, errors, detail)
  VALUES (p_agent, p_stage, now(), coalesce(p_domains,0), coalesce(p_addresses,0),
          coalesce(p_passed,0), coalesce(p_failed,0), coalesce(p_errors,0), p_detail)
  RETURNING id INTO v;
  RETURN v;
END 
```

### validator_tick(p_stage text, p_limit integer) -> bigint
- plpgsql | invoker | volatile | config: search_path=public, quarantine, pg_temp | exec anon=n authenticated=n | 19 lines
- comment: Fire one validator batch at the edge function. Called by the spine_validator_dns cron job, and callable by nothing else — it is SECURITY INVOKER and only postgres can read the vault secret it needs. Returns the pg_net request id; the reply lands in net._http_response.
```sql

DECLARE k text; req bigint;
BEGIN
  SELECT decrypted_secret INTO k FROM vault.decrypted_secrets WHERE name = 'spine_anon_key';
  IF k IS NULL THEN
    RAISE EXCEPTION 'vault secret spine_anon_key is missing' USING ERRCODE='42704';
  END IF;

  SELECT net.http_post(
    url     := 'https://<DATA_REF>.supabase.co/functions/v1/validate-emails',
    body    := jsonb_build_object('stage', p_stage, 'limit', p_limit),
    headers := jsonb_build_object('Content-Type','application/json',
                                  'Authorization', 'Bearer ' || k,
                                  'apikey', k),
    timeout_milliseconds := 120000)
  INTO req;

  RETURN req;
END 
```

### word_similarity(text, text) -> real
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
word_similarity
```

### word_similarity_commutator_op(text, text) -> boolean
- c | invoker | stable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
word_similarity_commutator_op
```

### word_similarity_dist_commutator_op(text, text) -> real
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
word_similarity_dist_commutator_op
```

### word_similarity_dist_op(text, text) -> real
- c | invoker | immutable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
word_similarity_dist_op
```

### word_similarity_op(text, text) -> boolean
- c | invoker | stable | config: - | exec anon=Y authenticated=Y | 1 lines
```sql
word_similarity_op
```

## Schema: revenue_brain (75 functions)

### apply_ai_reply_classification(p_handoff_id uuid, p_reply_class text, p_confidence numeric, p_reason text, p_next_action text, p_priority text, p_model text, p_ai_response jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 87 lines
```sql

declare
  h revenue_brain.reply_handoffs%rowtype;
  c text := upper(coalesce(p_reply_class,''));
  pri text := upper(coalesce(p_priority,'MEDIUM'));
begin
  if c not in (
    'POSITIVE','QUESTION','MEETING_REQUEST','NOT_NOW','NOT_INTERESTED',
    'WRONG_PERSON','OOO','UNSUBSCRIBE','OTHER_REVIEW'
  ) then
    return jsonb_build_object('status','ERROR','reason','INVALID_REPLY_CLASS');
  end if;

  if pri not in ('CRITICAL','HIGH','MEDIUM','LOW') then
    pri := 'MEDIUM';
-- ... truncated (87 lines total)
```

### approve_pilot_preflight_case(p_case_id uuid, p_decision text, p_approved_by text, p_notes text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 81 lines
```sql

declare
  c revenue_brain.pilot_preflight_cases%rowtype;
  v_decision text := upper(coalesce(p_decision,''));
begin
  if v_decision not in ('APPROVED','REJECTED') then
    return jsonb_build_object('status','ERROR','reason','INVALID_DECISION');
  end if;

  if nullif(btrim(coalesce(p_approved_by,'')),'') is null then
    return jsonb_build_object('status','ERROR','reason','APPROVER_REQUIRED');
  end if;

  select * into c
  from revenue_brain.pilot_preflight_cases
-- ... truncated (81 lines total)
```

### approve_programme_batch(p_batch_id uuid, p_approved_by text, p_reject_emails text[]) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 12 lines
```sql

begin
  if coalesce(btrim(p_approved_by),'') = '' then return jsonb_build_object('status','ERROR','reason','APPROVER_NAME_REQUIRED'); end if;
  if not exists (select 1 from revenue_brain.programme_batches where id = p_batch_id and status = 'PROPOSED') then
    return jsonb_build_object('status','ERROR','reason','BATCH_NOT_PROPOSED'); end if;
  update revenue_brain.programme_batch_leads set decision = case when email = any (select lower(x) from unnest(p_reject_emails) x) then 'REJECTED' else 'APPROVED' end
  where batch_id = p_batch_id and decision = 'PROPOSED';
  update revenue_brain.programme_batches set status = 'APPROVED', approved_by = p_approved_by, approved_at = now() where id = p_batch_id;
  return jsonb_build_object('status','APPROVED','batch_id',p_batch_id,
    'approved',(select count(*) from revenue_brain.programme_batch_leads where batch_id=p_batch_id and decision='APPROVED'),
    'rejected',(select count(*) from revenue_brain.programme_batch_leads where batch_id=p_batch_id and decision='REJECTED'));
end 
```

### arm_wow_pilot_launch(p_approved_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 129 lines
```sql

declare
  rel revenue_brain.wow_pilot_release%rowtype;
  approved_n int;
  sender_ok boolean;
  ingress_ok boolean;
  seq_ok boolean;
  e2e_ok boolean := false;
  real_send_ok boolean := false;
begin
  if nullif(btrim(coalesce(p_approved_by,'')),'') is null then
    return jsonb_build_object('status','ERROR','reason','APPROVER_REQUIRED');
  end if;

  select coalesce(bool_value,false) into e2e_ok
-- ... truncated (129 lines total)
```

### authorize_wow_real_send(p_approved_by text, p_confirmation text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 80 lines
```sql

declare
  e2e_ok boolean := false;
  polling_ok boolean := false;
  sender_ok boolean := false;
  campaign_route_ok boolean := false;
  phrase text := 'I APPROVE REAL WOW SCHOOL SENDING';
begin
  if nullif(btrim(coalesce(p_approved_by,'')),'') is null then
    return jsonb_build_object('status','ERROR','reason','APPROVER_REQUIRED');
  end if;

  if upper(btrim(coalesce(p_confirmation,''))) <> phrase then
    return jsonb_build_object(
      'status','BLOCK',
-- ... truncated (80 lines total)
```

### bind_yale_test_campaign(p_run_id uuid, p_campaign_id text, p_campaign_name text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=Y authenticated=Y | 11 lines
```sql

begin
  if coalesce(btrim(p_campaign_id),'')='' then return jsonb_build_object('status','ERROR','reason','CAMPAIGN_ID_REQUIRED'); end if;
  update revenue_brain.autonomous_test_runs set provider_campaign_id=p_campaign_id,
    notes=notes||jsonb_build_object('campaign_name',p_campaign_name), updated_at=now() where id=p_run_id;
  if not found then return jsonb_build_object('status','ERROR','reason','RUN_NOT_FOUND'); end if;
  update revenue_brain.campaign_routes set provider_campaign_id=p_campaign_id, campaign_name=p_campaign_name, enabled=true, updated_at=now()
  where programme_code='CO-YIRA-YGEC-090127' and route_key='TEST_AUTONOMOUS' and test_only=true;
  return jsonb_build_object('status','CAMPAIGN_BOUND','run_id',p_run_id,'provider_campaign_id',p_campaign_id);
end;

```

### claim_handoff_notification(p_handoff_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 36 lines
```sql

declare
  h revenue_brain.reply_handoffs%rowtype;
begin
  select * into h
  from revenue_brain.reply_handoffs
  where id=p_handoff_id
  for update;

  if not found then
    return jsonb_build_object('status','ERROR','reason','HANDOFF_NOT_FOUND');
  end if;

  if h.notification_status='SENT' then
    return jsonb_build_object('status','SKIP','reason','ALREADY_NOTIFIED');
  end if;

  if h.notification_status='PROCESSING'
     and h.updated_at > now()-interval '10 minutes' then
    return jsonb_build_object('status','SKIP','reason','ALREADY_PROCESSING');
  end if;

  update revenue_brain.reply_handoffs
  set notification_status='PROCESSING',
      notification_attempts=coalesce(notification_attempts,0)+1,
      notification_error=null,
      updated_at=now()
  where id=p_handoff_id;

  return jsonb_build_object(
    'status','CLAIMED',
    'handoff_id',p_handoff_id,
    'attempt',coalesce(h.notification_attempts,0)+1
  );
end;

```

### claim_wow_pilot_dispatch() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 70 lines
```sql

declare
  state jsonb;
  wr revenue_brain.wow_pilot_case_release%rowtype;
  prep jsonb;
  details_url text;
  apply_url text;
  calendly_url text;
begin
  state := revenue_brain.get_wow_pilot_controller_state();

  if coalesce((state->>'should_run_dispatch')::boolean,false)=false then
    return jsonb_build_object('status','HOLD','reason','PILOT_NOT_ARMED','state',state);
  end if;

-- ... truncated (70 lines total)
```

### classify_reply_text(p_reply_text text) -> jsonb
- plpgsql | invoker | immutable | config: search_path=revenue_brain, public, extensions, pg_temp | exec anon=n authenticated=n | 56 lines
```sql

declare
  t text := lower(trim(coalesce(p_reply_text,'')));
  c text;
  conf numeric;
  reason text;
  action text;
  pri text;
  due_hours integer;
begin
  if t ~ '(unsubscribe|remove me|stop emailing|stop email|do not contact|don''t contact|opt[ -]?out)' then
    c := 'UNSUBSCRIBE'; conf := 0.99; reason := 'Explicit unsubscribe / do-not-contact language';
    action := 'NO_OUTREACH_DNC'; pri := 'CRITICAL'; due_hours := 0;
  elsif t ~ '(verify your email address|trusted sender|opt-inbox|sendio|challenge[- ]response|this is an automated (message|response|reply)|auto-generated message)' then
    c := 'OOO'; conf := 0.95; reason := 'Automated message (sender verification / auto-response), not a person';
-- ... truncated (56 lines total)
```

### create_autonomous_test_run() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 96 lines
```sql

declare
  rid uuid;
  rcode text;
begin
  if not exists(select 1 from revenue_brain.test_recipient_allowlist where active and lower(email)='jatin@example-group.com')
     or not exists(select 1 from revenue_brain.test_recipient_allowlist where active and lower(email)='gunjan@example-group.com') then
    return jsonb_build_object('status','ERROR','reason','TEST_ALLOWLIST_INCOMPLETE');
  end if;

  rcode := 'AUTO-E2E-' || to_char(clock_timestamp(),'YYYYMMDD-HH24MISS');

  insert into revenue_brain.autonomous_test_runs(
    run_code,status,provider_campaign_id,notes,started_at
  ) values(
-- ... truncated (96 lines total)
```

### create_conversion_test_run() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 116 lines
```sql

declare
  rid uuid := gen_random_uuid();
  code text := 'CONV-E2E-' || to_char(clock_timestamp(),'YYYYMMDD-HH24MISSMS');
  pcamp text := 'V9-SYNTH-' || rid::text;
  c record;
  qid uuid;
  cases jsonb := '[]'::jsonb;
begin
  insert into revenue_brain.conversion_test_runs(id,run_code,status,notes)
  values(
    rid,code,'ACTIVE',
    jsonb_build_object(
      'purpose','V9 reply-to-revenue conversion E2E',
      'external_recipients',false,
-- ... truncated (116 lines total)
```

### create_pilot_copy_review() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 185 lines
```sql

declare
  pre_id uuid;
  seq_id uuid;
  seq_code text;
  rid uuid := gen_random_uuid();
  code text := 'PILOT-COPY-REVIEW-' || to_char(clock_timestamp(),'YYYYMMDD-HH24MISSMS');
  c revenue_brain.pilot_preflight_cases%rowtype;
  s revenue_brain.sequence_steps%rowtype;
  first_name text;
  step_subject text;
  step_body text;
  rendered jsonb;
  blockers jsonb;
  rec text;
-- ... truncated (185 lines total)
```

### create_pilot_preflight_run() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 391 lines
```sql

declare
  rid uuid := gen_random_uuid();
  code text := 'PILOT-PREFLIGHT-' || to_char(clock_timestamp(),'YYYYMMDD-HH24MISSMS');
  shadow_id uuid;
  programme text := 'PRG-FINEDU-FLAGSHIP-COMPETITION';
  v_route_key text := 'ET';
  route revenue_brain.campaign_routes%rowtype;
  sc revenue_brain.shadow_cases%rowtype;
  d_id uuid;
  q_id uuid;
  gate jsonb;
  checks jsonb;
  blockers jsonb;
  ready boolean;
-- ... truncated (391 lines total)
```

### create_programme_batch(p_programme_code text, p_size integer, p_region text, p_include_needs_zerobounce boolean, p_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 32 lines
```sql

declare pr revenue_brain.programme_profiles%rowtype; b uuid; n int; sz int;
begin
  select * into pr from revenue_brain.programme_profiles where programme_code = p_programme_code;
  if not found then return jsonb_build_object('status','ERROR','reason','UNKNOWN_PROGRAMME'); end if;
  if cardinality(pr.grade_bands) = 0 or cardinality(pr.target_roles) = 0 then
    return jsonb_build_object('status','ERROR','reason','PROGRAMME_AUDIENCE_NOT_SET'); end if;
  sz := least(greatest(coalesce(p_size, pr.batch_size_default), 1), 1000);
  insert into revenue_brain.programme_batches(programme_code, size_requested, region, include_needs_zerobounce, created_by)
  values (p_programme_code, sz, p_region, p_include_needs_zerobounce, p_by) returning id into b;
  insert into revenue_brain.programme_batch_leads(batch_id, rank, contact_id, school_id, school_name, country, geo_region, contact_name, greeting,
    sop_role, designation, email, email_domain, zb_status, zb_checked_at, role_evidence, role_seen_at, candidate_status)
  select b, row_number() over (order by k, rr, role_seen_at desc nulls last), contact_id, school_id, school_name, country, geo_region, contact_name, greeting,
    sop_role, designation, email, email_domain, zb_status, zb_checked_at, role_evidence, role_seen_at, status
  from (
    select c.*, c.role_rank rr, case c.status when 'READY' then 0 else 1 end k,
      row_number() over (partition by c.email_domain order by case c.status when 'READY' then 0 else 1 end, c.role_rank) dn
    from revenue_brain.programme_candidates(p_programme_code, sz * 20, p_region) c
    where c.status = 'READY' or (p_include_needs_zerobounce and c.status = 'NEEDS_ZEROBOUNCE')
  ) x where dn <= pr.max_per_domain
  order by k, rr, role_seen_at desc nulls last
  limit sz
  on conflict (batch_id, email) do nothing;
  get diagnostics n = row_count;
  if n = 0 then
    update revenue_brain.programme_batches set status = 'CANCELLED', notes = 'No eligible candidates' where id = b;
    return jsonb_build_object('status','EMPTY','batch_id',b,'reason','NO_ELIGIBLE_CANDIDATES');
  end if;
  return jsonb_build_object('status','PROPOSED','batch_id',b,'programme_code',p_programme_code,'leads',n,
    'ready',(select count(*) from revenue_brain.programme_batch_leads where batch_id=b and candidate_status='READY'),
    'needs_zerobounce',(select count(*) from revenue_brain.programme_batch_leads where batch_id=b and candidate_status='NEEDS_ZEROBOUNCE'));
end 
```

### create_reply_handoff_from_webhook(p_webhook_inbox_id uuid, p_owner text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 149 lines
```sql

declare
  w revenue_brain.webhook_inbox%rowtype;
  q revenue_brain.outbound_queue%rowtype;
  cls jsonb;
  reply_text text;
  hid uuid;
  hclass text;
  action text;
  pri text;
  conf numeric;
  reason text;
  due_hours integer;
  hstatus text := 'NEW';
begin
-- ... truncated (149 lines total)
```

### create_safety_test_run() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 104 lines
```sql

declare
  rid uuid := gen_random_uuid();
  code text := 'SAFE-E2E-' || to_char(clock_timestamp(),'YYYYMMDD-HH24MISSMS');
  c record;
  qid uuid;
  pcamp text := 'V8-SYNTH-' || rid::text;
  cases jsonb := '[]'::jsonb;
begin
  insert into revenue_brain.safety_test_runs(id,run_code,status,notes)
  values(
    rid,code,'ACTIVE',
    jsonb_build_object(
      'purpose','V8 safety / negative-event regression suite',
      'external_recipients',false,
-- ... truncated (104 lines total)
```

### create_shadow_run() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 214 lines
```sql

declare
  rid uuid := gen_random_uuid();
  code text := 'SHADOW-E2E-' || to_char(clock_timestamp(),'YYYYMMDD-HH24MISSMS');
  rec record;
  app_count int;
  paid_count int;
  last_app timestamptz;
  last_paid timestamptz;
  suppressed boolean;
  active_human boolean;
  cases jsonb := '[]'::jsonb;
begin
  if not coalesce((select bool_value from revenue_brain.runtime_controls where control_key='SHADOW_MODE_ENABLED'),false) then
    return jsonb_build_object('status','BLOCK','reason','SHADOW_MODE_DISABLED');
-- ... truncated (214 lines total)
```

### create_wow_internal_cert_run() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 164 lines
```sql

declare
  rid uuid := gen_random_uuid();
  code text := 'WOW-INTERNAL-CERT-' || to_char(clock_timestamp(),'YYYYMMDD-HH24MISSMS');
  programme text := 'PRG-FINEDU-FLAGSHIP-COMPETITION';
  route_id text := '74c60971-3952-408b-821c-6e275f97f89e';
  d1 uuid; d2 uuid; q1 uuid; q2 uuid;
  cases jsonb;
begin
  if not exists (
    select 1 from revenue_brain.campaign_routes
    where programme_code=programme
      and route_key='TEST_FINAL_V2'
      and test_only=true
      and enabled=true
-- ... truncated (164 lines total)
```

### create_yale_test_run() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=Y authenticated=Y | 34 lines
```sql

declare rid uuid; rcode text;
begin
  if not exists(select 1 from revenue_brain.test_recipient_allowlist where active and lower(email)='jatin@example-group.com')
     or not exists(select 1 from revenue_brain.test_recipient_allowlist where active and lower(email)='gunjan@example-group.com') then
    return jsonb_build_object('status','ERROR','reason','TEST_ALLOWLIST_INCOMPLETE');
  end if;
  rcode := 'YALE-E2E-' || to_char(clock_timestamp(),'YYYYMMDD-HH24MISS');
  insert into revenue_brain.autonomous_test_runs(run_code,status,provider_campaign_id,notes,started_at)
  values(rcode,'CREATED','PENDING-YALE',jsonb_build_object('purpose','Yale Essay 2027 internal rehearsal','programme','CO-YIRA-YGEC-090127',
    'external_recipients',false,'allowed_recipients',jsonb_build_array('jatin@example-group.com','gunjan@example-group.com')),now())
  returning id into rid;
  insert into revenue_brain.autonomous_test_cases(run_id,case_key,school_id,school_name,contact_id,contact_name,delivery_email,delivery_first_name,scenario,account_brief,expected_outcome)
  values
  (rid,'JATIN_POSITIVE','TEST-YALE-JATIN-'||left(rid::text,8),'Group Yale QA School — Jatin','TEST-YALE-CONTACT-JATIN-'||left(rid::text,8),'Jatin Sharma Internal QA',
   'jatin@example-group.com','Jatin','YALE_POSITIVE_REPLY',
   jsonb_build_object('school_profile','International secondary school with strong English, humanities and Model UN programmes.',
     'signals',jsonb_build_array('English department','Model UN club','essay writing','global issues'),'relationship','NEW_TEST_ACCOUNT',
     'available_programmes',jsonb_build_array(jsonb_build_object('programme_code','CO-YIRA-YGEC-090127','status','READY','fit_hint','HIGH'),
       jsonb_build_object('programme_code','PRG-FINEDU-FLAGSHIP-COMPETITION','status','HOLD','fit_hint','LOW'))),
   'Yale Essay selected; email sent; real reply locks school and creates handoff'),
  (rid,'GUNJAN_NOREPLY','TEST-YALE-GUNJAN-'||left(rid::text,8),'Group Yale QA School — Gunjan','TEST-YALE-CONTACT-GUNJAN-'||left(rid::text,8),'Gunjan Aggarwal Internal QA',
   'gunjan@example-group.com','Gunjan','YALE_NO_REPLY_FULL_SEQUENCE',
   jsonb_build_object('school_profile','High-performing school with debate, history and creative-writing activities.',
     'signals',jsonb_build_array('debate','history','creative writing','student competitions'),'relationship','NEW_TEST_ACCOUNT',
     'available_programmes',jsonb_build_array(jsonb_build_object('programme_code','CO-YIRA-YGEC-090127','status','READY','fit_hint','HIGH'),
       jsonb_build_object('programme_code','PRG-FINEDU-FLAGSHIP-COMPETITION','status','HOLD','fit_hint','LOW'))),
   'Yale Essay selected; full 3-email sequence completes');
  return jsonb_build_object('status','CREATED','run_id',rid,'run_code',rcode,'cases',(
    select jsonb_agg(jsonb_build_object('case_id',c.id,'case_key',c.case_key,'school_id',c.school_id,'school_name',c.school_name,'contact_id',c.contact_id,
      'contact_name',c.contact_name,'delivery_email',c.delivery_email,'delivery_first_name',c.delivery_first_name,'scenario',c.scenario,'account_brief',c.account_brief) order by c.case_key)
    from revenue_brain.autonomous_test_cases c where c.run_id=rid));
end;

```

### eb_touch_updated_at() -> trigger
- plpgsql | invoker | volatile | config: search_path="" | exec anon=Y authenticated=Y | 1 lines
```sql
 begin new.updated_at = now(); return new; end 
```

### enforce_instantly_campaign_ingress() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=Y authenticated=Y | 15 lines
```sql

begin
  if upper(coalesce(new.provider,''))='INSTANTLY'
     and coalesce(new.provider_campaign_id,'') <> ''
     and coalesce(new.event_payload->>'internal_test_only','false') <> 'true'
     and not exists (
       select 1 from revenue_brain.campaign_routes r
       where r.provider='INSTANTLY'
         and r.provider_campaign_id=new.provider_campaign_id
     ) then
    raise exception 'UNREGISTERED_INSTANTLY_CAMPAIGN_WEBHOOK';
  end if;
  return new;
end;

```

### enforce_production_kill_switch() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=Y authenticated=Y | 19 lines
```sql

declare
  prod_enabled boolean := false;
  kill_active boolean := true;
begin
  if coalesce(new.test_mode,true)=false
     and new.queue_status in ('READY_TO_DISPATCH','ENROLLED') then
    select coalesce(bool_value,false) into prod_enabled
      from revenue_brain.runtime_controls where control_key='PRODUCTION_SEND_ENABLED';
    select coalesce(bool_value,true) into kill_active
      from revenue_brain.runtime_controls where control_key='GLOBAL_KILL_SWITCH';

    if not prod_enabled or kill_active then
      raise exception 'PRODUCTION_KILL_SWITCH_ACTIVE';
    end if;
  end if;
  return new;
end;

```

### enforce_test_recipient_allowlist() -> trigger
- plpgsql | invoker | volatile | config: search_path=revenue_brain, public, extensions, pg_temp | exec anon=n authenticated=n | 15 lines
```sql

begin
  if new.test_mode then
    if not exists (
      select 1
      from revenue_brain.test_recipient_allowlist a
      where lower(a.email) = lower(new.delivery_email)
        and a.active = true
    ) then
      raise exception 'TEST_MODE_BLOCK: delivery_email % is not on the active internal test allowlist', new.delivery_email;
    end if;
  end if;
  return new;
end;

```

### evaluate_final_approval(p_queue_id uuid, p_route_key text, p_requested_at timestamp with time zone) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 79 lines
```sql

declare
  q revenue_brain.outbound_queue%rowtype;
  r revenue_brain.campaign_routes%rowtype;
  d revenue_brain.brain_decisions%rowtype;
  s revenue_brain.sender_accounts%rowtype;
  gate jsonb;
  v_verification_ok boolean := false;
  v_role_ok boolean := false;
  v_email_evidence_ok boolean := false;
  v_school_ok boolean := false;
  v_claims_ok boolean := false;
  v_sequence_ok boolean := false;
  v_human_approval_ok boolean := false;
begin
-- ... truncated (79 lines total)
```

### evaluate_send_gate(p_school_id text, p_contact_id text, p_email text, p_programme_code text, p_requested_at timestamp with time zone, p_include_test boolean) -> jsonb
- plpgsql | invoker | volatile | config: search_path=revenue_brain, public, extensions, pg_temp | exec anon=n authenticated=n | 104 lines
- comment: Deterministic pre-send gate for Group Revenue Brain. No AI reasoning; enforces suppression, active human relationship, active programme ownership, 14-day same-contact cooling, school attention/cooling, and duplicate campaign checks.
```sql

declare
  s revenue_brain.school_program_state%rowtype;
  v_last_contact timestamptz;
  v_duplicate boolean := false;
begin
  if p_school_id is null or p_contact_id is null or p_email is null or p_programme_code is null then
    return jsonb_build_object('status','HOLD_IDENTITY','reason','Missing school/contact/email/programme key');
  end if;

  if exists (
    select 1
    from leads.email_suppression es
    where lower(es.email)=lower(p_email)
  ) then
-- ... truncated (104 lines total)
```

### finalize_autonomous_test_run(p_run_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 124 lines
```sql

declare
  r revenue_brain.autonomous_test_runs%rowtype;
  j revenue_brain.autonomous_test_cases%rowtype;
  g revenue_brain.autonomous_test_cases%rowtype;
  j_sent int := 0;
  j_post_reply int := 0;
  g_sent int := 0;
  j_reply_at timestamptz;
  j_handoff_ok boolean := false;
  j_lock_ok boolean := false;
  g_no_reply_ok boolean := false;
  j_pass boolean := false;
  g_pass boolean := false;
  overall boolean := false;
-- ... truncated (124 lines total)
```

### finalize_conversion_test_run(p_run_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 112 lines
```sql

declare
  c revenue_brain.conversion_test_cases%rowtype;
  q revenue_brain.outbound_queue%rowtype;
  s revenue_brain.school_program_state%rowtype;
  h revenue_brain.reply_handoffs%rowtype;
  event_count int;
  revenue_minor bigint;
  payment_rows int;
  passed boolean;
  all_pass boolean := true;
  detail jsonb := '[]'::jsonb;
begin
  for c in
    select * from revenue_brain.conversion_test_cases
-- ... truncated (112 lines total)
```

### finalize_pilot_preflight_run(p_run_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 102 lines
```sql

declare
  r revenue_brain.pilot_preflight_runs%rowtype;
  total_n int;
  ready_n int;
  held_n int;
  provider_ok boolean;
  prod_enabled boolean;
  kill_active boolean;
  ingress_ok boolean;
  final_status text;
  cases jsonb;
begin
  select * into r from revenue_brain.pilot_preflight_runs where id=p_run_id for update;
  if not found then return jsonb_build_object('status','ERROR','reason','RUN_NOT_FOUND'); end if;
-- ... truncated (102 lines total)
```

### finalize_safety_test_run(p_run_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 101 lines
```sql

declare
  c revenue_brain.safety_test_cases%rowtype;
  q revenue_brain.outbound_queue%rowtype;
  s revenue_brain.school_program_state%rowtype;
  h revenue_brain.reply_handoffs%rowtype;
  suppression_reason text;
  passed boolean;
  all_pass boolean := true;
  detail jsonb := '[]'::jsonb;
begin
  for c in
    select * from revenue_brain.safety_test_cases
    where run_id=p_run_id
    order by case_key
-- ... truncated (101 lines total)
```

### finalize_shadow_run(p_run_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 71 lines
```sql

declare
  total_n int;
  eval_n int;
  would_n int;
  hold_n int;
  controls jsonb;
  cases jsonb;
begin
  select count(*)::int,
         count(*) filter(where status='EVALUATED')::int,
         count(*) filter(where would_send=true)::int,
         count(*) filter(where would_send=false)::int
  into total_n,eval_n,would_n,hold_n
  from revenue_brain.shadow_cases
-- ... truncated (71 lines total)
```

### get_all_polling_campaign_routes() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 16 lines
```sql

  select jsonb_build_object(
    'routes',
    coalesce(jsonb_agg(jsonb_build_object(
      'programme_code',programme_code,
      'route_key',route_key,
      'provider_campaign_id',provider_campaign_id,
      'campaign_name',campaign_name,
      'test_only',test_only
    ) order by test_only desc,programme_code,route_key),'[]'::jsonb)
  )
  from revenue_brain.campaign_routes
  where provider='INSTANTLY'
    and enabled=true
    and provider_campaign_id is not null;

```

### get_autonomous_case_send_status(p_run_id uuid, p_case_key text) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 25 lines
```sql

  select jsonb_build_object(
    'run_id', c.run_id,
    'case_id', c.id,
    'case_key', c.case_key,
    'queue_id', c.queue_id,
    'provider_lead_id', c.provider_lead_id,
    'sent_count', (
      select count(*)::int
      from revenue_brain.outreach_events oe
      where oe.queue_id=c.queue_id
        and oe.event_type='SENT'
    ),
    'ready', (
      select count(*) > 0
      from revenue_brain.outreach_events oe
      where oe.queue_id=c.queue_id
        and oe.event_type='SENT'
    )
  )
  from revenue_brain.autonomous_test_cases c
  where c.run_id=p_run_id
    and c.case_key=p_case_key
  limit 1;

```

### get_autonomous_test_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 49 lines
```sql

select jsonb_build_object(
  'run',to_jsonb(r),
  'cases',coalesce((
    select jsonb_agg(
      jsonb_build_object(
        'id',c.id,
        'case_key',c.case_key,
        'school_id',c.school_id,
        'school_name',c.school_name,
        'contact_id',c.contact_id,
        'contact_name',c.contact_name,
        'delivery_email',c.delivery_email,
        'scenario',c.scenario,
        'status',c.status,
-- ... truncated (49 lines total)
```

### get_conversion_test_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 24 lines
```sql

  select jsonb_build_object(
    'run',to_jsonb(r),
    'cases',coalesce((
      select jsonb_agg(to_jsonb(c) order by c.case_key)
      from revenue_brain.conversion_test_cases c
      where c.run_id=r.id
    ),'[]'::jsonb),
    'events',coalesce((
      select jsonb_agg(to_jsonb(e) order by e.occurred_at)
      from revenue_brain.conversion_events e
      join revenue_brain.conversion_test_cases c on c.queue_id=e.queue_id
      where c.run_id=r.id
    ),'[]'::jsonb),
    'revenue',coalesce((
      select jsonb_agg(to_jsonb(a) order by a.attributed_at)
      from revenue_brain.revenue_attribution a
      join revenue_brain.conversion_test_cases c on c.queue_id=a.queue_id
      where c.run_id=r.id
    ),'[]'::jsonb)
  )
  from revenue_brain.conversion_test_runs r
  where r.id=p_run_id;

```

### get_pilot_copy_review(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 12 lines
```sql

  select jsonb_build_object(
    'run',to_jsonb(r),
    'cases',coalesce((
      select jsonb_agg(to_jsonb(c) order by c.recommendation desc,c.school_name)
      from revenue_brain.pilot_copy_review_cases c
      where c.run_id=r.id
    ),'[]'::jsonb)
  )
  from revenue_brain.pilot_copy_review_runs r
  where r.id=p_run_id;

```

### get_pilot_preflight_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 22 lines
```sql

  select jsonb_build_object(
    'run',to_jsonb(r),
    'cases',coalesce((
      select jsonb_agg(to_jsonb(c) order by c.readiness desc,c.school_name)
      from revenue_brain.pilot_preflight_cases c
      where c.run_id=r.id
    ),'[]'::jsonb),
    'controls',coalesce((
      select jsonb_object_agg(control_key,jsonb_build_object(
        'bool_value',bool_value,'json_value',json_value,'description',description,'updated_at',updated_at
      ))
      from revenue_brain.runtime_controls
      where control_key in (
        'PRODUCTION_SEND_ENABLED','GLOBAL_KILL_SWITCH',
        'INSTANTLY_POLLING_INGRESS_READY','INSTANTLY_WEBHOOK_SIGNATURE_READY'
      )
    ),'{}'::jsonb)
  )
  from revenue_brain.pilot_preflight_runs r
  where r.id=p_run_id;

```

### get_polling_campaign_routes() -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 16 lines
```sql

  select jsonb_build_object(
    'routes',
    coalesce(jsonb_agg(jsonb_build_object(
      'programme_code',programme_code,
      'route_key',route_key,
      'provider_campaign_id',provider_campaign_id,
      'campaign_name',campaign_name
    ) order by programme_code,route_key),'[]'::jsonb)
  )
  from revenue_brain.campaign_routes
  where provider='INSTANTLY'
    and test_only=false
    and enabled=true
    and provider_campaign_id is not null;

```

### get_reply_handoff(p_handoff_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 45 lines
```sql

  select to_jsonb(x)
  from (
    select
      h.id,
      h.webhook_inbox_id,
      h.queue_id,
      h.school_id,
      h.school_name,
      h.contact_id,
      h.contact_name,
      h.contact_email,
      h.programme_code,
      h.provider,
      h.provider_campaign_id,
-- ... truncated (45 lines total)
```

### get_safety_test_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 12 lines
```sql

  select jsonb_build_object(
    'run',to_jsonb(r),
    'cases',coalesce((
      select jsonb_agg(to_jsonb(c) order by c.case_key)
      from revenue_brain.safety_test_cases c
      where c.run_id=r.id
    ),'[]'::jsonb)
  )
  from revenue_brain.safety_test_runs r
  where r.id=p_run_id;

```

### get_shadow_run(p_run_id uuid) -> jsonb
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 21 lines
```sql

  select jsonb_build_object(
    'run',to_jsonb(r),
    'cases',coalesce((
      select jsonb_agg(to_jsonb(c) order by c.school_name)
      from revenue_brain.shadow_cases c
      where c.run_id=r.id
    ),'[]'::jsonb),
    'controls',coalesce((
      select jsonb_object_agg(control_key,jsonb_build_object(
        'bool_value',bool_value,'description',description,'updated_at',updated_at
      ))
      from revenue_brain.runtime_controls
      where control_key in (
        'PRODUCTION_SEND_ENABLED','GLOBAL_KILL_SWITCH','SHADOW_MODE_ENABLED','INSTANTLY_WEBHOOK_SIGNATURE_READY'
      )
    ),'{}'::jsonb)
  )
  from revenue_brain.shadow_runs r
  where r.id=p_run_id;

```

### get_wow_internal_cert_status(p_run_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 83 lines
```sql

declare
  r revenue_brain.wow_internal_cert_runs%rowtype;
  enrolled_n int;
  reply_n int;
  handoff_n int;
  cases jsonb;
  final text;
begin
  select * into r
  from revenue_brain.wow_internal_cert_runs
  where id=p_run_id
  for update;

  if not found then
-- ... truncated (83 lines total)
```

### get_wow_pilot_bundle() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 74 lines
```sql

declare
  rel revenue_brain.wow_pilot_release%rowtype;
  copy_run uuid;
  pre_run uuid;
  cases jsonb;
  controls jsonb;
begin
  select * into rel
  from revenue_brain.wow_pilot_release
  where programme_code='PRG-FINEDU-FLAGSHIP-COMPETITION'
  limit 1;

  select id,preflight_run_id into copy_run,pre_run
  from revenue_brain.pilot_copy_review_runs
-- ... truncated (74 lines total)
```

### get_wow_pilot_controller_state() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 59 lines
```sql

declare
  rel revenue_brain.wow_pilot_release%rowtype;
  prod boolean := false;
  kill boolean := true;
  pilot boolean := false;
  e2e boolean := false;
  real_ok boolean := false;
  polling boolean := false;
  pending_n int := 0;
  dispatched_n int := 0;
begin
  select * into rel
  from revenue_brain.wow_pilot_release
  where programme_code='PRG-FINEDU-FLAGSHIP-COMPETITION'
-- ... truncated (59 lines total)
```

### get_yale_sequence() -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=revenue_brain, public | exec anon=Y authenticated=Y | 7 lines
```sql

  select jsonb_build_object('sequence_code',t.sequence_code,'status',t.status,
    'claims',(select jsonb_object_agg(claim_key, jsonb_build_object('value',claim_value,'status',status)) from revenue_brain.claims_registry where programme_code=t.programme_code),
    'steps',(select jsonb_agg(jsonb_build_object('step_order',s.step_order,'days_from_start',s.days_from_start,'delay_from_previous_days',s.delay_from_previous_days,
      'subject',s.subject_template,'body',s.body_text_template,'body_html',s.body_html_template) order by s.step_order) from revenue_brain.sequence_steps s where s.sequence_id=t.id))
  from revenue_brain.sequence_templates t where t.sequence_code='YALE_ESSAY_2027_V1_DRAFT';

```

### get_yale_sequence_by_code(p_sequence_code text) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 7 lines
```sql

  select jsonb_build_object('sequence_code',t.sequence_code,'status',t.status,
    'claims',(select jsonb_object_agg(claim_key, jsonb_build_object('value',claim_value,'status',status)) from revenue_brain.claims_registry where programme_code=t.programme_code),
    'steps',(select jsonb_agg(jsonb_build_object('step_order',s.step_order,'days_from_start',s.days_from_start,'delay_from_previous_days',s.delay_from_previous_days,
      'subject',s.subject_template,'body',s.body_text_template,'body_html',s.body_html_template) order by s.step_order) from revenue_brain.sequence_steps s where s.sequence_id=t.id))
  from revenue_brain.sequence_templates t where t.sequence_code = p_sequence_code;

```

### ingest_instantly_webhook(p_payload jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 225 lines
```sql

declare
  v_event_id text;
  v_event_type text;
  v_campaign_id text;
  v_lead_id text;
  v_sender text;
  v_recipient text;
  v_message_id text;
  v_inbox_id uuid;
  q revenue_brain.outbound_queue%rowtype;
  v_is_reply boolean := false;
  v_is_sent boolean := false;
  v_is_bounce boolean := false;
  v_is_unsubscribe boolean := false;
-- ... truncated (225 lines total)
```

### kill_wow_pilot(p_requested_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 22 lines
```sql

begin
  update revenue_brain.runtime_controls
  set bool_value=false,updated_by=p_requested_by,updated_at=now()
  where control_key in ('PRODUCTION_SEND_ENABLED','WOW_PILOT_DISPATCH_ENABLED');

  update revenue_brain.runtime_controls
  set bool_value=true,updated_by=p_requested_by,updated_at=now()
  where control_key='GLOBAL_KILL_SWITCH';

  update revenue_brain.wow_pilot_release
  set status='KILL_REQUESTED',
      notes=notes || jsonb_build_object('kill_requested_at',now(),'kill_requested_by',p_requested_by),
      updated_at=now()
  where programme_code='PRG-FINEDU-FLAGSHIP-COMPETITION';

  return jsonb_build_object(
    'status','KILL_REQUESTED',
    'note','V11D controller must pause the Instantly pilot campaign; no new dispatches are allowed.'
  );
end;

```

### mark_ai_reply_failure(p_handoff_id uuid, p_model text, p_error_text text, p_ai_response jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 20 lines
```sql

begin
  update revenue_brain.reply_handoffs
  set ai_attempted_at=now(),
      ai_model=p_model,
      ai_response=coalesce(p_ai_response,'{}'::jsonb),
      ai_error=left(coalesce(p_error_text,'Unknown AI classification failure'),4000),
      updated_at=now()
  where id=p_handoff_id;

  if not found then
    return jsonb_build_object('status','ERROR','reason','HANDOFF_NOT_FOUND');
  end if;

  return jsonb_build_object(
    'status','AI_FAILURE_RECORDED',
    'handoff_id',p_handoff_id
  );
end;

```

### mark_handoff_notification_result(p_handoff_id uuid, p_success boolean, p_channel text, p_recipients text, p_error_text text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 21 lines
```sql

begin
  update revenue_brain.reply_handoffs
  set notification_status=case when p_success then 'SENT' else 'FAILED' end,
      notification_channel=p_channel,
      notification_recipients=p_recipients,
      notified_at=case when p_success then now() else notified_at end,
      notification_error=case when p_success then null else left(p_error_text,4000) end,
      updated_at=now()
  where id=p_handoff_id;

  if not found then
    return jsonb_build_object('status','ERROR','reason','HANDOFF_NOT_FOUND');
  end if;

  return jsonb_build_object(
    'status',case when p_success then 'NOTIFIED' else 'NOTIFICATION_FAILED' end,
    'handoff_id',p_handoff_id
  );
end;

```

### prepare_dispatch(p_queue_id uuid, p_route_key text, p_requested_at timestamp with time zone) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 100 lines
```sql

declare
  q revenue_brain.outbound_queue%rowtype;
  r revenue_brain.campaign_routes%rowtype;
  approval jsonb;
  run_id uuid;
begin
  select * into q
  from revenue_brain.outbound_queue
  where id=p_queue_id
  for update;

  if not found then
    return jsonb_build_object('status','BLOCK','reason','QUEUE_NOT_FOUND');
  end if;
-- ... truncated (100 lines total)
```

### programme_candidates(p_programme_code text, p_limit integer, p_region text) -> TABLE(contact_id uuid, school_id uuid, school_name text, country text, geo_region text, contact_name text, greeting text, sop_role text, designation text, email text, email_domain text, zb_status text, zb_checked_at timestamp with time zone, role_evidence text, role_seen_at timestamp with time zone, status text, role_rank integer)
- sql | SECURITY DEFINER | stable | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 40 lines
```sql

  with p as (select * from revenue_brain.programme_profiles where programme_code = p_programme_code),
  base as (
    select c.id, c.school_id, c.school, c.country, c.geo_region, c.contact_name, c.sop_role, c.designation,
      lower(c.email) em, split_part(lower(c.email),'@',2) dom, z.zb_status, z.checked_at, nullif(c.role_evidence,'') ev, c.role_seen_at,
      array_position(p.target_roles, c.sop_role) rr
    from public.research_contacts c cross join p
    left join public.research_email_checks z on z.email = lower(c.email)
    where c.deleted_at is null and c.email is not null and not coalesce(c.generic_contact,false) and c.legal_state = 'allowed'
      and c.school_id is not null
      and c.grade_band = any(p.grade_bands) and c.sop_role = any(p.target_roles)
      and (cardinality(p.target_regions) = 0 or c.geo_region = any(p.target_regions))
      and (p_region is null or c.geo_region = p_region)
      and not exists (select 1 from leads.email_suppression s where lower(s.email) = lower(c.email))
      and not exists (select 1 from revenue_brain.outbound_queue q where not q.test_mode and lower(q.delivery_email) = lower(c.email))
      and not exists (select 1 from revenue_brain.programme_batch_leads bl where bl.email = lower(c.email) and bl.decision in ('PROPOSED','APPROVED','STAGED'))
      and not exists (select 1 from revenue_brain.school_program_state s where s.school_id = 'RS:' || c.school_id
                      and (s.active_human_conversation or s.school_lock_state <> 'CLEAR' or coalesce(s.next_eligible_at, now()) > now()))
  ),
  lab as (
    select *, case
      when zb_status is not null and zb_status <> 'valid' then 'BLOCKED_EMAIL'
      when ev is null or role_seen_at is null or role_seen_at < now() - interval '180 days' then 'NEEDS_EVIDENCE'
      when zb_status is null or checked_at < now() - interval '180 days' then 'NEEDS_ZEROBOUNCE'
      else 'READY' end st
    from base),
  best as (
    select distinct on (school_id) * from lab
    order by school_id, case st when 'READY' then 0 when 'NEEDS_ZEROBOUNCE' then 1 when 'NEEDS_EVIDENCE' then 2 else 3 end, rr, role_seen_at desc nulls last)
  select id, school_id, school, country, geo_region, contact_name,
    case when nullif(trim(contact_name),'') is null then 'colleague'
         when contact_name ~* '^(dr|mr|mrs|ms|mx|prof)\.?\s+\S' then
           initcap(regexp_replace(split_part(trim(contact_name),' ',1),'\.$','')) || '. ' ||
           (regexp_split_to_array(trim(contact_name),'\s+'))[array_length(regexp_split_to_array(trim(contact_name),'\s+'),1)]
         else initcap(split_part(trim(contact_name),' ',1)) end,
    sop_role, designation, em, dom, zb_status, checked_at, ev, role_seen_at, st, rr
  from best
  order by case st when 'READY' then 0 when 'NEEDS_ZEROBOUNCE' then 1 when 'NEEDS_EVIDENCE' then 2 else 3 end, rr, role_seen_at desc nulls last
  limit greatest(coalesce(p_limit,1000),1);

```

### programme_evidence_worklist(p_programme_code text, p_limit integer, p_region text) -> TABLE(priority integer, school_name text, country text, geo_region text, contact_name text, sop_role text, designation text, email text, school_website text, contact_id uuid, school_id uuid, what_to_fill text)
- sql | SECURITY DEFINER | stable | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 10 lines
```sql

  select (row_number() over (order by c.role_rank, c.school_name))::int, c.school_name, c.country, c.geo_region, c.contact_name, c.sop_role, c.designation,
    c.email, coalesce(nullif(s.website,''), 'https://' || nullif(s.domain,'')), c.contact_id, c.school_id,
    'Open the school staff page, confirm this person holds this role, then set role_evidence = page URL and role_seen_at = today'
  from revenue_brain.programme_candidates(p_programme_code, 1000000, p_region) c
  left join public.research_schools s on s.id = c.school_id
  where c.status = 'NEEDS_EVIDENCE'
  order by c.role_rank, c.school_name
  limit greatest(coalesce(p_limit,200),1);

```

### programme_pipeline_summary(p_programme_code text, p_region text) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 5 lines
```sql

  select jsonb_build_object('programme_code', p_programme_code, 'region', coalesce(p_region,'ALL'),
    'schools_by_status', coalesce(jsonb_object_agg(status, n), '{}'::jsonb))
  from (select status, count(*) n from revenue_brain.programme_candidates(p_programme_code, 1000000, p_region) group by status) x;

```

### protect_paid_customer_state() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=Y authenticated=Y | 18 lines
```sql

begin
  if old.relationship_state='PAID'
     and new.relationship_state in (
       'NEW_TEST','REPLIED','MEETING_HELD','CIRCULAR_SENT','CIRCULATION_CONFIRMED',
       'APPLICATION_STARTED','APPLICATION_COMPLETED'
     ) then
    raise exception 'PAID_STATE_DOWNGRADE_BLOCKED';
  end if;

  if old.school_lock_state='CUSTOMER'
     and new.school_lock_state in ('CLEAR','ACTIVE_HUMAN','OOO_HOLD') then
    raise exception 'CUSTOMER_LOCK_DOWNGRADE_BLOCKED';
  end if;

  return new;
end;

```

### record_autonomous_test_enrollment(p_case_id uuid, p_provider_lead_id text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 37 lines
```sql

declare
  c revenue_brain.autonomous_test_cases%rowtype;
  r revenue_brain.autonomous_test_runs%rowtype;
  enrolled_count int;
begin
  select * into c from revenue_brain.autonomous_test_cases where id=p_case_id for update;
  if not found then return jsonb_build_object('status','ERROR','reason','CASE_NOT_FOUND'); end if;

  select * into r from revenue_brain.autonomous_test_runs where id=c.run_id;

  update revenue_brain.autonomous_test_cases
  set provider_lead_id=p_provider_lead_id,status='ENROLLED',updated_at=now()
  where id=p_case_id;

  select count(*) into enrolled_count
  from revenue_brain.autonomous_test_cases
  where run_id=c.run_id and status='ENROLLED';

  update revenue_brain.autonomous_test_runs
  set status=case when enrolled_count >= expected_case_count then 'ACTIVE' else 'DISPATCHING' end,
      updated_at=now()
  where id=c.run_id;

  return jsonb_build_object(
    'status','ENROLLED',
    'case_id',p_case_id,
    'case_key',c.case_key,
    'scenario',c.scenario,
    'run_id',c.run_id,
    'run_code',r.run_code,
    'provider_campaign_id',r.provider_campaign_id,
    'provider_lead_id',p_provider_lead_id,
    'delivery_email',c.delivery_email
  );
end;

```

### record_conversion_test_event(p_run_id uuid, p_case_key text, p_event_type text, p_payload jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 106 lines
```sql

declare
  c revenue_brain.conversion_test_cases%rowtype;
  q revenue_brain.outbound_queue%rowtype;
  ev_id uuid;
  v_event_key text;
  v_amount_minor bigint := coalesce((p_payload->>'amount_minor')::bigint,0);
  v_currency text := coalesce(p_payload->>'currency','USD');
  next_stage text;
begin
  select * into c
  from revenue_brain.conversion_test_cases
  where run_id=p_run_id and case_key=p_case_key
  for update;

-- ... truncated (106 lines total)
```

### record_dispatch_result(p_run_id uuid, p_success boolean, p_provider_lead_id text, p_response jsonb, p_error_text text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 28 lines
```sql

declare
  rr revenue_brain.orchestration_runs%rowtype;
begin
  select * into rr from revenue_brain.orchestration_runs where id=p_run_id for update;
  if not found then
    return jsonb_build_object('status','BLOCK','reason','RUN_NOT_FOUND');
  end if;

  update revenue_brain.orchestration_runs
  set status = case when p_success then 'DISPATCHED' else 'FAILED' end,
      provider_lead_id = p_provider_lead_id,
      response_payload = coalesce(p_response,'{}'::jsonb),
      error_text = p_error_text,
      finished_at = now()
  where id=p_run_id;

  if rr.queue_id is not null then
    update revenue_brain.outbound_queue
    set provider_lead_id = coalesce(p_provider_lead_id, provider_lead_id),
        queue_status = case when p_success then 'ENROLLED' else 'DISPATCH_FAILED' end,
        updated_at = now()
    where id=rr.queue_id;
  end if;

  return jsonb_build_object('status',case when p_success then 'DISPATCHED' else 'FAILED' end,'run_id',p_run_id);
end;

```

### record_pilot_provider_snapshot(p_run_id uuid, p_snapshot jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 54 lines
```sql

declare
  r revenue_brain.pilot_preflight_runs%rowtype;
  sender_ok boolean;
  stop_reply_ok boolean;
  stop_auto_ok boolean;
  campaign_match boolean;
  snapshot_ok boolean;
begin
  select * into r from revenue_brain.pilot_preflight_runs where id=p_run_id for update;
  if not found then return jsonb_build_object('status','ERROR','reason','RUN_NOT_FOUND'); end if;

  campaign_match := coalesce(p_snapshot->>'id','')=coalesce(r.provider_campaign_id,'');
  sender_ok := exists(
    select 1
-- ... truncated (54 lines total)
```

### record_shadow_ai_decision(p_run_id uuid, p_case_key text, p_ai_decision jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 92 lines
```sql

declare
  c revenue_brain.shadow_cases%rowtype;
  b jsonb := '[]'::jsonb;
  v_action text := upper(coalesce(p_ai_decision->>'action','HOLD'));
  v_programme text := p_ai_decision->>'programme_code';
  v_readiness text;
  v_would_send boolean := false;
begin
  select * into c
  from revenue_brain.shadow_cases sc
  where sc.run_id=p_run_id and sc.case_key=p_case_key
  for update;

  if not found then
-- ... truncated (92 lines total)
```

### record_wow_pilot_dispatch(p_case_release_id uuid, p_success boolean, p_provider_lead_id text, p_response jsonb, p_error_text text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 39 lines
```sql

declare
  wr revenue_brain.wow_pilot_case_release%rowtype;
  core jsonb;
begin
  select * into wr
  from revenue_brain.wow_pilot_case_release
  where id=p_case_release_id
  for update;

  if not found then return jsonb_build_object('status','ERROR','reason','CASE_RELEASE_NOT_FOUND'); end if;
  if wr.orchestration_run_id is null then return jsonb_build_object('status','ERROR','reason','NO_ORCHESTRATION_RUN'); end if;

  core := revenue_brain.record_dispatch_result(
    wr.orchestration_run_id,p_success,p_provider_lead_id,p_response,p_error_text
  );

  update revenue_brain.wow_pilot_case_release
  set dispatch_status=case when p_success then 'DISPATCHED' else 'FAILED' end,
      provider_lead_id=coalesce(p_provider_lead_id,provider_lead_id),
      dispatch_error=p_error_text,
      dispatched_at=case when p_success then now() else dispatched_at end,
      updated_at=now()
  where id=wr.id;

  if p_success then
    update revenue_brain.wow_pilot_release
    set status='LIVE',
        updated_at=now()
    where programme_code='PRG-FINEDU-FLAGSHIP-COMPETITION';
  end if;

  return jsonb_build_object(
    'status',case when p_success then 'DISPATCHED' else 'FAILED' end,
    'case_release_id',wr.id,
    'core',core
  );
end;

```

### resolve_reply_handoff(p_handoff_id uuid, p_resolved_by text, p_resolution text, p_human_classification text, p_unlock_school boolean, p_next_eligible_at timestamp with time zone) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 55 lines
```sql

declare
  h revenue_brain.reply_handoffs%rowtype;
begin
  select * into h
  from revenue_brain.reply_handoffs
  where id=p_handoff_id
  for update;

  if not found then
    return jsonb_build_object('status','ERROR','reason','HANDOFF_NOT_FOUND');
  end if;

  update revenue_brain.reply_handoffs
  set handoff_status='RESOLVED',
-- ... truncated (55 lines total)
```

### revoke_wow_real_send(p_requested_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 25 lines
```sql

begin
  update revenue_brain.runtime_controls
  set bool_value=false,
      json_value=jsonb_build_object('revoked_by',p_requested_by,'revoked_at',now()),
      updated_by=p_requested_by,
      updated_at=now()
  where control_key='WOW_REAL_SEND_APPROVED';

  update revenue_brain.runtime_controls
  set bool_value=false,updated_by=p_requested_by,updated_at=now()
  where control_key in ('PRODUCTION_SEND_ENABLED','WOW_PILOT_DISPATCH_ENABLED');

  update revenue_brain.runtime_controls
  set bool_value=true,updated_by=p_requested_by,updated_at=now()
  where control_key='GLOBAL_KILL_SWITCH';

  update revenue_brain.campaign_routes
  set enabled=false,updated_at=now()
  where programme_code='PRG-FINEDU-FLAGSHIP-COMPETITION'
    and test_only=false;

  return jsonb_build_object('status','REVOKED');
end;

```

### set_autonomous_test_campaign(p_run_id uuid, p_provider_campaign_id text, p_campaign_name text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 46 lines
```sql

declare
  r revenue_brain.autonomous_test_runs%rowtype;
begin
  select * into r
  from revenue_brain.autonomous_test_runs
  where id=p_run_id
  for update;

  if not found then
    return jsonb_build_object('status','ERROR','reason','RUN_NOT_FOUND');
  end if;

  if p_provider_campaign_id is null or length(trim(p_provider_campaign_id))=0 then
    return jsonb_build_object('status','ERROR','reason','CAMPAIGN_ID_REQUIRED');
-- ... truncated (46 lines total)
```

### set_programme_outreach(p_programme_code text, p_enabled boolean, p_priority integer, p_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 14 lines
```sql

declare v record;
begin
  select * into v from revenue_brain.programme_readiness where programme_code = p_programme_code;
  if not found then return jsonb_build_object('status','ERROR','reason','UNKNOWN_PROGRAMME'); end if;
  if p_enabled and v.status = 'NOT_READY' then
    return jsonb_build_object('status','BLOCKED','reason','PROGRAMME_NOT_READY','missing',to_jsonb(v.missing));
  end if;
  update revenue_brain.programme_profiles
  set outreach_enabled = p_enabled, priority = coalesce(p_priority, priority),
      notes = coalesce(notes,'') || ' | ' || to_char(now(),'DD Mon YYYY') || ': outreach ' || case when p_enabled then 'ON' else 'OFF' end || ' by ' || p_by
  where programme_code = p_programme_code;
  return jsonb_build_object('status','OK','programme_code',p_programme_code,'outreach_enabled',p_enabled);
end 
```

### set_updated_at() -> trigger
- plpgsql | invoker | volatile | config: search_path=revenue_brain, public, extensions, pg_temp | exec anon=n authenticated=n | 6 lines
```sql

begin
  new.updated_at := now();
  return new;
end;

```

### set_wow_case_approval(p_preflight_case_id uuid, p_decision text, p_approved_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 89 lines
```sql

declare
  d text := upper(coalesce(p_decision,''));
  rel revenue_brain.wow_pilot_release%rowtype;
  pc revenue_brain.pilot_preflight_cases%rowtype;
  cc revenue_brain.pilot_copy_review_cases%rowtype;
  v_catchall boolean := true;
  core jsonb;
begin
  if d not in ('APPROVED','REJECTED') then
    return jsonb_build_object('status','ERROR','reason','INVALID_DECISION');
  end if;
  if nullif(btrim(coalesce(p_approved_by,'')),'') is null then
    return jsonb_build_object('status','ERROR','reason','APPROVER_REQUIRED');
  end if;
-- ... truncated (89 lines total)
```

### set_wow_copy_approval(p_decision text, p_approved_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 71 lines
```sql

declare
  d text := upper(coalesce(p_decision,''));
  seq_id uuid;
  stale_count int;
begin
  if d not in ('APPROVED','REJECTED') then
    return jsonb_build_object('status','ERROR','reason','INVALID_DECISION');
  end if;
  if nullif(btrim(coalesce(p_approved_by,'')),'') is null then
    return jsonb_build_object('status','ERROR','reason','APPROVER_REQUIRED');
  end if;

  select id into seq_id
  from revenue_brain.sequence_templates
-- ... truncated (71 lines total)
```

### stage_autonomous_test_decision(p_case_id uuid, p_ai_decision jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 50 lines
```sql

declare
  c revenue_brain.autonomous_test_cases%rowtype;
  bid uuid; qid uuid;
  programme text := p_ai_decision->>'programme_code';
  sender text := coalesce(p_ai_decision->>'sender_email','tenzin@example-group.com');
  custom1 text := left(coalesce(p_ai_decision->>'custom_line_1','Internal autonomous Revenue Brain QA.'),600);
  confidence numeric := greatest(0,least(coalesce((p_ai_decision->>'confidence')::numeric,0),1));
begin
  select * into c from revenue_brain.autonomous_test_cases where id=p_case_id for update;
  if not found then return jsonb_build_object('status','ERROR','reason','CASE_NOT_FOUND'); end if;
  if c.status <> 'READY_FOR_AI' then
    return jsonb_build_object('status','HOLD','reason','CASE_ALREADY_STAGED','case_status',c.status,'queue_id',c.queue_id);
  end if;
  if not exists(select 1 from revenue_brain.test_recipient_allowlist a where a.active and lower(a.email)=lower(c.delivery_email)) then
-- ... truncated (50 lines total)
```

### stage_programme_batch(p_batch_id uuid, p_by text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=n authenticated=n | 52 lines
```sql

declare bt revenue_brain.programme_batches%rowtype; pr revenue_brain.programme_profiles%rowtype; rd text;
  l record; zb record; v_school text; v_contact text; d_id uuid; q_id uuid; staged int := 0; skipped int := 0;
begin
  select * into bt from revenue_brain.programme_batches where id = p_batch_id for update;
  if not found or bt.status <> 'APPROVED' then return jsonb_build_object('status','ERROR','reason','BATCH_NOT_APPROVED'); end if;
  select * into pr from revenue_brain.programme_profiles where programme_code = bt.programme_code;
  select status into rd from revenue_brain.programme_readiness where programme_code = bt.programme_code;
  if rd not in ('READY_TO_LAUNCH','LIVE') then return jsonb_build_object('status','BLOCKED','reason','PROGRAMME_NOT_READY','readiness',rd); end if;
  if cardinality(pr.sender_pool) = 0 then return jsonb_build_object('status','BLOCKED','reason','NO_SENDER'); end if;

  for l in select * from revenue_brain.programme_batch_leads where batch_id = p_batch_id and decision = 'APPROVED' order by rank loop
    select * into zb from public.research_email_checks z where z.email = l.email order by z.checked_at desc nulls last limit 1;
    if not found or zb.zb_status <> 'valid' or zb.checked_at < now() - interval '180 days' then
      update revenue_brain.programme_batch_leads set decision='SKIPPED', skip_reason='NO_VALID_ZEROBOUNCE_RESULT' where id=l.id; skipped := skipped+1; continue; end if;
-- ... truncated (52 lines total)
```

### yale_apply_gate() -> void
- sql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=Y authenticated=Y | 12 lines
```sql

  update revenue_brain.yale_contacts set evidence_caveat =
    concat_ws(' | ', role_verified_raw, email_publisher_type) ~* '(not verified|unverified|not independently|mismatch|conflict|not published|unpublished|no email|no readable|historical|archive|legacy|not current|wrong |role only|inherited|not reproduced|not legible|personal|shared|generic|functional|external|different campus|unconfirmed|not confirmed|not reconfirm|not employer|undated|stale|not resolved)';
  update revenue_brain.yale_contacts set gate_reasons =
    array_remove(coalesce(gate_reasons,array[]::text[]), 'EVIDENCE_HAS_CAVEAT') || case when evidence_caveat then array['EVIDENCE_HAS_CAVEAT'] else array[]::text[] end;
  update revenue_brain.yale_contacts set gate_status = case
      when gate_reasons && array['NO_EMAIL','EMAIL_FORMAT_INVALID','ON_DO_NOT_EMAIL_LIST','EMAIL_DOMAIN_DEAD'] then 'BLOCKED'
      when gate_reasons && array['PERSONAL_EMAIL_DOMAIN','ROLE_NOT_CURRENTLY_VERIFIED','EMAIL_NOT_FROM_OFFICIAL_SOURCE'] then 'NEEDS_RESEARCH'
      when gate_reasons && array['EVIDENCE_HAS_CAVEAT','GENERIC_INBOX','DUPLICATE_EMAIL_ROWS'] then 'NEEDS_REVIEW'
      when gate_reasons && array['CRM_OWNER_MUST_CLEAR'] then 'NEEDS_OWNER_CLEARANCE'
      else 'READY_FOR_ZEROBOUNCE' end;

```

### yale_bind_route(p_route_key text, p_campaign_id text, p_campaign_name text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 8 lines
```sql

begin
  if p_route_key not in ('WAVE2_AMY','WAVE2_VERONICA') then return jsonb_build_object('status','ERROR','reason','UNKNOWN_ROUTE'); end if;
  if coalesce(btrim(p_campaign_id),'') = '' then return jsonb_build_object('status','ERROR','reason','CAMPAIGN_ID_REQUIRED'); end if;
  update revenue_brain.campaign_routes set provider_campaign_id = p_campaign_id, campaign_name = p_campaign_name, enabled = true, updated_at = now()
  where programme_code = 'CO-YIRA-YGEC-090127' and route_key = p_route_key and test_only = false;
  return jsonb_build_object('status','CAMPAIGN_BOUND','route_key',p_route_key,'provider_campaign_id',p_campaign_id);
end 
```

### yale_pilot_bind_campaign(p_campaign_id text, p_campaign_name text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 8 lines
```sql

begin
  if coalesce(btrim(p_campaign_id),'') = '' then return jsonb_build_object('status','ERROR','reason','CAMPAIGN_ID_REQUIRED'); end if;
  update revenue_brain.campaign_routes set provider_campaign_id = p_campaign_id, campaign_name = p_campaign_name, updated_at = now()
  where programme_code = 'CO-YIRA-YGEC-090127' and route_key = 'PILOT_US' and test_only = false;
  return jsonb_build_object('status','CAMPAIGN_BOUND','provider_campaign_id',p_campaign_id,'route_enabled',
    (select enabled from revenue_brain.campaign_routes where programme_code='CO-YIRA-YGEC-090127' and route_key='PILOT_US' and test_only=false));
end 
```

### yale_pilot_dispatch_batch(p_limit integer, p_route_key text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 36 lines
```sql

declare
  l record; prep jsonb; ready jsonb := '[]'::jsonb; held jsonb := '[]'::jsonb;
  v_apply text; v_book text; v_route revenue_brain.campaign_routes%rowtype;
begin
  select * into v_route from revenue_brain.campaign_routes where programme_code='CO-YIRA-YGEC-090127' and route_key=p_route_key and test_only=false;
  if not found then return jsonb_build_object('status','ERROR','reason','UNKNOWN_ROUTE'); end if;
  select claim_value into v_apply from revenue_brain.claims_registry where programme_code='CO-YIRA-YGEC-090127' and claim_key='apply_url' and status='APPROVED';
  select claim_value into v_book from revenue_brain.claims_registry where programme_code='CO-YIRA-YGEC-090127' and claim_key='booking_url' and status='APPROVED';
  for l in select * from revenue_brain.yale_pilot_leads where route_key = p_route_key and status in ('APPROVED','HELD','FAILED') and queue_id is not null
           order by n limit least(greatest(coalesce(p_limit,0),0),600) loop
    begin
      prep := revenue_brain.prepare_dispatch(l.queue_id, p_route_key, now());
    exception when others then
      prep := jsonb_build_object('status','HOLD','reason',sqlerrm);
    end;
    if coalesce(prep->>'status','') = 'CLEAR' then
      update revenue_brain.yale_pilot_leads set status='CLAIMED', run_id=(prep->>'run_id')::uuid, hold_reason=null, updated_at=now() where n=l.n;
      ready := ready || jsonb_build_array(jsonb_build_object(
        'n',l.n,'queue_id',l.queue_id,'run_id',prep->>'run_id','email',prep->>'delivery_email','intended_email',prep->>'intended_contact_email',
        'greeting',l.greeting,'school_name',l.school_name,'school_code',l.school_code,'role_line',coalesce(l.role_line,''),'test_mode',(prep->>'test_mode')::boolean,
        'provider_campaign_id',prep->>'provider_campaign_id','sender_email',prep->>'sender_email',
        'application_link', v_apply || '?utm_source=revenue_brain&utm_medium=email&utm_campaign=yale_essay_2027&utm_content=' || l.school_code,
        'booking_link', v_book));
    else
      update revenue_brain.yale_pilot_leads set status='HELD', updated_at=now(),
        hold_reason = coalesce(prep->>'reason','UNKNOWN') || coalesce(' / '||(prep->'approval'->>'reason'),'') || coalesce(' / '||(prep->'approval'->'gate'->>'status'),'')
      where n=l.n;
      held := held || jsonb_build_array(jsonb_build_object('n',l.n,'email',l.email,'reason',prep));
    end if;
  end loop;
  return jsonb_build_object('status','OK','route_key',p_route_key,'ready',ready,'ready_count',jsonb_array_length(ready),'held_count',jsonb_array_length(held),
    'held_sample',(select jsonb_agg(x) from (select x from jsonb_array_elements(held) x limit 3) s),
    'remaining',(select count(*) from revenue_brain.yale_pilot_leads where route_key=p_route_key and status in ('APPROVED','HELD','FAILED')),
    'route_campaign_id',v_route.provider_campaign_id,'route_enabled',v_route.enabled);
end 
```

### yale_pilot_record(p_n integer, p_success boolean, p_provider_lead_id text, p_response jsonb, p_error text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public | exec anon=n authenticated=n | 10 lines
```sql

declare l revenue_brain.yale_pilot_leads%rowtype; core jsonb;
begin
  select * into l from revenue_brain.yale_pilot_leads where n=p_n for update;
  if not found or l.run_id is null or l.status <> 'CLAIMED' then return jsonb_build_object('status','ERROR','reason','LEAD_NOT_CLAIMED','n',p_n); end if;
  core := revenue_brain.record_dispatch_result(l.run_id, p_success, p_provider_lead_id, coalesce(p_response,'{}'::jsonb), p_error);
  update revenue_brain.yale_pilot_leads set status = case when p_success then 'DISPATCHED' else 'FAILED' end,
    provider_lead_id = coalesce(p_provider_lead_id, provider_lead_id), hold_reason = p_error, updated_at = now() where n = p_n;
  return jsonb_build_object('status', case when p_success then 'DISPATCHED' else 'FAILED' end, 'n', p_n, 'core', core);
end 
```

### yale_reconcile() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=revenue_brain, public, leads | exec anon=Y authenticated=Y | 98 lines
```sql

declare n int;
begin
  update revenue_brain.yale_contacts c set
    email_valid_format = c.email_norm ~ '^[a-z0-9._%+''-]+@[a-z0-9.-]+\.[a-z]{2,}$',
    email_domain = split_part(c.email_norm,'@',2),
    personal_domain = split_part(c.email_norm,'@',2) in ('gmail.com','yahoo.com','hotmail.com','outlook.com','icloud.com','aol.com','live.com','msn.com','ymail.com','protonmail.com','proton.me','rediffmail.com','qq.com','163.com'),
    generic_inbox = split_part(c.email_norm,'@',1) ~ '^(info|office|admin|admissions|contact|enquiries|inquiries|reception|hello|mail|school|principal|headmaster|secretary|frontdesk|general)$',
    role_verified = (coalesce(c.role_verified_raw,'') ~* '^(yes|current official role verified)' ),
    email_official = (coalesce(c.email_publisher_type,'') ~* '^official'),
    role_relevance = case
      when concat_ws(' ',c.designation,c.precise_role) ~* '(english|literature|writing|humanities|language arts|essay)' then 90
      when concat_ws(' ',c.designation,c.precise_role) ~* '(model un|\mmun\M|debate|global politics|social studies|history|civics|government)' then 85
      when concat_ws(' ',c.designation,c.precise_role) ~* '(ib |\mdp\M|extended essay|gifted|enrichment|competition|academic coordinator|curriculum)' then 80
      when concat_ws(' ',c.designation,c.precise_role) ~* '(head of upper|upper school|high school principal|vice principal|deputy head|director of studies|academic dean|dean of)' then 70
-- ... truncated (98 lines total)
```

## Schema: attribution (4 functions)

### enforce_rules() -> trigger
- plpgsql | invoker | volatile | config: search_path=attribution, public, extensions, pg_temp | exec anon=Y authenticated=Y | 49 lines
```sql

declare c attribution.contacts; k attribution.campaigns; cap int; tz text; n int;
begin
  select * into c from attribution.contacts where contact_id = new.contact_id;
  select * into k from attribution.campaigns where campaign_id = new.campaign_id;

  if tg_op = 'INSERT' or (new.status = 'SENT' and old.status is distinct from 'SENT') then
    if not k.active then raise exception 'Campaign % is not active', k.campaign_id; end if;
    if not c.active or c.email_status <> 'VERIFIED' or c.contact_status not like 'VERIFIED%' then
      raise exception 'Contact % is not sendable (email %, contact %, active %). Only VERIFIED emails may be sent.',
        c.contact_id, c.email_status, c.contact_status, c.active;
    end if;
    if exists (select 1 from attribution.suppressions x where lower(x.email) = lower(c.email)
                and (x.campaign_id is null or x.campaign_id = new.campaign_id)) then
      raise exception '% is suppressed', c.email;
-- ... truncated (49 lines total)
```

### ids_are_permanent() -> trigger
- plpgsql | invoker | volatile | config: search_path=attribution, public, extensions, pg_temp | exec anon=Y authenticated=Y | 7 lines
```sql

begin
  if to_jsonb(new) ->> tg_argv[0] is distinct from to_jsonb(old) ->> tg_argv[0] then
    raise exception '% is permanent and cannot be changed', tg_argv[0];
  end if;
  return new;
end 
```

### new_outreach(p_contact_id text, p_campaign_id text, p_created_by text, p_parallel_approved_by text, p_is_test boolean) -> SETOF attribution.v_outreach_links
- plpgsql | invoker | volatile | config: search_path=attribution, public, extensions, pg_temp | exec anon=Y authenticated=Y | 11 lines
```sql

declare t int; tz text; id text;
begin
  select value into tz from attribution.settings where key = 'cap_timezone';
  perform pg_advisory_xact_lock(hashtext('attribution.touch:' || p_contact_id));
  select coalesce(max(touch_no), 0) + 1 into t from attribution.outreach where contact_id = p_contact_id;
  id := p_contact_id || '-' || split_part(p_campaign_id, '-', 1) || '-' || to_char(now() at time zone tz, 'YYYYMMDD') || '-' || lpad(t::text, 2, '0');
  insert into attribution.outreach (outreach_id, contact_id, campaign_id, touch_no, parallel_approved_by, is_test, created_by)
  values (id, p_contact_id, p_campaign_id, t, p_parallel_approved_by, p_is_test, p_created_by);
  return query select * from attribution.v_outreach_links where outreach_id = id;
end 
```

### suppress_on_reply() -> trigger
- plpgsql | invoker | volatile | config: search_path=attribution, public, extensions, pg_temp | exec anon=Y authenticated=Y | 10 lines
```sql

begin
  if new.reply_status in ('UNSUBSCRIBED','BOUNCED') and old.reply_status is distinct from new.reply_status then
    insert into attribution.suppressions (email, campaign_id, reason)
    select c.email, null, lower(new.reply_status) || ' via ' || new.outreach_id from attribution.contacts c where c.contact_id = new.contact_id
    on conflict do nothing;
  end if;
  if new.reply_status <> 'NO REPLY' then new.suppressed := true; new.suppression_reason := coalesce(new.suppression_reason, new.reply_status); end if;
  return new;
end 
```

## Schema: leads (48 functions)

### auto_confirm_school_matches() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public, extensions | exec anon=n authenticated=n | 44 lines
```sql

DECLARE
  v_meetings_reviewed INT := 0;
  v_apps_reviewed     INT := 0;
  v_schools_approved  INT := 0;
BEGIN
  UPDATE leads.meetings
     SET school_match_reviewed = true
   WHERE school_code IS NOT NULL
     AND school_match_reviewed = false
     AND (
       school_match_source = 'ai_auto_create'
       OR school_match_source = 'alias'
       OR (school_match_source = 'ai' AND COALESCE(school_match_confidence, 0) >= 0.85)
       OR school_match_source = 'human'
-- ... truncated (44 lines total)
```

### clear_score_on_non_completion() -> trigger
- plpgsql | invoker | volatile | config: search_path=leads, public, extensions | exec anon=Y authenticated=Y | 11 lines
```sql

BEGIN
  IF NEW.meeting_status IN ('No Show','Cancelled')
     AND (OLD.meeting_status IS DISTINCT FROM NEW.meeting_status
          OR TG_OP = 'INSERT') THEN
    NEW.lead_score := NULL;
    NEW.lead_temperature := NULL;
  END IF;
  RETURN NEW;
END;

```

### deal_create_repeat(p_deal_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 30 lines
```sql

declare d leads.deals; nid uuid;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  select * into d from leads.deals where id = p_deal_id;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;
  if exists (select 1 from leads.deals where repeat_of = p_deal_id) then
    return jsonb_build_object('ok', true, 'already', true);
  end if;

  insert into leads.deals (
    school_code, school_name, contact_name, contact_email, contact_phone,
    designation, decision_maker_name, decision_maker_role, country, region,
    campaign_tag, lead_source_type, stream, programme,
    stage, status, owner, currency, price_quoted_minor, minimum_required,
    next_action, next_action_owner, next_action_due, proposal_sent_date,
    repeat_of, created_by)
  values (
    d.school_code, d.school_name, d.contact_name, d.contact_email, d.contact_phone,
    d.designation, d.decision_maker_name, d.decision_maker_role, d.country, d.region,
    'repeat', 'repeat', d.stream, d.programme,
    2, 'active', d.owner, d.currency, d.price_quoted_minor, d.minimum_required,
    'Propose the next cohort date for this school', d.owner, current_date + 14,
    current_date, p_deal_id, leads.sop_actor())
  returning id into nid;

  update leads.deals set post_delivery = post_delivery || '{"repeat_record":true}'::jsonb
   where id = p_deal_id;
  return jsonb_build_object('ok', true, 'repeat_deal_id', nid);
end 
```

### deal_gate_recommendation(p_deal_id uuid) -> jsonb
- sql | SECURITY DEFINER | stable | config: search_path=leads, public | exec anon=n authenticated=n | 18 lines
```sql

  select jsonb_build_object(
    'deal_id', d.id, 'school', d.school_name,
    'programme_start', d.programme_start,
    'days_out', d.programme_start - current_date,
    'registered', coalesce(d.registered, 0),
    'minimum', coalesce(d.minimum_required, 15),
    'has_count', d.registered is not null,
    'recommendation', case
      when d.registered is null then 'no_count'
      when d.registered >= ceil(coalesce(d.minimum_required,15) * 0.8) then 'confirmed'
      when d.registered >= ceil(coalesce(d.minimum_required,15) * 0.53) then 'rescue'
      else 'postponed' end,
    'gate_decision', d.gate_decision,
    'gate_decision_at', d.gate_decision_at,
    'gate_due_on', d.programme_start - 45)
  from leads.deals d where d.id = p_deal_id;

```

### deal_log_meeting_notes(p_deal_id uuid, p_meeting_id uuid, p_notes jsonb) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 41 lines
```sql

declare d leads.deals;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  select * into d from leads.deals where id = p_deal_id;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;

  update leads.deals set
    decision_maker_name = coalesce(nullif(p_notes->>'decision_maker_name',''), decision_maker_name),
    decision_maker_role = coalesce(nullif(p_notes->>'decision_maker_role',''), decision_maker_role),
    programme           = coalesce(nullif(p_notes->>'programme',''), programme),
    expected_students   = coalesce((p_notes->>'expected_students')::int, expected_students),
    price_quoted_minor  = coalesce((p_notes->>'price_quoted_minor')::bigint, price_quoted_minor),
    scholarship_minor   = coalesce((p_notes->>'scholarship_minor')::bigint, scholarship_minor),
    dates_discussed     = coalesce(nullif(p_notes->>'dates_discussed',''), dates_discussed),
-- ... truncated (41 lines total)
```

### deal_log_registration(p_deal_id uuid, p_count integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 27 lines
```sql

declare d leads.deals; days_out int; slot text;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  select * into d from leads.deals where id = p_deal_id;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;
  if d.programme_start is null then
    raise exception 'SOP 5.1: set programme dates before counting registrations.';
  end if;

  days_out := d.programme_start - current_date;
  slot := case when days_out > 37 then 'day45'
               when days_out > 22 then 'day30'
               else 'day15' end;

  update leads.deals
     set registered = p_count,
         reg_day45 = case when slot = 'day45' then p_count else reg_day45 end,
         reg_day30 = case when slot = 'day30' then p_count else reg_day30 end,
         reg_day15 = case when slot = 'day15' then p_count else reg_day15 end
   where id = p_deal_id;

  insert into leads.deal_events (deal_id, kind, actor, payload)
  values (p_deal_id, 'registration_counted', leads.sop_actor(),
          jsonb_build_object('count', p_count, 'days_out', days_out, 'slot', slot));
  return jsonb_build_object('ok', true, 'slot', slot, 'days_out', days_out, 'registered', p_count);
end 
```

### deal_log_touch(p_deal_id uuid, p_channel text, p_note text, p_occurred_at timestamp with time zone) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 18 lines
```sql

declare d leads.deals; ts timestamptz := coalesce(p_occurred_at, now());
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  select * into d from leads.deals where id = p_deal_id;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;

  update leads.deals
     set last_touch_at = greatest(coalesce(last_touch_at, ts), ts),
         touch_count = touch_count + 1
   where id = p_deal_id;

  insert into leads.deal_events (deal_id, occurred_at, kind, channel, actor, payload)
  values (p_deal_id, ts, 'touch_logged', p_channel, leads.sop_actor(),
          jsonb_build_object('note', p_note));

  return jsonb_build_object('ok', true, 'deal_id', p_deal_id, 'at', ts);
end 
```

### deal_mark_lost(p_deal_id uuid, p_reason text, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 16 lines
```sql

declare d leads.deals;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  update leads.deals set status = 'lost', lost_reason = p_reason, lost_reason_note = p_note
   where id = p_deal_id returning * into d;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;

  update leads.tasks set status = 'dismissed', completed_at = now()
   where deal_id = p_deal_id and status not in ('done','dismissed');

  insert into leads.deal_events (deal_id, kind, actor, payload)
  values (p_deal_id, 'deal_lost', leads.sop_actor(),
          jsonb_build_object('reason', p_reason, 'note', p_note));
  return jsonb_build_object('ok', true, 'status', 'lost', 'reason', p_reason);
end 
```

### deal_park(p_deal_id uuid, p_until date, p_reason text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 25 lines
```sql

declare d leads.deals; adj date := p_until; cal_status text;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  if p_until is null or p_until <= current_date then
    raise exception 'SOP 2.2: Parked needs a reactivation date in the future.';
  end if;
  select * into d from leads.deals where id = p_deal_id;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;

  for i in 0..11 loop
    select status into cal_status from leads.sop_academic_calendar
     where region = d.region and month = extract(month from adj)::smallint;
    exit when cal_status is null or cal_status = 'active';
    adj := (date_trunc('month', adj) + interval '1 month')::date;
  end loop;

  update leads.deals set status = 'parked', parked_until = adj, parked_reason = p_reason
   where id = p_deal_id;

  insert into leads.deal_events (deal_id, kind, actor, payload)
  values (p_deal_id, 'deal_parked', leads.sop_actor(),
          jsonb_build_object('until', adj, 'requested', p_until, 'reason', p_reason));
  return jsonb_build_object('ok', true, 'parked_until', adj, 'moved_for_break', adj <> p_until);
end 
```

### deal_post_delivery_tick(p_deal_id uuid, p_item text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 14 lines
```sql

declare pd jsonb;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  if p_item not in ('feedback','testimonial','repeat_ask','referral_ask','repeat_record') then
    raise exception 'unknown post-delivery item %', p_item;
  end if;
  update leads.deals set post_delivery = post_delivery || jsonb_build_object(p_item, true)
   where id = p_deal_id returning post_delivery into pd;
  if pd is null then raise exception 'unknown deal %', p_deal_id; end if;

  if p_item = 'repeat_ask' then perform leads.deal_create_repeat(p_deal_id); end if;
  return jsonb_build_object('ok', true, 'post_delivery', pd);
end 
```

### deal_promise_asset(p_deal_id uuid, p_what text, p_hours integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 23 lines
```sql

declare d leads.deals; t uuid;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  select * into d from leads.deals where id = p_deal_id;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;

  insert into leads.tasks (type, sop_kind, title, description, deal_id, assigned_to,
                           due_at, due_date, priority, status, created_by_agent, dedupe_key)
  values ('manual', 'promised_asset',
          p_what || ' — promised to ' || d.school_name,
          'Promised on a call. SOP 3.3.3 gives this a 48-hour deadline.',
          p_deal_id, coalesce(d.owner, leads.sop_actor()),
          now() + make_interval(hours => p_hours),
          (now() + make_interval(hours => p_hours))::date,
          'high', 'pending', false,
          'promise:' || p_deal_id::text || ':' || md5(p_what))
  on conflict (dedupe_key) do nothing
  returning id into t;

  return jsonb_build_object('ok', true, 'task_id', t,
                            'due_at', now() + make_interval(hours => p_hours));
end 
```

### deal_reactivate(p_deal_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 11 lines
```sql

declare d leads.deals;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  update leads.deals set status = 'active' where id = p_deal_id and status = 'parked'
  returning * into d;
  if d.id is null then raise exception 'deal % is not parked', p_deal_id; end if;
  insert into leads.deal_events (deal_id, kind, actor, payload)
  values (p_deal_id, 'deal_reactivated', leads.sop_actor(), jsonb_build_object('stage', d.stage));
  return jsonb_build_object('ok', true, 'stage', d.stage);
end 
```

### deal_record_gate(p_deal_id uuid, p_decision text, p_note text) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 26 lines
```sql

declare d leads.deals;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  select * into d from leads.deals where id = p_deal_id;
  if d.id is null then raise exception 'unknown deal %', p_deal_id; end if;
  if d.registered is null then
    raise exception 'SOP 5.1.4: no programme runs past the gate without a number. Log a count first.';
  end if;

  update leads.deals
     set gate_decision = p_decision, gate_decision_at = current_date, gate_decision_note = p_note
   where id = p_deal_id;

  -- A postponed programme is parked, not lost: it has a named future date.
  if p_decision = 'postponed' then
    perform leads.deal_park(p_deal_id, coalesce(d.programme_start, current_date) + 90,
                            coalesce(p_note, 'Postponed at the 45-day gate'));
  end if;

  insert into leads.deal_events (deal_id, kind, actor, payload)
  values (p_deal_id, 'gate_decided', leads.sop_actor(),
          jsonb_build_object('decision', p_decision, 'registered', d.registered,
                             'minimum', d.minimum_required, 'note', p_note));
  return jsonb_build_object('ok', true, 'decision', p_decision);
end 
```

### deal_send_proposal(p_deal_id uuid, p_stream text, p_sent date, p_expiry_days integer) -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 24 lines
```sql

declare sent date := coalesce(p_sent, current_date); d leads.deals;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  select * into d from leads.deals where id = p_deal_id and status = 'active';
  if d.id is null then raise exception 'deal % is not an active deal', p_deal_id; end if;
  if coalesce(p_stream, d.stream) is null then
    raise exception 'SOP 2.1: tag the primary stream when the proposal is sent.';
  end if;

  update leads.deals
     set proposal_sent_date = sent,
         proposal_expiry = sent + p_expiry_days,
         stream = coalesce(p_stream, stream),
         stage = greatest(stage, 2),
         last_touch_at = now(),
         touch_count = touch_count + 1
   where id = p_deal_id;

  insert into leads.deal_events (deal_id, kind, channel, actor, payload)
  values (p_deal_id, 'proposal_sent', 'email', leads.sop_actor(),
          jsonb_build_object('expiry', sent + p_expiry_days));
  return jsonb_build_object('ok', true, 'proposal_expiry', sent + p_expiry_days);
end 
```

### deals_flag_price_conflict() -> trigger
- plpgsql | invoker | volatile | config: search_path=leads, public | exec anon=Y authenticated=Y | 16 lines
```sql

begin
  if new.price_quoted_minor is not null and exists (
    select 1 from leads.deals x
     where x.id <> new.id
       and coalesce(x.school_code, lower(x.school_name))
           = coalesce(new.school_code, lower(new.school_name))
       and coalesce(x.programme,'') = coalesce(new.programme,'')
       and x.price_quoted_minor is not null
       and x.price_quoted_minor <> new.price_quoted_minor
       and x.created_at > now() - interval '365 days')
  then
    new.needs_review := true;
  end if;
  return new;
end 
```

### deals_stage_stamp() -> trigger
- plpgsql | invoker | volatile | config: search_path=leads, public | exec anon=Y authenticated=Y | 80 lines
```sql

declare actor uuid := leads.sop_actor();
begin
  new.updated_at := now();

  -- Inserts only stamp. Historical records (SOP section 10) must be allowed to
  -- land at whatever stage they had already reached.
  if tg_op = 'INSERT' then
    new.stage_entered_at := coalesce(new.stage_entered_at, now());
    new.stage_history := jsonb_build_array(jsonb_build_object(
      'stage', new.stage, 'status', new.status, 'at', new.stage_entered_at, 'by', actor));
    return new;
  end if;

  if new.stage is not distinct from old.stage
-- ... truncated (80 lines total)
```

### derive_lead_segment(_designation text) -> text
- sql | invoker | immutable | config: search_path=leads, public, extensions | exec anon=Y authenticated=Y | 10 lines
```sql

  SELECT CASE
    WHEN _designation IS NULL OR btrim(_designation) = '' THEN 'unknown'
    WHEN lower(_designation) IN ('student') THEN 'student'
    WHEN lower(_designation) IN ('parent','guardian','mother','father') THEN 'parent'
    WHEN lower(_designation) ~ '(principal|representative|coordinator|teacher|founder|director|head|owner|dean|admin|counsell?or|leader|manager)'
      THEN 'school'
    ELSE 'unknown'
  END

```

### ensure_utm_direct() -> trigger
- plpgsql | invoker | volatile | config: search_path=leads, public, extensions | exec anon=Y authenticated=Y | 8 lines
```sql

BEGIN
  NEW.utm_source   := COALESCE(NULLIF(TRIM(NEW.utm_source),   ''), 'direct');
  NEW.utm_medium   := COALESCE(NULLIF(TRIM(NEW.utm_medium),   ''), 'direct');
  NEW.utm_campaign := COALESCE(NULLIF(TRIM(NEW.utm_campaign), ''), 'direct');
  RETURN NEW;
END;

```

### get_invitation_by_token(p_token text) -> TABLE(email text, role leads.app_role, full_name text, title text, expires_at timestamp with time zone, accepted boolean)
- sql | SECURITY DEFINER | stable | config: search_path=leads, public, extensions | exec anon=Y authenticated=Y | 6 lines
```sql

  SELECT i.email, i.role, i.full_name, i.title, i.expires_at, i.accepted
  FROM leads.invitations i
  WHERE i.token = p_token
  LIMIT 1;

```

### guard_member_meeting_update() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 33 lines
```sql

DECLARE
  caller uuid := auth.uid();
  is_admin boolean;
  allowed_cols text[] := ARRAY[
    'meeting_notes','personal_notes','status','attendance_status',
    'updated_at'
  ];
BEGIN
  IF caller IS NULL THEN RETURN NEW; END IF;
  SELECT leads.is_admin_or_owner(caller) INTO is_admin;
  IF is_admin THEN RETURN NEW; END IF;

  IF NEW.conversion_status      IS DISTINCT FROM OLD.conversion_status      THEN RAISE EXCEPTION 'Not allowed to change conversion_status'; END IF;
  IF NEW.paid_student_count     IS DISTINCT FROM OLD.paid_student_count     THEN RAISE EXCEPTION 'Not allowed to change paid_student_count'; END IF;
  IF NEW.applications_count     IS DISTINCT FROM OLD.applications_count     THEN RAISE EXCEPTION 'Not allowed to change applications_count'; END IF;
  IF NEW.ai_notes               IS DISTINCT FROM OLD.ai_notes               THEN RAISE EXCEPTION 'Not allowed to change ai_notes'; END IF;
  IF NEW.transcript_text        IS DISTINCT FROM OLD.transcript_text        THEN RAISE EXCEPTION 'Not allowed to change transcript_text'; END IF;
  IF NEW.escalation_needed      IS DISTINCT FROM OLD.escalation_needed      THEN RAISE EXCEPTION 'Not allowed to change escalation_needed'; END IF;
  IF NEW.calendly_raw_payload   IS DISTINCT FROM OLD.calendly_raw_payload   THEN RAISE EXCEPTION 'Not allowed to change calendly_raw_payload'; END IF;
  IF NEW.lifecycle_stage        IS DISTINCT FROM OLD.lifecycle_stage        THEN RAISE EXCEPTION 'Not allowed to change lifecycle_stage'; END IF;
  IF NEW.meeting_status         IS DISTINCT FROM OLD.meeting_status         THEN RAISE EXCEPTION 'Not allowed to change meeting_status'; END IF;
  IF NEW.interest_level         IS DISTINCT FROM OLD.interest_level         THEN RAISE EXCEPTION 'Not allowed to change interest_level'; END IF;
  IF NEW.assigned_to            IS DISTINCT FROM OLD.assigned_to            THEN RAISE EXCEPTION 'Not allowed to change assigned_to'; END IF;
  IF NEW.email                  IS DISTINCT FROM OLD.email                  THEN RAISE EXCEPTION 'Not allowed to change email'; END IF;
  IF NEW.full_name              IS DISTINCT FROM OLD.full_name              THEN RAISE EXCEPTION 'Not allowed to change full_name'; END IF;
  IF NEW.school_name            IS DISTINCT FROM OLD.school_name            THEN RAISE EXCEPTION 'Not allowed to change school_name'; END IF;
  IF NEW.meeting_date           IS DISTINCT FROM OLD.meeting_date           THEN RAISE EXCEPTION 'Not allowed to change meeting_date'; END IF;
  IF NEW.scheduled_time         IS DISTINCT FROM OLD.scheduled_time         THEN RAISE EXCEPTION 'Not allowed to change scheduled_time'; END IF;
  IF NEW.calendly_event_uri     IS DISTINCT FROM OLD.calendly_event_uri     THEN RAISE EXCEPTION 'Not allowed to change calendly_event_uri'; END IF;
  RETURN NEW;
END;

```

### guard_profile_role_change() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 20 lines
```sql

DECLARE
  caller uuid := auth.uid();
  caller_is_admin boolean;
BEGIN
  IF NEW.role IS DISTINCT FROM OLD.role OR NEW.is_active IS DISTINCT FROM OLD.is_active THEN
    IF caller IS NULL THEN
      RETURN NEW;
    END IF;
    SELECT leads.is_admin_or_owner(caller) INTO caller_is_admin;
    IF NOT caller_is_admin THEN
      RAISE EXCEPTION 'Only admins or owners can change role or active status';
    END IF;
    IF OLD.role = 'owner' AND NEW.role <> 'owner' AND caller = OLD.id THEN
      RAISE EXCEPTION 'Owners cannot demote themselves';
    END IF;
  END IF;
  RETURN NEW;
END;

```

### handle_new_user() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 12 lines
```sql

BEGIN
  INSERT INTO leads.user_profiles (id, email, full_name)
  VALUES (
    NEW.id,
    NEW.email,
    COALESCE(NEW.raw_user_meta_data->>'full_name', split_part(NEW.email, '@', 1))
  )
  ON CONFLICT (id) DO NOTHING;
  RETURN NEW;
END;

```

### has_role(_user_id uuid, _role leads.app_role) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 6 lines
```sql

  SELECT EXISTS (
    SELECT 1 FROM leads.user_profiles
    WHERE id = _user_id AND role = _role
  )

```

### is_active_member(_user_id uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=leads | exec anon=n authenticated=Y | 3 lines
```sql

  SELECT EXISTS (SELECT 1 FROM leads.user_profiles WHERE id = _user_id AND is_active = true);

```

### is_admin_or_owner(_user_id uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 6 lines
```sql

  SELECT EXISTS (
    SELECT 1 FROM leads.user_profiles
    WHERE id = _user_id AND role IN ('owner','admin') AND is_active = true
  )

```

### is_assigned_to_meeting(_meeting_id uuid, _user_id uuid) -> boolean
- sql | SECURITY DEFINER | stable | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 6 lines
```sql

  SELECT EXISTS (
    SELECT 1 FROM leads.meeting_assignees
    WHERE meeting_id = _meeting_id AND user_id = _user_id
  )

```

### notify_on_assignment() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 37 lines
```sql

DECLARE
  v_school TEXT;
  v_date   DATE;
  v_time   TIME;
  v_by     TEXT;
  v_by_uuid UUID;
BEGIN
  SELECT school_name, meeting_date, scheduled_time
    INTO v_school, v_date, v_time
    FROM leads.meetings WHERE id = NEW.meeting_id;

  IF NEW.assigned_by IS NOT NULL AND NEW.assigned_by <> '' THEN
    BEGIN
      v_by_uuid := NEW.assigned_by::uuid;
      SELECT full_name INTO v_by FROM leads.user_profiles WHERE id = v_by_uuid;
    EXCEPTION WHEN invalid_text_representation THEN
      v_by := NEW.assigned_by;
    END;
  END IF;

  INSERT INTO leads.notifications (user_id, type, title, body, meeting_id)
  VALUES (
    NEW.user_id,
    'meeting_assigned',
    COALESCE('New meeting assigned — ' || NULLIF(v_school,''), 'New meeting assigned'),
    COALESCE(
      to_char(v_date, 'DD Mon')
        || COALESCE(' at ' || to_char(v_time, 'HH24:MI'), '')
        || COALESCE(' · assigned by ' || v_by, ''),
      'You have been assigned to a meeting.'
    ),
    NEW.meeting_id
  );
  RETURN NEW;
END

```

### propagate_assignee_to_tasks() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 10 lines
```sql

BEGIN
  UPDATE leads.tasks
     SET assigned_to = NEW.user_id
   WHERE meeting_id = NEW.meeting_id
     AND assigned_to IS NULL
     AND status IN ('open','pending','snoozed','needs_info','revision_requested');
  RETURN NEW;
END;

```

### sop_actor() -> uuid
- sql | SECURITY DEFINER | stable | config: search_path=leads, public | exec anon=n authenticated=Y | 3 lines
```sql

  select p.id from leads.user_profiles p where p.id = auth.uid() and p.is_active

```

### sop_campaign_tag_ok(p_tag text) -> boolean
- sql | invoker | immutable | config: search_path=leads, public | exec anon=n authenticated=Y | 3 lines
```sql

  select p_tag is not null and p_tag ~ '^[a-z0-9_]+(-[a-z0-9_]+){3}$';

```

### sop_daily_board(p_user_id uuid, p_all boolean) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=leads, public | exec anon=n authenticated=n | 129 lines
- comment: SOP 7.1 employee view: what do I do today.
```sql

declare me uuid; res jsonb;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  if p_all then
    if not leads.is_admin_or_owner(auth.uid()) then
      raise exception 'only an admin or owner sees every owner''s board';
    end if;
    me := null;
  else
    me := coalesce(p_user_id, leads.sop_actor());
  end if;

  select jsonb_build_object(
    'as_of', current_date, 'user_id', me,
-- ... truncated (129 lines total)
```

### sop_data_quality() -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=leads, public | exec anon=n authenticated=n | 125 lines
- comment: SOP 7.3 admin view: missing fields, duplicates, bad tags, stale records, campaign performance.
```sql

declare res jsonb;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;

  with fields as (
    select d.id, d.owner, f.field, f.missing
      from leads.deals d
      cross join lateral (values
        ('decision_maker_name', d.stage >= 1 and coalesce(d.decision_maker_name,'') = ''),
        ('next_action_due',     d.next_action is not null and d.next_action_due is null),
        ('programme',           d.programme is null),
        ('stream',              d.stage >= 2 and d.stream is null),
        ('expected_students',   d.stage >= 2 and d.expected_students is null),
        ('price_quoted',        d.stage >= 2 and d.price_quoted_minor is null),
-- ... truncated (125 lines total)
```

### sop_deal_record(p_deal_id uuid) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=leads, public | exec anon=n authenticated=n | 88 lines
- comment: SOP section 6 — one lead record, with its gate, timeline and what blocks the next stage.
```sql

declare res jsonb;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;

  select jsonb_build_object(
    'deal', to_jsonb(d) - 'stage_history',
    'rag', leads.sop_touch_rag(d.last_touch_at, d.created_at),
    'days_since_touch', extract(day from now() - coalesce(d.last_touch_at, d.created_at))::int,
    'campaign_tag_valid', leads.sop_campaign_tag_ok(d.campaign_tag),
    'owner', (select jsonb_build_object('id', u.id, 'name', u.full_name, 'email', u.email)
                from leads.user_profiles u where u.id = d.owner),
    'school', (select jsonb_build_object('code', s.code, 'name', s.canonical_name,
                                         'country', s.country, 'region', s.region)
                 from leads.school_registry s where s.code = d.school_code),
-- ... truncated (88 lines total)
```

### sop_manager_review(p_from date, p_to date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=leads, public | exec anon=n authenticated=n | 176 lines
- comment: SOP 7.2 manager view: what moved stage, what is red, what is due, what did we lose and why.
```sql

declare
  v_to      date := coalesce(p_to, current_date);
  v_from    date := coalesce(p_from, coalesce(p_to, current_date) - 29);
  v_q_start date := date_trunc('quarter', coalesce(p_to, current_date))::date;
  v_m_start date := date_trunc('month',  coalesce(p_to, current_date))::date;
  v_quarter text := to_char(coalesce(p_to, current_date), 'YYYY') || 'Q'
                    || extract(quarter from coalesce(p_to, current_date))::text;
  res jsonb;
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;

  select jsonb_build_object(
    'window', jsonb_build_object('from', v_from, 'to', v_to, 'quarter', v_quarter),

-- ... truncated (176 lines total)
```

### sop_map_designation(p_raw text) -> text
- sql | invoker | immutable | config: search_path=leads, public | exec anon=n authenticated=Y | 14 lines
```sql

  select case
    when p_raw is null then null
    when p_raw ilike 'principal%'                              then 'principal'
    when p_raw ilike '%head of school%' or p_raw ilike 'head%' then 'head_of_school'
    when p_raw ilike '%counsel%'                               then 'counsellor'
    when p_raw ilike 'school rep%' or p_raw ilike '%teacher%'
      or p_raw ilike '%coordinator%'                           then 'teacher_coordinator'
    when p_raw ilike 'parent%'                                 then 'parent'
    when p_raw ilike 'student%'                                then 'student'
    when p_raw ilike 'universit%'                              then 'university'
    when p_raw ilike 'partner%' or p_raw ilike 'corporate%'    then 'partner_agency'
    else null end;

```

### sop_map_programme(p_raw text) -> text
- sql | invoker | immutable | config: search_path=leads, public | exec anon=n authenticated=Y | 19 lines
```sql

  select case
    when p_raw is null then null
    when p_raw ilike '%fellowship%'                                   then 'onwards'
    when p_raw ilike '%capital minds%' or p_raw ilike '%finedu%'  then 'fellowship_capital_minds'
    when p_raw ilike '%hcus%' or p_raw ilike '%harvard summer%'   then 'hcus_day_camp'
    when p_raw ilike '%lawyer%'                                   then 'fellowship_lawyers'
    when p_raw ilike '%testprep%'                                      then 'sat_instinct'
    when p_raw ilike '%hpair%'                                    then 'hpair_ai'
    when p_raw ilike '%hwics%'                                    then 'hwics'
    when p_raw ilike '%nyu%'                                      then 'nyu_entrepreneurship'
    when p_raw ilike '%doctor%'                                   then 'fellowship_doctors'
    when p_raw ilike '%research%'                                 then 'fellowship_research'
    when p_raw ilike '%hmun%'                                     then 'hmun'
    when p_raw ilike '%cambridge%'                                then 'cambridge_premed'
    when p_raw ilike '%geneva%'                                   then 'geneva'
    when p_raw ilike '%salamanca%'                                then 'salamanca'
    else null end;

```

### sop_meeting_owner(p_meeting_id uuid) -> uuid
- sql | invoker | stable | config: search_path=leads, public | exec anon=n authenticated=Y | 7 lines
```sql

  select coalesce(
    (select ma.user_id from leads.meeting_assignees ma
      where ma.meeting_id = p_meeting_id order by ma.assigned_at limit 1),
    (select d.owner from leads.meetings m join leads.deals d on d.id = m.deal_id
      where m.id = p_meeting_id));

```

### sop_metrics(p_from date, p_to date) -> jsonb
- plpgsql | SECURITY DEFINER | stable | config: search_path=leads, public | exec anon=n authenticated=Y | 42 lines
- comment: SOP 7.4: done rate, proposal, qualified, win, cycle time, revenue per meeting, fill rate.
```sql

declare
  d_to date := coalesce(p_to, current_date);
  d_from date := coalesce(p_from, coalesce(p_to, current_date) - 89);
begin
  if leads.sop_actor() is null then raise exception 'not authorised'; end if;
  return (select jsonb_build_object(
    'window', jsonb_build_object('from', d_from, 'to', d_to),
    'meeting_done_rate', (
      select case when count(*) filter (where meeting_status in ('Completed','No Show','Cancelled')) = 0 then null
             else round(100.0*count(*) filter (where meeting_status='Completed')
                        / count(*) filter (where meeting_status in ('Completed','No Show','Cancelled'))) end
      from leads.meetings where meeting_date between d_from and d_to),
    'by_stream', (
      select coalesce(jsonb_object_agg(s.k, s.m), '{}'::jsonb) from (
-- ... truncated (42 lines total)
```

### sop_routes_to(p_designation text) -> text
- sql | invoker | stable | config: search_path=leads, public | exec anon=n authenticated=Y | 5 lines
```sql

  select coalesce((select l.extra->>'routes_to' from leads.sop_lists l
                    where l.list = 'designation' and l.value = leads.sop_map_designation(p_designation)),
                  'unknown');

```

### sop_sweep() -> jsonb
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public | exec anon=n authenticated=n | 213 lines
- comment: SOP v1.0 nightly sweeper. Idempotent through leads.tasks.dedupe_key. Scheduled daily.
```sql

declare made int := 0; n int := 0; woke int := 0; det jsonb := '{}'::jsonb;
begin
  with c(day_no, label, prio) as (
    values (4,  'Day 4 touch: short check-in, one question', 'med'),
           (10, 'Day 10 touch: second email with one new piece of value', 'med'),
           (18, 'Day 18 touch: direct ask for a yes, no or a date', 'high')
  ), ins as (
    insert into leads.tasks (type, sop_kind, title, description, deal_id, assigned_to,
                             due_at, due_date, priority, status, created_by_agent, dedupe_key)
    select 'schedule_followup', 'cadence_day' || c.day_no,
           c.label || ' — ' || d.school_name,
           'SOP 3.4 follow-up cadence, day ' || c.day_no || ' after the proposal.',
           d.id, d.owner, (d.proposal_sent_date + c.day_no)::timestamptz,
           (d.proposal_sent_date + c.day_no), c.prio::leads.task_priority, 'pending', true,
-- ... truncated (213 lines total)
```

### sop_touch_rag(p_last timestamp with time zone, p_created timestamp with time zone) -> text
- sql | invoker | immutable | config: search_path=leads, public | exec anon=n authenticated=Y | 6 lines
```sql

  select case
    when extract(day from now() - coalesce(p_last, p_created))::int >= 14 then 'red'
    when extract(day from now() - coalesce(p_last, p_created))::int >= 8  then 'amber'
    else 'green' end;

```

### sync_lead_segment_from_designation() -> trigger
- plpgsql | invoker | volatile | config: search_path=leads, public, extensions | exec anon=Y authenticated=Y | 9 lines
```sql

BEGIN
  IF TG_OP = 'INSERT' OR NEW.designation IS DISTINCT FROM OLD.designation THEN
    IF NEW.lead_segment IS NULL OR NEW.lead_segment = '' OR NEW.lead_segment = 'unknown' THEN
      NEW.lead_segment := leads.derive_lead_segment(NEW.designation);
    END IF;
  END IF;
  RETURN NEW;
END 
```

### sync_lifecycle_from_status() -> trigger
- plpgsql | invoker | volatile | config: search_path=leads, public, extensions | exec anon=Y authenticated=Y | 22 lines
```sql

BEGIN
  IF NEW.meeting_status = 'Completed'
     AND OLD.meeting_status IS DISTINCT FROM 'Completed'
     AND NEW.lifecycle_stage IN ('new_lead', 'meeting_scheduled') THEN
    NEW.lifecycle_stage = 'meeting_done';
  END IF;

  IF NEW.meeting_status = 'Scheduled'
     AND NEW.lifecycle_stage = 'new_lead' THEN
    NEW.lifecycle_stage = 'meeting_scheduled';
  END IF;

  IF NEW.conversion_status = 'Converted'
     AND (OLD.conversion_status IS NULL
          OR OLD.conversion_status <> 'Converted') THEN
    NEW.lifecycle_stage = 'converted';
  END IF;

  RETURN NEW;
END;

```

### tasks_on_email_draft() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 61 lines
```sql

DECLARE
  v_assignee_id  uuid;
  v_assignee_nm  text;
  v_school       text;
  v_lead         text;
  v_md           date;
  v_prio         task_priority;
  v_due          date;
  v_type         task_type;
  v_title        text;
BEGIN
  IF NEW.status = 'draft' AND COALESCE(NEW.direction,'outbound') = 'outbound' THEN
    SELECT m.school_name, m.full_name, m.meeting_date, m.assigned_to
      INTO v_school, v_lead, v_md, v_assignee_nm
-- ... truncated (61 lines total)
```

### tasks_resolve_on_email_change() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 11 lines
```sql

BEGIN
  IF OLD.status = 'draft' AND NEW.status <> 'draft' THEN
    UPDATE leads.tasks
       SET status = 'done', completed_at = now()
     WHERE email_send_id = NEW.id
       AND status IN ('open','pending');
  END IF;
  RETURN NEW;
END

```

### tasks_resolve_on_email_delete() -> trigger
- plpgsql | SECURITY DEFINER | volatile | config: search_path=leads, public, extensions | exec anon=n authenticated=Y | 8 lines
```sql

BEGIN
  UPDATE leads.tasks
     SET status='dismissed', completed_at=now()
   WHERE email_send_id = OLD.id
     AND status IN ('open','pending','snoozed');
  RETURN OLD;
END 
```

### tasks_set_completed_at() -> trigger
- plpgsql | invoker | volatile | config: search_path=leads, public, extensions | exec anon=Y authenticated=Y | 13 lines
```sql

BEGIN
  IF NEW.status IN ('done','dismissed') AND OLD.status IN ('open','pending','snoozed') THEN
    NEW.completed_at := now();
    IF NEW.completed_by IS NULL THEN NEW.completed_by := auth.uid(); END IF;
  END IF;
  IF NEW.status IN ('open','pending') AND OLD.status NOT IN ('open','pending') THEN
    NEW.completed_at := NULL;
    NEW.completed_by := NULL;
  END IF;
  RETURN NEW;
END

```

### update_updated_at_column() -> trigger
- plpgsql | invoker | volatile | config: search_path=leads, public, extensions | exec anon=Y authenticated=Y | 6 lines
```sql

BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;

```

## Schema: migration (11 functions)

### probe_bridge() -> TABLE(source_table text, readable boolean, row_count bigint, err text)
- plpgsql | invoker | volatile | config: search_path=migration, public, extensions, pg_temp | exec anon=Y authenticated=Y | 23 lines
- comment: Per-table readability check across the legacy_ro FDW bridge. Reports which source tables the mapped remote role can actually SELECT, without one permission error aborting the whole query.
```sql

declare
  t text;
  n bigint;
begin
  for t in
    select c.relname
      from pg_class c
      join pg_namespace ns on ns.oid = c.relnamespace
     where ns.nspname = 'legacy_ro'
       and c.relkind = 'f'
       and (c.relname like '%_applications' or c.relname like '%payments')
     order by c.relname
  loop
    begin
      execute format('select count(*) from legacy_ro.%I', t) into n;
      return query select t, true, n, null::text;
    exception when others then
      return query select t, false, null::bigint, sqlerrm;
    end;
  end loop;
end;

```

### razorpay_id_time(p_id text) -> timestamp with time zone
- sql | invoker | immutable | config: search_path=migration, public, extensions, pg_temp | exec anon=Y authenticated=Y | 4 lines
- comment: Creation time encoded in a Razorpay payment id: first 10 characters after pay_, base62, nanoseconds since 2014-01-01 UTC. Checked 3 Oct 2026 against 1,127 payments whose stored date is unambiguous: within 14 s.
```sql

      select case when p_id ~ '^pay_[0-9A-Za-z]{14}$' then
        timestamptz '2014-01-01 00:00:00+00' + ((select sum((strpos('0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz', substr(p_id, 4 + i, 1)) - 1)::numeric * 62::numeric ^ (10 - i))
                                                  from generate_series(1, 10) i) / 1000000000) * interval '1 second' end 
```

### razorpay_true_local(p_id text, p_stored timestamp without time zone) -> timestamp without time zone
- plpgsql | invoker | immutable | config: search_path=migration, public, extensions, pg_temp | exec anon=Y authenticated=Y | 12 lines
- comment: True IST time of a legacy Razorpay payment: the stored value if the payment id confirms it, else the legacy importer's mix-up undone (minus 5h30, day and month swapped back) if the id confirms that. Used by migration.sync_razorpay_payments since 3 Oct 2026.
```sql

    declare t timestamptz := migration.razorpay_id_time(p_id); sh timestamp := p_stored - interval '5 hours 30 minutes'; fx timestamp;
    begin
      if t is null or p_stored is null then return p_stored; end if;
      if abs(extract(epoch from (p_stored at time zone 'Asia/Kolkata') - t)) <= 120 then return p_stored; end if;
      if extract(day from sh) <= 12 then
        fx := make_timestamp(extract(year from sh)::int, extract(day from sh)::int, extract(month from sh)::int,
                             extract(hour from sh)::int, extract(minute from sh)::int, extract(second from sh)::float8);
        if abs(extract(epoch from (fx at time zone 'Asia/Kolkata') - t)) <= 120 then return fx; end if;
      end if;
      return p_stored;  -- neither reading matches the id: keep what legacy says
    end 
```

### resolve_payment_id(p_id text) -> text
- sql | invoker | stable | config: search_path=public | exec anon=Y authenticated=Y | 10 lines
```sql

  with x as (select nullif(btrim(split_part(coalesce(p_id,''), ',', 1)), '') id)
  select case
    when x.id is null then null
    when exists (select 1 from public.payments p where p.provider_payment_id = x.id) then x.id
    when x.id ~ '^pi_[A-Za-z0-9]{24}[0-9]{1,3}$'
     and exists (select 1 from public.payments p where p.provider_payment_id = left(x.id, 27)) then left(x.id, 27)
    else x.id end
  from x

```

### sync_applications(p_force boolean) -> TABLE(rows_seen bigint, rows_changed bigint, table_total bigint)
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, migration, legacy_ro | exec anon=Y authenticated=Y | 67 lines
```sql

declare v_seen bigint; v_changed bigint; v_total bigint;
begin
  select count(*) into v_seen from migration.v_applications_all;

  insert into public.applications as a (
    source_table, source_row_id, programme_code, cohort_raw,
    student_name, student_email, student_phone, student_grade,
    parent_name, parent_email, parent_phone,
    school_raw, city, state, country,
    applied_at, paid_at, payment_status_raw, provider_payment_id,
    payment_provider_hint,
    utm_source, utm_medium, utm_campaign, referral_source,
    raw, needs_review, synced_at)
  select v.source_table, v.source_row_id, v.programme_code, v.cohort_raw,
-- ... truncated (67 lines total)
```

### sync_bank_refunds() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, migration, legacy_ro | exec anon=Y authenticated=Y | 51 lines
```sql

declare v int;
begin
  with w as (
    select id, nullif(btrim(program_code),'') code, upper(btrim(currency_buy)) cur,
           round(nullif(btrim(amount_buy),'')::numeric * 100)::bigint amt,
           to_date(left(coalesce(nullif(completed_date,''), creation_date), 11), 'DD Mon YYYY') on_date,
           nullif(btrim(beneficiary_name),'') bname, lower(regexp_replace(translate(coalesce(beneficiary_name,''), 'áàâäãåéèêëíìîïóòôöõúùûüçñ', 'aaaaaaeeeeiiiiooooouuuucn'), '[^a-zA-Z ]', '', 'g')) nm
      from legacy_ro.finance_wallex_transactions
     where nature_of_expense ilike 'refund' and status ilike 'completed' and nullif(btrim(amount_buy),'') is not null),
  m as (
    select w.id, array_agg(p.id order by p.occurred_at, p.id) pids
      from w join public.payments p on p.currency = w.cur and p.programme_code = w.code and p.amount_minor >= w.amt
                                   and lower(regexp_replace(translate(coalesce(p.payer_name,''), 'áàâäãåéèêëíìîïóòôöõúùûüçñ', 'aaaaaaeeeeiiiiooooouuuucn'), '[^a-zA-Z ]', '', 'g')) = w.nm and w.nm <> ''
     group by w.id)
-- ... truncated (51 lines total)
```

### sync_dbs_payments() -> TABLE(rows_seen bigint, rows_changed bigint, provider_total bigint)
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, migration, legacy_ro | exec anon=Y authenticated=Y | 58 lines
```sql

declare v_seen bigint; v_changed bigint; v_total bigint;
begin
  select count(*) into v_seen from legacy_ro.dbs_transactions;
  with src as (
    select t.*,
           -- One key per bank transaction: the bank's transaction id; without one,
           -- the transfer's own facts. Not stable_row_id: legacy reuses it across
           -- different transactions (S$17,043.65 on 4 Sep and S$10,440 on 22 Sep share one).
           coalesce(nullif(btrim(t.transaction_id),''),
                    'nokey:' || coalesce(t.completed_date, t.initiated_date)::text || '|' || t.amount::text || '|' || t.currency
                      || '|' || coalesce(nullif(btrim(t.bank_ref),''),'-') || '|' || coalesce(t.transaction_type,'-')) txn_key
      from legacy_ro.dbs_transactions t
     where t.amount > 0 and t.currency ~ '^[A-Z]{3}$'
       and coalesce(t.sender_name,'') not ilike 'STRIPE PAYMENTS%'
-- ... truncated (58 lines total)
```

### sync_programme_codes() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, migration, legacy_ro | exec anon=Y authenticated=Y | 30 lines
```sql

declare v int;
begin
  with d as (select ref_type, btrim(code) code, max(name) nm from legacy_ro.codes_dropdown group by 1,2)
  insert into public.programme_codes as t (code, category_code, category_name, organisation_code, organisation_name,
      programme_part, programme_name, school_code, school_name, country, batch, starts_on, synced_at)
  select btrim(p.program_code), p.category, (select nm from d where ref_type='category' and code=p.category),
         p.organisation, (select nm from d where ref_type='organisation' and code=p.organisation),
         p.program, (select nm from d where ref_type='program' and code=p.program),
         p.school, (select nm from d where ref_type='school' and code=p.school),
         nullif(btrim(p.country),''), nullif(btrim(p.batch),''),
         case when p.date ~ '^\d{4}-\d{2}-\d{2}$' then p.date::date end, now()
    from legacy_ro.program_codes p where nullif(btrim(p.program_code),'') is not null
  on conflict (code) do update set
    category_code=excluded.category_code, category_name=excluded.category_name,
    organisation_code=excluded.organisation_code, organisation_name=excluded.organisation_name,
    programme_part=excluded.programme_part, programme_name=excluded.programme_name,
    school_code=excluded.school_code, school_name=excluded.school_name,
    country=excluded.country, batch=excluded.batch, starts_on=excluded.starts_on, synced_at=now()
  where (t.category_code, t.category_name, t.organisation_code, t.organisation_name, t.programme_part, t.programme_name,
         t.school_code, t.school_name, t.country, t.batch, t.starts_on)
        is distinct from
        (excluded.category_code, excluded.category_name, excluded.organisation_code, excluded.organisation_name, excluded.programme_part,
         excluded.programme_name, excluded.school_code, excluded.school_name, excluded.country, excluded.batch, excluded.starts_on);
  get diagnostics v = row_count;
  insert into migration.run_log (wave, run_by, rows_in, rows_out, finished_at, notes)
  values ('sync/programme-codes', 'migration.sync_programme_codes', (select count(*) from legacy_ro.program_codes)::int, v, now(),
          format('%s codes changed, %s in catalogue', v, (select count(*) from public.programme_codes)));
  return v;
end 
```

### sync_razorpay_payments(p_force boolean) -> TABLE(rows_seen bigint, rows_changed bigint, provider_total bigint, flagged_future bigint)
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, migration, legacy_ro | exec anon=Y authenticated=Y | 81 lines
```sql

declare v_seen bigint; v_changed bigint; v_total bigint; v_future bigint;
begin
  select count(*) into v_seen from legacy_ro.razorpay_payments;

  insert into public.payments as p (
    provider, provider_payment_id, amount_minor, currency, status, occurred_at,
    description, source_code, external_ids, person_id, needs_review,
    payer_email, method, fee_minor, refunded_minor, programme_code)
  select 'razorpay', r.payment_id,
         round(r.amount * 100)::bigint, upper(r.currency),
         (case when r.status = 'captured' and coalesce(r.amount_refunded,0) >= r.amount
                    and coalesce(r.amount_refunded,0) > 0                then 'refunded'
               when r.status = 'captured' and coalesce(r.amount_refunded,0) > 0
                                                                         then 'partially_refunded'
-- ... truncated (81 lines total)
```

### sync_refunds_from_payments() -> integer
- sql | SECURITY DEFINER | volatile | config: search_path=public, migration | exec anon=Y authenticated=Y | 14 lines
```sql

  -- One refund row for every payment that carries a refunded amount and has none.
  -- The legacy exports give a cumulative refunded figure, not each refund, so this
  -- records the total; occurred_at stays null because no refund date is exported.
  with ins as (
    insert into public.refunds (payment_id, amount_minor, currency, is_full, occurred_at, source_code, external_ids)
    select p.id, p.refunded_minor, p.currency, p.refunded_minor >= p.amount_minor, null, p.source_code,
           jsonb_build_object('derived_by', 'migration.sync_refunds_from_payments', 'from', 'payments.refunded_minor')
      from public.payments p
     where coalesce(p.refunded_minor, 0) > 0
       and not exists (select 1 from public.refunds f where f.payment_id = p.id)
    returning 1)
  select count(*)::int from ins

```

### sync_stripe_payments(p_force boolean) -> TABLE(rows_seen bigint, rows_changed bigint, provider_total bigint)
- plpgsql | SECURITY DEFINER | volatile | config: search_path=public, migration, legacy_ro | exec anon=Y authenticated=Y | 81 lines
```sql

declare v_seen bigint; v_changed bigint; v_total bigint;
begin
  select count(*) into v_seen from legacy_ro.stripe_payments;

  insert into public.payments as p (
    provider, provider_payment_id, amount_minor, currency, status, occurred_at,
    description, source_code, external_ids, person_id,
    payer_email, payer_name, programme_code, utm_source, utm_medium, utm_campaign, refunded_minor)
  select 'stripe', s.payment_intent_id,
         round(s.amount * 100)::bigint, upper(s.currency),
         (case when s.status ilike 'paid (partially refunded%' then 'partially_refunded'
               when s.status ilike 'refunded%'                 then 'refunded'
               when s.status ilike 'paid%'                     then 'paid'
               else 'unknown' end)::public.payment_status,
-- ... truncated (81 lines total)
```

## Schema: quarantine (2 functions)

### refresh_research_schools() -> integer
- plpgsql | SECURITY DEFINER | volatile | config: search_path=quarantine, public, pg_temp | exec anon=Y authenticated=Y | 55 lines
```sql

DECLARE total_rows integer;   -- not `n`: that collided with the column alias below
BEGIN
  WITH per_country AS (
    SELECT lower(btrim(rc.school_name_raw))  AS name_key,
           nullif(btrim(rc.country_raw), '') AS country,
           count(*)                          AS claims
      FROM quarantine.research_contacts rc
     WHERE nullif(btrim(coalesce(rc.school_name_raw, '')), '') IS NOT NULL
     GROUP BY 1, 2
  ), modal AS (
    -- A real country always beats "none recorded", hence (country IS NULL) first in ORDER BY.
    SELECT DISTINCT ON (name_key)
           name_key, country AS modal_country, claims AS modal_hits
      FROM per_country
-- ... truncated (55 lines total)
```

### school_triage(p_name text) -> text
- sql | invoker | immutable | config: search_path=pg_catalog, pg_temp | exec anon=Y authenticated=Y | 17 lines
```sql

  WITH n AS (SELECT lower(btrim(coalesce(p_name, ''))) v)
  SELECT CASE
    WHEN (SELECT v FROM n) = '' THEN 'empty'
    WHEN (SELECT v FROM n) !~ '[a-z]' THEN 'no letters'
    WHEN length((SELECT v FROM n)) < 3 THEN 'too short to identify'
    WHEN (SELECT v FROM n) ~ '^(#?n/?a|na|nil|null|none|unknown|other|others|various|mixed|misc|test|tbd|-+|\.+)$'
      THEN 'placeholder, not a name'
    WHEN (SELECT v FROM n) IN ('gmail','googlemail','yahoo','ymail','hotmail','outlook','live',
         'msn','icloud','me','mac','aol','aim','protonmail','proton','gmx','zoho','yandex','mail',
         'email','qq','163','126','sina','naver','daum','rediffmail','rediff','comcast','verizon',
         'att','sbcglobal','bellsouth','cox','charter','earthlink','btinternet','orange','free',
         'web','t-online','libero','wanadoo','sky','virginmedia')
      THEN 'email provider, not a school'
    ELSE NULL
  END;

```

