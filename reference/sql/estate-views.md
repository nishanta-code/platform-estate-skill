# Shared estate views

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

## affiliate (1 views)

### affiliate.all_applications

reloptions: `(none)`

```sql
 SELECT 'group'::text AS platform,
    id,
    student_id,
    created_at,
    program_code,
    program_name,
    student_name,
    student_email,
    school_name,
    country,
    utm_source,
    utm_medium,
    utm_campaign,
    scholarship_code,
    payment_status,
    amount,
    currency,
    payment_link_id
   FROM applications;
```

## premed (11 views)

### premed.all_cohorts

reloptions: `security_invoker=on`

```sql
 SELECT c.id,
    c.program_id,
    c.cohort_key,
    c.cohort_label,
    c.program_code,
    c.start_date,
    c.end_date,
    c.application_deadline,
    c.seats_total,
    NULL::integer AS seats_filled,
    false AS is_cancelled,
    c.created_at,
    c.updated_at
   FROM all_cohorts c
     JOIN all_programs p ON p.id = c.program_id
  WHERE p.partner_id = (( SELECT premed.partner_id() AS partner_id));
```

### premed.all_masterclasses

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_id,
    cohort_id,
    slug,
    title,
    description,
    series,
    session_number,
    audience,
    scheduled_at,
    duration_minutes,
    display_timezone,
    host_mentor_id,
    host_name,
    learning_outcomes,
    apply_url,
    NULL::text AS recording_url,
    capacity,
    page_copy,
    is_published,
    false AS is_cancelled,
    created_at,
    updated_at
   FROM all_masterclasses
  WHERE partner_id = (( SELECT premed.partner_id() AS partner_id));
```

### premed.all_mentors

reloptions: `security_invoker=on`

```sql
 SELECT id,
    full_name,
    bio,
    university,
    image_url,
    is_active,
    created_at,
    updated_at,
    country,
    updated_by_platform
   FROM all_mentors
  WHERE is_active;
```

### premed.all_partners

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_slug,
    org_name,
    description,
    website,
    logo_url,
    country,
    contact_name,
    contact_phone,
    offerings,
    approval_status,
    approved_at,
    is_active,
    created_at,
    updated_at
   FROM all_partners
  WHERE partner_slug = 'future-premed'::text;
```

### premed.all_programs

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_name,
    short_description,
    description,
    categories,
    '{}'::text[] AS target_categories,
    '{}'::text[] AS tags,
    format,
    location,
    duration,
    NULL::integer AS age_min,
    NULL::integer AS age_max,
    eligibility_grades,
    sticker_price,
    early_bird_price,
    price_currency,
    pay_slug,
    cover_image_url,
    program_url,
    apply_url,
    '{}'::text[] AS product_match_keywords,
    is_active,
    created_at,
    updated_at
   FROM all_programs
  WHERE partner_id = (( SELECT premed.partner_id() AS partner_id));
```

### premed.premed_masterclass_regs

reloptions: `security_invoker=on`

```sql
 SELECT id,
    sessions AS selected_sessions,
    masterclass_ids,
    attendee_type AS role,
    full_name,
    email,
    phone,
    designation,
    student_name,
    parent_name,
    parent_email,
    parent_phone,
    school AS school_name,
    grade,
    country,
    status,
    category,
    utm_source,
    utm_medium,
    utm_campaign,
    consent,
    meta,
    created_at
   FROM premed_masterclass_regs r;
```

### premed.masterclass_registration_sessions

reloptions: `security_invoker=on`

```sql
 SELECT md5((r.id::text || ':'::text) || m.mid::text)::uuid AS id,
    r.id AS registration_id,
    m.mid AS masterclass_id,
    ( SELECT mc.title
           FROM all_masterclasses mc
          WHERE mc.id = m.mid) AS session_label,
    'masterclass_ids'::text AS resolution_source,
    false AS needs_review,
    r.created_at
   FROM premed_masterclass_regs r
     CROSS JOIN LATERAL unnest(COALESCE(r.masterclass_ids, '{}'::uuid[])) m(mid);
```

### premed.masterclass_sessions

reloptions: `security_invoker=on` -- has INSTEAD OF trigger(s): masterclass_sessions_write

```sql
 SELECT id,
    slug,
    title,
    page_copy ->> 'subtitle'::text AS subtitle,
    description,
    scheduled_at,
    duration_minutes,
    join_url AS zoom_url,
    host_name,
    page_copy ->> 'host_bio'::text AS host_bio,
        CASE
            WHEN false THEN 'cancelled'::text
            ELSE COALESCE(page_copy ->> 'status'::text, 'upcoming'::text)
        END AS status,
    is_published,
    COALESCE((page_copy ->> 'live_mode'::text)::boolean, false) AS live_mode,
    NULL::text AS recording_url,
    page_copy ->> 'slides_url'::text AS slides_url,
    COALESCE(page_copy -> 'agenda'::text, '[]'::jsonb) AS agenda,
    learning_outcomes,
    page_copy ->> 'cover_image_url'::text AS cover_image_url,
    created_at,
    updated_at,
    passcode,
    page_copy ->> 'eyebrow'::text AS eyebrow,
    COALESCE(page_copy ->> 'category'::text, audience) AS category
   FROM all_masterclasses m
  WHERE partner_id = (( SELECT premed.partner_id() AS partner_id));
