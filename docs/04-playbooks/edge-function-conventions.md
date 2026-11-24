# Edge-function conventions

Distilled from reading `_shared/` modules and function headers across the estate's ~480
edge-function directories. This is the skeleton to start from; copy it, don't start from a
blank `Deno.serve`.

## The `_shared/` modules every platform re-invents in roughly the same shape

| Module | Purpose |
|---|---|
| `db.ts` | client factories: `centralAdmin()`/`centralShared(schema)` (service role, pinned schema) + `localAuthAdmin()`/`localUserClient(authHeader)` for the own-project-auth split |
| `cors.ts` (or inline `corsHeaders`) | a fixed CORS header object reused by every function in the repo |
| `checkout.ts` / `pricing.ts` | catalogue price resolution, early-bird vs sticker, minor-unit conversion |
| `promo.ts` / `centralPromo.ts` | central promo-code validation and redemption recording |
| `paymentLinks.ts` | reads to the shared payment-links table via its one public RPC |
| `attribution*.ts` / `trackConversion.ts` | UTM capture, payment-click vs acquisition-touch separation, best-effort conversion reporting |
| `rateLimit.ts` | in-memory + durable (DB-backed) rate limiting, keyed by IP and/or token |
| `auth-guard.ts` / `cronAuth.ts` | the four auth-model helpers described below |
| `safe-equal.ts` | constant-time string comparison for secret/token checks |
| `ssrfGuard.ts` | validates an outbound URL before the function fetches it, when a function accepts a user-supplied URL |
| `*.test.ts` beside pure modules | unit tests for the parts of a function that don't need I/O |

## The canonical handler skeleton

```ts
import { createClient } from "https://esm.sh/@supabase/supabase-js@2.58.0"; // pin an exact version

const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers": "authorization, x-client-info, apikey, content-type, x-service-key",
  "Access-Control-Allow-Methods": "POST, OPTIONS",
};
const json = (body: unknown, status = 200) =>
  new Response(JSON.stringify(body), { status, headers: { ...corsHeaders, "Content-Type": "application/json" } });

Deno.serve(async (req) => {
  if (req.method === "OPTIONS") return new Response(null, { headers: corsHeaders });
  if (req.method !== "POST") return json({ error: "method_not_allowed" }, 405);

  // 1. Pick exactly one auth model (never mix two on the same function):
  //    a) bearer + role lookup        -> requireAdmin(client, token)
  //    b) anonymous + unguessable id  -> validate a submission_token/UUID, rate-limit
  //    c) webhook signature            -> verify against the provider's SDK before touching the body
  //    d) cron secret                  -> compare x-service-key / x-cron-secret with safeEqual()

  // 2. Parse and validate input (zod or hand-rolled). Reject unknown fields on anything
  //    that writes state (zod's .strict()), so a client can't smuggle an extra column.

  // 3. Do the one thing this function does. Keep business rules in a pure, unit-tested
  //    module (`_shared/<thing>-rules.ts`) and call it from here — don't inline logic that
  //    should be testable without network I/O.

  // 4. Return a typed, minimal response. Never echo a password, token or raw secret back.
});
```

## The four auth models (pick exactly one per function)

| Model | When | How |
|---|---|---|
| Bearer + role lookup | admin/privileged actions | `auth.getUser(token)` on the auth project, then a table lookup (never a JWT custom claim) against the caller's role; 403 on mismatch |
| Anonymous + unguessable token + rate limit | public checkout/intake | no bearer; the UUID token itself is the credential; rate-limit by IP and by token separately, both in-memory and durable |
| Webhook signature | Stripe/Razorpay/Calendly/Zoom/WhatsApp callbacks | verify with the provider's own signature check before parsing the body as trusted; drop a duplicate delivery by provider event id |
| Cron secret | scheduled jobs | compare a header value with `safeEqual()`, never `===`, against a value read from an env secret — never hard-coded in the cron job definition or the function source |

Deploy every function with `verify_jwt = false` and do the auth check in code — this is the
estate-wide convention, confirmed across every repo's `supabase/config.toml`. It is correct
for webhooks and anonymous intake; it means an admin-only function's safety depends entirely
on its own in-code check, so that check must exist and must be tested.

## Naming and organisation

- Prefix functions by platform (`portal-`, `premed-`, `affiliate-`, `ops-`) when several
  platforms deploy to the same Supabase project, so names never collide and a log line always
  identifies its owner at a glance.
- A `lwl_<name>`-style namespace migration (see `docs/02-platforms/learner-portal.md`) is the
  safe way to rename a whole function surface on a live, shared project: deploy new names
  first, rewire only existing schedules, publish the client, and retain the old names as a
  compatibility layer until verified — never delete the old ones in the same change that adds
  the new ones.
- A `deploy-all.sh` script that reads each function's `verify_jwt` setting straight out of
  `config.toml` (rather than re-deciding it by hand per function) is the safe way to redeploy
  an entire function surface to a new project during a migration.

## Idempotency and side effects

- Webhooks: drop a duplicate delivery by the provider's own event id before doing anything —
  an in-isolate `Set` is sufficient for the retry window most providers use.
- Any write that claims a limited resource (a usage cap, a one-time reward) needs its own
  atomic SQL function (`claim_x_use(id)` returning a boolean), called exactly once from the
  call site that has already confirmed this is the first successful attempt — the SQL function
  is atomic against concurrent callers but not idempotent against being called twice by the
  same caller, so the "only call this once" guarantee is the caller's job, not the function's.
- Anything that sends a message (email, WhatsApp, a reward grant) should be keyed to an
  immutable row id for its delivery record, not to a mutable parent state, so a retry of the
  outer webhook cannot duplicate the send.
- An AI-facing function should check a kill switch first and, when off, skip without touching
  anything — not log a partial write — so it is naturally picked up again once re-enabled.
