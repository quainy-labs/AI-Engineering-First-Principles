# Chapter 27: Vision, Image, and Multimodal Inputs

Chapter 26 handled documents: OCR, layout, tables, forms, confidence, validation, and review.

This chapter expands to broader visual inputs:

```text
screenshots
photos
charts
diagrams
UI states
whiteboards
product images
mixed text-image tasks
```

Vision models are useful when visual evidence matters. They are risky when teams treat visual interpretation as exact perception.

The engineering question is not:

> Can the model see?

The engineering question is:

> What visual evidence can the system safely use, validate, and act on?

## Concept Overview

Vision and multimodal AI systems process inputs that include images, text, layout, and sometimes multiple media types together.

They can help with:

- reading screenshots;
- identifying visible UI state;
- extracting visible text;
- interpreting charts or diagrams;
- detecting visible defects;
- summarizing images;
- combining image evidence with text instructions;
- routing cases that contain screenshots or photos.

But visual understanding is not perfect perception.

Vision systems must separate:

```text
Observed:
What is visibly present in the image.

Inferred:
What may be true based on visible evidence.

Uncertain:
What cannot be confidently determined.

Needs human review:
What should not be trusted automatically.
```

The first principle:

> Visual model output should distinguish observation from inference.

If the model says "the user is locked out," ask whether that is visible or inferred. The image may only show an error message. The actual account state may require database lookup.

## Why It Exists

Many user problems are visual.

Examples:

- user uploads screenshot of an error page;
- field technician uploads photo of broken equipment;
- analyst uploads chart from a report;
- designer uploads UI mockup;
- support agent uploads screen recording frame;
- customer uploads damaged product photo;
- employee uploads whiteboard diagram.

Text-only systems miss this evidence.

Consider a support user uploading a screenshot.

The system may need to identify:

- error message;
- product area;
- visible account state;
- button or form state;
- likely next step;
- whether sensitive data appears.

Bad default:

```text
Describe this image and solve the issue.
```

Problems:

- no task boundary;
- no confidence field;
- no crop or zoom strategy;
- no privacy check;
- no validation against known product states;
- no distinction between visible evidence and inference;
- no user correction path.

This chapter exists because visual AI should become a controlled evidence pipeline, not a free-form image description.

## Mental Model

Think of visual input as evidence from a camera, not direct truth.

The image captures a moment. It may be blurry, cropped, edited, outdated, misleading, or missing context. The model can observe patterns, but the system must decide how much weight those observations deserve.

The mental model:

```text
image input
-> quality check
-> privacy/sensitive data check
-> visual observation
-> structured output
-> validation against known sources
-> human review if uncertain or high risk
-> workflow action
```

For each visual task, ask:

```text
What is visible?
What is inferred?
What is missing?
What needs zoom/crop/OCR?
What should be validated elsewhere?
What should never trigger action automatically?
```

Vision is strongest when it supports a bounded workflow. It is weakest when the system asks it to make broad conclusions from incomplete visual evidence.

## Internal Mechanics

Vision systems begin with input handling.

### Image Quality

Image quality affects reliability.

Check:

- resolution;
- blur;
- crop;
- rotation;
- lighting;
- compression artifacts;
- text size;
- occlusion;
- duplicate uploads;
- unsupported file type.

If image quality is poor, the system should say so or request a better image instead of guessing.

### Privacy and Sensitive Data

Images can contain sensitive data:

- email addresses;
- names;
- phone numbers;
- account IDs;
- API keys;
- access tokens;
- payment details;
- health or legal information;
- faces;
- location data;
- internal URLs.

Design decisions:

```text
Should image be stored?
Should it be redacted?
Who can view it?
Should extracted text enter logs?
How long is image retained?
Can user delete it?
```

Sensitive data checks should happen before broad logging or sharing.

### Observation Extraction

The system should ask for task-specific observations.

Weak:

```text
Describe this image.
```

Stronger:

```text
Extract visible error message, product area, visible page title, selected button state, and any sensitive data.
Separate observed text from inference.
```

Useful output shape:

```text
observed_text
visible_ui_elements
visible_state
inferred_issue
uncertain_items
sensitive_data_detected
needs_human_review
```

### Crop and Zoom Strategy

Small text is easy to misread.

The system may need to:

- crop likely region;
- zoom screenshot;
- run OCR on region;
- ask user for higher-resolution image;
- request full screenshot instead of cropped snippet.

Do not pretend tiny text is reliable.

### Multimodal Context

Images often arrive with text.

Example:

```text
User message: I get this error when inviting a teammate.
Image: screenshot of invite page
```

The user text gives intent. The screenshot gives evidence. The system should combine them carefully.

Separate:

```text
user claim
visible evidence
system facts
model inference
```

If user says "billing page is broken" but screenshot shows account settings, the system should not blindly accept the user label.

### Validation Against System State

Visual evidence should often be validated.

Example:

Screenshot appears to show:

```text
Plan: Enterprise
```

But plan should be confirmed through account API before policy or billing action.

Use image for:

- identifying likely issue;
- routing;
- extracting visible text;
- requesting missing details;
- assisting support.

Use DB/API for:

- account truth;
- permission;
- current status;
- billing state;
- irreversible actions.

### Output Contracts

Vision output should be structured when software depends on it.

Example:

```json
{
  "observed_text": ["Error 403", "Access denied"],
  "visible_product_area": "workspace_settings",
  "visible_ui_state": "invite_member_modal",
  "inferred_issue": "permission_or_role_problem",
  "uncertain_items": ["exact workspace role not visible"],
  "sensitive_data_detected": true,
  "needs_human_review": true
}
```

The schema should forbid unsupported certainty.

## Real-World Example

Use case:

```text
Support screenshot triage
```

User uploads screenshot and says:

```text
I cannot invite a teammate.
```

Bad system:

```text
vision model describes screenshot
-> model tells user they lack admin permission
```

Problem:

The screenshot may show an error, but actual permission must come from the account system.

Better system:

```text
1. check image quality
2. detect sensitive data
3. extract visible text and UI state
4. classify likely product area
5. identify missing facts
6. query account API for user role if allowed
7. draft next step
8. route to human if evidence conflicts or sensitive data appears
```

Structured output:

```text
Observed:
- visible text: "You do not have permission to invite members"
- UI area: workspace member settings
- visible email address present

Inferred:
- likely permission issue

Uncertain:
- actual user role not visible
- workspace policy not visible

Needs validation:
- account role from workspace API
- invitation policy from product settings
```

The system uses vision as evidence, not final authority.

## Common Mistakes and Failure Modes

Mistake 1: Mixing observation and inference

"Error 403 is visible" is observation. "User lacks admin role" may be inference requiring API validation.

Mistake 2: Trusting small or blurry text

Tiny text, blur, and compression should trigger uncertainty, crop/zoom, OCR, or user retry.

Mistake 3: No sensitive data handling

Screenshots can contain secrets, customer data, internal URLs, and personal information.

Mistake 4: Solving from image alone

Visual input often needs database facts, product state, or policy retrieval.

Mistake 5: No crop or region strategy

Full screenshots may contain many unrelated regions. Important evidence may be small.

Mistake 6: Chart values treated as exact

Vision models may approximate chart values. Use source data when exact numbers matter.

Mistake 7: No user correction path

Users should be able to correct misread text, wrong UI state, or bad inference.

Mistake 8: Visual interpretation triggers irreversible action

High-impact actions should require validation or human review.

## System Design Decisions

Define input types:

```text
screenshots
photos
charts
diagrams
UI mockups
whiteboards
product images
mixed text-image requests
```

Define allowed observations:

```text
visible text
visible UI elements
visible error messages
visible object condition
chart labels
diagram components
presence of sensitive data
```

Define forbidden inferences:

```text
identity
intent beyond evidence
account truth
permission truth
medical/legal conclusions
safety-critical decisions
irreversible actions
```

Define quality handling:

```text
minimum resolution
blur threshold
crop/zoom rule
OCR fallback
ask user for better image
```

Define privacy handling:

```text
sensitive data detection
redaction
logging rules
retention
viewer permissions
delete path
```

Define validation:

```text
What must be confirmed by DB/API?
What must be confirmed by user?
What needs human review?
What can proceed automatically?
```

Define eval set:

```text
clear screenshot
blurry screenshot
cropped UI
chart
diagram
sensitive data
adversarial text in image
expected refusal
```

The goal is useful visual evidence with controlled uncertainty.

## Build Artifact

Create `vision-input-spec.md` for one visual workflow.

Use this template:

```text
Vision Input Spec

Workflow:
User goal:

Input Types:
- type:
  examples:
  quality requirements:

Allowed Observations:
- observation:
  output field:

Forbidden Inferences:
- inference:
  reason:
  correct validation source:

Sensitive Data:
- data type:
  detection rule:
  redaction/logging rule:

Output Schema:
- field:
  type:
  meaning:
  required:
  confidence/uncertainty rule:

Validation:
- visual claim:
  validation source:
  failure behavior:

Human Review:
- trigger:
  reviewer:
  action:

Eval Images:
- case:
  expected behavior:
```

Example:

```text
Workflow: Support screenshot triage

Allowed observation:
visible_error_message -> observed_text

Forbidden inference:
user lacks admin role
reason: role is not proven by screenshot
correct validation source: workspace permissions API

Sensitive data:
email address
detection rule: visible email-like text
redaction/logging rule: redact before trace storage

Human review:
trigger: sensitive data detected or action affects account access
```

The artifact is complete when another engineer can process visual input without confusing visible evidence, inference, privacy, and action.

## Active Recall Questions

1. Why should visual output separate observed, inferred, uncertain, and review-needed information?
2. What kinds of information can images carry?
3. Why is screenshot text not always reliable?
4. What visual claims should be validated through DB/API?
5. Why do screenshots need sensitive data handling?
6. When should the system ask for a better image?
7. Why should chart values often not be treated as exact?
8. When should visual interpretation route to human review?

## Summary

Vision and multimodal systems help AI use screenshots, photos, charts, diagrams, UI states, and mixed text-image inputs. They are powerful when visual evidence matters, but risky when uncertain perception becomes unsupported action.

Strong systems separate observation from inference, check image quality, detect sensitive data, use structured outputs, validate important claims, and route uncertain or high-impact cases to human review.

The most important rule:

> Use vision as evidence, not unchecked authority.

## Preview of Next Chapter

Chapter 27 handled visual inputs.

Chapter 28 moves into audio, speech, and realtime AI.

Next, you will learn how speech systems handle transcription, latency, turn-taking, interruption, consent, retention, correction, and realtime trust. This matters because realtime AI is not faster chat. It is interaction under timing pressure.
