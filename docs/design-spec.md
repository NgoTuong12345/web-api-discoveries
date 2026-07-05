# `/api-discoveries` Skill Design

**Date:** 2026-07-05
**Status:** Approved (brainstormed with user)

## What this is

A new **global** skill at `~/.claude/skills/api-discoveries/` that generalizes the
project-local `discover-store-api` skill (VN retail store locators → store_tracker) into a
domain-neutral workflow: given any greenfield website, discover the data API behind it,
verify it, and deliver a replay-verified contract. The skill is **self-improving**: its
final phase harvests new platform fingerprints and traps back into its own learning files.

`discover-store-api` is **not modified**. It stays authoritative for VN retail store
locators inside DKKD.gov. Knowledge flows one-way: domain-neutral lessons from any session
(including DKKD ones) may be harvested upward into the global skill's learning files.

## Decisions made (with user)

| Decision | Choice |
|---|---|
| Location | Global `~/.claude/skills/api-discoveries/` (not git-tracked, not project-local) |
| Deliverable | Verified `api-contract.md` + handoff hook (wire into project ingestion engine if one exists) |
| Self-update architecture | Stable-core SKILL.md + separate append-only learning files |
| Auth scope | Public **and** session/cookie-carried APIs. Login-gated = out of scope (document finding, stop) |
| Seeding | Learning files pre-seeded with generalized (domain-neutral) entries from discover-store-api; case-files start empty |
| Browser tooling | browser-harness is the mandatory capture engine, integrated per-phase |

## File structure

```
~/.claude/skills/api-discoveries/
├─ SKILL.md          # stable core: 6-phase workflow, doctrine, harvest protocol
├─ fingerprints.md   # GROWS: table of observable tell → platform family → action
├─ traps.md          # GROWS: entries of signal / reality / evidence / action
└─ case-files/       # GROWS: one short file per site solved (starts with README.md template only)
```

SKILL.md instructs: read `fingerprints.md` + `traps.md` before capture; `case-files/` is
grep-on-demand (not auto-loaded) so invocation context stays cheap as the corpus grows.

Registration: a trigger block for `/api-discoveries` is appended to `~/.claude/CLAUDE.md`,
matching the existing per-skill registration pattern there.

## The 6-phase workflow (SKILL.md core)

1. **Recon** — find where the target data surfaces in the UI. Nav/footer `<a>` dump via
   `js()`, sitemap.xml, robots.txt. Decoy-page doctrine: the obvious page may be a product
   filter/marketing page; plan to interact (selectors, dropdowns, map pins).
   Exit: candidate page + the interaction that loads the data identified.
2. **Capture** — browser-harness network capture primary (per its `network-requests`
   interaction skill). Fallback ladder when zero XHR fires: JS-bundle mining →
   `__NEXT_DATA__`/SSR-embed parse → platform discovery roots (`/wp-json/` route enum,
   GraphQL introspection probe). Consult `fingerprints.md` before brute-forcing.
   Exit: raw request + response captured (not inferred).
3. **Classify** — domain-neutral access archetypes: `single-call` (whole dataset in one
   request), `partitioned` (loop a closed enumeration: region/category/page), `graphql`,
   `ssr-html`, `proximity-limited` (radius-only feed → anchor sweep), `platform-rest`
   (WordPress-style CPT). Exit: archetype chosen.
4. **Verify** — two-tier replay gate. **Tier 1:** clean-shell replay (bare requests, no
   carried state) → *public* contract. **Tier 2:** header/cookie bisection — drop captured
   headers one at a time to find the minimal replay set → *session-carried* contract with
   load-bearing headers named. Count self-consistency assert. "Existing docs are
   hypotheses" doctrine. WAF-vs-absent disambiguation requires BOTH clean-shell `http_get`
   and in-browser `fetch()`. Exit: replay returns 200 with same shape + tier assigned.
