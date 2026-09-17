# Routing Logic

Every lead's outreach path, which email template, which follow-up, whether LinkedIn or WhatsApp applies, is computed once, in the pipeline's Build CRM Row stage, and written to the CRM as data. Instantly (the sending platform) does no routing logic of its own. It receives leads already sorted, via campaign assignment or a direct API push. This split matters, because it keeps the routing rules in one auditable place, in version-controlled code, rather than scattered across email-platform conditional-send rules that are hard to review as a set.

## Email 1: Template Route

```
gap = competitorReviews - reviewCount
callComplaint = topComplaint exists AND is call/response related

IF priority == 1 AND gap >= 10:
  ├── callComplaint            → Template A   (complaint + competitor gap, richest data)
  ├── bestReviewQuote          → Template A-v2 (social proof + competitor gap)
  └── neither                  → Template A-lite (competitor gap only)

IF callComplaint (any priority with a complaint):
  → Template B   (complaint only, no credible competitor gap)

ELSE:
  → Template D   (fallback: trade and scene only)
```

Two guards stop a template from making a claim its data doesn't support.

First, A and B both require a call-related complaint specifically. Both templates pivot the email body to a missed-call narrative. A complaint about something unrelated (late quotes, no-shows, workmanship) routed into a call-complaint template would produce a factual mismatch the recipient can spot immediately, because it's about their own business. Non-call complaints route elsewhere.

Second, the A-family requires Priority 1. A Priority 2 lead with a big gap but no review data of its own routes to D, not into a template that implies review-based personalisation it doesn't actually have.

### The keyword-list tuning that mattered more than expected

The call-complaint keyword list started at 19 terms and was later widened to 35, adding responsiveness terms the original list missed (`appointment`, `schedul`, `no-show`, `turn up`, `unrespon`, `chase`, `ignore`, `missed`). This was done narrow on purpose. Terms like `wait`, `delay`, `price` and `workmanship` were considered and excluded, because Template B's copy is specifically call-focused ("the phone rings and you miss it"), and routing a pricing complaint into it would produce an email that doesn't match what the recipient's own customers actually said. An earlier estimate that this widening would recover roughly 23 additional leads turned out to be wrong once measured against real data. The real number was closer to 5 to 10, because most of the added keyword matches were themselves outside the narrow scope that was set. Measuring against real leads rather than trusting the estimate is what caught the gap.

## Email 2: driven by the Email 1 route, not raw data

```
IF templateRoute is A, A-v2 or A-lite:
  ├── hours gap AND competitor has weekend coverage → E1
  └── otherwise                                      → E2

IF templateRoute is B or D:
  → E3
```

Keyed off the Email 1 route rather than recomputed from scratch, on purpose, so a lead never receives a follow-up that references a competitor Email 1 never mentioned. B and D leads, which never introduce a competitor in the first email, always get E3, the value-only follow-up.

## How the full lead population actually segments

Numbers below are from a real run against the complete reconciled dataset (5,603 leads after dedupe), not a projection. This is the shape routing produces once every lead has been scored and enriched:

| Split | Count | Share |
|---|---|---|
| Priority 1 (full pipeline eligible) | 1,822 | 32.5% |
| Priority 2 | 3,781 | 67.5% |

Template route distribution, after the routing rule changes described below had been applied (so this reflects the improved, not the original, split):

| Route | Count | Share of send-ready |
|---|---|---|
| A (complaint + competitor gap) | 22 | 1.1% |
| A-v2 (quote + competitor gap) | 19 | 1.0% |
| B (complaint, no credible gap) | 14 | 0.7% |
| A-lite (gap only) | 1,912 | 97.2% |
| D (fallback) | 2,674 | across the full non-A-family population |

The A-family (A, A-v2, A-lite combined) versus D is the more useful cut for measuring quality, since D leads get the least personalised copy by design: **1,967 leads (anything better than the fallback) versus 2,674 D-routed**, a 42% quality share of the emailable list once every enrichment and routing fix had been applied.

## Routing gate fix: a genuine 5,603-lead correction, measured before and after

A routing condition originally counted a complaint only if it was specifically call-related, a holdover from when the two complaint-driven templates' copy was call-specific. Once that copy was rewritten to work with any complaint type, the routing gate was never widened to match, so a real, substantive complaint about something else (pricing, workmanship, no-shows) still fell through to the generic fallback template instead of a complaint-driven one.

**Fixed by widening the gate** from "complaint exists and is call-related" to simply "complaint exists," applied in two places: the main enrichment pipeline (governs every future lead) and the Route Recompute utility's own copy of the same logic (needed the identical fix, or it would never have applied to the existing backlog). Both were tested in isolation against real leads before touching production.

