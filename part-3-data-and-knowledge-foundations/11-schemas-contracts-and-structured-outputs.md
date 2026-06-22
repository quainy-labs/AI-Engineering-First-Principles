# Chapter 11: Schemas, Contracts, and Structured Outputs

Chapter 10 showed how information enters the system:

```text
source -> fetch -> parse -> clean -> normalize -> metadata -> validate -> store -> index
```

This chapter asks how information moves between components without becoming ambiguous, broken, or unsafe.

AI systems need structure because model outputs are probabilistic, source data is messy, and downstream software needs clear fields. If a component expects a date, enum, document ID, tool argument, or confidence flag, free-form text is not enough.

The job of this chapter is to turn vague information into reliable contracts.

## Concept Overview

A schema defines the shape of information.

A contract defines the expectation between two components.

A structured output is information produced in a machine-readable format, often by a model, parser, extractor, or tool.

Together, they answer:

```text
What fields exist?
What does each field mean?
Which fields are required?
What values are allowed?
Who produces this object?
Who consumes it?
How is it validated?
What happens when it is invalid?
```

Contracts exist across the whole AI system:

- UI to API;
- API to service;
- service to worker;
- worker to database;
- ingestion pipeline to index;
- retriever to context builder;
- model to validator;
- tool caller to tool executor;
- eval dataset to eval runner;
- trace logger to observability system.

The first principle:

> Treat model output as untrusted input until it passes a contract.

Free text is fine when a human is the only consumer. Structure is required when software must act on the output.

## Why It Exists

Software cannot safely depend on vibes.

Suppose you ask a model:

> Extract customer name, issue type, urgency, and requested action from this support ticket.

The model replies:

```text
The customer seems upset and probably wants a refund quickly.
```

This may be useful to a human. It is not enough for software.

Software needs something like:

```json
{
  "customer_name": "Ravi Kumar",
  "issue_type": "refund_request",
  "urgency": "high",
  "requested_action": "refund_review",
  "needs_human_review": true
}
```

Even this is not enough unless the fields are defined:

```text
issue_type must be one of: refund_request, login_problem, billing_question, technical_bug, other
urgency must be one of: low, medium, high
needs_human_review must be true when policy exception or low confidence exists
```

Without a contract, systems fail quietly:

- model returns valid JSON with wrong field meaning;
- enum values overlap;
- required fields are inferred from nowhere;
- downstream code trusts hallucinated facts;
- a tool receives unsafe arguments;
- eval results cannot be compared;
- traces miss important fields;
- worker jobs cannot be retried safely.

This chapter exists because reliable AI systems do not only prompt models. They define and enforce interfaces.

## Mental Model

Think of every component as a worker at a handoff point.

One component gives an object to another. If the object is vague, the next component guesses. If the object has a clear contract, the next component can validate it, reject it, retry it, or route it to a human.

The mental model:

```text
producer
-> schema
-> validation
-> accepted object
-> consumer
```

For model output:

```text
prompt
-> model generates structured object
-> validator checks shape and rules
-> business logic checks meaning
-> system accepts, retries, or escalates
```

The schema is not the whole truth. It only defines shape. Validation and business logic must still decide whether the object is correct enough to use.

Important distinction:

```text
Schema validity:
Does the object have the right shape?

Semantic validity:
Does the object make sense?

Business validity:
Is the system allowed to act on it?
```

Example:

```json
{
  "refund_amount": 5000,
  "currency": "USD",
  "approved": true
}
```

This may be schema-valid. It may still be semantically wrong or business-invalid if the customer paid only 50 USD or the user lacks approval permission.

## Internal Mechanics

Start by deciding which information must become structured.

Use schemas for:

- ingestion records;
- document metadata;
- extracted fields;
- parsed tables;
- workflow state;
- worker job payloads;
- model outputs;
- tool arguments;
- tool results;
- eval examples;
- eval results;
- trace events;
- user feedback records.

Use free text for:

- final human-facing explanation;
- brainstorming output;
- long-form summaries;
- creative writing;
- cases where no downstream system acts on the content.

### Schema Fields

A useful schema defines more than field names.

For each field, specify:

```text
name
type
meaning
required or optional
allowed values
source of value
validation rule
default or fallback
confidence if model-produced
human-review rule
```

Example:

```text
field: urgency
type: enum
allowed values: low, medium, high
meaning: how quickly the support team should respond
source: support ticket text
validation: must be one of allowed values
human-review rule: review if urgency=high and confidence < 0.75
```

