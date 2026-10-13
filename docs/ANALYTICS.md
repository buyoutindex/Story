# Sponsor Analytics: Open-Source and Third-Party Options

Goal: every sponsor sees delivered, opens, unique clicks, CTR against the issue
average, and next available dates, without us building a reporting product. Two
stages, matching the platform plan (beehiiv in year one, own stack later).

## What sponsors actually need

| Metric | Source | Note |
|---|---|---|
| Delivered | ESP | Headline reach number |
| Opens / open rate | ESP | Inflated by Apple Mail Privacy Protection; show it, but sell on clicks |
| Unique clicks on their link(s) | Tracked link | The number that earns renewals |
| CTR vs issue average | Derived | Shows their creative performed, or did not |
| Landing-page visits and conversions | Their site or ours | Only if they install our pixel or use our landing page |
| Next available slots | Ad calendar | The renewal nudge |

## Stage 1 (beehiiv year): zero build

1. **beehiiv's own stats** give per-post and per-link clicks and an API to pull
   them. Enough for the Monday report.
2. **Dub.co** (open source, AGPL, also hosted free tier) for every sponsor link:
   short branded links on our domain (`go.thecommon.london/foxtons`), UTM tagging,
   per-link click analytics by day, device and referrer, and a **public shareable
   analytics page per link**. Create one link per campaign; the sponsor's "portal"
   is that shareable page plus the Monday email. Self-host later if volume grows.
3. **Monday report**: a small script (cron, Node) pulls the beehiiv post stats and
   Dub click counts, renders the one-page HTML report per sponsor, sends it via SES
   or Resend, and writes the numbers to our Postgres for the media kit. Half a day
   to build.
4. **Website analytics**: Plausible (AGPL, EU-hosted or self-hosted) or Umami
   (MIT). Both cookie-free (no consent banner) and both offer password-protected
   or public shared dashboards, so a sponsor with a featured directory listing can
   see its page views.

## Stage 2 (own stack): events into our database

1. **SES event publishing**: a configuration set per title publishes send,
   delivery, bounce, complaint, open and click events to SNS or Kinesis Firehose
   into Postgres (`send_events`). Click events carry the link URL, so per-sponsor
   clicks need no third party. Listmonk, if adopted as the sending UI, also
   records per-link clicks and campaign stats, but its sponsor-facing output is
   nil, so the events table is the source of truth either way.
2. **Dashboards without building them**: Metabase (AGPL) or Grafana (AGPL) on the
   same Postgres, one dashboard template parameterised by `sponsor_id`, shared via
   **signed embed URLs** so each sponsor sees only their rows. Metabase's public
   and signed embedding handles this out of the box.
3. **Keep Dub** for branded links and the public per-link page; it is the simplest
   artefact to paste into a renewal email.
4. **Sponsor portal** in our Next.js app becomes a thin page: campaign list,
   creatives, mockups, the embedded Metabase dashboard, next slots, "book again"
   button to Stripe. Most of it is embedding, not building.

## Things to avoid

- PostHog or Mixpanel for this: more than the job needs and the pricing scales
  with events.
- Pixel-based "conversion tracking" on sponsors' sites in year one: GDPR consent
  complexity for a number few local sponsors will read. Offer a landing page on
  our domain instead if they want it.
- Promising opens as the headline metric. Report clicks first; MPP makes opens
  unreliable and sponsors increasingly know it.

## Open-source inventory

| Tool | Licence | Role | Hosted option |
|---|---|---|---|
| Dub.co | AGPL-3.0 | Branded tracked links, public per-link analytics | Yes, free tier |
| Listmonk | AGPL-3.0 | Self-hosted campaigns and lists over SES; per-link click stats | No |
| Plausible | AGPL-3.0 | Cookie-free web analytics, shared dashboards | Yes, EU |
| Umami | MIT | Same, lighter | Yes |
| Metabase | AGPL-3.0 (OSS edition) | Sponsor dashboards, signed embeds | Yes |
| Grafana | AGPL-3.0 | Alternative dashboards | Yes |
| Keila | AGPL-3.0 | Newsletter tool with SES support; alternative to Listmonk | Yes, EU |

Licence note: AGPL only bites if we modify and redistribute the software as a
service to others; using it internally for our own reporting is fine.
