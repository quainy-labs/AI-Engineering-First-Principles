# Chapter 7: Search, Ranking, and Information Architecture

## Real Problem

A company has 500 documents:

- help center pages;
- release notes;
- onboarding guides;
- internal policies;
- product specs;
- troubleshooting docs.

User asks:

> How do I reset API key for enterprise workspace?

If system retrieves wrong document, model may produce wrong answer. Retrieval quality controls answer quality.

Before vector databases, learn search.

## Bad Default Solution

Bad solution:

- split documents randomly;
- embed everything;
- retrieve top 5 chunks;
- blame model when answer is wrong.

Often problem is not model. Problem is knowledge organization.

If documents lack titles, metadata, versioning, authority, and clear structure, retrieval becomes guesswork.

## First Principles

Search has three jobs:

1. represent information;
2. match query to candidates;
3. rank candidates by usefulness.

Keyword search matches words.

Semantic search matches meaning.

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

Bad content structure creates bad AI systems.

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

Then choose retrieval method:

| Query type | Good method |
|---|---|
| exact phrase, ID, code | keyword search |
| concept, paraphrase | semantic search |
| product + policy | metadata filter + search |
| latest answer | freshness filter |
| high-risk answer | authoritative source only |

Keyword search is not obsolete. It is precise, transparent, cheap, and useful.

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
2. query:
   expected document:
   reason:
```

Metadata example:

```text
title:
source:
owner:
product_area:
audience:
last_updated:
authority_level:
document_type:
```

## Failure Modes

Retrieval fails when:

- documents are stale;
- duplicate pages conflict;
- chunks lose headings;
- metadata missing;
- internal drafts rank above approved docs;
- query words do not match document words;
- semantic search retrieves related but wrong content;
- ranking ignores authority.

## Evaluation

Build small retrieval test set:

- 20 real questions;
- expected source document;
- acceptable alternate documents;
- forbidden documents.

Metrics:

- top-1 accuracy;
- top-3 recall;
- forbidden-source rate;
- stale-source rate.

Pass if correct source appears early enough for system to use.

Fail if retrieval "looks relevant" but misses authoritative answer.

## Improvement

Improve retrieval before touching model:

- fix document titles;
- add metadata;
- remove duplicate/stale docs;
- create canonical pages;
- improve headings;
- add filters;
- adjust ranking;
- add synonym mapping.

High-quality retrieval starts in content design, not vector math.

## Proof Artifact

Create `retrieval-design.md` with:

- document source list;
- metadata schema;
- authority order;
- freshness rule;
- 20-query retrieval eval.

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

Next chapter adds semantic retrieval. Same retrieval principles remain, but matching shifts from exact words to meaning.
