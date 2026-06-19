# Project 1: Workflow Teardown

## Goal

Analyze one real workflow and decide where AI can create useful, measurable improvement.

Do not build model yet.

Build clarity.

## Output

Create folder:

```text
projects/workflow-teardown/
```

Inside it, produce:

```text
system-brief.md
ai-necessity-table.md
workflow-boundary.md
baseline-and-eval-plan.md
```

## Choose Workflow

Pick workflow with:

- repeated work;
- real user;
- messy information;
- visible output;
- measurable baseline;
- low enough risk for first project.

Good examples:

- answering repeated support tickets;
- summarizing research notes;
- reviewing policy documents;
- extracting invoice data;
- creating first draft of sales follow-up;
- searching internal docs;
- triaging inbound requests.

Avoid:

- vague "personal productivity";
- high-stakes medical/legal decisions;
- fully automated financial approvals;
- tasks with no real user;
- tasks where simple form solves problem.

## Step 1: System Brief

Create `system-brief.md`.

Template:

```text
System name:
User:
Current workflow:
Pain:
Baseline:
Input:
Output:
Information sources:
AI role:
Non-AI role:
Human role:
Success metric:
Failure modes:
Proof artifact:
```

Pass criteria:

- real user named;
- current workflow specific;
- baseline measurable;
- AI role narrow;
- human role explicit.

## Step 2: AI Necessity Table

Create `ai-necessity-table.md`.

Template:

| Task | Best method | Reason | Risk if AI owns it |
|---|---|---|---|
|  | Deterministic / retrieval / model / human |  |  |

Use categories:

- deterministic code;
- database lookup;
- search/retrieval;
- structured extraction;
- generation;
- tool action;
- human decision.

Pass criteria:

- exact rules assigned to code;
- permissions assigned to code/human;
- ambiguous language tasks assigned to AI only where useful;
- risky decisions assigned to human.

## Step 3: Workflow Boundary

Create `workflow-boundary.md`.

Template:

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

Then add ownership table:

| Step | Owner | Notes |
|---|---|---|
|  | Code / retrieval / model / human / external service |  |

Pass criteria:

- every step has owner;
- approval points explicit;
- out-of-scope cases listed;
- data sources named.

## Step 4: Baseline and Eval Plan

Create `baseline-and-eval-plan.md`.

Template:

| Metric | Current baseline | Target | How measured |
|---|---:|---:|---|
| Work metric |  |  |  |
| Quality metric |  |  |  |
| System metric |  |  |  |

Then define eval set:

```text
Easy cases: 10
Normal cases: 10
Edge cases: 5
Should-refuse cases: 5
```

Pass criteria:

- at least one work metric;
- at least one quality metric;
- at least one system metric;
- should-refuse cases included;
- target defined.

## Final Review

Answer:

1. What should improve?
2. Who benefits?
3. What is baseline?
4. What should AI do?
5. What should AI not do?
6. What can fail?
7. What proof will show impact?

If these answers are clear, workflow is ready for Part 2.

If not, do not build. Observe more real work.

