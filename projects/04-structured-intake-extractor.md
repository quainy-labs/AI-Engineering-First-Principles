# Project 4: Structured Intake Extractor

## Concept

Models become useful in applications when they are given a bounded job, a clear interface, and a validator.

This project builds a structured intake extractor that converts messy inbound text into validated JSON records for downstream workflow.

The learner should finish with a runnable CLI, notebook, or local app that accepts messy messages, calls a model through a controlled interface, validates structured output, routes failures, supports human correction, and produces an error analysis report.

This is not a workflow automation project yet. It does not call external tools, update records, retrieve documents, or run agents.

The focus is model-as-component:

```text
messy text -> model interface -> structured output -> validator -> review or accepted record
```

## What You Will Build

Build a Structured Intake Extractor.

The system accepts support emails, forms, chat transcripts, or pasted messages and produces:

- validated JSON;
- extraction status;
- model route used;
- fallback reason, if any;
- validation errors;
- needs-review flag;
- human-correctable fields;
- labeled correction data;
- error analysis report;
- adaptation decision record.

The product answers one question:

```text
Can a model reliably convert messy language into a typed interface contract?
```

## Chapters Practiced

This project proves understanding from:

- Chapter 14: what language models actually do;
- Chapter 15: tokens, context, attention, and generation;
- Chapter 16: prompting as interface design;
- Chapter 17: model selection, routing, and fallbacks;
- Chapter 18: fine-tuning, adaptation, and when not to tune;
- Chapter 19: model failure modes and error analysis.

It may reuse previous parts for app structure and data handling, but it should not require future course concepts.

Do not require retrieval, embeddings, RAG, tools, agents, multimodal inputs, production observability, LLMOps, security, or governance review here.

Later parts can improve the same project:

- Part 5 can add retrieval for policy context;
- Part 6 can extract from PDFs, forms, images, and audio transcripts;
- Part 7 can route accepted records into tools and workflows;
- Part 8 can add production evals, tracing, cost, latency, and deployment;
- Part 9 can add privacy, prompt-injection tests, and governance.

## User Story

Primary user:

```text
Operations teammate, support lead, or internal workflow builder.
```

User story:

```text
As an operations teammate,
I want messy inbound messages converted into consistent intake records,
so requests can be routed, reviewed, searched, and measured without manual copying.
```

The user wants consistency:

```text
same fields
allowed values
missing information clearly marked
risky requests routed to review
bad model output rejected
```

## System Requirements

### Input

Accept one of:

- pasted text;
- `.txt` files;
- CSV rows with a `message` column;
- JSON list of messages.

Each input item should become one intake record.

### Target Schema

Define an intake schema.

Required fields:

```text
issue_type
urgency
requested_action
summary
needs_human_review
missing_information
```

Allowed `issue_type` values:

```text
billing_duplicate_charge
refund_request
api_key_reset
account_owner_change
bug_report
data_deletion
security_concern
how_to_question
unknown
```

Allowed `urgency` values:

```text
low
normal
high
critical
```

Allowed `requested_action` values:

```text
answer_question
send_instructions
refund_review
account_review
bug_triage
security_escalation
data_deletion_review
needs_clarification
```

Human review rule examples:

```text
needs_human_review = true if:
- requested_action is refund_review;
- requested_action is account_review;
- requested_action is security_escalation;
- requested_action is data_deletion_review;
- urgency is critical;
- required identifiers are missing for a risky request.
```

### Output

The system should produce:

- `extracted-records.json`;
- `validation-report.md`;
- `error-analysis.md`;
- `adaptation-decision.md`;
- optional corrected labels file.

### Interface

Choose one implementation path:

```text
CLI:
intake extract messages.csv --out extracted-records.json
intake validate extracted-records.json
intake review MSG-001
intake eval labeled-examples.json
intake errors eval-results.json

Notebook:
Run extraction, validation, review, and error analysis cells.

Local app:
Input panel, JSON output, validation errors, correction form.
```

CLI or notebook is enough for this project.

## Starter Data

Create `messages.json` or `messages.csv` with at least 40 inbound messages.

Example input:

```text
Subject: Charged twice

Hi, I upgraded to Pro yesterday but I see two charges on my card.
Can someone refund the duplicate? This is urgent because our finance
team closes books today.
```

Expected output:

```json
{
  "issue_type": "billing_duplicate_charge",
  "urgency": "high",
  "requested_action": "refund_review",
  "summary": "Customer reports duplicate Pro charge and requests urgent refund review.",
  "needs_human_review": true,
  "missing_information": ["account_id", "charge_id"]
}
```

