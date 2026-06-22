# Project 1: Support Workflow Intelligence Console

## Concept

Before building an AI feature, a team must understand the workflow. This project turns messy support tickets into a product decision console that shows where AI helps, where deterministic software helps, and where humans must stay in control.

This is not a writing exercise. It is a small analytical product.

The learner should finish with a runnable CLI or notebook that ingests tickets, classifies workflow steps, calculates baseline metrics, scores automation opportunities, and produces a clear recommendation.

## What You Will Build

Build a Support Workflow Intelligence Console.

The system accepts support tickets as CSV or JSON and produces:

- workflow map;
- task breakdown;
- repeated issue clusters;
- baseline metrics;
- automation opportunity score;
- AI necessity table;
- risk notes;
- recommended first automation;
- "do not automate yet" list;
- exportable project brief.

The product answers one question:

```text
What should we improve first, and should that improvement use AI?
```

## Chapters Practiced

This project proves understanding from:

- Chapter 1: AI engineering is system design;
- Chapter 2: when not to use AI;
- Chapter 3: messy workflow to system boundary;
- Chapter 4: metrics, baselines, and meaningful impact.

The main lesson:

```text
AI project selection should come from workflow evidence, not model excitement.
```

Do not require future course concepts here. This project should be possible after Part 1.

Later parts can improve the same project:

- Part 2 can turn it into a proper app with auth, storage, APIs, and background jobs;
- Part 3 can add stronger schemas, lineage, and data quality checks;
- Part 4 can add model-assisted classification for unclear tickets;
- Part 8 can add observability and deployment thinking;
- Part 9 can add security, privacy, and governance review.

## User Story

Primary user:

```text
Support lead, operations lead, or founder.
```

User story:

```text
As a support lead,
I want to upload recent support tickets,
so I can identify repeated workflows, estimate current workload, and choose the safest first automation.
```

The user does not want an abstract AI strategy. They want a concrete answer:

```text
Start here.
Do not automate this yet.
This is why.
This is the expected impact.
```

## System Requirements

### Input

Accept one of:

- CSV file;
- JSON file;
- pasted CSV text;
- hardcoded sample dataset for demo mode.

Required fields:

```text
ticket_id
created_at
category
customer_message
resolution
minutes_to_resolve
escalated
```

Optional fields:

```text
agent_id
customer_plan
refund_amount
tags
first_response_minutes
reopened
```

### Output

The system should produce:

- validation report;
- baseline dashboard or summary;
- workflow classification table;
- automation scoring table;
- top 3 recommended improvements;
- top 3 risky areas to avoid;
- exportable `workflow-intelligence-report.md` or JSON report.

### Interface

Choose one implementation path:

```text
CLI:
workflow-console analyze tickets.csv --out report.md

Notebook:
Run all cells and produce tables/charts/report.
```

For this first project, CLI or notebook is enough.

A local web app should be treated as a later Part 2 upgrade, not a requirement for Project 1.

## Starter Data

Create a sample file named `sample-support-tickets.csv`.

Minimum dataset:

- 40 tickets;
- at least 5 categories;
- at least 5 escalated tickets;
- at least 5 high-risk financial or account-change tickets;
- at least 5 low-risk repeated information tickets;
- at least 5 ambiguous tickets.

Example rows:

```csv
ticket_id,created_at,category,customer_message,resolution,minutes_to_resolve,escalated,customer_plan,reopened
T001,2026-01-10,billing,"I was charged twice after upgrading to Pro.","Refunded duplicate charge after checking billing records.",18,true,pro,false
T002,2026-01-11,access,"I cannot reset my API key. The reset email never arrives.","Sent API key reset instructions and verified email delivery.",7,false,team,false
T003,2026-01-11,how_to,"How do I invite a teammate to my workspace?","Shared workspace invitation documentation.",4,false,team,false
T004,2026-01-12,billing,"Please refund last month. We did not use the product.","Escalated to billing policy review.",22,true,starter,true
T005,2026-01-12,bug,"Export to CSV fails when I filter by date.","Created bug report with reproduction steps.",16,true,pro,false
T006,2026-01-13,account,"Change the account owner to priya@example.com.","Requested admin confirmation before ownership change.",14,true,enterprise,false
```

