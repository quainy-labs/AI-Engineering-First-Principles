# Project 1: Support Workflow Intelligence Console

## Build

Build a small app that analyzes a real or sample support workflow and identifies where AI should help, where code should help, and where humans must stay in control.

This is not a document exercise. It is a workflow analysis product.

## User

Support lead, operations lead, or founder who wants to understand repeated support work before building AI automation.

## Product Outcome

User uploads or enters support tickets. System produces:

- workflow map;
- task breakdown;
- automation opportunity score;
- AI necessity table;
- baseline metrics;
- risk notes;
- recommended first automation.

## Sample Data

Use CSV or JSON:

```csv
ticket_id,created_at,category,customer_message,resolution,minutes_to_resolve,escalated
T001,2026-01-10,billing,"I was charged twice","Refunded duplicate charge",18,true
T002,2026-01-11,access,"I cannot reset my API key","Sent reset instructions",7,false
```

Minimum dataset:

- 30 tickets;
- 5 categories;
- resolution text;
- time-to-resolve field;
- escalation flag.

## Core Features

### 1. Ticket Import

Input:

- CSV upload;
- JSON file;
- or pasted rows.

System validates required fields:

- `ticket_id`
- `customer_message`
- `resolution`
- `minutes_to_resolve`
- `escalated`

### 2. Workflow Step Classifier

Classify each ticket into workflow tasks:

- information lookup;
- policy decision;
- account lookup;
- draft response;
- refund/action;
- escalation;
- unclear.

Start with rules. Add model classification only after rules fail.

### 3. AI Necessity Analyzer

For each workflow task, assign best method:

- deterministic code;
- database/API lookup;
- search/retrieval;
- structured extraction;
- generation;
- tool action;
- human decision.

### 4. Baseline Dashboard

Show:

- ticket volume by category;
- median resolution time;
- escalation rate;
- top repeated issues;
- estimated minutes spent per category;
- candidate automation areas.

### 5. Recommendation Report

Generate first automation recommendation:

```text
Automate: draft answer for API key reset tickets
Why: high volume, low risk, repeated answer, no account mutation
Do not automate: refund approval
Why: policy/account-specific, financial action, needs approval
```

## Architecture

```text
CSV/JSON input
-> validator
-> ticket normalizer
-> rule/model classifier
-> metrics calculator
-> automation scorer
-> report generator
-> dashboard or CLI output
```

## Data Model

```json
{
  "ticket_id": "T001",
  "message": "I was charged twice",
  "resolution": "Refunded duplicate charge",
  "category": "billing",
  "minutes_to_resolve": 18,
  "escalated": true,
  "workflow_tasks": ["policy_decision", "financial_action"],
  "recommended_method": "human_decision",
  "risk_level": "high"
}
```

## Suggested Implementation

Start simple:

- Python CLI with CSV input;
- or local web app;
- or notebook pipeline.

Components:

- `load_tickets`
- `validate_tickets`
- `classify_task`
- `calculate_baseline`
- `score_automation_opportunity`
- `generate_report`

## Evaluation

Create 20 labeled tickets.

Measure:

- task classification accuracy;
- automation recommendation quality;
- false automation risk;
- baseline metric correctness.

Acceptance:

- identifies top 3 repeated workflows;
- separates AI-suitable tasks from deterministic/human tasks;
- flags high-risk automation candidates;
- produces readable recommendation.

## What Learner Understands

- AI starts from workflow, not model.
- Not all repeated work should be automated.
- Baselines create useful project direction.
- Human approval can be designed before any model call.

## Extension

Add:

- model-assisted classification;
- editable labels;
- export to `system-brief.md`;
- project recommendation score;
- before/after simulation.
