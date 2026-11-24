# The shared catalogue, programme codes and slugs

Sources: `application-portal/docs/CATALOGUE-SOP.md` (v2, authoritative), `application-portal/docs/SLUG-CONTRACT.md`, `growth-partner-portal/docs/LWL_Platform_Architecture_Master.md`, `growth-partner-portal/docs/shared_architecture_knowledge.md`, `learner-portal/mem/tech/architecture/*`, live schema on `<ESTATE_REF>`.

This is the contract that lets ten programme brands, one application portal, one LMS and a finance console agree on what a programme *is*. Get this right and everything else (checkout, enrollment, attribution, finance codes) lines up; get it wrong and you get the incidents listed at the bottom.

---

## 1. The four shared tables (`public`, on the shared estate)

| Table | Holds | One row = | Who writes |
|---|---|---|---|
| `all_partners` | the platforms / organisations | one partner | Console only |
| `all_programs` | the evergreen programme | one programme, owned by one partner | Console only (`central-admin-v1`) |
| `all_cohorts` | each intake of a programme | one dated intake | Console only |
| `all_masterclasses` | single webinars / info sessions | one session | each platform maintains rows carrying its own `partner_id`; clients read the `all_masterclasses_public` view (no `join_url`/`passcode`) |
| `all_mentors` | shared mentor identity (seeded from the Learning Portal) | one mentor | restricted; every platform reads |

Relationships: `all_partners.id --< all_programs.partner_id --< all_cohorts.program_id`; `all_cohorts.program_code -- payment records`. Partner is always reached **through the programme**. Never store `partner_id` on a cohort; never store `partner_name`; never store a `platform_key` (platform derives from partner, which derives from programme).

Every platform schema carries **restricted views** onto these tables (`<schema>.all_programs`, `.all_cohorts`, `.all_masterclasses`, `.all_masterclasses_public`, `.all_mentors`, sometimes `.all_partners`). Selling platforms filter to their own `partner_id`; cross-cutting tools (finance, lms, ops/marketplace, the Student Website) are unfiltered. `join_url` and `passcode` never appear in any view, API response or chatbot answer.

Live counts 2026-11-24: `all_partners` 0 rows visible to the metadata query (the master doc of 2026-10-14 reported 10 partners / 110 programmes / 173 cohorts — re-verify before trusting either number), `all_programs` 2, `all_cohorts` 11, `all_masterclasses` 1. The LMS memory note lists the full column sets (all_partners 15 cols, all_programs 24 cols pre-v2, all_cohorts 13 cols pre-v2, all_mentors 8 cols).

### Partners (2026-10-14 snapshot from the master doc)

| Partner slug | Website |
|---|---|
| `example-group` | example-group.com |
| `fellowship-group` | fellowship.example.com |
| `premed-group` | premed.example.com |
| `finedu-group` | finedu.example.com |
| `research-group` | research.example.com |
| `prelaw-group` | prelaw.example.com |
| `testprep-group` | testprep.example.com |
| `purpose-group` | purpose.example.com |
| `founders-group` | founders.example.com |
| `challenge-group` | challenge.example.com |

---

## 2. Golden rules (Catalogue SOP v2)

1. One programme = one row in `all_programs`. One intake = one row in `all_cohorts`. Never clone a programme because the dates changed.
2. Edit a cohort in place. A cohort that stops running is **deleted**, never flagged. Same for a masterclass. (v2 dropped `is_cancelled`, `seats_filled`, `recording_url`; platform views expose placeholders only.)
3. Descriptions are evergreen: no dates, months, years, deadlines, seat counts or fees. Those belong to the cohort.
4. Prices in whole currency units (`300`, never `300.00` or `30000`). Check `price_currency`; not everything is USD. Price is per programme; there is no per-cohort override.
5. Phone numbers digits-only everywhere (`917749990239`).
6. No em dashes in user-facing copy. The word "elite" is prohibited.
7. Programme codes are minted only in the Console. Never hand-write one.
8. Anything derivable is derived at display time, never stored (ages, seats left, live/past status, apply URLs for Group programmes).
9. Never invent missing data. A blank field is reported, not guessed. (An Fellowship Platform agent once wrote central programmes with a guessed 399 price; it had to be reverted. Deactivate, do not delete, when reverting.)
10. `is_active` means the programme exists, **not** that it is open or listed. Status is derived from dates.
11. A name collision is not proof of duplication: two "Pre-Med Platform" programmes exist and are different products. Ask before merging.
12. Scaffolding signature: null price + zero applications + rows created in one batch. Ask the platform before treating such a cohort as real.

---

## 3. `all_programs` columns (v2)

