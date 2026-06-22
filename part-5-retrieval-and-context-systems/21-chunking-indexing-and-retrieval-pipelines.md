# Chapter 21: Chunking, Indexing, and Retrieval Pipelines

Chapter 20 introduced embeddings and semantic retrieval.

This chapter builds the retrieval pipeline around them.

Retrieval quality depends less on the vector database brand and more on what the system puts into the index, what metadata survives, how chunks preserve meaning, and how evidence is returned to the model.

The retrieval pipeline decides what the model can know from external knowledge.

## Concept Overview

A retrieval pipeline turns source material into retrievable evidence.

The basic flow:

```text
source document
-> parse
-> clean
-> chunk
-> attach metadata
-> embed and index
-> retrieve candidates
-> filter
-> rerank
-> deduplicate
-> return evidence packet
```

Each stage changes what the model can later use.

Chunking decides what unit of content can be retrieved.

Indexing decides how content can be searched.

Metadata decides how content can be filtered, ranked, cited, governed, and debugged.

The first principle:

> The model cannot use meaning that the retrieval pipeline destroyed.

If a policy condition is separated from the policy rule, if a table loses its headers, if a chunk loses its title, or if permissions do not travel with the chunk, retrieval becomes fragile.

## Why It Exists

Imagine the user asks:

```text
Is this customer eligible for a refund?
```

The retrieval system finds the right document:

```text
Refund Policy
```

But it retrieves the wrong chunk:

```text
Heading: Refund Eligibility
Text: Customers may request refunds under the following conditions...
```

The actual conditions are in the next section:

```text
Refunds are available within 30 days unless an enterprise exception is approved.
```

The model answers from incomplete evidence.

The document was relevant. The chunk was not.

This chapter exists because retrieval problems often hide inside pipeline decisions:

- chunks too small to preserve meaning;
- chunks too large to rank precisely;
- headings discarded;
- tables split badly;
- metadata lost;
- stale chunks left active;
- deleted documents still indexed;
- duplicate pages crowd out canonical sources;
- permissions applied too late;
- index versions unknown.

Good retrieval is not just "put documents into a vector database." It is a pipeline with design choices.

## Mental Model

Think of retrieval as preparing evidence for a court case.

It is not enough to bring random paragraphs. You need the right excerpt, source title, section, date, authority, owner, and explanation of why it matters.

The mental model:

```text
raw knowledge
-> meaningful chunks
-> searchable indexes
-> filtered candidates
-> ranked evidence
-> evidence packet
-> model answer
```

The evidence packet is the bridge between retrieval and generation.

It should tell the model and the system:

- what the excerpt says;
- where it came from;
- what section it belongs to;
- whether it is current;
- whether it is authoritative;
- whether the user is allowed to use it;
- why it was retrieved.

Retrieval is not done when the system has a vector score. Retrieval is done when the system returns usable evidence.

## Internal Mechanics

The pipeline starts with source documents.

### Parse and Clean

Chapter 10 covered ingestion, parsing, and cleaning. Retrieval depends on that work.

Before chunking, the system should preserve:

- title;
- headings;
- sections;
- tables;
- lists;
- code blocks;
- captions;
- links;
- source URI;
- document metadata.

Cleaning should remove noise without removing meaning.

Examples:

- remove repeated footers;
- remove navigation menus;
- preserve table headers;
- preserve section headings;
- preserve policy exception language.

### Chunking

Chunking splits content into retrievable units.

Chunking is a product and system design decision, not just a token-size setting.

Good chunks:

- preserve meaning;
- include title and heading;
- keep rule and condition together;
- keep question and answer together;
- keep table headers with rows;
- keep code with explanation when needed;
- are small enough to rank precisely;
- are large enough to be useful;
- carry source metadata.

Bad chunks:

- split conditions away from rules;
- split table rows from headers;
- remove section path;
- mix unrelated topics;
- lose source identity;
- are too generic to answer;
- are too large to rank well.

Choose chunking strategy by content type:

| Content Type | Better Chunk Strategy |
|---|---|
| Policies | section-based chunks with conditions |
| FAQs | question-answer pairs |
| API docs | endpoint-level chunks |
| Tables | preserve table with caption and headers |
| Long guides | heading-aware chunks with overlap |
| Release notes | version/date-based chunks |
| Code docs | function or example-level chunks |
| Contracts/forms | clause or field-level chunks |

Overlap can help when meaning crosses boundaries, but too much overlap creates duplicate crowding.

### Metadata Inheritance

Every chunk should inherit metadata from its source.

Required metadata often includes:

```text
chunk_id
document_id
source_uri
title
section_path
owner
workspace_id
permission_labels
document_type
product_area
audience
authority_level
status
last_updated
source_version
parser_version
chunker_version
embedding_model
index_version
```

Metadata supports:

- access control;
- filtering;
- ranking;
- citations;
- freshness;
- deletion propagation;
- debugging;
- evaluation;
- governance.

A chunk without metadata becomes a loose paragraph. Loose paragraphs are dangerous in AI systems.

### Indexing

Indexing makes chunks searchable.

Common indexes:

- keyword index;
- vector index;
- hybrid index;
- metadata store;
- document store;
- specialized exact lookup for IDs or codes.

Indexing should record:

```text
index_name
index_version
corpus_version
embedding_model
chunker_version
created_at
quality_report
active_status
```

Index versions matter because retrieval behavior can change when sources, chunkers, embeddings, or ranking rules change.

### Retrieval Flow

A reliable retrieval flow:

```text
query
-> normalize
-> detect intent and filters
-> retrieve keyword candidates
-> retrieve semantic candidates
-> merge candidates
-> apply permission filters
-> apply freshness and authority filters
-> rerank
-> deduplicate
-> build evidence packet
```

Filter before generation. Do not retrieve forbidden context and ask the model not to reveal it.

### Reranking

Reranking reorders candidates after initial retrieval.

Signals may include:

- semantic similarity;
- keyword match;
- title match;
- heading match;
- authority level;
- freshness;
- audience match;
- exact term match;
- document type;
- source quality;
- citation success history.

Reranking should prefer useful evidence, not merely similar text.

### Deduplication

Duplicate or near-duplicate chunks can crowd out better evidence.

Deduplicate by:

- source document;
- content hash;
- canonical source;
- version;
- near-duplicate similarity;
- title/section match.

Duplicate crowding can make retrieval appear confident while returning the same weak evidence repeatedly.

### Reindexing and Deletion

Retrieval systems must handle change.

Define:

```text
When does a document reindex?
What happens to old chunks?
How are deleted documents removed?
How are permission changes propagated?
How are stale indexes retired?
Can the system roll back to previous index?
```

Deleting a document should remove or deactivate:

- raw file when required;
- parsed record;
- chunks;
- embeddings;
- search index entries;
- caches;
- derived summaries when needed.

### Evidence Packet

The evidence packet is what retrieval gives to the model and the rest of the system.

It should include:

```text
excerpt
title
section_path
source_uri
document_id
chunk_id
owner
last_updated
authority_level
permission_scope
score
retrieval_reason
index_version
```

Example:

```json
{
  "chunk_id": "policy-refunds#eligibility-v3",
  "document_id": "policy-refunds",
  "title": "Refund Policy",
  "section_path": "Eligibility > Enterprise Exceptions",
  "source_uri": "policies/refunds.md",
  "owner": "support-ops",
  "last_updated": "2026-01-10",
  "authority_level": "approved_policy",
  "permission_scope": "support_internal",
  "excerpt": "Enterprise customers may request exception review...",
  "score": 0.87,
  "retrieval_reason": "matched refund eligibility and enterprise exception",
  "index_version": "support_docs_2026_06_22_01"
}
```

This gives the model evidence and gives engineers traceability.

## Real-World Example

Imagine building retrieval for a company knowledge assistant.

Sources:

```text
help-center articles
internal policies
API docs
release notes
support macros
```

Weak pipeline:

```text
split every document into 1,000-token chunks
-> embed chunks
-> retrieve top 5
```

Failures:

- API endpoint docs split method from required permissions;
- release notes outrank canonical docs;
- old support macros remain active;
- internal policies appear in customer-facing answers;
- policy exceptions split away from eligibility rules;
- duplicate help articles crowd retrieval.

Better pipeline:

```text
parse documents by source type
-> clean source-specific noise
-> chunk policies by section
-> chunk FAQs by question-answer pair
-> chunk API docs by endpoint
-> preserve table headers
-> attach metadata
-> build keyword and vector indexes
-> filter by audience and permission
-> rerank by authority and freshness
-> deduplicate by canonical source
-> return evidence packets
```

Now retrieval is designed for the actual corpus instead of a generic chunk size.

## Common Mistakes and Failure Modes

Mistake 1: Fixed-size chunking for every source

Different content types carry meaning differently. Policies, FAQs, API docs, tables, and release notes need different chunking rules.

Mistake 2: Chunks lose headings

Headings often explain what the text means. Without headings, chunks become ambiguous.

Mistake 3: Tables split from headers

A table row without column names can become meaningless or misleading.

Mistake 4: Permissions checked after retrieval

