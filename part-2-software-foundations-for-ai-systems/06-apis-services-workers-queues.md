# Chapter 6: APIs, Services, Workers, and Queues

Chapter 5 gave you the map of an AI application:

```text
user interface
-> backend control point
-> data access
-> AI component
-> validation
-> storage and logs
-> response
```

This chapter asks a more operational question:

> How does work move through that architecture without timing out, duplicating itself, losing state, or confusing the user?

This matters because AI work is uneven. Some actions are fast, like classifying a short message. Some actions are slow, like parsing 200 PDFs, generating embeddings, calling external tools, or running evaluation batches. A serious AI system must know which work belongs in the user request and which work belongs in the background.

## Concept Overview

APIs, services, workers, and queues are the movement layer of an AI system.

They decide:

- how users ask the system to do work;
- where business rules live;
- which tasks happen immediately;
- which tasks happen later;
- how retries happen;
- how progress is tracked;
- how failures become visible instead of silent.

In a small demo, one function can do everything:

```text
receive request -> call model -> return answer
```

In a real system, that breaks quickly. AI applications need to handle files, model latency, rate limits, tool failures, retries, permissions, and user expectations. The work has to be split into reliable pieces.

The core components are:

```text
API
Receives user intent and returns a clear response.

Service
Owns business logic and coordinates system decisions.

Worker
Runs slow, retryable, or background tasks.

Queue
Buffers work so workers can process it safely.

Database
Stores durable state, job status, inputs, outputs, and failure records.
```

The learner should leave this chapter with one practical instinct:

> Do not put all AI work inside the web request.

## Why It Exists

AI systems need this structure because model work often violates normal request assumptions.

A normal web request is expected to be:

- short;
- predictable;
- easy to retry;
- easy to explain to the user;
- safe if the user refreshes the page.

AI work is often:

- slow;
- probabilistic;
- dependent on external APIs;
- expensive;
- rate-limited;
- connected to files, tools, or indexes;
- likely to need retry or review.

Imagine a user uploads 200 PDFs and clicks:

> Make these searchable.

A weak system does this:

```text
POST /upload
-> parse all files
-> chunk all pages
-> create embeddings
-> update vector index
-> return when done
```

This design looks simple, but it creates real product failure:

- the request times out;
- the user does not know progress;
- refreshing may duplicate work;
- one bad PDF can fail the whole upload;
- embedding rate limits can break the flow;
- there is no durable job record;
- support cannot debug what happened.

A stronger system does this:

```text
POST /documents
-> validate request
-> create document records
-> create ingestion job
-> enqueue job
-> return job_id

worker
-> parse documents
-> chunk content
-> generate embeddings
-> update search index
-> update job status
```

The user gets a quick response. The system keeps working. Failures can be retried. Progress can be shown. The architecture becomes understandable.

## Mental Model

Think of an AI application like an operating room, not a single desk.

The API is the front desk. It receives the request, checks that it is valid, creates a record, and tells the user what will happen next.

The service is the coordinator. It decides what must happen now, what can happen later, which rules apply, and which data must be stored.

The queue is the waiting line for work. It prevents the system from trying to do everything at once.

The worker is the specialist. It performs the slow task, updates progress, handles retries, and records the result.

The database is the memory of truth. It stores the job, current status, owner, inputs, outputs, errors, and timestamps.

The mental model:

```text
User asks for work.
API records the request.
Service decides the flow.
Queue holds slow work.
Worker performs slow work.
Database stores truth.
User sees status or result.
```

This separation makes the system easier to reason about:

- the API should not be responsible for long-running work;
- the worker should not invent business rules secretly;
- the queue should not be treated as the source of truth;
- the database should make progress and failure visible.

## Internal Mechanics

Most AI systems use the same basic flow for slow work.

Step 1: Receive intent

```text
POST /documents
```

The user is not really asking the server to "upload bytes only." They are asking the system to create a useful document resource that may later support search, summarization, extraction, or question answering.

The API should capture:

- user identity;
- organization or workspace;
- files or input references;
- requested operation;
- request id or idempotency key;
- basic validation result.

Step 2: Validate before accepting work

Validation should happen before expensive AI work starts.

Examples:

