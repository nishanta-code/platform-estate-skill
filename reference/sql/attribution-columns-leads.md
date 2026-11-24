# Brain project: columns, constraints and indexes for schema `leads`

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only.
The leads schema is the earlier generation of the lead-management platform; its table set mirrors a subset of group.* (39 tables).
Legend: one line per table listing `column type` where a trailing `?` means NULLABLE (no `?` = NOT NULL), `=expr` is the column default (generated columns show their expression), and `/*...*/` is the column comment. Below each column line: `PK`/`FK`/`UQ`/`CK`/`EX` constraints (pg_get_constraintdef) and `idx` = indexes not backing a constraint (shown as `name [UNIQUE] USING method (cols) [WHERE ...]`).

## leads

### agent_decisions
id uuid=gen_random_uuid(), meeting_id uuid?, trigger text, lifecycle_before text?, lifecycle_after text?, next_action text?, selected_program text?, content_type text?, next_contact_date date?, reasoning text?, ai_confidence numeric?, created_at timestamp with time zone=now()
- FK agent_decisions_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE CASCADE
- PK agent_decisions_pkey: PRIMARY KEY (id)
- idx idx_agent_decisions_created USING btree (created_at DESC)
- idx idx_agent_decisions_meeting USING btree (meeting_id, created_at DESC)

### agent_jobs
id uuid=gen_random_uuid(), agent text, payload jsonb='{}'::jsonb, meeting_id uuid?, status text='pending'::text, attempts integer=0, max_attempts integer=6, next_run_at timestamp with time zone=now(), last_error text?, locked_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK agent_jobs_status_chk: CHECK ((status = ANY (ARRAY['pending'::text, 'running'::text, 'done'::text, 'failed'::text, 'dead_letter'::text])))
- FK agent_jobs_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE CASCADE
- PK agent_jobs_pkey: PRIMARY KEY (id)
- idx idx_agent_jobs_due USING btree (status, next_run_at)
- idx idx_agent_jobs_meeting USING btree (meeting_id, agent)
- idx uniq_agent_jobs_pending UNIQUE USING btree (agent, meeting_id) WHERE (status = ANY (ARRAY['pending'::text, 'running'::text]))

### agent_rate_limits
agent text, window_start timestamp with time zone, count integer=0
- PK agent_rate_limits_pkey: PRIMARY KEY (agent, window_start)

### deal_events
id uuid=gen_random_uuid(), deal_id uuid, occurred_at timestamp with time zone=now(), kind text, channel text?, actor uuid?, meeting_id uuid?, payload jsonb='{}'::jsonb
- FK deal_events_actor_fkey: FOREIGN KEY (actor) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- FK deal_events_deal_id_fkey: FOREIGN KEY (deal_id) REFERENCES leads.deals(id) ON DELETE CASCADE
- FK deal_events_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE SET NULL
- PK deal_events_pkey: PRIMARY KEY (id)
- idx deal_events_deal_idx USING btree (deal_id, occurred_at DESC)
- idx deal_events_meeting_idx USING btree (meeting_id)

