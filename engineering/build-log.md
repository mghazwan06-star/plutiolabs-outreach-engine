# Build Log: Findings, Root Causes, Fixes

**5 critical, 9 major, 10 minor, 13 architectural decisions, logged across three named pipeline revisions (v2 to v3 to v4) between May and September 2026.** Same discipline as the sister repository's bug taxonomy, applied to a different system: every finding kept, with its root cause and its fix, not just the ones that make the project look good.

Severity follows the same scale used throughout. Critical blocks launch, compliance or spends real money. Major means wrong data would have reached a real inbox. Minor was fixed inline without wider consequence. Decision is an architectural choice worth recording so it can be revisited deliberately rather than rediscovered.

---

## The fabricated social proof

**Severity: critical. Caught before a single email shipped.**

`buildLinkedInMessage()` contained hardcoded placeholder figures: invented job counts, invented timeframes, invented city names, presented as real client results. They weren't drafted as obviously-fake scaffolding. They read as finished copy, sitting in code rather than only in a template document, which is what made them dangerous. Sending fabricated results to UK trade businesses is a misleading-advertising problem under UK consumer protection law and a reputational one in a small, connected trade where installers talk to each other.

**Fix.** LinkedIn message generation was removed from the pipeline entirely rather than patched. Its replacement (a WhatsApp follow-up) was designed from the outset to make no performance claims at all. The same figures were also present in the email template copy itself. That was logged as a hard launch blocker with three resolution paths (a real case study, a rewrite without any proof claim, or third-party sourced statistics), and no template shipped until the fabricated numbers were gone. The playbook that documents the framework behind these templates, [`../outreach/playbook.md`](../outreach/playbook.md), is written without the specific invented examples, for the same reason they were pulled from the workflow.

**Why it wasn't caught sooner.** The figures had migrated from a documentation file's "replace with real data" placeholder into the actual code path. Something explicitly marked as temporary stops being read as temporary once it's been sitting untouched long enough. The fix is procedural as much as technical: nothing labeled "placeholder" gets built into a code path with an execution date near it.

---

## Class 1: memory and scale

### The single-workflow version couldn't survive its own dataset

n8n retains every node's input and output for the life of an execution. A pipeline of roughly twenty sequential nodes over 5,603 leads holds on the order of twenty full copies of the dataset in memory at once. The first full production run didn't just fail. It saturated the entire n8n Cloud instance, every API endpoint returning 503, not just the one workflow's.

**Fix.** Split into a parent/child architecture. The parent orchestrates batches and holds only counts. The child does the expensive per-batch work and releases its memory the moment it returns. Full design rationale in [`../pipeline/architecture.md`](../pipeline/architecture.md).

### The competitor index quietly degraded twice, through two different doors

Splitting into batches solves the memory problem and creates a new one: a competitor-matching index built from a 500-lead batch, rather than the full 5,603-lead pool, finds a usable match for 48.6% of eligible leads instead of close to 100%. This was measured and designed around once, with a global index built before batching starts and shared across every batch call. It then broke a second time through an entirely different mechanism: a `maxLeads` slicing parameter, added later for testing convenience, was applied before the index build rather than after, silently shrinking the index back down to the size of whatever slice was being tested. Because the bug lived upstream of a live write path, it actually overwrote 25 real CRM rows with degraded routing before it was caught.

**The rule this produced.** A node that exists specifically to protect a global computation has to be fed the global set. Checking that it ran is not the same as checking what actually reached it. Verify the size of what a global-scope node actually received (`_indexStats.indexSize`), not just its output status.

---

## Class 2: silent write failures

### The write key went through three failed attempts before it was actually safe

`appendOrUpdate` matched on phone number looks idempotent on paper. It failed in three separate, compounding ways, found only because each fix was independently re-verified against a full CRM integrity check rather than assumed correct once applied.

First, a leading `+` on the phone number was silently stripped by Google Sheets' write semantics, so the match key on every re-run pointed at data that no longer existed, and every re-run appended a full duplicate of the dataset rather than updating it.

Second, fixing the format exposed the deeper problem: phone number was never a safe key. Some records legitimately share a phone number, some have none at all. A shared or blank key doesn't just duplicate rows, it can silently overwrite one business's row with another's data, confirmed in production when one real, Priority-1-eligible business's row was found missing entirely, its slot overwritten by a different company with a coincidentally blank match key.

**Fix.** A dedicated `Row Key` column, populated in priority order (certification number, then domain, then phone, then a name slug), guaranteeing a unique non-empty key regardless of source. Verified after: zero duplicates, zero missing rows, and, for the first time, genuinely idempotent re-runs.

Full detail on the missing-row incident and its recovery in [`../pipeline/crm-schema.md`](../pipeline/crm-schema.md).

### Disabled nodes in n8n pass their input straight through

