# Shared estate functions

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

## affiliate (45 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| admin_add_admin | p_admin_email text, p_email text | affiliate.admins | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_applications | p_admin_email text, p_platform text, p_search text, p_attributed_only boolean, p_limit integer, p_offset integer | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| admin_create_announcement | p_admin_email text, p_kind text, p_title text, p_link_url text | affiliate.announcements | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_create_payout | p_admin_email text, p_partner_id uuid, p_amount numeric, p_currency text, p_period_label text, p_reference text | affiliate.payouts | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_create_resource | p_admin_email text, p_title text, p_type text, p_url text, p_description text, p_featured boolean | affiliate.resources | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_create_school_achievement | p_admin_email text, p_school_id uuid, p_kind text, p_title text, p_description text, p_student_name text, p_program_name text, p_link_url text, p_cover_image_url text, p_featured boolean, p_published_at date | affiliate.school_achievements | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_delete_announcement | p_admin_email text, p_id uuid | boolean | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_delete_school_achievement | p_admin_email text, p_id uuid | boolean | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_list_admins | p_admin_email text | SETOF affiliate.admins | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| admin_list_announcements | p_admin_email text | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| admin_list_partners | p_admin_email text | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| admin_list_payouts | p_admin_email text | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| admin_list_resources | p_admin_email text | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| admin_list_school_achievements | p_admin_email text | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| admin_overview | p_admin_email text | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| admin_remove_admin | p_admin_email text, p_email text | boolean | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_set_school_status | p_admin_email text, p_school_id uuid, p_status text | affiliate.schools | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_update_partner | p_admin_email text, p_partner_id uuid, p_status text, p_rate_camps numeric, p_rate_inperson numeric, p_rate_online numeric, p_discount_cap numeric | affiliate.partners | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_update_payout | p_admin_email text, p_payout_id uuid, p_status text, p_reference text | affiliate.payouts | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_update_resource | p_admin_email text, p_id uuid, p_title text, p_type text, p_url text, p_description text, p_featured boolean | affiliate.resources | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| admin_update_school_achievement | p_admin_email text, p_id uuid, p_kind text, p_title text, p_description text, p_student_name text, p_program_name text, p_link_url text, p_cover_image_url text, p_featured boolean, p_published_at date | affiliate.school_achievements | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| apply_partner | p_auth_user_id uuid, p_full_name text, p_email text, p_channel_handle text, p_channel_type text, p_audience_size text | affiliate.partners | plpgsql | DEFINER | volatile | search_path=affiliate, pg_temp |  |
| auth_user_id_by_email | p_email text | uuid | sql | DEFINER | stable | search_path=affiliate, auth, pg_temp |  |
| commission_rate_for_program | p_format text, p_categories text[] | numeric | sql | invoker | immutable | search_path=affiliate, public, extensions, pg_temp |  |
| create_payment_link | p_auth_user_id uuid, p_program_pay_slug text, p_offer_price numeric, p_slug text, p_max_uses integer, p_expires_at timestamp with time zone | jsonb | plpgsql | DEFINER | volatile | search_path=affiliate, public, portal, pg_temp |  |
| create_promo_code | p_auth_user_id uuid, p_program_pay_slug text, p_discount_type text, p_discount_value numeric, p_code text, p_max_redemptions integer, p_expires_at timestamp with time zone | jsonb | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| get_partner_by_auth_id | p_auth_user_id uuid | affiliate.partners | sql | DEFINER | volatile | search_path=affiliate, pg_temp |  |
| get_school_for_user | p_auth_user_id uuid, p_email text | affiliate.schools | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| invite_school | p_admin_email text, p_school_name text, p_contact_name text, p_email text, p_school_code text | affiliate.schools | plpgsql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| is_admin | p_email text | boolean | sql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| leaderboard |  | TABLE(display_name text, channel_type text, conversions bigint, revenue numeric, currency text) | sql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| list_active_programs |  | TABLE(pay_slug text, program_name text, format text, categories text[], sticker_price integer, early_bird_price integer, price_currency text, rate_percent numeric) | sql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| list_announcements | p_auth_user_id uuid | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| list_my_applications | p_auth_user_id uuid, p_search text, p_limit integer, p_offset integer | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| list_my_payment_links | p_auth_user_id uuid | TABLE(id uuid, slug text, program_name text, program_pay_slug text, list_price numeric, offer_price numeric, currency text, rate_percent numeric, max_uses integer, uses_count integer, is_active boolean, expires_at timestamp with time zone, created_at timestamp with time zone, partner_website text, apply_url text) | sql | DEFINER | volatile | search_path=affiliate, public, portal, pg_temp |  |
| list_my_promo_codes | p_auth_user_id uuid | TABLE(id uuid, code text, program_name text, program_pay_slug text, discount_type text, discount_value numeric, currency text, max_redemptions integer, redemption_count integer, is_active boolean, expires_at timestamp with time zone, created_at timestamp with time zone, rate_percent numeric, list_price numeric) | sql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| list_my_school_programs | p_auth_user_id uuid | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| list_resources |  | TABLE(id uuid, title text, type text, url text, description text, featured boolean, created_at timestamp with time zone) | sql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| list_school_achievements | p_auth_user_id uuid | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| list_schools | p_admin_email text | SETOF affiliate.schools | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| list_upcoming_cohorts |  | TABLE(pay_slug text, program_name text, format text, categories text[], sticker_price integer, early_bird_price integer, price_currency text, rate_percent numeric, apply_url text, partner_slug text, partner_website text, cohort_key text, cohort_label text, program_code text, start_date date, end_date date, application_deadline date, seats_total integer) | sql | DEFINER | volatile | search_path=affiliate, public, pg_temp |  |
| my_school_overview | p_auth_user_id uuid | jsonb | plpgsql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| partner_earnings | p_auth_user_id uuid | jsonb | sql | DEFINER | stable | search_path=affiliate, public, pg_temp |  |
| partner_sales | p_partner_id uuid | TABLE(kind text, label text, program_name text, amount numeric, currency text, rate_percent numeric, commission numeric, occurred_at timestamp with time zone) | sql | DEFINER | stable | search_path=affiliate, public, portal, pg_temp |  |
| update_my_payout_settings | p_auth_user_id uuid, p_method text, p_last4 text, p_currency text | affiliate.partners | plpgsql | DEFINER | volatile | search_path=affiliate, pg_temp |  |

### affiliate: bodies

#### affiliate.admin_add_admin(p_admin_email text, p_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_add_admin(p_admin_email text, p_email text)
 RETURNS affiliate.admins
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.admins;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  insert into affiliate.admins(email) values (lower(trim(p_email))) on conflict (email) do nothing;
  select * into r from affiliate.admins where email = lower(trim(p_email));
  return r;
end $function$

```

#### affiliate.admin_applications(p_admin_email text, p_platform text, p_search text, p_attributed_only boolean, p_limit integer, p_offset integer)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_applications(p_admin_email text, p_platform text, p_search text, p_attributed_only boolean, p_limit integer, p_offset integer)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare res jsonb;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  with a as (
    select x.*, coalesce(pp.full_name, lp.full_name) partner_name,
           case when pc.id is not null then 'promo_code' when ll.id is not null then 'payment_link' when s.id is not null then 'school' end attributed_via,
           s.school_name partner_school
      from affiliate.all_applications x
      left join affiliate.promo_code_links pc on x.scholarship_code is not null and upper(pc.code) = upper(trim(x.scholarship_code))
      left join affiliate.partners pp on pp.id = pc.partner_id
      left join affiliate.payment_link_links ll on ll.central_payment_link_id = x.payment_link_id
      left join affiliate.partners lp on lp.id = ll.partner_id
      left join affiliate.schools s on x.scholarship_code is not null and s.school_code is not null and upper(s.school_code) = upper(trim(x.scholarship_code))
     where (coalesce(p_platform,'') = '' or x.platform = p_platform)
       and (coalesce(p_search,'') = '' or x.student_name ilike '%'||p_search||'%' or x.student_email ilike '%'||p_search||'%'
            or x.student_id ilike '%'||p_search||'%' or x.scholarship_code ilike '%'||p_search||'%' or x.utm_source ilike '%'||p_search||'%'
            or x.program_name ilike '%'||p_search||'%')
  ), f as (select * from a where not coalesce(p_attributed_only,false) or attributed_via is not null)
  select jsonb_build_object(
    'total', (select count(*) from f),
    'rows', coalesce((select jsonb_agg(to_jsonb(r)) from (select * from f order by created_at desc nulls last
                       limit greatest(1, least(coalesce(p_limit,50), 200))
                       offset greatest(0, coalesce(p_offset,0))) r), '[]'::jsonb)
  ) into res;
  return res;
end $function$

```

#### affiliate.admin_create_announcement(p_admin_email text, p_kind text, p_title text, p_link_url text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_create_announcement(p_admin_email text, p_kind text, p_title text, p_link_url text)
 RETURNS affiliate.announcements
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.announcements;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if p_kind not in ('brochure', 'form', 'update') then raise exception 'invalid_kind'; end if;
  if coalesce(trim(p_title), '') = '' or coalesce(trim(p_link_url), '') = '' then raise exception 'invalid_input'; end if;
  insert into affiliate.announcements (kind, title, link_url, created_by_email)
  values (p_kind, trim(p_title), trim(p_link_url), lower(trim(p_admin_email)))
  returning * into r;
  return r;
end $function$

```

#### affiliate.admin_create_payout(p_admin_email text, p_partner_id uuid, p_amount numeric, p_currency text, p_period_label text, p_reference text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_create_payout(p_admin_email text, p_partner_id uuid, p_amount numeric, p_currency text, p_period_label text, p_reference text)
 RETURNS affiliate.payouts
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.payouts;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if p_amount is null or p_amount <= 0 then raise exception 'invalid_amount'; end if;
  insert into affiliate.payouts(partner_id, amount, currency, status, period_label, reference)
  values (p_partner_id, p_amount, upper(p_currency), 'processing', nullif(trim(p_period_label),''), nullif(trim(p_reference),''))
  returning * into r;
  return r;
end $function$

```

#### affiliate.admin_create_resource(p_admin_email text, p_title text, p_type text, p_url text, p_description text, p_featured boolean)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_create_resource(p_admin_email text, p_title text, p_type text, p_url text, p_description text, p_featured boolean)
 RETURNS affiliate.resources
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.resources;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if p_type not in ('Image', 'Video', 'Sheet', 'PDF', 'Doc') then raise exception 'invalid_type'; end if;
  if coalesce(trim(p_title), '') = '' or coalesce(trim(p_url), '') = '' then raise exception 'invalid_input'; end if;
  insert into affiliate.resources (title, type, url, description, featured, created_by_email)
  values (trim(p_title), p_type, trim(p_url), nullif(trim(p_description), ''), coalesce(p_featured, false), lower(trim(p_admin_email)))
  returning * into r;
  return r;
end $function$

```

#### affiliate.admin_create_school_achievement(p_admin_email text, p_school_id uuid, p_kind text, p_title text, p_description text, p_student_name text, p_program_name text, p_link_url text, p_cover_image_url text, p_featured boolean, p_published_at date)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_create_school_achievement(p_admin_email text, p_school_id uuid, p_kind text, p_title text, p_description text, p_student_name text, p_program_name text, p_link_url text, p_cover_image_url text, p_featured boolean, p_published_at date)
 RETURNS affiliate.school_achievements
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.school_achievements;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if p_kind not in ('paper', 'certificate', 'testimonial', 'project') then raise exception 'invalid_kind'; end if;
  if coalesce(trim(p_title), '') = '' then raise exception 'invalid_input'; end if;
  insert into affiliate.school_achievements
    (school_id, kind, title, description, student_name, program_name, link_url, cover_image_url, featured, published_at, created_by_email)
  values (
    p_school_id, p_kind, trim(p_title), nullif(trim(p_description), ''), nullif(trim(p_student_name), ''),
    nullif(trim(p_program_name), ''), nullif(trim(p_link_url), ''), nullif(trim(p_cover_image_url), ''),
    coalesce(p_featured, false), coalesce(p_published_at, current_date), lower(trim(p_admin_email))
  )
  returning * into r;
  return r;
end $function$

```

#### affiliate.admin_delete_announcement(p_admin_email text, p_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_delete_announcement(p_admin_email text, p_id uuid)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  delete from affiliate.announcements where id = p_id;
  return found;
end $function$

```

#### affiliate.admin_delete_school_achievement(p_admin_email text, p_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_delete_school_achievement(p_admin_email text, p_id uuid)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  delete from affiliate.school_achievements where id = p_id;
  return found;
end $function$

```

#### affiliate.admin_list_admins(p_admin_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_list_admins(p_admin_email text)
 RETURNS SETOF affiliate.admins
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  return query select * from affiliate.admins order by created_at;
end $function$

```

#### affiliate.admin_list_announcements(p_admin_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_list_announcements(p_admin_email text)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  return coalesce((select jsonb_agg(to_jsonb(a) order by a.created_at desc) from affiliate.announcements a), '[]'::jsonb);
end $function$

```

#### affiliate.admin_list_partners(p_admin_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_list_partners(p_admin_email text)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  return coalesce((select jsonb_agg(to_jsonb(p) || jsonb_build_object(
      'codes', (select count(*) from affiliate.promo_code_links c where c.partner_id = p.id),
      'links', (select count(*) from affiliate.payment_link_links l where l.partner_id = p.id))
    order by p.created_at desc) from affiliate.partners p), '[]'::jsonb);
end $function$

```

#### affiliate.admin_list_payouts(p_admin_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_list_payouts(p_admin_email text)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  return coalesce((select jsonb_agg(to_jsonb(y) || jsonb_build_object('partner_name', p.full_name, 'partner_email', p.email) order by y.created_at desc)
    from affiliate.payouts y join affiliate.partners p on p.id = y.partner_id), '[]'::jsonb);
end $function$

```

#### affiliate.admin_list_resources(p_admin_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_list_resources(p_admin_email text)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  return coalesce((select jsonb_agg(to_jsonb(r) order by r.featured desc, r.created_at desc)
    from affiliate.resources r), '[]'::jsonb);
end $function$

```

#### affiliate.admin_list_school_achievements(p_admin_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_list_school_achievements(p_admin_email text)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  return coalesce((select jsonb_agg(to_jsonb(a) || jsonb_build_object('school_name', s.school_name) order by a.featured desc, a.published_at desc, a.created_at desc)
    from affiliate.school_achievements a join affiliate.schools s on s.id = a.school_id), '[]'::jsonb);
end $function$

```

#### affiliate.admin_overview(p_admin_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_overview(p_admin_email text)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  return jsonb_build_object(
    'partners', (select coalesce(jsonb_object_agg(status, n), '{}'::jsonb) from (select status, count(*) n from affiliate.partners group by 1) t),
    'schools', (select coalesce(jsonb_object_agg(status, n), '{}'::jsonb) from (select status, count(*) n from affiliate.schools group by 1) t),
    'payouts', (select coalesce(jsonb_agg(t), '[]'::jsonb) from (select status, currency, sum(amount) total, count(*) n from affiliate.payouts group by 1, 2) t),
    'applications', (select coalesce(jsonb_agg(t order by t.n desc), '[]'::jsonb) from (
        select platform, count(*) n, count(*) filter (where lower(payment_status) in ('paid', 'succeeded', 'completed')) paid
          from affiliate.all_applications group by 1) t),
    'codes', (select count(*) from affiliate.promo_code_links),
    'links', (select count(*) from affiliate.payment_link_links),
    'achievements', (select count(*) from affiliate.school_achievements),
    'resources', (select count(*) from affiliate.resources)
  );
end $function$

```

#### affiliate.admin_remove_admin(p_admin_email text, p_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_remove_admin(p_admin_email text, p_email text)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if lower(trim(p_email)) = lower(trim(p_admin_email)) then raise exception 'cannot_remove_self'; end if;
  delete from affiliate.admins where email = lower(trim(p_email));
  return found;
end $function$

```

#### affiliate.admin_set_school_status(p_admin_email text, p_school_id uuid, p_status text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_set_school_status(p_admin_email text, p_school_id uuid, p_status text)
 RETURNS affiliate.schools
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.schools;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if p_status not in ('invited','active','suspended') then raise exception 'invalid_status'; end if;
  update affiliate.schools set status = p_status, updated_at = now() where id = p_school_id returning * into r;
  if not found then raise exception 'not_found'; end if;
  return r;
end $function$

```

#### affiliate.admin_update_partner(p_admin_email text, p_partner_id uuid, p_status text, p_rate_camps numeric, p_rate_inperson numeric, p_rate_online numeric, p_discount_cap numeric)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_update_partner(p_admin_email text, p_partner_id uuid, p_status text, p_rate_camps numeric, p_rate_inperson numeric, p_rate_online numeric, p_discount_cap numeric)
 RETURNS affiliate.partners
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.partners;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if p_status not in ('pending','approved','rejected','suspended') then raise exception 'invalid_status'; end if;
  if least(p_rate_camps, p_rate_inperson, p_rate_online, p_discount_cap) < 0
     or greatest(p_rate_camps, p_rate_inperson, p_rate_online, p_discount_cap) > 100 then raise exception 'invalid_rate'; end if;
  update affiliate.partners set status = p_status, rate_camps = p_rate_camps, rate_inperson = p_rate_inperson,
         rate_online = p_rate_online, discount_cap_percent = p_discount_cap, updated_at = now()
   where id = p_partner_id returning * into r;
  if not found then raise exception 'not_found'; end if;
  return r;
end $function$

```

#### affiliate.admin_update_payout(p_admin_email text, p_payout_id uuid, p_status text, p_reference text)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_update_payout(p_admin_email text, p_payout_id uuid, p_status text, p_reference text)
 RETURNS affiliate.payouts
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.payouts;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if p_status not in ('processing','paid','cancelled') then raise exception 'invalid_status'; end if;
  update affiliate.payouts set status = p_status, reference = coalesce(nullif(trim(p_reference),''), reference),
         paid_at = case when p_status = 'paid' then coalesce(paid_at, now()) else null end
   where id = p_payout_id returning * into r;
  if not found then raise exception 'not_found'; end if;
  return r;
end $function$

```

#### affiliate.admin_update_resource(p_admin_email text, p_id uuid, p_title text, p_type text, p_url text, p_description text, p_featured boolean)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_update_resource(p_admin_email text, p_id uuid, p_title text, p_type text, p_url text, p_description text, p_featured boolean)
 RETURNS affiliate.resources
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.resources;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if p_type not in ('Image', 'Video', 'Sheet', 'PDF', 'Doc') then raise exception 'invalid_type'; end if;
  if coalesce(trim(p_title), '') = '' or coalesce(trim(p_url), '') = '' then raise exception 'invalid_input'; end if;
  update affiliate.resources set
    title = trim(p_title), type = p_type, url = trim(p_url),
    description = nullif(trim(p_description), ''), featured = coalesce(p_featured, false), updated_at = now()
   where id = p_id
  returning * into r;
  if not found then raise exception 'not_found'; end if;
  return r;
end $function$

```

#### affiliate.admin_update_school_achievement(p_admin_email text, p_id uuid, p_kind text, p_title text, p_description text, p_student_name text, p_program_name text, p_link_url text, p_cover_image_url text, p_featured boolean, p_published_at date)

```sql
CREATE OR REPLACE FUNCTION affiliate.admin_update_school_achievement(p_admin_email text, p_id uuid, p_kind text, p_title text, p_description text, p_student_name text, p_program_name text, p_link_url text, p_cover_image_url text, p_featured boolean, p_published_at date)
 RETURNS affiliate.school_achievements
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.school_achievements;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if p_kind not in ('paper', 'certificate', 'testimonial', 'project') then raise exception 'invalid_kind'; end if;
  if coalesce(trim(p_title), '') = '' then raise exception 'invalid_input'; end if;
  update affiliate.school_achievements set
    kind = p_kind, title = trim(p_title), description = nullif(trim(p_description), ''),
    student_name = nullif(trim(p_student_name), ''), program_name = nullif(trim(p_program_name), ''),
    link_url = nullif(trim(p_link_url), ''), cover_image_url = nullif(trim(p_cover_image_url), ''),
    featured = coalesce(p_featured, false), published_at = coalesce(p_published_at, published_at), updated_at = now()
   where id = p_id
  returning * into r;
  if not found then raise exception 'not_found'; end if;
  return r;
end $function$

```

#### affiliate.apply_partner(p_auth_user_id uuid, p_full_name text, p_email text, p_channel_handle text, p_channel_type text, p_audience_size text)

```sql
CREATE OR REPLACE FUNCTION affiliate.apply_partner(p_auth_user_id uuid, p_full_name text, p_email text, p_channel_handle text, p_channel_type text, p_audience_size text)
 RETURNS affiliate.partners
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'pg_temp'
AS $function$
declare
  v_row affiliate.partners;
begin
  insert into affiliate.partners (auth_user_id, full_name, email, channel_handle, channel_type, audience_size)
  values (p_auth_user_id, p_full_name, lower(p_email), p_channel_handle, p_channel_type, p_audience_size)
  on conflict (auth_user_id) do update set
    full_name = excluded.full_name,
    channel_handle = excluded.channel_handle,
    channel_type = excluded.channel_type,
    audience_size = excluded.audience_size,
    updated_at = now()
  returning * into v_row;
  return v_row;
end;
$function$

```

#### affiliate.auth_user_id_by_email(p_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.auth_user_id_by_email(p_email text)
 RETURNS uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'auth', 'pg_temp'
AS $function$
  select id from auth.users where lower(email) = lower(trim(p_email)) limit 1;
$function$

```

#### affiliate.commission_rate_for_program(p_format text, p_categories text[])

```sql
CREATE OR REPLACE FUNCTION affiliate.commission_rate_for_program(p_format text, p_categories text[])
 RETURNS numeric
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'affiliate', 'public', 'extensions', 'pg_temp'
AS $function$
  select case
    when 'Camps' = any(p_categories) then 1
    when 'Competitions' = any(p_categories) or 'Fellowships' = any(p_categories) then 2
    when p_format = 'Online' then 2
    when p_format = 'Hybrid' then 1.5
    when 'Conferences' = any(p_categories) then 1.5
    when p_format = 'In Person' then 1.5
    else 1.5
  end;
$function$

```

#### affiliate.create_payment_link(p_auth_user_id uuid, p_program_pay_slug text, p_offer_price numeric, p_slug text, p_max_uses integer, p_expires_at timestamp with time zone)

```sql
CREATE OR REPLACE FUNCTION affiliate.create_payment_link(p_auth_user_id uuid, p_program_pay_slug text, p_offer_price numeric, p_slug text, p_max_uses integer DEFAULT NULL::integer, p_expires_at timestamp with time zone DEFAULT NULL::timestamp with time zone)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'portal', 'pg_temp'
AS $function$
declare
  v_partner affiliate.partners;
  v_program public.all_programs;
  v_live_price numeric;
  v_floor numeric;
  v_rate numeric;
  v_note text;
  v_new_id uuid;
begin
-- ... truncated: showing first 15 of 76 lines
```

#### affiliate.create_promo_code(p_auth_user_id uuid, p_program_pay_slug text, p_discount_type text, p_discount_value numeric, p_code text, p_max_redemptions integer, p_expires_at timestamp with time zone)

```sql
CREATE OR REPLACE FUNCTION affiliate.create_promo_code(p_auth_user_id uuid, p_program_pay_slug text, p_discount_type text, p_discount_value numeric, p_code text, p_max_redemptions integer DEFAULT NULL::integer, p_expires_at timestamp with time zone DEFAULT NULL::timestamp with time zone)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare
  v_partner affiliate.partners;
  v_program public.all_programs;
  v_live_price numeric;
  v_discount_amount numeric;
  v_effective_pct numeric;
  v_rate numeric;
  v_note text;
  v_new_id uuid;
-- ... truncated: showing first 15 of 77 lines
```

#### affiliate.get_partner_by_auth_id(p_auth_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.get_partner_by_auth_id(p_auth_user_id uuid)
 RETURNS affiliate.partners
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'pg_temp'
AS $function$
  select * from affiliate.partners where auth_user_id = p_auth_user_id;
$function$

```

#### affiliate.get_school_for_user(p_auth_user_id uuid, p_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.get_school_for_user(p_auth_user_id uuid, p_email text)
 RETURNS affiliate.schools
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.schools;
begin
  select * into r from affiliate.schools where auth_user_id = p_auth_user_id;
  if found then return r; end if;
  update affiliate.schools
     set auth_user_id = p_auth_user_id, status = case when status = 'invited' then 'active' else status end, updated_at = now()
   where email = lower(trim(p_email)) and auth_user_id is null and status <> 'suspended'
   returning * into r;
  return r;
end $function$

```

#### affiliate.invite_school(p_admin_email text, p_school_name text, p_contact_name text, p_email text, p_school_code text)

```sql
CREATE OR REPLACE FUNCTION affiliate.invite_school(p_admin_email text, p_school_name text, p_contact_name text, p_email text, p_school_code text)
 RETURNS affiliate.schools
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare r affiliate.schools;
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  if coalesce(trim(p_school_name),'') = '' or coalesce(trim(p_contact_name),'') = '' or position('@' in coalesce(p_email,'')) = 0 then
    raise exception 'invalid_input';
  end if;
  insert into affiliate.schools(school_name, contact_name, email, school_code, invited_by_email)
  values (trim(p_school_name), trim(p_contact_name), lower(trim(p_email)), nullif(upper(trim(p_school_code)),''), lower(trim(p_admin_email)))
  returning * into r;
  return r;
end $function$

```

#### affiliate.is_admin(p_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.is_admin(p_email text)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
  select exists(select 1 from affiliate.admins where email = lower(trim(p_email)));
$function$

```

#### affiliate.leaderboard()

```sql
CREATE OR REPLACE FUNCTION affiliate.leaderboard()
 RETURNS TABLE(display_name text, channel_type text, conversions bigint, revenue numeric, currency text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
  select split_part(pa.full_name,' ',1) || coalesce(' ' || left(nullif(split_part(pa.full_name,' ',2),''),1) || '.', ''),
         pa.channel_type, count(*), sum(x.amount), x.currency
    from affiliate.partners pa, affiliate.partner_sales(pa.id) x
   where pa.status = 'approved'
   group by pa.id, pa.full_name, pa.channel_type, x.currency
   order by sum(x.amount) desc
   limit 20;
$function$

```

#### affiliate.list_active_programs()

```sql
CREATE OR REPLACE FUNCTION affiliate.list_active_programs()
 RETURNS TABLE(pay_slug text, program_name text, format text, categories text[], sticker_price integer, early_bird_price integer, price_currency text, rate_percent numeric)
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
  select p.pay_slug, p.program_name, p.format, p.categories,
         p.sticker_price, p.early_bird_price, p.price_currency::text,
         affiliate.commission_rate_for_program(p.format, p.categories)
  from public.all_programs p
  where p.is_active = true and p.pay_slug is not null
  order by p.program_name;
$function$

```

#### affiliate.list_announcements(p_auth_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.list_announcements(p_auth_user_id uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not exists (select 1 from affiliate.schools where auth_user_id = p_auth_user_id and status = 'active') then
    return '[]'::jsonb;
  end if;
  return coalesce((select jsonb_agg(to_jsonb(a) order by a.created_at desc) from affiliate.announcements a), '[]'::jsonb);
end $function$

```

#### affiliate.list_my_applications(p_auth_user_id uuid, p_search text, p_limit integer, p_offset integer)

```sql
CREATE OR REPLACE FUNCTION affiliate.list_my_applications(p_auth_user_id uuid, p_search text DEFAULT NULL::text, p_limit integer DEFAULT 100, p_offset integer DEFAULT 0)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare v_school_code text; res jsonb;
begin
  select school_code into v_school_code from affiliate.schools where auth_user_id = p_auth_user_id and status = 'active';
  if v_school_code is null then return jsonb_build_object('total', 0, 'rows', '[]'::jsonb); end if;

  with a as (
    select x.* from affiliate.all_applications x
    where x.scholarship_code is not null and upper(trim(x.scholarship_code)) = upper(v_school_code)
      and (coalesce(p_search,'') = '' or x.student_name ilike '%'||p_search||'%' or x.student_email ilike '%'||p_search||'%'
           or x.program_name ilike '%'||p_search||'%')
  )
  select jsonb_build_object(
    'total', (select count(*) from a),
    'rows', coalesce((select jsonb_agg(to_jsonb(r)) from (select * from a order by created_at desc nulls last
                       limit greatest(1, least(coalesce(p_limit,100), 500))
                       offset greatest(0, coalesce(p_offset,0))) r), '[]'::jsonb)
  ) into res;
  return res;
end $function$

```

#### affiliate.list_my_payment_links(p_auth_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.list_my_payment_links(p_auth_user_id uuid)
 RETURNS TABLE(id uuid, slug text, program_name text, program_pay_slug text, list_price numeric, offer_price numeric, currency text, rate_percent numeric, max_uses integer, uses_count integer, is_active boolean, expires_at timestamp with time zone, created_at timestamp with time zone, partner_website text, apply_url text)
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'portal', 'pg_temp'
AS $function$
  select l.central_payment_link_id, l.slug, l.program_name, l.program_pay_slug,
         l.list_price, l.offer_price, l.currency, l.rate_percent,
         pl.max_uses, pl.uses_count, pl.is_active, pl.expires_at, l.created_at,
         programme_partner.website, programme.apply_url
  from affiliate.payment_link_links l
  join affiliate.partners pa on pa.id = l.partner_id
  join portal.payment_links pl on pl.id = l.central_payment_link_id
  left join public.all_partners programme_partner on programme_partner.id = pl.partner_id
  left join public.all_programs programme on programme.id = pl.program_id
  where pa.auth_user_id = p_auth_user_id
  order by l.created_at desc;
$function$

```

#### affiliate.list_my_promo_codes(p_auth_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.list_my_promo_codes(p_auth_user_id uuid)
 RETURNS TABLE(id uuid, code text, program_name text, program_pay_slug text, discount_type text, discount_value numeric, currency text, max_redemptions integer, redemption_count integer, is_active boolean, expires_at timestamp with time zone, created_at timestamp with time zone, rate_percent numeric, list_price numeric)
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
  select pc.id, pc.code, pc.program_name, l.program_pay_slug, pc.discount_type,
         pc.discount_value, pc.currency, pc.max_redemptions, pc.redemption_count,
         pc.is_active, pc.expires_at, pc.created_at, l.rate_percent, l.list_price
  from affiliate.promo_code_links l
  join affiliate.partners pa on pa.id = l.partner_id
  join public.promo_codes pc on pc.id = l.central_promo_code_id
  where pa.auth_user_id = p_auth_user_id
  order by pc.created_at desc;
$function$

```

#### affiliate.list_my_school_programs(p_auth_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.list_my_school_programs(p_auth_user_id uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare v_school_code text;
begin
  select school_code into v_school_code from affiliate.schools where auth_user_id = p_auth_user_id and status = 'active';
  if v_school_code is null then return '[]'::jsonb; end if;

  return coalesce((
    select jsonb_agg(to_jsonb(t) order by t.students desc)
    from (
      select program_code, program_name, platform, count(*) students,
             count(*) filter (where lower(payment_status) in ('paid','succeeded','completed')) paid_students,
             sum(amount) filter (where lower(payment_status) in ('paid','succeeded','completed')) total_paid,
             max(currency) currency
        from affiliate.all_applications
       where scholarship_code is not null and upper(trim(scholarship_code)) = upper(v_school_code)
       group by program_code, program_name, platform
    ) t
  ), '[]'::jsonb);
end $function$

```

#### affiliate.list_resources()

```sql
CREATE OR REPLACE FUNCTION affiliate.list_resources()
 RETURNS TABLE(id uuid, title text, type text, url text, description text, featured boolean, created_at timestamp with time zone)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
  select r.id, r.title, r.type, r.url, r.description, r.featured, r.created_at
  from affiliate.resources r
  order by r.featured desc, r.created_at desc;
$function$

```

#### affiliate.list_school_achievements(p_auth_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.list_school_achievements(p_auth_user_id uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare v_school_id uuid;
begin
  select id into v_school_id from affiliate.schools where auth_user_id = p_auth_user_id;
  if v_school_id is null then return '[]'::jsonb; end if;
  return coalesce((select jsonb_agg(to_jsonb(a) order by a.featured desc, a.published_at desc, a.created_at desc)
    from affiliate.school_achievements a where a.school_id = v_school_id), '[]'::jsonb);
end $function$

```

#### affiliate.list_schools(p_admin_email text)

```sql
CREATE OR REPLACE FUNCTION affiliate.list_schools(p_admin_email text)
 RETURNS SETOF affiliate.schools
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
begin
  if not affiliate.is_admin(p_admin_email) then raise exception 'not_admin'; end if;
  return query select * from affiliate.schools order by created_at desc;
end $function$

```

#### affiliate.list_upcoming_cohorts()

```sql
CREATE OR REPLACE FUNCTION affiliate.list_upcoming_cohorts()
 RETURNS TABLE(pay_slug text, program_name text, format text, categories text[], sticker_price integer, early_bird_price integer, price_currency text, rate_percent numeric, apply_url text, partner_slug text, partner_website text, cohort_key text, cohort_label text, program_code text, start_date date, end_date date, application_deadline date, seats_total integer)
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
  select p.pay_slug, p.program_name, p.format, p.categories,
         p.sticker_price, p.early_bird_price, p.price_currency::text,
         affiliate.commission_rate_for_program(p.format, p.categories),
         p.apply_url, pa.partner_slug, pa.website,
         c.cohort_key, c.cohort_label, c.program_code,
         c.start_date, c.end_date, c.application_deadline, c.seats_total
  from public.all_cohorts c
  join public.all_programs p on p.id = c.program_id
  join public.all_partners pa on pa.id = p.partner_id
  where p.is_active = true
    and pa.is_active = true
    and p.apply_url is not null
    and (c.start_date is null or c.start_date >= current_date)
  order by coalesce(c.start_date, current_date), p.program_name;
$function$

```

#### affiliate.my_school_overview(p_auth_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.my_school_overview(p_auth_user_id uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
declare v_school_code text;
begin
  select school_code into v_school_code from affiliate.schools where auth_user_id = p_auth_user_id and status = 'active';
  if v_school_code is null then return jsonb_build_object('students', 0, 'paid_students', 0, 'programs', 0, 'totals', '[]'::jsonb); end if;

  return jsonb_build_object(
    'students', (select count(*) from affiliate.all_applications where scholarship_code is not null and upper(trim(scholarship_code)) = upper(v_school_code)),
    'paid_students', (select count(*) from affiliate.all_applications where scholarship_code is not null and upper(trim(scholarship_code)) = upper(v_school_code)
                        and lower(payment_status) in ('paid','succeeded','completed')),
    'programs', (select count(distinct program_code) from affiliate.all_applications where scholarship_code is not null and upper(trim(scholarship_code)) = upper(v_school_code)),
    'totals', (select coalesce(jsonb_agg(t), '[]'::jsonb) from (
        select currency, sum(amount) total from affiliate.all_applications
         where scholarship_code is not null and upper(trim(scholarship_code)) = upper(v_school_code)
           and lower(payment_status) in ('paid','succeeded','completed')
         group by currency) t)
  );
end $function$

```

#### affiliate.partner_earnings(p_auth_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.partner_earnings(p_auth_user_id uuid)
 RETURNS jsonb
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'pg_temp'
AS $function$
  with p as (select id from affiliate.partners where auth_user_id = p_auth_user_id),
  s as (select x.* from p, affiliate.partner_sales(p.id) x)
  select jsonb_build_object(
    'by_currency', coalesce((select jsonb_agg(t order by t.revenue desc) from (
        select currency, sum(amount) revenue, sum(commission) commission
          from s group by currency) t), '[]'::jsonb),
    'conversions', (select count(*) from s where kind='promo_code')
                   + coalesce((select sum(pl.uses_count) from affiliate.payment_link_links l join portal.payment_links pl on pl.id=l.central_payment_link_id, p where l.partner_id=p.id),0),
    'monthly', coalesce((select jsonb_agg(m order by m.month) from (
        select to_char(date_trunc('month', occurred_at),'YYYY-MM') as month, currency, sum(amount) revenue
          from s where occurred_at > now() - interval '8 months' group by 1,2) m), '[]'::jsonb),
    'top', coalesce((select jsonb_agg(t) from (
        select kind, label, program_name, currency, sum(amount) revenue, sum(commission) commission
          from s group by 1,2,3,4 order by sum(amount) desc limit 5) t), '[]'::jsonb),
    'payouts', coalesce((select jsonb_agg(po order by po.created_at desc) from (
        select o.id, o.amount, o.currency, o.status, o.period_label, o.reference, o.paid_at, o.created_at
          from affiliate.payouts o, p where o.partner_id = p.id) po), '[]'::jsonb)
  );
$function$

```

#### affiliate.partner_sales(p_partner_id uuid)

```sql
CREATE OR REPLACE FUNCTION affiliate.partner_sales(p_partner_id uuid)
 RETURNS TABLE(kind text, label text, program_name text, amount numeric, currency text, rate_percent numeric, commission numeric, occurred_at timestamp with time zone)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'affiliate', 'public', 'portal', 'pg_temp'
AS $function$
  select 'promo_code', l.code, l.program_name, r.amount_after, upper(r.currency), l.rate_percent,
         round(r.amount_after * l.rate_percent / 100, 2), r.created_at
    from affiliate.promo_code_links l
    join public.promo_redemptions r on r.promo_code_id = l.central_promo_code_id
   where l.partner_id = p_partner_id
  union all
  select 'payment_link', l.slug, l.program_name, l.offer_price * pl.uses_count, upper(l.currency), l.rate_percent,
         round(l.offer_price * pl.uses_count * l.rate_percent / 100, 2), coalesce(pl.updated_at, l.created_at)
    from affiliate.payment_link_links l
    join portal.payment_links pl on pl.id = l.central_payment_link_id
   where l.partner_id = p_partner_id and pl.uses_count > 0;
$function$

```

#### affiliate.update_my_payout_settings(p_auth_user_id uuid, p_method text, p_last4 text, p_currency text)

```sql
CREATE OR REPLACE FUNCTION affiliate.update_my_payout_settings(p_auth_user_id uuid, p_method text, p_last4 text, p_currency text)
 RETURNS affiliate.partners
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'affiliate', 'pg_temp'
AS $function$
declare r affiliate.partners;
begin
  if p_method not in ('Bank transfer', 'PayPal', 'Wise') then raise exception 'invalid_method'; end if;
  if p_currency !~ '^[A-Za-z]{3}$' then raise exception 'invalid_currency'; end if;
  if p_last4 is not null and p_last4 !~ '^[A-Za-z0-9]{0,4}$' then raise exception 'invalid_last4'; end if;

  update affiliate.partners
     set payout_method = p_method,
         payout_account_last4 = nullif(trim(p_last4), ''),
         payout_currency = upper(p_currency),
         updated_at = now()
   where auth_user_id = p_auth_user_id
  returning * into r;

  if not found then raise exception 'not_found'; end if;
  return r;
end $function$

```

## premed (47 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| attribution_value | v text | text | sql | invoker | immutable | search_path=pg_catalog | anon authenticated |
| audience_for_category | _category text | text | sql | invoker | immutable | search_path=premed, public | anon authenticated |
| auth_user_id_by_email | p_email text | uuid | sql | DEFINER | volatile | search_path=premed, auth, pg_temp |  |
| auth_verified_email |  | text | sql | DEFINER | stable | search_path=premed, public, auth | authenticated |
| captured_leads_set_defaults |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| claim_promo_slot | _code text, _max integer, _holder text | integer | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| concierge_record_ai_call | p_prompt_tokens bigint, p_completion_tokens bigint | void | sql | DEFINER | volatile | search_path=premed, public |  |
| consume_request_quota | p_key text, p_limit integer, p_minutes integer | boolean | plpgsql | DEFINER | volatile | search_path=premed, pg_temp |  |
| dedupe_masterclass_registration |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| enforce_application_insert_defaults |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| enforce_masterclass_registration_category |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| ensure_user_profile |  | premed.app_role | plpgsql | DEFINER | volatile | search_path=premed, public, auth | authenticated |
| fill_default_utm |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| get_fellowship_project_mentor_notes | _id uuid | text | plpgsql | DEFINER | stable | search_path=premed, public, pg_temp | authenticated |
| get_fellowship_projects_safe |  | TABLE(id uuid, title text, description text, track text, student_name text, cohort_name text, project_url text, is_active boolean, is_featured boolean, year integer, created_at timestamp with time zone, updated_at timestamp with time zone) | sql | DEFINER | stable | search_path=premed, public | authenticated |
| get_my_attendance |  | TABLE(id uuid, session_date date, attended boolean, participation_score integer, created_at timestamp with time zone) | sql | DEFINER | stable | search_path=premed, public | authenticated |
| get_session_secrets | _session_id uuid | TABLE(zoom_url text, passcode text) | plpgsql | DEFINER | stable | search_path=premed, public | authenticated |
| get_session_zoom_url | _session_id uuid | text | plpgsql | DEFINER | stable | search_path=premed, public, pg_temp | authenticated |
| get_user_role | _user_id uuid | premed.app_role | sql | DEFINER | stable | search_path=premed, public |  |
| handle_updated_at |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| has_role | _user_id uuid, _role premed.app_role | boolean | sql | DEFINER | stable | search_path=premed, public | authenticated |
| invoke_edge_function | p_name text, p_body jsonb | bigint | plpgsql | DEFINER | volatile | search_path=premed, public, extensions, vault |  |
| job_triggers_dispatch |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public, extensions |  |
| knowledge_entries_search_vector_update |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| match_knowledge | query_text text, filter_region text, filter_pathway text, filter_domain text, match_count integer | TABLE(id uuid, domain text, subdomain text, region text, school text, pathway text, claim text, claim_type text, confidence text, source_name text, verified_date date, expires date, display_text text, counselor_note text, contradictions text[], rank real) | sql | DEFINER | stable | search_path=premed, public | authenticated |
| match_public_knowledge | query_text text, match_count integer | TABLE(id uuid, domain text, subdomain text, claim text, claim_type text, confidence text, source_name text, source_url text, verified_date date, expires date, display_text text, rank real) | plpgsql | DEFINER | stable | search_path=premed, public | anon authenticated |
| mentor_cohort_ids | _mentor_id uuid | SETOF uuid | sql | DEFINER | stable | search_path=premed, public |  |
| normalize_application_category | _value text | text | sql | invoker | immutable | search_path=pg_catalog | anon authenticated |
| normalize_application_cohort |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| normalize_phone | _raw text, _country text | text | plpgsql | invoker | immutable | search_path=premed, public | anon authenticated |
| notify_goal_completed |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| notify_goal_review_published |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| notify_rsvp_created |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| partner_id |  | uuid | sql | DEFINER | stable | search_path=public | anon authenticated |
| phone_dial_code | _country text | text | plpgsql | invoker | immutable | search_path=premed, public | anon authenticated |
| phone_national_length | _cc text | integer | sql | invoker | immutable | search_path=premed, public | anon authenticated |
| prevent_role_escalation |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| read_programme_applications |  | TABLE(id uuid, student_name text, student_email text, parent_email text, school text, grade text, cohort_id uuid, cohort text, program_code text, program_name text, payment_status text, registration_status text, app_status text, created_at timestamp with time zone, applied_date timestamp with time zone, metadata jsonb) | sql | DEFINER | stable | search_path=pg_catalog, premed, public | authenticated |
| resolve_application_category | _code text, _form uuid | text | plpgsql | DEFINER | stable | search_path=pg_catalog, premed |  |
| series_for_category | _category text | text | sql | invoker | immutable | search_path=premed, public | anon authenticated |
| set_cron_key | p_key text | text | plpgsql | DEFINER | volatile | search_path=premed, public, extensions, vault |  |
| set_updated_at |  | trigger | plpgsql | DEFINER | volatile | search_path=premed |  |
| tg_masterclass_registrations_write |  | trigger | plpgsql | DEFINER | volatile | search_path=premed, public |  |
| tg_masterclass_sessions_write |  | trigger | plpgsql | invoker | volatile | search_path=premed, public | anon authenticated |
| tg_normalize_application_phones |  | trigger | plpgsql | invoker | volatile | search_path=premed, public | anon authenticated |
| tg_normalize_masterclass_phones |  | trigger | plpgsql | invoker | volatile | search_path=premed, public | anon authenticated |
| tg_normalize_profile_phone |  | trigger | plpgsql | invoker | volatile | search_path=premed, public | anon authenticated |

### premed: bodies

#### premed.attribution_value(v text)

```sql
CREATE OR REPLACE FUNCTION premed.attribution_value(v text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'pg_catalog'
AS $function$ SELECT coalesce(nullif(left(btrim(split_part(split_part(split_part(v,',',1),';',1),'|',1)),200),''),'unknown') $function$

```

#### premed.audience_for_category(_category text)

```sql
CREATE OR REPLACE FUNCTION premed.audience_for_category(_category text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'premed', 'public'
AS $function$
  select case _category
           when 'junior_doctors' then 'junior'
           when 'future_doctors' then 'senior'
           else 'general'
         end
$function$

```

#### premed.auth_user_id_by_email(p_email text)

```sql
CREATE OR REPLACE FUNCTION premed.auth_user_id_by_email(p_email text)
 RETURNS uuid
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'premed', 'auth', 'pg_temp'
AS $function$
  select id from auth.users where lower(email) = lower(p_email) limit 1;
$function$

```

#### premed.auth_verified_email()

```sql
CREATE OR REPLACE FUNCTION premed.auth_verified_email()
 RETURNS text
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public', 'auth'
AS $function$
  SELECT lower(u.email) FROM auth.users u
  WHERE u.id = auth.uid() AND u.email_confirmed_at IS NOT NULL LIMIT 1
$function$

```

#### premed.captured_leads_set_defaults()

```sql
CREATE OR REPLACE FUNCTION premed.captured_leads_set_defaults()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
BEGIN
  IF NEW.unsubscribe_token IS NULL THEN
    NEW.unsubscribe_token := replace(gen_random_uuid()::text,'-','') || replace(gen_random_uuid()::text,'-','');
  END IF;
  IF NEW.captured_at IS NULL THEN NEW.captured_at := now(); END IF;
  RETURN NEW;
END $function$

```

#### premed.claim_promo_slot(_code text, _max integer, _holder text)

```sql
CREATE OR REPLACE FUNCTION premed.claim_promo_slot(_code text, _max integer, _holder text)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
declare
  used int;
  c text := upper(trim(_code));
  h text := lower(trim(_holder));
begin
  if c = '' or h = '' then return 0; end if;
  perform pg_advisory_xact_lock(hashtext('premed.promo:' || c));
  insert into premed.promo_slot_claims (code, holder) values (c, h)
    on conflict (code, holder) do update set created_at = now();
  select count(*) into used
    from premed.promo_slot_claims
   where code = c and succeeded = false and created_at > now() - interval '30 minutes';
  if _max is not null and used > _max then
    delete from premed.promo_slot_claims where code = c and holder = h and succeeded = false;
    return -1;
  end if;
  return used;
end;
$function$

```

#### premed.concierge_record_ai_call(p_prompt_tokens bigint, p_completion_tokens bigint)

```sql
CREATE OR REPLACE FUNCTION premed.concierge_record_ai_call(p_prompt_tokens bigint DEFAULT 0, p_completion_tokens bigint DEFAULT 0)
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
  INSERT INTO premed.concierge_ai_usage (usage_day, calls, prompt_tokens, completion_tokens)
  VALUES (current_date, 1, GREATEST(p_prompt_tokens, 0), GREATEST(p_completion_tokens, 0))
  ON CONFLICT (usage_day) DO UPDATE
    SET calls = premed.concierge_ai_usage.calls + 1,
        prompt_tokens = premed.concierge_ai_usage.prompt_tokens + GREATEST(p_prompt_tokens, 0),
        completion_tokens = premed.concierge_ai_usage.completion_tokens + GREATEST(p_completion_tokens, 0),
        updated_at = now();
$function$

```

#### premed.consume_request_quota(p_key text, p_limit integer, p_minutes integer)

```sql
CREATE OR REPLACE FUNCTION premed.consume_request_quota(p_key text, p_limit integer, p_minutes integer)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'pg_temp'
AS $function$ DECLARE used bigint; BEGIN IF p_key IS NULL OR length(p_key)>200 OR p_limit NOT BETWEEN 1 AND 10000 OR p_minutes NOT BETWEEN 1 AND 1440 THEN RETURN false; END IF; PERFORM pg_advisory_xact_lock(hashtextextended('fd-quota:'||p_key,0)); SELECT coalesce(sum(request_count),0) INTO used FROM premed.concierge_rate_limits WHERE client_key=p_key AND window_start > now()-make_interval(mins=>p_minutes); IF used>=p_limit THEN RETURN false; END IF; INSERT INTO premed.concierge_rate_limits(client_key,window_start,request_count) VALUES(p_key,date_trunc('minute',now()),1) ON CONFLICT (client_key,window_start) DO UPDATE SET request_count=concierge_rate_limits.request_count+1; RETURN true; END $function$

```

#### premed.dedupe_masterclass_registration()

```sql
CREATE OR REPLACE FUNCTION premed.dedupe_masterclass_registration()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
DECLARE recent_id uuid;
BEGIN
  IF NEW.email IS NULL OR length(btrim(NEW.email)) = 0 THEN RETURN NEW; END IF;
  SELECT id INTO recent_id FROM public.premed_masterclass_regs
   WHERE lower(email) = lower(NEW.email) AND created_at > now() - interval '60 seconds'
   ORDER BY created_at DESC LIMIT 1;
  IF recent_id IS NOT NULL THEN
    RAISE EXCEPTION 'duplicate masterclass registration within 60 seconds' USING ERRCODE = 'unique_violation';
  END IF;
  RETURN NEW;
END $function$

```

#### premed.enforce_application_insert_defaults()

```sql
CREATE OR REPLACE FUNCTION premed.enforce_application_insert_defaults()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
declare jwt_role text;
begin
  begin
    jwt_role := current_setting('request.jwt.claims', true)::jsonb->>'role';
  exception when others then jwt_role := null;
  end;

  if jwt_role = 'service_role' or jwt_role is null then
    return new;
  end if;

  if premed.has_role(auth.uid(),'mentor') or premed.has_role(auth.uid(),'admin') then
    return new;
  end if;

  new.payment_status := 'pending';
  new.app_status     := 'submitted';
  new.payment_id     := null;
  new.payment_date   := null;
  new.amount         := null;
  new.currency       := null;
  return new;
end $function$

```

#### premed.enforce_masterclass_registration_category()

```sql
CREATE OR REPLACE FUNCTION premed.enforce_masterclass_registration_category()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
declare ids uuid[]; cats text[]; raw jsonb;
begin
  raw := coalesce(new.meta,'{}'::jsonb) -> 'session_ids';
  if jsonb_typeof(raw) = 'array' then
    select array_agg(elem::uuid) into ids
      from jsonb_array_elements_text(raw) as elem
     where elem ~* '^[0-9a-f-]{36}$';
  end if;
  if ids is null or array_length(ids,1) = 0 then
    ids := nullif(new.masterclass_ids, '{}');
  end if;

  if ids is not null and array_length(ids,1) > 0 then
    select array_agg(distinct s.category) into cats
      from premed.masterclass_sessions s
     where s.id = any(ids);
  elsif new.sessions is not null and array_length(new.sessions,1) > 0 then
    select array_agg(distinct s.category) into cats
      from premed.masterclass_sessions s
     where exists (select 1 from unnest(new.sessions) l where l ilike s.title || '%');
  end if;

  if cats is not null and array_length(cats,1) > 1 then
    raise exception 'Masterclass registration cannot mix categories: %', cats;
  end if;
  if cats is not null and array_length(cats,1) = 1 then
    new.category := cats[1];
  end if;
  if new.category is null or new.category = '' then
    new.category := 'future_doctors';
  end if;
  return new;
end $function$

```

#### premed.ensure_user_profile()

```sql
CREATE OR REPLACE FUNCTION premed.ensure_user_profile()
 RETURNS premed.app_role
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public', 'auth'
AS $function$
DECLARE _uid uuid := auth.uid(); _role premed.app_role;
BEGIN
  IF _uid IS NULL THEN RAISE EXCEPTION 'Not authenticated'; END IF;
  INSERT INTO premed.user_profiles (user_id, role, email, full_name)
  SELECT u.id, 'student', u.email, COALESCE(u.raw_user_meta_data->>'full_name','')
  FROM auth.users u WHERE u.id = _uid
  ON CONFLICT (user_id) DO NOTHING;
  SELECT role INTO _role FROM premed.user_profiles WHERE user_id = _uid;
  RETURN _role;
END $function$

```

#### premed.fill_default_utm()

```sql
CREATE OR REPLACE FUNCTION premed.fill_default_utm()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
DECLARE old_real boolean; new_real boolean;
BEGIN
 NEW.utm_source := coalesce(nullif(btrim(NEW.utm_source),''),'unknown');
 NEW.utm_medium := coalesce(nullif(btrim(NEW.utm_medium),''),'unknown');
 NEW.utm_campaign := coalesce(nullif(btrim(NEW.utm_campaign),''),'unknown');
 IF TG_OP='UPDATE' THEN
  old_real := (lower(OLD.utm_source) NOT IN ('unknown','direct') OR lower(OLD.utm_medium) NOT IN ('unknown','website') OR lower(OLD.utm_campaign) NOT IN ('unknown','direct'));
  new_real := (lower(NEW.utm_source) <> 'unknown' OR lower(NEW.utm_medium) <> 'unknown' OR lower(NEW.utm_campaign) <> 'unknown');
  IF old_real AND NOT new_real THEN
   NEW.utm_source := OLD.utm_source; NEW.utm_medium := OLD.utm_medium; NEW.utm_campaign := OLD.utm_campaign;
  END IF;
 END IF;
 RETURN NEW;
END $function$

```

#### premed.get_fellowship_project_mentor_notes(_id uuid)

```sql
CREATE OR REPLACE FUNCTION premed.get_fellowship_project_mentor_notes(_id uuid)
 RETURNS text
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public', 'pg_temp'
AS $function$
DECLARE
  _notes text;
BEGIN
  IF NOT premed.has_role((SELECT auth.uid()), 'admin'::premed.app_role) THEN
    RAISE EXCEPTION 'Not authorized' USING ERRCODE = '42501';
  END IF;

  SELECT mentor_notes
    INTO _notes
    FROM premed.fellowship_projects
    WHERE id = _id;

  RETURN _notes;
END;
$function$

```

#### premed.get_fellowship_projects_safe()

```sql
CREATE OR REPLACE FUNCTION premed.get_fellowship_projects_safe()
 RETURNS TABLE(id uuid, title text, description text, track text, student_name text, cohort_name text, project_url text, is_active boolean, is_featured boolean, year integer, created_at timestamp with time zone, updated_at timestamp with time zone)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
  SELECT id, title, description, track, student_name, cohort_name, project_url, is_active, is_featured, year, created_at, updated_at
  FROM premed.fellowship_projects
  WHERE premed.has_role(auth.uid(),'admin') OR premed.has_role(auth.uid(),'mentor');
$function$

```

#### premed.get_my_attendance()

```sql
CREATE OR REPLACE FUNCTION premed.get_my_attendance()
 RETURNS TABLE(id uuid, session_date date, attended boolean, participation_score integer, created_at timestamp with time zone)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
  SELECT id, session_date, attended, participation_score, created_at
  FROM premed.session_attendance WHERE user_id = auth.uid() ORDER BY session_date DESC;
$function$

```

#### premed.get_session_secrets(_session_id uuid)

```sql
CREATE OR REPLACE FUNCTION premed.get_session_secrets(_session_id uuid)
 RETURNS TABLE(zoom_url text, passcode text)
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
BEGIN
  IF premed.has_role(auth.uid(),'admin') THEN
    RETURN QUERY SELECT m.join_url, m.passcode FROM public.all_masterclasses m WHERE m.id = _session_id;
    RETURN;
  END IF;
  IF NOT premed.has_role(auth.uid(),'mentor') THEN
    RAISE EXCEPTION 'Not authorized';
  END IF;
  RETURN QUERY SELECT m.join_url, m.passcode FROM public.all_masterclasses m
   WHERE m.id = _session_id
     AND (m.host_mentor_id = auth.uid()
          OR m.cohort_id IN (SELECT premed.mentor_cohort_ids(auth.uid())));
  IF NOT FOUND THEN RAISE EXCEPTION 'Not authorized'; END IF;
END $function$

```

#### premed.get_session_zoom_url(_session_id uuid)

```sql
CREATE OR REPLACE FUNCTION premed.get_session_zoom_url(_session_id uuid)
 RETURNS text
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public', 'pg_temp'
AS $function$
DECLARE _uid uuid:=auth.uid(); _email text; _url text;
BEGIN
 IF _uid IS NULL THEN RETURN NULL; END IF;
 IF premed.has_role(_uid,'admin') THEN
  SELECT join_url INTO _url FROM public.all_masterclasses WHERE id=_session_id AND partner_id=premed.partner_id();
  RETURN _url;
 END IF;
 SELECT u.email INTO _email FROM auth.users u WHERE u.id=_uid AND u.email_confirmed_at IS NOT NULL;
 IF _email IS NULL THEN RETURN NULL; END IF;
 SELECT m.join_url INTO _url FROM public.all_masterclasses m
 JOIN public.all_cohorts c ON c.id=m.cohort_id
 JOIN public.all_programs p ON p.id=c.program_id AND p.partner_id=premed.partner_id()
 JOIN public.applications a ON a.program_code=c.program_code
 WHERE m.id=_session_id AND m.partner_id=premed.partner_id()
 AND a.registration_status='complete' AND a.form_data#>>'{internal,app_status}'='accepted'
 AND a.payment_status='paid'
 AND (lower(a.student_email)=lower(_email) OR lower(a.parent_email)=lower(_email))
 AND NOT EXISTS (SELECT 1 FROM public.all_cohorts other WHERE other.program_code=c.program_code AND other.id<>c.id)
 LIMIT 1;
 RETURN _url;
END $function$

```

#### premed.get_user_role(_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION premed.get_user_role(_user_id uuid)
 RETURNS premed.app_role
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
  SELECT role FROM premed.user_profiles WHERE user_id = _user_id LIMIT 1
$function$

```

#### premed.handle_updated_at()

```sql
CREATE OR REPLACE FUNCTION premed.handle_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END $function$

```

#### premed.has_role(_user_id uuid, _role premed.app_role)

```sql
CREATE OR REPLACE FUNCTION premed.has_role(_user_id uuid, _role premed.app_role)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
  SELECT EXISTS (
    SELECT 1 FROM premed.user_profiles
    WHERE user_id = _user_id AND role = _role
  )
  AND (
    auth.uid() IS NULL
    OR _user_id = auth.uid()
    OR EXISTS (SELECT 1 FROM premed.user_profiles WHERE user_id = auth.uid() AND role = 'admin')
  )
$function$

```

#### premed.invoke_edge_function(p_name text, p_body jsonb)

```sql
CREATE OR REPLACE FUNCTION premed.invoke_edge_function(p_name text, p_body jsonb DEFAULT '{}'::jsonb)
 RETURNS bigint
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public', 'extensions', 'vault'
AS $function$
declare
  v_key text;
  v_req bigint;
begin
  if p_name !~ '^[a-z0-9-]+$' then
    raise exception 'invalid function name';
  end if;
  select decrypted_secret into v_key
  from vault.decrypted_secrets
  where name = 'DOCTORS_SERVICE_ROLE_KEY'
  limit 1;
  if v_key is null then
    raise exception 'service key not configured';
  end if;
  select net.http_post(
    url := 'https://<ESTATE_REF>.supabase.co/functions/v1/' || p_name,
    headers := jsonb_build_object('Content-Type','application/json','Authorization','Bearer ' || v_key),
    body := p_body,
    timeout_milliseconds := 150000
  ) into v_req;
  return v_req;
end;
$function$

```

#### premed.job_triggers_dispatch()

```sql
CREATE OR REPLACE FUNCTION premed.job_triggers_dispatch()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public', 'extensions'
AS $function$
begin
  new.request_id := premed.invoke_edge_function(new.function_name, new.payload);
  return new;
end;
$function$

```

#### premed.knowledge_entries_search_vector_update()

```sql
CREATE OR REPLACE FUNCTION premed.knowledge_entries_search_vector_update()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
BEGIN
  NEW.search_vector :=
    setweight(to_tsvector('english', COALESCE(NEW.school, '')), 'A') ||
    setweight(to_tsvector('english', COALESCE(NEW.claim, '')), 'B') ||
    setweight(to_tsvector('english', COALESCE(NEW.display_text, '')), 'C') ||
    setweight(to_tsvector('english', COALESCE(NEW.subdomain, '') || ' ' || COALESCE(NEW.domain, '') || ' ' ||
              COALESCE(NEW.region, '') || ' ' || COALESCE(NEW.pathway, '')), 'D');
  NEW.updated_at := now();
  RETURN NEW;
END $function$

```

#### premed.match_knowledge(query_text text, filter_region text, filter_pathway text, filter_domain text, match_count integer)

```sql
CREATE OR REPLACE FUNCTION premed.match_knowledge(query_text text, filter_region text DEFAULT NULL::text, filter_pathway text DEFAULT NULL::text, filter_domain text DEFAULT NULL::text, match_count integer DEFAULT 5)
 RETURNS TABLE(id uuid, domain text, subdomain text, region text, school text, pathway text, claim text, claim_type text, confidence text, source_name text, verified_date date, expires date, display_text text, counselor_note text, contradictions text[], rank real)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
  SELECT ke.id, ke.domain, ke.subdomain, ke.region, ke.school, ke.pathway, ke.claim, ke.claim_type,
         ke.confidence, ke.source_name, ke.verified_date, ke.expires, ke.display_text,
         ke.counselor_note, ke.contradictions,
         ts_rank(ke.search_vector, websearch_to_tsquery('english', query_text)) AS rank
  FROM premed.knowledge_entries ke
  WHERE ke.active AND ke.expires > CURRENT_DATE
    AND ke.search_vector @@ websearch_to_tsquery('english', query_text)
    AND (filter_region IS NULL OR ke.region = filter_region OR ke.region = 'Global')
    AND (filter_pathway IS NULL OR ke.pathway = filter_pathway)
    AND (filter_domain IS NULL OR ke.domain = filter_domain)
  ORDER BY CASE ke.confidence WHEN 'VERIFIED' THEN 0 WHEN 'HIGH' THEN 1 WHEN 'MEDIUM' THEN 2
                              WHEN 'LOW' THEN 3 ELSE 4 END, rank DESC
  LIMIT match_count;
$function$

```

#### premed.match_public_knowledge(query_text text, match_count integer)

```sql
CREATE OR REPLACE FUNCTION premed.match_public_knowledge(query_text text, match_count integer DEFAULT 5)
 RETURNS TABLE(id uuid, domain text, subdomain text, claim text, claim_type text, confidence text, source_name text, source_url text, verified_date date, expires date, display_text text, rank real)
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
DECLARE
  q tsquery;
BEGIN
  SELECT nullif(string_agg(lexeme, ' | '), '')::tsquery
  INTO q
  FROM (
    SELECT DISTINCT t.lexeme
    FROM unnest(to_tsvector('english', coalesce(query_text, ''))) AS t
  ) w;

  IF q IS NULL THEN
    RETURN;
  END IF;

  RETURN QUERY
  SELECT ke.id, ke.domain, ke.subdomain, ke.claim, ke.claim_type, ke.confidence,
         ke.source_name, ke.source_url, ke.verified_date, ke.expires, ke.display_text,
         ts_rank(ke.search_vector, q) AS rank
  FROM premed.knowledge_entries ke
  WHERE ke.active
    AND ke.audience = 'public'
    AND ke.expires > CURRENT_DATE
    AND ke.search_vector @@ q
  ORDER BY rank DESC,
           CASE ke.confidence WHEN 'VERIFIED' THEN 0 WHEN 'HIGH' THEN 1 WHEN 'MEDIUM' THEN 2
                              WHEN 'LOW' THEN 3 ELSE 4 END
  LIMIT match_count;
END;
$function$

```

#### premed.mentor_cohort_ids(_mentor_id uuid)

```sql
CREATE OR REPLACE FUNCTION premed.mentor_cohort_ids(_mentor_id uuid)
 RETURNS SETOF uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
  SELECT cohort_id FROM premed.cohort_settings WHERE mentor_id = _mentor_id;
$function$

```

#### premed.normalize_application_category(_value text)

```sql
CREATE OR REPLACE FUNCTION premed.normalize_application_category(_value text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'pg_catalog'
AS $function$
SELECT CASE lower(btrim(coalesce(_value,'')))
 WHEN 'camp' THEN 'camp' WHEN 'camps' THEN 'camp'
 WHEN 'conference' THEN 'conference' WHEN 'conferences' THEN 'conference'
 WHEN 'certificate' THEN 'certificate' WHEN 'certificate-program' THEN 'certificate' WHEN 'certificate programs' THEN 'certificate'
 WHEN 'competition' THEN 'competition' WHEN 'competitions' THEN 'competition'
 WHEN 'fellowship' THEN 'fellowship' WHEN 'fellowships' THEN 'fellowship'
 WHEN 'in-person' THEN 'in-person' WHEN 'in person' THEN 'in-person'
 ELSE NULL END;
$function$

```

#### premed.normalize_application_cohort()

```sql
CREATE OR REPLACE FUNCTION premed.normalize_application_cohort()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
DECLARE resolved_key text; key text;
BEGIN
  key := COALESCE(NULLIF(BTRIM(NEW.cohort), ''), '');
  IF key = '' THEN
    RAISE EXCEPTION 'Application is missing a programme/cohort key';
  END IF;

  -- Competition lead-capture labels and the historic cohort key resolve to the
  -- current catalogue cohort (program_code CO-FDOC-PMCO-211126).
  IF key IN ('Medical Competition 2026 — Interest', 'Pre-Med Competition 2026 — Interest', 'premed_competition_2026') THEN
    NEW.cohort := 'premed_competition_nov26';
    RETURN NEW;
  END IF;

  IF EXISTS (SELECT 1 FROM public.all_cohorts WHERE cohort_key = key)
     OR EXISTS (SELECT 1 FROM premed.cohort_settings WHERE cohort_key = key) THEN
    NEW.cohort := key;
    RETURN NEW;
  END IF;

  SELECT c.cohort_key INTO resolved_key
  FROM premed.program_cohorts c
  WHERE c.program_key = key AND c.is_active AND NOT COALESCE(c.is_cancelled, false)
  ORDER BY COALESCE(c.start_date, CURRENT_DATE) ASC, c.sort_order ASC
  LIMIT 1;

  IF resolved_key IS NULL THEN
    RAISE EXCEPTION 'Unknown programme/cohort key: %', key;
  END IF;
  NEW.cohort := resolved_key;
  RETURN NEW;
END
$function$

```

#### premed.normalize_phone(_raw text, _country text)

```sql
CREATE OR REPLACE FUNCTION premed.normalize_phone(_raw text, _country text DEFAULT NULL::text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'premed', 'public'
AS $function$
DECLARE s text; d text; cc text; nl int; rest text; had_plus boolean;
BEGIN
  s := btrim(coalesce(_raw, ''));
  IF s = '' THEN RETURN NULL; END IF;
  s := regexp_replace(s, '^[''"\s]+', '');
  had_plus := position('+' in s) > 0;
  d := regexp_replace(s, '[^0-9]', '', 'g');
  IF d ~ '^00[1-9]' THEN d := regexp_replace(d, '^00', ''); had_plus := true; END IF;
  IF d = '' OR length(d) < 6 THEN RETURN NULL; END IF;
  cc := premed.phone_dial_code(_country);
  IF had_plus THEN
    IF cc <> '' AND left(d, length(cc)) = cc THEN
      rest := substr(d, length(cc) + 1);
      IF rest ~ '^0' THEN d := cc || regexp_replace(rest, '^0+', ''); END IF;
    END IF;
    RETURN d;
  END IF;
  IF cc = '' THEN RETURN d; END IF;
  nl := premed.phone_national_length(cc);
  IF nl IS NOT NULL AND length(d) = nl THEN RETURN cc || d; END IF;
  IF left(d, length(cc)) = cc AND length(d) >= length(cc) + 7 THEN RETURN d; END IF;
  rest := coalesce(nullif(regexp_replace(d, '^0+', ''), ''), d);
  RETURN cc || rest;
END $function$

```

#### premed.notify_goal_completed()

```sql
CREATE OR REPLACE FUNCTION premed.notify_goal_completed()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
BEGIN
  IF (NEW.progress >= 100 OR NEW.status = 'complete')
     AND (OLD.progress < 100 AND COALESCE(OLD.status,'') <> 'complete') THEN
    INSERT INTO premed.student_notifications (user_id, category, tone, title, body, link_url)
    VALUES (NEW.user_id, 'milestone', 'gold', 'Goal completed: ' || NEW.title,
            'Nicely done. Your mentor will see this in your next review.', '/student/goals');
  END IF;
  RETURN NEW;
END $function$

```

#### premed.notify_goal_review_published()

```sql
CREATE OR REPLACE FUNCTION premed.notify_goal_review_published()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
BEGIN
  IF NEW.is_published THEN
    INSERT INTO premed.student_notifications (user_id, category, tone, title, body, link_url)
    VALUES (NEW.user_id, 'mentor', 'gold', 'New mentor review',
            'Your mentor posted your monthly review.', '/student/goals');
  END IF;
  RETURN NEW;
END $function$

```

#### premed.notify_rsvp_created()

```sql
CREATE OR REPLACE FUNCTION premed.notify_rsvp_created()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
DECLARE s_title text;
BEGIN
  SELECT title INTO s_title FROM public.all_masterclasses WHERE id = NEW.session_id;
  INSERT INTO premed.student_notifications (user_id, category, tone, title, body, link_url)
  VALUES (NEW.user_id, 'live_class', 'teal', 'RSVP confirmed: ' || COALESCE(s_title,'Live class'),
          'We will remind you before it starts.', '/student/live-classes');
  RETURN NEW;
END $function$

```

#### premed.partner_id()

```sql
CREATE OR REPLACE FUNCTION premed.partner_id()
 RETURNS uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'public'
AS $function$ select id from public.all_partners where partner_slug = 'future-premed' $function$

```

#### premed.phone_dial_code(_country text)

```sql
CREATE OR REPLACE FUNCTION premed.phone_dial_code(_country text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'premed', 'public'
AS $function$
DECLARE k text; kc text; r text;
BEGIN
  k := btrim(lower(coalesce(_country, '')));
  IF k = '' THEN RETURN ''; END IF;
  kc := regexp_replace(k, '[^a-z0-9]', '', 'g');
  IF k LIKE '%узбек%' THEN RETURN '998'; END IF;
  SELECT dial INTO r FROM (VALUES
    ('afghanistan','93'),('albania','355'),('algeria','213'),('andorra','376'),('angola','244'),
    ('argentina','54'),('armenia','374'),('australia','61'),('austria','43'),('azerbaijan','994'),
-- ... truncated: showing first 15 of 64 lines
```

#### premed.phone_national_length(_cc text)

```sql
CREATE OR REPLACE FUNCTION premed.phone_national_length(_cc text)
 RETURNS integer
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'premed', 'public'
AS $function$
  SELECT n FROM (VALUES
    ('1',10),('91',10),('92',10),('44',10),('971',9),('966',9),('974',8),('968',8),
    ('973',8),('965',8),('63',10),('62',10),('65',8),('852',8),('61',9),('86',11),
    ('20',10),('234',10),('254',9),('27',9),('82',10),('81',10),('49',10),('33',9),
    ('39',10),('34',9),('90',10),('880',10),('94',9),('977',10),('998',9),('60',9),
    ('66',9),('84',9),('55',11),('52',10),('57',10),('56',9),('51',9),('58',10)
  ) AS m(cc, n) WHERE m.cc = _cc LIMIT 1;
$function$

```

#### premed.prevent_role_escalation()

```sql
CREATE OR REPLACE FUNCTION premed.prevent_role_escalation()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
DECLARE jwt_role text;
BEGIN
  BEGIN jwt_role := current_setting('request.jwt.claims', true)::jsonb->>'role';
  EXCEPTION WHEN OTHERS THEN jwt_role := NULL; END;

  IF OLD.role IS DISTINCT FROM NEW.role THEN
    IF jwt_role = 'service_role' OR current_user IN ('service_role','postgres','supabase_admin') THEN
      NULL;
    ELSIF NOT premed.has_role(auth.uid(), 'admin') THEN
      RAISE EXCEPTION 'Only admins can change user roles';
    END IF;
  END IF;
  IF OLD.user_id IS DISTINCT FROM NEW.user_id THEN
    RAISE EXCEPTION 'Cannot change user_id';
  END IF;
  RETURN NEW;
END $function$

```

#### premed.read_programme_applications()

```sql
CREATE OR REPLACE FUNCTION premed.read_programme_applications()
 RETURNS TABLE(id uuid, student_name text, student_email text, parent_email text, school text, grade text, cohort_id uuid, cohort text, program_code text, program_name text, payment_status text, registration_status text, app_status text, created_at timestamp with time zone, applied_date timestamp with time zone, metadata jsonb)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'pg_catalog', 'premed', 'public'
AS $function$
SELECT a.id,a.student_name,a.student_email,a.parent_email,a.school_name,a.grade,
 c.id,coalesce(cs.cohort_key,c.cohort_key),a.program_code,a.program_name,
 a.payment_status,a.registration_status,coalesce(a.form_data#>>'{internal,app_status}','submitted'),
 a.created_at,a.created_at,jsonb_build_object('program_code',a.program_code,'registration_status',a.registration_status)
FROM public.applications a
JOIN public.all_cohorts c ON c.program_code=a.program_code
JOIN public.all_programs p ON p.id=c.program_id AND p.partner_id=premed.partner_id()
LEFT JOIN premed.cohort_settings cs ON cs.cohort_id=c.id
WHERE auth.uid() IS NOT NULL
AND NOT EXISTS (SELECT 1 FROM public.all_cohorts other WHERE other.program_code=c.program_code AND other.id<>c.id)
AND (premed.has_role(auth.uid(),'admin')
 OR (premed.has_role(auth.uid(),'mentor') AND cs.mentor_id=auth.uid())
 OR (a.registration_status<>'withdrawn' AND EXISTS (
 SELECT 1 FROM auth.users u WHERE u.id=auth.uid() AND u.email_confirmed_at IS NOT NULL
 AND (lower(u.email)=lower(a.student_email) OR lower(u.email)=lower(a.parent_email)))))
$function$

```

#### premed.resolve_application_category(_code text, _form uuid)

```sql
CREATE OR REPLACE FUNCTION premed.resolve_application_category(_code text, _form uuid DEFAULT NULL::uuid)
 RETURNS text
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'pg_catalog', 'premed'
AS $function$
DECLARE categories text[]; main_label text; result text;
BEGIN
 SELECT array_agg(DISTINCT premed.normalize_application_category(t.settings->>'category')) FILTER (WHERE premed.normalize_application_category(t.settings->>'category') IS NOT NULL)
 INTO categories FROM portal.form_templates t
 WHERE (t.program_code=_code OR coalesce(t.settings->'program_codes','[]'::jsonb) ? _code)
 AND (_form IS NULL OR t.id=_form);
 IF cardinality(categories)=1 THEN RETURN categories[1]; END IF;
 IF cardinality(categories)>1 THEN RETURN NULL; END IF;
 -- Owner-approved fallback: catalogue MAIN category, not descriptive tags.
 SELECT p.categories[1] INTO main_label FROM public.all_cohorts c JOIN public.all_programs p ON p.id=c.program_id WHERE c.program_code=_code;
 result := premed.normalize_application_category(main_label);
 IF result IS NOT NULL THEN RETURN result; END IF;
 RETURN CASE split_part(_code,'-',1)
 WHEN 'CA' THEN 'camp' WHEN 'UC' THEN 'camp' WHEN 'CF' THEN 'conference'
 WHEN 'ZL' THEN 'certificate' WHEN 'CO' THEN 'certificate' WHEN 'OC' THEN 'certificate' WHEN 'OM' THEN 'certificate'
 WHEN 'OP' THEN 'competition' WHEN 'FL' THEN 'fellowship' WHEN 'IP' THEN 'in-person' WHEN 'SC' THEN 'in-person' ELSE NULL END;
END $function$

```

#### premed.series_for_category(_category text)

```sql
CREATE OR REPLACE FUNCTION premed.series_for_category(_category text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'premed', 'public'
AS $function$
  select case _category
           when 'junior_doctors' then 'Junior Pre-Med Masterclass Series'
           else 'Pre-Med Platform Masterclass Series'
         end
$function$

```

#### premed.set_cron_key(p_key text)

```sql
CREATE OR REPLACE FUNCTION premed.set_cron_key(p_key text)
 RETURNS text
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public', 'extensions', 'vault'
AS $function$
declare
  v_id uuid;
begin
  if p_key is null or length(p_key) < 20 then
    raise exception 'invalid key';
  end if;
  select id into v_id from vault.secrets where name = 'DOCTORS_SERVICE_ROLE_KEY' limit 1;
  if v_id is null then
    perform vault.create_secret(p_key, 'DOCTORS_SERVICE_ROLE_KEY', 'Service role key used by premed-* cron jobs');
  else
    perform vault.update_secret(v_id, p_key, 'DOCTORS_SERVICE_ROLE_KEY', 'Service role key used by premed-* cron jobs');
  end if;
  return 'ok';
end;
$function$

```

#### premed.set_updated_at()

```sql
CREATE OR REPLACE FUNCTION premed.set_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed'
AS $function$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$function$

```

#### premed.tg_masterclass_registrations_write()

```sql
CREATE OR REPLACE FUNCTION premed.tg_masterclass_registrations_write()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'premed', 'public'
AS $function$
declare jwt_role text; is_staff boolean;
begin
  begin
    jwt_role := current_setting('request.jwt.claims', true)::jsonb->>'role';
  exception when others then jwt_role := null;
  end;
  is_staff := jwt_role = 'service_role' or jwt_role is null
              or premed.has_role(auth.uid(),'mentor') or premed.has_role(auth.uid(),'admin');

-- ... truncated: showing first 15 of 90 lines
```

#### premed.tg_masterclass_sessions_write()

```sql
CREATE OR REPLACE FUNCTION premed.tg_masterclass_sessions_write()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'premed', 'public'
AS $function$
declare
  pc jsonb;
begin
  if tg_op = 'DELETE' then
    delete from public.all_masterclasses where id = old.id;
    return old;
  end if;

  pc := jsonb_strip_nulls(jsonb_build_object(
    'subtitle',        new.subtitle,
-- ... truncated: showing first 15 of 68 lines
```

#### premed.tg_normalize_application_phones()

```sql
CREATE OR REPLACE FUNCTION premed.tg_normalize_application_phones()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'premed', 'public'
AS $function$
BEGIN
  NEW.student_phone := premed.normalize_phone(NEW.student_phone, NEW.country);
  NEW.parent_phone  := premed.normalize_phone(NEW.parent_phone,  NEW.country);
  RETURN NEW;
END $function$

```

#### premed.tg_normalize_masterclass_phones()

```sql
CREATE OR REPLACE FUNCTION premed.tg_normalize_masterclass_phones()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'premed', 'public'
AS $function$
BEGIN
  NEW.phone        := premed.normalize_phone(NEW.phone, NEW.country);
  NEW.parent_phone := premed.normalize_phone(NEW.parent_phone, NEW.country);
  RETURN NEW;
END $function$

```

#### premed.tg_normalize_profile_phone()

```sql
CREATE OR REPLACE FUNCTION premed.tg_normalize_profile_phone()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'premed', 'public'
AS $function$
BEGIN NEW.phone := premed.normalize_phone(NEW.phone, NULL); RETURN NEW; END $function$

```

## finance (6 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| claim_code_sheet_sync |  | uuid | plpgsql | DEFINER | volatile | search_path=finance, public |  |
| complete_code_sheet_sync | p_claim uuid, p_error text | boolean | plpgsql | DEFINER | volatile | search_path=finance, public |  |
| is_finance_user |  | boolean | sql | DEFINER | stable | search_path=finance, public | authenticated |
| request_code_sheet_sync |  | trigger | plpgsql | DEFINER | volatile | search_path=finance, public |  |
| sync_dbs_transactions_batch | p_user_id uuid, p_rows jsonb, p_delete_cleared_ids jsonb | jsonb | plpgsql | DEFINER | volatile | search_path=finance, public |  |
| sync_wallex_transactions_batch | p_user_id uuid, p_rows jsonb | jsonb | plpgsql | DEFINER | volatile | search_path=finance, public |  |

### finance: bodies

#### finance.claim_code_sheet_sync()

```sql
CREATE OR REPLACE FUNCTION finance.claim_code_sheet_sync()
 RETURNS uuid
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finance', 'public'
AS $function$
DECLARE token uuid := gen_random_uuid(); state finance.code_sheet_sync_state;
BEGIN
  SELECT * INTO state FROM finance.code_sheet_sync_state WHERE id=true FOR UPDATE;
  IF state.claim IS NOT NULL AND state.claimed_at > now()-interval '5 minutes' THEN RETURN NULL; END IF;
  UPDATE finance.code_sheet_sync_state SET claim=token,claimed_at=now(),dirty=false WHERE id=true;
  RETURN token;
END $function$

```

#### finance.complete_code_sheet_sync(p_claim uuid, p_error text)

```sql
CREATE OR REPLACE FUNCTION finance.complete_code_sheet_sync(p_claim uuid, p_error text DEFAULT NULL::text)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finance', 'public'
AS $function$
DECLARE state finance.code_sheet_sync_state;
BEGIN
  SELECT * INTO state FROM finance.code_sheet_sync_state WHERE id=true FOR UPDATE;
  IF state.claim IS DISTINCT FROM p_claim THEN RETURN false; END IF;
  IF p_error IS NOT NULL THEN
    UPDATE finance.code_sheet_sync_state SET claim=NULL,claimed_at=NULL,dirty=true,last_error=left(p_error,2000) WHERE id=true;
    RETURN false;
  END IF;
  IF state.dirty THEN
    UPDATE finance.code_sheet_sync_state SET dirty=false,claimed_at=now(),last_synced_at=now(),last_error=NULL WHERE id=true;
    RETURN true;
  END IF;
  UPDATE finance.code_sheet_sync_state SET claim=NULL,claimed_at=NULL,last_synced_at=now(),last_error=NULL WHERE id=true;
  RETURN false;
END $function$

```

#### finance.is_finance_user()

```sql
CREATE OR REPLACE FUNCTION finance.is_finance_user()
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finance', 'public'
AS $function$
  SELECT auth.uid() IS NOT NULL AND EXISTS (
    SELECT 1
    FROM finance.user_profiles AS profile
    WHERE profile.user_id = auth.uid()
  );
$function$

```

#### finance.request_code_sheet_sync()

```sql
CREATE OR REPLACE FUNCTION finance.request_code_sheet_sync()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finance', 'public'
AS $function$
DECLARE state finance.code_sheet_sync_state;
BEGIN
  UPDATE finance.code_sheet_sync_state SET dirty=true WHERE id=true RETURNING * INTO state;
  IF state.callback_url IS NOT NULL AND (state.claim IS NULL OR state.claimed_at < now()-interval '5 minutes') THEN
    BEGIN
      PERFORM net.http_post(url:=state.callback_url,
        headers:=jsonb_build_object('Content-Type','application/json','Authorization','Bearer '||state.callback_secret),
        body:='{"action":"sync"}'::jsonb,timeout_milliseconds:=120000);
    EXCEPTION WHEN OTHERS THEN
      UPDATE finance.code_sheet_sync_state SET last_error='Unable to dispatch sheet sync' WHERE id=true;
    END;
  END IF;
  RETURN NULL;
END $function$

```

#### finance.sync_dbs_transactions_batch(p_user_id uuid, p_rows jsonb, p_delete_cleared_ids jsonb)

```sql
CREATE OR REPLACE FUNCTION finance.sync_dbs_transactions_batch(p_user_id uuid, p_rows jsonb, p_delete_cleared_ids jsonb DEFAULT '[]'::jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finance', 'public'
AS $function$
DECLARE
  v_upserted integer := 0;
  v_deleted integer := 0;
  v_dedup integer := 0;
BEGIN
  IF p_user_id IS NULL THEN RAISE EXCEPTION 'p_user_id is required'; END IF;
  IF p_rows IS NULL OR jsonb_typeof(p_rows) <> 'array' THEN RAISE EXCEPTION 'p_rows must be a JSON array'; END IF;

  PERFORM pg_advisory_xact_lock(hashtextextended('dbs-sync:' || p_user_id::text, 0));
-- ... truncated: showing first 15 of 97 lines
```

#### finance.sync_wallex_transactions_batch(p_user_id uuid, p_rows jsonb)

```sql
CREATE OR REPLACE FUNCTION finance.sync_wallex_transactions_batch(p_user_id uuid, p_rows jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finance', 'public'
AS $function$
DECLARE v_inserted_count integer := 0;
BEGIN
  IF p_user_id IS NULL THEN RAISE EXCEPTION 'p_user_id is required'; END IF;
  IF p_rows IS NULL OR jsonb_typeof(p_rows) <> 'array' THEN RAISE EXCEPTION 'p_rows must be a JSON array'; END IF;

  PERFORM pg_advisory_xact_lock(hashtextextended('wallex-sync:' || p_user_id::text, 0));

  WITH source_rows AS (SELECT row_data, ord FROM jsonb_array_elements(p_rows) WITH ORDINALITY AS t(row_data, ord)),
  normalized AS (
-- ... truncated: showing first 15 of 86 lines
```

## finedu (31 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| admin_catalogue_audit | payload jsonb | jsonb | sql | DEFINER | stable | search_path=finedu, public |  |
| admin_delete_cohort | payload jsonb | jsonb | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| admin_delete_masterclass | payload jsonb | jsonb | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| admin_upsert_cohort | payload jsonb | jsonb | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| admin_upsert_masterclass | payload jsonb | jsonb | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| admin_upsert_program | payload jsonb | jsonb | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| build_program_code | _program_id uuid, _start date | text | sql | DEFINER | stable | search_path=finedu, public |  |
| claim_paid_application | _program text, _student_email text, _parent_email text, _student_first_name text, _student_last_name text, _student_phone text, _student_dob date, _grade_level text, _school text, _city text, _state text, _country text, _parent_name text, _parent_phone text, _parent_consent boolean, _track text, _essay_motivation text, _referral_source text, _utm_source text, _utm_medium text, _utm_campaign text | uuid | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| claim_promo_slot | _code text, _max integer, _holder text | integer | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| cohort_key_exists | _key text | boolean | sql | DEFINER | stable | search_path=finedu, public |  |
| complete_application_by_token | _token text, _student_first_name text, _student_last_name text, _student_email text, _student_phone text, _student_dob date, _grade_level text, _school text, _city text, _state text, _country text, _parent_name text, _parent_email text, _parent_phone text, _parent_consent boolean, _track text, _essay_motivation text | uuid | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| default_utm_fields |  | trigger | plpgsql | invoker | volatile | search_path="" | anon authenticated |
| get_application_by_completion_token | _token text | TABLE(id uuid, cohort text, student_first_name text, student_last_name text, student_email text, student_phone text, student_dob date, grade_level text, school text, city text, state text, country text, parent_name text, parent_email text, parent_phone text, track text, payment_status text, amount_paid numeric, payment_currency text, completion_status text) | sql | DEFINER | stable | search_path=finedu, public |  |
| get_public_school_proposal | _slug text | TABLE(id uuid, slug text, school_name text, contact_name text, contact_title text, city text, seats integer, price_per_student_usd numeric, original_price_usd numeric, cohort_key text, custom_inclusions text[], valid_until date, stripe_payment_link text, status text) | sql | DEFINER | stable | search_path=finedu, public |  |
| has_role | _user_id uuid, _role finedu.app_role | boolean | sql | DEFINER | stable | search_path=finedu, public | authenticated |
| is_admin |  | boolean | sql | DEFINER | stable | search_path=finedu, public | authenticated |
| log_catalogue | _entity text, _id uuid, _action text, _actor text, _changes jsonb | void | sql | DEFINER | volatile | search_path=finedu, public |  |
| mark_promo_slot | _code text, _holder text | void | sql | DEFINER | volatile | search_path=finedu, public |  |
| next_cohort_after | _program_id uuid, _at timestamp with time zone | uuid | sql | DEFINER | stable | search_path=finedu, public |  |
| partner_id |  | uuid | sql | DEFINER | stable | search_path=public | authenticated |
| program_cohorts_del |  | trigger | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| program_cohorts_ins |  | trigger | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| program_cohorts_upd |  | trigger | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| record_addon_payment | _app_id uuid, _payment_link_id text, _payment_intent_id text, _amount numeric, _currency text | jsonb | plpgsql | DEFINER | volatile | search_path=public |  |
| redeem_voucher | _lead_id uuid, _application_id uuid, _voucher_code text, _amount_usd integer | boolean | plpgsql | DEFINER | volatile | search_path=finedu, public |  |
| tg_finlit_app_catalogue |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| tg_finlit_app_format |  | trigger | plpgsql | invoker | volatile | search_path=finedu, public | anon authenticated |
| tg_finlit_app_view_write |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| tg_finlit_shared_student_id |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=finedu, public | anon authenticated |
| upsert_pending_addon | _app_id uuid, _addons jsonb | void | sql | DEFINER | volatile | search_path=public |  |

### finedu: bodies

#### finedu.admin_catalogue_audit(payload jsonb)

```sql
CREATE OR REPLACE FUNCTION finedu.admin_catalogue_audit(payload jsonb)
 RETURNS jsonb
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
  SELECT COALESCE(jsonb_agg(to_jsonb(a) ORDER BY a.created_at DESC), '[]'::jsonb)
  FROM (
    SELECT * FROM finedu.catalogue_audit
    WHERE (payload->>'entity' IS NULL OR entity = payload->>'entity')
      AND (payload->>'entity_id' IS NULL OR entity_id = (payload->>'entity_id')::uuid)
    ORDER BY created_at DESC
    LIMIT COALESCE(NULLIF(payload->>'limit','')::int, 50)
  ) a
$function$

```

#### finedu.admin_delete_cohort(payload jsonb)

```sql
CREATE OR REPLACE FUNCTION finedu.admin_delete_cohort(payload jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE _id uuid := (payload->>'id')::uuid; _apps int; _key text;
BEGIN
  SELECT cohort_key, application_count INTO _key, _apps FROM finedu.admin_cohorts WHERE id = _id;
  IF _key IS NULL THEN RAISE EXCEPTION 'cohort not owned by Financial Literacy Platform'; END IF;
  IF _apps > 0 THEN RAISE EXCEPTION 'cohort has % application(s) — cancel it instead of deleting', _apps; END IF;
  IF EXISTS (SELECT 1 FROM public.all_masterclasses WHERE cohort_id = _id) THEN
    RAISE EXCEPTION 'cohort is linked to a masterclass — unlink it first';
  END IF;
  DELETE FROM public.all_cohorts WHERE id = _id;
  PERFORM finedu.log_catalogue('cohort', _id, 'delete', payload->>'_actor', jsonb_build_object('cohort_key', _key));
  RETURN jsonb_build_object('deleted', true);
END;
$function$

```

#### finedu.admin_delete_masterclass(payload jsonb)

```sql
CREATE OR REPLACE FUNCTION finedu.admin_delete_masterclass(payload jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE _id uuid := (payload->>'id')::uuid; _regs int; _slug text;
BEGIN
  SELECT slug, registration_count INTO _slug, _regs FROM finedu.admin_masterclasses WHERE id = _id;
  IF _slug IS NULL THEN RAISE EXCEPTION 'masterclass not owned by Financial Literacy Platform'; END IF;
  IF _regs > 0 THEN RAISE EXCEPTION 'masterclass has % registration(s) — unpublish or cancel it instead', _regs; END IF;
  DELETE FROM public.all_masterclasses WHERE id = _id;
  PERFORM finedu.log_catalogue('masterclass', _id, 'delete', payload->>'_actor', jsonb_build_object('slug', _slug));
  RETURN jsonb_build_object('deleted', true);
END;
$function$

```

#### finedu.admin_upsert_cohort(payload jsonb)

```sql
CREATE OR REPLACE FUNCTION finedu.admin_upsert_cohort(payload jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE
  _id uuid := NULLIF(payload->>'id','')::uuid;
  _actor text := payload->>'_actor';
  _program_id uuid := NULLIF(payload->>'program_id','')::uuid;
  _start date := NULLIF(payload->>'start_date','')::date;
  _end date := NULLIF(payload->>'end_date','')::date;
  _deadline date := NULLIF(payload->>'application_deadline','')::date;
  _key text := NULLIF(trim(payload->>'cohort_key'),'');
  _label text := NULLIF(trim(payload->>'cohort_label'),'');
-- ... truncated: showing first 15 of 72 lines
```

#### finedu.admin_upsert_masterclass(payload jsonb)

```sql
CREATE OR REPLACE FUNCTION finedu.admin_upsert_masterclass(payload jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE
  _id uuid := NULLIF(payload->>'id','')::uuid;
  _actor text := payload->>'_actor';
  _program_id uuid := NULLIF(payload->>'program_id','')::uuid;
  _cohort_id uuid := NULLIF(payload->>'cohort_id','')::uuid;
  _at timestamptz := NULLIF(payload->>'scheduled_at','')::timestamptz;
  _slug text := NULLIF(trim(payload->>'slug'),'');
  _mid uuid;
BEGIN
-- ... truncated: showing first 15 of 99 lines
```

#### finedu.admin_upsert_program(payload jsonb)

```sql
CREATE OR REPLACE FUNCTION finedu.admin_upsert_program(payload jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE
  _id uuid := NULLIF(payload->>'id','')::uuid;
  _actor text := payload->>'_actor';
  _key text := NULLIF(trim(payload->>'program_key'),'');
  _pid uuid;
BEGIN
  IF _id IS NOT NULL AND NOT EXISTS (
    SELECT 1 FROM public.all_programs WHERE id = _id AND partner_id = finedu.partner_id()
  ) THEN RAISE EXCEPTION 'program not owned by Financial Literacy Platform'; END IF;
-- ... truncated: showing first 15 of 90 lines
```

#### finedu.build_program_code(_program_id uuid, _start date)

```sql
CREATE OR REPLACE FUNCTION finedu.build_program_code(_program_id uuid, _start date)
 RETURNS text
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
  SELECT CASE WHEN _start IS NULL THEN NULL
              ELSE COALESCE((SELECT code_prefix FROM finedu.program_map WHERE program_id = _program_id), 'CO-FINL-XXXX')
                   || '-' || to_char(_start, 'DDMMYY') END
$function$

```

#### finedu.claim_paid_application(_program text, _student_email text, _parent_email text, _student_first_name text, _student_last_name text, _student_phone text, _student_dob date, _grade_level text, _school text, _city text, _state text, _country text, _parent_name text, _parent_phone text, _parent_consent boolean, _track text, _essay_motivation text, _referral_source text, _utm_source text, _utm_medium text, _utm_campaign text)

```sql
CREATE OR REPLACE FUNCTION finedu.claim_paid_application(_program text, _student_email text, _parent_email text, _student_first_name text, _student_last_name text, _student_phone text, _student_dob date, _grade_level text, _school text, _city text, _state text, _country text, _parent_name text, _parent_phone text, _parent_consent boolean, _track text, _essay_motivation text, _referral_source text DEFAULT NULL::text, _utm_source text DEFAULT 'direct'::text, _utm_medium text DEFAULT 'website'::text, _utm_campaign text DEFAULT 'direct'::text)
 RETURNS uuid
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE
  _id uuid;
  _prefix text;
  _se text := NULLIF(lower(trim(_student_email)),'');
  _pe text := NULLIF(lower(trim(_parent_email)),'');
  _email_rx text := '^[^@\s]+@[^@\s]+\.[a-z]{2,}$';
BEGIN
  IF _se IS NOT NULL AND _se !~ _email_rx THEN _se := NULL; END IF;
  IF _pe IS NOT NULL AND _pe !~ _email_rx THEN _pe := NULL; END IF;
-- ... truncated: showing first 15 of 81 lines
```

#### finedu.claim_promo_slot(_code text, _max integer, _holder text)

```sql
CREATE OR REPLACE FUNCTION finedu.claim_promo_slot(_code text, _max integer, _holder text)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$ declare used int; c text := upper(trim(_code)); h text := lower(trim(_holder)); begin if c = '' or h = '' then return 0; end if; perform pg_advisory_xact_lock(hashtext('finedu.promo:' || c)); insert into finedu.promo_slot_claims (code, holder) values (c, h) on conflict (code, holder) do update set created_at = now(); select count(*) into used from finedu.promo_slot_claims where code = c and succeeded = false and created_at > now() - interval '30 minutes'; if _max is not null and used > _max then delete from finedu.promo_slot_claims where code = c and holder = h and succeeded = false; return -1; end if; return used; end; $function$

```

#### finedu.cohort_key_exists(_key text)

```sql
CREATE OR REPLACE FUNCTION finedu.cohort_key_exists(_key text)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
  select exists (select 1 from finedu.program_cohorts where cohort_key = _key)
$function$

```

#### finedu.complete_application_by_token(_token text, _student_first_name text, _student_last_name text, _student_email text, _student_phone text, _student_dob date, _grade_level text, _school text, _city text, _state text, _country text, _parent_name text, _parent_email text, _parent_phone text, _parent_consent boolean, _track text, _essay_motivation text)

```sql
CREATE OR REPLACE FUNCTION finedu.complete_application_by_token(_token text, _student_first_name text, _student_last_name text, _student_email text, _student_phone text, _student_dob date, _grade_level text, _school text, _city text, _state text, _country text, _parent_name text, _parent_email text, _parent_phone text, _parent_consent boolean, _track text, _essay_motivation text)
 RETURNS uuid
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE
  _id uuid;
BEGIN
  UPDATE finedu.finlit_applications
     SET student_first_name = COALESCE(NULLIF(_student_first_name,''), student_first_name),
         student_last_name  = COALESCE(NULLIF(_student_last_name,''),  student_last_name),
         student_email      = COALESCE(NULLIF(lower(_student_email),''),student_email),
         student_phone      = COALESCE(NULLIF(_student_phone,''),      student_phone),
         student_dob        = COALESCE(_student_dob, student_dob),
-- ... truncated: showing first 15 of 41 lines
```

#### finedu.default_utm_fields()

```sql
CREATE OR REPLACE FUNCTION finedu.default_utm_fields()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO ''
AS $function$
begin
  new.utm_source   := coalesce(nullif(trim(new.utm_source),''),'direct');
  new.utm_medium   := coalesce(nullif(trim(new.utm_medium),''),'website');
  new.utm_campaign := coalesce(nullif(trim(new.utm_campaign),''),'direct');
  return new;
end $function$

```

#### finedu.get_application_by_completion_token(_token text)

```sql
CREATE OR REPLACE FUNCTION finedu.get_application_by_completion_token(_token text)
 RETURNS TABLE(id uuid, cohort text, student_first_name text, student_last_name text, student_email text, student_phone text, student_dob date, grade_level text, school text, city text, state text, country text, parent_name text, parent_email text, parent_phone text, track text, payment_status text, amount_paid numeric, payment_currency text, completion_status text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
  select a.id, a.cohort, a.student_first_name, a.student_last_name, a.student_email,
         a.student_phone, a.student_dob, a.grade_level, a.school, a.city, a.state, a.country,
         a.parent_name, a.parent_email, a.parent_phone, a.track,
         a.payment_status, a.amount_paid, a.payment_currency, a.completion_status
    from finedu.finlit_applications a
   where a.completion_token = _token and a.completion_status = 'stub_pending_details'
   limit 1;
$function$

```

#### finedu.get_public_school_proposal(_slug text)

```sql
CREATE OR REPLACE FUNCTION finedu.get_public_school_proposal(_slug text)
 RETURNS TABLE(id uuid, slug text, school_name text, contact_name text, contact_title text, city text, seats integer, price_per_student_usd numeric, original_price_usd numeric, cohort_key text, custom_inclusions text[], valid_until date, stripe_payment_link text, status text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
  select sp.id, sp.slug, sp.school_name, sp.contact_name, sp.contact_title, sp.city,
         sp.seats, sp.price_per_student_usd, sp.original_price_usd, sp.cohort_key,
         sp.custom_inclusions, sp.valid_until, sp.stripe_payment_link, sp.status
  from finedu.school_proposals sp
  where sp.slug = _slug and sp.status = 'active'
  limit 1;
$function$

```

#### finedu.has_role(_user_id uuid, _role finedu.app_role)

```sql
CREATE OR REPLACE FUNCTION finedu.has_role(_user_id uuid, _role finedu.app_role)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
  select exists (select 1 from finedu.user_roles where user_id = _user_id and role = _role)
     and (_user_id = auth.uid() or coalesce(auth.role(),'') = 'service_role')
$function$

```

#### finedu.is_admin()

```sql
CREATE OR REPLACE FUNCTION finedu.is_admin()
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
  select finedu.has_role(auth.uid(), 'admin'::finedu.app_role)
$function$

```

#### finedu.log_catalogue(_entity text, _id uuid, _action text, _actor text, _changes jsonb)

```sql
CREATE OR REPLACE FUNCTION finedu.log_catalogue(_entity text, _id uuid, _action text, _actor text, _changes jsonb)
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
  INSERT INTO finedu.catalogue_audit (entity, entity_id, action, actor, changes)
  VALUES (_entity, _id, _action, NULLIF(_actor,''), _changes)
$function$

```

#### finedu.mark_promo_slot(_code text, _holder text)

```sql
CREATE OR REPLACE FUNCTION finedu.mark_promo_slot(_code text, _holder text)
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$ update finedu.promo_slot_claims set succeeded = true where code = upper(trim(_code)) and holder = lower(trim(_holder)); $function$

```

#### finedu.next_cohort_after(_program_id uuid, _at timestamp with time zone)

```sql
CREATE OR REPLACE FUNCTION finedu.next_cohort_after(_program_id uuid, _at timestamp with time zone)
 RETURNS uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
  SELECT c.id FROM public.all_cohorts c
  WHERE c.program_id = _program_id
    AND COALESCE(c.is_cancelled, false) = false
    AND c.start_date >= (_at AT TIME ZONE 'UTC')::date
  ORDER BY c.start_date ASC LIMIT 1
$function$

```

#### finedu.partner_id()

```sql
CREATE OR REPLACE FUNCTION finedu.partner_id()
 RETURNS uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
  SELECT id FROM public.all_partners WHERE partner_slug = 'finedu-ai'
$function$

```

#### finedu.program_cohorts_del()

```sql
CREATE OR REPLACE FUNCTION finedu.program_cohorts_del()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
BEGIN
  DELETE FROM public.all_cohorts WHERE id = OLD.id;
  RETURN OLD;
END $function$

```

#### finedu.program_cohorts_ins()

```sql
CREATE OR REPLACE FUNCTION finedu.program_cohorts_ins()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE m finedu.program_map%ROWTYPE; new_id uuid;
BEGIN
  SELECT * INTO m FROM finedu.program_map WHERE program_key = NEW.program_key;
  IF m.program_id IS NULL THEN
    RAISE EXCEPTION 'unknown program_key %', NEW.program_key;
  END IF;

  INSERT INTO public.all_cohorts (program_id, cohort_key, cohort_label, program_code,
                                  start_date, end_date, seats_total, seats_filled, is_cancelled)
  VALUES (m.program_id, NEW.cohort_key, NEW.cohort_label,
          m.code_prefix || '-' || to_char(NEW.start_date, 'DDMMYY'),
          NEW.start_date, NEW.start_date + m.duration_days,
          COALESCE(NEW.seats_total, 52), 0, COALESCE(NEW.is_active, true) IS FALSE)
  RETURNING id INTO new_id;

  NEW.id := new_id;
  RETURN NEW;
END $function$

```

#### finedu.program_cohorts_upd()

```sql
CREATE OR REPLACE FUNCTION finedu.program_cohorts_upd()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE m finedu.program_map%ROWTYPE;
BEGIN
  SELECT * INTO m FROM finedu.program_map WHERE program_key = COALESCE(NEW.program_key, OLD.program_key);

  UPDATE public.all_cohorts
     SET cohort_key   = NEW.cohort_key,
         cohort_label = NEW.cohort_label,
         start_date   = NEW.start_date,
         end_date     = CASE WHEN NEW.start_date IS DISTINCT FROM OLD.start_date AND m.duration_days IS NOT NULL
                             THEN NEW.start_date + m.duration_days ELSE end_date END,
         program_code = CASE WHEN NEW.start_date IS DISTINCT FROM OLD.start_date AND m.code_prefix IS NOT NULL
                             THEN m.code_prefix || '-' || to_char(NEW.start_date, 'DDMMYY') ELSE program_code END,
         seats_total  = NEW.seats_total,
         program_id   = COALESCE(m.program_id, program_id),
         is_cancelled = CASE WHEN NEW.is_active IS DISTINCT FROM OLD.is_active
                             THEN NEW.is_active IS FALSE ELSE is_cancelled END,
         updated_at   = now()
   WHERE id = OLD.id;

  RETURN NEW;
END $function$

```

#### finedu.record_addon_payment(_app_id uuid, _payment_link_id text, _payment_intent_id text, _amount numeric, _currency text)

```sql
CREATE OR REPLACE FUNCTION finedu.record_addon_payment(_app_id uuid, _payment_link_id text, _payment_intent_id text, _amount numeric, _currency text)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
declare
  _current jsonb;
  _already boolean;
  _next jsonb;
begin
  select addon_payments into _current from public.finlit_applications where id = _app_id for update;
  if _current is null then
    _current := '[]'::jsonb;
  end if;
-- ... truncated: showing first 15 of 43 lines
```

#### finedu.redeem_voucher(_lead_id uuid, _application_id uuid, _voucher_code text, _amount_usd integer)

```sql
CREATE OR REPLACE FUNCTION finedu.redeem_voucher(_lead_id uuid, _application_id uuid, _voucher_code text, _amount_usd integer)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'finedu', 'public'
AS $function$
DECLARE ok boolean;
BEGIN
  SELECT true INTO ok FROM finedu.concierge_leads
   WHERE id = _lead_id AND upper(voucher_code) = upper(_voucher_code)
     AND (voucher_expires_at IS NULL OR voucher_expires_at > now());
  IF ok IS NOT TRUE THEN RETURN false; END IF;

  INSERT INTO finedu.voucher_redemptions (lead_id, application_id, voucher_code, amount_usd)
  VALUES (_lead_id, _application_id, upper(_voucher_code), _amount_usd);

  UPDATE finedu.concierge_leads SET voucher_redeemed_at = now(), updated_at = now() WHERE id = _lead_id;
  RETURN true;
END $function$

```

#### finedu.tg_finlit_app_catalogue()

```sql
CREATE OR REPLACE FUNCTION finedu.tg_finlit_app_catalogue()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
DECLARE pn text; cat text;
BEGIN
  IF NEW.program_code IS NOT NULL THEN
    SELECT p.program_name, p.categories[1] INTO pn, cat
      FROM public.all_cohorts c JOIN public.all_programs p ON p.id = c.program_id
     WHERE c.program_code = NEW.program_code LIMIT 1;
    IF pn IS NOT NULL THEN
      NEW.program_name := pn;
      NEW.program_category := CASE WHEN cat ILIKE 'fellowship%' THEN 'fellowship'
        WHEN cat ILIKE 'competition%' THEN 'competition' WHEN cat ILIKE 'in person%' THEN 'in-person'
        WHEN cat ILIKE 'camp%' THEN 'camp' WHEN cat ILIKE 'conference%' THEN 'conference'
        WHEN cat ILIKE 'certificate%' THEN 'certificate-program' ELSE NEW.program_category END;
    END IF;
  END IF;
  RETURN NEW;
END $function$

```

#### finedu.tg_finlit_app_format()

```sql
CREATE OR REPLACE FUNCTION finedu.tg_finlit_app_format()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'finedu', 'public'
AS $function$
BEGIN
  NEW.student_name   := btrim(regexp_replace(coalesce(NEW.student_name,''), '\s+', ' ', 'g'));
  NEW.student_email  := lower(btrim(NEW.student_email));
  NEW.parent_email   := nullif(lower(btrim(NEW.parent_email)), '');
  NEW.counsellor_email := nullif(lower(btrim(NEW.counsellor_email)), '');
  NEW.student_mobile := nullif(regexp_replace(coalesce(NEW.student_mobile,''), '\D', '', 'g'), '');
  NEW.parent_phone   := nullif(regexp_replace(coalesce(NEW.parent_phone,''), '\D', '', 'g'), '');
  NEW.currency       := upper(coalesce(nullif(btrim(NEW.currency),''), 'USD'));
  NEW.scholarship_code := nullif(upper(btrim(NEW.scholarship_code)), '');
  NEW.student_gender := nullif(lower(btrim(NEW.student_gender)), '');
  -- grade stored as the bare number like applications ("10"), anything non-numeric kept as chosen
  NEW.grade := nullif(btrim(NEW.grade), '');
  IF NEW.grade ~ '\d' AND NEW.grade ~* '^\s*(grade|year|y)?\s*\d{1,2}\s*(st|nd|rd|th)?\s*(grade)?\s*$' THEN
    NEW.grade := substring(NEW.grade from '\d{1,2}');
  END IF;
  NEW.school_name := nullif(btrim(NEW.school_name), '');
  NEW.country := nullif(btrim(NEW.country), '');
  NEW.state := nullif(btrim(NEW.state), '');
  NEW.city := nullif(btrim(NEW.city), '');
  -- status wiring
  IF NEW.payment_status IN ('paid','refunded') THEN
    NEW.payment_mode := coalesce(NEW.payment_mode, 'full');
    NEW.payment_date := coalesce(NEW.payment_date, now());
  END IF;
  IF NEW.payment_status = 'refunded' THEN NEW.registration_status := 'withdrawn'; END IF;
  IF TG_OP = 'UPDATE' AND OLD.payment_status = 'refunded' AND NEW.payment_status = 'paid' AND NEW.registration_status = 'withdrawn' THEN
    NEW.registration_status := 'complete';
  END IF;
  RETURN NEW;
END $function$

```

#### finedu.tg_finlit_app_view_write()

```sql
CREATE OR REPLACE FUNCTION finedu.tg_finlit_app_view_write()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
DECLARE
  r public.finlit_applications%ROWTYPE;
  fd jsonb; ans jsonb; internal jsonb;
  ins boolean := TG_OP = 'INSERT';
  v_code text; v_first text; v_last text; v_status text;
  ch_first boolean; ch_last boolean;
BEGIN
  IF TG_OP = 'DELETE' THEN
    DELETE FROM public.finlit_applications WHERE id = OLD.id;
-- ... truncated: showing first 15 of 206 lines
```

#### finedu.tg_finlit_shared_student_id()

```sql
CREATE OR REPLACE FUNCTION finedu.tg_finlit_shared_student_id()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
DECLARE found text;
BEGIN
  found := public.resolve_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  IF found IS NOT NULL THEN NEW.student_id := found;
  ELSIF NEW.student_id IS NULL OR NEW.student_id = '' OR public.student_id_in_use(NEW.student_id) THEN
    NEW.student_id := public.get_or_create_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  END IF;
  RETURN NEW;
END $function$

```

#### finedu.touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION finedu.touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'finedu', 'public'
AS $function$
begin new.updated_at = now(); return new; end $function$

```

#### finedu.upsert_pending_addon(_app_id uuid, _addons jsonb)

```sql
CREATE OR REPLACE FUNCTION finedu.upsert_pending_addon(_app_id uuid, _addons jsonb)
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
  update public.finlit_applications set addon_payments = _addons where id = _app_id;
$function$

```

## ops (8 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| agent_dialer_schedule | p_on boolean | void | plpgsql | DEFINER | volatile | search_path="" |  |
| claim_campaign_batch | p_campaign_id uuid | SETOF ops.agent_campaign_contacts | plpgsql | DEFINER | volatile | search_path="" |  |
| enforce_contact_suppression |  | trigger | plpgsql | DEFINER | volatile | search_path=ops, pg_temp |  |
| get_my_roles |  | ops.app_role[] | sql | DEFINER | stable | search_path="" | authenticated |
| has_role | _user_id uuid, _role ops.app_role | boolean | sql | DEFINER | stable | search_path="" | authenticated |
| is_admin |  | boolean | sql | DEFINER | stable | search_path="" | authenticated |
| nora_dialer_schedule | p_on boolean | void | plpgsql | DEFINER | volatile | search_path="" |  |
| touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=ops, public |  |

### ops: bodies

#### ops.agent_dialer_schedule(p_on boolean)

```sql
CREATE OR REPLACE FUNCTION ops.agent_dialer_schedule(p_on boolean)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO ''
AS $function$
BEGIN
IF p_on THEN
IF NOT EXISTS (SELECT 1 FROM cron.job WHERE jobname='ops-agent-campaign-dialer') THEN
PERFORM cron.schedule('ops-agent-campaign-dialer','* * * * *',$job$SELECT net.http_post(
url := 'https://<ESTATE_REF>.supabase.co/functions/v1/ops-agent-campaign-dialer',
headers := '{"Content-Type":"application/json"}'::jsonb,body := '{"source":"cron"}'::jsonb);$job$);
END IF;
ELSE
PERFORM cron.unschedule(jobid) FROM cron.job WHERE jobname='ops-agent-campaign-dialer';
END IF;
END $function$

```

#### ops.claim_campaign_batch(p_campaign_id uuid)

```sql
CREATE OR REPLACE FUNCTION ops.claim_campaign_batch(p_campaign_id uuid)
 RETURNS SETOF ops.agent_campaign_contacts
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO ''
AS $function$
DECLARE c ops.agent_campaigns; campaign_live integer; account_live integer; slots integer;
BEGIN
  PERFORM pg_advisory_xact_lock(71420261114);
  SELECT * INTO c FROM ops.agent_campaigns WHERE id=p_campaign_id FOR UPDATE;
  IF NOT FOUND OR c.status <> 'active' OR NOT c.dialer_running
    OR NOT EXISTS (SELECT 1 FROM ops.agent_profiles WHERE id=c.agent_id AND status='active')
    OR c.last_dial_at > now() - make_interval(secs => coalesce(c.cooldown_seconds,90)) THEN RETURN; END IF;
  SELECT count(*) INTO account_live FROM ops.agent_campaign_calls WHERE status IN ('queued','calling');
  SELECT account_live + count(*) INTO account_live FROM ops.agent_campaign_contacts k
    WHERE k.status='dialing' AND NOT EXISTS (SELECT 1 FROM ops.agent_campaign_calls l WHERE l.contact_id=k.id AND l.status IN ('queued','calling'));
  SELECT count(*) INTO campaign_live FROM ops.agent_campaign_calls WHERE campaign_id=c.id AND status IN ('queued','calling');
  SELECT campaign_live + count(*) INTO campaign_live FROM ops.agent_campaign_contacts k
    WHERE k.campaign_id=c.id AND k.status='dialing' AND NOT EXISTS (SELECT 1 FROM ops.agent_campaign_calls l WHERE l.contact_id=k.id AND l.status IN ('queued','calling'));
  slots := greatest(0,least(coalesce(c.max_concurrent,1)-campaign_live,8-account_live,
    CASE WHEN c.retry_window_days IS NOT NULL THEN 5 ELSE 1 END));
  IF slots=0 THEN RETURN; END IF;
  UPDATE ops.agent_campaign_contacts SET status='retry_expired',next_attempt_at=NULL,updated_at=now()
    WHERE campaign_id=c.id AND status IN ('queued','retry') AND c.retry_window_days IS NOT NULL
      AND retry_started_at + make_interval(days=>c.retry_window_days) <= now();
  RETURN QUERY WITH chosen AS (
    SELECT k.id FROM ops.agent_campaign_contacts k WHERE k.campaign_id=c.id AND NOT k.do_not_call
      AND k.status IN ('queued','retry') AND (k.next_attempt_at IS NULL OR k.next_attempt_at<=now())
      AND ((c.retry_window_days IS NULL AND k.attempts<coalesce(c.max_attempts,3)) OR
        (c.retry_window_days IS NOT NULL AND (k.retry_started_at IS NULL OR k.retry_started_at+make_interval(days=>c.retry_window_days)>now())))
    ORDER BY k.attempts,k.created_at LIMIT slots FOR UPDATE SKIP LOCKED
  ) UPDATE ops.agent_campaign_contacts k SET status='dialing',updated_at=now(),
    retry_started_at=coalesce(k.retry_started_at,now()) FROM chosen WHERE k.id=chosen.id RETURNING k.*;
  IF FOUND THEN UPDATE ops.agent_campaigns SET last_dial_at=now() WHERE id=c.id; END IF;
END $function$

```

#### ops.enforce_contact_suppression()

```sql
CREATE OR REPLACE FUNCTION ops.enforce_contact_suppression()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'ops', 'pg_temp'
AS $function$
BEGIN
 IF NEW.do_not_call THEN
  INSERT INTO ops.contact_suppressions (phone_e164) VALUES (NEW.phone_e164) ON CONFLICT DO NOTHING;
 END IF;
 IF EXISTS (SELECT 1 FROM ops.contact_suppressions s WHERE s.phone_e164 = NEW.phone_e164) THEN
  NEW.do_not_call := true;
  NEW.status := 'suppressed';
  NEW.next_attempt_at := NULL;
 END IF;
 RETURN NEW;
END;
$function$

```

#### ops.get_my_roles()

```sql
CREATE OR REPLACE FUNCTION ops.get_my_roles()
 RETURNS ops.app_role[]
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO ''
AS $function$
  select coalesce(array_agg(role order by case role when 'admin' then 1 when 'partner' then 2 else 3 end),'{}'::ops.app_role[])
  from ops.user_roles where user_id = auth.uid()
$function$

```

#### ops.has_role(_user_id uuid, _role ops.app_role)

```sql
CREATE OR REPLACE FUNCTION ops.has_role(_user_id uuid, _role ops.app_role)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO ''
AS $function$ select case when auth.role() = 'service_role' then exists (select 1 from ops.user_roles where user_id = _user_id and role = _role) when auth.uid() is not null and _user_id = auth.uid() then exists (select 1 from ops.user_roles where user_id = auth.uid() and role = _role) else false end $function$

```

#### ops.is_admin()

```sql
CREATE OR REPLACE FUNCTION ops.is_admin()
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO ''
AS $function$
  select ops.has_role(auth.uid(), 'admin')
$function$

```

#### ops.nora_dialer_schedule(p_on boolean)

```sql
CREATE OR REPLACE FUNCTION ops.nora_dialer_schedule(p_on boolean)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO ''
AS $function$
BEGIN
IF p_on THEN
IF NOT EXISTS (SELECT 1 FROM cron.job WHERE jobname='ops-nora-dialer') THEN
PERFORM cron.schedule('ops-nora-dialer','* * * * *',$job$SELECT net.http_post(
url := 'https://<ESTATE_REF>.supabase.co/functions/v1/ops-nora-dialer',
headers := '{"Content-Type":"application/json"}'::jsonb,body := '{"source":"cron"}'::jsonb);$job$);
END IF;
ELSE
PERFORM cron.unschedule(jobid) FROM cron.job WHERE jobname='ops-nora-dialer';
END IF;
END $function$

```

#### ops.touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION ops.touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'ops', 'public'
AS $function$
begin new.updated_at = now(); return new; end $function$

```

## prelaw (9 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| claim_promo_slot | _code text, _max integer, _holder text | integer | plpgsql | DEFINER | volatile | search_path=prelaw, public |  |
| cohort_slug_from_date | _d date | text | sql | invoker | immutable | search_path=prelaw, public | anon authenticated |
| has_role | _user_id uuid, _role prelaw.app_role | boolean | sql | DEFINER | stable | search_path=prelaw | authenticated |
| mark_promo_slot | _code text, _holder text | void | sql | DEFINER | volatile | search_path=prelaw, public |  |
| record_addon_payment | p_application_id uuid, p_amount numeric, p_currency text, p_payment_intent_id text, p_payment_link_id text, p_label text | TABLE(amount numeric, already_recorded boolean) | plpgsql | DEFINER | volatile | search_path=prelaw |  |
| set_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=prelaw, public | anon authenticated |
| tg_normalize_application_fields |  | trigger | plpgsql | invoker | volatile | search_path=prelaw, public | anon authenticated |
| tg_shared_student_id |  | trigger | plpgsql | DEFINER | volatile | search_path=prelaw, public |  |
| tg_touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=prelaw, public | anon authenticated |

### prelaw: bodies

#### prelaw.claim_promo_slot(_code text, _max integer, _holder text)

```sql
CREATE OR REPLACE FUNCTION prelaw.claim_promo_slot(_code text, _max integer, _holder text)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'prelaw', 'public'
AS $function$
declare used int; c text := upper(trim(_code)); h text := lower(trim(_holder));
begin
  if c = '' or h = '' then return 0; end if;
  perform pg_advisory_xact_lock(hashtext('prelaw.promo:' || c));
  insert into prelaw.promo_slot_claims (code, holder)
  values (c, h)
  on conflict (code, holder) do update set created_at = now();
  select count(*) into used
    from prelaw.promo_slot_claims
   where code = c
     and succeeded = false
     and created_at > now() - interval '30 minutes';
  if _max is not null and used > _max then
    delete from prelaw.promo_slot_claims
     where code = c and holder = h and succeeded = false;
    return -1;
  end if;
  return used;
end;
$function$

```

#### prelaw.cohort_slug_from_date(_d date)

```sql
CREATE OR REPLACE FUNCTION prelaw.cohort_slug_from_date(_d date)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'prelaw', 'public'
AS $function$
  select case when _d is null then null else 'lawyers_' || lower(to_char(_d,'FMMon')) || to_char(_d,'YY') end $function$

```

#### prelaw.has_role(_user_id uuid, _role prelaw.app_role)

```sql
CREATE OR REPLACE FUNCTION prelaw.has_role(_user_id uuid, _role prelaw.app_role)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'prelaw'
AS $function$ select exists (select 1 from prelaw.user_roles where user_id = _user_id and role = _role)
   and (_user_id = auth.uid() or coalesce(auth.role(),'') = 'service_role' or current_user in ('postgres','service_role')) $function$

```

#### prelaw.mark_promo_slot(_code text, _holder text)

```sql
CREATE OR REPLACE FUNCTION prelaw.mark_promo_slot(_code text, _holder text)
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'prelaw', 'public'
AS $function$
  update prelaw.promo_slot_claims
     set succeeded = true
   where code = upper(trim(_code)) and holder = lower(trim(_holder));
$function$

```

#### prelaw.record_addon_payment(p_application_id uuid, p_amount numeric, p_currency text, p_payment_intent_id text, p_payment_link_id text, p_label text)

```sql
CREATE OR REPLACE FUNCTION prelaw.record_addon_payment(p_application_id uuid, p_amount numeric, p_currency text, p_payment_intent_id text, p_payment_link_id text, p_label text DEFAULT NULL::text)
 RETURNS TABLE(amount numeric, already_recorded boolean)
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'prelaw'
AS $function$
declare
  v_already boolean;
begin
  select exists(
    select 1
    from prelaw.lawyers_applications a,
         jsonb_array_elements(coalesce(a.form_data->'addon_payments', '[]'::jsonb)) as item
    where a.id = p_application_id
      and (
-- ... truncated: showing first 15 of 48 lines
```

#### prelaw.set_updated_at()

```sql
CREATE OR REPLACE FUNCTION prelaw.set_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'prelaw', 'public'
AS $function$ begin new.updated_at = now(); return new; end $function$

```

#### prelaw.tg_normalize_application_fields()

```sql
CREATE OR REPLACE FUNCTION prelaw.tg_normalize_application_fields()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'prelaw', 'public'
AS $function$
begin
  new.student_email := lower(btrim(new.student_email));
  new.parent_email := nullif(lower(btrim(new.parent_email)), '');
  new.counsellor_email := nullif(lower(btrim(new.counsellor_email)), '');
  new.student_name := regexp_replace(btrim(new.student_name), '\s+', ' ', 'g');
  new.student_mobile := nullif(regexp_replace(coalesce(new.student_mobile,''), '\D', '', 'g'), '');
  new.parent_phone := nullif(regexp_replace(coalesce(new.parent_phone,''), '\D', '', 'g'), '');
  new.currency := upper(coalesce(new.currency, 'USD'));
  new.scholarship_code := nullif(upper(btrim(new.scholarship_code)), '');
  new.payment_status := lower(new.payment_status);
  return new;
end $function$

```

#### prelaw.tg_shared_student_id()

```sql
CREATE OR REPLACE FUNCTION prelaw.tg_shared_student_id()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'prelaw', 'public'
AS $function$
declare found text;
begin
  found := public.resolve_student_id(new.student_email, new.student_mobile, new.student_name);
  if found is not null then new.student_id := found;
  elsif new.student_id is null or new.student_id = '' or public.student_id_in_use(new.student_id) then
    new.student_id := public.get_or_create_student_id(new.student_email, new.student_mobile, new.student_name);
  end if;
  return new;
end $function$

```

#### prelaw.tg_touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION prelaw.tg_touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'prelaw', 'public'
AS $function$
begin new.updated_at := now(); return new; end $function$

```

## lms (133 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| _app_col | p_table text, p_cands text[] | text | sql | invoker | stable | search_path=pg_temp | anon authenticated |
| _channel_member_raw | _user_id uuid, _channel_id uuid | boolean | sql | DEFINER | stable | search_path=lms |  |
| _mentor_cohorts_for_user | _user uuid | SETOF uuid | sql | DEFINER | stable | search_path=lms, public |  |
| _raw_masterclass_regs |  | TABLE(platform_key text, email text, student_name text, registered_at timestamp with time zone, session_label text, masterclass_id uuid) | sql | DEFINER | stable | search_path=lms, public, pg_temp |  |
| _vault_cohort_started | p_cohort uuid | boolean | sql | DEFINER | stable | search_path=lms, public, pg_temp |  |
| _vault_expire_vouchers_for | p_student uuid | integer | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| _vault_release_voucher | p_voucher uuid, p_status text | text | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| _vault_reserve | p_student uuid, p_wallet text, p_amount numeric, p_voucher uuid, p_code text | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| active_students_7d | p_exclude uuid[] | integer | plpgsql | DEFINER | stable | search_path=lms, auth, public | authenticated |
| add_community_welcome_message |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| add_shared_whatsapp_blocks | p_phones text[], p_reason text, p_source text | integer | plpgsql | DEFINER | volatile | search_path=pg_catalog, lms, portal |  |
| admin_add_shared_whatsapp_blocks | p_phones text[], p_reason text, p_source text, p_actor uuid | integer | plpgsql | invoker | volatile | search_path=pg_catalog, lms, portal |  |
| admin_mentor_emails |  | TABLE(mentor_id uuid, email text) | sql | DEFINER | stable | search_path=lms, public | authenticated |
| admin_set_mentor_email | _mentor_id uuid, _email text | void | plpgsql | DEFINER | volatile | search_path=lms, public | authenticated |
| admin_unblock_shared_whatsapp_number | p_phone text, p_reason text, p_actor uuid | boolean | plpgsql | invoker | volatile | search_path=pg_catalog, lms, portal |  |
| am_i_registered_for | _masterclass_id uuid | boolean | sql | DEFINER | stable | search_path=lms, public, pg_temp | authenticated |
| apply_enrollment_change | p jsonb | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| auth_user_id_by_email | p_email text | uuid | sql | DEFINER | stable | search_path=auth, pg_temp |  |
| birthday_zone_for | p_country text, p_state text, p_phone text | text | plpgsql | invoker | immutable | search_path=lms, pg_temp | anon authenticated |
| can_view_channel | _user_id uuid, _channel_id uuid | boolean | sql | DEFINER | stable | search_path=lms, public | authenticated |
| claim_login_rollout_item |  | SETOF lms.login_rollout_queue | plpgsql | DEFINER | volatile | search_path=lms |  |
| claim_whatsapp_reminders | p_session_id uuid, p_offset_kind text, p_claim_token uuid, p_items jsonb | TABLE(student_user_id uuid, channel text) | plpgsql | DEFINER | volatile | search_path=lms, pg_temp |  |
| clear_test_seat | p_user uuid, p_cohort uuid | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| close_account_if_unenrolled | p_user_id uuid, p_reason text | boolean | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| community_members | _channel_id uuid | TABLE(user_id uuid, name text, role text) | sql | DEFINER | stable | search_path=lms, public | authenticated |
| community_messages_moderate |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| cooldown_login_rollout | p_minutes integer | void | sql | DEFINER | volatile | search_path=lms |  |
| defer_login_rollout_item | p_id uuid, p_detail text, p_ctx jsonb, p_minutes integer, p_count_attempt boolean | void | plpgsql | DEFINER | volatile | search_path=lms |  |
| ensure_community_channel |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| feedback_reward_offer |  | integer | sql | DEFINER | stable | search_path=lms, public, pg_temp | authenticated |
| finalize_join_attendance |  | integer | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| finish_login_rollout_item | p_id uuid, p_status text, p_result text, p_detail text, p_email_sent boolean | void | plpgsql | DEFINER | volatile | search_path=lms |  |
| finish_whatsapp_reminders | p_session_id uuid, p_offset_kind text, p_claim_token uuid, p_items jsonb, p_broadcast_ids text[] | integer | plpgsql | DEFINER | volatile | search_path=lms, pg_temp |  |
| get_masterclass_registration_counts |  | TABLE(masterclass_id uuid, registered_count integer) | sql | DEFINER | stable | search_path=lms, public, pg_temp | authenticated |
| get_masterclass_registrations |  | TABLE(platform_key text, email text, student_name text, registered_at timestamp with time zone, session_label text, masterclass_id uuid) | sql | DEFINER | stable | search_path=lms, public, pg_temp | authenticated |
| grade_reminder | p_year integer | integer | plpgsql | DEFINER | volatile | search_path=lms |  |
| has_role | _user_id uuid, _role text | boolean | sql | DEFINER | stable | search_path=lms, public | authenticated |
| is_channel_member | _user_id uuid, _channel_id uuid | boolean | sql | DEFINER | stable | search_path=lms | authenticated |
| is_cohort_mentor | _cohort_id uuid | boolean | sql | DEFINER | stable | search_path=lms | authenticated |
| is_cohort_student | _cohort_id uuid | boolean | sql | DEFINER | stable | search_path=lms | authenticated |
| is_my_catalog_mentor | _mentor_id uuid | boolean | sql | DEFINER | stable | search_path=lms | authenticated |
| issue_certificates | p_cohort_id uuid, p_kind text, p_issue_date date, p_items jsonb | integer | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp | authenticated |
| mark_birthday_popup_seen |  | void | sql | DEFINER | volatile | search_path=lms, public, pg_temp | authenticated |
| mark_certificates_seen | p_ids uuid[] | void | sql | DEFINER | volatile | search_path=lms, public | authenticated |
| mark_referral_promo_seen |  | void | sql | DEFINER | volatile | search_path=lms | authenticated |
| mc_norm | t text | text | sql | invoker | immutable | search_path=lms, public | anon authenticated |
| mentor_cohort_students | _cohort_id uuid, _exclude_emails text[] | TABLE(user_id uuid, full_name text, grade text, school_name text, group_name text, present_count integer, session_count integer) | sql | DEFINER | stable | search_path=lms | authenticated |
| my_birthday_offer |  | TABLE(grant_id uuid, amount numeric, remaining numeric, expires_at timestamp with time zone, popup_seen boolean, zone text) | sql | DEFINER | stable | search_path=lms, public, pg_temp | authenticated |
| my_certificates |  | TABLE(id uuid, cohort_id uuid, program_id uuid, kind text, issue_date date, tag_values jsonb, issued_at timestamp with time zone, seen_at timestamp with time zone, html text) | sql | DEFINER | stable | search_path=lms, public | authenticated |
| my_community_summary |  | TABLE(channel_id uuid, cohort_id uuid, unread_count integer, last_message_at timestamp with time zone, last_message_body text, last_message_sender text) | sql | DEFINER | stable | search_path=lms, public | authenticated |
| my_hosted_masterclasses |  | TABLE(id uuid, title text, series text, session_number integer, audience text, scheduled_at timestamp with time zone, duration_minutes integer, display_timezone text, join_url text, passcode text, apply_url text) | sql | DEFINER | stable | search_path=lms, public | authenticated |
| my_marked_sessions |  | TABLE(session_id uuid, cohort_id uuid, session_datetime timestamp with time zone) | sql | DEFINER | stable | search_path=lms | authenticated |
| my_mentor_cohorts |  | SETOF uuid | sql | DEFINER | stable | search_path=lms | authenticated |
| my_mentor_feedback |  | TABLE(session_id uuid, cohort_id uuid, title text, session_datetime timestamp with time zone, responses integer, avg_session numeric, avg_mentor numeric, avg_engagement numeric, comments text[]) | sql | DEFINER | stable | search_path=lms | authenticated |
| my_mentor_feedback_entries |  | TABLE(id uuid, session_id uuid, cohort_id uuid, session_title text, session_datetime timestamp with time zone, session_quality_rating integer, mentor_rating integer, engagement_rating integer, takeaway text, submitted_at timestamp with time zone) | sql | DEFINER | stable | search_path=lms | authenticated |
| my_mentor_id |  | uuid | sql | DEFINER | stable | search_path=lms | authenticated |
| my_mentor_profile |  | TABLE(mentor_id uuid, full_name text, university text, country text, bio text, image_url text, email text) | sql | DEFINER | stable | search_path=lms, public | authenticated |
| my_referral_offer |  | jsonb | sql | DEFINER | stable | search_path=lms, public, pg_temp | authenticated |
| my_referral_promo_seen |  | boolean | sql | DEFINER | stable | search_path=lms | authenticated |
| normalize_session_mode |  | trigger | plpgsql | invoker | volatile | search_path=lms, public, pg_temp | anon authenticated |
| post_due_community_session_reminders | p_dry_run boolean | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| promote_grades | p_year integer | integer | plpgsql | DEFINER | volatile | search_path=lms |  |
| purge_test_cohort_data | p_user uuid, p_cohort uuid | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| record_community_mentions |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| record_recording_watch | p_session_id uuid | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public | authenticated |
| record_session_join | p_session_id uuid | boolean | plpgsql | DEFINER | volatile | search_path=lms, public | authenticated |
| referral_begin_verification | p_invite uuid, p_hash text | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| referral_check_verification | p_invite uuid, p_hash text | boolean | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| referral_create_invite | p_inviter uuid, p_details jsonb, p_hash text | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| referral_email_identity | p_email text | text | sql | invoker | immutable | search_path=pg_catalog |  |
| referral_finish_join | p_invite uuid, p_friend uuid | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| referral_followup_claim | p_invite uuid, p_hash text | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| referral_followup_unclaim | p_invite uuid, p_prev_sent_at timestamp with time zone | void | sql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| referral_resend_invite | p_inviter uuid, p_invite uuid, p_hash text | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| revoke_certificate | p_id uuid | void | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp | authenticated |
| revoke_user_sessions | p_user uuid | integer | plpgsql | DEFINER | volatile | search_path=auth, lms |  |
| screen_community_message | p_body text | text | plpgsql | invoker | immutable | search_path=lms, public, pg_temp | anon authenticated |
| screen_contact_number | p_body text | boolean | plpgsql | invoker | immutable | search_path=lms, public, pg_temp | authenticated |
| stamp_community_automation_team |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| stamp_community_sender |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| start_reconcile_run | p_trigger text, p_mode text, p_actor uuid | uuid | plpgsql | DEFINER | volatile | search_path=lms |  |
| sync_profiles_from_applications | p_user uuid | integer | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| t_vault_welcome_on_profile |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_all_mentors_view_write |  | trigger | plpgsql | invoker | volatile | search_path=public, lms | anon authenticated |
| tg_dob_guard |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_enrollment_events_immutable |  | trigger | plpgsql | invoker | volatile | search_path=lms | anon authenticated |
| tg_group_member_section_check |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| tg_profile_referral_guard |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_referral_application |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_section_member_cleanup |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| tg_submission_review_guard |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_ticket_touch |  | trigger | plpgsql | invoker | volatile | search_path=lms, public, pg_temp | anon authenticated |
| tg_vault_cashback |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_vault_cohort_completed |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_vault_earn_attendance |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_vault_earn_feedback |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_vault_earn_project |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_vault_ledger_immutable |  | trigger | plpgsql | invoker | volatile | search_path=lms | anon authenticated |
| tg_vault_lot_notify |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| tg_vault_referral_reward |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=lms, public | anon authenticated |
| unblock_shared_whatsapp_number | p_phone text, p_reason text | boolean | plpgsql | DEFINER | volatile | search_path=pg_catalog, lms, portal |  |
| update_my_mentor_profile | p_full_name text, p_university text, p_country text, p_bio text | void | plpgsql | DEFINER | volatile | search_path=lms, public | authenticated |
| update_vault_settings | p_patch jsonb, p_reason text, p_actor uuid | lms.vault_settings | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| validate_whatsapp_recipient_config |  | trigger | plpgsql | invoker | volatile | search_path=pg_catalog, lms | anon authenticated |
| vault_adjust | p_student uuid, p_wallet text, p_amount numeric, p_reason text, p_actor uuid | lms.vault_lots | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_birthday_due |  | TABLE(grant_id uuid) | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_birthday_expire |  | integer | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_cancel_voucher | p_student uuid, p_voucher uuid | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_commit_voucher_from_redemption | p_redemption_id uuid | void | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_coupon_reminders |  | integer | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_create_voucher | p_student uuid, p_enrollment uuid, p_effort numeric, p_loyalty numeric | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_create_voucher_program | p_student uuid, p_program uuid, p_effort numeric, p_loyalty numeric | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_daily |  | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_earn | p_student uuid, p_wallet text, p_amount numeric, p_source_type text, p_source_key text, p_state text, p_enrollment uuid, p_cohort uuid, p_release_on timestamp with time zone, p_counts_toward_level boolean, p_description text | lms.vault_lots | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_expire_due |  | TABLE(wallet text, lots_expired integer, amount_expired numeric) | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_expire_vouchers |  | integer | sql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_flag_auto_resolve |  | trigger | plpgsql | DEFINER | volatile | search_path=lms, public |  |
| vault_grant_welcome | p_student uuid | void | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_integrity_report |  | TABLE(student_user_id uuid, wallet text, lot_remaining numeric, ledger_balance numeric) | sql | DEFINER | stable | search_path=lms, pg_temp |  |
| vault_lifetime_points | p_student uuid | numeric | sql | DEFINER | stable | search_path=lms, pg_temp |  |
| vault_mark_walkthrough_seen |  | void | plpgsql | DEFINER | volatile | search_path=lms, public | authenticated |
| vault_notify | p_student uuid, p_title text, p_body text | void | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_program_kind | p_cohort uuid | text | sql | DEFINER | stable | search_path=lms, public, pg_temp |  |
| vault_public_settings |  | jsonb | sql | DEFINER | stable | search_path=lms, public | authenticated |
| vault_quote | p_student uuid, p_enrollment uuid, p_effort numeric, p_loyalty numeric | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_quote_program | p_student uuid, p_program uuid, p_effort numeric, p_loyalty numeric | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_redeem_catalog | p_student uuid | jsonb | sql | DEFINER | stable | search_path=lms, public, pg_temp |  |
| vault_redemptions |  | jsonb | plpgsql | DEFINER | volatile | search_path=lms, public | authenticated |
| vault_referral_overview |  | jsonb | sql | DEFINER | stable | search_path=lms, public, pg_temp |  |
| vault_release | p_source_key text | lms.vault_lots | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |
| vault_report | p_from timestamp with time zone, p_to timestamp with time zone, p_exclude uuid[] | jsonb | plpgsql | DEFINER | stable | search_path=lms, public, pg_temp | authenticated |
| vault_reverse | p_source_key text | lms.vault_lots | plpgsql | DEFINER | volatile | search_path=lms, public, pg_temp |  |

### lms: bodies

#### lms._app_col(p_table text, p_cands text[])

```sql
CREATE OR REPLACE FUNCTION lms._app_col(p_table text, p_cands text[])
 RETURNS text
 LANGUAGE sql
 STABLE
 SET search_path TO 'pg_temp'
AS $function$
  select coalesce((select quote_ident(c) from unnest(p_cands) with ordinality u(c, i)
    where exists (select 1 from information_schema.columns where table_schema='public' and table_name=p_table and column_name=c)
    order by i limit 1), 'null');
$function$

```

#### lms._channel_member_raw(_user_id uuid, _channel_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms._channel_member_raw(_user_id uuid, _channel_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select exists (select 1 from lms.community_channels c join lms.enrollments e on e.cohort_id = c.cohort_id
                 where c.id = _channel_id and e.student_user_id = _user_id and e.access_status = 'active')
      or exists (select 1 from lms.community_channels c where c.id = _channel_id
                 and c.cohort_id in (select lms._mentor_cohorts_for_user(_user_id)))
      or exists (select 1 from lms.user_roles where user_id = _user_id and role = 'admin')
$function$

```

#### lms._mentor_cohorts_for_user(_user uuid)

```sql
CREATE OR REPLACE FUNCTION lms._mentor_cohorts_for_user(_user uuid)
 RETURNS SETOF uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  with m as (select mentor_id from lms.mentor_accounts where user_id = _user)
  select s.cohort_default from (
    select cohort_id as cohort_default from lms.cohort_group_settings where default_mentor_id in (select mentor_id from m)
    union select cohort_id from lms.cohort_groups where mentor_id in (select mentor_id from m)
    union select cohort_id from lms.session_mentor_overrides where mentor_id in (select mentor_id from m)
    union select se.cohort_id from lms.session_mentors sm join lms.sessions se on se.id = sm.session_id where sm.mentor_id in (select mentor_id from m)
  ) s where s.cohort_default is not null
$function$

```

#### lms._raw_masterclass_regs()

```sql
CREATE OR REPLACE FUNCTION lms._raw_masterclass_regs()
 RETURNS TABLE(platform_key text, email text, student_name text, registered_at timestamp with time zone, session_label text, masterclass_id uuid)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  with mc as materialized (
    select m1.id, m1.scheduled_at, lms.mc_norm(m1.slug) as ns, lms.mc_norm(m1.title) as nt
    from public.all_masterclasses m1
  ),
  base as (
    select 'fellowship'::text as platform_key, r.email,
           coalesce(nullif(r.child_name,''), r.full_name) as student_name,
           r.created_at, unnest(coalesce(r.sessions,'{}'::text[])) as session_label
    from public.fellowship_masterclass_regs r
-- ... truncated: showing first 15 of 52 lines
```

#### lms._vault_cohort_started(p_cohort uuid)

```sql
CREATE OR REPLACE FUNCTION lms._vault_cohort_started(p_cohort uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  select exists (
    select 1 from lms.all_cohorts c
    where c.id = p_cohort
      and (
        (c.start_date is not null and c.start_date <= current_date)
        or (c.start_date is null and exists (
              select 1 from lms.sessions s
              where s.cohort_id = c.id and s.session_datetime <= now()))
      )
  );
$function$

```

#### lms._vault_expire_vouchers_for(p_student uuid)

```sql
CREATE OR REPLACE FUNCTION lms._vault_expire_vouchers_for(p_student uuid)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare r record; n int := 0;
begin
  for r in select id from lms.vault_vouchers where status = 'active' and expires_at <= now()
           and (p_student is null or student_user_id = p_student) loop
    if lms._vault_release_voucher(r.id, 'expired') = 'expired' then n := n + 1; end if;
  end loop;
  return n;
end $function$

```

#### lms._vault_release_voucher(p_voucher uuid, p_status text)

```sql
CREATE OR REPLACE FUNCTION lms._vault_release_voucher(p_voucher uuid, p_status text)
 RETURNS text
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.vault_vouchers; a jsonb; v_lot lms.vault_lots; v_used int;
begin
  select * into v from lms.vault_vouchers where id = p_voucher for update;
  if not found or v.status <> 'active' then return coalesce(v.status, 'not_found'); end if;
  perform 1 from public.promo_codes where id = v.promo_code_id for update;
  select count(*) into v_used from public.promo_redemptions where promo_code_id = v.promo_code_id or promo_code = v.code;
  if v_used > 0 then return 'used'; end if;
  update public.promo_codes set is_active = false, updated_at = now() where id = v.promo_code_id;
  for a in select * from jsonb_array_elements(v.allocations) loop
    select * into v_lot from lms.vault_lots where id = (a->>'lot_id')::uuid for update;
    insert into lms.vault_ledger (student_user_id, wallet, lot_id, kind, amount, ref_type, ref_id, description)
      values (v.student_user_id, a->>'wallet', v_lot.id, 'unreserve', (a->>'amount')::numeric, 'voucher', v.id::text,
        case when p_status = 'expired' then 'Coupon ' || v.code || ' expired · credit returned' else 'Coupon ' || v.code || ' cancelled · credit returned' end);
    if v_lot.state = 'available' then
      update lms.vault_lots set remaining = remaining + (a->>'amount')::numeric where id = v_lot.id;
    else
      -- The lot expired/reversed while held: the returned part expires with it.
      insert into lms.vault_ledger (student_user_id, wallet, lot_id, kind, amount, ref_type, ref_id, description)
        values (v.student_user_id, a->>'wallet', v_lot.id, 'expire', -(a->>'amount')::numeric, 'voucher', v.id::text, 'Returned credit had already expired');
    end if;
  end loop;
  update lms.vault_vouchers set status = p_status, cancelled_at = now() where id = v.id;
  return p_status;
end $function$

```

#### lms._vault_reserve(p_student uuid, p_wallet text, p_amount numeric, p_voucher uuid, p_code text)

```sql
CREATE OR REPLACE FUNCTION lms._vault_reserve(p_student uuid, p_wallet text, p_amount numeric, p_voucher uuid, p_code text)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare r record; v_need numeric := p_amount; v_take numeric; v_alloc jsonb := '[]'::jsonb;
begin
  if p_amount <= 0 then return v_alloc; end if;
  for r in select id, remaining from lms.vault_lots
    where student_user_id = p_student and wallet = p_wallet and state = 'available' and remaining > 0
    order by expires_at nulls last, created_at for update loop
    exit when v_need <= 0;
    v_take := least(r.remaining, v_need);
    update lms.vault_lots set remaining = remaining - v_take where id = r.id;
    insert into lms.vault_ledger (student_user_id, wallet, lot_id, kind, amount, ref_type, ref_id, description)
      values (p_student, p_wallet, r.id, 'reserve', -v_take, 'voucher', p_voucher::text, 'Held for coupon ' || p_code);
    v_alloc := v_alloc || jsonb_build_object('lot_id', r.id, 'wallet', p_wallet, 'amount', v_take);
    v_need := v_need - v_take;
  end loop;
  if v_need > 0.001 then raise exception 'insufficient_balance'; end if;
  return v_alloc;
end $function$

```

#### lms.active_students_7d(p_exclude uuid[])

```sql
CREATE OR REPLACE FUNCTION lms.active_students_7d(p_exclude uuid[] DEFAULT '{}'::uuid[])
 RETURNS integer
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'auth', 'public'
AS $function$
declare n integer;
begin
  if not lms.has_role(auth.uid(), 'admin') then
    raise exception 'admin only' using errcode = '42501';
  end if;
  select count(*) into n
  from lms.student_profiles p
  join auth.users u on u.id = p.user_id
  where not (p.user_id = any(coalesce(p_exclude, '{}')))
    and (u.last_sign_in_at > now() - interval '7 days'
      or exists (select 1 from auth.sessions s
                 where s.user_id = u.id
                   and coalesce(s.refreshed_at, s.updated_at, s.created_at) > now() - interval '7 days'));
  return n;
end $function$

```

#### lms.add_community_welcome_message()

```sql
CREATE OR REPLACE FUNCTION lms.add_community_welcome_message()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare
  team_user_id uuid;
  programme_name text;
begin
  select ur.user_id into team_user_id
  from lms.user_roles ur
  where ur.role = 'admin'
  order by ur.user_id
  limit 1;

  if team_user_id is null then
    raise exception 'Cannot create community welcome message without an admin account';
  end if;

  select coalesce(p.program_name, 'the Group Programme') into programme_name
  from public.all_cohorts c
  left join public.all_programs p on p.id = c.program_id
  where c.id = new.cohort_id;

  insert into lms.community_messages
    (channel_id, sender_user_id, sender_name, sender_role, body)
  values
    (new.id, team_user_id, 'the Group', 'admin',
     'Greetings from the Group!' || E'\n\n' ||
     'Welcome to the ' || coalesce(programme_name, 'the Group Programme') ||
     '! This community chat is a space to connect with fellow participants and the the Group team. ' ||
     'To maintain a respectful and focused environment, our team is also present to assist whenever needed.' ||
     E'\n\nThank you!');

  return new;
end;
$function$

```

#### lms.add_shared_whatsapp_blocks(p_phones text[], p_reason text, p_source text)

```sql
CREATE OR REPLACE FUNCTION lms.add_shared_whatsapp_blocks(p_phones text[], p_reason text, p_source text DEFAULT NULL::text)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'pg_catalog', 'lms', 'portal'
AS $function$
DECLARE v_phone text; v_saved integer:=0; v_count integer;
BEGIN
  IF auth.uid() IS NULL OR NOT lms.has_role(auth.uid(),'admin') THEN
    RAISE EXCEPTION 'Admin access required' USING ERRCODE='42501';
  END IF;
  IF p_reason IS NULL OR p_reason NOT IN ('blocked','undelivered') OR p_phones IS NULL OR cardinality(p_phones) NOT BETWEEN 1 AND 200
     OR (p_source IS NOT NULL AND length(p_source) NOT BETWEEN 1 AND 255) THEN
    RAISE EXCEPTION 'Invalid blocklist request';
  END IF;
  PERFORM pg_advisory_xact_lock(hashtextextended('portal.whatsapp_undeliverable_numbers',0));
  FOREACH v_phone IN ARRAY p_phones LOOP
    IF v_phone IS NULL OR v_phone !~ '^[1-9][0-9]{7,14}$' THEN
      RAISE EXCEPTION 'Full international number required';
    END IF;
    INSERT INTO portal.whatsapp_undeliverable_numbers
      (phone,national_number,country,last_failure_date,source_file,last_action_by,lms_block_reason)
    VALUES (v_phone,v_phone,'Unknown',(now() AT TIME ZONE 'UTC')::date,
      coalesce(p_source,'LMS Manual Block'),auth.uid(),p_reason)
    ON CONFLICT (phone) DO NOTHING;
    GET DIAGNOSTICS v_count=ROW_COUNT;
    v_saved:=v_saved+v_count;
    -- Manual blocks may explicitly re-block; plain number-file imports never
    -- override a prior unblock without dated failure evidence.
    IF p_source IS NULL AND v_count=0 THEN
      PERFORM portal.set_whatsapp_undeliverable_status(v_phone,true,'Blocked From LMS Admin',auth.uid());
    END IF;
  END LOOP;
  RETURN v_saved;
END $function$

```

#### lms.admin_add_shared_whatsapp_blocks(p_phones text[], p_reason text, p_source text, p_actor uuid)

```sql
CREATE OR REPLACE FUNCTION lms.admin_add_shared_whatsapp_blocks(p_phones text[], p_reason text, p_source text, p_actor uuid)
 RETURNS integer
 LANGUAGE plpgsql
 SET search_path TO 'pg_catalog', 'lms', 'portal'
AS $function$
DECLARE v_phone text; v_count integer; v_saved integer:=0;
BEGIN
 IF auth.role() IS DISTINCT FROM 'service_role' OR p_actor IS NULL OR NOT lms.has_role(p_actor,'admin') THEN RAISE EXCEPTION 'Admin access required' USING ERRCODE='42501'; END IF;
 IF p_reason IS NULL OR p_reason NOT IN ('blocked','undelivered') OR p_phones IS NULL OR cardinality(p_phones) NOT BETWEEN 1 AND 200 OR (p_source IS NOT NULL AND length(p_source) NOT BETWEEN 1 AND 255) THEN RAISE EXCEPTION 'Invalid request'; END IF;
 PERFORM pg_advisory_xact_lock(hashtextextended('portal.whatsapp_undeliverable_numbers',0));
 FOREACH v_phone IN ARRAY p_phones LOOP
  IF v_phone IS NULL OR v_phone !~ '^[1-9][0-9]{7,14}$' THEN RAISE EXCEPTION 'Invalid international number'; END IF;
  INSERT INTO portal.whatsapp_undeliverable_numbers(phone,national_number,country,last_failure_date,source_file,last_action_by,lms_block_reason) VALUES(v_phone,v_phone,'Unknown',(now() AT TIME ZONE 'UTC')::date,coalesce(p_source,'LMS Manual Block'),p_actor,p_reason) ON CONFLICT(phone) DO NOTHING;
  GET DIAGNOSTICS v_count=ROW_COUNT; v_saved:=v_saved+v_count;
  IF p_source IS NULL AND v_count=0 THEN PERFORM portal.set_whatsapp_undeliverable_status(v_phone,true,'Blocked From LMS Admin',p_actor); END IF;
 END LOOP;
 RETURN v_saved;
END $function$

```

#### lms.admin_mentor_emails()

```sql
CREATE OR REPLACE FUNCTION lms.admin_mentor_emails()
 RETURNS TABLE(mentor_id uuid, email text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  select m.id, m.email from public.all_mentors m where lms.has_role(auth.uid(), 'admin')
$function$

```

#### lms.admin_set_mentor_email(_mentor_id uuid, _email text)

```sql
CREATE OR REPLACE FUNCTION lms.admin_set_mentor_email(_mentor_id uuid, _email text)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare v text := nullif(lower(trim(coalesce(_email, ''))), '');
begin
  if not lms.has_role(auth.uid(), 'admin') then raise exception 'not_allowed'; end if;
  if v is not null and v !~ '^[^@\s]+@[^@\s]+\.[^@\s]+$' then raise exception 'invalid_email'; end if;
  if v is not null and exists (select 1 from public.all_mentors where lower(email) = v and id <> _mentor_id) then
    raise exception 'email_in_use';
  end if;
  if exists (select 1 from lms.mentor_accounts where mentor_id = _mentor_id)
     and v is distinct from (select lower(email) from public.all_mentors where id = _mentor_id) then
    raise exception 'has_access';
  end if;
  update public.all_mentors set email = v, updated_at = now() where id = _mentor_id;
end $function$

```

#### lms.admin_unblock_shared_whatsapp_number(p_phone text, p_reason text, p_actor uuid)

```sql
CREATE OR REPLACE FUNCTION lms.admin_unblock_shared_whatsapp_number(p_phone text, p_reason text, p_actor uuid)
 RETURNS boolean
 LANGUAGE plpgsql
 SET search_path TO 'pg_catalog', 'lms', 'portal'
AS $function$
DECLARE v_exact boolean; v_prefixed boolean;
BEGIN
 IF auth.role() IS DISTINCT FROM 'service_role' OR p_actor IS NULL OR NOT lms.has_role(p_actor,'admin') THEN RAISE EXCEPTION 'Admin access required' USING ERRCODE='42501'; END IF;
 IF p_phone IS NULL OR p_phone !~ '^[1-9][0-9]{7,14}$' THEN RAISE EXCEPTION 'Invalid international number'; END IF;
 v_exact:=portal.set_whatsapp_undeliverable_status(p_phone,false,p_reason,p_actor);
 v_prefixed:=portal.set_whatsapp_undeliverable_status('00'||p_phone,false,p_reason,p_actor);
 RETURN v_exact OR v_prefixed;
END $function$

```

#### lms.am_i_registered_for(_masterclass_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.am_i_registered_for(_masterclass_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
    select lms.has_role(auth.uid(), 'admin')
        or exists (
            select 1 from lms._raw_masterclass_regs() r
            where r.masterclass_id = _masterclass_id
              and r.email = lower(coalesce(auth.jwt() ->> 'email', ''))
        );
$function$

```

#### lms.apply_enrollment_change(p jsonb)

```sql
CREATE OR REPLACE FUNCTION lms.apply_enrollment_change(p jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare
  v_type text := p->>'type';
  v_source text := p->>'source';
  v_uid uuid := nullif(p->>'student_user_id','')::uuid;
  v_cohort uuid := (p->>'cohort_id')::uuid;
  v_actor uuid := nullif(p->>'actor_user_id','')::uuid;
  v_reason text := p->>'reason';
  v_ext text := nullif(p->>'external_event_id','');
  v_at timestamptz := coalesce(nullif(p->>'occurred_at','')::timestamptz, now());
-- ... truncated: showing first 15 of 160 lines
```

#### lms.auth_user_id_by_email(p_email text)

```sql
CREATE OR REPLACE FUNCTION lms.auth_user_id_by_email(p_email text)
 RETURNS uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'auth', 'pg_temp'
AS $function$
  select id from auth.users where lower(email) = lower(trim(p_email)) limit 1
$function$

```

#### lms.birthday_zone_for(p_country text, p_state text, p_phone text)

```sql
CREATE OR REPLACE FUNCTION lms.birthday_zone_for(p_country text, p_state text, p_phone text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'lms', 'pg_temp'
AS $function$
declare c text := lower(trim(coalesce(p_country, ''))); s text := lower(trim(coalesce(p_state, '')));
  ph text := regexp_replace(coalesce(p_phone, ''), '[^0-9+]', '', 'g'); z text;
begin
  if c in ('united states','united states of america','usa','us','u.s.a.','america') then
    if s ~ '(california|washington|oregon|nevada|^ca$|^wa$|^or$|^nv$)' then return 'America/Los_Angeles'; end if;
    if s ~ '(arizona|^az$)' then return 'America/Phoenix'; end if;
    if s ~ '(colorado|utah|new mexico|montana|wyoming|idaho|^co$|^ut$|^nm$)' then return 'America/Denver'; end if;
    if s ~ '(texas|illinois|minnesota|wisconsin|missouri|louisiana|alabama|oklahoma|kansas|iowa|tennessee|mississippi|arkansas|nebraska|^tx$|^il$|^mn$)' then return 'America/Chicago'; end if;
    if s ~ '(hawaii|^hi$)' then return 'Pacific/Honolulu'; end if;
-- ... truncated: showing first 15 of 203 lines
```

#### lms.can_view_channel(_user_id uuid, _channel_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.can_view_channel(_user_id uuid, _channel_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  select lms.is_channel_member(_user_id, _channel_id) or lms.has_role(_user_id, 'admin');
$function$

```

#### lms.claim_login_rollout_item()

```sql
CREATE OR REPLACE FUNCTION lms.claim_login_rollout_item()
 RETURNS SETOF lms.login_rollout_queue
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
declare s lms.login_rollout_settings; v_id uuid; v_burst boolean;
begin
  perform pg_advisory_xact_lock(hashtext('lms.login_rollout'));
  select * into s from lms.login_rollout_settings where id;
  if s is null or not s.enabled then return; end if;
  if s.cooldown_until is not null and s.cooldown_until > now() then return; end if;
  update lms.login_rollout_queue set status = 'failed', detail = 'interrupted_while_sending', processed_at = now()
    where status = 'sending' and claimed_at < now() - interval '10 minutes';
  if exists (select 1 from lms.login_rollout_queue where status = 'sending') then return; end if;
  v_burst := s.burst_remaining > 0;
  if v_burst then
    if s.last_sent_at is not null and s.last_sent_at > now() - make_interval(secs => greatest(s.burst_gap_seconds - 1, 1)) then return; end if;
  else
    if s.last_sent_at is not null and s.last_sent_at > now() - make_interval(secs => s.gap_seconds - 20) then return; end if;
    if (select count(*) from lms.login_rollout_queue where email_sent and not burst and processed_at > now() - make_interval(hours => s.window_hours)) >= s.per_window then return; end if;
    if (select count(*) from lms.login_rollout_queue where email_sent and not burst and processed_at > now() - interval '24 hours') >= s.daily_cap then return; end if;
  end if;
  select id into v_id from lms.login_rollout_queue
    where status = 'queued' and (retry_after is null or retry_after <= now())
      and (cohort_id is null or cohort_id <> 'bd988fa4-47a6-4c69-907e-9169a5e5fa98'::uuid)
    order by priority, cohort_start nulls last, queued_at, id limit 1 for update skip locked;
  if v_id is null then return; end if;
  if v_burst then
    update lms.login_rollout_settings set burst_remaining = burst_remaining - 1 where id;
  end if;
  return query update lms.login_rollout_queue set status = 'sending', claimed_at = now(), burst = v_burst where id = v_id returning *;
end $function$

```

#### lms.claim_whatsapp_reminders(p_session_id uuid, p_offset_kind text, p_claim_token uuid, p_items jsonb)

```sql
CREATE OR REPLACE FUNCTION lms.claim_whatsapp_reminders(p_session_id uuid, p_offset_kind text, p_claim_token uuid, p_items jsonb)
 RETURNS TABLE(student_user_id uuid, channel text)
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'pg_temp'
AS $function$
begin
 if p_claim_token is null or jsonb_typeof(p_items) <> 'array' or jsonb_array_length(p_items)>200 or p_offset_kind not in ('24h','6h','60m') then raise exception 'invalid reminder claim'; end if;
 return query
 with candidates as (
 select distinct x.student_user_id,x.channel from jsonb_to_recordset(p_items) as x(student_user_id uuid,channel text)
 where x.channel in ('whatsapp','whatsapp_parent') and x.student_user_id is not null
 and not exists(select 1 from lms.session_reminder_log l where l.session_id=p_session_id and l.student_user_id=x.student_user_id and l.offset_kind=p_offset_kind and l.channel=x.channel)
 ), inserted as (
 insert into lms.whatsapp_reminder_dispatch as d(session_id,student_user_id,offset_kind,channel,claim_token,state)
 select p_session_id,c.student_user_id,p_offset_kind,c.channel,p_claim_token,'submitting' from candidates c
 on conflict do nothing returning d.student_user_id,d.channel
 ) select i.student_user_id,i.channel from inserted i;
end $function$

```

#### lms.clear_test_seat(p_user uuid, p_cohort uuid)

```sql
CREATE OR REPLACE FUNCTION lms.clear_test_seat(p_user uuid, p_cohort uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare r jsonb;
begin
  r := lms.purge_test_cohort_data(p_user, p_cohort);
  delete from lms.enrollments where student_user_id = p_user and cohort_id = p_cohort and access_status <> 'active';
  return r;
end $function$

```

#### lms.close_account_if_unenrolled(p_user_id uuid, p_reason text)

```sql
CREATE OR REPLACE FUNCTION lms.close_account_if_unenrolled(p_user_id uuid, p_reason text DEFAULT 'refund'::text)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_count int;
begin
  update lms.student_profiles
     set account_status = 'closed',
         account_closed_at = now(),
         account_closed_reason = p_reason,
         account_closed_by = null
   where user_id = p_user_id
     and account_status = 'active'
     and not exists (
       select 1 from lms.enrollments e
        where e.student_user_id = p_user_id and e.access_status = 'active'
     );
  get diagnostics v_count = row_count;
  return v_count > 0;
end
$function$

```

#### lms.community_members(_channel_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.community_members(_channel_id uuid)
 RETURNS TABLE(user_id uuid, name text, role text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  with ch as (select id, cohort_id from lms.community_channels where id = _channel_id
              and lms.can_view_channel(auth.uid(), _channel_id))
  select ma.user_id, m.full_name, 'mentor'::text from ch
    join lms.mentor_accounts ma on ch.cohort_id in (select lms._mentor_cohorts_for_user(ma.user_id))
    join public.all_mentors m on m.id = ma.mentor_id
  union
  select a.user_id, coalesce(nullif(trim(a.full_name), ''), 'Team'), 'admin' from ch, lms.admin_profiles a
    join lms.user_roles r on r.user_id = a.user_id and r.role = 'admin'
  union
  select e.student_user_id, coalesce(nullif(trim(p.full_name), ''), 'Student'), 'student' from ch
    join lms.enrollments e on e.cohort_id = ch.cohort_id and e.access_status = 'active'
    left join lms.student_profiles p on p.user_id = e.student_user_id
$function$

```

#### lms.community_messages_moderate()

```sql
CREATE OR REPLACE FUNCTION lms.community_messages_moderate()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare v_rule text;
begin
  if tg_op = 'UPDATE' then
    if new.sender_user_id is distinct from old.sender_user_id or new.channel_id is distinct from old.channel_id
       or new.sender_role is distinct from old.sender_role or new.sender_name is distinct from old.sender_name then
      raise exception 'Message identity cannot be changed';
    end if;
    if new.body is not distinct from old.body then return new; end if;
  end if;
  if lms.has_role(new.sender_user_id, 'admin') then return new; end if;
  v_rule := lms.screen_community_message(new.body);
  if v_rule is not null then raise exception 'MODERATION_BLOCKED:%', v_rule using errcode = 'P0001'; end if;
  return new;
end $function$

```

#### lms.cooldown_login_rollout(p_minutes integer)

```sql
CREATE OR REPLACE FUNCTION lms.cooldown_login_rollout(p_minutes integer)
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$ update lms.login_rollout_settings set cooldown_until = now() + make_interval(mins => p_minutes) where id; $function$

```

#### lms.defer_login_rollout_item(p_id uuid, p_detail text, p_ctx jsonb, p_minutes integer, p_count_attempt boolean)

```sql
CREATE OR REPLACE FUNCTION lms.defer_login_rollout_item(p_id uuid, p_detail text, p_ctx jsonb, p_minutes integer, p_count_attempt boolean)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
begin
  update lms.login_rollout_queue
    set status = 'queued',
        attempts = attempts + case when p_count_attempt then 1 else 0 end,
        retry_after = now() + make_interval(mins => p_minutes),
        retry_ctx = coalesce(p_ctx, retry_ctx),
        detail = p_detail, claimed_at = null
    where id = p_id and status = 'sending';
  update lms.login_rollout_settings set cooldown_until = now() + make_interval(mins => p_minutes) where id;
end $function$

```

#### lms.ensure_community_channel()

```sql
CREATE OR REPLACE FUNCTION lms.ensure_community_channel()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
begin
  if new.cohort_id is not null then
    insert into lms.community_channels (cohort_id)
    values (new.cohort_id) on conflict (cohort_id) do nothing;
  end if;
  return new;
end $function$

```

#### lms.feedback_reward_offer()

```sql
CREATE OR REPLACE FUNCTION lms.feedback_reward_offer()
 RETURNS integer
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  select case when auth.uid() is not null and coalesce((select vault_enabled from lms.vault_settings where id), false) then 5 else 0 end;
$function$

```

#### lms.finalize_join_attendance()

```sql
CREATE OR REPLACE FUNCTION lms.finalize_join_attendance()
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare n int := 0; r record; v_notif uuid;
begin
  create temp table if not exists _jc_new (session_id uuid, cohort_id uuid, student_user_id uuid, title text) on commit drop;
  truncate _jc_new;

  with due_click as (
    update lms.session_join_clicks c set finalized_at = now()
    from lms.sessions s
    where s.id = c.session_id and c.finalized_at is null
-- ... truncated: showing first 15 of 60 lines
```

#### lms.finish_login_rollout_item(p_id uuid, p_status text, p_result text, p_detail text, p_email_sent boolean)

```sql
CREATE OR REPLACE FUNCTION lms.finish_login_rollout_item(p_id uuid, p_status text, p_result text, p_detail text, p_email_sent boolean)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
declare v_claimed timestamptz;
begin
  update lms.login_rollout_queue set status = p_status, result = p_result, detail = p_detail,
    email_sent = p_email_sent, processed_at = now()
    where id = p_id and status = 'sending' returning claimed_at into v_claimed;
  if p_email_sent and v_claimed is not null then
    update lms.login_rollout_settings set last_sent_at = v_claimed where id;
  end if;
end $function$

```

#### lms.finish_whatsapp_reminders(p_session_id uuid, p_offset_kind text, p_claim_token uuid, p_items jsonb, p_broadcast_ids text[])

```sql
CREATE OR REPLACE FUNCTION lms.finish_whatsapp_reminders(p_session_id uuid, p_offset_kind text, p_claim_token uuid, p_items jsonb, p_broadcast_ids text[])
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'pg_temp'
AS $function$
declare n integer;
begin
 if jsonb_typeof(p_items) <> 'array' or jsonb_array_length(p_items)>200 then raise exception 'invalid reminder outcomes'; end if;
 with changed as (
 update lms.whatsapp_reminder_dispatch d set state=x.state,broadcast_ids=coalesce(p_broadcast_ids,'{}'),last_error=left(x.error,300),updated_at=now()
 from jsonb_to_recordset(p_items) as x(student_user_id uuid,channel text,state text,error text)
 where d.session_id=p_session_id and d.offset_kind=p_offset_kind and d.claim_token=p_claim_token
 and d.student_user_id=x.student_user_id and d.channel=x.channel
 and d.state in ('submitting','accepted') and x.state in ('accepted','rejected','uncertain')
 and (d.state <> 'accepted' or x.state='accepted')
 returning d.*
 ), logged as (
 insert into lms.session_reminder_log(session_id,student_user_id,offset_kind,channel)
 select c.session_id,c.student_user_id,c.offset_kind,c.channel from changed c where c.state='accepted'
 on conflict(session_id,student_user_id,offset_kind,channel) do nothing returning id
 ) select count(*) into n from changed;
 return n;
end $function$

```

#### lms.get_masterclass_registration_counts()

```sql
CREATE OR REPLACE FUNCTION lms.get_masterclass_registration_counts()
 RETURNS TABLE(masterclass_id uuid, registered_count integer)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
    select r.masterclass_id, count(distinct r.email)::integer as registered_count
    from lms._raw_masterclass_regs() r
    where lms.has_role(auth.uid(), 'admin') and r.masterclass_id is not null
    group by r.masterclass_id;
$function$

```

#### lms.get_masterclass_registrations()

```sql
CREATE OR REPLACE FUNCTION lms.get_masterclass_registrations()
 RETURNS TABLE(platform_key text, email text, student_name text, registered_at timestamp with time zone, session_label text, masterclass_id uuid)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  with adm as materialized (select lms.has_role(auth.uid(), 'admin') as is_admin),
  ok as materialized (
    select not exists (
      select 1 from lms.student_profiles sp
      where sp.user_id = auth.uid() and coalesce(sp.email_verified, true) = false
    ) as verified
  )
  select r.platform_key, r.email, r.student_name, r.registered_at, r.session_label, r.masterclass_id
  from lms._raw_masterclass_regs() r, adm, ok
  where (ok.verified and r.email = lower(coalesce(auth.jwt() ->> 'email', '')))
     or adm.is_admin;
$function$

```

#### lms.grade_reminder(p_year integer)

```sql
CREATE OR REPLACE FUNCTION lms.grade_reminder(p_year integer)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
declare v_notif uuid; n int;
begin
  perform pg_advisory_xact_lock(hashtext('lms.grade_yearly'));
  if exists (select 1 from lms.grade_yearly_runs where kind = 'reminder' and year = p_year) then return null; end if;
  insert into lms.notifications (type, title, body, link_url)
    values ('grade_reminder', 'Confirm Your Grade For ' || p_year,
            'A new year has started. Please check your grade in My Profile so your programs and updates stay right for you. We''ll move you up a grade automatically on 1 April.',
            '/student/settings')
    returning id into v_notif;
  insert into lms.notification_recipients (notification_id, student_user_id)
    select v_notif, p.user_id from lms.student_profiles p
    where p.account_status = 'active'
      and exists (select 1 from lms.user_roles r where r.user_id = p.user_id and r.role = 'student');
  get diagnostics n = row_count;
  insert into lms.grade_yearly_runs (kind, year, affected) values ('reminder', p_year, n);
  return n;
end $function$

```

#### lms.has_role(_user_id uuid, _role text)

```sql
CREATE OR REPLACE FUNCTION lms.has_role(_user_id uuid, _role text)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  select (
    _user_id = auth.uid()
    or (auth.jwt() ->> 'role') = 'service_role'
    or exists (select 1 from lms.user_roles where user_id = auth.uid() and role = 'admin')
  ) and exists (select 1 from lms.user_roles where user_id = _user_id and role = _role);
$function$

```

#### lms.is_channel_member(_user_id uuid, _channel_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.is_channel_member(_user_id uuid, _channel_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select (
    _user_id = auth.uid()
    or (auth.jwt() ->> 'role') = 'service_role'
    or exists (select 1 from lms.user_roles where user_id = auth.uid() and role = 'admin')
  ) and (
    exists (select 1 from lms.community_channels c join lms.enrollments e on e.cohort_id = c.cohort_id
            where c.id = _channel_id and e.student_user_id = _user_id and e.access_status = 'active')
    or exists (select 1 from lms.community_channels c where c.id = _channel_id
               and c.cohort_id in (select lms._mentor_cohorts_for_user(_user_id)))
  )
$function$

```

#### lms.is_cohort_mentor(_cohort_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.is_cohort_mentor(_cohort_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select auth.uid() is not null and _cohort_id in (select lms._mentor_cohorts_for_user(auth.uid()))
$function$

```

#### lms.is_cohort_student(_cohort_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.is_cohort_student(_cohort_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select auth.uid() is not null and exists (
    select 1 from lms.enrollments e
    where e.cohort_id = _cohort_id and e.student_user_id = auth.uid()
  )
$function$

```

#### lms.is_my_catalog_mentor(_mentor_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.is_my_catalog_mentor(_mentor_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select auth.uid() is not null and _mentor_id is not null and exists (
    select 1 from lms.mentor_accounts a where a.mentor_id = _mentor_id and a.user_id = auth.uid()
  )
$function$

```

#### lms.issue_certificates(p_cohort_id uuid, p_kind text, p_issue_date date, p_items jsonb)

```sql
CREATE OR REPLACE FUNCTION lms.issue_certificates(p_cohort_id uuid, p_kind text, p_issue_date date, p_items jsonb)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_program uuid; v_n int := 0; v_item jsonb; v_uid uuid; v_name text; v_notif uuid;
  v_ids uuid[] := '{}';
begin
  if not lms.has_role(auth.uid(), 'admin') then raise exception 'not_admin' using errcode = '42501'; end if;
  if p_kind not in ('participation','winner') then raise exception 'invalid_kind'; end if;
  if p_issue_date is null then raise exception 'issue_date_required'; end if;
  select program_id into v_program from public.all_cohorts where id = p_cohort_id;
  if v_program is null then raise exception 'cohort_not_found'; end if;
  if not exists (select 1 from lms.certificate_templates where program_id = v_program and kind = p_kind) then
-- ... truncated: showing first 15 of 45 lines
```

#### lms.mark_birthday_popup_seen()

```sql
CREATE OR REPLACE FUNCTION lms.mark_birthday_popup_seen()
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  update lms.birthday_grants set popup_seen_at = now()
  where student_user_id = auth.uid() and popup_seen_at is null and expires_at > now();
$function$

```

#### lms.mark_certificates_seen(p_ids uuid[])

```sql
CREATE OR REPLACE FUNCTION lms.mark_certificates_seen(p_ids uuid[])
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  update lms.certificates set seen_at = now()
  where id = any(p_ids) and student_user_id = auth.uid() and seen_at is null
$function$

```

#### lms.mark_referral_promo_seen()

```sql
CREATE OR REPLACE FUNCTION lms.mark_referral_promo_seen()
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  update lms.student_profiles
  set referral_promo_seen_at = now()
  where user_id = auth.uid() and referral_promo_seen_at is null
$function$

```

#### lms.mc_norm(t text)

```sql
CREATE OR REPLACE FUNCTION lms.mc_norm(t text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'lms', 'public'
AS $function$ select regexp_replace(lower(coalesce(t, '')), '[^a-z0-9]', '', 'g') $function$

```

#### lms.mentor_cohort_students(_cohort_id uuid, _exclude_emails text[])

```sql
CREATE OR REPLACE FUNCTION lms.mentor_cohort_students(_cohort_id uuid, _exclude_emails text[] DEFAULT '{}'::text[])
 RETURNS TABLE(user_id uuid, full_name text, grade text, school_name text, group_name text, present_count integer, session_count integer)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select e.student_user_id, p.full_name, p.grade, p.school_name,
         (select g.name from lms.cohort_group_members gm join lms.cohort_groups g on g.id = gm.group_id
           where gm.student_user_id = e.student_user_id and gm.cohort_id = _cohort_id limit 1),
         (select count(*)::int from lms.session_attendance a where a.student_user_id = e.student_user_id and a.cohort_id = _cohort_id and a.status = 'present'),
         (select count(distinct a.session_id)::int from lms.session_attendance a where a.cohort_id = _cohort_id)
  from lms.enrollments e
  left join lms.student_profiles p on p.user_id = e.student_user_id
  where e.cohort_id = _cohort_id and e.access_status = 'active'
    and lower(coalesce(p.email, '')) <> all (coalesce(_exclude_emails, '{}'))
    and (lms.is_cohort_mentor(_cohort_id) or lms.has_role(auth.uid(), 'admin'))
  order by p.full_name nulls last
$function$

```

#### lms.my_birthday_offer()

```sql
CREATE OR REPLACE FUNCTION lms.my_birthday_offer()
 RETURNS TABLE(grant_id uuid, amount numeric, remaining numeric, expires_at timestamp with time zone, popup_seen boolean, zone text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  select g.id, g.amount, coalesce(l.remaining, 0), g.expires_at, g.popup_seen_at is not null, g.zone
  from lms.birthday_grants g left join lms.vault_lots l on l.id = g.lot_id
  where g.student_user_id = auth.uid() and g.expires_at > now()
  order by g.created_at desc limit 1;
$function$

```

#### lms.my_certificates()

```sql
CREATE OR REPLACE FUNCTION lms.my_certificates()
 RETURNS TABLE(id uuid, cohort_id uuid, program_id uuid, kind text, issue_date date, tag_values jsonb, issued_at timestamp with time zone, seen_at timestamp with time zone, html text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  select c.id, c.cohort_id, c.program_id, c.kind, c.issue_date, c.tag_values, c.issued_at, c.seen_at, t.html
  from lms.certificates c
  join lms.certificate_templates t on t.program_id = c.program_id and t.kind = c.kind
  where c.student_user_id = auth.uid() and c.revoked_at is null
    and exists (select 1 from lms.enrollments e where e.cohort_id = c.cohort_id
                and e.student_user_id = auth.uid() and e.access_status = 'active')
  order by c.issued_at desc
$function$

```

#### lms.my_community_summary()

```sql
CREATE OR REPLACE FUNCTION lms.my_community_summary()
 RETURNS TABLE(channel_id uuid, cohort_id uuid, unread_count integer, last_message_at timestamp with time zone, last_message_body text, last_message_sender text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  with me as (select auth.uid() as uid, lms.has_role(auth.uid(), 'admin') as is_admin),
  vis as (
    select c.id, c.cohort_id from lms.community_channels c, me
    where me.uid is not null and (
      me.is_admin
      or c.cohort_id in (select e.cohort_id from lms.enrollments e where e.student_user_id = me.uid and e.access_status = 'active')
      or c.cohort_id in (select lms._mentor_cohorts_for_user(me.uid))
    )
  )
  select v.id, v.cohort_id,
    (select count(*)::int from lms.community_messages m
      where m.channel_id = v.id and m.deleted_at is null
        and m.sender_user_id <> (select uid from me)
        and m.created_at > coalesce(r.last_read_at, 'epoch'::timestamptz)),
    lm.created_at, lm.body, lm.sender_name
  from vis v
  left join lms.community_reads r on r.channel_id = v.id and r.user_id = (select uid from me)
  left join lateral (
    select body, created_at, sender_name from lms.community_messages m
    where m.channel_id = v.id and m.deleted_at is null
    order by m.created_at desc limit 1
  ) lm on true
$function$

```

#### lms.my_hosted_masterclasses()

```sql
CREATE OR REPLACE FUNCTION lms.my_hosted_masterclasses()
 RETURNS TABLE(id uuid, title text, series text, session_number integer, audience text, scheduled_at timestamp with time zone, duration_minutes integer, display_timezone text, join_url text, passcode text, apply_url text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  select m.id, m.title, m.series, m.session_number::int, m.audience,
         m.scheduled_at, m.duration_minutes::int, m.display_timezone,
         m.join_url, m.passcode, m.apply_url
  from public.all_masterclasses m
  join lms.mentor_accounts ma on ma.mentor_id = m.host_mentor_id
  where ma.user_id = auth.uid()
    and coalesce(m.is_published, true)
  order by m.scheduled_at;
$function$

```

#### lms.my_marked_sessions()

```sql
CREATE OR REPLACE FUNCTION lms.my_marked_sessions()
 RETURNS TABLE(session_id uuid, cohort_id uuid, session_datetime timestamp with time zone)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select distinct a.session_id, a.cohort_id, s.session_datetime
  from lms.session_attendance a
  join lms.sessions s on s.id = a.session_id
  join lms.enrollments e on e.cohort_id = a.cohort_id and e.student_user_id = auth.uid()
  where auth.uid() is not null
    and (s.pathway_id is null or e.pathway_id is null or s.pathway_id = e.pathway_id)
    and (s.section_id is null or exists (
      select 1 from lms.cohort_section_members sm
      where sm.section_id = s.section_id and sm.student_user_id = auth.uid()))
$function$

```

#### lms.my_mentor_cohorts()

```sql
CREATE OR REPLACE FUNCTION lms.my_mentor_cohorts()
 RETURNS SETOF uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select lms._mentor_cohorts_for_user(auth.uid())
$function$

```

#### lms.my_mentor_feedback()

```sql
CREATE OR REPLACE FUNCTION lms.my_mentor_feedback()
 RETURNS TABLE(session_id uuid, cohort_id uuid, title text, session_datetime timestamp with time zone, responses integer, avg_session numeric, avg_mentor numeric, avg_engagement numeric, comments text[])
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select s.id, s.cohort_id, s.title, s.session_datetime, count(f.id)::int,
         round(avg(f.session_quality_rating)::numeric, 2), round(avg(f.mentor_rating)::numeric, 2),
         round(avg(f.engagement_rating)::numeric, 2),
         (array_agg(f.takeaway order by f.submitted_at desc) filter (where nullif(trim(f.takeaway), '') is not null))[1:8]
  from lms.session_feedback f join lms.sessions s on s.id = f.session_id
  where lms.my_mentor_id() is not null
    and (f.mentor_id = lms.my_mentor_id() or (f.mentor_id is null and lms.is_cohort_mentor(s.cohort_id)))
  group by s.id order by s.session_datetime desc
$function$

```

#### lms.my_mentor_feedback_entries()

```sql
CREATE OR REPLACE FUNCTION lms.my_mentor_feedback_entries()
 RETURNS TABLE(id uuid, session_id uuid, cohort_id uuid, session_title text, session_datetime timestamp with time zone, session_quality_rating integer, mentor_rating integer, engagement_rating integer, takeaway text, submitted_at timestamp with time zone)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select f.id, s.id, s.cohort_id, s.title, s.session_datetime,
         f.session_quality_rating::int, f.mentor_rating::int, f.engagement_rating::int,
         nullif(trim(f.takeaway), ''), coalesce(f.submitted_at, f.created_at)
  from lms.session_feedback f join lms.sessions s on s.id = f.session_id
  where lms.my_mentor_id() is not null
    and (f.mentor_id = lms.my_mentor_id() or (f.mentor_id is null and lms.is_cohort_mentor(s.cohort_id)))
  order by s.session_datetime desc, coalesce(f.submitted_at, f.created_at) desc
$function$

```

#### lms.my_mentor_id()

```sql
CREATE OR REPLACE FUNCTION lms.my_mentor_id()
 RETURNS uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select mentor_id from lms.mentor_accounts where user_id = auth.uid()
$function$

```

#### lms.my_mentor_profile()

```sql
CREATE OR REPLACE FUNCTION lms.my_mentor_profile()
 RETURNS TABLE(mentor_id uuid, full_name text, university text, country text, bio text, image_url text, email text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  select m.id, m.full_name, m.university, m.country, m.bio, m.image_url, u.email::text
  from lms.mentor_accounts ma
  join public.all_mentors m on m.id = ma.mentor_id
  join auth.users u on u.id = ma.user_id
  where ma.user_id = auth.uid()
$function$

```

#### lms.my_referral_offer()

```sql
CREATE OR REPLACE FUNCTION lms.my_referral_offer()
 RETURNS jsonb
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  select case when exists (select 1 from lms.student_profiles where user_id = auth.uid()) then
    jsonb_build_object('invite_reward_enabled', coalesce(r.invite_reward_enabled and s.vault_enabled, false),
      'invite_reward_usd', 5, 'program_referral_enabled', coalesce(r.program_referral_enabled and s.vault_enabled, false),
      'release_days', s.referral_release_days)
    else '{}'::jsonb end
  from lms.referral_settings r cross join lms.vault_settings s where r.id and s.id;
$function$

```

#### lms.my_referral_promo_seen()

```sql
CREATE OR REPLACE FUNCTION lms.my_referral_promo_seen()
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
  select coalesce(
    (select p.referral_promo_seen_at is not null from lms.student_profiles p where p.user_id = auth.uid()),
    true
  )
$function$

```

#### lms.normalize_session_mode()

```sql
CREATE OR REPLACE FUNCTION lms.normalize_session_mode()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$ begin if new.session_mode = 'in_person' then new.session_link := null; new.recording_url := null; if new.location is null or btrim(new.location) = '' then raise exception 'An in-person session needs a location'; end if; else new.location := null; end if; return new; end $function$

```

#### lms.post_due_community_session_reminders(p_dry_run boolean)

```sql
CREATE OR REPLACE FUNCTION lms.post_due_community_session_reminders(p_dry_run boolean DEFAULT false)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare r record; team uuid; mid uuid; msg text; safe_link text; anchor text; total integer:=0; candidates integer:=0;
begin
  if coalesce(auth.role(),'') <> 'service_role' then raise exception 'Service role required'; end if;
  perform pg_advisory_xact_lock(hashtext('lms.community-session-reminders'));
  select user_id into team from lms.user_roles where role='admin' order by user_id limit 1;
  if team is null then raise exception 'Team admin unavailable'; end if;
  perform set_config('lms.community_reminder_team','on',true);
  for r in
    select s.*,c.id channel_id,cat.cohort_label,p.program_name,
-- ... truncated: showing first 15 of 60 lines
```

#### lms.promote_grades(p_year integer)

```sql
CREATE OR REPLACE FUNCTION lms.promote_grades(p_year integer)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
declare v_notif uuid; n int;
begin
  perform pg_advisory_xact_lock(hashtext('lms.grade_yearly'));
  if exists (select 1 from lms.grade_yearly_runs where kind = 'promotion' and year = p_year) then return null; end if;
  create temporary table _promoted on commit drop as
    with up as (
      update lms.student_profiles p set
        grade = case when p.grade = '12' then 'gap_university' else (p.grade::int + 1)::text end,
        grade_promoted_year = p_year
      where p.account_status = 'active'
        and p.grade is not null and p.grade <> 'gap_university'
        and coalesce(p.grade_promoted_year, 0) < p_year
        and (p.grade_updated_at is null or p.grade_updated_at < make_timestamptz(p_year, 1, 1, 0, 0, 0, 'UTC'))
      returning p.user_id
    ) select user_id from up;
  select count(*) into n from _promoted;
  if n > 0 then
    insert into lms.notifications (type, title, body, link_url)
      values ('grade_promoted', 'You''ve Moved Up A Grade',
              'Happy new school year! We''ve moved you up to the next grade. If that isn''t right, update it in My Profile.',
              '/student/settings')
      returning id into v_notif;
    insert into lms.notification_recipients (notification_id, student_user_id) select v_notif, user_id from _promoted;
  end if;
  insert into lms.grade_yearly_runs (kind, year, affected) values ('promotion', p_year, n);
  return n;
end $function$

```

#### lms.purge_test_cohort_data(p_user uuid, p_cohort uuid)

```sql
CREATE OR REPLACE FUNCTION lms.purge_test_cohort_data(p_user uuid, p_cohort uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare n int; total int := 0;
begin
  if p_user not in ('fa9a0cd4-2f3e-48ae-b8e2-a536bb5a0f4b','afd5a6fc-8b84-4ad2-9d5c-10bc16d2296c','d63c9632-1a2a-453e-97c8-ee82bf983d5f','20f568fb-21d6-438f-b922-03b40c248799') then
    raise exception 'not_a_test_account';
  end if;
  if not exists (select 1 from lms.enrollments where student_user_id = p_user and cohort_id = p_cohort and access_status <> 'active') then
    raise exception 'seat_not_closed';
  end if;
  delete from lms.session_attendance where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.session_feedback where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.session_submissions where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.feedback_email_tokens where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.program_feedback where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.mentor_feedback where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.certificates where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.cohort_section_members where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.cohort_group_members where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.support_tickets where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.referral_program_links where student_user_id = p_user and cohort_id = p_cohort; get diagnostics n = row_count; total := total + n;
  delete from lms.feedback_nudge_log where student_user_id = p_user and session_id in (select id from lms.sessions where cohort_id = p_cohort); get diagnostics n = row_count; total := total + n;
  delete from lms.session_join_clicks where student_user_id = p_user and session_id in (select id from lms.sessions where cohort_id = p_cohort); get diagnostics n = row_count; total := total + n;
  delete from lms.session_recording_watch where student_user_id = p_user and session_id in (select id from lms.sessions where cohort_id = p_cohort); get diagnostics n = row_count; total := total + n;
  delete from lms.notification_recipients where student_user_id = p_user and notification_id in (select id from lms.notifications where cohort_id = p_cohort); get diagnostics n = row_count; total := total + n;
  delete from lms.community_mentions where mentioned_user_id = p_user and channel_id in (select id from lms.community_channels where cohort_id = p_cohort); get diagnostics n = row_count; total := total + n;
  delete from lms.community_reads where user_id = p_user and channel_id in (select id from lms.community_channels where cohort_id = p_cohort); get diagnostics n = row_count; total := total + n;
  delete from lms.community_messages m where m.sender_user_id = p_user and m.channel_id in (select id from lms.community_channels where cohort_id = p_cohort)
    and not exists (select 1 from lms.community_session_reminder_log r where r.message_id = m.id); get diagnostics n = row_count; total := total + n;
  return jsonb_build_object('deleted', total);
end $function$

```

#### lms.record_community_mentions()

```sql
CREATE OR REPLACE FUNCTION lms.record_community_mentions()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare _uid uuid; _clean text; _everyone boolean;
begin
  if new.deleted_at is not null then return new; end if;
  if not lms._channel_member_raw(new.sender_user_id,new.channel_id) then return new; end if;
  _everyone := position('@[Everyone](everyone)' in new.body) > 0;
  _clean := replace(new.body, '@[Everyone](everyone)', '@Everyone');
  _clean := regexp_replace(_clean, '@\[([^\]]{1,80})\]\(([0-9a-fA-F-]{36})\)', '@\1', 'g');
  for _uid in
    select distinct candidates.uid from (
      select (m[2])::uuid as uid from regexp_matches(new.body,
        '@\[([^\]]{1,80})\]\(([0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12})\)', 'g') m
      union
      select e.student_user_id from lms.enrollments e join lms.community_channels c on c.cohort_id=e.cohort_id
        where _everyone and c.id=new.channel_id and e.access_status='active'
      union
      select ma.user_id from lms.mentor_accounts ma where _everyone
      union
      select r.user_id from lms.user_roles r where _everyone and r.role='admin'
    ) candidates where candidates.uid <> new.sender_user_id
      and lms._channel_member_raw(candidates.uid,new.channel_id)
  loop
    insert into lms.community_mentions(message_id,channel_id,mentioned_user_id,sender_user_id,sender_name,excerpt)
      values(new.id,new.channel_id,_uid,new.sender_user_id,new.sender_name,left(_clean,160))
      on conflict do nothing;
  end loop;
  return new;
end $function$

```

#### lms.record_recording_watch(p_session_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.record_recording_watch(p_session_id uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare
  v_uid uuid := auth.uid(); s lms.sessions%rowtype; w lms.session_recording_watch%rowtype;
  v_required int := 1800; v_gap numeric;
begin
  if v_uid is null then return jsonb_build_object('ok', false); end if;
  select * into s from lms.sessions where id = p_session_id;
  if not found or now() < s.session_datetime then return jsonb_build_object('ok', false); end if;
  if not exists (
    select 1 from lms.enrollments e
-- ... truncated: showing first 15 of 55 lines
```

#### lms.record_session_join(p_session_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.record_session_join(p_session_id uuid)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare v_uid uuid := auth.uid(); s lms.sessions%rowtype;
begin
  if v_uid is null then return false; end if;
  select * into s from lms.sessions where id = p_session_id;
  if not found or coalesce(s.session_mode, 'online') <> 'online'
     or coalesce(btrim(s.session_link), '') = '' and s.section_id is null and not exists (
       select 1 from lms.session_section_links l where l.session_id = s.id) then
    return false;
  end if;
  if now() < s.session_datetime - interval '60 minutes'
     or now() > s.session_datetime + interval '60 minutes' then
    return false;
  end if;
  if not exists (
    select 1 from lms.enrollments e
    where e.student_user_id = v_uid and e.cohort_id = s.cohort_id and e.access_status = 'active'
      and (s.pathway_id is null or s.pathway_id = e.pathway_id)
  ) then return false; end if;
  if s.section_id is not null and not exists (
    select 1 from lms.cohort_section_members sm
    where sm.section_id = s.section_id and sm.student_user_id = v_uid
  ) then return false; end if;

  insert into lms.session_join_clicks (session_id, student_user_id)
  values (s.id, v_uid)
  on conflict (session_id, student_user_id) do update
    set last_clicked_at = now(), click_count = lms.session_join_clicks.click_count + 1;
  return true;
end $function$

```

#### lms.referral_begin_verification(p_invite uuid, p_hash text)

```sql
CREATE OR REPLACE FUNCTION lms.referral_begin_verification(p_invite uuid, p_hash text)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.referral_verifications;
begin
  perform 1 from lms.referral_invites where id=p_invite and status='invited' and expires_at>now() for update;
  if not found then return jsonb_build_object('ok',false,'error','invalid'); end if;
  select * into v from lms.referral_verifications where invite_id=p_invite for update;
  if found and (v.sent_at>now()-interval '3 minutes' or v.send_count>=3) then
    return jsonb_build_object('ok',false,'error','cooldown'); end if;
  insert into lms.referral_verifications(invite_id,code_hash,expires_at) values(p_invite,p_hash,now()+interval '10 minutes')
    on conflict(invite_id) do update set code_hash=excluded.code_hash,expires_at=excluded.expires_at,
      sent_at=now(),verified_at=null,send_count=referral_verifications.send_count+1;
  return jsonb_build_object('ok',true);
end $function$

```

#### lms.referral_check_verification(p_invite uuid, p_hash text)

```sql
CREATE OR REPLACE FUNCTION lms.referral_check_verification(p_invite uuid, p_hash text)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.referral_verifications;
begin
  select * into v from lms.referral_verifications where invite_id=p_invite for update;
  if not found or v.attempts>=5 or v.expires_at<=now() then return false; end if;
  update lms.referral_verifications set attempts=attempts+1 where invite_id=p_invite;
  if v.code_hash<>p_hash then return false; end if;
  update lms.referral_verifications set verified_at=now() where invite_id=p_invite;
  return true;
end $function$

```

#### lms.referral_create_invite(p_inviter uuid, p_details jsonb, p_hash text)

```sql
CREATE OR REPLACE FUNCTION lms.referral_create_invite(p_inviter uuid, p_details jsonb, p_hash text)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare r lms.referral_settings; v lms.referral_invites; v_email text; v_identity text;
begin
  select * into r from lms.referral_settings where id;
  v_email := lower(btrim(p_details->>'friend_email'));
  v_identity := lms.referral_email_identity(v_email);
  perform pg_advisory_xact_lock(hashtext('referral-inviter:' || p_inviter));
  perform pg_advisory_xact_lock(hashtext('referral-email:' || v_identity));
  if not exists (select 1 from lms.student_profiles p join lms.user_roles u on u.user_id=p.user_id
    where p.user_id=p_inviter and u.role='student' and p.account_status='active') then
-- ... truncated: showing first 15 of 47 lines
```

#### lms.referral_email_identity(p_email text)

```sql
CREATE OR REPLACE FUNCTION lms.referral_email_identity(p_email text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'pg_catalog'
AS $function$
  select case when split_part(lower(btrim(p_email)), '@', 2) in ('gmail.com','googlemail.com')
    then replace(split_part(split_part(lower(btrim(p_email)), '@', 1), '+', 1), '.', '') || '@gmail.com'
    else lower(btrim(p_email)) end;
$function$

```

#### lms.referral_finish_join(p_invite uuid, p_friend uuid)

```sql
CREATE OR REPLACE FUNCTION lms.referral_finish_join(p_invite uuid, p_friend uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.referral_invites; r lms.referral_settings; v_identity text; v_key text; v_id uuid;
begin
  select * into v from lms.referral_invites where id=p_invite for update;
  if not found or v.status<>'joined' or (v.joined_user_id is not null and v.joined_user_id<>p_friend) then
    return jsonb_build_object('ok',false,'error','invalid'); end if;
  if not exists(select 1 from lms.referral_verifications where invite_id=p_invite and verified_at is not null and expires_at>now()) then
    return jsonb_build_object('ok',false,'error','verification_required'); end if;
  if p_friend=v.inviter_user_id or not exists(select 1 from lms.student_profiles where user_id=p_friend
    and email=v.friend_email and referral_invite_id=v.id and email_verified and registration_source='invite') then
    return jsonb_build_object('ok',false,'error','invalid_friend'); end if;
  if exists(select 1 from lms.referral_join_rewards where friend_identity=lms.referral_email_identity(v.friend_email)
    and friend_user_id<>p_friend) then return jsonb_build_object('ok',false,'error','duplicate_friend'); end if;
  update lms.referral_invites set joined_user_id=p_friend where id=p_invite;
  select * into r from lms.referral_settings where id;
  if not r.invite_reward_enabled or v.created_at<r.invite_reward_starts_at
    or not coalesce((select vault_enabled from lms.vault_settings where id),false) then
    return jsonb_build_object('ok',true,'rewarded',false); end if;
  v_identity:=lms.referral_email_identity(v.friend_email);
  perform pg_advisory_xact_lock(hashtext('referral-reward:'||v_identity));
  insert into lms.referral_join_rewards(friend_identity,friend_user_id,invite_id,inviter_user_id)
    values(v_identity,p_friend,v.id,v.inviter_user_id) on conflict do nothing returning invite_id into v_id;
  if v_id is null then return jsonb_build_object('ok',true,'rewarded',false); end if;
  v_key:='referral-join:'||v_identity;
  perform lms.vault_earn(v.inviter_user_id,'effort',5,'referral',v_key,'available',null,null,null,true,
    'Successful invitation · '||split_part(v.friend_name,' ',1));
  return jsonb_build_object('ok',true,'rewarded',true,'amount',5);
end $function$

```

#### lms.referral_followup_claim(p_invite uuid, p_hash text)

```sql
CREATE OR REPLACE FUNCTION lms.referral_followup_claim(p_invite uuid, p_hash text)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.referral_invites; r lms.referral_settings; v_identity text;
begin
  select * into v from lms.referral_invites where id = p_invite for update;
  if not found or v.status not in ('invited','expired') or v.joined_user_id is not null
     or v.followup_count >= 3 or v.last_sent_at > now() - interval '7 days' then
    return jsonb_build_object('ok', false, 'reason', 'not_due');
  end if;
  v_identity := lms.referral_email_identity(v.friend_email);
  if exists (select 1 from lms.student_profiles where lms.referral_email_identity(email) = v_identity)
     or exists (select 1 from lms.referral_invites where id <> v.id and status in ('invited','joined')
                and lms.referral_email_identity(friend_email) = v_identity) then
    return jsonb_build_object('ok', false, 'reason', 'has_account');
  end if;
  select * into r from lms.referral_settings where id;
  update lms.referral_invites set status = 'invited', token_hash = p_hash, last_sent_at = now(),
    followup_count = followup_count + 1, expires_at = now() + make_interval(days => r.invite_valid_days)
  where id = v.id;
  delete from lms.referral_verifications where invite_id = v.id;
  return jsonb_build_object('ok', true, 'followup', v.followup_count + 1, 'prev_sent_at', v.last_sent_at);
end $function$

```

#### lms.referral_followup_unclaim(p_invite uuid, p_prev_sent_at timestamp with time zone)

```sql
CREATE OR REPLACE FUNCTION lms.referral_followup_unclaim(p_invite uuid, p_prev_sent_at timestamp with time zone)
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  update lms.referral_invites set followup_count = greatest(followup_count - 1, 0), last_sent_at = p_prev_sent_at
  where id = p_invite and status = 'invited';
$function$

```

#### lms.referral_resend_invite(p_inviter uuid, p_invite uuid, p_hash text)

```sql
CREATE OR REPLACE FUNCTION lms.referral_resend_invite(p_inviter uuid, p_invite uuid, p_hash text)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.referral_invites; r lms.referral_settings; v_identity text;
begin
  perform pg_advisory_xact_lock(hashtext('referral-inviter:'||p_inviter));
  select * into v from lms.referral_invites where id=p_invite and inviter_user_id=p_inviter for update;
  if not found or v.status='joined' then return jsonb_build_object('ok',false); end if;
  v_identity:=lms.referral_email_identity(v.friend_email);
  perform pg_advisory_xact_lock(hashtext('referral-email:'||v_identity));
  select * into r from lms.referral_settings where id;
  if v.last_sent_at>now()-interval '3 minutes' or v.send_count>=r.max_sends
    or exists(select 1 from lms.referral_invites where id<>v.id and status in ('invited','joined')
      and lms.referral_email_identity(friend_email)=v_identity)
    or exists(select 1 from lms.student_profiles where lms.referral_email_identity(email)=v_identity)
    or (select count(*) from lms.referral_invites where inviter_user_id=p_inviter and last_sent_at>now()-interval '1 day')>=r.daily_invite_limit
    then return jsonb_build_object('ok',false); end if;
  update lms.referral_invites set status='invited',token_hash=p_hash,send_count=send_count+1,last_sent_at=now(),
    expires_at=now()+make_interval(days=>r.invite_valid_days) where id=v.id;
  delete from lms.referral_verifications where invite_id=v.id;
  return jsonb_build_object('ok',true);
end $function$

```

#### lms.revoke_certificate(p_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.revoke_certificate(p_id uuid)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
begin
  if not lms.has_role(auth.uid(), 'admin') then raise exception 'not_admin' using errcode = '42501'; end if;
  update lms.certificates set revoked_at = now(), revoked_by = auth.uid() where id = p_id and revoked_at is null;
end $function$

```

#### lms.revoke_user_sessions(p_user uuid)

```sql
CREATE OR REPLACE FUNCTION lms.revoke_user_sessions(p_user uuid)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'auth', 'lms'
AS $function$
declare n int;
begin
  delete from auth.sessions where user_id = p_user;
  get diagnostics n = row_count;
  delete from auth.refresh_tokens where user_id = p_user::text;
  return n;
end $function$

```

#### lms.screen_community_message(p_body text)

```sql
CREATE OR REPLACE FUNCTION lms.screen_community_message(p_body text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare
  raw text := lower(regexp_replace(coalesce(p_body, ''), '@\[([^\]]{1,80})\]\((?:[0-9a-fA-F-]{36}|everyone)\)', '@\1', 'g'));
  norm text;
  compact text;
  w text;
  abusive text[] := array[
    'fuck','fucks','fucked','fucker','fuckers','fucking','fuk','fuking','fck','fcking',
    'motherfucker','motherfuckers','mothafucka','stfu',
    'shit','shits','shitty','bullshit','bshit',
-- ... truncated: showing first 15 of 101 lines
```

#### lms.screen_contact_number(p_body text)

```sql
CREATE OR REPLACE FUNCTION lms.screen_contact_number(p_body text)
 RETURNS boolean
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare raw text; numbers text; words text[] := array['zero','oh','one','two','three','four','five','six','seven','eight','nine'];
        digits text[] := array['0','0','1','2','3','4','5','6','7','8','9']; i int;
begin
  raw := lower(normalize(coalesce(p_body,''), NFKC));
  raw := translate(raw, '٠١٢٣٤٥٦٧٨٩۰۱۲۳۴۵۶۷۸۹', '01234567890123456789');
  raw := regexp_replace(raw, U&'[\200B-\200F\202A-\202E\2060-\206F\FEFF]', '', 'g');
  numbers := raw;
  for i in 1..array_length(words,1) loop
    numbers := regexp_replace(numbers, '\m' || words[i] || '\M', digits[i], 'g');
  end loop;
  return raw ~ '^[^a-z0-9]*[0-9][^a-z]*$' or numbers ~ '([0-9][^a-z0-9]*){7,}';
end $function$

```

#### lms.stamp_community_automation_team()

```sql
CREATE OR REPLACE FUNCTION lms.stamp_community_automation_team()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$ begin if auth.uid() is null and new.sender_role='admin' and lms.has_role(new.sender_user_id,'admin') and current_setting('lms.community_reminder_team',true)='on' then new.sender_name:='Team'; end if; return new; end $function$

```

#### lms.stamp_community_sender()

```sql
CREATE OR REPLACE FUNCTION lms.stamp_community_sender()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare _name text; _is_admin boolean; _mentor uuid;
begin
  new.sender_user_id := coalesce(auth.uid(), new.sender_user_id);
  _is_admin := lms.has_role(new.sender_user_id, 'admin');
  select mentor_id into _mentor from lms.mentor_accounts where user_id = new.sender_user_id;
  if _is_admin then
    select full_name into _name from lms.admin_profiles where user_id = new.sender_user_id;
  elsif _mentor is not null then
    select full_name into _name from public.all_mentors where id = _mentor;
  end if;
  if _name is null then
    select full_name into _name from lms.student_profiles where user_id = new.sender_user_id;
  end if;
  if _name is null then
    select full_name into _name from lms.admin_profiles where user_id = new.sender_user_id;
  end if;
  new.sender_name := coalesce(nullif(trim(_name), ''), 'Member');
  new.sender_role := case when _is_admin then 'admin' when _mentor is not null then 'mentor' else 'student' end;
  new.created_at := now();
  new.deleted_at := null;
  return new;
end $function$

```

#### lms.start_reconcile_run(p_trigger text, p_mode text, p_actor uuid)

```sql
CREATE OR REPLACE FUNCTION lms.start_reconcile_run(p_trigger text, p_mode text, p_actor uuid DEFAULT NULL::uuid)
 RETURNS uuid
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms'
AS $function$
declare v_id uuid;
begin
  perform pg_advisory_xact_lock(hashtext('lms.reconcile_enrollments'));
  update lms.enrollment_sync_runs set status = 'failed', finished_at = now(), note = coalesce(note,'') || ' timed_out'
    where status = 'running' and trigger <> 'webhook' and started_at < now() - interval '20 minutes';
  if exists (select 1 from lms.enrollment_sync_runs where status = 'running' and trigger <> 'webhook') then
    return null;
  end if;
  insert into lms.enrollment_sync_runs (trigger, mode, actor_user_id) values (p_trigger, p_mode, p_actor) returning id into v_id;
  return v_id;
end $function$

```

#### lms.sync_profiles_from_applications(p_user uuid)

```sql
CREATE OR REPLACE FUNCTION lms.sync_profiles_from_applications(p_user uuid DEFAULT NULL::uuid)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare t record; parts text[] := '{}'; q text; n int;
begin
  for t in select c.table_name from information_schema.columns c
           where c.table_schema = 'public' and c.table_name like '%\_applications' and c.column_name = 'student_email'
             and exists (select 1 from information_schema.columns d where d.table_schema='public' and d.table_name=c.table_name and d.column_name='student_dob') loop
    parts := parts || format($f$select lower(trim(student_email)) e, %s::date dob, %s::text gender, %s::text country, %s::text state, %s::text city,
        %s::text phone, %s::text school, %s::text parent_name, %s::text parent_email, %s::text parent_phone, %s::timestamptz at
      from public.%I where student_email is not null$f$,
      'student_dob',
-- ... truncated: showing first 15 of 77 lines
```

#### lms.t_vault_welcome_on_profile()

```sql
CREATE OR REPLACE FUNCTION lms.t_vault_welcome_on_profile()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
begin
  perform lms.vault_grant_welcome(new.user_id);
  return new;
exception when others then
  -- never block account creation because of the Vault
  return new;
end $function$

```

#### lms.tg_all_mentors_view_write()

```sql
CREATE OR REPLACE FUNCTION lms.tg_all_mentors_view_write()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'lms'
AS $function$
declare v_id uuid;
begin
  if tg_op = 'INSERT' then
    insert into public.all_mentors (id, full_name, bio, university, image_url, is_active)
    values (coalesce(new.id, gen_random_uuid()), new.full_name, new.bio, new.university,
            new.image_url, coalesce(new.is_active, true))
    returning id into v_id;
    insert into lms.mentor_settings (mentor_id, display_order, legacy_tags, is_listed)
    values (v_id, coalesce(new.display_order, 0), coalesce(new.legacy_tags, '[]'::jsonb), coalesce(new.is_listed, true));
    new.id := v_id;
    return new;
  elsif tg_op = 'UPDATE' then
    update public.all_mentors set
      full_name = new.full_name, bio = new.bio, university = new.university,
      image_url = new.image_url, is_active = coalesce(new.is_active, true), updated_at = now()
    where id = old.id;
    insert into lms.mentor_settings (mentor_id, display_order, legacy_tags, is_listed)
    values (old.id, coalesce(new.display_order, 0), coalesce(new.legacy_tags, '[]'::jsonb), coalesce(new.is_listed, true))
    on conflict (mentor_id) do update set
      display_order = excluded.display_order,
      legacy_tags = excluded.legacy_tags,
      is_listed = excluded.is_listed;
    return new;
  else
    delete from lms.mentor_settings where mentor_id = old.id;
    delete from public.all_mentors where id = old.id;
    return old;
  end if;
end $function$

```

#### lms.tg_dob_guard()

```sql
CREATE OR REPLACE FUNCTION lms.tg_dob_guard()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
begin
  if coalesce(auth.role(), 'service_role') = 'service_role' then return new; end if;
  if lms.has_role(auth.uid(), 'admin') then return new; end if;
  if new.date_of_birth is distinct from old.date_of_birth or new.dob_source is distinct from old.dob_source
     or new.dob_set_at is distinct from old.dob_set_at or new.dob_locked_at is distinct from old.dob_locked_at then
    raise exception 'dob_locked';
  end if;
  return new;
end $function$

```

#### lms.tg_enrollment_events_immutable()

```sql
CREATE OR REPLACE FUNCTION lms.tg_enrollment_events_immutable()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'lms'
AS $function$
begin
  raise exception 'enrollment_events is append-only';
end $function$

```

#### lms.tg_group_member_section_check()

```sql
CREATE OR REPLACE FUNCTION lms.tg_group_member_section_check()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare v_section uuid;
begin
  select section_id into v_section from lms.cohort_groups where id = new.group_id;
  if v_section is not null and not exists (
    select 1 from lms.cohort_section_members
    where section_id = v_section and student_user_id = new.student_user_id
  ) then
    raise exception 'Student is not in this group''s Section';
  end if;
  return new;
end $function$

```

#### lms.tg_profile_referral_guard()

```sql
CREATE OR REPLACE FUNCTION lms.tg_profile_referral_guard()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
begin
  if auth.uid() is not null then
    if tg_op = 'INSERT' then
      new.invited_by_user_id := null; new.referral_invite_id := null;
    else
      new.invited_by_user_id := old.invited_by_user_id; new.referral_invite_id := old.referral_invite_id;
    end if;
  end if;
  return new;
end $function$

```

#### lms.tg_referral_application()

```sql
CREATE OR REPLACE FUNCTION lms.tg_referral_application()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare a jsonb:=to_jsonb(new); v_email text; v_referrer uuid; v_src text;
begin
  v_email:=lms.referral_email_identity(a->>'student_email');
  if v_email is null or v_email='' then return new; end if;
  perform pg_advisory_xact_lock(hashtext('referral-payment:'||v_email));
  if tg_op='INSERT' then
    v_src:=btrim(coalesce(a->>'utm_source',''));
    if coalesce((select program_referral_enabled from lms.referral_settings where id),false)
      and exists(select 1 from lms.referral_program_integrations where source_table=tg_table_name and enabled and verified_at is not null)
      and v_src ~ '^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$'
      and v_email not like '<redacted>@example-group.com' and v_email not like '<redacted>@partner-b.in' then
      select p.user_id into v_referrer from lms.student_profiles p
        join lms.user_roles u on u.user_id=p.user_id and u.role='student'
        where p.user_id=v_src::uuid and p.account_status='active'
          and lms.referral_email_identity(p.email)<>v_email
          and p.email not like '<redacted>@example-group.com' and p.email not like '<redacted>@partner-b.in';
    end if;
    insert into lms.referral_application_attribution(source_table,application_id,friend_identity,program_code,referrer_user_id)
      values(tg_table_name,new.id,v_email,coalesce(a->>'program_code',''),v_referrer)
      on conflict do nothing;
  end if;
  if a->>'payment_status'='paid' and coalesce((a->>'amount')::numeric,0)>0 then
    insert into lms.referral_first_payments(friend_identity,source_table,application_id,paid_at,paid_amount,paid_currency,referrer_user_id)
      select at.friend_identity,tg_table_name,new.id,coalesce((a->>'payment_date')::timestamptz,now()),
        (a->>'amount')::numeric,upper(btrim(a->>'currency')),at.referrer_user_id
      from lms.referral_application_attribution at where at.source_table=tg_table_name and at.application_id=new.id
      on conflict do nothing;
  end if;
  return new;
end $function$

```

#### lms.tg_section_member_cleanup()

```sql
CREATE OR REPLACE FUNCTION lms.tg_section_member_cleanup()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
begin
  delete from lms.cohort_group_members gm
  using lms.cohort_groups g
  where gm.group_id = g.id and g.section_id = old.section_id
    and gm.student_user_id = old.student_user_id
    and (tg_op = 'DELETE' or new.section_id <> old.section_id);
  return coalesce(new, old);
end $function$

```

#### lms.tg_submission_review_guard()

```sql
CREATE OR REPLACE FUNCTION lms.tg_submission_review_guard()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
begin
  if auth.uid() is null or lms.has_role(auth.uid(), 'admin') then
    if tg_op = 'UPDATE' and new.review_status is distinct from old.review_status then
      new.reviewed_by := coalesce(new.reviewed_by, auth.uid());
      new.reviewed_at := coalesce(new.reviewed_at, now());
    end if;
    return new;
  end if;
  if tg_op = 'INSERT' then
    new.review_status := 'pending'; new.reviewed_by := null; new.reviewed_at := null;
  else
    new.review_status := old.review_status; new.reviewed_by := old.reviewed_by; new.reviewed_at := old.reviewed_at;
  end if;
  return new;
end $function$

```

#### lms.tg_ticket_touch()

```sql
CREATE OR REPLACE FUNCTION lms.tg_ticket_touch()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
begin
  update lms.support_tickets
     set last_message_at = now(), updated_at = now()
   where id = new.ticket_id;
  return new;
end $function$

```

#### lms.tg_vault_cashback()

```sql
CREATE OR REPLACE FUNCTION lms.tg_vault_cashback()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_settings lms.vault_settings; v_key text; v_rate numeric; v_amt numeric;
  v_prev_paid boolean; v_prev_refund boolean; v_started boolean;
begin
  v_key := 'cashback:' || new.id::text;
  select * into v_settings from lms.vault_settings where id;
  if not v_settings.vault_enabled then return new; end if;
  if tg_op = 'UPDATE' then
    v_prev_paid := old.payment_status = 'paid';
    v_prev_refund := old.unenrolled_reason = 'refund';
  else
    v_prev_paid := false; v_prev_refund := false;
  end if;
  if new.payment_status = 'paid' and new.access_status = 'active'
     and not coalesce(v_prev_paid, false)
     and new.amount is not null and new.amount > 0 then
    if coalesce(new.currency, 'USD') <> 'USD' then
      insert into lms.vault_review_flags (student_user_id, kind, detail)
        values (new.student_user_id, 'integrity', jsonb_build_object('reason', 'non_usd_cashback_skipped', 'enrollment_id', new.id, 'currency', new.currency));
      return new;
    end if;
    v_rate := v_settings.cashback_rate_online;
    v_amt := floor(new.amount * v_rate / 100);
    if v_amt <= 0 then return new; end if;
    v_started := lms._vault_cohort_started(new.cohort_id);
    perform lms.vault_earn(new.student_user_id, 'loyalty', v_amt, 'cashback', v_key,
      case when v_started then 'available' else 'pending' end, new.id, new.cohort_id, null, false,
      case when v_started then 'Cashback released · the program has started'
           else 'Cashback pending · released when the program starts' end);
  elsif new.unenrolled_reason = 'refund' and not coalesce(v_prev_refund, false) then
    perform lms.vault_reverse(v_key);
  end if;
  return new;
end $function$

```

#### lms.tg_vault_cohort_completed()

```sql
CREATE OR REPLACE FUNCTION lms.tg_vault_cohort_completed()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_settings lms.vault_settings; r record; v_notif uuid; v_n int := 0; v_was_complete boolean;
begin
  v_was_complete := case when tg_op = 'UPDATE' then old.is_complete else false end;
  if not (new.is_complete and not coalesce(v_was_complete, false)) then
    return new;
  end if;
  select * into v_settings from lms.vault_settings where id;
  if not v_settings.vault_enabled then return new; end if;

  for r in select id, student_user_id from lms.enrollments where cohort_id = new.cohort_id and access_status = 'active' loop
    perform lms.vault_earn(r.student_user_id, 'effort', v_settings.effort_completion, 'completion',
      'completion:' || new.cohort_id::text || ':' || r.student_user_id::text,
      'available', null, new.cohort_id, null, true, 'Program completed');
    perform lms.vault_release('cashback:' || r.id::text);
    v_n := v_n + 1;
  end loop;

  if v_n > 0 then
    insert into lms.notifications (type, title, body, link_url)
      values ('vault', 'Your Program Is Complete', 'Your Ops Rewards completion reward has been credited in My Vault.', '/student/vault')
      returning id into v_notif;
    insert into lms.notification_recipients (notification_id, student_user_id)
      select v_notif, e.student_user_id from lms.enrollments e where e.cohort_id = new.cohort_id and e.access_status = 'active';
  end if;
  return new;
end $function$

```

#### lms.tg_vault_earn_attendance()

```sql
CREATE OR REPLACE FUNCTION lms.tg_vault_earn_attendance()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_student uuid; v_session uuid; v_cohort uuid; v_key text; v_settings lms.vault_settings; v_active boolean;
begin
  if tg_op = 'DELETE' then
    v_student := old.student_user_id; v_session := old.session_id;
  else
    v_student := new.student_user_id; v_session := new.session_id;
  end if;
  v_key := 'attendance:' || v_session::text || ':' || v_student::text;

  if (tg_op in ('INSERT','UPDATE')) and new.status = 'present' then
    select cohort_id into v_cohort from lms.sessions where id = new.session_id;
    select exists(
      select 1 from lms.enrollments e
      where e.student_user_id = v_student and e.cohort_id = v_cohort and e.access_status = 'active'
    ) into v_active;
    if v_active then
      select * into v_settings from lms.vault_settings where id;
      if v_settings.vault_enabled then
        perform lms.vault_earn(v_student, 'effort', v_settings.effort_session, 'attendance', v_key,
          'available', null, v_cohort, null, true, 'Attended a session');
      end if;
    end if;
  elsif (tg_op = 'DELETE') or (tg_op = 'UPDATE' and old.status = 'present' and new.status <> 'present') then
    perform lms.vault_reverse(v_key);
  end if;
  if tg_op = 'DELETE' then return old; end if;
  return new;
end $function$

```

#### lms.tg_vault_earn_feedback()

```sql
CREATE OR REPLACE FUNCTION lms.tg_vault_earn_feedback()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_cohort uuid; v_source text; v_key text;
begin
  if not coalesce((select vault_enabled from lms.vault_settings where id), false) then return new; end if;
  if tg_table_name = 'session_feedback' then
    select s.cohort_id into v_cohort from lms.sessions s
      where s.id = new.session_id and s.cohort_id = new.cohort_id
      and s.session_datetime <= now();
    v_source := 'session_feedback';
    v_key := v_source || ':' || new.session_id::text || ':' || new.student_user_id::text;
  elsif tg_table_name = 'program_feedback' then
    select c.cohort_id into v_cohort from lms.cohort_completions c
      where c.cohort_id = new.cohort_id and c.is_complete;
    v_source := 'program_feedback';
    v_key := v_source || ':' || new.cohort_id::text || ':' || new.student_user_id::text;
  else
    raise exception 'unsupported_feedback_table';
  end if;
  if v_cohort is null or not exists (
    select 1 from lms.enrollments e where e.cohort_id = v_cohort
      and e.student_user_id = new.student_user_id and e.access_status = 'active'
  ) then return new; end if;
  -- vault_earn locks per student and deduplicates source_key; feedback and
  -- its lot/ledger entry commit together. Browser never writes wallet credit.
  perform lms.vault_earn(new.student_user_id, 'effort', 5, v_source, v_key,
    'available', null, v_cohort, null, true,
    case when v_source = 'session_feedback' then 'Session feedback submitted' else 'Program feedback submitted' end);
  return new;
end $function$

```

#### lms.tg_vault_earn_project()

```sql
CREATE OR REPLACE FUNCTION lms.tg_vault_earn_project()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_key text; v_settings lms.vault_settings; v_rewarded_count int;
begin
  v_key := 'project:' || new.id::text;
  if new.review_status = 'approved' and (tg_op = 'INSERT' or old.review_status is distinct from 'approved') then
    select * into v_settings from lms.vault_settings where id;
    if not v_settings.vault_enabled then return new; end if;
    select count(*) into v_rewarded_count from lms.vault_lots
      where source_type = 'project' and cohort_id = new.cohort_id
        and student_user_id = new.student_user_id and state <> 'cancelled';
    if v_rewarded_count < v_settings.max_rewarded_projects_per_cohort then
      perform lms.vault_earn(new.student_user_id, 'effort', v_settings.effort_project, 'project', v_key,
        'available', null, new.cohort_id, null, true, 'Project approved');
    end if;
  elsif tg_op = 'UPDATE' and old.review_status = 'approved' and new.review_status <> 'approved' then
    perform lms.vault_reverse(v_key);
  end if;
  return new;
end $function$

```

#### lms.tg_vault_ledger_immutable()

```sql
CREATE OR REPLACE FUNCTION lms.tg_vault_ledger_immutable()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'lms'
AS $function$
begin raise exception 'vault_ledger is append-only'; end $function$

```

#### lms.tg_vault_lot_notify()

```sql
CREATE OR REPLACE FUNCTION lms.tg_vault_lot_notify()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare s lms.vault_settings; v_after numeric; v_before numeric; v_lv_a int := 0; v_lv_b int := 0; i int;
  fmt text;
begin
  select * into s from lms.vault_settings where id;
  if not s.vault_enabled then return new; end if;
  fmt := '$' || trim(to_char(new.amount, 'FM999999990.00'));
  fmt := regexp_replace(fmt, '\.00$', '');
  if tg_op = 'INSERT' then
    if new.source_type = 'completion' then
-- ... truncated: showing first 15 of 45 lines
```

#### lms.tg_vault_referral_reward()

```sql
CREATE OR REPLACE FUNCTION lms.tg_vault_referral_reward()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare f lms.referral_first_payments; s lms.vault_settings; r lms.referral_settings;
  v_key text; v_lot record; v_fx numeric; v_rate numeric; v_kind text;
begin
  if new.unenrolled_reason='refund' or new.payment_status in ('refunded_full','refunded') then
    for v_lot in select source_key from lms.vault_lots where source_type='referral' and wallet='loyalty' and enrollment_id=new.id loop
      perform lms.vault_reverse(v_lot.source_key);
    end loop;
    return new;
  end if;
  if new.payment_status<>'paid' or new.access_status<>'active' or coalesce(new.amount,0)<=0 then return new; end if;
  select * into s from lms.vault_settings where id;
  select * into r from lms.referral_settings where id;
  if not s.vault_enabled or not r.program_referral_enabled then return new; end if;
  select * into f from lms.referral_first_payments where source_table=new.source_table and application_id=new.source_application_id for update;
  if not found or f.referrer_user_id is null or f.referrer_user_id=new.student_user_id
    or f.paid_amount is null or f.paid_currency is null or upper(btrim(new.currency))<>f.paid_currency then return new; end if;
  if not exists(select 1 from lms.student_profiles where user_id=new.student_user_id
    and lms.referral_email_identity(email)=f.friend_identity) then return new; end if;
  v_key:='referral-payment:'||f.friend_identity;
  if exists(select 1 from lms.vault_lots where source_key=v_key) then return new; end if;
  -- Fixed rates only; an unknown currency never earns guessed credit.
  select case f.paid_currency when 'USD' then 1 when 'INR' then 85 when 'SAR' then 3.75
    when 'AUD' then 1.5 when 'EUR' then 0.9 when 'GBP' then 0.8 end into v_fx;
  if v_fx is null then return new; end if;
  v_kind:=coalesce(lms.vault_program_kind(new.cohort_id),'online');
  v_rate:=(case v_kind when 'camp' then r.rate_camp when 'in_person' then r.rate_in_person else r.rate_online end)*s.referral_multiplier;
  perform lms.vault_earn(f.referrer_user_id,'loyalty',floor(least(new.amount,f.paid_amount)/v_fx*v_rate/100),'referral',v_key,
    'pending',new.id,new.cohort_id,greatest(now(),f.paid_at)+make_interval(days=>s.referral_release_days),false,'First paid program referral');
  update lms.referral_first_payments set enrollment_id=new.id where friend_identity=f.friend_identity;
  return new;
end $function$

```

#### lms.touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION lms.touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'lms', 'public'
AS $function$
begin new.updated_at = now(); return new; end $function$

```

#### lms.unblock_shared_whatsapp_number(p_phone text, p_reason text)

```sql
CREATE OR REPLACE FUNCTION lms.unblock_shared_whatsapp_number(p_phone text, p_reason text)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'pg_catalog', 'lms', 'portal'
AS $function$
DECLARE v_exact boolean; v_prefixed boolean;
BEGIN
  IF auth.uid() IS NULL OR NOT lms.has_role(auth.uid(),'admin') THEN
    RAISE EXCEPTION 'Admin access required' USING ERRCODE='42501';
  END IF;
  IF p_phone IS NULL OR p_phone !~ '^[1-9][0-9]{7,14}$' THEN
    RAISE EXCEPTION 'Full international number required';
  END IF;
  -- Handle both exact and 00-prefixed report representations together.
  v_exact:=portal.set_whatsapp_undeliverable_status(p_phone,false,p_reason,auth.uid());
  v_prefixed:=portal.set_whatsapp_undeliverable_status('00'||p_phone,false,p_reason,auth.uid());
  RETURN v_exact OR v_prefixed;
END $function$

```

#### lms.update_my_mentor_profile(p_full_name text, p_university text, p_country text, p_bio text)

```sql
CREATE OR REPLACE FUNCTION lms.update_my_mentor_profile(p_full_name text, p_university text, p_country text, p_bio text)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
declare v_mid uuid; v_name text := btrim(coalesce(p_full_name, ''));
begin
  select mentor_id into v_mid from lms.mentor_accounts where user_id = auth.uid();
  if v_mid is null then raise exception 'not_a_mentor' using errcode = '42501'; end if;
  if length(v_name) < 2 or length(v_name) > 120 then raise exception 'invalid_name'; end if;
  if length(coalesce(p_university, '')) > 160 or length(coalesce(p_country, '')) > 80
     or length(coalesce(p_bio, '')) > 1500 then raise exception 'too_long'; end if;
  update public.all_mentors set
    full_name = v_name,
    university = nullif(btrim(coalesce(p_university, '')), ''),
    country = nullif(btrim(coalesce(p_country, '')), ''),
    bio = nullif(btrim(coalesce(p_bio, '')), ''),
    updated_by_platform = 'lms_mentor_self'
  where id = v_mid;
end $function$

```

#### lms.update_vault_settings(p_patch jsonb, p_reason text, p_actor uuid)

```sql
CREATE OR REPLACE FUNCTION lms.update_vault_settings(p_patch jsonb, p_reason text, p_actor uuid)
 RETURNS lms.vault_settings
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_before lms.vault_settings; v_after lms.vault_settings; k text;
begin
  if coalesce(trim(p_reason), '') = '' then raise exception 'reason_required'; end if;
  select * into v_before from lms.vault_settings where id;
  update lms.vault_settings set
    cashback_rate_online = coalesce((p_patch->>'cashback_rate_online')::numeric, cashback_rate_online),
    cashback_rate_in_person = coalesce((p_patch->>'cashback_rate_in_person')::numeric, cashback_rate_in_person),
    cashback_rate_camp = coalesce((p_patch->>'cashback_rate_camp')::numeric, cashback_rate_camp),
    effort_profile = coalesce((p_patch->>'effort_profile')::numeric, effort_profile),
    effort_session = coalesce((p_patch->>'effort_session')::numeric, effort_session),
    effort_project = coalesce((p_patch->>'effort_project')::numeric, effort_project),
    effort_completion = coalesce((p_patch->>'effort_completion')::numeric, effort_completion),
    max_rewarded_projects_per_cohort = coalesce((p_patch->>'max_rewarded_projects_per_cohort')::int, max_rewarded_projects_per_cohort),
    level_thresholds = coalesce(p_patch->'level_thresholds', level_thresholds),
    level_caps = coalesce(p_patch->'level_caps', level_caps),
    loyalty_expiry_months = coalesce((p_patch->>'loyalty_expiry_months')::int, loyalty_expiry_months),
    effort_inactivity_months = coalesce((p_patch->>'effort_inactivity_months')::int, effort_inactivity_months),
    voucher_valid_hours = coalesce((p_patch->>'voucher_valid_hours')::int, voucher_valid_hours),
    referral_multiplier = coalesce((p_patch->>'referral_multiplier')::numeric, referral_multiplier),
    referral_release_days = coalesce((p_patch->>'referral_release_days')::int, referral_release_days),
    vault_enabled = coalesce((p_patch->>'vault_enabled')::boolean, vault_enabled),
    redemption_enabled = coalesce((p_patch->>'redemption_enabled')::boolean, redemption_enabled),
    allow_free_programs = coalesce((p_patch->>'allow_free_programs')::boolean, allow_free_programs),
    restore_credit_on_refund = coalesce((p_patch->>'restore_credit_on_refund')::boolean, restore_credit_on_refund),
    updated_at = now(), updated_by = p_actor
  where id
  returning * into v_after;
  insert into lms.vault_settings_history (changed_by, reason, before, after)
    values (p_actor, p_reason, to_jsonb(v_before), to_jsonb(v_after));
  return v_after;
end $function$

```

#### lms.validate_whatsapp_recipient_config()

```sql
CREATE OR REPLACE FUNCTION lms.validate_whatsapp_recipient_config()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'pg_catalog', 'lms'
AS $function$
BEGIN
 IF NEW.recipient_mode IN ('parent','both')
    AND NEW.parent_template->'same_as_student' IS DISTINCT FROM 'true'::jsonb
    AND (NEW.parent_template->>'template_name' IS NULL OR btrim(NEW.parent_template->>'template_name')='')
    AND NEW.trigger_event IS DISTINCT FROM 'session_reminder' THEN
  IF NEW.is_active THEN RAISE EXCEPTION 'Configure a parent template before activating parent messages'; END IF;
 END IF;
 IF coalesce(NEW.parent_template->'variable_mapping','[]'::jsonb) ? 'temp_password' THEN
  RAISE EXCEPTION 'Passwords cannot be included in parent templates';
 END IF;
 IF NEW.recipient_mode IN ('parent','both')
    AND NEW.parent_template->'same_as_student' = 'true'::jsonb
    AND coalesce(NEW.variable_mapping,'[]'::jsonb) ? 'temp_password' THEN
  RAISE EXCEPTION 'Passwords cannot be included in parent templates';
 END IF;
 RETURN NEW;
END $function$

```

#### lms.vault_adjust(p_student uuid, p_wallet text, p_amount numeric, p_reason text, p_actor uuid)

```sql
CREATE OR REPLACE FUNCTION lms.vault_adjust(p_student uuid, p_wallet text, p_amount numeric, p_reason text, p_actor uuid)
 RETURNS lms.vault_lots
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.vault_lots; v_key text; v_avail numeric; v_need numeric; v_take numeric; r record;
begin
  if coalesce(trim(p_reason), '') = '' or length(trim(p_reason)) < 10 then raise exception 'reason_required_10_chars'; end if;
  if p_amount = 0 then raise exception 'amount_required'; end if;
  perform pg_advisory_xact_lock(hashtext('lms.vault:' || p_student::text));
  v_key := 'adjustment:' || gen_random_uuid()::text;
  if p_amount > 0 then
    return lms.vault_earn(p_student, p_wallet, p_amount, 'adjustment', v_key, 'available', null, null, null, false,
      'Admin adjustment: ' || p_reason);
  end if;
  select coalesce(sum(remaining), 0) into v_avail from lms.vault_lots
    where student_user_id = p_student and wallet = p_wallet and state = 'available';
  if v_avail < -p_amount then raise exception 'insufficient_balance'; end if;
  -- Deduct FIFO across available lots (oldest first), recording one ledger row.
  v_need := -p_amount;
  for r in
    select id, remaining from lms.vault_lots
    where student_user_id = p_student and wallet = p_wallet and state = 'available' and remaining > 0
    order by created_at
    for update
  loop
    exit when v_need <= 0;
    v_take := least(r.remaining, v_need);
    update lms.vault_lots set remaining = remaining - v_take where id = r.id;
    v_need := v_need - v_take;
  end loop;
  insert into lms.vault_ledger (student_user_id, wallet, kind, amount, ref_type, description)
    values (p_student, p_wallet, 'adjust', p_amount, 'adjustment', 'Admin adjustment: ' || p_reason);
  select * into v from lms.vault_lots where student_user_id = p_student and wallet = p_wallet order by created_at desc limit 1;
  return v;
end $function$

```

#### lms.vault_birthday_due()

```sql
CREATE OR REPLACE FUNCTION lms.vault_birthday_due()
 RETURNS TABLE(grant_id uuid)
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare s lms.vault_settings; r record; v_lot lms.vault_lots; v_gid uuid; v_exp timestamptz;
begin
  select * into s from lms.vault_settings where id;
  if not (s.vault_enabled and s.birthday_enabled) or coalesce(s.birthday_amount, 0) <= 0 then return; end if;
  for r in
    with base as (
      select p.user_id, p.date_of_birth dob, p.dob_source, p.dob_set_at,
        lms.birthday_zone_for(p.country, p.state, p.phone) z
      from lms.student_profiles p
-- ... truncated: showing first 15 of 48 lines
```

#### lms.vault_birthday_expire()

```sql
CREATE OR REPLACE FUNCTION lms.vault_birthday_expire()
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare r record; n int := 0;
begin
  perform lms._vault_expire_vouchers_for(null);
  for r in select * from lms.vault_lots where source_type = 'birthday' and state = 'available' and expires_at <= now() for update loop
    perform pg_advisory_xact_lock(hashtext('lms.vault:' || r.student_user_id::text));
    update lms.vault_lots set state = 'expired', remaining = 0 where id = r.id;
    if r.remaining > 0 then
      insert into lms.vault_ledger (student_user_id, wallet, lot_id, kind, amount, ref_type, ref_id, description)
        values (r.student_user_id, r.wallet, r.id, 'expire', -r.remaining, 'birthday', r.source_key, 'Birthday Bonus ended after 5 days');
    end if;
    n := n + 1;
  end loop;
  return n;
end $function$

```

#### lms.vault_cancel_voucher(p_student uuid, p_voucher uuid)

```sql
CREATE OR REPLACE FUNCTION lms.vault_cancel_voucher(p_student uuid, p_voucher uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_owner uuid; v_res text; v_promo uuid;
begin
  select student_user_id, promo_code_id into v_owner, v_promo from lms.vault_vouchers where id = p_voucher;
  if v_owner is null or v_owner <> p_student then return jsonb_build_object('ok', false, 'error', 'not_found'); end if;
  perform pg_advisory_xact_lock(hashtext('lms.vault:' || p_student::text));
  v_res := lms._vault_release_voucher(p_voucher, 'cancelled');
  if v_res = 'cancelled' then
    -- Remove the coupon code and the voucher record entirely.
    delete from public.promo_codes where id = v_promo;
    delete from lms.vault_vouchers where id = p_voucher;
    return jsonb_build_object('ok', true);
  end if;
  return jsonb_build_object('ok', false, 'error', case v_res when 'used' then 'already_used' else 'not_active' end);
end $function$

```

#### lms.vault_commit_voucher_from_redemption(p_redemption_id uuid)

```sql
CREATE OR REPLACE FUNCTION lms.vault_commit_voucher_from_redemption(p_redemption_id uuid)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare rd record; v lms.vault_vouchers; a jsonb; v_email text; v_disc numeric; v_total numeric; v_issue text[] := '{}';
begin
  select * into rd from public.promo_redemptions where id = p_redemption_id;
  select * into v from lms.vault_vouchers
    where (promo_code_id = rd.promo_code_id or code = upper(rd.promo_code)) order by created_at desc limit 1 for update;
  if not found then return; end if;
  update public.promo_codes set is_active = false, updated_at = now() where id = v.promo_code_id;
  v_total := v.effort_amount + v.loyalty_amount;
  v_disc := coalesce(rd.discount_amount, rd.amount_before - rd.amount_after);
-- ... truncated: showing first 15 of 50 lines
```

#### lms.vault_coupon_reminders()

```sql
CREATE OR REPLACE FUNCTION lms.vault_coupon_reminders()
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare r record; n integer := 0;
begin
  for r in
    select v.student_user_id, min(v.expires_at) as exp, count(*) as cnt,
           (array_agg(coalesce(p.program_name, 'your program') order by v.expires_at))[1] as program_name,
           (array_agg(v.code order by v.expires_at))[1] as code
    from lms.vault_vouchers v
    left join public.all_programs p on p.id = v.program_id
    where v.status = 'active' and v.expires_at > now()
      and not exists (
        select 1 from lms.notifications nt
        join lms.notification_recipients nr on nr.notification_id = nt.id
        where nr.student_user_id = v.student_user_id and nt.type = 'vault'
          and nt.title = 'Your Coupon Is Waiting' and nt.created_at > now() - interval '20 hours')
    group by v.student_user_id
  loop
    perform lms.vault_notify(r.student_user_id, 'Your Coupon Is Waiting',
      'Your coupon ' || r.code || ' for ' || r.program_name
      || case when r.cnt > 1 then ' (and ' || (r.cnt - 1) || ' more)' else '' end
      || ' is ready. Complete your application and apply it on the payment page before '
      || to_char(r.exp at time zone 'UTC', 'DD Mon YYYY') || '.');
    n := n + 1;
  end loop;
  return n;
end $function$

```

#### lms.vault_create_voucher(p_student uuid, p_enrollment uuid, p_effort numeric, p_loyalty numeric)

```sql
CREATE OR REPLACE FUNCTION lms.vault_create_voucher(p_student uuid, p_enrollment uuid, p_effort numeric, p_loyalty numeric)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare s lms.vault_settings; q jsonb; v_id uuid := gen_random_uuid(); v_code text; v_promo uuid; v_alloc jsonb;
  v_email text; v_exp timestamptz; v_eff numeric; v_loy numeric;
begin
  select * into s from lms.vault_settings where id;
  if not (s.vault_enabled and s.redemption_enabled) then return jsonb_build_object('ok', false, 'error', 'redemption_off'); end if;
  perform pg_advisory_xact_lock(hashtext('lms.vault:' || p_student::text));
  perform lms._vault_expire_vouchers_for(p_student);
  if exists (select 1 from lms.vault_vouchers where enrollment_id = p_enrollment and status = 'active') then
    return jsonb_build_object('ok', false, 'error', 'active_voucher_exists');
-- ... truncated: showing first 15 of 52 lines
```

#### lms.vault_create_voucher_program(p_student uuid, p_program uuid, p_effort numeric, p_loyalty numeric)

```sql
CREATE OR REPLACE FUNCTION lms.vault_create_voucher_program(p_student uuid, p_program uuid, p_effort numeric, p_loyalty numeric)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare s lms.vault_settings; q jsonb; v_id uuid := gen_random_uuid(); v_code text; v_promo uuid; v_alloc jsonb;
  v_email text; v_exp timestamptz; v_eff numeric; v_loy numeric;
begin
  select * into s from lms.vault_settings where id;
  if not (s.vault_enabled and s.redemption_enabled) then return jsonb_build_object('ok', false, 'error', 'redemption_off'); end if;
  perform pg_advisory_xact_lock(hashtext('lms.vault:' || p_student::text));
  perform lms._vault_expire_vouchers_for(p_student);
  if exists (select 1 from lms.vault_vouchers where student_user_id = p_student and program_id = p_program and status = 'active') then
    return jsonb_build_object('ok', false, 'error', 'active_voucher_exists');
-- ... truncated: showing first 15 of 55 lines
```

#### lms.vault_daily()

```sql
CREATE OR REPLACE FUNCTION lms.vault_daily()
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare r record; v_mismatch jsonb; v_vouchers int := 0; v_referrals int := 0; v_cashback int := 0;
begin
  -- Redeemed coupons are valid for one year, then the code retires. Credit is never expired.
  for r in select id, student_user_id from lms.vault_vouchers
            where status = 'active' and expires_at <= now()
            order by student_user_id, id loop
    perform pg_advisory_xact_lock(hashtext('lms.vault:' || r.student_user_id::text));
    if lms._vault_release_voucher(r.id, 'expired') = 'expired' then
      v_vouchers := v_vouchers + 1;
    end if;
  end loop;

  -- Referral rewards become spendable one day after the friend's payment is verified.
  for r in select source_key from lms.vault_lots
            where source_type = 'referral' and state = 'pending' and release_on <= now() loop
    perform lms.vault_release(r.source_key); v_referrals := v_referrals + 1;
  end loop;

  -- Loyalty cashback becomes spendable once the program has started.
  for r in select l.source_key from lms.vault_lots l
            where l.wallet = 'loyalty' and l.source_type = 'cashback' and l.state = 'pending'
              and lms._vault_cohort_started(l.cohort_id) loop
    perform lms.vault_release(r.source_key); v_cashback := v_cashback + 1;
  end loop;

  select coalesce(jsonb_agg(to_jsonb(x)), '[]'::jsonb) into v_mismatch from lms.vault_integrity_report() x;
  return jsonb_build_object('expired', '[]'::jsonb, 'warned', 0, 'vouchers_expired', v_vouchers,
    'referrals_released', v_referrals, 'cashback_released', v_cashback, 'mismatches', v_mismatch);
end $function$

```

#### lms.vault_earn(p_student uuid, p_wallet text, p_amount numeric, p_source_type text, p_source_key text, p_state text, p_enrollment uuid, p_cohort uuid, p_release_on timestamp with time zone, p_counts_toward_level boolean, p_description text)

```sql
CREATE OR REPLACE FUNCTION lms.vault_earn(p_student uuid, p_wallet text, p_amount numeric, p_source_type text, p_source_key text, p_state text DEFAULT 'available'::text, p_enrollment uuid DEFAULT NULL::uuid, p_cohort uuid DEFAULT NULL::uuid, p_release_on timestamp with time zone DEFAULT NULL::timestamp with time zone, p_counts_toward_level boolean DEFAULT false, p_description text DEFAULT ''::text)
 RETURNS lms.vault_lots
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.vault_lots;
begin
  if p_amount <= 0 then return null; end if;
  perform pg_advisory_xact_lock(hashtext('lms.vault:' || p_student::text));
  select * into v from lms.vault_lots where source_key = p_source_key;
  if found then return v; end if;
  insert into lms.vault_lots (student_user_id, wallet, amount, remaining, state, source_type, source_key,
    enrollment_id, cohort_id, counts_toward_level, release_on, released_at)
  values (p_student, p_wallet, p_amount, p_amount, p_state, p_source_type, p_source_key,
    p_enrollment, p_cohort, p_counts_toward_level, p_release_on,
    case when p_state = 'available' then now() end)
  returning * into v;
  insert into lms.vault_ledger (student_user_id, wallet, lot_id, kind, amount, ref_type, ref_id, description)
    values (p_student, p_wallet, v.id, case when p_state = 'pending' then 'pending' else 'earn' end,
      p_amount, p_source_type, p_source_key, coalesce(nullif(p_description, ''), p_source_type));
  return v;
end $function$

```

#### lms.vault_expire_due()

```sql
CREATE OR REPLACE FUNCTION lms.vault_expire_due()
 RETURNS TABLE(wallet text, lots_expired integer, amount_expired numeric)
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
#variable_conflict use_column
declare v_settings lms.vault_settings; r record; v_last timestamptz;
begin
  select * into v_settings from lms.vault_settings where id;
  for r in select * from lms.vault_lots where wallet = 'loyalty' and state = 'available' and expires_at <= now() for update loop
    update lms.vault_lots set state = 'expired', remaining = 0 where id = r.id;
    if r.remaining > 0 then
      insert into lms.vault_ledger (student_user_id, wallet, lot_id, kind, amount, ref_type, description)
        values (r.student_user_id, 'loyalty', r.id, 'expire', -r.remaining, 'expiry', 'Loyalty credit expired after ' || v_settings.loyalty_expiry_months || ' months');
    end if;
  end loop;
  for r in (
    select student_user_id, sum(remaining) as bal from lms.vault_lots
    where wallet = 'effort' and state = 'available' and remaining > 0
    group by student_user_id
  ) loop
    continue when exists (select 1 from lms.vault_vouchers vv where vv.student_user_id = r.student_user_id and vv.status = 'active');
    select max(created_at) into v_last from lms.vault_ledger where student_user_id = r.student_user_id and wallet = 'effort';
    if v_last is null or v_last <= now() - make_interval(months => v_settings.effort_inactivity_months) then
      update lms.vault_lots set state = 'expired', remaining = 0
        where student_user_id = r.student_user_id and wallet = 'effort' and state = 'available';
      insert into lms.vault_ledger (student_user_id, wallet, kind, amount, ref_type, description)
        values (r.student_user_id, 'effort', 'expire', -r.bal, 'expiry', 'Ops Rewards credit expired after ' || v_settings.effort_inactivity_months || ' months of inactivity');
    end if;
  end loop;
  return query
    select le.wallet, count(*)::int, coalesce(sum(-le.amount), 0)
    from lms.vault_ledger le
    where le.kind = 'expire' and le.ref_type = 'expiry' and le.created_at > now() - interval '1 minute'
    group by le.wallet;
end $function$

```

#### lms.vault_expire_vouchers()

```sql
CREATE OR REPLACE FUNCTION lms.vault_expire_vouchers()
 RETURNS integer
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  select lms._vault_expire_vouchers_for(null);
$function$

```

#### lms.vault_flag_auto_resolve()

```sql
CREATE OR REPLACE FUNCTION lms.vault_flag_auto_resolve()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
begin
  if new.resolved_at is null and (
       new.kind = 'reversal_shortfall'
       or (new.kind = 'integrity' and coalesce(new.detail->>'reason', '') in (
             'non_usd_cashback_skipped', 'non_usd_referral_skipped', 'unknown_currency_referral_skipped'))
     ) then
    new.resolved_at := now();
    new.detail := coalesce(new.detail, '{}'::jsonb) || jsonb_build_object('auto_resolved', true);
  end if;
  return new;
end $function$

```

#### lms.vault_grant_welcome(p_student uuid)

```sql
CREATE OR REPLACE FUNCTION lms.vault_grant_welcome(p_student uuid)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
begin
  if p_student is null then return; end if;
  perform lms.vault_earn(p_student, 'effort', 50, 'welcome', 'welcome:' || p_student::text,
    'available', null, null, null, true, 'Welcome bonus');
end $function$

```

#### lms.vault_integrity_report()

```sql
CREATE OR REPLACE FUNCTION lms.vault_integrity_report()
 RETURNS TABLE(student_user_id uuid, wallet text, lot_remaining numeric, ledger_balance numeric)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'pg_temp'
AS $function$
  WITH lot_balances AS (
    SELECT l.student_user_id, l.wallet, sum(l.remaining) AS balance
    FROM lms.vault_lots l WHERE l.state = 'available'
    GROUP BY l.student_user_id, l.wallet
  ), ledger_balances AS (
    SELECT le.student_user_id, le.wallet, sum(le.amount) AS balance
    FROM lms.vault_ledger le
    WHERE le.kind IN ('earn','release','reserve','unreserve','spend','reversal','expire','adjust')
      -- Historical pending cashback was rounded before release. Release already
      -- credits the rounded amount, so this pending-only adjustment must not
      -- also debit the available balance, even after that lot is released.
      AND NOT (
        le.kind = 'adjust' AND coalesce(le.ref_type, '') = 'rounding'
        AND EXISTS (
          SELECT 1 FROM lms.vault_ledger pending
          WHERE pending.lot_id = le.lot_id
            AND pending.student_user_id = le.student_user_id AND pending.wallet = le.wallet
            AND pending.kind = 'pending' AND pending.created_at <= le.created_at
        )
        AND NOT EXISTS (
          SELECT 1 FROM lms.vault_ledger credited
          WHERE credited.lot_id = le.lot_id
            AND credited.kind IN ('earn','release') AND credited.created_at <= le.created_at
        )
      )
    GROUP BY le.student_user_id, le.wallet
  )
  SELECT coalesce(l.student_user_id, le.student_user_id), coalesce(l.wallet, le.wallet),
    coalesce(l.balance, 0), coalesce(le.balance, 0)
  FROM lot_balances l FULL OUTER JOIN ledger_balances le
    USING (student_user_id, wallet)
  WHERE coalesce(l.balance, 0) <> coalesce(le.balance, 0);
$function$

```

#### lms.vault_lifetime_points(p_student uuid)

```sql
CREATE OR REPLACE FUNCTION lms.vault_lifetime_points(p_student uuid)
 RETURNS numeric
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'pg_temp'
AS $function$
  select coalesce(sum(amount), 0) from lms.vault_lots
  where student_user_id = p_student and wallet = 'effort' and counts_toward_level
    and state in ('available','expired','reversed'); -- reversed still keeps a (clamped) remainder via vault_reverse
$function$

```

#### lms.vault_mark_walkthrough_seen()

```sql
CREATE OR REPLACE FUNCTION lms.vault_mark_walkthrough_seen()
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
begin
  if auth.uid() is null then raise exception 'not authenticated'; end if;
  insert into lms.vault_walkthrough_seen(user_id) values (auth.uid()) on conflict (user_id) do nothing;
end $function$

```

#### lms.vault_notify(p_student uuid, p_title text, p_body text)

```sql
CREATE OR REPLACE FUNCTION lms.vault_notify(p_student uuid, p_title text, p_body text)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v_id uuid;
begin
  if not coalesce((select vault_enabled from lms.vault_settings where id), false) then return; end if;
  if not coalesce((select notifications_enabled from lms.vault_settings where id), false) then return; end if;
  insert into lms.notifications (type, title, body, link_url)
    values ('vault', p_title, p_body, '/student/vault') returning id into v_id;
  insert into lms.notification_recipients (notification_id, student_user_id) values (v_id, p_student);
end $function$

```

#### lms.vault_program_kind(p_cohort uuid)

```sql
CREATE OR REPLACE FUNCTION lms.vault_program_kind(p_cohort uuid)
 RETURNS text
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  select case
    when exists (select 1 from unnest(coalesce(p.categories, '{}'::text[])) c where lower(c) ~ '\mcamps?\M') then 'camp'
    when lower(coalesce(p.format, '')) like '%person%'
      or exists (select 1 from unnest(coalesce(p.categories, '{}'::text[])) c where lower(c) in ('in person','in-person')) then 'in_person'
    else 'online' end
  from public.all_cohorts c join public.all_programs p on p.id = c.program_id
  where c.id = p_cohort;
$function$

```

#### lms.vault_public_settings()

```sql
CREATE OR REPLACE FUNCTION lms.vault_public_settings()
 RETURNS jsonb
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
  select case
    when lms.has_role(auth.uid(), 'admin') then to_jsonb(s) - 'updated_by'
    else jsonb_build_object(
      'cashback_rate_online', s.cashback_rate_online,
      'cashback_rate_in_person', s.cashback_rate_in_person,
      'cashback_rate_camp', s.cashback_rate_camp,
      'effort_profile', s.effort_profile,
      'effort_session', s.effort_session,
      'effort_project', s.effort_project,
      'effort_completion', s.effort_completion,
      'level_thresholds', s.level_thresholds,
      'level_caps', s.level_caps,
      'referral_release_days', s.referral_release_days,
      'vault_enabled', s.vault_enabled,
      'redemption_enabled', s.redemption_enabled)
  end
  from lms.vault_settings s where s.id = true and auth.uid() is not null;
$function$

```

#### lms.vault_quote(p_student uuid, p_enrollment uuid, p_effort numeric, p_loyalty numeric)

```sql
CREATE OR REPLACE FUNCTION lms.vault_quote(p_student uuid, p_enrollment uuid, p_effort numeric, p_loyalty numeric)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare s lms.vault_settings; e record; p record; v_points numeric; v_level int := 0; v_cap numeric := 0; i int;
  v_eff_bal numeric; v_loy_bal numeric; v_list numeric; v_base numeric; v_eff_max numeric; v_eff numeric; v_loy numeric; v_floor numeric;
  v_kind text; v_rate numeric; v_cur text; v_fx numeric := 1; v_fx_date date; v_base_local numeric; v_list_local numeric; v_disc_local numeric;
begin
  select * into s from lms.vault_settings where id;
  select en.id, en.student_user_id, en.cohort_id, en.payment_status, en.access_status, c.start_date, c.program_id
    into e from lms.enrollments en join public.all_cohorts c on c.id = en.cohort_id
    where en.id = p_enrollment;
  if not found or e.student_user_id <> p_student then return jsonb_build_object('ok', false, 'error', 'not_found'); end if;
-- ... truncated: showing first 15 of 60 lines
```

#### lms.vault_quote_program(p_student uuid, p_program uuid, p_effort numeric, p_loyalty numeric)

```sql
CREATE OR REPLACE FUNCTION lms.vault_quote_program(p_student uuid, p_program uuid, p_effort numeric, p_loyalty numeric)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare s lms.vault_settings; p record; c record; v_points numeric; v_level int := 0; v_cap numeric := 0; i int;
  v_eff_bal numeric; v_loy_bal numeric; v_list numeric; v_base numeric; v_eff_max numeric; v_eff numeric; v_loy numeric; v_floor numeric;
  v_kind text; v_rate numeric; v_cur text; v_fx numeric := 1; v_fx_date date; v_base_local numeric; v_list_local numeric; v_disc_local numeric;
  v_enr uuid;
begin
  select * into s from lms.vault_settings where id;
  select id, program_name, sticker_price, early_bird_price, coalesce(price_currency, 'USD') as cur into p
    from public.all_programs where id = p_program and coalesce(is_active, true);
  if not found then return jsonb_build_object('ok', false, 'error', 'not_found'); end if;
-- ... truncated: showing first 15 of 65 lines
```

#### lms.vault_redeem_catalog(p_student uuid)

```sql
CREATE OR REPLACE FUNCTION lms.vault_redeem_catalog(p_student uuid)
 RETURNS jsonb
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  select coalesce(jsonb_agg(x order by x.next_start, x.program_name), '[]'::jsonb) from (
    select p.id as program_id, p.program_name, p.format, p.apply_url, upper(coalesce(p.price_currency, 'USD')) as currency,
      case when p.early_bird_price > 0 then least(p.sticker_price, p.early_bird_price) else p.sticker_price end as price_local,
      min(c.start_date) as next_start,
      (select coalesce(jsonb_agg(en.id), '[]'::jsonb) from lms.enrollments en join public.all_cohorts ac on ac.id = en.cohort_id
        where en.student_user_id = p_student and ac.program_id = p.id and en.payment_status = 'pending') as enrollment_ids,
      (select coalesce(jsonb_agg(en.id), '[]'::jsonb) from lms.enrollments en join public.all_cohorts ac on ac.id = en.cohort_id
        where en.student_user_id = p_student and ac.program_id = p.id and en.payment_status = 'paid' and en.unenrolled_at is null) as paid_enrollment_ids
    from public.all_programs p join public.all_cohorts c on c.program_id = p.id and c.start_date > current_date
    where coalesce(p.is_active, true) and p.sticker_price > 0
    group by p.id) x;
$function$

```

#### lms.vault_redemptions()

```sql
CREATE OR REPLACE FUNCTION lms.vault_redemptions()
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public'
AS $function$
begin
  if not lms.has_role(auth.uid(), 'admin') then
    raise exception 'not authorized';
  end if;

  return coalesce((
    select jsonb_agg(row_to_json(t) order by t.created_at desc)
    from (
      select
        v.id,
        v.code,
        v.status,
        v.student_user_id,
        sp.full_name as student_name,
        sp.email as student_email,
        ap.program_name,
        v.effort_amount,
        v.loyalty_amount,
        v.list_price,
        v.cash_due,
        v.currency,
        v.created_at,
        v.used_at,
        v.expires_at,
        v.cancelled_at
      from lms.vault_vouchers v
      left join lms.student_profiles sp on sp.user_id = v.student_user_id
      left join public.all_programs ap on ap.id = v.program_id
    ) t
  ), '[]'::jsonb);
end;
$function$

```

#### lms.vault_referral_overview()

```sql
CREATE OR REPLACE FUNCTION lms.vault_referral_overview()
 RETURNS jsonb
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
  with inv as (
    select i.*, (case when i.status = 'invited' and i.expires_at <= now() then 'expired' else i.status end) as eff_status
    from lms.referral_invites i
  ), friend_enr as (
    select i.id as invite_id,
      bool_or(e.payment_status = 'paid' and e.access_status = 'active' and e.unenrolled_at is null) as enrolled,
      bool_or(e.id is not null) as applied,
      bool_or(e.payment_status like 'refund%' or coalesce(e.unenrolled_reason, '') ilike '%refund%') as refunded,
      (array_agg(p.program_name order by e.enrolled_at) filter (where e.id is not null))[1] as program_name
-- ... truncated: showing first 15 of 52 lines
```

#### lms.vault_release(p_source_key text)

```sql
CREATE OR REPLACE FUNCTION lms.vault_release(p_source_key text)
 RETURNS lms.vault_lots
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.vault_lots;
begin
  select * into v from lms.vault_lots where source_key = p_source_key for update;
  if not found or v.state <> 'pending' then return v; end if;
  update lms.vault_lots set state = 'available', released_at = now(), expires_at = null
    where id = v.id returning * into v;
  insert into lms.vault_ledger (student_user_id, wallet, lot_id, kind, amount, ref_type, ref_id, description)
    values (v.student_user_id, v.wallet, v.id, 'release', v.amount, v.source_type, v.source_key, 'Released: ' || v.source_type);
  return v;
end $function$

```

#### lms.vault_report(p_from timestamp with time zone, p_to timestamp with time zone, p_exclude uuid[])

```sql
CREATE OR REPLACE FUNCTION lms.vault_report(p_from timestamp with time zone DEFAULT NULL::timestamp with time zone, p_to timestamp with time zone DEFAULT NULL::timestamp with time zone, p_exclude uuid[] DEFAULT '{}'::uuid[])
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v jsonb; f timestamptz := coalesce(p_from, '-infinity'); t timestamptz := coalesce(p_to, 'infinity');
  ex uuid[] := coalesce(p_exclude, '{}');
begin
  if not lms.has_role(auth.uid(), 'admin') then raise exception 'forbidden'; end if;
  with l as (select * from lms.vault_lots where not (student_user_id = any(ex))),
  lp as (select * from l where created_at >= f and created_at < t),
  vo as (select * from lms.vault_vouchers where not (student_user_id = any(ex)) and created_at >= f and created_at < t),
  used as (select vo.*, coalesce((select sum(amount) from l where l.source_type='cashback' and l.enrollment_id = vo.enrollment_id and l.state <> 'reversed'),0) cb from vo where status = 'used')
  select jsonb_build_object(
-- ... truncated: showing first 15 of 44 lines
```

#### lms.vault_reverse(p_source_key text)

```sql
CREATE OR REPLACE FUNCTION lms.vault_reverse(p_source_key text)
 RETURNS lms.vault_lots
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'lms', 'public', 'pg_temp'
AS $function$
declare v lms.vault_lots; v_spent numeric; v_rev numeric;
begin
  select * into v from lms.vault_lots where source_key = p_source_key for update;
  if not found or v.state in ('reversed','cancelled','expired') then return v; end if;
  if v.state = 'pending' then
    update lms.vault_lots set state = 'cancelled', remaining = 0 where id = v.id returning * into v;
    return v;
  end if;
  v_spent := greatest(v.amount - v.remaining, 0);
  v_rev := v.remaining;
  update lms.vault_lots set state = 'reversed', remaining = 0 where id = v.id returning * into v;
  if v_rev > 0 then
    insert into lms.vault_ledger (student_user_id, wallet, lot_id, kind, amount, ref_type, ref_id, description)
      values (v.student_user_id, v.wallet, v.id, 'reversal', -v_rev, v.source_type, v.source_key, 'Reversed: ' || v.source_type);
  end if;
  if v_spent > 0 then
    insert into lms.vault_review_flags (student_user_id, kind, detail)
      values (v.student_user_id, 'reversal_shortfall', jsonb_build_object('lot_id', v.id, 'source_key', v.source_key, 'spent', v_spent));
  end if;
  return v;
end $function$

```

## portal (53 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| align_custom_response_metadata | _data jsonb, _form_id uuid, _slug text, _schema jsonb | jsonb | sql | invoker | stable | search_path=pg_catalog |  |
| camp_intake_notify |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public, extensions |  |
| carries_valid_dial | _digits text | boolean | sql | invoker | immutable | search_path=portal, public | anon authenticated |
| check_seat_capacity | _application_id uuid | jsonb | plpgsql | DEFINER | stable | search_path=portal, public | anon authenticated |
| claim_paid_application | _application_id uuid, _payment_intent_id text, _amount numeric, _currency text, _payment_mode text | jsonb | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| cohort_code_start | _code text | date | sql | DEFINER | stable | search_path=portal, public | anon authenticated |
| custom_form_align_history |  | trigger | plpgsql | DEFINER | volatile | search_path=pg_catalog |  |
| custom_response_current_identity |  | trigger | plpgsql | DEFINER | volatile | search_path=pg_catalog |  |
| dial_from_country | _country text | text | sql | invoker | immutable | search_path=portal, public | anon authenticated |
| dial_lengths | _dial text | integer[] | sql | invoker | immutable | search_path=portal, public | anon authenticated |
| edit_program_code | p_id uuid, p_expected_code text, p_values jsonb | jsonb | plpgsql | invoker | volatile | search_path=pg_catalog, portal |  |
| generate_student_id | _year_digits text | text | plpgsql | invoker | volatile | search_path=portal, public |  |
| get_program_seat_status | _form_template_id uuid | jsonb | plpgsql | DEFINER | stable | search_path=portal, public | anon authenticated |
| get_public_form_template | _id uuid, _slug text, _program_code text | TABLE(id uuid, name text, description text, slug text, program_code text, program_name_override text, is_active boolean, schema jsonb, created_at timestamp with time zone, updated_at timestamp with time zone, settings jsonb, has_payment boolean) | sql | DEFINER | stable | search_path=portal, public | anon authenticated |
| get_public_payment_link | _slug text | TABLE(id uuid, name text, description text, slug text, program_code text, program_name_override text, is_active boolean, schema jsonb, created_at timestamp with time zone, updated_at timestamp with time zone, settings jsonb, has_payment boolean, payment_link_id uuid, payment_link_slug text, payment_link_name text, payment_link_note text, apply_url text) | sql | DEFINER | stable | search_path=portal, public | anon authenticated |
| get_student_school_id | user_id uuid | uuid | sql | DEFINER | stable | search_path=portal, public | authenticated |
| get_user_role | _user_id uuid | portal.app_role | sql | DEFINER | stable | search_path=portal, public | authenticated |
| get_user_school_name | _user_id uuid | text | sql | DEFINER | stable | search_path=portal, public | authenticated |
| has_role | _user_id uuid, _role text | boolean | sql | DEFINER | stable | search_path=portal, public | authenticated |
| import_whatsapp_undeliverable | p_rows jsonb, p_source_file text, p_actor uuid | jsonb | plpgsql | invoker | volatile | search_path=pg_catalog, portal |  |
| is_admin | user_id uuid | boolean | sql | DEFINER | stable | search_path=portal, public | authenticated |
| is_partner_school_admin | user_id uuid | boolean | sql | DEFINER | stable | search_path=portal, public | authenticated |
| is_lwl_admin | user_id uuid | boolean | sql | DEFINER | stable | search_path=portal, public | authenticated |
| is_manager | user_id uuid | boolean | sql | DEFINER | stable | search_path=portal, public | authenticated |
| is_school_admin | user_id uuid | boolean | sql | DEFINER | stable | search_path=portal, public | authenticated |
| is_student | user_id uuid | boolean | sql | DEFINER | stable | search_path=portal, public | authenticated |
| link_untracked_payments_for_application | _app_id uuid | integer | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| lwl_category_key | cat text | text | sql | invoker | immutable | search_path=public | anon authenticated |
| lwl_norm_country | v text | text | plpgsql | invoker | immutable | search_path=public | anon authenticated |
| lwl_norm_grade | g text | text | plpgsql | invoker | immutable | search_path=public | anon authenticated |
| normalize_phone | _raw text, _country text | text | plpgsql | invoker | immutable | search_path=portal, public | anon authenticated |
| notify_wati_payment_received |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| payment_link_base_url | _link_id uuid | text | sql | DEFINER | stable | search_path=portal, public |  |
| payment_links_cohort_guard |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| payment_links_count_use |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| payment_links_guard |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| payment_links_touch |  | trigger | plpgsql | invoker | volatile | search_path=portal, public |  |
| phone_fits | _dial text, _national text | boolean | plpgsql | invoker | immutable | search_path=portal, public | anon authenticated |
| prevent_profile_privilege_escalation |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| purge_captured_leads_for_email | _email text | integer | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| rl_hit | _key text, _limit integer, _window_seconds integer | boolean | plpgsql | DEFINER | volatile | search_path=portal |  |
| set_whatsapp_undeliverable_status | p_phone text, p_blocked boolean, p_reason text, p_actor uuid | boolean | plpgsql | invoker | volatile | search_path=pg_catalog, portal |  |
| sync_profile_to_applications |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| tg_apply_internal_tag |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| tg_default_utm |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| tg_lwl_normalize_formats |  | trigger | plpgsql | invoker | volatile | search_path=public |  |
| tg_normalize_captured_lead_phones |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| tg_normalize_lwl_application_phones |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| tg_normalize_school_inquiry_phones |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| tg_normalize_user_profile_phones |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| tg_purge_leads_for_application |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| update_updated_at_column |  | trigger | plpgsql | DEFINER | volatile | search_path=portal, public |  |
| upsert_application | _payload jsonb | TABLE(student_id text, row_id uuid, submission_token uuid, was_update boolean, payment_status text) | plpgsql | DEFINER | volatile | search_path=portal, public |  |

### portal: bodies

#### portal.align_custom_response_metadata(_data jsonb, _form_id uuid, _slug text, _schema jsonb)

```sql
CREATE OR REPLACE FUNCTION portal.align_custom_response_metadata(_data jsonb, _form_id uuid, _slug text, _schema jsonb)
 RETURNS jsonb
 LANGUAGE sql
 STABLE
 SET search_path TO 'pg_catalog'
AS $function$
SELECT coalesce(_data,'{}'::jsonb) || jsonb_build_object('form_id',_form_id::text,'form_slug',_slug) ||
CASE WHEN jsonb_typeof(_data->'answers')='array' THEN jsonb_build_object('answers',(
 SELECT coalesce(jsonb_agg(a.value || coalesce((SELECT jsonb_build_object('question',coalesce(nullif(q->>'column',''),q->>'label'),'section',coalesce(nullif(q->>'section',''),'General')) FROM jsonb_array_elements(CASE WHEN jsonb_typeof(_schema)='array' THEN _schema ELSE '[]'::jsonb END) q WHERE nullif(a.value->>'key','') IS NOT NULL AND q->>'key'=a.value->>'key' LIMIT 1),'{}'::jsonb) ORDER BY a.ordinality),'[]'::jsonb)
 FROM jsonb_array_elements(_data->'answers') WITH ORDINALITY a(value,ordinality)
)) ELSE '{}'::jsonb END;
$function$

```

#### portal.camp_intake_notify()

```sql
CREATE OR REPLACE FUNCTION portal.camp_intake_notify()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public', 'extensions'
AS $function$
begin
  perform net.http_post(
    url := 'https://osgnggxacjvsxwxsoqjr.supabase.co/functions/v1/cron-camp-intake-import',
    headers := jsonb_build_object('Content-Type','application/json','x-cron-secret', '<redacted-hex>'),
    body := jsonb_build_object('action','run','source', TG_TABLE_NAME)
  );
  return null;
exception when others then
  -- never block a Google Form insert: the hourly cron is the safety net
  return null;
end;
$function$

```

#### portal.carries_valid_dial(_digits text)

```sql
CREATE OR REPLACE FUNCTION portal.carries_valid_dial(_digits text)
 RETURNS boolean
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'portal', 'public'
AS $function$
  SELECT EXISTS (
    SELECT 1 FROM (VALUES
      ('1'),('7'),('20'),('27'),('30'),('31'),('33'),('39'),('44'),('49'),('60'),('61'),('62'),
      ('63'),('65'),('66'),('81'),('82'),('84'),('86'),('90'),('91'),('92'),('94'),('234'),
      ('254'),('880'),('966'),('968'),('971'),('973'),('974'),('965'),('977'),('852')
    ) AS v(dial)
    WHERE left(_digits, length(v.dial)) = v.dial
      AND (length(_digits) - length(v.dial)) = ANY (portal.dial_lengths(v.dial))
  )
$function$

```

#### portal.check_seat_capacity(_application_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.check_seat_capacity(_application_id uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
declare
  app portal.applications%rowtype;
  s jsonb;
  total_cap integer;
  gender_cap integer;
  paid_total integer;
  paid_gender integer;
  g text;
  gender_key text;
-- ... truncated: showing first 15 of 57 lines
```

#### portal.claim_paid_application(_application_id uuid, _payment_intent_id text, _amount numeric, _currency text, _payment_mode text)

```sql
CREATE OR REPLACE FUNCTION portal.claim_paid_application(_application_id uuid, _payment_intent_id text, _amount numeric, _currency text, _payment_mode text DEFAULT 'full'::text)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
declare
  app portal.applications%rowtype;
  s jsonb;
  lock_key bigint;
  total_cap integer;
  gender_cap integer;
  paid_total integer;
  paid_gender integer;
  g text;
-- ... truncated: showing first 15 of 76 lines
```

#### portal.cohort_code_start(_code text)

```sql
CREATE OR REPLACE FUNCTION portal.cohort_code_start(_code text)
 RETURNS date
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
  select coalesce(
    (select c.start_date::date from public.all_cohorts c
      where lower(c.program_code) = lower(btrim(_code)) and c.start_date is not null
      order by c.start_date limit 1),
    (select case when m[1] is null then null
                 else make_date(2000 + substr(m[1],5,2)::int, substr(m[1],3,2)::int, substr(m[1],1,2)::int) end
       from regexp_match(coalesce(_code,''), '(?:^|-)(\d{6})(?:-|$)') as m
      where substr(m[1],3,2)::int between 1 and 12 and substr(m[1],1,2)::int between 1 and 31)
  )
$function$

```

#### portal.custom_form_align_history()

```sql
CREATE OR REPLACE FUNCTION portal.custom_form_align_history()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'pg_catalog'
AS $function$
BEGIN
 IF NEW.name IS DISTINCT FROM OLD.name OR NEW.slug IS DISTINCT FROM OLD.slug OR NEW.schema IS DISTINCT FROM OLD.schema THEN
  UPDATE public.form_responses r SET form_name=NEW.name,data=portal.align_custom_response_metadata(r.data,NEW.id,NEW.slug,NEW.schema)
  WHERE r.data->>'form_id'=NEW.id::text OR (nullif(r.data->>'form_id','') IS NULL AND r.data->>'form_slug'=OLD.slug);
 END IF;
 RETURN NEW;
END $function$

```

#### portal.custom_response_current_identity()

```sql
CREATE OR REPLACE FUNCTION portal.custom_response_current_identity()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'pg_catalog'
AS $function$
DECLARE f portal.legacy_form_templates%ROWTYPE;
BEGIN
 SELECT t.* INTO f FROM portal.legacy_form_templates t WHERE (NEW.data->>'form_id'=t.id::text) OR (nullif(NEW.data->>'form_id','') IS NULL AND NEW.data->>'form_slug'=t.slug) FOR SHARE;
 IF FOUND THEN NEW.form_name=f.name; NEW.data=portal.align_custom_response_metadata(NEW.data,f.id,f.slug,f.schema); END IF;
 RETURN NEW;
END $function$

```

#### portal.dial_from_country(_country text)

```sql
CREATE OR REPLACE FUNCTION portal.dial_from_country(_country text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'portal', 'public'
AS $function$
  SELECT v.dial
  FROM (VALUES ('AF','afghanistan','93'),('AL','albania','355'),('DZ','algeria','213'),('AS','american samoa','1'),('AD','andorra','376'),('AO','angola','244'),('AI','anguilla','1'),('AG','antigua and barbuda','1'),('AR','argentina','54'),('AM','armenia','374'),('AW','aruba','297'),('AU','australia','61'),('AT','austria','43'),('AZ','azerbaijan','994'),('BS','bahamas','1'),('BH','bahrain','973'),('BD','bangladesh','880'),('BB','barbados','1'),('BY','belarus','375'),('BE','belgium','32'),('BZ','belize','501'),('BJ','benin','229'),('BM','bermuda','1'),('BT','bhutan','975'),('BO','bolivia','591'),('BA','bosnia and herzegovina','387'),('BW','botswana','267'),('BR','brazil','55'),('IO','british indian ocean territory','246'),('VG','british virgin islands','1'),('BN','brunei','673'),('BG','bulgaria','359'),('BF','burkina faso','226'),('BI','burundi','257'),('KH','cambodia','855'),('CM','cameroon','237'),('CA','canada','1'),('CV','cape verde','238'),('KY','cayman islands','1'),('CF','central african republic','236'),('TD','chad','235'),('CL','chile','56'),('CN','china','86'),('CX','christmas island','61'),('CC','cocos islands','61'),('CO','colombia','57'),('KM','comoros','269'),('CK','cook islands','682'),('CR','costa rica','506'),('HR','croatia','385'),('CU','cuba','53'),('CW','curaçao','599'),('CY','cyprus','357'),('CZ','czech republic','420'),('CD','democratic republic of the congo','243'),('DK','denmark','45'),('DJ','djibouti','253'),('DM','dominica','1'),('DO','dominican republic','1'),('EC','ecuador','593'),('EG','egypt','20'),('SV','el salvador','503'),('GQ','equatorial guinea','240'),('ER','eritrea','291'),('EE','estonia','372'),('SZ','eswatini','268'),('ET','ethiopia','251'),('FK','falkland islands','500'),('FO','faroe islands','298'),('FJ','fiji','679'),('FI','finland','358'),('FR','france','33'),('GF','french guiana','594'),('PF','french polynesia','689'),('GA','gabon','241'),('GM','gambia','220'),('GE','georgia','995'),('DE','germany','49'),('GH','ghana','233'),('GI','gibraltar','350'),('GR','greece','30'),('GL','greenland','299'),('GD','grenada','1'),('GP','guadeloupe','590'),('GU','guam','1'),('GT','guatemala','502'),('GG','guernsey','44'),('GN','guinea','224'),('GW','guinea-bissau','245'),('GY','guyana','592'),('HT','haiti','509'),('HN','honduras','504'),('HK','hong kong','852'),('HU','hungary','36'),('IS','iceland','354'),('IN','india','91'),('ID','indonesia','62'),('IR','iran','98'),('IQ','iraq','964'),('IE','ireland','353'),('IM','isle of man','44'),('IL','israel','972'),('IT','italy','39'),('CI','ivory coast','225'),('JM','jamaica','1'),('JP','japan','81'),('JE','jersey','44'),('JO','jordan','962'),('KZ','kazakhstan','7'),('KE','kenya','254'),('KI','kiribati','686'),('XK','kosovo','383'),('KW','kuwait','965'),('KG','kyrgyzstan','996'),('LA','laos','856'),('LV','latvia','371'),('LB','lebanon','961'),('LS','lesotho','266'),('LR','liberia','231'),('LY','libya','218'),('LI','liechtenstein','423'),('LT','lithuania','370'),('LU','luxembourg','352'),('MO','macau','853'),('MG','madagascar','261'),('MW','malawi','265'),('MY','malaysia','60'),('MV','maldives','960'),('ML','mali','223'),('MT','malta','356'),('MH','marshall islands','692'),('MQ','martinique','596'),('MR','mauritania','222'),('MU','mauritius','230'),('YT','mayotte','262'),('MX','mexico','52'),('FM','micronesia','691'),('MD','moldova','373'),('MC','monaco','377'),('MN','mongolia','976'),('ME','montenegro','382'),('MS','montserrat','1'),('MA','morocco','212'),('MZ','mozambique','258'),('MM','myanmar','95'),('NA','namibia','264'),('NR','nauru','674'),('NP','nepal','977'),('NL','netherlands','31'),('NC','new caledonia','687'),('NZ','new zealand','64'),('NI','nicaragua','505'),('NE','niger','227'),('NG','nigeria','234'),('NU','niue','683'),('NF','norfolk island','672'),('KP','north korea','850'),('MK','north macedonia','389'),('MP','northern mariana islands','1'),('NO','norway','47'),('OM','oman','968'),('PK','pakistan','92'),('PW','palau','680'),('PS','palestine','970'),('PA','panama','507'),('PG','papua new guinea','675'),('PY','paraguay','595'),('PE','peru','51'),('PH','philippines','63'),('PL','poland','48'),('PT','portugal','351'),('PR','puerto rico','1'),('QA','qatar','974'),('CG','republic of the congo','242'),('RE','réunion','262'),('RO','romania','40'),('RU','russia','7'),('RW','rwanda','250'),('BL','saint barthélemy','590'),('SH','saint helena','290'),('KN','saint kitts and nevis','1'),('LC','saint lucia','1'),('MF','saint martin','590'),('PM','saint pierre and miquelon','508'),('VC','saint vincent and the grenadines','1'),('WS','samoa','685'),('SM','san marino','378'),('ST','são tomé and príncipe','239'),('SA','saudi arabia','966'),('SN','senegal','221'),('RS','serbia','381'),('SC','seychelles','248'),('SL','sierra leone','232'),('SG','singapore','65'),('SX','sint maarten','1'),('SK','slovakia','421'),('SI','slovenia','386'),('SB','solomon islands','677'),('SO','somalia','252'),('ZA','south africa','27'),('KR','south korea','82'),('SS','south sudan','211'),('ES','spain','34'),('LK','sri lanka','94'),('SD','sudan','249'),('SR','suriname','597'),('SE','sweden','46'),('CH','switzerland','41'),('SY','syria','963'),('TW','taiwan','886'),('TJ','tajikistan','992'),('TZ','tanzania','255'),('TH','thailand','66'),('TL','timor-leste','670'),('TG','togo','228'),('TK','tokelau','690'),('TO','tonga','676'),('TT','trinidad and tobago','1'),('TN','tunisia','216'),('TR','turkey','90'),('TM','turkmenistan','993'),('TC','turks and caicos islands','1'),('TV','tuvalu','688'),('UG','uganda','256'),('UA','ukraine','380'),('AE','united arab emirates','971'),('GB','united kingdom','44'),('US','united states','1'),('UY','uruguay','598'),('VI','u.s. virgin islands','1'),('UZ','uzbekistan','998'),('VU','vanuatu','678'),('VA','vatican city','39'),('VE','venezuela','58'),('VN','vietnam','84'),('WF','wallis and futuna','681'),('YE','yemen','967'),('ZM','zambia','260'),('ZW','zimbabwe','263')) AS v(iso, cname, dial)
  WHERE lower(btrim(coalesce(_country,''))) <> ''
    AND (lower(btrim(_country)) = v.cname OR lower(btrim(_country)) = lower(v.iso))
  LIMIT 1
$function$

```

#### portal.dial_lengths(_dial text)

```sql
CREATE OR REPLACE FUNCTION portal.dial_lengths(_dial text)
 RETURNS integer[]
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'portal', 'public'
AS $function$
  SELECT v.lens FROM (VALUES
    ('1', ARRAY[10]),('7', ARRAY[10]),('20', ARRAY[9,10]),('27', ARRAY[9]),('30', ARRAY[10]),
    ('31', ARRAY[9]),('33', ARRAY[9]),('39', ARRAY[9,10]),('44', ARRAY[10]),('49', ARRAY[10,11]),
    ('60', ARRAY[9,10]),('61', ARRAY[9]),('62', ARRAY[9,10,11]),('63', ARRAY[10]),('65', ARRAY[8]),
    ('66', ARRAY[9]),('81', ARRAY[10]),('82', ARRAY[9,10]),('84', ARRAY[9]),('86', ARRAY[11]),
    ('90', ARRAY[10]),('91', ARRAY[10]),('92', ARRAY[10]),('94', ARRAY[9]),('234', ARRAY[10]),
    ('254', ARRAY[9]),('880', ARRAY[10]),('966', ARRAY[9]),('968', ARRAY[8]),('971', ARRAY[9]),
    ('973', ARRAY[8]),('974', ARRAY[8]),('965', ARRAY[8]),('977', ARRAY[10]),('852', ARRAY[8])
  ) AS v(dial, lens) WHERE v.dial = _dial LIMIT 1
$function$

```

#### portal.edit_program_code(p_id uuid, p_expected_code text, p_values jsonb)

```sql
CREATE OR REPLACE FUNCTION portal.edit_program_code(p_id uuid, p_expected_code text, p_values jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SET search_path TO 'pg_catalog', 'portal'
AS $function$
DECLARE old finance.program_codes%ROWTYPE; edited finance.program_codes%ROWTYPE; c public.all_cohorts%ROWTYPE; f portal.form_templates%ROWTYPE; s jsonb; next_codes jsonb; next_start date; delta integer; cohort_count integer;
BEGIN
 SELECT * INTO old FROM finance.program_codes WHERE id=p_id FOR UPDATE;
 IF NOT FOUND THEN RAISE EXCEPTION 'Program code not found'; END IF;
 IF old.program_code IS DISTINCT FROM p_expected_code THEN RAISE EXCEPTION 'This code changed since it was opened. Refresh before saving.'; END IF;
 IF EXISTS(SELECT 1 FROM finance.program_codes WHERE upper(program_code)=upper(p_values->>'program_code') AND id<>p_id) THEN RAISE EXCEPTION 'This code already exists'; END IF;
 IF p_values->>'date' ~ '^\d{4}-\d{2}-\d{2}$' THEN next_start := (p_values->>'date')::date; END IF;
 SELECT count(*) INTO cohort_count FROM public.all_cohorts WHERE program_code=old.program_code;
 IF cohort_count>1 THEN RAISE EXCEPTION 'Multiple catalogue cohorts match this code. Review before editing.'; END IF;
 IF cohort_count=1 AND next_start IS NULL THEN RAISE EXCEPTION 'A linked dated cohort requires a calendar date.'; END IF;
 UPDATE finance.program_codes SET program_code=p_values->>'program_code',category=p_values->>'category',organisation=p_values->>'organisation',program=p_values->>'program',school=nullif(p_values->>'school',''),country=nullif(p_values->>'country',''),date=p_values->>'date' WHERE id=p_id RETURNING * INTO edited;
 FOR c IN SELECT * FROM public.all_cohorts WHERE program_code=old.program_code FOR UPDATE LOOP
   delta := CASE WHEN c.start_date IS NOT NULL THEN next_start-c.start_date ELSE 0 END;
   UPDATE public.all_cohorts SET program_code=edited.program_code,start_date=next_start,end_date=CASE WHEN end_date IS NULL THEN NULL ELSE end_date+delta END,application_deadline=CASE WHEN application_deadline IS NULL THEN NULL ELSE application_deadline+delta END,cohort_label=CASE WHEN delta<>0 THEN null ELSE cohort_label END WHERE id=c.id;
 END LOOP;
 IF old.program_code IS DISTINCT FROM edited.program_code THEN
   FOR f IN SELECT * FROM portal.form_templates WHERE program_code=old.program_code OR coalesce(settings->'program_codes','[]'::jsonb) ? old.program_code OR coalesce(settings->'program_code_ids','[]'::jsonb) ? p_id::text FOR UPDATE LOOP
     s := coalesce(f.settings,'{}'::jsonb);
     SELECT coalesce(jsonb_agg(CASE WHEN value=old.program_code THEN edited.program_code ELSE value END),'[]'::jsonb) INTO next_codes FROM jsonb_array_elements_text(CASE WHEN jsonb_typeof(s->'program_codes')='array' THEN s->'program_codes' ELSE jsonb_build_array(coalesce(f.program_code,edited.program_code)) END);
     s := jsonb_set(s,'{program_codes}',next_codes,true);
     IF NOT coalesce(s->'program_code_ids','[]'::jsonb) ? p_id::text THEN s := jsonb_set(s,'{program_code_ids}',coalesce(s->'program_code_ids','[]'::jsonb)||jsonb_build_array(p_id::text),true); END IF;
     UPDATE portal.form_templates SET settings=s,program_code=CASE WHEN program_code=old.program_code THEN edited.program_code ELSE program_code END WHERE id=f.id;
   END LOOP;
   IF EXISTS(SELECT 1 FROM portal.program_email_settings WHERE program_code=old.program_code) AND EXISTS(SELECT 1 FROM portal.program_email_settings WHERE program_code=edited.program_code) THEN RAISE EXCEPTION 'Both codes have email settings. Review before editing.'; END IF;
   UPDATE portal.program_email_settings SET program_code=edited.program_code WHERE program_code=old.program_code;
 END IF;
 RETURN to_jsonb(edited);
END; $function$

```

#### portal.generate_student_id(_year_digits text)

```sql
CREATE OR REPLACE FUNCTION portal.generate_student_id(_year_digits text DEFAULT NULL::text)
 RETURNS text
 LANGUAGE plpgsql
 SET search_path TO 'portal', 'public'
AS $function$
declare alphabet text := '23456789ABCDEFGHJKMNPQRSTUVWXYZ'; yy text; cand text; i int; attempts int := 0;
begin
  yy := coalesce(_year_digits, to_char(now() at time zone 'UTC','YY'));
  loop
    cand := 'S' || yy;
    for i in 1..7 loop cand := cand || substr(alphabet, 1 + floor(random()*31)::int, 1); end loop;
    if not public.student_id_in_use(cand) then return cand; end if;  -- unique across every platform
    attempts := attempts + 1; if attempts > 10 then raise exception 'Could not generate unique student_id after 10 attempts'; end if;
  end loop;
end $function$

```

#### portal.get_program_seat_status(_form_template_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.get_program_seat_status(_form_template_id uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
declare
  s jsonb;
  paid_total int := 0;
  paid_male int := 0;
  paid_female int := 0;
begin
  if _form_template_id is null then
    return jsonb_build_object('enabled', false);
  end if;
  select settings -> 'seats' into s from portal.form_templates where id = _form_template_id;
  if s is null or coalesce((s ->> 'enabled')::boolean, false) is not true then
    return jsonb_build_object('enabled', false);
  end if;
  select count(*),
         count(*) filter (where lower(coalesce(student_gender, '')) = 'male'),
         count(*) filter (where lower(coalesce(student_gender, '')) = 'female')
    into paid_total, paid_male, paid_female
  from portal.applications
  where form_template_id = _form_template_id
    and payment_status = 'paid';
  return jsonb_build_object(
    'enabled', true,
    'total_cap', nullif(s ->> 'total', '')::int,
    'total_used', paid_total,
    'male_cap', nullif(s -> 'by_gender' ->> 'male', '')::int,
    'male_used', paid_male,
    'female_cap', nullif(s -> 'by_gender' ->> 'female', '')::int,
    'female_used', paid_female
  );
end
$function$

```

#### portal.get_public_form_template(_id uuid, _slug text, _program_code text)

```sql
CREATE OR REPLACE FUNCTION portal.get_public_form_template(_id uuid DEFAULT NULL::uuid, _slug text DEFAULT NULL::text, _program_code text DEFAULT NULL::text)
 RETURNS TABLE(id uuid, name text, description text, slug text, program_code text, program_name_override text, is_active boolean, schema jsonb, created_at timestamp with time zone, updated_at timestamp with time zone, settings jsonb, has_payment boolean)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
  SELECT
    ft.id, ft.name, NULL::text AS description, ft.slug, ft.program_code,
    ft.program_name_override, ft.is_active, ft.schema, ft.created_at, ft.updated_at,
    COALESCE(ft.settings, '{}'::jsonb) - 'stripe_price_id' AS settings,
    (
      COALESCE(NULLIF(btrim(COALESCE((ft.settings -> 'full_price') ->> 'amount', '')), '')::numeric, 0) > 0
      OR (
        jsonb_typeof(ft.settings -> 'installments') = 'array'
        AND EXISTS (
          SELECT 1 FROM jsonb_array_elements(COALESCE(ft.settings -> 'installments', '[]'::jsonb)) r(value)
          WHERE COALESCE(NULLIF(btrim(COALESCE(r.value ->> 'amount', '')), '')::numeric, 0) > 0
        )
      )
    ) AS has_payment
  FROM portal.form_templates ft
  WHERE ft.is_active = true
    AND (_id IS NULL OR ft.id = _id)
    AND (_slug IS NULL OR ft.slug = _slug)
    AND (_program_code IS NULL OR ft.program_code = _program_code)
$function$

```

#### portal.get_public_payment_link(_slug text)

```sql
CREATE OR REPLACE FUNCTION portal.get_public_payment_link(_slug text)
 RETURNS TABLE(id uuid, name text, description text, slug text, program_code text, program_name_override text, is_active boolean, schema jsonb, created_at timestamp with time zone, updated_at timestamp with time zone, settings jsonb, has_payment boolean, payment_link_id uuid, payment_link_slug text, payment_link_name text, payment_link_note text, apply_url text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
  SELECT
    coalesce(ft.id, pl.id) as id,
    coalesce(ft.name, p.program_name, pl.name) as name,
    p.short_description as description,
    coalesce(ft.slug, p.pay_slug, pl.slug) as slug,
    coalesce(pl.cohort_code, pl.program_code, ft.program_code) as program_code,
    coalesce(ft.program_name_override, p.program_name) as program_name_override,
    true as is_active,
    coalesce(ft.schema, '{}'::jsonb) as schema,
-- ... truncated: showing first 15 of 49 lines
```

#### portal.get_student_school_id(user_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.get_student_school_id(user_id uuid)
 RETURNS uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
  SELECT NULL::uuid;
$function$

```

#### portal.get_user_role(_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.get_user_role(_user_id uuid)
 RETURNS portal.app_role
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
  SELECT role FROM portal.user_profiles WHERE user_id = _user_id LIMIT 1;
$function$

```

#### portal.get_user_school_name(_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.get_user_school_name(_user_id uuid)
 RETURNS text
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$ SELECT school_name FROM portal.user_profiles WHERE user_id=_user_id LIMIT 1; $function$

```

#### portal.has_role(_user_id uuid, _role text)

```sql
CREATE OR REPLACE FUNCTION portal.has_role(_user_id uuid, _role text)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$ SELECT EXISTS (SELECT 1 FROM portal.user_profiles p WHERE p.user_id=_user_id AND p.role::text=_role); $function$

```

#### portal.import_whatsapp_undeliverable(p_rows jsonb, p_source_file text, p_actor uuid)

```sql
CREATE OR REPLACE FUNCTION portal.import_whatsapp_undeliverable(p_rows jsonb, p_source_file text, p_actor uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 SET search_path TO 'pg_catalog', 'portal'
AS $function$
DECLARE added integer; existing integer; total integer;
BEGIN
  IF jsonb_typeof(p_rows) <> 'array' OR jsonb_array_length(p_rows) NOT BETWEEN 1 AND 50000 OR p_actor IS NULL
     OR length(p_source_file) NOT BETWEEN 1 AND 255 THEN RAISE EXCEPTION 'Invalid report'; END IF;
  PERFORM pg_advisory_xact_lock(hashtextextended('portal.whatsapp_undeliverable_numbers', 0));
  SELECT count(*) INTO added FROM jsonb_to_recordset(p_rows) AS r(phone text)
    WHERE NOT EXISTS (SELECT 1 FROM portal.whatsapp_undeliverable_numbers b WHERE b.phone = r.phone);
  SELECT count(*) INTO existing FROM jsonb_to_recordset(p_rows) AS r(phone text)
    WHERE EXISTS (SELECT 1 FROM portal.whatsapp_undeliverable_numbers b WHERE b.phone = r.phone);
  INSERT INTO portal.whatsapp_undeliverable_numbers AS b
    (phone, country_code, national_number, country, last_failure_date, times_failed, last_campaign, source_file, last_action_by)
  SELECT r.phone, r.country_code, r.national_number, r.country, r.last_failure_date, r.times_failed, r.last_campaign, p_source_file, p_actor
  FROM jsonb_to_recordset(p_rows) AS r(phone text, country_code text, national_number text, country text, last_failure_date date, times_failed integer, last_campaign text)
  ON CONFLICT (phone) DO UPDATE SET
    country_code = CASE WHEN EXCLUDED.last_failure_date >= b.last_failure_date THEN EXCLUDED.country_code ELSE b.country_code END,
    national_number = CASE WHEN EXCLUDED.last_failure_date >= b.last_failure_date THEN EXCLUDED.national_number ELSE b.national_number END,
    country = CASE WHEN EXCLUDED.last_failure_date >= b.last_failure_date THEN EXCLUDED.country ELSE b.country END,
    last_campaign = CASE WHEN EXCLUDED.last_failure_date >= b.last_failure_date THEN EXCLUDED.last_campaign ELSE b.last_campaign END,
    source_file = CASE WHEN EXCLUDED.last_failure_date >= b.last_failure_date THEN EXCLUDED.source_file ELSE b.source_file END,
    times_failed = greatest(b.times_failed, EXCLUDED.times_failed),
    last_failure_date = greatest(b.last_failure_date, EXCLUDED.last_failure_date),
    last_imported_at = now(),
    blocked = b.blocked OR (b.unblocked_at IS NOT NULL AND EXCLUDED.last_failure_date > (b.unblocked_at AT TIME ZONE 'UTC')::date);
  SELECT count(*) INTO total FROM portal.whatsapp_undeliverable_numbers;
  RETURN jsonb_build_object('inserted', added, 'updated', existing, 'total', total);
END $function$

```

#### portal.is_admin(user_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.is_admin(user_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$ SELECT EXISTS (SELECT 1 FROM portal.user_profiles p WHERE p.user_id = is_admin.user_id AND p.role::text = 'admin'); $function$

```

#### portal.is_partner_school_admin(user_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.is_partner_school_admin(user_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$ SELECT EXISTS (SELECT 1 FROM portal.user_profiles p WHERE p.user_id = is_partner_school_admin.user_id AND p.role::text = 'partner_school_admin'); $function$

```

#### portal.is_lwl_admin(user_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.is_lwl_admin(user_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$ SELECT EXISTS (SELECT 1 FROM portal.user_profiles p WHERE p.user_id = is_lwl_admin.user_id AND p.role::text = 'lwl_admin'); $function$

```

#### portal.is_manager(user_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.is_manager(user_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$ SELECT EXISTS (SELECT 1 FROM portal.user_profiles p WHERE p.user_id = is_manager.user_id AND p.role::text = 'manager'); $function$

```

#### portal.is_school_admin(user_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.is_school_admin(user_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$ SELECT EXISTS (SELECT 1 FROM portal.user_profiles p WHERE p.user_id = is_school_admin.user_id AND p.role::text = 'school_admin'); $function$

```

#### portal.is_student(user_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.is_student(user_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$ SELECT EXISTS (SELECT 1 FROM portal.user_profiles p WHERE p.user_id = is_student.user_id AND p.role::text = 'student'); $function$

```

#### portal.link_untracked_payments_for_application(_app_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.link_untracked_payments_for_application(_app_id uuid)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
BEGIN
  -- The untracked_payments table was retired; Stripe reconciliation now happens
  -- in the stripe-webhook edge function. Kept as a no-op so callers stay valid.
  RETURN 0;
END; $function$

```

#### portal.lwl_category_key(cat text)

```sql
CREATE OR REPLACE FUNCTION portal.lwl_category_key(cat text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'public'
AS $function$
  SELECT CASE cat WHEN 'Camps' THEN 'camp' WHEN 'Competitions' THEN 'competition' WHEN 'Conferences' THEN 'conference'
    WHEN 'In Person' THEN 'in-person' WHEN 'Certificate Programs' THEN 'certificate' WHEN 'Online Programs' THEN 'certificate'
    WHEN 'Fellowships' THEN 'fellowship' END $function$

```

#### portal.lwl_norm_country(v text)

```sql
CREATE OR REPLACE FUNCTION portal.lwl_norm_country(v text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'public'
AS $function$
DECLARE c text := btrim(regexp_replace(coalesce(v,''),'\s+',' ','g')); l text;
BEGIN
  IF c='' THEN RETURN NULL; END IF;
  l := lower(c);
  RETURN CASE
    WHEN l IN ('uae','u.a.e','u.a.e.','united arab emirates','untied arab emirates','united arab of emirates','emirates','dubai','abu dhabi','sharjah') THEN 'United Arab Emirates'
    WHEN l IN ('usa','u.s.a','u.s.a.','u.s','u.s.','us','america','united states','united states of america','dallas') THEN 'United States'
    WHEN l IN ('uk','u.k.','united kingdom','england','great britain') THEN 'United Kingdom'
    WHEN l IN ('saudi','saudi arabia','saudia arabia','saaudi arabia','ksa','riyadh','jeddah','المملكة العربية السعودية') THEN 'Saudi Arabia'
    WHEN l IN ('korea','south korea','republic of korea') THEN 'South Korea'
    WHEN l IN ('台灣','taiwan') THEN 'Taiwan'
    WHEN l IN ('hong kong','hong kong s.a.r.','hong kong sar') THEN 'Hong Kong'
    WHEN l IN ('ivory coast','cote d''ivoire','côte d''ivoire','cote d''ivoire (ivory coast)') THEN 'Cote D''Ivoire (Ivory Coast)'
    WHEN l IN ('the bahamas','bahamas') THEN 'Bahamas'
    WHEN l IN ('cameroun','cameroon') THEN 'Cameroon'
    WHEN l IN ('mauritius island','mauritius') THEN 'Mauritius'
    WHEN l IN ('узбекистан','uzbekistan') THEN 'Uzbekistan'
    WHEN l LIKE 'honduras,%' THEN 'Honduras'
    WHEN l LIKE 'bangladesh,%' THEN 'Bangladesh'
    WHEN c = upper(c) OR c = lower(c) THEN initcap(l)
    ELSE c END;
END $function$

```

#### portal.lwl_norm_grade(g text)

```sql
CREATE OR REPLACE FUNCTION portal.lwl_norm_grade(g text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'public'
AS $function$
DECLARE c text := btrim(coalesce(g,'')); n int;
BEGIN
  IF c='' THEN RETURN NULL; END IF;
  IF c ~* '^year\s*\d{1,2}$' THEN n := substring(c from '\d+')::int - 1; RETURN CASE WHEN n BETWEEN 1 AND 12 THEN n::text ELSE c END; END IF;
  IF c ~* '^(grade\s*)?\d{1,2}(st|nd|rd|th)?(\s*grade)?$' THEN n := substring(c from '\d+')::int; IF n BETWEEN 1 AND 12 THEN RETURN n::text; END IF; END IF;
  IF lower(c)='other' THEN RETURN 'Other'; END IF;
  RETURN c;
END $function$

```

#### portal.normalize_phone(_raw text, _country text)

```sql
CREATE OR REPLACE FUNCTION portal.normalize_phone(_raw text, _country text DEFAULT NULL::text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'portal', 'public'
AS $function$
DECLARE
  s text := btrim(coalesce(_raw, ''));
  d text; cc text; lens int[]; cand text; rest text; n text; guard int := 0;
BEGIN
  IF s = '' THEN RETURN NULL; END IF;
  IF s ~ '^00[0-9]' THEN s := substr(s, 3); END IF;
  d := regexp_replace(s, '\D', '', 'g');
  IF d = '' THEN RETURN NULL; END IF;
  cc := portal.dial_from_country(_country);
-- ... truncated: showing first 15 of 51 lines
```

#### portal.notify_wati_payment_received()

```sql
CREATE OR REPLACE FUNCTION portal.notify_wati_payment_received()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
DECLARE
  _payload jsonb;
BEGIN
  IF NEW.payment_status = 'paid'
     AND (TG_OP = 'INSERT' OR OLD.payment_status IS DISTINCT FROM 'paid') THEN
    _payload := jsonb_build_object(
      'flow', 'payment',
      'application_id', NEW.id,
      'student_name', COALESCE(NEW.student_name, ''),
      'student_phone', COALESCE(NEW.student_mobile, ''),
      'student_email', COALESCE(NEW.student_email, ''),
      'student_country', COALESCE(NEW.country, ''),
      'parent_name', COALESCE(NEW.parent_name, ''),
      'parent_phone', COALESCE(NEW.parent_phone, ''),
      'parent_email', COALESCE(NEW.parent_email, ''),
      'parent_country', COALESCE(NEW.country, ''),
      'country', COALESCE(NEW.country, ''),
      'program_code', COALESCE(NEW.program_code, ''),
      'program_name', COALESCE(NEW.program_name, '')
    );
    PERFORM net.http_post(
      url := 'https://osgnggxacjvsxwxsoqjr.supabase.co/functions/v1/send-wati-welcome',
      headers := jsonb_build_object(
        'Content-Type', 'application/json',
        'apikey', '<redacted-jwt>',
        'Authorization', 'Bearer <redacted>'
      ),
      body := _payload
    );
  END IF;
  RETURN NEW;
END; $function$

```

#### portal.payment_link_base_url(_link_id uuid)

```sql
CREATE OR REPLACE FUNCTION portal.payment_link_base_url(_link_id uuid)
 RETURNS text
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
  select rtrim(pr.website, '/')
  from portal.payment_links pl
  left join public.all_partners pr on pr.id = pl.partner_id
  where pl.id = _link_id
  limit 1
$function$

```

#### portal.payment_links_cohort_guard()

```sql
CREATE OR REPLACE FUNCTION portal.payment_links_cohort_guard()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
begin
  new.deal_type := lower(coalesce(nullif(btrim(new.deal_type), ''), 'b2c'));
  new.cohort_code := nullif(btrim(coalesce(new.cohort_code, '')), '');
  if new.cohort_code is not null then
    if new.program_id is null then
      raise exception 'A locked start date needs a catalogue programme on the link';
    end if;
    if not exists (
      select 1 from public.all_cohorts c
      where c.program_id = new.program_id and lower(c.program_code) = lower(new.cohort_code)
    ) then
      raise exception 'Start date % does not belong to this programme', new.cohort_code;
    end if;
    select c.program_code into new.cohort_code from public.all_cohorts c
      where c.program_id = new.program_id and lower(c.program_code) = lower(new.cohort_code) limit 1;
    new.program_code := new.cohort_code;
  end if;
  return new;
end $function$

```

#### portal.payment_links_count_use()

```sql
CREATE OR REPLACE FUNCTION portal.payment_links_count_use()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
BEGIN
  IF NEW.payment_link_id IS NOT NULL
     AND NEW.payment_status = 'paid'
     AND COALESCE(OLD.payment_status,'') <> 'paid' THEN
    UPDATE portal.payment_links SET uses_count = uses_count + 1 WHERE id = NEW.payment_link_id;
  END IF;
  RETURN NEW;
END; $function$

```

#### portal.payment_links_guard()

```sql
CREATE OR REPLACE FUNCTION portal.payment_links_guard()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
declare
  v_slug text := lower(btrim(new.slug));
  v_prog record;
begin
  if v_slug is null or length(v_slug) < 3 then
    raise exception 'Link slug must be at least 3 characters';
  end if;
  if v_slug !~ '^[a-z0-9][a-z0-9-]*[a-z0-9]$' then
    raise exception 'Link slug may only contain lowercase letters, numbers and hyphens';
-- ... truncated: showing first 15 of 51 lines
```

#### portal.payment_links_touch()

```sql
CREATE OR REPLACE FUNCTION portal.payment_links_touch()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'portal', 'public'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END; $function$

```

#### portal.phone_fits(_dial text, _national text)

```sql
CREATE OR REPLACE FUNCTION portal.phone_fits(_dial text, _national text)
 RETURNS boolean
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'portal', 'public'
AS $function$
DECLARE lens int[] := portal.dial_lengths(_dial);
BEGIN
  IF lens IS NULL THEN
    RETURN length(_national) BETWEEN 6 AND 12;
  END IF;
  RETURN length(_national) = ANY (lens);
END; $function$

```

#### portal.prevent_profile_privilege_escalation()

```sql
CREATE OR REPLACE FUNCTION portal.prevent_profile_privilege_escalation()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
DECLARE
  _uid uuid := auth.uid();
  _is_admin boolean := false;
BEGIN
  IF _uid IS NULL THEN RETURN NEW; END IF;
  _is_admin := portal.is_admin(_uid) OR portal.is_lwl_admin(_uid);
  IF NEW.school_name IS DISTINCT FROM OLD.school_name AND NOT _is_admin THEN
    RAISE EXCEPTION 'school_name change not permitted';
  END IF;
  IF NEW.role IS DISTINCT FROM OLD.role AND NOT _is_admin THEN
    RAISE EXCEPTION 'role change not permitted';
  END IF;
  RETURN NEW;
END; $function$

```

#### portal.purge_captured_leads_for_email(_email text)

```sql
CREATE OR REPLACE FUNCTION portal.purge_captured_leads_for_email(_email text)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
DECLARE
  _e text := lower(trim(_email));
  _n integer;
BEGIN
  IF _e IS NULL OR _e = '' THEN RETURN 0; END IF;
  DELETE FROM portal.captured_leads
   WHERE lead_type = 'form' AND converted = false
     AND (lower(student_email) = _e OR lower(parent_email) = _e);
  GET DIAGNOSTICS _n = ROW_COUNT;
  RETURN _n;
END; $function$

```

#### portal.rl_hit(_key text, _limit integer, _window_seconds integer)

```sql
CREATE OR REPLACE FUNCTION portal.rl_hit(_key text, _limit integer, _window_seconds integer)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal'
AS $function$
DECLARE
  used INT;
BEGIN
  DELETE FROM portal.rate_limit_hits WHERE created_at < now() - INTERVAL '1 day';
  INSERT INTO portal.rate_limit_hits (key) VALUES (_key);
  SELECT count(*) INTO used
  FROM portal.rate_limit_hits
  WHERE key = _key AND created_at > now() - make_interval(secs => _window_seconds);
  RETURN used <= _limit;
END;
$function$

```

#### portal.set_whatsapp_undeliverable_status(p_phone text, p_blocked boolean, p_reason text, p_actor uuid)

```sql
CREATE OR REPLACE FUNCTION portal.set_whatsapp_undeliverable_status(p_phone text, p_blocked boolean, p_reason text, p_actor uuid)
 RETURNS boolean
 LANGUAGE plpgsql
 SET search_path TO 'pg_catalog', 'portal'
AS $function$
BEGIN
  IF p_phone !~ '^[0-9]{1,32}$' OR p_blocked IS NULL OR p_actor IS NULL OR length(trim(p_reason)) NOT BETWEEN 3 AND 500 THEN RAISE EXCEPTION 'Invalid change'; END IF;
  PERFORM pg_advisory_xact_lock(hashtextextended('portal.whatsapp_undeliverable_numbers', 0));
  UPDATE portal.whatsapp_undeliverable_numbers SET blocked = p_blocked,
    unblocked_at = CASE WHEN NOT p_blocked THEN now() ELSE unblocked_at END,
    unblocked_by = CASE WHEN NOT p_blocked THEN p_actor ELSE unblocked_by END,
    unblock_reason = CASE WHEN NOT p_blocked THEN trim(p_reason) ELSE unblock_reason END,
    last_action_at = now(), last_action_by = p_actor
  WHERE phone = p_phone;
  RETURN FOUND;
END $function$

```

#### portal.sync_profile_to_applications()

```sql
CREATE OR REPLACE FUNCTION portal.sync_profile_to_applications()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
DECLARE
  _full_name text;
BEGIN
  _full_name := TRIM(
    COALESCE(NEW.first_name, '') ||
    CASE WHEN COALESCE(NEW.middle_name, '') != '' THEN ' ' || NEW.middle_name ELSE '' END ||
    ' ' || COALESCE(NEW.last_name, '')
  );
  NEW.full_name := _full_name;
  IF NEW.email IS NOT NULL AND NEW.email <> '' THEN
    UPDATE portal.applications
       SET student_name = _full_name,
           school_name = COALESCE(NEW.school_name, applications.school_name),
           grade = COALESCE(NEW.grade_level, applications.grade)
     WHERE LOWER(student_email) = LOWER(NEW.email);
  END IF;
  RETURN NEW;
END; $function$

```

#### portal.tg_apply_internal_tag()

```sql
CREATE OR REPLACE FUNCTION portal.tg_apply_internal_tag()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
declare
  _tag text;
begin
  if new.admin_notes is not null and btrim(new.admin_notes) <> '' then
    return new;
  end if;
  if new.form_template_id is null then
    return new;
  end if;
  select settings->>'internal_tag' into _tag
  from portal.form_templates
  where id = new.form_template_id;
  if _tag is not null and btrim(_tag) <> '' then
    new.admin_notes := _tag;
  end if;
  return new;
end;
$function$

```

#### portal.tg_default_utm()

```sql
CREATE OR REPLACE FUNCTION portal.tg_default_utm()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
DECLARE
  ph text[] := array['','unknown','direct','website','none','null','undefined','(none)','(not set)','not set','n/a','-'];
BEGIN
  -- A real tag is never lost: on update a blank/placeholder value keeps the
  -- earlier real tag. Anything still missing is honestly 'unknown', per field.
  IF lower(btrim(coalesce(NEW.utm_source,''))) = ANY(ph) THEN
    NEW.utm_source := CASE WHEN TG_OP='UPDATE' AND NOT (lower(btrim(coalesce(OLD.utm_source,''))) = ANY(ph)) THEN OLD.utm_source ELSE 'unknown' END;
  END IF;
  IF lower(btrim(coalesce(NEW.utm_medium,''))) = ANY(ph) THEN
    NEW.utm_medium := CASE WHEN TG_OP='UPDATE' AND NOT (lower(btrim(coalesce(OLD.utm_medium,''))) = ANY(ph)) THEN OLD.utm_medium ELSE 'unknown' END;
  END IF;
  IF lower(btrim(coalesce(NEW.utm_campaign,''))) = ANY(ph) THEN
    NEW.utm_campaign := CASE WHEN TG_OP='UPDATE' AND NOT (lower(btrim(coalesce(OLD.utm_campaign,''))) = ANY(ph)) THEN OLD.utm_campaign ELSE 'unknown' END;
  END IF;
  NEW.utm_source := btrim(NEW.utm_source); NEW.utm_medium := btrim(NEW.utm_medium); NEW.utm_campaign := btrim(NEW.utm_campaign);
  RETURN NEW;
END; $function$

```

#### portal.tg_lwl_normalize_formats()

```sql
CREATE OR REPLACE FUNCTION portal.tg_lwl_normalize_formats()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public'
AS $function$
DECLARE v_cat text; v_raw text;
BEGIN
  NEW.grade := portal.lwl_norm_grade(NEW.grade);
  NEW.country := portal.lwl_norm_country(NEW.country);
  NEW.student_email := nullif(lower(btrim(NEW.student_email)),'');
  NEW.parent_email := nullif(lower(btrim(NEW.parent_email)),'');
  NEW.counsellor_email := nullif(lower(btrim(NEW.counsellor_email)),'');
  NEW.student_name := nullif(btrim(regexp_replace(NEW.student_name,'\s+',' ','g')),'');
  NEW.parent_name := nullif(btrim(regexp_replace(NEW.parent_name,'\s+',' ','g')),'');
  NEW.school_name := nullif(btrim(regexp_replace(NEW.school_name,'\s+',' ','g')),'');
-- ... truncated: showing first 15 of 43 lines
```

#### portal.tg_normalize_captured_lead_phones()

```sql
CREATE OR REPLACE FUNCTION portal.tg_normalize_captured_lead_phones()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
BEGIN
  NEW.student_phone := portal.normalize_phone(NEW.student_phone, NEW.country);
  NEW.parent_phone  := portal.normalize_phone(NEW.parent_phone, NEW.country);
  RETURN NEW;
END; $function$

```

#### portal.tg_normalize_lwl_application_phones()

```sql
CREATE OR REPLACE FUNCTION portal.tg_normalize_lwl_application_phones()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
BEGIN
  NEW.student_mobile := portal.normalize_phone(NEW.student_mobile, NEW.country);
  NEW.parent_phone   := portal.normalize_phone(NEW.parent_phone, NEW.country);
  RETURN NEW;
END; $function$

```

#### portal.tg_normalize_school_inquiry_phones()

```sql
CREATE OR REPLACE FUNCTION portal.tg_normalize_school_inquiry_phones()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
BEGIN
  NEW.phone := portal.normalize_phone(NEW.phone, NEW.country);
  RETURN NEW;
END; $function$

```

#### portal.tg_normalize_user_profile_phones()

```sql
CREATE OR REPLACE FUNCTION portal.tg_normalize_user_profile_phones()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
BEGIN
  NEW.phone_number := portal.normalize_phone(NEW.phone_number, NEW.country);
  NEW.parent_phone := portal.normalize_phone(NEW.parent_phone, NEW.country);
  RETURN NEW;
END; $function$

```

#### portal.tg_purge_leads_for_application()

```sql
CREATE OR REPLACE FUNCTION portal.tg_purge_leads_for_application()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
BEGIN
  PERFORM portal.purge_captured_leads_for_email(NEW.student_email);
  IF NEW.parent_email IS NOT NULL THEN
    PERFORM portal.purge_captured_leads_for_email(NEW.parent_email);
  END IF;
  RETURN NULL;
END; $function$

```

#### portal.update_updated_at_column()

```sql
CREATE OR REPLACE FUNCTION portal.update_updated_at_column()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END; $function$

```

#### portal.upsert_application(_payload jsonb)

```sql
CREATE OR REPLACE FUNCTION portal.upsert_application(_payload jsonb)
 RETURNS TABLE(student_id text, row_id uuid, submission_token uuid, was_update boolean, payment_status text)
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'portal', 'public'
AS $function$
DECLARE
  _email text := lower(trim(_payload->>'student_email'));
  _program_code text := _payload->>'program_code';
  _form_template_id uuid := NULLIF(_payload->>'form_template_id','')::uuid;
  _existing_app record;
  _existing_any_id text;
  _new_id text;
  _row_id uuid;
  _token uuid;
-- ... truncated: showing first 15 of 132 lines
```

## fellowship (13 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| all_cohorts_dml |  | trigger | plpgsql | DEFINER | volatile | search_path=fellowship, public |  |
| check_rate_limit | _bucket text, _key text, _limit integer, _window_seconds integer | boolean | plpgsql | DEFINER | volatile | search_path=fellowship |  |
| claim_promo_slot | _code text, _max integer, _holder text | integer | plpgsql | DEFINER | volatile | search_path=fellowship, public |  |
| country_dial_code | _country text | text | sql | invoker | immutable | search_path=fellowship, public | anon authenticated |
| known_dial_codes |  | text[] | sql | invoker | immutable | search_path=fellowship, public | anon authenticated |
| mark_promo_slot | _code text, _holder text | void | sql | DEFINER | volatile | search_path=fellowship, public |  |
| normalize_phone | _raw text, _country text | text | plpgsql | invoker | immutable | search_path=fellowship, public | anon authenticated |
| set_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=fellowship, pg_temp | anon authenticated |
| tg_normalize_std_application |  | trigger | plpgsql | invoker | volatile | search_path=fellowship, public | anon authenticated |
| tg_prefix_phone_applications |  | trigger | plpgsql | invoker | volatile | search_path=fellowship, public | anon authenticated |
| tg_prefix_phone_interest |  | trigger | plpgsql | invoker | volatile | search_path=fellowship, public | anon authenticated |
| tg_prefix_phone_masterclass |  | trigger | plpgsql | invoker | volatile | search_path=fellowship, public | anon authenticated |
| tg_prefix_phone_wwi |  | trigger | plpgsql | invoker | volatile | search_path=fellowship, public | anon authenticated |

### fellowship: bodies

#### fellowship.all_cohorts_dml()

```sql
CREATE OR REPLACE FUNCTION fellowship.all_cohorts_dml()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'fellowship', 'public'
AS $function$
declare
  v_cohort_id uuid; v_program_id uuid; v_key text; v_code text; v_prefix text;
begin
  if tg_op = 'DELETE' then
    delete from fellowship.cohort_settings where cohort_id = old.id;
    delete from public.all_cohorts where id = old.id;
    return old;
  end if;

-- ... truncated: showing first 15 of 70 lines
```

#### fellowship.check_rate_limit(_bucket text, _key text, _limit integer, _window_seconds integer)

```sql
CREATE OR REPLACE FUNCTION fellowship.check_rate_limit(_bucket text, _key text, _limit integer, _window_seconds integer)
 RETURNS boolean
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'fellowship'
AS $function$
declare _win timestamptz; _count integer;
begin
  _win := to_timestamp(floor(extract(epoch from now()) / _window_seconds) * _window_seconds);
  insert into fellowship.edge_rate_limits (bucket, key, window_start, count)
  values (_bucket, _key, _win, 1)
  on conflict (bucket, key, window_start)
  do update set count = fellowship.edge_rate_limits.count + 1
  returning count into _count;
  delete from fellowship.edge_rate_limits where created_at < now() - interval '1 day';
  return _count <= _limit;
end;
$function$

```

#### fellowship.claim_promo_slot(_code text, _max integer, _holder text)

```sql
CREATE OR REPLACE FUNCTION fellowship.claim_promo_slot(_code text, _max integer, _holder text)
 RETURNS integer
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'fellowship', 'public'
AS $function$
declare used int; c text := upper(trim(_code)); h text := lower(trim(_holder));
begin
  if c = '' or h = '' then return 0; end if;
  perform pg_advisory_xact_lock(hashtext('fellowship.promo:' || c));
  insert into fellowship.promo_slot_claims (code, holder) values (c, h)
    on conflict (code, holder) do update set created_at = now();
  select count(*) into used from fellowship.promo_slot_claims
    where code = c and succeeded = false and created_at > now() - interval '30 minutes';
  if _max is not null and used > _max then
    delete from fellowship.promo_slot_claims where code = c and holder = h and succeeded = false;
    return -1;
  end if;
  return used;
end; $function$

```

#### fellowship.country_dial_code(_country text)

```sql
CREATE OR REPLACE FUNCTION fellowship.country_dial_code(_country text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'fellowship', 'public'
AS $function$
  SELECT dial FROM (VALUES
    ('afghanistan','93'),('albania','355'),('algeria','213'),('angola','244'),('anguilla','1264'),
    ('argentina','54'),('armenia','374'),('australia','61'),('austria','43'),('azerbaijan','994'),
    ('bahrain','973'),('bangladesh','880'),('barbados','1246'),('belgium','32'),('bermuda','1441'),
    ('bolivia','591'),('botswana','267'),('brazil','55'),('brunei','673'),('bulgaria','359'),
    ('cambodia','855'),('cameroon','237'),('canada','1'),('chile','56'),('china','86'),
    ('colombia','57'),('costa rica','506'),('croatia','385'),('cyprus','357'),('czech republic','420'),
    ('denmark','45'),('dominican republic','1809'),('ecuador','593'),('egypt','20'),('estonia','372'),
    ('ethiopia','251'),('finland','358'),('france','33'),('gabon','241'),('georgia','995'),
-- ... truncated: showing first 15 of 48 lines
```

#### fellowship.known_dial_codes()

```sql
CREATE OR REPLACE FUNCTION fellowship.known_dial_codes()
 RETURNS text[]
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'fellowship', 'public'
AS $function$
  SELECT array_agg(dial ORDER BY length(dial) DESC, dial)
  FROM (
    SELECT DISTINCT fellowship.country_dial_code(name) AS dial
    FROM (VALUES
      ('india'),('united states'),('united kingdom'),('united arab emirates'),('saudi arabia'),
      ('qatar'),('oman'),('kuwait'),('bahrain'),('jordan'),('lebanon'),('egypt'),('morocco'),
      ('nigeria'),('kenya'),('ghana'),('south africa'),('tanzania'),('uganda'),('zambia'),
      ('zimbabwe'),('cameroon'),('angola'),('liberia'),('senegal'),('ethiopia'),('rwanda'),
      ('china'),('hong kong'),('taiwan'),('singapore'),('malaysia'),('indonesia'),('thailand'),
      ('vietnam'),('philippines'),('japan'),('south korea'),('pakistan'),('bangladesh'),
      ('sri lanka'),('nepal'),('maldives'),('australia'),('new zealand'),('canada'),('mexico'),
      ('brazil'),('chile'),('peru'),('colombia'),('argentina'),('panama'),('honduras'),
      ('guatemala'),('costa rica'),('ecuador'),('uruguay'),('venezuela'),('france'),('germany'),
      ('spain'),('italy'),('portugal'),('netherlands'),('belgium'),('switzerland'),('austria'),
      ('sweden'),('norway'),('denmark'),('finland'),('ireland'),('poland'),('romania'),('greece'),
      ('turkey'),('russia'),('kazakhstan'),('ukraine'),('israel'),('iraq'),('libya'),('sudan'),
      ('yemen'),('mauritius'),('malta'),('cyprus'),('armenia'),('georgia'),('azerbaijan'),
      ('kosovo'),('serbia'),('croatia'),('bulgaria'),('hungary'),('czech republic'),('slovakia'),
      ('slovenia'),('lithuania'),('latvia'),('estonia'),('luxembourg'),('iceland'),('botswana'),
      ('namibia'),('mozambique'),('malawi'),('myanmar'),('cambodia'),('brunei'),('macau'),
      ('mongolia'),('uzbekistan'),('moldova'),('palestine'),('bolivia'),('paraguay'),('tunisia'),
      ('algeria'),('afghanistan'),('albania'),('jamaica'),('barbados'),('bermuda'),('anguilla'),
      ('dominican republic'),('gabon'),('ivory coast')
    ) AS c(name)
  ) d
  WHERE dial IS NOT NULL
$function$

```

#### fellowship.mark_promo_slot(_code text, _holder text)

```sql
CREATE OR REPLACE FUNCTION fellowship.mark_promo_slot(_code text, _holder text)
 RETURNS void
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'fellowship', 'public'
AS $function$
  update fellowship.promo_slot_claims set succeeded = true
   where code = upper(trim(_code)) and holder = lower(trim(_holder));
$function$

```

#### fellowship.normalize_phone(_raw text, _country text)

```sql
CREATE OR REPLACE FUNCTION fellowship.normalize_phone(_raw text, _country text DEFAULT NULL::text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'fellowship', 'public'
AS $function$
DECLARE
  raw text;
  digits text;
  dial text;
  rest text;
  d text;
  is_intl boolean := false;
BEGIN
  raw := btrim(coalesce(_raw, ''));
-- ... truncated: showing first 15 of 58 lines
```

#### fellowship.set_updated_at()

```sql
CREATE OR REPLACE FUNCTION fellowship.set_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'fellowship', 'pg_temp'
AS $function$
begin new.updated_at = now(); return new; end $function$

```

#### fellowship.tg_normalize_std_application()

```sql
CREATE OR REPLACE FUNCTION fellowship.tg_normalize_std_application()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'fellowship', 'public'
AS $function$
DECLARE v_name text; v_label text;
BEGIN
  NEW.student_mobile := fellowship.normalize_phone(NEW.student_mobile, NEW.country);
  NEW.parent_phone   := fellowship.normalize_phone(NEW.parent_phone, NEW.country);
  NEW.student_email  := lower(btrim(NEW.student_email));
  NEW.parent_email   := NULLIF(lower(btrim(NEW.parent_email)), '');
  NEW.counsellor_email := NULLIF(lower(btrim(NEW.counsellor_email)), '');
  NEW.student_name   := regexp_replace(btrim(NEW.student_name), '\s+', ' ', 'g');
  NEW.scholarship_code := NULLIF(upper(btrim(NEW.scholarship_code)), '');
  NEW.currency       := upper(NEW.currency);
-- ... truncated: showing first 15 of 43 lines
```

#### fellowship.tg_prefix_phone_applications()

```sql
CREATE OR REPLACE FUNCTION fellowship.tg_prefix_phone_applications()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'fellowship', 'public'
AS $function$
BEGIN
  NEW.student_phone := fellowship.normalize_phone(NEW.student_phone, NEW.country);
  NEW.parent_phone  := COALESCE(fellowship.normalize_phone(NEW.parent_phone, NEW.country), '');
  IF NEW.utm_source IS NULL OR btrim(NEW.utm_source) = '' THEN NEW.utm_source := 'unknown'; END IF;
  IF NEW.utm_medium IS NULL OR btrim(NEW.utm_medium) = '' THEN NEW.utm_medium := 'unknown'; END IF;
  IF NEW.utm_campaign IS NULL OR btrim(NEW.utm_campaign) = '' THEN NEW.utm_campaign := 'unknown'; END IF;
  RETURN NEW;
END; $function$

```

#### fellowship.tg_prefix_phone_interest()

```sql
CREATE OR REPLACE FUNCTION fellowship.tg_prefix_phone_interest()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'fellowship', 'public'
AS $function$
BEGIN
  NEW.phone := fellowship.normalize_phone(NEW.phone, NULL);
  RETURN NEW;
END; $function$

```

#### fellowship.tg_prefix_phone_masterclass()

```sql
CREATE OR REPLACE FUNCTION fellowship.tg_prefix_phone_masterclass()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'fellowship', 'public'
AS $function$
BEGIN
  NEW.phone        := fellowship.normalize_phone(NEW.phone, NEW.country);
  NEW.parent_phone := fellowship.normalize_phone(NEW.parent_phone, NEW.country);
  RETURN NEW;
END; $function$

```

#### fellowship.tg_prefix_phone_wwi()

```sql
CREATE OR REPLACE FUNCTION fellowship.tg_prefix_phone_wwi()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'fellowship', 'public'
AS $function$
BEGIN
  NEW.phone := fellowship.normalize_phone(NEW.phone, NEW.country);
  RETURN NEW;
END; $function$

```

## public (45 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| admin_partner_contacts |  | TABLE(id uuid, org_name text, contact_name text, contact_phone text) | sql | DEFINER | stable | search_path=public | authenticated |
| current_verified_email |  | text | sql | DEFINER | stable | search_path=public, auth | authenticated |
| doctors_enforce_app_insert_defaults |  | trigger | plpgsql | DEFINER | volatile | search_path=public, premed |  |
| enforce_finlit_view_security_invoker |  | event_trigger | plpgsql | DEFINER | volatile | search_path=pg_catalog |  |
| enforce_update_permissions |  | trigger | plpgsql | DEFINER | volatile | search_path=public, pg_temp |  |
| execute_split | p_sm_ref text, p_splits jsonb | jsonb | plpgsql | invoker | volatile | search_path=public, pg_temp | anon authenticated |
| fn_assign_sr_no |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |
| fn_renumber_sr_no |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |
| get_or_create_student_id | _email text, _mobile text, _name text | text | plpgsql | DEFINER | volatile | search_path=public |  |
| handle_new_user |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| handle_updated_at |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| has_role | _user_id uuid, _role app_role | boolean | sql | DEFINER | stable | search_path=public | authenticated |
| is_finance_admin | _uid uuid | boolean | sql | DEFINER | stable | search_path=finance, public | authenticated |
| legacy_question_form_data | _q text[], _tpl uuid, _code text, _existing jsonb | jsonb | plpgsql | invoker | stable | search_path=public, portal | anon authenticated |
| log_lead_activity |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| lwl_form_touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=public, extensions, pg_temp | anon authenticated |
| merge_form_answers | _existing jsonb, _incoming jsonb | jsonb | sql | invoker | immutable | search_path=public, extensions, pg_temp | anon authenticated |
| mirror_legacy_application |  | trigger | plpgsql | DEFINER | volatile | search_path=public, portal |  |
| normalize_grade_level | input text | text | plpgsql | invoker | immutable | search_path=public, pg_temp | anon authenticated |
| normalize_place_name | input text | text | plpgsql | invoker | immutable | search_path=public, pg_temp | anon authenticated |
| normalize_teo_application_fields |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |
| onward_apps_set_cohort_id |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| outreach_data_normalize_email |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |
| outreach_data_touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |
| promo_sync_redemption_count |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| promo_touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=public |  |
| razorpay_payments_touch_synced_at |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |
| research_role | _user_id uuid | text | sql | DEFINER | stable | search_path=public | authenticated |
| resolve_student_id | _email text, _mobile text, _name text | text | plpgsql | DEFINER | stable | search_path=public |  |
| rls_auto_enable |  | event_trigger | plpgsql | DEFINER | volatile | search_path=pg_catalog |  |
| set_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |
| student_id_in_use | _id text | boolean | plpgsql | DEFINER | stable | search_path=public |  |
| student_name_tokens | _n text | text[] | sql | invoker | immutable | search_path=public, extensions, pg_temp | anon authenticated |
| student_names_match | _a text, _b text | boolean | sql | invoker | immutable | search_path=public, extensions, pg_temp | anon authenticated |
| tg_edge_shared_student_id |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| tg_lms_vault_promo_redeemed |  | trigger | plpgsql | DEFINER | volatile | search_path=public, lms, pg_temp |  |
| tg_lwl_shared_student_id |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| tg_protect_created_at |  | trigger | plpgsql | invoker | volatile | search_path=public, extensions, pg_temp | anon authenticated |
| tg_purpose_shared_student_id |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| tg_sirluther_shared_student_id |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| tg_teo_shared_student_id |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| touch_testprep_masterclass_regs_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=public |  |
| touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |
| update_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |
| update_updated_at_column |  | trigger | plpgsql | invoker | volatile | search_path=public, pg_temp |  |

### public: bodies

#### public.admin_partner_contacts()

```sql
CREATE OR REPLACE FUNCTION public.admin_partner_contacts()
 RETURNS TABLE(id uuid, org_name text, contact_name text, contact_phone text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
  select p.id, p.org_name::text, p.contact_name::text, p.contact_phone::text from public.all_partners p
  where lms.has_role(auth.uid(),'admin') or public.has_role(auth.uid(),'admin')
$function$

```

#### public.current_verified_email()

```sql
CREATE OR REPLACE FUNCTION public.current_verified_email()
 RETURNS text
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'public', 'auth'
AS $function$
  SELECT lower(u.email) FROM auth.users u
  WHERE u.id = auth.uid() AND u.email_confirmed_at IS NOT NULL AND u.email IS NOT NULL
$function$

```

#### public.doctors_enforce_app_insert_defaults()

```sql
CREATE OR REPLACE FUNCTION public.doctors_enforce_app_insert_defaults()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public', 'premed'
AS $function$
declare jwt_role text;
begin
  begin jwt_role := current_setting('request.jwt.claims', true)::jsonb->>'role';
  exception when others then jwt_role := null; end;
  if jwt_role = 'service_role' or jwt_role is null
     or current_user in ('service_role','postgres','supabase_admin')
     or premed.has_role(auth.uid(),'mentor') or premed.has_role(auth.uid(),'admin') then
    return new;
  end if;
  new.app_status        := 'submitted';
  new.payment_status    := 'pending';
  new.payment_id        := null;
  new.payment_intent_id := null;
  new.payment_date      := null;
  new.amount            := null;
  new.currency          := null;
  new.reviewed_at       := null;
  new.completion_token  := null;
  return new;
end $function$

```

#### public.enforce_finlit_view_security_invoker()

```sql
CREATE OR REPLACE FUNCTION public.enforce_finlit_view_security_invoker()
 RETURNS event_trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'pg_catalog'
AS $function$
declare
  cmd record;
begin
  for cmd in
    select *
    from pg_event_trigger_ddl_commands()
    where command_tag = 'CREATE VIEW'
      and object_type = 'view'
      and schema_name = 'finedu'
  loop
    execute format('alter view %s set (security_invoker = true);', cmd.object_identity);
  end loop;
end;
$function$

```

#### public.enforce_update_permissions()

```sql
CREATE OR REPLACE FUNCTION public.enforce_update_permissions()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public', 'pg_temp'
AS $function$
DECLARE
  v_role public.user_role;
BEGIN
  SELECT role INTO v_role
  FROM public.leads_meeting_profiles
  WHERE id = auth.uid();

  IF v_role IN ('super_admin', 'admin') THEN
    RETURN NEW;
-- ... truncated: showing first 15 of 60 lines
```

#### public.execute_split(p_sm_ref text, p_splits jsonb)

```sql
CREATE OR REPLACE FUNCTION public.execute_split(p_sm_ref text, p_splits jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
DECLARE
  orig          RECORD;
  n             INTEGER;
  orig_buy      NUMERIC;
  sum_input_buy NUMERIC := 0;
  orig_sell     NUMERIC;
  orig_fees     NUMERIC;
  orig_bank     NUMERIC;
  sell_running  NUMERIC := 0;
  rate_val      NUMERIC;
-- ... truncated: showing first 15 of 145 lines
```

#### public.fn_assign_sr_no()

```sql
CREATE OR REPLACE FUNCTION public.fn_assign_sr_no()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
BEGIN
  IF NEW.sr_no IS NULL THEN
    SELECT COALESCE(MAX(sr_no), 0) + 1
      INTO NEW.sr_no
      FROM public.wallex_transactions;
  END IF;
  RETURN NEW;
END;
$function$

```

#### public.fn_renumber_sr_no()

```sql
CREATE OR REPLACE FUNCTION public.fn_renumber_sr_no()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
BEGIN
  WITH ranked AS (
    SELECT id,
           ROW_NUMBER() OVER (
             ORDER BY
               COALESCE(split_parent_ref, sm_conversion_reference),
               split_index NULLS FIRST,
               created_at,
               id
           ) AS new_sr
    FROM  public.wallex_transactions
    WHERE sm_conversion_reference IS NOT NULL
      AND sm_conversion_reference <> ''
  )
  UPDATE public.wallex_transactions t
  SET    sr_no = r.new_sr
  FROM   ranked r
  WHERE  t.id = r.id;

  RETURN NULL;
END;
$function$

```

#### public.get_or_create_student_id(_email text, _mobile text, _name text)

```sql
CREATE OR REPLACE FUNCTION public.get_or_create_student_id(_email text, _mobile text DEFAULT NULL::text, _name text DEFAULT NULL::text)
 RETURNS text
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
declare found text; alphabet text := '23456789ABCDEFGHJKMNPQRSTUVWXYZ'; cand text; i int; tries int := 0;
begin
  found := public.resolve_student_id(_email, _mobile, _name);
  if found is not null then return found; end if;
  loop
    cand := 'S' || to_char(now() at time zone 'UTC','YY');
    for i in 1..7 loop cand := cand || substr(alphabet, 1 + floor(random()*31)::int, 1); end loop;
    if not public.student_id_in_use(cand) then return cand; end if;
    tries := tries + 1; if tries > 10 then raise exception 'Could not generate unique student_id'; end if;
  end loop;
end $function$

```

#### public.handle_new_user()

```sql
CREATE OR REPLACE FUNCTION public.handle_new_user()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
BEGIN
  INSERT INTO public.leads_meeting_profiles (id, full_name, email, role, is_active)
  VALUES (
    NEW.id,
    COALESCE(NEW.raw_user_meta_data->>'full_name', NEW.email),
    NEW.email,
    'sales_agent',
    false
  )
  ON CONFLICT (id) DO NOTHING;

  RETURN NEW;
END;
$function$

```

#### public.handle_updated_at()

```sql
CREATE OR REPLACE FUNCTION public.handle_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$function$

```

#### public.has_role(_user_id uuid, _role app_role)

```sql
CREATE OR REPLACE FUNCTION public.has_role(_user_id uuid, _role app_role)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
  SELECT EXISTS (
    SELECT 1 FROM public.user_roles
    WHERE user_id = _user_id AND role = _role
  )
$function$

```

#### public.is_finance_admin(_uid uuid)

```sql
CREATE OR REPLACE FUNCTION public.is_finance_admin(_uid uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'finance', 'public'
AS $function$
  select _uid = auth.uid() and exists (select 1 from finance.user_profiles u
    where u.user_id = _uid and u.role = any (array['owner'::finance.app_role, 'admin'::finance.app_role]));
$function$

```

#### public.legacy_question_form_data(_q text[], _tpl uuid, _code text, _existing jsonb)

```sql
CREATE OR REPLACE FUNCTION public.legacy_question_form_data(_q text[], _tpl uuid, _code text, _existing jsonb)
 RETURNS jsonb
 LANGUAGE plpgsql
 STABLE
 SET search_path TO 'public', 'portal'
AS $function$
DECLARE _fields jsonb; _label text; _out jsonb := '[]'::jsonb; _rest jsonb; i int;
BEGIN
  SELECT CASE WHEN jsonb_typeof(t.schema)='array' THEN t.schema ELSE t.schema->'fields' END INTO _fields
  FROM portal.form_templates t
  WHERE (_tpl IS NOT NULL AND t.id=_tpl) OR (_tpl IS NULL AND t.program_code=_code)
  ORDER BY (t.id=_tpl) DESC NULLS LAST, t.is_active DESC, t.created_at LIMIT 1;
  FOR i IN 1..4 LOOP
    IF NULLIF(btrim(coalesce(_q[i],'')),'') IS NOT NULL THEN
      SELECT f->>'label' INTO _label FROM jsonb_array_elements(coalesce(_fields,'[]'::jsonb)) f
       WHERE f->>'db_column'='question'||i AND NULLIF(btrim(f->>'label'),'') IS NOT NULL LIMIT 1;
      IF _label IS NULL THEN
        SELECT a->>'question' INTO _label FROM jsonb_array_elements(CASE WHEN jsonb_typeof(_existing->'answers')='array' THEN _existing->'answers' ELSE '[]'::jsonb END) a
         WHERE a->>'key'='question'||i AND a->>'question' NOT LIKE 'Question _' LIMIT 1;
      END IF;
      _out := _out || jsonb_build_array(jsonb_build_object('key','question'||i,'question',coalesce(_label,'Question '||i),'answer',_q[i]));
      _label := NULL;
    END IF;
  END LOOP;
  SELECT coalesce(jsonb_agg(a),'[]'::jsonb) INTO _rest FROM jsonb_array_elements(
    CASE WHEN jsonb_typeof(_existing->'answers')='array' THEN _existing->'answers' ELSE '[]'::jsonb END) a
   WHERE coalesce(a->>'key','') NOT IN ('question1','question2','question3','question4');
  RETURN coalesce(CASE WHEN jsonb_typeof(_existing)='object' THEN _existing END,'{}'::jsonb)
         || jsonb_build_object('answers', _out || _rest);
END $function$

```

#### public.log_lead_activity()

```sql
CREATE OR REPLACE FUNCTION public.log_lead_activity()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
DECLARE
  current_uid uuid;
BEGIN
  current_uid := auth.uid();

  IF TG_OP = 'INSERT' THEN
    INSERT INTO public.leads_meeting_activity_log (lead_id, user_id, action, old_value, new_value)
    VALUES (NEW.id, current_uid, 'Lead Created', NULL, NULL);
    RETURN NEW;
-- ... truncated: showing first 15 of 43 lines
```

#### public.lwl_form_touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION public.lwl_form_touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'extensions', 'pg_temp'
AS $function$
begin new.updated_at = now(); return new; end $function$

```

#### public.merge_form_answers(_existing jsonb, _incoming jsonb)

```sql
CREATE OR REPLACE FUNCTION public.merge_form_answers(_existing jsonb, _incoming jsonb)
 RETURNS jsonb
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'public', 'extensions', 'pg_temp'
AS $function$
  SELECT CASE WHEN jsonb_typeof(_incoming->'answers') <> 'array' OR _incoming IS NULL
    THEN coalesce(_existing,'{}'::jsonb)
  ELSE coalesce(CASE WHEN jsonb_typeof(_existing)='object' THEN _existing END,'{}'::jsonb) || jsonb_build_object('answers',
    coalesce((SELECT jsonb_agg(e) FROM jsonb_array_elements(CASE WHEN jsonb_typeof(_existing->'answers')='array' THEN _existing->'answers' ELSE '[]'::jsonb END) e
      WHERE NOT EXISTS (SELECT 1 FROM jsonb_array_elements(_incoming->'answers') n WHERE n->>'key' = e->>'key')),'[]'::jsonb)
    || (SELECT coalesce(jsonb_agg(n),'[]'::jsonb) FROM jsonb_array_elements(_incoming->'answers') n
        WHERE NULLIF(n->>'key','') IS NOT NULL AND NULLIF(btrim(coalesce(n->>'answer','')),'') IS NOT NULL))
  END $function$

```

#### public.mirror_legacy_application()

```sql
CREATE OR REPLACE FUNCTION public.mirror_legacy_application()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public', 'portal'
AS $function$
DECLARE
  rec public.applications;
  existing_id uuid;
BEGIN
  rec := jsonb_populate_record(NULL::public.applications, to_jsonb(NEW));
  IF rec.id IS NULL OR rec.program_code IS NULL OR coalesce(rec.student_email,'') ILIKE '<redacted>@example-group.com' THEN
    RETURN NEW;
  END IF;

-- ... truncated: showing first 15 of 51 lines
```

#### public.normalize_grade_level(input text)

```sql
CREATE OR REPLACE FUNCTION public.normalize_grade_level(input text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'public', 'pg_temp'
AS $function$
DECLARE
  n int;
  cleaned text;
BEGIN
  IF input IS NULL THEN RETURN NULL; END IF;
  cleaned := btrim(input);
  IF cleaned = '' THEN RETURN NULL; END IF;

  -- Extract leading integer if present (handles "9", "9th", "Grade 9", "10th Grade")
  n := NULLIF(regexp_replace(cleaned, '^\D*(\d{1,2}).*$', '\1'), '')::int;

  IF n IS NOT NULL AND n BETWEEN 1 AND 12 THEN
    RETURN CASE n
      WHEN 1 THEN '1st Grade'
      WHEN 2 THEN '2nd Grade'
      WHEN 3 THEN '3rd Grade'
      ELSE n::text || 'th Grade'
    END;
  END IF;

  RETURN cleaned; -- leave non-numeric labels (e.g. "Gap Year") untouched
EXCEPTION WHEN others THEN
  RETURN cleaned;
END;
$function$

```

#### public.normalize_place_name(input text)

```sql
CREATE OR REPLACE FUNCTION public.normalize_place_name(input text)
 RETURNS text
 LANGUAGE plpgsql
 IMMUTABLE
 SET search_path TO 'public', 'pg_temp'
AS $function$
DECLARE
  cleaned text;
  upper_input text;
BEGIN
  IF input IS NULL THEN RETURN NULL; END IF;
  cleaned := btrim(regexp_replace(input, '\s+', ' ', 'g'));
  IF cleaned = '' THEN RETURN NULL; END IF;

  upper_input := upper(cleaned);
  -- Preserve common acronyms
  IF upper_input IN ('UAE','USA','UK','UAS') THEN
    RETURN CASE upper_input
      WHEN 'UAE' THEN 'United Arab Emirates'
      WHEN 'USA' THEN 'United States'
      WHEN 'UK'  THEN 'United Kingdom'
      ELSE upper_input
    END;
  END IF;

  RETURN initcap(lower(cleaned));
END;
$function$

```

#### public.normalize_teo_application_fields()

```sql
CREATE OR REPLACE FUNCTION public.normalize_teo_application_fields()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
DECLARE grade_match text[];
BEGIN
  NEW.student_name := regexp_replace(btrim(NEW.student_name), '\s+', ' ', 'g');
  NEW.parent_name := nullif(regexp_replace(btrim(NEW.parent_name), '\s+', ' ', 'g'), '');
  -- Preserve valid nonnumeric grade choices; never infer Year -> Grade.
  IF TG_OP = 'INSERT' OR NEW.grade IS DISTINCT FROM OLD.grade THEN
    NEW.grade := nullif(btrim(NEW.grade), '');
    grade_match := regexp_match(NEW.grade, '^(?:grade\s+)?([0-9]{1,2})(?:st|nd|rd|th)?(?:\s+grade)?$', 'i');
    IF grade_match IS NOT NULL THEN NEW.grade := grade_match[1]::int::text; END IF;
  END IF;
  NEW.country := public.normalize_place_name(NEW.country);
  NEW.state := public.normalize_place_name(NEW.state);
  NEW.city := public.normalize_place_name(NEW.city);
  NEW.student_mobile := nullif(regexp_replace(coalesce(NEW.student_mobile,''), '\D', '', 'g'), '');
  NEW.parent_phone := nullif(regexp_replace(coalesce(NEW.parent_phone,''), '\D', '', 'g'), '');
  NEW.student_email := lower(btrim(NEW.student_email));
  NEW.parent_email := nullif(lower(btrim(NEW.parent_email)), '');
  NEW.counsellor_email := nullif(lower(btrim(NEW.counsellor_email)), '');
  NEW.currency := upper(coalesce(NEW.currency, 'USD'));
  -- Applied codes arrive from trusted checkout validation; preserve registry casing.
  IF TG_OP = 'INSERT' OR NEW.scholarship_code IS DISTINCT FROM OLD.scholarship_code THEN
    NEW.scholarship_code := nullif(btrim(NEW.scholarship_code), '');
  END IF;
  RETURN NEW;
END
$function$

```

#### public.onward_apps_set_cohort_id()

```sql
CREATE OR REPLACE FUNCTION public.onward_apps_set_cohort_id()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
BEGIN
  IF NEW.cohort IS NOT NULL AND (NEW.cohort_id IS NULL OR NEW.cohort IS DISTINCT FROM COALESCE(OLD.cohort, '')) THEN
    SELECT c.id INTO NEW.cohort_id FROM public.all_cohorts c WHERE c.cohort_key = NEW.cohort;
  END IF;
  IF NEW.synced_at IS NULL THEN NEW.synced_at := now(); END IF;
  RETURN NEW;
END $function$

```

#### public.outreach_data_normalize_email()

```sql
CREATE OR REPLACE FUNCTION public.outreach_data_normalize_email()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
BEGIN
  NEW.email_address = LOWER(TRIM(NEW.email_address));
  RETURN NEW;
END $function$

```

#### public.outreach_data_touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION public.outreach_data_touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END $function$

```

#### public.promo_sync_redemption_count()

```sql
CREATE OR REPLACE FUNCTION public.promo_sync_redemption_count()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
DECLARE
  target uuid := COALESCE(NEW.promo_code_id, OLD.promo_code_id);
  target_code text := COALESCE(NEW.promo_code, OLD.promo_code);
BEGIN
  IF target IS NULL THEN
    SELECT id INTO target FROM public.promo_codes WHERE code = target_code;
  END IF;
  IF target IS NOT NULL THEN
    UPDATE public.promo_codes p
       SET redemption_count = (
             SELECT count(*) FROM public.promo_redemptions r WHERE r.promo_code_id = p.id
           ),
           updated_at = now()
     WHERE p.id = target;
  END IF;
  RETURN NULL;
END;
$function$

```

#### public.promo_touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION public.promo_touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END;
$function$

```

#### public.razorpay_payments_touch_synced_at()

```sql
CREATE OR REPLACE FUNCTION public.razorpay_payments_touch_synced_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
begin
  new.synced_at = now();
  return new;
end;
$function$

```

#### public.research_role(_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION public.research_role(_user_id uuid)
 RETURNS text
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
  select role from public.research_staff_profiles where user_id = _user_id and active limit 1
$function$

```

#### public.resolve_student_id(_email text, _mobile text, _name text)

```sql
CREATE OR REPLACE FUNCTION public.resolve_student_id(_email text, _mobile text DEFAULT NULL::text, _name text DEFAULT NULL::text)
 RETURNS text
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
-- Read-only lookup across every platform's applications table (any schema, base tables
-- named %applications with student_id + student_email). Earliest matching ID wins.
-- Email (then digits-only mobile) must match AND names must share a word, so siblings
-- registered under one parent email/phone never get merged. Returns NULL if no match.
declare r record; best text; best_at timestamptz; cand text; cand_at timestamptz;
  em text := nullif(lower(trim(_email)),''); mob text := nullif(regexp_replace(coalesce(_mobile,''),'\D','','g'),'');
  toks text[] := public.student_name_tokens(_name); namecond text;
begin
  if em is null and (mob is null or length(mob) < 8) then return null; end if;
  namecond := 'public.student_names_match(student_name, $3)';
  for r in
    select n.nspname s, c.relname t,
      exists(select 1 from information_schema.columns x where x.table_schema=n.nspname and x.table_name=c.relname and x.column_name='student_mobile') has_mob,
      exists(select 1 from information_schema.columns x where x.table_schema=n.nspname and x.table_name=c.relname and x.column_name='student_name') has_name
    from pg_class c join pg_namespace n on n.oid=c.relnamespace
    where c.relkind in ('r','p') and c.relname like '%applications%'
      and n.nspname not in ('pg_catalog','information_schema')
      and c.relname <> 'leaders_applications'  -- Student Leaders tasks, not programme applications
      and exists(select 1 from information_schema.columns x where x.table_schema=n.nspname and x.table_name=c.relname and x.column_name='student_id')
      and exists(select 1 from information_schema.columns x where x.table_schema=n.nspname and x.table_name=c.relname and x.column_name='student_email')
      and exists(select 1 from information_schema.columns x where x.table_schema=n.nspname and x.table_name=c.relname and x.column_name='created_at')
  loop
    cand := null;
    execute format('select student_id::text, created_at from %I.%I where student_id is not null and student_id::text <> '''' and ('
      || 'lower(trim(student_email)) = $1' || case when r.has_mob then ' or (length($2) >= 8 and regexp_replace(coalesce(student_mobile,''''),''\D'','''',''g'') = $2)' else '' end
      || ') and ' || case when r.has_name then namecond else 'true' end || ' order by created_at limit 1', r.s, r.t)
      into cand, cand_at using em, coalesce(mob,''), _name;
    if cand is not null and (best_at is null or cand_at < best_at) then best := cand; best_at := cand_at; end if;
  end loop;
  return best;
end $function$

```

#### public.rls_auto_enable()

```sql
CREATE OR REPLACE FUNCTION public.rls_auto_enable()
 RETURNS event_trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'pg_catalog'
AS $function$
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
$function$

```

#### public.set_updated_at()

```sql
CREATE OR REPLACE FUNCTION public.set_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
begin
    new.updated_at = now();
    return new;
end;
$function$

```

#### public.student_id_in_use(_id text)

```sql
CREATE OR REPLACE FUNCTION public.student_id_in_use(_id text)
 RETURNS boolean
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
declare r record; hit boolean;
begin
  for r in select n.nspname s, c.relname t from pg_class c join pg_namespace n on n.oid=c.relnamespace
    where c.relkind in ('r','p') and n.nspname not in ('pg_catalog','information_schema')
      and c.relname <> 'leaders_applications'  -- Student Leaders tasks, not programme applications
      and exists(select 1 from information_schema.columns x where x.table_schema=n.nspname and x.table_name=c.relname and x.column_name='student_id')
      and c.relname like '%applications%'
  loop
    execute format('select exists(select 1 from %I.%I where student_id::text = $1)', r.s, r.t) into hit using _id;
    if hit then return true; end if;
  end loop;
  return false;
end $function$

```

#### public.student_name_tokens(_n text)

```sql
CREATE OR REPLACE FUNCTION public.student_name_tokens(_n text)
 RETURNS text[]
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'public', 'extensions', 'pg_temp'
AS $function$
  select coalesce(array_agg(t), '{}') from regexp_split_to_table(lower(coalesce(_n,'')), '[^a-z]+') t where length(t) >= 3
$function$

```

#### public.student_names_match(_a text, _b text)

```sql
CREATE OR REPLACE FUNCTION public.student_names_match(_a text, _b text)
 RETURNS boolean
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'public', 'extensions', 'pg_temp'
AS $function$
  -- Same person if names share 2+ words (any order), or all words of a one-word name.
  -- Blank or placeholder names "(...)" do not block a match. Siblings sharing only a surname do not match.
  with a as (select array(select distinct unnest(public.student_name_tokens(_a))) t),
       b as (select array(select distinct unnest(public.student_name_tokens(_b))) t)
  select case when coalesce(left(trim(_a),1),'(')='(' or coalesce(left(trim(_b),1),'(')='(' then true
              when cardinality(a.t)=0 or cardinality(b.t)=0 then true
              else (select count(*) from unnest(a.t) x where x = any(b.t)) >= least(2, cardinality(a.t), cardinality(b.t)) end
  from a, b
$function$

```

#### public.tg_edge_shared_student_id()

```sql
CREATE OR REPLACE FUNCTION public.tg_edge_shared_student_id()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
declare found text;
begin
  found := public.resolve_student_id(NEW.student_email, NEW.student_phone, NEW.student_name);
  if found is not null then NEW.student_id := found;
  elsif NEW.student_id is null or NEW.student_id = '' or public.student_id_in_use(NEW.student_id) then
    NEW.student_id := public.get_or_create_student_id(NEW.student_email, NEW.student_phone, NEW.student_name);
  end if;
  return NEW;
end $function$

```

#### public.tg_lms_vault_promo_redeemed()

```sql
CREATE OR REPLACE FUNCTION public.tg_lms_vault_promo_redeemed()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public', 'lms', 'pg_temp'
AS $function$
begin
  if upper(coalesce(new.promo_code, '')) like 'VAULT%'
     or exists (select 1 from lms.vault_vouchers where promo_code_id = new.promo_code_id) then
    perform lms.vault_commit_voucher_from_redemption(new.id);
  end if;
  return null;
exception when others then
  return null;
end $function$

```

#### public.tg_lwl_shared_student_id()

```sql
CREATE OR REPLACE FUNCTION public.tg_lwl_shared_student_id()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
declare found text;
begin
  found := public.resolve_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  if found is not null then NEW.student_id := found;
  elsif NEW.student_id is null or NEW.student_id = '' or public.student_id_in_use(NEW.student_id) then
    NEW.student_id := public.get_or_create_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  end if;
  return NEW;
end $function$

```

#### public.tg_protect_created_at()

```sql
CREATE OR REPLACE FUNCTION public.tg_protect_created_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'extensions', 'pg_temp'
AS $function$
begin
  if tg_op = 'INSERT' then
    if new.created_at is null then new.created_at := now(); end if;
  elsif new.created_at is distinct from old.created_at then
    new.created_at := old.created_at; -- original created date can never be changed or cleared
  end if;
  return new;
end $function$

```

#### public.tg_purpose_shared_student_id()

```sql
CREATE OR REPLACE FUNCTION public.tg_purpose_shared_student_id()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
declare found text;
begin
  found := public.resolve_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  if found is not null then NEW.student_id := found;
  elsif NEW.student_id is null or NEW.student_id = '' or public.student_id_in_use(NEW.student_id) then
    NEW.student_id := public.get_or_create_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  end if;
  return NEW;
end $function$

```

#### public.tg_sirluther_shared_student_id()

```sql
CREATE OR REPLACE FUNCTION public.tg_sirluther_shared_student_id()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
declare found text;
begin
  found := public.resolve_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  if found is not null then NEW.student_id := found;
  elsif NEW.student_id is null or NEW.student_id = '' or public.student_id_in_use(NEW.student_id) then
    NEW.student_id := public.get_or_create_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  end if;
  return NEW;
end $function$

```

#### public.tg_teo_shared_student_id()

```sql
CREATE OR REPLACE FUNCTION public.tg_teo_shared_student_id()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
declare found text;
begin
  found := public.resolve_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  if found is not null then NEW.student_id := found;
  elsif NEW.student_id is null or NEW.student_id = '' or public.student_id_in_use(NEW.student_id) then
    NEW.student_id := public.get_or_create_student_id(NEW.student_email, NEW.student_mobile, NEW.student_name);
  end if;
  return NEW;
end $function$

```

#### public.touch_testprep_masterclass_regs_updated_at()

```sql
CREATE OR REPLACE FUNCTION public.touch_testprep_masterclass_regs_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END; $function$

```

#### public.touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION public.touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
begin new.updated_at = now(); return new; end $function$

```

#### public.update_updated_at()

```sql
CREATE OR REPLACE FUNCTION public.update_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$function$

```

#### public.update_updated_at_column()

```sql
CREATE OR REPLACE FUNCTION public.update_updated_at_column()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public', 'pg_temp'
AS $function$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$function$

```

## testprep (17 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| current_session_id |  | text | sql | invoker | stable | search_path=testprep, public | anon authenticated |
| dial_code_for_country | p_country text | text | sql | invoker | stable | search_path=testprep, public |  |
| enforce_utm_defaults |  | trigger | plpgsql | invoker | volatile | search_path=testprep, public |  |
| is_trusted_writer |  | boolean | plpgsql | invoker | stable | search_path=testprep, public |  |
| normalize_masterclass_phones |  | trigger | plpgsql | DEFINER | volatile | search_path=testprep, public |  |
| normalize_phone | p_phone text, p_country text | text | plpgsql | invoker | stable | search_path=testprep, public |  |
| normalize_program_cohort_from_start_date |  | trigger | plpgsql | invoker | volatile | search_path=testprep, public |  |
| normalize_school_inquiry_phone |  | trigger | plpgsql | DEFINER | volatile | search_path=testprep, public |  |
| normalize_whatsapp_send_phone |  | trigger | plpgsql | DEFINER | volatile | search_path=testprep, public |  |
| program_cohorts_write |  | trigger | plpgsql | DEFINER | volatile | search_path=public |  |
| record_payment_attribution | p_application_id uuid, p_payment_intent_id text, p_entry jsonb | boolean | sql | DEFINER | volatile | search_path=testprep, public |  |
| sat_program_id |  | uuid | sql | DEFINER | stable | search_path=public |  |
| snapshot_student_potential |  | trigger | plpgsql | DEFINER | volatile | search_path=testprep, public |  |
| tg_sat_applications_freeze_utm |  | trigger | plpgsql | invoker | volatile | search_path=testprep, public |  |
| tg_sat_applications_normalize |  | trigger | plpgsql | DEFINER | volatile | search_path=testprep, public |  |
| touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=testprep, public |  |
| update_trap_stats |  | trigger | plpgsql | DEFINER | volatile | search_path=testprep, public |  |

### testprep: bodies

#### testprep.current_session_id()

```sql
CREATE OR REPLACE FUNCTION testprep.current_session_id()
 RETURNS text
 LANGUAGE sql
 STABLE
 SET search_path TO 'testprep', 'public'
AS $function$
  SELECT NULLIF(current_setting('request.headers', true)::json ->> 'x-session-id', '');
$function$

```

#### testprep.dial_code_for_country(p_country text)

```sql
CREATE OR REPLACE FUNCTION testprep.dial_code_for_country(p_country text)
 RETURNS text
 LANGUAGE sql
 STABLE
 SET search_path TO 'testprep', 'public'
AS $function$
  SELECT dial_code FROM testprep.country_dial_codes
  WHERE country_name = lower(btrim(coalesce(p_country, ''))) LIMIT 1;
$function$

```

#### testprep.enforce_utm_defaults()

```sql
CREATE OR REPLACE FUNCTION testprep.enforce_utm_defaults()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'testprep', 'public'
AS $function$
BEGIN
  NEW.utm_source   := CASE WHEN lower(btrim(coalesce(NEW.utm_source,''))) IN ('','direct','website','none','null','undefined','not-set','(none)','(direct)','unknown') THEN 'unknown' ELSE NEW.utm_source END;
  NEW.utm_medium   := CASE WHEN lower(btrim(coalesce(NEW.utm_medium,''))) IN ('','direct','website','none','null','undefined','not-set','(none)','(direct)','unknown') THEN 'unknown' ELSE NEW.utm_medium END;
  NEW.utm_campaign := CASE WHEN lower(btrim(coalesce(NEW.utm_campaign,''))) IN ('','direct','website','none','null','undefined','not-set','(none)','(direct)','unknown') THEN 'unknown' ELSE NEW.utm_campaign END;
  RETURN NEW;
END $function$

```

#### testprep.is_trusted_writer()

```sql
CREATE OR REPLACE FUNCTION testprep.is_trusted_writer()
 RETURNS boolean
 LANGUAGE plpgsql
 STABLE
 SET search_path TO 'testprep', 'public'
AS $function$
DECLARE jwt_role text;
BEGIN
  BEGIN jwt_role := current_setting('request.jwt.claims', true)::jsonb->>'role';
  EXCEPTION WHEN OTHERS THEN jwt_role := NULL; END;
  RETURN jwt_role = 'service_role'
      OR session_user IN ('service_role','postgres','supabase_admin');
END;
$function$

```

#### testprep.normalize_masterclass_phones()

```sql
CREATE OR REPLACE FUNCTION testprep.normalize_masterclass_phones()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'testprep', 'public'
AS $function$
BEGIN
  NEW.phone        := testprep.normalize_phone(NEW.phone, NEW.country);
  NEW.parent_phone := testprep.normalize_phone(NEW.parent_phone, NEW.country);
  RETURN NEW;
END;
$function$

```

#### testprep.normalize_phone(p_phone text, p_country text)

```sql
CREATE OR REPLACE FUNCTION testprep.normalize_phone(p_phone text, p_country text DEFAULT NULL::text)
 RETURNS text
 LANGUAGE plpgsql
 STABLE
 SET search_path TO 'testprep', 'public'
AS $function$
DECLARE raw text; digits text; dial text; rest text; has_plus boolean; own_dial text;
BEGIN
  raw := coalesce(p_phone, '');
  has_plus := raw LIKE '%+%';
  digits := regexp_replace(raw, '[^0-9]', '', 'g');
  IF digits = '' THEN RETURN NULL; END IF;
  digits := regexp_replace(digits, '^(00|011)', '');
  dial := testprep.dial_code_for_country(p_country);
  IF has_plus AND length(digits) BETWEEN 8 AND 15 THEN
-- ... truncated: showing first 15 of 43 lines
```

#### testprep.normalize_program_cohort_from_start_date()

```sql
CREATE OR REPLACE FUNCTION testprep.normalize_program_cohort_from_start_date()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'testprep', 'public'
AS $function$
begin
  if new.start_date is not null and (
    lower(trim(coalesce(new.cohort_label, ''))) = 'unassigned'
    or new.cohort_key like '%unassigned'
  ) then
    new.cohort_label := to_char(new.start_date, 'FMMonth YYYY');
    new.cohort_key := 'testprep-instinct-10-week-' || lower(to_char(new.start_date, 'MonYY'));
  end if;
  return new;
end;
$function$

```

#### testprep.normalize_school_inquiry_phone()

```sql
CREATE OR REPLACE FUNCTION testprep.normalize_school_inquiry_phone()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'testprep', 'public'
AS $function$
BEGIN
  NEW.contact_phone := testprep.normalize_phone(NEW.contact_phone, NULL);
  RETURN NEW;
END;
$function$

```

#### testprep.normalize_whatsapp_send_phone()

```sql
CREATE OR REPLACE FUNCTION testprep.normalize_whatsapp_send_phone()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'testprep', 'public'
AS $function$
BEGIN
  NEW.recipient_phone := testprep.normalize_phone(NEW.recipient_phone, NULL);
  RETURN NEW;
END;
$function$

```

#### testprep.program_cohorts_write()

```sql
CREATE OR REPLACE FUNCTION testprep.program_cohorts_write()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
DECLARE
  pid uuid := testprep.sat_program_id();
  sd  date;
BEGIN
  IF TG_OP = 'INSERT' THEN
    IF NEW.cohort_key IS NULL THEN
      RAISE EXCEPTION 'cohort_key is required';
    END IF;
    sd := NEW.start_date;
-- ... truncated: showing first 15 of 54 lines
```

#### testprep.record_payment_attribution(p_application_id uuid, p_payment_intent_id text, p_entry jsonb)

```sql
CREATE OR REPLACE FUNCTION testprep.record_payment_attribution(p_application_id uuid, p_payment_intent_id text, p_entry jsonb)
 RETURNS boolean
 LANGUAGE sql
 SECURITY DEFINER
 SET search_path TO 'testprep', 'public'
AS $function$
  with u as (
    update public.sat_applications
       set form_data = jsonb_set(coalesce(form_data,'{}'::jsonb), '{payment_attributions}',
             coalesce(form_data->'payment_attributions','{}'::jsonb) || jsonb_build_object(p_payment_intent_id, p_entry), true)
     where id = p_application_id
       and not coalesce((form_data->'payment_attributions') ? p_payment_intent_id, false)
    returning 1)
  select exists(select 1 from u);
$function$

```

#### testprep.sat_program_id()

```sql
CREATE OR REPLACE FUNCTION testprep.sat_program_id()
 RETURNS uuid
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
  SELECT id FROM public.all_programs
  WHERE partner_id = '3ade972c-03ca-4d84-9f41-29c89750d866'
  ORDER BY created_at
  LIMIT 1
$function$

```

#### testprep.snapshot_student_potential()

```sql
CREATE OR REPLACE FUNCTION testprep.snapshot_student_potential()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'testprep', 'public'
AS $function$
DECLARE
  v_accuracy_score INTEGER; v_mastery_score INTEGER; v_consistency_score INTEGER;
  v_speed_score INTEGER; v_potential_score INTEGER; v_questions_total INTEGER;
  v_traps_mastered INTEGER; v_growth_velocity NUMERIC(5,2); v_prev_potential INTEGER;
BEGIN
  IF NEW.ended_at IS NOT NULL AND OLD.ended_at IS NULL THEN
    SELECT COALESCE(ROUND((SUM(correct_answers)::NUMERIC / NULLIF(SUM(total_questions),0)) * 100),0)
      INTO v_accuracy_score FROM testprep.drill_sessions WHERE user_id = NEW.user_id AND total_questions > 0;

-- ... truncated: showing first 15 of 46 lines
```

#### testprep.tg_sat_applications_freeze_utm()

```sql
CREATE OR REPLACE FUNCTION testprep.tg_sat_applications_freeze_utm()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'testprep', 'public'
AS $function$
declare old_frozen jsonb; new_frozen jsonb;
begin
  if tg_op = 'INSERT' then
    -- Freeze on first save when the writer did not already do it.
    if coalesce(new.form_data,'{}'::jsonb) ? 'application_utm' is false then
      new.form_data := coalesce(new.form_data,'{}'::jsonb) || jsonb_build_object('application_utm', jsonb_build_object(
        'utm_source', coalesce(new.utm_source,'unknown'), 'utm_medium', coalesce(new.utm_medium,'unknown'),
        'utm_campaign', coalesce(new.utm_campaign,'unknown'), 'utm_content','unknown','utm_term','unknown',
        'captured_at', to_char(coalesce(new.created_at, now()) at time zone 'UTC','YYYY-MM-DD"T"HH24:MI:SS.MS"Z"')));
    end if;
    return new;
  end if;
  old_frozen := old.form_data -> 'application_utm';
  if old_frozen is null then return new; end if;
  -- After first save nothing may change application UTMs (payments, reminders,
  -- re-confirmed details, webhooks or admin edits).
  new.form_data := jsonb_set(coalesce(new.form_data,'{}'::jsonb), '{application_utm}', old_frozen, true);
  new.utm_source := old.utm_source; new.utm_medium := old.utm_medium; new.utm_campaign := old.utm_campaign;
  -- Recorded payment touches are append-only.
  if old.form_data ? 'payment_attributions' then
    new.form_data := jsonb_set(new.form_data, '{payment_attributions}',
      coalesce(new.form_data->'payment_attributions','{}'::jsonb) || (old.form_data->'payment_attributions'), true);
  end if;
  return new;
end $function$

```

#### testprep.tg_sat_applications_normalize()

```sql
CREATE OR REPLACE FUNCTION testprep.tg_sat_applications_normalize()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'testprep', 'public'
AS $function$
begin
  NEW.student_mobile := nullif(testprep.normalize_phone(NEW.student_mobile, NEW.country),'');
  NEW.parent_phone := nullif(testprep.normalize_phone(NEW.parent_phone, NEW.country),'');
  NEW.student_email := lower(btrim(NEW.student_email));
  NEW.parent_email := nullif(lower(btrim(NEW.parent_email)),'');
  NEW.counsellor_email := nullif(lower(btrim(NEW.counsellor_email)),'');
  NEW.student_name := regexp_replace(btrim(NEW.student_name),'\s+',' ','g');
  NEW.scholarship_code := nullif(upper(btrim(NEW.scholarship_code)),'');
  NEW.currency := upper(coalesce(NEW.currency,'USD'));
  return NEW;
end $function$

```

#### testprep.touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION testprep.touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'testprep', 'public'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END; $function$

```

#### testprep.update_trap_stats()

```sql
CREATE OR REPLACE FUNCTION testprep.update_trap_stats()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'testprep', 'public'
AS $function$
BEGIN
  IF NEW.is_correct THEN
    INSERT INTO testprep.user_trap_stats (user_id, trap_type, total_encounters, times_fallen, current_streak, best_streak)
    SELECT NEW.user_id, unnest(q.trap_types), 1, 0, 1, 1
    FROM testprep.questions q WHERE q.id = NEW.question_id
    ON CONFLICT (user_id, trap_type) DO UPDATE SET
      total_encounters = user_trap_stats.total_encounters + 1,
      current_streak = user_trap_stats.current_streak + 1,
      best_streak = GREATEST(user_trap_stats.best_streak, user_trap_stats.current_streak + 1),
      updated_at = now();
  ELSIF NEW.trap_type_hit IS NOT NULL THEN
    INSERT INTO testprep.user_trap_stats (user_id, trap_type, total_encounters, times_fallen, current_streak, best_streak)
    VALUES (NEW.user_id, NEW.trap_type_hit, 1, 1, 0, 0)
    ON CONFLICT (user_id, trap_type) DO UPDATE SET
      total_encounters = user_trap_stats.total_encounters + 1,
      times_fallen = user_trap_stats.times_fallen + 1,
      current_streak = 0,
      updated_at = now();
  END IF;
  RETURN NEW;
END;
$function$

```

## research (23 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| accept_invitation | _token uuid, _user_id uuid | jsonb | plpgsql | DEFINER | volatile | search_path=research, public | authenticated |
| attribution_value | v text | text | sql | invoker | immutable | search_path=research, public | anon authenticated |
| catalogue_for_cohort | _key text, OUT program_code text, OUT program_name text, OUT program_category text | record | sql | DEFINER | stable | search_path=public |  |
| dial_for_country | _country text | text | sql | invoker | stable | search_path=research, public | anon authenticated |
| enforce_application_insert_defaults |  | trigger | plpgsql | DEFINER | volatile | search_path=research, public |  |
| enforce_masterclass_insert_defaults |  | trigger | plpgsql | DEFINER | volatile | search_path=research, public |  |
| form_answer_get | _fd jsonb, _key text | text | sql | invoker | immutable | search_path=research, public, extensions, pg_temp | anon authenticated |
| form_answer_set | _fd jsonb, _key text, _question text, _answer text | jsonb | sql | invoker | immutable | search_path=research, public, extensions, pg_temp | anon authenticated |
| get_invitation_by_token | _token uuid | TABLE(id uuid, email text, role research.app_role, display_name text, school_lead_id uuid, expires_at timestamp with time zone, accepted_at timestamp with time zone) | sql | DEFINER | stable | search_path=research, public | anon authenticated |
| has_role | _user_id uuid, _role research.app_role | boolean | plpgsql | DEFINER | stable | search_path=research, public | anon authenticated |
| is_staff | _user_id uuid | boolean | sql | DEFINER | stable | search_path=research, public | anon authenticated |
| lock_teacher_students_student_update |  | trigger | plpgsql | DEFINER | volatile | search_path=research, public |  |
| lookup_teacher_by_email | input_email text | TABLE(id uuid, full_name text, email text) | sql | DEFINER | stable | search_path=research, public | anon authenticated |
| normalize_application_fields |  | trigger | plpgsql | invoker | volatile | search_path=public | anon authenticated |
| normalize_phone_digits | _raw text, _country text | text | plpgsql | invoker | stable | search_path=research, public | anon authenticated |
| record_payment_attribution | _id uuid, _intent text, _touch jsonb | void | plpgsql | DEFINER | volatile | search_path=research, public |  |
| sheet_safe_phone | _value text | text | sql | invoker | stable | search_path=research, public | anon authenticated |
| submit_public_application | _payload jsonb | uuid | plpgsql | DEFINER | volatile | search_path=research, public | anon authenticated |
| submit_public_masterclass_registration | _payload jsonb | uuid | plpgsql | DEFINER | volatile | search_path=research, public | anon authenticated |
| tg_default_utm |  | trigger | plpgsql | DEFINER | volatile | search_path=research, public |  |
| tg_sheet_safe_phone |  | trigger | plpgsql | DEFINER | volatile | search_path=research, public |  |
| tg_standard_application_format |  | trigger | plpgsql | DEFINER | volatile | search_path=research, public |  |
| update_updated_at_column |  | trigger | plpgsql | DEFINER | volatile | search_path=research, public |  |

### research: bodies

#### research.accept_invitation(_token uuid, _user_id uuid)

```sql
CREATE OR REPLACE FUNCTION research.accept_invitation(_token uuid, _user_id uuid)
 RETURNS jsonb
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
DECLARE
  v_invite research.staff_invitations%ROWTYPE;
  v_user_email text;
BEGIN
  SELECT * INTO v_invite FROM research.staff_invitations WHERE token = _token LIMIT 1;
  IF NOT FOUND THEN
    RETURN jsonb_build_object('ok', false, 'error', 'invitation_not_found');
  END IF;

-- ... truncated: showing first 15 of 54 lines
```

#### research.attribution_value(v text)

```sql
CREATE OR REPLACE FUNCTION research.attribution_value(v text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'research', 'public'
AS $function$
  SELECT CASE WHEN v IS NULL OR lower(btrim(v)) IN ('','unknown','direct','website','none','(none)','(direct)','null','undefined')
    THEN 'unknown' ELSE btrim(split_part(split_part(split_part(v, ',', 1), ';', 1), '|', 1)) END;
$function$

```

#### research.catalogue_for_cohort(_key text, OUT program_code text, OUT program_name text, OUT program_category text)

```sql
CREATE OR REPLACE FUNCTION research.catalogue_for_cohort(_key text, OUT program_code text, OUT program_name text, OUT program_category text)
 RETURNS record
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'public'
AS $function$
  SELECT c.program_code, p.program_name,
    CASE lower(coalesce(p.categories[1],''))
      WHEN 'fellowships' THEN 'fellowship' WHEN 'fellowship' THEN 'fellowship'
      WHEN 'certificate programs' THEN 'certificate-program' WHEN 'certificate program' THEN 'certificate-program'
      WHEN 'camps' THEN 'camp' WHEN 'camp' THEN 'camp' WHEN 'conferences' THEN 'conference' WHEN 'conference' THEN 'conference'
      WHEN 'competitions' THEN 'competition' WHEN 'competition' THEN 'competition' WHEN 'in-person' THEN 'in-person'
      ELSE NULL END
  FROM public.all_cohorts c LEFT JOIN public.all_programs p ON p.id = c.program_id
  WHERE c.cohort_key = _key LIMIT 1
$function$

```

#### research.dial_for_country(_country text)

```sql
CREATE OR REPLACE FUNCTION research.dial_for_country(_country text)
 RETURNS text
 LANGUAGE sql
 STABLE
 SET search_path TO 'research', 'public'
AS $function$
  SELECT dial FROM (
    SELECT c.dial, 1 AS pref
    FROM research.country_dial_codes c
    WHERE lower(btrim(coalesce(_country,''))) <> ''
      AND lower(c.name) = lower(btrim(_country))
    UNION ALL
    SELECT c.dial, 2
    FROM research.country_dial_aliases a
    JOIN research.country_dial_codes c ON c.iso2 = a.iso2
    WHERE lower(btrim(coalesce(_country,''))) <> ''
      AND a.alias = lower(btrim(_country))
    UNION ALL
    SELECT c.dial, 3
    FROM research.country_dial_codes c
    WHERE length(btrim(coalesce(_country,''))) = 2
      AND lower(c.iso2) = lower(btrim(_country))
  ) t
  ORDER BY pref
  LIMIT 1;
$function$

```

#### research.enforce_application_insert_defaults()

```sql
CREATE OR REPLACE FUNCTION research.enforce_application_insert_defaults()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
DECLARE jwt_role text;
BEGIN
  BEGIN
    jwt_role := current_setting('request.jwt.claims', true)::jsonb->>'role';
  EXCEPTION WHEN OTHERS THEN jwt_role := NULL;
  END;

  IF jwt_role = 'service_role'
     OR current_user IN ('service_role','postgres','supabase_admin') THEN
    RETURN NEW;
  END IF;

  IF auth.uid() IS NOT NULL AND (
       research.has_role(auth.uid(),'admin'::app_role)
    OR research.has_role(auth.uid(),'teacher'::app_role)
    OR research.has_role(auth.uid(),'mentor'::app_role)
  ) THEN
    RETURN NEW;
  END IF;

  -- Anon + regular users: force safe defaults regardless of client payload
  NEW.payment_status           := 'unpaid';
  NEW.payment_id               := NULL;
  NEW.stripe_session_id        := NULL;
  NEW.amount_paid              := NULL;
  NEW.amount_currency          := NULL;
  NEW.payment_date             := NULL;
  NEW.payment_reminder_sent_at := NULL;
  RETURN NEW;
END;
$function$

```

#### research.enforce_masterclass_insert_defaults()

```sql
CREATE OR REPLACE FUNCTION research.enforce_masterclass_insert_defaults()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
DECLARE jwt_role text;
BEGIN
  BEGIN
    jwt_role := current_setting('request.jwt.claims', true)::jsonb->>'role';
  EXCEPTION WHEN OTHERS THEN jwt_role := NULL;
  END;

  IF jwt_role = 'service_role'
     OR current_user IN ('service_role','postgres','supabase_admin') THEN
    RETURN NEW;
  END IF;

  IF auth.uid() IS NOT NULL AND (
       research.has_role(auth.uid(),'admin'::app_role)
    OR research.has_role(auth.uid(),'teacher'::app_role)
    OR research.has_role(auth.uid(),'mentor'::app_role)
  ) THEN
    RETURN NEW;
  END IF;

  -- Force role default; anon cannot spoof themselves as staff
  IF NEW.role IS NULL OR btrim(NEW.role) = '' THEN
    NEW.role := 'Student';
  END IF;
  RETURN NEW;
END;
$function$

```

#### research.form_answer_get(_fd jsonb, _key text)

```sql
CREATE OR REPLACE FUNCTION research.form_answer_get(_fd jsonb, _key text)
 RETURNS text
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'research', 'public', 'extensions', 'pg_temp'
AS $function$
  SELECT a->>'answer' FROM jsonb_array_elements(coalesce(_fd->'answers','[]'::jsonb)) a WHERE a->>'key' = _key LIMIT 1
$function$

```

#### research.form_answer_set(_fd jsonb, _key text, _question text, _answer text)

```sql
CREATE OR REPLACE FUNCTION research.form_answer_set(_fd jsonb, _key text, _question text, _answer text)
 RETURNS jsonb
 LANGUAGE sql
 IMMUTABLE
 SET search_path TO 'research', 'public', 'extensions', 'pg_temp'
AS $function$
  SELECT CASE WHEN _answer IS NULL THEN coalesce(_fd,'{}'::jsonb) ELSE
    jsonb_set(coalesce(_fd,'{}'::jsonb), '{answers}',
      coalesce((SELECT jsonb_agg(a) FROM jsonb_array_elements(coalesce(_fd->'answers','[]'::jsonb)) a WHERE a->>'key' <> _key),'[]'::jsonb)
      || jsonb_build_array(jsonb_build_object('key',_key,'question',_question,'answer',_answer)), true) END
$function$

```

#### research.get_invitation_by_token(_token uuid)

```sql
CREATE OR REPLACE FUNCTION research.get_invitation_by_token(_token uuid)
 RETURNS TABLE(id uuid, email text, role research.app_role, display_name text, school_lead_id uuid, expires_at timestamp with time zone, accepted_at timestamp with time zone)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
  SELECT id, email, role, display_name, school_lead_id, expires_at, accepted_at
  FROM research.staff_invitations
  WHERE token = _token
  LIMIT 1;
$function$

```

#### research.has_role(_user_id uuid, _role research.app_role)

```sql
CREATE OR REPLACE FUNCTION research.has_role(_user_id uuid, _role research.app_role)
 RETURNS boolean
 LANGUAGE plpgsql
 STABLE SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
DECLARE v_caller uuid := auth.uid();
BEGIN
  IF v_caller IS NOT NULL AND _user_id IS DISTINCT FROM v_caller THEN RETURN false; END IF;
  RETURN EXISTS (SELECT 1 FROM research.user_roles WHERE user_id = _user_id AND role = _role);
END; $function$

```

#### research.is_staff(_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION research.is_staff(_user_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
  SELECT EXISTS (SELECT 1 FROM research.user_roles WHERE user_id = _user_id AND role IN ('admin','teacher','mentor','school'));
$function$

```

#### research.lock_teacher_students_student_update()

```sql
CREATE OR REPLACE FUNCTION research.lock_teacher_students_student_update()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
BEGIN
  -- Teachers (owners) and admins may edit freely.
  IF auth.uid() IS NULL
     OR auth.uid() = OLD.teacher_id
     OR research.has_role(auth.uid(), 'admin'::app_role) THEN
    RETURN NEW;
  END IF;

  -- Student responding to their own invite: pin immutable fields.
  IF auth.uid() = OLD.student_id THEN
    NEW.id            := OLD.id;
    NEW.teacher_id    := OLD.teacher_id;
    NEW.student_id    := OLD.student_id;
    NEW.student_email := OLD.student_email;
    NEW.created_at    := OLD.created_at;
    IF NEW.status NOT IN ('pending', 'accepted', 'declined') THEN
      RAISE EXCEPTION 'invalid invite status';
    END IF;
  END IF;

  RETURN NEW;
END;
$function$

```

#### research.lookup_teacher_by_email(input_email text)

```sql
CREATE OR REPLACE FUNCTION research.lookup_teacher_by_email(input_email text)
 RETURNS TABLE(id uuid, full_name text, email text)
 LANGUAGE sql
 STABLE SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
  SELECT tl.id, tl.full_name, tl.email
  FROM research.teacher_leads tl
  WHERE tl.email = input_email
  LIMIT 1;
$function$

```

#### research.normalize_application_fields()

```sql
CREATE OR REPLACE FUNCTION research.normalize_application_fields()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'public'
AS $function$
declare c text;
begin
  -- empty strings -> NULL (group stores absent values as NULL)
  new.student_mobile := nullif(btrim(new.student_mobile), '');
  new.parent_phone   := nullif(btrim(new.parent_phone), '');
  new.parent_name    := nullif(btrim(new.parent_name), '');
  new.state          := nullif(btrim(new.state), '');
  new.city           := nullif(btrim(new.city), '');
  new.counsellor_name  := nullif(btrim(new.counsellor_name), '');
  new.counsellor_email := nullif(lower(btrim(new.counsellor_email)), '');
-- ... truncated: showing first 15 of 42 lines
```

#### research.normalize_phone_digits(_raw text, _country text)

```sql
CREATE OR REPLACE FUNCTION research.normalize_phone_digits(_raw text, _country text DEFAULT NULL::text)
 RETURNS text
 LANGUAGE plpgsql
 STABLE
 SET search_path TO 'research', 'public'
AS $function$
DECLARE
  s text := btrim(coalesce(_raw, ''));
  had_plus boolean;
  d text;
  dial text;
  rest text;
BEGIN
  IF s = '' THEN RETURN NULL; END IF;
  s := regexp_replace(s, '^''+', '');
-- ... truncated: showing first 15 of 54 lines
```

#### research.record_payment_attribution(_id uuid, _intent text, _touch jsonb)

```sql
CREATE OR REPLACE FUNCTION research.record_payment_attribution(_id uuid, _intent text, _touch jsonb)
 RETURNS void
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
DECLARE fd jsonb;
BEGIN
  IF _intent NOT LIKE 'pi_%' OR jsonb_typeof(_touch) <> 'object' THEN RAISE EXCEPTION 'invalid payment attribution'; END IF;
  SELECT coalesce(form_data, '{}'::jsonb) INTO fd FROM public.sirluther_applications WHERE id = _id FOR UPDATE;
  IF NOT FOUND THEN RAISE EXCEPTION 'application not found'; END IF;
  IF coalesce(fd->'payment_attributions', '{}'::jsonb) ? _intent THEN RETURN; END IF;
  UPDATE public.sirluther_applications SET form_data = jsonb_set(fd, '{payment_attributions}',
    coalesce(fd->'payment_attributions', '{}'::jsonb) || jsonb_build_object(_intent, _touch)) WHERE id = _id;
END;
$function$

```

#### research.sheet_safe_phone(_value text)

```sql
CREATE OR REPLACE FUNCTION research.sheet_safe_phone(_value text)
 RETURNS text
 LANGUAGE sql
 STABLE
 SET search_path TO 'research', 'public'
AS $function$
  SELECT research.normalize_phone_digits(_value, NULL);
$function$

```

#### research.submit_public_application(_payload jsonb)

```sql
CREATE OR REPLACE FUNCTION research.submit_public_application(_payload jsonb)
 RETURNS uuid
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
DECLARE
  v_id uuid := COALESCE(NULLIF(_payload->>'id', '')::uuid, gen_random_uuid());
  v_student_email text := NULLIF(lower(btrim(_payload->>'student_email')), '');
  v_parent_email  text := NULLIF(lower(btrim(_payload->>'parent_email')), '');
  v_cat record;
  v_essay text := NULLIF(_payload->>'motivation_essay', '');
  q_essay constant text := 'Why do you want to join this program? What do you hope to achieve?';
  v_target public.sirluther_applications%ROWTYPE;
BEGIN
-- ... truncated: showing first 15 of 91 lines
```

#### research.submit_public_masterclass_registration(_payload jsonb)

```sql
CREATE OR REPLACE FUNCTION research.submit_public_masterclass_registration(_payload jsonb)
 RETURNS uuid
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
DECLARE
  v_id uuid := COALESCE(NULLIF(_payload->>'id', '')::uuid, gen_random_uuid());
  v_email text := NULLIF(btrim(_payload->>'email'), '');
  v_sessions text[] := COALESCE(
    ARRAY(SELECT jsonb_array_elements_text(COALESCE(_payload->'selected_sessions', '[]'::jsonb))),
    ARRAY[]::text[]
  );
BEGIN
  IF v_email IS NULL OR v_email !~* '^[^\s@]+@[^\s@]+\.[^\s@]+$' THEN
-- ... truncated: showing first 15 of 64 lines
```

#### research.tg_default_utm()

```sql
CREATE OR REPLACE FUNCTION research.tg_default_utm()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
DECLARE frozen jsonb; fd jsonb;
BEGIN
  IF TG_TABLE_NAME <> 'sirluther_applications' THEN
    NEW.utm_source := research.attribution_value(NEW.utm_source);
    NEW.utm_medium := research.attribution_value(NEW.utm_medium);
    NEW.utm_campaign := research.attribution_value(NEW.utm_campaign);
    RETURN NEW;
  END IF;
  fd := coalesce(NEW.form_data, '{}'::jsonb);
-- ... truncated: showing first 15 of 45 lines
```

#### research.tg_sheet_safe_phone()

```sql
CREATE OR REPLACE FUNCTION research.tg_sheet_safe_phone()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
DECLARE
  col_name text;
  row_json jsonb;
  current_value text;
  safe_value text;
  country_value text;
BEGIN
  row_json := to_jsonb(NEW);
  country_value := NULLIF(btrim(coalesce(row_json ->> 'country', '')), '');

  FOREACH col_name IN ARRAY ARRAY['phone','student_phone','parent_phone','student_mobile','parent_mobile','mobile','mobile_number','contact_number','wa_number']
  LOOP
    IF row_json ? col_name THEN
      current_value := row_json ->> col_name;
      IF current_value IS NOT NULL AND btrim(current_value) <> '' THEN
        safe_value := research.normalize_phone_digits(current_value, country_value);
        row_json := jsonb_set(row_json, ARRAY[col_name], coalesce(to_jsonb(safe_value), 'null'::jsonb), true);
      END IF;
    END IF;
  END LOOP;

  NEW := jsonb_populate_record(NEW, row_json);
  RETURN NEW;
END;
$function$

```

#### research.tg_standard_application_format()

```sql
CREATE OR REPLACE FUNCTION research.tg_standard_application_format()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
BEGIN
  NEW.student_name := regexp_replace(btrim(NEW.student_name), '\s+', ' ', 'g');
  NEW.student_email := lower(btrim(NEW.student_email));
  NEW.parent_email := nullif(lower(btrim(NEW.parent_email)), '');
  NEW.counsellor_email := nullif(lower(btrim(NEW.counsellor_email)), '');
  NEW.currency := upper(coalesce(nullif(btrim(NEW.currency),''), 'USD'));
  NEW.scholarship_code := nullif(upper(btrim(NEW.scholarship_code)), '');
  IF NEW.payment_status = 'unpaid' THEN NEW.payment_status := 'pending'; END IF;
  RETURN NEW;
END $function$

```

#### research.update_updated_at_column()

```sql
CREATE OR REPLACE FUNCTION research.update_updated_at_column()
 RETURNS trigger
 LANGUAGE plpgsql
 SECURITY DEFINER
 SET search_path TO 'research', 'public'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END; $function$

```

## leaders (6 functions)

| function | args | returns | lang | security | volatility | config | EXECUTE granted to |
|---|---|---|---|---|---|---|---|
| has_role | _user_id uuid, _role leaders.app_role | boolean | sql | invoker | stable | search_path=leaders | anon authenticated |
| open_level_number | _user_id uuid | integer | sql | invoker | stable | search_path=leaders | anon authenticated |
| reviews_cohort | _user_id uuid, _cohort_id uuid | boolean | sql | invoker | stable | search_path=leaders | anon authenticated |
| set_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=leaders, public, extensions, pg_temp | anon authenticated |
| sl_touch |  | trigger | plpgsql | invoker | volatile | search_path=leaders, public, extensions, pg_temp | anon authenticated |
| touch_updated_at |  | trigger | plpgsql | invoker | volatile | search_path=leaders | anon authenticated |

### leaders: bodies

#### leaders.has_role(_user_id uuid, _role leaders.app_role)

```sql
CREATE OR REPLACE FUNCTION leaders.has_role(_user_id uuid, _role leaders.app_role)
 RETURNS boolean
 LANGUAGE sql
 STABLE
 SET search_path TO 'leaders'
AS $function$
  SELECT EXISTS (SELECT 1 FROM leaders.user_roles WHERE user_id = _user_id AND role = _role)
$function$

```

#### leaders.open_level_number(_user_id uuid)

```sql
CREATE OR REPLACE FUNCTION leaders.open_level_number(_user_id uuid)
 RETURNS integer
 LANGUAGE sql
 STABLE
 SET search_path TO 'leaders'
AS $function$
  SELECT COALESCE((SELECT count(*) FROM leaders.level_progress
                   WHERE ambassador_id = _user_id AND status = 'cleared'), 0) + 1
$function$

```

#### leaders.reviews_cohort(_user_id uuid, _cohort_id uuid)

```sql
CREATE OR REPLACE FUNCTION leaders.reviews_cohort(_user_id uuid, _cohort_id uuid)
 RETURNS boolean
 LANGUAGE sql
 STABLE
 SET search_path TO 'leaders'
AS $function$
  SELECT EXISTS (SELECT 1 FROM leaders.cohort_reviewers WHERE user_id = _user_id AND cohort_id = _cohort_id)
$function$

```

#### leaders.set_updated_at()

```sql
CREATE OR REPLACE FUNCTION leaders.set_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'leaders', 'public', 'extensions', 'pg_temp'
AS $function$ begin new.updated_at = now(); return new; end $function$

```

#### leaders.sl_touch()

```sql
CREATE OR REPLACE FUNCTION leaders.sl_touch()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'leaders', 'public', 'extensions', 'pg_temp'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END $function$

```

#### leaders.touch_updated_at()

```sql
CREATE OR REPLACE FUNCTION leaders.touch_updated_at()
 RETURNS trigger
 LANGUAGE plpgsql
 SET search_path TO 'leaders'
AS $function$
BEGIN NEW.updated_at = now(); RETURN NEW; END; $function$

```

