# Shared estate columns: lms (79 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### lms.admin_profiles

| column | type | nullable | default |
|---|---|---|---|
| user_id | uuid | NOT NULL |  |
| email | text | NOT NULL |  |
| full_name | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `admin_profiles_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `admin_profiles_pkey` PRIMARY KEY (user_id)
- `admin_profiles_email_key` UNIQUE (email)

Indexes:
- UNIQUE INDEX admin_profiles_email_key USING btree (email)
- UNIQUE INDEX admin_profiles_pkey USING btree (user_id)

### lms.birthday_grants

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_user_id | uuid | NOT NULL |  |
| year | integer | NOT NULL |  |
| birthday | date | NOT NULL |  |
| zone | text | NOT NULL |  |
| lot_id | uuid | null |  |
| amount | numeric | NOT NULL |  |
| expires_at | timestamp with time zone | NOT NULL |  |
| email_status | text | NOT NULL | 'pending'::text |
| email_attempts | integer | NOT NULL | 0 |
| email_last_error | text | null |  |
| email_sent_at | timestamp with time zone | null |  |
| popup_seen_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `birthday_grants_email_status_check` CHECK ((email_status = ANY (ARRAY['pending'::text, 'sent'::text, 'failed'::text, 'skipped'::text])))
- `birthday_grants_lot_id_fkey` FOREIGN KEY (lot_id) REFERENCES lms.vault_lots(id)
- `birthday_grants_student_user_id_fkey` FOREIGN KEY (student_user_id) REFERENCES lms.student_profiles(user_id) ON DELETE CASCADE
- `birthday_grants_pkey` PRIMARY KEY (id)
- `birthday_grants_student_user_id_year_key` UNIQUE (student_user_id, year)

Indexes:
- UNIQUE INDEX birthday_grants_pkey USING btree (id)
- UNIQUE INDEX birthday_grants_student_user_id_year_key USING btree (student_user_id, year)

### lms.certificate_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_id | uuid | NOT NULL |  |
| kind | text | NOT NULL |  |
| html | text | NOT NULL |  |
| demo_values | jsonb | NOT NULL | '{}'::jsonb |
| updated_at | timestamp with time zone | NOT NULL | now() |
| updated_by | uuid | null |  |

Constraints:
- `certificate_templates_kind_check` CHECK ((kind = ANY (ARRAY['participation'::text, 'winner'::text])))
- `certificate_templates_pkey` PRIMARY KEY (id)
- `certificate_templates_program_id_kind_key` UNIQUE (program_id, kind)

Indexes:
- UNIQUE INDEX certificate_templates_pkey USING btree (id)
- UNIQUE INDEX certificate_templates_program_id_kind_key USING btree (program_id, kind)

### lms.certificates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| program_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| kind | text | NOT NULL |  |
| issue_date | date | NOT NULL |  |
| tag_values | jsonb | NOT NULL | '{}'::jsonb |
| issued_at | timestamp with time zone | NOT NULL | now() |
| issued_by | uuid | null |  |
| seen_at | timestamp with time zone | null |  |
| revoked_at | timestamp with time zone | null |  |
| revoked_by | uuid | null |  |

Constraints:
- `certificates_kind_check` CHECK ((kind = ANY (ARRAY['participation'::text, 'winner'::text])))
- `certificates_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX certificates_one_active USING btree (cohort_id, student_user_id, kind) WHERE (revoked_at IS NULL)
- UNIQUE INDEX certificates_pkey USING btree (id)
- INDEX certificates_student USING btree (student_user_id)

### lms.cohort_completions

| column | type | nullable | default |
|---|---|---|---|
| cohort_id | uuid | NOT NULL |  |
| is_complete | boolean | NOT NULL | true |
| completed_at | timestamp with time zone | NOT NULL | now() |
| completed_by | uuid | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `cohort_completions_pkey` PRIMARY KEY (cohort_id)

Indexes:
- UNIQUE INDEX cohort_completions_pkey USING btree (cohort_id)

### lms.cohort_group_members

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| group_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| added_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `cohort_group_members_group_id_fkey` FOREIGN KEY (group_id) REFERENCES lms.cohort_groups(id) ON DELETE CASCADE
- `cohort_group_members_pkey` PRIMARY KEY (id)
- `cohort_group_members_cohort_id_student_user_id_key` UNIQUE (cohort_id, student_user_id)

Indexes:
- UNIQUE INDEX cohort_group_members_cohort_id_student_user_id_key USING btree (cohort_id, student_user_id)
- INDEX cohort_group_members_group_idx USING btree (group_id)
- UNIQUE INDEX cohort_group_members_pkey USING btree (id)

### lms.cohort_group_settings

| column | type | nullable | default |
|---|---|---|---|
| cohort_id | uuid | NOT NULL |  |
| groups_not_needed | boolean | NOT NULL | false |
| updated_by | uuid | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |
| default_mentor_id | uuid | null |  |

Constraints:
- `cohort_group_settings_pkey` PRIMARY KEY (cohort_id)

Indexes:
- UNIQUE INDEX cohort_group_settings_pkey USING btree (cohort_id)

### lms.cohort_groups

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| name | text | NOT NULL |  |
| sort_index | integer | NOT NULL |  |
| mentor_id | uuid | null |  |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| section_id | uuid | null |  |

Constraints:
- `cohort_groups_mentor_id_fkey` FOREIGN KEY (mentor_id) REFERENCES all_mentors(id) ON DELETE SET NULL
- `cohort_groups_section_id_fkey` FOREIGN KEY (section_id) REFERENCES lms.cohort_sections(id) ON DELETE SET NULL
- `cohort_groups_pkey` PRIMARY KEY (id)
- `cohort_groups_cohort_id_sort_index_key` UNIQUE (cohort_id, sort_index)

Indexes:
- UNIQUE INDEX cohort_groups_cohort_id_sort_index_key USING btree (cohort_id, sort_index)
- UNIQUE INDEX cohort_groups_pkey USING btree (id)

### lms.cohort_pathways

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| name | text | NOT NULL |  |
| sort_order | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `cohort_pathways_cohort_id_fkey` FOREIGN KEY (cohort_id) REFERENCES all_cohorts(id) ON DELETE CASCADE
- `cohort_pathways_pkey` PRIMARY KEY (id)
- `cohort_pathways_cohort_id_name_key` UNIQUE (cohort_id, name)

Indexes:
- UNIQUE INDEX cohort_pathways_cohort_id_name_key USING btree (cohort_id, name)
- UNIQUE INDEX cohort_pathways_pkey USING btree (id)

### lms.cohort_section_members

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| section_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| added_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `cohort_section_members_section_id_fkey` FOREIGN KEY (section_id) REFERENCES lms.cohort_sections(id) ON DELETE CASCADE
- `cohort_section_members_pkey` PRIMARY KEY (id)
- `cohort_section_members_cohort_id_student_user_id_key` UNIQUE (cohort_id, student_user_id)

Indexes:
- UNIQUE INDEX cohort_section_members_cohort_id_student_user_id_key USING btree (cohort_id, student_user_id)
- UNIQUE INDEX cohort_section_members_pkey USING btree (id)
- INDEX cohort_section_members_section_idx USING btree (section_id)

### lms.cohort_sections

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| name | text | NOT NULL |  |
| sort_index | integer | NOT NULL | 0 |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `cohort_sections_name_check` CHECK ((length(TRIM(BOTH FROM name)) > 0))
- `cohort_sections_pkey` PRIMARY KEY (id)
- `cohort_sections_cohort_id_name_key` UNIQUE (cohort_id, name)

Indexes:
- UNIQUE INDEX cohort_sections_cohort_id_name_key USING btree (cohort_id, name)
- INDEX cohort_sections_cohort_idx USING btree (cohort_id)
- UNIQUE INDEX cohort_sections_pkey USING btree (id)

### lms.community_ai_reviews

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| channel_id | uuid | NOT NULL |  |
| cohort_id | uuid | null |  |
| status | text | NOT NULL |  |
| headline | text | NOT NULL |  |
| highlights | jsonb | NOT NULL | '[]'::jsonb |
| message_count | integer | NOT NULL | 0 |
| from_message_at | timestamp with time zone | null |  |
| through_message_at | timestamp with time zone | null |  |
| model | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `community_ai_reviews_status_check` CHECK ((status = ANY (ARRAY['ok'::text, 'watch'::text, 'action'::text])))
- `community_ai_reviews_channel_id_fkey` FOREIGN KEY (channel_id) REFERENCES lms.community_channels(id) ON DELETE CASCADE
- `community_ai_reviews_pkey` PRIMARY KEY (id)

Indexes:
- INDEX community_ai_reviews_channel_idx USING btree (channel_id, created_at DESC)
- UNIQUE INDEX community_ai_reviews_pkey USING btree (id)

### lms.community_channels

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `community_channels_pkey` PRIMARY KEY (id)
- `community_channels_cohort_id_key` UNIQUE (cohort_id)

