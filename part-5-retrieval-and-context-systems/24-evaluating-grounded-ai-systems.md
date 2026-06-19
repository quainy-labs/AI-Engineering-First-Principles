# Chapter 24: Evaluating Grounded AI Systems

Chapter 23 defined grounding rules. Now we test whether system follows them.

A fluent answer is not enough. A grounded system must prove where the answer came from and when it should not answer.

## Real Problem

A RAG demo looks good.

Five hand-picked questions work.

Then users ask:

- questions with no answer in docs;
- questions requiring two documents;
- questions where old docs conflict with new docs;
- questions with exact policy nuance;
- questions outside their access scope.

Trust breaks.

## Bad Default Solution

Bad evaluation:

- ask few easy questions;
- judge answer by vibes;
- ignore retrieved sources;
- ignore citations;
- ignore refusals;
- ignore cost and latency.

This hides real failures.

Evaluate grounded systems in two layers:

1. retrieval quality;
2. answer quality.

If answer is bad, first inspect retrieval.

## First Principles

RAG system can fail in retrieval or generation.

Retrieval evaluation asks:

- did system find right source?
- did correct source appear near top?
- did it retrieve stale or forbidden source?
- did it include distractors?

Generation evaluation asks:

- did answer address question?
- did answer use retrieved evidence?
- did it cite correctly?
- did it refuse when evidence absent?
- did it avoid unsupported claims?

Trust needs both.

## System Decision

Build eval set before optimizing.

Eval set should include:

- easy questions;
- normal questions;
- multi-source questions;
- stale-source traps;
- conflicting-source traps;
- should-refuse questions;
- permission-boundary questions.

For each question, define:

```text
Question:
User role:
Expected source:
Forbidden source:
Expected answer:
Must cite:
Should refuse:
Risk:
```

## Minimal Build

Create `grounded-eval-set.md`:

```text
Eval set size: 40

Categories:
- easy: 10
- normal: 10
- multi-source: 5
- conflicting-source: 5
- stale-source: 5
- should-refuse: 5

Metrics:
- retrieval top-1 accuracy
- retrieval top-3 recall
- answer relevance
- faithfulness
- citation accuracy
- refusal accuracy
- forbidden-source rate
- latency
- cost
```

## Failure Modes

Grounded eval fails when:

- only successful questions are included;
- expected sources are not labeled;
- no should-refuse cases exist;
- no stale/conflict cases exist;
- answer is judged without source inspection;
- evaluator rewards fluent unsupported answers;
- eval set changes during comparison;
- cost and latency are ignored.

## Evaluation Report

Run report:

```text
Version:
Date:
Corpus version:
Retriever version:
Prompt version:
Model:

Retrieval:
- top-1 accuracy:
- top-3 recall:
- forbidden-source rate:
- stale-source rate:

Generation:
- answer relevance:
- faithfulness:
- citation accuracy:
- refusal accuracy:
- unsupported claim rate:

System:
- average latency:
- p95 latency:
- average cost:
```

Version everything. Otherwise you cannot explain improvement.

## Improvement

Choose fix by failure layer:

| Symptom | Likely Layer | Fix |
|---|---|---|
| Correct source missing | retrieval | query/chunk/metadata/rerank |
| Correct source present, answer wrong | generation/context | prompt/context policy |
| Answer unsupported | grounding | citation/refusal validation |
| Refuses answerable question | retrieval/context | improve evidence selection |
| Answers unanswerable question | refusal/eval | stricter source rule |
| Too slow | system | cache, reduce top-k, optimize context |

## Proof Artifact

Create `grounded-eval-set.md` and `grounded-eval-report.md`.

Include:

- categorized questions;
- expected sources;
- forbidden sources;
- expected behavior;
- metrics;
- first run results;
- top failure categories.

## Decision Checklist

- [ ] Eval set has should-refuse cases.
- [ ] Expected sources are labeled.
- [ ] Forbidden sources are labeled.
- [ ] Retrieval and generation measured separately.
- [ ] Citation accuracy measured.
- [ ] Cost and latency measured.
- [ ] Results are versioned.

## Active Recall

1. Why evaluate retrieval separately from answer?
2. What is faithfulness?
3. Why include should-refuse questions?
4. What is forbidden-source rate?
5. Why version corpus, retriever, prompt, and model?

## Next

Next chapter turns a grounded assistant into a maintained knowledge product. Reliable answers need owners, feedback, freshness, and review loops.