Seed the dataset with:

```text
Easy:
- "How do I reset my API key?"
- "Where can I download invoices?"

Normal:
- duplicate charge request;
- bug report with reproduction details;
- plan limit question.

Edge:
- long email thread;
- unclear complaint;
- multiple requests in one message;
- missing important identifiers.

Should-review:
- refund request;
- account ownership change;
- security concern;
- data deletion request.

Should-refuse-or-escalate:
- asks for another user's private data;
- asks for admin action without authorization;
- asks for account deletion without confirmation.
```

## MVP Milestone

Build the simplest useful version first.

MVP must:

1. Load messages.
2. Define extraction schema.
3. Build prompt or model interface.
4. Call model or mock model through one gateway function.
5. Parse JSON output.
6. Validate required fields and allowed values.
7. Mark invalid outputs as `invalid` or `needs_review`.
8. Save extracted records.

MVP flow:

```text
message input
  -> normalizer
  -> model gateway
  -> JSON parser
  -> schema validator
  -> accepted record or review queue
```

If a real model is unavailable, use a mock extractor for the first pass, but keep the same interface.

The model call should not be scattered across the app. It should sit behind a single boundary:

```text
extract_intake_record(message, schema, route)
```

## V1 Milestone

After MVP works, add reliability controls.

V1 should include:

- model route selection;
- fallback on validation failure;
- retry with validation errors;
- human correction flow;
- eval set;
- error categories;
- route metadata;
- cost and latency estimates, even if approximate.

Route examples:

```text
standard_extraction:
Short, ordinary messages.

long_context_extraction:
Long message thread.

strict_retry:
Validation failed, retry with explicit validation error.

human_review:
Risky, ambiguous, or repeatedly invalid.
```

Do not over-build routing. The point is to make model selection explicit and observable.

## Model Boundary Milestone

Add a short file named `model-boundary.md`.

It should explain:

- what the model is allowed to do;
- what the model is not allowed to do;
- what the schema requires;
- what validation happens after the model;
- when fallback happens;
- when human review happens;
- why fine-tuning is not the first fix for most failures.

This keeps the project aligned with Part 4: models as components, not autonomous systems.

## Core Components

### 1. Input Loader

Responsibilities:

- read text, CSV, or JSON;
- assign `message_id`;
- preserve original text;
- reject empty messages;
- report malformed input.

Expected behavior:

```text
If 40 messages are provided and 3 are empty,
the system should process 37 and report 3 validation errors.
```

### 2. Schema Definition

Define the output contract in code.

The schema should include:

- field names;
- field types;
- required fields;
- enum values;
- summary length limit;
- review rules;
- missing information format.

Example record:

```json
{
  "message_id": "MSG-001",
  "issue_type": "billing_duplicate_charge",
  "urgency": "high",
  "requested_action": "refund_review",
  "summary": "Customer reports duplicate Pro charge and requests refund review.",
  "needs_human_review": true,
  "missing_information": ["account_id", "charge_id"]
}
```

### 3. Prompt or Instruction Template

The prompt is an interface, not a motivational essay.

It should include:

- task;
- input text;
- schema;
- allowed values;
- escalation rules;
- output-only JSON instruction;
- examples only if useful;
- maximum summary length.

Bad instruction:

```text
Analyze this message and be helpful.
```

Better instruction:

```text
Convert the message into one JSON object matching this schema.
Use only allowed enum values.
If required facts are missing, add them to missing_information.
If the request involves refunds, account ownership, security, or deletion, set needs_human_review to true.
Return JSON only.
```

### 4. Model Gateway

Create one function or module:

```text
call_model(route, prompt, input, schema)
```

The gateway should record:

- route;
- model name or mock name;
- latency;
- fallback used;
- approximate cost;
- raw output;
- parsed output status.

This project may use a real provider or a local/mock model. The requirement is the boundary.

### 5. Router and Fallbacks

Route by task shape and risk.

Example routing table:

| Case | Route |
|---|---|
| short normal message | `standard_extraction` |
| long thread | `long_context_extraction` |
| missing JSON or invalid enum | `strict_retry` |
| risky request | `human_review` |
| repeated validation failure | `human_review` |

Persist route metadata:

```json
{
  "message_id": "MSG-001",
  "primary_route": "standard_extraction",
  "fallback_route": "strict_retry",
  "fallback_reason": "invalid enum value",
  "final_status": "needs_review"
}
```

