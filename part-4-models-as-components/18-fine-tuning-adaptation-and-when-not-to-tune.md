# Chapter 18: Fine-Tuning, Adaptation, and When Not to Tune

Chapter 17 chose models and routes:

```text
task contract
-> model selection
-> routing
-> fallback
-> human review
```

This chapter asks whether the model itself should be adapted.

Fine-tuning can be powerful. It can also be the wrong fix when the real problem is bad data, weak retrieval, vague schemas, missing evals, poor routing, or absent validation.

Fine-tuning changes model behavior. It should be treated like a product and infrastructure decision, not a reflex.

## Concept Overview

Adaptation means changing how the model behaves for a task.

Fine-tuning is one adaptation method, but not the only one.

Adaptation options include:

- clearer prompt or interface;
- better examples in context;
- better retrieval;
- better schemas and validation;
- better routing;
- better tool design;
- human review;
- fine-tuning;
- distillation into a smaller model;
- preference tuning or other specialized training methods when appropriate.

The first principle:

> Stable behavior can be tuned. Changing knowledge should usually be retrieved.

Use adaptation when the model needs to behave differently in a stable, repeated way.

Do not use fine-tuning to hide missing facts, stale documents, unclear ownership, weak evals, or bad workflow design.

## Why It Exists

Imagine a team building an internal support assistant.

It gives wrong answers about company policy.

The team says:

> We need to fine-tune on our documents.

But the documents change every week. Some are stale. Some contradict each other. Some are drafts. Some are restricted. Some should never appear in customer-facing responses.

Fine-tuning on that corpus may bake confusion into the model.

The better fix may be:

- source authority rules;
- retrieval filtering;
- freshness metadata;
- canonical policy pages;
- better context assembly;
- citations;
- refusal when evidence is missing.

Another team has a different problem.

They run the same extraction task millions of times:

```text
input: short support ticket
output: strict issue_type, urgency, missing_fields
```

The task is stable. The labels are well-defined. The eval set exists. A smaller tuned model may match a larger model at lower cost.

That may be a good adaptation case.

This chapter exists so learners ask the right question:

> Is the failure caused by model behavior that should live in the model, or by missing system design around the model?

## Mental Model

Think of adaptation like training an employee.

Training helps when the job is stable, examples are clear, feedback is consistent, and success can be measured.

Training does not help if the company gives the employee outdated policies, contradictory instructions, missing customer data, unclear authority, and no definition of success.

The mental model:

```text
baseline
-> failure analysis
-> alternative fixes
-> adaptation decision
-> dataset design
-> evaluation
-> release
-> monitoring
-> rollback
```

Fine-tuning should come after baseline and error analysis, not before.

Before tuning, ask:

```text
Do we know the current failure categories?
Do we have an eval set?
Do we know what behavior should improve?
Is the target behavior stable?
Is the training data reviewed?
Have simpler fixes been tried?
Can we detect regressions?
Can we roll back?
```

If the answer is no, tuning is likely expensive guessing.

## Internal Mechanics

Start by separating behavior from knowledge.

Behavior is how the model performs a task:

- follows a format;
- applies labels consistently;
- extracts fields from messy text;
- writes in a style;
- refuses certain request types;
- produces concise summaries;
- maps inputs to a stable taxonomy.

Knowledge is information the system needs:

- current policy;
- customer account data;
- prices;
- legal terms;
- product limits;
- private company documents;
- fast-changing procedures.

Behavior may be tuned.

Changing knowledge should usually be retrieved from controlled sources.

Good reasons to adapt:

- consistent output style;
- repeated domain-specific formatting;
- specialized extraction behavior;
- classification into stable labels;
- smaller model matching larger model on a narrow task;
- reducing prompt length for a high-volume workflow;
- improving instruction-following for a fixed interface;
- improving performance on a stable set of edge cases.

Bad reasons to fine-tune:

- missing current facts;
- private user or account data;
- fast-changing company policy;
- weak retrieval;
- unclear schema;
- missing validation;
- no eval set;
- no baseline;
- trying to bypass permissions;
- trying to make the model own business decisions.

### Adaptation Options

Choose the smallest effective fix.

