# Local Newsletter Network: Business and Build Plan

Status: proposal (October 2026). This repo is being repurposed from a toy project into
the platform described here.

---

## 1. The thesis in one paragraph

A local newsletter is a small media business with three assets: an owned audience
(email list), a local-business graph (directory + events), and a sponsor pipeline.
The Scottsdale Scoop example shows one person running 15k subscribers and 3 issues a
week with six systems. Every one of those six systems is software plus a thin layer
of human judgment. The judgment layer (voice, editorial approval, sponsor
relationships) is what does *not* scale per city. The software layer scales to N
cities at near-zero marginal cost. The business is therefore: build the platform
once, then launch cities as cheaply and quickly as possible and let the platform
carry the operating load.

The two things that kill this model are (a) paying per-subscriber SaaS fees that scale
with the list, and (b) treating each city as a separate operation. Self-hosted sending
on Amazon SES and a single multi-tenant codebase fix both.

---

## 2. How it works: the operating loop for one city

Per issue (3x per week):

1. **Ingest** (automated, nightly): scrape/pull local news RSS, city press releases,
   permit filings, restaurant openings (Google Places new businesses), venue
   calendars, Eventbrite/Meetup, Reddit r/<city>, Instagram hashtags, Nextdoor
   public posts, school district notices, local sports schedules.
2. **Triage** (Claude, automated): dedupe, cluster into stories, score for local
   interest, flag anything needing a human eye (legal, deaths, controversy).
3. **Draft** (Claude, automated): produce a full issue draft in the city voice from
   a structured brief. Sections: lead story, 5 quick hits, events this week,
   new openings, sponsor slot(s), "ask the readers" prompt.
4. **Edit and approve** (human, 15 to 30 minutes): the editor reads, tweaks, hits
   approve. This is the only mandatory human step per issue.
5. **Send** (automated): render MJML/HTML, personalise, send via SES at 06:00 local.
6. **Measure** (automated): opens, clicks per link, sponsor click-throughs, bounces,
   complaints, unsubscribes. Feed back into triage weights and sponsor reports.

Per week:

- **Sponsor reports** go out Monday automatically (impressions, clicks, CTR vs.
  benchmark, renewal CTA).
- **Outbound** sequences fire to prospect businesses; replies land in one inbox per
  city; the CRM board updates from email events.

Continuously:

- **Events calendar** fills itself via organiser self-submission (with Claude
  moderation and auto-formatting).
- **Business directory** pages are generated for every local business (seeded from
  Google Places/OSM, enriched over time), each a thin SEO page that links back to
  the newsletter signup. This is the slow compounding free-acquisition channel.
- **Growth loops**: referral program, Instagram cross-post of each issue's lead
  story (auto-generated card image), partner swaps with nearby cities.

---

## 3. The six systems as software modules

| # | System (Scoop name) | What we build | Build effort |
|---|---|---|---|
| 1 | Scoop Desk | Ingest pipeline + Claude drafting with per-city style guide, editor UI with approve/send | 3 to 4 weeks |
| 2 | Events calendar | Public calendar, organiser submission form, moderation queue, ICS/embed | 1 week |
| 3 | Business directory | Programmatic SEO pages per business, claim-your-listing flow, categories, maps | 1 to 2 weeks |
| 4 | Ad sales | Sponsor portal (creatives, mockups, stats), ad calendar/inventory, Stripe checkout, Monday auto-reports | 2 to 3 weeks |
| 5 | Mailer | SES sending, list management, signup forms/API, bounce/complaint handling, archive, analytics (MailCheap-class features) | 2 weeks (or adopt MailCheap/Listmonk and integrate) |
| 6 | Outbound HQ | Prospect scraping (Google Places → businesses with ad budgets), sequenced email outreach, pipeline board, reply detection | 2 weeks |

Total: roughly 10 to 14 weeks of focused build for a solid v1 of the whole platform,
or about 6 weeks to a minimum viable "city in a box" (modules 1, 5 and a manual
version of 4).

### Build vs. adopt per module

- **Mailer**: do not write from scratch first. Options: MailCheap (MIT, Next.js, SES,
  single-admin; small project, roughly 130 stars, so expect to fork and own it),
  Listmonk (Go, mature, multi-list, SES-compatible, strong admin), or Keila. Listmonk
  is the lowest-risk base for multi-city. Either way, SES does the sending.