```

### premed.masterclass_sessions_public

reloptions: `security_invoker=on`

```sql
 WITH ranked AS (
         SELECT m.id,
            m.slug,
            m.title,
            m.page_copy,
            m.description,
            m.scheduled_at,
            m.duration_minutes,
            m.host_name,
            m.learning_outcomes,
            m.is_published,
            m.created_at,
            m.updated_at,
            COALESCE(o.series_name, m.series) AS resolved_series,
            row_number() OVER (PARTITION BY (COALESCE(o.series_name, m.series)) ORDER BY m.scheduled_at) AS series_index
           FROM all_masterclasses m
             LEFT JOIN premed.masterclass_series_overrides o ON o.masterclass_id = m.id
          WHERE m.partner_id = (( SELECT premed.partner_id() AS partner_id)) AND m.is_published = true
        )
 SELECT id,
    slug,
    title,
    page_copy ->> 'subtitle'::text AS subtitle,
    description,
    scheduled_at,
    duration_minutes,
    host_name,
    page_copy ->> 'host_bio'::text AS host_bio,
        CASE
            WHEN false THEN 'cancelled'::text
            ELSE COALESCE(page_copy ->> 'status'::text, 'upcoming'::text)
        END AS status,
    is_published,
    COALESCE((page_copy ->> 'live_mode'::text)::boolean, false) AS live_mode,
    NULL::text AS recording_url,
    page_copy ->> 'slides_url'::text AS slides_url,
    COALESCE(page_copy -> 'agenda'::text, '[]'::jsonb) AS agenda,
    COALESCE(learning_outcomes, '[]'::jsonb) AS learning_outcomes,
    page_copy ->> 'cover_image_url'::text AS cover_image_url,
    'SESSION '::text || series_index AS eyebrow,
-- ... (truncated at 40 of 45 lines)
```

### premed.program_cohorts

reloptions: `security_invoker=on`

```sql
 SELECT c.id,
    s.program_key,
    p.program_name AS program_label,
    COALESCE(s.cohort_key, c.cohort_key) AS cohort_key,
    c.cohort_label,
    c.start_date,
    c.end_date,
    c.application_deadline,
    s.price_cents,
    s.currency,
    s.capacity,
    NULL::integer AS seats_filled,
    s.payment_link_url,
    s.mentor_id,
    s.is_active,
    s.sort_order,
    false AS is_cancelled,
    c.created_at,
    c.updated_at
   FROM all_cohorts c
     JOIN premed.cohort_settings s ON s.cohort_id = c.id
     LEFT JOIN all_programs p ON p.id = c.program_id;
```

### premed.programme_applications_read

reloptions: `security_invoker=true,security_barrier=true`

```sql
 SELECT id,
    student_name,
    student_email,
    parent_email,
    school,
    grade,
    cohort_id,
    cohort,
    program_code,
    program_name,
    payment_status,
    registration_status,
    app_status,
    created_at,
    applied_date,
    metadata
   FROM premed.read_programme_applications() read_programme_applications(id, student_name, student_email, parent_email, school, grade, cohort_id, cohort, program_code, program_name, payment_status, registration_status, app_status, created_at, applied_date, metadata);
```

## finance (6 views)

### finance.all_cohorts

reloptions: `security_invoker=on`

```sql
 SELECT id,
    program_id,
    cohort_key,
    program_code,
    cohort_label,
    start_date,
    end_date,
    application_deadline,
    seats_total,
    NULL::integer AS seats_filled,
    false AS is_cancelled,
    created_at,
    updated_at
   FROM all_cohorts;
```

### finance.all_masterclasses

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_id,
    cohort_id,
    slug,
    title,
    description,
    series,
    session_number,
    audience,
    scheduled_at,
    duration_minutes,
    display_timezone,
    host_mentor_id,
    host_name,
    learning_outcomes,
    apply_url,
    NULL::text AS recording_url,
    capacity,
    is_published,
    false AS is_cancelled,
    created_at,
    updated_at
   FROM all_masterclasses m;
```

### finance.all_mentors

reloptions: `security_invoker=true`

```sql
 SELECT id,
    full_name AS name,
    university,
    image_url,
    is_active,
    created_at,
    updated_at
   FROM all_mentors;
```

### finance.all_partners

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_slug,
    org_name,
    description,
    website,
    logo_url,
    country,
    contact_name,
    contact_phone,
    offerings,
    approval_status,
    approved_at,
    is_active,
    created_at,
    updated_at
   FROM all_partners;
```

### finance.all_programs

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_name,
    short_description,
    description,
    categories,
    '{}'::text[] AS target_categories,
    '{}'::text[] AS tags,
    format,
    location,
    duration,
    NULL::integer AS age_min,
    NULL::integer AS age_max,
    eligibility_grades,
    sticker_price,
    early_bird_price,
    price_currency,
    cover_image_url,
    program_url,
    apply_url,
    pay_slug,
    '{}'::text[] AS product_match_keywords,
    is_active,
    created_at,
    updated_at
   FROM all_programs;
```

### finance.stripe_payments

reloptions: `security_invoker=true`

```sql
 SELECT id,
    created_date_utc,
    customer_email,
    amount,
    currency,
    checkout_line_item_summary,
    payment_intent_id,
    status,
    card_name,
    plink,
    student_name,
    student_email,
    school_name,
    program_code,
    metadata,
    processing_status,
    reference,
    payout_id,
    export_status,
    program_type,
    utm_source,
    utm_medium,
    utm_campaign
   FROM stripe_payments;
```

## finedu (9 views)

### finedu.admin_cohorts

reloptions: `security_invoker=true`

```sql
 SELECT c.id,
    c.program_id,
    c.cohort_key,
    c.cohort_label,
    c.program_code,
    c.start_date,
    c.end_date,
    c.application_deadline,
    c.seats_total,
    NULL::integer AS seats_filled,
    false AS is_cancelled,
    c.created_at,
    c.updated_at,
    m.program_key,
    m.program_label,
    m.duration_days,
    p.sticker_price,
    p.price_currency,
    NOT COALESCE(false, false) AND c.start_date >= CURRENT_DATE AS is_active,
    0::bigint AS application_count
   FROM all_cohorts c
     JOIN all_programs p ON p.id = c.program_id
     LEFT JOIN finedu.program_map m ON m.program_id = c.program_id
  WHERE p.partner_id = finedu.partner_id();
```

### finedu.admin_masterclasses

