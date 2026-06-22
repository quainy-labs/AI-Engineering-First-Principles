# Chapter 8: Async Jobs, Background Processing, and Webhooks

Chapter 6 showed how work moves through APIs, services, workers, and queues.

Chapter 7 showed how that work must respect identity, permissions, files, storage, logs, and ownership.

This chapter finishes the software foundation by handling time.

AI systems do not always finish work during the user request. Documents index later. Evaluation runs finish later. External systems respond later. Human approvals may wait for hours. Webhooks may arrive after your original API call has already returned.

If the system cannot represent time clearly, it cannot represent truth clearly.

## Concept Overview

Async jobs, background processing, and webhooks help an AI system handle work that unfolds over time.

They answer:

- what work was requested;
- what work has started;
- what work is waiting;
- what work completed;
- what work failed;
- what external event arrived;
- what the user should see now;
- what the system should do next.

The core concepts:

```text
Async job
Internal work that runs outside the main request.

Background worker
Process that performs async jobs.

Webhook
External event sent to your system by another service.

State machine
Allowed statuses and transitions for a workflow.

Retry policy
Rules for repeating failed work safely.

Timeout
Rule for when waiting becomes failure, expiry, or manual review.

Idempotency
Protection against duplicate requests, retries, and repeated events.
```

This chapter extends the previous chapters:

```text
Chapter 6: work moves through the system.
Chapter 7: work respects access and storage rules.
Chapter 8: work changes state over time.
```

## Why It Exists

Real AI workflows are rarely instant.

Examples:

- a user uploads files and indexing finishes later;
- an extraction job processes 5,000 invoices overnight;
- a model evaluation run completes after many test cases;
- an agent asks for human approval before taking action;
- an external ticketing system sends final status through a webhook;
- an audio transcription job returns when processing completes;
- a tool call times out and needs retry;
- a user cancels a job before it finishes.

If the system assumes everything completes immediately, it creates phantom success.

Phantom success means:

> The system tells the user that something succeeded before it actually knows the final result.

Example:

A customer operations assistant creates a refund ticket in an external support system.

Bad flow:

```text
user clicks create refund ticket
-> AI calls ticketing API
-> ticketing API accepts request
-> app shows "Ticket created"
```

But "accepted" does not always mean "completed." The external system may process the request later, reject it, or send final status through a webhook.

Better flow:

```text
user clicks create refund ticket
-> app creates local action record
-> app calls ticketing API
-> app marks action as pending_external_confirmation
-> webhook arrives
-> app validates webhook
-> app updates action to completed or failed
-> user sees final status
```

The user sees truth instead of confidence theater.

## Mental Model

Think of async AI work like a package delivery system.

Placing an order is not the same as receiving the package. A tracking number is not the same as delivery. A delay is not the same as failure. A failed delivery may be retried. A duplicate notification should not create two packages.

AI systems need the same clarity.

The mental model:

```text
request
-> durable record
-> job or external call
-> waiting state
-> event or worker result
-> validated transition
-> user-visible status
```

A strong async system separates these moments:

```text
accepted
started
waiting
completed
failed
expired
cancelled
needs_review
```

The model should not invent the status. The UI should not guess the status. The database should store the status, and the system should update it through clear transitions.

This is especially important for AI because model output may trigger follow-up work:

```text
model suggests action
-> user approves
-> worker calls tool
-> external service responds later
-> webhook updates status
-> system summarizes result
```

Without explicit state, nobody knows where the workflow really is.

## Internal Mechanics

Async design starts with the workflow state.

### State Machine

A state machine defines allowed statuses and transitions.

Example for document ingestion:

```text
uploaded -> queued -> parsing -> embedding -> indexed
uploaded -> queued -> parsing -> failed
uploaded -> queued -> parsing -> partial_success
uploaded -> cancelled
```

Example for human-approved tool action:

```text
drafted -> awaiting_approval -> approved -> executing -> completed
drafted -> awaiting_approval -> rejected
drafted -> awaiting_approval -> expired
approved -> executing -> failed
```

The state machine prevents impossible or dishonest transitions.

For example:

```text
completed -> processing
rejected -> executing
expired -> approved
```

These should usually be invalid.

### Async Jobs

An async job should have a durable record:

```text
job_id
job_type
owner_id
workspace_id
status
input_reference
attempt_count
max_attempts
created_at
started_at
finished_at
last_error
idempotency_key
```

The job record lets the system answer:

