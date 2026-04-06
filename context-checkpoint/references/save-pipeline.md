# Save Pipeline — Detailed Section Formats

## Section 1: Objectives

List every goal the user stated, implied, or worked toward. Use bullet points. Be exhaustive and specific — avoid vague summaries like "build a model"; instead write "train a ConvNeXt-V2 image classifier in JAX/Flax for TPU v5e-8 with streaming HuggingFace datasets, targeting 90%+ validation accuracy."

## Section 2: Key Decisions and Conclusions

Capture every important decision, constraint, or conclusion reached during the session:

- Framework or library choices with reasons ("chose vLLM over Ollama for throughput on H100")
- Configs finalised (batch size, learning rate, LoRA rank, quantisation settings, exact version pins)
- Approaches ruled out and why
- Bugs found and their root cause
- Performance numbers observed
- Style preferences stated by the user (e.g., "no comments in code", "step-by-step delivery")

## Section 3: Artifacts

For every piece of code, config, schema, or document created or significantly edited during the session, reproduce the FULL content inside a fenced code block with the correct language tag. Label each artifact clearly:

```
### Artifact: training_loop.py
[full Python code]

### Artifact: config.yaml
[full YAML config]
```

Rules:
- Do NOT summarise or abbreviate code. No `# ... rest of code`, no `[truncated]`.
- Reproduction must be exact and complete so the next session can copy-paste and run without modification.
- If compression level is `high`, replace code blocks with a hash block (see `references/compression.md`) and note the external storage location.

## Section 4: Current State

Describe the exact state of the work at checkpoint time:

- What is working and verified (list with tick marks)
- What is partially done (list with dash)
- What is broken or blocked — reproduce the FULL error message or traceback exactly
- What was being worked on at the exact moment of export

## Section 5: Open Issues and Next Steps

List every unresolved question, bug, or planned next action. Use a numbered list ordered by priority. Include:

- The issue or task description
- Any known relevant code location (file name, function name, line number)
- Dependencies between tasks ("Task 3 requires Task 2 to be completed first")

## Section 6: Continuation Prompt

Write a self-contained prompt the user can paste verbatim into a new chat to resume. The prompt MUST be model-agnostic (no Claude-specific or GPT-specific phrasing). It must include:

- One paragraph introducing the project and its goal
- Current state (what works, what is broken, last error)
- The immediate next task
- All critical constraints (hardware, libraries, version pins, style preferences)
- An instruction to read the pasted Artifacts below

Format:

```
### Paste this into your new chat window:

[continuation prompt here — plain text, no markdown headers inside]

--- paste Artifacts section below this line ---
```

For model-specific adapter variants, see `references/cross-model.md`.

## Section 7: Pending Actions

List any tool calls, API calls, or actions the user instructed but that were not executed before the checkpoint. Format:

```
### Pending Action 1: [short label]
Type: [tool_call / api_call / file_write / other]
Description: [what was supposed to happen]
Parameters: [relevant inputs]
Status: NOT EXECUTED
```

On resume, the AI MUST surface these and ask: "Shall I execute the pending action [label] now?"

## Section 8: Bookmarks Index

List all named checkpoints created during the session using `/bookmark "name"`:

```
| # | Name | Token count at time | Message # | Date |
|---|------|--------------------:|----------:|------|
| 1 | API design decision point | 4200 | 18 | 2026-04-05 |
| 2 | Database schema finalised | 6800 | 31 | 2026-04-05 |
```

On `/resume bookmark "name"`, load the conversation state as it was at that message number.

## Section 9: Redundancy Block

Duplicate the 3-5 most critical facts from Sections 2, 4, and 5 verbatim. This block is used for self-healing if the main sections become corrupted or truncated. Label it clearly:

```
### REDUNDANCY — DO NOT EDIT

[critical fact 1]
[critical fact 2]
[critical fact 3]
```

## Section 10: Checksum Block

Compute SHA-256 of the concatenated text of Sections 1-9. Write:

```
SHA256: <hash>
Computed: [ISO datetime]
Covers: Sections 1-9
```

On resume, re-compute SHA-256 of the loaded file's Sections 1-9 and compare. If mismatch, invoke recovery (see `references/recovery.md`).
