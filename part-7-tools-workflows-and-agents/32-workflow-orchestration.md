# Chapter 32: Workflow Orchestration

Chapter 31 organized tools across systems:

```text
connectors
-> registries
-> allowed tool sets
-> workflow-scoped exposure
```

This chapter connects tools into reliable work.

Real work is not one model call. It is a workflow with steps, state, retries, approvals, timeouts, failure paths, and audit records.

Orchestration answers:

> What should happen, in what order, under what state, with what fallback?

## Concept Overview

Workflow orchestration is the design of multi-step work.

It defines:

- trigger;
- steps;
- owners;
- state;
- retries;
- timeouts;
- approvals;
- failure paths;
- compensation;
- end states;
- audit logs.

Example workflow:

```text
customer issue arrives
-> classify request
-> retrieve policy
-> check account status
-> draft action plan
-> ask human approval
-> execute tool
-> notify user
-> log result
```

The first principle:

> Let model help with ambiguous steps. Let system own workflow control.

Models can classify, summarize, draft, and propose. Code should usually own step order, state transitions, retries, approvals, and stop conditions.

## Why It Exists

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

Bad default:

```text
Let agent figure out steps.
```

This creates:

- hidden process;
- unpredictable loops;
- repeated tool calls;
- skipped approvals;
- no timeout;
- weak auditability;
- hard debugging;
- unsafe retry behavior.

For production, workflow should be explicit unless autonomy is clearly needed and evaluated.

This chapter exists because useful AI systems must survive normal software failure: timeout, duplicate request, user delay, tool error, permission denial, partial success, and process restart.

## Mental Model

Think of orchestration like a checklist with state.

Each step knows:

```text
what input it needs
who owns it
what output it creates
what success means
what failure means
what can retry
what must stop
what happens next
```

Mental model:

```text
trigger
-> step
-> state transition
-> next step
-> approval or tool
-> success/failure end state
```

Workflow should be inspectable:

```text
Where is it now?
What happened before?
What is it waiting for?
Who must decide?
Can it retry?
Can it be cancelled?
What side effects already happened?
```

If system cannot answer these questions, it does not have orchestration. It has a hidden chain of calls.

## Internal Mechanics

A workflow is a stateful sequence or graph of steps.

### Workflow Graph

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

The graph makes paths visible.

It should include:

- happy path;
- missing information path;
- approval denied path;
- timeout path;
- tool failure path;
- cancellation path;
- unsafe request path.

### Step Definition

Each step needs a contract:

```text
Step:
Owner:
Input:
Output:
Success condition:
Failure condition:
Retry rule:
Timeout:
Next step:
Trace fields:
```

Owner can be:

- deterministic code;
- retrieval;
- model;
- tool;
- human;
- scheduler/worker.

Use model for ambiguous language work:

- classify request;
- summarize issue;
- draft response;
- propose action.

Use code for control:

- schema validation;
- permission checks;
- state transition;
- retry count;
- timeout;
- idempotency;
- approval enforcement.

### Durable State

Durable execution means workflow can resume after interruption.

Store:

```text
workflow_id
current_state
user_id
resource_id
step_history
pending_approval
tool_call_ids
retry_count
last_error
created_at
updated_at
expires_at
```

Durable state lets workflow survive:

- process restart;
- worker crash;
- network timeout;
- user delay;
- duplicate request;
- connector outage.

### Retries

Retries can fix temporary failures.

Retries can also duplicate side effects.

Safe retry:

```text
read_ticket
search_docs
draft_reply
```

Risky retry:

```text
send_email
issue_refund
create_order
grant_access
```

Write actions need idempotency keys and clear retry policy.

### Timeouts

Every waiting state needs timeout.

Examples:

```text
awaiting_approval -> expired after 24 hours
waiting_for_tool_result -> needs_review after 30 minutes
queued -> failed if not started after 2 hours
```

Without timeout, workflows become stuck ghosts in system state.

### Approvals

Approval is a workflow state.

It is not a message in chat.

Approval record should include:

```text
action
tool
arguments
risk
approver
approved_at
expires_at
evidence_shown
```

If tool arguments change, approval should reset.

### Compensation

Compensation is a follow-up action that repairs or offsets partial failure.

Example:

```text
task created
-> notification failed
-> compensation: mark task notification_pending and retry notification
```

Or:

```text
calendar event created
-> CRM note failed
-> compensation: add repair job or cancel calendar event if required
```

Compensation matters when multi-step workflow has side effects.

### Audit and Trace

Workflow trace should show:

- step name;
- owner;
- input summary;
- output summary;
- state transition;
- tool call;
- approval;
- failure;
- retry;
- timestamp.

Trace is how humans debug, audit, and trust the system.

## Real-World Example

Workflow:

```text
Create follow-up task for support ticket
```

User asks:

```text
Create a follow-up task for ticket T-104 because customer still cannot access API keys.
```

Workflow graph:

```text
START
-> validate_user
-> read_ticket
-> classify_request
-> draft_task
-> validate_task_args
-> approval_required?
   -> no: create_task
   -> yes: request_approval -> create_task
-> log_result
-> END
```

Step examples:

```text
Step: read_ticket
Owner: tool
Input: ticket_id
Output: ticket record
Failure: permission denied, not found, connector timeout
Retry: retry timeout twice
Timeout: 10 seconds
Next: classify_request
```

```text
Step: create_task
Owner: tool
Input: validated task args
Output: task_id
Failure: connector error, duplicate request
Retry: only with idempotency key
Timeout: 20 seconds
Next: log_result
```

End states:

```text
completed
failed
cancelled
expired
needs_human_review
permission_denied
```

This is safer than asking the model to "handle ticket."

## Common Mistakes and Failure Modes

Mistake 1: Model controls every step

Model should not own hidden loops, approvals, retries, or side-effect sequencing.

Mistake 2: No stop condition

Workflow can loop forever or keep trying unsafe paths.

Mistake 3: Retry repeats side effects

Write actions need idempotency. Some actions should not auto-retry.

Mistake 4: Human approval not represented as state

Approval must be durable, explicit, expiring state.

Mistake 5: Timeout missing

Waiting forever creates stuck workflows and confused users.

Mistake 6: Logs only final answer

Debugging requires step transitions, tool calls, approvals, and failures.

Mistake 7: Failure path not designed

Every workflow needs paths for missing info, permission denied, user denial, tool error, timeout, and cancellation.

Mistake 8: Workflow cannot resume

If worker restarts and state is lost, durable execution is missing.

## System Design Decisions

Define workflow trigger:

```text
user request
webhook
scheduled job
file upload
approval event
tool result
```

Define end states:

```text
completed
failed
cancelled
expired
needs_review
permission_denied
refused
```

Define steps:

```text
step name
owner
input
output
success
failure
retry
timeout
next
```

Define ownership:

```text
What does model own?
What does code own?
What does retrieval own?
What does tool own?
What does human own?
```

Define state store:

```text
Where is workflow state stored?
What fields are durable?
What transitions are allowed?
What expires?
```

Define side-effect rules:

```text
Which steps are read-only?
Which steps write?
Which need approval?
Which need idempotency?
Which need compensation?
```

Define trace:

```text
What does user see?
What does admin see?
What does audit log store?
```

Good orchestration makes workflow understandable before it becomes autonomous.

## Build Artifact

Create `workflow-orchestration-map.md`.

Use this template:

```text
Workflow Orchestration Map

Workflow:
Trigger:
User goal:
End states:

Workflow Graph:
START -> ...

Steps:
- name:
  owner:
  input:
  output:
  success condition:
  failure condition:
  retry rule:
  timeout:
  next:
  trace fields:

State Store:
- field:
  purpose:
  lifespan:

Approval Gates:
- action:
  approver:
  context shown:
  expiry:
  reset condition:

Side Effects:
- step:
  idempotency key:
  compensation:

Escalation Paths:
- condition:
  path:

Stop Conditions:
- condition:
  result:

Eval Cases:
- case:
  expected path:
```

Example:

```text
Workflow: Create follow-up task

Trigger: user asks assistant to create task
End states: completed, failed, cancelled, permission_denied

Approval Gate:
action: create high-priority task
approver: requesting support agent
context shown: ticket_id, title, priority, reason
expiry: 1 hour
reset condition: title/priority/ticket_id changes

Side Effect:
step: create_task
idempotency key: workflow_id + tool_name + ticket_id
compensation: if notification fails, keep task and create notification_retry job
```

Artifact complete when engineer can implement happy path and failure paths without asking model to invent process.

## Active Recall Questions

1. Why should production workflows be explicit?
2. What fields define a workflow step?
3. When should model own a step?
4. What should deterministic code own?
5. Why can retries be dangerous?
6. What is durable execution?
7. Why is approval a workflow state?
8. What is compensation in multi-step workflows?

## Summary

Workflow orchestration connects tools into reliable work. It defines steps, owners, state, retries, timeouts, approvals, failure paths, compensation, and traces.

Models can help with ambiguous language tasks, but system should own control flow. Durable state lets workflows survive delay, failure, restart, duplicate requests, and human approval.

The most important rule:

> Do not let hidden model reasoning replace explicit workflow state.

## Preview of Next Chapter

Chapter 32 made workflow explicit.

Chapter 33 separates memory, state, and human approval.

Next, you will learn what system should remember, what should expire, what requires consent, and why approval must be stored as explicit audit state instead of inferred from chat history.
