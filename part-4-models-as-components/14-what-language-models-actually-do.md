# Chapter 14: What Language Models Actually Do

Part 3 built the information foundation:

```text
data, documents, and state
-> ingestion and parsing
-> schemas and contracts
-> quality, lineage, and versioning
-> search and ranking
```

Now the system has usable information.

Part 4 asks what the model should do with that information.

A language model is not the whole AI system. It is one component inside the system. It can understand language, transform text, extract structure, draft responses, classify intent, and reason over supplied context. But it should not become the database, permission system, policy engine, audit log, or final authority for high-impact actions.

This chapter gives you the boundary.

## Concept Overview

A language model maps input tokens to output tokens.

That sounds simple, but modern language models can do powerful work because they have learned patterns across language, code, documents, examples, instructions, and reasoning traces.

They are useful for:

- understanding messy human language;
- summarizing long text;
- classifying intent;
- extracting fields;
- rewriting and formatting;
- drafting human-facing responses;
- comparing pieces of text;
- translating between formats;
- reasoning over supplied context;
- producing structured outputs;
- choosing from allowed actions when bounded.

They are weak or risky as:

- source of truth;
- database;
- permission system;
- policy authority;
- calculator under strict correctness needs;
- audit log;
- state machine;
- hidden decision-maker;
- owner of irreversible actions.

The first principle:

> A model should transform and reason over context. The system should decide what context is trusted, what actions are allowed, and what must be verified.

## Why It Exists

Teams often give models work that actually belongs to the system.

Example request:

> Read this customer email, decide if they are eligible for a refund, and write the reply.

That sounds like one task. It is actually several tasks:

```text
understand customer request
-> extract issue and evidence
-> check customer account facts
-> retrieve refund policy
-> apply business rules
-> check permissions
-> decide whether approval is needed
-> draft reply
-> maybe send after review
```

The model can help with some of these:

- understand the email;
- classify the issue;
- extract requested action;
- identify missing information;
- draft a reply from supplied facts and policy.

The model should not own others:

- check live account truth;
- approve refund alone;
- enforce permissions;
- decide current policy without retrieval;
- send final message without workflow approval;
- invent missing facts.

Bad design:

```text
You are a support agent.
Read the email, decide refund eligibility, and write the final reply.
```

This creates failures:

- model invents account status;
- model uses generic policy instead of current policy;
- model makes a business decision without permissions;
- model sounds confident while missing evidence;
- retry produces a different decision;
- system cannot explain who owned the decision.

This chapter exists because AI engineering is not outsourcing system design to the model. It is deciding where the model fits.

## Mental Model

Think of the model as a very capable analyst inside a controlled workflow.

The analyst can read messy notes, summarize them, extract important details, compare evidence, draft language, and suggest next steps. But the analyst should not secretly access every database, approve payments, override permissions, or change official records without process.

The mental model:

```text
system owns boundaries
-> system supplies trusted context
-> model performs bounded language task
-> system validates output
-> system decides next action
```

The model is strongest when the system gives it:

- clear task;
- trusted context;
- explicit output contract;
- examples when useful;
- constraints on what not to decide;
- validation after generation;
- fallback or human review path.

The model is weakest when the system asks it:

- to know private facts not in context;
- to guess missing state;
- to enforce policy from memory;
- to choose permissions;
- to make irreversible decisions without checks;
- to be consistent across every edge case without evaluation.

The boundary is the product.

If the boundary is unclear, the model becomes responsible for too much and the system becomes hard to trust.

## Internal Mechanics

A language model receives context and generates output.

At a practical engineering level, that means:

```text
instructions
user input
retrieved context
tool results
examples
state
output contract
```

go in, and:

```text
text
structured object
tool call proposal
classification
summary
draft
```

comes out.

The model output is useful, but it is not automatically true.

Use the model for language-shaped work.

Strong model-owned tasks:

```text
Summarize this ticket.
Extract requested action from this email.
Classify the issue type from allowed labels.
Rewrite this policy explanation for a customer.
Compare these two policy snippets.
Draft a response using only supplied facts.
Identify missing fields in this document.
Convert unstructured notes into a schema.
```

Use deterministic code for exact rules.

Code-owned tasks:

```text
Is purchase date within 30 days?
Does this user have billing_read permission?
Is refund amount below approval threshold?
Is this job allowed to transition from pending to completed?
Is this JSON schema-valid?
Has this webhook event already been processed?
```

Use databases and APIs for current facts.

Data-owned tasks:

```text
What plan is the customer on?
Was invoice 123 paid?
Is this document indexed?
Who owns this file?
What is the latest ticket status?
Was approval recorded?
```

Use documents for policy and explanation.

Document-owned tasks:

```text
What does the refund policy say?
How should API key rotation be explained?
What are the troubleshooting steps?
Which exception rules apply?
```

Use humans for judgment, accountability, and high-impact approval.

Human-owned tasks:

```text
approve exception
send sensitive final response
override policy
handle unclear edge case
accept risk
review low-confidence extraction
```

The model boundary should be explicit.

For every model use, define:

```text
Task:
Input:
Trusted context:
Allowed output:
Not allowed:
Validation:
Fallback:
Human review:
Owner:
```

Example:

```text
Task: Draft support reply
Input: ticket text, account facts, retrieved refund policy, approval state
Trusted context: billing API result, current policy document, ticket workflow state
Allowed output: draft reply with cited evidence
Not allowed: approve refund, invent account facts, send final response
Validation: citations must map to supplied context
Fallback: needs_human_review
Human review: required before sending
Owner: support workflow service
```

This makes the model a bounded component.

## Real-World Example

Imagine a support refund assistant.

Customer email:

```text
I upgraded last month and was charged twice. I want a refund today.
```

Weak design:

```text
Model decides refund eligibility and sends reply.
```

Better design splits the work:

```text
Model extracts:
- customer asks for refund
- possible duplicate charge
- urgency is high
- missing invoice ID

Database/API checks:
- account plan
- invoice history
- duplicate charge status
- purchase date

Document retrieval provides:
- refund policy
- duplicate charge handling policy
- customer-facing explanation rules

Workflow state provides:
- whether approval is pending
- whether refund was already issued

Code decides:
- whether rule conditions are met
- whether approval is required
- whether action can proceed

Model drafts:
- response using supplied facts and policy
- clear next step
- no invented account facts
```

Good model task:

```text
Draft a customer response using only these facts:
- duplicate charge is under review
- refund cannot be promised until billing confirms
- support will follow up within 24 hours

Use the support tone guide.
Do not say the refund is approved.
```

Good output:

```text
Thanks for flagging this. We are reviewing the duplicate charge on your account and will follow up within 24 hours. I do not want to promise a refund until billing confirms the charge status, but we are treating this as urgent.
```

The model improves communication. The system owns truth and action.

## Common Mistakes and Failure Modes

Mistake 1: Asking the model to know private facts

The model does not know live account status unless the system retrieves it and supplies it as context.

Mistake 2: Treating fluent output as verified truth

A well-written answer can still be unsupported, stale, unauthorized, or wrong.

Mistake 3: Combining extraction, decision, and action into one prompt

Large mixed tasks hide failure. Split the workflow into extraction, retrieval, deterministic checks, draft, validation, and approval.

Mistake 4: Letting the model enforce permissions

Permissions must be enforced before context reaches the model and before tools execute.

Mistake 5: No validation after generation

Model output should be checked for schema, citations, forbidden claims, missing evidence, and unsafe actions.

Mistake 6: No fallback path

When context is missing or confidence is low, the system needs a safe response: ask for more information, retrieve another source, escalate, or route to human review.

Mistake 7: Draft becomes final action

