# Chapter 44: Privacy, PII, Data Retention, and Access Control

Chapter 43 focused on manipulation attacks: prompt injection, retrieval poisoning, and tool abuse.

This chapter focuses on sensitive data.

AI systems often touch private documents, customer records, emails, call transcripts, screenshots, support tickets, tool results, prompts, embeddings, traces, and feedback. Privacy cannot be added after those data flows already exist.

The Quainy principle is:

> The safest sensitive data is the data your AI system never receives, never stores, and never logs.

## Concept Overview

Privacy in AI engineering means controlling how personal, confidential, regulated, or sensitive data enters, moves through, leaves, and remains inside an AI system.

PII means personally identifiable information: data that can identify, contact, locate, or describe a specific person directly or indirectly.

Examples:

- name;
- email;
- phone number;
- address;
- government identifier;
- employee ID;
- customer ID;
- account number;
- payment details;
- device identifiers;
- location;
- face or voice data;
- health information;
- financial information;
- private messages;
- support history;
- behavioral records.

Privacy is broader than PII.

It also includes:

- confidential contracts;
- internal strategy;
- source code;
- security logs;
- sales notes;
- customer documents;
- screenshots;
- call transcripts;
- credentials;
- trade secrets;
- model traces.

Access control decides who or what can see data or perform actions.

Data retention decides how long data is stored.

Redaction removes or masks sensitive parts.

Deletion removes data when it is no longer needed or when policy requires removal.

Audit logs record who accessed what and what actions occurred.

In AI systems, these concepts must apply not only to databases, but also to:

- prompts;
- retrieved chunks;
- context windows;
- embeddings;
- vector indexes;
- model outputs;
- tool arguments;
- tool results;
- conversation logs;
- evaluation datasets;
- fine-tuning datasets;
- screenshots;
- audio transcripts;
- human feedback;
- trace exports.

Privacy fails when teams protect the main database but forget the AI pipeline around it.

## Why It Exists

AI systems increase privacy risk because they copy data into many places.

A normal application might show a record to an authorized user.

An AI application may:

1. retrieve the record;
2. insert it into a prompt;
3. combine it with other records;
4. send it to a model provider or inference service;
5. generate a summary;
6. store the prompt and output in traces;
7. use the trace for evaluation;
8. include the failure case in a test set;
9. store embeddings in an index;
10. show the output to another user.

Each step creates a privacy decision.

Questions the system must answer:

- Was the user allowed to access the source data?
- Did the task require this data?
- Was the minimum necessary data used?
- Was sensitive data sent to a provider?
- Was it stored in logs?
- Was it stored in memory?
- Was it included in evals?
- Was it embedded in an index?
- Can it be deleted later?
- Who can inspect the trace?
- Can another tenant retrieve it?
- Can support staff view it?
- Is the output allowed to reveal it?

Privacy exists because AI systems blur boundaries between data use, reasoning, logging, improvement, and product experience.

The first-principles rule is:

```text
Data should have a purpose, permission, minimum scope, retention limit, and deletion path.
```

If the team cannot explain why data is needed, who can access it, how long it stays, and how it can be removed, the data flow is not ready.

This chapter is not legal advice. Privacy laws and contractual duties vary by region, industry, and customer. The engineering responsibility is to make privacy controllable, inspectable, and enforceable so legal, security, and product teams can apply the right requirements.

## Mental Model

Think of sensitive data as water moving through pipes.

You need to know:

- where it enters;
- which pipes it flows through;
- which tanks store it;
- which filters clean it;
- which valves restrict it;
- where it exits;
- how to drain it.

In AI systems, the pipes are:

```text
source systems
  -> ingestion
  -> chunking
  -> embedding
  -> vector index
  -> retrieval
  -> context assembly
  -> model call
  -> output
  -> tool call
  -> memory
  -> traces
  -> eval datasets
  -> analytics
```

Privacy engineering means placing controls at each pipe.

