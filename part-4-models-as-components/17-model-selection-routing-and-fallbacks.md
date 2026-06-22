# Chapter 17: Model Selection, Routing, and Fallbacks

Chapter 16 defined the model interface:

```text
task
-> input fields
-> context sources
-> output contract
-> refusal and escalation
-> validation
-> eval cases
```

This chapter asks which model should serve that interface.

Real systems rarely need one model for everything. They need a model strategy. Different tasks have different quality bars, latency budgets, cost limits, context needs, privacy constraints, and failure impact.

Model choice is not a branding decision. It is a system design decision.

## Concept Overview

Model selection chooses the right model for a task.

Routing chooses the right path at runtime.

Fallbacks define what happens when the primary path fails.

Together, they answer:

```text
Which model should handle this task?
When is a smaller model enough?
When does a harder case need a stronger model?
What happens if validation fails?
What happens if the provider is slow or down?
What happens if evidence is missing?
When should the system route to a human?
How do we measure quality, latency, and cost together?
```

The first principle:

> The right model is the simplest, fastest, cheapest model that meets the quality bar for the specific task.

Not the biggest model.

Not the cheapest model.

The right model for the job.

## Why It Exists

Imagine a company uses the most capable model for every request:

- simple intent classification;
- short field extraction;
- long report drafting;
- code analysis;
- policy Q&A;
- batch back-office jobs;
- high-risk customer communication.

The system may work, but it becomes:

- expensive;
- slow;
- fragile during provider incidents;
- hard to evaluate;
- hard to control;
- wasteful on simple tasks.

Another team switches to a cheaper model everywhere.

Cost improves, but quality collapses on hard cases:

- ambiguous policy questions;
- long-context synthesis;
- strict structured outputs;
- safety-sensitive refusal;
- complex tool-use planning.

Both teams missed the same idea:

```text
Different tasks need different routes.
```

This chapter exists because production AI systems are portfolios of model calls, not one giant prompt attached to one default model.

## Mental Model

Think of model routing like a hospital triage desk.

Not every case needs a specialist. Not every case should go to the fastest available person. Some cases are simple. Some need escalation. Some need tests first. Some need human judgment. Some should not be handled automatically at all.

The mental model:

```text
request
-> task classification
-> route selection
-> model call
-> validation
-> fallback or escalation
-> final system response
```

Routing depends on signals:

- task type;
- input length;
- context size;
- user tier;
- risk level;
- required output format;
- required latency;
- validation result;
- model confidence or uncertainty;
- retrieval quality;
- provider availability;
- cost budget;
- privacy requirement.

The model strategy should make easy work cheap, hard work reliable, and risky work reviewable.

## Internal Mechanics

Start from the task contract, not from the model catalog.

For each model task, define:

```text
task
input
output
quality bar
latency budget
cost budget
context requirement
risk level
validation rule
fallback path
human review rule
eval set
```

### Selection Dimensions

Choose models using the dimensions that matter for the workflow.

Quality:

- task pass rate;
- field-level accuracy;
- refusal accuracy;
- unsupported claim rate;
- structured output validity;
- tool-use reliability;
- citation faithfulness.

Latency:

- p50 latency;
- p95 latency;
- timeout rate;
- streaming support;
- batch throughput.

Cost:

- cost per request;
- cost per successful task;
- cost by route;
- cost after retries;
- cost including human review.

Capability:

- context window;
- multimodal support;
- structured output support;
- tool-calling behavior;
- code reasoning;
- long-context reliability;
- language support.

Operational constraints:

- rate limits;
- provider reliability;
- privacy requirements;
- regional deployment;
- logging and observability;
- contractual or compliance constraints.

Cost per successful task is more useful than cost per token.

A cheap model that fails often may be expensive after retries, escalations, support tickets, and user distrust.

### Routing

Routing selects a path at runtime.

Example routes:

| Route | Use Case |
|---|---|
| simple | classify obvious intent with a small model |
| standard | extract fields with reliable structured output |
| hard | analyze ambiguous case with stronger model |
| long-context | process large evidence packet |
| private | use isolated deployment for sensitive data |
| batch | run low-priority jobs cheaply in background |
| fallback | retry through alternate model or provider |
| human | route high-risk or unresolved cases to review |

Routing should be explicit.

Example:

```text
if task=intent_classification and input_tokens < 1,000 and risk=low:
  route=simple

if validation_failed twice:
  route=hard

if retrieved_evidence_missing:
  route=no_answer_or_retrieve_again

if action_risk=high:
  route=human_review
```

### Fallbacks

Fallback is not:

```text
try another random model and hope
```

Fallback should define:

```text
primary route
failure condition
fallback route
what changes
what must not change
final escalation
```

Examples:

```text
invalid JSON
-> retry same model with validation error
-> stronger structured-output model
-> human review after two failures
```

```text
provider timeout
-> backup provider with same output contract
-> degrade to "we are processing this" async state
```

```text
missing evidence
-> rerun retrieval with expanded query
-> ask clarification
-> refuse to answer without evidence
```

Fallback must not silently reduce safety.

Bad fallback:

```text
primary model refuses unsafe request
-> fallback model answers it
```

Good fallback:

```text
primary model refuses unsafe request
-> preserve refusal unless policy evaluator says refusal was incorrect
```

### Human Routes

Human review is a route, not a failure.

Use human review for:

- high-risk decisions;
- low confidence;
- policy exceptions;
- source conflicts;
- irreversible actions;
- sensitive customer communication;
- repeated validation failure;
- safety or compliance uncertainty.

The system should define:

```text
what human sees
what evidence is shown
what decision is requested
how approval is stored
what state transition follows
```

### Provider Failure

Production systems need provider failure plans.

Failure modes:

- timeout;
- rate limit;
- degraded latency;
- invalid outputs spike;
- provider outage;
- model version behavior shift;
- cost spike;
- regional availability problem.

Plan:

```text
retry policy
backup provider or model
queue for later processing
partial feature degradation
user-visible status
alerting
manual override
```

Do not design as if the model provider is always available.

## Real-World Example

Imagine a support AI system with several model tasks.

Tasks:

```text
intent classification
ticket field extraction
policy question answering
support reply drafting
manager escalation summary
```

Bad strategy:

```text
use strongest model for everything
```

Better route table:

| Task | Primary Route | Fallback | Human Route |
|---|---|---|---|
| Intent classification | small fast model | stronger model if low confidence | no |
| Ticket extraction | structured-output model | retry with validation error, then stronger model | if invalid twice |
| Policy Q&A | grounded answer model | no answer if evidence missing | if source conflict |
| Reply drafting | standard model with citations | stronger model for complex cases | sensitive cases |
| Refund approval | deterministic code + human | none | always when exception |

Example flow:

```text
customer says: I was charged twice and need a refund today

small model:
classifies issue_type=refund_request

structured-output model:
extracts urgency, evidence, missing invoice ID

database/API:
checks payment history

policy retrieval:
finds refund and duplicate-charge policy

router:
risk=high because refund and payment data
route reply draft to standard model
route final approval to human
```

The model strategy reduces cost without lowering accountability.

## Common Mistakes and Failure Modes

Mistake 1: One model for every task

Simple tasks become expensive. Hard tasks may still need special handling. Risky tasks may need review.

Mistake 2: Choosing models from demos instead of evals

Use real task evals, not impressive examples from unrelated use cases.

Mistake 3: Measuring cost without quality

Cheap failures are not cheap if they create retries, escalations, wrong answers, or support load.

Mistake 4: Measuring quality without latency

A perfect answer that arrives too late may fail the product workflow.

Mistake 5: Fallback changes safety behavior

A fallback model should not bypass refusal, permission, validation, or human review requirements.

Mistake 6: No provider failure plan

If provider latency spikes or rate limits hit, the product should degrade intentionally.

Mistake 7: Strong model hides poor workflow design

A better model may mask missing retrieval, weak schemas, vague tasks, or absent validators.

Mistake 8: Human review is bolted on late

Review should have state, evidence, approval records, and clear decision ownership.

## System Design Decisions

Define routes by task.

For each task:

```text
task name
risk level
input size
output contract
quality bar
latency budget
cost budget
primary model
fallback model or route
human review rule
eval set
```

Define router signals:

```text
task type
input token count
retrieved evidence quality
validation failure count
user tier
risk level
confidence
latency budget
provider status
privacy requirement
```

Define fallback rules:

```text
What failure triggers fallback?
Does fallback retry same model or switch route?
What constraints remain unchanged?
When does fallback stop?
When does human review start?
```

Define budgets:

```text
max cost per task
max p95 latency
max fallback rate
max invalid output rate
max human-review rate for low-risk cases
```

Define observability:

```text
model used
route selected
router signal
input/output tokens
latency
cost
validation result
fallback reason
human review result
```

The routing system should make model decisions visible. Hidden routing is hard to debug.

## Build Artifact

Create `model-routing-plan.md` for one AI system.

Choose a system such as:

- support assistant;
- policy Q&A assistant;
- invoice extraction workflow;
- operations action assistant;
- document analyst.

Use this template:

```text
Model Routing Plan

System:
User goal:

Tasks:
- task:
  input:
  output:
  risk level:
  quality bar:
  latency budget:
  cost budget:
  context requirement:
  primary route:
  fallback route:
  human review rule:
  eval set:

Router Signals:
- signal:
  possible values:
  affects route how:

Fallback Rules:
- failure condition:
  first fallback:
  second fallback:
  final escalation:
  safety constraints preserved:

Provider Failure Plan:
- failure:
  system response:
  user-visible behavior:
  alert:

Budgets:
- metric:
  limit:
  owner:

Observability:
- field:
  reason:
```

Example:

```text
Task: Ticket extraction
Risk level: medium
Quality bar: 95% valid structured output, 90% field accuracy
Latency budget: p95 under 2 seconds
Cost budget: under $0.01 per successful ticket
Primary route: structured-output model
Fallback route: retry with validation error, then stronger model
Human review: after two validation failures or refund/legal/security issue
Eval set: 200 labeled tickets with edge cases

Fallback:
invalid enum -> retry same route with validation error
invalid twice -> stronger model
invalid after stronger model -> human review
Safety preserved: model may not approve refund or expose account secrets
```

The artifact is complete when another engineer can implement model routing, measure it, and explain why each model is used.

## Active Recall Questions

1. Why is one model for every task usually a weak production strategy?
2. What dimensions matter when selecting a model for a task?
3. Why is cost per successful task better than cost per token?
4. What is model routing?
5. What signals can a router use?
6. Why must fallback rules preserve safety constraints?
7. When should human review be treated as a route?
8. What should be logged for routing observability?

## Summary

Model selection chooses the right model for a task. Routing chooses the right path at runtime. Fallbacks define what happens when the primary path fails.

Good AI systems use smaller, faster, cheaper routes for simple work; stronger routes for hard work; explicit fallbacks for failures; and human routes for risk, ambiguity, or accountability. They measure quality, latency, and cost together.

The most important rule:

> Choose models by task contract and measured performance, not hype or habit.

## Preview of Next Chapter

Chapter 17 chose models and routes.

Chapter 18 asks whether the model itself should be adapted.

Next, you will learn fine-tuning, adaptation options, and when not to tune. This matters because many teams reach for fine-tuning when the real problem is missing context, weak retrieval, vague schemas, poor routing, no eval set, or unstable source data.