Seed the dataset with patterns:

```text
Low-risk automation candidates:
- password reset instructions;
- API key reset guidance;
- invite teammate instructions;
- where to find invoices;
- explain plan limits.

Medium-risk candidates:
- bug reproduction gathering;
- routing requests to correct team;
- summarizing long tickets;
- drafting support replies.

High-risk or human-required:
- refunds;
- account ownership change;
- data deletion;
- billing dispute;
- security incident;
- legal/compliance request.
```

## MVP Milestone

Build the simplest useful version first.

MVP must:

1. Load CSV or JSON.
2. Validate required fields.
3. Normalize booleans and numbers.
4. Classify each ticket into workflow tasks using deterministic rules.
5. Calculate baseline metrics.
6. Score automation opportunities.
7. Print or export a recommendation report.

MVP workflow:

```text
input tickets
  -> validate rows
  -> normalize fields
  -> classify workflow task
  -> compute baseline metrics
  -> score opportunities
  -> generate report
```

Do not use an LLM in the MVP.

The first principle is to prove that workflow structure and baseline metrics already reveal useful AI opportunities.

## V1 Milestone

After MVP works, improve it into a stronger product.

V1 should add:

- editable classification labels;
- repeated issue detection;
- risk scoring explanation;
- estimated time-saved model;
- "AI vs code vs human" recommendation;
- report export;
- evaluation against labeled examples.

Optional V1 AI feature:

```text
Use a model only for tickets that deterministic rules mark as unclear.
```

Treat this as a later Part 4 upgrade. Project 1 itself should work without a model.

This teaches the right pattern:

```text
rules first -> model for ambiguity -> human review for risk
```

## Product Boundary Milestone

This project is not a production system, but it should teach product boundary thinking.

Add short design notes for:

- what data should not be uploaded;
- what private customer data may appear in tickets;
- how recommendations should be reviewed before automation is built.

Do not implement detailed privacy, auth, retention, or governance controls here. Those belong to later parts.

Later Part 2 web app upgrade can add:

- file size limit;
- validation errors;
- upload status;
- export button;
- clear demo mode;
- no silent failure.

## Core Components

### 1. Ticket Loader

Responsibilities:

- read CSV or JSON;
- reject unsupported formats;
- preserve original row number;
- report malformed rows.

Expected behavior:

```text
If 40 rows are uploaded and 3 are invalid,
the system should process 37 valid rows and report 3 validation errors.
```

### 2. Validator

Required validations:

- `ticket_id` exists;
- `customer_message` is not empty;
- `resolution` is not empty;
- `minutes_to_resolve` is numeric and non-negative;
- `escalated` is boolean or convertible;
- `created_at` is parseable date.

Validation output:

```json
{
  "valid_rows": 37,
  "invalid_rows": 3,
  "errors": [
    {
      "row": 12,
      "field": "minutes_to_resolve",
      "problem": "not numeric"
    }
  ]
}
```

### 3. Workflow Classifier

Classify each ticket into one or more workflow tasks.

Task labels:

```text
information_lookup
how_to_guidance
account_lookup
policy_decision
billing_review
refund_or_credit
bug_triage
security_review
data_deletion
draft_response
escalation
unclear
```

Start with keyword and category rules.

Example:

```text
If category = billing and message contains refund:
workflow_tasks = ["billing_review", "refund_or_credit"]
risk_level = high
```

Then allow human correction:

```text
Predicted: how_to_guidance
Corrected: account_lookup
```

### 4. AI Necessity Analyzer

For each task, recommend the best method.

Method options:

```text
deterministic_code
database_or_api_lookup
search_or_retrieval
structured_extraction
generation
tool_action
human_decision
do_not_automate_yet
```

These are recommendation labels only. The project should not implement retrieval, generation, extraction, or tools yet.

Example rules:

```text
how_to_guidance:
search_or_retrieval or deterministic_code

draft_response:
generation with human review

refund_or_credit:
human_decision with tool-assisted evidence

data_deletion:
human_decision with strict workflow and audit

unclear:
do_not_automate_yet
```