reloptions: `security_invoker=true`

```sql
 SELECT m.id,
    m.program_id,
    m.cohort_id,
    m.slug,
    m.title,
    m.description,
    m.series,
    m.session_number,
    m.audience,
    m.scheduled_at,
    m.duration_minutes,
    m.display_timezone,
    m.host_name,
    m.learning_outcomes,
    m.join_url,
    m.passcode,
    m.apply_url,
    NULL::text AS recording_url,
    m.capacity,
    m.is_published,
    false AS is_cancelled,
    m.page_copy,
    pm.program_key,
    pm.program_label,
    c.cohort_key,
    c.cohort_label,
    c.start_date AS cohort_start_date,
    ( SELECT count(*) AS count
           FROM finedu.finedu_masterclass_regs r
          WHERE r.session_key = m.slug OR (m.slug = ANY (COALESCE(r.session_keys, ARRAY[]::text[])))) AS registration_count,
    m.created_at,
    m.updated_at
   FROM all_masterclasses m
     LEFT JOIN finedu.program_map pm ON pm.program_id = m.program_id
     LEFT JOIN all_cohorts c ON c.id = m.cohort_id
  WHERE m.partner_id = finedu.partner_id();
```

### finedu.admin_programs

reloptions: `security_invoker=true`

```sql
 SELECT p.id,
    p.program_name,
    p.short_description,
    p.description,
    p.duration,
    NULL::integer AS age_min,
    NULL::integer AS age_max,
    p.eligibility_grades,
    p.sticker_price,
    p.early_bird_price,
    p.price_currency,
    p.format,
    p.location,
    p.program_url,
    p.apply_url,
    p.pay_slug,
    p.is_active,
    p.created_at,
    p.updated_at,
    m.program_key,
    m.program_label,
    m.code_prefix,
    m.duration_days,
    ( SELECT count(*) AS count
           FROM all_cohorts c
          WHERE c.program_id = p.id) AS cohort_count
   FROM all_programs p
     LEFT JOIN finedu.program_map m ON m.program_id = p.id
  WHERE p.partner_id = finedu.partner_id();
```

### finedu.all_cohorts

reloptions: `security_invoker=true`

```sql
 SELECT c.id,
    c.program_id,
    c.cohort_key,
    c.program_code,
    c.start_date,
    c.end_date,
    c.application_deadline,
    c.seats_total,
    NULL::integer AS seats_filled,
    false AS is_cancelled,
    c.created_at,
    c.updated_at,
    c.cohort_label
   FROM all_cohorts c
     JOIN all_programs p ON p.id = c.program_id
  WHERE p.partner_id = (( SELECT all_partners.id
           FROM all_partners
          WHERE all_partners.partner_slug = 'finedu-ai'::text));
```

### finedu.all_masterclasses

reloptions: `security_invoker=true`

```sql
 SELECT id,
    partner_id,
    program_id,
    cohort_id,
    slug,
    title,
    description,
    series,
    session_number,
    audience,
    scheduled_at,
    duration_minutes,
    display_timezone,
    host_mentor_id,
    host_name,
    learning_outcomes,
    apply_url,
    NULL::text AS recording_url,
    capacity,
    is_published,
    false AS is_cancelled,
    created_at,
    updated_at
   FROM all_masterclasses m
  WHERE partner_id = (( SELECT all_partners.id
           FROM all_partners
          WHERE all_partners.partner_slug = 'finedu-ai'::text));
```

### finedu.all_mentors

reloptions: `security_invoker=true`

```sql
 SELECT id,
    full_name,
    bio,
    university,
    image_url,
    is_active,
    created_at,
    updated_at,
    email,
    country,
    updated_by_platform
   FROM all_mentors
  WHERE is_active;
```

### finedu.all_partners

reloptions: `security_invoker=true`

```sql
 SELECT id,
    partner_slug,
    org_name,
    description,
    website,
    logo_url,
    country,
    contact_name,
    contact_phone,
    offerings,
    approval_status,
    approved_at,
    is_active,
    created_at,
    updated_at
   FROM all_partners
  WHERE partner_slug = 'finedu-ai'::text;
```

### finedu.all_programs

reloptions: `security_invoker=true`

```sql
 SELECT id,
    partner_id,
    program_name,
    short_description,
    description,
    categories,
    '{}'::text[] AS target_categories,
    '{}'::text[] AS tags,
    format,
    location,
    duration,
    NULL::integer AS age_min,
    NULL::integer AS age_max,
    eligibility_grades,
    sticker_price,
    early_bird_price,
    price_currency,
    cover_image_url,
    program_url,
    apply_url,
    '{}'::text[] AS product_match_keywords,
    is_active,
    created_at,
    updated_at
   FROM all_programs p
  WHERE partner_id = (( SELECT all_partners.id
           FROM all_partners
          WHERE all_partners.partner_slug = 'finedu-ai'::text));
```

### finedu.finedu_masterclass_regs

reloptions: `security_invoker=true`

```sql
 SELECT id,
    selected_sessions,
    session_key,
    session_keys,
    full_name,
    email,
    phone,
    country,
    role,
    grade,
    school_name,
    parent_name,
    parent_email,
    parent_phone,
    student_name,
    student_grade,
    designation,
    utm_source,
    utm_medium,
    utm_campaign,
    created_at,
    updated_at
   FROM finedu_masterclass_regs;
```

## ops (1 views)

### ops.program_catalogue

reloptions: `security_invoker=true`

