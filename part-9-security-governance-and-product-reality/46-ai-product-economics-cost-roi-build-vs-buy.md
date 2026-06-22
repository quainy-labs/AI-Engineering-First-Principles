# Chapter 46: AI Product Economics: Cost, ROI, Build-vs-Buy

Chapter 45 made risk explicit. You learned how to identify harms, red-team failures, design controls, assign owners, and decide whether residual risk is acceptable for a specific release scope.

This chapter asks the final product question:

```text
Is this AI system worth building, buying, operating, expanding, or stopping?
```

Technical success is not the same as product success.

A system can work and still not be worth it.

The Quainy principle is:

> AI should earn its place through meaningful impact, not novelty.

## Concept Overview

AI product economics is the practice of comparing the full cost of an AI system against the value it creates and the alternatives available.

It includes:

- development cost;
- inference cost;
- model provider cost;
- storage and index cost;
- data pipeline cost;
- evaluation cost;
- observability cost;
- latency cost;
- support cost;
- risk-control cost;
- maintenance cost;
- user training cost;
- opportunity cost.

It also includes value:

- time saved;
- quality improved;
- errors reduced;
- revenue increased;
- retention improved;
- faster cycle time;
- better customer experience;
- lower support burden;
- new workflow capability;
- improved compliance or auditability;
- better decision support.

The economic question is not:

```text
Can we build it?
```

The better question is:

```text
Does this system create enough durable value to justify its full cost and risk compared with alternatives?
```

The alternatives usually are:

```text
build
buy
partner
automate without AI
improve process
use existing tool
do nothing
stop after pilot
```

AI engineering maturity includes knowing when not to build.

## Why It Exists

AI prototypes can be deceptively cheap.

A demo may require:

- one prompt;
- one model call;
- one uploaded document;
- one happy path;
- no auth;
- no evals;
- no observability;
- no rollback;
- no privacy review;
- no support process;
- no maintenance plan.

Production requires much more:

- application architecture;
- user permissions;
- ingestion;
- parsing;
- retrieval;
- model routing;
- fallbacks;
- structured outputs;
- tool validation;
- eval infrastructure;
- trace review;
- cost monitoring;
- latency optimization;
- deployment gates;
- security controls;
- privacy controls;
- human review;
- incident response;
- product support.

This is why teams underestimate AI economics.

They compare:

```text
prototype cost
```

against:

```text
imagined value
```

instead of comparing:

```text
production lifecycle cost
```

against:

```text
measured value and alternatives
```

AI product economics exists because enthusiasm can hide weak leverage.

Bad economic reasoning:

```text
Users like it, so it is valuable.
```

Better economic reasoning:

```text
For this workflow, the system saves 11 minutes per completed task,
reduces rework by 18%,
costs $0.23 per task to run,
requires 4 hours per week of review,
and performs better than vendor alternatives because it integrates with our private workflow data.
```

Usage is not value.

Value appears when the workflow becomes meaningfully better.

## Mental Model

Think of an AI system as a business machine.

It consumes:

- engineering time;
- user attention;
- data;
- compute;
- vendor services;
- review effort;
- support effort;
- risk budget.

It produces:

- completed tasks;
- better decisions;
- faster workflows;
- fewer errors;
- improved user experience;
- measurable business outcomes.

The mental model is:

```text
baseline workflow
  -> AI intervention
  -> changed workflow
  -> measured impact
  -> full cost
  -> alternative comparison
  -> decision
```

Start with baseline.

Without baseline, ROI becomes fiction.

Baseline asks:

- How is the task done today?
- How long does it take?
- What does it cost?
- How often does it happen?
- What errors occur?
- Who reviews the work?
- What is the current quality?
- What happens if nothing changes?

Then measure the AI system:

- How much time is saved?
- How often is output accepted?
- How much review is still needed?
- How many errors are introduced?
- How many errors are prevented?
- What does each task cost?
- What is the support burden?
- Does user trust improve or decline?
- Does the workflow become simpler or more complex?

Finally compare alternatives:

```text
Build:
More control, more maintenance.

Buy:
Less custom work, less control.

Automate without AI:
More predictable, less flexible.

Improve process:
May solve root problem without software.

Do nothing:
Valid when pain is low or change cost is too high.
```

The goal is not to justify AI.

The goal is to choose the best leverage.

## Internal Mechanics

AI economics has several cost and value components.

### Baseline Cost

Baseline cost is the cost of the current workflow.

Estimate:

