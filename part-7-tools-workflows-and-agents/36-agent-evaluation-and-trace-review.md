# Chapter 36: Agent Evaluation and Trace Review

Chapter 35 defined agents as bounded loops:

```text
observe
-> decide
-> act
-> observe
-> stop or continue
```

This chapter explains how to evaluate them.

Agents cannot be judged only by final answer. The path matters. A final answer can look correct while the agent used wrong tools, saw forbidden data, almost took unsafe action, or wasted ten steps.

## Concept Overview

Agent evaluation measures both outcome and trace.

It answers:

```text
Did agent achieve task?
Did it choose correct tools?
Did it use valid arguments?
Did it avoid forbidden tools?
Did it respect approvals?
Did it stop correctly?
Did it avoid loops?
Did it stay within budget?
Was trace understandable?
```

An agent trace records the path:

- goal;
- observations;
- decisions;
- tool choices;
- tool arguments;
- tool results;
- state changes;
- approval requests;
- errors;
- retries;
- stop reason;
- final output;
- cost and latency.

The first principle:

> Agent success equals final result plus safe path.

If final answer is right but path is unsafe, agent failed.

## Why It Exists

An agent returns correct final answer.

Trace shows:

- used wrong tool first;
- retried three unnecessary steps;
- saw forbidden data;
- almost submitted risky action;
- hid uncertainty;
- stopped only after max cost warning.

Final answer passed. System behavior failed.

Bad evaluation:

- check final response;
- ignore tool sequence;
- ignore cost;
- ignore unsafe near-misses;
- ignore loops;
- ignore human approval behavior.

This chapter exists because agentic systems can fail in the path before final answer shows any problem.

## Mental Model

Think of trace review like reviewing security camera footage of a task.

You do not only ask:

> Was package delivered?

You also ask:

> Did worker enter allowed building, use right key, avoid restricted rooms, follow process, and stop when blocked?

Mental model:

```text
task
-> trace
-> outcome check
-> path check
-> safety check
-> efficiency check
-> stop check
-> regression case
```

Trace review turns hidden autonomy into inspectable behavior.

## Internal Mechanics

Agent eval needs trace fields, rubric, task set, and metrics.

### Trace Fields

Minimum trace:

```text
trace_id
agent_goal
user_request
workflow_state
step_number
observation
decision
tool_selected
tool_arguments
tool_result
validation_result
approval_state
error
retry_count
stop_reason
final_output
cost
latency
```

For high-risk workflows, include:

```text
permission checks
risk classification
forbidden tool exposure
human approval evidence
state transitions
resource IDs
```

### Rubric

A trace rubric defines what good path means.

Example:

```text
Correct goal understood
Correct tools selected
No forbidden tool used
Arguments valid
Permission checked before read/write
Approval requested before side effect
No unnecessary repeated step
Stopped for missing evidence
Final output useful
Trace understandable
```

Rubric should include must-fail items.

Must-fail examples:

- used forbidden tool;
- executed write without approval;
- accessed unauthorized data;
- ignored stop condition;
- repeated side effect;
- exceeded max steps without safe stop.

### Task Set

Agent eval needs task categories:

- normal task;
- missing information;
- tool outage;
- forbidden action;
- prompt injection;
- ambiguous request;
- max-step trap;
- approval denied;
- permission denied;
- stale or conflicting evidence.

Only happy-path eval gives false confidence.

### Metrics

Measure:

```text
task success
correct tool choice
tool argument validity
unsafe action rate
approval accuracy
stop-condition accuracy
loop rate
unnecessary step rate
cost per task
p95 latency
trace completeness
human escalation accuracy
```

Track metrics by task category, not only average.

### Near-Misses

Near-miss means agent almost failed safely.

Examples:

- proposed forbidden tool but validator blocked it;
- selected send tool before approval gate caught it;
- retrieved private source but filter removed it;
- repeated action but idempotency prevented duplicate.

Near-misses matter. They show where defenses are working and where agent behavior is weak.

### Trace Review Loop

Review loop:

```text
run eval task
-> inspect trace
-> label failure category
-> identify layer
-> fix prompt/tool/registry/workflow/state
-> rerun same task
-> add regression case
```

Agent improvement should reduce unsafe path behavior, not just improve final text.

## Real-World Example

Agent:

```text
Investigate support ticket and draft response.
```

Task:

```text
Customer cannot access API keys.
```

Expected trace:

```text
1. read_ticket(T-104)
2. search_docs("enterprise API key access")
3. read_customer_plan(customer_id)
4. draft_reply(with citations)
5. stop: draft_ready
```

Bad trace with good final answer:

```text
1. search_docs("API key")
2. read_billing_status(customer_id)
3. search_docs("API key")
4. draft_reply
5. tool proposal: send_email
6. validator blocks send_email
7. stop: draft_ready
```

Final draft may be good.

Trace review finds:

- unnecessary billing access;
- repeated search;
- forbidden send proposal;
- near-miss caught by validator.

Fixes:

- remove billing tool from workflow;
- improve stop rule after draft;
- add eval case for draft vs send;
- improve tool descriptions.

## Common Mistakes and Failure Modes

Mistake 1: Final answer only

Agent can reach correct output through unsafe or wasteful path.

Mistake 2: Traces missing

If path is not recorded, agent cannot be evaluated properly.

Mistake 3: Near-misses ignored

Blocked unsafe proposals still reveal agent weakness.

Mistake 4: No max-step tests

Agents need tests where they must stop instead of looping.

Mistake 5: Tool outage not simulated

Real agents must handle connector failure.

Mistake 6: Approval behavior not tested

Tests must check denied, expired, and required approval paths.

Mistake 7: Cost ignored

An agent that solves task in 30 steps may be product failure.

Mistake 8: Trace unreadable

Trace must be useful to engineers, reviewers, and sometimes users.

## System Design Decisions

Define trace schema:

```text
What fields are logged?
Which fields are redacted?
Who can view trace?
How long is trace retained?
```

Define rubric:

```text
What counts as correct path?
What counts as unsafe path?
What must fail release?
What counts as near-miss?
```

Define eval categories:

```text
normal
missing info
tool failure
forbidden action
prompt injection
ambiguous request
max-step trap
approval denied
```

Define metrics:

```text
task success
tool accuracy
argument validity
unsafe action rate
approval accuracy
loop rate
cost
latency
trace completeness
```

Define release gate:

```text
zero forbidden tool execution
zero unapproved high-risk action
max loop rate
max cost per task
required trace completeness
```

Agent eval should block unsafe behavior before production users find it.

## Build Artifact

Create `agent-eval-rubric.md` and `agent-trace-review.md`.

Use this template for `agent-eval-rubric.md`:

```text
Agent Eval Rubric

Agent:
Workflow:
Task set version:

Trace Fields:
- field:
  reason:

Rubric:
- criterion:
  pass condition:
  fail condition:

Must-Fail Conditions:
- condition:
  reason:

Metrics:
- task success:
- tool accuracy:
- argument validity:
- unsafe action rate:
- approval accuracy:
- loop rate:
- cost:
- latency:
- trace completeness:
```

Use this template for `agent-trace-review.md`:

```text
Agent Trace Review

Trace ID:
Task:
Expected outcome:
Actual outcome:

Step Review:
- step:
  expected:
  actual:
  pass/fail:
  note:

Safety Review:
- forbidden tool used:
- approval respected:
- unauthorized data accessed:
- near-miss:

Efficiency Review:
- unnecessary steps:
- repeated actions:
- max steps:
- cost:
- latency:

Decision:
- pass:
- fail:
- regression case:
- fix:
```

Example must-fail:

```text
Condition: agent executes send_email without human approval
Reason: external communication side effect
```

Artifact complete when reviewer can judge both final output and path quality.

## Active Recall Questions

1. Why is final answer not enough for agent evaluation?
2. What fields belong in an agent trace?
3. What is a near-miss?
4. Why test approval denied and max-step cases?
5. Why should trace review inspect tool arguments?
6. What agent failures should block release?
7. Why measure cost per task for agents?
8. How can trace failures become regression tests?

## Summary

Agent evaluation must inspect outcome and path. A useful agent must complete task, choose correct tools, use valid arguments, respect approval, avoid forbidden access, stop correctly, stay within budget, and leave readable trace.

Trace review makes autonomy inspectable. It catches wrong-tool use, unsafe near-misses, loops, approval failures, and waste that final answer alone hides.

The most important rule:

> Judge agents by trace, not just final answer.

## Preview of Next Chapter

Part 7 built tools, workflows, and agents with control.

Part 8 moves into reliable production AI.

Next, Chapter 37 treats evaluation as product infrastructure. This matters because once AI systems have tools, workflows, and agents, evaluation cannot be a demo checklist. It must become repeatable release infrastructure that catches regressions before users do.
