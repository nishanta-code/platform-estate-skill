# Brain project: columns, constraints and indexes for schema `group`

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only.
The group schema is the sales / lead-management platform (meetings, Calendly/Zoom/Gmail/WhatsApp agent tables, deals, SOP tables, member reports). 101 tables.
Legend: one line per table listing `column type` where a trailing `?` means NULLABLE (no `?` = NOT NULL), `=expr` is the column default (generated columns show their expression), and `/*...*/` is the column comment. Below each column line: `PK`/`FK`/`UQ`/`CK`/`EX` constraints (pg_get_constraintdef) and `idx` = indexes not backing a constraint (shown as `name [UNIQUE] USING method (cols) [WHERE ...]`).

## Schema: group (101 tables)

### _wa_resend_audit_20260918
id uuid?, recipient_user_id uuid?, kind text?, ref_id text?, local_date date?, phone text?, template_name text?, status text?, error text?, payload jsonb?, created_at timestamp with time zone?

### agent_decisions
id uuid=gen_random_uuid(), meeting_id uuid?, trigger text, lifecycle_before text?, lifecycle_after text?, next_action text?, selected_program text?, content_type text?, next_contact_date date?, reasoning text?, ai_confidence numeric?, created_at timestamp with time zone=now()
- FK agent_decisions_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK agent_decisions_pkey: PRIMARY KEY (id)
- idx idx_agent_decisions_meeting USING btree (meeting_id, created_at DESC)
- idx idx_agent_decisions_created USING btree (created_at DESC)

### agent_email_examples
id uuid=gen_random_uuid(), meeting_id uuid, member_id uuid?, lead_type text?, school_name text?, programme text?, outcome text, emails jsonb, score numeric=0, refreshed_at timestamp with time zone=now()
- PK agent_email_examples_pkey: PRIMARY KEY (id)
- UQ agent_email_examples_meeting_id_key: UNIQUE (meeting_id)

### agent_gmail_drafts
id uuid=gen_random_uuid(), meeting_id uuid, lead_email text, member_id uuid, mailbox text, kind text, followup_n integer?, gmail_draft_id text?, gmail_message_id text?, gmail_thread_id text?, in_reply_to text?, subject text?, to_email text?, cc_emails text?, body_html text?, reason text?, status text='open'::text, created_at timestamp with time zone=now(), resolved_at timestamp with time zone?, error text?
- CK agent_gmail_drafts_kind_check: CHECK ((kind = ANY (ARRAY['24h'::text, 'followup'::text, 'reply'::text])))
- CK agent_gmail_drafts_status_check: CHECK ((status = ANY (ARRAY['open'::text, 'sent'::text, 'deleted'::text, 'failed'::text])))
- PK agent_gmail_drafts_pkey: PRIMARY KEY (id)
- idx agent_gmail_drafts_one_open UNIQUE USING btree (lower(lead_email)) WHERE (status = 'open'::text)
- idx agent_gmail_drafts_meeting USING btree (meeting_id)

### agent_jobs
id uuid=gen_random_uuid(), agent text, payload jsonb='{}'::jsonb, meeting_id uuid?, status text='pending'::text, attempts integer=0, max_attempts integer=6, next_run_at timestamp with time zone=now(), last_error text?, locked_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK agent_jobs_status_chk: CHECK ((status = ANY (ARRAY['pending'::text, 'running'::text, 'done'::text, 'failed'::text, 'dead_letter'::text])))
- FK agent_jobs_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK agent_jobs_pkey: PRIMARY KEY (id)
- idx idx_agent_jobs_due USING btree (status, next_run_at)
- idx idx_agent_jobs_meeting USING btree (meeting_id, agent)
- idx uniq_agent_jobs_pending UNIQUE USING btree (agent, meeting_id) WHERE (status = ANY (ARRAY['pending'::text, 'running'::text]))

### agent_rate_limits
agent text, window_start timestamp with time zone, count integer=0
- PK agent_rate_limits_pkey: PRIMARY KEY (agent, window_start)

### ai_settings
id boolean=true, enabled boolean=true, note text?, updated_by uuid?, updated_at timestamp with time zone=now()
- CK ai_settings_id_check: CHECK (id)
- PK ai_settings_pkey: PRIMARY KEY (id)

### application_school_matches
id uuid=gen_random_uuid(), name_key text, raw_name text?, application_id uuid?, school_code text?, confidence double precision?, method text, evidence text?, applications_linked integer=0, created_at timestamp with time zone=now()
- PK application_school_matches_pkey: PRIMARY KEY (id)
- idx application_school_matches_key USING btree (name_key, created_at DESC)

### attention_state
item_key text, state text, snooze_until date?, note text?, updated_by uuid?, updated_at timestamp with time zone=now()
- CK attention_state_state_check: CHECK ((state = ANY (ARRAY['done'::text, 'snoozed'::text])))
- FK attention_state_updated_by_fkey: FOREIGN KEY (updated_by) REFERENCES auth.users(id) ON DELETE SET NULL
- PK attention_state_pkey: PRIMARY KEY (item_key)

