# Shared estate columns: leaders (22 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### leaders.badge_awards

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| ambassador_id | uuid | NOT NULL |  |
| badge_key | text | NOT NULL |  |
| badge_name | text | NOT NULL |  |
| awarded_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `badge_awards_pkey` PRIMARY KEY (id)
- `badge_awards_ambassador_id_badge_key_key` UNIQUE (ambassador_id, badge_key)

Indexes:
- UNIQUE INDEX badge_awards_ambassador_id_badge_key_key USING btree (ambassador_id, badge_key)
- UNIQUE INDEX badge_awards_pkey USING btree (id)

### leaders.cohort_reviewers

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| user_id | uuid | NOT NULL |  |

Constraints:
- `cohort_reviewers_cohort_id_fkey` FOREIGN KEY (cohort_id) REFERENCES leaders.cohorts(id) ON DELETE CASCADE
- `cohort_reviewers_pkey` PRIMARY KEY (id)
- `cohort_reviewers_cohort_id_user_id_key` UNIQUE (cohort_id, user_id)

Indexes:
- UNIQUE INDEX cohort_reviewers_cohort_id_user_id_key USING btree (cohort_id, user_id)
- UNIQUE INDEX cohort_reviewers_pkey USING btree (id)

### leaders.cohorts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| year | integer | NOT NULL | EXTRACT(year FROM now()) |
| invite_code | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `cohorts_pkey` PRIMARY KEY (id)
- `cohorts_invite_code_key` UNIQUE (invite_code)

Indexes:
- UNIQUE INDEX cohorts_invite_code_key USING btree (invite_code)
- UNIQUE INDEX cohorts_pkey USING btree (id)

### leaders.interface_copy

| column | type | nullable | default |
|---|---|---|---|
| key | text | NOT NULL |  |
| label | text | NOT NULL |  |
| value | text | NOT NULL |  |

Constraints:
- `interface_copy_pkey` PRIMARY KEY (key)

Indexes:
- UNIQUE INDEX interface_copy_pkey USING btree (key)

### leaders.level_progress

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| ambassador_id | uuid | NOT NULL |  |
| level_id | uuid | NOT NULL |  |
| level_number | integer | NOT NULL |  |
| status | leaders.progress_status | NOT NULL | 'open'::leaders.progress_status |
| ticked_tasks | uuid[] | NOT NULL | '{}'::uuid[] |
| work_link | text | NOT NULL | ''::text |
| reflection | text | NOT NULL | ''::text |
| impact_report | jsonb | null |  |
| submitted_at | timestamp with time zone | null |  |
| reviewed_at | timestamp with time zone | null |  |
| reviewer_id | uuid | null |  |
| reviewer_note | text | null |  |
| points_awarded | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `level_progress_level_id_fkey` FOREIGN KEY (level_id) REFERENCES leaders.levels(id) ON DELETE CASCADE
- `level_progress_pkey` PRIMARY KEY (id)
- `level_progress_ambassador_id_level_id_key` UNIQUE (ambassador_id, level_id)

Indexes:
- UNIQUE INDEX level_progress_ambassador_id_level_id_key USING btree (ambassador_id, level_id)
- UNIQUE INDEX level_progress_pkey USING btree (id)

### leaders.level_tasks

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| level_id | uuid | NOT NULL |  |
| position | integer | NOT NULL |  |
| label | text | NOT NULL |  |
| note | text | NOT NULL | ''::text |

Constraints:
- `level_tasks_level_id_fkey` FOREIGN KEY (level_id) REFERENCES leaders.levels(id) ON DELETE CASCADE
- `level_tasks_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX level_tasks_pkey USING btree (id)

### leaders.levels

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| number | integer | NOT NULL |  |
| numeral | text | NOT NULL |  |
| kicker | text | NOT NULL |  |
| title | text | NOT NULL |  |
| summary | text | NOT NULL | ''::text |
| introduction | text | NOT NULL |  |
| creative_challenge | text | NOT NULL |  |
| reflection_prompt | text | NOT NULL |  |
| scripts | jsonb | NOT NULL | '[]'::jsonb |
| assets | jsonb | NOT NULL | '[]'::jsonb |
| points | integer | NOT NULL |  |
| badge_key | text | NOT NULL |  |
| badge_name | text | NOT NULL |  |
| reward | text | NOT NULL |  |
| has_impact_report | boolean | NOT NULL | false |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `levels_pkey` PRIMARY KEY (id)
- `levels_number_key` UNIQUE (number)

Indexes:
- UNIQUE INDEX levels_number_key USING btree (number)
- UNIQUE INDEX levels_pkey USING btree (id)

### leaders.masterclass_regs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_key | text | NOT NULL | 'leaders'::text |
| masterclass_title | text | null |  |
| session_date | timestamp with time zone | null |  |
| full_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| country | text | null |  |
| state | text | null |  |
| city | text | null |  |
| role | text | null |  |
| grade | text | null |  |
| school_name | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| questions | text | null |  |
| sessions | text[] | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| email_sent | boolean | NOT NULL | false |
| email_sent_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `masterclass_regs_pkey` PRIMARY KEY (id)

Indexes:
- INDEX masterclass_regs_created_idx USING btree (created_at DESC)
- INDEX masterclass_regs_email_idx USING btree (email)
- UNIQUE INDEX masterclass_regs_pkey USING btree (id)

### leaders.notification_templates

| column | type | nullable | default |
|---|---|---|---|
| key | text | NOT NULL |  |
| trigger_label | text | NOT NULL |  |
| subject | text | NOT NULL |  |
| body | text | NOT NULL |  |

Constraints:
- `notification_templates_pkey` PRIMARY KEY (key)

Indexes:
- UNIQUE INDEX notification_templates_pkey USING btree (key)

### leaders.notifications

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| subject | text | NOT NULL |  |
| body | text | NOT NULL |  |
| read_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `notifications_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX notifications_pkey USING btree (id)

