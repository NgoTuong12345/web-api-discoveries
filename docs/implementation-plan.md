# `/api-discoveries` Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a self-improving global skill `~/.claude/skills/api-discoveries/` that discovers, verifies, and documents the data API behind any greenfield website — generalizing the project-local `discover-store-api` skill without modifying it.

**Architecture:** A stable-core `SKILL.md` (6-phase workflow + harvest protocol) plus three append-only "learning files" (`fingerprints.md`, `traps.md`, `case-files/`) pre-seeded with domain-neutral knowledge. The skill's final phase harvests new platform tells and traps back into the learning files. Registered via a trigger block in `~/.claude/CLAUDE.md`.

**Tech Stack:** Markdown skill files only. No code. browser-harness (CDP) is the named capture engine. Verification uses `http_get()`/bare `requests` for clean-shell replay and in-browser `fetch()` via `js()` for the session-carried tier.

## Global Constraints

- **Do not modify** `discover-store-api`, `comp_website_scrapper/`, or any DKKD engine code. This plan touches ONLY files under `~/.claude/skills/api-discoveries/` and one appended block in `~/.claude/CLAUDE.md`.
- All skill files are **global** at `C:\Users\Admin\.claude\skills\api-discoveries\` — NOT inside the DKKD.gov repo.
- Learning files are **map-not-diary**: durable platform shapes, not session narration. Every `fingerprints.md`/`traps.md` entry ≤ 8 lines and carries an evidence anchor (site + date).
- Scope: public AND session/cookie-carried APIs. Login-gated APIs are out of scope — the skill documents the finding and stops.
- SKILL.md frontmatter `name:` must be exactly `api-discoveries` (matches the invocation trigger).
- No fabricated IDs, no trusting `.md` docs without live replay, no using banner counts as census targets — these doctrines are load-bearing and must appear verbatim-in-spirit in the relevant phases.
- Because these are files outside the git repo, there is no per-task `git commit` step. Each task's "verify" is a content check (file exists, key strings present). A single optional archival copy into the repo is the final task.

---

## File Structure

| File | Responsibility |
|---|---|
| `~/.claude/skills/api-discoveries/SKILL.md` | Stable core: frontmatter, when-to-use, 6-phase workflow, browser-harness integration, harvest protocol, red-flags table, references |
| `~/.claude/skills/api-discoveries/fingerprints.md` | Seeded + growing: table of observable tell → platform family → what to do |
| `~/.claude/skills/api-discoveries/traps.md` | Seeded + growing: structured entries (signal / reality / evidence / action) |
| `~/.claude/skills/api-discoveries/case-files/README.md` | Entry template + rules for per-site case files (the dir starts otherwise empty) |
| `~/.claude/CLAUDE.md` | Append a `/api-discoveries` trigger block (existing per-skill registration pattern) |

Verification uses the Bash tool (Git Bash on this Windows machine); `~` expands to `C:\Users\Admin`. Skill file reads use the Read tool with the full `C:\Users\Admin\.claude\...` path.

---

### Task 1: Seed `fingerprints.md`

**Files:**
- Create: `C:\Users\Admin\.claude\skills\api-discoveries\fingerprints.md`

**Interfaces:**
- Produces: the platform-recognition table SKILL.md Phase 2 tells the reader to consult before brute-forcing. Column contract: `Tell | Family | What to do`.

- [ ] **Step 1: Write the file**

Write `C:\Users\Admin\.claude\skills\api-discoveries\fingerprints.md` with exactly this content:

```markdown
# Platform Fingerprints

Observable tell → platform family → what to do. Read this in Phase 2 (Capture) BEFORE
brute-forcing endpoints. Append new rows via the Phase 6 harvest protocol (novelty-grep
first; ≤ 8 lines; evidence anchor `site, YYYY-MM-DD`).

