# Chapter 5: AI Application Architecture

Part 1 answered:

```text
What work should improve?
Where does AI belong?
What is the system boundary?
How will success be measured?
```

Part 2 asks a new question:

> Where does the AI system actually live inside software?

AI features do not float in the air. They live inside applications with users, APIs, databases, files, jobs, permissions, logs, evals, and releases.

## Concept Overview

AI application architecture is the shape of the software system around AI components.

It answers:

```text
Where does user input enter?
Where is state stored?
Where are documents stored?
Where are model calls made?
Where does retrieval happen?
Where are permissions checked?
Where are outputs validated?
Where are traces logged?
Where do evals run?
```

The model is not the architecture.

The model is one component inside architecture.

Good architecture gives every responsibility a home.

## Why It Exists

A team wants to build a support assistant.

They imagine:

```text
user -> model -> answer
```

This can work in a demo.

Real product needs more:

- user login;
- ticket data;
- permission checks;
- document storage;
- retrieval;
- model calls;
- validation;
- human feedback;
- trace logs;
- evals;
- deployment;
- rollback.

Real architecture looks closer to:

```text
user
-> UI
-> backend API
-> auth/permission check
-> ticket database
-> document store/search
-> model gateway
-> output validator
-> feedback/eval log
-> response
```

Architecture exists because AI output depends on everything around the model.

If architecture is weak, later AI work becomes fragile:

- prompts hardcoded in UI;
- model sees data user cannot access;
- document ingestion blocks user request;
- output is not validated;
- no trace exists when answer is wrong;
- evals cannot reproduce production behavior.

## Mental Model

Think of AI application as controlled flow:

```text
client/UI
-> backend control point
-> data and document access
-> AI components
-> validation
-> storage/logs
-> response
```

Each layer has a job:

| Layer | Job |
|---|---|
| UI/client | collect intent, show output, support correction |
| Backend API | own workflow control |
| Auth/permissions | decide what user can access |
| Database | store durable structured facts |
| File/document store | store documents, uploads, raw files |
| Retrieval/index | find relevant knowledge |
| Model gateway | call models through controlled interface |
| Validator | check structure, citations, safety, rules |
| Worker/queue | handle slow or async jobs |
| Observability | log traces, cost, latency, failures |
| Eval system | test behavior before and after changes |

Architecture is responsibility placement.

The key question:

```text
Where should this responsibility live?
```

## Internal Mechanics

An AI application usually has these components.

### UI or Client

The UI gathers user intent and displays result.

It should not own sensitive model logic.

It may show:

- input form;
- chat or task interface;
- citations;
- confidence/review flag;
- approval controls;
- correction controls.

### Backend API

Backend is the control point.

It should own:

- workflow routing;
- auth checks;
- model request construction;
- validation;
- persistence;
- logging;
- retries/fallbacks.

Avoid direct model calls from untrusted UI.

### Database

Database stores durable structured facts:

- users;
- tickets;
- records;
- workflow state;
- approvals;
- eval runs;
- audit logs.

Model memory should not replace database truth.

### File and Document Store

Documents are often not database rows.

Use file/document storage for:

- PDFs;
- images;
- transcripts;
- uploaded files;
- raw source documents;
- parsed document versions.

Later retrieval systems will index these documents.

### Retrieval or Index

Retrieval finds relevant knowledge.

It may include:

- keyword search;
- vector index;
- metadata filters;
- reranker;
- source authority rules.

Retrieval should respect permissions.

### Model Gateway

Model gateway is a controlled path for model calls.

It can own:

- model selection;
- prompt/interface version;
- retries;
- fallbacks;
- cost tracking;
- latency tracking;
- provider abstraction.

Do not scatter model calls across random files.

### Workers and Queues

Slow tasks should not block the user request.

Use workers for:

- document ingestion;
- OCR;
- embedding;
- batch evals;
- long report generation;
- webhook processing.

### Observability and Evals

AI systems need traces.

Trace should connect:

- request;
- user role;
- retrieved sources;
- model version;
- prompt version;
- output;
- validation result;
- cost;
- latency;
- feedback.

Evals need architecture location so quality can be tested repeatedly.

## Real-World Example

Build support reply drafting system from Part 1.

Bad architecture:

```text
support UI -> model API -> draft answer
```

Problems:

- UI contains prompt;
- no permission check;
- model cannot fetch account facts safely;
- documents are pasted manually;
- output has no citation validation;
- no eval trace;
- no feedback loop.

Better architecture:

```text
support UI
-> backend API
-> auth check
-> ticket database
-> account API
-> document retrieval service
-> model gateway
-> citation/output validator
-> draft store
-> feedback log
-> response to UI
```

Now each responsibility has a place.

Example flow:

```text
1. User opens ticket T-104.
2. UI sends draft request to backend.
3. Backend checks user can access ticket.
4. Backend fetches ticket and account plan.
5. Retrieval service finds relevant policy docs.
6. Model gateway asks model to draft reply with citations.
7. Validator checks schema and citation sources.
8. Draft is saved with trace id.
9. UI shows draft, sources, and edit controls.
10. User accepts, edits, or rejects.
```

This architecture supports later chapters:

- data ingestion;
- model interfaces;
- retrieval;
- tool use;
- evals;
- observability;
- security.

## Common Mistakes and Failure Modes

### Mistake 1: Frontend Calls Model Directly

This exposes prompts, weakens permission checks, and makes logging harder.

### Mistake 2: No Backend Control Point

Without backend control, workflow logic spreads across UI, scripts, and model calls.

### Mistake 3: Mixing Database Facts and Documents

Database facts and documents have different lifecycles.

Account status belongs in database/API.

Policy PDF belongs in document store and retrieval index.

### Mistake 4: Slow Work in User Request

OCR, embedding, ingestion, and long evals should often run in workers.

### Mistake 5: No Model Gateway

Scattered model calls make it hard to manage cost, routing, prompts, versions, and fallbacks.

### Mistake 6: Output Treated as Trusted

Model output should be validated before storage, display, or action.

### Mistake 7: No Trace

If system cannot explain what happened, it cannot improve reliably.

## System Design Decisions

Before implementing, decide:

```text
What does UI own?
What does backend own?
Where are durable facts stored?
Where are documents stored?
What runs synchronously?
What runs in background worker?
Where are model calls centralized?
Where does retrieval happen?
Where are permissions checked?
Where are outputs validated?
Where are traces and evals stored?
```

Decision rule:

| Responsibility | Better Home |
|---|---|
| collect user intent | UI |
| workflow control | backend API |
| durable facts | database |
| raw documents | file/document store |
| slow ingestion | worker/queue |
| model calls | model gateway |
| trusted source lookup | retrieval service/API |
| permission checks | backend/policy layer |
| output validation | backend validator |
| quality checks | eval runner |
| debugging | observability/traces |

If you cannot place a responsibility, architecture is not ready.

## Build Artifact

Create `architecture-map.md`.

Template:

```text
System:
Primary user:
Workflow:

Components:
- UI/client:
- Backend API:
- Database:
- File/document store:
- Retrieval/index:
- Model gateway:
- Background worker:
- Tool service:
- Validator:
- Eval runner:
- Observability:
- Auth/permissions:

Data flow:
1.
2.
3.

Trust boundaries:
- user input:
- database facts:
- retrieved content:
- model output:
- tool action:

Failure points:
-
```

Worked example:

```text
System: Support Reply Drafting System
Primary user: support teammate
Workflow: draft reply for product support ticket

Components:
- UI/client: ticket screen with draft panel and citation view
- Backend API: receives draft request and owns workflow
- Database: users, tickets, drafts, feedback, trace ids
- File/document store: approved support docs and changelog files
- Retrieval/index: searches approved docs by product area and metadata
- Model gateway: drafts reply with controlled prompt version
- Background worker: ingests docs and updates index
- Validator: checks JSON schema and citation ids
- Eval runner: tests 30 ticket examples before release
- Observability: logs trace, latency, cost, retrieval sources
- Auth/permissions: confirms user can access ticket and docs

Data flow:
1. UI sends draft request for ticket T-104.
2. Backend checks auth and fetches ticket/account facts.
3. Retrieval returns approved sources.
4. Model gateway drafts reply.
5. Validator checks schema/citations.
6. Draft and trace are stored.
7. UI shows draft and sources to user.

Trust boundaries:
- user input: untrusted
- database facts: trusted source
- retrieved content: trusted only if approved and authorized
- model output: untrusted until validated
- tool action: requires permission and approval if risky

Failure points:
- unauthorized ticket access
- retrieval miss
- invalid model output
- citation mismatch
- slow model response
- user rejects draft
```

Acceptance criteria:

- model is not called directly from untrusted UI;
- backend owns workflow control;
- durable facts and documents are separated;
- slow jobs have worker path;
- permissions happen before sensitive data access;
- model output has validation path;
- logs/evals have architecture location.

## Active Recall Questions

1. Why is the model not the architecture?
2. What should backend API own?
3. Why separate database and document store?
4. What is model gateway?
5. Why should slow jobs use workers or queues?
6. What is trust boundary?
7. Why do evals need a place in architecture?

## Summary

AI applications are software systems with probabilistic components.

The durable idea:

```text
AI feature = UI + backend + data + documents + retrieval + model gateway + validation + logs + evals
```

Good architecture gives every responsibility a home.

Before building prompts, retrieval, tools, or agents, decide where these components live and how data flows between them.

## Preview of Next Chapter

This chapter mapped the whole AI application.

Next chapter zooms into backend execution:

> How do APIs, services, workers, and queues move work through the system?
