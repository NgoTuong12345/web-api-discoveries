# nganhhang.vn

- date: 2026-07-11
- archetype: ssr-html (partitioned pagination via `?page=N` query param, plain HTML cards)
- endpoint: `GET https://nganhhang.vn/<category-slug>/?page=<N>` and `GET https://nganhhang.vn/search/?keyword=<kw>&page=<N>`
- tier: public (no cookies needed; site issues a `PHPSESSID` but never checks for one inbound)
- record key: URL slug/permalink at listing level (no numeric ID on listing cards); numeric `IdArticle` only server-rendered on the record's own detail page (hidden add-to-cart form input)
- count: no declared total anywhere (no banner). Only signal is the paginator's own last-page link; ~520+ for the Logistics category (52 pages × 10) — see clamp trap before trusting this as exact.
- traps hit: `clamped-last-page` (new this session — `?page=` far beyond the paginator's last link returns the last page's content again instead of emptying, so loop-until-empty convergence never terminates here)
- notes: VITIC (Trung tâm Thông tin Công nghiệp và Thương mại) — same ministry as `moit.gov.vn` (see `moit-gov-vn.md`) but a completely different platform/vendor: PHP/jQuery, Cloudflare-fronted, `ajax.php?modul=X&sub=Y&method=Z` gateway for commerce actions only (add-to-cart, newsletter) — no data reads through it. No CAPTCHA/JS-challenge hit on plain `http_get` GETs; fully scriptable without a browser. Site is a paid report/newsletter storefront (~90 category slugs across Nông nghiệp/Công nghiệp/Logistics/Bản tin/Nghiên cứu thị trường); this session verified the archetype on one category + search, did not crawl the full taxonomy.
