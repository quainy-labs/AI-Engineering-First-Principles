# Chapter 34: Computer Use and Browser Automation

Chapter 33 handled state and approval. This chapter covers a special kind of tool use: controlling user interfaces such as browsers and desktop apps.

Computer use is powerful because it can operate systems without APIs. It is risky because UI state is fragile and side effects are easy.

## Real Problem

A company has an internal admin panel with no API.

An assistant needs to:

- open customer record;
- read status;
- fill support note;
- prepare update;
- ask human approval;
- submit.

Browser automation can help, but the UI may change, buttons may be ambiguous, and a wrong click can modify data.

## Bad Default Solution

Bad solution:

- give model browser access;
- ask it to complete task;
- let it click freely;
- skip screenshots/traces;
- skip approval.

This is unsafe automation.

## First Principles

Computer use loop:

```text
observe screen -> decide action -> click/type -> observe result -> continue/stop
```

Safe computer use needs:

- bounded goal;
- allowed sites/apps;
- no-go actions;
- step limit;
- screenshot trace;
- form validation;
- approval before submit;
- recovery path.

Prefer APIs when available. Use browser automation when API does not exist or human-like UI operation is the actual task.

## System Decision

Classify UI actions:

| Action | Rule |
|---|---|
| read page | usually allowed |
| fill draft field | allowed with trace |
| submit form | approval required |
| financial change | strict approval |
| delete/permission change | disallow or special flow |

## Minimal Build

Create `computer-use-policy.md`:

```text
Workflow:
Allowed apps/sites:
Forbidden actions:
Max steps:
Observation data:
Approval-before-submit:
Screenshot trace:
Recovery path:
Eval tasks:
```

## Failure Modes

Computer use fails when:

- UI changes break selector;
- model clicks wrong button;
- hidden modal changes state;
- credentials leak in trace;
- repeated action duplicates side effect;
- human cannot inspect what happened;
- submit happens without approval.

## Evaluation

Test:

- happy path;
- changed UI label;
- modal appears;
- permission denied;
- wrong page;
- form validation error;
- risky submit;
- max-step stop.

Measure:

- task success;
- wrong-click rate;
- approval accuracy;
- recovery success;
- trace completeness.

## Proof Artifact

Create `computer-use-policy.md`.

## Next

Next chapter defines agents after tools, registries, workflows, state, approval, and computer use are understood.