The mental model is:

```text
classify -> minimize -> authorize -> redact -> use -> log carefully -> retain briefly -> delete reliably -> audit
```

Classify:

```text
What kind of data is this?
```

Minimize:

```text
Does the model need this exact data for this task?
```

Authorize:

```text
Is this user allowed to access this data or action?
```

Redact:

```text
Can sensitive fields be masked before model, log, or output?
```

Use:

```text
Is the data used only for the stated task?
```

Log carefully:

```text
What trace data is stored, and who can inspect it?
```

Retain briefly:

```text
How long does this data need to exist?
```

Delete reliably:

```text
Can the system remove source data, derived data, traces, indexes, and eval copies when required?
```

Audit:

```text
Can we prove who accessed what and why?
```

The privacy mistake is to think only about the first database.

The AI privacy mindset follows data through the entire system.

## Internal Mechanics

Privacy design has several practical layers.

### Data Classification

Data classification gives the system a vocabulary for risk.

A simple classification:

```text
Public:
Approved for anyone to see.

Internal:
For employees or approved users only.

Confidential:
Sensitive business information with restricted access.

Personal:
Information linked to a person.

Regulated:
Data subject to legal, contractual, or industry controls.

Secret:
Credentials, keys, security-sensitive data, or highly restricted information.
```

Classification should attach to data sources, fields, documents, chunks, traces, and outputs where possible.

Example:

```text
Document: employee_benefits_policy.pdf
Class: internal
PII: no
Allowed in retrieval: yes, employees only
Allowed in traces: chunk IDs only
Retention: source policy
```

Another example:

```text
Document: customer_contract_acme.pdf
Class: confidential
PII: possible
Allowed in retrieval: account team only
Allowed in traces: redacted excerpts only
Allowed in evals: no, unless synthetic or approved
Retention: contract policy
```

Without classification, every downstream decision becomes guesswork.

### Data Minimization

Data minimization means using only what is needed.

Bad pattern:

```text
Send entire customer profile to model for every support answer.
```

Better pattern:

```text
Send only the fields needed for the current task:
- product plan;
- latest ticket summary;
- relevant policy excerpt.
Exclude billing details unless the task requires them.
```

AI systems often over-collect context because larger context windows make it easy.

But larger context increases:

- privacy exposure;
- cost;
- latency;
- hallucination surface;
- logging risk;
- access-control complexity.

Minimum necessary context is both a privacy practice and a quality practice.

### Access Before Retrieval

Access control must happen before private data enters model context.

Bad flow:

```text
retrieve all relevant documents
  -> send to model
  -> ask model not to reveal unauthorized data
```

Better flow:

```text
identify user
  -> compute permissions
  -> filter allowed sources/chunks
  -> retrieve only authorized content
  -> assemble context
  -> answer
```

The model should not see data the user is not allowed to see.

This matters for:

- multi-tenant systems;
- employee role permissions;
- customer account boundaries;
- confidential projects;
- HR data;
- legal documents;
- health or financial records;
- support tickets.

Retrieval must respect authorization at the same or finer level as the source system.

If the source system has document-level access but the AI index ignores it, the AI system creates a new data leak.

### Redaction and Masking

Redaction removes sensitive values.

Masking hides part of a value.

Examples:

```text
Email:
narendra@example.com -> n*****a@example.com

Phone:
+1-415-555-0148 -> +1-415-***-****

Account:
123456789 -> ****6789
```

Redaction can happen before:

- model calls;
- logging;
- analytics;
- eval creation;
- support review;
- external tool calls;
- generated output.

But redaction is not magic.

If the task requires exact data, redaction may reduce usefulness. If redaction is poorly implemented, it may miss sensitive fields. If the output combines enough facts, a person may still be identifiable.

Use redaction as one control, not the only control.

### Trace and Log Privacy

AI observability creates rich traces.

Traces may include:

- user input;
- retrieved chunks;
- model prompts;
- model outputs;
- tool arguments;
- tool results;
- screenshots;
- audio transcripts;
- memory writes;
- feedback;
- error messages.

These traces are useful for debugging and evaluation.

They are also sensitive.

For each trace field, decide:

```text
store full value
store redacted value
store reference ID only
store aggregate metric only
do not store
```

Example:

```text
Retrieved document:
Store document ID and chunk ID.
Do not store full confidential chunk text unless debugging mode is approved.

Tool argument:
Store action type and record ID.
Redact personal fields.

Generated output:
Store for public documentation assistant.
Redact or restrict for customer data assistant.
```

Trace access should be restricted.

Not every engineer, support agent, or vendor should be able to inspect raw production prompts.

### Retention

Retention defines how long data is kept.

Different data types may need different retention:

```text
Application event metrics: 13 months
Raw prompts: 30 days
Redacted traces: 90 days
Security audit logs: policy-defined period
Uploaded temporary files: 24 hours
Memory: until user deletes or account policy expires
Eval cases: synthetic or approved only
```

The exact values depend on company policy, law, contracts, and product needs.

The engineering requirement is that retention is configurable and enforceable.

Avoid:

```text
Keep everything forever in case we need it.
```

That is not a strategy. It is future risk.

### Deletion and Derived Data

Deletion is harder in AI systems because data may be copied or transformed.

One uploaded document can produce:

- raw file;
- parsed text;
- chunks;
- embeddings;
- vector index entries;
- prompt traces;
- output summaries;
- feedback examples;
- eval cases;
- cached responses;
- memory entries.

Deletion must consider derived data.

Deletion plan:

```text
Source object:
Parsed text:
Chunks:
Embeddings:
Index entries:
Caches:
Traces:
Memory:
Eval copies:
Analytics:
Backups:
```

For some systems, deletion may mean removing future access while backups expire by retention policy. For others, stricter deletion may be required. The important engineering point is to know where copies exist.

If you cannot find derived data, you cannot delete it.

### Consent and Purpose

Some workflows require user notice, consent, contract terms, or policy approval before data is used in certain ways.

Engineering should represent purpose clearly:

```text
Purpose: answer this user request
Purpose: improve system quality
Purpose: monitor abuse
Purpose: create eval case
Purpose: train or fine-tune model
```

Data allowed for one purpose may not be allowed for another.

For example, a customer document may be allowed for answering that customer's question but not allowed for training, fine-tuning, or shared eval datasets.

Purpose should be part of data handling design.

## Real-World Example

Imagine a healthcare operations assistant for internal staff.

It can:

- summarize appointment notes;
- answer policy questions;
- draft patient follow-up messages;
- create administrative tasks;
- search internal procedures;
- log interactions for quality review.

The assistant touches sensitive personal information.

A weak design sends full patient records into every model call and stores all traces indefinitely for debugging.

This may work technically, but it creates serious privacy risk.

A stronger design starts with data classification:

```text
Patient name: personal, sensitive
Contact details: personal
Appointment notes: regulated/sensitive
Internal procedure docs: internal
Staff policy docs: internal
Tool credentials: secret
Trace logs: confidential/sensitive
```

Then it defines access before retrieval:

```text
Staff member identity:
Role:
Assigned clinic:
Patient relationship:
Allowed record fields:
Allowed tools:
```

The assistant does not retrieve patient records unless the staff member is allowed to see them.

The system minimizes context:

```text
For appointment reminder:
- patient first name;
- appointment time;
- clinic location.

Do not include:
- full medical history;
- payment details;
- unrelated notes;
- previous diagnoses.
```

Trace policy:

```text
Store:
- request ID;
- user ID;
- tool names;
- record IDs;
- redacted output;
- latency and cost.

Do not store by default:
- full patient notes;
- full generated message;
- full retrieved chunks.
```

Retention:

```text
Temporary uploaded files: delete after processing window
Raw prompts: short retention or disabled for sensitive workflows
Redacted audit logs: policy-defined retention
Memory: disabled for patient-specific content
Eval cases: synthetic examples only unless explicitly approved
```

Deletion plan:

```text
When a patient record deletion request is processed:
- remove eligible source copies;
- remove parsed text;
- remove vector entries;
- clear caches;
- remove memory entries;
- mark traces according to retention/deletion policy;
- ensure future retrieval cannot return removed content.
```

The result is not "no data." The result is controlled data.

The assistant can still be useful, but every data flow has purpose, permission, scope, retention, and audit.

## Common Mistakes and Failure Modes

### Treating Prompts as Temporary

Teams often assume prompts are temporary because they happen during inference.

But prompts may be stored in:

- application logs;
- model provider logs;
- observability traces;
- debugging exports;
- screenshots;
- support tickets;
- evaluation datasets;
- incident reports.

If sensitive data enters prompts, assume it may need privacy controls.

### Ignoring Embeddings and Indexes

Embeddings may feel abstract, but they are derived from source data.

Vector indexes can preserve access to sensitive content even after the original document changes or is deleted.

If a source document is removed, ask:

- were chunks deleted?
- were embeddings deleted?
- was the index rebuilt?
- were caches cleared?
- were eval copies created?

### Using Production Data for Evals Without Review

Production failures are valuable for evaluation, but they may contain sensitive data.

Before adding a trace to an eval set, decide:

- can it be redacted?
- can it be converted to synthetic form?
- is approval required?
- who can view it?
- how long is it retained?
- can it be deleted later?

The best eval case is often a synthetic version that preserves the failure pattern without preserving private data.

### Confusing Authentication with Authorization

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to access or do?
```

An authenticated employee should not automatically retrieve every internal document.

AI retrieval must respect authorization, not only login status.

### Logging Everything for Debugging

Full logging is tempting during development.

In production, it can become a privacy incident.

Use debug logging only with:

- explicit environment controls;
- short retention;
- access restrictions;
- redaction;
- clear owner;
- audit trail.

### Memory That Stores Sensitive Data

Memory can accidentally preserve data beyond the session where it was needed.

Bad memory:

```text
Remember this user's customer's bank account number.
```

Better memory policy:

```text
Memory may store user preferences.
Memory may not store secrets, payment details, health details, confidential customer data, or sensitive identifiers.
```

Memory writes should be inspectable, editable, and deletable.

### No Deletion Map

If the team cannot list where data copies exist, deletion becomes guesswork.

For every sensitive source, maintain a derived-data map:

```text
source -> parsed text -> chunks -> embeddings -> index -> cache -> trace -> eval -> memory
```

## System Design Decisions

### Decide Data Classes

Create a small classification system that engineers and product people can actually use.

Example:

```text
Public:
Can be used in prompts, logs, evals, and outputs.

Internal:
Can be used for authorized internal users. Logs restricted.

Confidential:
Use only when required. Redact logs. Restrict traces.

Personal:
Access-controlled. Minimize. Retention and deletion required.

Regulated:
Requires policy/legal review. Strong controls. Limited storage.

Secret:
Never send to model. Never log. Store only in secret manager.
```

### Decide Context Rules

For each data class, decide whether it can enter model context.

Example:

```text
Public docs:
Allowed in context.
Allowed in traces.

Confidential contracts:
Allowed only for authorized users.
Trace full text disabled.
Use chunk IDs by default.

Secrets:
Never allowed in model context.