5. **Deliver** — write `api-contract.md` from a fixed template (endpoint, method, minimal
   headers, request/response shape, pagination/partition scheme, record key, counts, traps,
   tier). Default location `docs/api-contracts/<domain>.md` in the current project. Then
   the **handoff hook**: if the current project has a config-driven ingestion engine
   (e.g. store_tracker), offer to wire the contract into its schema instead.
   Exit: contract written; wired if an engine exists.
6. **Harvest** — self-improvement protocol (below). Runs on success AND failure (dead ends
   are traps worth recording). Exit: learning files updated or explicit "nothing novel".

## browser-harness integration (per-phase)

- Phase 1: `new_tab()` first nav, screenshots drive exploration, `js()` for DOM dumps.
- Phase 2: `network-requests` interaction skill is the canonical capture technique (point,
  don't duplicate); `click_at_xy` to trigger loads through iframes/shadow DOM; `http_get()`
  for bundle mining and platform-root probes.
- Phase 4: the two verify tiers map to the two transports — in-browser `fetch()` via `js()`
  (session baseline) vs clean-shell `http_get()`/bare requests (tier-1 gate).
- Auth wall → stop and ask the user (never type credentials). This enforces the scope
  boundary. CAPTCHA → user solves manually, script continues.
- Boundary: `api-discoveries` records *what to look for* (platform shapes); browser-harness
  `interaction-skills/` records *how to drive the browser*. No duplication between them.

## Harvest protocol (the "harness itself" mechanism)

- **When:** end of every session that invoked the skill, success or failure.
- **What qualifies:** durable + generalizable beyond the one site + novel.
- **Novelty check:** grep `fingerprints.md`/`traps.md` first. If an existing entry is
  contradicted or refined, EDIT that entry; never append a rival duplicate.
- **Entry discipline:** map-not-diary; ≤ 8 lines per entry; evidence anchor (site + date)
  mandatory. Traps may carry a *capture-technique* note when the trap is about how to
  observe (e.g. "zero XHR + only analytics pixels in drain_events → platform-root probe").
- **Case file:** one short file per site session in `case-files/` (template in its README).
- **Autonomy split:** appends/edits to learning files are autonomous (reported in session
  summary). Edits to SKILL.md itself (new phase/archetype/doctrine) require explicit user
  confirmation.
- **Anti-bloat:** learning file > 150 lines → harvest step *proposes* (never auto-executes)
  a compaction pass merging near-duplicates into generalized entries.

## Seed content (day-one knowledge, generalized — VN specifics stripped)

**fingerprints.md:** WordPress (`wp-includes` scripts, `/wp-json/` enum, `X-WP-Total`
header-not-body count, `per_page` cap 100, multilingual CPT mixing); Next.js
(`__NEXT_DATA__`, `/_next/static/chunks` mining, build-ID rotation); GraphQL single-POST;
legacy ASP.NET MVC SSR fragments (`data-id` attrs, banner-only counts); central JSON
gateway (envelope `{code,data,...}`, service-segment paths); multi-tenant commerce platform
(tenant ID in request, one endpoint spans sister brands); proximity-limited feed (page 2
empty, two anchors → different totals).

**traps.md:** banner/marketing counts lie; multilingual CMS double-counts (`?lang=` can be
a silent no-op — find the field that actually varies); radius feeds undercount dense areas;
existing docs are fabrication-prone hypotheses; WAF-403/404 vs truly-absent endpoint
(verify both transports); browser-carried cookies mask the real auth tier; no-native-ID
records (documented text-key fallback, never fabricate synthetic IDs); Bearer token on a
public directory usually means wrong endpoint (heuristic, not law — tier 2 exists);
decoy page; self-submitting GET form with zero XHR.

**case-files/:** starts empty except README.md (entry template).

## Out of scope

- Login-gated APIs (credential flows, ToS questions) — separate problem, skill stops there.
- Modifying `discover-store-api` or `comp_website_scrapper` in any way.
- Rate-limit/etiquette design and downstream ingestion — the contract doc ends the job.
- Git-tracking the skill folder (user chose plain global; revisit if audit trail needed).
