# Project 8: Production AI Control Center

## Build

Build a control center around one previous AI system that tracks evals, traces, costs, latency, feedback, and risks.

This project shows what separates prototype from production.

This project belongs after Part 8. It should prove that learner can operate an AI system with eval gates, traces, cost/latency controls, optimization, versioned releases, rollback decisions, and improvement loops.

## User

Builder or team lead responsible for operating an AI system safely.

## Product Outcome

User can inspect:

- eval pass rates;
- regression failures;
- request traces;
- model/prompt versions;
- cost per task;
- latency;
- user feedback;
- optimization status;
- release manifest;
- release readiness.

## System to Harden

Choose one:

- Knowledge Base Search Engine;
- Structured Intake Extractor;
- Grounded Research Assistant;
- Operations Action Assistant.

## Core Features

### 1. Eval Dashboard

Show:

- eval set version;
- system version;
- pass/fail counts;
- category breakdown;
- regression failures;
- must-pass failures.

### 2. Trace Viewer

For each request:

- request ID;
- user role;
- model/prompt version;
- retrieved sources;
- tool calls;
- validation result;
- approval status;
- final output;
- latency;
- cost.

### 3. Cost and Latency Monitor

Show:

- average cost;
- p95 latency;
- cost by feature;
- model call count;
- retrieval time;
- tool time.

### 4. Risk Register

Track:

- prompt injection tests;
- unauthorized retrieval tests;
- unsafe tool call tests;
- sensitive data leakage tests;
- owner;
- control;
- status.

### 5. Optimization Panel

Show:

- cache hit rate;
- p95 latency before/after;
- batch job duration;
- streaming enabled;
- model route cost comparison.

### 6. LLMOps Manifest

Track versions:

- prompt;
- model;
- eval set;
- data corpus;
- retrieval index;
- tool schemas;
- routing rules.

### 7. Release Gate

Before release, system must pass:

- eval threshold;
- zero must-pass failures;
- cost budget;
- latency budget;
- rollback target.

Output:

```text
Release status: Ship / Pilot / Revise / Stop
Reason:
Blocking issues:
Owner:
```

### 8. Feedback Loop

User feedback becomes:

- issue;
- failure category;
- eval case;
- fix candidate.

## Architecture

```text
AI system
-> trace logger
-> eval runner
-> metrics store
-> optimization tracker
-> release manifest
-> dashboard/control center
-> release gate
```

## Data Model

```json
{
  "run_id": "RUN-001",
  "system_version": "rag-v3",
  "eval_version": "eval-2026-01",
  "pass_rate": 0.88,
  "must_pass_failures": 1,
  "avg_cost": 0.04,
  "p95_latency_ms": 4200,
  "manifest_version": "release-2026-01",
  "release_status": "revise"
}
```

## Suggested Implementation

Build local dashboard or CLI:

```bash
run-evals
show-traces
show-costs
run-risk-tests
show-optimization
show-manifest
release-check
```

Optional web UI:

- eval tab;
- traces tab;
- costs tab;
- optimization tab;
- manifest tab;
- release tab.

## Evaluation

Use seeded failures:

- bad citation;
- invalid JSON;
- unsafe tool call;
- high latency request;
- prompt injection attempt;
- stale release manifest;
- stale source answer.

Acceptance:

- dashboard exposes each failure;
- release gate blocks unsafe version;
- feedback can become eval case;
- cost/latency visible;
- optimization impact visible;
- release gate blocks unversioned changes.

## What Learner Understands

- Production AI needs operating system around model.
- Evals and traces are product infrastructure.
- Cost/latency are design constraints.
- Risk review must be testable.
- LLMOps makes changes explainable.
- Release decision can be "stop."

## Extension

Add:

- CI eval command;
- GitHub Actions gate;
- model comparison;
- redaction tester;
- incident report generator;
- weekly quality report.
