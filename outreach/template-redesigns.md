# Template Redesigns: Before and After

The [cross-template fixes](cross-template-fixes.md) document describes the eight defects found across five templates in the abstract. This document shows the actual line-level changes for each template, because the abstract version understates how much of this was trial and error, not a single clean rewrite applied once. Template D's call-to-action alone went through five different versions before landing on one that actually worked, and that story is worth telling in full rather than summarised down to "we fixed the CTA."

All examples below use the templates' own placeholder syntax (`[Company]`, `[City]`, `{{tradeScene}}`), the generic skeleton every real lead's data gets dropped into, not filled examples from real businesses.

## Template A: complaint plus competitor gap, the richest template

**The call-to-action, before:**
> "Worth a quick call to see how that gap could close? I am free at 3:30pm today or any time tomorrow morning."

This asks for a call and a specific time in the first email a stranger ever receives, which research says creates avoidable friction on a cold first touch regardless of phrasing, and it references "that gap" without the word "gap" ever having appeared earlier in the body.

**After:**
> "Same kind of gap you are dealing with right now. Want the numbers behind it?"

No call, no time commitment, a concrete offer to send something real, and "gap" now has an actual antecedent two sentences earlier where the review-count comparison is stated explicitly.

**The competitor consequence, before:**
> "...they are booking those enquiries in."

A flat fact, not a felt consequence.

**After:**
> "Every time that happens, [Top Competitor] picks up, banks the five-star, climbs past you on Google, and shows up first the next time someone in [City] searches for a [trade] installer, not you."

The full Competitor Compounding Flywheel completed in one sentence: the missed call becomes their booking, becomes their review, becomes their ranking, becomes them showing up first next time.

**The unbacked premise, before:**
> "Every time that happens while {{chainScene}}..."

This reads as grounded because "that happens" correctly points back at a real, quoted complaint two sentences earlier. Checked against all 15 real leads on this route, zero of the actual complaints connected to being physically busy on a job, they were about missed appointments, late quotes, workmanship, follow-up. The trade-scene clause was an independent, unbacked assumption riding along inside a sentence that only half-earned its credibility.

**After:** `while {{chainScene}}` dropped entirely. "Every time that happens" carries the reference correctly on its own, with nothing extra claimed.

**Measured result across all 15 real leads on this route:** 108-115 words before the final fix, 102-111 after. Reading grade 7.4-9.1 before, 7.1-8.9 after, still the highest of the five templates, a known, unclosed gap (see the reading-grade note below).

## Template A-v2: positive reviews, no complaint, competitor gap

**The missing-verb bug, present from before the redesign and carried straight through it:**
> "Your customers: '[bestReviewQuote].'"

A label and a colon, not a sentence. No verb, no indication of who's speaking. This wasn't caught by the redesign's own checklist, which was checking word count, guarantee language and the complaint-mismatch pattern, not basic sentence completeness on a line that wasn't the specific target of any of those checks. It was found only by reading the finished copy end to end after it had already shipped.

**After:**
> "Your customers put it best: '[bestReviewQuote].'"

Matches the verb-based pattern already used correctly elsewhere in the sequence ("your customers say why," "your reviews mention it").

**Word count, the most dramatic single change of any template:** 135-147 words before, against a 115-word ceiling, the worst overrun found in the whole redesign. 107-118 after the structural fixes. 100-111 in the final version, after splitting a sentence that was landing two numbers back to back ("...to your 33, 29 enquiries...") into two shorter sentences, which also dropped the reading grade from 8+ down to 5.0-7.7 as a side effect, not a deliberate rewrite for readability.

## Template A-lite: competitor gap only, no review data at all

**Structural order, found after shipping, by a plain read-through rather than any checklist:** the first version named the competitor and their review count before ever mentioning the reader's own company, the opposite order of every other working template. A cold reader has no idea yet why a stranger's review count matters, because the comparison itself was never made explicit, there was no "to your [number]" anchor anywhere near it.

**Fixed** by reordering to match the proven skeleton every other template already used: the reader's own company first, then the competitor, immediately anchored with "to your [reviewCount]," a field that had been sent with every lead's data the whole time but never actually used in this template until the reorder.

**Trade-word repetition, the worst case found across all five templates:** the original ran the trade word three to four times per email, `{{tradeScene}}` sitting directly after `{{tradeShort}}`, then repeated again in the case-study line. Reduced to two mentions per email using the de-duplicated scene variable.

