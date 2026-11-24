# Working alongside an AI app builder

Most of the estate's repos are built and continuously maintained by an AI app-building
platform (the kind that owns a project's sandbox, runs its own agent against the same
repository, and can push commits to the connected branch independent of anything a coding
assistant does locally). The rules below are what makes editing one of these repos locally,
or with a different AI agent, safe.

## What the builder manages vs. what is yours to edit

Each repo's own rules file marks a block (commonly delimited, e.g. `BEGIN`/`END` markers) that
names what the builder manages directly and should not be hand-edited:
- **Database migrations / schema** — ask the builder's own agent to change the schema rather
  than hand-writing a migration, *unless* the session has been explicitly given direct database
  access for a specific task (a personal access token, a CLI login) — in which case that
  override applies only for that session, not as a standing practice.
- **Generated files** — any file whose header states it is automatically generated.
- **The AI gateway** — keep model calls on the builder's own gateway; keep its API key out of
  browser-reachable code.
- **Connectors and secrets** — the builder provisions and stores these; a local session can
  read which connectors are enabled but generally cannot set secret values directly.

Everything outside that block is the platform's own code and is fair game, governed by the
repo's "Project rules" section instead.

## The one thing that changes everything: it can push while you work

The builder's own agent, running independently of any local coding session, can commit
directly to the connected branch and apply its own database migrations at any time — including
mid-session, with zero warning. This has happened for real: a builder's own security scan found
and fixed several issues and pushed multiple commits to the connected branch while a local
session was still working on the same repo, none of which the local session had in its working
tree.

**Rules that follow directly from this:**
1. `git fetch` immediately before every merge and push — not just once early in the session.
   A single mid-session fetch has revealed dozens of new upstream commits landing in the
   minutes since the previous fetch on a heavily-used repo.
2. `git checkout <branch>` does **not** update a local branch ref from the remote — only an
   explicit `fetch` + reset/merge does. Checking out a stale local branch and merging a feature
   branch into it can fast-forward past what the remote has actually gained, and the resulting
   push rejection can look exactly like the builder's agent "reverted" local work, when in
   fact the local session never had the remote's real state to begin with.
3. Read every diff from the other side before proceeding — never trust a one-line status
   summary (not even the builder's own chat panel) at face value. Independently re-verify any
   claim of "fixed" against the live system (a direct policy/grant/privilege query), not the
   summary text.
4. Re-check the migration-tracking state (e.g. `supabase migration list --remote`) after
   pulling in the other side's changes — if its migrations aren't recognised as already
   applied, the next migration push from this session could try to re-run them and fail.
5. A remote named something other than `origin` is not unusual on these repos — check
   `git remote -v` rather than assuming; a wrong remote name fails with a generic "access
   rights" error that reads like a permissions problem rather than a naming one.

## Keeping project knowledge in sync

Most of these builders let an operator attach a persistent knowledge note to a project — read
by the builder's own agent on every future session. When a local session hardens something
the builder's agent might otherwise regenerate away (a registration endpoint, a shared-schema
convention), write that convention into the project's knowledge note, not only into the repo's
own rules file — otherwise the builder's agent can unknowingly revert the hardening the next
time it touches that area, with no local session present to catch it.

Two practical notes: this knowledge store typically **replaces** its content on every write
rather than merging, so always read the existing content first and measure its size before
appending (there is usually a hard size limit, and exceeding it silently truncates or rejects
the write); and a workspace-level knowledge note may not be writable by a collaborator account
— if so, write it per-project instead of trying once at the workspace level.

## Mapping a local folder to the builder's own project record

Project display names inside the builder do not always match the local repo folder name —
one platform's local folder name bears no resemblance at all to its public-facing brand name
inside the builder's dashboard. The reliable way to confirm which builder project a local
folder corresponds to is an exact match on the **latest commit SHA**, not a name search.

## Build-pipeline gotchas that only show up on the builder's side

- A project with its own `bun.lock(b)` that builds cleanly with `bun run build` locally can
  still fail on the builder's own pipeline if that pipeline's default install step uses npm —
  check `npm install --dry-run` whenever a user reports "build errors" that a local bun build
  can't reproduce; an `overrides`/`dependencies` version mismatch that bun tolerates silently
  is a hard npm failure (`EOVERRIDE`).
- A Windows checkout with CRLF line endings repo-wide will fail a bulk lint run on nearly every
  file, including ones nobody touched, purely on line-ending rules — this is pre-existing and
  out of scope to mass-fix; filter lint checks to the files actually changed in the current
  session.
- A stale dependency cache after a fresh install can throw a spurious "invalid hook" style
  error the first time an affected component renders — not a real bug; clearing the relevant
  build-tool cache and restarting the dev server resolves it.

## Working across many of these repos at once

For a change that spans several sibling platforms (a shared bug fix, a cross-cutting audit),
run one independently-scoped agent per repo, each told explicitly to touch only its own
directory and never run git itself, and serialise every git operation (fetch, diff review,
commit, push) in the coordinating session so two agents never race on the same remote. A
"look but don't touch" recon task needs to say "do not edit" as forcefully as a real
instruction, not as an aside — a recon-only task has, in practice, gone ahead and made a live
edit anyway when the instruction was phrased softly.
