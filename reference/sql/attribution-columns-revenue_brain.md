# Brain project: columns, constraints and indexes for schema `revenue_brain`

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only.
The revenue_brain schema backs the revenue-brain-orchestrator(-v2) edge functions: outbound queue, sequences, claims registry, Instantly webhook inbox, Yale pilot tables, test/certification case tables (51 tables, 0 RLS policies).
Legend: one line per table listing `column type` where a trailing `?` means NULLABLE (no `?` = NOT NULL), `=expr` is the column default (generated columns show their expression), and `/*...*/` is the column comment. Below each column line: `PK`/`FK`/`UQ`/`CK`/`EX` constraints (pg_get_constraintdef) and `idx` = indexes not backing a constraint (shown as `name [UNIQUE] USING method (cols) [WHERE ...]`).

## revenue_brain

### account_program_fit
school_id text, school_name text, programme_id uuid?, programme_code text, fit_score integer?, fit_class text?, relationship_class text?, timing_score integer?, revenue_priority text?, recommended_contact_id text?, recommended_contact_name text?, recommended_contact_role text?, intended_contact_email text?, personalization_class text?, reasoning text?, source_ref text?, is_test_fixture boolean=false, calculated_at timestamp with time zone=now()
- CK account_program_fit_fit_score_check: CHECK (((fit_score >= 0) AND (fit_score <= 100)))
- CK account_program_fit_timing_score_check: CHECK (((timing_score >= 0) AND (timing_score <= 100)))
- PK account_program_fit_pkey: PRIMARY KEY (school_id, programme_code)

### approval_audit
id uuid=gen_random_uuid(), queue_id uuid?, decision text, approved_by text?, approved_at timestamp with time zone?, notes text?, is_test boolean=false, created_at timestamp with time zone=now()
- FK approval_audit_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id) ON DELETE CASCADE
- PK approval_audit_pkey: PRIMARY KEY (id)

### autonomous_test_cases
id uuid=gen_random_uuid(), run_id uuid, case_key text, school_id text, school_name text, contact_id text, contact_name text, delivery_email text, delivery_first_name text?, scenario text, account_brief jsonb='{}'::jsonb, ai_decision jsonb='{}'::jsonb, brain_decision_id uuid?, queue_id uuid?, provider_lead_id text?, status text='READY_FOR_AI'::text, expected_outcome text?, error_text text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK autonomous_test_cases_status_check: CHECK ((status = ANY (ARRAY['READY_FOR_AI'::text, 'STAGED'::text, 'GATED'::text, 'ENROLLED'::text, 'ACTIVE'::text, 'SIMULATED_REPLY'::text, 'COMPLETED'::text, 'FAILED'::text])))
- FK autonomous_test_cases_brain_decision_id_fkey: FOREIGN KEY (brain_decision_id) REFERENCES revenue_brain.brain_decisions(id) ON DELETE SET NULL
- FK autonomous_test_cases_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id) ON DELETE SET NULL
- FK autonomous_test_cases_run_id_fkey: FOREIGN KEY (run_id) REFERENCES revenue_brain.autonomous_test_runs(id) ON DELETE CASCADE
- PK autonomous_test_cases_pkey: PRIMARY KEY (id)
- UQ autonomous_test_cases_run_id_case_key_key: UNIQUE (run_id, case_key)

### autonomous_test_runs
id uuid=gen_random_uuid(), run_code text, status text='CREATED'::text, provider text='INSTANTLY'::text, provider_campaign_id text, expected_case_count integer=2, notes jsonb='{}'::jsonb, started_at timestamp with time zone?, completed_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK autonomous_test_runs_status_check: CHECK ((status = ANY (ARRAY['CREATED'::text, 'STAGED'::text, 'DISPATCHING'::text, 'ACTIVE'::text, 'COMPLETED'::text, 'FAILED'::text, 'CANCELLED'::text])))
- PK autonomous_test_runs_pkey: PRIMARY KEY (id)
- UQ autonomous_test_runs_run_code_key: UNIQUE (run_code)

### brain_decisions
id uuid=gen_random_uuid(), school_id text, school_name text, contact_id text, contact_name text, programme_code text, campaign_code text?, action text, personalization_class text?, priority_score integer?, confidence numeric(5,4)?, selected_sender_email text?, custom_line_1 text?, custom_line_2 text?, decision_reasoning jsonb='{}'::jsonb, decision_status text='DRAFT'::text, source_snapshot jsonb='{}'::jsonb, is_test boolean=false, created_at timestamp with time zone=now()
- CK brain_decisions_confidence_check: CHECK (((confidence >= (0)::numeric) AND (confidence <= (1)::numeric)))
- CK brain_decisions_priority_score_check: CHECK (((priority_score >= 0) AND (priority_score <= 100)))
- PK brain_decisions_pkey: PRIMARY KEY (id)

### campaign_routes
id uuid=gen_random_uuid(), programme_code text, route_key text, provider text='INSTANTLY'::text, provider_campaign_id text, campaign_name text?, timezone_key text, schedule_timezone text?, schedule_window text?, test_only boolean=false, enabled boolean=false, notes text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK campaign_routes_pkey: PRIMARY KEY (id)
- UQ campaign_routes_programme_code_route_key_test_only_key: UNIQUE (programme_code, route_key, test_only)