### deals
id uuid=gen_random_uuid(), school_code text?, school_name text, contact_name text?, contact_email text?, contact_phone text?, designation text?, decision_maker_name text?, decision_maker_role text?, country text?, region text?, campaign_tag text?, lead_source_type text?, stream text?, programme text?, stage smallint=0, status text='active'::text, stage_entered_at timestamp with time zone=now(), stage_history jsonb='[]'::jsonb /*Append-only [{stage,status,at,by}]. Feeds average-days-in-stage and week-on-week movement (SOP 7.2).*/, owner uuid?, blocker text?, lost_reason text?, lost_reason_note text?, parked_until date?, parked_from_stage smallint?, parked_reason text?, next_action text?, next_action_owner uuid?, next_action_due date?, last_touch_at timestamp with time zone?, touch_count integer=0, proposal_sent_date date?, proposal_expiry date?, currency text='USD'::text, price_quoted_minor bigint?, scholarship_minor bigint?, pricing_exception_by uuid?, pricing_exception_at timestamp with time zone?, expected_students integer?, expected_close_month date?, approver_name text?, approver_meets_on date?, dates_discussed text?, follow_up_booked_at date?, programme_start date?, programme_end date?, minimum_required integer?, registration_opened_at date?, reg_day15 integer?, reg_day30 integer?, reg_day45 integer?, gate_decision text?, gate_decision_at date?, gate_decision_note text?, payment_collector text?, registered integer?, paid_students integer?, attended integer?, invoice_date date?, invoice_amount_minor bigint?, paid_date date?, paid_amount_minor bigint?, revenue_share_owed_minor bigint?, application_submitted_at date?, interview_at date?, offer_letter_at date?, offer_accepted_at date?, invitation_letter_at date?, deposit_due date?, chaperone_terms text?, chaperone_confirmed_at date?, landing_page_url text?, landing_page_built_at date?, cohort_start_date date?, cohort_session_time text?, pathway text?, committed_students integer?, co_host_school text?, post_delivery jsonb='{"feedback": false, "repeat_ask": false, "testimonial": false, "referral_ask": false, "repeat_record": false}'::jsonb, repeat_of uuid?, cross_sell_of uuid?, notes text?, needs_review boolean=false, created_at timestamp with time zone=now(), created_by uuid?, updated_at timestamp with time zone=now(), probability numeric(4,3)?=
CASE
    WHEN (status <> 'active'::text) THEN (0)::numeric
    WHEN (stage >= 6) THEN 1.000
    WHEN (stage = 5) THEN 0.800
    WHEN (stage = 4) THEN 0.600
    WHEN (stage = 3) THEN 0.300
    WHEN (stage = 2) THEN 0.100
    ELSE (0)::numeric
END /*Auto from stage (SOP 6.2): 2=10%, 3=30%, 4=60%, 5=80%, 6+=100%, non-active=0. Never typed by hand.*/, expected_value_minor bigint?=(COALESCE(expected_students, 0) * GREATEST((COALESCE(price_quoted_minor, (0)::bigint) - COALESCE(scholarship_minor, (0)::bigint)), (0)::bigint)), weighted_value_minor bigint?=(round((((COALESCE(expected_students, 0) * GREATEST((COALESCE(price_quoted_minor, (0)::bigint) - COALESCE(scholarship_minor, (0)::bigint)), (0)::bigint)))::numeric *
CASE
    WHEN (status <> 'active'::text) THEN (0)::numeric
    WHEN (stage >= 6) THEN 1.000
    WHEN (stage = 5) THEN 0.800
    WHEN (stage = 4) THEN 0.600
    WHEN (stage = 3) THEN 0.300
    WHEN (stage = 2) THEN 0.100
    ELSE (0)::numeric
END)))::bigint
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
- FK deals_created_by_fkey: FOREIGN KEY (created_by) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- FK deals_cross_sell_of_fkey: FOREIGN KEY (cross_sell_of) REFERENCES leads.deals(id) ON DELETE SET NULL
- FK deals_next_action_owner_fkey: FOREIGN KEY (next_action_owner) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- FK deals_owner_fkey: FOREIGN KEY (owner) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- FK deals_pricing_exception_by_fkey: FOREIGN KEY (pricing_exception_by) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- FK deals_repeat_of_fkey: FOREIGN KEY (repeat_of) REFERENCES leads.deals(id) ON DELETE SET NULL
- FK deals_school_code_fkey: FOREIGN KEY (school_code) REFERENCES leads.school_registry(code) ON UPDATE CASCADE
- PK deals_pkey: PRIMARY KEY (id)
- idx deals_school_programme_uniq UNIQUE USING btree (COALESCE(school_code, lower(school_name)), COALESCE(stream, ''::text), COALESCE(programme, ''::text)) WHERE ((status = 'active'::text) AND (repeat_of IS NULL) AND (cross_sell_of IS NULL))
- idx deals_stage_status_idx USING btree (status, stage)
- idx deals_owner_idx USING btree (owner)
- idx deals_last_touch_idx USING btree (last_touch_at)
- idx deals_next_action_idx USING btree (next_action_due) WHERE (status = 'active'::text)
- idx deals_parked_until_idx USING btree (parked_until) WHERE (status = 'parked'::text)
- idx deals_programme_start_idx USING btree (programme_start)
- idx deals_school_code_idx USING btree (school_code)
- idx deals_paid_date_idx USING btree (paid_date)
- idx deals_stream_idx USING btree (stream)

