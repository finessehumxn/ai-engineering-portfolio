# AI Engineering Portfolio

**I build AI systems for domains where a wrong answer costs someone money, time, or safety.
So the failure mode is where the design starts, not where it gets patched.**

Healthcare, mental health, government contracting, and regulatory compliance. In all four, a
confidently wrong model output is worse than no output at all. That constraint shapes every
architecture decision in the case studies below.

---

## Case studies

| | What it demonstrates |
|---|---|
| **[A safety pipeline that runs before generation](case-studies/01-safety-first-langgraph.md)** | LangGraph `StateGraph`, guardrail-first routing, human-in-the-loop interrupt and resume |
| **[Grounded scoring instead of a confident number](case-studies/02-grounded-scoring.md)** | Retrieval over federal award history, calibration against outcomes, every score defensible |
| **[Evaluating what a model does, not what it knows](case-studies/03-llm-failure-evaluation.md)** | Behavioural evaluation on ambiguous and crisis-adjacent inputs |
| **[Local-first architecture as a privacy guarantee](case-studies/04-local-first-compliance.md)** | Regulated data that never leaves the browser, 304 tests, citation-backed rules |

Each one follows the same structure: the problem, the constraints that actually drove the design,
the architecture, the decisions I would defend in review, **how I know whether it works**, and what
I would do differently.

---

## How to read this

Most of the source is private, because these are commercial products and client work rather than
side projects. What is public:

- **[emosafe-ai](https://github.com/finessehumxn/emosafe-ai)** — LLM behaviour observation on
  emotionally sensitive prompts
- **[ai-failure-analysis](https://github.com/finessehumxn/ai-failure-analysis)** — structured
  evaluation of model behaviour at the messy-input boundary

The case studies here describe systems that are deployed and in use. I have written them so that
the reasoning is auditable even where the code is not, because at this level the reasoning is the
thing being hired.

---

## Where I am honest about limits

Several of these systems have thinner evaluation than I would like, and I say so in each case study
rather than implying a rigour that is not there. One scoring layer is explicitly modelled rather
than empirical and is labelled that way in the product itself. I would rather be caught being
accurate than caught overstating.

---

## Stack

Python, FastAPI, LangGraph, Anthropic Claude · JavaScript, React, Vite, Node · Postgres, Supabase ·
Railway, Netlify, Vercel · Playwright, `node --test`, pytest

**Contact:** finessehumxn@gmail.com