- who requested this work;
- what is being processed;
- whether it is safe to retry;
- what failed;
- what the user should see;
- whether the job is still allowed under current permissions.

### Background Processing

Background workers run jobs outside the user request.

Common AI background tasks:

- file parsing;
- document chunking;
- embedding generation;
- search index updates;
- batch extraction;
- report generation;
- eval runs;
- trace analysis;
- cache warming;
- notification sending;
- cleanup and deletion propagation.

Workers should update status as they move. They should not be silent.

```text
queued -> processing -> completed
queued -> processing -> failed
queued -> processing -> retry_scheduled
```

### Webhooks

A webhook is an external service calling your system to report an event.

Examples:

- transcription completed;
- payment failed;
- ticket created;
- file scan completed;
- integration sync finished;
- tool execution status changed.

Webhook handlers must be careful because external events can be delayed, duplicated, forged, or arrive out of order.

A webhook handler should:

```text
verify signature
parse event
check event id
check related resource
apply idempotency
validate allowed state transition
store event record
update state
trigger follow-up work if needed
```

### Idempotency

Idempotency protects the system when the same request or event arrives more than once.

Duplicate events are normal. Retries are normal. Network uncertainty is normal.

For a webhook, store the external event ID:

```text
provider: ticketing_system
event_id: evt_123
received_at: timestamp
processed: true
```

If the same event arrives again, the system should recognize it and avoid repeating side effects.

### Retry and Timeout

Not every failure should be retried.

Retryable:

- temporary network failure;
- rate limit;
- provider timeout;
- worker crash;
- temporary file processing error.

Not usually retryable:

- invalid permission;
- unsupported file type;
- malformed input;
- user rejected approval;
- resource deleted;
- policy violation.

Timeouts matter because waiting forever is also a bug.

Example:

```text
awaiting_approval -> expired after 24 hours
processing_external_action -> needs_review after 30 minutes
queued -> failed if not started after 2 hours
```

### User Feedback

Async systems should make status visible.

The user should know:

- what was accepted;
- what is still processing;
- what needs their approval;
- what failed;
- what can be retried;
- what completed;
- what was cancelled or expired.

The goal is not to expose backend complexity. The goal is to avoid silence and false certainty.

## Real-World Example

Imagine an operations action assistant.

The user asks:

> Create a high-priority ticket for this customer escalation and assign it to the enterprise support queue.

The AI drafts an action:

```text
tool: create_ticket
customer_id: cus_123
priority: high
queue: enterprise_support
summary: customer escalation
```

Because this action affects an external system, the product requires human approval.

Good workflow:

```text
drafted
-> awaiting_approval
-> approved
-> executing
-> pending_external_confirmation
-> completed
```

Failure paths:

```text
awaiting_approval -> rejected
awaiting_approval -> expired
executing -> failed
pending_external_confirmation -> failed
pending_external_confirmation -> needs_review
```

The system records:

```text
action_id
user_id
workspace_id
tool_name
tool_arguments
approval_status
external_request_id
webhook_event_id
current_status
last_error
audit_log
```

The external ticketing system may respond immediately with:

```text
accepted: true
request_id: req_789
```

But the assistant should not say:

> Ticket created.

It should say the truthful state:

> Ticket request submitted. Waiting for confirmation.

When the webhook arrives:

```text
event: ticket.created
request_id: req_789
ticket_id: TICK-456
```

The system validates the event, checks idempotency, updates the action to completed, stores the ticket ID, and notifies the user.

This is how an AI agent stops being a fragile demo and starts behaving like reliable software.

## Common Mistakes and Failure Modes

Mistake 1: Treating accepted as completed

External APIs often accept work before final completion. The local system should represent the waiting state.

Mistake 2: No explicit state machine

Without allowed transitions, workflows drift into impossible states. This makes debugging, UI, audit, and retries difficult.

Mistake 3: Duplicate webhook creates duplicate side effects

Webhook providers may send the same event more than once. The handler must recognize already-processed events.

Mistake 4: Retrying unsafe actions

Retrying a failed model call may be fine. Retrying a payment, email, ticket creation, or data update may create duplicate consequences unless idempotency is designed.

Mistake 5: No timeout state

If the system waits forever, users cannot tell whether work is delayed, failed, or forgotten.

Mistake 6: Webhook accepted without validation

Unsigned or unverified webhooks can let outside systems change internal state.

Mistake 7: Stale permissions

