# Project 8: Production AI Control Center

## Concept

An AI prototype becomes a production system when its behavior can be evaluated, observed, optimized, versioned, deployed, rolled back, and improved through evidence.

This project builds a control center around one previous AI system. It does not improve the AI feature itself first. It builds the operating layer around it.

The learner should finish with a runnable CLI, notebook, or local dashboard that collects eval results, traces, cost, latency, feedback, release manifests, rollout decisions, rollback triggers, and improvement actions.

The focus is:

```text
AI system behavior -> evals -> traces -> metrics -> release gate -> rollout or rollback -> improvement loop
```

## What You Will Build

Build a Production AI Control Center.

The system monitors one previous project and shows:

- eval pass rates;
- regression failures;
- request traces;
- prompt/model/index/schema versions;
- cost per task;
- latency;
- cache/optimization impact;
- feedback categories;
- release manifest;
- rollout status;
- rollback target;
- release decision.

The product answers one question:

```text
Is this AI system ready to pilot, expand, revise, roll back, or stop?
```

## Chapters Practiced

This project proves understanding from:

- Chapter 37: evaluation as product infrastructure;
- Chapter 38: observability, traces, cost, and latency;
- Chapter 39: caching, streaming, batching, and inference optimization;
- Chapter 40: LLMOps: versioning prompts, models, evals, data, and indexes;
- Chapter 41: deployment, rollbacks, feedback, and continuous improvement.

It may reuse earlier projects as the system under observation, but it should not require future course concepts.

Do not require security threat modeling, prompt-injection testing, privacy governance, responsible AI review, or product economics here. Those belong to Part 9.

Later Part 9 can improve this control center with:

- AI security threat model;
- prompt-injection and tool-abuse tests;
- privacy and retention policy;
- responsible AI risk register;
- product economics and build-vs-buy decision.

## User Story

Primary user:

```text
Builder, engineering lead, product owner, or founder operating an AI system.
```

User story:

```text
As an AI system owner,
I want a control center that shows evals, traces, cost, latency, versions, feedback, and release gates,
so I can decide whether to ship, pilot, revise, roll back, or stop.
```

The user needs operational clarity:

```text
what changed
what passed
what failed
what it cost
what got slower
what users disliked
what version produced behavior
what release decision is safe
```

## System Requirements

### System Under Observation

Choose one previous project:

- Project 4: Structured Intake Extractor;
- Project 5: Grounded Research Assistant;
- Project 6: Multimodal Document Analyst;
- Project 7: Operations Action Assistant.

Recommended first choice:

```text
Grounded Research Assistant
```

It has retrieval, generation, citations, refusals, evals, feedback, and clear quality metrics.

If the previous project is not implemented, use mock traces and eval results that represent its behavior.

### Output

The control center should produce:

- `eval-report.md`;
- `trace-summary.md`;
- `cost-latency-report.md`;
- `optimization-report.md`;
- `release-manifest.json`;
- `release-readiness.md`;
- `feedback-improvement-log.md`;
- optional local dashboard.

### Interface

Choose one implementation path:

```text
CLI:
control run-evals
control show-traces
control show-costs
control show-optimization
control show-manifest
control release-check
control feedback-review

Notebook:
Run eval, trace, cost, release, and feedback analysis cells.

Local dashboard:
Tabs for evals, traces, costs, optimization, manifest, release, feedback.
```

CLI or notebook is enough. Dashboard is optional.

## Starter Data

Create local sample data if the chosen AI system does not already emit it.

### Eval Results

```json
[
  {
    "eval_id": "EVAL-001",
    "system_version": "rag-v3",
    "eval_set_version": "rag-eval-2026-01",
    "category": "citation",
    "case_id": "Q001",
    "status": "pass",
    "must_pass": true
  },
  {
    "eval_id": "EVAL-002",
    "system_version": "rag-v3",
    "eval_set_version": "rag-eval-2026-01",
    "category": "refusal",
    "case_id": "Q019",
    "status": "fail",
    "must_pass": true,
    "failure_reason": "Answered unsupported question"
  }
]
```

### Traces

```json
[
  {
    "trace_id": "TR-001",
    "request_id": "REQ-001",
    "system_version": "rag-v3",
    "manifest_version": "release-2026-01-10",
    "prompt_version": "answer-prompt-v4",
    "model_version": "model-route-standard-v2",
    "index_version": "kb-index-v7",
    "status": "answered",
    "latency_ms": 3200,
    "cost_estimate": 0.042,
    "cache_hit": false,
    "validation_status": "pass"
  }
]
```

### Feedback

