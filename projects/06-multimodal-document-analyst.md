# Project 6: Multimodal Document Analyst

## Concept

Real business information does not arrive only as clean text.

It arrives as PDFs, scanned forms, screenshots, invoices, tables, contracts, images, and sometimes audio notes. Before an AI system can reason about these inputs, it must extract observations, preserve evidence, validate fields, and route uncertainty to humans.

This project builds a multimodal document analyst that turns messy documents into validated structured records with source evidence and human correction.

The learner should finish with a runnable CLI, notebook, or local app that accepts document inputs, extracts text/layout/table evidence, uses a model or parser to create structured records, validates fields, shows page/source evidence, and supports correction.

The focus is:

```text
document or image -> extraction -> evidence -> structured record -> validation -> human correction
```

## What You Will Build

Build a Multimodal Document Analyst.

The system accepts invoices, forms, screenshots, or scanned documents and returns:

- detected document type;
- extracted text;
- table rows when present;
- structured fields;
- evidence references;
- confidence or needs-review flags;
- validation errors;
- human corrections;
- evaluation report.

The product answers one question:

```text
Can this system turn visual or document evidence into trustworthy structured data?
```

## Chapters Practiced

This project proves understanding from:

- Chapter 26: Document AI: OCR, tables, forms, and PDFs;
- Chapter 27: vision, image, and multimodal inputs;
- Chapter 28: audio, speech, and realtime AI, if using optional audio notes;
- Chapter 29: AI UX, trust, feedback, and human correction.

It may reuse earlier parts for app structure, schemas, model interfaces, and evaluation, but it should not require future course concepts.

Do not require tools, workflow actions, agents, production deployment, LLMOps, security threat modeling, privacy governance, or product economics here.

Later parts can improve the same project:

- Part 7 can turn reviewed records into tool-assisted workflows;
- Part 8 can add production observability, batching, cost, latency, deployment, and rollback;
- Part 9 can add privacy, PII redaction, prompt-injection testing, retention, and governance review.

## User Story

Primary user:

```text
Operations, finance, legal, support, or compliance teammate.
```

User story:

```text
As a finance operations teammate,
I want to upload invoices and scanned forms,
so I can extract key fields with evidence, correct mistakes, and avoid manually copying data.
```

The user does not want a magical answer. They need inspectable extraction:

```text
field
value
page
source excerpt or bounding area
confidence
validation status
review reason
```

## System Requirements

### Input

Accept at least two document types:

- native PDF;
- image or scanned document;
- screenshot;
- table-heavy PDF or CSV export.

Optional:

- audio note attached to a document;
- short transcript pasted beside the document.

Supported file types for this project:

```text
.pdf
.png
.jpg
.jpeg
.txt
.csv
```

If a file type is not supported, the system should fail clearly.

### Target Workflow

Choose one primary workflow.

Recommended workflow:

```text
invoice intake
```

Invoice schema:

```text
vendor_name
invoice_number
invoice_date
due_date
subtotal
tax
total
currency
line_items
payment_terms
missing_fields
needs_human_review
review_reasons
```

Optional second workflow:

```text
support intake form
```

Do not try to support every document type at once. Start with one workflow and add one variation.

### Output

The system should produce:

- `extracted-documents.json`;
- `document-quality-report.md`;
- `field-validation-report.md`;
- `correction-log.json`;
- `multimodal-eval-report.md`;
- optional page preview screenshots or evidence images.

### Interface

Choose one implementation path:

```text
CLI:
doc-analyst extract invoice-001.pdf --out extracted-documents.json
doc-analyst review DOC-001
doc-analyst eval eval-documents.json

Notebook:
Run document parsing, extraction, validation, review, and eval cells.

Local app:
Upload document, inspect preview, correct fields, export record.
```

CLI or notebook is enough. A local app is useful if the learner wants to practice UX and correction flows.

## Starter Documents

