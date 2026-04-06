# Analytics and Export

## Overview

After saving a checkpoint, the user can request derivative outputs that transform the saved conversation into reusable knowledge assets.

---

## Export Commands

### `/export pdf`

Converts the checkpoint to a structured PDF report. Instruct the user to run:

```bash
pip install md-to-pdf
md-to-pdf context-checkpoint_v3.md
```

Or, if Pandoc is available:

```bash
pandoc context-checkpoint_v3.md -o context-report.pdf --pdf-engine=weasyprint
```

The AI should also offer to reformat the checkpoint content into a cleaner report layout before export:

```markdown
# Project Report: [Title]
Generated: [date]

## Executive Summary
[one-paragraph summary of the entire session]

## Decisions Log
[Section 2 reformatted as a table]

## Technical Artifacts
[Section 3 with syntax-highlighted code]

## Open Items
[Section 5 as a prioritised action table]
```

---

### `/export timeline`

Generates a chronological timeline of decisions and milestones from the conversation:

```
PROJECT TIMELINE: Zoglin ConvNeXt Classifier
─────────────────────────────────────────────
2026-04-05 10:00  Session started
2026-04-05 10:15  Decision: Use JAX/Flax over PyTorch
2026-04-05 10:45  Decision: Use grain for data loading
2026-04-05 11:20  Artifact: model.py (ConvNeXt-V2 architecture) created
2026-04-05 12:10  Bug: NHWC/NCHW mismatch discovered
2026-04-05 14:30  Bug resolved: collate_fn channel ordering fixed
2026-04-05 14:45  Checkpoint saved (v3) — next: first training epoch
```

Derive timestamps from conversation ordering (relative timing if exact timestamps are unavailable: "~30 min into session").

---

### `/export quiz`

Generates a review quiz based on key decisions and facts from the conversation. Useful for knowledge transfer and onboarding.

Format:

```markdown
# Session Review Quiz: [Title]

**Q1:** What library was chosen for data loading and why was tf.data ruled out?
> Answer: grain was chosen. tf.data was ruled out due to TPU v5e-8 firmware incompatibility.

**Q2:** What was the root cause of the training loop crash at message 28?
> Answer: Channel ordering mismatch — model expected NHWC but collate_fn produced NCHW arrays.

**Q3:** What are the next three tasks after resuming?
> Answer: (1) Run first full training epoch, (2) profile memory per TPU core, (3) add orbax checkpoint saving every 500 steps.
```

Generate 5-10 questions minimum. Prioritise decisions, bugs, and next steps.

---

### `/export mermaid`

Generates a Mermaid diagram of the decision flow or system architecture inferred from the conversation:

```
gantt
    title Project Timeline
    dateFormat YYYY-MM-DD
    section Architecture
    Model design :done, 2026-04-01, 2026-04-02
    section Data Pipeline
    grain streaming setup :done, 2026-04-03, 2026-04-04
    section Training
    Fix NHWC bug :done, 2026-04-05, 2026-04-05
    First epoch :active, 2026-04-05, 2026-04-06
```

Or a flowchart of the system:

```
flowchart TD
    HF[HuggingFace Dataset] -->|grain streaming| DP[Data Pipeline]
    DP --> CF[collate_fn NHWC]
    CF --> TL[JAX Training Loop pmap]
    TL --> CK[orbax Checkpoint]
    TL --> LG[Metrics Logger]
```

---

## Voice and Multimodal Anchoring (Future-Proof)

For sessions conducted via voice or multimodal input:

### Voice Style Fingerprint

If a transcript of the user's speech is available, include a voice style summary in the checkpoint:

```
### Voice Style Summary
Speaking pace: [fast / moderate / slow]
Preferred phrasing: [technical / conversational]
Frequently used terms: [list]
Typical question structure: [e.g., "How do I X given Y?"]
```

This allows a voice-enabled AI on resume to match the established communication style without the user having to re-explain preferences.

### Identity Anchor (Non-Biometric)

Do NOT store biometric data. Store only:
- Self-reported user name or identifier
- Stated role or expertise level
- Communication preferences

Example:

```
### User Identity Anchor
Name: Himanshu (self-reported)
Role: ML engineer, Kaggle competitor
Expertise: Advanced (JAX, vLLM, TPU, RL)
Preferred style: No comments in code, step-by-step delivery
```

On resume, the AI greets accordingly and calibrates technical depth without requiring re-introduction.