- **Directory and events**: build. These are mostly templated pages and forms and are
  where the multi-city data model matters most.
- **Drafting**: build. This is the moat and needs tight coupling to our data model.
- **CRM**: build a thin board on our own DB. Do not pay for HubSpot per city.
- **Payments**: Stripe. Sponsors pay by card or invoice; self-serve checkout for
  smaller slots.

---

## 4. Recommended architecture (what goes in this repo)

Monorepo, single deployable app, multi-tenant by `city_id` from day one.

- **Framework**: Next.js (App Router) + TypeScript. Public site, editor UI, sponsor
  portal and API all in one app. Per-city domains via middleware
  (`scottsdale.example.com` or custom domains).
- **Database**: Postgres (Supabase or Neon). Row-level `city_id` on every table.
  Core tables: `cities, subscribers, issues, issue_sections, sources, items (raw
  ingested), stories (clustered), events, businesses, sponsors, campaigns,
  ad_slots, prospects, touches, sends, send_events`.
- **Jobs/queues**: a worker process (BullMQ on Redis, or Inngest/Trigger.dev if we
  want managed) for ingest, drafting, sending, reports, outreach sequences.
- **Email sending**: Amazon SES via SESv2 API, SNS → webhook for bounces, complaints,
  deliveries, opens, clicks. One configuration set per city, shared dedicated IP
  pool once volume justifies it.
- **LLM**: Anthropic API. `claude-opus-5-5` for drafting and the editorial judgment
  calls (voice fidelity matters, this is the product); `claude-haiku-5-5` for
  high-volume classification, dedupe and event normalisation. Prompt caching on the
  per-city style guide and recent-issues context. Batch API for nightly triage
  (50% discount, no latency requirement).
- **Rendering**: MJML → HTML for issues; the same issue object renders the web
  archive page.
- **Hosting**: Vercel or Fly.io for the app, Fly/Railway for the worker, or a single
  Hetzner VPS running everything in Docker if we want the lowest run-rate.
- **Observability**: Sentry, plus our own send/engagement tables.

Repo layout (target):

```
apps/web            Next.js: public sites, editor, sponsor portal, API
apps/worker         queues: ingest, draft, send, report, outreach
packages/db         Prisma/Drizzle schema + migrations
packages/email      MJML templates, SES client, event webhook handlers
packages/ai         prompts, style-guide loader, drafting + triage functions
packages/ingest     source adapters (RSS, Places, Eventbrite, Reddit, ...)
cities/<slug>/      style guide, sources.yaml, branding, domain config
docs/               this plan, runbooks, media kit template
```

Adding a city should be: create `cities/<slug>/` with a style guide, a sources list
and branding, point a domain, seed the directory from Google Places, done.

---

## 5. Run-rate cost

### 5a. Platform fixed cost (shared across all cities)

| Item | Monthly |
|---|---|
| Hosting (app + worker + Postgres + Redis), VPS or small managed tier | $40 to $150 |
| Domains (one per city, amortised) | $1 to $2 per city |
| Sentry/monitoring, misc SaaS | $0 to $50 |
| SES dedicated IP (optional until ~100k sends/day) | $0 to $25 |
| **Total platform** | **$50 to $250** |

### 5b. Variable cost per city, by stage

Assumptions: 3 issues/week (13/month), one editor-hour per issue, drafting with Opus
5.5 at roughly 40k cached input + 3k output tokens per issue plus triage with Haiku
over ~500 items/night.

| | 2,000 subs (launch) | 15,000 subs (Scoop scale) | 50,000 subs (mature) |
|---|---|---|---|
| SES sending (13 issues × subs × $0.10/1k; use $0.16/1k if the reported 2026 "Essentials" plan applies) | $3 to $4 | $20 to $31 | $65 to $104 |
| Claude: drafting + triage + sponsor reports + event moderation | $15 to $30 | $20 to $40 | $30 to $60 |
| Data sources (Google Places API, Eventbrite, scraping proxies) | $10 to $40 | $20 to $60 | $40 to $100 |
| Stripe fees (2.9% + 30c of sponsor revenue) | ~$15 | ~$300 | ~$1,000 |
| **Software run-rate per city (ex Stripe)** | **~$30 to $75** | **~$60 to $130** | **~$135 to $265** |