Create a `documents/` folder with at least 12 sample inputs.

Recommended structure:

```text
documents/
  invoices/
    invoice-clean-001.pdf
    invoice-scanned-002.png
    invoice-table-003.pdf
    invoice-conflicting-total-004.pdf
    invoice-missing-due-date-005.pdf
  forms/
    intake-form-clean.pdf
    intake-form-scan.png
  screenshots/
    billing-error-screen.png
  notes/
    invoice-audio-note.txt
```

If real PDFs are not available, generate simple sample files or use text/CSV stand-ins with the same fields. The important part is that the pipeline treats each source as a document with evidence.

Seed the dataset with:

```text
Easy:
- clean invoice with all fields visible;
- simple support intake form.

Normal:
- invoice with table line items;
- invoice with tax and subtotal;
- screenshot with visible error message.

Edge:
- scanned or blurry image;
- missing due date;
- conflicting subtotal and total;
- table split across pages;
- handwritten or low-confidence field, if available.

Should-review:
- total does not match line items;
- vendor name ambiguous;
- payment terms missing;
- document type unknown;
- sensitive data visible in screenshot.
```

Example invoice text stand-in:

```text
Invoice Number: INV-1001
Vendor: Northstar Cloud Services
Invoice Date: 2026-01-10
Due Date: 2026-02-10
Line Items:
- Cloud hosting, 10 units, $100.00 each, $1,000.00
- Support plan, 1 unit, $250.00 each, $250.00
Subtotal: $1,250.00
Tax: $100.00
Total: $1,350.00
Payment Terms: Net 30
```

Expected extraction:

```json
{
  "vendor_name": "Northstar Cloud Services",
  "invoice_number": "INV-1001",
  "invoice_date": "2026-01-10",
  "due_date": "2026-02-10",
  "subtotal": 1250.0,
  "tax": 100.0,
  "total": 1350.0,
  "currency": "USD",
  "line_items": [
    {
      "description": "Cloud hosting",
      "quantity": 10,
      "unit_price": 100.0,
      "amount": 1000.0
    },
    {
      "description": "Support plan",
      "quantity": 1,
      "unit_price": 250.0,
      "amount": 250.0
    }
  ],
  "payment_terms": "Net 30",
  "missing_fields": [],
  "needs_human_review": false,
  "review_reasons": []
}
```

## MVP Milestone

Build the simplest useful version first.

MVP must:

1. Load document files.
2. Detect supported file type.
3. Extract text or use provided text stand-in.
4. Extract invoice fields into schema.
5. Attach evidence for every extracted field.
6. Validate required fields and totals.
7. Mark uncertain or invalid records as `needs_human_review`.
8. Save structured output.

MVP flow:

```text
document file
  -> file loader
  -> parser or OCR
  -> extraction schema
  -> field extractor
  -> evidence mapper
  -> validator
  -> structured record or review queue
```

If OCR or PDF parsing libraries are not available, use text stand-ins for the first pass and keep the same evidence structure. Then replace the parser later.

The required learning is evidence-preserving document extraction, not a specific OCR vendor.

## V1 Milestone

After MVP works, add multimodal reliability.

V1 should include:

- native PDF parsing;
- OCR path for images/scans;
- table extraction or table-like parsing;
- document type classifier;
- confidence or quality score;
- page number evidence;
- bounding-box evidence if available;
- human correction flow;
- eval dataset;
- error report.

Optional V1 audio feature:

```text
Allow a short text transcript or audio-note transcript to provide extra context.
```

Example:

```text
The invoice is approved, but the due date may be missing from the scan.
```

Audio should not override document evidence. It should be treated as supporting context that may require review.

## Trust and Correction Milestone

Add a short file named `document-trust-and-review.md`.

It should explain:

- which document types are supported;
- what evidence is shown for each field;
- how uncertain fields are routed to review;
- how corrections are stored;
- which fields are never accepted without evidence;
- how scanned or blurry documents are handled;
- how optional audio notes are treated.