- Is the user allowed to upload here?
- Is the file type supported?
- Is the file size within limit?
- Is the operation enabled for this workspace?
- Is required metadata present?
- Is this a duplicate request?

Step 3: Create durable records

Before enqueueing work, the system should create durable state:

```text
document_id
job_id
owner_id
status
created_at
input_location
requested_operation
```

This matters because queues can retry, workers can crash, and users can refresh. If the system has no durable record, it cannot tell the truth about what happened.

Step 4: Enqueue background work

The API puts a job on a queue:

```text
job_type: document_ingestion
job_id: job_123
document_id: doc_456
```

The queue does not perform the work. It holds the work until a worker is ready.

Queues help with:

- buffering spikes;
- retrying failed work;
- controlling concurrency;
- isolating slow tasks from user requests;
- preventing one expensive task from blocking the whole app.

Step 5: Process in a worker

The worker takes the job and performs the slow steps:

```text
load file
-> extract text
-> split into chunks
-> create embeddings
-> write chunks
-> update index
-> mark complete
```

The worker must update status as it moves:

```text
pending -> processing -> indexed
pending -> processing -> failed
pending -> processing -> partial_success
```

Step 6: Retry safely

Retry is useful only if the job is idempotent.

Idempotent means:

> Running the same operation more than once should not create duplicate or corrupted results.

For document ingestion, an idempotency key might be:

```text
workspace_id + file_hash + ingestion_version
```

Without idempotency, retry can create duplicate chunks, duplicate embeddings, duplicate emails, duplicate tool calls, or duplicate billing events.

Step 7: Return status to the user

The original request should return quickly:

```json
{
  "job_id": "job_123",
  "status": "pending"
}
```

Then the user can check:

```text
GET /jobs/job_123
```

Or the system can push progress through events, websockets, or notifications.

The important principle:

> The user should never have to guess whether the system is working.

## Real-World Example

Consider a company building an internal knowledge assistant.

The product goal:

> Employees can upload policy documents and ask questions over them.

The system must ingest documents before retrieval can work.

Bad implementation:

```text
POST /upload
-> upload file
-> parse PDF
-> embed chunks
-> update vector database
-> answer first question
```

This feels direct, but it couples too many responsibilities:

- upload;
- parsing;
- indexing;
- retrieval;
- answer generation;
- user response.

If PDF parsing fails, the user sees a generic error. If embedding is slow, the request times out. If the same file is uploaded twice, the index may contain duplicates. If retrieval answers from a half-indexed file, trust drops.

Better implementation:

```text
POST /documents
-> create document record
-> store file
-> enqueue ingestion job
-> return document_id and job_id

GET /documents/doc_123
-> return document metadata and ingestion status

POST /questions
-> only retrieve from documents with status = indexed
```

Worker flow:

```text
job: ingest_document
-> mark job processing
-> extract text
-> detect tables and sections
-> chunk text
-> embed chunks
-> write chunks with document_id and permissions
-> mark document indexed
-> mark job complete
```

Now the system has clean behavior:

- users can see ingestion status;
- retrieval avoids unfinished documents;
- failed files can show specific errors;
- retries can continue safely;
- support can inspect job history;
- later chapters can add permissions, retrieval, evals, and observability.

This is how software architecture becomes the foundation for AI quality.

## Common Mistakes and Failure Modes

Mistake 1: Doing slow AI work inside the API request

This creates timeouts, poor user feedback, and fragile retries.

Mistake 2: Returning success before durable state exists

If the API says "uploaded" but no document or job record exists, the system has created a lie. The user believes work exists, but the backend may have nothing to resume.

Mistake 3: Treating the queue as the database

A queue is not the source of truth. It is a transport mechanism for work. The database should store job state, ownership, inputs, outputs, and failures.

Mistake 4: Retrying non-idempotent jobs

Retries can be dangerous. A retry may send the same email twice, charge twice, create duplicate chunks, call the same tool twice, or overwrite newer output.

Mistake 5: Hiding worker failures

If failures only appear in logs, users and product teams cannot understand the system. Failure should become a visible state with useful reason codes.

Mistake 6: Giving workers unclear ownership

Workers should not contain hidden product decisions that the API and service layer do not know about. Business rules should be explicit and testable.

