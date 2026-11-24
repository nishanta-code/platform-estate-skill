# Frontend auth patterns

How the browser actually holds and defends a session, across both stack shapes and both
auth topologies. See `docs/01-backend/auth-and-identity.md` for the server-side registration
contract this all sits on top of.

## Session storage and lifecycle

- A hand-written Supabase client (never the auto-generated one, when the two differ) picks its
  `storageKey` and `persistSession` deliberately: `localStorage` for "keep me signed in,"
  `sessionStorage` for a tab-scoped session that clears on tab close. This choice alone
  implements the "remembered vs. this-device-only" UX — no separate server flag is needed.
- Every `signOut()` call should default to **local scope** (`{ scope: "local" }`); global scope
  revokes refresh tokens on every device the user is signed into, and should be reserved for a
  single, explicit "Sign out everywhere" admin action — never the default behaviour of an
  ordinary logout button.
- Revalidate the session on `visibilitychange`, `focus`, `online`, and cross-tab `storage`
  events, plus a periodic interval (a 5-minute interval is the pattern used where this is
  built out most fully) — a tab left open for hours should not silently hold a dead session.
- **Never sign a user out on a transient failure** (a role lookup that failed due to a network
  blip) — deny the specific route instead and retry the lookup once after a short delay (the
  reference implementation retries `getSession()` once after 800ms) to survive a token-rotation
  race rather than bouncing a legitimately signed-in user.
- **Never sign out on `pagehide`/`beforeunload`** — an ordinary page refresh fires both events,
  and wiring a sign-out to them logs out real users on every reload.

## Route guards

- Vite+React shape: a `RequireRole`/`ProtectedRoute` component wrapping the route tree, reading
  role from a dedicated roles table (never from a JWT custom claim, since roles are
  schema-scoped per platform and a JWT claim can't be revoked mid-session).
- TanStack Start shape: a global middleware (`requireSupabaseAuth`/`attachAuth` style, wired in
  `start.ts`) applied to a route group (`_authenticated/`), with a `createServerFn()` reading
  the session server-side rather than trusting a client-sent role.

## The two auth topologies, from the browser's side

- **Direct-estate platforms**: one Supabase client, one set of credentials, shared across every
  platform on that project — a password set on one platform's login page works on every other
  direct-estate platform's login page too, because it is the same `auth.users` row.
- **Own-project-auth platforms**: the browser's Supabase client talks only to that platform's
  own small project; it has no visibility into the estate at all. All estate reads/writes for
  that platform happen server-side (an edge function or a TanStack server function holding a
  service-role key), never from the browser. A password on one of these platforms does
  **not** work on a sibling own-project platform or on the direct-estate platforms — treat each
  as its own identity space.
- **The one place this has gone wrong in practice**: a repo with two Supabase client files
  (one auto-generated, pointing at a dead project; one hand-written, pointing at the live
  estate schema) where only the hand-written one is actually imported by real pages. Before
  answering "which project does this repo use," grep for which client file is actually
  imported — never trust `.env`/`config.toml` alone.

## Realtime subscriptions in a frontend auth context

A fixed Realtime channel topic name (e.g. keyed only by user id) will collide the moment a
component mounts twice back-to-back, which React's development mode does on purpose to catch
missing cleanup. The second mount can be handed the first mount's already-subscribed channel
object before its async cleanup completes, and calling `.on()` on an already-subscribed channel
throws — this has crashed a whole route via the root error boundary in at least one real
build. **Suffix every Realtime channel topic with a random value generated fresh inside the
effect**, so two mounts never share one channel object regardless of cleanup timing.

## Registration forms (the browser side of the server contract)

Every public registration form in this estate that follows the hardened pattern shares the
same client-side shape: a hidden honeypot field a real person never sees or fills, a captured
"form shown at" timestamp enforcing a minimum fill time before submission is even attempted,
and a neutral, identical error message for every rejection reason (duplicate email, rate
limit, bot signal) so the form never confirms or denies whether an email already has an
account.