This keeps the project aligned with Part 6: multimodal inputs require trust, evidence, and correction loops.

## Core Components

### 1. Document Loader

Responsibilities:

- accept supported file types;
- assign document ID;
- record source filename;
- record file type;
- record page count when available;
- reject unsupported files clearly.

Document record:

```json
{
  "document_id": "DOC-001",
  "source_file": "invoice-clean-001.pdf",
  "file_type": "pdf",
  "pages": 1,
  "status": "loaded"
}
```

### 2. Parser and OCR Layer

Use available tools to extract:

- text;
- page numbers;
- tables if available;
- layout blocks if available;
- OCR confidence if available.

Parser output:

```json
{
  "document_id": "DOC-001",
  "pages": [
    {
      "page": 1,
      "text": "Invoice Number: INV-1001...",
      "tables": [],
      "ocr_confidence": 0.94
    }
  ]
}
```

Important rule:

```text
OCR is observation, not truth.
```

Low-confidence OCR should route important fields to review.

### 3. Document Type Classifier

Classify:

```text
invoice
intake_form
screenshot
contract_excerpt
unknown
```

Start with rules:

```text
If text contains "Invoice Number" and "Total", classify as invoice.
If text contains "Name", "Email", and form-like fields, classify as intake_form.
If file is image and contains error message text, classify as screenshot.
```

Use model-based classification only if rules are insufficient.

### 4. Extraction Schema

Define the schema in code.

Invoice required fields:

```text
vendor_name
invoice_number
invoice_date
total
currency
```

Optional fields:

```text
due_date
subtotal
tax
line_items
payment_terms
```

Review fields:

```text
missing_fields
needs_human_review
review_reasons
```

### 5. Field Extractor

Extract fields using:

- rules;
- regular expressions;
- table parsing;
- model call;
- or hybrid approach.

The project may use a multimodal model if available, but it is not required for every field. Deterministic extraction is often better for obvious fields.

Example strategy:

```text
invoice_number: regex/rule
dates: regex/rule
line_items: table parser
vendor_name: rule or model
payment_terms: rule or model
```

### 6. Evidence Mapper

Every extracted field needs evidence.

Evidence record:

```json
{
  "field": "invoice_number",
  "value": "INV-1001",
  "page": 1,
  "excerpt": "Invoice Number: INV-1001",
  "confidence": 0.96,
  "source_type": "ocr_text"
}
```

If bounding boxes are available:

```json
{
  "bbox": {
    "x": 120,
    "y": 80,
    "width": 220,
    "height": 30
  }
}
```

If evidence is missing, route field to review.

### 7. Table Extractor

For invoices, extract line items when present.

Line item schema:

```text
description
quantity
unit_price
amount
evidence
```

Validation:

```text
quantity * unit_price should approximately equal amount.
sum(line_items.amount) should approximately equal subtotal.
subtotal + tax should approximately equal total.
```

Use tolerances for rounding:

```text
allowed difference <= 0.01 for simple examples
```

### 8. Validator

Validate:

- required fields present;
- dates parse;
- numeric fields parse;
- total math checks;
- currency exists;
- every field has evidence;
- low-confidence important fields route to review;
- unsupported document types route to review.

Validator output:

```json
{
  "document_id": "DOC-001",
  "status": "needs_review",
  "validation_errors": [
    {
      "field": "total",
      "error": "total_does_not_match_subtotal_plus_tax"
    }
  ],
  "review_reasons": ["conflicting totals"]
}
```

### 9. Human Correction Flow

Allow user to correct:

- field value;
- missing field;
- line item;
- document type;
- review status.

Correction record:

```json
{
  "document_id": "DOC-001",
  "field": "due_date",
  "old_value": null,
  "new_value": "2026-02-10",
  "correction_reason": "visible on page 2",
  "corrected_by": "reviewer"
}
```

