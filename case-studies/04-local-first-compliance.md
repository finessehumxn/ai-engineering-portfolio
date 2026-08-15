# Local-first architecture as a privacy guarantee

**System:** MCStanding — a data-broker compliance cockpit
**Stack:** Vite, React, JavaScript, `node --test`
**Status:** Built and tested. 205 test cases across 8 suites.

---

## The problem

Data brokers must comply with registration and deletion regimes across multiple jurisdictions: the
US states running a broker registry, plus GDPR, UK GDPR, LGPD, PIPEDA and Québec Law 25.

Complying means processing consumer suppression lists and matching them against customer records.
The data involved is precisely the data the regulation exists to protect. A compliance tool that
uploads it to a vendor server has created a second copy of the exact liability the customer is
trying to discharge, and has made itself a breach target on their behalf.

## Constraints that actually drove the design

1. **Regulated personal data must not leave the customer's machine.** Not encrypted in transit to
   us. Not sent at all.
2. **Nine jurisdictions with different rules**, and more arriving as states legislate.
3. **Every date, fee and penalty must be checkable**, because a user will act on them and their
   counsel will ask where the number came from.

## Architecture

**No backend.** The matching engine runs entirely in the browser. There is no server to send
suppression lists to, which converts a privacy policy promise into an architectural fact. "We do
not retain your data" is a claim; "there is nowhere for the data to go" is a property.

**Jurisdictions are profiles, not forks.** Each regime is a data profile in `src/lib/regimes.js`.
The matching engine underneath is jurisdiction-neutral. What differs between regimes is the
paperwork around it: who you register with, how long you have, what you may require a requester to
prove, and what vocabulary you report in. Adding a jurisdiction means adding a profile, not
branching the application.

**Citations are a schema field.** Every date, fee and penalty carries a `citation`. Where a figure
is set by agency rulemaking and could not be verified to primary source, the product says so rather
than guessing.

## Decisions I would defend in review

**Accepting the browser's limits to get the guarantee.** In-browser matching gives up server-side
scale and cross-device sync. For this data, the guarantee is worth more than the convenience, and it
is also the strongest thing the product has to sell.

**Navigation follows the obligation lifecycle, not the codebase.** Know, Prepare, Operate, Prove.
The dashboard surfaces one next action rather than nine metrics, because a compliance officer needs
to know what to do today, not how they are performing.

**Marking unverified figures as unverified.** The tempting alternative is to fill a gap with a
plausible number nobody will check. In a product whose value is that its numbers are checkable, one
invented figure discredits every honest one.

## How I know it works

**What is verified:** 205 test cases across 8 suites, covering the matching engine, suppression
handling, regime profiles, exposure calculation, evidence generation, conformance, licensing and
health checks. The jurisdiction-neutral engine is tested independently of the profiles, so a new
regime cannot silently change behaviour for an existing one.

**What is not yet where I want it:** the tests verify the engine against the rules as encoded. They
cannot verify that the encoded rules match current law, which is a research problem rather than a
testing one. The citation field exists so that a human can audit that layer, which is the honest
mitigation rather than a solution.

## What I would do differently

Build the regime profile schema first. I extracted it after the second jurisdiction, which was one
jurisdiction later than ideal. The refactor was contained because the matching logic was already
separate, but the shape of that separation would have been cleaner had I written the second regime
as a profile from the start instead of discovering the pattern by needing it.