### claims_registry
id uuid=gen_random_uuid(), programme_code text, claim_key text, claim_type text, claim_value text, status text, evidence_source text?, source_url text?, approved_by text?, approved_at timestamp with time zone?, expires_at timestamp with time zone?, notes text?, is_test_fixture boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK claims_registry_pkey: PRIMARY KEY (id)
- UQ claims_registry_programme_code_claim_key_key: UNIQUE (programme_code, claim_key)

### conversion_events
id uuid=gen_random_uuid(), event_key text, school_id text, school_name text?, contact_id text, programme_code text, queue_id uuid?, event_type text, application_ref text?, payment_ref text?, currency text?, amount_minor bigint?, event_payload jsonb='{}'::jsonb, occurred_at timestamp with time zone=now(), is_test boolean=false, created_at timestamp with time zone=now()
- FK conversion_events_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id)
- PK conversion_events_pkey: PRIMARY KEY (id)
- UQ conversion_events_event_key_key: UNIQUE (event_key)

### conversion_test_cases
id uuid=gen_random_uuid(), run_id uuid, case_key text, school_id text, school_name text, contact_id text, contact_name text, delivery_email text, queue_id uuid?, provider_campaign_id text, provider_lead_id text, application_ref text?, payment_ref text?, expected_net_minor bigint=0, expected_currency text='USD'::text, current_stage text='FIXTURE_READY'::text, status text='READY'::text, result jsonb='{}'::jsonb, error_text text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK conversion_test_cases_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id)
- FK conversion_test_cases_run_id_fkey: FOREIGN KEY (run_id) REFERENCES revenue_brain.conversion_test_runs(id) ON DELETE CASCADE
- PK conversion_test_cases_pkey: PRIMARY KEY (id)
- UQ conversion_test_cases_run_id_case_key_key: UNIQUE (run_id, case_key)

### conversion_test_runs
id uuid=gen_random_uuid(), run_code text, status text='CREATED'::text, expected_case_count integer=2, started_at timestamp with time zone=now(), completed_at timestamp with time zone?, notes jsonb='{}'::jsonb, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK conversion_test_runs_pkey: PRIMARY KEY (id)
- UQ conversion_test_runs_run_code_key: UNIQUE (run_code)

### eb_conversations
id uuid=gen_random_uuid(), lead_id uuid, instantly_thread_id text?, campaign_id text?, sender_email text?, programme_code text?, stage text='contacted'::text, last_reply_class text?, ai_summary text?, sentiment text?, next_action text?, next_action_due timestamp with time zone?, is_internal boolean=false, last_inbound_at timestamp with time zone?, last_outbound_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK eb_conversations_last_reply_class_check: CHECK ((last_reply_class = ANY (ARRAY['interested'::text, 'meeting_request'::text, 'question'::text, 'not_now'::text, 'not_interested'::text, 'unsubscribe'::text, 'out_of_office'::text, 'wrong_person'::text, 'bounce'::text, 'internal'::text, 'other'::text])))
- CK eb_conversations_sentiment_check: CHECK ((sentiment = ANY (ARRAY['positive'::text, 'neutral'::text, 'negative'::text])))
- CK eb_conversations_stage_check: CHECK ((stage = ANY (ARRAY['new'::text, 'contacted'::text, 'replied'::text, 'interested'::text, 'meeting_booked'::text, 'won'::text, 'lost'::text, 'not_interested'::text, 'do_not_contact'::text])))
- FK eb_conversations_lead_id_fkey: FOREIGN KEY (lead_id) REFERENCES revenue_brain.eb_leads(id) ON DELETE CASCADE
- PK eb_conversations_pkey: PRIMARY KEY (id)
- UQ eb_conversations_instantly_thread_id_key: UNIQUE (instantly_thread_id)
- idx eb_conv_lead_idx USING btree (lead_id)
- idx eb_conv_due_idx USING btree (next_action_due) WHERE (next_action_due IS NOT NULL)

### eb_do_not_contact
id uuid=gen_random_uuid(), email text?, domain text?, reason text, source text?, instantly_blocklisted_at timestamp with time zone?, created_at timestamp with time zone=now()
- CK eb_do_not_contact_check: CHECK (((email IS NOT NULL) OR (domain IS NOT NULL)))
- PK eb_do_not_contact_pkey: PRIMARY KEY (id)
- idx eb_dnc_email_uq UNIQUE USING btree (lower(email)) WHERE (email IS NOT NULL)
- idx eb_dnc_domain_uq UNIQUE USING btree (lower(domain)) WHERE (domain IS NOT NULL)

