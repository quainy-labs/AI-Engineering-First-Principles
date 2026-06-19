# Project 2: AI App Skeleton

## Build

Build a minimal AI application skeleton that can support future projects: API, storage, background jobs, auth boundary, model gateway, logs, and evaluation hook.

This project is not about model quality. It is about where AI work lives in real software.

## User

Builder who needs reusable foundation for AI applications.

## Product Outcome

You will have small runnable app with:

- backend API;
- simple UI or CLI;
- database or local persistence;
- file upload path;
- background job simulation;
- model gateway interface;
- auth/role placeholder;
- trace logging;
- eval command placeholder.

## Core Features

### 1. API Layer

Create endpoints or CLI commands:

```text
POST /requests
GET /requests/:id
POST /documents
GET /jobs/:id
POST /evals/run
```

CLI alternative:

```bash
submit-request
show-request
upload-document
show-job
run-eval
```

### 2. Storage Layer

Store:

- users;
- requests;
- documents;
- jobs;
- traces;
- eval runs.

Local JSON/SQLite is enough.

### 3. Background Job Path

Simulate document ingestion:

```text
upload document
-> create job
-> worker parses document
-> marks job complete/failed
```

### 4. Model Gateway

Create function/module:

```text
call_model(task, input, context, output_schema)
```

In v1, it can use mock response. Goal is interface, not model vendor.

### 5. Auth Boundary

Add role placeholder:

- admin;
- operator;
- viewer.

Permission examples:

- only admin can run eval;
- operator can submit request;
- viewer can read result.

### 6. Trace Logger

Each request gets trace:

```json
{
  "trace_id": "TR-001",
  "user_id": "U-001",
  "role": "operator",
  "request": "Summarize this ticket",
  "model_calls": [],
  "tool_calls": [],
  "status": "completed",
  "latency_ms": 120
}
```

## Architecture

```text
UI/CLI
-> API/controller
-> auth check
-> service layer
-> storage
-> job queue/worker
-> model gateway
-> trace logger
-> eval runner
```

## Data Model

```json
{
  "request": {
    "id": "REQ-001",
    "user_id": "U-001",
    "input": "text",
    "status": "pending",
    "created_at": "2026-01-10T10:00:00Z"
  },
  "job": {
    "id": "JOB-001",
    "type": "document_ingestion",
    "status": "processing",
    "retries": 0
  }
}
```

## Evaluation

Test:

- submit request;
- upload document;
- background job completes;
- unauthorized role blocked;
- trace created;
- eval command runs placeholder check;
- failure state visible.

Acceptance:

- no direct model call from UI;
- backend owns control;
- jobs have status;
- traces exist;
- permissions checked before restricted action;
- future projects can reuse skeleton.

## What Learner Understands

- AI apps are software systems.
- Model gateway isolates vendor/model details.
- Slow AI work needs background processing.
- Auth and traces are architectural concerns, not production garnish.

## Extension

Add:

- real web UI;
- real model provider;
- SQLite/Postgres;
- queue library;
- file storage;
- eval dashboard;
- deployment config.
