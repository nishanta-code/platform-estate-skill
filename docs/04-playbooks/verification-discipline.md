# Playbook: verification discipline ("looks done" is not "done")

The single most repeated lesson of the year. Every item here was learned by shipping something that passed review and failed in reality.

## The ladder — each rung is necessary, none is sufficient
1. `tsc --noEmit` clean.
2. `vite build` / `bun run build` clean — and `npm install --dry-run` if Lovable's pipeline might use npm (EOVERRIDE).
3. Unit tests (`vitest`) and, where present, Playwright/e2e (`Ops Tool`, `Growth Partner Portal`, `Learner Portal`, `research-platform/e2e`).
4. Lint on **changed files only** (CRLF noise elsewhere).
5. **Live DB verification** for anything touching Supabase: `pg_policies`, `pg_trigger`, `has_function_privilege`, `role_table_grants`, `supabase migration list --remote`, `get_advisors`.
6. **Live round-trip**: write → re-query → cleanup through the real authenticated client (not service role). Impersonate a JWT in SQL when needed: `select set_config('request.jwt.claim.sub','<uuid>',true);`.
7. **Real login** in a browser (or Playwright) for anything auth/RLS/Realtime. Two production-class bugs (missing GRANTs, Realtime topic collision) were invisible to rungs 1–6.
8. **Deployed artefact check**: fetch the deployed edge function body (`get_edge_function`), confirm Lovable `latest_commit_sha` equals your push, open the preview URL. `deploy_project`'s "pending" is not success.
9. **Responsive pass** on phone width for UI work (the house checklist in `docs/03-frontend/responsive-and-ui-house-rules.md`); when the user reports one clipped toolbar, re-sweep the whole bug class.
10. **Headless visual verification** for animated/3D pages: desktop + mobile × motion/reduced-motion, zero console errors, zero horizontal overflow, WebGL context present, screenshots (Growth Partner Portal homepage pattern, scripts deleted after use).

## Independent re-verification
- Re-verify a scanner's or another agent's "fixed" directly against the live system.
- Re-verify the memory/doc claim against the file it cites (`verify it still exists before recommending`).
- When a count looks right but behaviour is wrong, suspect silent failure: RLS-no-policy returns `[]`; CHECK violations roll back whole transactions; fire-and-forget sync logs success.

## Reporting
State what was verified and how, what was not, and what was left out. "Pushed to `feature/x`, not merged" vs "merged and live at commit `abc`" are different claims; the owner has explicitly asked for safe go-live after all checks on some work and immediate deploy on other work — never assume which.