### eb_events
id uuid=gen_random_uuid(), lead_id uuid?, conversation_id uuid?, campaign_id text?, type text, payload jsonb='{}'::jsonb, dedupe_key text?, at timestamp with time zone=now()
- CK eb_events_type_check: CHECK ((type = ANY (ARRAY['sent'::text, 'opened'::text, 'replied'::text, 'bounced'::text, 'unsubscribed'::text, 'meeting_booked'::text, 'auto_reply'::text, 'stage_changed'::text, 'draft_created'::text, 'draft_sent'::text, 'blocklisted'::text, 'paused_account'::text])))
- FK eb_events_conversation_id_fkey: FOREIGN KEY (conversation_id) REFERENCES revenue_brain.eb_conversations(id) ON DELETE SET NULL
- FK eb_events_lead_id_fkey: FOREIGN KEY (lead_id) REFERENCES revenue_brain.eb_leads(id) ON DELETE SET NULL
- PK eb_events_pkey: PRIMARY KEY (id)
- UQ eb_events_dedupe_key_key: UNIQUE (dedupe_key)
- idx eb_events_type_at_idx USING btree (type, at)

### eb_leads
id uuid=gen_random_uuid(), email text, first_name text?, last_name text?, org text?, role text?, country text?, region text?, timezone text?, source text?, source_ref text?, programme_code text?, custom_line text?, instantly_lead_id text?, status text='new'::text, interest_status text?, owner text?='Jatin Sharma'::text, verification text?, last_contacted_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK eb_leads_status_check: CHECK ((status = ANY (ARRAY['new'::text, 'contacted'::text, 'replied'::text, 'interested'::text, 'meeting_booked'::text, 'won'::text, 'lost'::text, 'not_interested'::text, 'do_not_contact'::text])))
- PK eb_leads_pkey: PRIMARY KEY (id)
- idx eb_leads_email_uq UNIQUE USING btree (lower(email))
- idx eb_leads_status_idx USING btree (status)

### eb_messages
id uuid=gen_random_uuid(), conversation_id uuid, direction text, instantly_email_id text?, message_id text?, from_email text?, to_email text?, cc_email text?, subject text?, body_text text?, step text?, sent_at timestamp with time zone?, created_at timestamp with time zone=now()
- CK eb_messages_direction_check: CHECK ((direction = ANY (ARRAY['in'::text, 'out'::text])))
- FK eb_messages_conversation_id_fkey: FOREIGN KEY (conversation_id) REFERENCES revenue_brain.eb_conversations(id) ON DELETE CASCADE
- PK eb_messages_pkey: PRIMARY KEY (id)
- UQ eb_messages_instantly_email_id_key: UNIQUE (instantly_email_id)
- idx eb_msg_conv_idx USING btree (conversation_id, sent_at)

### eb_notifications
id uuid=gen_random_uuid(), kind text, ref_type text?, ref_id text, recipient text, channel text='email'::text, body text?, sent_at timestamp with time zone=now()
- PK eb_notifications_pkey: PRIMARY KEY (id)
- UQ eb_notifications_kind_ref_id_recipient_key: UNIQUE (kind, ref_id, recipient)

### eb_reply_drafts
id uuid=gen_random_uuid(), conversation_id uuid, reply_to_instantly_email_id text, reply_class text?, subject text?, body text, facts_used jsonb='[]'::jsonb, open_questions jsonb='[]'::jsonb, status text='pending_approval'::text, edit_notes text?, approved_by text?, approved_at timestamp with time zone?, sent_instantly_email_id text?, sent_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK eb_reply_drafts_status_check: CHECK ((status = ANY (ARRAY['pending_approval'::text, 'approved'::text, 'edited'::text, 'sent'::text, 'rejected'::text, 'superseded'::text])))
- FK eb_reply_drafts_conversation_id_fkey: FOREIGN KEY (conversation_id) REFERENCES revenue_brain.eb_conversations(id) ON DELETE CASCADE
- PK eb_reply_drafts_pkey: PRIMARY KEY (id)
- idx eb_draft_one_pending_uq UNIQUE USING btree (reply_to_instantly_email_id) WHERE (status = 'pending_approval'::text)

### eb_sync_state
key text, cursor text?, last_synced_at timestamp with time zone?, last_run_at timestamp with time zone?, last_status text?, items_processed integer?=0, note text?
- PK eb_sync_state_pkey: PRIMARY KEY (key)

### import_tokens
token text, purpose text, expires_at timestamp with time zone
- PK import_tokens_pkey: PRIMARY KEY (token)

### orchestration_runs
id uuid=gen_random_uuid(), run_type text, queue_id uuid?, school_id text?, contact_id text?, programme_code text?, provider text?, provider_campaign_id text?, provider_lead_id text?, status text='STARTED'::text, gate_result jsonb='{}'::jsonb, request_payload jsonb='{}'::jsonb, response_payload jsonb='{}'::jsonb, error_text text?, started_at timestamp with time zone=now(), finished_at timestamp with time zone?, is_test boolean=true
- CK orchestration_runs_run_type_check: CHECK ((run_type = ANY (ARRAY['DISPATCH'::text, 'WEBHOOK'::text, 'RECONCILE'::text])))
- FK orchestration_runs_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id) ON DELETE SET NULL
- PK orchestration_runs_pkey: PRIMARY KEY (id)
- idx orchestration_runs_queue_idx USING btree (queue_id, started_at DESC)

