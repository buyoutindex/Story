# Local Newsletter Network (Europe): Business and Build Plan

Status: proposal v2 (October 2026), Europe-first. Supersedes the US-oriented v1.
This repo is being repurposed into the platform described here.

---

## 1. The thesis

A local newsletter is a small media business with three assets: an owned audience
(email list), a local-business graph (directory + events), and a sponsor pipeline.
The Scottsdale Scoop example shows one person running 15k subscribers and three
issues a week with six systems. Every one of those systems is software plus a thin
layer of human judgment. The judgment layer (voice, editorial approval, sponsor
relationships) does not scale per city. The software layer scales to N cities at
near-zero marginal cost. So: build the platform once, launch cities cheaply, let the
platform carry the operating load.

**Why Europe, and why now.** Three things make Europe a better market than the US
for this model, and one thing makes it harder.

- *Local press is consolidating, not yet dead.* In Germany almost half of all
  Landkreise (46.75%) now have only one independent local daily, down from an
  average of 2.26 titles per district in 1992 to 1.83 in 2023 (Wüstenradar study,
  Hamburg Media School, Nov 2024). A BDZV analysis projected physical delivery gaps
  in roughly 4,400 of 11,000 municipalities by 2025. That is exactly the "weak
  incumbent, not a vacuum" condition in which a light, free, daily-ish email wins:
  readers still want local news, the incumbent is paywalled, thin and slow.
- *Europeans pay for local journalism.* Mill Media (UK) runs six city titles with
  150k+ free subscribers, 10k+ paying members at roughly £8 to £9 a month, 20
  journalists, profitable, and over 90% of revenue from subscriptions. London
  Centric hit paying scale within months. That opens a second revenue line
  (membership) that barely exists for US local newsletters.
- *Almost no independent competition.* Nearly every German/Austrian/Swiss regional
  paper has a morning newsletter, but it is a traffic funnel for the paper, not a
  standalone product, and independents are rare and tiny (e.g. the LVZ's
  neighbourhood letter "Heiterblick" at ~1,250 subscribers). The US has 6AM City,
  Axios Local and hundreds of Scoop-style independents; DACH has almost none.
- *The hard part:* law. GDPR, double opt-in, Impressum and press-law duties,
  advertising labelling, VAT, e-invoicing, and the EU AI Act's transparency rules
  (in force for our use case from 2 August 2026). None of it is a blocker; all of it
  has to be built into the platform rather than bolted on per city.

**Recommended geography.** Launch region: DACH (Germany, Austria, German-speaking
Switzerland). One language covers ~100M people, ~80 German cities above 100k and
~600 Mittelstädte at 20k to 100k, the richest ad market in Europe, and the
consolidation data above. Second wave: UK (gaps around Mill Media), Netherlands,
Nordics. Avoid France/Italy/Spain in year one (different ad-market structures,
stronger regional-press lock-in, and more languages for the same build effort).

---

## 2. How it works: the operating loop for one city

Per issue (3x per week, 06:00 local, German cities tend to read early):

1. **Ingest** (automated, nightly). Sources in Europe are better structured than in
   the US, which helps automation:
   - Rathaus/Gemeinde press releases and Amtsblatt (RSS or scraped).
   - Ratsinformationssystem (council agendas, minutes, decisions). Most German
     councils run ALLRIS, SessionNet or Mandatos; these have a handful of standard
     formats, so a few adapters cover hundreds of cities. Austria: Gemeinde sites +
     RIS; Switzerland: Gemeindeversammlung agendas, cantonal Amtsblätter,
     Abstimmungsvorlagen.
   - Polizeipresseportal (presseportal.de/blaulicht): structured, per-district,
     free. The single best daily feed in Germany.
   - Handelsregister / Firmenbuch / Zefix for new business registrations (new
     openings), Bauanträge/Bebauungspläne, Immobilien portals for new builds.
   - Event sources: venue sites, Vereine, Volkshochschule, theatres, Stadtfest
     calendars, Eventfrog/Eventbrite (CH/DE), local Facebook groups (public).
   - Regional paper RSS (headlines only, link out; never copy).
   - Reddit (r/<city>, growing in DE), Instagram location tags, Nebenan.de public
     posts, Jodel for university towns.
   - OSM + Google Places for the directory seed.
