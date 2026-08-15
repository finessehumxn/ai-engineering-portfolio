# Grounded scoring instead of a confident number

**System:** MCGrantz — a grant winnability engine
**Stack:** FastAPI, Postgres, Anthropic Claude, Railway
**Status:** Live in production at grants.millennialscreatives.com

---

## The problem

Grant discovery tools answer *"what grants exist?"* That is the easy question and it is not the one
that costs anybody anything. The expensive question is *"which of these can I actually win?"*, and
getting it wrong means a small nonprofit spends three weeks on an application it was never in the
running for.

An LLM will happily answer the expensive question. It will produce a plausible score with a
plausible explanation, and there is no way for the user to tell a grounded 70 from a hallucinated
one.

## Constraints that actually drove the design

1. **No invented opportunities, deadlines, or amounts.** Every fact traceable to a public source.
2. **A score must be arguable.** If a user disagrees, they need to see what produced it.
3. **Discouraging answers must survive.** A tool that tells everyone they have a decent shot is
   useless, and politeness is the failure mode people ship by accident.

## Architecture

Scoring is retrieval over public record, not a model judgment dressed up with a number.

- **Federal opportunities** come from Grants.gov, then each is cross-referenced against
  **USASpending.gov award history for its exact CFDA program**: how many awards, typical size, and
  crucially *what kind of organisation actually won them.*
- **Private foundations** come from **IRS Form 990-PF filings**, which record who each foundation
  paid, how much, and for what. Indexed and refreshed as new filing years land.

The score compares the applicant against the actual winners of that specific programme. A $250k
nonprofit looking at an NIH R01 whose past recipients are UCLA, Johns Hopkins and Harvard receives
**11 out of 100, "Long shot"** — not a polite 70.

Every score expands into its contributing factors, so the user can argue with it.

## Decisions I would defend in review

**Retrieval decides, the model explains.** The number comes from comparing the applicant to a
distribution of real winners. The model's job is to render that comparison in language, not to form
the judgment. This is the decision that makes the whole product defensible: the failure mode of a
language model is fluent invention, so it is kept away from the part that must be true.

**Comparing against programme-specific winners, not generic eligibility.** Eligibility rules say who
*may* apply. Award history says who *does* win. Those are very different distributions, and the gap
between them is the entire value of the product.

**Letting the score be brutal.** An 11 is more useful than a 70 and much harder to ship, because
every instinct in product design pushes toward encouraging the user. The specific example above is
in the product documentation precisely so the calibration is a stated commitment rather than a
tuning knob someone softens later.

**Foundation matching turns a cold letter into a targeted ask.** Not "please fund youth mental
health" but "you gave a named clinic $25,000 for exactly this last year." That is a retrieval
result, not a generated one, which is why it can be put in front of a programme officer.

## How I know it works

**What is verified:** every score is decomposable into its factors, and the factors are traceable
to a source row. Claims that cannot be traced are not displayed. The system is live and running
against refreshed filing data.

**What is not yet where I want it:** I do not have a held-out set of applications with known
outcomes to calibrate against. The scoring is grounded in real award distributions, which is far
stronger than a model guess, but "grounded" and "calibrated" are not the same claim and I do not
make the second one. Proper calibration needs outcome data that only accumulates with users.

## What I would do differently

Instrument disagreement from day one. The most valuable evaluation signal available is a user
looking at an 11 and applying anyway, then winning. I built the factor breakdown so scores could be
argued with, but did not initially capture the argument. That feedback is the calibration set, and
it is only collectable in the moment.
