# Chapter 6: APIs, Services, Workers, and Queues

Chapter 5 mapped AI application architecture. This chapter explains how work moves through that architecture.

AI systems often combine fast user actions with slow background work. Search query may be fast. Document ingestion, embedding, eval runs, and batch extraction may be slow.

## Real Problem

User uploads 200 PDFs and asks:

> Make these searchable.

Bad system tries to parse, chunk, embed, index, and answer in one web request.

Result:

- timeout;
- partial indexing;
- no progress status;
- duplicate jobs;
- no retry path;
- confused user.

Good system uses API for request, worker for slow job, queue for reliability, database for status.

## Bad Default Solution

Bad design:

```text
POST /upload
-> parse files
-> chunk
-> embed
-> index
-> return when done
```

Problems:

- request timeout;
- retry duplicates work;
- user cannot see progress;
- failures hard to resume;
- scaling impossible.

## First Principles

API receives intent.

Service owns business logic.

Worker runs slow or retryable tasks.

Queue buffers work.

Database records state.

Pattern:

```text
API request
-> validate
-> create job record
-> enqueue job
-> return job_id
-> worker processes job
-> update status
-> user polls/subscribes for result
```

## System Decision

Use synchronous API when:

- task finishes quickly;
- no heavy model call;
- no file processing;
- no long retrieval/index update.

Use worker/queue when:

- processing files;
- generating embeddings;
- running evals;
- batch extracting data;
- calling slow tools;
- retrying external APIs.

## Minimal Build

Create `service-flow.md`.

Template:

```text
Endpoint:
Input:
Validation:
Synchronous steps:
Async job needed? yes/no
Queue:
Worker:
Job status:
Retry rule:
Idempotency key:
Failure state:
User notification:
```

Example:

```text
Endpoint: POST /documents
Sync: validate files, create document records
Async: parse, chunk, embed, index
Job status: pending -> processing -> indexed -> failed
Retry: retry parsing 2 times, embedding 3 times
Idempotency: file hash
```

## Failure Modes

Service design fails when:

- slow tasks run inside request;
- queue jobs not idempotent;
- retry repeats side effects;
- job state not stored;
- no failure state;
- no progress visibility;
- workers share too much hidden state;
- API returns success before durable record exists.

## Evaluation

Test service flow:

- normal upload;
- duplicate upload;
- worker crash;
- embedding API timeout;
- invalid file;
- partial success;
- retry;
- user checks status.

Pass if no duplicate side effects and status remains truthful.

## Improvement

Improve by:

- adding job table;
- adding idempotency key;
- splitting large jobs;
- adding retry policy;
- adding dead-letter state;
- adding progress events;
- logging worker trace.

## Proof Artifact

Create `service-flow.md`.

Include:

- endpoints;
- sync/async split;
- worker jobs;
- queue usage;
- retry/idempotency rules;
- failure states.

## Decision Checklist

- [ ] Slow work moved to worker.
- [ ] Job state stored.
- [ ] Queue retry safe.
- [ ] Idempotency key exists.
- [ ] API returns durable job ID.
- [ ] Failure state visible.
- [ ] User can see progress.

## Active Recall

1. Why not process PDFs inside request?
2. What does queue do?
3. Why is idempotency important?
4. What should job state include?
5. Which AI tasks usually belong in workers?

## Next

Now that work can move through APIs and workers, next chapter defines durable storage, files, auth, and permissions.