### leaders.profiles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL |  |
| full_name | text | NOT NULL | ''::text |
| email | text | null |  |
| country | text | null |  |
| school | text | null |  |
| programme | text | null |  |
| interests | text[] | NOT NULL | '{}'::text[] |
| other_interest | text | null |  |
| goal | text | null |  |
| ambassador_code | text | null |  |
| cohort_id | uuid | null |  |
| consent_confirmed | boolean | NOT NULL | false |
| induction_complete | boolean | NOT NULL | false |
| points | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| path | text | null |  |
| project_line | text | null |  |
| grade | text | null |  |
| city | text | null |  |
| channels | text[] | NOT NULL | '{}'::text[] |
| onboarding_complete | boolean | NOT NULL | false |
| started_at | timestamp with time zone | null |  |
| tracking_link | text | null |  |
| lor_status | text | null |  |
| certificate_issued | boolean | NOT NULL | false |
| staff_notes | text | null |  |
| cohort | text | null |  |
| certificate_path | text | null |  |
| badge_path | text | null |  |
| lor_path | text | null |  |
| must_change_password | boolean | NOT NULL | false |
| account_status | text | NOT NULL | 'active'::text |
| created_by | uuid | null |  |
| credentials_sent_at | timestamp with time zone | null |  |
| phone | text | null |  |
| dob | text | null |  |
| parent_name | text | null |  |
| parent_phone | text | null |  |
| parent_email | text | null |  |
| path_reason | text | null |  |
| applied_at | timestamp with time zone | null |  |
| temp_password | text | null |  |
| utm_slug | text | null |  |
| referral_code | text | null |  |

Constraints:
- `profiles_cohort_id_fkey` FOREIGN KEY (cohort_id) REFERENCES leaders.cohorts(id) ON DELETE SET NULL
- `profiles_pkey` PRIMARY KEY (id)
- `profiles_ambassador_code_key` UNIQUE (ambassador_code)

Indexes:
- UNIQUE INDEX profiles_ambassador_code_key USING btree (ambassador_code)
- UNIQUE INDEX profiles_pkey USING btree (id)
- UNIQUE INDEX profiles_referral_code_key USING btree (referral_code)
- UNIQUE INDEX profiles_utm_slug_key USING btree (utm_slug)

### leaders.referral_events

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| ambassador_code | text | NOT NULL |  |
| kind | text | NOT NULL |  |
| detail | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `referral_events_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX referral_events_pkey USING btree (id)

### leaders.sl_path_variants

| column | type | nullable | default |
|---|---|---|---|
| path | text | NOT NULL |  |
| helper | text | NOT NULL | ''::text |
| deliverable | text | NOT NULL | ''::text |
| launch | text | NOT NULL | ''::text |
| metric | text | NOT NULL | ''::text |

Constraints:
- `sl_path_variants_pkey` PRIMARY KEY (path)

Indexes:
- UNIQUE INDEX sl_path_variants_pkey USING btree (path)

### leaders.sl_resources

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| title | text | NOT NULL |  |
| description | text | NOT NULL | ''::text |
| url | text | null |  |
| kind | text | NOT NULL | 'template'::text |
| paths | text[] | NOT NULL | '{}'::text[] |
| position | integer | NOT NULL | 0 |
| levels | integer[] | NOT NULL | '{}'::integer[] |
| cohorts | text[] | NOT NULL | '{}'::text[] |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `sl_resources_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX sl_resources_pkey USING btree (id)

### leaders.sl_scores