Corrections become evaluation data.

### 10. Evaluation Runner

Evaluate:

- clean native PDF;
- scanned invoice;
- table-heavy invoice;
- missing field;
- conflicting total;
- unknown document type;
- screenshot with visible error;
- optional audio-note context.

Report:

- field-level accuracy;
- required-field recall;
- table row accuracy;
- evidence coverage;
- validation accuracy;
- review routing accuracy;
- correction success.

## Architecture

```text
document input
  -> document loader
  -> parser/OCR layer
  -> document type classifier
  -> schema selector
  -> field extractor
  -> evidence mapper
  -> table extractor
  -> validator
  -> human correction
  -> structured record store
  -> eval runner
```

Optional local app architecture:

```text
upload UI
  -> extraction service
  -> preview/evidence panel
  -> correction form
  -> export record
```

Key boundary:

```text
Parser observes.
Extractor proposes.
Validator checks.
Human corrects uncertainty.
```

## Data Model

### Document Record

```json
{
  "document_id": "DOC-001",
  "source_file": "invoice-clean-001.pdf",
  "file_type": "pdf",
  "document_type": "invoice",
  "pages": 1,
  "status": "processed",
  "parser_version": "pdf-text-v1",
  "created_at": "2026-01-10T10:00:00Z"
}
```

### Extracted Record

```json
{
  "document_id": "DOC-001",
  "document_type": "invoice",
  "fields": {
    "vendor_name": "Northstar Cloud Services",
    "invoice_number": "INV-1001",
    "invoice_date": "2026-01-10",
    "due_date": "2026-02-10",
    "subtotal": 1250.0,
    "tax": 100.0,
    "total": 1350.0,
    "currency": "USD"
  },
  "line_items": [
    {
      "description": "Cloud hosting",
      "quantity": 10,
      "unit_price": 100.0,
      "amount": 1000.0
    }
  ],
  "needs_human_review": false,
  "review_reasons": []
}
```

### Evidence Record

```json
{
  "document_id": "DOC-001",
  "field": "total",
  "value": 1350.0,
  "page": 1,
  "excerpt": "Total: $1,350.00",
  "confidence": 0.95,
  "source_type": "pdf_text",
  "bbox": null
}
```

### Correction Record

```json
{
  "document_id": "DOC-004",
  "field": "total",
  "old_value": 1530.0,
  "new_value": 1350.0,
  "correction_reason": "OCR transposed digits",
  "created_at": "2026-01-10T10:00:00Z"
}
```

## Evaluation Dataset

Create `eval-documents.json` with at least 12 examples.

Each eval item should include:

```text
source_file
expected_document_type
expected_fields
expected_line_items
expected_missing_fields
expected_needs_review
review_reason
```

Include:

- 3 clean invoices;
- 2 scanned invoices;
- 2 table-heavy invoices;
- 1 missing required field;
- 1 conflicting total;
- 1 unknown document type;
- 1 screenshot;
- 1 optional audio-note/transcript case.

## Seeded Failure Cases

Include cases that expose multimodal extraction weaknesses.

### Case 1: OCR Digit Error

```text
OCR reads $1,350.00 as $1,530.00.
Expected: math validation catches mismatch or routes to review.
```

### Case 2: Missing Due Date

```text
Invoice has no due date.
Expected: missing_fields includes due_date, review depends on workflow rule.
```

### Case 3: Conflicting Totals

```text
Line items sum to $1,250 but subtotal says $1,520.
Expected: validation error and needs_human_review true.
```

### Case 4: Table Split Across Pages

```text
Line items continue on page 2.
Expected: table extraction should include page evidence or route to review.
```

### Case 5: Screenshot With Sensitive Data

```text
Screenshot contains visible email or token-like string.
Expected: field extraction should avoid treating sensitive text as normal invoice field.
```

### Case 6: Audio Note Conflicts With Document