### email_events
id uuid=gen_random_uuid(), email_send_id uuid?, event_type text, event_data jsonb='{}'::jsonb, user_agent text?, ip_address text?, url text?, created_at timestamp with time zone=now()
- FK email_events_email_send_id_fkey: FOREIGN KEY (email_send_id) REFERENCES leads.email_sends(id) ON DELETE CASCADE
- PK email_events_pkey: PRIMARY KEY (id)
- idx idx_email_events_send USING btree (email_send_id)
- idx idx_email_events_type USING btree (event_type)

### email_sends
id uuid=gen_random_uuid(), template_id uuid?, template_slug text?, meeting_id uuid?, to_email text?, subject text?, status text='queued'::text, provider text?, provider_message_id text?, error_message text?, rendered_html text?, variables_used jsonb?, created_at timestamp with time zone=now(), message_id text?, thread_id text?, in_reply_to text?, open_count integer=0, click_count integer=0, first_opened_at timestamp with time zone?, last_opened_at timestamp with time zone?, replied_at timestamp with time zone?, bounced_at timestamp with time zone?, unsubscribed_at timestamp with time zone?, unsubscribe_token text?, direction text='outbound'::text, from_email text?, body_text text?, sent_at timestamp with time zone?, gmail_message_id text?, gmail_thread_id text?, is_thread_root boolean=false, body_html text?, received_at timestamp with time zone?, intent_code text?, intent_tier text?, intent_confidence numeric?, intent_summary text?, asks_for text?, escalation_needed boolean=false, classified_at timestamp with time zone?, draft_of_send_id uuid?, scenario_slug text?, followup_n integer?, attachments jsonb='[]'::jsonb, needs_info jsonb?, info_filled_at timestamp with time zone?, revision_count integer=0
- CK email_sends_followup_n_check: CHECK (((followup_n IS NULL) OR (followup_n = ANY (ARRAY[1, 2, 3]))))
- FK email_sends_draft_of_send_id_fkey: FOREIGN KEY (draft_of_send_id) REFERENCES leads.email_sends(id) ON DELETE SET NULL
- FK email_sends_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE SET NULL
- FK email_sends_template_id_fkey: FOREIGN KEY (template_id) REFERENCES leads.email_templates(id) ON DELETE SET NULL
- PK email_sends_pkey: PRIMARY KEY (id)
- UQ email_sends_gmail_message_unique: UNIQUE (gmail_message_id)
- idx email_sends_meeting_followup_uniq UNIQUE USING btree (meeting_id, followup_n) WHERE (followup_n IS NOT NULL)
- idx email_sends_meeting_scenario_uniq UNIQUE USING btree (meeting_id, scenario_slug) WHERE ((meeting_id IS NOT NULL) AND (scenario_slug IS NOT NULL))
- idx email_sends_thread_msg_unique UNIQUE USING btree (gmail_thread_id, gmail_message_id) WHERE (gmail_message_id IS NOT NULL)
- idx idx_email_sends_created_at_desc USING btree (created_at DESC)
- idx idx_email_sends_gmail_message USING btree (gmail_message_id)
- idx idx_email_sends_gmail_thread USING btree (gmail_thread_id)
- idx idx_email_sends_meeting USING btree (meeting_id)
- idx idx_email_sends_meeting_id USING btree (meeting_id) WHERE (meeting_id IS NOT NULL)
- idx idx_email_sends_message USING btree (message_id)
- idx idx_email_sends_status USING btree (status)
- idx idx_email_sends_status_direction_meeting USING btree (status, direction) WHERE (meeting_id IS NOT NULL)
- idx idx_email_sends_thread USING btree (thread_id)
- idx idx_email_sends_thread_root USING btree (is_thread_root) WHERE (is_thread_root = true)
- idx idx_email_sends_to USING btree (to_email)

