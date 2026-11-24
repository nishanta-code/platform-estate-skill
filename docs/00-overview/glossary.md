# Glossary

Terms, codes and names used across the estate. If a word appears in a prompt and is not here, check `docs/00-overview/estate-map.md`.

## Organisations, brands, platforms

| Term | Meaning |
|---|---|
| **Group** | the Group — the parent organisation (example-group.com) |
| **Application Portal** | apply.example-group.com — central forms, checkout and admin. Repo "Application Portal", schema `portal` |
| **Student Website** | example-group.com — discovery/SEO site. Repo "Marketing Site" (static export) + the TanStack rebuild project |
| **Learner Portal / LMS** | The shared learning portal for paid students of every brand. Repo "Learner Portal", schema `lms` |
| **Console** | The finance/admin console. Schema `finance` (formerly `sales`). Mints every programme code; the only writer of the shared catalogue |
| **Ops Tool** | Internal staff/ops tool (AI calling campaigns, school outreach). Repo remote is still `marketplace` from its previous identity |
| **Marketplace** | The former partner marketplace that became Ops Tool; schema `marketplace` is its legacy |
| **Growth Partner Programme** | The affiliate / school-ambassador / student-referral programme. Repo "Growth Partner Portal", schema `affiliate` |
| **Sales Command** | Internal sales platform: Calendly/Zoom meetings, AI analysis, follow-ups, Gmail + WhatsApp agents. Schema `sales` on the data project |
| **Brain** | The research-to-revenue data platform (`<BRAIN_REF>`), accessed only through `brain_api_*` RPCs |
| **Challenge Platform / Global Future Innovators Challenge** | challenge.example.com — the repo folder is "Challenge Platform" |
| **UNIV / UNIV / MUN** | Programme families (university-college-student-mentored championships, model-UN style events) appearing in programme codes and slugs |
| **Founders Platform** | Threshold Entrepreneurs Organization (founders.example.com), repo "Founders Platform" |
| **NPF / Meritto** | The same external email-campaign system; campaigns get a "Job Id" and performance CSVs are re-downloaded after ~48h and ~1 week |
| **Instantly** | External cold-email platform whose webhooks feed Brain outreach events |

## Identifiers

| Term | Format | Where it lives |
|---|---|---|
| `partner_slug` | lowercase-hyphen (`example-group`) | `public.all_partners` |
| `pay_slug` | lowercase-hyphen, globally unique (`geneva-camp`) | `public.all_programs` — drives `/apply/{pay_slug}` and `/pay/{pay_slug}` |
| `cohort_key` | `{pay_slug}-{mmm}{yy}` (`geneva-camp-dec26`); older rows `{program}_{monthYY}` | `public.all_cohorts` |
| `program_code` | `XX-ORG-PROG-DDMMYY[-SITE]` (`CO-UNIV-DEBATE-151126`) | `public.all_cohorts`, `finance.program_codes`; mandatory in Stripe metadata |
| `link_slug` | lowercase-hyphen | `portal.payment_links.slug` — custom priced offers, promos never apply |
| `submission_token` | UUID | anonymous checkout/intake authorisation on public edge functions |
| `ContactCode` | `^CT[A-Z0-9]{10}$` | Brain research contacts; carried as `utm_source` in the three-UTM scheme |
| `CampaignCode` | e.g. `C7SSJY79CYIY9Y` | Brain campaigns; carried as `utm_campaign` |
| `source_code` | SYSTEM-segmented minted code | data project `public.source_codes`; every message must map to one with an owner |

## Code segments

- Programme-code **category**: `CA` camp, `CF` conference, `CO` competition, `IP` in person, `ZL` Zoom-led.
- The `DDMMYY` segment must equal the cohort's `start_date`; read with `split_part(program_code,'-',4)`.

## Technical terms

| Term | Meaning here |
|---|---|
| **Shared estate** | Supabase project `<ESTATE_REF>`: one `auth.users`, one schema per platform, catalogue in `public` |
| **Data project** | Supabase project `<DATA_REF>`: schemas `sales`, `leads`, `public.research_*`, `brain`, `revenue_brain`, `attribution`, `quarantine`, `staging`, `migration` |
| **Own-project auth split** | A platform authenticates users on its own small Supabase project while its server code reads/writes its schema on the shared estate with a service-role key |
| **`centralAdmin()` / `centralShared(schema)`** | The two service-role client factories in `supabase/functions/_shared/db.ts` (Application Portal pattern) |
| **Restricted catalogue views** | Per-schema views of `all_programs`/`all_cohorts`/`all_masterclasses(_public)`/`all_mentors`, filtered to the platform's `partner_id` for selling platforms, unfiltered for cross-cutting tools, never exposing `join_url`/`passcode` |
| **Auto-draft** | The nightly sweep that sets a live form back to draft when its programme has no upcoming intake |
| **Vault** | The LMS credit/points system (`lms.vault_lots`, `vault_ledger`, `vault_vouchers`) usable at checkout |
| **Login rollout** | The throttled queue (`lms.login_rollout_queue`) that onboards pre-existing students in bounded batches |
| **Reclaim** | Forcing a password reset + session revocation on a self-registered account before attaching it to a real programme |
| **Side-effects queue** | `premed.side_effects_queue` — logged, retryable replacement for fire-and-forget triggers |
| **Unmatched payment** | A payment whose `program_code` does not resolve; goes to `<schema>.unmatched_payments` for manual review, never auto-matched |
| **Three-UTM scheme** | Brain's attribution: `utm_campaign` = CampaignCode, `utm_medium` = source slug, `utm_source` = ContactCode; everything else resolved via DB relationships |
| **`brain_api_*`** | The `SECURITY DEFINER` RPC surface that is the only working way to read Brain data |
| **Lovable** | The AI app builder that owns each project's sandbox, migrations, generated files, AI gateway and secrets; its agent can push to the same GitHub branch |
| **AGENTS.md** | Per-repo rules file; the `LOVABLE:BEGIN…END` block marks what Lovable manages, below it are the project's own rules |
| **roadmap.md** | Per-repo running task list with done markers |
| **mem/** | The LMS's structured project-memory notes (`index.md` + `features/`, `style/`, `tech/`) |