| Problem | Better First Fix |
|---|---|
| Missing current facts | Fetch from database/API |
| Wrong policy used | Fix retrieval, freshness, authority |
| Invalid output format | Schema, validation, retry |
| Ambiguous labels | Redefine taxonomy |
| Hard cases fail | Routing or stronger model |
| Easy cases too expensive | Smaller model or fine-tune |
| Style inconsistent | Prompt examples or tuning |
| Risk remains high | Human review |

Fine-tuning is most useful when examples teach stable behavior that the base model does not reliably perform.

### Baseline

Before adapting, measure current behavior.

Baseline should include:

- model used;
- prompt/interface version;
- routing path;
- eval dataset version;
- pass rate;
- failure categories;
- cost;
- latency;
- human-review rate.

Without baseline, you cannot tell whether tuning helped.

### Dataset Design

A tuning dataset needs:

- clear task definition;
- representative inputs;
- correct expected outputs;
- edge cases;
- refusal cases;
- escalation cases;
- hard negatives;
- human-reviewed labels;
- train/validation/test split;
- versioned dataset;
- label guidelines;
- quality checks.

Bad labels teach bad behavior.

If human reviewers disagree, do not rush to tune. Fix the task definition or label taxonomy first.

### Splits

Use separate data splits:

```text
training set:
used to adapt model

validation set:
used during development and selection

test set:
held back for final comparison
```

Do not train and test on the same examples.

Do not keep editing the test set until results look good.

### Regression

An adapted model can improve average score while breaking important cases.

Track:

- old cases that must keep working;
- refusal behavior;
- high-risk edge cases;
- rare labels;
- safety cases;
- long-tail formats;
- cost and latency changes.

### Release and Rollback

Treat adapted models like versioned artifacts.

Record:

```text
base model
adapted model version
training dataset version
validation dataset version
test dataset version
prompt/interface version
eval results
release date
rollback target
owner
```

Rollback means returning to a previous model route, prompt, or model version if quality regresses.

## Real-World Example

Imagine a high-volume ticket triage system.

Current system:

```text
model: strong general model
task: classify issue_type and urgency
volume: 2 million tickets/month
quality: good
cost: high
latency: acceptable but not ideal
```

The team wants to reduce cost.

Good adaptation question:

> Can a smaller adapted model match the larger model on this narrow classification task?

They build:

- 20,000 reviewed examples;
- stable label definitions;
- train/validation/test split;
- refusal and escalation cases;
- regression set for security, refund, and legal tickets;
- baseline for current route;
- cost and latency comparison.

Decision:

```text
Fine-tune or adapt smaller model for ticket classification only.
Keep policy Q&A and sensitive drafting on stronger route.
Human review remains for refund/legal/security cases.
```

This is a good use of adaptation because:

- task is narrow;
- behavior is stable;
- labels are reviewed;
- eval exists;
- cost goal is clear;
- fallback route exists;
- high-risk tasks are not delegated to the adapted model.

Bad adaptation version:

```text
fine-tune on all support documents so the model knows company policy
```

That would mix changing knowledge, stale policy, permission issues, and source authority into model weights. Retrieval and source control are better for that problem.

## Common Mistakes and Failure Modes

Mistake 1: Fine-tuning for changing knowledge

Policies, prices, account facts, and private documents usually belong in retrieval or APIs, not model weights.

Mistake 2: No baseline

Without baseline results, teams cannot prove the adapted model improved the system.

Mistake 3: No failure categories

If the team does not know what is failing, tuning may optimize the wrong behavior.

Mistake 4: Random training examples

Training data should represent real distribution, edge cases, refusals, and high-risk cases.

Mistake 5: Inconsistent labels

If humans label similar examples differently, the model will learn confusion.

Mistake 6: Train/test leakage

Testing on examples the model effectively trained on creates false confidence.

Mistake 7: Average score hides risky regression

A model may improve common cases and fail rare but important cases. Track category-level metrics.

Mistake 8: Tuned model bypasses system controls

Fine-tuning does not replace retrieval, permissions, validation, routing, or human approval.

## System Design Decisions

Before adapting, decide whether the failure belongs in the model.

Use this diagnostic:

```text
If failure is missing fact -> data/API/retrieval fix.
If failure is stale fact -> source freshness fix.
If failure is wrong source -> retrieval/ranking fix.
If failure is invalid format -> schema/validation fix.
If failure is ambiguous label -> taxonomy fix.
If failure is hard case only -> routing fix.
If failure is stable repeated behavior -> adaptation may help.
```

Define adaptation goal:

```text
Improve accuracy?
Reduce cost?
Reduce latency?
Improve style consistency?
Improve structured output reliability?
Make smaller model viable?
```

Define dataset plan:

```text
Where do examples come from?
Who labels them?
What guidelines exist?
How are disagreements resolved?
What cases are held out?
How is dataset versioned?
```

Define success metrics:

```text
task pass rate
field-level accuracy
invalid output rate
refusal accuracy
escalation accuracy
category-level regression
cost per successful task
p95 latency
human-review rate
```

Define release controls:

```text
Which route uses the adapted model?
What fallback remains?
What human review remains?
How is behavior monitored?
How is rollback triggered?
```

The safest adaptation is narrow, measured, and reversible.

## Build Artifact

Create `adaptation-decision.md` for one AI task.

Choose a task such as:

- support ticket classification;
- invoice field extraction;
- document section labeling;
- customer reply style rewriting;
- repeated compliance summary formatting.

Use this template:

```text
Adaptation Decision

System:
Task:
Current model route:
Current prompt/interface version:

Baseline:
- eval dataset:
- pass rate:
- invalid output rate:
- refusal accuracy:
- cost per successful task:
- p95 latency:

Failure Categories:
- category:
  count:
  examples:
  likely cause:

Adaptation Diagnostic:
- needs current facts: yes/no
- needs changing knowledge: yes/no
- needs stable behavior: yes/no
- schema/prompt fix possible: yes/no
- retrieval fix possible: yes/no
- routing fix possible: yes/no
- eval set ready: yes/no
- reviewed training data ready: yes/no

Decision:
- no tuning:
- prompt/context change:
- retrieval change:
- routing change:
- fine-tune/adapt:

If Adapt:
- training data:
- validation data:
- test data:
- label guidelines:
- success metrics:
- regression checks:
- release route:
- fallback route:
- rollback plan:

Final Rationale:
```

Example:

```text
Task: Support ticket issue_type classification

Baseline:
pass rate: 91%
invalid output rate: 2%
cost per successful task: high because strongest model used

Failure Categories:
- refund/legal/security edge cases need human review
- login_problem vs account_access labels overlap

Decision:
Do not tune yet.
First merge or clarify overlapping labels, update eval set, and test smaller model route.

If later adapting:
Use reviewed ticket examples after label taxonomy is stable.
Keep refund/legal/security escalation rule outside model.
```

The artifact is complete when another engineer can understand why tuning is or is not justified and what evidence would prove the decision worked.

## Active Recall Questions

1. When is fine-tuning the wrong fix?
2. What is the difference between stable behavior and changing knowledge?
3. Why should fast-changing policy usually be retrieved instead of tuned?
4. Why does adaptation require a baseline?
5. What should a tuning dataset contain?
6. Why are train/validation/test splits necessary?
7. Why can average improvement hide risky regression?
8. What does rollback mean for an adapted model?

## Summary

Fine-tuning is one adaptation option, not the default answer to model failure. It is useful when stable, repeated behavior should improve and when the team has clear task definitions, reviewed data, evals, baselines, regression checks, release controls, and rollback.

Do not tune missing facts, changing policies, private user data, weak retrieval, vague schemas, absent validation, or unclear business decisions into the model. Fix the system layer that owns the failure.

The most important rule:

> Tune behavior only after you understand the failure and prove simpler system fixes are not enough.

## Preview of Next Chapter

Chapter 18 decided when to adapt the model and when not to.

Chapter 19 closes Part 4 by studying failure directly.

Next, you will learn model failure modes and error analysis. This matters because even with good boundaries, context design, interfaces, routing, and adaptation decisions, real systems fail. The engineering skill is to turn "the model is unreliable" into specific error categories, causes, fixes, and regression tests.