```sql
 SELECT p.id,
    p.partner_id,
    ap.org_name AS partner_name,
    p.program_name AS title,
    p.description,
    p.short_description,
    p.sticker_price,
    p.duration,
    p.format,
    p.location,
    p.categories,
    p.eligibility_grades,
    p.cover_image_url,
    p.program_url,
        CASE
            WHEN p.is_active THEN 'live'::text
            ELSE 'draft'::text
        END AS status,
    p.apply_url,
    p.pay_slug,
    ap.partner_slug,
    COALESCE(( SELECT jsonb_agg(jsonb_build_object('name', COALESCE(c.cohort_label, c.cohort_key), 'cohort_id', c.id, 'program_code', c.program_code, 'start_date', c.start_date, 'end_date', c.end_date, 'application_deadline', c.application_deadline, 'seats_total', c.seats_total) ORDER BY c.start_date) AS jsonb_agg
           FROM all_cohorts c
          WHERE c.program_id = p.id AND COALESCE(c.end_date, c.start_date) >= CURRENT_DATE), '[]'::jsonb) AS cohorts
   FROM all_programs p
     LEFT JOIN all_partners ap ON ap.id = p.partner_id;
```

## prelaw (6 views)

### prelaw.all_cohorts

reloptions: `security_invoker=on`

```sql
 SELECT c.id,
    c.program_id,
    c.cohort_key,
    c.program_code,
    c.start_date,
    c.end_date,
    c.application_deadline,
    c.seats_total,
    NULL::integer AS seats_filled,
    false AS is_cancelled,
    c.created_at,
    c.updated_at,
    c.cohort_label
   FROM all_cohorts c
     JOIN all_programs p ON p.id = c.program_id
  WHERE p.partner_id = (( SELECT all_partners.id
           FROM all_partners
          WHERE all_partners.partner_slug = 'future-prelaw'::text));
```

### prelaw.all_masterclasses

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_id,
    cohort_id,
    slug,
    title,
    description,
    series,
    session_number,
    audience,
    scheduled_at,
    duration_minutes,
    display_timezone,
    host_mentor_id,
    host_name,
    learning_outcomes,
    join_url,
    passcode,
    apply_url,
    NULL::text AS recording_url,
    capacity,
    is_published,
    false AS is_cancelled,
    created_at,
    updated_at
   FROM all_masterclasses m
  WHERE partner_id = (( SELECT all_partners.id
           FROM all_partners
          WHERE all_partners.partner_slug = 'future-prelaw'::text));
```

### prelaw.all_mentors

reloptions: `security_invoker=on`

```sql
 SELECT id,
    full_name,
    bio,
    university,
    image_url,
    is_active,
    created_at,
    updated_at,
    email,
    country,
    updated_by_platform
   FROM all_mentors
  WHERE is_active;
```

### prelaw.all_partners

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_slug,
    org_name,
    description,
    website,
    logo_url,
    country,
    contact_name,
    contact_phone,
    offerings,
    approval_status,
    approved_at,
    is_active,
    created_at,
    updated_at
   FROM all_partners
  WHERE partner_slug = 'future-prelaw'::text;
```

### prelaw.all_programs

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_name,
    short_description,
    description,
    categories,
    '{}'::text[] AS target_categories,
    '{}'::text[] AS tags,
    format,
    location,
    duration,
    NULL::integer AS age_min,
    NULL::integer AS age_max,
    eligibility_grades,
    sticker_price,
    early_bird_price,
    price_currency,
    pay_slug,
    cover_image_url,
    program_url,
    apply_url,
    '{}'::text[] AS product_match_keywords,
    is_active,
    created_at,
    updated_at
   FROM all_programs p
  WHERE partner_id = (( SELECT all_partners.id
           FROM all_partners
          WHERE all_partners.partner_slug = 'future-prelaw'::text));
```

### prelaw.prelaw_masterclass_regs

reloptions: `(none)`

```sql
 SELECT id,
    created_at,
    sessions,
    masterclass_ids,
    attendee_type,
    full_name,
    email,
    phone,
    country,
    state,
    city,
    grade,
    school,
    hear_about,
    parent_name,
    parent_email,
    parent_phone,
    student_name,
    student_grade,
    designation,
    consent,
    source_url,
    referrer,
    user_agent,
    email_sent_at,
    utm_source,
    utm_medium,
    utm_campaign,
    promotes_program_code
   FROM prelaw_masterclass_regs;
```

## legacy_compat (1 views)

### legacy_compat.applications

reloptions: `(none)`

```sql
 SELECT id,
    student_id,
    created_at,
    program_code,
    program_name,
    program_category,
    student_name,
    student_email,
    student_mobile,
    student_dob,
    grade,
    parent_name,
    parent_email,
    parent_phone,
    school_name,
    country,
    state,
    city,
    counsellor_name,
    counsellor_email,
    NULL::text AS question1,
    NULL::text AS question2,
    NULL::text AS question3,
    NULL::text AS question4,
    visa_status,
    admin_notes,
    utm_source,
    utm_medium,
    utm_campaign,
    payment_status,
    amount,
    currency,
    payment_intent_id,
    payment_date,
    stripe_customer_id,
    submission_token,
    loa_sent_at,
    last_reminder_stage,
    last_reminder_sent_at,
    payment_mode,
-- ... (truncated at 40 of 49 lines)
```

## lms (8 views)

### lms.all_cohorts

reloptions: `security_invoker=on`

```sql
 SELECT id,
    program_id,
    cohort_key,
    program_code,
    cohort_label,
    start_date,
    end_date,
    application_deadline,
    seats_total,
    NULL::integer AS seats_filled,
    false AS is_cancelled,
    created_at,
    updated_at
   FROM all_cohorts;
