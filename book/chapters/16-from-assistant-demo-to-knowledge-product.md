# Chapter 16: From Assistant Demo to Knowledge Product

## Real Problem

Your RAG assistant works in prototype.

Users like it. But after two weeks:

- docs change;
- answers become stale;
- users ask who approved sources;
- managers ask usage metrics;
- security asks about permissions;
- support asks how to fix bad answer;
- nobody owns content quality.

Prototype answered questions. Product must stay trustworthy.

## Bad Default Solution

Bad product design:

- chat UI only;
- no source admin;
- no feedback loop;
- no eval dashboard;
- no content freshness process;
- no permission model;
- no review queue;
- no owner.

This becomes abandoned demo.

## First Principles

A knowledge product is maintained system for turning organizational knowledge into reliable answers.

It needs:

- source lifecycle;
- access control;
- retrieval quality;
- answer evaluation;
- user feedback;
- admin review;
- analytics;
- update process;
- trust signals.

Chat is interface, not product.

## System Decision

Design beyond answer box:

```text
Source owner:
Document ingestion:
Approval workflow:
Freshness policy:
Access rules:
Feedback collection:
Bad-answer review:
Eval schedule:
Usage analytics:
Improvement process:
```

User-facing trust signals:

- citations;
- source title;
- last updated;
- confidence/limitation;
- refusal when unsupported;
- "report issue" path.

Admin-facing controls:

- source list;
- stale docs;
- failed evals;
- top unanswered questions;
- bad feedback queue;
- permission audit.

## Minimal Build

Create `knowledge-product-spec.md`:

```text
Product:
Primary users:
Primary workflows:

Sources:
- source:
  owner:
  approval:
  update frequency:
  access:

Trust signals:
- 

Admin controls:
- 

Feedback loop:
- 

Eval loop:
- 

Analytics:
- 

Risk controls:
- 
```

## Failure Modes

Knowledge product fails when:

- nobody owns source quality;
- stale docs remain indexed;
- feedback never reaches builder;
- evals run once then stop;
- users cannot inspect sources;
- permissions are checked only after retrieval;
- product hides uncertainty;
- bad answers are not turned into test cases.

## Evaluation

Measure product health:

- active users;
- successful answer rate;
- citation click rate;
- bad feedback rate;
- unanswered question rate;
- stale source count;
- eval pass rate;
- permission violation count;
- median time to fix bad answer.

Most important operational metric:

> bad answer -> reviewed -> fixed -> added to eval set

If this loop does not exist, system does not learn.

## Improvement

Improve product by:

- assigning source owners;
- adding stale-source alerts;
- creating feedback triage;
- making failed answers eval cases;
- exposing citations;
- adding admin review UI;
- scheduling recurring evals;
- removing low-quality sources.

## Proof Artifact

Create `knowledge-product-spec.md`.

Include:

- source lifecycle;
- permission model;
- user trust signals;
- admin controls;
- feedback loop;
- eval loop;
- analytics;
- risk controls.

## Decision Checklist

- [ ] Source owners are named.
- [ ] Freshness process exists.
- [ ] Permissions are designed.
- [ ] Feedback loop exists.
- [ ] Bad answers become evals.
- [ ] Admin review exists.
- [ ] Trust signals visible to user.

## Active Recall

1. Why is chat UI not full product?
2. What source lifecycle does knowledge product need?
3. Why should bad answers become eval cases?
4. What trust signals should user see?
5. What admin controls should exist?

## Next

Part 5 moves from answering questions to taking action. Grounded knowledge becomes more powerful when connected to tools, workflows, state, and approval.
