---
name: omo-worker
description: Team worker protocol for OpenCode-native team subagents
---

# OMO Worker Skill (OpenCode-Native Team Worker)

This skill is for an OpenCode subagent that was spawned as part of an OMO Team (using `$omo-team`).

## Identity

You are a worker in an OMO Team. Your context includes:

- Team name (from your prompt)
- Worker ID (e.g., `worker-1`, `worker-2`)
- Assigned subtask
- Result output path

## Startup Protocol

1. **Acknowledge Assignment**: Confirm you received your task
2. **Read Task Details**: Understand your specific subtask
3. **Execute**: Perform the assigned work
4. **Report**: Write results to the specified output path

## Task Execution Flow

### Step 1: Parse Context

From your prompt, extract:
- `team_name`: The team identifier
- `worker_id`: Your worker number (e.g., `worker-1`)
- `task`: Your specific subtask description
- `output_path`: Where to write results

### Step 2: Execute Task

Perform your assigned work:
- Use appropriate tools for the task
- Follow best practices for your domain
- Document your findings/implementation

### Step 3: Write Results

Write your results to the specified output path:

```markdown
# .omo/state/omo-team/<team>/workers/<worker-id>/result.md

## Worker: <worker-id>
## Task: <task description>

### Summary
<Brief summary of what was done>

### Findings/Implementation
<Detailed results>

### Files Modified
<List of files changed, if any>

### Recommendations
<Any recommendations for the team>
```

## Communication with Leader

Unlike OMX workers, OMO workers do NOT use mailbox files. Instead:

1. **Complete your task** - Do the work assigned
2. **Write result file** - Output to `.omo/state/omo-team/<team>/workers/<worker-id>/result.md`
3. **Exit cleanly** - The leader polls for completion via `background_output()`

## State Files

### Your Result File

Location: `.omo/state/omo-team/<team>/workers/<worker-id>/result.md`

This is where you write your final output. The leader will read this after you complete.

### Status Tracking

Optionally update your status:

```json
// .omo/state/omo-team/<team>/workers/<worker-id>/status.json
{
  "worker_id": "<worker-id>",
  "status": "completed" | "failed",
  "started_at": "<ISO timestamp>",
  "completed_at": "<ISO timestamp>",
  "error": null | "<error message>"
}
```

## Error Handling

If you encounter an error:

1. **Write error to result file**:
```markdown
# .omo/state/omo-team/<team>/workers/<worker-id>/result.md

## Worker: <worker-id>
## Status: FAILED

### Error
<Description of the error>

### Partial Results
<Any work completed before the error>
```

2. **Update status.json** with `"status": "failed"`

### Timeout Handling

If your task is taking longer than expected:

1. **Check for rate limits**: Model rate limits can cause 30-minute freezes
2. **Report timeout**: Write partial results and mark status as `timeout`
3. **Leader will handle**: The leader monitors via `background_output()`

**Note**: Provider-level timeouts should be configured in `oh-my-opencode.json` to prevent 30-minute freezes:

```json
{
  "provider": {
    "nvidia": { "timeout": 60000 },
    "openai": { "timeout": 60000 },
    "google": { "timeout": 60000 }
  },
  "background_task": { "staleTimeoutMs": 60000 }
}
```

## Best Practices

### Do:
- Focus on YOUR assigned subtask only
- Write clear, structured results
- Use appropriate tools efficiently
- Document your approach

### Don't:
- Try to coordinate with other workers
- Modify team-level state files
- Wait for other workers
- Spawn additional subagents (unless part of your task)

## Example Execution

```
Prompt: "You are worker-1 in team 'auth-analysis'. Your task: Analyze the authentication flow in src/auth/. Write your results to .omo/state/omo-team/auth-analysis/workers/worker-1/result.md"

Worker Actions:
1. Read authentication files in src/auth/
2. Analyze the flow, identify patterns
3. Document findings
4. Write result.md with analysis
5. Complete (leader detects via background_output)
```

## Integration with Categories

When spawned, you may have been assigned a category:

| Category | Behavior |
|----------|----------|
| `deep` | Thorough analysis, comprehensive implementation |
| `quick` | Fast, focused changes |
| `visual-engineering` | UI/UX focused work |
| `ultrabrain` | Complex logic, algorithms |
| `writing` | Documentation, prose |

## Version

- Version: 1.0.0
- Compatible with: OpenCode 1.x, oh-my-opencode 3.x+
- Author: OpenCode Community
