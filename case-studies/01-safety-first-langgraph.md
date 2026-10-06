# A safety pipeline that runs before generation

**System:** HumxnMed — a patient health-briefing assistant
**Stack:** LangGraph, FastAPI, Anthropic Claude, Railway, Capacitor
**Status:** Deployed. Shipped to the App Store as a free core release.

---

## The problem

A patient types *"my blood sugar is high and my feet tingle."*

A general-purpose assistant will answer that helpfully and fluently. It will also answer
*"I've been thinking about ending things"* helpfully and fluently, in the same voice, with the same
confidence, because to the model both are just text.

The hard part of healthcare AI is not producing a good briefing. It is recognising the small
fraction of inputs where producing a briefing at all is the wrong action.

## Constraints that actually drove the design

1. **The system must never present itself as a doctor.** Not a tone preference. A liability line.
2. **Emergencies and crisis signals must be caught before generation**, not filtered afterwards.
   Post-hoc filtering means the unsafe text existed, and anything that exists can leak.
3. **The patient must confirm what is being looked up** before it is looked up. Ambiguous symptom
   descriptions are the norm, not the exception.
4. **Crisis resources are never paywalled.** No monetisation decision is allowed to touch that path.

## Architecture

A LangGraph `StateGraph` over a shared `PatientState`, composed of five purpose-built nodes.

```
raw input
   │
   ▼
┌──────────────┐   emergency / crisis / off_topic / invalid
│ 1. guardrail │ ─────────────────────────────────────────►  safe exit
└──────────────┘
   │ pass
   ▼
   interpretation  ──►  [INTERRUPT: patient confirms]  ──►  retrieval  ──►  briefing
```

The guardrail is a **routing** node, not a scoring node. It emits a class, and four of the five
classes terminate the graph before any generation happens. That is the single most important
property of the design: on the unsafe paths, the briefing model is never called.

The interrupt is a real LangGraph checkpoint. The graph halts, returns to the caller, and resumes
from persisted state after the patient confirms. It is not a UI confirmation dialog wrapped around
an already-completed pipeline.

## Decisions I would defend in review

**Guardrail as a separate node rather than a system prompt.** A system prompt asking the model to
be careful is a request. A routing node that terminates the graph is a control-flow guarantee. When
the failure mode is someone in crisis receiving a cheerful symptom explainer, I want the guarantee.

**Interrupt before retrieval, not after.** Confirming interpretation *after* retrieving is cheaper
to build and means the patient confirms a lookup that already happened. Interrupting first costs a
round trip and makes the confirmation meaningful.

**Five narrow nodes instead of one capable agent.** A single agent with tools would be less code
and much harder to reason about. With discrete nodes I can state plainly which paths can reach the
generation step. That property is worth the extra structure.

**Marketing claims held to the shipped defaults.** Whatever the product page says about what data
goes where has to stay literally true against the build in the store. When the design changed so
that a patient's saved health context is sent with each question to give better answers, the data
policy and trust pages were rewritten in the same change. I treat that copy as part of the system,
not as decoration around it.

## How I know it works

**What is verified:** the guardrail classes are exercised directly, and the four terminating classes
are asserted to never reach generation. That is the property that matters most, so it is the one
under test.

**What is not yet where I want it:** I do not have a large labelled corpus of crisis-adjacent inputs
with adjudicated ground truth. The classifier is evaluated against a hand-built set, which is honest
but small, and small evaluation sets flatter the system. Building that corpus is what
[the evaluation work](03-llm-failure-evaluation.md) exists to feed.

I would rather state that plainly than publish a precision figure derived from an evaluation set I
wrote myself to match the behaviour I already had.

## What I would do differently

Version the guardrail taxonomy from the start. The five classes were right, but I changed their
boundaries twice, and without versioning I could not cleanly compare behaviour before and after.
Any classifier that gates a safety path needs its label set under version control with the
evaluation attached, so a change is a measurable diff and not a vibe.