The only cost that genuinely scales is sending, and at $0.10 per 1,000 it is
negligible. Compare: beehiiv at 15k subscribers is roughly $120 to $240/month per
newsletter and climbs with the list. Across 20 cities that is $30k to $60k a year
in SaaS fees versus a few hundred dollars on SES. This is the whole reason to
self-host.

### 5c. People cost (the real run-rate)

- **Editor time** per city: ~1 hour per issue plus ~2 hours/week on sponsor and
  community replies, so roughly 20 to 25 hours/month. Contractor at $25 to $40/hour
  gives $500 to $1,000/month per city. This is the dominant cost.
- **Sales**: automated outreach gets the meetings, but a human closes. Budget a
  commission-only or part-time seller per 5 to 8 cities, or the founder for the
  first 3 cities.
- **Growth spend**: Meta/Instagram ads to local zip codes typically acquire a
  newsletter subscriber for $1 to $3 in mid-size US metros. Launching to 5,000
  subscribers costs roughly $5k to $15k, or $0 if we lean on directory SEO and
  partner swaps and accept a 6 to 12 month ramp.

### 5d. Unit economics per city at 15k subscribers

- Inventory: 13 issues × 2 to 3 sponsor slots = 26 to 39 slots/month.
- Local newsletter pricing is almost always flat-rate, not CPM. Realistic:
  primary slot $300 to $800 per issue, secondary $150 to $400, at a 15k list with
  40%+ opens. (Translates to roughly $25 to $60 CPM on sends, consistent with the
  $30 to $100 range reported for 10k to 50k lists.)
- At 60% sell-through: ~$8k to $15k/month revenue.
- Costs: ~$1k editor, ~$150 software, ~$400 Stripe/sales commission. Contribution
  margin roughly 80%+.
- Add-ons that lift revenue without lifting cost: featured directory listings
  ($50 to $150/month per business, self-serve), featured events ($25 to $75),
  job board, sponsored Instagram posts, annual "best of" awards.

At 1,000 to 3,000 subscribers a city is roughly break-even on editor cost; above
~5,000 it is profitable; above ~10,000 it funds the next launch.

---

## 6. How to build and scale it professionally

**Phase 0, weeks 1 to 2: foundations.** Monorepo, Postgres schema with `city_id`,
auth, SES account warmed (sandbox exit, DKIM/SPF/DMARC, start with a shared IP),
Listmonk or MailCheap evaluated and either integrated or replaced, first ingest
adapters (RSS, Google Places, Eventbrite), signup form and double opt-in.

**Phase 1, weeks 3 to 6: city in a box, launch city #1.** Drafting pipeline with
style guide, editor approve/send UI, archive pages, events calendar with submission,
directory pages seeded for the city, basic analytics. Launch with the founder as
editor to learn the voice problem first-hand. Sell the first 3 sponsors by hand.

**Phase 2, weeks 7 to 12: monetisation and sales automation.** Sponsor portal, ad
calendar, Stripe, Monday reports, outbound prospecting and sequences, CRM board.
Media kit auto-generated from live stats. Referral program.

**Phase 3, months 4 to 6: launch cities #2 to #5.** Prove the `cities/<slug>/`
onboarding takes a day, not a month. Hire contract editors. Measure cost per
subscriber, time to first sponsor, sell-through. Kill or merge cities that miss
thresholds at 90 days.

**Phase 4, months 6 to 18: network.** 10 to 30 cities. Regional sponsors buying
across cities (this is where 6AM City makes its money). Shared national ad
inventory via newsletter ad networks for unsold slots. Possibly acquire existing
small local newsletters and migrate them onto the platform (lists for $1 to $3 per
engaged subscriber are common).

**Operating discipline that makes it scale:**

- Every city has the same KPIs dashboard: subs, 30-day open rate, growth rate, CAC,
  sell-through, revenue per issue, editor hours per issue.
- Editors never touch infrastructure. They see a brief, a draft, an approve button,
  and a sponsor inbox.
- Deliverability is a shared service: one person owns SES reputation, list hygiene
  (sunset unengaged after 90 days), complaint rates under 0.1%.
