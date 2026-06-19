# Chapter 10: Data Ingestion, Parsing, and Cleaning

Chapter 9 identified what information the system needs. This chapter asks how that information enters system.

Bad AI often starts with bad ingestion. If documents are broken, duplicated, stale, or misparsed, retrieval and model answers fail later.

## Real Problem

You ingest company docs:

- PDFs;
- markdown files;
- help center pages;
- CSV exports;
- emails;
- spreadsheets.

Search results look strange:

- table rows lose columns;
- page footers become content;
- duplicate docs appear;
- old policy outranks new policy;
- PDFs extract words in wrong order.

Model did not fail. Ingestion failed.

## Bad Default Solution

Bad ingestion:

- scrape everything;
- store raw text only;
- ignore metadata;
- ignore duplicates;
- ignore parse errors;
- do not track source version;
- manually fix failures later.

This creates hidden data debt.

## First Principles

Ingestion converts external information into system-ready records.

Pipeline:

```text
source -> fetch -> parse -> clean -> normalize -> enrich metadata -> validate -> store -> index
```

Parsing extracts content.

Cleaning removes noise.

Normalization makes records consistent.

Validation catches broken inputs.

Metadata preserves meaning.

## System Decision

For each source, define:

```text
Source:
Format:
Owner:
Refresh schedule:
Parser:
Cleaning rules:
Metadata:
Validation:
Failure handling:
Storage target:
Index target:
```

Use different parsers for different formats:

- markdown;
- HTML;
- PDF;
- CSV;
- spreadsheet;
- image/OCR;
- email.

## Minimal Build

Create `ingestion-pipeline.md`:

```text
Sources:
- name:
  format:
  parser:
  cleaning:
  metadata:
  validation:
  failure path:

Pipeline stages:
1. fetch
2. parse
3. clean
4. normalize
5. validate
6. store
7. index
```

## Failure Modes

Ingestion fails when:

- text extraction loses layout;
- tables lose headers;
- duplicate docs remain;
- stale docs not removed;
- parser errors ignored;
- source URL missing;
- chunk text lacks title/headings;
- images contain text but OCR missing.

## Evaluation

Create ingestion test set:

- 5 markdown docs;
- 5 HTML pages;
- 5 PDFs;
- 3 tables/spreadsheets;
- 2 malformed files.

Measure:

- parse success rate;
- metadata completeness;
- duplicate rate;
- stale-source detection;
- human inspection pass.

## Improvement

Improve by:

- adding parser per file type;
- preserving headings/tables;
- storing raw + cleaned text;
- adding content hash;
- deduplicating;
- logging parse errors;
- marking low-confidence extraction.

## Proof Artifact

Create `ingestion-pipeline.md` and sample parsed records.

## Decision Checklist

- [ ] Sources are listed.
- [ ] Parser chosen per format.
- [ ] Cleaning rules exist.
- [ ] Metadata extracted.
- [ ] Parse failures logged.
- [ ] Raw and normalized records stored.
- [ ] Duplicate/stale docs handled.

## Active Recall

1. Why does ingestion quality affect model quality?
2. What is difference between parsing and cleaning?
3. Why store metadata during ingestion?
4. Why preserve raw text and normalized text?
5. How do PDFs break naive pipelines?

## Next

After ingestion, system needs contracts. Next chapter covers schemas, data contracts, and structured outputs.