### automations
slug text, name text, trigger_when text, does text, human_step text, state text='off'::text, sort integer=0, updated_by uuid?, updated_at timestamp with time zone=now()
- CK automations_state_check: CHECK ((state = ANY (ARRAY['off'::text, 'draft'::text, 'on'::text])))
- FK automations_updated_by_fkey: FOREIGN KEY (updated_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK automations_pkey: PRIMARY KEY (slug)

### bf_assignee
meeting_id uuid, user_id uuid

### bf_insert
payload jsonb

### bf_update
id uuid, col text, val text?

### calendly_question_registry
id uuid=gen_random_uuid(), question_norm text, question_label text, event_type_name text?, target_field text?, status text='pending'::text, source text?, confidence numeric?, hits integer=1, sample_answer text?, reviewed_by uuid?, notified_at timestamp with time zone?, backfilled_at timestamp with time zone?, first_seen timestamp with time zone=now(), last_seen timestamp with time zone=now()
- CK calendly_question_registry_field_chk: CHECK (((target_field IS NULL) OR (target_field = ANY (ARRAY['school_name'::text, 'designation'::text, 'city'::text, 'state'::text, 'phone'::text, 'country'::text, 'query'::text, 'linkedin_url'::text]))))
- CK calendly_question_registry_status_chk: CHECK ((status = ANY (ARRAY['pending'::text, 'mapped'::text, 'ignored'::text])))
- FK calendly_question_registry_reviewed_by_fkey: FOREIGN KEY (reviewed_by) REFERENCES auth.users(id) ON DELETE SET NULL
- PK calendly_question_registry_pkey: PRIMARY KEY (id)
- UQ calendly_question_registry_question_norm_key: UNIQUE (question_norm)
- idx calendly_question_registry_status_idx USING btree (status, last_seen DESC)

### campaign_registry
utm_key text, label text, channel text?, first_seen_at timestamp with time zone?=now(), reviewed boolean=false
- PK campaign_registry_pkey: PRIMARY KEY (utm_key)

### core_applications
id uuid, source_table text, source_row_id uuid, programme_code text?, cohort_raw text?, student_name text?, student_email text?, student_phone text?, student_grade text?, parent_name text?, parent_email text?, parent_phone text?, school_raw text?, school_code text?, city text?, state text?, country text?, applied_at timestamp with time zone?, paid_at timestamp with time zone?, payment_status_raw text?, provider_payment_id text?, payment_provider_hint text?, utm_source text?, utm_medium text?, utm_campaign text?, referral_source text?, person_id uuid?, needs_review boolean?, synced_at timestamp with time zone?, pulled_at timestamp with time zone=now(), utm_member_id uuid?
- PK core_applications_pkey: PRIMARY KEY (id)
- idx idx_ca_prog USING btree (programme_code, applied_at)
- idx idx_ca_email USING btree (lower(student_email))
- idx idx_ca_paid USING btree (paid_at)
- idx idx_ca_school USING btree (school_code, applied_at)
- idx core_applications_utm_member_idx USING btree (utm_member_id) WHERE (utm_member_id IS NOT NULL)

### core_applications_removed_20261003
id uuid?, source_table text?, source_row_id uuid?, programme_code text?, cohort_raw text?, student_name text?, student_email text?, student_phone text?, student_grade text?, parent_name text?, parent_email text?, parent_phone text?, school_raw text?, school_code text?, city text?, state text?, country text?, applied_at timestamp with time zone?, paid_at timestamp with time zone?, payment_status_raw text?, provider_payment_id text?, payment_provider_hint text?, utm_source text?, utm_medium text?, utm_campaign text?, referral_source text?, person_id uuid?, needs_review boolean?, synced_at timestamp with time zone?, pulled_at timestamp with time zone?

### core_bank_refunds
id uuid, provider text?, programme_code text?, currency text, amount_minor bigint, refunded_on date?, payment_id uuid?, link_basis text?, needs_review boolean?, synced_at timestamp with time zone?, pulled_at timestamp with time zone=now()
- PK core_bank_refunds_pkey: PRIMARY KEY (id)

### core_payments
id uuid, provider text?, provider_payment_id text?, amount_minor bigint?, currency text?, status text?, occurred_at timestamp with time zone?, payer_email text?, payer_name text?, programme_code text?, utm_source text?, utm_medium text?, utm_campaign text?, method text?, fee_minor bigint?, refunded_minor bigint?, person_id uuid?, cohort_id uuid?, source_code text?, needs_review boolean?, description text?, external_ids jsonb?, created_at timestamp with time zone?, updated_at timestamp with time zone?, pulled_at timestamp with time zone=now()
- PK core_payments_pkey: PRIMARY KEY (id)
- idx core_payments_provider_id_evidence_idx USING btree (provider_payment_id)
- idx idx_cp_provider USING btree (provider, occurred_at)
- idx idx_cp_occurred USING btree (occurred_at)
- idx idx_cp_email USING btree (lower(payer_email))

### core_pull_state
scope text, last_seen timestamp with time zone?, last_run_at timestamp with time zone?, last_rows integer?
- PK core_pull_state_pkey: PRIMARY KEY (scope)

### deal_events
id uuid=gen_random_uuid(), deal_id uuid, occurred_at timestamp with time zone=now(), kind text, channel text?, actor uuid?, meeting_id uuid?, payload jsonb='{}'::jsonb
- FK deal_events_actor_fkey: FOREIGN KEY (actor) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK deal_events_deal_id_fkey: FOREIGN KEY (deal_id) REFERENCES group.deals(id) ON DELETE CASCADE
- FK deal_events_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE SET NULL
- PK deal_events_pkey: PRIMARY KEY (id)
- idx deal_events_deal_idx USING btree (deal_id, occurred_at DESC)
- idx deal_events_meeting_idx USING btree (meeting_id)

### deal_merge_log
id uuid=gen_random_uuid(), source_id uuid, target_id uuid, merge_key text?, moved jsonb='{}'::jsonb, merged_at timestamp with time zone=now(), reversed_at timestamp with time zone?, reversed_reason text?
- PK deal_merge_log_pkey: PRIMARY KEY (id)

### deals
id uuid=gen_random_uuid(), school_code text?, school_name text, contact_name text?, contact_email text?, contact_phone text?, designation text?, decision_maker_name text?, decision_maker_role text?, country text?, region text?, campaign_tag text?, lead_source_type text?, stream text?, programme text?, stage smallint=0, status text='active'::text, stage_entered_at timestamp with time zone=now(), stage_history jsonb='[]'::jsonb, owner uuid?, blocker text?, lost_reason text?, lost_reason_note text?, parked_until date?, parked_from_stage smallint?, parked_reason text?, next_action text?, next_action_owner uuid?, next_action_due date?, last_touch_at timestamp with time zone?, touch_count integer=0, proposal_sent_date date?, proposal_expiry date?, currency text='USD'::text, price_quoted_minor bigint?, scholarship_minor bigint?, pricing_exception_by uuid?, pricing_exception_at timestamp with time zone?, expected_students integer?, expected_close_month date?, approver_name text?, approver_meets_on date?, dates_discussed text?, follow_up_booked_at date?, programme_start date?, programme_end date?, minimum_required integer?, registration_opened_at date?, reg_day15 integer?, reg_day30 integer?, reg_day45 integer?, gate_decision text?, gate_decision_at date?, gate_decision_note text?, payment_collector text?, registered integer?, paid_students integer?, attended integer?, invoice_date date?, invoice_amount_minor bigint?, paid_date date?, paid_amount_minor bigint?, revenue_share_owed_minor bigint?, application_submitted_at date?, interview_at date?, offer_letter_at date?, offer_accepted_at date?, invitation_letter_at date?, deposit_due date?, chaperone_terms text?, chaperone_confirmed_at date?, landing_page_url text?, landing_page_built_at date?, cohort_start_date date?, cohort_session_time text?, pathway text?, committed_students integer?, co_host_school text?, post_delivery jsonb='{"feedback": false, "repeat_ask": false, "testimonial": false, "referral_ask": false, "repeat_record": false}'::jsonb, repeat_of uuid?, cross_sell_of uuid?, notes text?, needs_review boolean=false, created_at timestamp with time zone=now(), created_by uuid?, updated_at timestamp with time zone=now(), probability numeric(4,3)?=
CASE
    WHEN (status <> 'active'::text) THEN (0)::numeric
    WHEN (stage >= 6) THEN 1.000
    WHEN (stage = 5) THEN 0.800
    WHEN (stage = 4) THEN 0.600
    WHEN (stage = 3) THEN 0.300
    WHEN (stage = 2) THEN 0.100
    ELSE (0)::numeric
END, merged_into uuid?, expected_value_minor bigint?=(
CASE
    WHEN (stage >= 2) THEN COALESCE(expected_students, minimum_required, 0)
    ELSE COALESCE(expected_students, 0)
END * GREATEST((COALESCE(price_quoted_minor, (0)::bigint) - COALESCE(scholarship_minor, (0)::bigint)), (0)::bigint)), weighted_value_minor bigint?=(round((((
CASE
    WHEN (stage >= 2) THEN COALESCE(expected_students, minimum_required, 0)
    ELSE COALESCE(expected_students, 0)
END * GREATEST((COALESCE(price_quoted_minor, (0)::bigint) - COALESCE(scholarship_minor, (0)::bigint)), (0)::bigint)))::numeric *
CASE
    WHEN (status <> 'active'::text) THEN (0)::numeric
    WHEN (stage >= 6) THEN 1.000
    WHEN (stage = 5) THEN 0.800
    WHEN (stage = 4) THEN 0.600
    WHEN (stage = 3) THEN 0.300
    WHEN (stage = 2) THEN 0.100
    ELSE (0)::numeric
END)))::bigint, waiting_until date?, waiting_note text?
- CK deals_blocker_check: CHECK (((blocker IS NULL) OR (blocker = ANY (ARRAY['budget'::text, 'approval'::text, 'timing'::text, 'minimum_numbers'::text, 'relevance'::text, 'none'::text]))))
- CK deals_designation_check: CHECK (((designation IS NULL) OR (designation = ANY (ARRAY['principal'::text, 'head_of_school'::text, 'counsellor'::text, 'teacher_coordinator'::text, 'parent'::text, 'student'::text, 'university'::text, 'partner_agency'::text]))))
- CK deals_expected_students_check: CHECK (((expected_students IS NULL) OR (expected_students >= 0)))
- CK deals_gate_decision_check: CHECK (((gate_decision IS NULL) OR (gate_decision = ANY (ARRAY['confirmed'::text, 'rescue'::text, 'postponed'::text, 'converted_online'::text]))))
- CK deals_lead_source_type_check: CHECK (((lead_source_type IS NULL) OR (lead_source_type = ANY (ARRAY['inbound_form'::text, 'outbound_email'::text, 'referral'::text, 'repeat'::text, 'partner'::text]))))
- CK deals_lost_needs_reason: CHECK (((status <> 'lost'::text) OR (lost_reason IS NOT NULL)))
- CK deals_lost_reason_check: CHECK (((lost_reason IS NULL) OR (lost_reason = ANY (ARRAY['budget'::text, 'timing_this_year'::text, 'minimum_numbers'::text, 'relevance'::text, 'chose_competitor'::text, 'venue_only'::text, 'no_response_3_touches'::text, 'internal_approval_declined'::text, 'other'::text]))))
- CK deals_other_needs_note: CHECK (((lost_reason IS DISTINCT FROM 'other'::text) OR (COALESCE(lost_reason_note, ''::text) <> ''::text)))
- CK deals_parked_needs_date: CHECK (((status <> 'parked'::text) OR (parked_until IS NOT NULL)))
- CK deals_payment_collector_check: CHECK (((payment_collector IS NULL) OR (payment_collector = ANY (ARRAY['school'::text, 'group'::text, 'split'::text]))))
- CK deals_price_quoted_minor_check: CHECK (((price_quoted_minor IS NULL) OR (price_quoted_minor >= 0)))
- CK deals_scholarship_minor_check: CHECK (((scholarship_minor IS NULL) OR (scholarship_minor >= 0)))
- CK deals_stage_check: CHECK (((stage >= 0) AND (stage <= 7)))
- CK deals_status_check: CHECK ((status = ANY (ARRAY['active'::text, 'lost'::text, 'parked'::text])))
- CK deals_stream_check: CHECK (((stream IS NULL) OR (stream = ANY (ARRAY['in_person'::text, 'camps'::text, 'online'::text, 'hosting'::text]))))
- CK deals_stream_from_stage2: CHECK (((stage < 2) OR (stream IS NOT NULL)))
- FK deals_created_by_fkey: FOREIGN KEY (created_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK deals_cross_sell_of_fkey: FOREIGN KEY (cross_sell_of) REFERENCES group.deals(id) ON DELETE SET NULL
- FK deals_merged_into_fkey: FOREIGN KEY (merged_into) REFERENCES group.deals(id)
- FK deals_next_action_owner_fkey: FOREIGN KEY (next_action_owner) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK deals_owner_fkey: FOREIGN KEY (owner) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK deals_pricing_exception_by_fkey: FOREIGN KEY (pricing_exception_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK deals_repeat_of_fkey: FOREIGN KEY (repeat_of) REFERENCES group.deals(id) ON DELETE SET NULL
- FK deals_school_code_fkey: FOREIGN KEY (school_code) REFERENCES group.school_registry(code) ON UPDATE CASCADE
- PK deals_pkey: PRIMARY KEY (id)
- idx deals_programme_start_idx USING btree (programme_start)
- idx deals_school_programme_uniq UNIQUE USING btree (COALESCE(school_code, lower(school_name)), COALESCE(stream, ''::text), COALESCE(programme, ''::text)) WHERE ((status = 'active'::text) AND (repeat_of IS NULL) AND (cross_sell_of IS NULL))
- idx deals_stage_status_idx USING btree (status, stage)
- idx deals_stream_idx USING btree (stream)
- idx deals_parked_until_idx USING btree (parked_until) WHERE (status = 'parked'::text)
- idx deals_merged_into_idx USING btree (merged_into)
- idx deals_last_touch_idx USING btree (last_touch_at)
- idx deals_paid_date_idx USING btree (paid_date)
- idx deals_school_code_idx USING btree (school_code)
- idx deals_next_action_idx USING btree (next_action_due) WHERE (status = 'active'::text)
- idx deals_owner_idx USING btree (owner)

### email_body_repair_attempts
kind text, row_id uuid, attempted_at timestamp with time zone=now(), result text, mailbox text?
- PK email_body_repair_attempts_pkey: PRIMARY KEY (kind, row_id)

### email_events
id uuid=gen_random_uuid(), email_send_id uuid?, event_type text, event_data jsonb='{}'::jsonb, user_agent text?, ip_address text?, url text?, created_at timestamp with time zone=now()
- FK email_events_email_send_id_fkey: FOREIGN KEY (email_send_id) REFERENCES group.email_sends(id) ON DELETE CASCADE
- PK email_events_pkey: PRIMARY KEY (id)
- idx idx_email_events_send USING btree (email_send_id)
- idx idx_email_events_type USING btree (event_type)

### email_sends
id uuid=gen_random_uuid(), template_id uuid?, template_slug text?, meeting_id uuid?, to_email text?, subject text?, status text='queued'::text, provider text?, provider_message_id text?, error_message text?, rendered_html text?, variables_used jsonb?, created_at timestamp with time zone=now(), message_id text?, thread_id text?, in_reply_to text?, open_count integer=0, click_count integer=0, first_opened_at timestamp with time zone?, last_opened_at timestamp with time zone?, replied_at timestamp with time zone?, bounced_at timestamp with time zone?, unsubscribed_at timestamp with time zone?, unsubscribe_token text?, direction text='outbound'::text, from_email text?, body_text text?, sent_at timestamp with time zone?, gmail_message_id text?, gmail_thread_id text?, is_thread_root boolean=false, body_html text?, received_at timestamp with time zone?, intent_code text?, intent_tier text?, intent_confidence numeric?, intent_summary text?, asks_for text?, escalation_needed boolean=false, classified_at timestamp with time zone?, draft_of_send_id uuid?, scenario_slug text?, followup_n integer?, attachments jsonb='[]'::jsonb, needs_info jsonb?, info_filled_at timestamp with time zone?, revision_count integer=0, sent_by_user_id uuid?, sent_via_gmail_account_id uuid?
- CK email_sends_followup_n_check: CHECK (((followup_n IS NULL) OR (followup_n = ANY (ARRAY[1, 2, 3]))))
- FK email_sends_draft_of_send_id_fkey: FOREIGN KEY (draft_of_send_id) REFERENCES group.email_sends(id) ON DELETE SET NULL
- FK email_sends_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE SET NULL
- FK email_sends_sent_by_user_id_fkey: FOREIGN KEY (sent_by_user_id) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK email_sends_sent_via_gmail_account_id_fkey: FOREIGN KEY (sent_via_gmail_account_id) REFERENCES group.user_gmail_accounts(id) ON DELETE SET NULL
- FK email_sends_template_id_fkey: FOREIGN KEY (template_id) REFERENCES group.email_templates(id) ON DELETE SET NULL
- PK email_sends_pkey: PRIMARY KEY (id)
- UQ email_sends_gmail_message_unique: UNIQUE (gmail_message_id)
- idx email_sends_thread_msg_unique UNIQUE USING btree (gmail_thread_id, gmail_message_id) WHERE (gmail_message_id IS NOT NULL)
- idx idx_email_sends_status_direction_meeting USING btree (status, direction) WHERE (meeting_id IS NOT NULL)
- idx idx_email_sends_created_at_desc USING btree (created_at DESC)
- idx idx_email_sends_message USING btree (message_id)
- idx idx_email_sends_thread_root USING btree (is_thread_root) WHERE (is_thread_root = true)
- idx email_sends_from_history_idx USING btree (from_email, created_at, id)
- idx idx_email_sends_to USING btree (to_email)
- idx idx_email_sends_gmail_thread USING btree (gmail_thread_id)
- idx email_sends_sent_via_gmail_account_idx USING btree (sent_via_gmail_account_id) WHERE (sent_via_gmail_account_id IS NOT NULL)
- idx idx_email_sends_thread USING btree (thread_id)
- idx idx_email_sends_meeting_id USING btree (meeting_id) WHERE (meeting_id IS NOT NULL)
- idx idx_email_sends_gmail_message USING btree (gmail_message_id)
- idx email_sends_meeting_scenario_uniq UNIQUE USING btree (meeting_id, scenario_slug) WHERE ((meeting_id IS NOT NULL) AND (scenario_slug IS NOT NULL))
- idx idx_email_sends_status USING btree (status)
- idx email_sends_meeting_followup_uniq UNIQUE USING btree (meeting_id, followup_n) WHERE (followup_n IS NOT NULL)
- idx idx_email_sends_meeting USING btree (meeting_id)
- idx idx_email_sends_meeting_created USING btree (meeting_id, created_at) WHERE (meeting_id IS NOT NULL)

### email_settings
id boolean=true, auto_enabled boolean=true, note text?, updated_by uuid?, updated_at timestamp with time zone=now(), auto_drafts_enabled boolean=false, auto_resumed_at timestamp with time zone?
- CK email_settings_id_check: CHECK (id)
- PK email_settings_pkey: PRIMARY KEY (id)

### email_suppression
id uuid=gen_random_uuid(), email text, reason text='unsubscribed'::text, meeting_id uuid?, created_at timestamp with time zone=now()
- PK email_suppression_pkey: PRIMARY KEY (id)
- UQ email_suppression_email_key: UNIQUE (email)

### email_templates
id uuid=gen_random_uuid(), name text, slug text, kind text='manual'::text, trigger_event text?, category text?, description text?, subject text, html text, variables jsonb='[]'::jsonb, is_active boolean=false, created_by text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK email_templates_kind_check: CHECK ((kind = ANY (ARRAY['manual'::text, 'automated'::text])))
- PK email_templates_pkey: PRIMARY KEY (id)
- UQ email_templates_slug_key: UNIQUE (slug)

### followup_plans
id uuid=gen_random_uuid(), meeting_id uuid, status text='active'::text, quiet_until date?, plan_reason text?, stop_reason text?, planned_at timestamp with time zone=now(), planner_version integer=1, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK followup_plans_status_check: CHECK ((status = ANY (ARRAY['active'::text, 'on_hold'::text, 'stopped'::text])))
- FK followup_plans_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK followup_plans_pkey: PRIMARY KEY (id)
- UQ followup_plans_meeting_id_key: UNIQUE (meeting_id)

### followup_report_runs
id uuid=gen_random_uuid(), report_date date, audience text, member_id uuid?, recipient text?, trigger text, test_mode boolean=false, status text, error text?, totals jsonb?, created_at timestamp with time zone=now()
- PK followup_report_runs_pkey: PRIMARY KEY (id)
- idx followup_report_runs_day USING btree (report_date, audience, member_id)

### followup_report_settings
id boolean=true, founder_recipients text[]='{}'::text[], members_enabled boolean=true, test_mode boolean=true, test_recipient text?, updated_at timestamp with time zone=now()
- CK followup_report_settings_id_check: CHECK (id)
- PK followup_report_settings_pkey: PRIMARY KEY (id)

### followup_touches
id uuid=gen_random_uuid(), plan_id uuid, meeting_id uuid, seq integer, followup_n integer, slug text, purpose text, reason text?, due_date date?, status text='scheduled'::text, email_send_id uuid?, created_by text='planner'::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK followup_touches_followup_n_check: CHECK (((followup_n >= 1) AND (followup_n <= 3)))
- CK followup_touches_seq_check: CHECK (((seq >= 1) AND (seq <= 3)))
- CK followup_touches_status_check: CHECK ((status = ANY (ARRAY['scheduled'::text, 'drafted'::text, 'sent'::text, 'skipped'::text, 'expired'::text])))
- FK followup_touches_email_send_id_fkey: FOREIGN KEY (email_send_id) REFERENCES group.email_sends(id) ON DELETE SET NULL
- FK followup_touches_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- FK followup_touches_plan_id_fkey: FOREIGN KEY (plan_id) REFERENCES group.followup_plans(id) ON DELETE CASCADE
- PK followup_touches_pkey: PRIMARY KEY (id)
- UQ followup_touches_meeting_id_seq_key: UNIQUE (meeting_id, seq)
- idx followup_touches_due_idx USING btree (status, due_date)
- idx followup_touches_meeting_idx USING btree (meeting_id)

### gmail_account_requests
id uuid=gen_random_uuid(), user_id uuid, email text, status text='pending'::text, created_at timestamp with time zone=now(), reviewed_at timestamp with time zone?, reviewed_by uuid?
- CK gmail_account_requests_status_check: CHECK ((status = ANY (ARRAY['pending'::text, 'approved'::text, 'rejected'::text])))
- PK gmail_account_requests_pkey: PRIMARY KEY (id)
- UQ gmail_account_requests_user_id_email_key: UNIQUE (user_id, email)

### invitations
id uuid=gen_random_uuid(), email text, role group.app_role='member'::group.app_role, full_name text?, title text?, invited_by uuid?, token text=replace((gen_random_uuid())::text, '-'::text, ''::text), accepted boolean=false, accepted_at timestamp with time zone?, expires_at timestamp with time zone=(now() + '7 days'::interval), created_at timestamp with time zone=now()
- FK invitations_invited_by_fkey: FOREIGN KEY (invited_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK invitations_pkey: PRIMARY KEY (id)
- UQ invitations_token_key: UNIQUE (token)

### knowledge_assets
id uuid=gen_random_uuid(), slug text, type text, title text, description text?, file_url text?, link_url text?, program_id uuid?, is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK knowledge_assets_type_check: CHECK ((type = ANY (ARRAY['brochure'::text, 'flyer'::text, 'proposal'::text, 'one_pager'::text, 'application_link'::text, 'calendly'::text, 'sample_outcome'::text, 'other'::text])))
- FK knowledge_assets_program_id_fkey: FOREIGN KEY (program_id) REFERENCES group.knowledge_programs(id) ON DELETE SET NULL
- PK knowledge_assets_pkey: PRIMARY KEY (id)
- UQ knowledge_assets_slug_key: UNIQUE (slug)

### knowledge_base
key text, value text, description text?, updated_at timestamp with time zone=now()
- PK knowledge_base_pkey: PRIMARY KEY (key)

### knowledge_cadence
id uuid=gen_random_uuid(), n integer, offset_days integer, purpose text, target_words text?, rules text[]?='{}'::text[], is_active boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK knowledge_cadence_pkey: PRIMARY KEY (id)
- UQ knowledge_cadence_n_key: UNIQUE (n)

### knowledge_faq
id uuid=gen_random_uuid(), question text, answer text, program_id uuid?, tags text[]?='{}'::text[], is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK knowledge_faq_program_id_fkey: FOREIGN KEY (program_id) REFERENCES group.knowledge_programs(id) ON DELETE SET NULL
- PK knowledge_faq_pkey: PRIMARY KEY (id)

### knowledge_intents
id uuid=gen_random_uuid(), tier text, code text, signals text[]?='{}'::text[], example text?, default_action text?, response_sla_hours integer?, is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK knowledge_intents_tier_check: CHECK ((tier = ANY (ARRAY['hot'::text, 'warm'::text, 'cold'::text])))
- PK knowledge_intents_pkey: PRIMARY KEY (id)
- UQ knowledge_intents_code_key: UNIQUE (code)

### knowledge_org
id uuid=gen_random_uuid(), name text, short_name text?, mission text?, parents text[]?='{}'::text[], website text?, agent_name text?, agent_email text?, agent_signature text?, cc_on_meeting_confirm text?, finlit_website text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK knowledge_org_pkey: PRIMARY KEY (id)

### knowledge_programs
id uuid=gen_random_uuid(), name text, target text?, format text?, mentors text?, pathways text[]?='{}'::text[], outcomes text?, differentiator text?, website text?, brochure_url text?, application_url text?, contact_email text?, is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), platform text?
- PK knowledge_programs_pkey: PRIMARY KEY (id)
- UQ knowledge_programs_name_key: UNIQUE (name)

### knowledge_scenarios
id uuid=gen_random_uuid(), slug text, name text, when_to_use text?, rules text[]?='{}'::text[], target_words text?, header_variant text?='info'::text, template_html text?, subject_hint text?, is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK knowledge_scenarios_pkey: PRIMARY KEY (id)
- UQ knowledge_scenarios_slug_key: UNIQUE (slug)

### knowledge_voice_rules
id uuid=gen_random_uuid(), kind text, text text, order_index integer=0, is_active boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK knowledge_voice_rules_kind_check: CHECK ((kind = ANY (ARRAY['always'::text, 'never'::text, 'positioning'::text, 'hard_rule'::text])))
- PK knowledge_voice_rules_pkey: PRIMARY KEY (id)

### lead_ai_calls
id uuid=gen_random_uuid(), meeting_id uuid, call_id text?, status text?, to_phone text?, from_phone text?, duration_seconds integer?, started_at timestamp with time zone?, ended_at timestamp with time zone?, summary text?, transcript jsonb?, questions_asked jsonb?, answers jsonb?, sentiment text?, recording_url text?, raw_payload jsonb?, source text?='n8n-bland'::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK lead_ai_calls_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK lead_ai_calls_pkey: PRIMARY KEY (id)
- idx lead_ai_calls_meeting_idx USING btree (meeting_id, created_at DESC)

### applications
id uuid=gen_random_uuid(), source_program text, source_sheet text, student_email text, student_name text?, student_grade text?, student_country text?, student_city text?, raw_school_name text?, raw_status text?, is_paid boolean=false, applied_at timestamp with time zone?, school_code text?, match_confidence double precision?, match_source text?, match_reviewed boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), source_cohort text?, cohort_display text?, paid_at timestamp with time zone?, provider_payment_id text?
- FK applications_school_code_fkey: FOREIGN KEY (school_code) REFERENCES group.school_registry(code) ON DELETE SET NULL
- PK applications_pkey: PRIMARY KEY (id)
- UQ applications_student_email_source_program_key: UNIQUE (student_email, source_program)
- idx idx_lwl_apps_program USING btree (source_program)
- idx idx_lwl_apps_paid USING btree (is_paid) WHERE (is_paid = true)
- idx idx_lwl_apps_email USING btree (student_email)
- idx idx_lwl_apps_school_code USING btree (school_code)
- idx idx_lwl_apps_unmatched USING btree (id) WHERE ((school_code IS NULL) AND (raw_school_name IS NOT NULL))
- idx idx_lwl_apps_school USING btree (school_code)

### meeting_assignees
meeting_id uuid, user_id uuid, assigned_by text?, assigned_at timestamp with time zone=now()
- FK meeting_assignees_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- FK meeting_assignees_user_id_fkey: FOREIGN KEY (user_id) REFERENCES group.user_profiles(id) ON DELETE CASCADE
- PK meeting_assignees_pkey: PRIMARY KEY (meeting_id, user_id)
- idx idx_meeting_assignees_user USING btree (user_id)
- idx idx_meeting_assignees_meeting_assigned USING btree (meeting_id, assigned_at)
- idx idx_meeting_assignees_meeting USING btree (meeting_id)

### meeting_edit_log
id bigint=nextval('group.meeting_edit_log_id_seq'::regclass), meeting_id uuid, field text, old_value text?, new_value text?, changed_by uuid?, changed_at timestamp with time zone=now()
- FK meeting_edit_log_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK meeting_edit_log_pkey: PRIMARY KEY (id)
- idx meeting_edit_log_meeting USING btree (meeting_id, changed_at DESC)

### meeting_followups
id uuid=gen_random_uuid(), meeting_id uuid, user_id uuid, account_id uuid, gmail_msgid text, gmail_thread_id text?, direction text, from_email text?, to_emails text?, subject text?, snippet text?, body_text text?, sent_at timestamp with time zone, hours_after_meeting numeric?, created_at timestamp with time zone=now()
- CK meeting_followups_direction_check: CHECK ((direction = ANY (ARRAY['out'::text, 'in'::text])))
- FK meeting_followups_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK meeting_followups_pkey: PRIMARY KEY (id)
- UQ meeting_followups_account_id_gmail_msgid_key: UNIQUE (account_id, gmail_msgid)
- idx meeting_followups_meeting_idx USING btree (meeting_id, sent_at)
- idx meeting_followups_user_idx USING btree (user_id)

### meeting_participants
id uuid=gen_random_uuid(), meeting_id uuid?, zoom_meeting_id text, participant_uuid text?, zoom_user_id text?, display_name text?, email text?, joined_at timestamp with time zone?, left_at timestamp with time zone?, staff_user_id uuid?, is_staff boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), participant_kind text?
- CK meeting_participants_participant_kind_check: CHECK ((participant_kind = ANY (ARRAY['staff'::text, 'guest'::text, 'shared_account'::text, 'bot'::text])))
- FK meeting_participants_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- FK meeting_participants_staff_user_id_fkey: FOREIGN KEY (staff_user_id) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK meeting_participants_pkey: PRIMARY KEY (id)
- idx meeting_participants_session_uniq UNIQUE USING btree (zoom_meeting_id, COALESCE(participant_uuid, display_name, ''::text), COALESCE(joined_at, '1970-01-01 00:00:00+00'::timestamp with time zone))
- idx meeting_participants_meeting_idx USING btree (meeting_id)
- idx meeting_participants_live_idx USING btree (zoom_meeting_id) WHERE (left_at IS NULL)

### meeting_programme_interest
id uuid=gen_random_uuid(), meeting_id uuid, programme text, temperature text, note text?, recorded_by uuid?=auth.uid(), recorded_at timestamp with time zone=now()
- CK meeting_programme_interest_temperature_check: CHECK ((temperature = ANY (ARRAY['hot'::text, 'warm'::text, 'cold'::text, 'do_not_contact'::text])))
- FK meeting_programme_interest_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK meeting_programme_interest_pkey: PRIMARY KEY (id)
- UQ meeting_programme_interest_once: UNIQUE (meeting_id, programme)
- idx mpi_recorded_at_idx USING btree (recorded_at DESC)
- idx mpi_meeting_idx USING btree (meeting_id)

### meeting_report_runs
id uuid=gen_random_uuid(), kind text, period_from date, period_to date, trigger text, triggered_by uuid?, recipients text[]='{}'::text[], status text, error text?, totals jsonb?, created_at timestamp with time zone=now()
- CK meeting_report_runs_kind_check: CHECK ((kind = ANY (ARRAY['weekly'::text, 'monthly'::text, 'custom'::text])))
- CK meeting_report_runs_status_check: CHECK ((status = ANY (ARRAY['started'::text, 'sent'::text, 'failed'::text, 'skipped'::text])))
- CK meeting_report_runs_trigger_check: CHECK ((trigger = ANY (ARRAY['schedule'::text, 'manual'::text])))
- FK meeting_report_runs_triggered_by_fkey: FOREIGN KEY (triggered_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK meeting_report_runs_pkey: PRIMARY KEY (id)
- idx meeting_report_runs_created USING btree (created_at DESC)

### meeting_report_settings
id boolean=true, recipients text[], updated_by uuid?, updated_at timestamp with time zone=now()
- CK meeting_report_settings_id_check: CHECK (id)
- FK meeting_report_settings_updated_by_fkey: FOREIGN KEY (updated_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK meeting_report_settings_pkey: PRIMARY KEY (id)

### meeting_sync_queue
id uuid=gen_random_uuid(), meeting_id uuid, run_at timestamp with time zone, attempts integer=0, max_attempts integer=3, last_error text?, done boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK meeting_sync_queue_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK meeting_sync_queue_pkey: PRIMARY KEY (id)
- idx meeting_sync_queue_due_idx USING btree (run_at) WHERE (done = false)

### meeting_welcome_jobs
id uuid=gen_random_uuid(), meeting_id uuid, zoom_meeting_id text?, join_url text?, join_at timestamp with time zone, state text='pending'::text, attempts integer=0, last_error text?, session_id text?, skip_reason text?, seen_staff text?, dispatched_at timestamp with time zone?, joined_at timestamp with time zone?, greeted_at timestamp with time zone?, left_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), lead_seen_at timestamp with time zone?, rep_alerted_at timestamp with time zone?, founder_alerted_at timestamp with time zone?, staff_joined_at timestamp with time zone?
- CK meeting_welcome_jobs_state_check: CHECK ((state = ANY (ARRAY['pending'::text, 'dispatched'::text, 'joined'::text, 'greeted'::text, 'left'::text, 'skipped'::text, 'failed'::text, 'cancelled'::text, 'missed'::text, 'waiting'::text, 'rep_alerted'::text, 'founder_alerted'::text, 'handed_over'::text, 'nobody_joined'::text])))
- FK meeting_welcome_jobs_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK meeting_welcome_jobs_pkey: PRIMARY KEY (id)
- idx meeting_welcome_jobs_due_idx USING btree (state, join_at)
- idx meeting_welcome_jobs_meeting_uniq UNIQUE USING btree (meeting_id)
- idx mwj_watch_idx USING btree (state) WHERE (state = ANY (ARRAY['pending'::text, 'waiting'::text, 'rep_alerted'::text, 'founder_alerted'::text]))

### meeting_zoom_links
zoom_meeting_id text, meeting_id uuid, source text, added_by uuid?, created_at timestamp with time zone=now()
- CK meeting_zoom_links_source_check: CHECK ((source = ANY (ARRAY['manual'::text, 'invitee_joined'::text])))
- FK meeting_zoom_links_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK meeting_zoom_links_pkey: PRIMARY KEY (zoom_meeting_id)
- idx meeting_zoom_links_meeting USING btree (meeting_id)

### meetings
id uuid=gen_random_uuid(), meeting_date date, scheduled_time time without time zone, full_name text?, school_name text?, email text?, phone text?, city text?, state text?, country text?, designation text?, meeting_type text?='New Lead'::text, utm_source text?='direct'::text, query text?, meeting_link text?, zoom_meeting_id text?, recording_url text?, calendly_event_id text?, calendly_invitee_id text?, calendly_raw_payload jsonb?, assigned_to text?, number_of_prev_applications integer?=0, meeting_status text?='Scheduled'::text, actual_start timestamp with time zone?, actual_end timestamp with time zone?, duration_minutes integer?, ai_notes text?, meeting_notes text?, zoom_ai_summary text?, transcript_text text?, program_pitched text?, interest_level text?, lifecycle_stage text?='new_lead'::text, follow_up_date text?, followup_email_24h text?, followup_link text?, follow_up_meeting text?, followup1_status text?, followup1_content text?, followup1_response text?, followup2_status text?, followup2_content text?, followup2_response text?, followup3_status text?, followup3_content text?, followup3_response text?, conversion_status text?, conversion_date date?, conversion_notes text?, program_start_date date?, program_end_date date?, paid_student_count integer?, application_count integer?, re_engage_after date?, post_conversion_notes text?, personal_notes text?, linkedin_url text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), utm_medium text?='direct'::text, utm_campaign text?='direct'::text, invitee_timezone text?, next_step text?, key_quotes text?, ai_analyzed_at timestamp with time zone?, ai_confidence numeric?, ai_input_snapshot jsonb?, ai_status text?='idle'::text, ai_error text?, lead_score integer?, lead_temperature text?, objections text?, commitments text?, escalation_needed boolean=false, escalation_reason text?, transcript_url text?, start_time_utc timestamp with time zone?, end_time_utc timestamp with time zone?, followup1_subject text?, followup2_subject text?, followup3_subject text?, followup1_scheduled_for date?, followup2_scheduled_for date?, followup3_scheduled_for date?, followup1_generated_at timestamp with time zone?, followup2_generated_at timestamp with time zone?, followup3_generated_at timestamp with time zone?, last_inbound_at timestamp with time zone?, calendly_start_time_raw text?, calendly_end_time_raw text?, calendly_event_timezone text?, next_contact_date date?, last_agent_decision text?, no_show_count integer=0, nurture_step integer=0, last_nurture_content_type text?, last_nurture_sent_at timestamp with time zone?, attendance_decision_source text?, attendance_decision_evidence text?, attendance_confidence numeric?, attendance_review_required boolean=false, rescheduled_from_invitee_id text?, rescheduled_to_meeting_id uuid?, substitute_attendee_name text?, substitute_attendee_email text?, score_reasons text?, recommended_goal text?, recommended_program_codes text[]?, lead_segment text?, scored_at timestamp with time zone?, school_code text?, school_match_confidence double precision?, school_match_source text?, school_match_reviewed boolean=false, last_agent_action_at timestamp with time zone?, attributed_app_ids uuid[]='{}'::uuid[], attributed_paid_count integer=0, attributed_at timestamp with time zone?, webhook_dispatched_at timestamp with time zone?, webhook_dispatch_status text?, webhook_dispatch_score integer?, host_name text?, host_email text?, host_zoom_account_id text?, host_user_id uuid?, host_source text?, host_resolved_at timestamp with time zone?, zoom_backfill_attempts integer=0, zoom_backfill_last_at timestamp with time zone?, whatsapp_number text?, whatsapp_group_created_at timestamp with time zone?, next_meeting_agreed_at timestamp with time zone?, promised_items jsonb?, commitments_extracted_at timestamp with time zone?, merged_into uuid?, duplicate_review boolean=false, assignment_confirmed_at timestamp with time zone?, assignment_confirmed_by uuid?, zoom_backfill_note text?, deal_id uuid?, lead_type text?, lead_group text?, lead_type_source text?, lead_type_locked boolean=false, lead_type_set_by uuid?, lead_type_at timestamp with time zone?, calendly_host_name text?, calendly_host_email text?, calendly_event_type_name text?, zoom_staff_participants jsonb?, suggested_taker_user_id uuid?, suggested_taker_name text?, suggested_taker_source text?, suggested_taker_at timestamp with time zone?, booking_rep_id uuid?, prep_score integer?, prep_temperature text?, prep_goal text?, prep_reasons text?, prep_signals jsonb?, prep_scored_at timestamp with time zone?, ai_summary_short text?, ai_summary_short_at timestamp with time zone?, manual_overrides jsonb='{}'::jsonb
- CK meetings_lead_score_range: CHECK (((lead_score IS NULL) OR ((lead_score >= 0) AND (lead_score <= 100))))
- FK meetings_assignment_confirmed_by_fkey: FOREIGN KEY (assignment_confirmed_by) REFERENCES group.user_profiles(id)
- FK meetings_booking_rep_id_fkey: FOREIGN KEY (booking_rep_id) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK meetings_deal_id_fkey: FOREIGN KEY (deal_id) REFERENCES group.deals(id) ON DELETE SET NULL
- FK meetings_host_user_id_fkey: FOREIGN KEY (host_user_id) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK meetings_rescheduled_to_meeting_id_fkey: FOREIGN KEY (rescheduled_to_meeting_id) REFERENCES group.meetings(id) ON DELETE SET NULL
- FK meetings_school_code_fkey: FOREIGN KEY (school_code) REFERENCES group.school_registry(code) ON DELETE SET NULL
- FK meetings_suggested_taker_user_id_fkey: FOREIGN KEY (suggested_taker_user_id) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK meetings_pkey: PRIMARY KEY (id)
- UQ meetings_calendly_event_id_key: UNIQUE (calendly_event_id)
- idx idx_meetings_suggested_taker USING btree (suggested_taker_user_id)
- idx idx_meetings_lead_segment USING btree (lead_segment)
- idx idx_meetings_assigned USING btree (assigned_to)
- idx idx_meetings_school_code USING btree (school_code)
- idx idx_meetings_email_norm USING btree (lower(btrim(email)))
- idx idx_meetings_host_email USING btree (lower(host_email))
- idx idx_meetings_needs_match USING btree (id) WHERE ((school_code IS NULL) AND (school_name IS NOT NULL))
- idx idx_meetings_next_contact USING btree (next_contact_date)
- idx idx_meetings_lifecycle USING btree (lifecycle_stage)
- idx meetings_deal_id_idx USING btree (deal_id)
- idx idx_meetings_commitments_pending USING btree (commitments_extracted_at, meeting_date DESC)
- idx meetings_completed_school_evidence_idx USING btree (school_code, meeting_date) WHERE ((meeting_status = 'Completed'::text) AND (merged_into IS NULL))
- idx idx_meetings_zoom_backfill USING btree (zoom_backfill_attempts, meeting_date DESC)
- idx meetings_prep_pending_idx USING btree (meeting_date) WHERE (prep_scored_at IS NULL)
- idx idx_meetings_interest USING btree (interest_level)
- idx idx_calendly_event USING btree (calendly_event_id)
- idx idx_meetings_rescheduled_to USING btree (rescheduled_to_meeting_id) WHERE (rescheduled_to_meeting_id IS NOT NULL)
- idx idx_meetings_lead_type USING btree (lead_type)
- idx idx_meetings_email USING btree (email)
- idx idx_meetings_lead_group USING btree (lead_group)
- idx idx_meetings_lifecycle_stage USING btree (lifecycle_stage)
- idx idx_meetings_date_status USING btree (meeting_date, meeting_status)
- idx idx_meetings_status_date USING btree (meeting_status, meeting_date)
- idx meetings_completed_email_evidence_idx USING btree (lower(btrim(email)), meeting_date) WHERE ((meeting_status = 'Completed'::text) AND (merged_into IS NULL))
- idx idx_meetings_high_score_pending USING btree (scored_at) WHERE ((webhook_dispatched_at IS NULL) AND (lead_score IS NOT NULL))
- idx meetings_calendly_host_email_idx USING btree (calendly_host_email)
- idx idx_meetings_assigned_to USING btree (assigned_to) WHERE (assigned_to IS NOT NULL)
- idx idx_meetings_school USING btree (school_name)
- idx idx_meetings_date_host USING btree (meeting_date, host_user_id)
- idx idx_meetings_host_user_id USING btree (host_user_id)
- idx idx_meetings_date_time USING btree (meeting_date, scheduled_time)
- idx idx_meetings_completed_no_zoom USING btree (end_time_utc) WHERE ((meeting_status = 'Completed'::text) AND (zoom_meeting_id IS NULL) AND (end_time_utc IS NOT NULL))
- idx idx_meetings_date USING btree (meeting_date)
- idx idx_meetings_school_unmatched USING btree (school_name) WHERE ((school_code IS NULL) AND (school_name IS NOT NULL) AND (school_name <> ''::text))
- idx idx_meetings_zoom_meeting_id USING btree (zoom_meeting_id) WHERE (zoom_meeting_id IS NOT NULL)
- idx idx_meetings_attributed_at USING btree (attributed_at)
- idx meetings_date_time_idx USING btree (meeting_date, scheduled_time)
- idx idx_meetings_status USING btree (meeting_status)
- idx idx_meetings_last_agent_action_at USING btree (last_agent_action_at)
- idx idx_meetings_start_time_utc USING btree (start_time_utc)
- idx meetings_booking_rep_idx USING btree (booking_rep_id)

### meetings_sheet_sync_state
id boolean=true, lock_token uuid?, locked_until timestamp with time zone?, cursor_date date?, cursor_time time without time zone?, cursor_id uuid?, last_result jsonb?, updated_at timestamp with time zone=now(), watermark_at timestamp with time zone?, watermark_id uuid?, normalize_row integer=1, review jsonb='{}'::jsonb, kick_pending boolean=false, last_kick_at timestamp with time zone?
- PK meetings_sheet_sync_state_pkey: PRIMARY KEY (id)

### member_daily_email_runs
id uuid=gen_random_uuid(), report_date date, member_id uuid, recipient text, trigger text, test_mode boolean, status text, error text?, totals jsonb?, created_at timestamp with time zone=now()
- PK member_daily_email_runs_pkey: PRIMARY KEY (id)
- idx member_daily_email_runs_day USING btree (report_date, member_id, recipient)

### member_daily_email_settings
id boolean=true, test_mode boolean=true, test_recipients text[]='{}'::text[], updated_at timestamp with time zone=now(), excluded_member_ids uuid[]='{}'::uuid[]
- CK member_daily_email_settings_id_check: CHECK (id)
- PK member_daily_email_settings_pkey: PRIMARY KEY (id)

### member_email_agent
member_id uuid, mailbox text, enabled boolean=false, followup_days integer=7, enabled_by text?, enabled_at timestamp with time zone?=now(), last_run_at timestamp with time zone?, last_result jsonb?
- PK member_email_agent_pkey: PRIMARY KEY (member_id)

### member_report_days
meeting_day date, target_key text, member_id uuid?, spreadsheet_id text, tab_name text, next_refresh_at timestamp with time zone=now(), lock_token uuid?, locked_until timestamp with time zone?, last_success_at timestamp with time zone?, last_result jsonb='{}'::jsonb, revision bigint=0, claimed_revision bigint?
- PK member_report_days_pkey: PRIMARY KEY (meeting_day, target_key)

### member_sheet_reports
member_id uuid, spreadsheet_id text, tab_name text, pilot_day date, enabled boolean=false, lock_token uuid?, locked_until timestamp with time zone?, content_hash text?, last_result jsonb='{}'::jsonb, last_success_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), last_attempt_at timestamp with time zone?
- FK member_sheet_reports_member_id_fkey: FOREIGN KEY (member_id) REFERENCES group.user_profiles(id)
- PK member_sheet_reports_pkey: PRIMARY KEY (member_id)

