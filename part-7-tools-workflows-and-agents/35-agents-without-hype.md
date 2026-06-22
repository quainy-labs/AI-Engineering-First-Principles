# Chapter 35: Agents Without Hype

Chapter 34 completed the action foundations:

```text
interfaces
-> tools
-> registries
-> workflows
-> state
-> approval
-> computer use
```

Now we can define agents without mythology.

An agent is not a product strategy. It is an architecture pattern. It is useful only when a bounded system needs to observe, decide, use tools, and continue under constraints.

## Concept Overview

An agent is a system that can loop through:

```text
observe
-> decide
-> act
-> observe result
-> continue, stop, or hand off
```

It may use tools, retrieve context, update state, ask for approval, and adapt next step based on what happened.

Useful agent systems need:

- bounded goal;
- minimal tool set;
- state;
- stop condition;
- max steps;
- cost and latency budget;
- approval gates;
- trace;
- evaluation.

The first principle:

> An agent is a controlled loop, not a free-roaming model.

If there is no bounded goal, no tool scope, no stop condition, and no trace, "agent" means uncontrolled automation.

## Why It Exists

A team says:

```text
We need an agent.
```

Ask:

```text
For what workflow?
With what tools?
Under what constraints?
Measured how?
What can it not do?
When must it stop?
```

If answer is unclear, they do not need an agent yet. They need system design.

Agentic systems can help when:

- task has multiple steps;
- next step depends on observed result;
- tools are needed;
- fixed workflow is too rigid;
- success criteria are clear;
- failure is recoverable;
- human oversight is possible.

Bad default:

- broad goal;
- many tools;
- no step limit;
- no approval;
- no trace;
- no eval;
- no stop condition;
- model decides everything.

This creates expensive, unpredictable automation.

## Mental Model

Think of an agent as a worker inside a fenced worksite.

The worker can inspect the current state, choose a next allowed step, use approved tools, and stop when done or blocked. The fence matters as much as the worker.

Mental model:

```text
bounded goal
-> allowed state
-> allowed tools
-> agent loop
-> stop condition
-> trace review
```

An agent should not invent its own authority.

It should know:

- what goal it is pursuing;
- what tools it may use;
- what tools are forbidden;
- what counts as success;
- what counts as failure;
- when human approval is needed;
- when to stop.

The loop is useful only when the boundaries are clear.

## Internal Mechanics

Agent systems combine pieces from earlier chapters.

### Goal

Goal should be specific.

Weak:

```text
Handle customer issue.
```

Better:

```text
Investigate support ticket T-104, find relevant policy, and draft a reply. Do not send message or issue refund.
```

Goal should include:

```text
target resource
desired output
forbidden actions
success condition
```

### Observation

Agent observes:

- user request;
- workflow state;
- tool results;
- retrieved documents;
- browser state;
- prior step output;
- approval state.

Observation must be scoped. Agent should not see data it is not allowed to use.

### Planning

Planning chooses next step.

For production, planning should be constrained:

```text
choose from allowed actions
respect max steps
prefer read before write
stop when evidence missing
ask approval before side effect
```

Do not let the model create arbitrary new tools or hidden steps.

### Acting

Agent acts through tools.

Tool calls should still follow Chapter 30 rules:

- schema validation;
- permission check;
- risk classification;
- approval gate;
- idempotency;
- audit log.

Agent loop does not remove tool safety. It increases need for it.

### Stop Conditions

Stop conditions are mandatory.

Examples:

```text
draft ready
evidence missing
tool error
human approval required
max steps reached
cost budget reached
unsafe request detected
user cancelled
```

Without stop conditions, agent can loop, spend, or act unpredictably.

### Budgets

Agent needs budgets:

```text
max steps
max tool calls
max cost
max time
max retries
max context tokens
```

Budgets are product controls.

### Human Approval

Approval gates remain explicit.

Agent may propose:

```text
send reply
create task
change status
issue refund
grant access
```

System decides which proposals require approval.

### Trace

Agent trace should include:

```text
goal
observations
plan steps
tool calls
tool arguments
tool results
state changes
approval requests
stop reason
cost
latency
```

If trace cannot be reviewed, agent cannot be trusted.

### Fixed Workflow vs Agent

Use fixed workflow when steps are known.

Use agent when next step depends on changing observations and cannot be cleanly pre-scripted.

Best agent systems often become less agentic over time. Once patterns are known, move them into deterministic workflows.

## Real-World Example

Use case:

```text
Investigate support ticket and draft reply.
```

Allowed:

