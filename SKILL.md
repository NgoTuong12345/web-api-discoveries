---
name: api-discoveries
description: Discover, verify, and document the data API behind ANY greenfield website — the endpoint that serves its catalog, listings, locations, prices, or content. Use when reverse-engineering how a site loads its data, finding an undocumented backend, or capturing a site's data feed for scraping. browser-harness drives the live capture; the user solves any CAPTCHA manually. Delivers a clean-shell-replay-verified api-contract.md. Self-improving: harvests new platform fingerprints and traps into its own learning files each session. NOT for VN retail store locators inside DKKD.gov (use discover-store-api) or login-gated APIs (out of scope).
---

# API Discoveries

Get from "here's a website" → a verified data endpoint → a replay-verified `api-contract.md`
another tool (or a human) can consume. This is a **domain-neutral** generalization of
`discover-store-api`; it does not assume any particular ingestion engine exists.

This skill has two stable parts (this file, and the workflow below) and three parts that
**grow every session** — read them, then feed them:
- `fingerprints.md` — observable tell → platform family → action. Read in Phase 2.
- `traps.md` — known failure modes. Read in Phases 2–4.
- `case-files/` — one file per site solved. Grep-on-demand; not auto-loaded.

## When to use

Triggers: "discover the API behind this site", "find the endpoint that loads X", "reverse-
engineer how this site fetches data", "capture this site's data feed", "how does this page
get its listings/prices/locations".