| Column | Owner | Required | Format / notes |
|---|---|---|---|
| `id` | system | auto | uuid |
| `partner_id` | Console | yes | uuid of `all_partners`; drives which domain may publish links |
| `program_name` | partner | yes | Title Case, no dates, no cohort words; external partner listed on someone else's portal: `<Partner> - <Programme>` |
| `pay_slug` | Console | yes | lowercase-hyphen, globally unique — the master identifier (section 5) |
| `short_description` | partner | yes | ~220 chars, one idea, evergreen |
| `description` | partner | yes | 60–110 words, evergreen |
| `categories` | Console | yes | **single taxonomy array**: type, subject, audience — `["Fellowship","Finance","High school students"]` |
| `format` | partner | yes | exactly `Online`, `In Person` or `Hybrid` |
| `location` | partner | physical only | `City, Country` |
| `duration` | partner | yes | `N days` or `Ongoing` (1 weekend = 2 days, 1 week = 7, 1 month = 30) |
| `eligibility_grades` | partner | yes | `Grades X-Y`, X/Y in 5..12; ages derived (grade 5 = 10, +1 per grade, upper = upper grade + 1) |
| `sticker_price` | partner | yes | whole units |
| `early_bird_price` | partner | optional | whole units; only when a genuine early rate exists, else null; **no date-based expiry anywhere — an admin clears it to end the offer** |
| `price_currency` | partner | yes | ISO code |
| `cover_image_url`, `program_url` | partner | optional | absolute https |
| `apply_url` | derived / partner | yes | Group programmes: derived `https://apply.example-group.com/apply/{pay_slug}`; external partners author their own |
| `is_active` | Console | yes | boolean (auto-draft rule, section 6) |

Dropped on 2026-09-10: `tags`, `target_categories`, `age_min`, `age_max`, `product_match_keywords` (views expose empty placeholders).

## 4. `all_cohorts` columns (v2)

| Column | Owner | Required | Format / notes |
|---|---|---|---|
| `program_id` | Console | yes | uuid of `all_programs` |
| `cohort_key` | Console | yes | `{pay_slug}-{mmm}{yy}` e.g. `global-debating-championship-nov26` (older docs: `{program}_{monthYY}` lowercase underscore — both forms exist in data; ~557 applications reference them as text) |
| `program_code` | Console | yes | `XX-ORG-PROG-DDMMYY[-SITE]` uppercase |
| `cohort_label` | Console | yes | the canonical student-facing name (`Cohort 2 - Aug 2026`) |
| `start_date`, `end_date`, `application_deadline` | partner | yes | ISO dates; deadline ≤ start ≤ end; never "TBA" |
| `seats_total` | partner | optional | published capacity; seats left are counted live from applications |

Dropped in v2: `seats_filled`, `is_cancelled`. City/country come from the programme's `location`.

## 5. `all_masterclasses` columns (v2)

`partner_id`, `program_id?`, `cohort_id?`, `slug` (`{series-slug}-s{n}` or `{topic}-{mmm}{yy}`), `title` (Title Case, no dates), `description` (40–90 words), `series`, `session_number`, `audience` (`general|students|parents`), `scheduled_at` (full ISO **with UTC offset**), `duration_minutes` (default 60), `display_timezone` (IANA), `host_mentor_id` (**required for every upcoming session**), `host_name`, `learning_outcomes` (3–5 strings), `join_url` (real meeting link, required before publish), `passcode`, `apply_url`, `capacity`, `is_published`, `page_copy` (JSON). Dropped: `is_cancelled`, `recording_url`.

The four fields platforms keep leaving blank: `host_mentor_id`, `host_name`, `join_url`, `passcode`. A session may be a draft with these blank; it must not be published with any blank. The Console's masterclass readiness report lists offenders.

Derive, never store: local time, upcoming/live/past, end time, "Session 2 of 6", seats left, eligible ages.

Registrations: every platform keeps its own regs table (`public.fellowship_masterclass_regs`, `testprep_`, `prelaw_`, `finedu_`, `research_`, `premed_`, `masterclass_regs`) storing free-text session labels. The LMS unions them in `lms.mc_reg_base` and matches labels to the catalogue via `lms.mc_norm()` (slug equality or title containment); unmatched labels are expected until the platform publishes its catalogue rows.

---

## 6. Programme codes

```
CATEGORY - ORG - PROGRAM - DDMMYY [ - SCHOOL ]
e.g. CO-UNIV-DEBATE-151126
```

- Category: `CA` camp, `CF` conference, `CO` competition, `IP` in person, `ZL` Zoom-led.
- ORG = the organisation supplying the mentors; PROGRAM = programme key; DDMMYY = cohort `start_date` (must match; read with `split_part(program_code,'-',4)`, never `right(program_code,6)`); optional 4-letter SCHOOL code only for a physical venue.
- Minted **only** in Console into `finance.program_codes` (formerly `sales.program_codes`; 299 codes existed on 2026-10-14). Nobody invents, guesses or infers one.
- Every PaymentIntent carries `metadata.program_code`; webhooks route on it first. A payment without it is unattributable.
- `finance.codes_dropdown` names each segment; `public.programme_codes` on the data project mirrors the registry via `migration.sync_programme_codes()`.