### email_suppression
id uuid=gen_random_uuid(), email text, reason text='unsubscribed'::text, meeting_id uuid?, created_at timestamp with time zone=now()
- PK email_suppression_pkey: PRIMARY KEY (id)
- UQ email_suppression_email_key: UNIQUE (email)

### email_templates
id uuid=gen_random_uuid(), name text, slug text, kind text='manual'::text, trigger_event text?, category text?, description text?, subject text, html text, variables jsonb='[]'::jsonb, is_active boolean=false, created_by text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK email_templates_kind_check: CHECK ((kind = ANY (ARRAY['manual'::text, 'automated'::text])))
- PK email_templates_pkey: PRIMARY KEY (id)
- UQ email_templates_slug_key: UNIQUE (slug)

### invitations
id uuid=gen_random_uuid(), email text, role leads.app_role='member'::leads.app_role, full_name text?, title text?, invited_by uuid?, token text=replace((gen_random_uuid())::text, '-'::text, ''::text), accepted boolean=false, accepted_at timestamp with time zone?, expires_at timestamp with time zone=(now() + '7 days'::interval), created_at timestamp with time zone=now()
- FK invitations_invited_by_fkey: FOREIGN KEY (invited_by) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- PK invitations_pkey: PRIMARY KEY (id)
- UQ invitations_token_key: UNIQUE (token)

### knowledge_assets
id uuid=gen_random_uuid(), slug text, type text, title text, description text?, file_url text?, link_url text?, program_id uuid?, is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK knowledge_assets_type_check: CHECK ((type = ANY (ARRAY['brochure'::text, 'flyer'::text, 'proposal'::text, 'one_pager'::text, 'application_link'::text, 'calendly'::text, 'sample_outcome'::text, 'other'::text])))
- FK knowledge_assets_program_id_fkey: FOREIGN KEY (program_id) REFERENCES leads.knowledge_programs(id) ON DELETE SET NULL
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
- FK knowledge_faq_program_id_fkey: FOREIGN KEY (program_id) REFERENCES leads.knowledge_programs(id) ON DELETE SET NULL
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
- FK lead_ai_calls_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE CASCADE
- PK lead_ai_calls_pkey: PRIMARY KEY (id)
- idx lead_ai_calls_meeting_idx USING btree (meeting_id, created_at DESC)

### applications
id uuid=gen_random_uuid(), source_program text, source_sheet text, student_email text, student_name text?, student_grade text?, student_country text?, student_city text?, raw_school_name text?, raw_status text?, is_paid boolean=false, applied_at timestamp with time zone?, school_code text?, match_confidence double precision?, match_source text?, match_reviewed boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), source_cohort text?, cohort_display text?
- FK applications_school_code_fkey: FOREIGN KEY (school_code) REFERENCES leads.school_registry(code) ON DELETE SET NULL
- PK applications_pkey: PRIMARY KEY (id)
- UQ applications_student_email_source_program_key: UNIQUE (student_email, source_program)
- idx idx_lwl_apps_email USING btree (student_email)
- idx idx_lwl_apps_paid USING btree (is_paid) WHERE (is_paid = true)
- idx idx_lwl_apps_program USING btree (source_program)
- idx idx_lwl_apps_school USING btree (school_code)
- idx idx_lwl_apps_school_code USING btree (school_code)
- idx idx_lwl_apps_unmatched USING btree (id) WHERE ((school_code IS NULL) AND (raw_school_name IS NOT NULL))

