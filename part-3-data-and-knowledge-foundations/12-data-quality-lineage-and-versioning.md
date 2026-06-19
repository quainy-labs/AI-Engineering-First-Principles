# Chapter 12: Data Quality, Lineage, and Versioning

Chapter 11 defined schemas and contracts. This chapter asks whether information remains reliable over time.

AI systems fail quietly when data changes without tracking.

## Real Problem

RAG answer changed overnight.

No prompt changed. No model changed.

What changed?

- document updated;
- parser changed;
- chunking changed;
- embedding model changed;
- stale source reappeared;
- metadata missing.

Without lineage and versioning, nobody knows.

## Bad Default Solution

Bad data operations:

- overwrite documents;
- re-index silently;
- no source version;
- no parser version;
- no chunk version;
- no embedding version;
- no quality checks;
- no rollback.

This makes AI behavior impossible to debug.

## First Principles

Data quality means data is fit for system purpose.

Lineage means knowing where data came from and how it changed.

Versioning means preserving identity of important changes.

For AI systems, version:

- source document;
- parser;
- cleaner;
- chunker;
- metadata schema;
- embedding model;
- index;
- eval dataset.

## System Decision

Track metadata:

```text
source_id:
source_uri:
source_version:
ingested_at:
parser_version:
cleaner_version:
chunker_version:
embedding_model:
index_version:
checksum:
quality_status:
```

Quality checks:

- required metadata present;
- duplicate detection;
- stale source detection;
- empty chunk detection;
- parse error rate;
- forbidden source exclusion;
- access metadata present.

## Minimal Build

Create `data-quality-plan.md`:

```text
Quality checks:
- check:
  rule:
  failure action:

Versioned artifacts:
- artifact:
  version format:
  owner:

Lineage fields:
- field:
  reason:

Rollback plan:
```

## Failure Modes

Data quality fails when:

- doc has no owner;
- title missing;
- access level missing;
- parser creates empty text;
- duplicate docs conflict;
- stale docs stay indexed;
- chunk has no source;
- eval result cannot identify corpus version.

## Evaluation

Run data quality report:

- document count;
- missing metadata count;
- duplicate count;
- stale count;
- parse failure count;
- empty chunk count;
- access-metadata missing count.

Pass if quality failures block indexing or are explicitly approved.

## Improvement

Improve by:

- adding required metadata;
- creating canonical docs;
- removing stale sources;
- adding checksums;
- versioning pipeline components;
- storing quality report with index build.

## Proof Artifact

Create `data-quality-report.md` for sample corpus.

## Decision Checklist

- [ ] Source lineage captured.
- [ ] Metadata quality checked.
- [ ] Parser/chunker/index versions tracked.
- [ ] Stale docs detected.
- [ ] Duplicate docs detected.
- [ ] Quality failures block unsafe indexing.
- [ ] Rollback path exists.

## Active Recall

1. Why can answer change if prompt/model did not change?
2. What is lineage?
3. What should be versioned in RAG pipeline?
4. Why should quality failures block indexing?
5. How does data versioning help evals?

## Next

With quality controlled, next chapter designs how people and systems find information: search, ranking, and information architecture.

