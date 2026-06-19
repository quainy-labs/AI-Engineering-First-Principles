# Chapter 3: From Messy Workflow to System Boundary

## Real Problem

Someone says:

> Our team spends too much time reviewing compliance documents.

This is not yet system requirement. It is pain.

To build useful AI system, convert pain into workflow:

- who starts work;
- what input arrives;
- what steps happen;
- what decisions happen;
- what tools are used;
- what output gets delivered;
- what errors matter;
- what should remain human-owned.

System boundary decides what belongs inside product and what stays outside.

## Bad Default Solution

Bad solution:

- upload documents to model;
- ask for compliance issues;
- show generated summary.

Maybe useful demo. Not enough system.

Missing:

- document types;
- policy source;
- risk categories;
- reviewer responsibility;
- citation requirements;
- approval flow;
- audit trail;
- escalation path;
- measurement.

Without boundary, every edge case becomes surprise.

## First Principles

A workflow is repeated transformation:

```text
input -> steps -> decisions -> output
```

A system boundary defines:

- what system accepts;
- what system produces;
- what system controls;
- what system refuses;
- what humans decide;
- what external tools provide.

AI system boundary matters more than model capability. Powerful model inside unclear boundary creates unreliable product.

## System Decision

Map workflow in five layers:

1. Trigger: what starts work?
2. Input: what information arrives?
3. Transformation: what changes?
4. Decision: what must be judged?
5. Output: what leaves system?

Then mark ownership:

- code;
- retrieval;
- model;
- human;
- external service.

Example:

| Step | Owner | Notes |
|---|---|---|
| Upload contract | User | Input |
| Detect contract type | Model | Classification with confidence |
| Retrieve policy | Code + search | Must use approved source |
| Identify risky clauses | Model + retrieval | Must cite clause and policy |
| Approve final risk rating | Human | Human owns decision |
| Store review record | Code | Audit trail |

## Minimal Build

Create workflow map:

```text
Trigger:
User:
Input:
Output:
Step 1:
Step 2:
Step 3:
Decision points:
Human approval points:
Data sources:
External tools:
Out of scope:
Failure modes:
```

Out of scope is critical. It prevents system from pretending to solve everything.

## Failure Modes

Workflow design fails when:

- input types are unknown;
- output has no owner;
- model makes hidden decision;
- human approval added only after failure;
- system accepts tasks it cannot verify;
- out-of-scope cases are not rejected;
- logs do not capture decision path.

## Evaluation

Test boundary with 10 real examples:

- 6 normal cases;
- 2 edge cases;
- 2 out-of-scope cases.

Pass if system behavior is clear for all 10 before implementation.

Fail if team says:

> model will handle it.

That phrase often hides missing design.

## Improvement

If boundary is unclear:

- narrow input type;
- reduce output promise;
- add human review;
- make citations required;
- add confidence threshold;
- refuse unsupported cases;
- log every decision.

Smaller reliable system beats broad vague assistant.

## Proof Artifact

Create `workflow-boundary.md`.

Include:

- workflow map;
- ownership table;
- out-of-scope list;
- approval points;
- 10-example boundary test.

## Decision Checklist

- [ ] Workflow trigger is clear.
- [ ] Inputs are known.
- [ ] Outputs are known.
- [ ] Each step has owner.
- [ ] Human approval points are explicit.
- [ ] Out-of-scope cases are defined.
- [ ] Boundary tested with real examples.

## Active Recall

1. What is system boundary?
2. Why does workflow come before model?
3. What does out-of-scope protect against?
4. Why should each step have owner?
5. Why test boundary before implementation?

## Next

Next chapter defines baseline and impact. Without measurement, system is only story.