2. **Triage** (Claude Haiku, automated): dedupe, cluster, score for local interest,
   flag for human attention (deaths, crime with named persons, legal risk,
   anything about minors, anything that could trigger a Gegendarstellung).
3. **Draft** (Claude Opus, automated): a full issue in the city's voice and
   language from a structured brief. Sections: Aufmacher, 5 Kurzmeldungen,
   Termine der Woche, Neu in der Stadt (openings), sponsor slot(s), Leserfrage.
   Austrian and Swiss cities get their own register (Swiss Standard German, no ß,
   Austrian vocabulary) via the style guide, not a separate model.
4. **Edit and approve** (human, 15 to 30 minutes). The only mandatory human step.
   This also satisfies the editorial-responsibility test under the AI Act (below).
5. **Send** (automated) via SES in eu-central-1.
6. **Measure** (automated): opens, clicks, sponsor CTR, bounces, complaints,
   unsubscribes, feeding back into triage weights and sponsor reports.

Per week: Monday sponsor reports; outbound sequences to prospect businesses;
membership nudges to the most engaged 10% of free readers.

Continuously: organiser-submitted events, directory pages compounding in Google,
Instagram cross-posts of the lead story, referral programme, swaps with
neighbouring cities.

---

## 3. The six systems as software modules

| # | System | What we build | Europe-specific additions | Effort |
|---|---|---|---|---|
| 1 | Writing desk | Ingest pipeline, Claude triage and drafting, editor UI with approve/send | Council-system adapters (ALLRIS, SessionNet), Presseportal adapter, multi-language style guides, "always verify / never cover" lists per city | 4 to 5 weeks |
| 2 | Events calendar | Public calendar, organiser submission, moderation queue, ICS/embed | Impressum-compliant organiser data, GDPR consent on submission | 1 week |
| 3 | Business directory | Programmatic SEO pages per business, claim-your-listing, categories, maps | Seed from OSM + Handelsregister; removal/correction flow (GDPR Art. 16/17 for sole traders whose name is the business name) | 1 to 2 weeks |
| 4 | Ad sales + membership | Sponsor portal (creatives, mockups, stats), ad calendar, Stripe checkout, Monday reports; paid membership tier | SEPA Direct Debit, invoices with VAT (DE 19%, AT 20%, CH 8.1%), reverse charge for EU B2B, XRechnung/ZUGFeRD e-invoices (mandatory outbound B2B in DE from 2027/2028), "Anzeige" labelling baked into templates; optional Steady integration for memberships | 3 weeks |
| 5 | Mailer | SES sending, lists, signup forms/API, bounce/complaint handling, archive, analytics | Double opt-in with timestamped consent log (UWG §7 DE, TKG §174 AT, UWG Art. 3 CH), one-click unsubscribe, Certified Senders Alliance onboarding for GMX/WEB.DE/T-Online deliverability, EU-only data residency | 2 weeks (or Listmonk as base) |
| 6 | Outbound HQ | Prospect scraping, sequenced email outreach, pipeline board, reply detection | B2B cold email is lawful under UWG §7(2) only with presumed interest; sequences must be short, relevant, with opt-out; prefer phone + LinkedIn + warm intros for DACH | 2 weeks |

Roughly 12 to 16 weeks for a solid v1 of all six, about 7 weeks for a minimum
"city in a box" (modules 1, 5, a manual 4, and the legal baseline).

### Build vs. adopt

- **Mailer base**: Listmonk (Go, mature, multi-list, SES-compatible, self-hosted,
  strong admin) is the lowest-risk base for multi-city and is already common in
  German self-hosting circles. MailCheap (MIT, Next.js, SES, single-admin, ~130
  stars) is a lighter reference but small enough that we would be maintaining a
  fork. Either way SES does the sending; we own consent logging and the archive.
