# Chapter 8: Embeddings and Semantic Retrieval

## Real Problem

User asks:

> How do I change who can access billing?

Document title says:

> Manage invoice administrator permissions

Keyword search may miss this. Words differ, meaning overlaps.

Semantic retrieval helps when users and documents describe same idea in different language.

## Bad Default Solution

Bad solution:

- assume embeddings solve search;
- use vector search for every query;
- ignore metadata and authority;
- never inspect retrieved results;
- measure final answer only.

Semantic retrieval improves some failures and creates new ones.

It may retrieve conceptually related but wrong documents. It may miss exact identifiers. It may blur distinctions between similar policies.

## First Principles

Embedding is numeric representation of meaning.

A model converts text into vector:

```text
"reset API key" -> [0.12, -0.44, 0.08, ...]
```

Texts with similar meaning should have nearby vectors.

Semantic retrieval process:

1. embed documents;
2. embed query;
3. compare query vector to document vectors;
4. return nearest documents;
5. rank/filter;
6. pass selected context to next system step.

The core idea:

> Search by meaning instead of exact words.

## System Decision

Use semantic retrieval when:

- query language differs from document language;
- users ask broad conceptual questions;
- document set is too large for manual navigation;
- synonyms and paraphrases are common;
- exact keyword is unknown.

Do not rely only on semantic retrieval when:

- query has exact ID;
- answer depends on newest source;
- policy distinctions are subtle;
- permissions matter;
- source authority matters;
- query requires database fact.

Best production systems often combine:

```text
metadata filters + keyword search + semantic retrieval + reranking
```

## Minimal Build

Create `semantic-search-eval.md`:

```text
Corpus:
Embedding model:
Metadata filters:
Similarity metric:
Top-k:

Query 1:
Expected result:
Actual top 5:
Good:
Bad:
Fix:
```

Evaluate both:

- semantic wins: where keyword failed;
- semantic failures: where meaning match was too loose.

## Failure Modes

Semantic retrieval fails when:

- chunks are too small and lose context;
- chunks are too large and dilute meaning;
- similar concepts are confused;
- metadata filters missing;
- old docs are semantically close;
- query asks for exact value;
- embedding model poorly matches domain language;
- retrieval top-k includes distractors.

Example:

Query:

> Can contractors see payroll?

Semantic search may retrieve general access policy, but correct answer may live in payroll permissions doc. Authority and metadata matter.

## Evaluation

Use retrieval eval from Chapter 7 and compare:

- keyword only;
- semantic only;
- hybrid.

Metrics:

- top-1 source accuracy;
- top-3 source recall;
- stale-source rate;
- forbidden-source rate;
- distractor rate.

Do not judge by vibes. Inspect results.

## Improvement

Improve semantic retrieval by:

- adding metadata filters;
- improving chunk strategy;
- preserving titles/headings;
- removing stale docs;
- using hybrid retrieval;
- adding reranking;
- creating domain glossary;
- tuning top-k.

If retrieval still fails, document architecture may be problem.

## Proof Artifact

Create `semantic-search-eval.md` with:

- corpus description;
- embedding model;
- retrieval settings;
- 20 query eval;
- keyword vs semantic comparison;
- improvement notes.

## Decision Checklist

- [ ] Semantic search solves real keyword failure.
- [ ] Metadata filters still apply.
- [ ] Authority order still applies.
- [ ] Exact queries still use exact lookup/search.
- [ ] Retrieval eval compares keyword/semantic/hybrid.
- [ ] Distractors are tracked.
- [ ] Improvements are evidence-based.

## Active Recall

1. What does embedding represent?
2. When does semantic retrieval help?
3. When can semantic retrieval hurt?
4. Why combine metadata with vectors?
5. Why compare keyword, semantic, and hybrid retrieval?

## Next

Part 3 begins models as components. With information foundations in place, next question is what language models actually do inside these systems.