A job may be queued while the user has permission, then run after permission changes. Long-running work should know its authorization model.

Mistake 8: Hidden failures

If failures live only in worker logs, users cannot recover and product teams cannot improve the workflow.

## System Design Decisions

For every async workflow, decide the state machine first.

Define states:

```text
What states can this workflow be in?
Which states are final?
Which states require user attention?
Which states can be retried?
```

Define transitions:

```text
What event moves work from one state to another?
Which transitions are forbidden?
Who or what is allowed to trigger each transition?
```

Define ownership:

```text
Which user, workspace, or service owns this job?
What permissions are checked at request time?
What permissions are checked at execution time?
```

Define idempotency:

```text
What key prevents duplicate work?
What external event ID is stored?
What side effects must not repeat?
```

Define retry policy:

```text
Which errors retry?
How many attempts?
What delay?
When does retry stop?
What state appears after final failure?
```

Define timeout policy:

```text
How long can this state last?
What happens after timeout?
Can a human revive or retry it?
```

Define webhook handling:

```text
How is the webhook authenticated?
How are duplicate events handled?
Can events arrive out of order?
What state transition does each event cause?
```

Define user feedback:

```text
What does the user see while waiting?
What does the user see on failure?
What can the user retry, cancel, or approve?
```

Async design is mostly about honesty. The system should say what it knows, what it is waiting for, and what changed.

## Build Artifact

Create `async-event-design.md` for one AI workflow.

Choose a workflow such as:

- document ingestion;
- external ticket creation;
- invoice extraction;
- evaluation run;
- report generation;
- human-approved tool action;
- audio transcription;
- integration sync.

Use this template:

```text
Async Event Design

Workflow:
User goal:

State Machine:
- state:
  meaning:
  final: yes/no
  user-visible message:

Allowed Transitions:
- from:
  to:
  trigger:
  allowed actor:
  validation:

Jobs:
- job name:
  trigger:
  owner:
  input:
  output:
  retry policy:
  timeout:
  idempotency key:

Webhooks:
- provider:
  event:
  signature validation:
  external event id:
  related resource:
  state transition:
  duplicate behavior:

Failures:
- failure:
  retryable: yes/no
  resulting state:
  user message:
  support/debug signal:

User Feedback:
- state:
  message:
  available action:

Tests:
- normal completion:
- duplicate webhook:
- invalid webhook:
- retry success:
- final failure:
- timeout:
- permission change:
- user cancellation:
```

Example:

```text
Workflow: Human-approved ticket creation
User goal: Create a support ticket from an AI-drafted action

State Machine:
- drafted: AI created proposed action
- awaiting_approval: user must approve
- approved: user approved action
- executing: worker is calling ticket API
- pending_external_confirmation: ticket API accepted request
- completed: webhook confirmed ticket creation
- failed: action failed
- expired: approval not given in time

Webhook:
provider: ticketing system
event: ticket.created
signature validation: required
external event id: event_id
related resource: external_request_id
state transition: pending_external_confirmation -> completed
duplicate behavior: ignore if event_id already processed

Timeout:
awaiting_approval expires after 24 hours
pending_external_confirmation moves to needs_review after 30 minutes
```

The artifact is complete when another engineer can implement the workflow without guessing what each status means or when each transition happens.

## Active Recall Questions

1. What is phantom success, and why is it dangerous in AI systems?
2. How is an async job different from a webhook?
3. Why should workflows have explicit state machines?
4. What makes retry safe or unsafe?
5. Why do webhook handlers need idempotency?
6. What should happen when a user approval expires?
7. Why should long-running jobs understand ownership and permissions?
8. What should the user see while a background task is still running?

## Summary

AI systems do not only answer questions. They wait, retry, receive events, process files, call tools, request approvals, and update state over time.

Async jobs handle internal background work. Webhooks handle external events. State machines make workflow truth explicit. Idempotency protects against duplicate work. Retry and timeout policies prevent silence, loops, and phantom success.

The most important rule:

> Never pretend async work is complete just because it was accepted. Store the state and tell the user the truth.

## Preview of Next Chapter

Part 2 established the software foundation:

```text
architecture
-> APIs and workers
-> auth and storage
-> async state over time
```

Part 3 moves into data and knowledge foundations.

Next, Chapter 9 asks what kind of information the AI system is using: structured data, documents, and state. This distinction matters because a model should not be used as a database, vector search should not be treated as truth, and chat history should not become an audit log.
