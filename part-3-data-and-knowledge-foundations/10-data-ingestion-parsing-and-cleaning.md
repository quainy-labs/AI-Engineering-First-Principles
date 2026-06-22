# Chapter 10: Data Ingestion, Parsing, and Cleaning

Chapter 9 classified the information an AI system needs:

```text
structured data
documents
state
audit records
```

This chapter asks how that information enters the system in a usable form.

An AI system does not reason over "company knowledge" in the abstract. It reasons over records, files, extracted text, metadata, chunks, tables, fields, and state. If those are broken, the model will appear broken even when the real failure happened earlier.

Bad AI often starts as bad ingestion.

## Concept Overview

Data ingestion is the process of bringing external information into the AI system and turning it into reliable internal records.

The basic pipeline:

```text
source
-> fetch
-> parse
-> clean
-> normalize
-> enrich metadata
-> validate
-> store
-> index
```

Each stage has a job.

```text
Fetch
Get the source content.

Parse
Extract structure and content from the source format.

Clean
Remove noise that should not become knowledge.

Normalize
Make records consistent across sources.

Enrich metadata
Preserve source, ownership, permissions, version, and meaning.

Validate
Catch broken or incomplete records.

Store
Save raw and processed versions.

Index
Prepare the information for search, retrieval, or downstream use.
```

The goal is not to "dump text into the model." The goal is to preserve enough meaning that later systems can retrieve, reason, cite, filter, evaluate, and update correctly.

## Why It Exists

Most organizations already have information, but not in a form AI systems can safely use.

Knowledge may live in:

- PDFs;
- markdown files;
- help-center pages;
- CSV exports;
- spreadsheets;
- emails;
- scanned images;
- internal wiki pages;
- support tickets;
- product docs;
- database exports;
- contracts and forms.

These sources are messy.

Common problems:

- PDF text extracts in the wrong order;
- tables lose headers;
- footers become repeated content;
- old pages remain visible;
- duplicate documents appear from multiple sources;
- scanned files need OCR;
- HTML navigation becomes document text;
- emails include quoted threads and signatures;
- spreadsheets hide meaning in sheet names and formulas;
- documents lack owner, version, or permission metadata.

If ingestion ignores these problems, downstream AI fails.

Search will retrieve noise. The model will summarize outdated policy. Evaluation will test against corrupted examples. Users will blame the model, but the system fed it broken knowledge.

The first principle:

> Retrieval quality cannot exceed ingestion quality.

## Mental Model

Think of ingestion as a factory that turns external information into system-grade knowledge.

Raw sources are not yet system-grade knowledge. A PDF is not automatically a useful retrieval record. A spreadsheet is not automatically structured data. A web page is not automatically clean policy content.

The mental model:

```text
raw source
-> extracted content
-> cleaned content
-> normalized record
-> metadata-rich record
-> validated knowledge object
-> searchable or queryable asset
```

At every step, ask:

```text
What meaning could be lost here?
What noise could be introduced here?
What metadata must survive?
What failure should be visible?
What downstream system depends on this output?
```

Good ingestion is not invisible. It creates records that explain where information came from, how it was processed, what version it represents, and whether the system trusts it.

Bad ingestion hides uncertainty. Good ingestion makes uncertainty inspectable.

## Internal Mechanics

Start ingestion design source by source.

Each source should have an explicit ingestion plan:

```text
source name
source owner
format
access method
refresh schedule
parser
cleaning rules
metadata required
validation rules
storage target
index target
failure path
```

### Fetch

Fetching gets content from the source.

Examples:

- read uploaded file;
- call source API;
- crawl help center page;
- sync documents from cloud storage;
- export database rows;
- receive webhook event;
- load batch files from object storage.

Fetch should preserve source identity:

```text
source_system
source_uri
source_id
fetched_at
source_version
content_hash
owner
workspace_id
permission labels
```

Without source identity, the system cannot refresh, deduplicate, delete, cite, or debug the content later.

### Parse

Parsing extracts content and structure from a format.

Different formats require different parsers:

- markdown parser for markdown;
- HTML parser for web pages;
- PDF parser for text PDFs;
- OCR for scanned PDFs or images;
- CSV parser for delimited data;
- spreadsheet parser for multi-sheet files;
- email parser for messages and threads;
- document AI parser for forms, tables, and layout-heavy documents.

Parsing should preserve structure where structure matters.

For documents, preserve:

- title;
- headings;
- sections;
- paragraphs;
- tables;
- lists;
- page numbers;
- captions;
- links;
- footnotes when relevant.

For tables, preserve:

- column names;
- row boundaries;
- units;
- sheet name;
- merged-cell meaning when possible;
- formulas or calculated fields when relevant.

For emails, preserve:

- sender;
- recipients;
- timestamp;
- subject;
- current message;
- quoted history if needed;
- attachments;
- signatures if useful or removable.

The parser should not pretend success when extraction is weak. It should record confidence, warnings, or failure reasons.

### Clean

Cleaning removes noise that should not become knowledge.

Common cleaning rules:

- remove repeated headers and footers;
- remove navigation menus;
- remove cookie banners;
- remove page numbers when they add no meaning;
- remove boilerplate legal text when irrelevant;
- strip broken OCR artifacts;
- remove duplicate email signatures;
- separate quoted email history from current message;
- remove empty rows and columns;
- fix obvious encoding issues.

Cleaning should be conservative. Removing too much can destroy meaning.

Example:

```text
Bad cleaning:
remove every repeated line

Problem:
some repeated lines may be table headers or legal clauses that matter.
```

Cleaning rules should be testable and source-specific.

### Normalize

Normalization makes records consistent.

Examples:

- convert dates to one format;
- standardize currency fields;
- normalize casing for enum-like labels;
- map source-specific labels to internal labels;
- split full names into fields when needed;
- represent missing values consistently;
- convert HTML tables into structured rows;
- convert document sections into common record shape.

Normalization helps downstream systems avoid special cases for every source.

### Enrich Metadata

Metadata gives context to content.

Useful metadata:

```text
document_id
source_system
source_uri
title
author
owner
workspace_id
created_at
updated_at
fetched_at
version
content_hash
language
format
sensitivity_level
permission labels
authority_level
parser_version
ingestion_job_id
parse_confidence
```

Metadata is not decoration. It controls:

- permission filtering;
- freshness checks;
- deduplication;
- citation;
- deletion propagation;
- ranking;
- evaluation;
- debugging;
- governance.

### Validate

Validation checks whether the ingested record is usable.

Examples:

- required metadata exists;
- extracted text is not empty;
- file type is supported;
- table has headers;
- document has source URI;
- content hash exists;
- language is detected;
- parse confidence is above threshold;
- permission labels exist;
- record does not duplicate an existing source version.

Validation should route bad records:

```text
valid -> store and index
warning -> store, index carefully, mark low confidence
invalid -> quarantine or manual review
```

### Store Raw and Processed Versions

Store raw input when allowed:

- original file;
- original HTML;
- original CSV;
- original email;
- original API payload.

Store processed output:

- cleaned text;
- structured records;
- extracted tables;
- normalized fields;
- metadata;
- parse warnings;
- validation result.

Raw storage helps with reprocessing when parsers improve. Processed storage helps the system use the information today.

### Index

Indexing prepares information for retrieval or lookup.

Different outputs may go to different stores:

- relational database for structured records;
- object storage for raw files;
- search index for keyword search;
- vector index for semantic retrieval;
- document store for parsed content;
- analytics store for aggregate reporting.

Indexing should preserve metadata. A chunk without source, owner, permission, version, and title becomes dangerous later.

## Real-World Example

Imagine building a company knowledge assistant.

Sources:

```text
help center articles
PDF policy documents
internal wiki pages
product release notes
CSV export of product limits
support macros
```

Bad ingestion:

```text
scrape everything
-> extract plain text
-> split every 1,000 characters
-> embed chunks
-> search
```

Failure appears later:

- old policies still answer questions;
- duplicate pages crowd search results;
- PDF tables lose column meaning;
- help-center navigation appears in answers;
- product limits from CSV are treated like paragraph text;
- restricted internal docs appear for all users;
- citations point to missing or unclear sources.

Better ingestion:

```text
source plan per source
-> fetch with source id and version
-> parse with format-specific parser
-> clean source-specific noise
-> preserve headings and tables
-> normalize records
-> attach metadata and permissions
-> validate content
-> store raw and processed versions
-> index with source metadata
```

Example parsed record:

```text
record_id: rec_123
source_system: help_center
source_uri: https://example.com/refunds
title: Refund Policy
section_heading: Enterprise Plan Exceptions
content: Enterprise customers may request exception review...
owner: support_ops
workspace_id: company
authority_level: published_policy
updated_at: 2026-04-12
content_hash: abc123
permission_labels: employee
parser_version: html_v2
parse_confidence: high
```

Now later systems can:

- retrieve by meaning;
- filter by permission;
- prefer current policy;
- cite source;
- remove stale versions;
- debug poor answers;
- rerun ingestion after parser improvement.

## Common Mistakes and Failure Modes

Mistake 1: Storing only raw text

Raw text alone loses structure, source identity, permissions, and version. Later retrieval has little context.

Mistake 2: Using one parser for every format

PDFs, HTML pages, spreadsheets, markdown, and emails carry meaning differently. One naive parser will destroy important structure.

Mistake 3: Ignoring tables

