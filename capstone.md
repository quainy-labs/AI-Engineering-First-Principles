# Capstone: Build One Real Intelligent System

## Goal

Prove end-to-end AI engineering capability by building one useful, inspectable system around real work.

No capstone starts with technology. Every capstone starts with real work someone wants improved.

The capstone is not "build an AI app." It is:

```text
problem -> workflow -> baseline -> system boundary -> data -> model/retrieval/tools -> eval -> risk -> deployment -> impact
```

If learner cannot explain the work before explaining the model, capstone is not ready.

## Capstone Promise

By the end, another builder should be able to inspect the repo and understand:

- what work was improved;
- why AI belongs in the system;
- where AI does not belong;
- how the system is built;
- how quality is measured;
- how risks are controlled;
- how the system would be operated.

## Scope Rule

Choose one narrow workflow.

Good scope:

```text
Convert inbound support emails into validated intake records and route risky cases to review.
```

Bad scope:

```text
Build an AI support agent.
```

Good capstone systems are small enough to finish and real enough to evaluate.

## Required Artifacts

Create:

```text
capstone/
  problem-brief.md
  workflow-map.md
  baseline.md
  architecture.md
  information-design.md
  model-interface.md
  retrieval-or-tool-design.md
  evaluation-dataset.md
  error-analysis.md
  observability-plan.md
  risk-review.md
  operating-model.md
  product-reality-review.md
  ai-product-economics.md
  cost-latency-estimate.md
  deployment-plan.md
  demo-notes.md
  public-build-note.md
```

Each artifact should be short but concrete. Do not write generic essays.

## System Requirements

Your system must include:

- real user;
- measurable baseline;
- clear system boundary;
- deterministic components;
- AI components;
- human approval or review where needed;
- eval dataset;
- failure analysis;
- trace/observability plan;
- security/risk review;
- owner and operating model;
- product reality review;
- improvement loop.

## Capstone Path

### 1. Choosing Meaningful Problem

Create:

- `problem-brief.md`
- `workflow-map.md`
- `baseline.md`

Answer:

```text
Who has the problem?
What work happens today?
Where does it fail?
What is current baseline?
What improvement would matter?
Why might AI help?
Why might AI not help?
```

### 2. Designing End-to-End System

Create:

- `architecture.md`
- `information-design.md`

Define:

- user;
- input;
- output;
- deterministic components;
- AI components;
- databases/files;
- queues/jobs if needed;
- human review points;
- excluded behavior.

### 3. Building Prototype

Create:

- `model-interface.md`
- `retrieval-or-tool-design.md`

Include:

- model task boundary;
- prompt/interface contract;
- output schema;
- validation;
- fallback;
- model route;
- retrieval design or tool interface;
- human approval rule.

Build:

- runnable CLI, web app, API, or notebook;
- sample data;
- deterministic validation;
- model/retrieval/tool path;
- user correction or approval flow.

### 4. Evaluating, Securing, and Shipping

Create:

- `evaluation-dataset.md`
- `error-analysis.md`
- `observability-plan.md`
- `risk-review.md`
- `cost-latency-estimate.md`
- `deployment-plan.md`

Minimum eval set:

- 20 normal cases;
- 5 edge cases;
- 5 should-refuse or should-escalate cases;
- 5 past or seeded failure cases.

Measure:

- task success;
- output validity;
- refusal/escalation accuracy;
- unsupported claim rate if answering from sources;
- unsafe action rate if using tools;
- cost;
- latency.

Include:

- trace fields;
- redaction rules;
- risk register;
- release gate;
- owner;
- rollback trigger;
- feedback loop.

### 5. Publishing Proof of Capability

Create:

- `operating-model.md`
- `product-reality-review.md`
- `demo-notes.md`
- `public-build-note.md`
- `ai-product-economics.md`

Include:

- data and access risks;
- prompt injection tests;
- owners;
- allowed and forbidden use cases;
- product claims;
- adoption plan;
- stop condition;
- build-vs-buy decision.

Demo should show:

- happy path;
- failure path;
- human correction or approval;
- eval result;
- trace/log;
- one improvement from error analysis.

## Good Capstone Examples

- internal knowledge assistant;
- support operations assistant;
- compliance document reviewer;
- research analyst;
- hiring workflow assistant;
- personal learning/research system;
- local business automation workflow.

## Bad Capstone Examples

- generic chatbot with no workflow;
- wrapper around one model call;
- RAG demo with no retrieval eval;
- agent with broad tools and no approval;
- fine-tuned model with no baseline;
- dashboard with no real user;
- prototype with no failure cases.

## Artifact Templates

### `problem-brief.md`

```text
User:
Workflow:
Pain:
Current baseline:
Target improvement:
Why AI may help:
Why AI may not help:
Decision:
```

### `workflow-map.md`

```text
Current workflow:
1.
2.
3.

Proposed workflow:
1.
2.
3.

Human-owned decisions:
Deterministic-code decisions:
AI-assisted decisions:
Out of scope:
```

### `architecture.md`

```text
Components:
- UI/CLI:
- API:
- worker/job:
- storage:
- retrieval:
- model:
- tools:
- observability:

Request flow:

Failure paths:
```

### `model-interface.md`

```text
Model task:
Inputs:
Context:
Output schema:
Validation:
Fallback:
Human review:
Model route:
Prompt/interface version:
```

### `evaluation-dataset.md`

```text
Eval version:
Cases:
- id:
  category:
  input:
  expected:
  must_pass:
  risk:

Metrics:
Release gate:
```

### `risk-review.md`

```text
Data handled:
User roles:
Actions/tools:

Risk register:
- risk:
  impact:
  control:
  detection:
  owner:
```

### `deployment-plan.md`

```text
Pilot users:
Scope:
Feature flag:
Release gate:
Rollback trigger:
Feedback process:
Improvement cadence:
```

### `product-reality-review.md`

```text
Primary user:
Workflow:
Baseline:
Measured impact:
Adoption signal:
Allowed product claims:
Forbidden product claims:
Support plan:
Decision: expand / revise / pause / stop
```

## Final Review Questions

1. What work improved?
2. Who used it?
3. What was baseline?
4. What changed?
5. How was quality measured?
6. What failed?
7. What risks remain?
8. Who owns the system?
9. What would next improvement be?

## Pass Standard

Capstone passes when:

- real workflow is clear;
- baseline exists;
- system boundary is explicit;
- AI component is bounded;
- deterministic components own deterministic work;
- eval set includes normal, edge, refusal, and failure cases;
- error analysis leads to at least one improvement;
- risk review is concrete;
- trace/observability plan exists;
- deployment plan has release gate and rollback;
- product reality review exists;
- demo proves useful behavior.

## Distinction Standard

Capstone is strong when it also includes:

- runnable local prototype;
- seeded sample data;
- repeatable eval command;
- trace viewer or log sample;
- before/after metric;
- cost and latency measurement;
- short public build note explaining what was learned.

## Final Principle

Do not prove that AI can generate output.

Prove that system improves work, handles failure, and can be trusted enough for its scope.
