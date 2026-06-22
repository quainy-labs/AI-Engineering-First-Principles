# Chapter 9: Data, Documents, and State

Part 2 built the software foundation:

```text
architecture
-> APIs and workers
-> auth and storage
-> async state over time
```

Now the system needs information.

AI systems do not become useful because a model is attached. They become useful when the right information reaches the right component at the right time, under the right permissions, with the right source of truth.

This chapter begins Part 3 by separating three things learners often mix together:

```text
data
documents
state
```

This distinction is one of the foundations of serious AI engineering.

## Concept Overview

AI systems use different kinds of information for different jobs.

Structured data tells the system exact facts:

```text
customer plan
order status
invoice amount
account permissions
refund date
subscription state
```

Documents explain meaning:

```text
policies
procedures
product docs
help-center articles
release notes
internal playbooks
contracts
```

State tells the system what is happening right now:

```text
current workflow step
pending approval
last tool call
job status
conversation stage
open ticket status
user correction
```

A model may reason over all three, but it should not replace them.

The first principle:

> Do not use one information type for every problem.

Do not use a model as a database. Do not use vector search as the source of truth for exact facts. Do not use chat history as an audit log. Do not use a policy document as proof that a specific customer qualifies.

Good AI engineering starts by asking:

> What kind of information does this answer require?

## Why It Exists

Many AI systems fail because they choose the wrong source of truth.

A support teammate asks:

> Can we build an AI assistant that answers customer questions from our company knowledge?

That sounds like one feature, but it hides multiple information types.

If a customer asks:

> Can I get a refund?

The answer may require:

- structured data: purchase date, plan type, region, refund history;
- documents: refund policy, exception policy, customer-facing explanation;
- state: current ticket status, whether manager approval is pending;
- permissions: whether the support agent can view billing data;
- logs or audit records: whether a refund was already processed.

A weak system does this:

```text
put all documents in vector database
-> retrieve similar text
-> ask model to answer
```

This may retrieve the refund policy:

> Refunds are available within 30 days.

But the database may say the customer purchased 45 days ago. The ticket state may say a manager already approved an exception. The audit log may show that a refund was already issued.

The model did not fail alone. The information design failed.

This chapter exists so learners stop thinking:

> We need data for the model.

And start thinking:

> We need a source map for the system.

## Mental Model

Think of the AI system as a decision room with three tables.

One table contains structured facts. These are precise and queryable.

One table contains documents. These explain rules, context, and human meaning.

One table contains state. This tells everyone what is currently happening in the workflow.

The model is not the table. The model is the reasoning layer that reads from the right tables when the system permits it.

The mental model:

```text
question
-> identify information need
-> choose source type
-> fetch from source of truth
-> apply permissions
-> combine evidence
-> generate or act
-> update state if workflow changed
```

A good system can explain:

- which facts came from a database;
- which guidance came from a document;
- which status came from workflow state;
- which permission allowed access;
- which parts are uncertain;
- which source should win when sources conflict.

The model can help interpret information, but source authority belongs to the system.

## Internal Mechanics

Start every AI feature by mapping information needs.

Ask:

```text
What must the system know?
Where does that knowledge live?
How fresh must it be?
Who owns it?
Who can access it?
Can it change?
What happens if it is missing?
What source wins during conflict?
```

Structured data is best for exact, current facts.

Use structured data when the answer depends on:

- counts;
- dates;
- prices;
- account status;
- permissions;
- transactions;
- inventory;
- ownership;
- workflow status;
- deterministic checks.

Examples:

```text
Is this customer on the enterprise plan?
Was the invoice paid?
How many open tickets does this account have?
Did the user approve this action?
Is this document indexed?
```

Structured data should usually be accessed through:

- database queries;
- internal APIs;
- typed service methods;
- permission-aware backend functions.

Documents are best for meaning, policy, explanation, and language.

Use documents when the answer depends on:

- policy interpretation;
- procedural steps;
- product behavior;
- examples;
- definitions;
- troubleshooting guidance;
- contractual language;
- long-form human knowledge.

