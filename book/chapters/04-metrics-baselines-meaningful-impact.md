# Chapter 4: Metrics, Baselines, and Meaningful Impact

## Real Problem

A team says:

> AI assistant improved productivity.

Maybe true. Maybe not.

What changed?

- Time per task?
- Error rate?
- User satisfaction?
- Throughput?
- Cost?
- Quality?
- Risk?
- Learning speed?

Meaningful impact needs measurement before and after.

## Bad Default Solution

Bad solution:

- build demo;
- ask users if they like it;
- show impressive examples;
- claim productivity gain.

This is not enough.

User excitement is useful signal, not proof. Demos show possibility, not reliability.

## First Principles

Baseline is current performance before new system.

Metric is chosen measurement of system quality or work improvement.

Impact is difference between baseline and new result, adjusted for cost and risk.

```text
impact = improvement - cost - new risk
```

AI system can be impressive and still not worth using if it:

- saves 2 minutes but adds review burden;
- improves draft speed but increases errors;
- lowers support cost but leaks private data;
- answers fast but cannot be trusted;
- automates task nobody needed.

## System Decision

Use three metric types:

### Work Metrics

Measure workflow improvement:

- time saved;
- tasks completed;
- queue length;
- handoff reduction;
- first response time;
- review time.

### Quality Metrics

Measure output:

- accuracy;
- citation correctness;
- extraction field accuracy;
- policy compliance;
- hallucination rate;
- human acceptance rate.

### System Metrics

Measure engineering constraints:

- latency;
- cost per task;
- failure rate;
- invalid output rate;
- retry rate;
- escalation rate.

Good project uses at least one metric from each type.

## Minimal Build

Create baseline table:

| Metric | Current baseline | Target | How measured |
|---|---:|---:|---|
| Draft time | 12 min | 5 min | 20 real tickets |
| Citation accuracy | unknown | 90% | reviewer labels |
| Cost per task | $0 | <$0.10 | token/API logs |
| Human acceptance | unknown | 70% | accepted drafts |

Unknown baseline is allowed only if first project step measures it.

## Failure Modes

Metrics fail when:

- target is vague;
- metric rewards bad behavior;
- sample examples are too easy;
- evaluation data differs from real use;
- latency and cost ignored;
- user trust not measured;
- risk introduced but not counted.

Example:

If metric is "answers more questions," model may answer uncertain questions instead of refusing. Better metric includes groundedness and correct refusal.

## Evaluation

Build tiny eval set before building system:

- 10 easy cases;
- 10 normal cases;
- 5 edge cases;
- 5 should-refuse cases.

For each example, define expected behavior.

This protects against demo bias.

## Improvement

After first measurement, improve one bottleneck:

- bad retrieval -> improve metadata/chunking;
- invalid JSON -> strengthen schema/validation;
- slow latency -> cache or reduce context;
- high hallucination -> require citations or refusal;
- low acceptance -> inspect user edits.

Improve from evidence, not vibes.

## Proof Artifact

Create `baseline-and-eval-plan.md`.

Include:

- work metric;
- quality metric;
- system metric;
- baseline values;
- target values;
- 30-example eval set design;
- first improvement hypothesis.

## Decision Checklist

- [ ] Current baseline exists or measurement plan exists.
- [ ] At least one work metric defined.
- [ ] At least one quality metric defined.
- [ ] At least one system metric defined.
- [ ] Eval examples include edge cases.
- [ ] Refusal behavior is measured.
- [ ] Cost and latency are included.

## Active Recall

1. What is baseline?
2. Why is demo not proof?
3. Why should impact subtract cost and risk?
4. What are three metric types?
5. Why include should-refuse cases in eval set?

## Next

Part 2 starts with information. Once problem and metrics are clear, system needs reliable data, documents, and state.

