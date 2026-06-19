# Chapter 9: Data, Documents, and State

Part 2 built software foundation: APIs, services, storage, workers, auth, and async flow. Now the system needs information.

AI systems do not become useful because a model is attached. They become useful when correct information reaches correct component at correct time under correct permissions.

## Real Problem

A support teammate asks:

> Can we build an AI assistant that answers customer questions from our company knowledge?

Before model, ask:

> What kind of knowledge?

Some knowledge lives in structured data:

- customer plan;
- order status;
- account permissions;
- refund date;
- subscription state.

Some knowledge lives in documents:

- help center articles;
- policy pages;
- product docs;
- release notes;
- internal playbooks.

Some knowledge lives in state:

- current conversation;
- previous action;
- ticket status;
- approval history;
- unresolved task.

If these are mixed together carelessly, system answers wrong questions with confidence.

## Bad Default Solution

Bad solution:

1. Dump all documents into vector database.
2. Connect model.
3. Ask questions.
4. Hope retrieval finds truth.

This ignores:

- database facts;
- document freshness;
- conflicting sources;
- permissions;
- state transitions;
- auditability.

Example failure:

Customer asks:

> Can I get refund?

Policy document says refunds allowed within 30 days. Database says customer bought 45 days ago. If system only retrieves policy, answer may be wrong. If system only checks database, it may miss exception policy.

## First Principles

AI system uses three information types.

### Data

Structured facts:

- tables;
- records;
- events;
- transactions;
- forms.

Use data for exact facts.

### Documents

Human-readable knowledge:

- policies;
- procedures;
- explanations;
- examples;
- playbooks.

Use documents for meaning and guidance.

### State

Current or historical process memory:

- current step;
- approval status;
- previous tool call;
- pending job;
- user correction.

Use state to continue workflow.

Do not use model as database. Do not use vector search as source of truth for exact facts. Do not use chat history as audit log.

## System Decision

Classify every information need:

| Need | Best source | Access method |
|---|---|---|
| Exact current fact | Database/API | deterministic query |
| Policy/explanation | Document | search/retrieval |
| Current task progress | State store | workflow state |
| User approval | Audit log | deterministic check |
| Ambiguous text meaning | Model | extraction/classification |

## Minimal Build

Create `information-map.md`:

```text
Workflow:
User:
Output:

Structured data:
- item:
  source:
  owner:
  freshness:
  access rule:

Documents:
- item:
  source:
  authority:
  freshness:
  conflicts:

State:
- item:
  lifespan:
  owner:
  reset rule:

Unknown information:
- item:
  consequence:
  workaround:
```

## Failure Modes

Information design fails when:

- stale document overrides live database;
- model invents missing facts;
- private document leaks to unauthorized user;
- chat memory becomes hidden source of truth;
- conflicting documents have no authority order;
- state persists longer than user expects.

## Evaluation

Test map with real questions:

1. database-only;
2. document-only;
3. database + document;
4. current state;
5. unavailable information.

Pass if system knows source path for each question.

## Proof Artifact

Create `information-map.md`.

## Decision Checklist

- [ ] Exact facts come from database/API.
- [ ] Policy/explanation comes from documents.
- [ ] Task progress lives in state.
- [ ] Source authority is clear.
- [ ] Freshness rules are clear.
- [ ] Permissions are clear.
- [ ] Unknown information is named.

## Active Recall

1. Why should model not act as database?
2. How are data, documents, and state different?
3. What happens when stale document beats live data?
4. Why does source authority matter?
5. What should live in state instead of prompt?

## Next

This chapter classified information. Next chapter shows how information enters system: ingestion, parsing, cleaning, and normalization.

