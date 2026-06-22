# Chapter 33: Memory, State, and Human Approval

Chapter 32 made workflow explicit:

```text
steps
-> state
-> retries
-> timeouts
-> approvals
-> traces
```

This chapter separates what the system remembers, what it can reuse, and what requires human decision.

Memory and state can make AI systems useful. They can also make systems creepy, unsafe, and impossible to audit.

## Concept Overview

State is information needed to continue current work.

Memory is information retained beyond current task.

Audit log is durable record of what happened.

Human approval is explicit permission for a specific action.

These are different things.

| Type | Purpose | Example | Lifespan |
|---|---|---|---|
| Session state | Current interaction | selected ticket | minutes/hours |
| Workflow state | Process progress | approval pending | task lifecycle |
| User memory | Reusable preference | timezone | until changed/deleted |
| Audit log | Accountability | refund approved by user X | long-term |
| Approval record | Action permission | approve create_task args | until used/expired |

The first principle:

> Do not use chat history as state, memory, approval, or audit log.

Chat history can provide context. It should not be the system of record for actions, approvals, or long-term memory.

## Why It Exists

An assistant says:

```text
I remember you prefer refunds to store credit.
```

User never approved storing that preference.

Another assistant says:

```text
I already created the replacement order.
```

But approval was never logged.

Bad default:

- store everything;
- call it memory;
- include prior conversations automatically;
- let model decide what matters;
- skip consent;
- infer approval from vague chat;
- treat chat history as audit log.

This creates privacy risk, stale behavior, unsafe action, and weak accountability.

This chapter exists because useful AI systems need explicit memory and approval policies, not vague "the assistant remembers."

## Mental Model

Think of system memory like labeled storage boxes.

Each box has a purpose, owner, retention period, and access rule.

Mental model:

```text
current task state
-> workflow state
-> optional long-term memory
-> audit log
-> approval record
```

Before storing anything, ask:

```text
Is this needed for current task?
Should it persist after task?
Did user consent?
Can user view/edit/delete it?
Is it evidence, preference, or action record?
When does it expire?
Who can access it?
```

Memory should help user, not surprise user.

Approval should permit one clear action, not unlimited future behavior.

## Internal Mechanics

Separate storage categories.

### Session State

Session state supports current interaction.

Examples:

- current ticket ID;
- selected document;
- draft response;
- temporary extracted fields;
- current user intent;
- pending clarification.

Session state should expire quickly.

It should not become long-term memory unless user agrees.

### Workflow State

Workflow state tracks process progress.

Examples:

- `pending_approval`;
- `tool_executed`;
- `waiting_for_webhook`;
- `failed_validation`;
- `needs_human_review`;
- `cancelled`.

Workflow state should be durable until workflow ends.

It lets system resume after:

- refresh;
- worker restart;
- user delay;
- tool failure;
- approval wait.

### Long-Term Memory

Long-term memory stores information beyond current task.

Examples:

- user timezone;
- preferred writing tone;
- default project;
- accessibility preference;
- recurring workflow preference.

Long-term memory needs controls:

```text
consent
view
edit
delete
expiration
source
last updated
```

Avoid storing sensitive data unless clearly needed and approved.

### Audit Log

Audit log records what happened.

Examples:

- user approved tool call;
- system executed tool;
- connector returned result;
- permission check passed/failed;
- human corrected extraction;
- workflow state changed.

Audit log should be append-only or tamper-resistant where risk demands it.

Audit log fields:

```text
event_id
workflow_id
user_id
action
resource_id
tool_name
arguments_summary
approval_id
timestamp
result
trace_id
```

Audit is for accountability. It is not normal memory.

### Human Approval

Approval is explicit state.

It must not be inferred from:

```text
sounds good
go ahead maybe
looks fine
```

unless product defines exact approval semantics.

Approval record should include:

```text
approval_id
approver
action
tool
arguments
risk
evidence_shown
approved_at
expires_at
used_at
status
```

Approval should expire.

If action arguments change, approval should reset.

### Memory Retrieval

Do not automatically inject all memory into every prompt.

Memory retrieval should be scoped:

```text
Which memory is relevant?
Is it allowed for this task?
Is it still current?
Could it conflict with current user instruction?
```

Current user intent should usually beat old memory.

Example:

```text
Old memory: user prefers concise replies
Current instruction: write detailed explanation
Winner: current instruction
```

### Deletion and Correction

User should be able to correct or delete long-term memory.

Correction path:

```text
show memory
-> user edits/deletes
-> store update event
-> prevent deleted memory from future context
```

Deletion must propagate to indexes or caches if memory was embedded or copied.

## Real-World Example

Use case:

```text
Support operations assistant
```

