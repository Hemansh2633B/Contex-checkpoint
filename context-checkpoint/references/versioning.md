# Versioning, Bookmarks, and Master Index

## Version History

### Naming Convention

Each save produces a versioned file:

```
context-checkpoint_v1.md
context-checkpoint_v2.md
context-checkpoint_v3.md
...
```

Maintain a rolling window of the last 5 versions. When v6 is created, instruct the user to delete v1 (or archive it). The AI cannot delete files — it instructs the user.

### Version Header

Every checkpoint file includes a version header:

```
Version: v3
Previous: v2 (saved 2026-04-05T10:22:00Z)
```

---

## Diff Between Versions (`/diff v1 v2`)

When the user runs `/diff v2 v3` (requires both files to be uploaded or pasted):

1. Compare Section 2 (Key Decisions) — list added and removed decisions
2. Compare Section 5 (Next Steps) — list tasks completed, tasks added, tasks removed
3. Compare Section 3 (Artifacts) — list files added, removed, or modified
4. Show a summary diff:

```
### Diff: v2 -> v3
Date range: 2026-04-05T10:22 -> 2026-04-05T14:45

DECISIONS ADDED:
+ Chose orbax for checkpoint saving (not flax.training.checkpoints)
+ Fixed NHWC/NCHW channel ordering bug in collate_fn

DECISIONS REMOVED (superseded):
- Using tf.data for streaming (replaced by grain)

TASKS COMPLETED:
+ Fix channel ordering in collate_fn (was item 1 in v2)
+ Verify TPU device mesh (was item 2 in v2)

TASKS ADDED:
+ Run first full training epoch and log metrics
+ Profile memory usage per TPU core

ARTIFACTS MODIFIED:
~ train.py (was 180 lines, now 247 lines)

ARTIFACTS ADDED:
+ collate_fn.py (new file)
```

---

## Named Bookmarks (`/bookmark "name"`)

### Creating a Bookmark

When user types `/bookmark "API design decision point"`:

1. Record: current message number, current token count, timestamp
2. Add entry to Section 8 (Bookmarks Index) in the checkpoint
3. Confirm: "Bookmark saved: 'API design decision point' at message 18 (4,200 tokens)"

### Listing Bookmarks (`/list bookmarks`)

Display the bookmarks table from Section 8:

```
Bookmarks in this session:
#  Name                           Tokens   Msg#   Saved
1  API design decision point       4,200    18    2026-04-05 10:15
2  Database schema finalised       6,800    31    2026-04-05 12:40
3  Auth flow approved              8,100    38    2026-04-05 13:55
```

### Resuming from a Bookmark (`/resume bookmark "name"`)

On resume, restore the conversation state as it was at the bookmarked message number:

1. Load the checkpoint file
2. Truncate the conversation history at the bookmarked message number
3. Inject the state as of that point (decisions, artifacts, current state at that moment)
4. Inform: "Resumed from bookmark 'API design decision point' (message 18). Decisions and artifacts after this point have been excluded."

---

## Master Context Index (`_master_context_index.md`)

The master index tracks all checkpoints across sessions for a long-running project.

### Structure

```markdown
# Master Context Index: [Project Name]

Last updated: [ISO date]

## Adaptive Config
auto_trigger_threshold: 80%
preferred_compression: medium
resume_rate: 3/5 (last 5 saves, 3 were resumed)

## Snapshots

| # | File | Date | Summary | Key Decision |
|---|------|------|---------|--------------|
| 1 | context-checkpoint_v1.md | 2026-04-01 | Initial architecture | Chose JAX over PyTorch |
| 2 | context-checkpoint_v2.md | 2026-04-03 | Data pipeline | Chose grain over tf.data |
| 3 | context-checkpoint_v3.md | 2026-04-05 | Training loop | Fixed NHWC bug |

## Dependencies Between Snapshots
- v3 depends on v2 (grain pipeline must be working before training loop)
- v2 depends on v1 (architecture must be finalised before data pipeline)

## Cross-Snapshot Queries
To answer: "What decisions were made about authentication across all snapshots?"
Load Sections 2 from all snapshots and search for the keyword.
```

### Cross-Snapshot Queries

When the user asks "Give me all decisions about authentication across all sessions":

1. Ask the user to upload all relevant checkpoint files
2. Extract Section 2 from each file
3. Filter entries matching the query keyword
4. Return a consolidated list with source snapshot attribution:

```
Authentication decisions across all snapshots:
- [v1, msg 12] Use JWT tokens with 15-min expiry (refresh via Redis)
- [v2, msg 8] Ruled out session cookies — SSR incompatibility
- [v4, msg 22] Added MFA requirement for admin routes
```
