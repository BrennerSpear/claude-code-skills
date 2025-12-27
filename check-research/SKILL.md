---
name: check-research
description: Check status of pending async research tasks and save completed results. Use when user wants to check if deep research tasks have finished.
---

# Check Research Status

Spawn a sub-agent to poll and save research results without polluting the main context.

## Instructions

Use the Task tool with `subagent_type: "general-purpose"` and this prompt:

```
Check pending research tasks and save completed results.

1. Find research-tracking.json in ~/repos/brain-dumps/ (most recent folder, or user-specified)
2. For each task with status "pending", call mcp__parallel-task__getResultMarkdown with the task_id
3. If complete:
   - Save the full markdown output to research/[slugified-title].md (create folder if needed)
   - Update the tracking entry: status="completed", add completed_at timestamp, add output_file path
4. If still running, note it as pending

Return ONLY a brief status table:
| Task | Status | File |
| Title | ✓ Complete | research/slug.md |
| Title | ⏳ Running | - |

Do NOT return the full research content - just save it to files.
```

Then relay the sub-agent's summary to the user.
