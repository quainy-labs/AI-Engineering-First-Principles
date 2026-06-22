# Project 2: AI App Skeleton

## Concept

AI features do not live inside prompts. They live inside software systems.

This project builds the reusable application skeleton that later AI projects can use: API or CLI boundary, storage, file handling, background jobs, permissions, service layer, and trace logs.

This is not about model quality. It is about where AI work belongs in real software.

The learner should finish with a runnable local app skeleton that can accept requests, store state, upload files, run a simulated background job, enforce simple roles, and record traces.

## What You Will Build

Build a minimal AI-ready application skeleton.

The system should include:

- API or CLI interface;
- service layer;
- local persistence;
- user and role model;
- file upload or file import path;
- background job simulation;
- model gateway placeholder;
- request trace log;
- simple test or verification command.

The product answers one question:

```text
Where should AI behavior live inside a real application?
```

## Chapters Practiced

This project proves understanding from:

- Chapter 5: AI application architecture;
- Chapter 6: APIs, services, workers, and queues;
- Chapter 7: auth, permissions, files, and storage;
- Chapter 8: async jobs, background processing, and webhooks.

It may reuse Part 1 thinking, but it should not require future course concepts.

Do not require real model calls, embeddings, retrieval, agents, LLMOps, or production deployment here. Later parts can upgrade this skeleton:

- Part 3 can add schemas, data quality, and document pipelines;
- Part 4 can replace the mock model gateway with real model routing;
- Part 5 can add retrieval;
- Part 7 can add tools and workflow actions;
- Part 8 can add real observability and eval infrastructure;
- Part 9 can harden security and governance.

## User Story

Primary user:

```text
Builder creating reusable foundation for future AI systems.
```

User story:

```text
As an AI application builder,
I want a clean app skeleton with request handling, storage, jobs, roles, and traces,
so future AI features do not become tangled scripts.
```

The user wants a foundation that is boring in the right way:

```text
clear boundaries
predictable state
visible job status
explicit permissions
traceable behavior
replaceable model gateway
```

## System Requirements

### Interface

Choose one implementation path:

```text
HTTP API:
Run local server and call endpoints.

CLI:
Run commands from terminal.

Minimal web app:
Use a simple local UI over the same backend.
```

CLI or HTTP API is enough for this project.

### Required Capabilities

The app must support:

- creating a user;
- assigning a role;
- submitting an AI-style request;
- saving request state;
- importing or uploading a document;
- creating a background job for document processing;
- checking job status;
- calling a mock model gateway;
- writing a trace log;
- blocking at least one unauthorized action.

### Roles

Use three roles:

```text
admin:
Can create users, submit requests, import documents, run verification.

operator:
Can submit requests and import documents.

viewer:
Can read request status and job status only.
```

This is not full authentication. It is an auth boundary simulation.

Do not build OAuth, sessions, password reset, or production identity here. The goal is to understand where authorization checks belong.

## Starter Data

Create seed data for local demo.

### Users

```json
[
  {
    "id": "U-001",
    "name": "Asha",
    "role": "admin"
  },
  {
    "id": "U-002",
    "name": "Ben",
    "role": "operator"
  },
  {
    "id": "U-003",
    "name": "Mira",
    "role": "viewer"
  }
]
```

### Documents

Create a sample file:

```text
sample-policy.txt
```

Content:

```text
Refund requests over $100 require manager review.
API key reset requests should be handled by sending the official reset guide.
Account ownership changes require confirmation from the current owner.
```

### Requests

Example request:

```json
{
  "user_id": "U-002",
  "input": "Summarize the refund policy from the uploaded document.",
  "request_type": "summary"
}
```

## MVP Milestone

Build the simplest useful skeleton first.

MVP must:

1. Start locally.
2. Load or create users.
3. Submit a request.
4. Save request state.
5. Call a mock model gateway.
6. Save response.
7. Write trace log.
8. Show request by ID.

MVP flow:

```text
submit request
  -> authorize user
  -> create request record
  -> call mock model gateway
  -> update request status
  -> write trace
  -> return response
```

Mock model gateway response:

```json
{
  "output": "Mock response for task: summary",
  "model_name": "mock-model",
  "tokens_in": 0,
  "tokens_out": 0
}
```

Do not call a real model in the MVP.

## V1 Milestone

After MVP works, add the software system pieces that make AI apps reliable.

V1 should include:

- file import or upload;
- document record in storage;
- background job creation;
- worker function that processes the file;
- job status transitions;
- role-based permission checks;
- error states;
- trace viewer or trace export.

Job flow:

```text
import document
  -> authorize user
  -> save document metadata
  -> create ingestion job
  -> worker reads file
  -> worker stores parsed text preview
  -> mark job complete or failed
  -> write trace event
```

