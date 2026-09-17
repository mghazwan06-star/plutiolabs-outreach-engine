# Cold Email Playbook

**Condensed from a 1,900+ line internal playbook, compiled from published research (Lavender, Gong, Salesforge, Jason Bay's 85-million-email analysis, Josh Braun, Nick Saraev) and cross-checked against our own live test sends.** This is the framework behind every template the pipeline routes leads into. It is not the templates themselves. Why, at the bottom.

## The meta-finding

Copy matters less than targeting and research specificity.

- Tight, well-targeted lists (≤50 contacts) measure a **5.8% reply rate** against **2.1%** for 1,000+ contact blasts. A 2.76x multiplier from list quality alone, before a single word of copy changes.
- Research-driven personalization, referencing a real review, a real competitor, real opening hours, measures a **+52% reply lift** over generic first-name/company-name insertion.
- Emails 50 to 125 words outperform everything longer. Past 200 words, reply rate measures **2.4x lower**.

The single biggest lever is proving you looked at this specific business. Everything else is secondary. This is also exactly why the pipeline described in [`../pipeline/`](../pipeline/) exists: the review complaint, the named competitor, the specific opening-hours gap all come from real per-lead data, not from a mail-merge field.

## Who's actually reading this

UK HVAC, solar and insulation installers. On a job site most of the day, email gets read on a phone in a van between calls. Burned by agencies before, so "digital marketing" translates in their head to Facebook ads that didn't convert. Cynical about vague claims: "triple your revenue" is noise, "3 extra booked installs a month" is a number they can check against their own week.

They respond to trades language, not B2B jargon.

| What they'd say | What an agency would say |
|---|---|
| "Call-outs" | "Service requests" |
| "Booking jobs" | "Lead conversion" |
| "Chasing up quotes" | "Follow-up sequences" |
| "Missing calls on jobs" | "Inbound lead leakage" |
| "Customers calling at night" | "After-hours enquiries" |

That table is rule one of every template written against this playbook, and it applies just as much to how this repository describes its own work. See the voice note in the main [README](../README.md).

## The psychology: loss aversion

People feel a loss roughly twice as painfully as an equivalent gain feels good, one of the most replicated findings in behavioural economics (Kahneman & Tversky). Every email frames around what's already leaving, never what could arrive.

| Weak frame (gain) | Strong frame (loss) |
|---|---|
| "You could book more jobs" | "You're losing jobs right now, today" |
| "Imagine capturing after-hours calls" | "Every call after 5pm is going to whoever answers first" |

### The compounding mechanism, sourced entirely from the CRM data

A missed call doesn't just cost one job. The competitor who answers it books the job, gets the review, ranks slightly higher next month, gets the next call too, and the gap widens every week. This chain gets delivered as a conclusion, not walked through step by step.

> *"Every missed call you send to voicemail is their booking, their five-star, their reason the next customer in [City] finds them first."*

One sentence. The chain is complete. The endpoint is something physical, a job not quoted, never an abstraction like "ranking" or "visibility."

### Never state the revenue number

The temptation is to calculate the prospect's losses for them: "that's £X a year walking out the door." Don't. You don't know their call volume, close rate or average job value, and if the number you guess doesn't match their own mental model, credibility is gone for the rest of the email.

Instead, imply the magnitude and let them do the arithmetic themselves. `"At [trade] prices, [X] isn't a small number."` Their brain fills in their own number, always the right one, because they know their margins better than a cold email ever could, and it lands harder because they calculated it, not you.

### CRM data determines which pain angle is available

Every lead carries different data. The pipeline computes which signal is strongest and routes accordingly. Full routing logic in [`routing-logic.md`](routing-logic.md). The hierarchy, strongest to weakest:

1. A real complaint from their own reviews plus a named competitor gap
2. A named competitor gap alone (10+ reviews ahead)
3. A real complaint alone, no credible competitor to name
4. An hours gap against a competitor with weekend coverage
5. No strong signal. Trade and scene only

### How much pain is enough

The target is one punch that lands, not a pain section. A named competitor, one specific gap, one consequence, one implied revenue magnitude, then move to the offer. Overdoing it (stacking three or four loss-frames in a row) reads as being sold to, and the reader stops before reaching the offer that was supposed to convert them.

## The four-part structure (Saraev framework)

Every email in the sequence fits this shape inside 100 to 120 words:

1. **Personalization.** An observation about their specific business, one to two sentences, never opening with "I"
2. **Who am I.** Social proof as the introduction, not a separate credential claim
3. **Offer.** A risk-reversal offer where all the downside sits on the sender's side, not the prospect's
4. **CTA.** A single, specific, low-friction ask

Underneath these four parts sit seven psychological triggers pulled from the same research base: give first, micro-commitments, specific social proof, domain-relevant authority, genuine rapport, real (never fabricated) scarcity, shared language. "I'm one of you," meaning trades language and trades pain points with no jargon, builds trust faster than any credential line.

## Subject lines: the data is counterintuitive

Sourced from Lavender's analysis of millions of live sends.

| Element | Measured effect |
|---|---|
| Question mark in subject | -56% opens |
| A number in the subject | -46% opens |
| Any punctuation | -36% opens |
| First name in subject | -12% replies |
| Slightly casual tone | **+23% replies** |

The rule: internal camo. A subject line should be able to appear in an internal Slack thread without looking out of place. `Missed Calls`, `Booking Rate`, `Lead Response`. Never "Increase Your Bookings With Proven AI Strategies." The goal isn't to trick anyone. It's to not trip the spam-pattern-recognition a busy tradesperson has built up from years of templated pitches landing in the same inbox.

## The first line

The inbox preview line is a second subject line. Most prospects decide open-or-delete on subject plus the first ten words. Never opens with "I" or "We" (self-focus, kills opens), never "Hope this finds you well" (an instant delete signal). Always opens with "Noticed" or "Saw" plus a real, specific, checkable data point, under 20 words.

Four tiers, strongest signal first: a review complaint (+25-35% reply lift when it's real and specific), a competitor review-count gap, an hours/availability gap, and, for data-sparse leads, a niche-plus-city fallback.

## Body structure and the BAR format

Plain text only, 50 to 100 words target, never over 125. No HTML, no images, no tracking pixels in the first email. A tracking pixel alone measures a 10 to 15% reply-rate penalty, because spam filters detect them and lower the email's trust score before a human ever sees it.

Social proof follows the BAR format: Background (a client similar to this prospect, same trade, similar size, UK-based), Action (specifically what changed), Result (a measurable outcome in units the reader cares about: jobs booked, calls answered, leads recovered). One rule governs every BAR line and it's non-negotiable: the company and the result must be real and stand-behind-able, or the line doesn't ship.

That rule isn't theoretical. See the note at the bottom of this document for what happened the one time it was checked properly before send rather than after.

## CTAs: the part that changed most in testing

The playbook's original position was a direct time-ask in the first email ("Worth a quick call? I'm free at 3:30pm today"). Later research, plus a direct pattern-recognition problem, moved every template off it. A specific time-ask reads as a sales signal on a cold first touch regardless of phrasing.

The resolved position, backed by a Gong study of 304,000+ sales emails and a Jason Bay analysis of 85 million cold emails: on a cold first touch, an interest-based or offer-based CTA outperforms a direct meeting ask. A concrete offer ("want the numbers behind it?") beats a vague curiosity line ("Curious?") by roughly 4x, because a vague one-word CTA has itself become a recognisable cliché to anyone who's seen enough templated cold email, which is everyone on this list.

The rule that ties it together: every noun a CTA references needs a real antecedent already in the email. An offer CTA can only ever name something actually buildable from real per-lead data. A specific-sounding offer that isn't backed by real data reads as fake precision, which is worse than an honestly vaguer one. Same standard the BAR rule holds copy to.

## The offer frame: demonstration, not pitch

The ask should never read as a request for the reader's time in exchange for nothing. It should read as evidence of competence offered before any commitment is asked for.

Weak: "Can we schedule a call to discuss your lead generation?" This asks for time before giving anything.

Better: "Happy to show you exactly how we'd set this up for [Company], no pitch, just a walkthrough."

Best: "Put together a quick look at how [Company] is currently set up for after-hours calls, found a couple of things worth a look. Want me to send it over?"

The strongest version does two things at once: it demonstrates that real, specific research already happened (this isn't a template with a name swapped in), and it asks for nothing beyond a reply. The reader isn't agreeing to a meeting, they're agreeing to receive something that already exists.

## Call-to-action formulas, by situation

| Type | Formula | Use when |
|---|---|---|
| Soft interest probe | "Worth [X] minutes to see if it applies to [Company]?" | Default, low-commitment |
| Poke the bear | "Is that something you're dealing with, or have you got it handled?" | After stating a specific, real pain, gives a genuine way to say no |
| Value delivery ask | "Want me to send over what this looked like for a similar installer?" | Offering the case-study evidence directly |
| Binary question | "Does this make sense for where [Company] is right now?" | When the empathy line already did the heavy lifting |
| Permission-based | "Should I follow up, or is timing not right?" | Later in a sequence, after no reply |

The "poke the bear" formula is the one Template D's call-to-action eventually landed on after four earlier attempts, documented in full in [`template-redesigns.md`](template-redesigns.md), and it's worth noting it had never actually been used by any template until that point despite being in the formula list from early on. Having a formula documented doesn't mean it gets reached for; a defect in one template surfaced a formula that had been sitting unused.

## Hook type performance: why the case study leads, not the pain point

Measured across live sends, by hook type:

| Hook | Reply rate | Meeting rate |
|---|---|---|
| Timeline ("helped X in Y weeks") | 10.01% | 2.34% |
| Numbers ("3x more jobs") | 8.57% | 1.86% |
| Social proof lead | 6.53% | 1.25% |
| Problem statement | 4.39% | 0.69% |

A timeline-shaped case study outperforms a flat problem statement by 2.3x on reply rate alone. This is the direct evidence behind leading every template's proof line with a background-action-result structure rather than opening on the pain point and only mentioning evidence afterward, and it's also why the case-study line rule (state the result, never the mechanism) matters as much as it does: the format that reply-rate data says works best is exactly the format that's easiest to accidentally turn into a pitch by explaining how the result happened.

## The two lead categories, and what determines which one a lead falls into

Every lead resolves into exactly one of two categories, and the category, not the trade or the city, is what determines the shape of the entire sequence a lead receives:

**Category 1, data-rich.** A competitor gap of 10 or more reviews exists, and either a real complaint or a real positive quote was extracted from reviews. Gets the full personalised sequence: a named competitor, a specific number, and either a complaint or a quote anchoring the email. This is the segment the A-family of templates serves.

**Category 2, data-sparse.** Either no meaningful competitor gap exists, or no usable review data was extracted. Gets a simplified sequence built around what's reliably available for every lead regardless of review depth: trade, city, review count if any. This is the segment Templates B and D serve, and it's precisely why D's call-to-action was the hardest one to get right: there is structurally nothing concrete to offer without inventing detail, which is what makes its "poke the bear" formula the correct choice rather than a compromise.

## Personalization data priority

Every signal available from the pipeline is ranked by measured reply-rate impact, so a template always reaches for the strongest available data point first rather than treating every field as equally worth using:

| Signal | Reply lift | How it gets used |
|---|---|---|
| Review complaint (topComplaint) | +25-35% | "Noticed a few reviews mention [complaint]," the most personal hook available |
| Review quote (bestReviewQuote) | +25-35% | The specific quote goes straight into the first line |
| Competitor gap (review counts) | +35-50% | Name the competitor, cite the real number gap |
| Hours gap (no weekend hours, early closing) | +20-30% | Infer missed after-hours calls |
| Business category / niche | +10-15% | Confirms the sender actually knows their specific trade |
| No website | +20-30% | A credible gap in its own right |
| Certifications / business attributes | +15-25% | MCS, Gas Safe and similar signals read as valuable niche knowledge |
| Negative or mixed review sentiment | +20-30% | Implies an opportunity to turn things around |
| Competitor name alone, no gap number | +10-15% | Adds credibility to the competitive frame even without a number |

This hierarchy is what the routing logic in [`routing-logic.md`](routing-logic.md) is built to serve: put each lead into the template that can actually use its strongest available signal, never a template that has to reach for a weaker one when a stronger one exists on the row.

## Benchmarks referenced against real outbound data

| Metric | Industry average | Solid | Strong |
|---|---|---|---|
| Open rate | 27.7% | 35-45% | 45-60% |
| Reply rate | 3-5% | 5-10% | 10-15% |

Hook type is the biggest lever inside reply rate itself, and the gap between hook types is large enough to change which one gets used by default: a timeline-shaped hook ("helped X in Y weeks") measures 10.01% reply versus 4.39% for a flat problem statement, a 2.3x difference from hook shape alone, holding everything else constant. This is the direct evidence behind leading with the BAR result rather than the pain point, referenced in the body-structure section above.

## UK legal compliance, built into the routing, not bolted on after

| Entity type | Cold email permitted? | Requirement |
|---|---|---|
| Limited company (Ltd) | Yes, under Legitimate Interest | Opt-out and sender identity disclosure required |
| LLP | Yes | Same as Ltd |
| Sole trader | No, without prior consent | Very common in this ICP, has to be filtered out, not just flagged |
| Partnership | No, without prior consent | Same consumer-level protection as a sole trader |

Every email carries four things as a floor, not a target: a real name and company name, a plain statement of how the contact details were sourced ("found your business on Google Maps"), a one-click or reply-based unsubscribe, and a physical business address. Segmentation by entity type happens before a lead is ever eligible for export, not as a filter applied at send time, which is the same architectural decision documented in [`../pipeline/dual-source-design.md`](../pipeline/dual-source-design.md)'s compliance section: keep a non-compliant lead in the CRM for phone outreach, exclude it from every email campaign structurally, so there's no send-time judgement call to get wrong.

## What kills a reply before a word is read

Split into two categories that get diagnosed differently, because the fix for one never fixes the other. A technical failure (missing SPF/DKIM/DMARC, sending from a primary business domain, a bounce rate over 2%, a spam-complaint rate over Gmail's 0.1% threshold, sending above 30 emails per inbox per day post-warmup) means the email never reaches an inbox regardless of copy quality. A copy failure (starting with "I," a generic case study with no specifics, more than one call-to-action, a buzzword list including "AI," "automate," "scalable," "revolutionise," any price or ROI math in the first email) means the email arrives and gets deleted or ignored. Diagnosing a low reply rate starts by ruling out the first category before touching the second, since no amount of copy work fixes a deliverability problem.

## Cross-template fixes

Five templates were built as structural siblings of one original skeleton, and eight defects turned out to be shared across some or all of them rather than isolated to one template's copy. That full checklist, including the ones only caught by reading finished copy end to end rather than any structured pass, is documented separately: [`cross-template-fixes.md`](cross-template-fixes.md).

## What this document deliberately leaves out

The full 1,900-line source playbook contains eight filled, per-template example emails with specific client-result numbers. Those numbers were caught as fabricated before send: invented BAR examples that had been drafted as placeholders and never replaced with real client results, flagged as a hard launch blocker the moment it was found, and never sent. The framework above, everything that governs how a real result gets written into an email once one exists, is unaffected and is what's documented here. See [`../engineering/build-log.md`](../engineering/build-log.md), finding F-A2, for the full write-up: what was found, why, and what shipped instead of it.
