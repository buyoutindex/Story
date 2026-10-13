# Build Timetable: weekly sprints from mid-December 2026

Status: plan, 13 October 2026. Calibrated against the transaction-monitor sprint of
6 to 15 October 2026 and its working pattern.

## Calibration

What the transaction-monitor sprint showed (from its sprint record):

- Four working days (6, 8, 9, 10 October) shipped six planned features (universe
  loader, PSC projection, market stats, ops fixes, accounts panel, events log plus
  alerts) and roughly six unplanned performance fixes: about 14 PRs merged, numbered
  #143 to #158, each a one-day build session followed by review, rebase and an
  operator gate.
- Pattern: one coordinator session scopes and reviews; build sessions run in
  parallel on disjoint files (Track A and Track B); the operator merges, applies
  migrations and runs gates. Test suite grew 1361 to 1597 in the sprint.
- That velocity sits on a mature codebase with a large test harness and settled
  conventions. A greenfield build does not get that for the first three sprints:
  early work is sequential (schema before renderer before pipeline) and the harness
  has to be written as we go.

Assumptions for this plan:

| Phase | Features per week | Why |
|---|---|---|
| Sprints 1 to 3 (foundations) | 3 to 4 | Sequential dependencies, one track, harness being built |
| Sprints 4 to 9 | 5 to 7 | Two disjoint tracks (platform, commercial), conventions settled |
| Sprints 10 onward | 5 to 8 | Mostly configuration and adapters on a stable core |

A "feature" is one PR-sized unit: a module with tests and its docs. Operator time
is about one hour a day for review, merge and gates, plus the non-code tasks listed
per sprint. Sprints are Monday to Friday. Where a sprint has fewer days, scope is
cut from the bottom, the transaction-monitor rule.

One decision this plan takes: because the rendering and drafting engine is being
built anyway, sending runs on our own SES stack from the start rather than
beehiiv for a year. That removes a migration and keeps the subscriber and event
data in one database. beehiiv stays the fallback if sending slips past sprint 5.

## Timetable

