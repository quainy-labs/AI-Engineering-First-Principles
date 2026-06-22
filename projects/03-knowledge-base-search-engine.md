# Project 3: Knowledge Base Search Engine

## Concept

Reliable AI systems need reliable knowledge foundations.

Before a system can answer from documents, it must know what documents exist, whether they are trustworthy, how they are structured, how fresh they are, who owns them, and whether search can find the right source.

This project builds a small searchable knowledge base using document ingestion, metadata, quality checks, lineage, keyword search, filters, and retrieval evaluation.

This is not a RAG chatbot. It is not an embedding project. It is not a vector database project.

The learner should finish with a runnable CLI, notebook, or local app that can ingest a small document collection, build a simple search index, show ranked results, and evaluate whether the right documents are being found.

## What You Will Build

Build a Knowledge Base Search Engine.

The system accepts a folder of markdown or text documents and produces:

- document inventory;
- parsed metadata;
- quality report;
- lineage records;
- keyword search index;
- ranked search results;
- metadata filters;
- source detail view;
- retrieval evaluation report.

The product answers one question:

```text
Can users find the right trusted source before we build AI answers on top?
```

## Chapters Practiced

This project proves understanding from:

- Chapter 9: data, documents, and state;
- Chapter 10: data ingestion, parsing, and cleaning;
- Chapter 11: schemas, contracts, and structured outputs;
- Chapter 12: data quality, lineage, and versioning;
- Chapter 13: search, ranking, and information architecture.

It may reuse Part 1 and Part 2 thinking, but it should not require future course concepts.

Do not require embeddings, semantic search, RAG generation, model routing, agents, multimodal parsing, production observability, or governance review here.

Later parts can improve the same project:

- Part 4 can add model-assisted query understanding;
- Part 5 can add embeddings, chunking strategy, RAG, grounding, and retrieval evaluation depth;
- Part 6 can add PDFs, OCR, tables, and multimodal document inputs;
- Part 8 can add observability, caching, and deployment;
- Part 9 can add access control, poisoning review, privacy, and governance.

## User Story

Primary user:

```text
Support teammate, researcher, student, operator, or founder.
```

User story:

```text
As a support teammate,
I want to search approved internal documents,
so I can find the right policy or procedure quickly and avoid using stale or untrusted information.
```

The user wants more than search results. They need trust signals:

```text
Who owns this document?
Is it approved?
When was it updated?
Why did it match?
Is there a newer source?
```

## System Requirements

### Input

Accept a local document folder:

```text
docs/
  billing/
  access/
  product/
  operations/
```

Supported formats for this project:

- `.md`;
- `.txt`;
- optional `.csv` index file for metadata.

Do not require PDF parsing here. PDF/OCR belongs to Part 6.

### Required Document Metadata

Each markdown document should include frontmatter:

```yaml
title: Refund Policy
owner: Support Ops
audience: support
authority: approved
last_updated: 2026-01-10
tags: [billing, refunds]
```

Required fields:

```text
title
owner
audience
authority
last_updated
tags
```

Allowed authority values:

```text
approved
draft
deprecated
unknown
```

### Output

The system should produce:

- `document-inventory.json`;
- `quality-report.md`;
- searchable index file or in-memory index;
- search results with snippets;
- source metadata view;
- `retrieval-eval-report.md`.

### Interface

Choose one implementation path:

```text
CLI:
kb ingest docs/
kb search "reset api key"
kb search "refund policy" --authority approved
kb quality-report
kb eval eval-queries.json

Notebook:
Run ingestion, search, quality, and eval cells.

Local app:
Search box, filters, results, source details.
```

CLI or notebook is enough for this project.

## Starter Data

Create a `docs/` folder with 20-40 documents.

Recommended structure:

```text
docs/
  billing/
    refund-policy.md
    invoice-admins.md
    duplicate-charge.md
    refund-policy-old.md
  access/
    api-key-reset.md
    password-reset.md
    account-owner-change.md
    production-log-access.md
  product/
    plan-limits.md
    release-notes-2026-01.md
    deprecated-export-guide.md
  operations/
    escalation-rules.md
    support-handoff.md
    security-incident-routing.md
```

Example document:

```markdown
---
title: API Key Reset Guide
owner: Platform Support
audience: support
authority: approved
last_updated: 2026-01-14
tags: [access, api, reset]
---

# API Key Reset Guide

Support agents should send the official reset link when a customer cannot reset an API key.

## Reset Steps

1. Confirm the requester is an admin.
2. Send the API key reset guide.
3. Ask the customer to confirm access.

## Do Not

Do not generate or reveal API keys in support replies.
```

Seed the collection with realistic problems:

```text
Good documents:
- approved API key reset guide;
- approved refund policy;
- approved escalation rules.

Stale documents:
- old refund policy;
- deprecated export guide.

Low-quality documents:
- missing owner;
- missing last_updated;
- empty body;
- duplicate title.

Distractor documents:
- release notes that mention billing but are not policy;
- product plan limit page that mentions API keys but not reset process.
```

## MVP Milestone

Build the simplest useful version first.

MVP must:

1. Load markdown or text documents from a folder.
2. Parse frontmatter metadata.
3. Validate required metadata.
4. Build document inventory.
5. Create simple keyword search.
6. Return ranked results with title, path, snippet, and score.
7. Show source metadata.

MVP flow:

```text
docs folder
  -> document loader
  -> metadata parser
  -> quality checker
  -> keyword index
  -> search query
  -> ranked results
```

Do not use embeddings or a language model in the MVP.

The first principle is to prove that documents, metadata, and search quality are already engineering problems before generation begins.

## V1 Milestone

After MVP works, improve search and knowledge quality.

V1 should add:

- metadata filters;
- stale document warnings;
- duplicate detection;
- content hash;
- parser version;
- ingested timestamp;
- section-level snippets;
- evaluation queries;
- retrieval quality report.

Filter examples:

```bash
kb search "refund policy" --authority approved
kb search "api key" --tag access
kb search "escalation" --audience support
```

Quality report should show:

- missing metadata;
- stale documents;
- duplicate titles;
- empty documents;
- deprecated documents;
- unknown authority;
- documents excluded from search.

## Knowledge Boundary Milestone

Add a short file named `knowledge-boundary.md`.

It should explain:

- which document types are included;
- which document types are excluded;
- which metadata fields are required;
- what counts as authoritative;
- how stale documents are handled;
- how search results should be interpreted;
- why this is not a chatbot yet.

This keeps the project aligned with Part 3: knowledge foundations before AI answer generation.

## Core Components

### 1. Document Loader

Responsibilities:

- walk the `docs/` folder;
- read `.md` and `.txt` files;
- preserve source path;
- record file size;
- report unreadable files.

Expected behavior:

```text
If 30 documents exist and 2 cannot be parsed,
the system should ingest 28 and report 2 errors.
```

### 2. Metadata Parser

For markdown files, parse frontmatter.

For text files, allow metadata from a sidecar file or mark missing metadata.

Parsed fields:

```text
title
owner
audience
authority
last_updated
tags
source_path
```

Metadata should become structured data, not loose strings.

### 3. Text Normalizer

Normalize text for search:

- lowercase;
- strip punctuation for indexing;
- preserve original text for snippets;
- split into terms;
- remove obvious empty terms;
- keep headings separately if available.

Do not over-engineer NLP here. This project teaches search foundations, not model semantics.

### 4. Quality Checker

Validate each document before indexing.

Checks:

- title exists;
- owner exists;
- audience exists;
- authority exists and is allowed;
- last_updated exists and parses as date;
- tags exist;
- body is not empty;
- duplicate title not found;
- content hash not duplicate;
- document is not too stale.

Suggested stale rule:

```text
stale if last_updated is more than 365 days old
```

Quality status:

```text
pass
warning
fail
```

Documents with `fail` should be excluded from normal search unless user passes a debug flag.

### 5. Lineage Recorder

For each document, record:

```json
{
  "source_path": "docs/access/api-key-reset.md",
  "content_hash": "sha256:...",
  "parser_version": "markdown-frontmatter-v1",
  "ingested_at": "2026-01-10T10:00:00Z",
  "quality_status": "pass"
}
```

Lineage matters because future AI systems need to know where an answer came from.

### 6. Keyword Index

Implement simple search.

Minimum:

- term frequency;
- title boost;
- heading boost;
- exact phrase boost if easy;
- metadata filter before ranking.

Example scoring idea:

```text
score =
title_matches * 5
+ heading_matches * 3
+ body_matches * 1
+ approved_authority_boost
- stale_penalty
```

The exact formula can differ. It must be explainable.

