# Capstone: Build One Real Intelligent System

## Goal

Prove end-to-end AI engineering capability by building one useful, inspectable system around real work.

No capstone starts with technology. Every capstone starts with a workflow someone wants improved.

The capstone is not:

```text
build an AI app
```

It is:

```text
problem -> workflow -> baseline -> boundary -> data -> model/retrieval/tools -> eval -> production readiness -> risk -> economics -> proof
```

If the learner cannot explain the work before explaining the model, the capstone is not ready.

## Capstone Promise

By the end, another builder should be able to inspect the repo and understand:

- what work was improved;
- who the system serves;
- why AI belongs in the system;
- where AI does not belong;
- how the system is built;
- how quality is measured;
- how failures are handled;
- how cost and latency are estimated;
- how risks are controlled;
- how the system would be piloted or operated;
- whether the system should ship, revise, pause, stop, buy, or stay a prototype.

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

## Recommended System Types

Choose one:

- structured intake extractor for a real workflow;
- grounded knowledge assistant for a controlled corpus;
- multimodal document analyst for invoices/forms/contracts;
- operations assistant with bounded tools and approvals;
- production hardening layer around an existing AI prototype;
- risk and governance review package for a real AI system;
- hybrid system that combines two of the above without becoming too broad.

Avoid generic chatbots.

## Required Repository Shape

Create a `capstone/` folder.

Recommended structure:

```text
capstone/
  README.md
  01-problem-brief.md
  02-workflow-map.md
  03-baseline.md
  04-system-boundary.md
  05-architecture.md
  06-data-and-knowledge-design.md
  07-model-interface.md
  08-retrieval-tool-or-multimodal-design.md
  09-evaluation-dataset.md
  10-error-analysis.md
  11-observability-and-release-plan.md
  12-security-and-privacy-review.md
  13-responsible-ai-risk-review.md
  14-product-economics.md
  15-launch-decision-memo.md
  16-demo-notes.md
  17-public-build-note.md
  data/
  src/
  outputs/
```

Each artifact should be short but concrete. Do not write generic essays.

If the capstone is primarily a review package, `src/` may be optional. If the capstone claims to be a product system, it should have runnable code or a reproducible notebook.

## System Requirements

Your capstone must include:

- real user;
- measurable baseline;
- clear system boundary;
- deterministic components;
- AI components;
- data or document handling;
- validation or quality checks;
- human correction or approval where needed;
- eval dataset;
- seeded failure cases;
- error analysis;
- trace or observability plan;
- cost and latency estimate;
- security and privacy review;
- responsible AI risk review;
- product economics;
- launch decision;
- demo.

Not every capstone needs every AI technique.

Use only what the workflow needs:

- structured extraction if the problem is messy input;
- retrieval if the problem needs trusted knowledge;
- multimodal extraction if the input is visual/document-based;
- tools only if the system must act;
- production controls only if the prototype is ready for pilot.

## Capstone Path

### 1. Choose a Meaningful Problem

Create:

- `01-problem-brief.md`
- `02-workflow-map.md`
- `03-baseline.md`

Answer:

```text
Who has the problem?
What work happens today?
Where does it fail?
What is the current baseline?
What improvement would matter?
Why might AI help?
Why might AI not help?
What is explicitly out of scope?
```

Minimum baseline:

- time per task;
- volume;
- error or rework rate;
- escalation or review rate;
- current cost or effort;
- qualitative pain from user perspective.

### 2. Design the System Boundary

Create:

- `04-system-boundary.md`
- `05-architecture.md`
- `06-data-and-knowledge-design.md`

Define:

- user;
- input;
- output;
- deterministic components;
- AI components;
- storage;
- files/documents;
- queues/jobs if needed;
- retrieval/data sources if needed;
- tool actions if needed;
- human review points;
- forbidden behavior.

Architecture should make ownership clear:

