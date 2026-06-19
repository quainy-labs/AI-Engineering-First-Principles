# Chapter 37: Evaluation as Product Infrastructure

Parts 6 and 7 built interfaces, tools, workflows, and agents. Now those systems must survive production.

Evaluation is not final QA. It is product infrastructure.

## Real Problem

Your assistant worked last week. Today it fails same question.

What changed?

- model version;
- prompt;
- retrieved documents;
- chunking;
- tool schema;
- policy source;
- user behavior.

Without evaluation infrastructure, every change is guess.

## Bad Default Solution

Bad solution:

- test manually before demo;
- ask few examples;
- ship if answers look good;
- collect user complaints;
- fix prompt reactively.

This does not scale.

Production AI needs evals that run repeatedly and block unsafe regressions.

## First Principles

Evaluation answers:

> Does system behavior meet expected standard for real tasks?

AI eval has layers:

- unit eval: one component;
- integration eval: full workflow;
- regression eval: old failures stay fixed;
- human eval: expert judgment;
- online eval: real user behavior.

Eval data should reflect work, not benchmark theater.

Good eval set includes:

- normal cases;
- edge cases;
- should-refuse cases;
- adversarial cases;
- past failures;
- high-value tasks.

## System Decision

Define release gate:

```text
System version:
Eval set:
Required pass rate:
Must-pass cases:
Allowed failure types:
Human review required:
Rollback rule:
```

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

Release gate makes quality explicit.

## Minimal Build

Create `evaluation-plan.md`:

```text
System:
Main tasks:
Eval categories:
- normal:
- edge:
- should-refuse:
- adversarial:
- past failures:

Metrics:
- task success:
- output validity:
- source faithfulness:
- refusal accuracy:
- unsafe action rate:
- cost:
- latency:

Release gate:
Human review:
Regression policy:
Eval schedule:
```

## Failure Modes

Evaluation fails when:

- eval examples are too easy;
- expected outputs are vague;
- no should-refuse cases exist;
- human judgment is inconsistent;
- automated judge trusted blindly;
- eval set changes every run;
- metrics ignore cost/risk;
- production failures are not added back.

## Evaluation

Evaluate eval system itself:

- Does eval catch known failures?
- Does it include real user examples?
- Does it detect regressions?
- Does it measure cost and latency?
- Does human review have rubric?
- Does release gate block bad versions?

Meta-eval matters. Bad eval gives false confidence.

## Improvement

Improve eval infrastructure by:

- adding failures from production;
- writing clearer expected behavior;
- splitting metrics by category;
- adding human rubric;
- versioning eval set;
- tracking before/after results;
- adding release gate.

## Proof Artifact

Create `evaluation-plan.md`.

Include:

- eval categories;
- metrics;
- release gate;
- human review rubric;
- regression policy;
- eval schedule.

## Decision Checklist

- [ ] Eval set has real examples.
- [ ] Should-refuse cases included.
- [ ] Past failures included.
- [ ] Metrics cover quality, cost, latency, risk.
- [ ] Release gate exists.
- [ ] Human review rubric exists.
- [ ] Eval set is versioned.

## Active Recall

1. Why is evaluation product infrastructure?
2. What belongs in release gate?
3. Why include past failures?
4. Why can automated judges be risky?
5. What is regression eval?

## Next

Next chapter adds observability, traces, cost, and latency. Evals test known cases; observability shows what happens in real use.
