# Brain project: views and materialized views

Live snapshot taken 2026-11-24 from <DATA_REF>. Definitions via pg_get_viewdef(oid, true), truncated to the first 40 lines where longer (line count noted). "options:" shows reloptions, e.g. security_invoker=on/true (view runs with the caller's privileges/RLS) versus none (security definer semantics of the view owner).

Counts: group 32 views + 1 materialized view (mv_meeting_outcomes_all, refreshed hourly by cron job group-refresh-meeting-outcomes), migration 18, public 13, attribution 4, revenue_brain 4, core_ro 3, staging 2, leads 1, extensions 2 (pg_stat_statements views, omitted), vault 1 (decrypted_secrets, omitted).

## Schema: attribution

### attribution.v_attributed_applications (view)
```sql
 SELECT a.id AS application_uuid,
    a.source_table,
    NULLIF(a.raw ->> 'application_id'::text, ''::text) AS application_id,
    upper(NULLIF(btrim(a.raw ->> 'school_ref'::text), ''::text)) AS school_ref,
    upper(NULLIF(btrim(a.raw ->> 'contact_ref'::text), ''::text)) AS contact_ref,
    upper(NULLIF(btrim(a.raw ->> 'campaign_ref'::text), ''::text)) AS campaign_ref,
    a.utm_source,
    a.utm_medium,
    a.utm_campaign,
    lower(NULLIF(btrim(a.raw ->> 'utm_content'::text), ''::text)) AS utm_content,
    a.school_raw AS school_name_typed,
    a.applied_at,
    l.outreach_id,
    p.id AS payment_id,
    p.currency,
    p.amount_minor - COALESCE(p.refunded_minor, 0::bigint) AS net_minor,
    p.occurred_at AS paid_at
   FROM applications a
     LEFT JOIN attribution.v_outreach_links l ON (lower(btrim(a.raw ->> 'utm_content'::text)) = l.details_content OR lower(btrim(a.raw ->> 'utm_content'::text)) = l.papers_content OR lower(btrim(a.raw ->> 'utm_content'::text)) = l.forward_content) AND l.contact_id = upper(btrim(a.raw ->> 'contact_ref'::text))
     LEFT JOIN payments p ON p.provider_payment_id = a.provider_payment_id AND (p.status::text = ANY (ARRAY['paid'::text, 'partially_refunded'::text]))
  WHERE NULLIF(a.raw ->> 'school_ref'::text, ''::text) IS NOT NULL;
```

### attribution.v_circulation (view)
```sql
 WITH sc AS (
         SELECT c.school_id,
            o.campaign_id,
            count(*) FILTER (WHERE o.status = 'SENT'::text) AS sent,
            count(*) FILTER (WHERE o.reply_status = 'REPLIED'::text) AS replied
           FROM attribution.outreach o
             JOIN attribution.contacts c USING (contact_id)
          WHERE NOT o.is_test
          GROUP BY c.school_id, o.campaign_id
        ), ap AS (
         SELECT v_attributed_applications.school_ref AS school_id,
            v_attributed_applications.campaign_ref AS campaign_id,
            count(*) AS apps
           FROM attribution.v_attributed_applications
          GROUP BY v_attributed_applications.school_ref, v_attributed_applications.campaign_ref
        )
 SELECT s.school_id,
    k.campaign_id,
    COALESCE(sc.sent, 0::bigint) AS sent,
    COALESCE(sc.replied, 0::bigint) AS replied,
    COALESCE(ap.apps, 0::bigint) AS applications,
        CASE
            WHEN ov.status IS NOT NULL THEN ov.status
            WHEN COALESCE(ap.apps, 0::bigint) >= 2 THEN 'CIRCULATION INFERRED FROM APPLICATIONS'::text
            WHEN COALESCE(sc.replied, 0::bigint) > 0 THEN 'SCHOOL REPLIED'::text
            WHEN COALESCE(sc.sent, 0::bigint) > 0 THEN 'SENT TO SCHOOL'::text
            ELSE 'NOT SENT'::text
        END AS circulation_status
   FROM attribution.schools s
     CROSS JOIN attribution.campaigns k
     LEFT JOIN sc ON sc.school_id = s.school_id AND sc.campaign_id = k.campaign_id
     LEFT JOIN ap ON ap.school_id = s.school_id AND ap.campaign_id = k.campaign_id
     LEFT JOIN attribution.circulation_overrides ov ON ov.school_id = s.school_id AND ov.campaign_id = k.campaign_id;
```

### attribution.v_outreach_links (view)
```sql
 WITH o AS (
         SELECT o_1.outreach_id,
            o_1.contact_id,
            o_1.campaign_id,
            o_1.touch_no,
            o_1.status,
            o_1.approved_to_send,
            o_1.approved_by,
            o_1.approved_at,
            o_1.parallel_approved_by,
            o_1.subject,
            o_1.sent_at,
            o_1.gmail_message_id,
            o_1.gmail_thread_id,
            o_1.reply_status,
            o_1.last_reply_at,
            o_1.follow_up_due,
            o_1.suppressed,
            o_1.suppression_reason,
            o_1.is_test,
            o_1.created_by,
            o_1.created_at,
            c.school_id,
            c.full_name,
            c.email,
            c.utm_name,
            s.name AS school_name,
            s.utm_source,
            k.name AS campaign_name,
            k.utm_campaign,
            k.apply_url,
            k.details_url,
            k.papers_url,
            (c.utm_name || '_email'::text) || o_1.touch_no AS utm_base
           FROM attribution.outreach o_1
             JOIN attribution.contacts c USING (contact_id)
             JOIN attribution.schools s ON s.school_id = c.school_id
             JOIN attribution.campaigns k USING (campaign_id)
        )
 SELECT outreach_id,
-- ... truncated (79 lines total)
```

### attribution.v_report (view)
```sql
 WITH sent AS (
         SELECT c_1.school_id,
            o.contact_id,
            o.campaign_id,
            count(*) FILTER (WHERE o.status = 'SENT'::text) AS emails_sent,
            count(*) FILTER (WHERE o.reply_status = 'REPLIED'::text) AS replies
           FROM attribution.outreach o
             JOIN attribution.contacts c_1 USING (contact_id)
          WHERE NOT o.is_test
          GROUP BY c_1.school_id, o.contact_id, o.campaign_id
        ), apps AS (
         SELECT v_attributed_applications.school_ref AS school_id,
            v_attributed_applications.contact_ref AS contact_id,
            v_attributed_applications.campaign_ref AS campaign_id,
            count(*) AS applications,
            count(v_attributed_applications.payment_id) AS paid,
            sum(v_attributed_applications.net_minor) FILTER (WHERE v_attributed_applications.currency = 'USD'::text) / 100.0 AS revenue_usd,
            count(v_attributed_applications.payment_id) FILTER (WHERE v_attributed_applications.currency <> 'USD'::text) AS paid_other_currency
           FROM attribution.v_attributed_applications
          GROUP BY v_attributed_applications.school_ref, v_attributed_applications.contact_ref, v_attributed_applications.campaign_ref
        )
 SELECT COALESCE(sent.school_id, apps.school_id) AS school_id,
    s.name AS school,
    COALESCE(sent.contact_id, apps.contact_id) AS contact_id,
    c.full_name AS contact,
    c.role AS contact_role,
    COALESCE(sent.campaign_id, apps.campaign_id) AS campaign_id,
    k.name AS campaign,
    COALESCE(sent.emails_sent, 0::bigint) AS emails_sent,
    COALESCE(sent.replies, 0::bigint) AS replies,
    COALESCE(apps.applications, 0::bigint) AS applications,
    COALESCE(apps.paid, 0::bigint) AS paid,
    COALESCE(apps.revenue_usd, 0::numeric) AS revenue_usd,
    COALESCE(apps.paid_other_currency, 0::bigint) AS paid_other_currency,
    round(100.0 * COALESCE(apps.paid, 0::bigint)::numeric / NULLIF(apps.applications, 0)::numeric, 1) AS app_to_paid_pct
   FROM sent
     FULL JOIN apps ON apps.school_id = sent.school_id AND COALESCE(apps.contact_id, ''::text) = COALESCE(sent.contact_id, ''::text) AND apps.campaign_id = sent.campaign_id
     LEFT JOIN attribution.schools s ON s.school_id = COALESCE(sent.school_id, apps.school_id)
     LEFT JOIN attribution.contacts c ON c.contact_id = COALESCE(sent.contact_id, apps.contact_id)
     LEFT JOIN attribution.campaigns k ON k.campaign_id = COALESCE(sent.campaign_id, apps.campaign_id);
```

## Schema: core_ro

### core_ro.v_puller_applications (view)
```sql
 SELECT id,
    source_table,
    source_row_id,
    programme_code,
    cohort_raw,
    student_name,
    student_email,
    student_phone,
    student_grade,
    parent_name,
    parent_email,
    parent_phone,
    school_raw,
    school_code,
    city,
    state,
    country,
    applied_at,
    paid_at,
    payment_status_raw,
    provider_payment_id,
    payment_provider_hint,
    utm_source,
    utm_medium,
    utm_campaign,
    referral_source,
    person_id,
    needs_review,
    synced_at
   FROM v_puller_applications;
```

### core_ro.v_puller_bank_refunds (view)
```sql
 SELECT id,
    provider,
    programme_code,
    currency,
    amount_minor,
    refunded_on,
    payment_id,
    link_basis,
    needs_review,
    synced_at
   FROM v_puller_bank_refunds;
```

### core_ro.v_puller_payments (view)
```sql
 SELECT id,
    provider,
    provider_payment_id,
    amount_minor,
    currency,
    status,
    occurred_at,
    payer_email,
    payer_name,
    programme_code,
    utm_source,
    utm_medium,
    utm_campaign,
    method,
    fee_minor,
    refunded_minor,
    person_id,
    cohort_id,
    source_code,
    needs_review,
    description,
    external_ids,
    created_at,
    updated_at
   FROM v_puller_payments;
```

## Schema: leads

### leads.v_deal_stage_spans (view) options: security_invoker=true
SOP 7.2 — average days in each stage.
```sql
 SELECT d.id AS deal_id,
    d.stream,
    d.owner,
    (h.value ->> 'stage'::text)::integer AS stage,
    (h.value ->> 'at'::text)::timestamp with time zone AS entered_at,
    COALESCE(lead((h.value ->> 'at'::text)::timestamp with time zone) OVER (PARTITION BY d.id ORDER BY h.ordinality),
        CASE
            WHEN d.status = 'active'::text THEN now()
            ELSE NULL::timestamp with time zone
        END) AS left_at
   FROM leads.deals d
     CROSS JOIN LATERAL jsonb_array_elements(d.stage_history) WITH ORDINALITY h(value, ordinality);
```

## Schema: group

### group.institution_application_stats (view) options: security_invoker=on
```sql
 WITH first_meet AS (
         SELECT meetings.school_code,
            min(meetings.meeting_date) AS first_meeting_date
           FROM group.meetings
          WHERE meetings.school_code IS NOT NULL
          GROUP BY meetings.school_code
        ), per_program AS (
         SELECT a_1.school_code,
            COALESCE(NULLIF(btrim(a_1.cohort_raw), ''::text), a_1.programme_code) AS program_label,
            count(*)::integer AS apps,
            count(*) FILTER (WHERE a_1.paid_at IS NOT NULL)::integer AS paid
           FROM group.v_core_applications_unique a_1
          WHERE a_1.school_code IS NOT NULL
          GROUP BY a_1.school_code, (COALESCE(NULLIF(btrim(a_1.cohort_raw), ''::text), a_1.programme_code))
        )
 SELECT a.school_code,
    count(*)::integer AS apps_total,
    count(*) FILTER (WHERE a.paid_at IS NOT NULL)::integer AS paid_total,
    count(*) FILTER (WHERE fm.first_meeting_date IS NOT NULL AND a.applied_at::date < fm.first_meeting_date)::integer AS apps_before_first_meeting,
    count(*) FILTER (WHERE fm.first_meeting_date IS NOT NULL AND a.applied_at::date >= fm.first_meeting_date)::integer AS apps_after_first_meeting,
    ( SELECT jsonb_agg(jsonb_build_object('label', p.program_label, 'apps', p.apps, 'paid', p.paid) ORDER BY p.apps DESC) AS jsonb_agg
           FROM per_program p
          WHERE p.school_code = a.school_code) AS programs
   FROM group.v_core_applications_unique a
     LEFT JOIN first_meet fm ON fm.school_code = a.school_code
  WHERE a.school_code IS NOT NULL
  GROUP BY a.school_code;
```

### group.mv_meeting_outcomes (view) options: security_invoker=on
```sql
 SELECT meeting_id,
    meeting_date,
    meeting_status,
    lead_segment,
    full_name,
    school_name,
    school_code,
    email,
    host_name,
    assigned_to,
    lead_score,
    utm_source,
    apps_after,
    apps_before,
    paid_after,
    first_app_at,
    first_paid_at,
    outcome_state
   FROM group.v_meeting_outcomes_base o
  WHERE NOT group.is_internal_lead(email, school_name, full_name);
```

### group.mv_meeting_outcomes_all (MATERIALIZED VIEW)
```sql
 WITH cand AS (
         SELECT m_1.id AS mid,
            c.id AS aid,
            c.applied_at,
            c.paid_at
           FROM group.meetings m_1
             JOIN group.core_applications c ON lower(c.student_email) = lower(btrim(m_1.email))
          WHERE (COALESCE(m_1.lead_segment, 'unknown'::text) = ANY (ARRAY['student'::text, 'parent'::text, 'unknown'::text])) AND m_1.email IS NOT NULL AND m_1.meeting_date IS NOT NULL AND c.applied_at >= (m_1.meeting_date - 90)::timestamp with time zone AND c.applied_at < (m_1.meeting_date + 90)::timestamp with time zone
        UNION
         SELECT m_1.id,
            c.id,
            c.applied_at,
            c.paid_at
           FROM group.meetings m_1
             JOIN group.core_applications c ON c.school_code = m_1.school_code
          WHERE m_1.lead_segment = 'school'::text AND m_1.school_code IS NOT NULL AND m_1.meeting_date IS NOT NULL AND c.applied_at >= (m_1.meeting_date - 90)::timestamp with time zone AND c.applied_at < (m_1.meeting_date + 90)::timestamp with time zone
        )
 SELECT m.id AS meeting_id,
    m.meeting_date,
    m.meeting_status,
    m.lead_segment,
    m.full_name,
    m.school_name,
    m.school_code,
    m.email,
    m.host_name,
    m.assigned_to,
    m.lead_score,
    m.utm_source,
    COALESCE(count(g.aid) FILTER (WHERE g.applied_at >= m.meeting_date::timestamp with time zone), 0::bigint)::integer AS apps_after,
    COALESCE(count(g.aid) FILTER (WHERE g.applied_at < m.meeting_date::timestamp with time zone), 0::bigint)::integer AS apps_before,
    COALESCE(count(g.aid) FILTER (WHERE g.paid_at >= m.meeting_date::timestamp with time zone AND g.paid_at < (m.meeting_date + 90)::timestamp with time zone), 0::bigint)::integer AS paid_after,
    min(g.applied_at) FILTER (WHERE g.applied_at >= m.meeting_date::timestamp with time zone) AS first_app_at,
    min(g.paid_at) FILTER (WHERE g.paid_at >= m.meeting_date::timestamp with time zone) AS first_paid_at,
        CASE
            WHEN m.email IS NULL AND m.school_code IS NULL THEN 'no_link'::text
            WHEN count(g.aid) FILTER (WHERE g.applied_at >= m.meeting_date::timestamp with time zone) > 0 THEN 'converted'::text
            WHEN count(g.aid) FILTER (WHERE g.paid_at >= m.meeting_date::timestamp with time zone AND g.paid_at < (m.meeting_date + 90)::timestamp with time zone) > 0 THEN 'paid_existing'::text
            WHEN count(g.aid) > 0 THEN 'in_pipeline'::text
            ELSE 'no_outcome'::text
-- ... truncated (44 lines total)
```

### group.user_profiles_safe (view) options: security_invoker=on
```sql
 SELECT id,
    full_name,
    title,
    avatar_color,
    role,
    is_active,
    created_at
   FROM group.user_profiles;
```

### group.v_activity (view) options: security_invoker=on
```sql
 SELECT 'mtg:'::text || m.id AS activity_id,
    m.meeting_date::timestamp with time zone + COALESCE(m.scheduled_time, '00:00:00'::time without time zone)::interval AS occurred_at,
    'meeting'::text AS kind,
    'person'::text AS actor_type,
    COALESCE(m.host_name, m.assigned_to, 'Unassigned'::text) AS actor_name,
    'Meeting — '::text || COALESCE(m.meeting_status, 'no status'::text) AS title,
    NULLIF(btrim(COALESCE(m.program_pitched, ''::text)), ''::text) AS detail,
    m.id AS meeting_id,
    lower(btrim(m.email)) AS lead_email,
    m.school_code,
    'meetings'::text AS source_table,
    m.id AS source_id,
    jsonb_build_object('status', m.meeting_status, 'programme', m.program_pitched, 'duration_minutes', m.duration_minutes, 'lead_score', m.lead_score, 'next_step', m.next_step, 'summary', "left"(COALESCE(m.zoom_ai_summary, m.ai_notes, ''::text), 4000), 'objections', m.objections, 'commitments', m.commitments, 'key_quotes', m.key_quotes) AS payload
   FROM group.meetings m
UNION ALL
 SELECT 'call:'::text || c.id AS activity_id,
    COALESCE(c.started_at, c.created_at) AS occurred_at,
    'ai_call'::text AS kind,
    'ai'::text AS actor_type,
    'AI caller'::text AS actor_name,
    ('AI call — '::text || COALESCE(c.status, 'unknown'::text)) ||
        CASE
            WHEN c.duration_seconds IS NOT NULL THEN (' ('::text || c.duration_seconds) || 's)'::text
            ELSE ''::text
        END AS title,
    "left"(COALESCE(c.summary, ''::text), 300) AS detail,
    c.meeting_id,
    lower(btrim(m.email)) AS lead_email,
    m.school_code,
    'lead_ai_calls'::text AS source_table,
    c.id AS source_id,
    jsonb_build_object('summary', c.summary, 'sentiment', c.sentiment, 'questions_asked', c.questions_asked, 'answers', c.answers, 'duration_seconds', c.duration_seconds, 'status', c.status, 'recording_url', c.recording_url, 'transcript', c.transcript) AS payload
   FROM group.lead_ai_calls c
     LEFT JOIN group.meetings m ON m.id = c.meeting_id
UNION ALL
 SELECT 'dec:'::text || d.id AS activity_id,
    d.created_at AS occurred_at,
    'agent_decision'::text AS kind,
    'ai'::text AS actor_type,
    'Lead agent'::text AS actor_name,
-- ... truncated (133 lines total)
```

### group.v_activity_feed (view) options: security_invoker=on
```sql
 SELECT m.id AS meeting_id,
    'meeting:'::text || m.id::text AS ref_id,
    COALESCE(m.actual_start, m.meeting_date::timestamp with time zone) AS occurred_at,
    'meeting'::text AS kind,
    'person'::text AS actor_type,
    COALESCE(m.host_name, m.assigned_to) AS actor_name,
    COALESCE(m.meeting_status, 'Meeting'::text) || ' meeting'::text AS title,
    "left"(COALESCE(m.zoom_ai_summary, m.ai_notes, m.meeting_notes, ''::text), 400) AS detail
   FROM group.meetings m
UNION ALL
 SELECT e.meeting_id,
    'email:'::text || e.id::text AS ref_id,
    COALESCE(e.sent_at, e.received_at, e.created_at) AS occurred_at,
        CASE
            WHEN e.direction = 'inbound'::text THEN 'email_in'::text
            ELSE 'email_out'::text
        END AS kind,
        CASE
            WHEN e.direction = 'inbound'::text THEN 'lead'::text
            ELSE 'agent'::text
        END AS actor_type,
        CASE
            WHEN e.direction = 'inbound'::text THEN e.from_email
            ELSE 'AI agent'::text
        END AS actor_name,
    COALESCE(e.subject, '(no subject)'::text) AS title,
    "left"(COALESCE(e.body_text, e.intent_summary, ''::text), 400) AS detail
   FROM group.email_sends e
  WHERE e.meeting_id IS NOT NULL
UNION ALL
 SELECT t.meeting_id,
    'task:'::text || t.id::text AS ref_id,
    COALESCE(t.completed_at, t.created_at) AS occurred_at,
        CASE
            WHEN t.status = 'done'::group.task_status THEN 'task_done'::text
            ELSE 'task_open'::text
        END AS kind,
        CASE
            WHEN t.created_by_agent THEN 'agent'::text
            ELSE 'person'::text
-- ... truncated (70 lines total)
```

### group.v_claims_this_week (view) options: security_invoker=on
```sql
 SELECT a.user_id,
    COALESCE(NULLIF(p.full_name, ''::text), 'Unknown'::text) AS member_name,
    count(*)::integer AS claims
   FROM group.meeting_assignees a
     LEFT JOIN group.user_profiles p ON p.id = a.user_id
  WHERE a.assigned_at >= date_trunc('week'::text, now())
  GROUP BY a.user_id, p.full_name;
```

### group.v_conversion_evidence (view)
```sql
 WITH aliases AS (
         SELECT group.conversion_school_key(school_aliases.raw_text) AS k,
            min(school_aliases.school_code) AS sc
           FROM group.school_aliases
          WHERE school_aliases.school_code IS NOT NULL AND school_aliases.match_confidence = 1::double precision AND (school_aliases.match_source = ANY (ARRAY['seed'::text, 'bulk_import'::text, 'registry_exact_norm'::text, 'domain'::text, 'sm_20261003'::text, 'human'::text, 'founder_ai'::text]))
          GROUP BY (group.conversion_school_key(school_aliases.raw_text))
         HAVING count(DISTINCT school_aliases.school_code) = 1
        ), apps AS MATERIALIZED (
         SELECT c_1.id,
            c_1.source_table,
            c_1.source_row_id,
            c_1.programme_code,
            c_1.cohort_raw,
            c_1.student_name,
            c_1.student_email,
            c_1.student_phone,
            c_1.student_grade,
            c_1.parent_name,
            c_1.parent_email,
            c_1.parent_phone,
            c_1.school_raw,
            c_1.school_code,
            c_1.city,
            c_1.state,
            c_1.country,
            c_1.applied_at,
            c_1.paid_at,
            c_1.payment_status_raw,
            c_1.provider_payment_id,
            c_1.payment_provider_hint,
            c_1.utm_source,
            c_1.utm_medium,
            c_1.utm_campaign,
            c_1.referral_source,
            c_1.person_id,
            c_1.needs_review,
            c_1.synced_at,
            c_1.pulled_at,
            c_1.utm_member_id,
            COALESCE(c_1.school_code, a.sc) AS resolved_school_code,
-- ... truncated (182 lines total)
```

### group.v_core_applications_unique (view) options: security_invoker=true
```sql
 SELECT DISTINCT ON (c.source_row_id) c.id,
    c.source_table,
    c.source_row_id,
    c.programme_code,
    c.cohort_raw,
    c.student_email,
    c.applied_at,
    COALESCE(c.school_code, o.school_code) AS school_code,
    COALESCE(c.paid_at, o.paid_at) AS paid_at,
    c.student_name,
    c.student_phone,
    c.student_grade,
    c.parent_name,
    c.parent_email,
    c.parent_phone,
    c.school_raw,
    c.city,
    c.state,
    c.country,
    COALESCE(c.payment_status_raw, o.payment_status_raw) AS payment_status_raw,
        CASE
            WHEN c.provider_payment_id IS NULL THEN o.provider_payment_id
            ELSE c.provider_payment_id
        END AS provider_payment_id,
        CASE
            WHEN c.provider_payment_id IS NULL THEN o.payment_provider_hint
            ELSE c.payment_provider_hint
        END AS payment_provider_hint,
    c.utm_source,
    c.utm_medium,
    c.utm_campaign,
    COALESCE(c.referral_source, o.referral_source) AS referral_source,
    COALESCE(c.person_id, o.person_id) AS person_id,
    c.needs_review,
    c.synced_at,
    c.pulled_at,
    COALESCE(c.utm_member_id, o.utm_member_id) AS utm_member_id
   FROM group.core_applications c
     LEFT JOIN group.core_applications o ON o.source_row_id = c.source_row_id AND o.id <> c.id
  ORDER BY c.source_row_id, (c.source_table = 'applications'::text) DESC, c.synced_at DESC NULLS LAST, c.id, o.id;
```

### group.v_deal_stage_spans (view) options: security_invoker=on
```sql
 SELECT d.id AS deal_id,
    d.stream,
    d.owner,
    (h.value ->> 'stage'::text)::integer AS stage,
    (h.value ->> 'at'::text)::timestamp with time zone AS entered_at,
    COALESCE(lead((h.value ->> 'at'::text)::timestamp with time zone) OVER (PARTITION BY d.id ORDER BY h.ordinality),
        CASE
            WHEN d.status = 'active'::text THEN now()
            ELSE NULL::timestamp with time zone
        END) AS left_at
   FROM group.deals d
     CROSS JOIN LATERAL jsonb_array_elements(d.stage_history) WITH ORDINALITY h(value, ordinality);
```

### group.v_deal_work (view) options: security_invoker=true
```sql
 SELECT deal_id,
    school_code,
    school_name,
    contact_name,
    contact_email,
    designation,
    country,
    region,
    stream,
    programme,
    stage,
    status,
    stage_entered_at,
    owner,
    owner_name,
    sla_action AS next_action,
    sla_due AS next_action_due,
    sla_source,
    waiting_until,
    waiting_note,
    blocker,
    last_touch_at,
    touch_count,
    follow_up_booked_at,
    expected_students,
    expected_value_minor,
    weighted_value_minor,
    paid_amount_minor,
    paid_students,
    currency,
    proposal_sent_date,
    programme_start,
    registered,
    minimum_required,
    first_met,
    last_met,
    meetings_held,
    next_meeting,
    replies_waiting,
    oldest_reply_at,
-- ... truncated (57 lines total)
```

### group.v_deal_worklist (view) options: security_invoker=on
```sql
 WITH m AS (
         SELECT mt.deal_id,
            count(*) AS meetings_total,
            count(*) FILTER (WHERE group.sop_meeting_held(mt.meeting_status)) AS meetings_held,
            count(*) FILTER (WHERE lower(COALESCE(mt.meeting_status, ''::text)) = 'no show'::text) AS no_shows,
            max(mt.start_time_utc) AS last_meeting_at,
            min(mt.meeting_date) AS first_meeting_on,
            COALESCE(sum(COALESCE(array_length(mt.attributed_app_ids, 1), 0)), 0::bigint) AS applications,
            min(mt.meeting_date) FILTER (WHERE group.sop_meeting_held(mt.meeting_status)) AS first_met,
            max(mt.meeting_date) FILTER (WHERE group.sop_meeting_held(mt.meeting_status)) AS last_held_on,
            min(mt.meeting_date) FILTER (WHERE mt.meeting_date >= CURRENT_DATE AND (mt.meeting_status = ANY (ARRAY['Scheduled'::text, 'In Progress'::text, 'Follow-up'::text]))) AS next_meeting_on,
            bool_or(group.sop_meeting_held(mt.meeting_status) AND COALESCE(mt.meeting_notes, ''::text) = ''::text AND mt.meeting_date > (CURRENT_DATE - 14)) AS notes_missing
           FROM group.meetings mt
          WHERE mt.deal_id IS NOT NULL AND mt.merged_into IS NULL
          GROUP BY mt.deal_id
        ), t AS (
         SELECT tk.deal_id,
            count(*) FILTER (WHERE (tk.status = ANY (ARRAY['open'::group.task_status, 'pending'::group.task_status, 'snoozed'::group.task_status, 'needs_info'::group.task_status, 'revision_requested'::group.task_status, 'revising'::group.task_status])) AND NOT tk.archived) AS open_tasks,
            count(*) FILTER (WHERE (tk.status = ANY (ARRAY['open'::group.task_status, 'pending'::group.task_status, 'needs_info'::group.task_status])) AND NOT tk.archived AND tk.due_date < CURRENT_DATE) AS overdue_tasks
           FROM group.tasks tk
          WHERE tk.deal_id IS NOT NULL
          GROUP BY tk.deal_id
        ), r AS (
         SELECT mt.deal_id,
            count(*) AS replies_waiting,
            min(e.received_at) AS oldest_reply_at
           FROM group.email_sends e
             JOIN group.meetings mt ON mt.id = e.meeting_id
          WHERE e.direction = 'inbound'::text AND e.replied_at IS NULL AND mt.deal_id IS NOT NULL
          GROUP BY mt.deal_id
        ), sla0 AS (
         SELECT d_1.id AS deal_id,
            COALESCE(d_1.next_action,
                CASE
                    WHEN d_1.status = 'parked'::text THEN 'Reactivate: the school is back, re-open at Proposal sent'::text
                    WHEN d_1.stage = 0 THEN
                    CASE
                        WHEN mf.next_meeting_on IS NOT NULL THEN 'Reconfirm the meeting 24 hours before'::text
                        ELSE 'Booked but never held — rebook it or release the slot'::text
                    END
-- ... truncated (202 lines total)
```

### group.v_lead_journey (view) options: security_invoker=on
```sql
 WITH mt AS (
         SELECT lower(btrim(meetings.email)) AS em,
            max(meetings.full_name) AS nm,
            max(meetings.school_name) AS sch,
            max(meetings.school_code) AS code,
            max(meetings.country) AS ctry,
            string_agg(DISTINCT COALESCE(meetings.host_name, meetings.assigned_to), ', '::text) AS hosts,
            count(*)::integer AS total,
            count(*) FILTER (WHERE meetings.meeting_status = 'Completed'::text)::integer AS held,
            count(*) FILTER (WHERE meetings.meeting_status = 'No Show'::text)::integer AS ns,
            min(meetings.meeting_date) AS first_m,
            max(meetings.meeting_date) FILTER (WHERE meetings.meeting_date <= CURRENT_DATE) AS last_m,
            min(meetings.meeting_date) FILTER (WHERE meetings.meeting_date > CURRENT_DATE) AS next_m
           FROM group.meetings
          WHERE meetings.email IS NOT NULL AND btrim(meetings.email) <> ''::text
          GROUP BY (lower(btrim(meetings.email)))
        ), ap AS (
         SELECT lower(btrim(core_applications.student_email)) AS em,
            count(*)::integer AS apps,
            count(*) FILTER (WHERE core_applications.paid_at IS NOT NULL)::integer AS paid,
            min(core_applications.applied_at) AS f_app,
            min(core_applications.paid_at) AS f_paid
           FROM group.core_applications
          WHERE core_applications.student_email IS NOT NULL AND btrim(core_applications.student_email) <> ''::text
          GROUP BY (lower(btrim(core_applications.student_email)))
        ), ac AS (
         SELECT v_activity.lead_email AS em,
            count(*) FILTER (WHERE v_activity.kind = 'ai_call'::text)::integer AS calls,
            count(*) FILTER (WHERE v_activity.kind = 'agent_decision'::text)::integer AS decs,
            count(*) FILTER (WHERE v_activity.kind = 'email_out'::text)::integer AS mails,
            count(*) FILTER (WHERE v_activity.kind = 'task'::text)::integer AS acts,
            count(*) FILTER (WHERE v_activity.kind = 'task'::text AND ((v_activity.payload ->> 'status'::text) = ANY (ARRAY['open'::text, 'pending'::text])))::integer AS acts_open,
            count(*)::integer AS ev,
            max(v_activity.occurred_at) AS last_at
           FROM group.v_activity
          WHERE v_activity.lead_email IS NOT NULL
          GROUP BY v_activity.lead_email
        )
 SELECT mt.em AS lead_email,
    mt.nm AS lead_name,
-- ... truncated (87 lines total)
```

### group.v_lead_ledger (view) options: security_invoker=on
```sql
 WITH m AS (
         SELECT lower(btrim(meetings.email)) AS em,
            max(meetings.full_name) AS nm,
            max(meetings.school_name) AS sch,
            max(meetings.school_code) AS code,
            max(meetings.country) AS ctry,
            string_agg(DISTINCT meetings.host_name, ', '::text) AS hosts,
            count(*)::integer AS n,
            count(*) FILTER (WHERE meetings.meeting_status = 'Completed'::text)::integer AS held,
            count(*) FILTER (WHERE meetings.meeting_status = 'No Show'::text)::integer AS ns,
            max(meetings.meeting_date) FILTER (WHERE meetings.meeting_date <= CURRENT_DATE) AS last_m,
            min(meetings.meeting_date) FILTER (WHERE meetings.meeting_date > CURRENT_DATE) AS next_m
           FROM group.meetings
          WHERE meetings.email IS NOT NULL AND btrim(meetings.email) <> ''::text
          GROUP BY (lower(btrim(meetings.email)))
        ), l AS (
         SELECT lower(btrim(mm.email)) AS em,
            bool_or(lg.analysed) AS an,
            bool_or(lg.whatsapp_captured) AS wa,
            bool_or(lg.next_meeting_agreed) AS nm,
            sum(lg.open_tasks)::integer AS tasks
           FROM group.v_meeting_ledger lg
             JOIN group.meetings mm ON mm.id = lg.meeting_id
          WHERE mm.email IS NOT NULL
          GROUP BY (lower(btrim(mm.email)))
        ), a AS (
         SELECT lower(btrim(core_applications.student_email)) AS em,
            count(*)::integer AS n,
            count(*) FILTER (WHERE core_applications.paid_at IS NOT NULL)::integer AS paid,
            min(core_applications.applied_at) AS f,
            min(core_applications.paid_at) AS p
           FROM group.core_applications
          WHERE core_applications.student_email IS NOT NULL
          GROUP BY (lower(btrim(core_applications.student_email)))
        )
 SELECT m.em AS email,
    m.nm AS lead_name,
    m.sch AS school_name,
    m.code AS school_code,
    m.ctry AS country,
-- ... truncated (68 lines total)
```

### group.v_lead_source (view) options: security_invoker=on
```sql
 WITH ranked AS (
         SELECT lower(btrim(m.email)) AS lead_email,
            m.utm_source,
            m.utm_medium,
            m.utm_campaign,
            group.utm_key(m.utm_source) AS k,
            m.meeting_date,
            row_number() OVER (PARTITION BY (lower(btrim(m.email))) ORDER BY m.meeting_date) AS rn
           FROM group.meetings m
          WHERE m.email IS NOT NULL AND m.merged_into IS NULL
        )
 SELECT r.lead_email,
    r.utm_source,
    r.utm_medium,
    r.utm_campaign,
    r.k AS utm_key,
    COALESCE(cr.label, initcap(replace(COALESCE(r.k, 'unrecorded'::text), '-'::text, ' '::text))) AS campaign_label,
    COALESCE(cr.channel, 'unknown'::text) AS channel,
    r.meeting_date AS first_seen
   FROM ranked r
     LEFT JOIN group.campaign_registry cr ON cr.utm_key = r.k
  WHERE r.rn = 1;
```

### group.v_meeting_assignee (view) options: security_invoker=on
```sql
 SELECT m.id AS meeting_id,
    ma.user_id,
    COALESCE(up.full_name, NULLIF(btrim(m.assigned_to), ''::text), m.host_name) AS member_name,
    m.meeting_date,
    m.meeting_status
   FROM group.meetings m
     LEFT JOIN LATERAL ( SELECT a.user_id
           FROM group.meeting_assignees a
          WHERE a.meeting_id = m.id
          ORDER BY a.assigned_at
         LIMIT 1) ma ON true
     LEFT JOIN group.user_profiles up ON up.id = ma.user_id;
```

### group.v_meeting_attendance (view) options: security_invoker=on
```sql
 SELECT meeting_id,
    zoom_meeting_id,
    count(*) FILTER (WHERE is_staff) AS staff_sessions,
    count(*) FILTER (WHERE NOT is_staff) AS guest_sessions,
    count(*) FILTER (WHERE is_staff AND left_at IS NULL) AS staff_in_room,
    count(*) FILTER (WHERE NOT is_staff AND left_at IS NULL) AS guests_in_room,
    min(joined_at) FILTER (WHERE is_staff) AS first_staff_join,
    min(joined_at) FILTER (WHERE NOT is_staff) AS first_guest_join,
    max(COALESCE(left_at, now())) AS last_activity,
    round(EXTRACT(epoch FROM max(COALESCE(left_at, now())) - min(joined_at)) / 60::numeric)::integer AS room_minutes,
    string_agg(DISTINCT display_name, ', '::text) FILTER (WHERE is_staff) AS staff_names,
    string_agg(DISTINCT display_name, ', '::text) FILTER (WHERE NOT is_staff) AS guest_names
   FROM group.meeting_participants p
  GROUP BY meeting_id, zoom_meeting_id;
```

### group.v_meeting_clean (view) options: security_invoker=on
```sql
 SELECT id,
    meeting_date,
    scheduled_time,
    full_name,
    school_name,
    email,
    phone,
    city,
    state,
    country,
    designation,
    meeting_type,
    utm_source,
    query,
    meeting_link,
    zoom_meeting_id,
    recording_url,
    calendly_event_id,
    calendly_invitee_id,
    calendly_raw_payload,
    assigned_to,
    number_of_prev_applications,
    meeting_status,
    actual_start,
    actual_end,
    duration_minutes,
    ai_notes,
    meeting_notes,
    zoom_ai_summary,
    transcript_text,
    program_pitched,
    interest_level,
    lifecycle_stage,
    follow_up_date,
    followup_email_24h,
    followup_link,
    follow_up_meeting,
    followup1_status,
    followup1_content,
    followup1_response,
-- ... truncated (144 lines total)
```

### group.v_meeting_closeout (view) options: security_invoker=true
```sql
 SELECT m.id AS meeting_id,
    m.deal_id,
    m.meeting_date,
    m.scheduled_time,
    m.full_name,
    m.school_name,
    m.designation,
    m.meeting_status,
    m.host_user_id,
    m.program_pitched,
    COALESCE(m.end_time_utc, m.actual_end, (m.meeting_date + m.scheduled_time)::timestamp with time zone) AS ended_at,
    EXTRACT(epoch FROM now() - COALESCE(m.end_time_utc, m.actual_end, (m.meeting_date + m.scheduled_time)::timestamp with time zone)) / 3600::numeric AS hours_since,
    ( SELECT array_agg(ma.user_id) AS array_agg
           FROM group.meeting_assignees ma
          WHERE ma.meeting_id = m.id) AS assignee_ids,
    vm.user_id AS member_user_id,
    vm.member_name
   FROM group.meetings m
     LEFT JOIN group.v_meeting_member vm ON vm.meeting_id = m.id AND vm.is_primary
  WHERE m.merged_into IS NULL AND group.sop_meeting_held(m.meeting_status) AND NULLIF(btrim(COALESCE(m.meeting_notes, ''::text)), ''::text) IS NULL AND m.meeting_date >= (CURRENT_DATE - 60);
```

### group.v_meeting_lead_type (view) options: security_invoker=on
```sql
 SELECT id AS meeting_id,
    meeting_date,
    lead_group,
    lead_type,
    meeting_status,
    meeting_status = 'Completed'::text AND NOT group.is_internal_lead(email, school_name, full_name) AND NOT lower(COALESCE(btrim(full_name), ''::text)) ~ '^(test|testing|demo|dummy)$'::text AND NOT lower(COALESCE(btrim(school_name), ''::text)) ~ '^(test|testing|demo|dummy)'::text AND NOT (COALESCE(btrim(email), ''::text) = ''::text AND COALESCE(btrim(school_name), ''::text) = ''::text) AS counts_held
   FROM group.meetings
  WHERE merged_into IS NULL;
```

### group.v_meeting_ledger (view) options: security_invoker=on
```sql
 SELECT o.meeting_id,
    o.meeting_date,
    o.full_name AS lead_name,
    o.school_name,
    o.school_code,
    m.country,
    m.state,
    o.host_name,
    m.program_pitched AS programme,
    o.lead_segment,
    o.lead_score,
    o.utm_source,
    o.outcome_state,
    o.apps_after,
    o.paid_after,
    o.first_app_at,
    o.first_paid_at,
        CASE
            WHEN o.first_paid_at IS NOT NULL THEN EXTRACT(day FROM o.first_paid_at - o.meeting_date::timestamp with time zone)::integer
            ELSE NULL::integer
        END AS days_to_pay,
    m.commitments_extracted_at IS NOT NULL AS analysed,
    (m.promised_items ->> 'whatsapp_number'::text) IS NOT NULL AS whatsapp_captured,
    COALESCE((m.promised_items ->> 'whatsapp_group_promised'::text)::boolean, false) AS whatsapp_group_promised,
    COALESCE((m.promised_items ->> 'next_meeting_agreed'::text)::boolean, false) AS next_meeting_agreed,
    NULLIF(m.promised_items ->> 'decision_maker'::text, ''::text) AS decision_maker,
    jsonb_array_length(COALESCE(m.promised_items -> 'documents_promised'::text, '[]'::jsonb)) AS docs_promised,
    jsonb_array_length(COALESCE(m.promised_items -> 'open_questions'::text, '[]'::jsonb)) AS open_questions,
    jsonb_array_length(COALESCE(m.promised_items -> 'promises'::text, '[]'::jsonb)) AS promises_made,
    COALESCE(t.n, 0) AS open_tasks,
    m.assigned_to
   FROM group.mv_meeting_outcomes o
     JOIN group.meetings m ON m.id = o.meeting_id
     LEFT JOIN ( SELECT tasks.meeting_id,
            count(*)::integer AS n
           FROM group.tasks
          WHERE (tasks.status = ANY (ARRAY['open'::group.task_status, 'pending'::group.task_status])) AND tasks.meeting_id IS NOT NULL
          GROUP BY tasks.meeting_id) t ON t.meeting_id = o.meeting_id
  WHERE o.meeting_status = 'Completed'::text;
```

### group.v_meeting_member (view) options: security_invoker=true
```sql
 WITH prim AS (
         SELECT m.id AS meeting_id,
            COALESCE(ma.uid, m.host_user_id, hp.id) AS user_id
           FROM group.meetings m
             LEFT JOIN LATERAL ( SELECT a.user_id AS uid
                   FROM group.meeting_assignees a
                  WHERE a.meeting_id = m.id
                  ORDER BY a.assigned_at, a.user_id
                 LIMIT 1) ma ON true
             LEFT JOIN LATERAL ( SELECT p.id
                   FROM group.user_profiles p
                  WHERE p.full_name = ANY (ARRAY[NULLIF(btrim(split_part(m.assigned_to, ','::text, 1)), ''::text), m.host_name])
                  ORDER BY (p.full_name = NULLIF(btrim(split_part(m.assigned_to, ','::text, 1)), ''::text)) DESC, p.id
                 LIMIT 1) hp ON true
        )
 SELECT a.meeting_id,
    a.user_id,
    p.full_name AS member_name,
    NOT a.user_id IS DISTINCT FROM pr.user_id AS is_primary
   FROM group.meeting_assignees a
     JOIN group.user_profiles p ON p.id = a.user_id
     JOIN prim pr ON pr.meeting_id = a.meeting_id
UNION
 SELECT pr.meeting_id,
    pr.user_id,
    COALESCE(p.full_name, 'Unassigned'::text) AS member_name,
    true AS is_primary
   FROM prim pr
     LEFT JOIN group.user_profiles p ON p.id = pr.user_id
  WHERE NOT (EXISTS ( SELECT 1
           FROM group.meeting_assignees a
          WHERE a.meeting_id = pr.meeting_id));
```

### group.v_meeting_outcomes (view) options: security_invoker=on
```sql
 SELECT meeting_id,
    meeting_date,
    meeting_status,
    lead_segment,
    full_name,
    school_name,
    school_code,
    email,
    host_name,
    assigned_to,
    lead_score,
    utm_source,
    apps_after,
    apps_before,
    paid_after,
    first_app_at,
    first_paid_at,
    outcome_state
   FROM group.v_meeting_outcomes_base;
```

### group.v_meeting_outcomes_base (view) options: security_invoker=true
```sql
 SELECT m.id AS meeting_id,
    m.meeting_date,
    m.meeting_status,
    m.lead_segment,
    m.full_name,
    m.school_name,
    m.school_code,
    m.email,
    m.host_name,
    m.assigned_to,
    m.lead_score,
    m.utm_source,
    COALESCE(e.apps_after, 0::bigint)::integer AS apps_after,
    COALESCE(e.apps_before, 0::bigint)::integer AS apps_before,
    COALESCE(e.paid_after, 0::bigint)::integer AS paid_after,
    e.first_app_at,
    e.first_paid_at,
        CASE
            WHEN COALESCE(e.apps_after, 0::bigint) > 0 THEN 'converted'::text
            WHEN COALESCE(e.paid_after, 0::bigint) > 0 THEN 'paid_existing'::text
            WHEN m.email IS NULL AND m.school_code IS NULL THEN 'no_link'::text
            ELSE 'no_outcome'::text
        END AS outcome_state
   FROM group.meetings m
     LEFT JOIN ( SELECT v_conversion_evidence.meeting_id,
            count(*) FILTER (WHERE v_conversion_evidence.application_after_meeting) AS apps_after,
            count(*) FILTER (WHERE NOT v_conversion_evidence.application_after_meeting) AS apps_before,
            count(*) FILTER (WHERE v_conversion_evidence.paid_after_meeting) AS paid_after,
            min(v_conversion_evidence.applied_at) FILTER (WHERE v_conversion_evidence.application_after_meeting) AS first_app_at,
            min(v_conversion_evidence.paid_at) FILTER (WHERE v_conversion_evidence.paid_after_meeting) AS first_paid_at
           FROM group.v_conversion_evidence
          WHERE v_conversion_evidence.credit_reason <> 'utm_member'::text
          GROUP BY v_conversion_evidence.meeting_id) e ON e.meeting_id = m.id;
```

### group.v_meeting_revenue (view) options: security_invoker=true
```sql
 SELECT b.meeting_id,
    p.currency,
    sum(GREATEST(p.amount_minor - COALESCE(p.refunded_minor, 0::bigint), 0::bigint))::bigint AS revenue_minor
   FROM group.v_payment_meeting b
     JOIN group.core_payments p ON p.id = b.payment_id
  GROUP BY b.meeting_id, p.currency;
```

### group.v_meeting_roi (view) options: security_invoker=on
```sql
 SELECT date_trunc('month'::text, paid_at)::date AS month,
    count(*)::integer AS paid_students,
    count(*) FILTER (WHERE (EXISTS ( SELECT 1
           FROM group.meetings m
          WHERE m.meeting_status = 'Completed'::text AND m.meeting_date IS NOT NULL AND m.meeting_date <= c.paid_at::date AND lower(btrim(m.email)) = lower(c.student_email))))::integer AS met_directly,
    count(*) FILTER (WHERE school_code IS NOT NULL AND (EXISTS ( SELECT 1
           FROM group.meetings m
          WHERE m.meeting_status = 'Completed'::text AND m.meeting_date IS NOT NULL AND m.meeting_date <= c.paid_at::date AND m.school_code = c.school_code AND lower(btrim(COALESCE(m.email, ''::text))) <> lower(c.student_email))))::integer AS met_via_school
   FROM group.v_core_applications_unique c
  WHERE paid_at IS NOT NULL
  GROUP BY (date_trunc('month'::text, paid_at)::date)
  ORDER BY (date_trunc('month'::text, paid_at)::date) DESC;
```

### group.v_member_meeting (view) options: security_invoker=on
```sql
 SELECT a.user_id,
    p.full_name AS member_name,
    a.meeting_id
   FROM group.meeting_assignees a
     JOIN group.user_profiles p ON p.id = a.user_id
UNION
 SELECT p.id AS user_id,
    p.full_name AS member_name,
    m.id AS meeting_id
   FROM group.meetings m
     JOIN group.user_profiles p ON p.full_name = m.host_name
  WHERE NOT (EXISTS ( SELECT 1
           FROM group.meeting_assignees a2
          WHERE a2.meeting_id = m.id));
```

### group.v_member_performance (view) options: security_invoker=on
```sql
 WITH base AS (
         SELECT vm.user_id,
            vm.member_name,
            m.id AS mid,
            m.meeting_date,
            m.meeting_status,
            lower(btrim(m.email)) AS em
           FROM group.v_meeting_member vm
             JOIN group.meetings m ON m.id = vm.meeting_id
          WHERE vm.is_primary AND NOT group.is_internal_lead(m.email, m.school_name, m.full_name)
        ), led AS (
         SELECT b.user_id,
            b.member_name,
            b.mid,
            b.meeting_date,
            b.meeting_status,
            b.em,
            l_1.outcome_state,
            l_1.apps_after,
            l_1.paid_after,
            l_1.days_to_pay,
            l_1.analysed,
            l_1.whatsapp_captured,
            l_1.next_meeting_agreed,
            l_1.docs_promised,
            l_1.open_questions
           FROM base b
             LEFT JOIN group.v_meeting_ledger l_1 ON l_1.meeting_id = b.mid
        ), act AS (
         SELECT b.user_id,
            COALESCE(sum(x.calls), 0::bigint::numeric)::integer AS calls,
            COALESCE(sum(x.mails), 0::bigint::numeric)::integer AS mails,
            COALESCE(sum(x.acts), 0::bigint::numeric)::integer AS acts,
            COALESCE(sum(x.acts_open), 0::bigint::numeric)::integer AS acts_open
           FROM base b
             JOIN ( SELECT c.meeting_id AS mid,
                    count(*) AS calls,
                    0::bigint AS mails,
                    0::bigint AS acts,
                    0::bigint AS acts_open
-- ... truncated (97 lines total)
```

### group.v_payment_attribution (view) options: security_invoker=true
```sql
 SELECT p.id AS payment_id,
    p.occurred_at,
    p.currency,
    p.amount_minor,
    COALESCE(p.refunded_minor, 0::bigint) AS refunded_minor,
    GREATEST(p.amount_minor - COALESCE(p.refunded_minor, 0::bigint), 0::bigint) AS net_minor,
    p.status,
    p.payer_name,
    p.payer_email,
    p.programme_code,
    p.method,
    p.provider,
    b.meeting_id,
    b.meeting_date,
    m.full_name AS lead_name,
    m.school_name,
    COALESCE(NULLIF(btrim(m.assigned_to), ''::text), m.host_name) AS host_name,
        CASE
            WHEN b.meeting_date IS NOT NULL THEN p.occurred_at::date - b.meeting_date
            ELSE NULL::integer
        END AS days_to_pay,
    b.meeting_id IS NOT NULL AS from_meeting,
    b.match_route
   FROM group.core_payments p
     LEFT JOIN group.v_payment_meeting b ON b.payment_id = p.id
     LEFT JOIN group.meetings m ON m.id = b.meeting_id
  WHERE p.status = ANY (ARRAY['paid'::text, 'partially_refunded'::text]);
```

### group.v_payment_meeting (view) options: security_invoker=true
```sql
 WITH cand_all AS (
         SELECT m.id AS mid,
            m.meeting_date AS mdate,
            c.id AS aid,
            c.applied_at,
            c.paid_at,
            c.person_id,
            c.provider_payment_id
           FROM group.meetings m
             JOIN group.core_applications c ON lower(c.student_email) = lower(btrim(m.email))
          WHERE (COALESCE(m.lead_segment, 'unknown'::text) = ANY (ARRAY['student'::text, 'parent'::text, 'unknown'::text])) AND m.email IS NOT NULL AND m.meeting_date IS NOT NULL AND c.applied_at >= (m.meeting_date - 90)::timestamp with time zone AND c.applied_at < (m.meeting_date + 90)::timestamp with time zone
        UNION
         SELECT m.id,
            m.meeting_date,
            c.id,
            c.applied_at,
            c.paid_at,
            c.person_id,
            c.provider_payment_id
           FROM group.meetings m
             JOIN group.core_applications c ON c.school_code = m.school_code
          WHERE m.lead_segment = 'school'::text AND m.school_code IS NOT NULL AND m.meeting_date IS NOT NULL AND c.applied_at >= (m.meeting_date - 90)::timestamp with time zone AND c.applied_at < (m.meeting_date + 90)::timestamp with time zone
        ), cand AS (
         SELECT DISTINCT ON (cand_all.aid) cand_all.mid,
            cand_all.mdate,
            cand_all.aid,
            cand_all.paid_at,
            cand_all.person_id,
            cand_all.provider_payment_id
           FROM cand_all
          ORDER BY cand_all.aid, (cand_all.applied_at >= cand_all.mdate::timestamp with time zone) DESC, (
                CASE
                    WHEN cand_all.applied_at >= cand_all.mdate::timestamp with time zone THEN cand_all.mdate
                    ELSE NULL::date
                END) DESC NULLS LAST, cand_all.mdate
        ), app_linked AS (
         SELECT DISTINCT c.mid,
            c.mdate,
            p.id AS payment_id
           FROM cand c
-- ... truncated (75 lines total)
```

### group.v_school_ledger (view) options: security_invoker=on
```sql
 WITH mtg AS (
         SELECT v_meeting_ledger.school_code,
            count(*)::integer AS n,
            max(v_meeting_ledger.meeting_date) AS last_m,
            min(v_meeting_ledger.meeting_date) AS first_m,
            string_agg(DISTINCT v_meeting_ledger.host_name, ', '::text) AS hosts,
            count(DISTINCT v_meeting_ledger.host_name)::integer AS hc
           FROM group.v_meeting_ledger
          WHERE v_meeting_ledger.school_code IS NOT NULL
          GROUP BY v_meeting_ledger.school_code
        ), app AS (
         SELECT core_applications.school_code,
            count(*)::integer AS n,
            count(*) FILTER (WHERE core_applications.paid_at IS NOT NULL)::integer AS paid,
            count(DISTINCT core_applications.programme_code)::integer AS progs,
            min(core_applications.applied_at)::date AS f,
            max(core_applications.applied_at)::date AS l
           FROM group.core_applications
          WHERE core_applications.school_code IS NOT NULL
          GROUP BY core_applications.school_code
        )
 SELECT r.code AS school_code,
    r.canonical_name AS school_name,
    r.country,
    r.city,
    COALESCE(m.n, 0) AS meetings,
    m.last_m AS last_meeting,
    m.first_m AS first_meeting,
    m.hosts,
    COALESCE(m.hc, 0) AS host_count,
    COALESCE(a.n, 0) AS applications,
    COALESCE(a.paid, 0) AS paid_students,
    COALESCE(a.progs, 0) AS programmes,
    a.f AS first_app,
    a.l AS last_app,
        CASE
            WHEN COALESCE(a.n, 0) = 0 THEN 'no applications'::text
            WHEN COALESCE(a.progs, 0) >= 2 THEN 'returning'::text
            ELSE 'first programme'::text
        END AS status
-- ... truncated (43 lines total)
```

### group.v_school_owner (view) options: security_invoker=true
```sql
 SELECT DISTINCT ON (d.school_code) d.school_code,
    d.owner,
    p.full_name AS owner_name
   FROM group.deals d
     JOIN group.user_profiles p ON p.id = d.owner
  WHERE d.school_code IS NOT NULL AND d.owner IS NOT NULL
  ORDER BY d.school_code, d.updated_at DESC NULLS LAST, d.created_at DESC NULLS LAST;
```

### group.v_school_source (view) options: security_invoker=on
```sql
 SELECT m.school_code,
    COALESCE(cr.label, initcap(replace(COALESCE(group.utm_key(m.utm_source), 'unrecorded'::text), '-'::text, ' '::text))) AS campaign_label,
    count(*)::integer AS meetings,
    count(DISTINCT lower(btrim(m.email)))::integer AS leads,
    min(m.meeting_date) AS first_seen,
    max(m.meeting_date) AS last_seen
   FROM group.meetings m
     LEFT JOIN group.campaign_registry cr ON cr.utm_key = group.utm_key(m.utm_source)
  WHERE m.school_code IS NOT NULL AND m.merged_into IS NULL
  GROUP BY m.school_code, (COALESCE(cr.label, initcap(replace(COALESCE(group.utm_key(m.utm_source), 'unrecorded'::text), '-'::text, ' '::text))));
```

## Schema: migration

### migration.v_app_doctors (view)
```sql
 SELECT 'doctors_applications'::text AS source_table,
    id AS source_row_id,
    'FDF'::text AS programme_code,
    cohort AS cohort_raw,
    student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    student_phone,
    grade AS student_grade,
    parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    parent_phone,
    school AS school_raw,
    city,
    NULL::text AS state,
    country,
    COALESCE(applied_date::timestamp with time zone, created_at) AS applied_at,
    payment_date AS paid_at,
    payment_status AS payment_status_raw,
    payment_id AS provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    hear_about AS referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.doctors_applications t;
```

### migration.v_app_edge (view)
Empty since 2026-11-16: source edge_applications table no longer exists. Previously read legacy_ro.edge_applications as programme FEDG.
```sql
 SELECT source_table,
    source_row_id,
    programme_code,
    cohort_raw,
    student_name,
    student_email,
    student_phone,
    student_grade,
    parent_name,
    parent_email,
    parent_phone,
    school_raw,
    city,
    state,
    country,
    applied_at,
    paid_at,
    payment_status_raw,
    provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    referral_source,
    raw
   FROM migration.v_app_doctors
  WHERE false;
```

### migration.v_app_finlit (view)
```sql
 SELECT 'finlit_applications'::text AS source_table,
    id AS source_row_id,
    'CMFL'::text AS programme_code,
    cohort AS cohort_raw,
    NULLIF(btrim(concat_ws(' '::text, student_first_name, student_last_name)), ''::text) AS student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    student_phone,
    grade_level AS student_grade,
    parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    parent_phone,
    school AS school_raw,
    city,
    state,
    country,
    created_at AS applied_at,
    payment_date AS paid_at,
    payment_status AS payment_status_raw,
    COALESCE(payment_intent_id, payment_id) AS provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.finlit_applications t;
```

### migration.v_app_huso_india (view)
Legacy HUSO India source is absent upstream as of 2026-10-26. Empty compatibility view prevents the whole application sync from failing while preserving historical public.applications rows.
```sql
 SELECT NULL::text AS source_table,
    NULL::uuid AS source_row_id,
    NULL::text AS programme_code,
    NULL::text AS cohort_raw,
    NULL::text AS student_name,
    NULL::text AS student_email,
    NULL::text AS student_phone,
    NULL::text AS student_grade,
    NULL::text AS parent_name,
    NULL::text AS parent_email,
    NULL::text AS parent_phone,
    NULL::text AS school_raw,
    NULL::text AS city,
    NULL::text AS state,
    NULL::text AS country,
    NULL::timestamp with time zone AS applied_at,
    NULL::timestamp with time zone AS paid_at,
    NULL::text AS payment_status_raw,
    NULL::text AS provider_payment_id,
    NULL::text AS utm_source,
    NULL::text AS utm_medium,
    NULL::text AS utm_campaign,
    NULL::text AS referral_source,
    NULL::jsonb AS raw
  WHERE false;
```

### migration.v_app_huso_me (view)
Legacy HUSO ME source is absent upstream as of 2026-10-26. Empty compatibility view prevents the whole application sync from failing while preserving historical public.applications rows.
```sql
 SELECT NULL::text AS source_table,
    NULL::uuid AS source_row_id,
    NULL::text AS programme_code,
    NULL::text AS cohort_raw,
    NULL::text AS student_name,
    NULL::text AS student_email,
    NULL::text AS student_phone,
    NULL::text AS student_grade,
    NULL::text AS parent_name,
    NULL::text AS parent_email,
    NULL::text AS parent_phone,
    NULL::text AS school_raw,
    NULL::text AS city,
    NULL::text AS state,
    NULL::text AS country,
    NULL::timestamp with time zone AS applied_at,
    NULL::timestamp with time zone AS paid_at,
    NULL::text AS payment_status_raw,
    NULL::text AS provider_payment_id,
    NULL::text AS utm_source,
    NULL::text AS utm_medium,
    NULL::text AS utm_campaign,
    NULL::text AS referral_source,
    NULL::jsonb AS raw
  WHERE false;
```

### migration.v_app_lawyers (view)
```sql
 SELECT 'lawyers_applications'::text AS source_table,
    id AS source_row_id,
    COALESCE(program_code, 'LAWY'::text) AS programme_code,
    cohort AS cohort_raw,
    student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    student_phone,
    grade AS student_grade,
    parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    parent_phone,
    school AS school_raw,
    city,
    state,
    country,
    created_at AS applied_at,
    payment_date AS paid_at,
    payment_status AS payment_status_raw,
    COALESCE(payment_intent_id, payment_id) AS provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    hear_about AS referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.lawyers_applications t;
```

### migration.v_app_lwl (view)
```sql
 SELECT 'applications'::text AS source_table,
    id AS source_row_id,
    NULLIF(btrim(program_code), ''::text) AS programme_code,
    program_name AS cohort_raw,
    NULLIF(btrim(student_name), ''::text) AS student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    NULLIF(btrim(student_mobile), ''::text) AS student_phone,
    grade AS student_grade,
    NULLIF(btrim(parent_name), ''::text) AS parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    NULLIF(btrim(parent_phone), ''::text) AS parent_phone,
    school_name AS school_raw,
    city,
    state,
    country,
    created_at AS applied_at,
    payment_date AS paid_at,
    payment_status AS payment_status_raw,
    NULLIF(btrim(payment_intent_id), ''::text) AS provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    NULL::text AS referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.applications t
  WHERE NOT (EXISTS ( SELECT 1
           FROM applications a
          WHERE (a.source_table = ANY (ARRAY['akbar_offline_sheet'::text, 'prelaw_applications'::text, 'premed_applications'::text])) AND a.programme_code = NULLIF(btrim(t.program_code), ''::text) AND lower(btrim(a.student_email)) = lower(NULLIF(btrim(t.student_email), ''::text))));
```

### migration.v_app_onward (view)
```sql
 SELECT 'onward_applications'::text AS source_table,
    id AS source_row_id,
    COALESCE(program_code, 'ONWD'::text) AS programme_code,
    cohort AS cohort_raw,
    student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    student_phone,
    grade_level AS student_grade,
    parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    parent_phone,
    school_name AS school_raw,
    city,
    state,
    country,
    COALESCE(form_submitted_at, created_at) AS applied_at,
    payment_date AS paid_at,
    payment_status AS payment_status_raw,
    payment_id AS provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.onward_applications t;
```

### migration.v_app_prelaw (view)
```sql
 SELECT 'prelaw_applications'::text AS source_table,
    id AS source_row_id,
    COALESCE(program_code, 'PLAW'::text) AS programme_code,
    NULL::text AS cohort_raw,
    NULLIF(btrim(concat_ws(' '::text, student_first_name, student_middle_name, student_last_name)), ''::text) AS student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    student_mobile_number AS student_phone,
    current_class AS student_grade,
    parent_guardian_name AS parent_name,
    lower(NULLIF(btrim(parent_guardian_email), ''::text)) AS parent_email,
    parent_guardian_mobile_number AS parent_phone,
    school_name AS school_raw,
    state_province_region_city AS city,
    NULL::text AS state,
    country,
    COALESCE(submitted_at, created_at) AS applied_at,
    payment_date AS paid_at,
    NULL::text AS payment_status_raw,
    payment_id AS provider_payment_id,
    NULL::text AS utm_source,
    NULL::text AS utm_medium,
    NULL::text AS utm_campaign,
    NULL::text AS referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.prelaw_applications t;
```

### migration.v_app_premed (view)
```sql
 SELECT 'premed_applications'::text AS source_table,
    id AS source_row_id,
    COALESCE(program_code, 'PMED'::text) AS programme_code,
    NULL::text AS cohort_raw,
    NULLIF(btrim(concat_ws(' '::text, student_first_name, student_middle_name, student_last_name)), ''::text) AS student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    student_mobile_number AS student_phone,
    current_class AS student_grade,
    parent_guardian_name AS parent_name,
    lower(NULLIF(btrim(parent_guardian_email), ''::text)) AS parent_email,
    parent_guardian_mobile_number AS parent_phone,
    school_name AS school_raw,
    state_province_region_city AS city,
    NULL::text AS state,
    country,
    COALESCE(submitted_at, created_at) AS applied_at,
    payment_date AS paid_at,
    NULL::text AS payment_status_raw,
    payment_id AS provider_payment_id,
    NULL::text AS utm_source,
    NULL::text AS utm_medium,
    NULL::text AS utm_campaign,
    NULL::text AS referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.premed_applications t;
```

### migration.v_app_purpose (view)
```sql
 SELECT 'purpose_applications'::text AS source_table,
    id AS source_row_id,
    COALESCE(program_code, 'PURP'::text) AS programme_code,
    cohort_key AS cohort_raw,
    student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    student_phone,
    grade_level AS student_grade,
    parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    parent_phone,
    school_name AS school_raw,
    city,
    NULL::text AS state,
    country,
    COALESCE(application_submitted_at, created_at) AS applied_at,
    payment_date AS paid_at,
    payment_status AS payment_status_raw,
    COALESCE(payment_intent_id, payment_id) AS provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.purpose_applications t;
```

### migration.v_app_sat (view)
```sql
 SELECT 'sat_applications'::text AS source_table,
    id AS source_row_id,
    COALESCE(NULLIF(btrim(program_code), ''::text), 'SATI'::text) AS programme_code,
    cohort AS cohort_raw,
    student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    student_phone,
    grade AS student_grade,
    parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    parent_phone,
    school_name AS school_raw,
    city,
    NULL::text AS state,
    country,
    created_at AS applied_at,
    payment_date AS paid_at,
    payment_status AS payment_status_raw,
    payment_id AS provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_sat.sat_applications t;
```

### migration.v_app_sirluther (view)
```sql
 SELECT 'sirluther_applications'::text AS source_table,
    id AS source_row_id,
    'SLC'::text AS programme_code,
    cohort AS cohort_raw,
    student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    COALESCE(NULLIF(btrim(student_phone), ''::text), NULLIF(btrim(phone), ''::text)) AS student_phone,
    grade_level AS student_grade,
    parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    NULL::text AS parent_phone,
    school_name AS school_raw,
    city,
    state,
    country,
    COALESCE(application_submitted_at, created_at) AS applied_at,
    payment_date AS paid_at,
    payment_status AS payment_status_raw,
    payment_id AS provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    NULL::text AS referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.sirluther_applications t;
```

### migration.v_app_stud (view)
```sql
 SELECT 'leaders_applications'::text AS source_table,
    id AS source_row_id,
    'STLD'::text AS programme_code,
    path AS cohort_raw,
    NULLIF(btrim(full_name), ''::text) AS student_name,
    lower(NULLIF(btrim(email), ''::text)) AS student_email,
    NULLIF(btrim(phone), ''::text) AS student_phone,
    grade AS student_grade,
    NULLIF(btrim(parent_name), ''::text) AS parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    NULLIF(btrim(parent_phone), ''::text) AS parent_phone,
    school AS school_raw,
    city,
    state,
    country,
    created_at AS applied_at,
    NULL::timestamp with time zone AS paid_at,
    NULL::text AS payment_status_raw,
    NULL::text AS provider_payment_id,
    NULL::text AS utm_source,
    NULL::text AS utm_medium,
    NULL::text AS utm_campaign,
    NULL::text AS referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.leaders_applications t;
```

### migration.v_app_teo (view)
```sql
 SELECT 'teo_applications'::text AS source_table,
    id AS source_row_id,
    'FOUNDERS'::text AS programme_code,
    cohort AS cohort_raw,
    student_name,
    lower(NULLIF(btrim(student_email), ''::text)) AS student_email,
    student_phone,
    grade_level AS student_grade,
    parent_name,
    lower(NULLIF(btrim(parent_email), ''::text)) AS parent_email,
    parent_phone,
    school_name AS school_raw,
    city,
    state,
    country,
    COALESCE(submitted_at, created_at) AS applied_at,
    payment_date AS paid_at,
    payment_status AS payment_status_raw,
    COALESCE(payment_intent_id, payment_id) AS provider_payment_id,
    utm_source,
    utm_medium,
    utm_campaign,
    referral_source,
    to_jsonb(t.*) AS raw
   FROM legacy_ro.teo_applications t;
```

### migration.v_app_yale (view)
Legacy Yale candidate source is absent upstream as of 2026-10-26. Empty compatibility view prevents the whole application sync from failing while preserving historical public.applications rows.
```sql
 SELECT NULL::text AS source_table,
    NULL::uuid AS source_row_id,
    NULL::text AS programme_code,
    NULL::text AS cohort_raw,
    NULL::text AS student_name,
    NULL::text AS student_email,
    NULL::text AS student_phone,
    NULL::text AS student_grade,
    NULL::text AS parent_name,
    NULL::text AS parent_email,
    NULL::text AS parent_phone,
    NULL::text AS school_raw,
    NULL::text AS city,
    NULL::text AS state,
    NULL::text AS country,
    NULL::timestamp with time zone AS applied_at,
    NULL::timestamp with time zone AS paid_at,
    NULL::text AS payment_status_raw,
    NULL::text AS provider_payment_id,
    NULL::text AS utm_source,
    NULL::text AS utm_medium,
    NULL::text AS utm_campaign,
    NULL::text AS referral_source,
    NULL::jsonb AS raw
  WHERE false;
```

### migration.v_applications_all (view)
```sql
 SELECT v_app_doctors.source_table,
    v_app_doctors.source_row_id,
    v_app_doctors.programme_code,
    v_app_doctors.cohort_raw,
    v_app_doctors.student_name,
    v_app_doctors.student_email,
    v_app_doctors.student_phone,
    v_app_doctors.student_grade,
    v_app_doctors.parent_name,
    v_app_doctors.parent_email,
    v_app_doctors.parent_phone,
    v_app_doctors.school_raw,
    v_app_doctors.city,
    v_app_doctors.state,
    v_app_doctors.country,
    v_app_doctors.applied_at,
    v_app_doctors.paid_at,
    v_app_doctors.payment_status_raw,
    v_app_doctors.provider_payment_id,
    v_app_doctors.utm_source,
    v_app_doctors.utm_medium,
    v_app_doctors.utm_campaign,
    v_app_doctors.referral_source,
    v_app_doctors.raw
   FROM migration.v_app_doctors
UNION ALL
 SELECT v_app_edge.source_table,
    v_app_edge.source_row_id,
    v_app_edge.programme_code,
    v_app_edge.cohort_raw,
    v_app_edge.student_name,
    v_app_edge.student_email,
    v_app_edge.student_phone,
    v_app_edge.student_grade,
    v_app_edge.parent_name,
    v_app_edge.parent_email,
    v_app_edge.parent_phone,
    v_app_edge.school_raw,
    v_app_edge.city,
    v_app_edge.state,
-- ... truncated (363 lines total)
```

### migration.v_remote_tables (view)
```sql
 SELECT table_schema,
    table_name
   FROM legacy_ro.remote_tables
  WHERE table_schema <> ALL (ARRAY['pg_catalog'::text, 'information_schema'::text]);
```

## Schema: public

### public.households_v (view) options: security_invoker=on
```sql
 SELECT id,
        CASE
            WHEN ( SELECT has_grant('pii_contact'::staff_grant) AS has_grant) THEN label
            ELSE 'Household '::text || "left"(id::text, 8)
        END AS label,
    source_code,
    created_at
   FROM households h
  WHERE (( SELECT auth_staff_id() AS auth_staff_id)) IS NOT NULL AND (( SELECT has_scope_all() AS has_scope_all) OR (EXISTS ( SELECT 1
           FROM household_members hm
          WHERE hm.household_id = h.id AND can_see_person(hm.person_id))));
```

### public.outreach_data_export (view) options: security_invoker=on
```sql
 SELECT p.id AS contact_id,
    o.name AS school,
    o.website,
    o.school_category,
    o.curriculum,
    o.grades,
    o.accreditation,
    o.student_count,
    o.max_fees_minor,
    o.fees_currency,
    o.academic_calendar,
    o.address AS school_address,
    o.city,
    o.region AS state,
    o.country,
    p.first_name,
    p.last_name,
    p.full_name,
    p.title AS designation,
    ( SELECT i.value_raw
           FROM person_identities i
          WHERE i.person_id = p.id AND i.kind = 'email'::identity_kind AND i.is_primary
         LIMIT 1) AS email_address,
    ( SELECT i.value_raw
           FROM person_identities i
          WHERE i.person_id = p.id AND i.kind = 'phone'::identity_kind
         LIMIT 1) AS contact_number,
    ( SELECT i.value_raw
           FROM person_identities i
          WHERE i.person_id = p.id AND i.kind = 'linkedin'::identity_kind
         LIMIT 1) AS contact_linkedin,
    o.linkedin_url AS company_linkedin_url,
    p.stage,
    p.zone,
    p.source_code,
    (EXISTS ( SELECT 1
           FROM suppressions s
          WHERE s.person_id = p.id)) AS do_not_contact,
    p.created_at,
    p.updated_at
-- ... truncated (44 lines total)
```

### public.people_v (view) options: security_invoker=on
The only way a client reads people. Rows filtered by scope, names masked by grant, and every scope/grant call wrapped in a scalar subquery so it is evaluated once per statement rather than once per row — 7,012 ms to 6 ms (025). Do not add security_invoker: 012 revoked the base table, so an invoker view fails 42501.
```sql
 SELECT id,
    role,
    stage,
    zone,
    country,
    is_minor,
    needs_review,
    graduation_year,
    timezone,
    source_code,
    merged_into,
        CASE
            WHEN is_minor AND NOT ( SELECT has_grant('pii_minor'::staff_grant) AS has_grant) THEN '—'::text
            WHEN NOT is_minor AND NOT ( SELECT has_grant('pii_contact'::staff_grant) AS has_grant) THEN '—'::text
            ELSE full_name
        END AS full_name,
        CASE
            WHEN is_minor AND NOT ( SELECT has_grant('pii_minor'::staff_grant) AS has_grant) THEN NULL::text
            WHEN NOT is_minor AND NOT ( SELECT has_grant('pii_contact'::staff_grant) AS has_grant) THEN NULL::text
            ELSE title
        END AS title,
    created_at,
    updated_at,
    city,
    state
   FROM people p
  WHERE (( SELECT auth_staff_id() AS auth_staff_id)) IS NOT NULL AND (( SELECT has_scope_all() AS has_scope_all) OR can_see_person(id));
```

### public.person_identities_v (view) options: security_invoker=on
```sql
 SELECT id,
    person_id,
    kind,
    is_primary,
    verified_at,
    verify_status,
        CASE
            WHEN NOT ( SELECT has_grant('pii_minor'::staff_grant) AS has_grant) AND (EXISTS ( SELECT 1
               FROM people p
              WHERE p.id = i.person_id AND p.is_minor)) THEN '—'::text
            WHEN NOT ( SELECT has_grant('pii_contact'::staff_grant) AS has_grant) THEN '—'::text
            ELSE value_norm
        END AS value_norm,
    source_code,
    created_at
   FROM person_identities i
  WHERE (( SELECT auth_staff_id() AS auth_staff_id)) IS NOT NULL AND (( SELECT has_scope_all() AS has_scope_all) OR can_see_person(person_id));
```

### public.research_ai_verified (view) options: security_invoker=true
```sql
 SELECT DISTINCT ON ((extra ->> 'contact_id'::text)) (extra ->> 'contact_id'::text)::uuid AS contact_id,
    id AS finding_id,
    task_id,
    school_id,
    value AS title_on_page,
    source_url,
    quote,
    COALESCE((extra ->> 'email_on_page'::text)::boolean, false) AS email_on_page,
    created_at
   FROM research_school_findings f
  WHERE kind = 'contact'::text AND verified AND COALESCE(extra ->> 'contact_id'::text, ''::text) <> ''::text
  ORDER BY (extra ->> 'contact_id'::text), created_at DESC;
```

### public.research_email_pending (view)
```sql
 SELECT email,
    min(priority) AS priority
   FROM research_email_targets t
  WHERE NOT (EXISTS ( SELECT 1
           FROM research_email_checks k
          WHERE k.email = t.email AND NOT k.recheck_requested))
  GROUP BY email;
```

### public.research_email_queue (view)
```sql
 SELECT email,
    label,
    requested_at
   FROM research_email_check_requests r
  WHERE NOT (EXISTS ( SELECT 1
           FROM research_email_checks k
          WHERE k.email = r.email AND NOT k.recheck_requested));
```

### public.research_email_targets (view)
```sql
 SELECT lower(TRIM(BOTH FROM c.email)) AS email,
    'school_contact'::text AS contact_type,
        CASE
            WHEN c.designation <> ''::text THEN 1
            ELSE 2
        END AS priority,
    c.id AS record_id,
    COALESCE(p.contact_code, ''::text) AS contact_code,
    c.contact_name AS name,
    c.designation,
    COALESCE(s.school_code, ''::text) AS school_code,
    c.school AS school_name,
    c.city,
    c.state,
    c.country,
    COALESCE(NULLIF(c.website, ''::text), s.website, ''::text) AS school_website,
    COALESCE(c.phone, ''::text) AS phone,
    COALESCE(c.linkedin_url, ''::text) AS linkedin,
    COALESCE(c.created_by::text, ''::text) AS owner,
    COALESCE(c.source_file, c.source, ''::text) AS source_sheet,
    COALESCE(c.category, ''::text) AS category,
    COALESCE(c.sop_role, ''::text) AS role
   FROM research_contacts c
     LEFT JOIN research_people p ON p.id = c.person_id
     LEFT JOIN research_schools s ON s.id = c.school_id
  WHERE c.deleted_at IS NULL AND COALESCE(TRIM(BOTH FROM c.email), ''::text) ~~ '%@%'::text
UNION ALL
 SELECT lower(TRIM(BOTH FROM p.primary_email)) AS email,
    p.person_type AS contact_type,
    3 AS priority,
    p.id AS record_id,
    p.contact_code,
    p.full_name AS name,
    p.designation,
    COALESCE(s.school_code, ''::text) AS school_code,
    COALESCE(s.name, ''::text) AS school_name,
    COALESCE(s.city, ''::text) AS city,
    COALESCE(s.state, ''::text) AS state,
    COALESCE(s.country, ''::text) AS country,
    COALESCE(s.website, ''::text) AS school_website,
-- ... truncated (51 lines total)
```

### public.research_found_people (view) options: security_invoker=true
```sql
 SELECT DISTINCT ON (f.school_id, (lower(btrim(f.value)))) f.id,
    f.school_id,
    s.school_code,
    s.name AS school,
    s.city,
    s.country,
    btrim(f.value) AS name,
    btrim(f.field) AS title,
    COALESCE(f.extra ->> 'email'::text, ''::text) AS email,
    COALESCE(f.extra ->> 'email_unverified'::text, ''::text) AS email_not_on_page,
    f.source_url,
    f.quote,
    f.created_at AS found_at,
    f.task_id,
    f.why,
        CASE
            WHEN f.applied = 'added as a new contact'::text THEN 'CONTACT_CREATED'::text
            WHEN (EXISTS ( SELECT 1
               FROM research_contacts c
              WHERE c.deleted_at IS NULL AND c.school_id = f.school_id AND lower(btrim(c.contact_name)) = lower(btrim(f.value)))) THEN 'ALREADY_ON_FILE'::text
            WHEN COALESCE(f.extra ->> 'email'::text, ''::text) <> ''::text AND (EXISTS ( SELECT 1
               FROM research_contacts c
              WHERE lower(c.email) = lower(f.extra ->> 'email'::text))) THEN 'ALREADY_ON_FILE'::text
            WHEN COALESCE(f.extra ->> 'email'::text, ''::text) <> ''::text THEN 'LEAD_WITH_EMAIL'::text
            ELSE 'LEAD_NO_EMAIL'::text
        END AS status
   FROM research_school_findings f
     JOIN research_schools s ON s.id = f.school_id
  WHERE f.kind = 'person'::text AND f.verified AND btrim(f.value) <> ''::text
  ORDER BY f.school_id, (lower(btrim(f.value))), f.created_at DESC;
```

### public.research_staff_name_keys (view)
```sql
 WITH s AS (
         SELECT research_staff_profiles.id,
            research_staff_profiles.name,
            research_staff_profiles.email,
            lower(regexp_replace(regexp_replace(research_staff_profiles.name, '[^A-Za-z0-9 ]'::text, ' '::text, 'g'::text), '\s+'::text, ' '::text, 'g'::text)) AS full_n
           FROM research_staff_profiles
        ), k AS (
         SELECT s.id,
            btrim(s.full_n) AS k
           FROM s
        UNION ALL
         SELECT s.id,
            split_part(btrim(s.full_n), ' '::text, 1) AS split_part
           FROM s
        UNION ALL
         SELECT s.id,
            lower(regexp_replace(split_part(COALESCE(s.email, ''::text), '@'::text, 1), '[._-]'::text, ' '::text, 'g'::text)) AS lower
           FROM s
          WHERE COALESCE(s.email, ''::text) <> ''::text
        )
 SELECT k,
    (array_agg(DISTINCT id))[1] AS staff_id
   FROM k
  WHERE k <> ''::text
  GROUP BY k
 HAVING count(DISTINCT id) = 1;
```

### public.v_puller_applications (view)
What the Lovable Cloud app reads: every programme application with its own paid_at, minus the raw source jsonb.
```sql
 SELECT id,
    source_table,
    source_row_id,
    programme_code,
    cohort_raw,
    student_name,
    student_email,
    student_phone,
    student_grade,
    parent_name,
    parent_email,
    parent_phone,
    school_raw,
    school_code,
    city,
    state,
    country,
    applied_at,
    paid_at,
    payment_status_raw,
    provider_payment_id,
    payment_provider_hint,
    utm_source,
    utm_medium,
    utm_campaign,
    referral_source,
    person_id,
    needs_review,
    synced_at
   FROM applications;
```

### public.v_puller_bank_refunds (view)
Bank refunds for the Lovable puller (core_ro). Beneficiary name deliberately left out.
```sql
 SELECT id,
    provider,
    programme_code,
    currency,
    amount_minor,
    refunded_on,
    payment_id,
    link_basis,
    needs_review,
    synced_at
   FROM bank_refunds;
```

### public.v_puller_payments (view)
What the Lovable Cloud app reads: both payment rails in one shape, with provider naming the rail and occurred_at carrying the real payment date.
```sql
 SELECT id,
    provider,
    provider_payment_id,
    amount_minor,
    currency,
    status::text AS status,
    occurred_at,
    payer_email,
    payer_name,
    programme_code,
    utm_source,
    utm_medium,
    utm_campaign,
    method,
    fee_minor,
    refunded_minor,
    person_id,
    cohort_id,
    source_code,
    needs_review,
    description,
    external_ids,
    created_at,
    updated_at
   FROM payments;
```

## Schema: revenue_brain

### revenue_brain.eb_v_suppressed (view) options: security_invoker=on
```sql
 SELECT lower(eb_do_not_contact.email) AS email,
    eb_do_not_contact.reason,
    'eb_do_not_contact'::text AS src
   FROM revenue_brain.eb_do_not_contact
  WHERE eb_do_not_contact.email IS NOT NULL
UNION
 SELECT lower(email_suppression.email) AS email,
    email_suppression.reason,
    'leads.email_suppression'::text AS src
   FROM leads.email_suppression
UNION
 SELECT lower(suppressions.email) AS email,
    suppressions.reason,
    'attribution.suppressions'::text AS src
   FROM attribution.suppressions
UNION
 SELECT lower(research_optouts.email) AS email,
    research_optouts.reason,
    'public.research_optouts'::text AS src
   FROM research_optouts
UNION
 SELECT lower(yale_contacts.email_norm) AS email,
    COALESCE(yale_contacts.suppression_source, 'suppressed'::text) AS reason,
    'revenue_brain.yale_contacts'::text AS src
   FROM revenue_brain.yale_contacts
  WHERE yale_contacts.suppressed IS TRUE AND yale_contacts.email_norm IS NOT NULL;
```

### revenue_brain.open_reply_handoffs (view) options: security_invoker=true
```sql
 SELECT id,
    school_id,
    school_name,
    contact_name,
    contact_email,
    programme_code,
    reply_class,
    classification_confidence,
    priority,
    owner,
    next_action,
    due_at,
    reply_subject,
    reply_snippet,
    unibox_url,
    created_at,
    is_test
   FROM revenue_brain.reply_handoffs
  WHERE handoff_status = ANY (ARRAY['NEW'::text, 'IN_PROGRESS'::text])
  ORDER BY (
        CASE priority
            WHEN 'CRITICAL'::text THEN 1
            WHEN 'HIGH'::text THEN 2
            WHEN 'MEDIUM'::text THEN 3
            ELSE 4
        END), due_at, created_at;
```

### revenue_brain.programme_readiness (view)
```sql
 WITH c AS (
         SELECT claims_registry.programme_code,
            count(*) FILTER (WHERE (claims_registry.status = ANY (ARRAY['APPROVED'::text, 'APPROVED_LOCKED_COPY'::text])) AND (claims_registry.expires_at IS NULL OR claims_registry.expires_at > now())) AS approved,
            count(*) FILTER (WHERE claims_registry.status <> ALL (ARRAY['APPROVED'::text, 'APPROVED_LOCKED_COPY'::text])) AS pending
           FROM revenue_brain.claims_registry
          GROUP BY claims_registry.programme_code
        ), s AS (
         SELECT t.programme_code,
            bool_or(t.status = 'LOCKED'::text AND t.stop_on_reply) AS locked,
            max(( SELECT count(*) AS count
                   FROM revenue_brain.sequence_steps x
                  WHERE x.sequence_id = t.id)) FILTER (WHERE t.status = 'LOCKED'::text) AS steps
           FROM revenue_brain.sequence_templates t
          GROUP BY t.programme_code
        ), r AS (
         SELECT campaign_routes.programme_code,
            bool_or(NOT campaign_routes.test_only AND campaign_routes.provider_campaign_id !~~ 'PENDING%'::text) AS prod_route,
            bool_or(NOT campaign_routes.test_only AND campaign_routes.enabled) AS prod_route_enabled
           FROM revenue_brain.campaign_routes
          GROUP BY campaign_routes.programme_code
        ), q AS (
         SELECT outbound_queue.programme_code,
            count(DISTINCT outbound_queue.school_id) AS schools_in_queue,
            count(*) FILTER (WHERE (outbound_queue.queue_status = ANY (ARRAY['ENROLLED'::text, 'HARD_BOUNCED'::text, 'UNSUBSCRIBED'::text])) OR outbound_queue.queue_status ~~ 'REPLIED%'::text) AS dispatched
           FROM revenue_brain.outbound_queue
          WHERE NOT outbound_queue.test_mode
          GROUP BY outbound_queue.programme_code
        ), e AS (
         SELECT outreach_events.programme_code,
            count(*) FILTER (WHERE outreach_events.event_type = 'SENT'::text) AS sent,
            count(*) FILTER (WHERE outreach_events.event_type = 'REPLY'::text) AS replies,
            count(*) FILTER (WHERE outreach_events.event_type = 'BOUNCE'::text) AS bounces
           FROM revenue_brain.outreach_events
          WHERE NOT outreach_events.is_test
          GROUP BY outreach_events.programme_code
        ), a AS (
         SELECT p_1.programme_code,
            count(x.*) AS apps,
            count(x.paid_at) AS paid
           FROM revenue_brain.programme_profiles p_1
-- ... truncated (117 lines total)
```

### revenue_brain.yale_campus_best_contact (view)
```sql
 SELECT DISTINCT ON ((COALESCE(c.campus_id, lower(c.school)))) COALESCE(c.campus_id, 'NO-ID'::text) AS campus_id,
    c.school,
    c.location,
    sc.school_code,
    c.contact_name,
    c.designation,
    c.precise_role,
    c.email_norm AS email,
    c.role_bucket,
    c.role_relevance,
    c.gate_status,
    c.gate_reasons,
    c.relationship,
    c.lwl_apps_2026,
    c.lwl_paid_2026,
    c.lwl_apps_2027,
    c.lwl_paid_2027,
    c.crm_owner,
    c.crm_stage,
    c.role_verified_raw,
    c.email_publisher_type,
    c.current_role_evidence_url,
    c.current_email_evidence_url,
    c.evidence_checked_on,
    c.campus_merge_status,
    ( SELECT count(*) AS count
           FROM revenue_brain.yale_contacts x
          WHERE COALESCE(x.campus_id, lower(x.school)) = COALESCE(c.campus_id, lower(c.school))) AS contacts_at_campus
   FROM revenue_brain.yale_contacts c
     LEFT JOIN revenue_brain.yale_school_codes sc ON sc.campus_key = COALESCE(c.campus_id, lower(c.school))
  ORDER BY (COALESCE(c.campus_id, lower(c.school))), (
        CASE c.gate_status
            WHEN 'READY_FOR_ZEROBOUNCE'::text THEN 0
            WHEN 'NEEDS_OWNER_CLEARANCE'::text THEN 1
            WHEN 'NEEDS_REVIEW'::text THEN 2
            WHEN 'NEEDS_RESEARCH'::text THEN 3
            ELSE 4
        END), c.role_relevance DESC, c.role_verified DESC, c.email_official DESC, c.id;
```

## Schema: staging

### staging.application_norm (view)
```sql
 SELECT source_table,
    source_id,
    submitted_at,
    student_name,
    student_email,
    student_dob,
    grade,
    school,
    city,
    country,
    parent_name,
    parent_email,
    counsellor_name,
    counsellor_email,
    cohort,
    payment_status,
    NULLIF(lower(btrim(COALESCE(student_email, ''::text))), ''::text) AS se,
    NULLIF(lower(btrim(COALESCE(parent_email, ''::text))), ''::text) AS pe,
    NULLIF(lower(btrim(COALESCE(counsellor_email, ''::text))), ''::text) AS ce,
        CASE
            WHEN student_dob >= (CURRENT_DATE - '80 years'::interval) AND student_dob <= (CURRENT_DATE - '5 years'::interval) THEN student_dob
            ELSE NULL::date
        END AS dob_ok,
    NULLIF("substring"(lower(btrim(COALESCE(grade, ''::text))), '[0-9]+'::text), ''::text)::integer AS grade_no
   FROM staging.applications s
  WHERE lower(btrim(COALESCE(student_email, ''::text))) ~ '^[^[:space:]@]+@[^[:space:]@.]+(\.[^[:space:]@.]+)+$'::text;
```

### staging.inbound_norm (view)
```sql
 SELECT source_table,
    source_id,
    submitted_at,
    full_name,
    email,
    country,
    who,
    grade,
    school_name,
    parent_name,
    parent_email,
    consent,
    NULLIF(lower(btrim(COALESCE(email, ''::text))), ''::text) AS e,
    NULLIF(lower(btrim(COALESCE(parent_email, ''::text))), ''::text) AS pe,
    lower(btrim(COALESCE(who, ''::text))) AS who_norm,
    lower(btrim(COALESCE(grade, ''::text))) AS grade_norm
   FROM staging.inbound_people s
  WHERE lower(btrim(COALESCE(email, ''::text))) ~ '^[^[:space:]@]+@[^[:space:]@.]+(\.[^[:space:]@.]+)+$'::text;
```

