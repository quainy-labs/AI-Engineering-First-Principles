# Project 3: Knowledge Base Search Engine

## Build

Build a searchable knowledge base over a small document collection with ingestion, metadata, quality checks, keyword search, filters, and retrieval evaluation.

This project is not a RAG chatbot yet. It is not a vector database project yet. It is the information foundation that trustworthy retrieval depends on.

## User

Support teammate, researcher, student, or operator who needs to find trusted information quickly.

## Product Outcome

User can search documents and see:

- ranked results;
- source metadata;
- matched snippets;
- authority level;
- last updated date;
- why result was retrieved;
- data quality warnings;
- retrieval quality report.

## Sample Data

Use 20-50 markdown files:

```text
docs/
  billing/refund-policy.md
  billing/invoice-admins.md
  access/api-key-reset.md
  access/production-log-access.md
  product/release-notes-2026-01.md
```

Each document needs frontmatter:

```yaml
title: Refund Policy
owner: Support Ops
audience: support
authority: approved
last_updated: 2026-01-10
tags: [billing, refunds]
```

## Core Features

### 1. Document Ingestion

Read markdown files, extract:

- title;
- headings;
- body;
- owner;
- tags;
- authority;
- last updated;
- path.

Also store:

- source URI;
- content hash;
- parser version;
- ingested timestamp;
- quality status.

### 2. Data Quality Checks

Before indexing, validate:

- title exists;
- owner exists;
- authority exists;
- last updated exists;
- document not duplicate;
- document not stale;
- body not empty;
- access level present if needed.

Bad documents should be excluded or marked.

### 3. Keyword Search

Implement simple keyword/inverted-index search.

Return:

- document title;
- snippet;
- score;
- matched terms.

### 4. Metadata Filtering

Support filters:

- tag;
- owner;
- audience;
- authority;
- last updated;

Important rule:

Search should not return forbidden/stale docs when filter excludes them.

### 5. Source and Lineage View

For any result, show:

- source path;
- owner;
- authority;
- last updated;
- content hash;
- ingestion time;
- parser version;
- quality status.

### 6. Retrieval Evaluation

Create `eval_queries.json`:

```json
[
  {
    "query": "How do I reset an API key?",
    "expected_doc": "access/api-key-reset.md",
    "forbidden_docs": ["product/release-notes-2026-01.md"]
  }
]
```

Report:

- top-1 accuracy;
- top-3 recall;
- stale-source rate;
- forbidden-source rate;
- distractor rate.
- missing metadata rate.

## Architecture

```text
docs/
-> document loader
-> metadata parser
-> quality checker
-> chunker
-> keyword index
-> search API
-> evaluation runner
-> search UI/CLI
```

## Data Model

```json
{
  "doc_id": "access/api-key-reset.md",
  "title": "Reset API Key",
  "owner": "Platform Team",
  "authority": "approved",
  "last_updated": "2026-01-10",
  "tags": ["access", "api"],
  "source_uri": "docs/access/api-key-reset.md",
  "content_hash": "sha256:...",
  "parser_version": "markdown-v1",
  "ingested_at": "2026-01-10T10:00:00Z",
  "quality_status": "pass",
  "chunks": [
    {
      "chunk_id": "access/api-key-reset.md#1",
      "heading": "Reset Process",
      "text": "To reset an API key..."
    }
  ]
}
```

## Suggested Implementation

Start as CLI:

```bash
search "reset api key" --mode keyword
search "billing admin" --authority approved
evaluate-retrieval
quality-report
```

Then optional web UI:

- search box;
- filters;
- result list;
- metadata panel;
- eval report view.
- quality report view.

## Evaluation

Use 20 queries:

- 8 direct keyword queries;
- 6 metadata/filter queries;
- 3 edge queries;
- 3 should-refuse/unavailable queries.

Acceptance:

- keyword search works for exact terms;
- metadata filters change results correctly;
- stale/forbidden docs excluded;
- quality report catches bad documents;
- eval report identifies failures.

## What Learner Understands

- Retrieval quality controls answer quality.
- Metadata often matters more than model choice.
- Data quality and lineage affect answer quality.
- Evaluation must inspect retrieval before generation.

## Extension

Add:

- hybrid search;
- semantic search;
- reranking;
- stale document warnings;
- synonym dictionary;
- search analytics;
- admin page for document quality.
