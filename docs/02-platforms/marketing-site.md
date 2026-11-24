# Marketing Site

The group's own top-level marketing and discovery site — distinct from any one programme
brand's own site, and distinct from the central Application Portal.

## Identity

- A static HTML export (`index.html`, `about.html`, topic hub pages, a camps/challenges/
  conferences section, a blog-media gallery, a build script that generates a testimonials
  gallery) rather than a Vite/React SPA — chosen for aggressive caching and server-rendered
  content, which search ranking rewards and a transactional checkout app does not need.
- Scale: a recent, much smaller repo (25 commits) relative to its sibling platforms — a
  deliberate rebuild, not an organic multi-year history.

## Division of labour with the Application Portal

Two separate projects deliberately share one schema's worth of discovery data, with a strict
write boundary: this site owns only its own discovery tables (content pages, the
finder/discover tool, calendar views, subject hubs, testimonials) and never touches forms or
checkout; the Application Portal owns applications, payments, admin and form templates and
never owns content pages. The reasoning recorded for keeping them permanently separate:
content wants aggressive caching for search ranking, forms and payments want a stable
transactional app, and merging the two would recreate the very fragmentation the
consolidation was meant to remove. The safety mechanism is write ownership, explicitly stated
in each project's own memory, because two independent AI agents sharing one schema is exactly
the collision pattern that has caused real incidents elsewhere in the estate.

## Lead capture

Public forms (newsletter, camp interest, school programme inquiries, masterclass discovery)
write into the discovery schema's lead tables, which the Application Portal and sibling
platforms read for lead nurture and email automation — read-only from this site's
perspective; it does not run its own checkout or send its own transactional confirmations.

## Sequencing rule for a broader domain consolidation

When multiple separate programme domains are being folded into one marketing site with
redirects, the hard rule that applies: **never redirect a satellite domain until its
destination page reads live data from the shared catalogue.** Pilot the smallest domain first,
wait a observation period, then proceed — never flip every redirect in one pass.
