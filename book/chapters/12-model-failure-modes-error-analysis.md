# Chapter 12: Model Failure Modes and Error Analysis

## Real Problem

Your structured extraction system works on demo cases. Then real tickets arrive.

Failures:

- wrong issue type;
- missing urgency;
- invalid JSON;
- refund request classified as billing question;
- dangerous request not escalated;
- model refuses harmless request.

Team says:

> Model is unreliable.

Maybe. But "unreliable" is not diagnosis. Need error analysis.

## Bad Default Solution

Bad response:

- add stronger instruction;
- change model;
- add "think carefully";
- test three examples;
- ship again.

This creates prompt churn. No one knows what improved.

Error analysis turns vague failure into categories, causes, and fixes.

## First Principles

Error analysis means:

1. collect failures;
2. label failure type;
3. identify cause;
4. choose fix;
5. retest same examples.

A model failure can come from:

- bad input;
- missing context;
- unclear task;
- weak schema;
- bad retrieval;
- conflicting sources;
- model limitation;
- wrong evaluation expectation.

Do not blame model until system is inspected.

## Error Taxonomy

Use categories:

| Error type | Meaning |
|---|---|
| Unsupported claim | model says fact not in context |
| Omission | model misses required fact |
| Wrong classification | label incorrect |
| Wrong extraction | field value incorrect |
| Invalid format | output fails schema |
| Over-refusal | refuses when should answer |
| Under-refusal | answers when should refuse |
| Context misuse | ignores or misreads evidence |
| Source confusion | cites wrong source |
| Tool misuse | wrong tool or arguments |
| Policy error | violates rule |

Every project should track top error types.

## System Decision

Fix depends on cause:

| Cause | Fix |
|---|---|
| Missing context | improve retrieval or input |
| Schema confusion | simplify schema |
| Ambiguous labels | merge or redefine labels |
| Invalid format | add validation/retry |
| Unsupported claims | require citations/refusal |
| Policy errors | move policy to code or human review |
| Tool misuse | improve tool interface |
| Edge cases | add eval examples |

Do not use same fix for every failure.

## Minimal Build

Create `model-error-report.md`:

```text
System:
Eval set size:
Pass rate:

Top errors:
1. error type:
   count:
   examples:
   likely cause:
   fix:

2. error type:
   count:
   examples:
   likely cause:
   fix:

Regression cases:
- 

Next experiment:
```

## Failure Modes

Error analysis fails when:

- failures are not saved;
- only funny examples reviewed;
- no expected output exists;
- errors are not categorized;
- cause assumed too quickly;
- prompt changed without retest;
- eval set changes during comparison;
- improvements measured on new easy examples.

## Evaluation

Use same eval set before and after fix.

Track:

- overall pass rate;
- pass rate by category;
- invalid output rate;
- refusal accuracy;
- unsupported claim rate;
- cost;
- latency.

Also track regression:

- cases fixed;
- cases newly broken.

AI systems often improve one behavior while hurting another.

## Improvement

Run improvement loop:

1. evaluate;
2. categorize errors;
3. choose one high-impact fix;
4. change system;
5. rerun eval;
6. inspect regressions;
7. document decision.

Examples:

- Many invalid JSON errors -> improve schema/validation, not retrieval.
- Many unsupported claims -> improve context/refusal/citation rules.
- Many wrong labels -> redefine labels or add examples.
- Many policy errors -> move decision into deterministic policy engine.

## Proof Artifact

Create `model-error-report.md`.

Include:

- eval set size;
- pass rate;
- top error categories;
- examples;
- likely causes;
- chosen fix;
- before/after results.

## Decision Checklist

- [ ] Failures are saved.
- [ ] Expected outputs exist.
- [ ] Error taxonomy used.
- [ ] Cause is identified.
- [ ] Fix matches cause.
- [ ] Same eval set rerun.
- [ ] Regressions checked.

## Active Recall

1. Why is "model unreliable" not enough diagnosis?
2. What are common model error categories?
3. Why can same prompt change fix one case and break another?
4. Why should eval set stay same before/after?
5. When should policy move from model to code?

## Next

Part 4 uses models with retrieval. Now that model interface and error analysis exist, next step is grounded AI systems.
