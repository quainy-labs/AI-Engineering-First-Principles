# Chapter 37: Evaluation as Product Infrastructure

Part 7 built tools, workflows, and agents with control.

Part 8 moves into reliable production AI.

Once AI systems can retrieve knowledge, call tools, run workflows, and operate agent loops, evaluation cannot be a demo checklist. It must become repeatable product infrastructure that catches regressions before users do.

## Concept Overview

Evaluation asks:

> Does the system behavior meet the expected standard for real tasks?

Evaluation is not final QA. It is infrastructure that runs repeatedly across product changes.

AI eval has layers:

- unit eval for one component;
- retrieval eval for evidence quality;
- model eval for output behavior;
- tool eval for action safety;
- workflow eval for step/state correctness;
- agent trace eval for path quality;
- regression eval for old failures;
- human eval for expert judgment;
- online eval for real-world behavior.

The first principle:

> If AI behavior can change, evaluation must run again.

Behavior can change when model, prompt, corpus, index, tool schema, router, policy, or user behavior changes.

## Why It Exists

Your assistant worked last week.

Today it fails same question.

What changed?

- model version;
- prompt;
- retrieved documents;
- chunking;
- embedding model;
- index version;
- tool schema;
- policy source;
- route table;
- user behavior.

Without evaluation infrastructure, every change is guess.

Bad default:

- test manually before demo;
- ask few examples;
- ship if answers look good;
- collect user complaints;
- fix prompt reactively.

This does not scale.

Production AI needs evals that run repeatedly and block unsafe regressions.

## Mental Model

Think of evaluation like test infrastructure plus product quality radar.

Unit tests catch code regressions. AI evals catch behavior regressions. But AI evals must also test judgment-like behavior: refusal, grounding, tool choice, approval, cost, latency, and trace quality.

Mental model:

```text
real tasks
-> eval set
-> expected behavior
-> metrics
-> release gate
-> regression tracking
-> production feedback
-> updated eval set
```

Evaluation is not one artifact. It is a loop.

The loop:

```text
production failure
-> reviewed
-> converted to eval case
-> system fixed
-> eval rerun
-> release gate checked
```

If production failures do not enter evals, system repeats mistakes.

## Internal Mechanics

Build evals around tasks, not benchmarks.

### Eval Categories

Good eval set includes:

- normal cases;
- edge cases;
- should-refuse cases;
- adversarial cases;
- past failures;
- high-value tasks;
- permission-boundary cases;
- stale/conflicting-source cases;
- tool failure cases;
- human approval cases.

Each eval item should have:

```text
input
user role
expected behavior
forbidden behavior
required sources or tools
risk level
pass/fail criteria
```

### Eval Types

Unit eval:

```text
Does one model call produce valid structured output?
```

Retrieval eval:

```text
Does expected source appear in top results?
Are forbidden sources excluded?
```

Workflow eval:

```text
Does task follow correct state transitions?
Does approval happen before side effect?
```

Agent eval:

```text
Does trace show safe, efficient tool use and correct stop reason?
```

Regression eval:

```text
Do past failures stay fixed?
```

Human eval:

```text
Does expert reviewer agree output is acceptable?
```

Online eval:

```text
What happens with real users after release?
```

### Metrics

Choose metrics by task.

Common metrics:

```text
task success
output validity
field-level accuracy
retrieval recall
faithfulness
citation accuracy
refusal accuracy
unsafe action rate
approval accuracy
tool selection accuracy
trace completeness
cost per successful task
p95 latency
```

Avoid one average score hiding risk categories.

Track by category:

```text
normal
edge
should-refuse
high-risk
past failure
```

### Release Gates

Release gate defines minimum acceptable behavior.

Example:

```text
RAG assistant release gate:
- retrieval top-3 recall >= 90%
- citation accuracy >= 90%
- refusal accuracy >= 85%
- zero forbidden-source answers
- p95 latency < 6 seconds
- cost < $0.08 per answer
```

Tool workflow release gate:

```text
- zero unapproved write actions
- tool argument validity >= 98%
- approval accuracy >= 99%
- duplicate action rate = 0
```

Release gate makes quality explicit.

### Human Review Rubric

Human eval needs rubric.

Without rubric, reviewers disagree and scores drift.

Rubric fields:

```text
correctness
grounding
completeness
safety
tone
actionability
citation quality
approval correctness
```

Human review should include examples of pass/fail.

### Automated Judges

Automated judges can help, but they are not perfect.

Use them for:

- scalable review;
- first-pass scoring;
- rubric assistance;
- detecting obvious unsupported claims.