```

### lms.all_masterclasses

reloptions: `security_invoker=true`

```sql
 SELECT m.id,
    m.slug,
    m.title,
    m.description,
    m.series,
    m.session_number,
    m.audience,
    m.scheduled_at,
    m.duration_minutes,
    m.display_timezone,
    m.scheduled_at + make_interval(mins => COALESCE(m.duration_minutes, 60)) AS ends_at,
    m.partner_id,
    p.org_name AS partner_name,
    p.logo_url AS partner_logo_url,
    m.program_id,
    pr.program_name,
    m.cohort_id,
    c.cohort_label,
    m.host_mentor_id,
    COALESCE(NULLIF(m.host_name, ''::text), mt.full_name) AS host_name,
    mt.bio AS host_bio,
    mt.university AS host_university,
    mt.image_url AS host_image_url,
    m.learning_outcomes,
    m.apply_url,
    NULL::text AS recording_url,
    m.capacity,
    m.is_published,
    false AS is_cancelled,
        CASE
            WHEN lms.am_i_registered_for(m.id) THEN m.join_url
            ELSE NULL::text
        END AS join_url,
        CASE
            WHEN lms.am_i_registered_for(m.id) THEN m.passcode
            ELSE NULL::text
        END AS passcode
   FROM all_masterclasses m
     LEFT JOIN all_partners p ON p.id = m.partner_id
     LEFT JOIN all_programs pr ON pr.id = m.program_id
-- ... (truncated at 40 of 42 lines)
```

### lms.all_mentors

reloptions: `security_invoker=on` -- has INSTEAD OF trigger(s): tg_all_mentors_view_del, tg_all_mentors_view_ins, tg_all_mentors_view_upd

```sql
 SELECT m.id,
    m.full_name,
    m.bio,
    m.university,
    m.image_url,
    m.is_active,
    COALESCE(s.display_order, 0) AS display_order,
    COALESCE(s.legacy_tags, '[]'::jsonb) AS legacy_tags,
    COALESCE(s.is_listed, true) AS is_listed,
    m.created_at,
    m.updated_at
   FROM all_mentors m
     LEFT JOIN lms.mentor_settings s ON s.mentor_id = m.id;
```

### lms.all_partners

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_slug,
    org_name,
    description,
    website,
    logo_url,
    country,
    contact_name,
    contact_phone,
    offerings,
    approval_status,
    approved_at,
    is_active,
    created_at,
    updated_at
   FROM all_partners;
```

### lms.all_programs

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_name,
    short_description,
    description,
    categories,
    '{}'::text[] AS target_categories,
    '{}'::text[] AS tags,
    format,
    location,
    duration,
    NULL::integer AS age_min,
    NULL::integer AS age_max,
    eligibility_grades,
    sticker_price,
    early_bird_price,
    price_currency,
    cover_image_url,
    program_url,
    apply_url,
    pay_slug,
    '{}'::text[] AS product_match_keywords,
    is_active,
    created_at,
    updated_at
   FROM all_programs;
```

### lms.masterclass_registration_counts

reloptions: `security_invoker=true`

```sql
 SELECT masterclass_id,
    registered_count
   FROM lms.get_masterclass_registration_counts() get_masterclass_registration_counts(masterclass_id, registered_count);
```

### lms.masterclass_registrations

reloptions: `security_invoker=true`

```sql
 SELECT platform_key,
    email,
    student_name,
    registered_at,
    session_label,
    masterclass_id
   FROM lms.get_masterclass_registrations() get_masterclass_registrations(platform_key, email, student_name, registered_at, session_label, masterclass_id);
```

### lms.whatsapp_blocklist

reloptions: `security_barrier=true,security_invoker=true`

```sql
 SELECT
        CASE
            WHEN phone ~ '^00'::text THEN substr(phone, 3)
            ELSE phone
        END AS phone,
    lms_block_reason AS reason,
    source_file,
    first_imported_at AS created_at,
    last_action_at AS updated_at,
    last_action_by AS created_by
   FROM portal.whatsapp_undeliverable_numbers
  WHERE blocked AND ((( SELECT auth.role() AS role)) = 'service_role'::text OR ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role));
```

## portal (4 views)

### portal.codes_dropdown

reloptions: `security_invoker=on`

```sql
 SELECT id,
    ref_type,
    code,
    name,
    created_at
   FROM finance.codes_dropdown;
```

### portal.applications

reloptions: `security_invoker=true`

```sql
 SELECT id,
    student_id,
    created_at,
    program_code,
    program_name,
    program_category,
    student_name,
    student_email,
    student_mobile,
    student_dob,
    grade,
    parent_name,
    parent_email,
    parent_phone,
    school_name,
    country,
    state,
    city,
    counsellor_name,
    counsellor_email,
    visa_status,
    admin_notes,
    utm_source,
    utm_medium,
    utm_campaign,
    payment_status,
    amount,
    currency,
    payment_intent_id,
    payment_date,
    stripe_customer_id,
    submission_token,
    loa_sent_at,
    last_reminder_stage,
    last_reminder_sent_at,
    payment_mode,
    form_template_id,
    source_flow,
    registration_status,
    registration_email_sent_at,
-- ... (truncated at 40 of 48 lines)
```

### portal.form_responses

reloptions: `(none)`

```sql
 SELECT id,
    form_name,
    data,
    created_at,
    updated_at
   FROM form_responses;
```

### portal.program_codes

reloptions: `security_invoker=on`

```sql
 SELECT id,
    program_code AS ref_code,
    category,
    organisation,
    program,
    school,
    country,
    batch,
    date,
    NULL::uuid AS vault_id,
    false AS is_stale,
    created_at AS last_synced_at,
    created_at,
    created_at AS updated_at
   FROM finance.program_codes c;
```

## fellowship (7 views)

### fellowship.all_cohorts

reloptions: `security_invoker=on` -- has INSTEAD OF trigger(s): all_cohorts_dml_trg

```sql
 SELECT c.id,
    c.program_id,
    c.cohort_key,
    c.program_code,
    c.start_date,
    c.end_date,
    c.application_deadline,
    c.seats_total,
    NULL::integer AS seats_filled,
    false AS is_cancelled,
    c.created_at,
    c.updated_at,
    COALESCE(s.cohort_label, c.cohort_label) AS cohort_label,
    p.program_name,
    s.program_key,
    pm.program_label,
    s.price_usd,
    s.payment_link_url,
    COALESCE(s.is_active, false) AS is_active,
    s.sort_order
   FROM all_cohorts c
     JOIN all_programs p ON p.id = c.program_id
     JOIN all_partners pa ON pa.id = p.partner_id AND pa.partner_slug = 'fellowship-fellowship'::text
     LEFT JOIN fellowship.cohort_settings s ON s.cohort_id = c.id
     LEFT JOIN fellowship.program_map pm ON pm.program_key = s.program_key;