Workflow:

```text
Create follow-up task for ticket T-104.
```

State:

```text
workflow_id: wf_123
ticket_id: T-104
current_state: awaiting_approval
draft_task_title: Follow up on API key access
expires_at: 2026-06-22T18:00:00Z
```

Approval:

```text
approval_id: appr_456
approver: support_agent_9
action: create_internal_task
arguments:
  ticket_id: T-104
  title: Follow up on API key access
risk: reversible_write
status: approved
```

Audit log:

```text
event: approval_granted
workflow_id: wf_123
approval_id: appr_456
timestamp: ...
```

Long-term memory:

```text
none needed
```

Bad design would store:

```text
User likes follow-up tasks for API key issues.
```

without consent. That is not needed. Keep workflow state, not surprise memory.

## Common Mistakes and Failure Modes

Mistake 1: Storing everything as memory

Most task details should expire as state, not persist as memory.

Mistake 2: Chat history treated as audit log

Audit needs structured, durable, queryable event records.

Mistake 3: Approval inferred from vague language

Approval should be explicit and tied to concrete action arguments.

Mistake 4: Old memory overrides current intent

Current user instruction should usually beat old preference memory.

Mistake 5: User cannot inspect memory

Hidden memory feels creepy and causes wrong personalization.

Mistake 6: Memory has no deletion path

Memory without delete/edit creates trust and privacy problems.

Mistake 7: Audit log editable like normal data

Audit needs integrity. Corrections should append events, not rewrite history silently.

Mistake 8: Approval does not expire

Old approval should not authorize later changed actions.

## System Design Decisions

Define state categories:

```text
session state
workflow state
long-term memory
audit log
approval record
```

Define retention:

```text
What expires after session?
What lives until workflow ends?
What persists long-term?
What must be retained for audit?
```

Define consent:

```text
What memory needs explicit user consent?
What is never stored?
Can user view/edit/delete memory?
How is consent recorded?
```

Define approval gates:

```text
Which actions require approval?
Who can approve?
What context must be shown?
When does approval expire?
What resets approval?
Where is approval logged?
```

Define audit:

```text
Which events are audited?
What fields are logged?
Who can view audit?
How long is audit retained?
How are corrections represented?
```

Define memory injection:

```text
When is memory retrieved?
What memory is relevant?
What memory is forbidden?
What wins when memory conflicts with current instruction?
```

The goal is usefulness without hidden persistence.

## Build Artifact

Create `state-and-approval-policy.md`.

Use this template:

```text
State and Approval Policy

Workflow:
User goal:

Session State:
- item:
  reason:
  lifespan:
  storage:

Workflow State:
- item:
  reason:
  allowed transitions:
  expiry:

Long-Term Memory:
- item:
  consent required:
  user can edit/delete:
  lifespan:
  retrieval rule:

Audit Log:
- event:
  fields:
  retention:
  viewer roles:

Approval Gates:
- action:
  risk:
  approver:
  required context:
  expiry:
  reset condition:
  log fields:

Deletion and Correction:
- item:
  correction path:
  deletion path:

Test Cases:
- case:
  expected behavior:
```

Example:

```text
Approval Gate:
action: create high-priority support task
risk: reversible_write
approver: requesting support agent
required context: ticket_id, title, priority, reason
expiry: 1 hour
reset condition: ticket_id/title/priority changes
log fields: approver, arguments, timestamp, approval_id

Long-Term Memory:
item: preferred reply tone
consent required: yes
user can edit/delete: yes
retrieval rule: only for drafting messages, never for policy or approval decisions
```

Artifact complete when engineer can tell what is state, what is memory, what is audit, and what requires approval.

## Active Recall Questions

1. What is difference between state, memory, audit log, and approval record?
2. Why is chat history not an audit log?
3. When does long-term memory require consent?
4. Why must approval be explicit state?
5. What should approval record contain?
6. Why should old memory not override current user intent?
7. Why should approval expire?
8. What can go wrong when workflow resumes with stale state?

## Summary

State keeps current work moving. Memory persists useful preferences beyond a task. Audit logs record what happened. Human approval authorizes specific actions. These must be separate.

Strong systems make memory visible, editable, deletable, consented, and scoped. Approval is explicit, expiring, logged state tied to concrete action arguments.

The most important rule:

> Remember less by default, and record approvals explicitly.

## Preview of Next Chapter

Chapter 33 separated memory, state, audit, and approval.

Chapter 34 covers computer use and browser automation.

Next, you will learn why UI-driving agents need even stronger state, approval, traces, step limits, and recovery paths. Computer use is powerful because it can operate systems without APIs, but risky because UI state is fragile and side effects are easy.
