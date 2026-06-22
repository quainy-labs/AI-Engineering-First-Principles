# Chapter 12: Data Quality, Lineage, and Versioning

Chapter 11 defined schemas and contracts:

```text
information shape
-> producer
-> consumer
-> validation
-> invalid-output path
```

This chapter asks whether the information remains reliable over time.

AI systems can change behavior even when the prompt and model stay the same. A document may update. A parser may change. A chunking rule may split content differently. An embedding model may change. A stale page may reappear. A required metadata field may disappear.

Without quality checks, lineage, and versioning, the team cannot explain what changed.

## Concept Overview

Data quality means information is fit for the system's purpose.

Lineage means the system knows where information came from and how it moved through the pipeline.

Versioning means important changes are identified instead of silently overwritten.

Together, they answer:

```text
Can we trust this record?
Where did it come from?
When was it ingested?
Which parser created it?
Which cleaner changed it?
Which chunker split it?
Which embedding model indexed it?
Which index version served it?
Which eval dataset tested it?
Can we compare this behavior to yesterday?
Can we roll back if needed?
```

For AI systems, quality is not only about clean rows in a table. It includes documents, chunks, embeddings, metadata, indexes, eval examples, traces, and generated artifacts.

The first principle:

> If an answer depends on an artifact, that artifact needs quality checks and identity.

## Why It Exists

Imagine a RAG system that answered correctly yesterday and incorrectly today.

No prompt changed.

No model changed.

So what happened?

Possible causes:

- source document updated;
- old document version returned;
- parser started dropping headings;
- cleaner removed exception text;
- chunker split policy from conditions;
- embedding model changed;
- index was rebuilt with stale sources;
- document permissions changed;
- duplicate page outranked canonical page;
- metadata field disappeared;
- eval dataset used a different corpus version.

Without lineage, every investigation becomes guesswork.

Weak teams ask:

> Why did the model do that?

Strong teams ask:

> Which source, parser, chunker, embedding model, index, and retrieval result produced that answer?

This chapter exists because AI systems are made from many moving artifacts. If those artifacts are not tracked, the system cannot be debugged, evaluated, governed, or improved reliably.

## Mental Model

Think of an AI answer as the final dish in a kitchen.

If the dish tastes wrong, you need to know the ingredients, supplier, recipe version, cook, oven temperature, and serving time. You cannot debug quality by staring only at the plate.

In AI systems, the answer is the plate.

The ingredients are:

```text
source documents
structured data
state
metadata
chunks
embeddings
indexes
prompts
model versions
tool results
eval examples
```

The mental model:

```text
source
-> ingestion job
-> parser version
-> cleaner version
-> normalized record
-> metadata schema
-> chunker version
-> embedding model
-> index version
-> retrieval result
-> model answer
```

Lineage connects the chain.

Quality checks guard the chain.

Versioning lets you compare one chain to another.

When the answer changes, the system should help you ask:

```text
Did the source change?
Did processing change?
Did retrieval change?
Did the model change?
Did permissions change?
Did state change?
Did evaluation use the same data version?
```

## Internal Mechanics

Data quality starts with defining what "good enough" means for each artifact.

### Quality Checks

Quality checks should run before information becomes active in the system.

Document checks:

- title exists;
- source URI exists;
- owner exists;
- updated timestamp exists;
- permission labels exist;
- content is not empty;
- source is not archived;
- document type is known;
- authority level is known.

Parsing checks:

- parse produced enough text;
- table structure was preserved;
- headings were preserved;
- OCR confidence is acceptable;
- parser warnings are recorded;
- unsupported formats are rejected or quarantined.

Chunk checks:

- chunk has source document ID;
- chunk has title or section path;
- chunk is not empty;
- chunk is not too long or too short;
- chunk inherited permissions;
- chunk inherited source version;
- chunk has chunker version.

Index checks:

- index build references corpus version;
- embedding model is recorded;
- forbidden sources are excluded;
- stale sources are excluded;
- deleted sources are removed;
- duplicate rate is measured;
- retrieval smoke tests pass.

Eval checks:

- eval examples have expected source references;
- eval dataset version is recorded;
- corpus version is recorded;
- prompt and model version are recorded;
- metric definitions are stable.

### Lineage Fields

Lineage is built from metadata.

Useful fields:

