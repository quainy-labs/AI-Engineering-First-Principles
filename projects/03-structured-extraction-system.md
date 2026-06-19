# Project 3: Structured Intake Extractor

## Build

Build a system that converts messy inbound text into validated structured JSON for downstream workflow.

Example: convert support emails into typed intake records.

## User

Operations teammate who receives messy text and needs consistent records for routing, search, or automation.

## Product Outcome

User pastes or uploads messages. System returns:

- validated JSON;
- confidence/needs-review flag;
- validation errors;
- human-correctable fields;
- error analysis report.

## Sample Input

```text
Subject: Charged twice

Hi, I upgraded to Pro yesterday but I see two charges on my card.
Can someone refund the duplicate? This is urgent because our finance
team closes books today.
```

## Target Output

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

## Core Features

### 1. Input Queue

Accept:

- pasted text;
- `.txt` files;
- CSV rows;
- JSON list.

### 2. Extraction Schema

Define schema with:

- required fields;
- allowed values;
- validation rules;
- fallback behavior.

Example fields:

- `issue_type`
- `urgency`
- `requested_action`
- `summary`
- `needs_human_review`
- `missing_information`

### 3. Model Interface

Model call must include:

- task;
- input text;
- schema;
- refusal/escalation rule;
- output-only JSON requirement.

### 4. Validator

Validate output:

- valid JSON;
- required fields;
- enum values;
- max summary length;
- human review rule.

Invalid output should not pass silently.

### 5. Human Correction UI/CLI

Let user correct extracted fields and save corrected label for future eval.

### 6. Error Report

Track:

- invalid JSON;
- wrong label;
- missing field;
- over-refusal;
- under-refusal;
- wrong review flag.

## Architecture

```text
message input
-> normalizer
-> model extraction
-> schema validator
-> human correction
-> structured record store
-> eval/error report
```

## Data Model

```json
{
  "record_id": "MSG-001",
  "raw_text": "...",
  "extracted": {},
  "validation_errors": [],
  "human_corrected": {},
  "status": "valid | needs_review | invalid",
  "created_at": "2026-01-10T10:00:00Z"
}
```

## Suggested Implementation

Start with CLI:

```bash
extract messages.csv
validate outputs.json
review MSG-001
eval extraction-eval.json
```

Optional web UI:

- input panel;
- JSON output panel;
- validation errors;
- correction form;
- eval summary.

## Evaluation

Create 30 labeled examples:

- 10 easy;
- 10 normal;
- 5 edge;
- 5 should-refuse.

Measure:

- valid JSON rate;
- field-level accuracy;
- refusal accuracy;
- human-review accuracy;
- invalid output rate.

Acceptance:

- 95%+ valid JSON on eval set;
- human-review flag catches risky cases;
- invalid outputs routed to review;
- error report shows top 3 failure types.

## What Learner Understands

- Prompting is interface design.
- Models should produce validated outputs, not vague text.
- Schema quality affects system reliability.
- Human corrections become eval data.

## Extension

Add:

- batch processing;
- record search;
- model comparison;
- active learning queue;
- automatic retry with safer fallback;
- downstream routing simulation.