- **EU-sovereign alternative to SES** if a client, lawyer or Datenschutzbeauftragter
  insists: Scaleway TEM (FR), Brevo (FR), Mailjet (Sinch, FR). SES in eu-central-1
  with the AWS DPA, SCCs and a documented Transfer Impact Assessment is standard
  practice and acceptable for a newsletter (email address + engagement data, no
  special categories). The CLOUD Act critique is real but widely accepted for this
  data class. Decide once, document it in the Verfahrensverzeichnis.
- **Memberships**: Stripe Billing handles SEPA and card. Steady (Berlin) is the
  known brand for journalism memberships in DACH and lowers trust friction for
  readers; it costs 10% + payment fees. Start with Stripe, offer Steady if
  conversion lags.
- **Drafting, directory, events, CRM**: build.

---

## 4. Architecture (what goes in this repo)

Monorepo, single deployable app, multi-tenant by `city_id` from day one, EU
data residency throughout.

- **Framework**: Next.js (App Router) + TypeScript. Public site, editor UI, sponsor
  portal, member area and API in one app. Per-city domains via middleware
  (`<stadt>.example.de` or `.at`/`.ch` custom domains).
- **Database**: Postgres in Frankfurt (Supabase eu-central-1, Neon Frankfurt, or
  Hetzner-hosted). `city_id` on every table. Core tables: `cities, subscribers,
  consents, issues, issue_sections, sources, items, stories, events, businesses,
  sponsors, campaigns, ad_slots, invoices, members, prospects, touches, sends,
  send_events, audit_log`.
- **Jobs**: worker process (BullMQ on Redis) for ingest, triage, drafting, sending,
  reports, outreach. Managed alternative: Inngest/Trigger.dev (check EU hosting).
- **Email**: Amazon SES v2 in eu-central-1. SNS → webhook for bounce, complaint,
  delivery, open, click. One configuration set per city; shared dedicated IP pool
  once volume justifies (~100k sends/day). DKIM, SPF, DMARC `p=quarantine` at
  minimum. CSA certification application once reputation is clean.
- **LLM**: Anthropic API. `claude-opus-5-5` for drafting and editorial judgment
  (voice and local nuance are the product); `claude-haiku-5-5` for high-volume
  classification, dedupe and event normalisation. Prompt caching on the per-city
  style guide and recent issues; Batch API for the nightly triage run (50% off).
  For EU processing, pin inference with the `inference_geo` request parameter or
  route via Bedrock eu-central-1 / Vertex europe-west; sign the Anthropic DPA. The
  LLM sees news items and drafts, not subscriber data, so this is low-risk either
  way, but document it.
- **Rendering**: MJML → HTML for issues; the same issue object renders the web
  archive page (with Impressum, Datenschutz, and "Anzeige" labels in the template).
- **Hosting**: Railway, EU West region, the same platform the transaction-monitor
  runs on: app service and worker service from one repo with continuous deploy from
  `main`, Railway Postgres and Redis plugins (or a Postgres-backed queue to keep one
  datastore), cron services for nightly ingest, Monday reports and list hygiene,
  Metabase and Dub.co from Railway templates. Secrets live in Railway, as the TM
  does. AWS is used for SES (eu-west-2 London) and R2-style backups only. UK GDPR
  treats the EU as adequate, so EU-region hosting is fine for a UK list.
- **Legal plumbing as code**: consent records with IP, timestamp, form version and
  text; export and deletion endpoints (Art. 15/17); cookie-free analytics on the
  web side (Plausible EU or self-hosted) to avoid consent banners; Impressum and
  Datenschutzerklärung generated from the `cities/<slug>/legal.yaml`.

Repo layout (target):

```
apps/web            Next.js: public sites, editor, sponsor portal, member area, API
apps/worker         queues: ingest, triage, draft, send, report, outreach
packages/db         Drizzle schema + migrations (Postgres, EU)
packages/email      MJML templates, SES client, SNS webhook handlers, consent log
packages/ai         prompts, style-guide loader, drafting + triage, locale registers
packages/ingest     adapters: ALLRIS, SessionNet, Presseportal, RSS, OSM, Handelsregister, ...
packages/legal      Impressum/Datenschutz generators, invoice/VAT rules, AI Act notice
packages/market-scan  city scoring model (section 7)
cities/<slug>/      style guide, sources.yaml, branding, legal.yaml, domain config
docs/               this plan, runbooks, media kit template
```

