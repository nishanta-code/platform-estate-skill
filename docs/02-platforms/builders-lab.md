# Builders Lab (Builders Lab) — public programme website

> Knowledge-skill reference for the `Builders Lab` repo. Written read-only from the source on 2026-11-24.
> Secret **values** are never reproduced here; env-var and secret **names** are, and Supabase project refs are.

---

## 1. Identity

| Item | Value | Evidence |
| --- | --- | --- |
| Local folder | `D:\the Group\Vs Code Studio\Builders Lab` | — |
| GitHub remote | `https://github.com/<org>/<repo>` (origin, fetch+push) | `git remote -v` |
| Branch | `main` | `git branch --show-current` |
| Commits | **2055** | `git rev-list --count HEAD` |
| Date range | **2026-08-20 → 2026-11-24** | first/last `git log --date=short` |
| Lovable project name | "Billion founders Lab aug 2026" (README H1); Lovable project id `<lovable-project-id>` (README editor link, every `src/assets/*.asset.json` `project_id`) | `README.md`, `src/assets/constellation-mark.png.asset.json` |
| Lovable preview / "live app" | `https://builders-preview.lovable.app` | `README.md` |
| Lovable template | `tanstack_start_ts_current` rev `tanstack_start_ts_current-b3e81c491308` | `.lovable/project.json`, first commit `0f401e5` |
| Public canonical domain | **`https://builders.example.com`** — hard-coded as `SITE_ORIGIN` | `src/lib/seo.ts`, `public/robots.txt` (`Sitemap: https://builders.example.com/sitemap.xml`), email `Message-ID` domain in `src/lib/masterclass-mail.server.ts` |
| Public contact address | `hello@builders.example.com` (fallback copy on all three apply forms) | `src/routes/competition.apply.tsx`, `access.apply.tsx`, `catalyst.apply.tsx` |
| Supabase project ref | **`<BUILDERS_REF>`** (its own Lovable Cloud project; schema `public`) | `supabase/config.toml` (`project_id = "<BUILDERS_REF>"`, the only line) |
| Env var names | `SUPABASE_PROJECT_ID`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_URL`, `VITE_SUPABASE_PROJECT_ID`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `VITE_SUPABASE_URL` (in `.env`, now git-ignored); server-only `SUPABASE_SERVICE_ROLE_KEY`, `GMAIL_SENDER_EMAIL`, `GMAIL_APP_PASSWORD`, `LOVABLE_CRON_SECRET`, `LOVABLE_CRON_SECRET_PREVIOUS`, `LOVABLE_DB_MIGRATION_URL` | `.env` (names only), `src/integrations/supabase/*.ts`, `src/lib/masterclass-mail.server.ts`, `drizzle.config.ts` |
| Framework | **TanStack Start** (`@tanstack/react-start` 1.168.60) on **React 19.2**, SSR via **Nitro** 3 (beta) with the Lovable preset `@lovable.dev/vite-tanstack-config` 2.26.5 targeting **`cloudflare-module`** | `package.json`, `vite.config.ts`, `node_modules/@lovable.dev/vite-tanstack-config/dist` |
| Router | `@tanstack/react-router` 1.170.41, file-based routes in `src/routes/`, generated `src/routeTree.gen.ts` | `src/router.tsx`, `src/routes/README.md` |
| Bundler | **Vite 8.1.6** (rolldown-based) with `overrides.rolldown = 1.2.1` | `package.json` |
| Package manager | **bun** — `bun.lock` is the lockfile of record; `package-lock.json` is git-ignored; `bunfig.toml` sets a 24 h `minimumReleaseAge` supply-chain guard | `bunfig.toml`, `.gitignore`, commit `6570233` |
| Styling | **Tailwind v4** via `@tailwindcss/vite` + `@theme inline` tokens in `src/styles.css`; shadcn/ui (new-york, slate) + Radix primitives in `src/components/ui/*`; `tw-animate-css` | `components.json`, `src/styles.css` |
| Data layer | `@supabase/supabase-js` 2.112 (anon client in browser, service-role client in server functions), `drizzle-kit`/`drizzle-orm` 0.x for the masterclass migrations, `@tanstack/react-query` provider in root | `src/integrations/supabase/*`, `drizzle/`, `src/routes/__root.tsx` |
| Email | `@portable/smtp` 0.1.0 + `@portable/sockets` 0.1.0 → Gmail SMTP over port 465 (see §4.6) | `src/lib/masterclass-mail.server.ts` |
| Animation | `gsap` 3.15 + `lenis` 1.3 — dynamically imported, scoped to `/competition` only | `src/components/innovate-fx/use-innovate-motion.ts` |
| Tests | 3 `node:test` files (`src/lib/*.test.ts`); no `test` script in `package.json` | §7.3 |
| QA scripts | 5 Python scripts in `scripts/` (`copy-qa`, `contrast-audit`, `a11y-audit`, `image-audit`, `build-docs`) | `package.json` scripts |

---

## 2. Business purpose

### 2.1 What Builders Lab is

Builders Lab is a **the Group (Group) programme brand** for social-impact entrepreneurship for school students (ages 11–18 / Grades 6–12). The site tagline is "**Build With. Not For.**" and the root `<title>` is "Builders Lab, Build With. Not For." (`src/routes/__root.tsx`). The Lovable north-star file states the purpose and metric exactly:

> **North star:** Young people build something that helps a real person earn a living.
> **Metric:** Students who submit an application to a programme each month.
> — `.lovable/north-star/north-star.md`

The homepage (`src/routes/index.tsx`) carries the mission ("No child sleeps hungry. Every child can feed themselves."), the method ("Build With. Not For."), the economics ("1 = 1. One paid seat funds one free seat.") and the proof stance ("We are new. Outcomes verified, never projected."). The copy is deliberately anti-charity: "We refuse to build for", "We refuse to create dependency", "We refuse to stop at awareness".

Relationship to the Group: the site co-brands with Group and HPAIR (university College Project for Asian and International Relations) logos (`/public/lwl-logo-new.png`, `/public/hpair-logo-new.png`, used on `/` and `/join`), every paid application CTA goes to the **Group Application Portal** (`https://apply.example-group.com/...`), the masterclass confirmation email is sent "From: the Group", and the FAQ page's first question is "What is Builders Lab, and who runs it?" (`src/routes/faq.tsx`).

### 2.2 Programme structure (as published on 2026-11-24)

| Programme | Status | Route | Facts shown on site | Entry mechanism |
| --- | --- | --- | --- | --- |
| **Innovate for Change Challenge 2026** (the "Competition", formerly "The Billion Founders Cup" → "Billion Founders Impact Challenge" → "Creating Livelihoods Challenge") | **Live, primary funnel** | `/competition` (`src/routes/competition.index.tsx`) | Grades 6–12, online, worldwide; 10 Saturdays 5 Dec 2026 → 6 Feb 2027, 11 AM–12 PM EST; must serve UN SDG 1, 2 or 8; university undergraduate mentors via HPAIR; solo or team of up to 5; **USD 80 early-bird (ends 30 Nov 2026) / USD 100 regular per participant**; top 20 / top 10 / top 3 tiers; "a share of USD 500" cash prizes; 15 %/30 %/full Group scholarships; seven live masterclasses for all, an eighth for the top 10, semi-final week 8, grand finale week 10 | External: `https://apply.example-group.com/apply/billion-founders-competition` (opened with carried UTM) |
| **Creating Livelihoods Challenge 2026** (previous competition page) | Preserved, `noindex`, not linked | `/dummy-comp` (`src/routes/dummy-comp.tsx`) | "The world's first global livelihood challenge for students in Grades 6 to 12"; same 10-week shape | Same external URL |
| **Access Fellowship** | Live | `/access`, `/access/apply` | Free six-month fellowship, 12 classes, 100 seats "Cohort One", by **NGO nomination only** (max 10 per NGO), adult supervisor in every session, paired with a real micro-business from month 3, "Starts September 2026" (schema has no start date because that date has passed) | On-site form → `public.access_applications` |
| **Catalyst Fellowship** (paid, for teen "teach-and-build" fellows) | **Paused** — both routes 302-redirect to `/` | `/catalyst`, `/catalyst/apply` (`src/routes/catalyst.*.tsx`) | Curriculum data kept in `src/data/catalyst-curriculum.ts`; fee and certification claims held `pending` in `content/verified-claims.md` | On-site form → `public.catalyst_applications` (unreachable while paused) |
| **Masterclasses** (free live Zoom sessions) | Live | `/masterclass`, `/masterclass/apply` | "Free, interactive masterclasses for students, parents and educators"; first session: "From Idea to Evidence…" by a BRDGthegap co-founder, 21 Nov 2026 1–2 PM UK | `/masterclass/apply` now 301s to `https://apply.example-group.com/forms/bfl-masterclass`; the on-site registration + Gmail confirmation server function is retained but no longer linked |
| **NGO Challenge Network** | Content page | `/ngo` | Challenge Library of briefs ("adapted from MIT Solve / XPRIZE / UNICEF / Grand Challenges" — still `pending` claims) | Enquiry form intent `ngo` |
| **Funders / partners / schools / parents** | Content pages | `/partner`, `/schools`, `/parents`, `/impact` | "Fund Access seats", school cohorts, safeguarding, impact-measurement method | Enquiry form (`/join`) |

### 2.3 Who it is for

Students (Grades 6–12), parents, schools, NGOs/community organisations, and funders. The `enquiries.intent` CHECK constraint enumerates the audience model exactly: `student | parent | school | ngo | funder | catalyst | cup`.

### 2.4 Pricing summary

- Competition: USD 80 early-bird until 30 Nov 2026 23:59 EST (`EARLY_BIRD_DEADLINE` in `src/components/innovate-fx/parts.tsx`), USD 100 after; verified in `content/verified-claims.md` ("owner confirmed competition fee update on 4 November 2026").
- Access Fellowship: USD 0, "fully funded, always" (Course schema `price: "0"`).
- Masterclasses: "Complimentary" (`masterclass_sessions.fee_label` default).
- Catalyst fee: `pending`, not published.
- Enquiry: "Cost to enquire: nothing".

---

## 3. Frontend architecture

### 3.1 Bootstrapping chain

```
src/server.ts            Cloudflare/Nitro fetch handler → wraps @tanstack/react-start/server-entry,
                         converts h3-swallowed 500 JSON into a branded HTML error page
src/start.ts             createStart(): functionMiddleware [attachAvailableAuth],
                         requestMiddleware [errorMiddleware, csrfMiddleware]
src/router.tsx           getRouter(): createRouter({ routeTree, context:{queryClient},
                         scrollRestoration, defaultPreload:"intent", preloadDelay 40ms, staleTime 60s })
src/routeTree.gen.ts     generated by @tanstack/router-plugin — never hand-edit
src/routes/__root.tsx    html/head/body shell, global meta + fonts + JSON-LD, SiteHeader/Outlet/SiteFooter,
                         404 + error boundaries, UTM capture on every navigation
```

`vite.config.ts` uses `defineConfig` from `@lovable.dev/vite-tanstack-config` (which already wires TanStack devtools, `tanstackStart`, `viteReact`, `tailwindcss`, `tsConfigPaths`, Nitro with the `cloudflare-module` preset, `VITE_*` env injection and the `@` alias — the file warns not to add them again). It adds two things on top: `build.rolldownOptions.external: ["cloudflare:sockets"]` and a `worker-native-sockets` pre-resolver plugin that marks `cloudflare:sockets` external, plus `tanstackStart.server.entry = "server"` so Nitro builds from `src/server.ts`. `src/types/cloudflare-sockets.d.ts` types the `connect` export using `@portable/sockets`.

### 3.2 Every route file (`src/routes/`)

| File | URL | `staticData.sitemap` | Purpose |
| --- | --- | --- | --- |
| `__root.tsx` | — | `false` | Shell: charset/viewport, default title/description/OG/Twitter, Google Fonts (Cormorant Garamond, Plus Jakarta Sans, Inter), favicon, Organization + WebSite JSON-LD, Google Search Console verification meta, `QueryClientProvider`, `SiteHeader`, `<main><Outlet/></main>`, `SiteFooter`, `captureUtmFromUrl()` on every href change, `NotFoundComponent`, `ErrorComponent` (reports to Lovable telemetry). |
| `index.tsx` | `/` | `true` | Homepage: hero with Group + HPAIR logos, "Season One starts 1 Nov 2026" stat, 4-step approach, builder pairs, three refusals, facts, two "doors" (Access, Innovate for Change), HPAIR mentor grid. |
| `competition.index.tsx` | `/competition` | `true` | **Innovate for Change Challenge 2026** page: Event JSON-LD, FAQPage, breadcrumb; GSAP/Lenis motion (`useInnovateMotion`), countdown, SDG flip cards, Washing Machine Project story with lazy YouTube lightbox, 5-step journey, 10-week schedule, judging rubric (100 points), recognition tiers, award categories, parents section, fee band (USD 100 struck → USD 80), mentors, FAQ, closing. All CTAs → Group portal with carried UTM. |
| `competition.apply.tsx` | `/competition/apply` | `false` (+`noindex`) | On-site entry form for the (older) Creating Livelihoods Challenge: zod schema, localStorage draft (`bfl_cup_apply_draft_v1`), team fields, parent fields, scholarship code, consent → `supabase.from("cup_applications").insert(...)`. Reachable from `CompetitionHeader` "apply" variant and `/cup/*` 301s. |
| `dummy-comp.tsx` | `/dummy-comp` | `false` (+`noindex`) | Preserved **Creating Livelihoods Challenge 2026** page (previous competition design) so neither programme overwrites the other (AGENTS.md rule). Uses `CompetitionHeader`. |
| `innovate-for-change.tsx` | `/innovate-for-change` | `false` | `beforeLoad` 301 → `/competition`. |
| `cup.index.tsx` | `/cup` | `false` | 301 → `/competition` (legacy "Cup" naming). |
| `cup.$.tsx` | `/cup/*` | `false` | 301 → `/competition/apply` (legacy `/cup/apply`). |
| `access.index.tsx` | `/access` | `true` | Access Fellowship page: Course JSON-LD (free, `P6M`), FAQPage, breadcrumb; facts bar, 3-phase curriculum, structure stats, nomination steps, FAQ, `EnquiryForm`. |
| `access.apply.tsx` | `/access/apply` | `false` (+`noindex`) | Apply **or nominate** form (`mode: "student" | "nominator"`), draft key `bfl_access_apply_draft_v1`, accent `--bfl-blue` → `supabase.from("access_applications").insert(...)`. |
| `catalyst.index.tsx` | `/catalyst` | `false` (+`noindex`) | Catalyst Fellowship page (full copy + curriculum retained) but `beforeLoad` **302 → `/`** ("Catalyst is paused"). |
| `catalyst.apply.tsx` | `/catalyst/apply` | `false` (+`noindex`) | Catalyst application form → `catalyst_applications`; also **302 → `/`**. |
| `masterclass.index.tsx` | `/masterclass` | `true` | Lists upcoming sessions via `loader: () => listUpcomingMasterclasses()` (server fn, service-role read); hero, marquee, session cards, mentor bio with headshot asset, "Reserve free seat" → external Group form. |
| `masterclass.apply.tsx` | `/masterclass/apply` | `false` | `beforeLoad` 301 → `https://apply.example-group.com/forms/bfl-masterclass` (on-site form retired 2026-11-09). |
| `join.tsx` | `/join` | `true` | "One door into the lab": facts, 3 human steps, `EnquiryForm` (intent `student` default, selectable) → `enquiries`. |
| `faq.tsx` | `/faq` | `true` | Site-wide FAQ (10 Qs: who runs it, challenge vs fellowship, cost, sessions, time, safeguarding, data, refunds, "1 paid = 1 free seat") + FAQPage JSON-LD + breadcrumb. |
| `blog.index.tsx` | `/blog` | `true` | Blog list from `src/data/blog-posts.ts`; breadcrumb JSON-LD. |
| `blog.$slug.tsx` | `/blog/:slug` | `false` (added to sitemap dynamically) | Loader `getBlogPost(slug)` or `notFound()`; Article JSON-LD, optional FAQPage, breadcrumb, `article:published_time/modified_time`. One post exists: `best-online-competitions-for-students-2026`. |
| `ngo.tsx` | `/ngo` | `true` | NGO Challenge Network / Challenge Library: 7-step process (Submit → Report back), safeguarding promises, roles. |
| `impact.tsx` | `/impact` | `true` | "Social Impact Measurement & Claims": Skills/Tools/Market/Money ladder, reviewer panel. |
| `parents.tsx` | `/parents` | `true` | Parents' guide, ages 11–18, weekly rhythm (live class, mentor clinic, build time), safeguarding. |
| `partner.tsx` | `/partner` | `true` | Funders: fund Access seats, sponsor a challenge track, corporate challenge, back the network. |
| `schools.tsx` | `/schools` | `true` | 12-week school programme (Look / Make / Test / Show). |
| `sitemap[.]xml.ts` | `/sitemap.xml` | `false` | Server route (`server.handlers.GET`) building XML from route `staticData.sitemap === true` + blog posts (see §3.7). |
| `README.md` | — | — | File-routing conventions (no `src/pages`, splat via `$`, only `__root.tsx` is a layout). |

Header selection by path happens in `src/components/site-header.tsx`: `/competition` → `InnovateHeader`, `/dummy-comp` → `CompetitionHeader`, `/competition/apply` → `CompetitionHeader variant="apply"`, otherwise the global header (links Home / Competition / Access / FAQ, plus "Masterclass" and "Join the Movement" buttons). `CatalystHeader` exists but is unreachable while Catalyst is paused.

### 3.3 Server functions and middleware

- `src/lib/masterclass.functions.ts`
  - `listUpcomingMasterclasses` — `createServerFn({ method: "GET" })`; lazily imports `supabaseAdmin`, selects the public columns of `masterclass_sessions` where `is_published` and `starts_at > now − 1 h`, ordered ascending. Deliberately never selects `zoom_url` / `passcode`.
  - `registerForMasterclass` — `createServerFn({ method: "POST" })` with `.inputValidator(registrationSchema.parse)`; rate-limits 5 registrations per email per hour (`ilike(email)` + `created_at`), checks the chosen sessions are published, **hard-codes a guard that the sessions belong to the November Devon Bailey event** (because the confirmation email copy is specific to it), inserts into `masterclass_registrations` with role-specific columns, then calls `sendMasterclassConfirmation` and records `email_status`/`email_sent_at`/`email_error_code`. **Currently not referenced by any route** (the apply page redirects externally) — kept per the 2026-11-09 plan "the existing on-site sign-up page and saved responses stay in place".
- `src/start.ts`
  - `attachAvailableAuth` (function-middleware, client side): tries `supabase.auth.getSession()` and attaches `Authorization: Bearer <token>`; if browser auth config is missing it proceeds anonymously (AGENTS.md rule: "public registration is not blocked by missing browser settings").
  - `errorMiddleware` (request): catches non-HTTP errors and returns the branded 500 page.
  - `createCsrfMiddleware({ filter: handlerType === "serverFn" })` — re-added explicitly because defining `src/start.ts` opts out of TanStack's default CSRF protection.
- Lovable-generated, present but unused by routes: `src/integrations/supabase/auth-attacher.ts` (`attachSupabaseAuth`), `auth-middleware.ts` (`requireSupabaseAuth` — verifies a Bearer JWT with `auth.getClaims`), `cron-auth.ts` (`authenticateCronRequest` — timing-safe check of `LOVABLE_CRON_SECRET` / `LOVABLE_CRON_SECRET_PREVIOUS`). No cron routes exist.

### 3.4 Data access layer

| File | Role |
| --- | --- |
| `src/integrations/supabase/client.ts` | Browser anon client (`VITE_SUPABASE_URL` / `VITE_SUPABASE_PUBLISHABLE_KEY`, falling back to `SUPABASE_URL` / `SUPABASE_PUBLISHABLE_KEY` on SSR). Lazy `Proxy` so it only instantiates on first use. Custom fetch strips `Authorization: Bearer <publishable key>` for new `sb_publishable_` keys and sets `apikey`. Auth storage = `brokeredPreviewStorage()`. |
| `src/integrations/supabase/client.server.ts` | `supabaseAdmin` service-role client (`SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY`), `persistSession:false`. Comment: load only inside server handlers via dynamic import, because route files and `*.functions.ts` ship to the client bundle. |
| `src/integrations/supabase/previewAuthStorage.ts` | Lovable preview-iframe auth broker: on `*.lovable.app` / `lovableproject.com` hosts, brokers the Supabase session to the editor via `postMessage` (origin-validated), else `localStorage`. |
| `src/integrations/supabase/types.ts` | Generated `Database` types: tables `access_applications`, `catalyst_applications`, `cup_applications`, `enquiries`, `masterclass_registrations`, `masterclass_sessions`; **no Views, Functions or Enums**. |
| `src/lib/utm.ts` | Campaign attribution (added 2026-11-24): captures `utm_*` + `gclid/fbclid/msclkid/ttclid/li_fat_id`, first-real-value-wins, persisted in `sessionStorage`+`localStorage` under `bfl_utm_attribution`; `sourcePathWithUtm(pathname)` packs attribution into the ≤200-char `source_path` column; `withCarriedUtm(url)` appends it to outbound Group-portal links; `skipsUtmCarryForward` excludes `/pay|payment|checkout|admin|auth|login|logout` (none exist here — "kept for parity with sibling platforms"). |
| `src/lib/seo.ts` | `SITE_ORIGIN`, `canonicalUrl/canonicalLink/ogUrl/absoluteUrl`, JSON-LD builders (`organizationSchema`, `websiteSchema`, `organizationRef`, `faqPageSchema`, `breadcrumbSchema`) and the `jsonLd()` head-meta adapter. |
| `src/lib/sitemap.ts` | Module augmentation making `staticData.sitemap: boolean | "exclude-subtree"` **required** on every route; `sitemapStaticPaths(router)`, `sitemapXML(origin, entries)` with path safety checks. |
| `src/lib/responsive-image.ts` | `import.meta.glob("@/assets/*.webp")` → `srcSetFor(src)` builds `768w/1152w/full` srcsets from pre-generated variants; `PHOTO_SIZES`. |
| `src/lib/masterclass-registration.ts` | Shared **client-safe** zod schema (`registrationSchema`, role-specific required fields for Student / Parent / Educator), `masterclassGrades`, `normaliseMasterclassPhone`. |
| `src/lib/masterclass-countries.ts` | ISO-2 + dial-code list with flag emoji; `findByDial/Name/Code`. |
| `src/lib/masterclass-email.ts` | Subject, the session facts object (includes Zoom URL + passcode — server-only by usage), `confirmationRecipients()` (student → parent CC unless same address), `renderMasterclassEmail()` table-based HTML + text. |
| `src/lib/masterclass-mail.server.ts` | SMTP send (see §4.6). |
| `src/lib/error-capture.ts`, `error-page.ts`, `lovable-error-reporting.ts` | Wraps `console.error` to expand Error cause chains, records the last error so `server.ts` can recover a stack after h3 swallows it; static HTML 500 page; forwards boundary errors to `window.__lovableEvents` / `__lovableReportRuntimeError`. |
| `src/lib/utils.ts` | `cn()` (clsx + tailwind-merge). |

### 3.5 Auth

There is **no user authentication feature** on the site: no login route, no `auth.signIn*` calls, no protected pages. All public writes are anonymous inserts under RLS `WITH CHECK` policies; the only privileged reads/writes go through `supabaseAdmin` inside server functions. The `attachAvailableAuth` middleware exists only so that, if a Lovable preview session happens to exist, its token is forwarded.

### 3.6 Design system (`src/styles.css`, 1 920 lines)

- **Fonts** (`@theme inline`): `--font-jakarta` Plus Jakarta Sans (headings/UI eyebrows), `--font-sans` Inter (body), `--font-display` Source Serif 4 (pull quotes and mission statement only), `--font-cormorant` Cormorant Garamond (hero + section headings on most pages: "Hero heading · Cormorant Garamond 700"). Google Fonts loaded in `__root.tsx` for Cormorant, Plus Jakarta Sans, Inter.
- **Colour law** ("Brand Guidelines v3 · Six colours, one job each"):
  - `--midnight #0b1420`, `--midnight-2 #111c2b`, `--midnight-foreground / --warm #f7f6f2`, `--paper #eceae4`
  - `--bfl-blue #1465ff` ("Catalyst Blue · something was built"), ink-safe `--bfl-blue-ink / --bfl-blue-deep #0e49bc`, `--bfl-blue-air #8fb2ff`
  - `--signal-green #1e9e5c` ("Access Green · something grew"), `-hi #22b573`, `-ink #116b3d`
  - `--signal-gold #d9a521` ("Gold · something was earned. Marks, never fills"), `-hi #f5c542`, `-ink #7a5600`
  - `--signal-orange #ff6835` ("Impact Orange · one appearance per page"), `-ink #a83a09`
  - `--signal-crimson #b3123f`, `--harvard-red #a51c30` (mentor emphasis), display-only `--signal-amber #ffb224`, `--signal-coral`
  - Wash surfaces `--wash-blue #e7eeff`, `--wash-green #e6f6ef`, `--wash-amber #fff4dc`, `--wash-neutral #f0efea`; `--ink-soft #4a5158`, `--dim #8a94a3`, `--line-dark #243248`; SDG tile colours `--sdg-one/two/eight`.
  - `--radius: 3px` (deliberately square).
- **Page accent system**: `[data-page-accent="gold|green|blue|…"]` sets `--page-accent` (ink-safe), `--page-accent-hi` (bright, for midnight surfaces) and `--page-accent-display` (bright, large type only); utilities `.accent-text`, `.btn-accent`, `.eyebrow`, `.display-*`, `.body-*`. The 2026-08-29 contrast plan explains why: bright accents failed WCAG AA as small text, so every accent got an "ink" partner.
- Scoped page skins: `.cup-page`, `.innovate-page` / `.innovate-hero` (AGENTS.md: Innovate styles must stay scoped), `.fx-*` motion classes, masterclass refinements, brand lockup masks (`.brand-lockup-*` in `src/components/brand-lockup.tsx`).
- Shared editorial primitives live in **`src/components/bfl.tsx` (3 556 lines, ~55 exports)**: `Reveal`, `Eyebrow`, `ArrowLink`, `ButtonLink`, `Section`, `PageHero`, `AnnotatedHero`, `Figure`, `ImageBreak`, `PullQuote`, `NumberedHeading`, `FactsBar`, `SectionHeading`, `StepGrid`, `JourneyRows`, `SpecTable`, `Checklist`, `QuoteBand`, `PairCard`, `Faq`/`FaqList`, `ClosingSection`, `ColorBreak`, `Marquee`, `BandSection`, `ImpactBand`, `MentorSection`/`MentorGrid`, `DownloadBar`, `PhaseTimeline`, `TrackCards`, `WeekCurriculum`, `MasterclassCurriculum`, `CredentialBand`, `FeeGrid`, `PricingCards`, `StatRow`, `SessionCalendar`, `WeekTable`, `PriceBand`, `SectionNav`, `Disclosure`, etc.
- Other components: `audience.tsx` (`AudienceFork`, `INTENTS`, `EnquiryForm`), `apply-form.tsx` (shared `FormSection`, `Field`, `ErrorSummary`, `SubmitPanel`, `HEAR_OPTIONS`, `--apply-accent` CSS var), `site-header.tsx`, `site-footer.tsx`, `brand-lockup.tsx`, `competition-header.tsx`, `innovate-header.tsx`, `catalyst-header.tsx` (sticky microsite headers with section anchors + apply CTA), `innovate-fx/*`.
- Images: `src/assets/*.webp` with `-768`/`-1152` variants (from the 2026-08-29 "convert every photo to webp" plan); Lovable-hosted assets referenced through `*.asset.json` descriptors whose `url` is `/__l5e/assets-v1/<asset-id>/<file>`.

### 3.7 SEO / AEO

- Per-route `head()` with title, description, OG, Twitter, `ogUrl()` and `canonicalLink()` on every public page (commit `987b0ab`, 2026-11-14); `noindex` on forms, Catalyst routes and `/dummy-comp` (`5acaaa4`).
- JSON-LD (commit `dfc1d90`, 2026-11-16): Organization + WebSite site-wide; Event (with two `Offer`s, `priceValidUntil` 2026-11-30, `EducationalAudience`) on `/competition`; Course on `/access`; FAQPage on `/competition`, `/access`, `/faq`, blog posts; BreadcrumbList on inner pages; Article on blog posts. Rendered via `{ "script:ld+json": data }` head-meta entries that TanStack escapes server-side.
- `public/robots.txt`: allow-all for Googlebot, Bingbot, Twitterbot, facebookexternalhit and `*`; `Sitemap:` line.
- Sitemap: `src/routes/sitemap[.]xml.ts` + `src/lib/sitemap.ts`. Every route must declare `staticData.sitemap` (TypeScript error otherwise); dynamic/splat routes are skipped by the static scan; blog posts are appended with `lastmod`; cache `public, max-age=3600`. AGENTS.md: "The sitemap is generated from required route-level sitemap decisions using the public site origin; this keeps static pages discoverable while excluding redirects, paused programmes and forms."
- Google Search Console verification meta in `RootShell` (`6154c18`).
- No prerender/SSG step: pages are SSR'd on request by Nitro.

### 3.8 Animation (GSAP + Lenis)

Only `/competition` is animated. `src/components/innovate-fx/use-innovate-motion.ts` (292 lines):

- Runs in `useEffect` after hydration; respects `prefers-reduced-motion` (no movement, content visible by default) and only wires pointer effects on `(hover: hover) and (pointer: fine)` devices; everything is pushed to a `cleanups[]` and reverted on unmount.
- Always-on: scroll-progress bar (`.fx-progress`), SDG flip-card tap/keyboard toggle.
- Lazy `Promise.all([import("gsap"), import("gsap/ScrollTrigger"), import("lenis")])`, then: Lenis smooth scroll (`duration 1.1`) driven by `gsap.ticker`; word-split hero drop-in; self-drawing SVG scribbles; per-section heading word reveals; `[data-count]` stat count-ups; SDG tiles dealt from a stack; story lines brighten/dim; washing-machine crank/drum scrub rotation; typewriter `.fx-type`; journey step `is-centre` toggles (horizontal pinned variant is coded but disabled via `horizontalSteps = false`); schedule fill line; judging points bars + total; award cards fade; price "strike then drop" timeline; parents fade; confetti burst on first hover of the top-3 card; custom cursor dot/ring (ring says "Claim →" over Group apply links); magnetic apply buttons; hero depth parallax + 3D tilt with glare; closing spotlight; gyroscope tilt on touch devices.
- `src/components/innovate-fx/parts.tsx`: `EARLY_BIRD_DEADLINE`, `Countdown` (renders zeros on the server, ticks after hydration), `Doodles`, `WashingMachineSketch`, `PhotoDoodles`, `VideoLightbox` (YouTube `youtube-nocookie` iframe loaded only after Play; AGENTS.md rule).
- AGENTS.md: "Innovate for Change motion lives in `src/components/innovate-fx/` (GSAP, ScrollTrigger and Lenis, loaded dynamically after hydration and scoped to that page); this keeps heavy animation out of other pages and out of SSR." Dependencies arrived in commit `8ef5da2` (2026-10-29).

### 3.9 Responsive patterns

`clamp()` fluid spacing (`py-[clamp(3rem,6.5vw,5.25rem)]`), `max-w-[1240px]` containers, mobile nav drawer with `aria-expanded/aria-controls` and body scroll lock, `useIsMobile()` hook at 768 px, `srcSetFor` + `PHOTO_SIZES` for photos, hero rules in `styles.css` for phones ("establish a calm reading field around the copy"), many `.lovable/plan/*` entries dedicated to "verify desktop and phone" passes, audits run at 1280×1800 and 390×844 (and 834 tablet for images).

### 3.10 Accessibility

`aria-label`s on logo links and flip cards, `role="alert"` error summaries, `role="timer"` countdown, `aria-hidden` decoration, `sr-only` brand text, reduced-motion checks in both the motion hook and header scroll-to, skip of third-party video until play, WCAG-AA ink tokens, and the two Playwright audits (§5.3).

---

## 4. Backend (Supabase project `<BUILDERS_REF>`)

### 4.1 Shape

- **No edge functions**: `supabase/` contains only `config.toml` (one line) and `migrations/` (4 files). There is no `supabase/functions/`, no `_shared/`, no `Deno.env.get` anywhere, no webhooks (no Stripe/Razorpay/Resend/WhatsApp code).
- **No SQL functions, views, enums, triggers or cron jobs** in any migration; `types.ts` confirms `Views/Functions/Enums: never`.
- Two migration streams: `supabase/migrations/*.sql` (Lovable Cloud, Aug–Sep 2026) and `drizzle/migrations/*.sql` (Nov 2026 masterclass feature, applied via `drizzle-kit` with `LOVABLE_DB_MIGRATION_URL`; `drizzle/schema.ts` is intentionally blank — "auto-generated and intentionally left blank, do not edit").

### 4.2 Tables

| Table | Migration | Columns (type, notes) | Grants / RLS |
| --- | --- | --- | --- |
| `public.enquiries` | `20260827103324…` | `id uuid pk`, `intent text NOT NULL CHECK IN ('student','parent','school','ngo','funder','catalyst','cup')`, `name`, `email`, `organisation`, `country`, `message`, `source_path`, `created_at` | `INSERT` to anon+authenticated; `ALL` to service_role; RLS on; policy "Anyone can submit an enquiry" FOR INSERT WITH CHECK (name 1–120, email 3–200 `LIKE '%_@_%.__%'`, message ≤4000, organisation ≤200, country ≤100, source_path ≤200). **No SELECT policy → write-only from the site.** |
| `public.catalyst_applications` | `20260902095641…` + `20260923111040…` | student_name/dob/email/phone, country/state/city, school_name, grade_level, parent_name/phone/email, scholarship_code, source_path, created_at; later `counsellor_name`, `counsellor_email`, `hear_about`, `motivation`, `comments` | INSERT anon/authenticated with length + email checks; write-only. |
| `public.cup_applications` | `20260902123851…` | team_name, team_size, team_members, student_* , country/state/city, school_name, grade_level, parent_*, challenge_interest, scholarship_code, source_path, created_at | INSERT policy "Anyone can submit a cup application" with regex email checks; write-only. |
| `public.access_applications` | `20260923111040…` | `applicant_type text NOT NULL` (`student`/`nominator` enforced in policy), organisation_name, nominator_name/role/email/phone, student_name (NOT NULL)/dob/email/phone, country (NN), state, city (NN), school_name, grade_level, guardian_name/phone/email, hear_about, `motivation text NOT NULL` (1–2000), comments, source_path, created_at | INSERT anon/authenticated with checks; write-only. |
| `public.masterclass_sessions` | `drizzle 0000_masterclass` | `id`, `slug UNIQUE`, `title`, `mentor_name`, `mentor_role`, `mentor_bio`, `starts_at timestamptz`, `duration_minutes` (60), `time_label`, `eligibility`, `fee_label` ('Complimentary'), `summary`, `outcomes text[]`, `closing_line`, **`zoom_url`, `passcode`** (private), `is_published` (true), `created_at` | **Column-level** `GRANT SELECT (…public columns…)` to anon/authenticated — `zoom_url`/`passcode`/`closing_line` excluded; policy "Published masterclasses are public" USING (is_published). Seeded with the 2026-11-21 session (including its Zoom link and passcode — values not reproduced here). |
| `public.masterclass_registrations` | `drizzle 0000` + `0001` + `0002` | `session_ids uuid[]`, `full_name`, `email`, `role`, `grade`, `country`, `phone`, `hear_about`, `email_sent bool`; 0001 adds `school_name`, `parent_name/email/phone`, `student_name`, `student_grade`, `designation`; 0002 adds `email_status` ('pending'|'sent'|'failed'|'partial' CHECK), `email_sent_at`, `email_error_code` | **service_role only** (no anon grants at all) — inserts happen inside the server function with the admin client. |

Pattern notes:
- Every public-insert table repeats the same defence: explicit `GRANT INSERT` to `anon, authenticated` (the GRANT-plus-RLS gotcha recorded in the shared Group memory) and a `WITH CHECK` policy that acts as server-side validation.
- `source_path` (≤200 chars) is the only attribution field; since 2026-11-24 it carries `path?utm_source=…&utm_medium=…&utm_campaign=…` (`sourcePathWithUtm`).
- Nothing reads from these tables in the app; review happens in the Supabase/Lovable dashboard.

### 4.3 Edge functions

None (table intentionally empty):

| name | trigger | auth | tables | purpose |
| --- | --- | --- | --- | --- |
| — | — | — | — | No `supabase/functions` directory exists. |

### 4.4 Webhooks / payments

None. Payment for the competition happens on the Group Application Portal (`apply.example-group.com`), not here.

### 4.5 External integrations by secret name

| Secret / env name | Used in | Purpose |
| --- | --- | --- |
| `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `VITE_SUPABASE_PROJECT_ID` | `src/integrations/supabase/client.ts`, `.env` | Browser anon client |
| `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_PROJECT_ID` | SSR fallback, `auth-middleware.ts` | Server-side anon/JWT client |
| `SUPABASE_SERVICE_ROLE_KEY` | `client.server.ts` | `supabaseAdmin` (masterclass read/insert/update) |
| `GMAIL_SENDER_EMAIL`, `GMAIL_APP_PASSWORD` | `src/lib/masterclass-mail.server.ts` | Gmail SMTP submission (app password, whitespace stripped) |
| `LOVABLE_CRON_SECRET`, `LOVABLE_CRON_SECRET_PREVIOUS` | `cron-auth.ts` | Lovable cron bearer check (unused) |
| `LOVABLE_DB_MIGRATION_URL` | `drizzle.config.ts` | Drizzle migration connection |

### 4.6 What `@portable/smtp` and `@portable/sockets` do here

Both are 0.1.0 packages from `github.com/<org>/<repo>` (MIT), added in commit `830b541` on 2026-11-09 for the masterclass confirmation email.

- **`@portable/smtp`** — "Send email over SMTP with STARTTLS, AUTH, and pipelining, in any JavaScript runtime that has a TCP socket API" (replaces nodemailer's SMTP transport). You pass it a `connect` function; it only needs Web Streams.
- **`@portable/sockets`** — types for the WinterTC Sockets API (the shape of `connect` from `cloudflare:sockets`) plus a `@portable/sockets/node` implementation on `node:net`/`node:tls` so the same client runs in Workers, Node 24+, Bun and Deno.

In `src/lib/masterclass-mail.server.ts`:

```ts
// Vite preview uses Node sockets; the deployed runtime uses Worker-native sockets.
const { connect } = import.meta.env.DEV
  ? await import("@portable/sockets/node")
  : await import("cloudflare:sockets");
const result = await Smtp.send({ from: sender, to: recipients, message }, {
  connect, host: "smtp.gmail.com", port: 465, secure: "on",
  auth: { user: sender, pass: password.replace(/\s/g, "") },
  signal: AbortSignal.timeout(25_000),
});
```

The module builds the raw MIME message itself (multipart/alternative, base64 bodies, UTF-8 B-encoded subject, `Message-ID: <masterclass-<id>@builders.example.com>`), refuses header-injection characters in addresses, treats `rejected.length === 0 && accepted.length === recipients.length` as `sent`, otherwise `partial`; `emailErrorCode()` maps `Smtp.CommandError.code` or `EMAIL_NOT_CONFIGURED` / `EMAIL_SEND_FAILED` and the comment says "Never log SMTP replies or authentication payloads". This is why `vite.config.ts` must declare `cloudflare:sockets` as a native external for both the resolver and Nitro's final bundle (AGENTS.md rule).

The original 2026-11-09 plan intended Lovable's built-in `sendTemplateEmail` with a React Email template; the shipped implementation instead mirrors Pre-Med Platform' Gmail-account approach ("Published and tested Gmail", "Verified Gmail live sending" commits). `roadmap.md` records: "Gmail accepted live To and CC; inbox placement is not guaranteed".

---

## 5. Key end-to-end flows

### 5.1 Visit → competition application (primary revenue funnel)

1. Visitor lands anywhere; `__root.tsx` `RootComponent` calls `captureUtmFromUrl()` on every `location.href` change → attribution saved under `bfl_utm_attribution`.
2. `/competition` renders SSR with Event JSON-LD; after hydration `useInnovateMotion()` loads GSAP/Lenis; `Countdown` ticks toward 30 Nov 23:59 EST.
3. Every "Claim your spot" (`ApplyButton`), header CTA (`InnovateHeader`) and closing CTA is an `<a target="_blank" rel="noopener noreferrer">` to `withCarriedUtm("https://apply.example-group.com/apply/billion-founders-competition")`.
4. Payment, forms and confirmation happen on the **Group Application Portal**; nothing is written to this project's database for the live competition.

### 5.2 On-site forms → tables (no emails)

| Form | Route / component | Validation | Insert | After |
| --- | --- | --- | --- | --- |
| Enquiry | `EnquiryForm` in `src/components/audience.tsx` (on `/join`, `/access`, audience pages) | HTML required fields; DB policy | anon `supabase.from("enquiries").insert({intent, name, email, organisation, country, message, source_path})` | `state = "sent"`; error copy "We could not send that…" |
| Access apply / nominate | `src/routes/access.apply.tsx` | zod `schemaFor(mode)`, consent checkbox, draft autosave in localStorage | anon insert into `access_applications` | "Received" screen; localStorage draft cleared |
| Competition (old) | `src/routes/competition.apply.tsx` | zod, consent | anon insert into `cup_applications` | done screen |
| Catalyst | `src/routes/catalyst.apply.tsx` | zod, consent | anon insert into `catalyst_applications` | unreachable (302) |

No email, Slack or CRM notification is triggered by any of these inserts; there are no DB triggers.

### 5.3 Masterclass flow (as deployed)

1. `/masterclass` loader → `listUpcomingMasterclasses()` (service-role, public columns only, sessions ending within the last hour dropped) → session cards.
2. "Reserve free seat" → external `https://apply.example-group.com/forms/bfl-masterclass` (new tab). `/masterclass/apply` 301s to the same URL so old `?session=` links survive.
3. Retired-but-retained path: `registerForMasterclass` → rate-limit → verify sessions → insert `masterclass_registrations` → `sendMasterclassConfirmation` (To registrant, CC parent for students) → update `email_status`. AGENTS.md: "records recipient acceptance, and preserves registrations on email failure; this prevents credentials entering the browser and avoids claiming delivery on failure."

### 5.4 Content pipeline (`content/`)

`content/` is **not** a markdown/JSON page source. It holds:

- `content/verified-claims.md` — the **claims ledger** read by `scripts/copy-qa.py`. Format: `- claim: "…"` / `status: verified | pending` / `source: who confirmed, when`. Sections: Partnerships and affiliations (e.g. "Innovate for Change Challenge 2026" verified 9 Oct 2026; HPAIR-at-university, "mentors from HPAIR", MIT Solve/XPRIZE/UNICEF/Grand Challenges attributions still **pending**), Prices and funding (USD 80/100 verified 4 Nov 2026; "students pay nothing", "one hundred fully funded seats", "Catalyst fee", "prize" pending), Dates (Saturdays 11–12 EST verified 24 Sep 2026; "key dates" pending), Catalyst document claims held from publication (Anthropic Academy / Hugging Face / Lovable / AWS certificates, cohort cadence, mentor ratios — all pending). Two trailing entries use a different `verified: 2026-09-24` / `note:` shape that the parser does not read as `status: verified`.
- `content/references/Curriculum_for_Billion_Founders_Lab.pdf` and `catalyst-website-and-curriculum.docx` — the owner-supplied source documents the pages were rebuilt from (referenced by `.lovable/plan/*` and AGENTS.md "uploaded-document livelihood presentation").

Actual page copy lives in TSX string literals (`src/routes/*.tsx`) and typed data modules (`src/data/blog-posts.ts`, `challenges.ts`, `mentors.ts`, `cup-curriculum.ts`, `catalyst-curriculum.ts`). Blog posts are added by appending to the `posts` array; `/blog`, `/blog/<slug>` and the sitemap pick them up automatically. There is no CMS.

### 5.5 QA / publication gate (`scripts/`)

| Script | npm script | What it checks | Exit |
| --- | --- | --- | --- |
| `scripts/copy-qa.py` (302 lines) | `qa:copy` | Static scan of string literals in `src/routes/*.tsx`, `src/data/*.ts`, `src/components/bfl.tsx` for **price** (currency, fee, free, scholarship, prize…), **date** (capitalised month names, years, deadline, intake, cohort starts…), **partnership** (HPAIR, university, MIT Solve, XPRIZE, UNICEF, partner, accredited…) claims; matches against `content/verified-claims.md`. **Placeholders (`XX`, `TBC`, `TBD`, `TODO`, `Lorem`) always fail and cannot be allow-listed.** `--json`, `--show-verified`. | 1 if any pending/unsigned/placeholder |
| `scripts/contrast-audit.py` (295) | `audit:contrast` | Playwright + Pillow: on `/`, `/access`, `/catalyst`, `/competition`, `/ngo`, `/impact` at desktop 1280×1800 and mobile 390×844, finds text over images/gradients, hides it, screenshots the backdrop, computes WCAG 2.1 ratio against the **worst 5th-percentile** pixels; fails if >10 % of backdrop is below 4.5:1 (3:1 for large text). Needs dev server on `localhost:8080`. | 1 on failures |
| `scripts/a11y-audit.py` (88) | `audit:a11y` | Playwright injects `node_modules/axe-core/axe.min.js`, runs `wcag2a/wcag2aa/wcag21a/wcag21aa` on the same six routes × two viewports; fails on serious/critical. | 1 on serious/critical |
| `scripts/image-audit.py` (242) | (none) | Per-image sharpness vs DPR, mean luminance, crushed-black share, alt/width/height hygiene at mobile/tablet/desktop. | — |
| `scripts/build-docs.py` (738) | (none) | python-docx generator for `BFL_Content_Master.docx` and `BFL_Design_System.docx` from Playwright extractions in `/tmp/browser/bfl` → `/mnt/documents` (Lovable sandbox paths; not runnable locally as-is). | — |

`qa:prepublish` = `npm run qa:copy && npm run audit:access` (`audit:access` = a11y then contrast). It is a manual gate (no CI config in the repo); the 2026-08-29 plan states the first run was expected to fail by design until placeholders and unconfirmed fees were resolved.

### 5.6 Newsletter / lead capture

No newsletter. Lead capture = `enquiries` table (intent-tagged) and the application tables, all with `source_path` attribution. No marketing pixels/scripts are loaded (only Google Fonts and the YouTube embed on demand).

---

## 6. Standalone status vs the shared Group estate

**Fully standalone backend.** Evidence:

- `grep -rn "all_programs\|application_portal\|lwl_" src` → no matches except unrelated copy words ("catalogue" in page text). The generated `types.ts` lists only the six local tables; no cross-schema or cross-project client exists.
- Only one Supabase URL/key pair is configured (`VITE_SUPABASE_*` for project `<BUILDERS_REF>`); no second `createClient` anywhere.
- Integration with the wider Group estate is **by hyperlink only**: `apply.example-group.com/apply/billion-founders-competition` (competition) and `apply.example-group.com/forms/bfl-masterclass` (masterclass), both carrying UTM via `withCarriedUtm`. The commit body of `38653ee` confirms the UTM helper was "Adapted from purpose-platform/Test-Prep Platform's src/lib/utm.ts pattern" and that "this repo's Supabase project isn't reachable from this session to apply a migration".
- Masterclass email identifies as the Group, but is sent via this project's own Gmail credentials, not a shared service.

---

## 7. Conventions, tests, scripts, deployment

### 7.1 `AGENTS.md` (key rules, quoted)

- Lovable banner: "Avoid rewriting published git history — force pushing, or rebasing/amending/squashing commits that are already pushed … Commits you push to the connected branch sync back to Lovable and show up in the editor, so keep the branch in a working state."
- "Catalyst is paused: preserve its application data and route files, but redirect both public Catalyst addresses to the homepage; this keeps old links safe without exposing the programme."
- "The Competition page keeps its verified programme facts in `content/verified-claims.md` and its uploaded-document livelihood presentation in the page route; this separates claim provenance from display copy."
- "Competition programmes use separate route files: Innovate for Change lives at `/competition` (with `/innovate-for-change` redirecting to `/competition`) while Creating Livelihoods is preserved at `/dummy-comp`; this prevents either programme from overwriting the other."
- "Keep Innovate for Change opening styles scoped to `.innovate-hero` and its children … Scope Innovate for Change section refinements under `.innovate-page` rather than changing shared section or mentor components."
- "Load the Innovate story video embed only after the visitor presses play, using a hosted thumbnail and a direct YouTube fallback link."
- "Innovate for Change motion lives in `src/components/innovate-fx/` (GSAP, ScrollTrigger and Lenis, loaded dynamically after hydration and scoped to that page)."
- "Masterclass registration uses a shared client-safe validation schema for role-specific contact fields, with server-only persistence."
- "Masterclass confirmation sending uses a server-only portable SMTP helper with separate preview and Worker socket adapters, records recipient acceptance, and preserves registrations on email failure."
- "Declare cloudflare:sockets as a native external in production build.rolldownOptions as well as the resolver plugin; Vite 8's publication linker must preserve the runtime-provided module without externalising npm dependencies."
- "Global function middleware attaches available browser auth but allows anonymous calls if browser auth configuration is unavailable."
- "The sitemap is generated from required route-level sitemap decisions using the public site origin."

### 7.2 `roadmap.md`

A checklist log (all items `[x]`) of the last delivered features, newest first: Masterclass confirmation email (table-based HTML, Gmail with parent CC, "inbox placement is not guaranteed"); Masterclass page/registration refinement ("match the Financial Literacy Platform registration fields"); Competition rename (`/competition`, preserve `/cup` redirects, keep internal `cup_*` identifiers); session time 11–12 EST + PDF update; top-10 sequence; Impact Challenge rename; Creating Livelihoods page update (Grades 6–12); Fellowship mobile polish; Catalyst visibility pause; Innovate for Change rebuild from the supplied PDF brief; keep both competition pages; multiple Innovate polish passes (hero, curriculum rows, screenshots, awards/pricing, "Standardise Claim your spot").

### 7.3 Tests

- `src/lib/sitemap.test.ts` — asserts `competition.index.tsx` has `sitemap: true`, both Catalyst routes and `dummy-comp.tsx` have `sitemap: false`, dedupes URLs, no invented `lastmod`, rejects `//example.com` paths.
- `src/lib/masterclass-email.test.ts` — recipient/CC rules; checks the Zoom path and passcode constants.
- `src/lib/masterclass-registration.test.ts` — role-specific required fields, phone validation, dial-code normalisation.
- All use `node:test` + `node:assert`; **no `test` script exists** in `package.json`, so they must be run manually (e.g. `node --test` with TypeScript stripping, or `bun test`). Undetermined which runner the team uses.

### 7.4 Scripts (`package.json`)

`dev` (`vite dev`), `build`, `build:dev`, `preview`, `lint` (eslint 9 flat config with a custom rule banning the Next.js `server-only` import), `format` (prettier, printWidth 100), `audit:contrast`, `audit:a11y`, `audit:access`, `qa:copy`, `qa:prepublish`.

### 7.5 Deployment

- Built by Lovable from `main`; Nitro preset `cloudflare-module` (Lovable config default) → Cloudflare Worker runtime, hence `cloudflare:sockets` for SMTP. No `wrangler.toml` in the repo (Lovable manages it); `.wrangler/`, `.dev.vars`, `.output`, `.nitro` are git-ignored.
- Human pushes to GitHub `main` sync back into Lovable (README). PR #1 (`add-blog-footer-link`, 2026-11-12) shows the team occasionally uses branches + PRs.
- `bunfig.toml`: `saveTextLockfile = true`, `minimumReleaseAge = 86400` with excludes only for `@lovable.dev/*` packages ("confirm with the user before adding any").
- `tsconfig.json`: strict + `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noPropertyAccessFromIndexSignature` (hence `process.env['X']` bracket style everywhere).

---

## 8. Timeline and milestones

### 8.1 Commits by month

| Month | Commits |
| --- | --- |
| 2026-08 | 1 133 |
| 2026-09 | 617 |
| 2026-10 | 219 |
| 2026-11 | 86 |

Authors: `an-ai-builder-bot` 2 034 (Lovable), another-contributor 8 + another-contributor 1, a-team-member 5, the-operator 5, Lovable 2. **1 634 commits are titled just "Changes" / "Update plan" / "Work in progress"** — Lovable's generic messages; the 136 files in `.lovable/plan/` are the real change log for Aug–Oct.

### 8.2 Key milestones

| Date | Commit / plan | Milestone |
| --- | --- | --- |
| 2026-08-20 | `0f401e5` | Lovable TanStack Start template created. |
| 2026-08-26 | plans `billion-founders-lab-website`, `rebuild-the-site-to-the-website-build-pack`, `e56fa55` "Added automated a11y checks" | Site built from the "website build pack"; a11y audit script introduced. |
| 2026-08-27 | first Supabase migration (`enquiries`), `0f81c5f` "Rebuilt Cup page with visuals" | Enquiry capture live; Cup page. |
| 2026-08-29 | plans `contrast-and-readability-qa-pass`, `copy-qa-checklist-in-the-build-flow`, `fix-slow-image-loading-convert-every-photo-to-webp`, `brand-v3-rollout` | WCAG ink tokens, `copy-qa.py` + `verified-claims.md`, WebP pipeline, Brand v3 colour law. |
| 2026-08-30 | `13bba97` "Built Cup microsite header", fee-section plans | Cup microsite + fees. |
| 2026-09-02 | migrations `catalyst_applications`, `cup_applications`; `f737416`, `43cb900` | On-site apply forms for Catalyst and Cup. |
| 2026-09-17 | `05d9e4f` "/cup/apply", `80bd0b3` curriculum masterclasses | Cup rebuilt "in the Pre-Med Platform competition" style. |
| 2026-09-18/21 | `5207bb4`, `022cff0` (a-team-member) | First human commits: hero typography/partner logos; "light textured university theme"; `@lovable.dev/vite-tanstack-config`, `rolldown` override and `axe-core` enter `package.json`. |
| 2026-09-23 | migration `access_applications` + Catalyst columns; `c752465` "Updated Cup to Competition"; `a2601ea` "Linked Competition Apply btns" | Access form; rename to Competition; CTAs → Group portal. |
| 2026-09-24/25 | `4a25c21`, `209f760`, `a5b3831` | 11–12 EST slot, eighth masterclass for top 10, rename to Impact Challenge, Catalyst hidden. |
| 2026-09-26 | `3d1d3f2` | Catalyst paused (redirects). |
| 2026-10-09/10 | `dd33773` "Split competition pages", `68b1f63` | Innovate for Change page built alongside Creating Livelihoods. |
| 2026-10-29 | `8ef5da2` (gsap + lenis), plan `innovate-for-change-motion-effects-from-the-revised-spec` | Premium motion layer. |
| 2026-11-05 | `301aae0` (a-team-member) | Innovate for Change → `/competition`, Livelihoods → `/dummy-comp`, fees USD 80/100; the-operator bumps Vite to 8.1.6, untracks `.env` and stale `package-lock.json`. |
| 2026-11-09 | `03cfb76`…`22a7452`, `830b541`, drizzle 0000–0002 | Masterclass page, tables, Gmail SMTP via `@portable/smtp`, then CTA switched to the external Group form and `/masterclass/apply` redirected. |
| 2026-11-12/14 | `db1bc81`, `49dfae2`, `987b0ab`, `5acaaa4`, `6154c18`, `1c5a6bc`, `9373568` | Blog, FAQ, canonical/og:url, noindex, GSC verification, title rewrites, early-bird date fixed to 30 Nov. |
| 2026-11-16 | `dfc1d90` | Structured data across the site. |
| 2026-11-24 | `38653ee` | UTM capture/carry-forward (last commit). |

---

## 9. Gotchas and lessons

1. **Lovable drift and generic commits** — 2 034 bot commits, mostly "Changes". Before any local work, pull `origin/main`; the 2026-11-24 merge commit shows the team already does this. Never rewrite history (AGENTS.md).
2. **bun, not npm** — `bun.lock` is authoritative; `package-lock.json` was committed once by accident and is now ignored (commit `6570233`). `overrides.rolldown = "1.2.1"` pins the Vite 8 bundler; bun tolerates overrides that npm may reject (see the shared Group memory note). `bunfig.toml`'s 24 h release-age guard can block brand-new package versions.
3. **`.env` was tracked historically** — it held only publishable values ("no service-role secret, so no key rotation is needed"), and is now ignored. Do not re-add it.
4. **`cloudflare:sockets` must be external twice** — in the pre-resolver plugin *and* `build.rolldownOptions.external`; otherwise Vite 8's production linker breaks the SMTP sender. In dev the Node adapter is used instead.
5. **Server-only modules** — `*.server.ts` files (`client.server.ts`, `masterclass-mail.server.ts`) must be imported dynamically inside server-function handlers; route files and `*.functions.ts` are shipped to the browser. ESLint blocks the Next.js `server-only` package.
6. **Defining `src/start.ts` opts out of default CSRF middleware** — it is re-added explicitly; keep `createCsrfMiddleware` when editing.
7. **h3 swallows handler throws into `{"unhandled":true,"message":"HTTPError"}`** — `src/server.ts` + `error-capture.ts` exist to turn that into a readable log and branded page. Keep `import "./lib/error-capture"` first.
8. **Every route must set `staticData.sitemap`** — it is a required TypeScript field via module augmentation; forgetting it fails type-check. Forms, redirects and paused programmes are `false`; dynamic routes are added by hand in `sitemap[.]xml.ts`.
9. **Copy QA gate** — any new price/date/partnership wording must be added to `content/verified-claims.md` with `status: verified` or `qa:copy` fails; placeholders can never be allow-listed. Several live claims (HPAIR-at-university, MIT Solve attributions, "students pay nothing") are still `pending`, so the gate currently fails by design until resolved.
10. **Contrast rules** — bright brand colours are display-only; small text/buttons on light surfaces use the `*-ink` tokens; `text-white/50` was lifted to `/70`; the contrast audit measures real pixels under text over photos.
11. **Innovate page isolation** — all motion and styling scoped to `.innovate-page` / `.innovate-hero` / `.fx-*`; `Countdown` must render zeros on the server to avoid hydration mismatch; third-party video loads only on play.
12. **Masterclass email is event-specific** — `registerForMasterclass` refuses sessions other than the November 2026 Devon Bailey one, and `masterclass-email.ts` hard-codes that session's details; adding a session requires generalising the template. The Zoom URL and passcode are also present in the seed migration, the email module and a test — treat those files as sensitive.
13. **Write-only tables** — no SELECT policies for anon; dashboards must use the service role. `masterclass_registrations` has no anon grants at all (service-role only).
14. **Legacy naming persists in the database** — tables are `cup_applications` and the `enquiries.intent` CHECK still uses `'cup'` and `'catalyst'` although the public names changed (roadmap: "Keep challenge terminology and internal saved-data identifiers unchanged").
15. **Old apply forms still reachable** — `/competition/apply` (old Livelihoods form → `cup_applications`) remains linked from `CompetitionHeader` on `/dummy-comp` and via `/cup/*` redirects, while the live competition applies on the Group portal. Expect occasional rows there.
16. **`scripts/build-docs.py` and the audits assume the Lovable sandbox** (`/tmp/browser/bfl`, `/mnt/documents`, dev server on `:8080`); Vite's default dev port differs unless the Lovable config forces 8080.
17. **UTM columns do not exist** — attribution is squeezed into `source_path` (≤200 chars); adding real `utm_*` columns needs a migration run with DB access to `<BUILDERS_REF>`.

---

## 10. File map (most important files)

| Path | One-line purpose |
| --- | --- |
| `AGENTS.md` | Lovable sync warning + project-specific architectural rules (Catalyst pause, scoped Innovate styles, SMTP/socket externals, sitemap policy). |
| `roadmap.md` | Delivered-feature checklist, newest first. |
| `README.md` | Lovable project link, preview URL, npm dev instructions (stale: repo uses bun). |
| `package.json` | Scripts (incl. `qa:prepublish`), deps, `overrides.rolldown`. |
| `bunfig.toml` | Text lockfile + 24 h supply-chain guard. |
| `vite.config.ts` | Lovable TanStack preset + `cloudflare:sockets` external + `src/server.ts` entry. |
| `drizzle.config.ts` | drizzle-kit → `drizzle/migrations`, `LOVABLE_DB_MIGRATION_URL`. |
| `supabase/config.toml` | `project_id = "<BUILDERS_REF>"`. |
| `supabase/migrations/*.sql` (4) | `enquiries`, `catalyst_applications`, `cup_applications`, `access_applications` + RLS insert policies. |
| `drizzle/migrations/000{0,1,2}_*.sql` | `masterclass_sessions` (column-level grants, seed) and `masterclass_registrations` (+ role fields, + email status). |
| `content/verified-claims.md` | Claims ledger consumed by `copy-qa.py`. |
| `scripts/copy-qa.py` | Price/date/partnership claim gate. |
| `scripts/contrast-audit.py` / `a11y-audit.py` / `image-audit.py` | Playwright WCAG and image audits. |
| `src/server.ts` | Worker fetch entry with SSR error normalisation. |
| `src/start.ts` | Global middleware (auth attach, error page, CSRF). |
| `src/router.tsx` | Router + QueryClient context. |
| `src/routes/__root.tsx` | HTML shell, global meta/JSON-LD/fonts, header/footer, UTM capture, 404/error UI. |
| `src/routes/competition.index.tsx` | Innovate for Change Challenge 2026 page (Event schema, GSAP page). |
| `src/routes/dummy-comp.tsx` | Preserved Creating Livelihoods Challenge page. |
| `src/routes/access.index.tsx` / `access.apply.tsx` | Access Fellowship page and apply/nominate form. |
| `src/routes/catalyst.index.tsx` / `catalyst.apply.tsx` | Paused Catalyst pages (302 → `/`). |
| `src/routes/masterclass.index.tsx` / `masterclass.apply.tsx` | Masterclass list (server loader) and external redirect. |
| `src/routes/join.tsx` | Enquiry page. |
| `src/routes/faq.tsx`, `blog.index.tsx`, `blog.$slug.tsx` | FAQ and blog with structured data. |
| `src/routes/sitemap[.]xml.ts` | Sitemap server route. |
| `src/lib/seo.ts` | Canonical origin and JSON-LD builders. |
| `src/lib/sitemap.ts` | Required `staticData.sitemap` + XML builder. |
| `src/lib/utm.ts` | Attribution capture/carry-forward. |
| `src/lib/masterclass.functions.ts` | `listUpcomingMasterclasses`, `registerForMasterclass` server functions. |
| `src/lib/masterclass-mail.server.ts` | Gmail SMTP via `@portable/smtp` (Node vs Worker sockets). |
| `src/lib/masterclass-email.ts` / `masterclass-registration.ts` | Email template/recipients; client-safe zod schema. |
| `src/integrations/supabase/client.ts` / `client.server.ts` | Anon and service-role clients. |
| `src/integrations/supabase/types.ts` | Generated DB types (6 tables). |
| `src/components/bfl.tsx` | ~55 editorial/layout primitives shared by all pages. |
| `src/components/audience.tsx` | `EnquiryForm` → `enquiries`. |
| `src/components/apply-form.tsx` | Shared form building blocks for the three apply pages. |
| `src/components/site-header.tsx` / `site-footer.tsx` | Global chrome; header swaps to microsite headers by path. |
| `src/components/innovate-header.tsx` / `competition-header.tsx` / `catalyst-header.tsx` | Sticky programme microsite headers with anchor nav + apply CTA. |
| `src/components/innovate-fx/use-innovate-motion.ts` / `parts.tsx` | GSAP + ScrollTrigger + Lenis motion layer; countdown, doodles, video lightbox. |
| `src/data/blog-posts.ts`, `challenges.ts`, `mentors.ts`, `cup-curriculum.ts`, `catalyst-curriculum.ts` | Typed content modules. |
| `src/styles.css` | Tailwind v4 theme tokens, Brand v3 colour law, page-accent system, scoped page skins, motion CSS. |
| `public/robots.txt`, `public/docs/Builders Lab-Billion-Founders-Competition.pdf` | Crawl rules; the one remaining downloadable programme guide. |
| `.lovable/plan/*.md` (136) | Per-request Lovable plans — the de-facto design history for Aug–Nov 2026. |
| `.lovable/north-star/north-star.md` | Purpose and success metric. |

---

## 11. Undetermined / not verifiable from the repo

- Whether `builders.example.com` is already pointed at this Lovable deployment (only the code's `SITE_ORIGIN`, robots and GSC tag assert it).
- Which test runner the team uses for the `node:test` files (no script, no CI).
- Whether `qa:prepublish` is run before each publish (no CI/hook; several claims remain `pending`).
- Whether the Gmail sender secrets are still configured in Lovable Cloud now that the on-site masterclass form is retired.
- Whether `.output`/Cloudflare settings (e.g. `wrangler`) live only in Lovable's infrastructure (no config in repo).