### 6. JSON Parser

The parser should handle:

- valid JSON;
- extra text around JSON;
- malformed JSON;
- missing fields;
- wrong types.

Invalid output should not pass silently.

If parsing fails:

```text
status = invalid
validation_errors includes parse error
route to retry or human review
```

### 7. Validator

Validate:

- required fields exist;
- enum values are allowed;
- `summary` is not too long;
- `missing_information` is a list;
- `needs_human_review` follows risk rules;
- high-risk actions are not marked safe.

Validator output:

```json
{
  "message_id": "MSG-001",
  "status": "needs_review",
  "validation_errors": [],
  "review_reasons": ["refund_review requires human review"]
}
```

### 8. Human Correction Flow

Allow user to correct:

- issue type;
- urgency;
- requested action;
- summary;
- missing information;
- review flag.

Save correction:

```json
{
  "message_id": "MSG-001",
  "model_output": {},
  "human_corrected": {},
  "correction_reason": "wrong requested_action"
}
```

Corrections become evaluation examples.

### 9. Error Analysis

Classify failures:

```text
invalid_json
missing_required_field
wrong_enum
wrong_issue_type
wrong_urgency
wrong_requested_action
bad_summary
missing_information_error
over_review
under_review
wrong_route
```

Error report should show:

- top failure categories;
- examples;
- likely cause;
- recommended fix.

Example:

```text
Failure: wrong_requested_action
Count: 7
Likely cause: enum names are too similar
Fix: improve schema descriptions and add examples
Not a fine-tuning candidate yet
```

### 10. Adaptation Decision

Before fine-tuning, create `adaptation-decision.md`.

It should answer:

- What is baseline accuracy?
- What are top failure categories?
- Are failures caused by weak schema?
- Are failures caused by prompt ambiguity?
- Are failures caused by missing facts?
- Are failures caused by model capability?
- Would examples or routing fix it first?
- Is there enough labeled data to tune?
- What risk would tuning introduce?

Default stance:

```text
Do not fine-tune until schema, prompt, route, validation, and examples have been improved and measured.
```

## Architecture

```text
messages
  -> input loader
  -> normalizer
  -> router
  -> prompt/template builder
  -> model gateway
  -> JSON parser
  -> schema validator
  -> review queue
  -> correction store
  -> eval runner
  -> error analysis
```

Key boundary:

```text
Model proposes structured output.
Validator decides whether it is usable.
Human review handles risk and uncertainty.
```

## Data Model

### Message

```json
{
  "message_id": "MSG-001",
  "source": "email",
  "raw_text": "Hi, I upgraded to Pro yesterday...",
  "created_at": "2026-01-10T10:00:00Z"
}
```

### Extracted Record

```json
{
  "record_id": "REC-001",
  "message_id": "MSG-001",
  "issue_type": "billing_duplicate_charge",
  "urgency": "high",
  "requested_action": "refund_review",
  "summary": "Customer reports duplicate Pro charge and requests urgent refund review.",
  "needs_human_review": true,
  "missing_information": ["account_id", "charge_id"],
  "status": "needs_review"
}
```

### Route Metadata

```json
{
  "message_id": "MSG-001",
  "primary_route": "standard_extraction",
  "fallback_used": false,
  "fallback_reason": null,
  "model_name": "chosen-or-mock-model",
  "latency_ms": 820,
  "cost_estimate": 0.002,
  "validation_result": "pass"
}
```

### Validation Error

```json
{
  "message_id": "MSG-014",
  "field": "urgency",
  "error": "invalid_enum",
  "value": "very urgent",
  "allowed_values": ["low", "normal", "high", "critical"]
}
```

### Labeled Example

```json
{
  "message_id": "MSG-001",
  "raw_text": "Hi, I upgraded to Pro yesterday...",
  "expected": {
    "issue_type": "billing_duplicate_charge",
    "urgency": "high",
    "requested_action": "refund_review",
    "needs_human_review": true
  }
}
```

## Evaluation Dataset

Create `labeled-examples.json` with at least 30 examples.

Include:

- 10 easy;
- 10 normal;
- 5 edge;
- 5 should-review or should-escalate.

Each labeled example should include:

```text
raw_text
expected_issue_type
expected_urgency
expected_requested_action
expected_needs_human_review
expected_missing_information
```

## Seeded Failure Cases

Include cases that expose model and interface weaknesses.

### Case 1: Multiple Requests

