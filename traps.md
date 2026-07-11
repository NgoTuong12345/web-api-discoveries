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

### cloudrity-waf-burst-reset
signal: Rapid back-to-back requests (no delay) to a site start failing mid-sweep with `ConnectionResetError`/`RemoteDisconnected`/`SSLEOFError`, after ~10 consecutive requests worked fine.
reality: `Server: Cloudrity` (a VN CDN/WAF) throttles bursty traffic per-connection — not an auth wall, not the endpoint breaking. Retried pages succeed.
action: Add a ~1s delay between requests when harvesting more than a handful of pages from a `Server: Cloudrity` site; don't mistake the reset for a dead endpoint or a session requirement.
evidence: moit.gov.vn `Content.Listing` redraw endpoint, 2026-07-11.
