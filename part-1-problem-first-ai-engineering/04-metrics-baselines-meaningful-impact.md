# Chapter 4: Metrics, Baselines, and Meaningful Impact

Chapter 3 defined the system boundary.

Now we need proof.

An AI system can be impressive, demo well, and still fail to improve real work.

This chapter teaches how to define baseline, metrics, and evaluation before building too much.

> If you cannot measure what improved, you do not yet know whether the system helped.

## Concept Overview

Baseline is current performance before the AI system.

Metric is what you measure.

Target is the result you want.

Impact is the meaningful difference between baseline and new result after considering cost, risk, and user effort.

Simple version:

```text
impact = improvement - cost - added risk - added friction
```

Good AI engineering measures at least three things:

- work improvement;
- output quality;
- system cost/performance.

Without these, teams confuse demo quality with real impact.

## Why It Exists

A team says:

```text
The AI assistant improved productivity.
```

Maybe.

But what changed?

- Time per task?
- Error rate?
- Review burden?
- Customer response time?
- Cost per task?
- User trust?
- Escalation rate?
- Risk?

If the team cannot answer, the claim is not engineering proof.

User excitement is useful. It is not enough.

Demos show possibility. Metrics show whether the system deserves to exist.

AI systems especially need measurement because they can look useful while quietly adding problems:

- faster drafts but more mistakes;
- more answers but fewer correct refusals;
- cheaper support but worse trust;
- impressive automation but more human review;
- high usage but low real value.

Metrics exist to keep the system honest.

## Mental Model

Use this map:

```text
current workflow
-> baseline
-> target
-> eval examples
-> AI-assisted workflow
-> measured result
-> impact decision
```

The key question:

```text
Compared to what?
```

Every AI project needs a comparison point.

For each workflow, define:

```text
Before:
- how long does it take?
- how often does it fail?
- how much does it cost?
- what quality is acceptable?

After:
- what changed?
- what got better?
- what got worse?
- what new risk appeared?
```

If you skip baseline, any improvement claim floats in the air.

## Internal Mechanics

Metrics work best in three layers.

### 1. Work Metrics

Work metrics measure whether the workflow improved.

Examples:

- time per task;
- first response time;
- review time;
- tickets handled;
- queue length;
- handoff count;
- manual search time;
- completion rate.

Work metrics answer:

```text
Did the user's work get better?
```

### 2. Quality Metrics

Quality metrics measure whether output is good.

Examples:

- field extraction accuracy;
- citation accuracy;
- answer faithfulness;
- policy compliance;
- human acceptance rate;
- invalid output rate;
- refusal accuracy;
- unsafe action rate.

Quality metrics answer:

```text
Can users trust the output?
```

### 3. System Metrics

System metrics measure whether the system is viable.

Examples:

- latency;
- cost per task;
- token/API cost;
- retry rate;
- failure rate;
- escalation rate;
- cache hit rate;
- uptime.

System metrics answer:

```text
Can this run in the real world?
```

Good AI products need all three.

Fast but wrong is not useful.

Accurate but too slow may not be used.

Cheap but risky may not be acceptable.

### Eval Examples

Metrics need examples.

Start with a small eval set:

```text
10 easy cases
10 normal cases
5 edge cases
5 should-refuse or should-escalate cases
```

For each example, define expected behavior before testing the model.

This protects against demo bias.

## Real-World Example

Return to support reply drafting.

Vague claim:

```text
AI makes support faster.
```

Better measurement plan:

```text
Workflow: draft support replies for repeated product questions
User: support teammate
Baseline: median first draft time = 12 minutes
Target: median first draft time < 5 minutes
Quality target: citation accuracy > 90%
System target: cost per draft < $0.10, p95 latency < 8 seconds
Risk guardrail: no final replies sent without human approval
```

Baseline table:

| Metric | Current Baseline | Target | How Measured |
|---|---:|---:|---|
| First draft time | 12 min | <5 min | 20 past tickets |
| Citation accuracy | unknown | >90% | reviewer labels |
| Human acceptance | unknown | >70% | accepted drafts |
| Cost per draft | $0 AI cost | <$0.10 | API logs |
| Unsafe send rate | 0 | 0 | audit log |

Unknown baseline is not failure.

But it must become a measurement task:

```text
First project step: label 20 past tickets and measure current draft time.
```

Now the team can learn.

If draft time improves but citation accuracy drops, the system is not ready.

If quality improves but cost is too high, model routing or retrieval must improve.