```text
source_id
source_uri
source_version
source_updated_at
ingestion_job_id
ingested_at
parser_version
cleaner_version
normalizer_version
metadata_schema_version
chunker_version
chunk_id
embedding_model
embedding_created_at
index_name
index_version
quality_status
content_hash
permission_version
```

For structured data, lineage may include:

```text
table_name
record_id
source_system
sync_job_id
sync_timestamp
schema_version
transformation_version
```

For generated outputs, lineage may include:

```text
prompt_version
model_version
retrieval_run_id
context_document_ids
tool_call_ids
trace_id
eval_run_id
```

### Versioning

Version the artifacts that change behavior.

In AI systems, version:

- source documents;
- schemas;
- parsers;
- cleaners;
- normalizers;
- chunkers;
- embedding models;
- indexes;
- prompts;
- eval datasets;
- model configurations;
- retrieval ranking rules;
- tool contracts.

Versioning does not always require a complex system. It can start with clear identifiers:

```text
parser_version: pdf_parser_v2
chunker_version: policy_chunker_v3
embedding_model: text-embedding-model-name
index_version: support_docs_2026_06_22_01
eval_dataset_version: refund_eval_v4
```

The key is that important artifacts should not change invisibly.

### Quality Gates

A quality gate decides whether an artifact is allowed to become active.

Examples:

```text
block indexing if permission labels missing
block indexing if source URI missing
block retrieval if document is archived
block release if forbidden-source rate increases
block deployment if eval dataset version is missing
send to review if parse confidence is low
```

Not every quality issue must block the system. But every serious issue should have a defined action:

```text
block
warn
quarantine
manual review
retry
accept with exception
```

### Rollback

Rollback is only possible if versions exist.

You may need to roll back:

- a bad parser release;
- a broken chunking strategy;
- an index built with stale documents;
- an embedding migration;
- an eval dataset change;
- a metadata schema migration;
- a prompt version.

Rollback plan:

```text
previous active index version
previous parser/chunker versions
source corpus snapshot
quality report for current and previous build
eval comparison
decision owner
rollback command or process
```

Rollback is not failure. It is operational control.

## Real-World Example

Imagine a company knowledge assistant.

Yesterday, a user asked:

> Are enterprise customers eligible for refund exceptions?

The answer was correct:

```text
Enterprise customers may request exception review through support leadership.
```

Today, the assistant answers:

```text
No exceptions are available.
```

No prompt or model changed.

A lineage-aware system can inspect:

```text
retrieval_run_id: ret_456
index_version: support_docs_2026_06_22_01
embedding_model: embedding_v3
chunker_version: policy_chunker_v4
source_document_id: refund_policy
source_version: 2026-06-21
parser_version: html_parser_v2
quality_report: qr_789
```

Investigation finds:

```text
chunker_version policy_chunker_v4 split the "enterprise exceptions" section away from the refund eligibility heading.
ranking then preferred the generic "no refunds after 30 days" chunk.
```

The fix is not random prompt tuning.

Better fix:

- improve chunking around policy headings;
- preserve section path in chunk metadata;
- add retrieval test for enterprise exception question;
- rebuild index as a new version;
- compare eval results against previous index;
- roll back if forbidden or stale result rate increases.

This is how lineage turns confusion into engineering.

## Common Mistakes and Failure Modes

Mistake 1: Overwriting sources without version identity

If the system overwrites documents silently, it cannot compare old and new behavior.

Mistake 2: Rebuilding indexes without build metadata

An index should know which corpus, parser, chunker, embedding model, and quality report created it.

Mistake 3: Treating metadata as optional

Missing owner, source, permission, version, or timestamp makes retrieval and governance weaker.

Mistake 4: Quality checks only after users complain

Quality gates should catch obvious issues before bad data becomes active.

Mistake 5: No duplicate or stale-source detection

Duplicate and stale documents can dominate retrieval and make wrong answers look well-supported.

Mistake 6: Eval results without data version

An eval score is incomplete unless you know the prompt, model, corpus, index, retrieval rules, and eval dataset versions.

Mistake 7: No rollback path

If every build overwrites the previous one, the team cannot recover quickly from a bad ingestion or index release.

Mistake 8: Lineage exists in theory but not in traces

Lineage must be visible during debugging. If traces cannot show source IDs, chunk IDs, and index versions, the team still works blind.

## System Design Decisions

