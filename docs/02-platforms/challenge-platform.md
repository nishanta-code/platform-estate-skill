# Challenge Platform (Challenge Platform / Global Future Innovators Challenge)

> Knowledge-skill reference for the `Challenge Platform` repository. Written read-only from the source
> tree and git history on 2026-11-24. No secret values are reproduced; env-var and secret *names*
> are listed where they matter.

---

## 1. Identity

| Item | Value |
| --- | --- |
| Local folder | `D:\the Group\Vs Code Studio\Challenge Platform` |
| GitHub remote | `https://github.com/<org>/<repo>` (branch `main`) |
| Lovable project | Editor link in `README.md`: `lovable.dev/projects/<uuid>`; Lovable preview domain `challenge-preview.lovable.app`; template `tanstack_start_ts_current` (`.lovable/project.json`) |
| Public brand | **Global Future Innovators Challenge (GFIC 2026): Challenge Platform** |
| Live domain | `https://challenge.example.com` (hard-coded as `BASE_URL` in `src/routes/sitemap[.]xml.ts`, and in `public/robots.txt` `Sitemap:` line) |
| Supabase project | `<ESTATE_REF>` (shared the Group estate) – `supabase/config.toml`, `src/integrations/supabase/client.ts`, `AGENTS.md` |
| Supabase schema | `public` only. `src/integrations/supabase/types.ts` is generated for `public`; there are **no** `.schema(...)` or `db: { schema }` calls anywhere in `src/`. No Future-Career-specific schema exists. |
| Env var names | `.env`: `SUPABASE_PROJECT_ID`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_URL`, `VITE_SUPABASE_PROJECT_ID`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `VITE_SUPABASE_URL`. Server code additionally reads `SUPABASE_SERVICE_ROLE_KEY`, `LOVABLE_CRON_SECRET`, `LOVABLE_CRON_SECRET_PREVIOUS`; `drizzle.config.ts` reads `LOVABLE_DB_MIGRATION_URL`. **None of the `VITE_*`/`import.meta.env` names are read by application code today** (the browser client hard-codes URL + publishable key). |
| Framework | TanStack Start (`@tanstack/react-start` 1.168.60) + TanStack Router 1.170.41 (file-based routes), React 19.3, Vite 8 with `@lovable.dev/vite-tanstack-config` 2.27.1, Nitro 3 beta (preset `cloudflare-module`) |
| Styling | Tailwind v4 via `@tailwindcss/vite` (`src/styles.css` with `@theme inline`), shadcn "new-york" config (`components.json`, only `button` + `popover` survive), hand-written CSS files `src/styles/home-reference.css` and `src/styles/masterclass.css`, Outfit webfont |
| Package manager | **bun** (`bun.lock`, `bunfig.toml` with a 24 h `minimumReleaseAge` supply-chain guard). README still says `npm i`; `.output/package-lock.json` exists from a Lovable build. |
| Tests | vitest 5 (`tests/`, most `src/**/*.test.ts`) plus some `node:test` files; no `test` script in `package.json` |
| Commits | **876** on `main`, first `2026-08-09` ("Initial commit from remix"), last `2026-11-23` ("Merge branch 'feature/shared-auth-hardening'") |
| Authorship | 862 commits by the Lovable bot, 14 by humans (the human commits are the UTM/fee fixes in Nov, the `/nz` page, and the AGENTS.md shared-backend note) |

### Commits by month

```
  17  2026-08   (template remix; a "Deep Work timer" theme experiment, then reset)
 705  2026-10   (the entire site was built 9–27 Oct; nothing between 9 Aug and 9 Oct)
 154  2026-11   (fees/UTM fixes, masterclass, backend removed, NZ cohort, docs)
```

---

## 2. Business purpose

Source of truth for every fact: `src/config/challenge.ts` (single config object `challenge`, a derived
`nzChallenge`, and a separate `masterclass` object). `AGENTS.md` mandates that unconfirmed facts stay
`null` and render as an explicit "to be confirmed" state rather than being invented.

**What it is.** A four-weekend, live-online "invention challenge" for school students in grades 5 to 12
worldwide, run by the Group, "led by university student mentors", "in association with" HPAIR
(partner logos: `hpair`, `sales`). Tagline: *"Explore Challenge Platforms. Build Future Skills. Solve
Tomorrow's Challenges."* North star (`.lovable/north-star/north-star.md`): "Students invent and
present their own idea for solving a problem of the future"; metric = students completing registration
per month.

**Tracks / problems.** There are no fee "tracks"; instead six 2035 problem statements
(`src/config/problems.ts`: Orbit, Sky, Heat, Health, Robots, Agents), each with scene / problem / today /
trap / unsolved / jobClue, plus 14 "fields" x 2 horizon years (2035, 2126) of sample ideas in
`src/data/ideas.ts`, and six "imagined jobs" cards. Students pick a field, imagine it in 2035 (semifinal
round) and 2126 (final round). Copy is deliberately restricted to only those two years (tested).

**Rounds / weekends (GFIC 2026, US-East time).**

| Weekend | Date | What happens |
| --- | --- | --- |
| 1 Orientation + case brief | Sat 5 Dec 2026, 9:00–10:00 a.m. ET | meet mentors, read six problems |
| 2 Idea submission + discussion | Sat 12 Dec 2026 | every team submits, live discussion |
| 3 Semifinals (2035) | Sat 19 Dec 2026 | every team presents; top 10 advance |
| 4 Finals (2126) | **Sun** 3 Jan 2027 | top 10 present to judges; winners announced |

**Divisions.** Explorers (grades 5–7) and Pioneers (grades 8–12); `divisionForGrade()` in
`src/lib/registration.ts`. Solo or teams of up to 5, cross-school/cross-country allowed.

**Fees (USD, per student).** `fees: { earlyBird: 80, regular: 100 }`; early bird deadline
`2026-11-30T23:59:59-05:00` (changed from 15 Nov on 2026-11-16, and from 60/80 to 80/100 on
2026-11-04). Refund disclosure (5 % non-refundable processing fee) is config-backed and rendered as a
Radix popover under every price (`src/components/site/RefundPolicy.tsx`). `paymentUrl` is the literal
placeholder `'[PAYMENT LINK]'` – there is **no checkout** in this repo.

**Prizes.** `prizePool: 500`, `prizeWinnerCount: 3`, certificate for every participant, a "skill card"
concept (five skills scored by two judges, issued by the Group).

**Cohorts.**
- `challenge` – GFIC 2026, worldwide, `/`.
- `nzChallenge` – "GFIC 2027 · NZ" New Zealand edition on `/nz` (added 2026-11-20): Sat 16, 23, 30 Jan
  and Sun 7 Feb 2027, 11:30 AM–12:30 PM NZDT, early bird until 10 Jan 2027 (+13:00), "summer break"
  copy. Same `applyUrl`, same fees.
- `masterclass` – free 60-minute "What Young Innovators Do Differently" session, Sun 29 Nov 2026
  9:00 a.m. ET (`key: 'young-innovators-2026-11-29'`), for students/parents/teachers.

**AI policy.** `ai: { optional: true, competition: false, allowedTools: [...] }` – six named tools; copy
must say "no coding needed, AI optional" (enforced by `src/config/challenge.test.ts`).

---

## 3. Frontend architecture

### 3.1 Routes (`src/routes/`) – every file

| File | URL | Purpose |
| --- | --- | --- |
| `__root.tsx` | – | Root shell: `<html>` document via `shellComponent`, `HeadContent`/`Scripts`, global meta (`og:site_name`, `twitter:card`), Outfit Google-font link, favicon links, `QueryClientProvider`, 404 and error boundary components, and the **one-time install of UTM carry + outbound UTM tagging** (`installUtmCarry`, `installOutboundUtmTagging`). `staticData.sitemap: false`. |
| `index.tsx` | `/` | Homepage: SEO head from `challenge.seo`, renders `<ReferenceHome />`. `sitemap: true`. |
| `nz.tsx` | `/nz` | New Zealand cohort page: identical component with `config={nzChallenge}`. `sitemap: true`. |
| `masterclass.tsx` | `/masterclass` | Free masterclass landing page built from `masterclass` config inside `MasterclassShell`; every "REGISTER FOR FREE" button is an external `<a href={masterclass.applyUrl}>`. `sitemap: true`. |
| `sitemap[.]xml.ts` | `/sitemap.xml` | Server route (`server.handlers.GET`) that builds XML from the live router via `src/lib/sitemap.ts`; 404 if nothing is public; `Cache-Control: public, max-age=3600`. `sitemap: false`. |
| `README.md` | – | Lovable's routing conventions note (file-based, `$param`, `_layout`, don't edit `routeTree.gen.ts`). |

`src/routeTree.gen.ts` confirms exactly four routes: `/`, `/masterclass`, `/nz`, `/sitemap.xml`.
Deleted routes still visible in history: `/register` (multi-step registration form, 9 Oct–16 Nov),
`/postcard` (postcard-from-2126 with image upload, 9 Oct–16 Nov), `/ver2`, `/ver3` (design variants,
19 Oct only), `/masterclass/apply` (16–17 Nov), `/edge-logo`, `/program-cover` (asset redirects).

### 3.2 App shell and server entry

- `src/router.tsx` – `getRouter()` creates a `QueryClient` and `createRouter({ routeTree, context, scrollRestoration: true, defaultPreloadStaleTime: 0 })`.
- `src/start.ts` – `createStart()` with two request middlewares: an error middleware that renders `renderErrorPage()` HTML on non-HTTP errors, and `createCsrfMiddleware({ filter: handlerType === 'serverFn' })` (explicitly re-added because defining `start.ts` opts out of the default).
- `src/server.ts` – custom server entry (wired via `tanstackStart.server.entry: "server"` in `vite.config.ts`). Lazily imports `@tanstack/react-start/server-entry`, wraps `fetch` in try/catch, and detects h3's swallowed `{"unhandled":true,"message":"HTTPError"}` JSON 500s to replace them with a friendly HTML error page while logging the real stack recovered from `src/lib/error-capture.ts`.
- `src/lib/error-capture.ts` – monkey-patches `console.error` to expand Error cause chains and record the last error for 5 s; hooks `error`/`unhandledrejection`.
- `src/lib/error-page.ts` – static HTML "This page didn't load" fallback.
- `src/lib/lovable-error-reporting.ts` – forwards boundary errors to `window.__lovableEvents` / `__lovableReportRuntimeError` (editor preview only).
- **No `createServerFn` exists in the current tree** (grep confirms). All former server functions were deleted on 2026-11-16/17.

### 3.3 Homepage rendering model (important)

The live homepage is **not** built from the React section components. `src/components/site/ReferenceHome.tsx`:

1. Imports two raw HTML templates: `src/content/home-main.html` (~90 KB, one line, inline styles) and `src/content/home-closing.html` (~5 KB) with `?raw`.
2. `fill(html, config)` substitutes `{{token}}` placeholders (43 distinct tokens: `registerUrl` x13, `calendlyUrl`, `earlyBird`, `regular`, `saving`, `earlyDate`, `w1..w4`, `finaleUpper`, `timeShort`, `prizePool`, `winners`, `aboutBody`, `regionEyebrowHtml`, `homeHref`, `hpairLogo`, `email`, ...). Results are memoised per config in a `WeakMap`.
3. `getMarkup()` string-patches the template to inject hosts: `#mobile-menu-host`, `#programme-dean-host`, adds `class="ref-colour-hero"` and `ref-colour-ring`.
4. The markup is rendered with `dangerouslySetInnerHTML`; React components are then **portalled** into the hosts: the mobile menu toggle `Button`, `<FclDean>` (programme dean band), `<RefundPolicy>` under the hero price and `#schools h2`.
5. `<FclMentors>` (auto-advancing carousel, pauses on hover/focus/reduced-motion) and `<FclFaq>` (`<details>` list) are rendered as real React after the main template; then the closing template (footer).
6. A hero "ring dial" animation is driven in an effect: every 2.6 s it rotates `.ring` SVGs by 60°, cycles `data-palette` 0–5 on the hero (colour tokens in `home-reference.css`), toggles the active orbit label and swaps the career pill text (`careers[]` array: "Space lawyer", "Sky traffic planner", ...). Stopped under `prefers-reduced-motion`.
7. Click tracking: any anchor whose href is `/register` or `config.applyUrl` fires `track('register_click', { location })`.

