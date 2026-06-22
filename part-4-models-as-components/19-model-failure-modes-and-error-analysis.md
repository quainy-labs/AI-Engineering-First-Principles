# Chapter 19: Model Failure Modes and Error Analysis

Chapters 14-18 made models usable as bounded components:

```text
model boundaries
-> context design
-> model interfaces
-> routing and fallbacks
-> adaptation decisions
```

This chapter closes Part 4 with the improvement loop that prevents superstition.

Real model systems fail. The engineering skill is not pretending they will not fail. The engineering skill is turning vague failure into categories, causes, fixes, and regression tests.

## Concept Overview

Error analysis is the process of studying failures systematically.

It answers:

```text
What failed?
How often does it fail?
Which category does the failure belong to?
What likely caused it?
Which part of the system owns the fix?
Did the fix improve the same examples?
Did the fix break anything else?
```

A model failure may come from the model, but it may also come from:

- bad input;
- missing context;
- weak retrieval;
- unclear task;
- vague schema;
- conflicting sources;
- wrong route;
- unsafe fallback;
- missing validation;
- incorrect expected output;
- absent human review.

The first principle:

> "The model is unreliable" is not a diagnosis.

Good error analysis turns that sentence into:

```text
27% of refund tickets are classified as billing_question because the labels overlap.
The fix is to redefine the taxonomy and update eval examples, not change the model first.
```

## Why It Exists

Imagine your structured extraction system works on demo cases.

Then real support tickets arrive.

Failures appear:

- wrong issue type;
- missing urgency;
- invalid JSON;
- refund request classified as billing question;
- dangerous request not escalated;
- harmless request refused;
- fallback model behaves differently;
- answer cites the wrong source;
- tool proposal includes unsafe argument.

The team says:

> The model is unreliable.

Then they try:

- add "think carefully";
- change the model;
- add stronger instruction;
- increase temperature or reduce it randomly;
- test three examples;
- ship again.

This creates prompt churn. Maybe one case improves. Another breaks. Nobody knows why.

This chapter exists so learners build the habit:

```text
collect failures
-> categorize
-> identify cause
-> choose matching fix
-> rerun same eval set
-> inspect regressions
```

## Mental Model

Think of AI system improvement like debugging a distributed system.

You would not fix a payment bug by randomly rewriting frontend copy. You would inspect logs, inputs, state, API calls, validation, database records, and expected behavior.

Model systems deserve the same discipline.

The mental model:

```text
failure example
-> expected output
-> actual output
-> error category
-> likely cause
-> owning component
-> fix
-> before/after eval
-> regression check
```

The model is one component in a chain:

```text
user input
-> context assembly
-> retrieval
-> prompt/interface
-> model route
-> generation
-> validation
-> fallback
-> final response
```

Error analysis walks the chain before blaming any one piece.

## Internal Mechanics

Start by saving failures.

A useful failure record includes:

```text
example_id
user input
expected output
actual output
model used
route used
prompt/interface version
retrieved context
source IDs
tool results
validation result
fallback path
human review result
error category
likely cause
chosen fix
```

If failures are not saved, the team cannot learn from them.

### Error Taxonomy

Use categories so patterns become visible.

| Error Type | Meaning |
|---|---|
| Unsupported claim | Model states a fact not supported by context |
| Omission | Model misses required information |
| Wrong classification | Label is incorrect |
| Wrong extraction | Extracted field value is wrong |
| Invalid format | Output fails schema or parser |
| Over-refusal | Model refuses when it should answer |
| Under-refusal | Model answers when it should refuse |
| Context misuse | Model ignores or misreads supplied evidence |
| Source confusion | Model cites or uses wrong source |
| Retrieval miss | Expected source was not retrieved |
| Route error | Wrong model or path was selected |
| Fallback error | Fallback changed required behavior |
| Tool misuse | Wrong tool, unsafe argument, or bad tool plan |
| Policy error | Model violates rule that should be enforced |
| State error | Model or system uses stale/incorrect workflow state |

Every project should track the top error categories.

### Cause Analysis

The fix depends on the cause.

| Likely Cause | Better Fix |
|---|---|
| Missing context | Improve input, retrieval, or context assembly |
| Wrong source retrieved | Improve search, filters, ranking, or metadata |
| Conflicting sources | Add authority and conflict rules |
| Ambiguous labels | Redefine taxonomy or merge labels |
| Invalid format | Improve schema, validation, retry, or model interface |
| Unsupported claims | Require citations, evidence checks, or refusal |
| Policy decision inside model | Move policy to code or human review |
| Wrong route | Improve router signals or route table |
| Unsafe fallback | Preserve constraints across fallback |
| Model limitation | Use stronger model, split task, or adapt if justified |
| Bad expected output | Fix eval label or requirement |

Do not apply the same fix to every failure.

### Retesting

Use the same eval set before and after the fix.

Track:

- overall pass rate;
- pass rate by error category;
- invalid output rate;
- unsupported claim rate;
- refusal accuracy;
- escalation accuracy;
- retrieval success;
- route success;
- fallback success;
- cost;
- latency.

Also track regressions:

```text
cases fixed
cases newly broken
categories improved
categories worsened
```

AI system changes often trade one behavior for another.

### Choosing One Fix

Choose one high-impact fix at a time when possible.

Bad improvement cycle:

```text
change prompt
change model
change retrieval
change schema
change examples
rerun five examples
```

Now nobody knows which change helped.

Better cycle:

```text
top error: invalid urgency enum
likely cause: overlapping urgency definitions
fix: redefine urgency labels and schema descriptions
rerun same eval
inspect improvement and regression
```