---

## 7. Slugs and URLs (Slug Contract)

| Slug | Table | Format | Used for |
|---|---|---|---|
| `partner_slug` | `all_partners.partner_slug` | lowercase-hyphen | identifies the platform; picks the publishing domain |
| `pay_slug` | `all_programs.pay_slug` | lowercase-hyphen, unique across all partners | drives BOTH `/apply/{pay_slug}` and `/pay/{pay_slug}` |
| `cohort_key` | `all_cohorts.cohort_key` | `{pay_slug}-{mmm}{yy}` | `?cohort=` deep link |
| `program_code` | `all_cohorts.program_code` | uppercase code | finance key; also accepted as inbound apply identifier |
| `link_slug` | `portal.payment_links.slug` | lowercase-hyphen | custom priced offers `/pay/{link_slug}`; promo codes never apply |

How to make a pay slug: lowercase; spaces and `&`/`/` → single hyphen; strip punctuation/accents; drop filler words only if still unique; remove year/month/cohort tokens (slugs outlive intakes); 2–4 words under ~40 chars; keep organiser prefix when disambiguating (`univ-`, `mun-`); verify uniqueness across ALL partners. Renames keep the old slug as an alias; never reuse a retired slug.

URL rules:
1. Apply: `https://apply.example-group.com/apply/{pay_slug}`
2. Pay: `https://apply.example-group.com/pay/{pay_slug}`
3. Custom offer: `.../pay/{link_slug}`
4. Optional params: `?cohort={cohort_key|program_code}`, `?email=`, `utm_source`/`utm_medium`/`utm_campaign` (captured on both application and payment).
5. Publish a programme's links only on the owning partner's domain (`partner_slug`). Deep-link into Group programmes only when `partner_slug = 'example-group'`; other partners use their `apply_url`.
6. Sibling platforms' own pay pages: `{all_partners.website}/pay/{all_programs.pay_slug}`, fallback `all_programs.apply_url`. Never hardcode a domain.
7. Backward compatibility already handled: `/apply?ref={slug}` and `/apply?form={slug}` redirect to `/apply/{slug}`; `?ref=` may carry an uppercase `program_code`.

LMS outbound apply links always set `utm_campaign=lms`, `utm_source=past-paid-students`, `utm_medium=lms-portal-direct` (overriding those three, preserving the rest).

---

## 8. Lifecycle and automation

- Expired intakes drop out automatically: a nightly sweep removes links whose `program_code` start date has passed.
- Auto-draft: when the sweep leaves a live form with no upcoming intake, the form is set back to draft. Nothing is ever switched live automatically.
- External listings carry the partner's own apply page — no local payment plan, promo setup or form schema.
- Normalisation ("Normalize catalogue", "Masterclass clean-up" in the Console) is deterministic and idempotent: canonical `format`, `duration`, `Grades X-Y`, merged `categories`, derived apply URLs. It never rewrites descriptions, prices, dates, codes, partner URLs, and never deletes.

---

## 9. Checklists

**New programme**: not already present; Title Case date-free name; `pay_slug` unique; `format` valid; `duration` converted; grades from dropdown; `categories` type+subject+audience; evergreen descriptions in length; price whole units + currency; `location` if physical; apply URL derived (Group) or verified; first cohort with ISO dates, deadline, seats, Console-minted code; `/apply/{pay_slug}` and `/pay/{pay_slug}` load.

**New cohort**: reuse programme row; `cohort_key` format; dates valid; `seats_total` if published; description untouched; delete previous intake if it won't run again.

**New masterclass**: unique slug; series matches siblings; `scheduled_at` with offset + `display_timezone` + duration; host id AND name; real `join_url`; passcode if any; 3–5 outcomes; live `apply_url`; capacity if published; publish last.

---

## 10. Incidents that produced these rules

| What happened | Rule |
|---|---|
| Fellowship Platform's agent wrote central programmes with a guessed 399 price | Never write a central row without confirmed values; deactivate rather than delete when reverting |
| Zoom links and passcodes leaked through four platform views, two to `anon` | No secret column in any view; verify by column list |
| Test-Prep Platform's agent ran a full migration when told to assess | Assessment and migration are separate documents |
| Portal Form Builder read programmes from a 151 KB hardcoded `studentPrograms.ts` | Read the catalogue live; never cache slugs |
| Fellowship Platform and Pre-Med Platform ended up with more programmes than agreed | Reconcile catalogue rows against decisions on record before trusting |
| Founders Platform's loose payment matching emailed strangers | Unattributed payments go to manual review; never auto-match on amount/time |