Section ids in the template: `top`, `banner`, `about`, `tracks`, `discover`, `story`, `how`, `schools`,
`register`; nav links `#about #tracks #how #schools #mentors #faq` and `/masterclass`.

**Dormant React components** (present, tested, but not imported by any route): `Hero.tsx` (with
`useCountdown`), `Glance.tsx`, `TheChallenge.tsx`, `HowItWorks.tsx`, `IdeaGalaxy.tsx` (six problems),
`AfterIdeas.tsx` (`WaysIn`), `SkillCard.tsx`, `LowerSections.tsx` (`Mentors`, `PrizesAndFees`, `Faq`,
`FinalCta`), `SiteLayout.tsx`, `SiteNav.tsx`, `SiteFooter.tsx`, `SectionHeading.tsx`, `Icons.tsx`,
`ui/sunset-skyline-hero.tsx` (`FutureCityScene` 2126 city video band) and `ui/orbit-delivery-hero.tsx`.
They are the pre-19-Oct "config-driven" homepage that the HTML-reference redesign replaced.

### 3.4 Data access layer

- `src/integrations/supabase/client.ts` – browser client for `<ESTATE_REF>` with a hard-coded publishable key (value not reproduced here) and `brokeredPreviewStorage()` auth storage. **Not imported by any route/component** (grep). Generated by Lovable; "do not edit".
- `src/integrations/supabase/client.server.ts` – lazy `supabaseAdmin` Proxy using `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY`, with a fetch shim that strips `Authorization: Bearer <sb_secret_…>` for new opaque keys. Unused today.
- `auth-attacher.ts`, `auth-middleware.ts` (`requireSupabaseAuth` using `auth.getClaims`), `cron-auth.ts` (`LOVABLE_CRON_SECRET` timing-safe compare), `previewAuthStorage.ts` (postMessage broker to the Lovable editor) – all Lovable template scaffolding, all unused.
- `src/integrations/supabase/types.ts` (4 099 lines) – generated `Database` type of the **shared `public` schema**, so it documents the whole estate (see §6).
- `src/lib/utm.ts` – the only "data" layer actually exercised in production (sessionStorage key `fe_utm_v1`).
- `src/lib/analytics.ts` – `track(name, props)` pushes to `window.dataLayer`, calls `gtag` if present, and dispatches a `fe:analytics` DOM event. Event names: `register_click`, `idea_ticket_viewed`, `registration_completed`, `postcard_submitted`. No GTM snippet is installed in the repo; nothing listens unless a tag is injected at the edge.

### 3.5 Auth

None. `AGENTS.md` (added 2026-11-23): the project shares `auth.users` with Ops Tool, Application Portal,
Pre-Med Platform, Growth Partner Portal and Learner Portal, "This repo has no login/auth UI of its own today", and if
that changes: never `supabase.auth.signUp()` from the browser, look emails up server-side first, don't
build a new cross-platform profile/role table.

### 3.6 Design tokens