### From Error Analysis to Evals

Each serious failure should become a regression example.

If the system once failed by answering from a stale policy, add a test:

```text
query includes stale and current policy
expected behavior: use current approved policy
forbidden behavior: cite stale policy
```

This turns pain into infrastructure.

## Real-World Example

Imagine a support ticket classifier.

Eval results:

```text
Eval set size: 200 tickets
Pass rate: 78%
Invalid output rate: 4%
Human-review accuracy: 70%
```

Top errors:

```text
1. refund_request classified as billing_question
   count: 22

2. security-sensitive tickets not escalated
   count: 9

3. invalid urgency enum
   count: 8
```

Weak response:

```text
Add "be more accurate" to the prompt.
```

Better error analysis:

```text
Error 1:
refund_request vs billing_question labels overlap.
Cause: taxonomy unclear.
Fix: redefine labels:
- refund_request when customer asks for money back or charge reversal
- billing_question when customer asks about invoice/payment explanation without reversal request
Add examples and eval cases.

Error 2:
security-sensitive tickets not escalated.
Cause: escalation rule missing.
Fix: add deterministic post-check for secret/password/security keywords and human-review route.

Error 3:
invalid urgency enum.
Cause: output contract allows only low/medium/high but examples used "urgent".
Fix: align examples with schema and validate enum.
```

Notice each fix matches a cause. The team does not just "make the model better." It improves the system.

## Common Mistakes and Failure Modes

Mistake 1: Reviewing only funny or dramatic failures

Memorable examples are not always representative. Count categories across a real sample.

Mistake 2: No expected output

Without expected behavior, disagreement becomes opinion instead of diagnosis.

Mistake 3: Changing eval set during comparison

If examples change between before and after, improvement is hard to trust.

Mistake 4: Assuming cause too quickly

An unsupported claim may come from missing context, bad retrieval, vague prompt, or model overreach. Inspect the chain.

Mistake 5: Prompt churn

Repeated wording changes without categorized errors and retesting creates illusion of progress.

Mistake 6: Ignoring regressions

A fix can improve invalid JSON while hurting refusal accuracy or increasing cost.

Mistake 7: Blaming model for retrieval failure

If the expected source never reached the context, the answer problem started before generation.

Mistake 8: Not turning failures into tests

If a serious failure is not added to evals, it may return later.

## System Design Decisions

Decide what failure data to collect.

```text
input
expected output
actual output
model route
prompt/interface version
retrieved source IDs
context packet
validation result
fallback path
human decision
latency
cost
```

Decide error taxonomy.

Start with:

```text
unsupported claim
omission
wrong classification
wrong extraction
invalid format
over-refusal
under-refusal
context misuse
source confusion
retrieval miss
route error
fallback error
tool misuse
policy error
state error
```

Decide improvement process:

```text
How often are failures reviewed?
Who labels errors?
How are disagreements resolved?
Which failures become eval cases?
How is before/after measured?
Who approves release?
```

Decide fix ownership:

```text
retrieval team
model interface owner
schema owner
router owner
workflow owner
data owner
human review owner
```

Decide regression policy:

```text
Which cases must never regress?
What score drop blocks release?
What risky categories require manual inspection?
```

The goal is a learning system, not endless prompt edits.

## Build Artifact

Create `model-error-report.md` for one model workflow.

Use this template:

```text
Model Error Report

System:
Workflow:
Model task:
Eval set version:
Eval set size:
Prompt/interface version:
Model route:

Overall Results:
- pass rate:
- invalid output rate:
- unsupported claim rate:
- refusal accuracy:
- escalation accuracy:
- average cost:
- p95 latency:

Top Error Categories:
- error type:
  count:
  representative examples:
  likely cause:
  owning component:
  chosen fix:

Regression Cases:
- case:
  why it matters:
  expected behavior:

Before/After Comparison:
- metric:
  before:
  after:
  changed:

New Regressions:
- case:
  likely cause:
  decision:

Next Experiment:
- hypothesis:
- change:
- success metric:
- rollback condition:
```

Example:

```text
Top Error Category:
error type: wrong classification
count: 22
representative examples: refund requests labeled billing_question
likely cause: overlapping label definitions
owning component: taxonomy/schema
chosen fix: redefine labels and update examples

Regression Case:
case: customer asks for charge reversal
expected behavior: issue_type=refund_request
why it matters: determines routing and human review
```

The artifact is complete when another engineer can understand what failed, why it likely failed, what fix was chosen, and how the fix will be evaluated.

## Active Recall Questions

1. Why is "the model is unreliable" not enough diagnosis?
2. What information should be saved with a failure example?
3. What are common model error categories?
4. Why should the same eval set be used before and after a fix?
5. Why can one fix improve one category and regress another?
6. When should a failure be blamed on retrieval instead of generation?
7. When should policy move from model behavior into deterministic code or human review?
8. Why should serious failures become regression tests?

## Summary

Model systems improve through error analysis, not superstition.

Collect failures. Label error types. Identify likely causes. Match fixes to causes. Rerun the same eval set. Inspect regressions. Turn serious failures into tests.

The model is one component in a larger chain. Many "model failures" are actually context, retrieval, schema, routing, fallback, policy, or evaluation failures.

The most important rule:

> Diagnose the failure before changing the model.

## Preview of Next Chapter

Part 4 made models usable as bounded, measurable components.

Part 5 moves into retrieval and context systems.

Next, Chapter 20 introduces embeddings and semantic retrieval. This matters because many model failures are really grounding failures: the system did not find the right information, did not rank it correctly, or did not pass the right context to the model.