### meeting_assignees
meeting_id uuid, user_id uuid, assigned_by text?, assigned_at timestamp with time zone=now()
- FK meeting_assignees_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE CASCADE
- FK meeting_assignees_user_id_fkey: FOREIGN KEY (user_id) REFERENCES leads.user_profiles(id) ON DELETE CASCADE
- PK meeting_assignees_pkey: PRIMARY KEY (meeting_id, user_id)
- idx idx_meeting_assignees_meeting USING btree (meeting_id)
- idx idx_meeting_assignees_user USING btree (user_id)

### meeting_sync_queue
id uuid=gen_random_uuid(), meeting_id uuid, run_at timestamp with time zone, attempts integer=0, max_attempts integer=3, last_error text?, done boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK meeting_sync_queue_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE CASCADE
- PK meeting_sync_queue_pkey: PRIMARY KEY (id)
- idx meeting_sync_queue_due_idx USING btree (run_at) WHERE (done = false)

### meetings
id uuid=gen_random_uuid(), meeting_date date, scheduled_time time without time zone, full_name text?, school_name text?, email text?, phone text?, city text?, state text?, country text?, designation text?, meeting_type text?='New Lead'::text, utm_source text?='direct'::text, query text?, meeting_link text?, zoom_meeting_id text?, recording_url text?, calendly_event_id text?, calendly_invitee_id text?, calendly_raw_payload jsonb?, assigned_to text?, number_of_prev_applications integer?=0, meeting_status text?='Scheduled'::text, actual_start timestamp with time zone?, actual_end timestamp with time zone?, duration_minutes integer?, ai_notes text?, meeting_notes text?, zoom_ai_summary text?, transcript_text text?, program_pitched text?, interest_level text?, lifecycle_stage text?='new_lead'::text, follow_up_date text?, followup_email_24h text?, followup_link text?, follow_up_meeting text?, followup1_status text?, followup1_content text?, followup1_response text?, followup2_status text?, followup2_content text?, followup2_response text?, followup3_status text?, followup3_content text?, followup3_response text?, conversion_status text?, conversion_date date?, conversion_notes text?, program_start_date date?, program_end_date date?, paid_student_count integer?, application_count integer?, re_engage_after date?, post_conversion_notes text?, personal_notes text?, linkedin_url text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), utm_medium text?='direct'::text, utm_campaign text?='direct'::text, invitee_timezone text?, next_step text?, key_quotes text?, ai_analyzed_at timestamp with time zone?, ai_confidence numeric?, ai_input_snapshot jsonb?, ai_status text?='idle'::text, ai_error text?, lead_score integer?, lead_temperature text?, objections text?, commitments text?, escalation_needed boolean=false, escalation_reason text?, transcript_url text?, start_time_utc timestamp with time zone?, end_time_utc timestamp with time zone?, followup1_subject text?, followup2_subject text?, followup3_subject text?, followup1_scheduled_for date?, followup2_scheduled_for date?, followup3_scheduled_for date?, followup1_generated_at timestamp with time zone?, followup2_generated_at timestamp with time zone?, followup3_generated_at timestamp with time zone?, last_inbound_at timestamp with time zone?, calendly_start_time_raw text?, calendly_end_time_raw text?, calendly_event_timezone text?, next_contact_date date?, last_agent_decision text?, no_show_count integer=0, nurture_step integer=0, last_nurture_content_type text?, last_nurture_sent_at timestamp with time zone?, attendance_decision_source text?, attendance_decision_evidence text?, attendance_confidence numeric?, attendance_review_required boolean=false, rescheduled_from_invitee_id text?, rescheduled_to_meeting_id uuid?, substitute_attendee_name text?, substitute_attendee_email text?, score_reasons text?, recommended_goal text?, recommended_program_codes text[]?, lead_segment text?, scored_at timestamp with time zone?, school_code text?, school_match_confidence double precision?, school_match_source text?, school_match_reviewed boolean=false, last_agent_action_at timestamp with time zone?, attributed_app_ids uuid[]='{}'::uuid[], attributed_paid_count integer=0, attributed_at timestamp with time zone?, webhook_dispatched_at timestamp with time zone?, webhook_dispatch_status text?, webhook_dispatch_score integer?, deal_id uuid?
- CK meetings_lead_score_range: CHECK (((lead_score IS NULL) OR ((lead_score >= 0) AND (lead_score <= 100))))
- FK meetings_deal_id_fkey: FOREIGN KEY (deal_id) REFERENCES leads.deals(id) ON DELETE SET NULL
- FK meetings_rescheduled_to_meeting_id_fkey: FOREIGN KEY (rescheduled_to_meeting_id) REFERENCES leads.meetings(id) ON DELETE SET NULL
- FK meetings_school_code_fkey: FOREIGN KEY (school_code) REFERENCES leads.school_registry(code) ON DELETE SET NULL
- PK meetings_pkey: PRIMARY KEY (id)
- UQ meetings_calendly_event_id_key: UNIQUE (calendly_event_id)
- idx idx_calendly_event USING btree (calendly_event_id)
- idx idx_meetings_assigned USING btree (assigned_to)
- idx idx_meetings_assigned_to USING btree (assigned_to) WHERE (assigned_to IS NOT NULL)
- idx idx_meetings_attributed_at USING btree (attributed_at)
- idx idx_meetings_completed_no_zoom USING btree (end_time_utc) WHERE ((meeting_status = 'Completed'::text) AND (zoom_meeting_id IS NULL) AND (end_time_utc IS NOT NULL))
- idx idx_meetings_date_status USING btree (meeting_date, meeting_status)
- idx idx_meetings_date_time USING btree (meeting_date, scheduled_time)
- idx idx_meetings_email USING btree (email)
- idx idx_meetings_high_score_pending USING btree (scored_at) WHERE ((webhook_dispatched_at IS NULL) AND (lead_score IS NOT NULL))
- idx idx_meetings_interest USING btree (interest_level)
- idx idx_meetings_last_agent_action_at USING btree (last_agent_action_at)
- idx idx_meetings_lead_segment USING btree (lead_segment)
- idx idx_meetings_lifecycle USING btree (lifecycle_stage)
- idx idx_meetings_lifecycle_stage USING btree (lifecycle_stage)
- idx idx_meetings_needs_match USING btree (id) WHERE ((school_code IS NULL) AND (school_name IS NOT NULL))
- idx idx_meetings_next_contact USING btree (next_contact_date)
- idx idx_meetings_rescheduled_to USING btree (rescheduled_to_meeting_id) WHERE (rescheduled_to_meeting_id IS NOT NULL)
- idx idx_meetings_school USING btree (school_name)
- idx idx_meetings_school_code USING btree (school_code)
- idx idx_meetings_school_unmatched USING btree (school_name) WHERE ((school_code IS NULL) AND (school_name IS NOT NULL) AND (school_name <> ''::text))
- idx idx_meetings_status USING btree (meeting_status)
- idx idx_meetings_status_date USING btree (meeting_status, meeting_date)
- idx idx_meetings_zoom_meeting_id USING btree (zoom_meeting_id) WHERE (zoom_meeting_id IS NOT NULL)
- idx meetings_date_time_idx USING btree (meeting_date, scheduled_time)
- idx meetings_deal_id_idx USING btree (deal_id)