`src/styles.css` (1 016 lines):
- `@import "tailwindcss" source(none); @import "tw-animate-css"; @import "./styles/home-reference.css"; @import "./styles/masterclass.css";`
- `@theme inline`: `--font-sans: 'Inter'`, `--font-heading: 'Plus Jakarta Sans'`, radii 6/12/16 px, colour aliases mapping Tailwind `--color-*` onto CSS variables.
- Base `:root` block (lines 39–95): oklch tokens, `--content-width: 1200px`, `--section-gutter: clamp(24px,3vw,40px)`, `--page-edge: max(gutter, (100% - content)/2)` (AGENTS: one shared content-width/page-edge token for nav, hero, sections, footer), legacy `--ver2-modern-*` teal/violet/coral tokens.
- **"Challenge Platforms Lab" override block** at line 913 onward (AGENTS rule): `--fcl-paper` warm off-white, `--fcl-ink` near-black, `--fcl-night` black bands, `--fcl-gold` / `--fcl-gold-deep`, `--fcl-sky` / `--fcl-sky-deep`, `--fcl-line` hairline; it re-points `--primary`, `--brand-*`, `--gold` etc. so the shared `Button` becomes a black pill with pale-gold arrow badge.
- `src/styles/home-reference.css` (300 lines): `.fcl-ref` uses `font-family:'Outfit'`; six hero palettes via `.ref-colour-hero[data-palette="n"]` (yellow/amber, emerald, sky/blue, orange, lime/green, cyan/indigo); phone-only layout scoped **below 761 px** (`@media (max-width: 760px)`); `.ref-menu-toggle`, dean band `.fx-dean-*`, mentor slider `.fx-m*`, FAQ `.fx-*`.
- `src/styles/masterclass.css` (240 lines): `.mc` scope with `--mc-ink/paper/warm/gold/line/muted/crimson` oklch tokens, `.mc-pill` (black pill + gold `.disc`), `.mc-editorial` sections (`mc-intro`, `mc-session-strip`, `mc-band`, `mc-tile`, `mc-faq`).
- Fonts actually loaded: Outfit (Google Fonts link in `__root.tsx`). Inter / Plus Jakarta Sans are referenced by tokens but no `<link>` loads them (system fallback), which only matters for dormant components.
- `src/components/ui/button.tsx` – cva variants `default | teal | secondary | outline | ghost`, sizes `default | lg | icon`, `asChild` implemented via `cloneElement` (no Radix Slot).

### 3.7 Responsive patterns

- Phone layouts scoped under 761 px; `ReferenceHome` toggles `nav[data-open]` and closes on outside click, Escape, link click, or crossing the 760 px media query.
- `FclMentors` computes `perView` 1/2/3 at <700 / <1000 / else.
- Masterclass page uses `clamp()` typography and `.hide-sm` for the HOME pill.
- `roadmap.md` records verification at 360/768/1440 px and "ten widths".

### 3.8 SEO / AEO

- Per-route `head()` with `title`, `description`, `og:title`, `og:description`, `og:type`, `twitter:card`; root adds `og:site_name` = organiser. **No `og:image`, no canonical link, no JSON-LD** in the current tree (an `og-home.jpg.asset.json` exists in `src/assets` but is not wired).
- `/sitemap.xml` is router-derived: `src/lib/sitemap.ts` declares `StaticDataRouteOption.sitemap: boolean | 'exclude-subtree'`; only routes with `staticData.sitemap === true` and no `$`/`*` segments are listed, no `<lastmod>` (AGENTS: "omit dates without authoritative timestamps"). Tested in `src/lib/sitemap.test.ts`.
- `public/robots.txt` allows all major bots and points to `https://challenge.example.com/sitemap.xml`.
- No prerender/SSG config; pages are SSR'd by Nitro at request time.
- `public/favicon.svg` + `favicon.png`.

### 3.9 3D / animation

- `src/components/ui/orbit-delivery-hero.tsx` (1 168 lines, `// @ts-nocheck`, bundled single-file component from 21st.dev): a `@react-three/fiber` `Canvas` scene that loads `public/orbit-assets/models/whimsical-world.glb` (planet) and `courier.glb` (running character) with `GLTFLoader` + DRACO, drag-to-spin planet physics (`stepPlanet`, `stepRunner`), cloud bank, about dialog, reduced-motion handling, light/dark tokens. Added 2026-10-12 ("Added 3D hero component"); **never imported by a route** today. This is why `three` and `@react-three/fiber` remain in `package.json`.
- Earlier attempts: `RealisticGlobe.tsx` + `public/globe/student.glb` ("Added 3D Earth hero component", 2026-10-13, reverted twice the same day), `Starfield.tsx`, `three-object.ts` – all deleted.
- Current homepage motion is CSS/SVG only: rotating ring dial + palette cycling (see §3.3), mentor carousel, and (dormant) `FutureCityScene` video band with IntersectionObserver autoplay and a poster for reduced motion.
- Hero videos (`hero-city-*.mp4/webm`, `city-2050-*.webm`) live in `src/assets/*.asset.json` (Lovable-hosted asset manifests) – currently only `city-2050-generated.webm` is referenced via `challenge.futureCity.video`, which no route renders.

### 3.10 Image handling (fast-png / jpeg-js / fflate)