### outbound_queue
id uuid=gen_random_uuid(), school_id text, school_name text, contact_id text, intended_contact_name text, intended_contact_role text?, intended_contact_email text?, delivery_email text, delivery_first_name text?, programme_code text, campaign_code text, personalization_class text?, custom_line_1 text?, custom_line_2 text?, subject text?, body_text text?, body_html text?, sender_email text?, scheduled_for timestamp with time zone?, queue_status text='DRAFT'::text, approval_status text='PENDING'::text, qa_status text='PENDING'::text, test_mode boolean=true, provider text?, provider_campaign_id text?, provider_lead_id text?, provider_message_id text?, brain_decision_id uuid?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK outbound_queue_brain_decision_id_fkey: FOREIGN KEY (brain_decision_id) REFERENCES revenue_brain.brain_decisions(id)
- PK outbound_queue_pkey: PRIMARY KEY (id)

### outreach_events
id uuid=gen_random_uuid(), queue_id uuid?, school_id text, contact_id text, programme_code text, provider text, event_type text, provider_campaign_id text?, provider_message_id text?, event_payload jsonb='{}'::jsonb, occurred_at timestamp with time zone=now(), is_test boolean=false, created_at timestamp with time zone=now()
- FK outreach_events_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id)
- PK outreach_events_pkey: PRIMARY KEY (id)
- idx idx_rb_outreach_events_school_time USING btree (school_id, occurred_at DESC)
- idx idx_rb_outreach_events_contact_time USING btree (contact_id, occurred_at DESC)

### pilot_copy_review_cases
id uuid=gen_random_uuid(), run_id uuid, preflight_case_id uuid, school_name text, contact_name text, contact_role text?, contact_email text, normalized_first_name text, lwl_history jsonb='{}'::jsonb, recommendation text, copy_blockers jsonb='[]'::jsonb, rendered_steps jsonb='[]'::jsonb, status text='REVIEW_READY'::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK pilot_copy_review_cases_preflight_case_id_fkey: FOREIGN KEY (preflight_case_id) REFERENCES revenue_brain.pilot_preflight_cases(id)
- FK pilot_copy_review_cases_run_id_fkey: FOREIGN KEY (run_id) REFERENCES revenue_brain.pilot_copy_review_runs(id) ON DELETE CASCADE
- PK pilot_copy_review_cases_pkey: PRIMARY KEY (id)
- UQ pilot_copy_review_cases_run_id_preflight_case_id_key: UNIQUE (run_id, preflight_case_id)

### pilot_copy_review_runs
id uuid=gen_random_uuid(), run_code text, preflight_run_id uuid, sequence_id uuid, sequence_code text, status text='CREATED'::text, ready_count integer=0, held_count integer=0, no_send boolean=true, notes jsonb='{}'::jsonb, started_at timestamp with time zone=now(), completed_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK pilot_copy_review_runs_preflight_run_id_fkey: FOREIGN KEY (preflight_run_id) REFERENCES revenue_brain.pilot_preflight_runs(id)
- FK pilot_copy_review_runs_sequence_id_fkey: FOREIGN KEY (sequence_id) REFERENCES revenue_brain.sequence_templates(id)
- PK pilot_copy_review_runs_pkey: PRIMARY KEY (id)
- UQ pilot_copy_review_runs_run_code_key: UNIQUE (run_code)

### pilot_preflight_cases
id uuid=gen_random_uuid(), run_id uuid, shadow_case_id uuid?, school_id text, school_name text, contact_id text, contact_name text, contact_role text?, contact_email text, sender_email text, programme_code text, route_key text, brain_decision_id uuid?, queue_id uuid?, lwl_history jsonb='{}'::jsonb, send_gate jsonb='{}'::jsonb, preflight_checks jsonb='{}'::jsonb, launch_blockers jsonb='[]'::jsonb, readiness text='PENDING'::text, approval_decision text?, approved_by text?, approved_at timestamp with time zone?, preview_subject text?, preview_body text?, status text='CREATED'::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK pilot_preflight_cases_brain_decision_id_fkey: FOREIGN KEY (brain_decision_id) REFERENCES revenue_brain.brain_decisions(id)
- FK pilot_preflight_cases_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id)
- FK pilot_preflight_cases_run_id_fkey: FOREIGN KEY (run_id) REFERENCES revenue_brain.pilot_preflight_runs(id) ON DELETE CASCADE
- FK pilot_preflight_cases_shadow_case_id_fkey: FOREIGN KEY (shadow_case_id) REFERENCES revenue_brain.shadow_cases(id)
- PK pilot_preflight_cases_pkey: PRIMARY KEY (id)
- UQ pilot_preflight_cases_run_id_school_id_contact_id_programme_key: UNIQUE (run_id, school_id, contact_id, programme_code)

### pilot_preflight_runs
id uuid=gen_random_uuid(), run_code text, status text='CREATED'::text, source_shadow_run_id uuid?, programme_code text, route_key text, provider_campaign_id text?, provider_snapshot jsonb='{}'::jsonb, candidate_count integer=0, ready_count integer=0, held_count integer=0, no_send boolean=true, notes jsonb='{}'::jsonb, started_at timestamp with time zone=now(), completed_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK pilot_preflight_runs_source_shadow_run_id_fkey: FOREIGN KEY (source_shadow_run_id) REFERENCES revenue_brain.shadow_runs(id)
- PK pilot_preflight_runs_pkey: PRIMARY KEY (id)
- UQ pilot_preflight_runs_run_code_key: UNIQUE (run_code)