```text
UI/CLI:
API/service:
storage:
worker/job:
retrieval or parser:
model gateway:
validator:
approval or correction:
tracing:
```

### 3. Build the Prototype

Create:

- `07-model-interface.md`
- `08-retrieval-tool-or-multimodal-design.md`

Build:

- runnable CLI, notebook, local API, or local web app;
- sample data;
- deterministic validation;
- model/retrieval/tool/multimodal path, depending on need;
- user correction or approval flow;
- output records or reports.

The AI component must be bounded.

Define:

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

If using retrieval:

```text
Corpus:
Metadata:
Chunking:
Indexing:
Retriever:
Context builder:
Citation rules:
Refusal rules:
```

If using tools:

```text
Tool registry:
Allowed tools:
Forbidden tools:
Argument validation:
Approval rules:
Workflow state:
Trace review:
```

If using documents/images/audio:

```text
Input types:
Parser/OCR/transcription:
Evidence mapping:
Schema:
Validation:
Correction flow:
```

### 4. Evaluate and Improve

Create:

- `09-evaluation-dataset.md`
- `10-error-analysis.md`

Minimum eval set:

- 20 normal cases;
- 5 edge cases;
- 5 should-refuse or should-escalate cases;
- 5 seeded failure cases.

Measure what fits the system:

- task success;
- output validity;
- field-level accuracy;
- retrieval top-k recall;
- citation accuracy;
- refusal/escalation accuracy;
- unsupported claim rate;
- unsafe action rate;
- validation failure rate;
- human correction rate;
- cost;
- latency.

Error analysis should produce at least one improvement.

Example loop:

```text
failure -> trace/example -> category -> fix -> eval update -> rerun -> decision
```

### 5. Prove Production Readiness

Create:

- `11-observability-and-release-plan.md`

Include:

- trace fields;
- eval gate;
- cost budget;
- latency budget;
- release manifest;
- pilot scope;
- rollback trigger;
- feedback process;
- improvement cadence;
- owner.

The capstone does not need real cloud deployment. It does need a believable operating plan.

### 6. Review Risk and Governance

Create:

- `12-security-and-privacy-review.md`
- `13-responsible-ai-risk-review.md`

Include:

- assets;
- users and affected people;
- data sources;
- trust boundaries;
- PII/sensitive fields;
- access rules;
- retention and deletion notes;
- prompt-injection or tool-abuse tests if applicable;
- red-team cases;
- harm map;
- controls;
- residual risk;
- owner.

The review must be specific to the system.

Bad:

```text
We care about privacy.
```

Good:

```text
Trace logs store customer messages. Raw message text is retained for 7 days in local demo only. Production design stores redacted text and record IDs. Billing details are not sent to the model unless needed for the task.
```

### 7. Decide Product Reality

Create:

- `14-product-economics.md`
- `15-launch-decision-memo.md`

Estimate:

- baseline cost;
- build cost;
- run cost;
- support cost;
- review cost;
- risk-control cost;
- expected time saved;
- quality improvement;
- alternatives.

Compare:

```text
build
buy
automate without AI
improve process
do nothing
```

Decision must be one of:

```text
ship
pilot
revise
pause
stop
buy
do nothing
```

Every decision needs:

- reason;
- blocking issues;
- accepted residual risks;
- owner;
- review date;
- stop condition.

### 8. Publish Proof of Capability

Create:

- `16-demo-notes.md`
- `17-public-build-note.md`

Demo should show:

- happy path;
- failure path;
- human correction or approval;
- eval result;
- trace or log;
- one improvement from error analysis;
- final launch decision.

Public build note should explain:

- problem;
- approach;
- what worked;
- what failed;
- what was learned;
- what would be improved next.

Do not expose private customer, company, or personal data.

## Artifact Templates

### `01-problem-brief.md`

```text
User:
Workflow:
Pain:
Current baseline:
Target improvement:
Why AI may help:
Why AI may not help:
Out of scope:
Decision:
```