### Contracts Between Components

A contract should name both producer and consumer.

Example:

```text
Contract: ParsedDocument
Producer: ingestion parser
Consumer: chunker and indexer
```

This makes accountability clear. If the indexer needs `source_uri`, the parser contract must require it.

Example fields:

```text
document_id
source_uri
title
content
sections
tables
owner
permission_labels
parser_version
parse_warnings
```

### Structured Model Outputs

Structured model output should be constrained and validated.

Bad prompt:

```text
Return JSON with the extracted ticket fields.
```

Better contract:

```text
Extract only facts present in the ticket.
Do not infer account status, billing eligibility, or refund approval.
If a required field is missing, set it to null and explain missing evidence.
Use only allowed enum values.
Return an object matching TicketExtraction.
```

Example schema:

```json
{
  "customer_name": "string or null",
  "issue_type": "refund_request | login_problem | billing_question | technical_bug | other",
  "urgency": "low | medium | high",
  "requested_action": "string or null",
  "evidence": ["string"],
  "missing_fields": ["string"],
  "confidence": 0.0,
  "needs_human_review": true
}
```

The validator should check:

- valid JSON or object shape;
- required fields;
- allowed enum values;
- field types;
- length limits;
- evidence exists for extracted claims;
- confidence range;
- safety or permission constraints when needed.

### Extraction Versus Decision

Do not mix extraction and decision too early.

Extraction answers:

```text
What is present in the input?
```

Decision answers:

```text
What should the system do?
```

Example:

```text
Extraction:
customer asks for refund
purchase date mentioned: no
reason: product did not work

Decision:
requires database lookup for purchase date
requires refund policy
requires agent approval if outside policy
```

If a model decides refund eligibility only from ticket text, it may invent missing facts. Let the model extract. Let deterministic code, database facts, policy retrieval, and approval state participate in the decision.

### Validation and Repair

Invalid structured output needs a path.

Options:

```text
reject and retry
ask model to repair against validation error
route to human review
mark field unknown
fallback to safer behavior
fail the workflow
```

Do not silently coerce dangerous outputs.

Example:

```text
model output: "very urgent"
schema enum: low, medium, high
bad fix: map anything emotional to high
better fix: validation failure or explicit classifier rule
```

### Versioning

Schemas change over time.

Version important contracts:

```text
TicketExtraction.v1
TicketExtraction.v2
ParsedDocument.v3
ToolCall.v1
EvalExample.v2
TraceEvent.v1
```

Versioning matters because old records, evals, traces, and model outputs may still exist. Chapter 12 will go deeper into quality, lineage, and versioning.

## Real-World Example

Imagine a support ticket triage system.

User goal:

> Automatically classify incoming tickets and prepare them for routing.

Input ticket:

```text
Hi, I was charged twice after upgrading my plan. I need someone to fix this today.
```

Weak output:

```text
This is a billing issue and sounds urgent.
```

Better structured output:

```json
{
  "issue_type": "billing_question",
  "urgency": "high",
  "requested_action": "investigate_duplicate_charge",
  "customer_sentiment": "frustrated",
  "evidence": [
    "charged twice",
    "fix this today"
  ],
  "missing_fields": [
    "account_id",
    "invoice_id"
  ],
  "needs_human_review": true
}
```

Contract:

```text
Producer: ticket extraction model
Consumer: routing service

Required fields:
- issue_type
- urgency
- requested_action
- evidence
- missing_fields
- needs_human_review

Rules:
- issue_type must be an allowed enum
- urgency must be low, medium, or high
- evidence must quote or summarize input content
- model must not decide refund approval
- missing account_id requires lookup or human review
```

Then routing service uses other sources:

```text
structured output says billing issue
-> database lookup finds account
-> payment API checks duplicate charge
-> policy document explains refund handling
-> workflow state decides whether approval is required
```

The model contributed interpretation. The contract prevented the system from pretending interpretation was a complete decision.

## Common Mistakes and Failure Modes

Mistake 1: Asking for JSON without a real schema

JSON is syntax. A schema defines meaning, allowed values, required fields, and invalid states.

Mistake 2: Trusting valid structure as truth

An object can have the right shape and still contain wrong facts. Validate shape, then verify meaning where the stakes require it.

Mistake 3: Vague enum labels