Disabling a node's function was assumed to sever it from the flow. It doesn't. n8n treats a disabled node as transparent, so a disabled fetch step still delivered its upstream input (the config object itself) into a downstream enrichment node as a 26th, malformed "lead." The fix generalises: disabling a node is never enough to remove it from a data path. The connection itself has to go.

### A re-run without Claude silently blanks existing enrichment

A step responsible for preparing the Claude prompt unconditionally clears five enrichment fields on every lead it touches, then only refills the ones it actually calls the API for. Combined with a spend cap on Claude calls, this means any partial re-run, one that reprocesses more rows than the cap allows to be re-analysed, erases previously-written complaint and quote data on every row past the cap, silently, because the write path treats "field not returned this run" the same as "field genuinely empty." Logged as an open hazard with a documented operational workaround (never re-run a routing-only pass without also re-analysing the affected rows) rather than left undocumented, because the proper fix, preserve the existing sheet value when the incoming field is empty, touches the same write path the Row Key fix above had just stabilised, and doing both at once risked reintroducing what had just been fixed.

---

## Class 3: infrastructure that reports success while doing nothing

### Code nodes cannot make HTTP calls on this n8n version, and nothing said so

Every external API call in the enrichment pipeline was written inside Code nodes using a helper function that doesn't exist in this n8n version's Code node sandbox. Every one of those calls threw immediately, and was silently swallowed by an error-handling setting configured to keep the workflow running regardless (`onError: continueRegularOutput`), so the pipeline reported success while doing none of the actual enrichment work it claimed to.

This survived multiple audit passes because the validator had flagged the exact pattern as a warning, filed away as a known false positive on the reasoning that an earlier version of the pipeline used the same approach and "always worked." On investigation, the earlier version had never actually been run with real API keys either. Two workflows shared one untested assumption, and "it has always been written this way" got mistaken for "it has been proven to work."

What actually gave it away: runtime. A workflow claiming to have completed real external API polling in 17 seconds hadn't. Genuine Apify polling cannot finish that fast. Duration, not status, was the tell.

**Fix.** Every Code-node HTTP call was converted to a proper HTTP Request node behind a small shared sub-workflow that handles the start/poll/fetch cycle for asynchronous jobs. The rule this produced, stated plainly because it generalises beyond this one bug: a node that reports success in less time than its external dependency physically requires has not actually run. And never dismiss a validator warning on the grounds that another workflow shares the same pattern. Shared code is shared risk, not evidence the risk never materialised.

### API keys were being written into plaintext execution logs on every run

Configuration was passed as a plain data object into every child workflow call, and n8n stores execution input data by default. Once real API keys populated that configuration, every one of the pipeline's dozen-plus batch executions would have stored those keys in plaintext, readable by anyone with access to the execution history.

**Fix.** Moved to n8n's native credential system, which never appears in stored execution data. As a side effect of restructuring two separate Claude calls onto this new pattern, they got combined into a single call per lead, halving the estimated Claude cost for a full run.

### The review-text fetch was never actually built

A downstream node was configured to skip fetching review text, returning only review counts, meaning the two fields that drive the strongest email templates (a real complaint, a real positive quote) could never populate for any lead, no matter how well every other stage of the pipeline performed. This sat unnoticed underneath an entirely separate, correctly-designed review-sort fix (see [`../pipeline/dual-source-design.md`](../pipeline/dual-source-design.md)). The sort order was right, there was just nothing being sorted. Once fixed and verified against a real test batch, the top-tier, complaint-driven template went from zero eligible leads to double digits in a single test run.

---

## Class 4: reference and documentation drift

The written pipeline specification described competitor matching as reading "the top three nearby places returned by Apify." The deployed code had never done that. It grouped by city string and took a max-review match, a materially different and weaker approach. The documentation was corrected to match the deployed behaviour rather than the reverse, and the actual matching logic was independently rewritten (see [`../pipeline/dual-source-design.md`](../pipeline/dual-source-design.md)) once the gap was found. A second, smaller instance of the same pattern: a doc claiming lead and competitor reviews were batched into a single API call "to halve cost." The code had never sent competitor reviews to that call at all.

**The rule.** Documentation drift and code bugs are the same failure mode wearing different clothes, and the fix is the same either way. Check the deployed artefact, correct whichever side is wrong, and don't assume the newer-looking document is the accurate one.

---

## Class 5: building the outreach connection

### Verifying the API contract before writing a single node

The original outreach handoff was manual: filter the CRM by template route, export a CSV, upload it to the sending platform by hand, once per route. Replacing that with a direct API push started with reading the sending platform's current API documentation, not assuming the shape of the request from memory or an earlier integration. This is a direct application of the standing rule from the `$helpers.httpRequest` incident above: an assumption about how an external API behaves is exactly the kind of thing that silently breaks a pipeline, and it costs almost nothing to check first.

