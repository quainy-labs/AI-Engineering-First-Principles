# Chapter 20: Embeddings and Semantic Retrieval

Part 4 made models usable as bounded components. Now we ground those components in external knowledge.

Chapter 13 introduced basic search and information architecture. This chapter adds semantic retrieval: finding meaning, not only matching words.

## Real Problem

User asks:

```text
How can I rotate my access token?
```

The document says:

```text
Reset your API key from workspace settings.
```

Keyword search may miss this. The words differ, but the meaning is close.

Semantic retrieval helps system find related meaning across different wording.

## Bad Default Solution

Bad solution:

- embed every chunk;
- retrieve top 5 vectors;
- assume semantic search solved knowledge.

Problems:

- similar does not mean correct;
- embeddings may retrieve broad related text;
- exact terms, IDs, and error codes may be missed;
- stale documents can still rank high;
- permissions and authority are not solved;
- no eval proves retrieval quality.

Embeddings improve matching. They do not replace information architecture.

## First Principles

An embedding is a numerical representation of content.

Text goes in:

```text
Reset your API key.
```

Vector comes out:

```text
[0.12, -0.03, 0.88, ...]
```

Similar meanings should have nearby vectors.

Semantic retrieval flow:

```text
user query
-> query embedding
-> compare against document embeddings
-> return nearest chunks
-> rank/filter
```

Embeddings are useful when:

- user wording differs from document wording;
- query is conceptual;
- synonyms matter;
- documents have natural language explanations.

Embeddings are weak when:

- exact identifier matters;
- query contains code, SKU, or error string;
- metadata filter is required;
- authority/freshness decides answer;
- corpus has many near-duplicate pages.

## System Decision

Choose retrieval method by question type:

| Question Type | Better Starting Method |
|---|---|
| exact ID, code, error string | keyword |
| policy by department | metadata + keyword/semantic |
| conceptual question | semantic |
| latest approved process | freshness + authority filters |
| mixed wording and exact terms | hybrid |

Hybrid retrieval often wins:

```text
keyword candidates + semantic candidates -> merge -> rerank -> filter
```

## Minimal Build

Create `semantic-retrieval-plan.md`:

```text
Use case:
Corpus:
Question types:

Embedding model:
Vector index:
Metadata filters:
Hybrid search? yes/no
Reranker? yes/no

Where keyword is required:
-

Where semantic search is required:
-

Eval queries:
1. query:
   expected source:
   reason:
```

## Failure Modes

Semantic retrieval fails when:

- chunks are too large or too small;
- metadata is missing;
- similar but wrong source ranks high;
- exact-match queries are treated as semantic;
- duplicate documents compete;
- stale docs remain indexed;
- query embedding lacks needed context;
- no reranking exists for hard cases.

## Evaluation

Evaluate retrieval before answer generation.

Create eval set:

- exact-term queries;
- synonym queries;
- conceptual queries;
- stale-source traps;
- duplicate-source traps;
- should-not-retrieve private source.

Metrics:

- top-1 accuracy;
- top-3 recall;
- forbidden-source rate;
- stale-source rate;
- exact-query failure rate;
- semantic-query success rate.

## Improvement

Improve retrieval by:

- adding metadata filters;
- using hybrid search;
- improving chunk boundaries;
- adding titles/headings to chunks;
- removing duplicates;
- filtering stale sources;
- adding reranking;
- rewriting queries with constraints.

## Proof Artifact

Create `semantic-retrieval-plan.md`.

Include:

- retrieval method choice;
- embedding model;
- metadata filters;
- hybrid/rerank decision;
- eval queries;
- failure cases.

## Decision Checklist

- [ ] Keyword vs semantic use cases are separated.
- [ ] Metadata filters exist.
- [ ] Exact-match queries are protected.
- [ ] Stale and forbidden sources are tested.
- [ ] Retrieval eval exists before generation.
- [ ] Hybrid search is considered.

## Active Recall

1. What is an embedding?
2. Why does semantic similarity not equal correctness?
3. When is keyword search better than semantic search?
4. What is hybrid retrieval?
5. Why evaluate retrieval before generation?

## Next

Next chapter turns documents into an index: chunking, metadata, ingestion, reranking, and retrieval pipelines.