Examples:

```text
What does the refund policy say?
How should support explain this limitation?
What steps should an admin follow?
Which troubleshooting path matches this symptom?
```

Documents should carry metadata:

```text
document_id
title
source
owner
version
published_at
last_updated_at
authority_level
workspace_id
permission labels
```

State is best for ongoing work.

Use state when the system needs to continue a workflow:

- current step;
- pending action;
- previous decision;
- current draft;
- job status;
- approval status;
- retry count;
- last tool result;
- user correction;
- cancellation or expiry.

Examples:

```text
Is this action awaiting approval?
Did the worker already run?
What was the last tool result?
Has the user corrected this extraction?
Can this workflow transition to completed?
```

State should be explicit. It should not hide only inside the prompt.

Common state stores:

- database row;
- job table;
- workflow table;
- event log;
- task record;
- conversation state object;
- audit log for irreversible actions.

Logs and audit records are related to state but not identical.

Logs help debugging and observability. Audit records prove that something happened.

Use audit records for:

- approvals;
- tool actions;
- external system changes;
- data exports;
- permission changes;
- deletion requests;
- user-visible decisions.

Do not use chat history as the only proof that an action was approved. Store approval as structured state or an audit event.

Conflict rules matter.

AI systems often see conflicting information:

```text
old help article says refunds allowed for 60 days
new policy says refunds allowed for 30 days
database says this customer has an exception
ticket state says manager approval is pending
```

The system needs source authority:

```text
current database fact beats stale document
newer policy beats older policy
explicit account exception beats generic policy
approved workflow state beats draft model suggestion
```

Without authority rules, the model may choose whichever text sounds most relevant.

## Real-World Example

Imagine a support refund assistant.

User goal:

> Help support agents answer refund questions accurately and draft the next action.

The user asks:

> Can this customer get a refund, and what should I tell them?

The system should not answer from documents alone.

Information map:

```text
Structured data:
- purchase date
- plan type
- payment status
- region
- refund history
- account flags

Documents:
- refund policy
- exception policy
- support tone guide
- regional compliance note

State:
- current ticket status
- pending manager approval
- previous refund request
- drafted response
- last tool action

Audit:
- previous approvals
- refund already issued
- agent who approved exception
```

Strong flow:

```text
authenticate support agent
-> check permission for billing data
-> query purchase and payment facts
-> retrieve current refund policy
-> check ticket state and approval history
-> combine evidence
-> draft answer
-> if refund requires approval, update workflow state
```

Possible output:

```text
The customer is outside the standard 30-day refund window, but a manager exception is pending. Do not promise the refund yet. Tell the customer that the request is under review and you will follow up after approval.
```

Notice what happened:

- database gave exact purchase timing;
- document gave policy;
- state gave pending approval;
- audit prevented duplicate refund;
- model drafted a human response.

Each information type did its own job.

## Common Mistakes and Failure Modes

Mistake 1: Treating vector search as the source of truth

Vector search helps find relevant text. It does not prove that a fact is current, authorized, complete, or exact.

Mistake 2: Using the model as a database

Models may remember general patterns, but they should not be trusted for current customer-specific facts, account status, prices, permissions, or transactions.

Mistake 3: Letting stale documents override live data

A document may describe a policy, but live data may show whether a specific customer qualifies.

Mistake 4: Storing workflow state only in chat history

Chat history is useful context, but important state should be durable, queryable, and auditable.

Mistake 5: No source authority order

When sources conflict, the system needs rules. Otherwise the model may select the most fluent source, not the correct one.

Mistake 6: Missing freshness requirements

Some facts must be live. Some documents can refresh daily. Some reports may be acceptable if they are a week old. Freshness should be explicit.

Mistake 7: Ignoring permissions by source type

The user may be allowed to read help articles but not billing data, HR documents, logs, or customer attachments.

Mistake 8: Treating unknown information as answerable

If the system does not have the needed source, it should say what is missing or route the workflow, not invent a confident answer.

