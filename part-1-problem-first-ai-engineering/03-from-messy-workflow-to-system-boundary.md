# Chapter 3: From Messy Workflow to System Boundary

Chapter 1 said AI engineering is system design.

Chapter 2 separated work that belongs to AI from work that belongs to code, data, retrieval, or humans.

This chapter turns that separation into a boundary.

> A system boundary defines what the system accepts, what it produces, what it controls, what it refuses, and who owns each decision.

Without boundary, "AI assistant" becomes a vague promise.

With boundary, it becomes an engineered system.

## Concept Overview

A messy workflow is how work happens in real life.

It includes:

- unclear inputs;
- human judgment;
- copy-paste steps;
- searching;
- checking;
- exceptions;
- approvals;
- undocumented habits.

A system boundary turns that mess into an explicit design:

```text
inside system:
- accepted inputs
- system steps
- AI tasks
- code tasks
- data sources
- output contract

outside system:
- tasks system refuses
- decisions humans keep
- external systems only referenced
- unsupported edge cases
```

The goal is not to automate everything.

The goal is to define a reliable slice of work that can be built, evaluated, and improved.

## Why It Exists

Teams often begin with pain:

```text
Our team spends too much time reviewing compliance documents.
```

That is useful, but it is not yet a system requirement.

Pain does not tell you:

- which documents;
- which user;
- which policy source;
- which risk categories;
- what output should be produced;
- who approves final decision;
- what should happen when evidence is missing;
- what system must refuse.

Without boundary, teams build demos:

```text
upload document -> ask model for issues -> show summary
```

The demo may look good.

Then real work arrives:

- scanned PDFs;
- missing pages;
- outdated policy;
- ambiguous clause;
- high-risk legal decision;
- user asks for final approval;
- citation is required;
- audit trail is needed.

Every edge case becomes surprise because the system never defined what it owns.

System boundary exists to prevent surprise.

## Mental Model

Start with workflow:

```text
trigger -> input -> steps -> decisions -> output
```

Then draw boundary:

```text
                 outside system
        --------------------------------
        policy owner, legal approval,
        unsupported document types,
        final high-risk decision

user -> input -> [ system boundary ] -> output -> human/user
                 accepted docs
                 extraction
                 retrieval
                 model analysis
                 validation
                 review flag
                 audit record
```

Inside the boundary, the system must be designed and evaluated.

Outside the boundary, the system must hand off, refuse, or escalate.

Boundary answers:

```text
What do we accept?
What do we produce?
What do we decide?
What do we not decide?
What do we need from outside?
Who owns final accountability?
```

## Internal Mechanics

To convert workflow into boundary, inspect five layers.

### 1. Trigger

Trigger starts the work.

Examples:

- user uploads document;
- ticket arrives;
- customer sends email;
- scheduled job runs;
- employee asks question.

If trigger is vague, system entry point is vague.

### 2. Input

Input is what system receives.

Define:

- format;
- source;
- required fields;
- optional fields;
- quality limits;
- unsupported inputs.

Example:

```text
Accepted: vendor contract PDF under 30 pages
Unsupported: handwritten notes, images without readable text, legal filings
```

### 3. Transformation

Transformation is what changes inside the system.

Examples:

- extract fields;
- classify type;
- retrieve policy;
- identify risky clause;
- draft summary;
- validate output.

Each transformation needs owner:

- code;
- retrieval;
- model;
- human;
- external service.

### 4. Decision

Decision is where something is judged.

Examples:

- is document in scope?
- is clause risky?
- is evidence enough?
- should this be escalated?
- can output be trusted?

Some decisions can be code. Some can be model-assisted. Some must be human.

### 5. Output

Output is what leaves the system.

Examples:

- structured record;
- draft reply;
- cited answer;
- risk report;
- task proposal;
- refusal message.

Output needs contract:

```text
format:
required fields:
citations:
confidence/review flag:
owner:
```

If output is vague, evaluation becomes vague.

## Real-World Example

A compliance team says:

```text
We spend too much time reviewing vendor contracts.
```

Messy workflow today:

1. Vendor sends contract PDF.
2. Operations teammate uploads it to shared folder.
3. Reviewer searches policy doc.
4. Reviewer scans contract for risky clauses.
5. Reviewer writes summary.
6. Legal reviews high-risk cases.
7. Decision is stored in spreadsheet.

Bad AI request:

```text
Build an AI contract reviewer.
```

Better boundary:

```text
System accepts:
- vendor contract PDFs under 30 pages
- text-based PDF or OCR-readable scan

System does:
- detect contract type
- extract parties, dates, renewal terms
- retrieve approved policy
- identify possible risky clauses
- cite contract clause and policy source
- produce review draft

System does not:
- give legal approval
- negotiate contract
- approve vendor
- review non-contract documents
- decide high-risk cases automatically

Human owns:
- final risk rating
- legal approval
- exception handling
```