The analyzer should explain the recommendation.

Bad output:

```text
Use AI.
```

Good output:

```text
Use retrieval-backed draft response with human review because the task is repeated, language-heavy, medium risk, and benefits from policy context.
```

### 5. Baseline Metrics

Calculate:

- total tickets;
- valid tickets;
- invalid tickets;
- tickets by category;
- median minutes to resolve;
- average minutes to resolve;
- escalation rate;
- reopened rate, if available;
- total minutes spent by category;
- top repeated message patterns;
- high-risk ticket count.

Example output:

```text
Total valid tickets: 40
Median resolution time: 9 minutes
Escalation rate: 22.5%
Top category by workload: billing, 310 minutes/month
Top low-risk repeated issue: API key reset
```

### 6. Automation Scorer

Score each workflow candidate from 0 to 100.

Suggested scoring:

```text
volume score: 0-25
time cost score: 0-20
repeatability score: 0-20
risk adjustment: -30 to 0
data availability score: 0-15
human approval need: -10 to 0
```

Example:

```json
{
  "workflow": "api_key_reset_guidance",
  "volume_score": 22,
  "time_cost_score": 12,
  "repeatability_score": 18,
  "risk_adjustment": -2,
  "data_availability_score": 12,
  "human_approval_adjustment": 0,
  "total_score": 62
}
```

Interpretation:

```text
70-100: strong first automation candidate
50-69: promising, needs design review
30-49: improve process or gather more data first
0-29: do not automate yet
```

### 7. Recommendation Report

Generate a report with:

- executive summary;
- baseline metrics;
- top repeated workflows;
- automation candidate table;
- AI necessity table;
- risk notes;
- recommended first build;
- workflows to avoid;
- next data to collect.

Example recommendation:

```text
Recommended first automation:
API key reset guidance assistant.

Why:
High volume, low risk, repeated resolution text, no account mutation, clear success criteria.

Suggested system:
Retrieval-backed answer draft with citation to reset documentation.

Do not automate yet:
Refund approvals.

Why:
Financial action, policy exceptions, account-specific context, escalation history required.
```

## Architecture

```text
CSV/JSON input
  -> ticket loader
  -> validator
  -> normalizer
  -> workflow classifier
  -> baseline calculator
  -> automation scorer
  -> recommendation engine
  -> report exporter
```

Optional web architecture:

```text
browser upload
  -> API endpoint
  -> validation service
  -> analysis service
  -> local database or in-memory store
  -> dashboard
  -> report export
```

## Data Model

### Ticket

```json
{
  "ticket_id": "T001",
  "created_at": "2026-01-10",
  "category": "billing",
  "customer_message": "I was charged twice after upgrading to Pro.",
  "resolution": "Refunded duplicate charge after checking billing records.",
  "minutes_to_resolve": 18,
  "escalated": true,
  "customer_plan": "pro",
  "reopened": false
}
```

### Analyzed Ticket

```json
{
  "ticket_id": "T001",
  "workflow_tasks": ["billing_review", "refund_or_credit"],
  "recommended_method": "human_decision",
  "risk_level": "high",
  "reason": "Financial action with policy/account-specific context.",
  "automation_candidate": false
}
```

### Workflow Candidate

```json
{
  "workflow": "api_key_reset_guidance",
  "ticket_count": 8,
  "estimated_monthly_minutes": 64,
  "risk_level": "low",
  "recommended_method": "search_or_retrieval",
  "automation_score": 76,
  "first_build_recommendation": true
}
```

## Evaluation Dataset

Create `labeled-support-tickets.csv` with at least 25 labeled rows.

Required label fields:

```text
expected_workflow_tasks
expected_risk_level
expected_recommended_method
expected_automation_candidate
```

Include:

- 8 low-risk repeated guidance tickets;
- 5 billing/refund tickets;
- 4 account-change tickets;
- 3 bug triage tickets;
- 3 ambiguous tickets;
- 2 security or data deletion tickets.

## Seeded Failure Cases