The confirmed contract: a single endpoint accepting up to 1,000 leads per request, each with an email, name, company and a set of custom variables that have to match the CRM's own column headers exactly, case and space sensitive, per the field map the routing logic depends on.

### Three independent safety layers, because a duplicate send is a trust problem, not just a data problem

Every push carries three separate protections rather than relying on any single one:

1. **Status-gated selection.** Only rows marked as not-yet-pushed are eligible for a given run, and a successful push immediately marks them as queued, so a second run can't select the same rows again even if triggered before the first run's results are reviewed.
2. **Server-side dedupe**, using the sending platform's own duplicate protection against everything already in the account, independent of whatever the CRM's own status field says.
3. **Explicit double confirmation.** Nothing writes or sends without both an explicit apply flag and a real target campaign ID. A dry run, which selects and reports what it would do without doing it, is the default behaviour, not an opt-in safety net.

A dry run against a real segment of the CRM confirmed the selection logic worked correctly before the first live push: every field a template needs (competitor name, review counts, the computed review gap, city, trade, trade scene) came through populated, and fields that should be correctly empty for that route (a complaint field on a gap-only route) were correctly empty rather than silently defaulting to something misleading.

### The brand-name sweep, done as a single verified pass

A business rename partway through this build required scrubbing the old name out of every document, template and piece of config it had spread into, case-sensitively, since the old lowercase form was also load-bearing infrastructure (part of a live hosting subdomain) and had to survive the sweep untouched while every other occurrence changed. Done as one pass across every affected file rather than incrementally, then verified with a direct search for the old brand string across every workflow node afterward. Zero occurrences remained. The kind of mechanical, whole-codebase sweep this generalises to is the same technique behind the cross-workflow consistency audits described in the sister repository: find one instance, then check everywhere the same pattern could exist, rather than fixing the one instance and moving on.

---

## Class 6: fixes to a shared downstream template don't propagate on their own

Five email templates feed into a shared pool of follow-up templates further down each sequence: three of them share one hours/gap-compounding Email 2 variant, two share a universal response-time Email 2 variant, and all five share the same break-up email. Fixing a problem in an Email 1 template doesn't touch any of those shared downstream templates, even when the fix changes something the downstream template also assumes.

This surfaced as three separate, logged instances once the downstream templates were checked against fixes already applied upstream, documented in full in [`../outreach/sequence-and-cadence.md`](../outreach/sequence-and-cadence.md): a shared follow-up still using call-specific framing after the complaint-driven templates it follows moved away from asserting a call-specific cause, a shared universal template still using an un-deduplicated trade-scene phrase after Email 1 switched to a deduplicated one, and a shared break-up email whose reference to "inbound calls" contradicted the same complaint-driven fix from a different angle. None of these produce a false claim about any individual lead. All three are internal inconsistencies within a single lead's own four-email conversation, invisible unless that conversation gets read end to end rather than template by template.

**Why this is logged as a distinct class rather than folded into the cross-template fixes above.** The eight cross-template fixes are about a defect appearing independently in multiple sibling templates built from the same original skeleton. This is a different mechanism: one template correctly fixed, and a second, dependent template left stale because fixing a shared template requires verifying the change holds for every route that depends on it, not just the route that prompted the fix. A template shared by three routes with different upstream copy can't be patched unilaterally by whichever route happened to surface the problem first, it needs a decision that's checked against all three before it ships.

**Standing rule this produces:** a fix to any template that other templates reference downstream, whether by shared logic or shared copy, is not complete until every downstream consumer of that template has been re-checked against the change, not just re-validated on its own terms. Fixing the source and leaving the propagation for later is a legitimate scoping decision, but it has to be a decision made on purpose and logged, not a gap that goes unnoticed because the audit that found the original bug considered its job finished once that one template was fixed.

---

## Verification, done properly, at the end

The last entries in this log aren't bug fixes. They're a template going through the same live-verification discipline as everything above, applied prospectively rather than reactively. Six real test emails were sent through the sending platform's own test-send endpoint and read as a real recipient would, not judged from stored JSON. That process caught a dangling reference with no clear antecedent, a generic guarantee line that reads as a trope to a skeptical, agency-burned audience, and, stress-tested against all 22 real leads eligible for that template at the time rather than one hand-typed example, a text-duplication bug that only two trade categories' data could ever trigger, invisible on any single example lead.

One check from that pass is worth calling out on its own. When directly asked whether the AI-generated complaint summaries could be verified as real rather than invented, the honest answer required a fresh, small-cost re-scrape of two live businesses' current Google reviews, compared by eye against what the pipeline had stored. Both traced cleanly to real customer language. Confirming a system doesn't fabricate isn't a one-time audit. It's a question that has to stay answerable on demand, and the pipeline is built so that answering it costs a few cents and a few minutes rather than being structurally unverifiable.