```text
read_ticket
search_docs
read_customer_plan
draft_reply
```

Forbidden:

```text
send_email
issue_refund
delete_customer
grant_access
```

Agent boundary:

```text
Goal:
Investigate ticket T-104 and draft reply with citations.

Max steps:
6

Stop when:
- draft is ready
- evidence missing
- tool error
- max steps reached
- user approval needed

Approval:
Human sends final reply.
```

Good trace:

```text
1. read_ticket(T-104)
2. search_docs("API key access enterprise workspace")
3. read_customer_plan(customer_id)
4. draft_reply(evidence + account facts)
5. stop: draft_ready
```

Bad trace:

```text
1. search_docs
2. search_docs again
3. read unrelated billing data
4. send_email
```

Final reply might look fine, but path failed safety.

## Common Mistakes and Failure Modes

Mistake 1: Goal too broad

Broad goals force model to invent process and authority.

Mistake 2: Too many tools

Large tool sets increase wrong-tool use and unsafe action.

Mistake 3: No max steps

Agent can loop, repeat actions, or spend too much.

Mistake 4: No stop condition

Agent should know when to stop, refuse, escalate, or ask human.

Mistake 5: Approval skipped

Autonomy does not remove approval for risky actions.

Mistake 6: Trace hidden

Final answer cannot prove safe behavior. Trace matters.

Mistake 7: Tool output trusted blindly

Tool results may be partial, stale, failed, or permission-scoped.

Mistake 8: Agent used to hide unclear workflow

If humans cannot describe the desired workflow, an agent will not make it clear.

## System Design Decisions

Decide if agent is needed:

```text
Can fixed workflow solve this?
Are steps known?
Does next step depend on observation?
Are tools needed?
Can success be measured?
Can failure be recovered?
```

Define agent boundary:

```text
goal
user
allowed tools
forbidden tools
state available
max steps
max cost
max time
stop conditions
approval gates
escalation path
```

Define safety:

```text
Which actions are read-only?
Which actions write?
Which need approval?
Which are disallowed?
What user data is visible?
```

Define trace:

```text
What observations are logged?
What tool arguments are logged?
What state transitions are logged?
What stop reason is logged?
Who can review trace?
```

Define evaluation:

```text
task success
correct tool use
wrong-tool rate
unsafe action rate
loop rate
stop-condition accuracy
cost per task
trace clarity
```

Agent design starts with constraint, not ambition.

## Build Artifact

Create `agent-boundary.md`.

Use this template:

```text
Agent Boundary

Agent goal:
User:
Workflow:

Allowed Tools:
- tool:
  allowed use:

Forbidden Tools:
- tool:
  reason:

State Available:
- state:
  source:
  lifespan:

Limits:
- max steps:
- max cost:
- max time:
- max retries:

Stop Conditions:
- condition:
  stop reason:

Approval Gates:
- proposed action:
  required approver:
  evidence shown:

Trace Fields:
- field:
  reason:

Success Metric:
- metric:
  target:

Failure Metric:
- metric:
  threshold:

Escalation Path:
- condition:
  path:

Eval Cases:
- case:
  expected trace behavior:
```

Example:

```text
Agent goal:
Investigate support ticket and draft response.

Allowed tools:
read_ticket, search_docs, read_customer_plan, draft_reply

Forbidden tools:
send_email, issue_refund, delete_customer

Max steps:
6

Stop:
draft ready, evidence missing, tool error, max steps, approval required

Approval:
human sends final reply
```

Artifact complete when engineer can run agent with bounded autonomy and evaluate its trace.

## Active Recall Questions

1. What is the agent loop?
2. When is fixed workflow better than agent?
3. Why do agents need stop conditions?
4. Why should agent tool set be minimal?
5. What should be in an agent trace?
6. Why evaluate trace, not only final output?
7. Why do agents still need approval gates?
8. Why might a good agent system become less agentic over time?

## Summary

An agent is a controlled loop that observes, decides, acts, observes result, and continues until success, stop, or handoff. It is useful when next steps depend on observed results and fixed workflow is too rigid.

Agents need bounded goals, minimal tools, state, budgets, stop conditions, approval gates, traces, and evals. Without these, agents become uncontrolled loops.

The most important rule:

> Use agents for bounded uncertainty, not unclear product thinking.

## Preview of Next Chapter

Chapter 35 defined agents without hype.

Chapter 36 evaluates agents by trace.

Next, you will learn why final output is not enough. Agent evaluation must inspect tool choice, step efficiency, unsafe near-misses, approval behavior, loop behavior, cost, latency, and stop reasons.