### programme_batch_leads
id bigint=nextval('revenue_brain.programme_batch_leads_id_seq'::regclass), batch_id uuid, rank integer, contact_id uuid, school_id uuid, school_name text?, country text?, geo_region text?, contact_name text?, greeting text, sop_role text?, designation text?, email text, email_domain text, zb_status text?, zb_checked_at timestamp with time zone?, role_evidence text?, role_seen_at timestamp with time zone?, candidate_status text, decision text='PROPOSED'::text, skip_reason text?, queue_id uuid?
- CK programme_batch_leads_decision_check: CHECK ((decision = ANY (ARRAY['PROPOSED'::text, 'APPROVED'::text, 'REJECTED'::text, 'STAGED'::text, 'SKIPPED'::text])))
- FK programme_batch_leads_batch_id_fkey: FOREIGN KEY (batch_id) REFERENCES revenue_brain.programme_batches(id) ON DELETE CASCADE
- PK programme_batch_leads_pkey: PRIMARY KEY (id)
- UQ programme_batch_leads_batch_id_email_key: UNIQUE (batch_id, email)

### programme_batches
id uuid=gen_random_uuid(), programme_code text, status text='PROPOSED'::text, size_requested integer, region text?, include_needs_zerobounce boolean=true, created_by text, created_at timestamp with time zone=now(), approved_by text?, approved_at timestamp with time zone?, staged_by text?, staged_at timestamp with time zone?, notes text?
- CK programme_batches_status_check: CHECK ((status = ANY (ARRAY['PROPOSED'::text, 'APPROVED'::text, 'STAGED'::text, 'CANCELLED'::text])))
- FK programme_batches_programme_code_fkey: FOREIGN KEY (programme_code) REFERENCES revenue_brain.programme_profiles(programme_code)
- PK programme_batches_pkey: PRIMARY KEY (id)

### programme_profiles
programme_code text, public_programme_code text?, display_name text, brand text?, category text?, format text?, grades text?, age_min integer?, age_max integer?, target_regions text[]='{}'::text[], target_roles text[]='{}'::text[], summary text?, eligibility text?, apply_url text?, details_url text?, booking_url text?, application_code_patterns text[]='{}'::text[], past_participant_rule text='WELCOME_BACK'::text, outreach_enabled boolean=false, priority integer=100, programme_start date?, application_deadline date?, season_end date?, sender_pool text[]='{}'::text[], daily_send_cap integer=25, batch_size_default integer=100, zerobounce_budget integer=0, owner text?, copy_approver text?, notes text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), grade_bands text[]='{}'::text[], max_per_domain integer=3
- CK programme_profiles_batch_size_default_check: CHECK (((batch_size_default >= 1) AND (batch_size_default <= 1000)))
- CK programme_profiles_daily_send_cap_check: CHECK (((daily_send_cap >= 0) AND (daily_send_cap <= 2000)))
- CK programme_profiles_past_participant_rule_check: CHECK ((past_participant_rule = ANY (ARRAY['EXCLUDE'::text, 'WELCOME_BACK'::text, 'INCLUDE'::text])))
- PK programme_profiles_pkey: PRIMARY KEY (programme_code)

### reply_handoffs
id uuid=gen_random_uuid(), webhook_inbox_id uuid, queue_id uuid?, school_id text, school_name text?, contact_id text?, contact_name text?, contact_email text?, programme_code text?, provider text='INSTANTLY'::text, provider_campaign_id text?, provider_message_id text?, reply_subject text?, reply_text text?, reply_snippet text?, unibox_url text?, reply_class text, classification_confidence numeric?, classification_source text='RULES_V1'::text, classification_reason text?, priority text='MEDIUM'::text, owner text?, next_action text, due_at timestamp with time zone?, handoff_status text='NEW'::text, human_override boolean=false, human_classification text?, human_notes text?, resolution text?, resolved_by text?, resolved_at timestamp with time zone?, is_test boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), ai_attempted_at timestamp with time zone?, ai_model text?, ai_response jsonb='{}'::jsonb, notification_status text='PENDING'::text, notification_attempts integer=0, notification_channel text?, notification_recipients text?, notified_at timestamp with time zone?, notification_error text?, ai_error text?
- CK reply_handoffs_classification_confidence_check: CHECK (((classification_confidence >= (0)::numeric) AND (classification_confidence <= (1)::numeric)))
- CK reply_handoffs_handoff_status_check: CHECK ((handoff_status = ANY (ARRAY['NEW'::text, 'IN_PROGRESS'::text, 'RESOLVED'::text, 'AUTO_RESOLVED'::text])))
- CK reply_handoffs_notification_status_check: CHECK ((notification_status = ANY (ARRAY['PENDING'::text, 'PROCESSING'::text, 'SENT'::text, 'FAILED'::text])))
- CK reply_handoffs_priority_check: CHECK ((priority = ANY (ARRAY['CRITICAL'::text, 'HIGH'::text, 'MEDIUM'::text, 'LOW'::text])))
- CK reply_handoffs_reply_class_check: CHECK ((reply_class = ANY (ARRAY['POSITIVE'::text, 'QUESTION'::text, 'MEETING_REQUEST'::text, 'NOT_NOW'::text, 'NOT_INTERESTED'::text, 'WRONG_PERSON'::text, 'OOO'::text, 'UNSUBSCRIBE'::text, 'OTHER_REVIEW'::text])))
- FK reply_handoffs_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id) ON DELETE SET NULL
- FK reply_handoffs_webhook_inbox_id_fkey: FOREIGN KEY (webhook_inbox_id) REFERENCES revenue_brain.webhook_inbox(id) ON DELETE CASCADE
- PK reply_handoffs_pkey: PRIMARY KEY (id)
- UQ reply_handoffs_webhook_inbox_id_key: UNIQUE (webhook_inbox_id)
- idx reply_handoffs_open_idx USING btree (handoff_status, priority, due_at)
- idx reply_handoffs_school_idx USING btree (school_id, created_at DESC)

