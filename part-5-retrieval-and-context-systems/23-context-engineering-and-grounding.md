# Chapter 23: Context Engineering and Grounding

Chapter 22 built RAG flow. This chapter controls the evidence packet given to model.

Retrieval is necessary. It is not enough.

## Real Problem

Your RAG system retrieves correct document, but answer is still wrong.

Inspection shows:

- correct snippet was included;
- irrelevant snippets were also included;
- stale document appeared above current document;
- model blended both;
- citation pointed to correct source, but answer used stale claim.

Context engineering decides what model sees, in what order, with what rules.

## Bad Default Solution

Bad solution:

```text
Take top 5 chunks and paste them into prompt.
```

This ignores:

- authority;
- freshness;
- source type;
- conflicting evidence;
- user permissions;
- context budget;
- citation requirement;
- refusal rule.

Context should be built, not dumped.

## First Principles

Grounding means generated answer is supported by provided evidence.

Context is not memory. Context is input to current model call.

Good context has:

- relevant evidence;
- source labels;
- authority order;
- freshness;
- enough surrounding text;
- no forbidden data;
- no unnecessary distractors;
- clear instruction on how to use evidence.

Grounded answer should answer only what evidence supports.

## System Decision

Create context policy:

```text
1. What sources are allowed?
2. What sources are preferred?
3. What sources are forbidden?
4. How are conflicts resolved?
5. How much context fits?
6. What metadata must be attached?
7. When should system refuse?
8. How should citations work?
```

Example authority order:

```text
1. Current approved policy
2. Product documentation
3. Internal playbook
4. Historical notes
5. Drafts excluded
```

Example refusal rule:

```text
If retrieved context does not contain direct support, answer:
"I do not have enough approved information to answer."
```

## Minimal Build

Create `context-policy.md`:

```text
Use case:
Allowed sources:
Forbidden sources:
Authority order:
Freshness rule:
Conflict rule:
Metadata required:
Max chunks:
Max tokens:
Citation format:
Refusal rule:
Compression rule:
```

Add context assembly example:

```text
[SOURCE 1]
title:
owner:
last_updated:
authority:
excerpt:

[SOURCE 2]
...
```

## Failure Modes

Grounding fails when:

- source labels are missing;
- context includes stale document;
- too many chunks distract model;
- context lacks exact answer;
- prompt allows outside knowledge;
- citations are not validated;
- evidence is compressed until meaning changes;
- conflicting sources have no resolution rule.

## Evaluation

Use context stress tests:

1. answer directly supported;
2. answer absent;
3. stale source present;
4. conflicting sources present;
5. irrelevant high-similarity distractor present;
6. malicious instruction inside document;
7. user asks outside permission scope;
8. answer requires multiple sources.

Expected behavior:

- answer with citation;
- refuse;
- prefer authoritative source;
- ignore malicious document text;
- respect access rules.

## Improvement

Improve grounding by:

- adding authority metadata;
- filtering stale sources;
- reducing distractors;
- adding citation validation;
- adding conflict rule;
- improving chunk boundaries;
- using reranking;
- separating source text from instructions.

## Proof Artifact

Create `context-policy.md`.

Include:

- source rules;
- authority order;
- freshness rule;
- conflict rule;
- citation format;
- refusal rule;
- context stress tests.

## Decision Checklist

- [ ] Allowed sources named.
- [ ] Forbidden sources named.
- [ ] Authority order exists.
- [ ] Freshness rule exists.
- [ ] Conflict rule exists.
- [ ] Citation format exists.
- [ ] Refusal rule exists.

## Active Recall

1. What is grounding?
2. Why can correct retrieval still produce wrong answer?
3. What is authority order?
4. Why must source labels stay in context?
5. How should system behave when evidence is missing?

## Next

Next chapter measures RAG. Grounding policy is only useful if system can prove answers are grounded.