```text
Task volume:
Time per task:
People involved:
Hourly cost:
Error rate:
Rework rate:
Escalation rate:
Cycle time:
Customer impact:
Revenue impact:
Compliance impact:
```

Example:

```text
Support ticket triage:
10,000 tickets/month
4 minutes manual triage per ticket
Average loaded cost: $45/hour
Monthly triage labor cost:
10,000 * 4/60 * 45 = $30,000
```

This does not mean AI can save all $30,000.

It only defines the starting point.

### Development Cost

Development cost includes:

- product discovery;
- design;
- engineering;
- data work;
- model integration;
- retrieval pipeline;
- tool integration;
- auth and permissions;
- eval development;
- observability;
- security and privacy review;
- deployment;
- documentation;
- user training.

Prototype engineering is only one line item.

If the system touches important workflows, evals, privacy, security, support, and rollout may cost as much as model integration.

### Run Cost

Run cost includes every cost per task or per month after launch.

Examples:

- input tokens;
- output tokens;
- embedding calls;
- reranking calls;
- model routing;
- retries;
- tool calls;
- storage;
- vector database;
- logs and traces;
- monitoring;
- batch jobs;
- cache infrastructure;
- human review time;
- support time.

A simple formula:

```text
run cost per task =
model cost
+ retrieval cost
+ tool cost
+ infrastructure cost
+ observability cost
+ review cost
+ support cost
```

Review cost is often the hidden killer.

If AI saves 5 minutes of writing but adds 6 minutes of checking, the economics are bad.

### Quality Value

AI may create value by improving quality, not only saving time.

Quality value may include:

- fewer mistakes;
- better coverage;
- better citations;
- more consistent outputs;
- faster detection of missing information;
- improved customer experience;
- better audit trails;
- reduced rework;
- improved compliance.

Quality is harder to price than time, but it still matters.

For example, a legal review assistant may save little time but catch missing clauses more reliably. That can be valuable if the avoided risk is meaningful.

### Latency and Adoption Cost

Slow AI changes user behavior.

If a task normally takes 5 seconds and the AI takes 30 seconds, users may abandon it even if output quality is high.

Latency cost appears as:

- lower adoption;
- context switching;
- queue delays;
- user frustration;
- workflow interruption;
- lower trust.

Economics should include usability and adoption, not only compute.

### Risk and Control Cost

Risk controls add cost.

Examples:

- human approval;
- redaction;
- audit logging;
- access checks;
- security reviews;
- red-team testing;
- eval maintenance;
- legal review;
- incident response;
- support escalation.

These controls are not optional for serious systems. They are part of total cost.

If controls make the workflow slower than the baseline, the AI system may not be worth it.

### Maintenance Cost

AI systems drift.

Maintenance includes:

- prompt updates;
- model upgrades;
- eval updates;
- corpus updates;
- index rebuilds;
- tool schema changes;
- provider changes;
- cost tuning;
- latency tuning;
- monitoring review;
- incident response;
- user feedback triage.

The question is not only:

```text
Can we launch this?
```

It is:

```text
Can we afford to keep this good?
```

### Build-vs-Buy

Build when differentiation or control matters.

Build is stronger when:

- workflow is unique;
- private data is central;
- deep system integration is required;
- product experience is strategic;
- vendor options are weak;
- governance and audit need custom controls;
- performance requirements are special;
- internal capability is a long-term advantage.

Buy when the capability is commodity.

Buy is stronger when:

- vendor product already solves the workflow;
- integration needs are light;
- speed matters more than customization;
- internal team lacks maintenance capacity;
- risk is better handled by mature vendor controls;
- feature is not core differentiation.

Do nothing when the baseline is acceptable.

Do nothing is stronger when:

- task volume is low;
- error cost is low;
- users do not feel pain;
- process change would be more effective;
- risk outweighs benefit;
- AI adds complexity without impact.

The best AI engineer is not the one who builds everything. It is the one who chooses correctly.

## Real-World Example

Imagine a company considering a custom AI assistant for customer support reply drafting.

Baseline:

```text
Tickets per month: 20,000
Average reply drafting time: 6 minutes
Loaded support cost: $40/hour
Baseline drafting labor: 20,000 * 6/60 * 40 = $80,000/month
```

The team runs a pilot.

Pilot results:

```text
AI draft accepted with light edits: 55%
AI draft rejected: 25%
AI draft needs heavy edits: 20%
Average time saved on accepted drafts: 3 minutes
Average time added on rejected drafts: 1 minute
Cost per AI draft: $0.07
Human review still required: yes
Bad feedback: 6%
Unsupported workflow attempts: 14%
```

