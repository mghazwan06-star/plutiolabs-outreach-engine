# CRM Schema: 34+ Columns

The CRM is a Google Sheet, not a database, by choice. Why, at the [bottom](#why-a-spreadsheet-and-not-a-database). Four systems write to it or read from it at various points (the scraper pipeline, the routing logic, the outreach push, and manual tracking by a human), and that's exactly the kind of contention that makes a written schema worth having rather than reverse-engineering column meaning from code every time something needs to change.

## Data columns: written by the pipeline

| Column | Header | Source | Notes |
|---|---|---|---|
| A | Company Name | MCS / Google Maps | |
| B | City | MCS `town` / Maps | |
| C | Phone | MCS / Maps | Kept in local UK format on the sheet, not E.164. See the write-pattern note below for why this mattered |
| D | Website | MCS / Maps + Apify fill | ~85% coverage after enrichment |
| E | First Name | Enrichment API, MCS email local-part fallback | ~60% |
| F | Email | **MCS 100%**, enrichment upgrade where it adds a named contact | ~100% once MCS entered the pipeline |
| G | LinkedIn URL | Enrichment API | ~30-40% |
| H | Google Maps URL | Apify Tier-1 lookup | ~85% |
| I | Google Rating | Apify Tier-1 | ~85%. Never used as a routing discriminator (see [`icp-scoring.md`](icp-scoring.md)) |
| J | Review Count | Apify Tier-1, MCS pre-fills some | ~85%. Drives ICP threshold and competitor gap |
| K | Business Category | Apify Tier-1 | Noisy. Superseded by Technologies where MCS data exists |
| L | Opening Hours | Apify Tier-1 | Feeds the hours-gap email variant |
| N | Top Complaint | Claude, from Tier-2 review text | Priority 1 only |
| O | Best Review Quote | Claude, from Tier-2 review text | Priority 1 only |
| P | Top Competitor | Competitor match stage | ~75%+ of Priority 1 |
| Q | Competitor Reviews | Competitor match stage | Paired with P |
| R | Competitor Opening Hours | Competitor match stage | Needs the competitor to also have a Tier-1 match |
| S | Priority | ICP score stage | `1` or `2` |
| T | Template Route | Build CRM Row | See [`../outreach/routing-logic.md`](../outreach/routing-logic.md) |
| U | Email 2 Route | Build CRM Row | Derived from Template Route, not raw gap |
| V | Hours Gap | Build CRM Row | Boolean |
| W | Closing Time | Normalize stage | Numeric hour |
| X | Trade Scene | Build CRM Row | 100% fill. The fallback template's entire personalisation budget |
| Y | Competitor Has Weekend | Build CRM Row | Boolean |
| Z | Call Prep Card | Claude | Priority 1 only. Briefing for the Day-2 call |
| AH | Trade | Build CRM Row | 100% fill, prefers MCS technologies |
| AI | Review Gap | Build CRM Row | Competitor reviews minus own review count |

## New columns added when MCS entered the pipeline

| Column | Header | Why it exists |
|---|---|---|
| AJ | Source | `mcs` / `maps` / `both`. Lets reply rate be measured by source |
| AK | MCS Cert Body | Channel-partner segmentation (NICEIC, NAPIT, etc.) |
| AL | MCS Cert Number | Credibility line, and the stable dedupe key for the monthly MCS diff |
| AM | Technologies | Precise trade targeting, cleaner than scraped categories |
| AN | BUS Registered | Signals eligibility for the £7,500 government grant. A real, factual personalisation angle |
| AO | Entity Type | Derived from name suffix. Drives the PECR compliance filter |
| AP | Email Source | `mcs` / `enrichment`. Measures named-contact reply rate vs generic |
| AQ | Postcode | Geo segmentation for sending batches by region |

## Tracking columns: filled manually

| Column | Header | Filled by |
|---|---|---|
| AB | Status | User. Updated as a lead progresses through outreach |
| AC | Date Added | Pipeline, auto, ISO timestamp |
| AD | Email Sent | User |
| AE | Call Status | User, after the Day-2 call |
| AF | LinkedIn Status | User |
| AG | Notes | User, free text |

## The write pattern: and why it changed twice

The obvious approach is `appendOrUpdate` matched on phone number. It's idempotent in theory, so running the pipeline twice on the same data updates rather than duplicates. That was the design going in. It turned out to have three separate, compounding failure modes, found in production, in this order.

**Format mismatch.** The pipeline wrote phone numbers with a leading `+` (E.164 style). Google Sheets' `USER_ENTERED` write semantics interpret a leading `+` as a formula prefix and coerce the value, so the stored value silently lost the `+`. The match key on the next run, still built with a `+`, matched nothing, so every re-run appended a full duplicate copy of the dataset instead of updating it.

**Uniqueness.** Even after fixing the format, phone number turned out to be the wrong key entirely. Some records legitimately share a phone number in the source data (multiple businesses under one office line), and a meaningful fraction had no phone at all. A shared or blank key doesn't just duplicate rows. It lets `appendOrUpdate` silently overwrite one business's row with another's data.

**The fix.** A dedicated `Row Key` column, populated in priority order (certification number where it exists, then normalised domain, then normalised phone, then a name slug as the last resort), guarantees a unique, non-empty key for every row regardless of source. `matchingColumns` switched from Phone to Row Key. Verified after the fix: 5,603 rows, zero duplicates, zero missing, and re-runs became genuinely idempotent for the first time.

Full incident trail, including the specific row that got overwritten and how it was traced and restored, is in [`../engineering/build-log.md`](../engineering/build-log.md) under F-R3 through F-R7.

## Cross-system contention: the ten utility workflows

The CRM is written to by the discovery pipeline, read by the routing/outreach push, and edited manually by whoever runs Day-2 calls. That's the shape of a spreadsheet-backed CRM under real concurrent use, and it's why ten maintenance workflows exist around it rather than zero.

| Utility | Purpose |
|---|---|
| CRM Audit | Full sweep of record health: fill rates, blank-field detection |
| CRM Integrity Check | Rebuilds the expected row set from source data and diffs it against the live sheet. The tool that caught the missing/overwritten row incident |
| Dedupe CRM | Duplicate detection and merge, keyed on Row Key |
| CRM Status | Status distribution snapshot |
| Routing Audit | Checks every row's Template Route against the routing rules, to catch drift after a rule change |
| Route Recompute | Backfills routing decisions across the whole CRM after a routing-rule change, at zero enrichment cost. See [`../engineering/build-log.md`](../engineering/build-log.md) for a worked example that upgraded 713 rows without a single Apify or Claude call |
| Contact Quality | Scores completeness and deliverability |
| Add Row Key + Size Tier | Backfills the Row Key and a review-count size tier for existing rows |
| Write CRM Headers | Repairs the header row if it ever drifts from the schema |
| CRM to Instantly Push | The outreach connection. See [`../outreach/routing-logic.md`](../outreach/routing-logic.md) |

Several of these exist because something went wrong earlier and needed a repeatable fix rather than a one-off patch. The header repair, the deduper and the route backfill are all cleaning up after bugs logged in the build history.

## Why a spreadsheet, and not a database

A proper database would remove most of the utility workflows above: no positional-write fragility, no header-drift risk, real constraints instead of a diff script checking for them. That trade was made on purpose, in favour of a store the client can open, read and understand without a technical intermediary. The cost is the ten utilities. The benefit is that "the CRM" means something a non-technical business owner can actually look at.