Mistake 7: No progress model

For long-running work, users need truthful progress. Even simple states are better than silence:

```text
pending
processing
needs_review
complete
failed
```

## System Design Decisions

When designing an AI feature, decide where each piece of work belongs.

Use a synchronous API path when:

- the work finishes quickly;
- the user needs an immediate answer;
- failure can be shown immediately;
- the operation is cheap enough to repeat;
- no long file, tool, or indexing process is involved.

Use a worker and queue when:

- processing files;
- generating embeddings;
- running batch extraction;
- calling slow external APIs;
- invoking tools with retry;
- running evals;
- producing reports;
- sending notifications;
- updating indexes;
- handling work that may exceed request time limits.

Decide the job state model:

```text
pending
processing
blocked
needs_review
complete
failed
partial_success
cancelled
```

Decide the retry policy:

```text
What errors are retryable?
How many times should retry happen?
How long between retries?
When does work move to failed?
Can a human retry it manually?
```

Decide the idempotency rule:

```text
What makes this job unique?
What happens if the same request arrives twice?
What outputs should be reused?
What side effects must never repeat?
```

Decide the user feedback path:

```text
polling
webhook
websocket
email
notification center
dashboard status
```

The goal is not to make the system complicated. The goal is to make work visible, resumable, and safe.

## Build Artifact

Create `service-flow.md` for one AI feature.

Choose a feature from your book project, such as:

- document upload and indexing;
- support ticket classification;
- report generation;
- extraction from invoices;
- agent tool execution;
- evaluation run.

Use this template:

```text
Service Flow

Feature:
User goal:

API

Endpoint:
Input:
Output:
Validation:
Immediate response:

Service Logic

Business rules:
Permission checks needed:
Data records created:
Model/tool calls needed:
Synchronous steps:
Asynchronous steps:

Job

Job name:
Job id:
Job owner:
Job input:
Job output:
Status states:

Queue

Queue name:
Why queue is needed:
Concurrency rule:
Retry policy:
Dead-letter or failed state:

Worker

Worker steps:
Progress updates:
Idempotency key:
Side effects:
Failure handling:

User Feedback

How user sees progress:
How user sees failure:
How user retries or fixes the issue:

Test Cases

Normal success:
Invalid input:
Duplicate request:
Worker crash:
External API timeout:
Partial failure:
Permission failure:
```

Example:

```text
Feature: Document ingestion for knowledge assistant
User goal: Upload PDFs and make them searchable

Endpoint: POST /documents
Immediate response: document_id, job_id, status=pending

Synchronous steps:
- authenticate user
- validate file type and size
- store file
- create document record
- create ingestion job
- enqueue job

Asynchronous steps:
- parse PDF
- chunk content
- generate embeddings
- write chunks
- update search index
- mark document indexed

Status states:
pending -> processing -> indexed
pending -> processing -> failed
pending -> processing -> partial_success

Idempotency key:
workspace_id + file_hash + ingestion_version
```

This artifact should be specific enough that an engineer could implement the first version without guessing the flow.

## Active Recall Questions

1. Why should long document ingestion not run inside a normal API request?
2. What is the difference between an API, a service, a worker, and a queue?
3. Why should the database store job state instead of relying only on queue messages?
4. What does idempotency protect against?
5. Which AI tasks in your system should be synchronous, and which should be asynchronous?
6. What should the user see when a background job fails?
7. Why is a retry policy dangerous without side-effect control?
8. How does this chapter prepare the system for retrieval, tools, evals, and observability later?

## Summary

AI engineering is not only about prompts and models. It is also about moving work through software in a way that users can trust.

APIs receive intent. Services coordinate rules. Workers perform slow work. Queues buffer and retry tasks. Databases store the truth about state. Together, they turn fragile AI demos into systems that can survive real users, real files, real latency, and real failure.

The most important lesson:

> Slow, expensive, retryable, or stateful AI work should become a visible job, not a hidden web request.

## Preview of Next Chapter

Chapter 6 showed how work moves through the system.

Chapter 7 asks what that work is allowed to access.

Next, you will design auth, permissions, files, and storage. This is where the system learns that a model should not retrieve, read, remember, or act on information just because the information exists. It must be allowed for the current user, workspace, and task.