### notifications
id uuid=gen_random_uuid(), user_id uuid, type text, title text, body text?, meeting_id uuid?, read boolean=false, created_at timestamp with time zone=now(), link_url text?, task_id uuid?, severity text='info'::text, dedupe_key text?
- FK notifications_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- FK notifications_task_id_fkey: FOREIGN KEY (task_id) REFERENCES group.tasks(id) ON DELETE SET NULL
- FK notifications_user_id_fkey: FOREIGN KEY (user_id) REFERENCES group.user_profiles(id) ON DELETE CASCADE
- PK notifications_pkey: PRIMARY KEY (id)
- idx idx_notifications_user_created USING btree (user_id, created_at DESC)
- idx ux_notifications_user_dedupe UNIQUE USING btree (user_id, dedupe_key) WHERE (dedupe_key IS NOT NULL)
- idx idx_notifications_user_unread USING btree (user_id, read, created_at DESC)

### program_cohorts
id uuid=gen_random_uuid(), ext_table_name text, cohort_slug text, program_name text?, cohort_label text?, display_name text?, needs_review boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK program_cohorts_pkey: PRIMARY KEY (id)
- UQ program_cohorts_ext_table_name_cohort_slug_key: UNIQUE (ext_table_name, cohort_slug)

### review_commitments
id uuid=gen_random_uuid(), user_id uuid, body text, set_on date=CURRENT_DATE, due_on date, status text='open'::text, outcome_note text?, created_by uuid?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK review_commitments_status_check: CHECK ((status = ANY (ARRAY['open'::text, 'done'::text, 'missed'::text])))
- FK review_commitments_created_by_fkey: FOREIGN KEY (created_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK review_commitments_user_id_fkey: FOREIGN KEY (user_id) REFERENCES group.user_profiles(id) ON DELETE CASCADE
- PK review_commitments_pkey: PRIMARY KEY (id)
- idx review_commitments_user_idx USING btree (user_id, status)

### scheduled_emails
id uuid=gen_random_uuid(), meeting_id uuid?, email_type text, send_after timestamp with time zone, status text='scheduled'::text, cancelled_reason text?, email_send_id uuid?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK scheduled_emails_email_send_id_fkey: FOREIGN KEY (email_send_id) REFERENCES group.email_sends(id) ON DELETE SET NULL
- FK scheduled_emails_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK scheduled_emails_pkey: PRIMARY KEY (id)
- idx scheduled_emails_meeting_type_active UNIQUE USING btree (meeting_id, email_type) WHERE (status = 'scheduled'::text)
- idx scheduled_emails_status_send_after USING btree (status, send_after)

### school_aliases
id uuid=gen_random_uuid(), raw_text text, school_code text?, match_confidence double precision=1.0, match_source text='seed'::text, times_seen integer=1, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK school_aliases_school_code_fkey: FOREIGN KEY (school_code) REFERENCES group.school_registry(code) ON DELETE SET NULL
- PK school_aliases_pkey: PRIMARY KEY (id)
- idx school_aliases_raw_lower UNIQUE USING btree (lower(TRIM(BOTH FROM raw_text)))

### school_domains
domain text, kind text='school'::text, school_code text?, verified_name text?, verified_at timestamp with time zone?, sightings integer=0, source text?, note text?, updated_at timestamp with time zone=now()
- CK school_domains_kind_check: CHECK ((kind = ANY (ARRAY['school'::text, 'multi_school'::text, 'generic'::text])))
- FK school_domains_school_code_fkey: FOREIGN KEY (school_code) REFERENCES group.school_registry(code) ON DELETE SET NULL
- PK school_domains_pkey: PRIMARY KEY (domain)
- idx school_domains_code_idx USING btree (school_code) WHERE (school_code IS NOT NULL)

### school_match_log
id bigint=nextval('group.school_match_log_id_seq'::regclass), entity text, row_id uuid, old_code text?, new_code text, evidence text, detail text?, batch text, applied_at timestamp with time zone=now()
- PK school_match_log_pkey: PRIMARY KEY (id)

### school_memory
school_code text, summary jsonb='{}'::jsonb, meetings_analysed integer=0, updated_at timestamp with time zone=now()
- FK school_memory_school_code_fkey: FOREIGN KEY (school_code) REFERENCES group.school_registry(code) ON DELETE CASCADE
- PK school_memory_pkey: PRIMARY KEY (school_code)

### school_merge_log
id uuid=gen_random_uuid(), source_code text, target_code text, reason text?, confidence numeric?, rows_repointed jsonb?, merged_at timestamp with time zone=now()
- PK school_merge_log_pkey: PRIMARY KEY (id)

### school_ownership
id uuid=gen_random_uuid(), school_code text, quarter text, owner uuid?, target_minor bigint=0, currency text='USD'::text, principal_name text?, counsellor_name text?, ambassador_name text?, principal_met_at date?, newsletter boolean=false, ambassador_named boolean=false, masterclass_at date?, whatsapp boolean=false, tuesday_emails integer=0, counsellor_meetings integer=0, webinars integer=0, last_friday_update date?, quarters_no_revenue integer=0, locked boolean=false, notes text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK school_ownership_owner_fkey: FOREIGN KEY (owner) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK school_ownership_school_code_fkey: FOREIGN KEY (school_code) REFERENCES group.school_registry(code) ON DELETE CASCADE
- PK school_ownership_pkey: PRIMARY KEY (id)
- UQ school_ownership_school_code_quarter_key: UNIQUE (school_code, quarter)
- idx school_ownership_owner_idx USING btree (owner, quarter)

### school_registry
id uuid=gen_random_uuid(), code text, canonical_name text, country text?, city text?, region text?, is_active boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), auto_created boolean=false, auto_created_at timestamp with time zone?
- PK school_registry_pkey: PRIMARY KEY (id)
- UQ school_registry_code_key: UNIQUE (code)
- idx idx_school_registry_norm USING btree (group.norm_school_name(canonical_name))
- idx idx_sr_code USING btree (code)
- idx idx_sr_name USING btree (canonical_name)

