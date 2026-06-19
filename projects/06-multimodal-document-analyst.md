# Project 6: Multimodal Document Analyst

## Build

Build a system that ingests PDFs, images, forms, or scanned documents and turns them into validated structured records with human correction.

This project belongs after Part 6. It proves that learner can handle non-text inputs: OCR, tables, forms, images, audio/realtime notes if useful, trust signals, and correction loops.

## User

Operations, finance, legal, or support teammate who receives messy documents and needs reliable extracted information.

## Product Outcome

User uploads document or image. System returns:

- extracted fields;
- page/source evidence;
- table rows if present;
- confidence/needs-review flags;
- validation errors;
- human correction UI or CLI;
- eval report.

## Example Documents

Use:

```text
docs/
  invoices/invoice-001.pdf
  forms/intake-form-scan.png
  contracts/vendor-agreement.pdf
  screenshots/error-screen.png
```

## Core Features

### 1. Document Intake

Accept:

- native PDF;
- scanned PDF/image;
- table-heavy document;
- screenshot;
- optional audio note attached to document.

### 2. OCR and Parsing

Extract:

- text;
- pages;
- layout blocks;
- tables;
- form fields;
- visual markers.

Track OCR confidence where available.

### 3. Multimodal Model Interface

Model task:

- inspect document evidence;
- extract fields into schema;
- separate observed evidence from inference;
- mark uncertain fields.

### 4. Schema and Validation

Define schema for one workflow, such as invoice intake:

- vendor;
- invoice number;
- date;
- total;
- line items;
- payment terms;
- missing fields;
- needs review.

### 5. Human Correction

User can correct extracted fields. Corrections become eval examples.

### 6. Eval Runner

Test:

- clean PDF;
- scanned PDF;
- table document;
- missing field;
- blurry image;
- conflicting totals;
- screenshot with sensitive data.

## Architecture

```text
document upload
-> parser/OCR
-> layout/table extractor
-> multimodal model extraction
-> schema validator
-> human correction
-> structured record store
-> eval report
```

## Data Model

```json
{
  "document_id": "DOC-001",
  "source_file": "invoice-001.pdf",
  "pages": 2,
  "extracted": {},
  "evidence": [
    {
      "field": "invoice_total",
      "page": 2,
      "excerpt": "$1,240.00",
      "confidence": 0.91
    }
  ],
  "validation_errors": [],
  "needs_review": true
}
```

## Evaluation

Measure:

- field-level accuracy;
- table row accuracy;
- missing-field detection;
- OCR/layout failure rate;
- human-review routing accuracy;
- correction success.

Acceptance:

- every extracted field has evidence;
- uncertain fields route to review;
- invalid totals fail validation;
- corrections can be saved and rerun.

## What Learner Understands

- Multimodal inputs need evidence and confidence.
- OCR is observation, not truth.
- Tables/forms require structure-preserving extraction.
- Human correction is part of product, not fallback shame.

## Extension

Add:

- document type classifier;
- page preview with highlighted evidence;
- audio note transcription;
- redaction preview;
- batch processing;
- active learning queue.