## System Design Decisions

For every important user question, classify the required information.

Use this decision frame:

| Question type | Best source | Why |
|---|---|---|
| Exact current fact | Database or API | Deterministic and fresh |
| Policy or explanation | Document | Human-authored meaning |
| Current workflow status | State store | Tracks process truth |
| Proof of action | Audit log | Verifiable history |
| Similar examples | Search index | Finds related cases |
| Ambiguous language | Model | Interprets or generates |

Decide source authority:

```text
Which source wins for exact facts?
Which source wins for policy?
Which source wins for workflow status?
Which source wins if document and database disagree?
Which source can the model cite?
```

Decide freshness:

```text
Must this be live?
Can it be cached?
How often is it refreshed?
What timestamp should be shown?
What happens when data is stale?
```

Decide access:

```text
Who can read this source?
Can it enter model context?
Can it appear in logs?
Can it be used for training or evals?
Can it be exported?
```

Decide missing-information behavior:

```text
Should the system ask a follow-up question?
Should it query another source?
Should it escalate to a human?
Should it refuse to answer?
Should it answer with uncertainty?
```

The goal is not to gather every possible source. The goal is to route each information need to the correct source.

## Build Artifact

Create `information-map.md` for one AI workflow.

Choose a workflow such as:

- support refund assistant;
- HR policy assistant;
- document knowledge assistant;
- invoice extraction workflow;
- operations action assistant;
- multimodal document analyst.

Use this template:

```text
Information Map

Workflow:
User goal:
Main user questions:

Structured Data:
- item:
  source:
  owner:
  freshness requirement:
  permission rule:
  access method:
  used for:

Documents:
- item:
  source:
  owner:
  authority level:
  freshness requirement:
  permission rule:
  used for:

State:
- item:
  storage:
  owner:
  lifespan:
  reset rule:
  used for:

Audit Records:
- item:
  source:
  retention:
  used for:

Conflict Rules:
- conflict:
  winning source:
  reason:

Missing Information:
- missing item:
  consequence:
  system behavior:

Model Role:
- interpretation:
- generation:
- classification:
- should not decide:
```

Example:

```text
Workflow: Support refund assistant
User goal: Determine refund eligibility and draft response

Structured Data:
- purchase date
  source: billing database
  freshness: live
  permission: billing_read
  used for: eligibility check

Documents:
- refund policy
  source: policy knowledge base
  authority: current published policy
  freshness: daily sync
  used for: explanation and customer-facing language

State:
- manager approval status
  storage: ticket workflow table
  lifespan: until ticket closed
  used for: next action

Conflict Rule:
- generic policy says no refund, account exception says approved
  winning source: account exception
  reason: specific approved exception beats generic policy

Model Role:
- draft response in support tone
- summarize evidence
- should not decide billing eligibility without database check
```

The artifact is complete when another engineer can see where every important piece of knowledge comes from and which source should be trusted for each decision.

## Active Recall Questions

1. Why should a model not be used as a database?
2. What is the difference between structured data, documents, and state?
3. When should a system use a database instead of retrieval?
4. When should a system use documents instead of structured data?
5. Why is chat history not enough for durable workflow state?
6. What should happen when a document conflicts with a live database fact?
7. Why does source freshness matter?
8. What should the system do when required information is missing?

## Summary

AI engineering begins with information design.

Structured data gives exact facts. Documents give meaning and policy. State gives workflow truth over time. Audit records prove important actions. Search helps find related knowledge. Models interpret, transform, classify, and generate, but they should not replace the source of truth.

The most important rule:

> First decide what kind of information the system needs. Then choose the correct source. Only then ask the model to reason.

## Preview of Next Chapter

Chapter 9 classified the information an AI system needs.

Chapter 10 asks how that information enters the system.

Next, you will learn ingestion, parsing, cleaning, normalization, metadata, validation, and failure handling. This matters because even the best information map fails if PDFs parse badly, tables lose structure, duplicates remain, stale documents stay indexed, or metadata disappears during ingestion.