### school_research
school_key text, school_code text?, school_name text, country text?, status text='ready'::text, payload jsonb?, model text?, error text?, requested_by uuid?, generated_at timestamp with time zone=now()
- PK school_research_pkey: PRIMARY KEY (school_key)
- idx school_research_code_idx USING btree (school_code)

### sheet_import_stage
id bigint=nextval('group.sheet_import_stage_id_seq'::regclass), mdate date?, mtime text?, email text?, zoom_id text?, full_name text?, school_name text?, phone text?, city text?, state text?, country text?, designation text?, utm_source text?, query text?, meeting_link text?, linkedin text?, prev_apps text?, owners jsonb?, owner_primary text?, meeting_status text?, ai_notes text?, meeting_notes text?, program_pitched text?, interest_level text?, follow_up_date text?, conversion_status text?, conversion_notes text?, personal_notes text?, matched_meeting_id uuid?, match_method text?
- PK sheet_import_stage_pkey: PRIMARY KEY (id)
- idx idx_sheet_stage_zoom USING btree (zoom_id)
- idx idx_sheet_stage_email_date USING btree (email, mdate)

### sop_academic_calendar
region text, month smallint, status text, note text?
- CK sop_academic_calendar_month_check: CHECK (((month >= 1) AND (month <= 12)))
- CK sop_academic_calendar_status_check: CHECK ((status = ANY (ARRAY['active'::text, 'caution'::text, 'break'::text])))
- PK sop_academic_calendar_pkey: PRIMARY KEY (region, month)