Indexes:
- UNIQUE INDEX community_channels_cohort_id_key USING btree (cohort_id)
- UNIQUE INDEX community_channels_pkey USING btree (id)

### lms.community_mentions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| message_id | uuid | NOT NULL |  |
| channel_id | uuid | NOT NULL |  |
| mentioned_user_id | uuid | NOT NULL |  |
| sender_user_id | uuid | NOT NULL |  |
| sender_name | text | null |  |
| excerpt | text | null |  |
| read_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `community_mentions_channel_id_fkey` FOREIGN KEY (channel_id) REFERENCES lms.community_channels(id) ON DELETE CASCADE
- `community_mentions_message_id_fkey` FOREIGN KEY (message_id) REFERENCES lms.community_messages(id) ON DELETE CASCADE
- `community_mentions_pkey` PRIMARY KEY (id)
- `community_mentions_message_id_mentioned_user_id_key` UNIQUE (message_id, mentioned_user_id)

Indexes:
- UNIQUE INDEX community_mentions_message_id_mentioned_user_id_key USING btree (message_id, mentioned_user_id)
- UNIQUE INDEX community_mentions_pkey USING btree (id)
- INDEX community_mentions_user_idx USING btree (mentioned_user_id, created_at DESC)

### lms.community_messages

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| channel_id | uuid | NOT NULL |  |
| sender_user_id | uuid | NOT NULL |  |
| sender_name | text | null |  |
| sender_role | text | NOT NULL | 'student'::text |
| body | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| edited_at | timestamp with time zone | null |  |
| deleted_at | timestamp with time zone | null |  |

Constraints:
- `community_messages_channel_id_fkey` FOREIGN KEY (channel_id) REFERENCES lms.community_channels(id) ON DELETE CASCADE
- `community_messages_pkey` PRIMARY KEY (id)

Indexes:
- INDEX community_messages_channel_created_idx USING btree (channel_id, created_at)
- UNIQUE INDEX community_messages_pkey USING btree (id)

### lms.community_moderation_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| channel_id | uuid | null |  |
| sender_user_id | uuid | null |  |
| body | text | NOT NULL |  |
| rule | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `community_moderation_log_pkey` PRIMARY KEY (id)

Indexes:
- INDEX community_moderation_log_created_idx USING btree (created_at DESC)
- UNIQUE INDEX community_moderation_log_pkey USING btree (id)
- INDEX community_moderation_log_sender_idx USING btree (sender_user_id)

### lms.community_reads

| column | type | nullable | default |
|---|---|---|---|
| channel_id | uuid | NOT NULL |  |
| user_id | uuid | NOT NULL |  |
| last_read_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `community_reads_channel_id_fkey` FOREIGN KEY (channel_id) REFERENCES lms.community_channels(id) ON DELETE CASCADE
- `community_reads_pkey` PRIMARY KEY (channel_id, user_id)

Indexes:
- UNIQUE INDEX community_reads_pkey USING btree (channel_id, user_id)

### lms.community_session_reminder_log

| column | type | nullable | default |
|---|---|---|---|
| session_id | uuid | NOT NULL |  |
| channel_id | uuid | NOT NULL |  |
| message_id | uuid | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `community_session_reminder_log_channel_id_fkey` FOREIGN KEY (channel_id) REFERENCES lms.community_channels(id) ON DELETE CASCADE
- `community_session_reminder_log_message_id_fkey` FOREIGN KEY (message_id) REFERENCES lms.community_messages(id)
- `community_session_reminder_log_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `community_session_reminder_log_pkey` PRIMARY KEY (session_id, channel_id)

Indexes:
- UNIQUE INDEX community_session_reminder_log_pkey USING btree (session_id, channel_id)

### lms.email_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| slug | text | NOT NULL |  |
| kind | text | NOT NULL | 'automated'::text |
| trigger_event | text | null |  |
| category | text | null |  |
| description | text | null |  |
| subject | text | NOT NULL |  |
| html | text | NOT NULL |  |
| variables | jsonb | NOT NULL | '[]'::jsonb |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_templates_pkey` PRIMARY KEY (id)
- `email_templates_slug_key` UNIQUE (slug)

Indexes:
- UNIQUE INDEX email_templates_pkey USING btree (id)
- UNIQUE INDEX email_templates_slug_key USING btree (slug)

### lms.enrollment_errors

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| source_table | text | null |  |
| source_application_id | uuid | null |  |
| raw_cohort_value | text | null |  |
| reason | text | null |  |
| resolved_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `enrollment_errors_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX enrollment_errors_pkey USING btree (id)

### lms.enrollment_events

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| enrollment_id | uuid | null |  |
| student_user_id | uuid | null |  |
| cohort_id | uuid | null |  |
| event_type | text | NOT NULL |  |
| action | text | NOT NULL |  |
| source | text | NOT NULL |  |
| actor_user_id | uuid | null |  |
| reason | text | null |  |
| before | jsonb | null |  |
| after | jsonb | null |  |
| external_event_id | text | null |  |
| occurred_at | timestamp with time zone | NOT NULL | now() |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `enrollment_events_source_check` CHECK ((source = ANY (ARRAY['webhook'::text, 'reconcile'::text, 'admin'::text, 'provision'::text])))
- `enrollment_events_pkey` PRIMARY KEY (id)

Indexes:
- INDEX enrollment_events_enrollment_idx USING btree (enrollment_id, created_at DESC)
- UNIQUE INDEX enrollment_events_external_event_id_key USING btree (external_event_id) WHERE (external_event_id IS NOT NULL)
- UNIQUE INDEX enrollment_events_pkey USING btree (id)
- INDEX enrollment_events_student_idx USING btree (student_user_id, created_at DESC)

### lms.enrollment_run_items

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| run_id | uuid | NOT NULL |  |
| source_table | text | null |  |
| source_application_id | text | null |  |
| student_email | text | null |  |
| student_name | text | null |  |
| result | text | NOT NULL |  |
| detail | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `enrollment_run_items_run_id_fkey` FOREIGN KEY (run_id) REFERENCES lms.enrollment_runs(id) ON DELETE CASCADE
- `enrollment_run_items_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX enrollment_run_items_pkey USING btree (id)

### lms.enrollment_runs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| admin_user_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| is_test | boolean | NOT NULL | false |
| requested | integer | NOT NULL | 0 |
| succeeded | integer | NOT NULL | 0 |
| failed | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `enrollment_runs_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX enrollment_runs_pkey USING btree (id)

### lms.enrollment_settings

| column | type | nullable | default |
|---|---|---|---|
| id | boolean | NOT NULL | true |
| auto_enroll_enabled | boolean | NOT NULL | false |
| auto_unenroll_enabled | boolean | NOT NULL | false |
| provision_unpaid_accounts | boolean | NOT NULL | false |
| dry_run | boolean | NOT NULL | true |
| max_changes_per_run | integer | NOT NULL | 25 |
| alert_email | text | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `enrollment_settings_id_check` CHECK (id)
- `enrollment_settings_max_changes_per_run_check` CHECK (((max_changes_per_run >= 0) AND (max_changes_per_run <= 1000)))
- `enrollment_settings_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX enrollment_settings_pkey USING btree (id)

### lms.enrollment_sync_items

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| run_id | uuid | NOT NULL |  |
| source_table | text | null |  |
| source_application_id | text | null |  |
| email | text | null |  |
| cohort_id | uuid | null |  |
| planned_action | text | NOT NULL |  |
| result | text | NOT NULL |  |
| detail | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `enrollment_sync_items_run_id_fkey` FOREIGN KEY (run_id) REFERENCES lms.enrollment_sync_runs(id) ON DELETE CASCADE
- `enrollment_sync_items_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX enrollment_sync_items_pkey USING btree (id)
- INDEX enrollment_sync_items_run_idx USING btree (run_id)

### lms.enrollment_sync_runs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| trigger | text | NOT NULL |  |
| started_at | timestamp with time zone | NOT NULL | now() |
| finished_at | timestamp with time zone | null |  |
| mode | text | NOT NULL |  |
| status | text | NOT NULL | 'running'::text |
| planned_count | integer | NOT NULL | 0 |
| applied_count | integer | NOT NULL | 0 |
| skipped_count | integer | NOT NULL | 0 |
| error_count | integer | NOT NULL | 0 |
| note | text | null |  |
| actor_user_id | uuid | null |  |

Constraints:
- `enrollment_sync_runs_mode_check` CHECK ((mode = ANY (ARRAY['dry_run'::text, 'live'::text])))
- `enrollment_sync_runs_status_check` CHECK ((status = ANY (ARRAY['running'::text, 'ok'::text, 'blocked_by_guard'::text, 'failed'::text])))
- `enrollment_sync_runs_trigger_check` CHECK ((trigger = ANY (ARRAY['scheduled'::text, 'manual'::text, 'webhook'::text])))
- `enrollment_sync_runs_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX enrollment_sync_runs_pkey USING btree (id)
- INDEX enrollment_sync_runs_started_idx USING btree (started_at DESC)