Forbidden content should not enter model context. Permission filtering belongs before context assembly.

Mistake 5: Deleted sources remain indexed

If old chunks stay active after source deletion, the system can answer from knowledge that should no longer exist.

Mistake 6: Duplicate chunks crowd out better evidence

Repeated content can dominate top-k results and hide more authoritative sources.

Mistake 7: Index version unknown

If an answer changes and the index version is unknown, debugging becomes guesswork.

Mistake 8: Evidence packet is just text

The model and system need source, title, authority, freshness, and retrieval reason, not just an excerpt.

## System Design Decisions

Design chunking by content type.

```text
What source types exist?
What structure matters in each?
What should never be split?
What should be grouped?
How much overlap is useful?
```

Define metadata schema.

```text
Which metadata must every chunk carry?
Which metadata supports access control?
Which metadata supports ranking?
Which metadata supports citations?
Which metadata supports deletion and reindexing?
```

Choose indexes.

```text
Keyword index?
Vector index?
Hybrid retrieval?
Exact lookup index?
Metadata store?
```

Define retrieval flow.

```text
How is the query normalized?
Which filters are detected?
Which candidate sources are searched?
How are candidates merged?
When are permissions applied?
How are results reranked?
How are duplicates removed?
```

Define lifecycle behavior.

```text
What triggers reindexing?
How are old chunks retired?
How are deletes propagated?
How are permission changes applied?
How is index version tracked?
```

Define evaluation.

```text
Expected document queries
Expected chunk queries
Multi-section questions
Exact ID queries
Stale-source traps
Permission boundary cases
Duplicate crowding tests
Delete/reindex tests
```

The goal is to make retrieval reliable enough that generation has a fair chance.

## Build Artifact

Create `retrieval-pipeline-spec.md` for one knowledge workflow.

Choose a workflow such as:

- product docs assistant;
- support refund assistant;
- HR policy assistant;
- internal operations assistant;
- multimodal document analyst.

Use this template:

```text
Retrieval Pipeline Spec

Workflow:
User goal:
Corpus:

Source Types:
- source type:
  examples:
  parser:
  cleaning rules:
  chunking strategy:
  must preserve:

Metadata Schema:
- field:
  required: yes/no
  used for:

Indexes:
- index:
  content:
  metadata:
  version:

Retrieval Flow:
- step:
  input:
  output:
  failure mode:

Filters:
- filter:
  applied before generation: yes/no
  reason:

Reranking:
- signal:
  boost or penalty:
  reason:

Deduplication:
- rule:
  reason:

Freshness and Deletion:
- reindex trigger:
- stale chunk behavior:
- deleted source behavior:
- permission-change behavior:

Evidence Packet:
- field:
  reason:

Eval Cases:
- query:
  expected document:
  expected chunk:
  forbidden source:
  reason:
```

Example:

```text
Source type: API docs
Parser: markdown parser
Chunking strategy: endpoint-level
Must preserve: method, path, required permissions, request example, response example

Source type: policy docs
Chunking strategy: section-based with exceptions kept beside rule
Must preserve: title, section path, conditions, exception language

Filter:
permission_labels
applied before generation: yes
reason: unauthorized chunks must not enter model context

Evidence packet field:
index_version
reason: needed to debug answer changes after reindex
```

The artifact is complete when another engineer can build the index, retrieve evidence, handle updates/deletes, and evaluate whether the right chunks are returned.

## Active Recall Questions

1. Why is chunking a product and system design decision?
2. Why can retrieving the correct document still produce the wrong answer?
3. What metadata should travel with every chunk?
4. Why should permissions be applied before context assembly?
5. What is an evidence packet?
6. Why does index version matter for debugging?
7. How can duplicate chunks harm retrieval?
8. What should happen when a source document is deleted?

## Summary

Retrieval pipelines turn source knowledge into usable evidence. Parsing, cleaning, chunking, metadata, indexing, filtering, reranking, deduplication, and evidence packet design all shape what the model can later use.

Good chunks preserve meaning. Good metadata preserves context, permissions, freshness, authority, and traceability. Good pipelines handle updates, deletes, and index versions. Good evidence packets give the model useful context and engineers a debugging trail.

The most important rule:

> Retrieval quality is built before the model sees the evidence.

## Preview of Next Chapter

Chapter 21 built the retrieval pipeline.

Chapter 22 combines retrieval with generation.

Next, you will learn retrieval-augmented generation, or RAG. This matters because RAG is not "vector database plus chat." It is a knowledge system, retrieval pipeline, context builder, model interface, citation policy, refusal policy, and evaluation loop working together.
