# Chapter 26: Document AI: OCR, Tables, Forms, and PDFs

Part 5 built text-based retrieval and grounded knowledge products.

Part 6 moves into multimodal and human interfaces.

Real company knowledge often does not begin as clean text. It arrives as PDFs, scans, invoices, forms, tables, contracts, screenshots, handwritten notes, stamps, signatures, and messy attachments.

Document AI is the bridge between messy business documents and reliable AI systems.

## Concept Overview

Document AI turns document files into usable system records.

It is not just text extraction.

A document may contain:

- selectable text;
- scanned images;
- layout;
- tables;
- form fields;
- checkboxes;
- handwriting;
- signatures;
- stamps;
- page order;
- headers and footers;
- visual marks;
- metadata.

Reliable Document AI extracts information while preserving evidence, structure, confidence, and review paths.

The basic flow:

```text
document
-> classify document type
-> parse or OCR
-> detect layout blocks
-> extract fields and tables
-> attach evidence and confidence
-> validate against schema
-> route uncertain fields to human review
-> store structured record
```

The first principle:

> Do not treat OCR as truth. Treat OCR as observation with possible errors.

Document AI systems must know what was seen, what was inferred, what is uncertain, and what needs review.

## Why It Exists

Many business workflows depend on documents.

Examples:

- vendor invoices;
- purchase orders;
- insurance forms;
- tax forms;
- contracts;
- resumes;
- bank statements;
- medical forms;
- shipping documents;
- handwritten applications;
- scanned approvals.

Consider a finance team receiving vendor invoices as PDFs.

They need:

- vendor name;
- invoice number;
- invoice date;
- line items;
- totals;
- tax;
- payment terms;
- purchase order number;
- approval status.

Some PDFs contain selectable text. Some are scans. Tables wrap across pages. Stamps and signatures matter. A naive text extractor loses structure, and the model guesses.

Bad default:

```text
extract all PDF text
-> ask model to summarize it
```

This fails because:

- OCR errors become facts;
- table columns lose alignment;
- checkboxes disappear;
- page context is lost;
- signatures and stamps are ignored;
- fields are extracted without confidence;
- validation is missing;
- human correction is not captured.

This chapter exists because documents are not plain text. They are structured visual artifacts.

## Mental Model

Think of a document as layered evidence.

The visible page is one layer. Text is another. Layout is another. Tables, marks, fields, stamps, and signatures add meaning. The system must preserve enough of these layers to support trustworthy extraction.

The mental model:

```text
document image/text
-> observations
-> candidate fields
-> evidence spans or regions
-> validation
-> human correction
-> structured business record
```

A model can help interpret variation, but the system must preserve evidence and check output.

For every extracted field, ask:

```text
Where did this value come from?
How confident is the extraction?
Does it satisfy the schema?
Does it match business rules?
Does a human need to review it?
How will corrections be stored?
```

Document AI is not successful when it produces a fluent summary. It is successful when it creates a reliable record the business workflow can use.

## Internal Mechanics

Document AI begins by identifying the document type.

### Document Type

Different document types need different strategies.

Examples:

| Document Type | Strategy |
|---|---|
| Native PDF text | Text extraction plus layout preservation |
| Scanned PDF | OCR plus confidence and page regions |
| Forms | Field detection plus schema validation |
| Tables | Table parser plus row/column checks |
| Contracts | Section-aware parsing and clause extraction |
| Invoices | Field extraction, line-item parsing, validation |
| Receipts | OCR, totals validation, merchant/date extraction |
| Handwritten forms | OCR/vision plus human review threshold |

Use deterministic parsers when layout is stable.

Use models when document variation is high or language interpretation matters.

Use human review when confidence is low or business impact is high.

### OCR

OCR converts visual text into machine-readable text.

OCR can fail through:

- blurry scans;
- rotated pages;
- low contrast;
- handwriting;
- small fonts;
- stamps over text;
- poor image quality;
- unusual layouts;
- multilingual text.

OCR output should include:

```text
text
page number
bounding box
confidence
reading order
language
warnings
```

