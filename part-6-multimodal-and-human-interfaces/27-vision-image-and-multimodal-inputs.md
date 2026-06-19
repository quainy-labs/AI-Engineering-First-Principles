# Chapter 27: Vision, Image, and Multimodal Inputs

Chapter 26 handled documents. This chapter handles broader visual inputs: images, screenshots, diagrams, charts, photos, and mixed text-image tasks.

Vision models are useful when visual evidence matters. They are risky when teams treat visual interpretation as exact perception.

## Real Problem

A support user uploads a screenshot of an error page.

The system must identify:

- error message;
- product area;
- visible account state;
- likely next step;
- whether sensitive data appears.

Text-only systems miss the screenshot. Vision systems can help, but they can also misread small text or infer things not visible.

## Bad Default Solution

Bad solution:

```text
Describe this image and solve the issue.
```

Problems:

- no task boundary;
- no confidence field;
- no crop/zoom strategy;
- no privacy check;
- no validation against known product states;
- no distinction between visible evidence and inference.

## First Principles

Images carry:

- visible text;
- objects;
- spatial relationships;
- UI state;
- charts;
- visual defects;
- sensitive data.

Vision model output should separate:

```text
Observed:
Inferred:
Uncertain:
Needs human review:
```

For engineering, the key question is not "Can model see?" It is "What visual evidence can the system safely use?"

## System Decision

Use vision when:

- visual state matters;
- screenshots/photos are primary input;
- OCR alone is insufficient;
- user needs interpretation of layout, chart, or object.

Avoid or require review when:

- safety-critical perception;
- identity verification;
- medical/legal visual judgment;
- tiny text or poor image quality;
- visual evidence affects irreversible action.

## Minimal Build

Create `vision-input-spec.md`:

```text
Input type:
User task:
Allowed observations:
Forbidden inferences:
Sensitive data check:
Output schema:
Confidence/uncertainty rule:
Human review rule:
Eval images:
```

## Failure Modes

Vision systems fail when:

- model invents hidden context;
- small text is misread;
- chart values are approximated;
- sensitive data is logged;
- image quality is not checked;
- output mixes observation and conclusion;
- no user correction path exists.

## Evaluation

Use images with:

- clear screenshot;
- blurry screenshot;
- cropped UI;
- chart;
- diagram;
- sensitive data;
- adversarial text in image;
- expected refusal.

Measure:

- observation accuracy;
- text reading accuracy;
- hallucinated visual claim rate;
- sensitive-data detection;
- uncertainty accuracy.

## Proof Artifact

Create `vision-input-spec.md`.

## Next

Next chapter covers audio, speech, and realtime AI, where timing and turn-taking become part of system design.