### `02-workflow-map.md`

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
Forbidden automation:
```

### `05-architecture.md`

```text
Components:
- UI/CLI:
- API/service:
- worker/job:
- storage:
- data/document pipeline:
- retrieval/model/tool/multimodal path:
- validator:
- review/approval:
- tracing:

Request flow:
Failure paths:
```

### `07-model-interface.md`

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

### `09-evaluation-dataset.md`

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

### `10-error-analysis.md`

```text
Failure category:
Example:
Likely cause:
Fix:
Eval case added:
Result after fix:
Remaining risk:
```

### `11-observability-and-release-plan.md`

```text
Trace fields:
Eval gate:
Cost budget:
Latency budget:
Release manifest:
Pilot users:
Scope:
Feature flag:
Rollback trigger:
Feedback process:
Improvement cadence:
Owner:
```

### `12-security-and-privacy-review.md`

```text
Assets:
Users:
Data sources:
Tools/actions:
Trust boundaries:
PII/sensitive fields:
Access rules:
Retention:
Deletion:
Threats:
Controls:
Detection:
Owner:
```

### `13-responsible-ai-risk-review.md`

```text
Affected people:
Known harms:
Red-team cases:
Human review:
Residual risks:
Incident process:
Owner:
Review cadence:
```

### `14-product-economics.md`

```text
Workflow:
Baseline cost:
Build cost:
Run cost:
Review/support cost:
Expected savings:
Quality impact:
Risk cost:
Alternatives:
- build:
- buy:
- automate without AI:
- improve process:
- do nothing:
Stop condition:
```

### `15-launch-decision-memo.md`

```text
Decision: ship / pilot / revise / pause / stop / buy / do nothing
Reason:
Blocking issues:
Required controls:
Accepted residual risks:
Owner:
Review date:
Stop condition:
```

## Good Capstone Examples

- support intake extractor with review routing;
- internal knowledge assistant with grounded citations;
- invoice/document analyst with evidence and correction;
- operations action assistant with tool approval;
- compliance document reviewer;
- research analyst over a controlled corpus;
- local business workflow automation with narrow tool use.

## Bad Capstone Examples

- generic chatbot with no workflow;
- wrapper around one model call;
- RAG demo with no retrieval eval;
- agent with broad tools and no approval;
- fine-tuned model with no baseline;
- dashboard with no real user;
- prototype with no failure cases;
- product claim that exceeds measured capability.

## Final Review Questions

1. What work improved?
2. Who used it or would use it?
3. What was the baseline?
4. What changed?
5. Where is AI used?
6. Where is AI intentionally not used?
7. How was quality measured?
8. What failed?
9. What improved after error analysis?
10. What risks remain?
11. What does it cost?
12. Who owns the system?
13. Should it ship, pilot, revise, pause, stop, buy, or stay a prototype?

## Pass Standard

Capstone passes when:

- real workflow is clear;
- baseline exists;
- system boundary is explicit;
- AI component is bounded;
- deterministic components own deterministic work;
- prototype is runnable or reproducible;
- eval set includes normal, edge, refusal/escalation, and failure cases;
- error analysis leads to at least one improvement;
- trace/observability plan exists;
- deployment plan has release gate and rollback;
- security/privacy review is concrete;
- responsible AI risk review is concrete;
- product economics exists;
- launch decision memo exists;
- demo proves useful behavior and safe failure handling.

## Distinction Standard

Capstone is strong when it also includes:

- polished local demo;
- seeded sample data;
- repeatable eval command;
- trace viewer or log sample;
- before/after metric;
- cost and latency measurement;
- release manifest;
- feedback-to-eval loop;
- threat or abuse test results;
- build-vs-buy comparison;
- short public build note explaining what was learned.

## Final Principle

Do not prove that AI can generate output.

Prove that a system improves work, handles failure, and can be trusted enough for its scope.