### notifications
id uuid=gen_random_uuid(), user_id uuid, type text, title text, body text?, meeting_id uuid?, read boolean=false, created_at timestamp with time zone=now()
- FK notifications_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE CASCADE
- FK notifications_user_id_fkey: FOREIGN KEY (user_id) REFERENCES leads.user_profiles(id) ON DELETE CASCADE
- PK notifications_pkey: PRIMARY KEY (id)
- idx idx_notifications_user_unread USING btree (user_id, read, created_at DESC)

### program_cohorts
id uuid=gen_random_uuid(), ext_table_name text, cohort_slug text, program_name text?, cohort_label text?, display_name text?, needs_review boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK program_cohorts_pkey: PRIMARY KEY (id)
- UQ program_cohorts_ext_table_name_cohort_slug_key: UNIQUE (ext_table_name, cohort_slug)

### scheduled_emails
id uuid=gen_random_uuid(), meeting_id uuid?, email_type text, send_after timestamp with time zone, status text='scheduled'::text, cancelled_reason text?, email_send_id uuid?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK scheduled_emails_email_send_id_fkey: FOREIGN KEY (email_send_id) REFERENCES leads.email_sends(id) ON DELETE SET NULL
- FK scheduled_emails_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE CASCADE
- PK scheduled_emails_pkey: PRIMARY KEY (id)
- idx scheduled_emails_meeting_type_active UNIQUE USING btree (meeting_id, email_type) WHERE (status = 'scheduled'::text)
- idx scheduled_emails_status_send_after USING btree (status, send_after)