- Legal baseline: CAN-SPAM (physical address, one-click unsubscribe), GDPR-style
  consent records if ever outside the US, clear sponsored-content labelling,
  directory data sourced from public listings with a claim/removal flow.
- Editorial risk: Claude drafts, a human approves, every time. Never send unreviewed.
  Keep a per-city "do not cover" and "always verify" list in the style guide.

---

## 7. Identifying the commercially viable places

Score candidate metros on a simple model, run it over all US places with 50k to
500k population, and launch top-ranked first. The scraping and scoring is itself a
small module in this repo (`packages/market-scan`).

**Demand side (readers):**

- Population 60k to 400k in the core city, or a well-defined suburb cluster of a
  bigger metro with its own identity (Scottsdale is exactly this: 240k, affluent,
  distinct from Phoenix).
- Household income above the national median (sponsor budgets follow it).
- High share of 25 to 54 year-olds and recent in-migration (newcomers want local
  orientation; fast-growing Sunbelt suburbs are ideal).
- Weak or dying local newspaper (check for Gannett-owned, cut newsroom, paywall).
  A local-news vacuum is the single best predictor.
- Existing r/<city> subreddit with 20k+ members and an active local Facebook group
  signals appetite for local information.

**Supply side (sponsors):**

- Density of discretionary local businesses per capita: restaurants, bars, gyms,
  med-spas, dentists, realtors, home services, boutiques, event venues. Pull from
  Google Places; count businesses with 4.0+ rating and 100+ reviews within the
  city boundary. Those are the ones with marketing budgets.
- Real estate activity (realtors and mortgage brokers are the most reliable local
  advertisers).
- Tourism or seasonal population (events and hospitality inventory).
- Competition: no established local newsletter with over ~5k subscribers (search
  beehiiv/Substack directories, 6AM City's and Axios Local's city lists, "<city>
  newsletter" on Instagram). Avoid Axios Local and 6AM City markets early; target
  the gaps next to them.

**Operational:**

- Enough public event and news feed volume to fill 3 issues a week without
  original reporting.
- A credible local editor is hireable (university town, or a place with a freelance
  writer pool).
- English-dominant market for v1.

Scoring sketch: `score = 0.25*income_index + 0.20*growth_index + 0.20*business_density
+ 0.15*news_vacuum + 0.10*community_signal - 0.10*competition`. Shortlist the top 50,
then manually sanity check the top 10.

Likely first-wave profile: affluent, fast-growing suburbs and mid-size cities in AZ,
TX, FL, NC, TN, CO, UT, ID, GA with no 6AM City or Axios Local presence. Examples
of the archetype (to be verified by the scan, not assumed): Chandler/Gilbert AZ,
Frisco/McKinney TX, Boise ID, Fort Collins CO, Cary NC, Franklin TN, St. George UT,
Naples FL, Alpharetta GA.

---

## 8. Risks and honest caveats

- **Voice is the product.** Readers open a local newsletter because it feels like a
  person. If drafts read as generic, growth stalls. Budget real time on the style
  guides and keep humans in the loop.
- **Deliverability is unforgiving.** A bad list import or a spam spike on a shared
  SES account hurts every city. Warm properly, use double opt-in, sunset the dead.
- **Sales is the bottleneck, not software.** Automated outreach books meetings;
  closing local sponsors is relationship work. The platform makes one seller
  5x more productive, it does not remove them.
- **Pricing data is secondary-sourced.** SES per-1,000 rate (reported $0.10 à la
  carte vs. $0.16 on a new 2026 "Essentials" tier) and beehiiv tiers should be
  checked against the vendors' own pages before the model is finalised.
- **Scraping sources break.** Adapters need monitoring; treat them as disposable.

---

## 9. Immediate next steps

1. Pick the stack choices above (Next.js + Postgres + Listmonk-or-build mailer) or
   propose alternatives.
2. Name city #1. Ideally one of us already lives there or knows it; that shortcuts
   the voice problem.
3. Scaffold the monorepo in this repo: schema, SES send path, signup form, one
   ingest adapter, one drafted issue end to end.
4. Open an AWS account for SES, request production access, set up DKIM/DMARC on the
   first domain, start warming with a small list.
5. Build `packages/market-scan` and run it to produce the ranked city shortlist.