Tables often contain exact facts. If table rows become random text, the model may misunderstand relationships between columns and values.

Mistake 4: Losing metadata during chunking

Chunks must inherit document metadata. A chunk without title, source, version, permission, and owner is hard to trust.

Mistake 5: No deduplication

Duplicate content can dominate retrieval and make the system overconfident in repeated but outdated information.

Mistake 6: No stale-source handling

If old documents remain indexed after update or deletion, the system may answer from knowledge that should no longer exist.

Mistake 7: Silent parse failures

If a parser extracts empty or corrupted text and the system indexes it anyway, retrieval quality drops without obvious errors.

Mistake 8: Cleaning away meaning

Aggressive cleaning can remove headings, table headers, legal clauses, or exceptions that matter.

## System Design Decisions

Design ingestion around the source and the downstream use.

For each source, decide:

```text
What is the source format?
Who owns it?
How is it accessed?
How often does it change?
What parser is appropriate?
What structure must be preserved?
What noise should be removed?
What metadata is required?
What validation makes it trustworthy?
Where are raw and processed versions stored?
Where is it indexed?
How are updates and deletes handled?
```

Choose parser strategy:

```text
simple text parser
HTML/document parser
PDF layout parser
OCR
document AI
spreadsheet parser
custom source connector
```

Choose freshness strategy:

```text
one-time import
scheduled sync
webhook-driven update
manual reingestion
versioned source tracking
delete propagation
```

Choose failure behavior:

```text
fail whole batch
skip bad record
quarantine bad source
mark low confidence
send to human review
retry later
```

Choose storage strategy:

```text
raw file storage
parsed record store
structured database
search index
vector index
parse error log
ingestion job table
```

The key design question:

> What would break later if this ingestion step loses meaning?

## Build Artifact

Create `ingestion-pipeline.md` for one AI workflow.

Choose a workflow such as:

- company knowledge assistant;
- support refund assistant;
- invoice extraction system;
- HR policy assistant;
- multimodal document analyst;
- operations action assistant.

Use this template:

```text
Ingestion Pipeline

Workflow:
User goal:
Downstream use:

Sources:
- name:
  format:
  owner:
  access method:
  refresh schedule:
  permissions:

Pipeline Stages:
- stage: fetch
  input:
  output:
  failure path:

- stage: parse
  parser:
  structure to preserve:
  output:
  failure path:

- stage: clean
  noise to remove:
  meaning to preserve:
  output:

- stage: normalize
  rules:
  output shape:

- stage: enrich metadata
  required metadata:
  optional metadata:

- stage: validate
  checks:
  invalid behavior:
  warning behavior:

- stage: store
  raw storage:
  processed storage:

- stage: index
  search index:
  vector index:
  metadata copied:

Update and Delete Rules:
- update detection:
- stale version behavior:
- delete propagation:

Quality Tests:
- source:
  expected result:
  failure condition:
```

Example:

```text
Workflow: Company knowledge assistant
Source: refund policy help article
Format: HTML
Owner: support operations
Refresh: daily

Parser:
HTML parser that extracts title, headings, body, tables, and links

Cleaning:
remove navigation, footer, cookie banner, related-article sidebar
preserve headings, policy exceptions, dates, and tables

Metadata:
source_uri, title, owner, updated_at, authority_level, permission_labels, content_hash

Validation:
title exists
body length above threshold
updated_at exists
permission labels exist
content_hash exists

Delete rule:
if source disappears, mark document archived and remove active chunks from retrieval
```

The artifact is complete when another engineer can ingest the source, inspect failures, and understand how the processed record will support retrieval or structured use later.

## Active Recall Questions

1. Why does ingestion quality limit retrieval and model quality?
2. What is the difference between parsing, cleaning, normalization, and validation?
3. Why should ingestion preserve raw and processed versions?
4. What metadata should survive from source document to retrieval chunk?
5. Why do tables require special handling?
6. What can go wrong when duplicate documents remain indexed?
7. How should the system handle stale or deleted sources?
8. Why should parse failures be visible instead of silently indexed?

## Summary

Ingestion turns external information into system-grade knowledge.

Fetching gets content. Parsing extracts structure. Cleaning removes noise. Normalization makes records consistent. Metadata preserves meaning, ownership, permission, version, and freshness. Validation catches broken inputs. Storage and indexing make information usable by later systems.

The most important rule:

> Do not treat ingestion as file loading. Treat it as knowledge preparation.

## Preview of Next Chapter

Chapter 10 showed how information enters the system.

Chapter 11 asks how information moves between components reliably.

Next, you will learn schemas, contracts, and structured outputs. This matters because once ingestion produces records, every downstream component needs a clear shape: document metadata, extracted fields, eval examples, tool arguments, model outputs, and trace events.