```text
Message asks for invoice download and refund.
Expected: refund_review, needs_human_review true, missing charge/account info.
```

### Case 2: High-Risk Action Hidden in Friendly Text

```text
"Could you quickly transfer ownership to my teammate?"
Expected: account_review, needs_human_review true.
```

### Case 3: Missing Identifiers

```text
Customer asks for refund but gives no account or charge ID.
Expected: missing_information includes account_id and charge_id.
```

### Case 4: Ambiguous Complaint

```text
"This is broken and unacceptable."
Expected: unknown or needs_clarification, needs_human_review true or review reason.
```

### Case 5: Invalid Model Output

```text
Mock or test model returns urgency = "very high".
Expected: validator rejects invalid enum.
```

### Case 6: Unsafe Private Data Request

```text
User asks for another customer's billing details.
Expected: needs_human_review true or escalation, not normal answer_question.
```

## Acceptance Rubric

### Basic Pass

- loads messages;
- defines schema;
- calls model or mock model through one gateway;
- parses JSON;
- validates required fields;
- saves extracted records;
- marks invalid output as invalid or review.

### Strong Pass

- supports model routes;
- retries validation failures with explicit error;
- routes risky cases to human review;
- saves route metadata;
- supports human correction;
- produces validation report;
- includes labeled eval examples.

### Excellent

- measures field-level accuracy;
- measures review-flag accuracy;
- reports invalid JSON rate;
- reports fallback success rate;
- includes seeded failure cases;
- produces `error-analysis.md`;
- produces `adaptation-decision.md`;
- explains why fine-tuning is or is not justified.

## Metrics

Measure:

- valid JSON rate;
- schema validation pass rate;
- field-level accuracy;
- issue type accuracy;
- requested action accuracy;
- human-review flag precision and recall;
- invalid output rate;
- fallback success rate;
- under-review rate for risky messages;
- cost per valid extraction;
- p95 latency.

Suggested targets:

```text
Valid JSON rate: >= 95%
Schema validation pass rate after retry: >= 95%
Human-review recall for risky cases: >= 95%
High-risk under-review count: 0
Error report generated: yes
```

Human-review recall matters more than convenience.

It is better to route a risky message to review than to incorrectly mark it safe.

## Final Deliverables

At the end, the learner should have:

- runnable CLI, notebook, or local app;
- `messages.json` or `messages.csv`;
- `labeled-examples.json`;
- schema definition;
- model gateway;
- router/fallback logic;
- validator;
- correction store;
- `extracted-records.json`;
- `validation-report.md`;
- `error-analysis.md`;
- `adaptation-decision.md`;
- `model-boundary.md`;
- short project `README.md`.

Example file layout:

```text
project-04-structured-intake-extractor/
  README.md
  model-boundary.md
  adaptation-decision.md
  data/
    messages.json
    labeled-examples.json
  src/
    loader.py
    schema.py
    prompt.py
    model_gateway.py
    router.py
    parser.py
    validator.py
    review.py
    evaluate.py
    errors.py
  outputs/
    extracted-records.json
    validation-report.md
    error-analysis.md
```

The exact language and framework can differ. The contract, validation, and error analysis should not.

## Demo Script

A 3-minute demo should show:

1. Load sample messages.
2. Extract structured records.
3. Show one valid record.
4. Show one risky record routed to human review.
5. Show one invalid model output rejected by validator.
6. Correct one record manually.
7. Run evaluation.
8. Open `error-analysis.md`.
9. Open `adaptation-decision.md`.

Demo narrative:

```text
This project proves that models should be treated as bounded components.
The model converts messy text into a typed contract, but the validator decides whether output is usable.
Risky requests go to review, invalid outputs do not pass silently, and failures become error analysis before any fine-tuning decision.
```

## What Learner Understands

After completing this project, the learner should understand:

- prompting is interface design;
- model outputs need contracts and validators;
- context budget affects extraction quality;
- routing and fallbacks should be explicit;
- fine-tuning is not the first fix for bad schema or bad prompt;
- invalid JSON is a system failure, not a small formatting issue;
- human correction creates evaluation data;
- error analysis tells what to improve next.

## Extension

Later upgrades:

- Part 5: add retrieval for policy-aware extraction;
- Part 6: extract from PDFs, forms, screenshots, and transcripts;
- Part 7: route approved records into workflow tools;
- Part 8: add production traces, eval dashboard, cost, latency, and release gates;
- Part 9: add privacy rules, prompt-injection tests, and retention policy.
