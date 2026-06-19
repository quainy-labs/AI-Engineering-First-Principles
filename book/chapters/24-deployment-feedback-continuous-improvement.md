# Chapter 24: Deployment, Feedback, and Continuous Improvement

## Real Problem

Prototype passes eval. Team ships to all users.

Within days:

- cost spikes;
- users ask unsupported questions;
- policy source changes;
- answer quality drops;
- one risky action almost executes;
- nobody knows whether to roll back.

Shipping AI system is not finish line. It is start of controlled learning.

## Bad Default Solution

Bad deployment:

- release to everyone;
- no feature flag;
- no rollback;
- no owner;
- no monitoring;
- no feedback triage;
- no eval schedule;
- no impact review.

This makes system fragile and politically expensive. Once trust breaks, users may not return.

## First Principles

Deployment should reduce uncertainty safely.

Good release asks:

- who gets access first?
- what can system do?
- what is monitored?
- what failure triggers rollback?
- who owns incidents?
- how is feedback reviewed?
- how are improvements validated?

AI deployment loop:

```text
prepare -> limited release -> observe -> evaluate -> improve -> expand or rollback
```

## System Decision

Use staged rollout:

1. internal builder testing;
2. small trusted user group;
3. limited workflow scope;
4. monitored pilot;
5. broader release;
6. continuous evaluation.

Release checklist:

```text
Eval gate passed:
Risk review complete:
Observability live:
Cost budget set:
Rollback path tested:
User instructions clear:
Feedback channel open:
Owner assigned:
```

## Minimal Build

Create `deployment-and-improvement-plan.md`:

```text
System:
Release owner:
Pilot users:
Scope:
Feature flag:
Eval gate:
Monitoring:
Cost budget:
Latency budget:
Risk controls:
Rollback trigger:
Feedback process:
Improvement cadence:
Impact review date:
```

## Failure Modes

Deployment fails when:

- release scope too broad;
- no rollback trigger;
- cost budget missing;
- users misunderstand system limits;
- feedback ignored;
- evals not rerun after changes;
- prompt/model changes unversioned;
- ownership unclear;
- product claims exceed system ability.

## Evaluation

During pilot, measure:

- task success;
- user acceptance;
- bad feedback rate;
- escalation rate;
- cost per task;
- p95 latency;
- safety incidents;
- rollback triggers;
- time saved vs baseline;
- quality improvement vs baseline.

Impact review:

```text
Baseline:
Current result:
Improvement:
New cost:
New risk:
User feedback:
Decision: expand / revise / pause / stop
```

Stopping is valid engineering decision.

## Improvement

Use feedback loop:

1. collect user feedback;
2. connect feedback to trace;
3. classify failure;
4. add eval case;
5. fix system;
6. rerun eval;
7. release safely.

Do not patch production from anecdote alone.

## Proof Artifact

Create `deployment-and-improvement-plan.md`.

Include:

- staged rollout;
- release checklist;
- rollback triggers;
- monitoring plan;
- feedback process;
- improvement cadence;
- impact review.

## Decision Checklist

- [ ] Pilot scope defined.
- [ ] Release owner named.
- [ ] Eval gate passed.
- [ ] Observability live.
- [ ] Cost/latency budgets set.
- [ ] Rollback trigger exists.
- [ ] Feedback becomes eval cases.
- [ ] Impact review scheduled.

## Active Recall

1. Why is deployment start of learning loop?
2. What belongs in release checklist?
3. Why use staged rollout?
4. What should trigger rollback?
5. Why can stopping be correct decision?

## Next

Now build capstone. Choose real workflow, apply full loop, and ship proof: problem, system, eval, risk review, demo, and public build note.