| Tell (what you observe) | Family | What to do |
|---|---|---|
| `wp-includes`/`wp-emoji-release.min.js` in `<script>`s; zero `/api/` in bundles | Stock WordPress | Hit `/wp-json/` and enumerate `routes`; grep for a domain-shaped custom post type (`store`, `mall`, `product`, `event`). REST often exposes it with zero page reference. |
| `/wp-json/wp/v2/<cpt>` responds | WordPress REST CPT | No auth; `per_page` hard-capped at 100 (paginate); **total is `X-WP-Total` HTTP header, not JSON body** — don't wire a body count. Multilingual (Polylang/WPML) mixes languages as separate posts — filter one language first. |
| `__NEXT_DATA__` JSON blob in HTML; `/_next/static/chunks/*.js` | Next.js SPA | Data is embedded in `__NEXT_DATA__` (parse it) AND endpoints are in the chunks — grep chunks for `/api/`, `/gw/`, accessor names. Build-ID in the CDN asset path rotates on redeploy. |
| Single `POST …/graphql` with `{query, variables}` | GraphQL | One query can return the whole dataset. Try an introspection probe if the query shape is unknown. Watch for `X-Client-Type`-style required headers. |
| `/…/ViewMore…` etc. returning HTML `<li data-id="…">`, no JSON | Legacy ASP.NET MVC SSR | Parse `data-id` as the record key. **No count API** — the only total is a banner number (don't trust it as census). |
| `api.<brand>.com/gw/…` or `webapi.<brand>.com/gw/…`; envelope `{code,data,serverTime}` | Central JSON gateway | `code==0` = ok; service segment in path (`bus-…`, `mt/`, `it/`) is mineable from JS chunks. One base host fronts many microservices. |
| Request carries a tenant/`platformId` param; records carry a brand-suffixed key | Multi-tenant commerce platform | Third-party backend shared across unrelated brands. One tenant's endpoint can span sister sub-brands in one response — check for siblings before assuming single scope. Census query often 401s; only radius query is public. |
| Only a "nearby"/radius query exists; page 2 empty; two anchors → different totals | Proximity-limited feed | No census endpoint. Enumerate a closed anchor list (regions/districts) from the site's own location API, query each once, dedupe on record key. No convergence counting — the anchor list is exhaustive by construction. |
| A "search" form is `<form method="GET">` with no `action`; submit does full page nav; `drain_events()` shows only analytics pixels (GTM/Meta/admicro) | Server-rendered GET form, no XHR | Not an SPA. Don't conclude "no API" — check the platform (WordPress `/wp-json/`, sitemap) before falling back to SSR-HTML scraping. |
```

- [ ] **Step 2: Verify content landed**

Run: `grep -c '^|' ~/.claude/skills/api-discoveries/fingerprints.md`
Expected: `9` (1 header row + 1 separator + 7… ) — actually prints `10` (header, separator, 8 data rows). Accept any value ≥ 9.

Run: `grep -q 'X-WP-Total' ~/.claude/skills/api-discoveries/fingerprints.md && grep -q 'platformId' ~/.claude/skills/api-discoveries/fingerprints.md && echo OK`
Expected: `OK`

---

### Task 2: Seed `traps.md`

**Files:**
- Create: `C:\Users\Admin\.claude\skills\api-discoveries\traps.md`

**Interfaces:**
- Produces: the trap corpus SKILL.md Phases 2–4 consult. Entry contract: `### <name>` heading then `signal:` / `reality:` / `evidence:` / `action:` lines (plus optional `capture:` line).

- [ ] **Step 1: Write the file**

Write `C:\Users\Admin\.claude\skills\api-discoveries\traps.md` with exactly this content:

```markdown
# Traps

Known ways API discovery goes wrong. Read in Phases 2–4. Append via the Phase 6 harvest
protocol: grep for an existing entry first (edit it if contradicted/refined — don't append a
rival); ≤ 8 lines; `evidence:` anchor mandatory. Optional `capture:` line when the trap is
about how to *observe*, not what the data means.

### docs-are-hypotheses
signal: An existing `.md` / README lists an endpoint, host, path, or count.
reality: Docs are fabrication-prone. A prior doc invented a host (→ HTTP 000) and a path (→ 404) wholesale.
action: Replay the exact request headless from a clean shell BEFORE trusting any documented value.
evidence: Long Châu fabricated doc, 2026-07.

### banner-counts-lie
signal: Homepage/marketing banner states "N stores / N locations".
reality: Marketing numbers are not censuses (regional rollups, combined networks, aspirational).
action: Use as a sanity ceiling only. Assert `len(items)` against a real count endpoint, never the banner.
evidence: WinMart "62 provinces" (really 57); DMX "2948" (combined network), 2026-07.

### cookie-masks-auth
signal: The request works in the browser, so you assume it's public.
reality: The browser silently carries session cookies; the endpoint may need them.
action: Two-tier verify. Clean-shell replay (no cookies) → public. If that 401s, bisect headers/cookies to the minimal set → session-carried. Name the load-bearing header.
evidence: General; formalized 2026-07-05.

### waf-vs-absent
signal: `/wp-json/` (or any probe) returns 403/404 from your script.
reality: A WAF blocking bare requests looks identical to a genuinely disabled/absent endpoint.
action: Test BOTH a clean-shell `http_get` AND an in-browser `fetch()`. Disagreement = WAF/headers. Agreement on 404 = truly absent — then fall back (SSR scrape), don't keep hunting.
capture: in-browser `fetch()` via `js()` carries the page's origin/cookies; `http_get` doesn't.
evidence: Co.opXtra `/wp-json/` truly disabled vs AEON's live one, 2026-07.

### multilingual-double-count
signal: A CMS list returns ~2× the expected records with unrelated IDs.
reality: Bilingual sites (Polylang/WPML) store each language as a separate post — no shared key.
action: Find the field that actually varies by language (often `link` path `/en/`), filter to one language BEFORE dedup/count. `?lang=` can be a silent no-op — verify the count changed.
evidence: AEON `/wp/v2/mall` 71 raw → 36 real, 2026-07.

### radius-undercounts-metros
signal: A "nearby"/lat-long feed returns results and seems complete.
reality: Radius feeds undercount dense metros (~2.5× under for one brand post ward-mergers).
action: If a region/province feed exists, use it. If only radius exists, sweep a closed anchor list (see proximity-limited fingerprint), never trust one anchor.
evidence: WinMart radius feed, 2026-07.

### no-native-id
signal: Records have no stable ID — no `data-id`, no detail-page href, no coordinates.
reality: Some small/greenfield sites genuinely have zero native key (checked the click handlers, not just the DOM).
action: Fall back to a confirmed-unique text field as the key ONLY if every value was read + confirmed unique AND carries a disambiguating suffix. Document the accepted risk. NEVER fabricate a synthetic ID (hash/row-index) — it hides instability.
evidence: Co.opXtra, 6 stores, no ID anywhere, 2026-07.

### bearer-on-public-directory
signal: A directory/listing endpoint wants a Bearer token.
reality: Public directories are usually unauthenticated; Bearer is for account/orders. Often you're on the wrong endpoint.
action: Look for the public sibling first. Heuristic, not law — session-carried public endpoints do exist (see cookie-masks-auth).
evidence: General store-locator pattern, 2026-07.

### decoy-page
signal: The obvious "store system"/"catalog" page is titled like the data directory.
reality: It can be a product filter or marketing page; the real data hides behind a delivery/address selector, a secondary nav item, or the footer.
action: Don't trust the page title. Dump every `<a>` (href+text), plan to interact with selectors/dropdowns/map pins.
evidence: WinMart real directory behind "Giao Hàng" selector, 2026-07.
```

- [ ] **Step 2: Verify all entries present**

Run: `grep -c '^### ' ~/.claude/skills/api-discoveries/traps.md`
Expected: `9`

Run: `grep -c '^evidence:' ~/.claude/skills/api-discoveries/traps.md`
Expected: `9` (every entry has an evidence anchor)

---

### Task 3: Seed `case-files/README.md`

**Files:**
- Create: `C:\Users\Admin\.claude\skills\api-discoveries\case-files\README.md`

**Interfaces:**
- Produces: the per-site case-file template referenced by SKILL.md Phase 6. The directory otherwise starts empty.

- [ ] **Step 1: Write the file**

Write `C:\Users\Admin\.claude\skills\api-discoveries\case-files\README.md` with exactly this content:

```markdown
# Case Files

One short file per site solved, written in Phase 6 (Harvest). These are grep-on-demand
evidence anchors — NOT auto-loaded into context on skill invocation (keeps the skill cheap
as the corpus grows). Search here when a new site smells like a past one.

Filename: `<domain-without-tld>.md` (e.g. `aeon-com-vn.md`). Keep each under ~30 lines —
map, not diary. If a durable lesson generalizes beyond this one site, it also belongs in
`fingerprints.md` or `traps.md`, not only here.

## Template

```
# <domain>

- date: YYYY-MM-DD
- archetype: single-call | partitioned | graphql | ssr-html | proximity-limited | platform-rest
- endpoint: <METHOD> <url>
- tier: public | session-carried (load-bearing headers: <list>)
- record key: <field>
- count: <n> (cross-check: <how verified>)
- traps hit: <trap-name>, <trap-name>
- notes: <anything a future session needs and can't re-derive fast>
```
```

- [ ] **Step 2: Verify**

Run: `test -f ~/.claude/skills/api-discoveries/case-files/README.md && grep -q 'archetype:' ~/.claude/skills/api-discoveries/case-files/README.md && echo OK`
Expected: `OK`

---

### Task 4: Write the core `SKILL.md`

**Files:**
- Create: `C:\Users\Admin\.claude\skills\api-discoveries\SKILL.md`

**Interfaces:**
- Consumes: `fingerprints.md` (Task 1), `traps.md` (Task 2), `case-files/README.md` (Task 3) — referenced by relative path.
- Produces: the invocable skill. Frontmatter `name: api-discoveries` is the trigger Task 5 registers.

- [ ] **Step 1: Write the file**

Write `C:\Users\Admin\.claude\skills\api-discoveries\SKILL.md` with exactly this content:

```markdown
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
```

- [ ] **Step 2: Verify frontmatter and structure**

Run: `grep -q '^name: api-discoveries$' ~/.claude/skills/api-discoveries/SKILL.md && echo NAME_OK`
Expected: `NAME_OK`

Run: `grep -c '^### Phase ' ~/.claude/skills/api-discoveries/SKILL.md`
Expected: `6`

Run: `grep -q 'fingerprints.md' ~/.claude/skills/api-discoveries/SKILL.md && grep -q 'traps.md' ~/.claude/skills/api-discoveries/SKILL.md && grep -q 'case-files' ~/.claude/skills/api-discoveries/SKILL.md && echo REFS_OK`
Expected: `REFS_OK`

- [ ] **Step 3: Verify all six archetype names are consistent across the file**

Run: `for a in single-call partitioned graphql ssr-html proximity-limited platform-rest; do grep -q "$a" ~/.claude/skills/api-discoveries/SKILL.md && echo "$a ok" || echo "$a MISSING"; done`
Expected: six `ok` lines, no `MISSING`.

---

### Task 5: Register the trigger in `~/.claude/CLAUDE.md`

**Files:**
- Modify: `C:\Users\Admin\.claude\CLAUDE.md` (append a new block at end of file)

**Interfaces:**
- Consumes: the `name: api-discoveries` from Task 4.
- Produces: the `/api-discoveries` slash-trigger, matching the existing per-skill registration pattern in that file.

- [ ] **Step 1: Read the tail of the file to match the existing pattern**

Read the last ~15 lines of `C:\Users\Admin\.claude\CLAUDE.md` to confirm the registration-block format (heading + bullet + "When the user types … invoke the Skill tool").

- [ ] **Step 2: Append the registration block**

Append this block to the end of `C:\Users\Admin\.claude\CLAUDE.md` (match the exact spacing/wording of the neighbouring blocks; if their phrasing differs from below, follow theirs):

```markdown

# api-discoveries
- **api-discoveries** (`~/.claude/skills/api-discoveries/SKILL.md`) - discover, verify, and document the data API behind any greenfield website; self-improving via its own fingerprints/traps/case-files. Trigger: `/api-discoveries`
When the user types `/api-discoveries`, invoke the Skill tool with `skill: "api-discoveries"` before doing anything else.
```

- [ ] **Step 3: Verify the block landed**

Run: `grep -q '/api-discoveries' ~/.claude/CLAUDE.md && grep -q 'skill: "api-discoveries"' ~/.claude/CLAUDE.md && echo REGISTERED`
Expected: `REGISTERED`

---

### Task 6: End-to-end sanity check + repo archival copy

**Files:**
- Create: `C:\Users\Admin\Documents\KIS Research\DKKD.gov\docs\superpowers\reference\api-discoveries-skill-snapshot.md` (an in-repo pointer/snapshot, since the live skill lives outside the repo)

**Interfaces:**
- Consumes: all files from Tasks 1–5.
- Produces: a committed in-repo record that the global skill exists and where it lives (the skill folder itself is outside git).

- [ ] **Step 1: Verify the whole skill tree exists**

Run:
```bash
ls -R ~/.claude/skills/api-discoveries/
```
Expected: `SKILL.md`, `fingerprints.md`, `traps.md`, and `case-files/README.md` all present.

- [ ] **Step 2: Confirm no cross-file archetype drift**

Run:
```bash
grep -oh 'single-call\|partitioned\|graphql\|ssr-html\|proximity-limited\|platform-rest' ~/.claude/skills/api-discoveries/SKILL.md ~/.claude/skills/api-discoveries/fingerprints.md ~/.claude/skills/api-discoveries/case-files/README.md | sort -u
```
Expected: exactly the six archetype tokens, no stray variants (e.g. no `single_call`, no `json_national`).

- [ ] **Step 3: Write the in-repo snapshot pointer**

Write `docs/superpowers/reference/api-discoveries-skill-snapshot.md`:

```markdown
# api-discoveries skill (snapshot pointer)

The `/api-discoveries` skill is **global**, living at `~/.claude/skills/api-discoveries/`
(outside this repo). It generalizes this project's `discover-store-api` skill into a
domain-neutral, self-improving API-discovery workflow for any greenfield website.

- Design spec: `docs/superpowers/specs/2026-07-05-api-discoveries-skill-design.md`
- Plan: `docs/superpowers/plans/2026-07-05-api-discoveries-skill.md`
- Live files (not in git): `~/.claude/skills/api-discoveries/{SKILL.md,fingerprints.md,traps.md,case-files/}`

Relationship: `discover-store-api` is unchanged and remains authoritative for VN retail
store locators + the store_tracker handoff. Domain-neutral lessons flow one-way UP into the
global skill's learning files via its Phase 6 harvest protocol.
```

- [ ] **Step 4: Commit the in-repo pointer**

Run:
```bash
cd "C:/Users/Admin/Documents/KIS Research/DKKD.gov" && git add docs/superpowers/reference/api-discoveries-skill-snapshot.md && git commit -m "docs(reference): pointer to global api-discoveries skill

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```
Expected: one file committed.

- [ ] **Step 5: Final invocation smoke test (manual)**

In a fresh turn, type `/api-discoveries` and confirm the skill loads (frontmatter parses, SKILL.md is read). If it doesn't appear, re-check Task 5's registration block.

---

## Self-Review

**Spec coverage:**
- Global location → Tasks 1–5 all write to `~/.claude/skills/api-discoveries/`. ✓
- Stable-core + learning-files architecture → Task 4 (core) + Tasks 1–3 (learning files). ✓
- 6-phase workflow → Task 4 Step 1, verified in Task 4 Step 2 (`grep -c '### Phase ' == 6`). ✓
- Contract doc + handoff hook → Phase 5 in Task 4. ✓
- Session-cookie scope (two-tier verify) → Phase 4 in Task 4 + `cookie-masks-auth` trap in Task 2. ✓
- Login-gated out of scope → "When NOT" + auth-wall rule in Task 4. ✓
- Seeded generalized knowledge → Tasks 1 (fingerprints) + 2 (traps); case-files empty except README (Task 3). ✓
- browser-harness integration per-phase → Task 4 "Tooling" section + per-phase mentions. ✓
- Harvest protocol w/ autonomy split + anti-bloat → Phase 6 in Task 4. ✓
- discover-store-api untouched → Global Constraints; no task touches it. ✓
- Registration → Task 5. ✓

**Placeholder scan:** No TBD/TODO. Every file's full content is inline. Verify steps use exact commands with expected output. ✓

**Type consistency:** The six archetype tokens (`single-call`, `partitioned`, `graphql`, `ssr-html`, `proximity-limited`, `platform-rest`) are used identically in Task 4's classify table, Task 1's fingerprints, Task 3's README template, and re-checked for drift in Task 6 Step 2. Trap names referenced in SKILL.md (`decoy-page`, `docs-are-hypotheses`, `cookie-masks-auth`, `waf-vs-absent`, `banner-counts-lie`) all exist as `### ` headings in Task 2. ✓

**Note on Task 1 Step 2 count:** the fingerprints table has 8 data rows + header + separator = 10 `|`-leading lines; the check accepts ≥ 9 to avoid a brittle exact match.
```
