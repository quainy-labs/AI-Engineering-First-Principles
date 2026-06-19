# Chapter 17: Model Selection, Routing, and Fallbacks

Chapter 16 defined the model interface. Now we choose which model should implement it.

Real systems rarely need one model for everything. They need a model strategy.

## Real Problem

A company uses the most capable model for every request:

- simple classification;
- short extraction;
- long report drafting;
- code analysis;
- policy Q&A;
- batch back-office jobs.

The system works, but it is:

- expensive;
- slow;
- fragile during provider incidents;
- hard to evaluate;
- hard to control.

Another team switches to a cheaper model everywhere. Cost improves, but quality collapses on hard cases.

Both teams missed the same principle: model choice is a system design decision.

## Bad Default Solution

Bad solution:

```text
MODEL=best_available_model
```

or:

```text
MODEL=cheapest_model
```

Problems:

- task difficulty varies;
- context needs vary;
- latency requirements vary;
- privacy requirements vary;
- output constraints vary;
- failure impact varies.

One default model hides tradeoffs.

## First Principles

Choose model by task contract, not hype.

Important dimensions:

- quality on task evals;
- cost per successful task;
- latency;
- context window;
- structured output reliability;
- tool-use reliability;
- multimodal support;
- privacy and deployment constraints;
- rate limits;
- provider reliability;
- observability support.

The right model is the cheapest, fastest, simplest model that meets the quality bar for the specific task.

## Routing

Routing means selecting a model path at runtime.

Common routes:

| Route | Example |
|---|---|
| simple | classify intent with small model |
| standard | extract fields with reliable structured-output model |
| hard | analyze ambiguous case with stronger model |
| long-context | summarize large evidence packet |
| private | use local or isolated deployment |
| fallback | retry with alternate model/provider |
| human | route to review when risk is high |

Routing input can include:

- task type;
- user tier;
- risk level;
- input length;
- retrieved evidence count;
- confidence;
- validation failure;
- latency budget;
- cost budget.

## Fallbacks

Fallback is not "try random other model."

Fallback should be explicit:

```text
Primary route:
Failure condition:
Fallback route:
Changed constraints:
Final escalation:
```

Examples:

- invalid JSON -> retry same model with validation error;
- repeated invalid JSON -> stronger structured-output model;
- missing evidence -> refuse or ask retrieval system for better sources;
- provider timeout -> backup provider;
- high-risk decision -> human review.

Fallback must not silently lower safety.

## Minimal Build

Create `model-routing-plan.md`:

```text
System:

Tasks:
1. task:
   input:
   output:
   quality bar:
   latency budget:
   cost budget:
   primary model:
   fallback:
   human review rule:
   eval set:

Router signals:
-

Provider failure plan:
-

Budget limits:
-
```

Example route table:

| Task | Primary | Fallback | Escalate When |
|---|---|---|---|
| Intent classification | small model | stronger model | low confidence |
| JSON extraction | structured-output model | retry + stronger model | validation fails twice |
| Policy answer | grounded model call | no answer | evidence missing |
| Refund approval | code/human | none | always human |

## Failure Modes

Model strategy fails when:

- no eval exists per route;
- cost is tracked without quality;
- quality is tracked without latency;
- fallback changes behavior silently;
- provider failure has no backup path;
- small model handles high-risk tasks;
- strong model hides poor workflow design;
- human review route is missing.

## Evaluation

Evaluate per route:

- pass rate;
- invalid output rate;
- refusal accuracy;
- cost per successful case;
- p50 and p95 latency;
- fallback frequency;
- human-review rate;
- regression by task type.

Do not compare models only on demos. Compare them on the system's real eval sets.

## Improvement

Improve model strategy by:

- splitting tasks;
- using smaller models for simple work;
- routing hard cases to stronger models;
- adding deterministic validators;
- adding retries with clear limits;
- reducing context size;
- caching stable outputs;
- escalating risky cases.

## Proof Artifact

Create `model-routing-plan.md`.

Include:

- task-to-model table;
- quality, cost, and latency budgets;
- fallback rules;
- human-review rules;
- route-specific evals.

## Decision Checklist

- [ ] Model choice is tied to task.
- [ ] Each route has evals.
- [ ] Cost is measured per successful task.
- [ ] Latency budget exists.
- [ ] Fallback behavior is explicit.
- [ ] Provider failure path exists.
- [ ] High-risk work can route to human.

## Active Recall

1. Why is one model for every task usually wrong?
2. What dimensions matter in model selection?
3. What is model routing?
4. Why should fallback rules be explicit?
5. Why measure cost per successful task?

## Next

Next chapter asks when to adapt a model. Fine-tuning can be useful, but it is often the wrong fix for missing context, weak schemas, or bad evals.