```

### fellowship.all_masterclasses

reloptions: `(none)`

```sql
 SELECT m.id,
    m.partner_id,
    m.program_id,
    m.cohort_id,
    m.slug,
    m.title,
    m.description,
    m.series,
    m.session_number,
    m.audience,
    m.scheduled_at,
    m.duration_minutes,
    m.display_timezone,
    m.host_mentor_id,
    m.host_name,
    m.learning_outcomes,
    m.join_url,
    m.passcode,
    m.apply_url,
    NULL::text AS recording_url,
    m.capacity,
    m.is_published,
    false AS is_cancelled,
    m.created_at,
    m.updated_at,
    m.page_copy
   FROM all_masterclasses m
     JOIN all_partners pa ON pa.id = m.partner_id AND pa.partner_slug = 'fellowship-fellowship'::text;
```

### fellowship.all_mentors

reloptions: `(none)`

```sql
 SELECT id,
    full_name,
    bio,
    university,
    image_url,
    is_active,
    created_at,
    updated_at,
    email,
    country,
    updated_by_platform
   FROM all_mentors;
```

### fellowship.all_partners

reloptions: `(none)`

```sql
 SELECT id,
    partner_slug,
    org_name,
    description,
    website,
    logo_url,
    country,
    contact_name,
    contact_phone,
    offerings,
    approval_status,
    approved_at,
    is_active,
    created_at,
    updated_at
   FROM all_partners
  WHERE partner_slug = 'fellowship-fellowship'::text;
```

### fellowship.all_programs

reloptions: `(none)`

```sql
 SELECT p.id,
    p.partner_id,
    p.program_name,
    p.short_description,
    p.description,
    p.categories,
    '{}'::text[] AS target_categories,
    '{}'::text[] AS tags,
    p.format,
    p.location,
    p.duration,
    NULL::integer AS age_min,
    NULL::integer AS age_max,
    p.eligibility_grades,
    p.sticker_price,
    p.early_bird_price,
    p.price_currency,
    p.cover_image_url,
    p.program_url,
    p.apply_url,
    '{}'::text[] AS product_match_keywords,
    p.is_active,
    p.created_at,
    p.updated_at,
    p.pay_slug
   FROM all_programs p
     JOIN all_partners pa ON pa.id = p.partner_id AND pa.partner_slug = 'fellowship-fellowship'::text;
```

### fellowship.fellowship_masterclass_regs

reloptions: `security_barrier=true`

```sql
 SELECT id,
    program_key,
    full_name,
    email,
    phone,
    country,
    role,
    grade,
    school_name,
    parent_name,
    parent_email,
    parent_phone,
    child_name,
    child_grade,
    designation,
    sessions,
    email_sent,
    email_sent_at,
    created_at,
    updated_at,
    synced_at,
    utm_source,
    utm_medium,
    utm_campaign,
    questions,
    utm_content,
    utm_term
   FROM fellowship_masterclass_regs;
```

### fellowship.week_without_walls_inquiries

reloptions: `(none)`

```sql
 SELECT id,
    created_at,
    updated_at,
    synced_at,
    school_name,
    contact_name,
    contact_role,
    email,
    phone,
    country,
    city,
    programs,
    student_count,
    grade_levels,
    preferred_dates,
    backup_dates,
    notes,
    source,
    email_sent,
    email_sent_at,
    utm_source,
    utm_medium,
    utm_campaign,
    utm_content,
    utm_term
   FROM week_without_walls_inquiries;
```

## public (5 views)

### public.all_masterclasses_public

reloptions: `security_invoker=on`

```sql
 SELECT m.id,
    m.partner_id,
    p.org_name AS partner_name,
    p.logo_url AS partner_logo_url,
    m.program_id,
    pr.program_name,
    m.cohort_id,
    c.cohort_label,
    c.start_date AS cohort_start_date,
    m.slug,
    m.title,
    m.description,
    m.series,
    m.session_number,
    m.audience,
    m.scheduled_at,
    m.duration_minutes,
    m.display_timezone,
    m.scheduled_at + ((m.duration_minutes || ' minutes'::text)::interval) AS ends_at,
    m.host_mentor_id,
    COALESCE(mt.full_name, m.host_name) AS host_name,
    mt.bio AS host_bio,
    mt.university AS host_university,
    mt.image_url AS host_image_url,
    m.learning_outcomes,
    m.apply_url,
    NULL::text AS recording_url,
    m.capacity,
        CASE
            WHEN false THEN 'cancelled'::text
            WHEN m.scheduled_at IS NULL THEN 'unscheduled'::text
            WHEN (m.scheduled_at + ((m.duration_minutes || ' minutes'::text)::interval)) < now() THEN 'completed'::text
            WHEN m.scheduled_at <= now() THEN 'live'::text
            ELSE 'upcoming'::text
        END AS status
   FROM all_masterclasses m
     JOIN all_partners p ON p.id = m.partner_id
     LEFT JOIN all_programs pr ON pr.id = m.program_id
     LEFT JOIN all_cohorts c ON c.id = m.cohort_id
     LEFT JOIN all_mentors mt ON mt.id = m.host_mentor_id
-- ... (truncated at 40 of 41 lines)
```

### public.codes_dropdown

reloptions: `security_invoker=true`

```sql
 SELECT id,
    ref_type,
    code,
    name,
    created_at
   FROM portal.codes_dropdown;