```json
[
  {
    "feedback_id": "FB-001",
    "trace_id": "TR-001",
    "label": "wrong_source",
    "comment": "It cited release notes instead of the API key guide.",
    "triage_status": "new"
  }
]
```

### Release Manifest

```json
{
  "manifest_version": "release-2026-01-10",
  "system_version": "rag-v3",
  "prompt_version": "answer-prompt-v4",
  "model_route_version": "model-route-standard-v2",
  "eval_set_version": "rag-eval-2026-01",
  "corpus_version": "support-docs-2026-01",
  "index_version": "kb-index-v7",
  "schema_version": "answer-record-v2",
  "release_owner": "builder",
  "rollback_manifest": "release-2025-12-20"
}
```

Seed data with both good and bad signals:

```text
Good:
- eval pass rate above threshold;
- low p95 latency;
- cost within budget;
- feedback mostly useful.

Bad:
- one must-pass eval failure;
- one high-latency trace;
- one unversioned trace;
- one stale manifest reference;
- one feedback item that should become eval case.
```

## MVP Milestone

Build the simplest useful control center first.

MVP must:

1. Load eval results.
2. Load traces.
3. Load release manifest.
4. Calculate eval pass rate.
5. Identify must-pass failures.
6. Calculate average cost and p95 latency.
7. Check whether every trace has manifest version.
8. Produce release readiness decision.

MVP flow:

```text
evals + traces + manifest
  -> metric calculator
  -> release gate
  -> readiness report
```

Release decision options:

```text
ship
pilot
revise
rollback
stop
```

Do not build full dashboards first. Start with clear release logic.

## V1 Milestone

After MVP works, add operational depth.

V1 should include:

- eval category breakdown;
- regression detection;
- trace viewer;
- cost by route or feature;
- latency percentiles;
- optimization comparison;
- manifest diff;
- feedback triage;
- improvement log;
- rollback trigger check.

V1 should make failures explainable:

```text
What failed?
Which version caused it?
Which traces show it?
Which eval case catches it?
What release decision follows?
```

## Release Discipline Milestone

Add a short file named `production-release-policy.md`.

It should explain:

- eval thresholds;
- must-pass categories;
- cost budget;
- latency budget;
- rollout stages;
- rollback triggers;
- feedback triage process;
- improvement cadence;
- who owns release decisions.

This keeps the project aligned with Part 8: production reliability is an operating discipline, not a one-time launch.

## Core Components

### 1. Eval Runner or Eval Loader

Use real eval output from a previous project or load seeded eval results.

Track:

- eval set version;
- system version;
- category;
- pass/fail;
- must-pass flag;
- failure reason;
- previous result, if available.

Eval summary:

```json
{
  "system_version": "rag-v3",
  "eval_set_version": "rag-eval-2026-01",
  "total": 40,
  "passed": 36,
  "failed": 4,
  "pass_rate": 0.9,
  "must_pass_failures": 1
}
```

### 2. Regression Detector

Compare current eval run to previous run.

Detect:

- newly failing cases;
- fixed cases;
- category regressions;
- must-pass regressions.

Example:

```text
Regression:
Q019 passed in release-2025-12-20 but failed in release-2026-01-10.
Category: refusal
Impact: blocks release
```

### 3. Trace Viewer

Trace fields:

```text
trace_id
request_id
system_version
manifest_version
prompt_version
model_route_version
index_version
status
latency_ms
cost_estimate
cache_hit
validation_status
feedback_label
```

For RAG systems, include:

```text
retrieved_sources
citation_validation
refusal_status
```

For structured extraction systems, include:

```text
schema_version
validation_errors
fallback_used
```

For tool systems, include:

```text
tool_schema_version
approval_status
execution_status
```

### 4. Cost and Latency Monitor

Calculate:

- average cost;
- p50 latency;
- p95 latency;
- max latency;
- cost by route;
- latency by route;
- cost by status;
- slowest traces.

Example:

```text
Average cost per answer: $0.042
p95 latency: 4.8s
Cost budget: $0.060
Latency budget: 6.0s
Status: within budget
```

### 5. Optimization Tracker

Track before/after impact for:

- caching;
- streaming;
- batching;
- route changes;
- prompt/context reduction;
- fallback reduction.

Optimization record:

```json
{
  "optimization": "cache frequent API key reset answers",
  "before_p95_latency_ms": 4800,
  "after_p95_latency_ms": 2100,
  "before_avg_cost": 0.042,
  "after_avg_cost": 0.018,
  "quality_regression": false
}
```

Optimization must not silently reduce quality.

### 6. Release Manifest Manager

Track behavior-shaping assets:

```text
prompt_version
model_route_version
eval_set_version
corpus_version
index_version
tool_schema_version
routing_rules_version
schema_version
```

Manifest check:

