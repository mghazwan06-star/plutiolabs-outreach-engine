# Launch Readiness: How a Go-Live Gets Gated

A pipeline that produces a clean CRM row is necessary but not sufficient. Nothing here goes to a real inbox until it clears a series of measurement gates, each one designed to fail cheaply rather than expensively. This is the process discipline behind the technical build: four explicit go/no-go checkpoints, ordered so the cheapest, most informative tests run first.

## Two things with long lead times, started first

Before any pipeline code, two items got identified as the actual critical path. Everything else fits inside the time these take.

Domain and inbox warmup takes 2 to 3 weeks, non-negotiable. Sending domains need DNS authentication (SPF, DKIM, DMARC) and a genuine warmup period before volume sending, or the domains burn and deliverability collapses for good. Nothing about pipeline quality accelerates this. It's simply the longest pole.

The fabricated-proof decision (see [`build-log.md`](build-log.md)) got resolved before any template copy could be considered final, on the reasoning that starting the multi-week warmup clock on copy that might need rewriting anyway wastes the one resource that can't be sped up.

## The four gates

Each gate exists to catch a failure mode cheaply, on a small sample, before it's expensive on the full list.

**Gate 1, Apify place-match rate.** Run the place-lookup enrichment on 100 sample leads and measure what percentage resolve to the correct business listing. Threshold: 70% or better. Below that, a large share of the seed list would never acquire review data, and every one of those leads would be pinned to the weakest available template regardless of how well the rest of the pipeline performed. A data-availability ceiling that no amount of downstream tuning can lift.

**Gate 2, call-complaint rate.** Run 100 Priority-1 leads through full review analysis and measure what share produce a genuine call-related complaint. This is explicitly named as the one number nobody can estimate from any dataset in advance. Everything upstream of it (competitor-match rate, contact-fill rate) is measurable ahead of time. This one only exists once real reviews are actually analysed. If it comes back low, the documented response is to check the review-sort logic before concluding the templates themselves are the problem. An implementation bug and a genuine data-scarcity problem look identical from the output side, and only one of them is worth changing copy over.

**Gate 3, email validity.** Verify 200 sample addresses through an email-verification service. Threshold: under 5% invalid is healthy, over 10% means the source data is stale enough to warrant a fresh pull before continuing rather than pressing ahead and burning sender reputation on a list that's already gone bad.

**Gate 4, end to end.** Run 100 leads through the complete pipeline into the CRM, verify every column that feeds an email is actually populated, and, the step that resists automation entirely, manually read ten finished emails and ask a single, deliberately non-technical question: would a real UK installer reply to this?

The operating rule across all four: do not build past a failed gate. A gate failing is treated as information about where to spend the next hour, not an obstacle to route around.

## Domain and inbox setup, specific enough to actually follow

Never send from the primary business domain. One spam complaint on a primary domain risks the whole company's email deliverability, so cold outreach runs from three to four secondary domains bought specifically for this, using close variants of the main brand name rather than anything unrelated to it. Each secondary domain gets pointed at a simple redirect or a one-page site, since a bare, unused domain with no content is itself a weak deliverability signal, and carries a published privacy policy page, a UK GDPR transparency requirement independent of the PECR rules governing the emails themselves.

Three inboxes per domain, using real-looking human names and a real email signature with a genuine postal address, never `info@` or `sales@` prefixes, since PECR requires an identifiable sender and a generic inbox reads as automated before the recipient even opens the email. Each domain carries full DNS authentication before anything sends from it: SPF, DKIM signing, and a DMARC record starting at the most permissive policy and tightened only once the domain has a sending history to tighten against. A custom tracking domain gets configured in the sending platform rather than using its shared default, and every domain gets verified against a mail-testing tool before warmup begins, not after.

Warmup runs on every inbox from day one, targeting 30 to 40 sends per day per inbox once it completes, the figure the capacity table below is built from. And a reply-monitoring inbox gets connected before the first real send, not after, since a system built entirely around getting a stranger to reply is only as good as how quickly a real reply actually gets read once it lands.

## Sending infrastructure, sized against measured demand

Capacity was modelled against the pipeline's own measured emailable-lead count (roughly 4,600 from the primary seed source alone), not against a generic assumption.

| Inboxes | Rate/inbox | Total/day | Time to clear Email 1 |
|---|---|---|---|
| 2 | 40/day | 80 | 59 days ❌ |
| 6 | 35/day | 210 | 23 days |
| **9** | **35/day** | **315** | **15 days** ✅ |
| 12 | 35/day | 420 | 12 days |

Nine inboxes across three to four domains was the practical minimum identified. Below that, the follow-up sequence (which roughly doubles total send volume once Email 2 through 4 are included) pushes the full campaign past what a reasonable send window can absorb.

## Compliance, gated before the first send, not after

Ltd-only filter enforced at export, not just computed. Sole traders and unincorporated partnerships are individual subscribers under UK PECR and require prior consent for cold email. They're written to the CRM and remain phone-eligible, but structurally excluded from every email export.

A plain "reply STOP" opt-out line sits in every email, judged to read more naturally to a trades audience than a formal unsubscribe footer, wired into a shared suppression list that blocks a contact across every campaign, not just the one they opted out of.

Real sender identity: company name and postal address in every signature, a PECR requirement for identifiable senders.

TPS and CTPS screening happens ahead of the Day-2 phone leg specifically. A separate compliance regime from email, required regardless of entity type, and checked before the call rather than assumed clear.

## First send, deliberately small, with hard stop conditions

The first real send was scoped to 200 leads, sorted best-signal first (multi-source-confirmed leads, then Priority 1, then by template strength), with explicit measurement thresholds at the 72-hour mark: bounce rate under 3%, open rate over 50%, reply rate over 5%. Scaling to full volume is gated on clearing those thresholds, not on elapsed time or completed build work. The two failure modes point at different problems entirely: a high bounce rate means the list needs re-verification, a low reply rate means the copy needs work. Conflating the two would send the fix effort in the wrong direction.