If users reject most drafts, the interface or task boundary is wrong.

## Common Mistakes and Failure Modes

### Mistake 1: Measuring Vibes

Bad:

```text
Users like it.
```

Better:

```text
70% of drafts accepted with minor edits.
```

### Mistake 2: No Baseline

If you do not know current performance, you cannot prove improvement.

### Mistake 3: One Metric Only

If metric is only speed, system may become fast and wrong.

If metric is only accuracy, system may become slow and expensive.

### Mistake 4: Rewarding Bad Behavior

Metric:

```text
answers more questions
```

Bad result:

```text
model answers unsupported questions instead of refusing.
```

Better metric:

```text
answer accuracy + groundedness + refusal accuracy
```

### Mistake 5: Easy Eval Set

If eval examples are too easy, system will pass before it is useful.

Include:

- edge cases;
- missing information;
- conflicting context;
- should-refuse cases.

### Mistake 6: Ignoring Cost and Risk

Impact is not just quality.

A system can improve output and still be too expensive, slow, risky, or hard to operate.

## System Design Decisions

Before building, decide:

```text
What baseline matters?
What work metric proves improvement?
What quality metric protects trust?
What system metric protects viability?
What risk guardrail must never fail?
What examples will test normal behavior?
What examples will test edge behavior?
What examples should refuse or escalate?
How will feedback become eval data?
```

Use this metric map:

| Question | Metric Type | Example |
|---|---|---|
| Did work improve? | work | time per draft |
| Is output good? | quality | citation accuracy |
| Can it run? | system | latency/cost |
| Is it safe? | guardrail | unsafe action rate |
| Is it adopted? | product | human acceptance |

Metric choice shapes system design.

If citation accuracy matters, system must preserve sources.

If latency matters, architecture must support caching or smaller routes.

If refusal matters, eval set must include unsupported cases.

## Build Artifact

Create `baseline-and-eval-plan.md`.

Template:

```text
Workflow:
Primary user:
Current baseline:

Work metric:
- metric:
- current:
- target:
- measurement method:

Quality metric:
- metric:
- current:
- target:
- measurement method:

System metric:
- metric:
- current:
- target:
- measurement method:

Risk guardrail:
- metric:
- target:

Eval set:
- easy cases:
- normal cases:
- edge cases:
- should-refuse/escalate cases:

First improvement hypothesis:
```

Worked example:

```text
Workflow: support reply drafting
Primary user: support teammate
Current baseline: median first draft time = 12 minutes

Work metric:
- metric: first draft time
- current: 12 minutes
- target: <5 minutes
- measurement method: 20 real tickets

Quality metric:
- metric: citation accuracy
- current: unknown
- target: >90%
- measurement method: reviewer labels

System metric:
- metric: cost per draft
- current: $0 AI cost
- target: <$0.10
- measurement method: token/API logs

Risk guardrail:
- metric: final reply sent without human approval
- target: 0

Eval set:
- easy cases: repeated FAQ tickets
- normal cases: product troubleshooting tickets
- edge cases: ambiguous policy tickets
- should-refuse/escalate cases: refund approval, account secrets

First improvement hypothesis:
- better retrieval metadata will improve citation accuracy
```

Acceptance criteria:

- baseline exists or measurement plan exists;
- at least one work metric exists;
- at least one quality metric exists;
- at least one system metric exists;
- risk guardrail exists;
- eval set includes edge and refusal/escalation cases;
- first improvement hypothesis is written.

## Active Recall Questions

1. What is baseline?
2. Why is demo not proof?
3. Why should impact subtract cost, risk, and friction?
4. What are work, quality, and system metrics?
5. Why include should-refuse or should-escalate cases?
6. How can a metric reward bad behavior?
7. Why does metric choice affect architecture?

## Summary

AI systems need proof, not vibes.

The durable idea:

```text
baseline -> target -> eval set -> measured result -> impact decision
```

Meaningful impact requires:

- current baseline;
- work metric;
- quality metric;
- system metric;
- risk guardrail;
- eval examples;
- improvement hypothesis.

An AI system is not valuable because it uses AI.

It is valuable when it improves real work enough to justify cost and risk.

## Preview of Next Chapter

Part 1 answered four questions:

```text
What work should improve?
Where does AI belong?
What boundary should the system have?
How will improvement be measured?
```

Next, Part 2 starts software foundations.

Once problem, boundary, and metrics are clear, the learner needs to understand where AI components live inside real applications: APIs, services, workers, queues, auth, permissions, files, storage, and background jobs.