Job statuses:

```text
queued
processing
completed
failed
```

## System Boundary Milestone

Add a short architecture note named `system-boundary.md`.

It should explain:

- what the UI or CLI is allowed to do;
- what the service layer owns;
- where authorization checks happen;
- where files are stored;
- where background work happens;
- where the model gateway sits;
- where traces are written;
- which parts are mock placeholders.

This keeps the project aligned with Part 2: software boundaries before AI sophistication.

## Core Components

### 1. Controller or Command Layer

Responsibilities:

- parse request or command input;
- call service functions;
- return formatted output;
- avoid direct storage or model logic.

Example HTTP endpoints:

```text
POST /users
GET /users/:id
POST /requests
GET /requests/:id
POST /documents
GET /jobs/:id
GET /traces/:id
```

Example CLI commands:

```bash
app create-user --name Asha --role admin
app submit-request --user U-002 --input "Summarize refund policy"
app show-request REQ-001
app import-document --user U-002 --file sample-policy.txt
app show-job JOB-001
app show-trace TR-001
```

### 2. Authorization Check

Create one clear function:

```text
can(user, action, resource)
```

Example rules:

```text
admin can do everything.
operator can submit_request and import_document.
viewer can read_request and read_job.
viewer cannot submit_request.
viewer cannot import_document.
```

Expected behavior:

```text
If viewer tries to import document,
system returns permission denied and writes trace event.
```

### 3. Storage Layer

Use SQLite, JSON files, or another simple local store.

Store:

- users;
- requests;
- documents;
- jobs;
- traces.

Do not scatter persistence across random files without structure.

Recommended tables or collections:

```text
users
requests
documents
jobs
traces
```

### 4. Request Service

Responsibilities:

- authorize user;
- create request record;
- call model gateway placeholder;
- update request status;
- write trace.

Request statuses:

```text
pending
completed
failed
denied
```

### 5. Document Service

Responsibilities:

- authorize import;
- validate file exists;
- save document metadata;
- create ingestion job;
- return job ID.

Allowed file types for this project:

```text
.txt
.md
.csv
```

Do not implement PDF parsing here. That belongs to later multimodal/document projects.

### 6. Background Worker

The worker can be simple.

Options:

- run synchronously after job creation;
- run with a local queue list;
- run with a `process-jobs` command;
- run with an in-memory worker loop.

The important part is state transition:

```text
queued -> processing -> completed
queued -> processing -> failed
```

Failure should be visible.

### 7. Model Gateway Placeholder

Create a replaceable boundary:

```text
call_model(task, input, context)
```

For this project, return mock output.

The point is not intelligence. The point is that application code does not call model vendors directly from controllers, UI components, or random scripts.

Gateway output:

```json
{
  "model_name": "mock-model",
  "task": "summary",
  "output": "Mock summary response.",
  "latency_ms": 12,
  "cost_estimate": 0
}
```

### 8. Trace Logger

Every important operation should create a trace or trace event.

Trace fields:

```json
{
  "trace_id": "TR-001",
  "request_id": "REQ-001",
  "user_id": "U-002",
  "role": "operator",
  "operation": "submit_request",
  "status": "completed",
  "events": [
    "authorized",
    "request_created",
    "model_gateway_called",
    "request_completed"
  ],
  "latency_ms": 42,
  "created_at": "2026-01-10T10:00:00Z"
}
```

Trace logging here is simple application logging. Full observability belongs to Part 8.

## Architecture

```text
CLI or API
  -> controller/command handler
  -> authorization check
  -> service layer
  -> storage layer
  -> background worker
  -> model gateway placeholder
  -> trace logger
```

Key boundary:

```text
UI/CLI does not own AI behavior.
Controller does not own business logic.
Model gateway does not own permissions.
Storage does not decide policy.
```

## Data Model

### User

```json
{
  "id": "U-001",
  "name": "Asha",
  "role": "admin",
  "created_at": "2026-01-10T10:00:00Z"
}
```

### Request

```json
{
  "id": "REQ-001",
  "user_id": "U-002",
  "request_type": "summary",
  "input": "Summarize refund policy",
  "status": "completed",
  "output": "Mock summary response.",
  "created_at": "2026-01-10T10:00:00Z",
  "updated_at": "2026-01-10T10:00:03Z"
}
```

### Document

```json
{
  "id": "DOC-001",
  "user_id": "U-002",
  "filename": "sample-policy.txt",
  "content_type": "text/plain",
  "status": "processed",
  "text_preview": "Refund requests over $100 require manager review.",
  "created_at": "2026-01-10T10:00:00Z"
}
```

### Job

