# Compression and Selective Forgetting

## Overview

The compression pipeline reduces checkpoint file size and token footprint during resume without losing critical information. Apply before assembling the checkpoint file.

---

## Compression Levels

### Low (full fidelity)
- Keep all messages
- Reproduce all code blocks verbatim
- Use for short conversations or when the user explicitly wants full history

### Medium (default)
- Drop low-value messages (see criteria below)
- Keep all code blocks verbatim
- Summarise repetitive exchanges into one line
- Expected size reduction: 20-40%

### High (aggressive)
- Drop low-value messages
- Hash long code blocks (>50 lines): replace with a hash block
- Summarise all exploratory discussion into bullet points
- Expected size reduction: 50-70%

---

## Low-Value Message Identification

Messages eligible for removal (Medium and High compression):

| Category | Examples | Action |
|----------|----------|--------|
| Greetings and closings | "Hi", "Thanks", "Sure, let's go", "Got it!" | Drop entirely |
| Repetitive confirmations | "Yes, that's correct", "Exactly", "That works" (repeated) | Drop entirely |
| Filler acknowledgements | "Understood", "OK", "I see" | Drop entirely |
| Redundant re-explanation | AI re-explaining something the user already confirmed understanding of | Drop |
| Abandoned tangents | Threads where user said "never mind" or "let's skip this" | Drop entire thread |
| Tool call echoes | AI narrating what it is about to do before doing it | Drop; keep the result only |

Do NOT drop:
- Any message containing a decision, constraint, or conclusion
- Any message containing code, config, or schema
- Error messages or tracebacks
- User-stated preferences or requirements
- The last 5 messages before the checkpoint (preserve recency)

---

## Code Block Hashing (High compression only)

For code blocks longer than 50 lines, replace the full block with a hash block:

```
### Artifact: training_loop.py [HASHED]
SHA256: 9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a8b
Lines: 247
Summary: JAX/Flax training loop with pmap, cosine LR schedule, gradient clipping at 1.0, and checkpoint saving every 500 steps via orbax.
External storage: User confirmed full code saved locally at ~/projects/zoglin/train.py
```

Rules:
- Always ask the user to confirm they have the full code saved locally before hashing
- Never hash code shorter than 50 lines
- Always include a summary accurate enough to reconstruct intent
- On resume, if the user pastes the full file, the AI verifies the hash matches before proceeding

---

## Summarisation of Exploratory Discussion

For long exploratory sections (brainstorming, approach comparison), compress into a structured summary:

```
### Compressed: Approach Exploration (messages 8-22)
Explored 3 approaches for data loading:
1. tf.data: Ruled out — TPU firmware incompatibility on v5e-8
2. grain (chosen): Compatible, streaming confirmed working
3. PyTorch DataLoader: Ruled out — framework mismatch with JAX

Decision: Use grain library with streaming=True.
```

---

## Context Size Estimation

Before compressing, estimate raw character count:

```
Raw conversation: ~N characters
After medium compression: ~M characters (~X KB)
After high compression: ~P characters (~Y KB)
```

Present this to the user and ask for compression level preference before proceeding.

---

## Selective Context Injection on Resume

When the user resumes with `/resume light`:

1. Inject ONLY:
   - Section 6 (Continuation Prompt)
   - Section 5 (Open Issues and Next Steps — top 3 items only)
   - The single most recent artifact
2. Skip: full transcript, all decisions history, older artifacts
3. Inform the user: "Running in light mode — only critical state injected. Type `/resume full` to load the complete checkpoint."

Light mode saves ~60-80% of tokens in the new session and is ideal for mobile users or simple task continuation.