### revenue_attribution
id uuid=gen_random_uuid(), school_id text, school_name text?, contact_id text, programme_code text, queue_id uuid?, application_ref text?, payment_ref text, currency text, gross_minor bigint=0, refunded_minor bigint=0, net_minor bigint=0, attribution_method text, confidence numeric=1, source_event_id uuid?, is_test boolean=false, attributed_at timestamp with time zone=now(), created_at timestamp with time zone=now()
- FK revenue_attribution_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id)
- FK revenue_attribution_source_event_id_fkey: FOREIGN KEY (source_event_id) REFERENCES revenue_brain.conversion_events(id)
- PK revenue_attribution_pkey: PRIMARY KEY (id)
- UQ revenue_attribution_payment_ref_key: UNIQUE (payment_ref)

### runtime_controls
control_key text, bool_value boolean?, json_value jsonb='{}'::jsonb, description text?, updated_by text?, updated_at timestamp with time zone=now()
- PK runtime_controls_pkey: PRIMARY KEY (control_key)

### safety_test_cases
id uuid=gen_random_uuid(), run_id uuid, case_key text, school_id text, school_name text, contact_id text, delivery_email text, queue_id uuid?, provider_campaign_id text, provider_lead_id text, expected_result jsonb='{}'::jsonb, result jsonb='{}'::jsonb, status text='READY'::text, error_text text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK safety_test_cases_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id)
- FK safety_test_cases_run_id_fkey: FOREIGN KEY (run_id) REFERENCES revenue_brain.safety_test_runs(id) ON DELETE CASCADE
- PK safety_test_cases_pkey: PRIMARY KEY (id)
- UQ safety_test_cases_run_id_case_key_key: UNIQUE (run_id, case_key)

### safety_test_runs
id uuid=gen_random_uuid(), run_code text, status text='CREATED'::text, expected_case_count integer=4, started_at timestamp with time zone=now(), completed_at timestamp with time zone?, notes jsonb='{}'::jsonb, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK safety_test_runs_pkey: PRIMARY KEY (id)
- UQ safety_test_runs_run_code_key: UNIQUE (run_code)

### school_program_state
school_id text, school_name text, active_programme_code text?, active_campaign_code text?, relationship_state text?, active_human_conversation boolean=false, school_lock_state text='CLEAR'::text, lock_reason text?, last_proactive_touch_at timestamp with time zone?, touches_30d integer=0, next_eligible_at timestamp with time zone?, relationship_owner text?, source_ref text?, is_test_fixture boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK school_program_state_touches_30d_check: CHECK ((touches_30d >= 0))
- PK school_program_state_pkey: PRIMARY KEY (school_id)

### sender_accounts
sender_email text, provider text, provider_account_id text?, display_name text?, domain text?, status text='UNKNOWN'::text, daily_limit integer?, mx_ok boolean?, spf_ok boolean?, dkim_ok boolean?, dmarc_ok boolean?, warmup_status text?, health_checked_at timestamp with time zone?, allowed_for_test boolean=false, notes text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK sender_accounts_pkey: PRIMARY KEY (sender_email)

### sequence_steps
id uuid=gen_random_uuid(), sequence_id uuid, step_order integer, days_from_start integer, delay_from_previous_days integer, step_name text, subject_template text?, body_text_template text, body_html_template text?, required_variables jsonb='[]'::jsonb, claim_keys jsonb='[]'::jsonb, locked boolean=false, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK sequence_steps_days_from_start_check: CHECK ((days_from_start >= 0))
- CK sequence_steps_delay_from_previous_days_check: CHECK ((delay_from_previous_days >= 0))
- CK sequence_steps_step_order_check: CHECK ((step_order >= 1))
- FK sequence_steps_sequence_id_fkey: FOREIGN KEY (sequence_id) REFERENCES revenue_brain.sequence_templates(id) ON DELETE CASCADE
- PK sequence_steps_pkey: PRIMARY KEY (id)
- UQ sequence_steps_sequence_id_step_order_key: UNIQUE (sequence_id, step_order)