Labels like `important`, `urgent`, and `critical` overlap unless the system defines them clearly.

Mistake 4: Schema requires unavailable information

If the input does not contain purchase date, the model should not invent it. The schema should allow `null`, `unknown`, or `missing_fields` when appropriate.

Mistake 5: Mixing extraction with business decision

The model may extract a refund request. It should not approve a refund without database facts, policy checks, permissions, and workflow state.

Mistake 6: No invalid-output path

If validation fails, the system needs a decision: retry, repair, fallback, human review, or fail safely.

Mistake 7: Downstream code trusts model output directly

Model output should pass through validation and business rules before it triggers tools, state changes, or user-visible decisions.

Mistake 8: Unversioned contracts

When schemas change silently, old outputs, evals, traces, and stored records become hard to compare or debug.

## System Design Decisions

Decide where structure is necessary.

Use this rule:

```text
If software depends on it, structure it.
If humans only read it, free text may be acceptable.
```

Define each contract:

```text
Contract name:
Producer:
Consumer:
Purpose:
Version:
```

Define fields:

```text
Field:
Type:
Meaning:
Required:
Allowed values:
Source:
Validation:
Fallback:
Human-review rule:
```

Define validation:

```text
What makes the object shape-valid?
What makes it semantically suspicious?
What makes it business-invalid?
What happens on failure?
```

Define model boundaries:

```text
What can the model infer?
What must come from a database or API?
What must come from retrieved policy?
What requires human approval?
What should never be generated?
```

Define versioning:

```text
How is the schema version recorded?
Can old records still be read?
How are eval examples migrated?
How are traces compared across versions?
```

Good contracts make later chapters possible: quality reports, lineage, retrieval records, context assembly, tool calls, eval datasets, observability traces, and governance reviews all depend on structured information.

## Build Artifact

Create `schemas.md` for one AI workflow.

Choose a workflow such as:

- support ticket triage;
- document ingestion;
- invoice extraction;
- HR policy assistant;
- operations action assistant;
- evaluation runner.

Use this template:

```text
Schemas and Contracts

Workflow:
User goal:

Contract:
- name:
  version:
  producer:
  consumer:
  purpose:

Fields:
- name:
  type:
  meaning:
  required:
  allowed values:
  source:
  validation:
  fallback:
  human-review rule:

Validation:
- shape checks:
- semantic checks:
- business checks:
- invalid-output behavior:

Model Boundaries:
- model may:
- model must not:
- must come from database/API:
- must come from documents:
- requires human approval:

Invalid Examples:
- input:
  bad output:
  why invalid:
  correct behavior:

Versioning:
- current version:
- stored with:
- migration rule:
```

Example:

```text
Workflow: Support ticket triage

Contract:
name: TicketExtraction
version: v1
producer: ticket extraction model
consumer: routing service
purpose: classify ticket and identify missing information

Field:
name: issue_type
type: enum
required: yes
allowed values: refund_request, login_problem, billing_question, technical_bug, other
source: ticket text
validation: must be allowed value
fallback: other

Field:
name: needs_human_review
type: boolean
required: yes
source: extraction rules
validation: true when confidence low, missing required info, or policy exception suspected

Model must not:
- approve refund
- invent account status
- call billing tool
```

The artifact is complete when another engineer can implement validation and know exactly what the model is allowed to produce.

## Active Recall Questions

1. What is the difference between a schema, a contract, and a structured output?
2. Why is valid JSON not the same as a reliable contract?
3. Why should model output be treated as untrusted input?
4. What is the difference between schema validity, semantic validity, and business validity?
5. Why should extraction and decision often be separated?
6. What should happen when structured output fails validation?
7. When is free text acceptable?
8. Why should important schemas be versioned?

## Summary

Schemas make information shape explicit. Contracts define what one component promises another. Structured outputs let probabilistic systems communicate with deterministic software, but only after validation and business rules.

Use structure whenever downstream software depends on the output. Treat model output as untrusted input. Separate extraction from decisions. Version important contracts so behavior can be debugged over time.

The most important rule:

> If software acts on it, give it a contract.

## Preview of Next Chapter

Chapter 11 defined the shape of information moving through the system.

Chapter 12 asks whether that information remains reliable over time.

Next, you will learn data quality, lineage, and versioning. This matters because schemas alone do not tell you whether a document was stale, which parser created a record, which embedding model built an index, or why an answer changed overnight.
