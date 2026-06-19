# Chapter 21: Chunking, Indexing, and Retrieval Pipelines

Chapter 20 explained semantic retrieval. This chapter builds the retrieval pipeline that makes it reliable.

Retrieval quality depends less on the vector database brand and more on what you put into the index.

## Real Problem

Your retrieval system finds the right document but wrong chunk.

Example:

- query asks for refund eligibility;
- retrieved chunk contains refund heading;
- actual eligibility rules are in next section;
- model answers from incomplete evidence.

The document was relevant. The chunk was not.

## Bad Default Solution

Bad solution:

- split every document into fixed 1,000-token chunks;
- ignore headings;
- discard metadata;
- index once;
- never track source version.

This creates retrieval that looks technical but cannot be trusted.

## First Principles

Retrieval pipeline has stages:

```text
source document
-> parse
-> clean
-> chunk
-> attach metadata
-> embed/index
-> retrieve candidates
-> filter
-> rerank
-> return evidence
```

Each stage changes what the model can later know.

Good chunks:

- preserve meaning;
- include title/heading;
- include source identity;
- are not too large;
- are not too small;
- keep tables/code/policies intact when needed;
- carry metadata.

Metadata matters:

- source;
- owner;
- access level;
- document type;
- product area;
- last updated;
- approval status;
- authority level;
- version.

## System Decision

Design indexing policy:

```text
Source types:
Parser:
Cleaning rules:
Chunking strategy:
Chunk metadata:
Embedding model:
Keyword index:
Vector index:
Reranker:
Freshness policy:
Reindex trigger:
Deletion policy:
```

Choose chunking by content:

| Content | Chunk Strategy |
|---|---|
| policies | section-based chunks |
| FAQs | question-answer pairs |
| API docs | endpoint-level chunks |
| tables | preserve table with caption |
| long guides | heading-aware chunks with overlap |
| release notes | date/version chunks |

## Retrieval Pipeline

Basic reliable pipeline:

```text
query
-> normalize
-> detect filters
-> keyword + semantic retrieve
-> metadata filter
-> rerank
-> deduplicate
-> apply authority/freshness
-> return evidence packet
```

Evidence packet should include:

- excerpt;
- title;
- source path;
- heading;
- owner;
- last updated;
- authority;
- access level;
- score;
- retrieval reason.

## Minimal Build

Create `retrieval-pipeline-spec.md`:

```text
Corpus:
Source types:
Parsing rules:
Cleaning rules:
Chunking rules by source type:
Metadata fields:
Indexes:
Retrieval method:
Filters:
Reranking:
Deduplication:
Freshness:
Access control:
Reindex process:
```

Add sample evidence packet:

```json
{
  "chunk_id": "policy-refunds#eligibility",
  "title": "Refund Policy",
  "heading": "Eligibility",
  "source": "policies/refunds.md",
  "owner": "support-ops",
  "last_updated": "2026-01-10",
  "authority": "approved_policy",
  "excerpt": "Refunds are eligible when...",
  "score": 0.87
}
```

## Failure Modes

Retrieval pipelines fail when:

- chunks lose headings;
- tables are split badly;
- stale chunks remain after document update;
- deleted documents remain indexed;
- permissions are checked after retrieval;
- reranker rewards fluent but wrong text;
- duplicate chunks crowd out better evidence;
- index version is unknown.

## Evaluation

Test retrieval pipeline with:

- expected chunk queries;
- expected document queries;
- multi-section questions;
- exact ID queries;
- stale source traps;
- permission boundary cases.

Measure:

- expected chunk top-3 recall;
- expected document top-3 recall;
- stale-source rate;
- forbidden-source rate;
- duplicate crowding rate;
- reindex correctness.

## Improvement

Improve pipeline by:

- chunking by structure;
- preserving headings;
- adding metadata filters;
- using hybrid retrieval;
- adding reranking;
- deduplicating results;
- versioning index builds;
- testing reindex/delete behavior.

## Proof Artifact

Create `retrieval-pipeline-spec.md`.

Include:

- parsing/chunking rules;
- metadata schema;
- index choices;
- retrieval/reranking flow;
- reindex policy;
- eval cases.

## Decision Checklist

- [ ] Chunking strategy matches content type.
- [ ] Required metadata is attached.
- [ ] Deleted/stale source handling exists.
- [ ] Access control is part of retrieval.
- [ ] Reranking decision is explicit.
- [ ] Index version is tracked.
- [ ] Retrieval eval covers chunks and documents.

## Active Recall

1. Why is chunking a product decision?
2. What metadata should travel with chunks?
3. Why can correct document retrieval still fail?
4. What is an evidence packet?
5. Why version retrieval index builds?

## Next

Next chapter combines retrieval with generation. That is RAG, but now built on search, embeddings, metadata, and pipeline design instead of demo glue.