### sequence_templates
id uuid=gen_random_uuid(), sequence_code text, programme_code text, name text, version text, status text, sender_email text?, stop_on_reply boolean=true, cadence_days integer[]='{}'::integer[], variable_schema jsonb='[]'::jsonb, content_constraints jsonb='{}'::jsonb, approved_by text?, approved_at timestamp with time zone?, source_label text?, notes text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK sequence_templates_status_check: CHECK ((status = ANY (ARRAY['DRAFT'::text, 'LOCKED'::text, 'RETIRED'::text])))
- PK sequence_templates_pkey: PRIMARY KEY (id)
- UQ sequence_templates_sequence_code_key: UNIQUE (sequence_code)

### shadow_cases
id uuid=gen_random_uuid(), run_id uuid, case_key text, research_contact_id uuid?, school_name text, city text?, state text?, country text?, contact_name text?, contact_role text?, contact_email text?, website text?, source text?, source_kind text?, source_detail text?, email_validated boolean=false, verify_state text?, sop_priority text?, sop_role text?, sop_signal text?, lwl_history jsonb='{}'::jsonb, deterministic_snapshot jsonb='{}'::jsonb, ai_decision jsonb='{}'::jsonb, blockers jsonb='[]'::jsonb, readiness text='PENDING_AI'::text, would_send boolean=false, preview_subject text?, preview_body text?, status text='READY_FOR_AI'::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK shadow_cases_run_id_fkey: FOREIGN KEY (run_id) REFERENCES revenue_brain.shadow_runs(id) ON DELETE CASCADE
- PK shadow_cases_pkey: PRIMARY KEY (id)
- UQ shadow_cases_run_id_case_key_key: UNIQUE (run_id, case_key)

### shadow_runs
id uuid=gen_random_uuid(), run_code text, status text='CREATED'::text, programme_code text='PRG-FINEDU-FLAGSHIP-COMPETITION'::text, case_count integer=5, no_send boolean=true, started_at timestamp with time zone=now(), completed_at timestamp with time zone?, notes jsonb='{}'::jsonb, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK shadow_runs_pkey: PRIMARY KEY (id)
- UQ shadow_runs_run_code_key: UNIQUE (run_code)

### source_evidence
id uuid=gen_random_uuid(), entity_type text, entity_id text, fact_key text, fact_value text?, source_type text, source_url text?, source_label text?, verified_at timestamp with time zone?, confidence text?, evidence_status text='VERIFIED'::text, is_test_fixture boolean=false, created_at timestamp with time zone=now()
- CK source_evidence_entity_type_check: CHECK ((entity_type = ANY (ARRAY['SCHOOL'::text, 'CONTACT'::text, 'PROGRAMME'::text, 'HISTORY'::text, 'CLAIM'::text, 'SENDER'::text])))
- PK source_evidence_pkey: PRIMARY KEY (id)
- UQ source_evidence_entity_type_entity_id_fact_key_source_url_key: UNIQUE (entity_type, entity_id, fact_key, source_url)

### test_recipient_allowlist
email text, label text, active boolean=true, created_at timestamp with time zone=now()
- PK test_recipient_allowlist_pkey: PRIMARY KEY (email)

### verification_events
id uuid=gen_random_uuid(), contact_id text?, email text, provider text, status text, sub_status text?, verified_at timestamp with time zone=now(), raw_response jsonb='{}'::jsonb, send_eligibility text?, is_test boolean=false, created_at timestamp with time zone=now()
- PK verification_events_pkey: PRIMARY KEY (id)
- idx idx_rb_verification_email_provider_time USING btree (lower(email), provider, verified_at DESC)

### webhook_inbox
id uuid=gen_random_uuid(), provider text, provider_event_id text?, event_type text, provider_campaign_id text?, provider_lead_id text?, sender_email text?, recipient_email text?, event_payload jsonb='{}'::jsonb, received_at timestamp with time zone=now(), processed_at timestamp with time zone?, processing_status text='RECEIVED'::text, processing_error text?, is_test boolean=true
- PK webhook_inbox_pkey: PRIMARY KEY (id)
- idx webhook_inbox_provider_event_uidx UNIQUE USING btree (provider, provider_event_id) WHERE (provider_event_id IS NOT NULL)
- idx webhook_inbox_unprocessed_idx USING btree (processing_status, received_at)

### wow_internal_cert_cases
id uuid=gen_random_uuid(), run_id uuid, case_key text, school_name text, intended_contact_name text, intended_contact_role text?, intended_contact_email text?, delivery_email text, expected_reply_class text?, brain_decision_id uuid?, queue_id uuid?, created_at timestamp with time zone=now()
- FK wow_internal_cert_cases_brain_decision_id_fkey: FOREIGN KEY (brain_decision_id) REFERENCES revenue_brain.brain_decisions(id)
- FK wow_internal_cert_cases_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id)
- FK wow_internal_cert_cases_run_id_fkey: FOREIGN KEY (run_id) REFERENCES revenue_brain.wow_internal_cert_runs(id) ON DELETE CASCADE
- PK wow_internal_cert_cases_pkey: PRIMARY KEY (id)
- UQ wow_internal_cert_cases_run_id_case_key_key: UNIQUE (run_id, case_key)

