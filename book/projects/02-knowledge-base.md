# Project 2: Knowledge Base

## Goal

Build information foundation for a real workflow.

Do not build chat yet.

Build reliable retrieval.

## Output

Create:

```text
projects/knowledge-base/
  information-map.md
  output-schema.md
  retrieval-design.md
  semantic-search-eval.md
```

## Step 1: Information Map

Identify:

- structured data;
- documents;
- state;
- source owners;
- freshness rules;
- access rules;
- unknown information.

Pass:

- exact facts have source of truth;
- documents have authority order;
- state has lifespan.

## Step 2: Output Schema

Design one structured output needed by workflow.

Examples:

- support ticket classification;
- invoice field extraction;
- policy risk label;
- document summary record;
- search result object.

Pass:

- schema fields typed;
- validation rules defined;
- invalid examples included.

## Step 3: Retrieval Design

Create document metadata:

- title;
- source;
- owner;
- audience;
- product area;
- last updated;
- authority level;
- document type.

Create 20 test queries with expected documents.

Pass:

- canonical sources known;
- stale/forbidden sources named;
- expected results defined.

## Step 4: Semantic Search Eval

Compare:

- keyword retrieval;
- semantic retrieval;
- hybrid retrieval.

Track:

- top-1 accuracy;
- top-3 recall;
- stale-source rate;
- forbidden-source rate;
- distractor rate.

## Final Review

Answer:

1. What information does system need?
2. What is source of truth?
3. What structure does downstream code need?
4. Which queries fail keyword search?
5. Which queries fail semantic search?
6. What retrieval method is best for this workflow?

If these are clear, workflow is ready for model interface design.

