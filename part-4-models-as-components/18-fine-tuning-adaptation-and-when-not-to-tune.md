# Chapter 18: Fine-Tuning, Adaptation, and When Not to Tune

Chapter 17 chose models and routes. Now we decide whether the model itself should be adapted.

Fine-tuning is powerful. It is also overused when the real problem is bad data, bad retrieval, bad prompts, or missing validation.

## Real Problem

A team builds an internal support assistant.

It gives wrong answers about company policy. The team says:

> We need to fine-tune on our documents.

But the documents change every week. Some are stale. Some contradict each other. Some are drafts.

Fine-tuning on this corpus will bake confusion into the model. The real fix is source control, retrieval, and authority rules.

## Bad Default Solution

Bad solution:

- collect random examples;
- fine-tune model;
- test on a few happy paths;
- ship because answers "sound more like us."

Problems:

- no baseline;
- no eval split;
- no regression check;
- no rollback plan;
- no understanding of what tuning should change.

Fine-tuning without eval is expensive guessing.

## First Principles

Adaptation changes model behavior. It should be used when behavior should live in the model.

Good reasons to adapt:

- consistent output style;
- repeated domain patterns;
- specialized extraction behavior;
- smaller model matching larger model on narrow task;
- instruction-following for stable task format;
- reducing prompt length for high-volume workflow.

Bad reasons to adapt:

- missing current facts;
- private user/account data;
- fast-changing policies;
- weak retrieval;
- unclear schema;
- no eval set;
- trying to bypass validation;
- trying to make model responsible for business decisions.

Stable behavior can be tuned. Changing knowledge should usually be retrieved.

## Adaptation Options

| Option | Use When |
|---|---|
| Prompt examples | behavior is simple and eval gap is small |
| Better context | model needs facts |
| Retrieval | facts are external or changing |
| Structured output/schema | format is unreliable |
| Routing | hard cases need stronger model |
| Fine-tuning | stable repeated behavior needs to improve |
| Human review | risk remains high |

Fine-tuning is one option, not the default.

## Dataset Design

A tuning dataset needs:

- clear task definition;
- representative inputs;
- correct expected outputs;
- edge cases;
- refusal cases;
- human-reviewed labels;
- train/validation/test split;
- versioned dataset;
- eval baseline before tuning.

Bad labels teach bad behavior.

## Minimal Build

Create `adaptation-decision.md`:

```text
System:
Task:
Current baseline:
Failure categories:

Need current facts? yes/no
Need stable behavior? yes/no
Can schema/prompt/retrieval fix it? yes/no
Is eval set ready? yes/no
Is training data reviewed? yes/no

Decision:
- no tuning
- prompt/context change
- retrieval change
- routing change
- fine-tune

If fine-tune:
Training data:
Validation data:
Test data:
Success metric:
Regression checks:
Rollback plan:
```

## Failure Modes

Adaptation fails when:

- tuning data contains stale facts;
- train and test examples overlap;
- labels are inconsistent;
- eval only checks happy path;
- model memorizes format but misses meaning;
- new model improves average score but fails risky cases;
- rollback path is missing;
- tuned model hides system design problems.

## Evaluation

Before tuning:

- freeze eval set;
- measure baseline;
- categorize failures;
- define success metric.

After tuning:

- run same eval;
- run regression cases;
- compare cost and latency;
- inspect risky failures;
- compare against prompt/retrieval/routing alternatives.

Measure:

- task pass rate;
- field-level accuracy;
- invalid output rate;
- refusal accuracy;
- regression rate;
- cost per successful task.

## Improvement

Choose smallest effective fix:

- bad facts -> fix retrieval/data;
- bad format -> schema/validation;
- bad labels -> redefine task;
- easy cases too expensive -> small tuned model;
- stable domain pattern missed -> tune with reviewed examples;
- risky failures remain -> human review.

## Proof Artifact

Create `adaptation-decision.md`.

Include:

- baseline;
- failure categories;
- alternatives tried;
- tune/no-tune decision;
- dataset plan;
- success metrics;
- rollback plan.

## Decision Checklist

- [ ] Baseline exists.
- [ ] Failure categories are known.
- [ ] Retrieval/prompt/schema alternatives were considered.
- [ ] Training data is reviewed.
- [ ] Eval split exists.
- [ ] Regression cases exist.
- [ ] Rollback plan exists.

## Active Recall

1. When is fine-tuning the wrong fix?
2. What kind of knowledge should be retrieved instead of tuned?
3. Why need train/validation/test split?
4. Why compare tuning against prompt and retrieval fixes?
5. What does rollback mean for adapted models?

## Next

Next chapter studies failure. Even with good model choice and adaptation decisions, real systems need error analysis.