```

### public.program_codes

reloptions: `security_invoker=true`

```sql
 SELECT id,
    ref_code,
    category,
    organisation,
    program,
    school,
    country,
    batch,
    date,
    vault_id,
    is_stale,
    last_synced_at,
    created_at,
    updated_at
   FROM portal.program_codes;
```

### public.sirluther_questionnaire_response

reloptions: `security_invoker=true`

```sql
 SELECT id,
    created_at,
    student_name,
    student_grade,
    student_email,
    parent_email,
    enjoy_subjects_top3,
    enjoy_subject_one,
    rating_experiments,
    rating_reading_articles,
    rating_arguing_position,
    rating_building_coding,
    rating_interviewing_people,
    rating_numbers_charts,
    subject_outside_class,
    noticed_problem,
    last_curiosity_lookup,
    question_type,
    community_problem,
    help_audience,
    writing_comfort,
    read_research_paper,
    tools_used,
    work_preference,
    hours_per_week,
    program_goals,
    prior_research,
    existing_work,
    topic_certainty,
    page_url,
    utm_source,
    utm_medium,
    utm_campaign
   FROM research.questionnaire_responses;
```

### public.sirluther_school_nominations

reloptions: `security_invoker=true`

```sql
 SELECT n.id AS nomination_id,
    n.created_at,
    n.school_name,
    n.city,
    n.country,
    n.nominator_name,
    n.nominator_role,
    n.nominator_email,
    n.nominator_phone,
    s.student_position,
    s.student_name,
    s.student_grade,
    s.contact_email,
    n.program_name,
    n.page_url,
    n.utm_source,
    n.utm_medium,
    n.utm_campaign
   FROM research.school_nominations n
     LEFT JOIN research.school_nomination_students s ON s.nomination_id = n.id
  ORDER BY n.created_at DESC, s.student_position;
```

## testprep (7 views)

### testprep.all_cohorts

reloptions: `security_invoker=on`

```sql
 SELECT c.id,
    c.program_id,
    c.cohort_key,
    c.program_code,
    c.start_date,
    c.end_date,
    c.application_deadline,
    c.seats_total,
    NULL::integer AS seats_filled,
    false AS is_cancelled,
    c.created_at,
    c.updated_at,
    c.cohort_label
   FROM all_cohorts c
     JOIN all_programs p ON p.id = c.program_id
  WHERE p.partner_id = '3ade972c-03ca-4d84-9f41-29c89750d866'::uuid;
```

### testprep.all_masterclasses

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_id,
    cohort_id,
    slug,
    title,
    description,
    series,
    session_number,
    audience,
    scheduled_at,
    duration_minutes,
    display_timezone,
    host_mentor_id,
    host_name,
    learning_outcomes,
    join_url,
    passcode,
    apply_url,
    NULL::text AS recording_url,
    capacity,
    is_published,
    false AS is_cancelled,
    created_at,
    updated_at,
    page_copy
   FROM all_masterclasses
  WHERE partner_id = '3ade972c-03ca-4d84-9f41-29c89750d866'::uuid;
```

### testprep.all_mentors

reloptions: `security_invoker=on`

```sql
 SELECT id,
    full_name,
    bio,
    university,
    image_url,
    country,
    is_active,
    created_at,
    updated_at
   FROM all_mentors
  WHERE is_active IS TRUE;
```

### testprep.all_partners

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_slug,
    org_name,
    description,
    website,
    logo_url,
    country,
    contact_name,
    contact_phone,
    offerings,
    approval_status,
    approved_at,
    is_active,
    created_at,
    updated_at
   FROM all_partners
  WHERE id = '3ade972c-03ca-4d84-9f41-29c89750d866'::uuid;
```

### testprep.all_programs

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_name,
    short_description,
    description,
    categories,
    '{}'::text[] AS target_categories,
    '{}'::text[] AS tags,
    format,
    location,
    duration,
    NULL::integer AS age_min,
    NULL::integer AS age_max,
    eligibility_grades,
    sticker_price,
    early_bird_price,
    price_currency,
    cover_image_url,
    program_url,
    apply_url,
    '{}'::text[] AS product_match_keywords,
    is_active,
    created_at,
    updated_at,
    pay_slug
   FROM all_programs
  WHERE partner_id = '3ade972c-03ca-4d84-9f41-29c89750d866'::uuid;
```

### testprep.program_cohorts

reloptions: `security_invoker=on` -- has INSTEAD OF trigger(s): program_cohorts_del, program_cohorts_ins, program_cohorts_upd

```sql
 SELECT c.id,
    'main'::text AS program_key,
    regexp_replace(p.program_name, '^SAT Instinct\s*[-–]\s*'::text, ''::text) AS program_label,
    c.cohort_key,
    COALESCE(NULLIF(c.cohort_label, ''::text), to_char(c.start_date::timestamp with time zone, 'FMMonth YYYY'::text)) AS cohort_label,
    c.start_date,
    c.end_date,
    c.application_deadline,
    c.program_code,
    c.seats_total,
    NULL::integer AS seats_filled,
    false IS NOT TRUE AND c.start_date IS NOT NULL AND c.start_date >= CURRENT_DATE AS is_active,
    c.created_at,
    c.updated_at
   FROM all_cohorts c
     JOIN all_programs p ON p.id = c.program_id
  WHERE p.partner_id = '3ade972c-03ca-4d84-9f41-29c89750d866'::uuid;
```

### testprep.testprep_masterclass_regs

reloptions: `security_invoker=true`

```sql
 SELECT id,
    full_name,
    email,
    phone,
    country,
    role,
    selected_sessions,
    grade,
    school_name,
    parent_name,
    parent_email,
    parent_phone,
    student_name,
    student_grade,
    designation,
    utm_source,
    utm_medium,
    utm_campaign,
    email_sent,
    created_at,
    updated_at,
    utm_content,
    utm_term
   FROM testprep_masterclass_regs;
```

## research (7 views)

### research.all_cohorts

reloptions: `security_invoker=true`