Adding a city should be: create `cities/<slug>/`, fill in style guide, sources,
legal entity and branding, point a domain, seed the directory, send a test issue.
Target: one day.

---

## 5. Run-rate cost (EUR)

### 5a. Platform fixed cost (shared across all cities)

| Item | Monthly |
|---|---|
| Hosting (Railway: app + worker + Postgres + Redis, EU West) | €20 to €60 |
| Domains (.de/.at/.ch per city, amortised) | €1 to €2 per city |
| Monitoring, misc SaaS | €0 to €50 |
| SES dedicated IP (optional until ~100k sends/day) | €0 to €25 |
| Legal: Datenschutzerklärung/Impressum review, AV-Verträge (one-off, amortised) | €50 to €100 |
| **Total platform** | **€100 to €300** |

### 5b. Variable software cost per city, by stage

Assumptions: 13 issues/month; Opus 5.5 drafting at ~40k cached input + 3k output
tokens per issue; Haiku triage over ~500 items/night; SES at $0.10 per 1,000
(use $0.16 if the reported 2026 "Essentials" tier applies to a new account).

| | 2,000 subs (launch) | 15,000 subs | 50,000 subs |
|---|---|---|---|
| SES sending | €3 to €4 | €18 to €30 | €60 to €100 |
| Claude (drafting, triage, reports, moderation) | €15 to €30 | €20 to €40 | €30 to €60 |
| Data sources (Places API, proxies; most DACH public sources are free) | €5 to €25 | €10 to €40 | €20 to €60 |
| **Software run-rate per city** | **€25 to €60** | **€50 to €110** | **€110 to €220** |
| Stripe/SEPA fees (~1.5% + €0.25 SEPA, 1.5% + €0.25 EU cards) | ~€10 | ~€150 to €250 | ~€600 |

Compare beehiiv at 15k subscribers: roughly €110 to €220 per month per newsletter,
rising with the list; across 20 cities €25k to €50k a year versus a few hundred
euros on SES. Self-hosting is the structural advantage.

### 5c. People cost (the real run-rate)

- **Editor** per city: ~1 hour per issue + ~2 hours/week on replies and sponsors,
  so 20 to 25 hours/month. Freelance local journalists in DACH bill €35 to €60/hour;
  a Werkstudent or part-time editor costs less. Budget €800 to €1,400/month per
  city. In the UK, £600 to £1,000. This is the dominant cost everywhere.
- **Sales**: automation books meetings; a human closes. DACH Mittelstand buys on
  trust and phone, less on cold email. One part-time seller per 4 to 6 cities, or
  the founder for the first three.
- **Growth spend**: Meta ads to local postcodes acquire a newsletter subscriber for
  roughly €1.50 to €4 in German Mittelstädte (less competition than US metros, but
  smaller addressable pools). Launching to 5,000 subscribers costs €7k to €20k
  paid, or close to €0 over 6 to 12 months via directory SEO, Instagram, local
  Facebook groups and Nebenan.de.

### 5d. Unit economics per city at 15,000 subscribers (German Mittelstadt)

- **Ads**: 13 issues × 2 to 3 slots = 26 to 39 slots/month. DACH local pricing is
  flat-rate: primary slot €250 to €600 per issue, secondary €120 to €300 at a 15k
  list with 40%+ opens (that is roughly €20 to €45 CPM on sends, in line with the
  low end of reported newsletter CPM ranges, which is right for local). At 60%
  sell-through: €5k to €11k/month.
- **Membership**: Mill Media converts roughly 6 to 7% of its free list to paying.
  A conservative 3% of 15k at €6/month = €2.7k/month; 6% = €5.4k. Offer: ad-light
  edition, Friday long read, member events, "Frag die Redaktion".
