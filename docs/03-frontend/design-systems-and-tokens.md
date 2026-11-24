# Design systems and tokens

Every platform in the estate has its own brand, but a handful of design decisions get copied
deliberately between sibling platforms rather than invented fresh each time. This document
records the pattern, not a specific brand's hex codes (those belong to each platform, not to a
reusable skill).

## The group-identity brand layer

The parent marketing site defines a small, fixed brand-token layer: a near-white paper tone, a
soft neutral wash for section backgrounds, a slightly deeper wash for nested panels, a hairline
border colour, three ink tones (full-strength, soft, faint) for text hierarchy, a primary
accent colour with a deeper hover/active variant, a secondary accent (often gold/amber) with
its own deep and soft/tint variants, and a matching hairline border tint for the secondary
accent. Typography: one humanist sans for body and headings, one monospace for small metadata
(timestamps, codes, counters) — never a second display serif unless a specific platform's
brand calls for it.

**This exact token set, copied verbatim (not reinvented), is what a new internal tool or
partner-facing platform should start from** when the brief says "look like the rest of the
estate" rather than "build a new brand." One partner platform's build explicitly copied this
layer's values line-for-line from the marketing site's stylesheet, verified against the
source file rather than trusted from memory, while keeping its own distinct 3D visual
language layered on top.

## Per-platform brand identities (the general shape)

Each consumer-facing programme site has earned its own full identity: a primary colour, a
secondary/accent colour, a display typeface distinct from its body typeface, and — in the more
recent builds — a willingness to depart from flat design entirely in favour of a specific
"3D/futuristic" visual language (WebGL scenes, glass/iridescent materials, generated video).
When asked to build "the exact same design" as an unrelated external reference site, the
correct response demonstrated across this estate is a two-part one: decline the literal
clone/scrape, and offer an **original** design in the same general spirit and quality using the
platform's own content and brand colours — then let the person confirm scope before touching
already-shipped work.

A client CRM build intentionally used a notably more restrained scale than its first draft: a
first pass with saturated solid-colour blocks, oversized headings and very large corner radii
read as "playful" rather than "premium" for a professional-services brand; the correction was
desaturating the accent colour, shrinking the base radius, capping heading sizes, and tightening
padding throughout — the general lesson being that **brand intensity and professional
restraint are usually in tension, and a counselling/professional-services brand should default
toward restraint** unless the brief explicitly asks for a bold, energetic register.

## Interaction and motion libraries actually in use across the estate

| Library | Where | What for |
|---|---|---|
| `framer-motion` | most marketing sites | page transitions, scroll-triggered reveals |
| `gsap` + `lenis` | the research platform site, the partner portal homepage | scroll-pinned, shader-synced scenes; smooth-scroll easing |
| `three` / `@react-three/fiber` / `@react-three/drei` | the partner portal homepage | WebGL hero scenes, lit 3D cards, environment-map materials |
| `tw-animate-css` | the counselling CRM | entrance/hover motion without pulling in a full animation library |
| `remotion` | the founders platform | programmatic video rendering (chapter recaps, identity documents) |
| `recharts` | admin dashboards across several platforms | charts in the admin surfaces |

## Rules that apply regardless of brand

- **Any animated/3D section must degrade for `prefers-reduced-motion`.** The reduced-motion
  path is an accessibility fallback for the subset of visitors with that OS setting on — it is
  not what ships to everyone by default, and must never be described to a stakeholder as "what
  you'll see," since the stakeholder checking a preview link sees the full animated build.
- **Verify an animated build headlessly before shipping it**: desktop and mobile, motion and
  reduced-motion, zero console errors, zero horizontal overflow, and — for a WebGL scene —
  confirm the context actually initialised rather than silently failing to a blank canvas.
- A design system's token values, once copied from a source file, should be **re-verified
  against that source file**, not trusted from an earlier summary — colour and shell claims
  about a sibling platform have been wrong before and were only caught by reading the actual
  stylesheet/component directly.