Payment details:
Redact unless task explicitly requires last four digits.
```

### Decide Trace Policy

Trace policy should define:

- fields stored;
- fields redacted;
- access roles;
- retention period;
- export rules;
- support access;
- deletion handling;
- incident review process.

Privacy-friendly traces often store references instead of raw content.

### Decide Retrieval Authorization

For RAG systems, access control belongs in retrieval.

Design options:

```text
Separate indexes by tenant.
Filter chunks by permission metadata.
Check document access before returning chunks.
Use source-system permissions during retrieval.
Rebuild indexes when permissions change.
```

The right design depends on scale and risk, but the principle is stable:

```text
Unauthorized content should not enter context.
```

### Decide Memory Policy

Memory policy should define:

- what may be remembered;
- what must never be remembered;
- who can inspect memory;
- how users can edit memory;
- how memory expires;
- how deletion works;
- whether memory is allowed for sensitive workflows.

Memory should be treated as stored data, not as invisible context.

### Decide Provider and Tool Data Boundaries

If data is sent to external model providers, tools, or APIs, define:

- what data is sent;
- purpose of transfer;
- retention expectations;
- logging behavior;
- encryption requirements;
- access restrictions;
- fallback options;
- customer or contract constraints.

Engineering should make these flows visible. Hidden data transfer is one of the fastest ways to lose trust.

## Build Artifact

Create `privacy-and-access-policy.md`.

Use this structure:

```text
# Privacy and Access Policy

## System
Name:
Purpose:
Users:
Data sources:
Model/provider boundary:

## Data Classes
Class:
Examples:
Allowed in model context:
Allowed in traces:
Allowed in evals:
Retention:
Deletion requirement:

## PII and Sensitive Fields
Field:
Source:
Purpose:
Redaction rule:
Allowed users:
Allowed tools:
Forbidden storage:

## Access Control
User identity source:
Roles:
Tenant/account boundary:
Document permission source:
Chunk-level permission:
Tool permission:
Approval requirement:

## Retrieval Privacy
Data source:
Index location:
Permission filter:
Metadata required:
Freshness rule:
Deletion/reindex process:

## Trace and Logging Policy
Stored fields:
Redacted fields:
Reference-only fields:
Forbidden fields:
Retention:
Access roles:
Export policy:

## Memory Policy
Allowed memory:
Forbidden memory:
User inspection:
User deletion:
Expiration:
Sensitive workflow rule:

## Deletion Map
Source data:
Parsed text:
Chunks:
Embeddings:
Index entries:
Caches:
Traces:
Evals:
Memory:
Backups:

## Audit
Events logged:
Access review cadence:
Owner:
Incident escalation:
```

Artifact complete when another engineer can understand:

- what sensitive data exists;
- who can access it;
- whether it can enter prompts;
- whether it can be logged;
- how long it stays;
- how it is deleted;
- how access is audited.

## Active Recall Questions

1. Why is privacy broader than PII?
2. Why should access control happen before retrieval?
3. How can AI traces become sensitive data stores?
4. Why is data minimization important for both privacy and quality?
5. What is the difference between authentication and authorization?
6. Why are embeddings and vector indexes part of privacy design?
7. What makes deletion difficult in AI systems?
8. Why should production traces be reviewed before becoming eval cases?
9. What should a memory policy define?
10. Why is "keep everything forever" risky?

## Summary

Privacy in AI engineering is the discipline of controlling sensitive data across the full AI pipeline: ingestion, retrieval, context assembly, model calls, outputs, tools, memory, traces, evals, analytics, and deletion.

The practical loop is:

```text
classify -> minimize -> authorize -> redact -> use -> log carefully -> retain briefly -> delete reliably -> audit
```

The model should not receive data it does not need or data the user is not allowed to access. Traces, embeddings, evals, memory, and caches must be treated as real data stores. Retention and deletion must include derived data, not only source records.

Privacy is not the enemy of useful AI. It is how useful AI earns trust.

## Preview of Next Chapter

This chapter protected sensitive data.

Chapter 45 moves from data protection to broader harm prevention: responsible AI, red teaming, and risk management.

You will learn how to identify who can be harmed, how failures should be tested, how red teams expose weaknesses before users do, and how risk reviews become repeatable engineering practice instead of vague policy language.