| column | type | nullable | default |
|---|---|---|---|
| student_id | uuid | NOT NULL |  |
| consistency | integer | null |  |
| initiative | integer | null |  |
| impact | integer | null |  |
| reflection | integer | null |  |
| notes | text | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `sl_scores_pkey` PRIMARY KEY (student_id)

Indexes:
- UNIQUE INDEX sl_scores_pkey USING btree (student_id)

### leaders.sl_session_attendance

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| student_id | uuid | NOT NULL |  |
| status | text | NOT NULL | 'present'::text |
| marked_by | uuid | null |  |
| marked_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `sl_session_attendance_session_id_fkey` FOREIGN KEY (session_id) REFERENCES leaders.sl_sessions(id) ON DELETE CASCADE
- `sl_session_attendance_pkey` PRIMARY KEY (id)
- `sl_session_attendance_session_id_student_id_key` UNIQUE (session_id, student_id)

Indexes:
- UNIQUE INDEX sl_session_attendance_pkey USING btree (id)
- UNIQUE INDEX sl_session_attendance_session_id_student_id_key USING btree (session_id, student_id)
- INDEX sl_session_attendance_student_idx USING btree (student_id)

### leaders.sl_sessions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| title | text | NOT NULL |  |
| starts_at | timestamp with time zone | null |  |
| note | text | null |  |
| join_url | text | null |  |
| cohort | text | null |  |
| path | text | null |  |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| recording_url | text | null |  |

Constraints:
- `sl_sessions_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX sl_sessions_pkey USING btree (id)

### leaders.sl_settings

| column | type | nullable | default |
|---|---|---|---|
| key | text | NOT NULL |  |
| value | text | NOT NULL | ''::text |

Constraints:
- `sl_settings_pkey` PRIMARY KEY (key)

Indexes:
- UNIQUE INDEX sl_settings_pkey USING btree (key)

### leaders.sl_task_drafts

| column | type | nullable | default |
|---|---|---|---|
| student_id | uuid | NOT NULL |  |
| task_id | uuid | NOT NULL |  |
| answers | jsonb | NOT NULL | '{}'::jsonb |
| files | jsonb | NOT NULL | '[]'::jsonb |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `sl_task_drafts_task_id_fkey` FOREIGN KEY (task_id) REFERENCES leaders.sl_tasks(id) ON DELETE CASCADE
- `sl_task_drafts_pkey` PRIMARY KEY (student_id, task_id)

Indexes:
- UNIQUE INDEX sl_task_drafts_pkey USING btree (student_id, task_id)

### leaders.sl_task_progress

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_id | uuid | NOT NULL |  |
| task_id | uuid | NOT NULL |  |
| level | integer | NOT NULL |  |
| day | integer | NOT NULL |  |
| state | text | NOT NULL | 'submitted'::text |
| answers | jsonb | NOT NULL | '{}'::jsonb |
| files | jsonb | NOT NULL | '[]'::jsonb |
| submitted_at | timestamp with time zone | null |  |
| approved_at | timestamp with time zone | null |  |
| admin_comment | text | null |  |
| internal_note | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| late | boolean | NOT NULL | false |

Constraints:
- `sl_task_progress_task_id_fkey` FOREIGN KEY (task_id) REFERENCES leaders.sl_tasks(id) ON DELETE CASCADE
- `sl_task_progress_pkey` PRIMARY KEY (id)
- `sl_task_progress_student_id_task_id_key` UNIQUE (student_id, task_id)

Indexes:
- UNIQUE INDEX sl_task_progress_pkey USING btree (id)
- UNIQUE INDEX sl_task_progress_student_id_task_id_key USING btree (student_id, task_id)
- INDEX sl_task_progress_student_idx USING btree (student_id)

### leaders.sl_tasks

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| level | integer | NOT NULL |  |
| day | integer | NOT NULL |  |
| title | text | NOT NULL |  |
| task_copy | text | NOT NULL |  |
| checkpoint | text | NOT NULL |  |
| fields | jsonb | NOT NULL | '[]'::jsonb |
| approval_gate | boolean | NOT NULL | false |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `sl_tasks_pkey` PRIMARY KEY (id)
- `sl_tasks_level_day_key` UNIQUE (level, day)

Indexes:
- UNIQUE INDEX sl_tasks_level_day_key USING btree (level, day)
- UNIQUE INDEX sl_tasks_pkey USING btree (id)

### leaders.user_roles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | leaders.app_role | NOT NULL |  |

Constraints:
- `user_roles_pkey` PRIMARY KEY (id)
- `user_roles_user_id_role_key` UNIQUE (user_id, role)

Indexes:
- UNIQUE INDEX user_roles_pkey USING btree (id)
- UNIQUE INDEX user_roles_user_id_role_key USING btree (user_id, role)
