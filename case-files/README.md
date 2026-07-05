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
