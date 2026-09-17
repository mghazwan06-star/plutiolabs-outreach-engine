# Sequence & Cadence

## The full cadence

```
Day 1      Email 1 sent    (Template A, A-v2, A-lite, B or D: see routing-logic.md)
Day 2      Phone call      (Call Prep Card if Priority 1, basic data if Priority 2)
Day 2      WhatsApp follow-up, same day, referencing the call. Opt-in only, never cold
Day 3-4    Email 2 sent as a reply in the same thread   (E1, E2 or E3)
Day 8-10   Email 3 sent as a reply, pattern-interrupt, short
Day 14-17  Email 4 sent as a reply, break-up, no ask
```

Every follow-up email sends as a reply inside the original thread, not as a new message. Thread continuity is a deliberate deliverability choice: a reply carries the sender-recipient relationship the mail server already trusts, rather than starting cold each time.

## Email 2: three angles, chosen by what Email 1 already established

Email 2 isn't a second attempt at the same pitch. It picks up a different piece of the same lead's real data, whichever one Email 1 didn't already use, so the recipient gets new information rather than a repeated ask.

**E1, the hours-gap angle.** Fires for Category 1 leads (competitor gap of 10+ reviews) whose opening hours show an early close or no weekend coverage. The angle: their own listed hours are handing every evening and weekend enquiry straight to the competitor already named in Email 1. This is the strongest Email 2 available, because it names a second real, checkable fact rather than restating the first one.

**E2, the gap-compounding angle.** Fires for the same Category 1 leads when no hours data exists to build E1 from. Restates the competitor gap from Email 1, but adds the time dimension: the gap hasn't closed since Email 1 went out, and every day that passes is more bookings and more reviews accumulating on the competitor's side. Same core fact as Email 1, viewed through a different lens (time, not just size) rather than duplicated outright.

**E3, the response-time angle.** Fires for every Category 2 lead, meaning both Template B and Template D routes, regardless of whether a complaint exists. This is deliberate, not a routing gap: E3's universal missed-call framing doesn't contradict Template B's complaint-specific Email 1, it deepens the same underlying mechanism from a different angle, without needing per-lead complaint or competitor data that these routes don't have.

Selection logic in full:

```
Category 1 lead? (Template A, A-v2, or A-lite)
├── YES
│   ├── Opening hours show an early close or no weekend coverage → E1
│   └── No hours data → E2
└── NO → Category 2 (Template B or Template D)
    └── E3, regardless of complaint presence
```

## Email 3: a documentation drift caught by checking the live step against the spec

The original design for the third-touch email specified an HTML pattern-interrupt message built around an image, roughly 30 words of surrounding text. That was never actually built. The live version is plain text only, no image, no subject line (it threads as a reply), a short two-line nudge asking whether to keep following up.

This was found and corrected during an unrelated audit of a different template's live sequence, when checking that template's actual four-step flow against what the documentation claimed the shared Email 3 step did. The documentation had described the original plan rather than what had shipped, and nothing had gone back to correct it once the simpler plain-text version replaced it. The fix was mechanical (rewrite the doc to match the deployed step), but the finding itself is the same class of lesson as the API-contract and competitor-matching documentation drift logged in [`../engineering/build-log.md`](../engineering/build-log.md): a doc can go stale silently, and the only way to catch it is checking it against the live artefact rather than trusting that it was accurate when it was written.

## Email 4: the break-up, no ask

Two documented formats exist for this final touch, both built around the same principle: remove pressure rather than apply more of it, since a fourth email that's still asking for something reads as exactly the persistence this audience already distrusts from agencies.

**The Surrender**, the format actually deployed: take the blame for not having said the right thing yet, ask nothing, leave the door open with a plain "reply if anything changes." No call-to-action, no question requiring an answer.