```sql
 SELECT c.id,
    c.program_id,
    c.cohort_key,
    c.program_code,
    c.start_date,
    c.end_date,
    c.application_deadline,
    c.seats_total,
    NULL::integer AS seats_filled,
    false AS is_cancelled,
    c.created_at,
    c.updated_at,
    c.cohort_label
   FROM all_cohorts c
     JOIN all_programs p ON p.id = c.program_id
     JOIN all_partners pt ON pt.id = p.partner_id
  WHERE pt.partner_slug = 'sir-luther-center'::text;
```

### research.all_masterclasses

reloptions: `security_invoker=on`

```sql
 SELECT id,
    partner_id,
    program_id,
    cohort_id,
    slug,
    title,
    description,
    series,
    session_number,
    audience,
    scheduled_at,
    duration_minutes,
    display_timezone,
    host_mentor_id,
    host_name,
    learning_outcomes,
    apply_url,
    NULL::text AS recording_url,
    capacity,
    is_published,
    false AS is_cancelled,
    created_at,
    updated_at
   FROM all_masterclasses m
  WHERE partner_id = (( SELECT all_partners.id
           FROM all_partners
          WHERE all_partners.partner_slug = 'sir-luther-center'::text));
```

### research.all_masterclasses_public

reloptions: `security_invoker=true`

```sql
 SELECT v.id,
    v.partner_id,
    v.partner_name,
    v.partner_logo_url,
    v.program_id,
    v.program_name,
    v.cohort_id,
    v.cohort_label,
    v.cohort_start_date,
    v.slug,
    v.title,
    v.description,
    v.series,
    v.session_number,
    v.audience,
    v.scheduled_at,
    v.duration_minutes,
    v.display_timezone,
    v.ends_at,
    v.host_mentor_id,
    v.host_name,
    v.host_bio,
    v.host_university,
    v.host_image_url,
    v.learning_outcomes,
    v.apply_url,
    NULL::text AS recording_url,
    v.capacity,
    v.status
   FROM all_masterclasses_public v
     JOIN all_partners pt ON pt.id = v.partner_id
  WHERE pt.partner_slug = 'sir-luther-center'::text;
```

### research.all_mentors

reloptions: `security_invoker=on`

```sql
 SELECT id,
    full_name,
    bio,
    university,
    image_url,
    is_active,
    created_at,
    updated_at,
    email,
    country,
    updated_by_platform
   FROM all_mentors m;
```

### research.all_partners

reloptions: `security_invoker=true`

```sql
 SELECT id,
    partner_slug,
    org_name,
    description,
    website,
    logo_url,
    country,
    contact_name,
    contact_phone,
    offerings,
    approval_status,
    approved_at,
    is_active,
    created_at,
    updated_at
   FROM all_partners
  WHERE partner_slug = 'sir-luther-center'::text;
```

### research.all_programs

reloptions: `security_invoker=true`

```sql
 SELECT p.id,
    p.partner_id,
    p.program_name,
    p.short_description,
    p.description,
    p.categories,
    '{}'::text[] AS target_categories,
    '{}'::text[] AS tags,
    p.format,
    p.location,
    p.duration,
    NULL::integer AS age_min,
    NULL::integer AS age_max,
    p.eligibility_grades,
    p.sticker_price,
    p.early_bird_price,
    p.price_currency,
    p.cover_image_url,
    p.program_url,
    p.apply_url,
    '{}'::text[] AS product_match_keywords,
    p.is_active,
    p.created_at,
    p.updated_at,
    p.pay_slug
   FROM all_programs p
     JOIN all_partners pt ON pt.id = p.partner_id
  WHERE pt.partner_slug = 'sir-luther-center'::text;
```

### research.research_masterclass_regs

reloptions: `security_invoker=true`

```sql
 SELECT id,
    full_name,
    email,
    phone,
    country,
    grade,
    role,
    selected_sessions,
    school_name,
    parent_name,
    parent_email,
    parent_phone,
    student_name,
    student_grade,
    designation,
    utm_source,
    utm_medium,
    utm_campaign,
    created_at,
    category
   FROM research_masterclass_regs;
```

## leaders (5 views)

### leaders.leaders_applications

reloptions: `(none)`

```sql
 SELECT id,
    full_name,
    parent_details,
    school,
    grade,
    city_country,
    email,
    phone,
    path,
    path_reason,
    declaration_confirmed,
    created_at,
    student_dob,
    country,
    state,
    city,
    parent_name,
    parent_phone,
    parent_email,
    utm_source,
    utm_medium,
    utm_campaign
   FROM leaders_applications;
```

### leaders.v_offering_cohorts

reloptions: `(none)`

```sql
 SELECT id,
    program_id,
    cohort_label,
    start_date,
    end_date,
    application_deadline
   FROM all_cohorts;
```

### leaders.v_offering_programs

reloptions: `(none)`

```sql
 SELECT p.id,
    p.partner_id,
    p.program_name,
    p.short_description,
    p.description,
    p.categories,
    p.format,
    p.location,
    p.duration,
    p.eligibility_grades,
    p.cover_image_url,
    p.program_url,
    p.apply_url,
    p.pay_slug,
    p.is_active,
    pa.org_name AS partner_name,
    pa.partner_slug
   FROM all_programs p
     LEFT JOIN all_partners pa ON pa.id = p.partner_id;
```

### leaders.v_offerings_cohorts

reloptions: `(none)`

```sql
 SELECT id,
    program_id,
    cohort_label,
    start_date,
    end_date,
    application_deadline
   FROM all_cohorts;
```

### leaders.v_offerings_programs

reloptions: `(none)`

```sql
 SELECT id,
    partner_id,
    program_name,
    short_description,
    description,
    categories,
    format,
    location,
    duration,
    eligibility_grades,
    cover_image_url,
    program_url,
    apply_url,
    pay_slug,
    is_active
   FROM all_programs;
```

