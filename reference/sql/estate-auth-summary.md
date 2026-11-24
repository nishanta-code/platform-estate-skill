# Shared estate: auth summary

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate). Aggregate counts only; no e-mails, names or ids were read. `auth.config` is not queryable and is skipped.

This single `auth.users` table is the shared identity store for every Group platform on this project (Ops Tool, Application Portal / portal, Pre-Med, Pre-Law, Financial Literacy, Learner Portal / lms, Affiliate, Test-Prep, Research, Fellowship, Student Leaders). Platform membership is **not** recorded in `raw_app_meta_data` (no `platform` key on any user); it lives in each schema's `user_roles` / `user_profiles` / `profiles` table.

## Headline counts

| metric | value |
|---|---|
| auth.users total | 4,228 |
| users with email_confirmed_at | 4,228 (100%) |
| banned users | 0 |
| anonymous users (is_anonymous) | 0 |
| users with any last_sign_in_at | 1,104 (26%) |
| users signed in within last 30 days | 614 |
| auth.identities total | 4,228 (exactly one per user) |
| auth.sessions total | 2,006 |
| sessions refreshed/created in last 7 days | 395 |
| auth.refresh_tokens total | 22,238 (2,003 not revoked) |
| auth.mfa_factors | 0 |
| auth.sso_providers | 0 |
| auth.audit_log_entries | 0 |
| auth.one_time_tokens | 9 |

Reading: ~3,100 of the 4,228 accounts have never signed in - consistent with the LMS "login rollout" (bulk-created learner accounts that are activated in batches via `lms.login_rollout_queue`, 4,224 rows). `audit_log_entries = 0` means GoTrue audit logging is off or purged.

## Provider breakdown

| dimension | key | users |
|---|---|---|
| raw_app_meta_data->>'provider' | email | 4,228 |
| raw_app_meta_data->>'providers' | ["email"] | 4,228 |
| auth.identities.provider | email | 4,228 |

Email/password is the only provider in use. No Google/Apple/phone/magic-link-only identities, no SSO. (Google One Tap "captured leads" in premed/testprep/research are lead rows, not auth identities.)

## Sign-ups by month (created_at, UTC)

| month | new users |
|---|---|
| 2026-04 | 7 |
| 2026-06 | 1 |
| 2026-07 | 2 |
| 2026-08 | 2 |
| 2026-09 | 84 |
| 2026-10 | 3,165 |
| 2026-11 (to 24th) | 967 |

The October spike (3,165) is the bulk import of existing learners into the Learner Portal; organic growth since is ~1,000/month.

## raw_user_meta_data keys (how many users carry each key)

| key | users |
|---|---|
| email_verified | 4,228 |
| full_name | 4,227 |
| phone | 4,187 |
| email | 7 |
| sub | 7 |
| phone_verified | 7 |
| first_name | 1 |
| last_name | 1 |

The 7 users with `email`/`sub`/`phone_verified` keys were created through the standard GoTrue signup path (those keys are what the client SDK writes); the other ~4,220 were created by the admin API (bulk rollout) with just `full_name` + `phone`. Only one account uses `first_name`/`last_name` - shared-auth registration should standardise on `full_name`.

## Foreign keys into auth.users from business schemas

From the column dumps: `premed.cohort_settings.mentor_id`, `premed.goal_reviews.user_id/mentor_id`, `premed.learning_assessments.user_id`, `premed.live_class_rsvps.user_id`, `premed.mmi_coach_sessions.user_id`, `premed.mmi_practice_attempts.user_id`, `premed.parent_reports.user_id`, `premed.saved_universities.student_id`, `premed.session_attendance.user_id`, `premed.student_goals.user_id`, `premed.student_notifications.user_id`, `premed.student_university_applications.student_id`, `premed.user_profiles.user_id` all `REFERENCES auth.users(id) ON DELETE CASCADE/SET NULL`. Other schemas (lms, portal, affiliate, ops, research, leaders, finedu, prelaw, testprep, fellowship) store `user_id`/`auth_user_id` as plain uuid columns **without** a foreign key, so deleting an auth user does not cascade there.

No trigger exists on `auth.users` (see `shared-estate-triggers.md`).
