# Chapter 13: Search, Ranking, and Information Architecture

Chapter 12 made information reliable and traceable:

```text
quality checks
-> lineage
-> versioned artifacts
-> quality gates
-> rollback
```

This chapter asks how people and systems find the right information.

Before retrieval-augmented generation, before embeddings, before vector databases, before agent memory, there is a simpler truth:

> If the system retrieves the wrong information, the model will usually produce the wrong answer.

Search, ranking, and information architecture decide what context reaches the model.

## Concept Overview

Search finds candidate information.

Ranking orders candidates by usefulness.

Information architecture makes information easier to find in the first place.

Together, they answer:

```text
What sources are searchable?
Which sources are authoritative?
Which sources are stale or forbidden?
What metadata can filter results?
How should documents be titled and organized?
What result should rank first?
What result should never be used?
How do we know retrieval is working?
```

This chapter is not yet about advanced RAG. It is about the foundation that RAG depends on.

Good retrieval starts before embeddings. It starts with clear documents, good titles, useful headings, canonical sources, metadata, freshness, ownership, and ranking rules.

The first principle:

> Retrieval is not magic. It is information design plus matching plus ranking.

## Why It Exists

Imagine a company has 500 documents:

- help-center pages;
- release notes;
- onboarding guides;
- internal policies;
- product specs;
- troubleshooting docs;
- draft proposals;
- archived migration notes;
- customer-specific runbooks.

A user asks:

> How do I reset an API key for an enterprise workspace?

There may be many related documents:

- old API key article;
- new enterprise workspace guide;
- internal security policy;
- release note about key rotation;
- draft admin UI guide;
- customer-specific exception note;
- troubleshooting doc for failed resets.

If search retrieves the old article first, the model may confidently answer with outdated steps.

If search retrieves internal policy for a customer-facing user, the system may leak information.

If search retrieves a release note instead of the canonical guide, the answer may be incomplete.

The model is downstream of retrieval. It can only reason over the context it receives.

This chapter exists so learners stop treating retrieval as:

```text
split documents -> embed chunks -> top 5 -> generate answer
```

And start treating retrieval as a product and system design problem.

## Mental Model

Think of search like a library.

If books have no titles, shelves have no labels, old editions sit beside new editions, private documents are mixed with public ones, and drafts are indistinguishable from official policy, even a brilliant librarian will struggle.

The mental model:

```text
information architecture
-> candidate generation
-> filtering
-> ranking
-> context selection
-> model reasoning
```

Information architecture shapes the corpus before search happens.

Candidate generation finds possible matches.

Filtering removes sources that should not be used.

Ranking orders the remaining candidates.

Context selection chooses what enters the model prompt.

The model reasons over selected context, not the whole company knowledge base.

This creates a clear debugging path:

```text
Bad answer?
Did the right source exist?
Was it indexed?
Was it allowed?
Was it retrieved?
Was it ranked high enough?
Was enough context passed?
Did the model use it correctly?
```

Do not start by blaming the model. First inspect retrieval.

## Internal Mechanics

Search and ranking start with the corpus.

### Information Architecture

Information architecture is how knowledge is organized so people and systems can find it.

Useful document qualities:

- clear title;
- descriptive headings;
- stable URL or source ID;
- owner;
- audience;
- product area;
- document type;
- authority level;
- freshness timestamp;
- permission labels;
- canonical or duplicate status;
- archived or active status.

Poor information architecture creates retrieval problems:

- vague titles like "FAQ";
- many duplicate pages;
- old docs not archived;
- drafts mixed with approved docs;
- missing product area;
- no audience label;
- headings that do not describe sections;
- long pages with unrelated topics;
- missing source owner.

Search quality often improves when documents improve.

### Candidate Generation

Candidate generation finds possible results.

Common methods:

```text
keyword search
metadata-filtered search
semantic search
hybrid search
exact ID lookup
database/API lookup
```

Keyword search is still useful.

It works well for:

- product names;
- error codes;
- exact phrases;
- IDs;
- API names;
- feature names;
- commands;
- dates;
- compliance terms.

Semantic search is useful when:

- users paraphrase;
- concepts matter more than exact words;
- the same idea appears in different language;
- the corpus is written inconsistently.

