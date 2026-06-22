# Chapter 34: Computer Use and Browser Automation

Chapter 33 separated memory, state, audit, and approval.

This chapter covers a special kind of tool use: controlling user interfaces such as browsers and desktop apps.

Computer use is powerful because it can operate systems without APIs. It is risky because UI state is fragile and side effects are easy.

## Concept Overview

Computer use means an AI system can observe and operate a user interface.

Examples:

- open browser page;
- read visible content;
- click buttons;
- type into fields;
- navigate an admin panel;
- download a file;
- prepare form submission;
- use a desktop app.

The basic loop:

```text
observe screen
-> decide next UI action
-> click/type/navigate
-> observe result
-> continue, stop, or ask approval
```

The first principle:

> Prefer APIs when available. Use computer use when UI operation is necessary or no safe API exists.

Browser automation should not be a shortcut around proper tool design. It is a fallback for systems that only expose UI or workflows where visual UI state matters.

## Why It Exists

A company has an internal admin panel with no API.

Assistant needs to:

- open customer record;
- read status;
- fill support note;
- prepare update;
- ask human approval;
- submit.

Browser automation can help.

But UI may change. Buttons may be ambiguous. Hidden modals may appear. A wrong click can modify data. A repeated submit can duplicate side effects.

Bad default:

- give model browser access;
- ask it to complete task;
- let it click freely;
- skip screenshots/traces;
- skip approval.

This is unsafe automation.

This chapter exists because UI-driving AI needs stricter boundaries than normal read-only tools.

## Mental Model

Think of computer use as a remote assistant operating a screen while you watch.

You would not say:

```text
Go do whatever is needed in admin panel.
```

You would give:

- target site;
- task boundary;
- no-go actions;
- max steps;
- when to stop;
- what to show before submit;
- how to recover from error.

Mental model:

```text
bounded goal
-> allowed app/site
-> observe screen
-> propose next action
-> execute safe UI step
-> screenshot trace
-> approval before side effect
-> submit or stop
```

Computer use should be inspectable. If user cannot see what happened, trust is weak.

## Internal Mechanics

Computer use has a loop and a policy.

### Observation

System observes UI state.

Observation may include:

- screenshot;
- visible text;
- DOM accessibility tree;
- URL;
- focused element;
- form values;
- modal state;
- browser tab state.

Observation should avoid leaking secrets into logs.

### Action

Possible UI actions:

- click;
- type;
- scroll;
- select;
- navigate;
- upload;
- download;
- submit.

Not all actions have same risk.

| UI Action | Default Rule |
|---|---|
| Read page | Usually allowed with permission |
| Fill draft field | Allowed with trace |
| Navigate | Allowed within approved domains |
| Download | Needs data policy |
| Submit form | Approval required |
| Financial change | Strict approval |
| Permission change | Strict approval or disallow |
| Delete | Disallow or special flow |

### Bounded Goal

Goal must be narrow.

Weak:

```text
Handle this customer in admin panel.
```

Better:

```text
Open customer record C-104, read current plan, prepare internal note draft, stop before submit.
```

Bounded goal prevents wandering.

### Allowed Sites and Apps

Define:

```text
allowed domains
allowed apps
allowed pages
forbidden pages
credential handling
data download rules
```

Do not let automation browse anywhere unless workflow needs it.

### Step Limits

Computer use needs max steps.

Example:

```text
max steps: 12
if not completed: stop and ask human
```

Step limits prevent loops.

### Approval Before Submit

Submitting is a boundary.

Before submit, show:

```text
target app
page
form fields
changed values
side effects
risk
submit button label
```

Approval should be tied to exact UI action.

If form values change after approval, approval resets.

### Screenshot Trace

Trace should record:

- screenshots or safe redacted snapshots;
- action taken;
- UI element selected;
- timestamp;
- result;
- approval state.

Trace helps debug wrong clicks and prove what happened.

Sensitive data must be redacted where needed.

### Recovery Path

UI automation fails often.

Recovery cases:

- selector changed;
- modal appears;
- login expires;
- permission denied;
- validation error;
- wrong page;
- network timeout;
- duplicate submit warning.

Define:

```text
retry?
stop?
ask human?
refresh?
go back?
open manual handoff?
```

### API Preference

If API exists, prefer API for write actions.

APIs are easier to:

- validate;
- test;
- retry;
- log;
- authorize;
- make idempotent.

Use UI automation when:

- no API exists;
- UI state is the task;
- legacy system only supports UI;
- human-like browser operation is required.

## Real-World Example