### sop_lists
list text, value text, label text, extra jsonb='{}'::jsonb, sort integer=0, active boolean=true
- PK sop_lists_pkey: PRIMARY KEY (list, value)

### sop_pricing_floors
programme text, quarter text, floor_minor bigint, currency text='USD'::text, set_by uuid?, set_at timestamp with time zone=now()
- CK sop_pricing_floors_floor_minor_check: CHECK ((floor_minor >= 0))
- FK sop_pricing_floors_set_by_fkey: FOREIGN KEY (set_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK sop_pricing_floors_pkey: PRIMARY KEY (programme, quarter)

### sop_targets
quarter text, stream text, target_paid_minor bigint=0, currency text='USD'::text, revenue_per_meeting_minor bigint?, meetings_needed integer?
- CK sop_targets_stream_check: CHECK ((stream = ANY (ARRAY['in_person'::text, 'camps'::text, 'online'::text, 'hosting'::text])))
- PK sop_targets_pkey: PRIMARY KEY (quarter, stream)

### staging_utm_restore_20261021
id uuid?, utm_source text?, utm_medium text?, utm_campaign text?, saved_at timestamp with time zone?

### sync_runs
id uuid=gen_random_uuid(), job_key text, ran_at timestamp with time zone=now(), status text='ok'::text, inserted integer=0, updated integer=0, scanned integer=0, trigger_source text?, detail jsonb?, error text?
- CK sync_runs_status_chk: CHECK ((status = ANY (ARRAY['ok'::text, 'partial'::text, 'error'::text])))
- PK sync_runs_pkey: PRIMARY KEY (id)
- idx idx_sync_runs_ran_at USING btree (ran_at DESC)
- idx sync_runs_job_ran_idx USING btree (job_key, ran_at DESC)

### sync_state
id uuid=gen_random_uuid(), source_key text, last_synced_at timestamp with time zone='2020-01-01 00:00:00+00'::timestamp with time zone, last_row_count integer?=0, new_rows_last_run integer?=0, payment_updates_last_run integer?=0, last_run_at timestamp with time zone?, last_run_status text='never_run'::text, last_error text?, updated_at timestamp with time zone=now(), ext_table_name text?, dry_run_only boolean=true, ext_source text='ext1'::text
- PK sync_state_pkey: PRIMARY KEY (id)
- UQ sync_state_source_key_key: UNIQUE (source_key)

### system_health
check_key text, label text, status text, detail text?, failing_since timestamp with time zone?, checked_at timestamp with time zone=now()
- CK system_health_status_check: CHECK ((status = ANY (ARRAY['ok'::text, 'warn'::text, 'fail'::text])))
- PK system_health_pkey: PRIMARY KEY (check_key)

### tasks
id uuid=gen_random_uuid(), type group.task_type='other'::group.task_type, title text, description text?, priority group.task_priority='med'::group.task_priority, status group.task_status='open'::group.task_status, meeting_id uuid?, email_send_id uuid?, assigned_to uuid?, created_by_agent boolean=false, due_at timestamp with time zone?, completed_at timestamp with time zone?, completed_by uuid?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), snooze_until timestamp with time zone?, due_date date?, created_by text='system'::text, attachment_url text?, attachment_name text?, needs_info jsonb?, revision_request text?, revision_count integer=0, escalation_template_slug text?, dedupe_key text?, completion_note text?, archived boolean=false, deal_id uuid?, sop_kind text?, sop_key text?
- FK tasks_assigned_to_fkey: FOREIGN KEY (assigned_to) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK tasks_completed_by_fkey: FOREIGN KEY (completed_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- FK tasks_deal_id_fkey: FOREIGN KEY (deal_id) REFERENCES group.deals(id) ON DELETE CASCADE
- FK tasks_email_send_id_fkey: FOREIGN KEY (email_send_id) REFERENCES group.email_sends(id) ON DELETE SET NULL
- FK tasks_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE CASCADE
- PK tasks_pkey: PRIMARY KEY (id)
- idx ux_tasks_dedupe_key UNIQUE USING btree (dedupe_key) WHERE (dedupe_key IS NOT NULL)
- idx tasks_archived_status_idx USING btree (archived, status)
- idx tasks_one_active_escalation_per_meeting UNIQUE USING btree (meeting_id) WHERE ((type = 'escalation'::group.task_type) AND (status = ANY (ARRAY['pending'::group.task_status, 'open'::group.task_status, 'needs_info'::group.task_status, 'revision_requested'::group.task_status, 'revising'::group.task_status, 'snoozed'::group.task_status])))
- idx idx_tasks_assigned USING btree (assigned_to) WHERE (status = 'open'::group.task_status)
- idx idx_tasks_due_date USING btree (due_date)
- idx tasks_deal_id_idx USING btree (deal_id)
- idx tasks_sop_key_uniq UNIQUE USING btree (sop_key)
- idx idx_tasks_snooze USING btree (snooze_until) WHERE (status = 'snoozed'::group.task_status)
- idx idx_tasks_meeting USING btree (meeting_id)
- idx idx_tasks_created_at_desc USING btree (created_at DESC)
- idx idx_tasks_meeting_status USING btree (meeting_id, status)
- idx idx_tasks_email USING btree (email_send_id)
- idx idx_tasks_assigned_to_status USING btree (assigned_to, status) WHERE (assigned_to IS NOT NULL)
- idx idx_tasks_status USING btree (status)
- idx tasks_one_active_per_email_send UNIQUE USING btree (email_send_id) WHERE ((email_send_id IS NOT NULL) AND (status = ANY (ARRAY['pending'::group.task_status, 'open'::group.task_status, 'needs_info'::group.task_status, 'revision_requested'::group.task_status, 'revising'::group.task_status, 'snoozed'::group.task_status])))

### team_mailbox_import
mailbox text, backfill_status text='pending'::text, backfill_started_at timestamp with time zone?, backfill_heartbeat_at timestamp with time zone?, backfill_completed_at timestamp with time zone?, backfill_cursor integer=0, backfill_offset integer=0, backfilled_count integer=0, backfill_error text?
- CK team_mailbox_import_backfill_status_check: CHECK ((backfill_status = ANY (ARRAY['pending'::text, 'running'::text, 'done'::text, 'failed'::text])))
- PK team_mailbox_import_pkey: PRIMARY KEY (mailbox)

### user_gmail_accounts
id uuid=gen_random_uuid(), user_id uuid, email text, app_password_encrypted text?, status text='connected'::text, last_polled_at timestamp with time zone?, last_error text?, connected_at timestamp with time zone=now(), created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), sender_name text?, is_primary boolean=false, backfill_status text='pending'::text, backfill_started_at timestamp with time zone?, backfill_completed_at timestamp with time zone?, backfill_error text?, backfill_cursor integer=0, backfilled_count integer=0, followup_synced_at timestamp with time zone?, backfill_heartbeat_at timestamp with time zone?, backfill_offset integer=0
- CK user_gmail_accounts_status_check: CHECK ((status = ANY (ARRAY['connected'::text, 'error'::text, 'disconnected'::text])))
- FK user_gmail_accounts_user_id_fkey: FOREIGN KEY (user_id) REFERENCES group.user_profiles(id) ON DELETE CASCADE
- PK user_gmail_accounts_pkey: PRIMARY KEY (id)
- UQ user_gmail_accounts_user_email_key: UNIQUE (user_id, email)
- idx user_gmail_accounts_one_primary_idx UNIQUE USING btree (user_id) WHERE is_primary
- idx user_gmail_accounts_one_primary UNIQUE USING btree (user_id) WHERE is_primary

