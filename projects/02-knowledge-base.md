# Project 2: Knowledge Base Search Engine

## Build

Build a searchable knowledge base over a small document collection with keyword search, metadata filters, semantic search, and retrieval evaluation.

This project is not a RAG chatbot yet. It is the retrieval system that a trustworthy assistant depends on.

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

### 2. Keyword Search

Implement simple keyword/inverted-index search.

Return:

- document title;
- snippet;
- score;
- matched terms.

### 3. Metadata Filtering

Support filters:

- tag;
- owner;
- audience;
- authority;
- last updated;

Important rule:

Search should not return forbidden/stale docs when filter excludes them.

### 4. Semantic Search

Add embedding-based similarity search.

Can be:

- local embedding model;
- API embedding model;
- simple placeholder vectors for learning if no model access.

The system should compare keyword vs semantic results.

### 5. Retrieval Evaluation

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

## Architecture

```text
docs/
-> document loader
-> metadata parser
-> chunker
-> keyword index
-> embedding index
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
search "change billing admin" --mode semantic
evaluate-retrieval
```

Then optional web UI:

- search box;
- filters;
- result list;
- metadata panel;
- eval report view.

## Evaluation

Use 20 queries:

- 8 direct keyword queries;
- 6 paraphrased semantic queries;
- 3 edge queries;
- 3 should-refuse/unavailable queries.

Acceptance:

- keyword search works for exact terms;
- semantic search helps at least 3 paraphrased queries;
- metadata filters change results correctly;
- eval report identifies failures.

## What Learner Understands

- Retrieval quality controls answer quality.
- Metadata often matters more than model choice.
- Semantic search helps, but can retrieve plausible wrong results.
- Evaluation must inspect retrieval before generation.

## Extension

Add:

- hybrid search;
- reranking;
- stale document warnings;
- synonym dictionary;
- search analytics;
- admin page for document quality.