- **Add-ons**: featured directory listings (€40 to €120/month, self-serve),
  featured events (€20 to €60), Stellenanzeigen (local job ads are a strong DACH
  product; €99 to €249 per posting), annual "Beste der Stadt" awards with sponsor.
- **Revenue**: €8k to €17k/month. **Costs**: ~€1.2k editor, ~€100 software, ~€400
  payment fees and commission. Contribution margin above 80%.

Break-even on editor cost sits around 2,500 to 4,000 subscribers (one or two
sponsors plus a few hundred members). Above ~6,000 a city is clearly profitable;
above ~10,000 it funds the next launch.

### 5e. Legal and compliance cost (often forgotten)

- Company and press-law setup: a GmbH/UG (DE), GmbH (AT/CH) or UK Ltd with a named
  "Verantwortlicher im Sinne des Presserechts" (V.i.S.d.P.) per title, Impressum
  per domain, Datenschutzerklärung, Verzeichnis von Verarbeitungstätigkeiten,
  AV-Verträge with AWS, Anthropic, hosting, Stripe. One-off €2k to €5k with a
  media lawyer, then marginal per city.
- Datenschutzbeauftragter is only mandatory in Germany above 20 people regularly
  processing personal data; not needed early.
- Press liability insurance (Medienhaftpflicht): roughly €800 to €2,000/year for
  the group. Worth having before the first Abmahnung.
- EU AI Act, Art. 50(4): AI-generated text published to inform the public on
  matters of public interest must be disclosed, *unless* it has undergone human
  review and a natural or legal person holds editorial responsibility. Our
  approve-before-send step and the V.i.S.d.P. meet that exemption. Keep the
  approval audit log, and add a short transparency note in the Impressum anyway.
  Image generation (if we ever auto-generate visuals) must be labelled.
- Advertising: every paid placement labelled "Anzeige" (DE/AT) or "Werbung"/
  "Publireportage" (CH); Schleichwerbung is actionable under UWG and the
  Pressekodex. Health advertising (Heilmittelwerbegesetz) and alcohol/gambling
  carry extra restrictions; build a category blocklist into the ad-slot model.

---

## 6. How to build and scale it professionally

**Phase 0, weeks 1 to 3: foundations + legal baseline.** Monorepo, Postgres (EU),
auth, SES eu-central-1 out of sandbox, DKIM/SPF/DMARC, Listmonk evaluated or
replaced, consent-logging double opt-in, Impressum/Datenschutz generators, first
adapters (Presseportal, one council system, RSS, OSM), entity + V.i.S.d.P. + DPA set
signed.

**Phase 1, weeks 4 to 8: city in a box, launch city #1.** Drafting with style
guide, editor approve/send UI, archive, events with submission, directory seeded,
basic analytics. The founder is editor for city #1 to learn the voice problem
first-hand. Sell the first three sponsors by phone and in person.

**Phase 2, weeks 9 to 14: monetisation and sales automation.** Sponsor portal,
ad calendar, Stripe (SEPA + card + invoices with VAT), Monday reports, membership
tier, outbound prospecting and pipeline, auto-generated media kit (Mediadaten PDF,
the format DACH agencies expect), referral programme.

**Phase 3, months 4 to 7: cities #2 to #5 in the same region.** Prove one-day
onboarding. Hire freelance editors. Track cost per subscriber, time to first
sponsor, sell-through, member conversion. Kill or merge cities that miss 90-day
thresholds. Cluster geographically (e.g. five towns around Munich) so one seller
and one brand family can work them.

**Phase 4, months 7 to 18: network.** 10 to 30 cities across DACH. Regional
sponsors (Sparkassen/Volksbanken, Stadtwerke, car dealer groups, regional
retailers, universities, Klinikverbünde) buying across clusters. National unsold
inventory via EU newsletter ad networks. Acquire or absorb existing small
independent letters. Then the second language: UK cities Mill Media has not taken
(Leeds, Bristol, Edinburgh, Nottingham, Cardiff, Newcastle), then NL and the
Nordics using the same codebase with a new locale register.

**Operating discipline:**