```text
Every trace must include manifest version.
Manifest must include rollback target.
Manifest must include eval set version.
Manifest must include release owner.
```

### 7. Release Gate

Release gate uses evidence to decide.

Inputs:

- eval pass rate;
- must-pass failures;
- regression count;
- cost budget;
- latency budget;
- manifest completeness;
- rollback target;
- feedback blockers.

Example gate:

```json
{
  "decision": "revise",
  "blocking_issues": [
    "1 must-pass eval failure",
    "2 traces missing manifest_version"
  ],
  "owner": "builder",
  "next_action": "fix refusal behavior and rerun evals"
}
```

### 8. Feedback Triage

Feedback should become structured improvement work.

Triage fields:

```text
feedback_id
trace_id
label
failure_category
severity
linked_eval_case
fix_candidate
status
owner
```

Feedback flow:

```text
feedback
  -> trace
  -> failure category
  -> eval case
  -> fix candidate
  -> release decision
```

Do not patch production from one anecdote without trace and eval follow-up.

### 9. Rollback Planner

Define rollback target and triggers.

Rollback fields:

```text
current_manifest
rollback_manifest
rollback_trigger
rollback_action
owner
validation_after_rollback
```

Examples:

```text
Rollback if p95 latency > 10s for pilot window.
Rollback if must-pass eval fails.
Rollback if bad feedback rate exceeds threshold.
Rollback if trace manifest is incomplete.
```

### 10. Improvement Log

Track continuous improvement.

Improvement item:

```json
{
  "item_id": "IMP-001",
  "source": "feedback",
  "trace_id": "TR-001",
  "failure_category": "wrong_source",
  "eval_case_added": "Q041",
  "fix_candidate": "boost approved API reset guide",
  "status": "planned"
}
```

The improvement loop is complete only when:

```text
production signal -> eval case -> fix -> rerun eval -> release decision
```

## Architecture

```text
AI system or mock data
  -> eval loader/runner
  -> trace loader
  -> cost/latency calculator
  -> optimization tracker
  -> manifest manager
  -> feedback triage
  -> release gate
  -> readiness report
```

Optional dashboard architecture:

```text
local data store
  -> metrics service
  -> dashboard tabs
  -> report exporter
```

Key boundary:

```text
The control center does not decide by vibes.
It decides from evals, traces, budgets, manifests, feedback, and rollback readiness.
```

## Data Model

### Eval Result

```json
{
  "eval_run_id": "ER-001",
  "case_id": "Q019",
  "system_version": "rag-v3",
  "eval_set_version": "rag-eval-2026-01",
  "category": "refusal",
  "status": "fail",
  "must_pass": true,
  "failure_reason": "Answered unsupported question"
}
```

### Trace

```json
{
  "trace_id": "TR-001",
  "request_id": "REQ-001",
  "manifest_version": "release-2026-01-10",
  "prompt_version": "answer-prompt-v4",
  "model_route_version": "model-route-standard-v2",
  "index_version": "kb-index-v7",
  "latency_ms": 3200,
  "cost_estimate": 0.042,
  "status": "answered",
  "validation_status": "pass"
}
```

### Release Manifest

```json
{
  "manifest_version": "release-2026-01-10",
  "system_version": "rag-v3",
  "prompt_version": "answer-prompt-v4",
  "model_route_version": "model-route-standard-v2",
  "eval_set_version": "rag-eval-2026-01",
  "corpus_version": "support-docs-2026-01",
  "index_version": "kb-index-v7",
  "schema_version": "answer-record-v2",
  "routing_rules_version": "routing-v2",
  "release_owner": "builder",
  "rollback_manifest": "release-2025-12-20"
}
```

### Release Decision

```json
{
  "manifest_version": "release-2026-01-10",
  "decision": "revise",
  "blocking_issues": [
    "must-pass refusal eval failed"
  ],
  "cost_status": "within_budget",
  "latency_status": "within_budget",
  "rollback_ready": true,
  "owner": "builder"
}
```

## Evaluation Dataset

Create seeded operational data for at least:

- 2 release manifests;
- 2 eval runs;
- 30 traces;
- 10 feedback items;
- 3 optimization records;
- 5 release gate scenarios.

Release gate scenarios:

```text
Scenario 1:
All gates pass.
Expected: pilot or ship.

Scenario 2:
Must-pass eval fails.
Expected: revise.

Scenario 3:
p95 latency over budget.
Expected: revise or pilot blocked.

Scenario 4:
manifest missing rollback target.
Expected: block release.

Scenario 5:
previous stable release exists and current release regresses.
Expected: rollback candidate.
```

## Seeded Failure Cases

Include failures that test production discipline.

### Case 1: Must-Pass Eval Failure