### user_profiles
id uuid, full_name text=''::text, email text?, title text?, phone text?, avatar_color text?, is_active boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), role group.app_role='member'::group.app_role, calendly_link text?, zoom_account_id text?, invited_by uuid?, invited_at timestamp with time zone?, last_login_at timestamp with time zone?, timezone text='Asia/Kolkata'::text, wa_opt_in boolean=true
- FK user_profiles_id_fkey: FOREIGN KEY (id) REFERENCES auth.users(id) ON DELETE CASCADE
- FK user_profiles_invited_by_fkey: FOREIGN KEY (invited_by) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK user_profiles_pkey: PRIMARY KEY (id)
- idx idx_user_profiles_full_name USING btree (full_name)

### view_def_backup
name text?, saved_at timestamp with time zone?=now(), def text?

### wa_agent_runs
id uuid=gen_random_uuid(), inbound_id uuid?, user_id uuid?, question text?, answer text?, tools jsonb='[]'::jsonb, status text='ok'::text, error text?, local_date date=((now() AT TIME ZONE 'Asia/Kolkata'::text))::date, created_at timestamp with time zone=now()
- FK wa_agent_runs_inbound_id_fkey: FOREIGN KEY (inbound_id) REFERENCES group.wa_inbound(id) ON DELETE CASCADE
- FK wa_agent_runs_user_id_fkey: FOREIGN KEY (user_id) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK wa_agent_runs_pkey: PRIMARY KEY (id)
- idx wa_agent_runs_user_day_idx USING btree (user_id, local_date)