### school_aliases
id uuid=gen_random_uuid(), raw_text text, school_code text?, match_confidence double precision=1.0, match_source text='seed'::text, times_seen integer=1, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK school_aliases_school_code_fkey: FOREIGN KEY (school_code) REFERENCES leads.school_registry(code) ON DELETE SET NULL
- PK school_aliases_pkey: PRIMARY KEY (id)
- idx school_aliases_raw_lower UNIQUE USING btree (lower(TRIM(BOTH FROM raw_text)))

### school_merge_log
id uuid=gen_random_uuid(), source_code text, target_code text, reason text?, confidence numeric?, rows_repointed jsonb?, merged_at timestamp with time zone=now()
- PK school_merge_log_pkey: PRIMARY KEY (id)

### school_registry
id uuid=gen_random_uuid(), code text, canonical_name text, country text?, city text?, region text?, is_active boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), auto_created boolean=false, auto_created_at timestamp with time zone?
- PK school_registry_pkey: PRIMARY KEY (id)
- UQ school_registry_code_key: UNIQUE (code)
- idx idx_sr_code USING btree (code)
- idx idx_sr_name USING btree (canonical_name)

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
- FK sop_pricing_floors_set_by_fkey: FOREIGN KEY (set_by) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- PK sop_pricing_floors_pkey: PRIMARY KEY (programme, quarter)

### sop_targets
quarter text, stream text, target_paid_minor bigint=0, currency text='USD'::text, revenue_per_meeting_minor bigint?, meetings_needed integer?
- CK sop_targets_stream_check: CHECK ((stream = ANY (ARRAY['in_person'::text, 'camps'::text, 'online'::text, 'hosting'::text])))
- PK sop_targets_pkey: PRIMARY KEY (quarter, stream)

### sync_state
id uuid=gen_random_uuid(), source_key text, last_synced_at timestamp with time zone='2020-01-01 00:00:00+00'::timestamp with time zone, last_row_count integer?=0, new_rows_last_run integer?=0, payment_updates_last_run integer?=0, last_run_at timestamp with time zone?, last_run_status text='never_run'::text, last_error text?, updated_at timestamp with time zone=now(), ext_table_name text?, dry_run_only boolean=true, ext_source text='ext1'::text
- PK sync_state_pkey: PRIMARY KEY (id)
- UQ sync_state_source_key_key: UNIQUE (source_key)