Hybrid search combines keyword and semantic signals. Later retrieval chapters will build on this. But even hybrid search needs metadata, authority, freshness, and evaluation.

### Filtering

Filtering removes candidates that should not be used.

Common filters:

- user permission;
- workspace or tenant;
- product area;
- audience;
- document type;
- language;
- freshness;
- active status;
- authority level;
- source system;
- region;
- customer segment.

Filtering should happen before the model sees context.

Example:

```text
user = customer
allowed audience = public docs
exclude internal policies, drafts, private runbooks
```

Filtering prevents many failures before generation begins.

### Ranking

Ranking decides order.

Ranking should consider more than text similarity.

Useful ranking signals:

- query match;
- semantic relevance;
- title match;
- heading match;
- source authority;
- freshness;
- audience match;
- product area match;
- document type;
- usage feedback;
- citation success;
- known canonical source;
- forbidden-source penalty.

Example:

```text
query: reset API key for enterprise workspace

Result A:
title: Reset API Keys
updated: 2024
audience: all users
authority: old help article

Result B:
title: Enterprise Workspace API Key Management
updated: 2026
audience: enterprise admins
authority: canonical guide
```

Even if both match, Result B should rank higher for this query.

### Context Selection

Search may return many results, but the model receives limited context.

Context selection decides:

- how many chunks or documents to include;
- whether to include surrounding sections;
- whether to include titles and metadata;
- whether to include citations;
- whether to include conflicting sources;
- whether to ask a clarifying question instead.

Do not pass isolated text without source context.

Better context includes:

```text
title
section heading
source URI
updated date
authority level
chunk text
```

### Retrieval Evaluation

Retrieval should be evaluated before generation.

Build test queries:

```text
query
expected source
acceptable alternate sources
forbidden sources
reason
```

Useful metrics:

- top-1 accuracy;
- top-3 recall;
- top-5 recall;
- forbidden-source rate;
- stale-source rate;
- no-result rate;
- duplicate-source rate;
- canonical-source rate.

If retrieval fails, generation will be unstable no matter how carefully the prompt is written.

## Real-World Example

Imagine a company knowledge assistant for product support.

Question:

> How do I rotate an API key for an enterprise workspace?

Corpus contains:

```text
Document A: API Key FAQ
status: active
audience: all users
updated: 2023

Document B: Enterprise Workspace API Key Management
status: active
audience: enterprise admins
updated: 2026
authority: canonical

Document C: Internal Security Escalation Policy
status: active
audience: internal support
updated: 2026

Document D: API Key Rotation Draft
status: draft
audience: internal docs team
updated: 2026
```

Bad retrieval:

```text
top result: Document A
reason: exact title match "API Key"
```

The model answers with outdated generic steps.

Better retrieval:

```text
filter:
- status=active
- audience allowed for user
- product_area=enterprise_workspace

ranking:
- canonical source boost
- freshness boost
- title/heading match
- enterprise audience match

top result: Document B
```

The model now receives the right source:

```text
Title: Enterprise Workspace API Key Management
Section: Rotating API Keys
Updated: 2026-05-10
Authority: canonical
Audience: enterprise admins
```

The answer improves because the retrieval improved.

## Common Mistakes and Failure Modes

Mistake 1: Embedding everything without information architecture

Embeddings cannot fully compensate for messy, stale, duplicated, or poorly labeled documents.

Mistake 2: Random chunking destroys meaning

If chunks lose title, heading, section path, or surrounding context, search may retrieve fragments that look relevant but lack the necessary conditions.

Mistake 3: Ranking by similarity only

The most text-similar result is not always the most authoritative, fresh, or safe result.

Mistake 4: No forbidden-source list

Some documents should not be used for some users or tasks, even if they match the query.

Mistake 5: Stale pages remain active

Old documents can outrank newer sources if freshness and canonical status are ignored.

Mistake 6: No retrieval eval set

Without expected-source tests, teams tune prompts while retrieval keeps failing.

Mistake 7: Missing metadata

No product area, audience, owner, freshness, or authority metadata means ranking has little signal beyond text match.

Mistake 8: Blaming the model for bad context