Use case:

```text
Internal admin panel with no API
```

Task:

```text
Add internal support note to customer C-104.
```

Good policy:

```text
Allowed site: admin.company.internal
Allowed pages: customer profile, support notes
Forbidden actions: delete customer, change plan, grant admin, issue refund
Max steps: 15
Approval required: before clicking Submit Note
Trace: screenshot before and after each form change, redacted
Recovery: stop on modal, permission denied, or unexpected page
```

Good flow:

```text
open admin panel
-> search customer C-104
-> read visible account status
-> navigate support notes
-> fill note draft
-> show note and submit button to user
-> user approves
-> click submit
-> capture result
-> log trace
```

Bad flow:

```text
model clicks through admin panel freely and submits without approval
```

The difference is control.

## Common Mistakes and Failure Modes

Mistake 1: Browser access treated like normal tool

Computer use can touch many UI paths. It needs allowed apps, no-go actions, and step limits.

Mistake 2: No approval before submit

Submit buttons create side effects. User should approve concrete form values.

Mistake 3: No screenshot trace

Without trace, wrong-click debugging becomes guesswork.

Mistake 4: UI changes not handled

Labels, modals, layout, and validation can change. Automation needs recovery path.

Mistake 5: Credentials leak in trace

Screenshots and DOM snapshots can contain secrets.

Mistake 6: Repeated submit duplicates side effect

UI actions often lack idempotency. Be careful with retries.

Mistake 7: Wrong page or wrong account

Always verify target resource before write.

Mistake 8: Using browser automation when API exists

APIs are safer for most structured actions.

## System Design Decisions

Define workflow:

```text
What UI task is allowed?
What goal is out of scope?
What resource must be verified?
```

Define allowed environment:

```text
allowed apps/sites
allowed pages
forbidden pages
credential rules
download rules
```

Define action policy:

```text
read allowed?
draft fill allowed?
submit requires approval?
delete disallowed?
financial/permission changes special flow?
```

Define limits:

```text
max steps
max time
max retries
stop conditions
```

Define trace:

```text
screenshot cadence
redaction
action log
approval capture
viewer permissions
retention
```

Define recovery:

```text
unexpected modal
wrong page
validation error
login expired
permission denied
connector/browser crash
```

The policy should make UI automation boring and inspectable.

## Build Artifact

Create `computer-use-policy.md`.

Use this template:

```text
Computer Use Policy

Workflow:
User goal:

Allowed Apps/Sites:
- app/site:
  allowed pages:
  forbidden pages:

Allowed Actions:
- action:
  condition:

Forbidden Actions:
- action:
  reason:

Resource Verification:
- resource:
  verification rule:

Limits:
- max steps:
- max time:
- max retries:

Approval Before Submit:
- submit action:
  fields shown:
  side effect:
  approval reset condition:

Trace:
- screenshots:
- redaction:
- action log:
- retention:

Recovery Path:
- failure:
  behavior:

Eval Tasks:
- case:
  expected behavior:
```

Example:

```text
Workflow: Add internal support note

Allowed site:
admin.company.internal

Forbidden actions:
- delete customer
- change plan
- issue refund
- grant admin access

Approval Before Submit:
fields shown: customer_id, note text, submit button label
side effect: internal note created
approval reset condition: note text or customer_id changes

Recovery:
if modal appears, stop and ask human
if wrong customer page, stop
if submit validation fails, show error and do not retry submit automatically
```

Artifact complete when engineer can automate UI task with bounded goal, trace, approval, and recovery.

## Active Recall Questions

1. Why is computer use riskier than many API tools?
2. When should API be preferred over browser automation?
3. What should happen before a form submit?
4. Why do UI-driving systems need screenshot traces?
5. What are no-go actions?
6. Why do step limits matter?
7. What can go wrong with retrying UI submit?
8. What should recovery path handle?

## Summary

Computer use lets AI systems operate browsers and desktop apps, especially when APIs do not exist. It is powerful but fragile because UI state changes and side effects are easy.

Safe computer use needs bounded goals, allowed apps, forbidden actions, step limits, approval before submit, screenshot traces, redaction, resource verification, and recovery paths.

The most important rule:

> UI automation must stop before irreversible action unless exact action is approved.

## Preview of Next Chapter

Chapter 34 covered computer use and browser automation.

Chapter 35 defines agents without hype.

Next, you will learn when an agent loop is useful, when fixed workflow is better, and why agents need bounded goals, minimal tool sets, stop conditions, budgets, approval gates, and trace evaluation.