**The Presumptive Negative**, documented as an alternative but not the one shipped: assume the reader has moved on, state that outreach is stopping, and offer future help without asking for anything now either. Where the Surrender takes the blame onto the sender, the Presumptive Negative simply states the read (no reply means timing isn't right) and closes the loop without asking the reader to confirm or explain anything.

Shared across every route rather than templated per-route, since by the fourth touch with no reply, further personalisation has diminishing value against simply being the kind of sender who doesn't keep pushing.

## LinkedIn: one touch, mirrors the email routing, gated on data availability

LinkedIn uses the identical scraped data as the email sequence, no additional research per lead. A short, no-pitch connection request goes out first; if accepted, a single templated message fires, matched to the same template route the lead already has for email. There is no LinkedIn follow-up sequence, one message only, since a second unsolicited message after an accepted connection request reads very differently on LinkedIn than a follow-up email does in an inbox.

Routing mirrors the email system almost exactly, with two exceptions where the richer LinkedIn variant doesn't exist yet and the lead falls back to the generic one:

| Email template route | LinkedIn template |
|---|---|
| A | LinkedIn A (complaint + competitor gap) |
| A-v2 | LinkedIn A-v2 (positive reviews + competitor gap) |
| A-lite | LinkedIn D (fallback, no complaint or quote data to build A/A-v2's opener from) |
| B | LinkedIn D (a complaint-specific LinkedIn variant doesn't exist yet) |
| D | LinkedIn D |

A lead only enters LinkedIn outreach at all if a LinkedIn URL was actually resolved during contact enrichment. Without one, the lead skips the channel entirely rather than falling back to a generic connection request with no personalisation basis.

## A finding worth its own callout: fixes to Email 1 don't automatically propagate downstream

Several of the [cross-template fixes](cross-template-fixes.md) applied to Email 1 copy exposed a second, separate class of issue once the shared Email 2/3/4 templates got checked against them: a fix made to one template's first touch doesn't automatically update the shared templates further down the same sequence, and those templates can end up contradicting the very thing that was just fixed upstream.

Three concrete instances, logged rather than silently patched, because each one needs a decision about scope before it gets touched:

- One of the shared Email 2 variants still uses the same call-specific framing ("the calls you miss") that Email 1's complaint-driven templates deliberately moved away from, once real data showed most complaints weren't actually about missed calls. The Email 2 variant is shared by three separate routes, so fixing it isn't a single-template edit, it's a decision that has to hold correctly for all three at once.
- The universal response-time Email 2 variant still uses the raw, non-deduplicated trade-scene phrase rather than the de-duplicated version Email 1 switched to after the trade-word-repetition fix. This variant is shared by two routes with different upstream Email 1 copy, so the same fix has to be verified against both before it ships.
- The break-up email references "inbound calls" specifically as the thing a lead's business handles, a leftover from when every Email 1 template was call-specific. Once the complaint-driven Email 1 templates were rewritten to stop asserting a call-specific cause, this line started contradicting its own sequence's earlier emails, and it's shared across every route, so the fix has to be worded generally enough to stay true for all five.

None of these are wrong in a way that would mislead a specific recipient, since none of them assert a false claim about that individual lead. They're inconsistencies between what a sequence's first touch established and what a later touch in the same thread assumes, the kind of drift that's easy to miss because each template gets audited on its own rather than read as a connected four-email conversation. Flagged here rather than patched immediately, because fixing a shared template means checking the fix holds correctly for every route that depends on it, not just the route that surfaced the problem.

## Post-launch: the recurring stream, not just the one-off backlog

The MCS dataset is a finite backlog, roughly 5,600 records, burned through once, not a renewable source of new leads on its own. The renewable side is Google Maps keyword discovery on the trades MCS structurally can't cover (boiler, gas, plumbing, insulation, none of which require MCS certification), plus one recurring signal worth calling out on its own.

MCS adds new certified installers continuously, measured at roughly 50 to 80 per month by comparing successive snapshots. A newly certified business has just spent real money and effort specifically to start winning grant-funded work and wants jobs immediately. "Saw you just came through certification" is a stronger, more time-sensitive opener than anything available to the bulk backlog list, and it costs almost nothing to generate: a lightweight monthly diff against a small index file, enriching only the genuinely new records rather than re-running the whole pipeline.

| Stream | Volume | Intent |
|---|---|---|
| MCS backlog (one-off) | ~4,600 | Medium |
| **MCS new certifications (monthly)** | **50-80/month** | **High, a real trigger event, not a cold list** |
| Google Maps keyword discovery | Renewable, on-demand | Medium |