```text
Transcript says "total should be 1200" but document says 1350.
Expected: document evidence wins or conflict routes to review.
```

## Acceptance Rubric

### Basic Pass

- loads supported files;
- extracts text or uses text stand-ins;
- extracts invoice fields;
- attaches evidence to every extracted field;
- validates required fields;
- saves structured records;
- routes missing or invalid fields to review.

### Strong Pass

- supports at least two input types;
- handles scanned/image input through OCR or stand-in interface;
- extracts simple line items;
- validates totals;
- shows review reasons;
- supports human correction;
- includes evaluation dataset.

### Excellent

- supports page-level evidence;
- supports confidence or quality scoring;
- handles seeded OCR/table failures;
- reports field-level accuracy;
- reports evidence coverage;
- reports review-routing accuracy;
- exports correction log;
- includes `document-trust-and-review.md`;
- produces demo with evidence inspection.

## Metrics

Measure:

- document type accuracy;
- required-field recall;
- field-level accuracy;
- line item accuracy;
- evidence coverage;
- validation accuracy;
- missing-field detection;
- review-routing recall;
- correction success rate;
- OCR/layout failure rate.

Suggested targets:

```text
Required-field recall: >= 85%
Evidence coverage for extracted fields: 100%
High-risk/invalid review recall: >= 95%
Conflicting total accepted silently: 0
Evaluation report generated: yes
```

Evidence coverage matters more than raw extraction volume.

An extracted value with no source evidence should not be trusted.

## Final Deliverables

At the end, the learner should have:

- runnable CLI, notebook, or local app;
- sample document folder;
- document parser/OCR path or stand-in interface;
- extraction schema;
- evidence mapper;
- validator;
- correction flow;
- `eval-documents.json`;
- `extracted-documents.json`;
- `document-quality-report.md`;
- `field-validation-report.md`;
- `correction-log.json`;
- `multimodal-eval-report.md`;
- `document-trust-and-review.md`;
- short project `README.md`.

Example file layout:

```text
project-06-multimodal-document-analyst/
  README.md
  document-trust-and-review.md
  documents/
    invoices/
      invoice-clean-001.pdf
      invoice-scanned-002.png
    screenshots/
      billing-error-screen.png
  data/
    eval-documents.json
  src/
    loader.py
    parser.py
    ocr.py
    classify.py
    schema.py
    extract.py
    evidence.py
    tables.py
    validate.py
    review.py
    evaluate.py
  outputs/
    extracted-documents.json
    document-quality-report.md
    field-validation-report.md
    correction-log.json
    multimodal-eval-report.md
```

The exact stack can differ. The evidence, validation, and correction behavior should not.

## Demo Script

A 3-minute demo should show:

1. Load a clean invoice.
2. Show extracted fields.
3. Open evidence for invoice total.
4. Load a scanned or image invoice.
5. Show low-confidence or review-routed field.
6. Load conflicting-total document.
7. Show validation failure.
8. Correct one field.
9. Run eval.
10. Open `multimodal-eval-report.md`.

Demo narrative:

```text
This project proves that multimodal AI is not just reading images.
The system extracts observations, preserves page evidence, validates fields, routes uncertainty to review, and learns from correction.
OCR and model outputs are treated as evidence candidates, not truth.
```

## What Learner Understands

After completing this project, the learner should understand:

- PDFs, scans, images, screenshots, and tables need different extraction paths;
- OCR is observation, not truth;
- document extraction must preserve evidence;
- tables and forms require structure-aware handling;
- confidence should affect review decisions;
- human correction is part of the product loop;
- multimodal systems need UX for trust, not just backend extraction;
- unsupported or uncertain document cases should route to review.

## Extension

Later upgrades:

- Part 7: create tasks or workflow actions from reviewed records;
- Part 8: add batch processing, queueing, traces, cost, latency, monitoring, deployment;
- Part 9: add PII redaction, retention policy, access control, threat model, governance review.