- One KPI dashboard per city: subs, 30-day open rate, growth, CAC, sell-through,
  revenue per issue, member conversion, editor hours per issue.
- Editors never touch infrastructure; they see a brief, a draft, an approve button,
  a sponsor inbox and a member inbox.
- Deliverability is a shared service: SES reputation, CSA status, list hygiene
  (sunset after 90 days unengaged), complaints under 0.1%. German inboxes are
  dominated by GMX/WEB.DE/T-Online, which are stricter than Gmail; CSA
  certification matters more here than in the US.
- Editorial risk: Claude drafts, a human approves, every time. Per-city "do not
  cover" and "always verify" lists. Named-person crime stories follow Pressekodex
  rules (no names, no identifying detail unless public figure).
- Legal is code: consent, Impressum, labelling, invoicing and deletion are
  platform features, not per-city chores.

---

## 7. Identifying the commercially viable places (Europe)

Build `packages/market-scan` to score every DACH municipality of 25k to 400k
inhabitants, then UK/NL/Nordic towns with the same model and local data sources.

**Data sources (all public):** Zensus 2022 and Regionalstatistik (DE), Statistik
Austria, BFS (CH), ONS (UK), CBS (NL); GfK/NIQ Kaufkraftindex by Kreis and
Gemeinde; IVW circulation and Wüstenradar one-paper districts; Handelsregister /
Firmenbuch / Zefix business density; OSM POI counts; Destatis Wanderungssaldo
(net migration) and Baugenehmigungen; subreddit and Facebook-group sizes;
Google Trends for "<Stadt> news/veranstaltungen".

**Demand side (readers):**

- Population 40k to 300k, or a well-defined affluent commuter belt around a
  Metropole with its own identity (the German Speckgürtel: Starnberg, Erding,
  Freising, Dachau, Fürstenfeldbruck around Munich; Bad Homburg, Oberursel,
  Hofheim around Frankfurt; Norderstedt, Ahrensburg, Pinneberg around Hamburg;
  Potsdam, Falkensee around Berlin; Mödling, Klosterneuburg, Baden around Vienna;
  Zug, Baden, Winterthur, Thun around Zurich/Bern).
- Kaufkraft above the national index (ad budgets follow it; Swiss and Bavarian/
  Baden-Württemberg towns lead).
- Positive net migration, new-build activity (newcomers need orientation).
- One-newspaper district (Wüstenradar) or a paper that has cut its local desk,
  closed its Lokalredaktion or moved to a regional edition.
- Community signal: active r/<Stadt>, large local Facebook groups, Nebenan.de
  density, a Stadtfest/Weinfest culture with real event volume.
- University towns (Erlangen, Tübingen, Konstanz, Regensburg, Jena, Innsbruck,
  St. Gallen) for young readers and editor supply, but lower Kaufkraft.

**Supply side (sponsors):**

- Density of discretionary local businesses with marketing budgets: Gastronomie,
  Fitness, Autohäuser, Immobilienmakler, Handwerk (Küchen, Bäder, Solar), Optiker,
  Zahnärzte (restricted messaging), Hotels, Weingüter, Einzelhandel in the
  Innenstadt, Stadtwerke, Sparkasse/Volksbank/Raiffeisen, local Krankenhaus.
  Count businesses with 4.0+ rating and 50+ Google reviews inside the boundary.
- Real estate activity (Makler and Bauträger are the most dependable local
  advertisers in DACH too).
- Tourism or second-home population (Bodensee, Alpenvorland, Nordsee/Ostsee,
  Salzkammergut, Engadin).
- Competition: no independent local newsletter above ~3k subscribers; the
  regional paper's newsletter is a brand funnel, not a competitor. Check Substack,
  Steady, beehiiv directories and "<Stadt> Newsletter" on Instagram.

**Operational:**

- Enough feed volume (Presseportal, council system, events) for three issues a
  week without original reporting.
- A credible local editor is hireable (Journalistik programmes, Volontariat
  alumni, retired Lokalredakteure after the latest paper merger).
- Mobile-first, German-language; dialect only in headlines and tone.

Scoring sketch:

```
score = 0.25*kaufkraft_index
      + 0.20*growth_index           (net migration + Baugenehmigungen)
      + 0.20*business_density       (discretionary POIs per 1k inhabitants)
      + 0.15*news_vacuum            (one-paper district, Lokalredaktion cuts)
      + 0.10*community_signal       (subreddit, FB groups, Nebenan)
      - 0.10*competition
```

Shortlist the top 50, sanity-check the top 10 by hand (visit, read the local
paper for a week, call five businesses and ask what they spend on ads).

**Likely first-wave archetype (to be verified by the scan, not assumed):**
affluent, growing towns of 40k to 150k in the Munich, Frankfurt, Stuttgart,
Hamburg and Zurich/Vienna commuter belts, plus standalone Mittelstädte with a
single weakened paper: e.g. Starnberg, Erding, Freising, Rosenheim, Konstanz,
Ravensburg, Reutlingen, Esslingen, Bad Homburg, Wiesbaden, Darmstadt, Lüneburg,
Norderstedt, Potsdam, Erlangen, Regensburg, Ulm, Freiburg; Austria: Mödling,
Baden, Wels, Dornbirn, Salzburg; Switzerland: Zug, Baden, Winterthur, Thun,
Luzern.

---

## 7b. Scope extension: vertical trade journals (UK and Ireland)

The same platform (ingest, Claude triage and drafting, human approval, SES send,
sponsor system, membership) runs a second product line: niche B2B newsletters for
specific UK and Irish verticals. Different sources, different readers, far higher
revenue per subscriber. Not a year-one build; scoped now so the platform's data
model does not block it.

**Why it fits.** Trade readers pay (£10 to £40 a month is normal for a weekly
deal-flow or regulatory digest), B2B sponsors pay £50 to £120 CPM against £20 to
£45 for local lifestyle, and the ingest is more structured than any neighbourhood
feed. The editorial loop is identical: brief, draft, approve, send.

**The data advantage: the transaction-monitor product (separate repo).** It
already holds, in Postgres:

- `universe.companies`: the full Companies House Basic Company Data snapshot
  (every UK company: status, incorporation and dissolution dates, SIC codes,
  postcode and generated postcode area, charges counts, accounts category,
  previous names), refreshed monthly from R2. Incorporations and dissolutions by
  sector and region fall straight out of it.
- `events_out`: an append-only log of classified transactions (new, label changed,
  state changed, published, removed) with a snapshot payload (company, status,
  SIC codes, implied label, headline, parties, dates, publish state). Consumers
  read forward from an offset in `events_out_consumers`.
- A 15-bucket sector taxonomy (Agriculture & Resources through Wholesale & Retail)
  mapped from SIC codes in the database, and an existing Monday 08:00 "Buyout
  Index" weekly digest per followed bucket.

**Integration contract (read-only, no changes to that repo from here).**

1. The trade-journal pipeline is a new consumer of `events_out` with its own
   offset row (for example `story-trade-journal`), reading published transaction
   events for the buckets a title covers. At-least-once semantics as that table's
   documentation specifies; we dedupe on `transaction_key`.
2. Incorporation and dissolution counts and lists per sector, SIC code and
   postcode area come from `universe.companies` via a read-only role or an export
   endpoint. The monthly snapshot cadence means "new incorporations this month" is
   the natural unit; weekly figures need the daily CH stream, which is that repo's
   decision, not ours.
3. Access is through an API or read replica owned by the transaction-monitor
   repo. Any new grant, endpoint or consumer registration is written up as a task
   in that repo's own queue (its CLAUDE.md forbids changes from outside its
   process). This repo holds only the client and the schema it expects.
4. The transaction-monitor's own digest is the data product; a trade journal is
   the editorial layer on top (context, named parties, who advised, what it means
   for the sector). Keep them distinct so the journal does not cannibalise the
   paid data product and vice versa; cross-link each from the other.

**Ireland.** CRO (Companies Registration Office) publishes open company data and
a CORE API with incorporations, dissolutions, charges and filings; Iris Oifigiúil
carries strike-off and liquidation notices. These are the Irish equivalents of
the universe snapshot and would be a new ingest adapter in this repo, not a
transaction-monitor change, unless that product chooses to add Ireland.