Measured value:

```text
Accepted drafts:
20,000 * 55% * 3 minutes = 33,000 minutes saved

Rejected drafts:
20,000 * 25% * 1 minute = 5,000 minutes lost

Net time saved:
28,000 minutes = 466.7 hours

Labor value:
466.7 * $40 = $18,668/month

Inference cost:
20,000 * $0.07 = $1,400/month
```

But this is not full ROI.

Add:

```text
Weekly trace review: 6 hours/month
Prompt/eval maintenance: 12 hours/month
Support training: 8 hours/month
Incident review: 4 hours/month
Engineering maintenance: 20 hours/month
```

If loaded internal cost is $80/hour:

```text
Maintenance labor:
50 * $80 = $4,000/month
```

Approximate monthly net:

```text
$18,668 labor value
- $1,400 inference
- $4,000 maintenance
= $13,268/month
```

Now compare alternatives.

Build:

```text
Pros:
Custom policies, private workflow integration, better citations.

Cons:
Maintenance burden, eval ownership, support effort.
```

Buy:

```text
Pros:
Faster rollout, vendor support, lower internal maintenance.

Cons:
Weaker integration with internal policies, less control over evals and traces.
```

Do nothing:

```text
Pros:
No new complexity.

Cons:
Support team continues spending time on repeated drafts.
```

Decision:

```text
Continue build for billing support only.
Do not expand to technical troubleshooting yet.
Re-evaluate after 60 days.
Stop if bad feedback exceeds 10% or net monthly value drops below $5,000.
```

This is product economics.

The team does not ask whether the assistant is impressive.

It asks whether it improves a real workflow enough to justify its cost, maintenance, risk, and alternatives.

## Common Mistakes and Failure Modes

### Counting Token Cost Only

Token cost is visible, but it is not total cost.

Total cost includes:

- engineering;
- data maintenance;
- evals;
- observability;
- support;
- human review;
- security;
- privacy;
- rollout;
- incidents;
- vendor management.

Cheap inference can still produce expensive operations.

### Treating Usage as Value

Usage means people tried or used the system.

Value means the workflow improved.

Measure:

- task completion;
- time saved;
- error reduction;
- accepted outputs;
- quality improvement;
- reduced escalations;
- user trust;
- business outcome.

### Ignoring Review Time

AI-generated output often requires review.

If review time is high, ROI drops.

Measure:

```text
draft time saved
- review time added
- correction time added
- escalation time added
```

### Forgetting Maintenance

AI systems need upkeep.

Models change. Prompts drift. Corpora become stale. Tools evolve. Evals need updates. Costs shift. User behavior changes.

If nobody owns maintenance, product quality declines.

### Building Commodity Features

If many vendors already solve the workflow well, custom build may be waste.

Build only when your data, workflow, integration, control, or product experience creates real advantage.

### Automating a Broken Process

AI can make a broken process faster without making it better.

Before building, ask:

- can the workflow be simplified?
- can rules be clarified?
- can data quality be improved?
- can a form or deterministic automation solve this?
- can policy ambiguity be removed?

Sometimes the best AI project is a process redesign.

### No Stop Condition

Without stop conditions, teams continue systems because they already invested effort.

Define stop conditions before expansion:

```text
Stop if value remains below threshold.
Pause if risk incidents exceed threshold.
Revisit build-vs-buy if vendor reaches required capability.
Stop if maintenance exceeds expected savings.
Stop if users do not trust the output after improvement cycle.
```

## System Design Decisions

### Decide the Economic Unit

Choose the unit of value.

Examples:

```text
cost per ticket resolved
cost per document processed
cost per qualified lead
cost per successful support answer
cost per analyst report
cost per approved workflow action
cost per hour saved
```

The economic unit should match the workflow.

### Decide Value Metrics

Choose value metrics before launch.

Examples:

- time saved per task;
- acceptance rate;
- rework reduction;
- escalation reduction;
- quality score improvement;
- customer satisfaction;
- cycle time reduction;
- revenue influenced;
- risk reduction;
- audit completeness.

Avoid vague metrics like:

```text
AI engagement
number of generated responses
coolness
```

### Decide Cost Metrics

Track:

- cost per request;
- cost per completed task;
- cost per accepted output;
- cost per successful workflow;
- p95 latency;
- support time;
- review time;
- eval maintenance;
- incident cost;
- infrastructure cost.

