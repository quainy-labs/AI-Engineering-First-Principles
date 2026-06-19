# Chapter 32: Workflow Orchestration

Chapter 31 organized tool access. This chapter connects tools into reliable work.

Real work is not one model call. It is a workflow with state, retries, approvals, and failure paths.

## Real Problem

An operations assistant must:

1. read request;
2. classify task;
3. retrieve policy;
4. check account status;
5. draft action plan;
6. ask approval;
7. execute tool;
8. notify user;
9. log result.

Workflow orchestration decides order, state, retries, fallbacks, and ownership of each step.

## Bad Default Solution

Bad solution:

> Let agent figure out steps.

This creates:

- hidden process;
- unpredictable loops;
- repeated tool calls;
- skipped approvals;
- no timeout;
- hard debugging;
- weak auditability.

For production, workflow should be explicit unless autonomy is clearly needed and evaluated.

## First Principles

A workflow is stateful sequence of steps.

Each step has:

- input;
- owner;
- output;
- success condition;
- failure condition;
- retry rule;
- timeout;
- next step.

Owner can be:

- deterministic code;
- retrieval;
- model;
- tool;
- human.

AI should own ambiguous steps. Code should own control flow where possible.

Durable execution means workflow can survive:

- timeout;
- process restart;
- tool failure;
- human delay;
- duplicate request;
- retry.

## System Decision

Represent workflow as graph:

```text
START
-> classify_request
-> retrieve_context
-> check_policy
-> draft_action
-> approval_required?
   -> yes: request_human_approval
   -> no: execute_tool
-> log_result
-> END
```

For each step, define:

```text
Step:
Owner:
Input:
Output:
Success:
Failure:
Retry:
Timeout:
Next:
```

## Minimal Build

Create `workflow-orchestration-map.md`:

```text
Workflow:
Trigger:
End states:

Steps:
1. name:
   owner:
   input:
   output:
   success condition:
   failure condition:
   retry rule:
   timeout:
   next:

Escalation paths:
Stop conditions:
State store:
Audit log:
Idempotency keys:
```

## Failure Modes

Workflow orchestration fails when:

- model controls every step;
- no stop condition exists;
- retries repeat side effects;
- failure path not designed;
- timeout missing;
- human approval not represented as state;
- logs only final output;
- workflow cannot resume after interruption.

## Evaluation

Test workflow paths:

- happy path;
- missing information;
- tool error;
- permission denied;
- human rejects approval;
- timeout;
- duplicate request;
- should-refuse request.

Metrics:

- task completion;
- safe stop rate;
- retry correctness;
- approval handling;
- duplicate-action prevention;
- trace completeness.

## Improvement

Improve orchestration by:

- making steps explicit;
- moving control flow into code;
- adding state machine;
- adding timeouts;
- adding human checkpoints;
- separating read and write steps;
- logging every transition;
- testing failure paths.

## Proof Artifact

Create `workflow-orchestration-map.md`.

Include:

- workflow graph;
- step definitions;
- owners;
- retry/timeout rules;
- escalation paths;
- stop conditions;
- trace examples.

## Decision Checklist

- [ ] Workflow trigger is clear.
- [ ] End states are clear.
- [ ] Each step has owner.
- [ ] Retry rules exist.
- [ ] Timeout rules exist.
- [ ] Stop conditions exist.
- [ ] Failure paths tested.

## Active Recall

1. Why should workflow be explicit?
2. What fields define workflow step?
3. When should model own a step?
4. Why can retries be dangerous?
5. What is durable execution?

## Next

Next chapter handles memory, state, and approval. Workflow needs to remember where it is and when human must decide.