Decide which artifacts must be versioned.

Start with:

```text
source documents
parsed records
metadata schema
chunking strategy
embedding model
index build
eval dataset
prompt template
model configuration
```

Decide quality gates:

```text
Which failures block indexing?
Which failures create warnings?
Which failures need human review?
Which failures are acceptable for low-risk sources?
```

Decide lineage requirements:

```text
What fields must every document carry?
What fields must every chunk carry?
What fields must every index build record?
What fields must every eval run record?
What fields must every trace expose?
```

Decide release process:

```text
Can a new index become active automatically?
Does it need eval pass?
Who approves release?
How is previous version preserved?
How is rollback performed?
```

Decide quality reporting:

```text
document count
missing metadata count
duplicate count
stale source count
parse failure count
empty chunk count
forbidden source count
permission metadata missing count
retrieval smoke test result
```

The practical goal:

> When behavior changes, the team should be able to explain which artifact changed.

## Build Artifact

Create `data-quality-plan.md` and `data-quality-report.md` for one AI workflow.

Choose a workflow such as:

- company knowledge assistant;
- support refund assistant;
- HR policy assistant;
- invoice extraction system;
- multimodal document analyst.

Use this template for `data-quality-plan.md`:

```text
Data Quality Plan

Workflow:
User goal:
Critical answer types:

Versioned Artifacts:
- artifact:
  version format:
  owner:
  why it matters:

Required Lineage Fields:
- field:
  applies to:
  reason:

Quality Checks:
- check:
  rule:
  severity:
  failure action:

Quality Gates:
- gate:
  blocks release when:
  owner:

Rollback Plan:
- rollback target:
  previous version stored at:
  rollback trigger:
  rollback owner:

Trace Requirements:
- field:
  shown during debugging:
```

Use this template for `data-quality-report.md`:

```text
Data Quality Report

Workflow:
Run date:
Corpus version:
Index version:
Parser version:
Chunker version:
Embedding model:

Counts:
- documents:
- chunks:
- sources:
- indexed records:

Quality Results:
- missing title:
- missing source URI:
- missing owner:
- missing permission labels:
- duplicate sources:
- stale sources:
- parse failures:
- empty chunks:
- low-confidence parses:

Retrieval Smoke Tests:
- query:
  expected source:
  top result:
  passed:

Release Decision:
- approved:
- blocked:
- warnings:
- owner:
```

Example:

```text
Workflow: Company knowledge assistant

Versioned Artifacts:
- support_docs corpus: support_docs_2026_06_22
- parser: html_parser_v2
- chunker: heading_chunker_v3
- embedding model: embedding_v3
- index: support_docs_index_2026_06_22_01

Quality Gate:
Block index release if permission labels are missing from any internal document.

Retrieval Smoke Test:
Query: enterprise refund exception
Expected source: Refund Policy, Enterprise Exceptions section
Pass condition: expected source appears in top 3 results
```

The artifact is complete when another engineer can tell what changed, what quality checks ran, whether release should proceed, and how to roll back.

## Active Recall Questions

1. Why can an AI answer change when the prompt and model did not change?
2. What is the difference between data quality, lineage, and versioning?
3. Which artifacts should be versioned in a retrieval or RAG pipeline?
4. Why should chunks carry source and version metadata?
5. Why is an eval score incomplete without corpus and index versions?
6. What kinds of quality failures should block indexing?
7. How does lineage help debug a bad answer?
8. Why is rollback difficult without versioned artifacts?

## Summary

AI systems depend on many changing artifacts: sources, schemas, parsers, cleaners, chunkers, embeddings, indexes, prompts, models, eval datasets, and traces. If these artifacts change silently, behavior becomes impossible to explain.

Data quality checks whether information is fit for use. Lineage shows where information came from and how it changed. Versioning preserves identity across important changes. Quality gates and rollback plans turn data operations into reliable engineering.

The most important rule:

> If the system can answer from it, retrieve from it, evaluate with it, or act on it, track where it came from and which version it is.

## Preview of Next Chapter

Chapter 12 made information reliable and traceable over time.

Chapter 13 asks how people and systems find that information.

Next, you will learn search, ranking, and information architecture. This matters because even high-quality data fails the user if documents are poorly organized, stale pages outrank canonical ones, metadata is missing, or search retrieves the wrong source before the model ever sees context.
