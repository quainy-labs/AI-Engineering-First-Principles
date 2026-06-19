# Chapter 20: Agents Without Hype

## Real Problem

A team says:

> We need an agent.

Ask:

> For what workflow, with what tools, under what constraints, measured how?

If answer is unclear, they do not need agent yet. They need system design.

Agentic systems can be useful when task requires multiple steps, tool use, adaptation, and feedback. They are harmful when used to hide unclear workflow behind autonomy.

## Bad Default Solution

Bad agent:

- broad goal;
- many tools;
- no step limit;
- no approval;
- no trace;
- no eval;
- no stop condition;
- model decides everything.

This creates expensive, unpredictable automation.

Agent is not product strategy. Agent is architecture pattern.

## First Principles

An agent is system that can:

1. observe state;
2. decide next step;
3. use tools;
4. evaluate result;
5. continue until goal, stop, or handoff.

Core loop:

```text
observe -> plan -> act -> observe -> decide
```

Useful agent needs:

- bounded goal;
- tool set;
- state;
- stop condition;
- cost/latency budget;
- approval gates;
- trace;
- evaluation.

Without these, "agent" means uncontrolled loop.

## System Decision

Use agent when:

- task cannot be solved by fixed workflow;
- next step depends on observed result;
- tools are needed;
- success criteria are clear;
- failures are recoverable;
- human oversight possible.

Prefer fixed workflow when:

- steps are known;
- policy strict;
- action high-risk;
- output must be predictable;
- evaluation is weak.

Decision:

```text
Can fixed workflow solve it?
If yes, use workflow.
If no, define bounded agent loop.
```

## Minimal Build

Create `agent-boundary.md`:

```text
Agent goal:
User:
Allowed tools:
Forbidden tools:
Max steps:
Max cost:
Max time:
State available:
Stop conditions:
Approval gates:
Trace fields:
Success metric:
Failure metric:
Escalation path:
```

Example:

```text
Agent goal: investigate support ticket and draft response
Allowed tools: search_docs, read_ticket, draft_reply
Forbidden tools: send_email, issue_refund
Max steps: 6
Stop: draft ready, evidence missing, tool error, max steps
Approval: human sends final reply
```

## Failure Modes

Agents fail when:

- goal too broad;
- tools too powerful;
- no max steps;
- no budget;
- agent repeats same action;
- agent trusts tool output without validation;
- trace hidden;
- human cannot interrupt;
- eval checks final answer only.

## Evaluation

Agent eval must inspect trace.

Measure:

- task success;
- correct tool choice;
- step efficiency;
- stop-condition accuracy;
- unsafe action rate;
- approval handling;
- cost per task;
- latency;
- trace clarity.

Test categories:

- normal task;
- missing information;
- tool failure;
- ambiguous request;
- forbidden action;
- adversarial instruction;
- max-step condition.

## Improvement

Improve agent by:

- narrowing goal;
- reducing tools;
- adding planner constraints;
- adding max steps;
- adding approval;
- improving tool docs;
- adding trace review;
- converting repeated patterns into fixed workflow.

Best agent systems often become less agentic over time. Once pattern is known, move it into deterministic workflow.

## Proof Artifact

Create `agent-boundary.md`.

Include:

- bounded goal;
- allowed/forbidden tools;
- budgets;
- stop conditions;
- approval gates;
- trace fields;
- eval cases.

## Decision Checklist

- [ ] Fixed workflow considered first.
- [ ] Agent goal bounded.
- [ ] Tool set minimal.
- [ ] Max steps set.
- [ ] Stop conditions clear.
- [ ] Approval gates exist.
- [ ] Trace evaluated.

## Active Recall

1. What is agent loop?
2. When is fixed workflow better than agent?
3. Why do agents need stop conditions?
4. Why evaluate trace, not just final output?
5. Why might good agent system become less agentic over time?

## Next

Part 6 focuses on production. A useful agent or workflow still needs eval infrastructure, observability, cost control, security, deployment, and continuous improvement.
