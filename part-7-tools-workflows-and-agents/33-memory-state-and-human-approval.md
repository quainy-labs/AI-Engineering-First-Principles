# Chapter 33: Memory, State, and Human Approval

Chapter 32 made workflow explicit. This chapter separates what system remembers, what it can reuse, and what requires human decision.

Memory and state can make systems useful. They can also make systems creepy, unsafe, and impossible to audit.

## Real Problem

An assistant says:

```text
I remember you prefer refunds to store credit.
```

User never approved storing that preference.

Another assistant says:

```text
I already created replacement order.
```

But approval was never logged.

## Bad Default Solution

Bad solution:

- store everything;
- call it memory;
- include prior conversations automatically;
- let model decide what matters;
- skip consent;
- treat chat history as audit log.

This creates privacy risk and unreliable behavior.

## First Principles

State is information needed to continue current workflow.

Memory is information retained beyond current task.

Audit log is immutable record of what happened.

These are different.

| Type | Purpose | Example | Lifespan |
|---|---|---|---|
| Session state | current task | approval pending | minutes/hours |
| Workflow state | process progress | ticket triaged | task lifecycle |
| User memory | preference | timezone | until changed/deleted |
| Audit log | accountability | refund approved by X | long-term |

Do not mix them.

## System Decision

Decide what to remember:

```text
What is needed for current task?
What should persist after task?
Who can see it?
Can user edit/delete it?
Does it need consent?
Is it evidence or preference?
Is it audit record?
When does it expire?
```

Approval design:

```text
Action:
Risk:
Approver:
Information shown:
Approve/deny options:
Expiration:
Audit log:
```

Human approval must be explicit state, not implied conversation.

## Minimal Build

Create `state-and-approval-policy.md`:

```text
Workflow:

Session state:
- item:
  reason:
  lifespan:

Long-term memory:
- item:
  consent:
  edit/delete:
  lifespan:

Audit log:
- event:
  fields:
  retention:

Approval gates:
- action:
  risk:
  approver:
  required context:
  expiry:
  log fields:
```

## Failure Modes

State/memory fails when:

- system remembers sensitive info without consent;
- old memory overrides current user intent;
- approval not logged;
- model infers approval from vague message;
- audit log is editable;
- state expires too late or too soon;
- user cannot correct memory;
- workflow resumes with stale state.

## Evaluation

Test:

- approval required;
- approval denied;
- approval expired;
- memory correction;
- memory deletion;
- stale state;
- sensitive data request;
- workflow resume.

Metrics:

- approval accuracy;
- unauthorized action rate;
- memory correction success;
- stale-state error rate;
- audit completeness.

## Improvement

Improve by:

- separating state stores;
- requiring explicit approval records;
- adding memory consent;
- adding expiration;
- exposing memory to user;
- logging every state transition;
- preventing model from fabricating approval.

## Proof Artifact

Create `state-and-approval-policy.md`.

Include:

- state categories;
- memory rules;
- consent rules;
- approval gates;
- audit log fields;
- expiry rules;
- test cases.

## Decision Checklist

- [ ] State and memory separated.
- [ ] Audit log separated.
- [ ] Long-term memory requires consent.
- [ ] Approval gates explicit.
- [ ] Approval logged before action.
- [ ] Expiry rules exist.
- [ ] User can correct/delete memory.

## Active Recall

1. Difference between state, memory, audit log?
2. Why is chat history not audit log?
3. When does memory need consent?
4. Why must approval be explicit state?
5. What can go wrong with stale state?

## Next

Next chapter covers computer use and browser automation, where state, approval, and traces become even more important.