**When NOT:**
- VN retail store locator being wired into DKKD's `comp_website_scrapper` → use
  `discover-store-api` (that skill knows the store_tracker handoff; this one doesn't).
- Data is behind a user login → **out of scope**. Document the finding (what's gated, where
  the login wall is) and stop. Never type credentials from a screenshot.

## Tooling: browser-harness is the capture engine

browser-harness (CDP) is the default and mandatory driver. First navigation of a session is
`new_tab(url)` — never `goto_url` (it clobbers the user's active tab). Screenshots drive
exploration: `capture_screenshot()` to read the page and find the interaction, then act.
`js()` for DOM dumps and in-browser `fetch()`. `http_get()` for static fetches (bundle
mining, platform-root probes) — no browser needed. Coordinate clicks (`click_at_xy`) pass
through iframes/shadow DOM. The `network-requests` interaction skill is the canonical way to
dump captured XHR/fetch — this skill points there, it does not re-document it.

**Boundary:** this skill records *what to look for* (platform shapes, traps). browser-harness
`interaction-skills/` records *how to drive the browser* (dialogs, dropdowns, iframes,
downloads). Don't duplicate browser mechanics here.

**Auth wall / CAPTCHA:** redirected to a login → stop and ask the user (this is also the
scope boundary). CAPTCHA on the way to public data → the user solves it manually, the script
continues after.

## The 6-phase workflow

Create a todo per phase. Each phase has an exit criterion; don't advance until it's met.

### Phase 1 — Recon (find where the data surfaces; watch for the decoy)
Find the page/interaction that renders the target data. Techniques: `capture_screenshot()`
to see it; `js()`-dump every `<a>` (href + textContent) and scan for the data you want;
check `sitemap.xml` and `robots.txt` for a machine index. **Decoy trap** (see `traps.md`
`decoy-page`): the obviously-named page may be a product filter or marketing page — the real
data can hide behind a delivery/address selector, a secondary nav item, or the footer. Don't
trust the page title; plan to interact.
**Exit:** candidate page + the interaction that loads the data identified.

### Phase 2 — Capture (browser-harness; consult fingerprints first)
**Read `fingerprints.md` now** — recognize the platform family before brute-forcing.
Primary technique: live network capture. Open the page, trigger the data load (select a
filter, click a pin, submit the form), dump the XHR/fetch traffic (URL, method, request
headers, body, response shape) via the `network-requests` interaction skill.

If **zero XHR fires**, walk the fallback ladder (all in `fingerprints.md`):
- JS-bundle mining — `http_get` the `/_next/static/chunks/*.js` (or equivalent), grep for
  `/api/`, `/gw/`, and accessor names. Surfaces endpoints the UI never calls on load.
- `__NEXT_DATA__` / SSR-embed parse — records or a count are embedded in the initial HTML.
- Platform discovery roots — `/wp-json/` route enum (WordPress), GraphQL introspection probe.
- Self-submitting GET form — confirm `drain_events()` shows only analytics pixels before
  concluding "no API"; then check the platform, then fall back to SSR scraping.
**Exit:** raw request + response captured (not inferred).

### Phase 3 — Classify (archetype)
Map the captured call to one domain-neutral archetype:

| Archetype | Shape |
|---|---|
| `single-call` | one request returns the whole dataset |
| `partitioned` | no all-data query; loop a closed enumeration (region / category / page) |
| `graphql` | POST `{query, variables}` |
| `ssr-html` | server-rendered HTML fragments, no JSON |
| `proximity-limited` | radius/nearest-N only, no census — sweep a closed anchor list, dedupe on key |
| `platform-rest` | stock CMS REST (e.g. WordPress CPT) found via route enum, not page traffic |

`proximity-limited` needs no convergence counting: the anchor list (regions/districts from
the site's own location API) is exhaustive by construction — query each anchor once.
**Exit:** archetype chosen.

### Phase 4 — Verify (two-tier replay — non-negotiable)
Existing `.md` docs are **hypotheses, never truth** until replayed live (`traps.md`
`docs-are-hypotheses`). In order:
1. **Tier 1 — clean-shell replay.** Re-issue the exact captured request with bare
   `requests`/`http_get` — no browser, no carried cookies. 200 with the same shape → this is
   a **public** contract.
2. **Tier 2 — if Tier 1 401/403s, bisect.** Re-add captured headers/cookies one at a time
   until it replays. The minimal set that works → this is a **session-carried** contract;
   name the load-bearing header(s). (See `traps.md` `cookie-masks-auth`.)
3. **WAF-vs-absent** — a probe that 403/404s from a clean shell must be re-tested with an
   in-browser `fetch()` via `js()` before you call the endpoint absent (`traps.md`
   `waf-vs-absent`).
4. **Count self-consistency** — assert `len(items) == <declared total>`, or a second count
   source agrees. A mismatch is a soft `truncated` flag, not a fatal error. Banner counts are
   a ceiling, never the assert target (`traps.md` `banner-counts-lie`).
**Exit:** replay returns 200 with the same shape + a tier (public / session-carried) assigned.

### Phase 5 — Deliver (contract doc + handoff hook)
Write `api-contract.md` (default: `docs/api-contracts/<domain>.md` in the current project)
from this template:
```
# <domain> API contract
- verified: YYYY-MM-DD (clean-shell replay)
- archetype: <one of the six>
- tier: public | session-carried (load-bearing headers: <list>)
- endpoint: <METHOD> <url>
- request: <headers that matter, body/params, partition/pagination scheme>
- response: <path to records, record shape, the stable record key>
- count: <n> — cross-check: <how verified>
- traps: <trap-names that applied>
```
**Handoff hook:** if the current project has a config-driven ingestion engine (a fetcher
that reads per-source config — e.g. `comp_website_scrapper/store_tracker`), offer to wire the
contract into its schema instead of leaving a standalone doc. If no such engine exists, the
contract doc is the deliverable.
**Exit:** `api-contract.md` written (or wired into the project engine).

### Phase 6 — Harvest (self-improvement — runs on success AND failure)
Dead ends are traps worth recording. Before ending the session:
1. **Case file** — write `case-files/<domain>.md` from the template in `case-files/README.md`.
2. **Generalize** — did this session reveal a durable, novel platform tell or failure mode?
   - **Grep `fingerprints.md` / `traps.md` first.** If a matching entry exists, EDIT it
     (refine/correct) — never append a rival duplicate.
   - New entry: map-not-diary, ≤ 8 lines, evidence anchor (`site, YYYY-MM-DD`) mandatory.
   - Appends/edits to these learning files are **autonomous** — report them in the session
     summary, don't ask permission.
3. **SKILL.md edits** (a new phase, a new archetype, changed doctrine) are **NOT autonomous**
   — propose them to the user and get explicit confirmation. The stable core changes
   deliberately; that's what stops the skill from rotting.
4. **Anti-bloat** — if a learning file exceeds ~150 lines, *propose* (don't auto-run) a
   compaction pass merging near-duplicates into generalized entries.
**Exit:** case file written; learning files updated or an explicit "nothing novel this session".

## Red flags (stop — you're rationalizing)

| Thought | Reality |
|---|---|
| "The doc lists the endpoint, I'll use it" | Docs are hypotheses. Clean-shell replay first (`traps.md` docs-are-hypotheses). |
| "It worked in the browser, ship it" | Browser carries cookies. Tier-1 clean-shell replay, then bisect if it 401s. |
| "No XHR fired, this site has no API" | Check the platform first (`fingerprints.md`): `/wp-json/`, `__NEXT_DATA__`, bundle strings, sitemap. |
| "The banner says N, count done" | Banners lie. Assert against a real count source. |
| "The nearby feed returns data, good enough" | Radius feeds undercount metros. Find the region feed or sweep anchors. |
| "Name/address is unique enough to dedupe" | Both drift. Use the site's own ID; if none exists, a confirmed-unique text field, documented — never a fabricated ID. |
| "/wp-json/ 404'd, nothing here" | Re-test in-browser `fetch()` — could be a WAF, not absence. |
| "This session found nothing worth saving" | Failed discoveries are traps. Write the case file; record the dead end. |

## References
- `fingerprints.md`, `traps.md`, `case-files/` — this skill's growing knowledge (read them).
- browser-harness `network-requests` interaction skill — how to dump captured XHR/fetch.
- browser-harness `interaction-skills/` — dialogs, dropdowns, iframes, shadow DOM, downloads.
- `discover-store-api` (project-local, DKKD.gov) — the VN-retail specialization this
  generalizes; consult it for store_tracker handoff specifics, don't duplicate it here.
