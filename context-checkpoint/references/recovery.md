# Emergency Recovery and Self-Healing

## Overview

This reference covers: checksum verification on resume, self-healing from redundancy blocks, emergency hard-stop saves, and the `/recover_last` command.

---

## Checksum Verification on Resume

When any checkpoint file is loaded, immediately verify its integrity:

### Verification Steps

1. Read Section 10 (Checksum Block) and extract the stored SHA-256 hash
2. Concatenate the text content of Sections 1-9 exactly as they appear in the file
3. Compute SHA-256 of the concatenation
4. Compare computed hash to stored hash

### Results

**Match:** Proceed normally. No integrity issues detected.

**Mismatch — minor (suspect copy-paste truncation):**

```
[!] Checksum mismatch detected. The checkpoint file may have been truncated or edited.
Attempting self-repair using Redundancy Block (Section 9)...
```

Proceed to self-healing (see below).

**Mismatch — TTL field changed:**

```
[!] Checksum mismatch detected. The TTL field appears to have been modified.
This may indicate the expiry date was altered to bypass the TTL check.
Loading is blocked. To override with acknowledgement: /resume force --acknowledge-tamper
```

**Mismatch — severe (>30% of content differs from hash):**

```
[!!] Severe checksum mismatch. The file may be corrupted or from a different source.
Recommend: obtain a fresh copy of the checkpoint from the original source.
You can attempt partial recovery with: /recover_last
```

---

## Self-Healing from Redundancy Block

When a minor mismatch is detected, attempt repair using Section 9 (Redundancy Block):

1. Extract the 3-5 critical facts from Section 9
2. Search Section 2 (Key Decisions) for each fact — check if present
3. For any missing fact, re-insert it into Section 2 with a note: `[RESTORED FROM REDUNDANCY]`
4. Report to user:

```
Self-repair complete.
Restored 2 facts from Redundancy Block:
+ [RESTORED] Use grain library for data loading (not tf.data)
+ [RESTORED] Batch size: 128 per TPU core

Sections 3, 4, 5 appear intact. Proceeding with resume.
Note: Verify restored facts are correct before relying on them.
```

5. Recompute checksum after repair and write updated Section 10.

---

## Emergency Hard Stop

If the token limit is hit unexpectedly mid-response (model error, unexpected cutoff):

### Immediate Actions

1. Save whatever conversation text is still accessible in the buffer
2. Mark the file header:

```
[INCOMPLETE — EMERGENCY SAVE]
Reason: Token limit hit unexpectedly at [timestamp]
Last confirmed message: [message number or description]
Possibly missing: messages after [last confirmed point]
```

3. In Section 4 (Current State), add:

```
### Emergency Save Note
The session ended abruptly. The following may be missing or incomplete:
- [Description of what was being worked on]
- [Last known state of the task]
- Last partial output (if any): [reproduce whatever was visible]
```

4. Output the incomplete file for download with filename: `context-checkpoint_vN_INCOMPLETE.md`

5. Show recovery instructions immediately:

```
[!!] EMERGENCY SAVE COMPLETE

Your session was saved as context-checkpoint_v3_INCOMPLETE.md

To recover:
1. Download the file immediately
2. In a new chat, type: /resume context-checkpoint_v3_INCOMPLETE.md
3. The AI will load what was saved and ask you to fill in any gaps from memory
4. Run /save to create a clean, complete checkpoint after you've verified the state
```

---

## `/recover_last` Command

When the user types `/recover_last` in a new session:

1. Ask the user to upload the most recent checkpoint file (complete or incomplete)
2. Load the file and display what was recovered:

```
Loaded: context-checkpoint_v3_INCOMPLETE.md
Status: INCOMPLETE — possible message loss after message 38

Recovered:
- Objectives: complete
- Key Decisions: 12 entries (appears complete)
- Artifacts: 3 files (train.py, collate_fn.py, config.yaml)
- Current State: partial — last confirmed task was "fixing NHWC bug"
- Open Issues: 4 items (may be outdated)

What may be missing:
- Messages 39+ are not in the file
- The session may have ended mid-task

To fill gaps: Tell me what happened after "fixing NHWC bug" and I will update the checkpoint.
```

3. Enter an interactive gap-filling mode where the user can verbally describe what was done after the cutoff point
4. Update all relevant sections based on user's description
5. Run a clean `/save` to produce a repaired, complete checkpoint

---

## Corruption Patterns and Remedies

| Symptom | Likely Cause | Remedy |
|---------|-------------|--------|
| Sections missing from file | File truncated during download | Re-download; if unavailable, use Redundancy Block |
| Code blocks show `[TRUNCATED]` | Manual editing or character limit in paste | Ask user to re-paste the original artifact |
| Checksum mismatch, content looks OK | Whitespace or encoding difference | Accept with warning; user confirms content is correct |
| TTL field changed | Manual edit | Warn of possible tamper; require `/resume force --acknowledge-tamper` |
| Section 9 empty | Checkpoint created before v2.0 of this skill | Proceed without self-healing; note in session |
| File is `.md.enc` or `.md.gpg` | Encrypted save | Instruct user to decrypt locally first, then upload |