```json
{
  "id": "JOB-001",
  "type": "document_ingestion",
  "document_id": "DOC-001",
  "status": "completed",
  "attempts": 1,
  "error": null,
  "created_at": "2026-01-10T10:00:00Z",
  "updated_at": "2026-01-10T10:00:05Z"
}
```

### Trace

```json
{
  "id": "TR-001",
  "request_id": "REQ-001",
  "job_id": null,
  "user_id": "U-002",
  "operation": "submit_request",
  "status": "completed",
  "events": ["authorized", "request_created", "model_gateway_called", "request_completed"],
  "latency_ms": 42
}
```

## Verification Dataset

Create a small verification script or checklist with these cases:

```text
Case 1:
operator submits request.
Expected: request completed, mock output saved, trace created.

Case 2:
viewer submits request.
Expected: permission denied, denied trace created.

Case 3:
operator imports sample-policy.txt.
Expected: document saved, job created, job completed.

Case 4:
operator imports missing file.
Expected: job or request fails visibly with error.

Case 5:
admin reads all records.
Expected: allowed.
```

## Seeded Failure Cases

Include failure paths on purpose.

### Case 1: Unauthorized User

```text
Viewer tries to submit a request.
Expected: denied.
```

### Case 2: Missing File

```text
Operator imports file path that does not exist.
Expected: failed job or validation error, not crash.
```

### Case 3: Invalid Role

```text
Create user with role "super_ai_admin".
Expected: validation error.
```

### Case 4: Empty Request

```text
Operator submits empty input.
Expected: validation error.
```

### Case 5: Worker Failure

```text
Worker receives unsupported file type.
Expected: job marked failed with reason.
```

## Acceptance Rubric

### Basic Pass

- app runs locally;
- users can be created or seeded;
- request can be submitted;
- mock model gateway returns response;
- request state is stored;
- trace is created.

### Strong Pass

- role checks block unauthorized actions;
- document import creates job;
- worker updates job status;
- failures are visible;
- storage has clear tables or collections;
- controller, service, storage, gateway, and logger are separate modules.

### Excellent

- includes verification command or tests;
- includes seeded failure cases;
- supports both request and job traces;
- exports data for inspection;
- includes `system-boundary.md`;
- future projects can reuse the skeleton without rewriting core structure.

## Metrics

Measure:

- request success path works;
- unauthorized action blocked;
- job status transitions correctly;
- failed job records useful error;
- trace created for each operation;
- no controller directly calls storage and model gateway together without service boundary.

Suggested targets:

```text
Required verification cases passing: 100%
Unauthorized restricted actions blocked: 100%
Operations with trace record: 100%
```

## Final Deliverables

At the end, the learner should have:

- runnable CLI or local API;
- seed users;
- sample document;
- local storage;
- mock model gateway;
- background job simulation;
- trace log;
- verification script or manual checklist;
- `system-boundary.md`;
- short project `README.md`.

Example file layout:

```text
project-02-ai-app-skeleton/
  README.md
  system-boundary.md
  data/
    sample-policy.txt
    seed-users.json
  src/
    app.py
    auth.py
    services/
      requests.py
      documents.py
      jobs.py
    storage/
      store.py
    ai/
      model_gateway.py
    tracing/
      logger.py
  tests/
    verify_skeleton.py
  local_data/
    app.db or app.json
```

The exact language and framework can differ. The boundaries should not.

## Demo Script

A 3-minute demo should show:

1. Start the app.
2. Show seeded users and roles.
3. Submit request as operator.
4. Show stored request and mock response.
5. Import sample document.
6. Show job status moving to completed.
7. Attempt restricted action as viewer.
8. Show denied result and trace.
9. Open `system-boundary.md`.

Demo narrative:

```text
This project does not optimize model quality.
It proves the AI system has a real software skeleton:
requests enter through a boundary,
permissions are checked,
state is stored,
slow work is modeled as a job,
model access is isolated behind a gateway,
and every important operation leaves a trace.
```

## What Learner Understands

After completing this project, the learner should understand:

- AI apps are software systems before they are model calls;
- controllers, services, storage, workers, and gateways have different jobs;
- permissions should be checked before work happens;
- file handling needs explicit state;
- slow work should not be hidden inside a request path;
- model calls should sit behind replaceable boundaries;
- traces make behavior inspectable;
- future AI features become easier when the skeleton is clean.

## Extension

Later upgrades:

- Part 3: add schema validation and document data quality checks;
- Part 4: connect real model provider and route by task;
- Part 5: add retrieval over imported documents;
- Part 7: add safe tool actions;
- Part 8: add observability dashboard and eval runner;
- Part 9: add privacy policy, retention, and threat model.
