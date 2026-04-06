# Resume Modes and Commands

## Full Resume (`/resume [file or code]`)

### From uploaded .md file

1. User uploads `context-checkpoint_v3.md` to the new chat
2. User types `/resume` or "here's my checkpoint file"
3. AI reads the file and:
   a. Verifies checksum (Section 10) — warn if mismatch
   b. Checks TTL — refuse to load if expired (unless `/resume force`)
   c. Loads all sections into working context
   d. Surfaces any pending actions from Section 7: "You have 2 pending actions from last session. Execute now?"
   e. States current position: "Resuming at: [Section 4 Current State summary]. Next task: [Section 5 item 1]."

### From short code (`/resume 3F9K2A`)

1. Ask user to paste the URL and decryption key for their pastebin
2. Fetch and decrypt the content
3. Proceed as above

### Read-Only / Collaborative Resume

Generate a read-only variant of the checkpoint for handovers to another person or AI:

- Omit Section 3 raw code if confidential (replace with hash blocks)
- Add header: `READ-ONLY CONTEXT SHARE — recipient cannot modify original`
- Add: "To take ownership of this session, the originator must share an editable copy."

---

## Light Resume (`/resume light`)

Injects only:
- Section 6 (Continuation Prompt) — full text
- Section 5 (Open Issues) — top 3 items only
- Most recent artifact from Section 3 — one file only

Skips: full transcript, all decision history, older artifacts.

Token savings: ~60-80% compared to full resume.

Inform the user at the start of the resumed session:
"Running in light mode. If you need context from earlier in the session, type `/resume full` and upload the checkpoint file."

---

## Actionable Resume — Pending Task Execution

On any resume (full or light), check Section 7 (Pending Actions). If non-empty:

1. List all pending actions at the start of the session:

```
Pending actions from last session:
1. [tool_call] Run SQL migration on staging DB — NOT EXECUTED
2. [api_call] POST /api/deploy to trigger deployment — NOT EXECUTED

Shall I execute these now? (yes / no / skip item N)
```

2. Wait for user confirmation before executing any pending action.
3. On "yes": execute in order, report results.
4. On "no": clear the pending list and continue.
5. On "skip item 1": execute items 2+ only.

---

## Collaborative Context Merge (`/merge`)

When the user has two parallel chat contexts on related topics:

1. Ask the user to upload both checkpoint files: `checkpoint_A.md` and `checkpoint_B.md`
2. Compare Section 2 (Key Decisions) across both files — identify conflicts:

```
Conflict detected:
- Chat A (msg 14): Use PostgreSQL for primary storage
- Chat B (msg 9): Use MongoDB for primary storage

Which takes precedence? (A / B / merge both)
```

3. For each conflict, ask the user which version overrides.
4. Merge Sections 1, 2, 3, 4, 5, 7, and 8 from both files into a unified checkpoint:
   - De-duplicate identical entries
   - Preserve attribution: note which chat each decision came from
   - Use conflict resolutions chosen by user
5. Output merged file as `context-checkpoint_merged.md`

Inform the user: "Merged context from Chat A (N decisions, M artifacts) and Chat B (P decisions, Q artifacts). R conflicts resolved."

---

## Resume via Browser Extension (Companion Feature)

Note: This requires a companion browser extension not bundled with the skill. Describe the expected behaviour so an extension developer can implement it.

Expected extension behaviour:
1. Detects when a new chat window opens in a supported AI interface
2. Reads the most recent `context-checkpoint_vN.md` from the user's local download folder
3. Displays a toast: "Resume latest checkpoint? (context-checkpoint_v3.md)"
4. On click: inserts the Section 6 Continuation Prompt into the chat input and submits it
5. Then pastes Section 3 Artifacts as a follow-up message

The AI skill itself outputs a machine-readable metadata block at the top of each checkpoint for extension consumption:

```
<!-- EXTENSION-METADATA
version: 3
title: Zoglin ConvNeXt JAX Classifier
saved: 2026-04-05T14:45:00Z
resume_command: /resume
light_available: true
-->
```
