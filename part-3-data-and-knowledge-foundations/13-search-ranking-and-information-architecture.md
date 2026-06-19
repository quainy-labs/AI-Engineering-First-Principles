# Chapter 13: Search, Ranking, and Information Architecture

Chapter 12 made data reliable. This chapter makes knowledge findable.

Before semantic retrieval and RAG, learn search.

## Real Problem

Company has 500 documents:

- help center pages;
- release notes;
- onboarding guides;
- internal policies;
- product specs;
- troubleshooting docs.

User asks:

> How do I reset API key for enterprise workspace?

If system retrieves wrong document, model will likely produce wrong answer. Retrieval quality controls answer quality.

## Bad Default Solution

Bad solution:

- split documents randomly;
- embed everything;
- retrieve top 5 chunks;
- blame model when answer is wrong.

Often problem is not model. Problem is knowledge organization.

## First Principles

Search has three jobs:

1. represent information;
2. match query to candidates;
3. rank candidates by usefulness.

Keyword search matches words.

Ranking decides order.

Information architecture makes documents retrievable:

- good titles;
- clear headings;
- metadata;
- canonical pages;
- updated dates;
- ownership;
- tags;
- audience;
- product area;
- document type.

## System Decision

Start with retrieval design:

```text
Who asks questions?
What questions do they ask?
What documents should answer them?
What metadata helps filter?
Which sources are authoritative?
Which sources should be excluded?
How fresh must answer be?
```

Choose method:

| Query type | Good method |
|---|---|
| exact phrase, ID, code | keyword search |
| product + policy | metadata filter + search |
| latest answer | freshness filter |
| high-risk answer | authoritative source only |
| paraphrase/concept | later semantic retrieval |

Semantic retrieval comes later. Do not skip basic search.

## Minimal Build

Create `retrieval-design.md`:

```text
Document sources:
Canonical sources:
Excluded sources:
Metadata fields:
Authority order:
Freshness rule:

Test queries:
1. query:
   expected document:
   reason:
```

## Failure Modes

Search fails when:

- documents are stale;
- duplicate pages conflict;
- chunks lose headings;
- metadata missing;
- internal drafts rank above approved docs;
- query words do not match document words;
- ranking ignores authority.

## Evaluation

Build retrieval test set:

- 20 real questions;
- expected source document;
- acceptable alternate documents;
- forbidden documents.

Metrics:

- top-1 accuracy;
- top-3 recall;
- forbidden-source rate;
- stale-source rate.

## Improvement

Improve before touching model:

- fix document titles;
- add metadata;
- remove duplicates;
- create canonical pages;
- improve headings;
- add filters;
- adjust ranking;
- add synonym mapping.

## Proof Artifact

Create `retrieval-design.md` and `keyword-search-eval.md`.

## Decision Checklist

- [ ] Canonical sources are known.
- [ ] Stale sources are handled.
- [ ] Metadata exists.
- [ ] Authority order exists.
- [ ] Test queries exist.
- [ ] Forbidden sources are named.
- [ ] Ranking is evaluated.

## Active Recall

1. Why should retrieval be evaluated before generation?
2. Why is keyword search still useful?
3. What is information architecture?
4. How can stale documents break AI answers?
5. What is forbidden-source rate?

## Next

Part 3 built reliable information foundations. Next part should introduce models as components: what language models do, how context works, how to design model interfaces, how to choose models, and when adaptation is needed.