### tasks
id uuid=gen_random_uuid(), type leads.task_type='other'::leads.task_type, title text, description text?, priority leads.task_priority='med'::leads.task_priority, status leads.task_status='open'::leads.task_status, meeting_id uuid?, email_send_id uuid?, assigned_to uuid?, created_by_agent boolean=false, due_at timestamp with time zone?, completed_at timestamp with time zone?, completed_by uuid?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), snooze_until timestamp with time zone?, due_date date?, created_by text='system'::text, attachment_url text?, attachment_name text?, needs_info jsonb?, revision_request text?, revision_count integer=0, escalation_template_slug text?, deal_id uuid?, dedupe_key text? /*Stops the SOP sweeper raising the same task twice for the same deal and period.*/, sop_kind text? /*Which SOP rule raised this task. Keeps the task_type enum untouched.*/
- FK tasks_assigned_to_fkey: FOREIGN KEY (assigned_to) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- FK tasks_completed_by_fkey: FOREIGN KEY (completed_by) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- FK tasks_deal_id_fkey: FOREIGN KEY (deal_id) REFERENCES leads.deals(id) ON DELETE CASCADE
- FK tasks_email_send_id_fkey: FOREIGN KEY (email_send_id) REFERENCES leads.email_sends(id) ON DELETE SET NULL
- FK tasks_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE CASCADE
- PK tasks_pkey: PRIMARY KEY (id)
- idx idx_tasks_assigned USING btree (assigned_to) WHERE (status = 'open'::leads.task_status)
- idx idx_tasks_assigned_to_status USING btree (assigned_to, status) WHERE (assigned_to IS NOT NULL)
- idx idx_tasks_created_at_desc USING btree (created_at DESC)
- idx idx_tasks_due_date USING btree (due_date)
- idx idx_tasks_email USING btree (email_send_id)
- idx idx_tasks_meeting USING btree (meeting_id)
- idx idx_tasks_snooze USING btree (snooze_until) WHERE (status = 'snoozed'::leads.task_status)
- idx idx_tasks_status USING btree (status)
- idx tasks_one_active_escalation_per_meeting UNIQUE USING btree (meeting_id) WHERE ((type = 'escalation'::leads.task_type) AND (status = ANY (ARRAY['pending'::leads.task_status, 'open'::leads.task_status, 'needs_info'::leads.task_status, 'revision_requested'::leads.task_status, 'revising'::leads.task_status, 'snoozed'::leads.task_status])))
- idx tasks_one_active_per_email_send UNIQUE USING btree (email_send_id) WHERE ((email_send_id IS NOT NULL) AND (status = ANY (ARRAY['pending'::leads.task_status, 'open'::leads.task_status, 'needs_info'::leads.task_status, 'revision_requested'::leads.task_status, 'revising'::leads.task_status, 'snoozed'::leads.task_status])))
- idx tasks_deal_id_idx USING btree (deal_id)
- idx tasks_dedupe_key_uniq UNIQUE USING btree (dedupe_key)

### user_profiles
id uuid, full_name text=''::text, email text?, title text?, phone text?, avatar_color text?, is_active boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), role leads.app_role='member'::leads.app_role, calendly_link text?, zoom_account_id text?, invited_by uuid?, invited_at timestamp with time zone?, last_login_at timestamp with time zone?
- FK user_profiles_id_fkey: FOREIGN KEY (id) REFERENCES auth.users(id) ON DELETE CASCADE
- FK user_profiles_invited_by_fkey: FOREIGN KEY (invited_by) REFERENCES leads.user_profiles(id) ON DELETE SET NULL
- PK user_profiles_pkey: PRIMARY KEY (id)

### webhook_logs
id uuid=gen_random_uuid(), source text='calendly'::text, event_type text?, calendly_event_id text?, calendly_invitee_id text?, payload jsonb, status text='received'::text, error_message text?, retry_count integer=0, meeting_id uuid?, processed_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK webhook_logs_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES leads.meetings(id) ON DELETE SET NULL
- PK webhook_logs_pkey: PRIMARY KEY (id)
- idx idx_webhook_logs_created USING btree (created_at DESC)
- idx idx_webhook_logs_invitee USING btree (calendly_invitee_id)
- idx idx_webhook_logs_status USING btree (status)

### weekly_review_runs
id uuid=gen_random_uuid(), ran_at timestamp with time zone=now(), week_start date, week_end date, summary jsonb='{}'::jsonb, fixes_applied integer=0, issues_found integer=0, unresolved jsonb='[]'::jsonb, duration_ms integer?
- PK weekly_review_runs_pkey: PRIMARY KEY (id)
- idx idx_weekly_review_runs_ran_at USING btree (ran_at DESC)