### wow_internal_cert_runs
id uuid=gen_random_uuid(), run_code text, provider_campaign_id text, status text='CREATED'::text, case_count integer=0, enrolled_count integer=0, reply_count integer=0, handoff_count integer=0, final_result text?, notes jsonb='{}'::jsonb, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), completed_at timestamp with time zone?
- PK wow_internal_cert_runs_pkey: PRIMARY KEY (id)
- UQ wow_internal_cert_runs_run_code_key: UNIQUE (run_code)

### wow_pilot_case_release
id uuid=gen_random_uuid(), preflight_case_id uuid, copy_review_case_id uuid?, queue_id uuid, school_name text, contact_name text, contact_email text, approval_status text='PENDING'::text, approved_by text?, approved_at timestamp with time zone?, dispatch_status text='NOT_AUTHORIZED'::text, orchestration_run_id uuid?, provider_lead_id text?, dispatch_error text?, dispatched_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK wow_pilot_case_release_copy_review_case_id_fkey: FOREIGN KEY (copy_review_case_id) REFERENCES revenue_brain.pilot_copy_review_cases(id) ON DELETE SET NULL
- FK wow_pilot_case_release_orchestration_run_id_fkey: FOREIGN KEY (orchestration_run_id) REFERENCES revenue_brain.orchestration_runs(id)
- FK wow_pilot_case_release_preflight_case_id_fkey: FOREIGN KEY (preflight_case_id) REFERENCES revenue_brain.pilot_preflight_cases(id) ON DELETE CASCADE
- FK wow_pilot_case_release_queue_id_fkey: FOREIGN KEY (queue_id) REFERENCES revenue_brain.outbound_queue(id) ON DELETE CASCADE
- PK wow_pilot_case_release_pkey: PRIMARY KEY (id)
- UQ wow_pilot_case_release_preflight_case_id_key: UNIQUE (preflight_case_id)

### wow_pilot_release
id uuid=gen_random_uuid(), programme_code text='PRG-FINEDU-FLAGSHIP-COMPETITION'::text, sequence_code text='WOW_V2_DRAFT_2026-10-12'::text, route_key text='PILOT_ET_V2'::text, provider_campaign_id text='87d416cc-6ab4-44f6-a15d-6c6ef8d5c034'::text, copy_status text='PENDING'::text, copy_approved_by text?, copy_approved_at timestamp with time zone?, launch_status text='PENDING'::text, launch_approved_by text?, launch_approved_at timestamp with time zone?, max_cases integer=3, status text='SETUP'::text, notes jsonb='{}'::jsonb, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK wow_pilot_release_pkey: PRIMARY KEY (id)
- idx wow_pilot_release_singleton UNIQUE USING btree (programme_code)

### yale_contacts
id bigint=nextval('revenue_brain.yale_contacts_id_seq'::regclass), source_file text, source_row integer, programme_code text='CO-YIRA-YGEC-090127'::text, campus_id text?, school text?, location text?, contact_name text?, designation text?, precise_role text?, email_raw text?, email_norm text?, email_source_url text?, role_source_url text?, current_email_evidence_url text?, current_role_evidence_url text?, evidence_date text?, evidence_checked_on text?, email_publisher_type text?, role_verified_raw text?, verification_tier text?, campus_merge_status text?, dup_flags text?, audit_notes text?, email_valid_format boolean?, email_domain text?, personal_domain boolean?, generic_inbox boolean?, duplicate_email_rows integer?, suppressed boolean?, suppression_source text?, domain_dns text?, role_verified boolean?, email_official boolean?, role_bucket text?, role_relevance integer?, lwl_apps_2026 integer?, lwl_paid_2026 integer?, lwl_apps_2027 integer?, lwl_paid_2027 integer?, lwl_apps_any integer?, crm_owner text?, crm_stage text?, relationship text?, gate_status text?, gate_reasons text[]?, reconciled_at timestamp with time zone?, imported_at timestamp with time zone=now(), evidence_caveat boolean?
- PK yale_contacts_pkey: PRIMARY KEY (id)
- UQ yale_contacts_source_file_source_row_key: UNIQUE (source_file, source_row)
- idx yale_contacts_email_idx USING btree (email_norm)
- idx yale_contacts_campus_idx USING btree (campus_id)

### yale_pilot_leads
n integer, email text, greeting text, campus_key text, school_id text?, contact_id text?, school_name text?, school_code text?, queue_id uuid?, status text='APPROVED'::text, hold_reason text?, run_id uuid?, provider_lead_id text?, approved_by text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), wave text='PILOT_100'::text, route_key text='PILOT_US'::text, role_line text?, evidence_level text?
- PK yale_pilot_leads_pkey: PRIMARY KEY (n)
- UQ yale_pilot_leads_email_key: UNIQUE (email)

### yale_school_codes
campus_key text, campus_id text?, school text?, school_code text, utm_query text, created_at timestamp with time zone=now()
- PK yale_school_codes_pkey: PRIMARY KEY (campus_key)
- UQ yale_school_codes_school_code_key: UNIQUE (school_code)