**Word count:** 149-157 words before, the single worst overrun of any template in the redesign, worse even than A-v2's. 91-96 after every fix, comfortably inside target.

## Template B: complaint only, gap too small to name a competitor credibly

**The complaint-mismatch bug, worse here than on any other template:** the original line asserted "the phone rings and you miss it" regardless of what the real complaint actually said. Checked against all 9 real leads on this route: zero of nine complaints were actually about a missed call, a 100% mismatch rate, worse than Template A's 68%.

**After:**
> "Every time that happens, it becomes someone else's booking, someone else's five-star, and the reason the next customer in [City] finds them first, not you."

Points back at whatever the complaint quote actually said, rather than asserting an independent, usually-wrong cause.

**The call-to-action's own evolution:** started as a direct call-and-time ask, moved through an intermediate offer-based phrasing that read slightly awkward ("worth a quick call to see what that could look like fixed?", not natural English), and settled on:
> "Same thing you are dealing with right now. Want to know exactly what I'd fix first?"

**Word count:** roughly 150-155 words before, against the 115 ceiling. 87-97 after the full fix sequence, including dropping an unbacked trade-scene clause that suffered the identical Fix #8 problem Template A had, found in the same audit pass once the pattern was known to look for.

## Template D: the fallback, and the clearest evidence that a fix can look right and still be wrong

D has the least data of any template, no complaint, no competitor, no quote, which makes its call-to-action the hardest one to write honestly: there's nothing concrete to offer without inventing detail. It went through five distinct versions before landing on one that was actually correct, and the middle three are worth showing because each one fixed the previous version's problem while introducing a new one.

**Version 1, the original:**
> "Can I give you a ring at 3:30pm today or noon tomorrow? 15 minutes"

A direct call-and-time ask, removed for the same reason every template's original CTA was: friction on a cold first touch.

**Version 2:**
> "Worth a look?"

Removes the call ask, but implies something visual or documented exists to look at. D has nothing visual or data-based by design, since that's precisely why it sits in the lowest offer tier.

**Version 3, the first attempted fix, and the one that shipped live before being caught:**
> "Worth fixing?"

The reasoning at the time was that this was more honest, since nothing visual exists to reference. What that reasoning missed: "worth fixing?" literally asks "should I fix this for you?", an offer of service, not information. That's the exact sales-trope pattern the guarantee-language fix had already removed from every other template. An overcorrection that traded a minor, arguable problem for a real one, and it went out live before it was caught.

**Version 4:**
> "Sound familiar?"

Fixes the offering-a-service problem, since it proposes no action. But combined with the line before it ("Same thing you are dealing with right now"), it asks the reader to confirm something that was never actually named, two vague references stacked on an unbacked claim.

**Version 5, the one that actually worked, rebuilt from the ground up around a formula already documented in the playbook but never yet used by any template:**
> "Is that happening at your end, or have you got after-hours covered?"

State the general truth, then ask whether it applies, and leave a genuine way to answer either way. Names the subject (after-hours) explicitly, so nothing is left for the reader to infer, and the second half makes it non-presumptuous rather than asking for agreement on something unstated.

**The premise underneath the CTA had the same journey.** The original body asserted:
> "The enquiries that come in while {{chainScene}} do not wait."

A claim that the reader personally misses enquiries while out on a job. Templates A and B can make an adjacent claim safely because it's anchored to a real, quoted complaint. D has no complaint to anchor anything to, so this was pure assumption, and a weak one: most small firms have someone in the office or a van who answers. The first reaction is "no we don't," and the email is dead before the call-to-action is even read.

**Fixed** with an after-hours framing instead, a claim about the enquiries themselves rather than about the reader's behaviour:
> "The enquiries that land in the evening or over the weekend do not wait."

An enquiry landing at 8pm on a Saturday genuinely does sit there for essentially every small installer. Nothing to dispute, because nothing is being asserted about what the reader personally does all day.

**Word count:** naturally the shortest of the five at 79-86 words, correctly reflecting that D has the least data available, not a template that fell short of a target.

## The pattern across all five

Every template's hardest bug was the same shape: a sentence that read as grounded because it sat next to something real, when only part of the sentence was actually earning that credibility. The fix was never a rewrite from scratch, it was identifying exactly which clause was doing unlicensed work and either removing it or replacing it with a claim that needed no specific evidence to be true. And in every single case, the bug was caught by reading the finished copy as an actual recipient would, not by any structured checklist pass, which is why that plain read-through is now a mandatory final step on every new template rather than an occasional extra check.