### wa_claim_taps
wati_message_id text, phone text, user_id uuid?, outcome text?, created_at timestamp with time zone=now()
- FK wa_claim_taps_user_id_fkey: FOREIGN KEY (user_id) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK wa_claim_taps_pkey: PRIMARY KEY (wati_message_id)

### wa_inbound
id uuid=gen_random_uuid(), wati_message_id text, phone text, user_id uuid?, role text?, text text?, received_at timestamp with time zone=now(), handled boolean=false, skip_reason text?, created_at timestamp with time zone=now()
- FK wa_inbound_user_id_fkey: FOREIGN KEY (user_id) REFERENCES group.user_profiles(id) ON DELETE SET NULL
- PK wa_inbound_pkey: PRIMARY KEY (id)
- UQ wa_inbound_wati_message_id_key: UNIQUE (wati_message_id)
- idx wa_inbound_pending_idx USING btree (handled, received_at DESC)

### wa_limits
role text, daily_cap integer, note text?, updated_at timestamp with time zone=now()
- CK wa_limits_daily_cap_check: CHECK ((daily_cap >= 0))
- CK wa_limits_role_check: CHECK ((role = ANY (ARRAY['owner'::text, 'admin'::text, 'member'::text])))
- PK wa_limits_pkey: PRIMARY KEY (role)

