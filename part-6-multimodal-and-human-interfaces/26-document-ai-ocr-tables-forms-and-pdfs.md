# Chapter 26: Document AI: OCR, Tables, Forms, and PDFs

Part 5 taught retrieval and grounded answers over text. Real company knowledge often does not begin as clean Markdown. It arrives as PDFs, scans, invoices, forms, tables, contracts, screenshots, and messy attachments.

Document AI is the bridge between messy business documents and reliable AI systems.

## Real Problem

A finance team receives vendor invoices as PDFs.

They need:

- vendor name;
- invoice number;
- line items;
- totals;
- tax;
- payment terms;
- approval status.

Some PDFs contain selectable text. Some are scans. Tables wrap across pages. Stamps and signatures matter. A naive text extractor loses structure, and the model guesses.

## Bad Default Solution

Bad solution:

```text
Extract all PDF text and ask model to summarize it.
```

Problems:

- OCR errors become facts;
- table columns lose alignment;
- checkboxes disappear;
- page context is lost;
- signatures and stamps are ignored;
- fields are extracted without confidence;
- human correction is not captured.

Document AI is not just text extraction. It is layout, evidence, schema, confidence, and review.

## First Principles

Documents contain multiple information layers:

- text;
- layout;
- tables;
- form fields;
- images;
- handwriting;
- page order;
- visual marks;
- metadata.

Reliable extraction needs:

```text
document -> parse/OCR -> layout blocks -> field candidates -> validation -> human review -> structured record
```

Do not treat OCR as truth. Treat OCR as observation with errors.

## System Decision

Choose extraction strategy:

| Document Type | Strategy |
|---|---|
| native PDF text | text extraction + layout preservation |
| scanned PDF | OCR + confidence |
| forms | field detection + schema validation |
| tables | table parser + row/column checks |
| contracts | section-aware parsing |
| receipts/invoices | template + model extraction + validation |

Use model when document variation is high. Use deterministic parsers when layout is stable.

## Minimal Build

Create `document-ai-plan.md`:

```text
Document types:
Fields to extract:
Parser/OCR method:
Layout representation:
Table strategy:
Validation rules:
Confidence rule:
Human review rule:
Correction storage:
Eval set:
```

## Failure Modes

Document AI fails when:

- OCR confidence is hidden;
- tables are flattened into nonsense;
- model extracts fields not present;
- page numbers are lost;
- validation is missing;
- low-confidence fields skip review;
- corrected fields do not become eval data.

## Evaluation

Create eval documents:

- clean native PDF;
- scanned PDF;
- multi-page table;
- missing field;
- ambiguous field;
- low-quality image;
- handwritten note;
- duplicate invoice.

Measure:

- field-level accuracy;
- table row accuracy;
- missing-field detection;
- invalid extraction rate;
- human-review routing accuracy.

## Proof Artifact

Create `document-ai-plan.md` and `document-extraction-eval.md`.

## Next

Next chapter expands from documents to general vision and image inputs.
