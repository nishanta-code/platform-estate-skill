# Brain project: auth summary

Live snapshot taken 2026-11-24 from <DATA_REF>. Counts only; no e-mail addresses, names or ids were read.

## auth.users

| metric | value |
|---|---|
| users total | 32 |
| email confirmed | 32 (100 %) |
| with phone | 0 |
| is_sso_user | 0 |
| is_anonymous | 0 |
| banned (banned_until > now) | 0 |
| soft-deleted | 0 |
| never signed in (last_sign_in_at IS NULL) | 23 |
| signed in within last 30 days | 7 |
| signed in within last 7 days | 1 |
| by app provider (raw_app_meta_data.provider) | email: 32 |
| by role / aud | authenticated: 32 / authenticated: 32 |
| raw_user_meta_data key sets | email_verified + full_name: 29; email_verified only: 2; email + email_verified + phone_verified + sub: 1 (shape produced by a dashboard "create user") |

### Users created per month

| month | users |
|---|---|
| 2026-08 | 15 |
| 2026-09 | 2 |
| 2026-10 | 1 |
| 2026-11 | 14 |

The August batch is the original staff load (research platform / spine); the November batch matches the group platform roll-out (invitations via invite-user / accept-invite edge functions and the auth.users AFTER INSERT triggers).

## auth.identities

| metric | value |
|---|---|
| identities total | 32 |
| by provider | email: 32 |

No OAuth/social, SAML or SSO identities. auth.sso_providers 0, auth.saml_providers 0, auth.oauth_clients 0, auth.flow_state 0 rows.

## Sessions and tokens

| metric | value |
|---|---|
| auth.sessions | 14 (all aal1) |
| sessions created in last 7 days | 3 |
| auth.refresh_tokens total | 177 |
| refresh tokens not revoked | 14 |
| auth.mfa_factors | 0 (0 verified) |
| auth.one_time_tokens | 0 |
| auth.audit_log_entries | 0 (table empty; auth audit log is not being persisted to the DB table) |

## How auth connects to the application schemas

- Two triggers on auth.users (see brain-project-triggers.md): `lwl_on_auth_user_created` -> group.handle_new_user_if_invited() and `on_auth_user_created` -> public.accept_staff_invitation(). A sign-up only becomes a staff/user-profile row if a matching invitation exists; there is no self-service staff creation (public.staff comment: "The absence of a row is what separates a parent or student from an admin console -- they share the same Supabase Auth pool").
- Profile tables keyed on auth.users.id: group.user_profiles (role group.app_role, 24 rows have Gmail accounts linked in group.user_gmail_accounts), leads.user_profiles (FK ON DELETE CASCADE), public.research_staff_profiles.user_id (FK ON DELETE CASCADE, UNIQUE), public.staff (+ staff_grants, staff_scopes, staff_role_assignments, staff_targets).
- RLS helpers resolve the caller from auth.uid(): group.is_admin_or_owner / is_active_staff / has_role / sop_actor / my_assigned_meeting_ids; leads.is_admin_or_owner / is_assigned_to_meeting; public.auth_staff_id / has_scope_all / has_grant / can_see_person / research_role.
- Edge functions that touch auth: invite-user, accept-invite, admin-manage-user (verify_jwt=true), send-account-credentials (verify_jwt=true), auth-email (custom auth e-mail hook, verify_jwt=false).
