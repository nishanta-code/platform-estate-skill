# App shells and the responsive house rules

Two shells get copied between platforms more than any design file does; the responsive rules
below were earned the hard way, across two separate audits of the same codebase.

## The Learner Portal shell (the one every new internal tool should start from)

`src/components/layout/AppShell.tsx` + a `RoleLayout`/`Sidebar`/`Topbar` family:

```tsx
className={`absolute left-0 top-0 flex h-full w-[248px] max-w-[82vw] transform flex-col
  bg-card shadow-xl transition-transform duration-300 ease-[cubic-bezier(0.32,0.72,0,1)] ...`}
```

- A **persistent sidebar** on desktop; on mobile it becomes a **drawer capped at `82vw`** of
  the viewport regardless of its own fixed width, so it never fully covers the screen on a
  narrow phone.
- Nav items are **grouped by section**, not a flat list — grouping survives role growth (new
  admin pages slot into an existing group instead of lengthening a flat list).
- Role switching (student / mentor / admin) changes which group set renders, not a different
  shell component.
- A sidebar footer card separates the "go to profile" link from the "sign out" action as two
  distinct tap targets, so signing out never requires navigating into profile first.
- A sibling platform copied this shell wholesale (same component names, same widths) when
  building its own internal dashboard — the correct way to reuse a shell: copy the pattern, not
  the file, so each platform's shell stays independently maintainable while looking identical.

## The Application Portal admin shell

`AdminLayout.tsx` + `components/admin-portal/ui/PortalSidebar.tsx` / `PortalTopbar.tsx` /
`PortalPage.tsx`:
- Dynamic viewport height (`100dvh`-equivalent handling) so mobile browser chrome doesn't clip
  the bottom of the layout.
- A viewport-clamped mobile drawer width, the same `max-w-[Nvw]` idea as the Learner Portal
  shell.
- **44px touch targets** on every interactive control in the shell by default.
- `safe-area-inset` padding so the layout respects a phone's notch/home-indicator area.
- The base dialog primitive used everywhere in this app bakes these defaults in once, for
  every dialog, rather than each dialog re-implementing them:
```tsx
"fixed left-[50%] top-[50%] z-50 grid w-[calc(100vw-1.5rem)] max-w-lg max-h-[85vh]
 overflow-y-auto translate-x-[-50%] translate-y-[-50%] ... sm:rounded-lg sm:p-6"
```
  plus a 44px mobile close button. Put shared responsive behaviour in the primitive component,
  not in each call site — it is the single highest-leverage place to fix a whole class of bug
  at once.

## The house responsive patterns (verified by reading the actual files, twice)

| Pattern | Rule |
|---|---|
| Data tables | `hidden md:block` table + a `md:hidden` card-list rendering of the same rows — used consistently across every data-table page in the biggest platform's admin section |
| A toolbar mixing a fixed-width button with a flexible search input | the search wrapper needs **both** `flex-1` **and** `min-w-0` — a browser text input's intrinsic min-width ignores `flex-1` alone and refuses to shrink, pushing its sibling button off-screen. This exact bug recurred four times across one platform after the first fix, always in an unaudited corner |
| Any toolbar/header row with more than one child on a narrow screen | needs an explicit `flex-wrap` or `flex-col` mobile fallback, not just `flex` |
| A table wrapper meant to scroll horizontally | must be `overflow-x-auto`, never `overflow-hidden` — the latter silently clips content instead of letting it scroll |
| A 2-up info grid inside a dialog | needs `min-w-0` plus `break-words`/`break-all` on any cell that can hold a long email or name, or it overflows the dialog horizontally |
| Checkout/payment pages | a sticky mobile-only CTA bar with safe-area insets, 16px input font (prevents iOS auto-zoom on focus), and scroll-into-view when a payment panel unlocks |
| A drag-and-drop builder (the Application Portal's Form Builder) | below `lg`, swap side-by-side panels for `Sheet` drawers and swap drag-and-drop for tap-to-move buttons — don't ship drag-and-drop as the only mobile interaction |
| Touch targets (signature pads, mobile close buttons) | 44px minimum, even when the desktop equivalent is visually smaller |

## The lesson behind the table above

A full audit of one platform's admin area (51 files, every page, every dialog) found the app
was already excellently built — six real gaps, all of this exact shape, all layout-only. A
second sweep triggered by one user-reported screenshot found **four more instances of the same
bug class** in files the first audit's file list never named, because that list was built from
the route table rather than the actual `src/pages` directory and its real imports. **When a
live bug is reported, re-sweep for the whole bug class across every file that actually exists,
not just patch the one instance and call the class closed.**