### wa_poll_state
phone text, last_message_at timestamp with time zone?, last_polled_at timestamp with time zone=now()
- PK wa_poll_state_pkey: PRIMARY KEY (phone)

### wa_sends
id uuid=gen_random_uuid(), recipient_user_id uuid, kind text, ref_id text=''::text, local_date date, phone text?, template_name text?, status text='sent'::text, error text?, payload jsonb?, created_at timestamp with time zone=now(), wati_local_id text?
- PK wa_sends_pkey: PRIMARY KEY (id)
- UQ wa_sends_recipient_user_id_kind_ref_id_local_date_key: UNIQUE (recipient_user_id, kind, ref_id, local_date)
- idx wa_sends_claim_lookup_idx USING btree (recipient_user_id, kind, created_at DESC) WHERE (kind = 'team_claims'::text)
- idx wa_sends_created_idx USING btree (created_at DESC)
- idx wa_sends_wati_local_id_idx USING btree (wati_local_id) WHERE (wati_local_id IS NOT NULL)

### wa_session_window
phone text, last_inbound_at timestamp with time zone?, checked_at timestamp with time zone=now()
- PK wa_session_window_pkey: PRIMARY KEY (phone)

### wa_settings
id boolean=true, enabled boolean=false, app_link text='https://lwlleadmanagement.lovable.app'::text, updated_at timestamp with time zone=now()
- CK wa_settings_id_check: CHECK (id)
- PK wa_settings_pkey: PRIMARY KEY (id)

### wa_templates
key text, template_name text?, broadcast_name text?, enabled boolean=false, updated_at timestamp with time zone=now(), body_text text?
- PK wa_templates_pkey: PRIMARY KEY (key)

### webhook_logs
id uuid=gen_random_uuid(), source text='calendly'::text, event_type text?, calendly_event_id text?, calendly_invitee_id text?, payload jsonb, status text='received'::text, error_message text?, retry_count integer=0, meeting_id uuid?, processed_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK webhook_logs_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES group.meetings(id) ON DELETE SET NULL
- PK webhook_logs_pkey: PRIMARY KEY (id)
- idx idx_webhook_logs_invitee USING btree (calendly_invitee_id)
- idx idx_webhook_logs_status USING btree (status)
- idx idx_webhook_logs_created USING btree (created_at DESC)

### weekly_commitments
id uuid=gen_random_uuid(), user_id uuid, week_start date, meetings_planned integer=0, proposals_planned integer=0, closes_planned integer=0, touches_planned integer=0, note text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK weekly_commitments_user_id_fkey: FOREIGN KEY (user_id) REFERENCES group.user_profiles(id) ON DELETE CASCADE
- PK weekly_commitments_pkey: PRIMARY KEY (id)
- UQ weekly_commitments_user_id_week_start_key: UNIQUE (user_id, week_start)

### weekly_review_runs
id uuid=gen_random_uuid(), ran_at timestamp with time zone=now(), week_start date, week_end date, summary jsonb='{}'::jsonb, fixes_applied integer=0, issues_found integer=0, unresolved jsonb='[]'::jsonb, duration_ms integer?
- PK weekly_review_runs_pkey: PRIMARY KEY (id)
- idx idx_weekly_review_runs_ran_at USING btree (ran_at DESC)