**Applied retroactively to the full existing CRM**, not just future leads. Route Recompute ran in dry-run mode first against all 5,603 real rows, got reviewed, then committed. Real result: **13 total changes, 0.23% of the CRM**. Five leads moved from the fallback template to the complaint-driven one (a real complaint, now correctly used). Eight moved from the quote-based template to the complaint-driven one (leads that had both a complaint and a quote, now correctly prioritising the complaint, which is the stronger signal per the hierarchy in [`playbook.md`](playbook.md)).

**A measurement-discipline note worth keeping.** An early small-sample check on 10 leads suggested roughly 70% of the fallback template's population was misrouted by this bug. A proper 40-lead audit showed the true rate was closer to 5%. The small sample was simply unlucky, not representative. The standing rule that came out of it: don't trust a routing-scale estimate built from a sample under 30 to 40 real leads, the variance at that size is large enough to produce a wildly wrong headline number.

## The A-lite expansion: a rule change applied without re-scraping a single lead

Two "levers" widened the A-family beyond its original scope, and both were applied to the existing CRM at zero additional Apify or Claude cost, because the routing rule only reads columns already stored on each row.

A P2 lead with a competitor gap of 30+ reviews now qualifies for A-lite, even without enough of its own reviews to hit Priority 1. The A-lite angle doesn't require the lead itself to have many reviews. It lands, arguably harder, precisely when the lead has few and the gap is large. Gated at 30 rather than 10 specifically to exclude very small businesses the offer wouldn't fit.

The keyword-list widening above was applied the same way.

Applying a routing-rule change to already-scraped data, at zero re-enrichment cost, is exactly what the Route Recompute utility exists for (see [`../pipeline/crm-schema.md`](../pipeline/crm-schema.md)). A worked run against the full CRM reassigned 713 rows' template routes (some of those reassignments moved a lead into a stronger tier, others moved it between two templates within the same tier, such as A-v2 to A). Net effect on the send-ready quality inventory, anything better than the fallback: 1,372 rows before, 1,967 after, a 43% increase, without a single additional API call.

## LinkedIn and WhatsApp: one replaced the other

The original plan generated LinkedIn connection requests and messages per lead, mirroring the email routing. Two things killed that channel.

LinkedIn coverage on this ICP measured 30 to 40% at best. Small UK tradespeople are thinly represented in B2B contact databases, so a large share of the list was never reachable there. And the fabricated-proof finding (see [`playbook.md`](playbook.md) and [`../engineering/build-log.md`](../engineering/build-log.md) F-A2) was first caught living inside the LinkedIn message generator, not the email templates. Same invented figures, just in code rather than copy.

LinkedIn generation was removed entirely rather than patched. WhatsApp replaced it: mobile-number coverage on this list measures roughly 45% (from the mobile-prefix flag already present in the phone data), which beats LinkedIn's reach, and it's the channel this audience actually works from on a job site.

The WhatsApp replacement makes no performance claims at all. Not an oversight, a deliberate constraint following directly from why LinkedIn got cut. And critically, WhatsApp is never sent cold. Export only happens where a prior contact already occurred: a call answered, or an email replied to. This is stricter than the legal minimum (the Ltd/sole-trader PECR split works the same way for WhatsApp as for email), because Meta's own Business Messaging Policy bans unsolicited business-initiated contact and will suspend a number for it. The Day-2 phone call, when it happens, is treated as the opt-in event that makes a WhatsApp follow-up legitimate.

## The Day-2 phone call

Sits between Email 1 and Email 2 in the cadence, using the Call Prep Card (Priority 1) or basic company data (Priority 2) computed earlier in the pipeline. Not automated, a human makes this call, but the data that makes the call efficient is fully assembled before the caller ever picks up: lead summary, the identified complaint, the competitor gap context, and suggested openers. Full cadence in [`sequence-and-cadence.md`](sequence-and-cadence.md).

## From CRM to send: the direct API connection

The original design exported the CRM as a filtered CSV and uploaded it to Instantly by hand, once per template route. The CRM to Instantly Push workflow replaces that process with a direct API connection, built against Instantly's live API contract (verified against current documentation before building, not assumed from memory or an earlier integration, see the standing rule on this in [`../engineering/build-log.md`](../engineering/build-log.md)).

Three independent safety layers sit on every push, because a duplicate send to the same business is exactly the kind of mistake that damages trust in a small, connected trade.

Status-gated selection: only rows marked `new` are eligible, and a successful push marks them `queued` before the next batch can select the same rows again.

Server-side dedupe: the push sets `skip_if_in_campaign` and `skip_if_in_workspace`, so Instantly independently refuses a lead already present anywhere in the account.

Explicit double confirmation: nothing writes or pushes without both an explicit `apply: true` flag and a real campaign ID. A dry run is the default, not an opt-in.

The push also restricts to named-person emails by default, excluding role inboxes (`info@`, `sales@`) from automated sending, since those addresses are lower-converting and higher-risk to send unsolicited commercial email to at volume.