### 7. Search Results

Each result should include:

```json
{
  "rank": 1,
  "doc_id": "access/api-key-reset.md",
  "title": "API Key Reset Guide",
  "snippet": "Support agents should send the official reset link...",
  "score": 14,
  "matched_terms": ["api", "key", "reset"],
  "owner": "Platform Support",
  "authority": "approved",
  "last_updated": "2026-01-14",
  "quality_status": "pass",
  "why_matched": "Matched title and heading terms: api, key, reset."
}
```

Search should make ranking inspectable.

### 8. Source Detail View

For any result, show:

- title;
- source path;
- owner;
- audience;
- authority;
- last updated;
- tags;
- content hash;
- parser version;
- ingested timestamp;
- quality status;
- warnings;
- matched sections.

This teaches users not to trust results blindly.

### 9. Retrieval Evaluation

Create `eval-queries.json`.

Example:

```json
[
  {
    "query": "How do I reset an API key?",
    "expected_docs": ["docs/access/api-key-reset.md"],
    "forbidden_docs": ["docs/product/release-notes-2026-01.md"]
  },
  {
    "query": "Can we refund duplicate charges?",
    "expected_docs": ["docs/billing/refund-policy.md", "docs/billing/duplicate-charge.md"],
    "forbidden_docs": ["docs/billing/refund-policy-old.md"]
  }
]
```

Report:

- top-1 accuracy;
- top-3 recall;
- forbidden-source rate;
- stale-source rate;
- no-result rate;
- missing metadata rate.

Do not generate answers yet. Evaluate whether search finds the right source.

## Architecture

```text
docs/
  -> document loader
  -> metadata parser
  -> text normalizer
  -> quality checker
  -> lineage recorder
  -> keyword index builder
  -> search interface
  -> source detail view
  -> retrieval evaluator
```

Optional app architecture:

```text
CLI or UI
  -> search service
  -> index store
  -> document store
  -> quality report
  -> eval runner
```

Key boundary:

```text
Search retrieves sources.
It does not generate answers.
```

## Data Model

### Document Record

```json
{
  "doc_id": "docs/access/api-key-reset.md",
  "title": "API Key Reset Guide",
  "owner": "Platform Support",
  "audience": "support",
  "authority": "approved",
  "last_updated": "2026-01-14",
  "tags": ["access", "api", "reset"],
  "source_path": "docs/access/api-key-reset.md",
  "body": "Support agents should send the official reset link...",
  "headings": ["API Key Reset Guide", "Reset Steps", "Do Not"],
  "content_hash": "sha256:...",
  "parser_version": "markdown-frontmatter-v1",
  "ingested_at": "2026-01-10T10:00:00Z",
  "quality_status": "pass",
  "quality_warnings": []
}
```

### Index Entry

```json
{
  "term": "reset",
  "documents": [
    {
      "doc_id": "docs/access/api-key-reset.md",
      "title_count": 1,
      "heading_count": 1,
      "body_count": 3
    }
  ]
}
```

### Search Result

```json
{
  "query": "reset api key",
  "rank": 1,
  "doc_id": "docs/access/api-key-reset.md",
  "title": "API Key Reset Guide",
  "score": 18,
  "snippet": "Support agents should send the official reset link...",
  "matched_terms": ["reset", "api", "key"],
  "why_matched": "Matched title, heading, and body terms.",
  "quality_status": "pass"
}
```

### Evaluation Query

```json
{
  "query_id": "Q001",
  "query": "reset api key",
  "expected_docs": ["docs/access/api-key-reset.md"],
  "forbidden_docs": ["docs/product/release-notes-2026-01.md"],
  "notes": "Should find reset guide, not release notes."
}
```

## Evaluation Dataset

Create `eval-queries.json` with at least 20 queries.

Include:

- 6 direct title queries;
- 5 policy/procedure queries;
- 4 metadata/filter queries;
- 3 stale-document traps;
- 2 unavailable or no-answer queries.

Examples:

```text
Direct:
"reset api key"

Policy:
"refund duplicate charge"

Metadata/filter:
"billing policy for support audience"

Stale trap:
"old refund process"
Expected: should not return deprecated old policy as top result.

Unavailable:
"how do I change the company logo color?"
Expected: no strong result.
```

## Seeded Failure Cases

Include bad documents and confusing queries on purpose.

