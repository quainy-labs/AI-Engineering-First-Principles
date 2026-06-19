# Chapter 8: Async Jobs, Background Processing, and Webhooks

Chapter 7 established storage and permissions. This chapter finishes software foundations by handling time.

AI work often happens after user request:

- documents finish indexing;
- eval run completes;
- external tool returns result;
- webhook receives event;
- background job retries;
- user approval expires.

If system does not handle time, it cannot handle real workflows.

## Real Problem

Operations assistant sends request to external ticketing API.

API accepts request but actual task creation happens later. A webhook arrives with final status.

If system assumes immediate success, user sees wrong state.

## Bad Default Solution

Bad design:

- assume all actions complete instantly;
- no event handler;
- no retry;
- no timeout;
- no state transition;
- no dead-letter queue;
- no duplicate webhook protection.

This creates phantom success.

## First Principles

Async system separates:

- request accepted;
- work started;
- work completed;
- work failed;
- user notified.

Webhook is external event sent to your system.

Background job is internal work run outside main request.

State machine defines allowed transitions.

Example:

```text
pending -> processing -> completed
pending -> processing -> failed
pending -> expired
```

## System Decision

Use async/events when:

- external tool takes time;
- file processing takes time;
- user approval may wait;
- eval run is batch;
- webhook notifies status;
- task needs retry.

Define:

```text
Event:
Source:
Payload:
Validation:
Idempotency key:
State transition:
Retry rule:
Timeout:
User notification:
```

## Minimal Build

Create `async-event-design.md`.

Template:

```text
Workflow:

Jobs:
- name:
  trigger:
  input:
  output:
  states:
  retry:
  timeout:

Webhooks:
- event:
  source:
  validation:
  idempotency:
  state transition:

Notifications:
- user event:
  channel:
  message:
```

## Failure Modes

Async systems fail when:

- duplicate webhook creates duplicate action;
- job retries unsafe side effect;
- timeout not handled;
- user sees stale status;
- event accepted without validation;
- job failure hidden;
- approval expires but action still runs;
- state transition impossible to audit.

## Evaluation

Test:

- normal job completion;
- job failure;
- retry success;
- duplicate webhook;
- invalid webhook;
- timeout;
- approval expiry;
- user checks status mid-job.

Pass if state remains correct and no duplicate side effects occur.

## Improvement

Improve by:

- adding idempotency keys;
- validating webhook signatures;
- recording event IDs;
- making state transitions explicit;
- adding timeout states;
- adding dead-letter handling;
- notifying users on failure.

## Proof Artifact

Create `async-event-design.md`.

Include:

- background jobs;
- webhook events;
- state transitions;
- retry rules;
- timeout rules;
- idempotency rules;
- failure handling.

## Decision Checklist

- [ ] Long-running work uses jobs.
- [ ] Webhooks are validated.
- [ ] Duplicate events safe.
- [ ] State transitions explicit.
- [ ] Timeouts defined.
- [ ] Retry rules safe.
- [ ] User can see current status.

## Active Recall

1. Why does AI system need async jobs?
2. What is webhook?
3. Why use idempotency key?
4. What is phantom success?
5. Why model/tool workflows need state transitions?

## Next

Part 2 established software foundations. Next, Part 3 should move into data and knowledge foundations: data, documents, ingestion, schemas, quality, search, and information architecture.