```text
Refusal eval fails.
Expected: release gate blocks ship.
```

### Case 2: Missing Manifest Version

```text
Trace lacks manifest_version.
Expected: manifest completeness failure.
```

### Case 3: High Latency

```text
Several traces exceed p95 latency budget.
Expected: release gate blocks expansion or flags optimization.
```

### Case 4: Cost Spike

```text
Average cost per task exceeds budget.
Expected: release decision revise.
```

### Case 5: Optimization Regresses Quality

```text
Caching improves latency but stale answer eval fails.
Expected: optimization not accepted.
```

### Case 6: Feedback Without Trace

```text
Feedback item has no trace_id.
Expected: triage marks insufficient evidence.
```

### Case 7: No Rollback Target

```text
Manifest has no rollback_manifest.
Expected: release blocked.
```

## Acceptance Rubric

### Basic Pass

- loads evals, traces, and manifest;
- calculates eval pass rate;
- detects must-pass failures;
- calculates average cost and p95 latency;
- checks manifest completeness;
- produces release readiness report.

### Strong Pass

- detects regressions;
- breaks metrics down by category or route;
- shows trace details;
- tracks optimization before/after;
- triages feedback into failure categories;
- checks rollback readiness.

### Excellent

- includes multiple release scenarios;
- blocks releases for unversioned behavior changes;
- converts feedback into eval cases;
- compares manifests;
- produces improvement log;
- includes `production-release-policy.md`;
- provides dashboard or polished CLI reports.

## Metrics

Measure:

- eval pass rate;
- must-pass failure count;
- regression count;
- average cost per task;
- p50 latency;
- p95 latency;
- traces missing manifest version;
- cache hit rate;
- optimization quality regression count;
- feedback items linked to traces;
- feedback items converted to eval cases.

Suggested gates:

```text
Eval pass rate: >= 90%
Must-pass failures: 0
p95 latency: <= defined budget
Average cost per task: <= defined budget
Traces with manifest version: 100%
Rollback manifest exists: yes
Blocking feedback unresolved: 0
```

The exact thresholds can vary by system. The release decision must be explicit.

## Final Deliverables

At the end, the learner should have:

- runnable CLI, notebook, or dashboard;
- seeded eval results;
- seeded traces;
- seeded feedback;
- release manifest;
- release gate logic;
- optimization tracker;
- feedback triage;
- rollback planner;
- `eval-report.md`;
- `trace-summary.md`;
- `cost-latency-report.md`;
- `optimization-report.md`;
- `release-manifest.json`;
- `release-readiness.md`;
- `feedback-improvement-log.md`;
- `production-release-policy.md`;
- short project `README.md`.

Example file layout:

```text
project-08-production-ai-control-center/
  README.md
  production-release-policy.md
  data/
    eval-runs.json
    previous-eval-runs.json
    traces.json
    feedback.json
    optimization-records.json
    release-manifest.json
  src/
    evals.py
    traces.py
    metrics.py
    optimization.py
    manifests.py
    feedback.py
    release_gate.py
    rollback.py
    reports.py
  outputs/
    eval-report.md
    trace-summary.md
    cost-latency-report.md
    optimization-report.md
    release-readiness.md
    feedback-improvement-log.md
```

The exact stack can differ. The operating discipline should not.

## Demo Script

A 3-minute demo should show:

1. Load evals, traces, feedback, and manifest.
2. Show eval pass rate and must-pass failure.
3. Show a trace with prompt/model/index versions.
4. Show cost and p95 latency.
5. Show optimization before/after.
6. Run release gate.
7. Show release blocked because of a seeded issue.
8. Fix or swap to passing scenario.
9. Show release decision changes.
10. Show feedback converted into eval/improvement item.

Demo narrative:

```text
This project shows the operating layer around production AI.
The system is not shipped because it feels good in a demo.
It is shipped only when evals pass, traces are versioned, cost and latency are within budget, rollback is ready, and feedback has a path back into improvement.
```

## What Learner Understands

After completing this project, the learner should understand:

- production AI needs evals as infrastructure;
- traces connect behavior to versions;
- cost and latency are product constraints;
- optimization must preserve quality;
- LLMOps means versioning behavior-shaping assets;
- release gates should be evidence-based;
- rollback readiness is part of deployment;
- feedback becomes useful only when connected to traces and evals;
- release decisions can be ship, pilot, revise, rollback, or stop.

## Extension

Later upgrades:

- Part 9: add threat model, prompt-injection tests, privacy review, responsible AI risk review, product economics;
- CI: run eval gate on every pull request;
- Dashboard: add charts and drill-down trace explorer;
- Reports: generate weekly quality and cost review.
