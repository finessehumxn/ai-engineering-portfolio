# Evaluating what a model does, not what it knows

**Systems:** [emosafe-ai](https://github.com/finessehumxn/emosafe-ai) ·
[ai-failure-analysis](https://github.com/finessehumxn/ai-failure-analysis)
**Stack:** Python, Jupyter, Anthropic Claude
**Status:** Public. Active research, feeding the guardrail work in
[case study 01](01-safety-first-langgraph.md).

---

## The problem

Standard evaluation measures what a model **knows**, on clean, well-formed inputs, against a
benchmark with a right answer.

Production systems in mental health and healthcare receive inputs that are ambiguous, emotionally
loaded, indirect, and structurally atypical. The user in the most distress is frequently the one
whose message is least well-formed. Benchmark accuracy says nothing about that boundary.

The question is not *can the model get it right on a benchmark?* It is *what does the model do when
the input is messy and the cost of a wrong answer is human?*

## What I am actually measuring

Behaviour, not correctness. Specifically, how model responses shift across:

- **prompt type** — direct statement, indirect allusion, third-party framing ("my friend said...")
- **emotional register** — flat, escalated, minimising, joking-about-serious-things
- **model configuration** — how much the behaviour moves with settings that are supposed to be
  orthogonal to safety

The output is a structured record of where alignment between model output and safe, appropriate
response breaks down. Not a leaderboard number.

## Why this feeds the product work

The guardrail node in HumxnMed routes on five classes, and every one of them is a claim about model
and system behaviour on exactly the inputs this project characterises. Building the classifier
without this work would mean choosing thresholds by intuition.

The honest ordering is that the product shipped first and the evaluation is catching up. I would
rather say that than imply the research preceded the system.

## Decisions I would defend in review

**Observation before scoring.** The first output is a documented record of behaviour, not a metric.
Metrics defined before understanding the failure surface measure the wrong thing confidently, and
then get optimised.

**Indirect and third-party framings treated as first-class.** "My friend has been feeling like this"
is one of the most common ways a person raises their own crisis. A test set built from direct
statements will report a safety rate that does not survive contact with users.

**Emotional register varied independently of content.** The same underlying content delivered flat
versus escalated should not produce categorically different safety handling. Where it does, that is
the finding.

## How I know it works, and where it falls short

**Being straight about this:** this is early-stage research, and the repositories reflect that. The
framing and methodology are sound and the READMEs describe the intended shape of the work
accurately, but the implementation is one notebook and a small harness, not a completed study.

Anyone evaluating me on this should read it as evidence of how I think about evaluation, not as a
finished result. I have deliberately not padded it to look like more than it is.

## What comes next

The concrete gap is a labelled corpus with adjudicated ground truth on crisis-adjacent inputs,
sized properly, with inter-rater agreement recorded. That is the artefact that would let me state a
precision figure for the HumxnMed guardrail and defend it. It is the single highest-value piece of
work outstanding across everything in this portfolio.