**Niche selection method.** Score the 15 buckets, then sub-sectors within them,
on: consolidation activity (events per month in `events_out`), B2B sponsor depth
(lenders, brokers, lawyers, accountants, software vendors who sell to that
sector), fragmented or dying trade press, professional readers who expense
subscriptions, and regulatory churn. Likely first candidates are
practice-consolidation verticals where deal flow is steady and advisers pay for
reach: dental, veterinary, pharmacy, GP and care homes under Health & Care;
accountancy, IFA and wealth, insurance brokers and law firms under Professional
Services; MSPs under Software & IT; nurseries and children's services; estate
agencies under Real Estate. The data decides, not the hunch.

**Product shape per title.** Free weekly digest (deals, incorporations,
dissolutions, appointments, regulatory items for the vertical) with two B2B sponsor
slots; a Pro tier at £15 to £40 a month with the full list, filters, early access
and an archive; sponsored reports and league tables (most active acquirers,
advisers) as annual products. Cadence weekly, Tuesday 07:00, moving to twice weekly
when deal volume supports it.

**What this adds to the platform scope now.**

- Tenant model: a title is a `title` with a `kind` (local | trade) rather than a
  `city`; sources, taxonomy and sections are per title.
- Ingest: a generic "events consumer" adapter (offset-based, idempotent) and a
  "snapshot query" adapter, both pointed at the transaction-monitor's interfaces.
- Drafting: a trade register in the style-guide loader (named parties, adviser
  credits, no speculation beyond the filed evidence, the "nothing here is advice"
  line the data product already uses).
- Monetisation: CPM pricing alongside flat-rate, and a Pro tier with gated data
  views, which the local titles do not need.
- Legal: FCA financial-promotion rules if any title touches investment content;
  keep to factual register reporting and label sponsor content.

## 8. Risks and honest caveats

- **Voice is the product.** Generic drafts stall growth. Invest in style guides and
  keep humans in the loop; Austrian and Swiss readers notice German-German instantly.
- **Deliverability is unforgiving**, and German inbox providers are stricter than
  Gmail. Double opt-in, warm-up, list hygiene and CSA are not optional.
- **Sales is the bottleneck, not software.** DACH Mittelstand buys on relationship;
  automation makes one seller much more productive but does not replace them.
- **Legal exposure is real but routine.** Abmahnungen over Impressum, labelling or
  photo rights are a cottage industry in Germany. Template everything, insure, and
  never use images without a licence (use own photos, city press photos with
  credit, or licensed stock).
- **Data points are secondary-sourced.** SES pricing ($0.10 à la carte vs. a
  reported $0.16 2026 "Essentials" tier), beehiiv tiers, Mill Media figures, and
  Wüstenradar/BDZV numbers should be checked against primary sources before the
  model is finalised.
- **Sovereignty pressure.** Some sponsors (Sparkassen, Stadtwerke, public bodies)
  may ask about US cloud providers. Keep the EU-sovereign mailer and EU inference
  options documented so the answer is a config change, not a rebuild.

---

## 9. Immediate next steps

1. Confirm DACH-first and the stack (Next.js + Postgres on Hetzner/Supabase EU +
   Listmonk-or-build mailer + SES eu-central-1), or propose alternatives.
2. Name city #1. Ideally one of us lives there or knows it; that shortcuts the
   voice problem and the first three sponsor conversations.
3. Scaffold the monorepo here: schema, consent-logging signup, SES send path,
   Presseportal + one council adapter, one drafted issue end to end.
4. Set up the AWS account in eu-central-1, request production SES access, DKIM/
   DMARC on the first domain, start warming with a seed list. Sign the AWS and
   Anthropic DPAs.
5. Build `packages/market-scan` against Zensus/GfK/Wüstenradar/OSM and produce the
   ranked DACH shortlist.
6. Engage a media lawyer for the Impressum/Datenschutz/AV templates and the
   V.i.S.d.P. structure (one-off).
