# Project 9: AI Risk and Governance Review

## Build

Build a review package for one AI system that documents threats, injection tests, privacy controls, red-team results, product economics, and launch decision.

This project belongs after Part 9. It proves that learner can evaluate whether an AI system is secure, governed, economically justified, and product-real.

## User

Engineering lead, product owner, security reviewer, or founder deciding whether to ship, revise, pause, buy, or stop an AI system.

## Product Outcome

Reviewer can inspect:

- security threat model;
- prompt injection and retrieval poisoning tests;
- privacy and retention policy;
- access control design;
- responsible AI risk register;
- product economics;
- build-vs-buy decision;
- final launch recommendation.

## System to Review

Choose one:

- Multimodal Document Analyst;
- Grounded Research Assistant;
- Operations Action Assistant;
- Production AI Control Center;
- capstone system.

## Core Features

### 1. Threat Model

Map:

- assets;
- users;
- data sources;
- tools/actions;
- trust boundaries;
- controls.

### 2. Attack Test Suite

Test:

- prompt injection;
- malicious retrieved document;
- retrieval poisoning;
- unauthorized tool request;
- sensitive data exfiltration;
- unsafe browser/tool action.

### 3. Privacy Review

Document:

- PII fields;
- retention;
- deletion;
- redaction;
- access roles;
- trace visibility.

### 4. Responsible AI Review

Track:

- affected users;
- known harms;
- red-team results;
- human review points;
- incident owner;
- review cadence.

### 5. Product Economics

Compare:

- build;
- buy;
- do nothing.

Estimate:

- build cost;
- run cost;
- support cost;
- risk cost;
- expected savings;
- stop condition.

### 6. Decision Memo

Output:

```text
Decision: ship / pilot / revise / pause / stop / buy
Reason:
Blocking risks:
Required controls:
Owner:
Review date:
```

## Architecture

```text
system description
-> threat model
-> red-team tests
-> privacy/access review
-> economics model
-> decision memo
```

## Evaluation

Use seeded issues:

- injected document;
- forbidden data source;
- risky tool request;
- raw PII in logs;
- uneconomic feature;
- vague product claim.

Acceptance:

- review catches seeded risks;
- decision memo names blockers;
- product claims match capability;
- stop/buy is allowed outcome.

## What Learner Understands

- Security follows attack surface.
- Governance means ownership and decision rights.
- Product economics prevents impressive waste.
- Responsible shipping may mean stopping.

## Extension

Add:

- risk dashboard;
- policy checklist;
- incident report generator;
- cost calculator;
- vendor comparison table;
- recurring review schedule.