A model draft is not the same as approval to send, refund, delete, update, or execute.

Mistake 8: Evaluating only final text

Evaluate whether the model stayed inside its boundary: did it extract correctly, avoid unsupported claims, ask for missing facts, and route uncertainty?

## System Design Decisions

For every model use, decide what the model owns and what it does not own.

Define the model task:

```text
Is this classification, extraction, summarization, transformation, drafting, comparison, or reasoning over context?
```

Define required context:

```text
What facts must come from database/API?
What policy must come from documents?
What state must come from workflow records?
What user input is relevant?
What should be excluded?
```

Define output contract:

```text
Free text?
Structured object?
Draft response?
Tool proposal?
Classification label?
Evidence list?
```

Define forbidden responsibilities:

```text
What should the model never decide?
What facts must it never invent?
What actions must it never take directly?
What permissions must be checked outside the model?
```

Define validation:

```text
Schema validation
citation validation
policy consistency check
unsupported-claim check
permission check
human-review trigger
```

Define fallback:

```text
ask clarification
retrieve more context
return unknown
route to human
refuse unsafe request
defer action until approval
```

The core design question:

> If the model is wrong, what part of the system catches it before harm?

## Build Artifact

Create `model-capability-boundary.md` for one AI workflow.

Choose a workflow such as:

- support refund assistant;
- HR policy assistant;
- invoice extraction workflow;
- document knowledge assistant;
- operations action assistant.

Use this template:

```text
Model Capability Boundary

Workflow:
User goal:

Model Task:
- task:
  why model is useful:
  input:
  trusted context:
  output:
  output contract:

Good Model Uses:
- use:
  reason:

Bad Model Uses:
- use:
  why not:
  correct owner:

Ownership Split:
- model owns:
- deterministic code owns:
- database/API owns:
- documents own:
- workflow state owns:
- human owns:

Forbidden Claims or Actions:
- item:
  reason:

Validation:
- check:
  failure behavior:

Fallback:
- condition:
  response:

Evaluation Cases:
- input:
  expected model role:
  should refuse:
  human review:
  forbidden behavior:
```

Example:

```text
Workflow: Support refund assistant

Model Task:
Extract refund request details and draft a response from supplied facts.

Model owns:
- summarize customer issue
- extract requested action
- draft response using supplied policy and facts

Database/API owns:
- purchase date
- payment status
- account plan

Documents own:
- current refund policy
- tone guide

Deterministic code owns:
- 30-day eligibility check
- approval threshold
- state transition

Human owns:
- policy exception approval
- final send for sensitive cases

Forbidden:
- model must not approve refund
- model must not invent purchase date
- model must not claim refund was processed unless tool result confirms it
```

The artifact is complete when another engineer can tell exactly where the model belongs and what catches it when it overreaches.

## Active Recall Questions

1. Why is a language model not a source of truth?
2. Which tasks are language models usually good at?
3. Which tasks should databases, APIs, deterministic code, or humans own instead?
4. What is a model capability boundary?
5. Why should extraction, decision, and action often be separated?
6. Why is a fluent answer not the same as a verified answer?
7. What validation should happen after model output?
8. Why should a model draft not automatically become a final action?

## Summary

Language models are powerful components for language-shaped work: understanding, summarizing, extracting, classifying, rewriting, drafting, comparing, and reasoning over supplied context.

They are not the database, permission system, policy engine, audit log, workflow state machine, or final authority for risky actions. The surrounding system must supply trusted context, enforce permissions, validate outputs, own deterministic decisions, and route uncertainty.

The most important rule:

> Bound the model's job before you ask it to be intelligent.

## Preview of Next Chapter

Chapter 14 bounded the model's role.

Chapter 15 explains the mechanics that shape that role: tokens, context, attention, and generation.

Next, you will learn why more context is not always better, why important evidence can be buried, why cost and latency grow with tokens, and why context design is a core AI engineering skill rather than prompt decoration.
