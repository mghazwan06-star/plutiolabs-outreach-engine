# Dual-Source Design: MCS + Google Maps

**Status when this spec was written: not yet built.** Everything below is measured against the live MCS dataset before a single line of the merge was coded. The point was to know the shape of the win before spending build time on it.

## The verdict

MCS is not a scrape. It's one HTTP GET returning a 4.2 MB JSON array of 5,620 UK certified installers. It adds three nodes to the pipeline. It doesn't add a second scraper, a second Apify actor, or a second dedupe path.

Run MCS-only and 100% of leads route to the generic fallback template: no review data, no complaint, no competitor to name. Run Maps-only and you get the existing 20 to 30% no-email hole. Neither source alone can fill the CRM.

## Measured facts, not estimates

Pulled live from the MCS public API on the day this spec was written.

| Field | Coverage |
|---|---|
| name, address, postcode, town, county, region, lat, lon | 100% |
| email | **100%** |
| telephone | 99.1% |
| technologies (12 types) | 100% |
| certification body + number | 100% |
| website | 53.8% |
| Google rating + review count | 46.9% |

Segments: solar PV 4,262, battery 2,592, air-source heat pump 2,229, ground-source 668, solar thermal 188, biomass 70.

**82.9%** of records carry a name ending Ltd/Limited/LLP/plc, a corporate subscriber under UK PECR rules, which means cold email is legally permitted without prior consent (see [`../engineering/launch-readiness.md`](../engineering/launch-readiness.md) for the compliance filter this drives). **40.7%** of the emails are generic prefixes (`info@`, `sales@`) with no first name to extract. That's exactly the population the contact-enrichment step is scoped to fix.

A subtler signal, worth checking because it's the kind of thing that separates a dataset you can trust from one you can't: only **76.8%** of records have a matching email domain and website domain. Not 100%. That's the tell that the emails were independently sourced rather than synthesised as `info@{domain}`. A 100% match rate would have been the red flag, not this one.

## The single biggest lever: competitor matching

The original competitor-match logic grouped leads by lowercased city string and picked the highest-review same-trade peer. Two ways this breaks:

1. MCS town values ("Tonbridge and Malling") never match Google Maps city values ("London"), so merged MCS leads got no competitor at all.
2. An unbanded max-review match returns a median gap of 224 to 554 reviews. That's not "your local competitor." That's naming a national chain as the rival to a two-van outfit, and it reads as demoralising rather than motivating to the person reading the email.

Measured against the 1,313 leads that qualified as full-pipeline candidates:

| Rule | Leads with usable gap ≥10 | Median gap |
|---|---|---|
| Nearest highest-review, no band, 10mi | 85.0% | 224 |
| Nearest highest-review, no band, 25mi | 96.0% | 554 |
| **Same-technology, 10mi, gap banded 10-150** | **75.5%** | **87** |
| Same-technology, 15mi, gap banded 10-150 | 84.3% | 106 |

The banded rule was chosen over the higher-coverage unbanded ones on purpose. A gap of 60 to 110 from a genuine same-technology rival six miles away is believable and checkable. A gap of 300+ from an unbanded max is neither. It costs credibility the moment the recipient looks the company up, which they will, because naming a specific competitor is the whole point of the line. Coverage dropped from 96% to 75.5% on purpose, trading reach for every remaining lead's claim being one a sceptical tradesperson can verify and believe.

The MCS dataset supplies latitude, longitude and technology tags for every record, which turns competitor matching into a free geospatial join over data already in hand rather than a second scrape.

## The second biggest lever: review sort order

`topComplaint` drives the two highest-converting email templates, and complaints live in one-to-three-star reviews. The Apify Google Maps actor defaults to "most relevant" review sort, which skews heavily five-star. Feed Claude five five-star reviews and `topComplaint` comes back empty, and the lead falls to a weaker, complaint-free template through no fault of its actual review history. It's purely an artefact of scrape order.

The fix: pull the lowest-rated reviews and the highest-rated reviews together. The low end surfaces the complaint, the high end preserves the social-proof quote a different template needs. Costs nothing extra per lead. This one line item is the difference between the top-tier template and the fallback on a large share of the list, and it was caught at the spec stage, before it ever shipped wrong. It later broke anyway in production for an unrelated reason. See F-R14 in [`../engineering/build-log.md`](../engineering/build-log.md).

## Compliance filter, built into the pipeline

Ltd companies and LLPs are corporate subscribers under UK PECR, so cold email is permitted. Sole traders and unincorporated partnerships are individual subscribers and require prior consent. **82.9%** of the MCS list is Ltd/Limited/LLP by name suffix, detected in the seed-filter stage. Leads that don't clear that filter are written to the CRM and stay eligible for phone outreach, but they're excluded from every email export. The routing logic in [`../outreach/routing-logic.md`](../outreach/routing-logic.md) never sees them.

## Data quality audit: because "measured facts" means checking, not assuming

Run against the live snapshot, including live DNS MX lookups and live HTTP reachability checks on a sample of the recorded websites.

| Check | Result |
|---|---|
| Duplicate certification number | 0. Confirmed safe as the primary dedupe key |
| Invalid UK postcode | 0 of 5,620 |
| MX record present (119-domain live sample) | 98.3% |
| Website missing `http://` scheme | 83.2%, a mandatory Stage 2 fix, or every domain-based enrichment call fails silently |
| Region field filterable | No. England-only. Filtering on it silently drops the 16.5% that are Scotland, Wales and NI |
| Google rating exactly 5.0 | 48.6% of the rated subset. Confirms `google_review_count`, never `google_rating`, is the usable discriminator |

That region-field finding never throws an error. The pipeline runs clean, the CRM fills up, and a sixth of the UK is just quietly absent from every campaign. It only got caught because the audit checked distribution, not just presence.

## Sending capacity: the real bottleneck once the list exists

A 5,000-lead emailable list through 2 inboxes at 40/day takes **59 days for the first email alone**, before any follow-up sequence doubles the volume. The fix isn't pipeline work at all. It's buying 8 to 12 inboxes across 3 to 4 secondary domains and running 2 to 3 weeks of warmup before the first send. Full infrastructure checklist in [`../engineering/launch-readiness.md`](../engineering/launch-readiness.md).

## What this spec got right, and what it got wrong

Every number in this document was measured before the build started, and every one held up once real code ran against real data: the 75.5% competitor-match rate, the 82.9% Ltd share, the single-GET architecture. What the spec couldn't predict, because no dataset can answer it in advance, was the call-complaint rate: what fraction of Priority-1 leads would actually produce a call-related complaint once reviews were analysed. That number turned out to depend on a production bug (F-R14, review text was never actually being fetched) rather than anything measurable in the source data. The gap between what you can measure in advance and what only shows up once you run it is the throughline of [`../engineering/build-log.md`](../engineering/build-log.md).