### lms.enrollments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_user_id | uuid | NOT NULL |  |
| cohort_id | uuid | null |  |
| pathway_id | uuid | null |  |
| payment_status | text | NOT NULL | 'pending'::text |
| access_status | text | NOT NULL | 'pending'::text |
| amount | numeric | null |  |
| currency | character(3) | null | 'USD'::bpchar |
| source_table | text | null |  |
| source_application_id | uuid | null |  |
| enrolled_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| unenrolled_at | timestamp with time zone | null |  |
| unenrolled_reason | text | null |  |
| manual_override | boolean | NOT NULL | false |
| override_by | uuid | null |  |
| override_reason | text | null |  |
| override_at | timestamp with time zone | null |  |
| last_event_at | timestamp with time zone | null |  |

Constraints:
- `enrollments_unenrolled_reason_check` CHECK (((unenrolled_reason IS NULL) OR (unenrolled_reason = ANY (ARRAY['refund'::text, 'admin'::text, 'other'::text]))))
- `enrollments_cohort_id_fkey` FOREIGN KEY (cohort_id) REFERENCES all_cohorts(id) ON DELETE RESTRICT
- `enrollments_pathway_id_fkey` FOREIGN KEY (pathway_id) REFERENCES lms.cohort_pathways(id) ON DELETE SET NULL
- `enrollments_student_user_id_fkey` FOREIGN KEY (student_user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `enrollments_pkey` PRIMARY KEY (id)
- `enrollments_student_user_id_cohort_id_key` UNIQUE (student_user_id, cohort_id)

Indexes:
- INDEX enrollments_cohort_id_idx USING btree (cohort_id)
- UNIQUE INDEX enrollments_pkey USING btree (id)
- INDEX enrollments_student_active_idx USING btree (student_user_id, cohort_id) WHERE (access_status = 'active'::text)
- UNIQUE INDEX enrollments_student_user_id_cohort_id_key USING btree (student_user_id, cohort_id)
- INDEX enrollments_student_user_id_idx USING btree (student_user_id)

### lms.feedback_email_tokens

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| token_hash | text | NOT NULL |  |
| session_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| sent_at | timestamp with time zone | NOT NULL | now() |
| expires_at | timestamp with time zone | NOT NULL | (now() + '14 days'::interval) |
| used_at | timestamp with time zone | null |  |
| send_status | text | null |  |

Constraints:
- `feedback_email_tokens_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `feedback_email_tokens_pkey` PRIMARY KEY (id)
- `feedback_email_tokens_session_id_student_user_id_key` UNIQUE (session_id, student_user_id)
- `feedback_email_tokens_token_hash_key` UNIQUE (token_hash)

Indexes:
- UNIQUE INDEX feedback_email_tokens_pkey USING btree (id)
- UNIQUE INDEX feedback_email_tokens_session_id_student_user_id_key USING btree (session_id, student_user_id)
- UNIQUE INDEX feedback_email_tokens_token_hash_key USING btree (token_hash)

### lms.feedback_nudge_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| kind | text | NOT NULL | 'session'::text |
| sent_on | date | NOT NULL | ((now() AT TIME ZONE 'Asia/Kolkata'::text))::date |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `feedback_nudge_log_pkey` PRIMARY KEY (id)
- `feedback_nudge_log_session_id_student_user_id_kind_sent_on_key` UNIQUE (session_id, student_user_id, kind, sent_on)

Indexes:
- UNIQUE INDEX feedback_nudge_log_pkey USING btree (id)
- INDEX feedback_nudge_log_sent_on_idx USING btree (sent_on)
- UNIQUE INDEX feedback_nudge_log_session_id_student_user_id_kind_sent_on_key USING btree (session_id, student_user_id, kind, sent_on)

### lms.grade_yearly_runs

| column | type | nullable | default |
|---|---|---|---|
| kind | text | NOT NULL |  |
| year | integer | NOT NULL |  |
| affected | integer | NOT NULL | 0 |
| ran_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `grade_yearly_runs_kind_check` CHECK ((kind = ANY (ARRAY['reminder'::text, 'promotion'::text])))
- `grade_yearly_runs_pkey` PRIMARY KEY (kind, year)

Indexes:
- UNIQUE INDEX grade_yearly_runs_pkey USING btree (kind, year)

### lms.help_messages

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_user_id | uuid | NOT NULL |  |
| role | text | NOT NULL |  |
| body | text | NOT NULL |  |
| offer_ticket | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `help_messages_role_check` CHECK ((role = ANY (ARRAY['user'::text, 'assistant'::text])))
- `help_messages_student_user_id_fkey` FOREIGN KEY (student_user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `help_messages_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX help_messages_pkey USING btree (id)
- INDEX help_messages_student_idx USING btree (student_user_id, created_at)

### lms.login_rollout_queue

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| kind | text | NOT NULL |  |
| priority | integer | NOT NULL |  |
| email | text | NOT NULL |  |
| cohort_id | uuid | null |  |
| cohort_start | date | null |  |
| source_table | text | NOT NULL |  |
| source_application_id | text | NOT NULL |  |
| student | jsonb | NOT NULL | '{}'::jsonb |
| amount | numeric | null |  |
| currency | text | null |  |
| status | text | NOT NULL | 'queued'::text |
| result | text | null |  |
| detail | text | null |  |
| email_sent | boolean | NOT NULL | false |
| queued_at | timestamp with time zone | NOT NULL | now() |
| queued_by | uuid | null |  |
| claimed_at | timestamp with time zone | null |  |
| processed_at | timestamp with time zone | null |  |
| attempts | integer | NOT NULL | 0 |
| retry_after | timestamp with time zone | null |  |
| retry_ctx | jsonb | null |  |
| burst | boolean | NOT NULL | false |

Constraints:
- `login_rollout_queue_kind_check` CHECK ((kind = ANY (ARRAY['enroll_paid'::text, 'login_only'::text, 'pending_unpaid'::text])))
- `login_rollout_queue_status_check` CHECK ((status = ANY (ARRAY['queued'::text, 'sending'::text, 'sent'::text, 'skipped'::text, 'failed'::text, 'cancelled'::text])))
- `login_rollout_queue_pkey` PRIMARY KEY (id)

Indexes:
- INDEX login_rollout_queue_next USING btree (status, priority, cohort_start, queued_at)
- UNIQUE INDEX login_rollout_queue_once USING btree (lower(email), COALESCE(cohort_id, '00000000-0000-0000-0000-000000000000'::uuid))
- UNIQUE INDEX login_rollout_queue_pkey USING btree (id)
- INDEX login_rollout_queue_sent USING btree (processed_at) WHERE email_sent

### lms.login_rollout_settings

| column | type | nullable | default |
|---|---|---|---|
| id | boolean | NOT NULL | true |
| enabled | boolean | NOT NULL | false |
| gap_seconds | integer | NOT NULL | 180 |
| window_hours | integer | NOT NULL | 6 |
| per_window | integer | NOT NULL | 100 |
| daily_cap | integer | NOT NULL | 400 |
| last_sent_at | timestamp with time zone | null |  |
| paused_reason | text | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |
| updated_by | uuid | null |  |
| cooldown_until | timestamp with time zone | null |  |
| burst_remaining | integer | NOT NULL | 0 |
| burst_gap_seconds | integer | NOT NULL | 5 |

Constraints:
- `login_rollout_settings_burst_gap_seconds_check` CHECK (((burst_gap_seconds >= 3) AND (burst_gap_seconds <= 60)))
- `login_rollout_settings_burst_remaining_check` CHECK (((burst_remaining >= 0) AND (burst_remaining <= 200)))
- `login_rollout_settings_daily_cap_check` CHECK (((daily_cap >= 1) AND (daily_cap <= 800)))
- `login_rollout_settings_gap_seconds_check` CHECK (((gap_seconds >= 60) AND (gap_seconds <= 3600)))
- `login_rollout_settings_id_check` CHECK (id)
- `login_rollout_settings_per_window_check` CHECK (((per_window >= 1) AND (per_window <= 200)))
- `login_rollout_settings_window_hours_check` CHECK (((window_hours >= 1) AND (window_hours <= 24)))
- `login_rollout_settings_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX login_rollout_settings_pkey USING btree (id)

### lms.mentor_accounts

| column | type | nullable | default |
|---|---|---|---|
| mentor_id | uuid | NOT NULL |  |
| user_id | uuid | NOT NULL |  |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `mentor_accounts_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `mentor_accounts_pkey` PRIMARY KEY (mentor_id)
- `mentor_accounts_user_id_key` UNIQUE (user_id)

Indexes:
- UNIQUE INDEX mentor_accounts_pkey USING btree (mentor_id)
- UNIQUE INDEX mentor_accounts_user_id_key USING btree (user_id)

### lms.mentor_feedback

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| feedback | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| updated_by | uuid | null |  |
| winner_category | text | null |  |

Constraints:
- `mentor_feedback_feedback_check` CHECK (((length(btrim(feedback)) >= 1) AND (length(btrim(feedback)) <= 10000)))
- `mentor_feedback_winner_category_check` CHECK (((winner_category IS NULL) OR ((length(btrim(winner_category)) >= 1) AND (length(btrim(winner_category)) <= 200))))
- `mentor_feedback_pkey` PRIMARY KEY (id)
- `mentor_feedback_cohort_id_student_user_id_key` UNIQUE (cohort_id, student_user_id)

Indexes:
- UNIQUE INDEX mentor_feedback_cohort_id_student_user_id_key USING btree (cohort_id, student_user_id)
- UNIQUE INDEX mentor_feedback_pkey USING btree (id)

### lms.mentor_settings

| column | type | nullable | default |
|---|---|---|---|
| mentor_id | uuid | NOT NULL |  |
| display_order | integer | NOT NULL | 0 |
| legacy_tags | jsonb | NOT NULL | '[]'::jsonb |
| is_listed | boolean | NOT NULL | true |

Constraints:
- `mentor_settings_mentor_id_fkey` FOREIGN KEY (mentor_id) REFERENCES all_mentors(id) ON DELETE CASCADE
- `mentor_settings_pkey` PRIMARY KEY (mentor_id)

Indexes:
- UNIQUE INDEX mentor_settings_pkey USING btree (mentor_id)

### lms.notification_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_user_id | uuid | null |  |
| channel | text | NOT NULL |  |
| recipient | text | NOT NULL |  |
| event_type | text | NOT NULL |  |
| status | text | NOT NULL |  |
| payload | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `notification_log_student_user_id_fkey` FOREIGN KEY (student_user_id) REFERENCES auth.users(id) ON DELETE SET NULL
- `notification_log_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX notification_log_pkey USING btree (id)

### lms.notification_recipients

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| notification_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| read_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `notification_recipients_notification_id_fkey` FOREIGN KEY (notification_id) REFERENCES lms.notifications(id) ON DELETE CASCADE
- `notification_recipients_student_user_id_fkey` FOREIGN KEY (student_user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `notification_recipients_pkey` PRIMARY KEY (id)
- `notification_recipients_notification_id_student_user_id_key` UNIQUE (notification_id, student_user_id)

Indexes:
- UNIQUE INDEX notification_recipients_notification_id_student_user_id_key USING btree (notification_id, student_user_id)
- UNIQUE INDEX notification_recipients_pkey USING btree (id)

### lms.notifications

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | null |  |
| program_id | uuid | null |  |
| partner_id | uuid | null |  |
| pathway_id | uuid | null |  |
| type | text | NOT NULL |  |
| title | text | NOT NULL |  |
| body | text | null |  |
| link_url | text | null |  |
| created_by_admin_id | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| image_url | text | null |  |
| image_aspect | text | null |  |
| pinned | boolean | NOT NULL | false |
| pinned_at | timestamp with time zone | null |  |
| edited_at | timestamp with time zone | null |  |

Constraints:
- `notifications_image_aspect_check` CHECK (((image_aspect IS NULL) OR (image_aspect = ANY (ARRAY['16:9'::text, '4:3'::text, '1:1'::text, '21:9'::text, 'original'::text]))))
- `notifications_scope_check` CHECK (((type = 'vault'::text) OR (((((cohort_id IS NOT NULL))::integer + ((program_id IS NOT NULL))::integer) + ((partner_id IS NOT NULL))::integer) = 1)))
- `notifications_cohort_id_fkey` FOREIGN KEY (cohort_id) REFERENCES all_cohorts(id) ON DELETE CASCADE
- `notifications_created_by_admin_id_fkey` FOREIGN KEY (created_by_admin_id) REFERENCES auth.users(id)
- `notifications_partner_id_fkey` FOREIGN KEY (partner_id) REFERENCES all_partners(id) ON DELETE CASCADE
- `notifications_pathway_id_fkey` FOREIGN KEY (pathway_id) REFERENCES lms.cohort_pathways(id) ON DELETE SET NULL
- `notifications_program_id_fkey` FOREIGN KEY (program_id) REFERENCES all_programs(id) ON DELETE CASCADE
- `notifications_pkey` PRIMARY KEY (id)

Indexes:
- INDEX notifications_cohort_id_idx USING btree (cohort_id)
- INDEX notifications_pinned_created_idx USING btree (pinned, created_at DESC)
- UNIQUE INDEX notifications_pkey USING btree (id)
- INDEX notifications_program_id_idx USING btree (program_id)

### lms.partner_checkout_config

| column | type | nullable | default |
|---|---|---|---|
| partner_id | uuid | NOT NULL |  |
| checkout_base_url | text | null |  |
| token_param_name | text | NOT NULL | 'token'::text |
| is_configured | boolean | NOT NULL | false |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `partner_checkout_config_pkey` PRIMARY KEY (partner_id)

Indexes:
- UNIQUE INDEX partner_checkout_config_pkey USING btree (partner_id)

### lms.program_feedback

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| mentor_id | uuid | null |  |
| overall_rating | smallint | null |  |
| mentor_rating | smallint | null |  |
| content_rating | smallint | null |  |
| skills_rating | smallint | null |  |
| recommend_rating | smallint | null |  |
| enjoyed_most | text | null |  |
| improvements | text | null |  |
| other_comments | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_feedback_pkey` PRIMARY KEY (id)
- `program_feedback_cohort_id_student_user_id_key` UNIQUE (cohort_id, student_user_id)

Indexes:
- UNIQUE INDEX program_feedback_cohort_id_student_user_id_key USING btree (cohort_id, student_user_id)
- UNIQUE INDEX program_feedback_pkey USING btree (id)

### lms.referral_application_attribution

| column | type | nullable | default |
|---|---|---|---|
| source_table | text | NOT NULL |  |
| application_id | uuid | NOT NULL |  |
| friend_identity | text | NOT NULL |  |
| program_code | text | NOT NULL |  |
| referrer_user_id | uuid | null |  |
| referral_link_id | uuid | null |  |
| submitted_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `referral_application_attribution_referral_link_id_fkey` FOREIGN KEY (referral_link_id) REFERENCES lms.referral_program_links(id)
- `referral_application_attribution_pkey` PRIMARY KEY (source_table, application_id)

Indexes:
- UNIQUE INDEX referral_application_attribution_pkey USING btree (source_table, application_id)
- INDEX referral_application_friend USING btree (friend_identity)

### lms.referral_first_payments

| column | type | nullable | default |
|---|---|---|---|
| friend_identity | text | NOT NULL |  |
| source_table | text | NOT NULL |  |
| application_id | uuid | NOT NULL |  |
| paid_at | timestamp with time zone | NOT NULL |  |
| paid_amount | numeric | null |  |
| paid_currency | text | null |  |
| referrer_user_id | uuid | null |  |
| enrollment_id | uuid | null |  |
| recorded_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `referral_first_payments_pkey` PRIMARY KEY (friend_identity)
- `referral_first_payments_source_table_application_id_key` UNIQUE (source_table, application_id)

Indexes:
- UNIQUE INDEX referral_first_payments_pkey USING btree (friend_identity)
- UNIQUE INDEX referral_first_payments_source_table_application_id_key USING btree (source_table, application_id)

### lms.referral_invites

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| inviter_user_id | uuid | NOT NULL |  |
| friend_name | text | NOT NULL |  |
| friend_email | text | NOT NULL |  |
| friend_phone | text | null |  |
| friend_grade | text | null |  |
| friend_school | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| token_hash | text | NOT NULL |  |
| status | text | NOT NULL | 'invited'::text |
| send_count | integer | NOT NULL | 1 |
| last_sent_at | timestamp with time zone | NOT NULL | now() |
| expires_at | timestamp with time zone | NOT NULL |  |
| consent_confirmed_at | timestamp with time zone | NOT NULL |  |
| joined_user_id | uuid | null |  |
| joined_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| followup_count | integer | NOT NULL | 0 |

Constraints:
- `referral_invites_friend_email_check` CHECK (((friend_email = lower(friend_email)) AND (length(friend_email) <= 255)))
- `referral_invites_friend_name_check` CHECK (((length(friend_name) >= 1) AND (length(friend_name) <= 120)))
- `referral_invites_status_check` CHECK ((status = ANY (ARRAY['invited'::text, 'joined'::text, 'expired'::text, 'cancelled'::text])))
- `referral_invites_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX referral_invites_current_identity USING btree (lms.referral_email_identity(friend_email)) WHERE (status = ANY (ARRAY['invited'::text, 'joined'::text]))
- INDEX referral_invites_inviter USING btree (inviter_user_id, created_at DESC)
- UNIQUE INDEX referral_invites_open_email USING btree (friend_email) WHERE (status = ANY (ARRAY['invited'::text, 'joined'::text]))
- UNIQUE INDEX referral_invites_pkey USING btree (id)
- UNIQUE INDEX referral_invites_token USING btree (token_hash)

### lms.referral_join_rewards

| column | type | nullable | default |
|---|---|---|---|
| friend_identity | text | NOT NULL |  |
| friend_user_id | uuid | NOT NULL |  |
| invite_id | uuid | NOT NULL |  |
| inviter_user_id | uuid | NOT NULL |  |
| amount | numeric | NOT NULL | 5 |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `referral_join_rewards_amount_check` CHECK ((amount = (5)::numeric))
- `referral_join_rewards_invite_id_fkey` FOREIGN KEY (invite_id) REFERENCES lms.referral_invites(id)
- `referral_join_rewards_pkey` PRIMARY KEY (friend_identity)
- `referral_join_rewards_friend_user_id_key` UNIQUE (friend_user_id)
- `referral_join_rewards_invite_id_key` UNIQUE (invite_id)

Indexes:
- UNIQUE INDEX referral_join_rewards_friend_user_id_key USING btree (friend_user_id)
- UNIQUE INDEX referral_join_rewards_invite_id_key USING btree (invite_id)
- UNIQUE INDEX referral_join_rewards_pkey USING btree (friend_identity)

### lms.referral_lookup_attempts

| column | type | nullable | default |
|---|---|---|---|
| id | bigint | NOT NULL | nextval('lms.referral_lookup_attempts_id_seq'::regclass) |
| ip_hash | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `referral_lookup_attempts_pkey` PRIMARY KEY (id)

Indexes:
- INDEX referral_lookup_attempts_idx USING btree (ip_hash, created_at)
- UNIQUE INDEX referral_lookup_attempts_pkey USING btree (id)

### lms.referral_program_integrations

| column | type | nullable | default |
|---|---|---|---|
| source_table | text | NOT NULL |  |
| program_ids | uuid[] | NOT NULL | '{}'::uuid[] |
| enabled | boolean | NOT NULL | false |
| verified_at | timestamp with time zone | null |  |

Constraints:
- `referral_program_integrations_pkey` PRIMARY KEY (source_table)

Indexes:
- UNIQUE INDEX referral_program_integrations_pkey USING btree (source_table)

### lms.referral_program_links

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_user_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| program_code | text | NOT NULL |  |
| source_table | text | NOT NULL |  |
| token_hash | text | NOT NULL |  |
| expires_at | timestamp with time zone | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `referral_program_links_source_table_fkey` FOREIGN KEY (source_table) REFERENCES lms.referral_program_integrations(source_table)
- `referral_program_links_pkey` PRIMARY KEY (id)
- `referral_program_links_token_hash_key` UNIQUE (token_hash)

Indexes:
- UNIQUE INDEX referral_program_links_pkey USING btree (id)
- UNIQUE INDEX referral_program_links_token_hash_key USING btree (token_hash)

### lms.referral_settings

| column | type | nullable | default |
|---|---|---|---|
| id | boolean | NOT NULL | true |
| invite_valid_days | integer | NOT NULL | 14 |
| daily_invite_limit | integer | NOT NULL | 10 |
| max_sends | integer | NOT NULL | 3 |
| rate_online | numeric(5,2) | NOT NULL | 2 |
| rate_in_person | numeric(5,2) | NOT NULL | 1.5 |
| rate_camp | numeric(5,2) | NOT NULL | 1 |
| updated_at | timestamp with time zone | NOT NULL | now() |
| updated_by | uuid | null |  |
| invite_reward_enabled | boolean | NOT NULL | false |
| program_referral_enabled | boolean | NOT NULL | false |
| invite_reward_starts_at | timestamp with time zone | NOT NULL | now() |
| total_invite_limit | integer | NOT NULL | 50 |

Constraints:
- `referral_settings_daily_invite_limit_check` CHECK (((daily_invite_limit >= 1) AND (daily_invite_limit <= 100)))
- `referral_settings_id_check` CHECK (id)
- `referral_settings_invite_valid_days_check` CHECK (((invite_valid_days >= 1) AND (invite_valid_days <= 90)))
- `referral_settings_max_sends_check` CHECK (((max_sends >= 1) AND (max_sends <= 10)))
- `referral_settings_rate_camp_check` CHECK (((rate_camp >= (0)::numeric) AND (rate_camp <= (50)::numeric)))
- `referral_settings_rate_in_person_check` CHECK (((rate_in_person >= (0)::numeric) AND (rate_in_person <= (50)::numeric)))
- `referral_settings_rate_online_check` CHECK (((rate_online >= (0)::numeric) AND (rate_online <= (50)::numeric)))
- `referral_settings_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX referral_settings_pkey USING btree (id)

### lms.referral_verifications

| column | type | nullable | default |
|---|---|---|---|
| invite_id | uuid | NOT NULL |  |
| code_hash | text | NOT NULL |  |
| expires_at | timestamp with time zone | NOT NULL |  |
| sent_at | timestamp with time zone | NOT NULL | now() |
| attempts | integer | NOT NULL | 0 |
| verified_at | timestamp with time zone | null |  |
| send_count | integer | NOT NULL | 1 |

Constraints:
- `referral_verifications_invite_id_fkey` FOREIGN KEY (invite_id) REFERENCES lms.referral_invites(id)
- `referral_verifications_pkey` PRIMARY KEY (invite_id)

Indexes:
- UNIQUE INDEX referral_verifications_pkey USING btree (invite_id)

### lms.registration_attempts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| ip_hash | text | NOT NULL |  |
| email_hash | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `registration_attempts_pkey` PRIMARY KEY (id)

Indexes:
- INDEX registration_attempts_email_idx USING btree (email_hash, created_at DESC)
- INDEX registration_attempts_ip_idx USING btree (ip_hash, created_at DESC)
- UNIQUE INDEX registration_attempts_pkey USING btree (id)

### lms.resources

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| pathway_id | uuid | null |  |
| session_id | uuid | null |  |
| title | text | NOT NULL |  |
| description | text | null |  |
| url | text | NOT NULL |  |
| kind | text | NOT NULL | 'link'::text |
| created_by_admin_id | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `resources_cohort_id_fkey` FOREIGN KEY (cohort_id) REFERENCES all_cohorts(id) ON DELETE CASCADE
- `resources_created_by_admin_id_fkey` FOREIGN KEY (created_by_admin_id) REFERENCES auth.users(id)
- `resources_pathway_id_fkey` FOREIGN KEY (pathway_id) REFERENCES lms.cohort_pathways(id) ON DELETE SET NULL
- `resources_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE SET NULL
- `resources_pkey` PRIMARY KEY (id)

Indexes:
- INDEX resources_cohort_id_idx USING btree (cohort_id)
- UNIQUE INDEX resources_pkey USING btree (id)

### lms.session_attendance

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| status | text | NOT NULL | 'present'::text |
| marked_by | uuid | null |  |
| marked_at | timestamp with time zone | NOT NULL | now() |
| source | text | NOT NULL | 'paste'::text |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `session_attendance_status_chk` CHECK ((status = ANY (ARRAY['present'::text, 'absent'::text, 'excused'::text])))
- `session_attendance_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_attendance_pkey` PRIMARY KEY (id)
- `session_attendance_session_id_student_user_id_key` UNIQUE (session_id, student_user_id)

Indexes:
- INDEX idx_session_attendance_session USING btree (session_id)
- INDEX idx_session_attendance_student USING btree (student_user_id)
- UNIQUE INDEX session_attendance_pkey USING btree (id)
- UNIQUE INDEX session_attendance_session_id_student_user_id_key USING btree (session_id, student_user_id)

### lms.session_feedback

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| cohort_id | uuid | null |  |
| mentor_id | uuid | null |  |
| session_quality_rating | smallint | NOT NULL |  |
| mentor_rating | smallint | NOT NULL |  |
| engagement_rating | smallint | NOT NULL |  |
| takeaway | text | null |  |
| submitted_at | timestamp with time zone | NOT NULL | now() |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `session_feedback_engagement_rating_check` CHECK (((engagement_rating >= 1) AND (engagement_rating <= 5)))
- `session_feedback_mentor_rating_check` CHECK (((mentor_rating >= 1) AND (mentor_rating <= 5)))
- `session_feedback_session_quality_rating_check` CHECK (((session_quality_rating >= 1) AND (session_quality_rating <= 5)))
- `session_feedback_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_feedback_pkey` PRIMARY KEY (id)
- `session_feedback_session_id_student_user_id_key` UNIQUE (session_id, student_user_id)

Indexes:
- UNIQUE INDEX session_feedback_pkey USING btree (id)
- UNIQUE INDEX session_feedback_session_id_student_user_id_key USING btree (session_id, student_user_id)
- INDEX session_feedback_session_idx USING btree (session_id)
- INDEX session_feedback_student_idx USING btree (student_user_id)

### lms.session_group_links

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| group_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| session_link | text | NOT NULL |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `session_group_links_session_link_check` CHECK ((length(TRIM(BOTH FROM session_link)) > 0))
- `session_group_links_group_id_fkey` FOREIGN KEY (group_id) REFERENCES lms.cohort_groups(id) ON DELETE CASCADE
- `session_group_links_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_group_links_pkey` PRIMARY KEY (id)
- `session_group_links_session_id_group_id_key` UNIQUE (session_id, group_id)

Indexes:
- INDEX session_group_links_cohort_idx USING btree (cohort_id)
- UNIQUE INDEX session_group_links_pkey USING btree (id)
- UNIQUE INDEX session_group_links_session_id_group_id_key USING btree (session_id, group_id)

### lms.session_join_clicks

| column | type | nullable | default |
|---|---|---|---|
| session_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| first_clicked_at | timestamp with time zone | NOT NULL | now() |
| last_clicked_at | timestamp with time zone | NOT NULL | now() |
| click_count | integer | NOT NULL | 1 |
| finalized_at | timestamp with time zone | null |  |

Constraints:
- `session_join_clicks_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_join_clicks_pkey` PRIMARY KEY (session_id, student_user_id)

Indexes:
- INDEX session_join_clicks_pending_idx USING btree (session_id) WHERE (finalized_at IS NULL)
- UNIQUE INDEX session_join_clicks_pkey USING btree (session_id, student_user_id)

### lms.session_mentor_overrides

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| group_id | uuid | null |  |
| mentor_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `session_mentor_overrides_group_id_fkey` FOREIGN KEY (group_id) REFERENCES lms.cohort_groups(id) ON DELETE CASCADE
- `session_mentor_overrides_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_mentor_overrides_pkey` PRIMARY KEY (id)

Indexes:
- INDEX session_mentor_overrides_cohort_idx USING btree (cohort_id)
- UNIQUE INDEX session_mentor_overrides_cohort_uniq USING btree (session_id) WHERE (group_id IS NULL)
- UNIQUE INDEX session_mentor_overrides_group_uniq USING btree (session_id, group_id) WHERE (group_id IS NOT NULL)
- UNIQUE INDEX session_mentor_overrides_pkey USING btree (id)

### lms.session_mentors

| column | type | nullable | default |
|---|---|---|---|
| session_id | uuid | NOT NULL |  |
| mentor_id | uuid | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `session_mentors_mentor_id_fkey` FOREIGN KEY (mentor_id) REFERENCES all_mentors(id) ON DELETE CASCADE
- `session_mentors_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_mentors_pkey` PRIMARY KEY (session_id, mentor_id)

Indexes:
- UNIQUE INDEX session_mentors_pkey USING btree (session_id, mentor_id)

### lms.session_recording_watch

| column | type | nullable | default |
|---|---|---|---|
| session_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| watched_seconds | integer | NOT NULL | 0 |
| first_seen_at | timestamp with time zone | NOT NULL | now() |
| last_beat_at | timestamp with time zone | NOT NULL | now() |
| qualified_at | timestamp with time zone | null |  |
| finalized_at | timestamp with time zone | null |  |

Constraints:
- `session_recording_watch_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_recording_watch_pkey` PRIMARY KEY (session_id, student_user_id)

Indexes:
- INDEX session_recording_watch_pending_idx USING btree (qualified_at) WHERE ((finalized_at IS NULL) AND (qualified_at IS NOT NULL))
- UNIQUE INDEX session_recording_watch_pkey USING btree (session_id, student_user_id)

### lms.session_reminder_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| offset_kind | text | NOT NULL |  |
| channel | text | NOT NULL | 'email'::text |
| sent_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `session_reminder_log_offset_kind_check` CHECK ((offset_kind = ANY (ARRAY['24h'::text, '6h'::text, '60m'::text, '10m'::text])))
- `session_reminder_log_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_reminder_log_student_user_id_fkey` FOREIGN KEY (student_user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `session_reminder_log_pkey` PRIMARY KEY (id)
- `session_reminder_log_session_id_student_user_id_offset_kind_key` UNIQUE (session_id, student_user_id, offset_kind, channel)

Indexes:
- UNIQUE INDEX session_reminder_log_pkey USING btree (id)
- UNIQUE INDEX session_reminder_log_session_id_student_user_id_offset_kind_key USING btree (session_id, student_user_id, offset_kind, channel)

### lms.session_section_links

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| section_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| session_link | text | NOT NULL |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `session_section_links_session_link_check` CHECK ((length(TRIM(BOTH FROM session_link)) > 0))
- `session_section_links_section_id_fkey` FOREIGN KEY (section_id) REFERENCES lms.cohort_sections(id) ON DELETE CASCADE
- `session_section_links_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_section_links_pkey` PRIMARY KEY (id)
- `session_section_links_session_id_section_id_key` UNIQUE (session_id, section_id)

Indexes:
- INDEX session_section_links_cohort_idx USING btree (cohort_id)
- UNIQUE INDEX session_section_links_pkey USING btree (id)
- UNIQUE INDEX session_section_links_session_id_section_id_key USING btree (session_id, section_id)

### lms.session_section_recordings

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| section_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| recording_url | text | NOT NULL |  |
| updated_by | uuid | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `session_section_recordings_recording_url_check` CHECK ((length(TRIM(BOTH FROM recording_url)) > 0))
- `session_section_recordings_section_id_fkey` FOREIGN KEY (section_id) REFERENCES lms.cohort_sections(id) ON DELETE CASCADE
- `session_section_recordings_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_section_recordings_pkey` PRIMARY KEY (id)
- `session_section_recordings_session_id_section_id_key` UNIQUE (session_id, section_id)

Indexes:
- INDEX session_section_recordings_cohort_idx USING btree (cohort_id)
- UNIQUE INDEX session_section_recordings_pkey USING btree (id)
- UNIQUE INDEX session_section_recordings_session_id_section_id_key USING btree (session_id, section_id)

### lms.session_submissions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| cohort_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| kind | text | NOT NULL |  |
| title | text | NOT NULL |  |
| url | text | null |  |
| storage_path | text | null |  |
| file_size | bigint | null |  |
| mime_type | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| review_status | text | NOT NULL | 'pending'::text |
| reviewed_by | uuid | null |  |
| reviewed_at | timestamp with time zone | null |  |

Constraints:
- `session_submissions_kind_check` CHECK ((kind = ANY (ARRAY['file'::text, 'link'::text])))
- `session_submissions_review_status_check` CHECK ((review_status = ANY (ARRAY['pending'::text, 'approved'::text, 'rejected'::text])))
- `session_submissions_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `session_submissions_pkey` PRIMARY KEY (id)

Indexes:
- INDEX session_submissions_cohort_idx USING btree (cohort_id)
- UNIQUE INDEX session_submissions_pkey USING btree (id)
- INDEX session_submissions_session_idx USING btree (session_id)
- INDEX session_submissions_student_idx USING btree (student_user_id)

### lms.sessions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| pathway_id | uuid | null |  |
| title | text | NOT NULL |  |
| description | text | null |  |
| session_datetime | timestamp with time zone | null |  |
| session_link | text | null |  |
| recording_url | text | null |  |
| posted_by_admin_id | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| session_mode | text | NOT NULL | 'online'::text |
| location | text | null |  |
| section_id | uuid | null |  |

Constraints:
- `sessions_session_mode_check` CHECK ((session_mode = ANY (ARRAY['online'::text, 'in_person'::text])))
- `sessions_cohort_id_fkey` FOREIGN KEY (cohort_id) REFERENCES all_cohorts(id) ON DELETE CASCADE
- `sessions_pathway_id_fkey` FOREIGN KEY (pathway_id) REFERENCES lms.cohort_pathways(id) ON DELETE SET NULL
- `sessions_posted_by_admin_id_fkey` FOREIGN KEY (posted_by_admin_id) REFERENCES auth.users(id)
- `sessions_section_id_fkey` FOREIGN KEY (section_id) REFERENCES lms.cohort_sections(id) ON DELETE SET NULL
- `sessions_pkey` PRIMARY KEY (id)

Indexes:
- INDEX sessions_cohort_id_idx USING btree (cohort_id)
- UNIQUE INDEX sessions_pkey USING btree (id)
- INDEX sessions_session_datetime_idx USING btree (session_datetime)

### lms.student_profiles

| column | type | nullable | default |
|---|---|---|---|
| user_id | uuid | NOT NULL |  |
| email | text | NOT NULL |  |
| full_name | text | null |  |
| phone | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| must_change_password | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| school_name | text | null |  |
| parent_name | text | null |  |
| policy_status | text | NOT NULL | 'pending'::text |
| policy_accepted_at | timestamp with time zone | null |  |
| policy_version | text | null |  |
| account_status | text | NOT NULL | 'active'::text |
| account_closed_at | timestamp with time zone | null |  |
| account_closed_reason | text | null |  |
| account_closed_by | uuid | null |  |
| registration_source | text | null | 'enrollment'::text |
| email_verified | boolean | NOT NULL | true |
| registered_at | timestamp with time zone | null |  |
| profile_prompted_at | timestamp with time zone | null |  |
| grade | text | null |  |
| grade_updated_at | timestamp with time zone | null |  |
| grade_promoted_year | integer | null |  |
| invited_by_user_id | uuid | null |  |
| referral_invite_id | uuid | null |  |
| date_of_birth | date | null |  |
| dob_source | text | null |  |
| dob_set_at | timestamp with time zone | null |  |
| dob_locked_at | timestamp with time zone | null |  |
| gender | text | null |  |
| country | text | null |  |
| state | text | null |  |
| city | text | null |  |
| referral_promo_seen_at | timestamp with time zone | null |  |
| referral_code | text | NOT NULL | ('Group-'::text \|\| upper(substr(replace((gen_random_uuid())::text, '-'::text, ''::text), 1, 10))) |

Constraints:
- `student_profiles_dob_source_check` CHECK ((dob_source = ANY (ARRAY['application'::text, 'student'::text, 'admin'::text])))
- `student_profiles_grade_chk` CHECK (((grade IS NULL) OR (grade = ANY (ARRAY['5'::text, '6'::text, '7'::text, '8'::text, '9'::text, '10'::text, '11'::text, '12'::text, 'gap_university'::text]))))
- `student_profiles_registration_source_chk` CHECK ((registration_source = ANY (ARRAY['enrollment'::text, 'invite'::text, 'self_signup'::text, 'admin'::text])))
- `student_profiles_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `student_profiles_pkey` PRIMARY KEY (user_id)

Indexes:
- INDEX student_profiles_created_at_idx USING btree (created_at DESC)
- UNIQUE INDEX student_profiles_pkey USING btree (user_id)
- UNIQUE INDEX student_profiles_referral_code_key USING btree (referral_code)

### lms.support_tickets

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| ref | text | NOT NULL | ('TKT-'::text \|\| lpad((nextval('lms.ticket_ref_seq'::regclass))::text, 5, '0'::text)) |
| student_user_id | uuid | NOT NULL |  |
| cohort_id | uuid | null |  |
| category | text | NOT NULL | 'other'::text |
| subject | text | NOT NULL |  |
| description | text | NOT NULL |  |
| screenshot_url | text | null |  |
| chat_context | jsonb | null |  |
| status | text | NOT NULL | 'open'::text |
| priority | text | NOT NULL | 'normal'::text |
| assigned_admin_id | uuid | null |  |
| student_last_read_at | timestamp with time zone | null |  |
| admin_last_read_at | timestamp with time zone | null |  |
| last_message_at | timestamp with time zone | NOT NULL | now() |
| resolved_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `support_tickets_priority_check` CHECK ((priority = ANY (ARRAY['low'::text, 'normal'::text, 'high'::text, 'urgent'::text])))
- `support_tickets_status_check` CHECK ((status = ANY (ARRAY['open'::text, 'in_progress'::text, 'waiting_student'::text, 'resolved'::text, 'closed'::text])))
- `support_tickets_assigned_admin_id_fkey` FOREIGN KEY (assigned_admin_id) REFERENCES auth.users(id) ON DELETE SET NULL
- `support_tickets_student_user_id_fkey` FOREIGN KEY (student_user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `support_tickets_pkey` PRIMARY KEY (id)
- `support_tickets_ref_key` UNIQUE (ref)

Indexes:
- UNIQUE INDEX support_tickets_pkey USING btree (id)
- UNIQUE INDEX support_tickets_ref_key USING btree (ref)
- INDEX support_tickets_status_idx USING btree (status, last_message_at DESC)
- INDEX support_tickets_student_idx USING btree (student_user_id, created_at DESC)

### lms.ticket_messages

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| ticket_id | uuid | NOT NULL |  |
| sender_user_id | uuid | NOT NULL |  |
| sender_role | text | NOT NULL |  |
| sender_name | text | null |  |
| body | text | NOT NULL |  |
| attachment_url | text | null |  |
| is_internal | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `ticket_messages_sender_role_check` CHECK ((sender_role = ANY (ARRAY['student'::text, 'admin'::text])))
- `ticket_messages_sender_user_id_fkey` FOREIGN KEY (sender_user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `ticket_messages_ticket_id_fkey` FOREIGN KEY (ticket_id) REFERENCES lms.support_tickets(id) ON DELETE CASCADE
- `ticket_messages_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX ticket_messages_pkey USING btree (id)
- INDEX ticket_messages_ticket_idx USING btree (ticket_id, created_at)

### lms.user_roles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `user_roles_role_check` CHECK ((role = ANY (ARRAY['admin'::text, 'student'::text, 'mentor'::text])))
- `user_roles_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `user_roles_pkey` PRIMARY KEY (id)
- `user_roles_user_id_role_key` UNIQUE (user_id, role)

Indexes:
- UNIQUE INDEX user_roles_pkey USING btree (id)
- UNIQUE INDEX user_roles_user_id_role_key USING btree (user_id, role)

### lms.vault_fx_rates

| column | type | nullable | default |
|---|---|---|---|
| currency | text | NOT NULL |  |
| per_usd | numeric | NOT NULL |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `vault_fx_rates_per_usd_check` CHECK ((per_usd > (0)::numeric))
- `vault_fx_rates_pkey` PRIMARY KEY (currency)

Indexes:
- UNIQUE INDEX vault_fx_rates_pkey USING btree (currency)

### lms.vault_ledger

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_user_id | uuid | NOT NULL |  |
| wallet | text | NOT NULL |  |
| currency | text | NOT NULL | 'USD'::text |
| lot_id | uuid | null |  |
| kind | text | NOT NULL |  |
| amount | numeric(12,2) | NOT NULL |  |
| ref_type | text | null |  |
| ref_id | text | null |  |
| description | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `vault_ledger_kind_check` CHECK ((kind = ANY (ARRAY['earn'::text, 'pending'::text, 'release'::text, 'reserve'::text, 'unreserve'::text, 'spend'::text, 'reversal'::text, 'expire'::text, 'adjust'::text])))
- `vault_ledger_wallet_check` CHECK ((wallet = ANY (ARRAY['loyalty'::text, 'effort'::text])))
- `vault_ledger_lot_id_fkey` FOREIGN KEY (lot_id) REFERENCES lms.vault_lots(id)
- `vault_ledger_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX vault_ledger_pkey USING btree (id)
- INDEX vault_ledger_student_idx USING btree (student_user_id, created_at DESC)

### lms.vault_lots

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_user_id | uuid | NOT NULL |  |
| wallet | text | NOT NULL |  |
| currency | text | NOT NULL | 'USD'::text |
| amount | numeric(12,2) | NOT NULL |  |
| remaining | numeric(12,2) | NOT NULL |  |
| state | text | NOT NULL |  |
| source_type | text | NOT NULL |  |
| source_key | text | NOT NULL |  |
| enrollment_id | uuid | null |  |
| cohort_id | uuid | null |  |
| counts_toward_level | boolean | NOT NULL | false |
| release_on | timestamp with time zone | null |  |
| released_at | timestamp with time zone | null |  |
| expires_at | timestamp with time zone | null |  |
| created_by | uuid | null |  |
| reason | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| expiry_warned_at | timestamp with time zone | null |  |

Constraints:
- `vault_lots_amount_check` CHECK ((amount > (0)::numeric))
- `vault_lots_remaining_check` CHECK ((remaining >= (0)::numeric))
- `vault_lots_source_type_check` CHECK ((source_type = ANY (ARRAY['profile'::text, 'attendance'::text, 'project'::text, 'completion'::text, 'cashback'::text, 'referral'::text, 'adjustment'::text, 'welcome'::text, 'session_feedback'::text, 'program_feedback'::text, 'birthday'::text])))
- `vault_lots_state_check` CHECK ((state = ANY (ARRAY['pending'::text, 'available'::text, 'expired'::text, 'reversed'::text, 'cancelled'::text])))
- `vault_lots_wallet_check` CHECK ((wallet = ANY (ARRAY['loyalty'::text, 'effort'::text])))
- `vault_lots_pkey` PRIMARY KEY (id)
- `vault_lots_source_key_key` UNIQUE (source_key)

Indexes:
- INDEX vault_lots_expiry_idx USING btree (state, expires_at) WHERE (state = 'available'::text)
- UNIQUE INDEX vault_lots_pkey USING btree (id)
- INDEX vault_lots_release_idx USING btree (state, release_on) WHERE (state = 'pending'::text)
- UNIQUE INDEX vault_lots_source_key_key USING btree (source_key)
- INDEX vault_lots_student_idx USING btree (student_user_id, wallet, state)

### lms.vault_review_flags

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_user_id | uuid | null |  |
| kind | text | NOT NULL |  |
| detail | jsonb | NOT NULL | '{}'::jsonb |
| resolved_at | timestamp with time zone | null |  |
| resolved_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `vault_review_flags_kind_check` CHECK ((kind = ANY (ARRAY['reversal_shortfall'::text, 'integrity'::text, 'manual'::text])))
- `vault_review_flags_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX vault_review_flags_pkey USING btree (id)

### lms.vault_settings

| column | type | nullable | default |
|---|---|---|---|
| id | boolean | NOT NULL | true |
| cashback_rate_online | numeric(5,2) | NOT NULL | 2.5 |
| cashback_rate_in_person | numeric(5,2) | NOT NULL | 1.5 |
| cashback_rate_camp | numeric(5,2) | NOT NULL | 1.0 |
| effort_profile | numeric(12,2) | NOT NULL | 100 |
| effort_session | numeric(12,2) | NOT NULL | 5 |
| effort_project | numeric(12,2) | NOT NULL | 10 |
| effort_completion | numeric(12,2) | NOT NULL | 25 |
| max_rewarded_projects_per_cohort | integer | NOT NULL | 2 |
| level_thresholds | jsonb | NOT NULL | '[50, 125, 250, 450, 750]'::jsonb |
| level_caps | jsonb | NOT NULL | '[5, 10, 15, 20, 25]'::jsonb |
| loyalty_expiry_months | integer | NOT NULL | 18 |
| effort_inactivity_months | integer | NOT NULL | 6 |
| voucher_valid_hours | integer | NOT NULL | 48 |
| referral_multiplier | numeric(5,2) | NOT NULL | 1 |
| referral_release_days | integer | NOT NULL | 14 |
| vault_enabled | boolean | NOT NULL | false |
| redemption_enabled | boolean | NOT NULL | false |
| allow_free_programs | boolean | NOT NULL | false |
| restore_credit_on_refund | boolean | NOT NULL | false |
| updated_at | timestamp with time zone | NOT NULL | now() |
| updated_by | uuid | null |  |
| notifications_enabled | boolean | NOT NULL | false |
| birthday_enabled | boolean | NOT NULL | true |
| birthday_amount | numeric | NOT NULL | 50 |
| birthday_valid_days | integer | NOT NULL | 5 |

Constraints:
- `vault_settings_cashback_rate_camp_check` CHECK ((cashback_rate_camp >= (0)::numeric))
- `vault_settings_cashback_rate_in_person_check` CHECK ((cashback_rate_in_person >= (0)::numeric))
- `vault_settings_cashback_rate_online_check` CHECK ((cashback_rate_online >= (0)::numeric))
- `vault_settings_effort_completion_check` CHECK ((effort_completion >= (0)::numeric))
- `vault_settings_effort_inactivity_months_check` CHECK ((effort_inactivity_months > 0))
- `vault_settings_effort_profile_check` CHECK ((effort_profile >= (0)::numeric))
- `vault_settings_effort_project_check` CHECK ((effort_project >= (0)::numeric))
- `vault_settings_effort_session_check` CHECK ((effort_session >= (0)::numeric))
- `vault_settings_id_check` CHECK (id)
- `vault_settings_loyalty_expiry_months_check` CHECK ((loyalty_expiry_months > 0))
- `vault_settings_max_rewarded_projects_per_cohort_check` CHECK ((max_rewarded_projects_per_cohort >= 0))
- `vault_settings_referral_multiplier_check` CHECK ((referral_multiplier >= (0)::numeric))
- `vault_settings_referral_release_days_check` CHECK ((referral_release_days >= 0))
- `vault_settings_voucher_valid_hours_check` CHECK ((voucher_valid_hours > 0))
- `vault_settings_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX vault_settings_pkey USING btree (id)

### lms.vault_settings_history

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| changed_by | uuid | null |  |
| reason | text | NOT NULL |  |
| before | jsonb | NOT NULL |  |
| after | jsonb | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `vault_settings_history_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX vault_settings_history_pkey USING btree (id)

### lms.vault_vouchers

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_user_id | uuid | NOT NULL |  |
| enrollment_id | uuid | null |  |
| cohort_id | uuid | null |  |
| currency | text | NOT NULL | 'USD'::text |
| price_snapshot | numeric(12,2) | null |  |
| effort_amount | numeric(12,2) | NOT NULL | 0 |
| loyalty_amount | numeric(12,2) | NOT NULL | 0 |
| allocations | jsonb | NOT NULL | '[]'::jsonb |
| status | text | NOT NULL | 'active'::text |
| code | text | null |  |
| token_hash | text | null |  |
| expires_at | timestamp with time zone | null |  |
| used_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| promo_code_id | uuid | null |  |
| program_id | uuid | null |  |
| list_price | numeric(12,2) | null |  |
| cash_due | numeric(12,2) | null |  |
| cancelled_at | timestamp with time zone | null |  |
| redemption | jsonb | null |  |
| promo_currency | text | null |  |
| promo_discount | numeric | null |  |
| fx_rate | numeric | null |  |
| cash_due_local | numeric | null |  |

Constraints:
- `vault_vouchers_status_check` CHECK ((status = ANY (ARRAY['active'::text, 'used'::text, 'cancelled'::text, 'expired'::text])))
- `vault_vouchers_pkey` PRIMARY KEY (id)
- `vault_vouchers_code_key` UNIQUE (code)

Indexes:
- INDEX vault_vouchers_code USING btree (code)
- UNIQUE INDEX vault_vouchers_code_key USING btree (code)
- UNIQUE INDEX vault_vouchers_one_active USING btree (enrollment_id) WHERE (status = 'active'::text)
- UNIQUE INDEX vault_vouchers_pkey USING btree (id)

### lms.vault_walkthrough_seen

| column | type | nullable | default |
|---|---|---|---|
| user_id | uuid | NOT NULL |  |
| seen_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `vault_walkthrough_seen_pkey` PRIMARY KEY (user_id)

Indexes:
- UNIQUE INDEX vault_walkthrough_seen_pkey USING btree (user_id)

### lms.whatsapp_blocklist_archive

| column | type | nullable | default |
|---|---|---|---|
| phone | text | NOT NULL |  |
| reason | text | NOT NULL | 'blocked'::text |
| source_file | text | null |  |
| created_by | uuid | null | auth.uid() |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_blocklist_phone` CHECK ((phone ~ '^[1-9][0-9]{7,14}$'::text))
- `whatsapp_blocklist_reason` CHECK ((reason = ANY (ARRAY['blocked'::text, 'undelivered'::text])))
- `whatsapp_blocklist_source_length` CHECK ((length(source_file) <= 255))
- `whatsapp_blocklist_pkey` PRIMARY KEY (phone)

Indexes:
- UNIQUE INDEX whatsapp_blocklist_pkey USING btree (phone)

### lms.whatsapp_reminder_dispatch

| column | type | nullable | default |
|---|---|---|---|
| session_id | uuid | NOT NULL |  |
| student_user_id | uuid | NOT NULL |  |
| offset_kind | text | NOT NULL |  |
| channel | text | NOT NULL |  |
| claim_token | uuid | NOT NULL |  |
| state | text | NOT NULL |  |
| broadcast_ids | text[] | NOT NULL | '{}'::text[] |
| last_error | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_reminder_dispatch_channel_check` CHECK ((channel = ANY (ARRAY['whatsapp'::text, 'whatsapp_parent'::text])))
- `whatsapp_reminder_dispatch_offset_kind_check` CHECK ((offset_kind = ANY (ARRAY['24h'::text, '6h'::text, '60m'::text, '10m'::text])))
- `whatsapp_reminder_dispatch_state_check` CHECK ((state = ANY (ARRAY['reserved'::text, 'submitting'::text, 'accepted'::text, 'rejected'::text, 'uncertain'::text])))
- `whatsapp_reminder_dispatch_session_id_fkey` FOREIGN KEY (session_id) REFERENCES lms.sessions(id) ON DELETE CASCADE
- `whatsapp_reminder_dispatch_pkey` PRIMARY KEY (session_id, student_user_id, offset_kind, channel)

Indexes:
- UNIQUE INDEX whatsapp_reminder_dispatch_pkey USING btree (session_id, student_user_id, offset_kind, channel)

### lms.whatsapp_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| trigger_event | text | null |  |
| template_name | text | NOT NULL |  |
| label | text | NOT NULL |  |
| language_code | text | NOT NULL | 'en'::text |
| category | text | NOT NULL | 'UTILITY'::text |
| header | text | null |  |
| body | text | null |  |
| footer | text | null |  |
| button_text | text | null |  |
| button_url | text | null |  |
| variable_mapping | jsonb | NOT NULL | '[]'::jsonb |
| example_values | jsonb | NOT NULL | '{}'::jsonb |
| is_system | boolean | NOT NULL | false |
| is_active | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| custom_variables | jsonb | NOT NULL | '[]'::jsonb |
| recipient_mode | text | NOT NULL | 'student'::text |
| parent_template | jsonb | NOT NULL | '{}'::jsonb |

Constraints:
- `whatsapp_templates_custom_variables_array` CHECK (((jsonb_typeof(custom_variables) = 'array'::text) AND (jsonb_array_length(custom_variables) <= 10)))
- `whatsapp_templates_parent_template_check` CHECK ((jsonb_typeof(parent_template) = 'object'::text))
- `whatsapp_templates_recipient_mode_check` CHECK ((recipient_mode = ANY (ARRAY['student'::text, 'parent'::text, 'both'::text])))
- `whatsapp_templates_pkey` PRIMARY KEY (id)
- `whatsapp_templates_trigger_event_key` UNIQUE (trigger_event)

Indexes:
- UNIQUE INDEX whatsapp_templates_pkey USING btree (id)
- UNIQUE INDEX whatsapp_templates_trigger_event_key USING btree (trigger_event)
