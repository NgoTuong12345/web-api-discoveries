# moit.gov.vn

- date: 2026-07-11
- archetype: ssr-html (partitioned pagination, HTML fragment response)
- endpoint: POST https://moit.gov.vn/?module=Content.Listing&moduleId=12&cmd=redraw&site=2005517&url_mode=rewrite&submitFormId=12&moduleId=12&page=Article.Download.list&site=2005517
- tier: public (no cookies/auth; only `X-Requested-With: XMLHttpRequest` needed)
- record key: `/upload/2005517/<YYYYMMDD>/<filename>` download path (no numeric ID on rows)
- count: 432 (18 full pages × 24; page 19 empty — clean boundary)
- traps hit: `moit-cloudrity-waf-throttle` (new), pagination anchors are `href="javascript:void(0)"` with a jQuery-delegated click handler — coordinate clicks are fragile here, used `js()` to call `element.click()` directly and captured the resulting POST via `drain_events()`.
- notes: Full site nav has 5 statistics sub-portals (idea.gov.vn = dead IIS default page, 2 survey subdomains = ASP.NET WebForms, main-site `/thong-ke/bao-cao-tong-hop` = the live one). The `?module=Content.Listing&moduleId=N&cmd=redraw&...&widgetCode=...&parentId=...&categoryId=...` shape is almost certainly the same generic listing-widget engine behind this CMS's other sections (news, legal documents) — swap `parentId`/`categoryId`/`widgetCode` to repoint it. Not verified this session (scope was statistics only). Data itself is an index of monthly `.doc`/`.xls` report files, not a live queryable table.
