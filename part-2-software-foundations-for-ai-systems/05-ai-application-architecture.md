# Chapter 5: AI Application Architecture

Chapter 4 ended Part 1 with baseline, metrics, and proof of impact. Now the question changes:

> Where does the AI system actually live?

Before data pipelines, prompts, retrieval, models, tools, or agents, learner needs software architecture. AI features do not float in air. They live inside applications with users, APIs, databases, jobs, files, logs, permissions, and releases.

## Real Problem

A team wants to build support assistant.

They imagine:

```text
user -> model -> answer
```

Real system is closer to:

```text
user
-> web app
-> backend API
-> auth check
-> ticket database
-> document store
-> retrieval service
-> model gateway
-> validation
-> feedback log
-> observability
-> response
```

If architecture is unclear, every later AI decision becomes fragile.

## Bad Default Solution

Bad architecture:

- frontend calls model directly;
- prompts hardcoded in UI;
- no backend state;
- no user permissions;
- no source versioning;
- no eval endpoint;
- no logs;
- no retry/fallback path.

This may demo fast. It cannot be trusted.

## First Principles

AI application is software system with probabilistic components.

Core components:

- client/UI;
- backend API;
- database;
- file/document store;
- background workers;
- model gateway;
- retrieval service;
- tool service;
- evaluation system;
- observability/logging;
- auth and permissions.

The model is one component, not architecture.

## System Decision

Start with architecture map:

```text
User:
Primary workflow:
Client:
Backend API:
Data store:
Document store:
Model calls:
Retrieval:
Tools:
Jobs:
Auth:
Logs:
Evaluation:
```

Decision rule:

- UI gathers intent.
- Backend owns control.
- Database owns durable facts.
- File store owns documents.
- Workers own slow jobs.
- Model gateway owns model calls.
- Eval system owns quality checks.
- Logs own traceability.

## Minimal Build

Create `architecture-map.md`.

Template:

```text
System:
User:
Workflow:

Components:
- UI:
- API:
- Database:
- File storage:
- Background worker:
- Model gateway:
- Retrieval/index:
- Tool service:
- Eval runner:
- Observability:

Data flow:
1.
2.
3.

Trust boundaries:
- user input:
- retrieved content:
- model output:
- tool action:

Failure points:
- 
```

## Failure Modes

Architecture fails when:

- model call bypasses backend;
- user permissions ignored;
- documents and database facts mixed;
- slow ingestion blocks user request;
- model output treated as trusted;
- prompt changes not versioned;
- evals separate from release;
- logs omit intermediate steps.

## Evaluation

Review architecture with 10 workflow examples:

- normal request;
- missing data;
- long document;
- unauthorized user;
- model invalid output;
- retrieval miss;
- slow model call;
- tool failure;
- user correction;
- rollback.

Pass if architecture has place for each behavior.

## Improvement

Improve by:

- adding backend control point;
- moving slow work to worker;
- adding model gateway;
- separating DB and document store;
- adding auth before retrieval;
- adding trace IDs;
- adding eval runner.

## Proof Artifact

Create `architecture-map.md`.

Include:

- component map;
- data flow;
- trust boundaries;
- failure points;
- 10-example architecture review.

## Decision Checklist

- [ ] Model is not called directly from untrusted UI.
- [ ] Backend owns workflow control.
- [ ] Durable facts live in database.
- [ ] Documents live in document store.
- [ ] Slow jobs have worker path.
- [ ] Auth exists before sensitive data access.
- [ ] Logs/evals have system location.

## Active Recall

1. Why is model not architecture?
2. What should backend own?
3. Why separate database and document store?
4. What is trust boundary?
5. Why do evals need architecture location?

## Next

This chapter mapped the whole AI application. Next chapter zooms into backend structure: APIs, services, workers, and queues.