Include cases that test whether the system avoids bad automation decisions.

### Case 1: Refund Request Looks Repeated

```text
Many users ask for refunds.
Expected: system should not recommend full automation.
Reason: financial action and policy exceptions require human decision.
```

### Case 2: Account Ownership Change

```text
User asks to transfer account ownership.
Expected: high risk, human decision, approval workflow.
```

### Case 3: Ambiguous Complaint

```text
"Nothing works and this is urgent."
Expected: unclear or escalation, not direct automation.
```

### Case 4: Low-Risk How-To

```text
"How do I invite a teammate?"
Expected: strong automation candidate using deterministic docs/search.
```

### Case 5: Data Deletion Request

```text
User asks to delete all account data.
Expected: high risk, do not automate yet, needs controlled workflow and audit.
```

## Acceptance Rubric

### Basic Pass

- loads CSV or JSON;
- validates required fields;
- calculates baseline metrics;
- classifies workflow tasks with rules;
- exports a readable report.

### Strong Pass

- handles invalid rows without crashing;
- separates low-risk, medium-risk, and high-risk workflows;
- recommends method per workflow;
- identifies top 3 automation candidates;
- flags top 3 do-not-automate workflows;
- includes explanation for each recommendation.

### Excellent

- includes labeled evaluation dataset;
- reports classification accuracy;
- reports false automation risk;
- supports human correction of labels;
- exports `workflow-intelligence-report.md`;
- includes estimated time saved;
- includes privacy/risk notes for uploaded ticket data.

## Metrics

Measure:

- validation success rate;
- workflow classification accuracy;
- risk classification accuracy;
- false automation recommendation rate;
- baseline metric correctness;
- report completeness.

Suggested targets:

```text
Workflow classification accuracy: >= 80% on labeled examples
High-risk recall: >= 95%
False automation recommendation rate for high-risk tickets: 0%
Report includes all required sections: 100%
```

High-risk recall matters more than overall accuracy.

It is better to mark a ticket as needing review than to recommend unsafe automation.

## Final Deliverables

At the end, the learner should have:

- runnable CLI, notebook, or local app;
- `sample-support-tickets.csv`;
- `labeled-support-tickets.csv`;
- generated `workflow-intelligence-report.md`;
- generated `analysis-output.json`;
- short `README.md` explaining how to run the project;
- screenshots or terminal output for demo;
- notes explaining top recommendation and rejected automation candidates.

Example final file layout:

```text
project-01-support-workflow-console/
  README.md
  data/
    sample-support-tickets.csv
    labeled-support-tickets.csv
  src/
    loader.py
    validator.py
    classifier.py
    metrics.py
    scorer.py
    report.py
  outputs/
    analysis-output.json
    workflow-intelligence-report.md
```

The exact stack can differ. The deliverables should not.

## Demo Script

A 3-minute demo should show:

1. Load `sample-support-tickets.csv`.
2. Show validation summary.
3. Show baseline metrics.
4. Show workflow candidate table.
5. Open recommendation report.
6. Explain why the top recommendation is safe.
7. Explain why one tempting workflow should not be automated.

Demo narrative:

```text
I uploaded 40 support tickets.
The system found API key reset guidance as the strongest first automation because it is repeated, low risk, and has clear documentation.
It rejected refund approval as full automation because it is financial, policy-dependent, and needs human review.
This proves the project starts from workflow evidence instead of assuming AI is the answer.
```

## What Learner Understands

After completing this project, the learner should understand:

- AI starts from workflow, not model choice;
- baseline metrics reveal where impact may exist;
- repeated work is not automatically safe to automate;
- deterministic code, retrieval, generation, tools, and humans each have different roles;
- risk should affect automation priority;
- false automation recommendations are dangerous;
- a good AI engineering project begins with system boundary and evidence.

## Extension

Add:

- model-assisted classification for unclear tickets;
- editable review UI;
- clustering for repeated message patterns;
- before/after simulation;
- chart dashboard;
- export to `system-brief.md`;
- privacy redaction before analysis;
- multi-team comparison;
- recommendation confidence score.