If the model receives the wrong source, the first bug is retrieval, not generation.

## System Design Decisions

Start by defining the corpus.

```text
Which sources are searchable?
Which sources are canonical?
Which sources are excluded?
Which sources are internal only?
Which sources are customer-facing?
Which sources are stale or archived?
```

Define metadata:

```text
title
source_uri
owner
audience
product_area
document_type
authority_level
updated_at
status
permission_labels
language
region
```

Define retrieval behavior:

```text
Which filters apply before search?
Which ranking signals matter?
Which documents receive authority boost?
Which documents receive freshness boost?
Which sources should be penalized?
When should the system ask a clarifying question?
```

Define search method:

| Query type | Useful method |
|---|---|
| exact ID, error code, API name | keyword or exact lookup |
| current policy | metadata filter + authoritative source |
| product how-to | keyword + metadata + ranking |
| paraphrased concept | semantic or hybrid search |
| customer-specific fact | database/API, not document search |
| high-risk answer | authoritative source only |

Define evaluation:

```text
20-50 real user questions
expected source
acceptable alternates
forbidden sources
stale-source check
top-k target
owner for failures
```

Improve retrieval before improving generation when:

- expected source is missing from top results;
- stale source appears high;
- forbidden source appears;
- duplicate pages dominate;
- chunks lose necessary context;
- metadata filters are unavailable.

## Build Artifact

Create `retrieval-design.md` and `keyword-search-eval.md` for one AI workflow.

Use this template for `retrieval-design.md`:

```text
Retrieval Design

Workflow:
User goal:
Main question types:

Corpus:
- source:
  included:
  reason:
  audience:
  owner:

Canonical Sources:
- source:
  authority reason:

Excluded Sources:
- source:
  exclusion reason:

Metadata Fields:
- field:
  used for:
  required: yes/no

Filters:
- filter:
  applied before search:
  reason:

Ranking Signals:
- signal:
  boost or penalty:
  reason:

Context Selection:
- include:
  reason:
- exclude:
  reason:

Failure Behavior:
- condition:
  system response:
```

Use this template for `keyword-search-eval.md`:

```text
Keyword Search Eval

Corpus version:
Index version:
Date:

Test Queries:
- query:
  user type:
  expected source:
  acceptable alternates:
  forbidden sources:
  reason:
  top result:
  passed:

Metrics:
- top-1 accuracy:
- top-3 recall:
- forbidden-source rate:
- stale-source rate:
- no-result rate:
```

Example:

```text
Query: reset API key for enterprise workspace
Expected source: Enterprise Workspace API Key Management
Acceptable alternates: Enterprise Admin Security Guide
Forbidden sources: API Key Rotation Draft, Internal Security Escalation Policy
Reason: user needs current customer-facing enterprise admin steps
```

The artifact is complete when another engineer can see what should be searched, what should be filtered, what should rank highly, and how retrieval quality is measured.

## Active Recall Questions

1. Why should retrieval be evaluated before generation?
2. What is information architecture?
3. Why is keyword search still useful in AI systems?
4. Why is similarity alone not enough for ranking?
5. What metadata helps retrieval quality?
6. How can stale documents break AI answers?
7. What is a forbidden-source rate?
8. When should a system use database/API lookup instead of document search?

## Summary

Search finds candidates. Ranking orders them. Information architecture makes the right sources findable before search begins. Retrieval quality controls answer quality because the model reasons over the context it receives.

Strong AI systems do not simply embed everything and hope. They define canonical sources, exclude unsafe or stale sources, preserve metadata, evaluate retrieval, and improve information organization before blaming the model.

The most important rule:

> If the wrong source reaches the model, the answer is already in danger.

## Preview of Next Chapter

Part 3 built the information foundation:

```text
data, documents, and state
-> ingestion and parsing
-> schemas and contracts
-> quality, lineage, and versioning
-> search and ranking
```

Part 4 moves into models as components.

Next, Chapter 14 asks what language models actually do and what they should not own. This matters because once the system has reliable information, the next engineering decision is how to bound the model's role: understanding, extracting, classifying, drafting, transforming, and reasoning over supplied context without becoming the database, policy engine, permission system, or final authority.