Do not store only plain text if page position matters.

### Layout

Layout explains relationships.

Examples:

- label near value;
- table header above column;
- checkbox beside option;
- signature near approval field;
- stamp over invoice status;
- section heading above clause.

Layout representation may include:

```text
pages
blocks
paragraphs
lines
tokens
tables
form fields
bounding boxes
reading order
```

Layout matters because the same text can mean different things depending on position.

### Tables

Tables are fragile.

A table is not just text lines. It is rows, columns, headers, units, page breaks, and sometimes merged cells.

Invoice line item table:

```text
Description | Quantity | Unit Price | Tax | Total
```

If headers are lost, the row values become ambiguous.

Table extraction should check:

- headers detected;
- row count;
- column count;
- totals add up;
- page continuation;
- currency consistency;
- missing cells;
- duplicate rows.

### Forms

Forms often contain labels, fields, checkboxes, radio buttons, signatures, and dates.

Extraction should map document observations to a schema:

```text
field: vendor_name
value: Acme Systems
evidence: page 1, region x/y
confidence: 0.93
validation: non-empty
review_required: no
```

Checkboxes should be extracted as structured values, not ignored as visual decoration.

### Schema and Validation

Document extraction should produce structured records.

Example invoice schema:

```text
vendor_name
invoice_number
invoice_date
purchase_order_number
line_items[]
subtotal
tax
total
currency
payment_terms
```

Validation rules:

```text
total = subtotal + tax
invoice_date must be valid date
currency must be allowed value
line item totals must match quantity * unit price
invoice_number required
duplicate invoice_number flagged
```

The model should not be the only validator. Business rules belong in code.

### Confidence and Review

Confidence should affect workflow.

Examples:

```text
confidence >= 0.95 and validation passes -> auto-accept
confidence 0.75-0.95 -> accept with warning or sample review
confidence < 0.75 -> human review
validation failure -> human review
high-value invoice -> human review regardless of confidence
```

Human review is not a patch. It is part of the system design.

### Correction Storage

Human corrections should be stored.

Correction record:

```text
document_id
field_name
original_value
corrected_value
reviewer
reason
timestamp
evidence_region
```

Corrections can improve:

- eval sets;
- parser rules;
- prompts;
- extraction models;
- business validation rules.

### Downstream Use

Extracted document records may feed:

- approval workflow;
- payment processing;
- search/retrieval;
- analytics;
- compliance review;
- customer support;
- audit logs.

The downstream use decides how strict extraction must be.

An invoice payment workflow needs more validation than a rough document summary.

## Real-World Example

Use case:

```text
Vendor invoice processing
```

Input:

```text
multi-page invoice PDF
```

Target output:

```json
{
  "vendor_name": "Acme Systems",
  "invoice_number": "INV-2026-1048",
  "invoice_date": "2026-06-01",
  "purchase_order_number": "PO-8821",
  "line_items": [
    {
      "description": "Cloud support package",
      "quantity": 1,
      "unit_price": 1200.00,
      "total": 1200.00
    }
  ],
  "subtotal": 1200.00,
  "tax": 96.00,
  "total": 1296.00,
  "currency": "USD",
  "review_required": false
}
```

Strong pipeline:

```text
classify document as invoice
-> OCR or extract text depending on PDF type
-> detect layout and tables
-> extract invoice fields
-> extract line items
-> attach page/region evidence
-> validate totals and dates
-> check duplicate invoice number
-> route low-confidence fields to review
-> store structured record and correction history
```

If OCR reads:

```text
INV-2026-104B
```

but validation or human review corrects it to:

```text
INV-2026-1048
```

the correction should be stored and used for future evaluation.

## Common Mistakes and Failure Modes

Mistake 1: Treating PDFs as plain text

PDFs can contain layout, tables, images, scanned pages, and visual marks. Plain text may lose meaning.

Mistake 2: Hiding OCR confidence

Low-confidence OCR should not silently become a business fact.

Mistake 3: Flattening tables

Tables need rows, columns, headers, units, and page continuity.