| Sprint | Dates | Track A (platform) | Track B (commercial, content, ops) | Operator tasks | Exit gate |
|---|---|---|---|---|---|
| 0 | before 14 Dec | | | Buy thecommon.london and .co.uk variant; secure handles; AWS account in eu-west-2 (London), request SES production access (takes days); ICO registration; company; draft style guide v1 for The Common; write 20 seed sources list | Domain live, SES out of sandbox |
| 1 | 14 to 18 Dec | Monorepo, Postgres schema (titles, subscribers, consents, issues, sections, sources, items, stories, events, businesses, sponsors, campaigns, slots, sends, send_events), tenant config loader, The Common config, rendering engine: issue object to MJML email and to web page, CI with tests | DKIM, SPF, DMARC on the sending subdomain; style guide v1 into `titles/the-common/` | Review daily | A hand-written issue object renders to email and web in The Common's brand |
| 2 | 21 to 23 Dec (3 days) | Source store and adapters v1: RSS, Wandsworth and Lambeth council news and planning, TfL, Met Police local, venue and Eventbrite events; nightly ingest job | | Seed subscriber list of 300 (friends, societies, BIDs) | Nightly run fills `items` from real sources |
| break | 24 Dec to 3 Jan | | | | |
| 3 | 4 to 8 Jan | Claude triage and clustering (Haiku, Batch API), drafting pipeline (Opus 5.5) from brief plus style guide with prompt caching, approval screen (edit, approve, send-later) | Anchor sponsor outreach starts (BIDs, Battersea Power Station, one estate agency) | Approve test issues; refine voice | First end-to-end issue: sources to draft to approved render, internal only |
| 4 | 11 to 15 Jan | Signup with double opt-in and consent log, SES sending via configuration set, SNS event capture into `send_events`, one-click unsubscribe, archive pages, welcome sequence | Test sends to the 300 seed list begin (warm-up), Monday and Thursday | Review deliverability in Postmaster Tools | Two real issues delivered to seed list; events captured |
| 5 | 18 to 22 Jan | Events calendar (public, organiser submission, moderation), business directory seeded from OSM and Places (noindex until enriched), weekend page, four area guides, sitemap and schema | Referral block with per-reader link, WhatsApp Channel, Nextdoor application, press note drafted | Approve launch content | Site complete enough for a press mention to land on real pages |
| 6 | 25 to 29 Jan | Hardening: monitoring, retries, send-time guard, KPI dashboard (subs, opens, clicks, unsubs, referral share) | **Public launch of The Common: Thursday 28 Jan 07:00.** Launch note to Press Gazette, HTFP, Behind Local News; cards to 40 venues; Reddit thread | Final approve; founding partner signed | Issue 1 public; KPI dashboard live |
| 7 | 1 to 5 Feb | Prospect pipeline: Google Places scrape for SW11/SW4/SW8/SW12, Companies House match (Ltd vs sole trader), enrichment, segments | Cold-email sequence engine (3 touches), auto-generated ad mockup inside a real issue, PECR routing (email only to incorporated) | First sequences to 300 prospects | Pipeline board shows prospects, touches, replies |
| 8 | 8 to 12 Feb | Ad calendar and slot model; booked slots inserted into issues automatically with "Partner" label; Stripe self-serve booking for classifieds, spotlights, listings, jobs | Dub.co tracked links per campaign; Monday sponsor report (delivered, clicks, CTR vs issue, next slots) | Publish rate card at 2,000 subs if reached | First paid slot runs and is reported automatically |
| 9 | 15 to 19 Feb | Sponsor portal (campaign list, creatives, Metabase or in-app stats, book again), media kit PDF generated from live stats | Title-2 naming checks (The Lordship), source list and style guide for Dulwich | Choose title-2 name and domain | Portal live for sponsors; media kit auto-generated |
| 10 | 22 to 26 Feb | Membership: Stripe subscriptions, member edition render (Saturday "Where to go"), member perks list (Common Card partners), 10%-to-cause ledger | Founding member cohort launch (100 at locked price) | Approve member edition | First paying members; Saturday edition sent |
| 11 | 1 to 5 Mar | Second title config `titles/the-lordship/` end to end, proving the one-day onboarding; cross-title sponsor packages; shared prospect pipeline per title | Dulwich seed list and anchor sponsor outreach | | The Lordship sends its first internal issue within one day of config |
| 12 | 8 to 12 Mar | Trade-journal title kind: `kind = trade`, events-consumer adapter (offset-based, idempotent) against the transaction-monitor `events_out` contract, snapshot-query adapter for `universe.companies`, trade register in style-guide loader, CPM pricing and Pro gated views | Raise the read-access and consumer-registration task in the transaction-monitor queue; pick the first vertical from event volume | Rule on the TM-side task in that repo | Adapter runs against staging data; first vertical chosen |
| 13 | 15 to 19 Mar | Buffer and hardening: list hygiene job (90-day prune with re-engagement), backups, runbooks, cover plan for issues | **The Lordship public launch: Thursday 18 Mar.** Print drop planning for Clapham Common summer | | Two local titles live; platform v1 complete |
| 14+ | from 22 Mar | Trade title 1 build on real data (sprints 14 to 16), title 3 (Fulham or Crouch End) config, per-reader personalisation only if data says so | Ongoing sales, membership growth, print event | | Trade title 1 live by end of April |

## Summary

| Milestone | Date | Sprint |
|---|---|---|
| First end-to-end drafted issue (internal) | 8 Jan | 3 |
| Seed-list sends begin | 12 Jan | 4 |
| The Common public launch | 28 Jan | 6 |
| First paid slot auto-inserted and reported | 12 Feb | 8 |
| Membership live | 26 Feb | 10 |
| The Lordship public launch | 18 Mar | 13 |
| Platform v1 complete (two local titles, trade kind ready) | 19 Mar | 13 |
| First trade journal live | end April | 16 |

That is 13 working sprints (about 12 weeks after the Christmas break) to the
platform described in PLAN.md, consistent with the 12 to 16 week estimate there,
with the first title live at sprint 6. The transaction-monitor comparison suggests
this is achievable at one build session per feature with two tracks from sprint 4;
the risk is in sprints 1 to 3, where nothing can be parallelised and the harness is
being written.

## Risks to the dates

- SES production access and warm-up: request in sprint 0; sending to the seed list
  must start by sprint 4 or the launch slips a week.
- Christmas: sprint 2 is three days and the break is ten; scope there is deliberately
  thin.
- Style guide and voice: operator time, not build time. If the voice is not right by
  sprint 3's test issues, hold the launch, not the platform.
- Anchor sponsor: not a build dependency, but launching without one costs the
  "brought to you by" credibility. Start outreach in sprint 3.
- Transaction-monitor access: owned by that repo's process and blackout calendar
  (its record shows a 16 Oct to 9 Nov blackout); the sprint-12 adapter is built
  against the documented contract and staging data, so a delay there does not block
  the local titles.