Do not blindly trust them for:

- high-stakes release decisions;
- subtle policy interpretation;
- safety-critical behavior;
- labels without calibration.

Meta-evaluate judges against human-reviewed examples.

### Eval Versioning

Version:

```text
eval set
expected outputs
rubric
model
prompt/interface
retriever
corpus/index
tool registry
workflow version
```

Without versions, eval history cannot explain improvement.

## Real-World Example

System:

```text
Support operations assistant
```

Main tasks:

- answer policy question;
- draft customer reply;
- create internal follow-up task;
- route refund request to human review.

Eval categories:

```text
normal support question
missing evidence
stale policy
forbidden source
tool write action
approval denied
past failure
prompt injection
```

Release gate:

```text
retrieval expected-source recall >= 90%
unsupported claim rate <= 3%
zero forbidden-source answers
zero unapproved external sends
tool argument validity >= 98%
p95 latency < 8 seconds
cost per successful task < target
```

Past production failure:

```text
agent drafted reply and proposed send_email instead of draft_customer_reply
```

That becomes regression case:

```text
User request: Tell customer we are looking into this.
Expected: draft reply only.
Forbidden: send_customer_email tool.
```

This is evaluation as infrastructure.

## Common Mistakes and Failure Modes

Mistake 1: Eval examples too easy

Easy evals produce false confidence.

Mistake 2: No should-refuse cases

System learns to answer everything.

Mistake 3: Past failures not added

Same failure returns later.

Mistake 4: Expected behavior vague

"Good answer" is not enough. Define exact pass/fail.

Mistake 5: Automated judge trusted blindly

Judge can be wrong, biased, or misaligned with product risk.

Mistake 6: Eval set changes every run

Before/after comparison becomes meaningless.

Mistake 7: Metrics ignore cost and latency

Quality must be product-viable.

Mistake 8: Release gate missing

Eval without decision rule becomes dashboard decoration.

## System Design Decisions

Define eval ownership:

```text
Who owns eval set?
Who reviews failures?
Who updates rubric?
Who approves release gate?
```

Define eval categories:

```text
normal
edge
should-refuse
adversarial
past failures
high-risk
permission boundary
```

Define metrics:

```text
task success
output validity
grounding
refusal
unsafe action
cost
latency
trace quality
```

Define release gate:

```text
required pass rate
must-pass cases
zero-tolerance failures
human review requirement
rollback rule
```

Define schedule:

```text
on prompt change
on model change
on corpus/index change
on tool schema change
before release
nightly or weekly
after production incident
```

Evaluation becomes infrastructure when it runs automatically and changes product decisions.

## Build Artifact

Create `evaluation-plan.md`.

Use this template:

```text
Evaluation Plan

System:
Main tasks:
Owners:

Eval Categories:
- category:
  purpose:
  target count:

Eval Item Schema:
- input:
- user role:
- expected behavior:
- forbidden behavior:
- required sources/tools:
- risk level:
- pass/fail rule:

Metrics:
- metric:
  target:
  category:

Release Gate:
- required pass rate:
- must-pass cases:
- zero-tolerance failures:
- human review required:
- rollback rule:

Human Review Rubric:
- criterion:
  pass:
  fail:

Regression Policy:
- production failure becomes eval:
- owner:

Eval Schedule:
- trigger:
  required evals:
```

Example:

```text
Zero-tolerance failure:
unapproved external send

Regression Policy:
Every P1/P2 production failure must become eval case before fix closes.
```

Artifact complete when release can be accepted or blocked based on eval results.

## Active Recall Questions

1. Why is evaluation product infrastructure?
2. What belongs in a release gate?
3. Why include past failures in evals?
4. Why can automated judges be risky?
5. What is regression eval?
6. Why should evals track cost and latency?
7. When should evals run?
8. Why should evals be versioned?

## Summary

Evaluation is repeatable infrastructure for measuring AI behavior across real tasks. It includes unit, retrieval, workflow, agent, regression, human, and online evals.

Production AI needs eval sets with normal cases, edge cases, refusals, adversarial inputs, past failures, and high-risk workflows. Release gates turn eval scores into product decisions.

The most important rule:

> Evals should decide releases, not decorate dashboards.

## Preview of Next Chapter

Chapter 37 created release gates.

Chapter 38 adds observability, traces, cost, and latency.

Next, you will learn how to see what happens in real use. Evals test known cases; observability shows actual requests, retrieved context, model calls, tool calls, failures, cost, latency, and feedback.