Mistake 4: Extracting fields without evidence

Every important field should trace back to a page, region, line, or source span.

Mistake 5: No validation rules

Dates, totals, currencies, required fields, duplicate IDs, and business constraints should be checked.

Mistake 6: Low-confidence fields skip review

Review rules should be explicit and tied to confidence, validation, and risk.

Mistake 7: Corrections disappear

Human corrections should become data for evals and system improvement.

Mistake 8: One pipeline for every document type

Invoices, contracts, forms, receipts, and scanned notes need different extraction strategies.

## System Design Decisions

Define document types:

```text
What document classes exist?
How are they detected?
What parser or model handles each?
Which fields matter?
```

Define layout strategy:

```text
Is plain text enough?
Do tables matter?
Do forms matter?
Do signatures or stamps matter?
Are page regions needed as evidence?
```

Define extraction schema:

```text
Which fields are required?
Which fields are optional?
What types and formats are expected?
What evidence must attach to each field?
```

Define validation:

```text
What business rules check extracted values?
Which validation failures block automation?
Which failures route to human review?
```

Define confidence:

```text
How is OCR confidence used?
How is model confidence handled?
What confidence threshold requires review?
Which fields require review regardless of confidence?
```

Define correction loop:

```text
How are human corrections stored?
Who reviews them?
How do corrections update evals?
How do corrections improve parser/model behavior?
```

Define downstream use:

```text
Will extracted data trigger payment?
Will it update a system of record?
Will it only support search?
Will it require auditability?
```

The higher the impact, the stronger the validation and review path.

## Build Artifact

Create `document-ai-plan.md` and `document-extraction-eval.md`.

Use this template for `document-ai-plan.md`:

```text
Document AI Plan

Workflow:
Business goal:

Document Types:
- type:
  examples:
  detection method:
  parser/OCR method:
  model role:

Extraction Schema:
- field:
  type:
  required: yes/no
  evidence required:
  validation rule:
  review threshold:

Layout Strategy:
- tables:
- forms:
- signatures/stamps:
- page regions:
- reading order:

Validation:
- rule:
  failure behavior:

Human Review:
- trigger:
  reviewer:
  action:

Correction Storage:
- stored fields:
- used for eval: yes/no

Downstream Use:
- system:
  risk:
  required checks:
```

Use this template for `document-extraction-eval.md`:

```text
Document Extraction Eval

Document set version:
Parser/OCR version:
Model/interface version:

Eval Documents:
- id:
  type:
  condition:
  expected fields:
  expected tables:
  review expected:

Metrics:
- field-level accuracy:
- table row accuracy:
- missing-field detection:
- invalid extraction rate:
- evidence accuracy:
- human-review routing accuracy:
- correction rate:
```

Example eval set:

```text
clean native PDF invoice
scanned invoice
multi-page invoice table
missing invoice number
ambiguous total
low-quality image
handwritten note
duplicate invoice
```

The artifact is complete when another engineer can understand how documents become structured records, how uncertainty is handled, and how extraction quality is measured.

## Active Recall Questions

1. Why is Document AI not just text extraction?
2. Why should OCR be treated as observation instead of truth?
3. What document layers can carry meaning?
4. Why do tables require special extraction logic?
5. What evidence should attach to extracted fields?
6. Why should validation happen outside the model?
7. When should human review be required?
8. How should human corrections improve the system?

## Summary

Document AI turns messy documents into reliable structured records. It must handle OCR, layout, tables, forms, page order, visual marks, confidence, validation, evidence, and human correction.

The goal is not a nice summary. The goal is a trustworthy record that downstream systems can use, audit, and improve.

The most important rule:

> Extracted document fields should carry evidence, confidence, validation, and a review path.

## Preview of Next Chapter

Chapter 26 handled documents.

Chapter 27 expands to broader vision, image, and multimodal inputs.

Next, you will learn how to work with screenshots, diagrams, charts, photos, and mixed text-image tasks. This matters because visual AI systems must separate what is observed from what is inferred, handle privacy, and avoid turning uncertain perception into unsupported action.