Now the system is buildable.

You can test whether it accepts the right inputs, produces useful output, cites sources, and escalates risky cases.

## Common Mistakes and Failure Modes

### Mistake 1: Boundary Is Just "Chat With Documents"

Chat is interface, not boundary.

Boundary must define accepted input, output, ownership, refusal, and handoff.

### Mistake 2: System Accepts Everything

If system accepts every document, every question, and every workflow, it cannot be reliable.

Start narrow.

### Mistake 3: Human Approval Is Added Too Late

Approval should be designed before failure.

If legal, financial, access, or customer-impacting decisions exist, define approval points early.

### Mistake 4: Output Has No Contract

Bad:

```text
Give useful summary.
```

Better:

```text
Return risk_summary, cited_clauses, policy_sources, missing_information, needs_legal_review.
```

### Mistake 5: Hidden Model Decisions

If model decides something important, name it.

Do not hide decisions inside "analysis."

### Mistake 6: No Out-of-Scope List

Out of scope protects trust.

It tells system when to refuse or escalate instead of pretending.

## System Design Decisions

For a workflow, decide:

```text
What starts the workflow?
Who is the user?
What input is accepted?
What input is rejected?
What output is promised?
What output is not promised?
Which steps belong to code?
Which steps belong to retrieval?
Which steps belong to model?
Which decisions belong to human?
What external systems are required?
What failure modes matter?
What gets logged?
```

Ownership table:

| Step | Owner | Reason |
|---|---|---|
| Upload contract | user/system | input |
| Detect document type | model + validation | messy but checkable |
| Extract dates/parties | model + schema | language/layout extraction |
| Retrieve policy | code + search | trusted source |
| Identify possible risky clauses | model + retrieval | language judgment with evidence |
| Final legal risk rating | human | accountability |
| Store review record | code | audit trail |

The best boundary usually makes the first version smaller than the original ambition.

Smaller reliable system beats broad vague assistant.

## Build Artifact

Create `workflow-boundary.md`.

Template:

```text
Workflow:
Primary user:

Trigger:
Accepted inputs:
Rejected inputs:

Output:
Output contract:

Steps:
- step:
  owner: code | retrieval | model | human | external service
  reason:

Human approval points:
External systems:
Data sources:
Out of scope:
Failure modes:
Logs/audit trail:

Boundary test cases:
- normal:
- edge:
- out of scope:
```

Worked example:

```text
Workflow: vendor contract review draft
Primary user: compliance reviewer

Trigger: vendor contract uploaded
Accepted inputs: vendor contract PDF under 30 pages
Rejected inputs: handwritten scans, non-contract documents, missing pages

Output: review draft
Output contract:
- contract_type
- extracted_terms
- risky_clauses
- cited_policy_sources
- missing_information
- needs_legal_review

Steps:
- step: classify document type
  owner: model + validation
  reason: language/layout variation

- step: retrieve policy
  owner: code + search
  reason: must use approved source

- step: identify risky clauses
  owner: model + retrieval
  reason: language judgment, must cite evidence

- step: final risk approval
  owner: human
  reason: legal accountability

Human approval points:
- any high-risk clause
- missing policy evidence
- contract value above threshold

External systems:
- document store
- policy search index
- review database

Out of scope:
- final legal approval
- vendor negotiation
- non-contract document review

Failure modes:
- wrong policy source
- missing clause
- unsupported claim
- out-of-scope document accepted

Logs/audit trail:
- uploaded document id
- policy sources used
- model output version
- reviewer decision

Boundary test cases:
- normal: standard vendor contract
- edge: contract missing renewal section
- out of scope: handwritten amendment photo
```

Acceptance criteria:

- trigger is clear;
- accepted and rejected inputs are clear;
- output contract exists;
- every step has owner;
- human approval points are explicit;
- out-of-scope cases are named;
- boundary is tested with real examples.

## Active Recall Questions

1. What is system boundary?
2. Why does workflow come before model?
3. What does out of scope protect against?
4. Why should every workflow step have owner?
5. What is difference between workflow and system boundary?
6. Why should output have contract?
7. Why test boundary before implementation?

## Summary

A messy workflow is not yet a system.

The durable idea:

```text
workflow shows how work happens
boundary defines what system owns
```

System boundary makes AI engineering concrete:

- accepted input;
- promised output;
- step ownership;
- human approval;
- external systems;
- refusal behavior;
- failure modes;
- audit trail.

Without boundary, a powerful model becomes unreliable product.

With boundary, the system can be built, evaluated, and improved.

## Preview of Next Chapter

This chapter defined what the system will and will not do.

Next chapter asks:

> How do we measure whether that boundary creates meaningful improvement?