Cost per request can be misleading. A cheap request that fails may cost more than an expensive request that completes the job.

### Decide Build-vs-Buy Criteria

Create criteria before falling in love with a solution.

Example:

```text
Build if:
- private workflow data is required;
- vendor cannot support permission model;
- custom evals and traces are required;
- workflow is strategic differentiation.

Buy if:
- vendor solves 80% of workflow;
- integration is acceptable;
- controls meet requirements;
- internal maintenance would distract from core work.

Do nothing if:
- baseline pain is low;
- usage volume is low;
- risk is high;
- process fix is better.
```

### Decide Expansion and Stop Conditions

Expansion conditions:

```text
task success >= target
net value >= threshold
bad feedback <= threshold
cost per task <= threshold
p95 latency <= threshold
risk incidents = acceptable
support burden = acceptable
```

Stop conditions:

```text
value below threshold after improvement cycle
maintenance exceeds benefit
controls make workflow slower than baseline
high-severity incidents without acceptable mitigation
users do not trust output
vendor alternative becomes clearly better
```

### Decide Ownership

AI economics needs an owner.

Owners:

```text
Product:
value, adoption, workflow fit.

Engineering:
run cost, latency, reliability, maintenance.

Operations/domain:
quality, review time, process impact.

Security/risk:
control cost, incidents, acceptable use.

Finance/business:
ROI assumptions and decision review.
```

Small teams may combine roles, but the questions remain.

## Build Artifact

Create `ai-product-economics.md`.

Use this structure:

```text
# AI Product Economics

## Workflow
Workflow:
Users:
Task volume:
Current process:
Pain:
Baseline quality:
Baseline cost:

## AI System
Proposed capability:
Scope:
Required data:
Required tools:
Human review:
Risk controls:

## Value Model
Time saved:
Quality improvement:
Error reduction:
Revenue/retention impact:
Risk reduction:
User experience impact:
Measured pilot evidence:

## Cost Model
Development cost:
Inference cost:
Embedding/retrieval cost:
Storage/index cost:
Observability/eval cost:
Human review cost:
Support cost:
Security/privacy cost:
Maintenance cost:
Incident/risk cost:

## Unit Economics
Cost per request:
Cost per completed task:
Cost per accepted output:
Value per completed task:
Net value per month:
Sensitivity assumptions:

## Alternatives
Build:
Buy:
Automate without AI:
Improve process:
Do nothing:

## Decision
Decision: build / buy / pilot / pause / stop / do nothing
Reason:
Owner:
Review date:
Expansion condition:
Stop condition:
```

Artifact complete when another engineer or founder can understand:

- current workflow cost;
- expected AI value;
- full production cost;
- key assumptions;
- alternatives;
- decision;
- stop condition.

## Active Recall Questions

1. Why is technical success not the same as product success?
2. Why should every AI project start with a baseline workflow?
3. What costs are often missing from AI ROI estimates?
4. Why is usage not the same as value?
5. How can human review change economics?
6. When should a team build instead of buy?
7. When should a team buy instead of build?
8. Why can doing nothing be the correct decision?
9. What is a useful economic unit for an AI workflow?
10. Why should stop conditions be defined before expansion?

## Summary

AI product economics compares full lifecycle cost with measured workflow value and realistic alternatives.

The practical loop is:

```text
baseline workflow
  -> AI intervention
  -> pilot evidence
  -> full cost model
  -> value model
  -> alternatives
  -> decision
  -> expansion or stop condition
```

Do not count only token cost. Include engineering, data, evals, observability, support, review, security, privacy, maintenance, and risk. Do not count usage as value. Measure time saved, quality improved, errors reduced, cycle time reduced, or revenue protected.

Build when custom workflow, private data, integration, control, or differentiation matters.

Buy when the capability is commodity and vendor fit is strong.

Do nothing or stop when the baseline is acceptable, risk outweighs value, or maintenance exceeds benefit.

The goal of AI engineering is not to maximize AI usage. It is to create meaningful, durable leverage.

## Preview of Next Chapter

This chapter completes the main AI Engineering First Principles course.

You now have the pieces: problem selection, software architecture, data foundations, models, retrieval, multimodal systems, tools, agents, production reliability, security, governance, and product economics.

The next step is proof of capability through projects. Use the project sequence to build systems that demonstrate what you learned: workflow teardown, app skeleton, knowledge search, structured extraction, grounded research, multimodal document analysis, operations actions, production control, and AI risk review.