### Case 1: Deprecated Document Ranks Too High

```text
Old refund policy contains exact keywords.
Expected: approved current refund policy should outrank deprecated document.
```

### Case 2: Missing Metadata

```text
Document has useful text but no owner.
Expected: quality warning or exclusion.
```

### Case 3: Duplicate Content

```text
Two files contain same policy text.
Expected: duplicate warning using content hash.
```

### Case 4: Distractor Release Notes

```text
Release notes mention API keys but do not explain reset process.
Expected: reset guide should rank higher.
```

### Case 5: No Good Source

```text
Query asks for unavailable policy.
Expected: no strong result or low-confidence result, not fake certainty.
```

## Acceptance Rubric

### Basic Pass

- loads documents from folder;
- parses metadata;
- reports missing required fields;
- builds keyword search;
- returns ranked results with snippets;
- shows source path and authority.

### Strong Pass

- supports metadata filters;
- excludes failed-quality documents from normal search;
- records content hash and ingestion time;
- detects stale or deprecated documents;
- explains why each result matched;
- runs evaluation queries.

### Excellent

- includes 20+ eval queries;
- reports top-1 accuracy and top-3 recall;
- reports forbidden-source and stale-source rates;
- handles seeded failure cases;
- exports `quality-report.md`;
- exports `retrieval-eval-report.md`;
- includes `knowledge-boundary.md`;
- ranking behavior is explainable.

## Metrics

Measure:

- valid document rate;
- missing metadata rate;
- duplicate document count;
- stale document count;
- top-1 accuracy;
- top-3 recall;
- forbidden-source rate;
- stale-source rate;
- no-result handling quality.

Suggested targets:

```text
Top-1 accuracy: >= 70% on eval queries
Top-3 recall: >= 85% on eval queries
Forbidden-source rate: 0%
Failed-quality docs in normal search: 0%
Quality report generated: yes
```

Do not optimize only for top-1 accuracy.

In knowledge systems, returning an untrusted or forbidden source can be worse than returning no result.

## Final Deliverables

At the end, the learner should have:

- runnable CLI, notebook, or local app;
- `docs/` sample collection;
- `document-inventory.json`;
- keyword index file or in-memory build script;
- `quality-report.md`;
- `eval-queries.json`;
- `retrieval-eval-report.md`;
- `knowledge-boundary.md`;
- short project `README.md`;
- screenshot or terminal output for demo.

Example file layout:

```text
project-03-knowledge-base-search/
  README.md
  knowledge-boundary.md
  docs/
    billing/
      refund-policy.md
      refund-policy-old.md
    access/
      api-key-reset.md
  data/
    eval-queries.json
  src/
    loader.py
    metadata.py
    quality.py
    lineage.py
    index.py
    search.py
    evaluate.py
    report.py
  outputs/
    document-inventory.json
    quality-report.md
    retrieval-eval-report.md
```

The exact language and framework can differ. The knowledge quality behavior should not.

## Demo Script

A 3-minute demo should show:

1. Ingest the `docs/` folder.
2. Show document inventory count.
3. Show quality warnings.
4. Search for `reset api key`.
5. Open the top result and show metadata.
6. Search for `refund duplicate charge` with `authority=approved`.
7. Show that deprecated policy does not win.
8. Run retrieval evaluation.
9. Open `retrieval-eval-report.md`.

Demo narrative:

```text
This project proves that trustworthy AI starts before generation.
The system knows which documents exist, whether metadata is complete, which sources are approved, and whether search can find the right source.
It does not answer with an LLM yet because first we need reliable knowledge retrieval.
```

## What Learner Understands

After completing this project, the learner should understand:

- documents are data assets with owners, freshness, and authority;
- parsing and metadata shape downstream quality;
- search quality can be evaluated before generation;
- stale or untrusted documents can be more dangerous than missing documents;
- lineage makes future answers auditable;
- information architecture affects AI system behavior;
- RAG should be built on a tested knowledge foundation, not a messy folder.

## Extension

Later upgrades:

- Part 4: model-assisted query rewriting for unclear searches;
- Part 5: embeddings, semantic search, hybrid search, chunking, RAG, grounding;
- Part 6: PDF ingestion, OCR, table extraction;
- Part 8: search analytics, caching, monitoring, deployment;
- Part 9: access control, prompt-injection review, retrieval poisoning tests, privacy policy.