These three dependencies were added on 2026-10-22 for the now-deleted `src/lib/postcard-image.server.ts`:
server-side re-encoding of uploaded postcard pictures with bounded dimensions (`MAX_IMAGE_PIXELS = 4_000_000`),
a PNG chunk whitelist (`IHDR/PLTE/tRNS/IDAT/IEND`) and an `fflate` `Unzlib` streaming guard against
decompression bombs before `fast-png` decode / `jpeg-js` decode, then re-encode and store privately
(AGENTS rule "Decode and re-encode postcard images server-side with bounded dimensions and
decompression; keep storage private"). The postcard feature was removed on 2026-11-16, so the packages
are now **unused leftovers**, as are `recharts`, `embla-carousel-react`, `react-hook-form`, `vaul`,
`cmdk`, `input-otp`, most `@radix-ui/*` packages, `date-fns`, `sonner`, `html-react-parser`.

---

## 4. Backend

### 4.1 Supabase client configuration

- Project `<ESTATE_REF>`, schema `public`, URL + publishable key hard-coded in `client.ts`; server client from `SUPABASE_URL` / `SUPABASE_SERVICE_ROLE_KEY`.
- Connected to the shared project on 2026-11-16 ("Connect Supabase project the Group - Website") – that commit swapped `client.ts`, regenerated `types.ts` (+3 980 lines) and changed `supabase/config.toml`.
- Before that the repo used a Lovable Cloud built-in database (drizzle migrations below).
- **Runtime dependency: none.** AGENTS.md final rule: "omit local application routes, submission functions and Supabase middleware so the site has no database runtime dependency." `roadmap.md`: "no database requests" verified.

### 4.2 Tables this repo has created

**Legacy Lovable-Cloud database** (`drizzle/migrations/`, journal entries 0–4; `drizzle/schema.ts` is intentionally blank; `drizzle.config.ts` targets `LOVABLE_DB_MIGRATION_URL`) – these tables are NOT in the shared project:

| Migration | Table | Notes |
| --- | --- | --- |
| `0000_migration.sql` | `public.teams` (6-char `[A-Z0-9]` code, unique), `public.registrations` (grade 5–12, division Explorers/Pioneers, team FK, parent fields, `fee_due CHECK IN (60, 80)`) | RLS on; `anon/authenticated` INSERT only `WITH CHECK (consent_given AND agreed_terms)` |
| `0001_create_postcards.sql` | `public.postcards` (first_name, grade, country, parent_email required if grade ≤ 7, message or image_path) | service_role only |
| `0002_restrict_private_submission_table_grants.sql` | revokes everything except INSERT on registrations from anon/authenticated | security hardening 2026-10-22 |
| `0003_add_utm_fields.sql` | adds five `utm_*` columns (NOT NULL DEFAULT 'unknown') to registrations and postcards | 2026-11-02 UTM carry-forward |
| `0004_create_masterclass_registrations.sql` | `public.masterclass_registrations` (role student/parent/educator, child_name, designation, utm_*) | 2026-11-16, insert-only policy `consent_given = true` |

**Shared project** (`supabase/migrations/`, applied to `<ESTATE_REF>`):

| Migration | Effect |
| --- | --- |
| `20261116134259_…sql` | `CREATE TABLE public.futureedge_masterclass_regs` mirroring `finedu_masterclass_regs` (session_key, session_keys[], selected_sessions, role, grade, student_*, parent_*, designation, consent_given, utm_* x5, timestamps), GRANT INSERT to anon/authenticated, RLS insert-only policy, `set_updated_at()` trigger |
| `20261116135537_…sql` | "Fixed security finding f840ae5a": drops the public insert policy and revokes INSERT from anon/authenticated (service_role only) |
| `20261117070909_…sql` | `DROP TABLE IF EXISTS public.futureedge_masterclass_regs` – table removed the next day when the form moved to the Application Portal |

Net result: **this repo owns zero tables in the shared project today.**

### 4.3 SQL functions, triggers, RLS, cron

- Only trigger ever created: `futureedge_mc_updated BEFORE UPDATE … EXECUTE FUNCTION public.set_updated_at()` (dropped with the table). `set_updated_at()` is an estate-level function, not defined here.
- RLS approach while tables existed: RLS enabled, anon/authenticated insert-only with `WITH CHECK`, never SELECT; reads only via service role on the server.
- No cron jobs; `cron-auth.ts` is template scaffolding.
- Estate RPCs visible in `types.ts` `Functions` (not called here): `admin_partner_contacts`, `append_doctors_add_on_payment`, `execute_split`, `get_or_create_student_id`, `has_role`, `legacy_question_form_data`, `merge_form_answers`, `resolve_student_id`, `student_names_match`.

### 4.4 Edge functions

`supabase/functions/` **does not exist**; there is no `_shared/` folder, no `verify_jwt` entries
(`supabase/config.toml` is a single `project_id` line), no `Deno.env.get` usage, no webhooks. All
server-side work that ever existed ran as TanStack `createServerFn` handlers inside the Nitro
(Cloudflare) worker, not as Supabase edge functions.

### 4.5 External integrations (by name)

| Integration | Where | Secret / config name |
| --- | --- | --- |
| LwL Application Portal | `challenge.applyUrl`, `masterclass.applyUrl` | none (public URLs) |
| Calendly | `challenge.calendlyUrl` (with baked-in UTMs `futurecareerchallengewebsitedirect`) | none |
| Google Fonts (Outfit) | `__root.tsx` | none |
| Supabase (dormant) | `client.server.ts`, `auth-middleware.ts` | `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_PUBLISHABLE_KEY` |
| Lovable cron (dormant) | `cron-auth.ts` | `LOVABLE_CRON_SECRET`, `LOVABLE_CRON_SECRET_PREVIOUS` |
| Lovable Cloud DB migrations (legacy) | `drizzle.config.ts` | `LOVABLE_DB_MIGRATION_URL` |
| Planned but never shipped | `.lovable/plan/send-masterclass-registrations-…md` | `EXTERNAL_SUPABASE_URL`, `EXTERNAL_SUPABASE_SERVICE_KEY` (the FinLit pattern; superseded by connecting the repo directly to the shared project) |
| GTM / gtag | `src/lib/analytics.ts` | none in repo |

---

## 5. Key end-to-end flows

### 5.1 Visit → Register → Application Portal (the only conversion path)

1. Visitor lands on `/` (or `/nz`) possibly with `?utm_*` params. `RootComponent` effect runs `installUtmCarry()` (`src/lib/utm.ts`):
   - `captureUtm()` parses the query (case-insensitive keys, `&amp;` tolerated, each value trimmed to 200 chars, missing fields = `'unknown'`) and merges with sessionStorage `fe_utm_v1` using **first-real-touch-wins** (`mergeFirstTouch`).
   - It monkey-patches `history.pushState/replaceState`, retags the current URL on load and `popstate`, and rewrites `href` on capture-phase `click/auxclick/contextmenu` for same-origin links – except paths matching `/(admin|auth|login|signin|signup|checkout|payment|pay)`.
2. `installOutboundUtmTagging(isOutboundTarget, skipOutbound)` is installed with the hosts of `challenge.applyUrl`, `nzChallenge.applyUrl`, both `calendlyUrl`s and `masterclass.applyUrl` (i.e. `apply.example-group.com`, `calendly.com`). At click time on a cross-origin link to one of those hosts it calls `withUtm(raw, storedUtm, location.href, override = true)` – stored real tags **override** any UTMs already on the destination; `'unknown'` fields are never written. Paths matching `(pay|payment|checkout|auth|login|admin)` or the `paymentUrl` host are skipped.
3. Every "Register" CTA in `home-main.html` / `home-closing.html` is `href="{{registerUrl}}"` → `challenge.applyUrl` = **`https://apply.example-group.com/apply/global-future-career-challenge`**. There is no programme/cohort code parameter: the slug `global-future-career-challenge` in the Portal path identifies the programme; the Portal decides cohort/pricing. `/nz` uses the same URL (no NZ-specific slug yet).
4. `ReferenceHome` fires `track('register_click', { location: <section id> })` to `dataLayer`.
5. The Application Portal (`apply.example-group.com`) handles the form, payment, promo codes and early-bird logic. **This repo's fee display (`80/100`, deadline 30 Nov) must be kept in sync by hand with the Portal/catalogue** – nothing reads `all_programs.early_bird_price` at runtime.

Tests: `src/lib/utm.test.ts` (8 cases incl. "overrides tags on outbound registration links",
"carries x/y/z through links, history and back navigation without changing first touch").

### 5.2 Masterclass registration

`/masterclass` → "REGISTER FOR FREE" pills (hero, about, FAQ, navbar in `MasterclassShell`) →
`masterclass.applyUrl` = **`https://apply.example-group.com/forms/futureinnovators-masterclass`**,
tagged with the five UTMs at click time (tested in `src/lib/masterclass.test.ts`). The dark band links
to `challenge.applyUrl`. Nothing is written locally.

History of this flow (16–17 Nov 2026, see `.lovable/plan/*masterclass*`):
`/masterclass/apply` local form → `masterclass_registrations` (local DB) → `futureedge_masterclass_regs`
(shared) → `masterclass_regs` (shared, with `masterclass_id`, `partner_id`, `program_category:
'challenge-group'`, `form_data.source: 'challenge.example.com/masterclass/apply'`) → **external Portal form**
(final). The catalogue row created in `all_masterclasses` was kept ("preserved the catalogue listing
linked to three real registrations" – `roadmap.md`).

### 5.3 Registration form (removed)

`/register` (9 Oct–16 Nov) was a multi-step form validated by `registrationSchema`
(`src/lib/registration.ts` – still present: grade 5–12, E.164-style parent phone, team create/join with
6-char codes, `consent`/`agreedTerms` literal `true`, `utm` sub-object) and submitted through
`submitRegistration` / `checkTeamCode` server functions into `teams` + `registrations` with
`fee_due = feeFor(now)`. Removed when the Portal became the single application path.

### 5.4 Postcard from 2126 (removed)

`/postcard` collected first name, grade, country, parent email (if ≤ grade 7), 100-word message or a
≤5 MB image, feature consent; image sanitised server-side (§3.10) into private storage, row in
`postcards`. Config block `challenge.postcard` and analytics event `postcard_submitted` survive.

### 5.5 Interactive features still live

- Hero ring dial + career pill rotation (§3.3).
- Mentor carousel with dots/arrows, auto-advance 4.5 s.
- FAQ `<details>` accordions (homepage and masterclass).
- Refund-policy popover (hover on mouse, tap on touch, Escape/outside dismiss).
- Mobile menu toggle.
- Dormant: field/year idea selector (`IdeaGalaxy`, `ideas.ts`), countdown to early-bird (`useCountdown`), 2126 city video reveal, 3D orbit planet.

---

## 6. Shared catalogue: reads and writes

**Reads: none at runtime.** The homepage, NZ page and masterclass page are fully config-driven from
`src/config/challenge.ts`; no component queries `all_programs`, `all_cohorts`, `all_partners` or
`all_masterclasses_public`. The generated `types.ts` nevertheless documents those objects (columns of
interest for future wiring):

- `all_programs`: `program_name, partner_id, apply_url, pay_slug, sticker_price, early_bird_price, price_currency, alt_currency, eligibility_grades, format, duration, location, categories, cover_image_url, is_active`.
- `all_masterclasses_public` (view): `title, slug, scheduled_at, ends_at, display_timezone, duration_minutes, host_name, host_university, host_mentor_id, partner_name, partner_logo_url, program_name, apply_url, capacity, status, series, session_number, learning_outcomes, recording_url`.
- `all_cohorts`, `all_partners`, `all_mentors`, `all_sessions`, `all_events`, `all_conversions`, `all_visitors`, `all_utm_sources`, `all_utm_mediums`, `promo_codes`, `promo_redemptions`, `stripe_payments`, `razorpay_payments`, `*_applications`, `*_masterclass_regs` for doctors/finlit/lawyers/lwl/onward/purpose/sat/sirluther/teo, `offsite_program_inquiries`, views `codes_dropdown`, `program_codes`, `sirluther_*`.

**Writes: none at runtime.** Historically (17 Nov, a few hours) it wrote `public.masterclass_regs`
rows with `masterclass_id` `<lovable-project-id>`, `partner_id`
`<lovable-project-id>`, `program_category 'challenge-group'`, `status 'registered'`,
`attendee_type`, five `utm_*`, and a `form_data` JSON (`source`, `session_key`, `session_title`,
`session_starts_at`, `session_label`, `answers[]`). Three real rows reportedly remain.

**Own-schema filtered views:** none (no Future-Career schema exists).

---

## 7. Conventions, tests, scripts, deployment

### 7.1 `AGENTS.md` – key rules (quoted/abridged)

- Lovable banner: never force-push/rebase/amend published history; pushes to `main` sync into Lovable.
- "This project's Supabase connection points at the shared the Group estate project (`<ESTATE_REF>`) … one email/account across those platforms. This repo has no login/auth UI of its own today (registrations are a public insert-only write, never read back to the browser)."
- "Keep masterclass presentation scoped to `.mc-editorial` … so redesigns do not alter registration or other pages."
- "Keep the server code-splitting guard self-contained in vite.config.ts … disable splitting in both server passes to prevent circular ESM startup failures while preserving browser splitting."
- "Reuse the shared config-backed refund disclosure and Radix popover beneath prices; portal it into stable reference HTML hosts."
- "Pin vulnerable transitive packages through compatible package overrides and verify the lockfile with the dependency scanner after updates."
- "Derive sitemap URLs from required route-level staticData.sitemap decisions … omit dates without authoritative page-specific timestamps."
- "Keep phone-only reference layouts scoped below 761px; portal the shared Button into a memoised HTML navigation host."
- "Keep editable challenge facts, mentor identities and destination links in `src/config/challenge.ts` so all displayed details share one source of truth."
- "Leave unconfirmed challenge facts null and display an explicit pending state rather than inventing dates, fees, prizes or registration destinations."
- "Derive Key dates only from configured weekends; show registration status and the offer deadline separately."
- "Homepage uses the Challenge Platforms Lab style (Outfit, white and black bands, pill buttons) as an override block at the end of src/styles.css."
- "Carry UTMs through src/lib/utm.ts (installed once in the root route): first real capture wins in sessionStorage, missing fields are 'unknown', internal URLs and configured outbound registration links get the carried tags, and every form submission saves all five fields."
- "Resolve masterclass registration from the config-owned external URL with carried UTMs; omit local application routes, submission functions and Supabase middleware so the site has no database runtime dependency."
- Copy rules recorded in `.lovable/plan/*`: British spelling, no em dashes, "university student mentors" wording, only 2035/2126 as future years, no claims about Python/repositories/prototypes.

### 7.2 `roadmap.md`

A reverse-chronological checklist of ~60 Lovable tasks (all `[x]` except one): the open item is
"Verify recovery on challenge.example.com after redeploying the fix (blocked: publication has not been
requested)" – i.e. the `__exportAll` server-startup fix and later changes may not have been published
from Lovable. Also notes "Automated email remains unconfigured" and "Automated submission throttling
remains a platform limitation".

### 7.3 Tests

- Runner: vitest 5 (`tests/server-runtime.test.ts`, `src/lib/utm.test.ts`, `src/lib/sitemap.test.ts`, `src/lib/masterclass.test.ts`, `src/config/refund-policy.test.ts`) **and** Node's built-in `node:test` + `node:assert` (`src/config/challenge.test.ts`, `src/lib/registration.test.ts`, `src/lib/fees.test.ts`, `src/lib/timeline.test.ts`, `src/components/ui/sunset-skyline-hero.test.ts`). Vitest picks up both styles because `node:test` files still execute under it only if imported; in practice run `bunx vitest run` (roadmap mentions "15 tests", "31 tests", "48 regression tests" at different points). No `vitest.config.*` and no `"test"` script.
- What is tested: programme facts and copy guards (`challenge.test.ts`, 28 cases: fees 80/100, prize 500/3 winners, team ≤ 5, dates, 2035/2126-only, NZ cohort times incl. "9:30 AM AEDT" equivalence, nav/footer "BY FUTURE EDGE", strike-through fee rendering of the HTML template), fee presentation, early-bird cut-over at ET midnight, division mapping, timeline next-milestone selection, UTM parsing/merging/rewriting, sitemap inclusion rules and XML, masterclass date/URL/UTM carry, refund wording, `FutureCityScene` null-safety, and a rolldown fixture proving the single-chunk server bundle boots with CommonJS helpers (`serverRuntimeOutput`).

### 7.4 Scripts and tooling

`package.json`: `dev` (vite dev), `build`, `build:dev`, `preview`, `lint` (eslint 9 flat config with
react-hooks, react-refresh, prettier; bans `server-only` imports), `format` (prettier, printWidth 100,
double quotes). `overrides`: `js-yaml 4.3.2`, `source-map-js 1.2.2` (vulnerability pins, 2026-10-22).
`tsconfig.json` is strict with `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` (hence the
`!` non-null assertions in `nzChallenge`).

### 7.5 Build and deployment

- `vite.config.ts` wraps `@lovable.dev/vite-tanstack-config` (which bundles tanstackStart, viteReact, tailwind, tsconfig paths, nitro with Cloudflare target, VITE_* injection) and adds one plugin `challenge-group:server-runtime-bundle` that sets `rolldownOptions.output.codeSplitting = false` for the `ssr` and `nitro` environments (fix for "__exportAll is not a function" production crash, 2026-11-17).
- `.output/nitro.json`: preset `cloudflare-module`, `nodeCompat: true`, `serverEntry: server/index.mjs`, deploy via `npx wrangler deploy`; `.wrangler/deploy/config.json` points at `.output/server/wrangler.json`. Lovable publishes to `challenge.example.com`; local `.output` is from a 2026-11-17 build.
- Lovable sync: commits on `main` flow both ways; 98 % of commits are bot "Changes"/"Work in progress"/"Update plan" messages. Human commits were pushed via a feature branch (`feature/shared-auth-hardening`) and merged on 2026-11-23.
- `.lovable/plan/*.md` are Lovable's plan documents; `north-star.md` must be kept (Lovable note).
- `.gitignore` excludes `.output`, `.tanstack`, `.wrangler`, `supabase/.temp/`.

---

## 8. Timeline and milestones (from `git log`)

| Date | Milestone |
| --- | --- |
| 2026-08-09 | Repo created from a Lovable remix; a "Deep Work timer" theme experiment, reset to slate (17 commits) |
| 2026-10-09 | Site actually starts: "Added winter challenge page", base + nav, hero, How It Works, Idea Galaxy, `/register` step form, `/postcard` page, first drizzle migrations (`teams`, `registrations`, `postcards`) |
| 2026-10-12 | Dark theme, 3D hero component (`orbit-delivery-hero.tsx`, three + r3f added), HPAIR/LwL logos, 48 reference images, Dr. Jojo photo, 2050 city window section |
| 2026-10-13 | Hero globe iterations (RealisticGlobe, two "Reverted to commit 952f384" rollbacks), hero video, registrations/dates update, Financial Literacy Platform-style mentors |
| 2026-10-17 | Six problem statements, weekend structure, rounds 2035/2050, skill card section |
| 2026-10-19 | "Change every 2060 to 2050" check; `/ver2` and `/ver3` design variants; Challenge Platforms Lab homepage redesign plan; `/ver2`/`/ver3` removed the same day; prizes+fees merged |
| 2026-10-21/22 | HTML-reference homepage (`home-main.html`), cycled hero colours, Dr. Jojo section, "Global…" rename, Calendly link, sitemap, favicon, vulnerability fixes (`overrides`, postcard image hardening), auto-scroll mentors, meta tags |
| 2026-10-27 | "Fixed challenge wording claims" (no coding/AI-optional copy, test guard) |
| 2026-11-02 | UTM carry-forward implemented (+ migration 0003) |
| 2026-11-04 | Fees to USD 80 early bird / 100 regular; README |
| 2026-11-05 | Human fixes: stale early-bird tile past deadline; stop forcing stored UTM into address bar; restore outbound UTM tagging; ignore `supabase/.temp` |
| 2026-11-09 | Rebrand to "Global Future Innovators Challenge (GFIC 2027)" then corrected to GFIC 2026; new tagline; UTM carry restored |
| 2026-11-10 | Refund tooltip under prices |
| 2026-11-16 | Masterclass page + menu; `masterclass_registrations` (local); **Connect to shared Supabase project**; `futureedge_masterclass_regs` created then locked down (security finding f840ae5a); `/register` and `/postcard` deleted; early-bird deadline → 30 Nov |
| 2026-11-17 | "Removed backend from site"; table dropped; masterclass → `masterclass_regs` + catalogue entry; then "Point masterclass registration to the the Group application portal"; `__exportAll` server fix + Vite config fix; masterclass hero redesign; unused logo assets removed (~22 asset files deleted over the project's life) |
| 2026-11-20 | `/nz` New Zealand cohort page |
| 2026-11-23 | AGENTS.md shared-backend/auth note merged from `feature/shared-auth-hardening` |

Keyword counts in subjects (non-generic commits only): hero 25+, fee/price 12, mentors 10,
masterclass 14, UTM 6, 3D/globe 8, sitemap 1, supabase/DB 5, cohort/NZ 1, round/weekend 5.

---

## 9. Gotchas and lessons

1. **Fees and dates are hand-maintained constants.** `fees`, `earlyBirdDeadline.iso` (with explicit `-05:00` / `+13:00` offsets), weekend `startIso`, and `sessionTime` live in `src/config/challenge.ts` and are locked by tests in `src/config/challenge.test.ts`, `src/lib/fees.test.ts`, `src/lib/registration.test.ts`. Changing a date or price means editing config **and** tests, and the Application Portal / `all_programs` row must be updated separately – there is no shared pricing source (cf. the 2026-11-05 cross-repo early-bird fix recorded in the LwL memory). The 2026-11-05 "stale early-bird price past deadline" bug came from a component computing early-bird status without the deadline.
2. **Legacy `registrations.fee_due CHECK IN (60, 80)`** (drizzle 0000) predates the 80/100 change; irrelevant now the table lives only in the abandoned Lovable Cloud DB, but a trap if anyone re-applies those migrations.
3. **The homepage is an HTML template, not JSX.** Editing copy means editing `src/content/home-main.html` (one 90 KB line) or the `fill()` token map in `ReferenceHome.tsx`; the React section components in `src/components/site/*` are dormant and will silently diverge. Portals depend on string replacements (`<header style=`, `<section id="tracks"`, `stroke="#DCEBFD"`), which break if the template markup changes.
4. **Server bundle must not be code-split.** Production crashed with `__exportAll is not a function` (circular ESM chunks under Nitro/Cloudflare). The fix is the `isolateServerRuntime` plugin in `vite.config.ts`; `tests/server-runtime.test.ts` guards it. Keep the helper inside `vite.config.ts` – an earlier version importing from `build/` broke because that folder is not in deployment source ("Fixed missing config import").
5. **h3 swallows handler throws** into a JSON 500 `{"unhandled":true,"message":"HTTPError"}`; `src/server.ts` + `error-capture.ts` exist solely to surface the real stack. Keep `console.error` patch semantics in mind when debugging logs.
6. **`src/start.ts` opt-out.** Defining `start.ts` disables TanStack's automatic CSRF middleware; it is re-added explicitly. Any new server function must keep that.
7. **UTM rules are subtle.** First real touch wins for the session; outbound apply/Calendly links get *overridden* tags; internal links never get tags on `/admin|auth|login|checkout|pay` paths; `installOutboundUtmTagging` is guarded by a module-level `outboundInstalled` flag so hot reloads/double effects do not stack listeners. The 2026-11-05 regression ("stop forcing stored UTM into address bar") shows that rewriting `location` eagerly broke things; the current code only retags via `replaceState` when the URL would change.
8. **Lovable drift.** `README.md` still says `npm i`; the repo is bun-managed with `minimumReleaseAge = 86400` (new package versions younger than 24 h are refused – expect install failures right after a release) and an exclusion list for `@lovable.dev/*`. `package.json` `overrides` is honoured by bun/npm; keep both lockfiles consistent (an `.output/package-lock.json` is generated by the Lovable build). Human commits must be merged, not rebased, because Lovable mirrors history.
9. **Unpublished state.** `roadmap.md` flags that the production fix was "Not published" / "publication has not been requested" – verify `challenge.example.com` actually runs the current `main` before assuming.
10. **Hard-coded Supabase credentials in `client.ts`** (publishable key; generated file). Nothing imports it, but it ships to the browser bundle only if imported – keep it that way or move to `VITE_SUPABASE_*`.
11. **Dead dependencies** (three, r3f, fast-png, jpeg-js, fflate, recharts, embla, react-hook-form, vaul, cmdk, most radix) inflate install and audit surface; the only radix packages in use are popover (+ slot indirectly). `src/assets/*.asset.json` files are Lovable-hosted asset manifests (`{ url }`), not local binaries.
12. **Two test styles** (vitest + `node:test`) coexist; a `node:test` file run under vitest works only because vitest provides the module shim – do not assume `bun test` runs them identically.
13. **Catalogue leftovers.** An `all_masterclasses` row and three `masterclass_regs` rows tagged `program_category = 'challenge-group'` exist in the shared project from the 17 Nov experiment even though the site no longer writes there.
14. **`/nz` shares the apply URL** – the Portal cannot distinguish NZ applicants except via UTMs; consider a cohort-specific slug or `utm_campaign` when the NZ campaign launches.
15. **No `og:image`/canonical/JSON-LD** despite the SEO-conscious sitemap work; the hero OG asset exists in `src/assets/og-home.jpg.asset.json` but is unused.

---

## 10. File map (most important files)

| Path | One line |
| --- | --- |
| `AGENTS.md` | Lovable banner + shared-backend/auth note + 25 architecture rules (quoted in §7.1) |
| `roadmap.md` | Lovable task log; one open item about unpublished production fix |
| `README.md` | Lovable project link, preview URL, (stale) npm instructions |
| `package.json` | TanStack Start/React 19 deps, `overrides`, scripts (no test script) |
| `bunfig.toml` | bun install guard: text lockfile, 24 h minimum release age, Lovable exclusions |
| `vite.config.ts` | Lovable TanStack config + `isolateServerRuntime` (server `codeSplitting: false`) + `server.entry: "server"` |
| `drizzle.config.ts` / `drizzle/migrations/*.sql` | Legacy Lovable-Cloud DB: `teams`, `registrations`, `postcards`, `masterclass_registrations`, UTM columns |
| `supabase/config.toml` | `project_id = "<ESTATE_REF>"` |
| `supabase/migrations/2026111*.sql` | create → lock down → drop `futureedge_masterclass_regs` |
| `src/config/challenge.ts` | **Single source of truth**: `challenge`, `nzChallenge`, `masterclass`, `partnerLogos`, SEO, fees, dates, mentors, FAQ, apply/Calendly URLs |
| `src/config/challenge.test.ts` | Fact and copy guards (28 tests) |
| `src/config/problems.ts` | Six 2035 problem statements |
| `src/data/ideas.ts` | 14 fields × 2035/2126 sample ideas |
| `src/routes/__root.tsx` | Document shell, head, fonts, error/404, installs UTM carry + outbound tagging |
| `src/routes/index.tsx` | `/` → `ReferenceHome` |
| `src/routes/nz.tsx` | `/nz` → `ReferenceHome config={nzChallenge}` |
| `src/routes/masterclass.tsx` | `/masterclass` landing page, external register links |
| `src/routes/sitemap[.]xml.ts` | Router-derived sitemap for `https://challenge.example.com` |
| `src/routeTree.gen.ts` | Generated route tree (4 routes) |
| `src/router.tsx` | `getRouter()` with React Query context |
| `src/start.ts` | Request middlewares: error page + CSRF for server functions |
| `src/server.ts` | Custom Nitro entry wrapping TanStack server entry; h3 error normalisation |
| `src/lib/utm.ts` | UTM capture/merge/rewrite engine (`fe_utm_v1`, five keys, first-touch) |
| `src/lib/sitemap.ts` | `staticData.sitemap` typing, path inventory, XML builder |
| `src/lib/fees.ts`, `src/lib/registration.ts`, `src/lib/timeline.ts` | Fee presentation, early-bird cut-over, zod registration schema (legacy), milestone helpers |
| `src/lib/analytics.ts` | `track()` → dataLayer/gtag/DOM event |
| `src/lib/error-capture.ts`, `error-page.ts`, `lovable-error-reporting.ts` | Error plumbing |
| `src/components/site/ReferenceHome.tsx` | Homepage renderer: HTML templates + token fill + portals + ring-dial animation |
| `src/content/home-main.html`, `home-closing.html` | The homepage markup (inline styles, `{{tokens}}`) |
| `src/components/site/FclExtras.tsx` | `FclDean`, `FclMentors` carousel, `FclFaq` |
| `src/components/site/MasterclassShell.tsx` | Masterclass header/footer shell with external register pill |
| `src/components/site/RefundPolicy.tsx` | Config-backed refund popover (hover + touch) |
| `src/components/ui/button.tsx`, `popover.tsx` | The only shadcn primitives left |
| `src/components/ui/orbit-delivery-hero.tsx` | Dormant react-three-fiber planet/courier hero (uses `public/orbit-assets/models/*.glb`) |
| `src/components/ui/sunset-skyline-hero.tsx` | Dormant `FutureCityScene` 2126 video band |
| `src/components/site/{Hero,Glance,TheChallenge,HowItWorks,IdeaGalaxy,AfterIdeas,SkillCard,LowerSections,SiteNav,SiteFooter,SiteLayout}.tsx` | Dormant config-driven homepage sections from before the HTML-reference redesign |
| `src/styles.css` | Tailwind v4 theme, base tokens, Challenge Platforms Lab override block |
| `src/styles/home-reference.css`, `src/styles/masterclass.css` | Scoped CSS for `.fcl-ref` homepage and `.mc` masterclass |
| `src/integrations/supabase/*` | Lovable-generated clients/middleware for the shared project (unused at runtime); `types.ts` = estate `public` schema |
| `public/robots.txt`, `public/favicon.*`, `public/mentors/*`, `public/orbit-assets/models/*.glb` | Static assets |
| `tests/server-runtime.test.ts` | Rolldown fixture proving the un-split server bundle boots |
| `.lovable/plan/*.md`, `.lovable/north-star/north-star.md`, `.lovable/project.json` | Lovable plans (masterclass evolution, redesign), north star, template id |
| `.env` | Supabase URL/key env names (values not reproduced); not read by app code today |

---

## Appendix A – `challenge` config key reference (`src/config/challenge.ts`)

Top-level keys of the `challenge` object, in source order, with what consumes them today
("HTML" = `home-main.html`/`home-closing.html` via `fill()`; "MC" = `/masterclass`; "dormant" =
only the unused React sections or tests read it).

| Key | Type / example | Consumer |
| --- | --- | --- |
| `homeHref` | `'/'` (`'/nz'` for NZ) | HTML logo link |
| `regionBadge` | `null` / `'NEW ZEALAND EDITION · NZDT'` | HTML hero eyebrow pill (crimson outline) |
| `name`, `shortName`, `code`, `editionYear` | `'Global Future Innovators Challenge'`, `'GFIC 2026'`, `2026` | HTML hero/banner/footer, MC CTA band, nav alt text |
| `seo.title`, `seo.description` | route `head()` | `/`, `/nz` |
| `artwork.studentDesign`, `artwork.futureNotebook` | local PNG imports | MC about figure; dormant |
| `sectionLabels` | section eyebrow strings | dormant |
| `tagline` | three sentences; last word bolded by `fill()` | HTML hero headline |
| `hero.*` | `datesSummary`, `eyebrow`, `subheading` (first "Pick any field" bolded), `partners[]`, `glanceLabels`, `futureYears [2035,2126]` | HTML hero + tests; glance labels dormant |
| `challengeSection` | three "what will you be doing" steps + `note` | dormant (tests guard `note`) |
| `about` | `label/title/lead/stepTitles[3]/body[3]` with `{grades} {team} {pool}` tokens | HTML `#about` via `aboutBody` |
| `problemAnnouncement` | orientation-day note | HTML `#tracks` |
| `organiser`, `byline` | `'the Group'`, `'Future Edge'` | root `og:site_name`, HTML "BY FUTURE EDGE" |
| `audience` | `{ minGrade 5, maxGrade 12, location }` | HTML grades, tests |
| `format` | `'Live group sessions'` | tests |
| `weekends[4]` | `label, dates, startIso, sessionLabel, year, items[]` | HTML `w1..w4`, `finaleUpper`, `startDate`; `timeline.ts` |
| `howItWorks` | `title, intro, sessionNote, join[], divisionTitle` | HTML `intro`; dormant |
| `timeline` | `labels[4], registrationStatus, earlyBirdOffer, pendingLabel` | HTML `statusUpper`; `timeline.ts` |
| `structure` | four dated cards + `sessionNote`, `joinText` | dormant |
| `sampleIdeas` | `imagesByField` (14 fields × 2035/2126 images), `jobs[6]` | dormant (`IdeaGalaxy`/ideas browser) |
| `registrationOpens / registrationCloses / submissionDeadline` | `{ iso: string|null, placeholder }` | tests; closes/submission are `null` + `'[DATE TBC]'` |
| `sessionTime` | `{ iso, label, timeZone, startHour.. }` | HTML `sessionUpper`, `timeShort`, FAQ `{sessionTime}` |
| `earlyBirdDeadline` | `{ iso '2026-11-30T23:59:59-05:00', dateLabel }` | HTML `earlyDate`, `fees.ts`, `registration.ts` |
| `currency`, `fees` | `USD`, `{ earlyBird 80, regular 100, basis 'per student' }` | HTML `earlyBird/regular/saving` |
| `refundPolicy` | 5 % processing-fee disclosure | `RefundPolicy` popover |
| `prizePool`, `prizeWinnerCount` | `500`, `3` | HTML `#story` |
| `divisions[2]` | Explorers 5–7 / Pioneers 8–12 | dormant + tests |
| `maxTeamSize` | `5` | HTML, FAQ token |
| `paymentUrl` | `'[PAYMENT LINK]'` placeholder | root outbound-skip host (guarded by try/catch) |
| `legal` | `#terms`, `#privacy` anchors (no real pages) | dormant |
| `applyUrl` | Portal apply URL | HTML `registerUrl` ×13, MC band, outbound tagging |
| `contactEmail` | team mailbox | HTML footer `mailto:` |
| `calendlyUrl` | Calendly with baked UTMs | HTML "book a call", outbound tagging |
| `waysIn`, `pathBack` | three ways in; 2126→2026 backcast steps | dormant |
| `feesSection`, `prizesFeesSection`, `prizesSection` | labels/notes incl. `{date} {saving} {days}` tokens | dormant |
| `mentors[11]` | `Mentor` objects (`name, university, field, secondary, photo, specialties[], bio`); photos from `public/mentors/*` or asset manifests | `FclMentors`, MC hosts (first 4) |
| `mentorsSection` | `title, speakerLine` (dean bio), placeholders | `FclDean` |
| `skillCard` | copy + sample image | dormant |
| `futureCity` | 2126 video scene copy, `video`, `poster` | dormant (`FutureCityScene`) |
| `faq[10]` | Q/A with `{maxTeamSize}` `{sessionTime}` tokens | `FclFaq` |
| `postcard` | legacy postcard copy (100 words, 5 MB) | dormant |
| `finalCta` | `'Your winter break could change the future.'` | dormant (HTML has its own closing band) |
| `speaker` | programme dean name/title/photo | `FclDean` |
| `mentorLabel` | `'university student mentors'` | tests |
| `ai` | `{ optional, competition, allowedTools[6] }` | tests |
| `nav[7]` | anchors `the-challenge, timeline, structure, ideas, fees, mentors, faq` | dormant `SiteNav` (HTML has its own nav) |

`nzChallenge` overrides: `homeHref, regionBadge, code, editionYear, seo, audience.location, hero.datesSummary/eyebrow/subheading, about.lead, weekends[0..3].dates/startIso, howItWorks.sessionNote, structure.cards[].date/sessionNote, sessionTime, earlyBirdDeadline, faq (session-time answer), finalCta`. Everything else (fees, mentors, applyUrl, FAQ) is inherited.

`masterclass` keys: `key, applyUrl, label, title, startsAt, dateLabel, timeLabel, durationLabel, host, isFree, intro, outcomes[4], audience[3], faq[4]`.

Helper exports: `partnerLogos` (`hpair`, `sales` → asset URLs), `isPlaceholder(v)` (`/^\[.*\]$/`), `type ChallengeConfig`, `type Mentor`.

---

## Appendix B – `src/lib/utm.ts` API

| Export | Purpose |
| --- | --- |
| `UTM_KEYS` | `['utm_source','utm_medium','utm_campaign','utm_term','utm_content']` |
| `UNKNOWN` | `'unknown'` – the value stored for any missing field |
| `utmSchema` | zod object: each key `string().trim().max(200).default('unknown')` – reused by the (legacy) registration schema |
| `emptyUtm()` | all five keys = `'unknown'` |
| `hasReal(u)` | true if any key is a non-`unknown` value |
| `parseUtm(search)` | case-insensitive key match, `&amp;` → `&`, first occurrence wins, returns `null` when no real tag |
| `mergeFirstTouch(stored, incoming)` | stored wins if it has a real value, else incoming |
| `withUtm(href, utm, base, override=false)` | adds real tags to a URL; keeps existing params unless `override`; returns path-only string for same-origin URLs |
| `shouldSkipPath(pathname)` | `/^\/(admin|auth|login|signin|signup|checkout|payment|pay)(\/|$)/i` |
| `getUtm()` | sessionStorage → current query → empty; always five fields (what forms used to save) |
| `captureUtm()` | merge + persist to `sessionStorage['fe_utm_v1']` |
| `initUtmCapture()` | once-only guard around `captureUtm()` |
| `installUtmCarry()` | patches `history.pushState/replaceState`, retags on load/`popstate`, rewrites same-origin anchors on capture-phase click/auxclick/contextmenu; returns a cleanup fn |
| `installOutboundUtmTagging(isTarget, skip)` | once-only; at click time rewrites cross-origin anchors whose host is an apply/Calendly host with `override=true` |

Root wiring (`__root.tsx`): targets = hosts of `challenge.applyUrl`, `nzChallenge.applyUrl`, both
`calendlyUrl`s, `masterclass.applyUrl`; skip = `paymentUrl` host or pathname matching
`(pay|payment|checkout|auth|login|admin)`.

---

## Appendix C – Homepage template walk-through (`src/content/home-main.html` → `home-closing.html`)

| Order | Section id / element | Content and tokens |
| --- | --- | --- |
| 1 | `<header class="ref-header">` + `#reference-navigation` | Ring-mark logo linking `{{homeHref}}`, `{{programNameUpper}}` / `{{bylineUpper}}`, links `#about #tracks #how #schools #mentors #faq`, `/masterclass`, REGISTER pill → `{{registerUrl}}`; `#mobile-menu-host` injected before nav |
| 2 | `<main id="top">` hero (`.ref-colour-hero`) | Left: SVG ring dial (`.ring` rotates, `.ref-orbit-label` spans, `.jobUpA/.jobUpB` career pill); right: `{{regionEyebrowHtml}}`, `{{taglineHtml}}` headline, `{{subheading}}`, `{{hpairLogo}}`, start card `{{startDate}}`/`{{startTime}}`, price `{{earlyBird}}` vs struck `{{regular}}` (refund popover host appended after `.ref-hero-start > div:last-child`), `{{calendlyUrl}}` link |
| 3 | `#banner` | Black band: `{{programNameUpper}} {{editionYear}}`, grades `{{minGrade}}–{{maxGrade}}`, `{{statusUpper}}`, register pill |
| 4 | `#programme-dean-host` (injected) | `<FclDean>` portal – dean photo, name, title, `mentorsSection.speakerLine` |
| 5 | `#about` | `{{aboutTitle}}`, `{{aboutLead}}`, three numbered steps from `{{aboutBody}}` |
| 6 | `#tracks` | "Six problems set in 2035. Pick one to solve." – six illustrated cards (static copy duplicated from `problems.ts`), `{{problemAnnounce}}` |
| 7 | `#discover` | "Pick a problem. Imagine its future." – process strip, `{{intro}}`, `{{maxTeamSize}}` |
| 8 | `#story` | Prize band: `${{prizePool}}` pool shared among top `{{winners}}`, certificate line |
| 9 | `#how` | "Orientation. Submission. Semifinals. Finals." key-date cards `{{w1}}..{{w4}}`, `{{timeShort}}` ×4, `{{sessionUpper}}`, `{{finaleUpper}}` |
| 10 | `#schools` | Fees: "Early bird" `{{earlyBird}}` until `{{earlyDate}}`, regular `{{regular}}`, save `{{saving}}`; refund popover host after `#schools h2`; register pill |
| 11 | React: `<FclMentors>` `#mentors`, `<FclFaq>` `#faq` | carousel + accordion from config |
| 12 | `home-closing.html` `#register` + footer | Final CTA band `{{programName}}`, `{{earlyDateUpper}}`, `{{registerUrl}}` ×2, `{{calendlyUrl}}`, footer columns (`#about … #faq`, `/masterclass`, `#top`), `mailto:{{email}}`, `{{bylineUpper}}` |

---

## Appendix D – Test inventory

| File | Runner | Cases |
| --- | --- | --- |
| `src/config/challenge.test.ts` | node:test | 28 – grades/worldwide; live groups; AI optional; six allowed tools; fees 80/100; fee copy names deadline + savings; prize 500/3; team ≤ 5; divisions; no poster/comic/video formats; no coding / no removed claims; registrations open 13 Oct 2026; early bird through 30 Nov ET; Saturdays + Sunday finale 9 a.m. ET; dates 5/12/19 Dec + 3 Jan; mentor roster; four weekends; 2035 semis / top 10 → 2126 finals; skill card copy; glance "Online"; only 2035/2126 in copy; idea horizons; about section plain words; problems announced at orientation; key-date cards show time and no register button; "by Future Edge" in nav + footer; hero + fees show struck regular fee; GFIC 2026 branding + tagline; NZ cohort schedule (11:30 NZDT = 9:30 AEDT) |
| `src/lib/utm.test.ts` | vitest | 8 – parse all five case-insensitively + `&amp;`; fill missing with unknown; untagged → null; first real capture wins; internal link tagging; outbound override; preserve destination params/fragments; history/back navigation carry |
| `src/lib/sitemap.test.ts` | vitest | 3 – include/exclude by `staticData.sitemap`; `exclude-subtree`; XML without `<lastmod>` |
| `src/lib/masterclass.test.ts` | vitest | 3 – 29 Nov 2026 09:00 ET; external Portal form URL; five UTMs carried |
| `src/config/refund-policy.test.ts` | vitest | 1 – 5 % deduction wording |
| `src/lib/fees.test.ts` | node:test | 2 – early-bird presentation vs after deadline |
| `src/lib/registration.test.ts` | node:test | 7 – division mapping; `feeFor` before/after/no deadline; configured cut-over at 1 Dec ET |
| `src/lib/timeline.test.ts` | node:test | 5 – four milestones; next-milestone selection through the finale |
| `src/components/ui/sunset-skyline-hero.test.ts` | node:test | 1 – missing `futureCity` config renders null, no crash |
| `tests/server-runtime.test.ts` | vitest | 1 – rolldown single-chunk ESM + CJS helper boots |

Run with `bunx vitest run` (or `npx vitest run`); there is no `package.json` script and no
`vitest.config.*` (defaults: `**/*.test.ts`, `@/` alias resolved through `vite-tsconfig-paths` in the
Lovable Vite config).

---

## Appendix E – Env vars and secrets by name

| Name | Where read | Status |
| --- | --- | --- |
| `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `VITE_SUPABASE_PROJECT_ID` | `.env` only | injected by Lovable Vite config; not read by code |
| `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_PROJECT_ID` | `.env`; `client.server.ts`, `auth-middleware.ts` | dormant |
| `SUPABASE_SERVICE_ROLE_KEY` | `client.server.ts` | dormant (never in `.env`; Lovable Cloud secret) |
| `LOVABLE_CRON_SECRET`, `LOVABLE_CRON_SECRET_PREVIOUS` | `cron-auth.ts` | dormant |
| `LOVABLE_DB_MIGRATION_URL` | `drizzle.config.ts` | legacy Lovable Cloud DB |
| `EXTERNAL_SUPABASE_URL`, `EXTERNAL_SUPABASE_SERVICE_KEY` | plan doc only | never implemented |
