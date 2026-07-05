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
