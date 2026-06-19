# Project 3: Structured Extraction System

## Goal

Convert messy text into validated structured data.

## Output

Create:

```text
projects/structured-extraction/
  model-capability-boundary.md
  context-budget.md
  model-interface-spec.md
  model-error-report.md
```

## Required System

Input:

- messy text from real workflow.

Output:

- validated JSON that downstream code can use.

Examples:

- support ticket classifier;
- invoice extractor;
- contract clause labeler;
- research note tagger;
- lead qualification extractor.

## Evaluation

Use 30 examples:

- 10 easy;
- 10 normal;
- 5 edge;
- 5 should-refuse.

Measure:

- valid JSON rate;
- field accuracy;
- refusal accuracy;
- invalid output rate;
- human review rate.

## Final Artifact

Ship extraction report:

- schema;
- examples;
- eval results;
- failures;
- fixes;
- remaining risks.

