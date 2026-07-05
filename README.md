# api-discoveries

A **self-improving** Claude Code skill that discovers, verifies, and documents the data API
behind **any greenfield website** — the endpoint that actually serves its catalog, listings,
store locations, prices, or content.

Point it at a site and it walks a disciplined 6-phase workflow (recon → capture → classify →
verify → deliver → harvest), driving the browser via [browser-harness](https://github.com/)
(CDP). It ends with a **clean-shell-replay-verified** `api-contract.md`. Each session it
learns: new platform patterns and failure modes get harvested back into its own knowledge
files, so it gets sharper with use.

## Why it exists

It generalizes a project-local skill (`discover-store-api`, built for reverse-engineering
Vietnamese retail store-locator APIs) into a domain-neutral tool for any site. The hard-won
traps from ~15 real brand sites are pre-seeded so it's useful on day one.

## Layout

```
api-discoveries/
├─ SKILL.md          # STABLE core: the 6-phase workflow, browser-harness integration, harvest protocol
├─ fingerprints.md   # GROWS: observable tell → platform family → what to do
├─ traps.md          # GROWS: known failure modes (signal / reality / evidence / action)
├─ case-files/       # GROWS: one short file per site solved (grep-on-demand, not auto-loaded)
│  └─ README.md      #   per-site entry template
└─ docs/             # design spec + implementation plan
```

**The split is the point.** `SKILL.md` is the invariant workflow and changes only
deliberately (with human confirmation). The three learning files grow freely and cheaply:
the skill reads them before acting and appends to them afterward.

## The 6 phases

1. **Recon** — find where the target data surfaces; watch for decoy pages.
2. **Capture** — browser-harness network capture; fallback ladder (JS-bundle mining,
   `__NEXT_DATA__`, platform discovery roots) when zero XHR fires.
3. **Classify** — one of six archetypes: `single-call`, `partitioned`, `graphql`, `ssr-html`,
   `proximity-limited`, `platform-rest`.
4. **Verify** — two-tier replay: clean-shell (no cookies) = *public*; header/cookie bisection
   = *session-carried*. Existing docs are hypotheses until replayed live.
5. **Deliver** — write `api-contract.md`; if the project has an ingestion engine, offer to
   wire the contract into its schema.
6. **Harvest** — record novel platform tells and traps back into the learning files
   (autonomously); propose core-workflow changes for human review.

## Scope

- **In:** public **and** session/cookie-carried data APIs.
- **Out:** login-gated APIs (the skill documents the finding and stops — no credential
  handling).

## Install

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/NgoTuong12345/web-api-discoveries.git ~/.claude/skills/api-discoveries
```

Then register a trigger in your `~/.claude/CLAUDE.md`:

```markdown
# api-discoveries
- **api-discoveries** (`~/.claude/skills/api-discoveries/SKILL.md`) - discover, verify, and document the data API behind any greenfield website. Trigger: `/api-discoveries`
When the user types `/api-discoveries`, invoke the Skill tool with `skill: "api-discoveries"` before doing anything else.
```

Invoke with `/api-discoveries` (or describe the task: "find the API behind this site").
